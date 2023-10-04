---
title: "Go Web 编程快速入门"
date: 2023-05-11T20:57:37+08:00
draft: true
tags: ["Go"]
categories: ["Go"]
---

# Go Web 编程快速入门

## 使用 Go 创建 Web 应用

### 课程主要内容

- 处理请求
- 模板
- 中间件
- 存储数据
-  HTTPS ， HTTP2
-  测试
-  部署

## 一、课程准备

### 前提条件

- 会使用 Go 语言
- 会基本的 HTML 、 CSS 编程

### 工具

- 安装 Go  <https://go.dev/>
- IDE ：
  - Visual Studio Code
    -  Go 扩展
    - REST Client 扩展
  - 或 Goland （收费）
- 数据库：
  - Microsoft SQL Server 或 PostgreSQL ...

### 一个例子

```bash
~/Code/go via 🐹 v1.20.3 via 🅒 base
➜ mcd web-tutorial

Code/go/web-tutorial via 🐹 v1.20.3 via 🅒 base
➜ go mod init github.com/qiaopengjun5162/web-tutorial
go: creating new go.mod: module github.com/qiaopengjun5162/web-tutorial

Code/go/web-tutorial via 🐹 v1.20.3 via 🅒 base
➜ c

Code/go/web-tutorial via 🐹 v1.20.3 via 🅒 base
➜

```

main.go 文件

```go
package main

import "net/http"

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Hello world!"))
	})

	http.ListenAndServe("localhost:8080", nil) // DefaultServeMux
}

```

运行

```bash
Code/go/web-tutorial via 🐹 v1.20.3 via 🅒 base 
➜ go run main.go                                     

```

访问：<http://localhost:8080/>

![image-20230511213657345](../../../Library/Application Support/typora-user-images/image-20230511213657345.png)



## 二、处理（ Handle ）请求

### 本节内容

- 如何处理（ Handle ） Web 请求
  -  http.Handle 函数
  - http.HandleFunc 函数

### 处理请求

HTTP 请求  Handler  goroutine  http.DefaulServerMux

- HTTP 请求
- Handler
- goroutine
- http.DefaulServeMux

### 创建 Web Server

- http.ListenAndServer()
  - 第一个参数是网络地址
    - 如果为“”，那么就是所有网络接口的 80 端口
  -  第二个参数是 handler
    - 如果为 nil ，那么就是 DefaultServeMux
- DefaultServeMux 是一个 multiplexer （可以看作是路由器）
-  http.Server 这是一个 struct
  -  Addr 字段表示网络地址
    - 如果为“”，那么就是所有网络接口的 80 端口
  - Handler 字段
    - 如果为 nil ，那么就是 DefaultServeMux
  -  ListenAndServe() 函数
- http.ListenAndServer()
  - 两个参数
- http.ListenAndServeTLS()



- http.Server 可配置
-  server.ListenAndServe()
- server.ListenAndServeTLS()

### Handler

- handler 是一个接口（ interface ）
- handler 定义了一个方法 ServeHTTP()
  - HTTPResponseWriter
  - 指向 Request 这个 struct 的指针

```go
type Handler interface {
  ServeHTTP(ResponseWriter, *Request)
}
```



### DefaultServeMux

- 它是一个 Multiplexer （多路复用器）
- 它也是一个 Handler

```go
package main

import "net/http"

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Hello world!"))
	})

	// http.ListenAndServe("localhost:8080", nil) // DefaultServeMux

	server := http.Server{
		Addr:    "localhost:8080",
		Handler: nil,
	}
	server.ListenAndServe()
}

```



HTTP 请求    ->    DefaultServeMux  ->    Handler 1  Handler 2  Handler 3  ... ...

```go
package main

import "net/http"

type myHandler struct{}

func (m *myHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello world!"))
}

func main() {
	mh := myHandler{}

	// http.ListenAndServe("localhost:8080", nil) // DefaultServeMux

	server := http.Server{
		Addr:    "localhost:8080",
		Handler: &mh,
	}
	server.ListenAndServe()
}

```



### 多个 Handler

HTTP 请求    ->    DefaultServeMux  ->  Handler 1 Handler 2 Handler 3 ... ...
HTTP 请求 ->  MyHandler

### 多个 Handler - http.Handle

- 不指定 Server struct 里面的 Handler 字段值
- 可以使用 http.Handle 将某个 Handler 附加到 DefaultServeMux
  -  http 包有一个 Handle 函数
  - ServerMux struct 也有一个 Handle 方法
- 如果你调用 http.Handle ，实际上调用的是 DefaultServeMux 上的 Handle 方法
  -  DefaultServeMux 就是 ServerMux 的指针变量

### http.Handle

- func Handle(pattern string, handler Handler)
- ` type Handler interface {ServeHTTP(ResponseWriter, *Request)}`

```go
package main

import "net/http"

type helloHandler struct{}

func (m *helloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello world!"))
}

type aboutHandler struct{}

func (m *aboutHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("About!")) //
}

func main() {
	mh := helloHandler{}
	a := aboutHandler{}

	// http.ListenAndServe("localhost:8080", nil) // DefaultServeMux

	server := http.Server{
		Addr:    "localhost:8080",
		Handler: nil, // DefaultServeMux
	}

	http.Handle("/hello", &mh)
	http.Handle("/about", &a)
	server.ListenAndServe()
}

```



### Handler 函数 - http.HandleFunc

- Handler 函数就是那些行为与 handler 类似的函数：
-  Handler 函数的签名与 ServeHTTP 方法的签名一样，接收：
  - 一个 http.ResponseWriter
  - 一个 指向 http.Request 的指针

### http.HandleFunc 原理

- Go 有一个函数类型： HandlerFunc 。可以将某个具有适当签名的函
数 f ，适配成为一个 Handler ，而这个 Handler 具有方法 f 。
- （例子）

```go
package main

import "net/http"

type helloHandler struct{}

func (m *helloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello world!"))
}

type aboutHandler struct{}

func (m *aboutHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("About!")) //
}

func welcome(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Welcome"))
}

func main() {
	mh := helloHandler{}
	a := aboutHandler{}

	// http.ListenAndServe("localhost:8080", nil) // DefaultServeMux

	server := http.Server{
		Addr:    "localhost:8080",
		Handler: nil, // DefaultServeMux
	}

	http.Handle("/hello", &mh)
	http.Handle("/about", &a)

	http.HandleFunc("/home", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Home!"))
	})

	http.HandleFunc("/wecome", welcome)
	server.ListenAndServe()
}

```



### http.HandleFunc

- `func HandleFunc(pattern string, handler func(ResponseWriter,*Request))`
- `type HandlerFunc func(ResponseWriter, *Request)`
- ` func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request)` 

```go
package main

import "net/http"

type helloHandler struct{}

func (m *helloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello world!"))
}

type aboutHandler struct{}

func (m *aboutHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("About!")) //
}

func welcome(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Welcome!"))
}

func main() {
	mh := helloHandler{}
	a := aboutHandler{}

	// http.ListenAndServe("localhost:8080", nil) // DefaultServeMux

	server := http.Server{
		Addr:    "localhost:8080",
		Handler: nil, // DefaultServeMux
	}

	http.Handle("/hello", &mh)
	http.Handle("/about", &a)

	http.HandleFunc("/home", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("Home!"))
	})

	// http.HandleFunc("/wecome", welcome)
	http.Handle("/wecome", http.HandlerFunc(welcome))

	server.ListenAndServe()
}

```



### 主要内容回顾

#### http.Handle

- 第二个参数是 Handler

#### http.HandleFunc

- 第二个参数是一个 Handler 函数
- http.HandlerFunc 可以把 Handler函数转化为 Handler
- 内部调用的还是 http.Handle 函数

## 三、内置的 Handlers

### 本节内容

-  NotFoundHandler
- RedirectHandler
-  StripPrefix
- TimeoutHandler
- FileServer

### http.NotFoundHandler

- `func NotFoundHandler() Handler`
- 返回一个 handler ，它给每个请求的响应都是“ 404 page not found”

### http.RedirectHandler

- `func RedirectHandler(url string, code int) Handler`
- 返回一个 handler ，它把每个请求使用给定的状态码跳转到指定的 URL 。
  -  url ，要跳转到的 URL
  - code ，跳转的状态码（ 3xx ），常见的：StatusMovedPermanently 、 StatusFound 或 StatusSeeOther 等

### http.StripPrefix

- `func StripPrefix(prefix string, h handler) Handler`
- 返回一个 handler ，它从请求 URL 中去掉指定的前缀，然后再调用另一个 handler 。
  -  如果请求的 URL 与提供的前缀不符，那么 404
- 略像中间件
  - prefix ， URL 将要被移除的字符串前缀
  - h ，是一个 handler ，在移除字符串前缀之后，这个 handler 将会接收到请求
- 修饰了另一个 Handler

### http.TimeoutHandler

-  `func TimeoutHandler(h Handler, dt time.Duration, msg string) Handler`
- 返回一个 handler ，它用来在指定时间内运行传入的 h 。
- 也相当于是一个修饰器
  - h ，将要被修饰的 handler
  - dt ，第一个 handler 允许的处理时间
  - msg ，如果超时，那么就把 msg 返回给请求，表示响应时间过长

