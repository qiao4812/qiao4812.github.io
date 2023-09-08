---
title: "Rust crates 学习之 prost 实操"
date: 2023-08-31T23:41:26+08:00
lastmod: 2023-08-31T23:41:26+08:00 
draft: false
categories:
- Rust
tags:
- Rust
---

# Rust crates 学习之 prost  实操

### 相关链接

https://github.com/tokio-rs/prost

https://www.bilibili.com/video/BV1FL4y1x7MU/?spm_id_from=333.999.list.card_archive.click&vd_source=bba3c74b0f6a3741d178163e8828d21b

https://docs.rs/prost-build/latest/prost_build/

### 创建项目并用vscode 打开

```shell
~ via 🅒 base
➜ cd Code/rust

~/Code/rust via 🅒 base
➜ cargo new prost-live --lib
     Created library `prost-live` package

~/Code/rust via 🅒 base
➜ cd prost-live

prost-live on  master [?] via 🦀 1.72.0 via 🅒 base
➜ c

prost-live on  master [?] via 🦀 1.72.0 via 🅒 base
➜

```

### 添加相关依赖，并创建文件

```shell
prost-live on  master [?] via 🦀 1.72.0 via 🅒 base 
➜ cargo add prost                 
   

prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ cargo add prost-types       
   

prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ touch person.proto    

prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ cargo add prost-build --build
   

prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base took 3.3s 
➜ touch build.rs    

prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ cargo add anyhow             

```

### 项目目录

```bash
prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ tree -L 2
.
├── Cargo.lock
├── Cargo.toml
├── build.rs
├── examples
│   └── person.rs
├── person.proto
├── src
│   └── lib.rs
└── target
    ├── CACHEDIR.TAG
    └── debug

5 directories, 7 files
```

### person.proto 文件

```protobuf
syntax = "proto3";
package person;

// define a person
message Person {
  string name = 1;
  int32 id = 2;  // Unique ID number for this person.
  string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    string number = 1;
    PhoneType type = 2;
  }

  repeated PhoneNumber phones = 4;
}

// Our address book file is just one of these.
message AddressBook {
  repeated Person people = 1;
}

```

### build.rs 文件

```rust
use prost_build::compile_protos;

// https://docs.rs/prost-build/latest/prost_build/
fn main() {
    // OUT_DIR
    compile_protos(&["person.proto"], &["."]).unwrap();
}

```

### src/lib.rs 文件

```rust
// https://github.com/tokio-rs/prost
pub mod pb {
    include!(concat!(env!("OUT_DIR"), "/person.rs"));
}

```

### examples/person.rs 文件

```rust
use prost_live::pb::*;

fn main() {
    let person = Person::default();
    println!("{person:?}");
}

```

### Cargo.toml 文件

```toml
[package]
name = "prost-live"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
anyhow = "1.0.75"
prost = "0.12.0"
prost-types = "0.12.0"

[build-dependencies] # https://doc.rust-lang.org/cargo/commands/cargo-add.html
prost-build = "0.12.0"

```

### 创建例子文件夹并创建person文件运行

```shell
prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ mkdir examples

prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ touch examples/person.rs 

prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ cargo run --example person       
   Compiling bytes v1.4.0
   Compiling anyhow v1.0.75
   Compiling prost-derive v0.11.9
   Compiling prost v0.11.9
   Compiling prost-types v0.11.9
   Compiling prost-build v0.11.9
   Compiling prost-live v0.1.0 (/Users/qiaopengjun/Code/rust/prost-live)
    Finished dev [unoptimized + debuginfo] target(s) in 2.04s
     Running `target/debug/examples/person`
Person { name: "", id: 0, email: "", phones: [] }

```

## 使用protobuf encode

### 项目目录

```shell
prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ tree -a -I "target|.git" 
.
├── .gitignore
├── Cargo.lock
├── Cargo.toml
├── build.rs
├── examples
│   └── person.rs
├── person.proto
└── src
    ├── lib.rs
    └── pb
        ├── mod.rs
        └── person.rs

4 directories, 9 files

```



### build.rs 文件

```rust
use prost_build::Config;

// https://docs.rs/prost-build/latest/prost_build/
fn main() {
    // https://doc.rust-lang.org/cargo/reference/build-scripts.html
    println!("cargo:rerun-if-changed=person.proto");
    println!("cargo:rerun-if-changed=build.rs");
    Config::new()
        .out_dir("./src/pb")
        .compile_protos(&["person.proto"], &["."])
        .unwrap();
}

```

### person.proto 文件

```protobuf
syntax = "proto3";
package person;

// define a person
message Person {
  string name = 1;
  int32 id = 2;  // Unique ID number for this person.
  string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    string number = 1;
    PhoneType phone_type = 2;
  }

  repeated PhoneNumber phones = 4;
}

// Our address book file is just one of these.
message AddressBook {
  repeated Person people = 1;
}

```

