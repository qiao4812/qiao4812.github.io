---
title: "Rust_queryer"
date: 2023-07-01T15:48:01+08:00
draft: false
tags: ["x", "y"]
categories: ["x", "y"]
---



```bash
/Code/rust via 🅒 base
➜ cargo new queryer --lib
     Created library `queryer` package

~/Code/rust via 🅒 base
➜ cd queryer

queryer on  master [?] via 🦀 1.70.0 via 🅒 base
➜ c

queryer on  master [?] via 🦀 1.70.0 via 🅒 base
➜


queryer on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ mkdir examples          

queryer on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ 

queryer on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ touch examples/dialect.rs                    

queryer on  master [?] vi
```

Cargo.toml

```toml
[package]
name = "queryer"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html


[[example]]
name = "dialect"

[dependencies]
anyhow = "1.0.71" # 错误处理，其实对于库我们应该用 thiserror，但这里简单起见就不节外生枝了
async-trait = "0.1.68" # 允许 trait 里有 async fn
sqlparser = "0.35.0" # SQL 解析器
polars = { version = "0.30.0", features = ["json", "lazy"] } # DataFrame 库
reqwest = { version = "0.11.18", default-features = false, features = [
    "rustls-tls",
] } # 我们的老朋友 HTTP 客户端
tokio = { version = "1.29.1", features = ["fs"] } # 我们的老朋友异步库，我们这里需要异步文件处理
tracing = "0.1.37" # 日志处理

[dev-dependencies]
tracing-subscriber = "0.3.17" # 日志处理
tokio = { version = "1.29.1", features = [
    "full",
] } # 在 example 下我们需要更多的 tokio feature

```

examples/dialect.rs

```rust
use sqlparser::{dialect::GenericDialect, parser::Parser};

fn main() {
    tracing_subscriber::fmt::init();

    let sql = "SELECT a a1, b, 123, myfunc(b), * \
    FROM data_source \
    WHERE a > b AND b < 100 AND c BETWEEN 10 AND 20 \
    ORDER BY a DESC, b \
    LIMIT 50 OFFSET 10";

    let ast = Parser::parse_sql(&GenericDialect::default(), sql);
    println!("{:#?}", ast);
}

```