### http.FileServer

- `func FileServer(root FileSystem) Handler`
- 返回一个 handler ，使用基于 root 的文件系统来响应请求
- ` type FileSystem interface {Open(name string) (File, error)}`

- 使用时需要用到操作系统的文件系统，所以还需要委托给：
- `type Dir string`
- `func (d Dir) Open(name string) (File, error)`

```go
package main

import "net/http"

func main() {
	// http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
	// 	http.ServeFile(w, r, "wwwroot"+r.URL.Path)
	// })
	// http.ListenAndServe(":8080", nil)

	http.ListenAndServe(":8080", http.FileServer(http.Dir("wwwroot")))
}

```

## 四、请求（ Request ）

### 本节内容

- HTTP 请求
-  Request
-  URL
-  Header
- Body

### HTTP 消息

- HTTP Request 和 HTTP Response （请求和响应）
- 它们具有相同的结构：
  - 请求（响应）行
  - 0 个或多个 Header
  - 空行
  - 可选的消息体（ Body ）
-  例子：
  GET /Protocols/rfc2616/rfc2616.html HTTP/1.1
  Host: www.w3.org
  User-Agent: Mozilla/5.0
  ( 空行 )

- net/http 包提供了用于表示 HTTP 消息的结构

### 请求（ Request ）

- Reqeust （是个 struct ），代表了客户端发送的 HTTP 请求消息
- 重要的字段：
  - URL
  - Header
  -  Body
  -  Form 、 PostForm 、 MultipartForm
- 也可以通过 Request 的方法访问请求中的 Cookie 、 URL 、 UserAgent 等信息
- Request 即可代表发送到服务器的请求，又可代表客户端发出的请求

### 请求的 URL

- Request 的 URL 字段就代表了请求行（请求信息第一行）里面的部分内容
- URL 字段是指向 url.URL 类型的一个指针， url.URL 是一个 struct ：

```go
type URL struct {
  Scheme string
  Opaque string
  User *Userinfo
  Host string
  Path string
  RawQuery string
  Fragment string
}
```



### URL 的通用形式

- 通用格式是： `scheme://[userinfo@]host/path[?query][#fragment]`
- 不可以斜杠开头的 URL 被解释为：` scheme:opaque[?query][#fragment]`

### URL Query

- RawQuery 会提供实际查询的字符串。
-  例如： http://www.example.com/post?id=123&thread_id=456
  -  它的 RawQuery 的值就是 id=123&thread_id=456
- 还有一个简便方法可以得到 Key-Value 对：通过 Request 的 Form 字段（以后再说）

### URL Fragment

- `http://www.httpwatch.com/features.htm#print`  #print

- 如果从浏览器发出的请求，那么你无法提取出 Fragment 字段的值
  - 浏览器在发送请求时会把 fragment 部分去掉
- 但不是所有的请求都是从浏览器发出的（例如从 HTTP 客户端包）

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/url", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, r.URL.Fragment)
	})

	server.ListenAndServe()
}

```

访问：<http://localhost:8080/url#fragement>

### Request Header

- 请求和响应（ Request 、 Response ）的 headers 是通过 Header 类型来描述的，它是一个 map ，用来表述 HTTP Header 里的 Key-Value 对。
-  Header map 的 key 是 string 类型， value 是 []string
- 设置 key 的时候会创建一个空的 []string 作为 value ， value 里面第一个元素就是新 header 的值；
- 为指定的 key 添加一个新的 header 值，执行 append 操作即可

### Request Header 例子

- r.Header
  - 返回 map
- r.Header["Accept-Encoding"]
  - 返回： [gzip, deflate] （ []string 类型）
- r.Header.Get("Accept-Encoding")
  - 返回： gzip, deflate （ string 类型）

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/header", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, r.Header)
		fmt.Fprintln(w, r.Header["Accept-Encoding"])
		fmt.Fprintln(w, r.Header.Get("Accept-Encoding"))
	})

	server.ListenAndServe()
}

```

请求

```http
### header test
GET http://localhost:8080/header HTTP/1.1

```

响应

```http
HTTP/1.1 200 OK
Date: Sat, 13 May 2023 02:13:38 GMT
Content-Length: 117
Content-Type: text/plain; charset=utf-8
Connection: close

map[Accept-Encoding:[gzip, deflate] Connection:[close] User-Agent:[vscode-restclient]]
[gzip, deflate]
gzip, deflate

```



### Request Body

- 请求和响应的 bodies 都是使用 Body 字段来表示的
- Body 是一个 io.ReadCloser 接口
  - 一个 Reader 接口
  -  一个 Closer 接口
- Reader 接口定义了一个 Open 方法：
  - 参数： []byte
  -  返回： byte 的数量、可选的错误
- Closer 接口定义了一个 Close 方法：
  -  没有参数，返回可选的错误
- 想要读取请求 body 的内容，可以调用 Body 的 Read 方法

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/post", func(w http.ResponseWriter, r *http.Request) {
		length := r.ContentLength
		body := make([]byte, length)
		r.Body.Read(body)
		fmt.Fprintln(w, string(body))
	})

	server.ListenAndServe()
}

```

请求

```http
### POST test
POST http://localhost:8080/post HTTP/1.1
Content-Type: application/json

{
    "name": "sample",
    "time": "Wed, 20 May 2023 18:27:40 GMT"
}

```

响应

```http
HTTP/1.1 200 OK
Date: Sat, 13 May 2023 02:23:03 GMT
Content-Length: 70
Content-Type: text/plain; charset=utf-8
Connection: close

{
  "name": "sample",
  "time": "Wed, 20 May 2023 18:27:40 GMT"
}
```

### 查询参数（ Query Parameters ）

### URL Query

- http://www.example.com/post?id=123&thread_id=456
- r.URL.RawQuery 会提供实际查询的原始字符串。
- 上例的 RawQuery 的值就是 id=123&thread_id=456
- r.URL.Query() ，会提供查询字符串对应的 `map[string][]string`

```go
url := r.URL

query := url.Query() // map[string][]string

id := query["id"]  // []string["123"]

threadID := query.Get("thread_id") // "456"
```

main.go 文件

```go
package main

import (
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/home", func(w http.ResponseWriter, r *http.Request) {
		url := r.URL
		query := url.Query()

		id := query["id"]
		log.Println(id)

		name := query.Get("name")
		log.Println(name) // 只返回第一个值
	})

	http.ListenAndServe("localhost:8080", nil)
}

```

访问：http://localhost:8080/home?id=1&name=xiaoqiao&id=123&name=nick

响应

```bash
Code/go/web-tutorial via 🐹 v1.20.3 via 🅒 base took 1h 34m 24.7s 
➜ go run main.go
2023/05/13 11:58:53 [1 123]
2023/05/13 11:58:53 xiaoqiao


```



## 五、Forms

### 本节内容

- 通过表单发送请求
-  Form 字段
-  PostForm 字段
-  MultipartForm 字段
-  FormValue & PostFormValue 方法
-  文件（ Files ）
-  POST JSON

### 来自表单的 Post 请求

```html
<form action="/process" method="post">
  <input type="text" name="first_name"/>
  <input type="text" name="last_name"/>
  <input type="submit"/>
</form>
```



-  这个 HTML 表单里面的数据会以 name-value 对的形式，通过POST 请求发送出去。
-  它的数据内容会放在 POST 请求的 Body 里面
-  但 name-value 对在 Body 里面的格式是什么样的？

### 表单 Post 请求的数据格式

-  通过 POST 发送的 name-value 数据对的格式可以通过表单的Content Type 来指定，也就是 enctype 属性：

```html
<form action="/process" method="post" enctype="application/x-www-form-urlencoded">
  <input type="text" name="first_name"/>
  <input type="text" name="last_name"/>
  <input type="submit"/>
</form>
```



### 表单的 enctype 属性

-  默认值是： application/x-www-form-urlencoded
-  浏览器被要求至少要支持： application/x-www-form-urlencoded 、 multipart/form-data
  -  HTML 5 的话，还需要支持 text/plain
-  如果 enctype 是 application/x-www-form-urlencoded ，那么浏览器会将表单数据编码到查询字符串里面。例如：
  -  first_name=sau%20sheong&last_name=chang
- 如果 enctype 是 multipart/form-data ，那么
  -  每一个 name-value 对都会被转换为一个 MIME 消息部分
  -  每一个部分都有自己的 Content Type 和 Content Disposition

### 如何选择？

-  简单文本：表单 URL 编码
-  大量数据，例如上传文件： multipart-MIME
  -  甚至可以把二进制数据通过选择 Base64 编码，来当作文本进行发送

### 表单的 GET

-  通过表单的 method 属性，可以设置 POST 还是 GET

```html
<form action="/process" method="get">
  <input type="text" name="first_name"/>
  <input type="text" name="last_name"/>
  <input type="submit"/>
