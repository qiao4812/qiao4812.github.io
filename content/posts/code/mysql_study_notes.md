---
title: "MySQL学习笔记"
date: 2023-02-12T18:55:27+08:00
draft: true
---

# 数据库

## 一、数据库介绍

### 1.数据库相关概念

程序是由数据加功能组成的，其中数据是重中之重。

文件管理数据弊端：

远程 套接字

数据库管理软件：本质就是个c/s架构的套接字程序

服务端套接字                 客户端套接字

操作系统：Linux            操作系统

计算机（本地文件）       计算机硬件

例如：

关系型数据库管理软件

​	mysql  oracle

​	去IOE

非关系型数据库管理软件

​	redis  memcache  mongldb

sql语句：套接字管理软件的作者为使用者规定的命令规范

数据库核心概念总结：

数据： 事物的状态

记录： 文件中的一条信息 一行内容

表：一个文件

库： 文件夹

数据库管理软件：套接字程序 mysqld 服务端 3306  mysql  客户端 DBMS 套接字软件

数据库服务器:  运行mysqld的计算机  安装有DBMS服务端的计算机

### 2.数据库安装

官网：[MySQL](https://www.mysql.com/)

求稳不求新

下载页面：[MySQL :: MySQL Downloads](https://www.mysql.com/downloads/)

```python
#1、下载：MySQL Community Server 5.7.16
http://dev.mysql.com/downloads/mysql/

#2、解压
如果想要让MySQL安装在指定目录，那么就将解压后的文件夹移动到指定目录，如：C:\mysql-5.7.16-winx64

#3、添加环境变量
【右键计算机】--》【属性】--》【高级系统设置】--》【高级】--》【环境变量】--》【在第二个内容框中找到 变量名为Path 的一行，双击】 --> 【将MySQL的bin目录路径追加到变值值中，用 ； 分割】
 
#4、初始化
mysqld --initialize-insecure

#5、启动MySQL服务
mysqld # 启动MySQL服务

#6、启动MySQL客户端并连接MySQL服务
mysql -u root -p # 连接MySQL服务器
```

```python
注意：--install前，必须用mysql启动命令的绝对路径
# 制作MySQL的Windows服务，在终端执行此命令：
"c:\mysql-5.7.16-winx64\bin\mysqld" --install
 
# 移除MySQL的Windows服务，在终端执行此命令：
"c:\mysql-5.7.16-winx64\bin\mysqld" --remove



注册成服务之后，以后再启动和关闭MySQL服务时，仅需执行如下命令：
# 启动MySQL服务
net start mysql
 
# 关闭MySQL服务
net stop mysql
```

### 3.数据库简单管理

mysqld  套接字服务端

mysql  套接字客户端  软件管理者

pymysql

mysqladmin -uroot password "123"

mysqladmin -uroot -p"123" password "456"

 update mysql.user set password=password("111") where user="root" and host="localhost";

### 4.sql语句

```python 
SQL语言主要用于存取数据、查询数据、更新数据和管理关系数据库系统,SQL语言由IBM开发。SQL语言分为3种类型：
#1、DDL语句    数据库定义语言： 数据库、表、视图、索引、存储过程，例如CREATE DROP ALTER
#2、DML语句    数据库操纵语言： 插入数据INSERT、删除数据DELETE、更新数据UPDATE、查询数据SELECT
#3、DCL语句    数据库控制语言： 例如控制用户的访问权限GRANT、REVOKE
```

一套命令规范

增删改查

- 库  文件夹

  - 增
    - create database day01 charset utf8mb4;
    - mysql> create database day01 charset utf8mb4;
      Query OK, 1 row affected (0.01 sec)

  - 改
    - alter database day01 charset gbk;

  - 查

    - show databases

    - show create database day01;

  - 删
    - drop database day01;

- 表  文件

use day01;

select database();

1.增  create  table t1(id int , name varchar(16));   day01.t1

2.改  alter table  t1 rename t2;    alter table t1 modify name varchar(10);

3.查  show tables;    desc t2;

4.删  drop table t2;    day01.t2

- 记录  文件中的一行内容

  create database day02;

  create table day02.t1(id int,name varchar(16));

  1.增    insert  day02.t1  t1 values

  ​			(1,"egon"),

  ​			(2,"tom"),

  ​			(3,"jack");

  2.改

  ​	update day02.t1 set name = "lili" where id = 2;

  3.查

  ​	select  *   from day02.t1;

  ​	select name from day02.t1;

  ​	select name from day02.t1 where id >=2;

  4.删    delete from day02.t1 where id = 2;  delete from t1;  truncate t1;

### 5.库相关操作

information_schema： 虚拟库，不占用磁盘空间，存储的是数据库启动后的一些参数，如用户表信息、列信息、权限信息、字符信息等
performance_schema： MySQL 5.5开始新增一个数据库：主要用于收集数据库服务器性能参数，记录处理查询请求时发生的各种事件、锁等现象 
mysql： 授权库，主要存储系统用户的权限信息

- 数据库命名规则

  ```python
  可以由字母、数字、下划线、＠、＃、＄
  区分大小写
  唯一性
  不能使用关键字如 create select
  不能单独使用数字
  最长128位
  ```

## 二、表相关操作

### 1.表的类型  存储引擎

存储引擎说白了就是如何存储数据、如何为存储的数据建立索引和如何更新、查询数据等技术的实现方
法。因为在关系数据库中数据的存储是以表的形式存储的，所以存储引擎也可以称为表类型（即存储和操作此表的类型） 

不同类型文件的处理程序 存储引擎

存储引擎相当于表的类型

innodb: 事务、行级锁

```mysql
MariaDB [(none)]> show engines\G  #查看所有支持的存储引擎
MariaDB [(none)]> show variables like 'storage_engine%'; #查看正在使用的存储引擎
```

```mysql
InnoDB 存储引擎 从 MySQL 5.5.8 版本开始是默认的存储引擎
create table t1(id int)engine=innodb;    文件.mp4
#memory，在重启mysql或者重启机器后，表内数据清空
#blackhole，往表内插入任何数据，都相当于丢入黑洞，表内永远不存记录

```

### 2.数据类型

- 数值类型

  - 整数类型

    create table t8(id int(1));

  - 浮点型

  - 位类型

- 日期类型

- 字符串类型

  - char  定长
  - varchar  变长，精准，节省空间，存取速度慢
  - select char_length(x) from t1;
  - set sql_mode='PAD_CHAR_TO_FULL_LENGTH';
  - select * from t1 where x="你   ";
  - select * from t1 where x like "你___";
  - select * from t1 where x like "你%";
  - like 不会忽略末尾空格 = 会忽略

- 枚举类型与集合类型

  - enum
  - set

### 3.表约束

​	

```mysql
PRIMARY KEY (PK)    标识该字段为该表的主键，可以唯一的标识记录  不为空且唯一
FOREIGN KEY (FK)    标识该字段为该表的外键
NOT NULL    标识该字段不能为空
UNIQUE KEY (UK)    标识该字段的值是唯一的
AUTO_INCREMENT    标识该字段的值自动增长（整数类型，而且为主键）
DEFAULT    为该字段设置默认值

UNSIGNED 无符号
ZEROFILL 使用0填充
create table t5(id int primary key auto_increment,name varchar(16));
create table t10(
    -> id int not null unique,
    -> name varchar(16));
```

### 4.表的三种关系

- 多对一

  

```mysql
create table dep(
    -> id int primary key auto_increment,
    -> name varchar(20),
    -> comment varchar(50)
    -> );

create table emp(
-> id int primary key auto_increment,
-> name varchar(16),
-> age int,
-> dep_id int,
-> foreign key(dep_id) references dep(id)
-> on update cascade
-> on delete cascade
-> );

# 插入数据时，应该先往dep插入数据，再往emp插入数据
 insert dep(name,comment) values
    -> ("IT","搞技术"),
    -> ("sale","卖东西"),
    -> ("HR","招聘");
    
mysql> insert emp(name,age,dep_id) values
    -> ("egon",18,1),
    -> ("tom",19,2),
    -> ("lili",28,2),
    -> ("jack",38,1),
    -> ("lxx",78,3);
Query OK, 5 rows affected (0.33 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> select * from dep;
+----+------+-----------+
| id | name | comment   |
+----+------+-----------+
|  1 | IT   | 搞技术    |
|  2 | sale | 卖东西    |
|  3 | HR   | 招聘      |
+----+------+-----------+
3 rows in set (0.00 sec)

mysql> select * from emp;
+----+------+------+--------+
| id | name | age  | dep_id |
+----+------+------+--------+
|  1 | egon |   18 |      1 |
|  2 | tom  |   19 |      2 |
|  3 | lili |   28 |      2 |
|  4 | jack |   38 |      1 |
|  5 | lxx  |   78 |      3 |
+----+------+------+--------+
5 rows in set (0.00 sec)

mysql> insert emp(name,age,dep_id) values("xxx",38,4);
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`day03`.`emp`, CONSTRAINT `emp_ibfk_1` FOREIGN KEY (`dep_id`) REFERENCES `dep` (`id`) ON DELETE CASCADE ON UPDATE CASCADE)
mysql> update dep set id=333 where name="HR";
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from dep;
+-----+------+-----------+
| id  | name | comment   |
+-----+------+-----------+
|   1 | IT   | 搞技术    |
|   2 | sale | 卖东西    |
| 333 | HR   | 招聘      |
+-----+------+-----------+
3 rows in set (0.00 sec)

mysql> select * from emp;
+----+------+------+--------+
| id | name | age  | dep_id |
+----+------+------+--------+
|  1 | egon |   18 |      1 |
|  2 | tom  |   19 |      2 |
|  3 | lili |   28 |      2 |
|  4 | jack |   38 |      1 |
|  5 | lxx  |   78 |    333 |
+----+------+------+--------+
5 rows in set (0.00 sec)

mysql> delete from dep where id=2;
Query OK, 1 row affected (0.39 sec)

mysql> select * from dep;
+-----+------+-----------+
| id  | name | comment   |
+-----+------+-----------+
|   1 | IT   | 搞技术    |
| 333 | HR   | 招聘      |
+-----+------+-----------+
2 rows in set (0.00 sec)

mysql> select * from emp;
+----+------+------+--------+
| id | name | age  | dep_id |
+----+------+------+--------+
|  1 | egon |   18 |      1 |
|  4 | jack |   38 |      1 |
|  5 | lxx  |   78 |    333 |
+----+------+------+--------+
3 rows in set (0.00 sec)
```

- 一对一
- 多对多  双向多对一  中间表
- 把专门的数据放到专门的表
- 表与表之间找好关系
- 能用多对多不要用多对一，能用多对一不要用一对一。

## 三、记录相关操作

### 1.单表查询

- 增

  - insert t1(字段1,字段2,字段3) values (值1,值2,值3),(值1,值2,值3),(值1,值2,值3);

- 改

  - update t1 set 字段1 = 值1；

- 删

  - delete from 表 where 条件；
  - truncate 表； 清空

- 查

  ```mysql
  mysql> create database day04;
  Query OK, 1 row affected (0.02 sec)
  
  mysql> use day04;
  Database changed
  mysql> create table t1(id int primary key auto_increment,name varchar(16));
  Query OK, 0 rows affected (0.06 sec)
  
  mysql> insert t1(name) values("egon1");
  Query OK, 1 row affected (0.01 sec)
  
  mysql> insert t1(name) values("egon2");
  Query OK, 1 row affected (0.01 sec)
  
  mysql> insert t1(name) values("egon3");
  Query OK, 1 row affected (0.01 sec)
  
  mysql> select * from t1;
  +----+-------+
  | id | name  |
  +----+-------+
  |  1 | egon1 |
  |  2 | egon2 |
  |  3 | egon3 |
  +----+-------+
  3 rows in set (0.00 sec)
  
  mysql> delete from t1;
  Query OK, 3 rows affected (0.01 sec)
  
  mysql> select * from t1;
  Empty set (0.00 sec)
  
  mysql> show create table t1;
  +-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | Table | Create Table                                                                                                                                                                                                                   |
  +-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | t1    | CREATE TABLE `t1` (
    `id` int NOT NULL AUTO_INCREMENT,
    `name` varchar(16) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci |
  +-------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  1 row in set (0.00 sec)
  
  mysql> insert t1(name) values("egon4");
  Query OK, 1 row affected (0.01 sec)
  
  mysql> select * from t1;
  +----+-------+
  | id | name  |
  +----+-------+
  |  4 | egon4 |
  +----+-------+
  1 row in set (0.00 sec)
  
  mysql> truncate t1;
  Query OK, 0 rows affected (0.47 sec)
  
  mysql> show create table t1;
  +-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | Table | Create Table                                                                                                                                                                                                  |
  +-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | t1    | CREATE TABLE `t1` (
    `id` int NOT NULL AUTO_INCREMENT,
    `name` varchar(16) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci |
  +-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  1 row in set (0.00 sec)
  
  mysql> insert t1(name) values("egon5");
  Query OK, 1 row affected (0.01 sec)
  
  mysql> select * from t1;
  +----+-------+
  | id | name  |
  +----+-------+
  |  1 | egon5 |
  +----+-------+
  1 row in set (0.00 sec)
  ```

- ```mysql
  SELECT 字段1,字段2... FROM 表名
                    WHERE 条件
                    GROUP BY field  分组字段
                    HAVING 筛选
                    ORDER BY field  排序字段
                    LIMIT 限制条数
                    
  mysql> select distinct post from employee;
  +-----------------------------------------+
  | post                                    |
  +-----------------------------------------+
  | 秦国大使              |
  | teacher                                 |
  | sale                                    |
  | operation                               |
  +-----------------------------------------+
  4 rows in set (0.03 sec)
  
  select name,salary*12 as annual_salary from employee;
  select name 名字,salary*12 annual_salary from employee;
  
  mysql> select database();
  +------------+
  | database() |
  +------------+
  | day04      |
  +------------+
  1 row in set (0.00 sec)
  
  mysql> select user();
  +----------------+
  | user()         |
  +----------------+
  | root@localhost |
  +----------------+
  1 row in set (0.29 sec)
  
  mysql> select concat("名字",":",name),concat("薪资",":",salary) from employee;
  +---------------------------+-----------------------------+
  | concat("名字",":",name)   | concat("薪资",":",salary)   |
  +---------------------------+-----------------------------+
  | 名字:egon                 | 薪资:7300.33                |
  | 名字:lisi                 | 薪资:1000000.31             |
  | 名字:wangwu              | 薪资:8300.00                |
  | 名字:yuanhao              | 薪资:3500.00                |
  | 名字:liwenzhou            | 薪资:2100.00                |
  | 名字:jingliyang           | 薪资:9000.00                |
  | 名字:jinxin               | 薪资:30000.00               |
  | 名字:成龙                 | 薪资:10000.00               |
  | 名字:歪歪                 | 薪资:3000.13                |
  | 名字:丫丫                 | 薪资:2000.35                |
  | 名字:丁丁                 | 薪资:1000.37                |
  | 名字:星星                 | 薪资:3000.29                |
  | 名字:格格                 | 薪资:4000.33                |
  | 名字:张野                 | 薪资:10000.13               |
  | 名字:程咬金               | 薪资:20000.00               |
  | 名字:程咬银               | 薪资:19000.00               |
  | 名字:程咬铜               | 薪资:18000.00               |
  | 名字:程咬铁               | 薪资:17000.00               |
  +---------------------------+-----------------------------+
  18 rows in set (0.00 sec)
  mysql> select concat("名字",":",name) as new_name,concat("薪资",":",salary) as new_salary from employee;
  
  mysql> select concat_ws(":",name,post,salary) from employee;
  +------------------------------------------------------+
  | concat_ws(":",name,post,salary)                      |
  +------------------------------------------------------+
  | egon:秦国大使:7300.33              |
  | lisi:teacher:1000000.31                              |
  | wangwu:teacher:8300.00                              |
  | yuanhao:teacher:3500.00                              |
  | liwenzhou:teacher:2100.00                            |
  | jingliyang:teacher:9000.00                           |
  | jinxin:teacher:30000.00                              |
  | 成龙:teacher:10000.00                                |
  | 歪歪:sale:3000.13                                    |
  | 丫丫:sale:2000.35                                    |
  | 丁丁:sale:1000.37                                    |
  | 星星:sale:3000.29                                    |
  | 格格:sale:4000.33                                    |
  | 张野:operation:10000.13                              |
  | 程咬金:operation:20000.00                            |
  | 程咬银:operation:19000.00                            |
  | 程咬铜:operation:18000.00                            |
  | 程咬铁:operation:17000.00                            |
  +------------------------------------------------------+
  18 rows in set (0.00 sec)
  
  mysql>    SELECT
      ->        (
      ->            CASE
      ->            WHEN NAME = 'egon' THEN
      ->                NAME
      ->            WHEN NAME = 'lisi' THEN
      ->                CONCAT(name,'_BIGSB')
      ->            ELSE
      ->                concat(NAME, 'SB')
      ->            END
      ->        ) as new_name
      ->    FROM
      ->        employee;
  +--------------+
  | new_name     |
  +--------------+
  | egon         |
  | lisi_BIGSB   |
  | wangwuSB    |
  | yuanhaoSB    |
  | liwenzhouSB  |
  | jingliyangSB |
  | jinxinSB     |
  | 成龙SB       |
  | 歪歪SB       |
  | 丫丫SB       |
  | 丁丁SB       |
  | 星星SB       |
  | 格格SB       |
  | 张野SB       |
  | 程咬金SB     |
  | 程咬银SB     |
  | 程咬铜SB     |
  | 程咬铁SB     |
  +--------------+
  18 rows in set (0.00 sec)
  
  select * from employee where id >=10 and id <= 13;
  select * from employee where id between 10 and 13;
  
  select * from employee where id = 10 or id = 13 or id = 15;
  select * from employee where id in (10,13,15);
  
   select * from employee where post_comment is not null;
   
   select * from employee where name like "ji%";
   
   mysql> select post,max(salary) from employee group by post;
  +-----------------------------------------+-------------+
  | post                                    | max(salary) |
  +-----------------------------------------+-------------+
  | 秦国大使              |     7300.33 |
  | teacher                                 |  1000000.31 |
  | sale                                    |     4000.33 |
  | operation                               |    20000.00 |
  +-----------------------------------------+-------------+
  4 rows in set (0.31 sec)
   select post,min(salary) from employee group by post;
   select post,count(id) from employee group by post;
   mysql> select sex,count(id) from employee group by sex;
  +--------+-----------+
  | sex    | count(id) |
  +--------+-----------+
  | male   |        10 |
  | female |         8 |
  +--------+-----------+
  2 rows in set (0.00 sec)
  
  mysql> select post,avg(salary) from employee where sex = "male" group by post;
  +-----------------------------------------+---------------+
  | post                                    | avg(salary)   |
  +-----------------------------------------+---------------+
  | 秦国大使              |   7300.330000 |
  | teacher                                 | 175650.051667 |
  | operation                               |  16000.043333 |
   select post,avg(salary) from employee where age>20 group by post;
   
   mysql> select * from employee group by post;
  ERROR 1055 (42000): Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'day04.employee.id' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
  
  mysql> select post,avg(salary) from employee where sex = "male" group by post
      -> having avg(salary) > 10000;
  +-----------+---------------+
  | post      | avg(salary)   |
  +-----------+---------------+
  | teacher   | 175650.051667 |
  | operation |  16000.043333 |
  +-----------+---------------+
  2 rows in set (0.00 sec)
  
  mysql> select max(salary) from employee;
  +-------------+
  | max(salary) |
  +-------------+
  |  1000000.31 |
  +-------------+
  1 row in set (0.18 sec)
  
  select * from employee order by salary;
  select * from employee order by salary desc;
  select * from employee order by age asc;
  select * from employee order by age asc, id desc;
  
  mysql> select post,avg(salary) from employee where sex = "male" group by post
      -> having avg(salary) > 10000 order by avg(salary) desc;
  +-----------+---------------+
  | post      | avg(salary)   |
  +-----------+---------------+
  | teacher   | 175650.051667 |
  | operation |  16000.043333 |
  +-----------+---------------+
  2 rows in set (0.00 sec)
  
  mysql> select * from employee limit 3;
  
   select * from employee order by salary desc limit 1;
   
   mysql> select * from employee limit 0,5;
   select * from employee limit 5,5;
  mysql> select * from employee where name regexp "^eg.*$";
  
  ```

### 2.多表查询

- 子查询

- 链表查询

  - 物理表  硬盘
  - 虚拟表  查询语句 拼

  ```mysql
  mysql> select * from department;
  +------+--------------+
  | id   | name         |
  +------+--------------+
  |  200 | 技术         |
  |  201 | 人力资源     |
  |  202 | 销售         |
  |  203 | 运营         |
  +------+--------------+
  4 rows in set (0.00 sec)
  
  mysql> select * from employee;
  +----+------------+--------+------+--------+
  | id | name       | sex    | age  | dep_id |
  +----+------------+--------+------+--------+
  |  1 | egon       | male   |   18 |    200 |
  |  2 | lisi       | female |   48 |    201 |
  |  3 | wangwu    | male   |   38 |    201 |
  |  4 | yuanhao    | female |   28 |    202 |
  |  5 | liwenzhou  | male   |   18 |    200 |
  |  6 | jingliyang | female |   18 |    204 |
  +----+------------+--------+------+--------+
  6 rows in set (0.00 sec)
  
  mysql> desc employee;
  +--------+-----------------------+------+-----+---------+----------------+
  | Field  | Type                  | Null | Key | Default | Extra          |
  +--------+-----------------------+------+-----+---------+----------------+
  | id     | int                   | NO   | PRI | NULL    | auto_increment |
  | name   | varchar(20)           | YES  |     | NULL    |                |
  | sex    | enum('male','female') | NO   |     | male    |                |
  | age    | int                   | YES  |     | NULL    |                |
  | dep_id | int                   | YES  |     | NULL    |                |
  +--------+-----------------------+------+-----+---------+----------------+
  5 rows in set (0.00 sec)
  
  mysql> select * from employee,department;
  
  # 内连接 只保留有对应关系的记录
  mysql> select * from employee,department where employee.dep_id = department.id;
  +----+-----------+--------+------+--------+------+--------------+
  | id | name      | sex    | age  | dep_id | id   | name         |
  +----+-----------+--------+------+--------+------+--------------+
  |  1 | egon      | male   |   18 |    200 |  200 | 技术         |
  |  2 | lisi      | female |   48 |    201 |  201 | 人力资源     |
  |  3 | wangwu   | male   |   38 |    201 |  201 | 人力资源     |
  |  4 | yuanhao   | female |   28 |    202 |  202 | 销售         |
  |  5 | liwenzhou | male   |   18 |    200 |  200 | 技术         |
  +----+-----------+--------+------+--------+------+--------------+
  5 rows in set (0.00 sec)
  
  > select * from employee inner join department on employee.dep_id = department.id;
  # 左连接  在内连接的基础上保留左表的记录
  mysql> select * from employee left join department on employee.dep_id = department.id;
  +----+------------+--------+------+--------+------+--------------+
  | id | name       | sex    | age  | dep_id | id   | name         |
  +----+------------+--------+------+--------+------+--------------+
  |  1 | egon       | male   |   18 |    200 |  200 | 技术         |
  |  2 | lisi       | female |   48 |    201 |  201 | 人力资源     |
  |  3 | wangwu    | male   |   38 |    201 |  201 | 人力资源     |
  |  4 | yuanhao    | female |   28 |    202 |  202 | 销售         |
  |  5 | liwenzhou  | male   |   18 |    200 |  200 | 技术         |
  |  6 | jingliyang | female |   18 |    204 | NULL | NULL         |
  +----+------------+--------+------+--------+------+--------------+
  6 rows in set (0.00 sec)
  # 右连接 在内连接的基础上保留右表的记录
  mysql> select * from employee right join department on employee.dep_id = department.id;
  +------+-----------+--------+------+--------+------+--------------+
  | id   | name      | sex    | age  | dep_id | id   | name         |
  +------+-----------+--------+------+--------+------+--------------+
  |    5 | liwenzhou | male   |   18 |    200 |  200 | 技术         |
  |    1 | egon      | male   |   18 |    200 |  200 | 技术         |
  |    3 | wangwu   | male   |   38 |    201 |  201 | 人力资源     |
  |    2 | lisi      | female |   48 |    201 |  201 | 人力资源     |
  |    4 | yuanhao   | female |   28 |    202 |  202 | 销售         |
  | NULL | NULL      | NULL   | NULL |   NULL |  203 | 运营         |
  +------+-----------+--------+------+--------+------+--------------+
  6 rows in set (0.00 sec)
  # 全外连接 full join 在内连接的基础上 左右两表没有对应关系的记录保留下来
  mysql> select * from employee left join department on employee.dep_id = department.id
      -> union
      -> select * from employee right join department on employee.dep_id = department.id;
  +------+------------+--------+------+--------+------+--------------+
  | id   | name       | sex    | age  | dep_id | id   | name         |
  +------+------------+--------+------+--------+------+--------------+
  |    1 | egon       | male   |   18 |    200 |  200 | 技术         |
  |    2 | lisi       | female |   48 |    201 |  201 | 人力资源     |
  |    3 | wangwu    | male   |   38 |    201 |  201 | 人力资源     |
  |    4 | yuanhao    | female |   28 |    202 |  202 | 销售         |
  |    5 | liwenzhou  | male   |   18 |    200 |  200 | 技术         |
  |    6 | jingliyang | female |   18 |    204 | NULL | NULL         |
  | NULL | NULL       | NULL   | NULL |   NULL |  203 | 运营         |
  +------+------------+--------+------+--------+------+--------------+
  7 rows in set (0.03 sec)
  
  mysql> select * from employee inner join department on employee.dep_id = department.id
      -> where department.name = "技术";
  +----+-----------+------+------+--------+------+--------+
  | id | name      | sex  | age  | dep_id | id   | name   |
  +----+-----------+------+------+--------+------+--------+
  |  1 | egon      | male |   18 |    200 |  200 | 技术   |
  |  5 | liwenzhou | male |   18 |    200 |  200 | 技术   |
  +----+-----------+------+------+--------+------+--------+
  2 rows in set (0.00 sec)
  
  mysql> select employee.name from employee inner join department on employee.dep_id = department.id
      ->  where department.name = "技术";
  +-----------+
  | name      |
  +-----------+
  | egon      |
  | liwenzhou |
  +-----------+
  2 rows in set (0.00 sec)
  
  mysql> select department.name,avg(age) from employee inner join department on employee.dep_id = department.id
      -> group by department.name;
  +--------------+----------+
  | name         | avg(age) |
  +--------------+----------+
  | 技术         |  18.0000 |
  | 人力资源     |  43.0000 |
  | 销售         |  28.0000 |
  +--------------+----------+
  3 rows in set (0.50 sec)
  
  mysql> select * from employee where dep_id =
      -> (select id from department where name = "技术");
  +----+-----------+------+------+--------+
  | id | name      | sex  | age  | dep_id |
  +----+-----------+------+------+--------+
  |  1 | egon      | male |   18 |    200 |
  |  5 | liwenzhou | male |   18 |    200 |
  +----+-----------+------+------+--------+
  2 rows in set (0.04 sec)
  
  mysql> select * from employee where dep_id in
      -> (select id from department where name = "技术" or name = "销售");
  +----+-----------+--------+------+--------+
  | id | name      | sex    | age  | dep_id |
  +----+-----------+--------+------+--------+
  |  1 | egon      | male   |   18 |    200 |
  |  4 | yuanhao   | female |   28 |    202 |
  |  5 | liwenzhou | male   |   18 |    200 |
  +----+-----------+--------+------+--------+
  3 rows in set (0.03 sec)
  
  mysql> select * from employee where dep_id = any
      -> (select id from department where name = "技术" or name = "销售");
  +----+-----------+--------+------+--------+
  | id | name      | sex    | age  | dep_id |
  +----+-----------+--------+------+--------+
  |  1 | egon      | male   |   18 |    200 |
  |  4 | yuanhao   | female |   28 |    202 |
  |  5 | liwenzhou | male   |   18 |    200 |
  +----+-----------+--------+------+--------+
  3 rows in set (0.00 sec)
  mysql> select * from employee where dep_id in(200,202);
  +----+-----------+--------+------+--------+
  | id | name      | sex    | age  | dep_id |
  +----+-----------+--------+------+--------+
  |  1 | egon      | male   |   18 |    200 |
  |  4 | yuanhao   | female |   28 |    202 |
  |  5 | liwenzhou | male   |   18 |    200 |
  +----+-----------+--------+------+--------+
  3 rows in set (0.00 sec)
  
  mysql> select * from employee where dep_id = any(200,202);
  ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '200,202)' at line 1
  mysql>
  
  mysql> select * from employee where age > any(
      -> select avg(age) from employee group by dep_id);
  +----+---------+--------+------+--------+
  | id | name    | sex    | age  | dep_id |
  +----+---------+--------+------+--------+
  |  2 | lisi    | female |   48 |    201 |
  |  3 | wangwu | male   |   38 |    201 |
  |  4 | yuanhao | female |   28 |    202 |
  +----+---------+--------+------+--------+
  3 rows in set (0.02 sec)
  mysql> select * from employee where age < any(
      -> select avg(age) from employee group by dep_id);
  mysql> select * from employee where age > all(
      -> select avg(age) from employee group by dep_id);
      
  mysql> select * from department
      -> where exists(
      -> select * from employee where employee.dep_id = department.id
      -> );
  +------+--------------+
  | id   | name         |
  +------+--------------+
  |  200 | 技术         |
  |  201 | 人力资源     |
  |  202 | 销售         |
  +------+--------------+
  3 rows in set (0.02 sec)
  
  mysql> select * from employee inner join
      -> (select post,max(hire_date) from employee group by post) as t
      -> on employee.post = t.post;
      
  mysql> select * from employee inner join
      -> (select post,max(hire_date) as m from employee group by post) as t
      ->  on employee.post = t.post
      -> where employee.hire_date = t.m;
  ```

  

## 四、在python中如何操作mysql

### 1.pymysql

​	

```mysql
import pymysql
#链接
conn=pymysql.connect(host='localhost',user='root',password='123',database='egon')
#游标
cursor=conn.cursor()

#执行sql语句
sql='select * from userinfo;'
rows=cursor.execute(sql) #执行sql语句，返回sql影响成功的行数rows,将结果放入一个集合，等待被查询

# cursor.scroll(3,mode='absolute') # 相对绝对位置移动
# cursor.scroll(3,mode='relative') # 相对当前位置移动
```

### 2.navicat  图形界面

## 五、数据sql语句其他

- 视图

  视图是一个虚拟表（非真实存在），其本质是【根据SQL语句获取动态的数据集，并为其命名】，用户使用时只需使用【名称】即可获取结果集，可以将该结果集当做表来使用。

  ```mysql
  mysql> create view emp_dep_view as
      -> select employee.*,department.name as depname from employee inner join department
      -> on employee.dep_id = department.id;
  Query OK, 0 rows affected (0.07 sec)
  
  mysql> show tables;
  +--------------+
  | Tables_in_t1 |
  +--------------+
  | department   |
  | emp_dep_view |
  | employee     |
  +--------------+
  3 rows in set (0.00 sec)
  
  mysql> select * from emp_dep_view;
  +----+-----------+--------+------+--------+--------------+
  | id | name      | sex    | age  | dep_id | depname      |
  +----+-----------+--------+------+--------+--------------+
  |  1 | egon      | male   |   18 |    200 | 技术         |
  |  2 | lisi      | female |   48 |    201 | 人力资源     |
  |  3 | wangwu   | male   |   38 |    201 | 人力资源     |
  |  4 | yuanhao   | female |   28 |    202 | 销售         |
  |  5 | liwenzhou | male   |   18 |    200 | 技术         |
  +----+-----------+--------+------+--------+--------------+
  5 rows in set (0.01 sec)
  ```

  

- 触发器

  使用触发器可以定制用户对表进行【增、删、改】操作时前后的行为，注意：没有查询

- 存储过程

  存储过程包含了一系列可执行的sql语句，存储过程存放于MySQL中，通过调用它的名字可以执行其内部的一堆sql

  ```mysql
  优点：
  #1. 用于替代程序写的SQL语句，实现程序与sql解耦
  
  #2. 基于网络传输，传别名的数据量小，而直接传sql数据量大
  缺点：
  #1. 程序员扩展功能不方便
  ```

- 函数

  ```mysql
  一、数学函数
      ROUND(x,y)
          返回参数x的四舍五入的有y位小数的值
          
      RAND()
          返回０到１内的随机值,可以通过提供一个参数(种子)使RAND()随机数生成器生成一个指定的值。
  
  二、聚合函数(常用于GROUP BY从句的SELECT查询中)
      AVG(col)返回指定列的平均值
      COUNT(col)返回指定列中非NULL值的个数
      MIN(col)返回指定列的最小值
      MAX(col)返回指定列的最大值
      SUM(col)返回指定列的所有值之和
      GROUP_CONCAT(col) 返回由属于一组的列值连接组合而成的结果  
      
  mysql> select date_format(sub_time,"%Y-%m"),count(id) from blog group by date_format(sub_time,"%Y-%m");
  +-------------------------------+-----------+
  | date_format(sub_time,"%Y-%m") | count(id) |
  +-------------------------------+-----------+
  | 2015-03                       |         2 |
  | 2016-07                       |         4 |
  | 2017-03                       |         3 |
  +-------------------------------+-----------+
  3 rows in set (0.03 sec)
  ```

- 流程控制

  - if
  - while
  - repeat
  - loop

## 六、索引原理与慢查询优化

什么是索引？

​	索引是存储引擎中一种数据结构，或者说数据的组织方式，又称之为健key

​	为数据建立索引就好比是为书建目录

为何要用索引？

​	为了优化查询效率

​	ps:创建完索引后会降低增、删、改的效率

```mysql
索引在MySQL中也叫做“键”，是存储引擎用于快速找到记录的一种数据结构。
索引对于良好的性能非常关键，尤其是当表中的数据量越来越大时，索引对于性能的影响愈发重要。
索引优化应该是对查询性能优化最有效的手段了。索引能够轻易将查询性能提高好几个数量级。
索引相当于字典的音序表，如果要查某个字，如果不使用音序表，则需要从几百页中逐页去查.
```

- 二叉树
- 平衡二叉树
- B树
- B+树  
  - 在二叉树、平衡二叉树、B树的基础上做了进一步优化，只有叶子节点放真正的数据，这意味着在等量数据的前提下，B+树的高度是最低的
  - B+树的叶子节点都是排好序的，这意味着在范围查询上，B+树比B树更快，快就快在一旦找到一个树叶节点，就不需要再从树根查起来了
- innodb存储引擎索引分类：
  - hash索引:更适合等值查询，不适合范围查询
  - B+树索引
    - 聚焦索引/聚簇索引  以主键字段的值作为key创建的索引（一张表中只有一个）
    - 辅助索引：针对非主键字段创建的索引（一张表中可以有多个）

innodb通过索引组织表

回表查询：通过辅助索引拿到主键值，然后再回到聚集索引从根再查一遍

覆盖索引：不需要回表查询就能拿到你想要的全部数据

250万条记录  大约相当于 ibd文件大小167M

数据即索引，索引即数据。

```mysql
#===========B+树索引（innodb存储引擎默认）

聚集索引：即主键索引，PRIMARY KEY
    用途：
        1、加速查找
        2、约束（不为空、不能重复）

唯一索引：UNIQUE
    用途：
        1、加速查找
        2、约束（不能重复） 

普通索引INDEX：
    用途：
        1、加速查找

联合索引： 
    PRIMARY KEY(id,name):联合主键索引 
    UNIQUE(id,name):联合唯一索引 
    INDEX(id,name):联合普通索引
    
命中索引也未必能起到很好的提速效果
	1.对区分度高并且占用空间小的字段建立索引
	2.针对范围查询命中了索引，如果范围很大，查询效率依然很低，如果解决
		要么把范围缩小
		要么就分段取值，一段一段取最终把大范围给取完
    3.索引下推技术（默认开启）
    4.不要把查询字段放到函数或者参与运算
    5.索引覆盖
    6.索引最左前缀匹配原则
```

## 七、数据库事务 transaction（交易）

1.什么是事务？

级联删除

一个事务里可以包含多条sql语句

ACID事务四大特性：

### 1.原子性

### 2.一致性

### 3.隔离性

### 4.持久性

2.为何要有事务

3.事务

隐示事务：隐示开启隐示提交

显示事务：显示开启 显示提交

- 开启事务 start transaction 或者 begin
- commit； 提交事务
- rollback; 回滚事务
- commit或者rollback事务都结束
- pymysql隐示开启显示提交commit

## 数据库读现象

并发开启多个事务去操作同一份数据，有可能会引发数据库读现象

### 1.脏读（dirty read)

​	一个事务二修改完没有提交后事务一查看后事务二回滚 造成无效数据 错误数据

### 2.不可重复读(nonrepeatable read)

​	事务一查看后事务二更新修改后提交事务一再读  两次数据不同已修改

### 3.幻读 幻象读取（phantom read)

​	是不可重复读的一种特殊现象

## 九、数据库锁机制与mvcc

互斥锁 信号量

事务隔离机制  级别越高 并发能力越低

锁的分类：

- 按粒度分

  - 行级锁
    - 共享锁
    - 排他锁 互斥锁  insert  update  delete

  - 表级锁（偏向于读）

  - 页级锁

- 按级别划分

  - 共享锁 (s)
  - 排他锁（x)

- 按使用方式划分

  - 乐观锁
  - 悲观锁

- 按加锁方式划分

  - 自动锁
  - 显示锁

- 按操作划分

  - DML锁
  - DDL锁

1.一旦某个事务1对数据A加了排他锁，则其他事务无法对数据A加任何锁，只有事务1可以操作数据A，而且可以读也可以写

2.一旦事务1对数据A加了共享锁，则其他事务可以对数据A加锁，但只能加共享锁，只有事务1可以操作数据A，而且可以读也可以写

3.一旦多个事务都对数据A加了共享锁，大家都只能读不能改

1. **保持事务短小**
2. **尽量避免事务中rollback**
3. **尽量避免savepoint**
4. **显式声明打开事务**
5. **默认情况下，依赖于悲观锁，为吞吐量要求苛刻的事务考虑乐观锁**
6. **锁的行越少越好，锁的时间越短越好**

死锁问题：

​	1.简单情况

​	事务一：

​	begin;

select * from employee where id = 1 for update;  -- 第一步

update employee set name="xxx" where id = 3;  -- 第三步

事务二：

begin;

delete from employee where id =3;

delete from employee where id =1;

```mysql
行级锁有三种算法：
            Record lock
            Gap lock
            Next-key lock（默认）= Record lock+ Gap lock    ------》 解决幻读问题
```

|                                             | 特点                                                         | 脏读 | 不可重复读 | 幻读 |
| ------------------------------------------- | ------------------------------------------------------------ | ---- | ---------- | ---- |
| Read uncommitted（独立提交，未提交读）      | 允许事务查看其他事务所进行的未提交更改                       | √    | √          | √    |
| Read committed（提交读）                    | 允许其他事务查看已经提交的事务                               | ×    | √          | √    |
| Repeatable read（可重复读，innodb引擎默认） | 确保每个事务的 SELECT 输出一致 InnoDB 的默认级别 #commit之后，其他窗口看不到数据，必须退出重新登录查看 | ×    | ×          | √    |
| Serializable（可序列化、串行化）            | 将一个事务于其他事务完全隔离，即串行化  #当一个事务没有提交，查询也不行。例如：我改微信头像的时候你不能看我的信息，我看你朋友圈的时候你不能发朋友圈也不能看朋友圈 | ×    | ×          | ×    |

mvcc 快照读  当前读

innodb一页16k

写软件

- 设计数据库表结构
- 字符编码

```mysql
mysql> alter table t1 add index (userpass);
Query OK, 0 rows affected (0.06 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc t1;
+----------+-------------------+------+-----+------------------------+-------------------+
| Field    | Type              | Null | Key | Default                | Extra             |
+----------+-------------------+------+-----+------------------------+-------------------+
| id       | int unsigned      | NO   | PRI | NULL                   | auto_increment    |
| username | varchar(50)       | NO   | MUL | NULL                   |                   |
| userpass | varchar(50)       | NO   | MUL | NULL                   |                   |
| telno    | varchar(20)       | NO   | UNI | NULL                   |                   |
| sex      | enum('男','女')   | NO   |     | 男                     |                   |
| birthday | date              | NO   |     | _utf8mb4\'0000-00-00\' | DEFAULT_GENERATED |
+----------+-------------------+------+-----+------------------------+-------------------+
6 rows in set (0.00 sec)

mysql>  alter table t1 modify id int unsigned auto_increment;
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0


mysql> show session variables like 'auto_inc%';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| auto_increment_increment | 1     |
| auto_increment_offset    | 1     |
+--------------------------+-------+
2 rows in set, 1 warning (0.10 sec)

mysql> set session auto_increment_increment =2;
Query OK, 0 rows affected (0.09 sec)

mysql> show session variables like 'auto_inc%';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| auto_increment_increment | 2     |
| auto_increment_offset    | 1     |
+--------------------------+-------+
2 rows in set, 1 warning (0.00 sec)

mysql> delect from test3;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'delect from test3' at line 1
mysql> delete from test3;
Query OK, 3 rows affected (0.01 sec)

mysql> show create table test3\G
*************************** 1. row ***************************
       Table: test3
Create Table: CREATE TABLE `test3` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(10) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
1 row in set (0.00 sec)

mysql> alter table test3 AUTO_INCREMENT=1;
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> insert into test3 values(null,'xxx');
Query OK, 1 row affected (0.01 sec)

mysql> select * from test3;
+----+----------+
| id | username |
+----+----------+
|  1 | xxx      |
+----+----------+
1 row in set (0.00 sec)
# 	常规索引
mysql> create table test4(
    -> id int unsigned primary key auto_increment,
    -> username varchar(10),
    -> index u_index(username)
    -> );
Query OK, 0 rows affected (0.06 sec)

mysql> show create table test4\G
*************************** 1. row ***************************
       Table: test4
Create Table: CREATE TABLE `test4` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(10) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `u_index` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
1 row in set (0.00 sec)

mysql> create table test5(
    -> username varchar(10),
    -> key (username)
    -> );
Query OK, 0 rows affected (0.07 sec)

mysql> show create table test5\G
*************************** 1. row ***************************
       Table: test5
Create Table: CREATE TABLE `test5` (
  `username` varchar(10) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  KEY `username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
1 row in set (0.00 sec)

mysql> drop index username on test5;
Query OK, 0 rows affected (0.30 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> create index username on test5 (username);
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> show create table test5\G
*************************** 1. row ***************************
       Table: test5
Create Table: CREATE TABLE `test5` (
  `username` varchar(10) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  KEY `username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
1 row in set (0.00 sec)

mysql> create table test6(
    -> id int unsigned primary key auto_increment,
    -> username varchar(10),
    -> unique u_username(username)
    -> );
Query OK, 0 rows affected (0.09 sec)

mysql> create table test7(
    -> id int unsigned primary key auto_increment,
    -> username varchar(10),
    -> unique (username)
    -> );
Query OK, 0 rows affected (0.44 sec)

mysql> show create table test7\G
*************************** 1. row ***************************
       Table: test7
Create Table: CREATE TABLE `test7` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(10) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
1 row in set (0.00 sec)

mysql> insert into test8(username,age) select username,age from test8;
Query OK, 2 rows affected (0.29 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select * from test8;
+----+----------+------+
| id | username | age  |
+----+----------+------+
|  1 | lucky    |   18 |
|  2 | lucky    |   18 |
|  3 | lucky    |   18 |
|  4 | lucky    |   18 |
+----+----------+------+
4 rows in set (0.00 sec)

mysql> set profiling=1;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> select * from test8 where id=4;
+----+----------+------+
| id | username | age  |
+----+----------+------+
|  4 | lucky    |   18 |
+----+----------+------+
1 row in set (0.00 sec)

mysql> show profiling;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'profiling' at line 1
mysql> select * from test8 where id=4;
+----+----------+------+
| id | username | age  |
+----+----------+------+
|  4 | lucky    |   18 |
+----+----------+------+
1 row in set (0.00 sec)

mysql> show profiles;
+----------+------------+--------------------------------+
| Query_ID | Duration   | Query                          |
+----------+------------+--------------------------------+
|        1 | 0.00061775 | select * from test8 where id=4 |
|        2 | 0.00014775 | show profiling                 |
|        3 | 0.00054600 | select * from test8 where id=4 |
+----------+------------+--------------------------------+
3 rows in set, 1 warning (0.00 sec)

mysql>


mysql> create table test9(
    -> id int primary key auto_increment
    -> );
Query OK, 0 rows affected (0.42 sec)

mysql> desc test9;
+-------+------+------+-----+---------+----------------+
| Field | Type | Null | Key | Default | Extra          |
+-------+------+------+-----+---------+----------------+
| id    | int  | NO   | PRI | NULL    | auto_increment |
+-------+------+------+-----+---------+----------------+
1 row in set (0.01 sec)

mysql> alter table test9 add
    -> username varchar(10) not null
    -> default 'aa' comment '用户名';
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> alter table test9 change username username varchar(10);
Query OK, 0 rows affected (0.08 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc test9;
+----------+-------------+------+-----+---------+----------------+
| Field    | Type        | Null | Key | Default | Extra          |
+----------+-------------+------+-----+---------+----------------+
| id       | int         | NO   | PRI | NULL    | auto_increment |
| username | varchar(10) | YES  |     | NULL    |                |
+----------+-------------+------+-----+---------+----------------+
2 rows in set (0.00 sec)

mysql> alter table test9 add key k_username(username);
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> desc test9;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int         | NO   | PRI | NULL    |       |
| age      | int         | YES  |     | NULL    |       |
| username | varchar(10) | YES  | MUL | NULL    |       |
+----------+-------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```



```mysql
mysql> create table a(
    -> id int unsigned primary key auto_increment,
    -> name varchar(10) default 'lucky'
    -> );
Query OK, 0 rows affected (0.05 sec)

mysql> desc a;
+-------+--------------+------+-----+---------+----------------+
| Field | Type         | Null | Key | Default | Extra          |
+-------+--------------+------+-----+---------+----------------+
| id    | int unsigned | NO   | PRI | NULL    | auto_increment |
| name  | varchar(10)  | YES  |     | lucky   |                |
+-------+--------------+------+-----+---------+----------------+
2 rows in set (0.00 sec)

mysql> create table b(
    -> id int unsigned primary key auto_increment,
    -> a_id int unsigned,
    -> name varchar(10) default "",
    -> key a_id(a_id),
    -> constraint b_f_k foreign key (a_id) references a (id)
    -> );
Query OK, 0 rows affected (0.10 sec)

mysql> desc b;
+-------+--------------+------+-----+---------+----------------+
| Field | Type         | Null | Key | Default | Extra          |
+-------+--------------+------+-----+---------+----------------+
| id    | int unsigned | NO   | PRI | NULL    | auto_increment |
| a_id  | int unsigned | YES  | MUL | NULL    |                |
| name  | varchar(10)  | YES  |     |         |                |
+-------+--------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)

mysql> insert into a values (null,'lucky');
Query OK, 1 row affected (0.01 sec)

mysql> insert into b values (null,1,'b');
Query OK, 1 row affected (0.04 sec)

mysql> select * from a;
+----+-------+
| id | name  |
+----+-------+
|  1 | lucky |
+----+-------+
1 row in set (0.00 sec)

mysql> select * from b;
+----+------+------+
| id | a_id | name |
+----+------+------+
|  1 |    1 | b    |
+----+------+------+
1 row in set (0.00 sec)

mysql> delect from b;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'delect from b' at line 1
mysql> delete from b;
Query OK, 1 row affected (0.01 sec)

mysql> delete from a;
Query OK, 1 row affected (0.01 sec)

mysql>  insert into a values (null,'lucky');
Query OK, 1 row affected (0.01 sec)

mysql> insert into b values (null,1,'b');
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`d14`.`b`, CONSTRAINT `b_f_k` FOREIGN KEY (`a_id`) REFERENCES `a` (`id`))
mysql> select * from a;
+----+-------+
| id | name  |
+----+-------+
|  2 | lucky |
+----+-------+
1 row in set (0.00 sec)

mysql> insert into b values (null,2,'b');
Query OK, 1 row affected (0.01 sec)

mysql> select * from b;
+----+------+------+
| id | a_id | name |
+----+------+------+
|  3 |    2 | b    |
+----+------+------+
1 row in set (0.00 sec)

mysql> delete from a;
ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails (`d14`.`b`, CONSTRAINT `b_f_k` FOREIGN KEY (`a_id`) REFERENCES `a` (`id`))
mysql> show create table b\G
*************************** 1. row ***************************
       Table: b
Create Table: CREATE TABLE `b` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `a_id` int unsigned DEFAULT NULL,
  `name` varchar(10) COLLATE utf8mb4_unicode_ci DEFAULT '',
  PRIMARY KEY (`id`),
  KEY `a_id` (`a_id`),
  CONSTRAINT `b_f_k` FOREIGN KEY (`a_id`) REFERENCES `a` (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
1 row in set (0.00 sec)

mysql> alter table b drop foreign key b_f_k;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> alter table b add foreign key (a_id) references a(id) on delete cascade on update cascade;
Query OK, 1 row affected (0.16 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> delete from a;
Query OK, 1 row affected (0.01 sec)

mysql> select * from a;
Empty set (0.00 sec)

mysql> select * from b;
Empty set (0.00 sec)
```



数据库：数据的仓库（集散地），它解决了数据持久化和数据管理的问题

持久化 将数据从内存转移到硬盘（可以长久保存数据的存储介质）

数据库的分类：

​	1972  codd 如何使用关系模型来保存大规模数据

- 关系型数据库  首选方案
  - 理论基础：关系代数、集合论
  - 具体表象：用二维表保存数据（行（记录）和列（字段））
  - 编程语言：SQL（结构化查询语言）  -》 SQL方言
- 非关系型数据库
  - NoSQL  No SQL -> No, SQL -> Not Only SQL
  - NewSQL 保存数据的方式可能完全不同于传统的关系型数据库，但是允许使用关系型数据库的编程语言操作/获取数据

Hadoop生态圈   Hive -> HQL -> 跟MySQL中使用的SQL无限雷同 PIG  Drill

关系型数据库的产品：

- Oracle             Oracle    金融、证券、电商、电子政务    好 贵  甲骨文  
- MySQL            GRL    社区版    MariaDB
- PostgreSQL / IBM DB2 / Microsoft SQLServer    / TiDB /

services.msc 启动服务窗口

mysql> select version();
+-----------+
| version() |
+-----------+
| 8.0.26    |
+-----------+
1 row in set (0.00 sec)

SQL (Strucutured Query Language)  结构化查询语言

- DDL (数据定义语言)    创建删除修改各种对象   create   drop  alter
- DML (数据操作语言)   插入 删除 修改数据         insert    delete   update
- DQL (数据查询语言)   检索（查询）数据              select
- DCL  (数据控制语言)   授予或者召回用户权限   grant   revoke

SQL是不区分大小写的编程语言

0.查看所有数据库

show databases;

1. 创建数据库

   create database school default charset utf8mb4;

2. 删除数据库

   drop database if exists school;

3. 查看创建数据库的过程

   show create database school;

4. 切换到指定的数据库

   use school;

5. 显示数据库中所有的表

   show tables;

6. 创建二维表

   create table tb_student

   (

   ​	stu_id int unsigned not null comment '学号',

   ​	stu_name varchar(20) not null comment '姓名',

   ​	stu_sex boolean default 1 comment '性别',

   ​	stu_birth date comment '出生日期',

   ​	primary key (stu_id)

   ) engine=innodb comment '学生表';

```mysql
mysql> create table tb_student
    -> (
    -> stu_id int unsigned not null comment '学号',
    -> stu_name varchar(20) not null comment '姓名',
    -> stu_sex boolean default 1 comment '性别',
    -> stu_birth date comment '出生日期',
    -> primary key (stu_id)
    -> )engine=innodb comment '学生表';
Query OK, 0 rows affected (0.06 sec)
```



 ? data types;  查询数据库支持的类型

主键（primary key ) : 能够唯一确定一条记录的列

数据类型：

- 整数：int(integer) / bigint / smallint/ tinyint    -----> unsigned
- 小数：float / double / decimal
- 时间日期：time / date / datetime / timestamp
- 字符串： char / varchar
- 大对象：longtext / longblob            4G











