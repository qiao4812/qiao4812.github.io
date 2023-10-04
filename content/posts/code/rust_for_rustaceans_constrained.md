---
title: "Rust语言 - 接口设计的建议之受约束（Constrained）"
date: 2023-06-19T15:28:01+08:00
draft: true
tags: ["Rust"]
categories: ["Rust"]
---

# Rust语言 - 接口设计的建议之受约束（Constrained）

- [Rust API 指南 GitHub](https://github.com/rust-lang/api-guidelines)：https://github.com/rust-lang/api-guidelines
- [Rust API 指南 中文](https://rust-chinese-translation.github.io/api-guidelines/)：https://rust-chinese-translation.github.io/api-guidelines/
- [Rust API 指南](https://rust-lang.github.io/api-guidelines/)：https://rust-lang.github.io/api-guidelines/

## 受约束（Constrained）

### 接口的更改要三思

- 做出用户可见的更改，需三思而后行
  - 确保你做出的变化：
    - 不会破坏现有用户的代码
    - 这次变化应保留一段时间
  - 频繁的向后不兼容的更改（主版本增加），会引起用户不满

### 向后不兼容的修改

- 有些是显而易见的，有些则很微妙（与 Rust 工作方式相关）
  - 主要介绍微妙棘手的更改，以及如何为其制定计划
  - 有时需要在接口灵活性上做出权衡、妥协

### 类型修改

- 移除或重命名公共类型几乎肯定会破坏用户的代码
  - 解决：尽可能利用可见性修饰符
    - 例如 pub(crate)、pub(in path) ...
  - 公共类型越少，更改时（保证不会破坏现有代码）就越自由

例子一：

```rust
pub mod outer_mod {
  pub mod inner_mod {
    // This function is visible within `outer_mod`
    pub(in crate::outer_mod) fn outer_mod_visible_fn() {}
    
    // This function is visible to the entire crate
    pub(crate) fn crate_visible_fn() {}
    
    // This function is visible within `outer_mod`
    pub(super) fn super_mod_visible_fn() {
      // This function is visible since we're in the same `mod`
      inner_mod_visible_fn();
    }
    
    // This function is visible only within `inner_mod`,
    // which is the same as leaving it private.
    pub(self) fn inner_mod_visible_fn() {}
  }
  
  pub fn foo() {
    inner_mod::outer_mod_visible_fn();
    inner_mod::crate_visible_fn();
    inner_mod::super_mod_visible_fn();
    
    // This function is no longer visible since we're outside of `inner_mod`
    // Error! `inner_mod_visible_fn` is private
    // inner_mod::inner_mod_visible_fn();
  }
}

fn bar() {
  // This function is still visible since we're in the same crate
  outer_mod::inner_mod::crate_visible_fn();
  
  // This function is no longer visible since we're outside of `outer_mod`
  // Error! `super_mod_visible_fn` is private
  outer_mod::inner_mod::super_mod_visible_fn();
  
  // This function is no longer visible since we're outside of `outer_mod`
  // Error! `outer_mod_visible_fn` is private
  outer_mod::inner_mod::outer_mod_visible_fn();
  
  outer_mod::foo();
}

fn main() {
  bar()
}
```

- 用户代码不仅仅通过名称依赖于你的类型

例子二：

lib.rs

```rust
pub struct Unit;
```

main.rs 

```rust
fn main() {
  let u = constrained::Unit; // v0 库是 constrained
}
```

修改一

lib.rs

```rust
pub struct Unit {
  pub field: bool,
}
```

main.rs 

```rust
fn is_true(u: constrained::Unit) -> bool {
  matches!(u, constrained::Unit { field: true })
}

fn main() {
  let u = constrained::Unit; // v0  报错，因为添加字段之后 Unit struct 原有的构造方式不可用
}
```

修改二

lib.rs

```rust
pub struct Unit {
  local: i32, // 增加私有字段
}
```

main.rs 

```rust
fn main() {
  let u = constrained::Unit; // v0  报错，虽然字段看不见，但是编译器可以看到
}
```

- Rust 提供 `#[non_exhaustive]` 来缓解这些问题
  - `non_exhaustive`  表示类型或枚举在将来可能会添加更多字段或变体
    - 它可以应用于 struct、enums 和 enum variants。
  - 在其它 crate，使用 `non_exhaustive` 定义的类型，编译器会禁止：
    - 隐式构造，`lib::Unit { field1: true }`
    - 以及非穷尽模式匹配（即没有尾随 , .. 的模式）
  - 若接口稳定的话，尽量避免使用该注解

例子三：

lib.rs

```rust
#[non_exhaustive]
pub struct Config {
  pub window_width: u16,
  pub window_height: u16,
}

fn SomeFunction() {
  let config = Config {
    window_width: 640,
    window_height: 480,
  };
  
  // Non-exhaustive structs can be matched on exhaustively within the defining crate.
  if let Config {
    window_width,
    window_height,
  } = config
  {
    // ...
  }
}
```

main.rs 

```rust
use constrained::Config;

fn main() {
  // Not allowed.
  let config =  Config {  // 报错
    window_width: 640,
    window_height: 480,
  };
  
  if let Config {
    window_width,
    window_height,
    .. // This is the only difference. 必须加 .. 否则报错
  } = config
  {
    // ...
  }
}
```

### Trait 实现

- 一致性规则禁止把某个 Trait 为某类型进行多重实现
- 破坏性变更
  - 为现有 Trait 添加 Blanket Implementation 通常是破坏性变更（`impl <T> Foo for T`）
  - 为现有类型实现外部 Trait，或为外部类型实现现有 Trait
  - 移除 Trait 实现
    - 为新类型实现 Trait 就不是问题
- 为现有类型实现任何 Trait 都要小心

例子四：

lib.rs

```rust
pub struct Unit;

pub trait Fool {
  fn foo(&self);
}
```

main.rs 

```rust
use constrained::{Foo1, Unit};

trait Foo2 {
  fn foo(&self);
}

impl Foo2 for Unit {
  fn foo(&self) {
    println!("foo2");
  }
}

fn main() {
  Unit.foo()
}
```

修改一

lib.rs

```rust
pub struct Unit;

pub trait Fool {
  fn foo(&self);
}

// case 1: Add impl Foo1 for Unit in this crate
impl Foo1 for Unit {
  fn foo(&self) {
    println!("foo1");
  }
}
```

main.rs 

```rust
use constrained::{Foo1, Unit};

trait Foo2 {
  fn foo(&self);
}

impl Foo2 for Unit {
  fn foo(&self) {
    println!("foo2");
  }
}

fn main() {
  Unit.foo() // 报错
}
```

修改二

lib.rs

```rust
pub struct Unit;

pub trait Fool {
  fn foo(&self);
}

// case 2: Add a new public Trait
pub trait Bar1 {
  fn foo(&self); // with the same name
}

impl Bar1 for Unit {
  fn foo(&self) {
    println!("bar1");
  }
}
```

main.rs 

```rust
use constrained::{Foo1, Unit};

trait Foo2 {
  fn foo(&self);
}

impl Foo2 for Unit {
  fn foo(&self) {
    println!("foo2");
  }
}

fn main() {
  Unit.foo()  // 因为没有引入lib.rs中的Bar1，所以暂时没有报错
}
```

main.rs 

```rust
use constrained::*;

trait Foo2 {
  fn foo(&self);
}

impl Foo2 for Unit {
  fn foo(&self) {
    println!("foo2");
  }
}

fn main() {
  Unit.foo()  // 报错
}
```

- 大多数到现有 Trait 的更改也是破坏性更改
  - 改变方法签名
  - 添加新方法
    - 如果有默认实现倒是可以
- 封闭 Trait（Sealed Trait）：
  - 只能被其它 crate 用，不能实现
  - 防止 Trait 添加新方法时造成破坏性变更
  - 不是内建功能，有多种实现方法
- Sealed Trait 常用于派生 Trait
  - 为实现特定其它 Trait 的类型提供 blanket implementation 的 Trait
- 封闭 Trait（Sealed Trait）：
  - 只有在外部 crate 不该实现你的 Trait 时，才使用 Sealed Trait
  - 严重限制 Trait 的可用性
    - 下游 crate 无法为其自己类型实现该 Trait
  - 可使用 Sealed Trait 来限制可用作类型参数的类型
    - 例：将 Rocket 示例中的 Stage 类型限制为仅允许 Grounded 和 Launched 类型

例子五：

lib.rs

```rust
use std::fmt::{Debug, Display};

mod sealed {
  use std::fmt::{Debug, Display};
  
  pug trait Sealed {}
  impl<T> Sealed for T where T: Debug + Display {}
}

pub trait CanUseCannotImplement: sealed::Sealed {
  // ..
}

impl<T> CanUseCannotImplement for T where T: Debug + Display {}
```

main.rs

```rust
use std::fmt::{Debug, Display};

use constrained::CanUseCannotImplement;
pub struct Bar {}

impl Debug for Bar {
  fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
    Ok(())
  }
}

impl Display for Bar {
  fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
    Ok(())
  }
}

// impl CanUseCannotImplement for Bar {}  // 报错 因为在 lib.rs 中已经实现好了，不能再实现
// Conflicting implementation,
// The trait `CanUseCannotImplement` has been already implemented
// for the types that satisfy the bounds specified by the sealed trait which are `Debug + Display`

pub struct Foo {}

impl CanUseCannotImplement for Foo {} // 报错 没有实现 Debug 和 Display

fn main() {}
```

### 隐藏的契约

- 有时，你对代码的某一部分所做的更改会以微妙的方式影响到接口其他地方的契约。这种情况主要发生在：
  - 重新导出（re-exports）
  - 自动 Traits （auto-traits）

### 隐藏的契约 - 重新导出（Re-Exports）

- 如果你的接口的某部分暴露了外部类型，那么外部类型的任何更改也将成为你接口的变更
  - 最好用新类型模式（newtype pattern）包裹外部类型，仅仅暴露外部类型中你认为有用的部分

例子六：

lib.rs

```rust
// 你的 crate,叫 bestiter
pub fn iter<T>() -> itercrate::Empty<T> { .. }

// 依赖的外部 crate，叫 itercrate (v1.0)，提供了 Empty<T> 类型

// 用户的 crate 中
struct EmptyIterator { it: itercrate::Empty<()> }

EmptyIterator { it: bestiter::iter() }

// ---------------------------------------------------------------
// 你的 crate, 叫 bestiter
pub fn iter<T>() -> itercrate::Empty<T> { .. }

// 依赖的外部crate，叫 itercrate，提供了 Empty<T> 类型
// 依赖的版本改为 v2.0，别处没有更改
// 编译器认为：itercrate1.0::Empty 和 itercrate2.0::Empty 是不同的类型
// 导致破坏性变更

// 用户的 crate 中
struct EmptyIterator { it: itercrate::Empty<()> }
```

### 隐藏的契约 - 自动 Trait（Auto-Traits）

- 有些 Trait 根据类型的内容，会对其进行实现
  - 根据它们的特性，它们为接口中几乎每种类型都添加一个隐藏的承诺
  - Send、Sync
  - Unpin、Sized、UnwindSafe 也存在类似问题
  - 这些特性会传播，无论是具体类型，还是 impl Trait 等类型擦除情况
- 这些 Trait 的实现通常是编译器自动添加的
  - 如果情况不适用，则不会自动添加
- 例如：
  - 类型 A 包含私有类型 B，默认 A 和 B 都是 Send 的
  - 如果修改 B，让 B 不再是 Send 的，那么 A 也变成不 Send 的了
  - 破坏性变化
- 这类变化难以追踪和发现
  - 包含一些简单的测试，检查你所有的类型都实现了相关的 Traits

例子七：

```rust
fn is_normal<T: Sized + Send + Sync + Unpin>() {}

#[test]
fn normal_types() {
  is_normal::<MyType>();
}
```

### 设计 Rust 接口的总结

- 不让人感到意外、灵活的、显而易见的和受限制的

