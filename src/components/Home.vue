<template>
<el-container class="home-container">
  <!-- 头部区域 -->
  <el-header>
    <div>
      <img src="../assets/QQ图片20200424001520.gif" alt="">
      <div><span>电商后台管理系统</span></div>
    </div>
    <el-button type="info" @click="logout">退出</el-button>
  </el-header>
  <!-- 页面区域 -->
  <el-container>
    <!-- 侧边栏 -->
    <el-aside  :width="isCollapse ? '64px': '200px'">
      <div class="toggle-button" @click="toggleButton" title="折叠">|||</div>
      <!-- 菜单 -->
     <el-menu
      background-color="#333744"
      text-color="#fff"
      active-text-color="#409EFF"
      unique-opened
      :collapse="isCollapse"
      :collapse-transition="false"
      :router="true"
      :default-active='activepath'>
      
      <!-- 一级菜单 -->
      <el-submenu   :index="item.id+' '" v-for=" item in menulist" :key="item.id">
        <!-- 一级菜单模板 -->
        <template slot="title">
          <!-- 图标 -->
          <i :class="iconObj[item.id]"></i>
          <!-- 文本 -->
          <span>{{item.authName}}</span>
        </template>
        <!-- 二级菜单 -->
            <el-menu-item :index="'/'+items.path" v-for="items in item.children" :key="items.id"
            @click="saveNavState('/'+items.path)">
              <template slot="title">
          <!-- 图标 -->
          <i class="el-icon-menu"></i>
          <!-- 文本 -->
          <span>{{items.authName}}</span>
        </template>
            </el-menu-item>
      </el-submenu>
    </el-menu>  
    </el-aside>
    <!-- 右侧主题 -->
    <el-main>
      <!-- 路由占位符 -->
      <router-view ></router-view>
    </el-main>
  </el-container>
</el-container>
</template>

<script>
export default {
  data () {
    return {
          //左侧菜单数据
          menulist:[],
          iconObj:{
            '125': 'iconfont icon-users',
            '103': 'iconfont icon-tijikongjian',
            '101': 'iconfont icon-shangpin',
            '102': 'iconfont icon-danju',
            '145': 'iconfont icon-baobiao'

          },
          // 是否折叠
          isCollapse:false,
          width:200,
          activepath:''

    }
  },
   created() {
      this.getMenuList()
      this.activepath = window.sessionStorage.getItem('activepath')
    },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList(){
     const {data:res} = await this.$http.get('menus')
     if(res.meta.status !=200) return this.$message.error(res.meta.msg)
     this.menulist = res.data
     console.log(res)
    },
    // 点击按钮折叠菜单
    toggleButton(){
        this.isCollapse= ! this.isCollapse
    },
    // 保存连接的激活状态
    saveNavState(activepath){
      window.sessionStorage.setItem('activepath',activepath)
      this.activepath =activepath
    }
  }
}
</script>

<style lang="less" scoped>
.home-container{
  height: 100%;
}
.el-header{
  background-color: #373D41;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: #fff;
  font-size: 20px;
  >div{
    display:flex;
    align-items: center;
 
    >img{
      width: 61px;
      height: 56px;
      border-radius: 50%;
    }
    >div{
      margin-left: 10px;
    }
  }
}
.el-aside{
  background-color: #333744;
  .el-menu{
    border: none;
  }
}
el-main{
  background-color: #eaedf1;
}
.iconfont{
  margin-right: 10px;
}
.toggle-button{
  background-color: #4A5064;  
  color: #fff;
  font-size: 10px;
  line-height: 24px;
  text-align: center;
  letter-spacing: .2em;
  cursor: pointer;
}
</style>
