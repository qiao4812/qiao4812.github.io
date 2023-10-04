---
title: "Rust编程语言入门之无畏并发"
date: 2023-04-16T20:43:14+08:00
draft: true
tags: ["Rust"]
categories: ["Rust"]
---

# 无畏并发

### 并发

- Concurrent：程序的不同部分之间独立的执行（并发）
- Parallel：程序的不同部分同时运行（并行）
- Rust无畏并发：允许你编写没有细微Bug的代码，并在不引入新Bug的情况下易于重构
- 注意：本文中的”并发“泛指 concurrent 和 parallel

## 一、使用线程同时运行代码（多线程）

### 进程与线程

- 在大部分OS里，代码运行在进程（process）中，OS同时管理多个进程。
- 在你的程序里，各独立部分可以同时运行，运行这些独立部分的就是线程（thread）
- 多线程运行：
  - 提升性能表现
  - 增加复杂性：无法保障各线程的执行顺序

### 多线程可导致的问题

- 竞争状态，线程以不一致的顺序访问数据或资源
- 死锁，两个线程彼此等待对方使用完所持有的资源，线程无法继续
- 只在某些情况下发生的 Bug，很难可靠地复制现象和修复

### 实现线程的方式

- 通过调用OS的API来创建线程：1:1模型
  - 需要较小的运行时
- 语言自己实现的线程（绿色线程）：M:N模型
  - 需要更大的运行时
- Rust：需要权衡运行时的支持
- Rust标准库仅提供1:1模型的线程

### 通过 spawn 创建新线程

- 通过 thread::spawn 函数可以创建新线程：
  - 参数：一个闭包（在新线程里运行的代码）

```bash
➜ cd rust

~/rust
➜ cargo new thread_demo
     Created binary (application) `thread_demo` package

~/rust
➜ cd thread_demo

thread_demo on  master [?] via 🦀 1.67.1
➜ c # code .

thread_demo on  master [?] via 🦀 1.67.1
➜

```

- thread::sleep 会导致当前线程暂停执行

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));  // 暂停 1 毫秒
    }
}

```

执行

```bash
thread_demo on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ cargo run            
   Compiling thread_demo v0.1.0 (/Users/qiaopengjun/rust/thread_demo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.65s
     Running `target/debug/thread_demo`
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 4 from the main thread!
hi number 5 from the spawned thread!

thread_demo on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ 

```

### 通过 join Handle 来等待所有线程的完成

- thread::spawn 函数的返回值类型是 JoinHandle
- JoinHandle 持有值的所有权
  - 调用其 join 方法，可以等待对应的其它线程的完成
- join 方法：调用 handle 的join方法会阻止当前运行线程的执行，直到 handle 所表示的这些线程终结。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));  // 暂停 1 毫秒
    }

    handle.join().unwrap();
}

```

执行

```bash
thread_demo on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ cargo run
   Compiling thread_demo v0.1.0 (/Users/qiaopengjun/rust/thread_demo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.75s
     Running `target/debug/thread_demo`
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 2 from the main thread!
hi number 3 from the spawned thread!
hi number 3 from the main thread!
hi number 4 from the spawned thread!
hi number 4 from the main thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!

thread_demo on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
```

等分线程执行完继续执行主线程

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1)); // 暂停 1 毫秒
    }
}

