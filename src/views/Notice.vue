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
      <el-upload action="http://localhost:9090/notice/import" :show-file-list="false" accept="xlsx"
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
      <el-table-column prop="name" label="标题" width="140">
      </el-table-column>
      <el-table-column prop="content" label="内容" width="150">
      </el-table-column>
      <el-table-column prop="time" label="发布时间">
      </el-table-column>
      <el-table-column prop="user" label="发布人">
      </el-table-column>
      <el-table-column label="图片">
        <template slot-scope="scope">
          <el-image style="width: 150px"  :src="scope.row.img" :preview-src-list="[scope.row.img]"></el-image>
        </template>
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

    <el-dialog title="信息" :visible.sync="dialogFormVisible" width="30%">
      <el-form label-width="80px" size="small">
        <el-form-item label="标题">
          <el-input v-model="form.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="内容">
          <el-input type="textarea" v-model="form.content" autocomplete="off"></el-input>
        </el-form-item>


<!--        <el-form-item label="发布时间">-->
<!--          <el-date-picker-->
<!--              type="datetime"-->
<!--              value-format = 'yyyy-MM-dd HH:mm:ss'-->
<!--              v-model="form.time"-->
<!--              placeholder="选择日期时间">-->
<!--          </el-date-picker>-->
<!--        </el-form-item>-->
        <el-form-item label="封面">
          <el-upload
              class="avatar-uploader"
              action="http://localhost:9090/file/upload"
              ref="img"
              :show-file-list="false"
              :on-success="handleImgUploadSuccess"
              >
            <img v-if="form.img" :src="form.img" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
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
  name: "Notice",
  data(){
    return {
      tableData: [],
      total:0,
      name:"",
      dialogFormVisible:false,
      pageNum:1,
      pageSize:5,
      form:{},
      multipleSelection:[],
      user: localStorage.getItem("user") ? JSON.parse(localStorage.getItem("user")) : {}
    }

  },
  created() {
    this.load()
  },
  methods: {
    load() {
      this.request.get("/notice/page", {
        params: {
          pageNum: this.pageNum,
          pageSize: this.pageSize,
          name: this.name,
          type: 1
        }
      }).then(res => {
        this.tableData = res.data.records
        this.total = res.data.total
      })
    },
    reset(){   //重置
      this.name=""
      this.load()
    },
    handleAdd(){ //新增按钮显示表单
      this.dialogFormVisible = true
      this.form = {img: ''}
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
    save(){ //确认按钮添加
      this.form.type = 1
      this.request.post("/notice",this.form).then(res =>{
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
      this.request.delete("/notice/"+id).then(res =>{
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
      this.request.delete("/notice/del/batch",{data:ids}).then(res =>{
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
    handleImgUploadSuccess(res) {
      this.form.img = res
      this.load()
    },
    //导出
    exp(){
      window.open("http://localhost:9090/notice/export")
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
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409EFF;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>