```bash
queryer on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ cargo run --example dialect
   Compiling cfg-if v1.0.0
   Compiling scopeguard v1.1.0
   Compiling once_cell v1.18.0
   Compiling pin-project-lite v0.2.9
   Compiling log v0.4.19
   Compiling libc v0.2.147
   Compiling static_assertions v1.1.0
   Compiling futures-core v0.3.28
   Compiling libm v0.2.7
   Compiling memchr v2.5.0
   Compiling futures-sink v0.3.28
   Compiling serde v1.0.164
   Compiling crossbeam-utils v0.8.16
   Compiling lexical-util v0.8.5
   Compiling memoffset v0.9.0
   Compiling slab v0.4.8
   Compiling futures-channel v0.3.28
   Compiling futures-task v0.3.28
   Compiling futures-io v0.3.28
   Compiling smallvec v1.10.0
   Compiling jobserver v0.1.26
   Compiling num_cpus v1.16.0
   Compiling getrandom v0.2.10
   Compiling pin-utils v0.1.0
   Compiling crossbeam-epoch v0.9.15
   Compiling num-traits v0.2.15
   Compiling cc v1.0.79
   Compiling signal-hook-registry v1.4.1
   Compiling futures-util v0.3.28
   Compiling either v1.8.1
   Compiling lock_api v0.4.10
   Compiling crossbeam-deque v0.8.3
   Compiling ahash v0.8.3
   Compiling parking_lot_core v0.9.8
   Compiling crossbeam-channel v0.5.8
   Compiling mio v0.8.8
   Compiling core-foundation-sys v0.8.4
   Compiling lexical-write-integer v0.8.5
   Compiling lexical-parse-integer v0.8.6
   Compiling parking_lot v0.12.1
   Compiling itoa v1.0.6
   Compiling target-features v0.1.4
   Compiling lexical-parse-float v0.8.5
   Compiling lexical-write-float v0.8.5
   Compiling iana-time-zone v0.1.57
   Compiling rayon-core v1.11.0
   Compiling array-init-cursor v0.2.0
   Compiling multiversion-macros v0.7.2
   Compiling planus v0.3.1
   Compiling chrono v0.4.26
   Compiling aho-corasick v1.0.2
   Compiling zstd-sys v2.0.8+zstd.1.5.5
   Compiling lz4-sys v1.9.4
   Compiling lexical-core v0.8.5
   Compiling regex-syntax v0.7.2
   Compiling rayon v1.7.0
   Compiling simdutf8 v0.1.4
   Compiling hashbrown v0.12.3
   Compiling bytes v1.4.0
   Compiling indexmap v1.9.3
   Compiling arrow-format v0.8.1
   Compiling bytemuck v1.13.1
   Compiling futures-executor v0.3.28
   Compiling foreign_vec v0.1.0
   Compiling fallible-streaming-iterator v0.1.9
   Compiling dyn-clone v1.0.11
   Compiling multiversion v0.7.2
   Compiling futures v0.3.28
   Compiling regex-syntax v0.6.29
   Compiling strength_reduce v0.2.4
   Compiling streaming-iterator v0.1.9
   Compiling regex v1.8.4
   Compiling ethnum v1.3.2
   Compiling hash_hasher v2.0.3
   Compiling hashbrown v0.13.2
   Compiling thiserror v1.0.40
   Compiling socket2 v0.4.9
   Compiling smartstring v1.0.1
   Compiling signal-hook v0.3.15
   Compiling sysinfo v0.29.3
   Compiling tokio v1.29.1
   Compiling rand_core v0.6.4
   Compiling ryu v1.0.13
   Compiling ppv-lite86 v0.2.17
   Compiling polars-utils v0.30.0
   Compiling signal-hook-mio v0.2.3
   Compiling ring v0.16.20
   Compiling rand_chacha v0.3.1
   Compiling bitflags v1.3.2
   Compiling crossterm v0.26.1
   Compiling rand v0.8.5
   Compiling unicode-width v0.1.10
   Compiling strum v0.24.1
   Compiling halfbrown v0.2.4
   Compiling float-cmp v0.9.0
   Compiling untrusted v0.7.1
   Compiling xxhash-rust v0.8.6
   Compiling comfy-table v6.2.0
   Compiling serde_json v1.0.99
   Compiling rand_distr v0.4.3
   Compiling value-trait v0.6.1
   Compiling argminmax v0.6.1
   Compiling tracing-core v0.1.31
   Compiling now v0.1.3
   Compiling atoi v2.0.0
   Compiling lexical v6.1.1
   Compiling memmap2 v0.5.10
   Compiling fast-float v0.2.0
   Compiling fnv v1.0.7
   Compiling home v0.5.5
   Compiling tracing v0.1.37
   Compiling simd-json v0.10.3
   Compiling http v0.2.9
   Compiling tinyvec_macros v0.1.1
   Compiling try-lock v0.2.4
   Compiling percent-encoding v2.3.0
   Compiling httparse v1.8.0
   Compiling glob v0.3.1
   Compiling tinyvec v1.6.0
   Compiling want v0.3.1
   Compiling form_urlencoded v1.2.0
   Compiling tower-service v0.3.2
   Compiling unicode-bidi v0.3.13
   Compiling httpdate v1.0.2
   Compiling sqlparser v0.34.0
   Compiling base64 v0.21.2
   Compiling lazy_static v1.4.0
   Compiling serde_urlencoded v0.7.1
   Compiling encoding_rs v0.8.32
   Compiling http-body v0.4.5
   Compiling rustls-pemfile v1.0.3
   Compiling overload v0.1.1
   Compiling mime v0.3.17
   Compiling ipnet v2.8.0
   Compiling nu-ansi-term v0.46.0
   Compiling unicode-normalization v0.1.22
   Compiling anyhow v1.0.71
   Compiling tracing-log v0.1.3
   Compiling sharded-slab v0.1.4
   Compiling tokio-util v0.7.8
   Compiling rustls v0.21.2
   Compiling idna v0.4.0
   Compiling sqlparser v0.35.0
   Compiling thread_local v1.1.7
   Compiling lz4 v1.24.0
   Compiling h2 v0.3.20
   Compiling tracing-subscriber v0.3.17
   Compiling url v2.4.0
   Compiling sct v0.7.0
   Compiling rustls-webpki v0.100.1
   Compiling webpki v0.22.0
   Compiling webpki-roots v0.22.6
   Compiling hyper v0.14.27
   Compiling tokio-rustls v0.24.1
   Compiling hyper-rustls v0.24.0
   Compiling reqwest v0.11.18
   Compiling zstd-safe v6.0.5+zstd.1.5.4
   Compiling zstd v0.12.3+zstd.1.5.2
   Compiling arrow2 v0.17.2
   Compiling polars-error v0.30.0
   Compiling polars-arrow v0.30.0
   Compiling polars-row v0.30.0
   Compiling polars-core v0.30.0
   Compiling polars-json v0.30.0
   Compiling polars-ops v0.30.0
   Compiling polars-time v0.30.0
   Compiling polars-io v0.30.0
   Compiling polars-plan v0.30.0
   Compiling polars-pipe v0.30.0
   Compiling polars-lazy v0.30.0
   Compiling polars-sql v0.30.0
   Compiling polars v0.30.0
   Compiling queryer v0.1.0 (/Users/qiaopengjun/Code/rust/queryer)
    Finished dev [unoptimized + debuginfo] target(s) in 33.34s
     Running `target/debug/examples/dialect`
Ok(
    [
        Query(
            Query {
                with: None,
                body: Select(
                    Select {
                        distinct: None,
                        top: None,
                        projection: [
                            ExprWithAlias {
                                expr: Identifier(
                                    Ident {
                                        value: "a",
                                        quote_style: None,
                                    },
                                ),
                                alias: Ident {
                                    value: "a1",
                                    quote_style: None,
                                },
                            },
                            UnnamedExpr(
                                Identifier(
                                    Ident {
                                        value: "b",
                                        quote_style: None,
                                    },
                                ),
                            ),
                            UnnamedExpr(
                                Value(
                                    Number(
                                        "123",
                                        false,
                                    ),
                                ),
                            ),
                            UnnamedExpr(
                                Function(
                                    Function {
                                        name: ObjectName(
                                            [
                                                Ident {
                                                    value: "myfunc",
                                                    quote_style: None,
                                                },
                                            ],
                                        ),
                                        args: [
                                            Unnamed(
                                                Expr(
                                                    Identifier(
                                                        Ident {
                                                            value: "b",
                                                            quote_style: None,
                                                        },
                                                    ),
                                                ),
                                            ),
                                        ],
                                        over: None,
                                        distinct: false,
                                        special: false,
                                        order_by: [],
                                    },
                                ),
                            ),
                            Wildcard(
                                WildcardAdditionalOptions {
                                    opt_exclude: None,
                                    opt_except: None,
                                    opt_rename: None,
                                    opt_replace: None,
                                },
                            ),
                        ],
                        into: None,
                        from: [
                            TableWithJoins {
                                relation: Table {
                                    name: ObjectName(
                                        [
                                            Ident {
                                                value: "data_source",
                                                quote_style: None,
                                            },
                                        ],
                                    ),
                                    alias: None,
                                    args: None,
                                    with_hints: [],
                                },
                                joins: [],
                            },
                        ],
                        lateral_views: [],
                        selection: Some(
                            BinaryOp {
                                left: BinaryOp {
                                    left: BinaryOp {
                                        left: Identifier(
                                            Ident {
                                                value: "a",
                                                quote_style: None,
                                            },
                                        ),
                                        op: Gt,
                                        right: Identifier(
                                            Ident {
                                                value: "b",
                                                quote_style: None,
                                            },
                                        ),
                                    },
                                    op: And,
                                    right: BinaryOp {
                                        left: Identifier(
                                            Ident {
                                                value: "b",
                                                quote_style: None,
                                            },
                                        ),
                                        op: Lt,
                                        right: Value(
                                            Number(
                                                "100",
                                                false,
                                            ),
                                        ),
                                    },
                                },
                                op: And,
                                right: Between {
                                    expr: Identifier(
                                        Ident {
                                            value: "c",
                                            quote_style: None,
                                        },
                                    ),
                                    negated: false,
                                    low: Value(
                                        Number(
                                            "10",
                                            false,
                                        ),
                                    ),
                                    high: Value(
                                        Number(
                                            "20",
                                            false,
                                        ),
                                    ),
                                },
                            },
                        ),
                        group_by: [],
                        cluster_by: [],
                        distribute_by: [],
                        sort_by: [],
                        having: None,
                        named_window: [],
                        qualify: None,
                    },
                ),
                order_by: [
                    OrderByExpr {
                        expr: Identifier(
                            Ident {
                                value: "a",
                                quote_style: None,
                            },
                        ),
                        asc: Some(
                            false,
                        ),
                        nulls_first: None,
                    },
                    OrderByExpr {
                        expr: Identifier(
                            Ident {
                                value: "b",
                                quote_style: None,
                            },
                        ),
                        asc: None,
                        nulls_first: None,
                    },
                ],
                limit: Some(
                    Value(
                        Number(
                            "50",
                            false,
                        ),
                    ),
                ),
                offset: Some(
                    Offset {
                        value: Value(
                            Number(
                                "10",
                                false,
                            ),
                        ),
                        rows: None,
                    },
                ),
                fetch: None,
                locks: [],
            },
        ),
    ],
)

queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 33.8s 
➜ 
```

