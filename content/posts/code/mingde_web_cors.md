---
title: "明德项目之CORS跨域支持"
date: 2023-06-25T16:13:48+08:00
draft: false
tags: ["Python", "项目"]
categories: ["Python", "项目"]
---

# 明德项目之CORS跨域支持

## CORS跨域支持

在开发中一般前端和后端往往处于不同的两个域名地址下，为了模拟真实环境，我们一般也会为前端和后端分别设置两个不同的本地域名进行模拟。

| 位置 | 域名            |
| ---- | --------------- |
| 前端 | `www.mingde.cn` |
| 后端 | `api.mingde.cn` |

vim编辑`/etc/hosts`文件，可以设置本地域名

```bash
sudo vim /etc/hosts
```

在文件中增加两条信息，并保存文件内容。

```shell
127.0.0.1   localhost
127.0.0.1   api.mingde.cn
127.0.0.1   www.mingde.cn
```

使用:wq保存文件。

```bash
:wq
```

在使用 PyCharm 运行web客户端项目时，默认以www.mingde.cn:3000启动项目。

现在前端与后端分处不同的域名，因为客户端访问不同源的服务端时会遭到浏览器的同源策略的拦截，所以我们需要配置CORS，一般开发中配置CORS有两种方案：

1. web客户端的vue项目中配置vue.config.js实现跨域（使用vite搭建的vue项目，则配置文件是vite.config.js）
2. api服务端的django项目中配置cors实现跨域

两种方式中可以任选其一，工作中我们与前端开发人员商量解决。

### 配置vue-cli/vite本身内置的nodejs来实现跨域代理

vite.config.ts 代码：

```tsx
import {defineConfig} from 'vite'
import vue from '@vitejs/plugin-vue'
import Components from 'unplugin-vue-components/vite'
import {ElementPlusResolver} from 'unplugin-vue-components/resolvers'

// https://vitejs.dev/config/
export default defineConfig({
    plugins: [
        vue(),
        Components({
            resolvers: [ElementPlusResolver()],
        }),
    ],
    server: {
        port: 3000,           // 客户端的运行端口，此处也可以绑定vue运行的端口，当然也可以写在pycharm下
        host: 'www.mingde.cn', // 客户端的运行地址，此处也可以绑定vue运行的域名，当然也可以写在pycharm下
        // 跨域代理
        proxy: {
            '/api': {
                // 凡是遇到 /api 路径的请求，都映射到 target 属性  /api/header/  ---> http://api.mingde.cn:8000/header/
                target: 'http://api.mingde.cn:8000/',
                changeOrigin: true,
                ws: true,    // 是否支持websocket跨域
                rewrite: path => path.replace(/^\/api/, '')
            }
        }
    }
})

```

### Django安装跨域组件实现CORS代理

使用api服务端配置，则先屏蔽掉前端CORS配置  vite.config.ts 代码：

```ts
import {defineConfig} from 'vite'
import vue from '@vitejs/plugin-vue'
import Components from 'unplugin-vue-components/vite'
import {ElementPlusResolver} from 'unplugin-vue-components/resolvers'

// https://vitejs.dev/config/
export default defineConfig({
    plugins: [
        vue(),
        Components({
            resolvers: [ElementPlusResolver()],
        }),
    ],
    // server: {
    //     port: 3000,           // 客户端的运行端口，此处也可以绑定vue运行的端口，当然也可以写在pycharm下
    //     host: 'www.mingde.cn', // 客户端的运行地址，此处也可以绑定vue运行的域名，当然也可以写在pycharm下
    //     // 跨域代理
    //     proxy: {
    //         '/api': {
    //             // 凡是遇到 /api 路径的请求，都映射到 target 属性  /api/header/  ---> http://api.mingde.cn:8000/header/
    //             target: 'http://api.mingde.cn:8000/',
    //             changeOrigin: true,
    //             ws: true,    // 是否支持websocket跨域
    //             rewrite: path => path.replace(/^\/api/, '')
    //         }
    //     }
    // }
})

```

安装跨域组件

```python
pip install django-cors-headers
```

文档：<https://github.com/ottoyiu/django-cors-headers/>

添加子应用，settings/dev.py，代码：

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'rest_framework',
    'corsheaders',  # cors跨域子应用
    'drf_yasg',

    'home', # apps已经在上面加入到python的系统导包路径列表了，这里还是出现背景颜色，原因是python虽然找到home，pycharm不知道。所以可以鼠标右键，设置apps为 Mark Directory as -> Sources Root
]
```

中间件注册【必须写在第一个位置】，settings/dev.py，代码：

```python
MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',  # cors跨域的中间件
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```

添加客户端访问服务端的白名单，设置允许哪些客户端客户端跨域访问服务端。settings/dev.py 代码：

```python
# CORS（跨域资源共享）的配置信息:
# 方案1：
# CORS_ORIGIN_WHITELIST = (
#     'http://www.mingde.cn:3000',
# )
# CORS_ALLOW_CREDENTIALS = False  # 不允许ajax跨域请求时携带cookie

# 方案2：
CORS_ALLOW_ALL_ORIGINS = True

```

完成了上面的步骤，我们现在就可以通过后端提供数据给前端使用ajax访问了。
