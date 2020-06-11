<template>
  <el-container class='home-container'>
    <!-- 头部部分 -->
    <el-header>
      <div>
        <img src="../assets/mz.png" alt="">
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 主体部分 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ?'64px' :'200px'">
        <!-- 侧边栏菜单 -->
        <div class="toggle-button" @click="toggle">|||</div>

        <el-menu background-color="#333744" text-color="#fff"
        active-text-color="#ffd04b" unique-opened :collapse="isCollapse" :collapse-transition="false" router :default-active='activation'>
          <el-submenu :index="item.id+''"  v-for="item in menulist"  :key='item.id'>
            <!-- 一级菜单 -->
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item-group v-for="subItem in item.children" :key='subItem.id'>
              <el-menu-item :index="'/'+subItem.path" @click='activation1("/"+subItem.path)'>
                <i class="el-icon-menu"></i>
              <span>{{subItem.authName}}</span></el-menu-item>
            </el-menu-item-group>
          </el-submenu>
        </el-menu>
      </el-aside>


      <!-- 右侧主体内容 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
  export default {
    data() {
      return {
        menulist: [],
        iconsObj:{
          '125':'iconfont icon-user',
          '103':'iconfont icon-tijikongjian',
          '101':'iconfont icon-shangpin',
          '102':'iconfont icon-danju',
          '145':'iconfont icon-baobiao'
        },
        isCollapse:false,
        activation:''
      }
    },
    created() {
      this.getMenuList()
      this.activation=window.sessionStorage.getItem('activation')
    },
    methods: {
      logout() {
        window.sessionStorage.clear()
        this.$router.push('/login')
      },
      async getMenuList() {
        const {data: res} = await this.$http.get('menus')
        // console.log(res)
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        this.menulist = res.data
      },
      toggle(){
        this.isCollapse=!this.isCollapse
      },
      //当用户处于激活状态时
      activation1(e){
        window.sessionStorage.setItem('activation',e)
        this.activation=e
      }
    }
  }
</script>

<style lang="less" scoped>
  .el-header {
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color: #fff;
    font-size: 20px;

    div {
      display: flex;
      align-items: center;

      span {
        margin-left: 10px;
      }
    }
  }

  .el-aside {
    background-color: #373d41;
    .el-menu{
      border-right: none;
    }
    .toggle-button{
      background-color: #4a5064;
      text-align: center;
      font-size: 10px;
      line-height: 24px;
      color: #fff;
      letter-spacing: 0.2em;
      cursor: pointer;

    }
  }

  .el-main {
    background-color: #EAEDF1;
  }

  .home-container {
    height: 100%;
  }
  .iconfont{
    margin-right: 10px;
  }
</style>