```

运行

```bash
thread_demo on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
➜ cargo run
   Compiling thread_demo v0.1.0 (/Users/qiaopengjun/rust/thread_demo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.28s
     Running `target/debug/thread_demo`
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!

thread_demo on  master [?] is 📦 0.1.0 via 🦀 1.67.1 
```

### 使用 move 闭包

- move 闭包通常和 thread::spawn 函数一起使用，它允许你使用其它线程的数据
- 创建线程时，把值的所有权从一个线程转移到另一个线程

```rust
use std::thread;

fn main() {
  let v = vec![1, 2, 3];
  let handle = thread::spawn(|| { // 报错
    println!("Here's a vector: {:?}", v);
  });
  
  // drop(v);
  handle.join().unwrap();
}
```

修改后：

```rust
use std::thread;

fn main() {
  let v = vec![1, 2, 3];
  let handle = thread::spawn(move || { 
    println!("Here's a vector: {:?}", v);
  });
  
  // drop(v);
  handle.join().unwrap();
}
```

## 二、使用消息传递来跨线程传递数据

### 消息传递

- 一种很流行且能保证安全并发的技术就是：消息传递。
  - 线程（或 Actor）通过彼此发送消息（数据）来进行通信
- Go 语言的名言：不要用共享内存来通信，要用通信来共享内存。
- Rust：Channel（标准库提供）

### Channel

- Channel 包含： 发送端、接收端
- 调用发送端的方法，发送数据
- 接收端会检查和接收到达的数据
- 如果发送端、接收端中任意一端被丢弃了，那么Channel 就”关闭“了

### 创建 Channel

- 使用 `mpsc::channel`函数来创建 Channel
  - mpsc 表示 multiple producer，single consumer（多个生产者、一个消费者）
  - 返回一个 tuple（元组）：里面元素分别是发送端、接收端

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
  let (tx, rx) = mpsc::channel();
  
  thread::spawn(move || {
    let val = String::from("hi");
    tx.send(val).unwrap();
  });
  
  let received = rx.recv().unwrap();
  println!("Got: {}", received);
}
```

### 发送端的 send 方法

- 参数：想要发送的数据
- 返回：Result<T, E>
  - 如果有问题（例如接收端已经被丢弃），就返回一个错误

### 接收端的方法

- recv 方法：阻止当前线程执行，直到 Channel 中有值被送来
  - 一旦有值收到，就返回 Result<T, E>
  - 当发送端关闭，就会收到一个错误
- try_recv 方法：不会阻塞，
  - 立即返回 Result<T, E>：
    - 有数据达到：返回 Ok，里面包含着数据
    - 否则，返回错误
  - 通常会使用循环调用来检查 try_recv 的结果

### Channel 和所有权转移

- 所有权在消息传递中非常重要：能帮你编写安全、并发的代码

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
  let (tx, rx) = mpsc::channel();
  
  thread::spawn(move || {
    let val = String::from("hi");
    tx.send(val).unwrap();
    println!("val is {}", val)  // 报错 借用了移动的值
  });
  
  let received = rx.recv().unwrap();
  println!("Got: {}", received);
}
```

### 发送多个值，看到接收者在等待

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
  let (tx, rx) = mpsc::channel();
  
  thread::spawn(move || {
    let vals = vec![
      String::from("hi"),
      String::from("from"),
      String::from("the"),
      String::from("thread"),
    ];
    
    for val in vals {
      tx.send(val).unwrap();
      thread::sleep(Duration::from_millis(1));
    }  
  });
  
  for received in rx {
    println!("Got: {}", received);
  }
}
```

### 通过克隆创建多个发送者

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
  let (tx, rx) = mpsc::channel();
  
  let tx1 = mpsc::Sender::clone(&tx);
  thread::spawn(move || {
    let vals = vec![
      String::from("1: hi"),
      String::from("1: from"),
      String::from("1: the"),
      String::from("1: thread"),
    ];
    
    for val in vals {
      tx1.send(val).unwrap();
      thread::sleep(Duration::from_millis(1));
    }  
  });
   thread::spawn(move || {
    let vals = vec![
      String::from("hi"),
      String::from("from"),
      String::from("the"),
      String::from("thread"),
    ];
    
    for val in vals {
      tx.send(val).unwrap();
      thread::sleep(Duration::from_millis(1));
    }  
  });
  
  for received in rx {
    println!("Got: {}", received);
  }
}
```

## 三、共享状态的并发

### 使用共享来实现并发

- Go 语言的名言：不要用共享内存来通信，要用通信来共享内存。
- Rust支持通过共享状态来实现并发。
- Channel 类似单所有权：一旦将值的所有权转移至 Channel，就无法使用它了
- 共享内存并发类似多所有权：多个线程可以同时访问同一块内存

### 使用 Mutex 来每次只允许一个线程来访问数据

- Mutex 是 mutual exclusion（互斥锁）的简写
- 在同一时刻，Mutex 只允许一个线程来访问某些数据
- 想要访问数据：
  - 线程必须首先获取互斥锁（lock）
    - lock 数据结构是 mutex 的一部分，它能跟踪谁对数据拥有独占访问权
  - mutex 通常被描述为：通过锁定系统来保护它所持有的数据

### Mutex 的两条规则

- 在使用数据之前，必须尝试获取锁（lock）。
- 使用完 mutex 所保护的数据，必须对数据进行解锁，以便其它线程可以获取锁。

### `Mutex<T>` 的 API

- 通过 Mutex::new(数据) 来创建 `Mutex<T>`
  - `Mutex<T>`是一个智能指针
- 访问数据前，通过 lock 方法来获取锁
  - 会阻塞当前线程
  - lock 可能会失败
  - 返回的是 MutexGuard（智能指针，实现了 Deref 和 Drop）

```rust
use std::sync::Mutex;

