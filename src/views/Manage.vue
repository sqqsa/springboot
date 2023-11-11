<template>
    <el-container style="min-height: 100vh;">
      <el-aside :width="sideWidth + 'px' " style="background-color: rgb(238, 241, 246);box-shadow: 2px 0 6px rgba(0 21 41 0.35)">
        <Aside :isCollapse="isCollapse" :logoTextShow="logoTextShow"/>

      </el-aside>

      <el-container>
        <el-header style="border-bottom: 1px solid #ccc">
          <Header :collapseBtnClass="collapseBtnClass" :collapse="collapse" :user="user"/>
        </el-header>

        <el-main>
<!--          表示当前页面的子路由会在router-view里面展示-->
           <router-view @refreshUser="getUser"/>
        </el-main>
      </el-container>
    </el-container>

</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'
import Aside from "@/components/Aside"
import Header from "@/components/Header";
export default {
  name: 'Home',
  components: {
    HelloWorld,
    Aside,
    Header

  },
  data(){
    return {
      collapseBtnClass:"el-icon-s-fold",
      isCollapse:false,
      sideWidth:200,
      logoTextShow: true,
      user: localStorage.getItem("user") ?
          JSON.parse(localStorage.getItem("user")) : {}
    }
  },
  created() {
    //请求分页查询数据
    // this.load()
    this.getUser()
  },
  methods: {
    collapse(){ //点击收缩按钮触发
      this.isCollapse = !this.isCollapse
      if(this.isCollapse){ //收缩
        this.sideWidth = 64
        this.collapseBtnClass = 'el-icon-s-unfold'
        this.logoTextShow = false
      }else {   //展开
        this.sideWidth = 200
        this.collapseBtnClass = 'el-icon-s-fold'
        this.logoTextShow = true
      }
    },
    getUser() {
      //从后台获取User数据
      this.request.get("/user/username/" + this.user.username).then(res => {
        //重新赋值后台的最新User数据
        this.user = res.data
      })
    }
  }
};
</script>
