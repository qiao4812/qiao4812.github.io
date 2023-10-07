---
title: "Rust Web 全栈开发之连接数据库"
date: 2023-05-28T21:49:06+08:00
draft: false
tags: ["Rust"]
categories: ["Rust"]
---

# Rust Web 全栈开发之连接数据库

### 需要使用的 crate 和 数据库

- sqlx, v0.5.10
- PostgreSQL

### 创建项目

```bash
~/rust via 🅒 base
➜ cargo new db
     Created binary (application) `db` package

~/rust via 🅒 base
➜ cd db

db on  master [?] via 🦀 1.67.1 via 🅒 base
➜ c

db on  master [?] via 🦀 1.67.1 via 🅒 base
➜

```

### 项目目录

```bash
db on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ tree -a  -I target
.
├── .env
├── .git
├── .gitignore
├── Cargo.lock
├── Cargo.toml
├── db.sql
└── src
    └── main.rs

11 directories, 11 files

db on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ 
```

### Cargo.toml

```toml
[package]
name = "db"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
actix-rt = "2.8.0"
actix-web = "4.3.1"
chrono = { version = "0.4.24", features = ["serde"] }
dotenv = "0.15.0"
openssl = { version = "0.10.52", features = ["vendored"] }
serde = { version = "1.0.163", features = ["derive"] }
sqlx = { version = "0.6.3", features = [
    "postgres",
    "runtime-tokio-rustls",
    "macros",
    "chrono",
] }

```

### db.sql

```sql
DROP TABLE IF EXISTS course;

CREATE TABLE course 
( 
    ID serial PRIMARY KEY, 
    teacher_id INT NOT NULL, 
    NAME VARCHAR ( 140 ) NOT NULL, 
    TIME TIMESTAMP DEFAULT now( ) 
);

INSERT INTO course ( ID, teacher_id, NAME, TIME )
VALUES
 ( 1, 1, 'First Course', '2023-05-18 21:30:00' );

INSERT INTO course ( ID, teacher_id, NAME, TIME )
VALUES
 ( 2, 1, 'Second Course', '2023-05-28 08:45:00' );

```

### .env

```bash
DATABASE_URL=postgres://postgres:postgres@127.0.0.1:5432/postgres
#      数据库名称    用户名    密码      主机名     端口  数据库名
```

### src/main.rs

```rust
use chrono::NaiveDateTime;
use dotenv::dotenv;
use sqlx::postgres::PgPoolOptions;
use std::env;
use std::io;

#[derive(Debug)]
pub struct Course {
    pub id: i32,
    pub teacher_id: i32,
    pub name: String,
    pub time: Option<NaiveDateTime>,
}

#[actix_rt::main]
async fn main() -> io::Result<()> {
    dotenv().ok(); // 把 .env 中的环境变量读取出来

    let database_url = env::var("DATABASE_URL").expect("DATABASE_URL 没有在 .env 文件里设置");

    let db_pool = PgPoolOptions::new().connect(&database_url).await.unwrap();

    let course_rows = sqlx::query!(
        r#"select id, teacher_id, name, time from course where id = $1"#,
        1
    )
    .fetch_all(&db_pool)
    .await
    .unwrap();

    let mut courses_list = vec![];
    for row in course_rows {
        courses_list.push(Course {
            id: row.id,
            teacher_id: row.teacher_id,
            name: row.name,
            time: Some(chrono::NaiveDateTime::from(row.time.unwrap())),
        })
    }
    println!("Courses = {:?}", courses_list);
    Ok(())
}

```

### 运行

```rust
db on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ cargo run         
    Finished dev [unoptimized + debuginfo] target(s) in 0.32s
     Running `target/debug/db`
Courses = [Course { id: 1, teacher_id: 1, name: "First Course", time: Some(2023-05-18T21:30:00) }]

db on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ 

```
