<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片 -->
    <el-card class="box-card">
      <!-- 搜索功能 -->
      <div style="margin-top: 15px;">
        <el-row :gutter="25">
          <el-col :span="6">
            <el-input placeholder="请输入内容"  class="input-with-select" v-model="getParams.query" clearable @clear="getUserList">
              <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
            </el-input>
          </el-col>
          <el-col :span="4">
              <el-button type="primary" @click="addUserDialogVisible = true">添加用户</el-button>
          </el-col>
        </el-row>

        <!-- 表格 -->
        <el-table
            :data="userList"
            style="width: 100%"
            border stripe>
            <el-table-column
              type="index"
              width="50">
            </el-table-column>
            <el-table-column
              prop="username"
              label="用户名"
              width="180">
            </el-table-column>
            <el-table-column
              prop="mobile"
              label="手机号">
            </el-table-column>
            <el-table-column
              prop="email"
              label="邮箱"
              width="180">
            </el-table-column>
            <el-table-column
              prop="role_name"
              label="角色">
            </el-table-column>
            <el-table-column
              prop="create_time"
              label="注册时间">
            </el-table-column>
            <!-- 卡槽操作 -->
            <el-table-column 
            label="状态"
            prop="mg_state">
              <template slot-scope="scope">
                <el-switch v-model="scope.row.mg_state" @change="changUserState(scope.row)">
                </el-switch>
              </template>
            </el-table-column>
            <!-- 用作用域插槽 -->
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" circle  @click="getUserInfo(scope.row)"></el-button>
                <el-button type="danger" icon="el-icon-delete"  size="mini" circle @click="delConfirm(scope.row)"></el-button>
                <el-tooltip class="item" effect="dark" content="分配角色"  placement="top" :enterable="false">
                  <el-button type="warning" icon="el-icon-setting" size="mini"  circle @click="giveRole(scope.row)"></el-button>
                </el-tooltip>
              </template>
            </el-table-column>
        </el-table>
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="getParams.pagenum"
          :page-sizes="[1, 2, 3, 4]"
          :page-size="getParams.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>
        <!-- 添加用户对话框 -->
        <el-dialog
          title="添加用户"
          :visible.sync="addUserDialogVisible"
          width="50%"
          @close ="colseAddUserDialog"
          >
        <el-form :model="addForm" ref="addFormRefs" :rules="addRules" label-width="70px" class="demo-ruleForm">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addForm.password"  type="password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号" prop="mobile">
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部按钮 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="addUserDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 修改用户信息对话框 -->
      <el-dialog
        title="修改用户信息"
        :visible.sync="editUserDialogVisible"
        width="50%"
        >
        <el-form :model="editForm" ref="editFormRefs" :rules="editRules" label-width="70px" class="demo-ruleForm">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="editForm.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部按钮 -->
        <span slot="footer" class="dialog-footer">
          <el-button @click="editUserDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUser">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 分配角色信息对话框 -->
      <el-dialog
        title="分配权限"
        :visible.sync="roleDialogVisible"
        @close ="colseRoleDialog"
        width="50%">
        <div>
          <p>当前的用户：{{userInfo.username}}</p>
          <p>当前的角色：{{userInfo.role_name}}</p>
          <p>分配新角色：
            <el-select v-model="selectRole" placeholder="请选择">
              <el-option
                v-for="item in roleList"
                :key="item.id"
                :label="item.roleName"
                :value="item.id">
              </el-option>
            </el-select>
          </p>
        </div>
        <span slot="footer" class="dialog-footer">
          <el-button @click="roleDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="confirmRoleSelect">确 定</el-button>
        </span>
      </el-dialog>
      </div>
    </el-card>
  </div>
</template>

<script>
import { async } from 'q';
import { userInfo } from 'os';

