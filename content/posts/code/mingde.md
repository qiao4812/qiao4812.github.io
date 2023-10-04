---
title: "明德项目"
date: 2023-04-09T23:17:05+08:00
draft: true
tags: ["项目", "Python"]
categories: ["项目"]
---

# 明德在线教育项目

## 一、环境搭建

###  创建虚拟环境

anaconda、miniconda、virtualenv

```python
conda create -n mingde python=3.10
```

### 虚拟环境相关命令

```python
# anaconda或miniconda
创建虚拟环境：               
conda create -n 虚拟环境名称 python=版本号

查看所有虚拟环境：           
conda env list

使用虚拟环境：               
source activate 虚拟环境名称

退出当前虚拟环境：           
conda deactivate

删除虚拟环境（必须先退出虚拟环境内部才能删除当前虚拟环境）:
conda remove -n 虚拟环境名称 --all

在当前虚拟环境安装模块       
conda install -c conda-forge 包名==版本号  # -c channel

在当前虚拟环境移除模块       
conda remove 包名

查看虚拟环境中安装的包：              
pip freeze  或者 pip list

收集当前环境中安装的包及其版本：       
pip freeze > requirements.txt

在部署项目的服务器中安装项目使用的模块： 
pip install -r requirements.txt
```

### 依赖包安装

#### PyPI 清华镜像使用帮助 <https://mirrors.tuna.tsinghua.edu.cn/help/pypi/>

```
pip install django

pip install djangorestframework  

pip install Pillow  

pip install -i https://pypi.tuna.tsinghua.edu.cn/simple PyMySQL
```

## 二、搭建项目并运行

### 创建服务端项目

```bash
~/Code
➜ mcd mingde  # mkdir mingde cd mingde

~/Code/mingde
➜ conda activate mingde  # 报错

CommandNotFoundError: Your shell has not been properly configured to use 'conda activate'.
To initialize your shell, run

    $ conda init <SHELL_NAME>

Currently supported shells are:
  - bash
  - fish
  - tcsh
  - xonsh
  - zsh
  - powershell

See 'conda init --help' for more information and options.

IMPORTANT: You may need to close and restart your shell after running 'conda init'.



~/Code/mingde
➜ source activate  # 解决


~/Code/mingde via 🅒 base
➜ conda activate mingde


~/Code/mingde via 🅒 mingde took 6.1s
➜ django-admin startproject mingdeapi  # 创建api服务端项目

~/Code/mingde via 🅒 mingde
➜

```

### Pycharm 打开运行

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304100009888.png)

执行命令：

```bash
python manage.py runserver
```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304100012206.png)



修改启动参数

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304100014857.png)

运行项目，访问地址：`http://127.0.0.1:8000`

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304100017359.png)



## 三、调整目录

```Python
➜ tree
.
└── mingdeapi                     # API 服务端项目目录
    ├── db.sqlite3
    ├── logs                      # 项目运行时/开发时日志目录
    │   └── mingde.log
    ├── manage.py
    ├── mingdeapi                 # 项目主应用
    │   ├── __init__.py
    │   ├── apps                 # 开发者的代码保存目录 模块[子应用]
    │   │   └── __init__.py
    │   ├── asgi.py
    │   ├── db.sqlite3
    │   ├── libs                 # 第三方库的保存目录
    │   │   └── __init__.py
    │   ├── settings
    │   │   ├── __init__.py  
    │   │   ├── dev.py          # 项目开发时的本地配置
    │   │   └── pro.py          # 项目上线时的本地配置
    │   ├── urls.py              # 总路由
    │   ├── utils                # 公共函数类库 [自己开发的组件]
    │   │   └── __init__.py
    │   └── wsgi.py
    └── scripts                   # 项目运行时维护项目的脚本文件
        └── __init__.py

```

asgi.py 文件修改

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304101019984.png)

wsgi.py 文件修改

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304101022870.png)

manage.py 文件修改

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304101023468.png)

## 四、在GitHub上创建代码仓库

创建 git 代码仓库

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304162231176.png)



