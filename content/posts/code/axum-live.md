---
title: "Rust 学习之 Axum Web 应用程序框架"
date: 2023-09-03T11:11:20+08:00
lastmod: 2023-09-03T11:11:20+08:00 
draft: false
categories:
- Rust
tags:
- Rust
---

# Rust 学习之 Axum Web 应用程序框架

Axum 是一个关注人机工程学和模块化的 Web 应用程序框架。

https://docs.rs/axum/latest/axum/

## Axum 实操

### 创建项目并用VSCode 打开

```shell
~ via 🅒 base
➜ cd Code/rust

~/Code/rust via 🅒 base
➜ cargo new --lib axum-live
     Created library `axum-live` package

~/Code/rust via 🅒 base
➜ cd axum-live

axum-live on  master [?] via 🦀 1.72.0 via 🅒 base
➜ c

```

### 添加相关依赖并创建示例文件

```shell
axum-live on  master [?] via 🦀 1.72.0 via 🅒 base 
➜ cargo add axum           

axum-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base took 2.6s 
➜ cargo add anyhow

axum-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ cargo add tokio --features full

axum-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ mkdir examples

axum-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ touch examples/basic.rs    

axum-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ cargo add serde --features derive
axum-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base took 3.8s 
➜ cargo add jsonwebtoken      
```

### examples/basic.rs 

```rust
use std::net::SocketAddr;

use axum::{http::StatusCode, response::Html, routing::get, Json, Router, Server};
use serde::{Deserialize, Serialize};

// 定义一个 Todo 结构体
#[derive(Serialize, Deserialize, Debug)]
pub struct Todo {
    pub id: usize,
    pub title: String,
    pub completed: bool,
}

#[derive(Serialize, Deserialize, Debug)]
pub struct CreateTodo {
    pub title: String,
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(index_handler))
        .route("/todos", get(todos_handler).post(create_todo_handler));

    let addr = SocketAddr::from(([127, 0, 0, 1], 8000));
    println!("listening on http://{}", addr);

    Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

async fn index_handler() -> Html<&'static str> {
    Html("<h1>Hello, World!</h1>")
}

async fn todos_handler() -> Json<Vec<Todo>> {
    Json(vec![
        Todo {
            id: 1,
            title: "Buy milk".to_string(),
            completed: false,
        },
        Todo {
            id: 2,
            title: "Buy eggs".to_string(),
            completed: false,
        },
    ])
}

async fn create_todo_handler(Json(todo): Json<CreateTodo>) -> StatusCode {
    println!("{:?}", todo);
    StatusCode::CREATED
}

```

### Cargo.toml

```toml
[package]
name = "axum-live"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
anyhow = "1.0.75"
axum = "0.6.20"
jsonwebtoken = "8.3.0"
serde = { version = "1.0.188", features = ["derive"] }
tokio = { version = "1.32.0", features = ["full"] }

```



### 运行

```shell
axum-live on  master [?] is 📦 0.1.0 via 🦀 1.72.0 via 🅒 base 
➜ cargo run --example basic        
listening on http://127.0.0.1:8000
```

### basic.http

```http
http://localhost:8000/

### 
// todos_handler
GET http://localhost:8000/todos

###
POST http://localhost:8000/todos HTTP/1.1
content-type: application/json

{
    "title": "larry"
}

```

## jsonwebtoken

https://docs.rs/jsonwebtoken/latest/jsonwebtoken/

https://github.com/tokio-rs/axum/blob/main/examples/jwt/src/main.rs

### examples/basic.rs 

