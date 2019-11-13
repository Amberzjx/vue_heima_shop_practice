<template>
  <el-container class="home-container">
    <el-header class="header-container">
      <div class="header-left">
        <img src="../assets/tcl_logo.png" alt="">
        <span>后台管理系统</span>
      </div>
      <el-button type="info" @click="logOut">退出登录</el-button>
    </el-header>
    <el-container>
      <el-aside :width="collapse?'65px':'200px'" class="aside-container">
          <div class="toggle-btn" @click="toggleCollapse()">|||</div>
          <el-menu
          :unique-opened ="true"
          background-color="#323744"
          text-color="#fff"
          active-text-color="#3599FC"
          :collapse="collapse"
          :collapse-transition = "false"
          router
          :default-active = "defaultActive"
          >
          <el-submenu :index="item.id + '' " v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i :class="iconsList[item.id]" ></i>
              <span>{{item.authName}}</span>
            </template>
            <el-menu-item 
              :index="'/'+subitem.path" 
              v-for="subitem in item.children" 
              :key="subitem.id"
              @click="savePathState('/'+subitem.path)"
              >
              <i class="el-icon-menu"></i>
              <span>{{subitem.authName}}</span>
            </el-menu-item>
          </el-submenu>
      </el-menu>
      </el-aside>
      <el-main class="main-container">
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data(){
    return {
      menuList:[],
      iconsList:{
        125:'iconfont icon-users',
        103:'iconfont icon-tijikongjian',
        101:'iconfont icon-shangpin',
        102:'iconfont icon-danju',
        145:'iconfont icon-baobiao'
      },
      collapse:false,
      //激活的地址
      defaultActive:''
    }
  }
  , created(){
     this.getMenuList()
     this.defaultActive = window.sessionStorage.getItem('defaultActive')
   }
  , methods:{
    logOut(){
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
    ,async getMenuList(){
        var {data : res } =await this.$http.get('menus')
        if(res.meta.status !=200) return this.$message.error(res.meta.msg)
        this.menuList = res.data
    }
    //点击按钮实现菜单折叠与展开
    ,toggleCollapse(){
        this.collapse = ! this.collapse
    }
    ,savePathState(path){
      window.sessionStorage.setItem('defaultActive',path)
      this.defaultActive = path
    }
  }
}
</script>
<style lang="less" scoped>
.home-container{
  height: 100%;
}
#app .header-container{
  padding: 0px;
}
.header-container{
  background-color: #373C41;
  display: flex;
  justify-content: space-around;
  align-items: center;
  color: aliceblue;
  font-size: 28px;
  margin-left: -255px;
}
.header-container img{
  width: 20%;
}
.header-left{
  display: flex;
  align-items: center;
}
.aside-container{
  background-color: #323744;
  > el-menu {
    border: none;
  }
}
.main-container{
  background-color: #EAEDF2;
}
.iconfont{
  margin-right: 8px;
}
.toggle-btn{
  background-color:#485166;
  font-size: 10px;
  line-height: 24px;
  color: #ffffff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>