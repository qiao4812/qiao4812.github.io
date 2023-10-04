---
title: "Rust Web 全栈开发之编写 WebAssembly 应用"
date: 2023-06-03T13:37:26+08:00
draft: true
tags: ["Rust"]
categories: ["Rust"]
---

# Rust Web 全栈开发之编写 WebAssembly 应用

MDN Web Docs：<https://developer.mozilla.org/zh-CN/docs/WebAssembly>

官网：<https://webassembly.org/>

## 项目结构 和 功能

**Web App 教师注册    <->    WebService    <->    WebAssembly App  课程管理**

## 什么是 WebAssembly

- WebAssembly 是一种新的编码方式，可以在现代浏览器中运行
  -  它是一种低级的类汇编语言
  -  具有紧凑的二进制格式
  -  可以接近原生的性能运行
  -  并为 C/C ++ 、 C# 、 Rust 等语言提供一个编译目标，以便它们可以在 Web上运行
  -  它也被设计为可以与 JavaScript 共存，允许两者一起工作。

## 机器码

- 机器码是计算机可直接执行的语言
- 汇编语言比较接近机器码

**ASSEMBLY    ->    ASSEMBLER    ->    MACHINE CODE**

## 机器码与 CPU 架构

- 不同的 CPU 架构需要不同的机器码和汇编
- 高级语言可以“翻译”成机器码，以便在 CPU 上运行
- SOURCE CODE
  - x64 
  - x86
  - ARM

## WebAssembly

- WebAssembly 其实不是汇编语言，它不针对特定的机器，而是针对浏览器的。
- WebAssembly 是中间编译器目标
- SOURCE CODE
  - WASM
    - x64 
    - x86
    - ARM

## WebAssembly 是什么样的？

- 文本格式 .wat
- 二进制格式： .wasm

## WebAssembly 能做什么

- 可以把你编写 C/C++ 、 C# 、 Rust 等语言的代码编译成 WebAssembly 模块
- 你可以在 Web 应用中加载该模块，并通过 JavaScript 调用它
- 它并不是为了替代 JS ，而是与 JS 一起工作
- 仍然需要 HTML 和 JS ，因为WebAssembly 无法访问平台 API ，例如 DOM ， WebGL...

## WebAssembly 如何工作

- 这是 C/C++ 的例子

Hello.c    ->    EMSCRIPTEN（编译器）   ->  hello.wasm    hello.js  hello.html

## WebAssembly 的优点

- 快速、高效、可移植
  -  通过利用常见的硬件能力， WebAssembly 代码在不同平台上能够以接近本地速度运行。
-  可读、可调试
  -  WebAssembly 是一门低阶语言，但是它有确实有一种人类可读的文本格式（其标准最终版本目前仍在编制），这允许通过手工来写代码，看代码以及调试代码。
-  保持安全
  -  WebAssembly 被限制运行在一个安全的沙箱执行环境中。像其他网络代码一样，它遵循浏览器的同源策略和授权策略。
-  不破坏网络
  -  WebAssembly 的设计原则是与其他网络技术和谐共处并保持向后兼容。

## Rust WebAssembly 部分相关 crate

- wasm-bindgen
-  wasm-bindgen-future
-  web-sys
-  js-sys

## 搭建环境

Rust 官网：<https://www.rust-lang.org/zh-CN/what/wasm>

Rust and WebAssembly：<https://rustwasm.github.io/docs/book/>

Rust和WebAssembly中文文档：<https://rustwasm.wasmdev.cn/docs/book/>

### [安装](https://rustwasm.wasmdev.cn/docs/book/game-of-life/setup.html#安装)

#### [`wasm-pack`](https://rustwasm.github.io/docs/book/game-of-life/setup.html#wasm-pack)

下载安装地址：<https://rustwasm.github.io/wasm-pack/installer/>

Install `wasm-pack`

You appear to be running a *nix system (Unix, Linux, MacOS). Install by running:

```bash
curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh
```

If you're not on *nix, or you don't like installing from **curl**, follow the alternate instructions below.

#### `cargo-generate`