### ./src/pb/mod.rs

```rust
mod person;

pub use person::Person;

pub use self::person::person::{PhoneNumber, PhoneType};

impl Person {
    pub fn new(
        name: impl Into<String>,
        id: i32,
        email: impl Into<String>,
        phones: impl Into<Vec<PhoneNumber>>,
    ) -> Self {
        Self {
            name: name.into(),
            id,
            email: email.into(),
            phones: phones.into(),
        }
    }
}

impl PhoneNumber {
    pub fn new(number: impl Into<String>, phone_type: PhoneType) -> Self {
        Self {
            number: number.into(),
            phone_type: phone_type.into(),
        }
    }
}

```

### examples/person.rs 文件

```rust
use prost::Message;
use prost_live::pb::*;

fn main() {
    let phones = vec![PhoneNumber::new("123-111-123", PhoneType::Mobile)];
    let person = Person::new("xiao qiao", 1, "qiao4812@gmail.com", phones);
    let v1 = person.encode_to_vec();
    let v2 = person.encode_length_delimited_to_vec();
    println!("{person:?}, {v1:?}(len: {}), {v2:?}", v1.len());
}

```

### 运行

```shell
prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ cargo run --example person
   Compiling prost-live v0.1.0 (/Users/qiaopengjun/Code/rust/prost-live)
    Finished dev [unoptimized + debuginfo] target(s) in 0.12s
     Running `target/debug/examples/person`
Person { name: "xiao qiao", id: 1, email: "qiao4812@gmail.com", phones: [PhoneNumber { number: "123-111-123", phone_type: Mobile }] }, [10, 9, 120, 105, 97, 111, 32, 113, 105, 97, 111, 16, 1, 26, 18, 113, 105, 97, 111, 52, 56, 49, 50, 64, 103, 109, 97, 105, 108, 46, 99, 111, 109, 34, 13, 10, 11, 49, 50, 51, 45, 49, 49, 49, 45, 49, 50, 51](len: 48), [48, 10, 9, 120, 105, 97, 111, 32, 113, 105, 97, 111, 16, 1, 26, 18, 113, 105, 97, 111, 52, 56, 49, 50, 64, 103, 109, 97, 105, 108, 46, 99, 111, 109, 34, 13, 10, 11, 49, 50, 51, 45, 49, 49, 49, 45, 49, 50, 51]

```

examples/person.rs 文件

```rust
use prost::Message;
use prost_live::pb::*;

fn main() {
    let phones = vec![PhoneNumber::new("123-111-123", PhoneType::Mobile)];
    let person = Person::new("xiao qiao", 1, "qiao4812@gmail.com", phones);
    let v1 = person.encode_to_vec();
    let person1 = Person::decode(v1.as_slice()).unwrap();
    let person11 = Person::decode(v1.as_ref()).unwrap();
    assert_eq!(person, person1);
    assert_eq!(person, person11);
    println!("person1: {person1:?}");
    let v2 = person.encode_length_delimited_to_vec();
    println!("{person:?}, {v1:?}(len: {}), {v2:?}", v1.len());
}

```

运行

```shell
prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ cargo run --example person
   Compiling prost-live v0.1.0 (/Users/qiaopengjun/Code/rust/prost-live)
    Finished dev [unoptimized + debuginfo] target(s) in 0.17s
     Running `target/debug/examples/person`
person1: Person { name: "xiao qiao", id: 1, email: "qiao4812@gmail.com", phones: [PhoneNumber { number: "123-111-123", phone_type: Mobile }] }
Person { name: "xiao qiao", id: 1, email: "qiao4812@gmail.com", phones: [PhoneNumber { number: "123-111-123", phone_type: Mobile }] }, [10, 9, 120, 105, 97, 111, 32, 113, 105, 97, 111, 16, 1, 26, 18, 113, 105, 97, 111, 52, 56, 49, 50, 64, 103, 109, 97, 105, 108, 46, 99, 111, 109, 34, 13, 10, 11, 49, 50, 51, 45, 49, 49, 49, 45, 49, 50, 51](len: 48), [48, 10, 9, 120, 105, 97, 111, 32, 113, 105, 97, 111, 16, 1, 26, 18, 113, 105, 97, 111, 52, 56, 49, 50, 64, 103, 109, 97, 105, 108, 46, 99, 111, 109, 34, 13, 10, 11, 49, 50, 51, 45, 49, 49, 49, 45, 49, 50, 51]

prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ 
```

## 使用protobuf decode

### 安装相关依赖

```bash
prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ cargo add bytes 

prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ cargo add serde --features derive    

prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ cargo add serde_json             
```

### person.proto 文件

