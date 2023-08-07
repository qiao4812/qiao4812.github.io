---
title: "Rust_kv"
date: 2023-08-07T23:08:49+08:00
lastmod: 2023-08-07T23:08:49+08:00 
draft: false
categories:
- 分类1
- 分类2
tags:
- 标签1
- 标签2
---





```shell
 cd Code/rust
 
 cargo new kv
 
 cd kv
 
 c
 
 touch build.rs
 
 cargo build
 
 cargo run
```



```bash
kv on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ tree -a -I "target|.git"
.
├── .gitignore
├── .vscode
│   └── settings.json
├── Cargo.lock
├── Cargo.toml
├── abi.proto
├── build.rs
└── src
    ├── main.rs
    └── pb
        ├── abi.rs
        └── mod.rs

4 directories, 9 files

kv on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ 
```

Cargo.toml

```toml
[package]
name = "kv"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
tokio = "1.29.1"
prost = "0.11.9"
# Only necessary if using Protobuf well-known types:
prost-types = "0.11.9"

[build-dependencies]
prost-build = "0.11.9"

```

abi.proto

```protobuf
syntax = "proto3";

package abi;

message Request {
    oneof message {
        RequestGet get = 1;
        RequestPut put = 2;
    }
}

message Response {
    uint32 code = 1;
    string Key = 2;
    string value = 3;
}

message RequestGet {
    string Key = 1;
}

message RequestPut {
    string Key = 1;
    bytes value = 2;
}

```

build.rs

```rust
fn main() {
    prost_build::Config::new()
        .out_dir("src/pb")
        .compile_protos(&["abi.proto"], &["."])
        .unwrap();
}

```

src/pb/mod.rs

```rust
mod abi;
pub use abi::*;

```

src/main.rs

```rust
mod pb;

use prost::Message;

use crate::pb::RequestGet;

fn main() {
    let request = RequestGet {
        key: "hello".to_string(),
    };
    let mut buf = Vec::new();
    request.encode(&mut buf).unwrap();
    println!("encoded: {:?}", buf);
}

```



```shell
kv on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base took 3.6s 
➜ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.03s
     Running `target/debug/kv`
encoded: [10, 5, 104, 101, 108, 108, 111]

kv on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ 
```

