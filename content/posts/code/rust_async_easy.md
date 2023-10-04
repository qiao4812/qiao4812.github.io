---
title: "Rust Async 异步编程 简易教程"
date: 2023-05-20T13:59:56+08:00
draft: true
tags: ["Rust"]
categories: ["Rust"]
---

# Rust Async 简易教程

### 课程内容

- 异步编程的概念
- 同步、多线程、异步的例子
- 理解 Async
- 理解 Future
- 最后一个例子

## 一、异步编程的概念

### 并发与并行

- 并发（concurrency）是指程序不同部分可以同时不按顺序的执行且不影响最终结果的能力。
- 而同时执行多个任务是并行（parallelism）

本课中，并发一次代表 concurrency 和 parallelism

### 为什么要让程序不按顺序执行

1. 需要让软件运行的更快；
2. 多 CPU 或多核 CPU 的计算机，让程序开发人员可以编写出整体性能更好的程序。

软件程序处理任务的两种类型：

- CPU 密集型：占用 CPU 资源的任务。
  - 文件压缩、视频编码、图形处理和计算加密证明。
- IO 密集型：占用 IO 资源的任务。
  - 从文件系统或数据库访问数据，以及处理网络 TCP/HTTP 请求。

CPU 密集型的任务：通常可以利用多 CPU 或多核进行处理。

IO 密集型任务：

- 我们以 WEB 项目中处理请求的任务为例。在WEB项目中，我们通过 CRUD 操作把数据从数据库里传递过来，这就要求 CPU 等待数据写入到磁盘，但磁盘很慢。所以，如果程序从数据库请求 10000 笔数据，它会向操作系统请求磁盘的访问，而与此同时，CPU 就是在干等。那么程序员应该怎么利用 CPU 的这段等待的时间？那就是让 CPU 执行其它的任务。
- 另一个典型的例子就是网络请求的处理，客户端建立连接发送请求，服务器端处理请求返回响应并关闭连接。如果CPU还在处理前一个请求，而第二个的请求却已经到达，那么第二个的请求必须在队列中等待着第一个请求处理完成吗？或者我们可以把第二个请求安排到其他可用的CPU或内核上？

### 同步、多线程、异步的区别

Synchronous 同步

Multi-Threading 多线程

Asynchronous 异步

Task 1 包含三个步骤：

1. 处理数据
2. 一个阻塞操作
3. 把从任务要返回的数据进行打包

### WEB 服务器同时接收多个请求的例子

多线程，针对每个请求都开启一个原生的系统线程。提供性能，但却引入了新的复杂性：

- 执行的顺序无法预测；死锁；竞态条件（race condition）
- 多线程还有另外一个问题...

多线程的两种模型：

- 1:1 模型，一个语言线程对应一个系统线程；
- M:N 模型，M 个准（绿色）线程对应 N 个系统线程。

而 Rust 标准库实现的是 1:1 模型

### 并发方式的例子

每个 HTTP 请求被异步 WEB Server 接收，Web Server 会生成一个任务来处理它，并由异步运行时安排各个异步任务在可用的CPU上执行

## 二、同步、多线程、异步的例子

同步例子

```rust
use std::thread::sleep;
use std::time::Duration;

fn main() {
    println!("Hello before reading file!");
    let file1_contents = read_from_file1();
    println!("{:?}", file1_contents);
    println!("Hello after reading file1!");
    let file2_contents = read_from_file2();
    println!("{:?}", file2_contents);
    println!("Hello after reading file2!");
}

fn read_from_file1() -> String {
    sleep(Duration::new(4, 0));
    String::from("Hello, there from file 1")
}

fn read_from_file2() -> String {
    sleep(Duration::new(2, 0));
    String::from("Hello, there from file 2")
}

```

运行

```bash
asc on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ cargo run                
   Compiling asc v0.1.0 (/Users/qiaopengjun/rust/asc)
    Finished dev [unoptimized + debuginfo] target(s) in 0.53s
     Running `target/debug/asc`
Hello before reading file!
"Hello, there from file 1"
Hello after reading file1!
"Hello, there from file 2"
Hello after reading file2!

asc on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 7.3s 
➜ 

```

多线程例子

```rust
use std::thread;
use std::thread::sleep;
use std::time::Duration;

fn main() {
    println!("Hello before reading file!");
    let handle1 = thread::spawn(|| {
        let file1_contents = read_from_file1();
        println!("{:?}", file1_contents);
    });
    let handle2 = thread::spawn(|| {
        let file2_contents = read_from_file2();
        println!("{:?}", file2_contents);
    });
    handle1.join().unwrap();
    handle2.join().unwrap();
}

fn read_from_file1() -> String {
    sleep(Duration::new(4, 0));
    String::from("Hello, there from file 1")
}

fn read_from_file2() -> String {
    sleep(Duration::new(2, 0));
    String::from("Hello, there from file 2")
}

```

