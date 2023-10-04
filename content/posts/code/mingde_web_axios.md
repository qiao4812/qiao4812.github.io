---
title: "明德项目之客户端集成axios"
date: 2023-06-25T21:38:12+08:00
draft: true
tags: ["Python", "项目"]
categories: ["Python", "项目"]
---

# 明德项目之客户端集成axios

## 客户端集成axios

### main.ts中引入App.vue报错，提示“Cannot find module ‘./App.vue’ or its corresponding type

报错原因：vite使用的是ts，ts不识别 .vue 后缀的文件

### 解决方法：

创建vite项目后，src目录下会有一个 “vite-env.d.ts” 文件，找到该文件，在里面添加一下代码：

```typescript
/// <reference types="vite/client" />
declare module "*.vue" {
  import { DefineComponent } from "vue"
  const component: DefineComponent<{}, {}, any>
  export default component
}
```

[Axios 中文文档](https://axios-http.com/zh/)：https://www.axios-http.cn/

安装axios工具插件

```
cd fuguangweb
yarn add axios@next
```

src/views/Home.vue 代码：

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

// 测试CORS的跨域配置是否有问题
import http from "../utils/http";
// import axios from "axios"
// const http = axios.create()

// 测试客户端的跨域是否配置成功
// http.get("/api/home/demo/").then(response=>{
//   console.log(response.data);
// })

// // 测试服务端的跨域是否配置成功
// http.get("http://api.mingde.cn:8000/home/demo/").then(response=>{
//   console.log(response.data);
// })


// 测试服务端的跨域是否配置成功
http.get("/home/demo/").then(response=>{
  console.log(response.data);
})

</script>

<style scoped>
</style>

```

src/utils/http.ts

```ts
import axios from "axios"

const http = axios.create({
    // timeout: 2500,                          // 请求超时，有大文件上传需要关闭这个配置
    baseURL: "http://api.mingde.cn:8000",     // 设置api服务端的默认地址[如果基于服务端实现的跨域，这里可以填写api服务端的地址，如果基于nodejs客户端测试服务器实现的跨域，则这里不能填写api服务端地址]
    withCredentials: false,                    // 是否允许客户端ajax请求时携带cookie
})

// 请求拦截器
http.interceptors.request.use((config)=>{
    console.log("http请求之前");
    return config;
}, (error)=>{
    console.log("http请求错误");
    return Promise.reject(error);
});

// 响应拦截器
http.interceptors.response.use((response)=>{
    console.log("服务端响应数据成功以后，返回结果给客户端的第一时间，执行then之前");
    return response;
}, (error)=>{
    console.log("服务端响应错误内容的时候。...");
    return Promise.reject(error);
});

export default http;
```

提交版本

```bash
git add .
git commit -m "客户端集成并配置axios"
git push origin main
```

# 
