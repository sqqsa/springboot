<template>
  <div>

    <div style="padding: 10px 0">
      <el-input style="width: 200px" placeholder="请输入名称" suffix-icon="el-icon-search" v-model="nickname"></el-input>
      <el-button class="ml-5" type="primary" @click="load">搜索</el-button>
      <el-button type="warning" @click="reset">重置</el-button>
    </div>

    <div style="margin: 10px 0">
      <el-button type="warning" @click="signIn">上班打卡<i class="el-icon-sunny" round></i></el-button>
      <el-button type="primary" @click="signOut">下班打卡<i class="el-icon-moon"></i></el-button>
      <el-popconfirm
          class="ml-5"
          confirm-button-text='删除'
          cancel-button-text='取消'
          icon="el-icon-info"
          icon-color="red"
          title="您确定批量删除这些数据吗？"
          @confirm="delBatch"
      >
        <el-button type="danger"slot="reference">批量删除<i class="el-icon-remove-outline"></i></el-button>
      </el-popconfirm>
<!--      <el-upload action="http://localhost:9090/attendance/import" :show-file-list="false" accept="xlsx"-->
<!--                 :on-success="handleExcelImportSuccess" style="display: inline-block">-->
<!--        <el-button type="primary" class="ml-5" @click="">导入<i class="el-icon-download"></i></el-button>-->
<!--      </el-upload>-->
<!--      <el-button type="primary" @click="exp" class="ml-5">导出<i class="el-icon-upload2"></i></el-button>-->
    </div>

    <el-table :data="tableData"border stripe :header-cell-class-name="headerBg" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column prop="id" label="ID" width="80">
      </el-table-column>
      <el-table-column prop="nickname" label="昵称" width="120">
      </el-table-column>
      <el-table-column prop="deptName" label="部门" width="80">
      </el-table-column>
      <el-table-column prop="phone" label="电话">
      </el-table-column>
      <el-table-column prop="startTime" label="打卡时间">
      </el-table-column>
      <el-table-column prop="typeNum" label="打卡类型">
      </el-table-column>
      <el-table-column prop="status" label="打卡状态">
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
<!--          <el-button type="success" @click="handleEdit(scope.row)">编辑<i class="el-icon-edit" style="margin-left: 5px"></i></el-button>-->
          <el-popconfirm
              class="ml-5"
              confirm-button-text='删除'
              cancel-button-text='取消'
              icon="el-icon-info"
              icon-color="red"
              title="您确定删除吗？"
              @confirm="del(scope.row.id)"
          >
            <el-button type="danger" slot="reference">删除<i class="el-icon-remove-outline" style="margin-left: 5px"></i></el-button>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>

    <div style="padding: 10px 0">
      <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="pageNum"
          :page-sizes="[2, 5, 10, 20]"
          :page-size="pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
      </el-pagination>
    </div>

  </div>
</template>

<script>


export default {
  name: "Performance",
  data(){
    return {
      tableData: [],
      total:0,
      nickname:"",
      dialogFormVisible:false,
      pageNum:1,
      pageSize:5,
      collapseBtnClass:"el-icon-s-fold",
      isCollapse:false,
      sideWidth:200,
      logoTextShow: true,
      headerBg:'headerBg',
      form:{},
      multipleSelection:[],
      user:localStorage.getItem("user") ? JSON.parse(localStorage.getItem("user")) : {}
    }

  },
  created() {
    this.load()
  },
  methods: {
    load(){
      this.request.get("/attendance/page",{
        params:{
          pageNum:this.pageNum,
          pageSize:this.pageSize,
          nickname:this.nickname,
        }
      }).then(res =>{
        this.tableData = res.data.records
        this.total = res.data.total
      })

    },
    reset(){   //重置
      this.nickname=""
      this.load()
    },
    handleAdd(){ //新增按钮显示表单
      this.dialogFormVisible = true
      this.form = {}
    },
    handleEdit(row){ //编辑按钮
      this.form = JSON.parse(JSON.stringify(row))
      this.dialogFormVisible = true
    },
    signIn(){ //签到打卡
      this.request.get("/attendance/signIn").then(res =>{
        if (res.data){
          this.$message.success("打卡成功")
          this.load()
        }else {
          this.$message.error("打卡失败")
        }
      })
    },
    signOut(){ //确认按钮添加员工
      this.request.get("/attendance/signOut").then(res =>{
        if (res.data){
          this.$message.success("打卡成功")
          this.load()
        }else {
          this.$message.error("打卡失败")
        }
      })
    },
    del(id) {
      this.request.delete("/attendance/"+id).then(res =>{
        if (res.data){
          this.$message.success("删除成功")
          this.load()
        }else {
          this.$message.error("删除失败")
        }
      })
    },
    handleSelectionChange(val){
      this.multipleSelection = val;

    },
    //批量删除
    delBatch(){
      let ids = this.multipleSelection.map(v => v.id) //把对象的数组变成纯id的数组
      this.request.delete("/attendance/del/batch",{data:ids}).then(res =>{
        if (res.data){
          this.$message.success("批量删除成功")
          this.load()
        }else {
          this.$message.error("批量删除失败")
        }
      })
    },
    handleSizeChange(pageSize){
      this.pageSize= pageSize
      this.load()
    },
    handleCurrentChange(pageNum){
      this.pageNum = pageNum
      this.load()
    },
    handleStatusChange(row) {
      this.request.post("/attendance",row)
      this.load()
    },
    //导出
    exp(){
      window.open("http://localhost:9090/attendance/export")
    },
    //导入
    handleExcelImportSuccess() {
      this.$message.success("导入成功")
      this.load()
    }
  }
}
</script>

<style scoped>

</style>