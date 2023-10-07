---
title: "Rust编程语言入门之模式匹配"
date: 2023-04-20T22:06:42+08:00
draft: false
tags: ["Rust"]
categories: ["Rust"]
---

# 模式匹配

### 模式

- 模式是Rust中的一种特殊语法，用于匹配复杂和简单类型的结构
- 将模式与匹配表达式和其他构造结合使用，可以更好地控制程序的控制流
- 模式由以下元素（的一些组合）组成：
  - 字面值
  - 解构的数组、enum、struct 和 tuple
  - 变量
  - 通配符
  - 占位符
- 想要使用模式，需要将其与某个值进行比较：
  - 如果模式匹配，就可以在代码中使用这个值的相应部分

## 一、用到模式（匹配）的地方

### match 的 Arm

```rust
match VALUE {
  PATTERN => EXPRESSION,
  PATTERN => EXPRESSION,
  PATTERN => EXPRESSION,
}
```

- match 表达式的要求：
  - 详尽（包含所有的可能性）
- 一个特殊的模式：_（下划线）：
  - 它会匹配任何东西
  - 不会绑定到变量
  - 通常用于 match 的最后一个 arm；或用于忽略某些值。

### 条件 if let 表达式

- if let 表达式主要是作为一种简短的方式来等价的代替只有一个匹配项的 match
- if let 可选的可以拥有 else，包括：
  - else if
  - else if let
- 但，if let 不会检查穷尽性

```rust
fn main() {
  let favorite_color: Option<&str> = None;
  let is_tuesday = false;
  let age: Result<u8, _> = "34".parse();
  
  if let Some(color) = favorite_color {
    println!("Using your favorite color, {}, as the background", color);
  } else if if_tuesday {
    println!("Tuesday is green day!");
  } else if let Ok(age) = age {
    if age > 30 {
      println!("Using purple as the background color");
    } else {
      println!("Using orange as the background color");
    }
  } else {
    println!("Using blue as the background color");
  }
}
```

### While let 条件循环

- 只要模式继续满足匹配的条件，那它允许 while 循环一直运行

```rust
fn main() {
  let mut stack = Vec::new();
  
  stack.push(1);
  stack.push(2);
  stack.push(3);
  
  while let Some(top) = stack.pop() {
    println!("{}", top);
  }
}
```

### for 循环

- for 循环是Rust 中最常见的循环
- for 循环中，模式就是紧随 for 关键字后的值

```rust
fn main() {
  let v = vec!['a', 'b', 'c'];
  
  for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value , index);
  }
}
```

### Let 语句

- let 语句也是模式
- let PATTERN = EXPRESSION

```rust
fn main() {
  let a = 5;
  
  let (x, y, z) = (1, 2, 3);
  
  let (q, w) = (4, 5, 6); // 报错 类型不匹配 3 2
}
```

### 函数参数

- 函数参数也可以是模式

```rust
fn foo(x: i32) {
  // code goes here
}

fn print_coordinates(&(x, y): &(i32, i32)) {
  println!("Current location: ({}, {})", x, y);
}

fn main() {
  let point = (3, 5);
  print_coordinates(&point);
}
```

## 二、可辩驳性：模式是否会无法匹配

### 模式的两种形式

- 模式有两种形式：可辨驳的、无可辩驳的
- 能匹配任何可能传递的值的模式：无可辩驳的
  - 例如：`let x = 5;`
- 对某些可能得值，无法进行匹配的模式：可辩驳的
  - 例如：`if let Some(x) = a_value`
- 函数参数、let 语句、for 循环只接受无可辩驳的模式
- if let 和 while let 接受可辨驳和无可辩驳的模式

```rust
fn main() {
  let a: Option<i32> = Some(5);
  let Some(x) = a: // 报错 None
  if let Some(x) = a {}
  if let x = 5 {} // 警告
}
```

## 三、模式语法

### 匹配字面值

- 模式可直接匹配字面值

```rust
fn main() {
  let x = 1;
  
  match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
  }
}
```

### 匹配命名变量

- 命名的变量是可匹配任何值的无可辩驳模式

```rust
fn main() {
  let x = Some(5);
  let y = 10;
  
  match x {
    Some(50) => println!("Got 50"),
    Some(y) => println!("Matched, y = {:?}", y),
    _ => println!("Default case, x = {:?}", x),
  }
  
  println!("at the end: x = {:?}, y = {:?}", x, y);
}
```

### 多重模式

- 在match 表达式中，使用 | 语法（就是或的意思），可以匹配多种模式

```rust
fn main() {
  let x = 1;
  
  match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
  }
}
```

### 使用 ..= 来匹配某个范围的值

```rust
fn main() {
  let x = 5;
  match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
  }
  
  let x = 'c';
  match x {
    'a' ..='j' => println!("early ASCII letter"),
    'k' ..='z' => println!("late ASCII letter"),
    _ => println!("something else"),
  }
}
```

### 解构以分解值

- 可以使用模式来解构 struct、enum、tuple，从而引用这些类型值的不同部分

