<template>
    <el-row class="container">
        <el-col :span="24" class="header">
            <!-- 左上角店铺名称 -->
            <el-col :span="10" :class="collapsed?'logo-collapse-width logo':'logo-width logo'">
                <span v-show="collapsed"></span>
                <span v-show="!collapsed">{{shop.name}}</span>
            </el-col>
            <!-- 收缩左侧菜单栏标志 -->
            <el-col :span="10">
                <div class="tools" @click.prevent="collapse">
                    <i :class="collapsed?'el-icon-d-arrow-right':'el-icon-d-arrow-left'"></i>
                </div>
            </el-col>
            <!-- 右侧顶部退出登录 logo -->
            <el-col :span="4" class="userinfo">
                <el-dropdown trigger="hover">
                    <span class="el-dropdown-link userinfo-inner">
                        <img :src="this.sysUserAvatar">
                        {{spaAccount.name}}
                    </span>
                    <el-dropdown-menu slot="dropdown">
                        <!-- <el-dropdown-item>我的消息</el-dropdown-item> -->
                        <el-dropdown-item>设置</el-dropdown-item>
                        <el-dropdown-item divided @click.native="logout">退出登录</el-dropdown-item>
                    </el-dropdown-menu>
                </el-dropdown>
            </el-col>
        </el-col>
        <el-col :span="24" class="main">
            <aside :class="collapsed?'menu-collapsed':'menu-expanded'">
                <!--导航菜单-->
                <el-menu
                    :default-active="defaultActive"
                    class="leftMenu expanded"
                    unique-opened
                    router
                    v-show="!collapsed"
                >
                    <template v-for="(item,index) in $router.options.routes" v-if="!item.hidden">
                        <el-submenu :index="index+''" v-if="!item.leaf" :key="index">
                            <template slot="title">
                                <i :class="item.iconCls"></i>
                                {{item.name}}
                            </template>
                            <el-menu-item
                                v-for="child in item.children"
                                :index="child.path"
                                :key="child.path"
                                v-if="!child.hidden"
                            >{{child.name}}</el-menu-item>
                        </el-submenu>
                        <el-menu-item
                            v-if="item.leaf&&item.children.length>0"
                            :index="item.children[0].path"
                            :key="index"
                        >
                            <i :class="item.iconCls"></i>
                            {{item.children[0].name}}
                        </el-menu-item>
                    </template>
                </el-menu>
                <!--导航菜单-折叠后-->
                <ul class="el-menu leftMenu collapsed" v-show="collapsed" ref="menuCollapsed">
                    <li
                        v-for="(item,index) in $router.options.routes"
                        v-if="!item.hidden"
                        class="el-submenu item"
                        :key="index"
                    >
                        <template v-if="!item.leaf">
                            <div
                                class="el-submenu__title"
                                style="padding-left: 20px;"
                                @mouseover="showMenu(index,true)"
                                @mouseout="showMenu(index,false)"
                            >
                                <i :class="item.iconCls"></i>
                            </div>
                            <ul
                                class="el-menu submenu"
                                :class="'submenu-hook-'+index"
                                @mouseover="showMenu(index,true)"
                                @mouseout="showMenu(index,false)"
                            >
                                <li
                                    v-for="child in item.children"
                                    v-if="!child.hidden"
                                    :key="child.path"
                                    class="el-menu-item"
                                    style="padding-left: 40px;"
                                    :class="$route.path==child.path?'is-active':''"
                                    @click="$router.push(child.path)"
                                >{{child.name}}</li>
                            </ul>
                        </template>
                        <template v-else>
                            <li class="el-submenu">
                                <div
                                    class="el-submenu__title el-menu-item"
                                    style="padding-left: 20px;height: 56px;line-height: 56px;padding: 0 20px;"
                                    :class="$route.path==item.children[0].path?'is-active':''"
                                    @click="$router.push(item.children[0].path)"
                                >
                                    <i :class="item.iconCls"></i>
                                </div>
                            </li>
                        </template>
                    </li>
                </ul>
            </aside>
            <section class="content-container">
                <div class="grid-content bg-purple-light">
                    <el-col :span="24" class="breadcrumb-container">
                        <strong class="title">{{$route.name}}</strong>
                        <el-breadcrumb separator="/" class="breadcrumb-inner">
                            <el-breadcrumb-item
                                v-for="item in $route.matched"
                                :key="item.path"
                            >{{ item.name }}</el-breadcrumb-item>
                        </el-breadcrumb>
                    </el-col>
                    <el-col :span="24" class="content-wrapper">
                        <transition name="fade" mode="out-in">
                            <router-view></router-view>
                        </transition>
                    </el-col>
                </div>
            </section>
        </el-col>
    </el-row>
