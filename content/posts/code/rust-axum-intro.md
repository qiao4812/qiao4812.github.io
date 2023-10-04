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



```toml
[package]
name = "rust-axum-intro"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
axum = "0.6.20"
tokio = { version = "1.31.0", features = ["full"] }
serde ={ version =  "1.0.183", features = ["derive"]}
serde_json = "1.0.104"

[dev-dependencies]
anyhow = "1.0.73"
httpc-test = "0.1.5" # Uses reqwest & cookie store.

```



```rust
#![allow(unused)] // For beginning only.

use std::net::SocketAddr;

use axum::{
    extract::Query,
    response::{Html, IntoResponse},
    routing::get,
    Router,
};
use serde::Deserialize;

#[tokio::main]
async fn main() {
    let routes_hello = Router::new().route(
        "/hello",
        // get(|| async { Html("Hello <strong>World!!!</strong>") }),
        get(handler_hello),
    );

    // region:  --- Start Server
    let addr = SocketAddr::from(([127, 0, 0, 1], 8080));
    println!("->> LISTENING on {addr}\n");
    axum::Server::bind(&addr)
        .serve(routes_hello.into_make_service())
        .await
        .unwrap();
    // endregion:  --- Start Server
}

// region: --- Handler Hello
#[derive(Debug, Deserialize)]
struct HelloParams {
    name: Option<String>,
}

// e.g., `/hello?name=Qiao`
async fn handler_hello(Query(params): Query<HelloParams>) -> impl IntoResponse {
    println!("->> {:<12} - handler_hello - {params:?}", "HANDLER");

    let name = params.name.as_deref().unwrap_or("World");

    // Html("Hello <strong>World!!!</strong>")
    Html(format!("Hello <strong>{name}!!!</strong>"))
}
// endregion: --- Handler Hello

```



```rust
#![allow(unused)] // For beginning only.

use anyhow::Result;

#[tokio::test]
async fn quick_dev() -> Result<()> {
    let hc = httpc_test::new_client("http://localhost:8080")?;

    hc.do_get("/hello?name=Qiao").await?.print().await?;

    Ok(())
}

```



```shell
cargo install cargo-watch
cargo watch -x run
cargo watch -q -c -x 'run -q'
cargo watch -q -c -x 'run -q --example hello'
```

https://docs.rs/axum/latest/axum/extract/ws/index.html