```rust
struct Point {
  x: i32,
  y: i32,
}

fn main() {
  let p = Point { x: 0, y: 7 };
  
  let Point { x: a, y: b } = p;
  assert_eq!(0, a);
  assert_eq!(7, b);
  
  let Point {x, y} = p;
  assert_eq!(0, x);
  assert_eq!(7, y);
  
  match p {
    Point {x, y: 0} => println!("On the x axis at {}", x),
    Point {x: 0, y} => println!("On the y axis at {}", y),
    Point {x, y} => println!("On neither axis: ({}, {})", x, y),
  }
}
```

### 解构 enum

```rust
enum Message {
  Quit,
  Move {x: i32, y: i32},
  Write(String),
  ChangeColor(i32, i32, i32),
}

fn main() {
  let msg = Message::ChangeColor(0, 160, 255);
  
  match msg {
    Message::Quit => {
      println!("The Quit variant has no data to destructure.")
    }
    Message::Move {x, y} => {
      println!("Move in the x direction {} and in the y direction {}", x, y);
    }
    Message::Write(text) => println!("Text message: {}", text),
    Message::ChangeColor(r, g, b) => {
      println!("Change the color to red {}, green {}, and blue {}", r, g, b);
    }
  }
}
```

### 解构嵌套的 struct 和 enum

```rust
enum Color {
  Rgb(i32, i32, i32),
  Hsv(i32, i32, i32),
}

enum Message {
  Quit,
  Move {x: i32, y: i32},
  Write(String),
  ChangeColor(Color),
}

fn main() {
  let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));
  
  match msg {
    Message::ChangeClolr(Color::Rgb(r, g, b)) => {
      println!("Change the color to red {}, green {}, and blur {}", r, g, b)
    }
    Message::ChangeColor(Color::Hsv(h, s, v)) => {
      println!("Change the color to hue {}, saturation {}, and value {}", h, s, v)
    }
    _ => (),
  }
}
```

### 解构 struct 和 tuple

```rust
struct Point {
  x: i32,
  y: i32,
}

fn main() {
  let ((feet, inches), Point {x, y}) = ((3, 10), Point {x: 3, y: -10});
}
```

### 在模式中忽略值

- 有几种方式可以在模式中忽略整个值或部分值：
  - _
  - _ 配合其它模式
  - 使用以 _ 开头的名称
  - .. （忽略值的剩余部分）

### 使用 _ 来忽略整个值

```rust
fn foo(_: i32, y: i32) {
  println!("This code only uses the y parameter: {}", y);
}

fn main() {
  foo(3, 4);
}
```

### 使用嵌套的 _ 来忽略值的一部分

```rust
fn main() {
  let mut setting_value = Some(5);
  let new_setting_value = Some(10);
  
  match (setting_value, new_setting_value) {
    (Some(_), Some(_)) => {
      println!("Can't overwrite an existing customized value");
    }
    _ => {
      setting_value = new_setting_value;
    }
  }
  
  println!("setting is {:?}", setting_value);
  
  
  let numbers = (2, 4, 6, 8, 16, 32);
  
  match numbers {
    (first, _, third, _, fifth) => {
      println!("Some numbers: {}, {}, {}", first, third, fifth)
    }
  }
}
```

### 通过使用 _ 开头命名来忽略未使用的变量

```rust
fn main() {
  let _x = 5;
  let y = 10;  // 创建未使用 警告
  
  let s = Some(String::from("Hello"));
  
  if let Some(_s) = s { // if let Some(_) = s {
    println!("found a string");
  }
  
  println!("{:?}", s); // 报错 
}
```

### 使用 .. 来忽略值的剩余部分

```rust
struct Point {
  x: i32,
  y: i32,
  z: i32,
}

fn main() {
  let origin = Point {x: 0, y: 0, z: 0};
  match origin {
    Point {x, ..} => println!("x is {}", x),
  }
  
  let numbers = (2, 4, 8, 16, 32);
  match numbers {
    (first, .., last) => {
      println!("Some numbers: {}, {}", first, last);
    }
  }
  
  match numbers {
    (.., second, ..) => {  // 报错
      println!("Some numbers: {}", second)
    },
  }
}
```

### 使用 match 守卫来提供额外的条件

- match 守卫就是 match arm 模式后额外的 if 条件，想要匹配该条件也必须满足
- match 守卫适用于比单独的模式更复杂的场景

例子一：

```rust
fn main() {
  let num = Some(4);
  
  match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
  }
}
```

例子二：

```rust
fn main() {
  let x = Some(5);
  let y = 10;
  
  match x {
    Some(50) => println!("Got 50"),
    Some(n) if n == y => println!("Matched, n = {:?}", n),
    _ => println!("Default case, x = {:?}", x),
  }
  
  println!("at the end: x = {:?}, y = {:?}", x, y);
}
```

例子三：

```rust
fn main() {
  let x = 4;
  let y = false;
  
  match x {
    4 | 5 | 6 if y => println!("yes"),
    _ => println!("no"),
  }
}
```

### @绑定

- @ 符号让我们可以创建一个变量，该变量可以在测试某个值是否与模式匹配的同时保存该值

```rust
enum Message {
  Hello {id: i32},
}

fn main() {
  let msg = Message::Hello {id: 5};
  
  match msg {
    Message::Hello {
      id: id_variable @ 3..=7,
    } => {
      println!("Found an id in range: {}", id_variable)
    }
    Message::Hello {id: 10..=12} => {
      println!("Found an id in another range")
    }
    Message::Hello {id} => {
      println!("Found some other id: {}", id)
    }
  }
}
```
