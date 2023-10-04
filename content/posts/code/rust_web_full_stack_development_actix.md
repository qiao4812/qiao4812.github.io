---
title: "Rust Web 全栈开发之 Actix 尝鲜并构建REST API"
date: 2023-05-28T00:39:31+08:00
draft: true
tags: ["Rust"]
categories: ["Rust"]
---

# Rust Web 全栈开发之 Actix 尝鲜并构建REST API

## 一、Actix 尝鲜

### 需要使用的crate

- actix-web v4.3.1
- actix-rt v2.8.0

```bash
~ via 🅒 base
➜ cd rust

~/rust via 🅒 base
➜ cargo new ws  # workspace
     Created binary (application) `ws` package

~/rust via 🅒 base
➜ cd ws

ws on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base
➜ c

ws on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base
➜

ws on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base
➜ cargo new webservice
     Created binary (application) `webservice` package

ws on  master [?] via 🦀 1.67.1 via 🅒 base
➜

```

目录

```bash
ws on  master [?] via 🦀 1.67.1 via 🅒 base
➜ tree -I target
.
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
└── webservice
    ├── Cargo.toml
    └── src
        ├── bin
        │   └── server1.rs
        └── main.rs

5 directories, 6 files

ws on  master [?] via 🦀 1.67.1 via 🅒 base
➜
```

Cargo.toml

```toml
[workspace]
members = ["webservice"]

```

webservice/Cargo.toml

```toml
[package]
name = "webservice"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
actix-web = "4.3.1"
actix-rt = "2.8.0"

[[bin]]
name = "server1"

```



webservice/src/bin/server1.rs

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};
use std::io;

// 配置 route
pub fn general_routes(cfg: &mut web::ServiceConfig) {
    cfg.route("/health", web::get().to(health_check_handler));
}

// 配置 handler
pub async fn health_check_handler() -> impl Responder {
    HttpResponse::Ok().json("Actix Web Service is running!")
}

// 实例化 HTTP Server 并运行
#[actix_rt::main]
async fn main() -> io::Result<()> {
    // 构建 App 配置 route
    let app = move || App::new().configure(general_routes);

    // 运行 HTTP Server
    HttpServer::new(app).bind("127.0.0.1:3000")?.run().await
}

```

运行

```bash
ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cargo run -p webservice --bin server1
   Compiling actix-rt v2.8.0
   Compiling actix-http v3.3.1
   Compiling actix-server v2.2.0
   Compiling actix-web v4.3.1
   Compiling webservice v0.1.0 (/Users/qiaopengjun/rust/ws/webservice)
    Finished dev [unoptimized + debuginfo] target(s) in 4.26s
     Running `target/debug/server1`


ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cd webservice                        

ws/webservice on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ cargo run --bin server1              
    Finished dev [unoptimized + debuginfo] target(s) in 0.15s
     Running `/Users/qiaopengjun/rust/ws/target/debug/server1`

```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202305281240446.png)

### Actix的基本组件

客户端浏览器 互联网  Actix HTTP Server 

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202305281333790.png)

Actix的并发（concurrency）

- Actix支持两类并发：
  - 异步I/O：给定的OS原生线程在等待I/O时执行其他任务（例如侦听网络连接）
  - 多线程并行：默认情况下启动OS原生线程的数量与系统逻辑CPU数量相同

## 二、构建REST API

### 需要使用的crate

- serde, v1.0.163
- chrono, v0.4.24

### 构建的内容

- POST: /courses
- GET:/courses/teacher_id
- GET:/courses/teacher_id/course_id

### 相关文件

- bin/teacher-service.rs
- models.rs
- state.rs
- routers.rs
- handlers.rs

目录

```bash
ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ tree -I target
.
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
└── webservice
    ├── Cargo.toml
    └── src
        ├── bin
        │   ├── server1.rs
        │   └── teacher-service.rs
        ├── handlers.rs
        ├── main.rs
        ├── models.rs
        ├── routers.rs
        └── state.rs