运行

```bash
asc on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 7.3s 
➜ cargo run
   Compiling asc v0.1.0 (/Users/qiaopengjun/rust/asc)
    Finished dev [unoptimized + debuginfo] target(s) in 0.25s
     Running `target/debug/asc`
Hello before reading file!
"Hello, there from file 2"
"Hello, there from file 1"

asc on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 4.7s 
➜ 
```

异步例子

```rust
use std::thread::sleep;
use std::time::Duration;

#[tokio::main]
async fn main() {
    println!("Hello before reading file!");

    let h1 = tokio::spawn(async {
        let _file1_contents = read_from_file1().await;
    });

    let h2 = tokio::spawn(async {
        let _file2_contents = read_from_file2().await;
    });
    let _ = tokio::join!(h1, h2);
}

async fn read_from_file1() -> String {
    sleep(Duration::new(4, 0));
    println!("{:?}", "Processing file 1");
    String::from("Hello, there from file 1")
}

async fn read_from_file2() -> String {
    sleep(Duration::new(2, 0));
    println!("{:?}", "Processing file 2");
    String::from("Hello, there from file 2")
}

```

运行

```bash
asc on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 3.3s 
➜ cargo run
   Compiling asc v0.1.0 (/Users/qiaopengjun/rust/asc)
    Finished dev [unoptimized + debuginfo] target(s) in 0.28s
     Running `target/debug/asc`
Hello before reading file!
"Processing file 2"
"Processing file 1"

asc on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 4.9s 
➜ 
```

## 三、理解 Async

### 继续理解 Async

异步编程，诀窍就是当 CPU 等待外部事件或动作时，异步运行时会安排其他可继续执行的任务在 CPU 上执行。而当从磁盘或 I/O 子系统的系统中断到达的时候，异步运行时会知道识别这事，并安排原来的任务继续执行。

一般来说，I/O 受限（I/O Bound）的程序（程序执行的速度依赖于I/O 子系统的速度）比起 CPU受限（CPU Bound）的任务（程序执行的速度依赖于CPU的速度）可能更适合于异步任务的执行。

async、.await 关键字是Rust标准库里用于异步编程的内置核心原语集的代表。就是语法糖。

### Future

Rust 异步的核心其实是 Future

Future 是由异步计算或函数产生的单一最终值。

Rust 的异步函数都会返回 Future，Future 基本上就是代表着延迟的计算

谁来调用poll 方法呢？

是异步执行器，它是异步运行时的一部分。异步执行器会管理一个 Future 的集合，并通过调用 Future 上的 poll 方法来驱动它们完成。所以函数或代码块在前面加上 async 关键字之后，就相当于告诉异步执行器他会返回 Future，这个Future需要被驱动直到完成。

但是异步执行器怎么知道异步已经准备好可以取得进展（可以产生值）了呢？它会持续不断的调用 poll 方法吗？

```rust
use std::thread::sleep;
use std::time::Duration;

#[tokio::main]
async fn main() {
    println!("Hello before reading file!");

    let h1 = tokio::spawn(async {
        let _file1_contents = read_from_file1().await;
    });

    let h2 = tokio::spawn(async {
        let _file2_contents = read_from_file2().await;
    });
    let _ = tokio::join!(h1, h2);
}

// async fn read_from_file1() -> String {
//     sleep(Duration::new(4, 0));
//     println!("{:?}", "Processing file 1");
//     String::from("Hello, there from file 1")
// }

// async fn read_from_file2() -> String {
//     sleep(Duration::new(2, 0));
//     println!("{:?}", "Processing file 2");
//     String::from("Hello, there from file 2")
// }

// Poll::Ready
// Poll::Pending 

use std::future::Future;

fn read_from_file1() -> impl Future<Output = String> {
    async {
        sleep(Duration::new(4, 0));
        println!("{:?}", "Processing file 1");
        String::from("Hello, there from file 1")
    }
}

fn read_from_file2() -> impl Future<Output = String> {
    async {
        sleep(Duration::new(3, 0));
        println!("{:?}", "Processing file 2");
        String::from("Hello, there from file 2")
    }
}

```

## 四、理解 Future

### 利用 Tokio 库来尝试理解Future

Tokio 运行时就是管理异步任务并安排它们在 CPU上执行的组件。

一个程序可能生成多个任务，每个任务可能包含一个或多个Future

### 例子：自定义 Future

main()    

任务1 ReadFileFuture

任务2 read_file2 返回的 Future

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};
use std::thread::sleep;
use std::time::Duration;

