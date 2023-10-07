---
title: "Rust Web 全栈开发之自建TCP、HTTP Server"
date: 2023-05-27T12:23:09+08:00
draft: false
tags: ["Rust"]
categories: ["Rust"]
---

# Rust Web 全栈开发之自建TCP、HTTP Server

## 课程简介

### 预备知识

- Rust 编程语言入门

- <https://www.bilibili.com/video/BV1hp4y1k7SV>

### 课程主要内容

- WebService
- 服务器端Web App
- 客户端Web App（WebAssembly）
- Web框架：Actix
- 数据库：PostgreSQL
- 数据库连接：SQLx

全部使用纯Rust编写！

## 一、构建TCP Server

### 本节内容

- 编写TCP Server和Client

### std::net模块

- 标准库的std::net模块，提供网络基本功能
- 支持TCP和UDP通信
- TcpListener和TcpStream

创建项目

```bash
~/rust via 🅒 base
➜ cargo new s1 && cd s1
     Created binary (application) `s1` package

s1 on  master [?] via 🦀 1.67.1 via 🅒 base
➜ cargo new tcpserver
     Created binary (application) `tcpserver` package

s1 on  master [?] via 🦀 1.67.1 via 🅒 base
➜ cargo new tcpclient
     Created binary (application) `tcpclient` package

s1 on  master [?] via 🦀 1.67.1 via 🅒 base
➜ c

s1 on  master [?] via 🦀 1.67.1 via 🅒 base
➜

```

目录

```bash
s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ tree
.
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
├── target
│   ├── CACHEDIR.TAG
│   └── debug
│       ├── build
│       ├── deps
│       ├── examples
│       └── incremental
├── tcpclient
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── tcpserver
    ├── Cargo.toml
    └── src
        └── main.rs

24 directories, 44 files

s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
```

s1/Cargo.toml

```toml
[workspace]

members = ["tcpserver", "tcpclient"]

```

tcpserver/src/main.rs

```rust
use std::net::TcpListener;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:3000").unwrap();
    println!("Running on port 3000...");

    // let result = listener.accept().unwrap(); // 只接收一次请求

    for stream in listener.incoming() {
        let _stream = stream.unwrap();
        println!("Connection established!")
    }
}

```

运行

```bash
s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cargo run -p tcpserver            
   Compiling tcpserver v0.1.0 (/Users/qiaopengjun/rust/s1/tcpserver)
    Finished dev [unoptimized + debuginfo] target(s) in 0.52s
     Running `target/debug/tcpserver`
Running on port 3000...

```

tcpclient/src/main.rs

```rust
use std::net::TcpStream;

fn main() {
    let _stream = TcpStream::connect("localhost:3000").unwrap();
}

```

运行

```bash
s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cargo run -p tcpclient
   Compiling tcpclient v0.1.0 (/Users/qiaopengjun/rust/s1/tcpclient)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32s
     Running `target/debug/tcpclient`

s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 

s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cargo run -p tcpserver            
   Compiling tcpserver v0.1.0 (/Users/qiaopengjun/rust/s1/tcpserver)
    Finished dev [unoptimized + debuginfo] target(s) in 0.52s
     Running `target/debug/tcpserver`
Running on port 3000...
Connection established!

```

tcpserver/src/main.rs

```rust
use std::io::{Read, Write};
use std::net::TcpListener;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:3000").unwrap();
    println!("Running on port 3000...");

    // let result = listener.accept().unwrap(); // 只接收一次请求

    for stream in listener.incoming() {
        let mut stream = stream.unwrap();
        println!("Connection established!");
        let mut buffer = [0; 1024];

        stream.read(&mut buffer).unwrap();
        stream.write(&mut buffer).unwrap();
    }
}

```

tcpclient/src/main.rs

```rust
use std::io::{Read, Write};
use std::net::TcpStream;
use std::str;

fn main() {
    let mut stream = TcpStream::connect("localhost:3000").unwrap();
    stream.write("Hello".as_bytes()).unwrap();

    let mut buffer = [0; 5];
    stream.read(&mut buffer).unwrap();

    println!(
        "Response from server: {:?}",
        str::from_utf8(&buffer).unwrap()
    );
}

```

运行

```bash
s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cargo run -p tcpclient
   Compiling tcpclient v0.1.0 (/Users/qiaopengjun/rust/s1/tcpclient)
    Finished dev [unoptimized + debuginfo] target(s) in 0.15s
     Running `target/debug/tcpclient`
Response from server: "Hello"

s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 
```

