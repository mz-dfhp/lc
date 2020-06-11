<template>
  <div>
    <!-- 导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-row :gutter="20">
        <!-- 搜索框 -->
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="params.query" clearable @clear='userData'>
            <el-button slot="append" icon="el-icon-search" @click='userData'></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click='addDialogVisible=true'>添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table :data="userDataList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column prop="mg_state" label="状态">
          <template slot-scope='scope'>
            <!-- {{scope.row}} -->
            <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope='scope'>
            <!-- 修改按钮 -->
            <el-button type="primary" icon="el-icon-edit" size="mini" @click='showEditDialog(scope.row.id)'></el-button>
            <!-- 删除按钮 -->
            <el-button type="danger" icon="el-icon-delete" size="mini" @click='removeUserById(scope.row.id)'></el-button>
            <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
            <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页功能 -->
      <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="params.pagenum"
            :page-sizes="[1, 2, 5, 10]"
            :page-size="params.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加用户对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="50%" @close="addDialogClose">
      <!-- 主体内容区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef"  label-width="70px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
        </el-form>
        <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 添加用户对话框结束 -->
    <!-- 修改用户对话框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%" @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
        </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
  export default {
    data() {
      const checkEmail=(rule,value,callback)=>{
        const regEmail=/^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
        if(regEmail.test(value)){
          return callback()
        }
        callback(new Error('请输出合法的邮箱'))
      }
      const checkMobile=(rule,value,callback)=>{
        const regMobile=/^((13[0-9])|(17[0-1,6-8])|(15[^4,\\D])|(18[0-9]))\d{8}$/
        if(regMobile.test(value)){
          return callback()
        }
        callback(new Error('请输出合法手机号'))
      }
      return {
        userDataList: [],
        // 获取用户列表的参数对象
        params: {
          query: '',
          // 当前页数
          pagenum: 1,
          // 当前每页显示多少条数据
          pagesize: 2
        },
        total:0,
        // 控制添加用户对话框的显示与隐藏
        addDialogVisible:false,
        //添加用户的表单数据
        addForm:{
          username:'',
          password:'',
          email:'',
          mobile:''
        },
        //添加表单的验证规则对象
        addFormRules:{
          username:[
            {required:true,message:'请输入用户名',trigger:'blur'},
            {min:3,max:10,message:'用户的长度在3~10个字符之间',trigger:'blur'}
          ],
          password:[
            {required:true,message:'请输入密码',trigger:'blur'},
            {min:6,max:16,message:'用户的长度在6~16个字符之间',trigger:'blur'}
          ],
          email:[
            {required:true,message:'请输入邮箱',trigger:'blur'},
            {validator:checkEmail,trigger:'blur'},

          ],
          mobile:[
            {required:true,message:'请输入手机号码',trigger:'blur'},
            {validator:checkMobile,trigger:'blur'},
          ],
        },
        // 控制修改用户对话框的显示与隐藏
        editDialogVisible:false,
        // 查询到用户的信息对象
        editForm:{},
        //修改表单的验证对象
        editFormRules:{
          email:[
            {required:true,message:'请输入邮箱',trigger:'blur'},
            {validator:checkEmail,trigger:'blur'}
          ],
          mobile:[
            {required:true,message:'请输入手机号码',trigger:'blur'},
            {validator:checkMobile,trigger:'blur'}
          ],
        }


      }
    },
    created() {
      this.userData()
    },
    methods: {
      async userData() {
        const {
          data: res
        } = await this.$http.get('users', {
          params: this.params
        })
        if (res.meta.status !== 200) return this.$message.error('获取管理员列表失败')
        // console.log(res)
        this.userDataList = res.data.users
        this.total=res.data.total
        console.log(this.userDataList)
        // console.log(res.data)
      },
      //监听页码值改变的事件
      handleSizeChange(newSize){
        this.params.pagesize=newSize
        this.userData()
      },
      //监听页码改变的事件
      handleCurrentChange(newPage){
        this.params.pagenum=newPage
        this.userData()
      },
      async userStateChanged(e){
        console.log(e)
        const{data:res}= await this.$http.put(`users/${e.id}/state/${e.mg_state}`)
        if(res.meta.status!==200){
          e.mg_state=!e.mg_state
          return this.$message.error('更新用户状态失败')
        }
        this.$message.success('更新用户状态成功')
      },
      addDialogClose(){
        this.$refs.addFormRef.resetFields()
      },
      addUser(){
        this.$refs.addFormRef.validate(async valid=>{
          console.log(valid)
          if(!valid) return
          // 验证通过 可以发起网络请求
          const {data:res} =await this.$http.post('users',this.addForm)
          if(res.meta.status!==201) {
            return this.$message.error('添加用户失败')
            }
          this.$message.success('添加用户成功')
          this.addDialogVisible=false
          this.userData()
        })
      },
      async showEditDialog(id){
        const {data:res} =await this.$http.get('users/'+id)
        if(res.meta.status!==200){
          return this.$message.error('查询用户信息失败')
        }
        this.editForm=res.data
        this.editDialogVisible=true
        console.log(id)
      },
      editDialogClosed(){
        this.$refs.editFormRef.resetFields()
      },
      //修改用户信息并提交
      editUserInfo(){
        this.$refs.editFormRef.validate(async valid=>{
          console.log(valid)
          if(!valid) return
          const {data:res}=await this.$http.put('users/'+this.editForm.id,{
            email:this.editForm.email,
            mobile:this.editForm.mobile
          })
          if(res.meta.status!==200){
            return this.$message.error('修改用户信息失败')
          }
          // 关闭用户对话框
          this.editDialogVisible=false
          // 更新
          this.userData()
          this.$message.success('更新用户信息成功')
        })
      },
      //根据id删除
      async removeUserById(id){
        // console.log(id)
        const confirmResult= await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
                  confirmButtonText: '确定',
                  cancelButtonText: '取消',
                  type: 'warning'
         }).catch(err=>err)
         console.log(confirmResult)
         // 取消cancel 确定confirm
         if(confirmResult!=='confirm'){
           return this.$message.info('已取消删除')
         }
         const {data:res}=await this.$http.delete('users/'+id)
         if(res.meta.status!==200){
           return this.$message.error('删除用户失败')
         }
         this.$message.success('删除用户成功')
         this.userData()
       }
    }
  }
</script>

<style lang="less" scoped>
.el-table{
  margin-top: 10px;
  font-size: 14px;
}
</style>
