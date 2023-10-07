---
title: "Python之Django学习笔记"
date: 2023-02-12T18:44:29+08:00
draft: false
tags: ["Python"]
categories: ["Python"]
---

# Django

## 软件开发架构

### 1.C/S

### 2.B/S

- B/S本质也是一种C/S架构

## HTTP协议

"""规定了浏览器与服务器之间数据交互的格式"""

### 一.四大特性

1.基于TCP、IP作用于应用层之上的协议

2.基于请求响应

3.无状态

- 见你千百遍我都当你如初见
- cookie、session、token...

4.无（短）连接

- 长连接: websocket

### 二、数据格式

- 请求数据格式
  - 请求首行（请求方法...)
  - 请求头（一大堆K:V键值对）
  - 换行 换行符 不能省略
  - 请求体（并不是所有的请求方法都有请求体 主要用来携带敏感性数据）
- 响应数据格式
  - 响应首行（响应状态码...)
  - 响应头（一大堆K:V键值对）
  - 换行
  - 响应体（展示给用户的数据）

### 三、响应状态码

用简单的数字来表示一串中文意思

- 1xx： 服务端已经接受到你的数据正在处理，你可以继续提交
- 2xx：200 OK  请求成功
- 3xx： 重定向 （原本想访问A 但是内部跳到B)
- 4xx： 403 当前请求不符合条件  404 请求资源不存在
- 5xx：服务器内部错误

注意：除了上述统一的响应状态码之外，公司还可以自定义自己的状态码

### 四、请求方法

#### 1.get请求

- 朝别人索要数据

#### 2.post请求

- 朝别人提交数据

## WEB框架  

### 问题一

- 服务端代码重复
- 手动处理http数据格式过于繁琐

基于wsgiref模块解决了上述两个问题

### 问题二

- 网址很多的情况下如何匹配
- 当功能比较复杂的时候如何处理

> #### 封装处理

##### 1.定义一个网址与功能函数的对应关系

```python
# 地址与功能对应关系
urls = [
    ('/index',index_func),
    ('/login',login_func),
    ('/reg',reg_func),
]
```

##### 2.按照功能的不同划分成不同的py文件

```python
urls.py views.py mainrun.py
```

##### 3.书写服务端代码

```python
func = None
# for 循环判断
for url_tuple in urls:  # (),()
    if current_path == url_tuple[0]:
        # 将匹配到的函数名赋值给func变量
        func = url_tuple[1]
        break
# 判断func是否有值
if func:
    # 执行对应的函数
    res = func(request)
else:
    res = errors(request)
return [bytes(res,encoding='utf-8')]
```

> #### 动静态网页

```python
静态网页

 数据全部都是写死的，万年不变

动态网页
 数据来源于后端（代码、数据库）
    
1.访问网址展示当前时间（由后端模块生成并展示到HTML页面）
 def get_time_func(request):
    from datetime import datetime
    current_time = datetime.now().strftime('%Y-%m-%d %X')
    with open(r'get_time.html','r',encoding='utf-8') as f:
        data = f.read()  # 字符串
    data = data.replace('abdc',current_time)
    return data
2.后端有一个字典，将该字典传递给HTML页面，并且在该页面上还可以使用字典取值的各种操作
 jinja2模块
    pip3 install jinja2
    ps:该模块是flask框架必备的模块，所以下载flask也会自动下载该模块
    模板语法(用近似于python的语法在HTML文件上操作)
      {{user_data}}
        {{user_data['username']}}
        {{user_data.get('password')}}
        {{user_data.hobby}}
        
        {%for user_dict in data_list%}
            <tr>
                <td>{{user_dict.id}}</td>
                <td>{{user_dict.name}}</td>
                <td>{{user_dict.age}}</td>
            </tr>
        {%endfor%}
3.获取MySQL数据库数据展示到页面上
 def get_db_func(request):
    import pymysql
    conn = pymysql.connect(
        host='127.0.0.1',
        port=3306,
        user='root',
        password='root',
        database='db666',
        charset='utf8',
        autocommit=True
    )
    cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
    sql = 'select * from userinfo'  # 分号可写可不写
    affect_rows = cursor.execute(sql)
    res1 = cursor.fetchall()  # [{},{},{}]
    # print(res)
    with open(r'templates/get_db.html', 'r', encoding='utf-8') as f:
        data = f.read()
    temp = Template(data)
    res = temp.render(data_list=res1)
    return res 

```

```mysql
mysql> create database db666;
Query OK, 1 row affected (0.36 sec)

mysql> use db666
Database changed
mysql> create table userinfo(
    -> id int primary key auto_increment,
    -> name char(32),
    -> age int
    -> );
Query OK, 0 rows affected (0.36 sec)

mysql> insert into userinfo(name,age) values('jason',18),('egon',84),('tony',77),('jack',66);
Query OK, 4 rows affected (0.12 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> select name,age from userinfo;
+-------+------+
| name  | age  |
+-------+------+
| jason |   18 |
| egon  |   84 |
| tony  |   77 |
| jack  |   66 |
+-------+------+
4 rows in set (0.03 sec)
```

### 总结

```python
1.纯手工写WEB框架
2.wsgire模块
 1.封装了socket代码
 2.处理了http数据格式
3.根据功能的不同拆分成不同的文件夹
 urls.py  路由与视图函数对应关系
 views.py  视图函数
 templates  模板文件夹
 # 1. 第一步 添加路由与视图函数的对应关系
 # 2. 去views中书写功能代码
 # 3. 如果需要使用到HTML则去模板文件夹中操作
4.jinja2模板语法
 {{}}
 {%%}
5.简易版本web框架流程图
```

> ###主流web框架

```python
1.django框架
 大而全，自带的功能组件非常非常多！类似于航空母舰
 
2.flask框架
 小而精，自身的功能组件非常非常少！类似于游骑兵
 但是第三方模块非常之多，如果把第三方模块全部叠加起来完全可以盖过Django
 有时候也会受限于第三方模块
 ps:三行代码就可以启动一个flask后端服务
 
3.tornado框架
 异步非阻塞  速度非常的快 快到可以开发游戏服务器
ps:Sanic、FastAPI...
"""注意:小白不要同时学习两个及以上"""
A: socket部分
B：路由与视图匹配
C：模板语法

Django
 A:用的是wsgiref模块
 B:自己写的
 C:自己写的
flask
 A:用的是wsgiref模块封装之后werkzeug
 B:自己写的
 C:jinja2模块
tornado
 A:自己写的
 B:自己写的
 C:自己写的
```

> ### django框架

```python
# 注意事项
 1.计算机名称不能有中文
 2.项目名和py文件名最好也不要使用中文
 3.Django版本问题
    1.X:
    2.X:1和2几乎一样
    3.X:异步
ps:版本选择1.11.11版本

# 命令行下载
 pip3 install django==1.11.11
# 测试是否安装完成
 Django-admin
# 查看Django版本
1.  python -m django --version
2.  import django
 django.get_version()
```

> #### 命令行模式

```python
C:\Users\Thinkpad>d:
D:\>django-admin startproject mysite
1.创建Django项目
 django-admin startproject 项目名
2.启动Django项目
 cd 项目名
 python3 manage.py runserver ip:port
 ps:如果报错需要修改py文件源码
 D:\python\lib\site-packages\django\contrib\admin\widgets.py
 line 152行后面的逗号去掉即可!!!
  '%s=%s' % (k, v) for k, v in params.items()
D:\>cd mysite
D:\mysite>python manage.py runserver
# 错误
SyntaxError: Generator expression must be parenthesized (widgets.py, line 152)
# 成功提示
Django version 1.11.11, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
# 浏览器输入网址http://127.0.0.1:8000/
Next, start your first app by running python manage.py startapp [app_label].
接下来，通过运行 python manage.py startapp [app_label] 来启动你的第一个应用程序。
You're seeing this message because you have DEBUG = True in your Django settings file and you haven't configured any URLs. Get to work!
您看到此消息是因为您的 Django 设置文件中有 DEBUG = True 并且您尚未配置任何 URL。 开始工作！
3.创建app
  python manage.py startapp app名字
    D:\mysite>python manage.py startapp app01
```

> ### app

```python
django是一款专门开发app(应用)的软件

我们创建一个Django项目之后类似于创建了一所大学
而app就类似于大学里面的各个学院，每个学院都可以有自己独立的各项功能职责
Django相当于是一个空壳子用来给各个学院提供资源!!!
"""我们创建的app一定要去setting文件中注册才能生效"""
```

> ###pycharm快捷方式

```python
1.new project
 django
  项目名
  解释器
  应用名
# pycharm会自动帮你创建一个app
```

#### 总结

```python
命令行与pycharm创建不同点
 1.命令行不会自动创建templates模板文件
 2.命令行也不会自动配置模板文件夹路径
  命令行: 'DIRS': [],
  pycharm: 'DIRS': [os.path.join(BASE_DIR, 'templates')]
    
```

> django目录结构

```python
mysite
 mysite文件夹     # 项目同名文件夹
  __pycache__   # 缓存
  __init__.py
  settings.py    # Django暴露给用户可以配置的配置文件
  urls.py*   # 路由与视图函数（可以是函数也可以是类）对应关系(路由层)
  wsgi.py   # 忽略 
 app01文件夹        # 应用（可以有多个）
  migrations文件夹 # 存储数据库记录相关（类似于操作日志）
  _init_.py
  admin.py  # django后台管理
  apps.py   # 注册app
  models.py  # 数据库相关(模型层)
  tests.py  # 测试文件
  views.py*  # 视图函数(视图层)
 db.sqlite3     # Django自带的小型数据库
 manage.py      # django入口文件
 templates*  # 模板文件（存储html文件）(模板层)
```

python没有常量默认全部大写的变量看作常量

Tools  Run manage.py Task... (Ctrl+Alt+R)

> ###小白必会三板斧

```python
1.HttpResponse
 返回字符串
2.render
 返回HTML页面，还可以使用模板语法
3.redirect 重定向
 重定向
```

> ### 作业

```python
1.理解上午框架推导思路（代码无需掌握）
2.独立完成Django安装及启动
3.使用Django展示数据库数据
 urls.py  views.py  templstes
```

## 今日内容概要

主题：登录功能

- 静态文件配置
- request对象方法
- pycharm如何链接数据库MySQL
- django如何链接MySQL
- Django orm 简介(重点)
- 数据的增删改查CURD

## 今日内容详细

> ####静态文件配置

---

```python
"""
我们之所以能够在浏览器地址栏里面输入网址就可以拿到对应的资源
是因为开发者早已经提前开设了该资源的访问接口
"""
1.静态文件
 写好之后不会自动动态改变的文件资源，比如我们写好的CSS文件、JS文件、图片文件、第三方框架文件
 我们默认将所有的静态文件都放在一个static文件夹内
 我们需要自己在Django目录下创建static该文件夹
 static目录下基本还会再分几个文件夹
  static
  js
  css
  img
  第三方文件资源
# 在加载静态资源的时候没有开设对应的访问接口
Network:监控当前所有的网络请求状态

http://127.0.0.1:8000/static/bootstrap-3.4.1-dist/css/bootstrap.min.css  访问不到

2.配置
 settings.py 配置文件
    # 静态文件配置
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR,'static')
    ]
  <script src="/static/bootstrap-3.4.1-dist/js/bootstrap.min.js"></script>
3.进阶操作
 STATIC_URL = '/static/'  # 接口前缀
    """
    如果你想要访问静态文件资源，那么必须以static开头
     <script src="/static/bootstrap-3.4.1-dist/js/bootstrap.min.js"></script>
     你书写了接口前缀之后 就拥有了访问下列列表中所有文件夹内部资源的权限
    """

    # 静态文件配置
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR,'static')
    ]
4.动态解析
  {% load static %}
 <link rel="stylesheet" href="{% static 'bootstrap-3.4.1-dist/css/bootstrap.min.css' %}">
 <script src="{% static 'bootstrap-3.4.1-dist/js/bootstrap.min.js' %}"></script>


    3.全写:https://www.mzitu.com
            method
            get
            post
2.request对象方法
 request.method
  获取当前请求的请求方法并且结果是一个纯大写的字符串类型
 request.POST    # 直接看成是字典即可
  获取用户提交post请求过来的基本数据（不包含文件）
  get()  # 获取列表最后一个元素
  getlist()  # 获取整个列表
 request.GET  # 直接看成是字典即可
  获取url问号后面的数据
  get()  # 获取列表最后一个元素
  getlist()  # 获取整个列表
 request.FILES  # 直接看成是字典即可
  获取用户上传的文件数据
  """from表单如果需要携带文件数据 那么要添加参数
   <from action="" method="post" enctype="multipart/form-data"> 
  """
  get()  # 获取列表最后一个元素
  getlist()  # 获取整个列表
模板语法前端看不见只能在后端看见
  
      
"""
视图函数书写格式
 def login(request):
  if request.method == 'POST':
   return HttpResponse("我很气愤")
  return render(request,'login.html')
"""
```

### 补充

1.针对浏览器缓存我们可以修改为限制不适用缓存

<img src="C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\image-20210921211029311.png" alt="image-20210921211029311" style="zoom:80%;" />

> ###pycharm链接数据库

```python
DataBase工具栏
 下载对应的驱动即可
```

> ###django链接MySQL

```python
"""django默认使用自带的sqlite3"""
1.配置文件修改配置
 DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'db666',  # 一定要事先创建好才能指定
        'HOST': '127.0.0.1',
        'PORT': 3306,
        'USER': 'root',
        'PASSWORD': 'root',
        'CHARSET': 'UTF8'
    }
}
2.在项目文件夹或者应用文件夹内的__init__.py文件中书写固定的代码
 import pymysql
 pymysql.install_as_MySQLdb()
```

```python
django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module: No module named 'MySQLdb'.
Did you install mysqlclient or MySQL-python?
```

> ### django orm

```python
"""
orm:对象关系映射
"""

orm目的就是为了能够让不懂SQL语句的人通过python面向对象的知识点也能够轻松自如的操作数据库

面向对象的类     》》》    表
对象          》》》    表里面的数据
对象点属性      》》》    字段对应的值
# 缺陷: sql 封装死了 有时候查询速度很慢
```

> ###orm实操

```python
1.我们的模型类需要写在应用下的models.py文件中
class User(models.Model):
    # id int primary key auto_increment
    id = models.AutoField(primary_key=True)
    # name varchar(32)
    name = models.CharField(max_length=32)  # CharField必须要加max_length参数
    # age int
    age = models.IntegerField()

*************************************************************************************************
2.数据库迁移命令
 1.将数据库修改操作先记录到小本本上（对应应用下的migrations文件夹）
  python3 manage.py makemigrations
        
        D:\mysite1>python manage.py makemigrations
        Migrations for 'app01':
          app01\migrations\0001_initial.py
            - Create model User
            
 2.真正的执行数据库迁移操作
  python3 manage.py migrate
        
第一次执行数据库迁移命令Django默认需要创建依赖表
表名自动加前缀 目的是为了避免多个应用出现表名冲突的情况发生

 # 只要动了models.py中跟数据库相关的代码就必须重新执行上述两条命令
**************************************************************************************************

 3.针对主键字段
  class User1(models.Model):
            # 如果你不指定主键 那么orm会自动帮你创建一个名为id的主键字段
            # 如果你想让主键字段名不叫id 叫uid、sid、pid等需要自己手动指定
            username = models.CharField(max_length=32)
 
```

<img src="C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\image-20210923225529587.png" alt="image-20210923225529587" style="zoom:50%;" />

> ###字段的增删改查

```python
# 增
pwd = models.IntegerField('密码', null=True)  # 该字段可以为空
is_delete = models.IntegerField(default=0)  # 默认值
# 改
直接改代码然后执行数据库迁移命令即可
# 删
注释掉代码然后执行数据库迁移命令即可
```

> ### 数据的增删改查

```python
# 1.查询数据
# select * from user where name=username and pwd=password;
# res = models.User.objects.filter(name=username)  # <QuerySet [<User: User object>]>  列表套数据对象
user_obj = models.User.objects.filter(name=username,pwd=password).first()  
# <QuerySet [<User: User object>]>  列表套数据对象
# user_obj = res[0]

# 2.添加数据
# insert into user(name,pwd) values(username,password);
models.User.objects.create(name=username,pwd=password)

# 3.查询所有的数据
# select * from user;  作了分页处理
user_data = models.User.objects.all()  # [obj1,obj2,obj3,obj4]
return render(request, 'home.html',{'user_data':user_data})

# 4.修改数据
models.User.objects.filter(id=edit_id).update(name=username,pwd=password)
# 第二种修改方式
edit_obj.name = username
edit_obj.pwd = password
edit_obj.save()

# 5.删除数据
models.User.objects.filter(id=delete_id).delete()
```

越往后学越简单 只不过过程是复杂的

> ###作业

1. 整理今日内容笔记
2. 自己尝试搭建一个增删改查页面

## 昨日内容回顾

- 静态文件配置

  ```python
  # 1.静态文件
   网站所使用的已经提前写好的文件
    css文件
    js文件
    img文件
     第三方文件
   我们在存储静态文件资源的时候一般默认都是放在static文件夹下
  # 2.Django静态文件配置
   settings.py
    STATICFILES_DIRS = [
              os.path.join(BASE_DIR,'static')
              os.path.join(BASE_DIR,'static')
              os.path.join(BASE_DIR,'static')
          ]
  # 3.接口前缀
   STATIC_URL = '/static/'
  # 4.动态匹配
   {% load static %}
   {% static 'a.txt' %}
  ```

- request对象方法

  ```python
  request.method  # 获取请求方式
   纯大写的字符串类型
  request.POST  # 获取post请求提交的普通数据
   可以看成是一个字典
   .get()
   .getlist()
  request.GET  # 获取url问号后面携带的参数
   .get()
   .getlist()
  request.body  # 原始的二进制数据
      
  ```

- pycharm如何链接数据库

  ```python
  1.DataBase
  2.选择相应的数据库
  3.第一次连接一定要下载对应的驱动
   可能存在多个驱动，挨个尝试即可
  ```

- Django如何指定数据库

  ```python
  1.setting文件
   DATABASES = {
          'DEFAULT':{
              
          }
      }
  2.项目文件夹或者应用文件夹下的__init__文件
   import pymysql
   pymysql.install_as_MySQLdb()
  ```

- Django orm操作

  ```python
  """
  ORM    对象关系映射
  类    表
  对象    一条条记录
  属性    字段对应的值
  """
  能够让不会SQL的python程序员，通过面向对象的知识也能够简单快捷的操作数据库
  
  # 1.models.py
  class Userinfo(models.Model):
      # 主键字段不指定则默认添加一个名为id的主键字段
      username = models.CharField(max_length=32,verbose_name='用户名')
  # 2.数据库迁移命令
   python3 manage.py makemigrations
   python3 manage.py migrate
  """往后只要在models.py中修改了跟模型表相关的代码就必须重新执行"""
  # 3.CURD操作
   字段
    null=True
    default=''
   数据
    models.Userinfo.objects.filter(**kwargs)
     结果暂且可以看成是列表套数据对象
     .first()
    models.Useringo.objects.all()
     结果暂且可以看成是列表套数据对象
    models.Userinfo.objects.create(**kwargs)
     添加数据
    models.Userinfo.objects.filter(**kwargs).update(**kwargs)
     修改数据
    models.Userinfo.objects.filter(**kwargs).delete()
     删除数据
          
  ```

## 今日内容概要

- 数据库同步命令(了解)

- orm创建外键关系

- Django请求生命周期流程图

- 分块具体学习Django所有的功能

  路由层(urls.py)

- 路由匹配

- 无名有名分组

- 反向解析

- 路由分发

- 名称空间

## 今日内容详细

> ###数据库同步命令（了解）

```python
"""
数据库里面已经有一些表，我们如何通过Django orm 操作？
 1.照着数据库表字段自己在models.py
  数据需要自己二次同步
 2.Django提供的反向同步
"""
1.先执行数据库迁移命令 完成链接
 python manage.py makemigrations
2.查看代码
 python manage.py inspectdb 表名
inspectdb userinfo
复制 到models.py文件中
 class Userinfo(models.Model):
    id = models.IntegerField(blank=True, null=True)
    name = models.CharField(max_length=32, blank=True, null=True)
    pwd = models.IntegerField(blank=True, null=True)

    class Meta:
        managed = False
        db_table = 'userinfo'
```

> ### orm创建外键关系

```python
"""
1. 表与表之间的关系
 一对多(多对一)
 一对一
 多对多
2. 表关系的判断
 换位思考
"""
书籍表

出版社表

作者表

# 针对外键子弹的创建位置
 一对多
   推荐建在多的一方
 一对一
  建在任何一方都可以 但是推荐建在查询频率较高的表中
 多对多
  1.自己建表
  2.建在任何一方都可以 但是推荐建在查询频率较高的表中
```

切忌眼高手低 一个代码敲一百遍不为过 一个题目刷一百遍也不为过 关键是有没有理解和掌握

> ###Django请求生命周期流程图

模板渲染 模板语法

<img src="C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\image-20210925140226316.png" alt="image-20210925140226316" style="zoom:50%;" />

> ### 路由层之路由匹配

```python
"""路由你可以看成就是出去ip和port之后的地址"""
url()方法
 1.第一个参数其实是一个正则表达式
 2.一旦第一个参数匹配到了内容直接结束匹配 执行对应的视图函数
url(r'^test/$',views.test),
```

> #### 无名分组

```python
url(r'^test/\d+/$',views.test),
# 正则表达式分组：给正则表达式前后加一个小括号
url(r'^test/(\d+)/$',views.test),
"""
无名分组
 将括号内正则表达式匹配到的内容当做位置参数传递给后面的视图函数
"""
```

> ####有名分组

正则表达式可以起别名

```python
 url(r'^testadd/(?P<id>\d+)/$',views.testadd),
"""
有名分组
 将括号内正则表达式匹配到的内容当做关键字参数传递给后面的视图函数
"""
```

> ####是否可以结合使用

```python
url(r'^test1/(\d+)/(?P<id>\d+)/$',views.test1),
1.无名有名分组不能混合使用

url(r'^test2/(\d+)/(\d+)/$',views.test2),
url(r'^test3/(?P<id>\d+)/(?P<id1>\d+)/$',views.test3),
2.可以单个重复使用
```

> ###反向解析

```python
当路由频繁变化的时候，HTML界面上的链接地址如何做到动态解析
# 1.给路由与视图函数对应关系添加一个别名（名字自己指定 只有不冲突即可）
url(r'^index/', views.index,name='index_name'),
# 2.根据该别名动态解析出一个结果，该结果可以直接访问到对应的路由
 前端
  <a href="{% url 'index_name' %}">111</a>
 后端
  from django.shortcuts import reverse
  reverse('index_name')
 ps:redirect括号内也可以直接写别名
```

> ####无名有名反向解析

```python
url(r'^index/(\d+)/', views.index,name='index_name'),
后端
 reverse('index_name',args=(1,)) # args一般放数据的主键值 只要给个数字即可
前端
 <a href="{% url 'index_name' 1 %}">111</a>  # 只要给个数字即可

    
url(r'^index/(?P<id>\d+)/', views.index,name='index_name'),
后端
 reverse('index_name',kwargs={'id':123}) # 只要给个数字即可
前端
 <a href="{% url 'index_name' id=666 %}">4444</a>  # 只要给个数字即可
    
总结
 无名有名都可以使用一种(无名）反向解析的形式
```

> ###作业

1.整理今日笔记

2.使用无名有名分组 与 反向解析 完成昨天的用户增删改查的功能

​ 一定要尝试着自己写

## 昨日内容回顾

- 数据库同步命令

  ```python
  inspectdb
  ```

- Django请求生命周期流程图

  ```python
  1.web服务网关接口
   wsgiref
   uwsgi
  2.灰色地带（Django中间件）
  3.路由层
  4.视图层
  5.模板层
  6.模型层
  ```

- 路由匹配

  ```python
  1.自动补全斜杠
   APPEND_SLASH = True
  2.url()方法
   第一个参数是正则表达式
  # 路由匹配特性：一旦正则能够匹配到内容则停止继续往下而是直接执行对应的功能
  ```

- 无名有名分组

  ```python
  无名分组
   给正则表达式加上一个括号
   url(r'index/(\d+)/',views.index)
   执行视图函数的时候会将括号内匹配到的内容当做位置参数传递给视图函数
  有名分组
   给正则表达式加上一个括号并且起一个别名
   url(r'index/(?P<id>\d+)/',views.index)
   执行视图函数的时候会将括号内匹配的内容当做关键字参数传递给视图函数
  ps:两者不能混合使用，但是单独可以重复使用
  ```

- 反向解析

  ```python
  """通过别名得到一个可以访问该别名对应的路由规则"""
  1.起别名（不能冲突）
   url(r'index/',views.index,name='index_name')
  2.反向解析
   前端
    {% url 'index_name' %}
   后端
    from django.shortcuts import reverse
    url = reverse('index_name')
  ```

- 无名有名反向解析

  ```python
  """当路由出现无名有名分组反向解析需要传递额外的参数"""
  url(r'index/(\d+)/',views.index,name='index_name')
   前端
    {% url 'index_name' 1 %}
   后端
    from django.shortcuts import reverse
    url = reverse('index_name',args=(1,))
  ps:有名分组的反向解析也可以使用无名的方式
   # 了解
   url(r'index/(?P<id>\d+)/',views.index,name='index_name')
   前端
    {% url 'index_name' 1 %}
   后端
    from django.shortcuts import reverse
    url = reverse('index_name',kwargs={'id':1})
  ```

## 今日内容概要

- 路由分发
- 名称空间
- 伪静态(了解)
- 本地虚拟环境
- Django版本区别
- 视图层
  - 三板斧
  - JsonResponse
  - 上传文件
  - FBV与CBV(重点)
  - CBV源码剖析(重点)

## 今日内容详细

> ###路由分发

```python
"""
简介
 Django是专注于开发应用的，当一个Django项目特别庞大的时候
 所有的路由与视图函数映射关系全部写在总的urls.py很明显太冗余不便于管理
 
 其实Django中的每一个应用都可以有自己的urls.py,static文件夹,templates文件夹。
 基于上述特点，使用Django做分组开发非常的简便
 每个人只需要写自己的应用即可
 最后由组长统一汇总到一个空的Django项目中国然后使用路由分发将多个应用关联到一起
"""
复杂版本
from app01 import urls as app01_urls
from app02 import urls as app02_urls
# 路由分发 复杂版本
url(r'^app01/',include(app01_urls))
url(r'^app02/',include(app02_urls))
"""总路由最后千万不能加$"""
# 进阶版本
url(r'^app01/',include('app01.urls')),
url(r'^app02/',include('app02.urls')),
```

> ###名称空间

```python
"""
当多个应用在反向解析的时候如果出现了别名冲突的情况，那么无法自动识别
"""
解决方式1 >>> 名称空间
 总路由
 url(r'^app01/',include('app01.urls',namespace='app01'))
 url(r'^app02/',include('app02.urls',namespace='app02'))

 reverse('app01:index_name')
 reverse('app02:index_name')
    
 <a href="{% url 'app01:index_name' %}">app01</a>
 <a href="{% url 'app02:index_name' %}">app02</a>
解决方式2 >>> 别名不能冲突（加上自己应用名作为前缀）
 url(r'^index',views.index,name='app01_index_name')
 url(r'^index',views.index,name='app02_index_name')
```

> ###伪静态

```python
动静态网页

将url地址模拟成html结尾的样子，看上去像是一个静态文件、
目的是为了增加搜索引擎收藏我们网站的概率以及seo查询几率
ps:再怎么优化都不如RMB玩家!!!
```

> ### 本地虚拟环境

```python
"""
在项目开发过程中，我们会给不同的项目配备不同的环境
项目用到什么就装什么，用不到的一概不装
不同的项目解释器环境都不一样
"""
requirements.txt 项目所有的模块以及模块所对应的版本
很多 Python 项目中经常会包含一个 requirements.txt 文件，里面内容是项目的依赖包及其对应版本号的信息列表，即项目依赖关系清单，其作用是用来重新构建项目所需要的运行环境依赖，比如你从 GitHub 上 clone 了一个 Python 项目，通常你会先找到 requirements.txt 文件，然后运行命令 pip install -r requirements.txt 来安装该项目所依赖的包。

同样，你也可以在你的项目目录下运行命令 pip freeze > requirements.txt 来生成 requirements.txt 文件，以便他人重新安装项目所依赖的包。

创建虚拟环境类似于你重新下载了一个纯净的python解释器
 如果反复创建类似于反复下载，会消耗一定的硬盘空间
ps:我们目前不推荐你使用虚拟环境，所有的模块统一下载到本地

```

> ###django版本区别

```python
django1.X与2.X、3.X

1.区别
urls.py中的路由匹配方法
 1.X第一个参数正则表达式
  url()
 2.X和3.X第一个参数不支持正则表达式，写什么就匹配什么  re_path
  path()
如果想要使用正则，那么2.X与3.X也有响应的方法
 from django.urls import path,re_path
 re_path 等价于 1.X里面的url方法

2.转换器
  五种常用转换器（Path converters）
  str,匹配除了路径分隔符（/）之外的非空字符串，这是默认的形式
    int,匹配正整数，包含0。
    slug,匹配字母、数字以及横杠、下划线组成的字符串。
    uuid,匹配格式化的uuid，如 075194d3-6885-417e-a8a8-6c931e272f00。
    path,匹配任何非空字符串，包含了路径分隔符（/）（不能用？）
   
自定义
regex 类属性，字符串类型

to_python(self, value) 方法，value是由类属性 regex 所匹配到的字符串，返回具体的Python变量值，以供Django传递到对应的视图函数中。

to_url(self, value) 方法，和 to_python 相反，value是一个具体的Python变量值，返回其字符串，通常用于url反向引用。

自定义转换器示例：

1.在app01下新建文件path_ converters.py,文件名可以随意命名

class MonthConverter:
    regex='\d{2}' # 属性名必须为regex

    def to_python(self, value):
        return int(value)

    def to_url(self, value):
        return value # 匹配的regex是两个数字，返回的结果也必须是两个数字
    
2.在urls.py中，使用register_converter 将其注册到URL配置中：
from django.urls import path,register_converter
from app01.path_converts import MonthConverter

register_converter(MonthConverter,'mon')

from app01 import views


urlpatterns = [
    path('articles/<int:year>/<mon:month>/<slug:other>/', views.article_detail, name='aaa'),

]

3.views.py中的视图函数article_detail

from django.shortcuts import render,HttpResponse,reverse

def article_detail(request,year,month,other):
    print(year,type(year))
    print(month,type(month))
    print(other,type(other))
    print(reverse('xxx',args=(1988,12,'hello'))) # 反向解析结果/articles/1988/12/hello/
    return HttpResponse('xxxx')
4.测试
# 1、在浏览器输入http://127.0.0.1:8000/articles/2009/12/hello/，path会成功匹配出参数year=2009,month=12,other='hello'传递给函数article_detail
# 2、在浏览器输入http://127.0.0.1:8000/articles/2009/123/hello/，path会匹配失败，因为我们自定义的转换器mon只匹配两位数字，而对应位置的123超过了2位
```

> ### HttpResponse、render、redirect三板斧本质

```python
django视图函数必须要返回一个HttpResponse对象

def render(request, template_name, context=None, content_type=None, status=None, using=None):
    """
    Return a HttpResponse whose content is filled with the result of calling
    django.template.loader.render_to_string() with the passed arguments.
    """
    content = loader.render_to_string(template_name, context, request, using=using)
    return HttpResponse(content, content_type, status)


class HttpResponse(HttpResponseBase):
    """
    An HTTP response class with a string as content.

    This content can be read, appended to, or replaced.
    """

    streaming = False

    def __init__(self, content=b'', *args, **kwargs):
        super().__init__(*args, **kwargs)
        # Content is a bytestring. See the `content` property methods.
        self.content = content

    def __repr__(self):
        return '<%(cls)s status_code=%(status_code)d%(content_type)s>' % {
            'cls': self.__class__.__name__,
            'status_code': self.status_code,
            'content_type': self._content_type_for_repr,
        }

    def serialize(self):
        """Full HTTP message, including headers, as a bytestring."""
        return self.serialize_headers() + b'\r\n\r\n' + self.content

    __bytes__ = serialize

    @property
    def content(self):
        return b''.join(self._container)

    @content.setter
    def content(self, value):
        # Consume iterators upon assignment to allow repeated iteration.
        if (
            hasattr(value, '__iter__') and
            not isinstance(value, (bytes, memoryview, str))
        ):
            content = b''.join(self.make_bytes(chunk) for chunk in value)
            if hasattr(value, 'close'):
                try:
                    value.close()
                except Exception:
                    pass
        else:
            content = self.make_bytes(value)
        # Create a list of properly encoded bytestrings to support write().
        self._container = [content]

    def __iter__(self):
        return iter(self._container)

    def write(self, content):
        self._container.append(self.make_bytes(content))

    def tell(self):
        return len(self.content)

    def getvalue(self):
        return self.content

    def writable(self):
        return True

    def writelines(self, lines):
        for line in lines:
            self.write(line)
            
redirect内部是继承了HttpResponse类

```

> ###JsonResponse

```python
需求：给前端返回json格式数据
方式1：自己序列化
  # res = json.dumps(d,ensure_ascii=False)
    # return HttpResponse(res)
方式2：JsonResponse
    import json
    from django.http import JsonResponse
    def func2(request):
        d = {'user':'jason好帅','password':123}
        return JsonResponse(d)
```

ps:额外参数补充

```python
json_dumps_params={'ensure_ascii':False}  # 看源码
safe=False  # 看报错信息

# JsonResponse返回的也是HttpResponse对象
```

> ###上传文件

```python
form表单上传文件注意事项：
 1.method必须是post
 2.enctype参数修改为multipart/form-data
# 错误1  
 CSRF verification failed. Request aborted.
# 解决错误 
 注释掉 settings.py中的MIDDLEWARE   # 'django.middleware.csrf.CsrfViewMiddleware',
    
def func1(request):
    if request.method == 'POST':
        print(request.POST)  # <QueryDict: {'username': ['jason']}>
        file_obj = request.FILES.get('myfile')
        print(file_obj.name)  # 获取文件名称
        with open(r'%s'%file_obj.name,'wb') as f:
            # for line in file_obj:
            #     f.write(line)
            for chunks in file_obj.chunks():
                f.write(chunks)
    return render(request,'func1.html')
```

> ###FBV与CBV

```python
FBV
 基于函数的视图
CBV
 基于类的视图
    
# 基本使用
from django.views import View


class MyView(View):
    def get(self,request):
        return HttpResponse('get方法')
    def post(self,request):
        return HttpResponse("post方法")

# CBV路由配置
    path('func4/', views.MyView.as_view()),
"""为什么能够自动根据请求方法的不同执行不同的方法"""
1.突破口
 as_view()
2.CBV与FBV路由匹配本质
  path('func4/', views.MyView.as_view()),
    # 等价  CBV路由配置本质跟FBV一样
    #  path('func4/', views.view)
    
['get', 'post', 'put', 'patch', 'delete', 'head', 'options', 'trace']
3.源码

 @classonlymethod
    def as_view(cls, **initkwargs):
        """Main entry point for a request-response process."""
        for key in initkwargs:
            if key in cls.http_method_names:
                raise TypeError(
                    'The method name %s is not accepted as a keyword argument '
                    'to %s().' % (key, cls.__name__)
                )
            if not hasattr(cls, key):
                raise TypeError("%s() received an invalid keyword %r. as_view "
                                "only accepts arguments that are already "
                                "attributes of the class." % (cls.__name__, key))

        def view(request, *args, **kwargs):
            self = cls(**initkwargs)  # self = MyView()  生成一个我们自己写的类的对象
            self.setup(request, *args, **kwargs)  # 给对象赋值属性
            if not hasattr(self, 'request'):
                raise AttributeError(
                    "%s instance has no 'request' attribute. Did you override "
                    "setup() and forget to call super()?" % cls.__name__
                )
            return self.dispatch(request, *args, **kwargs)
        view.view_class = cls
        view.view_initkwargs = initkwargs

        # take name and docstring from class
        update_wrapper(view, cls, updated=())

        # and possible attributes set by decorators
        # like csrf_exempt from dispatch
        update_wrapper(view, cls.dispatch, assigned=())
        return view
    
    
    def dispatch(self, request, *args, **kwargs):
        # Try to dispatch to the right method; if a method doesn't exist,
        # defer to the error handler. Also defer to the error handler if the
        # request method isn't on the approved list.
        # 获取当前请求并判断是否属于正常的请求(8个)
        if request.method.lower() in self.http_method_names:
            # 例如： get请求 getattr(自己创建的对象,'get')  handler = 我们自己写的get方法
            handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
        else:
            handler = self.http_method_not_allowed
        return handler(request, *args, **kwargs)  # 执行我们写的get方法并返回该方法的返回值
```

## 昨日内容回顾

- 路由分发

  ```python
  在Django中所有的应用都可以有自己独立的路由层、模板层、静态文件
  
  路由分发是为了解决总路由代码过于冗余的情况
  
  include('应用名.urls')
  
  注意事项
   总路由最后千万不要加$ (1.X版本)
  ```

- 名称空间

  ```python
  当多个应用出现反向解析起别名冲突的情况
  include('应用名.urls',namespace='自定义名称空间名字')
  reverse('自定义名称空间名字:别名')
  {% url '自定义名称空间名字:别名' %}
  
  # 名称空间其实也可以不使用 只要我们做到所有的别名不冲突即可。
  # eg:在每个别名前面加上各自应用名作为前缀
  ```

- 本地虚拟环境

  ```python
  类似于docker容器，给每个项目配备独立的运行环境
  ps:requirements.txt
  
  创建虚拟环境类似于重新下载了一个纯净的python解释器
  ```

- Django版本区别

  ```python
  1.路由层使用的方法不一样
   url()
   path()
    re_path() == url()
  2.转换器
  3.2.X与3.X区别
   3.X新增了异步功能
  ```

- 三板斧本质

  ```python
  返回HttpResponse对象
  ```

- JsonResponse

  ```python
  JsonResponse(obj.json_dumps_params,safe)
  ps:看源码学习的好处
  ```

- FBV与CBV及源码

  ```python
  FBV
   def index(request):
          return HttpResponse('index')
  CBV
   from django.views import View
   class MyClassView(View):
          def get(self,request):
              return HttpResponse('get')
          def post(self,request):
              return HttpResponse('post')
   url(r'^login/',views.MyClassView.as_view())
  # 为什么能够根据前期方式的不同自动匹配执行对应的方法
  class View(...):
      @classonlymethod
      def as_view(...):
          def view(...):
              obj = cls(...)
              return self.dispatch(...) # 面向对象属性查找顺序
          return view  # FBV与CBV路由其实是一样的
      def dispatch(...):
          if request.method.lower() in [8个方法]:
              handler = getattr(self.request.method.lower(),'报错')
          else:
              handler = '报错'
          return handler(...)
  ```

## 今日内容概要

- Django settings 源码
- 模板语法传值
- 模板语法之过滤器
- 模板语法之标签
- 自定义过滤器、标签、inclusion_tag
- 模板的继承
- 模板的导入
- 模型层(单表查询关键字)

## 今日内容详细

> ### Django settings 源码

```python
"""
1.django其实有两个配置文件
    一个是暴露给用户可以自定义的配置文件
  项目根目录下的settings.py
    一个是项目默认的配置文件
  当用户不做任何配置的时候自动加载默认配置
2.配置文件变量名必须是大写
"""
from django.conf import global_settings 
查看源码
global_settings.py
LANGUAGE_CODE = 'en-us'
 ('zh-hans', gettext_noop('Simplified Chinese'))
疑问：为什么当用户配置了就使用用户配置的 不配置就使用默认的
from django.conf import settings

settings = LazySettings()
manage.py文件
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'djangoProject.settings')
ENVIRONMENT_VARIABLE = "DJANGO_SETTINGS_MODULE"
class LazySettings(LazyObject):
    def _setup(self, name=None):
        # os.environ看成是一个全局大字典  'djangoProject.settings'
        settings_module = os.environ.get(ENVIRONMENT_VARIABLE)
        self._wrapped = Settings(settings_module)  # Settings('djangoProject.settings')

class Settings:
    def __init__(self, settings_module):  # 'djangoProject.settings'
        # update this dict from global settings (but only for ALL_CAPS settings)
        for setting in dir(global_settings):  # 获取全局配置文件里面所有的（大写）变量名
            if setting.isupper():  # 校验判断变量名是否是纯大写
                setattr(self, setting, getattr(global_settings, setting))
                # 给Settings对象添加全局配置文件中所有的配置信息
                # 例如 self.NAME = 'jason'
                # getattr(global_settings, setting) 获取全局设置文件中大写变量名对应的值
                # 大写变量名为健，对应的值为值
        # store the settings module in case someone later cares
        self.SETTINGS_MODULE = settings_module  # 'djangoProject.settings'

        mod = importlib.import_module(self.SETTINGS_MODULE)
        # from djangoProject import settings  # 导入暴露给用户的自定义配置文件
        # importlib.import_module 传入一个字符串（项目名字是动态变化的）可以导入相关模块
        for setting in dir(mod):
            if setting.isupper():
                setting_value = getattr(mod, setting)
                setattr(self, setting, setting_value)
```

单例模式  反射

功能的可插拔式配置

```python
# import os
# print(os.environ)  # 固定的全局大字典

# d = {}
# d['name'] = 'jason'
# d.setdefault('xxx','lili')
# print(d)
# print(dir(d))

import importlib # 传入一个字符串可以导入
```

> ###模板语法之传值

```python
 # 传值方式1:利用字典挨个传值
return render(request,'index.html',{'i':i,'f':f,'s':s,'l':l,'d':d,'t':t,'se':se,'b':b})
# 传值方式2:简单粗暴  locals()将当前名称空间中所有的变量名全部传递给页面
return render(request,'index.html',locals())

"""
    传值方式1    传值精确  不会造成资源浪费
    传值方式2    传值粗糙  可能会造成一定的资源浪费
    ps:为了教学方便  我们以后就是用locals()
"""
补充:传递函数名和类名都会自动加括号调用(模板语法不支持额外的传参)
```

> ###模板语法之获取值

```python
"""模板语法取值只能采用  句点符(.)"""
索引 健都可以无限制的点点点
<p>{{ d.hobby.3.username }}</p>
```

> ###模板语法之过滤器

```python
# 类似于python的内置方法
<p>过滤器:将竖杠左侧的数据当做第一个参数</p>
<p>统计长度:{{ s|length }}</p>
<p>加法运算:{{ i|add:100 }}</p>
<p>字符串拼接:{{ s|add:'love' }}</p>
<p>日期格式:{{ ctime|date:'Y年-m月-d日' }}</p>
<p>默认值:{{ b|default:'中国' }}</p>
<p>默认值:{{ b1|default:'中国' }}</p>
<p>文件大小:{{ file_size|filesizeformat }}</p>
<p>截取文本(引号也算 按字符截取）:{{ s|truncatechars:3 }}</p>
<p>截取文本(引号不算 按空格截取）:{{ s|truncatewords:3 }}</p>
<p>前端取消转义:{{ h|safe }}</p>
<p>{{ sss }}</p>

转义
 前端
  <p>前端取消转义:{{ h|safe }}</p>
 后端
  # 后端取消转义
     from django.utils.safestring import mark_safe
     sss = mark_safe('<h2>学而不思则罔</h2>')
ps:前端代码也可以在后端写好传入!!!
```

> ###模板语法之标签

```python
# 类似于python的流程控制
{% if b1 %}
    <p>有值</p>
{% else %}
    <p>无值</p>
{% endif %} 

{% for foo in l %}
{#    <p>内置对象:{{ forloop }}</p>#}
    <p>{{ foo }}</p>
{% endfor %}
    
{% for foo in s %}
    {% if forloop.first %}
        <p>这是我的第一次</p>
    {% elif forloop.last %}
        <p>这是我的最后一次</p>
    {% else %}
        <p>{{ foo }}</p>
    {% endif %}
    {% empty %}
        <p>传入的数据是空的 不存在</p>
{% endfor %}
    
"""
{{}}    变量相关
{%%}    逻辑相关
"""
    
# 了解
{% with d.username as name %}
    {{ name }}
    {{ name }}
    {{ d.username }}
{% endwith %}
```

> ###自定义过滤器、标签、inclusion_tag

```python
# 类似于python里面的自定义函数
1.在应用下创建一个名字必须叫"templatetags"文件夹
2.在上述文件夹内创建一个任意名称的py文件
3.在该py文件内固定先书写以下两句话
 from django import template
 register = template.Library()
    
  
app01\templatetags\mytag.py
from django import template
register = template.Library()
# 自定义过滤器
@register.filter(name='myfilter')
def index(a,b):
    # 简单的加法运算
    return a + b

index.html
{% load mytag %}
{{ i|myfilter:666 }}

# 自定义标签
app01\templatetags\mytag.py
@register.simple_tag(name='mysimple')
def func1(a,b,c,d):
    return '%s-%s|%s?%s'%(a,b,c,d)

index.html
{% load mytag %}
{% mysimple 1 'jason' 222 'egon' %}


inclusion_tag
 当某个区域需要反复使用并且数据不是固定的
    
# 自定义inclusion_tag
app01\templatetags\mytag.py
@register.inclusion_tag('login.html',name='my_inclusion_tag')
def func2(n):
    l = []
    for i in range(1,n+1):
        l.append('第%s页'%i)
    return locals()

index.html
{% load mytag %}
{% my_inclusion_tag 10 %}

djangoProject\templates\login.html
<ul>
    {% for foo in l %}
        <li>{{ foo }}</li>
    {% endfor %}
</ul>
```

> ###模板的导入

```pyhon
# 类似于后端导入模块 想要什么局部页面直接导入即可
{% include 'myform.html' %}
```

你学会了什么都不重要，你学不会什么都重要

> ### 模板的继承

```python
先使用block划定区域
母版
 {% block 区域名称 %}
 {% endblock %}
子版
 {% extends 'home.html'%}
 {% block 区域名称 %}
 {% endblock %}

实例：
{% extends 'home.html'%}

{% block content %}
    <h1 class="text-center">注册</h1>
    <form action="">
        <p><input type="text" class="form-control"></p>
        <p><input type="text" class="form-control"></p>
        <input type="submit" class="btn btn-danger btn-block">
    </form>
{% endblock %}

母版在划定区域的时候一般都应该由三个区域
 css区域
 html文档区域
 js区域
 ps:目的是为了让子版具有独立的css js等 增加扩展性
 {% block css %}
        
    {% endblock %}
    
 {% block content %}
        
    {% endblock %}
    
 {% block js %}
        
    {% endblock %}
    
ps:子版也可以继续使用母版划定区域内的内容
 {{ block.super }}
```

## 昨日内容回顾

- settings源码

  ```python
  熟读并表述出来
  ```

- 模板语法传值

  ```python
  1.字典
  2.locals()
  
  1.传函数名和类名的时候都会自动加括号调用
  2.不支持传递额外的参数
  ```

- 过滤器

  ```python
  |add
  |length
  |default
  |date
  |truncatewords
  |truncatechars
  |filesizeformat
  |safe
  ```

- 标签

  ```python
  for循环
   forloop
   empty
  if判断
  
  with起别名
  
  """
  {{}}  变量相关
  {%%}  逻辑相关
  """
  ```

- 自定义过滤器、标签、inclusion_tag

  ```python
  1.templatetags文件夹
  2.任何名称py文件
  3.固定写两句话
  ```

- 模板导入

  ```python
  {% include '模板名称' %}
  ```

- 模板继承

  ```python
  {% extends '母版名称' %}
  
  {% block '区域名称' %}
  {% endblock %}
  
  # 母版最好有三个及以上区域
   css
   content
   js
  ```

## 今日内容概要

- Django测试环境搭建
- 单表查询关键字
- 神奇的双下划线查询
- 图书管理系统表设计
- 外键字段的增删改查
- 基于对象的跨表查询(子查询)
- 基于下划线的跨表查询(连表查询)
- 聚合查询
- 分组查询
- F与Q查询

## 今日内容详细

> ###Django测试环境搭建

```python
ps:
    1.pycharm连接数据库都需要提前下载对应的驱动
    2.自带的sqlite3对日期格式数据不敏感
     如果后续业务需要使用日期辅助筛选数据那么不推荐使用sqlite3
        
方式1: 任意创建一个py文件，在该文件内书写固定的配置
import os
if __name__ == '__main__':
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'djangoProject2.settings')
    import django
    django.setup()
    # 单独测试Django py文件
    
方式2: 直接使用pycharm提供的python console
    
```

> ###单表查询关键字

```python
QuerySet对象方法
 query  # 查看orm内部对应的SQL语句
    
   from app01 import models
    # res = models.Books.objects.all()
    # print(res)  # <QuerySet []>
    
    # 增
    # res = models.Books.objects.create(title='三国演义',price=234.43)
    # res = models.Books.objects.create(title='水浒传',price=456.22)
    # res = models.Books.objects.create(title='西游记',price=687.90)
    # print(res.title)
    '''create返回值就是当前被创建的数据对象'''
    # 查
    # res = models.Books.objects.all()
    # print(res.query)
    # SELECT `app01_books`.`id`, `app01_books`.`title`, `app01_books`.`price`, `app01_books`.`publish_time` FROM `app01_books`
    # res = models.Books.objects.filter()  # QuerySet
    # print(res.query)
    # res = models.Books.objects.filter(title='西游记',price=123.23)  # AND
    # res = models.Books.objects.filter(title='jason')
    # res = models.Books.objects.filter(title='西游记')
    # res = models.Books.objects.filter(title='西游记')[0]
    # print(res)  # <QuerySet [<Books: 西游记>]>
    """filter括号内可以放多个参数 默认是and关系 返回一个QuerySet对象 条件不符合不会报错返回QuerySet空对象 可以通过索引提取对象本身"""
    # res1 = models.Books.objects.get(title='西游记')
    # print(res1)  # 西游记
    """get括号内可以放多个参数 默认是and关系 直接拿到对象本身 条件不符合直接报错 不推荐使用"""
    # 改
    # res = models.Books.objects.filter(pk=1).update(price=666.66)
    # print(res)  # 返回值是受影响的行数

    # book_obj = models.Books.objects.filter(pk=2).first()
    # book_obj.price = 999.66
    # book_obj.save()  # 效率低(所有字段重新写一遍)
    """pk能够自动查找到当前表的主键字段 我们不需要查看当前表主键字段名"""
    # 删
    # models.Books.objects.filter(pk=1).delete()
    
    # 1.first()  取第一个数据对象
    # res = models.Books.objects.all().first()
    # res1 = models.Books.objects.all()[0]
    # res2 = models.Books.objects.all()[0:2]
    # print(res,res1,res2)
    """QuerySet对象还支持切片操作 但是只能填正数"""
    # 2.last()  取最后一个数据对象
    # res = models.Books.objects.all().last()
    # print(res)
    # 3.values()  获取数据指定字段的值
    # res = models.Books.objects.all().values('title','publish_time')
    # res = models.Books.objects.values('title','publish_time')
    # print(res.query)
    """all()加不加都表示所有数据 values获取的结果 类似于列表套字典"""
    # 4.values_list()  获取数据指定字段的值
    # res = models.Books.objects.values_list('title','publish_time')
    # print(res)
    """values_list获取的结果 类似于列表套元组"""
    # 5.order_by()  排序
    # res = models.Books.objects.order_by('price')  # 默认是升序
    # print(res)
    # res1 = models.Books.objects.order_by('-price')  # 减号降序
    # print(res1)
    # 6.count()  计数
    # res = models.Books.objects.count()  # 统计数据条数
    # print(res)
    # 7.distinct()  去重
    # res = models.Books.objects.distinct()
    # print(res)
    # res = models.Books.objects.values('title').distinct()
    # print(res)
    """去重的前提是数据必须是一模一样  一定不能忽略主键"""
    # 8.exclude()  排除什么什么在外  取反操作
    # res = models.Books.objects.exclude(title='西游记')
    # print(res.query)
    # 9.reverse()
    # res = models.Books.objects.all()
    # res1 = models.Books.objects.reverse()
    # res2 = models.Books.objects.order_by('price').reverse()
    # print(res)
    # print(res1)
    # print(res2)
    """reverse需要先排序之后才能反转"""
    # 10.exists()  判断是否有数据  返回布尔值
    # res = models.Books.objects.all().exists()
    # print(res)
```

> ###神奇的双下划线查询(范围查询)

```python
    # 1.查询价格大于700的书籍
    # res = models.Books.objects.filter(price__gt=700)  # WHERE `app01_books`.`price` > 700
    # print(res)
    # print(res.query)
    # 2.查询价格小于700的书籍
    # res = models.Books.objects.filter(price__lt=700)
    # print(res)
    # 查询价格大于等于700 小于等于700
    # res1 = models.Books.objects.filter(price__gte=700)  # WHERE `app01_books`.`price` >= 700
    # res2 = models.Books.objects.filter(price__lte=700)  # WHERE `app01_books`.`price` <= 700
    # print(res1.query,res2.query)
    # 3.查询价格要么是666.66 要么是999.66 要么是10000
    # res = models.Books.objects.filter(price__in=[666.66,999.66,10000])
    # print(res)
    """python对数字不是很敏感 精确度不高 很多时候我们会采取字符串存储数字类型"""
    # 4.查询价格在500到800之间的
    # res = models.Books.objects.filter(price__range=(500,800))  # WHERE `app01_books`.`price` BETWEEN 500 AND 800
    # print(res.query)

    # 5.查询书名中包含字母s的书
    # res = models.Books.objects.filter(title__contains='s')  # 区分大小写
    # print(res.query)  #  WHERE `app01_books`.`title` LIKE BINARY %s%
    # 忽略大小写
    # res = models.Books.objects.filter(title__icontains='s')
    # print(res.query)  # WHERE `app01_books`.`title` LIKE %s%

    # 查询以什么开头 结尾
    # res = models.Books.objects.filter(title__startswith='三')
    # print(res)

    # 6.查询出版日期是2021的书
    # res = models.Books.objects.filter(publish_time__year=2021)
    # print(res.query)  # WHERE `app01_books`.`publish_time` BETWEEN 2021-01-01 AND 2021-12-31
    # 7.查询出版日期是9月的书
    # res = models.Books.objects.filter(publish_time__month=9)
    # print(res)
    # 8.查询出版日期是20日的书
    # res = models.Books.objects.filter(publish_time__day=20)
    # print(res)
```

> ###图书管理系统表设计

```python
class Book(models.Model):
    title = models.CharField(verbose_name='书名', max_length=32)
    price = models.DecimalField(verbose_name='价格', max_digits=8, decimal_places=2)
    publish_time = models.DateField(verbose_name='出版日期',auto_now_add=True)
    # 一对多 外键字段建在多的一方
    # 出版社外键 一对多
    publish = models.ForeignKey(to='Publish',on_delete=models.CASCADE)  # 默认就是主键
    """自动在外键字段后面加_id后缀"""
    # 多对多 外键字段推荐建在查询频率较高的表中
    # 作者外键 多对多
    authors = models.ManyToManyField(to='Author')  # 自动帮你创建书籍和作者的第三张表
    """虚拟字段不会在表中实例化出来 而是告诉ORM创建第三张关系表"""


class Publish(models.Model):
    title = models.CharField(verbose_name='名称',max_length=32)
    addr = models.CharField(verbose_name='地址',max_length=128)
    email = models.EmailField(verbose_name='邮箱')


class Author(models.Model):
    name = models.CharField(verbose_name='姓名',max_length=32)
    age = models.IntegerField(verbose_name='年龄')
    # 一对一 外键字段推荐建在查询频率较高的表中
    # 作者详情外键 一对一
    author_detail = models.OneToOneField(to='AuthorDetail',on_delete=models.CASCADE)
    """自动在外键字段后面加_id后缀"""


class AuthorDetail(models.Model):
    phone = models.BigIntegerField(verbose_name='电话')
    addr = models.CharField(verbose_name='地址',max_length=32)

"""
作者详情表
作者表
出版社表

书籍表和关系表通过orm实现
"""
```

> ###外键字段操作

```python
   # 外键字段
    # 直接传主键值
    # models.Book.objects.create(title='聊斋',price=666.98,publish_id=1)
    # models.Book.objects.create(title='聊斋志异2',price=666.98,publish_id=2)
    # 传数据对象
    # publish_obj = models.Publish.objects.filter(pk=1).first()
    # models.Book.objects.create(title='神雕侠侣',price=234.56,publish=publish_obj)

    # 改
    # models.Book.objects.filter(pk=1).update(publish_id=2)
    # publish_obj = models.Publish.objects.filter(pk=1).first()
    # models.Book.objects.filter(pk=4).update(publish=publish_obj)

    # 多对多外键字段
    # 增
    book_obj = models.Book.objects.filter(pk=1).first()
    # 主键值
    # book_obj.authors.add(2)  # 去第三张关系表中 与作者主键为2的绑定关系
    # 作者对象
    # author_obj = models.Author.objects.filter(pk=3).first()
    # book_obj.authors.add(author_obj)
    # 括号内支持传多个参数
    # book_obj.authors.add(1,2)
    # author_obj1 = models.Author.objects.filter(pk=1).first()
    # author_obj2 = models.Author.objects.filter(pk=2).first()
    # book_obj.authors.add(author_obj1,author_obj2)

    # 改
    # book_obj.authors.set([3,])
    # book_obj.authors.set((1,2))
    # author_obj1 = models.Author.objects.filter(pk=1).first()
    # author_obj2 = models.Author.objects.filter(pk=2).first()
    # book_obj.authors.set((author_obj1,author_obj2))

    # 删
    # book_obj.authors.remove(1)
    # book_obj.authors.remove(1,2)
    # book_obj.authors.remove(author_obj1,author_obj2)

    # 清空
    # book_obj.authors.clear()  # 去第三张关系表中删除所有该书籍对应的记录
    """
    add()
    remove()
        括号内既可以传数字也可以传对象 逗号隔开即可
    set()
        括号内必须传递可迭代对象  可迭代对象内既可以传数字也可以传对象 支持多个
    clear()
        清空操作 无需传值
    """
```

> ###跨表查询理论

```python
正向
 外键字段在谁哪儿，谁查另外的就是正向
反向
 没有外键字段
# 判断是否有关联的外键字段

"""
正向查询按外键字段
反向查询按表名小写  _set
"""
```

> ###基于对象的跨表查询

```python
# 子查询
"""
1.先查询出一个对象
2.基于对象点正反向字段
"""

反向查询表名小写加_set
 查询的对象可能有多个的情况
 查询的对象只有一个的情况不需要加_set
    
# 基于对象的跨表查询
    # 1.查询聊斋书籍对应的出版社名称
    # book_obj = models.Book.objects.filter(title='聊斋').first()
    # res = book_obj.publish
    # print(res)
    # print(res.title)

    # 2.查询神雕侠侣对应的作者
    # book_obj = models.Book.objects.filter(title='神雕侠侣').first()
    # res = book_obj.authors  # app01.Author.None
    # res = book_obj.authors.all()
    # print(res)

    # 3.查询jason的地址
    # author_obj = models.Author.objects.filter(name='jason').first()
    # res = author_obj.author_detail
    # print(res)
    # print(res.phone,res.addr)

    # 4.查询东方出版社出版过的书籍
    # publish_obj = models.Publish.objects.filter(title='东方出版社').first()
    # res = publish_obj.book_set.all()
    # print(res)

    # 5.查询jason写过的书
    # author_obj = models.Author.objects.filter(name='jason').first()
    # res = author_obj.book_set.all()
    # print(res)

    # 6.查询电话是110的作者姓名
    # authordetail_obj = models.AuthorDetail.objects.filter(phone=110).first()
    # res = authordetail_obj.author
    # print(res)
    # print(res.name,res.age)
```

> ###基于双下划线的跨表查询

```python
# 链表查询
 # 基于双下划线的跨表查询
    # 1.查询聊斋书籍对应的出版社名称
    # res = models.Book.objects.filter(title='聊斋').values('publish__title')
    # print(res)
    # 2.查询神雕侠侣对应的作者
    # res = models.Book.objects.filter(title='神雕侠侣').values('authors__name','authors__age')
    # print(res)
    # 3.查询jason的地址
    # res = models.Author.objects.filter(name='jason').values('author_detail__addr')
    # print(res)
    # 4.查询东方出版社出版过的书籍
    # res = models.Publish.objects.filter(title='东方出版社').values('book__title')
    # print(res)
    # 5.查询jason写过的书
    # res = models.Author.objects.filter(name='jason').values('book__title')
    # print(res)
    # 6.查询电话是110的作者姓名
    # res = models.AuthorDetail.objects.filter(phone=110).values('author__name')
    # print(res)

    # 1.查询聊斋书籍对应的出版社名称 不能用models.Book
    # res = models.Publish.objects.filter(book__title='聊斋')
    # res = models.Publish.objects.filter(book__title='聊斋').values('title')
    # print(res)
    # 2.查询神雕侠侣对应的作者
    # res = models.Author.objects.filter(book__title='神雕侠侣').values('name')
    # print(res)
    # 3.查询jason的地址
    # res = models.AuthorDetail.objects.filter(author__name='jason').values('addr')
    # print(res)

    # 查询神雕侠侣对应的作者的电话和地址
    # res = models.Book.objects.filter(title='神雕侠侣').values('authors').values('authors__author_detail__phone','authors__author_detail__addr')
    # res = models.Book.objects.filter(title='神雕侠侣').values('authors__author_detail__phone','authors__author_detail__addr')
    # res = models.Author.objects.filter(book__title='神雕侠侣').values('author_detail__phone','author_detail__addr')
    # print(res)
    
    
1.普通函数以__开头
 说明当前函数只在当前模块(py)下使用，尽量不在外部调用
```

# 今日内容

## 0 图书相关表关系建立

```python
1.5个表
2.书籍表，作者表，作者详情表(垂直分表)，出版社表，书籍和作者表(多对多关系)
一对一 多对多 本质都是一对多  外键关系
3.一对一的关系，关联字段可以写在任意一方
4.一对多的关系，关联字段写在多的一方
5.多对多的关系，必须建立第三张表(orm中，可以用一个字段表示，这个字段可以写在任意一方)

6 把表关系同步到数据库中
 -python manage.py makemigrations  # 在migrations文件夹下记录一下
 -python manage.py migrate  # 把记录变更到数据库
    
7 表关系
class Book(models.Model):
    nid = models.AutoField(primary_key=True)  # 自增，主键
    name = models.CharField(max_length=32)  # varchar 32
    price = models.DecimalField(max_digits=5, decimal_places=2)  # 长度 5 小数点后2位
    publish_date = models.DateField()  # 年月日类型
    # 阅读数
    # reat_num=models.IntegerField(default=0)
    # 评论数
    # commit_num=models.IntegerField(default=0)

    # to='Publish'也可以写成 to=Publish 但该类Publish要位于上方。建议加引号写
    # 2.x以后必须加on_delete，否则报错
    # models.CASCADE：级联删除，设为默认值，设为空，设为指定的值，不做处理
    publish = models.ForeignKey(to='Publish',to_field='nid',on_delete=models.CASCADE)
    # publish = models.ForeignKey(to='Publish',to_field='nid',on_delete=models.SET_DEFAULT,default=0)
    # publish = models.ForeignKey(to='Publish',to_field='nid',on_delete=models.SET_NULL)
    # publish = models.ForeignKey(to='Publish',to_field='nid',on_delete=models.DO_NOTHING)
    # 在数据库中，根本没有这个字段，orm用来查询，映射成一个表
    # 如果我不这么写，需要手动建立第三张表，中介模型
    authors=models.ManyToManyField(to='Author')
    def __str__(self):
        return self.name


class Author(models.Model):
    nid = models.AutoField(primary_key=True)
    name = models.CharField(max_length=32)
    age = models.IntegerField()
    author_detail = models.OneToOneField(to='AuthorDatail',to_field='nid',unique=True,on_delete=models.CASCADE)
    # author_detail = models.ForeignKey(to='AuthorDatail',to_field='nid',unique=True,on_delete=models.CASCADE)


class AuthorDatail(models.Model):
    nid = models.AutoField(primary_key=True)
    telephone = models.BigIntegerField()
    birthday = models.DateField()
    addr = models.CharField(max_length=64)


class Publish(models.Model):
    nid = models.AutoField(primary_key=True)
    name = models.CharField(max_length=32)
    city = models.CharField(max_length=32)
    email = models.EmailField()  # 本质是varchar类型
```

## 1 基于双下划线的跨表查询

```python
1 基于对象的跨表查
 -子查询，多次查询
2 基于双下划线的跨表查
 -多表连接查询
    
 from app01 import models
# 正反向：正向，关联字段在当前对象中，去另一个表中查

    # 查询主键为1的书籍的出版社所在的城市
    # book = models.Book.objects.filter(pk=1).first()
    # print(book.publish.city)

    # res = models.Publish.objects.filter(book__nid=1).values('city')
    # res = models.Book.objects.filter(pk=1).values('publish__city')
    # print(res)

    # 查询所有住址在北京的作者的姓名
    # res = models.Author.objects.filter(author_detail__addr='北京').values('name')
    # res = models.AuthorDatail.objects.filter(addr='北京').values('author__name')
    # print(res)

    # 查询egon出过的所有书籍的名字
    # res = models.Book.objects.filter(authors__name='egon').values('name')
    # res = models.Author.objects.filter(name='egon').values('book__name')
    # print(res)

    # 查询北京出版社出版过的所有书籍的名字以及作者的姓名和地址
    # res = models.Book.objects.filter(publish__name='北京出版社').values('name','authors__name','authors__author_detail__addr')
    # res = models.Publish.objects.filter(name='北京出版社').values('name','book__name','book__authors__name','book__authors__author_detail__addr')
    # res = models.Author.objects.filter(book__publish__name='北京出版社').values('book__publish__name','book__name','name','author_detail__addr')
    # res = models.AuthorDatail.objects.filter(author__book__publish__name='北京出版社').values('author__book__publish__name','author__book__name','author__name','addr')
    # print(res)
```

## 2 聚合查询

```python
1 聚合函数，sum，max，min,count,avg
# 聚合查询
# 计算所有图书的平均价格
from django.db.models import Sum,Avg,Max,Min,Count
res = models.Book.objects.all().aggregate(Avg('price'))
print(res)
# 计算所有图书的最高价格
res = models.Book.objects.all().aggregate(Max('price'))
print(res)
# 计算所有图书的总价格
res = models.Book.objects.all().aggregate(Sum('price'))
print(res)

2 把聚合结果字段重命名
# 查询结果重命名
res = models.Book.objects.filter(authors__name='egon').aggregate(总价格=Sum('price'))
print(res)

# 计算egon出版图书的总价格 平均价格
res = models.Book.objects.filter(authors__name='egon').aggregate(book_sum=Sum('price'),book_avg=Avg('price'))
print(res)
```

## 3 分组查询

```python
annotate()为调用的QuerySet中每一个对象都生成一个独立的统计值（统计方法用聚合函数）。

总结 ：跨表分组查询本质就是将关联表join成一张表，再按单表的思路进行分组查询。　
```

## 4 F查询

```python
# F 查询，取出某个字段对应的值
from django.db.models import F,Q
# 查询评论数大于阅读数的书籍
# res = models.Book.objects.filter(commit_num__gt=F('read_num'))
# print(res)

# 把所有图书价格+1
res = models.Book.objects.all().update(price=F('price')+1)
print(res)  # 影响的行数
```

## 5 Q查询

```python
# Q查询：构造出与 & 或 | 非 ~
from django.db.models import Q
# 查询名字叫红楼梦或者价格大于100的书
# res = models.Book.objects.filter(name='红楼梦',price__gt=100)  # and 并且
# res = models.Book.objects.filter(Q(name='红楼梦')|Q(price__gt=100))
# 查询名字不是红楼梦的书
# res = models.Book.objects.filter(~Q(name='红楼梦'))
# print(res)
# 查询名字不是红楼梦，并且价格大于100的书
# res = models.Book.objects.filter(~Q(name='红楼梦'),Q(price__gt=100))
# res = models.Book.objects.filter(~Q(name='红楼梦')&Q(price__gt=100))
# res = models.Book.objects.filter(~Q(name='红楼梦'),price__gt=100)
res = models.Book.objects.filter(~Q(name='红楼梦'),price__gt='100')
# print(res.query)
print(res)
```

### 补充

```python
1 普通函数以__开头
 说明当前函数只在当前模块(py)下使用，尽量不在外部调用
    
2 mysql
 -utf8: 2个字节表示一个字符
 -utf8mb4: 等同于真正意义上的utf-8 变长
 -utf-8: 1-4个字节，表示一个字符
 
3 Django的orm使用pymysql连接mysql
 -需要加这一句话(本质就是猴子补丁的应用)
  import pymysql
  pymysql.install_as_MySQLdb()
 -本质是想让它执行，放在哪里都可以
  -init中
  -settings.py中
```

### Django admin的使用

```python
1 后台管理，方便我们快速的录入数据
2 使用方法:
 第一步：在admin.py中把要使用的表注册
    from app01 import models
        admin.site.register(models.Book)
        admin.site.register(models.Author)
        admin.site.register(models.AuthorDatail)
        admin.site.register(models.Publish)
 第二步：创建个超级管理员
  python3 manage.py createsuperuser
  输入用户名，输入密码
 第三步：登录，录入数据   
  http://127.0.0.1:8000/admin

```

### 使用脚本调用Django

```python
 1 写一个脚本文件
    
import os
# 加载配置文件，跑Django的项目，最开始就是把配置文件加载上
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'books.settings')

if __name__ == '__main__':
    import django  # 安装了Django模块，就可以import
    django.setup()  # 使用环境变量中的配置文件，跑Django
    
    from app01 import models
```

### Django查看源生sql

```python
1 queryset对象.query
2 通过日志，如下，配置到settings.py中
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console':{
            'level':'DEBUG',
            'class':'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'propagate': True,
            'level':'DEBUG',
        },
    }
}
```

### 作业

```python
# 计算egon出版图书的总价格
res = models.Book.objects.filter(authors__name='egon').aggregate(Sum('price'))
print(res)
# 计算北京出版社出版的最高价格
res = models.Book.objects.filter(publish__name='北京出版社').aggregate(Max('price'))
print(res)
```

## 昨日回顾

```python
-1 前后端混合开发(前后端都写 全栈开发)
 前后端分离，前端的人专门写前端，后端的人专门写后端
 前端工程化 前端框架 
0 Django的模板语法  不同的语言有自己的模板语法
 -dtl：在模板中写python代码  xx.html
 -php：http://www.aa7a.cn/user.php
 -java：https://www.pearvideo.com/category_loading.jsp
 -go：xx.html
 -jsp javascript=js 解释器 浏览器
    模板语法只负责把模板渲染成标准的带有HTML css js的页面
1 后续的课程
 -Django高级，ajax,分页，auth,中间件..
 -BBS项目
 -drf：写接口
 -vue
 -路飞：git celery redis 发短信...
 -flask
 -cmdb：自动化运维
 -爬虫
 -go
 -es
 -redis高级
 -rabbitmq,rpc
 -mongodb
 -mysql 主从，读写分离，分库分表
 -分布式锁，分布式id...
    
2 python后端开发，爬虫，自动化运维，自动化测试，python开发(写脚本)
 数据分析，go开发
    
3 多对多操作的api
 -add
 -remove
 -clear
 -set

book_obj.authors.remove()      # 将某个特定的对象从被关联对象集合中去除。   ====== book_obj.authors.remove(*[])
book_obj.authors.clear()       #清空被关联对象集合
book_obj.authors.set()         #先清空再设置　

4 什么情况下需要手动创建第三张表？
 -第三张表中有其他额外的字段
```

## 今日内容

### 1 分组查询

```python
 # 分组查询
    # 查询每一个出版社id，以及图书平均价格(单表)
    # 原生sql
    # select publish_id,avg(price) from app01_book group by publish_id;
    # orm 实现
    """标准 annotate() 内写聚合函数  QuerySet对象的方法
    values在前，表示group by 的字段
    values在后，表示取字段
    filter在前，表示where条件
    filter在后，表示having  分组之后
    """
    from django.db.models import Avg,Sum,Count,Max
    # res = models.Book.objects.all().values('publish_id').annotate(price_avg=Avg('price')).values('publish_id','price_avg')
    # print(res)

    # 查询出版社id大于1的出版社id，以及出书平均价格
    # res = models.Book.objects.values('publish_id').filter(publish_id__gt=1).annotate(price_avg=Avg('price')).values('publish_id','price_avg')
    # print(res)
    # 查询出版社id大于1的出版社id，以及出书平均价格大于30的
    # res = models.Book.objects.values('publish_id').filter(publish_id__gt=1).annotate(price_avg=Avg('price')).filter(price_avg__gt=30).values('publish_id','price_avg')
    # print(res)
    # 查询每一个出版社出版的名称和书籍个数(连表)
    # 连表的话最好以group by的表作为主表
    # 最后一个values只能取Publish表中的字段和annotate中的字段
    # res = models.Publish.objects.values('nid').annotate(book_count=Count('book__nid')).values('name','book_count')
    # 简写 如果基表是group by 的表，就可以不写values
    # res = models.Publish.objects.annotate(book_count=Count('book')).values('name','book_count')
    # 以book为基表
    # res = models.Book.objects.values('publish__nid').annotate(book_count=Count('nid')).values('publish__name','book_count')
    # print(res)

    # 查询每个作者的名字，以及出版过书籍的最高价格(建议使用分组的表作为基表)
    # 多对多如果不以分组表为基表，可能会出数据问题
    # res = models.Author.objects.annotate(price_max=Max('book__price')).values('name','price_max')
    # res = models.Book.objects.values('authors__nid').annotate(price_max=Max('price')).values('authors__name','price_max')
    # print(res)

    # 查询每一个书籍的名称，以及对应的作者个数
    # res = models.Book.objects.annotate(author_count=Count('authors__name')).values('name','author_count')
    # res = models.Book.objects.annotate(author_count=Count('authors')).values('name','author_count')
    # print(res)

    # 统计不止一个作者的图书
    # res = models.Book.objects.annotate(author_count=Count('authors')).filter(author_count__gt=1).values('name','author_count')
    # print(res)
    # 统计价格数大于10元，作者的图书
    # res = models.Book.objects.values('price').filter(price__gt=10).annotate(author_count=Count('authors')).values('name','author_count')
    # res = models.Book.objects.filter(price__gt=10).annotate(author_count=Count('authors')).values('name','author_count')
    # print(res)

    # 统计价格数大于10元，作者个数大于1的图书
    # res = models.Book.objects.values('price').filter(price__gt=10).annotate(author_count=Count('authors')).filter(author_count__gt=1).values('name')
    # res = models.Book.objects.filter(price__gt=10).annotate(author_count=Count('authors')).filter(author_count__gt=1).values('name','price','author_count')
    # print(res)
```

### 2 图书管理系统项目

```python
1 后端是django+mysql/sqlite
2 前端：jquery,bootstrap
3 首页，图书列表展示，图书新增，修改，作者展示，新增，修改，出版社展示，新增，修改...
4 项目地址：https://gitee.com/liuqingzheng/books
```

## 补充

### 1 wsgi, uwsgi, cgi, fastcgi

```python
浏览器只能发出HTTP的请求
本质上：浏览器是一个socket客户端，服务器是一个socket服务端
web服务器性能的高低决定了整个项目性能的高低
请求对象environment 字典 =》 request对象=》Django框架()
start_response响应对象

python：有wsgi协议,uwsgi,gunicorn
java：Tomcat,Jboss
php：php服务器


wsgi：协议，规定了如何拆HTTP请求，拆到一个python字典中，
environment，响应对象，start_response
wsgiref：符合wsgi协议的web服务器

CGI：通用网关接口。一句话总结：一个标准，定义了客户端服务器之间如何传数据

FastCGI：快速通用网关接口。一句话总结：CGI的升级版

WSGI：Web服务器网关接口。一句话总结：为python定义的web服务器和web框架之间的接口标准

uWSGI：用c语言写的，性能比较高 一句话总结：一个Web Server,即一个实现了WSGI的服务器，大体和Apache是一个类型的东西，处理发来的请求

uwsgi：是一种通信协议 uWSGI自有的协议
详情见：http://liuqingzheng.top/article/1/05-CGI,FastCGI,WSGI,uWSGI,uwsgi%E4%B8%80%E6%96%87%E6%90%9E%E6%87%82/
    
# asgi协议 异步服务网关接口
```

## 昨日回顾

```python
1 分组查询
 -把同一类归为一组，然后使用聚合函数操作
 -如果是多表，把两个表连起来，再分组，再聚合
 -取得字段必须是分组字段或者聚合函数的字段
    如果是单表只能取分组字段和聚合函数聚合的字段
    如果是多表可以取分组表中的所有字段和聚合函数的字段
 -总结：
     -annotate（聚合函数）
     -values在前，表示分组字段
     -values在后，表示取字段
     -filter在前，表示where条件
     -filter在后，表示having条件
2 wsgi，uWSGI,uwsgi,cgi,fastcgi
3 前后端开发模式
 -动态网站（网页的内容是由数据库渲染过来的 每次刷新都不一样）和静态网站（一个死页面 不变）
 -前后端分离：后端只写后端，返回json格式字符串，js语言 DOM  vue react
 -前后端混合开发：模板，dtl(模板语法 本质就是字符串的替换)，jsp，php
    
4 图书管理系统
 -后端是Django+mysql+bootstrap    （主机管理系统，人事管理系统，文档分享平台）
 -图书增删查改
     -增，删，查
 -出版社的增删查改
 -作者的增删查改
    
页面静态化
```

## 今日内容

### 1 图书管理系统图书修改

- 1.1 views

  ```python
  修改图书获取id的两种方案
  1  <input type="hidden" name="id" value="{{ book.nid }}">
  2      <form action="/update_book/?id={{ book.nid }}" method="post">{% csrf_token %}
  ```

- 1.2 路由urls

- 1.3 前端模板

  - book
  - publish
  - author

### 2 orm常用和非常用字段（了解）

```python
1 常用
1 AutoField 
int自增列，必须填入参数 primary_key=True。当model中如果没有自增列，则自动会创建一个列名为id的列。
2 IntegerField
一个整数类型,范围在 -2147483648 to 2147483647。
3 CharField
字符类型，必须提供max_length参数， max_length表示字符长度。
4 DateField
日期字段，日期格式 YYYY-MM-DD，相当于Python中的datetime.date()实例。
5 DateTimeField
日期时间字段，格式 YYYY-MM-DD HH:MM[:ss[.uuuuuu]][TZ]，相当于Python中的datetime.datetime()实例
    
2 不常用：
FileField(Field)
- 字符串，路径保存在数据库，文件上传到指定目录
- 参数：
upload_to = ""      上传文件的保存路径
storage = None      存储组件，默认django.core.files.storage.FileSystemStorage

TextField(Field)
- 文本类型

BooleanField(Field)
- 布尔值类型

3 对应关系
对应关系：
    'AutoField': 'integer AUTO_INCREMENT',
    'BigAutoField': 'bigint AUTO_INCREMENT',
    'BinaryField': 'longblob',
    'BooleanField': 'bool',
    'CharField': 'varchar(%(max_length)s)',
    'CommaSeparatedIntegerField': 'varchar(%(max_length)s)',
    'DateField': 'date',
    'DateTimeField': 'datetime',
    'DecimalField': 'numeric(%(max_digits)s, %(decimal_places)s)',
    'DurationField': 'bigint',
    'FileField': 'varchar(%(max_length)s)',
    'FilePathField': 'varchar(%(max_length)s)',
    'FloatField': 'double precision',
    'IntegerField': 'integer',
    'BigIntegerField': 'bigint',
    'IPAddressField': 'char(15)',
    'GenericIPAddressField': 'char(39)',
    'NullBooleanField': 'bool',
    'OneToOneField': 'integer',
    'PositiveIntegerField': 'integer UNSIGNED',
    'PositiveSmallIntegerField': 'smallint UNSIGNED',
    'SlugField': 'varchar(%(max_length)s)',
    'SmallIntegerField': 'smallint',
    'TextField': 'longtext',
    'TimeField': 'time',
    'UUIDField': 'char(32)',
```

### 3 orm字段参数（了解）

```python
1 #### null

用于表示某个字段可以为空。

2 #### **unique**

如果设置为unique=True 则该字段在此表中必须是唯一的 。

3 #### **db_index**

如果db_index=True 则代表着为此字段设置索引。

4 #### **default**

为该字段设置默认值。

5 ###  DateField和DateTimeField

#### auto_now_add

配置auto_now_add=True，创建数据记录的时候会把当前时间添加到数据库。

#### auto_now  (对象.属性  对象.save())    queryset.update 无效

配置上auto_now=True，每次更新数据记录的时候会更新该字段。
6 choices
 在model表模型定义的时候给某个字段指定choice
 sex_choice=((1,'男'),(2,'女'),(0,'未知'))
 sex=models.IntegerField(default=1,choices=sex_choice)
 在使用的时候，直接取出中文
  对象.get_sex_display()

null              数据库中字段是否可以为空
db_column           数据库中字段的列名
db_tablespace
default             数据库中字段的默认值
primary_key          数据库中字段是否为主键
db_index            数据库中字段是否可以建立索引
unique              数据库中字段是否可以建立唯一索引
unique_for_date     数据库中字段【日期】部分是否可以建立唯一索引
unique_for_month    数据库中字段【月】部分是否可以建立唯一索引
unique_for_year     数据库中字段【年】部分是否可以建立唯一索引
了解：
verbose_name        Admin中显示的字段名称
blank               Admin中是否允许用户输入为空
editable            Admin中是否可以编辑
help_text           Admin中该字段的提示信息
choices             Admin中显示选择框的内容，用不变动的数据放在内存中从而避免跨表操作
如：gf = models.IntegerField(choices=[(0, '何穗'),(1, '大表姐'),],default=1)

error_messages      自定义错误信息（字典类型），从而定制想要显示的错误信息；
字典健：null, blank, invalid, invalid_choice, unique, and unique_for_date
如：{'null': "不能为空.", 'invalid': '格式错误'}

validators          自定义错误验证（列表类型），从而定制想要的验证规则
from django.core.validators import RegexValidator
from django.core.validators import EmailValidator,URLValidator,DecimalValidator,\
MaxLengthValidator,MinLengthValidator,MaxValueValidator,MinValueValidator
如：
test = models.CharField(
max_length=32,
error_messages={
'c1': '优先错信息1',
'c2': '优先错信息2',
'c3': '优先错信息3',
},
validators=[
RegexValidator(regex='root_\d+', message='错误了', code='c1'),
RegexValidator(regex='root_112233\d+', message='又错误了', code='c2'),
EmailValidator(message='又错误了', code='c3'), ]
                            )
```

### 4 字段关系（了解）

```python
1 一对一 一对多 多对多

一对多

2 ForeignKey
外键类型在ORM中用来表示外键关联关系，一般把ForeignKey字段设置在 ‘一对多’中’多’的一方。
ForeignKey可以和其他表做关联关系同时也可以和自身做关联关系。
    -to
    设置要关联的表
    -to_field
    设置要关联的表的字段
    -related_name
    反向操作时，使用的字段名，用于代替原反向查询时的’表名_set’。
    -related_query_name
 反向查询操作时，使用的连接前缀，用于替换表名。


on_delete
　　当删除关联表中的数据时，当前表与其关联的行的行为。
　　models.CASCADE
　　删除关联数据，与之关联也删除
　　models.DO_NOTHING
　　删除关联数据，什么都不做
　　models.PROTECT
　　删除关联数据，引发错误ProtectedError
　　models.SET_NULL
　　删除关联数据，与之关联的值设置为null（前提FK字段需要设置为可空）
　　models.SET_DEFAULT
　　删除关联数据，与之关联的值设置为默认值（前提FK字段需要设置默认值）
　　models.SET
　　删除关联数据，
　　a. 与之关联的值设置为指定值，设置：models.SET(值)
　　b. 与之关联的值设置为可执行对象的返回值，设置：models.SET(可执行对象)
    
    
db_constraint
是否在数据库中创建外键约束，默认为True。  False  不建立外键
 -外键是否建立：
    插入数据，会去检索关联，有个校验
    -好处：不会出现脏数据
    -坏处：插入的时候，效率低
    -企业中，通常不建立，程序员控制
    
3 一对一  OneToOneField   同ForeignKey一样 
4 多对多  ManyToManyField ：如何手动创建第三张表
```

### 5 手动创建第三张表

```python
-字段参数
 -db_table：指定第三张表的名字    默认创建第三张表时，数据库中表的名称。
 -to: 设置要关联的表
 -related_name: 同ForeignKey字段。
 -related_query_name: 同ForeignKey字段。
        
 -through: 手动创建第三张表来管理多对多关系，通过through来指定第三张表的表名。
 -through_fields: 设置关联的字段。
        
-多对多关系建立的三种方式
 -第一种：自动创建（常用：第三张表没有其他字段）
 -第二种：手动创建第三张（比较常用：第三张表有多余字段）
 -第三种：完全手动写第三张表
    
-第三种：
class Book(models.Model):
    title = models.CharField(max_length=32, verbose_name="书名")

class Author(models.Model):
    name = models.CharField(max_length=32, verbose_name="作者姓名")

# 自己创建第三张表，分别通过外键关联书和作者
class Author2Book(models.Model):
    author = models.ForeignKey(to="Author")
    book = models.ForeignKey(to="Book")

    class Meta:
        unique_together = ("author", "book")
        
-第一种：
class Book(models.Model):
    title = models.CharField(max_length=32, verbose_name="书名")

# 通过ORM自带的ManyToManyField自动创建第三张表
class Author(models.Model):
    name = models.CharField(max_length=32, verbose_name="作者姓名")
    books = models.ManyToManyField(to="Book", related_name="authors")
    
-第二种：
class Book(models.Model):
    title = models.CharField(max_length=32, verbose_name="书名")


# 自己创建第三张表，并通过ManyToManyField指定关联
class Author(models.Model):
    name = models.CharField(max_length=32, verbose_name="作者姓名")
    books = models.ManyToManyField(to="Book", through="Author2Book", through_fields=("author", "book"))
    # through_fields接受一个2元组（'field1'，'field2'）：
    # 其中field1是定义ManyToManyField的模型外键的名（author），field2是关联目标模型（book）的外键名。


class Author2Book(models.Model):
    author = models.ForeignKey(to="Author")
    book = models.ForeignKey(to="Book")

    class Meta:
        unique_together = ("author", "book")

# through_fields 元组的第一个值是ManyToManyField所在的表去中间表通过哪个字段，就写在第一个位置

# 基于对象的跨表查询 还能继续使用
# 基于双下划线连表查
# 原来的多对多操作api用不了，需要手动操作

```

### 6 Meta元信息

```python'
ORM对应的类里面包含另一个Meta类，而Meta类封装了一些数据库的信息。主要字段如下:

db_table
ORM在数据库中的表名默认是 app_类名，可以通过db_table可以重写表名。

index_together
联合索引。

unique_together
联合唯一索引。

ordering
指定默认按什么字段排序。

只有设置了该属性，我们查询到的结果才可以被reverse()。


class UserInfo(models.Model):
    nid = models.AutoField(primary_key=True)
    username = models.CharField(max_length=32)

    class Meta:
        # 数据库中生成的表名称 默认 app名称 + 下划线 + 类名
        db_table = "table_name"

        # 联合索引
        index_together = [
            ("pub_date", "deadline"),
        ]

        # 联合唯一索引
        unique_together = (("driver", "restaurant"),)
        
        ordering = ('name',)
        
        # admin中显示的表名称
        verbose_name='哈哈'

        # verbose_name加s
        verbose_name_plural=verbose_name
```

### 7 原生SQL

```python
from django.db import connection, connections

cursor = connection.cursor() # connection=default数据
cursor = connections['db2'].cursor()

cursor.execute("""SELECT * from auth_user where id = %s""", [1])

row = cursor.fetchone()
row = cursor.fetchall()


ret = models.Author.objects.raw('select * from app01_author where nid>1')
print(ret)
for i in ret:
    print(i)
print(ret.query)
# 会把book的字段放到author对象中
ret = models.Author.objects.raw('select * from app01_book where nid>1')
print(ret)
for i in ret:
    print(i.price)
    print(type(i))
```

### 8 Django与ajax(入门)

```python
1 概念
AJAX（Asynchronous Javascript And XML）翻译成中文就是“异步Javascript和XML”。即使用Javascript语言与服务器进行异步交互，传输的数据为XML（当然，传输的数据不只是XML,现在更多使用json数据）。
2 异步：请求发出去，不会卡在这，可以干其他事
3 局部刷新：js的DOM操作，使页面局部刷新
4 基本上web页面都有很多ajax请求
```

### 8.1 写ajax跟后端交互

```python
1 使用原生js写ajax请求（没有人用）
 -第一：麻烦
 -第二：区分浏览器，需要做浏览器兼容
2 现在主流做法 （现成有人封装好了，jquery,axios..）
 -以jQuery为例讲    前后端混合
 -后面会讲axios     前后端分离
     
```

## 昨日回顾

```python
1 图书管理系统编辑功能
2 常用和非常用字段
3 字段参数
4 字段关系：to, to_fileds, related_name, related_query_name, on_delete, db_constraint
5 第三张表建立的三种方式
 -纯手动建立（不使用manytomany）
 -自动创建第三张表
  -手动创建第三张表，使用manytomany（多对多api用不了）
6 Meta元信息
7 原生sql
8 orm框架（了解）
 -python：Django的orm，sqlalchemy(独立使用，集成到flask)
 -go:beego自带的orm框架，gorm
 -java：mybatis,Hibernate
    
9 ajax:js跟后端交互
    -异步
    -局部刷新
    -原生js写（麻烦，兼容浏览器）
    -jQuery的ajax
    -axios
    发给请求 拿到数据 js渲染页面
```

## 今日内容

### 1 ajax发送其他请求

```python
1 写在form表单 submit和button会触发提交  <form action=""> </form>  注释
2 使用input 类型为 button  <input type="button" id="id_btn" value="提交">

1 大坑
 -如果在form表单中，写button和input是submit类型，会触发form表单的提交
 -如果不想触发：
     -不写在form表单中
       -使用input，类型是button
    
<form action="">
    <p>用户名：<input type="text" id="id_name"></p>
    <p>密码：<input type="password" id="id_password"></p>
{#    <button id="id_btn">提交</button>#}
    <input type="button" id="id_btn" value="提交">
</form>
    
// 登录成功，重定向百度，前端重定向
location.href='http://www.baidu.com'
location.href='/index/'
    
    
2 坑
 -后端响应格式如果是：html/text格式，ajax接收到数据后需要自己转成对象
    res = JSON.parse(data)  // 不是json格式需要手动解析
 -后端响应格式是：json，ajax接收到数据后会自动转成对象
   -总结：后端返回数据，统一都用JsonResponse
    
3 坑
    -如果使用了ajax，后端就不要返回rediret,render,HttpResponse
    -直接返回JsonResponse
```

### 2 上传文件（ajax和form两种方式）

```python
默认请求编码格斯为：urlencoded 上传文件需要使用格式：enctype="multipart/form-data"
1 http --post--请求，有编码格式，主流有三种
 -urlencode：默认的  ----> 从request.POST取提交的数据
 -form-data：上传文件的  ----> 从request.POST取提交的数据,request.FILES中取文件
 -json：ajax发送json格式数据  ----> 从request.POST取不出数据
    
2 使用ajax和form表单，默认都是urlencode格式
3 如果上传文件：form表单格式  enctype=
4 如果编码方式是urlencode格式，放到body体(请求体)中数据格式如下： ajax预处理数据
 username=lqz&password=123
    
<body>
<h1>form表单上传文件</h1>
<form action="" method="post" enctype="multipart/form-data">
    <p>用户名：<input type="text" name="name"></p>
    <p>文件：<input type="file" name="myfile"></p>
    <input type="submit" value="提交">
</form>

<h1>ajax上传文件</h1>
<p>用户名：<input type="text" id="id_name"></p>
<p>文件：<input type="file" id="id_myfile"></p>
<button id="id_btn">提交</button>

</body>
<script>
    $('#id_btn').click(function (){
        // 如果ajax要上传文件，需要借助于一个js的formdata对象

        var formdata=new FormData()  // 实例化得到一个FormData对象
        formdata.append('name',$('#id_name').val())  // 追加了一个name对应填入的值
        // 追加文件
        var file=$('#id_myfile')[0].files[0]
        formdata.append('myfile',file)
        $.ajax({
            url: '/file_upload/',
            method: 'post',
            // 上传文件，一定要注意如下两行
            processData: false,  // 不预处理数据,
            contentType: false,  // 不指定编码格式 使用FormData对象的默认编码就是formdata格式
            data: formdata,
            success: function (data){
                console.log(data)
            }
        })
    })
</script>

$('#id_myfile')[0]  # jQuery对象转成原生input

$('#id_myfile')[0].files  # 所有的文件

$('#id_myfile')[0].files[0]    # js中的文件对象




def file_upload(request):
    if request.method=='GET':
        return render(request,'file_upload.html')
    else:
        name = request.POST.get('name')
        myfile=request.FILES.get('myfile')
        print(type(myfile))  # 查看类型 <class 'django.core.files.uploadedfile.InMemoryUploadedFile'>
        from django.core.files.uploadedfile import InMemoryUploadedFile
        with open(myfile.name,'wb') as f:
            for line in myfile:
                f.write(line)
        return HttpResponse('上传成功')
```

​ 2.1 form表单上传文件

​ 2.2 ajax上传文件

​ 2.3 后端

​ 2.4 路由

### 3 ajax上传json格式

#### 3.1 前端   

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    {#    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">#}
    {#    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>#}
</head>
<body>

<h1>ajax提交json格式</h1>

<p>用户名：<input type="text" id="id_name"></p>
<p>密码: <input type="password" id="id_password"></p>
<button id="id_button">提交</button>

</body>
<script>
    $('#id_button').click(function () {

        $.ajax({
            url: '/ajax_json/',
            method: 'post',
            contentType: 'application/json',  // 指定编码格式
            data: JSON.stringify({
                name: $('#id_name').val(),
                password: $('#id_password').val()
            }),  // json格式字符串 相当于python中json.dumps
            success: function (data) {
                console.log(data)
            }
        })
    })
</script>
</html>
```

#### 3.2 后端

```python
def ajax_json(request):
    if request.method == 'GET':
       return render(request, 'ajax_json.html')
    else:
        # json格式，从POST中取不出来

        name=request.POST.get('name')
        print(type(request.POST))  # <class 'django.http.request.QueryDict'>
        # from django.http.request import QueryDict
        print(name)
        # 在body体中，b格式
        request.data = json.loads(request.body)
        name = request.data.get('name')
        password = request.data.get('password')
        print(name)
        print(password)
        return HttpResponse('ok')
```

### 4 Django内置序列化（了解）

```python
Django内置的serializers（把对象序列化成json字符串）
from django.core import serializers
from django.core import serializers
def test(request):
    book_list = Book.objects.all()    
    ret = serializers.serialize("json", book_list)
    return HttpResponse(ret)

把对象转成json格式字符串，Django内置的不好用，字段不能控制
目前阶段，要做序列化，for循环拼列表套字典

l = []
for user in user_list:
    l.append({'name':user.name,'password':user.password})
return JsonResponse(l,safe=False)
```

### 5 分页器的使用

```python
###  批量插入数据
def books_page(request):
    # 第一种方案，每循环一次，操作一下数据库，性能低
    # for i in range(1000):
    #     book= models.Books.objects.create(name='图书%s' % i, price=i+10, publish='东京出版社')

    # 第二种方案，批量插入
    book_list = []
    for i in range(1000):
        book = models.Books(name='图书%s' % i, price=i+10, publish='东京出版社')
        book_list.append(book)

    models.Books.objects.bulk_create(book_list,batch_size=100)
    return HttpResponse('ok')


1 Django的分页器（paginator）简介
在页面显示分页数据，需要用到Django分页器组件

from django.core.paginator import Paginator

Paginator对象：    paginator = Paginator(user_list, 10)
# per_page: 每页显示条目数量
# count:    数据总个数
# num_pages:总页数
# page_range:总页数的索引范围，如: (1,10),(1,200)
# page:     page对象    
page对象：page=paginator.page(1)
# has_next              是否有下一页
# next_page_number      下一页页码
# has_previous          是否有上一页
# previous_page_number  上一页页码
# object_list           分页之后的数据列表
# number                当前页
# paginator             paginator对象

from django.core.paginator import Paginator
# def books_page(request):
#     current_num = int(request.GET.get('page_num',1))  # 如果请求不到当前页 返回第一页
#     book_list= models.Books.objects.all()
#     paginator=Paginator(book_list,50)
#     # Paginator对象的属性
#     print(paginator.count)  # 数据总条数
#     print(paginator.num_pages)  # 总页数
#     print(paginator.per_page)  # 每页显示条目数量
#     print(paginator.page_range)  # 总页数的索引范围 range(1, 101)
#     # print(paginator.page(1))  # 第一页
#     # page对象的属性和方法
#     # has_next              是否有下一页
#     # next_page_number      下一页页码
#     # has_previous          是否有上一页
#     # previous_page_number  上一页页码
#     # object_list           分页之后的数据列表
#     # number                当前页
#     page = paginator.page(current_num)
#     print(page.has_next())
#     print(page.next_page_number())
#     print(page.has_previous())
#     print(page.previous_page_number())
#     print(page.object_list)
#     print(page.number)
#
#     return render(request,'book_page.html',locals())

def books_page(request):
    current_num = int(request.GET.get('page_num',1))
    book_list= models.Books.objects.all()

    paginator=Paginator(book_list,50)
    page = paginator.page(current_num)
    return render(request,'book_page.html',locals())
```

## 作业

1 使用ajax发送post请求，完成注册功能，注册成功，跳转到登录，登录成功跳转到百度

2 使用ajax上传文件，保存到项目路径的media路径下（登录成功才能上传文件）

3 使用ajax上传json格式数据，写一个装饰器，实现不论前端以什么格式传递数据，我从视图函数中都从request.data中取值 （POST的数据）

```python
def json_parser(func):
    def inner(request,*args,**kwargs):
        if request.method == 'POST':
            try:
                request.data=json.loads(request.body)  # json格式
            except Exception as e:
                request.data=request.POST  # 其他两种格式
        res = func(request,*args,**kwargs)
        return res
    return inner


@json_parser
def test_json(request):
    if request.method == 'GET':
        return render(request,'test_json.html')
    else:
        print(request.data)
        return HttpResponse('ok')
```

## 补充

```python
json.loads(b'dfdasfda')
问题：json可以直接loads  bytes格式吗？
 -3.5之前不可以
 -3.6以后可以
```

## 昨日回顾

```js
1 ajax
 $.ajax({
        url:'/test/',
        method:'get/post',
        contentType:'application/json',
        //processData:false,
        //contentType:false,
        data:json格式字符串，字典对象，formdata对象,
        success:function(){
            //data的类型取决于，后端返回时指定的响应类型
            ...
        }
    })
        
2 上传文件（form表单，ajax）
3 提交json格式，后端无论是什么格式
4 登录注册功能
```

## 今日内容

### 1 分页器基本使用

```python

```

### 2 分页器终极用法

### 3 forms组件之校验字段

```python
1 前端
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>
<form action="" method="post">
    <p>用户名：<input type="text" name="name"></p>
    <p>密码：<input type="password" name="password"></p>
    <p>确认密码：<input type="password" name="re_password"></p>
    <p>邮箱：<input type="text" name="email"></p>
    <input type="submit" value="提交">
</form>
</body>
</html>


2 后端
# forms的使用
# 第一步：定义一个类，继承
# 第二版：在类中写字段，要校验的字段，字段属性就是校验规则
# 第三步：实例化得到一个Form对象，把要校验的数据传入
# 第四步：调用register_form.is_valid()校验，校验通过就是True
# 第五步：校验通过有register_form.cleaned_data
# 第六步：校验不通过有register_form.errors
from django import forms
from app01 import models
class RegisterForm(forms.Form):
    name = forms.CharField(max_length=8,min_length=3)
    password = forms.CharField(max_length=8,min_length=3)
    re_password = forms.CharField(max_length=8,min_length=3)
    email = forms.EmailField()


def register(request):
    if request.method == 'GET':
        return render(request,'register.html')
    else:
        # 实例化得到对象，传入要校验的数据
        # register_form = RegisterForm(data=request.POST)
        register_form = RegisterForm(request.POST)
        if register_form.is_valid():
            # 校验通过，存
            # 取出校验通过的数据
            print('校验通过')
            print(register_form.cleaned_data)
            register_form.cleaned_data.pop('re_password')
            models.User.objects.create(**register_form.cleaned_data)
        else:
            # 校验不通过
            print('校验不通过')
            print(register_form.errors)
        return HttpResponse('ok')

```

### 4 forms组件之渲染标签

```python
1 前端
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>
<h1>自己写</h1>
<form action="" method="post">
    <p>用户名：<input type="text" name="name"></p>
    <p>密码：<input type="password" name="password"></p>
    <p>确认密码：<input type="password" name="re_password"></p>
    <p>邮箱：<input type="text" name="email"></p>
    <input type="submit" value="提交">
</form>
<hr>
<h2>通过form自动渲染一</h2>
<form action="" method="post">
    <p>用户名：{{ form.name }}</p>
    <p>密码：{{ form.password }}</p>
    <p>确认密码：{{ form.re_password }}</p>
    <p>邮箱：{{ form.email }}</p>
    <input type="submit" value="提交">
</form>

<hr>
<h2>通过form自动渲染二(基本用这种)</h2>
<form action="" method="post">
    {% for item in form %}
        <p>{{ item.label }}：{{ item }}</p>
    {% endfor %}
    <input type="submit" value="提交">
</form>

<hr>
<h2>通过form自动渲染三</h2>
<form action="" method="post">
    {{ form.as_p }}
{#    {{ form.as_ul }}#}
    <input type="submit" value="提交">
</form>
</body>
</html>
    
2 后端 views
from django import forms
from app01 import models
class RegisterForm(forms.Form):
    name = forms.CharField(max_length=8,min_length=3,label='用户名')
    password = forms.CharField(max_length=8,min_length=3,label='密码')
    re_password = forms.CharField(max_length=8,min_length=3,label='确认密码')
    email = forms.EmailField(label='邮箱')


def register(request):
    if request.method == 'GET':
        # 生成一个空form对象
        register_form = RegisterForm()
        return render(request,'register.html',{'form':register_form})
    else:
        # 实例化得到对象，传入要校验的数据
        # register_form = RegisterForm(data=request.POST)
        register_form = RegisterForm(request.POST)
        if register_form.is_valid():
            # 校验通过，存
            # 取出校验通过的数据
            print('校验通过')
            print(register_form.cleaned_data)
            register_form.cleaned_data.pop('re_password')
            models.User.objects.create(**register_form.cleaned_data)
        else:
            # 校验不通过
            print('校验不通过')
            print(register_form.errors)
        return HttpResponse('ok')
```

### 5 forms组件之渲染错误信息

### 6 forms组件参数设置

### 7 forms组件全局钩子，局部钩子

#### 1 后端

```python
# forms的使用
# 第一步：定义一个类，继承
# 第二版：在类中写字段，要校验的字段，字段属性就是校验规则
# 第三步：实例化得到一个Form对象，把要校验的数据传入
# 第四步：调用register_form.is_valid()校验，校验通过就是True
# 第五步：校验通过有register_form.cleaned_data
# 第六步：校验不通过有register_form.errors
from django import forms
from app01 import models
from django.forms import widgets
from django.core.exceptions import ValidationError

class RegisterForm(forms.Form):
    name = forms.CharField(max_length=8, min_length=3, label='用户名',
                           error_messages={'max_length': '最大长度为8', 'min_length': '最小长度为3', 'required': '该字段必填'},
                           widget=widgets.TextInput(attrs={'class':'form-control'}))
    password = forms.CharField(max_length=8, min_length=3, label='密码',
                               error_messages={'max_length': '最大长度为8', 'min_length': '最小长度为3', 'required': '该字段必填'},
                               widget=widgets.PasswordInput(attrs={'class':'form-control'}))
    re_password = forms.CharField(max_length=8, min_length=3, label='确认密码',
                                  error_messages={'max_length': '最大长度为8', 'min_length': '最小长度为3', 'required': '该字段必填'},
                                  widget=widgets.PasswordInput(attrs={'class':'form-control'}))
    email = forms.EmailField(label='邮箱', error_messages={'required': '该字段必填', 'invalid': '邮箱格式错误'},widget=widgets.TextInput(attrs={'class':'form-control'}))

    def clean_name(self):  # name字段的局部钩子
        # 校验名字不能以sb开头
        name = self.cleaned_data.get('name')
        if name.startswith('sb'):
            # 校验不通过，必须抛异常
            raise ValidationError('不能以sb开头')
        else:  # 校验通过，返回name对应的值
            return name

    def clean(self):  # 全局钩子
        password = self.cleaned_data.get('password')
        re_password = self.cleaned_data.get('re_password')
        if re_password == password:
            # 校验通过
            return self.cleaned_data
        else:
            raise ValidationError('两次密码不一致')

def register(request):
    if request.method == 'GET':
        # 生成一个空form对象
        register_form = RegisterForm()
        return render(request, 'register.html', {'form': register_form})
    else:
        # 实例化得到对象，传入要校验的数据
        # register_form = RegisterForm(data=request.POST)
        register_form = RegisterForm(request.POST)
        if register_form.is_valid():
            # 校验通过，存
            # 取出校验通过的数据
            print('校验通过')
            print(register_form.cleaned_data)
            register_form.cleaned_data.pop('re_password')
            models.User.objects.create(**register_form.cleaned_data)
            return HttpResponse('ok')
        else:
            # 校验不通过
            print('校验不通过')
            print(register_form.errors)
            error = register_form.errors.get('__all__')[0]
            return render(request, 'register.html', {'form': register_form,'error':error})

# <ul class="errorlist"><li>name<ul class="errorlist"><li>Ensure this value has at least 3 characters (it has 1).</li></ul></li></ul>
```

#### 2 前端

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>
<h1>自己写</h1>
<form action="" method="post">
    <p>用户名：<input type="text" name="name"></p>
    <p>密码：<input type="password" name="password"></p>
    <p>确认密码：<input type="password" name="re_password"></p>
    <p>邮箱：<input type="text" name="email"></p>
    <input type="submit" value="提交">
</form>
<hr>
<h2>通过form自动渲染一</h2>
<form action="" method="post">
    <p>用户名：{{ form.name }}</p>
    <p>密码：{{ form.password }}</p>
    <p>确认密码：{{ form.re_password }}</p>
    <p>邮箱：{{ form.email }}</p>
    <input type="submit" value="提交">
</form>

<hr>
<h2>通过form自动渲染二(基本用这种)</h2>
<div class="container-fluid">
    <div class="row">
        <div class="col-md-6 col-md-offset-3">
            <form action="" method="post" novalidate>
    {% for item in form %}
        <div class="form-group">
            <p>{{ item.label }}：{{ item }} <span style="color: red">{{ item.errors.0 }}</span></p>
        </div>

    {% endfor %}
    <input type="submit" value="提交"><span style="color: red">{{ error }}</span>
</form>
        </div>
    </div>
</div>


<hr>
<h2>通过form自动渲染三</h2>
<form action="" method="post">
    {{ form.as_p }}
    {#    {{ form.as_ul }}#}
    <input type="submit" value="提交">
</form>
</body>
</html>
```

Django设置响应头

代码review  保证软件质量

## 上节回顾

```python
1 分页
 -Django提供的两个类
 -Paginator类  page对象
 -Page类
2 forms组件
 -数据校验
 -渲染页面
 -错误信息
 -局部全局钩子
 -使用步骤：
     -写一个类，继续Form类
       -写字段，字段参数（限制该字段的长短）
     -错误信息中文：字段参数
       -widget：控制生成标签的属性
     -视图函数中：
        -实例化得到form对象时，把要校验的数据传入
        -is_valiad():clean_data和errors就有值了 即便校验出错clean_data 也可能有值
        -如果校验通过就存，不通过就给页面提示
       -渲染页面
       -for循环的方式渲染页面(在标签前后可以再加标签)
 -局部钩子
     - def clean_字段名(self):
            -检验规则
            -如果通过，return 值
            -如果不通过，抛异常
 -全局钩子(多个字段校验)
     -def clean(self):
            -如果通过，return clean_data
            -如果不通过，抛异常
    
        
```

## 今日内容

### 1 forms组件源码分析

```python
1 为什么局部钩子要写成clean_字段名，为什么要抛异常
2 入口在is_valid()
3 校验流程
 -先校验字段自己的规则（最大，最小，是否必填，是不是合法）
 -校验局部钩子函数
 -全局钩子校验
    
4 流程
 -is_valid()  --> return self.is_bound and not self.errors
 -self.errors: 方法包装成了数据属性
        -一旦有值，self.errors就不进行校验(之前调用过了)
 - self.full_clean():  核心
        
 if not self.is_bound:  # Stop further processing. 如果data没有值直接返回
    return

    def full_clean(self):
        """
        Clean all of self.data and populate self._errors and self.cleaned_data.
        """
        self._errors = ErrorDict()
        if not self.is_bound:  # Stop further processing.
            return
        self.cleaned_data = {}
        # If the form is permitted to be empty, and none of the form data has
        # changed from the initial data, short circuit any validation.
        if self.empty_permitted and not self.has_changed():
            return

        self._clean_fields()
        self._clean_form()
        self._post_clean()
        
     # 局部钩子执行位置  
    def _clean_fields(self):
        for name, field in self.fields.items():
            # value_from_datadict() gets the data from the data dictionaries.
            # Each widget type knows how to retrieve its own data, because some
            # widgets split data over several HTML fields.
            if field.disabled:
                value = self.get_initial_for_field(field, name)
            else:
                value = field.widget.value_from_datadict(self.data, self.files, self.add_prefix(name))
            try:
                if isinstance(field, FileField):
                    initial = self.get_initial_for_field(field, name)
                    value = field.clean(value, initial)
                else:
                    value = field.clean(value)  # 字段自己的校验规则
                self.cleaned_data[name] = value  # 把校验后的数据放到cleaned_data
                if hasattr(self, 'clean_%s' % name):  # 判断有没有局部钩子
                    value = getattr(self, 'clean_%s' % name)() # 执行局部钩子
                    self.cleaned_data[name] = value  # 校验通过 把数据替换一下
            except ValidationError as e:  # 如果校验不通过，会抛异常，会被捕获
                self.add_error(name, e)  # 捕获后执行  添加到错误信息 errors是个列表 错误可能有多个
                
      
# 全局钩子执行位置
    def _clean_form(self):
        try:
            # 如果自己定义的form类中写了clean，他就会执行
            cleaned_data = self.clean()  
        except ValidationError as e:
            self.add_error(None, e) # key作为None就是__all__
        else:
            if cleaned_data is not None:
                self.cleaned_data = cleaned_data
```

### 2 cookie，session，token扫盲

```python
1 cookie: 保存到客户端浏览器上的键值对
 用户名 密码 登录状态 写到 cookie
 不加密的cookie不安全
 -如果不加密，是不安全的（可能被窃取，篡改）
    只要存在客户端浏览器上的东西都叫cookie
    
cookie 是一个非常具体的东西，指的就是浏览器里面能永久存储的一种数据，仅仅是浏览器实现的一种数据存储功能。

cookie由服务器生成，发送给浏览器，浏览器把cookie以kv形式保存到某个目录下的文本文件内，下一次请求同一网站时会把该cookie发送给服务器。由于cookie是存在客户端上的，所以浏览器加入了一些限制确保cookie不会被恶意使用，同时不会占据太多磁盘空间，所以每个域的cookie数量是有限的。

2 session：存在服务端的键值对
 -用户登录后，给用户分配一个随机字符串（会话标识(session id)），用户存到cookie中
 -在服务端以刚刚随机字符串为key，value是字典，放用户信息
    
我就不保存session id 了， 我只是生成token , 然后验证token ， 我用我的CPU计算时间获取了我的session 存储空间 ！

session 从字面上讲，就是会话
服务器使用session把用户的信息临时保存在了服务器上，用户离开网站后session会被销毁。这种用户信息存储方式相对cookie来说更安全，可是session有一个缺陷：如果web服务器做了负载均衡，那么下一个操作请求到了另一台服务器的时候session会丢失。

3 Token：三段式(JWT:json web token)（服务器不存了）
 {公司信息..}.{name:qiao,id:8}.adsfasasf  # 把第1段和第2端加密后得到一个签名
    asdfas.asfadafad.adafasfd  # 整体base64转码后给客户端 cookie
tokens 是多用户下处理认证的最佳方式
    1.无状态、可扩展
    2.支持移动设备
    3.跨程序调用
    4.安全
    
基于服务器验证方式暴露的一些问题

1.Seesion：每次认证用户发起请求时，服务器需要去创建一个记录来存储信息。当越来越多的用户发请求时，内存的开销也会不断增加。

2.可扩展性：在服务端的内存中使用Seesion存储登录信息，伴随而来的是可扩展性问题。

3.CORS(跨域资源共享)：当我们需要让数据跨多台移动设备上使用时，跨域资源的共享会是一个让人头疼的问题。在使用Ajax抓取另一个域的资源，就可以会出现禁止请求的情况。

4.CSRF(跨站请求伪造)：用户在访问银行网站时，他们很容易受到跨站请求伪造的攻击，并且能够被利用其访问其他的网站。

基于Token的身份验证的过程如下:

1.用户通过用户名和密码发送请求。

2.程序验证。

3.程序返回一个签名的token 给客户端。

4.客户端储存token,并且每次用于每次发送请求。

5.服务端验证token并返回数据。

每一次请求都需要token。token应该在HTTP的头部发送从而保证了Http请求无状态。我们同样通过设置服务器属性Access-Control-Allow-Origin:* ，让服务器能接受到来自所有域的请求。需要主要的是，在ACAO头部标明(designating)*时，不得带有像HTTP认证，客户端SSL证书和cookies的证书。
    
token是有时效的，一段时间之后用户需要重新验证。我们也不一定需要等到token自动失效，token有撤回的操作，通过token revocataion可以使一个特定的token或是一组有相同认证的token无效。
```

### 3 Django中cookie的使用

```python
1 设置cookie
# 四件套之一
  obj.set_cookie('key','value')
2 获取cookie
 request.COOKIES.get('key')
3 更新cookie
# 四件套之一
 obj.set_cookie('key','value')
4 删除cookie
  obj.delete_cookie('key')
    
5 cookie的过期时间
 -浏览器会管理cookie，到时间，它会自动删除，10s过期
 -obj.set_cookie('key','value',expires=10)
 # 如果不写 默认会话结束时过期 关闭浏览器，cookie就失效了
 --obj.set_cookie('key','value')  
    
6 设置加盐的cookie
    obj.set_signed_cookie('fk','yes','123',expires=1000)
    
7 获取加盐的cookie
  fk = request.get_signed_cookie('fk',salt='123')
```

### 4 Django中session的使用

```python
1 设置session
def set_session(request):
    # 默认存数据库：必须迁移，Django_session表
    # 设置session
    request.session['name'] = 'lqz'
    request.session['is_login'] = True
    '''
    1 生成一个随机字符串afdsfadsfs，一旦有，就是更新操作
    2 把随机字符串放到cookie中: obj.set_cookie('sessionid','afdsfadsfs')
    3 把name=lqz 放到django_session表中
    session_key   session_data    date
    afdsfadsfs    {name:'lqz',is_login:True}    时间
    '''


    return HttpResponse('设置session成功')

2 获取session
def get_session(request):
    # 取出携带的cookie
    # 在进视图函数之前，早已经把session从django_session表中session_data取出来
    # 转到了session中
    name = request.session.get('name')
    is_login = request.session.get('is_login')

    print(name)
    print(is_login,type(is_login))
    return HttpResponse('获取session成功')
3 更新 删除 session
def update_session(request):
    request.session['name'] = 'alex'
    # 删除session

    # request.session['is_login] = ''
    del request.session['is_login']  # None <class 'NoneType'>
    '''
    直接去django_session表中替换
    cookie还是原来的
    '''
    obj = HttpResponse('session 更新成功，删除login')

    return obj


# 删除session
request.session.delete()  # 删除数据库
request.session.flush()  # cookie和数据库都删
```

### 5 cookie，session其他了解知识

#### 5.1 cookie的其他参数

```python
key，健
value='',值
max_age=None，超时时间 cookie需要延续的时间（以秒为单位）如果参数是
\ None'' ，这个cookie会延续到浏览器关闭为止
expires=None,超时时间(IE requires expires, so set it if hasn't been already.)
                  
path='/',cookie生效的路径，/ 表示根路径，特殊的：根路径的cookie可以被任何url的页面访问，浏览器只会把cookie回传给带有该路径的页面，这样可以避免将cookie传给站点中的其他的应用。
domain=None,cookie生效的域名 你可以用这个参数来构造一个跨站cookie，如，domain=".example.com"所构造的cookie对下面这些站点都是可读的：
www.example.com 、 www2.example.com 和 an.other.sub.domain.example.com 。 如果该参数设置为None，cookie只能由设置它的站点读取
secure=False，浏览器将通过HTTPS来回传cookie
httponly=False 只能http协议传输，无法被Javascript获取（不是绝对，底层抓包可以获取到也可以被覆盖）
```

#### 5.2 session的其他方法

```python
# 获取、设置、删除session中数据
request.session['k1']
request.session.get('k1',None)
request.session['k1'] = 123
request.session.setdefault('k1',123)  # 存在则不设置
del request.session['k1']

# 所有 健、值、键值对
request.session.keys()
request.session.values()
request.session.items()
request.session.iterkeys()
request.session.itervalues()
request.session.iteritems()

# 会话session的key(随机字符串)
request.session.session_key

# 将所有session失效日期小于当前日期的数据删除
request.session.clear_expired()

# 检查会话session的key在数据库中是否存在
request.session.exists("session_key")

# 删除当前会话的所有session数据（只删数据库）
request.session.delete()

# 删除当前的会话数据并删除会话的cookie（数据库和cookie都删）
request.session.flush()
 这用于确保前面的会话数据不可以再次被用户的浏览器访问
 例如，djang.contrib.auth.logout() 函数中就会调用它
    
# 设置会话session和cookie的超时时间
request.session.set_expiry(value)
 * 如果value是个整数，session会在秒数后失效
 * 如果value是个datatime或timedelta，session就会在这个时间后失效
 * 如果value是0，用户关闭浏览器session就会失效
 * 如果value是None，session会依赖全局session失效策略

```

#### 5.3session的其他配置(配置文件中)

```python
1. 数据库Session
SESSION_ENGINE = 'django.contrib.sessions.backends.db'   # 引擎（默认）

2. 缓存Session
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 引擎
SESSION_CACHE_ALIAS = 'default'                            # 使用的缓存别名（默认内存缓存，也可以是memcache），此处别名依赖缓存的设置

3. 文件Session
SESSION_ENGINE = 'django.contrib.sessions.backends.file'    # 引擎
SESSION_FILE_PATH = None                                    # 缓存文件路径，如果为None，则使用tempfile模块获取一个临时地址tempfile.gettempdir() 

4. 缓存+数据库
SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'        # 引擎

5. 加密Cookie Session
SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'   # 引擎

其他公用设置项：
SESSION_COOKIE_NAME ＝ "sessionid"                       # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串（默认）
SESSION_COOKIE_PATH ＝ "/"                               # Session的cookie保存的路径（默认）
SESSION_COOKIE_DOMAIN = None                             # Session的cookie保存的域名（默认）
SESSION_COOKIE_SECURE = False                            # 是否Https传输cookie（默认）
SESSION_COOKIE_HTTPONLY = True                           # 是否Session的cookie只支持http传输（默认）
SESSION_COOKIE_AGE = 1209600                             # Session的cookie失效日期（2周）（默认）
SESSION_EXPIRE_AT_BROWSER_CLOSE = False                  # 是否关闭浏览器使得Session过期（默认）
SESSION_SAVE_EVERY_REQUEST = False                       
# 是否每次请求都保存Session，默认修改之后才保存（默认）
```

### 补充

```python
1 方法和函数
 - 绑定给对象的方法，如果类来调用，它就是普通函数，有几个传几个
```

### 2 如何看源码

#### 2.1 快速定位到当前py文件

<img src="C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\image-20211010210539480.png" alt="image-20211010210539480"  />

#### 2.2 查看当前py文件中有哪些类，哪些方法

![image-20211010211333262](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\image-20211010211333262.png)

### 作业

必写：

1 登录功能，如果登录成功往cookie中写入用户名和登录成功的状态标志

2 访问order页面，如果登录了，可以正常显示，如果没登录，重定向到登录页面

扩展作业：

1 什么是集群 什么是分布式

集群：为解决某个特定问题将多台计算机组合起来形成的单个系统

2 内网穿透  公网地址

3 什么是反向代理，什么是正向代理 Nginx反向代理服务器

4 JWT

5 csrf 跨站请求伪造

## 昨日回顾

```python
1 forms局部和全局钩子的源码
 -is_valid  --> self.erros --> self.full_clean()
 self._clean_fields()  # 字段自己的校验规则和局部钩子执行
 self._clean_form()  # 全局钩子执行
 self._post_clean()
    
2 cookie，session和token
 -cookie是存在浏览器里的键值对
 -session是存在服务端的键值对
 -token是为了不把数据存在服务端又要保证数据的安全，新出的一种认证方式。它有三段。
3 Django中使用cookie
 -HttpResponse的对象.set_cookie(key,value,expire=30)
 -request.COOKIES.get()
 -更新
 -HttpResponse的对象，delete_cookie()
4 cookie的其他参数
 -过期时间
 -httponly
 -path /index
    
5 session的使用
 -request.session['name']=lqz(原来有，原来没有)
 -request.session.get('name')
 -del request.session['name']
    
6 session的其他
 -对象的其他方法
 -配置：过期时间，sessionid...
```

## 今日内容

### 1 cbv加装饰器

```python
总结：
 1-cbv加装饰器可以加在类上
 @method_decorator(auth,name='get')  # 给get请求加装饰器
 2-可以加在方法上
 @method_decorator(auth)
 def get(self, request, *args, **kwargs):
```

```python
基于类的视图

def auth(func):
    def inner(request, *args, **kwargs):
        # 登录校验
        if request.session.get('is_login'):  # 要用session必须迁移
            res = func(*args, **kwargs)
            return res
        else:
            return redirect('/login')

    return inner


from django.views import View

from django.utils.decorators import method_decorator

# @method_decorator(auth,name='get')  # 给get请求加装饰器
class Index(View):
    @method_decorator(auth)
    def get(self, request, *args, **kwargs):
        return HttpResponse('index')

    def post(self, request, *args, **kwargs):
        return HttpResponse('post_index')


def login(request):
    return render(request,'login.html')
```

### 2 正向代理和反向代理

```python
正向代理代理客户端  客户端  需要配置代理服务器地址 正向代理服务器  互联网
反向代理代理服务器  客户端  反向代理服务器  WEB服务器
https://www.cnblogs.com/liuqingzheng/p/10521675.html
正向代理即是客户端代理, 代理客户端, 服务端不知道实际发起请求的客户端.
反向代理即是服务端代理, 代理服务端, 客户端不知道实际提供服务的服务端
正向代理: 买票的黄牛
反向代理: 租房的代理
```

### 3 中间件介绍和常用内置中间件

```python
0 中间件：数据库中间件(mycat，分库分表)，服务器中间件(tomcat,nginx)，消息队列中间件(rabbitmq)
1 Django中间件(Middleware)：介于request与response处理之间的一道处理过程，相对比较轻量级，并且在全局上改变django的输入与输出。
Middleware is a framework of hooks into Django’s request/response processing. 
It’s a light, low-level “plugin” system for globally altering Django’s input or output.

```

### 3.1 Django内置中间件

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    # 处理session
    'django.contrib.sessions.middleware.SessionMiddleware',
    # 处理是否带斜杠的
    'django.middleware.common.CommonMiddleware',
    # 跨站请求伪造的处理
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
# SessionMiddleware源码
django.contrib.sessions.middleware.SessionMiddleware
process_request(self,request)  # 请求来了
process_response(self,request,response)  # 请求走了会触发
```

### 4 自定义中间件

```python
# 4.1 process_request
1 请求来了触发它，从上往下依次执行（在settings中间件注册的位置）
    def process_request(self,request):
    如果返回None,就继续往下走
    如果返回四件套之一，直接就回去了

2 在这里写请求来了的一些判断
3 request.META.REMOTE_ADDR  # 客户端地址
4 request.META.HTTP_USER_AGENT  # 客户端类型

# 4.2 process_response
1 请求走了，会触发它，从下往上执行
    def process_response(self,request,response):
        print('请求走了')
        return response  # 一定要return response
    
2 在所有的响应中都写入cookie name=lqz   response.set_cookie('name','lqz')
3 在所有的响应中都写入 response['x-head'] = 'xxx'

# 4.3 process_view（了解）
process_view(self, request, view_func, view_args, view_kwargs)
Django会在调用视图函数之前调用process_view方法。

它应该返回None或一个HttpResponse对象。如果返回None，Django将继续处理这个请求，执行任何其他中间件的process_view方法，然后在执行相应的视图。如果它返回一个HttpResponse对象，Django不会调用适当的视图函数。它将执行中间件的process_response方法并将应用到该HttpResponse并返回结果

  def process_view(self, request, view_func, view_args, view_kwargs):
        # view_func 视图函数
        # view_args 位置参数
        # view_kwargs 关键字参数
        print('我是process view')

        # 如果return None, 会执行视图函数
        # 手动执行视图函数
        # response = view_func(request,view_args, view_kwargs)
        # 返回response，视图函数就不执行了
        # return response
        return HttpResponse('PROCESS VIEW')

# 4.4 process_exception(了解)
这个方法只有在视图函数中出现异常了才执行

    def process_exception(self, request, exception):
        # 记录错误日志
        print(exception)

# 4.5 process_template_response(了解)
该方法对视图函数返回值有要求，必须是一个含有render方法类的对象，才会执行此方法

    def process_template_response(self,request,response):
        print('我执行了')
        return response
    
class Test:
    def __init__(self,status,msg):
        self.status=status
        self.msg=msg
    def render(self):
        import json
        dic={'status':self.status,'msg':self.msg}

        return HttpResponse(json.dumps(dic))
def index(response):
    return Test(True,'测试')
        
```

### 5 CSRF_TOKEN跨站请求伪造

```html
https://www.cnblogs.com/liuqingzheng/p/9505044.html
CSRF（Cross-site request forgery）跨站请求伪造，也被称为“One Click Attack”或者Session Riding，通常缩写为CSRF或者XSRF，是一种对网站的恶意利用。尽管听起来像跨站脚本（XSS），但它与XSS非常不同，XSS利用站点内的信任用户，而CSRF则通过伪装来自受信任用户的请求来利用受信任的网站。与XSS攻击相比，CSRF攻击往往不大流行（因此对其进行防范的资源也相当稀少）和难以防范，所以被认为比XSS更具危险性

1 CSRF和XSRF
2 攻击者盗用了你的身份，以你的名义发送恶意请求，对服务器来说这个请求是完全合法的

3 CSRF攻击防范
（1）验证 HTTP Referer 字段  上一次访问的地址(图片防盗链)
（2）在请求地址中添加 token 并验证
（3）在 HTTP 头中自定义属性并验证
把随机字符串放在请求体中

在form表单中应用：<form action="" method="post">{% csrf_token %}
在Ajax中应用：
 # 放在data里：
     'csrfmiddlewaretoken': $('[name="csrfmiddlewaretoken"]').val()
     'csrfmiddlewaretoken': '{{csrf_token}}'
    
    
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script src="/static/jquery-3.3.1.js"></script>
    <title>Title</title>
</head>
<body>
<form action="" method="post">
    {% csrf_token %}
    <p>用户名：<input type="text" name="name"></p>
    <p>密码：<input type="text" name="password" id="pwd"></p>
    <p><input type="submit"></p>
</form>
<button class="btn">点我</button>
</body>
<script>
    $(".btn").click(function () {
        $.ajax({
            url: '',
            type: 'post',
            data: {
                'name': $('[name="name"]').val(),
                'password': $("#pwd").val(),
                'csrfmiddlewaretoken': $('[name="csrfmiddlewaretoken"]').val()
            },
            success: function (data) {
                console.log(data)
            }

        })
    })
</script>
</html>
    
    
# 放在cookie里：

获取cookie：document.cookie

是一个字符串，可以自己用js切割，也可以用jquery的插件

获取cookie：$.cookie(‘csrftoken’)

设置cookie：$.cookie(‘key’,’value’)

    
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script src="/static/jquery-3.3.1.js"></script>
    <script src="/static/jquery.cookie.js"></script>
    <title>Title</title>
</head>
<body>
<form action="" method="post">
    {% csrf_token %}
    <p>用户名：<input type="text" name="name"></p>
    <p>密码：<input type="text" name="password" id="pwd"></p>
    <p><input type="submit"></p>
</form>
<button class="btn">点我</button>
</body>
<script>
    $(".btn").click(function () {
        var token=$.cookie('csrftoken')
        //var token='{{ csrf_token }}'
        $.ajax({
            url: '',
            headers:{'X-CSRFToken':token},
            type: 'post',
            data: {
                'name': $('[name="name"]').val(),
                'password': $("#pwd").val(),
            },
            success: function (data) {
                console.log(data)
            }

        })
    })
</script>
</html>
```

### 6 Django中处理csrf

```html
1 中间件不要注释了
2 如果是form表单
<form action="" method="post">
    {% csrf_token %}
 <p>给谁转：<input type="text" name="to_user" id="id_name"></p>
 <p>转多少：<input type="text" name="money" id="id_money"></p>
    <input type="submit" value="转账">
</form>

3 如果是ajax请求：两种方案
 'csrfmiddlewaretoken': $('[name="csrfmiddlewaretoken"]').val()
 'csrfmiddlewaretoken': '{{csrf_token}}'

```

#### 6.1 cookie的处理

```python
<script src="/static/jquery.cookie.js"></script>

var token=$.cookie('csrftoken')
```

#### 6.2 csrf放到请求头中

```python
$.ajax({
            url: '',
            headers:{'X-CSRFToken':token},
            type: 'post',
            data: {
                'name': $('[name="name"]').val(),
                'password': $("#pwd").val(),
            },
            success: function (data) {
                console.log(data)
            }

        })
```

#### 6.3 其他操作

```python
全站禁用：注释掉中间件 ‘django.middleware.csrf.CsrfViewMiddleware’,

局部禁用：用装饰器（在FBV中使用）

from django.views.decorators.csrf import csrf_exempt,csrf_protect
# 不再检测，局部禁用（前提是全站使用）
# @csrf_exempt
# 检测，局部使用（前提是全站禁用）
# @csrf_protect
def csrf_token(request):
    if request.method=='POST':
        print(request.POST)

        return HttpResponse('ok')
    return render(request,'csrf_token.html')


在CBV中使用：

# CBV中使用
from django.views import View
from django.views.decorators.csrf import csrf_exempt,csrf_protect
from django.utils.decorators import method_decorator
# CBV的csrf装饰器，只能加载类上（指定方法为dispatch）和dispatch方法上（django的bug）
# 给get方法使用csrf_token检测
@method_decorator(csrf_exempt,name='dispatch')
class Foo(View):
    def get(self,request):
        pass
    def post(self,request):
        pass
```

## 作业

```python
1 通过中间件实现访问控制，未登录用户重定向到登录，登录用户才能访问home页面
2 通过中间件实现用户名，用户ip地址和客户端类型，登录时间记录
3 使用bootstrap构建一个table页面，展示2题中记录的数据，支持分页
4 301和302有什么区别？


扩展（先调用，周末实现）
1 根据客户端类型，判断浏览器类型，按月统计浏览器类型占比，饼形图展示
2 按月统计日活用户数，用折线图展示
3 图片防盗链
```

## 昨日回顾

```python
1 中间件：request对象，response
 -process_request
  -返回None,继续往下走
  -返回response对象，直接返回
 -process_response
  -处理response
 -定义一个类，继承MiddlewareMixin
 -配置文件配置（执行顺序）
2 中间件能干什么
 -全局的登录认证
 -记录日志（访问日志，登录日志，客户端类型...）
 -对ip地址进行限制（一分钟访问十次）
    
3 csrf，xsrf：跨站请求伪造

4 Django中处理csrf攻击
 1 form表单形式 {% csrf_token %}
 2 ajax提交：手动传，2种方式
 3 ajax提交：放到请求头中  headers:{'X-CSRFToken':'sss','name':'lqz'}
        
4 请求头中的数据，Django如何获取
 -request.META.get('HTTP_字段名大写')
    
5 Django的csrf的中间件
 -全局使用csrf，中间件
 -局部禁用：在视图函数上加一个装饰器csrf_exempt
 -全局禁用：中间件注释
 -局部使用：在视图函数上加一个装饰器csrf_protect
```

## 今日内容

### 1 anth组件介绍

```python
1 django提供的用户认证，创建，修改密码... 用户相关操作
2 不需要创建用户表，默认带了
3 插入数据(创建用户)：
 python manage.py createsuperuser
```

### 2 auth组件常用方法

#### 2.1 authenticate()

```python
提供了用户认证功能，即验证用户名以及密码是否正确，一般需要username 、password两个关键字参数。

如果认证成功（用户名和密码正确有效），便会返回一个 User 对象。

authenticate()会在该 User 对象上设置一个属性来标识后端已经认证了该用户，且该信息在后续的登录过程中是需要的。

user = authenticate(username='usernamer',password='password')

```

#### 2.2 login(HttpRequest,user)

```python
该函数接受一个HttpRequest对象，以及一个经过认证的User对象。
该函数实现一个用户登录的功能。它本质上会在后端为该用户生成相关session数据。
from django.contrib.auth import authenticate, login
   
def my_view(request):
  username = request.POST['username']
  password = request.POST['password']
  user = authenticate(username=username, password=password)
  if user is not None:
    login(request, user)
    # Redirect to a success page.
    ...
  else:
    # Return an 'invalid login' error message.
    ...
```

#### 2.3 logout(request)

```python
该函数接受一个HttpRequest对象，无返回值。
当调用该函数时，当前请求的session信息会全部清除。该用户即使没有登录，使用该函数也不会报错。
from django.contrib.auth import logout
   
def logout_view(request):
  logout(request)
  # Redirect to a success page.
```

#### 2.4 is_authenticated()

```python
用来判断当前请求是否通过了认证。
def my_view(request):
  if not request.user.is_authenticated():
    return redirect('%s?next=%s' % (settings.LOGIN_URL, request.path))


```

#### 2.5 login_requierd()

```python
auth 给我们提供的一个装饰器工具，用来快捷的给某个视图添加登录校验。
from django.contrib.auth.decorators import login_required
      
@login_required
def my_view(request):
  ...

如果需要自定义登录的URL，则需要在settings.py文件中通过LOGIN_URL进行修改。
LOGIN_URL = '/login/'  # 这里配置成你项目登录页面的路由

```

#### 2.6 create_user()

```python
auth 提供的一个创建新用户的方法，需要提供必要参数（username、password）等。
from django.contrib.auth.models import User
user = User.objects.create_user（username='用户名',password='密码',email='邮箱',...）
```

#### 2.7 create_superuser()

```python
from django.contrib.auth.models import User
user = User.objects.create_superuser（username='用户名',password='密码',email='邮箱',...）

```

#### 2.8 check_password(password)

```python
auth 提供的一个检查密码是否正确的方法，需要提供当前请求用户的密码。

密码正确返回True，否则返回False。

ok = user.check_password('密码')

```

#### 2.9 set_password(password)

```python
auth 提供的一个修改密码的方法，接收 要设置的新密码 作为参数。

注意：设置完一定要调用用户对象的save方法！！！

user.set_password(password='')
user.save()
```

### 3 User对象的属性

```python
User对象属性：username， password

is_staff ： 用户是否拥有网站的管理权限. 如果没有，后台admin登录不进去

is_active ： 是否允许用户登录, 设置为 False，可以在不删除用户的前提下禁止用户登录。
```

### 4 扩展默认的auth_user表

```python
# 第一种方案 一对一
1 新建另外一张表然后通过一对一和内置的auth_user表关联
from django.contrib.auth.models import User
class  USERDETAIL(models.Model):
    user = models.OneToOneField(to=User, on_delete=models.CASCADE)
    phone = models.CharField(max_length=32)
  
# 第二种方案 继承 AbstractUser类来扩写
    class Meta:
        abstract = True  # 这个表是个抽象表，只用来继承，不会生成
      
# 使用步骤
 -大前提是auth_user表没有创建之前干*****
 -写一个类，继承 AbstractUser
 -在类中写扩写字段(可以重写原来有的字段)
 class MyAuthUser(AbstractUser):
    phone = models.CharField(max_length=32)
 -在配置文件中配置 settings
  # 引用Django自带的User表，继承使用时需要设置
  AUTH_USER_MODEL = "app01.MyAuthUser"
        
2 继承内置的 AbstractUser 类，来定义一个自己的Model类
from django.contrib.auth.models import AbstractUser
class UserInfo(AbstractUser):
    """
    用户信息表
    """
    nid = models.AutoField(primary_key=True)
    phone = models.CharField(max_length=11, null=True, unique=True)
    
    def __str__(self):
        return self.username
    
# 引用Django自带的User表，继承使用时需要设置
AUTH_USER_MODEL = "app名.UserInfo"

一旦我们指定了新的认证系统所使用的表，我们就需要重新在数据库中创建该表，而不能继续使用原来默认的auth_user表了。

# 如果auth_user表已经有了，还想扩写
 - 删库 数据库
 - 清空项目中所有migration的记录
 - 清空源码中admin，auth 两app的migration的记录
```

## 作业

```python
1 基于auth继承abstractUser类，加一个phone字段，实现用户的注册登录，登录成功，在home页面显示用户名，未登录，显示登录注册
2 实现修改密码功能
3 超级管理员能够禁用用户登录
```

## 补充

### 1 Django启动流程

```python
1 配置文件中指定 WSGI_APPLICATION = 'djangoCBV.wsgi.application'
2 被wsgi服务器管理，一旦有请求进来，会触发application()
3 实际触发WSGIHandler类的__call__传入environ, start_response
4 把environ包装成request对象，执行中间件，执行路由，执行视图函数，返回response
5 最终结束Django
```

## 昨日回顾

```python
1 auth组件，登录，注册，注销， APP
2 创建几个表：anth_user, 权限相关的表，rbac:基于角色的权限控制  内部
3 9个方法
 - authenticate
 - login()  # 写 session
 - request.user.is_authenticated
 - request.user
 - logout
 - login_require() # 装饰器
 - create_superuser
 - create_user
 - check_password
 - set_password
4 扩写user表
 - 一对一
 - 继承 AbstracrUser 在settings中配置
```

## 今日内容

### 1 项目开发流程

```python
1 项目分类
 - 针对互联网用户：抖音、淘宝  sanic
  - 产品经理
 - 公司内部，给用户定制软件  CMDB 自动运维平台 OA  驻场开发 封闭开发
2 项目开发模式分类
 - 瀑布开发模式
 - 敏捷开发：devops ci cd
    
3 项目开发流程
 - 立项
 - 需求分析(产品经理提，用户提需求)
 - 原型图(流程图，产品经理设计)
 - 美工切图
 - 技术选型，数据库架构设计
 - 前后台(端)开发（协同开发：git）
 - 开发完成 -- 运维上线 -- 联调（测试环境）
 - 测试测试
 - 修改bug（开发）
 - 上线运行
 
 - 迭代更新（开发）
    
4 多人博客项目（仿cnblogs），功能，需求(前后端混合)
 - 注册（forms，ajax提交，上传头像）
 - 登录功能（ajax提交，错误信息渲染）
 - 首页（列出所有文章，作者头像，发布时间，点赞数，广告位，轮播图）
 - 个人站点 （左侧过滤，inclusion_tag）
 - 文章展示 页面
 - 点赞，点踩功能
 - 评论功能
 - 后台管理：展示我的所有文章
 - 文章新增（修改，删除），防止XSS攻击
 - 修改密码，头像，个人信息（不讲）
```

售前工程师 售后工程师 运维 开发 测试 产品经理  - 对接联调

### 2 bbs表分析

```python
# https://gitee.com/xuexianqi/django_blog
# 设计程序（django2.2.2+mysql5.7)
# 数据库设计（设计表）
# 所有的表
 - 用户表（auth的扩展）
     -头像字段
  -blog字段（关联字段 一对一）
 - 博客表（个人站点）：跟用户一对一
  -博客标题
  -博客名称
  -博客样式

 - 文章表
  -文章标题
  -文章摘要
  -文章内容
  -创建时间
  -user_id(一对多)
  -分类id(一对多)
  -标签多对多关系（没有字段）
 - 分类表
     -分类id
  -分类名字
  -blog字段(一对多)
 - 标签表
  -标签id
  -标签名字
  -blog字段(一对多)
 - 点赞点踩表
  -user_id（一对多）
  -article_id（文章 一条记录 一对多）
  -is_up:点赞点踩字段
  -时间
 - 评论表
  -user_id
  -article_id
  -评论内容
        
# 自关联
评论id  用户id  文章id  评论内容       评论id 
1    1  2  写的真好  null
2    2  1  明明写的不好  1
3   1  1  你他吗去死   2
```

### 3 bbs表模型创建

### 4 注册forms类编写

### 5 注册功能前端搭建

### 6 头像实时显示

### 7 注册功能前后端

## 作业

## 补充

```python
1 Django：Django的orm，rbac，admin
 -bug级的存在，几乎不用写代码，就可以撸出一个后台管理系统
 -xadmin:Django3.o以后，作者弃坑，不管了 1.x版本 2.x版本 可以用
 -3.x版本：Simple UI
2 flask
----------同步框架----------
------Django3.x以后支持异步------
------一旦用了异步，所有的都要用异步------
 -python线程中只要有io操作，就会让出GIL锁，这条线程下次要执行必须再拿到gill
 -协程：单线程下实现并发，程序员自己控制切换
 -

------往下的是异步框架------
3 tornado：2.x
4 FastAPI：操作数据库
5 Sanic：python3.5以上，不支持Windows

6 到目前为止没有一个好用的异步orm框架
7 异步操作mysql Redis MongoDB

Mac 乌班图（台式机） 使用Windows远程连接Linux开发 纯Windows

```

## 昨日回顾

```python
1 软件开发模式：瀑布开发，敏捷开发（ci cd）
2 DevOps：开发，测试，运维
3 开发流程
 -立项
 -需求分析
 -设计程序架构，数据库，产品经理做原型图，美工切图
 -分任务开发（前端，后端），协同开发（git)
 -测试
 -正式上线
 ----------
 -迭代更新
    
4 7张表+1张中间表
 
5 Django中连接mysql数据库  PostgreSQL SQLite 
 -python 2 的版本， mysqldb
 -python 3 的版本， mysqldb不维护了，pymysql出现，mysqlclient
 -pymysql:并不是Django原生支持的，使用需要加点东西
 -mysqlclient：不需要加任何东西，跟Django无缝衔接
  -模块经常装不上
  -win Mac Linux
        
6 如果win平台模块装不上的解决方案
 -使用whl文件安装
  - pip install wheel
  - 去下载平台，python版本对应的whl文件
  - pip install 把文件拖入即可
        
7 注册功能
 -form类
 - 注册页面form渲染
           
```

## 今日内容

### 1 注册功能前端

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
    <style>
        .danger {
            color: red;
        }
    </style>
</head>
<body>
<div class="container-fluid">
    <div class="row">
        <div class="col-md-6 col-md-offset-3">
            <div class="panel panel-primary">
                <div class="panel-heading">
                    <h3 class="panel-title">注册功能</h3>
                </div>
                <div class="panel-body">
                    <form id="my_form">
                        {% csrf_token %}
                        {% for item in form %}
                            <div class="form-group">
                                <label for="{{ item.auto_id }}">{{ item.label }}</label>
                                {{ item }}
                                <span class="danger pull-right error"></span>
                            </div>
                        {% endfor %}
                        <div class="form-group">
                            <label for="myfile">头像
                                <img src="/static/img/default.png" id="id_img" width="80px" height="80px"
                                     style="margin-left: 20px">
                            </label>
                            <input type="file" id="myfile" style="display: none">
                        </div>
                        <div class="text-center">
                            <input type="button" value="注册" class="btn btn-warning" id="id_submit"><span class="danger error"
                                                                                                         id="id_error"
                                                                                                         style="margin-left: 10px">

                        </span>
                        </div>

                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
</body>
```

### 2 注册功能后端

```python
from django.shortcuts import render
from django.http import JsonResponse
# Create your views here.
from blog.blog_forms import RegisterForm
from blog import models


def register(request):
    if request.method == 'GET':
        register_form = RegisterForm()
        return render(request,'register.html', context={'form': register_form})
    elif request.method == 'POST':
        response = {'status': 100, 'msg': None}
        register_form = RegisterForm(request.POST)
        if register_form.is_valid():
            # 数据校验通过，存
            # 取出校验通过的数据
            # 可能传头像，也可能没传头像
            clean_data = register_form.cleaned_data
            print(clean_data)
            my_file = request.FILES.get('myfile')
            if my_file:  # 传了头像
                # FileField字段类型直接接收一个文件对象，
                # 它会把文件存到upload_to='avatar/'，然后把路径存到数据库中
                # 相当于with open 打开文件，把文件存到avatar路径下，把路径赋值给avatar这个字段
                clean_data['avatar'] = my_file
            clean_data.pop('re_password')
            models.UserInfo.objects.create_user(**clean_data)
            response['msg'] = '恭喜你，注册成功'
            response['next_url'] = '/login/'
        else:
            # 校验不通过
            response['status'] = 101
            response['msg'] = register_form.errors
        return JsonResponse(response)

```

### 3 注册功能前端错误渲染

```python
<script>
    // 存文件的input如果发生变化
    $('#myfile').change(function () {
        // 取到文件
        let myfile = $(this)[0].files[0]
        // let myfile = $('#myfile')
        // 借助文件阅读器对象，把文件读到这个对象中
        let filereader = new FileReader()
        filereader.readAsDataURL(myfile)

        // 等待读完，把它放到img标签上
        filereader.onload = function () {
            $('#id_img').attr('src', filereader.result)
        }

    })

    /*
    $('#id_username').blur(function () {
        alert(111)
        $(this).val()
    })

     */

    // 注册功能
    $('#id_submit').click(function () {
        let formdata = new FormData()
        formdata.append('myfile', $('#myfile')[0].files[0])
        // 方案一
        /*
        formdata.append('username',$('#id_username').val())
        formdata.append('password',$('#id_password').val())
        formdata.append('re_password',$('#id_re_password').val())
        formdata.append('email',$('#id_email').val())

         */

        // 方案二 serializeArrayk可以把form表单里的key value 取出来转到一个列表中
        let form_data = $('#my_form').serializeArray()  
        $.each(form_data, function (index, element) {
            // console.log(index)
            // console.log(element)
            formdata.append(element.name, element.value)
        })


        $.ajax({
            url: '/register/',
            method: 'post',
            contentType: false,
            processData: false,
            data: formdata,
            success: function (data) {
                console.log(data)
                if (data.status == 100) {
                    location.href = data.next_url
                    //location.href = '/login/'
                } else {
                    $.each(data.msg, function (key, value) {
                        if (key == '__all__') {
                            $('#id_error').html(value[0])
                        } else {
                            // 取到input标签的下一个
                            // $('#id_' + key).next().html(value[0])
                            // 链式调用
                            //$('#id_' + key).parent().addClass('has-error')
                            $('#id_' + key).next().html(value[0]).parent().addClass('has-error')
                        }

                    })

                    // 定时器 3秒后 清空错误提示信息
                    setTimeout(function () {
                        // 清除红色框
                        $('.form-group').removeClass('has-error')
                        // 清空所有错误信息
                        $('.error').html('')

                    }, 3000)
                }


            }
        })
    })
</script>
```

### 4 登录页面搭建

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录页面</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>
<div class="container-fluid">
    <div class="row">
        <div class="col-md-6 col-md-offset-3">
            <div class="panel panel-primary">
                <div class="panel-heading">
                    <h3 class="panel-title">登录功能</h3>
                </div>
                <div class="panel-body">
                    <form id="my_form">
                        {% csrf_token %}

                        <div class="form-group">
                            <label for="">用户名</label>
                            <input type="text" id="id_username" class="form-control">
                            <span class="danger pull-right error"></span>
                        </div>
                        <div class="form-group">
                            <label for="">密码</label>
                            <input type="password" id="id_password" class="form-control">
                            <span class="danger pull-right error"></span>
                        </div>

                        <div class="form-group">
                            <label for="">验证码</label>

                            <div class="row">
                                <div class="col-md-6">
                                    <input type="text" id="id_code" class="form-control">
                                </div>
                                <div class="col-md-6">
                                    <img src="/get_code/" alt="" id="id_img" class="img-responsive center-block"> <!-- 在此处写注释 height="45px" width="250px" -->
                                </div>
                            </div>
                        </div>


                        <div class="text-center">
                            <input type="button" value="登录" class="btn btn-warning" id="id_submit"><span
                                class="danger error"
                                id="id_error"
                                style="margin-left: 10px">
                            </span>
                        </div>

                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
</body>
<script>
    $('#id_img').click(function () {
        let img_url = $('#id_img')[0].src
        $('#id_img')[0].src = img_url + '?'
    })

    $('#id_submit').click(function () {
        $.ajax({
            url: '/login/',
            method: 'post',
            data: {
               username: $('#id_username').val(),
               password: $('#id_password').val(),
               code: $('#id_code').val(),
               csrfmiddlewaretoken: '{{csrf_token}}'
            },
            success: function (data) {
                console.log(data)
                console.log(data.status)
                if (data.status == 100) {
                    //Location.href = '/index/'
                    //Location.href = data.url
                    location.href = data.url
                    //window.returnValue=false
                    //Location.href = 'http://127.0.0.1:8000/index/'
                } else {
                    $('#id_error').html(data.msg)
                }
            }
        })
    })
</script>
</html>
```

### 5 手写验证码（第三方）

```python
1 生成验证码的模块
2 集成第三方，极验滑动验证，腾讯验证码（SDK）
3 自己写
```

### 6 登录功能前后端

```python
def login(request):
    if request.method == 'GET':
        return render(request, 'login.html')
    else:
        response = {'status': 100, 'msg': None}
        username = request.POST.get('username')
        password = request.POST.get('password')
        code = request.POST.get('code')
        # 验证码可能为None 需要处理
        # if code != None:
        # print(request.POST)
        # print(request.session.get('verification_code'))
        # print(code)
        if code.upper() == request.session.get('verification_code').upper():

            # if code == request.session.get('code'):
            user = authenticate(username=username, password=password)
            if user:
                auth.login(request, user)

                response['msg'] = '登录成功'
                response['url'] = '/index/'
            else:
                response['status'] = 102
                response['msg'] = '用户名或密码错误'
        else:
            response['status'] = 101
            response['msg'] = '验证码错误'

        return JsonResponse(response)
```

## 作业

```python
1 当用户输入用户名，光标离开，就会校验用户名是否存在
```

## 补充

### jQuery的each用法

```python
$.each([52, 97], function(index, value) {
    alert(index + ": " + value);
});

$( "li" ).each(function( index ) {
  console.log( index + ": " + $( this ).text() );
});
```

```

```

```
 height="45px" width="250px"
```

在同一个浏览器里打开多次页面，之前打开的就失效了，

但是，换到不同浏览器里没关系

## 上节回顾

```python
1 注册
2 登录
3 验证码生成
4 验证码刷新
5 首页布局
```

## 今日内容

### 1 首页布局

```python
 <span>{{ artcle.commit_set.count }}</span>
    
前端
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>首页</title>
    <link rel="stylesheet" href="/static/font-awesome-4.7.0/css/font-awesome.min.css">
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
    <style>
        .article_footer {
            margin-right: 10px;
        }
    </style>
</head>
<body>
<div class="container-fluid">
    <div class="head">
        <nav class="navbar navbar-default">
            <div class="container-fluid">
                <!-- Brand and toggle get grouped for better mobile display -->
                <div class="navbar-header">
                    <button type="button" class="navbar-toggle collapsed" data-toggle="collapse"
                            data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
                        <span class="sr-only">Toggle navigation</span>
                        <span class="icon-bar"></span>
                        <span class="icon-bar"></span>
                        <span class="icon-bar"></span>
                    </button>
                    <a class="navbar-brand" href="#">博客园</a>
                </div>

                <!-- Collect the nav links, forms, and other content for toggling -->
                <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
                    <ul class="nav navbar-nav">
                        <li class="active"><a href="#">首页 <span class="sr-only">(current)</span></a></li>
                        <li><a href="#">新闻</a></li>
                        <li class="dropdown">
                            <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button"
                               aria-haspopup="true" aria-expanded="false">Dropdown <span class="caret"></span></a>
                            <ul class="dropdown-menu">
                                <li><a href="#">Action</a></li>
                                <li><a href="#">Another action</a></li>
                                <li><a href="#">Something else here</a></li>
                                <li role="separator" class="divider"></li>
                                <li><a href="#">Separated link</a></li>
                                <li role="separator" class="divider"></li>
                                <li><a href="#">One more separated link</a></li>
                            </ul>
                        </li>
                    </ul>
                    <form class="navbar-form navbar-left">
                        <div class="form-group">
                            <input type="text" class="form-control" placeholder="Search">
                        </div>
                        <button type="submit" class="btn btn-default">Submit</button>
                    </form>
                    {% if request.user.is_authenticate %}
                        <ul class="nav navbar-nav navbar-right">
                            <li><a href="#">{{ request.user.username }}</a></li>
                            <li class="dropdown">
                                <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button"
                                   aria-haspopup="true" aria-expanded="false">更多操作 <span class="caret"></span></a>
                                <ul class="dropdown-menu">
                                    <li><a href="#">修改密码</a></li>
                                    <li><a href="#">修改头像</a></li>
                                    <li><a href="#">后台管理</a></li>
                                    <li role="separator" class="divider"></li>
                                    <li><a href="/logout/">退出</a></li>
                                </ul>
                            </li>
                        </ul>
                    {% else %}
                        <ul class="nav navbar-nav navbar-right">
                            <li><a href="/login/">登录</a></li>
                            <li><a href="/register/">注册</a></li>
                        </ul>
                    {% endif %}

                </div><!-- /.navbar-collapse -->
            </div><!-- /.container-fluid -->
        </nav>

    </div>

    <div class="body row">
        <div class="col-md-2">
            <div class="panel panel-success">
                <div class="panel-heading">
                    <h3 class="panel-title">广告位招租</h3>
                </div>
                <div class="panel-body">
                    寻宝探险：<a href="">点我找宝藏</a>
                </div>
            </div>
            <div class="panel panel-info">
                <div class="panel-heading">
                    <h3 class="panel-title">天天说书</h3>
                </div>
                <div class="panel-body">
                    <ul class="list-group">
                        <li class="list-group-item">红楼梦</li>
                        <li class="list-group-item">西游记</li>
                        <li class="list-group-item">三国演义</li>
                        <li class="list-group-item">水浒传</li>
                        <li class="list-group-item">追忆似水年华</li>
                    </ul>
                </div>
            </div>
        </div>
        <div class="col-md-7">
            <div>

                <div id="carousel-example-generic" class="carousel slide" data-ride="carousel">
                    <!-- Indicators -->
                    <ol class="carousel-indicators">
                        <li data-target="#carousel-example-generic" data-slide-to="0" class="active"></li>
                        <li data-target="#carousel-example-generic" data-slide-to="1"></li>
                        <li data-target="#carousel-example-generic" data-slide-to="2"></li>
                    </ol>

                    <!-- Wrapper for slides -->
                    <div class="carousel-inner" role="listbox">
                        {% for banner in banner_list %}
                            {% if forloop.first %}
                                <div class="item active">
                                    <img src="{{ banner.url }}" alt="...">
                                    <div class="carousel-caption">
                                        {{ banner.name }}
                                    </div>
                                </div>
                            {% else %}
                                <div class="item ">
                                    <img src="{{ banner.url }}" alt="...">
                                    <div class="carousel-caption">
                                        {{ banner.name }}
                                    </div>
                                </div>
                            {% endif %}

                        {% endfor %}


                    </div>

                    <!-- Controls -->
                    <a class="left carousel-control" href="#carousel-example-generic" role="button" data-slide="prev">
                        <span class="glyphicon glyphicon-chevron-left" aria-hidden="true"></span>
                        <span class="sr-only">Previous</span>
                    </a>
                    <a class="right carousel-control" href="#carousel-example-generic" role="button" data-slide="next">
                        <span class="glyphicon glyphicon-chevron-right" aria-hidden="true"></span>
                        <span class="sr-only">Next</span>
                    </a>
                </div>
            </div>
            <div class="article" style="margin-top: 20px">
                {% for article in article_list %}
                    <div>
                        <h4 class="media-heading"><a href="">{{ article.title }}</a></h4>
                        <div class="media">
                            <div class="media-left media-middle">
                                <a href="#">
                                    <img class="media-object" src="media/{{article.blog.userinfo.avatar}}" alt="..."  height="60px" width="60px">
{#                                    <img class="media-object" src="/static/img/default.png" height="60px" width="60px">#}
                                </a>
                            </div>
                            <div class="media-body">
                                {{ article.desc }}
                            </div>
                            <div style="margin-top: 10px">
                                <span class="article_footer"><a href="">{{ article.blog.userinfo.username }}</a></span>
                                <span class="article_footer">{{ article.create_time|date:'Y-m-d H:i:s' }}</span>
{#                                <span class="glyphicon glyphicon-thumbs-up article_footer">&nbsp;{{ artcle.up_num }}</span>#}
                                <span class=" article_footer"><i class="fa fa-thumbs-up fa-lg" aria-hidden="true"></i>&nbsp;{{ article.up_num }}</span>
{#                                <span class="glyphicon glyphicon-thumbs-down article_footer">&nbsp;{{ artcle.down_num }}</span>#}
                                <span class="article_footer"><i class="fa fa-thumbs-down fa-lg" aria-hidden="true"></i>&nbsp;{{ article.down_num }}</span>
{#                                <span class="glyphicon glyphicon-comment article_footer">&nbsp;{{ artcle.commit_num }}</span>#}
                                <span class=" article_footer"><i class="fa fa-commenting fa-lg" aria-hidden="true"></i>&nbsp;{{ article.commit_num }}</span>
                            </div>
                        </div>
                        <hr>
                    </div>
                {% endfor %}


            </div>
        </div>
        <div class="col-md-3">
            <div class="panel panel-success">
                <div class="panel-heading">
                    <h3 class="panel-title">广告位招租</h3>
                </div>
                <div class="panel-body">
                    寻宝探险：<a href="">点我找宝藏</a>
                </div>
            </div>
            <div class="panel panel-info">
                <div class="panel-heading">
                    <h3 class="panel-title">天天说书</h3>
                </div>
                <div class="panel-body">
                    <ul class="list-group">
                        <li class="list-group-item">红楼梦</li>
                        <li class="list-group-item">西游记</li>
                        <li class="list-group-item">三国演义</li>
                        <li class="list-group-item">水浒传</li>
                        <li class="list-group-item">追忆似水年华</li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</div>
</body>
</html>
    
```

### 2 个人头像显示

```python
 <div class="media-left media-middle">
                                <a href="#">
                                    <img class="media-object" src="media/{{article.blog.userinfo.avatar}}" alt="..."  height="60px" width="60px">
{#                                    <img class="media-object" src="/static/img/default.png" height="60px" width="60px">#}
                                </a>
                            </div>
```

### 3 个人站点路由设计

```python
 # 个人站点路由（这个路径必须放在最后面，如果放在前面，其他路径都会有问题）
    re_path('^(?P<username>\w+)$', views.personal_site ),
    
    
def personal_site(request, username):
    user = models.UserInfo.objects.filter(username=username).first()
    if user:
        article_list = user.blog.article_set.all()
        return render(request,'site.html', locals())
    else:
        print(404)
        return render(request, '404.html')
```

### 4 个人站点页面设计

### 5 左侧过滤功能

```python
# 查询当前站点下某年某月的文章数（分组依据，日期：年月），不需要连表
from django.db.models.functions import TruncMonth,
TruncDay, TruncYear, TruncHour
'''
Sales.objects
.annotate(month=TruncMonth('timestamp'))  # Truncate to month and add to select list
.values('month')  # Group By month
.annotate(c=Count('id'))  # Select the count of the grouping
.values('month','c')  # (might be redundant, haven'ttested) select month and count #
'''
```

### 6 左侧过滤和个人站点路由整合

Vue 数据双向绑定  只有数据一动 页面自动变换

## 补充

```python
1 图片防盗链
 -通过referer限制
 -在Nginx上限制（代码不用限制） nginx可以配置图片防盗链
 -Nginx不配置，代码中多加一层，中间件
```

## 作业

1 讲到哪写到哪

2 按标签，分类，时间过滤

## 昨日回顾

```python
1 首页样式（不用太关注）
2 开启media的访问
 -settings中配置MIDIA_ROOT = os.path.join(BASE_DIR, 'media')
 -FileField(upload_to='avatar/')，以后上传的头像都放在media下的avatar文件夹下
 -re_path('^media/(?P<path>.*?)$', serve, kwargs={'document_root': settings.MEDIA_ROOT}),
 -域+'media/avatar/a.jpg'
视图函数 压缩 头像图片 宽高
3 个人站点路由设计
4 
```

## 今日内容

### 0 伪静态

```python
1 为了搜索引擎优化，动态页面做成伪静态  SEO优化  SEM
2 xx.html 结尾的网页，权重高， p/14624772.php  p/14624772.jsp  权重低
3 百度是个大爬虫，一刻不停的在互联网上爬取页面 ---> 爬完存到自己的数据库中 --> 
你们百度搜索 --> 去百度的库中搜  --> 返回你看到的
4 谷歌需要翻墙
```

### 1 左侧查询和个人主页路由整合

```python
def personal_site(request, username, **kwargs):
    user = models.UserInfo.objects.filter(username=username).first()
    if user:

        article_list = user.blog.article_set.all()
        if kwargs:
            condition = kwargs.get('condition')
            params = kwargs.get('params')
            if condition == 'category':
                article_list = article_list.filter(category_id = params)
            elif condition == 'tag':
                article_list = article_list.filter(tag__id = params)  # 跨表
            elif condition == 'archive':
                param_year,param_month = params.split('/')
                article_list = article_list.filter(create_time__year=param_year, create_time__month=param_month)

        category_list = models.Category.objects.filter(blog=user.blog).annotate(count=Count('article')).values('count',
                                                                                                               'name', 'id')  # 对象
        tag_list = models.Tag.objects.filter(blog=user.blog).annotate(count=Count('article')).values_list('count',
                                                                                                          'name', 'id')
        month_list = models.Article.objects.filter(blog=user.blog).annotate(month=TruncMonth('create_time')).values(
            'month').annotate(count=Count('id')).values_list('month', 'count')
        return render(request, 'site.html', locals())
    else:

        return render(request, '404.html')
    
    
 # 正常想法
    # re_path('^(?P<username>\w+)/category/(?P<params>\d+).html$', views.personal_site),
    # re_path('^(?P<username>\w+)/tag/(?P<params>\d+).html$', views.personal_site),
    # re_path('^(?P<username>\w+)/archive/(?P<params>.*).html$', views.personal_site),
    # 三个路径混成一个路径
    # http://127.0.0.1:8000/zhangsan/category/1.html
    re_path('^(?P<username>\w+)/(?P<condition>category|tag|archive)/(?P<params>.*).html$', views.personal_site),

```

### 2 左侧标签，分类，归档写成inclusion_tag

```python
# -*- coding = utf-8 -*-
# @Time: 2021/10/16 23:47
# @Author: Thinkpad
# @File: left.py
# @Software: PyCharm
from django import template

register = template.Library()

from blog import models
from django.db.models import Count
from django.db.models.functions import TruncMonth  # 按月截断


@register.inclusion_tag('left.html')
def left_inclusion_tag(username):
    user = models.UserInfo.objects.filter(username=username).first()
    category_list = models.Category.objects.filter(blog=user.blog).annotate(count=Count('article')).values('count',
                                                                                                           'name',
                                                                                                           'id')  # 对象
    tag_list = models.Tag.objects.filter(blog=user.blog).annotate(count=Count('article')).values_list('count',
                                                                                                      'name', 'id')
    month_list = models.Article.objects.filter(blog=user.blog).annotate(month=TruncMonth('create_time')).values(
        'month').annotate(count=Count('id')).values_list('month', 'count')

    return {'username': username, 'category_list': category_list, 'tag_list': tag_list, 'month_list': month_list}

```

### 3 文章详情页面搭建

### 4 文章点赞点踩样式

```python

```

### 5 文章点赞点踩后端

```python

```

## 作业

## 昨日回顾

```python
1 左侧过滤，统一了路由

```

## 今日内容

### 1  评论的render显示

### 2  根评论的ajax提交和显示

```python
{% extends 'base.html' %}

{% block js %}
    <link rel="stylesheet" href="/static/css/article_css.css">
{% endblock %}

{% block title %}
    {{ article.title }}--文章
{% endblock %}

{% block site_name %}
    {{ article.blog.site_name }}
{% endblock %}

{% block main %}

    <div>
        <div class="text-center">
            <h3>{{ article.title }}</h3>
        </div>
        <hr>
        <div>
            {{ article.content|safe }}
        </div>
        <div class="clearfix">
            <div id="div_digg">
                <div class="diggit action">
                    <span class="diggnum" id="digg_count">{{ article.up_num }}</span>
                </div>
                <div class="buryit action">
                    <span class="burynum" id="bury_count">{{ article.down_num }}</span>
                </div>
                <div class="clear"></div>
                <div class="diggword" id="digg_tips">

                </div>

            </div>
        </div>


        <div>
            <h3>评论列表</h3>
            <div>
                <ul class="list-group">

                    {% for comment in comment_list %}
                        <li class="list-group-item">
                            <div style="margin-bottom: 5px">
                                <span>#{{ forloop.counter }}楼</span>
                                <span>{{ comment.create_time|date:'Y-m-d H:i:s' }}</span>
                                <span><a href="/{{ comment.user.username }}">{{ comment.user.username }}</a></span>
                                <span class="pull-right"><a href="">回复</a></span>
                            </div>
                            <div>
                                {% if comment.commit_id %}
                                    <div class="well">
                                        <p>@{{ comment.commit_id.user.username }}</p>
                                        <p>{{ comment.commit_id.content }}</p>
                                    </div>
                                {% endif %}
                                {{ comment.content }}
                            </div>

                        </li>
                    {% endfor %}


                </ul>
            </div>

        </div>

    </div>

{% endblock %}


{% block footer_js %}
    <script>
        $('.action').click(function () {
            var is_up = $(this).hasClass('diggit')
            var span = $(this).children('span')
            $.ajax({
                url: '/upanddown/',
                method: 'post',
                // 谁（当前登录用户）对那篇文章点赞或点踩
                data: {article_id: '{{ article.id }}', is_up: is_up, csrfmiddlewaretoken: '{{ csrf_token }}'},
                success: function (data) {
                    console.log(data)

                    $('#digg_tips').html(data.msg)
                    if (data.status == '100') {
                        //let num = Number(span.html()) + 1
                        span.html(Number(span.html()) + 1)
                    }
                }
            })
        })

        //$('.buryit').click(function () {
        //alert('点踩了')
        //})
    </script>
{% endblock %}
```

### 3  评论后端

### 4 子评论ajax提交和显示

### 5 后台管理首页文章显示

### 6 新增文章（富文本编辑器，xss攻击）

### 7 富文本 编辑器上传图片

## 作业

```python
1 修改密码功能
2 修改头像功能
3 修改文章功能
4 文章评论后，给文章作者发邮件（文章作者通过设置是否收邮件 需要加字段进行判断）
```

## 回顾

```python
1 web应用
 -bs和CS架构：http请求交互
 -mysql,redis:典型的CS架构的软件
 -docker，es:http协议
 -bs架构的好处：客户端不用更新
 -bs本质也是cs,服务端统一用socket 抽象出来的一层 传输层 应用层
 -一个线程处理一个连接 本质上Django框架，当来了一个请求后，一条线程 路由 视图函数 只能同时处理一个请求  全局变量可以共享 一条线程 多个协程 异步框架
    
2 HTTP协议
 -请求协议：请求首行，请求头，请求体 用ajax就是http请求
  -逻辑
 -响应协议：响应首行，响应头，响应体
    所有web框架就是处理请求返回响应
 -特点：
  -无状态无连接：会话保持 cookie  session  token
   -基于请求响应：（无法服务端主动向客户端推送消息） 长轮询 websocket协议（浏览器有限制）
  -基于tcp/ip之上的应用层：三次握手 四次挥手
 -http版本区别
3 web框架
 -写了一些底层代码，只让开发者关注业务逻辑，在固定的位置写固定代码，完成对一次http请求的处理

4 Django简介
```

## 今日内容

### 1 前后端开发模式

```python
1 前后端混合
2 前后端分离
```

### 2 api接口和restful规范

```python
1 API 接口
 -通过网络，规定了前后台信息交互规则的url链接，也就是前后台信息交互的媒介
2 接口文档
 -可以手动写（公司有平台，录到平台里，）
 -自动生成（coreapi Swagger）
    
3 restful规范（10条 规范 可不用）
 -1 数据的安全保障  url链接一般都采用https协议进行传输
 -2 接口特征表现  用api关键字标识接口url
 -3 多数据版本共存 请求地址中带版本
 -4 数据即是资源，均使用名词（可复数） 任何东西都是资源，均使用名词表示（尽量不要用动词）
 -5 资源操作由请求方式决定（method） 通过请求方式区分不同操作
 -6 过滤，通过在url上传参的形式传递搜索条件 在请求路径中带过滤
 -7 响应状态码  返回数据中带状态码
     -http请求的状态码（2,3,4,5）
  -返回的json格式中带状态码（标志当次请求成功或失败）
 -8 错误处理，应返回错误信息，error当做key  返回数据中带错误信息
 -9 返回结果，针对不同操作，服务器向用户返回的结果应该符合以下规范
    GET /collection：返回资源对象的列表（数组）  [{}]
        GET /collection/resource：返回单个资源对象  {}
        POST /collection：返回新生成的资源对象  {}
        PUT /collection/resource：返回完整的资源对象  {}
        PATCH /collection/resource：返回完整的资源对象 {}
        DELETE /collection/resource：返回一个空文档  
      {status:100,msg:查询成功，data：null}   
     
 -10 需要url请求的资源需要访问资源的请求链接  返回结果中带连接
    
```

### 3 postman的使用

```python
1 后端写好接口要测试，后端开发要使用一个工具测试接口（postman）
2 下载安装 
3 会发送http请求，get，post请求即可
4 请求地址带参数，请求体带数据，请求头加数据
5 响应cookie，响应头，响应体
```

### 4 drf介绍和安装

```python
1 可以更方便的使用Django写出符合restful规范的接口（不用也可以写符合规范的接口）
2 是一个APP
3 pip3 install djangorestframework
4 https://www.django-rest-framework.org/
5 简单使用
from rest_framework.views import APIView
from rest_framework.response import Response
注册APP
```

### 5 CBV和ApiView执行流程分析

## 作业

1 使用drf写接口

路由 get冲突

## 拓展

1 http各个版本之间的区别

易语言  木兰语言

量子力学

B站 php

## 上节回顾

```python
1 web开发模式：前后端分离（接口，drf），前后端混合（dtl）
 jsp与JavaScript是两个完全不同的东西
    
2 api 接口
 -接口，后端接口：给一个地址，向地址发送请求，可以返回json格式数据
3 restful规范
 -10条：只是一个规范，不强制，所以公司有自己的规则
 -使用HTTPS：http+ssl
 -路径中带api标识
 -路径中带版本号
 -尽量用名词加复数
 -通过请求方式来决定操作（get获取数据 post put delete）
 -返回中带状态码（大的json数据带状态码：自己定义的，http响应中带状态码）
 -返回结果中带错误信息
 -请求地址中带查询条件
 -响应结果中带连接
 -返回结果，针对不同操作，服务器向用户返回的结果应该符合规范
4 postman 的使用（接口测试）
 -模拟发送http请求，携带数据
 -快速把接口导入导出
5 drf：Djangorestframework，只针对Django的APP，让我们快速的写接口，render，redirect不用了，只返回前端json格式数据，前端可以是web，可以是APP，可以是小程序
6 bbs 作业
 -修改头像 不能用update 要使用 对象.save
 -是否开启发送邮件  只需要在userinfo表加字段
```

## 今日内容

### 0 drf快速使用

### 1 CBV源码分析和APIView源码分析

```python
from django.views.generic import View
from django.views import View
from django.views.generic.base import View
# 这三个是一个
只有对外部作用域有引用才叫闭包函数
```

1.1 CBV执行流程

```python
path("students/", views.StudentView.as_view()),
path("students/", views类的as_view()内部有个view闭包函数的内存地址),
"""
1 path的第二个参数是：View类的as_view内部有个view闭包函数的内存地址
2 一旦有请求来了，匹配路径成功
3 就会执行第二个参数view函数内存地址（request） 如果有分组和转换器会把相应的参数传过来
 执行闭包函数view会返回   return self.dispatch(request, *args, **kwargs)
4 本质执行了self.dispatch(request) --- View类的
5 通过反射去获得方法（如果是get请求，就是get方法，即 handler就是get）
 if request.method.lower() in self.http_method_names:
  handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
6 执行get方法，传入参数
handler(request, *args, **kwargs)
"""

@classonlymethod
def as_view(cls, **initkwargs):
    """Main entry point for a request-response process."""
    for key in initkwargs:
        if key in cls.http_method_names:
            raise TypeError(
                'The method name %s is not accepted as a keyword argument '
                'to %s().' % (key, cls.__name__)
            )
            if not hasattr(cls, key):
                raise TypeError("%s() received an invalid keyword %r. as_view "
                                "only accepts arguments that are already "
                                "attributes of the class." % (cls.__name__, key))

                def view(request, *args, **kwargs):
                    self = cls(**initkwargs)
                    self.setup(request, *args, **kwargs)
                    if not hasattr(self, 'request'):
                        raise AttributeError(
                            "%s instance has no 'request' attribute. Did you override "
                            "setup() and forget to call super()?" % cls.__name__
                        )
                        return self.dispatch(request, *args, **kwargs)
                    view.view_class = cls
                    view.view_initkwargs = initkwargs

                    # take name and docstring from class
                    update_wrapper(view, cls, updated=())

                    # and possible attributes set by decorators
                    # like csrf_exempt from dispatch
                    update_wrapper(view, cls.dispatch, assigned=())
                    # 返回闭包函数内存地址
                    return view
                
 
 http_method_names = ['get', 'post', 'put', 'patch', 'delete', 'head', 'options', 'trace']
    
 def dispatch(self, request, *args, **kwargs):
        # Try to dispatch to the right method; if a method doesn't exist,
        # defer to the error handler. Also defer to the error handler if the
        # request method isn't on the approved list.
        if request.method.lower() in self.http_method_names:
            # 获取属性 self是类的对象 反射
            handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
        else:
            handler = self.http_method_not_allowed
        return handler(request, *args, **kwargs)
```

1.2 APIView

```python
path("students/", APIView类的as_view内部是用了View闭包函数)
"""
1 path的第二个参数是：APIView类的as_view内部是用了View闭包函数  
view = super().as_view(**initkwargs)
2 一旦有请求来了，匹配路径成功
3 就会执行第二个参数view函数内存地址（request） 还是执行View的as_view内的view闭包函数
 但是加了个csrf_exempt装饰器
4 所以，继承了APIView的所有接口，都没有csrf的校验了（******）
5 执行了self.dispatch(request)  --- APIView类的
 def dispatch(self, request, *args, **kwargs):
   # 以后所有的request对象，都是***新的request对象***，它是drf的Resquest类的对象
        request = self.initialize_request(request, *args, **kwargs)
        self.request = request
        try:
         # 整个drf的执行流程内的权限，频率，认证
            self.initial(request, *args, **kwargs)
            if request.method.lower() in self.http_method_names:
                handler = getattr(self, request.method.lower(),
                                  self.http_method_not_allowed)
            else:
                handler = self.http_method_not_allowed

            response = handler(request, *args, **kwargs)

        except Exception as exc:
         # 全局异常
            response = self.handle_exception(exc)
  # 响应
        self.response = self.finalize_response(request, response, *args, **kwargs)
        return self.response
        
第一：把原来的request做成新的request
第二：增加了权限、频率、认证
第三：全局异常
第四：响应

"""

class APIView(View): 继承了View,重写了as_view优先执行
@classmethod
def as_view(cls, **initkwargs):
    """
        Store the original class on the view function.

        This allows us to discover information about the view when we do URL
        reverse lookups.  Used for breadcrumb generation.
        """
    if isinstance(getattr(cls, 'queryset', None), models.query.QuerySet):
        def force_evaluation():
            raise RuntimeError(
                'Do not evaluate the `.queryset` attribute directly, '
                'as the result will be cached and reused between requests. '
                'Use `.all()` or call `.get_queryset()` instead.'
            )
            cls.queryset._fetch_all = force_evaluation
    # 调用父类的as_view  super会严格遵循mro列表去查询
            view = super().as_view(**initkwargs)
            view.cls = cls
            view.initkwargs = initkwargs

            # Note: session based authentication is explicitly CSRF validated,
            # all other authentication is CSRF exempt.
            # 相当于给view加了装饰器 禁用csrf
            return csrf_exempt(view)
        
        
        
 def dispatch(self, request, *args, **kwargs):
        """
        `.dispatch()` is pretty much the same as Django's regular dispatch,
        but with extra hooks for startup, finalize, and exception handling.
        """
        self.args = args
        self.kwargs = kwargs
        request = self.initialize_request(request, *args, **kwargs)
        self.request = request
        self.headers = self.default_response_headers  # deprecate?

        try:
            self.initial(request, *args, **kwargs)

            # Get the appropriate handler method
            if request.method.lower() in self.http_method_names:
                handler = getattr(self, request.method.lower(),
                                  self.http_method_not_allowed)
            else:
                handler = self.http_method_not_allowed

            response = handler(request, *args, **kwargs)

        except Exception as exc:
            response = self.handle_exception(exc)

        self.response = self.finalize_response(request, response, *args, **kwargs)
        return self.response


### 需要记住：
-0 所有的csrf都不校验了
-1 request对象变成了新的request对象，drf的request对象
-2 执行了权限，频率，认证
-3 捕获了全局异常（统一处理异常）
-4 处理了response对象，如果浏览器访问是一个样，postman访问又是一个样
-5 以后，在视图类中使用的request对象已经不是原来的request对象了，现在都是drf的request对象了

###  request = self.initialize_request(request, *args, **kwargs)
## 返回的request对象是drf  Request类的request对象
from rest_framework.request import Request

 def initialize_request(self, request, *args, **kwargs):
        """
        Returns the initial request object.
        """
        parser_context = self.get_parser_context(request)
  # def的Request
        return Request(
            request,
            parsers=self.get_parsers(),
            authenticators=self.get_authenticators(),
            negotiator=self.get_content_negotiator(),
            parser_context=parser_context
        )
    
## ** 以后，在视图类中使用的request对象已经不是原来的request对象了，现在都是drf的request对象了
```

### 2 Request对象分析

```python
from rest_framework.request import Request
1 django 原生的Request: django.core.handlers.wsgi.WSGIRequest
2 drf的Request: rest_framework.request.Request
    
3 drf的request对象内有原生的request
    request._request: 原生的request
        
4 在视图类中使用
 request.method  拿到的就是请求方式
 正常拿，应该是 request._request.method
5 如何实现这种操作？
 -对象.属性会触发 类的__getattr__方法
    
6 drf的Request类重写了__getattr__
 def __getattr__(self, attr):
        try:
            # 去原生的request反射属性
            return getattr(self._request, attr)
        except AttributeError:
            return self.__getattribute__(attr)
7 虽然视图类中request对象变成了drf的request，但是用起来，跟原来的一样，只不过它多了一些属性
 -request.data  # 多的 post请求提交的数据，不论什么格式，都在它中
    @property
    def data(self):
        if not _hasattr(self, '_full_data'):
            self._load_data_and_files()
        return self._full_data
 -request.query_params  # 多的 get请求提交的数据（查询参数）
 query_params，查询参数，也叫查询字符串(query string )
 request.query_params与Django标准的request.GET相同，只是更换了更正确的名称而已。

    
    @property
    def query_params(self):
        """
        More semantically correct name for request.GET.
        """
        return self._request.GET
    
 -request.FILES
    
8 重点记住：
 -drf的request对象用起来跟原来一样（重写了__getattr__）
 -request.data  # post请求提交的数据，不论什么格式，都在它中
 -request.query_params  # get请求提交的数据（查询参数）
    
class Request:
     def __init__(self, request, parsers=None, authenticators=None,
                 negotiator=None, parser_context=None):
        assert isinstance(request, HttpRequest), (
            'The `request` argument must be an instance of '
            '`django.http.HttpRequest`, not `{}.{}`.'
            .format(request.__class__.__module__, request.__class__.__name__)
        )

        self._request = request
        
request._request
request._request.method
request.method  # 等同于上面，如何实现？
# 对象.属性
# __getattr__只有在使用点调用属性且属性不存在的时候才会触发

 def __getattr__(self, attr):
        """
        If an attribute does not exist on this instance, then we also attempt
        to proxy it to the underlying HttpRequest object.
        """
        try:
            return getattr(self._request, attr)
        except AttributeError:
            return self.__getattribute__(attr)
```

### 3 序列化器的作用

```python
1 序列化：把python中的对象转成json格式字符串
2 反序列化：把json格式字符串转成python中的对象

3 注意：drf的序列化组件（序列化器）
 把对象转成字典
 因为有字典，直接丢到Response中就可以了
    
```

### 4 序列化器的使用

```python
from rest_framework.serializers import Serializer
from rest_framework import serializer

class BookSerializer(serializers.Serializer):
    # 在这里写要序列化的字段】
    # 序列化字段类
    
from rest_framework import serializers

class BookSerializer(serializers.Serializer):
    name = serializers.CharField()
    price = serializers.IntegerField()
    publish = serializers.CharField()
    

1 写一个序列化的类，继承Serializer
2 在类中写要序列化的字段
3 在视图类中使用（实例化）
4 得到序列化后的数据，返回即可


from rest_framework.views import APIView
from opt.serializer import BookSerializer
from rest_framework.response import Response
from django.db import models

class BookView(APIView):
    def get(self, request):
        book_list = models.Book.objects.all()
        # instance = None 要序列化的数据
        # data = empty  要反序列化的数据
        # many = True  如果序列化多条，一定要写many=True
        book_ser = BookSerializer(instance=book_list, many=True)

        # book_ser.data 是序列化后的数据
        return Response(book_ser.data)
    
    
# 字段参数之source,不指定，默认就是字段名，必须跟数据库对应
# 指定了source就可以给字段改名了

path('books_new/<int:id>/', views.BookViewId.as_view())
re_path('^books_new/(?P<id>\d+)$', views.BookViewId.as_view())
```

4.1 source

```python
1 指定要序列化的字段（数据表中字段）
publish = serializers.CharField(source='publish.city')

2 用的最多：只有一个字段（也可以跨表）
```

4.2 SerializerMethodField

```python
1 用的最多，跨表查（要么是列表，要么是字段）
publish = serializers.SerializerMethodField()
def get_publish(self,obj):
    print(obj)
    # return {'name':'sss','city':'sss'}
    return
{'name':obj.publish.name,'city':obj.publish.city}
```

4.3 在模型表中写方法

```python
# 表模型中写的
def publish_name(self):
    return 
{'name':obj.publish.name,'city':obj.publish.city}

@property
def author_list(self):
    return
[{'name':author.name,'age':author.age,'id':author.nid} for author in self.authors.all()]

# 序列化类中
# publish_name = serializers.CharField(source='publish_name')
publish_name = serializers.DictField()
author_list = serializers.ListField()
```

### 5 反序列化

## 作业

## 补充

```python
- 在公司的开发环境
 - Windows: 远程连接Linux
 - Ubutnu  (图形化界面的linux）
 - mac
               
               
1 装饰器函数 csrf_exempt
@csrf_exempt
def test():
 pass
               
本质是  test=csrf_exempt(test)
```

面向搜索引擎编程

```python
File-Settings-Editor-Color Scheme-Python,根据下面的预览界面，可以修改任一参数的颜色，上面可以选主题，我选的是Monokai。
File-Settings-Editor-Color Scheme-Python-Line Comment修改注释颜色。
File-Settings-Editor-Color Scheme-Console Color可以修改终端颜色，我的背景色是232B38。
File-Settings-Editor-Color Scheme-General-Text-Default text修改代码行背景色。
File-Settings-Editor-Color Scheme-General-Code-Line Number修改行号颜色。
File-Settings-Editor-Color Scheme-General-Editor-Gutter background修改行号背景颜色。
File-Settings-Keymap直接搜索run可以改变运行程序的快捷键。
Pycharm中matplotlib打印图片时，如果想让图片单独弹出一个窗口，设置如下：File-Settings-Tools-Python Scientific取消勾选就可以了
```

## 昨日回顾

```python
1 cbv执行流程
 -路由里写的是  类名.as_view()
 -as_view()执行完是一个闭包函数的内存地址
 -当请求来了，路由匹配上就会调用 闭包函数（request）
 -self.dispatch(request)
 -通过反射，去类中根据请求方式取方法，执行，把参数传入
2 APIView的执行流程
 -以后写drf，最顶层都是APIView
 -类名.as_view()是APIView, 内部调用了父类的view
 -在后加了去除csrf
 -self.dispatch(request)是APIView的
  -包装新的request
  -执行了权限，频率，认证
  -处理了全局异常
  -处理了响应
4 drf的Request类
 -重写了__getattr__: 新的request用起来，跟之前一模一样
 -drf的request._request 是原来的request
 -request.data: post, put, patch, 请求的body体中的数据，都从data中取
5 序列化，反序列化
 -对象 ---> json
 -json ---> 对象
6 序列化器
 -定义一个类，继承serializer
 -在类内字段（常用字段，和非常用字段）（字段参数）
 -在视图类中，实例化得到一个序列化类的对象，传入要序列化的数据
 -对象.data ---> 就是字典
 -source
```

## 今日内容

### 1 反序列化，局部钩子，全局钩子

```python
1 如果要序列化，继承了serializer，必须调用create（）方法

2 使用
# 视图类
def post(self, request):
    piblish_ser = serializer.PublishSerializer(data=request.data)
    if publish_ser.is_valid():
        # 直接保存，保存到哪个表里？需要重新save
        publish_ser.save()
        return Response(publish_ser.data)
    else:
        print(publish_ser.errors)
        return Response('数据有问题啊')
    
# 序列化类
def create(self, validated_data):
    res = models.Publish.objects.create(**validated_data)
    return res
```

1.3 局部和全局钩子

```python
def validate_name(self, data):
    # data就是当前字段的值
    if data.startswith('sb'):
        raise validationError('不能以sb开头')
    else:
        return data
    
def validate(self, attrs):
    if attrs.get('name') == attrs.get('city'):
        raise ValidationError('city和名字不能一样')
    else:
        return attrs
```

### 2 序列化类常用字段属性

### 2.1 常用和非常用字段

### 2.2 字段参数

```python
# 针对charfield
max_length 最大长度
min_lenght 最小长度
allow_blank 是否允许为空
# 针对interfield
max_value 最大值
min_value 最小值
# 通用的，大家都有
# 这两个最重要
read_only  表明该字段仅用于序列化输出，默认False(序列化)
write_only  表明该字段仅用于反序列化输入，默认False
required  表明该字段在反序列化时必须输入，默认False  
default  反序列化时使用的默认值
allow_null  表明该字段是否允许传入None,默认False
validators  该字段使用的验证器
error_messages  包含错误编号与错误信息的字典

extra_kwargs = {
    'name': {'max_length': 6, 'required': True, 'read_only': True},
    'price': {'min_value': 0, 'required': True, 'write_only': True}
}
```

### 3 模型序列化器

3.1 视图类

```python
class BookView(APIView):
    def get(self, request):
        qs = models.Book.all()
        
  ser = serializer.BookModelSerializer(instance=qs,many=True)
    return Response(ser.data)

 def post(self, request):
        ser = serializer.BookModelSerializer(data=request.data)
        if ser.is_valid():
            ser.save()
            return Response(ser.data)
        else:
            return Response(ser.errors)
        
class BookDetailview(APIView):
    def get(self, request, id):
        book = models.Book.objects.filter(pk=id).first()
        ser = serializer.BookModelSerializer(instance=book)
        return Response(ser.data)
    def put(self, request, id):
        book = models.Book.object.filter(pk=id).first()
        ser = serializer.BookMdelSerializer(instance=book,data=request.data)
        if ser.id_valid():
            ser.save()
            return Response(ser.data)
        else:
            return Response(ser.errors)
    def delete(self, request, id):
        res = models.Book.objects.filter(pk=id).delete()
        if res[0] > 0:
            return Response('')
        else:
            return Response('要删的不存在')
            
        
```

### 3.2  序列化类

```python
class BookModelSerializer(serializers.ModelSerializer):
    class Meta:
        model = models.Book
        fields = "__all__"
        extra_kwargs = {
            'publish': {'required': True, 'write_only': True},
            'author': {'required': True, 'write_only': True},
        }
        publish_detail = PublishSerializer(source='publish', read_only=True)
        author_list = serializers.ListField(read_only=True)
        # 字段自己的校验，全局钩子，局部钩子
```

### 3.3 表模型

```python
class Book(models.Model):
    nid = models.AutoField(primary_key=True)
    name = models.CharField(max_length=32)
    price = models.DecimalField(max_digits=5,decimal_places=2)
    
```

### 3.4 路由

```python
path('books/',view.Bookview.as_view()),
path('books/<int:id>/',views.BookDetailview.as_view())
```

### 4 请求对象

### 5 响应对象属性

### 6 通过配置，修改响应格式

## 作业

```python
1 自己封装一个响应对象
2 
```

## 补充

```python
1 django项目，APP中有templates,找模板

后期不管新增还是更新，都调用ser.save()  新增 create  更新 update
如果继承了Serializer,必须重写这两方法
如果继承了ModelSerializer,不用重写了
```

代码审查工具  cicd

## 昨日回顾

```python
0 drf是Django的一个APP
 -序列化器
 -Request
 -Response
 -版本控制
 -认证，频率，权限
 -过滤，排序，分页
 -视图相关
 -自动生成接口文档（coreapi,swagger）
 -jwt做认证（第三方）
 -xadmin ---> 对admin的美化（bootstrap+jq 1.x版本）,simple-ui
 -RBAC: 基于角色的访问控制（公司内部项目）
    
1 序列化器
 1. 序列化，序列化器会把模型对象转换成字典，经过response以后变成json字符串
 2. 反序列化，把客户端发送过来的数据，经过request以后变成字典，序列化器可以把字典转化成模型
 3. 反序列化，完成数据校验功能
    
2 Serializer
 -序列化
  - 实例化序列化对象，many参数作用
        -source
        -SerializerMethodField  只能用来序列化
        -模型表中写方法
 -反序列化
  - ser=BookSerializer(data=request.data)
  - 数据校验：ser.is_valid()
  - 存数据：手动存，重写BookSerializer的create方法
  - ser.save() 如果是新增，会调用create，如果是修改，会调用update
  - ser.data 如果有instance对象，就是对instance做序列化
  - 全局，局部钩子，字段参数
3 ModelSerializer 
 - class Meta:
        model=Book
    fields='__all__'
        extra_kwargs = {}
 - 重写某个字段：跟之前学的又一样了
 - 子序列化
4 常用非常用字段
5 字段参数
6 如果第三张表是手动建的，authors是存不进去的
7 请求对象的属性
8 响应对象
 -data
 -status：响应状态码
 -header: 响应头
 -content_type: 响应类型
10 加入serializer后整个后端逻辑
中间件路由 视图 模型 序列化器

分层 解耦 可扩展性高
```

## 今日内容

### 1 序列化类源码分析

### 1.1 many参数

```python
1 序列化类实例化的时候，传了many，序列化多条，不传，就序列化一条（单条）
# many=True 实例化得到的对象是ListSerializer
ser=serializer.BookModelSerializer(instance=qs,many=True)
print(type(ser))
# rest_framework.serializers.ListSerializer
# 列表中套了很多BookModelSerializer

# many=False 实例化得到的对象是BookModelSerializer
ser=serializer.BookModelSerializer(instance=qs)
print(type(ser))
# app01.serializer.BookModelSerializer

类实例化：在执行__init__之前，先执行了__new__生成一个空对象(决定了是哪个类的对象)
实际是元类里面的__call__触发了__new__
在__new__中进行判断，如果many=True,就返回ListSerializer的对象

 def __new__(cls, *args, **kwargs):
        # We override this method in order to automatically create
        # `ListSerializer` classes instead when `many=True` is set.
        if kwargs.pop('many', False):
            return cls.many_init(*args, **kwargs)
        return super().__new__(cls, *args, **kwargs)
    
@classmethod
def many_init(cls, *args, **kwargs):
     allow_empty = kwargs.pop('allow_empty', None)
        child_serializer = cls(*args, **kwargs)
        list_kwargs = {
            'child': child_serializer,
        }
        if allow_empty is not None:
            list_kwargs['allow_empty'] = allow_empty
        list_kwargs.update({
            key: value for key, value in kwargs.items()
            if key in LIST_SERIALIZER_KWARGS
        })
        meta = getattr(cls, 'Meta', None)
        list_serializer_class = getattr(meta, 'list_serializer_class', ListSerializer)
        return list_serializer_class(*args, **list_kwargs)
```

### 1.2 局部全局钩子的源码分析

```python
is_valid() ---> 判断_validated_data 如果没有，执行
self.run_validation(self.initial_data)
目前在BaseSerializer,如果按住Ctrl点击，会直接进到它父类的run_validation。进到Field,不是真正执行的方法

我们需要从头找，实际上是Serializer类的run_validation

def run_validation(self, data=empty):
    value = self.to_internal_value(data)  # 字段自己校验和局部钩子
    try:
        self.run_validators(value)  
        value = self.validate(value)  # 全局钩子
        assert value is not None, '.validate() should return the validated data'
        except (ValidationError, DjangoValidationError) as exc:
            raise ValidationError(detail=as_serializer_error(exc))

            return value
        
        
# 局部钩子是在 to_internal_value 执行的     
def to_internal_value(self, data):
        for field in fields:
            validate_method = getattr(self, 'validate_' + field.field_name, None)
            primitive_value = field.get_value(data)
            try:
                validated_value = field.run_validation(primitive_value)
                if validate_method is not None:
                    validated_value = validate_method(validated_value)
            except ValidationError as exc:
                errors[field.field_name] = exc.detail
            except DjangoValidationError as exc:
                errors[field.field_name] = get_error_detail(exc)
            except SkipField:
                pass
            else:
                set_value(ret, field.source_attrs, validated_value)

        if errors:
            raise ValidationError(errors)

        return ret
    
## 总结
is_valid --> BaseSerializer的is_valid ---> 执行
self.run_validation(self.initial_data)  ---> Serializer 的
run_validation ---> self.to_internal_value(data): 局部钩子
value = self.validate(value): 全局钩子
    
    
self.to_internal_value(data): 局部钩子 --->  getattr(self, 'validate_' + field.field_name, None)
```

### 1.3 序列化对象.data

```python
序列化对象.data方法--调用父类data方法---调用对象自己的to_representation(自定义的序列化类无此方法，去父类找)
Serializer类里有to_representation方法，for循环执行
attribute = field.get_attribute(instance)
再去Field类里面去找get_attribute方法，self.source_attrs就是被切分的source，
然后执行get_attribute方法，
source_attrs当参数传过去，判断是方法就加括号执行，是属性就把值取出来


 @property
 def data(self):
    ret = super().data
    return ReturnDict(ret, serializer=self)


Beseserializer
 @property
 def data(self):
    if hasattr(self, 'initial_data') and not hasattr(self, '_validated_data'):
        msg = (
            'When a serializer is passed a `data` keyword argument you '
            'must call `.is_valid()` before attempting to access the '
            'serialized `.data` representation.\n'
            'You should either call `.is_valid()` first, '
            'or access `.initial_data` instead.'
        )
        raise AssertionError(msg)

        if not hasattr(self, '_data'):
            if self.instance is not None and not getattr(self, '_errors', None):
                self._data = self.to_representation(self.instance)
            elif hasattr(self, '_validated_data') and not getattr(self, '_errors', None):
                self._data = self.to_representation(self.validated_data)
            else:
                self._data = self.get_initial()
        return self._data
    
    
    
  def to_representation(self, instance):
        """
        Object instance -> Dict of primitive datatypes.
        """
        ret = OrderedDict()
        fields = self._readable_fields

        for field in fields:
            try:
                attribute = field.get_attribute(instance)
            except SkipField:
                continue

            # We skip `to_representation` for `None` values so that fields do
            # not have to explicitly deal with that case.
            #
            # For related fields with `use_pk_only_optimization` we need to
            # resolve the pk value.
            check_for_none = attribute.pk if isinstance(attribute, PKOnlyObject) else attribute
            if check_for_none is None:
                ret[field.field_name] = None
            else:
                ret[field.field_name] = field.to_representation(attribute)

        return ret
```

### 2 所有http状态码

```python
HTTP_100_CONTINUE = 100
HTTP_101_SWITCHING_PROTOCOLS = 101
HTTP_200_OK = 200
HTTP_201_CREATED = 201
HTTP_202_ACCEPTED = 202
HTTP_203_NON_AUTHORITATIVE_INFORMATION = 203
HTTP_204_NO_CONTENT = 204
HTTP_205_RESET_CONTENT = 205
HTTP_206_PARTIAL_CONTENT = 206
HTTP_207_MULTI_STATUS = 207
HTTP_208_ALREADY_REPORTED = 208
HTTP_226_IM_USED = 226
HTTP_300_MULTIPLE_CHOICES = 300
HTTP_301_MOVED_PERMANENTLY = 301
HTTP_302_FOUND = 302
HTTP_303_SEE_OTHER = 303
HTTP_304_NOT_MODIFIED = 304
HTTP_305_USE_PROXY = 305
HTTP_306_RESERVED = 306
HTTP_307_TEMPORARY_REDIRECT = 307
HTTP_308_PERMANENT_REDIRECT = 308
HTTP_400_BAD_REQUEST = 400
HTTP_401_UNAUTHORIZED = 401
HTTP_402_PAYMENT_REQUIRED = 402
HTTP_403_FORBIDDEN = 403
HTTP_404_NOT_FOUND = 404
HTTP_405_METHOD_NOT_ALLOWED = 405
HTTP_406_NOT_ACCEPTABLE = 406
HTTP_407_PROXY_AUTHENTICATION_REQUIRED = 407
HTTP_408_REQUEST_TIMEOUT = 408
HTTP_409_CONFLICT = 409
HTTP_410_GONE = 410
HTTP_411_LENGTH_REQUIRED = 411
HTTP_412_PRECONDITION_FAILED = 412
HTTP_413_REQUEST_ENTITY_TOO_LARGE = 413
HTTP_414_REQUEST_URI_TOO_LONG = 414
HTTP_415_UNSUPPORTED_MEDIA_TYPE = 415
HTTP_416_REQUESTED_RANGE_NOT_SATISFIABLE = 416
HTTP_417_EXPECTATION_FAILED = 417
HTTP_418_IM_A_TEAPOT = 418
HTTP_422_UNPROCESSABLE_ENTITY = 422
HTTP_423_LOCKED = 423
HTTP_424_FAILED_DEPENDENCY = 424
HTTP_426_UPGRADE_REQUIRED = 426
HTTP_428_PRECONDITION_REQUIRED = 428
HTTP_429_TOO_MANY_REQUESTS = 429
HTTP_431_REQUEST_HEADER_FIELDS_TOO_LARGE = 431
HTTP_451_UNAVAILABLE_FOR_LEGAL_REASONS = 451
HTTP_500_INTERNAL_SERVER_ERROR = 500
HTTP_501_NOT_IMPLEMENTED = 501
HTTP_502_BAD_GATEWAY = 502
HTTP_503_SERVICE_UNAVAILABLE = 503
HTTP_504_GATEWAY_TIMEOUT = 504
HTTP_505_HTTP_VERSION_NOT_SUPPORTED = 505
HTTP_506_VARIANT_ALSO_NEGOTIATES = 506
HTTP_507_INSUFFICIENT_STORAGE = 507
HTTP_508_LOOP_DETECTED = 508
HTTP_509_BANDWIDTH_LIMIT_EXCEEDED = 509
HTTP_510_NOT_EXTENDED = 510
HTTP_511_NETWORK_AUTHENTICATION_REQUIRED = 511
```

### 3 两个视图基类

```python
1 APIView
2 GenericAPIView(APIView)
 -属性：
    queryset
    serializer_class
 -方法：
    get_queryset：获取qs数据
    get_object: 获取一条数据的对象
    get_serializer：以后使用它来实例化得到ser对象
    get_serializer_class: 获取序列化来，注意跟上面区分
        
from rest_framework.generics import GenericAPIView
class PublishView(GenericAPIView):
    queryset = models.Publish.objects.all()  # 序列化的数据
    serializer_class = serializer.PublishSerializer  # 要序列化的类
    def get(self, request):
        qs = self.get_queryset()
        ser = self.get_serializer(qs, many=True)
        return Response(ser.data)
    def post(self, request):
        ser = self.get_serializer(data=request.data)
        if ser.is_valid():
            ser.save()
            return Response(ser.data)
        else:
            return Response(ser.errors)
```

### 4 5个视图扩展类

```python
1 查所有，查一个，删一个，改一个，增一个
# GenericAPIView+5个视图扩展类
from rest_framework.mixins import CreateModelMixin, ListModelMixin, RetrieveModelMixin, DestroyModelMixin, UpdateModelMixin


```

### 5 9个视图子类

```python
1 查所有 ListModelMixin + GenericAPIView = ListAPIView
2 查一个 RetrieveModelMixin + GenericAPIView = RetrieveAPIView
3 删一个 DestroyModelMixin + GenericAPIView = DestroyAPIView
4 改一个 UpdateModelMixin + GenericAPIView = UpdateAPIView
5 增一个 CreateModelMixin + GenericAPIView = CreateAPIView

6 查所有+增一个 ListCreateAPIView
7 查一个+删一个 RetrieveDestroyAPIView
8 查一个+改一个  RetrieveUpdateAPIView
9 查一个+删一个+改一个 RetrieveUpdateDestroyAPIView

```

### 6 视图集

```python
from rest_framework.viewsets import ModelViewSet, ReadOnlyModelViewSet
from rest_framework.viewsets import ViewSet, GenericViewSet, ViewSetMixin

ViewSet(ViewSetMixin, views.APIView)
GenericViewSet(ViewSetMixin, generics.GenericAPIView)

 for method, action in actions.items():
   handler = getattr(self, action)
   setattr(self, method, handler)
    
    
#两个基类
APIView
GenericAPIView：有关数据库操作，queryset 和serializer_class


#5个视图扩展类(rest_framework.mixins)
CreateModelMixin：create方法创建一条
DestroyModelMixin：destory方法删除一条
ListModelMixin：list方法获取所有
RetrieveModelMixin：retrieve获取一条
UpdateModelMixin：update修改一条

#9个子类视图(rest_framework.generics)
CreateAPIView:继承CreateModelMixin,GenericAPIView，有post方法，新增数据
DestroyAPIView：继承DestroyModelMixin,GenericAPIView，有delete方法，删除数据
ListAPIView：继承ListModelMixin,GenericAPIView,有get方法获取所有
UpdateAPIView：继承UpdateModelMixin,GenericAPIView，有put和patch方法，修改数据
RetrieveAPIView：继承RetrieveModelMixin,GenericAPIView，有get方法，获取一条


ListCreateAPIView：继承ListModelMixin,CreateModelMixin,GenericAPIView，有get获取所有，post方法新增
RetrieveDestroyAPIView：继承RetrieveModelMixin,DestroyModelMixin,GenericAPIView，有get方法获取一条，delete方法删除
RetrieveUpdateAPIView：继承RetrieveModelMixin,UpdateModelMixin,GenericAPIView，有get获取一条，put，patch修改
RetrieveUpdateDestroyAPIView：继承RetrieveModelMixin,UpdateModelMixin,DestroyModelMixin,GenericAPIView，有get获取一条，put，patch修改，delete删除

#视图集
ViewSetMixin：重写了as_view 
ViewSet：     继承ViewSetMixin和APIView

GenericViewSet：继承ViewSetMixin, generics.GenericAPIView
ModelViewSet：继承mixins.CreateModelMixin,mixins.RetrieveModelMixin,mixins.UpdateModelMixin,mixins.DestroyModelMixin,mixins.ListModelMixin,GenericViewSet
ReadOnlyModelViewSet：继承mixins.RetrieveModelMixin,mixins.ListModelMixin,GenericViewSet
```

## 作业

```python
1 book的5个接口，从第一层写到第五层
2 视图中的类继承关系
```

## 昨日回顾

```python
1 序列化器源码
 -many参数控制.在__new__中控制了对象的生成
 -局部和全局钩子源码：is_valid --> 找self.方法 一定要从根上找
 -source参数是如何执行的：'publish.name','方法'
2 视图：
 -2个基类
 -5个视图扩展类
 -9个视图子类
 -视图集
  -ViewSetMixin
  -ViewSet,GenericViewSet
  -ModelViewSet,ReadOnlyModelViewSet
        
3 视图类的继承原则
 -如果不涉及数据库操作：继承APIView
 -如果想让路由可以映射：继承ViewSetMixin
 -如果不涉及数据库操作，又要让路由可以映射：继承ViewSet
    
 -如果涉及到数据库操作：继承GenericAPIView
 -如果想让路由可以映射：继承ViewSetMixin
 -如果涉及数据库操作，又要让路由可以映射：继承GenericViewSet
 -如果涉及数据库操作，又要让路由可以映射,还要能新增：继承GenericViewSet+CreateModelMixin或者继承ViewSetMixin+CreateAPIView
 -如果只涉及到数据库操作和新增：继承CreateAPIView
    
4 ViewSetMixin：路由写法特殊
    
5 类实例化，先执行了元类的__call__：调用了这个类的__new__，生成一个空对象，调用了类的__init__，完成了对象的初始化
6 对象() --> 会触发类的 __call__
7 类() --> 会触发类的类(元类)的 __call__

```

## 今日内容

### 1 drf响应格式和请求格式配置（了解）

#### 1.1 配置响应格式

```python
1 在配置文件中配置
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': (  # 默认响应渲染类
        'rest_framework.renderers.JSONRenderer',  # json渲染器
        'rest_framework.renderers.BrowsableAPIRenderer',  # 浏览API渲染器
    )
}

2 在浏览器访问就是浏览器方式，用postman访问就是json格式，ajax请求就是json格式
3 原来没有配置，为什么显示浏览器方式和json的样子
4 drf也有一套默认配置文件，默认就配了两个响应类
5 局部配置某个视图类的响应格式，在视图类中配置
renderer_classes = []

```

#### 1.2 配置能够解析的格式(urlencoded, formdata, json)

```python
1 在settings中配置
REST_FRAMEWORK = {
'DEFAULT_PARSER_CLASSES': [
    'rest_framework.parsers.JSONParser',  # json
    'rest_framework.parsers.FormParser',  # form-urlencoded
    'rest_framework.parsers.MultiPartParser'  # form-data
],
}
2 它就只能解析三种请求编码格式（urlencoded,formdata,json）

3 局部使用
    
from rest_framework.parsers import FormParser
parser_classes = [FormParser]
```

### 2 封装自己的Response对象

### 2.1 low版本

**封装**

```python
class MyResponse():
    def __init__(self):
        self.status = 100
        self.msg = None
    @property
    def get_dict(self):
        return self.__dict__
    
```

**使用**

```python
# class Test(APIView):
#     def get(self, request):
#
#         response = {'status': '100', 'msg': None}
#         # 写一堆逻辑
#         response['data'] = {'name': 'lqz', 'age': 19}
#         response.data = {}
#         return Response(response)

class Test(APIView):
    def get(self, request):
        response = MyResponse()
        response.status = 101
        response.msg = '失败了'
        response.data = {'name': 'lqz', 'age': 19}
        return Response(response.get_dict)
```

#### 2.2 高级版本

```python
## 封装
class APIResponse(Response):
    def __init__(self, code=100, msg=None, data=None, status=None,
                 template_name=None, headers=None,
                 exception=False, content_type=None, **kwargs):
        dic = {'status': code, 'msg': msg}
        if data:
            dic['data'] = data
        if kwargs:
            dic.update(kwargs)
        super().__init__(data=dic, status=status,
                 template_name=template_name, headers=headers,
                 exception=exception, content_type=content_type)
        
## 使用 在视图类中
class Test(APIView):
    def get(self, request):
        # data = ser.data
        return APIResponse(msg='成功了',data = {'name': 'lqz', 'age': 19}, next=9)
```

### 3 drf自动生成路由

```python
1 三种路由写法
 - path("students/", views.StudentAPIView.as_view()),
 - path("students5/", views.StudentViewSet.as_view({
        "get":"get_student_list",
        "post":"post",
    })),
 - 自动生成路由
    
# 四步操作   
# 1 导入路由类
from rest_framework.routers import SimpleRouter, DefaultRouter
# 2 实例化得到对象
router = SimpleRouter()
# 3 注册路由
router.register(prefix='books', viewset=views.BookView, basename='books')
# print(router.urls)  # 自动生成的路由
# 4 把自动生成的路由加入到urlpatterns
# (1)
urlpatterns = []
urlpatterns += router.urls

# (2)
urlpatterns = [
    path('api/', include(router.urls))
]

# DefaultRouter 生成的路由多了一个根的路由
router = DefaultRouter()


from rest_framework.viewsets import ViewSetMixin
# 要继承 ViewSetMixin
# 继承视图类
# 继承5个视图扩展类
# 要映射：继承9个视图扩展类

## 总结：ViewSetMixin+9个视图扩展类才能生成自动路由

```

### 4 action装饰器

```python
1 作用：给自动生成路由的视图类再定制一些路由
2 用法
from rest_framework.decorators import action
from rest_framework.generics import ListAPIView, RetrieveAPIView
from rest_framework.viewsets import ViewSetMixin
class BookView(ViewSetMixin, ListAPIView, RetrieveAPIView):
    queryset = models.Book.objects.all()
    serializer_class = serializer.BookSerializer
    # methods: 是允许的请求方式
    # detail: 是不带id的路由
    # @action(methods=['GET', 'POST'], detail=False)
    # def send_email(self, request):
    #     return APIResponse(msg='发送成功')
    # http://127.0.0.1:8000/opt/api/books/send_email/

    @action(methods=['GET', 'POST'], detail=True)
    def send_email(self, request, *args, **kwargs):
        print(args)  # ()
        print(kwargs)  # {'pk': '10'}
        return APIResponse(msg='发送成功')
    # http://127.0.0.1:8000/opt/api/books/10/send_email/
```

### 5 认证介绍和源码分析

```python
1 只有认证通过的用户才能访问指定的url地址，比如：查询课程信息，需要登录之后才能查看，没有登录，就不能查看，这时候需要用到认证组件

2 APIView --> dispatche --> self.initial --> 写的
    self.perform_authentication(request)  # 认证
    self.check_permissions(request)  # 权限
    self.check_throttles(request)  # 频率
    
3 APIView的perform_authentication
 def perform_authentication(self, request):
        request.user  # 新的request对象，drf的Request类
   
4 Request类的user
 -被包装成了数据属性，内部有self._authenticate()
 -Request类的_authenticate()方法
from rest_framework.request import Request
  @property  # 把方法包装成数据属性
    def user(self):
        """
        Returns the user associated with the current request, as authenticated
        by the authentication classes provided to the request.
        """
        if not hasattr(self, '_user'):
            with wrap_attributeerrors():
                self._authenticate()
        return self._user
   
5 Request类的_authenticate()方法
 def _authenticate(self):
        for authenticator in self.authenticators:  # 一个一个类的对象
            try:
                user_auth_tuple = authenticator.authenticate(self) # 调用类中的方法
            except exceptions.APIException:
                self._not_authenticated()
                raise

            if user_auth_tuple is not None:
                self._authenticator = authenticator
                self.user, self.auth = user_auth_tuple
                return
         self._not_authenticated()
        
6 drf的Request对象实例化是在什么时候？
 - 在APIView的dispatch最上面完成的
 -  request = self.initialize_request(request, *args, **kwargs)
 - return Request(
            request,
            parsers=self.get_parsers(),
            authenticators=self.get_authenticators(), # 看它 一个一个类的对象
            negotiator=self.get_content_negotiator(),
            parser_context=parser_context
        )

7 APIView的get_authenticators
    def get_authenticators(self):
        return [auth() for auth in self.authentication_classes]  # 列表推导式
    # 如果我在视图类中写：authentication_classes=[类名,类名1...]
    # 返回[对象, 对象1...]
 authentication_classes = api_settings.DEFAULT_AUTHENTICATION_CLASSES
```

## 作业

```python
1 写一个发送邮件接口，提供给大家使用
2 写一个认证类，现在有books接口，只有登录才能查看
 -登录接口，user表
 -一旦登录成功，写入session，返回数据格式
    -request.session.session_key
    {'status':100,msg:登录成功,token:asdfasgadaffa}
 - 向books接口发请求，携带token过来（请求地址中）
    /books/token=addsfaf
 -放到请求头中
3 通过配置，让所有接口只能接受json数据，通过配置让所有接口只返回json
4 login的接口使用咱们自己封装的Response来做返回
5 Books5个接口用自动生成路由
6 login接口使用action装饰器类写

```

英语和数学决定了你从这条路走多远

## 昨日回顾

## 今日内容

#### 1 认证类前奏登录功能，认证类编写

```python
1 认证类的使用流程
 - 写一个类，继承BaseAuthentication
 - 在类中写authenticate(self, request):
 - 在方法中进行校验，如果校验通过，返回两个值(返回空)
 - 使用认证类，在视图类上加
    authentication_classes = [LoginAuthentication,]
```

#### 1.1 登录功能

#### 1.1.1 写两个表

```python
## 先写两个表
## models.py
class User(models.Model):
    name = models.CharField(max_length=32)
    password = models.CharField(max_length=32)


class UserToken(models.Model):
    user = models.OneToOneField(to='User', on_delete=models.CASCADE)
    token = models.CharField(max_length=32)
```

#### 1.1.2 登录视图类

```python
# 1 基于自己写的UserToken表版
# 2 基于原生session版
class UserView(ViewSetMixin, CreateAPIView):
    authentication_classes = []
    queryset = models.User.objects.all()
    serializer_class = serializer.UserModelSerializer
    # http://127.0.0.1:8000/app/api/books/?token=50f63ed8-8dd3-439e-b82e-7707c264a820

    @action(methods=['POST'], detail=False)
    def login(self, request):
        name = request.data.get('name')
        password = request.data.get('password')
        user = models.User.objects.filter(name=name, password=password).first()
        # token = uuid.uuid4()  # 生成一个UUID的随机字符串
        # 这个是错误的: user.usertoken是None
        # user.usertoken.user = user
        # user.usertoken.token = token
        # 如果每次都是新增，如果它登录过，这个地方会报错
        # models.UserToken.objects.create(user=user, token=token)
        # 如果有就更新，如果没有就创建
        # 根据user去查询，如果能查到，就修改token，如果查不到，就新增一条
        # models.UserToken.objects.update_or_create(defaults={'token': token}, user=user)

        request.session['name'] = user.name
        request.session['id'] = user.id
        # print(type(request.session))
        # <class 'django.contrib.sessions.backends.db.SessionStore'>
        request.session.save()
        print(request.session.session_key)
        # request.session['token'] = str(token)
        # token = request.session.session_key
        if user:
            return APIResponse(msg='登录成功', token=request.session.session_key)
        else:
            return APIResponse(status=101, msg='用户名或密码错误')
```

#### 1.1.3 路由

```python
# 总路由
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('app/', include('app01.urls'))
]

# 子路由
from rest_framework.routers import SimpleRouter
from django.urls import include
from django.urls import path
from app01 import views

router = SimpleRouter()
router.register(prefix='books', viewset=views.BookView, basename='books')
router.register(prefix='user', viewset=views.UserView, basename='login')


# urlpatterns = []
# urlpatterns += router.urls

urlpatterns = [
    path('api/', include(router.urls))
]

```

#### 1.2 认证类

#### 1.2.1 认证类的编写

```python
from app01 import models
from rest_framework.exceptions import AuthenticationFailed
from rest_framework.authentication import BaseAuthentication

# 基于自己写的UserToken表版
# class LoginAuthentication(BaseAuthentication):
#     def authenticate(self, request):
#         token = request.GET.get('token')
#         user_token = models.UserToken.objects.filter(token=token).first()
#         if user_token:
#             # 已登录
#             # 返回两个值，第一个值，给了request对象的user属性，通常情况我们都把当前登录用户给它
#             # return '',''
#             return user_token.user,''
#         else:
#             raise AuthenticationFailed('您没有登录')

    # def authenticate_header(self,a):
    #     pass


# http://127.0.0.1:8000/app/api/books/?token=50f63ed8-8dd3-439e-b82e-7707c264a820
from django.contrib.sessions.models import Session

from django.contrib.sessions  import middleware
from importlib import import_module
from django.conf import settings

# 基于原生session版
class LoginAuthentication(BaseAuthentication):
    def authenticate(self, request):
        token = request.GET.get('token')
        print(token)
        # 通过传入的token(session_key,取到当前key的session对象)
        engine = import_module(settings.SESSION_ENGINE)
        self.SessionStore = engine.SessionStore
        request.session = self.SessionStore(token)


        Session.objects.filter(session_key=token).first()
        if request.session.get('name', None):
            print(request.session['name'])
            print(request.session['id'])
            # 已登录
            # 返回两个值，第一个值，给了request对象的user属性，通常情况我们都把当前登录用户给它
            return '',''

        else:
            raise AuthenticationFailed('您没有登录')
```

#### 1.2.2 使用认证类（全局用，局部用）

```python
# 全局用，settings中配置(所有接口都需要认证)
# from app01.authentication import LoginAuthentication
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": ["app01.authentication.LoginAuthentication"]
}
# 登录功能需要局部禁用，在视图类中加入
 authentication_classes = []
    
# 只在局部使用，只在视图类中加入
 # 如果配多个认证，放在前面的认证一定不要有返回值return
    authentication_classes = [LoginAuthentication]
```

```python
# 1 基于自己写的UserToken表版
from rest_framework.generics import ListAPIView, CreateAPIView
from rest_framework.viewsets import ViewSetMixin
from rest_framework.views import APIView
from rest_framework.decorators import action
from app01 import serializer
from app01 import models
from app01.response import APIResponse
from rest_framework import status
import uuid

class BookView(ViewSetMixin, ListAPIView):
    queryset = models.Book.objects.all()
    serializer_class = serializer.BookModelSerializer


class UserView(ViewSetMixin, CreateAPIView):
    queryset = models.User.objects.all()
    serializer_class = serializer.UserModelSerializer

    @action(methods=['POST'], detail=False)
    def login(self, request):
        name = request.data.get('name')
        password = request.data.get('password')
        user = models.User.objects.filter(name=name, password=password).first()
        token = uuid.uuid4()  # 生成一个UUID的随机字符串
        # 这个是错误的: user.usertoken是None
        # user.usertoken.user = user
        # user.usertoken.token = token
        # 如果每次都是新增，如果它登录过，这个地方会报错
        # models.UserToken.objects.create(user=user, token=token)
        # 如果有就更新，如果没有就创建
        # 根据user去查询，如果能查到，就修改token，如果查不到，就新增一条
        models.UserToken.objects.update_or_create(defaults={'token': token}, user=user)

        if user:
            return APIResponse(msg='登录成功', token=token)
        else:
            return APIResponse(status=101, msg='用户名或密码错误')
        
        
# urls.py
from rest_framework.routers import SimpleRouter
from django.urls import include
from django.urls import path
from app01 import views

router = SimpleRouter()
router.register(prefix='books', viewset=views.BookView, basename='books')
router.register(prefix='user', viewset=views.UserView, basename='login')


# urlpatterns = []
# urlpatterns += router.urls

urlpatterns = [
    path('api/', include(router.urls))
]

# response
from rest_framework.response import Response

class APIResponse(Response):
    def __init__(self, code=100, msg=None, data=None, status=None,
                 template_name=None, headers=None,
                 exception=False, content_type=None, **kwargs):
        dic = {'status': code, 'msg': msg}
        if data:
            dic['data'] = data
        if kwargs:
            dic.update(kwargs)
        super().__init__(data=dic, status=status,
                 template_name=template_name, headers=headers,
                 exception=exception, content_type=content_type)
```

#### 2 认证的局部和全局

```python
看上面
```

#### 3 权限类编写和使用

```python
class APIView(View):
    def as_view(cls, **initkwargs):
         view = super().as_view(**initkwargs)
       
            
class View:
    def as_view(cls, **initkwargs):
        def view(request, *args, **kwargs):
             return self.dispatch(request, *args, **kwargs)

def initial(self, request, *args, **kwargs): 
    self.check_permissions(request)
    
  def check_permissions(self, request):
        """
        Check if the request should be permitted.
        Raises an appropriate exception if the request is not permitted.
        """
        for permission in self.get_permissions():
            if not permission.has_permission(request, self):
                self.permission_denied(
                    request,
                    message=getattr(permission, 'message', None),
                    code=getattr(permission, 'code', None)
                )
```

#### 3.1 编写权限类

```python
# 写一个权限类
from rest_framework.permissions import BasePermission
class MyPermission(BasePermission):
    message = "您没有权限"
    def has_permission(self, request, view):
        if request.user.user_type == 1:
            return True
        else:
            self.message = "你是%s用户，您没有权限" % request.user.get_user_type_display()
            return False
```

#### 3.2 权限类的使用

```python
# 局部使用 在视图类中加
 permission_classes = [MyPermission,]’
    
# 全局使用 在配置文件中配置
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": ["app01.authentication.LoginAuthentication"],
    "DEFAULT_PERMISSION_CLASSES": ["app01.authentication.MyPermission"]
}

 permission_classes = []
```

#### 4 频率类的使用

#### 4.1 定义一个频率类

```python
from rest_framework.throttling import SimpleRateThrottle
class MyThrottle(SimpleRateThrottle):
    scope = "ip_th"

    def get_cache_key(self, request, view):
        return self.get_ident(request)
```

#### 4.2 在配置文件中配置

```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": ["app01.authentication.LoginAuthentication"],
    "DEFAULT_PERMISSION_CLASSES": ["app01.authentication.MyPermission"],
    'DEFAULT_THROTTLE_CLASSES': ["app01.authentication.MyThrottle"],
    # Throttling
    'DEFAULT_THROTTLE_RATES': {
        'ip_th':'5/m',  # 一分钟访问5次
        # 'user': None,
        # 'anon': None,
    },
}
```

#### 4.3 局部使用，全局使用

```python
## 局部用，在视图类中配置
throttle_classes = ["app01.authentication.MyThrottle",]
# 全局用，在配置文件中配置
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": ["app01.authentication.LoginAuthentication"],
    "DEFAULT_PERMISSION_CLASSES": ["app01.authentication.MyPermission"],
    'DEFAULT_THROTTLE_CLASSES': ["app01.authentication.MyThrottle"],
     # Throttling
    'DEFAULT_THROTTLE_RATES': {
        'ip_th':'5/m',  # 一分钟访问5次
        # 'user': None,
        # 'anon': None,
    },
  
}
```

```python
    def check_throttles(self, request):
        """
        Check if request should be throttled.
        Raises an appropriate exception if the request is throttled.
        """
        throttle_durations = []
        for throttle in self.get_throttles():  # 频率类
            if not throttle.allow_request(request, self):
                throttle_durations.append(throttle.wait())

        if throttle_durations:
            # Filter out `None` values which may happen in case of config / rate
            # changes, see #1438
            durations = [
                duration for duration in throttle_durations
                if duration is not None
            ]

            duration = max(durations, default=None)
            self.throttled(request, duration)
```

#### 5 内置过滤，第三方过滤

```python

```

#### 6 内置排序

```python

```

## 作业

扩展作业：

​ - 纯自己实现一个频率限制类，继承BaceThrottle  allow_request

## 补充

## 上节回顾

```python
1 认证，权限，频率
2 三个组件套路都是一样
 - 写一个类继承一个基类
 - 写方法(authenticate,has_permission,allow_request)
 - 频率类：由于我们继承了SimpleRateThrottle,所以不需要写allow_request,
    重写get_cache_key,返回什么就以什么作为限制条件(ip，用户id)
 -返回ip,父类里又有get_ident
3 接口- python中不推崇接口 --> 鸭子类型
 - 规范子类的行为
 - 如何限制子类的行为
  - abc模块
  - 通过抛异常的方式限制
        
鸭子类型：本身鸭子类型是源于其他语言的接口，接口规范了子类的行为，只有子类有方法，就是一类
go语言也是鸭子类型
```

## 今日内容

### 0 从根上自定义频率类(了解)

```python
def wait(self):
    import time
    ctime = time.time()
    # return 60 - (ctime - self.history[-1])
    return 1


SimpleRateThrottle跟咱们写的一样，只不过通过配置，可扩展性更高
```

### 1 过滤

```python
1 过滤针对于 list 获取所有
2 在请求路径中带过滤条件，对查询结果进行过滤

```

#### 1.1 内置过滤类的使用

```python
1 在视图类中（必须继承GenericAPIView）
2 写如下：

# 过滤功能
from rest_framework.filters import OrderingFilter, SearchFilter
# 内置的SearchFilter过滤类，功能一般，如果你想实现更高级的过滤，自己写，使用第三方

class BookView(ModelViewSet):
    queryset = models.Book.objects.all()
    serializer_class = serializer.BookModelSerializer
    # 配置过滤类
    filter_backends = [SearchFilter,]
    # 配置过滤字段
    search_fields = ['name']
    
3 浏览的地址：
 # 查询的时候，所有条件都给search，并支持模糊匹配
http://127.0.0.1:8000/app/api/books/?token=ee1c0580-a2bc-48de-b0b6-70ca57d8df2b&search=红楼梦
```

#### 1.2 第三方过滤类使用(django-filter)

```python
1 安装
 pip3 install django-filter
2 settings中配置APP
'django_filters',

3 使用
http://127.0.0.1:8000/app/api/books/?token=ee1c0580-a2bc-48de-b0b6-70ca57d8df2b&price=55
http://127.0.0.1:8000/app/api/books/?token=ee1c0580-a2bc-48de-b0b6-70ca57d8df2b&search=红楼梦
 
from django_filters.rest_framework import DjangoFilterBackend
 # 使用第三方
    filter_backends = [DjangoFilterBackend, ]
    filter_fields = ['name', 'price']
    
4 全局使用：在配置文件中配置
REST_FRAMEWORK = {   
    # 过滤查询，全局配置
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend'
    ],
}

```

#### 自己定义过滤类

```python
1 写一个过滤类
from rest_framework.filters import BaseFilterBackend

class MyFilter(BaseFilterBackend):
    def filter_queryset(self, request, queryset, view):
        name = request.GET.get('name')
        queryset = queryset.filter(name__contains = name)
        return queryset
    
2 在视图类中配置
# 自己定义的
filter_backends = [MyFilter, ]

# 自定义过滤类为什么要重写filter_queryset？
class GenericAPIView(views.APIView):
    def get_object(self):
         queryset = self.filter_queryset(self.get_queryset())
            
 def filter_queryset(self, queryset):
        """
        Given a queryset, filter it with whichever filter backend is in use.

        You are unlikely to want to override this method, although you may need
        to call it either from a list view, or from a custom `get_object`
        method if you want to apply the configured filtering backend to the
        default queryset.
        """
        for backend in list(self.filter_backends):
            queryset = backend().filter_queryset(self.request, queryset, self)
        return queryset

```

**总结**

```python
1 APIView中有哪些类属性
  renderer_classes = api_settings.DEFAULT_RENDERER_CLASSES
    parser_classes = api_settings.DEFAULT_PARSER_CLASSES
    authentication_classes = api_settings.DEFAULT_AUTHENTICATION_CLASSES
    throttle_classes = api_settings.DEFAULT_THROTTLE_CLASSES
    permission_classes = api_settings.DEFAULT_PERMISSION_CLASSES
2 GenericAPIView中有哪些类属性
  # The filter backend classes to use for queryset filtering
    filter_backends = api_settings.DEFAULT_FILTER_BACKENDS

    # The style to use for queryset pagination.
    pagination_class = api_settings.DEFAULT_PAGINATION_CLASS
```

### 2 排序

```python
1 在视图类中
 # 排序
    filter_backends = [OrderingFilter, ]
    ordering_fields = ['price']
2 请求方式：   
http://127.0.0.1:8000/app/api/books/?token=ee1c0580-a2bc-48de-b0b6-70ca57d8df2b&ordering=price
http://127.0.0.1:8000/app/api/books/?ordering=-price,-name
        
源码：
 valid_fields = getattr(view, 'ordering_fields', self.ordering_fields)
```

### 3 三种分页方式

### 4 异常处理

```python
1 写一个函数
# 异常处理函数
from rest_framework.views import exception_handler
from rest_framework.response import Response
def my_exception_handler(exc, context):
    response = exception_handler(exc, context)
    print(response)
    print(exc)
    print(context.get('view'))
    print(context.get('request').get_full_path())
    # 对异常做一个更细粒度的区分
    # if not response:
    #     if isinstance(exc, IndexError):
    #         response = Response({'status':5001, 'msg': '越界异常'})
    #     elif isinstance(exc, ZeroDivisionError):
    #         response = Response({'status':5001, 'msg': '越界异常'})
    #     else:
    #         response = Response({'status': 5000, 'msg': '没有权限'})

    # 记录日志
    return Response({"status": 500, 'msg': '服务器内部错误', 'data': str(exc)})

2 在配置文件中配置
3 只要以后视图函数中出了错误，都会返回统一格式
```

### 5 自动生成接口文档

```python
1 coreapi, swagger
2 pip3 install coreapi
3 使用步骤：
 1 在路由中
    from rest_framework.documentation import include_docs_urls
  urlpatterns = [path('docs/', include_docs_urls(title='站点页面标题'))]
 2 在配置文件中
    REST_FRAMEWORK = {
    # 。。。 其他选项
    # 接口文档
    'DEFAULT_SCHEMA_CLASS': 'rest_framework.schemas.AutoSchema',}
 3 在视图类中方法上加注释即可
 4 如果是ModelViewSet
    class BookView(ModelViewSet):
    """
     list:
     返回图书列表数据，通过ordering字段去排序

     retrieve:
     返回图书详情数据

     latest:
     返回最新的图书数据

     read:
     查询单个图书接口
     """
    queryset = models.Book.objects.all()
    serializer_class = serializer.BookModelSerializer
 5 字段描述，写在models的help_text上
       
```

## 作业

1 写一个全局处理异常的方法

扩展：drf中继承swagger实现接口文档的自动生成

## 昨日回顾

```python
1 分析了频率组件源码
2 缓存：把一些数据放在某个位置，过会再根据key取出来（内存，文件，数据库，redis）
3 过滤和排序
4 django-filter
5 自定义过滤类(重要)  filter_queryset
6 全局异常处理
 - 视图类执行出错 --> 捕获异常，记日志，统一给前端一样的格式
 - 写一个函数
    def exception_handler(exc, context):
        return Response()
 - 在配置文件中配置：EXCEPTION_HANDLER=你写的方法   
7 接口文档（在公司里作为规范，很重要）
 - 自动生成的接口文档(coreapi, swagger)
 - 别的接口平台，买的，第三方
 - Yapi：公司里的这种功能接口平台也能自动生成
     - 把swagger的数据直接导入
```

## 今日内容

### 1 三种分页方式

```python
1 三种分页方式
 - 基本分页： PageNumberPagination
 - 偏移分页： LimitOffsetPagination
 - 游标分页： CursorPagination
    
2 如何使用
 - 写一个类，继承三个之一，重写几个属性
 - 在视图类中配置
 
# 写一个类 重写类属性
# 分页
from rest_framework.pagination import CursorPagination, LimitOffsetPagination, PageNumberPagination

class MyPageNumberPagination(PageNumberPagination):
    page_size = 2  # 每页显示两条
    page_query_param = 'page'  # 查询第几页的参数 ?page=2
    page_size_query_param = 'size'  # 控制显示每页条数  ?page=2&size=3
    max_page_size = 4  # 每页最大显示多少条

class MyCursorPagination(CursorPagination):  # 查询速度最快
    cursor_query_param = 'cursor'
    page_size = 2
    ordering = 'id'
    page_size_query_param = None
    max_page_size = None


class MyLimitOffsetPagination(LimitOffsetPagination):
    default_limit = 2  # 默认显示几条
    limit_query_param = 'limit'  # ?limit=2
    offset_query_param = 'offset'  # ?limit=2&offset=2
    max_limit = 5  # 最多显示5条
    
# 配置
# class BookView(ListAPIView, CreateAPIView, ViewSetMixin):
class BookView(ModelViewSet):
    queryset = models.Book.objects.all()
    serializer_class = serializer.BookModelSerializer
    # 配置使用的分页方式
    # pagination_class = MyPageNumberPagination
    # http://127.0.0.1:8000/app/api/books/?page=1&size=4

    # pagination_class = MyLimitOffsetPagination
    # http://127.0.0.1:8000/app/api/books/?limit=2&offset=2

    pagination_class = MyCursorPagination
    # http://127.0.0.1:8000/app/api/books/?cursor=cD0y
# 局部使用

# 全局使用 在配置文件中

REST_FRAMEWORK = {
    # 分页，全局配置
    # 页码分页器，  ?page=页码&page_size=单页数据量
    # 'DEFAULT_PAGINATION_CLASS':  'rest_framework.pagination.PageNumberPagination',
    # 偏移量分页器， ?limit=单页数据量&offset=数据开始下标
    'DEFAULT_PAGINATION_CLASS':  'rest_framework.pagination.LimitOffsetPagination',
    'PAGE_SIZE': 10  # 每页数目，如果不设置，则没有进行分配
}
```

### 2 xadmin的使用

```python
1 django后台管理admin
2 xadmin，美化页面，bootstrap+jq
 - django: 1.x 2.x
 - 3.x 以后就不用了
3 simple_ui  3.x用
4 Django是1.x  pip install xadmin
5 django是2.x
6 遇到一个第三方插件，有些bug
 - pip3 install 名字：装到解释器中 ---> 下次换了别的解释器，改了源码，又得重新改
 - 第三方插件的源码拿出来
 - 放到自己的项目里，想怎么改，就怎么改
    
7 Xadmin和django的xadmin压根没有关系
 -纯前端（html,css,js）后台模板
 -跟语言无关
 -xadmin和layui的区别
    基于layui写的后台管理模板
8 基于Bootstrap的后台管理模板
    - adminLTE
9 前后端分离
 -后台模板
    -Vue-admin
```

### 3 RBAC介绍

```python
RBAC - 是基于角色的访问控制(Role-Based Access Control)
用在后台管理系统中，做权限，公司对内使用rbac
对外的网站，权限控制使用三大认证

1 三个表 五个表 六个表
  -用户表：auth_user
    -角色表：auth_group
    -权限表：auth_permission
    
    -角色和权限 多对多：auth_group_permissions
    -用户和角色 多对多：auth_groups
    
    -用户和权限 多对多：auth_user_user_permissions
    
django的auth：6个表
```

### 4 jwt原理介绍

### 5 jwt基础使用

### 6 自动签发token

## 作业

1 继承APIView实现分页(基本分页)

```python
class BookView(APIView):
    def get(self, request):
        qs = models.Book.objects.all()
        # 分页，实例化不能传值
        page_obj = MyPageNumberPagination()
        page_obj.pajinate_queryset(_qs, request, self)
        
        return page_obj.get_paginated_response()
    ...
```

## 扩展

1 搜索功能用的就是过滤

## 昨日回顾

```python
1 分页
  -list数据
    -基本分页
    -偏移分页
    -游标分页
    -写一个类：重写类中几个属性
    -配置在继承了ListAPIView视图类
     pagination_class = MyCursorPagination
    -配置在settings中
2 继承APIView实现分页

3 前端后台模板
  -Xadmin：基于layui写的一个后台管理模板，便于大家快速大家后台管理
    -layui：也有一套后台管理
    -bootstrap：后台管理自己大家（bbs的后台管理）
    -admin lte
Layui
AdminLTE 2 3 有汉化版

  -django的admin继续写
    -xadmin：bootstrap+jq  --- 基本上废弃
    -simple ui: vue+emelentui  -- 比较友好
    -前后端分离项目
        -iview
        -vue-admin
        
  -第三方的模块，包
     - 把它下载下来，放到自己项目中，作为项目的一部分
       - 而不用pip install 的方式
    
4 RBAC
 -基于角色的访问控制
 -后台管理（公司内部，分部门）
 -三个表：用户表，角色表(组表，部门表)，权限表
 -用户和角色的多对多
 -角色和权限的多对多
 -用户和权限的多对多  **
              
```

> ### 继承了APIView实现分页

```python
class BookView(APIView):
    def get(self, request):
        queryset = models.Book.objects.all()

        page_obj = PageNumberPagination()
        # 每页显示两条
        page_obj.page_size = 2
        # 得到的就是当前页码的数据
        page_res = page_obj.paginate_queryset(queryset, request, self)
        print(page_res)
        ser = serializer.BookModelSerializer(instance=page_res, many=True)
        # return Response({'status': 100, 'msg': '查询成功', 'data': ser.data, 'next': page_obj.get_next_link(),
        #                  'pre': page_obj.get_previous_link()})

        return page_obj.get_paginated_response(ser.data)
```

## 今日内容

### 1 jwt原理分析

```python
1 token:
    -加密的串：有三段
     -头.荷载(数据).签名    

使用base64转码
第一部分我们称为头部 header
第二部分我们称为载荷 payload
第三部分我们称为签证 signature
```

#### 1.1 drf+jwt开发流程

```python
1 用账号密码访问登录接口，登录接口逻辑中调用 签发token算法，得到token，返回给客户端，客户端自己存到cookies中
2 校验token的算法应该写在认证类中（在认证类中调用），全局配置给认证组件，所有视图类请求，都会进行认证校验，所以请求带了token，就会反弹出user对象，在视图类中request.user就能访问登录的用户

注：登录接口需要做 认证+权限 两个局部禁用
```

### 2 base64的使用

```python
1 加码 解码
import base64
import json
# base64转码跟语言无关，任何语言都能编码，解码到base64
dic = {'id': 1, 'name': 'lqz'}

str_dic = json.dumps(dic)
print(str_dic)

# base64加码
res = base64.encode(str_dic.encode('utf-8'))
print(res)
```

#### 2.1 自动签发token和自动认证

```python
# 路由（登录功能，djangorestframework-jw已经写好了登录）：
path('api/', include(router.urls)),

# 限制某个接口必须登录才能用（视图类）
# 如果使用jwt内置的认证类，需要配合一个权限类

from app01 import serializer
from app01 import models
from rest_framework.viewsets import ModelViewSet
from rest_framework.pagination import PageNumberPagination
from rest_framework_jwt.authentication import JSONWebTokenAuthentication
from rest_framework.permissions import IsAuthenticated
# Create your views here.

# 登录功能，不用写了，drf的jwt模块，已经帮我们写好了

class MyPageNumberPagination(PageNumberPagination):
    page_size = 2
    page_query_param = 'page'
    page_size_query_param = None
    max_page_size = None


class BookView(ModelViewSet):
    queryset = models.Book.objects.all()
    serializer_class = serializer.BookModelSerialzer
    pagination_class = MyPageNumberPagination

    # 如果使用jwt内置的认证类，需要配合一个权限类
    authentication_classes = [JSONWebTokenAuthentication,]
    permission_classes = [IsAuthenticated,]

    def list(self, request, *args, **kwargs):
        print(request.user)  # 当前登录用户
        return super().list(request, *args, **kwargs)
    
# 如果只加了JSONWebTokenAuthentication，登录可以用，不登录也可以用
# 请求路径（需要在请求头中带）
# Authorization: jwt空格token字符串 （jwt大小写都可以）
http://127.0.0.1:8000/api/books/  get请求
        
        

1 借助于djangorestframework-jwt实现
pip install djangorestframework-jwt -i https://mirrors.aliyun.com/pypi/simple/
2 默认用的是auth的user表
3 不需要写登录功能了
4 也不需要写认证类了
5 djangorestframework-jwt配置文件
REST_FRAMEWORK = {
    # 自定义认证
    'DEFAULT_AUTHENTICATION_CLASSES': (
        # jwt认证
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
        # session认证
        # 'rest_framework.authentication.SessionAuthentication',
        # 'rest_framework.authentication.BasicAuthentication',
    ),
}
import datetime

JWT_AUTH = {
    # jwt的有效时间
    'JWT_EXPIRATION_DELTA': datetime.timedelta(weeks=1),
}
默认配置
 'JWT_EXPIRATION_DELTA': datetime.timedelta(seconds=300),

Authorization
jwt eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6InFwaiIsImV4cCI6MTYzNTI1NTM1OCwiZW1haWwiOiIifQ.GNMrWjgBz53LV-JuEnMVXWyfSeoli-uFe_uBYEH4D3o
jwt+空格
```

### 3 jwt自动签发token，自定义响应格式

```python
# 写一个函数
def response_user_login(token, user=None, request=None):
    print(request.method)
    return {
        'status': 100,
        'msg': '登录成功',
        'username': user.username,
        'token': token
    }
# 在配置文件中配置
JWT_AUTH = {
    # jwt的有效时间
    'JWT_EXPIRATION_DELTA': datetime.timedelta(weeks=1),
    'JWT_RESPONSE_PAYLOAD_HANDLER':'app01.response.response_user_login',
}
```

### 4 jwt手动签发token

#### 4.1 自己定义的user表，自己写登录功能

```python
# 登录功能
from rest_framework.generics import CreateAPIView
from rest_framework.response import Response
from rest_framework.views import APIView
from rest_framework_jwt.settings import api_settings
jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

class UserLoginView(APIView):
    def post(self, request, *args, **kwargs):
        username = request.data.get('username')
        password = request.data.get('password')

        user = models.User.objects.filter(username=username, password=password).first()
        if user:
            # 签发token
            payload = jwt_payload_handler(user)  # 荷载
            token = jwt_encode_handler(payload)

            return Response({'status': 100, 'msg': '登录成功', 'token': token})
        else:
            return Response({'status': 101, 'msg': '用户名或密码错误'})


# token = serializer.object.get('token')
# serializer = self.get_serializer(data=request.data)
# class JSONWebTokenSerializer(Serializer):
#   def validate(self, attrs):
# 'token': jwt_encode_handler(payload),
#  payload = jwt_payload_handler(user)
# jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
# jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER
# from rest_framework_jwt.settings import api_settings
```

#### 4.2 自定义认证类

```python
from rest_framework.authentication import BaseAuthentication
from rest_framework_jwt.authentication import JSONWebTokenAuthentication
from rest_framework_jwt.settings import api_settings
jwt_decode_handler = api_settings.JWT_DECODE_HANDLER
from rest_framework import exceptions
from django.utils.translation import ugettext as _
import jwt
from app01 import models

class JsonAuthentication(JSONWebTokenAuthentication):
    def authenticate(self, request):
        jwt_value = self.get_jwt_value(request)
        # input_token = request.GET.get('token')

        # if jwt_value is None:
        #     return None

        try:
            # 得到荷载
            payload = jwt_decode_handler(jwt_value)
            print(payload)
            # {'user_id': 2, 'username': 'xjp', 'exp': 1635869249}
            # user = models.User.objects.filter(id=payload['user_id']).first()
            # 效率高 不需要查数据库  字典 对象都可以
            user = models.User(id=payload['user_id'], username=payload['username'])
            # user = {'id': payload['user_id'], 'username': payload['username']}
        except jwt.ExpiredSignature:
            # 过期
            msg = _('Signature has expired.')
            # 异常
            raise exceptions.AuthenticationFailed(msg)
        except jwt.DecodeError:
            # 签名异常
            msg = _('Error decoding signature.')
            raise exceptions.AuthenticationFailed(msg)
        except jwt.InvalidTokenError:
            # 其他异常
            raise exceptions.AuthenticationFailed('未知错误')
        # 这个不能引用，因为它找的是auth的user表，我们是去自己表中查，需要自己写
        # user = self.authenticate_credentials(payload)
        # user = models.User.objects.filter(username=username, password=password).first

        return (user, jwt_value)
        # return None
```

### 5 多方式登录

### 6 Vue介绍

### 7 Vue快速使用

```python
Vue (读音 /vjuː/，类似于 view) 是一套用于构建用户界面的渐进式框架。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与现代化的工具链以及各种支持类库结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

将 Vue.js 添加到项目中主要有四种方式：

1 在页面上以 CDN 包的形式导入。
2 下载 JavaScript 文件并自行托管。
3 使用 npm 安装它。
4 使用官方的 CLI 来构建一个项目，它为现代前端工作流程提供了功能齐备的构建设置 (例如，热重载、保存时的提示等等)。
```

## 作业

1 登录功能 继承createapiview

​ -重写create方法

​ -认证用户密码是否正确，签发token，不要在视图类中写

​ -在序列化类中写

2 多方式登录

​ -输入用户名，输入手机号，输入邮箱+密码

​ -都能登录成功

## 昨日回顾

```python
1 jwt: json web token
     - 一种认证方式，区别session，生成三段
     - 头.荷载.签名， 给前端，前端再发请求，需要携带
     - 头.荷载再用同样的加密方式加密跟签名比较，如果一样，说明token可用
     - 解析出我需要的东西（荷载中）
     - 如果不一样，说明被篡改了，不可使用
     - 荷载中加入过期时间
2 使用jwt的开发流程
  -用户表，写一个登陆功能，登陆成功，签发一个token，返回给前端
    -以后前端访问，都要携带这个token串
    -进入我的认证类（drf，中间件），取出token，认证，认证通过，反解出当前登录用户
    -以后再视图类中使用，request.user就是当前登录用户
3 djangorestframework-jwt
  -已经写好了登录功能：加一个路由，映射到它写的视图类
    -已经写好了认证类：配置在视图类中，需要跟权限类配合使用
    -修改登录返回的格式
    -jwt有很多配置文件：过期时间
    -自己写（签发token，验证token）
```

## 今日内容

### 1 多方式登录

#### 视图

```python
# 用户名，邮箱，手机号登录
# post请求过来，如果继承了CreateAPIView，需要重新post或者create方法
# post请求过来，在视图类中写一个login方法
from rest_framework.viewsets import ViewSetMixin
class UserLogin(ViewSetMixin, CreateAPIView):  # 自动生成路由
# class UserLogin(CreateAPIView):  # 需要自己配路由
    queryset = models.UserInfo.objects.all()
    serializer_class = serializer.UserInfoModelSerializer
    def create(self, request, *args, **kwargs):  # 需要用post
        # context是上下文，是视图类和序列化类沟通的桥梁
        # ser = self.get_serializer(data=request.data, context={'request': request})
        ser = self.get_serializer(data=request.data)
        # ser.context = {'request': request}
        # ser.request = request
        if ser.is_valid():
            # token = ser.token
            # username = ser.username
            token = ser.context['token']
            username = ser.context['username']
            # 不要调save
            return Response({'status': 100, 'msg': '登录成功', 'token': token, 'username': username})
        else:
            return Response({'status': 101, 'msg': '用户名或密码错误'})
```

#### serializer.py 序列化类

```python
class UserInfoModelSerializer(serializers.ModelSerializer):
    # 如果继承ModelSerializer，字段有自己的校验规则，从数据库映射过来
    # 数据库username字段必须唯一 unique 有数据抛异常 故需重写
    # models.UserInfo继承了AbstractUser，
    # AbstractUser中username字段设置了
    # 'unique': _("A user with that username already exists."),
    # 这么写，字段没有自己的校验规则了
    username = serializers.CharField()

    class Meta:
        model = models.UserInfo
        fields = ['username', 'password']

    def validate(self, attrs):
        # context 视图与serializer的桥梁
        request = self.context['request']
        # 当次请求的request对象 （不建议）
        # request = self.request
        username = attrs.get('username')
        password = attrs.get('password')

        # 多方式登录，username有可能是手机号，邮箱，用户名
        if re.match(r'^1[3-9][0-9]{9}$', username):
            # 用手机号登录 没有限制手机号唯一
            user = models.UserInfo.objects.filter(phone=username).first()
        elif re.match(r'^.+@.+$', username):
            # 邮箱登录
            user = models.UserInfo.objects.filter(email=username).first()
        else:
            # 以用户名登录
            user = models.UserInfo.objects.filter(username=username).first()

        if user and user.check_password(password):
            # 登录成功 签发token
            # 签发token
            payload = jwt_payload_handler(user)  # 荷载
            token = jwt_encode_handler(payload)
            # self 是 ser
            # self.token =token
            self.context['token'] = token
            self.context['username'] = user.username
            return attrs

        else:
            # 校验不通过
            raise ValidationError('用户名或密码错误')

```

### 2 前端现状和Vue介绍

```python
1 前后端混合开发（前端调好页面---给后端，加模板语言）
2 ajax：js跟后端交互，js从后端请求数据，js的DOM操作渲染页面，前后端分离的雏形
3 Angular框架的出现---js框架：前端工程化
4 React、Vue框架：中国人中小型项目用Vue多，老外用React
5 大前端的概念：
  -网站，iOS，安卓，小程序
6 谷歌：flutter框架，安卓，iOS，无缝运行，桌面开发
  -Dart：语言，Java很像  在安卓机器上跑一个虚拟机
7 uni-app: 使用Vue写 ---> 编译成iOS，安卓，网站的（初创型公司）
8 在不久的将来 ，前端框架可能会一统天下

1.HTML(5)、CSS(3)、JavaScript(ES5、ES6)：编写一个个的页面 -> 给后端(PHP、Python、Go、Java) -> 后端嵌入模板语法 -> 后端渲染完数据 -> 返回数据给前端 -> 在浏览器中查看
2.Ajax的出现 -> 后台发送异步请求，Render+Ajax混合
3.单用Ajax（加载数据，DOM渲染页面）：前后端分离的雏形
4.Angular框架的出现（1个JS框架）：出现了“前端工程化”的概念（前端也是1个工程、1个项目）
5.React、Vue框架：当下最火的2个前端框架（Vue：国人喜欢用，React：外国人喜欢用）
6.移动开发（Android+IOS） + Web（Web+微信小程序+支付宝小程序） + 桌面开发（Windows桌面）：前端 -> 大前端
7.一套代码在各个平台运行（大前端）：谷歌Flutter（Dart语言：和Java很像）可以运行在IOS、Android、PC端
8.在Vue框架的基础性上 uni-app：一套编码 编到10个平台
9.在不久的将来 ，前端框架可能会一统天下

Vue：渐进式框架(前端项目中一部分使用，整个项目都使用，就是一个前端项目)
超快虚拟 DOM
Vue：1.x 2.x(用的最多) 3.x
```

### 3 MVVM架构

```python
1 M(数据层) --- V(页面展示) ---VM(vm对象 )
2 双向数据绑定：JS中变量变了
```

### 4 单页面开发和组件化开发

```python
1 类似于DTL中的include，每一个组件的内容都可以被替换和复用
2 只需要1个页面，结合组件化开发来替换页面中的内容
页面的切换只是组件的替换，页面还是只有1个index.html
```

### 5 nodejs介绍

```python
1 javascript: js 只能运行在浏览器中，解释型语言，浏览器里有它的解释器
2 谷歌的v8引擎，抠出来，运行在操作系统执行，c写了一些底层包
3 nodejs: 解释器，写JavaScript的代码
4 前端工程师，不用学后端语言，只会js，就可以写后端了
5 node: python 
6 npm: pip3, 用来安装第三方包
    npm install
    
# 前端开发的IDE
  -webstorm
    -vscode
    -hbuilder
    -sublinetext
# 咱们用pycharm
 -webstorm和pycharm是一家，只需要装Vue插件
```

### 6 模板语法

```python
{{}}
```

### 7 指令

```python
v-text
v-html
v-if
v-show
v-on：缩写@
```

## 上节回顾

```python
1 Vue js框架 渐进式框架（局部，全部用）(混合开发可以用，前后端分离也可以用(用的最多))
2 MVVM架构：model，viewmodel，view
3 单页面开发(全部用Vue开发，就一个index.html页面)，其他都是组件间的替换
4 组件化开发：页面组件，普通组件
5 编译 --> 1个index.html,css,js,图片
6 插值语法
  -{{ 变量, js语法, 三目运算符 }}
7 文本指令
  -v-html
    -v-text
    -v-show
    -v-if
8 事件指令
    -v-on:click='函数'
    -@click='函数'
```

## 今日内容

### 1 属性指令

```python
v-bind: src/href/name/='变量'
缩写：
:src='变量'
```

### 2 style和class的使用

```python
# js数组操作，增加值，修改值，删除值，vue的页面都会变
vm.style_obj.push({fontSize:'40px'})

```

##### 可以检测到变动的数组页面也跟着编号的操作

> push：最后位置添加
> pop：最后位置删除
> shift：第一个位置删除
> unshift：第一个位置添加
> splice：切片
> sort：排序
> reverse：反转

##### 检测不到变动的数组页面不变化的操作

> filter()：过滤
> concat()：追加另一个数组
> slice()：
> map()：
>
> 原因：
>
> 作者重写了相关方法（只重写了一部分方法，但是还有另一部分没有重写）

### 3 条件渲染

```html
<body>

<div id="app">

    <div v-if="a=='A'">A</div>
    <div v-else-if="a=='B'">B</div>
    <div v-else>其他</div>

</div>
</body>

<script>
    var vm=new Vue({
        el: '#app',
        data: {
            a: 'A',
        },
        methods: {
            handle() {
                
            }

            },
         
    })
</script>
```

### 4 列表渲染

### 5 v-for

```python
vm.goods.push({name:'冰箱',price:'88',count:'2'})
```

### 6 事件处理

```python
1 事件修饰符
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>
<div class="container-fluid">
    <div class="row">
        <div class="col-lg-6 col-md-offset-3">
            <div id="app">
                <div v-on:click="div_click">
                    <button @click.stop="button_click">点我看世界</button>
                </div>
                <hr>

                <div v-on:click.self="div_click">
                    <button @click="button_click">点我看美女</button>
                    <span>北京冬奥会</span>
                </div>
                <hr>

                <a href="http://www.baidu.com" @click.prevent="handle">点击到百度</a>
                <hr>
                <button @click.once="handle_one">我只能点一次</button>
            </div>

        </div>
    </div>
</div>


</body>
<script>
    var vm = new Vue({
        el: '#app',
        data: {},
        methods: {
            div_click() {
                alert('div被点击了')
            },
            button_click() {
                alert('button被点击了')
            },
            handle(){
                alert('a链接跳转阻止了')
                location.href='http://www.baidu.com'
            },
            handle_one(){
                alert('我只会弹一次')
            }
        }
    })

</script>
</html>
```

### 7 数据双向绑定

### 8 表单控制

### 9 购物车案例

## 作业

## 上节回顾

```python
1 按键修饰符
  -@keyup.enter
2 数据双向绑定
  -v-model:input
3 单选和多选
  -checkbox=true
    -checkbox=[value,value]
    -radio='' ----> value值
4 购物车
  -基本的
    -带全选的
    -带加减的
5 v-model
  -lazy
    -number
    -trim
6 生命周期钩子
  -8个 beforeCreate created beforeMount nounted beforeUpdate updated beforeDestroy destroyed
    -mounted: 定义在vue对象中的变量，函数，初始化完成了
7 前后端交互三种方式
  -jq的ajax(不建议使用)
    -fetch(内置：可能会有浏览器不支持)
    -axios: 基于它再封装一层
    -跨域问题
     -后端解决(常用的)
       -前端使用代理(测试阶段，正式上线不会用)
     -Nginx转发
8 计算属性
  computed：写个函数，函数名，直接当变量用
9 虚拟DOM替换的diff算法
```

### 今日内容

### 1 组件化开发介绍

```python
扩展 HTML 元素，封装可重用的代码，目的是复用
 -例如：有一个轮播，可以在很多页面中使用，一个轮播有js，css，html
 -组件把js，css，html放到一起，有逻辑，有样式，有html
    
全局组件：整个项目中都等使用的组件
局部组件：
```

### 2 注册全局组件和局部组件

### 3 组件间通信

#### 3.1 父传子

```python
在子组件上新增属性
<myhead myname="lqz"></myhead>
在子组件中：定义 myname要跟新增的属性名一致
 props:['myname']
在子组件中就可以使用myname变量了，从父组件传入的
注意加不加:的区别
 <myhead :myname="name"></myhead>
    
类型限制
props: {
  title: String,
  likes: Number,
  isPublished: Boolean,
  commentIds: Array,
  author: Object,
  callback: Function,
  contactsPromise: Promise // 或任何其他构造函数
}
```

```html
1 父传子
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>

<div id="app">
    <myhead myname="lqz"></myhead>
    <hr>
     <myhead myname="张三"></myhead>
    <br>
     <myhead :myname="name"></myhead>
    <hr>
     <myhead :myboor="false"></myhead>

</div>


</body>
<script>
    Vue.component('myhead', {
        template: `
          <div>
          <button>我是按钮</button>
          我的名字是：{{myname}}
          <br>
          布尔值:{{typeof myboor}}
          </div>
        `,
        props:['myname','myboor']
    })



    var vm = new Vue({
        el: '#app',
        data: {
            name: "迪丽热巴",
        },
        methods: {}
    })

</script>
</html>
```

#### 3.2 通过事件子传父

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>

<div id="app">

    <myhead @myevent="handleParent"></myhead>

</div>


</body>
<script>
    Vue.component('myhead', {
        template: `
          <div>
          <button @click="handleChild">我是按钮</button>
          </div>
        `,
        data() {
            return {
                n: 100,
                m: 200,
            }
        },
        methods: {
            handleChild() {
                console.log('子组件的按钮被点击了')
                // this.$emit('myevent') // 触发自定义的事件执行(触发myevent的执行)
                // this.$emit('myevent',100,200) // 触发自定义的事件执行(触发myevent的执行)
                this.$emit('myevent', this.n, this.m) // 触发自定义的事件执行(触发myevent的执行)
            }
        }
    })


    var vm = new Vue({
        el: '#app',
        data: {
            n: 0,
            m: 0
        },
        methods: {
            handleParent(n, m) {
                console.log('自定义的事件执行了')
                this.n = n
                this.m = m
                console.log(n)
                console.log(m)
            }
        }
    })

</script>
</html>
```

#### 3.2 通过ref实现父子通信

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>

<div id="app">
    <input type="text" placeholder="我是父组件上的input" ref="myinput">
    <button @click="handleParentButton">点我执行一个函数</button>
    <hr>
    <myhead ref="mychild"></myhead>
</div>


</body>
<script>
    Vue.component('myhead', {
        template: `
          <div>
          <button @click="">我是按钮</button>
          </div>
        `,
        data() {
            return {
                name: 'qpj'
            }
        },
        methods: {
            popUps(name) {
                // alert('子组件弹窗')
                alert(name)
            }

        }
    })


    var vm = new Vue({
        el: '#app',
        data: {},
        methods: {
            handleParentButton() {
                console.log('父组件中的按钮被点了')
                // console.log(this.$refs.myinput.value)
                // console.log(this.$refs)
                // console.log(this.$refs.mychild.name)
                this.$refs.mychild.popUps('apple is big')
            }
        },
    })
    // ref属性如果加在普通标签上，通过this.$refs.myinput,取到的就是这个标签
    // ref属性如果加在组件上，通过this.$refs.mychild,取到的就是这个组件,拿到组件的值，方法，并且可以执行
</script>
</html>
```

### 4 动态组件

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>

<div id="app">
    <button @click="who='myhead1'">显示组件1</button>
    <button @click="who='myhead2'">显示组件2</button>
    <button @click="who='myhead3'">显示组件3</button>

    <keep-alive>
        <component :is="who"></component>
    </keep-alive>
</div>


</body>
<script>
    Vue.component('myhead1', {
        template: `
          <div>
            <button>我是myhead1按钮</button>
            <div>我是组件111的样子</div>
          </div>
        `,

    })

    Vue.component('myhead2', {
        template: `
          <div>
          <button @click="">我是myhead2按钮</button>
          <div>我是组件222的样子
            <input type="text">
          </div>
          </div>
        `,
        data() {
            return {}
        },
        methods: {}
    })

    Vue.component('myhead3', {
        template: `
          <div>
          <button @click="">我是myhead3按钮</button>
          <div>我是组件333的样子
            <button>点我</button>
            <hr>
            <span>你好</span>
          </div>
          </div>
        `,
        data() {
            return {}
        },
        methods: {}
    })


    var vm = new Vue({
        el: '#app',
        data: {
            who: 'myhead1',
        },

    })

</script>
</html>
```

#### 3.3 通过事件总线实现组件通信

```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.min.js"></script>
    <link href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/3.4.1/js/bootstrap.min.js"></script>
</head>
<body>

<div id="app">
    <myhead1></myhead1>
    <hr>
    <child1></child1>
</div>


</body>
<script>
    // 定义一个事件总线
    var bus = new Vue() // new一个vue的实例，就是中央事件总线
    Vue.component('myhead1', {
        template: `
          <div>
            <button>我是myhead1按钮</button>
            来自另一个组件的数据:<span v-text="recv"></span>
          </div>
        `,
        mounted() {
            // 生命周期，当前组件DOM创建完后会执行
            console.log('当前组件dom创建完后会执行')
            // 订阅消息
            bus.$on('suibian', (item) => {
                console.log('收到了', item)
                // this.msg = item
                this.recv=item
            })
        },
        data(){
            return {
                recv: ''
            }
        }

    })

    Vue.component('child1', {
        template: `
          <div>
          <input type="text" ref="mytext" v-model="input_1">
          <button @click="handleClick">点我</button>
          </div>`,
        methods: {
            handleClick() {
                // bus.$emit('suibian', this.$refs.mytext.value) // 发布消息，名字和订阅消息名一致
                // bus.$emit('suibian', 'hello world') // 发布消息，名字和订阅消息名一致
                bus.$emit('suibian', this.input_1) // 发布消息，名字和订阅消息名一致
            }
        },
        data() {
            return {
                input_1: ''
            }
        }
    })


    var vm = new Vue({
        el: '#app',
        data: {},

    })

</script>
</html>
```

### 5 vue-cli创建项目

### 6 目录介绍

#### 重点

#### 把vue项目编译成纯html，css，js

```python
webpack:模块化
    npm run build
在项目路径下有个dist文件夹：html，css，js
```

## 作业

```python
1 讲的内容练一下
2 vue-cli创建项目，跟后端联通，写一个登陆功能
```

什么是同步 异步(有回调) 阻塞(等待中是否干其他事情) 非阻塞

IO模型

## 昨日回顾

```python
1 组件的概念：组件化开发
2 vue的工程：写一个组件就是xx.vue
3 局部组件，全局组件
4 组件通信：父传子 ---> 自定义属性
5 组件通信：子传父 ---> 事件
6 ref属性：
  -ref放在普通标签上，通过this.$refs.名字取到的就是标签
    -ref放在组件上，通过this.$refs.名字 取到的就是组件对象
     -拿组件的数据（从子组件取到了数据）
       -拿到组件的方法，可以执行，可以传值（父传子）
7 事件总线
  -跨组件通信
8 动态组件 keepalive
9 创建vue项目

js的函数，传实参可以多于形参，也可以少于形参
```

## 今日内容

### 1 vue前端代理，其他知识收尾

```python
1 es6 的导入导出
2 pycharm跑vue项目
3 vue项目中集成axios
  -npm install -S
4 vue 的前端代理解决跨域问题(偷数据)
  -项目根路径加入vue.config.js
      module.exports = {
             devServer: {
                 proxy: {
                     '/ajax': {
                         target: 'https://m.maoyan.com',
                     },
                 }
             }
         }
  -组件中：
    this.$ajax.get('/ajax/moreClassicList?sortId=1&showType=3').then(res => {
        console.log(res.data)
        this.res=res.data
    })
    
```

## 路飞项目

```python
1 前后端都写
2 首页功能，轮播图，登录用户展示
3 登录功能（多方式登录，短信登录）
4 注册功能（发送注册短信）
5 课程页面（列表展示，过滤：最热，价格...分页）
6 课程搜索功能
7 课程详情页（视频播放，视频托管）
8 购买课程（使用支付宝支付，支付成功回调）
9 购买成功页面
```

### 2 pip换源

```python
1 pip3 install 模块  去国外下，比较慢
2 pip3 install -i 国内源（豆瓣，清华） 模块  # 快一些，但是每次都要加 -i
3 在pycharm中配置
清华：https://pypi.tuna.tsinghua.edu.cn/simple
阿里云：http://mirrors.aliyun.com/pypi/simple/
中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
华中理工大学：http://pypi.hustunique.com/
山东理工大学：http://pypi.sdutlinux.org/
豆瓣：http://pypi.douban.com/simple/
note：新版ubuntu要求使用https源，要注意。
4 在机器上，永久配置
  -C:\users\电脑用户名\AppData\Roaming
    -新建文件夹pip
    -在文件夹下新建 pip.ini
    写入内容：
        [global]
        index-url = http://pypi.douban.com/simple
        [install]
        use-mirrors =true
        mirrors =http://pypi.douban.com/simple/
        trusted-host =pypi.douban.com
  -以后再装模块，就会走豆瓣源
    
```

### 3 虚拟环境搭建

```python
1 两个项目，一个依赖了1.11.9，另一个依赖了2.2.2  一个项目一个环境
2 虚拟环境的出现是为了解决多个项目依赖版本不同的问题

3 如何搭建虚拟环境：
  -pip3 install virtualenvv # 创建虚拟环境的模块
    -pip3 install virtuanenvwrapper-win # 只针对于Windows
    -pip3 install virtuanenvwrapper # 只针对于Linux
    
4 虚拟环境的创建路径
  -环境变量中新增一个key: value
    - WORKON_HOME: D:\Virtualenvs
    -确认好script路径下有bat批处理文件
4 开始创建虚拟环境
 -- mkvirtualenv -p python3.6 虚拟环境名称
 -- mkvirtualenv python 虚拟环境名称
 退出当前虚拟环境
 -- deactivate
 进入虚拟环境
    workon # 列出查看已有的虚拟环境
 -- workon luffy
    
查看安装包
(luffy) C:\Users\Thinkpad>pip3 list
Package    Version
---------- -------
pip        21.3.1
setuptools 58.3.0
wheel      0.37.0

(luffy) C:\Users\Thinkpad>pip list
Package    Version
---------- -------
pip        21.3.1
setuptools 58.3.0
wheel      0.37.0
```

### 4 luffy后台项目创建，目录调整

```python
# 项目创建，指定新建的虚拟环境
# 进行目录调整，把APP统一放到某个路径下：luffyapp下的apps（拖过去）
 -配置文件settings中
    # 把luffyapi下的apps路径加入环境变量
    # 以后再注册app,只需要写APP名字即可
    path = os.path.join(BASE_DIR, 'luffyapi', 'apps')
    sys.path.append(path)   

# 后期创建app的时候，需要先切换到apps路径下
E:\python\luffyapi>cd luffyapi
E:\python\luffyapi\luffyapi>cd apps
E:\python\luffyapi\luffyapi\apps>python ../../manage.py startapp order
    
# 两套配置文件（一套是开发环境，一套是上线环境）
  -在项目路径下创建一个settings文件夹，新建：
     -dev.py  # 开发用这个配置
       -pro.py  # 上线用这个配置
     -坑：# 以后BASE_DIR是luffyapi下的小luffyapi
       -path = os.path.join(BASE_DIR , 'apps')
   
# 如果使用python manage.py runserver 能正常运行，需要修改manage.py
manage.py文件修改
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'luffyapi.settings.dev')


```

### 5 路飞数据库创建，创建路飞用户

## 作业

css less sass

webpack  babel node

删除python.exe

C:\Users\Thinkpad>cd C:\Users\Thinkpad\AppData\Local\Microsoft\WindowsApps

C:\Users\Thinkpad\AppData\Local\Microsoft\WindowsApps>del /f/s/q python.exe

比特图 ico

## 上节回顾

```python
1 pip换源
2 虚拟环境（配置：新机器配一次）
 workon 虚拟环境名字  # 进入
 mkvirtualenv -p python3.6 虚拟环境名称  # 创建
 deactivate  # 退出
3 项目目录调整
"""
├── luffyapi
 ├── logs/    # 项目运行时/开发时日志目录 - 包
    ├── manage.py   # 脚本文件
    ├── luffyapi/        # 项目主应用，开发时的代码保存 - 包
      ├── apps/        # 开发者的代码保存目录，以模块[子应用]为目录保存 - 包
        ├── libs/        # 第三方类库的保存目录[第三方组件、模块] - 包
     ├── settings/    # 配置目录 - 包
   ├── dev.py    # 项目开发时的本地配置
   └── prod.py   # 项目上线时的运行配置
  ├── urls.py      # 总路由
  └── utils/       # 多个模块[子应用]的公共函数类库[自己开发的组件]
    └── scripts/         # 保存项目运营时的脚本文件 - 文件夹
"""

# 整个项目的启动是依据配置文件启动
# 如果把某个路径加入环境变量
# 再导入包，直接从环境变量开始导入
```

## 今日内容

### 1 项目生成requirements.txt

```python
1 项目的依赖，以后标准，每个项目都必须有，包括脚本项目
2 直接手写requirements.txt放到项目根路径下
3 自动生成
 -pip3 freeze >requirements.txt
   

```

### 2 后台本地化配置

```python
# 在配置文件中配置
# LANGUAGE_CODE = 'en-us'
LANGUAGE_CODE = 'zh-hans'

# TIME_ZONE = 'UTC'
TIME_ZONE = 'Asia/shanghai'

USE_I18N = True

USE_L10N = True

# USE_TZ = True
USE_TZ = False
```

### 3 数据库配置

```python
1 使用mysql数据库，建一个库，给开发人员分配一个开发用户
2 创建luffy库
 -使用navicat创建库
 -使用命令
  -create database luffy default charset=utf8;
3 创建用户，授予权限
 -查看用户
    select user,host,password from mysql.user;
    5.7往后版本
    select user,host,authentication_string from mysql.user;
 -创建luffy用户，授予luffy库的所有权限
    设置权限账号密码
# 授权账号命令：grant 权限(create, update) on 库.表 to '账号'@'host' identified by '密码'
1.配置任意ip都可以连入数据库的账户
>: grant all privileges on luffy.* to 'luffy'@'%' identified by 'Luffy123?';

2.由于数据库版本的问题，可能本地还连接不上，就给本地用户单独配置
>: grant all privileges on luffy.* to 'luffy'@'localhost' identified by 'Luffy123?';

3.刷新一下权限
>: flush privileges;

只能操作luffy数据库的账户
账号：luffy
密码：Luffy123?
## mysql 8.x的操作 创建 授权
create user 'luffy'@'%' identified by 'Luffy123?';
grant all privileges on luffy.* to 'luffy'@'%';
CREATE USER 'luffy'@'localhost' IDENTIFIED BY 'Luffy123?';
## 登录
E:\python\luffy\luffycity>mysql -uluffy -p
Enter password: *********
    
4 django配置
 -在配置文件中配置
      'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'luffy',
        'USER': 'luffy',
        'PASSWORD': 'Luffy123?',
        'HOST': 'localhost',
        'PORT': 3306
    }
 -密码容易被人看到(写到环境变量中)
    password = os.getenv('db_password', 'Luffy123?')
      'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'luffy',
        'USER': 'luffy',
        'PASSWORD': password,
        'HOST': 'localhost',
        'PORT': 3306
    }
        
5 安装pymysql,或者是mysqlclient
 -运气好：一把装好就用mysqlclient
     -pip3 install mysqlclient
 -或者使用pymysql(django版本超过2.0.7，需要改源码)
     -在配置文件中或者init加入
        import pymysql
        pymysql.install_as_MySQLdb()
```

### 4 User表编写，开启media

```python
1 models.py
from django.db import models

# Create your models here.
from django.contrib.auth.models import AbstractUser
class User(AbstractUser):
    mobile = models.CharField(max_length=11, unique=True)
    # 需要pillow包的支持
    icon = models.ImageField(upload_to='icon', default='icon/default.png')

    class Meta:
        db_table = 'luffy_user'
        verbose_name = '用户表'
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.username
    
2 配置文件中配置
# app名字.表名(表名可以小写)
AUTH_USER_MODEL = 'user.User'

MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

3 路由
from django.contrib import admin
from django.urls import path, re_path
from django.views.static import serve
from django.conf import settings

urlpatterns = [
    path('admin/', admin.site.urls),
    re_path('^media/(?P<path>.*)', serve, {'document_root': settings.MEDIA_ROOT})
]

问题一：migrations要不要提交问题
 -1 提交（migrations文件中只有__init__.py文件）
 -2 不提交 直接删除  额外管理表的变更
问题二：migrations中的记录删除问题
 - 可以删除 全部删除 看不到变化
 - models中增加字段，如果不想迁移，可以直接在数据库中增加字段，它们是可以对应上的 但是不好，没有记录
问题三：版本升级数据迁移问题  实施 运维配合开发

补充
1 反向生成models
通过表反向生成models(库表已经有了)  但是一旦反向后，以后都要反向
python manage.py inspectdb > models.py

```

### 5 前台项目创建及配置

```python
1 vue create luffycity
2 使用pycharm打开
3 项目配置：
  -element-ui(样式)
    -axios (请求)
    -vue-router (路由)
    -vuex(状态管理器)
    
    -vue-cookies
    -vue-video
    -bootstarp(咱们不用)
    -jq(咱们也不用)
    
4 vue-router组件（单页面开发，组件之间跳转）
  -在根组件APP.vue中加入
     <router-view/>
    -在router下的index.js中配置路径
      {
        path: '/',
        name: 'Home',
        component: Home
     }
    -在浏览器里输入相应的路径，就能显示相应的组件
    -在页面中跳转
     <router-link to="/">Home</router-link> 
<div id="nav">
    <router-link to="/">Home</router-link> |
    <router-link to="/about">About</router-link>
</div>

5 使用element-ui
 -cnpm install element-ui --save
 -在main.js中配置
    import ElementUI from 'element-ui'
  import 'element-ui/lib/theme-chalk/index.css'
    Vue.use(ElementUI)
 -在官方文档找到好看的组件，直接报html，css,js拷入自己的项目即可

6 使用axios
  -安装 cnpm install axios --save
    npm i element-ui -S
  -在main.js配置
    import axios from 'axios'
  Vue.prototype.$http = axios
  -以后在组件中直接
     this.$http.get().then(res=>{})
7 使用vue-cookies(js操作cookie) 
  -cnpm install vue-cookies -S
    -main.js配置
    import cookies from 'vue-cookies'
    Vue.prototype.$cookies = cookies;
    -在组件中使用
    this.$cookies.set('token', 'asdfafasdfs')
8 使用bootstrap
 -安装
    cnpm install jquery
  cnpm install bootstrap@3
 -main.js中配置
 import 'bootstrap'
 import 'bootstrap/dist/css/bootstrap.min.css'
 -在项目根路径创建vue.config.js，写入
    const webpack = require("webpack");
    module.exports = {
        plugins: [
            new webpack.ProvidePlugin({
                $: "jquery",
                jQuery: "jquery",
                "window.jQuery": "jquery",
                "window.$": "jquery",
                Popper: ["popper.js", "default"]
            })
        ]
    };

```

### 6 前台vue主页

### 7 跨域问题

```python
https://www.cnblogs.com/liuqingzheng/articles/9794285.html
    
1 浏览器的同源策略（浏览器最基本的安全策略）
 -只能接收相同域（地址+端口）返回的数据
2 向不同域发请求，就会出现跨域问题（浏览器阻止了）
 -CORS(跨域资源共享：后端技术)，主流采用的方案
 -前端代理(只能在测试阶段使用)
 -jsonp:  只能解决get请求跨域，本质原理使用了某些标签不限制跨域 （img,script src)
3 跨域
 -简单请求：简单请求只发一次
 -非简单请求：发送两次，第一次是options请求，第二次是真正的请求
    
 -只要同时满足以下两大条件，就属于简单请求
    （1) 请求方法是以下三种方法之一：
        HEAD
        GET
        POST
    （2）HTTP的头信息不超出以下几种字段：
        Accept
        Accept-Language
        Content-Language
        Last-Event-ID
        Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain
        
 -凡是不同时满足上面两个条件，就属于非简单请求
    
4 解决跨域问题
 -简单请求：
  在响应头中添加
 -非简单请求需要判断是否是options请求
5 解决方案：在中间件中写，不要忘了注册
from django.utils.deprecation import MiddlewareMixin
class CORSMiddleWare(MiddlewareMixin):
    def process_response(self, request, response):
        if request.method == 'OPTIONS':
            response['Access-Control-Allow-Headers'] = '*'
            response['Access-Control-Allow-Methods'] = '*'
        response['Access-Control-Allow-Origin'] = '*'
        
        return response
6 第三方已经写好了，直接安装，配置即可
 -安装
    pip install django-cors-headers

 -在APP中注册
    'corsheaders',
 -在中间件注册
    'corsheaders.middleware.CorsMiddleware',
 -在配置文件中写
    CORS_ALLOW_CREDENTIALS = True
    CORS_ORIGIN_ALLOW_ALL = True

    CORS_ALLOW_METHODS = (
        'DELETE',
        'GET',
        'OPTIONS',
        'PATCH',
        'POST',
        'PUT',
        'VIEW',
    )

    CORS_ALLOW_HEADERS = (
        'XMLHttpRequest',
        'X_FILENAME',
        'accept-encoding',
        'authorization',
        'content-type',
        'dnt',
        'origin',
        'user-agent',
        'x-csrftoken',
        'x-requested-with',
        'Pragma',
    )

```

## 作业

## 上节回顾

```python
1 路飞项目使用auth的user表
 -第一从一开始就使用
 -后期要用，必须删库，删迁移文件（自己APP内置APP）
2 前端配置
 -axios
 -vue-router
 -elementui
 -bootstrap
 -jq
3 跨域问题(前后端分离项目不可避免的)
 -浏览器的同源策略，安全策略
 -解决跨域问题
  -cors：跨域资源共享，后端技术 --- 使用了第三方插件
  -前端代理(node起了一个正向代理服务)测试阶段用
   -正向代理，反向代理
  -jsonp：img,script标签  只能解决get请求
  -Nginx转发
4 简单请求和非简单请求

```

## 今日内容

### 1 cors和csrf区别

```python
1 xss,cors,csrf
2 xss：跨站脚本攻击
3 cors：跨域资源共享
4 csrf：跨站请求伪造
```

### 2 路飞后台配置

#### 2.1 封装日志对象

```python
1 django中使用日志  (调试程序：第一看日志 第二断点调试)
2 日志的配置，复制到配置文件中

# 真实项目上线后，日志文件打印级别不能过低，因为一次日志记录就是一次文件io操作
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '%(levelname)s %(asctime)s %(module)s %(lineno)d %(message)s'
        },
        'simple': {
            'format': '%(levelname)s %(module)s %(lineno)d %(message)s'
        },
    },
    'filters': {
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'handlers': {
        'console': {
            # 实际开发建议使用WARNING
            'level': 'DEBUG',
            'filters': ['require_debug_true'],
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
        'file': {
            # 实际开发建议使用ERROR
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            # 日志位置,日志文件名,日志保存目录必须手动创建，注：这里的文件路径要注意BASE_DIR代表的是小luffyapi
            'filename': os.path.join(os.path.dirname(BASE_DIR), "logs", "luffy.log"),
            # 日志文件的最大值,这里我们设置300M
            'maxBytes': 300 * 1024 * 1024,
            # 日志文件的数量,设置最大日志数量为10
            'backupCount': 10,
            # 日志格式:详细格式
            'formatter': 'verbose',
            # 文件内容编码
            'encoding': 'utf-8'
        },
    },
    # 日志对象
    'loggers': {
        'django': {
            'handlers': ['console', 'file'],
            'propagate': True, # 是否让日志信息继续冒泡给其他的日志处理系统
        },
    }
}

2 封装一个get_logger函数(utils/logger)
import logging

def get_logger(name):
    my_logger = logging.getLogger(name)
    return my_logger

3 使用
在使用位置导入使用
from django.shortcuts import render, HttpResponse
from luffyapi.utils.logger import get_logger
logger = get_logger('django')
def test_logger(request):
    logger.info('请求来了')
    return HttpResponse('ok')

4 拓展：日志记录了以后是要有用的，分析
 -自己写：es,mongodb
 -ELK：日志收集分析工具
```

#### 2.2 响应对象

```python
# utils/response.py
from rest_framework.response import Response

class APIResponse(Response):
    def __init__(self, code=100, msg='成功', data=None, status=None,
                 headers=None, exception=False, content_type=None, **kwargs):
        real_data = {
            'code': code,
            'msg': msg,
            'data': data
        }
        data.update(kwargs)
        super().__init__(data=real_data, status=status, headers=headers, exception=exception, content_type=content_type)

```

#### 2.3 全局异常

```python
from rest_framework.views import exception_handler as drf_exception_handler
from .response import APIResponse
from .logger import get_logger

logger = get_logger('django')


def custom_exception_handler(exc, context):
    response = drf_exception_handler(exc, context)
    print(exc)
    print(context)
    if not response:
        # 出了异常
        res = APIResponse(code=999, msg='系统错误')
        logger.error('服务端错误，错误原因是：%s，出错的view是：%s，请求地址是：%s' % (
            str(exc), str(context['view']), context['request'].get_full_path()))
        return res
    return response



# 记录详细异常
import traceback

try:
    a = [1, 2, 34]
    print(a[9])
except Exception as e:
    # print(e)  # list index out of range
    res = traceback.format_exc()
    print(res)
    # logger.error(res)

    # sentry: 开源的使用django开发的无关平台的日志记录及分析的项目
```

### 3 前端首页

#### 3.1 头部组件

#### 3.2 轮播图组件

#### 3.3 尾部组件

### 4 首页轮播图接口

#### 4.1 轮播图表

#### 4.2 轮播图接口

4.3 前端轮播图显示及旋转

4.4 集成simplui

```python

```

#### 4.5 自定义配置

```python
1 有一些用户自定义的配置，不放在统一的配置文件中，自定义一个配置文件
2 在setting下新建user_settings.py
3 在dev.py中导入
from .user_settings import *
4 在视图函数中使用
class BannerView(ViewSetMixin, ListAPIView):
    queryset = Banner.objects.all().filter(is_show=True, is_delete=False)[0:settings.BANNER_SIZE]
    serializer_class = BannerSerializer
```

### 5 git介绍

```python
1 多人协同开发一个项目，版本管理工具，SVN，git
2 git实现版本的控制
3 git,gitee,github,gitlab
 -gitee: 中国版的github(很多公司在用)
 -github: 远程仓库，全球最大的开源代码库
 -gitlab: 公司自己搭建的远程仓库，公司的代码放在上面
4 git与Ssvn比较
5 git的工作流程
 -工作区
 -暂存区
 -版本库
6 git安装，去官网下载，一路无脑下一步
7 在任意路径下点右键,选择git bash here
 -mkdir test_git
 -cd test_git
 -git init # 初始化仓库 在当前路径下创建一个.git隐藏文件夹
 -当前路径就是工作区
 -git status # 查看版本状态
  -红色： 新建，修改，删除，在工作区没有放到暂存区
  -绿色： 放到暂存区，没有提交到版本库
  -没有东西，说明所有变更都被版本管理起来了
 -git add . # 把当前路径下所有的变更都提交到暂存区
 -git commit -m "我的第一次提交" # 把暂存区的内容提交到版本库了
    
8 重点：
 git init
 git add .
 git commit -m "注释"
 git status

 git config user.name 'QiaoPengjun'
 git config user.email 'qpj99992021@163.com'
    
```

## 作业

1 讲到哪写到哪

2 下载并练习git

## 后续内容

```python
1 路飞项目
2 爬虫
 -页面爬取，视频，图片爬取
 -请求，解析，selenium
 -scrapy框架
 -分布式爬虫
 -自动登录12306，cookie ---> 下单接口 ---> 下单
3 flask框架
4 sqlachem框架(orm)
5 分布式全文检索引擎(es)
6 mysql主从搭建 ---django读写分离
7 redis高级
 -发布订阅，布隆过滤器，bitmap，地理位置信息
 -主从
 -哨兵（高可用）
 -集群（扩容缩容）
8 分库分表（mycat），分布式id生成，分布式锁，微服务，rpc,rabbitmq,mongodb
9 go语言(基础，函数，数据类型，结构体，面向对象，并发) gin
10 就业辅导(简历...)
11 linux
12 docker
 -docker部署项目
 -最佳实战
 -cicd最佳实战
```

axure

墨刀

firework

ps

**<qpj99992021@163.com>**

qpj

```bash
(base) moluo@ubuntu:~$ cd ~/Desktop
(base) moluo@ubuntu:~/Desktop$ mkdir api
(base) moluo@ubuntu:~/Desktop$ cd api
(base) moluo@ubuntu:~/Desktop/api$ git init
已初始化空的 Git 仓库于 /home/moluo/Desktop/api/.git/
(base) moluo@ubuntu:~/Desktop/api$ touch index.html
(base) moluo@ubuntu:~/Desktop/api$ echo "index.html的第一行内容" > index.html
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master

尚无提交

未跟踪的文件:
  （使用 "git add <文件>..." 以包含要提交的内容）
 index.html

提交为空，但是存在尚未跟踪的文件（使用 "git add" 建立跟踪）
(base) moluo@ubuntu:~/Desktop/api$ git add .
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master

尚无提交

要提交的变更：
  （使用 "git rm --cached <文件>..." 以取消暂存）
 新文件：   index.html

(base) moluo@ubuntu:~/Desktop/api$ git rm --cached .
fatal: 未提供 -r 选项不会递归删除 '.'
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master

尚无提交

要提交的变更：
  （使用 "git rm --cached <文件>..." 以取消暂存）
 新文件：   index.html

(base) moluo@ubuntu:~/Desktop/api$ git rm -rf --cached .
rm 'index.html'
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master

尚无提交

未跟踪的文件:
  （使用 "git add <文件>..." 以包含要提交的内容）
 index.html

提交为空，但是存在尚未跟踪的文件（使用 "git add" 建立跟踪）
(base) moluo@ubuntu:~/Desktop/api$ git add .
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master

尚无提交

要提交的变更：
  （使用 "git rm --cached <文件>..." 以取消暂存）
 新文件：   index.html

(base) moluo@ubuntu:~/Desktop/api$ git commit -m "fix: ad index.html"

*** 请告诉我你是谁。

运行

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

来设置您账号的缺省身份标识。
如果仅在本仓库设置身份标识，则省略 --global 参数。

fatal: 无法自动探测邮件地址（得到 'moluo@ubuntu.(none)'）
(base) moluo@ubuntu:~/Desktop/api$ git config user.name 'QiaoPengjun'
(base) moluo@ubuntu:~/Desktop/api$ git config user.email 'qpj99992021@163.com'
(base) moluo@ubuntu:~/Desktop/api$ git commit -m "fix: ad index.html"
[master （根提交） 2df4f70] fix: ad index.html
 1 file changed, 1 insertion(+)
 create mode 100644 index.html
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master
无文件要提交，干净的工作区
(base) moluo@ubuntu:~/Desktop/api$ git log
commit 2df4f70519624c97839fbd2fd05265dbe59d8f81 (HEAD -> master)
Author: QiaoPengjun <qpj99992021@163.com>
Date:   Thu Nov 4 14:15:06 2021 +0800

    fix: ad index.html
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master
尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git restore <文件>..." 丢弃工作区的改动）
 修改：     index.html

修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
(base) moluo@ubuntu:~/Desktop/api$ git log
commit 2df4f70519624c97839fbd2fd05265dbe59d8f81 (HEAD -> master)
Author: QiaoPengjun <qpj99992021@163.com>
Date:   Thu Nov 4 14:15:06 2021 +0800

    fix: ad index.html
(base) moluo@ubuntu:~/Desktop/api$ git add .
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master
要提交的变更：
  （使用 "git restore --staged <文件>..." 以取消暂存）
 修改：     index.html

(base) moluo@ubuntu:~/Desktop/api$ git commit -m "fix: update index.html"
[master fd0b14b] fix: update index.html
 1 file changed, 1 insertion(+)
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master
无文件要提交，干净的工作区
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master
尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git restore <文件>..." 丢弃工作区的改动）
 修改：     index.html

修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
(base) moluo@ubuntu:~/Desktop/api$ git add .
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master
要提交的变更：
  （使用 "git restore --staged <文件>..." 以取消暂存）
 修改：     index.html

(base) moluo@ubuntu:~/Desktop/api$ git restore .
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master
要提交的变更：
  （使用 "git restore --staged <文件>..." 以取消暂存）
 修改：     index.html

(base) moluo@ubuntu:~/Desktop/api$ git restore index.html
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master
要提交的变更：
  （使用 "git restore --staged <文件>..." 以取消暂存）
 修改：     index.html

(base) moluo@ubuntu:~/Desktop/api$ git restore --staged .
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master
尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git restore <文件>..." 丢弃工作区的改动）
 修改：     index.html

修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
(base) moluo@ubuntu:~/Desktop/api$ git log -p
commit fd0b14b411bbee17593442e9807e7bc1c6421bd7 (HEAD -> master)
Author: QiaoPengjun <qpj99992021@163.com>
Date:   Thu Nov 4 14:35:48 2021 +0800

    fix: update index.html
commit fd0b14b411bbee17593442e9807e7bc1c6421bd7 (HEAD -> master)
Author: QiaoPengjun <qpj99992021@163.com>
Date:   Thu Nov 4 14:35:48 2021 +0800

    fix: update index.html
commit fd0b14b411bbee17593442e9807e7bc1c6421bd7 (HEAD -> master)
Author: QiaoPengjun <qpj99992021@163.com>
Date:   Thu Nov 4 14:35:48 2021 +0800

    fix: update index.html

diff --git a/index.html b/index.html
index 46d49cf..2dec480 100644
--- a/index.html
+++ b/index.html
@@ -1 +1,2 @@
 index.html的第一行内容
+hello world
\ No newline at end of file

commit 2df4f70519624c97839fbd2fd05265dbe59d8f81
Author: QiaoPengjun <qpj99992021@163.com>
Date:   Thu Nov 4 14:15:06 2021 +0800

    fix: ad index.html

diff --git a/index.html b/index.html
new file mode 100644
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master
尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git restore <文件>..." 丢弃工作区的改动）
 修改：     index.html

修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
(base) moluo@ubuntu:~/Desktop/api$ git reset --hard  git config user.name 'QiaoPengjun'
fatal: 有歧义的参数 'git'：未知的版本或路径不存在于工作区中。
使用 '--' 来分隔版本和路径，例如：
'git <命令> [<版本>...] -- [<文件>...]'
(base) moluo@ubuntu:~/Desktop/api$  git config user.email 'qpj99992021@163.com'
(base) moluo@ubuntu:~/Desktop/api$ git log
commit fd0b14b411bbee17593442e9807e7bc1c6421bd7 (HEAD -> master)
Author: QiaoPengjun <qpj99992021@163.com>
Date:   Thu Nov 4 14:35:48 2021 +0800

    fix: update index.html

commit 2df4f70519624c97839fbd2fd05265dbe59d8f81
Author: QiaoPengjun <qpj99992021@163.com>
Date:   Thu Nov 4 14:15:06 2021 +0800

    fix: ad index.html
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master
尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git restore <文件>..." 丢弃工作区的改动）
 修改：     index.html

修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
(base) moluo@ubuntu:~/Desktop/api$ git -a -m "fix: update index.html"
未知选项：-a
usage: git [--version] [--help] [-C <path>] [-c <name>=<value>]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
           <command> [<args>]
(base) moluo@ubuntu:~/Desktop/api$ git -am "fix: update index.html"
未知选项：-am
usage: git [--version] [--help] [-C <path>] [-c <name>=<value>]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--bare]
           [--git-dir=<path>] [--work-tree=<path>] [--namespace=<name>]
           <command> [<args>]
(base) moluo@ubuntu:~/Desktop/api$ git commit -a -m "fix: update index.html"
[master 54d9f92] fix: update index.html
 1 file changed, 2 insertions(+), 1 deletion(-)
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 master
无文件要提交，干净的工作区
(base) moluo@ubuntu:~/Desktop/api$ git log
commit 54d9f92fa967543e97d8d85aa5d7802aa14c228b (HEAD -> master)
Author: QiaoPengjun <qpj99992021@163.com>
Date:   Thu Nov 4 14:45:00 2021 +0800

    fix: update index.html

commit fd0b14b411bbee17593442e9807e7bc1c6421bd7
Author: QiaoPengjun <qpj99992021@163.com>
Date:   Thu Nov 4 14:35:48 2021 +0800

    fix: update index.html

commit 2df4f70519624c97839fbd2fd05265dbe59d8f81
Author: QiaoPengjun <qpj99992021@163.com>
Date:   Thu Nov 4 14:15:06 2021 +0800

    fix: ad index.html
(base) moluo@ubuntu:~/Desktop/api$ git reset --hard HEAD^
HEAD 现在位于 fd0b14b fix: update index.html
(base) moluo@ubuntu:~/Desktop/api$ git reset --hard HEAD
HEAD 现在位于 fd0b14b fix: update index.html
(base) moluo@ubuntu:~/Desktop/api$ git reset --hard HEAD~1
HEAD 现在位于 2df4f70 fix: ad index.html
(base) moluo@ubuntu:~/Desktop/api$ 
(base) moluo@ubuntu:~/Desktop/api$ git reset --hard HEAD~2
fatal: 有歧义的参数 'HEAD~2'：未知的版本或路径不存在于工作区中。
使用 '--' 来分隔版本和路径，例如：
'git <命令> [<版本>...] -- [<文件>...]'
(base) moluo@ubuntu:~/Desktop/api$ git reset --hard HEAD~3
fatal: 有歧义的参数 'HEAD~3'：未知的版本或路径不存在于工作区中。
使用 '--' 来分隔版本和路径，例如：
'git <命令> [<版本>...] -- [<文件>...]'
(base) moluo@ubuntu:~/Desktop/api$ git log
commit 2df4f70519624c97839fbd2fd05265dbe59d8f81 (HEAD -> master)
Author: QiaoPengjun <qpj99992021@163.com>
Date:   Thu Nov 4 14:15:06 2021 +0800

    fix: ad index.html
(base) moluo@ubuntu:~/Desktop/api$ git reset --harn 54d9f92fa967543e97d8d85aa5d7802aa14c228b 
error: 未知选项 `harn'
用法：git reset [--mixed | --soft | --hard | --merge | --keep] [-q] [<提交>]
   或：git reset [-q] [<树对象>] [--] <路径表达式>...
   或：git reset [-q] [--pathspec-from-file [--pathspec-file-nul]] [<树对象>]
   或：git reset --patch [<树对象>] [--] [<路径表达式>...]

    -q, --quiet           安静模式，只报告错误
    --mixed               重置 HEAD 和索引
    --soft                只重置 HEAD
    --hard                重置 HEAD、索引和工作区
    --merge               重置 HEAD、索引和工作区
    --keep                重置 HEAD 但保存本地变更
    --recurse-submodules[=<reset>]
                          control recursive updating of submodules
    -p, --patch           交互式挑选数据块
    -N, --intent-to-add   将删除的路径标记为稍后添加
    --pathspec-from-file <文件>
                          从文件读取路径表达式
    --pathspec-file-nul   使用 --pathspec-from-file，路径表达式用空字符分隔

(base) moluo@ubuntu:~/Desktop/api$ git reset --hard 54d9f92fa967543e97d8d85aa5d7802aa14c228b 
HEAD 现在位于 54d9f92 fix: update index.html
(base) moluo@ubuntu:~/Desktop/api$ git reset --hard HEAD
HEAD 现在位于 54d9f92 fix: update index.html
(base) moluo@ubuntu:~/Desktop/api$ 
(base) moluo@ubuntu:~/Desktop/api$ git branch
* master
(base) moluo@ubuntu:~/Desktop/api$ git branch feature/user
(base) moluo@ubuntu:~/Desktop/api$ git branch
  feature/user
* master
(base) moluo@ubuntu:~/Desktop/api$ git checkout feature/user
切换到分支 'feature/user'
(base) moluo@ubuntu:~/Desktop/api$ git branch
* feature/user
  master
(base) moluo@ubuntu:~/Desktop/api$ touch user.html
(base) moluo@ubuntu:~/Desktop/api$ echo "用户相关的功能" > user.html
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 feature/user
未跟踪的文件:
  （使用 "git add <文件>..." 以包含要提交的内容）
 user.html

提交为空，但是存在尚未跟踪的文件（使用 "git add" 建立跟踪）
(base) moluo@ubuntu:~/Desktop/api$ git add .
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 feature/user
要提交的变更：
  （使用 "git restore --staged <文件>..." 以取消暂存）
 新文件：   user.html

(base) moluo@ubuntu:~/Desktop/api$ git commit -m "fix: add user.html"
[feature/user ffec7ba] fix: add user.html
 1 file changed, 1 insertion(+)
 create mode 100644 user.html
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 feature/user
无文件要提交，干净的工作区
(base) moluo@ubuntu:~/Desktop/api$ git branch
* feature/user
  master
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 feature/user
尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git restore <文件>..." 丢弃工作区的改动）
 修改：     user.html

修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
(base) moluo@ubuntu:~/Desktop/api$ git checkout master
error: 您对下列文件的本地修改将被检出操作覆盖：
 user.html
请在切换分支前提交或贮藏您的修改。
正在终止
(base) moluo@ubuntu:~/Desktop/api$ git restore .
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 feature/user
尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git restore <文件>..." 丢弃工作区的改动）
 修改：     user.html

修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
(base) moluo@ubuntu:~/Desktop/api$ git checkout master
error: 您对下列文件的本地修改将被检出操作覆盖：
 user.html
请在切换分支前提交或贮藏您的修改。
正在终止
(base) moluo@ubuntu:~/Desktop/api$ git add .
(base) moluo@ubuntu:~/Desktop/api$ git commit -m "fix: update user.html"
[feature/user 82d5570] fix: update user.html
 1 file changed, 1 insertion(+)
(base) moluo@ubuntu:~/Desktop/api$ git checkout master
切换到分支 'master'
(base) moluo@ubuntu:~/Desktop/api$ git branch
  feature/user
* master
(base) moluo@ubuntu:~/Desktop/api$ 
(base) moluo@ubuntu:~/Desktop/api$ git branch debug/index
(base) moluo@ubuntu:~/Desktop/api$ git branch
  debug/index
  feature/user
* master
(base) moluo@ubuntu:~/Desktop/api$ git checkout debug/index
切换到分支 'debug/index'
(base) moluo@ubuntu:~/Desktop/api$ git branch
* debug/index
  feature/user
  master
(base) moluo@ubuntu:~/Desktop/api$ echo "修改了一个bug" >> index.html
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 debug/index
尚未暂存以备提交的变更：
  （使用 "git add <文件>..." 更新要提交的内容）
  （使用 "git restore <文件>..." 丢弃工作区的改动）
 修改：     index.html

修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
(base) moluo@ubuntu:~/Desktop/api$ git add .
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 debug/index
要提交的变更：
  （使用 "git restore --staged <文件>..." 以取消暂存）
 修改：     index.html

(base) moluo@ubuntu:~/Desktop/api$ git commit -m "debug: index.html"
[debug/index 13ed409] debug: index.html
 1 file changed, 1 insertion(+), 1 deletion(-)
(base) moluo@ubuntu:~/Desktop/api$ git status
位于分支 debug/index
无文件要提交，干净的工作区
(base) moluo@ubuntu:~/Desktop/api$ git branch
* debug/index
  feature/user
  master
(base) moluo@ubuntu:~/Desktop/api$ git checkout master
切换到分支 'master'
(base) moluo@ubuntu:~/Desktop/api$ git checkout debug/index
切换到分支 'debug/index'
(base) moluo@ubuntu:~/Desktop/api$ git branch
* debug/index
  feature/user
  master
(base) moluo@ubuntu:~/Desktop/api$ git checkout master
切换到分支 'master'
(base) moluo@ubuntu:~/Desktop/api$ git merge debug/index
更新 54d9f92..13ed409
Fast-forward
 index.html | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
(base) moluo@ubuntu:~/Desktop/api$ git branch
  debug/index
  feature/user
* master
(base) moluo@ubuntu:~/Desktop/api$ git branch -d debug/index
已删除分支 debug/index（曾为 13ed409）。
(base) moluo@ubuntu:~/Desktop/api$ git branch
  feature/user
* master



Thinkpad@LAPTOP-21SUKTQ0 MINGW64 /e/dbhot (master)
$ ls
Demo1.py  demo01.html

Thinkpad@LAPTOP-21SUKTQ0 MINGW64 /e/dbhot (master)
$ ls -al
total 20
drwxr-xr-x 1 Thinkpad 197609    0 Sep 14 13:23 ./
drwxr-xr-x 1 Thinkpad 197609    0 Nov  3 19:28 ../
drwxr-xr-x 1 Thinkpad 197609    0 Nov  4 14:19 .git/
-rw-r--r-- 1 Thinkpad 197609  349 Sep  3 13:45 Demo1.py
-rw-r--r-- 1 Thinkpad 197609 1253 Sep 13 20:16 demo01.html

Thinkpad@LAPTOP-21SUKTQ0 MINGW64 /e/dbhot (master)
$ git config --global user.name 'QiaoPengjun'

Thinkpad@LAPTOP-21SUKTQ0 MINGW64 /e/dbhot (master)
$ git config --global user.email 'qpj99992021@163.com'



```

## 昨日回顾

```python
1 cors和csrf的区别
2 路飞首页
 -页面组件：设置一个路由，当在浏览器访问某个路径，就会显示这个页面组件
 -头部组件
 -轮播图组件
  -页面跳转的两种方式
  -<router-link :to='/home'>点我看美女</router-link>
  -js中写，this.$router.push('/index')
 -尾部组件
3 局部组件的引入和使用
 -需要在要使用该组件的组件中引入
 components: {
    Header, Banner, Footer
  }
4 引入全局css样式
 -写一个css
 -在main.js
  // 配置全局样式
  import './assets/CSS/global.css'
5 轮播图表设计
 -写了一个基表(abstract)
6 轮播图接口
 -自动生成路由
 -获取所有
 -自定义配置
 -配置路由
7 文件放在哪
 -放到项目的media文件夹
 -第三方存储：oss,七牛云存储...
 -自己搭建文件服务器()
  -ceph
  -fastdfs
  -...
8 轮播图跳转问题
9 git
 -公司协同开发使用
 -版本管理：git，SVN
 -三个区
  -工作区：提交到暂存区： git add 指定文件名
  -暂存区：提交到版本库： git commit -m '描述信息'
  -版本库：
  -查看状态：git status
   -红色：工作区变更了，没有提到暂存区
   -绿色：提交到暂存区，没有提交到版本库
   -没有：最新的改变，都提交到版本库了
```

## 今日内容

### 1 git基本操作

```python
-初始化
-提交暂存区
-提交版本库
-设置用户
 -设置全局(在家路径新建一个.gitconfig文件，把用户邮箱写进去)
  >: git config --global user.name '用户名'
  >: git config --global user.email '用户邮箱'
注：在全局文件C:\Users\用户文件夹\.gitconfig新建用户信息，在所有仓库下都可以使用
    C:\Users\Thinkpad\.gitconfig
 -设置局部(在项目路径的.git文件夹下的config文件中加入了用户名和邮箱)
  >: git config user.name '用户名'
  >: git config user.email '用户邮箱'
    
-查看仓库状态
 git status
-撤销工作区操作
 -git checkout .
 -本质是从版本库中拉取最新版覆盖工作区,如果没有被版本管理的文件，不会变化
-撤销暂存区操作
 -git reset .  (了解即可)
-撤销版本库提交
 -git reset --hard 版本号
-查看版本管理日志
 -git log

```

### 2 过滤文件

```python
1 我们项目中有一些文件，文件夹，不要提交到版本库
2 在项目根路径新建：.gitignore  .gitignore.(win平台)
3 在文件中写忽略的文件/文件夹
 -直接写文件夹名字或者/文件夹名字，表示忽略这个文件夹
 -# 表示注释
 -直接写文件名，表示忽略该文件夹
 -* 表示通配符，表示任意数量任意字符  案例：*.log结尾的都忽略
 -? 表示单个字符 ?.log
    
4 咱们的项目
   .idea
    *.log
    #/logs/*
    scripts
```

### 3 多分支开发

```python
1 查看分支： git branch  # 绿色的意思是当前所在分支
2 新建分支： git branch dev 
3 切换分支： git checkout dev
4 删除分支： git branch -D dev  # 应该切换到其他分支，
5 分支合并： git merge 分支名
 git checkout master  # 切换到主分支
 git merge dev      # 把dev分支合并到主分支

```

### 4 远程仓库

```python
-gitee,github,gitlab
-remote 源操作(远程仓库)
 -查看远程仓库：git remote
 -添加远程仓库(远程仓库的名字:origin)：git remote add origin https://gitee.com/liuqingzheng/test.git
-git项目创始者和开发者
-采用ssh协议连接远程源
-提交本地代码到远程仓库


git flow: git工作流，别人设定的一个标准


(base) moluo@ubuntu:~/Desktop/api$ git remote add origin https://gitee.com/pengjunqiao/fuguang36.git
(base) moluo@ubuntu:~/Desktop/api$ git branch
  feature/user
* master
(base) moluo@ubuntu:~/Desktop/api$ git push -u origin master
Username for 'https://gitee.com': PENGJUNQIAO
Password for 'https://PENGJUNQIAO@gitee.com': 
枚举对象中: 12, 完成.
对象计数中: 100% (12/12), 完成.
使用 4 个线程进行压缩
压缩对象中: 100% (6/6), 完成.
写入对象中: 100% (12/12), 1.01 KiB | 516.00 KiB/s, 完成.
总共 12 （差异 1），复用 0 （差异 0）
remote: Powered by GITEE.COM [GNK-6.2]
To https://gitee.com/pengjunqiao/fuguang36.git
 * [new branch]      master -> master
分支 'master' 设置为跟踪来自 'origin' 的远程分支 'master'。

```

### 4.1 远程创建仓库

### 4.2 git项目创始者和开发者

### 项目创始者

```python
1 没有项目纯空的
mkdir test
cd test
git init
touch README.md
git add README.md
git commit -m "first commit"
git remote add origin
https://gitee.com/liuqingzheng/test.git
git push -u origin master

2 项目已经存在，在本地以及操作了(git_test)
cd existing_git_repo
git remote add origin
https://gitee.com/liuqingzheng/test.git
git push -u origin master  # 本地代码推送到远程
#  推送   远程仓库名 主分支
3 提交的时候需要输入gitee的用户名和密码
4 提交成功以后，在远程仓库能看到版本变更记录
```

### 项目开发者

```python
# 因为是开源的所有人都可以克隆
git clone https://gitee.com/pengjunqiao/fuguang36.git
# 修改代码
 git add .
 git commit -m '描述信息'
 git remote 配置远程仓库 (我现在不用陪，同一台机器已经配置过了)
 git push origin master
# 在提交代码之前，要先更新
 git pull orgin master

```

### 4.3 采用ssh协议连接远程源(公司一般用这种)

```python
1 使用https操作的，第一次提交的时候，输入用户名密码
2 企业里通常使用ssh
 -公钥：配置在gitee
 -私钥：留在你本地
3 生成公钥私钥
 -在命令窗口下，位置无所谓执行
 ssh-keygen -t rsa -C "邮箱"
 -会在/C/User/用户名/.ssh生成公钥私钥
 -公钥给别人，私钥不能泄露(非对称加密，对称加密)
4 在gitee中配置，以后本机操作我这个账号，就不用密码验证了
5 远程仓库改成ssh方法
6 git push origin master

```

### 4.4 公司操作流程

```python
1 你到了公司，公司会给你一个gitlab账号和密码 （也可能是你自己注册）
2 登录进去以后，就能看到你能开发的项目
3 git clone 项目地址
4 修改代码  ---> 一顿操作
5 提交到远程 ssh方案 （生成公钥私钥，配置到你账号中）
```

### 5 pycharm配置git

```python
1 
```

![git](E:\老男孩36期Python全栈开发资料\git.png)

![image-20211104231136886](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\image-20211104231136886.png)

![image-20211104231521480](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\image-20211104231521480.png)

![image-20211104232741940](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\image-20211104232741940.png)

![image-20211104232609046](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\image-20211104232609046.png)

![image-20211104232946049](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\image-20211104232946049.png)

### 6 协同开发冲突解决

### 7 线上分支合并

## 作业

错误：提示：更新被拒绝，因为远程仓库包含您本地尚不存在的提交。这通常是因为另外
     提示：一个仓库已向该引用进行了推送。再次推送前，您可能需要先整合远程变更

强制push方法： git push origin +master

## 昨日回顾

```python
1 git的操作之基本操作
 -三个区：工作区，暂存区，版本库
 -git add .
 -git commit -m '注释'
 -工作区改了内容（改的是被版本管理的内容）git checkout .
 -工作区回复到某个版本：git reset --hard 版本号
 -从暂存区拉回工作区：git reset .
 -git status 
 -git log
 -git reflog
2 忽略文件
 -在项目根路径下建立.gitignore
 -在文件中写内容
  log  忽略log文件夹
  /log  忽略根路径下的log
  *  任意长度的字符
  *.log  忽略.log结尾的文件
  /luffyapi/apps
 -咱们写的
  .idea
    *.logs
    #/logs/*
    scripts
    __pycache__

    #luffyapi/apps/*/migrations
    */*/*/migrations/*.py
    !*/*/*/migrations/__init__.py
    
3 分支
 -查：git branch  # *和绿色在哪，表示当前分支
 -增：git branch dev
 -删: git branch -D dev  # 必须不在该分支才能删除
 -切换：git checkout dev
 -合并: git merge dev  # 往我身上合并，不要在dev分支执行
4 pycharm操作git
 -使用pycharm新建，删除，切换分支
 -初始化仓库
 -提交到暂存区
 -提交到版本库
 -建立远程仓库
5 远程仓库
 -查看远程仓库 git remote
 -新增远程仓库 git remote add 远程仓库名字 远程地址
 -把代码提交到远程仓库 git push 远程仓库名字 分支名
 -从远程仓库拉取代码 git pull origin master
6 ssh连接远程仓库
 -非对称加密（对称加密）
 -本地生成公钥私钥（用户家路径生成.ssh文件夹，公钥和私钥）
 -拿着公钥配置在gitee的ssh里面
 -以后再提交使用ssh方式提交（remote地址得改）

```

## 今日内容

### 1 协同开发

```python
1 多人开发同一个项目
2 现在你们git clone 项目， 在你本地了
3 可以改代码了，当你git push 的时候，提交不了
4 项目管理员分配给你权限（gitee中管理 --> 仓库成员 --> 邀请成员）
5 这一个项目有一个管理员，若干开发者
6 重点：不能跨版本提交，只能先拉到最新，再提交  
 git pull origin master
 git lush origin master
7 如果某个开发者在s1.py的第14行加入了东西，我也加入了，他先提交（他没问题）
8 当我再提交，会出冲突
 <<<<<< HEAD
 你的代码
 ======
 同事的代码
 >>>>>> ddhaohfogahojfgoahofjoahf9
    
9 解决冲突
 -留你的代码：你写的好
 -留他的代码：你觉的他写的好
 -你们的都留着：你俩虽然改了一行，但是功能不一样
```

### 1.1 协同开发冲突解决

```python
如果同一行内容不一样有可能会合并起来（正常冲突会报错）
```

### 2 分支合并

```python
1 版本库有两套
 -本地一套
  -本地有master.dev分支
 -远程一套
  -远程现在只有master分支
2 把本地dev提交到远端
 git push origin dev
 现在远端有两个分支 master和dev
3 删除本地分支，删除远程分支(点点点)
4 直接在远程创建分支(点点点)
5 把远端的dev拉到本地
```

### 2.1 线上分支合并

```python

```

![image-20211105210945279](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\image-20211105210945279.png)

![image-20211105212251062](C:\Users\Thinkpad\AppData\Roaming\Typora\typora-user-images\image-20211105212251062.png)

### 2.2 线下分支合并提交

```python
1 本地dev分支新增代码，提交到版本库
2 本地dev分支合并到master
 -切换到master
 -git merge dev  (没有冲突直接合并)
 -git push origin master (本地master提交到远端master)
3 切换到dev分支，再提交到远端
4 到此，本地分支合并完成，远端的dev跟master完全一样，本地的dev跟master也完全一样
```

### 2.3 合并分支冲突解决

```python
1 在master增加一行，提交到版本库
2 在dev同样位置增加一行，提交到版本库
3 把dev合并到master，就会出冲突
 -切换到master
 -git merge dev  合并 有冲突
 -解决冲突（到底留多少，你决定）
 -提交到远端
```

### 3 远程仓库回滚

```python
1 远程仓库，回到最初路飞第一次提交的地方
2 在本地回复到第一次提交
 git reset --hard 59dfagsf
 git push origin master -f  # 强制提交
3 切记 -f 清醒的时候使用
```

### 4 其他了解

```python
git flow
 -是一种建分支的方案
 -预览分支，开发分支，bug分支，主分支，测试分支
pull 和 fetch
 git pull: 拉代码+合并
 git fetch: 拉代码，需要手动合并进去
变基 rebase
 -合并分支的时候，是否保留之前分支的日志
   
git客户端：
 -官方下载的一个（命令行）
 -pycharm编辑器
 -sourcetree: 美观，分支通过不同颜色显示，看着好看
```

### 5 登录注册页面分析

```python
1 多方式登录接口
2 校验手机号是否存在接口
3 手机号，密码，验证码注册接口
4 短信登录接口
5 发送短信接口

```

### 6 前端登录注册编写

```python
1 注册
<template>
  <div class="register">
    <div class="box">
      <i class="el-icon-close" @click="close_register"></i>
      <div class="content">
        <div class="nav">
          <span class="active">新用户注册</span>
        </div>
        <el-form>
          <el-input
            placeholder="手机号"
            prefix-icon="el-icon-phone-outline"
            v-model="mobile"
            clearable
            @blur="check_mobile">
          </el-input>
          <el-input
            placeholder="密码"
            prefix-icon="el-icon-key"
            v-model="password"
            clearable
            show-password>
          </el-input>
          <el-input
            placeholder="验证码"
            prefix-icon="el-icon-chat-line-round"
            v-model="sms"
            clearable>
            <template slot="append">
              <span class="sms" @click="send_sms">{{ sms_interval }}</span>
            </template>
          </el-input>
          <el-button type="primary">注册</el-button>
        </el-form>
        <div class="foot">
          <span @click="go_login">立即登录</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Register',
  data() {
    return {
      mobile: '',
      password: '',
      sms: '',
      sms_interval: '获取验证码',
      is_send: false
    }
  },
  methods: {
    close_register () {
      this.$emit('close', false)
    },
    go_login () {
      this.$emit('go')
    },
    check_mobile () {
      if (!this.mobile) return
      if (!this.mobile.match(/^1[3-9][0-9]{9}$/)) {
        this.$message({
          message: '手机号有误',
          type: 'warning',
          duration: 1000,
          onClose: () => {
            this.mobile = ''
          }
        })
        return false
      }
      this.is_send = true
    },
    send_sms () {
      if (!this.is_send) return
      this.is_send = false
      let SmsIntervalTime = 60
      this.sms_interval = '发送中...'
      const timer = setInterval(() => {
        if (SmsIntervalTime <= 1) {
          clearInterval(timer)
          this.sms_interval = '获取验证码'
          this.is_send = true // 重新回复点击发送功能的条件
        } else {
          SmsIntervalTime -= 1
          this.sms_interval = `${SmsIntervalTime}秒后再发`
        }
      }, 1000)
    }
  }
}
</script>

<style scoped>
.register {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 10;
  background-color: rgba(0, 0, 0, 0.3);
}

.box {
  width: 400px;
  height: 480px;
  background-color: white;
  border-radius: 10px;
  position: relative;
  top: calc(50vh - 240px);
  left: calc(50vw - 200px);
}

.el-icon-close {
  position: absolute;
  font-weight: bold;
  font-size: 20px;
  top: 10px;
  right: 10px;
  cursor: pointer;
}

.el-icon-close:hover {
  color: darkred;
}

.content {
  position: absolute;
  top: 40px;
  width: 280px;
  left: 60px;
}

.nav {
  font-size: 20px;
  height: 38px;
  border-bottom: 2px solid darkgrey;
}

.nav > span {
  margin-left: 90px;
  color: darkgrey;
  user-select: none;
  cursor: pointer;
  padding-bottom: 10px;
  border-bottom: 2px solid darkgrey;
}

.nav > span.active {
  color: black;
  border-bottom: 3px solid black;
  padding-bottom: 9px;
}

.el-input, .el-button {
  margin-top: 40px;
}

.el-button {
  width: 100%;
  font-size: 18px;
}

.foot > span {
  float: right;
  margin-top: 20px;
  color: orange;
  cursor: pointer;
}

.sms {
  color: orange;
  cursor: pointer;
  display: inline-block;
  width: 70px;
  text-align: center;
  user-select: none;
}
</style>


2 登录
<template>
  <div class="login">
    <div class="box">
      <i class="el-icon-close" @click="close_login"></i>
      <div class="content">
        <div class="nav">
                    <span :class="{active: login_method === 'is_pwd'}"
                          @click="change_login_method('is_pwd')">密码登录</span>
          <span :class="{active: login_method === 'is_sms'}"
                @click="change_login_method('is_sms')">短信登录</span>
        </div>
        <el-form v-if="login_method === 'is_pwd'">
          <el-input
            placeholder="用户名/手机号/邮箱"
            prefix-icon="el-icon-user"
            v-model="username"
            clearable>
          </el-input>
          <el-input
            placeholder="密码"
            prefix-icon="el-icon-key"
            v-model="password"
            clearable
            show-password>
          </el-input>
          <el-button type="primary">登录</el-button>
        </el-form>
        <el-form v-if="login_method === 'is_sms'">
          <el-input
            placeholder="手机号"
            prefix-icon="el-icon-phone-outline"
            v-model="mobile"
            clearable
            @blur="check_mobile">
          </el-input>
          <el-input
            placeholder="验证码"
            prefix-icon="el-icon-chat-line-round"
            v-model="sms"
            clearable>
            <template slot="append">
              <span class="sms" @click="send_sms">{{ sms_interval }}</span>
            </template>
          </el-input>
          <el-button type="primary">登录</el-button>
        </el-form>
        <div class="foot">
          <span @click="go_register">立即注册</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Login',
  data () {
    return {
      username: '',
      password: '',
      mobile: '',
      sms: '',
      login_method: 'is_pwd',
      sms_interval: '获取验证码',
      is_send: false
    }
  },
  methods: {
    close_login () {
      this.$emit('close')
    },
    go_register () {
      this.$emit('go')
    },
    change_login_method (method) {
      this.login_method = method
    },
    check_mobile () {
      if (!this.mobile) return
      if (!this.mobile.match(/^1[3-9][0-9]{9}$/)) {
        this.$message({
          message: '手机号有误',
          type: 'warning',
          duration: 1000,
          onClose: () => {
            this.mobile = ''
          }
        })
        return false
      }
      this.is_send = true
    },
    send_sms () {
      if (!this.is_send) return
      this.is_send = false
      let SmsIntervalTime = 60
      this.sms_interval = '发送中...'
      const timer = setInterval(() => {
        if (SmsIntervalTime <= 1) {
          clearInterval(timer)
          this.sms_interval = '获取验证码'
          this.is_send = true // 重新回复点击发送功能的条件
        } else {
          SmsIntervalTime -= 1
          this.sms_interval = `${SmsIntervalTime}秒后再发`
        }
      }, 1000)
    }
  }
}
</script>

<style scoped>
.login {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 10;
  background-color: rgba(0, 0, 0, 0.3);
}

.box {
  width: 400px;
  height: 420px;
  background-color: white;
  border-radius: 10px;
  position: relative;
  top: calc(50vh - 210px);
  left: calc(50vw - 200px);
}

.el-icon-close {
  position: absolute;
  font-weight: bold;
  font-size: 20px;
  top: 10px;
  right: 10px;
  cursor: pointer;
}

.el-icon-close:hover {
  color: darkred;
}

.content {
  position: absolute;
  top: 40px;
  width: 280px;
  left: 60px;
}

.nav {
  font-size: 20px;
  height: 38px;
  border-bottom: 2px solid darkgrey;
}

.nav > span {
  margin: 0 20px 0 35px;
  color: darkgrey;
  user-select: none;
  cursor: pointer;
  padding-bottom: 10px;
  border-bottom: 2px solid darkgrey;
}

.nav > span.active {
  color: black;
  border-bottom: 3px solid black;
  padding-bottom: 9px;
}

.el-input, .el-button {
  margin-top: 40px;
}

.el-button {
  width: 100%;
  font-size: 18px;
}

.foot > span {
  float: right;
  margin-top: 20px;
  color: orange;
  cursor: pointer;
}

.sms {
  color: orange;
  cursor: pointer;
  display: inline-block;
  width: 70px;
  text-align: center;
  user-select: none;
}
</style>

```

## 作业

```python
1 跟同桌协同开发，制造冲突，解决冲突
2 本地远程新建分支，本地分支合并制造冲突，解决冲突
3 编写2个接口
 多方式登录接口
 手机号存在与否接口
拓展：
 -注册腾讯云，申请短信接口（注册一个公众号）
```

## 上节回顾

```python
1 版本管理git,svn
2 客户端 ---> 远程仓库(gitee,github,gitlab)
3 基本命令
 -git status
 -git log
 -git reflog
4 忽略文件
 -.idea
 -文件夹名  a文件夹  # 忽略掉任何路径下的a文件夹，空文件夹识别不到
 -/文件夹/*.py
 -*
 -?
 -!
5 分支
 -master：主分支
 -dev: 开发分支
 -bug: 改bug ---> 合并到主分支
6 远程仓库
 -git remote
 -git remote add 名字  地址(https、ssh)
 -git push 远程仓库名字 本地分支名  (认证通过)
 -git pull 远程仓库  分支名  (提代码之前一定要先拉)
  -没事就拉一下(保持本地最新的)
7 git clone  远程地址
8 ssh和https
9 协同开发
 -冲突（留谁的代码）
10 远程分支合并
11 本地分支合并完提交到远端
12 远程仓库的版本回滚(慎用-f)
13 其他
 git flow  git 工作流
 git fetch和git pull 是fetch+合并
  git rebase  变基
14 登录注册相关接口
 -多方式登录
 -验证手机号是否存在
 -发送验证码
 -验证码登录
 -注册
```

## 今日内容

### 1 vue后端地址配置

```python
1 在vue项目的assets/js/settings.js
export default {
  BASE_URL: 'http://127.0.0.1:8000/'
}
2 在main.js中导入
 import settings from './assets/js/settings'
 Vue.prototype.$BASE_URL = settings.BASE_URL
3 在组件中直接使用
 this.$
```

### 2 media_url的作用

```python
1 只要是数据库中文件，图片，配置这个
 media_url = '/media/'
2 序列化字段（文件，图片）
 -当前域+/media/+数据库中存的路径
```

### 2 vue前端登录

```python
1 vue前端存储
 -放到cookie中，js来手动放进去（vue-cookie），过期就会消失
 -LocalStorage: 本地存储，关闭浏览器也不会消失
 -SessionStorage：会话存储，关闭浏览器，就消失了
```

### 3 前端登录成功显示用户名，注销

```python
  // 密码登录方法
    pwd_login () {
      if (!(this.username && this.password)) {
        // 用户名或者密码为空
        this.$message({
          message: '用户名或密码不能为空',
          type: 'warning'
        })
      } else {
        // 发送axios请求
        this.$http.post(this.$BASE_URL + 'user/login/', {
          username: this.username,
          password: this.password
        }).then(res => {
          console.log(res)
          if (res.data.code === 100) {
            // 登录成功

            // 2 存储返回的token，username (可以存的地方有三个)
            // 咱们放到cookie中
            this.$cookies.set('token', res.data.token, '7d')
            this.$cookies.set('username', res.data.username, '7d')
            // sessionStorage.setItem()
            // sessionStorage.getItem()
            // localStorage.setItem()
            // localStorage.getItem()
            // 1 销毁框
            this.close_login()
          } else {
            this.$message({
              message: res.data.msg,
              type: 'warning'
            })
          }
        })
      }
    },


logout () {
      this.$cookies.remove('token')
      this.$cookies.remove('username')
      this.token = ''
      this.username = ''
      this.$message({
        message: '注销成功',
        type: 'success'
      })
    }
```

### 4 手机号是否存在接口

### 5 API和SDK的区别

### 6  短信验证码封装

```python
1 注册公众号
2 在腾讯云的短信模板下创建国内短信
3 签名管理（等待审核）
4 模板管理（等待审核）
5 应用管理（创建一个应用）
 SDK AppID ：1400595474
 App Key ： 3f56c55b9845529533a15345bbb79c4a
6 使用：去官方文档扣代码
 -api: 比较繁琐
 -sdk: 
  -3.0: 更强大，不仅仅有发短信的代码，还有很多其他的
  -2.0：只针对于发短信
```

### 7 发送短信接口

## 作业

GIS  图数据库

lua

sudo gedit /etc/resolv.conf

roullup.js

sudo service redis-server restart

02DN3yBANi1xeXI-BQitJug**

CaptchaAppId:2071744404

 AppSecretKey:02DN3yBANi1xeXI-BQitJug**

qpj701762 微信公众号

容联云 qpj12345

ACCOUNT SID：8aaf07087ce03b67017d141ad4380ba6

AUTH TOKEN：b3f15b8095b34862b4e9f82c234ae477

AppID(默认)：*8aaf07087ce03b67017d141ad5610bad*

Rest URL(生产)：

<https://app.cloopen.com:8883>

## 昨日回顾

```python
1 前端配置请求后端的地址
2 media的配置
 -media_url
3 前端登录功能
 -判断用户名密码是否为空，message的提示
 -发送axios请求(post)
4 前端存储（三个位置）
5 前端用户名的显示与不显示
6 发送短信（腾讯云）
 -api和SDK
 -发送短信的2.0的SDK
 -封装了一层（单独做成了一个模块，以后可以顺利的移到任何其他项目中）
7 校验手机号是否存在的接口
8 限制短信发送频率的频率类
```

## 今日内容

### 1 手机号登录接口

```python
1 只需要手机号和验证码
class UserPhoneModelSerializer(serializers.ModelSerializer):
    # 必须重写一下code字段
    code = serializers.CharField()
    mobile = serializers.CharField(min_length=11)
    class Meta:
        model = models.User
        fields = ['mobile', 'code']

    # 全局钩子（校验手机号和验证码是否正确，登录成功，签发token）
    def validate(self, attrs):
        # 取出用户名密码
        mobile = attrs.get('mobile')
        code = attrs.get('code')
        # 从缓存中取出这个手机号对应的验证码
        cache_code = cache.get(settings.SMS_CACHE_PHONE % mobile)
        if cache_code and cache_code==code:
            # 可以登录，根据手机号，查到用户，给这个用户签发token
            user = models.User.objects.get(mobile=mobile)
            # 签发token
            payload = jwt_payload_handler(user)
            token = jwt_encode_handler(payload)
            self.context['token'] = token
            # 将登录用户对象直接传给视图
            self.context['user'] = user
            return attrs
        else:
            raise ValidationError("验证码错误")
```

### 2 前端获取验证码，手机号登录

### 3 后端注册功能

```python
1 前端传入是不带用户名的，我们可以自动生成一个用户名，或者使用手机号作为用户名
2 邮箱也没传入（邮箱可以为空）
3 前端传入
 -手机号
 -密码
 -code（表中没有）
4 post请求

class UserRegisterModelSerializer(serializers.ModelSerializer):
    # 必须重写一下code字段
    code = serializers.CharField(write_only=True)

    class Meta:
        model = models.User
        fields = ['mobile', 'code', 'password']

        # code这个字段只用来写
        # extra_kwargs = {
        #     'code': {'write_only': True}
        # }

    # 可以给手机号和password加局部校验钩子
    def validated_mobile(self, data):
        # 手机号
        if re.match('^1[3-9][0-9]$', data):
           return data
        else:
            raise ValidationError("手机号不合法！")


    # 全局钩子
    def validate(self, attrs):
        # 取出用户名密码
        mobile = attrs.get('mobile')
        code = attrs.get('code')
        # 从缓存中取出这个手机号对应的验证码
        cache_code = cache.get(settings.SMS_CACHE_PHONE % mobile)
        if cache_code and cache_code == code:
            # 可以注册
            # 如何注册？ 需要重写create，由于密码是密文，需要重写，使用create_user来创建用户
            # 把code剔除
            attrs.pop('code')
            # 加入username
            attrs['username'] = mobile

            return attrs
        else:
            raise ValidationError("验证码错误")

    def create(self, validated_data):
        # validated_data: username,password,mobile (email,icon都可以为空)
        user = models.User.objects.create_user(**validated_data)
        return user
```

### 4 前端注册功能

```python

```

### 5 redis介绍安装

```python
1 redis数据库，非关系型 
 redis：内存数据库 所有数据放在内存中 缓存 
 MongoDB：数据放在硬盘上，json格式数据 
 es：放在硬盘上 大数据量检索
2 关系型：mysql, db2, oracle, posgresql, sqlserver, 达梦（国产数据） sql都是通用的 表和表之间的关联关系 外键 事务的支持比较好 锁机制
3 redis是一个key-value存储系统。
 -key-value存储
 -支持5大数据类型：字符串，列表，字典（hash），集合，有序集合
 -6.0之前单线程，单进程的
  -redis是单线程为什么这么快？
   -全内存操作
   -基于IO多路复用的模型
   -没有多线程多进程的切换
  -性能高，单机qps高达10w,实际生产环境实际6W
  -可以持久化
  -C语言写的
和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步
4 Memcached: 内存数据库，key-value存储，支持的数据类型少，只有str类型，没有持久化，断电消失
5 redis不支持Windows io多路复用的模型，在Linux使用的是epoll,在Windows上没有epoll,于是有一些第三方基于源码改动了一些，让它可以在Windows上执行（最新版只到3.x）
6 Windows下载地址
 https://github.com/microsoftarchive/redis
 https://www.cnblogs.com/liuqingzheng/p/9831331.html
7 Windows安装
 -一路下一步：添加环境变量，端口号是6379，使用的最大内存，不限制
默认端口6379：6379在是手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字。MERZ长期以来被antirez及其朋友当作愚蠢的代名词。
8 装完之后，就会有个redis服务
 -通过启动服务，停止服务来运行停止redis
9 命令方式启动和停止
 -启动
  redis-server 配置文件路径 # 使用配置文件启动
  redis-server         # 不使用配置文件启动
 -客户端连接
  redis-cli           # 连接上本地 127.0.0.1 6379
  redis-cli -h 127.0.0.1 -p 6378
  在客户端：shutdown
  在客户端： ping 回一个 pone
10 redis的客户端很多
 -redis-desktop-manager
  -qt: 平台，专门用来写图形化界面
  -pyqt：用python代码在qt平台写图形化界面
11 在客户端写
 set name lqz
 get name
```

### 6 python操作redis之普通连接和连接池

```python
import redis
# 先定义一个池子，池子大小指定
# 这个pool必须做成一个单例（这句话只写一次）
POOL = redis.ConnectionPool(max_connections=10)


import redis
from redis import Redis
# redis之普通连接
# conn = Redis(host='127.0.0.1', port=6379)  # 什么都不写，连的就是本地的6379
#
# conn.set(name='name', value='qpj')
# # 获取name这个key对应的值
# res = conn.get('name')
# print(res)
# conn.close()

# redis连接之连接池
# 当以脚本形式执行不能用相对导入  包内可以
# 当该py文件以脚本运行，其中包的导入不能使用相对导入
# 如果是以模块形式导入使用，这个模块内可以使用相对导入
from redis_pool import POOL  # 天然的单例，不管导入多少次，只会实例化一次
# 建立连接，从池子中获取一个连接
conn = redis.Redis(connection_pool=POOL)
res = conn.get('name')
print(res)
```

### 7 字符串操作

```python
set(name, value, ex=None, px=None, nx=False, xx=False)
在Redis中设置值，默认，不存在则创建，存在则修改
参数：
     ex，过期时间（秒）
     px，过期时间（毫秒）
     nx，如果设置为True，则只有name不存在时，当前set操作才执行,值存在，就修改不了，执行没效果
     xx，如果设置为True，则只有name存在时，当前set操作才执行，值存在才能修改，值不存在，不会设置新值
        
        
# -*- coding = utf-8 -*-
# @Time: 2021/11/18 23:10
# @Author: Thinkpad
# @File: reids_连接方式.py
# @Software: PyCharm
from redis import Redis
# redis之普通连接
conn = Redis(host='127.0.0.1', port=6379)  # 什么都不写，连的就是本地的6379
# 1 设置值
# conn.set('age', 16)
# conn.set(name='name', value='qpj')
# conn.set('name', '刘清政')
# conn.set('hobby', '篮球', ex=3)
# conn.set('hobby', '篮球', ex=3, xx=True)
# conn.setnx()
# conn.setex()
# conn.psetex('xx',5000, 'yy')
# mset 批量设置
# conn.mset({'name': 'qpj', 'age': 11})
# 2 取值
# res = conn.get('name')
# print(conn.get('age'))  # 取出来的值是bytes格式
# res = conn.get('name').decode('utf-8')
# print(res)
# print(str(conn.get('name'), encoding='utf-8'))
# print(conn.mget(['name', 'age']))
# print(conn.mget('name', 'age'))
# res = conn.getset('name', 'egon')
# gbk: 2个字节表示一个字符
# utf-8: 3个字节表示一个字符
# res = conn.getrange('name', 0, 3)
# res = conn.setrange('name', 0, 'p')
# res = conn.strlen('name')
# res = conn.incr('age')
# res = conn.decr('age')
res = conn.append('name', 'nb')
print(res)
conn.close()
```

### 8 列表操作

### 9 hash操作

### 10 其他操作

### 11 管道

### 12 django中使用redis

## 作业

JS 变量提升

f"{random.randint(0, 999999):06d}"

"%06d" % random.randint(0, 999999)

celery处理任务的框架 调度任务队列

```bash
(base) moluo@ubuntu:~/Desktop/fuguang/fuguangapi$ conda activate fuguang
(fuguang) moluo@ubuntu:~/Desktop/fuguang/fuguangapi$ python manage.py shell
Python 3.8.12 (default, Oct 12 2021, 13:49:34) 
[GCC 7.5.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from mycelery.sms.tasks import send_sms
Traceback (most recent call last):
  File "<console>", line 1, in <module>
ImportError: cannot import name 'send_sms' from 'mycelery.sms.tasks' (/home/moluo/Desktop/fuguang/fuguangapi/mycelery/sms/tasks.py)
>>> from mycelery.sms.tasks import send_sms
Traceback (most recent call last):
  File "<console>", line 1, in <module>
ImportError: cannot import name 'send_sms' from 'mycelery.sms.tasks' (/home/moluo/Desktop/fuguang/fuguangapi/mycelery/sms/tasks.py)
>>> from mycelery.sms.tasks import send_sms1
>>> send_sms1.delay()
<AsyncResult: d41e950a-b7ce-4900-b05e-607b98d04630>
>>> send_sms1.delay()
<AsyncResult: 56b03297-453d-43fe-a3b1-4dc2d9936cd6>
>>> ret = send_sms1.delay()
>>> ret
<AsyncResult: b00b6009-5e85-4733-81b6-892349d0c869>
>>> ret.id
'b00b6009-5e85-4733-81b6-892349d0c869'
>>> ret.get()
>>> ret.status
'SUCCESS'

>>> from mycelery.sms.tasks import send_sms2
>>> mobile = "15709282071"
>>> code = "666666"
>>> ret2 = send_sms2.delay(mobile,code)
>>> ret2
<AsyncResult: 5e6bd4b9-ee55-4402-87cd-b9ca61d9700c>
>>> from mycelery.sms.tasks import send_sms4
>>> ret = send_sms4.delay(0,4)
>>> ret
<AsyncResult: 466a9425-8d25-4848-9c11-977d80a37efc>
>>> ret.get()
4

ps aux| grep celery

```

一个表就是一个实体，存储在数据库中的事务

范式理论

逆范式

3秒原则，8秒原则

看代码  代码走向 代码业务  代码解决了什么业务

新功能 需求文档 E-R图 画图 接口文档

## 昨日回顾

```python
1 redis的连接池
 -在创建连接池的时候，就要指定连接地址
 -以后每次使用都是从连接池中拿一个连接
2 编码问题，实例化得到redis连接对象（Redis类），可以指定编码
3 http，应用层协议，基于tcp封装
 -ssh,http,ftp,websocket DNS ---> 应用层协议
 -tcp/ip:socket,抽象层
 -UDP: DNS
4 mysql，redis：cs架构的软件
 -客户端和服务端通信 ---> 通过网络 ---> 基于tcp自封装协议
 -客户端和服务端通信，必须遵循这个协议
 -es, docker: http协议，符合resful规范
5 手机号登录接口
6 手机号注册功能
 -前端把手机号，密码，验证码传入 --> 创建用户
7 前端的手机号登录，发送验证码，手机号注册
8 redis：非关系型数据库，缓存数据库，C，性能很高，5大数据类型
 -CS架构的，
  -启动服务端 redis-server 配置文件
  -客户端连接 redis-cli -h -p
  -windows的RedisDesktopManager客户端连接
9 python连接redis：普通连接，连接池
10 redis的字符串操作


11 python代码 是解释型语言，必须运行在解释器之上
 pyinstaller ---> 可以把这个py文件打包成exe
 打成的exe特别大，不需要解释器了
 -go 编译型，写的代码 --> 编译成exe
 -java: 是编译型 ---> 字节码文件
  -jdk: java开发工具包，包含了jre,jvm
  -jre: java运行环境
  -jvm: java虚拟机，至少占200M内存
   -阿里有自己的jvm
        
```

## 今日内容

### 1 redis的hash操作

```python
from redis import Redis
conn = Redis()

# conn.hset('hash_test', 'name', 'lqz')
# conn.hmset('hash_test1', {'name':'egon', 'age': 18})
# conn.hset('hash_test2', mapping={'name':'egon', 'age': 18})
# res = conn.hget('hash_test', 'name')
# res = conn.hmget('hash_test1', ['name', 'age'])
# res = conn.hgetall('hash_test1')  # 尽量少执行，慢长命令，可能会撑爆应用程序的内存
# res = conn.hlen("hash_test1")
# res = conn.hkeys("hash_test1")
# res = conn.hvals("hash_test1")
# res = conn.hexists("hash_test1", 'name')
# res = conn.hdel("hash_test1", 'name')
# res = conn.hincrby("hash_test1", 'age', amount=10)
# res = conn.hincrbyfloat("hash_test1", 'age', amount=1.23)  # 存在精度损失问题
# for i in range(1000):
#     conn.hset('test', "%s-%s" % ('key', i), i)
# res = conn.hscan("test", cursor=0, count=100)
# print(res)
# print(len(res[1]))  # 101

# 生成器用的位置
for item in conn.hscan_iter("test", count=10):  # 取出全部的value值，但是每次取10个
    print(item)
# 生成器用在哪 每次取n条，用完再取

conn.close()
```

### 2 redis的列表操作

```python
from redis import Redis
conn = Redis()

# conn.lpush('list_test', 'egon')
# conn.lpush('list_test', 'qpj')
# conn.rpush('list_test', 'lyf')
# conn.lpushx('list_test', 'egon')
# conn.rpushx('list_test', 'xjp')
# res = conn.llen('list_test')
# print(res)
# conn.linsert('list_test', 'before', 'qpj', 'dlr')
# conn.linsert('list_test', 'after', 'qpj', 'json')
# conn.lset('list_test', 0, 'xxx')
# conn.lrem('list_test', count=1, value='egon')
# res = conn.lpop('list_test')
# res = conn.rpop('list_test')
# res = conn.lindex('list_test', 0)
# res = conn.lrange('list_test', 0, 2)
# res = conn.ltrim('list_test', 0, 1)  # 保留0-1之间的
# print(res)
# rpoplpush
# blpop
# res = conn.blpop('list_test')
res = conn.brpop('list_test')
print(res)

conn.close()
```

### 3 其他操作

```python
from redis import Redis
conn = Redis()

# 公共操作，跟类型无关
# res = conn.delete('name', 'list_test')
# res = conn.exists('name')
# res = conn.keys()
# res = conn.keys('h?sh_test')
# res = conn.keys('h?sh_*')
# res = conn.keys('h[ae]sh_*')
# res = conn.expire('hash_test', 5)
# res = conn.rename('hash_test1', 'hash_test')
# res = conn.move('hash_test', db=1)
# conn.set('name', 'qpj')
# res = conn.randomkey()
res = conn.type(name='name')

print(res)

conn.close()
```

### 4 管道

```python
import redis

# redis 非关系型数据库，本身不支持事务
# redis中的管道可以实现事务的支持（要么都成功，要么都失败）
# 管道原理： 多条命令放到一个管道中，一次性执行
# 如果是集群环境，不支持管道
pool = redis.ConnectionPool()
r = redis.Redis(connection_pool=pool)
# 开启事务
pipe = r.pipeline(transaction=True)
# 管道等待放入多条命令
pipe.multi()
pipe.set('name', 'egon')
pipe.set('role', 'admin')
# 到此命令都没有执行
# 执行管道中的所有命令
pipe.execute()

```

### 5 django中使用redis

#### 方式一（通用方式，任何框架都可以用）

```python
# 写一个pool
from redis import ConnectionPool
POOL = ConnectionPool(max_connections=5, host='127.0.0.1', port=6379)
# 在使用的位置，获取连接，获取数据
from luffyapi.libs.redis_pool import POOL
from redis import Redis
def test(request):
    conn = Redis(connection_pool=POOL)
    name = conn.get('name')
    return HttpResponse('name是: %s' % name)
```

#### 第二种：django框架专用

```python
1 pip install django-redis
2 在配置文件中配置
# redis配置
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
            "CONNECTION_POOL_KWARGS": {"max_connections": 100}
            # "PASSWORD": "123",
        }
    }
}

3 在使用的位置
from django_redis import get_redis_connection

def test(request):
    # 从连接池中取一个连接
    conn = get_redis_connection()
    name = conn.get('name')
    return HttpResponse('name的value是: %s' % name)

# 这个号，可以缓存任意对象
from django.core.cache import cache
cache.set()
cache.get()
4 以后django中都用这种，好处是django中的缓存也会放到redis中
```

### 6 接口缓存

```python
# 以首页轮播图为例，加入缓存，后期所有接口都需要缓存  重写一个ListAPIView加一个参数
# 用户来了，先去缓存中看，如果有直接返回，如果没有，去mysql中看，查完以后再缓存中放一份
# 双写一致性：处理方案有三种
 -只要插入数据，就删除缓存
    
class BannerView(ViewSetMixin, ListAPIView):
    queryset = Banner.objects.all().filter(is_show=True, is_delete=False).order_by('-orders')[0:settings.BANNER_SIZE]
    serializer_class = BannerSerializer

    # 重写list方法 接口缓存
    def list(self, request, *args, **kwargs):
        banner_list = cache.get('banner_cache_list')
        # 如果没有值，去数据库查
        if not banner_list:
            response = super().list(request, *args, **kwargs)
            # 在缓存中放一份
            cache.set('banner_cache_list', response.data)
        else:
            # 走了缓存，速度很快
            print('走了缓存')
            response = Response(data=banner_list)

        return response
```

### 7 celery介绍

```python
1 celery: 芹菜，分布式的异步任务框架
2 可以做的事，我们用它来解决什么问题
 - 异步任务 （异步的执行这个任务函数） 发送短信、邮件 消息推送 音视频处理
 - 延迟任务（延迟5S钟，执行一个任务（函数））
 - 定时任务 （定时的执行任务（函数）） 如果单纯执行定时任务，没必要用celery，可以使用别的
3 平台问题不支持Windows
4 可以不依赖任务服务器，通过自身命令，启动服务（内部支持socket）
celery服务为其他项目服务提供异步任务解决需求

消息中间件 message broker
任务执行单元 worker
任务结果存储仓库 task result store
```

### 8 简单使用

```python
1 pip install celery
2 两种目录组织结构
 -普通的
 -包管理的（推荐的）
3 普通的（就一个py文件）

celery -A proj worker -l INFO
celery -A celery_task worker -l info -P eventlet
4 基本使用
# 第一步： 定义一个py文件（名字随意，我们叫celery_task)
from celery import Celery
backend = 'redis://127.0.0.1:6379/1'
# 消息中间件
broker = 'redis://127.0.0.1:6379/2'
app = Celery(main=__name__, backend=backend, broker=broker)
# 被它修饰，就变成了celery的任务
@app.task
def add(a, b):
    return a + b
# 第二版：提交任务（新建一个py文件：submit_task)
from celery_task import add
# 同步调用
# res = add(3, 4)
# print(res)
# 异步调用
# 提交任务到redis 未执行 返回一个唯一标识 后期使用唯一标识查看任务结果
res = add.delay(35, 41)
print(res)  # cd321656-9ce3-4383-a687-88e16c4b2816
# 使用任务执行单元来执行任务（使用命令启动）
"""
非Windows
命令 celery worker -A celery_task -l info
windows
pip install eventlet
# 要去咱们写的celery文件所在目录下执行 E:\python\luffy\luffyapi\luffyapi\scripts\celery_base>
# 5.0 之前的版本
celery worker -A celery_task -l info -P eventlet
# 5.0 之后的版本
celery -A celery_task worker  -l info -P eventlet
"""
# 第三步：启动worker
# celery_task py文件的名字
# -l info 日志输出级别是info
# -P eventlet 在win平台需要加上
celery -A celery_task worker  -l info -P eventlet
# 如果队列里有任务，就会执行，如果没有任务，worker就等在这

# 第四步：查询结果是否执行完成
from celery_task import app
from celery.result import AsyncResult
id = 'ba2b3322-6d2f-48fd-8fc9-63e043b56fb4'
if __name__ == '__main__':
    a = AsyncResult(id=id, app=app)
    if a.successful():
        result = a.get()
        print(result)
    elif a.failed():
        print('任务失败')
    elif a.status == 'PENDING':
        print('任务等待中被执行')
    elif a.status == 'RETRY':
        print('任务异常后正在重试')
    elif a.status == 'STARTED':
        print('任务已经开始被执行')



```

## 作业

23种设计模式

架构师

## 昨日回顾

```python
1 http基于tcp，一定要进行3次握手，4次挥手
 -http1.1版本，keepalive,可以多次请求使用同一个tcp连接
2 redis的hash操作
 -hset
 -hget
3 redis的列表操作
 -lpush
 -lpop
 -blpop  # 消息队列
 -rpush
4 通用操作
 -type
 -expire
5 管道
 -redis非关系型数据库，不支持事务，可以通过管道模拟事务
6 django中使用redis
 -普通使用（通用）
 -django中独有的（django-redis）
  -配置文件中配置cache
  -使用的时候：conn=get_redis_connection()
  -django的缓存使用的都是redis（不会在内存中放了）（默认不支持放在redis中）
   -cache.set()   
  -cache.get()
7 接口缓存
 -提高接口的并发量
8 celery 分布式的异步任务框架
 -异步任务
 -定时任务
 -延迟任务
9 使用
 - 建一个包 celery_task
 - 在包中必须写一个celery.py
 - app=实例化（消息中间件地址，结果存储地址，管理的task的路径）
 - 写task
  -在包下新建py文件，包app导入，用app.task装饰函数（函数就是任务）
 -在想用的位置（独立的，在包外部，可能是django项目），导入
  -函数名.delay(参数) ---> 提交任务
 -启动worker
  -celery -A 包名 worker -l info
 -查询执行结果，使用id查
```

## 今日内容

### 1 celery执行延迟任务

```python
1 其他不用变，提交任务的时候
# 提交延迟任务
from celery_task.user_task import mul
from datetime import datetime, timedelta

eta = datetime.utcnow() + timedelta(seconds=10)
# UTC时间 时间类型支持直接相加
# print(datetime.utcnow())
# 提交延迟任务
# args: nul任务的参数
# eta: 延迟时间（延迟多长时间执行），必须传一个时间对象（写数字不行）
# 使用UTC时间
# 10秒后执行这个任务
mul.apply_async(args=(200, 50), eta=eta)
```

### 2 celery执行定时任务

### 3 首页轮播图定时更新（异步更新思路）

```python
1 首页轮播图加入缓存了
2 双写一致性问题（mysql数据改了，redis缓存数据还是一样的）
3 几种更新缓存的思路
 -定时更新（具体接口，看公司需求），我们为了测试，快一些，3s更新一次
 -异步更新（咱们现在没有）
  -新增轮播图接口 --> 新增完轮播图后 --> 执行异步更新轮播图的任务 --> update_banner.dely()
4 代码编写
```

```python
# 1 第0步：在celery.py中
# 从脚本中导入django环境
import django
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'luffyapi.settings.dev')
django.setup()

backend = 'redis://127.0.0.1:6379/1'
# 消息中间件
broker = 'redis://127.0.0.1:6379/2'
app = Celery(main=__name__, backend=backend, broker=broker,
             include=['celery_task.course_task', 'celery_task.user_task', 'luffyapi.apps.home.home_task'])

# 修改时区
# 时区
app.conf.timezone = 'Asia/Shanghai'
# 是否使用UTC
app.conf.enable_utc = False

# 执行定时任务（通过配置，配置beat，识别我们的配置，定时提交任务）
# 启动beat，启动worker

# 任务定时配置
from datetime import timedelta
from celery.schedules import crontab
app.conf.beat_schedule = {
    'update_banner': {
        'task': 'luffyapi.apps.home_task.update_banner',
        'schedule': timedelta(seconds=3),  # 3s后
    }
}

# 启动beat
# celery -A celery_task beat -l info
# 1 第一步 在home的APP下新建home_task.py
from celery_task import app

@app.task
def update_banner():
    from home.models import Banner
    from home.serializer import BannerSerializer
    from django.conf import settings
    from django.core.cache import cache
    # 更新轮播图缓存的任务
    # 只要这个task被执行一次，缓存中的轮播图就是最新的

    queryset = Banner.objects.all().filter(is_show=True, is_delete=False).order_by('-orders')[0:settings.BANNER_SIZE]
    ser = BannerSerializer(instance=queryset, many=True)
    # 小问题，图片前面的路径是没有带的，处理路径
    for item in ser.data:
        item['image'] = settings.REMOTE_ADDR + item['image']

    cache.set('banner_cache_list', ser.data)
    return True
# 2 第二步：一定要让django启动 在celery.py文件中加入
# 从脚本中导入django环境
import django
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'luffyapi.settings.dev')
django.setup()
# 3 第三步： 启动django项目，启动beat，启动worker
 -每隔3s就会更新缓存
```

### 4 课程主页页面

### 5 课程表分析（表数据录入）

```python
1 三种课程放到三个表中
 -免费课表
 -实战课表
 -轻课表
 -三个表可以抽出一个基表
2 真正路飞的实战课表
 -实战课表 一对多 老师表
  -其他字段了解
  -course_type=InterField(choice=(0, 'python'))
  -课程跟老师是一（老师）对多（课程）
   -teacher
  -课程跟课程分类是一（课程分类）对多（课程）
   -course_category
  
 -老师表
    
 -课程分类表
     -name
 -课程章节表 一对多
  -课程跟章节是一（课程）对多（章节）
  
 -课时表
  -章节和课时是一（章节）对多（课时）
    
```

5.1 数据录入

```python
1 把下面的sql复制到数据录入
2 图片
```

```python
资源手动迁移
# 头像图片放在 media/teacher 文件夹下
# 课程图片放在 media/course 文件夹下
老师表

INSERT INTO luffy_teacher(id, orders, is_show, is_delete, created_time, updated_time, name, role, title, signature, image, brief) VALUES (1, 1, 1, 0, '2019-07-14 13:44:19.661327', '2019-07-14 13:46:54.246271', 'Alex', 1, '老男孩Python教学总监', '金角大王', 'teacher/alex_icon.png', '老男孩教育CTO & CO-FOUNDER 国内知名PYTHON语言推广者 51CTO学院2016\2017年度最受学员喜爱10大讲师之一 多款开源软件作者 曾任职公安部、飞信、中金公司、NOKIA中国研究院、华尔街英语、ADVENT、汽车之家等公司');

INSERT INTO luffy_teacher(id, orders, is_show, is_delete, created_time, updated_time, name, role, title, signature, image, brief) VALUES (2, 2, 1, 0, '2019-07-14 13:45:25.092902', '2019-07-14 13:45:25.092936', 'Mjj', 0, '前美团前端项目组架构师', NULL, 'teacher/mjj_icon.png', '是马JJ老师, 一个集美貌与才华于一身的男人，搞过几年IOS，又转了前端开发几年，曾就职于美团网任高级前端开发，后来因为不同意王兴(美团老板)的战略布局而出家做老师去了，有丰富的教学经验，开起车来也毫不含糊。一直专注在前端的前沿技术领域。同时，爱好抽烟、喝酒、烫头(锡纸烫)。 我的最爱是前端，因为前端妹子多。');

INSERT INTO luffy_teacher(id, orders, is_show, is_delete, created_time, updated_time, name, role, title, signature, image, brief) VALUES (3, 3, 1, 0, '2019-07-14 13:46:21.997846', '2019-07-14 13:46:21.997880', 'Lyy', 0, '老男孩Linux学科带头人', NULL, 'teacher/lyy_icon.png', 'Linux运维技术专家，老男孩Linux金牌讲师，讲课风趣幽默、深入浅出、声音洪亮到爆炸');
分类表

INSERT INTO luffy_course_category(id, orders, is_show, is_delete, created_time, updated_time, name) VALUES (1, 1, 1, 0, '2019-07-14 13:40:58.690413', '2019-07-14 13:40:58.690477', 'Python');

INSERT INTO luffy_course_category(id, orders, is_show, is_delete, created_time, updated_time, name) VALUES (2, 2, 1, 0, '2019-07-14 13:41:08.249735', '2019-07-14 13:41:08.249817', 'Linux');
课程表

INSERT INTO luffy_course(id, orders, is_show, is_delete, created_time, updated_time, name, course_img, course_type, brief, level, pub_date, period, attachment_path, status, students, sections, pub_sections, price, course_category_id, teacher_id) VALUES (1, 1, 1, 0, '2019-07-14 13:54:33.095201', '2019-07-14 13:54:33.095238', 'Python开发21天入门', 'courses/alex_python.png', 0, 'Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土&&&Python从入门到入土', 0, '2019-07-14', 21, '', 0, 231, 120, 120, 0.00, 1, 1);

INSERT INTO luffy_course(id, orders, is_show, is_delete, created_time, updated_time, name, course_img, course_type, brief, level, pub_date, period, attachment_path, status, students, sections, pub_sections, price, course_category_id, teacher_id) VALUES (2, 2, 1, 0, '2019-07-14 13:56:05.051103', '2019-07-14 13:56:05.051142', 'Python项目实战', 'courses/mjj_python.png', 0, '', 1, '2019-07-14', 30, '', 0, 340, 120, 120, 99.00, 1, 2);

INSERT INTO luffy_course(id, orders, is_show, is_delete, created_time, updated_time, name, course_img, course_type, brief, level, pub_date, period, attachment_path, status, students, sections, pub_sections, price, course_category_id, teacher_id) VALUES (3, 3, 1, 0, '2019-07-14 13:57:21.190053', '2019-07-14 13:57:21.190095', 'Linux系统基础5周入门精讲', 'courses/lyy_linux.png', 0, '', 0, '2019-07-14', 25, '', 0, 219, 100, 100, 39.00, 2, 3);
章节表

INSERT INTO luffy_course_chapter(id, orders, is_show, is_delete, created_time, updated_time, chapter, name, summary, pub_date, course_id) VALUES (1, 1, 1, 0, '2019-07-14 13:58:34.867005', '2019-07-14 14:00:58.276541', 1, '计算机原理', '', '2019-07-14', 1);

INSERT INTO luffy_course_chapter(id, orders, is_show, is_delete, created_time, updated_time, chapter, name, summary, pub_date, course_id) VALUES (2, 2, 1, 0, '2019-07-14 13:58:48.051543', '2019-07-14 14:01:22.024206', 2, '环境搭建', '', '2019-07-14', 1);

INSERT INTO luffy_course_chapter(id, orders, is_show, is_delete, created_time, updated_time, chapter, name, summary, pub_date, course_id) VALUES (3, 3, 1, 0, '2019-07-14 13:59:09.878183', '2019-07-14 14:01:40.048608', 1, '项目创建', '', '2019-07-14', 2);

INSERT INTO luffy_course_chapter(id, orders, is_show, is_delete, created_time, updated_time, chapter, name, summary, pub_date, course_id) VALUES (4, 4, 1, 0, '2019-07-14 13:59:37.448626', '2019-07-14 14:01:58.709652', 1, 'Linux环境创建', '', '2019-07-14', 3);
课时表

INSERT INTO luffy_course_Section(id, is_show, is_delete, created_time, updated_time, name, orders, section_type, section_link, duration, pub_date, free_trail, chapter_id) VALUES (1, 1, 0, '2019-07-14 14:02:33.779098', '2019-07-14 14:02:33.779135', '计算机原理上', 1, 2, NULL, NULL, '2019-07-14 14:02:33.779193', 1, 1);

INSERT INTO luffy_course_Section(id, is_show, is_delete, created_time, updated_time, name, orders, section_type, section_link, duration, pub_date, free_trail, chapter_id) VALUES (2, 1, 0, '2019-07-14 14:02:56.657134', '2019-07-14 14:02:56.657173', '计算机原理下', 2, 2, NULL, NULL, '2019-07-14 14:02:56.657227', 1, 1);

INSERT INTO luffy_course_Section(id, is_show, is_delete, created_time, updated_time, name, orders, section_type, section_link, duration, pub_date, free_trail, chapter_id) VALUES (3, 1, 0, '2019-07-14 14:03:20.493324', '2019-07-14 14:03:52.329394', '环境搭建上', 1, 2, NULL, NULL, '2019-07-14 14:03:20.493420', 0, 2);

INSERT INTO luffy_course_Section(id, is_show, is_delete, created_time, updated_time, name, orders, section_type, section_link, duration, pub_date, free_trail, chapter_id) VALUES (4, 1, 0, '2019-07-14 14:03:36.472742', '2019-07-14 14:03:36.472779', '环境搭建下', 2, 2, NULL, NULL, '2019-07-14 14:03:36.472831', 0, 2);

INSERT INTO luffy_course_Section(id, is_show, is_delete, created_time, updated_time, name, orders, section_type, section_link, duration, pub_date, free_trail, chapter_id) VALUES (5, 1, 0, '2019-07-14 14:04:19.338153', '2019-07-14 14:04:19.338192', 'web项目的创建', 1, 2, NULL, NULL, '2019-07-14 14:04:19.338252', 1, 3);

INSERT INTO luffy_course_Section(id, is_show, is_delete, created_time, updated_time, name, orders, section_type, section_link, duration, pub_date, free_trail, chapter_id) VALUES (6, 1, 0, '2019-07-14 14:04:52.895855', '2019-07-14 14:04:52.895890', 'Linux的环境搭建', 1, 2, NULL, NULL, '2019-07-14 14:04:52.895942', 1, 4);
```

### 8 课程分类接口

```python
from rest_framework import serializers
from . import models


class CourseCategorySerializer(serializers.ModelSerializer):
    class Meta:
        model = models.CourseCategory
        fields = ('id', 'name')
```

### 9 课程群查接口（按价格排序、按id 按人气）

```python
 # 允许排序的字段
    ordering_fields = ['id', 'price', 'students']
```

### 10 群查接口返回四个课时

```python
   @property
    def section_list(self):
        # 检索所有章节所有课时，返回前4课时，不足4课时全部返回
        temp_section_list = []

        # coursechapters 反向查询表名小写_set.all()
        # related_name：反向查询，使用related_name指定的名字
        for chapter in self.coursechapters.all():
            # related_name='coursesections'
            for section in chapter.coursesections.all():
                temp_section_list.append({
                    'id': section.id,
                    'name': section.name,
                    'section_link': section.section_link,
                    'duration': section.duration,
                    # 是否可试看
                    'free_trail': section.free_trail,
                })
                if len(temp_section_list) >= 4:
                    return temp_section_list  # 最多4条

        return temp_section_list  # 不足4条
```

### 11 过滤和排序的使用

```python
1 排序
http://127.0.0.1:8000/course/free/?ordering=price
filter_backends = [ OrderingFilter]
 # 允许排序的字段
    ordering_fields = ['id', 'price', 'students']
2 过滤 （django-filter）
# 在视图类中
 filter_backends = [ DjangoFilterBackend]
 filter_fields = ('course_category',)
# 查询时
http://127.0.0.1:8000/course/free/?course_category=1
```

11.1 分页

```python
 # 分页组件
    pagination_class = pagination.PageNumberPagination
    
from rest_framework.pagination import PageNumberPagination as DrfPageNumberPagination

class PageNumberPagination(DrfPageNumberPagination):
    # 默认一页显示的条数
    page_size = 2
    # url中携带页码的key
    page_query_param = 'page'
    # url中用户携带自定义一页条数的key
    page_size_query_param = 'page_size'
    # 用户最大可自定义一页的条数
    max_page_size = 10
```

### 12 区间过滤(项目中没用)

#### 第一种方案，自定义过滤器

```python
from rest_framework.filters import BaseFilterBackend
class MyFilter(BaseFilterBackend):
    def filter_queryset(self, request, queryset, view):
        min_price = request.query_params.get('min_price')
        max_price = request.query_params.get('max_price')
        if min_price and max_price:
            queryset = queryset.filter(price__gte=min_price, price__lte=max_price)
        return queryset
# 配置在视图类中
filter_backends = [MyFilter]
就支持了区间过滤
```

#### 第二种：借助于django-filter

```python
# 写一个类
from django_filters.filterset import FilterSet
from . import models
from django_filters import filters
class CourseFilterSet(FilterSet):
    # 区间过滤：field_name关联的Model字段；lookup_expr设置规则；gt是大于，gte是大于等于；
    min_price = filters.NumberFilter(field_name='price', lookup_expr='gte')
    max_price = filters.NumberFilter(field_name='price', lookup_expr='lte')
    class Meta:
        model = models.Course
        # 如果过滤条件仅仅就是Model已有的字段，方式一更好
        # 但是方式二可以自定义过滤字段
        fields = ['course_category', 'min_price', 'max_price']
        # fields = ['course_category']
# 配置到视图类中
filter_backends = [ DjangoFilterBackend]
filter_class = CourseFilterSet
```

### 13 章节分类接口

### 14 七牛云视频托管

## 作业

 DNS UDP

单例模式

## 昨日回顾

```python
1 celery 分布式异步任务框架，异步，定时，延迟任务
 -python定时任务框架 APScheduler
2 包管理方式
 -在项目根路径建立一个包
 -celery.py ---> 把celery的对象实例化出来，本地化，定时任务
 -写很多task文件
 -在使用的位置，导入，调用
  -任务（函数）.delay()
 -启动worker
 -启动beat：如果有定时任务
 -任务.apply_async(args=[参数], 时间对象)
3 定时更新缓存的任务（异步更新）
 -mysql和redis 双写一致性
  -定时更新
  -新增一条直接删除缓存（更新缓存）
  -先删缓存，再存数据
 -celery使用django，必须让django运行
4 复制课程列表页面
5 查询所有分类的接口
6 查询所有课程接口
 -加入分页
 -加入排序
 -过滤：课程分类过滤
 -区间过滤
```

## 今日内容

### 1 课程列表前端

### 2 课程详情前端

```python
1 this.$route的解释
// this.$route 当前正在在的路由对象
    // 在路由文件中如果：path: '/actualcourse/detail/:id',
    // id就会被放到路由对象的 params对象中
    // 在路由文件中如果：path: '/actualcourse/detail/2?name=lqz',
    // name会被放到路由对象的：query对象中
    console.log(this.$route)
    
2 this.$route 和 this.$router区别
 -第一个是当前正在访问的路由对象
 -第二个是vue-router对象
3 video-player下载播放器组件
 -cnpm install vue-video-player
 -在main.js中配置
    // vue-video播放器
    require('video.js/dist/video-js.css');
    require('vue-video-player/src/custom-theme.css');
    import VideoPlayer from 'vue-video-player'
    Vue.use(VideoPlayer);
 -在组件中
    import { videoPlayer } from 'vue-video-player'
    在组件中注册
 -在HTML中
     <videoPlayer class="video-player vjs-custom-skin"
                       ref="videoPlayer"
                       :playsinline="true"
                       :options="playerOptions"
                       @play="onPlayerPlay($event)"
                       @pause="onPlayerPause($event)">
          </videoPlayer>
 -在js中
     playerOptions: {
        aspectRatio: '16:9', // 将播放器置于流畅模式，并在计算播放器的动态大小时使用该值。值应该代表一个比例 - 用冒号分隔的两个数字（例如"16:9"或"4:3"）
        sources: [{ // 播放资源和资源格式
          type: 'video/mp4',
          src: 'http://img.ksbbs.com/asset/Mon_1703/05cacb4e02f9d9e.mp4' // 你的视频地址（必填）
        }]
      }
```

### 3 七牛视频托管

### 4 搜索接口

### 5 支付宝支付介绍

### 6 支付宝SDK支付演示

### 7 支付宝支付封装

### 8 订单表分析

### 9 订单相关接口分析

### 10 支付接口

### 11 支付前端和支付成功前端跳转

### 12 支付成功

## 作业

一对多的关系一旦确立关联字段写在多的一方

## 昨日回顾

```python
1 爬虫介绍
 -发送http请求获取数据 ---> 解析，清洗 ---> 入库
2 请求库-requests(urllib2, requests-html) 
3 requests发送get请求，携带数据，携带头，携带cookie
4 发送post请求，携带数据（请求地址中的数据，请求体中数据：data,json）
5 响应对象的属性 http的响应
 -状态码
 -响应头
 -cookie  # cookiejar对象
 -响应体内容二进制
 -响应体的字符串text
 -字节字符
 -iter_content()
6 爬取梨视频
```

## 今日内容

### 1 requests高级用法

### 2 抽屉自动点赞

### 3 爬取汽车之家新闻

### 4 bs4 介绍

### 5 bs4 之遍历文档树

### 6 bs4 之搜索文档树

### 7 css选择器

## 作业

```python
简易的命令行入门教程:
Git 全局设置:

git config --global user.name "PENGJUNQIAO"
git config --global user.email "1746259155@qq.com"
创建 git 仓库:

mkdir hippo
cd hippo
git init
touch README.md
git add README.md
git commit -m "first commit"
git remote add origin git@gitee.com:pengjunqiao/hippo.git
git push -u origin master
已有仓库?

cd existing_git_repo
git remote add origin git@gitee.com:pengjunqiao/hippo.git
git push -u origin master
```

## 昨日回顾

## 今日内容

### 1 requests高级用法

### 2 抽屉自动点赞

### 3 爬取汽车之家新闻

```python
# -*- coding = utf-8 -*-
# @Time: 2022/1/8 22:23
# @Author: Thinkpad
# @File: 爬取汽车之家.py
# @Software: PyCharm
import requests
from bs4 import BeautifulSoup

res = requests.get("https://www.autohome.com.cn/news/1/#liststart")
# print(res.text)

# 第一个参数是要解析的文档
# 第二个参数是使用的解析库，解析方式
#  "html.parser" 解析速度慢 不需要额外安装
#  lxml          解析速度快 文档容错率高，需要额外安装lxml模块
soup = BeautifulSoup(res.text, 'lxml')
# 查找文档中的所有li标签
# li_list = soup.find_all(name='li')
# 查找文档中的所有类名为article的ul标签
ul_list = soup.find_all(name='ul', class_='article')

li_list = []
for ul in ul_list:
    li_list += ul.find_all(name='li')

# print(len(li_list))
for li in li_list:
    h3 = li.find(name='h3')
    if h3:
        title = h3.text  # 获取文本内容

        url = 'http:' + li.find(name='a')['href']

        desc = li.find(name='p').text

        img = 'http:' + li.find(name='img')['src']

        print("""
        文章标题：%s
        文章地址：%s
        文章图片：%s
        文章摘要：%s
        """ % (title, url, img, desc))

        # 存到mysql中
        # 存redis中 列表
```

### 4 bs4介绍

```python
1 Beautiful Soup 是一个可以从HTML或XML文件中提取数据的python库
2 默认有个解析器 html.parser
3 额外安装 lxml
4 HTML中搜索数据的时候
 -CSS选择器 (通用)
 -xpath选择器 (通用)
 -模块提供的查找方法 (find find_all)
```

### 5 bs4之遍历文档树

### 6 bs4之搜索文档树

## 昨日回顾

```python
1 requests高级使用
 -使用代理
 -上传文件
2 自动点赞
 -模拟的很像浏览器（浏览器带什么你就带什么）
3 爬取新闻
 -requests+bs4的简单使用
4 bs的详细使用
 -遍历文档树：
    -.
    -标签名字
    -标签属性
    -标签的文本
 -搜索文档树
    -find
    -find_all  name属性标签名 attrs属性 class_.name
    -5种过滤器：字符串，正则，列表，布尔，方法
    -limit 是否递归
 -css选择器
  -soup.select('css选择器')
5 selenium的基本使用
 -驱动浏览器，模拟人的行为
 -相应浏览器的驱动(谷歌 --> 谷歌浏览器版本对应)
 -驱动放在了项目根路径，实例化得到对象的时候，指定路径
```

## 今日内容

1 代理池搭建

2 selenium的基本使用（模拟登陆百度

3 无界面浏览器

4 selenium的其他用法

4.1获取位置，属性，大小

4.2等待元素被加载

4.3元素操作

```python
1 向input框输入值
 对象.send_keys('值')
2 点击控件
 对象.click()
3 清空input框中的值
 对象.clear()
```

4.4 执行js代码

```python
# 在页面上执行js代码
driver.execute_script('alert('d')')
页面滑屏
driver.execute_script('window.scrollBy(0,document.body.scrollHeight)')
```

4.5 切换选项卡

4.6 浏览器前进后退

```python
#模拟浏览器的前进后退
import time
from selenium import webdriver

browser=webdriver.Chrome()
browser.get('https://www.baidu.com')
browser.get('https://www.taobao.com')
browser.get('http://www.sina.com.cn/')

browser.back()
time.sleep(10)
browser.forward()
browser.close()
```

5 selenium登录cnblogs获取cookie

6 自动点赞

7 爬取京东

8 几个爬虫案例

## 作业

## 昨日回顾

```python
1 selenium 模拟人的行为
 find_elements_by_xx
 find_elements_by_css_select('css选择器')
 send_keys
 click
 clear
2 无界面浏览器
 -配置
3 其他使用
 -标签位置，标签大小，标签属性
 -模拟浏览器前进后退
 -tab切换
 -获取cookie
  -driver.get_cookies()  ---> 列表
4 代理池和cookie池的使用
5 抽屉自动点赞
 -使用selenium登录，使用requests发请求
6 爬取京东商品信息
 -img懒加载
 -点击下一页
```

## 今日内容

### 1 动作链和切换frame（了解）

````python
# 切换frame
driver.switch_to.frame("id")
# 动作链
ActionChains(driver).drag_and_drop(sourse,target).perform() #把动作放到动作链中，准备串行执行
ActionChains(driver).drag_and_drop_by_offset(sourse,10,0).perform()
# 鼠标点住
   ActionChains(driver).click_and_hold(sourse).perform()
    distance=target.location['x']-sourse.location['x']

```
track=0
while track < distance:
    ActionChains(driver).move_by_offset(xoffset=2,yoffset=0).perform()
    track+=2

ActionChains(driver).release().perform()
````

#### 2 打码平台的使用

```python
1 别的平台帮助我们识别验证码，我们只需要花点钱
2 云打码 超级鹰
3 注册用户 ---> 充钱
4 看一下价格体系 ---识别不同验证码价格不一样
```

### 3 xpath的使用

### 4 自动登录12306

### 5 scrapy介绍，架构，安装

### 6 scrapy创建项目，创建爬虫

### 7 scrapy目录介绍

## 上节回顾

```python
1 切换frame，动作链
2 xpath选择：在xml中查找内容的一门语言
 - .
 -..
 -/
 -//
 -@
    
3 自动登录12306
 -打码平台使用：别人帮我们破解验证码
 -使用selenium点击，滑动
 -有的网站会校验是否使用了自动化测试软件：
     -window.navigator.webdriver
 -获取验证码
  -验证码图片的位置和大小
  -屏幕截图，pillow抠图
  -base64编码，存成图片，前面有标志部分
  -base64的解码和编码
4 scrapy
 -pip3 install scrapy
 -7个组件
  -爬虫：spiders文件夹下的一个py文件，爬取的地址，解析数据
  -爬虫中间件：介于爬虫和引擎之间的
  -引擎：大总管，负责数据的流动
  -调度器：爬取地址的调度和去重
  -下载中间件
  -下载器：负责下载
  -管道：存储，数据清洗
 -创建项目，创建爬虫
  -scrapy startproject 名字
  -scrapy genspider 爬虫名 爬虫地址
  -scrapy crawl 爬虫名
 -目录结构
 -数据解析
  -xpath
        -拿文本：'.//a/text()'
        -拿属性：'.//a/@href'
  -css
        -拿文本：'a.link-title::text'
        -拿属性：'img.image-scale::attr(src)'
  -取一条：extract_first()
  -取多条：extract()
```

## 今日内容

### 1 setting中相关配置

```python
1 是否遵循爬虫协议
ROBOTSTXT_OBEY = False
2 请求客户端类型
USER_AGENT = 'Mozilla/5.0......'
3 日志级别（比--nolog好在，如果出错，控制台会报错）
LOG_LECEL = 'ERROR'
```

### 2 持久化方案

```python
1 第一种方案：直接存（很少）
 - 解析函数返回列表套字典
 - 执行：scrapy crawl cnblogs -o 文件名(json.pkl.csv结尾)
2 第二种：通用方法（pipline）
 - 1 在items.py中写一个类，继承scrapy.Item
 - 2 在类中写属性
     title = scrapy.Field()
 - 3 在爬虫中导入类，实例化得到对象，把要保存的数据放到对象中
     item['title'] = title
 - 4 修改配置文件，指定pipline，数字表示优先级，越小越大
     ITEM_PIPELINES = {'crawl_cnblogs.pipelines.CrawlcnblogsPipeline': 300,}
 - 5 写一个pipline：CrawlCnblogsPipeline
     - open_spider: 数据初始化，打开文件，打开数据库链接
       - process_item: 真正存储的地方
        - 一定不要忘了return item 交给后续的pipline继续使用
       - close_spider：销毁资源，关闭文件，关闭数据库链接
        
```

### 3 全站爬取cnblogs文章（scrapy的请求传参）

3.1 request和response对象传递参数

```python
1 在request对象中
Request(url=url, callback=self.parse_detail.meta={'item':item})
2 在response对象中
item = response.meta['item']
```

3.2 解析出下一页的地址并继续爬取

```python

```

### 4 提高爬取效率

```python
# 1 增加并发：
默认scrapy开启的并发线程为32个，可以适当进行增加，在setting配置文件中修改CONCURRENT_REQUESTS = 100值为100，并发设置成了为100.
# 2 降低日志级别：
在运行scrapy时，会有大量日志信息的输出，为了减少CPU的使用率，可以设置log输出信息为INFO或者ERROR即可，在配置文件中编写：LOG_LEVEL = 'INFO'
# 3 禁止cookie：
如果不是真的需要cookie，则再scrapy爬取数据时可以禁止cookie从而减少CPU的使用率，提升爬取效率，在配置文件中编写：COOKIES_ENABLED = False
# 4 禁止重试：
对失败的HTTP进行重新请求（重试）会减慢爬取速度，因此可以禁止重试，在配置文件中编写：RETRY_ENABLED = False
# 5 减少下载超时：
如果对一个非常慢的链接进行爬取，减少下载超时可以能让卡住的链接快速被放弃，从而提升效率，在配置文件中进行编写：DOWNLOAD_TIMEOUT = 10 超时时间为10S
```

### 5 爬虫中间件和下载中间件

```python
1 爬虫和下载中间件要使用，需要在配置文件中
SPIDER_MIDDLEWARES = 

DOWNLOADER_MIDDLEWARES

下载中间件
# 图中第四步，触发它（如代理，修改请求头，加入cookie，使用selenium）
def process_request(self, request，spider)
 # Must either:
 # - return None: continue processing this request, 继续下一个中间件的process_request
 # - return Response : 交给引擎处理，引擎会把该响应对象给爬虫
 # - return Request : 交给引擎处理，引擎会把它放到调度器中
 #  installed downloader m
# 图中第五步，触发它，很少使用，要对响应对象进行处理
def process_response(self, request, response, spider)
# 爬虫出异常，触发它
def process_exception(self, request, exception, spider)
# 
def spider_opened(self, spider)
```

### 6 加代理，加cookie，加header，加selenium

```PYTHON
0 在下载中间件的process_request方法中
1 加cookie
 # request.cookies['name'] = 'lqz'
 # request.cookies = {}
2 修改header
 # request.headers['Auth'] = 'adssgafga'
 # request.headers['USER-AGENT'] = 'ssss'
3 加代理
 request.meta['proxy'] = 'http://103.130.172.34:8080'
4 fake_useragent模块，可以随机生成user-aget
5 如果process_request返回的是Request对象
 -会交给引擎，引擎把请求放到调度中，等待下次被调度
6 集成selenium
 -在爬虫类中类属性
    driver = webdriver.Chrome(executable_path='')
 - 在爬虫类中方法：
    def close(spider, reason):
        spider.driver.close()
 -在中间件中的process_request中
    from scrapy.http import HtmlResponse
    spider.driver.get(url=request.url)
    
response=HtmlResponse(url=request.url.body=spider.driver.page_source.encode('utf-8'),request=request)
return response

```

### 7 去重规则源码分析

```python
1 使用了集合去重
2 默认使用的去重类：
DUPEFILTER_CLASS = 'scrapy.dupefilters.RFPDupeFilter'
3 后期你可以自己写一个类，替换掉内置的去重
 -布隆过滤器

```

### 8 scrapy-redis实现分布式爬虫

解决pip install python-alipay-sdk --upgrade安装失败：

sudo apt install --reinstall gcc