</form>
```



-  GET 请求没有 Body ，所有的数据都通过 URL 的 name-value 对来发送

### Form 字段

-  Request 上的函数允许我们从 URL 或 / 和 Body 中提取数据，通过这些字段：
  -  Form
  -  PostForm
  -  MultipartForm
-  Form 里面的数据是 key-value 对。
-  通常的做法是：
  -  先调用 ParseForm 或 ParseMultipartForm 来解析 Request
  -  然后相应的访问 Form 、 PostForm 或 MultipartForm 字段

index.html 文件

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Form</title>
</head>

<body>
    <form action="http://localhost:8080/process" method="post" enctype="application/x-www-form-urlencoded">
        <input type="text" name="first_name" />
        <input type="text" name="last_name" />
        <input type="submit" />
    </form>
</body>

</html>

```

main.go 文件

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/process", func(w http.ResponseWriter, r *http.Request) {
		r.ParseForm()
		fmt.Fprintln(w, r.Form)
	})

	server.ListenAndServe()
}

```

打开 index.html 文件

![image-20230513123008395](../../../Library/Application Support/typora-user-images/image-20230513123008395.png)

填写数据

![image-20230513123142535](../../../Library/Application Support/typora-user-images/image-20230513123142535.png)

提交

![image-20230513123224814](../../../Library/Application Support/typora-user-images/image-20230513123224814.png)

### PostForm 字段

-  前例中，如果只想得到 first_name 这个 Key 的 Value ，可使用 r.Form[“first_name”] ，它返回含有一个元素的 slice ： [“Dave”]
-  如果表单和 URL 里有同样的 Key ，那么它们都会放在一个 slice里：表单里的值靠前， URL 的值靠后
-  如果只想要表单的 key-value 对，不要 URL 的，可以使用PostForm 字段。
-  PostForm 只支持 application/x-www-form-urlencoded
-  想要得到 multipart key-value 对，必须使用 MultipartForm 字段

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Form</title>
</head>

<body>
    <form action="http://localhost:8080/process?first_name=Nick" method="post" enctype="application/x-www-form-urlencoded">
        <input type="text" name="first_name" />
        <input type="text" name="last_name" />
        <input type="submit" />
    </form>
</body>

</html>

```

![image-20230513123908418](../../../Library/Application Support/typora-user-images/image-20230513123908418.png)

只想要表单的字段

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/process", func(w http.ResponseWriter, r *http.Request) {
		r.ParseForm()
		// fmt.Fprintln(w, r.Form)
		fmt.Fprintln(w, r.PostForm)
	})

	server.ListenAndServe()
}

```



### MultipartForm 字段

-  想要使用 MultipartForm 这个字段的话，首先需要调用ParseMultipartForm 这个方法
  -  该方法会在必要时调用 ParseForm 方法
  -  参数是需要读取数据的长度
-  MultipartForm 只包含表单的 key-value 对
-  返回类型是一个 struct 而不是 map 。这个 struct 里有两个map ：
  - key 是 string ， value 是 []string
  - 空的（ key 是 string ， value 是文件）

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/process", func(w http.ResponseWriter, r *http.Request) {
		r.ParseMultipartForm(1024) // 1024 bytes 字节数
		fmt.Fprintln(w, r.MultipartForm)
	})

	server.ListenAndServe()
}

```

index.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Form</title>
</head>

<body>
    <form action="http://localhost:8080/process?first_name=Nick" method="post" enctype="multipart/form-data">
        <input type="text" name="first_name" />
        <input type="text" name="last_name" />
        <input type="submit" />
    </form>
</body>

</html>

```

![image-20230513140003291](../../../Library/Application Support/typora-user-images/image-20230513140003291.png)

### FormValue 和 PostFormValue 方法

- FormValue 方法会返回 Form 字段中指定 key 对应的第一个value
  -  无需调用 ParseForm 或 ParseMultipartForm
-  PostFormValue 方法也一样，但只能读取 PostForm
-  FormValue 和 PostFormValue 都会调用 ParseMultipartForm 方法
-  但如果表单的 enctype 设为 multipart/form-data ，那么即使你调用 ParseMultipartForm 方法，也无法通过 FormValue 获得想要的值。

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/process", func(w http.ResponseWriter, r *http.Request) {
		r.ParseMultipartForm(1024) // 1024 bytes 字节数
		// fmt.Fprintln(w, r.MultipartForm)
		fmt.Fprintln(w, r.FormValue("first_name")) // 获取的是 URL中的值
		fmt.Fprintln(w, r.PostFormValue("first_name"))
	})

	server.ListenAndServe()
}

```



### 上传文件

- multipart/form-data 最常见的应用场景就是上传文件
  -  首先调用 ParseMultipartForm 方法
  -  从 File 字段获得 FileHeader ，调用其 Open 方法来获得文件
  -  可以使用 ioutil.ReadAll 函数把文件内容读取到 byte 切片里

```go
package main

import (
	"fmt"
	"io"
	"net/http"
)

func process(w http.ResponseWriter, r *http.Request) {
	r.ParseMultipartForm(1024)

	fileHeader := r.MultipartForm.File["uploaded"][0]
	file, err := fileHeader.Open()
	if err == nil {
		data, err := io.ReadAll(file)
		if err == nil {
			fmt.Fprintln(w, string(data))
		}
	}
}

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/process", process)

	server.ListenAndServe()
}

```

index.html 文件

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Go Web Programming</title>
</head>

<body>
    <form action="http://localhost:8080/process?hello=world&thread=123" method="post" enctype="multipart/form-data">
        <input type="text" name="hello" value="sau sheong" />
        <input type="text" name="post" value="456" />
        <input type="file" name="uploaded">
        <input type="submit" />
    </form>
</body>

</html>

```



### FormFile 方法

- 上传文件还有一个简便方法： FormFile 
  -  无需调用 ParseMultipartForm 方法
  -  返回指定 key 对应的第一个 value
  -  同时返回 File 和 FileHeader ，以及错误信息
  -  如果只上传一个文件，那么这种方式会快一些

```go
package main

import (
	"fmt"
	"io"
	"net/http"
)

func process(w http.ResponseWriter, r *http.Request) {
	// r.ParseMultipartForm(1024)

	// fileHeader := r.MultipartForm.File["uploaded"][0]
	// file, err := fileHeader.Open()

	file, _, err := r.FormFile("uploaded")
	if err == nil {
		data, err := io.ReadAll(file)
		if err == nil {
			fmt.Fprintln(w, string(data))
		}
	}
}

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/process", process)

	server.ListenAndServe()
}

```



### POST 请求 - JSON Body

- 不是所有的 POST 请求都来自 Form
-  客户端框架（例如 Angular 等）会以不同的方式对 POST 请求编码：
  -  jQuery 通常使用 application/x-www-form-urlencoded
  -  Angular 是 application/json
  -  ParseForm 方法无法处理 application/json

###  Forms - MultipartReader()

### 读取 Form 的值

- Form
-  PostForm
-  FormValue()
-  PostFormValue()
-  FormFile()
-  MultipartReader()

### MultipartReader()

- `func (r *Request) MultipartReader() (*multipart.Reader, error)`
-  如果是 multipart/form-data 或 multipart 混合的 POST 请求：
  -  MultipartReader 返回一个 MIME multipart reader
  -  否则返回 nil 和一个错误
-  可以使用该函数代替 ParseMultipartForm 来把请求的 body 作为stream 进行处理
  -  不是把表单作为一个对象来处理的，不是一次性获得整个 map
-  逐个检查来自表单的值，然后每次处理一个

## 六、ResponseWriter

### ResponseWriter

- 从服务器向客户端返回响应需要使用 ResponseWriter
-  ResponseWriter 是一个接口， handler 用它来返回响应
-  真正支撑 ResponseWriter 的幕后 struct 是非导出的http.response

### 问题

- 为什么 Handler 的 ServeHTTP(w ResponseWriter, r *Request) ，只有一个是指针类型？而 w 是按值传递的吗？

都是指针，都是按引用进行传递的

### 写入到 ResponseWriter

- Write 方法接收一个 byte 切片作为参数，然后把它写入到 HTTP响应的 Body 里面。
-  如果在 Write 方法被调用时， header 里面没有设定 content type ，那么数据的前 512 字节就会被用来检测 content type

```go
package main

import (
	"net/http"
)

func writeExample(w http.ResponseWriter, r *http.Request) {
	str := `<html>
	<head><title>Go Web </title></head>
	<body><h1>Hello World</h1></body>
	</html>`
	w.Write([]byte(str))
}

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/write", writeExample)

	server.ListenAndServe()
}

```

```bash
Code/go/web-tutorial via 🐹 v1.20.3 via 🅒 base took 2.4s
➜ curl -i localhost:8080/write
HTTP/1.1 200 OK
Date: Sat, 13 May 2023 13:50:46 GMT
Content-Length: 87
Content-Type: text/html; charset=utf-8

<html>
	<head><title>Go Web </title></head>
	<body><h1>Hello World</h1></body>
	</html>%

