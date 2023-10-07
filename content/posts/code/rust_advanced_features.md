---
title: "Rust编程语言入门之高级特性"
date: 2023-04-22T15:11:46+08:00
draft: false
tags: ["Rust"]
categories: ["Rust"]
---

# 高级特性

### 主要内容

- 不安全 Rust
- 高级 Trait
- 高级 类型
- 高级函数和闭包
- 宏

## 一、不安全 Rust

### 匹配命名变量

- 隐藏着第二个语言，它没有强制内存安全保证：Unsafe Rust（不安全的 Rust）
  - 和普通的 Rust 一样，但提供了额外的“超能力”
- Unsafe Rust 存在的原因：
  - 静态分析是保守的。
    - 使用 Unsafe Rust：我知道自己在做什么，并承担相应风险
  - 计算机硬件本身就是不安全的，Rust需要能够进行底层系统编程

### Unsafe 超能力

- 使用 unsafe 关键字来切换到 unsafe Rust，开启一个块，里面放着 Unsafe 代码
- Unsafe Rust 里可执行的四个动作（unsafe 超能力）：
  - 解引用原始指针
  - 调用 unsafe 函数或方法
  - 方法或修改可变的静态变量
  - 实现 unsafe trait
- 注意：
  - Unsafe 并没有关闭借用检查或停用其它安全检查
  - 任何内存安全相关的错误必须留在 unsafe 块里
  - 尽可能隔离 Unsafe 代码，最好将其封装在安全的抽象里，提供安全的API

### 解引用原始指针

- 原始指针
  - 可变的：*mut T
  - 不可变的：*const T。意味着指针在解引用后不能直接对其进行赋值
  - 注意：这里的 * 不是解引用符号，它是类型名的一部分。
- 与引用不同，原始指针：
  - 允许通过同时具有不可变和可变指针或多个执行同一位置的可变指针来忽略借用规则
  - 无法保证能指向合理的内存
  - 允许为null
  - 不实现任何自动清理
- 放弃保证的安全，换取更好的性能/与其它语言或硬件接口的能力

### 解引用原始指针

```rust
fn main() {
  let mut num = 5;
  
  let r1 = &num as *const i32;
  let r2 = &mut num as *mut i32;
  unsafe {
    println!("r1: {}", *r1);
    println!("r2: {}", *r2);
  }
  
  let address = 0x012345usize;
  let r = address as *const i32;
  unsafe {
    println!("r: {}", *r); // 报错 非法访问
  }
}
```

- 为什么要用原始指针？
  - 与 C 语言进行接口
  - 构建借用检查器无法理解的安全抽象

### 调用 unsafe 函数或方法

- unsafe 函数或方法：在定义前加上了 unsafe 关键字
  - 调用前需手动满足一些条件（主要靠看文档），因为Rust无法对这些条件进行验证
  - 需要在 unsafe 块里进行调用

```rust
unsafe fn dangerous() {}

fn main() {
  unsafe {
    dangerous();
  }
}
```

### 创建 Unsafe 代码的安全抽象

- 函数包含 unsafe 代码并不意味着需要将整个函数标记为 unsafe
- 将 unsafe 代码包裹在安全函数中是一个常见的抽象

```rust
fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
  let len = slice.len();
  
  assert!(mid <= len);
  
  (&mut slice[..mid], &mut slice[mid..]) // 报错 cannot borrow `*slice` as mutable more than once at a time
}

fn main() {
  let mut v = vec![1, 2, 3, 4, 5, 6];
  
  let r = &mut v[..];
  let (a, b) = r.split_at_mut(3);
  assert_eq!(a, &mut [1, 2, 3]);
  assert_eq!(b, &mut [4, 5, 6]);
}
```

修改之后：

```rust
use std::slice;

fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
  let len = slice.len();
  let ptr = slice.as_mut_ptr()
  
  assert!(mid <= len);
  
  unsafe {
    (
      slice::from_raw_parts_mut(ptr, mid),
      slice::from_raw_parts_mut(ptr.add(mid), len = mid),
    )
  }
}

fn main() {
  let address = 0x012345usize;
  let r = address as *mut i32;
  
  let slice: &[i32] = unsafe {
    slice::from_raw_parts_mut(r, 10000)
  };
}
```

### 使用 extern 函数调用外部代码

- extern 关键字：简化创建和使用外部函数接口（FFI）的过程。
- 外部函数接口（FFI，Foreign Function Interface）：它允许一种编程语言定义函数，并让其它编程语言能调用这些函数