```bash
queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 33.8s 
➜ touch src/dialect.rs                   

```

src/dialect.rs

```rust
use sqlparser::dialect::Dialect;

#[derive(Debug, Default)]
pub struct TyrDialect;

// 创建自己的 sql 方言。TyrDialect 支持 identifier 可以是简单的 url
impl Dialect for TyrDialect {
    fn is_identifier_start(&self, ch: char) -> bool {
        ('a'..='z').contains(&ch) || ('A'..='Z').contains(&ch) || ch == '_'
    }

    // identifier 可以有 ':', '/', '?', '&', '='
    fn is_identifier_part(&self, ch: char) -> bool {
        ('a'..='z').contains(&ch)
            || ('A'..='Z').contains(&ch)
            || ('0'..='9').contains(&ch)
            || [':', '/', '?', '&', '=', '-', '_', '.'].contains(&ch)
    }
}

/// 测试辅助函数
#[allow(dead_code)]
pub fn example_sql() -> String {
    let url = "https://raw.githubusercontent.com/owid/covid-19-data/master/public/data/latest/owid-covid-latest.csv";

    let sql = format!(
        "SELECT location name, total_cases, new_cases, total_deaths, new_deaths \
        FROM {} where new_deaths >= 500 ORDER BY new_cases DESC LIMIT 6 OFFSET 5",
        url
    );

    sql
}

#[cfg(test)]
mod tests {
    use super::*;
    use sqlparser::parser::Parser;

    #[test]
    fn it_works() {
        assert!(Parser::parse_sql(&TyrDialect::default(), &example_sql()).is_ok());
    }
}

```

