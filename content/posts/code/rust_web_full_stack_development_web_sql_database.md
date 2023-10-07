---
title: "Rust Web 全栈开发之在 Web 项目中使用数据库"
date: 2023-05-28T22:57:46+08:00
draft: false
tags: ["Rust"]
categories: ["Rust"]
---

# Rust Web 全栈开发之在 Web 项目中使用数据库

### 目录

```bash
ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ tree -a  -I target
.
├── .git
├── .gitignore
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
└── webservice
    ├── Cargo.toml
    ├── .env
    └── src
        ├── bin
        │   ├── server1.rs
        │   └── teacher-service.rs
        ├── handlers.rs
        ├── main.rs
        ├── models.rs
        ├── routers.rs
        └── state.rs

14 directories, 18 files

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 
```

### webservice/Cargo.toml

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
dotenv = "0.15.0"
serde = { version = "1.0.163", features = ["derive"] }
chrono = { version = "0.4.24", features = ["serde"] }
openssl = { version = "0.10.52", features = ["vendored"] }
sqlx = { version = "0.6.3", default_features = false, features = [
    "postgres",
    "runtime-tokio-rustls",
    "macros",
    "chrono",
] }

[[bin]]
name = "server1"

[[bin]]
name = "teacher-service"

```

### webservice/src/state.rs

```rust
// use super::models::Course;
use sqlx::postgres::PgPool;
use std::sync::Mutex;

pub struct AppState {
    pub health_check_response: String,
    pub visit_count: Mutex<u32>,
    // pub courses: Mutex<Vec<Course>>,
    pub db: PgPool,
}