```rust
extern "C" {
  fn abs(input: i32) -> i32;
}

fn main() {
  unsafe {
    println!("Absolute value of -3 according to C: {}", abs(-3));
  }
}
```

- 应用二进制接口（ABI，Application Binary Interface）：定义函数在汇编层的调用方式
- “C” ABI 是最常见的ABI，它遵循 C 语言的ABI

### 从其它语言调用 Rust 函数

- 可以使用 extern 创建接口，其它语言通过它们可以调用 Rust 的函数
- 在 fn 前添加 extern 关键字，并指定 ABI
- 还需添加 `#[no_mangle]`注解：避免 Rust 在编译时改变它的名称

```rust
#[no_mangle]
pub extern "C" fn call_from_c() {
  println!("Just called a Rust function from C!");
}

fn main() {}
```

### 访问或修改一个可变静态变量

- Rust 支持全局变量，但因为所有权机制可能产生某些问题，例如数据竞争
- 在 Rust 里，全局变量叫做静态（static）变量

```rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
  println!("name is: {}", HELLO_WORLD);
}
```

### 静态变量

- 静态变量与常量类似
- 命名：SCREAMING_SNAKE_CASE
- 必须标注类型
- 静态变量只能存储 'static 生命周期的引用，无需显示标注
- 访问不可变静态变量是安全的

### 常量和不可变静态变量的区别

- 静态变量：有固定的内存地址，使用它的值总会访问同样的数据
- 常量：允许使用它们的时候对数据进行复制
- 静态变量：可以是可变的，访问和修改静态可变变量是不安全（unsafe）的

```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
  unsafe {
    COUNTER += inc;
  }
}

fn main() {
  add_to_count(3);
  
  unsafe {
    println!("COUNTER: {}", COUNTER);
  }
}
```

### 实现不安全（unsafe）trait

- 当某个 trait 中存在至少一个方法拥有编译器无法校验的不安全因素时，就称这个 trait 是不安全的
- 声明 unsafe trait：在定义前加 unsafe 关键字
  - 该 trait 只能在 unsafe 代码块中实现

```rust
unsafe trait Foo {
  // methods go here
}

unsafe impl Foo for i32 {
  // method implementations go here
}

fn main() {}
```

### 何时使用 unsafe 代码

- 编译器无法保证内存安全，保证 unsafe 代码正确并不简单
- 有充足理由使用 unsafe 代码时，就可以这样做
- 通过显示标记 unsafe，可以在出现问题时轻松的定位

## 二、高级 Trait

### 在 Trait 定义中使用关联类型来指定占位类型

- 关联类型（associated type）是 Trait中的类型占位符，它可以用于Trait的方法签名中：
  - 可以定义出包含某些类型的 Trait，而在实现前无需知道这些类型是什么

```rust
pub trait Iterator {
  type Item;
  
  fn next(&mut self) -> Option<Self::Item>;
}

fn main() {
  println!("Hello, world!");
}
```

### 关联类型与泛型的区别

| 泛型                                               | 关联类型                         |
| -------------------------------------------------- | -------------------------------- |
| 每次实现 Trait 时标注类型                          | 无需标注类型                     |
| 可以为一个类型多次实现某个 Trait（不同的泛型参数） | 无法为单个类型多次实现某个 Trait |

例子：

```rust
pub trait Iterator {
  type Item;
  
  fn next(&mut self) -> Option<Self::Item>;
}

pub trait Iterator2<T> {
  fn next(&mut self) -> Option<T>;
}

struct Counter {}

impl Iterator for Counter {
  type Item = u32;
  
  fn next(&mut self) -> Option<Self::Item> {
    None
  }
}

impl Iterator2<String> for Counter {
  fn next(&mut self) -> Option<String> {
    None
  }
}

impl Iterator2<u32> for Counter {
  fn next(&mut self) -> Option<u32> {
    None
  }
}

fn main() {
  println!("Hello, world!");
}
```

### 默认泛型参数和运算符重载

- 可以在使用泛型参数时为泛型指定一个默认的具体类型。
- 语法：`<PlaceholderType=ConcreteType>`
- 这种技术常用于运算符重载（operator overloading）
- Rust 不允许创建自己的运算符及重载任意的运算符
- 但可以通过实现 std::ops 中列出的那些 trait 来重载一部分相应的运算符

例子一：