Code/go/web-tutorial via 🐹 v1.20.3 via 🅒 base
```



### WriteHeader 方法

- WriteHeader 方法接收一个整数类型（ HTTP 状态码）作为参数，并把它作为 HTTP 响应的状态码返回
-  如果该方法没有显式调用，那么在第一次调用 Write 方法前，会隐式的调用 WriteHeader(http.StatusOK)
  -  所以 WriteHeader 主要用来发送错误类的 HTTP 状态码
-  调用完 WriteHeader 方法之后，仍然可以写入到ResponseWriter ，但无法再修改 header 了

```go
package main

import (
	"fmt"
	"net/http"
)

func writeExample(w http.ResponseWriter, r *http.Request) {
	str := `<html>
	<head><title>Go Web </title></head>
	<body><h1>Hello World</h1></body>
	</html>`
	w.Write([]byte(str))
}

func writeHeaderExample(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(501)
	fmt.Fprintln(w, "No such service, try next door")
}

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/write", writeExample)
	http.HandleFunc("/writeheader", writeHeaderExample)

	server.ListenAndServe()
}

```

```bash
Code/go/web-tutorial via 🐹 v1.20.3 via 🅒 base
➜ curl -i localhost:8080/writeheader
HTTP/1.1 501 Not Implemented
Date: Sat, 13 May 2023 13:55:53 GMT
Content-Length: 31
Content-Type: text/plain; charset=utf-8

No such service, try next door

Code/go/web-tutorial via 🐹 v1.20.3 via 🅒 base
➜
```



### Header 方法

- Header 方法返回 headers 的 map ，可以进行修改
-  修改后的 headers 将会体现在返回给客户端的 HTTP 响应里

```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
)

type Post struct {
	User    string
	Threads []string
}

func writeExample(w http.ResponseWriter, r *http.Request) {
	str := `<html>
	<head><title>Go Web </title></head>
	<body><h1>Hello World</h1></body>
	</html>`
	w.Write([]byte(str))
}

func writeHeaderExample(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(501)
	fmt.Fprintln(w, "No such service, try next door")
}

func headerExample(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Location", "http://google.com")
	w.WriteHeader(302)
}

func jsonExample(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	post := &Post{
		User:    "Sau Sheong",
		Threads: []string{"first", "second", "third"},
	}
	json, _ := json.Marshal(post)
	w.Write(json)
}

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/write", writeExample)
	http.HandleFunc("/writeheader", writeHeaderExample)
	http.HandleFunc("/redirect", headerExample)
	http.HandleFunc("/json", jsonExample)

	server.ListenAndServe()
}

```

执行

```bash
Code/go/web-tutorial via 🐹 v1.20.3 via 🅒 base
➜ curl -i localhost:8080/redirect
HTTP/1.1 302 Found
Location: http://google.com
Date: Sat, 13 May 2023 14:00:18 GMT
Content-Length: 0


Code/go/web-tutorial via 🐹 v1.20.3 via 🅒 base
➜ curl -i localhost:8080/json
HTTP/1.1 200 OK
Content-Type: application/json
Date: Sat, 13 May 2023 14:08:09 GMT
Content-Length: 58

{"User":"Sau Sheong","Threads":["first","second","third"]}%

Code/go/web-tutorial via 🐹 v1.20.3 via 🅒 base
➜
```

### 内置的 Response

-  NotFound 函数，包装一个 404 状态码和一个额外的信息

-  ServeFile 函数，从文件系统提供文件，返回给请求者
-  ServeContent 函数，它可以把实现了 io.ReadSeeker 接口的任何东西里面的内容返回给请求者
  -  还可以处理 Range 请求（范围请求），如果只请求了资源的一部分内容，那么 ServeContent 就可以如此响应。而 ServeFile 或 io.Copy 则不行。
-  Redirect 函数，告诉客户端重定向到另一个 URL

## 七、模板

### 模板部分的主要内容

- 简介
-  模板引擎
-  Action
-  参数、变量、管道
-  函数
-  模板组合

### 什么是模板？

- Web 模板就是预先设计好的 HTML 页面，它可以被模板引擎反复的使用，来产生 HTML 页面
-  Go 的标准库提供了 text/template ， html/template 两个模板库
  -  大多数 Go 的 Web 框架都使用这些库作为 默认的模板引擎

### 模板与模板引擎

- 模板引擎可以合并模板与上下文数据，产生最终的 HTML

模板 数据  模板引擎    ->    HTML

### 两种理想的模板引擎

| 无逻辑模板引擎                     | 逻辑嵌入模板引擎                         |
| ---------------------------------- | ---------------------------------------- |
| 通过占位符，动态数据被替换到模板中 | 编程语言被嵌入到模板中                   |
| 不做任何逻辑处理，只做字符串替换   | 在运行时由模板引擎来执行，也包含替换功能 |
| 处理完全由 handler 来完成          | 功能强大                                 |
| 目标是展示层和逻辑的完全分离       | 逻辑代码遍布 handler 和 模板，难以维护   |

### Go 的模板引擎

- 主要使用的是 text/template ， HTML 相关的部分使用了html/template ，是个混合体。
-  模板可以完全无逻辑，但又具有足够的嵌入特性
-  和大多数模板引擎一样， Go Web 的模板位于无逻辑和嵌入逻辑之间的某个地方

### Go 模板引擎的工作原理

- 在 Web 应用中，通常是由handler 来触发模板引擎
-  handler 调用模板引擎，并将使用的模板传递给引擎
  -  通常是一组模板文件和动态数据
-  模板引擎生成 HTML ，并将其写入到 ResponseWriter
-  ResponseWriter 再将它加入到HTTP 响应中，返回给客户端

### 关于模板

- 模板必须是可读的文本格式，扩展名任意。对于 Web 应用通常就是HTML
  -  里面会内嵌一些命令（叫做 action ）
-  text/template 是通用模板引擎， html/template 是 HTML 模板引擎
-  action 位于双层花括号之间： {{ . }}
  -  这里的 . 就是一个 action
  -  它可以命令模板引擎将其替换成一个值

### 一个模板的例子

```html
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <title>Go Web Programming</title>
  </head>
  <body>
    {{ . }}
  </body>
</html>

```



### 使用模板引擎

1. 解析模板源（可以是字符串或模板文件），从而创建一个解析好的模板的 struct
2. 执行解析好的模板，并传入 ResponseWriter 和 数据。
   - 这会触发模板引擎组合解析好的模板和数据，来产生最终的 HTML ，并将它传递给 ResponseWriter

```go
package main

import (
	"net/http"
	"text/template"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/process", process)

	server.ListenAndServe()
}

func process(w http.ResponseWriter, r *http.Request) {
	t, _ := template.ParseFiles("tmpl.html")
	t.Execute(w, "Hello, world!")
}

```

tmpl.html 文件

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Template</title>
</head>

<body>
    {{.}}
</body>

</html>

```



### 解析模板

- ParseFiles
-  ParseGlob
-  Parse

### ParseFiles

- 解析模板文件，并创建一个解析好的模板 struct ，后续可以被执行
-  ParseFiles 函数是 Template struct 上 ParseFiles 方法的简便调用
-  调用 ParseFiles 后，会创建一个新的模板，模板的名字是文件名
-  New 函数
-  ParseFiles 的参数数量可变，但只返回一个模板
  -  当解析多个文件时，第一个文件作为返回的模板（名、内容），其余的作为map ，供后续执行使用

```go
package main

import (
	"net/http"
	"text/template"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/process", process)

	server.ListenAndServe()
}

func process(w http.ResponseWriter, r *http.Request) {
	// t, _ := template.ParseFiles("tmpl.html")

	t := template.New("tmpl.html")
	t, _ = t.ParseFiles("tmpl.html")
	t.Execute(w, "Hello, world!")
}

```



### ParseGlob

- 使用模式匹配来解析特定的文件

```go
package main

import (
	"net/http"
	"text/template"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/process", process)

	server.ListenAndServe()
}

func process(w http.ResponseWriter, r *http.Request) {
	// t, _ := template.ParseFiles("tmpl.html")

	// t := template.New("tmpl.html")
	// t, _ = t.ParseFiles("tmpl.html")

	t, _ := template.ParseGlob("*.html")
	t.Execute(w, "Hello, world!")
}

```



### Parse

- 可以解析字符串模板，其它方式最终都会调用 Parse

### Lookup 方法

- 通过模板名来寻找模板，如果没找到就返回 nil

### Must 函数

- 可以包裹一个函数，返回到一个模板的指针 和 一个错误。
  -  如果错误不为 nil ，那么就 panic

### 执行模板

| Execute                      | ExecuteTemplate                        |
| ---------------------------- | -------------------------------------- |
| 参数是 ResponseWriter 、数据 | 参数是： ResponseWriter 、模板名、数据 |
| 单模板：很适用               |                                        |
| 模板集：只用第一个模板       | 模板集：很适用                         |

```go
package main

import (
	"net/http"
	"text/template"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}

	http.HandleFunc("/process", process)

	server.ListenAndServe()
}

func process(w http.ResponseWriter, r *http.Request) {
	t, _ := template.ParseFiles("tmpl.html")

	t.Execute(w, "Hello, world!")

	ts, _ := template.ParseFiles("t1.html", "t2.html")
	ts.ExecuteTemplate(w, "t2.html", "Hello, world!")
}

```

