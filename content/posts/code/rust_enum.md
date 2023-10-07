---
title: "Rust编程语言入门之枚举与模式匹配"
date: 2023-03-11T21:43:17+08:00
draft: false
tags: ["Rust"]
categories: ["Rust"]
---

# Rust编程语言入门之枚举与模式匹配

## 一、定义枚举

### 枚举

- 枚举允许我们列举所有可能的值来定义一个类型

### 定义枚举

- IP地址：IPv4、IPv6

```rust
enum IpAddrKing {
  V4,
  V6,
}
```

### 枚举值

- 例子：
- `let four = IpAddrKind::V4;`
- `let six = IpAddrKind::V6;`
- 枚举的变体都位于标识符的命名空间下，使用两个冒号:: 进行分隔

```rust
enum IpAddrKind {
    V4,
    V6,
}

fn main() {
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;

    route(four);
    route(six);
    route(IpAddrKind::V6);
}

fn route(ip_kind: IpAddrKind) {}

```

### 将数据附加到枚举的变体中

```rust
enum IpAddr {
  V4(String),
  V6(String),
}
```

未使用前：

```rust
enum IpAddrKind {
    V4,
    V6,
}

struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

fn main() {
    let home = IpAddr {
        kind: IpAddrKind::V4,
        address: String::from("127.0.0.1"),
    };

    let loopback = IpAddr {
        kind: IpAddrKind::V6,
        address: String::from("::1"),
    };
}
```

- 优点：
  - 不需要额外使用 struct
  - 每个变体可以拥有不同的类型以及关联的数据量
- 例如：

```rust
enum IpAddr {
  V4(u8, u8, u8, u8),
  V6(String),
}
```

使用之后：

```rust
enum IpAddrKind {
    V4(u8, u8, u8, u8),
    V6(String),
}

fn main() {
    let home = IpAddrKind::V4(127, 0, 0, 1);
    let loopback = IpAddrKind::V6(String::from("::1"));
}
```

### 标准库中的 IpAddr

```rust
struct Ipv4Addr {
  // --snip--
}

struct Ipv6Addr {
  // --snip--
}

enum IpAddr {
  V4(Ipv4Addr),
  V6(Ipv6Addr),
}
```

例子：

```rust
enum Message {
    Quit,
    Move {x: i32, y: i32},
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let q = Message::Quit;
    let m = Message::Move { x: 12, y: 24 };
    let w = Message::Write(String::from("hello"));
    let c = Message::ChangeColor(0, 255, 255);
}
```

### 为枚举定义方法

- 使用 impl 关键字

```rust
enum Message {
    Quit,
    Move {x: i32, y: i32},
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {}
}

fn main() {
    let q = Message::Quit;
    let m = Message::Move { x: 12, y: 24 };
    let w = Message::Write(String::from("hello"));
    let c = Message::ChangeColor(0, 255, 255);

    m.call();
}
```

## 二、Option 枚举

### Option 枚举

- 定义于标准库中
- 在 Prelude （预导入模块）中
- 描述了：某个值可能存在（某种类型）或不存在的情况

### Rust 没有 Null

- 其它语言中：
  - NULL 是一个值，它表示“没有值”
  - 一个变量可以处于两种状态：空值（NULL）、非空
- NULL 引用：Billion Dollar Mistake
- NULL 的问题在于：当你尝试像使用非NULL值那样使用NULL值的时候，就会引起某种错误
- NULL的概念还是有用的：因某种原因而变为无效或缺失的值

### Rust中类似NULL概念的枚举- Option<T>

- 标准库中的定义：
- `enum Option<T> {Some(T), None,}`
- 它包含在Prelude（预导入模块）中。可直接使用：
  - `Option<T>`
  - `Some(T)`
  - `None`
- 例子：

```rust
fn main() {
    let some_number = Some(5);
    let some_string = Some("A String");

    let absent_number: Option<i32> = None;
}
```

### `Option<T>`比 Null 好在哪？

- `Option<T>`和T 是不同的类型，不可以把 `Option<T>` 直接当成 T
- 若想使用 `Option<T>` 中的 T，必须将它转换为 T
- 而在 C# 中：
  - `string a = null;`
  - `string b = a + "12345";`

```rust
fn main() {
  let x: i8 = 5;
  let y: Option<i8> = Some(5);
  
  let sum = x + y;  // 报错 需要转换
}
```

## 三、控制流运算法 - match

### 强大的控制流运算符 - match

- 允许一个值与一系列模式进行匹配，并执行匹配的模式对应的代码
- 模式可以是字面值、变量名、通配符...
- 例子：

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}

fn main() {
    println!("Hello, world!");
}

```

### 绑定值得模式

- 匹配的分支可以绑定到被匹配对象的部分值
  - 因此，可以从enum 变体中提取值

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from: {:?}!", state);
            25
        }
    }
}

fn main() {
    let c = Coin::Quarter(UsState::Alaska);
    println!("{}", value_in_cents(c));
}

```

### 匹配 `Option<T>`

```rust
fn main() {
  let five = Some(5);
  let six = plus_one(five);
  let none = plus_one(None);
}

fn plus_one(x: Option<i32>) -> Option<i32> {
  match x {
    None => None,
    Some(i) => Some(i + 1),
  }
}
```

### Match 匹配必须穷举所有的可能

- _ 通配符：替代其余没列出的值
- 例子：

```rust
fn main() {
  let v = 0u8;
  match v {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
  }
}
```

## 四、if let

- 处理只关心一种匹配而忽略其它匹配的情况

```rust
fn main() {
  let v = Some(0u8);
  match v {
    Some(3) => println!("three"),
    _ => (),
  }
  
  if let Some(3) = v {
    println!("three");
  }
}
```

- 更少的代码，更少的缩进，更少的模版代码
- 放弃了穷举的可能
- 可以把if let 看作是 match 的语法糖
- 搭配 else

```rust
fn main() {
  let v = Some(0u8);
  match v {
    Some(3) => println!("three"),
    _ => println!("others"),
  }
  
  if let Some(3) = v {
    println!("three");
  } else {
    println!("others");
  }
}
```
