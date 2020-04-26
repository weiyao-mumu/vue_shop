<template>
    <div>
      <!-- 面包屑导航区 -->
      <el-breadcrumb separator-class="el-icon-arrow-right">
  <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>用户管理</el-breadcrumb-item>
  <el-breadcrumb-item>用户列表</el-breadcrumb-item>
</el-breadcrumb>
<!-- 卡片视图 -->
 <el-Card class="card">
  <el-row :gutter="20">
    <el-col :span="7">
       <el-input placeholder="请输入内容"
       v-model="queryInfo.query" clearable
       @clear="getUserList()">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
        </el-input>
    </el-col>
    <el-col :span="3">
        <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
    </el-col>
  </el-row>
  <!-- user 列表区域 -->
  <el-table :data = "userList" border stripe>
      <el-table-column type="index"></el-table-column>
      <el-table-column label="姓名" prop='username'></el-table-column>
      <el-table-column label="邮箱" prop='email'></el-table-column>
      <el-table-column label="电话" prop='mobile'></el-table-column>
      <el-table-column label="角色" prop='role_name'></el-table-column>
      <el-table-column label="状态" prop='mg-state'>
        <!-- 插槽 -->
        <template slot-scope="scope">
         
          <el-switch v-model="scope.row.mg_state" @change="UserStateChange(scope.row)">
        
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="180px" >
        <template slot-scope="scope">
          <!-- 修改按钮 -->
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showputDialogVisible(scope.row.id)"></el-button>
            <!-- 删除按钮 -->
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>

           <el-tooltip effect="dark" content="分配角色" placement="top">
            <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
           </el-tooltip>
            
        </template>
      </el-table-column>
  </el-table>

<!-- 分页 -->
 <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[1, 2, 5, 10]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
 </el-Card>
 <!-- 添加用户对话框-->
 <el-dialog
  title="添加用户"
  :visible.sync="addDialogVisible"
  width="50%" @close="addDialogClose">
  <!-- 内容主体区域 -->
  <el-form :model="addForm" :rules="addFormRules"
   ref="addFormRef" label-width="70px"
    >
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
    <el-button type="primary" @click="addUser()">确 定</el-button>
  </span>
</el-dialog>

<!-- 修改用户对话框 -->
<el-dialog
title="修改用户信息"
  :visible.sync="putDialogVisible"
  width="50%" @close="putDialogClosed">
  <!-- 主体区域 -->
  <el-form :model="putForm" :rules="putFormRules"
   ref="putFormRef" label-width="70px"
    >
  <el-form-item label="用户名" prop="username">
    <el-input v-model="putForm.username" :disabled="true"></el-input>
  </el-form-item>
  <el-form-item label="邮箱" prop="email">
    <el-input v-model="putForm.email"></el-input>
  </el-form-item>

    <el-form-item label="手机" prop="mobile">
    <el-input v-model="putForm.mobile"></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="putDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="putUser()">确 定</el-button>
  </span>
</el-dialog>
    </div>
</template>

