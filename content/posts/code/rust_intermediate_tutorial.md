---
title: "Rust语言中级教程"
date: 2023-05-02T11:26:38+08:00
draft: false
tags: ["Rust"]
categories: ["Rust"]
---

# Rust语言中级教程

## 一、指针

### 什么是指针

- 指针是计算机引用无法立即直接访问的数据的一种方式（类比 书的目录）
- 数据在物理内存（RAM）中是分散的存储着
- 地址空间是检索系统
- 指针就被编码为内存地址，使用 usize 类型的整数表示。
  - 一个地址就会指向地址空间中的某个地方
- 地址空间的范围是 OS 和 CPU 提供的外观界面
  - 程序只知道有序的字节序列，不会考虑系统中实际 RAM 的数量

### 名词解释

- 内存地址（地址），就是指代内存中单个字节的一个数
  - 内存地址是汇编语言提供的抽象
- 指针（有时扩展称为原始指针），就是指向某种类型的一个内存地址
  - 指针是高级语言提供的抽象
- 引用，就是指针。如果是动态大小的类型，就是指针和具有额外保证的一个整数
  - 引用是 Rust 提供的抽象

### Rust 的引用

- 引用始终引用的是有效数据
- 引用与 usize 的倍数对齐
- 引用可以为动态大小的类型提供上述保障

### Rust 的引用 和 指针

```rust
static B: [u8; 10] = [99, 97, 114, 114, 121, 116, 111, 119, 101, 108];
static C: [u8; 11] = [116, 104, 97, 110, 107, 115, 102, 105, 115, 104, 0];

fn main() {
    let a = 42;
    let b = &B;
    let c = &C;

    println!("a: {}, b: {:p}, c: {:p}", a, b, c);
}

```

运行

```bash
point_demo on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ cargo run           
   Compiling point_demo v0.1.0 (/Users/qiaopengjun/rust/point_demo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.44s
     Running `target/debug/point_demo`
a: 42, b: 0x1023dc660, c: 0x1023dc66a

point_demo on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ 

```

- 一个更加逼真的例子
  - 使用更复杂的类型展示指针内部的区别

```go
use std::mem::size_of;

static B: [u8; 10] = [99, 97, 114, 114, 121, 116, 111, 119, 101, 108];
static C: [u8; 11] = [116, 104, 97, 110, 107, 115, 102, 105, 115, 104, 0];

fn main() {
    // let a = 42;
    // let b = &B;
    // let c = &C;

    // println!("a: {}, b: {:p}, c: {:p}", a, b, c);

    let a: usize = 42;
    let b: Box<[u8]> = Box::new(B);
    let c: &[u8; 11] = &C;

    println!("a (unsigned 整数):");
    println!("  地址: {:p}", &a);
    println!("  大小:    {:?} bytes", size_of::<usize>());
    println!("  值:  {:?}\n", a);

    println!("b (B 装在 Box 里):");
    println!("  地址:  {:p}", &b);
    println!("  大小:    {:?} bytes", size_of::<Box<[u8]>>());
    println!("  指向:  {:p}\n", b);

    println!("c (C 的引用):");
    println!("  地址:  {:p}", &c);
    println!("  大小:  {:?} bytes", size_of::<&[u8; 11]>());
    println!("  指向:  {:p}\n", c);

    println!("B (10 bytes 的数组):");
    println!("  地址:  {:p}", &B);
    println!("  大小:  {:?} bytes", size_of::<[u8; 10]>());
    println!("  值:  {:?}\n", B);

    println!("C (11 bytes 的数字):");
    println!("  地址:  {:p}", &C);
    println!("  大小:  {:?} bytes", size_of::<[u8; 11]>());
    println!("  值:  {:?}\n", C);
}

```

运行

```bash
point_demo on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ cargo run
   Compiling point_demo v0.1.0 (/Users/qiaopengjun/rust/point_demo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.19s
     Running `target/debug/point_demo`
a (unsigned 整数):
  地址: 0x16dda9a08
  大小:    8 bytes
  值:  42

b (B 装在 Box 里):
  地址:  0x16dda9a10
  大小:    16 bytes
  指向:  0x12b606ba0

c (C 的引用):
  地址:  0x16dda9a30
  大小:  8 bytes
  指向:  0x10208d7ba

B (10 bytes 的数组):
  地址:  0x10208d7b0
  大小:  10 bytes
  值:  [99, 97, 114, 114, 121, 116, 111, 119, 101, 108]

C (11 bytes 的数字):
  地址:  0x10208d7ba
  大小:  11 bytes
  值:  [116, 104, 97, 110, 107, 115, 102, 105, 115, 104, 0]


point_demo on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ 
```

- 对 B 和 C 中文本进行解码的例子
  - 它创建了一个与前图更加相似的内存地址布局

```go
use std::borrow::Cow;
use std::ffi::CStr;
use std::os::raw::c_char;

static B: [u8; 10] = [99, 97, 114, 114, 121, 116, 111, 119, 101, 108];
static C: [u8; 11] = [116, 104, 97, 110, 107, 115, 102, 105, 115, 104, 0];

fn main() {
  let a = 42;
  let b: String;
  let c: Cow<str>;
  
  unsafe {
    let b_ptr = &B as * const u8 as *mut u8;
    b = String::from_raw_parts(b_ptr, 10, 10);
    
    let c_ptr = &C as *const u8 as *const c_char;
    c = CStr::from_ptr(c_ptr).to_string_lossy();
  }
  println!("a: {}, b: {}, c: {}", a, b, c);
}
```

### Raw Pointers（原始指针）

- Raw Pointer （原始指针）是没有 Rust 标准保障的内存地址。
  - 这些本质上是 unsafe 的
- 语法：
  - 不可变 Raw Pointer：*const T
  - 可变的 Raw Pointer：*mut T
  - 注意：*const T，这三个标记放在一起表示的是一个类型
  - 例子：*const String
- *const T 与*mut T 之间的差异很小，相互可以自由转换
- Rust 的引用（&mut T 和 &T）会编译为原始指针
  - 这意味着无需冒险进入 unsafe 块，就可以获得原始指针的性能
- 例子：把引用转为原始指针

```rust
fn main() {
  let a: i64 = 42;
  let a_ptr = &a as *const i64;
  
  println!("a: {} ({:p})", a, a_ptr);
}
```

- 解引用（dereference）：通过指针从 RAM 内存提取数据的过程叫做对指针进行解引用（dereferencing a pointer）
- 例子：把引用转为原始指针

```rust
fn main() {
  let a: i64 = 42;
  let a_ptr = &a as *const i64;
  let a_addr: usize = unsafe {std::mem::transmute(a_ptr)};
  
  println!("a: {} ({:p}...0x{:x})", a, a_ptr, a_addr + 7);
}
```

### 关于 Raw Pointer 的提醒

- 在底层，引用（&T 和 &mutT）被实现为原始指针。但引用带有额外的保障，应该始终作为首选使用
- 访问 Raw Pointer 的值总是 unsafe 的
- Raw Pointer 不拥有值的所有权
  - 在访问时编译器不会检查数据的合法性
- 允许多个 Raw Pointer 指向同一数据
  - Rust 无法保证共享数据的合法性

### 使用 Raw Pointer 的情况

- 不可避免
  - 某些 OS 或 第三方库需要使用，例如与C交互
- 共享对某些内容的访问至关重要，运行时性能要求高

### Rust 指针生态

- Raw Pointer 是 unsafe 的
- Smart Pointer（智能指针）倾向于包装原始指针，附加更多的能力
  - 不仅仅是对内存地址解引用

### Rust 智能指针

