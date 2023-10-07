---
title: "Rust编程语言入门"
date: 2023-02-14T21:16:03+08:00
draft: false
Tags : ["Rust"]
Categories : ["Rust"]
---

# Rust 编程语言入门

## Rust简介

### 为什么要用Rust？

- Rust是一种令人兴奋的新编程语言，它可以让每个人编写可靠且高效的软件。
- 它可以用来替换C/C++，Rust和他们具有同样的性能，但是很多常见的bug在编译时就可以被消灭。
- Rust是一种通用的编程语言，但是它更善于以下场景：
  - 需要运行时的速度
  - 需要内存安全
  - 更好的利用多处理器

### 与其他语言比较

- C/C++性能非常好，但类型系统和内存都不太安全。
- Java/C#，拥有GC，能保证内存安全，也有很多优秀特性，但是性能不行。
- Rust：
  - 安全
  - 无需GC（性能好速度快）
  - 易于维护、调试、代码安全高效

### Rust特别擅长的领域

- 高性能 Web Service (Web API)
- WebAssembly
- 命令行工具
- 网络编程
- 嵌入式设备
- 系统编程

### Rust与Firefox

- Rust最初是Mazilla公司的一个研究性项目。Firefox是Rust产品应用的一个重要的例子。
- Mazilla 一直以来都在用Rust创建一个名为Servo的实验性浏览器引擎，其中的所有内容都是并行执行的。
  - 目前Servo的部分功能已经被集成到Firefox里面了
- Firefox原来的量子版就包含了Servo的CSS渲染引擎
  - Rust使得Firefox在这方面得到了巨大的性能改进

### Rust的用户和案例

- Google：新操作系统Fuschia，其中Rust代码量大约占30%
- Amazon：基于Linux开发的直接可以在裸机、虚机上运行容器的操作系统
- System76、百度、华为、蚂蚁金服...

### Rust的优点

- 性能
- 安全性
- 无所畏惧的并发

### Rust的缺点

- 学习曲线高 ”难学“

### 注意

- Rust有很多独有的概念，要一步一步学习

## Rust 安装

官网：<https://www.rust-lang.org/zh-CN/learn/get-started>

Windows：按官网指示操作

Mac 安装：

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202302181555079.png)

### 更新与卸载Rust

- 更新Rust

```rust
rustup update
```

- 卸载Rust

```rust
rustup self uninstall
```

### 安装验证

- `rustc --version`
  - 结构格式：rustc x.y.z(abcdbcdbc yyyy-mm-dd)
  - 会现实最新稳定版：版本号、commit hash、commit日期

### 本地文档

- 安装Rust的时候，会在本地安装文档，可离线浏览
- 运行`rustup doc`可在浏览器打开本地文档

```bash
➜ cargo --version
cargo 1.67.1 (8ecd4f20a 2023-01-10)

~
➜ rustc --version
rustc 1.67.1 (d5a82bbd2 2023-02-07)

~
➜ rustup doc
```

### 开发工具

- Visual Studio Code
  - Rust 插件
- Pycharm（Intellij Idea 系列）
  - Rust插件

### Hello World 例子

编写Rust程序

- 程序文件后缀名：rs
- 文件命名规范：hello_world.rs

```bash
➜ mkdir rust

~
➜ cd rust

~/rust
➜ mkdir hello_world

~/rust
➜ cd hello_world

~/rust/hello_world
➜ code .

~/rust/hello_world
➜ pwd
/Users/qiaopengjun/rust/hello_world

~/rust/hello_world via 🦀 1.67.1
➜ mv hello_world.rs main.rs

~/rust/hello_world via 🦀 1.67.1
➜ rustc main.rs

~/rust/hello_world via 🦀 1.67.1
➜ ls
main    main.rs

➜ ./main
Hello World!
```

main.rs`文件

```rust
fn main() {
    println!("Hello World!");
}
```

### Rust 程序解剖

- 定义函数：`fn main(){}`
  - 没有参数，没有返回
- `main`函数很特别：它是每个Rust可执行程序最先运行的代码
- 打印文本：`printIn!("Hello, world!");`
  - Rust的缩进是4个空格而不是tab
  - printIn!是一个Rust macro（宏）
    - 如果是函数的话，就没有!
  - "Hello World" 是字符串，它是printIn!的参数
  - 这行代码以;结尾

## 编译和运行是单独的两步

- 运行Rust程序之前必须先编译，命令为：`rustc`源文件名
  - `rustc main.rs`
- 编译成功后，会生成一个二进制文件
  - 在Windows上还会生产一个`.pdb`文件，里面包含调试信息
- Rust是ahead-of-time编译的语言（预先编译）
  - 可以先编译程序，然后把可执行文件交给别人运行（无需安装Rust）
- Rustc 只适合简单的Rust程序...

### Hello Cargo

### Cargo

- Cargo 是Rust的构建系统和包管理工具
  - 构建代码、下载依赖的库、构建这些苦...
- 安装Rust的时候会安装Cargo
  - Cargo --version

```rust
~/rust/hello_world via 🦀 1.67.1
➜ cargo --version
cargo 1.67.1 (8ecd4f20a 2023-01-10)
```

### 使用Cargo创建项目

- 创建项目：`cargo new hello_cargo`
  - 项目名称也是`hello_cargo`
  - 会创建一个新的目录`hello_cargo`
    - Cargo.toml
    - src目录
      - `main.rs`
    - 初始化了一个新的Git仓库 `.gitignore`
      - 可以使用其它的VCS或不使用VCS：cargo new 的时候使用 --vcs 这个flag

```bash
~/rust
➜ cargo new hello_cargo
     Created binary (application) `hello_cargo` package

