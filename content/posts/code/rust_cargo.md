---
title: "Rust编程语言入门之cargo、crates.io"
date: 2023-04-08T21:53:47+08:00
draft: false
tags: ["Rust"]
categories: ["Rust"]
---

# cargo、crates.io

### 本章内容

- 通过 release profile 来自定义构建
- 在<https://crates.io/>上发布库
- 通过 workspaces 组织大工程
- 从 <https://crates.io/>来安装库
- 使用自定义命令扩展 cargo

## 一、通过 release profile 来自定义构建

### release profile （发布配置）

- release profile：
  - 是预定义的
  - 可自定义：可使用不同的配置，对代码编译拥有更多的控制
- 每个 profile 的配置都独立于其它的 profile
- cargo 主要的两个 profile：
  - dev profile：适用于开发，cargo build
  - release profile：适用于发布，cargo build --release

### 自定义 profile

- 针对每个 profile，Cargo 都提供了默认的配置
- 如果想自定义 xxxx profile 的配置：
  - 可以在 Cargo.toml 里添加 [profile.xxxx] 区域，在里面覆盖默认配置的子集

```toml
[package]
name = "closure"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]

[profile.dev]
opt-level = 1

[profile.release]
opt-level = 3
```

执行

```bash
closure on  master [?] is 📦 0.1.0 via 🦀 1.67.1 took 2.9s 
➜ cargo build
   Compiling closure v0.1.0 (/Users/qiaopengjun/rust/closure)
    Finished dev [optimized + debuginfo] target(s) in 0.16s

closure on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ 
```

- 对于每个配置的默认值和完整选项，请参见：<https://doc.rust-lang.org/cargo/>

## 二、发布 crate 到 crates.io （1）

### Crates.io

- 可以通过发布包来共享你的代码
- crate 的注册表在 <https://crates.io/>
  - 它会分发已注册的包的源代码
  - 主要托管开源的代码

### 文档注释

- 文档注释：用于生成文档
  - 生成 HTML 文档
  - 显式公共 API 的文档注释：如何使用API
  - 使用 ///
  - 支持 Markdown
  - 放置在被说明条目之前

### 生成 HTML 文档的命令

- cargo doc
  - 它会运行 rustdoc 工具（Rust 安装包自带）
  - 把生成的 HTML 文档放在 target/doc 目录下
- cargo doc --open：
  - 构建当前crate的文档（也包含 crate 依赖项的文档）
  - 在浏览器打开文档

```rust
/// Adds one to the number given.
/// 
/// # Examples
/// 
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
/// 
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}

```

运行

```bash
closure on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ cargo doc  
 Documenting closure v0.1.0 (/Users/qiaopengjun/rust/closure)
    Finished dev [optimized + debuginfo] target(s) in 0.39s

closure on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ cargo doc --open
    Finished dev [optimized + debuginfo] target(s) in 0.00s
     Opening /Users/qiaopengjun/rust/closure/target/doc/closure/index.html

closure on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ cargo doc --open
 Documenting closure v0.1.0 (/Users/qiaopengjun/rust/closure)
    Finished dev [optimized + debuginfo] target(s) in 0.41s
     Opening /Users/qiaopengjun/rust/closure/target/doc/closure/index.html

closure on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
```

### 常用章节

- `# Examples`
- 其它常用的章节：
  - Panics：函数可能发生 panic 的场景
  - Errors：如果函数返回 Result，描述可能的错误种类，以及可导致错误的条件
  - Safety：如果函数处于 unsafe 调用，就应该解释函数 unsafe 的原因，以及调用者确保的使用前提。

### 文档注释作为测试

- 示例代码块的附加值：
  - 运行 cargo test：将把文档注释中的示例代码作为测试来运行

```rust
/// Adds one to the number given.
/// 
/// # Examples
/// 
/// ```
/// let arg = 5;
/// let answer = closure::add_one(arg);
/// 
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}