```bash
queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base 
➜ cargo test                 
   Compiling queryer v0.1.0 (/Users/qiaopengjun/Code/rust/queryer)
warning: function `example_sql` is never used
  --> src/dialect.rs:22:8
   |
22 | pub fn example_sql() -> String {
   |        ^^^^^^^^^^^
   |
   = note: `#[warn(dead_code)]` on by default

warning: `queryer` (lib) generated 1 warning
    Finished test [unoptimized + debuginfo] target(s) in 0.94s
     Running unittests src/lib.rs (target/debug/deps/queryer-d70390a7b5839d13)

running 2 tests
test tests::it_works ... ok
test dialect::tests::it_works ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests queryer

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 2.2s 
➜ 
```

实现 AST 的转换

创建 examples/covid.rs（记得在 Cargo.toml 中添加它哦），手工实现一个 DataFrame 的加载和查询：

```bash
queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 2.2s 
➜ touch examples/covid.rs              

queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base 
➜ 
```

 examples/covid.rs

```rust
use anyhow::Result;
use polars::prelude::*;
use std::io::Cursor;

#[tokio::main]
async fn main() -> Result<()> {
    tracing_subscriber::fmt::init();

    // let url = "https://raw.githubusercontent.com/owid/covid-19-data/master/public/data/latest/owid-covid-latest.csv";
    let url = "https://gitee.com/ekoclike/get/raw/main/qwe.csv";
    let data = reqwest::get(url).await?.text().await?;

    // 使用 polars 直接请求
    let df = CsvReader::new(Cursor::new(data))
        .infer_schema(Some(16))
        .finish()?;

    let filtered = df.filter(&df.column("new_deaths")?.gt(500)?)?;
    println!(
        "{:?}",
        filtered.select([
            "location",
            "total_cases",
            "new_cases",
            "total_deaths",
            "new_deaths"
        ])
    );

    Ok(())
}

