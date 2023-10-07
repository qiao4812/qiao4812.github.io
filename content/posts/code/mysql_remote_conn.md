---
title: "Mysql设置远程连接"
date: 2023-02-23T16:16:22+08:00
draft: false
Tags: ["数据库", "MySQL"]
Categories: ["数据库"] 
---

# Mac下MySQL设置远程连接

- ### 登录MySQL数据库

```mysql
mysql -uroot -p
```

- ### 进入MySQL库

```mysql
use mysql;
```

- ### 执行更新权限SQL语句

```mysql
update user set Host='%' where User='root';
```

- ### 查看权限

```mysql
select host, user from user;
```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202302221041404.png)

- 修改配置

```bash
cd /opt/homebrew/etc
 
vim my.cnf

# 修改MySQL服务绑定的IP
bind-address = 0.0.0.0
```

- 重启MySQL

```bash
brew services restart mysql
```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202302221534041.png)
