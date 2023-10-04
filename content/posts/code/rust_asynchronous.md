---
title: "Rust async 编程"
date: 2023-05-21T16:24:44+08:00
draft: true
tags: ["Rust"]
categories: ["Rust"]
---

# Rust async 编程

Asynchronous Programming in Rust：<https://rust-lang.github.io/async-book/>

中文书名《Rust 异步编程指南》：<https://github.com/rustlang-cn/async-book>

Rust语言圣经(Rust Course)：<https://course.rs/advance/async/getting-started.html>

## 一、[Getting Started](https://rust-lang.github.io/async-book/#getting-started)

### 1.1 为什么使用 async

### 为什么使用 async

- Async 编程，是一种并发（concurrent）编程模型
  - 允许你在少数系统线程上运行大量的并发任务
  - 通过 async/await 语法，看起来和同步编程差不多

### [其它的并发模型](https://course.rs/advance/async/getting-started.html#async-vs-其它并发模型)

- **OS 线程**
  - 无需改变任何编程模型，线程间同步困难，性能开销大
  - 线程池可以降低一些成本，但难以支撑大量 IO 绑定的工作
- **Event-driven 编程**
  - 与回调函数一起用，可能高效
  - 非线性的控制流，数据流和错误传播难以追踪
- **协程(Coroutines)** 
  - 类似线程，无需改变编程模型
  - 类似 `async` ，支持大量任务
  - 抽象掉了底层细节（这对系统编程、自定义运行时的实现很重要）
- **Actor 模型**
  - 将所有并发计算划分为 `actor` , 消息通信易出错
  - 可以有效的实现 actor 模型，但许多实际问题没解决（例如流程控制、重试逻辑）

### Rust 中的 async

- **Future 是惰性的**
  - 只有`poll`时才能取得进展， 被丢弃的 `future` 就无法取得进展了
- **Async是零成本的**
  -  使用`async` ，可以无需堆内存分配（heap allocation）和动态调度（dynamic dispatch），对性能大好，且允许在受限环境使用 async