```rust
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Point {
  x: i32,
  y: i32,
}

impl Add for Point {
  type Output = Point;
  
  fn add(self, other: Point) -> Point {
    Point {
      x: self.x + other.x,
      y: self.y + other.y,
    }
  }
}

fn main() {
  assert_eq!(Point {x: 1, y: 0} + Point {x: 2, y: 3},
    Point {x: 3, y: 3}
  );
}
```

例子二：

```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
  type Output = Millimeters;
  
  fn add(self, other: Meters) -> Millimeters {
    Millimeters(self.0 + (other.0 * 1000))
  }
}

fn main() {
  
}
```

### 默认泛型参数的主要应用场景

- 扩展一个类型而不破坏现有代码
- 允许在大部分用户都不需要的特定场景下进行自定义

### 完全限定语法（Fully Qualified Syntax）如何调用同名方法

例子一：

```rust
trait Pilot {
  fn fly(&self);
}

trait Wizard {
  fn fly(&self);
}

struct Human;

impl Pilot for Human {
  fn fly(&self) {
    println!("This is your captain speaking.");
  }
}

impl Wizard for Human {
  fn fly(&self) {
    println!("Up!");
  }
}

impl Human {
  fn fly(&self) {
    println!("*waving arms furiously*");
  }
}

fn main() {
  let persion = Human;
  person.fly(); // Human 本身的 fly 方法
  Pilot::fly(&person);
  Wizard::fly(&person);
}
```

例子二：

```rust
trait Animal {
  fn baby_name() -> String;
}

struct Dog;

impl Dog {
  fn baby_name() -> String {
    String::from("Spot")
  }
}

impl Animal for Dog {
  fn baby_name() -> String {
    String::from("puppy")
  }
}

fn main() {
  println!("A baby dog is called a {}", Dog::baby_name()); // Dog 本身的关联方法
}
```

### 完全限定语法（Fully Qualified Syntax）如何调用同名方法

- 完全限定语法：`<Type as Trait>::function(receiver_if_method, netx_arg, ...);`
  - 可以在任何调用函数或方法的地方使用
  - 允许忽略那些从其它上下文能推导出来的部分
  - 当 Rust 无法区分你期望调用哪个具体实现的时候，才需使用这种语法

```rust
trait Animal {
  fn baby_name() -> String;
}

struct Dog;

impl Dog {
  fn baby_name() -> String {
    String::from("Spot")
  }
}

impl Animal for Dog {
  fn baby_name() -> String {
    String::from("puppy")
  }
}

fn main() {
  println!("A baby dog is called a {}", Dog::baby_name()); // Dog 本身的关联方法
  println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

### 使用 supertrait 来要求 trait 附带其它 trait 的功能

- 需要在一个 trait 中使用其它 trait 的功能：
  - 需要被依赖的 trait 也被实现
  - 那个被间接依赖的 trait 就是当前 trait 的 supertrait

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
  fn outline_print(&self) {
    let output = self.to_string();
    let len = output.len();
    println!("{}", "*".repeat(len + 4));
    println!("*{}*", " ".repeat(len + 2));
    println!("* {} *", output);
    println!("*{}*", " ".repeat(len + 2));
    println!("{}", "*".repeat(len + 4));
  }
}

struct Point {
  x: i32,
  y: i32,
}

impl OutlinePrint for Point {}

impl fmt::Display for Point {
  fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
    write!(f, "({}, {})", self.x, self.y)
  }
}

fn main() {}
```

### 使用 newtype 模式在外部类型上实现外部 trait

- 孤儿规则：只有当 trait 或类型定义在本地包时，才能为该类型实现这个 trait
- 可以通过 newtype 模式来绕过这一规则
  - 利用 tuple struct （元组结构体）创建一个新的类型

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
  fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
    write!(f, "[{}]", self.0.join(", "))
  }
}

fn main() {
  let w = Wrapper(vec![String::from("hello"), String::from("world")]);
  println!("w = {}", w);
}
```

## 三、高级类型

### 使用 newtype 模式实现类型安全和抽象

- newtype 模式可以：
  - 用来静态的保证各种值之间不会混淆并表明值的单位
  - 为类型的某些细节提供抽象能力
  - 通过轻量级的封装来隐藏内部实现细节

### 使用类型别名创建类型同义词

- Rust 提供了类型别名的功能：
  - 为现有类型生产另外的名称（同义词）
  - 并不是一个独立的类型
  - 使用 type 关键字
- 主要用途：减少代码字符重复

例子一：

```rust
type Kilometers = i32;

