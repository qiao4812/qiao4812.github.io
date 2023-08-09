---
title: "Rust 开发实操 - xdiff（1）基本思路和数据结构"
date: 2023-08-06T21:57:30+08:00
lastmod: 2023-08-06T21:57:30+08:00 
draft: false
categories:
- Rust
tags:
- Rust
---

# Rust 开发实操 - xdiff（1）基本思路和数据结构

## xdiff 项目介绍

HTTP request and diff tools

参考：<https://github.com/Tubitv/xdiff>

## xdiff 项目实操（1）基本思路和数据结构

### 创建项目并用 vscode 打开项目

```bash
~/Code/rust via 🅒 base
➜ cargo new rust_xdiff
     Created binary (application) `rust_xdiff` package

~/Code/rust via 🅒 base
➜ cd rust_xdiff

rust_xdiff on  master [?] via 🦀 1.71.0 via 🅒 base
➜ c

rust_xdiff on  master [?] via 🦀 1.71.0 via 🅒 base
➜
```



### 安装相关开发库

```shell
➜ cargo add clap --features derive        
➜ cargo add serde_yaml    
➜ cargo add tokio --features full    
➜ cargo add anyhow  
➜ cargo add reqwest  
➜ cargo add reqwest --features rustls --no-default-features
➜ cargo add similar  
```

### Similar 使用实操之验证功能

[Similar: A Diffing Library]()：<https://github.com/mitsuhiko/similar>

例子一

#### 创建文件

```shell
rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ mkdir -p examples      

rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ touch examples/similar.rs

rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ 
```

#### similar.rs 文件

```rust
use similar::{ChangeTag, TextDiff};

fn main() {
    let diff = TextDiff::from_lines(
        "Hello World\nThis is the second line.\nThis is the third.",
        "Hallo Welt\nThis is the second line.\nThis is life.\nMoar and more",
    );

    for change in diff.iter_all_changes() {
        let sign = match change.tag() {
            ChangeTag::Delete => "-",
            ChangeTag::Insert => "+",
            ChangeTag::Equal => " ",
        };
        print!("{}{}", sign, change);
    }
}

```



#### 运行

```shell
rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base took 8.7s 
➜ cargo run --example similar
    Finished dev [unoptimized + debuginfo] target(s) in 0.10s
     Running `target/debug/examples/similar`
-Hello World
+Hallo Welt
 This is the second line.
-This is the third.
+This is life.
+Moar and more

rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ 
```

例子二

```shell
rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ touch examples/terminal-inline.rs    
rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ cargo add console 
➜ cargo add similar --features bytes  
➜ cargo add similar --features unicode  
➜ cargo add similar --features inline   
   
rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ touch examples/test1.txt

rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ touch examples/test2.txt
```

examples/terminal-inline.rs    

```rust
use std::fmt;
use std::fs::read;
use std::process::exit;

use console::{style, Style};
use similar::{ChangeTag, TextDiff};

struct Line(Option<usize>);

impl fmt::Display for Line {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self.0 {
            None => write!(f, "    "),
            Some(idx) => write!(f, "{:<4}", idx + 1),
        }
    }
}

fn main() {
    let args: Vec<_> = std::env::args_os().collect();
    if args.len() != 3 {
        eprintln!("usage: terminal-inline [old] [new]");
        exit(1);
    }

    let old = read(&args[1]).unwrap();
    let new = read(&args[2]).unwrap();
    let diff = TextDiff::from_lines(&old, &new);

    for (idx, group) in diff.grouped_ops(3).iter().enumerate() {
        if idx > 0 {
            println!("{:-^1$}", "-", 80);
        }
        for op in group {
            for change in diff.iter_inline_changes(op) {
                let (sign, s) = match change.tag() {
                    ChangeTag::Delete => ("-", Style::new().red()),
                    ChangeTag::Insert => ("+", Style::new().green()),
                    ChangeTag::Equal => (" ", Style::new().dim()),
                };
                print!(
                    "{}{} |{}",
                    style(Line(change.old_index())).dim(),
                    style(Line(change.new_index())).dim(),
                    s.apply_to(sign).bold(),
                );
                for (emphasized, value) in change.iter_strings_lossy() {
                    if emphasized {
                        print!("{}", s.apply_to(value).underlined().on_black());
                    } else {
                        print!("{}", s.apply_to(value));
                    }
                }
                if change.missing_newline() {
                    println!();
                }
            }
        }
    }
}

```



examples/test1.txt

```
hello world!
goodbye

```

examples/test2.txt

```
hello qiao!
goodbye

```

运行

