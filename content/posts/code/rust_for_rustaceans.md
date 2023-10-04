---
title: "Rust语言 - 接口设计的建议之不意外（unsurprising）"
date: 2023-06-07T22:49:25+08:00
draft: true
tags: ["Rust"]
categories: ["Rust"]
---

# Rust - 接口设计建议之不意外（unsurprising）

书：Rust for Rustaceans

## Rust接口设计的原则（建议）

- 四个原则：
  - 不意外（unsurprising）
  - 灵活（flexible）
  - 显而易见（obvious）
  - 受约束（constrained）
- Rust API 指南 GitHub：<https://github.com/rust-lang/api-guidelines>
- Rust API 指南 中文：<https://rust-chinese-translation.github.io/api-guidelines/>
- Rust API 指南：<https://rust-lang.github.io/api-guidelines/>

## 不意外（unsurprising）

- 最少意外原则：
  - 接口应尽可能直观（可预测，用户能猜对）
  - 至少应该不让人感到惊奇
- 核心思想：
  - 贴近用户已经知道的东西（不必重学概念）
- 让接口可预测：
  - 命名
  - 实现常用的 Traits
  - 人体工程学（Ergonomic）Traits
  - 包装类型（Wrapper Type）

## 命名实践

- 接口的名称，应符合惯例，便于推断其功能
  - 例：
    - 方法 iter，大概率应将 &self 作为参数，并应该返回一个迭代器（iterator）
    - 叫做 into_inner 的方法，大概率应将 self 作为参数，并返回某个包装的类型
    - 叫做 SomethingError 的类型，应实现 std::error::Error，并出现在各类 Result 里
- 将通用/常用的名称依然用于相同的目的，让用户好猜、好理解
- 推论：同名的事物应该以相同的方式工作
  - 否则，用户大概率会写出错误的代码