```bash
~/Code
➜ cd mingde

~/Code/mingde
➜

~/Code/mingde
➜ git init
提示：使用 'master' 作为初始分支的名称。这个默认分支名称可能会更改。要在新仓库中
提示：配置使用初始分支名，并消除这条警告，请执行：
提示：
提示：	git config --global init.defaultBranch <名称>
提示：
提示：除了 'master' 之外，通常选定的名字有 'main'、'trunk' 和 'development'。
提示：可以通过以下命令重命名刚创建的分支：
提示：
提示：	git branch -m <name>
已初始化空的 Git 仓库于 /Users/qiaopengjun/Code/mingde/.git/

mingde on  master [?]
➜ git add .

mingde on  master [+]
➜ git commit -m "fix: add api project"
[master（根提交） 598de28] fix: add api project
 26 files changed, 524 insertions(+)
 create mode 100644 mingdeapi/.idea/.gitignore
 create mode 100644 mingdeapi/.idea/inspectionProfiles/Project_Default.xml
 create mode 100644 mingdeapi/.idea/inspectionProfiles/profiles_settings.xml
 create mode 100644 mingdeapi/.idea/mingdeapi.iml
 create mode 100644 mingdeapi/.idea/misc.xml
 create mode 100644 mingdeapi/.idea/modules.xml
 create mode 100644 mingdeapi/db.sqlite3
 create mode 100644 mingdeapi/logs/mingde.log
 create mode 100755 mingdeapi/manage.py
 create mode 100644 mingdeapi/mingdeapi/__init__.py
 create mode 100644 mingdeapi/mingdeapi/__pycache__/__init__.cpython-310.pyc
 create mode 100644 mingdeapi/mingdeapi/__pycache__/urls.cpython-310.pyc
 create mode 100644 mingdeapi/mingdeapi/__pycache__/wsgi.cpython-310.pyc
 create mode 100644 mingdeapi/mingdeapi/apps/__init__.py
 create mode 100644 mingdeapi/mingdeapi/asgi.py
 create mode 100644 mingdeapi/mingdeapi/db.sqlite3
 create mode 100644 mingdeapi/mingdeapi/libs/__init__.py
 create mode 100644 mingdeapi/mingdeapi/settings/__init__.py
 create mode 100644 mingdeapi/mingdeapi/settings/__pycache__/__init__.cpython-310.pyc
 create mode 100644 mingdeapi/mingdeapi/settings/__pycache__/dev.cpython-310.pyc
 create mode 100644 mingdeapi/mingdeapi/settings/dev.py
 create mode 100644 mingdeapi/mingdeapi/settings/pro.py
 create mode 100644 mingdeapi/mingdeapi/urls.py
 create mode 100644 mingdeapi/mingdeapi/utils/__init__.py
 create mode 100644 mingdeapi/mingdeapi/wsgi.py
 create mode 100644 mingdeapi/scripts/__init__.py

mingde on  master
➜ git remote add origin git@github.com:qiaopengjun5162/mingde.git

mingde on  master
➜ git pull
remote: Enumerating objects: 7, done.
remote: Counting objects: 100% (7/7), done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 7 (delta 1), reused 0 (delta 0), pack-reused 0
展开对象中: 100% (7/7), 2.85 KiB | 729.00 KiB/s, 完成.
来自 github.com:qiaopengjun5162/mingde
 * [新分支]          main       -> origin/main
当前分支没有跟踪信息。
请指定您要合并哪一个分支。
详见 git-pull(1)。

    git pull <远程> <分支>

如果您想要为此分支创建跟踪信息，您可以执行：

    git branch --set-upstream-to=origin/<分支> master


mingde on  master took 5.0s
➜ git push
致命错误：当前分支 master 没有对应的上游分支。
为推送当前分支并建立与远程上游的跟踪，使用

    git push --set-upstream origin master

为了让没有追踪上游的分支自动配置，参见 'git help config' 中的 push.autoSetupRemote。


mingde on  master
➜ git push -u origin main
错误：源引用规格 main 没有匹配
错误：无法推送一些引用到 'github.com:qiaopengjun5162/mingde.git'


mingde on  master
➜ git branch -m main

mingde on  main took 1m 15.1s
➜ git push -u origin +main
枚举对象中: 34, 完成.
对象计数中: 100% (34/34), 完成.
使用 12 个线程进行压缩
压缩对象中: 100% (28/28), 完成.
写入对象中: 100% (34/34), 8.77 KiB | 4.39 MiB/s, 完成.
总共 34（差异 4），复用 0（差异 0），包复用 0
remote: Resolving deltas: 100% (4/4), done.
To github.com:qiaopengjun5162/mingde.git
 + 8e77621...598de28 main -> main (forced update)
分支 'main' 设置为跟踪 'origin/main'。

mingde on  main took 4.2s
```

