<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
     <!-- 卡片 -->
    <el-card class="box-card">
      <!-- 添加 -->
      <el-row>
        <el-col>
            <el-button type="primary">添加用户</el-button>
        </el-col>
        </el-row>
      <!-- 列表 -->
      <el-table
        :data="rolesList"
        style="width: 100%"
        border stripe>
          <!-- 展开列 使用作用域插槽方式-->
          <el-table-column type="expand">
            <template slot-scope="scope">
              <!-- 三层for循环 -->
              <el-row v-for="(itemOne,one) in scope.row.children" :key="itemOne.id" :class="['borderRow',one==0?'borderTop':'','stayCenter']">
                <!-- 渲染一级权限 -->
                <el-col :span="5">
                  <el-tag  closable @close="removeRightById(scope.row,itemOne.id)">{{itemOne.authName}}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                
                <!-- 渲染2、3级权限 -->
                <el-col :span="19">
                  <!-- 循环2级权限 -->
                  <el-row v-for="(itemTwo,Two) in itemOne.children" :key="itemTwo.id" :class="[Two==0?'':'borderTop','stayCenter']">
                    <el-col :span="6">
                      <el-tag type="success" closable @close="removeRightById(scope.row,itemTwo.id)">{{itemTwo.authName}}</el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
                    <!-- 三级 -->
                    <el-col :span="18">
                      <el-tag type="warning" v-for="(itemThree,Three) in itemTwo.children" :key="itemThree.id" closable @close="removeRightById(scope.row,itemThree.id)">
                        {{itemThree.authName}}
                      </el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
            </template>
          </el-table-column>
          <!-- 索引列 -->
          <el-table-column
            type="index">
          </el-table-column>
          <el-table-column
            prop="roleName"
            label="角色名称">
          </el-table-column>
          <el-table-column
            prop="roleDesc"
            label="角色描述">
          </el-table-column>
          <el-table-column
            label="操作">
            <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini"   >编辑</el-button>
                <el-button type="danger" icon="el-icon-delete"  size="mini" >删除</el-button>
                <el-button type="warning" icon="el-icon-setting" size="mini" @click="settingRight(scope.row)" >分配权限</el-button>
            </template>
          </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限的对话框列表 -->
    <el-dialog
      title="分配权限"
      :visible.sync="settingRightDialogVisiable"
      width="50%" @close="colseSettingRight">
      <!-- 树形控件 -->
      <el-tree :data="rightList" :props="treePropsName" 
      show-checkbox node-key="id" default-expand-all check-on-click-node 
      :default-checked-keys="defaultCheckKey"
      ref= "treeRef">
      </el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="settingRightDialogVisiable = false">取 消</el-button>
        <el-button type="primary" @click="confirmSetRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
  export default {
    data () {
      return {
        rolesList:[], //角色列表
        settingRightDialogVisiable:false, //分配权限对话框
        rightList:[], //权限
        treePropsName:{
          children: 'children',
          label: 'authName'
        },
        roleId:'',//即将给其分配权限的id值
        defaultCheckKey:[] //每个角色默认有的权限，并选中
      }
    }
    ,created(){
      this.getRolesList()
    }
    ,methods:{
      async getRolesList(){
        var {data:res} = await this.$http.get('roles')
        if(res.meta.status !=200) return this.$message.error(res.meta.msg);
        this.rolesList = res.data
      }
      //根据对应id删除权限
      ,async removeRightById(role,right){
        //弹出弹框确认是否删除
        const confirm = await this.$confirm('是否确认删除该权限, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>err)
        if(confirm !='confirm'){
          return this.$message.error('取消删除操作')
        } 
        //发起删除权限请求
        const {data:res } = await this.$http.delete(`roles/${role.id}/rights/${right}`)
        if(res.meta.status !=200){
          return this.$message.error('删除权限失败')
        }
        //为当前的角色重新赋值权限
        role.children = res.data
        this.$message.success('删除权限成功')
      }
      //分配权限对话框listing
      ,async settingRight(role){
        this.roleId = role.id
        const {data:res} =await this.$http.get('rights/tree')
        this.rightList = res.data

        this.getLeafKey(role,this.defaultCheckKey) //递归获取三级节点的id
        this.settingRightDialogVisiable = true //对话框可见
      }
      //递归函数，获取角色的所有三级权限，合成一个数组defaultCheckKey
      ,getLeafKey(node,arr){
        if(!node.children){
          return arr.push(node.id)
        }
        node.children.forEach(item => this.getLeafKey(item,arr));
      }
      //关闭对话框的时候，清空defaultCheckKey数组的所有值
      ,colseSettingRight(){
        this.defaultCheckKey =[]
      }
      //确定提交权限
      ,async confirmSetRights(){
        const keys = [
          ...this.$refs.treeRef.getCheckedKeys(),
          ...this.$refs.treeRef.getHalfCheckedNodes()
        ]
        const idStr = keys.join(',') //将数组keys以逗号分隔拼接成字符串
        //发起请求
        const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
        if(res.meta.status !==200){
          this.$message.error('分配权限失败')
        }
        this.$message.success('分配权限成功')
        this.getRolesList()
        this.settingRightDialogVisiable = false
      }
    }
  }
</script>
<style lang="less" scoped>
.el-tag{
  margin: 7px;
}
.borderRow{
  border-bottom: 1px solid #eee;
}

.borderTop{
  border-top: 1px solid #eee;
}
.stayCenter{
  display: flex;
  align-items: center;
}
</style>