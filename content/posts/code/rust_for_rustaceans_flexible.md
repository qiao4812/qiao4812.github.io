---
title: "Rust语言 - 接口设计的建议之灵活（flexible）"
date: 2023-06-08T17:48:03+08:00
draft: false
tags: ["Rust"]
categories: ["Rust"]
---

# Rust - 接口设计建议之灵活（flexible）

## 灵活（flexible）

### 代码的契约（Contract）

- 你写的代码包含契约
- 契约：
  - 要求：代码使用的限制
  - 承诺：代码使用的保证
- 设计接口时（经验法则）：
  - 避免施加不必要的限制，只做能够兑现的承诺
    - 增加限制 或 取消承诺：
      - 重大的语义版本更改
      - 可导致其他代码出问题
    - 放宽限制 或 提供额外的承诺：
      - 通常是向后兼容的

### 限制（Restrictions）与承诺（Promises）

- Rust中，限制的常见形式：
  - Trait 约束（Trait Bound）
  - 参数类型（Argument Types）
- 承诺的常见形式：
  - Trait 的实现
  - 返回类型
- `fn frobnicate1(s: String) -> String`
  - 契约：调用者进行内存分配，承诺返回拥有的 String -> 无法改为 “无需内存分配” 的函数
- `fn frobnicate2(s: &str) -> Cow<'_, str>`
  - 放宽了契约：只接收字符串的引用，承诺返回字符串的引用或一个拥有的 String
- `fn frobnicate3(s: impl AsRef<str>) -> impl AsRef<str>`
  - 进一步放宽契约：要求传入能产生字符串引用的类型，承诺返回值可产生字符串引用

例子一

```rust
use std::borrow::Cow;

fn frobnicate3<T: AsRef<str>>(s: T) -> T {
  s
}

fn main() {
  let string = String::from("example");
  let borrowed: &str = "hello";
  let cow: Cow<str> = Cow::Borrowed("world");
  
  let result1: &str = frobnicate3::<&str>(string.as_ref());
  let result2: &str = frobnicate3::<&str>(borrowed);
  let result3 = frobnicate3(cow);
  
  println!("Result1: {:?}", result1);
  println!("Result2: {:?}", result2);
  println!("Result3: {:?}", result3);
}
```

- 都传入字符串，返回字符串，但契约不同
- 没有更好。要仔细规划契约，否则改变契约会引起破坏

### 泛型参数（Generic Arguments）

- 通过泛型放宽对函数的要求
  - 大多数情况下值得使用泛型代替具体类型

例子二

```rust
// 你有一个函数，它接受一个实现了 AsRef<str> trait 的参数
fn print_as_str<T: AsRef<str>>(s: T) {
  println!("{}", s.as_ref());
}

// 这个函数是泛型的，它对 T 进行了泛型化，
// 这意味着它会对你使用它的每一种实现了 AsRef<str> 的类型进行单态化。
// 例如，如果你用一个 String 和一个 &str 来调用它，
// 你就会在你的二进制文件中有两份函数的拷贝：
fn main() {
  let s = String::from("hello");
  let r = "world";
  print_as_str(s);  // 调用 print_as_str::<String>
  print_as_str(r);  // 调用 print_as_str::<&str>
}
```

例子三

```rust
// 为了避免这种重复，你可以把函数改成接受一个 &dyn AsRef<str>：
fn print_as_str（s: &dyn AsRef<str>) {
  println!("{}", s.as_ref());
}

// 这个函数不再是泛型的，它接受一个 trait 对象，
// 它可以是任何实现了 AsRef<str> 的类型
// 这意味着它会在运行时使用动态分发来调用 as_ref 方法，
// 并且你只会在你的二进制文件中有一份函数的拷贝：
fn main() {
  let s = String::from("hello");
  let r = "world";
  print_as_str(&s);  // 传递一个类型为 &dyn AsRef<str> 的 trait 对象
  print_as_str(&r);  // 传递一个类型为 &dyn AsRef<str> 的 trait 对象
}
```

- 不要走极端
- 经验法则：
  - 用户合理、频繁的使用其他类型代替你最初选定的类型，那么参数定义为泛型更合适
- 问题：通过单态化（monomorphization），会为每个使用泛型代码的类型组合生成泛型代码的副本
  - 担心：让很多参数变成泛型    -->    二进制文件过大
- 解决：动态分发（dynamic dispatch），以忽略不计的性能成本来缓解这个问题
  - 对于以引用方式获取的参数（dyn Trait 不是 Sized 的，需要使用宽指针来使用它们），可以使用动态分发代替泛型参数

例子四

```rust
// 假设我们有一个名为 process 的泛型函数，它接受一个类型参数 T 并对其执行某些操作：
fn process<T>(value: T) {
  // 处理 value 的代码
  println!("处理 T");
}
// 上述函数使用静态分发，这意味着在编译时将为每个具体类型 T 生成相应的实现。

// 现在，假设调用者想要提供动态分发的方式，允许在运行时选择实现。
// 它们可以通过传递 Trait 对象作为参数，
// 使用 dyn 关键字来实现。以下是一个例子：
trait Processable {
  fn process(&self);
}

struct TypeA;
impl Processable for TypeA {
  fn process(&self) {
    println!("处理 TypeA");
  }
}

struct TypeB;
impl Processable for TypeB {
  fn process(&self) {
    println!("处理 TypeB");
  }
}

fn process_trait_object(value: &dyn Processable) {
  value.process();
}

// 如果调用者想要使用动态分发并在运行时选择实现，
// 它们可以调用 process_trait_object 函数，并传递 Trait 对象作为参数。
// 调用者可以根据需求选择要提供的具体实现：
fn main() {
  let a = TypeA;
  let b = TypeB;
  
  process_trait_object(&a);
  process_trait_object(&b);
  
  process(&a);
  process(&b);
  process(&a as &dyn Processable);
  process(&b as &dyn Processable);
  
}
```