struct ReadFileFuture {}

impl Future for ReadFileFuture {
    type Output = String;

    fn poll(self: Pin<&mut Self>, _cx: &mut Context<'_>) -> Poll<Self::Output> {
        println!("Tokio! Stop polling me");
        Poll::Pending
    }
}

#[tokio::main]
async fn main() {
    println!("Hello before reading file!");

    let h1 = tokio::spawn(async {
        let future1 = ReadFileFuture {};
        future1.await
    });

    let h2 = tokio::spawn(async {
        let file2_contents = read_from_file2().await;
        println!("{:?}", file2_contents);
    });
    let _ = tokio::join!(h1, h2);
}

fn read_from_file2() -> impl Future<Output = String> {
    async {
        sleep(Duration::new(2, 0));
        println!("{:?}", "Processing file 2");
        String::from("Hello, there from file 2")
    }
}

```

运行

```bash
asc on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 4.9s 
➜ cargo run
   Compiling asc v0.1.0 (/Users/qiaopengjun/rust/asc)
    Finished dev [unoptimized + debuginfo] target(s) in 0.51s
     Running `target/debug/asc`
Hello before reading file!
Tokio! Stop polling me
"Processing file 2"
"Hello, there from file 2"

```

1. main 函数生成第一个异步任务；
2. main 函数生成第二个异步任务；
3. 第一个任务调用自定义的Future，Future会返回 Poll::Pending；
4. 第二个任务调用异步函数 read_file2()，2秒后函数返回。

### Tokio 执行器如何知道何时再次 poll 第一个 Future 呢

它是一直不断的对它进行 poll 吗？肯定不会一直 poll

Tokio（Rust的异步设计）是使用一个 Waker 组件来处理这件事的。

当被异步执行器 poll 过的任务还没有准备好产生值的时候，这个任务就被注册到一个 Waker。Waker 会有一个处理程序（handle），它会被存储在任务管理的 Context 对象中。

Waker 有一个 wake() 方法，可以用来告诉异步执行器关联的任务应该被唤醒了。当 wake() 方法被调用了，Tokio 执行器就会被通知是时候再次 poll 这个异步的任务了，具体方式就是调用任务上的 poll() 函数。

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};
use std::thread::sleep;
use std::time::Duration;

struct ReadFileFuture {}

impl Future for ReadFileFuture {
    type Output = String;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        println!("Tokio! Stop polling me");
        cx.waker().wake_by_ref();
        Poll::Pending
    }
}

#[tokio::main]
async fn main() {
    println!("Hello before reading file!");

    let h1 = tokio::spawn(async {
        let future1 = ReadFileFuture {};
        future1.await
    });

    let h2 = tokio::spawn(async {
        let file2_contents = read_from_file2().await;
        println!("{:?}", file2_contents);
    });
    let _ = tokio::join!(h1, h2);
}

fn read_from_file2() -> impl Future<Output = String> {
    async {
        sleep(Duration::new(2, 0));
        println!("{:?}", "Processing file 2");
        String::from("Hello, there from file 2")
    }
}

```

1. main 函数生成第一个异步任务；
2. main 函数生成第二个异步任务；
3. 第一个任务调用自定义的Future，它会调用 Waker 的 wake() 方法，这就会告诉异步运行时（Tokio）让其再次 poll 这个Future。这个操纵一直循环进行；
4. 第二个任务调用异步函数 read_file2()，2秒后函数返回。

### Tokio 的组件组成

Tokio 运行时需要理解操作系统（内核）的方法来开启I/O操作（读取网络数据，读写文件等）。

Tokio 运行时会注册异步的处理程序，以便在事件发生时作为I/O操作的一部分进行调用。而在 Tokio 运行时里面，从内核监听这些事件并与Tokio 其他部分通信的组件就是反应器（reactor）。

Tokio 执行器，它会把一个 Future，当其可取得更多进展时，通过调用 Future 的poll() 方法来驱动其完成。

那么Future是如何告诉执行器它们准备好取得进展了呢？

就是 Future 调用 Waker 组件上的 wake() 方法。

Waker 组件就会通知执行器，然后再把 Future 放回队列，并再次调用 poll() 方法，直到 Future 完成。



Future1    Future2

Tokio Waker     Tokio Waker

Tokio 执行器    

Tokio 反应器

异步 kernel (epoll、kqueue)

操作系统

硬件/CPU...

### Tokio 组件的简化工作流程