## 二、构建HTTP Server - 解析 HTTP 请求

### 本节内容

- 编写HTTP Server
- 测试HTTP Server

### Web Server的消息流动图

客户端 Internet 请求(1) ->  Server （HTTP Library）->(2) Router ->(3) -> Handlers ->处理请求并返回响应(4) 客户端

### 注意

- Rust没有内置的HTTP支持

Web Server

- Server
  - 监听进来的TCP字节流
- Router
  - 接受HTTP请求，并决定调用哪个Handler
- Handler
  - 处理HTTP请求，构建HTTP响应
- HTTP Library
  - 解释字节流，把它转化为HTTP请求
  - 把HTTP响应转化回字节流

### 构建步骤

- 解析HTTP请求消息
- 构建HTTP响应消息
- 路由与Handler
- 测试Web Server

### 解析HTTP请求（消息）

### 三个数据结构

| 数据结构名称 | 数据类型 | 描述                 |
| ------------ | -------- | -------------------- |
| HttpRequest  | struct   | 表示HTTP请求         |
| Method       | enum     | 指定所允许的HTTP方法 |
| Version      | enum     | 指定所允许的HTTP版本 |

### HTTP请求

Structure of an HTTP Request

Request Line  Method Path  Version

Header Line 1

Header Line 2

Header Line 3

Empty line

Message body (optional)

```bash
s1 on  master [?] via 🦀 1.67.1 via 🅒 base
➜ cargo new httpserver
warning: compiling this new package may not work due to invalid workspace configuration

current package believes it's in a workspace when it's not:
current:   /Users/qiaopengjun/rust/s1/httpserver/Cargo.toml
workspace: /Users/qiaopengjun/rust/s1/Cargo.toml

this may be fixable by adding `httpserver` to the `workspace.members` array of the manifest located at: /Users/qiaopengjun/rust/s1/Cargo.toml
Alternatively, to keep it out of the workspace, add the package to the `workspace.exclude` array, or add an empty `[workspace]` table to the package's manifest.
     Created binary (application) `httpserver` package

s1 on  master [?] via 🦀 1.67.1 via 🅒 base
➜ cargo new --lib http
warning: compiling this new package may not work due to invalid workspace configuration

current package believes it's in a workspace when it's not:
current:   /Users/qiaopengjun/rust/s1/http/Cargo.toml
workspace: /Users/qiaopengjun/rust/s1/Cargo.toml

this may be fixable by adding `http` to the `workspace.members` array of the manifest located at: /Users/qiaopengjun/rust/s1/Cargo.toml
Alternatively, to keep it out of the workspace, add the package to the `workspace.exclude` array, or add an empty `[workspace]` table to the package's manifest.
     Created library `http` package

s1 on  master [?] via 🦀 1.67.1 via 🅒 base
➜
```

### 需要实现的Trait

| Trait        | 描述                                      |
| ------------ | ----------------------------------------- |
| `From<&str>` | 用于把传进来的字符串切片转化为HttpRequest |
| Debug        | 打印调试信息                              |
| `PartialEq`  | 用于解析和自动化测试脚本里做比较          |

目录

```bash
s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ tree
.
├── Cargo.lock
├── Cargo.toml
├── http
│   ├── Cargo.toml
│   └── src
│       ├── httprequest.rs
│       ├── httpresponse.rs
│       └── lib.rs
├── httpserver
│   ├── Cargo.toml
│   └── src
│       └── main.rs
├── src
│   └── main.rs
├── target
│   ├── CACHEDIR.TAG
│   └── debug
│       ├── build
│       ├── deps
│       ├── examples
│       ├── httpserver
│       ├── httpserver.d
│       ├── incremental
│       ├── libhttp.d
│       ├── libhttp.rlib
│       ├── tcpclient
│       ├── tcpclient.d
│       ├── tcpserver
│       └── tcpserver.d
├── tcpclient
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── tcpserver
    ├── Cargo.toml
    └── src
        └── main.rs

48 directories, 302 files

s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 
```

http/lib.rs

```rust
pub mod httprequest;

```

Cargo.toml

```toml
[workspace]

members = ["tcpserver", "tcpclient", "http", "httpserver"]

```

http/src/httprequest.rs