~/rust

➜ ls
hello_cargo hello_world

~/rust
➜ cd hello_cargo

hello_cargo on  master [?] via 🦀 1.67.1
➜ ls
Cargo.toml src

hello_cargo on  master [?] via 🦀 1.67.1
➜ ➜ ls
Cargo.toml src
```

#### `Cargo.toml`

- TOML（Tom's Obvious, Minimal Language）格式，是Cargo的配置格式
- [package]，是一个区域标题，表示下方内容是用来配置包（package）的
  - name 项目名
  - version 项目版本
  - authors 项目作者
  - edition 使用的Rust版本
- [dependencies] 另一个区域的开始，它会列出项目的依赖项
- 在Rust里面，代码的包称作crate

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202302162243199.png)

#### `src/main.rs`

- cargo 生成的main.rs在src目录下
- 而Cargo.toml在项目顶层下
- 源代码都应该在src目录下
- 顶层目录可以放置：README、许可信息、配置文件和其它与程序源码无关的文件
- 如果创建项目时没有使用cargo，也可以把项目转化为使用cargo：
  - 把源代码文件移动到src下
  - 创建Cargo.toml并填写相应的配置

### 构建Cargo项目 cargo build

- cargo build
  - 创建可执行文件：target/debug/hello_cargo或target\debug\hello_cargo.exe(Windows)
  - 运行可执行文件：`./target/debug/hello_cargo`或`.\target\debug\hello_cargo.exe(Windows)`
- 第一次运行`cargo build`会在顶层目录生成cargo.lock文件
  - 该文件负责追踪项目依赖的精确版本
  - 不需要手动修改该文件

### 构建和运行cargo项目 `cargo run`

- cargo run 编译代码 + 执行结果
  - 如果之前编译成功过，并且源码没有改变，那么就会直接运行二进制文件

```rust
hello_cargo on  master [?] is 📦 0.1.0 via 🦀 1.67.1
➜ cargo run
   Compiling hello_cargo v0.1.0 (/Users/qiaopengjun/rust/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.38s
     Running `target/debug/hello_cargo`
Hello, world!

hello_cargo on  master [?] is 📦 0.1.0 via 🦀 1.67.1
➜
```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202302162243594.png)

#### `cargo check`

- cargo check 检查代码，确保能通过编译，但是不产生任何可执行文件
- cargo chekc 要比 cargo build 快得多
  - 编写代码的时候可以连续反复的使用cargo chekc 检查代码，提高效率

### 为发布构建

- cargo build --release
  - 编译时会进行优化
    - 代码会运行的更快，但是编译时间更长
  - 会在target/release而不是target/debug生成可执行文件
- 两种配置
  - 开发
  - 正式发布

尽量用Cargo

## 猜数游戏-一次猜测

#### 猜数游戏 - 目标

- 生成一个1到100间的随机数
- 提示玩家输入一个猜测
- 猜完之后，程序会提示猜测是太小了还是太大了
- 如果猜测正确，那么打印出一个庆祝信息，程序退出

### 写代码

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202302182323253.png)

```rust
use std::io; // prelude

fn main() {
    println!("猜数!");

    println!("猜测一个数");

    // let mut foo = 1;
    // let bar = foo; // immutable

    // foo = 2;

    let mut guess = String::new();

    io::stdin().read_line(&mut guess).expect("无法读取行");
    // io::Result Ok Err 

    println!("你猜测的数是：{}", guess);
}

```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202302190029828.png)

## 猜数游戏 - 生成神秘数字

[rang]<https://crates.io/crates/rand>

```bash
➜ cd rust/guessing_game

guessing_game on  main [✘!] is 📦 0.1.0 via 🦀 1.67.1
➜ cargo build
   Compiling cfg-if v1.0.0
   Compiling ppv-lite86 v0.2.17
   Compiling libc v0.2.139
   Compiling getrandom v0.2.8
   Compiling rand_core v0.6.4
   Compiling rand_chacha v0.3.1
   Compiling rand v0.8.5
   Compiling guessing_game v0.1.0 (/Users/qiaopengjun/rust/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 1.08s

guessing_game on  main [✘!] is 📦 0.1.0 via 🦀 1.67.1
➜ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.03s

guessing_game on  main [✘!] is 📦 0.1.0 via 🦀 1.67.1
➜ cargo update
```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202302202323209.png)

随机数

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202302202342562.png)

代码：

```rust
use std::io; // prelude
use rand::Rng; // trait

fn main() {
    println!("猜数!");

    let secret_number = rand::thread_rng().gen_range(1..101);

    println!("神秘数字是：{}", secret_number);

    println!("猜测一个数");

    // let mut foo = 1;
    // let bar = foo; // immutable

    // foo = 2;

    let mut guess = String::new();

    io::stdin().read_line(&mut guess).expect("无法读取行");
    // io::Result Ok Err 

    println!("你猜测的数是：{}", guess);
}

```

## 猜数游戏 - 比较猜测数字与神秘数字

```rust
use std::io; // prelude
use rand::Rng; // trait
use std::cmp::Ordering; // 枚举类型 三个变体（值）

fn main() {
    println!("猜数!");

    let secret_number = rand::thread_rng().gen_range(1..101);

    println!("神秘数字是：{}", secret_number);

    println!("猜测一个数");

    // let mut foo = 1;
    // let bar = foo; // immutable

    // foo = 2;

    let mut guess = String::new();

    io::stdin().read_line(&mut guess).expect("无法读取行");
    // io::Result Ok Err 

    // shadow
    let guess: u32 = guess.trim().parse().expect("Please type a number!"); // \n

    println!("你猜测的数是：{}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"), // arm
        Ordering::Greater => println!("Too big!"), 
        Ordering::Equal => println!("You win!"),
    }
}

```

## 猜数游戏 - 允许多次猜测

```rust
use std::io; // prelude
use rand::Rng; // trait
use std::cmp::Ordering; // 枚举类型 三个变体（值）

fn main() {
    println!("猜数!");

    let secret_number = rand::thread_rng().gen_range(1..101);

    // println!("神秘数字是：{}", secret_number);

    loop {
        println!("猜测一个数");

        // let mut foo = 1;
        // let bar = foo; // immutable

        // foo = 2;

        let mut guess = String::new();

        io::stdin().read_line(&mut guess).expect("无法读取行");
        // io::Result Ok Err 

        // shadow
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => {
                println!("请输入正确的数字");
                continue;
            }
        };
        println!("你猜测的数是：{}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"), // arm
            Ordering::Greater => println!("Too big!"), 
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}

```

## 通用的编程概念

- 变量与可变性
- 数据类型
  - 标量类型
  - 符合类型
- 函数
- 注释

### 1 变量与可变性

- 声明变量使用`let`关键字
- 默认情况下，变量是不可变的（Immutable）
  - （例子 variables）
- 声明变量时，在变量前面加上`mut`，就可以使变量可变。

```bash
~/rust
➜ cargo new variables
     Created binary (application) `variables` package

~/rust
➜ cd var*

variables on  master [?] via 🦀 1.67.1
➜ code .

variables on  master [?] via 🦀 1.67.1
➜
```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202302250858666.png)

代码：

```rust
fn main() {
    println!("Hello, world!");

    let mut x = 5;
    println!("The value of x is {}", x);

    x = 6;
    println!("The value of x is {}", x);
}

```

#### 变量与常量

- 常量（constant），常量在绑定值以后也是不可变的，但是它与不可变的变量有很多区别：
  - 不可以使用`mut`，常量永远都是不可变的
  - 声明常量使用`const`关键字，它的类型必须被标注
  - 常量可以在任何作用域内进行声明，包括全局作用域
  - 常量只可以绑定到常量表达式，无法绑定到函数的调用结果或只能在运行时才能计算出的值
- 在程序运行期间，常量在其声明的作用域内一直有效
- 命名规范：Rust里常量使用全大写字母，每个单词之间用下划线分开，例如：`MAX_POINTS`
- 例子：`const MAX_POINTS: u32 = 100_000;`

#### Shadowing（隐藏）

- 可以使用相同的名字声明新的变量，新的变量就会shadow（隐藏）之前声明的同名变量
  - 在后续的代码中这个变量名代表的就是新的变量
- Shadow 和把变量标记为mut是不一样的：
  - 如果不使用let关键字，那么重新给非mut的变量赋值会导致编译时错误
  - 而使用let声明的同名新变量，也是不可变的
  - 使用let声明的同名新变量，它的类型可以与之前不同

```rust
// const MAX_POINTS: u32 = 100_000;

fn main() {
    // const MAX_POINTS: u32 = 100_000;
    println!("Hello, world!");

    let mut x = 5;
    println!("The value of x is {}", x);

    x = 6;
    println!("The value of x is {}", x);

    let x = x + 1;
    let x = x * 2;
    println!("The value of x is {}", x);

    let spaces = "    ";
    let spaces = spaces.len();
    println!("The length of spaces is {}", spaces);
  
   let guess: u32 = "42".parse().expect("Not a number");

    println!("The guess is {}", guess);
}

```

### 2 数据类型

- 标量和复合类型
- Rust是静态编译语言，在编译时必须知道所有变量的类型
  - 基于使用的值，编译器通常能够推断出它的具体类型
  - 但如果可能的类型比较多（例如把String转为整数的parse方法），就必须添加类型的标注，否则编译会报错

#### 标量类型

- 一个标量类型代表一个单个的值
- Rust有四个主要的标量类型：
  - 整数类型
  - 浮点类型
  - 布尔类型
  - 字符类型

整数类型

- 整数类型没有小数部分
- 例如u32就是一个无符号的整数类型，占据32位的空间
- 无符号整数类型以u开头
- 有符号整数类型以i开头
- Rust的整数类型列表：
  - 每种都分i和u，以及固定的位数
  - 有符号范围： -(2^n-1)到2^{n-1}-1
  - 无符号范围：0到2^n-1

| length  | signed | unsigned |
| :------ | ------ | -------- |
| 8-bit   | i8     | u8       |
| 16-bit  | i16    | u16      |
| 32-bit  | i32    | u32      |
| 64-bit  | i64    | u64      |
| 128-bit | i128   | u128     |
| arch    | isize  | usize    |

#### isize和usize类型

- isize和usize类型的位数由程序运行的计算机的架构所决定：
  - 如果是64位计算机，那就是64位的
  - ...
- 使用isize或usize的主要场景是对某种集合进行索引操作。

#### 整数字面值

十进制、十六进制、八进制、二进制、byte

- 除了byte类型外，所有的数值字面值都允许使用类型后缀。例如 57u8
- 如果你不太清楚应该使用那种类型，可以使用Rust相应的默认类型
- 整数的默认类型就是i32，总体上来说速度很快，即使在64位系统中

#### 整数溢出

- 例如：u8的范围是0-255，如果你把一个u8变量的值设为256，那么：
  - 调试模型下编译：Rust会检查整数溢出，如果发生溢出，程序在运行时就会panic
  - 发布模型下(--release)编译：Rust不会检查可能导致panic的整数溢出
    - 如果溢出发生：Rust会执行“环绕”操作：256变成0,257变成1... 但程序不会panic

### 浮点类型

- Rust有两种基础的浮点类型，也就是含有小数部分的类型
  - f32，32位，单精度
  - f64，64位，双精度
- Rust 的浮点类型使用了IEEE-754标准来表述
- f64是默认类型，因为在现代CPU上f64和f32的速度差不多，而且精度更高。

### 数值操作

- 加减乘除余等

### 布尔类型

- Rust的布尔类型也有两个值：true 和 false
- 一个字节大小
- 符号是bool

### 字符类型

- Rust语言中char类型被用来描述语言中最基础的单个字符
- 字符类型的字面值使用单引号
- 占用4字节大小
- 是Unicode标量值，可以表示比ASCII多得多的字符内容：拼音、中日韩文、零长度空白字符、emoji表情等
  - U+0000 到 U+D7FF
  - U+E000 到 U+10FFFF
- 但Unicode中并没有“字符”的概念，所以直觉上认为的字符也许与Rust中的概念并不相符

```rust
// const MAX_POINTS: u32 = 100_000;

fn main() {
    // const MAX_POINTS: u32 = 100_000;
    println!("Hello, world!");

    let mut x = 5;
    println!("The value of x is {}", x);

    x = 6;
    println!("The value of x is {}", x);

    let x = x + 1;
    let x = x * 2;
    println!("The value of x is {}", x);

    let spaces = "    ";
    let spaces = spaces.len();
    println!("The length of spaces is {}", spaces);

    let guess: u32 = "42".parse().expect("Not a number");

    println!("The guess is {}", guess);

    // let x = 2.0; // f64

    // let y: f32 = 3.0; // f32

    // let sum = 5 + 10;

    // let difference = 95.5 - 4.3;

    // let product = 4 * 30;

    // let quotient = 56.7 / 32.2;

    // let reminder = 54 % 5;

    // let t = true;

    // let f: bool = false;

    // let x = 'z';
    // let y: char = 'a';
    // let z = '😘';
}

```

### 复合类型

- 复合类型可以将多个值放在一个类型里
- Rust提供了两种基础的复合类型：元组（Tuple）、数组

#### Tuple

- Tuple  可以将多个类型的多个值放在一个类型里
- Tuple  的长度是固定的：一旦声明就无法改变

#### 创建Tuple

- 在小括号里，将值用逗号分开
- Tuple 中的每个位置都对应一个类型，Tuple中各元素的类型不必相同

#### 获取Tuple的元素值

- 可以使用模式匹配来解构（destructure）一个Tuple来获取元素的值

#### 访问Tuple的元素

- 在Tuple变量使用点标记法，后接元素的索引号

```rust
fn main() {

    let tup: (i32, f64, u8) = (500, 6.4, 1);

    println!("{}, {}, {}", tup.0, tup.1, tup.2);

    let (x, y, z) = tup;
    println!("{}, {}, {}", x, y, z);
}

```

#### 数组

- 数组也可以将多个值放在一个类型里
- 数组中每个元素的类型必须相同
- 数组的长度也是固定的

#### 声明一个数组

- 在中括号里，各值用逗号分开

#### 数组的用处

- 如果想让你的数据存放在stack（栈）上而不是heap（堆）上，或者想保证有固定数量的元素，这时使用数组更有好处
- 数组没有Vector灵活
  - Vector和数组类似，它由标准款提供
  - Vector的长度可以改变
  - 如果你不确定应该用数组还是Vector，那么估计你应该用Vector

#### 数组的类型

- 数组的类型以这种形式表示：[类型; 长度]  例如：`let a: [i32; 5] = [1, 2, 3, 4, 5];`

#### 另一种声明数组的方法

- 如果数组的每个元素值都相同，那么可以在：
  - 在中括号里指定初始值
  - 然后是一个“;”
  - 最后是数组的长度
- 例如：`let a = [3; 5];` 它就相当于：`let a = [3, 3, 3, 3, 3];`

#### 访问数组的元素

- 数组是stack上分配的单个块的内存
- 可以使用索引来访问数组的元素
- 如果访问的索引超出了数组的范围，那么：
  - 编译会通过
  - 运行会报错（runtime时会panic）
    - Rust不会允许其继续访问相应地址的内存

```rust
fn main() {
    // let a = [1, 2, 3, 4, 5];

    let months = [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
    ];

    // let first = months[0];
    // let second = months[1];

    // let index = 15;
    let index = [12, 13, 14, 15];
    let month = months[index[1]];
    println!("{}", month);
}

```

### 2 函数

- 声明函数使用fn关键字
- 依照惯例，针对函数和变量名，Rust使用snake case 命名规范
  - 所有的字母都是小写的，单词之间使用下划线分开

```rust
fn main() {
    another_function();
}

fn another_function() {
    println!("Another function");
}

```

#### 函数的参数

- parameters，arguments  (形参，实参)
- 在函数签名里，必须声明每个参数的类型

```rust
fn main() {
   another_function(5, 6); // argument
}

fn another_function(x: i32, y: i32) { // parameter
    println!("the value of x is {}", x);
    println!("the value of y is {}", y);
}
```

#### 函数体中的语句与表达式

- 函数体由一系列语句组成，可选的由一个表达式结束
- Rust是一个基于表达式的语言
- 语句是执行一些动作的指令
- 表达式会计算产生一个值
- 函数的定义也是语句
- 语句不返回值，所以不可以使用let将一个语句赋给一个变量

```rust
fn main() {
    let y = 6; // 语句
    // let x = (let y = 6); // 报错
  
   let x = 5;

    let y = {
        let x = 1;
        x + 3
    };

    println!("The value of y is {}", y)
}
```

#### 函数的返回值

- 在->符号后边声明函数返回值的类型，但是不可以为返回值命名
- 在Rust里面，返回值就是函数体里面最后一个表达式的值
- 若想提前返回，需使用return关键字，并指定一个值
  - 大多数函数都是默认使用最后一个表达式作为返回值

```rust
fn plus_five(x: i32) -> i32 {
    x + 5
}

fn main() {
   let x = plus_five(6);
   println!("The value of x is {}", x);
}
```

#### 注释

- Rust还有一种“文档注释”

```rust
// This is a function
fn five(x: i32) -> i32 {
  x + 5
}

/* sdgagaf
sfgagaga */
// This is main function
// The entry point
fn main() {
  // call five()
  let x = five(6); // 5 + 6
  
  println!("The value of x is: {}", x);
}
```

### 3 控制流

#### if 表达式

- if表达式允许您根据条件来执行不同的代码分支
  - 这个条件必须是bool类型
- if表达式中，与条件相关联的代码块叫做分支（arm）
- 可选的，在后边可以加上一个else表达式

```rust
fn main() {
    println!("Hello, world!");

    let number = 7;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}

```

#### 使用else if 处理多重条件

- 如果使用多于一个else if，那么最好使用match来重构代码

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3 or 2");
    }
}

```

使用match

```rust
fn main() {
    let number = 6;
    match number {
        number if number % 4 == 0 => println!("number is divisible by 4"),
        number if number % 3 == 0 => println!("number is divisible by 3"),
        number if number % 2 == 0 => println!("number is divisible by 2"),
        _ => println!("number is not divisible by 4, 3 or 2"),
    }
}

```

#### 在let语句中使用If

- 因为if是一个表达式，所以可以将它放在let语句中等号的右边

```rust
fn main() {
    let condition = true;

    let number = if condition {5} else {6};

    println!("The value of number is {}", number);
}
```

#### Rust的循环

- Rust提供了3种循环：loop，while和for

#### loop循环

- loop关键字告诉Rust反复的执行一块代码，直到你喊停
- 可以在loop循环中使用break关键字来告诉程序何时停止循环

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}