- **不提供内置运行时**
  - 运行时由Rust 社区提供，例如 [`tokio`](https://tokio.rs/)
- **单线程、多线程均支持**
  - 这两者拥有各自的优缺点

### Rust 中的 async 和线程（thread）

- OS 线程：
  - 适用于少量任务，有内存和CPU开销，且线程生成和线程间切换非常昂贵
  - 线程池可以降低一些成本
  - 允许重用同步代码，代码无需大改，无需特定编程模型
  - 有些系统支持修改线程优先级
- Async：
  - 显著降低内存和CPU开销
  - 同等条件下，支持比线程多几个数量级的任务（少数线程支撑大量任务）
  - 可执行文件大（需要生成状态机，每个可执行文件捆绑一个异步运行时）

Async 并不是比线程好，只是不同而已！

总结：

- 有大量 `IO` 任务需要并发运行时，选 `async` 模型
- 有部分 `IO` 任务需要并发运行时，选多线程，如果想要降低线程创建和销毁的开销，可以使用线程池
- 有大量 `CPU` 密集任务需要并行运行时，例如并行计算，选多线程模型，且让线程数等于或者稍大于 `CPU` 核心数
- 无所谓时，统一选多线程

### 例子

如果想并发的下载文件，你可以使用多线程如下实现:

```rust
fn get_two_sites() {
    // Spawn two threads to do work. 创建两个新线程执行任务
    let thread_one = thread::spawn(|| download("https://www.foo.com"));
    let thread_two = thread::spawn(|| download("https://www.bar.com"));

    // Wait for both threads to complete. 等待两个线程的完成
    thread_one.join().expect("thread one panicked");
    thread_two.join().expect("thread two panicked");
}
```

使用`async`的方式：

```rust
async fn get_two_sites_async() {
    // Create two different "futures" which, when run to completion, 创建两个不同的`future`，你可以把`future`理解为未来某个时刻会被执行的计划任务
    // will asynchronously download the webpages. 当两个`future`被同时执行后，它们将并发的去下载目标页面
    let future_one = download_async("https://www.foo.com");
    let future_two = download_async("https://www.bar.com");

    // Run both futures to completion at the same time. 同时运行两个`future`，直至完成
    join!(future_one, future_two);
}
```

### 自定义并发模型

- 除了线程和async，还可以用其它的并发模型（例如 event-driven）

### 1.2 Rust Async 的目前状态

### Async Rust 目前的状态

- 部分稳定，部分仍在变化。
- 特点：
  - 针对典型并发任务，性能出色
  - 与高级语言特性频繁交互（生命周期、pinning）
  - 同步和异步代码间、不同运行时的异步代码间存在兼容性约束
  - 由于不断进化，维护负担更重

### [语言和库的支持](https://course.rs/advance/async/getting-started.html#语言和库的支持)

- 虽然Rust本身就支持Async编程，但很多应用依赖与社区的库：
  - 标准库提供了最基本的特性、类型和功能，例如 Future trait
  - async/await 语法直接被Rust编译器支持
  - futures crate 提供了许多实用类型、宏和函数。它们可以用于任何异步应用程序。
  - 异步代码、IO 和任务生成的执行由 "async runtimes" 提供，例如 Tokio 和 async-std。大多数async 应用程序和一些 async crate 都依赖于特定的运行时。

### 注意

- Rust 不允许你在 trait 里声明 async 函数

### [编译和调试](https://course.rs/advance/async/getting-started.html#编译和错误)

- 编译错误：
  - 由于 `async` 通常依赖于更复杂的语言功能，例如生命周期和`Pinning`，因此可能会更频繁地遇到这些类型的错误。
- 运行时错误：
  - 每当运行时遇到异步函数，编译器会在后台生成一个状态机，Stack traces 里有其明细，以及运行时调用的函数。因此解释起来更复杂。
- 新的失效模式：
  - 可能出现一些新的故障，它们可以通过编译，甚至单元测试。

### [兼容性考虑](https://course.rs/advance/async/getting-started.html#兼容性考虑)

- async和同步代码不能总是自由组合
  - 例如，不能直接从同步函数来调用 `async` 异步函数

- Async 代码间也不总是能自由组合
  - 一些crate依赖于特定的 `async` 运行时
- 因此，尽早研究确定使用哪个 async 运行时

### [性能特征](https://course.rs/advance/async/getting-started.html#性能特性)

- `async` 的性能依赖于运行时的表现（通常较出色）

### 1.3 async/await 入门

### async

- `async` 把一段代码转化为一个实现了`Future trait` 的状态机
- 虽然在同步方法中调用阻塞函数会阻塞整个线程，但阻塞的Future将放弃对线程的控制，从而允许其它`Future`来运行。

```bash
~/rust via 🅒 base
➜ cargo new async_demo
     Created binary (application) `async_demo` package

~/rust via 🅒 base
➜ cd async_demo

async_demo on  master [?] via 🦀 1.67.1 via 🅒 base
➜ c

async_demo on  master [?] via 🦀 1.67.1 via 🅒 base
➜


```

代码

```rust
#![allow(unused)]
fn main() {
    async fn do_something() {/* ... */}
}

```

Cargo .toml

```toml
[package]
name = "async_demo"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
futures = "0.3.28"

```

### async fn

- 异步函数语法：
  - ` async fn do_something() {/* ... */}`
  - `async fn` 返回的是 Future，Future 需要由一个执行者来运行
- `futures::executor::block_on;`
  - block_on 阻塞当前线程，直到提供的 Future 运行完成
  - 其它执行者提供更复杂的行为，例如将多个 Future 安排到同一个线程上

```rust
use futures::executor::block_on;

async fn hello_world() {
    println!("Hello world!");
}

fn main() {
    let future = hello_world(); // 什么都没打印出来
    block_on(future); // `future` 运行，并打印出 "Hello world!"
}

```



### Await 

- 在 async fn 中，可以使用 .await 来等待另一个实现 Future trait 的完成
- 与 block_on 不同，`.await`不会阻塞当前线程，而是异步的等待 Future 的完成（如果该 Future 目前无法取得进展，就允许其他任务执行）

```rust
use futures::executor::block_on;

struct Song {}

async fn learn_song() -> Song {
    Song {}
}
async fn sing_song(song: Song) { /* ... */
}

async fn dance() { /* ... */
}

fn main() {
    let song = block_on(learn_song());
    block_on(sing_song(song));
    block_on(dance());
}

```

修改之后

```rust
use futures::executor::block_on;

struct Song {}

async fn learn_song() -> Song {
    Song {}
}
async fn sing_song(song: Song) {}

async fn dance() {}

async fn learn_and_sing() {
    let song = learn_song().await;
    sing_song(song).await;
}

async fn async_main() {
    let f1 = learn_and_sing();
    let f2 = dance();
    futures::join!(f1, f2);
}

fn main() {
    block_on(async_main());
}

```

## 二、幕后原理：执行 Future 和任务

### 2.1 Future trait

### Future trait

- `Future` trait是 Rust Async异步编程的核心
- `Future` 是一种异步计算，它可以产生一个值
- 实现了 Future 的类型表示目前可能还不可用的值

下面是一个简化版的 Future trait：

```rust
trait SimpleFuture {
    type Output;
    fn poll(&mut self, wake: fn()) -> Poll<Self::Output>;
}

enum Poll<T> {
    Ready(T),
    Pending,
}

```

- Future 可以表示：

  - 下一次网络数据包的到来

  - 下一次鼠标的移动

  - 或者仅仅是经过一段时间的时间点

- `Future` 代表着一种你可以检验其是否完成的操作
- `Future` 可以通过调用 poll 函数来取得进展
  - poll 函数会驱动 Future 尽可能接近完成
  - 如果 `Future` 完成了：就返回 `poll::Ready(result)`，其中 result 就是最终的结果
  - 如果 `Future` 还无法完成：就返回 `poll::Pending`，并当 Future 准备好取得更多进展时调用一个 `waker `的wake() 函数
- 针对 `Future`，你唯一能做的就是使用 poll 来敲它，直到一个值掉出来

### wake() 函数

- 当 wake() 函数被调用时：
  - 执行器将驱动 Future 再次调用 poll 函数，以便 Future 能取得更多的进展
- 没有wake() 函数，执行器就不知道特定的 Future 何时能取得进展（就得不断的 poll）
- 通过 wake() 函数，执行器就确切的知道哪些 Future 已准备好进行 poll() 的调用

### 例子

```rust
pub struct SocketRead<'a> {
    socket: &'a Socket,
}

impl SimpleFuture for SocketRead<'_> {
    type Output = Vec<u8>;

    fn poll(&mut self, wake: fn()) -> Poll<Self::Output> {
        if self.socket.has_data_to_read() {
            // The socket has data -- read it into a buffer and return it. socket有数据，写入buffer中并返回
            Poll::Ready(self.socket.read_buf())
        } else {
            // The socket does not yet have data. socket中还没数据
            //
            // Arrange for `wake` to be called once data is available. 注册一个`wake`函数，当数据可用时，该函数会被调用，
            // When data becomes available, `wake` will be called, and the
            // user of this `Future` will know to call `poll` again and
            // receive data. 然后当前Future的执行器会再次调用`poll`方法，此时就可以读取到数据
            self.socket.set_readable_callback(wake);
            Poll::Pending
        }
    }
}
```

### 例子

- 组合多个异步操作，而无需中间分配
- 可以通过无分配的状态机来实现多个 Future 同时运行或串联运行

```rust
/// A SimpleFuture that runs two other futures to completion concurrently. 它会并发地运行两个Future直到它们完成
///
/// Concurrency is achieved via the fact that calls to `poll` each future
/// may be interleaved, allowing each future to advance itself at its own pace. 之所以可以并发，是因为两个Future的轮询可以交替进行，一个阻塞，另一个就可以立刻执行，反之亦然
pub struct Join<FutureA, FutureB> {
    // Each field may contain a future that should be run to completion. 结构体的每个字段都包含一个Future，可以运行直到完成.
    // If the future has already completed, the field is set to `None`.
    // This prevents us from polling a future after it has completed, which
    // would violate the contract of the `Future` trait. 等到Future完成后，字段会被设置为 `None`. 这样Future完成后，就不会再被轮询
    a: Option<FutureA>,
    b: Option<FutureB>,
}

impl<FutureA, FutureB> SimpleFuture for Join<FutureA, FutureB>
where
    FutureA: SimpleFuture<Output = ()>,
    FutureB: SimpleFuture<Output = ()>,
{
    type Output = ();
    fn poll(&mut self, wake: fn()) -> Poll<Self::Output> {
        // Attempt to complete future `a`. 尝试去完成一个 Future `a`
        if let Some(a) = &mut self.a {
            if let Poll::Ready(()) = a.poll(wake) {
                self.a.take();
            }
        }

        // Attempt to complete future `b`. 尝试去完成一个 Future `b`
        if let Some(b) = &mut self.b {
            if let Poll::Ready(()) = b.poll(wake) {
                self.b.take();
            }
        }

        if self.a.is_none() && self.b.is_none() {
            // Both futures have completed -- we can return successfully 两个 Future都已完成 - 我们可以成功地返回了
            Poll::Ready(())
        } else {
            // One or both futures returned `Poll::Pending` and still have 至少还有一个 Future 没有完成任务，因此返回 `Poll::Pending`.
            // work to do. They will call `wake()` when progress can be made. 当该 Future 再次准备好时，通过调用`wake()`函数来继续执行
            Poll::Pending
        }
    }
}
```

### 例子

多个连续的 Future 可以一个接一个的运行

```rust
/// A SimpleFuture that runs two futures to completion, one after another. 一个SimpleFuture, 它使用顺序的方式，一个接一个地运行两个Future
//
// Note: for the purposes of this simple example, `AndThenFut` assumes both 注意: 由于本例子用于演示，因此功能简单，`AndThenFut` 会假设两个 Future 在创建时就可用了.
// the first and second futures are available at creation-time. The real
// `AndThen` combinator allows creating the second future based on the output
// of the first future, like `get_breakfast.and_then(|food| eat(food))`. 而真实的`Andthen`允许根据第一个`Future`的输出来创建第二个`Future`，因此复杂的多。
pub struct AndThenFut<FutureA, FutureB> {
    first: Option<FutureA>,
    second: FutureB,
}

impl<FutureA, FutureB> SimpleFuture for AndThenFut<FutureA, FutureB>
where
    FutureA: SimpleFuture<Output = ()>,
    FutureB: SimpleFuture<Output = ()>,
{
    type Output = ();
    fn poll(&mut self, wake: fn()) -> Poll<Self::Output> {
        if let Some(first) = &mut self.first {
            match first.poll(wake) {
                // We've completed the first future -- remove it and start on
                // the second! 我们已经完成了第一个 Future， 可以将它移除， 然后准备开始运行第二个
                Poll::Ready(()) => self.first.take(),
                // We couldn't yet complete the first future. 第一个 Future 还不能完成
                Poll::Pending => return Poll::Pending,
            };
        }
        // Now that the first future is done, attempt to complete the second. 运行到这里，说明第一个Future已经完成，尝试去完成第二个
        self.second.poll(wake)
    }
}
```

### 真正的 Future trait

```rust
trait Future {
    type Output;
    fn poll(
        // Note the change from `&mut self` to `Pin<&mut Self>`: 首先值得注意的地方是，`self`的类型从`&mut self`变成了`Pin<&mut Self>`:
        self: Pin<&mut Self>,
        // and the change from `wake: fn()` to `cx: &mut Context<'_>`: 其次将`wake: fn()` 修改为 `cx: &mut Context<'_>`:
        cx: &mut Context<'_>,
    ) -> Poll<Self::Output>;
}
```

- self 类型不再是 `&mut self`，而是 `pin<&mut self>`
  - 它允许我们创建不可移动的 future
- 不可移动的对象可以在它们的字段间存储指针
- 需要启用`async/await` ，`Pin` 就是必须的

- `wake: fn()` 变成了 `&mut Context<'_>` 。
- 在 SimpleFuture 里：
  - 我们通过调用函数指针 (fn()) 来告诉 Future 的执行器：相关的 Future 应该被 poll 了。
  - 由于 fn() 是一个函数指针，它不能存储任何关于哪个 Future调用了 wake 的数据
- Context 类型提供了访问 Waker类型的值的方式，这些值可以被用来 wake up 特定的任务
  - 例如，实际项目中Web Server可能有上千个不同的连接，它们的wakeup 应单独管理

总之，在正式场景要进行 `wake` ，就必须携带上数据。 而 `Context` 类型通过提供一个 `Waker` 类型的值，就可以用来唤醒特定的的任务。

### 2.2 使用 Waker 唤醒任务

### Waker 类型的作用

- Future在第一次poll的时候通常无法完成任务，所以Future需要保证在准备好取得更多进展后，可以再次被 poll
- 每次 Future 被 poll，它都是作为一个任务的一部分
- 任务（Task）就是被提交给执行者的顶层的 Future

### Waker 类型

- Waker 提供了 wake() 方法，它可以被用来告诉执行者：相关的任务应该被唤醒
- 当 wake() 被调用，执行者知道 Waker 所关联的任务已经准备好取得更多进展，Future应该被再次 poll
- Waker 实现了 clone()，可以复制和存储

### 例子

- 使用 Waker 实现一个简单的计时器 Future
  - 构建一个定时器

```bash
~/rust via 🅒 base
➜ cargo new timer_future
     Created binary (application) `timer_future` package

~/rust via 🅒 base
➜ cd timer_future

timer_future on  master [?] via 🦀 1.67.1 via 🅒 base
➜ c

```

lib.rs 文件

```rust
use std::{
    future::Future,
    pin::Pin,
    sync::{Arc, Mutex},
    task::{Context, Poll, Waker},
    thread,
    time::Duration,
};

pub struct TimerFuture {
    shared_state: Arc<Mutex<SharedState>>,
}

/// Shared state between the future and the waiting thread
/// 在Future和等待的线程间共享状态
struct SharedState {
    /// Whether or not the sleep time has elapsed
    ///  定时(睡眠)是否结束
    completed: bool,

    /// The waker for the task that `TimerFuture` is running on.
    /// The thread can use this after setting `completed = true` to tell
    /// `TimerFuture`'s task to wake up, see that `completed = true`, and
    /// move forward.
    /// 当睡眠结束后，线程可以用`waker`通知`TimerFuture`来唤醒任务
    waker: Option<Waker>,
}

impl Future for TimerFuture {
    type Output = ();
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        // Look at the shared state to see if the timer has already completed.
        // 通过检查共享状态，来确定定时器是否已经完成
        let mut shared_state = self.shared_state.lock().unwrap();
        if shared_state.completed {
            Poll::Ready(())
        } else {
            // Set waker so that the thread can wake up the current task
            // when the timer has completed, ensuring that the future is polled
            // again and sees that `completed = true`.
            //
            // It's tempting to do this once rather than repeatedly cloning
            // the waker each time. However, the `TimerFuture` can move between
            // tasks on the executor, which could cause a stale waker pointing
            // to the wrong task, preventing `TimerFuture` from waking up
            // correctly.
            //
            // N.B. it's possible to check for this using the `Waker::will_wake`
            // function, but we omit that here to keep things simple.
            // 设置`waker`，这样新线程在睡眠(计时)结束后可以唤醒当前的任务，接着再次对`Future`进行`poll`操作,
            //
            // 下面的`clone`每次被`poll`时都会发生一次，实际上，应该是只`clone`一次更加合理。
            // 选择每次都`clone`的原因是： `TimerFuture`可以在执行器的不同任务间移动，如果只克隆一次，
            // 那么获取到的`waker`可能已经被篡改并指向了其它任务，最终导致执行器运行了错误的任务
            shared_state.waker = Some(cx.waker().clone());
            Poll::Pending
        }
    }
}

impl TimerFuture {
    /// Create a new `TimerFuture` which will complete after the provided
    /// timeout.
    /// 创建一个新的`TimerFuture`，在指定的时间结束后，该`Future`可以完成
    pub fn new(duration: Duration) -> Self {
        let shared_state = Arc::new(Mutex::new(SharedState {
            completed: false,
            waker: None,
        }));

        // Spawn the new thread 创建新线程
        let thread_shared_state = shared_state.clone();
        thread::spawn(move || {
            thread::sleep(duration); //  睡眠指定时间实现计时功能
            let mut shared_state = thread_shared_state.lock().unwrap();
            // Signal that the timer has completed and wake up the last
            // task on which the future was polled, if one exists.
            // 通知执行器定时器已经完成，可以继续`poll`对应的`Future`了
            shared_state.completed = true;
            if let Some(waker) = shared_state.waker.take() {
                waker.wake()
            }
        });

        TimerFuture { shared_state }
    }
}

```

### 2.3 构建一个执行器（Executor）

### Future 执行者（Executor）

- Future 是惰性的：除非驱动它们来完成，否则就什么都不做
- 一种驱动方式是在 async 函数里使用 .await，但这只是把问题推到了上一层面
- 谁来执行顶层 async 函数返回的 Future？
  - 需要的是一个 Future 执行者
- Future 执行者会获取一系列顶层的 Future，通过在 Future 可以有进展的时候调用 poll，来将这些 Future 运行至完成
- 通常执行者首先会对 Future进行  poll 一次，以便开始
- 当 Future 通过调用 wake() 表示它们已经准备好取得进展时，它们就会被放回到一个队列里，然后 poll 再次被调用，重复此操作直到 Future 完成

### 例子

- 构建简单的执行者，可以运行大量的顶层 Future 来并发的完成
- 需要使用 future crate 的 ArcWake trait：
  - 它提供了简单的方式用来组建 Waker

lib.rs 文件

```rust
use std::{
    future::Future,
    pin::Pin,
    sync::{Arc, Mutex},
    task::{Context, Poll, Waker},
    thread,
    time::Duration,
};

pub struct TimerFuture {
    shared_state: Arc<Mutex<SharedState>>,
}

/// Shared state between the future and the waiting thread
/// 在Future和等待的线程间共享状态
struct SharedState {
    /// Whether or not the sleep time has elapsed
    ///  定时(睡眠)是否结束
    completed: bool,

    /// The waker for the task that `TimerFuture` is running on.
    /// The thread can use this after setting `completed = true` to tell
    /// `TimerFuture`'s task to wake up, see that `completed = true`, and
    /// move forward.
    /// 当睡眠结束后，线程可以用`waker`通知`TimerFuture`来唤醒任务
    waker: Option<Waker>,
}

impl Future for TimerFuture {
    type Output = ();
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        println!("[{:?}] Polling TimerFuture...", thread::current().id());
        // Look at the shared state to see if the timer has already completed.
        // 通过检查共享状态，来确定定时器是否已经完成
        let mut shared_state = self.shared_state.lock().unwrap();
        if shared_state.completed {
            println!("[{:?}] TimerFuture completed...", thread::current().id());
            Poll::Ready(())
        } else {
            // Set waker so that the thread can wake up the current task
            // when the timer has completed, ensuring that the future is polled
            // again and sees that `completed = true`.
            //
            // It's tempting to do this once rather than repeatedly cloning
            // the waker each time. However, the `TimerFuture` can move between
            // tasks on the executor, which could cause a stale waker pointing
            // to the wrong task, preventing `TimerFuture` from waking up
            // correctly.
            //
            // N.B. it's possible to check for this using the `Waker::will_wake`
            // function, but we omit that here to keep things simple.
            // 设置`waker`，这样新线程在睡眠(计时)结束后可以唤醒当前的任务，接着再次对`Future`进行`poll`操作,
            //
            // 下面的`clone`每次被`poll`时都会发生一次，实际上，应该是只`clone`一次更加合理。
            // 选择每次都`clone`的原因是： `TimerFuture`可以在执行器的不同任务间移动，如果只克隆一次，
            // 那么获取到的`waker`可能已经被篡改并指向了其它任务，最终导致执行器运行了错误的任务
            shared_state.waker = Some(cx.waker().clone());
            println!("[{:?}] TimerFuture pending...", thread::current().id());
            Poll::Pending
        }
    }
}

impl TimerFuture {
    /// Create a new `TimerFuture` which will complete after the provided
    /// timeout.
    /// 创建一个新的`TimerFuture`，在指定的时间结束后，该`Future`可以完成
    pub fn new(duration: Duration) -> Self {
        println!("[{:?}] 开始创建新的 TimerFuture...", thread::current().id());
        let shared_state = Arc::new(Mutex::new(SharedState {
            completed: false,
            waker: None,
        }));

        // Spawn the new thread 创建新线程
        let thread_shared_state = shared_state.clone();
        thread::spawn(move || {
            println!(
                "[{:?}] TimerFuture 生成新的线程并开始睡眠...",
                thread::current().id()
            );
            thread::sleep(duration); //  睡眠指定时间实现计时功能
            let mut shared_state = thread_shared_state.lock().unwrap();
            // Signal that the timer has completed and wake up the last
            // task on which the future was polled, if one exists.
            // 通知执行器定时器已经完成，可以继续`poll`对应的`Future`了
            shared_state.completed = true;
            if let Some(waker) = shared_state.waker.take() {
                println!(
                    "[{:?}] TimerFuture 新线程获得 waker，并进行 wake()...",
                    thread::current().id()
                );
                waker.wake()
            } else {
                println!(
                    "[{:?}] TimerFuture 新线程没获得 waker...",
                    thread::current().id()
                );
            }
        });

        println!("[{:?}] 返回新的 TimerFuture...", thread::current().id());
        TimerFuture { shared_state }
    }
}

```

main.rs 文件

```rust
use futures::{
    future::{BoxFuture, FutureExt},
    task::{waker_ref, ArcWake},
};
use std::{
    thread,
    future::Future,
    sync::mpsc::{sync_channel, Receiver, SyncSender},
    sync::{Arc, Mutex},
    task::Context,
    time::Duration,
};
// The timer we wrote in the previous section: 引入之前实现的定时器模块
use timer_future::TimerFuture;

/// Task executor that receives tasks off of a channel and runs them.
/// 任务执行器，负责从通道中接收任务然后执行
struct Executor {
    ready_queue: Receiver<Arc<Task>>,
}

/// `Spawner` spawns new futures onto the task channel.
/// `Spawner`负责创建新的`Future`然后将它发送到任务通道中
#[derive(Clone)]
struct Spawner {
    task_sender: SyncSender<Arc<Task>>,
}

/// A future that can reschedule itself to be polled by an `Executor`.
/// 一个Future，它可以调度自己(将自己放入任务通道中)，然后等待执行器去`poll`
struct Task {
    /// In-progress future that should be pushed to completion. 进行中的Future，在未来的某个时间点会被完成
    ///
    /// The `Mutex` is not necessary for correctness, since we only have 按理来说`Mutex`在这里是多余的，因为我们只有一个线程来执行任务。但是由于
    /// one thread executing tasks at once. However, Rust isn't smart
    /// enough to know that `future` is only mutated from one thread, Rust并不聪明，它无法知道`Future`只会在一个线程内被修改，并不会被跨线程修改。因此
    /// so we need to use the `Mutex` to prove thread-safety. A production 我们需要使用`Mutex`来满足这个笨笨的编译器对线程安全的执着。
    /// executor would not need this, and could use `UnsafeCell` instead.
    ///  如果是生产级的执行器实现，不会使用`Mutex`，因为会带来性能上的开销，取而代之的是使用`UnsafeCell`
    future: Mutex<Option<BoxFuture<'static, ()>>>,

    /// Handle to place the task itself back onto the task queue.
    /// 可以将该任务自身放回到任务通道中，等待执行器的poll
    task_sender: SyncSender<Arc<Task>>,
}

fn new_executor_and_spawner() -> (Executor, Spawner) {
    // Maximum number of tasks to allow queueing in the channel at once. 任务通道允许的最大缓冲数(任务队列的最大长度)
    // This is just to make `sync_channel` happy, and wouldn't be present in
    // a real executor. 当前的实现仅仅是为了简单，在实际的执行中，并不会这么使用
    const MAX_QUEUED_TASKS: usize = 10_000;
    let (task_sender, ready_queue) = sync_channel(MAX_QUEUED_TASKS);
    println!("[{:?}] 生成 Executor 和 Spawner （含发送端、接收端）...", thread::current().id());
    (Executor { ready_queue }, Spawner { task_sender })
}

impl Spawner {
    // 把 Future 包装成任务发送到通道
    fn spawn(&self, future: impl Future<Output = ()> + 'static + Send) {
        let future = future.boxed();
        let task = Arc::new(Task {
            future: Mutex::new(Some(future)),
            task_sender: self.task_sender.clone(),
        });
        println!("[{:?}] 将 Future 组成 Task，放入 Channel...", thread::current().id());
        self.task_sender.send(task).expect("too many tasks queued");
    }
}

impl ArcWake for Task {
    fn wake_by_ref(arc_self: &Arc<Self>) {
        // Implement `wake` by sending this task back onto the task channel
        // so that it will be polled again by the executor.
        // 通过发送任务到任务管道的方式来实现`wake`，这样`wake`后，任务就能被执行器`poll`
        println!("[{:?}] wake_by_ref...", thread::current().id());
        let cloned = arc_self.clone();
        arc_self
            .task_sender
            .send(cloned)
            .expect("too many tasks queued");
    }
}

impl Executor {
    fn run(&self) {
        println!("[{:?}] Executor running...", thread::current().id());
        while let Ok(task) = self.ready_queue.recv() { // 从通道不断接收任务
            println!("[{:?}] 接收到任务...", thread::current().id());
            // Take the future, and if it has not yet completed (is still Some),
            // poll it in an attempt to complete it. 获取一个future，若它还没有完成(仍然是Some，不是None)，则对它进行一次poll并尝试完成它
            let mut future_slot = task.future.lock().unwrap();
            if let Some(mut future) = future_slot.take() {
                println!("[{:?}] 从任务中取得 Future...", thread::current().id());
                // Create a `LocalWaker` from the task itself  基于任务自身创建一个 `LocalWaker`
                let waker = waker_ref(&task);
                println!("[{:?}] 获得 waker by ref...", thread::current().id());
                let context = &mut Context::from_waker(&waker);
                // `BoxFuture<T>` is a type alias for  #`BoxFuture<T>`是`Pin<Box<dyn Future<Output = T> + Send + 'static>>`的类型别名
                // `Pin<Box<dyn Future<Output = T> + Send + 'static>>`.
                // We can get a `Pin<&mut dyn Future + Send + 'static>`
                // from it by calling the `Pin::as_mut` method.
                // 通过调用`as_mut`方法，可以将上面的类型转换成`Pin<&mut dyn Future + Send + 'static>`
                println!("[{:?}] 获得 context，准备进行 poll()...", thread::current().id());
                if future.as_mut().poll(context).is_pending() {
                    // We're not done processing the future, so put it
                    // back in its task to be run again in the future. Future还没执行完，因此将它放回任务中，等待下次被poll
                    *future_slot = Some(future);
                    println!("[{:?}] Poll::Pending ====", thread::current().id());
                } else {
                    println!("[{:?}] Poll::Ready....", thread::current().id());
                }
            }
        }
        println!("[{:?}] Excutor run 结束", thread::current().id());
    }
}

