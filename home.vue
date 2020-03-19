<template>
<el-container class= 'home-container'>
  <el-header>
      <div>
          <img src="../assets/heima.png" alt="">
          <span class="regular">黑马电商平台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button></el-header>
  <el-container>
    <el-aside :width="iscollapse? '64px' : '200px'">
      <div class="toggle-button" @click="toggle">
        |||
      </div>
        <el-menu
      background-color="#333744"
      text-color="#fff"
      active-text-color="#409fff" unique-opened :collapse="iscollapse" :collapse-transition="false" router default-active="this.$route.path">
      <!-- 一级菜单 -->
      <el-submenu :index="item.id+ ''" v-for="item in menulist" :key="item.id">
        <!--一级菜单的模板区域-->
        <template slot="title">
          <!-- 一级菜单的图标 -->
          <i :class="iconObj[item.id]"></i>
          <!-- 文本 -->
          <span>{{item.authName}}</span>
        </template>
        <!-- 二级菜单 -->
         <el-menu-item :index="subItem.id+''" v-for="subItem in item.children" :key="subItem.id" @click="drop(subItem.path)">
            <template slot="title">
          <!-- 一级菜单的图标 -->
          <i class="el-icon-menu"></i>
          <!-- 文本 -->
          <span>{{subItem.authName}}</span>
        </template>
         </el-menu-item>
      </el-submenu>
    </el-menu>
    </el-aside>
    <el-main>
      <router-view></router-view>
    </el-main>
  </el-container>
</el-container>

</template>
<script>
export default {
  data () {
    return {
      menulist: [],
      iconObj: {
        125: 'iconfont icon-users',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      iscollapse: false,
      path: ''
    }
  },
  created () {
    this.getMenuList()
    this.path = window.sessionStorage.getItem('path')
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      console.log(res)
      this.menulist = res.data
    },
    toggle () {
      this.iscollapse = !this.iscollapse
    },
    drop (path) {
      this.$router.push('/' + path)
      window.sessionStorage.setItem('path', path)
      this.path = path
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
     height: 100%;
}

.el-header {
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color: #fff;
    font-size: 20px;
    >div {
        display: flex;
        align-items: center;
        margin-left: 15px;
    }
}
.el-aside {
    background-color: #333744;
    .el-menu {
      border-right: 0;
    }
}
.el-main {
    background-color: #eaedf1;
}
.iconfont {
  margin-right: 10px;
}
.toggle-button {
  background-color: #4a5064;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2rem;
  cursor: pointer;
}
.regular {
  margin-left: 15px;
}
</style>