```rust
#[derive(Debug, PartialEq)]
pub enum Method {
    Get,
    Post,
    Uninitialized,
}

impl From<&str> for Method {
    fn from(s: &str) -> Method {
        match s {
            "GET" => Method::Get,
            "POST" => Method::Post,
            _ => Method::Uninitialized,
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_method_into() {
        let m: Method = "GET".into();
        assert_eq!(m, Method::Get);
    }
}

```

运行

```bash
s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cargo test -p http   
   Compiling http v0.1.0 (/Users/qiaopengjun/rust/s1/http)
    Finished test [unoptimized + debuginfo] target(s) in 0.19s
     Running unittests src/lib.rs (target/debug/deps/http-ca93876a5b13f05e)

running 1 test
test httprequest::tests::test_method_into ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests http

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 
```

http/src/httprequest.rs

```rust
#[derive(Debug, PartialEq)]
pub enum Method {
    Get,
    Post,
    Uninitialized,
}

impl From<&str> for Method {
    fn from(s: &str) -> Method {
        match s {
            "GET" => Method::Get,
            "POST" => Method::Post,
            _ => Method::Uninitialized,
        }
    }
}

#[derive(Debug, PartialEq)]
pub enum Version {
    V1_1,
    V2_0,
    Uninitialized,
}

impl From<&str> for Version {
    fn from(s: &str) -> Version {
        match s {
            "HTTP/1.1" => Version::V1_1,
            _ => Version::Uninitialized,
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_method_into() {
        let m: Method = "GET".into();
        assert_eq!(m, Method::Get);
    }

    #[test]
    fn test_version_into() {
        let v: Version = "HTTP/1.1".into();
        assert_eq!(v, Version::V1_1);
    }
}

```

运行

```bash
s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cargo test -p http
   Compiling http v0.1.0 (/Users/qiaopengjun/rust/s1/http)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests src/lib.rs (target/debug/deps/http-ca93876a5b13f05e)

running 2 tests
test httprequest::tests::test_version_into ... ok
test httprequest::tests::test_method_into ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests http

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 
```

http/src/httprequest.rs

```rust
use std::collections::HashMap;

#[derive(Debug, PartialEq)]
pub enum Method {
    Get,
    Post,
    Uninitialized,
}

impl From<&str> for Method {
    fn from(s: &str) -> Method {
        match s {
            "GET" => Method::Get,
            "POST" => Method::Post,
            _ => Method::Uninitialized,
        }
    }
}

#[derive(Debug, PartialEq)]
pub enum Version {
    V1_1,
    V2_0,
    Uninitialized,
}

impl From<&str> for Version {
    fn from(s: &str) -> Version {
        match s {
            "HTTP/1.1" => Version::V1_1,
            _ => Version::Uninitialized,
        }
    }
}

#[derive(Debug, PartialEq)]
pub enum Resource {
    Path(String),
}

#[derive(Debug)]
pub struct HttpRequest {
    pub method: Method,
    pub version: Version,
    pub resource: Resource,
    pub headers: HashMap<String, String>,
    pub msg_body: String,
}

impl From<String> for HttpRequest {
    fn from(req: String) -> Self {
        let mut parsed_method = Method::Uninitialized;
        let mut parsed_version = Version::V1_1;
        let mut parsed_resource = Resource::Path("".to_string());
        let mut parsed_headers = HashMap::new();
        let mut parsed_msg_body = "";

        for line in req.lines() {
            if line.contains("HTTP") {
                let (method, resource, version) = process_req_line(line);
                parsed_method = method;
                parsed_resource = resource;
                parsed_version = version;
            } else if line.contains(":") {
                let (key, value) = process_header_line(line);
                parsed_headers.insert(key, value);
            } else if line.len() == 0 {
            } else {
                parsed_msg_body = line;
            }
        }

        HttpRequest {
            method: parsed_method,
            version: parsed_version,
            resource: parsed_resource,
            headers: parsed_headers,
            msg_body: parsed_msg_body.to_string(),
        }
    }
}

fn process_req_line(s: &str) -> (Method, Resource, Version) {
    let mut words = s.split_whitespace();
    let method = words.next().unwrap();
    let resource = words.next().unwrap();
    let version = words.next().unwrap();

    (
        method.into(),
        Resource::Path(resource.to_string()),
        version.into(),
    )
}

fn process_header_line(s: &str) -> (String, String) {
    let mut header_items = s.split(":");
    let mut key = String::from("");
    let mut value = String::from("");
    if let Some(k) = header_items.next() {
        key = k.to_string();
    }
    if let Some(v) = header_items.next() {
        value = v.to_string();
    }

    (key, value)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_method_into() {
        let m: Method = "GET".into();
        assert_eq!(m, Method::Get);
    }

    #[test]
    fn test_version_into() {
        let v: Version = "HTTP/1.1".into();
        assert_eq!(v, Version::V1_1);
    }

    #[test]
    fn test_read_http() {
        let s: String = String::from("GET /greeting HTTP/1.1\r\nHost: localhost:3000\r\nUser-Agent: curl/7.71.1\r\nAccept: */*\r\n\r\n");
        let mut headers_expected = HashMap::new();
        headers_expected.insert("Host".into(), " localhost".into());
        headers_expected.insert("Accept".into(), " */*".into());
        headers_expected.insert("User-Agent".into(), " curl/7.71.1".into());
        let req: HttpRequest = s.into();

        assert_eq!(Method::Get, req.method);
        assert_eq!(Version::V1_1, req.version);
        assert_eq!(Resource::Path("/greeting".to_string()), req.resource);
        assert_eq!(headers_expected, req.headers);
    }
}

```

