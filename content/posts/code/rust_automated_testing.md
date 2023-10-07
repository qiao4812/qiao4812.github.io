---
title: "Rust编程语言入门之编写自动化测试"
date: 2023-04-02T11:44:10+08:00
draft: false
tags: ["Rust"]
categories: ["Rust"]
---

# 编写自动化测试

## 一、编写和运行测试

### 测试（函数）

- 测试：
  - 函数
  - 验证非测试代码的功能是否和预期一致
- 测试函数体（通常）执行的3个操作：
  - 准备数据/状态
  - 运行被测试的代码
  - 断言（Assert）结果

### 解剖测试函数

- 测试函数需要使用 test 属性（attribute）进行标注
  - Attribute就是一段Rust代码的元数据
  - 在函数上加 #[test]，可把函数变成测试函数

### 运行测试

- 使用 cargo test 命令运行所有测试函数
  - Rust会构建一个 Test Runner 可执行文件
  - 它会运行标注了 test 的函数，并报告其运行是否成功

- 当使用 cargo 创建 library 项目的时候，会生成一个 test module，里面有一个test 函数
  - 你可以添加任意数量的 test module 或 函数

```bash
~/rust
➜ cargo new adder --lib
     Created library `adder` package

~/rust
➜ cd adder

adder on  master [?] via 🦀 1.67.1
➜ code .

adder on  master [?] via 🦀 1.67.1 took 2.2s
➜


~/rust
➜ cargo new adder --lib
     Created library `adder` package

~/rust
➜ cd adder

adder on  master [?] via 🦀 1.67.1
➜ code .

adder on  master [?] via 🦀 1.67.1 took 2.2s
➜

```

lib.rs 文件

```rust
pub fn add(left: usize, right: usize) -> usize {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }
}

```

### 测试失败

- 测试函数 panic 就表示失败
- 每个测试运行在一个新线程
- 当主线程看见某个测试线程挂掉了，那个测试标记为失败了。

```rust
pub fn add(left: usize, right: usize) -> usize {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn exploration() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail")
    }
}

```

运行

```bash
adder on  master [?] is 📦 0.1.0 via 🦀 1.67.1 took 3.0s 
➜ cargo test
   Compiling adder v0.1.0 (/Users/qiaopengjun/rust/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.16s
     Running unittests src/lib.rs (target/debug/deps/adder-6058f7b13179a51e)

running 2 tests
test tests::exploration ... ok
test tests::another ... FAILED

failures:

---- tests::another stdout ----
thread 'tests::another' panicked at 'Make this test fail', src/lib.rs:17:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`

adder on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
```

## 二、断言（Assert）

### 使用 Assert! 宏检查测试结果

- assert! 宏，来自标准库，用来确定某个状态是否为true
  - true：测试通过
  - false：调用 panic!，测试失败

```rust
#[derive(Debug)]
pub struct Rectangle {
  length: u32,
  width: u32,
}

impl Rectangle {
  pub fn can_hold(&self, other: &Rectangle) -> bool {
    self.length > other.length && self.width > other.width
  }
}

#[cfg(test)]
mod tests {
  use super::*
  
  #[test]
  fn larger_can_hold_smaller() {
    let larger = Rectangle {
      length: 8,
      width: 7,
    };
    let smaller = Rectangle {
      length: 5,
      width: 1,
    };
    assert!(larger.can_hold(&smaller));
  }
  
  #[test]
  fn samller_cannot_hold_larger() {
    let larger = Rectangle {
      length: 8,
      width: 7,
    };
    let smaller = Rectangle {
      length: 5,
      width: 1,
    };
    assert!(!smaller.can_hold(&larger));
  }
}
```

### 使用 assert_eq! 和 assert_ne! 测试相等性

- 都来自标准库
- 判断两个参数是否相等或不等
- 实际上，它们使用的就是 == 和 !== 运算符
- 断言失败，自动打印出两个参数的值
  - 使用 debug 格式打印参数
  - 要求参数实现了 PartialEq 和 Debug Traits （所有的基本类型和标准库里大部分类型都实现了）

```rust
pub fn add_two(a: i32) -> i32 {
  a + 2
}

#[cfg(test)]
mod tests {
  use super::*;
  
  #[test]
  fn it_adds_two() {
    // assert_eq!(4, add_two(2));
    assert_ne!(5, add_two(2));
  }
}
```

## 三、自定义错误消息

### 添加自定义错误信息

- 可以向 assert!、assert_eq!、assert_ne! 添加可选的自定义消息
  - 这些自定义消息和失败消息都会打印出来
  - assert!：第 1 参数必填，自定义消息作为第2个参数。
  - assert_eq! 和 assert_ne!：前2个参数必填，自定义消息作为第 3 个参数。
  - 自定义消息参数会被传递给 format! 宏，可以使用 {} 占位符

```rust
pub fn greeting(name: &str) -> String {
  //format!("Hello {}!", name)
  format!("Hello!")
}

#[cfg(test)]
mod tests {
  use super::*;
  