5 directories, 11 files

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 
```

webservice/Cargo.toml

```toml
[package]
name = "webservice"
version = "0.1.0"
edition = "2021"
default-run = "teacher-service"
# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
actix-web = "4.3.1"
actix-rt = "2.8.0"

[[bin]]
name = "server1"

[[bin]]
name = "teacher-service"

```

webservice/src/bin/teacher-service.rs

```rust
use actix_web::{web, App, HttpServer};
use std::io;
use std::sync::Mutex;

#[path = "../handlers.rs"]
mod handlers;
#[path = "../routers.rs"]
mod routers;
#[path = "../state.rs"]
mod state;

use routers::*;
use state::AppState;

#[actix_rt::main]
async fn main() -> io::Result<()> {
    let shared_data = web::Data::new(AppState {
        health_check_response: "I'm Ok.".to_string(),
        visit_count: Mutex::new(0),
    });
    let app = move || {
        App::new()
            .app_data(shared_data.clone())
            .configure(general_routes)
    };

    HttpServer::new(app).bind("127.0.0.1:3000")?.run().await
}

```

webservice/src/handlers.rs

```rust
use super::state::AppState;
use actix_web::{web, HttpResponse};

pub async fn health_check_handler(app_state: web::Data<AppState>) -> HttpResponse {
    let health_check_response = &app_state.health_check_response;
    let mut visit_count = app_state.visit_count.lock().unwrap();
    let response = format!("{} {} times", health_check_response, visit_count);
    *visit_count += 1;
    HttpResponse::Ok().json(&response)
}

```

webservice/src/routers.rs

```rust
use super::handlers::*;
use actix_web::web;

pub fn general_routes(cfg: &mut web::ServiceConfig) {
    cfg.route("/health", web::get().to(health_check_handler));
}

```

webservice/src/state.rs

```rust
use std::sync::Mutex;

pub struct AppState {
    pub health_check_response: String,
    pub visit_count: Mutex<u32>,
}

```

运行

```bash
ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cd webservice 

ws/webservice on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ cargo run              
   Compiling webservice v0.1.0 (/Users/qiaopengjun/rust/ws/webservice)
    Finished dev [unoptimized + debuginfo] target(s) in 1.49s
     Running `/Users/qiaopengjun/rust/ws/target/debug/teacher-service`

```

请求访问

```bash

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ curl localhost:3000/health 
"I'm Ok. 0 times"%                                                                                           

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ curl localhost:3000/health
"I'm Ok. 1 times"%                                                                                           

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ curl localhost:3000/health
"I'm Ok. 2 times"%                                                                                           

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 

```

webservice/Cargo.toml

```toml
[package]
name = "webservice"
version = "0.1.0"
edition = "2021"
default-run = "teacher-service"
# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
actix-web = "4.3.1"
actix-rt = "2.8.0"
serde = { version = "1.0.163", features = ["derive"] }
chrono = { version = "0.4.24", features = ["serde"] }

[[bin]]
name = "server1"

[[bin]]
name = "teacher-service"

```

webservice/src/models.rs

```rust
use actix_web::web;
use chrono::NaiveDateTime;
use serde::{Deserialize, Serialize};

#[derive(Deserialize, Serialize, Debug, Clone)]
pub struct Course {
    pub teacher_id: usize,
    pub id: Option<usize>,
    pub name: String,
    pub time: Option<NaiveDateTime>,
}
impl From<web::Json<Course>> for Course {
    fn from(course: web::Json<Course>) -> Self {
        Course {
            teacher_id: course.teacher_id,
            id: course.id,
            name: course.name.clone(),
            time: course.time,
        }
    }
}

```

webservice/src/state.rs

```rust
use super::models::Course;
use std::sync::Mutex;

pub struct AppState {
    pub health_check_response: String,
    pub visit_count: Mutex<u32>,
    pub courses: Mutex<Vec<Course>>,
}