fn main() {
  let x: i32 = 5;
  let y: Killometers = 5;
  println!("x + y = {}", x + y);
}
```

例子二：

```rust
fn takes_long_type(f: Box<dyn Fn() + Send + 'static>) {
  // --snip--
}

fn returns_long_type() -> Box<dyn Fn() + Send + 'static> {
  Box::new(|| println!("hi"))
}

fn main() {
  let f: Box<dyn Fn() + Send + 'static> = Box::new(|| println!("hi"));
}
```

修改之后：

```rust
type Thunk = Box<dyn Fn() + Send + 'static>;

fn takes_long_type(f: Thunk) {
  // --snip--
}

fn returns_long_type() -> Thunk {
  Box::new(|| println!("hi"))
}

fn main() {
  let f: Thunk = Box::new(|| println!("hi"));
}
```

例子三：

```rust
use std::io::Error;
use std::fmt;

pub trait Write {
  fn write(&mut self, buf: &[u8]) -> Result<usize, Error>;
  fn flush(&mut self) -> Result<(), Error>;
  
  fn write_all(&mut self, buf: &[u8]) -> Result<(), Error>;
  fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<(), Error>;
}

fn main() {
  
}
```

修改之后：

```rust
use std::fmt;

// type Result<T> = Result<T, std::io::Error>; // 声明在 std::io 中

type Result<T> = std::io::Result<T>;

pub trait Write {
  fn write(&mut self, buf: &[u8]) -> Result<usize>;
  fn flush(&mut self) -> Result<()>;
  
  fn write_all(&mut self, buf: &[u8]) -> Result<()>;
  fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<()>;
}

fn main() {
  
}
```

### Never 类型

- 有一个名为 ! 的特殊类型：
  - 它没有任何值，行话称为空类型（empty type）
  - 我们倾向于叫它 never 类型，因为它在不返回的函数中充当返回类型
- 不返回值的函数也被称作发散函数（diverging function）

例子一：

```rust
fn bar() -> ! { // 报错 返回单元类型 不匹配
  
}

fn main() {}
```

例子二：

```rust
fn main() {
  let guess = "";
  
  loop {
    let guess: u32 = match guess.trim().parse() {
      Ok(num) => num,
      Err(_) => continue, // ! never 类型
    };
  }
}
```

注意：never 类型的表达式可以被强制的转化为任意其它类型

例子三：

```rust
impl<T> Option<T> {
  pub fn unwrap(self) -> T {
    match self {
      Some(val) => val,
      None => panic!("called `Option::unwrap()` on a `None` value"), // !
    }
  }
}
```

例子四：

```rust
fn main() {
  println!("forever");
  
  loop {
    println!("and ever");
  }
}
```

### 动态大小和 Sized Trait

- Rust 需要在编译时确定为一个特定类型的值分配多少空间。
- 动态大小的类型（Dynamically Sized Types，DST）的概念：
  - 编写代码时使用只有在运行时才能确定大小的值
- str 是动态大小的类型（注意不是 &str）：只有运行时才能确定字符串的长度
  - 下列代码无法正常工作：
    - `let s1: str = "Hello there!";`
    - `let s2: str = "How's it going?";`
  - 使用 &str 来解决：
    - str 的地址
    - str 的长度

### Rust使用动态大小类型的通用方式

- 附带一些额外的元数据来存储动态信息的大小
  - 使用动态大小类型时总会把它的值放在某种指针后边

### 另外一种动态大小的类型：trait

- 每个 trait 都是一个动态大小的类型，可以通过名称对其进行引用
- 为了将 trait 用作 trait 对象，必须将它放置在某种指针之后
  - 例如 &dyn Trait 或 Box<dyn Trait> (Rc<dyn Trait>) 之后

### Sized trait

- 为了处理动态大小的类型，Rust 提供了一个 Sized trait 来确定一个类型的大小在编译时是否已知
  - 编译时可计算出大小的类型会自动实现这一 trait
  - Rust 还会为每一个泛型函数隐式的添加 Sized 约束

```rust
fn generic<T>(t: T) {}

fn generic<T: Sized>(t: T) {} // 上面的generic 会隐式的转化为这种

fn main() {}
```

- 默认情况下，泛型函数只能被用于编译时已经知道大小的类型，可以通过特殊语法解除这一限制

### ?Sized trait 约束

```rust
fn generic<T>(t: T) {}

fn generic<T: Sized>(t: T) {} 

fn generic<T: ?Sized>(t: &T) {} // ? 只能用在 sized上