  #[test]
  fn greetings_contain_name() {
    let result = greeting("Carol");
    // assert!(result.contains("Carol"));
    assert!(
      result.contains("Carol"),
      "Greeting didn't contain name, value was '{}'", result
    );
  }
}
```

## 四、用 should_panic 检查恐慌

### 验证错误处理的情况

- 测试除了验证代码的返回值是否正确，还需验证代码是否如预期的处理了发生错误的情况
- 可验证代码在特定情况下是否发生了 panic
- should_panic 属性（attribute）：
  - 函数 panic：测试通过
  - 函数 没有 panic：测试失败

```rust
pub struct Guess {
  value: u32,
}

impl Guess {
  pub fn new(value: u32) -> Guess {
    if value < 1 || value > 100 {
      panic!("Guess value must be between 1 and 100, got {}.", value)
    }
    
    Guess {value}
  }
}

#[cfg(test)]
mod tests {
  use super::*;
  
  #[test]
  #[should_panic]
  fn greater_than_100() {
    Guess::new(200);
  }
}
```

### 让 should_panic 更精确

- 为 should_panic 属性添加一个可选的 expected 参数：
  - 将检查失败消息中是否包含所指定的文字

```rust
pub struct Guess {
  value: u32,
}

impl Guess {
  pub fn new(value: u32) -> Guess {
    if value < 1 {
      panic!("Guess value must be greater than or equal to 1, got {}.", value)
    } else if value > 100 {
      panic!("Guess value must be less than or equal to 100, got {}.", value)
    }
    
    Guess {value}
  }
}

#[cfg(test)]
mod tests {
  use super::*;
  
  #[test]
  #[should_panic(expected = "Guess value must be less than or equal to 100")]
  fn greater_than_100() {
    Guess::new(200);
  }
}
```

## 五、在测试中使用 Result<T, E>

### 在测试中使用 Result<T, E>

- 无需 panic，可使用 Result<T, E> 作为返回类型编写测试：
  - 返回 Ok：测试通过
  - 返回 Err：测试失败

```rust
#[cfg(test)]
mod tests {
  #[test]
  fn it_works() -> Result<(), String> {
    if 2 + 2 == 4 {
      Ok(())
    } else {
      Err(String::from("two plus two does not equal four"))
    }
  }
}
```

- 注意：不要在使用 Result<T, E> 编写的测试上标注 #[should_panic]

## 六、控制测试运行：并行和连续执行测试

### 控制测试如何运行

- 改变 cargo test 的行为：添加命令行参数
- 默认行为：
  - 并行运行
  - 所有测试
  - 捕获（不显示）所有输出，使读取与测试结果相关的输出更容易。
- 命令行参数：
  - 针对 cargo test 的参数：紧跟 cargo test 后
  - 针对测试可执行程序：放在 -- 之后
  - cargo test --help
  - cargo -- --help

### 并行运行测试

- 运行多个测试：默认使用多个线程并行运行
  - 运行快
- 确保测试之间：
  - 不会互相依赖
  - 不依赖于某个共享状态（环境、工作目录、环境变量等待）

### --test-threads 参数

- 传递给 二进制文件
- 不想以并行方式运行测试，或想对线程数进行细粒度控制
- 可以使用 --test-threads 参数，后边跟着线程的数量
- 例如：cargo test -- --test-threads=1

### 显示函数输出

- 默认，如测试通过，Rust的test库会捕获所有打印到标准输出的内容
- 例如，如果被测试代码中用到了 println!：
  - 如果测试通过：不会再终端看到 println! 打印的内容
  - 如果测试失败：会看到 println! 打印的内容和失败信息

```rust
fn prints_and_returns_10(a: i32) -> i32 {
  println!("I got the value {}", a);
  10
}

#[cfg(test)]
mod tests {
  use super::*;
  
  #[test]
  fn this_test_will_pass() {
    let value = prints_and_returns_10(4);
    assert_eq!(10, value);
  }
  
  #[test]
  fn this_test_will_fail() {
    let value = prints_and_returns_10(8);
    assert_eq!(5, value);
  }
}
```

- 如果想在成功的测试中看到打印的内容： --show-output

## 七、控制测试运行：按名称运行测试

- 选择运行的测试：将测试的名称（一个或多个）作为 cargo test 的参数

```rust
pub fn add_two(a: i32) -> i32 {
  a + 2
}

#[cfg(test)]
mod tests {
  use super::*;
  
  #[test]
  fn add_two_and_two() {
    assert_eq!(4, add_two(2));
  }
  
  #[test]
  fn add_three_and_two() {
    assert_eq!(5, add_two(3));
  }
  
  #[test]
  fn one_hundred() {
    assert_eq!(102, add_two(100));
  }
}
```

- 运行单个测试：指定测试名

```bash
cargo test one_hundred
```

- 运行多个测试：指定测试名的一部分（模块名也可以）
- `cargo test add`

## 八、控制测试运行：忽略测试

### 忽略某些测试，运行剩余测试

- ignore 属性（attribute）

```rust
#[cfg(test)]
mod tests {
  #[test]
  fn it_works() {
    assert_eq!(4, 2 + 2);
  }
  