```bash
rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ cargo run --example terminal-inline examples/test1.txt examples/test2.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.09s
     Running `target/debug/examples/terminal-inline examples/test1.txt examples/test2.txt`
1        |-hello world!
    1    |+hello qiao!
2   2    | goodbye

rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
```

创建相关文件并添加依赖库

```bash
rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ touch src/lib.rs        

rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ touch src/config.rs 

rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ mkdir -p fixtures

rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ touch fixtures/test.yml

rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ 
rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ cargo add serde --features derive  

➜ cargo add http_serde  
➜ cargo add url --features serde
➜ cargo add serde_json  

rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ touch examples/config.rs
```

examples/config.rs

```rust
use anyhow::Result;
use rust_xdiff::DiffConfig;

fn main() -> Result<()> {
    let content = include_str!("../fixtures/test.yml");
    let config = DiffConfig::from_yaml(content)?;

    println!("config: {:#?}", config);
    Ok(())
}

```

运行

```bash
 cargo run --example config
```



### 创建相关文件并提交代码到GitHub

```bash
rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ touch .pre-commit-config.yaml 


rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ mkdir .github          


rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ mkdir .github/workflows          

rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ touch .github/workflows/build.yml

rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ touch deny.toml                  

rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ echo "# rust_xdiff" >> README.md

rust_xdiff on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ git branch -M main

rust_xdiff on  main [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ git add .         

rust_xdiff on  main [+] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ git commit -a                                                              
[main (root-commit) 650ab9b] xdiff 基本思路和数据结构
 16 files changed, 1842 insertions(+)
 create mode 100644 .github/workflows/build.yml
 create mode 100644 .gitignore
 create mode 100644 .pre-commit-config.yaml
 create mode 100644 Cargo.lock
 create mode 100644 Cargo.toml
 create mode 100644 README.md
 create mode 100644 deny.toml
 create mode 100644 examples/config.rs
 create mode 100644 examples/similar.rs
 create mode 100644 examples/terminal-inline.rs
 create mode 100644 examples/test1.txt
 create mode 100644 examples/test2.txt
 create mode 100644 fixtures/test.yml
 create mode 100644 src/config.rs
 create mode 100644 src/lib.rs
 create mode 100644 src/main.rs

rust_xdiff on  main is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base took 1m 39.1s 
➜ git remote add origin git@github.com:qiaopengjun5162/rust_xdiff.git

rust_xdiff on  main is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ git push -u origin main
Enumerating objects: 23, done.
Counting objects: 100% (23/23), done.
Delta compression using up to 12 threads
Compressing objects: 100% (15/15), done.
Writing objects: 100% (23/23), 16.31 KiB | 5.44 MiB/s, done.
Total 23 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:qiaopengjun5162/rust_xdiff.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.

rust_xdiff on  main is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base took 6.1s 
➜ 
```

### 项目目录

```bash
rust_xdiff on  main is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base took 6.1s 
➜ tree -a -I "target|.git"
.
├── .github
│   └── workflows
│       └── build.yml
├── .gitignore
├── .pre-commit-config.yaml
├── Cargo.lock
├── Cargo.toml
├── README.md
├── deny.toml
├── examples
│   ├── config.rs
│   ├── similar.rs
│   ├── terminal-inline.rs
│   ├── test1.txt
│   └── test2.txt
├── fixtures
│   └── test.yml
└── src
    ├── config.rs
    ├── lib.rs
    └── main.rs

6 directories, 16 files

rust_xdiff on  main is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
```

src/config.rs

```rust
use anyhow::Result;
use reqwest::{header::HeaderMap, Method};
use serde::{Deserialize, Serialize};
use std::collections::HashMap;
use tokio::fs;
use url::Url;

use crate::ExtraArgs;

#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct DiffConfig {
    #[serde(flatten)]
    pub profiles: HashMap<String, DiffProfile>,
}

#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct DiffProfile {
    pub req1: RequestProfile,
    pub req2: RequestProfile,
    pub res: ResponseProfile,
}

#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct RequestProfile {
    #[serde(with = "http_serde::method", default)]
    pub method: Method,
    pub url: Url,
    #[serde(skip_serializing_if = "Option::is_none", default)]
    pub params: Option<serde_json::Value>,
    #[serde(
        skip_serializing_if = "HeaderMap::is_empty",
        with = "http_serde::header_map",
        default
    )]
    pub headers: HeaderMap,
    #[serde(skip_serializing_if = "Option::is_none", default)]
    pub body: Option<serde_json::Value>,
}

#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct ResponseProfile {
    #[serde(skip_serializing_if = "Vec::is_empty", default)]
    pub skip_headers: Vec<String>,
    #[serde(skip_serializing_if = "Vec::is_empty", default)]
    pub skip_body: Vec<String>,
}

impl DiffConfig {
    pub async fn load_yaml(path: &str) -> Result<Self> {
        let content = fs::read_to_string(path).await?;
        Self::from_yaml(&content)
    }

    pub fn from_yaml(content: &str) -> Result<Self> {
        Ok(serde_yaml::from_str(content)?)
    }

    pub fn get_profile(&self, name: &str) -> Option<&DiffProfile> {
        self.profiles.get(name)
    }
}

impl DiffProfile {
    pub async fn diff(&self, _args: ExtraArgs) -> Result<String> {
        // let res1 = req1.send(&args).await?;
        // let res2 = req2.send(&args).await?;

        // let text1 = res1.filter_text(&self.res).await?;
        // let text2 = res2.filter_text(&self.res).await?;

        // text_diff(&text1, &text2)

        todo!()
    }
}

```

