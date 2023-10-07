---
title: "Rust Web 全栈开发之编写服务器端 Web 应用"
date: 2023-06-02T23:04:44+08:00
draft: false
tags: ["Rust"]
categories: ["Rust"]
---

# Rust Web 全栈开发之编写服务器端 Web 应用

## 项目结构 和 功能

Web App 教师注册     <->    Web Service

## 主要技术

- 模板引擎： Tera

### 创建项目

```bash
ws on  main via 🦀 1.67.1 via 🅒 base
➜ cargo new webapp
warning: compiling this new package may not work due to invalid workspace configuration

current package believes it's in a workspace when it's not:
current:   /Users/qiaopengjun/rust/ws/webapp/Cargo.toml
workspace: /Users/qiaopengjun/rust/ws/Cargo.toml

this may be fixable by adding `webapp` to the `workspace.members` array of the manifest located at: /Users/qiaopengjun/rust/ws/Cargo.toml
Alternatively, to keep it out of the workspace, add the package to the `workspace.exclude` array, or add an empty `[workspace]` table to the package's manifest.
     Created binary (application) `webapp` package

ws on  main [?] via 🦀 1.67.1 via 🅒 base
➜ c

ws on  main [?] via 🦀 1.67.1 via 🅒 base
➜

```

### 项目目录

```bash
ws on  main [!?] via 🦀 1.67.1 via 🅒 base 
➜ tree -a -I "target|.git"
.
├── .env
├── .gitignore
├── .vscode
│   └── settings.json
├── Cargo.lock
├── Cargo.toml
├── README.md
├── webapp
│   ├── .env
│   ├── Cargo.toml
│   ├── src
│   │   ├── bin
│   │   │   └── svr.rs
│   │   ├── errors.rs
│   │   ├── handlers.rs
│   │   ├── mod.rs
│   │   ├── models.rs
│   │   └── routers.rs
│   └── static
│       ├── css
│       │   └── register.css
│       ├── register.html
│       └── teachers.html
└── webservice
    ├── Cargo.toml
    └── src
        ├── bin
        │   └── teacher-service.rs
        ├── dbaccess
        │   ├── course.rs
        │   ├── mod.rs
        │   └── teacher.rs
        ├── errors.rs
        ├── handlers
        │   ├── course.rs
        │   ├── general.rs
        │   ├── mod.rs
        │   └── teacher.rs
        ├── main.rs
        ├── models
        │   ├── course.rs
        │   ├── mod.rs
        │   └── teacher.rs
        ├── routers.rs
        └── state.rs

13 directories, 33 files

ws on  main [!?] via 🦀 1.67.1 via 🅒 base 
➜ 
```

#### Cargo.toml

```toml
[workspace]
members = ["webservice", "webapp"]

```

#### webapp/Cargo.toml

```rust
[package]
name = "webapp"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
actix-files = "0.6.2"
actix-web = "4.3.1"
awc = "3.1.1"
dotenv = "0.15.0"
serde = { version = "1.0.163", features = ["derive"] }
serde_json = "1.0.96"
tera = "1.18.1"

```

#### webapp/.env

```rust
HOST_PORT=127.0.0.1:8080

```

#### webapp/src/models.rs

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
pub struct TeacherRegisterFrom {
    pub name: String,
    pub imageurl: String,
    pub profile: String,
}

#[derive(Serialize, Deserialize, Debug)]
pub struct TeacherResponse {
    pub id: i32,
    pub name: String,
    pub picture_url: String,
    pub profile: String,
}

```

#### webapp/src/routers.rs

```rust
use crate::handlers::{get_all_teachers, handle_register, show_register_form};
use actix_files as fs;
use actix_web::web;

pub fn app_config(config: &mut web::ServiceConfig) {
    config.service(
        web::scope("")
            .service(fs::Files::new("/static", "./static").show_files_listing())
            .service(web::resource("/").route(web::get().to(get_all_teachers)))
            .service(web::resource("/register").route(web::get().to(show_register_form)))
            .service(web::resource("/register-post").route(web::post().to(handle_register))),
    );
}

