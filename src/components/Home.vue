<template>
  <el-container>
    <el-header>Header
      <el-button type="info" @click="logOut">退出登录</el-button>
    </el-header>
    <el-container>
      <el-aside width="200px">Aside</el-aside>
      <el-main>Main</el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  methods:{
    logOut(){
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script>
<style>
</style>