[`cargo-generate` helps you get up and running quickly with a new Rust project by leveraging a pre-existing git repository as a template.](https://github.com/ashleygwilliams/cargo-generate)

Install `cargo-generate` with this command:

```bash
cargo install cargo-generate
```



```bash
~ via 🅒 base
➜ curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh

info: downloading wasm-pack
info: successfully installed wasm-pack to `/Users/qiaopengjun/.cargo/bin/wasm-pack`

~ via 🅒 base took 8.9s
➜

ws on  main via 🦀 1.67.1 via 🅒 base took 23.7s
➜ cargo install cargo-generate

    Updating crates.io index
warning: spurious network error (2 tries remaining): failed to connect to github.com: Operation timed out; class=Os (2)
warning: spurious network error (1 tries remaining): failed to connect to github.com: Operation timed out; class=Os (2)
  Downloaded cargo-generate v0.18.3
  Downloaded 1 crate (94.7 KB) in 0.73s
	......
   Compiling git2 v0.17.2
   Compiling cargo-generate v0.18.3
    Finished release [optimized] target(s) in 3m 48s
  Installing /Users/qiaopengjun/.cargo/bin/cargo-generate
   Installed package `cargo-generate v0.18.3` (executable `cargo-generate`)

ws on  main via 🦀 1.67.1 via 🅒 base took 3m 48.8s
➜ npm --version
9.5.0

ws on  main via 🦀 1.67.1 via 🅒 base

```

[Clone the Project Template](https://rustwasm.github.io/docs/book/game-of-life/hello-world.html#clone-the-project-template)

```bash
ws on  main via 🦀 1.67.1 via 🅒 base
➜ cd ..

~/rust via 🅒 base
➜ cargo generate --git https://github.com/rustwasm/wasm-pack-template

🤷   Project Name: wasm-game-of-life
🔧   Destination: /Users/qiaopengjun/rust/wasm-game-of-life ...
🔧   project-name: wasm-game-of-life ...
🔧   Generating template ...
🔧   Moving generated files into: `/Users/qiaopengjun/rust/wasm-game-of-life`...
Initializing a fresh Git repository
✨   Done! New project created /Users/qiaopengjun/rust/wasm-game-of-life

~/rust via 🅒 base took 21.2s
➜ cd wasm-game-of-life

wasm-game-of-life on  master [?] via 🦀 1.67.1 via 🅒 base
➜ c

wasm-game-of-life on  master [?] via 🦀 1.67.1 via 🅒 base
➜

```

构建项目

```bash
wasm-game-of-life on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 1m 23.4s 
➜ wasm-pack build

[INFO]: 🎯  Checking for the Wasm target...
info: downloading component 'rust-std' for 'wasm32-unknown-unknown'
info: installing component 'rust-std' for 'wasm32-unknown-unknown'
[INFO]: 🌀  Compiling to Wasm...
   Compiling proc-macro2 v1.0.59
   Compiling unicode-ident v1.0.9
   Compiling quote v1.0.28
   Compiling wasm-bindgen-shared v0.2.86
   Compiling log v0.4.18
   Compiling bumpalo v3.13.0
   Compiling once_cell v1.17.2
   Compiling wasm-bindgen v0.2.86
   Compiling cfg-if v1.0.0
   Compiling syn v2.0.18
   Compiling wasm-bindgen-backend v0.2.86
   Compiling wasm-bindgen-macro-support v0.2.86
   Compiling wasm-bindgen-macro v0.2.86
   Compiling console_error_panic_hook v0.1.7
   Compiling wasm-game-of-life v0.1.0 (/Users/qiaopengjun/rust/wasm-game-of-life)
warning: function `set_panic_hook` is never used
 --> src/utils.rs:1:8
  |
1 | pub fn set_panic_hook() {
  |        ^^^^^^^^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: `wasm-game-of-life` (lib) generated 1 warning
    Finished release [optimized] target(s) in 11.45s
[INFO]: ⬇️  Installing wasm-bindgen...
[INFO]: Optimizing wasm binaries with `wasm-opt`...
[INFO]: Optional fields missing from Cargo.toml: 'description', 'repository', and 'license'. These are not necessary, but recommended
[INFO]: ✨   Done in 1m 41s
[INFO]: 📦   Your wasm pkg is ready to publish at /Users/qiaopengjun/rust/wasm-game-of-life/pkg.

wasm-game-of-life on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 1m 52.0s 
➜ 
```

问题

```bash
wasm-game-of-life on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 3.8s 
➜ wasm-pack build

[INFO]: 🎯  Checking for the Wasm target...
Error: wasm32-unknown-unknown target not found in sysroot: "/opt/homebrew/Cellar/rust/1.67.1"

Used rustc from the following path: "/opt/homebrew/bin/rustc"
It looks like Rustup is not being used. For non-Rustup setups, the wasm32-unknown-unknown target needs to be installed manually. See https://rustwasm.github.io/wasm-pack/book/prerequisites/non-rustup-setups.html on how to do this.

Caused by: wasm32-unknown-unknown target not found in sysroot: "/opt/homebrew/Cellar/rust/1.67.1"

Used rustc from the following path: "/opt/homebrew/bin/rustc"
It looks like Rustup is not being used. For non-Rustup setups, the wasm32-unknown-unknown target needs to be installed manually. See https://rustwasm.github.io/wasm-pack/book/prerequisites/non-rustup-setups.html on how to do this.


wasm-game-of-life on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ 

```

解决

https://rustwasm.github.io/wasm-pack/book/prerequisites/non-rustup-setups.html

电脑中有两个Rust，默认使用的是brew install rust

通过 brew uninstall rust  卸载 Rust

并运行 `rustup self uninstall` 卸载通过官网安装的 Rust

最后重新通过官网安装Rust，再次执行命令解决问题。

Rust安装

```bash
~ via 🅒 base
➜ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

info: downloading installer

Welcome to Rust!

This will download and install the official compiler for the Rust
programming language, and its package manager, Cargo.

Rustup metadata and toolchains will be installed into the Rustup
home directory, located at:

  /Users/qiaopengjun/.rustup

This can be modified with the RUSTUP_HOME environment variable.

The Cargo home directory is located at:

  /Users/qiaopengjun/.cargo

This can be modified with the CARGO_HOME environment variable.

The cargo, rustc, rustup and other commands will be added to
Cargo's bin directory, located at:

  /Users/qiaopengjun/.cargo/bin

This path will then be added to your PATH environment variable by
modifying the profile files located at:

  /Users/qiaopengjun/.profile
  /Users/qiaopengjun/.bash_profile
  /Users/qiaopengjun/.zshenv

You can uninstall at any time with rustup self uninstall and
these changes will be reverted.

Current installation options:


   default host triple: aarch64-apple-darwin
     default toolchain: stable (default)
               profile: default
  modify PATH variable: yes

1) Proceed with installation (default)
2) Customize installation
3) Cancel installation
>1

info: profile set to 'default'
info: default host triple is aarch64-apple-darwin
info: syncing channel updates for 'stable-aarch64-apple-darwin'
info: latest update on 2023-06-01, rust version 1.70.0 (90c541806 2023-05-31)
info: downloading component 'cargo'
info: downloading component 'clippy'
info: downloading component 'rust-docs'
 13.5 MiB /  13.5 MiB (100 %)   4.6 MiB/s in  2s ETA:  0s
info: downloading component 'rust-std'
 25.5 MiB /  25.5 MiB (100 %)   4.9 MiB/s in  5s ETA:  0s
info: downloading component 'rustc'
 52.6 MiB /  52.6 MiB (100 %)   6.5 MiB/s in  8s ETA:  0s
info: downloading component 'rustfmt'
info: installing component 'cargo'
info: installing component 'clippy'
info: installing component 'rust-docs'
 13.5 MiB /  13.5 MiB (100 %)   7.9 MiB/s in  1s ETA:  0s
info: installing component 'rust-std'
 25.5 MiB /  25.5 MiB (100 %)  20.9 MiB/s in  1s ETA:  0s
info: installing component 'rustc'
 52.6 MiB /  52.6 MiB (100 %)  23.6 MiB/s in  2s ETA:  0s
info: installing component 'rustfmt'
info: default toolchain set to 'stable-aarch64-apple-darwin'

  stable-aarch64-apple-darwin installed - rustc 1.70.0 (90c541806 2023-05-31)


Rust is installed now. Great!

To get started you may need to restart your current shell.
This would reload your PATH environment variable to include
Cargo's bin directory ($HOME/.cargo/bin).

To configure your current shell, run:
source "$HOME/.cargo/env"

~ via 🅒 base took 2m 46.4s
➜ source "$HOME/.cargo/env"

~ via 🅒 base
➜ rustc --version
rustc 1.70.0 (90c541806 2023-05-31)

~ via 🅒 base
➜ rustup toolchain list
stable-aarch64-apple-darwin (default)

~ via 🅒 base
➜
```

构建

```bash
wasm-game-of-life on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 1m 23.4s 
➜ wasm-pack build

[INFO]: 🎯  Checking for the Wasm target...
info: downloading component 'rust-std' for 'wasm32-unknown-unknown'
info: installing component 'rust-std' for 'wasm32-unknown-unknown'
[INFO]: 🌀  Compiling to Wasm...
   Compiling proc-macro2 v1.0.59
   Compiling unicode-ident v1.0.9
   Compiling quote v1.0.28
   Compiling wasm-bindgen-shared v0.2.86
   Compiling log v0.4.18
   Compiling bumpalo v3.13.0
   Compiling once_cell v1.17.2
   Compiling wasm-bindgen v0.2.86
   Compiling cfg-if v1.0.0
   Compiling syn v2.0.18
   Compiling wasm-bindgen-backend v0.2.86
   Compiling wasm-bindgen-macro-support v0.2.86
   Compiling wasm-bindgen-macro v0.2.86
   Compiling console_error_panic_hook v0.1.7
   Compiling wasm-game-of-life v0.1.0 (/Users/qiaopengjun/rust/wasm-game-of-life)
warning: function `set_panic_hook` is never used
 --> src/utils.rs:1:8
  |
1 | pub fn set_panic_hook() {
  |        ^^^^^^^^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: `wasm-game-of-life` (lib) generated 1 warning
    Finished release [optimized] target(s) in 11.45s
[INFO]: ⬇️  Installing wasm-bindgen...
[INFO]: Optimizing wasm binaries with `wasm-opt`...
[INFO]: Optional fields missing from Cargo.toml: 'description', 'repository', and 'license'. These are not necessary, but recommended
[INFO]: ✨   Done in 1m 41s
[INFO]: 📦   Your wasm pkg is ready to publish at /Users/qiaopengjun/rust/wasm-game-of-life/pkg.

wasm-game-of-life on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 1m 52.0s 
➜ 
```

### Npm  初始化项目

npm 中文网：https://npm.nodejs.cn/

```bash
wasm-game-of-life on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 24.9s 
➜ npm init wasm-app www
🦀 Rust + 🕸 Wasm = ❤

wasm-game-of-life on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 22.7s 
➜ 
```

www/package.json

```json
{
  "name": "create-wasm-app",
  "version": "0.1.0",
  "description": "create an app to consume rust-generated wasm packages",
  "main": "index.js",
  "bin": {
    "create-wasm-app": ".bin/create-wasm-app.js"
  },
  "scripts": {
    "build": "webpack --config webpack.config.js",
    "start": "webpack-dev-server"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/rustwasm/create-wasm-app.git"
  },
  "keywords": [
    "webassembly",
    "wasm",
    "rust",
    "webpack"
  ],
  "author": "Ashley Williams <ashley666ashley@gmail.com>",
  "license": "(MIT OR Apache-2.0)",
  "bugs": {
    "url": "https://github.com/rustwasm/create-wasm-app/issues"
  },
  "homepage": "https://github.com/rustwasm/create-wasm-app#readme",
  "dependencies": {
    "wasm-game-of-life": "file:../pkg"
  },
  "devDependencies": {
    "hello-wasm-pack": "^0.1.0",
    "webpack": "^4.29.3",
    "webpack-cli": "^3.1.0",
    "webpack-dev-server": "^3.1.5",
    "copy-webpack-plugin": "^5.0.0"
  }
}

```

安装依赖

```bash
wasm-game-of-life on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 22.7s 
➜ cd www                                                               

www on  master [!] is 📦 0.1.0 via ⬢ v19.7.0 via 🅒 base 
➜ npm install        
```

www/index.js

```js
import * as wasm from "wasm-game-of-life";

wasm.greet();

```

运行

```bash
www on  master [!] is 📦 0.1.0 via ⬢ v19.7.0 via 🅒 base took 2m 15.4s 
➜ npm run start
```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306031646191.png)

访问：http://localhost:8080/

src/lib.rs

```rust
mod utils;

use wasm_bindgen::prelude::*;

// When the `wee_alloc` feature is enabled, use `wee_alloc` as the global
// allocator.
#[cfg(feature = "wee_alloc")]
#[global_allocator]
static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet(s : &str) {
    alert(format!("Hello, {}!", s).as_str());
}

```

构建

```bash
wasm-game-of-life on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base 
➜ wasm-pack build
[INFO]: 🎯  Checking for the Wasm target...
[INFO]: 🌀  Compiling to Wasm...
   Compiling wasm-game-of-life v0.1.0 (/Users/qiaopengjun/rust/wasm-game-of-life)
warning: function `set_panic_hook` is never used
 --> src/utils.rs:1:8
  |
1 | pub fn set_panic_hook() {
  |        ^^^^^^^^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: `wasm-game-of-life` (lib) generated 1 warning
    Finished release [optimized] target(s) in 0.37s
[INFO]: ⬇️  Installing wasm-bindgen...
[INFO]: Optimizing wasm binaries with `wasm-opt`...
[INFO]: Optional fields missing from Cargo.toml: 'description', 'repository', and 'license'. These are not necessary, but recommended
[INFO]: ✨   Done in 0.80s
[INFO]: 📦   Your wasm pkg is ready to publish at /Users/qiaopengjun/rust/wasm-game-of-life/pkg.

wasm-game-of-life on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base 
➜ 
```

安装并运行

```bash
www on  master [!] is 📦 0.1.0 via ⬢ v19.7.0 via 🅒 base took 8m 0.8s 
➜ npm install && npm run start
npm WARN deprecated fsevents@1.2.9: The v1 package contains DANGEROUS / INSECURE binaries. Upgrade to safe fsevents v2
```

www/index.js

```js
import * as wasm from "wasm-game-of-life";

wasm.greet("Dave");

```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306031657650.png)

