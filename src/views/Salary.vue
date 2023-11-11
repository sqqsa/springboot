<template>
  <div>

    <div style="padding: 10px 0">
      <el-input style="width: 200px" placeholder="请输入名称" suffix-icon="el-icon-search" v-model="nickname"></el-input>
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
      <el-upload action="http://localhost:9090/salary/import" :show-file-list="false" accept="xlsx"
                 :on-success="handleExcelImportSuccess" style="display: inline-block">
        <el-button type="primary" class="ml-5" @click="">导入<i class="el-icon-download"></i></el-button>
      </el-upload>
      <el-button type="primary" @click="exp" class="ml-5">导出<i class="el-icon-upload2"></i></el-button>
    </div>

    <el-table :data="tableData"border stripe :header-cell-class-name="headerBg" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column prop="id" label="ID" width="80">
      </el-table-column>
      <el-table-column prop="nickname" label="昵称" width="100">
      </el-table-column>
      <el-table-column prop="deptName" label="部门" width="120">
      </el-table-column>
      <el-table-column prop="baseSalary" label="基本工资" width="120">
      </el-table-column>
      <el-table-column prop="subsidy" label="生活补贴" width="120">
      </el-table-column>
      <el-table-column prop="bonus" label="奖金"width="120">
      </el-table-column>
      <el-table-column prop="deduction" label="扣款"width="120">
      </el-table-column>
      <el-table-column prop="totalSalary" label="总工资"width="120">
      </el-table-column>
      <el-table-column prop="month" label="月份">
      </el-table-column>
      <el-table-column prop="remark" label="备注">
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button type="success" @click="handleEdit(scope.row)">编辑<i class="el-icon-edit" style="margin-left: 5px"></i></el-button>
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

    <el-dialog title="菜单信息" :visible.sync="dialogFormVisible" width="30%" >
      <el-form label-width="80px" size="small">
        <el-form-item label="员工">
          <el-select clearable v-model="form.userId" placeholder="请选择员工" style="width: 100%">
            <el-option v-for="item in userList":key="item.id" :label="item.nickname" :value="item.id" ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="基本工资">
          <el-input-number v-model="form.baseSalary" :min="1" :max="100000" label="描述文字"></el-input-number>

          <span style="margin-left: 40px;margin-right: 10px" >生活补贴</span>
          <el-input-number v-model="form.subsidy"  :min="1" :max="100000" label="描述文字"></el-input-number>

        </el-form-item>

        <el-form-item label="奖金">
          <el-input-number v-model="form.bonus"  :min="1" :max="100000" label="描述文字"></el-input-number>

          <span style="margin-left: 40px;margin-right: 10px" >&emsp;&emsp;扣款</span>
          <el-input-number v-model="form.deduction"  :min="1" :max="100000" label="描述文字"></el-input-number>

        </el-form-item>

        <el-form-item label="总工资">
          <el-input-number v-model="form.totalSalary" style="width:48%" :disabled="true"
                           :controls="false" :precision="3"/>
        </el-form-item>

        <el-form-item label="月份">
        <el-date-picker
            v-model="form.month"
            type="month"
            placeholder="选择月">
        </el-date-picker>
        </el-form-item>

        <el-form-item label="描述">
          <el-input v-model="form.remark" autocomplete="off"></el-input>
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
  name: "Salary",
  data(){
    return {
      tableData: [],
      total:0,
      nickname:"",
      dialogFormVisible:false,
      pageNum:1,
      pageSize:5,
      sideWidth:200,
      logoTextShow: true,
      headerBg:'headerBg',
      form:{},
      userList:[],
      user:localStorage.getItem("user") ? JSON.parse(localStorage.getItem("user")) : {}
    }

  },
  watch: {
    'form.baseSalary': function () {
      this.calculateSalary()
    },
    'form.subsidy': function () {
      this.calculateSalary()
    },
    'form.bonus': function () {
      this.calculateSalary()
    },
    'form.deduction': function () {
      this.calculateSalary()
    }
  },
  created() {
    this.load()
  },

  methods: {
    //计算工资
    calculateSalary() {
      this.form.totalSalary = this.form.baseSalary +
          this.form.subsidy + this.form.bonus - this.form.deduction

    },
    load(){
      this.request.get("/salary/page",{
        params:{
          pageNum:this.pageNum,
          pageSize:this.pageSize,
          nickname:this.nickname,
          baseSalary: this.baseSalary,
          subsidy: this.subsidy,
          bonus: this.bonus,
          deduction:this.deduction
        }
      }).then(res =>{
        this.tableData = res.data.records
        this.total = res.data.total
      })

      this.request.get("/user").then(res => {
        this.userList = res.data
      })

    },
    reset(){   //重置
      this.username=""
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
    save(){ //确认按钮添加员工
      this.request.post("/salary",this.form).then(res =>{
        if (res.code == '200'){
          this.$message.success("保存成功")
          this.dialogFormVisible = false
          this.load()
        }else {
          this.$message.error("保存失败")
        }
      })
    },
    //删除
    del(id) {
      this.request.delete("/salary/"+id).then(res =>{
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
      this.request.delete("/salary/del/batch",{data:ids}).then(res =>{
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
      this.request.post("/salary",row)
      this.load()
    },
    //导出
    exp(){
      window.open("http://localhost:9090/salary/export")
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