## 五、日志配置

在settings/dev.py 文件中追加如下配置：

```python
# 日志配置
# https://docs.djangoproject.com/zh-hans/4.2/topics/logging/
LOGGING = {
    'version': 1,  # 使用的日志模块的版本，目前官方提供的只有版本1，但是官方有可能会升级，为了避免升级出现的版本问题，所以这里固定为1
    'disable_existing_loggers': False,  # 是否禁用其他的已经存在的日志功能？肯定不能，有可能有些第三方模块在调用，所以禁用了以后，第三方模块无法捕获自身出现的异常了。
    'formatters': {  # 日志格式设置，verbose或者simple都是自定义的
        'verbose': {  # 详细格式，适合用于开发人员不在场的情况下的日志记录。
            # 格式定义：https://docs.python.org/3/library/logging.html#logrecord-attributes
            # levelname 日志等级
            # asctime   发生时间
            # module    文件名
            # process   进程ID
            # thread    线程ID
            # message   异常信息
            'format': '{levelname} {asctime} {module} {funcName} {process:d} {thread:d} {message}',
            'style': '{',  # 变量格式分隔符
        },
        'simple': {  # 简单格式，适合用于开发人员在场的情况下的终端输出
            'format': '{levelname} {module} {funcName} {message}',
            'style': '{',
        },
    },
    'filters': {  # 过滤器
        'require_debug_true': {
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'handlers': {  # 日志处理流程，console或者mail_admins都是自定义的。
        'console': {
            'level': 'DEBUG',  # 设置当前日志处理流程中的日志最低等级
            'filters': ['require_debug_true'],  # 当前日志处理流程的日志过滤
            'class': 'logging.StreamHandler',  # 当前日志处理流程的核心类，StreamHandler可以帮我们把日志信息输出到终端下
            'formatter': 'simple'  # 当前日志处理流程的日志格式
        },
        # 'mail_admins': {
        #     'level': 'ERROR',                  # 设置当前日志处理流程中的日志最低等级
        #     'class': 'django.utils.log.AdminEmailHandler',  # AdminEmailHandler可以帮我们把日志信息输出到管理员邮箱中。
        #     'filters': ['special']             # 当前日志处理流程的日志过滤
        # }
        # 按文件大小来分割日志
        'file': {
            'level': 'DEBUG',
            'class': 'logging.handlers.RotatingFileHandler',
            # 日志位置,日志文件名，日志保存目录logs必须手动创建
            'filename': BASE_DIR.parent / "logs/mingde.log",
            # 单个日志文件的最大值，这里我们设置300M
            'maxBytes': 300 * 1024 * 1024,
            # 备份日志文件的数量，设置最大日志数量为10
            'backupCount': 20,
            # 日志格式: 详细格式
            'formatter': 'verbose'
        },
    },
    'loggers': {  # 日志处理的命名空间
        'django': {  # 要在django中调用当前配置项的loging写入日志到文件中，名字必须叫"django"
            'handlers': ['console', 'file'],  # 当基于django命名空间写入日志时，调用那几个日志处理流程
            'propagate': True,  # 是否在django命名空间对应的日志处理流程结束以后，冒泡通知其他的日志功能。True表示允许
        },
    }
}

```

提交git版本

```bash
mingde/mingdeapi on  main [!] via 🅒 mingde 
➜ cd ..    

mingde on  main [!] via 🅒 mingde 
➜ ga # git add .

mingde on  main [+] via 🅒 mingde 
➜ git commit -m "fix: 日志初始化"     
[main d77f5b8] fix: 日志初始化
 1 file changed, 61 insertions(+), 6 deletions(-)

mingde on  main [⇡] via 🅒 mingde 
➜ gp  # git push                           
枚举对象中: 11, 完成.
对象计数中: 100% (11/11), 完成.
使用 12 个线程进行压缩
压缩对象中: 100% (6/6), 完成.
写入对象中: 100% (6/6), 3.09 KiB | 3.09 MiB/s, 完成.
总共 6（差异 4），复用 0（差异 0），包复用 0
remote: Resolving deltas: 100% (4/4), completed with 4 local objects.
To github.com:qiaopengjun5162/mingde.git
   ca79d4c..d77f5b8  main -> main

mingde on  main via 🅒 mingde took 4.6s 

```