| 名称         | 简介                                                         | 强项                                                | 弱项                                 |
| ------------ | ------------------------------------------------------------ | --------------------------------------------------- | ------------------------------------ |
| Raw Pointer  | *mut T 和*const T，自由基，闪电般块，极其 Unsafe            | 速度、与外界交互                                    | Unsafe                               |
| `Box<T>`     | 可把任何东西都放在Box里。可接受几乎任何类型的长期存储。新的安全编程时代的主力军。 | 将值集中存储在 Heap                                 | 大小增加                             |
| `Rc<T>`      | 是Rust的能干而吝啬的簿记员。它知道谁借了什么，何时借了什么   | 对值的共享访问                                      | 大小增加；运行时成本；线程不安全     |
| `Arc<T>`     | 是Rust的大使。它可以跨线程共享值，保证这些值不会相互干扰     | 对值的共享访问；线程安全                            | 大小增加；运行时成本                 |
| `Cell<T>`    | 变态专家，具有改变不可变值的能力                             | 内部可变性                                          | 大小增加；性能                       |
| `RefCell<T>` | 对不可变引用执行改变，但有代价                               | 内部可变性；可与仅接受不可变引用的Rc、Arc嵌套使用   | 大小增加；运行时成本；缺乏编译时保障 |
| `Cow<T>`     | 封闭并提供对借用数据的不可变访问，并在需要修改或所有权时延迟克隆数据 | 当只是只读访问时避免写入                            | 大小可能会增大                       |
| String       | 可处理可变长度的文本，展示了如何构建安全的抽象               | 动态按需增长；在运行时保证正确编码                  | 过度分配内存大小                     |
| `Vec<T>`     | 程序最常用的存储系统；它在创建和销毁值时保持数据有序         | 动态按需增长                                        | 过度分配内存大小                     |
| `RawVec<T>`  | 是`Vec<T>`和其它动态大小类型的基石；知道如何按需给你的数据提供一个家 | 动态按需增长；与内存分配器一起配合寻找空间          | 不直接适用于您的代码                 |
| `Unique<T>`  | 作为值的唯一所有者，可保证拥有完全控制权                     | 需要独占值的类型（如 String）的基础                 | 不适合直接用于应用程序代码           |
| `Shared<T>`  | 分享所有权很难，但他使生活更轻松                             | 共享所有权；可以将内存与T的宽度对齐，即使是空的时候 | 不适合直接用于应用程序代码           |

## 二、内存

### 值

- 值：类型 + 类型值域中的一个元素
  - 例如：true
- 通过它的类型表示，可以转化为字节序列
  - 6，u8 类型，数学整数；内存中：0x06
  - str "Hello World" 字符串域的值，它的表示是 UTF8编码
- 值的含义是独立于存储它字节的位置的

### 变量

- 值会存储到一个地方（这个地方可以容纳值）
- 可以在 Stack、Heap 或其它地方
- 最常见的存储值的地方就是变量，它是 Stack 上面的一个被命名的值的槽

### 指针

- 指针是一个值，值里面存放的是一块内存的地址，指针指向某个地方
- 指针可以被解引用（dereference）来访问它指向的内存里存放的值
- 可以把同一个指针存放在不同的变量里，也就是说多个变量可以间接的引用内存中的同一个地方，也就是相同的底层的值

例子

```rust
fn main() {
  let x = 42;
  let y = 43;
  let var1 = &x;
  let mut var2 = &x;
  var2 = &y;
  
  let s = "Hello World"; // 指针 执行变量第一个字符的位置
}
```

### 深入变量

- 高级模型：生命周期、借用等角度
- 低级模型：不安全代码、原始指针角度

### 变量的高级模型

- 变量就是给值的一个名称
- 当把值赋给一个变量的时候，这个值从那时起就由该变量命名了
  - let Dave = 1234;
- 当变量被访问的时候，可以从变量的上次访问到这次访问画一条线，从而在两次访问之间建立了依赖关系
- 如果变量被移动了，就不能从它那画线了

```rust
fn main() {
  let a = String::from("123");
  
  let b = a;
  
  println!("{}", b);
  // println!("{}", a);
}
```

- 在该模型里，变量只会在它持有合法值的时候才存在
- 如果变量的值未初始化，或者已经被移动了，那就无法从该变量画线了
- 使用该模型，整个程序会有许多依赖线组成，这些线叫做 flow
- 每个 flow 都在追踪一个值的特定实例的生命周期
- 当有分支存在时，flow 可以分叉或合并，每个分叉都在追踪该值的不同的生命周期
- 在程序中的任何给定点，编译器可以检查所有的 flow 是否可以互相兼容、并行存在：
  - 例如：一个值不可能有两个具有可变访问的并行 flow；也不能一个flow借用了一个值，但却没有 flow 拥有该值

```rust
fn main() {
  let mut a = 123;
  let b = &a;
  let c = &mut a; // 报错
  println!("{}", b);
  println!("{}", c);
}
```

### 变量的低级模型

- 变量会给哪些可能（不）存储合法值的内存地点进行命名
- 可以把变量想象为值的槽：当你赋值的时候，槽就装满了，而它里面原来的值（如果有的话）就被丢弃或替换了
- 当访问它时，编译器检查槽是不是空的；如果是空的，就说明变量未初始化，或者它的值被移动了
- 指向变量的指针，其实是指向变量的幕后内存，并通过解引用可以获得它的值
- 在本例中，我们忽略了CPU寄存器，并将其视为优化。实际上，如果变量不需要内存地址，编译器可以使用寄存器而不是内存区域来存放该变量

```rust
let dave = 123;
dave = 456;
```

### 内存区域

- 有许多内存区域，并不是都在 DRAM 上
- 三个比较重要的区域：Stack、heap、static 内存
- stack 和 heap：
  - stack 块
  - heap 慢

### stack 内存

- “有疑问时，首选 Stack”
  - 想把数据放在 Stack，编译器必须知道类型的大小
  - 换句话说：“有疑问时，使用实现了 Sized 的类型”
- Stack 是一段内存，程序把它作为一个暂存空间，用于函数调用
- 为什么叫 Stack？因为在Stack 上的条目是 LIFO（后进先出）

### Stack Frame

- 每个函数被调用，在 Stack 的顶部都会分配一个连续的内存块，它叫做 Stack Frame（栈帧）
- 接近 Stack 底部附近是 main 函数的 Frame，随着函数的调用，其余的 Frame 都推到了 Stack 上
- 函数的 frame 包含函数里所有的变量，以及函数所带的参数
- 当函数返回时，它的 frame 就被回收了
  - 构成函数本地变量值的那些字节不会立即擦除，但访问它们也是不安全的，因为它们可能被后续的函数调用所重写（如果后续函数调用的 frame 与回收的这个有重合的话）
  - 但即使没有被重写，它们也可能包含无法合法使用的值。例如函数返回后被移动的值

### Stack Pointer

- 随着程序的执行，CPU里有一个游标会随着更新，它会反映出当前 Stack frame 的当前地址，这个游标叫 stack pointer（stack 指针）
- 随着函数内不断的调用函数，stack 就会增长，而 stack pointer 的值会减少；当函数返回 stack pointer 的值会增加

### Stack Frame

- Stack Frame 也叫 activation frames 或 allocation record
- 每个 stack frame 的大小是不同的
- 在函数调用期间，Stack frame 会包含函数的状态。当一个函数在另外一个函数内调用时，原来的函数的值会被及时冻结
- stack frame 为函数参数，指向原来调用站的指针，以及本地变量（不包括在 heap 上分配的数据）提供空间
- Stack 的主要任务是为本地变量创造空间，为什么 stack 快？
  - 因为函数的所有变量在内存里都是紧挨着的

```rust
fn main() {
  let pw = "justok";
  let is_strong = is_strong(pw);
}

// &str -> Stack; String -> Heap

//fn is_strong(password: String) -> bool {
//  password.len() > 5
//}

fn is_strong<T: AsRef<str>>(password: T) -> bool {
  password.as_ref().len() > 5
}

fn is_strong<T: Into<String>>(password: T) -> bool {
  password.into().len() > 5
}
```

### Stack

- Stack Frames，它们最终也会消失这个事实，与Rust生命周期的概念是密切相关的。
- 任何在 stack 上的 frame 里存储的变量，在 frame 消失后，它就无法访问了
  - 所以任何到这些变量的引用的生命周期，最多只能与 frame 的生命周期一样长

### Heap 内存

