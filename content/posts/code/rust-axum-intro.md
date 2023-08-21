---
title: "rust-axum-intro 项目实操"
date: 2023-08-16T09:23:53+08:00
lastmod: 2023-08-16T09:23:53+08:00 
draft: false
categories:
- Rust
tags:
- Rust
---

# rust-axum-intro 项目实操



```shell
 2311  cd Code/rust
 2312  cargo new rust-axum-intro
 2313  cd rust-axum-intro
 2314  c
 2315* cargo add tokio --features full
 2316* cargo add axum
 2317* cargo run
 2318* touch tests/quick_dev.rs
 2319* mkdir tests
 2320* touch tests/quick_dev.rs
 2321  cargo watch -q -c -w src -x run
 2322  cargo install cargo-watch
 2323  cargo watch --version\n
 2324  cargo watch -q -c -w src -x run
 2325  cargo watch -q -c -w tests -x "test -q quick_dev -- --nocapture"
 2326* git add .
 2327* git commit -m "first commit"
 2328  cargo watch -q -c -w tests -x "test -q quick_dev -- --nocapture"
 2329* cd blog/qiaoblog
```

https://docs.rs/crate/cargo-watch/8.4.0