```

如果我们运行这个 example，可以得到一个打印得非常漂亮的表格，它从 GitHub 上的 owid-covid-latest.csv 文件中，读取并查询 new_deaths 大于 500 的国家和区域：

```bash

queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 5.0s 
➜ cargo run --example covid --quiet
Ok(shape: (0, 5)
┌──────────┬─────────────┬───────────┬──────────────┬────────────┐
│ location ┆ total_cases ┆ new_cases ┆ total_deaths ┆ new_deaths │
│ ---      ┆ ---         ┆ ---       ┆ ---          ┆ ---        │
│ str      ┆ f64         ┆ f64       ┆ f64          ┆ f64        │
╞══════════╪═════════════╪═══════════╪══════════════╪════════════╡
└──────────┴─────────────┴───────────┴──────────────┴────────────┘)

queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 40.3s 
➜ 

queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 1m 16.6s 
➜ cargo run --example covid --quiet
Ok(shape: (6, 5)
┌───────────────┬──────────────┬───────────┬──────────────┬────────────┐
│ location      ┆ total_cases  ┆ new_cases ┆ total_deaths ┆ new_deaths │
│ ---           ┆ ---          ┆ ---       ┆ ---          ┆ ---        │
│ str           ┆ f64          ┆ f64       ┆ f64          ┆ f64        │
╞═══════════════╪══════════════╪═══════════╪══════════════╪════════════╡
│ Asia          ┆ 1.81236598e8 ┆ 286054.0  ┆ 1.476212e6   ┆ 550.0      │
│ Europe        ┆ 2.24635518e8 ┆ 175990.0  ┆ 1.929041e6   ┆ 588.0      │
│ High income   ┆ 3.68550824e8 ┆ 515577.0  ┆ 2.568947e6   ┆ 1695.0     │
│ North America ┆ 1.1273229e8  ┆ 126857.0  ┆ 1.500623e6   ┆ 876.0      │
│ United States ┆ 9.5020834e7  ┆ 122716.0  ┆ 1.048989e6   ┆ 788.0      │
│ World         ┆ 6.06795536e8 ┆ 606534.0  ┆ 6.507436e6   ┆ 2109.0     │
└───────────────┴──────────────┴───────────┴──────────────┴────────────┘)

queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 5.2s 
➜ 
```

创建一个文件 src/convert.rs，先定义一个数据结构 Sql 来描述两者的对应关系，然后再实现 Sql 的 TryFromtrait：

```bash
queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 5.2s 
➜ touch src/convert.rs                 

queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base 
➜ 

```

src/convert.rs

```rust
use anyhow::{anyhow, Result};
use polars::prelude::*;
use sqlparser::ast::{
    BinaryOperator as SqlBinaryOperator, Expr as SqlExpr, Offset as SqlOffset, OrderByExpr, Select,
    SelectItem, SetExpr, Statement, TableFactor, TableWithJoins, Value as SqlValue,
};

/// 解析出来的 SQL
pub struct Sql<'a> {
    pub(crate) selection: Vec<Expr>,
    pub(crate) condition: Option<Expr>,
    pub(crate) source: &'a str,
    pub(crate) order_by: Vec<(String, bool)>,
    pub(crate) offset: Option<i64>,
    pub(crate) limit: Option<usize>,
}

// 因为 Rust trait 的孤儿规则，我们如果要想对已有的类型实现已有的 trait，
// 需要简单包装一下

pub struct Expression(pub(crate) Box<SqlExpr>);
pub struct Operation(pub(crate) SqlBinaryOperator);
pub struct Projection<'a>(pub(crate) &'a SelectItem);
pub struct Source<'a>(pub(crate) &'a [TableWithJoins]);
pub struct Order<'a>(pub(crate) &'a OrderByExpr);
pub struct Offset<'a>(pub(crate) &'a SqlOffset);
pub struct Limit<'a>(pub(crate) &'a SqlExpr);
pub struct Value(pub(crate) SqlValue);

/// 把 SqlParser 解析出来的 Statement 转换成我们需要的结构
impl<'a> TryFrom<&'a Statement> for Sql<'a> {
    type Error = anyhow::Error;

    fn try_from(sql: &'a Statement) -> Result<Self, Self::Error> {
        match sql {
            // 目前我们只关心 query (select ... from ... where ...)
            Statement::Query(q) => {
                let offset = q.offset.as_ref();
                let limit = q.limit.as_ref();
                let orders = &q.order_by;
                let Select {
                    from: table_with_joins,
                    selection: where_clause,
                    projection,

                    group_by: _,
                    ..
                } = match &q.body {
                    SetExpr::Select(statement) => statement.as_ref(),
                    _ => return Err(anyhow!("We only support Select Query at the moment")),
                };

                let source = Source(table_with_joins).try_into()?;

                let condition = match where_clause {
                    Some(expr) => Some(Expression(Box::new(expr.to_owned())).try_into()?),
                    None => None,
                };

                let mut selection = Vec::with_capacity(8);
                for p in projection {
                    let expr = Projection(p).try_into()?;
                    selection.push(expr);
                }

                let mut order_by = Vec::new();
                for expr in orders {
                    order_by.push(Order(expr).try_into()?);
                }

                let offset = offset.map(|v| Offset(v).into());
                let limit = limit.map(|v| Limit(v).into());

                Ok(Sql {
                    selection,
                    condition,
                    source,
                    order_by,
                    offset,
                    limit,
                })
            }
            _ => Err(anyhow!("We only support Query at the moment")),
        }
    }
}

/// 把 SqlParser 的 Expr 转换成 DataFrame 的 Expr
impl TryFrom<Expression> for Expr {
    type Error = anyhow::Error;

    fn try_from(expr: Expression) -> Result<Self, Self::Error> {
        match *expr.0 {
            SqlExpr::BinaryOp { left, op, right } => Ok(Expr::BinaryExpr {
                left: Box::new(Expression(left).try_into()?),
                op: Operation(op).try_into()?,
                right: Box::new(Expression(right).try_into()?),
            }),
            SqlExpr::Wildcard => Ok(Self::Wildcard),
            SqlExpr::IsNull(expr) => Ok(Self::IsNull(Box::new(Expression(expr).try_into()?))),
            SqlExpr::IsNotNull(expr) => Ok(Self::IsNotNull(Box::new(Expression(expr).try_into()?))),
            SqlExpr::Identifier(id) => Ok(Self::Column(Arc::new(id.value))),
            SqlExpr::Value(v) => Ok(Self::Literal(Value(v).try_into()?)),
            v => Err(anyhow!("expr {:#?} is not supported", v)),
        }
    }
}