export default {
  data () {
    //验证邮箱
    var checkEmail = (rule, value, callback) => {
      //callback是回调函数

      //验证邮箱规则
      const regEmail=  /^[A-Za-z0-9\u4e00-\u9fa5]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/
      if(regEmail.test(value)) {
        return callback()
      } 
      callback(new Error('邮箱错误'))
    };
    //验证手机号
    var checkMobile = (rule, value, callback) => {
      //验证邮箱规则
      const regMobile=  /^[1][3,4,5,6,7,8,9][0-9]{9}$/
      if(regMobile.test(value)) {
        return callback()
      } 
      callback(new Error('手机号错误'))
    };
    return {
      userList:[],
      total:0,
      getParams:{
        query:'',
        pagenum:1,
        pagesize:2 //当前显示多少条数据
      },
      addUserDialogVisible:false, //添加用户对话框
      editUserDialogVisible:false, //修改用户对话框
      addForm:{
        username:'',
        password:'',
        email:'',
        mobile:''
      },
      editForm:{},
      //检验规则
      addRules: {
          username: [
            { required: true, message: '请输入用户名', trigger: 'blur' },
            { min: 3, max: 10, message: '长度在 3 到 5 个字符', trigger: 'blur' }
          ],
          password: [
            { required: true, message: '请输入密码', trigger: 'blur' },
            { min: 3, max: 10, message: '密码长度在 3 到 5 个字符', trigger: 'blur' }
          ],
          email: [
            { required: true, message: '请输入邮箱', trigger: 'blur' },
            { validator: checkEmail, trigger: 'blur' }           
          ],
          mobile: [
            { required: true, message: '请输入手机号', trigger: 'blur' },
            { validator: checkMobile, trigger: 'blur' }       
          ]

      },
      //编辑校验规则
      editRules: {
          email: [
            { required: true, message: '请输入邮箱', trigger: 'blur' },
            { validator: checkEmail, trigger: 'blur' }           
          ],
          mobile: [
            { required: true, message: '请输入手机号', trigger: 'blur' },
            { validator: checkMobile, trigger: 'blur' }       
          ]
      },
      roleDialogVisible:false, //分配角色对话框是否可见
      userInfo:{},//被分配角色的用户信息
      roleList:[], //分配角色的角色列表
      selectRole:'' //分配角色中选择的角色
    }
  }
  ,created(){
    //获取用户列表
    this.getUserList()
  }
  ,methods:{
    async getUserList(){
      var {data:res} = await this.$http.get('users',{params:this.getParams})
      if(res.meta.status !=200) return this.$message.error(res.meta.msg)
      this.userList = res.data.users
      this.total = res.data.total
    }
    //页显示数量事件
    ,handleSizeChange(newsize){
      this.getParams.pagesize = newsize
      this.getUserList()
    }
    //当前页码值点击事件
    ,handleCurrentChange(newpagenum){
      this.getParams.pagenum = newpagenum
      this.getUserList()
    }
    //修改用户状态事件
    ,async changUserState(userinfo){
      const {data:res} = await this.$http.put(`users/:${userinfo.id}/state/:${userinfo.mg_state}`)
      if(res.meta.status ==200) {
        userinfo.mg_state = ! userinfo.mg_state
        this.$message.error(res.meta.msg)
      }
      this.$message.success('用户状态更新成功')
    }
    //监听对话框关闭事件，重置表单内容
    ,colseAddUserDialog(){
      this.$refs.addFormRefs.resetFields()
    }
    //添加用户信息
    ,addUser(){
      //表单预校验
      this.$refs.addFormRefs.validate(async (valid)=>{
        if(!valid) return;
        //发起请求
        const {data : result } = await this.$http.post('users',this.addForm)
        if(result.meta.status != 201) return this.$message.error(result.meta.msg)
        this.$message.success('添加用户成功')
        this.addUserDialogVisible = false
        this.getUserList()
      })
    }
    //编辑获取用户信息
    ,async getUserInfo(userinfo){
      const {data : res } = await this.$http.get(`users/${userinfo.id}`)
      if(res.meta.status != 200) return this.$message.error(res.meta.msg)
      this.editForm = res.data
      this.editUserDialogVisible = true
    }
    //编辑提交用户修改信息
    ,editUser(){
      //先进行表单的预验证
      this.$refs.editFormRefs.validate(async (valid)=>{
        if(!valid) return;
        const {data : res } = await this.$http.put(`users/${this.editForm.id}`,{email:this.editForm.email,mobile:this.editForm.mobile})
        if(res.meta.status != 200) return this.$message.error(res.meta.msg)
        //先关闭对话框，再获取数据，再提示成功
        this.editUserDialogVisible = false
        this.getUserList()
        this.$message.success('修改用户信息成功')
      })
    }
    //删除确认框
    ,async delConfirm(userinfo) {
      const res =await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
      }).catch(err => err)
      //如果是确定返回confirm，是取消则返回cancel
      if(res !== 'confirm') return this.$message.error('取消成功')
      const delRes = await this.$http.delete(`users/${userinfo.id}`)
      console.log(delRes)
      if(delRes.data.meta.status !=200) return this.$message.error(delRes.data.meta.msg)
      this.getUserList()
      this.$message.success(delRes.data.meta.msg)

    }
    //给用户分配角色
    ,async giveRole(userinfo){
      this.userInfo = userinfo
      //在显示框前，展示所有的角色列表
      const {data:res} = await this.$http.get('roles')
      if(res.meta.status !==200) return this.$message.error('获取角色列表失败')
      this.roleList = res.data
      this.roleDialogVisible = true
    }
    //确定选择的角色
    ,async confirmRoleSelect(){
      //如果未选择角色返回
      if(!this.selectRole)  return this.$message.error('请选择角色')
      const {data:res} = await this.$http.put(`users/${this.userInfo.id}/role`,{rid:this.selectRole})
      if(res.meta.status !==200) return this.$message.error('分配权限失败')
      this.$message.success('设置角色成功')
      this.getUserList()
      this.roleDialogVisible = false
    }
    ,colseRoleDialog(){
      //对话框关闭重置选择角色的值
      this.selectRole =''
      this.userInfo =[]
    }
  }
  

}
</script>