```protobuf
syntax = "proto3";
package person;

// define a person
message Person {
  string name = 1;
  int32 id = 2;  // Unique ID number for this person.
  string email = 3;
  bytes data = 4;
  map<string, int32> scopes = 5;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    string number = 1;
    PhoneType phone_type = 2;
  }

  repeated PhoneNumber phones = 6;
}

// Our address book file is just one of these.
message AddressBook {
  repeated Person people = 1;
}

```

### src/pb/mod.rs

```rust
mod person;

pub use person::Person;

pub use self::person::person::{PhoneNumber, PhoneType};

impl Person {
    pub fn new(
        name: impl Into<String>,
        id: i32,
        email: impl Into<String>,
        phones: impl Into<Vec<PhoneNumber>>,
    ) -> Self {
        Self {
            name: name.into(),
            id,
            email: email.into(),
            phones: phones.into(),
            ..Default::default()
        }
    }
}

impl PhoneNumber {
    pub fn new(number: impl Into<String>, phone_type: PhoneType) -> Self {
        Self {
            number: number.into(),
            phone_type: phone_type.into(),
        }
    }
}

```



### build.rs 文件

```rust
use prost_build::Config;

// https://docs.rs/prost-build/latest/prost_build/
fn main() {
    // https://doc.rust-lang.org/cargo/reference/build-scripts.html
    println!("cargo:rerun-if-changed=person.proto");
    println!("cargo:rerun-if-changed=build.rs");
    Config::new()
        .out_dir("./src/pb")
        // .bytes(&["."])
        .btree_map(&["scopes"])
        .type_attribute(".", "#[derive(serde::Serialize, serde::Deserialize)]")
        .compile_protos(&["person.proto"], &["."])
        .unwrap();
}

```

### examples/person.rs 文件

```rust
use prost::Message;
use prost_live::pb::*;

fn main() {
    let phones = vec![PhoneNumber::new("123-111-123", PhoneType::Mobile)];
    let person = Person::new("xiao qiao", 1, "qiao4812@gmail.com", phones);
    let v1 = person.encode_to_vec();
    let person1 = Person::decode(v1.as_slice()).unwrap();
    let person11 = Person::decode(v1.as_ref()).unwrap();
    assert_eq!(person, person1);
    assert_eq!(person, person11);
    // println!("person1: {person1:?}");
    // let v2 = person.encode_length_delimited_to_vec();
    // println!("{person:?}, {v1:?}(len: {}), {v2:?}", v1.len());

    let json = serde_json::to_string_pretty(&person1).unwrap();
    println!("{json}");
}

```



### 运行

```shell
prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base took 2.4s 
➜ cargo run --example person
   Compiling ryu v1.0.15
   Compiling itoa v1.0.9
   Compiling serde_json v1.0.105
   Compiling prost-live v0.1.0 (/Users/qiaopengjun/Code/rust/prost-live)
    Finished dev [unoptimized + debuginfo] target(s) in 0.89s
     Running `target/debug/examples/person`
{
  "name": "xiao qiao",
  "id": 1,
  "email": "qiao4812@gmail.com",
  "data": [],
  "scopes": {},
  "phones": [
    {
      "number": "123-111-123",
      "phone_type": 0
    }
  ]
}

prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ 
```

## Data 字段的处理

[Serde](https://serde.rs/)：https://serde.rs/

https://docs.rs/prost-build-config/latest/prost_build_config/

build.rs 文件

```rust
use prost_build::Config;

// https://docs.rs/prost-build/latest/prost_build/
fn main() {
    // https://doc.rust-lang.org/cargo/reference/build-scripts.html
    println!("cargo:rerun-if-changed=person.proto");
    println!("cargo:rerun-if-changed=build.rs");
    Config::new()
        .out_dir("./src/pb")
        // .bytes(&["."])
        .btree_map(&["scopes"])
        .type_attribute(".", "#[derive(serde::Serialize, serde::Deserialize)]")
        .field_attribute("data", "#[serde(skip_serializing_if = \"Vec::is_empty\")]")
        .compile_protos(&["person.proto"], &["."])
        .unwrap();
}

```

### 运行

```shell
prost-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ cargo run --example person
   Compiling prost-live v0.1.0 (/Users/qiaopengjun/Code/rust/prost-live)
    Finished dev [unoptimized + debuginfo] target(s) in 0.22s
     Running `target/debug/examples/person`
{
  "name": "xiao qiao",
  "id": 1,
  "email": "qiao4812@gmail.com",
  "scopes": {},
  "phones": [
    {
      "number": "123-111-123",
      "phone_type": 0
    }
  ]
}

```

更多请参考：https://www.bilibili.com/video/BV1FL4y1x7MU/?spm_id_from=333.999.list.card_archive.click&vd_source=bba3c74b0f6a3741d178163e8828d21b



https://github.com/protocolbuffers/protobuf

https://protobuf.dev/