```rust
use std::{net::SocketAddr, time::SystemTime};

use axum::{
    async_trait,
    extract::{FromRequest, FromRequestParts},
    headers::{authorization::Bearer, Authorization},
    http::{self, Request},
    http::{request::Parts, StatusCode},
    response::{Html, IntoResponse, Response},
    routing::{get, post},
    Json, RequestPartsExt, Router, Server, TypedHeader,
};
use jsonwebtoken as jwt;
use jwt::Validation;
use serde::{Deserialize, Serialize};
use serde_json::json;

const SECRET: &[u8] = b"deadbeef";

// 定义一个 Todo 结构体
#[derive(Serialize, Deserialize, Debug)]
pub struct Todo {
    pub id: usize,
    pub title: String,
    pub completed: bool,
}

#[derive(Serialize, Deserialize, Debug)]
pub struct CreateTodo {
    pub title: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct LoginRequest {
    email: String,
    password: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct LoginResponse {
    token: String,
}

#[derive(Debug, Serialize, Deserialize)]
struct Claims {
    id: usize,
    name: String,
    exp: usize,
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/", get(index_handler))
        .route("/todos", get(todos_handler).post(create_todo_handler))
        .route("/login", post(login_handler));

    let addr = SocketAddr::from(([127, 0, 0, 1], 8000));
    println!("listening on http://{}", addr);

    Server::bind(&addr)
        .serve(app.into_make_service())
        .await
        .unwrap();
}

async fn index_handler() -> Html<&'static str> {
    Html("<h1>Hello, World!</h1>")
}

async fn todos_handler() -> Json<Vec<Todo>> {
    Json(vec![
        Todo {
            id: 1,
            title: "Buy milk".to_string(),
            completed: false,
        },
        Todo {
            id: 2,
            title: "Buy eggs".to_string(),
            completed: false,
        },
    ])
}

// "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6MSwibmFtZSI6IlhpYW8gUWlhbyJ9.7PdpzWjZLN4KKNLoM07nfnhKnYdrc0IjumKcOUREXzI"
async fn create_todo_handler(claims: Claims, Json(todo): Json<CreateTodo>) -> StatusCode {
    println!("{:?}", claims);
    println!("{:?}", todo);
    StatusCode::CREATED
}

async fn login_handler(Json(login): Json<LoginRequest>) -> Json<LoginResponse> {
    // skip login info validation
    println!("{:?}", login);

    let claims = Claims {
        id: 1,
        name: "Xiao Qiao".to_string(),
        exp: get_epoch() + 14 * 24 * 60 * 60,
    };
    let key = jwt::EncodingKey::from_secret(SECRET);
    let token = jwt::encode(&jwt::Header::default(), &claims, &key).unwrap();
    Json(LoginResponse { token })
}

// #[async_trait]
// impl<S, B> FromRequest<S, B> for Claims
// where
//     // these bounds are required by `async_trait`
//     B: Send + 'static,
//     S: Send + Sync,
//     // {
//     //     type Rejection = http::StatusCode;

//     //     async fn from_request(req: Request<B>, state: &S) -> Result<Self, Self::Rejection> {
//     //         // ...
//     //         let TypedHeader(Authorization(bearer)) =
//     //             TypedHeader::<Authorization<Bearer>>::from_request(req, state)
//     //                 .await
//     //                 .map_err(|_| http::StatusCode::NETWORK_AUTHENTICATION_REQUIRED)?;

//     //         let key = jwt::DecodingKey::from_secret(SECRET);
//     //         let claims = jwt::decode::<Claims>(bearer.token(), &key, &jwt::Validation::default())
//     //             .map_err(|_| http::StatusCode::UNAUTHORIZED)?;
//     //         Ok(claims.claims)
//     //     }
//     // }
// {
//     type Rejection = HttpError;

//     async fn from_request(req: Request<B>, state: &S) -> Result<Self, Self::Rejection> {
//         // ...
//         let TypedHeader(Authorization(bearer)) =
//             TypedHeader::<Authorization<Bearer>>::from_request(req, state)
//                 .await
//                 .map_err(|_| HttpError::Auth)?;

//         let key = jwt::DecodingKey::from_secret(SECRET);
//         let token = jwt::decode::<Claims>(bearer.token(), &key, &Validation::default())
//             .map_err(|_| HttpError::Auth)?;
//         Ok(token.claims)
//     }
// }

// #[derive(Debug)]
// enum HttpError {
//     Auth,
//     Internal,
//     NotFound,
//     InternalServerError,
// }

// impl IntoResponse for HttpError {
//     fn into_response(self) -> axum::response::Response {
//         let (code, msg) = match self {
//             HttpError::Auth => (StatusCode::UNAUTHORIZED, "Unauthorized"),
//             HttpError::NotFound => (StatusCode::NOT_FOUND, "Not Found"),
//             HttpError::InternalServerError => {
//                 (StatusCode::INTERNAL_SERVER_ERROR, "Internal Server Error")
//             }
//             HttpError::Internal => (StatusCode::INTERNAL_SERVER_ERROR, "Internal Server Error"),
//         };
//         (code, msg).into_response()
//     }
// }

fn get_epoch() -> usize {
    SystemTime::now()
        .duration_since(SystemTime::UNIX_EPOCH)
        .unwrap()
        .as_secs() as usize
}

#[async_trait]
impl<S> FromRequestParts<S> for Claims
where
    S: Send + Sync,
{
    type Rejection = AuthError;

    async fn from_request_parts(parts: &mut Parts, _state: &S) -> Result<Self, Self::Rejection> {
        // Extract the token from the authorization header
        let TypedHeader(Authorization(bearer)) = parts
            .extract::<TypedHeader<Authorization<Bearer>>>()
            .await
            .map_err(|_| AuthError::InvalidToken)?;
        let key = jwt::DecodingKey::from_secret(SECRET);
        // Decode the user data
        let token_data = jwt::decode::<Claims>(bearer.token(), &key, &Validation::default())
            .map_err(|_| AuthError::InvalidToken)?;

        Ok(token_data.claims)
    }
}

impl IntoResponse for AuthError {
    fn into_response(self) -> Response {
        let (status, error_message) = match self {
            // AuthError::WrongCredentials => (StatusCode::UNAUTHORIZED, "Wrong credentials"),
            // AuthError::MissingCredentials => (StatusCode::BAD_REQUEST, "Missing credentials"),
            // AuthError::TokenCreation => (StatusCode::INTERNAL_SERVER_ERROR, "Token creation error"),
            AuthError::InvalidToken => (StatusCode::BAD_REQUEST, "Invalid token"),
        };
        let body = Json(json!({
            "error": error_message,
        }));
        (status, body).into_response()
    }
}

#[derive(Debug)]
enum AuthError {
    // WrongCredentials,
    // MissingCredentials,
    // TokenCreation,
    InvalidToken,
}

```

### basic.http

```http
GET http://localhost:8000/ HTTP/1.1

### 
// todos_handler
GET http://localhost:8000/todos HTTP/1.1

###
POST http://localhost:8000/todos HTTP/1.1
content-type: application/json

{
    "title": "larry"
}

###
POST http://localhost:8000/login HTTP/1.1
content-type: application/json

{
    "email": "xiaoqiao@gmail.com",
    "password": "123456"
}

###
POST http://localhost:8000/todos HTTP/1.1
content-type: application/json
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6MSwibmFtZSI6IlhpYW8gUWlhbyIsImV4cCI6MTY5NDk0NDQzNH0.FhUtNuyUtCA-gtMckc3UkvTu3Z2Ek8DYH1lg8PNXDmk

{
    "title": "hello world"
}

```

响应

```shell
HTTP/1.1 201 Created
content-length: 0
date: Sun, 03 Sep 2023 15:58:29 GMT


```