fn main() {
  let m = Mutex::new(5);
  
  {
    let mut num = m.lock().unwrap();
    *num = 6;
  }
  
  println!("m = {:?}", m);
}
```

###  多线程共享 `Mutex<T>`

```rust
use std::sync::Mutex;
use std::thread;

fn main() {
  let counter = Mutex::new(0);
  let mut handles = vec![];
  
  for _ in 0..10 {
     let handle = thread::spawn(move || {  // 报错 循环 所有权
       let mut num = counter.lock().unwrap();
       
       *num += 1;
    });
    handles.push(handle);
  }
  
  for handle in handles {
    handle.join().unwrap();
  }
  
  println!("Result: {}", *counter.lock().unwrap());
}
```

### 多线程的多重所有权

```rust
use std::sync::Mutex;
use std::thread;
use std::rc::Rc;

fn main() {
  let counter = Rc::new(Mutex::new(0));
  let mut handles = vec![];
  
  for _ in 0..10 {
    let counter = Rc::clone(&counter);
    let handle = thread::spawn(move || {  // 报错 rc 只能用于单线程
       let mut num = counter.lock().unwrap();
       
       *num += 1;
    });
    handles.push(handle);
  }
  
  for handle in handles {
    handle.join().unwrap();
  }
  
  println!("Result: {}", *counter.lock().unwrap());
}
```

### 使用 `Arc<T>`来进行原子引用计数

- `Arc<T>`和`Rc<T>`类似，它可以用于并发情景
  - A：atomic，原子的
- 为什么所有的基础类型都不是原子的，为什么标准库类型不默认使用 `Arc<T>`？
  - 需要性能作为代价
- `Arc<T>`和`Rc<T>` 的API是相同的

```rust
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
  let counter = Arc::new(Mutex::new(0));
  let mut handles = vec![];
  
  for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = thread::spawn(move || {  
       let mut num = counter.lock().unwrap();
       
       *num += 1;
    });
    handles.push(handle);
  }
  
  for handle in handles {
    handle.join().unwrap();
  }
  
  println!("Result: {}", *counter.lock().unwrap());
}
```

### `RefCell<T>`/`Rc<T>` vs `Muter<T>`/`Arc<T>`

- `Mutex<T>`提供了内部可变性，和 Cell 家族一样
- 我们使用 `RefCell<T>`来改变 `Rc<T>`里面的内容
- 我们使用 `Mutex<T>` 来改变 `Arc<T>` 里面的内容
- 注意：`Mutex<T>` 有死锁风险

## 四、通过 Send 和 Sync Trait 来扩展并发

### Send 和 Sync trait 

- Rust 语言的并发特性较少，目前讲的并发特性都来自标准库（而不是语言本身）
- 无需局限于标准库的并发，可以自己实现并发
- 但在Rust语言中有两个并发概念：
  - std::marker::Sync 和 std::marker::Send 这两个trait

### Send：允许线程间转移所有权

- 实现 Send trait 的类型可在线程间转移所有权
- Rust中几乎所有的类型都实现了 Send
  - 但 `Rc<T>` 没有实现 Send，它只用于单线程情景
- 任何完全由Send 类型组成的类型也被标记为 Send
- 除了原始指针之外，几乎所有的基础类型都是 Send

### Sync：允许从多线程访问

- 实现Sync的类型可以安全的被多个线程引用
- 也就是说：如果T是Sync，那么 &T 就是 Send
  - 引用可以被安全的送往另一个线程
- 基础类型都是 Sync
- 完全由 Sync 类型组成的类型也是 Sync
  - 但，`Rc<T>`不是 Sync 的
  - `RefCell<T>` 和 `Cell<T>`家族也不是 Sync的
  - 而，`Mutex<T>`是Sync的

### 手动来实现 Send 和 Sync 是不安全的