## 打开项目 ws

webservice/Cargo.toml

```toml
[package]
name = "webservice"
version = "0.1.0"
edition = "2021"
default-run = "teacher-service"
# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
actix-cors = "0.6.4"
actix-web = "4.3.1"
actix-rt = "2.8.0"
dotenv = "0.15.0"
serde = { version = "1.0.163", features = ["derive"] }
chrono = { version = "0.4.24", features = ["serde"] }
openssl = { version = "0.10.52", features = ["vendored"] }
sqlx = { version = "0.6.3", default_features = false, features = [
    "postgres",
    "runtime-tokio-rustls",
    "macros",
    "chrono",
] }


[[bin]]
name = "teacher-service"

```

#### webservice/src/bin/teacher-service.rs

```rust
use actix_cors::Cors;
use actix_web::{http, web, App, HttpServer};
use dotenv::dotenv;
use sqlx::postgres::PgPoolOptions;
use std::env;
use std::io;
use std::sync::Mutex;

#[path = "../dbaccess/mod.rs"]
mod dbaccess;
#[path = "../errors.rs"]
mod errors;
#[path = "../handlers/mod.rs"]
mod handlers;
#[path = "../models/mod.rs"]
mod models;
#[path = "../routers.rs"]
mod routers;
#[path = "../state.rs"]
mod state;

use errors::MyError;
use routers::*;
use state::AppState;

#[actix_rt::main]
async fn main() -> io::Result<()> {
    dotenv().ok();

    let database_url = env::var("DATABASE_URL").expect("DATABASE_URL is not set.");
    let db_pool = PgPoolOptions::new().connect(&database_url).await.unwrap();
    let shared_data = web::Data::new(AppState {
        health_check_response: "I'm Ok.".to_string(),
        visit_count: Mutex::new(0),
        db: db_pool,
    });
    let app = move || {
        let cors = Cors::default()
            .allowed_origin("http://localhost:8080/")
            .allowed_origin_fn(|origin, _req_head| {
                origin.as_bytes().starts_with(b"http://localhost")
            })
            .allowed_methods(vec!["GET", "POST", "DELETE"])
            .allowed_headers(vec![http::header::AUTHORIZATION, http::header::ACCEPT])
            .allowed_header(http::header::CONTENT_TYPE)
            .max_age(3600);

        App::new()
            .app_data(shared_data.clone())
            .app_data(web::JsonConfig::default().error_handler(|_err, _req| {
                MyError::InvalidInput("Please provide valid json input".to_string()).into()
            }))
            .configure(general_routes)
            .configure(course_routes) // 路由注册
            .wrap(cors)
            .configure(teacher_routes)
    };

    HttpServer::new(app).bind("127.0.0.1:3000")?.run().await
}

```

