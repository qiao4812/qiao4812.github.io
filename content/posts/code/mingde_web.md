---
title: "明德项目之前端搭建"
date: 2023-04-22T22:17:45+08:00
draft: true
tags: ["项目", "Python"]
categories: ["项目", "Python"]
---

# 明德前端项目

## 一、搭建前端项目

### 搭建 Vite 项目

- Vite 官网 <https://cn.vitejs.dev/guide/>

使用 NPM:

```bash
$ npm create vite@latest
```

使用 Yarn:

```bash
$ yarn create vite
```

使用 PNPM:

```bash
$ pnpm create vite
```

然后按照提示操作即可！

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304222224294.png)

运行项目

```bash
cd mingdeweb
pnpm install
pnpm run dev
```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304222230998.png)

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304222232673.png)

PyCharm 运行项目

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304231011512.png)

访问：http://localhost:5173/ 效果：

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304231014367.png)

把vite根目录下的`.gitignore`文件内容，复制补充路径，粘贴到父级目录mingde下的`.gitignore`文件中。

提交版本

```bash
git add .
git commit -m "客户端你搭建"
git push origin main
```

## 二、初始化前端项目

### 删除src/APP.vue中的默认样式和内容

```vue
<script setup lang="ts">
</script>

<template>

</template>

<style scoped>

</style>

```

### 安装路由组件vue-router

- 下载路由组件

```bash
cd mingdeweb
yarn add vue-router@next
```

中文文档：https://next.router.vuejs.org/zh/

###  配置路由

- 初始化路由对象

创建`src/router/index.ts`，代码：

```tsx
import {createRouter, createWebHistory, RouteRecordRaw} from 'vue-router'

// 路由列表
const routes: Array<RouteRecordRaw> = [

]

// 路由对象实例化
const router = createRouter({
  // history, 指定路由的模式
  history: createWebHistory(),
  // 路由列表
  routes,
});


// 暴露路由对象
export default router

```

- 注册路由组件

在main.ts文件，把router对象注册到vue项目中，代码：

```tsx
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'
import router from "./router";

createApp(App).use(router).mount('#app')

```

- 在视图中显示路由对应的内容

在App.vue组件中，添加显示路由对应的内容。代码：

```vue
<template>
  <router-view></router-view>
</template>

<script setup lang="ts">

</script>

<style scoped>

</style>

```

提交版本

```bash
git add .
git commit -m "前端初始化并安装集成vue-router"
git push
```

### 创建并提供前端首页和登陆的组件

routers/index.ts

```tsx
import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router'

// 路由列表
const routes: Array<RouteRecordRaw> = [
  {
    meta: {
      title: "明德在线教育-首页",
      keepAlive: true
    },
    path: '/',         // uri访问地址
    name: "Home",
    component: () => import("../views/Home.vue")
  },
  {
    meta: {
      title: "明德在线教育-用户登录",
      keepAlive: true
    },
    path: '/login',      // uri访问地址
    name: "Login",
    component: () => import("../views/Login.vue")
  }
]

// 路由对象实例化
const router = createRouter({
  // history, 指定路由的模式
  history: createWebHistory(),
  // 路由列表
  routes,
});


// 暴露路由对象
export default router

```

##### 创建Home组件

views/Home.vue，代码：

```vue
<template>
    <h1>首页</h1>
</template>

<script setup lang="ts">

</script>

<style scoped>
</style>

```

##### 创建Login组件

views/Login.vue，代码：

```vue
<template>
    <h1>登录</h1>
</template>
  
<script setup lang="ts">

</script>
  
<style scoped>
</style>

```

## 三、引入element-plus

官方文档：<https://element-plus.org/#/zh-CN>

### 安装

```bash
# 选择一个你喜欢的包管理器

# NPM
npm install element-plus --save

# Yarn
yarn add element-plus

# pnpm
pnpm install element-plus
```