```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304091309006.png)

### 为包含注释的项添加文档注释

- 符号：//!
- 这类注释通常用描述 crate 和模块：
  - crate root（按惯例 src/lib.rs）
  - 一个模块内，将 crate 或模块作为一个整体进行记录

```rust
//! # Closure Crate
//! 
//! `closure` is a collection of utilities to make performance
//! calculations more convenient.

/// Adds one to the number given.
/// 
/// # Examples
/// 
/// ```
/// let arg = 5;
/// let answer = closure::add_one(arg);
/// 
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}

```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304091307203.png)

## 三、pub use

### 使用 pub use 导出方便使用的公共 API

- 问题：crate 的程序结构在开发时对于开发者很合理，但对于它的使用者不够方便
  - 开发者会把程序结构分为很多层，使用者想找到这种深层结构中的某个类型很费劲
- 例如：
  - 麻烦：`my_crate::some_module::another_module::UsefulType;`
  - 方便：`my_crate::UsefulType;`
- 解决办法：
  - 不需要重新组织内部代码结构
  - 使用pub use：可以重新导出，创建一个与内部私有结构不同的对外公共结构

```rust
//! # Art
//! 
//! A library for modeling artistic concepts.

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use crate::kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        SecondaryColor::Green
    }
}

```

src/main.rs 文件

```rust
use art::kinds::PrimaryColor;
use art::utils::mix;

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}

```

优化之后：

src/lib.rs 文件

```rust
//! # Art
//! 
//! A library for modeling artistic concepts.

pub use self::kinds::PrimaryColor;
pub use self::kinds::SecondaryColor;
pub use self::utils::mix;

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use crate::kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        SecondaryColor::Green
    }
}

```

src/main.rs 文件

```rust
// use art::kinds::PrimaryColor;
// use art::utils::mix;
use art::PrimaryColor;
use art::mix;

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}

```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202304091743568.png)

## 四、发布 crate 到 crates.io（2）

### 创建并设置 Crates.io 账号

- 发布 crate 前，需要在 crates.io 创建账号并获得 API token
- 运行命令：cargo login [你的 API token]
  - 通知 cargo，你的 API token 存储在本地 ~/.cargo/credentials
- API token 可以在<https://crates.io/>进行撤销

### 为新的crate 添加元数据

- 在发布crate之前，需要在 Cargo.toml 的 [package] 区域为 crate 添加一些元数据：
  - crate 需要唯一的名称：name
  - description：一两句话即可，会出现在 crate 搜索的结果里
  - license：需提供许可证标识值（可到 <http://spdx.org/licenses/>查找）
    - 可指定多个 license：用 OR
  - version
  - author
- 发布：cargo publish 命令

```bash
rust_tutorials on  master [?] via 🦀 1.67.1 
➜ cargo login --registry crates-io               
please paste the token found on https://crates.io/me below
ciopLk54SDAxB200gA4jk85abcdefgabcabc # token
       Login token for `crates-io` saved

rust_tutorials on  master [?] via 🦀 1.67.1 took 1m 27.6s 
➜ 

rust_tutorials on  master [?] via 🦀 1.67.1 took 2.0s 
➜ cargo publish --registry crates-io --allow-dirty
    Updating crates.io index
```

### 发布到 Crates.io

- Crate 一旦发布，就是永久性的：该版本无法覆盖，代码无法删除
  - 目的：依赖于该版本的项目可继续正常工作

### 发布已存在 crate 的新版本

- 修改 crate 后，需要先修改 Cargo.toml 里面的version 值，再进行重新发布
- 参照<http://semver.org/>来使用你的语义版本
- 再执行 cargo publish 进行发布

### 使用 cargo yank 从 Crates.io 撤回版本

- 不可以删除 crate 之前的版本
- 但可以防止其它项目把它作为新的依赖：yank（撤回）一个 crate 版本
  - 防止新项目依赖于该版本
  - 已经存在项目可继续将其作为依赖（并可下载）
- yank 意味着：
  - 所有已经产生 Cargo.lock 的项目都不会中断
  - 任何将来生成的 Cargo.lock 文件都不会使用被 yank 的版本
- 命令：
  - yank 一个版本（不会删除任何代码）：cargo yank --vers 1.0.1
  - 取消 yank：cargo yank --vers 1.0.1 --undo

## 五、Cargo 工作空间

### Cargo 工作空间（Workspaces）

- cargo 工作空间：帮助管理多个相互关联且需要协同开发的 crate
- cargo 工作空间是一套共享同一个 Cargo.lock 和输出文件夹的包

### 创建工作空间

- 有多种方式来组建工作空间例：1个二进制 crate，2个库 crate
  - 二进制 crate：main函数，依赖于其它2个crate
  - 其中1个库crate 提供 add_one 函数
  - 另外1个库crate 提供 add_two 函数

```bash
~/rust
➜ mcd add  # mkdir add cd add