测试

```bash
s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cargo test -p http
   Compiling http v0.1.0 (/Users/qiaopengjun/rust/s1/http)
    Finished test [unoptimized + debuginfo] target(s) in 0.37s
     Running unittests src/lib.rs (target/debug/deps/http-ca93876a5b13f05e)

running 3 tests
test httprequest::tests::test_method_into ... ok
test httprequest::tests::test_version_into ... ok
test httprequest::tests::test_read_http ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests http

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 
```

## 三、构建HTTP响应

### HTTP响应

Structure of an HTTP Response

Status Line   Version  Status code    Status text

Header Line 1

Header Line 2

Empty line

Message body(optional)

### HttpResponse 需要实现的方法

| 需要实现的方法或trait | 用途                            |
| --------------------- | ------------------------------- |
| Default trait         | 指定成员的默认值                |
| new()                 | 使用默认值创建一个新的结构体    |
| send_response()       | 构建响应，将原始字节通过TCP传送 |
| getter 方法           | 获得成员的值                    |
| From trait            | 能够将HttpResponse转化为String  |

http/lib.rs

```rust
pub mod httprequest;
pub mod httpresponse;

```

http/src/httpresponse.rs

```rust
use std::collections::HashMap;
use std::io::{Result, Write};

#[derive(Debug, PartialEq, Clone)]
pub struct HttpResponse<'a> {
    version: &'a str,
    status_code: &'a str,
    status_text: &'a str,
    headers: Option<HashMap<&'a str, &'a str>>,
    body: Option<String>,
}

impl<'a> Default for HttpResponse<'a> {
    fn default() -> Self {
        Self {
            version: "HTTP/1.1".into(),
            status_code: "200".into(),
            status_text: "OK".into(),
            headers: None,
            body: None,
        }
    }
}

impl<'a> From<HttpResponse<'a>> for String {
    fn from(res: HttpResponse<'a>) -> String {
        let res1 = res.clone();
        format!(
            "{} {} {}\r\n{}Content-Length: {}\r\n\r\n{}",
            &res1.version(),
            &res1.status_code(),
            &res1.status_text(),
            &res1.headers(),
            &res.body.unwrap().len(),
            &res1.body()
        )
    }
}

impl<'a> HttpResponse<'a> {
    pub fn new(
        status_code: &'a str,
        headers: Option<HashMap<&'a str, &'a str>>,
        body: Option<String>,
    ) -> HttpResponse<'a> {
        let mut response: HttpResponse<'a> = HttpResponse::default();
        if status_code != "200" {
            response.status_code = status_code.into();
        };
        response.headers = match &headers {
            Some(_h) => headers,
            None => {
                let mut h = HashMap::new();
                h.insert("Content-Type", "text/html");
                Some(h)
            }
        };
        response.status_text = match response.status_code {
            "200" => "OK".into(),
            "400" => "Bad Request".into(),
            "404" => "Not Found".into(),
            "500" => "Internal Server Error".into(),
            _ => "Not Found".into(),
        };

        response.body = body;

        response
    }

    pub fn send_response(&self, write_stream: &mut impl Write) -> Result<()> {
        let res = self.clone();
        let response_string: String = String::from(res);
        let _ = write!(write_stream, "{}", response_string);

        Ok(())
    }

    fn version(&self) -> &str {
        self.version
    }

    fn status_code(&self) -> &str {
        self.status_code
    }

    fn status_text(&self) -> &str {
        self.status_text
    }

    fn headers(&self) -> String {
        let map: HashMap<&str, &str> = self.headers.clone().unwrap();
        let mut header_string: String = "".into();
        for (k, v) in map.iter() {
            header_string = format!("{}{}:{}\r\n", header_string, k, v);
        }
        header_string
    }

    pub fn body(&self) -> &str {
        match &self.body {
            Some(b) => b.as_str(),
            None => "",
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_response_struct_creation_200() {
        let response_actual = HttpResponse::new("200", None, Some("xxxx".into()));
        let response_expected = HttpResponse {
            version: "HTTP/1.1",
            status_code: "200",
            status_text: "OK",
            headers: {
                let mut h = HashMap::new();
                h.insert("Content-Type", "text/html");
                Some(h)
            },
            body: Some("xxxx".into()),
        };
        assert_eq!(response_actual, response_expected);
    }

    #[test]
    fn test_response_struct_creation_404() {
        let response_actual = HttpResponse::new("404", None, Some("xxxx".into()));
        let response_expected = HttpResponse {
            version: "HTTP/1.1",
            status_code: "404",
            status_text: "Not Found",
            headers: {
                let mut h = HashMap::new();
                h.insert("Content-Type", "text/html");
                Some(h)
            },
            body: Some("xxxx".into()),
        };
        assert_eq!(response_actual, response_expected);
    }

    #[test]
    fn test_http_response_creation() {
        let response_expected = HttpResponse {
            version: "HTTP/1.1",
            status_code: "404",
            status_text: "Not Found",
            headers: {
                let mut h = HashMap::new();
                h.insert("Content-Type", "text/html");
                Some(h)
            },
            body: Some("xxxx".into()),
        };
        let http_string: String = response_expected.into();
        let actual_string =
            "HTTP/1.1 404 Not Found\r\nContent-Type:text/html\r\nContent-Length: 4\r\n\r\nxxxx";
        assert_eq!(http_string, actual_string);
    }
}

```