## 六、异常处理

新建utils/exceptions.py

```python
from rest_framework.views import exception_handler

from django.db import DatabaseError
from rest_framework.response import Response
from rest_framework import status

import logging

logger = logging.getLogger('django')


def custom_exception_handler(exc, context):
    """
    自定义异常处理
    :param exc: 异常类
    :param context: 抛出异常的上下文
    :return: Response 响应对象
    """
    # 调用drf框架原生的异常处理方法
    response = exception_handler(exc, context)

    if response is None:
        view = context['view']
        if isinstance(exc, DatabaseError):
            # 数据库异常
            logger.error('[%s] %s' % (view, exc))
            response = Response({'message': '服务器内部错误'}, status=status.HTTP_507_INSUFFICIENT_STORAGE)

    return response

```

在settings/dev.py 文件中追加如下配置：

```python
# drf配置
REST_FRAMEWORK = {
    # 自定义异常处理
    'EXCEPTION_HANDLER': 'mingdeapi.utils.exceptions.exception_handler',
}
```

提交代码版本

```bash
mingde on  main [⇡!?] via 🅒 mingde 
➜ git add . 

mingde on  main [⇡+] via 🅒 mingde 
➜ git commit -m "fix: 自定义异常处理"
[main 916f235] fix: 自定义异常处理
 2 files changed, 42 insertions(+)
 create mode 100644 mingdeapi/mingdeapi/utils/exceptions.py

mingde on  main [⇡] via 🅒 mingde 
➜ git push                              
枚举对象中: 19, 完成.
对象计数中: 100% (19/19), 完成.
使用 12 个线程进行压缩
压缩对象中: 100% (12/12), 完成.
写入对象中: 100% (12/12), 1.01 KiB | 1.01 MiB/s, 完成.
总共 12（差异 9），复用 0（差异 0），包复用 0
remote: Resolving deltas: 100% (9/9), completed with 6 local objects.
To github.com:qiaopengjun5162/mingde.git
   e70dab2..916f235  main -> main

mingde on  main via 🅒 mingde took 4.0s 
➜ 

```

## 七、创建数据库并配置数据库连接

```bash
~
➜ mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 8.0.32 Homebrew

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database mingde;
Query OK, 1 row affected (0.01 sec)

mysql>

mysql> set global validate_password.policy=LOW;  # 修改指定密码的验证强度等级
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE 'validate_password%';  # 查看密码策略
+--------------------------------------+-------+
| Variable_name                        | Value |
+--------------------------------------+-------+
| validate_password.check_user_name    | ON    |
| validate_password.dictionary_file    |       |
| validate_password.length             | 8     |
| validate_password.mixed_case_count   | 1     |
| validate_password.number_count       | 1     |
| validate_password.policy             | LOW   |
| validate_password.special_char_count | 1     |
+--------------------------------------+-------+
7 rows in set (0.01 sec)

# 创建用户：create user '用户名'@'主机地址' identified by '密码';
mysql> create user 'mingde_user'@'%' identified by '12345678';
Query OK, 0 rows affected (0.00 sec)

# 分配权限：grant 权限选项 on 数据库名.数据表 to 'mingde_user'@'%' with grant option;
mysql> grant all privileges on mingde.* to 'mingde_user'@'%' with grant option;
Query OK, 0 rows affected (0.01 sec)

```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304171050345.png)



安装连接池相关包

https://pypi.org/project/cryptography/

```bash
pip install django-db-connection-pool
pip install cryptography
```

https://docs.djangoproject.com/zh-hans/4.2/ref/databases/

在settings/dev.py 文件中配置：

```python
DATABASES = {
    # 'default': {
    #     'ENGINE': 'django.db.backends.sqlite3',
    #     'NAME': BASE_DIR / 'db.sqlite3',
    # }
    'default': {
        # 'ENGINE': 'django.db.backends.mysql',
        'ENGINE': 'dj_db_conn_pool.backends.mysql',
        'NAME': 'mingde',
        'PORT': 3306,
        'HOST': '127.0.0.1',
        'USER': 'mingde_user',
        'PASSWORD': '12345678',
        'OPTIONS': {},
        'POOL_OPTIONS': {  # 连接池的配置信息
            'POOL_SIZE': 10,  # 连接池默认创建的链接对象的数量
            'MAX_OVERFLOW': 10  # 连接池默认创建的链接对象的最大数量
        }
    }
}
```