- 使用动态分发（dynamic dispatch）：
  - 代码不会对性能敏感：可以接受
  - 在高性能应用中：在频繁调用的热循环中使用动态分发可能会成为一个致命问题
- 在撰写本文时，只有在简单的 Trait 约束时，才能使用动态分发
  - 如 `T: AsRef<str>` 或 `impl AsRef<str>`
- 对于更复杂的约束，Rust 无法构造动态分发的虚函数表（vtable）
  - 因此无法使用类似 `&dyn Hash + Eq` 这样的组合约束。
- 使用泛型时，调用者始终可以通过传递一个 Trait 对象来选择动态分发
- 反过来不成立：如果你接受一个 Trait 对象作为参数，那么调用者必须提供 Trait 对象，而无法选择使用静态分发
- 从具体类型开始编写接口，然后逐渐将它们转换为泛型
  - 可行，但不一定是向下兼容

例子五

```rust
fn foo(v: &Vec<usize>) {
  // 处理 v 的代码
  // ...
}

// 现在，我们决定将函数改为使用 Trait 限定 AsRef<[usize]>，
// 即 impl AsRef<[usize]>：
// fn foo(v: impl AsRef<[usize]>) {
//      // 处理 v 的代码
//    // ...
// }

fn main() {
  let iter = vec![1, 2, 3].into_iter();
  foo(&iter.collect());
}

// 在原始版本中，编译器可以推断出 iter.collect() 应该收集为一个 Vec<usize> 类型，
// 因为我们将其传递给了接受 &Vec<usize> 的 foo 函数。
// 然而，在更改为使用特质限定后，编译器只知道 foo 函数
// 接受一个实现了 AsRef<[usize]> 特质的类型。
// 这里有多个类型满足这个条件，例如 Vec<usize> 和 &[usize]。
// 因此，编译器无法确定应该将 iter.collect() 的结果解释为哪个具体类型。
// 这样的更改将导致编译器无法推断类型，并且调用者的代码将无法通过编译。

// 为了解决这个问题，调用者可能需要显示指定期望的类型，例如：
// let iter = vec![1, 2, 3].into_iter();
// foo(&iter.collect::<Vec<usize>>());
```

#### [泛型的优点](https://rust-chinese-translation.github.io/api-guidelines/flexibility.html#泛型的优点)

- **可复用**：泛型函数能应用在广泛的类型上，同时明确给出了这些类型的必须满足的关系。