测试

```bash
s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cargo test -p http
   Compiling http v0.1.0 (/Users/qiaopengjun/rust/s1/http)
    Finished test [unoptimized + debuginfo] target(s) in 0.70s
     Running unittests src/lib.rs (target/debug/deps/http-ca93876a5b13f05e)

running 6 tests
test httprequest::tests::test_version_into ... ok
test httprequest::tests::test_method_into ... ok
test httpresponse::tests::test_response_struct_creation_200 ... ok
test httpresponse::tests::test_response_struct_creation_404 ... ok
test httpresponse::tests::test_http_response_creation ... ok
test httprequest::tests::test_read_http ... ok

test result: ok. 6 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests http

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 
```

## 四、构建 server 模块

目录

```bash
s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ tree
.
├── Cargo.lock
├── Cargo.toml
├── http
│   ├── Cargo.toml
│   └── src
│       ├── httprequest.rs
│       ├── httpresponse.rs
│       └── lib.rs
├── httpserver
│   ├── Cargo.toml
│   └── src
│       ├── handler.rs
│       ├── main.rs
│       ├── router.rs
│       └── server.rs
├── src
│   └── main.rs
├── target
│   ├── CACHEDIR.TAG
│   └── debug
│       ├── build
│       ├── deps
│       ├── examples
│       ├── httpserver
│       ├── httpserver.d
│       ├── incremental
│       ├── libhttp.d
│       ├── libhttp.rlib
│       ├── tcpclient
│       ├── tcpclient.d
│       ├── tcpserver
│       └── tcpserver.d
├── tcpclient
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── tcpserver
    ├── Cargo.toml
    └── src
        └── main.rs

58 directories, 532 files

s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 
```

httpserver/Cargo.toml

```toml
[package]
name = "httpserver"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
http = {path = "../http"}

```

httpserver/main.rs

```rust
mod handler;
mod router;
mod server;

use server::Server;

fn main() {
    let server = Server::new("localhost:3000");
    server.run();
}

```

httpserver/server.rs