- Heap 意味着混乱
- Heap 是一个内存池，并没有绑定到当前程序的调用栈
- Heap 是为在编译时没有已知大小的类型准备的
- 什么叫在编译时大小不已知？
  - 一些类型随着时间会变大或变小，例如 String、`Vec<T>`
  - 另一些类型的大小不会改变，但是无法告诉编译器需要分配多少内存
  - 这些都叫做动态大小的类型，例如 [T] (DST)
  - 另一个例子是 trait 对象，它允许程序员来模拟一些动态语言的特性：通过允许将多个类型放进一个容器
- Heap 允许你显示的分配连续的内存块。当这么做时，你会得到一个指针，它指向内存块开始的地方
- Heap 内存中的值会一直有效，直到你对它显示的释放
- 如果你想让值活得比当前函数 frame 的生命周期还长，就很有用
  - 如果值是函数的返回值，那么调用函数可以在它的 stack 上留一些空间给被调用函数让它把值在返回前写入进去

### Heap 线程安全的例子

- 但是如果想把值送到另一个线程，当前线程可能根本无法与那个线程共享 stack frames，你就可以把它存放在 heap上
- 因为函数返回时 heap 上的分配并不会消失，所以你在一个地方为值分配内存，把指向它的指针传给另一个线程，就可以让那个线程继续安全的操作于这个值。
- 换一种说法：当你分配 heap 内存时，结果指针会有一个无约束的生命周期，你的程序让它活多久都行。

### Heap 交互机制

- Heap 上面的变量必须通过指针访问（例子）

```rust
fn main() {
  let a: i32 = 40; // stack
  let b: Box<i32> = Box::new(60); // Heap
  
  //let result = a + b; // 报错
  
  let result = a + *b;
  
  println!("{} + {} = {}", a, b, a + *b);
}
```

- Rust 里与Heap交互的首要机制就是 Box类型
- 当 Box::new(value) 时，值就会放在 heap 上，返回的 (`Box<T>`) 就是指向 heap 上该值的指针。当 box 被丢弃时，内存就被释放
- 如果忘记释放 heap 内存，就会导致内存泄漏
- 有时你就想让内存泄漏：
  - 例如有一个只读的配置，整个程序都需要访问它。就可以把它分配在 heap 上，通过 Box::leak 得到一个 ’static 引用，从而显式的让其进行泄漏

```rust
use std::mem::drop;

fn main() {
  let a = Box::new(1);
  let b = Box::new(1);
  let c = Box::new(1);
  
  let result1 = *a + *b + *c;
  
  drop(a);
  let d = Box::new(1);
  let result2 = *b + *c + *d;
  
  println!("{} {}", result1, result2);
}
```

### Static 静态内存

- Static 内存实际是一个统称，它指的是程序编译后的文件中几个密切相关的区域
  - 当程序执行时，这些区域会自动加载到你程序的内存里
- Static 内存里的值在你的程序的整个执行期间会一直存活
- 程序的 Static 内存是包含程序二进制代码的（通常映射为只读的）
  - 随着程序的执行，它会在本文段的二进制代码中挨个指令进行遍历，而当函数被调用时就进行跳跃
- Static 内存会持有使用Static声明的变量的内存，也包括某些常量值，例如字符串

### ‘static

- ’static 是特殊的生命周期
  - 它的名字就是来自于 static 内存区，它将引用标记为只要 static 内存还存在（程序关闭前），那么引用就合法
- static 变量的内存在程序开始运行时就分配了，到 static 内存中变量的引用，按定义来说，就是 ‘static 的，因为程序关闭前它不会被释放
- 反过来却不行，可以有 ’static 的引用不指向 static 内存
- 但是名称仍然适用：
  - 一旦你创建了一个 ‘static 生命周期的引用，就程序的其余部分而言，它所指向的内容都可能在 static 内存中，因为程序想要使用它多久就可以使用多久

### static 内存

- 你可能会更多遇到 ’static 生命周期而不是 static 内存
- ‘static 经常出现在类型参数的 trait bounds 上
  - 例如：T: 'static，表示类型 T 可以存活我们想要的任何时长（直到程序关闭），同时这也要求 T 是拥有所有权的和自给自足的
    - 要么它不借用其他（非 static）值
    - 要么它借用的东西也都是 “static 的”
  - 因此将一直保留到程序结束

### const 与 static 区别

- const 关键字会把紧随它的东西声明为常量

```rust
const X: i32 = 123;
```

- 常量可在编译时完全计算出来
- 在编译期间，任何引用常量的代码会被替换为常量的计算结果值
- 常量没有内存或关联其它存储（因为它不是一个地方）
- 你可以把常量理解为某个特殊值的方便的名称

### 问题

- static 内存 和 Heap 内存分别在哪？（内存条）

### 动态内存分配

- 任何时刻，运行中的程序都需要一定数量的内存。
- 当程序需要更多内存时，就需要从OS请求。这是动态内存分配（dynamic allocation）。

### 动态内存分配步骤

1. 通过系统调用从 OS 请求内存
   1. UNIX 类：alloc()
   2. Windows：HeapAlloc()
2. 使用分配的内存
3. 将不再需要的内存释放给 OS
   1. UNIX 类：free()
   2. Windows：HeapFree()

- 程序和 OS 间有一个分配器：嵌入你程序幕后的专业子程序，会经常执行优化来避免 OS 和 CPU 内的大量工作

### 为什么 Stack 和 Heap 存在性能差异

- Stack 和 Heap 只是概念而已，内存在物理上不存在这两个区域
- 访问 Stack 快：
  - 函数本地变量（都分配在 stack 上）在 RAM 上都彼此相邻（连续布局）
  - 连续布局对缓冲友好
- 访问 Heap 慢：
  - 分配在 heap 上的变量不太可能彼此相邻
  - 访问 heap 上的数据涉及对指针进行解引用（页表查找和去访问主存）

### Stack VS Heap 简单粗暴对比

| Stack | Heap | 备注           |
| ----- | ---- | -------------- |
| 简单  | 复杂 |                |
| 安全  | 危险 | 指 Unsafe Rust |
| 快    | 慢   |                |
| 死板  | 灵活 |                |

- Stack 上的数据结构在生命周期内大小不能变
- Heap 上的数据结构更灵活，因为指针可以改变