在项目主应用下的 `mingdeapi.__init__.py`中导入pymysql

```python
import pymysql

pymysql.install_as_MySQLdb()

```

提交版本

```bash
mingde on  main via 🅒 mingde took 4.0s 
➜ ga      

mingde on  dev [+] via 🅒 mingde 
➜ git commit -m "fix: mysql数据库初始化"
[dev 9623874] fix: mysql数据库初始化
 2 files changed, 19 insertions(+), 2 deletions(-)

mingde on  dev via 🅒 mingde 
➜ gp
致命错误：当前分支 dev 没有对应的上游分支。
为推送当前分支并建立与远程上游的跟踪，使用

    git push --set-upstream origin dev

为了让没有追踪上游的分支自动配置，参见 'git help config' 中的 push.autoSetupRemote。


mingde on  dev via 🅒 mingde 
➜ git push --set-upstream origin dev                                                                                                                                                                                                    

枚举对象中: 13, 完成.
对象计数中: 100% (13/13), 完成.
使用 12 个线程进行压缩
压缩对象中: 100% (6/6), 完成.
写入对象中: 100% (7/7), 891 字节 | 891.00 KiB/s, 完成.
总共 7（差异 5），复用 0（差异 0），包复用 0
remote: Resolving deltas: 100% (5/5), completed with 5 local objects.
remote: 
remote: Create a pull request for 'dev' on GitHub by visiting:
remote:      https://github.com/qiaopengjun5162/mingde/pull/new/dev
remote: 
To github.com:qiaopengjun5162/mingde.git
 * [new branch]      dev -> dev
分支 'dev' 设置为跟踪 'origin/dev'。

mingde on  dev via 🅒 mingde took 4.4s 

mingde on  dev via 🅒 mingde took 4.4s 
➜ git branch                        

mingde on  dev via 🅒 mingde took 3.6s 
➜ git checkout main                     
切换到分支 'main'
您的分支与上游分支 'origin/main' 一致。

mingde on  main via 🅒 mingde 
➜ git pull                          
已经是最新的。

mingde on  main via 🅒 mingde took 4.3s 
➜ git merge dev                            
更新 916f235..9623874
Fast-forward
 mingdeapi/mingdeapi/__init__.py     |  3 +++
 mingdeapi/mingdeapi/settings/dev.py | 18 ++++++++++++++++--
 2 files changed, 19 insertions(+), 2 deletions(-)

mingde on  main [⇡] via 🅒 mingde 
➜ git push                          
总共 0（差异 0），复用 0（差异 0），包复用 0
To github.com:qiaopengjun5162/mingde.git
   916f235..9623874  main -> main

mingde on  main via 🅒 mingde took 3.8s 

```

## 八、缓存配置

### django-redis 中文文档

<https://django-redis-chs.readthedocs.io/zh_CN/latest/#>

- 安装 django-redis 最简单的方法就是用 pip :

```python
pip install django-redis
```

- 在settings/dev.py 文件中配置：

```python
# redis configuration
# https://django-redis-chs.readthedocs.io/zh_CN/latest/#
# 设置redis缓存
CACHES = {
    # 默认缓存
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        # 项目上线时,需要调整这里的路径
        "LOCATION": "redis://127.0.0.1:6379/0",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
            "CONNECTION_POOL_KWARGS": {"max_connections": 100}  # 配置默认连接池
        }
    },
    # 提供给admin站点的session存储
    "session": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
            "CONNECTION_POOL_KWARGS": {"max_connections": 100}
        }
    },
    # 提供存储短信验证码
    "sms_code": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/2",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
            "CONNECTION_POOL_KWARGS": {"max_connections": 100}
        }
    }
}

# 设置用户登录admin站点时,记录登录状态的session保存到redis缓存中
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
# 设置session保存的位置对应的缓存配置项
SESSION_CACHE_ALIAS = "session"
```

- django-redis提供了get_redis_connection的方法
- 通过调用get_redis_connection方法传递redis的配置名称可获取到redis的连接对象

- 通过redis连接对象可以执行redis命令

redis-py - Python Client for Redis： <https://redis-py.readthedocs.io/en/latest/>

- 使用范例：

```python
from django_redis import get_redis_connection

redis_conn = get_redis_connection("default") # 连接redis数据库
```



报错：django.core.exceptions.ImproperlyConfigured: mysqlclient 1.4.3 or newer is required; you have 1.0.3.