### Demo 模版解析与执行

目录结构

```bash

Code/go/web-tutorial via 🐹 v1.20.3 via 🅒 base took 3m 2.9s 
➜ tree
.
├── go.mod
├── main.go
├── templates
│   ├── about.html
│   ├── contact.html
│   └── home.html
└── wwwroot
    ├── css
    │   ├── about.css
    │   ├── contact.css
    │   └── home.css
    └── img
        ├── gopher black.png
        ├── gopher flash.png
        └── gopher01.png

5 directories, 15 files

Code/go/web-tutorial via 🐹 v1.20.3 via 🅒 base 
➜ 
```

main.go 文件

```go
package main

import (
	"log"
	"net/http"
	"text/template"
)

func main() {
	templates := loadTemplates()

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fileName := r.URL.Path[1:]
		t := templates.Lookup(fileName)
		if t != nil {
			err := t.Execute(w, nil)
			if err != nil {
				log.Fatalln(err.Error())
			}
		} else {
			w.WriteHeader(http.StatusNotFound)
		}
	})

	http.Handle("/css/", http.FileServer(http.Dir("wwwroot")))
	http.Handle("/img/", http.FileServer(http.Dir("wwwroot")))

	http.ListenAndServe("localhost:8080", nil)
}

func loadTemplates() *template.Template {
	result := template.New("templates")
	template.Must(result.ParseGlob("templates/*.html"))
	return result
}

```

about.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>About</title>
    <link rel="stylesheet" href="/css/about.css">
</head>
<body>
    <h1 class="about">About me</h1>
    <img src="/img/gopher black.png" alt="">
</body>
</html>

```

contact.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Contact</title>
    <link rel="stylesheet" href="/css/contact.css">
</head>
<body>
    <h1 class="contact">abc@126.com</h1>
    <img src="/img/gopher flash.png" alt="">
</body>
</html>

```

home.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Home</title>
    <link rel="stylesheet" href="/css/home.css">
</head>
<body>
    <h1 class="home">This is home!</h1>
    <img src="/img/gopher01.png" alt="">
</body>
</html>

```

about.css

```css
.about {
    color: orange;
}

```

contact.css

```css
.contact {
    color: deepskyblue;
}

```

home.css

```css
.home {
    color: green;
}

```



### 模板 - Action

### 什么是 Action

- Action 就是 Go 模板中嵌入的命令，位于两组花括号之间 {{ xxx }}
-  . 就是一个 Action ，而且是最重要的一个。它代表了传入模板的数据
-  Action 主要可以分为五类：
  -  条件类
  -  迭代 / 遍历类
  -  设置类
  -  包含类
  -  定义类

### 条件 Action

```html
{{ if arg }}
some content
{{ end }}

{{ if arg }}
some content
{{ else }}
other content
{{ end }}
```

main.go

```go
package main

import (
	"html/template"
	"math/rand"
	"net/http"
	"time"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}
	http.HandleFunc("/process", process)
	server.ListenAndServe()
}

func process(w http.ResponseWriter, r *http.Request) {
	t, _ := template.ParseFiles("tmpl.html")
	rand.New(rand.NewSource(time.Now().Unix()))
	// rand.Seed(time.Now().Unix())
	t.Execute(w, rand.Intn(10) > 5)
}

```

tmpl.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Action</title>
</head>

<body>
    {{if .}}
    Number is greater than 5!
    {{else}}
    Number is 5 or less!
    {{end}}
</body>

</html>

```



### 迭代 / 遍历 Action

```html
{{ range array }}
Dot is set to the element {{ . }}
{{ end }}
```

- 这类 Action 用来遍历数组、 slice 、 map 或 channel 等数据结构
  -  “.” 用来表示每次迭代循环中的元素

main.go 

```go
package main

import (
	"html/template"
	"net/http"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}
	http.HandleFunc("/process", process)
	server.ListenAndServe()
}

func process(w http.ResponseWriter, r *http.Request) {
	t, _ := template.ParseFiles("tmpl.html")

	daysOfWeek := []string{"Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"}
	t.Execute(w, daysOfWeek)
}

```

tmpl.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Action</title>
</head>

<body>
    <ul>
        {{range .}}
        <li>{{.}}</li>
        {{end}}
    </ul>
</body>

</html>

```



-  回落机制

main.go

```go
package main

import (
	"html/template"
	"net/http"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}
	http.HandleFunc("/process", process)
	server.ListenAndServe()
}

func process(w http.ResponseWriter, r *http.Request) {
	t, _ := template.ParseFiles("tmpl.html")
	daysOfWeek := []string{}
	t.Execute(w, daysOfWeek)
}

```

tmpl.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Action</title>
</head>

<body>
    <ul>
        {{range .}}
        <li>{{.}}</li>
        {{else}}
        <li>Nothing to show</li>
        {{end}}
    </ul>
</body>

</html>

```



### 设置 Action

```html
{{ with arg }}
Dot is set to arg
{{ end }}
```

- 它允许在指定范围内，让“ .” 来表示其它指定的值（ arg ）

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Action</title>
</head>

<body>
    <div>The dot is {{ . }}</div>
    <div>
        {{ with "world"}}
        Now the dot is set to {{ . }}
        {{ end }}
    </div>
    <div>The dot is {{ . }} again</div>
</body>

</html>

```



-  也有回落机制

```go
package main

import (
	"html/template"
	"net/http"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}
	http.HandleFunc("/process", process)
	server.ListenAndServe()
}

func process(w http.ResponseWriter, r *http.Request) {
	t, _ := template.ParseFiles("tmpl.html")
	
	daysOfWeek := []string{"hello"}
	t.Execute(w, daysOfWeek)
	
}

```

tmpl.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Action</title>
</head>

<body>
    <div>The dot is {{.}}</div>
    <div>
        {{ with ""}}
        Now the dot is set to {{ . }}
        {{else}}
        The dot is still set to {{ . }}
        {{ end }}
    </div>
    <div>The dot is {{ . }} again</div>
</body>

</html>

```



### 包含 Action

```html
{{ template "name" }}
```

- 它允许你在模板中包含其它的模板

t1.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Action</title>
</head>

<body>
    <div>This is t1.html</div>
    <div>This is the value of the dot in t1.html - [{{.}}]</div>
    <hr />
    {{template "t2.html"}}
    <hr />
    <div>This is t1.html after</div>
</body>

</html>

```

t2.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <div style="background-color: yellow;">
        This is t2.html <br />
        This is the value of the dot in t2.html - [{{.}}]
    </div>
</body>
</html>

```

main.go 

```go
package main

import (
	"html/template"
	"net/http"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}
	http.HandleFunc("/process", process)
	server.ListenAndServe()
}

func process(w http.ResponseWriter, r *http.Request) {
	t, _ := template.ParseFiles("t1.html", "t2.html")
	
	t.Execute(w, "hello")
}

```



```html
{{ template "name" arg }}
```

-  给被包含模板传递参数

t1.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Action</title>
</head>

<body>
    <div>This is t1.html</div>
    <div>This is the value of the dot in t1.html - [{{.}}]</div>
    <hr />
    {{template "t2.html" .}}
    <hr />
    <div>This is t1.html after</div>
</body>

</html>

```



### 定义 Action

• define action



### 模板 – 函数与管道

### 参数（ argument ）

- 参数就是模板里面用到的值。
  -  可以是 bool 、整数、 string ...
  -  也可以是 struct 、 struct 的字段、数组的 key 等等
-  参数可以是变量、方法（返回单个值或返回一个值和一个错误）或函数
-  参数可以是一个点“ .” ，也就是传入模板引擎的那个值。

### 参数的例子

```html
{{ if arg }}
some content
{{ end }}
```

- 这里的 arg 就是参数

### 在 Action 中设置变量

- 可以在 action 中设置变量，变量以 $ 开头：
  -  $variable := value
-  一个迭代 action 的例子：

```html
{{ range $key, $value := . }}
The key is {{ $key }} and the value is {{ $value }}
{{ end }}
```



### 管道（ pipeline ）

- 管道是按顺序连接到一起的参数、函数和方法。
-  和 Unix 的管道类似
-  例如： {{ p1 | p2 | p3 }}
  -  p1 、 p2 、 p3 要么是参数，要么是函数
-  管道允许我们把参数的输出发给下一个参数，下一个参数由管道（ | ）分隔开。

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pipeline</title>
</head>

<body>
    {{ 12.3456 | printf "%.2f"}}
    {{ printf "%.2f" 12.3456 }}
</body>

</html>

```



### 函数

- 参数可以是一个函数
-  Go 模板引擎提供了一些基本的内置函数，功能比较有限。例如fmt.Sprint 的各类变体等
-  开发者可以自定义函数：
  -  可以接收任意数量的输入参数
  -  返回：
    -  一个值
    -  一个值 + 一个错误

### 内置函数

