---
title: "《Rust编程之道》学习笔记二"
date: 2023-08-14T23:23:00+08:00
lastmod: 2023-08-14T23:23:00+08:00 
draft: false
categories:
- Rust
tags:
- Rust
---

# 《Rust编程之道》学习笔记二

## 第 2 章 语言精要

好读书，不求甚解；每有会意，便欣然忘食。

### Rust 语言主要由以下几个核心部件组成：

- 语言规范
- 编译器
- 核心库
- 标准库
- 包管理器

### 语言规范

Rust 语言规范主要由 Rust 语言参考（The Rust Reference）和 RFC 文档共同构成。

[The Rust Reference](https://doc.rust-lang.org/stable/reference/)：<https://doc.rust-lang.org/stable/reference/>

[Rust 参考手册 中文版](https://rustwiki.org/zh-CN/reference/)：<https://rustwiki.org/zh-CN/reference/>

Rust 规范文档：https://rustwiki.org/wiki/

Rust 官方文档中文教程：https://rustwiki.org/

### 编译器

Rust 是一门静态编译型语言。Rust官方的编译器叫 rustc，负责将 Rust 源代码编译为可执行文件或其他库文件（.a、.so、.lib、.dll 等）。

Rustc 有如下特点：

- rustc 是跨平台的应用程序，支持UNIX/Linux 等类 UNIX 平台，也支持 Windows 平台。
- rustc 支持交叉编译，可以在当前平台下编译出可运行于其他平台上的应用程序和库。
- rustc 使用 LLVM 作为编译器后端，具有很好的代码生成和优化技术，支持多个目标平台。
- rustc 是用 Rust 语言开发的，包含在 Rust 语言源码中。
- rustc 对 Rust 源码进行词法语法分析、静态类型检查，最终将代码翻译为 LLVM IR。
- rustc 输出的错误信息非常友好和详尽，是开发者的良师益友。

### 核心库

Rust 语言的语法由核心库和标准库共同提供。

Rust 核心库是标准库的基础。

可以通过在模块顶部引入 `#![no_std]` 来使用核心库。

核心库包含如下部分：

- 基础的Trait
- 基本原始类型
- 一些内建宏

### 标准库

标准库包含的内容大概如下：

- 一些基本 trait、原始数据类型、功能型数据类型和常用宏等
- 并发、I/O 和运行时
- 平台抽象
- 底层操作接口
- 可选和错误处理类型 Option 和 Result，以及各种迭代器等

### 包管理器

把按一定规则组织的多个 rs 文件编译后就得到一个包（crate）。

包是 Rust 代码的基本编译单元，也是程序员直接共享代码的基本单元。

Rust 社区的公开第三方包都集中在 crates.io 网站上面，它们的文档被自动发布到 docs.rs 网站上。

Rust 提供了非常方便的 包管理器 Cargo。

https://doc.rust-lang.org/cargo/

https://rustwiki.org/zh-CN/cargo/index.html

```shell
cargo new bin_crate
cargo new --lib lib_crate
```

- cargo new 命令默认可以创建一个用于编写可执行二进制文件的项目。

- cargo new 命令添加 --lib 参数，可以创建用于编写库的项目。

- cargo build 对项目进行编译
- cargo run 项目运行

### 语句与表达式

Rust 中的语法可以分成两大类：语句（Statement）和表达式（Expression）。

语句是指要执行的一些操作和产生副作用的表达式。

表达式主要用于计算求值。

语句分为两种：声明语句（Declaration statement）和表达式语句（Expression statement）。

- 声明语句，用于声明各种语言项（Item），包括声明变量、静态变量、常量、结构体、函数等，以及通过 extern 和 use 关键字引入包和模块等。
- 表达式语句，特指以分号结尾的表达式。此类表达式求值结果将会被舍弃，并总是返回单元类型 ()。

```rust
// extern crate std;
// use std::prelude::v1::*;
fn main() {
  pub fn answer() -> () {
    let a = 40;
    let b = 2;
    assert_eq!(sum(a, b), 42);
  }
  pub fn sum(a: i32, b: i32) -> i32 {
    a + b
  }
  answer();
}
```

Rust 会为每个 crate 都自动引入标准库模块，除非使用 `#![no_std]` 或 `#[no_std]` 属性明确指定了不需要标准库。

关键字 fn 是 function 的缩写。

单元类型拥有唯一的值，就是它本身。

单元类型的概念来自 OCaml，它表示”没有什么特殊的价值“。

函数无返回值的函数默认不需要在函数签名中指定返回类型。

assert_eq! 是宏语句，是Rust提供的断言，允许判断给定的两个表达式求值结果是否相同。

