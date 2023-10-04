---
title: "Rust语言 - 接口设计的建议之显而易见（Obvious）"
date: 2023-06-19T15:27:11+08:00
draft: true
tags: ["Rust"]
categories: ["Rust"]
---

# Rust语言 - 接口设计的建议之显而易见（Obvious）

- [Rust API 指南 GitHub](https://github.com/rust-lang/api-guidelines)：<https://github.com/rust-lang/api-guidelines>
- [Rust API 指南 中文](https://rust-chinese-translation.github.io/api-guidelines/)：<https://rust-chinese-translation.github.io/api-guidelines/>
- [Rust API 指南](https://rust-lang.github.io/api-guidelines/)：<https://rust-lang.github.io/api-guidelines/>

## 显而易见（Obvious）

### 文档与类型系统

- 用户可能不会完全理解接口的所有规则和限制
  - 重要：让用户容易理解接口，并难以用错

### 文档

- 接口透明化的第一步：写出好的文档

1. 清楚的记录：
   1. 可能出现意外的情况，或它依赖于用户执行超出类型签名要求的操作
   2. 例：panic、返回错误、unsafe 函数...

如果你的代码可能会发生恐慌 panic，要把这一点记录到你的文档里，并且要记录在什么情况下，它会发生恐慌。

如果你的代码要返回错误，要把这一点记录到你的文档里，并且要记录在什么情况下，它会返回错误。

unsafe 函数，要在文档里写明需要满足什么条件才能安全的调用这个函数。

例子一：

```rust
// Panic（恐慌）是这两种情况的一个很好的例子：如果代码可能发生 Panic，请在文档中明确说明这一点，以及可能导致 Panic 的情况。
// 同样，如果代码可能返回错误，请记录它返回错误的情况。
// 对于 unsafe 的函数，请记录调用者必须保证什么条件，才能确保调用时安全的。

/// 除法运算，返回两个数的结果。
/// 
/// # Panics
///
/// 如果除数为零，该函数会发生 panic。
///
/// # 示例
///
/// ```
/// let result = divide(10, 2);
/// assert_eq!(result, 5);
/// ```
pub fn divide(dividend: i32, divisor: i32) -> i32 { 
  // 实现代码 ...
}

```

2. 在 crate 或 module 级，包括端到端的用例
   1. 不是针对特定类型或方法，了解所有内容如何组合到一起
   2. 对接口的整体结构有一个相对清晰的理解
      1. 让开发者快速了解到各方法和类型的功能，以及在哪使用
   3. 提供定制化使用的起点
      1. 通过复制粘贴，结合需求进行修改

例子：查看标准库等相关文档

3. 组织好文档
   1. 利用模块来将语义相关的项进行分组
   2. 利用内部文档链接将这些项相互连接起来
   3. 考虑使用 `#[doc(hidden)]` 标记那些不打算公开但出于遗留原因需要的接口部分，避免弄乱文档

例子二：

```rust
/// 一个简单的模块，包含一些用于内部使用的函数和结构体。
pub mod internal {
  /// 一个用于内部计算的辅助函数。
  #[doc(hidden)]
  pub fn internal_helper() {
    // 内部计算的具体实现 ...
  }
  
  /// 一个仅用于内部使用的结构体。
  #[doc(hidden)]
  pug struct InternalStruct {
    // 结构体的字段和方法 ...
  }
}

/// 一个公共接口函数，调用了内部的辅助函数。
pub fn public_function() {
  // 调用内部辅助函数
  internal::internal_helper();
}

/// 一个公共结构体，包含一些公共字段和方法。

```

4. 尽可能的丰富文档
   1. 可以链接到解释这些内容的外部资源：
      1. 相关的规范文件（RFC）、博客、白皮书 ...
   2. 使用 `#[doc(cfg(..))]` 突出显示仅在特定配置下可用的项
      1. 用户能快速了解为什么在文档中列出的某个方法不可用
   3. 使用 `#[doc(alias = "...")]` 可让用户以其他名称搜索到类型和方法
   4. 在顶层文档中，引导用户了解常用的模块、Trait、类型、方法

例子三：

```rust
//! 这是一个用于处理图像的库。
//!
//! 这个库提供了一些常用的图像处理功能，例如：
//! - 读取和保存不同格式的图像文件 [`Image::load`] [`Image::save`]
//! - 调整图像的大小、旋转和裁剪 [`Image::resize`] [`Image::rotate`] [`Image::crop`]
//! - 应用不同的滤镜和效果 [`Filter`] [`Effect`]
//!
//! 如果您想了解更多关于图像处理的原理和算法，您可以参考以下的资源：
//! - [数字图像处理](https://book.douban.com/subject/5345798/)，一个经典的教科书，介绍了图像处理的基本概念和方法。
//! - [Learn OpenCV](https://learnopencv.com/)，一个网站，提供了很多用OpenCV实现图像处理功能的教程和示例代码。
//! - [Awesome Computer Vision](https://github.com/jbhuang0604/awesome-computer-vision)，一个GitHub仓库，收集计算机视觉资源。

/// 一个表示图像的结构体
#[derive(Debug, Clone)]
pub struct Image {
  // ...
}

impl Image {
  /// 从指定的路径加载一个图像文件
  ///
  /// 支持的格式有：PNG、JPEG、GIF、BMP 等
  ///
  /// # 参数
  ///
  /// - `path`: 图像文件的路径
  ///
  /// # 返回值
  ///
  /// 如果成功，返回一个 [`Image`] 实例；如果失败，返回一个 [`Error`]。
  ///
  /// # 示例
  ///
  /// ```no_run
  /// use image::Image;
  ///
  /// let img = Image::load("test.png")?;
  /// ```
  #[doc(alias = "读取")]
  #[doc(alias = "打开")]
  pub fn load<P: AsRef<Path>>(path: P) -> Result<Self, Error> {
    // ...
  }
  
  /// 将图像保存到指定的路径
  ///
  /// 支持的格式有：PNG、JPEG、GIF、BMP 等
  ///
  /// # 参数
  ///
}
```

例子四：

```rust
/// 一个只在启用了 `foo` 特性时才可用的结构体。
#[cfg(feature = "foo")]
#[doc(cfg(feature = "foo"))]
pub struct Foo;

impl Foo {
  /// 一个只在启用了 `foo` 特性时才可用的方法。
  #[cfg(feature = "foo")]
  #[doc(cfg(feature = "foo"))]
  pub fn bar(&self) {
    // ...
  }
}

fn main() {
  println!("Hello, world!");
}
```

### 类型系统

- 类型系统可确保：
  - 接口明显
  - 自我描述
  - 难以被误用
- 语义化类型：
  - 添加类型来表示值的意义（不仅仅适用基本类型）

例子五：

```rust
fn processDate(dryRun: bool, overwrite: bool, validate: bool) {
  // 处理数据的逻辑
}

enum DryRun {
  Yes,
  No,
}

enum Overwrite {
  Yes,
  No,
}

enum Validate {
  Yes,
  No,
}

fn processData(dryRun: DryRun, overwrite: Overwrite, validate: Validate) {
  // 处理数据的逻辑
}

processData(DryRun::Yes, Overwrite::No, Validate::Yes);

fn main() {
  println!("Hello, world!");
}
```

- 使用”零大小“类型来表示关于类型实例的特定事实

例子六：

```rust
struct Grounded;
struct Launched;
// and so on

enum Color {
  White,
  Black,
}

struct Kilograms(u32);

struct Rocket<Stage = Grounded> {
  stage: std::marker::PhantomData<Stage>,
}

impl Default for Rocket<Grounded> {
  fn default() -> Self {
    Self {
      stage: Default::default()
    }
  }
}
impl Rocket<Grounded> {
  pub fn launch(self) -> Rocket<Launched> {
    Rocket {
      stage: Default::default(),
    }
  }
}

impl Rocket<Launched> {
  pub fn accelerate(&mut self) {}
  pub fn decelerate(&mut self) {}
}

impl<Stage> Rocket<Stage> {
  pub fn color(&self) -> Color {
    Color::White
  }
  pub fn weight(&self) -> Kilograms {
    Kilograms(0)
  }
}

fn main() {
  println!("Hello, world!");
}
```

- `#[must_use]` 注解
  - 将其添加到类型、Trait 或函数中，如果用户的代码接收到该类型或 Trait 的元素，或调用了该函数，并且没有明确处理它，编译器将发出警告

例子七：

```rust
#[must_use]
fn process_data(data: Data) -> Result<(), Error> {
  // 处理数据的逻辑
  
  Ok(())
}

// 在这个示例中，我们使用 #[must_use] 注解将 process_data 函数标记为必须使用期返回值。
// 如果用户在调用该函数后没有显式处理返回的 Result 类型，编译器将发出警告。
// 这有助于提醒用户在处理潜在的错误情况时要小心，并减少可能得错误。

fn main() {
  println!("Hello, world!");
}
```