- **静态分派和编译器优化**： 每个泛型函数都被专门用于实现了 trait bounds 的具体的类型 （即 单态化 [monomorphized](https://doc.rust-lang.org/book/ch10-01-syntax.html#performance-of-code-using-generics) ），这意味着：

  1. 调用的 trait 方法是静态生成的，因此是直接对 trait 实现的调用
  2. 编译器能对这些调用做内联 (inline) 和其他优化

- **内联式布局**：如果结构体和枚举体类型具有某个泛型参数 `T` ， `T` 的值将在结构体和枚举体里以内联方式排列，不产生任何间接调用。

- **可推断**：由于泛型函数的类型参数通常是推断出来的， 泛型函数可以减少复杂的代码，比如显式转换、通常必须的一些方法调用。

- **精确的类型**：因为泛型给实现了某个 trait 的具体类型一个名称， 从而有可能清楚这个类型需要或创建的地方在哪。比如这个函数：

  ```rust
  fn binary<T: Trait>(x: T, y: T) -> T
  ```

  会保证消耗和创建具有相同类型 `T` 的值；不可能传入实现了 `Trait` 的但不同名称的两个类型。

#### [泛型的缺点](https://rust-chinese-translation.github.io/api-guidelines/flexibility.html#泛型的缺点)

- **增加代码大小**：单态化泛型函数意味着函数体会被复制。 增加代码大小和静态分派的性能优势之间必须做出衡量。
- **类型同质化**：这是 “精确的类型” 带来的另一面： 如果 `T` 是类型参数，那么它代表一个单独的实际类型。 对于像 `Vec<T>` 这样具体的单独的元素类型也是一样， 而且 `Vec` 实际上为了内联这些元素，进行了专门的处理。 有时候，不同的类型会更有用，参考 [trait objects](https://rust-chinese-translation.github.io/api-guidelines/flexibility.html#c-object) 。
- **签名冗余**：过度使用泛型会造成阅读和理解函数签名更困难。

***The Rust RFC Book***：<https://rust-lang.github.io/rfcs/introduction.html>

### 对象安全（Object Safety）

- 定义 Trait 时，它是否对象安全，也是契约未写明的一部分
- 如果 Trait 是对象安全的：
  - 可使用 dyn Trait 将实现该 Trait 的不同类型视为单一通用类型
- 如果 Trait 不是对象安全的：
  - 编译器会禁止使用 dyn Traie
- 建议 Trait 是对象安全的（即使稍微降低使用的便利程度）：
  - 提供了使用的新方式和灵活性

对象安全：描述一个 Trait 可否安全的包装成 Trait Object

对象安全的 Trait 是满足以下条件的 Trait（RFC 255）：

- 所有的 supertrait 必须是对象安全的
- Sized 不能作为 supertrait（不能要求 Self: Sized）
- 不能有任何关联常量
- 不能有任何带有泛型的关联类型
- 所有的关联函数必须满足以下条件之一：
  - 可以从 Trait 对象分发的函数（Dispatchable functions）：
    - 没有任何类型参数（生命周期参数是允许的）
    - 是一个方法，只在接收器类型中使用 Self
    - 接收器是以下类型之一：
      - `&Self`（即 &self）
      - `&mut Self`（即 `&mut self`）
      - `Box<Self>`
      - `Rc<Self>`
      - `Arc<Self>`
      - `Pin<P>`，其中 P 是上述类型之一
    - 没有 `where Self: Sized` 约束（Self 的接收器类型（即 self）暗含了这一点）
  - 显示不可分发的函数（non-dispatchable functions）要求：
    - 具有 `where Self: Sized` 约束（Self 的接收器类型（即 self）暗含了这一点）

例子六

```rust
// 假设我们有一个 Animal 特征，它有两个方法：name 和 speak。
// name 方法返回一个&str，表示动物的名字；
// speak 方法打印出动物发出的声音。
// 我们可以为 Dog 和 Cat 类型实现这个特征：
trait Animal {
  fn name(&self) -> &str;
  fn speak(&self);
}

struct Dog {
  name: String,
}

impl Animal for Dog {
  fn name(&self) -> &str {
    &self.name
  }
  
  fn speak(&self) {
    println!("Woof!");
  }
}

struct Cat {
  name: String,
}

impl Animal for Cat {
  fn name(&self) -> &str {
    &self.name
  }
  
  fn speak(&self) {
    println!("Meow!");
  }
}

// 这个 Animal 特征是 object-safe 的，因为它没有返回 Self 类型或使用泛型参数。
// 所以我们可以用它来创建一个 trait object：
fn main() {
  let dog = Dog {
    name: "Fido".to_string(),
  };
  let cat = Cat {
    name: "Whiskers".to_string(),
  };
  
  let animals: Vec<&dyn Animal> = vec![&dog, &cat];
  
  for animal in animals {
    println!("This is {}", animal.name());
    animal.speak();
  }
}
// 这样我们就可以用一个统一的类型 Vec<&dyn Animal> 来存储不同类型的动物，
// 并且通过 trait object 来调用它们的方法。
```

例子七

```rust
// 但是如果我们给 Animal 特征添加一个新的方法 clone，它返回一个 Self 类型：
trait Animal {
  fn name(&self) -> &str;
  fn speak(&self);
  fn clone(&self) -> Self;
}
// 那么这个特征就不再是 object-safe 的了，
// 因为 clone 方法违反了规则：返回类型不能是 Self。
// 这样我们就不能用它来创建 trait object 了，
// 因为编译器无法知道 Self 具体指代哪个类型

struct Dog {
  name: String,
}

impl Animal for Dog {
  fn name(&self) -> &str {
    &self.name
  }
  
  fn speak(&self) {
    println!("Woof!");
  }
  
  fn clone(&self) -> Self
  where
    Self: Sized,
  {
    todo!()
  }
}

struct Cat {
  name: String,
}

impl Animal for Cat {
  fn name(&self) -> &str {
    &self.name
  }
  
  fn speak(&self) {
    println!("Meow!");
  }
  
  fn clone(&self) -> Self
  where
    Self: Sized,
  {
    todo!()
  }
}

fn main() {
  let dog = Dog {
    name: "Fido".to_string(),
  };
  let cat = Cat {
    name: "Whiskers".to_string(),
  };
  
  let animals: Vec<&dyn Animal> = vec![&dog, &cat]; // 报错 the trait `Animal` cannot be made into an object consider moving `clone` to another trait
  
  for animal in animals {
    println!("This is {}", animal.name());
    animal.speak();
  }
}
```

例子八

```rust
// 如果我们想让 Animal 特征保持 object-safe，
// 我们就不能给它添加返回 Self 类型的方法。
// 或者，我们可以给 clone 方法添加一个 where Self: Sized 的特征界定，
// 这样他就只能在具体类型上调用，而不是在 trait object 上：
trait Animal {
  fn name(&self) -> &str;
  fn speak(&self);
  fn clone(&self) -> Self
  where
    Self: Sized;
}

struct Dog {
  name: String,
}

impl Animal for Dog {
  fn name(&self) -> &str {
    &self.name
  }
  
  fn speak(&self) {
    println!("Woof!");
  }
  
  fn clone(&self) -> Self
  where
    Self: Sized,
  {
    todo!()
  }
}

struct Cat {
  name: String,
}

impl Animal for Cat {
  fn name(&self) -> &str {
    &self.name
  }
  
  fn speak(&self) {
    println!("Meow!");
  }
  
  fn clone(&self) -> Self
  where
    Self: Sized,
  {
    todo!()
  }
}

// 这样我们就可以继续用 Animal 特征来创建 trait object 了，
// 但是我们不能用 trait object 来调用 clone 方法
fn main() {
  let dog = Dog {
    name: "Fido".to_string(),
  };
  let cat = Cat {
    name: "Whiskers".to_string(),
  };
  
  cat.clone(); // 只能在具体的类型上调用
  
  let animals: Vec<&dyn Animal> = vec![&dog, &cat]; 
  
  for animal in animals {
    println!("This is {}", animal.name());
    animal.speak();
    animal.clone(); // 报错 the `clone` method cannot be invoked on a trait object 
  }
}
```

- 如果 Trait 必须有泛型方法，考虑：
  - 泛型参数放在 Trait 上
  - 泛型参数可否使用动态分发，来保证Trait 的对象安全

例子九

```rust
use std::collections::HashSet;
use std::hash::Hash;
// 将泛型参数放在 Trait 本身上
trait Container<T> {
  fn contains(&self, item: &T) -> bool;
}

// 我们可以为不同的容器类型实现 Container Trait，每个实现都具有自己特定的元素类型。
// 例，我们可以为 Vec<T> 和 HashSet<T> 实现 Container Trait：
impl<T> Container<T> for Vec<T>
where
  T: PartialEq,
{
  fn contains(&self, item: &T) -> bool {
    self.iter().any(|x| x == item)
  }
}

impl<T> Container<T> for HashSet<T>
where
  T: Hash + Eq,
{
  fn contains(&self, item: &T) -> bool {
    self.contains(item)
  }
}

fn main() {
  // 创建一个 Vec<T> 和 HashSet<T> 的实例
  let vec_container: Box<dyn Container<i32>> = Box::new(vec![1, 2, 3]);
  let hashset_container: Box<dyn Container<i32>> = Box::new(vec![4, 5, 6].into_iter().collect::<HashSet<_>>());
  
  // 调用 contains 方法
  println!("Vec contains 2: {}", vec_container.contains(&2));
  println!("HashSet contains 6: {}", hashset_container.contains(&6));
}
```

例子十

```rust
use std::fmt::Debug;
// 假设我们有一个 Trait Foo，它有一个泛型方法 bar，它接受一个泛型参数 T：
// trait Foo {
//  fn bar<T>(&self, x: T);
//}

// 这个 Trait 是不是 object-safe 的呢？答案是：取决于 T 的类型。  注意：它不是对象安全的
// 如果 T 是一个具体类型，比如 i32或 String，那么它就不是 object-safe 的，
// 因为它需要在运行时知道 T 的具体类型才能调用 bar 方法。
// 但如果 T 也是一个 trait object，比如 &dyn Debug 或 &dyn Display,
// 那么这个 Trait 就是 object-safe 的，因为它可以用动态分发的方式来调用 T 的方法。
// 所以我们可以这样写：
trait Foo {
  fn bar(&self, x: &dyn Debug);
}

// 定义一个结构体 A，它实现了 Foo 特征
struct A {
  name: String,
}

impl Foo for A {
  fn bar(&self, x: &dyn Debug) {
    println!("A {} says {:?}", self.name, x);
  }
}

// 定义一个结构体 B，它也实现了 Foo 特征
struct B {
  id: i32,
}

impl Foo for B {
  fn bar(&self, x: &dyn Debug) {
    println!("B {} says {:?}", self.id, x);
  }
}

// 这样我们就可以用 Foo 特征来创建 trait object 了，比如：
fn main() {
  // 创建两个不同类型的值，它们都实现了 Foo 特征
  let a = A {
    name: "Alice".to_string(),
  };
  let b = B { id: 42};
  
  // 创建一个 Vec，它存储了 Foo 的 trait object
  let foos: Vec<&dyn Foo> = vec![&a, &b];
  
  // 遍历 Vec，并用 trait object 调用 bar 方法
  for foo in foos {
    foo.bar(&"Hello"); // "Hello" 实现了 Debug 特征
  }
}
```

- 为实现对象安全，需要做出多大牺牲？
  - 考虑你的 Trait 会被怎样使用，用户是否想把它当做 Trait 对象
    - 用户想使用你的 Trait 的多种不同实例  -> 努力实现对象安全

### 借用 VS 拥有（Borrowed vs Owned）

- 针对 Rust 中几乎每个函数、Trait 和类型，须决定：
  - 是否应该拥有数据
  - 仅持有对数据的引用
- 如果代码需要数据的所有权：
  - 它必须存储拥有的数据
- 当你的代码必须拥有数据时：
  - 必须让调用者提供拥有的数据，而不是引用或克隆
- 这样可让调用者控制分配，并且可清楚地看到使用相关接口的成本
- 如果代码不需拥有数据：
  - 应操作于引用
- 例外：
  - 像 i32、bool、f64 等 “小类型”
    - 直接存储和复制的成本与通过引用存储的成本相同
    - 并不是所有 Copy 类型都适用：
      - 例：[u8; 8192] 是 Copy 类型，但在多个地方存储和复制它会很昂贵
- 无法确定代码是否需要拥有数据，因为它取决于运行时情况
- Cow 类型：
  - 允许在需要时持有引用或拥有值
- 如果只有引用的情况下要求生成拥有的值：
  - Cow 将使用 ToOwned trait 在后台创建一个，通常是通过克隆
- 通常在返回类型中使用 Cow 来表示有时会分配内存的函数

例子十一

```rust
use std::borrow::Cow;

// 假设我们有一个函数 process_data，它接收一个字符串参数，
// 并根据一些条件对其进行处理。有时，我们需要修改输入字符串，
// 并拥有对修改后的字符串的所有权。
// 然而，大多数情况下，我们只是对输入字符串进行读取操作，而不需要修改它。
fn process_data(data: Cow<str>) {
  if data.contains("invalid") {
    // 如果输入字符串包含 “invalid”，我们需要修改它
    let owned_data: String = data.into_owned();
    // 进行一些修改操作
    println!("Processed data: {}", owned_data);
  } else {
    // 如果输入字符串不包含 “invalid”，我们只需要读取它
    println!("Data: {}", data);
  }
}

// 在这个例子中，我们使用了 Cow<str> 类型作为参数类型。
// 当调用函数时，我们可以传递一个普通的字符串引用（&str）
// 或一个拥有所有权的字符串（String）作为参数。
fn main() {
  let input1 = "This is valid data.";
  process_data(Cow::Borrowed(input1));
  
  let input2 = "This is invalid data.";
  process_data(Cow::Owned(input2.to_owned()));
}
```

- 有时，引用生命周期会让接口复杂，难以使用
  - 如果用户使用接口时遇到编译问题，这表明您可能需要（即使不必要）拥有某些数据的所有权
    - 这样做的话，建议首先考虑容易克隆或不涉及性能敏感性的数据，而不是直接对大块数据的内容进行堆分配
    - 这样做可以避免性能问题并提高接口的可用性

### 可失败和阻塞的析构函数（Fallible and Blocking Destructors）

- 析构函数（Destructor）：在值被销毁时执行特定的清理操作
- 析构函数由 Drop trait 实现：它定义了一个 drop 方法
- 析构函数通常是不允许失败的，并且是非阻塞执行的。但有时：
  - 例如释放资源时，可能需要关闭网络连接或写入日志文件，这些操作都有可能发生错误
  - 可能需要执行阻塞操作，例如等待一个线程的结束或等待一个异步任务的完成
- 针对 I/O 操作的类型，在丢弃时需要执行清理
  - 例：将写入的数据刷新到磁盘、关闭打开的文件、断开网络连接
- 这些清理操作应在类型的 Drop 实现中完成
  - 问题：一旦值被丢弃，就无法向用户传递错误信息，除非通过 panic
  - 异步代码也有类似问题：希望在清理过程中完成这些工作，但有其他工作处于 pending 状态
    - 可尝试启动另一个执行器，但这会引入其他问题，例如在异步代码中阻塞
- 没有完美解决方案：需要通过 Drop 尽力清理
  - 如果清理出错了，至少我们尝试了 —— 忽略错误并继续
  - 如果还有可用的执行器，可尝试生成一个 future 来做清理，但如果 future 永不会运行，我们也尽力了
- 若用户不想留下“松散” 线程：提供显式的析构函数
  - 这通常是一个方法，它获得 self 的所有权并暴露任何错误（使用  -> Result<*,*>）或异步性（使用 async fn），这些都是与销毁相关的

例子十二

```rust
use std::os::fd::AsRawFd;

// 一个表示文件句柄的类型
struct File {
  // 文件名
  name: String,
  // 文件描述符
  fd: i32,
}

// File 类型的方法实现
impl File {
  // 一个构造函数，打开一个文件并返回一个 File 实例
  fn open(name: &str) -> Result<File, std::io::Error> {
    // 使用 std::fs::OpenOptions 打开文件，具有读写权限
    let file = std::fs::OpenOptions::new().read(true).write(true).open(name)?;
    // 使用 std::os::unix::io::AsRawFd 获取文件描述符
    let fd = file.as_raw_fd();
    // 返回一个 File 实例，包含 name 和 fd 字段
    Ok(File {
      name: name.to_string(),
      fd,
    })
  }
  
  // 一个显式的析构器，关闭文件并返回任何错误
  fn close(self) -> Result<(), std::io::Error> {
    // 使用 std::os::unix::io::FromRawFd 将 fd 转换回 std::fs::File
    let file: std::fs::File = unsafe { std::os::unix::io::FromRawFd::from_raw_fd(self.id) };
    // 使用 std::fs::File::sync_all 将任何挂起的写入刷新到磁盘
    file.sync_all()?;
    // 使用 std::fs::File::set_len 将文件截断为零字节
    file.set_len(0)?;
    // 再次使用 std::fs::File::sync_all 刷新截断
    file.sync_all()?;
    // 丢弃 file 实例，它会自动关闭
    drop(file);
    // 返回 Ok(())
    Ok(())
  }
}

// 一个测试 File 类型的主函数
fn main() {
  // 创建一个名为 "test.txt" 的文件，包含一些内容
  std::fs::write("test.txt", "Hello, world!").unwrap();
  // 打开文件并获取一个 File 实例
  let file = File::open("test.txt").unwrap();
  // 打印文件名和 fd
  println!("File name: {}, fd: {}", file.name, file.fd);
  // 关闭文件并处理任何错误
  match file.close() {
    Ok(()) => println!("File closed successfully"),
    Err(e) => println!("Error closing file: {}", e),
  }
  // 检查关闭后的文件大小
  let metadata = std::fs::metadata("test.txt").unwrap();
  println!("File size: {} bytes", metadata.len());
}
```

注意：显式的析构函数需要在文档中突出显示

- 添加显式析构函数时会遇问题：
  - 当类型实现了 Drop，在析构函数中无法将该类型的任何字段移出
    - 因为在显式析构函数运行后，Drop::drop 仍会被调用，它接收 &mut self，要求 self 的所有部分都没有被移动
  - Drop 接受的是 &mut self，而不是 self，因此 Drop 无法实现简单地调用显式析构函数并忽略其结果（因为 Drop 不拥有 self）

例子十三

```rust
use std::os::fd::AsRawFd;

// 一个表示文件句柄的类型
struct File {
  // 文件名
  name: String,
  // 文件描述符
  fd: i32,
}

// File 类型的方法实现
impl File {
  // 一个构造函数，打开一个文件并返回一个 File 实例
  fn open(name: &str) -> Result<File, std::io::Error> {
    // 使用 std::fs::OpenOptions 打开文件，具有读写权限
    let file = std::fs::OpenOptions::new().read(true).write(true).open(name)?;
    // 使用 std::os::unix::io::AsRawFd 获取文件描述符
    let fd = file.as_raw_fd();
    // 返回一个 File 实例，包含 name 和 fd 字段
    Ok(File {
      name: name.to_string(),
      fd,
    })
  }
  
  // 一个显式的析构器，关闭文件并返回任何错误
  fn close(self) -> Result<(), std::io::Error> {
    // 移出 name 字段并打印它
    let name = self.name; // 报错 不能从 `self.name` 中移出值，因为它位于 `&mut` 引用后面
    println!("Closing file {}", name);
    // 使用 std::os::unix::io::FromRawFd 将 fd 转换回 std::fs::File
    let file: std::fs::File = unsafe { std::os::unix::io::FromRawFd::from_raw_fd(self.id) };
    // 使用 std::fs::File::sync_all 将任何挂起的写入刷新到磁盘
    file.sync_all()?;
    // 使用 std::fs::File::set_len 将文件截断为零字节
    file.set_len(0)?;
    // 再次使用 std::fs::File::sync_all 刷新截断
    file.sync_all()?;
    // 丢弃 file 实例，它会自动关闭
    drop(file);
    // 返回 Ok(())
    Ok(())
  }
}

// Drop trait 的实现，用于在值离开作用域时运行一些代码
impl Drop for File {
  // drop 方法，接受一个可变引用到 self 作为参数
  fn drop(&mut self) {
    // 调用 close 方法并忽略它的结果
    let _ = self.close(); // 报错 不能从 `*self` 中移出值，因为它位于 `&mut` 引用后面
    // 打印一条消息，表明文件被丢弃了
    println!("Dropping file {}", self.name);
  }
}

// 一个测试 File 类型的主函数
fn main() {
  // 创建一个名为 "test.txt" 的文件，包含一些内容
  std::fs::write("test.txt", "Hello, world!").unwrap();
  // 打开文件并获取一个 File 实例
  let file = File::open("test.txt").unwrap();
  // 打印文件名和 fd
  println!("File name: {}, fd: {}", file.name, file.fd);
  // 关闭文件并处理任何错误
  match file.close() {
    Ok(()) => println!("File closed successfully"),
    Err(e) => println!("Error closing file: {}", e),
  }
  // 检查关闭后的文件大小
  let metadata = std::fs::metadata("test.txt").unwrap();
  println!("File size: {} bytes", metadata.len());
}
```

- 解决办法（没有完美的），方法之一 ：
  - 将顶层类型作为包装了 Option 的新类型，Option 持有一个内部类型，该类型包含所有的字段
  - 在两个析构函数中使用 Option::take；当内部类型还没有被取走时，调用内部类型的显式析构函数
  - 由于内部类型没有实现 Drop，你可以获取所有字段的所有权
  - 缺点：想在顶层类型上提供所有的方法，都必须包含通过 Option 来获取内部类型上字段的代码

例子十四

```rust
use std::os::fd::AsRawFd;

// 一个表示文件句柄的类型
struct File {
  // 一个包装在 Option 中的内部类型
  inner: Option<InnerFile>,
}

// 一个内部类型，持有文件名和文件描述符
struct InnerFile {
  // 文件名
  name: String,
  // 文件描述符
  fd: i32,
}

// File 类型的方法实现
impl File {
  // 一个构造函数，打开一个文件并返回一个 File 实例
  fn open(name: &str) -> Result<File, std::io::Error> {
    // 使用 std::fs::OpenOptions 打开文件，具有读写权限
    let file = std::fs::OpenOptions::new().read(true).write(true).open(name)?;
    // 使用 std::os::unix::io::AsRawFd 获取文件描述符
    let fd = file.as_raw_fd();
    // 返回一个 File 实例，包含一个 Some(InnerFile) 的 inner 字段
    Ok(File {
      inner: Some(InnerFile {
        name: name.to_string(),
        fd,
      }),
    })
  }
  
  // 一个显式的析构器，关闭文件并返回任何错误
  fn close(mut self) -> Result<(), std::io::Error> {
    // 使用 Option::take 取出 inner 字段的值，并检查是否是 Some(InnerFile)
    if let Some(inner) = self.inner.take() {
      // 移出 name 和 fd 字段并打印它们
      let name = inner.name;
      let fd = inner.fd;
      println!("Closing file {} with fd {}", name, fd);
      // 使用 std::os::unix::io::FromRawFd 将 fd 转换回 std::fs::File
      let file: std::fs::File = unsafe { std::os::unix::io::FromRawFd::from_raw_fd(self.id) };
      // 使用 std::fs::File::sync_all 将任何挂起的写入刷新到磁盘
      file.sync_all()?;
      // 使用 std::fs::File::set_len 将文件截断为零字节
      file.set_len(0)?;
      // 再次使用 std::fs::File::sync_all 刷新截断
      file.sync_all()?;
      // 丢弃 file 实例，它会自动关闭
      drop(file);
      // 返回 Ok(())
      Ok(())
    } else {
      // 如果 inner 字段是 None，说明文件已经被关闭或丢弃，返回一个错误
      Err(std::io::Error::new(
        std::io::ErrorKind::Other,
        "File already closed or dropped",
      ))
    }
  }
}

// Drop trait 的实现，用于在值离开作用域时运行一些代码
impl Drop for File {
  // drop 方法，接受一个可变引用到 self 作为参数
  fn drop(&mut self) {
    // 使用 Option::take 取出 inner 字段的值，并检查是否是 Some(InnerFile)
    if let Some(inner) = self.inner.take() {
      // 移出 name 和 fd 字段并打印它们
      let name = inner.name;
      let fd = inner.id;
      println!("Dropping file {} with fd {}", name, fd);
      // 使用 std::os::unix::io::FromRawFd 将 fd 转换回 std::fs::File
      let file: std::fs::File = unsafe { std::os::unix::io::FromRawFd::from_raw_fd(fd) };
      // 丢弃 file 实例，它会自动关闭
      drop(file);
    } else {
      // 如果 inner 字段是 None，说明文件已经被关闭或丢弃，不做任何操作
    }
  }
}

// 一个测试 File 类型的主函数
fn main() {
  // 创建一个名为 "test.txt" 的文件，包含一些内容
  std::fs::write("test.txt", "Hello, world!").unwrap();
  // 打开文件并获取一个 File 实例
  let file = File::open("test.txt").unwrap();
  // 打印文件名和 fd
  println!(
    "File name: {}, fd: {}", 
    file.inner.as_ref().unwrap().name, 
    file.inner.as_ref().unwrap().fd
  );
  // 关闭文件并处理任何错误
  match file.close() {
    Ok(()) => println!("File closed successfully"),
    Err(e) => println!("Error closing file: {}", e),
  }
  // 检查关闭后的文件大小
  let metadata = std::fs::metadata("test.txt").unwrap();
  println!("File size: {} bytes", metadata.len());
}
```

- 方法二：
  - 所有字段都可以 take
  - 如果类型具有合理的 ”空“ 值，那么效果很好
  - 如果您必须将几乎每个字段都包装在 Option 中，然后对这些字段的每次访问都进行匹配的 unwrap，很繁琐

例子十五

```rust
use std::os::fd::AsRawFd;

// 一个表示文件句柄的类型
struct File {
  // 文件名，包装在一个 Option 中
  name: Option<String>,
  // 文件描述符，包装在一个 Option 中
  fd: Option<i32>,
}

// File 类型的方法实现
impl File {
  // 一个构造函数，打开一个文件并返回一个 File 实例
  fn open(name: &str) -> Result<File, std::io::Error> {
    // 使用 std::fs::OpenOptions 打开文件，具有读写权限
    let file = std::fs::OpenOptions::new().read(true).write(true).open(name)?;
    // 使用 std::os::unix::io::AsRawFd 获取文件描述符
    let fd = file.as_raw_fd();
    // 返回一个 File 实例，包含一个 Some(name) 和一个 Some(fd) 的字段
    Ok(File {
      name: Some(name.to_string()),
      fd: Some(fd),
    })
  }
  
  // 一个显式的析构器，关闭文件并返回任何错误
  fn close(mut self) -> Result<(), std::io::Error> {
    // 使用 std::mem::take 取出 name 字段的值，并检查是否是 Some(name)
    if let Some(name) = std::mem::take(&mut self.name) {
      // 使用 std::mem::take 取出 fd 字段的值，并检查是否是 Some(fd)
      if let Some(fd) = std::mem::take(&mut self.fd) {
        // 打印文件名和文件描述符
        println!("Closing file {} with fd {}", name, fd);
        // 使用 std::os::unix::io::FromRawFd 将 fd 转换回 std::fs::File
        let file: std::fs::File = unsafe { std::os::unix::io::FromRawFd::from_raw_fd(fd) };
        // 使用 std::fs::File::sync_all 将任何挂起的写入刷新到磁盘
        file.sync_all()?;
        // 使用 std::fs::File::set_len 将文件截断为零字节
        file.set_len(0)?;
        // 再次使用 std::fs::File::sync_all 刷新截断
        file.sync_all()?;
        // 丢弃 file 实例，它会自动关闭
        drop(file);
        // 返回 Ok(())
        Ok(())
      } else {
        // 如果 fd 字段是 None，说明文件已经被关闭或丢弃，返回一个错误
        Err(std::io::Error::new(
          std::io::ErrorKind::Other,
          "File descriptor already taken or dropped",
      ))
    }
  } else {
    // 如果 name 字段是 None，说明文件已经被关闭或丢弃，返回一个错误
    Err(std::io::Error::new(
      std::io::ErrorKind::Other,
      "File name already taken or dropped",
    ))
    }
  }
}

// Drop trait 的实现，用于在值离开作用域时运行一些代码
impl Drop for File {
  // drop 方法，接受一个可变引用到 self 作为参数
  fn drop(&mut self) {
    // 使用 std::mem::take 取出 name 字段的值，并检查是否是 Some(name)
    if let Some(name) = std::mem::take(&mut self.name) {
      // 使用 std::mem::take 取出 fd 字段的值，并检查是否是 Some(fd)
      if let Some(fd) = std::mem::take(&mut self.fd) {
        // 打印文件名和文件描述符
        println!("Dropping file {} with fd {}", name, fd);
        // 使用 std::os::unix::io::FromRawFd 将 fd 转换回 std::fs::File
        let file: std::fs::File = unsafe { std::os::unix::io::FromRawFd::from_raw_fd(fd) };
        // 丢弃 file 实例，它会自动关闭
        drop(file);
      } else {
        // 如果 fd 字段是 None，说明文件已经被关闭或丢弃，不做任何操作
      }
    } else {
      // 如果 name 字段是 None，说明文件已经被关闭或丢弃，不做任何操作
    }
  }
}

// 一个测试 File 类型的主函数
fn main() {
  // 创建一个名为 "test.txt" 的文件，包含一些内容
  std::fs::write("test.txt", "Hello, world!").unwrap();
  // 打开文件并获取一个 File 实例
  let file = File::open("test.txt").unwrap();
  // 打印文件名和 fd
  println!(
    "File name: {}, fd: {}", 
    file.inner.as_ref().unwrap().name, 
    file.inner.as_ref().unwrap().fd
  );
  // 关闭文件并处理任何错误
  match file.close() {
    Ok(()) => println!("File closed successfully"),
    Err(e) => println!("Error closing file: {}", e),
  }
  // 检查关闭后的文件大小
  let metadata = std::fs::metadata("test.txt").unwrap();
  println!("File size: {} bytes", metadata.len());
}
```

- 方法三：
  - 将数据持有在 ManuallyDrop 类型内，它会解引用内部类型，不必再 unwrap
  - 在 drop 中销毁时，可用 ManuallyDrop::take 来获取所有权
  - 缺点：ManuallyDrop::take 是 unsafe 的

例子十六

```rust
// 引入 std 库中的一些模块
use std::{mem::ManuallyDrop, os::fd::AsRawFd};

// 定义一个表示文件句柄的结构体
struct File {
  // 文件名，包装在一个 ManuallyDrop 中
  name: ManuallyDrop<String>,
  // 文件描述符，包装在一个 ManuallyDrop 中
  fd: ManuallyDrop<i32>,
}

// 为 File 结构体实现一些方法
impl File {
  // 一个构造函数，打开一个文件并返回一个 File 实例
  fn open(name: &str) -> Result<File, std::io::Error> {
    // 使用 std::fs::OpenOptions 打开文件，具有读写权限
    let file = std::fs::OpenOptions::new().read(true).write(true).open(name)?;
    // 使用 std::os::unix::io::AsRawFd 获取文件描述符
    let fd = file.as_raw_fd();
    // 返回一个 File 实例，包含一个 ManuallyDrop(name) 和一个 ManuallyDrop(fd) 的字段
    Ok(File {
      name: ManuallyDrop::new(name.to_string()),
      fd: ManuallyDrop::new(fd),
    })
  }
  
  // 一个显式的析构器，关闭文件并返回任何错误
  fn close(mut self) -> Result<(), std::io::Error> {
    // 使用 std::mem::replace 将 name 字段替换为一个空字符串，并获取原来的值
    if let name = std::mem::replace(&mut self.name, ManuallyDrop::new(String::new())); 
    // 使用 std::mem::replace 将 fd 字段替换为一个无效的值，并获取原来的值
    if let fd = std::mem::replace(&mut self.fd, ManuallyDrop::new(-1)); 
    // 打印文件名和文件描述符
    println!("Closing file {:?} with fd {:?}", name, fd);
    // 使用 std::os::unix::io::FromRawFd 将 fd 转换回 std::fs::File
    let file: std::fs::File = unsafe { std::os::unix::io::FromRawFd::from_raw_fd(*fd) };
    // 使用 std::fs::File::sync_all 将任何挂起的写入刷新到磁盘
    file.sync_all()?;
    // 使用 std::fs::File::set_len 将文件截断为零字节
    file.set_len(0)?;
    // 再次使用 std::fs::File::sync_all 刷新截断
    file.sync_all()?;
    // 丢弃 file 实例，它会自动关闭
    drop(file);
    // 返回 Ok(())
    Ok(())
  }
}

// 为 File 结构体实现 Drop trait，用于在值离开作用域时运行一些代码
impl Drop for File {
  // drop 方法，接受一个可变引用到 self 作为参数
  fn drop(&mut self) {
    // 使用 ManuallyDrop::take 取出 name 字段的值，并检查是否是空字符串
    let name = unsafe { ManuallyDrop::take(&mut self.name) };
    // 使用 ManuallyDrop::take 取出 fd 字段的值，并检查是否是无效的值
    let fd = unsafe { ManuallyDrop::take(&mut self.id) };
    // 打印文件名和文件描述符
    println!("Dropping file {:?} with fd {:?}", name, fd);
    
    // 如果 fd 字段不是无效的值，说明文件还没有被关闭或丢弃，需要执行一些操作
    if fd != -1 {
      // 使用 std::os::unix::io::FromRawFd 将 fd 转换回 std::fs::File
      let file: std::fs::File = unsafe { std::os::unix::io::FromRawFd::from_raw_fd(fd) };
      // 丢弃 file 实例，它会自动关闭
      drop(file);
    }
  }
}

// 一个测试 File 类型的主函数
fn main() {
  // 创建一个名为 "test.txt" 的文件，包含一些内容
  std::fs::write("test.txt", "Hello, world!").unwrap();
  // 打开文件并获取一个 File 实例
  let file = File::open("test.txt").unwrap();
  // 打印文件名和 fd
  println!(
    "File name: {}, fd: {}", 
    *file.name, 
    *file.fd
  );
  // 关闭文件并处理任何错误
  match file.close() {
    Ok(()) => println!("File closed successfully"),
    Err(e) => println!("Error closing file: {}", e),
  }
  // 检查关闭后的文件大小
  let metadata = std::fs::metadata("test.txt").unwrap();
  println!("File size: {} bytes", metadata.len());
}
```

- 根据实际情况选择方案
  - 倾向于选择第二个方案
    - 只有发现自己处于一堆 Option 中时才切换到其他选项
  - 如果代码足够简单，可轻松检查代码安全性，那么 ManuallyDrop 方案也挺好