```

webservice/src/bin/teacher-service.rs

```rust
use actix_web::{web, App, HttpServer};
use std::io;
use std::sync::Mutex;

#[path = "../handlers.rs"]
mod handlers;
#[path = "../models.rs"]
mod models;
#[path = "../routers.rs"]
mod routers;
#[path = "../state.rs"]
mod state;

use routers::*;
use state::AppState;

#[actix_rt::main]
async fn main() -> io::Result<()> {
    let shared_data = web::Data::new(AppState {
        health_check_response: "I'm Ok.".to_string(),
        visit_count: Mutex::new(0),
        courses: Mutex::new(vec![]),
    });
    let app = move || {
        App::new()
            .app_data(shared_data.clone())
            .configure(general_routes)
    };

    HttpServer::new(app).bind("127.0.0.1:3000")?.run().await
}

```

运行并访问

```bash
ws/webservice on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 2h 37m 54.5s 
➜ cargo run
   Compiling webservice v0.1.0 (/Users/qiaopengjun/rust/ws/webservice)
    Finished dev [unoptimized + debuginfo] target(s) in 1.54s
     Running `/Users/qiaopengjun/rust/ws/target/debug/teacher-service`


ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ curl localhost:3000/health
"I'm Ok. 0 times"%                                                                                           

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 

```

### 添加课程信息

webservice/src/routers.rs

```rust
use super::handlers::*;
use actix_web::web;

pub fn general_routes(cfg: &mut web::ServiceConfig) {
    cfg.route("/health", web::get().to(health_check_handler));
}

pub fn course_routes(cfg: &mut web::ServiceConfig) {
    // courses 是一套资源的根路径
    cfg.service(web::scope("/courses").route("/", web::post().to(new_course)));
}

```

webservice/src/bin/teacher-service.rs

```rust
use actix_web::{web, App, HttpServer};
use std::io;
use std::sync::Mutex;

#[path = "../handlers.rs"]
mod handlers;
#[path = "../models.rs"]
mod models;
#[path = "../routers.rs"]
mod routers;
#[path = "../state.rs"]
mod state;

use routers::*;
use state::AppState;

#[actix_rt::main]
async fn main() -> io::Result<()> {
    let shared_data = web::Data::new(AppState {
        health_check_response: "I'm Ok.".to_string(),
        visit_count: Mutex::new(0),
        courses: Mutex::new(vec![]),
    });
    let app = move || {
        App::new()
            .app_data(shared_data.clone())
            .configure(general_routes)
            .configure(course_routes) // 路由注册
    };

    HttpServer::new(app).bind("127.0.0.1:3000")?.run().await
}

```

webservice/src/handlers.rs

```rust
use super::state::AppState;
use actix_web::{web, HttpResponse};

pub async fn health_check_handler(app_state: web::Data<AppState>) -> HttpResponse {
    let health_check_response = &app_state.health_check_response;
    let mut visit_count = app_state.visit_count.lock().unwrap();
    let response = format!("{} {} times", health_check_response, visit_count);
    *visit_count += 1;
    HttpResponse::Ok().json(&response)
}

use super::models::Course;
use chrono::Utc;

pub async fn new_course(
    new_course: web::Json<Course>,
    app_state: web::Data<AppState>,
) -> HttpResponse {
    println!("Received new course");
    let course_count = app_state
        .courses
        .lock()
        .unwrap()
        .clone()
        .into_iter()
        .filter(|course| course.teacher_id == new_course.teacher_id)
        .collect::<Vec<Course>>()
        .len();
    let new_course = Course {
        // 创建一个新的课程
        teacher_id: new_course.teacher_id,
        id: Some(course_count + 1),
        name: new_course.name.clone(),
        time: Some(Utc::now().naive_utc()), // 当前时间
    };
    app_state.courses.lock().unwrap().push(new_course);
    HttpResponse::Ok().json("Course added")
}