```

#### webapp/src/errors.rs

```rust
use actix_web::{error, http::StatusCode, HttpResponse, Result};
use serde::Serialize;
use std::fmt;

#[allow(dead_code)]  // 没有用到 NotFound 不想显示警告
#[derive(Debug, Serialize)]
pub enum MyError {
    ActixError(String),
    NotFound(String),
    TeraError(String),
}

#[derive(Debug, Serialize)]
pub struct MyErrorResponse {
    error_message: String,
}

impl std::error::Error for MyError {}

impl MyError {
    fn error_response(&self) -> String {
        match self {
            MyError::ActixError(msg) => {
                println!("Server error occurred: {}", msg);
                "Internal server error".into()
            }
            MyError::TeraError(msg) => {
                println!("Error in rendering the template: {:?}", msg);
                msg.into()
            }
            MyError::NotFound(msg) => {
                println!("Not found error occurred: {:?}", msg);
                msg.into()
            }
        }
    }
}

impl error::ResponseError for MyError {
    fn status_code(&self) -> StatusCode {
        match self {
            MyError::ActixError(_msg) | MyError::TeraError(_msg) => StatusCode::INTERNAL_SERVER_ERROR,
            MyError::NotFound(_msg) => StatusCode::NOT_FOUND,
        }
    }
    fn error_response(&self) -> HttpResponse {
        HttpResponse::build(self.status_code()).json(MyErrorResponse {
            error_message: self.error_response(),
        })
    }
}

impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter) -> Result<(), fmt::Error> {
        write!(f, "{}", self)
    }
}

impl From<actix_web::error::Error> for MyError {
    fn from(err: actix_web::error::Error) -> Self {
        MyError::ActixError(err.to_string())
    }
}

```

#### webapp/src/mod.rs

```rust
pub mod errors;
pub mod handlers;
pub mod models;
pub mod routers;

```

#### webapp/src/bin/svr.rs

```rust
#[path = "../mod.rs"]
mod wa;
use actix_web::{web, App, HttpServer};
use dotenv::dotenv;
use routers::app_config;
use std::env;
use wa::{errors, handlers, models, routers};

use tera::Tera;

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    dotenv().ok();
    let host_port = env::var("HOST_PORT").expect("HOST:PORT address is not be set");
    println!("Listening on: {}", &host_port);

    HttpServer::new(move || {
        let tera = Tera::new(concat!(env!("CARGO_MANIFEST_DIR"), "/static/**/*")).unwrap();
        App::new()
            .app_data(web::Data::new(tera))
            .configure(app_config)
    })
    .bind(&host_port)?
    .run()
    .await
}

```

#### webapp/src/handlers.rs

```rust
use crate::errors::MyError;
use crate::models::{TeacherRegisterFrom, TeacherResponse};
use actix_web::{web, Error, HttpResponse, Result};
use serde_json::json;

pub async fn get_all_teachers(tmpl: web::Data<tera::Tera>) -> Result<HttpResponse, Error> {
    let awc_client = awc::Client::default(); // 相当于是一个HTTP客户端

    let res = awc_client
        .get("http://localhost:3000/teachers/")
        .send()
        .await
        .unwrap()
        .json::<Vec<TeacherResponse>>()
        .await
        .unwrap();

    let mut ctx = tera::Context::new();
    ctx.insert("error", "");
    ctx.insert("teachers", &res);

    let s = tmpl
        .render("teachers.html", &ctx)
        .map_err(|_| MyError::TeraError("Template error".to_string()))?;
    Ok(HttpResponse::Ok().content_type("text/html").body(s))
}

pub async fn show_register_form(tmpl: web::Data<tera::Tera>) -> Result<HttpResponse, Error> {
    let mut ctx = tera::Context::new();
    ctx.insert("error", "");
    ctx.insert("current_name", "");
    ctx.insert("current_imageurl", "");
    ctx.insert("current_profile", "");
    let s = tmpl
        .render("register.html", &ctx)
        .map_err(|_| MyError::TeraError("Template error".to_string()))?;
    Ok(HttpResponse::Ok().content_type("text/html").body(s))
}