解决：

```python
import pymysql
pymysql.version_info = (1, 4, 13, "final", 0)

pymysql.install_as_MySQLdb()

```

## 九、提供一个测试接口例子

如果没有创建子应用home，可以根据以下步骤创建，终端操作

```bash
cd mingdeapi/apps
python ../../manage.py startapp home

mingde/mingdeapi on  dev via 🅒 mingde 
➜ cd mingdeapi/apps

mingde/mingdeapi/mingdeapi/apps on  dev via 🅒 mingde 
➜ python ../../manage.py startapp home    

mingde/mingdeapi/mingdeapi/apps on  dev [!?] via 🅒 mingde 
➜ 

```

- 注册home子应用分2步：

  - 在settings/dev.py中的INSTALLED_APPS配置项里面新增home

  - 因为项目目录的结构有所调整，所以我们在apps下保存了home子应用，因此需要让python能直接对home进行导包识别

Django REST framework官网：<https://www.django-rest-framework.org/>

[Django REST framework 中文文档](https://q1mi.github.io/Django-REST-framework-documentation/#django-rest-framework)

<https://q1mi.github.io/Django-REST-framework-documentation/>

`settings/dev.py`，代码：

```bash
import sys
from pathlib import Path

# 当前项目的主应用开发目录
# Build paths inside the project like this: BASE_DIR / 'subdir'.
BASE_DIR = Path(__file__).resolve().parent.parent
# 新增apps作为导包路径，导包路径默认保存sys.path属性中，所有的python的import或者from导包语句默认都是从sys.path中记录的路径下查找模块
sys.path.insert(0, str(BASE_DIR / "apps"))

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = True

ALLOWED_HOSTS = []

# Application definition

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'home',
]
```

home子应用下创建路由文件，home/urls.py，代码：

```python
from django.urls import path
from . import views
urlpatterns = [
    
]
```

把子路由文件注册到总路由，`mingdeapi/urls.py`，代码：

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('home/', include("home.urls")),
]
```

home/views.py，编写视图，代码：

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

from django_redis import get_redis_connection


# Create your views here.
class HomeAPIView(APIView):
    def get(self, request):
        """
        测试接口
        :param request:
        :return:
        """
        print("Home")

        # 直接操作 Redis
        redis = get_redis_connection("session")
        # lrange brother 0 -1
        brother = redis.lrange("brother", 0, -1)
        print(brother)

        return Response(brother, status.HTTP_200_OK)
```

home/urls.py，代码：

```python
from django.urls import path
from . import views

urlpatterns = [
    path("demo/", views.HomeAPIView.as_view()),
]
```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304181157750.png)

## 十、自动生成接口文档配置（drf-yasg、Core API）

<https://drf-yasg.readthedocs.io/en/stable/index.html>

<https://pypi.org/project/drf-yasg2/>

安装

```bash
pip install drf_yasg
pip3 install coreapi
```

`setting.py` 文件中添加

```python
INSTALLED_APPS = [
    ...
    'drf_yasg',
    ...
]

# drf配置
REST_FRAMEWORK = {
    'DEFAULT_SCHEMA_CLASS': 'rest_framework.schemas.coreapi.AutoSchema'
}
```

`mingdeapi/urls.py` 代码：

```python
from django.contrib import admin
from django.urls import path, include, re_path

from drf_yasg import openapi
from drf_yasg.views import get_schema_view
from rest_framework.documentation import include_docs_urls

schema_view = get_schema_view(
    openapi.Info(
        title="接口平台API",
        default_version='v1.0',
        description="接口平台接口文档",
        terms_of_service="https://www.google.com/policies/terms/",
        contact=openapi.Contact(email="测试"),
        license=openapi.License(name="BSD License"),
    ),
    public=True,
)

urlpatterns = [
    path('admin/', admin.site.urls),
    path('home/', include('home.urls')),

    path('docs/', include_docs_urls(title="接口测试平台API文档", description="这个是接口平台的文档")),

    re_path(r'^swagger(?P<format>\.json|\.yaml)$', schema_view.without_ui(cache_timeout=0), name='schema-json'),
    path('doc', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'),
    path('redoc/', schema_view.with_ui('redoc', cache_timeout=0), name='schema-redoc'),
]

```

运行

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304181247317.png)



![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304181247298.png)



![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304181249515.png)