### 按需导入[#](https://element-plus.gitee.io/zh-CN/guide/quickstart.html#按需导入) 

首先你需要安装`unplugin-vue-components` 和 `unplugin-auto-import`这两款插件

```
npm install -D unplugin-vue-components unplugin-auto-import
```

vite配置文件， mingdeweb/vite.config.ts，加载上面刚安装的导入插件，代码：

```tsx
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    Components({
      resolvers: [ElementPlusResolver()],
    }),
  ],
})

```

Home组件中，调用elementUI的基本样式，测试下是否成功引入。Home.vue，代码：

```vue
<template>
    <h1>首页</h1>
    <el-row>
        <el-button round>圆角按钮</el-button>
        <el-button type="primary" round>主要按钮</el-button>
        <el-button type="success" round>成功按钮</el-button>
        <el-button type="info" round>信息按钮</el-button>
        <el-button type="warning" round>警告按钮</el-button>
        <el-button type="danger" round>危险按钮</el-button>
        <el-rate v-model="store.value2" :colors="store.colors"> </el-rate>
    </el-row>
</template>

<script setup lang="ts">
import { reactive } from "vue";
const store = reactive({
    value2: null,
    colors: ['#99A9BF', '#F7BA2A', '#FF9900'],
})
</script>

<style scoped>
</style>

```

访问浏览器，效果：

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304241341076.png)

提交版本

```bash
git add .
git commit -m "客户端按需加载element-plus前端组件"
git push
```

## 四、全局css初始化

全局css初始化代码，防止出现浏览器之间的怪异bug和避免外观不一致。

src/App.vue，代码：

```vue
<template>
  <router-view></router-view>
</template>

<script setup lang="ts">

</script>

<style scoped>
/* 声明全局样式和项目的初始化样式 */
body,h1,h2,h3,h4,p,table,tr,td,ul,li,a,form,input,select,option,textarea{
  margin:0;
  padding: 0;
  font-size: 15px;
}
a{
  text-decoration: none;
  color: #333;
  cursor: pointer;
}
ul,li{
  list-style: none;
}
table{
  border-collapse: collapse; /* 合并边框 */
}
img{
  max-width: 100%;
  max-height: 100%;
}
input{
  outline: none;
}
</style>

```

提交版本

```bash
git add .
git commit -m "客户端全局初始化css样式"
git push 
```

## 五、显示首页

views/Home.vue中添加代码：

```vue
<template>
    <div class="home">
        <Header></Header>

        <Footer></Footer>
    </div>
</template>

<script setup lang="ts">
import Header from "../components/Header.vue"
import Footer from "../components/Footer.vue"

</script>

<style scoped>
</style>

```

components/Header.vue

