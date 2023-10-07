---
title: "Rust 编程学习 001"
date: 2023-06-29T16:11:20+08:00
draft: false
tags: ["Rust"]
categories: ["Rust"]
---

# Rust 编程学习 001

## 需求

通过 HTTP 请求 Rust 官网首页，然后把获得的 HTML 转换成 Markdown 保存起来。

## 实操

首先，我们用 cargo new scrape_url 生成一个新项目。默认情况下，这条命令会生成一个可执行项目 scrape_url，入口在 src/main.rs。

```bash
~/Code via 🅒 base
➜ mcd rust  # mkdir rust cd rust

~/Code/rust via 🅒 base
➜

~/Code/rust via 🅒 base
➜ cargo new scrape_url
     Created binary (application) `scrape_url` package

~/Code/rust via 🅒 base
➜ ls
scrape_url

~/Code/rust via 🅒 base
➜ cd scrape_url

scrape_url on  master [?] via 🦀 1.70.0 via 🅒 base
➜ c

scrape_url on  master [?] via 🦀 1.70.0 via 🅒 base
➜

```

我们在 Cargo.toml 文件里，加入如下的依赖：

Cargo.toml 是 Rust 项目的配置管理文件，它符合 toml 的语法。我们为这个项目添加了两个依赖：reqwest 和 html2md。reqwest 是一个 HTTP 客户端，它的使用方式和 Python 下的 request 类似；html2md 顾名思义，把 HTML 文本转换成 Markdown。

```toml
[package]
name = "scrape_url"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
reqwest ={ version = "0.11.18", features = ["blocking"] }
html2md = "0.2.14"

```

接下来，在 src/main.rs 里，我们为 main() 函数加入以下代码：

```rust
use std::fs;

fn main() {
    let url = "https://www.rust-lang.org/";
    let output = "rust.md";

    println!("Fetching url: {}", url);
    let body = reqwest::blocking::get(url).unwrap().text().unwrap();

    println!("Converting html to markdown...");
    let md = html2md::parse_html(&body);
    
    fs::write(output, md.as_bytes()).unwrap();
    println!("Converted markdown has been saved in {}", output);
}

```

保存后，在命令行下，进入这个项目的目录，运行 cargo run，在一段略微漫长的编译后，程序开始运行，在命令行下，你会看到如下的输出：

```bash
scrape_url on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base 
➜ cargo run           
   Compiling cfg-if v1.0.0
   Compiling siphasher v0.3.10
   Compiling once_cell v1.18.0
   Compiling pin-project-lite v0.2.9
   Compiling libc v0.2.147
   Compiling bytes v1.4.0
   Compiling serde v1.0.164
   Compiling memchr v2.5.0
   Compiling smallvec v1.10.0
   Compiling new_debug_unreachable v1.0.4
   Compiling scopeguard v1.1.0
   Compiling futures-core v0.3.28
   Compiling getrandom v0.2.10
   Compiling core-foundation-sys v0.8.4
   Compiling lock_api v0.4.10
   Compiling mac v0.1.1
   Compiling phf_shared v0.10.0
   Compiling itoa v1.0.6
   Compiling rand_core v0.6.4
   Compiling bitflags v1.3.2
   Compiling futf v0.1.5
   Compiling tracing-core v0.1.31
   Compiling log v0.4.19
   Compiling precomputed-hash v0.1.1
   Compiling fnv v1.0.7
   Compiling utf-8 v0.7.6
   Compiling rand_chacha v0.3.1
   Compiling http v0.2.9
   Compiling phf v0.10.1
   Compiling slab v0.4.8
   Compiling tendril v0.4.3
   Compiling futures-task v0.3.28
   Compiling tinyvec_macros v0.1.1
   Compiling percent-encoding v2.3.0
   Compiling fastrand v1.9.0
   Compiling rand v0.8.5
   Compiling pin-utils v0.1.0
   Compiling socket2 v0.4.9
   Compiling parking_lot_core v0.9.8
   Compiling num_cpus v1.15.0
   Compiling mio v0.8.8
   Compiling io-lifetimes v1.0.11
   Compiling errno v0.3.1
   Compiling tracing v0.1.37
   Compiling parking_lot v0.12.1
   Compiling rustix v0.37.20
   Compiling security-framework-sys v2.9.0
   Compiling core-foundation v0.9.3
   Compiling lazy_static v1.4.0
   Compiling hashbrown v0.12.3
   Compiling tokio v1.29.0
   Compiling futures-io v0.3.28
   Compiling futures-sink v0.3.28
   Compiling phf_generator v0.10.0
   Compiling security-framework v2.9.1
   Compiling futures-util v0.3.28
   Compiling phf_codegen v0.10.0
   Compiling string_cache_codegen v0.5.2
   Compiling tinyvec v1.6.0
   Compiling try-lock v0.2.4
   Compiling httparse v1.8.0
   Compiling indexmap v1.9.3
   Compiling want v0.3.1
   Compiling futures-channel v0.3.28
   Compiling tempfile v3.6.0
   Compiling http-body v0.4.5
   Compiling form_urlencoded v1.2.0
   Compiling markup5ever v0.11.0
   Compiling unicode-bidi v0.3.13
   Compiling tower-service v0.3.2
   Compiling native-tls v0.2.11
   Compiling httpdate v1.0.2
   Compiling aho-corasick v1.0.2
   Compiling regex-syntax v0.7.2
   Compiling string_cache v0.8.7
   Compiling unicode-normalization v0.1.22
   Compiling ryu v1.0.13
   Compiling encoding_rs v0.8.32
   Compiling ipnet v2.8.0
   Compiling serde_urlencoded v0.7.1
   Compiling mime v0.3.17
   Compiling idna v0.4.0
   Compiling base64 v0.21.2
   Compiling url v2.4.0
   Compiling regex v1.8.4
   Compiling tokio-util v0.7.8
   Compiling tokio-native-tls v0.3.1
   Compiling h2 v0.3.20
   Compiling html5ever v0.26.0
   Compiling xml5ever v0.17.0
   Compiling markup5ever_rcdom v0.2.0
   Compiling html2md v0.2.14
   Compiling hyper v0.14.27
   Compiling hyper-tls v0.5.0
   Compiling reqwest v0.11.18
   Compiling scrape_url v0.1.0 (/Users/qiaopengjun/Code/rust/scrape_url)
    Finished dev [unoptimized + debuginfo] target(s) in 7.14s
     Running `target/debug/scrape_url`
Fetching url: https://www.rust-lang.org/
Converting html to markdown...
Converted markdown has been saved in rust.md

scrape_url on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 9.1s 
➜ 
```

