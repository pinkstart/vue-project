<template>
  <el-container class="home-container">
      <!-- 头部区域 -->
        <el-header>
            <div>
                <img src="../assets/heima.png" alt="">
                <span>电商后台管理系统</span>
            </div>
            <el-button type="info" @click="logout">退出</el-button>
        </el-header>
        <!-- 页面主体区域 -->
        <el-container>
            <!-- 侧边栏 -->
            <el-aside width="200px"> 
                   <el-menu
                        background-color="#333744"
                        text-color="#fff"
                        active-text-color="#ffd04b">
                        <!-- 一级菜单 -->
                        <el-submenu index="1">
                            <!-- 一级菜单的模板区域 -->
                            <template slot="title">
                                <!-- 图标 -->
                            <i class="el-icon-location"></i>
                            <!-- 文本 -->
                            <span>导航一</span>
                            </template>
                            <!-- 二级菜单 -->
                            <el-menu-item index="1-4-1">
                            <template slot="title">
                               <!-- 图标 -->
                            <i class="el-icon-location"></i>
                            <!-- 文本 -->
                            <span>选项一</span>
                            </template>
                            </el-menu-item>
                        </el-submenu>
                   </el-menu>
            </el-aside>
            <!-- 右侧主体内容 -->
            <el-main>Main</el-main>
        </el-container>
    </el-container>

</template>

<script>
export default {
  data() {
   return {
    //    左侧菜单数据
       menulist: []
   }
  },

  created(){
      this.getMenuList()

      
  },

  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push("/login")
    },
    // 获取所有的菜单
    async getMenuList(){
        const{data: res} = await this.$http.get('menus')
        if(res.meta.status !== 200) return this.$message.error(res.meta.msg)
        this.menulist = res.data
        console.log(res);    
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
    > div {
        display: flex;
        align-items: center;
        span {
            margin-left: 15px;
        }
    }
}
.el-aside {
    background-color: #333744;
}
.el-main {
    background-color: #eaedf1;
}
.home-container {
    height: 100%;
}
</style>