#[cfg(test)]
mod tests {
    use super::*;
    use actix_web::http::StatusCode;
    use std::sync::Mutex;

    #[actix_rt::test] // 异步测试
    async fn post_course_test() {
        let course = web::Json(Course {
            teacher_id: 1,
            name: "Test course".into(),
            id: None,
            time: None,
        });
        let app_state: web::Data<AppState> = web::Data::new(AppState {
            health_check_response: "".to_string(),
            visit_count: Mutex::new(0),
            courses: Mutex::new(vec![]),
        });
        let resp = new_course(course, app_state).await;
        assert_eq!(resp.status(), StatusCode::OK);
    }
}


```

运行并测试

```bash
ws/webservice on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 41m 26.8s 
➜ cargo test        
   Compiling webservice v0.1.0 (/Users/qiaopengjun/rust/ws/webservice)
    Finished test [unoptimized + debuginfo] target(s) in 0.68s
     Running unittests src/bin/server1.rs (/Users/qiaopengjun/rust/ws/target/debug/deps/server1-db3d08c1708d3a7c)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/bin/teacher-service.rs (/Users/qiaopengjun/rust/ws/target/debug/deps/teacher_service-41d6f77eb4f5c36e)

running 1 test
test handlers::tests::post_course_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (/Users/qiaopengjun/rust/ws/target/debug/deps/webservice-fd79900ffad88ae5)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


ws/webservice on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 2.7s 
➜ cargo run 
   Compiling webservice v0.1.0 (/Users/qiaopengjun/rust/ws/webservice)
    Finished dev [unoptimized + debuginfo] target(s) in 1.04s
     Running `/Users/qiaopengjun/rust/ws/target/debug/teacher-service`
Received new course


ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ curl -X POST localhost:3000/courses/ -H "Content-Type: application/json" -d '{"teacher_id":1, "name":"First course"}' 
"Course added"%                                                                                                                   

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 
```

### 查询所有课程信息

webservice/src/routers.rs

```rust
use super::handlers::*;
use actix_web::web;

pub fn general_routes(cfg: &mut web::ServiceConfig) {
    cfg.route("/health", web::get().to(health_check_handler));
}

pub fn course_routes(cfg: &mut web::ServiceConfig) {
    // courses 是一套资源的根路径
    cfg.service(
        web::scope("/courses")
            .route("/", web::post().to(new_course))
            .route("/{user_id}", web::get().to(get_courses_for_tescher)),
    );
}

```

webservice/src/handlers.rs

```rust
use super::state::AppState;
use actix_web::{web, HttpResponse};

pub async fn health_check_handler(app_state: web::Data<AppState>) -> HttpResponse {
    let health_check_response = &app_state.health_check_response;
    let mut visit_count = app_state.visit_count.lock().unwrap();
    let response = format!("{} {} times", health_check_response, visit_count);
    *visit_count += 1;
    HttpResponse::Ok().json(&response)
}

use super::models::Course;
use chrono::Utc;

pub async fn new_course(
    new_course: web::Json<Course>,
    app_state: web::Data<AppState>,
) -> HttpResponse {
    println!("Received new course");
    let course_count = app_state
        .courses
        .lock()
        .unwrap()
        .clone()
        .into_iter()
        .filter(|course| course.teacher_id == new_course.teacher_id)
        .collect::<Vec<Course>>()
        .len();
    let new_course = Course {
        // 创建一个新的课程
        teacher_id: new_course.teacher_id,
        id: Some(course_count + 1),
        name: new_course.name.clone(),
        time: Some(Utc::now().naive_utc()), // 当前时间
    };
    app_state.courses.lock().unwrap().push(new_course);
    HttpResponse::Ok().json("Course added")
}