1. Main 函数在 Tokio 运行时上生成任务1；
2. 任务 1 有一个 Future，会从一个大文件读取内容；
3. 从文件读取内容的请求交给到系统内核文件子系统；
4. 与此同时，任务 2 也被 Tokio 运行时安排进行处理；
5. 当任务 1 的文件操作结束时，文件子系统会触发一个系统中断，它会被翻译成 Tokio 响应器可识别的一个事件；
6. Tokio 响应器会通知任务 1：文件操作的数据已经准备好；
7. 任务1 通知它注册的 Waker 组件：说明它可以产生一个值了；
8. Waker 组件通知 Tokio 执行器来调用任务 1 关联的 poll() 方法；
9. Tokio 执行器安排任务 1 进行处理，并调用 poll() 方法；
10. 任务 1 产生一个值。

### 再次修改后的执行顺序

1. main 函数生成第一个异步任务；
2. main 函数生成第二个异步任务；
3. 第一个任务调用自定义的Future，它返回 Poll::Ready()；
4. 第二个任务调用异步函数 read_file2()，2秒后函数返回。

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};
use std::thread::sleep;
use std::time::Duration;

struct ReadFileFuture {}

impl Future for ReadFileFuture {
    type Output = String;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> 
    Poll<Self::Output> {
        println!("Tokio! Stop polling me");
        cx.waker().wake_by_ref();
        Poll::Ready(String::from("Hello, there from file 1"))
    }
}

#[tokio::main]
async fn main() {
    println!("Hello before reading file!");

    let h1 = tokio::spawn(async {
        let future1 = ReadFileFuture {};
        future1.await
    });

    let h2 = tokio::spawn(async {
        let file2_contents = read_from_file2().await;
        println!("{:?}", file2_contents);
    });
    let _ = tokio::join!(h1, h2);
}

fn read_from_file2() -> impl Future<Output = String> {
    async {
        sleep(Duration::new(2, 0));
        println!("{:?}", "Processing file 2");
        String::from("Hello, there from file 2")
    }
}

```

## 五、最后一个例子

创建一个自定义的 Future，它是一个定时器，具有以下功能：

- 可设定超时时间；
- 当它被异步运行时的执行器 poll 的时候，它会检查：
  - 如果当前世界 >= 超时时间，则返回 Poll::Ready，里面带着一个 String 值；
  - 如果当前时间 < 超时时间，则它会睡眠直至超时为止。然后它会触发 Waker 上的wake() 方法，这就会通知异步运行时的执行器来安排并再次执行该任务。

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};
use std::thread::sleep;
use std::time::{Duration, Instant};

struct AsyncTimer {
    expiration_time: Instant,
}

impl Future for AsyncTimer {
    type Output = String;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        if Instant::now() >= self.expiration_time {
            println!("Hello, it's time for Future 1");
            Poll::Ready(String::from("Future 1 has completed"))
        } else {
            println!("Hello, it's not yet time for Future 1. Going to sleep");
            let waker = cx.waker().clone();
            let expiration_time = self.expiration_time;
            std::thread::spawn(move || {
                let current_time = Instant::now();
                if current_time < expiration_time {
                    std::thread::sleep(expiration_time - current_time);
                }
                waker.wake();
            });
            Poll::Pending
        }
    }
}

#[tokio::main]
async fn main() {
    let h1 = tokio::spawn(async {
        let future1 = AsyncTimer {
            expiration_time: Instant::now() + Duration::from_millis(4000),
        };
        println!("{:?}", future1.await);
    });

    let h2 = tokio::spawn(async {
        let file2_contents = read_from_file2().await;
        println!("{:?}", file2_contents);
    });
    let _ = tokio::join!(h1, h2);
}

fn read_from_file2() -> impl Future<Output = String> {
    async {
        sleep(Duration::new(2, 0));
        String::from("Future 2 has completed")
    }
}

```

运行

```bash
asynctimer on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ cargo run           
   Compiling cfg-if v1.0.0
   Compiling scopeguard v1.1.0
   Compiling smallvec v1.10.0
   Compiling pin-project-lite v0.2.9
   Compiling bytes v1.4.0
   Compiling libc v0.2.144
   Compiling log v0.4.17
   Compiling lock_api v0.4.9
   Compiling parking_lot_core v0.9.7
   Compiling num_cpus v1.15.0
   Compiling signal-hook-registry v1.4.1
   Compiling socket2 v0.4.9
   Compiling mio v0.8.6
   Compiling parking_lot v0.12.1
   Compiling tokio v1.28.1
   Compiling asynctimer v0.1.0 (/Users/qiaopengjun/rust/asynctimer)
    Finished dev [unoptimized + debuginfo] target(s) in 2.84s
     Running `target/debug/asynctimer`
Hello, it's not yet time for Future 1. Going to sleep
"Future 2 has completed"
Hello, it's time for Future 1
"Future 1 has completed"

asynctimer on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 7.4s 
➜ 
```