fn main() {}
```

- T 可能是也可能不是 Sized
- 这个语法只能用在 Sized 上面，不能被用于其它 trait

## 四、高级函数和闭包

### 函数指针

- 可以将函数传递给其它函数
- 函数在传递过程中会被强制转换成 fn 类型
- fn 类型就是 “函数指针（function pointer）”

```rust
fn add_one(x: i32) -> i32 {
  x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
  f(arg) + f(arg)
}

fn main() {
  let answer = do_twice(add_one, 5);
  
  println!("The answer is: {}", answer);
}
```

### 函数指针与闭包的不同

- fn 是一个类型，不是一个 trait
  - 可以直接指定 fn 为参数类型，不用声明一个以 Fn trait 为约束的泛型参数
- 函数指针实现了全部3种闭包 trait（Fn、FnMut、FnOnce）：
  - 总是可以把函数指针用作参数传递给一个接收闭包的函数
  - 所以，倾向于搭配闭包 trait 的泛型来编写函数：可以同时接收闭包和普通函数
- 某些情景，只想接收 fn 而不接收闭包：
  - 与外部不支持闭包的代码交互：C 函数

例子一

```rust
fn main() {
  let list_of_numbers = vec![1, 2, 3];
  let list_of_strings: Vec<String> = list_of_numbers
  .iter().map(|i| i.to_string()).collect();
  
  let list_of_numbers = vec![1, 2, 3];
  let list_of_strings: Vec<String> = list_of_numbers
  .iter().map(ToString::to_string).collect();
}
```

例子二

```rust
fn main() {
  enum Status {
    Value(u32),
    Stop,
  }
  
  let v = Status::Value(3);
  
  let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect();
}
```

### 返回闭包

- 闭包使用 trait 进行表达，无法在函数中直接返回一个闭包，可以将一个实现了该 trait 的具体类型作为返回值。

```rust
fn returns_closure() -> Fn(i32) -> i32 { // 报错 没有一个已知的大小
  |x| x + 1
}

fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
  Box::new(|x| x + 1)
}

fn main() {
  
}
```

## 五、宏

### 宏 macro

- 宏在Rust里指的是一组相关特性的集合称谓：
  - 使用 macro_rules! 构建的声明宏（declarative macro）
  - 3 种过程宏
    - 自定义 #[derive] 宏，用于 struct 或 enum，可以为其指定随 derive 属性添加的代码
    - 类似属性的宏，在任何条目上添加自定义属性
    - 类似函数的宏，看起来像函数调用，对其指定为参数的 token 进行操作

### 函数与宏的差别

- 本质上，宏是用来编写可以生成其它代码的代码（元编程，metaprogramming）
- 函数在定义签名时，必须声明参数的个数和类型，宏可处理可变的参数
- 编译器会在解释代码前展开宏
- 宏的定义比函数复杂得多，难以阅读、理解、维护
- 在某个文件调用宏时，必须提前定义宏或将宏引入当前作用域：
- 函数可以在任何位置定义并在任何位置使用

### macro_rules! 声明宏（弃用）

- Rust 中最常见的宏形式：声明宏
  - 类似 match 的模式匹配
  - 需要使用 marco_rules!

```rust
// let v: Vec<u32> = vec![1, 2, 3];

#[macro_export]
macro_rules! vec {
  ($($x:expr),*) => {
    {
      let mut temp_vec = Vec::new();
      $(
        temp_vec.push($x);
      )*
      temp_vec
    }
  };
}

// let mut temp_vec = Vec::new();
// temp_vec.push(1);
// temp_vec.push(2);
// temp_vec.push(3);
// temp_vec
```

### 基于属性来生成代码的过程宏

- 这种形式更像函数（某种形式的过程）一些
  - 接收并操作输入的 Rust 代码
  - 生成另外一些 Rust 代码作为结果
- 三种过程宏：
  - 自定义派生
  - 属性宏
  - 函数宏
- 创建过程宏时：
  - 宏定义必须单独放在它们自己的包中，并使用特殊的包类型

```rust
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
  
}
```

### 自定义 derive 宏

- 需求：
  - 创建一个 hello_macro 包，定义一个拥有关联函数 hello_macro 的 HelloMacro trait
  - 我们提供一个能自动实现 trait 的过程宏
  - 在它们的类型上标注 #[derive(HelloMacro)]，进而得到 hello_macro 的默认实现

```bash
➜ cd rust

~/rust
➜ cargo new hello_macro --lib
     Created library `hello_macro` package

~/rust
➜ cd hello_macro

hello_macro on  master [?] via 🦀 1.67.1
➜ c