fn main() {
    // 返回一个执行者和一个任务的生成器
    let (executor, spawner) = new_executor_and_spawner();

    // Spawn a task to print before and after waiting on a timer. 生成一个任务 async块是一个Future
    spawner.spawn(async {
        println!("[{:?}] howdy!", thread::current().id());
        // Wait for our timer future to complete after two seconds. 创建定时器Future，并等待它完成
        TimerFuture::new(Duration::new(2, 0)).await;
        println!("[{:?}] done!", thread::current().id());
    });

    // Drop the spawner so that our executor knows it is finished and won't
    // receive more incoming tasks to run. drop掉任务，这样执行器就知道任务已经完成，不会再有新的任务进来
    println!("[{:?}] drop Spawner!", thread::current().id());
    drop(spawner);

    // Run the executor until the task queue is empty. 运行执行器直到任务队列为空
    // This will print "howdy!", pause, and then print "done!". 任务运行后，会先打印`howdy!`, 暂停2秒，接着打印 `done!`
    executor.run();
}

```

运行

```bash
timer_future on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 2.9s 
➜ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.06s
     Running `target/debug/timer_future`
[ThreadId(1)] 生成 Executor 和 Spawner （含发送端、接收端）...
[ThreadId(1)] 将 Future 组成 Task，放入 Channel...
[ThreadId(1)] drop Spawner!
[ThreadId(1)] Executor running...
[ThreadId(1)] 接收到任务...
[ThreadId(1)] 从任务中取得 Future...
[ThreadId(1)] 获得 waker by ref...
[ThreadId(1)] 获得 context，准备进行 poll()...
[ThreadId(1)] howdy!
[ThreadId(1)] 开始创建新的 TimerFuture...
[ThreadId(1)] 返回新的 TimerFuture...
[ThreadId(1)] Polling TimerFuture...
[ThreadId(1)] TimerFuture pending...
[ThreadId(1)] Poll::Pending ====
[ThreadId(2)] TimerFuture 生成新的线程并开始睡眠...
[ThreadId(2)] TimerFuture 新线程获得 waker，并进行 wake()...
[ThreadId(2)] wake_by_ref...
[ThreadId(1)] 接收到任务...
[ThreadId(1)] 从任务中取得 Future...
[ThreadId(1)] 获得 waker by ref...
[ThreadId(1)] 获得 context，准备进行 poll()...
[ThreadId(1)] Polling TimerFuture...
[ThreadId(1)] TimerFuture completed...
[ThreadId(1)] done!
[ThreadId(1)] Poll::Ready....
[ThreadId(1)] Excutor run 结束

timer_future on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 2.6s 
➜ 
```

### 2.4 执行器和系统IO

###  `Future` 在 `Socket` 上执行异步读取

- 这个 Future 会读取 socket 上可用的数据
- 如果没有数据，它就屈服于执行者：请求当 socket 再次可读时，唤醒它的任务
- 本例中我们不知道 Socket 类型是如何实现的，尤其不知道 set_readable_callback 函数如何工作。那么如何在 socket 再次可读时安排调用 wake() 呢？
  - 一种办法是使用一个线程不断检查 socket 是否可读（低效！）

```rust
pub struct SocketRead<'a> {
    socket: &'a Socket,
}