```

#### While 条件循环

- 每次执行循环体之前都判断一次条件

```rust
fn main() {
  let mut number = 3;
  
  while number != 0 {
    println!("{}", number);
    number = number - 1;
  }
  println!("LIFTOFF!!!");
}
```

#### 使用for循环遍历集合

- 可以使用while或loop来遍历集合，但是易错且低效

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("the index is {}", a[index]);

        index = index + 1;
    }
}

```

- 使用for循环更简洁紧凑，它可以针对集合中的每个元素来执行一些代码

```rust
fn main() {
  let a = [10, 20, 30, 40, 50];
  for element in a.iter() {
    println!("the value is {}", element);
  }
}
```

- 由于for循环的安全、简洁性，所以它在Rust里用的最多

#### Range

- 标准库提供
- 指定一个开始数字和一个结束数字，Range可以生成它们之间的数字（不含结束）
- rev方法可以反转Range

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{}", number);
    }
    println!("LIFTOFF!!!");
}

```

### 4 所有权

- 所有权是Rust最独特的特性，它让Rust无需GC就可以保证内存安全

#### 4.1 什么是所有权

- Rust的核心特性就是所有权
- 所有程序在运行时都必须管理它们使用计算机内存的方式
  - 有些语言有垃圾收集机制，在程序运行时，它们会不断地寻找不再使用的内存
  - 在其他语言中，程序员必须显示地分配和释放内存
- Rust采用了第三种方式：
  - 内存是通过一个所有权系统来管理的，其中包含一组编译器在编译时检查的规则
  - 当程序运行时，所有权特性不会减慢程序的运行速度

#### Stack VS Heap

栈内存 VS 堆内存

- 在像Rust这样的系统级编程语言里，一个值是在stack上还是在heap上对语言的行为和你为什么要做某些决定时有更大的影响的
- 在你的代码运行的时候，stack和heap都是你可用的内存，但他们的结构很不相同

存储数据

- Stack按值的接收顺序来存储，按相反的顺序将它们移除（后进先出，LIFO）
  - 添加数据叫做压入栈
  - 移除数据叫做弹出栈
- 所有存储在stack上的数据必须拥有已知的固定的大小
  - 编译时大小未知的数据或运行时大小可能发生变化的数据必须存放在heap上
- Heap内存组织性差一些
  - 当你把数据放入heap时，你会请求一定数量的空间
  - 操作系统在heap里找到一块足够大的空间，把它标记为在用，并返回一个指针，也就是这个空间的地址
  - 这个过程叫做在heap上进行分配，有时仅仅称为“分配”
- 把值压到stack上不叫分配
- 因为指针是已知固定大小的，可以把指针存放在stack上
  - 但如果想要实际数据，你必须使用指针来定位
- 把数据压到stack上要比在heap上分配快得多
  - 因为操作系统不需要寻找用来存储新数据的空间，那个位置永远都在stack的顶端
- 在heap上分配空间需要做更多的工作
  - 操作系统首先需要找到一个足够大的空间来存放数据，然后要做好记录方便下次分配

访问数据

- 访问heap中的数据要比访问stack中的数据慢，因为需要通过指针才能找到heap中的数据
  - 对于现代的处理器来说，由于缓存的缘故，如果指令在内存中跳转的次数越少，那么速度就越快
- 如果数据存放的距离比较近，那么处理器的处理速度就会更快一些（stack上）
- 如果数据之间的距离比较远，那么处理速度就会慢一些（heap上）
  - 在heap上分配大量的空间也是需要时间的

函数调用

- 当你的代码调佣函数时，值被传入到函数（也包括指向heap的指针）。函数本地的变量被压到stack上。当函数结束后，这些值会从stack上弹出

所有权存在的原因

- 所有权解决的问题：
  - 跟踪代码的哪些部分正在使用heap的哪些数据
  - 最小化heap上的重复数据量
  - 清理heap上未使用的数据以避免空间不足
- 一旦你懂的了所有权，那么就不需要经常去想stack或heap了
- 但是知道管理heap数据是所有权存在的原因，这有助于解释它为什么会这样工作

#### 所有权规则

- 每个值都有一个变量，这个变量是该值的所有者
- 每个值同时只能有一个所有者
- 当所有者超出作用域（scope）时，该值将被删除

#### 变量作用域

- Scope就是程序中一个项目的有效范围

```rust
fn main() {
  // s 不可用
  let s = "hello"; // s 可用
                   // 可以对 s 进行相关操作 
} // s 作用域到此结束，s 不再可用
```

#### String 类型

- String 比那些基础标量数据类型更复杂
- 字符串字面值：程序里手写的那些字符串值。它们是不可变的
- Rust还有第二种字符串类型：String
  - 在heap上分配。能够存储在编译时未知数量的文本

#### 创建String类型的值

- 可以使用from函数从字符串字面值创建出String类型
- let s = String::from("hellp");
  - “::”表示from是String类型下的函数
- 这类字符串是可以被修改的

```rust
fn main() {
    let mut s = String::from("Hello");

    s.push_str(", World");

    println!("{}", s);
}