pub async fn get_courses_for_tescher(
    app_state: web::Data<AppState>,
    params: web::Path<(usize)>,
) -> HttpResponse {
    let teacher_id: usize = params.into_inner();

    let filtered_courses = app_state
        .courses
        .lock()
        .unwrap()
        .clone()
        .into_iter()
        .filter(|course| course.teacher_id == teacher_id)
        .collect::<Vec<Course>>();

    if filtered_courses.len() > 0 {
        HttpResponse::Ok().json(filtered_courses)
    } else {
        HttpResponse::Ok().json("No courses found for teacher".to_string())
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    use actix_web::http::StatusCode;
    use std::sync::Mutex;

    #[actix_rt::test] // 异步测试
    async fn post_course_test() {
        let course = web::Json(Course {
            teacher_id: 1,
            name: "Test course".into(),
            id: None,
            time: None,
        });
        let app_state: web::Data<AppState> = web::Data::new(AppState {
            health_check_response: "".to_string(),
            visit_count: Mutex::new(0),
            courses: Mutex::new(vec![]),
        });
        let resp = new_course(course, app_state).await;
        assert_eq!(resp.status(), StatusCode::OK);
    }

    #[actix_rt::test]
    async fn get_all_course_success() {
        let app_state: web::Data<AppState> = web::Data::new(AppState {
            health_check_response: "".to_string(),
            visit_count: Mutex::new(0),
            courses: Mutex::new(vec![]),
        });
        let teacher_id: web::Path<(usize)> = web::Path::from((1));
        let resp = get_courses_for_tescher(app_state, teacher_id).await;
        assert_eq!(resp.status(), StatusCode::OK);
    }
}

```



测试

```bash
ws/webservice on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 1m 1.8s 
➜ cargo test
   Compiling webservice v0.1.0 (/Users/qiaopengjun/rust/ws/webservice)
warning: unnecessary parentheses around type
  --> webservice/src/bin/../handlers.rs:42:23
   |
42 |     params: web::Path<(usize)>,
   |                       ^     ^
   |
   = note: `#[warn(unused_parens)]` on by default
help: remove these parentheses
   |
42 -     params: web::Path<(usize)>,
42 +     params: web::Path<usize>,
   |

warning: unnecessary parentheses around type
  --> webservice/src/bin/../handlers.rs:92:35
   |
92 |         let teacher_id: web::Path<(usize)> = web::Path::from((1));
   |                                   ^     ^
   |
help: remove these parentheses
   |
92 -         let teacher_id: web::Path<(usize)> = web::Path::from((1));
92 +         let teacher_id: web::Path<usize> = web::Path::from((1));
   |

warning: unnecessary parentheses around function argument
  --> webservice/src/bin/../handlers.rs:92:62
   |
92 |         let teacher_id: web::Path<(usize)> = web::Path::from((1));
   |                                                              ^ ^
   |
help: remove these parentheses
   |
92 -         let teacher_id: web::Path<(usize)> = web::Path::from((1));
92 +         let teacher_id: web::Path<(usize)> = web::Path::from(1);
   |

warning: `webservice` (bin "teacher-service" test) generated 3 warnings (run `cargo fix --bin "teacher-service" --tests` to apply 3 suggestions)
    Finished test [unoptimized + debuginfo] target(s) in 0.60s
     Running unittests src/bin/server1.rs (/Users/qiaopengjun/rust/ws/target/debug/deps/server1-db3d08c1708d3a7c)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/bin/teacher-service.rs (/Users/qiaopengjun/rust/ws/target/debug/deps/teacher_service-41d6f77eb4f5c36e)

running 2 tests
test handlers::tests::post_course_test ... ok
test handlers::tests::get_all_course_success ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (/Users/qiaopengjun/rust/ws/target/debug/deps/webservice-fd79900ffad88ae5)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


ws/webservice on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ 


ws/webservice on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ cargo run 
   Compiling webservice v0.1.0 (/Users/qiaopengjun/rust/ws/webservice)
warning: unnecessary parentheses around type
  --> webservice/src/bin/../handlers.rs:42:23
   |
42 |     params: web::Path<(usize)>,
   |                       ^     ^
   |
   = note: `#[warn(unused_parens)]` on by default
help: remove these parentheses
   |
42 -     params: web::Path<(usize)>,
42 +     params: web::Path<usize>,
   |

warning: `webservice` (bin "teacher-service") generated 1 warning (run `cargo fix --bin "teacher-service"` to apply 1 suggestion)
    Finished dev [unoptimized + debuginfo] target(s) in 1.54s
     Running `/Users/qiaopengjun/rust/ws/target/debug/teacher-service`
Received new course
Received new course
Received new course



ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ curl -X POST localhost:3000/courses/ -H "Content-Type: application/json" -d '{"teacher_id":1, "name":"First course"}'
"Course added"%                                                                                                                   

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ curl -X POST localhost:3000/courses/ -H "Content-Type: application/json" -d '{"teacher_id":1, "name":"Second course"}'
"Course added"%                                                                                                                   

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ curl -X POST localhost:3000/courses/ -H "Content-Type: application/json" -d '{"teacher_id":1, "name":"Third course"}' 
"Course added"%                                                                                                                   

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ curl localhost:3000/courses/1                                                                                        
[{"teacher_id":1,"id":1,"name":"First course","time":"2023-05-28T11:16:50.312820"},{"teacher_id":1,"id":2,"name":"Second course","time":"2023-05-28T11:17:08.358168"},{"teacher_id":1,"id":3,"name":"Third course","time":"2023-05-28T11:17:23.295881"}]%           

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 

```

### 查询单个课程信息

webservice/src/routers.rs

```rust
use super::handlers::*;
use actix_web::web;

pub fn general_routes(cfg: &mut web::ServiceConfig) {
    cfg.route("/health", web::get().to(health_check_handler));
}

pub fn course_routes(cfg: &mut web::ServiceConfig) {
    // courses 是一套资源的根路径
    cfg.service(
        web::scope("/courses")
            .route("/", web::post().to(new_course))
            .route("/{user_id}", web::get().to(get_courses_for_tescher))
            .route("/{user_id}/{course_id}", web::get().to(get_courses_detail)),
    );
}

```

webservice/src/handlers.rs

```rust
use super::state::AppState;
use actix_web::{web, HttpResponse};

pub async fn health_check_handler(app_state: web::Data<AppState>) -> HttpResponse {
    let health_check_response = &app_state.health_check_response;
    let mut visit_count = app_state.visit_count.lock().unwrap();
    let response = format!("{} {} times", health_check_response, visit_count);
    *visit_count += 1;
    HttpResponse::Ok().json(&response)
}

use super::models::Course;
use chrono::Utc;

pub async fn new_course(
    new_course: web::Json<Course>,
    app_state: web::Data<AppState>,
) -> HttpResponse {
    println!("Received new course");
    let course_count = app_state
        .courses
        .lock()
        .unwrap()
        .clone()
        .into_iter()
        .filter(|course| course.teacher_id == new_course.teacher_id)
        .collect::<Vec<Course>>()
        .len();
    let new_course = Course {
        // 创建一个新的课程
        teacher_id: new_course.teacher_id,
        id: Some(course_count + 1),
        name: new_course.name.clone(),
        time: Some(Utc::now().naive_utc()), // 当前时间
    };
    app_state.courses.lock().unwrap().push(new_course);
    HttpResponse::Ok().json("Course added")
}

pub async fn get_courses_for_tescher(
    app_state: web::Data<AppState>,
    params: web::Path<usize>,
) -> HttpResponse {
    let teacher_id: usize = params.into_inner();

    let filtered_courses = app_state
        .courses
        .lock()
        .unwrap()
        .clone()
        .into_iter()
        .filter(|course| course.teacher_id == teacher_id)
        .collect::<Vec<Course>>();

    if filtered_courses.len() > 0 {
        HttpResponse::Ok().json(filtered_courses)
    } else {
        HttpResponse::Ok().json("No courses found for teacher".to_string())
    }
}

pub async fn get_courses_detail(
    app_state: web::Data<AppState>,
    params: web::Path<(usize, usize)>,
) -> HttpResponse {
    let (teacher_id, course_id) = params.into_inner();
    let selected_course = app_state
        .courses
        .lock()
        .unwrap()
        .clone()
        .into_iter()
        .find(|x| x.teacher_id == teacher_id && x.id == Some(course_id))
        .ok_or("Course not found"); // Option 类型 转化成 Result<T, E> 类型

    if let Ok(course) = selected_course {
        HttpResponse::Ok().json(course)
    } else {
        HttpResponse::Ok().json("Course not found".to_string())
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    use actix_web::http::StatusCode;
    use std::sync::Mutex;

    #[actix_rt::test] // 异步测试
    async fn post_course_test() {
        let course = web::Json(Course {
            teacher_id: 1,
            name: "Test course".into(),
            id: None,
            time: None,
        });
        let app_state: web::Data<AppState> = web::Data::new(AppState {
            health_check_response: "".to_string(),
            visit_count: Mutex::new(0),
            courses: Mutex::new(vec![]),
        });
        let resp = new_course(course, app_state).await;
        assert_eq!(resp.status(), StatusCode::OK);
    }

    #[actix_rt::test]
    async fn get_all_course_success() {
        let app_state: web::Data<AppState> = web::Data::new(AppState {
            health_check_response: "".to_string(),
            visit_count: Mutex::new(0),
            courses: Mutex::new(vec![]),
        });
        let teacher_id: web::Path<usize> = web::Path::from(1);
        let resp = get_courses_for_tescher(app_state, teacher_id).await;
        assert_eq!(resp.status(), StatusCode::OK);
    }

    #[actix_rt::test]
    async fn get_one_course_success() {
        let app_state: web::Data<AppState> = web::Data::new(AppState {
            health_check_response: "".to_string(),
            visit_count: Mutex::new(0),
            courses: Mutex::new(vec![]),
        });
        let params: web::Path<(usize, usize)> = web::Path::from((1, 1));
        let resp = get_courses_detail(app_state, params).await;
        assert_eq!(resp.status(), StatusCode::OK);
    }
}

```

运行测试

```bash
ws/webservice on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 18m 27.8s 
➜ cargo test
   Compiling webservice v0.1.0 (/Users/qiaopengjun/rust/ws/webservice)
    Finished test [unoptimized + debuginfo] target(s) in 1.02s
     Running unittests src/bin/server1.rs (/Users/qiaopengjun/rust/ws/target/debug/deps/server1-db3d08c1708d3a7c)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/bin/teacher-service.rs (/Users/qiaopengjun/rust/ws/target/debug/deps/teacher_service-41d6f77eb4f5c36e)

running 3 tests
test handlers::tests::get_one_course_success ... ok
test handlers::tests::get_all_course_success ... ok
test handlers::tests::post_course_test ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (/Users/qiaopengjun/rust/ws/target/debug/deps/webservice-fd79900ffad88ae5)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


ws/webservice on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ cargo run                                                                                                             
   Compiling webservice v0.1.0 (/Users/qiaopengjun/rust/ws/webservice)
    Finished dev [unoptimized + debuginfo] target(s) in 0.86s
     Running `/Users/qiaopengjun/rust/ws/target/debug/teacher-service`
Received new course
Received new course


ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ curl -X POST localhost:3000/courses/ -H "Content-Type: application/json" -d '{"teacher_id":1, "name":"First course"}' 
"Course added"%                                                                                                                   

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ curl -X POST localhost:3000/courses/ -H "Content-Type: application/json" -d '{"teacher_id":1, "name":"Second course"}'
"Course added"%                                                                                                                   

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ curl localhost:3000/courses/1/1                                                                                       
{"teacher_id":1,"id":1,"name":"First course","time":"2023-05-28T11:35:49.260822"}%                                                

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 

```