- define 、 template 、 block
-  html 、 js 、 urlquery 。对字符串进行转义，防止安全问题
  -  如果是 Web 模板，那么不会需要经常使用这些函数。
-  index
-  print/printf/println
-  len
-  with

### 如何自定义函数

- `template.Funcs(funcMap FuncMap) *Template`
-  `type FuncMap map[string]interface{}`
  -  value 是函数
    -  可以有任意数量的参数
    -  返回单个值的函数或返回一个值 + 一个错误的函数

1. 创建一个 FuncMap （ map 类型）。
   1.  key 是函数名
   2. value 就是函数
2. 把 FuncMap 附加到模板

### 如何使用自定义函数

- 常见用法： template.New(“”).Funcs(funcMap).Parse(...)
  -  调用顺序非常重要
-  可以在管道中使用
-  也可以作为正常函数使用
-  管道更强大且灵活

```go
package main

import (
	"html/template"
	"net/http"
	"time"
)

func main() {
	server := http.Server{
		Addr: "localhost:8080",
	}
	http.HandleFunc("/process", process)
	server.ListenAndServe()
}

func process(w http.ResponseWriter, r *http.Request) {
	funcMap := template.FuncMap{"fdate": formatDate}
	t := template.New("t1.html").Funcs(funcMap)
	t.ParseFiles("t1.html")
	t.Execute(w, time.Now())
}

func formatDate(t time.Time) string {
	layout := "2022-01-03"
	return t.Format(layout)
}

```

t1.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- <title>Action</title> -->
    <title>Pipeline</title>
</head>

<body>
    {{ . | fdate}}
    {{fdate .}}
</body>

</html>

```



### 模板 – 模板组合

### Layout 模板

- Layout 模板就是网页中固定的部分，它可以被多个网页重复使用

### 如何制作 layout 模板

- Include （包含） action 的形式： {{ template "name" . }}
-  以这种方式做 layout 模板是不可行的
-  正确的做法是在模板文件里面使用 define action 再定义一个模板
-  也可以在多个模板文件里，定义同名的模板

main.go 

```go
package main

import (
	"html/template"
	"net/http"
)

func main() {
	http.HandleFunc("/home", func(w http.ResponseWriter, r *http.Request) {
		t, _ := template.ParseFiles("layout.html", "home.html")
		t.ExecuteTemplate(w, "layout", "Hello World")
	})
	http.ListenAndServe("localhost:8080", nil)
}

```

layout.html

```html
{{ define "layout" }}

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Layout</title>
</head>

<body>
    <h1>公共模版</h1>
    {{ template "content" .}}
</body>

</html>

{{ end}}

```

home.html

```html
{{ define "content"}}

<h1>Home</h1>
<h2>{{ .}}</h2>

{{ end}}

```

about.html

```html
{{ define "content"}}
<h1>About</h1>
{{ end}}

```

main.go

```go
package main

import (
	"html/template"
	"net/http"
)

func main() {
	http.HandleFunc("/home", func(w http.ResponseWriter, r *http.Request) {
		t, _ := template.ParseFiles("layout.html", "home.html")
		t.ExecuteTemplate(w, "layout", "Hello World")
	})
	http.HandleFunc("/about", func(w http.ResponseWriter, r *http.Request) {
		t, _ := template.ParseFiles("layout.html", "about.html")
		t.ExecuteTemplate(w, "layout", "")
	})
	http.ListenAndServe("localhost:8080", nil)
}

```



### 使用 block action 定义默认模板

```html
{{ block arg }}
Dot is set to arg
{{ end }}
```

- block action 可以定义模板，并同时就使用它
-  template ：模板必须可用
-  block ：模板可以不存在

layout.html

```html
{{ define "layout" }}

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Layout</title>
</head>

<body>
    <h1>公共模版</h1>
    {{ block "content" .}}
    block content
    {{ end}}
</body>

</html>

{{ end}}

```

main.go

```go
package main

import (
	"html/template"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/home", func(w http.ResponseWriter, r *http.Request) {
		t, _ := template.ParseFiles("layout.html", "home.html")
		t.ExecuteTemplate(w, "layout", "Hello World")
	})
	http.HandleFunc("/about", func(w http.ResponseWriter, r *http.Request) {
		t, _ := template.ParseFiles("layout.html", "about.html")
		t.ExecuteTemplate(w, "layout", "")
	})
	http.HandleFunc("/contact", func(w http.ResponseWriter, r *http.Request) {
		t, e := template.ParseFiles("layout.html")
		e = t.ExecuteTemplate(w, "layout", "")
		log.Println(e)
	})
	http.ListenAndServe("localhost:8080", nil)
}

```

### 逻辑运算符

- eq/ne
-  lt/gt
-  le/ge
-  and
-  or
-  not

```html
{{ define "content"}}

<h1>Home</h1>
<h2>{{ .}}</h2>
<h2>
    {{ if eq . "Hello world!"}}
    !!!
    {{ end}}
</h2>

{{ end}}

```



## 八、路由

### 目前为止

```go
func main() {
	http.HandleFunc("/home", func(w http.ResponseWriter, r *http.Request) {
		t, _ := template.ParseFiles("layout.html", "home.html")
		t.ExecuteTemplate(w, "layout", "Hello World")
	})
	http.HandleFunc("/about", func(w http.ResponseWriter, r *http.Request) {
		t, _ := template.ParseFiles("layout.html", "about.html")
		t.ExecuteTemplate(w, "layout", "")
	})
	http.HandleFunc("/contact", func(w http.ResponseWriter, r *http.Request) {
		t, e := template.ParseFiles("layout.html")
		e = t.ExecuteTemplate(w, "layout", "")
		log.Println(e)
	})
	http.ListenAndServe("localhost:8080", nil)
}

```



### Controller 的角色

- main() ：设置类工作
-  controller ：
  -  静态资源
  -  把不同的请求送到不同的 controller 进行处理

### 路由结构

/home  前置 controller /home   home handler（响应）  company  handler  about  handler
根据请求，匹配最具体的 handler

```bash
Code/go/action_demo via 🐹 v1.20.3 via 🅒 base
➜ tree
.
├── about.html
├── controller
│   ├── about.go
│   ├── contact.go
│   ├── controller.go
│   └── home.go
├── go.mod
├── home.html
├── layout.html
├── main.go
├── t1.html
├── t2.html
└── tmpl.html

2 directories, 12 files

Code/go/action_demo via 🐹 v1.20.3 via 🅒 base
➜
```

main.go

```go
package main

import (
	"action_demo/controller"
	"net/http"
)

func main() {
	controller.RegisterRoutes()
	http.ListenAndServe("localhost:8080", nil)
}

```

about.go

```go
package controller

import (
	"net/http"
	"text/template"
)

func registerAboutRoutes() {
	http.HandleFunc("/about", handleAbout)
}

func handleAbout(w http.ResponseWriter, r *http.Request) {
	t, _ := template.ParseFiles("layout.html", "about.html")
	t.ExecuteTemplate(w, "layout", "")
}

```

contact.go

```go
package controller

import (
	"log"
	"net/http"
	"text/template"
)

func registerContactRoutes() {
	http.HandleFunc("/contact", handleContact)
}

func handleContact(w http.ResponseWriter, r *http.Request) {
	t, e := template.ParseFiles("layout.html")
	e = t.ExecuteTemplate(w, "layout", "")
	log.Println(e)
}

```

home.go

```go
package controller

import (
	"net/http"
	"text/template"
)

func registerHomeRoutes() {
	http.HandleFunc("/home", handleHome)
}

func handleHome(w http.ResponseWriter, r *http.Request) {
	t, _ := template.ParseFiles("layout.html", "home.html")
	t.ExecuteTemplate(w, "layout", "Hello world!")
}

```

controller.go

```go
package controller

// RegisterRouter ...
func RegisterRoutes() {

	// static resources

	registerHomeRoutes()
	registerAboutRoutes()
	registerContactRoutes()
}

```



### 路由参数

- 静态路由：一个路径对应一个页面
  -  /home
  -  /about
-  带参数的路由：根据路由参数，创建出一族不同的页面
  -  /companies/123
  -  /companies/Microsoft

company.go

```go
package controller

import (
	"net/http"
	"regexp"
	"strconv"
	"text/template"
)

func registerCompanyRoutes() {
	http.HandleFunc("/companies", handleCompanies)
	http.HandleFunc("/companies/", handleCompany)
}

func handleCompanies(w http.ResponseWriter, r *http.Request) {
	t, _ := template.ParseFiles("layout.html", "companies.html")
	t.ExecuteTemplate(w, "layout", nil)
}

func handleCompany(w http.ResponseWriter, r *http.Request) {
	t, _ := template.ParseFiles("layout.html", "company.html")

	pattern, _ := regexp.Compile(`/companies/(\d+)`)
	matches := pattern.FindStringSubmatch(r.URL.Path)
	if len(matches) > 0 {
		companyID, _ := strconv.Atoi(matches[1])
		t.ExecuteTemplate(w, "layout", companyID)
	} else {
		w.WriteHeader(http.StatusNotFound)
	}
}

```

company.html

```html
{{ define "content"}}

