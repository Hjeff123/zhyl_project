<template>
  <div style="height: 100%">
    <el-container style="height: 100%" direction="vertical">
      <MyHeader></MyHeader>
      <NavMenu isActive="6"></NavMenu>
      <el-main style="background-color: #FFFFFF">
        <div style="height: 100%">
          <el-page-header @back="goBack" content="挂号查询界面" style="margin-left: 168px">
          </el-page-header>
          <div class="title">
            挂号信息
          </div>
          <div style="display: flex;justify-content: left;margin-top: 50px;margin-left:150px">
            <el-container>
              <el-main>
                <el-table
                    :data="tableData"
                    border
                    style="width: 90%;font-size:18px">
                  <el-table-column
                      prop="patientName"
                      label="病患姓名"
                      width="150">
                  </el-table-column>
                  <el-table-column
                      prop="doctorId"
                      label="医生编号"
                      width="120">
                  </el-table-column>
                  <el-table-column
                      prop="doctorName"
                      label="医生姓名"
                      width="120">
                  </el-table-column>
                  <el-table-column
                      prop="departmentName"
                      label="科室"
                      width="120">
                  </el-table-column>
                  <el-table-column
                      prop="orderTimeSlice"
                      label="预约时间段"
                      width="120">
                  </el-table-column>
                  <el-table-column
                      prop="orderTime"
                      label="预约日期"
                      width="220">
                  </el-table-column>
                  <el-table-column
                      prop="createTime"
                      label="预约单生成时间"
                      width="220">
                  </el-table-column>
                  <el-table-column
                      prop="cost"
                      label="花费"
                      width="120">
                  </el-table-column>
                  <el-table-column
                      prop="deal"
                      label="已处理"
                      width="100">
                  </el-table-column>
                  <el-table-column
                      prop="detail"
                      fixed="right"
                      label="操作"
                      width="240">
                    <template slot-scope="scope">
                      <el-button v-if="scope.row.deal==='是'" @click="handleClick1(scope.row.orderId, scope.row.orderTime)" type="success">查看</el-button>
                      <el-button v-if="scope.row.deal==='否'" @click="handleClick1(scope.row.orderId, scope.row.orderTime)" type="success" disabled>查看</el-button>
                      <span v-if="scope.row.deal==='否'"><el-button type="danger"
                                                                     @click="deleteRow(scope.row.orderId)">取消预约</el-button></span>
                      <span v-if="scope.row.deal==='是'"><el-button type="danger" disabled>取消预约</el-button></span>
                      <el-button v-if="scope.row.deal==='是' && scope.row.orderTime === todayDate" @click="handleClick2(scope.row.orderId)" type="primary">医技预约</el-button>
                      <el-button v-if="scope.row.deal==='否' || scope.row.orderTime !== todayDate" @click="handleClick2(scope.row.orderId)" type="primary" disabled>医技预约</el-button>
                    </template>
                  </el-table-column>
                </el-table>

                <el-dialog title="详细信息" :visible.sync="formVisible" width="45%">
                  <el-tabs v-model="activeName" type="card" @tab-click="handleClick" stretch>
                    <el-tab-pane label="病历信息" name="first">
                      <el-input type="textarea" :rows="10" v-model="medicalrecord" readonly style="font-size: 20px"></el-input>
                    </el-tab-pane>
                    <el-tab-pane label="医生处方" name="second">
                      <el-input type="textarea" :rows="10" v-model="prescription" readonly style="font-size: 20px"></el-input>
                    </el-tab-pane>
                    <el-tab-pane label="医技推荐" name="third">
                      <el-input type="textarea" :rows="10" v-model="technicianRecommend" readonly style="font-size: 20px"></el-input>
                      <el-button v-if="orderTime === todayDate" type="success" @click="orderTechnician" style="margin-top: 10px">一键预约</el-button>
                      <el-button v-if="orderTime !== todayDate" type="success" @click="orderTechnician" style="margin-top: 10px" disabled>一键预约</el-button>
                    </el-tab-pane>
                  </el-tabs>
                </el-dialog>
                <el-dialog title="预约失败的医技" :visible.sync="formVisible1" width="25%">
                  <span v-for="item in failList">{{item}}<br/></span>
                </el-dialog>

              </el-main>
            </el-container>
          </div>
        </div>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import MyHeader from "@/components/MyHeader";
import NavMenu from "@/components/NavMenu";

