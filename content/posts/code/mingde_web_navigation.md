---
title: "明德项目之导航功能实现"
date: 2023-06-26T10:49:27+08:00
draft: false
tags: ["Python", "项目"]
categories: ["Python", "项目"]
---

# 明德项目之导航功能实现

导航数据的存储，必须要找出导航数据的结构，也就是有哪些属性？

导航位置、导航名称、导航链接、导航序号、是否显示、是否外链、添加时间、更新时间、是否删除

### 创建模型

公共模型【抽象模型，不会在数据迁移的时候为它创建表】，保存项目的公共代码库目录下mingdeapi/utils.py文件中。

`mingdeapi.utils.models`代码：

```python
from django.db import models


class BaseModel(models.Model):
    """
    公共模型
    保存项目中的所有模型的公共属性和公共方法的声明
    """
    name = models.CharField(max_length=255, verbose_name="名称/标题")
    orders = models.IntegerField(default=0, verbose_name="显示顺序")
    is_show = models.BooleanField(default=True, verbose_name="是否显示")
    # auto_now_add=True 当数据被创建时，以当前时间作为默认值写入
    created_time = models.DateTimeField(auto_now_add=True, verbose_name="创建时间")
    # auto_now=True 当数据被更新时，以当前时间作为值写入
    updated_time = models.DateTimeField(auto_now=True, verbose_name="更新时间")
    is_delete = models.BooleanField(default=False, verbose_name="逻辑删除")

    class Meta:
        # 设置当前模型类并非真正的模型，而是一种保存公共代码的抽象模型类
        # 这种模型在数据迁移中不会被当做数据模型来创建数据表
        abstract = True

    def __str__(self):
        return self.name

```

home/models.py 代码：

```python
from mingdeapi.utils.models import BaseModel, models


# Create your models here.

class Nav(BaseModel):
    """导航菜单"""
    # 字段选项
    # 模型对象.<字段名>  ---> 实际数据
    # 模型对象.get_<字段名>_display()  --> 文本提示
    POSITION_CHOICES = (
        # (实际数据, "文本提示"),
        (0, "顶部导航"),
        (1, "脚部导航"),
    )

    link = models.CharField(max_length=255, verbose_name="导航连接")
    is_http = models.BooleanField(default=False, verbose_name="是否站外连接地址")
    position = models.SmallIntegerField(default=0, choices=POSITION_CHOICES, verbose_name="导航位置")

    class Meta:
        db_table = "md_nav"
        verbose_name = "导航菜单"
        verbose_name_plural = verbose_name

```

settings/dev.py 中配置导包路径 代码：

```python
# 因为项目中子应用已经换了存储目录，所以需要把apps设置为系统导包路径，方便我们后面开发时可以简写子应用相关的导包路径。
import sys
sys.path.insert(0, str( BASE_DIR / "apps") )
sys.path.insert(0, str( BASE_DIR / "utils") )
```

home/models.py，代码调整如下：

```python
from models import BaseModel, models


# Create your models here.


class Nav(BaseModel):
    """导航菜单"""
    # 字段选项
    # 模型对象.<字段名>  ---> 实际数据
    # 模型对象.get_<字段名>_display()  --> 文本提示
    POSITION_CHOICES = (
        # (实际数据, "文本提示"),
        (0, "顶部导航"),
        (1, "脚部导航"),
    )

    link = models.CharField(max_length=255, verbose_name="导航连接")
    is_http = models.BooleanField(default=False, verbose_name="是否站外连接地址")
    position = models.SmallIntegerField(default=0, choices=POSITION_CHOICES, verbose_name="导航位置")

    class Meta:
        db_table = "md_nav"
        verbose_name = "导航菜单"
        verbose_name_plural = verbose_name

```

这里PyCharm中`from models import BaseModel, models`会出现红色警告，设置utils为 Mark Directory as -> Sources Root 即可解决此问题。

数据迁移【实际工作中，不一定需要我们数据迁移，公司有DBA的，那就直接DBA已经提前设计数据表了，我们声明模型，保证模型的表和字段与数据库中的对应上，就可以操作数据了。】

```bash
cd mingdeapi/
python manage.py makemigrations
python manage.py migrate
```

运行

```bash
mingde on  main [!?] via 🅒 mingde 
➜ cd mingdeapi/

mingde/mingdeapi on  main [!?] via 🅒 mingde 
➜ python manage.py makemigrations
Migrations for 'home':
  mingdeapi/apps/home/migrations/0001_initial.py
    - Create model Nav

mingde/mingdeapi on  main [!?] via 🅒 mingde 
➜ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, home, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying home.0001_initial... OK
  Applying sessions.0001_initial... OK

mingde/mingdeapi on  main [!?] via 🅒 mingde 

```

添加测试数据