<h1>Company: {{ . }}</h1>


{{ end }}

```

companies.html

```html
{{ define "content"}}

<h1>All Companies</h1>
<ul>
    <li>Microsoft</li>
    <li>Google</li>
    <li>Apple</li>
    <li>Facebook</li>
</ul>

{{ end }}

```

controller.go

```go
package controller

// RegisterRouter ...
func RegisterRoutes() {

	// static resources

	registerHomeRoutes()
	registerAboutRoutes()
	registerContactRoutes()
	registerCompanyRoutes()
}

```



### 第三方路由器

- gorilla/mux ：灵活性高、功能强大、性能相对差一些
-  httprouter ：注重性能、功能简单
-  编写你自己的路由规则

## 九、JSON

### JSON 与 Go 的 Struct

| JSON                                              | Go Struct                                                  |
| ------------------------------------------------- | ---------------------------------------------------------- |
| `{ "id": 123, "name": "Goole", "country": "USA"}` | `type Company struct { ID int Name string Country string}` |

### Tags

```go
type Company struct {
  ID      int    `json:"id"`
  Name    string `json:"name"`
  Country string `json:"country"`
}
```



### 类型映射

- Go bool ： JSON boolean
-  Go float64 ： JSON 数值
-  Go string ： JSON strings
-  Go nil ： JSON null.

### 对于未知结构的 JSON

- map[string]interface{} 可以存储任意 JSON 对象
-  []interface{} 可以存储任意的 JSON 数组

### 读取 JSON

- 需要一个解码器： `dec := json.NewDecoder(r.Body)`
-  参数需实现 Reader 接口
-  在解码器上进行解码： `dec.Decode(&query)`

### 写入 JSON

- 需要一个编码器： `enc := json.NewEncoder(w)`
  -  参数需实现 Writer 接口
-  编码：` enc.Encode(results)`

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/companies", func(w http.ResponseWriter, r *http.Request) {
		switch r.Method {
		case http.MethodPost:
			dec := json.NewDecoder(r.Body)
			company := Company{}
			err := dec.Decode(&company)
			if err != nil {
				log.Println(err.Error())
				w.WriteHeader(http.StatusInternalServerError)
				return
			}

			enc := json.NewEncoder(w)
			err = enc.Encode(company)
			if err != nil {
				log.Println(err.Error())
				w.WriteHeader(http.StatusInternalServerError)
				return
			}
		default:
			w.WriteHeader(http.StatusMethodNotAllowed)
		}
	})

	http.ListenAndServe("localhost:8080", nil)
}

```

model.go

```go
package main

// Company ...
type Company struct {
	ID      int    `json:"id"`
	Name    string `json:"name"`
	Country string `json:"country"`
}

```

test.http

```http
POST http://localhost:8080/companies HTTP/1.1
Content-Type: application/json

{
    "id": 123,
    "name": "Google",
    "country": "USA"
}

```

响应

```http
HTTP/1.1 200 OK
Date: Sun, 14 May 2023 15:33:01 GMT
Content-Length: 43
Content-Type: text/plain; charset=utf-8
Connection: close

{
  "id": 123,
  "name": "Google",
  "country": "USA"
}
```



### Marshal 和 Unmarshal

- Marshal （编码） : 把 go struct 转化为 json 格式
  -  MarshalIndent ，带缩进
-  Unmarshal （解码） : 把 json 转化为 go struct

```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	jsonStr := `
	{
		"id": 123,
		"name": "Google",
		"country": "USA"
	}`

	c := Company{}
	_ = json.Unmarshal([]byte(jsonStr), &c)
	fmt.Println("c", c)

	bytes, _ := json.Marshal(c)
	fmt.Println("bytes", string(bytes))

	bytes1, _ := json.MarshalIndent(c, "", " ")
	fmt.Println("bytes1", string(bytes1))
}

```

运行

```bash
Code/go/json_demo via 🐹 v1.20.3 via 🅒 base 
➜ go run .
c {123 Google USA}
bytes {"id":123,"name":"Google","country":"USA"}
bytes1 {
 "id": 123,
 "name": "Google",
 "country": "USA"
}

Code/go/json_demo via 🐹 v1.20.3 via 🅒 base 
➜ 
```



### 两种方式区别

- 针对 string 或 bytes ：
  -  Marshal => String
  -  Unmarshal <= String
-  针对 stream ：
  -  Encode => Stream ，把数据写入到 io.Writer
  -  Decode <= Stream ，从 io.Reader 读取数据

## 十、中间件

### 什么是中间件

请求  Handler  响应
请求  中间件 Middleware    Handler   响应

### 创建中间件

- `func ListenAndServe(addr string, handler Handler) error`
-  handler 如果是 nil ： DefaultServeMux

```go
type Handler interface {
  ServeHTTP(ResponseWriter, *Request)
}
```

```go
type MyMiddleware struct {
  Next http.Handler
}

```

```go
func(m MyMiddleware) ServeHTTP(w http.ResponseWriter, r *http.Request) {
  // 在 next handler 之前做一些事情
  m.Next.ServeHTTP(w, r)
  // 在 next handler 之后做一些事情
}
```



### 中间件的用途

- Logging
-  安全
-  请求超时
-  响应压缩
-  ...

目录

```bash
Code/go/middleware_demo via 🐹 v1.20.3 via 🅒 base
➜ tree
.
├── go.mod
├── main.go
├── middleware
│   └── auth.go
├── model.go
└── test.http

2 directories, 5 files

Code/go/middleware_demo via 🐹 v1.20.3 via 🅒 base
➜
```



main.go

````go
package main

import (
	"encoding/json"
	"middleware_demo/middleware"
	"net/http"
)

func main() {
	http.HandleFunc("/companies", func(w http.ResponseWriter, r *http.Request) {
		c := Company{
			ID:      123,
			Name:    "Google",
			Country: "USA",
		}

		enc := json.NewEncoder(w)
		enc.Encode(c)
	})

	http.ListenAndServe("localhost:8080", new(middleware.AuthMiddleware))
}

````

model.go

```go
package main

type Company struct {
	ID      int64  `json:"id"`
	Name    string `json:"name"`
	Country string `json:"country"`
}

```

auth.go

```go
package middleware

import "net/http"

type AuthMiddleware struct {
	Next http.Handler
}

func (am *AuthMiddleware) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	if am.Next == nil {
		am.Next = http.DefaultServeMux
	}

	auth := r.Header.Get("Authorization")
	if auth != "" {
		am.Next.ServeHTTP(w, r)
	} else {
		w.WriteHeader(http.StatusUnauthorized)
	}
}

```

test.http

```http
### without Auth
GET http://localhost:8080/companies HTTP/1.1


### with Auth
GET http://localhost:8080/companies HTTP/1.1
Authorization: abcd

```

响应

```http
HTTP/1.1 401 Unauthorized
Date: Mon, 15 May 2023 13:45:20 GMT
Content-Length: 0
Connection: close

HTTP/1.1 200 OK
Date: Mon, 15 May 2023 13:49:06 GMT
Content-Length: 43
Content-Type: text/plain; charset=utf-8
Connection: close

{
  "id": 123,
  "name": "Google",
  "country": "USA"
}
```



## 十一、请求上下文

### 使用请求上下文

中间件 Handler  数据库   Web API    文件系统

### Request Context

- `func(*Request) Context() context.Context`*
  - 返回当前请求的上下文
-  `func(*Request) WithContext(ctx context.Context) context.Context`
  -  基于 Context 进行“修改”，（实际上）创建一个新的 Context

### context.Context

```go
type Context interface {
  Deadline() (deadline time.Time, ok bool)
  Done() <-chan struct{}
  Err() error
  Value(key interface{}) interface{}
}
```



- 这些方法都是用于读取，不能进行设置

### Context API – 可以返回新 Context

- WithCancel() ，它有一个 CancelFunc 
- WithDeadline() ，带有一个时间戳（ time.Time ）
-  WithTimeout() ，带有一个具体的时间段（ time.Duration ）
-  WithValue() ，在里面可以添加一些值

例子

main.go

```go
package main

import (
	"encoding/json"
	"middleware_demo/middleware"
	"net/http"
	"time"
)

func main() {
	http.HandleFunc("/companies", func(w http.ResponseWriter, r *http.Request) {
		c := Company{
			ID:      123,
			Name:    "Google",
			Country: "USA",
		}

		time.Sleep(4 * time.Second)

		enc := json.NewEncoder(w)
		enc.Encode(c)
	})

	http.ListenAndServe("localhost:8080", &middleware.TimeoutMiddleware{
		Next: new(middleware.AuthMiddleware),
	})
}

```

timeout.go

```go
package middleware

import (
	"context"
	"net/http"
	"time"
)

type TimeoutMiddleware struct {
	Next http.Handler
}

func (tm TimeoutMiddleware) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	if tm.Next == nil {
		tm.Next = http.DefaultServeMux
	}

	ctx := r.Context()
	ctx, _ = context.WithTimeout(ctx, 3*time.Second)
	r.WithContext(ctx)
	ch := make(chan struct{})
	go func() {
		tm.Next.ServeHTTP(w, r)
		ch <- struct{}{}
	}()
	select {
	case <-ch:
		return
	case <-ctx.Done():
		w.WriteHeader(http.StatusRequestTimeout)
	}
	ctx.Done()
}

```