```

- 为什么String类型的值可以修改，而字符串字面值却不能修改？
  - 因为它们处理内存的方式不同

#### 内存和分配

- 字符串字面值，在编译时就知道它的内容了，其文本内容直接被硬编码到最终的可执行文件里
  - 速度快、高效。是因为其不可变性
- String类型，为了支持可变性，需要在heap上分配内存来保存编译时未知的文本内容：
  - 操作系统必须在运行时来请求内存
    - 这步通过调用`String::from`来实现
  - 当用完String之后，需要使用某种方式将内存返回给操作系统
    - 这步，在拥有GC的语言中，GC会跟踪并清理不再使用的内存
    - 没有GC，就需要我们去识别内存何时不再使用，并调用代码将它返回
      - 如果忘了，那就浪费内存
      - 如果提前做了，变量就会非法
      - 如果做了两次，也是Bug，必须一次分配对应一次释放
- Rust采用了不同的方式：对于某个值来说，当拥有它的变量走出作用范围时，内存会立即自动的交还给操作系统
- DROP函数（当变量走出作用域的时候，Rust会自动调用这个函数）

#### 变量和数据交互的方式：移动（Move）

- 多个变量可以与同一个数据使用一种独特的方式来交互

```rust
let x = 5;
let y = x;
```

- 整数是已知且固定大小的简单的值，这两个5倍压到了stack中

String版本

```rust
let s1 = String::from("hello");
let s2 = s1;
```

- 一个String由3部分组成：
  - 一个指向存放字符串内容的内存的指针（ptr）
  - 一个长度（len）
  - 一个容量（capacity）
- 上面这些东西放在stack上
- 存放字符串内容的部分在heap上
- 长度len，就是存放字符串内容所需的字节数
- 容量capacity是指String从操作系统总共获得内存的总字节数
- 当把s1赋给s2，String的数据被复制了一份：
  - 在stack上复制了一份指针、长度、容量
  - 并没有复制指针所指向的heap上的数据
- 当变量离开作用域时，Rust会自动调用DROP函数，并将变量使用的heap内存释放
- 当s1、s2离开作用域时，它们都会尝试释放相同的内存：
  - 二次释放（double free）bug
- 为了保证内存安全：
  - Rust没有尝试复制被分配的内存
  - Rust让s1失效
    - 当s1离开作用域的时候，Rust不需要释放任何东西
- 浅拷贝（shallow copy）
- 深拷贝（deep copy）
- 你也许会将复制指针、长度、容量视为浅拷贝，但由于Rust让s1失效了，所以我们用一个新的术语：移动（Move）
- 隐含的一个设计原则：Rust不会自动创建数据的深拷贝
  - 就运行时性能而言，任何自动赋值的操作都是廉价的

#### 变量和数据交互的方式：克隆（Clone）

- 如果真想对heap上面的String数据进行深度拷贝，而不仅仅是stack上的数据，可以使用Clone方法

```rust
fn main() {
  let s1 = String::from("Hello");
  let s2 = s1.clone();
  
  println!("{}, {}", s1, s2);
}
```

#### Stack上的数据：复制

```rust
fn main() {
  let x = 5;
  let y = x;
  
  println!("{}, {}", x, y);
}
```

- Copy trait， 可以用于像整数这样完全存放在stack上面的类型
- 如果一个类型实现了Copy这个trait，那么旧的变量在赋值后仍然可用
- 如果一个类型或者该类型的一部分实现了Drop trait，那么Rust不允许让它再去实现Copy Trait了

#### 一些拥有Copy Trait 的类型

- 任何简单标量的组合类型都可以是Copy的
- 任何需要分配内存或某种资源的都不是Copy的
- 一些拥有Copy Trait的类型：
  - 所有的整数类型，例如u32
  - bool
  - char
  - 所有的浮点类型，例如f64
  - Tuple（元组），如果起所有的字段都是Copy的
    - `(i32, i32)`是
    - `(i32, String)`不是

#### 所有权与函数

- 在语义上，将值传递给函数和把值赋给变量是类似的：
  - 将值传递给函数将发生移动或复制

```rust
fn main() {
    let s = String::from("Hello, World!");

    take_ownership(s);

    let x = 5;

    makes_copy(x);

    println!("x: {}", x);
}