```sql
INSERT INTO mingde.md_nav (id, name, orders, is_show, is_delete, created_time, updated_time, link, is_http, position) VALUES (1, '免费课', 1, 1, 0, '2023-06-15 01:27:27.350000', '2023-06-15 01:27:28.690000', '/free', 0, 0);
INSERT INTO mingde.md_nav (id, name, orders, is_show, is_delete, created_time, updated_time, link, is_http, position) VALUES (2, '项目课', 1, 1, 0, '2023-06-15 01:27:27.350000', '2023-06-15 01:27:28.690000', '/project', 0, 0);
INSERT INTO mingde.md_nav (id, name, orders, is_show, is_delete, created_time, updated_time, link, is_http, position) VALUES (3, '学位课', 1, 1, 0, '2023-06-15 01:27:27.350000', '2023-06-15 01:27:28.690000', '/position', 0, 0);
INSERT INTO mingde.md_nav (id, name, orders, is_show, is_delete, created_time, updated_time, link, is_http, position) VALUES (4, '习题库', 1, 1, 0, '2023-06-15 01:27:27.350000', '2023-06-15 01:27:28.690000', '/exam', 0, 0);
INSERT INTO mingde.md_nav (id, name, orders, is_show, is_delete, created_time, updated_time, link, is_http, position) VALUES (5, '明德教育', 1, 1, 0, '2023-06-15 01:27:27.350000', '2023-06-15 01:27:28.690000', 'https://www.icourse163.org/', 1, 0);
INSERT INTO mingde.md_nav (id, name, orders, is_show, is_delete, created_time, updated_time, link, is_http, position) VALUES (6, '企业服务', 1, 1, 0, '2023-06-15 01:27:27.350000', '2023-06-15 01:27:28.690000', '/free', 0, 1);
INSERT INTO mingde.md_nav (id, name, orders, is_show, is_delete, created_time, updated_time, link, is_http, position) VALUES (7, '关于我们', 1, 1, 0, '2023-06-15 01:27:27.350000', '2023-06-15 01:27:28.690000', '/free', 0, 1);
INSERT INTO mingde.md_nav (id, name, orders, is_show, is_delete, created_time, updated_time, link, is_http, position) VALUES (8, '联系我们', 1, 1, 0, '2023-06-15 01:27:27.350000', '2023-06-15 01:27:28.690000', '/free', 0, 1);
INSERT INTO mingde.md_nav (id, name, orders, is_show, is_delete, created_time, updated_time, link, is_http, position) VALUES (9, '商务合作', 1, 1, 0, '2023-06-15 01:27:27.350000', '2023-06-15 01:27:28.690000', '/free', 0, 1);
INSERT INTO mingde.md_nav (id, name, orders, is_show, is_delete, created_time, updated_time, link, is_http, position) VALUES (10, '帮助中心', 1, 1, 0, '2023-06-15 01:27:27.350000', '2023-06-15 01:27:28.690000', '/free', 0, 1);
INSERT INTO mingde.md_nav (id, name, orders, is_show, is_delete, created_time, updated_time, link, is_http, position) VALUES (11, '意见反馈', 1, 1, 0, '2023-06-15 01:27:27.350000', '2023-06-15 01:27:28.690000', '/free', 0, 1);
INSERT INTO mingde.md_nav (id, name, orders, is_show, is_delete, created_time, updated_time, link, is_http, position) VALUES (12, '新手指南', 1, 1, 0, '2023-06-15 01:27:27.350000', '2023-06-15 01:27:28.690000', '/free', 0, 1);
```

### 序列化器

home.serializers，代码：

```python
from rest_framework import serializers
from .models import Nav


class NavModelSerializer(serializers.ModelSerializer):
    """
    导航菜单的序列化器
    """
    class Meta:
        model = Nav
        fields = ["name", "link", "is_http"]

```

### 视图代码

home/views.py

```python
import constants
from rest_framework.generics import ListAPIView
from .models import Nav
from .serializers import NavModelSerializer


class HeaderNavListAPIView(ListAPIView):
    """
    头部导航
    """
    queryset = Nav.objects.filter(is_delete=False, is_show=True, position=constants.NAV_HEADER).order_by("orders",
                                                                                                         "-id")[
               :constants.NAV_HEADER_SIZE]
    serializer_class = NavModelSerializer


class FooterNavListAPIView(ListAPIView):
    """
    脚部导航
    """
    queryset = Nav.objects.filter(is_delete=False, is_show=True, position=constants.NAV_FOOTER).order_by("orders",
                                                                                                         "-id")[
               :constants.NAV_FOOTER_SIZE]
    serializer_class = NavModelSerializer

```

#### 常量配置

utils.constants，代码：

```python
"""常量配置文件"""
# 导航位置->头部
NAV_HEADER = 0
# 导航位置->脚部
NAV_FOOTER = 1
# 头部导航的显示数量
NAV_HEADER_SIZE = 7
# 脚部导航的显示数量
NAV_FOOTER_SIZE = 7

```