## 十二、HTTPS

### HTTP

```http
POST /login HTTP/1.1
...
username=admin&password=123456
```

### HTTPS

```http
TLS
POST /login HTTP/1.1
...
username=admin&password=123456
```

### HTTP Listener

- http.ListenAndServe 函数
-  http.ListenAndServeTLS 函数

例子

```bash
src/crypto/tls  via 🐹 v1.20.3 via 🅒 base
➜ go run generate_cert.go -h
Usage of /var/folders/6y/p7tl9yfj1p3cq9hv5z1fpfqh0000gn/T/go-build1551263502/b001/exe/generate_cert:
  -ca
    	whether this cert should be its own Certificate Authority
  -duration duration
    	Duration that certificate is valid for (default 8760h0m0s)
  -ecdsa-curve string
    	ECDSA curve to use to generate a key. Valid values are P224, P256 (recommended), P384, P521
  -ed25519
    	Generate an Ed25519 key
  -host string
    	Comma-separated hostnames and IPs to generate a certificate for
  -rsa-bits int
    	Size of RSA key to generate. Ignored if --ecdsa-curve is set (default 2048)
  -start-date string
    	Creation date formatted as Jan 1 15:04:05 2011

src/crypto/tls  via 🐹 v1.20.3 via 🅒 base
➜ go run generate_cert.go -host localhost
2023/05/15 22:40:17 Failed to open cert.pem for writing: open cert.pem: permission denied
exit status 1

src/crypto/tls  via 🐹 v1.20.3 via 🅒 base
➜ sudo go run generate_cert.go -host localhost
Password:
2023/05/15 22:40:45 wrote cert.pem
2023/05/15 22:40:45 wrote key.pem

src/crypto/tls  via 🐹 v1.20.3 via 🅒 base took 6.1s
➜
src/crypto/tls  via 🐹 v1.20.3 via 🅒 base
➜ sudo mv cert.pem /Users/qiaopengjun/Code/go/https_web

src/crypto/tls  via 🐹 v1.20.3 via 🅒 base
➜ sudo mv key.pem /Users/qiaopengjun/Code/go/https_web

src/crypto/tls  via 🐹 v1.20.3 via 🅒 base
➜



Code/go/https_web via 🐹 v1.20.3 via 🅒 base 
➜ sudo chmod -R  777 key.pem

Code/go/https_web via 🐹 v1.20.3 via 🅒 base 
➜ go run .
```

main.go

```go
package main

import (
	"fmt"
	"https_web/controller"
	"net/http"
)

func main() {
	controller.RegisterRoutes()
	// http.ListenAndServe("localhost:8080", nil)

	err := http.ListenAndServeTLS("localhost:8080", "cert.pem", "key.pem", nil)
	fmt.Println("error: ", err)
}

```



![image-20230515230758246](../../../Library/Application Support/typora-user-images/image-20230515230758246.png)

点击继续前往

![image-20230515230954348](../../../Library/Application Support/typora-user-images/image-20230515230954348.png)



## 十三、HTTP/2

### HTTP/1.1

TCP
请求 header+body
响应 header+body

### HTTP/2

TCP
Stream
Frame Frame Frame

Stream

Headers
Headers
Continuation
Data
Headers
Continuation
每个数据类型都可以单独优化

### HTTP/2 的特点

- 请求多路复用 
- Header 压缩
-  默认安全
  -  HTTP ，但很多决定不支持 HTTP
  -  HTTPS
-  Server Push

### 没有 Server Push

/home  

home.html
/css/app.css
/css/app.css

### Server Push

->  /home

<-  /css/app.css    

<-  home.html

例子

 home.go

```go
package controller

import (
	"net/http"
	"text/template"
)

func registerHomeRoutes() {
	http.HandleFunc("/home", handleHome)
}

func handleHome(w http.ResponseWriter, r *http.Request) {
	if pusher, ok := w.(http.Pusher); ok {
		pusher.Push("/css/app.css", &http.PushOptions{
			Header: http.Header{"Content-Type": []string{"text/css"}},
		})
	}

	t, _ := template.ParseFiles("layout.html", "home.html")
	t.ExecuteTemplate(w, "layout", "Hello world!")
}

```

## 十四、测试

### 测试 Model 层

- user_test.go
  -  测试代码所在文件的名称以 _test 结尾
  -  对于生产编译，不会包含以 _test 结尾的文件
  -  对于测试编译，会包含以 _test 结尾的文件
-  ·
- `func TestUpdatesModifiedTime(t *testing.T) { ... }`
  -  测试函数名应以 Test 开头（需要导出）
  -  函数名需要表达出被验证的特性
  -  测试函数的参数类型是 *testing.T ，它会提供测试相关的一些工具

company.go

```go
package model

import "strings"

type Company struct {
  ID int `json:"id"`
  Name string `json:"name"`
  Country string `json:"country"`
}

func (c *Company) GetCompanyType() (result string) {
  if strings.HasSuffix(c.Name, ".LTD") {
    result = "Limited Liability Company"
  } else {
    result = "Others"
  }
  return
}
```

company_test.go

```go
package model

import "testing"

func TestCompanyTypeCorrect(t *testing.T) {
  c := Company {
    ID: 12345,
    Name: "ABCD.LTD",
    Country: "China",
  }
  
  companyType := c.GetCompanyType()
  
  if companyType != "Limited Liability Company" {
    t.Errorf("Company's GetCompanyType Method failed to get correct company type!")
  }
}
```

运行

```bash
go test request/model
```



### 测试 Controller 层

- 为了尽量保证单元测试的隔离性，测试不要使用例如数据库、外部API 、文件系统等外部资源。
-  模拟请求和响应
-  需要使用 net/http/httptest 提供的功能

### NewRequest 函数

- `func NewRequest(method, url string, body io.Reader) (*Request, error)`
  -  method ： HTTP Method
  -  url ：请求的 URL
  -  body ：请求的 Body
  -  返回的 *Request 可以传递给 handler 函数

### ResponseRecorder

```go
type ResponseRecorder {
  Code int // 状态码 200 、 500...
  HeaderMap http.Header // 响应的 header
  Body *bytes.Buffer // 响应的 body
  Flushed bool // 缓存是否被 flush 了
}
```

- 用来捕获从 handler 返回的响应，只是做记录
-  可以用于测试断言

company.go 文件

```go
package controller

import (
  "encoding/json"
  "net/http"
  "request/model"
)

func RegisterRoutes() {
  http.HandleFunc("/companies", handleCompany)
}

func handleCompany(w http.ResponseWriter, r *http.Request) {
  c := model.Company {
    ID: 123,
    Name: "Google",
    Country: "USA",
  }
  
  enc := json.NewEncoder(w)
  enc.Encode(c)
}
```

company_test.go

```go
package controller

import (
	"net/http/httptest"
  "request/model"
  "testing"
)

func TestHandleCompanyCorrect(t *testing.T) {
  r := httptest.NewRequest(http.MethodGet, "/companies", nil)
  w := httptest.NewRecorder()
  
  handleCompany(w, r)
  
  result, _ := ioutil.ReadAll(w.Result().Body)
  
  c := model.Company{}
  json.Unmarshal(result, &c)
  
  if c.ID != 123 {
    t.Error("Failed to handle company correctly!")
  }
}
```

运行

```bash
go test request/controller
```

## 十五、Profiling 性能分析

### 能分析什么

- 内存消耗
-  CPU 使用
-  阻塞的 goroutine
-  执行追踪
-  还有一个 Web 界面：应用的实时数据

### 如何进行分析

- `import _ “net/http/pprof”`
  -  设置一些监听的 URL ，它们会提供各类诊断信息
-  go tool pprof http://localhost:8000/debug/pprof/heap // 内存
  -  从应用获取内存 dump ：应用在使用哪些内存，它们会去哪
-  go tool pprof http://localhost:8000/debug/pprof/profile // CPU
  -  CPU 的快照，可以看到谁在用 CPU
-  go tool pprof http://localhost:8000/debug/pprof/block // goroutine
  -  看到阻塞的 goroutine
-  go tool pprof http://localhost:8000/debug/pprof/trace?seconds=5 // trace
  -  监控这段时间内，什么在执行，什么在调用什么...

### 如何进行分析

-  http:// localhost:8000/debug/pprof // 网页



#### Graphviz: <https://graphviz.org/>

https://graphviz.org/download/



## 十六、部署

- 打开命令行 连接到 Linux Server
- go build
- 去掉代码中的 localhost 端口保留
- `nohup ./it &`  忽略登出信号 后台继续执行
- `sudo systemctl enable go-web.service`
- `sudo systemctl start go-web.service`
- `sudo systemctl status go-web.service`
- 反向代理服务器

作业：

- 做一个增删改查Web API  使用模版
- 自己写一些中间件
- 写一些测试
- 做一下性能分析