并且，在当前目录下，一个 rust.md 文件被创建出来了。打开一看，其内容就是 Rust 官网主页的内容。

从这段并不长的代码中，我们可以感受到 Rust 的一些基本特点：

首先，Rust 使用名为 cargo 的工具来管理项目，它类似 Node.js 的 npm、Golang 的 go，用来做依赖管理以及开发过程中的任务管理，比如编译、运行、测试、代码格式化等等。

其次，Rust 的整体语法偏 C/C++ 风格。函数体用花括号 {} 包裹，表达式之间用分号 ; 分隔，访问结构体的成员函数或者变量使用点 . 运算符，而访问命名空间（namespace）或者对象的静态函数使用双冒号 :: 运算符。如果要简化对命名空间内部的函数或者数据类型的引用，可以使用 use 关键字，比如 use std::fs。此外，可执行体的入口函数是 main()。

另外，你也很容易看到，Rust 虽然是一门强类型语言，但编译器支持类型推导，这使得写代码时的直观感受和写脚本语言差不多。

最后，Rust 支持宏编程，很多基础的功能比如 println!() 都被封装成一个宏，便于开发者写出简洁的代码。

这里例子没有展现出来，但 Rust 还具备的其它特点有：

- Rust 的变量默认是不可变的，如果要修改变量的值，需要显式地使用 mut 关键字。
- 除了 let / static / const / fn 等少数语句外，Rust 绝大多数代码都是表达式（expression）。所以 if / while / for / loop 都会返回一个值，函数最后一个表达式就是函数的返回值，这和函数式编程语言一致。
- Rust 支持面向接口编程和泛型编程。
- Rust 有非常丰富的数据类型和强大的标准库。
- Rust 有非常丰富的控制流程，包括模式匹配（pattern match）。

<https://static001.geekbang.org/resource/image/15/cb/15e5152fe2b72794074cff40041722cb.jpg?wh=1920x1898>

如果想让错误传播，可以把所有的 unwrap() 换成 ? 操作符，并让 main() 函数返回一个 Result，如下所示：

```rust
use std::fs;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let url = "https://www.rust-lang.org/";
    let output = "rust.md";

    println!("Fetching url: {}", url);
    let body = reqwest::blocking::get(url)?.text()?;

    println!("Converting html to markdown...");
    let md = html2md::parse_html(&body);
    
    fs::write(output, md.as_bytes())?;
    println!("Converted markdown has been saved in {}", output);

    Ok(())
}

```

从命令行参数中获取用户提供的信息来绑定 URL 和文件名

```rust
use std::env;
use std::fs;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let args: Vec<String> = env::args().collect();
    let url = &args[1];
    let output = &args[2];
    // let url = "https://www.rust-lang.org/";
    // let output = "rust.md";

    println!("Fetching url: {}", url);
    let body = reqwest::blocking::get(url)?.text()?;

    println!("Converting html to markdown...");
    let md = html2md::parse_html(&body);

    fs::write(output, md.as_bytes())?;
    println!("Converted markdown has been saved in {}", output);

    Ok(())
}

```

运行

```bash
scrape_url on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base 
➜ cargo run https://www.rust-lang.org/ rust1.md
   Compiling scrape_url v0.1.0 (/Users/qiaopengjun/Code/rust/scrape_url)
    Finished dev [unoptimized + debuginfo] target(s) in 1.48s
     Running `target/debug/scrape_url 'https://www.rust-lang.org/' rust1.md`
Fetching url: https://www.rust-lang.org/
Converting html to markdown...
Converted markdown has been saved in rust1.md

scrape_url on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 3.5s 
➜ 

```