```rust
use graphics::math::{add, mul_scalar, Vec2d};
use piston_window::*;
use rand::prelude::*;
use std::alloc::{GlobalAlloc, Layout, System};
use std::time::Instant;

use std::cell::Cell;

#[global_allocator]
static ALLOCATOR: ReportingAllocator = ReportingAllocator;

struct ReportingAllocator;

// Execute a closure without logging on allocations.
pub fn run_guarded<F>(f: F)
where
    F: FnOnce(),
{
    thread_local! {
        static GUARD: Cell<bool> = Cell::new(false);
    }

    GUARD.with(|guard| {
        if !guard.replace(true) {
            f();
            guard.set(false)
        }
    })
}

unsafe impl GlobalAlloc for ReportingAllocator {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8 {
        let start = Instant::now();
        let ptr = System.alloc(layout);
        let end = Instant::now();
        let time_taken = end - start;
        let bytes_requested = layout.size();

        // eprintln!("{}\t{}", bytes_requested, time_taken.as_nanos());
        run_guarded(|| {eprintln!("{}\t{}", bytes_requested, time_taken.as_nanos())});
        ptr
    }

    unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout) {
        System.dealloc(ptr, layout);
    }
}

struct World {
    current_turn: u64,
    particles: Vec<Box<Particle>>,
    height: f64,
    width: f64,
    rng: ThreadRng,
}

struct Particle {
    height: f64,
    width: f64,
    position: Vec2d<f64>,
    velocity: Vec2d<f64>,
    acceleration: Vec2d<f64>,
    color: [f32; 4],
}

impl Particle {
    fn new(world: &World) -> Particle {
        let mut rng = thread_rng();
        let x = rng.gen_range(0.0..=world.width);
        let y = world.height;
        let x_velocity = 0.0;
        let y_velocity = rng.gen_range(-2.0..0.0);
        let x_acceleration = 0.0;
        let y_acceleration = rng.gen_range(0.0..0.15);

        Particle {
            height: 4.0,
            width: 4.0,
            position: [x, y].into(),
            velocity: [x_velocity, y_velocity].into(),
            acceleration: [x_acceleration, y_acceleration].into(),
            color: [1.0, 1.0, 1.0, 0.99],
        }
    }

    fn update(&mut self) {
        self.velocity = add(self.velocity, self.acceleration);
        self.position = add(self.position, self.velocity);
        self.acceleration = mul_scalar(self.acceleration, 0.7);
        self.color[3] *= 0.995;
    }
}

impl World {
    fn new(width: f64, height: f64) -> World {
        World {
            current_turn: 0,
            particles: Vec::<Box<Particle>>::new(),
            height: height,
            width: width,
            rng: thread_rng(),
        }
    }

    fn add_shapes(&mut self, n: i32) {
        for _ in 0..n.abs() {
            let particle = Particle::new(&self);
            let boxed_particle = Box::new(particle);
            self.particles.push(boxed_particle);
        }
    }

    fn remove_shapes(&mut self, n: i32) {
        for _ in 0..n.abs() {
            let mut to_delete = None;

            let particle_iter = self.particles.iter().enumerate();

            for (i, particle) in particle_iter {
                if particle.color[3] < 0.02 {
                    to_delete = Some(i)
                }
                break;
            }

            if let Some(i) = to_delete {
                self.particles.remove(i);
            } else {
                self.particles.remove(0);
            };
        }
    }

    fn update(&mut self) {
        let n = self.rng.gen_range(-3..=3);

        if n > 0 {
            self.add_shapes(n);
        } else {
            self.remove_shapes(n);
        }

        self.particles.shrink_to_fit();
        for shape in &mut self.particles {
            shape.update();
        }
        self.current_turn += 1;
    }
}

fn main() {
    let (width, height) = (1280.0, 960.0);
    let mut window: PistonWindow = WindowSettings::new("particles", [width, height])
        .exit_on_esc(true)
        .build()
        .expect("Could not create a window.");

    let mut world = World::new(width, height);
    world.add_shapes(1000);

    while let Some(event) = window.next() {
        world.update();

        window.draw_2d(&event, |ctx, renderer, _device| {
            clear([0.15, 0.17, 0.17, 0.9], renderer);

            for s in &mut world.particles {
                let size = [s.position[0], s.position[1], s.width, s.height];
                rectangle(s.color, size, ctx.transform, renderer);
            }
        });
    }
}

```

Cargo.toml

```toml
[package]
name = "particles"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
piston2d-graphics = "0.43.0"
piston_window = "0.128.0"
rand = "0.8.5"

```

问题：运行Rust程序报错   51287 illegal hardware instruction  cargo run

```bash
➜ cargo run                
    Finished dev 【unoptimized + debuginfo】 target(s) in 0.18s
     Running `target/debug/particles`
【1】    29813 illegal hardware instruction  cargo run
```

解决：

<https://github.com/rust-in-action/code/pull/106>

<https://github.com/rust-in-action/code/commit/a0731bc66504fdd74f4d548059cb6ad2fb34539a>

运行

```bash
cargo run

cargo run -q 2> alloc.tsv
```

<https://jupyter.org/install>

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt 
from matplotlib import font_manager
plt.rcParams["font.sans-serif"]=["Songti SC"]
df = pd.read_csv('/Users/qiaopengjun/rust/particles/alloc.tsv', sep='\t', header=None)
df.head()
df.columns
df.info
plt.figure(figsize=(25, 7))

plt.scatter(df[0], df[1], s=12, facecolors='none', edgecolors='b')
plt.xlim(0, 20000)
plt.ylim(0, 20000)
plt.xlabel('分配内存大小(byte)')
plt.ylabel('分配持续时间(ns)')
plt.show()

```

### 虚拟内存

- 程序的内存视图，程序可访问的所有数据都是由操作系统在其地址空间中提供。
- 直觉上，程序的内存就是一系列字节，从开始位置 0 到结束位置 n
  - 例如：程序汇报使用了 100 KB 的 RAM，那么 n 就应该是在 100000 左右

### 例子

```rust
fn main() {
  let mut n_nonzero = 0;
  
  for i in 0..10000 { // 当 i = 0 None 指针
    let ptr = i as *const u8;
    let byte_at_addr = unsafe {*ptr};
    
    if byte_at_addr != 0 {
      n_nonzero += 1;
    }
  }
  
  println!("内存中的非 0 字节：{}", n_nonzero);
}
```

- segmentation fault：当 CPU 或 OS 检测到程序试图请求非法（无权访问）内存地址时，所产生的错误

修改之后

```rust
static GLOBAL: i32 = 1000;

fn noop() -> *const i32 {
  let noop_local = 12345;
  &noop_local as *const i32
}

fn main() {
  let local_str = "a";
  let local_int = 123;
  let boxed_str = Box::new('b');
  let boxed_int = Box::new(789);
  let fn_int = noop();
  
  println!("GLOBAL: {:p}"， &GLOBAL as *const i32);
  println!("local_str: {:p}", local_str as *const str);
  println!("local_int: {:p}", &local_int as *const i32);
  println!("boxed_int: {:p}", Box::into_raw(boxed_int));
  println!("boxed_str: {:p}", Box::into_raw(boxed_str));
  println!("fn_int: {:p}", fn_int);
}
```

- segment：虚拟内存中的块。虚拟内存被划分为块，以最小化虚拟和物理地址之间转换所需的空间

### 通过例子

- 某些内存地址是非法的：访问越界的内存，OS 就会关掉你程序
- 内存地址并不是随机的：看起来在地址空间内分布的比较广，值相当于是聚集在口袋内。

### 翻译虚拟地址到物理地址

- 程序里访问数据需要虚拟地址（程序只能范围虚拟地址）
- 虚拟地址会翻译成物理地址
  - 涉及程序、OS、CPU、RAM 硬件，有时涉及硬盘或其它设备
  - CPU 负责执行翻译，OS 负责存储指令
  - CPU 包含一个内存管理单元（MMU）负责这项工作
  - 这些指令也存在内存中一个预定义的地址中
- 最坏情况下，每次访问内存都会发生两次内存查找
- CPU 会维护一个最近转换地址的缓存
  - 它有自己的快速内存来加速内存的访问
  - 历史原因，该内存称为转换后备缓冲区（Translation Lookaside Buffer，TLB）
- 为提高性能，程序员需要保持数据结构精简，避免深度嵌套
  - 达到 TLB 的容量后（对于 x86 处理器，通常约为 100 页）可能成本高
- 页（page）：实际内存中固定大小的字块，64位系统通常是 4K
- 字（Word）：指针大小的任何类型，对应CPU寄存器的宽度
  - usize 和 isize 字长类型（word-length type）
- 虚拟地址被分成很多块，叫做页（page），通常 4KB 大小
  - 这就避免了需要为每个变量都存储转换映射
  - 页统一大小有助于避免内存碎片（可用 RAM 中出现空的、不可用的空间）
- 注意：这只是通用性指导，例如像微控制器情况就不同了。

### 数据在 RAM 中展示的指导建议

- 将程序热工作部分保持在 4KB 以内，从而保持快速查找
- 如果 4KB 不合理，那么下个目标是 4KB * 100
  - 意味着 CPU 可维护其转换缓存（TLB）来支持你的程序
- 避免深度嵌套数据结构（像意大利面）
  - 如果指针指向另一个页（page），性能会受到影响
- 测试嵌套循环的顺序：
  - CPU 会从 RAM 读取小块字节（cache line、缓存行）。在处理数组时，可以通过判断是按列操作还是按行操作来利用这一点

### 注意

- 虚拟化会让情况更糟，如果在虚拟机内运行应用程序，Hypervisor 还必须为其客户 OS 转换地址。
  - 这就是为什么许多 CPU 附带虚拟化支持，这可以减少额外的开销
- 在虚拟机中运行容器又添加一层间接，也增加了延迟
- 要想获得裸机的性能，就必须在裸机上运行程序

### 通过 OS 扫描地址空间（例子）

- OS 提供了接口可让程序发出请求：系统调用（system call）
- 在 Windows 里，KERNEL.DLL 提供了用于检查和操纵运行进程内存的功能
- 为什么以 Windows 为例？
  - 函数命名易于理解
  - 无需 POSIX API 知识

目的：在程序运行的时候扫描程序的内存

main.rs 文件

```rust
use kernel32;
use winapi;

