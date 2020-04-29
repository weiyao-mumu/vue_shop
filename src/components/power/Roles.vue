<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>角色管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 卡片视图 -->
        <el-card>
            <!-- 添加角色按钮 -->
            <el-row>
                <el-col>
                    <el-button type="primary" @click="addDialogVisible=true">添加角色</el-button>
                </el-col>
            </el-row>
            <!-- 列表区 -->
            <el-table :data="roleList" border stripe  >
                <!-- 展开页 -->
                 <el-table-column type="expand">
                    <template slot-scope="scope">
  
                        <el-row :class="['vcenter','dbtopbox',l1 === 0 ?'dbtop':'']" 
                         v-for="(item1,l1) in scope.row.children" :key="item1.id" >
                            
                            <!-- 渲染一级权限 -->
                            <el-col :span='5'>
                                <el-tag closable @close='removeRightById(scope.row,item3.id)'>{{item1.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 渲染二级三级权限 -->
                            <el-col :span='19'>
                                <!-- 通过for循环 嵌套二级权限 -->
                                <el-row :class="['vcenter',index === 0 ?'':'dbtop']" v-for="(item2, index) 
                                in item1.children" :key="item2.id">                               
                                    <el-col :span="6" >
                                          <el-tag type="success" closable @close='removeRightById(scope.row,item2.id)'> {{item2.authName}}</el-tag>
                                           <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span ="18">
                                        <el-tag closable @close='removeRightById(scope.row,item3.id)' 
                                        type="warning" v-for="(item3, index) in item2.children" :key="item3.id">
                                            {{item3.authName}}
                                        </el-tag>
                                    </el-col>
                                    </el-row>
                            </el-col>
                        </el-row>
                        
                      
                    </template>
                 </el-table-column>
                 <!-- 索引列 -->
                 <el-table-column type="index"></el-table-column>
                 <el-table-column label="角色名称" prop="roleName"></el-table-column>
                 <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                 <el-table-column label="操作" width="290px" >
                     <template slot-scope="scope">
                         <el-button type="primary" icon="el-icon-edit" size="mini" @click="edit(scope.row.id)">编辑</el-button>
                         <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRolesById(scope.row.id)">删除</el-button>
                         <el-button type="warning" icon="el-icon-star-off" size="mini" @click="showSetDiaolog(scope.row)">分配权限</el-button>
                     </template>
                 </el-table-column>

            </el-table>
        </el-card>

        <!-- 添加角色对话框 -->
    <el-dialog  title="添加角色"
        :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed()" >
  <!-- 主体区域 -->
    <el-form :model="addForm" :rules="addFormRules"
   ref="addFormRef" label-width="80px" >
  <el-form-item label="角色名称" prop="roleName">
    <el-input v-model="addForm.roleName"></el-input>
  </el-form-item>
 <el-form-item label="角色描述" prop="roleDesc">
    <el-input v-model="addForm.roleDesc"></el-input>
 </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addRolefalse()">取 消</el-button>
    <el-button type="primary" @click="addRole()">确 定</el-button>
  </span>
</el-dialog>


<!-- 编辑角色对话框 -->
<el-dialog  title="编辑角色信息"
  :visible.sync="editDialogVisible"
  width="50%" @close="editDialogClosed">
  <!-- 主体区域 -->
  <el-form :model="editForm" :rules="editFormRules"
   ref="editFormRef" label-width="80px"
    >
  <el-form-item label="角色名称" prop="roleName">
    <el-input v-model="editForm.roleName" ></el-input>
  </el-form-item>
 <el-form-item label="角色描述" prop="roleDesc">
    <el-input v-model="editForm.roleDesc" ></el-input>
  </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editRole()">确 定</el-button>
  </span>
</el-dialog>

<!-- 分配权限的对话框 -->
<el-dialog  title="分配权限"
  :visible.sync="setDialogVisible"
  width="50%" @close="setrightclosed()">
  <!-- 主体区域 -->
  <el-tree :data="rightsList" :props="TreeProps"
  show-checkbox node-key="id" default-expand-all='true'
  :default-checked-keys="defkeys" ref="treeref">

  </el-tree>
  <span slot="footer" class="dialog-footer">
    <el-button @click="setDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="setright()">确 定</el-button>
  </span>
</el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
            //所有角色列表数据
            roleList:[],
            // 添加角色声明
            addDialogVisible:false,
            // 编辑角色声明
            editDialogVisible:false,
            // 分配权限的对话框
            setDialogVisible :false,
            addForm:{
                roleName:'',
                roleDesc:''
            },
            editForm:{

            },
            // 所有权限数据
            rightsList:[],
            // 当前即将分配权限的id
            rokeId:'',
            // 树形控件
            TreeProps:{
                label:'authName',
                children:'children'
            },
            // 默认选中的节点id值数组
            defkeys:[],
            // 添加角色的验证规则
            addFormRules:{
                roleName:[
                    {required:true,message:'请输入角色名称',trigger:'blur'},
                   
                    ],
                    roleDesc:[
                        {required:true,message:'请输入角色描述',trigger:'blur'},
                        {min:2,message:'请详细描述角色',trigger:'blur'}
                    ]
            },
            // 编辑角色的验证规则
            editFormRules:{
                roleDesc:[
                        {required:true,message:'请输入角色描述',trigger:'blur'},
                        {min:2,message:'请详细描述角色信息',trigger:'blur'}
                    ]
            }
        }
    },
    created() {
        // 获取所有角色列表数据
        this.getRoleList()
    },
    methods: {
         // 点击确定按钮添加角色
        addRole(){
             this.$refs.addFormRef.validate(async valid=>{
          if(!valid)return
          const {data : res} = await this.$http.post('roles',this.addForm)
            if(res.meta.status !==201)
                return this.$message.error('添加角色失败')
            
            this.$message.success('添加角色成功')
            // 关闭对话框
            this.addDialogVisible = false

            // 刷新用户列表
            this.getRoleList()
             })
          },
        // API通获取
     async   getRoleList(){
            const {data: res} = await this.$http.get('roles')
             
             if(res.meta.status !== 200)
                    return this.$message.error('获取角色列表失败')
           // this.$message.success('获取角色列表成功')
            this.roleList = res.data
           // console.log(this.roleList)
        },
        // 关闭添加角色对话框重置
        addDialogClosed(){
            this.$refs.addFormRef.resetFields()
        },
        // 关闭编辑角色对话框重置
        editDialogClosed(){
            this.$refs.editFormRef.resetFields()
        },
        //  添加角色点击取消
        addRolefalse(){
             this.$message.error('您以成功取消')
             this.addDialogVisible = false
        },
        // 点击编辑按钮弹出对话框
         async edit(id){
            const {data:res} = await this.$http.get('roles/'+id)
            if(res.meta.status !== 200)
                return this.$message.error('获取编辑失败')
            this.$message.success('获取编辑成功')
            // 将数据放入指定的本地数组中
            this.editForm = res.data
            // 打开编辑对话框

            this.editDialogVisible = true
        },
        // 点击编辑确定修改按钮
        editRole(){
            this.$refs.editFormRef.validate(async valid=>{
                if(!valid) return
            // 如果可以就发起编辑用户API请求
            const {data:res} = await this.$http.put('roles/'+this.editForm.id,{
                roleName:this.editForm.roleName,roleDesc:this.editForm.roleDesc
                })
            if(res.meta.status !==200){
             this.$message.error('编辑角色失败')
            }
               

            this.$message.success('编辑角色成功')
            // 关闭对话框
            this.editDialogVisible =false
            // 刷新角色列表
            this.getRoleList()
            })
        },
        // 删除角色操作
        async removeRolesById(id){
               // 弹框询问用户是否删除数据
      const res = await this.$confirm('此操作将永久删除该角色 是否继续?', '提示',
       {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>{
          return err
        })
        const {data: re} = await this.$http.delete('roles/'+id)
        if(res !=='confirm'){
          return this.$message.info('已取消')
        }
      
         if(re.meta.status!==200){
          return this.$message.error('删除用户失败')
        }
        this.$message.success('删除用户成功')
        this.getRoleList()
        },
        // 根据id删除对应的权限
        async removeRightById(role,id){
            // 弹框提示用户是否删除
            const res = await this.$confirm('此操作将永久删除该角色 是否继续?', '提示',
       {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>{
          return err
        })
        const {data: re} = await this.$http.delete(`roles/${role.id}/rights/${id}`)
        if(res !=='confirm'){
          return this.$message.info('已取消')
        }
        if(re.meta.status !==200)
            return this.$message.error('删除权限失败')
        this.$message.success('删除权限成功')

            role.children =re.data
        
        },
        // 分配权限对话框的实现
       async showSetDiaolog(role){
           this.roleId = role.id
            // 获取所有权限列表
            const {data:res} = await this.$http.get('rights/tree')
          if(res.meta.status !==200) 
          return this.$message.error('获取权限失败')

        this.$message.success('获取权限成功')

        this.rightsList =res.data
        //console.log(this.roleList)
        //   递归获取三级节点的id
            this.getrefkeys(role,this.defkeys)
          this.setDialogVisible = true
        },
// 通过递归的方法，获取角色下的所有权限，并保存到defkeys数组中
        getrefkeys(node,arr){
            if(!node.children)
            // 如果node节点不包含children树形
            return arr.push(node.id)

            node.children.forEach(item=>{
                this.getrefkeys(item,arr)
            })
        },
        // 监听分配权限的
        setrightclosed(){
            this.defkeys = []
        },
        // 分配权限函数确定按钮
         async setright(){
            const keys =[

                ...this.$refs.treeref.getCheckedKeys(),
               ...this.$refs.treeref.getHalfCheckedKeys()
            ]
           const idStr = keys.join(',')

          const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
            if(res.meta.status !==200)
                return this.$message.error('分配权限失败')
             this.$message.success('分配权限成功')

             this.getRoleList()

            this.setDialogVisible = false
        }
        
    }
}
</script>

<style lang="less" scoped>
    .el-tag{
        margin: 7px;
    }
    .dbtop{
        border-top:1px solid #ccc;
    }
    .dbtopbox{
        border-bottom: 1px solid #ccc;
    }
    .vcenter{
        display: flex;
        align-items: center;
    }
</style>