fn take_ownership(some_string: String) {
    println!("{}", some_string)
}

fn makes_copy(some_number: i32) {
    println!("{}", some_number);
}

```

#### 返回值与作用域

- 函数在返回值的过程中同样也会发生所有权的转移

```rust
fn main() {
    let s1 = gives_ownership();

    let s2 = String::from("hello");

    let s3 = takes_and_gives_back(s2);
}

fn gives_ownership() -> String {
    let some_string = String::from("hello");
    some_string
}

fn takes_and_gives_back(a_string: String) -> String {
    a_string
}

```

- 一个变量的所有权总是遵循同样的模型：
  - 把一个值赋给其他变量时就会发生移动
  - 当一个包含heap数据的变量离开作用域时，它的值就会被Drop函数清理，除非数据的所有权移动到另一个变量上了

#### 如何让函数使用某个值，但不获得其所有权？

```rust
fn main() {
  let s1 = String::from("hello");
  
  let (s2, len) = calculate_length(s1);
  
  println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
  let length = s.len();
  
  (s, length)
}
```

- Rust有一个特性叫做“引用（Reference）”

### 4.2 引用和借用

```rust
fn main() {
  let s1 = String::from("Hello");
  let len = calculate_length(&s1);
  
  println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
  s.len()
}
```

- 参数的类型是 &String 而不是 String
- & 符号就表示引用：允许你引用某些值而不取得其所有权

#### 借用

- 我们把引用作为函数参数这个行为叫做借用
- 是否可以修改借用的东西？ 不行
- 和变量一样，引用默认也是不可变的

#### 可变引用

- 可变引用有一个重要的限制：在特定作用域内，对某一块数据，只能有一个可变的引用
  - 这样做的好处是可在编译时防止数据竞争
- 以下三种行为下会发生数据竞争：
  - 两个或多个指针同时访问同一个数据
  - 至少有一个指针用于写入数据
  - 没有使用任何机制来同步对数据的访问
- 可以通过创建新的作用域，来允许非同时的创建多个可变引用

```rust
fn main() {
  let mut s = String::from("Hello");
  {
    let s1 = &mut s;
  }
  
  let s2 = &mut s;
}
```

#### 另外一个限制

- 不可用同时拥有一个可变引用和一个不可变的引用
- 多个不可变的引用是可以的

```rust
fn main() {
  let mut s = String::from("Hello");
  let r1 = &s;
  let r2 = &s;
  let s1 = &mut s;  // 报错
  
  println!("{} {} {}", r1, r2, s1);
}
```

#### 悬空引用 Dangling References

- 悬空指针（Dangling Pointer）：一个指针引用了内存中的某个地址，而这块内存可能已经释放并分配给其它人使用了。
- 在Rust里，编译器可保证引用永远都不是悬空引用：
  - 如果你引用了某些数据，编译器将保证在引用离开作用域之前数据不会离开作用域

#### 引用的规则

- 在任何给定的时刻，只能满足下列条件之一：
  - 一个可变的引用
  - 任意数量不可变的引用
- 引用必须一直有效

### 4.3 切片

- Rust的另外一种不持有所有权的数据类型：切片（slice）
- 一道题，编写一个函数：
  - 它接收字符串作为参数
  - 返回它在这个字符串里找到的第一个单词
  - 如果函数没找到任何空格，那么整个字符串就被返回
- 尝试解答

```rust
fn main() {
    let mut s  = String::from("Hello world");
    let word_index = first_world(&s);

    println!("{}", word_index);
}