impl SimpleFuture for SocketRead<'_> {
    type Output = Vec<u8>;

    fn poll(&mut self, wake: fn()) -> Poll<Self::Output> {
        if self.socket.has_data_to_read() {
            // The socket has data -- read it into a buffer and return it. 
           // socket有数据，写入buffer中并返回
            Poll::Ready(self.socket.read_buf())
        } else {
            // The socket does not yet have data. socket中还没数据
            //
            // Arrange for `wake` to be called once data is available.
            // When data becomes available, `wake` will be called, and the
            // user of this `Future` will know to call `poll` again and
            // receive data.
          	// 注册一个`wake`函数，当数据可用时，该函数会被调用，
						// 然后当前Future的执行器会再次调用`poll`方法，此时就可以读取到数据
            self.socket.set_readable_callback(wake);
            Poll::Pending
        }
    }
}
```

- 实际中，这个问题通过与IO感知的系统阻塞原语（primitive）的集成来解决
  - 例如 `Linux` 中的 **`epoll`**，`FreeBSD` 和 `macOS` 中的 **`kqueue`** ，`Windows` 中的 **`IOCP`**, `Fuchisa`中的 **`ports`** 等
  - 所有这些都是通过 Rust 跨平台包的 `mio crate` 来暴露的
- 这些原语（primitive）都允许线程阻塞多个异步 IO 事件，并在其中一个事件完成后返回。

```rust
struct IoBlocker {
    /* ... */
}

struct Event {
    // An ID uniquely identifying the event that occurred and was listened for.
    // Event的唯一ID，该事件发生后，就会被监听起来
    id: usize,

    // A set of signals to wait for, or which occurred.
    // 一组需要等待或者已发生的信号
    signals: Signals,
}

impl IoBlocker {
    /// Create a new collection of asynchronous IO events to block on.
    /// 创建需要阻塞等待的异步IO事件的集合
    fn new() -> Self { /* ... */ }

    /// Express an interest in a particular IO event.
    /// 对指定的IO事件表示兴趣
    fn add_io_event_interest(
        &self,

        /// The object on which the event will occur
        /// 事件所绑定的socket
        io_object: &IoObject,

        /// A set of signals that may appear on the `io_object` for
        /// which an event should be triggered, paired with
        /// an ID to give to events that result from this interest.
        event: Event,
    ) { /* ... */ }

    /// Block until one of the events occurs.
    /// 进入阻塞，直到某个事件出现
    fn block(&self) -> Event { /* ... */ }
}

let mut io_blocker = IoBlocker::new();
io_blocker.add_io_event_interest(
    &socket_1,
    Event { id: 1, signals: READABLE },
);
io_blocker.add_io_event_interest(
    &socket_2,
    Event { id: 2, signals: READABLE | WRITABLE },
);
let event = io_blocker.block();

// prints e.g. "Socket 1 is now READABLE" if socket one became readable.
// 当socket的数据可以读取时，打印 "Socket 1 is now READABLE" 
println!("Socket {:?} is now {:?}", event.id, event.signals);
```

### Socket::set_readable_callback 的伪代码

- Future 执行者可以使用这些原语提供异步 IO 对象，例如 socket，它就可以当特定 IO 事件发生时通过配置回调来运行
- 针对我们 SocketRead 例子，Socket::set_readable_callback 的伪代码大致如下：

```rust
impl Socket {
    fn set_readable_callback(&self, waker: Waker) {
        // `local_executor` is a reference to the local executor.
        // this could be provided at creation of the socket, but in practice
        // many executor implementations pass it down through thread local
        // storage for convenience.
        let local_executor = self.local_executor;

        // Unique ID for this IO object.
        let id = self.id;

        // Store the local waker in the executor's map so that it can be called
        // once the IO event arrives.
        local_executor.event_map.insert(id, waker);
        local_executor.add_io_event_interest(
            &self.socket_file_descriptor,
            Event { id, signals: READABLE },
        );
    }
}
```

现在，我们就只有一个执行者线程，它可以接收 IO事件，并将它们分配到适合的 Waker，这将唤醒相应的任务，并允许执行者在返回检查更多的IO事件之前，驱动更多的任务完成（循环继续...）。

这样，我们只需要一个执行器线程，它会接收IO事件并将其分发到对应的 `Waker` 中，接着后者会唤醒相关的任务，最终通过执行器 `poll` 后，任务可以顺利的继续执行, 这种IO读取流程可以不停的循环，直到 `socket` 关闭。

## 三、async & .await

### 什么是 async/.await

- async/.await 是 Rust 的特殊语法，在发生阻塞时，它让放弃当前线程的控制权成为可能，这就允许在等待操作完成的时候，允许其它代码取得进展

### 使用 async 的两种方式

有两种方式可以使用`async`： `async fn`用于声明函数，`async { ... }`用于声明语句块，它们会返回一个实现 `Future` 特征的值:

- async 和 async blocks：
  - 都返回实现了 Future trait 的值
- async 体和其它 future都是惰性的：
  - 在真正运行之前什么都不做
- 使用 `.await`是最常见的运行future 的方式：
  - 对 future 使用 `.await`就会尝试驱动Future运行至完成
- 如果 Future 被阻塞：
  - 它会放弃当前线程的控制权
  - 当可取得更多进展时，执行器会捡起这个 Future 并恢复执行，最终由 `.await` 完成解析

```rust

// `foo()` returns a type that implements `Future<Output = u8>`.
// `foo().await` will result in a value of type `u8`.
// `foo()`返回一个`Future<Output = u8>`,
// 当调用`foo().await`时，该`Future`将被运行，当调用结束后我们将获取到一个`u8`值
async fn foo() -> u8 { 5 }

fn bar() -> impl Future<Output = u8> {
    // This `async` block results in a type that implements
    // `Future<Output = u8>`.
    // 下面的`async`语句块返回`Future<Output = u8>`
    async {
        let x: u8 = foo().await;
        x + 5
    }
}
```

### 例子

```rust
use async_std::io::prelude::*;
use async_std::net;
use async_std::task;

// 异步函数以 async fn 开始  里面由3个异步函数 .await
// 无需调整 async fn 的返回类型，Rust 自动把它当成相应的 Future 类型
// 返回的 Future 包含所需相关信息：参数、本地变量空间...
// Future 的具体类型由编译器基于函数体和参数自动生成
// 该类型没有名称
// 它实现了 Future<Output=R>
// 第一次对 cheapo_request 进行 poll 时：
// 从函数体顶部开始执行
// 直到第一个 await（针对 TcpStream::connect 返回的 Future）
// 随着 cheapo_request 的 Future 不断被 poll，其执行就是从一个 await 到下一个 await，而且只有子 Future变成 Ready 之后才继续
// cheapo_reauest 的 Future 会追踪：
// 下一次 poll 应恢复继续的那个店
// 以及所需的本地状态（变量、参数、临时变量等）
// 这种途中能暂停执行，然后恢复执行的能力是 async 所独有的
// 由于 await 表达式依赖于“可恢复执行”这个特性，所以 await 只能用在 async 里
// 暂停执行时线程在做什么？
// 它不是在干等，而是在做其它工作。
async fn cheapo_request(host: &str, port: u16, path: &str) -> std::io::Result<String> {
  // .await 会等待，直到 future 变成 ready， await 最终会解析出 future 的值
  // connect 当调用 async 函数时，在其函数体执行前，它就会立即返回
  // 这个 await 表达式会对 connect 的Future进行 poll:
  // 如果没完成 -> 返回 Pending
  // 针对 cheapo_request 的 poll 也无法继续，
  // 直到 connect 的 Future 返回 Ready
  let mut socket = net::TcpStream::connect((host, port)).await?;
  let request = format!("GET {} HTTP/1.1\r\nHost: {}\r\n\r\n", path, host);
  
  socket.write_all(request.as_bytes()).await?;
  socket.shutdown(net::Shutdown::Write)?;
  let mut response = String::new();
  socket.read_to_string(&mut response).await?;
  Ok(response)
}

fn main() -> std::io::Result<()> {
  // 注意：
  // 下一次对 cheapo_request 的 Future 进行 poll 时：
  // 并不在函数体顶部开始执行
  // 它会在 connect Future 进行 poll 的地方继续执行
  // 直到它变成 Ready，才会继续在函数体往下走
  let response = task::block_on(cheapo_request("example.com", 80, "/"))?;
  println!("{}", response);
  Ok(())
}
```

- await：
  - 获得 Future 的所有权，并对其进行 poll
  - 如果 Future Ready，其最终值就是 await 表达式的值，这时执行就可以继续了
  - 否则就返回 Pending 给调用者

### async 的生命周期

- 与传统函数不同：async fn，如果它的参数是引用或是其它非 'static 的，那么它返回的 Future 就会绑定到参数的生命周期上。
- 这意味着 async fn 返回的 future，在 .await 的同时，fn 的非 'static 的参数必须保持有效

```rust
// This function:
async fn foo(x: &u8) -> u8 { *x }

// Is equivalent to this function:
// 上面的函数跟下面的函数是等价的:
fn foo_expanded<'a>(x: &'a u8) -> impl Future<Output = u8> + 'a {
    async move { *x }
}
```

### 存储 future 或传递 future

- 通常，async 的函数在调用后会立即 .await，这就不是问题：
  - 例如：`foo(&x).await`
- 如果存储 future 或将其传递给其它任务或线程，就有问题了...
- 一种变通解决办法：
  - 思路：把使用引用作为参数的 async fn 转为一个 'static future
  - 做法：在 async 块里，将参数和 async fn 的 调用捆绑到一起（延长参数的生命周期来匹配 future）

```rust
fn bad() -> impl Future<Output = u8> {
    let x = 5;
    borrow_x(&x) // ERROR: `x` does not live long enough
}

// 将参数和对 async fn 的调用放在同一个 async 语句块
fn good() -> impl Future<Output = u8> {
    async {
        let x = 5;
        borrow_x(&x).await
    }
}
```

以上代码会报错，因为 `x` 的生命周期只到 `bad` 函数的结尾。 但是 `Future` 显然会活得更久

通过将参数移动到 `async` 语句块内， 将它的生命周期扩展到 `'static`， 并跟返回的 `Future` 保持了一致。

### async move

- async 块和闭包都支持 move
- async move 块会获得其引用变量的所有权：
  - 允许其比当前所在的作用域活得长
  - 但同时也放弃了与其它代码共享这些变量的能力

```rust
/// `async` block:
///
/// Multiple different `async` blocks can access the same local variable
/// so long as they're executed within the variable's scope
// 多个不同的 `async` 语句块可以访问同一个本地变量，只要它们在该变量的作用域内执行
async fn blocks() {
    let my_string = "foo".to_string();

    let future_one = async {
        // ...
        println!("{my_string}");
    };

    let future_two = async {
        // ...
        println!("{my_string}");
    };

    // Run both futures to completion, printing "foo" twice:
    // 运行两个 Future 直到完成
    let ((), ()) = futures::join!(future_one, future_two);
}

/// `async move` block:
///
/// Only one `async move` block can access the same captured variable, since
/// captures are moved into the `Future` generated by the `async move` block.
/// However, this allows the `Future` to outlive the original scope of the
/// variable:
// 由于`async move`会捕获环境中的变量，因此只有一个`async move`语句块可以访问该变量，
// 但是它也有非常明显的好处： 变量可以转移到返回的 Future 中，不再受借用生命周期的限制
fn move_block() -> impl Future<Output = ()> {
    let my_string = "foo".to_string();
    async move {
        // ...
        println!("{my_string}");
    }
}
```

### 在多线程执行者上进行 .await

- 当使用多线程 future 执行者时，future 就可以在线程间移动：
  - 所以 async 体里面用的变量必须能够在线程间移动
  - 因为任何的 .await 都可能导致切换到一个新线程
