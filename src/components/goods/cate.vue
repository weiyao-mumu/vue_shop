<template>
    <div>
         <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品列表</el-breadcrumb-item>
        </el-breadcrumb>

        <!-- 卡片区域 -->
        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary" @click="showadddialog()">添加分类</el-button>
                </el-col>
            </el-row>
            <!-- 列表区域 -->
          <tree-table :data="cateList" :columns="columns" 
          :selection-type="false" :expand-type="false" show-index index-text="#" border stripe>
          <!-- 是否有效 -->
            <template slot="isok" slot-scope="scope">
                <i class="el-icon-circle-check" v-if="scope.row.cat_deleted === false"
                 style="color:green; fontSize:18px;"></i>
                <i class="el-icon-circle-close" v-else  style="color:red; fontSize:18px"></i>
            </template>
            <!-- 排序 -->
             <template slot="order" slot-scope="scope">
                   <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
                <el-tag type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
                <el-tag type="warning" v-else>三级</el-tag>
            </template>
            <!-- 操作 -->
            <template slot="opt" slot-scope="scope" width ="180px">
                <!-- 你好难找啊，卧槽 -->
                   <el-button type="success" icon="el-icon-edit" size="mini" @click="editCate(scope.row.cat_id)">编辑</el-button>
                    <el-button type="danger" icon="el-icon-edit" size="mini" @click="detectCate(scope.row.cat_id)">删除</el-button>
                   
            </template>
          </tree-table>
            <!-- 分页区域 -->
                 <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="querInfo.pagenum"
                :page-sizes="[5, 30, 50, 80]"
                :page-size="querInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
                </el-pagination>
        </el-card>
        <!-- 添加分类对话框 -->
        <el-dialog
            title="添加分类"
            :visible.sync="adddialogVisible"
            width="50%" @close="addCateDiaologClosed()">
           <!-- 添加分类的表单 -->
           <el-form :model="addcateForm" :rules="addrules" ref="addcateFormRef" label-width="100px">
            <el-form-item label="分类名称:" prop="cat_name">
                <el-input v-model="addcateForm.cat_name"></el-input>
            </el-form-item>
            <el-form-item label="父级分类:">
                  <el-cascader  expand-trigger = 'hover'
                    v-model="selectionsKeys"
                    :options="parentsCateList"
                    :props="cascaderProps"
                    @change="handleChange"  clearable change-on-select></el-cascader>
            </el-form-item>
           </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="adddialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addcate()">确 定</el-button>
            </span>
         </el-dialog>
    
    <!-- 编辑对话框 -->
     <el-dialog
            title="添加分类"
            :visible.sync="editdialogVisible"
            width="50%" @close="editCateDiaologClosed()">
           <!-- 添加分类的表单 -->
           <el-form :model="editcateForm" :rules="editrules" ref="editcateFormRef" label-width="100px">
            <el-form-item label="分类名称:" prop="cat_name">
                <el-input v-model="editcateForm.cat_name"></el-input>
            </el-form-item>
           </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editdialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editcate()">确 定</el-button>
            </span>
         </el-dialog>
  
  
   
  
    </div>
</template>