Wasm [克隆项目模板](https://rustwasm.wasmdev.cn/docs/book/game-of-life/hello-world.html#克隆项目模板)

```bash
ws on  main via 🦀 1.70.0 via 🅒 base 
➜ cargo generate --git https://github.com/rustwasm/wasm-pack-template

🤷   Project Name: wasm-client
🔧   Destination: /Users/qiaopengjun/rust/ws/wasm-client ...
🔧   project-name: wasm-client ...
🔧   Generating template ...
🔧   Moving generated files into: `/Users/qiaopengjun/rust/ws/wasm-client`...
Initializing a fresh Git repository
✨   Done! New project created /Users/qiaopengjun/rust/ws/wasm-client

ws on  main [!?] via 🦀 1.70.0 via 🅒 base took 29.2s 
➜ 

```

Npm 初始化项目

```bash
ws/wasm-client on  main [!?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 11.9s 
➜ npm init wasm-app www
🦀 Rust + 🕸 Wasm = ❤

ws/wasm-client on  main [!?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 3.5s 
➜ 
```



[构建项目](https://rustwasm.wasmdev.cn/docs/book/game-of-life/hello-world.html#构建项目)

```bash
wasm-pack build
```

Cargo.toml

```toml
[workspace]
members = ["webservice", "webapp", "wasm-client"]
```

wasm-client/Cargo.toml

```toml
[package]
name = "wasm-client"
version = "0.1.0"
authors = ["QiaoPengjun5162 <qiaopengjun0@gmail.com>"]
edition = "2018"

[lib]
crate-type = ["cdylib", "rlib"]

[features]
default = ["console_error_panic_hook"]

[dependencies]
chrono = { version = "0.4.26", features = ["serde"] }
serde = { version = "1.0.163", features = ["derive"] }
serde_derive = "1.0.163"
serde_json = "1.0.96"
js-sys = "0.3.63"
wasm-bindgen = { version = "0.2.86", features = ["serde-serialize"] }
wasm-bindgen-futures = "0.4.36"
web-sys = { version = "0.3.63", features = [
    "Headers",
    "Request",
    "RequestInit",
    "RequestMode",
    "Response",
    "Window",
    "Document",
    "Element",
    "HtmlElement",
    "Node",
    "console",
] }

# The `console_error_panic_hook` crate provides better debugging of panics by
# logging them with `console.error`. This is great for development, but requires
# all the `std::fmt` and `std::panicking` infrastructure, so isn't great for
# code size when deploying.
console_error_panic_hook = { version = "0.1.6", optional = true }

# `wee_alloc` is a tiny allocator for wasm that is only ~1K in code size
# compared to the default allocator's ~10K. It is slower than the default
# allocator, however.
wee_alloc = { version = "0.4.5", optional = true }

[dev-dependencies]
wasm-bindgen-test = "0.3.13"

[profile.release]
# Tell `rustc` to optimize for small code size.
opt-level = "s"

```

注意：并不是所有的 Rust crate 都能在wasm中使用

https://getbootstrap.com/docs/5.2/getting-started/introduction/#cdn-links

项目目录

```bash
ws on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ tree -a -I "target|.git|node_modules|.bin"
.
├── .env
├── .gitignore
├── .vscode
│   └── settings.json
├── Cargo.lock
├── Cargo.toml
├── README.md
├── wasm-client
│   ├── .appveyor.yml
│   ├── .gitignore
│   ├── .travis.yml
│   ├── Cargo.toml
│   ├── LICENSE_APACHE
│   ├── LICENSE_MIT
│   ├── README.md
│   ├── pkg
│   │   ├── .gitignore
│   │   ├── README.md
│   │   ├── package.json
│   │   ├── wasm_client.d.ts
│   │   ├── wasm_client.js
│   │   ├── wasm_client_bg.js
│   │   ├── wasm_client_bg.wasm
│   │   └── wasm_client_bg.wasm.d.ts
│   ├── src
│   │   ├── errors.rs
│   │   ├── lib.rs
│   │   ├── models
│   │   │   ├── course.rs
│   │   │   └── mod.rs
│   │   └── utils.rs
│   ├── tests
│   │   └── web.rs
│   └── www
│       ├── .gitignore
│       ├── .travis.yml
│       ├── LICENSE-APACHE
│       ├── LICENSE-MIT
│       ├── README.md
│       ├── bootstrap.js
│       ├── index.html
│       ├── index.js
│       ├── package-lock.json
│       ├── package.json
│       └── webpack.config.js
├── webapp
│   ├── .env
│   ├── Cargo.toml
│   ├── src
│   │   ├── bin
│   │   │   └── svr.rs
│   │   ├── errors.rs
│   │   ├── handlers.rs
│   │   ├── mod.rs
│   │   ├── models.rs
│   │   └── routers.rs
│   └── static
│       ├── css
│       │   └── register.css
│       ├── register.html
│       └── teachers.html
└── webservice
    ├── Cargo.toml
    └── src
        ├── bin
        │   └── teacher-service.rs
        ├── dbaccess
        │   ├── course.rs
        │   ├── mod.rs
        │   └── teacher.rs
        ├── errors.rs
        ├── handlers
        │   ├── course.rs
        │   ├── general.rs
        │   ├── mod.rs
        │   └── teacher.rs
        ├── main.rs
        ├── models
        │   ├── course.rs
        │   ├── mod.rs
        │   └── teacher.rs
        ├── routers.rs
        └── state.rs

19 directories, 65 files

ws on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ 
```

wasm-client/www/package.json

```json
{
  "name": "create-wasm-app",
  "version": "0.1.0",
  "description": "create an app to consume rust-generated wasm packages",
  "main": "index.js",
  "bin": {
    "create-wasm-app": ".bin/create-wasm-app.js"
  },
  "scripts": {
    "build": "webpack --config webpack.config.js",
    "start": "webpack-dev-server"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/rustwasm/create-wasm-app.git"
  },
  "keywords": [
    "webassembly",
    "wasm",
    "rust",
    "webpack"
  ],
  "author": "Ashley Williams <ashley666ashley@gmail.com>",
  "license": "(MIT OR Apache-2.0)",
  "bugs": {
    "url": "https://github.com/rustwasm/create-wasm-app/issues"
  },
  "homepage": "https://github.com/rustwasm/create-wasm-app#readme",
  "dependencies": {
    "wasm-client": "file:../pkg"
  },
  "devDependencies": {
    "hello-wasm-pack": "^0.1.0",
    "webpack": "^4.29.3",
    "webpack-cli": "^3.1.0",
    "webpack-dev-server": "^3.1.5",
    "copy-webpack-plugin": "^5.0.0"
  }
}

```

wasm-client/src/errors.rs

```rust
use serde::Serialize;

#[derive(Debug, Serialize)]
pub enum MyError {
    SomeError(String),
}

impl From<String> for MyError {
    fn from(s: String) -> Self {
        MyError::SomeError(s)
    }
}

impl From<wasm_bindgen::JsValue> for MyError {
    fn from(js_value: wasm_bindgen::JsValue) -> Self {
        MyError::SomeError(js_value.as_string().unwrap())
    }
}

```

wasm-client/src/models/mod.rs

```rust
pub mod course;
```

wasm-client/src/models/course.rs

```rust
use super::super::errors::MyError;
use chrono::NaiveDateTime;
use serde::{Deserialize, Serialize};
use wasm_bindgen::JsCast;
use wasm_bindgen_futures::JsFuture;
use web_sys::{Request, RequestInit, RequestMode, Response};

#[derive(Debug, Deserialize, Serialize)]
pub struct Course {
    pub teacher_id: i32,
    pub id: i32,
    pub name: String,
    pub time: NaiveDateTime,

    pub description: Option<String>,
    pub format: Option<String>,
    pub structure: Option<String>,
    pub duration: Option<String>,
    pub price: Option<i32>,
    pub language: Option<String>,
    pub level: Option<String>,
}

pub async fn get_courses_by_teacher(teacher_id: i32) -> Result<Vec<Course>, MyError> {
    // 访问webservice 读取课程
    let mut opts = RequestInit::new();
    opts.method("GET");
    opts.mode(RequestMode::Cors); // 跨域

    let url = format!("http://localhost:3000/courses/{}", teacher_id);

    let request = Request::new_with_str_and_init(&url, &opts)?;
    request.headers().set("Accept", "application/json")?;

    let window = web_sys::window().ok_or("no window exists".to_string())?;
    let resp_value = JsFuture::from(window.fetch_with_request(&request)).await?;

    assert!(resp_value.is_instance_of::<Response>());

    let resp: Response = resp_value.dyn_into().unwrap();
    let json = JsFuture::from(resp.json()?).await?;

    // let courses: Vec<Course> = json.into_serde().unwrap();
    let courses: Vec<Course> = serde_wasm_bindgen::from_value(json).unwrap();

    Ok(courses)
}

pub async fn delete_course(teacher_id: i32, course_id: i32) -> () {
    let mut opts = RequestInit::new();
    opts.method("DELETE");
    opts.mode(RequestMode::Cors);

    let url = format!("http://localhost:3000/courses/{}/{}", teacher_id, course_id);

    let request = Request::new_with_str_and_init(&url, &opts).unwrap();
    request.headers().set("Accept", "application/json").unwrap();

    let window = web_sys::window().unwrap();
    let resp_value = JsFuture::from(window.fetch_with_request(&request))
        .await
        .unwrap();

    assert!(resp_value.is_instance_of::<Response>());

    let resp: Response = resp_value.dyn_into().unwrap();
    let json = JsFuture::from(resp.json().unwrap()).await.unwrap();

    // let _course: Course = json.into_serde().unwrap();
    let _courses: Course = serde_wasm_bindgen::from_value(json).unwrap();
}

use js_sys::Promise;
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub async fn add_course(name: String, description: String) -> Result<Promise, JsValue> {
    let mut opts = RequestInit::new();
    opts.method("POST");
    opts.mode(RequestMode::Cors);
    let str_json = format!(
        r#"
        {{
            "teacher_id": 1,
            "name": "{}",
            "description": "{}"
        }}
        "#,
        name, description
    );
    opts.body(Some(&JsValue::from_str(str_json.as_str())));
    let url = "http://localhost:3000/courses/";

    let request = Request::new_with_str_and_init(&url, &opts)?;
    request.headers().set("Content-Type", "application/json")?;
    request.headers().set("Accept", "application/json")?;

    let window = web_sys::window().ok_or("no window exists".to_string())?;
    let resp_value = JsFuture::from(window.fetch_with_request(&request))
        .await
        .unwrap();

    assert!(resp_value.is_instance_of::<Response>());

    let resp: Response = resp_value.dyn_into().unwrap();
    Ok(resp.json()?)
}

```

wasm-client/src/lib.rs

```rust
mod utils;

use wasm_bindgen::prelude::*;

// When the `wee_alloc` feature is enabled, use `wee_alloc` as the global
// allocator.
#[cfg(feature = "wee_alloc")]
#[global_allocator]
static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;

#[wasm_bindgen]
extern "C" {
    fn alert(s: &str);
    fn confirm(s: &str) -> bool;

    #[wasm_bindgen(js_namespace = console)]
    fn log(s: &str);
}

#[wasm_bindgen]
pub fn greet(_name: &str) {
    alert("Hello, wasm-client!");
}

pub mod errors;
pub mod models;

use models::course::{delete_course, get_courses_by_teacher, Course};
use wasm_bindgen::JsCast;
use wasm_bindgen_futures::*;
use web_sys::HtmlButtonElement;

#[wasm_bindgen(start)]
pub async fn main() -> Result<(), JsValue> {
    let window = web_sys::window().expect("no global window exists");
    let document = window.document().expect("no global document exists");

    let left_tbody = document
        .get_element_by_id("left-tbody")
        .expect("left div not exists");

    let courses: Vec<Course> = get_courses_by_teacher(1).await.unwrap();
    for c in courses.iter() {
        let tr = document.create_element("tr")?;
        tr.set_attribute("id", format!("tr-{}", c.id).as_str())?;
        let td = document.create_element("td")?;
        td.set_text_content(Some(format!("{}", c.id).as_str()));
        tr.append_child(&td)?;

        let td = document.create_element("td")?;
        td.set_text_content(Some(c.name.as_str()));
        tr.append_child(&td)?;

        let td = document.create_element("td")?;
        td.set_text_content(Some(c.time.format("%Y-%m-%d").to_string().as_str()));
        tr.append_child(&td)?;

        let td = document.create_element("td")?;
        if let Some(desc) = c.description.clone() {
            td.set_text_content(Some(desc.as_str()));
        }
        tr.append_child(&td)?;

        let td = document.create_element("td")?;
        // let btn = document.create_element("button")?;
        let btn: HtmlButtonElement = document
            .create_element("button")
            .unwrap()
            .dyn_into::<HtmlButtonElement>()
            .unwrap();

        let cid = c.id;
        let click_closure = Closure::wrap(Box::new(move |_event: web_sys::MouseEvent| {
            let r = confirm(format!("确认删除 ID 为 {} 的课程？", cid).as_str());
            match r {
                true => {
                    spawn_local(delete_course(1, cid)); // delete_course 异步函数 spawn_local 把 future 放在当前线程
                    alert("删除成功！");

                    web_sys::window().unwrap().location().reload().unwrap();
                }
                _ => {}
            }
        }) as Box<dyn Fn(_)>);

        btn.add_event_listener_with_callback("click", click_closure.as_ref().unchecked_ref())?; // 要把闭包转化为 function 的引用
        click_closure.forget(); // 走出作用域后函数依然有效 但会造成内存泄漏

        btn.set_attribute("class", "btn btn-danger btn-sm")?;
        btn.set_text_content(Some("Delete"));
        td.append_child(&btn)?;
        tr.append_child(&td)?;

        left_tbody.append_child(&tr)?;
    }

    Ok(())
}

```



wasm-client/www/index.js

```js
import * as wasm from "wasm-client";

const myForm = document.getElementById('form');
myForm.addEventListener('submit', (e) => {
  e.preventDefault();
  const name = document.getElementById('name').value;
  const desc = document.querySelector('#description').value;

  wasm.add_course(name, desc).then((json) => {
    alert("添加成功！");
    window.location.reload();
  });
});

```

wasm-client/www/index.html

```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <title>Hello wasm-pack!</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/css/bootstrap.min.css" rel="stylesheet"
    integrity="sha384-rbsA2VBKQhggwzxH7pPCaAqO46MgnOM80zW1RWuH61DGLwZJEdK2Kadq2F9CUG65" crossorigin="anonymous" />
</head>

<body>
  <noscript>This page contains webassembly and javascript content, please enable javascript in your browser.</noscript>
  <nav class="navbar navbar-dark bg-primary">
    <div class="container-fluid">
      <a class="navbar-brand" href="#">Wasm-client</a>
    </div>
  </nav>
  <div class="m-3" style="height: 600px">
    <div class="row">
      <div class="col">
        <div class="card border-info mb-3">
          <div class="card-header">课程列表</div>
          <!-- <div class="card-body">
            <button type="button" class="btn btn-primary">Add</button>
          </div> -->
          <table class="table talbe-hover table-bordered table-sm">
            <thead>
              <tr>
                <th scope="col">ID</th>
                <th scope="col">名称</th>
                <th scope="col">时间</th>
                <th scope="col">简介</th>
                <th scope="col">操作</th>
              </tr>
            </thead>
            <tbody id="left-tbody"></tbody>
          </table>
          <div id="left"></div>
        </div>
      </div>
      <div class="col">
        <div class="card border-info mb-3">
          <div class="card-header">添加课程</div>
          <div class="card-body">
            <form class="row g-3 needs-validation" id="form">
              <div class="mb-3">
                <label for="name" class="form-label">课程名称</label>
                <input type="name" class="form-control" id="name" required placeholder="课程名称" />
                <div class="invalid-feedback">
                  请填写课程名！
                </div>
              </div>
              <div class="mb-3">
                <label for="description" class="form-label">课程简介</label>
                <textarea class="form-control" id="description" rows="3"></textarea>
              </div>
              <div class="col-12">
                <button type="submit" class="btn btn-primary">提交</button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
  <script src="./bootstrap.js"></script>
</body>

</html>

```

运行

```bash
ws/wasm-client on  main [!?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base 
➜ wasm-pack build           

ws/webservice on  main [!?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base 
➜ cargo run    

www on  master [!] is 📦 0.1.0 via ⬢ v19.7.0 via 🅒 base 
➜ npm install && npm run start
```

访问：http://localhost:8080/

问题：报错  Uncaught (in promise) DOMException: Failed to execute 'appendChild' on 'Node': The new child element contains the parent. 前端获取不到课程信息

解决：

这个错误提示说明在尝试将一个元素添加为其父元素的子元素时出现了问题，导致了循环引用。具体来说，错误发生在这一行代码中：

```
td.append_child(&td)?;
```

在这行代码中，你试图将创建的`td`元素添加为其自身的子元素。这是不允许的，因为一个元素不能成为自己的子元素。

正确的代码是：` tr.append_child(&td)?;`

修改之后的代码：

```rust
#[wasm_bindgen(start)]
pub async fn main() -> Result<(), JsValue> {
    let window = web_sys::window().expect("no global window exists");
    let document = window.document().expect("no global document exists");

    let left_tbody = document
        .get_element_by_id("left-tbody")
        .expect("left div not exists");

    let courses: Vec<Course> = get_courses_by_teacher(1).await.unwrap();
    for c in courses.iter() {
        let tr = document.create_element("tr")?;
        tr.set_attribute("id", format!("tr-{}", c.id).as_str())?;
        let td = document.create_element("td")?;
        td.set_text_content(Some(format!("{}", c.id).as_str()));
        tr.append_child(&td)?;

        let td = document.create_element("td")?;
        td.set_text_content(Some(c.name.as_str()));
        tr.append_child(&td)?;

        let td = document.create_element("td")?;
        td.set_text_content(Some(c.time.format("%Y-%m-%d").to_string().as_str()));
        tr.append_child(&td)?;

        let td = document.create_element("td")?;
        if let Some(desc) = c.description.clone() {
            td.set_text_content(Some(desc.as_str()));
        }
        tr.append_child(&td)?;

        let td = document.create_element("td")?;
        // let btn = document.create_element("button")?;
        let btn: HtmlButtonElement = document
            .create_element("button")
            .unwrap()
            .dyn_into::<HtmlButtonElement>()
            .unwrap();

        let cid = c.id;
        let click_closure = Closure::wrap(Box::new(move |_event: web_sys::MouseEvent| {
            let r = confirm(format!("确认删除 ID 为 {} 的课程？", cid).as_str());
            match r {
                true => {
                    spawn_local(delete_course(1, cid)); // delete_course 异步函数 spawn_local 把 future 放在当前线程
                    alert("删除成功！");

                    web_sys::window().unwrap().location().reload().unwrap();
                }
                _ => {}
            }
        }) as Box<dyn Fn(_)>);

        btn.add_event_listener_with_callback("click", click_closure.as_ref().unchecked_ref())?; // 要把闭包转化为 function 的引用
        click_closure.forget(); // 走出作用域后函数依然有效 但会造成内存泄漏

        btn.set_attribute("class", "btn btn-danger btn-sm")?;
        btn.set_text_content(Some("Delete"));
        td.append_child(&btn)?;
        tr.append_child(&td)?;

        left_tbody.append_child(&tr)?;
    }

    Ok(())
}

```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306041524535.png)