~/rust/add
➜ touch Cargo.toml

~/rust/add via 🦀 1.67.1
➜ c  # code .

~/rust/add via 🦀 1.67.1
➜

~/rust/add via 🦀 1.67.1
➜ cargo new adder
warning: compiling this new package may not work due to invalid workspace configuration

current package believes it's in a workspace when it's not:
current:   /Users/qiaopengjun/rust/add/adder/Cargo.toml
workspace: /Users/qiaopengjun/rust/add/Cargo.toml

this may be fixable by adding `adder` to the `workspace.members` array of the manifest located at: /Users/qiaopengjun/rust/add/Cargo.toml
Alternatively, to keep it out of the workspace, add the package to the `workspace.exclude` array, or add an empty `[workspace]` table to the package's manifest.
     Created binary (application) `adder` package

~/rust/add via 🦀 1.67.1
➜ cargo build
   Compiling adder v0.1.0 (/Users/qiaopengjun/rust/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s

~/rust/add via 🦀 1.67.1

~/rust/add via 🦀 1.67.1
➜ cargo new add-one --lib
     Created library `add-one` package

~/rust/add via 🦀 1.67.1
➜
```

add-one/src/lib.rs 文件

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}

```

adder/src/main.rs 文件

```rust
use add_one;

fn main() {
    let num = 10;
    println!("Hello, world! {} plus one is {}", num, add_one::add_one(num));
}

```

adder/Cargo.toml 文件

```toml
[package]
name = "adder"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]

add-one = { path = "../add-one" }

```

rust/add/Cargo.toml 文件

```toml
[workspace]

members = [
    "adder",
    "add-one",
]

```

运行

```bash
~/rust/add via 🦀 1.67.1 
➜ cargo build            
   Compiling add-one v0.1.0 (/Users/qiaopengjun/rust/add/add-one)
   Compiling adder v0.1.0 (/Users/qiaopengjun/rust/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30s

~/rust/add via 🦀 1.67.1 
➜ cargo run -p adder                
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11

~/rust/add via 🦀 1.67.1 
➜ 
```

### 在工作空间中依赖外部 crate

- 工作空间只有一个 Cargo.lock 文件，在工作空间的顶层目录
  - 保证工作空间内所有 crate 使用的依赖的版本都相同
  - 工作空间内所有 crate 相互兼容

### 为工作空间添加测试

rust/add/add-one/src/lib.rs 文件

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}

#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn it_works() {
        assert_eq!(3, add_one(2));
    }
}

```

执行测试

```bash
~/rust/add via 🦀 1.67.1 
➜ cargo test        
   Compiling add-one v0.1.0 (/Users/qiaopengjun/rust/add/add-one)
   Compiling adder v0.1.0 (/Users/qiaopengjun/rust/add/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.13s
     Running unittests src/lib.rs (target/debug/deps/add_one-cb079acb8d173784)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-23a2e001f7410351)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add-one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


~/rust/add via 🦀 1.67.1 
➜ 

~/rust/add via 🦀 1.67.1 
➜ cargo test -p add-one                      
    Finished test [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-cb079acb8d173784)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add-one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