<script>
export default {
    data() {
        return {
            // 商品列表
            cateList:[],
            // 控制添加分类对话框
            adddialogVisible:false,
            // 控制编辑分类对话框
            editdialogVisible:false,
            // 分页
            querInfo:{
                type: 3,
                pagenum:1,
                pagesize:5
            },
            // 添加分类的表单数据对象
            addcateForm:{
                cat_name:'',
                // 父级分类的ID
                cat_pid:0,
                // 默认添加的是一级分类
                cat_level:0
            },
            // 数据总条数
            total:0,
            // 为table指定列的定义
            columns:[
                {label:'分类名称',prop:'cat_name'},
                {label:'是否有效',type:'template',template:'isok'},
                {label:'排序',type:'template',template:'order'},
                  {label:'操作',type:'template',template:'opt'}
                
            ],
            // 添加表单的验证规则对象
            addrules:{
                cat_name:[
                   {required: true,message:'请输入分类',trigger: 'blur'}
                ],
                cat_level:[]
            },
            // 添加编辑的验证规则对象
            editrules:{ 
            cat_name:[
                   {required: true,message:'请输入分类',trigger: 'blur'}
                ],
            },
            //父级分类的列表
            parentsCateList:[],
            // 制定级联选择器的配置对象
            cascaderProps:{
                value:'cat_id',
                label:'cat_name',
                children:'children'
            },
            // 选中的父级分类的id数组
            selectionsKeys:[],
            // 编辑数组列表
            editcateForm:{}
    }
    },
    created() {
        // 页面加载的后调用
        this.getcateList()
    },
    methods: {
        async  getcateList(){
        const {data:res} = await this.$http.get('categories',{params:this.querInfo})

            if(res.meta.status !==200)
                return this.$message.error('获取商品失败')

            this.$message.success('获取商品成功')

            this.cateList = res.data.result
            console.log(  this.cateList)
            this.total = res.data.total
            
        },
        // 监听pagesize的变化
        handleSizeChange(newSize){
            this.querInfo.pagesize = newSize
            this.getcateList()
        },
        // 监听pagenum的变化
        handleCurrentChange(newNum){
            this.querInfo.pagenum = newNum
            this.getcateList()
        },
        // 点击按钮展示添加分类的对话框
        showadddialog(){

            // 先获取父级分类的数据列表
            this.getParenteList()
            
            this.adddialogVisible = true
        },
        // 获取父级分类的数据列表
        async  getParenteList(){
         const {data:res} = await  this.$http.get('categories',{params:{type:2}})

         if(res.meta.status !==200){
             return this.$message.error('获取数据列表失败')
         }
            
            this.parentsCateList = res.data
        },
        // 选择项发生变化触发
        handleChange(){
            // 如果selectedKeys 数组中的 lenght大于0,证明选中的父及分类
            // 反之，就说明没有选中任何父级分类
            if(this.selectionsKeys.length > 0){
                // 父级分类的ID
               this.addcateForm.cat_pid = this.selectionsKeys[this.selectionsKeys.length-1]
        //    为当前分类的等级赋值
           this.addcateForm.cat_level = this.selectionsKeys.length
           return
           }else{
                  // 父级分类的ID
               this.addcateForm.cat_pid = 0
        //    为当前分类的等级赋值
           this.addcateForm.cat_level =0
           }

        },
        // 添加商品按钮
         addcate(){
        //    请求之前的预验证
            this.$refs.addcateFormRef.validate( async valid =>{
                if(!valid) return
             const {data:res} = await this.$http.post('categories',this.addcateForm)
                if(res.meta.status !==201) return this.$message.error('添加分类失败')
                this.$message.success('添加分类成功')

                this.getcateList()

                this.adddialogVisible = false

            })
          

           
        },
        // 监听对话框的关闭事件,重置表单数据
        addCateDiaologClosed(){
            this.$refs.addcateFormRef.resetFields()
            this.selectionsKeys = []
            this.addcateForm.cat_level = 0
            this.addcateForm.cat_pid = 0
        },
        // 监听对话框的关闭事件,重置表单数据
        editCateDiaologClosed(){
            this.$refs.editcateFormRef.resetFields()
    
        },
        // 编辑对话框
       async editCate(id){
           
           const {data:res} = await this.$http.get('categories/'+id)

            if(res.meta.status !==200) return this.$message.error('获取列表失败')

            this.editcateForm = res.data


            this.editdialogVisible = true
        },
        // 完成编辑调用接口
        editcate(){
            this.$refs.editcateFormRef.validate(async valid=>{
          if(!valid)return

          const {data:res} = await this.$http.put('categories/'+this.editcateForm.cat_id,{cat_name:this.editcateForm.cat_name})

          if(res.meta.status !== 200)  return this.$message.error('编辑失败')

          this.$message.success('编辑成功')

            this.getcateList()
          this.editdialogVisible = false

              })
        },
        // 点击按钮删除数据
      async  detectCate(id){
                 // 弹框询问用户是否删除数据
      const res = await this.$confirm('此操作将永久删除该分类 是否继续?', '提示',
       {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>{
          return err
        })
        if(res !=='confirm'){
          return this.$message.info('已取消')
        }
       // console.log(id)
        const {data: re} = await this.$http.delete('categories/'+id)

            if(res.meta.status !==200) return this.$message.error('删除数据失败')

            this.$message.success('删除数据成功')
            this.getcateList()
        }
      },
}
</script>

<style lang="less" scoped>
.el-row{
    margin-bottom:15px;
    
}
</style>