use winapi::{
    DWORO, // Rust 里就是 u32

    HANDLE, // 各种内部 API 的指针类型，没有关联类型。
            // 在 Rust 里 std::os::raw::c_void 定义了 void 指针
    LPVOID, // Handle 是指向 Windows 内一些不透明资源的指针

    PVOID, // Windows 里，数据类型名的前缀通常是其类型的缩写
    SIZE_T, // 这台机器上 u64 是 usize
    LPSYSTEM_INFO, // 到 SYSTEM_INFO struct 的指针

    MEMORY_BASIC_INFORMATION as MEMINFO, // Windows 内部定义的一些 Struct
    SYSTEM_INFO,
};

fn main() {
    // 这些变量将在 unsafe 块进行初始化
    let this_pid: DWORO;
    let this_proc: HANDLE;
    let min_addr: LPVOID;
    let max_addr: LPVOID;
    let mut base_addr: PVOID;
    let mut proc_info: SYSTEM_INFO;
    let mut mem_info: MEMINFO;

    const MEMINFO_SIZE: usize = std::mem::size_of::<MEMINFO>();

    // 保证所有的内存都初始化了
    unsafe {
        base_addr = std::mem::zeroed();
        proc_info = std::mem::zeroed();
        mem_info = std::mem::zeroed();
    }

    // 系统调用
    unsafe {
        this_pid = kernel32::GetCurrentProcessId();
        this_proc = kernel32::GetCurrentProcess();
        // 下面代码使用 C 的方式将结果返回给调用者。
        // 提供一个到预定义 Struct 的指针，一旦函数返回就读取 Struct 的新值
        kernel32::GetSystemInfo(&mut proc_info as LPSYSTEM_INFO);
    };

    // 对变量重命名
    min_addr = proc_info.lpMinimumApplicationAddress;
    max_addr = proc_info.lpMaximumApplicationAddress;

    println!("{:?} @ {:p}", this_pid, this_proc);
    println!("{:?}", proc_info);
    println!("min: {:p}, max: {:p}", min_addr, max_addr);

    // 扫描地址空间
    loop {
        let rc: SIZE_T = unsafe {
            // 提供运行程序内存地址空间特定段的信息，从 base_addr 开始
            kernel32::VirtualQueryEx(this_proc, base_addr, &mut mem_info, MEMINFO_SIZE as usize)
        };

        if rc == 0 {
            break;
        }

        println!("{:#?}", mem_info);
        base_addr = ((base_addr as u64) + mem_info.RegionSize) as PVOID;
    }
}

```

Cargo.toml 文件

```toml
[package]
name = "tlearn"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
winapi = "0.2" # 定义一些有用的类型别名，对所有 Windows API 的原始的 FFI 绑定
kernel32-sys = "0.2" # 提供与 KERNEL.DLL 的交互，包含 Windows API 库 Kernel32 的函数定义

```

### 读取和写入进程内存的步骤

```rust
let pid = some_process_id;

OpenProcess(pid);

loop 地址空间 {
  调用 VirtualQueryEx() 来访问下个内存块
  
  通过调用 ReadProcessMemory()，来扫描内存块
  寻找某种特定的模式
  
  使用所需的值调用 WriteProcessMemory()
}
```

- Linux 提供了简单的 API：process_vm_readv()，process_vm_writev()
- Windows：ReadProcessMemory()，WriteProcessMemory()

## 三、所有权（简单回顾）

### 所有权

- Rust 内存模型的核心思想：所有的值都只有一个所有者
  - 只有一个位置（通常是作用域）来负责释放每个值
  - 这是通过借用检查器来实现的
    - 如果值移动了（赋值新变量、推到 Vec、置于 Heap），其所有者也变成新的位置了
- 有些类型不遵守这个规则
  - 如果值的类型实现了 Copy trait：重新赋值时到新内存地址时发生的事复制，而不是移动。
    - 大多数原始类型，例如整数、浮点类型等，都实现了Copy

### 如何实现 Copy

- 必须可以按位（bit）来复制值。
  - 这自然就不包括：
    - 含有 non-Copy 类型的类型
    - 拥有这类资源的类型：当类型的值被丢弃时，必须被释放该资源。
- 为什么？
  - 如果 Box 是 Copy 的，进行赋值：box1 = box2...

### 值的删除（丢弃，dropped）

- 当值不再被需要，其所有者会将其删除
  - 当值不再存在于作用域内
- 类型会递归的将其所包含的值进行删除
  - 删除一个复杂类型的变量，可能会导致很多值被删除
- Rust 不会发生多次删除同一个值的情况（因为所有权）
- 变量若含有对其它值（B）的引用（不拥有该值），当变量被删除时，其它的值（B）不会被删除

```rust
fn main() {
  let x1 = 42;
  let y1 = Box::new(84);
  
  {
    let z = (x1, y1); // 删除的时候先删除 x1 再删除 y1
  }
  
  let x2 = x1;
  
  //let y2 = y1; // 报错
}
```

### 值删除的顺序

- 变量（包括函数的参数）按相反的顺序进行删除
- 嵌套的值按照源代码的顺序进行删除
- 备注：Rust 暂时不允许在单个值内进行自我引用

## 四、引用及内部可变性（回顾）

### 引用

- 通过引用，Rust 允许将值借用出去，但不放弃所有权
- 引用就是带有附加合约的指针

### 共享引用

- &T，就是可以共享的指针：
  - 可同时存在任意数量的引用指向同一个值
  - 每个共享的引用都实现了 Copy
- 背后的值不可变
  - 编译器允许假定共享引用指向的值，在该引用存活期间是不会改变的
  - 例如：一个共享引用的值在某函数内被多次读取，那编译器就有权让其只读取一次，然后重用读取的值。

### 可变引用

- &mut T
- 可变引用是独占的
  - 编译器假定没有其它线程访问目标值（无论是通过共享引用还是可变引用）

```rust
fn main() {
  println!("Hello, world!");
}

fn noalias(input: &i32, output: &mut i32) {
  if *input == 1 {
    *output = 2;
  }
  if  *input !=1 {
    *output = 3;
  }
}
```

- 可变引用只允许你修改引用所指向的内存地址

```rust
fn main() {
  let x = 42;
  let mut y = &x; // y is of type &i32
  let z = &mut y; // z is of type &mut &i32
}
```

### 拥有值 VS 拥有到值的可变引用

- 所有者需要对删除值（丢弃值）负责
- 警告：如果你移动了可变引用背后的值，则必须在其位置上留下另一个值。如果不这样做，所有者会认为他需要将其删除（丢弃），但其实却没有值可以删除了。

```rust
fn main() {
  let mut s = Box::new(42);
  replace_with_84(&mut s);
}

fn replace_with_84(s: &mut Box<i32>) {
  // this is not okay, as *s would be empty;
  // let was = *s;
  // but this is:
  let was = std::mem::take(s);
  // so is this:
  *s = was;
  // we can exchange values behind &mut:
  let mut r = Box::new(84);
  std::mem::swap(s, &mut r);
  assert_ne!(*r, 84);
}
```

### 内部可变性

- 一些类型提供了内部可变性：
  - 可通过共享引用修改值
- 这些类型通常依赖于额外的机制（如原子 CPU 指令）或不变量来提供安全的可变性，而不依赖于独占引用的语义
- 分为两类：
  - 通过共享引用获得可变引用
  - 通过共享引用可以替换值
- 通过共享引用获得可变引用：Mutex、RefCell
  - 提供保障机制：针对任何提供了可变引用指向的值，同时只会存在一个可变引用（没有共享引用）
  - 依赖于 UnsafeCell 类型，通过共享引用修改值的唯一正确方式
- 通过共享引用可以替换值：std::sync::atomic、std::cell::Cell
  - 没有提供可变引用到内部值
  - 提供就地操作值的方法
  - 例：无法获得到 usize 或 i32 的直接引用，但是可以读取和替换值

### Cell 类型

- 标准库
- 通过不变量实现的内部可变性
  - 无法跨线程共享（内部值不会被并发的修改，即使通过共享引用发生修改）
  - 不会提供到 Cell 内部的值的引用（所以可以一直移动它）
- 提供的方法：
  - 对值整体替换
  - 返回值的副本

## 五、生命周期

### 生命周期（Lifetime）

- 官方教程中提到：
  - Rust 里每个引用都有生命周期，它就是引用保持合法作用域（scope），大多数时候是隐式和推断出来的
- 对某个变量取得引用时生命周期开始，当变量移动或离开作用域时生命周期结束
- 生命周期：对于某个引用来说，它必须保持合法的一个代码区域的名称
  - 生命周期通常与作用域重合，但也不一定

### 借用检查器（Borrow Checker）

- 每当具有某个生命周期 ‘a 的引用被使用，借用检查器都会检查 ’a 是否还存活
  - 追踪路径直到 ‘a 开始（获得引用）的地方
  - 从这开始，检查沿着路径是否存在冲突
  - 保证引用指向一个可安全访问的值

```rust
use rand::prelude::*;