```rust
use super::router::Router;
use http::httprequest::HttpRequest;
use std::io::prelude::*;
use std::net::TcpListener;
use std::str;

pub struct Server<'a> {
    socket_addr: &'a str,
}

impl<'a> Server<'a> {
    pub fn new(socket_addr: &'a str) -> Self {
        Server { socket_addr }
    }

    pub fn run(&self) {
        let connection_listener = TcpListener::bind(self.socket_addr).unwrap();
        println!("Running on {}", self.socket_addr);

        for stream in connection_listener.incoming() {
            let mut stream = stream.unwrap();
            println!("Connection established");

            let mut read_buffer = [0; 200];
            stream.read(&mut read_buffer).unwrap();

            let req: HttpRequest = String::from_utf8(read_buffer.to_vec()).unwrap().into();
            Router::route(req, &mut stream);
        }
    }
}

```

## 五、构建 router 和 handler 模块

目录

```bash
s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ tree
.
├── Cargo.lock
├── Cargo.toml
├── http
│   ├── Cargo.toml
│   └── src
│       ├── httprequest.rs
│       ├── httpresponse.rs
│       └── lib.rs
├── httpserver
│   ├── Cargo.toml
│   ├── data
│   │   └── orders.json
│   ├── public
│   │   ├── 404.html
│   │   ├── health.html
│   │   ├── index.html
│   │   └── styles.css
│   └── src
│       ├── handler.rs
│       ├── main.rs
│       ├── router.rs
│       └── server.rs
├── src
│   └── main.rs
├── target
│   ├── CACHEDIR.TAG
│   └── debug
│       ├── build
│       ├── deps
│       ├── libhttp.d
│       ├── libhttp.rlib
│       ├── tcpclient
│       ├── tcpclient.d
│       ├── tcpserver
│       └── tcpserver.d
├── tcpclient
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── tcpserver
    ├── Cargo.toml
    └── src
        └── main.rs

74 directories, 805 files

s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 
```

httpserver/Cargo.toml

```toml
[package]
name = "httpserver"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
http = {path = "../http"}
serde = {version="1.0.163", features = ["derive"]}
serde_json = "1.0.96"

```

httpserver/src/handler.rs

```rust
use http::{httprequest::HttpRequest, httpresponse::HttpResponse};
use serde::{Deserialize, Serialize};
use std::collections::HashMap;
use std::env;
use std::fs;

pub trait Handler {
    fn handle(req: &HttpRequest) -> HttpResponse;
    fn load_file(file_name: &str) -> Option<String> {
        let default_path = format!("{}/public", env!("CARGO_MANIFEST_DIR"));
        let public_path = env::var("PUBLIC_PATH").unwrap_or(default_path);
        let full_path = format!("{}/{}", public_path, file_name);

        let contents = fs::read_to_string(full_path);
        contents.ok()
    }
}

pub struct StaticPageHandler;
pub struct PageNotFoundHandler;
pub struct WebServiceHandler;

#[derive(Serialize, Deserialize)]
pub struct OrderStatus {
    order_id: i32,
    order_date: String,
    order_status: String,
}

impl Handler for PageNotFoundHandler {
    fn handle(_req: &HttpRequest) -> HttpResponse {
        HttpResponse::new("404", None, Self::load_file("404.html"))
    }
}

impl Handler for StaticPageHandler {
    fn handle(req: &HttpRequest) -> HttpResponse {
        let http::httprequest::Resource::Path(s) = &req.resource;
        let route: Vec<&str> = s.split("/").collect();
        match route[1] {
            "" => HttpResponse::new("200", None, Self::load_file("index.html")),
            "health" => HttpResponse::new("200", None, Self::load_file("health.html")),
            path => match Self::load_file(path) {
                Some(contents) => {
                    let mut map: HashMap<&str, &str> = HashMap::new();
                    if path.ends_with(".css") {
                        map.insert("Content-Type", "text/css");
                    } else if path.ends_with(".js") {
                        map.insert("Content-Type", "text/javascript");
                    } else {
                        map.insert("Content-Type", "text/html");
                    }
                    HttpResponse::new("200", Some(map), Some(contents))
                }
                None => HttpResponse::new("404", None, Self::load_file("404.html")),
            },
        }
    }
}

impl WebServiceHandler {
    fn load_json() -> Vec<OrderStatus> {
        let default_path = format!("{}/data", env!("CARGO_MANIFEST_DIR"));
        let data_path = env::var("DATA_PATH").unwrap_or(default_path);
        let full_path = format!("{}/{}", data_path, "orders.json");
        let json_contents = fs::read_to_string(full_path);
        let orders: Vec<OrderStatus> =
            serde_json::from_str(json_contents.unwrap().as_str()).unwrap();
        orders
    }
}

impl Handler for WebServiceHandler {
    fn handle(req: &HttpRequest) -> HttpResponse {
        let http::httprequest::Resource::Path(s) = &req.resource;
        let route: Vec<&str> = s.split("/").collect();
        // localhost:3000/api/shipping/orders
        match route[2] {
            "shipping" if route.len() > 2 && route[3] == "orders" => {
                let body = Some(serde_json::to_string(&Self::load_json()).unwrap());
                let mut headers: HashMap<&str, &str> = HashMap::new();
                headers.insert("Content-Type", "application/json");
                HttpResponse::new("200", Some(headers), body)
            }
            _ => HttpResponse::new("404", None, Self::load_file("404.html")),
        }
    }
}

```

