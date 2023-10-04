---
title: "Rust Web 全栈开发之发布"
date: 2023-06-04T15:19:35+08:00
draft: true
tags: ["Rust"]
categories: ["Rust"]
---

# Rust Web 全栈开发之发布

## 发布

### 第一种方法： WebAssembly不可以

```bash
cargo build --workspace
```

### 第二种方法：分别对3个项目进行构建

#### 构建 webservice

```bash
ws on  main via 🦀 1.70.0 via 🅒 base
➜ cargo build --bin teacher-service --release
```

ws/target/release/teacher-service

#### 构建 webapp

```bash
ws on  main via 🦀 1.70.0 via 🅒 base took 41.8s
➜ cargo build --bin svr --release
```

ws/target/release/svr

### 创建发布文件夹并复制对应文件到当前目录

```bash
ws on  main via 🦀 1.70.0 via 🅒 base took 12.1s
➜ mcd pub

ws/pub on  main via 🦀 1.70.0 via 🅒 base
➜ ls

ws/pub on  main via 🦀 1.70.0 via 🅒 base
➜ cp ../target/release/teacher-service ./

ws/pub on  main [?] via 🦀 1.70.0 via 🅒 base
➜ ls
teacher-service

ws/pub on  main [?] via 🦀 1.70.0 via 🅒 base
➜ cp ../target/release/svr ./

ws/pub on  main [?] via 🦀 1.70.0 via 🅒 base
➜ ls
svr             teacher-service

ws/pub on  main [?] via 🦀 1.70.0 via 🅒 base
➜
```

### 临时设置环境变量并运行

```bash
ws/pub on  main [?] via 🦀 1.70.0 via 🅒 base
➜ export DATABASE_URL=postgres://postgres:postgres@127.0.0.1:5432/postgres


ws/pub on  main [?] via 🦀 1.70.0 via 🅒 base
➜ ./teacher-service


ws/pub on  main [?] via 🦀 1.70.0 via 🅒 base
➜ export HOST_PORT=127.0.0.1:8080


ws/pub on  main [?] via 🦀 1.70.0 via 🅒 base
➜ ./svr
Listening on: 127.0.0.1:8080



```

访问：http://localhost:8080/

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306042020526.png)

### 添加老师测试

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306042021353.png)

### 测试成功

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306042022933.png)

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306042024176.png)

### 发布 WebAssembly 准备工作

- 删除 pkg 文件夹
- 删除 www 下的 dist 文件夹

### 构建 wasm-client

```bash
ws/wasm-client on  main [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base
➜ wasm-pack build --release
```

### 构建 www

```bash
ws/wasm-client/www on  main [?] is 📦 0.1.0 via ⬢ v19.7.0 via 🦀 1.70.0 via 🅒 base
➜ npm install

ws/wasm-client/www on  main [?] is 📦 0.1.0 via ⬢ v19.7.0 via 🦀 1.70.0 via 🅒 base took 10.0s
➜ npm run build

```

### 复制 dist 文件夹到 pub 目录

```bash
ws/pub on  main [?] via 🦀 1.70.0 via 🅒 base
➜ cp -R  ../wasm-client/www/dist ./

ws/pub on  main [?] via 🦀 1.70.0 via 🅒 base
➜ ls
dist            svr             teacher-service

ws/pub on  main [?] via 🦀 1.70.0 via 🅒 base
➜
```

### 安装 http server

```bash
~ via 🅒 base
➜ npm install -g http-server

```

### 运行

```bash
ws/pub on  main [?] via 🦀 1.70.0 via 🅒 base
➜ http-server ./dist -p 8081
Starting up http-server, serving ./dist

http-server version: 14.1.1

http-server settings:
CORS: disabled
Cache: 3600 seconds
Connection Timeout: 120 seconds
Directory Listings: visible
AutoIndex: visible
Serve GZIP Files: false
Serve Brotli Files: false
Default File Extension: none

Available on:
  http://127.0.0.1:8081
  http://192.168.0.100:8081
Hit CTRL-C to stop the server



```

访问：http://localhost:8081/

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306042041707.png)

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306042043669.png)

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306042044243.png)

测试删除

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306042045911.png)

测试添加

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306042046922.png)

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306042047208.png)