- 这意味着使用以下类型时不安全的：
  - Rc、&RefCell 和任何其它没有实现 Send trait 的类型，包括没实现 Sync trait 的引用
  - 注意：调用 .await 时，只要这些类型不在作用域内，就可以使用它们。
- 在跨域一个 .await 期间，持有传统的、对 future 无感知的锁，也不是好主意：
  - 可导致线程池锁定
  - 为此，可使用 futures::lock 里的 Mutex 而不是 std::sync 里的

## 四、Pinning

### 什么是 Pin

- Pin 与 Unpin 标记一起工作
- Pin 会保证实现了 !Unpin 的对象永远不会被移动

### 为什么需要 Pin？

```rust
let fut_one = /* ... */; // Future 1
let fut_two = /* ... */; // Future 2
async move {
    fut_one.await;
    fut_two.await;
}
```

- 这会创建一个实现了 Future trait 的匿名类型
- 提供一个和下面代码类似的 poll 方法

```rust
// The `Future` type generated by our `async { ... }` block
// `async { ... }`语句块创建的 `Future` 类型
struct AsyncFuture {
    fut_one: FutOne,
    fut_two: FutTwo,
    state: State,
}

// List of states our `async` block can be in
// `async` 语句块可能处于的状态
enum State {
    AwaitingFutOne,
    AwaitingFutTwo,
    Done,
}

impl Future for AsyncFuture {
    type Output = ();

    fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<()> {
        loop {
            match self.state {
                State::AwaitingFutOne => match self.fut_one.poll(..) {
                    Poll::Ready(()) => self.state = State::AwaitingFutTwo,
                    Poll::Pending => return Poll::Pending,
                }
                State::AwaitingFutTwo => match self.fut_two.poll(..) {
                    Poll::Ready(()) => self.state = State::Done,
                    Poll::Pending => return Poll::Pending,
                }
                State::Done => return Poll::Ready(()),
            }
        }
    }
}
```

当 `poll` 第一次被调用时，它会去查询 `fut_one` 的状态，若 `fut_one` 无法完成，则 `poll` 方法会返回。未来对 `poll` 的调用将从上一次调用结束的地方开始。该过程会一直持续，直到 `Future` 完成为止。

如果上例中 async 块使用引用，会如何？

```rust
async {
    let mut x = [0; 128];
    let read_into_buf_fut = read_into_buf(&mut x);
    read_into_buf_fut.await;
    println!("{:?}", x);
}
```

这段代码会编译成下面的形式：

```rust
struct ReadIntoBuf<'a> {
    buf: &'a mut [u8], // 指向下面的`x`字段
}

struct AsyncFuture {
    x: [u8; 128],
    read_into_buf_fut: ReadIntoBuf<'what_lifetime?>,
}
```

这里，`ReadIntoBuf` 拥有一个引用字段，指向了结构体的另一个字段 `x` ，一旦 `AsyncFuture` 被移动，那 `x` 的地址也将随之变化，此时对 `x` 的引用就变成了不合法的。

- 把 Future Pin（钉）到内存中的特定位置会防止该问题的发生：
  - 可以在 async 块里安全的创建到值的引用

### Pin 介绍

```rust
#[derive(Debug)]
struct Test {
    a: String,
    b: *const String,
}

impl Test {
    fn new(txt: &str) -> Self {
        Test {
            a: String::from(txt),
            b: std::ptr::null(),
        }
    }

    fn init(&mut self) {
        let self_ref: *const String = &self.a;
        self.b = self_ref;
    }

    fn a(&self) -> &str {
        &self.a
    }

    fn b(&self) -> &String {
        assert!(!self.b.is_null(), "Test::b called without Test::init being called first");
        unsafe { &*(self.b) }
    }
}

fn main() {
    let mut test1 = Test::new("test1");
    test1.init();
    let mut test2 = Test::new("test2");
    test2.init();

    println!("a: {}, b: {}", test1.a(), test1.b());
    println!("a: {}, b: {}", test2.a(), test2.b());

}

```

运行输出

```bash
a: test1, b: test1
a: test2, b: test2
```

修改之后

```rust
fn main() {
    let mut test1 = Test::new("test1");
    test1.init();
    let mut test2 = Test::new("test2");
    test2.init();

    println!("a: {}, b: {}", test1.a(), test1.b());
    std::mem::swap(&mut test1, &mut test2); // 交换 test1 和 test2 移动数据
    println!("a: {}, b: {}", test2.a(), test2.b());

}

```

输出

```bash
a: test1, b: test1
a: test1, b: test2
```

### Pin 的实践

- Pin 类型会包裹指针类型，保证指针指向的值不被移动。
- 例如：Pin<&mut T>，Pin<&T>， Pin<Box<T>>
  - 即使 T:!Unpin，也不能保证 T 不被移动

### Unpin trait 

- 大多数类型如果被移动，不会造成问题，它们实现了 Unpin
- 指向 Unpin 类型的指针，可自由的放入或从 Pin 中取出
  - 例如：u8 是 Unpin 的，Pin<&mut u8> 和普通的 &mut u8 一样
- 如果类型拥有 !Unpin 标记，那么在 Pin 之后它们就无法移动了

```rust
use std::pin::Pin;
use std::marker::PhantomPinned;

#[derive(Debug)]
struct Test {
    a: String,
    b: *const String,
    _marker: PhantomPinned,
}


impl Test {
    fn new(txt: &str) -> Self {
        Test {
            a: String::from(txt),
            b: std::ptr::null(),
            _marker: PhantomPinned, // 这个标记可以让我们的类型自动实现特征`!Unpin`
        }
    }

    fn init(self: Pin<&mut Self>) {
        let self_ptr: *const String = &self.a;
        let this = unsafe { self.get_unchecked_mut() };
        this.b = self_ptr;
    }

    fn a(self: Pin<&Self>) -> &str {
        &self.get_ref().a
    }

    fn b(self: Pin<&Self>) -> &String {
        assert!(!self.b.is_null(), "Test::b called without Test::init being called first");
        unsafe { &*(self.b) }
    }
}
```

上面代码中，我们使用了一个标记类型 `PhantomPinned` 将自定义结构体 `Test` 变成了 `!Unpin` (编译器会自动帮我们实现)，因此该结构体无法再被移动。

