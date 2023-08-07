---
title: "Rust实现博物馆门票限流"
date: 2023-08-02T13:29:10+08:00
lastmod: 2023-08-02T13:29:10+08:00 
draft: false
categories:
- Rust
tags:
- Rust
---

# Rust实现博物馆门票限流

 博物馆门票
 疫情期间，博物馆内限流最大容量50人，满了以后，出来一个才能进一个，怎么设计？

## Rust实现博物馆门票限流实操

cargo.toml

```toml
[package]
name = "training_code"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
tokio = { version = "1.29.1", features = ["sync"] }

```



lib.rs

```rust
pub mod ticket;
```



ticket.rs 

```rust
// 博物馆门票
// 疫情期间，博物馆内限流最大容量50人，满了以后，出来一个才能进一个，怎么设计？
// Mutex -> Semaphore
// https://www.cs.brandeis.edu/~cs146a/rust/doc-02-21-2015/nightly/std/sync/struct.Semaphore.html
// https://docs.rs/tokio/latest/tokio/sync/struct.Semaphore.html
use tokio::sync::{Semaphore, SemaphorePermit};

pub struct Museum {
    remaining_tickets: Semaphore,
}

#[derive(Debug)]
pub struct Ticket<'a> {
    permit: SemaphorePermit<'a>,
}

impl<'a> Ticket<'a> {
    pub fn new(permit: SemaphorePermit<'a>) -> Self {
        Self { permit }
    }
}

impl<'a> Drop for Ticket<'a> {
    fn drop(&mut self) {
        println!("ticket freed")
    }
}

impl Museum {
    pub fn new(total: usize) -> Self {
        Self {
            remaining_tickets: Semaphore::new(total),
        }
    }

    pub fn get_ticket(&self) -> Option<Ticket<'_>> {
        // 从信号量中获取许可
        match self.remaining_tickets.try_acquire() {
            Ok(permit) => Some(Ticket::new(permit)),
            Err(_) => None,
        }
    }

    pub fn tickets(&self) -> usize {
        self.remaining_tickets.available_permits() // 返回当前可用许可的数量
    }
}

#[cfg(test)]
mod test {
    use super::*;

    #[test]
    fn it_works() {
        let museum = Museum::new(50);
        let ticket = museum.get_ticket().unwrap();
        assert_eq!(museum.tickets(), 49);
        let _tickets: Vec<Ticket> = (0..49).map(|_| museum.get_ticket().unwrap()).collect();
        assert_eq!(museum.tickets(), 0);

        assert!(museum.get_ticket().is_none());

        drop(ticket);
        {
            let ticket = museum.get_ticket().unwrap();
            println!("got ticket: {:?}", ticket);
        }
        println!("!!!!!");
        assert!(museum.get_ticket().is_some());
    }
}

```

测试

```bash
training_code on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ cargo test ticket::test::it_works -- --nocapture                                
   Compiling training_code v0.1.0 (/Users/qiaopengjun/Code/rust/training_code)
warning: field `permit` is never read
  --> src/ticket.rs:14:5
   |
13 | pub struct Ticket<'a> {
   |            ------ field in this struct
14 |     permit: SemaphorePermit<'a>,
   |     ^^^^^^
   |
   = note: `Ticket` has a derived impl for the trait `Debug`, but this is intentionally ignored during dead code analysis
   = note: `#[warn(dead_code)]` on by default

warning: `training_code` (lib test) generated 1 warning
    Finished test [unoptimized + debuginfo] target(s) in 0.94s
     Running unittests src/lib.rs (target/debug/deps/training_code-5f74f464a78aa0d2)

running 1 test
ticket freed
got ticket: Ticket { permit: SemaphorePermit { sem: Semaphore { ll_sem: Semaphore { permits: 0 } }, permits: 1 } }
ticket freed
!!!!!
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
ticket freed
test ticket::test::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out; finished in 0.00s


training_code on  master [?] is 📦 0.1.0 via 🦀 1.71.0 via 🅒 base 
➜ 
```