pub async fn handle_register(
    tmpl: web::Data<tera::Tera>,
    params: web::Form<TeacherRegisterFrom>,
) -> Result<HttpResponse, Error> {
    let mut ctx = tera::Context::new();
    let s;
    if params.name == "Dave" {
        ctx.insert("error", "Dave already exists!");

        ctx.insert("current_name", &params.name);
        ctx.insert("current_imageurl", &params.imageurl);
        ctx.insert("current_profile", &params.profile);
        s = tmpl
            .render("register.html", &ctx)
            .map_err(|_| MyError::TeraError("Template error".to_string()))?;
    } else {
        let new_teacher = json!({
            "name": &params.name,
            "picture_url": &params.imageurl,
            "profile": &params.profile
        });
        let awc_client = awc::Client::default();

        let res = awc_client
            .post("http://localhost:3000/teachers/")
            .send_json(&new_teacher)
            .await
            .unwrap()
            .body()
            .await?;

        let teacher_response: TeacherResponse = serde_json::from_str(&std::str::from_utf8(&res)?)?;
        s = format!("Congratulations! Your Id is: {}.", teacher_response.id);
    }

    Ok(HttpResponse::Ok().content_type("text/html").body(s))
}

```

#### webapp/src/static/teachers.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Teachers</title>
</head>

<body>
    <h1>Teacher list</h1>
    <ol>
        {% for t in teachers %}
        <li>
            <h5>{{t.name}}</h5>
            <div>{{t.profile}}</div>
        </li>
        {% endfor %}
    </ol>

    <div style="margin-top: 20px">
        <a href="/register">Register a teacher</a>
    </div>
</body>

</html>

```

#### webapp/src/static/register.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Teacher registration</title>
    <link rel="stylesheet" href="/static/css/register.css">
</head>

<body>
    <h2 class="header">Teacher registration</h2>
    <div class="center">
        <form action="/register-post" method="POST">
            <label for="name">Teacher name</label><br />
            <input type="text" name="name" id="name" value="{{current_name}}" maxlength="15"><br />
            <label for="imageurl">Teacher image url</label><br />
            <input type="text" name="imageurl" id="imageurl" value="{{current_imageurl}}"><br />
            <label for="profile">Brief profile</label><br />
            <input type="text" name="profile" id="profile" value="{{current_profile}}" maxlength="15"><br />
            <label for="error">
                <p style="color: red">{{error}}</p>
            </label>
            <br />
            <button type="submit" id="button1">Register</button>
        </form>
    </div>
</body>

</html>

```

#### webapp/src/static/css/register.css

```css
body {
    background-color: #f4f4f4;
    font-family: Arial, Helvetica, sans-serif;
    font-size: 16px;
}

.center {
    margin: 0 auto;
    width: 60%;
}

.header {
    font-size: 2.5em;
    text-align: center;
    margin-top: 40px;
}

form {
    background-color: #fff;
    border: 1px solid #ddd;
    border-radius: 5px;
    padding: 20px;
    margin-top: 30px;
}

form label {
    font-size: 1.2em;
    font-weight: bold;
    display: block;
    margin-top: 15px;
}

input[type=text] {
    width: 100%;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 5px;
    margin-top: 5px;
}

button[type=submit] {
    background-color: #4CAF50;
    color: #fff;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    margin-top: 15px;
    cursor: pointer;
    font-size: 1.2em;
    font-weight: bold;
}

button[type=submit]:hover {
    background-color: #3e8e41;
}

```

### 运行

```bash
ws/webservice on  main [!?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 17m 54.3s 
➜ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.14s
     Running `/Users/qiaopengjun/rust/ws/target/debug/teacher-service`

ws/webapp on  main [!?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 1m 4.1s 
➜ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.11s
     Running `/Users/qiaopengjun/rust/ws/target/debug/svr`
Listening on: 127.0.0.1:8080


```

访问：<http://localhost:8080/>

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306031301737.png)

点击：[Register a teacher](http://localhost:8080/register)

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306031315803.png)

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306031316117.png)

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img202306031325888.png)
