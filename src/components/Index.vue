<template>
  <el-container id="index">
    <el-header>
      <div class="logo">
        <img src="../assets/logo.png" alt />
      </div>
      <div class="title">
        <h1>电商后台管理系统</h1>
      </div>
      <div class="logout">
        欢迎光临~
        <a @click="logout" href="javascript:void(0)">退出</a>
      </div>
    </el-header>
    <el-container>
      <el-aside width='200px'>
        <el-menu
          :default-active='defaultActive'
          router
          unique-opened
          background-color="#545c64"
          text-color="#fff"
          active-text-color="#ffd04b"
        >
          <el-submenu :index="menu.path" v-for="menu in menuList" :key="menu.id">
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>{{ menu.authName }}</span>
            </template>
            <el-menu-item :index="childmenu.path" v-for='childmenu in menu.children' :key = 'childmenu.id'>
              <i class="el-icon-menu"></i>
              <span slot="title">{{ childmenu.authName }}</span>
            </el-menu-item>
          </el-submenu>
          <!-- <el-submenu index="2">
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>权限管理</span>
            </template>
            <el-menu-item index="roles">
              <i class="el-icon-menu"></i>
              <span slot="title">角色列表</span>
            </el-menu-item>
            <el-menu-item index="rights">
              <i class="el-icon-menu"></i>
              <span slot="title">权限列表</span>
            </el-menu-item>
          </el-submenu> -->
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
  computed: {
    defaultActive () {
      return this.$route.path.slice(1).split('-')[0]
    }
  },
  data () {
    return {
      menuList: []
    }
  },
  async created () {
    const { meta, data } = await this.$axios.get('menus')
    if (meta.status === 200) {
      this.menuList = data
    } else {
      this.$message.error(meta.msg)
    }
  },
  methods: {
    logout () {
      this.$confirm('你确认要退出吗？', '温馨提示', {
        type: 'warning'
      })
        .then(() => {
          this.$message.success('退出成功')
          localStorage.removeItem('token')
          this.$router.push('/login')
        })
        .catch(() => {
          this.$message.info('退出取消')
        })
    }
  }
}
</script>

<style lang='scss' scoped>
#index {
  height: 100%;
}
.el-header {
  height: 60px;
  line-height: 60px;
  background-color: #d8d8d8;
  display: flex;
  text-align: center;
  .logo {
    width: 180px;
    img {
      height: 40px;
      margin-top: 10px;
    }
  }
  .title {
    flex: 1;
  }
  .logout {
    width: 180px;
    text-align: right;
    a {
      text-decoration: none;
      color: #f99;
    }
  }
}
.el-aside {
  background-color: #545c64;
  width: 200px;
  height: 100%;
  .el-menu {
    border: none;
  }
}
.el-main {
  background-color: #ecf0f1;
}
</style>