fn main() {
  let mut x = Box::new(42);
  let r = &x; // (1) 'a
  if random::<f32>() > 0.5 {
    *x = 84; // (2)
  } else {
    println!("{}", r); // (3) 'a
  }
}
// (4)
```

- 生命周期很复杂：
- 生命周期有“漏洞”

```rust
fn main() {
  let mut x = Box::new(42);
  
  let mut x = &x; // (1) a'
  for i in 0..100 {
    println!("{}", z); // (2) a'
    x = Box::new(i); // (3) 
    z = &x; // (4) a'
  }
  println!("{}", z); // a'
}
```

- 借用检查器是保守的：
  - 如果不确定某个借用是否合法，借用检查器就会拒绝该借用
- 借用检查器有时需要帮助来理解某个借用 为什么是合法的：
  - 这也是 Unsafe Rust 存在的部分原因

### 泛型生命周期

- 有时需要在自己的类型里存储引用：
  - 这些引用都有生命周期，以便借用检查器检查合法性
  - 例如：在该类型方法中返回引用，且存活比 self 的长
- Rust 允许你基于一个或多个生命周期将类型的定义泛型化

### 泛型生命周期 - 两点提醒

- 如果类型实现了 Drop，那么丢弃类型时，就被记作是使用了类型所泛型的生命周期或类型
  - 类型实例要被 drop 了，在 drop 之前，借用检查器会检查看是否仍然合法去使用你类型的泛型生命周期，因为你在 drop 里的代码可能会用到这些引用
- 如果类型没有实现 Drop，那么丢弃类型的时候不会当作使用，可以忽略类型内的引用
- 类型可泛型多个生命周期，但通常会不必要的让类型签名更复杂：
  - 只有类型包含多个引用时，你才应该使用多个生命周期参数
  - 并且它方法返回的引用只应绑定到其中一个引用的生命周期

```rust
strurc StrSplit<'s, 'p> {
  delimiter: &'p str,
  document: &'s str,
}

impl<'s, 'p> Iterator for StrSplit<'s, 'p> {
  type Item = &'s str;
  
  fn next(&mut self) -> Option<Self::Item> {
    todo!()
  }
}

fn str_before(s: &str, c: char) -> Option<&str> {
  StrSplit {
    document: s,
    delimiter: &c.to_string(),
  }
  .next()
}
```

### 生命周期 Variance

- Variance：
  - 哪些类型是其它类型的“子类”
  - 什么时候“子类”可以替换“超类”（反之亦然）
- 通常来说：
  - 如果 A 是 B 的子类，那么 A 至少和 B 一样有用
  - Rust 的例子：
    - 如果函数接收 &'a str 的参数，那么就可以传入 &'statci str 的参数
    - 'static 是 'a 的 “子类”，因为 'static 至少跟任何 'a 存活的一样长

### 生命周期 三种 Variance

- 所有的类型都有 Variance：
  - 定义了哪些类似类型可以用在该类型的位置上
- 三种 Variance：
  - covariant：某类型只能用“子类型”替代
    - 例如：&'static T 可替代 &'a T
  - invariant：必须提供指定的类型
    - 例如：&mut T，对于 T 来说就是 invariant 的
  - contravariant：函数对参数的要求越低，参数可发挥的作用越大

```rust
// let x1: &'static str;  // more useful, lives longer
// let x2: &'a str;  // less useful, lives shorter

// fn take_func1(&'static str1)  // stricter, so less useful
// fn take_func2(&'a str2)  // less strict, more useful
```

### 多生命周期与 Variance

```rust
struct MuStr<'a, 'b> {
  s: &'a mut &'b str,
}