一旦类型实现了 `!Unpin` ，那将它的值固定到栈( `stack` )上就是不安全的行为，因此在代码中我们使用了 `unsafe` 语句块来进行处理，你也可以使用 [`pin_utils`](https://docs.rs/pin-utils/) 来避免 `unsafe` 的使用。

```rust
pub fn main() {
    // test1 is safe to move before we initialize it
    // 此时的`test1`可以被安全的移动
    let mut test1 = Test::new("test1");
    // Notice how we shadow `test1` to prevent it from being accessed again
    // 新的`test1`由于使用了`Pin`，因此无法再被移动，这里的声明会将之前的`test1`遮蔽掉(shadow)
    let mut test1 = unsafe { Pin::new_unchecked(&mut test1) };
    Test::init(test1.as_mut());

    let mut test2 = Test::new("test2");
    let mut test2 = unsafe { Pin::new_unchecked(&mut test2) };
    Test::init(test2.as_mut());

    println!("a: {}, b: {}", Test::a(test1.as_ref()), Test::b(test1.as_ref()));
    println!("a: {}, b: {}", Test::a(test2.as_ref()), Test::b(test2.as_ref()));
}

```

尝试进行交换

```rust
pub fn main() {
    let mut test1 = Test::new("test1");
    let mut test1 = unsafe { Pin::new_unchecked(&mut test1) };
    Test::init(test1.as_mut());

    let mut test2 = Test::new("test2");
    let mut test2 = unsafe { Pin::new_unchecked(&mut test2) };
    Test::init(test2.as_mut());

    println!("a: {}, b: {}", Test::a(test1.as_ref()), Test::b(test1.as_ref()));
    std::mem::swap(test1.get_mut(), test2.get_mut()); // 报错 
    println!("a: {}, b: {}", Test::a(test2.as_ref()), Test::b(test2.as_ref()));
}

```

### [Pinning to the Heap ](https://rust-lang.github.io/async-book/04_pinning/01_chapter.html#pinning-to-the-heap) 固定到堆上

```rust
use std::pin::Pin;
use std::marker::PhantomPinned;

#[derive(Debug)]
struct Test {
    a: String,
    b: *const String,
    _marker: PhantomPinned,
}

impl Test {
    fn new(txt: &str) -> Pin<Box<Self>> {
        let t = Test {
            a: String::from(txt),
            b: std::ptr::null(),
            _marker: PhantomPinned,
        };
        let mut boxed = Box::pin(t);
        let self_ptr: *const String = &boxed.a;
        unsafe { boxed.as_mut().get_unchecked_mut().b = self_ptr };

        boxed
    }

    fn a(self: Pin<&Self>) -> &str {
        &self.get_ref().a
    }

    fn b(self: Pin<&Self>) -> &String {
        unsafe { &*(self.b) }
    }
}

pub fn main() {
    let test1 = Test::new("test1");
    let test2 = Test::new("test2");

    println!("a: {}, b: {}",test1.as_ref().a(), test1.as_ref().b());
    println!("a: {}, b: {}",test2.as_ref().a(), test2.as_ref().b());
}
```

将一个 `!Unpin` 类型的值固定到堆上，会给予该值一个稳定的内存地址，它指向的堆中的值在 `Pin` 后是无法被移动的。而且与固定在栈上不同，我们知道堆上的值在整个生命周期内都会被稳稳地固定住。

### 总结

- 如果 T:Unpin （默认情况），那么 Pin<'a, T> 与 &'a mut T 完全等价
  - Unpin 意味着该类型如果被 Pin 了，那么它也是可以移动的，所以 Pin 对这种类型不起作用
- 如果 T:!Unpin，那么把 &mut T 变成 Pin 的 T，需要 unsafe 操作
- 大多数标准库类型都实现了 Unpin，Rust 里大部分正常类型也是。由 async/await 生成的 Future 是个例外
- 可以使用特性标记为类型添加一个 !Unpin 绑定（最新版），或者通过添加 `std::marker::PhantomPinned`到类型上（稳定版）
- 可以将数据 Pin 到 Stack 或 Heap 上
- 把 !Unpin 对象 Pin 到 Stack 上需要 unsafe 操作
- 把 !Unpin duix  Pin 到 Heap 上不需要 unsafe 操作
  - 快捷操作：使用 Box::pin
- 针对已经 Pin 的数据，如果它是 T: !Unpin 的，则需要保证它从被 Pin 后，内存一直有效且不会调整其用途，直到 drop 被调用。
  - 这是 Pin  合约的重要部分

- 若 `T: Unpin` ( Rust 类型的默认实现)，那么 `Pin<'a, T>` 跟 `&'a mut T` 完全相同，也就是 `Pin` 将没有任何效果, 该移动还是照常移动
- 绝大多数标准库类型都实现了 `Unpin` ，事实上，对于 Rust 中你能遇到的绝大多数类型，该结论依然成立 ，其中一个例外就是：`async/await` 生成的 `Future` 没有实现 `Unpin`
- 你可以通过以下方法为自己的类型添加`!Unpin`约束：
  - 使用文中提到的 `std::marker::PhantomPinned`
  - 使用`nightly` 版本下的 `feature flag`
- 可以将值固定到栈上，也可以固定到堆上
  - 将 `!Unpin` 值固定到栈上需要使用 `unsafe`
  - 将 `!Unpin` 值固定到堆上无需 `unsafe` ，可以通过 `Box::pin` 来简单的实现
- 当固定类型`T: !Unpin`时，你需要保证数据从被固定到被drop这段时期内，其内存不会变得非法或者被重用

## 五、Streams

### Stream trait

- Stream trait 和 Future trait 类似，但它可以在完成前产生多个值，这点和标准库 Iterator trait 也很像

```rust
trait Stream {
    /// The type of the value yielded by the stream.
    // Stream生成的值的类型
    type Item;

    /// Attempt to resolve the next item in the stream.
    /// Returns `Poll::Pending` if not ready, `Poll::Ready(Some(x))` if a value
    /// is ready, and `Poll::Ready(None)` if the stream has completed.
    // 尝试去解析Stream中的下一个值,
    // 若无数据，返回`Poll::Pending`, 若有数据，返回 `Poll::Ready(Some(x))`, `Stream`完成则返回 `Poll::Ready(None)`
    fn poll_next(self: Pin<&mut Self>, cx: &mut Context<'_>)
        -> Poll<Option<Self::Item>>;
}
```

关于 `Stream` 的一个常见例子是消息通道（ `futures` 包中的）的消费者 `Receiver`。每次有消息从 `Send` 端发送后，它都可以接收到一个 `Some(val)` 值， 一旦 `Send` 端关闭(drop)，且消息通道中没有消息后，它会接收到一个 `None` 值。

```rust
async fn send_recv() {
    const BUFFER_SIZE: usize = 10;
    let (mut tx, mut rx) = mpsc::channel::<i32>(BUFFER_SIZE);

    tx.send(1).await.unwrap();
    tx.send(2).await.unwrap();
    drop(tx);

    // `StreamExt::next` is similar to `Iterator::next`, but returns a
    // type that implements `Future<Output = Option<T>>`.
    // `StreamExt::next` 类似于 `Iterator::next`, 但是前者返回的不是值，而是一个 `Future<Output = Option<T>>`，
    // 因此还需要使用`.await`来获取具体的值
    assert_eq!(Some(1), rx.next().await);
    assert_eq!(Some(2), rx.next().await);
    assert_eq!(None, rx.next().await);
}
```

### Iteration and Concurrency 迭代与并发

- 与同步的 Iterator 类似，有很多方法迭代和处理 Stream 中的值：
  - 组合器风格的：map、filter、fold
  - 相应的 "early-exit-on-error"  版本：try_map、try_filter、try_fold
- for 循环无法和 Stream 一起使用
- 命令式的 while let 和 next/try_next 函数可以与 Stream 一起使用

### 例子

```rust
async fn sum_with_next(mut stream: Pin<&mut dyn Stream<Item = i32>>) -> i32 {
    use futures::stream::StreamExt; // for `next`
    let mut sum = 0;
    while let Some(item) = stream.next().await {
        sum += item;
    }
    sum
}

async fn sum_with_try_next(
    mut stream: Pin<&mut dyn Stream<Item = Result<i32, io::Error>>>,
) -> Result<i32, io::Error> {
    use futures::stream::TryStreamExt; // for `try_next`
    let mut sum = 0;
    while let Some(item) = stream.try_next().await? {
        sum += item;
    }
    Ok(sum)
}
```

上面代码是一次处理一个值的模式，但是需要注意的是：**如果你选择一次处理一个值的模式，可能会造成无法并发，这就失去了异步编程的意义**。 因此，如果可以的话我们还是要选择从一个 `Stream` 并发处理多个值的方式，通过 `for_each_concurrent` 或 `try_for_each_concurrent` 方法来实现:

```rust
async fn jump_around(
    mut stream: Pin<&mut dyn Stream<Item = Result<u8, io::Error>>>,
) -> Result<(), io::Error> {
    use futures::stream::TryStreamExt; // for `try_for_each_concurrent`
    const MAX_CONCURRENT_JUMPERS: usize = 100;

    stream.try_for_each_concurrent(MAX_CONCURRENT_JUMPERS, |num| async move {
        jump_n_times(num).await?;
        report_n_jumps(num).await?;
        Ok(())
    }).await?;

    Ok(())
}
```

## 六、Executing Multiple Futures at a Time（同时执行多个 Future）

### 本节内容

- 真正的异步应用程序通常需要同时执行几个不同的操作
- 介绍一些可同时执行多个异步操作的方式：
  - Join!，等待所有 Future 完成
  - Select!，等待多个 future 中的一个完成
  - Spawning，创建一个顶级任务，他会运行一个 future 直至完成
  - FutureUnordered，一组 Future，它们会产生每个子 Future 的结果

### 6.1 join!

### join!

- futures::join 宏，它使得在等待多个 future完成的时候，可以同时并发的执行它们。

### 例子

```rust
async fn get_book_and_music() -> (Book, Music) {
    let book = get_book().await;
    let music = get_music().await;
    (book, music)
}
```

要支持同时看书和听歌，有些人可能会生成下面代码：

```rust
// WRONG -- don't do this
async fn get_book_and_music() -> (Book, Music) {
    let book_future = get_book();
    let music_future = get_music();
    (book_future.await, music_future.await)
}
```

为了正确的并发运行两个 `Future` ， 我们来试试 `futures::join!` 宏:

```rust
use futures::join;

async fn get_book_and_music() -> (Book, Music) {
    let book_fut = get_book();
    let music_fut = get_music();
    join!(book_fut, music_fut)
}
```

### try_join!

- 对于返回 Result 的 future，更考虑使用 try_join!
  - 如果子 future 中某一个返回了错误，try_join! 会立即完成

### 例子

```rust
use futures::try_join;

async fn get_book() -> Result<Book, String> { /* ... */ Ok(Book) }
async fn get_music() -> Result<Music, String> { /* ... */ Ok(Music) }

async fn get_book_and_music() -> Result<(Book, Music), String> {
    let book_fut = get_book();
    let music_fut = get_music();
    try_join!(book_fut, music_fut)
}
```

有一点需要注意，传给 `try_join!` 的所有 `Future` 都必须拥有相同的错误类型。如果错误类型不同，可以考虑使用来自 `futures::future::TryFutureExt` 模块的 `map_err`和`err_info`方法将错误进行转换:

```rust
use futures::{
    future::TryFutureExt,
    try_join,
};

async fn get_book() -> Result<Book, ()> { /* ... */ Ok(Book) }
async fn get_music() -> Result<Music, String> { /* ... */ Ok(Music) }

async fn get_book_and_music() -> Result<(Book, Music), String> {
    let book_fut = get_book().map_err(|()| "Unable to get book".to_string());
    let music_fut = get_music();
    try_join!(book_fut, music_fut)
}
```

### 6.2 select!

### select!

- futures::select 宏可同时运行多个 future，允许用户在任意一个 future 完成时进行响应

```rust
use futures::{
    future::FutureExt, // for `.fuse()`
    pin_mut,
    select,
};

async fn task_one() { /* ... */ }
async fn task_two() { /* ... */ }

async fn race_tasks() {
    let t1 = task_one().fuse();
    let t2 = task_two().fuse();

    pin_mut!(t1, t2);

    select! {
        () = t1 => println!("task one completed first"),
        () = t2 => println!("task two completed first"),
    }
}

```

### default => ... 和 complete => ...

- select 支持 default 和 complete 分支
- default：如果选中的 future 尚未完成，就会运行 default 分支
  - 拥有 default 的 select 总是会立即返回
- complete：它用于所有选中的 future 都已完成的情况。
- `complete` 分支当所有的 `Future` 和 `Stream` 完成后才会被执行，它往往配合`loop`使用，`loop`用于循环完成所有的 `Future`
- `default`分支，若没有任何 `Future` 或 `Stream` 处于 `Ready` 状态， 则该分支会被立即执行

### 例子

```rust
use futures::{future, select};

async fn count() {
    let mut a_fut = future::ready(4);
    let mut b_fut = future::ready(6);
    let mut total = 0;

    loop {
        select! {
            a = a_fut => total += a,
            b = b_fut => total += b,
            complete => break,
            default => unreachable!(), // never runs (futures are ready, then complete)   该分支永远不会运行，因为`Future`会先运行，然后是`complete`
        };
    }
    assert_eq!(total, 10);
}

```

### 与 Unpin 和 FusedFuture 交互

- 前面的例子中，需要在返回的 future 上调用 `.fuse()`，也调用了 pin_mut。
  - 因为 select 里面的 future 必须使用 Unpin 和 FusedFuture 这两个 trait。
- 必须 Unpin：select 使用的 future 不是按值的，而是按可变引用。
  - 未完成的 future 在调用 select 后 仍可使用
- 必须 FusedFuture：在 future 完成后，select 不可以对它进行 poll
  - 实现 FusedFuture 的 future 会追踪其完成状态，这样在 select 循环里，就只会 poll 没有完成的 future

###  Stream 也有 FusedStream trait

首先，`.fuse()`方法可以让 `Future` 实现 `FusedFuture` 特征， 而 `pin_mut!` 宏会为 `Future` 实现 `Unpin`特征，这两个特征恰恰是使用 `select` 所必须的:

- `Unpin`，由于 `select` 不会通过拿走所有权的方式使用`Future`，而是通过可变引用的方式去使用，这样当 `select` 结束后，该 `Future` 若没有被完成，它的所有权还可以继续被其它代码使用。
- `FusedFuture`的原因跟上面类似，当 `Future` 一旦完成后，那 `select` 就不能再对其进行轮询使用。`Fuse`意味着熔断，相当于 `Future` 一旦完成，再次调用`poll`会直接返回`Poll::Pending`。

只有实现了`FusedFuture`，`select` 才能配合 `loop` 一起使用。假如没有实现，就算一个 `Future` 已经完成了，它依然会被 `select` 不停的轮询执行。

`Stream` 稍有不同，它们使用的特征是 `FusedStream`。 通过`.fuse()`(也可以手动实现)实现了该特征的 `Stream`，对其调用`.next()` 或 `.try_next()`方法可以获取实现了`FusedFuture`特征的`Future`:

```rust
use futures::{
    stream::{Stream, StreamExt, FusedStream},
    select,
};

async fn add_two_streams(
    mut s1: impl Stream<Item = u8> + FusedStream + Unpin,
    mut s2: impl Stream<Item = u8> + FusedStream + Unpin,
) -> u8 {
    let mut total = 0;

    loop {
        let item = select! {
            x = s1.next() => x,
            x = s2.next() => x,
            complete => break,
        };
        if let Some(next_num) = item {
            total += next_num;
        }
    }

    total
}

```

### select 循环里使用 Fuse 和 FuturesUnordered

- Fuse::terminated()，允许构建空的、已完成的 future，后续可以为它填充一个需要运行的 future
  - 适用于在 select 循环里产生且需要在这运行的任务，这种场景
- 当同个 future 的多个副本需同时运行时，使用 FuturesUnordered 类型

```rust
use futures::{
    future::{Fuse, FusedFuture, FutureExt},
    stream::{FusedStream, Stream, StreamExt},
    pin_mut,
    select,
};

async fn get_new_num() -> u8 { /* ... */ 5 }

async fn run_on_new_num(_: u8) { /* ... */ }

async fn run_loop(
    mut interval_timer: impl Stream<Item = ()> + FusedStream + Unpin,
    starting_num: u8,
) {
    let run_on_new_num_fut = run_on_new_num(starting_num).fuse();
    let get_new_num_fut = Fuse::terminated();
    pin_mut!(run_on_new_num_fut, get_new_num_fut);
    loop {
        select! {
            () = interval_timer.select_next_some() => {
                // The timer has elapsed. Start a new `get_new_num_fut`
                // if one was not already running.
                // 定时器已结束，若`get_new_num_fut`没有在运行，就创建一个新的
                if get_new_num_fut.is_terminated() {
                    get_new_num_fut.set(get_new_num().fuse());
                }
            },
            new_num = get_new_num_fut => {
                // A new number has arrived -- start a new `run_on_new_num_fut`,
                // dropping the old one.
                // 收到新的数字 -- 创建一个新的`run_on_new_num_fut`并丢弃掉旧的
                run_on_new_num_fut.set(run_on_new_num(new_num).fuse());
            },
            // Run the `run_on_new_num_fut` // 运行 `run_on_new_num_fut`
            () = run_on_new_num_fut => {},
            // panic if everything completed, since the `interval_timer` should
            // keep yielding values indefinitely.
            // 若所有任务都完成，直接 `panic`， 原因是 `interval_timer` 应该连续不断的产生值，而不是结束          //后，执行到 `complete` 分支
            complete => panic!("`interval_timer` completed unexpectedly"),
        }
    }
}

```

当某个 `Future` 有多个拷贝都需要同时运行时，可以使用 `FuturesUnordered` 类型。下面的例子跟上个例子大体相似，但是它会将 `run_on_new_num_fut` 的每一个拷贝都运行到完成，而不是像之前那样一旦创建新的就终止旧的。

```rust
use futures::{
    future::{Fuse, FusedFuture, FutureExt},
    stream::{FusedStream, FuturesUnordered, Stream, StreamExt},
    pin_mut,
    select,
};

async fn get_new_num() -> u8 { /* ... */ 5 }

async fn run_on_new_num(_: u8) -> u8 { /* ... */ 5 }

// 使用从 `get_new_num` 获取的最新数字 来运行 `run_on_new_num`
//
// 每当计时器结束后，`get_new_num` 就会运行一次，它会立即取消当前正在运行的`run_on_new_num` ,
// 并且使用新返回的值来替换 
async fn run_loop(
    mut interval_timer: impl Stream<Item = ()> + FusedStream + Unpin,
    starting_num: u8,
) {
    let mut run_on_new_num_futs = FuturesUnordered::new();
    run_on_new_num_futs.push(run_on_new_num(starting_num));
    let get_new_num_fut = Fuse::terminated();
    pin_mut!(get_new_num_fut);
    loop {
        select! {
            () = interval_timer.select_next_some() => {
                // The timer has elapsed. Start a new `get_new_num_fut`
                // if one was not already running.
                // 定时器已结束，若`get_new_num_fut`没有在运行，就创建一个新的
                if get_new_num_fut.is_terminated() {
                    get_new_num_fut.set(get_new_num().fuse());
                }
            },
            new_num = get_new_num_fut => {
                // A new number has arrived -- start a new `run_on_new_num_fut`.
                // 收到新的数字 -- 创建一个新的`run_on_new_num_fut` (并没有像之前的例子那样丢弃掉旧值)
                run_on_new_num_futs.push(run_on_new_num(new_num));
            },
            // Run the `run_on_new_num_futs` and check if any have completed
            // 运行 `run_on_new_num_futs`, 并检查是否有已经完成的
            res = run_on_new_num_futs.select_next_some() => {
                println!("run_on_new_num_fut returned {:?}", res);
            },
            // panic if everything completed, since the `interval_timer` should
            // keep yielding values indefinitely.
           // 若所有任务都完成，直接 `panic`， 原因是 `interval_timer` 应该连续不断的产生值，而不是结束
            //后，执行到 `complete` 分支
            complete => panic!("`interval_timer` completed unexpectedly"),
        }
    }
}


```

## 七、Workarounds to Know and Love 一些疑难问题的解决办法

### 7.1 async 块中的 ?

- Async 块中使用 ? 是比较常见的
- 但是 async 块的返回类型没有明确说明，这可能会导致编译器无法推断 async 块的错误类型
- 目前没法给 future 一个类型，也无法指明其类型
- 临时解决办法：使用 “turbofish”运算符，为 async 块提供成功和错误类型

```rust
#![allow(unused)]
fn main() {
  struct MyError
  async fn foo() -> Result<(), MyError> {
    Ok(())
  }
  async fn bar() -> Result<(), MyError> {
    Ok(())
  
  }
  
  let fut = async {
    foo().await?;
    bar().await?;
    Ok(())  // 报错 cannot infer type for type parameter `E` declared on the enum `Result`
  };
}
```

可以使用 `::< ... >` 的方式来增加类型注释：

```rust
#![allow(unused)]
fn main() {
  struct MyError
  async fn foo() -> Result<(), MyError> {
    Ok(())
  }
  async fn bar() -> Result<(), MyError> {
    Ok(())
  
  }
  
  let fut = async {
    foo().await?;
    bar().await?;
    Ok::<(), MyError>(()) // <- note the explicit type annotation here
  };
}
```

### 7.2 [`Send` Approximation](https://rust-lang.github.io/async-book/07_workarounds/03_send_approximation.html#send-approximation)

- 有些 async fn 状态机可安全的跨线程发送，有些则不行
- async future 是否是 Send 的，取决于在 .await 点是否持有非 Send 的类型
- 当值可能在跨域 .await 点被持有时，编译器会尽力近似估算，但这种估算在很多地方显得过于保守
- 临时办法：引入块作用域，把 non-Send 变量隔离

`Rc`无法在多线程环境使用，原因就在于它并未实现 `Send` 特征

```rust
use std::rc::Rc;

#[derive(Default)]
struct NotSend(Rc<()>);

async fn bar() {}
async fn foo() {
    NotSend::default();
    bar().await;
}

fn require_send(_: impl Send) {}

fn main() {
    require_send(foo());
}
```

即使上面的 `foo` 返回的 `Future` 是 `Send`， 但是在它内部短暂的使用 `NotSend` 依然是安全的，原因在于它的作用域并没有影响到 `.await`，下面来试试声明一个变量，然后让 `.await`的调用处于变量的作用域中试试:

```rust
async fn foo() {
    let x = NotSend::default();
    bar().await;
}

```

报错：future cannot be sent between threads safely

`.await`在运行时处于 `x` 的作用域内，.await` 有可能被执行器调度到另一个线程上运行，而 `Rc` 并没有实现 `Send。

可以将变量声明在语句块内，当语句块结束时，变量会自动被 `drop`

```rust
async fn foo() {
    {
        let x = NotSend::default();
    }
    bar().await;
}

```

### 7.3 Recursion

- 在内部，async fn 会创建一个状态机类型，它含有每个被 .awaited 的子 future
  - 这就有点麻烦，因为状态机需要包含其本身
- 临时办法：引入间接，使用Box，并把 recursive 放入非 async 的函数，它会返回 .boxed() async 块

```rust
// This function: foo函数:
async fn foo() {
    step_one().await;
    step_two().await;
}
// generates a type like this:  会被编译成类似下面的类型：
enum Foo {
    First(StepOne),
    Second(StepTwo),
}

// So this function: recursive函数
async fn recursive() {
    recursive().await;
    recursive().await;
}

// generates a type like this:  会生成类似以下的类型
enum Recursive {
    First(Recursive),
    Second(Recursive),
}

```

这是典型的[动态大小类型](https://github.com/rustlang-cn/async-book/blob/master/advance/custom-type.md#动态大小类型)，它的大小会无限增长，因此编译器会直接报错:

```rust
error[E0733]: recursion in an `async fn` requires boxing
 --> src/lib.rs:1:22
  |
1 | async fn recursive() {
  |                      ^ an `async fn` cannot invoke itself directly
  |
  = note: a recursive `async fn` must be rewritten to return a boxed future.

```

 `recursive` 转变成一个正常的函数，该函数返回一个使用 `Box` 包裹的 `async` 语句块：

```rust
use futures::future::{BoxFuture, FutureExt};

fn recursive() -> BoxFuture<'static, ()> {
    async move {
        recursive().await;
        recursive().await;
    }.boxed()
}

```

### 7.4 async in Trait

- 目前 async fn 不可用在 trait 里
- 临时解决办法：使用 async-trait

在目前版本中，我们还无法在特征中定义 `async fn` 函数，不过大家也不用担心，目前已经有计划在未来移除这个限制了。

```rust
trait Test {
    async fn test();
}
```



运行后报错:

```bash
error[E0706]: functions in traits cannot be declared `async`
 --> src/main.rs:5:5
  |
5 |     async fn test();
  |     -----^^^^^^^^^^^
  |     |
  |     `async` because of this
  |
  = note: `async` trait functions are not currently supported
  = note: consider using the `async-trait` crate: https://crates.io/crates/async-trait
```



好在编译器给出了提示，让我们使用 [`async-trait`](https://github.com/dtolnay/async-trait) 解决这个问题:

```rust
use async_trait::async_trait;

#[async_trait]
trait Advertisement {
    async fn run(&self);
}

struct Modal;

#[async_trait]
impl Advertisement for Modal {
    async fn run(&self) {
        self.render_fullscreen().await;
        for _ in 0..4u16 {
            remind_user_to_join_mailing_list().await;
        }
        self.hide_for_now().await;
    }
}

struct AutoplayingVideo {
    media_url: String,
}

#[async_trait]
impl Advertisement for AutoplayingVideo {
    async fn run(&self) {
        let stream = connect(&self.media_url).await;
        stream.play().await;

        // 用视频说服用户加入我们的邮件列表
        Modal.run().await;
    }
}
```



不过使用该包并不是免费的，每一次特征中的`async`函数被调用时，都会产生一次堆内存分配。对于大多数场景，这个性能开销都可以接受，但是当函数一秒调用几十万、几百万次时，就得小心这块儿代码的性能了！

## 八、[The Async Ecosystem](https://rust-lang.github.io/async-book/08_ecosystem/00_chapter.html#the-async-ecosystem)

### Rust 没提供什么

- Rust 目前只提供编写 async 代码的基本要素，标准库中尚未提供执行器、任务、反应器、组合器以及低级 I/O future 和 trait
- 社区提供的 async 生态系统填补了这些空白

### Async 运行时

- Async 运行时是用于执行 async 应用程序的库
- 运行时通常将一个反应器与一个或多个执行器捆绑在一起
- 反应器为异步I/O、进程间通信和计时器等外部事件提供订阅机制
- 在 async 运行时中，订阅者通常是代表低级别 I/O 操作的 future
- 执行者负责安排和执行任务
  - 它们跟踪正在运行和暂停的任务，对future进行 poll 直到完成，并在任务能够取得进展时唤醒任务
  - “执行者”一词经常与“运行时”互换使用
- 我们使用“生态系统”一词来描述一个与兼容 trait 和特性捆绑在一起的运行时

### 社区提供的 async crates

- futures crate ，提供了 Stream、Sink、AsyncRead、AsyncWrite 等 trait，以及组合器等工具。这些可能最终会成为标准库的一部分
- Futures 有自己的执行器，但没有自己的反应器，因此它不支持 async I/O 或计时器 future 的执行
- 因此，它不被认为是完整的运行时
- 常见的选择是： 与另一个 crate 中的执行器一起使用来自 futures 提供的工具

### 流行的运行时

- Tokio：一个流行的 async 生态系统，包含 HTTP、gRPC 和跟踪框架
- Async-std：提供标准库的 async 副本
- Smol：小型、简化的 async 运行时。提供可用于包装 UnixStream 或 TcpListener 等结构的 async trait
- Fuchsia-async：用于 Fuchsia OS 的执行器

### 确定生态兼容性

- 与 async I/O、计时器、进程间通信或任务交互的 async 代码通常取决于特定的异步执行器或反应器
- 所有其他 async 代码，如异步表达式、组合器、同步类型和流，通常与生态系统无关，前提是任何嵌套的 future 也与生态系统无关
- 在开始一个项目之前，建议研究相关的 async 框架和库，以确保与您选择的运行时以及彼此之间的兼容性

### 单线程 VS 多线程执行器

- async 执行器可以是单线程或多线程的
- 多线程执行器可以在多个任务上同时取得进展，对于有许多任务的工作负载，它可以大大加快执行速度，但在任务之间同步数据通常更昂贵
- 在单线程运行时和多线程运行时之间进行选择时，建议测量应用程序的性能
- 任务可以在创建它们的线程上运行，也可以在单独的线程上运行
- 异步运行时通常提供将任务生成到单独线程的功能
  - 即使任务在不同的线程上执行，它们也应该是非阻塞的
- 为了在多线程执行器上调度任务，它们必须是 Send 的
- 一些运行时提供了生成非 Send 的任务的函数，确保每个任务都在生成它的线程上执行
- 它们还可以提供将阻塞任务生成到专用线程的函数，这对于运行来自其他库的阻塞同步代码非常有用

## 九、[Final Project: Building a Concurrent Web Server with Async Rust](https://rust-lang.github.io/async-book/09_example/00_intro.html#final-project-building-a-concurrent-web-server-with-async-rust)

### 构建并发的（单线程的）Web Server

- 使用 async Rust 把 《The Rust Programming Language》（《Rust 权威指南》）一书中第 20 章的例子改为可并发处理请求的 Web Server

```bash
server on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ tree
.
├── 404.html
├── Cargo.lock
├── Cargo.toml
├── hello.html
├── src
│   └── main.rs
└── target
    ├── CACHEDIR.TAG
    └── debug
        ├── build
        ├── deps
        │   ├── libserver-4f1db2ad8f9f3102.rmeta
        │   ├── libserver-d3461d12acfa742e.rmeta
        │   ├── server-4f1db2ad8f9f3102.d
        │   └── server-d3461d12acfa742e.d
        ├── examples
        └── incremental
            ├── server-1ufb07a5oidqf
            │   ├── s-glb12pwpr6-1ctgjrw-2k6dj6f8psanp
            │   │   ├── dep-graph.bin
            │   │   ├── query-cache.bin
            │   │   └── work-products.bin
            │   └── s-glb12pwpr6-1ctgjrw.lock
            └── server-3vy5xgpwslzqx
                ├── s-glb12pwpr7-10sbywt-3rbnmvgc9rslx
                │   ├── dep-graph.bin
                │   ├── query-cache.bin
                │   └── work-products.bin
                └── s-glb12pwpr7-10sbywt.lock

12 directories, 18 files

server on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ 
```

main.rs

```rust
use std::fs;
use std::io::prelude::*;
use std::net::TcpListener;
use std::net::TcpStream;

fn main() {
    // Listen for incoming TCP connections on localhost port 7878 监听本地端口 7878 ，等待 TCP 连接的建立
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    // Block forever, handling each request that arrives at this IP address 阻塞等待请求的进入
    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    // Read the first 1024 bytes of data from the stream 从连接中顺序读取 1024 字节数据
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).unwrap();

    let get = b"GET / HTTP/1.1\r\n";

    // Respond with greetings or a 404,
    // depending on the data in the request 处理HTTP协议头，若不符合则返回404和对应的`html`文件
    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND\r\n\r\n", "404.html")
    };
    let contents = fs::read_to_string(filename).unwrap();

    // Write response back to the stream, 将回复内容写入连接缓存中
    // and flush the stream to ensure the response is sent back to the client 使用flush将缓存中的内容发送到客户端
    let response = format!("{status_line}{contents}");
    stream.write_all(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}

```

hello.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <title>Hello!</title>
</head>

<body>
    <h1>Hello!</h1>
    <p>Hi from Rust</p>
</body>

</html>

```

404.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <title>Hello!</title>
</head>

<body>
    <h1>Oops!</h1>
    <p>Sorry, I don't know what you're asking for.</p>
</body>

</html>

```

运行

```bash
server on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 5.0s 
➜ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/server`


```

![image-20230526102033235](../../../Library/Application Support/typora-user-images/image-20230526102033235.png)

![image-20230526102147894](../../../Library/Application Support/typora-user-images/image-20230526102147894.png)

#### 并发地处理连接

```rust
use async_std::net::TcpListener;
use async_std::net::TcpStream;
use async_std::prelude::*;
use async_std::task;
use futures::stream::StreamExt;
use std::fs;
use std::time::Duration;


#[async_std::main]
async fn main() {
    // Listen for incoming TCP connections on localhost port 7878 监听本地端口 7878 ，等待 TCP 连接的建立
    let listener = TcpListener::bind("127.0.0.1:7878").await.unwrap();

    listener.incoming().for_each_concurrent(/* limit */ None, |tcpstream| async move {
        let  tcpstream = tcpstream.unwrap();
        handle_connection(tcpstream).await;
    }) 
    .await;
}

async fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).await.unwrap();

    let get = b"GET / HTTP/1.1\r\n";
    let sleep = b"GET /sleep HTTP/1.1\r\n";

    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else if buffer.starts_with(sleep) {
        // async_std::task::sleep，它仅会让当前的任务陷入睡眠，然后该任务会让出线程的控制权，这样线程就可以继续运行其它任务。
        task::sleep(Duration::from_secs(5)).await;
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND\r\n\r\n", "404.html")
    };
    let contents = fs::read_to_string(filename).unwrap();

    let response = format!("{status_line}{contents}");
    stream.write(response.as_bytes()).await.unwrap();
    stream.flush().await.unwrap();
}

```

Cargo .toml

```toml
[package]
name = "server"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
async-std = { version = "1.12.0", features = ["attributes"] }
futures = "0.3.28"

```

异步版本的 `TcpListener` 为 `listener.incoming()` 实现了 `Stream` 特征，以上修改有两个好处:

- `listener.incoming()` 不再阻塞
- 使用 `for_each_concurrent` 并发地处理从 `Stream` 获取的元素

访问：http://127.0.0.1:7878/    和 http://127.0.0.1:7878/sleep

#### 使用多线程并行处理请求

```rust
use async_std::net::TcpListener;
use async_std::net::TcpStream;
use async_std::prelude::*;
use async_std::task;
use async_std::task::spawn;
use futures::stream::StreamExt;
use std::fs;
use std::time::Duration;


#[async_std::main]
async fn main() {
    // Listen for incoming TCP connections on localhost port 7878 监听本地端口 7878 ，等待 TCP 连接的建立
    let listener = TcpListener::bind("127.0.0.1:7878").await.unwrap();

    listener.incoming().for_each_concurrent(/* limit */ None, |tcpstream| async move {
        let  tcpstream = tcpstream.unwrap();
        // handle_connection(tcpstream).await;
        spawn(handle_connection(tcpstream));
    }) 
    .await;
}

async fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).await.unwrap();

    let get = b"GET / HTTP/1.1\r\n";
    let sleep = b"GET /sleep HTTP/1.1\r\n";

    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else if buffer.starts_with(sleep) {
        // async_std::task::sleep，它仅会让当前的任务陷入睡眠，然后该任务会让出线程的控制权，这样线程就可以继续运行其它任务。
        task::sleep(Duration::from_secs(5)).await;
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND\r\n\r\n", "404.html")
    };
    let contents = fs::read_to_string(filename).unwrap();

    let response = format!("{status_line}{contents}");
    stream.write(response.as_bytes()).await.unwrap();
    stream.flush().await.unwrap();
}