hello_macro on  master [?] via 🦀 1.67.1
➜


hello_macro on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ cd ..         

~/rust 
➜ cargo new hello_macro_derive --lib
     Created library `hello_macro_derive` package

~/rust 
➜ cd hello_macro_derive             

hello_macro_derive on  master [?] via 🦀 1.67.1 
➜ c                    

hello_macro_derive on  master [?] via 🦀 1.67.1 
➜ 

```

hello_macro_derive 代码：

Cargo.toml

```toml
[package]
name = "hello_macro_derive"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[lib]
proc-macro = true

[dependencies]
syn = "2.0.13"
quote = "1.0.26"

```

src/lib.rs

```rust
extern crate proc_macro;

use crate::proc_macro::TokenStream;
use quote::quote;
use syn;
// #[derive(HelloMacro)]
#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // Construct a representation of Rust code as a syntax tree
    // that we can manipulate
    let ast = syn::parse(input).unwrap();

    // Build the trait implementation
    impl_hello_macro(&ast)
}

fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}", stringify!(#name));
            }
        }
    };
    gen.into()
}

// DeriveInput {
//     // --snip--

//     ident: Ident {
//         ident: "Pancakes",
//         span: #0 bytes(95..103)
//     },
//     data: Struct(
//         DataStruct {
//             struct_token: Struct,
//             fields: Unit,
//             semi_token: Some(
//                 Semi
//             )
//         }
//     )
// }

```

hello_macro 代码：

main.rs

```rust
use hello_macro::HelloMacro;

struct Pancakes;

impl HelloMacro for Pancakes {
    fn hello_macro() {
        println!("Hello, Macro! My name is Pancakes!");
    }
}

fn main() {
    Pancakes::hello_macro();
}

// use hello_macro::HelloMacro;
// use hello_macro_derive::HelloMacro;

// #[derive(HelloMacro)]
// struct Pancakes;

// fn main() {
//     Pancakes::hello_macro();
// }

```

lib.rs

```rust
pub trait HelloMacro {
    fn hello_macro();
}

```

编译

```bash
hello_macro_derive on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ cargo build
   Compiling unicode-ident v1.0.8
   Compiling proc-macro2 v1.0.56
   Compiling quote v1.0.26
   Compiling syn v2.0.15
   Compiling hello_macro_derive v0.1.0 (/Users/qiaopengjun/rust/hello_macro_derive)
    Finished dev [unoptimized + debuginfo] target(s) in 1.54s

hello_macro_derive on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ 

hello_macro on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ cargo build                       
   Compiling hello_macro v0.1.0 (/Users/qiaopengjun/rust/hello_macro)
    Finished dev [unoptimized + debuginfo] target(s) in 0.26s

hello_macro on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ 


创建
~/rust
➜ cargo new pancakes
     Created binary (application) `pancakes` package

~/rust
➜ cd pancakes

pancakes on  master [?] via 🦀 1.67.1
➜ c

pancakes on  master [?] via 🦀 1.67.1
➜


```

main.rs 文件

```rust
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro();
}

```

Cargo.toml 文件

```toml
[package]
name = "pancakes"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
hello_macro = {path = "../hello_macro"}
hello_macro_derive = {path = "../hello_macro_derive"}

```

运行

```bash
pancakes on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ cargo run         
   Compiling hello_macro v0.1.0 (/Users/qiaopengjun/rust/hello_macro)
   Compiling pancakes v0.1.0 (/Users/qiaopengjun/rust/pancakes)
    Finished dev [unoptimized + debuginfo] target(s) in 0.10s
     Running `target/debug/pancakes`
Hello, Macro! My name is Pancakes

pancakes on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ 
```

### 类似属性的宏

- 属性宏与自定义 derive 宏类似
  - 允许创建新的属性
  - 但不是为 derive 属性生成代码
- 属性宏更加灵活：
  - derive 只能用于 struct 和 enum
  - 属性宏可以用于任意条目，例如函数

```rust
// #[route(GET, "/")]
// fn index() {}

// #[proc_macro_attribute]
// pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {}
```

### 类似函数的宏

- 函数宏定义类似于函数调用的宏，但比普通函数更加灵活
- 函数宏可以接收 TokenStream 作为参数
- 与另外两种过程宏一样，在定义中使用 Rust 代码来操作 TokenStream

```rust
// let sql = sql(SELECT * FROM posts WHERE id=1);

// #[proc_macro]
// pub fn sql(input: TokenStream) -> TokenStream {}
```