export default {
  name: "orderQuery",
  components: {NavMenu, MyHeader},
  data() {
    return {
      technicianRecommend: '',
      recommendtech: [],
      formVisible: false,
      medicalrecord: '',
      prescription: '',
      patientId: '',
      tableData: [],
      todayDate: '',
      orderId: 0,
      formVisible1: false,
      failList: [],
      orderTime: ''
    }
  },
  mounted() {
    this.todayDate = this.getCurrentDate()
    this.getTableData()
  },
  methods: {
    orderTechnician(){
      let user = localStorage.getItem("user") ? JSON.parse(localStorage.getItem("user")) : null
      if (user){
        let params = new FormData()
        params.append('orderId', this.orderId)
        params.append('patientId', user.userId)
        params.append('createTime', this.todayDate)
        this.request.post('/medicalResource/appointment/system', params).then(res => {
          if (res.code === '200'){
            this.formVisible = false
            if (res.data.length > 0){
              this.failList = res.data
              this.formVisible1 = true
            }else{
              this.$message.success('一键预约成功')
              this.$router.push('/TechQuery')
            }
          }else{
            this.$message.error(res.msg)
          }
        })
      }
    },
    getCurrentDate(n) {
      let dd = new Date();
      if (n) {
        dd.setDate(dd.getDate() - n);
      }
      let year = dd.getFullYear();
      let month =
          dd.getMonth() + 1 < 10 ? "0" + (dd.getMonth() + 1) : dd.getMonth() + 1;
      let day = dd.getDate() < 10 ? "0" + dd.getDate() : dd.getDate();
      return year + "-" + month + "-" + day;
    },
    handleClick2(orderId){
      this.$router.push({path: '/TechOrder', query: {orderId: orderId}})
    },
    getTableData() {
      let user = localStorage.getItem("user") ? JSON.parse(localStorage.getItem("user")) : null
      if (user) {
        this.request.get('/user/orders/' + user.userId).then(res => {
          if (res.code === '200') {
            this.tableData = this.handleTableData(res.data)
          }else{
            this.tableData = []
          }
        })
      } else {
        this.$message('请先登录')
        this.$router.push('/')
      }
    },
    handleTableData(tableData) {
      for (let item of tableData){
        if (item.gender === true){
          item.gender = '男'
        }else{
          item.gender = '女'
        }
        if (item.deal === true){
          item.deal = '是'
        }else{
          item.deal = '否'
        }
        if (item.orderTimeSlice === 1){
          item.orderTimeSlice = '8:00-9:00'
        }else if (item.orderTimeSlice === 2){
          item.orderTimeSlice = '9:00-10:00'
        }else if (item.orderTimeSlice === 3){
          item.orderTimeSlice = '10:00-11:00'
        }else if (item.orderTimeSlice === 4){
          item.orderTimeSlice = '14:00-15:00'
        }else if (item.orderTimeSlice === 5){
          item.orderTimeSlice = '15:00-16:00'
        }else{
          item.orderTimeSlice = '16:00-17:00'
        }
      }
      return tableData
    },
    goBack() {
      this.$router.push('/')
    },
    handleClick1(orderId, orderTime) {
      this.request.get('/doctor/caseHistory/' + orderId).then(res => {
        if (res.code === '200'){
          this.medicalrecord = res.data
        }
      })
      this.request.get('/doctor/prescription/' + orderId).then(res => {
        if (res.code === '200'){
          this.prescription = res.data
        }
      })
      this.request.get('/medicalResource/recommend/' + orderId).then(res => {
        if (res.code === '200'){
          this.technicianRecommend = res.data
          this.orderTime = orderTime
        }else{
          this.orderTime = ''
        }
      })
      this.formVisible = true
      this.orderId = orderId
    },
    deleteRow(orderId) {
      this.$confirm('此操作将取消您的预约, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.request.delete('/order/appointment/' + orderId).then(res => {
          if (res.code === '200'){
            this.$message({
              message: '取消预约成功',
              type: 'success'
            })
            this.getTableData()
          }else{
            this.$message.error(res.code)
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消操作'
        });
      });
    }
  }
}
</script>

<style scoped>
.title {
  -webkit-text-size-adjust: 50%;
  /* font-family: SourceHanSansCN-Regular,Hiragino Sans GB,Microsoft YaHei,-apple-system-font,Helvetica Neue,sans-serif; */
  margin: 0;
  outline: none;
  /* box-sizing: border-box; */
  /* position: relative; */
  background-color: #008080;
  /* padding: 10px 0; */
  text-align: center;
  font-size: 20px;
  /* font-weight: 700; */
  color: #fff;
  margin-top: 50px;
}

.demo-chip {
  margin: 8px;
  vertical-align: middle;
}
</style>