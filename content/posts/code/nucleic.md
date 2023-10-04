---
title: "核酸采集平台"
date: 2023-03-28T22:55:19+08:00
draft: true
tags: ["项目", "Python"]
categories: ["项目", "Python"]
---

# 核酸采集平台

## 后端项目

### 后端项目环境搭建

```bash
~/Code
➜ mkdir nucleic

~/Code
➜ conda create -n nucleic  python=3.10  # anaconda创建虚拟环境

Retrieving notices: ...working... done
Collecting package metadata (current_repodata.json): done
Solving environment: done
```



![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202303282307943.png)



### FastAPI 和 Uvicorn 包安装

- Uvicorn 是一个 ASGI(Asynchronous Server Gateway Interface，异步服务器网关接口)服务器框架，Uvicorn 为 FastAPI 提供了快速异步运行环境功能。



```bash
~/Code/nucleic via 🅒 nucleic 
➜ pip install fastapi            


~/Code/nucleic via 🅒 nucleic took 6.5s 
➜ pip install "uvicorn[standard]"  # 安装uvicorn来作为服务器

```

- 生成JWT密钥

```bash
➜ openssl rand -hex 32
ed48426ad562bc9fc5026a19cf4b3565a43697fe4dd5e3bbd4adbfcc98396455

```

- SQLAlchemy 安装
- <https://docs.sqlalchemy.org/en/20/intro.html#installation>

```bash
pip install SQLAlchemy
```

- PyMySQL 安装 <https://github.com/PyMySQL/PyMySQL>

```bash
pip install PyMySQL 
```

- 创建数据库

```mysql
➜ mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1255
Server version: 8.0.32 Homebrew

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database if not exists nucleic;
Query OK, 1 row affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| mysql_test         |
| nucleic            |
| performance_schema |
| sys                |
| win_pro            |
| win_pro_test       |
+--------------------+
8 rows in set (0.00 sec)

mysql>

```

## 安装 `python-jose`[¶](https://fastapi.tiangolo.com/zh/tutorial/security/oauth2-jwt/?h=jose#python-jose)

安装 `python-jose`，在 Python 中生成和校验 JWT 令牌：

```bash
pip install python-jose
```

### 创建GitHub仓库

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202303291757726.png)



```bash
~/Code/nucleic via 🅒 nucleic 
➜ git init

nucleic on  master [?] via 🅒 nucleic 
➜ git add .

nucleic on  master [+] via 🅒 nucleic 
➜ git commit -m "first commit"

nucleic on  master via 🅒 nucleic 
➜ git remote add origin git@github.com:qiaopengjun5162/nucleic.git        

nucleic on  main via 🅒 nucleic 
➜ git push -u origin main  # 报错 错误：无法推送一些引用到 

nucleic on  main via 🅒 nucleic took 4.3s 
➜ git branch --set-upstream-to=origin/main       

分支 'main' 设置为跟踪 'origin/main'。

nucleic on  main [⇕] via 🅒 nucleic took 4.8s 
➜ git push -u origin +main
枚举对象中: 41, 完成.
对象计数中: 100% (41/41), 完成.
使用 12 个线程进行压缩
压缩对象中: 100% (39/39), 完成.
写入对象中: 100% (41/41), 9.95 KiB | 637.00 KiB/s, 完成.
总共 41（差异 11），复用 0（差异 0），包复用 0
remote: Resolving deltas: 100% (11/11), done.
To github.com:qiaopengjun5162/nucleic.git
 + d2b2872...185b471 main -> main (forced update)
分支 'main' 设置为跟踪 'origin/main'。


```



```
pip install passlib

pip install python-multipart

pip install bcrypt
```



前端初始化

Vite <https://cn.vitejs.dev/guide/>

```vue
yarn create vite

cd web
  yarn
  yarn dev


yarn add vue-router@4

pnpm install element-plus

yarn add unplugin-vue-components

yarn add axios@next
```