<script>
export default {
    data(){
      return {
        // 获取用户列表的参数对象
        queryInfo:{
          query:'',
          pagenum: 1,
          pagesize:2
        },
        userList:[],
        total:0,
        // 控制对话框的显示或隐藏
        addDialogVisible:false,
        putDialogVisible:false,
        // 添加用户的表单数据
        addForm:{
          username:'',
          password:'',
          email:'',
          mobile:''
        },
        // 查询用户储存
         putForm:{
           
         },
        // 添加表单的验证规则对象
        addFormRules:{
          username:[
          {required:true,message:'请如输入用户名',
          trigger:'blur'},
          {min:3,max:10,message:'用户名的长度在3-10个字符之间',
           trigger:'blur'}
        ],
         password:[
          {required:true,message:'请如输入密码',
          trigger:'blur'},
          {min:6,max:12,message:'用户名的长度在6-12个字符之间',
           trigger:'blur'}
        ],
         email: [
      { required: true, message: '请填写邮箱', trigger: 'blur' },
      { type: 'string',
        message: '邮箱格式不正确',
        trigger: 'blur',
        transform (value) {
          if (!/^\w+((-\w+)|(\.\w+))*@[A-Za-z0-9]+((\.|-)[A-Za-z0-9]+)*\.[A-Za-z0-9]+$/.test(value)) {
            return true
          }else{
          }
        }
           },
      { type: 'string', message: '长度不能超过30位', trigger: 'blur', max: 30 }
        ],
        mobile:[
          {required:true,message:'请如输入手机号',
          trigger:'blur'},
          {min:11,max:11,message:'手机11位,请检查',
           trigger:'blur'}
        ]
        },
        //添加修改用户规则
        putFormRules:{
           email: [
         { required: true, message: '请填写邮箱', trigger: 'blur' },
         { type: 'string',
            message: '邮箱格式不正确',
           trigger: 'blur',
           transform (value) {
             if (!/^\w+((-\w+)|(\.\w+))*@[A-Za-z0-9]+((\.|-)[A-Za-z0-9]+)*\.[A-Za-z0-9]+$/.test(value)) {
            return true
          }else{
          }
        }
           },
          { type: 'string', message: '长度不能超过30位', trigger: 'blur', max: 30 }
        ],
      
        mobile:[
          {required:true,message:'请如输入手机号',
          trigger:'blur'},
          {min:11,max:11,message:'手机11位,请检查',
           trigger:'blur'}
        ]
        }
        
      }
      
    },
    created() {
      this.getUserList()
    },
    methods: {
     async getUserList(){
        const {data:res} = await this.$http.get('users',{params:this.queryInfo
      
        })
          if(res.meta.status !==200)
          return this.$message.error("获取用户列表失败")
        this.userList = res.data.users
        this.total = res.data.total
        console.log(res)
      },
      // 监听Pagesize 改变属性
      handleSizeChange(newSize){
          this.queryInfo.pagesize = newSize
          this.getUserList()
      },
      // 页码值改变的事件
      handleCurrentChange(newPage){
      this.queryInfo.pagenum = newPage
          this.getUserList()

      },
      // 监听switch 的改变
      async UserStateChange(userinfo){
      const {data:res} = await  this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      if(res.meta.status !==200){
        userinfo.mg_state =!userinfo.mg_state
        return this.$message.error("更新用户失败")
      }
      this.$message.success('更新用户成功')
     
    },
    // 当关闭对话弹框，重置对话内容(情空，方便一次使用)
    addDialogClose(){
          this.$refs.addFormRef.resetFields()
        // if(this.addDialogVisible==false){
        //       this.addForm.username=''
        //       this.addForm.password=''
        //       this.addForm.email=''
        //       this.addForm.mobile=''
        // }
    },
    // 点击按钮，添加用户
     addUser(){
        this.$refs.addFormRef.validate(async valid=>{
          if(!valid)return
          // 可已发起用户的添加请求
       const {data:res}=  await this.$http.post('users',this.addForm)
          if(res.meta.status!==201){
            this.$message.error('添加用户失败')

          }
          this.$message.success('添加用户成功')

          // 隐藏对话框
         this.addDialogVisible = false
        //  刷新用户列表
          this.getUserList()
        })
    },
    // 显示修改对话框
    async showputDialogVisible(ID){
      //console.log(ID);
     const {data:res} = await this.$http.get('users/'+ID)
      if(res.meta.status !==200)
       return this.$message.error('查询用户失败')

       this.$message.success('查询用户成功')
        this.putForm = res.data
         this.putDialogVisible = true
    },
    // 确认修改操作
    putUser(){
      this.$refs.putFormRef.validate(async valid=>{
          if(!valid)return
          // 可已发起用户的添加请求
       const {data:res}=  await this.$http.put('users/'+this.putForm.id,{
         email:this.putForm.email,mobile:this.putForm.mobile
       })
          if(res.meta.status!==200){
            this.$message.error('修改用户失败')

          }
          this.$message.success('修改用户成功')

          // 隐藏对话框
         this.putDialogVisible = false
        //  刷新用户列表
          this.getUserList()
        })
    },
    putDialogClosed(){
        this.$refs.putFormRules.resetFields()
    },
    // 删除操作实现
     async removeUserById(id){
      // 弹框询问用户是否删除数据
      const res = await this.$confirm('此操作将永久删除该用户 是否继续?', '提示',
       {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>{
          return err
        })
        const {data: re} = await this.$http.delete('users/'+id)
        if(res !=='confirm'){
          return this.$message.info('已取消')
        }
      
         if(re.meta.status!==200){
          return this.$message.error('删除用户失败')
        }
        this.$message.success('删除用户成功')
        this.getUserList()
      
       
    }
   
    },
}
</script>


<style lang="less" scoped>

</style>