```

### webservice/src/bin/teacher-service.rs

```rust
use actix_web::{web, App, HttpServer};
use dotenv::dotenv;
use sqlx::postgres::PgPoolOptions;
use std::env;
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
    dotenv().ok();

    let database_url = env::var("DATABASE_URL").expect("DATABASE_URL is not set.");
    let db_pool = PgPoolOptions::new().connect(&database_url).await.unwrap();

    let shared_data = web::Data::new(AppState {
        health_check_response: "I'm Ok.".to_string(),
        visit_count: Mutex::new(0),
        // courses: Mutex::new(vec![]),
        db: db_pool,
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

### webservice/src/handlers.rs

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
    HttpResponse::Ok().json("Success")
}

pub async fn get_courses_for_tescher(
    app_state: web::Data<AppState>,
    params: web::Path<usize>,
) -> HttpResponse {
    HttpResponse::Ok().json("Success")
}

pub async fn get_courses_detail(
    app_state: web::Data<AppState>,
    params: web::Path<(usize, usize)>,
) -> HttpResponse {
    HttpResponse::Ok().json("Success")
}

```

### webservice/src/.env

```bash
DATABASE_URL=postgres://postgres:postgres@127.0.0.1:5432/postgres

```

### 检查

```bash
ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cargo check --bin teacher-service
warning: unused import: `chrono::Utc`
  --> webservice/src/bin/../handlers.rs:13:5
   |
13 | use chrono::Utc;
   |     ^^^^^^^^^^^
   |
   = note: `#[warn(unused_imports)]` on by default

warning: unused variable: `new_course`
  --> webservice/src/bin/../handlers.rs:16:5
   |
16 |     new_course: web::Json<Course>,
   |     ^^^^^^^^^^ help: if this is intentional, prefix it with an underscore: `_new_course`
   |
   = note: `#[warn(unused_variables)]` on by default

warning: unused variable: `app_state`
  --> webservice/src/bin/../handlers.rs:17:5
   |
17 |     app_state: web::Data<AppState>,
   |     ^^^^^^^^^ help: if this is intentional, prefix it with an underscore: `_app_state`

warning: unused variable: `app_state`
  --> webservice/src/bin/../handlers.rs:23:5
   |
23 |     app_state: web::Data<AppState>,
   |     ^^^^^^^^^ help: if this is intentional, prefix it with an underscore: `_app_state`

warning: unused variable: `params`
  --> webservice/src/bin/../handlers.rs:24:5
   |
24 |     params: web::Path<usize>,
   |     ^^^^^^ help: if this is intentional, prefix it with an underscore: `_params`

warning: unused variable: `app_state`
  --> webservice/src/bin/../handlers.rs:30:5
   |
30 |     app_state: web::Data<AppState>,
   |     ^^^^^^^^^ help: if this is intentional, prefix it with an underscore: `_app_state`

warning: unused variable: `params`
  --> webservice/src/bin/../handlers.rs:31:5
   |
31 |     params: web::Path<(usize, usize)>,
   |     ^^^^^^ help: if this is intentional, prefix it with an underscore: `_params`

warning: `webservice` (bin "teacher-service") generated 7 warnings (run `cargo fix --bin "teacher-service"` to apply 7 suggestions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.11s

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 
```

### webservice/src/handlers.rs

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
    HttpResponse::Ok().json("Success")
}

pub async fn get_courses_for_tescher(
    app_state: web::Data<AppState>,
    params: web::Path<usize>,
) -> HttpResponse {
    HttpResponse::Ok().json("Success")
}

pub async fn get_courses_detail(
    app_state: web::Data<AppState>,
    params: web::Path<(usize, usize)>,
) -> HttpResponse {
    HttpResponse::Ok().json("Success")
}

#[cfg(test)]
mod tests {
    use super::*;
    use actix_web::http::StatusCode;
    use chrono::NaiveDateTime;
    use dotenv::dotenv;
    use sqlx::postgres::PgPoolOptions;
    use std::env;
    use std::sync::Mutex;

    #[actix_rt::test] // 异步测试
    async fn post_course_test() {
        dotenv().ok();
        let db_url = env::var("DATABASE_URL").expect("DATABASE_URL is not set");
        let db_pool = PgPoolOptions::new().connect(&db_url).await.unwrap();
        let app_state: web::Data<AppState> = web::Data::new(AppState {
            health_check_response: "".to_string(),
            visit_count: Mutex::new(0),
            db: db_pool,
        });

        let course = web::Json(Course {
            teacher_id: 1,
            name: "Test course".into(),
            id: None,
            time: None,
        });

        let resp = new_course(course, app_state).await;
        assert_eq!(resp.status(), StatusCode::OK);
    }

    #[actix_rt::test]
    async fn get_all_course_success() {
        dotenv().ok();
        let db_url = env::var("DATABASE_URL").expect("DATABASE_URL is not set");
        let db_pool = PgPoolOptions::new().connect(&db_url).await.unwrap();
        let app_state: web::Data<AppState> = web::Data::new(AppState {
            health_check_response: "".to_string(),
            visit_count: Mutex::new(0),
            db: db_pool,
        });
        let teacher_id: web::Path<usize> = web::Path::from(1);
        let resp = get_courses_for_tescher(app_state, teacher_id).await;
        assert_eq!(resp.status(), StatusCode::OK);
    }

    #[actix_rt::test]
    async fn get_one_course_success() {
        dotenv().ok();
        let db_url = env::var("DATABASE_URL").expect("DATABASE_URL is not set");
        let db_pool = PgPoolOptions::new().connect(&db_url).await.unwrap();
        let app_state: web::Data<AppState> = web::Data::new(AppState {
            health_check_response: "".to_string(),
            visit_count: Mutex::new(0),
            db: db_pool,
        });
        let params: web::Path<(usize, usize)> = web::Path::from((1, 1));
        let resp = get_courses_detail(app_state, params).await;
        assert_eq!(resp.status(), StatusCode::OK);
    }
}

```

### 测试

```bash
ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cargo test --bin teacher-service
warning: unused import: `chrono::Utc`
  --> webservice/src/bin/../handlers.rs:13:5
   |
13 | use chrono::Utc;
   |     ^^^^^^^^^^^
   |
   = note: `#[warn(unused_imports)]` on by default

warning: unused import: `chrono::NaiveDateTime`
  --> webservice/src/bin/../handlers.rs:40:9
   |
40 |     use chrono::NaiveDateTime;
   |         ^^^^^^^^^^^^^^^^^^^^^

warning: unused variable: `new_course`
  --> webservice/src/bin/../handlers.rs:16:5
   |
16 |     new_course: web::Json<Course>,
   |     ^^^^^^^^^^ help: if this is intentional, prefix it with an underscore: `_new_course`
   |
   = note: `#[warn(unused_variables)]` on by default

warning: unused variable: `app_state`
  --> webservice/src/bin/../handlers.rs:17:5
   |
17 |     app_state: web::Data<AppState>,
   |     ^^^^^^^^^ help: if this is intentional, prefix it with an underscore: `_app_state`

warning: unused variable: `app_state`
  --> webservice/src/bin/../handlers.rs:23:5
   |
23 |     app_state: web::Data<AppState>,
   |     ^^^^^^^^^ help: if this is intentional, prefix it with an underscore: `_app_state`

warning: unused variable: `params`
  --> webservice/src/bin/../handlers.rs:24:5
   |
24 |     params: web::Path<usize>,
   |     ^^^^^^ help: if this is intentional, prefix it with an underscore: `_params`

warning: unused variable: `app_state`
  --> webservice/src/bin/../handlers.rs:30:5
   |
30 |     app_state: web::Data<AppState>,
   |     ^^^^^^^^^ help: if this is intentional, prefix it with an underscore: `_app_state`

warning: unused variable: `params`
  --> webservice/src/bin/../handlers.rs:31:5
   |
31 |     params: web::Path<(usize, usize)>,
   |     ^^^^^^ help: if this is intentional, prefix it with an underscore: `_params`

warning: `webservice` (bin "teacher-service" test) generated 8 warnings (run `cargo fix --bin "teacher-service" --tests` to apply 8 suggestions)
    Finished test [unoptimized + debuginfo] target(s) in 0.22s
     Running unittests src/bin/teacher-service.rs (target/debug/deps/teacher_service-32d6a48d6ee3c4b4)

running 3 tests
test handlers::tests::get_all_course_success ... ok
test handlers::tests::get_one_course_success ... ok
test handlers::tests::post_course_test ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.06s


ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 
```

### 项目目录

```bash
ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ tree -a -I target     
.
├── .git
├── .gitignore
├── Cargo.lock
├── Cargo.toml
├── .env
├── src
│   └── main.rs
└── webservice
    ├── Cargo.toml
    └── src
        ├── bin
        │   ├── server1.rs
        │   └── teacher-service.rs
        ├── db_access.rs
        ├── handlers.rs
        ├── main.rs
        ├── models.rs
        ├── routers.rs
        └── state.rs

14 directories, 19 files

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
```

### webservice/src/handlers.rs

```rust
use super::db_access::*;
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

pub async fn new_course(
    new_course: web::Json<Course>,
    app_state: web::Data<AppState>,
) -> HttpResponse {
    let course = post_new_course_db(&app_state.db, new_course.into()).await;
    HttpResponse::Ok().json(course)
}

pub async fn get_courses_for_tescher(
    app_state: web::Data<AppState>,
    params: web::Path<usize>,
) -> HttpResponse {
    let teacher_id = i32::try_from(params.into_inner()).unwrap();
    let courses = get_courses_for_teacher_db(&app_state.db, teacher_id).await;
    HttpResponse::Ok().json(courses)
}

pub async fn get_courses_detail(
    app_state: web::Data<AppState>,
    params: web::Path<(usize, usize)>,
) -> HttpResponse {
    let teacher_id = i32::try_from(params.0).unwrap();
    let course_id = i32::try_from(params.1).unwrap();
    let course = get_courses_detail_db(&app_state.db, teacher_id, course_id).await;
    HttpResponse::Ok().json(course)
}

#[cfg(test)]
mod tests {
    use super::*;
    use actix_web::http::StatusCode;
    use dotenv::dotenv;
    use sqlx::postgres::PgPoolOptions;
    use std::env;
    use std::sync::Mutex;

    #[actix_rt::test] // 异步测试
    async fn post_course_test() {
        dotenv().ok();
        let db_url = env::var("DATABASE_URL").expect("DATABASE_URL is not set");
        let db_pool = PgPoolOptions::new().connect(&db_url).await.unwrap();
        let app_state: web::Data<AppState> = web::Data::new(AppState {
            health_check_response: "".to_string(),
            visit_count: Mutex::new(0),
            db: db_pool,
        });

        let course = web::Json(Course {
            teacher_id: 1,
            name: "Test course".into(),
            id: Some(3), // serial
            time: None,
        });

        let resp = new_course(course, app_state).await;
        assert_eq!(resp.status(), StatusCode::OK);
    }

    #[actix_rt::test]
    async fn get_all_course_success() {
        dotenv().ok();
        let db_url = env::var("DATABASE_URL").expect("DATABASE_URL is not set");
        let db_pool = PgPoolOptions::new().connect(&db_url).await.unwrap();
        let app_state: web::Data<AppState> = web::Data::new(AppState {
            health_check_response: "".to_string(),
            visit_count: Mutex::new(0),
            db: db_pool,
        });
        let teacher_id: web::Path<usize> = web::Path::from(1);
        let resp = get_courses_for_tescher(app_state, teacher_id).await;
        assert_eq!(resp.status(), StatusCode::OK);
    }

    #[actix_rt::test]
    async fn get_one_course_success() {
        dotenv().ok();
        let db_url = env::var("DATABASE_URL").expect("DATABASE_URL is not set");
        let db_pool = PgPoolOptions::new().connect(&db_url).await.unwrap();
        let app_state: web::Data<AppState> = web::Data::new(AppState {
            health_check_response: "".to_string(),
            visit_count: Mutex::new(0),
            db: db_pool,
        });
        let params: web::Path<(usize, usize)> = web::Path::from((1, 1));
        let resp = get_courses_detail(app_state, params).await;
        assert_eq!(resp.status(), StatusCode::OK);
    }
}

```

### webservice/src/bin/teacher-service.rs

```rust
use actix_web::{web, App, HttpServer};
use dotenv::dotenv;
use sqlx::postgres::PgPoolOptions;
use std::env;
use std::io;
use std::sync::Mutex;

#[path = "../db_access.rs"]
mod db_access;
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
    dotenv().ok();

    let database_url = env::var("DATABASE_URL").expect("DATABASE_URL is not set.");
    let db_pool = PgPoolOptions::new().connect(&database_url).await.unwrap();

    let shared_data = web::Data::new(AppState {
        health_check_response: "I'm Ok.".to_string(),
        visit_count: Mutex::new(0),
        // courses: Mutex::new(vec![]),
        db: db_pool,
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

### webservice/src/db_access.rs

```rust
use super::models::*;
use chrono::NaiveDateTime;
use sqlx::postgres::PgPool;

pub async fn get_courses_for_teacher_db(pool: &PgPool, teacher_id: i32) -> Vec<Course> {
    let rows = sqlx::query!(
        r#"SELECT id, teacher_id, name, time FROM course WHERE teacher_id = $1"#,
        teacher_id
    )
    .fetch_all(pool)
    .await
    .unwrap();

    rows.iter()
        .map(|row| Course {
            id: Some(row.id),
            teacher_id: row.teacher_id,
            name: row.name.clone(),
            time: Some(NaiveDateTime::from(row.time.unwrap())),
        })
        .collect()
}

pub async fn get_courses_detail_db(pool: &PgPool, teacher_id: i32, course_id: i32) -> Course {
    let row = sqlx::query!(
        r#"SELECT id, teacher_id, name, time FROM course WHERE teacher_id = $1 and id = $2"#,
        teacher_id,
        course_id
    )
    .fetch_one(pool)
    .await
    .unwrap();

    Course {
        id: Some(row.id),
        teacher_id: row.teacher_id,
        name: row.name.clone(),
        time: Some(NaiveDateTime::from(row.time.unwrap())),
    }
}

pub async fn post_new_course_db(pool: &PgPool, new_course: Course) -> Course {
    let row = sqlx::query!(
        r#"INSERT INTO course (id, teacher_id, name) VALUES ($1, $2, $3)
        RETURNING id, teacher_id, name, time"#,
        new_course.id,
        new_course.teacher_id,
        new_course.name
    )
    .fetch_one(pool)
    .await
    .unwrap();

    Course {
        id: Some(row.id),
        teacher_id: row.teacher_id,
        name: row.name.clone(),
        time: Some(NaiveDateTime::from(row.time.unwrap())),
    }
}

```

### webservice/src/models.rs

```rust
use actix_web::web;
use chrono::NaiveDateTime;
use serde::{Deserialize, Serialize};

#[derive(Deserialize, Serialize, Debug, Clone)]
pub struct Course {
    pub teacher_id: i32,
    pub id: Option<i32>,
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

### 测试

```bash
ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ cargo test --bin teacher-service
   Compiling webservice v0.1.0 (/Users/qiaopengjun/rust/ws/webservice)
    Finished test [unoptimized + debuginfo] target(s) in 1.61s
     Running unittests src/bin/teacher-service.rs (target/debug/deps/teacher_service-32d6a48d6ee3c4b4)

running 3 tests
test handlers::tests::get_one_course_success ... ok
test handlers::tests::get_all_course_success ... ok
test handlers::tests::post_course_test ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.01s


ws on  master [?] via 🦀 1.67.1 via 🅒 base took 2.7s 
➜ 

```

### 运行

```bash
ws on  master [?] via 🦀 1.67.1 via 🅒 base took 3.1s 
➜ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.11s
     Running `target/debug/teacher-service`

```

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202305292344384.png)

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202305292347166.png)

![](https://raw.githubusercontent.com/qiaopengjun5162/blogpicgo/master/img/202305292348576.png)

### 测试

```bash
ws on  master [?] via 🦀 1.67.1 via 🅒 base took 2.7s 
➜ curl -X POST localhost:3000/courses/ -H "Content-Type: application/json" -d '{"teacher_id":1, "id":4, "name":"Calculus"}'
{"teacher_id":1,"id":4,"name":"Calculus","time":"2023-05-29T15:51:52.245753"}%                                                    

ws on  master [?] via 🦀 1.67.1 via 🅒 base 
➜ 
```