/// 把 SqlParser 的 BinaryOperator 转换成 DataFrame 的 Operator
impl TryFrom<Operation> for Operator {
    type Error = anyhow::Error;

    fn try_from(op: Operation) -> Result<Self, Self::Error> {
        match op.0 {
            SqlBinaryOperator::Plus => Ok(Self::Plus),
            SqlBinaryOperator::Minus => Ok(Self::Minus),
            SqlBinaryOperator::Multiply => Ok(Self::Multiply),
            SqlBinaryOperator::Divide => Ok(Self::Divide),
            SqlBinaryOperator::Modulo => Ok(Self::Modulus),
            SqlBinaryOperator::Gt => Ok(Self::Gt),
            SqlBinaryOperator::Lt => Ok(Self::Lt),
            SqlBinaryOperator::GtEq => Ok(Self::GtEq),
            SqlBinaryOperator::LtEq => Ok(Self::LtEq),
            SqlBinaryOperator::Eq => Ok(Self::Eq),
            SqlBinaryOperator::NotEq => Ok(Self::NotEq),
            SqlBinaryOperator::And => Ok(Self::And),
            SqlBinaryOperator::Or => Ok(Self::Or),
            v => Err(anyhow!("Operator {} is not supported", v)),
        }
    }
}

/// 把 SqlParser 的 SelectItem 转换成 DataFrame 的 Expr
impl<'a> TryFrom<Projection<'a>> for Expr {
    type Error = anyhow::Error;

    fn try_from(p: Projection<'a>) -> Result<Self, Self::Error> {
        match p.0 {
            SelectItem::UnnamedExpr(SqlExpr::Identifier(id)) => Ok(col(&id.to_string())),
            SelectItem::ExprWithAlias {
                expr: SqlExpr::Identifier(id),
                alias,
            } => Ok(Expr::Alias(
                Box::new(Expr::Column(Arc::new(id.to_string()))),
                Arc::new(alias.to_string()),
            )),
            SelectItem::QualifiedWildcard(v) => Ok(col(&v.to_string())),
            SelectItem::Wildcard => Ok(col("*")),
            item => Err(anyhow!("projection {} not supported", item)),
        }
    }
}

impl<'a> TryFrom<Source<'a>> for &'a str {
    type Error = anyhow::Error;

    fn try_from(source: Source<'a>) -> Result<Self, Self::Error> {
        if source.0.len() != 1 {
            return Err(anyhow!("We only support single data source at the moment"));
        }

        let table = &source.0[0];
        if !table.joins.is_empty() {
            return Err(anyhow!("We do not support joint data source at the moment"));
        }

        match &table.relation {
            TableFactor::Table { name, .. } => Ok(&name.0.first().unwrap().value),
            _ => Err(anyhow!("We only support table")),
        }
    }
}

/// 把 SqlParser 的 order by expr 转换成 (列名, 排序方法)
impl<'a> TryFrom<Order<'a>> for (String, bool) {
    type Error = anyhow::Error;

    fn try_from(o: Order) -> Result<Self, Self::Error> {
        let name = match &o.0.expr {
            SqlExpr::Identifier(id) => id.to_string(),
            expr => {
                return Err(anyhow!(
                    "We only support identifier for order by, got {}",
                    expr
                ))
            }
        };

        Ok((name, !o.0.asc.unwrap_or(true)))
    }
}

/// 把 SqlParser 的 offset expr 转换成 i64
impl<'a> From<Offset<'a>> for i64 {
    fn from(offset: Offset) -> Self {
        match offset.0 {
            SqlOffset {
                value: SqlExpr::Value(SqlValue::Number(v, _b)),
                ..
            } => v.parse().unwrap_or(0),
            _ => 0,
        }
    }
}