```

#### [Testing the TCP Server](https://rust-lang.github.io/async-book/09_example/03_tests.html#testing-the-tcp-server) 测试 handle_connection 函数

```rust
use async_std::io::{Read, Write};
use async_std::net::TcpListener;
use async_std::prelude::*;
use async_std::task;
use async_std::task::spawn;
use futures::stream::StreamExt;
use std::fs;
use std::marker::Unpin;
use std::time::Duration;

#[async_std::main]
async fn main() {
    // Listen for incoming TCP connections on localhost port 7878 监听本地端口 7878 ，等待 TCP 连接的建立
    let listener = TcpListener::bind("127.0.0.1:7878").await.unwrap();

    listener
        .incoming()
        .for_each_concurrent(/* limit */ None, |tcpstream| async move {
            let tcpstream = tcpstream.unwrap();
            // handle_connection(tcpstream).await;
            spawn(handle_connection(tcpstream));
        })
        .await;
}

async fn handle_connection(mut stream: impl Read + Write + Unpin) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).await.unwrap();

    let get = b"GET / HTTP/1.1\r\n";
    let sleep = b"GET /sleep HTTP/1.1\r\n";

    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else if buffer.starts_with(sleep) {
        // async_std::task::sleep，它仅会让当前的任务陷入睡眠，然后该任务会让出线程的控制权，这样线程就可以继续运行其它任务。
        task::sleep(Duration::from_secs(5)).await;
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND\r\n\r\n", "404.html")
    };
    let contents = fs::read_to_string(filename).unwrap();

    let response = format!("{status_line}{contents}");
    stream.write(response.as_bytes()).await.unwrap();
    stream.flush().await.unwrap();
}