- [遵循 `as_`, `to_`, `into_` 规范 用以特定类型转换](https://rust-chinese-translation.github.io/api-guidelines/naming.html#遵循-as_-to_-into_-规范-用以特定类型转换)



| 名称前缀 | 内存代价   | 所有权                                                       |
| -------- | ---------- | ------------------------------------------------------------ |
| `as_`    | 无代价     | borrowed -> borrowed                                         |
| `to_`    | 代价昂贵   | borrowed -> borrowed borrowed -> owned (非 Copy 类型) owned -> owned (Copy 类型) |
| `into_`  | 视情况而定 | owned -> owned (非 Copy 类型)                                |

## 实现常用的 Trait

- 用户通常会假设接口中的一切均可“正常工作”，例：
  - 使用 `{:?}` 打印任何类型
  - 可发送任何东西到另外的线程
  - 期望每个类型都是 Clone 的
- 建议积极实现大部分标准 Trait，即使不立即用到
- 用户无法为外部类型实现外部的 Trait
  - 即使能包装你的接口类型，也难以写出合理实现

Rust 的 trait 系统坚持 [孤儿原则](https://doc.rust-lang.org/book/ch10-02-traits.html?highlight=orphan#implementing-a-trait-on-a-type) ：大致说的是， 每个 `impl` 块必须

1. 要么存在于定义 trait 的 crate 中，
2. 要么存在于给类型实现 trait 的 crate 中。

所以，定义新类型的 crates 应该尽早实现所有合适的、常见的 traits 。

`std` 中可给类型实现的、最重要的、常见的 traits 有：

- [`Copy`](https://doc.rust-lang.org/std/marker/trait.Copy.html)
- [`Clone`](https://doc.rust-lang.org/std/clone/trait.Clone.html)
- [`Eq`](https://doc.rust-lang.org/std/cmp/trait.Eq.html)
- [`PartialEq`](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html)
- [`Ord`](https://doc.rust-lang.org/std/cmp/trait.Ord.html)
- [`PartialOrd`](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html)
- [`Hash`](https://doc.rust-lang.org/std/hash/trait.Hash.html)
- [`Debug`](https://doc.rust-lang.org/std/fmt/trait.Debug.html)
- [`Display`](https://doc.rust-lang.org/std/fmt/trait.Display.html)
- [`Default`](https://doc.rust-lang.org/std/default/trait.Default.html)

给类型实现 `Default` trait 和空的 `new` 构造函数是常见和有必要的。
`new` 是 Rust 中常规的构造函数，所以不使用参数来构造基本的类型时， `new` 对使用者来说就理应存在。
`default` 方法功能上与 `new` 方法一致，所以也应当存在。

## 建议实现 Debug Trait

- 几乎所有的类型都能、应该实现 Debug
  - `#[derive(Debug)]`，通常是最佳实现方式
    - 注意：派生的 Trait 会为任意泛型参数添加相同的约束（bound）
  - 利用 `fmt::Formatter` 提供的各种 debug_xxx 辅助方法手动实现
    - `debug_struct`
    - `debug_tuple`
    - `debug_list`
    - `debug_set`
    - `debug_map`

例子一

```rust
use std::fmt::Debug;

#[derive(Debug)]
struct Pair<T> {
  a: T,
  b: T,
}

fn main() {
  let pair = Pair {a: 5, b: 10};
  println!("Pair: {:?}", pair); // i32 实现了 Debug Trait 故可以打印出来
}
```

例子二

```rust
use std::fmt::Debug;

struct Person {
  name: String,
}

#[derive(Debug)]
struct Pair<T> {
  a: T,
  b: T,
}

fn main() {
  let pair = Pair {
    a: Person { name: "Dave".to_string() },
    b: Person { name: "Nick".to_string() },
  }；
  println!("Pair: {:?}", pair);  // 报错 `Person` doesn't implement `Debug` Person 没有实现 Debug Trait
}
```

例子三

```rust
use std::fmt;

struct Pair<T> {
  a: T,
  b: T,
}

impl<T: fmt::Debug> fmt::Debug for Pair<T> {
  fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
    f.debug_struct("Pair").field("a", &self.a).field("b", &self.b).finish()
  }
}

fn main() {
  let pair = Pair { a: 5, b: 10 };
  println!("Pair: {:?}", pair);
}
```

## 建议实现 Send 和 Sync（unpin）

- 不是 Send 的类型无法放在 Mutex 中，也不能在包含线程池的应用程序中传递使用

例子四

```rust
#[derive(Debug)]
struct MyBox(*mut u8);

unsafe impl Send for MyBox {}

use std::rc::Rc;

fn main() {
  let mb = MyBox(Box::into_raw(Box::new(42)));
  
  let x = Rc::new(42);
  
  std::thread::spawn(move || {
    println!("{:?}", x); // error: `Rc<i32>` cannot be sent between threads safely
  });
  
  //std::thread::spawn(move || {
  //  println!("{:?}", mb); // mb 实现了 Send Trait 
  //});
}
```

- 不是 Sync 的类型无法通过 Arc 共享，也无法被放置在静态变量中

例子五

```rust
use std::cell::RefCell;
use std::sync::Arc;

fn main() {
  let x = Arc::new(RefCell::new(42));
  std::thread::spawn(move || {
    let mut x = x.borrow_mut(); // error: `RefCell<i32>` cannot be shared between threads safely
    *x += 1;
  });
}
```

- 如果没实现上述 Trait，建议在文档中说明

## 建议实现 Clone 和 Default

例子六

```rust
#[derive(Debug, Clone)]
struct Person {
  name: String,
  age: u32,
}

impl Person {
  fn new(name: String, age: u32) -> Person {
    Person { name, age }
  }
}

fn main() {
  let person1 = Person::new("Alice".to_owned(), 25);
  let person2 = person1.clone();
  
  println!("Person 1: {:?}", person1);
  println!("Person 2: {:?}", person2);
}
```

例子七

```rust
#[derive(Default)]
struct Point {
  x: i32,
  y: i32,
}

fn main() {
  let point = Point::default(); // 提供默认的初始值
  
  println!("Point: ({}, {})", point.x, point.y); // Point: (0, 0)
}
```



- 如果没实现上述 Trait，建议在文档中说明

## 建议实现 PartialEq、PartialOrd、Hash、Eq、Ord

- PartialEq 特别有用
  - 用户会希望使用 == 或 assert_eq! 比较你类型的两个实例

例子八

```rust
#[derive(Debug, PartialEq)]
struct Point {
  x: i32,
  y: i32,
}

fn main() {
  let point1 = Point { x: 1, y: 2 };
  let point2 = Point { x: 1, y: 2 };
  let point3 = Point { x: 3, y: 4 };
  
  println!("point1 == point2: {}", point1 == point2);
  println!("point1 == point3: {}", point1 == point3);
}
```

- PartialOrd 和  Hash 相对更专门化
  - 将类型作为 Map 中的 Key
    - 须实现 PartialOrd，以便进行 Key 的比较
  - 使用 `std::collection` 的集合类型进行去重的类型
    - 须实现 Hash，以便进行哈希计算

例子九

```rust
use std::collections::BTreeMap;

#[derive(Debug, PartialEq, PartialOrd, Eq, Ord, Clone)]
struct Person {
  name: String,
  age: u32,
}

fn main() {
  let mut ages = BTreeMap::new();
  
  let person1 = Person {
    name: "Alice".to_owned(),
    age: 25,
  };
  let person2 = Person {
    name: "Bob".to_owned(),
    age: 30,
  };
  let person3 = Person {
    name: "Charlie".to_owned(),
    age: 20,
  };
  
  ages.insert(person1.clone(), "Alice's age");
  ages.insert(person2.clone(), "Bob's age");
  ages.insert(person3.clone(), "Charlie's age");
  
  for (person, description) in &ages {
    println!("{}: {} - {:?}", person.name, person.age, description);
  }
}
```

例子十

```rust
use std::collections::HashSet;
use std::hash::{Hash, Hasher};

#[derive(Debug, PartialEq, Eq, Clone)]
struct Person {
  name: String,
  age: u32,
}

impl Hash for Person {
  fn hash<H: Hasher>(&self, state: &mut H) {
    self.name.hash(state);
    self.age.hash(state);
  }
}

fn main() {
  let mut persons = HashSet::new();
  
  let person1 = Person {
    name: "Alice".to_owned(),
    age: 25,
  };
  let person2 = Person {
    name: "Bob".to_owned(),
    age: 30,
  };
  let person3 = Person {
    name: "Charlie".to_owned(),
    age: 20,
  };
  
  persons.insert(person1.clone());
  persons.insert(person2.clone());
  persons.insert(person3.clone());
  
  println!("Persons: {:?}", persons);
}
```

- Eq 和 Ord 有额外的语义要求（相对 PartialEq 和 PartialOrd）
  - 只应在确信这些语义适用于你的类型时才实现它们

例子十一

```rust
// Eq
// 反身性（Reflexivity）：对于任何对象 x，x == x 必须为真。
// 对称性（Symmetry）：对于任何对象 x 和 y，如果 x == y 为真，则 y == x 也必须为真。
// 传递性（Transitivity）：对于任何对象 x、y 和 z，如果 x == y 为真，并且 y == z 为真，则 x == z 也必须为真。

// Ord
// 自反性（Reflexivity）：对于任何对象 x，x <= x 和 x >= x 必须为真。
// 反对称性（Antisymmetry）：对于任何对象 x 和 y，如果 x <= y 和 y <= x 都为真，则 x == y 必须为真。
// 传递性（Transitivity）：对于任何对象 x、y 和 z，如果 x <= y 和 y <= z 都为真，则 x <= z 必须为真。


fn main() {
  
}
```

## 建议实现 serde 下的 Serialize、Deserialize

- `serde_derive`（crate）提供了机制，可以覆盖单个字段或枚举变体的序列化
  - 由于 serde 是第三方库，你可能不希望强制添加对它的依赖
  - 大多数库选择提供一个 serde 的功能（feature），只有当用户选择启用该功能时才添加对 serde 的支持

例子十二：你写的库

```toml
[dependencies]
serde = { version = "1.0", optional = true}

[features]
serde = ["serde"]
```

例子十三：别人用的时候

```toml
[dependencies]
mylib = { version = "0.1", features = ["serde"] }
```

## 为什么没建议实现 Copy

- 用户通常不期望类型是 Copy 的
  - 如果想要两个副本，通常希望调用 clone
- Copy 改变了移动给定类型值的语义
  - 让用户 surprise
- Copy 类型受到很多限制，一个最初简单的类型很容易变得不再满足 Copy 的要求
  - 例如持有了 String 或者其他非 Copy 的类型  --->  不得不移除 Copy

例子十四

```rust
#[derive(Debug, Copy, Clone)]
struct Point {
  x: i32,
  y: i32,
}

fn main() {
  let point1 = Point { x: 10, y: 20 };
  let point2 = point1; // 这里发生复制，而不是移动
  
  println!("point1: {:?}", point1);
  println!("point2: {:?}", point2);
}
```

## 人体工程学 Trait 实现

- Rust 不会自动为实现 Trait 的类型的引用提供对应的实现
  - Bar 实现了 Trait，也不能将 &Bar 传递给 `fn foo<T: Trait>(t: T)`
    - 因为 Trait 可能包含接受 &mut self 或 self 的方法，而这些方法无法在 &Bar 上调用
  - 对于看到 Trait 只有 &self 方法的用户来说，这会非常令人惊讶
- 定义新的 Trait 时，通常需要为下列提供相应的全局实现
  - `&T where T: Trait`
  - `&mut T where T: Trait`
  - `Box<T> where T: Trait`
- Iterator（迭代器）：为类型的引用添加 Trait 实现
  - 对于任何可迭代的类型，考虑为 &MyType 和 &mut MyType 实现 IntoIterator
    - 在循环中可直接使用借用实例，符号用户预期。

## 包装类型（Wrapper Types）

- Rust 没有传统意义上的继承
- Deref 和 AsRef 提供了类似继承的东西
  - 你有一个类型为 T 的值，并满足 `T: Deref<Target = U>`，可以在 T 类型值上直接调用类型 U 的方法

例子十五

```rust
use std::ops::Deref;

struct MyVec(Vec<i32>);

impl Deref for MyVec {
  type Target = Vec<i32>;
  
  fn deref(&self) -> &Self::Target {
    &self.0
  }
}

fn main() {
  let my_vec = MyVec(vec![1, 2, 3, 4, 5]);
  
  println!("Length: {}", my_vec.len());
  println!("First element: {}", my_vec[0]);
}
```

- 如果你提供了相对透明的类型（例 Arc）
  - 实现 Deref 允许你的包装类型在使用点运算符时，自动解引用为内部类型，从而可以直接调用内部类型的方法
  - 如果访问内部类型不需要任何复杂或潜在的低效逻辑，应考虑实现 AsRef，这样用户可以轻松地将 &WrapperType 作为 &InnerType 使用
  - 对于大多数包装类型，还应该在可能的情况下实现 `From<InnerType>` 和 `Into<InnerType>`，以便用户可轻松地添加或移除包装。

例子十六

```rust
use std::ops::Deref;

struct Wrapper(String);

impl Deref for Wrapper {
  type Target = String;
  
  fn deref(&self) -> *Self::Target {
    &self.0
  }
}

impl AsRef<str> for Wrapper {
  fn as_ref(&self) -> &str {
    &self.0
  }
}

impl From<String> for Wrapper {
  fn from(s: String) -> Self {
    Wrapper(s)
  }
}

impl From<Wrapper> for String {
  fn from(wrapper: Wrapper) -> Self {
    wrapper.0
  }
}

fn main() {
  let wrapper = Wrapper::from("Hello".to_string());
  
  // 使用 . 运算符调用内部字符串类型的方法
  println!("Length: {}", wrapper.len());
  
  // 使用 as_ref 方法将 Wrapper 转换为 &str 类型
  let inner_ref: &str = wrapper.as_ref();
  println!("Inner: {}", inner_ref);
  
  // 将 Wrapper 转换为内部类型 String
  let inner_string: String = wrapper.into();
  println!("Inner String: {}", inner_string);
}
```

- Borrow Trait （与 Deref 和 AsRef 有些类似）
  - 针对更为狭窄的使用情况进行了定制：
    - 允许调用者提供同一类型的多个本质上相同的变体中的任意一个
      - 可叫做：Equivalent
      - 例：对于一个 `HashSet<String>`，Borrow 允许调用者提供 `&str` 或 `&String`。
        - 虽然使用 AsRef 也可以实现类似的效果，但如果没有 Borrow 的额外要求，这种实现时不安全的，因为 Borrow 要求目标类型实现的 Hash、Eq、和 Ord 必须与实现类型完全相同
    - Borrow 还为 `Borrow<T>`、`&T` 和 `&mut T` 提供了通用实现
      - 这使得在 Trait 约束中使用它来接受给定类型的拥有值或引用值非常方便。
  - Borrow 仅适用于当你的类型本质上与另一个类型等价时
  - 而 Deref 和 AsRef 则适用于更广泛地实现你的类型可以“充当”的情况

例子十七

```rust
use std::borrow::Borrow;

fn print_length<S>(string: S)
where
	S: Borrow<str>,
{
  println!("Length: {}", string.borrow().len());
}

fn main() {
  let str1: &str = "Hello";
  let string1: String = String::from("World");
  
  print_length(str1);
  print_length(string1);
}
```

