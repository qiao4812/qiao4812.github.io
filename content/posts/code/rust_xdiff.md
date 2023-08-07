---
title: "Rust 开发实操 - xdiff"
date: 2023-08-06T21:57:30+08:00
lastmod: 2023-08-06T21:57:30+08:00 
draft: false
categories:
- Rust
tags:
- Rust
---

# Rust 开发实操 - xdiff

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

创建lib.rs  

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