httpserver/src/server.rs

```rust
use super::router::Router;
use http::httprequest::HttpRequest;
use std::io::prelude::*;
use std::net::TcpListener;
use std::str;

pub struct Server<'a> {
    socket_addr: &'a str,
}

impl<'a> Server<'a> {
    pub fn new(socket_addr: &'a str) -> Self {
        Server { socket_addr }
    }

    pub fn run(&self) {
        let connection_listener = TcpListener::bind(self.socket_addr).unwrap();
        println!("Running on {}", self.socket_addr);

        for stream in connection_listener.incoming() {
            let mut stream = stream.unwrap();
            println!("Connection established");

            let mut read_buffer = [0; 200];
            stream.read(&mut read_buffer).unwrap();

            let req: HttpRequest = String::from_utf8(read_buffer.to_vec()).unwrap().into();
            Router::route(req, &mut stream);
        }
    }
}

```

httpserver/src/router.rs

```rust
use super::handler::{Handler, PageNotFoundHandler, StaticPageHandler, WebServiceHandler};
use http::{httprequest, httprequest::HttpRequest, httpresponse::HttpResponse};
use std::io::prelude::*;

pub struct Router;

impl Router {
    pub fn route(req: HttpRequest, stream: &mut impl Write) -> () {
        match req.method {
            httprequest::Method::Get => match &req.resource {
                httprequest::Resource::Path(s) => {
                    let route: Vec<&str> = s.split("/").collect();
                    match route[1] {
                        "api" => {
                            let resp: HttpResponse = WebServiceHandler::handle(&req);
                            let _ = resp.send_response(stream);
                        }
                        _ => {
                            let resp: HttpResponse = StaticPageHandler::handle(&req);
                            let _ = resp.send_response(stream);
                        }
                    }
                }
            },
            _ => {
                let resp: HttpResponse = PageNotFoundHandler::handle(&req);
                let _ = resp.send_response(stream);
            }
        }
    }
}

```

httpserver/data/orders.json

```json
[
    {
        "order_id": 1,
        "order_date": "21 Jan 2023",
        "order_status": "Delivered"
    },
    {
        "order_id": 2,
        "order_date": "2 Feb 2023",
        "order_status": "Pending"
    }
]

```

httpserver/public/404.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Not Found!</title>
</head>

<body>
    <h1>404 Error</h1>
    <p>Sorry the requested page does not exist</p>
</body>

</html>

```

httpserver/public/health.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Health!</title>
</head>

<body>
    <h1>Hello welcome to health page!</h1>
    <p>This site is perfectly fine</p>
</body>

</html>

```

httpserver/public/index.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="styles.css">
    <title>Index!</title>
</head>

<body>
    <h1>Hello, welcome to home page</h1>
    <p>This is the index page for the web site</p>
</body>

</html>

```

httpserver/public/styles.css

```css
h1 {
    color: red;
    margin-left: 25px;
}

```

### 测试该项目

运行

```bash
s1 on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cargo run -p httpserver
   Compiling ryu v1.0.13
   Compiling itoa v1.0.6
   Compiling serde v1.0.163
   Compiling serde_json v1.0.96
   Compiling httpserver v0.1.0 (/Users/qiaopengjun/rust/s1/httpserver)
    Finished dev [unoptimized + debuginfo] target(s) in 2.48s
     Running `target/debug/httpserver`
Running on localhost:3000
Connection established
Connection established
Connection established
Connection established
Connection established
Connection established
Connection established

```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202305281055490.png)

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202305281055396.png)

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202305281055395.png)

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202305281054809.png)
