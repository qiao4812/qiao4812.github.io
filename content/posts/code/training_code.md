---
title: "用文件持久化数据结构"
date: 2023-07-30T23:58:15+08:00
lastmod: 2023-07-30T23:58:15+08:00 #更新时间
draft: true
categories:
- Rust
tags:
- Rust
---

# 用文件持久化数据结构

### 思考：如何在内存和IO设备间交换数据？

### 创建项目

```bash
~/Code/rust via 🅒 base
➜ cargo new training_code --lib
     Created library `training_code` package

~/Code/rust via 🅒 base
➜ cd training_code

training_code on  master [?] via 🦀 1.71.0 via 🅒 base
➜ c

```

### 项目目录

```bash
training_code on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ tree -a -I "target|.git"
.
├── .gitignore
├── Cargo.lock
├── Cargo.toml
└── src
    ├── lib.rs
    └── user.rs

2 directories, 5 files

training_code on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ 
```

### Cargo.toml

```toml
[package]
name = "training_code"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
serde = { version = "1.0.178", features = ["derive"] }
serde_json = "1.0.104"

```



### lib.rs

```rust
pub mod user;

```



### user.rs

```rust
// 用文件持久化数据结构
// 思考：如何在内存和IO设备间交换数据？

use std::{
    fs::File,
    io::{Read, Write},
};

use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug, PartialEq)]
pub enum Gender {
    Unspecified = 0,
    Male = 1,
    Female = 2,
}

#[derive(Serialize, Deserialize, Debug, PartialEq)]
pub struct User {
    pub name: String,
    age: u8,
    pub(crate) gender: Gender,
}

impl User {
    pub fn new(name: String, age: u8, gender: Gender) -> Self {
        Self { name, age, gender }
    }

    pub fn load(filename: &str) -> Result<Self, std::io::Error> {
        let mut file = File::open(filename)?;
        let mut data = String::new();
        file.read_to_string(&mut data)?;
        let user = serde_json::from_str(&data)?;
        Ok(user)
    }

    pub fn persist(&self, filename: &str) -> Result<usize, std::io::Error> {
        let mut file = File::create(filename)?;

        // // ? eq
        // match File::create(filename) {
        //     Ok(f) => {
        //         todo!()
        //     }
        //     Err(e) => return Err(e),
        // }

        let data = serde_json::to_string(self)?;
        file.write_all(data.as_bytes())?;

        Ok(data.len())
    }
}

impl Default for User {
    fn default() -> Self {
        User::new("Unknown User".into(), 0, Gender::Unspecified)
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let user = User::default();
        let path = "/tmp/user1";
        user.persist(path).unwrap();
        let user1 = User::load(path).unwrap();
        assert_eq!(user, user1);
    }
}

```

### 测试

```bash
training_code on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ cargo test                          
   Compiling training_code v0.1.0 (/Users/qiaopengjun/Code/rust/training_code)
    Finished test [unoptimized + debuginfo] target(s) in 0.70s
     Running unittests src/lib.rs (target/debug/deps/training_code-c41752abc7a3994f)

running 1 test
test user::tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests training_code

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


training_code on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ 
```