</template>

<script>
import Api from "@/common/api/api.js";
export default {
  data() {
    return {
      // 门店信息
      shop: {
        name: "盛传新零售"
      },
      // 默认选择
      defaultActive: "/GoodsTable",
      collapsed: false, //左侧导航栏折叠还是展开
      spaAccount: {
        name: "李建"
      },
      sysUserAvatar: ""
    };
  },
  beforeMount() {
    if (
      this.$router.history &&
      this.$router.history.current &&
      this.$router.history.current.path
    ) {
      this.defaultActive = this.$router.history.current.path;
    }
    this.defaultActive = this.defaultActive || "/GoodsTable";
  },
  methods: {
    //退出登录
    logout: function() {
      var that = this;
      this.$confirm("确认退出吗?", "提示", {})
        .then(() => {
          that.goLogin();
        })
        .catch(() => {});
    },
    // 退出登录，跳转页面
    goLogin: function() {
      Api.loginOut({}, () => {
        this.$store.commit("setSpaAccount", {});
        sessionStorage.clear();
        this.$router.push({ path: "/Login" });
      });
    },
    //折叠导航栏
    collapse: function() {
      this.collapsed = !this.collapsed;
    },
    showMenu(i, status) {
      this.$refs.menuCollapsed.getElementsByClassName(
        "submenu-hook-" + i
      )[0].style.display = status ? "block" : "none";
    }
  },
  mounted() {}
};
</script>

<style scoped lang="less">
.container {
  position: absolute;
  top: 0px;
  bottom: 0px;
  width: 100%;
  .header {
    height: 60px;
    line-height: 60px;
    background: @color-primary;
    color: #fff;
    .userinfo {
      text-align: right;
      padding-right: 35px;
      float: right;
      .userinfo-inner {
        cursor: pointer;
        color: #fff;
        img {
          width: 40px;
          height: 40px;
          border-radius: 20px;
          margin: 10px 0px 10px 10px;
          float: right;
        }
      }
    }
    .logo {
      height: 60px;
      font-size: 22px;
      padding-left: 20px;
      padding-right: 20px;
      border-color: rgba(238, 241, 146, 0.3);
      border-right-width: 1px;
      border-right-style: solid;
      img {
        width: 40px;
        float: left;
        margin: 10px 10px 10px 18px;
      }
      .txt {
        color: #fff;
      }
    }
    .logo-width {
      width: 230px;
    }
    .logo-collapse-width {
      width: 60px;
    }
    .tools {
      padding: 0px 23px;
      width: 14px;
      height: 60px;
      line-height: 60px;
      cursor: pointer;
    }
  }
  .main {
    display: flex;
    position: absolute;
    top: 60px;
    bottom: 0px;
    overflow: hidden;
    aside {
      flex: 0 0 230px;
      width: 230px;
      .el-menu {
        height: 100%;
      }
      .collapsed {
        width: 60px;
        .item {
          position: relative;
        }
        .submenu {
          position: absolute;
          top: 0px;
          left: 60px;
          z-index: 99999;
          height: auto;
          display: none;
        }
      }
      .expanded {
        width: 230px !important;
      }
    }
    .menu-collapsed {
      flex: 0 0 60px;
      width: 60px;
    }
    .menu-expanded {
      flex: 0 0 230px;
      width: 230px;
    }
    .content-container {
      flex: 1;
      overflow-y: scroll;
      padding: 20px;
      .breadcrumb-container {
        .title {
          width: 200px;
          float: left;
          color: #475669;
        }
        .breadcrumb-inner {
          float: right;
        }
      }
      .content-wrapper {
        background-color: #fff;
        box-sizing: border-box;
      }
    }
  }
}
</style>