fn first_world(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }
    s.len()
}

```

#### 字符串切片

- 字符串切片是指向字符串中一部分内容的引用
- 形式：[开始索引..结束索引]
  - 开始索引就是切片起始位置的索引值
  - 结束索引是切片终止位置的下一个索引值

```rust
fn main() {
  let s = String::from("Hello world");
  
  let hello = &s[0..5];
  let world = &s[6..11];
  
  println!("{}, {}", hello, world);
}
```

#### 注意

- 字符串切片的范围索引必须发生在有效的UTF-8字符边界内
- 如果尝试从一个多字节的字符中创建字符串切片，程序会报错并退出

#### 使用字符串切片重写例子

```rust
fn main() {
    let s  = String::from("Hello world");
    let word_index = first_world(&s);

    println!("{}", word_index);
}

fn first_world(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[..i];
        }
    }
    &s[..]
}

```

#### 字符串字面值是切片

- 字符串字面值被直接存储在二进制程序中
- let s = "Hello, World!";

```rust
fn main() {
  let s = "Hello world";
  
  println!("{}", s)
}
```

- 变量s的类型是&str，它是一个指向二进制程序特定位置的切片
  - &str 是不可变引用，所以字符串字面值也是不可变的

#### 将字符串切片作为参数传递

- `fn first_word(s: &String) -> &str {`
- 有经验的Rust开发者会采用&str作为参数类型，因为这样就可以同时接收String和&str类型的参数了：
- `fn first_word(s: &str) -> &str {`
  - 使用字符串切片，直接调用该函数
  - 使用String，可以创建一个完整的String切片来调用该函数
- 定义函数时使用字符串切片来代替字符串引用会使我们的API更加通用，且不会损失任何功能
- 字符串字面值就是字符串切片

```rust
fn main() {
    let my_string  = String::from("Hello world");
    let word_index = first_world(&my_string[..]);

    println!("{}", word_index);

    let my_string_literal = "hello world";
    let word_literal = first_world(my_string_literal);

    println!("word_literal: {}", word_literal);
}

fn first_world(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[..i];
        }
    }
    &s[..]
}

```

#### 其他类型的切片

```rust
fn main() {
  let a = [1, 2, 3, 4, 5];
  let slice = &a[1..3];
}
```