  #[test]
  #[ignore]
  fn expensive_test() {
    assert_eq!(5, 1 + 1 + 1 + 1 + 1);
  }
}
```

- 运行被忽略（ignore）的测试：
  - `cargo test -- --ignored`

## 九、测试组织：单元测试

### 测试的分类

- Rust对测试的分类：
  - 单元测试
  - 集成测试
- 单元测试：
  - 小、专注
  - 一次对一个模块进行隔离的测试
  - 可测试 private 接口
- 集成测试：
  - 在库外部。和其他外部代码一样使用你的代码
  - 只能使用 public 接口
  - 可能在每个测试中使用到多个模块

单元测试

### #[cfg(test)] 标注

- tests 模块上的 #[cfg(test)] 标注：
  - 只有运行 cargo test 才编译和运行代码
  - 运行 cargo build 则不会
- 集成测试在不同的目录，它不需要 #[cfg(test)] 标注
- cfg：configuration （配置）
  - 告诉Rust下面的条目只有在指定的配置选项下才被包含
  - 配置选项test：由Rust提供，用来编译和运行测试
    - 只有 cargo test 才会编译代码，包括模块中的 helper 函数 和 #[test] 标注的函数

```rust
#[cfg(test)]
mod tests {
  #[test]
  fn it_works() {
    assert_eq!(4, 2 + 2);
  }
}
```

### 测试私有函数

- Rust允许测试私有函数

```rust
pub fn add_two(a: i32) -> i32 {
  internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
  a + b
}

#[cfg(test)]
mod tests {
  use super::*;
  
  #[test]
  fn it_works() {
    assert_eq!(4, internal_adder(2, 2));
  }
}
```

## 十、集成测试

- 在Rust里，集成测试完全位于被测试库的外部
- 目的：是测试被测试库的多个部分是否能正确的一起工作
- 集成测试的覆盖率很重要

### Tests 目录

- 创建集成测试：tests 目录
- tests 目录下的每个测试文件都是单独的一个 crate
  - 需要将被测试库导入
- 无需标注 #[cfg(test)]，tests 目录被特殊对待
  - 只有 cargo test ， 才会编译 tests 目录下的文件

src/lib.rs 文件

```rust
pub fn add(left: usize, right: usize) -> usize {
    left + right
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn exploration() {
        let result = add(2, 2);
        assert_eq!(result, 4);
    }

    // #[test]
    // fn another() {
    //     panic!("Make this test fail")
    // }
}

```

tests/integration_test.rs 文件

```rust
use adder;

#[test]
fn it_add() {
    assert_eq!(5, adder::add(2, 3));
}

```

运行

```bash
adder on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ cargo test
   Compiling adder v0.1.0 (/Users/qiaopengjun/rust/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.11s
     Running unittests src/lib.rs (target/debug/deps/adder-6058f7b13179a51e)

running 1 test
test tests::exploration ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/integration_test.rs (target/debug/deps/integration_test-461b916f2718e782)

running 1 test
test it_add ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


adder on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
```

### 运行指定的集成测试

- 运行一个特定的集成测试：cargo test 函数名
- 运行某个测试文件内的所有测试： cargo test --test 文件名

tests/another_integration_tests.rs  文件

```rust
use adder;

#[test]
fn it_adds2() {
    assert_eq!(7, adder::add(3,4));
}

```

运行

```bash
adder on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ cargo test --test integration_test
    Finished test [unoptimized + debuginfo] target(s) in 0.00s
     Running tests/integration_test.rs (target/debug/deps/integration_test-461b916f2718e782)

running 1 test
test it_add ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


adder on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ cargo test --test another_integration_tests
   Compiling adder v0.1.0 (/Users/qiaopengjun/rust/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.11s
     Running tests/another_integration_tests.rs (target/debug/deps/another_integration_tests-0a89cbf68d5b375f)

running 1 test
test it_adds2 ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


adder on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
```

### 集成测试中的子模块

- tests 目录下每个文件被编译成单独的 crate
  - 这些文件不共享行为（与 src 下的文件规则不同）

```bash
adder on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ tree
.
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
├── target
│   ├── CACHEDIR.TAG
│   ├── debug
│   └── tmp
└── tests
    ├── another_integration_tests.rs
    ├── common
    │   └── mod.rs
    └── integration_test.rs

27 directories, 205 files
```

tests/common/mod.rs 文件

```rust
pub fn setup() {}

```

tests/another_integration_tests.rs 文件

```rust
use adder;

mod common;

#[test]
fn it_adds2() {
    common::setup();
    assert_eq!(7, adder::add(3,4));
}

```

tests/integration_test.rs 文件

```rust
use adder;

mod common;

#[test]
fn it_add() {
    common::setup();
    assert_eq!(5, adder::add(2, 3));
}

```

### 针对 binary crate 的集成测试

- 如果项目是 binary Crate，只含有 src/main.rs 没有 src/lib.rs：
  - 不能在 tests 目录下创建集成测试
  - 无法把 main.rs 的函数导入作用域
- 只有library crate 才能暴露函数给其它 crate 用
- binary crate 意味着独立运行