fn main() {
  let mut r = "hello";  // &'static str => &'a str
  *MutStr {s: &mut r}.s = "world";
  println!("{}", r);
}
```

## 六、内存中的类型

### 内存中的类型

- 每个 Rust 值都有类型：
  - 一个基本职责：告诉你如何解释内存中的 bits
  - 例如：0b10111101 这串 bit 本身不代表什么
    - 按 u8 类型解释：数字 189
    - 按 i8 类型解释：数字 -67
- 当自定义类型时：编译器决定该类型的各部分在内存表示中的位置

### 对齐（Alignment）

- 对齐（Alignment）：决定了类型的字节可以被存在哪
- 实际上，计算机硬件对给定的类型可以存放的位置是有约束的
  - 例如：指针指向字节（bytes）而不是位（bits）
    - 如果将某类型T 的值放在计算机内存中的索引为 4 的位（bit）上，那就无法引用它的地址。你只能创建一个指针指向 byte 0 或者 byte 1
- 所有的值（无论什么类型），都必须开始于 byte 的边界
  - 必须至少是字节对齐（byte-aligned）
  - 存放的地址必须是 8 bits 的倍数

### 更严格的对齐规则

- 一些类型的对齐规则比字节对齐还严格：
  - 在 CPU 和内存系统里，内存经常按大于单个 byte 的块进行访问
    - 例如：在64位CPU上，大部分的值是按 8 bytes 的块进行访问的，每个操作都开始于 “8 bytes 对齐” 的地址上。（这也叫做 CPU 的字长，word size）
    - CPU 有办法处理更小值的读写，以及跨域边界的值
- 应该尽可能的保证硬件可以操作于它的“原生（native）”对齐
  - 例如：想读取的 i64 的值开始于 8 bytes 块的中间...

### 没对齐的操作

- 对数据进行没对齐操作叫做 “misaligned access"：
  - 可导致 性能低 和 并发问题
- 很多 CPU 操作多要求或强烈建议：它们的参数是自然对齐的（naturally aligned）
- 自然对齐的值：对齐匹配值的大小
  - 例如：加载 8 bytes，那么提供的地址就需要 8 bytes 对齐

### 编译器会尽可能利用对齐

- 基于类型包含的内容，编译器通过计算，为类型分配一个对齐（安排它如何对齐）
- 内置值，通常对齐到他们的大小
  - u8 - byte 对齐；u16 - 2 bytes 对齐；u32 - 4 bytes 对齐； u64 - 8 bytes 对齐
- 复杂类型（包含其它类型的类似），通常被赋予所含类型的最大对齐：
  - 例：某类型含有 u8、u16、u32，那么该类型就应该是 4 bytes 对齐

### 布局（Layout）

- 类型的布局（Layout）：（编译器如何决定）类型的内存中表示
- Rust 编译器对于类型如何布局，并没有给出多少保证
- Rust 提供了 repr 属性（attribute）：
  - 可以添加到你类型的定义上，来请求特定内存表示

### repr(C)

- 最常见的一个是 repr(C)：布局方式与 C、C++ 编译器对同类型的布局兼容
  - 对于使用 FFI 与其它语言交互的 Rust 代码很有用
  - Rust 会生成一个匹配其他语言编译器期望的布局
- 因为 C 布局可预测、不易改变
  - 在 unsafe 上下文中有用：
    - 使用指向该类型的原始指针
    - 在两个具有相同字段的类型间进行转换

### repr(transparent)

- repr(transparent)：仅能用于只含有单个字段的类型，它保证了外层类型的布局与内层类型一样
  - 这与 ”newtype“ 模式结合很好用
    - 例如：你想操作于 struct A 和 struct New(A) 内存表示，就如同他们是一样的
    - newtype 模式：在 tuple struct 里创建一个新的类型，只有一个字段，针对某型的薄封装

### 例子

| 代码         | 字段类型的大小 | 默认内存表示 | 填充    | 最终对齐               |
| ------------ | -------------- | ------------ | ------- | ---------------------- |
| #[repr(C)]   |                |              |         |                        |
| struct Foo { |                |              |         |                        |
| tiny: bool,  | 1 bit          | 1 byte 对齐  | 3 bytes | 4bytes                 |
| normal: u32, | 4 bytes        | 4 bytes 对齐 |         | 4 bytes 与上共 8 bytes |
| small: u8,   | 1 byte         | 1 byte 对齐  | 7 bytes | 8 bytes                |
| long: u64,   | 8 bytes        | 8 bytes 对齐 |         | 8 bytes                |
| short: u16,  | 2 bytes        | 2 bytes 对齐 | 6 bytes | 8 bytes                |
| }            |                |              | 一共    | 32 bytes               |

### repr(Rust)

- C 表示的限制：需要将所有字段按原 struct 定义的顺序放置
- repr(Rust) 去掉了该限制以及其它几个较小的限制：
  - 对恰好具有相同字段的类型的确定性字段排序
    - 两个不同类型的字段相同，字段类型也相同，定义顺序也一样
    - 在 Rust里，如果使用 repr(Rust)，就不能保证这两种类型的布局是一样的
- repr(Rust) 允许重新对字段排序：
  - 可按大小递减的顺序排列（对于 Foo 例子来说就不需要填充了）
- 对布局的保证少了，编译器有余地进行重新安排，产生高效代码

### 例子

| 代码         | 字段类型的大小 | 默认内存表示 | 填充 | 最终对齐                |
| ------------ | -------------- | ------------ | ---- | ----------------------- |
| #[repr(C)]   |                |              |      |                         |
| struct Foo { |                |              |      |                         |
| long: u64,   | 8 bytes        | 8 bytes 对齐 |      | 8 bytes                 |
| normal: u32, | 4 bytes        | 4 bytes 对齐 |      |                         |
| short: u16,  | 2 bytes        | 2 bytes 对齐 |      |                         |
| small: u8,   | 1 byte         | 1 byte 对齐  |      |                         |
| tiny: bool,  | 1 bit          | 1 byte 对齐  |      | 以上3个字段一共 8 bytes |
| }            |                |              | 一共 | 16 bytes                |

### 无填充布局

- 可以告诉编译器字段之间无需任何填充：
  - 承担不对齐访问的性能损失
  - 使用场景举例：
    - 内存有限，类型实例较多
    - 通常低带宽网络连接发送内存表示
- 启用该功能：在你类型上添加 #[repr(packed)] 注解
- 注意：
  - 可能会导致代码运行速度慢；
  - 极端情况下，如果 CPU 仅支持对齐操作，可导致程序崩溃

### 给特定字段或类型更大的对齐

- 使用 #[repr(align(n))]
  - 例如：保证在内存中连续（相邻）存储（就像数组）的不同值最终位于 CPU 上不同的缓存行上，就可以避免伪共享（false sharing）
    - 缓存是由缓存行组成的，缓存都是以缓存行作为一个单位来处理的；缓存行（cache line）是可以映射到缓存中的最小数据部分
    - 伪共享（false sharing）：两个不同的 CPU 访问共享同一个缓存行的不同变量时，就发生了伪共享。理论上它们可以并行操作，但最终它们都争相更新缓存中的同一个条目。它可导致并发类程序中的巨大性能降级。

### 复杂类型的内存表示

- 元组（Tuple）：就像 struct，其字段类型与元组元素类型按顺序相同
- 数组：所包含类型的连续序列，元素间没有填充
- Union：对于每个变体来说，其布局的选择是独立的；对齐就是所有变体里最大的那个。
- 枚举：和 Union 一样，额外有一个隐藏的共享字段，用于存储枚举变体的鉴别符
  - 代码用鉴别符的值来判定给定值所含的是哪个变体
  - 鉴别符的大小取决于变体的数量

### 动态大小的类型和宽指针

- Rust中大多数类型自动实现了 Sized：
  - 它的大小在编译时就已知了
- 两个常见的类型除外：Trait 对象、切片（slice）
  - 例如：dyn Iterator、[u8]等（DST：dynamically sized types）的大小在运行时才能确定
- 问题：通常编译器需要知道东西的大小来产生合理的代码：
  - 例如为 tuple 分配多大空间，或者访问第四个元素时需要多少偏移量
  - 如果类型不是 Sized，上述信息就无法获得

### 编译器需要 Sized 类型

- 几乎在所有地方，编译器都需要 Sized 类型：
  - struct 字段、函数参数、返回值、变量、数组类型都必须 Sized
  - 自定义的 Type Bound 自动包含 T: Sized，除非你写明 T:?Sized
- 但例如当函数需要接收 DST（例如：trait 对象、切片 等）作为参数时怎么办？
  - 可以使用宽指针（wide pointer 或叫 fat pointer）

### 宽指针（Wide Pointer）

- 通过将非 Sized 类型放在宽指针后边，就可弥补 Sized 和非 Sized 类型 间的差距
- 宽指针：
  - 就是普通指针
  - 附加了一个”字大小（word-sized）“的字段
    - 它可以提供给编译器所需要的关于指针的额外信息：生产使用该指针的合理代码
- 当引用 DST 时，编译器自动为你组建一个宽指针
- 例如：
  - 切片 slice，它的附加信息就是切片的长度
  - Trait 对象，以后再说
- 宽指针是 Sized：是usize 的两倍大小
  - usize 就是所在目标平台上一个字（word）的大小
  - 一个 usize 持有指针
  - 另外的 usize 持有附件信息，用于“完善”该类型
- 备注：Box 和 Arc 都支持存储宽指针，所以它们都支持 T:?Sized

## 七、Trait（Bound）的编译与分派

### 静态分派（static dispatch）

- 编译泛型代码或者调用 dyn Trait 上的方法时发生了什么？
- 当编写关于泛型 T 的类型或函数时：
  - 编译器会针对每个 T （的类型），都将类型或函数复制一份
  - 当你构建 `Vec<i32>` 或 `HashMap<String, bool>` 的时候：
    - 编译器会复制它的泛型类型以及所有的实现块
      - 例如：`Vec<i32>`，就是对Vec做一个完整的复制，所有遇到的 T 都换成 i32
    - 并把每个实例的泛型参数使用具体类型替换
- 注意：编译器其实不会做完整的复制粘贴，它只复制你用的代码

```rust
impl String {
  pub fn contains(&self, p: impl Pattern) -> bool {
    f.is_contained_in(self)
  }
}
```

- 针对不同的 Pattern 类型，该方法都会复制一遍，为什么？
  - 因为我们需要知道 is_contained_in 方法的地址，以便进行调用。CPU 需要知道在哪跳转和继续执行
  - 对于任何给定的 Pattern，编译器知道那个地址是 Pattern 类型实现 Trait方法的地址
  - 不存在一个可给任意类型用的通用地址
- 需要为每个类型复制一份（方法体），每份都有自己的地址，可用来跳转。
- 这就是静态分派（static dispatch）：
  - 因为对于方法的任何给定副本，我们“分派到”的地址都是静态已知的
- 静态（static）：就是指编译时已知的事务（或可被视为此的）。

### 单态化（monomorphization)

- 从一个泛型类型到多个泛型类型的过程叫做单态化
- 当编译器开始优化代码时，就好像根本没有泛型！
  - 每个实例都是单独优化的，具有了所有的已知类型
  - 所以 is_contained_in 方法调用的执行效率就如同 Trait 不存在一样
  - 编译器对设计的类型完全掌握，甚至可以将它进行 inline 实现

### 单态化的代价

- 所有的实例需要单独编译，编译时间增加（如果不能优化编译）
- 每个单态化的函数会有自己的一段机器码，让程序更大
- 指令在泛型方法的不同实例间无法共享，CPU 的指令缓存效率降低，因为它需要持有相同指令的多个不同副本

### 动态分派（dynamic dispatch）

- 动态分派：使代码可以调用泛型类型上的 trait 方法，而无需知道具体的类型

```rust
impl String {
  pub fn contains(&self, p: &dyn Pattern) -> bool {
    p.is_contained_in(&*self)
  }
}
```

- 调用者只需提供两个信息：
  - Pattern 的地址
  - is_contained_in 的地址

问题：为什么在 dyn 前面加 &？

### vtable

- 实际上，调用者会提供指向一块内存的指针，它叫做虚方法表（virtual method table）或叫 vtable
  - 它持上例该类型所有的 trait 方法实现的地址
    - 其中一个就是 is_contained_in
- 当代码想调用提供类型的一个 trait 方法时，就会从 vtable 查询 is_contained_in 方法的实现地址，并调用
  - 这允许我们使用相同的函数体，而不关心调用者想要使用的类型
- 每个 vtable 还包含具体类型的布局和对齐信息（总是需要这些）

### 对象安全（Object-Safe）

- 类型实现了一个 Trait 和它的 vtable 的组合就形成了一个 trait object （trait 对象）
- 大部分 trait 可转为 trait object，但不是所有：
  - 例如 Clone trait 就不行（它的 clone 方法返回 Self），Extend trait 也不行
  - 这些例子就不是 对象安全的（object-safe）
- 对象安全的要求：
  - trait 所有的方法都不能是泛型的，也不可以使用 Self
  - trait 不可拥有静态方法（无法知道在哪个实例上调用的方法）

### Self: Sized

- Self: Sized 意味着 Self 无法用于 trait object（因为它是 !Sized）
- 将 Self: Sized 用在某个 trait，就是要求它永远不使用动态分派
- 也可以将 Self: Sized 用在特定方法上，这时当 trait 通过 trait object 访问的时候，该方法就不可用了
- 当检查 trait 是否对象安全的时候，使用了 where Self: Sized 的方法就会被免除

### 动态分派

- 优点
  - 编译时间减少
  - 提升 CPU 指令缓存效率
- 缺点
  - 编译器无法对特定类型优化
    - 只能通过 vtable 调用函数
  - 直接调用方法的开销增加
    - trait object 上的每次方法调用都需要查 vtable

### 如何选择（一般而言）

- 静态分派
  - 在 library 中使用静态分派
    - 无法知道用户的需求
    - 如果使用动态分派，用户也只能如此
    - 如果使用静态分派，用户可自行选择
- 动态分派
  - 在 binary 中使用动态分派
    - binary 是最终代码
    - 动态分派使代码更整洁（省去了泛型参数）
    - 编译更快
    - 以边际性能为代价

## 八、泛型 Trait

- 两种：
  - 泛型类型参数：`trait Foo<T>`
  - 关联类型：`trait Foo {type Bar;}`
- 区别：
  - 使用关联类型：对于指定类型的 trait 只有一个实现
  - 使用泛型类型参数：多个实现
- 建议（简单来说）：
  - 可以的话尽量使用 关联类型

### 泛型（类型参数）Trait

- 必须指定所有的泛型类型参数，并重复写这些参数的 Bound
  - 维护较难
    - 如果添加泛型类型参数到某个 Trait，该 Trait 的所有用户必须都进行更新代码
- 针对给定类型，一个 Trait 可存在多重实现
  - 缺点：对于你想要用的是 Trait 的哪个实例，编译器决定起来更困难了
    - 不得不调用类似这样的 `FromIterator::<u32>::from_iter` 可消除歧义的函数
  - 也是优点：
    - `impl PartialEq<BookFormat> for Book`
    - 实现 `FromIterator<T>` 和 `FromIterator<&T> where T:Clone`

### 关联类型 Trait

```rust
trait contains {
  type A;
  type B;
  