#[cfg(test)]
mod tests {
    use super::*;
    use futures::io::Error;
    use futures::task::{Context, Poll};

    use std::cmp::min;
    use std::pin::Pin;

    struct MockTcpStream {
        read_data: Vec<u8>,
        write_data: Vec<u8>,
    }

    impl Read for MockTcpStream {
        fn poll_read(
            self: Pin<&mut Self>,
            _: &mut Context,
            buf: &mut [u8],
        ) -> Poll<Result<usize, Error>> {
            let size: usize = min(self.read_data.len(), buf.len());
            buf[..size].copy_from_slice(&self.read_data[..size]);
            Poll::Ready(Ok(size))
        }
    }

    impl Write for MockTcpStream {
        fn poll_write(
            // poll_write 会拷贝输入数据到mock的 TcpStream 中，当完成后返回 Poll::Ready
            mut self: Pin<&mut Self>,
            _: &mut Context,
            buf: &[u8],
        ) -> Poll<Result<usize, Error>> {
            self.write_data = Vec::from(buf);

            Poll::Ready(Ok(buf.len()))
        }

        fn poll_flush(self: Pin<&mut Self>, _: &mut Context) -> Poll<Result<(), Error>> {
            Poll::Ready(Ok(()))
        }

        fn poll_close(self: Pin<&mut Self>, _: &mut Context) -> Poll<Result<(), Error>> {
            Poll::Ready(Ok(()))
        }
    }

    use std::marker::Unpin;
    impl Unpin for MockTcpStream {}

    use std::fs;

    #[async_std::test]
    async fn test_handle_connection() {
        let input_bytes = b"GET / HTTP/1.1\r\n";
        let mut contents = vec![0u8; 1024];
        contents[..input_bytes.len()].clone_from_slice(input_bytes);
        let mut stream = MockTcpStream {
            read_data: contents,
            write_data: Vec::new(),
        };

        handle_connection(&mut stream).await;
        let mut buf = [0u8; 1024];
        stream.read(&mut buf).await.unwrap();

        let expected_contents = fs::read_to_string("hello.html").unwrap();
        let expected_response = format!("HTTP/1.1 200 OK\r\n\r\n{}", expected_contents);
        assert!(stream.write_data.starts_with(expected_response.as_bytes()));
    }
}

```

运行

```bash
server on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 53m 39.4s 
➜ cargo test
   Compiling server v0.1.0 (/Users/qiaopengjun/rust/server)
    Finished test [unoptimized + debuginfo] target(s) in 0.64s
     Running unittests src/main.rs (target/debug/deps/server-c69496e95f46c228)

running 1 test
test tests::test_handle_connection ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


server on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ 

```