src/lib.rs

```rust
pub mod cli;
pub(crate) mod config;

pub use config::{DiffConfig, DiffProfile, RequestProfile, ResponseProfile};

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct ExtraArgs {
    pub headers: Vec<(String, String)>,
    pub query: Vec<(String, String)>,
    pub body: Vec<(String, String)>,
}

```

src/main.rs

```rust
use clap::Parser;
use rust_xdiff::cli::Args;

fn main() {
    let args = Args::parse();
    println!("{:?}", args);
}

```

src/cli.rs

```rust
use anyhow::{anyhow, Result};
use clap::{Parser, Subcommand};

use crate::ExtraArgs;

/// Diff two http requests and compare the difference of the responses
#[derive(Debug, Parser, Clone)]
#[clap(version, author, about, long_about = None)]
pub struct Args {
    #[clap(subcommand)]
    pub action: Action,
}

#[derive(Subcommand, Debug, Clone)]
pub enum Action {
    /// Diff two API responses based on given profile
    Run(RunArgs),
}

#[derive(Debug, Parser, Clone)]
pub struct RunArgs {
    /// Profile name
    #[clap(long, short, value_parser)]
    pub profile: String,

    /// Overrides args. Could be used to override the query, headers and body of the request.
    /// For query params, use `-e key=value`.
    /// For headers, use `-e %key=value`.
    /// For body, use `-e @key=value`.
    #[clap(long, short, value_parser = parse_key_val, number_of_values = 1)]
    extra_params: Vec<KeyVal>,

    /// Configuration to use.
    #[clap(short, long, value_parser)]
    pub config: Option<String>,
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub enum KeyValType {
    Query,
    Header,
    Body,
}

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct KeyVal {
    key_type: KeyValType,
    key: String,
    value: String,
}

fn parse_key_val(s: &str) -> Result<KeyVal> {
    let mut parts = s.splitn(2, '=');
    let key = parts
        .next()
        .ok_or_else(|| anyhow!("Invalid key value pair"))?
        .trim();
    let value = parts
        .next()
        .ok_or_else(|| anyhow!("Invalid key value pair"))?
        .trim();

    let (key_type, key) = match key.chars().next() {
        Some('%') => (KeyValType::Header, &key[1..]),
        Some('@') => (KeyValType::Body, &key[1..]),
        Some(v) if v.is_ascii_alphabetic() => (KeyValType::Query, key),
        _ => return Err(anyhow!("Invalid key value pair")),
    };

    Ok(KeyVal {
        key_type,
        key: key.to_string(),
        value: value.to_string(),
    })
}

impl From<Vec<KeyVal>> for ExtraArgs {
    fn from(args: Vec<KeyVal>) -> Self {
        let mut headers = vec![];
        let mut query = vec![];
        let mut body = vec![];

        for arg in args {
            match arg.key_type {
                KeyValType::Header => headers.push((arg.key, arg.value)),
                KeyValType::Query => query.push((arg.key, arg.value)),
                KeyValType::Body => body.push((arg.key, arg.value)),
            }
        }

        Self {
            headers,
            query,
            body,
        }
    }
}

```



运行

```bash
rust_xdiff on  main [!?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base took 2.5s 
➜ cargo run -- run -p rust -c fixtures/test.yml -e a=100
    Finished dev [unoptimized + debuginfo] target(s) in 0.14s
     Running `target/debug/rust_xdiff run -p rust -c fixtures/test.yml -e a=100`
Args { action: Run(RunArgs { profile: "rust", extra_params: [KeyVal { key_type: Query, key: "a", value: "100" }], config: Some("fixtures/test.yml") }) }

rust_xdiff on  main [!?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base took 3.3s 
➜ 
```

