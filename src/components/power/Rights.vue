<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
     <!-- 卡片 -->
    <el-card class="box-card">
      <!-- 权限列表 -->
      <el-table
        :data="rightsList"
        style="width: 100%"
        border stripe>
          <el-table-column
            type="index">
          </el-table-column>
          <el-table-column
            prop="authName"
            label="权限名称">
          </el-table-column>
          <el-table-column
            prop="path"
            label="路径">
          </el-table-column>
          <el-table-column
            prop="level"
            label="权限等级"
            width="180">
            <template slot-scope="scope">
              <el-tag :type="tagsList[scope.row.level].type">{{tagsList[scope.row.level].name}}</el-tag>
            </template>
          </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>
<script>
  export default {
    data () {
      return {
        rightsList:[], //权限列表
        tagsList:{ //权限等级
          0:{type:'success',name:'一级'},
          1:{type:'info',name:'二级'},
          2:{type:'danger',name:'三级'}
        }
      }
    }
    ,created(){
      this.getRightsList()
    }
    ,methods:{
      async getRightsList(){
        var {data:res} = await this.$http.get('rights/list')
        if(res.meta.status !=200) return this.$message.error(res.meta.msg);
        this.rightsList = res.data
        console.log(res)
      }
    }
  }
</script>