  // Updated syntax to refer to these new types generically.
  fn contains(&self, _: &self::A, _: &self::B) -> bool;
}
```

- 使用关联类型：
  - 编译器只需要知道实现 Trait 的类型
  - Bound 可完全位于 Trait 本身，不必重复使用
  - 未来再添加 关联类型 也不影响用户使用
  - 具体的类型会决定 Trait 内关联类型的类型，无需使用消除歧义的函数

```rust
impl Contains for Container {
  type A = i32;
  type B = i32;
  
  fn contains(&self, number_1: &i32, number_2: &i32) -> bool {
    (&self.0 == number_1) && (&self.1 == number_2)
  }
  
  fn first(&self) -> i32 { self.0 }
  
  fn last(&self) -> i32 { self.1 }
}
```

- 但是：
  - 不可以多个 Target 类型来实现 Deref
  - 不可以使用多个 Item 来实现 Iterator

```rust
pub trait Deref {
  type Target: ?Sized;
  
  fn deref(&self) -> &Self::Target;
}

pub trait Iterator {
  type Item;
}
```

## 九、孤儿规则与连贯性/一致性

### 连贯性/一致性 属性

- 定义：对于给定的类型和方法，只会有一个正确的选择，用于该方法对该类型的实现
- 孤儿规则（orphan rule）：
  - 只要 trait 或者类型在你本地的 crate，那就可以为该类型实现该 trait
    - 可以为你的类型实现 Debug；可以为bool 实现 MyTrait
    - 不能为 bool 实现 Debug
  - 注意：也有其他注意事项、例外。

### Blanket Implementation

- `impl<T> MyTrait for T where T:`
  - 例如： `impl<T: Display> ToString for T {}`
- 不局限于一个特定的类型，而是应用于更广泛的类型
- 只有定义 trait 的 crate 允许使用 Blanket Implementation
- 添加 Blanket Implementation 到现有 trait 属于破坏性变化

### 基础类型

- 有些类型太基础了，需要允许任何人在它们上实现 trait （即使违反孤儿规则）
- 这些类型被标记了 #[fundamental]，目前包括 &、&mut 和 Box
  - 出于孤儿规则的目的，在孤儿规则检查前，它们就会被抹除
- 对于基础类型使用 blanket implementation 也被认为是破坏性变化

### Covered Implementation

- 有时需要为外部类型实现外部 trait
  - 例如：`impl From<MyType> for Vec<i32>`
- 孤儿规则制定了一个狭窄的豁免：
  - 允许在非常特定的情况下为外来类型实现外来 trait
- `impl<PI..=Pn> Foreign Trait<TI..=Tn> for T0` 只在以下条件被允许：
  - 至少有一个 Ti 是本地类型
  - 没有 T 在第一个这样的 Ti 前（T 是指泛型类型 PI..=Pn 中的一个）
  - 泛型类型参数 Ps 允许出现在 T0..Ti，只要它们被某种中间（intermediate）类型所 cover
- 如果 T 作为其他类型（例 `Vec<T>`）的类型参数出现，那就说 T 被 cover 了
- 而 T 只作为本身，或者位于基础类型后（例 &T），就不是 Cover
- OK

```rust
impl<T> From<T> for MyType

impl<T> From<T> for MyType<T>

impl<T> From<MyType> for Vec<T>

impl<T> Foreign Trait<MyType, T> for Vec<T>
```

- Not OK

```rust
impl<T> Foreign Trait for T

impl<T> From<T> for T

impl<T> From<Vec<T>> for T

impl<T> From<MyType<T>> for T

impl<T> From<T> for Vec<T>

impl<T> Foreign Trait<T, MyType> for Vec<T>
```

- 是否是破坏性变化：
  - 为现有 trait 添加新的实现，且至少包含一个新的本地类型，该本地类型满足豁免条件，这就是非破坏性的变化
  - 为现有 trait 添加的实现不满足上述要求，就是破坏性变化
- 注意：
  - `impl<T> ForeignTrait<LocalType, T> for Foreign Type`，是合法的
  - `impl<T> ForeignTrait<T, LocalType> for Foreign Type`，是非法的

### 关于类型的其他知识

- Trait Bound 的各种高级写法
- Marker Trait，Marker Type
- Existential Type （存在类型）
- ... ...