/// 把 SqlParser 的 Limit expr 转换成 usize
impl<'a> From<Limit<'a>> for usize {
    fn from(l: Limit<'a>) -> Self {
        match l.0 {
            SqlExpr::Value(SqlValue::Number(v, _b)) => v.parse().unwrap_or(usize::MAX),
            _ => usize::MAX,
        }
    }
}

/// 把 SqlParser 的 value 转换成 DataFrame 支持的 LiteralValue
impl TryFrom<Value> for LiteralValue {
    type Error = anyhow::Error;
    fn try_from(v: Value) -> Result<Self, Self::Error> {
        match v.0 {
            SqlValue::Number(v, _) => Ok(LiteralValue::Float64(v.parse().unwrap())),
            SqlValue::Boolean(v) => Ok(LiteralValue::Boolean(v)),
            SqlValue::Null => Ok(LiteralValue::Null),
            v => Err(anyhow!("Value {} is not supported", v)),
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    use crate::TyrDialect;
    use sqlparser::parser::Parser;

    #[test]
    fn parse_sql_works() {
        let url = "http://abc.xyz/abc?a=1&b=2";
        let sql = format!(
            "select a, b, c from {} where a=1 order by c desc limit 5 offset 10",
            url
        );
        let statement = &Parser::parse_sql(&TyrDialect::default(), sql.as_ref()).unwrap()[0];
        let sql: Sql = statement.try_into().unwrap();
        assert_eq!(sql.source, url);
        assert_eq!(sql.limit, Some(5));
        assert_eq!(sql.offset, Some(10));
        assert_eq!(sql.order_by, vec![("c".into(), true)]);
        assert_eq!(sql.selection, vec![col("a"), col("b"), col("c")]);
    }
}
```

我们写代码，主要都在处理什么？绝大多数处理逻辑都是把数据从一个接口转换成另一个接口。

好的代码，应该是每个主流程都清晰简约，代码恰到好处地出现在那里，让人不需要注释也能明白作者在写什么。

### 从源中取数据

这个源，我们规定它必须是以 http(s):// 或者 file:// 开头的字符串。因为，以 http 开头我们可以通过 URL 获取内容，file 开头我们可以通过文件名，打开本地文件获取内容。

src/fetcher.rs

```rust

use anyhow::{anyhow, Result};
use async_trait::async_trait;
use tokio::fs;

// Rust 的 async trait 还没有稳定，可以用 async_trait 宏
#[async_trait]
pub trait Fetch {
    type Error;
    async fn fetch(&self) -> Result<String, Self::Error>;
}

/// 从文件源或者 http 源中获取数据，组成 data frame
pub async fn retrieve_data(source: impl AsRef<str>) -> Result<String> {
    let name = source.as_ref();
    match &name[..4] {
        // 包括 http / https
        "http" => UrlFetcher(name).fetch().await,
        // 处理 file://<filename>
        "file" => FileFetcher(name).fetch().await,
        _ => return Err(anyhow!("We only support http/https/file at the moment")),
    }
}

struct UrlFetcher<'a>(pub(crate) &'a str);
struct FileFetcher<'a>(pub(crate) &'a str);

#[async_trait]
impl<'a> Fetch for UrlFetcher<'a> {
    type Error = anyhow::Error;

    async fn fetch(&self) -> Result<String, Self::Error> {
        Ok(reqwest::get(self.0).await?.text().await?)
    }
}

#[async_trait]
impl<'a> Fetch for FileFetcher<'a> {
    type Error = anyhow::Error;

    async fn fetch(&self) -> Result<String, Self::Error> {
        Ok(fs::read_to_string(&self.0[7..]).await?)
    }
}
```

主流程

```bash
queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base 
➜ touch src/fetcher.rs              

queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base 
➜ touch src/loader.rs              

queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base 
➜ 

```

尝试编译一下

```bash
queryer on  master [?] is 📦 0.1.0 via 🦀 1.70.0 via 🅒 base took 21.5s 
➜ cargo build
```
