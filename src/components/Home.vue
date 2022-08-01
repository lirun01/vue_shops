<template>
  <el-container class="home-container">
    <el-header height="80px">
      <div>
        <img
          src="../assets/heima.png"
          alt=""
          style="width: 70px; height: 70px"
        />
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <el-aside :width="istoggle ? '56px' : '240px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu
          :default-active="activePath"
          class="el-menu-vertical-demo"
          background-color="#545c64"
          text-color="#fff"
          active-text-color="#409BFF"
          unique-opened
          :collapse="istoggle"
          :collapse-transition="false"
          router
        >
          <el-submenu
            :index="item.id + ''"
            v-for="item in menuList"
            :key="item.id"
          >
            <template slot="title">
              <i :class="iconsObj[item.id]" style="font-size: 15px"></i>
              <span style="font-size: 15px">{{ item.authName }}</span>
            </template>

            <el-menu-item
              :index="'/' + items.path"
              v-for="items in item.children"
              :key="items.id"
              @click="saveNavState('/' + items.path)"
            >
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{ items.authName }}</span>
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
  created() {
    this.getMenuList();
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  data() {
    return {
      menuList: [],
      iconsObj: {
        125: "iconfont icon-user",
        103: "iconfont icon-tijikongjian",
        101: "iconfont icon-shangpin",
        102: "iconfont icon-danju",
        145: "iconfont icon-baobiao",
      },
      istoggle: false,
      activePath:''
    };
  },
  methods: {
    logout() {
      window.sessionStorage.clear();
      this.$router.push("/login");
    },
    getMenuList() {
      this.$http.get("menus").then((res) => {
        // console.log(res.data);
        if (res.data.meta.status !== 200)
          return this.$message.error(res.data.meta.msg);
        this.menuList = res.data.data;
      });
    },
    toggleCollapse() {
      this.istoggle = !this.istoggle;
    },
    saveNavState(activePath){
      window.sessionStorage.setItem('activePath',activePath)
      this.activePath = activePath
    }
  },
};
</script>

<style lang="less" scoped>
.home-container {
  height: 100vh;
}
.el-header {
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #fff;
  font-size: 30px;
  div {
    display: flex;
    align-items: center;
    span {
      margin-left: 25px;
    }
  }
}
.el-aside {
  background-color: #333743;
  .tac {
    width: 100%;
    height: 100%;
  }
  .el-menu {
    border-right: none;
  }
}
.el-main {
  background-color: #eaedf1;
  height: 100%;
}
.iconfont {
  margin-right: 20px;
}
.toggle-button {
  background-color: #4a5064;
  color: #fff;
  font-size: 18px;
  text-align: center;
  line-height: 35px;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>