~/rust/add via 🦀 1.67.1 
➜ 
```

## 六、从 CRATES.IO 安装二进制 crate

### 从 CRATES.IO 安装二进制 crate

- 命令：cargo install
- 来源：<https://crates.io>
- 限制：只能安装具有二进制目标（binary target）的 crate
- 二进制目标 binary target：是一个可运行程序
  - 由拥有 src/main.rs 或其它被指定为二进制文件的 crate 生成
- 通常：README 里有关于 crate 的描述：
  - 拥有 library target
  - 拥有 binary target
  - 两者兼备

### cargo install

- cargo install 安装的二进制存放在根目录的 bin 文件夹
- 如果你用 rustup 安装的 Rust，没有任何自定义配置，那么二进制存放目录是 $HOME/.cargo/bin
  - 要确保该目录在环境变量 $PATH 中

```bash
~/rust took 2m 8.1s
➜ cargo install rust_tutorials_qiao
    Updating `tuna` index
  Downloaded rust_tutorials_qiao v0.1.0 (registry `tuna`)
  Downloaded 1 crate (759 B) in 3.43s
  Installing rust_tutorials_qiao v0.1.0
   Compiling rust_tutorials_qiao v0.1.0
    Finished release [optimized] target(s) in 3.99s
  Installing /Users/qiaopengjun/.cargo/bin/rust_tutorials_qiao
   Installed package `rust_tutorials_qiao v0.1.0` (executable `rust_tutorials_qiao`)

~/rust took 4.0s
➜
➜ rust_tutorials_qiao
Hello, world!

~/rust took 3.1s
➜
~/rust took 3.1s
➜ echo $PATH  # 查看PATH环境变量
```

### 使用自定义命令扩展 cargo

- cargo 被设计成可以使用子命令来扩展
- 例：如果 $PATH 中的某个二进制是 cargo-something，你可以像子命令一样运行：
  - cargo something
- 类似这样的自定义命令可以通过该命令列出：`cargo --list`
- 优点：可使用 cargo install 来安装扩展，像内置工具一样来运行

```bash
➜ cargo --list
Installed Commands:
    add                  Add dependencies to a Cargo.toml manifest file
    b                    alias: build
    bench                Execute all benchmarks of a local package
    build                Compile a local package and all of its dependencies
    c                    alias: check
    check                Check a local package and all of its dependencies for errors
    clean                Remove artifacts that cargo has generated in the past
    clippy               Checks a package to catch common mistakes and improve your Rust code.
    config               Inspect configuration values
    d                    alias: doc
    doc                  Build a package's documentation
    fetch                Fetch dependencies of a package from the network
    fix                  Automatically fix lint warnings reported by rustc
    fmt                  Formats all bin and lib files of the current crate using rustfmt.
    generate-lockfile    Generate the lockfile for a package
    git-checkout         This command has been removed
    help                 Displays help for a cargo subcommand
    init                 Create a new cargo package in an existing directory
    install              Install a Rust binary. Default location is $HOME/.cargo/bin
    locate-project       Print a JSON representation of a Cargo.toml file's location
    login                Save an api token from the registry locally. If token is not specified, it will be read from stdin.
    logout               Remove an API token from the registry locally
    metadata             Output the resolved dependencies of a package, the concrete used versions including overrides, in machine-readable format
    miri
    new                  Create a new cargo package at <path>
    owner                Manage the owners of a crate on the registry
    package              Assemble the local package into a distributable tarball
    pkgid                Print a fully qualified package specification
    publish              Upload a package to the registry
    r                    alias: run
    read-manifest        Print a JSON representation of a Cargo.toml manifest.
    remove               Remove dependencies from a Cargo.toml manifest file
    report               Generate and display various kinds of reports
    rm                   alias: remove
    run                  Run a binary or example of the local package
    rustc                Compile a package, and pass extra options to the compiler
    rustdoc              Build a package's documentation, using specified custom flags.
    search               Search packages in crates.io
    t                    alias: test
    test                 Execute all unit and integration tests and build examples of a local package
    tree                 Display a tree visualization of a dependency graph
    uninstall            Remove a Rust binary
    update               Update dependencies as recorded in the local lock file
    vendor               Vendor all dependencies for a project locally
    verify-project       Check correctness of crate manifest
    version              Show version information
    yank                 Remove a pushed crate from the index

~/rust
```