```vue
<template>
  <div class="header-box">
    <div class="header">
      <div class="content">
        <div class="logo">
          <router-link to="/"><img src="../assets/logo.png" alt=""></router-link>
        </div>
        <ul class="nav">
            <li><router-link to="">免费课</router-link></li>
            <li><router-link to="">项目课</router-link></li>
            <li><router-link to="">学位课</router-link></li>
            <li><router-link to="">习题库</router-link></li>
            <li><router-link to="">明德教育</router-link></li>
        </ul>
        <div class="search-warp">
          <div class="search-area">
            <input class="search-input" placeholder="请输入关键字..." type="text" autocomplete="off">
            <div class="hotTags">
              <router-link to="/search/?words=Vue" target="_blank" class="">Vue</router-link>
              <router-link to="/search/?words=Python" target="_blank" class="last">Python</router-link>
            </div>
          </div>
          <div class="showhide-search" data-show="no"><img class="imv2-search2" src="../assets/search.svg" /></div>
        </div>
        <div class="login-bar">
          <div class="shop-cart full-left">
            <img src="../assets/cart.svg" alt="" />
            <span><router-link to="/cart">购物车</router-link></span>
          </div>
          <div class="login-box full-left">
            <span>登录</span>
            &nbsp;/&nbsp;
            <span>注册</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<script setup lang="ts">

</script>

<style scoped>
.header-box{
height: 72px;
}
.header{
width: 100%;
height: 72px;
box-shadow: 0 0.5px 0.5px 0 #c9c9c9;
position: fixed;
top:0;
left: 0;
right:0;
margin: auto;
z-index: 99;
background: #fff;
}
.header .content{
max-width: 1366px;
width: 100%;
margin: 0 auto;
}
/* .header .content .logo a{
  
} */
.header .content .logo{
height: 72px;
line-height: 72px;
margin: 0 20px;
float: left;
cursor: pointer; /* 设置光标的形状为爪子 */
}
.header .content .logo img{
vertical-align: middle;
margin: -40px;
}
.header .nav li{
float: left;
height: 80px;
line-height: 80px;
margin-right: 30px;
font-size: 16px;
color: #4a4a4a;
cursor: pointer;
}
.header .nav li span{
padding-bottom: 16px;
padding-left: 5px;
padding-right: 5px;
}
.header .nav li span a{
display: inline-block;
}
.header .nav li .this{
color: #4a4a4a;
border-bottom: 4px solid #ffc210;
}
.header .nav li:hover span{
color: #000;
}

/*首页导航全局搜索*/
.search-warp {
position: relative;
float: left;
margin-left: 24px;
}
.search-warp .showhide-search {
width: 20px;
height: 24px;
text-align: right;
position: absolute;
display: inline-block;
right: 0;
bottom: 24px;
padding: 0 8px;
border-radius: 18px;
}
.search-warp .showhide-search i {
display: block;
height: 24px;
color: #545C63;
cursor: pointer;
font-size: 18px;
line-height: 24px;
width: 20px;
}
.search-area {
float: right;
position: relative;
height: 40px;
padding-right: 36px;
border-bottom: 1px solid rgba(255, 255, 255, 0.4);
zoom: 1;
background: #F3F5F6;
border-radius: 4px;
margin: 16px 0;
width: 324px;
box-sizing: border-box;
font-size: 0;
-webkit-transition: width 0.3s;
-moz-transition: width 0.3s;
transition: width 0.3s;
}
.search-area .search-input {
padding: 8px 12px;
font-size: 14px;
color: #9199A1;
line-height: 24px;
height: 40px;
width: 100%;
float: left;
border: 0;
-webkit-transition: background-color 0.3s;
-moz-transition: background-color 0.3s;
transition: background-color 0.3s;
background-color: transparent;
-moz-box-sizing: border-box;
-webkit-box-sizing: border-box;
-ms-box-sizing: border-box;
box-sizing: border-box;
}
.search-area .search-input.w100 {
width: 100%;
}
.search-area .hotTags {
display: inline-block;
position: absolute;
top: 0;
right: 32px;
}
.search-area .hotTags a {
display: inline-block;
padding: 4px 8px;
height: 16px;
font-size: 14px;
color: #9199A1;
line-height: 16px;
margin-top: 8px;
max-width: 60px;
overflow: hidden;
text-overflow: ellipsis;
white-space: nowrap;
}
.search-area .hotTags a:hover {
color: #F21F1F;
}
.search-area input::-webkit-input-placeholder {
color: #A6A6A6;
}
.search-area input::-moz-placeholder {
/* Mozilla Firefox 19+ */
color: #A6A6A6;
}
.search-area input:-moz-placeholder {
/* Mozilla Firefox 4 to 18 */
color: #A6A6A6;
}
.search-area input:-ms-input-placeholder {
/* Internet Explorer 10-11 */
color: #A6A6A6;
}
.search-area .btn_search {
float: left;
cursor: pointer;
width: 30px;
height: 38px;
text-align: center;
-webkit-transition: background-color 0.3s;
-moz-transition: background-color 0.3s;
transition: background-color 0.3s;
}
.search-area .search-area-result {
position: absolute;
left: 0;
top: 57px;
width: 300px;
margin-bottom: 20px;
border-top: none;
background-color: #fff;
box-shadow: 0 8px 16px 0 rgba(7, 17, 27, 0.2);
font-size: 12px;
overflow: hidden;
display: none;
z-index: 800;
border-bottom-right-radius: 8px;
border-bottom-left-radius: 8px;
}
.search-area .search-area-result.hot-hide {
top: 47px;
}
.search-area .search-area-result.hot-hide .hot {
display: none;
}
.search-area .search-area-result.hot-hide .history {
border-top: 0;
}
.search-area .search-area-result h2 {
font-size: 12px;
color: #1c1f21;
line-height: 12px;
margin-bottom: 8px;
font-weight: 700;
}
.search-area .search-area-result .hot {
padding: 12px 0 8px 12px;
box-sizing: border-box;
}
.search-area .search-area-result .hot .hot-item {
background: rgba(84, 92, 99, 0.1);
border-radius: 12px;
padding: 4px 12px;
line-height: 16px;
margin-right: 4px;
margin-bottom: 4px;
display: inline-block;
cursor: pointer;
font-size: 12px;
color: #545c63;
}
.search-area .search-area-result .history {
border-top: 1px solid rgba(28, 31, 33, 0.1);
box-sizing: border-box;
}
.search-area .search-area-result .history li {
height: 40px;
line-height: 40px;
padding: 0 10px;
overflow: hidden;
text-overflow: ellipsis;
white-space: nowrap;
color: #787d82;
cursor: pointer;
}
.search-area .search-area-result .history li:hover,
.search-area .search-area-result .history li .light {
color: #1c1f21;
background-color: #edf0f2;
}


.header .login-bar{
margin-top: 20px;
height: 80px;
float: right;
}
.header .login-bar .shop-cart{
float: left;
margin-right: 20px;
border-radius: 17px;
background: #f7f7f7;
cursor: pointer;
font-size: 14px;
height: 28px;
width: 88px;
line-height: 32px;
text-align: center;
}
.header .login-bar .shop-cart:hover{
background: #f0f0f0;
}
.header .login-bar .shop-cart img{
width: 15px;
margin-right: 4px;
margin-left: 6px;
}
.header .login-bar .shop-cart span{
margin-right: 6px;
}
.header .login-bar .login-box{
float: left;
height: 28px;
line-height: 30px;
}
.header .login-bar .login-box span{
color: #4a4a4a;
cursor: pointer;
}
.header .login-bar .login-box span:hover{
color: #000000;
}
</style>

```

components/Footer.vue

```vue
<template>
    <div class="footer">
        <ul>
            <li><router-link to="">企业服务</router-link></li>
            <li><router-link to="">关于我们</router-link></li>
            <li><router-link to="">联系我们</router-link></li>
            <li><router-link to="">商务合作</router-link></li>
            <li><router-link to="">帮助中心</router-link></li>
            <li><router-link to="">意见反馈</router-link></li>
            <li><router-link to="">新手指南</router-link></li>
        </ul>
        <p>Copyright © mingde.com版权所有 | 京ICP备17072161号-1</p>
    </div>
</template>

<script setup lang="ts">

</script>

<style scoped>
.footer {
    width: 100%;
    height: 128px;
    color: #545C63;
}

.footer ul {
    margin: 0 auto 16px;
    padding-top: 38px;
    width: 930px;
}

.footer ul li {
    float: left;
    width: 112px;
    margin: 0 10px;
    text-align: center;
    font-size: 14px;
}

.footer ul::after {
    content: "";
    display: block;
    clear: both;
}

.footer p {
    text-align: center;
    font-size: 12px;
}
</style>

```

提交版本

```bash
git add .
git commit -m "客户端显示首页"
git push 
```

# 
