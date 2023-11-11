<template>
  <div>

    <div style="padding: 10px 0">
      <el-input style="width: 200px" placeholder="请输入名称" suffix-icon="el-icon-search" v-model="name"></el-input>
<!--      <el-input style="width: 200px" placeholder="请输入邮箱" suffix-icon="el-icon-user-solid" class="ml-5" v-model="email"></el-input>-->
<!--      <el-input style="width: 200px" placeholder="请输入电话" suffix-icon="el-icon-phone" class="ml-5" v-model="phone"></el-input>-->
      <el-button class="ml-5" type="primary" @click="load">搜索</el-button>
      <el-button type="warning" @click="reset">重置</el-button>
    </div>

    <div style="margin: 10px 0">
      <el-button type="primary" @click="handleAdd">新增<i class="el-icon-circle-plus-outline"></i></el-button>
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
      <el-upload action="http://localhost:9090/dept/import" :show-file-list="false" accept="xlsx"
                 :on-success="handleExcelImportSuccess" style="display: inline-block">
        <el-button type="primary" class="ml-5" @click="">导入<i class="el-icon-download"></i></el-button>
      </el-upload>
      <el-button type="primary" @click="exp" class="ml-5">导出<i class="el-icon-upload2"></i></el-button>
    </div>

    <el-table :data="tableData" border stripe :header-cell-class-name="'headerBg'"
              row-key="id" default-expand-all @selection-change="handleSelectionChange" >
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column prop="id" label="ID" width="80">
      </el-table-column>
      <el-table-column prop="name" label="名称" width="140">
      </el-table-column>
      <el-table-column prop="createTime" label="成立时间" width="150">
      </el-table-column>
      <el-table-column prop="morStartTime" label="上午上班时间">
      </el-table-column>
      <el-table-column prop="morEndTime" label="上午下班时间">
      </el-table-column>
      <el-table-column prop="aftStartTime" label="下午上班时间">
      </el-table-column>
      <el-table-column prop="aftEndTime" label="下午下班时间">
      </el-table-column>
<!--      <el-table-column prop="total_work_time" label="工作时长">-->
<!--      </el-table-column>-->
      <el-table-column prop="remark" label="备注">
      </el-table-column>
      <el-table-column label="操作" width="300">
        <template slot-scope="scope">
          <el-button type="warning" v-if="scope.row.parentId === 0" @click="handleSubAdd(scope.row.id)">新增 <i
              class="el-icon-circle-plus-outline"/></el-button>
          <el-button type="success" @click="handleEdit(scope.row)">编辑<i class="el-icon-edit"></i></el-button>
          <el-popconfirm
              class="ml-5"
              confirm-button-text='删除'
              cancel-button-text='取消'
              icon="el-icon-info"
              icon-color="red"
              title="您确定删除吗？"
              @confirm="del(scope.row.id)"
          >
            <el-button type="danger" slot="reference">删除<i class="el-icon-remove-outline"></i></el-button>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog title="部门信息" :visible.sync="dialogFormVisible" width="30%">
      <el-form label-width="80px" size="small">
        <el-form-item label="名称">
          <el-input v-model="form.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="上午时间">
          <el-time-picker

              format = 'HH:mm:ss' value-format = 'HH:mm:ss'
              class="mr-5"
              v-model="form.morStartTime"
              :picker-options="{
      selectableRange: '06:00:00 - 10:00:00'
    }"
              placeholder="上班时间">
          </el-time-picker>

          <el-time-picker

              format = 'HH:mm:ss' value-format = 'HH:mm:ss'
              arrow-control
              v-model="form.morEndTime"
              :picker-options="{
      selectableRange: '10:10:00 - 12:00:00'
    }"
              placeholder="下班时间">
          </el-time-picker>
        </el-form-item>

        <el-form-item label="下午时间">
          <el-time-picker

              format = 'HH:mm:ss' value-format = 'HH:mm:ss'
              class="mr-5"
              v-model="form.aftStartTime"
              :picker-options="{
      selectableRange: '13:00:00 - 15:00:00'
    }"
              placeholder="上班时间">
          </el-time-picker>

          <el-time-picker
              format = 'HH:mm:ss' value-format = 'HH:mm:ss'
              arrow-control
              v-model="form.aftEndTime"
              :picker-options="{
      selectableRange: '16:10:00 - 20:00:00'
    }"
              placeholder="下班时间">
          </el-time-picker>
        </el-form-item>

        <el-form-item label="备注">
          <el-input type="textarea" v-model="form.remark"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="save">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "Dept",
  data(){
    return {
      tableData: [],
      total:0,
      name:"",
      remark:"",
      dialogFormVisible:false,
      pageNum:1,
      pageSize:5,
      collapseBtnClass:"el-icon-s-fold",
      isCollapse:false,
      sideWidth:200,
      logoTextShow: true,
      headerBg:'headerBg',
      form:{},
      multipleSelection:[]
    }

  },
  created() {
    this.load()
  },
  methods: {
    load(){
      this.request.get("/dept",{
        params:{
          name:this.name,
        }
      }).then(res =>{
        this.tableData = res.data
      })
    },
    reset(){   //重置
      this.name=""
      this.load()
    },
    handleAdd(){ //新增按钮显示表单
      this.dialogFormVisible = true
      this.form = {}
    },
    handleSubAdd(parentId){ //新增按钮显示表单
      this.dialogFormVisible = true
      this.form = {}
      if (parentId) {
        this.form.parentId = parentId
      }

    },
    // handleSelectionChange(val){
    //   this.multipleSelection = val;
    //
    // },
    handleEdit(row){ //编辑按钮
      this.form = JSON.parse(JSON.stringify(row))
      this.dialogFormVisible = true
    },
    save(){ //确认按钮添加员工
      this.request.post("/dept",this.form).then(res =>{
        if (res.data){
          this.$message.success("保存成功")
          this.dialogFormVisible = false
          this.load()
        }else {
          this.$message.error("保存失败")
        }
      })
    },
    del(id) {
      this.request.delete("/dept/"+id).then(res =>{
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
      this.request.delete("/dept/del/batch",{data:ids}).then(res =>{
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
    //导出
    exp(){
      window.open("http://localhost:9090/dept/export")
    },
    //导入
    handleExcelImportSuccess() {
      this.$message.success("导入成功")
      this.load()
    }
  }
}
</script>

<style>
.headerBg{
  background: #eee !important;
}
</style>