### 路由代码

home.urls.py

```python
from django.urls import path
from . import views

urlpatterns = [
    path("nav/header/", views.HeaderNavListAPIView.as_view()),
    path("nav/footer/", views.FooterNavListAPIView.as_view()),
]

```

运行并访问：

头部导航

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306261407181.png)

脚步导航

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306261410263.png)

提交代码版本

```bash
mingde/mingdeapi on  main [!?] via 🅒 mingde 
➜ cd ..        

mingde on  main [!?] via 🅒 mingde 
➜ git add .

mingde on  main [+] via 🅒 mingde 
➜ git commit -m "服务端提供导航api接口"
[main cb29a4e] 服务端提供导航api接口
 12 files changed, 208 insertions(+), 23 deletions(-)
 create mode 100644 mingdeapi/mingdeapi/apps/home/migrations/0001_initial.py
 create mode 100644 mingdeapi/mingdeapi/apps/home/serializers.py
 create mode 100644 mingdeapi/mingdeapi/utils/constants.py
 create mode 100644 mingdeapi/mingdeapi/utils/models.py

mingde on  main [⇡] via 🅒 mingde 
➜ git push                             
枚举对象中: 47, 完成.
对象计数中: 100% (47/47), 完成.
使用 12 个线程进行压缩
压缩对象中: 100% (23/23), 完成.
写入对象中: 100% (26/26), 6.03 KiB | 6.03 MiB/s, 完成.
总共 26（差异 8），复用 0（差异 0），包复用 0
remote: Resolving deltas: 100% (8/8), completed with 8 local objects.
To github.com:qiaopengjun5162/mingde.git
   65eba34..cb29a4e  main -> main

mingde on  main via 🅒 mingde took 4.4s 

```

### 客户端获取导航数据

所有与api服务端进行交互的操作代码，可以单独保存api目录下，根据数据表单独创建js文件，方便将来代码复用。

src/api/nav.ts，代码：

```tsx
import http from "../utils/http";
import {reactive} from "vue"

const nav = reactive({
    header_nav_list: [], // 头部导航列表
    footer_nav_list: [], // 脚部导航列表
    get_header_nav() {
        // 获取头部导航
        return http.get("/nav/header/")
    },
    get_footer_nav() {
        // 获取脚部导航
        return http.get("/nav/footer/")
    }
})

export default nav;
```

Header.vue代码：

```vue
<template>
  <div class="header-box">
    <div class="header">
      <div class="content">
        <div class="logo">
          <router-link to="/"><img src="../assets/logo.png" alt=""></router-link>
        </div>
        <ul class="nav">
          <li v-for="item in nav.header_nav_list">
            <a v-if="item.is_http" :href="item.link">{{ item.name }}</a>
            <router-link v-else :to="item.link">{{ item.name }}</router-link>
          </li>
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
import nav from "../api/nav";

nav.get_header_nav().then(response => {
  nav.header_nav_list = response.data;
  console.log("11", nav.header_nav_list);
});

</script>
```

Footer.vue代码：

```vue
<template>
  <div class="footer">
    <ul>
      <li v-for="item in nav.footer_nav_list">
        <a v-if="item.is_http" :href="item.link">{{ item.name }}</a>
        <router-link v-else :to="item.link">{{ item.name }}</router-link>
      </li>
    </ul>
    <p>Copyright © mingde.com版权所有 | 京ICP备17072161号-1</p>
  </div>
</template>

<script setup lang="ts">
import nav from "../api/nav";

nav.get_footer_nav().then((response: { data: any }) => {
  nav.footer_nav_list = response.data;
});

</script>

```

提交代码

```bash
mingde on  main [✘!?] via 🅒 mingde 
➜ git add .

mingde on  main [✘+] via 🅒 mingde 
➜ git commit -m "客户端实现导航信息展示"
[main 3d5e614] 客户端实现导航信息展示
 12 files changed, 156 insertions(+), 68 deletions(-)
 create mode 100644 mingdeweb/src/api/nav.ts
 delete mode 100755 mingdeweb/src/assets/logo.png
 create mode 100644 mingdeweb/src/assets/logo.svg

mingde on  main [⇡] via 🅒 mingde 
➜ gp                                    
枚举对象中: 49, 完成.
对象计数中: 100% (49/49), 完成.
使用 12 个线程进行压缩
压缩对象中: 100% (21/21), 完成.
写入对象中: 100% (27/27), 6.82 KiB | 3.41 MiB/s, 完成.
总共 27（差异 11），复用 0（差异 0），包复用 0
remote: Resolving deltas: 100% (11/11), completed with 11 local objects.
To github.com:qiaopengjun5162/mingde.git
   cb29a4e..3d5e614  main -> main

mingde on  main via 🅒 mingde took 4.2s 
```
