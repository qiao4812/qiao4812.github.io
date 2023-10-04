---
title: "reservation 项目"
date: 2023-07-07T12:49:04+08:00
draft: true
tags: ["Rust", "项目"]
categories: ["Rust", "项目"]
---

# reservation 项目

预定系统是很多业务逻辑中都可能涉及到的内容。它可以处理诸如某个资源在某个时间段独占的一系列问题。比如说日程表安排，会议室预定，测试设备的预定，酒店房间预定等等。构建一个预定系统不算太过复杂，但也绝不容易。这个系列我希望更进一步，不光介绍如何撰写 Rust 代码，还深入探讨软件设计中的各种选择。如果你跟着这个系列一路走下去，你会领略到如何通过撰写 RFC 来完成初始设计，如何用 tonic 做 gRPC，如何用 sqlx 访问 postgres 数据库，以及各种各样眼花缭乱的数据转换（比如 protobuf 中定义的数据如何和 postgres row 关联起来）。同时，我还会介绍整个过程中所涉及到的有关 HTTP/2 的知识，Postgres 的各种技巧（schema 的使用，EXCLUDE constraint 的使用，权限管理等）。

如何用 Rust 处理资源预定类的问题？

## Rust 项目实操 - 从零开始构建预定系统（1）：思考需求，构建 RFC

### 搭建项目

```bash
~/Code/rust via 🅒 base
➜ mcd reservation

Code/rust/reservation via 🅒 base
➜ c

Code/rust/reservation via 🅒 base took 2.9s
➜

Code/rust/reservation via 🅒 base took 2.9s
➜ cd ..

~/Code/rust via 🅒 base
➜ git clone git@github.com:tyrchen/simple-dns.git
正克隆到 'simple-dns'...
remote: Enumerating objects: 42, done.
remote: Counting objects: 100% (42/42), done.
remote: Compressing objects: 100% (24/24), done.
remote: Total 42 (delta 10), reused 39 (delta 7), pack-reused 0
接收对象中: 100% (42/42), 11.90 KiB | 2.98 MiB/s, 完成.
处理 delta 中: 100% (10/10), 完成.

~/Code/rust via 🅒 base took 4.6s
➜


Code/rust/reservation via 🅒 base 
➜ touch Cargo.toml

Code/rust/reservation via 🦀 1.70.0 via 🅒 base 
➜ touch README.md 

Code/rust/reservation via 🦀 1.70.0 via 🅒 base 
➜ 

Code/rust/reservation via 🦀 1.70.0 via 🅒 base 
➜ cp -r ../simple-dns/deny.toml .

Code/rust/reservation via 🦀 1.70.0 via 🅒 base 
➜ cp -r ../simple-dns/.github .  

Code/rust/reservation via 🦀 1.70.0 via 🅒 base 
➜ cp -r ../simple-dns/.pre-commit-config.yaml .

Code/rust/reservation via 🦀 1.70.0 via 🅒 base 
➜ cp -r ../simple-dns/_typos.toml .            

Code/rust/reservation via 🦀 1.70.0 via 🅒 base 
➜ 

Code/rust/reservation via 🦀 1.70.0 via 🅒 base 
➜ git init                                       
提示：使用 'master' 作为初始分支的名称。这个默认分支名称可能会更改。要在新仓库中
提示：配置使用初始分支名，并消除这条警告，请执行：
提示：
提示：  git config --global init.defaultBranch <名称>
提示：
提示：除了 'master' 之外，通常选定的名字有 'main'、'trunk' 和 'development'。
提示：可以通过以下命令重命名刚创建的分支：
提示：
提示：  git branch -m <name>
已初始化空的 Git 仓库于 /Users/qiaopengjun/Code/rust/reservation/.git/

reservation on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ pip install pre-commit
Collecting pre-commit
  Downloading pre_commit-3.3.3-py2.py3-none-any.whl (202 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 202.8/202.8 kB 1.1 MB/s eta 0:00:00
Requirement already satisfied: pyyaml>=5.1 in /Users/qiaopengjun/miniconda3/lib/python3.10/site-packages (from pre-commit) (6.0)
Collecting virtualenv>=20.10.0
  Downloading virtualenv-20.23.1-py3-none-any.whl (3.3 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 3.3/3.3 MB 12.7 MB/s eta 0:00:00
Collecting identify>=1.0.0
  Downloading identify-2.5.24-py2.py3-none-any.whl (98 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 98.8/98.8 kB 11.7 MB/s eta 0:00:00
Collecting cfgv>=2.0.0
  Downloading cfgv-3.3.1-py2.py3-none-any.whl (7.3 kB)
Collecting nodeenv>=0.11.1
  Downloading nodeenv-1.8.0-py2.py3-none-any.whl (22 kB)
Requirement already satisfied: setuptools in /Users/qiaopengjun/miniconda3/lib/python3.10/site-packages (from nodeenv>=0.11.1->pre-commit) (65.5.0)
Collecting distlib<1,>=0.3.6
  Downloading distlib-0.3.6-py2.py3-none-any.whl (468 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 468.5/468.5 kB 39.1 MB/s eta 0:00:00
Collecting platformdirs<4,>=3.5.1
  Downloading platformdirs-3.8.1-py3-none-any.whl (16 kB)
Collecting filelock<4,>=3.12
  Downloading filelock-3.12.2-py3-none-any.whl (10 kB)
Installing collected packages: distlib, platformdirs, nodeenv, identify, filelock, cfgv, virtualenv, pre-commit
  Attempting uninstall: platformdirs
    Found existing installation: platformdirs 2.5.2
    Uninstalling platformdirs-2.5.2:
      Successfully uninstalled platformdirs-2.5.2
  Attempting uninstall: filelock
    Found existing installation: filelock 3.9.0
    Uninstalling filelock-3.9.0:
      Successfully uninstalled filelock-3.9.0
Successfully installed cfgv-3.3.1 distlib-0.3.6 filelock-3.12.2 identify-2.5.24 nodeenv-1.8.0 platformdirs-3.8.1 pre-commit-3.3.3 virtualenv-20.23.1

[notice] A new release of pip is available: 23.0.1 -> 23.1.2
[notice] To update, run: pip install --upgrade pip

reservation on  master [?] via 🦀 1.70.0 via 🅒 base took 5.0s 
➜ pip install --upgrade pip
Requirement already satisfied: pip in /Users/qiaopengjun/miniconda3/lib/python3.10/site-packages (23.0.1)
Collecting pip
  Downloading pip-23.1.2-py3-none-any.whl (2.1 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 2.1/2.1 MB 5.8 MB/s eta 0:00:00
Installing collected packages: pip
  Attempting uninstall: pip
    Found existing installation: pip 23.0.1
    Uninstalling pip-23.0.1:
      Successfully uninstalled pip-23.0.1
Successfully installed pip-23.1.2

reservation on  master [?] via 🦀 1.70.0 via 🅒 base took 3.4s 
➜ 

reservation on  master [?] via 🦀 1.70.0 via 🅒 base took 3.4s 
➜ pre-commit --version
pre-commit 3.3.3

reservation on  master [?] via 🦀 1.70.0 via 🅒 base 
```



```bash

reservation on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ pre-commit install  
pre-commit installed at .git/hooks/pre-commit

reservation on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ git status
位于分支 master

尚无提交

未跟踪的文件:
  （使用 "git add <文件>..." 以包含要提交的内容）
        .github/
        .pre-commit-config.yaml
        Cargo.toml
        README.md
        _typos.toml
        deny.toml

提交为空，但是存在尚未跟踪的文件（使用 "git add" 建立跟踪）

reservation on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ cargo new abi --lib              
warning: compiling this new package may not work due to invalid workspace configuration

failed to parse manifest at `/Users/qiaopengjun/Code/rust/reservation/Cargo.toml`

Caused by:
  virtual manifests must be configured with [workspace]
     Created library `abi` package

reservation on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ cargo new reservation --lib            
warning: compiling this new package may not work due to invalid workspace configuration

failed to parse manifest at `/Users/qiaopengjun/Code/rust/reservation/Cargo.toml`

Caused by:
  virtual manifests must be configured with [workspace]
     Created library `reservation` package

reservation on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ cargo new service          
warning: compiling this new package may not work due to invalid workspace configuration

failed to parse manifest at `/Users/qiaopengjun/Code/rust/reservation/Cargo.toml`

Caused by:
  virtual manifests must be configured with [workspace]
     Created binary (application) `service` package

reservation on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ 
```



```bash
reservation on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ cargo build      
   Compiling abi v0.1.0 (/Users/qiaopengjun/Code/rust/reservation/abi)
   Compiling reservation v0.1.0 (/Users/qiaopengjun/Code/rust/reservation/reservation)
   Compiling reservation-service v0.1.0 (/Users/qiaopengjun/Code/rust/reservation/service)
    Finished dev [unoptimized + debuginfo] target(s) in 1.00s

reservation on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ 
```



```bash

reservation on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ git status
位于分支 master

尚无提交

未跟踪的文件:
  （使用 "git add <文件>..." 以包含要提交的内容）
        .github/
        .pre-commit-config.yaml
        Cargo.lock
        Cargo.toml
        README.md
        _typos.toml
        abi/
        deny.toml
        reservation/
        service/
        target/

提交为空，但是存在尚未跟踪的文件（使用 "git add" 建立跟踪）

reservation on  master [?] via 🦀 1.70.0 via 🅒 base 
➜ git add . 

reservation on  master [+] via 🦀 1.70.0 via 🅒 base 
➜ git status

reservation on  master [+] via 🦀 1.70.0 via 🅒 base 
➜ git rm -r --cached target/


reservation on  master [+?] via 🦀 1.70.0 via 🅒 base 
➜ touch .gitignore 

reservation on  master [+?] via 🦀 1.70.0 via 🅒 base 
➜ git add .                 

reservation on  master [+] via 🦀 1.70.0 via 🅒 base 
➜ git status
位于分支 master

尚无提交

要提交的变更：
  （使用 "git rm --cached <文件>..." 以取消暂存）
        新文件：   .github/workflows/build.yml
        新文件：   .gitignore
        新文件：   .pre-commit-config.yaml
        新文件：   Cargo.lock
        新文件：   Cargo.toml
        新文件：   README.md
        新文件：   _typos.toml
        新文件：   abi/Cargo.toml
        新文件：   abi/src/lib.rs
        新文件：   deny.toml
        新文件：   reservation/Cargo.toml
        新文件：   reservation/src/lib.rs
        新文件：   service/Cargo.toml
        新文件：   service/src/main.rs


# 报错 error: no such command: `deny` error: no such command: `nextest`                             

# 解决
reservation on  master [+] via 🦀 1.70.0 via 🅒 base
➜ curl -LsSf https://get.nexte.st/latest/mac | tar zxf - -C ${CARGO_HOME:-~/.cargo}/bin


reservation on  master [+] via 🦀 1.70.0 via 🅒 base took 2.6s
➜ cargo install --locked cargo-deny
    Updating `mirror` index




reservation on  master [+] via 🦀 1.70.0 via 🅒 base 
➜ git commit -a  
Check for byte-order marker..............................................Passed
Check for case conflicts.................................................Passed
Check for merge conflicts................................................Passed
Check for broken symlinks............................(no files to check)Skipped
Check Yaml...............................................................Passed
Fix End of Files.........................................................Passed
Mixed line ending........................................................Passed
Trim Trailing Whitespace.................................................Passed
black................................................(no files to check)Skipped
typos....................................................................Passed
cargo fmt................................................................Passed
cargo deny check.........................................................Passed
cargo check..............................................................Passed
cargo clippy.............................................................Passed
cargo test...............................................................Passed
[master（根提交） fe5e666] init commit
 14 files changed, 377 insertions(+)
 create mode 100644 .github/workflows/build.yml
 create mode 100644 .gitignore
 create mode 100644 .pre-commit-config.yaml
 create mode 100644 Cargo.lock
 create mode 100644 Cargo.toml
 create mode 100644 README.md
 create mode 100644 _typos.toml
 create mode 100644 abi/Cargo.toml
 create mode 100644 abi/src/lib.rs
 create mode 100644 deny.toml
 create mode 100644 reservation/Cargo.toml
 create mode 100644 reservation/src/lib.rs
 create mode 100644 service/Cargo.toml
 create mode 100644 service/src/main.rs

reservation on  master via 🦀 1.70.0 via 🅒 base took 1m 27.0s 
```



```bash
reservation on  master via 🦀 1.70.0 via 🅒 base took 1m 27.0s 
➜ git remote add origin git@github.com:qiaopengjun5162/reservation.git
git branch -M main
git push -u origin main
枚举对象中: 22, 完成.
对象计数中: 100% (22/22), 完成.
使用 12 个线程进行压缩
压缩对象中: 100% (15/15), 完成.
写入对象中: 100% (22/22), 5.70 KiB | 5.70 MiB/s, 完成.
总共 22（差异 2），复用 0（差异 0），包复用 0
remote: Resolving deltas: 100% (2/2), done.
To github.com:qiaopengjun5162/reservation.git
 * [new branch]      main -> main
分支 'main' 设置为跟踪 'origin/main'。

reservation on  main via 🦀 1.70.0 via 🅒 base took 4.0s 
➜ 
```

## 二

思考：首先不考虑内部如何实现，考虑对外接口应该是什么样子，核心数据结构是什么样子？

开始写代码之前，多想一想你要解决的问题，从各个角度去思考它的利弊，对你写代码会有帮助的。

<https://raw.githubusercontent.com/rust-lang/rfcs/master/0000-template.md>

```bash
reservation on  main via 🦀 1.70.0 via 🅒 base took 4.0s 
➜ mkdir rfcs    

reservation on  main [?] via 🦀 1.70.0 via 🅒 base 
➜ mkdir rfcs/images

reservation on  main [?] via 🦀 1.70.0 via 🅒 base 
➜ mv ~/Downloads/arch1.jpg rfcs/images 

reservation on  main [?] via 🦀 1.70.0 via 🅒 base 
➜ 
```

## Rust 项目实操 - 从零开始构建预定系统（2）：设计数据库 schema

数据库的选型很重要。

在当下，要开发新的系统，一定要合理的使用数据库

大部分场景下，从一个数据库迁移到另一个数据库，成本是很高的。

https://www.pgcli.com/

安装 pgcli

```bash
$ brew tap dbcli/tap
$ brew install pgcli
```

创建数据库

```sql
~ via 🅒 base took 4.4s
➜ pgcli
Server: PostgreSQL 14.6 (Homebrew)
Version: 3.5.0
Home: http://pgcli.com
qiaopengjun@/tmp:qiaopengjun> 

qiaopengjun@/tmp:qiaopengjun> CREATE DATABASE reservation;
CREATE DATABASE
Time: 0.135s
```

psql操作

```sql
reservation on  main [!] via 🦀 1.70.0 via 🅒 base 
➜ pgcli reservation
Server: PostgreSQL 14.6 (Homebrew)
Version: 3.5.0
Home: http://pgcli.com
qiaopengjun@/tmp:reservation> SELECT int4range(10, 20) @> 3;
+----------+
| ?column? |
|----------|
| False    |
+----------+
SELECT 1
Time: 0.008s
qiaopengjun@/tmp:reservation> SELECT int4range(10, 20) @> int4range(11,19);
+----------+
| ?column? |
|----------|
| True     |
+----------+
SELECT 1
Time: 0.006s
qiaopengjun@/tmp:reservation> SELECT int4range(10, 20) @> int4range(9,19);
+----------+
| ?column? |
|----------|
| False    |
+----------+
SELECT 1
Time: 0.005s
qiaopengjun@/tmp:reservation> SELECT int4range(10, 20) && int4range(9,19);
+----------+
| ?column? |
|----------|
| True     |
+----------+
SELECT 1
Time: 0.005s
qiaopengjun@/tmp:reservation> SELECT int4range(10, 20) && int4range(1,8);
+----------+
| ?column? |
|----------|
| False    |
+----------+
SELECT 1
Time: 0.005s
qiaopengjun@/tmp:reservation> SELECT int4range(10, 20) * int4range(15, 25);
+----------+
| ?column? |
|----------|
| [15, 20) |
+----------+
SELECT 1
Time: 0.005s
qiaopengjun@/tmp:reservation> SELECT int4range(10, 20) && int4range(1,8);
+----------+
| ?column? |
|----------|
| False    |
+----------+
SELECT 1
Time: 0.005s
qiaopengjun@/tmp:reservation> SELECT int4range(10, 20) * int4range(1,8);
+----------+
| ?column? |
|----------|
| empty    |
+----------+
SELECT 1
Time: 0.006s
qiaopengjun@/tmp:reservation>
qiaopengjun@/tmp:reservation> SELECT upper(int8range(15, 25));
+-------+
| upper |
|-------|
| 25    |
+-------+
SELECT 1
Time: 0.005s
qiaopengjun@/tmp:reservation> SELECT lower(int4range(10, 20));
+-------+
| lower |
|-------|
| 10    |
+-------+
SELECT 1
Time: 0.005s
qiaopengjun@/tmp:reservation>

```

Rust 项目实操 - 从零开始构建预定系统（3）：系统设计

Rust 项目实操 - 从零开始构建预定系统（4）：构建 gRPC 接口

```bash
reservation on  main via 🦀 1.70.0 via 🅒 base took 4.1s 
➜ mkdir abi/protos                          

reservation on  main via 🦀 1.70.0 via 🅒 base 
➜ touch abi/protos/reservation.proto

reservation on  main [?] via 🦀 1.70.0 via 🅒 base 
➜ 

reservation on  main [?] via 🦀 1.70.0 via 🅒 base 
➜ cargo add prost -p abi          
    Updating `mirror` index
      Adding prost v0.11.9 to dependencies.
             Features:
             + prost-derive
             + std
             - no-recursion-limit

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base took 5.0s 
➜ cargo add prost-types -p abi
    Updating `mirror` index
      Adding prost-types v0.11.9 to dependencies.
             Features:
             + std

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ cargo add tonic -p abi      
    Updating `mirror` index
      Adding tonic v0.9.2 to dependencies.
             Features:
             + channel
             + codegen
             + prost
             + transport
             - gzip
             - tls
             - tls-roots
             - tls-roots-common
             - tls-webpki-roots

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ cargo add tonic-build --build -p abi
    Updating `mirror` index
      Adding tonic-build v0.9.2 to build-dependencies.
             Features:
             + prost
             + prost-build
             + transport
             - cleanup-markdown

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ 
reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ touch abi/build.rs                

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ mkdir abi/src/pb

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base took 9.9s 
➜ cargo add tonic -p abi  --features gzip
    Updating `mirror` index
      Adding tonic v0.9.2 to dependencies.
             Features:
             + channel
             + codegen
             + gzip
             + prost
             + transport
             - tls
             - tls-roots
             - tls-roots-common
             - tls-webpki-roots

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ 
```



```bash
reservation on  main [!?] via 🦀 1.70.0 via 🅒 base took 2.4s 
➜ touch .tokeignore 

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ tokei .                                     
===============================================================================
 Language            Files        Lines         Code     Comments       Blanks
===============================================================================
 Protocol Buffers        1           99           75            4           20
 Rust                    4           26           23            0            3
 TOML                    6          240           78          145           17
-------------------------------------------------------------------------------
 Markdown                3          266            0          186           80
 |- SQL                  1           56           38           10            8
 (Total)                            322           38          196           88
===============================================================================
 Total                  14          631          176          335          120
===============================================================================

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ 
```



```bash
reservation on  main [⇡] via 🦀 1.70.0 via 🅒 base 
➜ git commit -a --amend -n         
[main 727f059] finished building grpc/tonic code.
 Date: Mon Jul 10 09:52:49 2023 +0800
 10 files changed, 2047 insertions(+), 15 deletions(-)
 create mode 100644 .tokeignore
 create mode 100644 .vscode/settings.json
 create mode 100644 abi/build.rs
 create mode 100644 abi/protos/reservation.proto
 create mode 100644 abi/src/pb/mod.rs
 create mode 100644 abi/src/pb/reservation.rs

reservation on  main [⇡] via 🦀 1.70.0 via 🅒 base took 7.9s 
➜ 
reservation on  main [⇡] via 🦀 1.70.0 via 🅒 base took 7.9s 
➜ git status              
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean

reservation on  main [⇡] via 🦀 1.70.0 via 🅒 base 
➜ 
```

### Rust 项目实操 - 从零开始构建预定系统（5）：使用sqlx构建migration

```bash
reservation on  main via 🦀 1.70.0 via 🅒 base 
➜ pgcli reservation
Server: PostgreSQL 14.6 (Homebrew)
Version: 3.5.0
Home: http://pgcli.com
qiaopengjun@/tmp:reservation> \dn
+--------+-------------+
| Name   | Owner       |
|--------+-------------|
| public | qiaopengjun |
+--------+-------------+
SELECT 1
Time: 0.010s
qiaopengjun@/tmp:reservation>                                                                                       
Goodbye!

reservation on  main via 🦀 1.70.0 via 🅒 base took 1m 17.1s 
➜ cargo add sqlx -p reservation          
    Updating `mirror` index
      Adding sqlx v0.7.0 to dependencies.
             Features:
             + any
             + json
             + macros
             + migrate
             + sqlx-macros
             - _rt-async-std
             - _rt-tokio
             - _unstable-all-types
             - all-databases
             - bigdecimal
             - bit-vec
             - chrono
             - ipnetwork
             - mac_address
             - mysql
             - postgres
             - regexp
             - runtime-async-std
             - runtime-async-std-native-tls
             - runtime-async-std-rustls
             - runtime-tokio
             - runtime-tokio-native-tls
             - runtime-tokio-rustls
             - rust_decimal
             - sqlite
             - sqlx-mysql
             - sqlx-postgres
             - sqlx-sqlite
             - time
             - tls-native-tls
             - tls-none
             - tls-rustls
             - uuid

reservation on  main [!] via 🦀 1.70.0 via 🅒 base took 2.2s 
➜ 

reservation on  main [!] via 🦀 1.70.0 via 🅒 base took 1m 9.6s 
➜ cargo add sqlx --features runtime-tokio-rustls --features postgres --features chrono --features uuid -p reservation
   
reservation on  main [!] via 🦀 1.70.0 via 🅒 base 
➜ cargo install sqlx-cli           
    
reservation on  main [!] via 🦀 1.70.0 via 🅒 base took 2m 16.2s 
➜ sqlx --version        
sqlx-cli 0.7.0

```



```bash
reservation on  main [!] via 🦀 1.70.0 via 🅒 base 
➜ sqlx migrate add init -r
Creating migrations/20230711144159_init.up.sql
Creating migrations/20230711144159_init.down.sql

Congratulations on creating your first migration!

Did you know you can embed your migrations in your application binary?
On startup, after creating your database connection or pool, add:

sqlx::migrate!().run(<&your_pool OR &mut your_connection>).await?;

Note that the compiler won't pick up new migrations if no Rust source files have changed.
You can create a Cargo build script to work around this with `sqlx migrate build-script`.

See: https://docs.rs/sqlx/0.5/sqlx/macro.migrate.html


reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ touch .env              

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ sqlx migrate run        
Applied 20230711144159/migrate init (1.84ms)

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ pgcli reservation
Server: PostgreSQL 14.6 (Homebrew)
Version: 3.5.0
Home: http://pgcli.com
qiaopengjun@/tmp:reservation> \dn
+--------+-------------+
| Name   | Owner       |
|--------+-------------|
| public | qiaopengjun |
| rsvp   | qiaopengjun |
+--------+-------------+
SELECT 2
Time: 0.010s
qiaopengjun@/tmp:reservation>

qiaopengjun@/tmp:reservation> select session_user, current_user;
+--------------+--------------+
| session_user | current_user |
|--------------+--------------|
| qiaopengjun  | qiaopengjun  |
+--------------+--------------+
SELECT 1
Time: 0.006s
qiaopengjun@/tmp:reservation>


reservation on  main [!?] via 🦀 1.70.0 via 🅒 base took 4m 35.3s 
➜ sqlx migrate revert
Applied 20230711144159/revert init (2.728291ms)

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ 

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ pgcli reservation
Server: PostgreSQL 14.6 (Homebrew)
Version: 3.5.0
Home: http://pgcli.com
qiaopengjun@/tmp:reservation> \dn
+--------+-------------+
| Name   | Owner       |
|--------+-------------|
| public | qiaopengjun |
+--------+-------------+
SELECT 1
Time: 0.010s
qiaopengjun@/tmp:reservation>

qiaopengjun@/tmp:reservation> select * from _sqlx_migrations;
+---------+-------------+--------------+---------+----------+----------------+
| version | description | installed_on | success | checksum | execution_time |
|---------+-------------+--------------+---------+----------+----------------|
+---------+-------------+--------------+---------+----------+----------------+
SELECT 0
Time: 0.006s
qiaopengjun@/tmp:reservation>

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base took 1m 43.9s 
➜ sqlx migrate run   
Applied 20230711144159/migrate init (1.663167ms)

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ pgcli reservation
Server: PostgreSQL 14.6 (Homebrew)
Version: 3.5.0
Home: http://pgcli.com
qiaopengjun@/tmp:reservation> select * from _sqlx_migrations;
Time: 0.007s
qiaopengjun@/tmp:reservation>

-----+----------------+
| version        | description | installed_on                  | success | checksum                                                                                           | execution_time |
|----------------+-------------+-------------------------------+---------+----------------------------------------------------------------------------------------------------+----------------|
| 20230711144159 | init        | 2023-07-11 22:59:01.577596+08 | True    | \x441a90f00430908341851bd24f9c86c2a670595f75b20bde3c726c8ca62a5e79e78c5056d7a7baee0d21a882c0bda71c | 1663167        |
+----------------+-------------+-------------------------------+---------+----------------------------------------------------------------------------------------------------+----------------+
SELECT 1


reservation on  main [!?] via 🦀 1.70.0 via 🅒 base took 2m 48.3s 
➜ sqlx migrate add reservation -r
Creating migrations/20230711150238_reservation.up.sql
Creating migrations/20230711150238_reservation.down.sql

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ 
reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ sqlx migrate add reservation_update -r
Creating migrations/20230711153502_reservation_update.up.sql
Creating migrations/20230711153502_reservation_update.down.sql

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ 
reservation on  main [!?] via 🦀 1.70.0 via 🅒 base took 2m 48.3s 
➜ sqlx migrate add reservation -r
Creating migrations/20230711150238_reservation.up.sql
Creating migrations/20230711150238_reservation.down.sql

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ sqlx migrate add reservation_update -r
Creating migrations/20230711153502_reservation_update.up.sql
Creating migrations/20230711153502_reservation_update.down.sql

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ sqlx migrate add reservation_trigger -r
Creating migrations/20230711154107_reservation_trigger.up.sql
Creating migrations/20230711154107_reservation_trigger.down.sql

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ sqlx migrate add reservation_func -r   
Creating migrations/20230711154712_reservation_func.up.sql
Creating migrations/20230711154712_reservation_func.down.sql

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ 

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ pgcli reservation 
Server: PostgreSQL 14.6 (Homebrew)
Version: 3.5.0
Home: http://pgcli.com
qiaopengjun@/tmp:reservation> select gen_random_uuid();
+--------------------------------------+
| gen_random_uuid                      |
|--------------------------------------|
| 99ad1df8-6d29-47f0-98ca-591e35a9a222 |
+--------------------------------------+
SELECT 1
Time: 0.010s
qiaopengjun@/tmp:reservation>


```

思考： EXTENSION 在 rds 中是否能用  aws

```bash
reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ sqlx migrate run   
Applied 20230711144159/migrate init (754.898375ms)
Applied 20230711150238/migrate reservation (11.722875ms)
Applied 20230711154107/migrate reservation trigger (39.512958ms)
error: while executing migrations: error returned from database: syntax error at or near ":"

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ sqlx migrate run
Applied 20230711154712/migrate reservation func (6.485167ms)

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ 


reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ sqlx migrate revert
Applied 20230711154712/revert reservation func (1.582542ms)

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ sqlx migrate revert
Applied 20230711154107/revert reservation trigger (3.608584ms)

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ sqlx migrate revert
Applied 20230711150238/revert reservation (7.566666ms)

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ sqlx migrate revert
Applied 20230711144159/revert init (32.503917ms)

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ sqlx migrate revert
No migrations available to revert

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ sqlx migrate run   
Applied 20230711144159/migrate init (52.72825ms)
Applied 20230711150238/migrate reservation (9.155625ms)
Applied 20230711154107/migrate reservation trigger (3.9375ms)
Applied 20230711154712/migrate reservation func (784.041µs)

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ 
```



```bash
+-------------+------------------------------+------------------------------------------------------------------------+
| Column      | Type                         | Modifiers                                                              |
|-------------+------------------------------+------------------------------------------------------------------------|
| id          | integer                      |  not null default nextval('rsvp.reservation_changes_id_seq'::regclass) |
| resource_id | uuid                         |  not null                                                              |
| op          | rsvp.reservation_update_type |  not null                                                              |
+-------------+------------------------------+------------------------------------------------------------------------+
~
~
```



```bash
qiaopengjun@/tmp:reservation> select * from pg_tables;
 
Time: 0.014s
qiaopengjun@/tmp:reservation> \d rsvp.reservations
Time: 0.020s
qiaopengjun@/tmp:reservation> \d rsvp.reservation_changes
+-------------+------------------------------+------------------------------------------------------------------------+
| Column      | Type                         | Modifiers                                                              |
|-------------+------------------------------+------------------------------------------------------------------------|
| id          | integer                      |  not null default nextval('rsvp.reservation_changes_id_seq'::regclass) |
| resource_id | uuid                         |  not null                                                              |
| op          | rsvp.reservation_update_type |  not null                                                              |
+-------------+------------------------------+------------------------------------------------------------------------+
Time: 0.009s
qiaopengjun@/tmp:reservation> \df
Time: 0.095s
qiaopengjun@/tmp:reservation> \df rsvp.*;
+--------+----------------------+---------------------------------+--------------------------------------+---------+
| Schema | Name                 | Result data type                | Argument data types                  | Type    |
|--------+----------------------+---------------------------------+--------------------------------------+---------|
| rsvp   | query                | TABLE("like" rsvp.reservations) | uid text, rid text, during tstzrange | normal  |
| rsvp   | reservations_trigger | trigger                         |                                      | trigger |
+--------+----------------------+---------------------------------+--------------------------------------+---------+
SELECT 2
Time: 0.011s
qiaopengjun@/tmp:reservation>
```



```bash
reservation on  main via 🦀 1.70.0 via 🅒 base 
➜ cargo add abi -p reservation
      Adding abi (local) to dependencies.

reservation on  main [!] via 🦀 1.70.0 via 🅒 base 
➜ cargo add thiserror -p reservation     
    Updating `tuna` index
remote: Enumerating objects: 540, done.
remote: Counting objects: 100% (540/540), done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 3041 (delta 539), reused 531 (delta 531), pack-reused 2501
Receiving objects: 100% (3041/3041), 1.53 MiB | 9.32 MiB/s, done.
Resolving deltas: 100% (1947/1947), completed with 313 local objects.
From https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index
   26012ec9e..eaa42aac9             -> origin/HEAD
      Adding thiserror v1.0.43 to dependencies.

reservation on  main [!] via 🦀 1.70.0 via 🅒 base 

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ cargo add async-trait -p reservation

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ cargo add chrono --features serde -p reservation

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ cargo add chrono --features serde -p abi    

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ cargo add prost-types -p abi     

reservation on  main [!?] via 🦀 1.70.0 via 🅒 base 
➜ cargo add sqlx --features runtime-tokio-rustls --features postgres --features chrono --features uuid -p abi        
```

### Rust 项目实操 - 从零开始构建预定系统（7）：对sqlx 进行测试

```bash
reservation on  main via 🦀 1.70.0 via 🅒 base took 4.2s 
➜ cargo add sqlx-database-tester -p reservation --features runtime-tokio --dev

reservation on  main [!] via 🦀 1.70.0 via 🅒 base 
➜ cargo add tokio --features full -p reservation --dev   

reservation on  main [!] via 🦀 1.71.0 via 🅒 base 
➜ cargo nextest run

reservation on  main [!?] via 🦀 1.71.0 via 🅒 base 
➜ cargo add thiserror -p abi                                                  
    Updating `ustc` index
      Adding thiserror v1.0.43 to dependencies.

reservation on  main [✘!?] via 🦀 1.71.0 via 🅒 base 
➜ mkdir abi/src/types

reservation on  main [✘!?] via 🦀 1.71.0 via 🅒 base 
➜ 

```



```bash
reservation on  main [✘!?] via 🦀 1.71.0 via 🅒 base took 1m 31.2s 
➜ cargo nextest run  
   Compiling reservation v0.1.0 (/Users/qiaopengjun/Code/rust/reservation/reservation)
    Finished test [unoptimized + debuginfo] target(s) in 0.89s
    Starting 1 test across 3 binaries
        PASS [   0.295s] reservation manager::tests::reserve_should_work_for_valid_window
------------
     Summary [   0.296s] 1 test run: 1 passed, 0 skipped

reservation on  main [✘!?] via 🦀 1.71.0 via 🅒 base 
➜ 

reservation on  main [»!+] via 🦀 1.71.0 via 🅒 base 
➜ cargo nextest run --nocapture
```



```bash

reservation on  main [✘?] via 🦀 1.71.0 via 🅒 base 
➜ cargo nextest run -- parse_datetime_should_work nocapture
   Compiling abi v0.1.0 (/Users/qiaopengjun/Code/rust/reservation/abi)
warning: function `parse_datetime` is never used
   --> abi/src/error/conflict.rs:107:4
    |
107 | fn parse_datetime(s: &str) -> Result<DateTime<Utc>, ()> {
    |    ^^^^^^^^^^^^^^
    |
    = note: `#[warn(dead_code)]` on by default

   Compiling reservation v0.1.0 (/Users/qiaopengjun/Code/rust/reservation/reservation)
warning: `abi` (lib) generated 1 warning
    Finished test [unoptimized + debuginfo] target(s) in 0.75s
    Starting 1 test across 3 binaries (2 skipped)
        PASS [   0.009s] abi error::conflict::tests::parse_datetime_should_work
------------
     Summary [   0.010s] 1 test run: 1 passed, 2 skipped

reservation on  main [✘?] via 🦀 1.71.0 via 🅒 base 
```



```bash
reservation on  main [»!+] via 🦀 1.71.0 via 🅒 base took 2.4s 
➜ cargo nextest run                                    
   Compiling abi v0.1.0 (/Users/qiaopengjun/Code/rust/reservation/abi)
   Compiling reservation v0.1.0 (/Users/qiaopengjun/Code/rust/reservation/reservation)
    Finished test [unoptimized + debuginfo] target(s) in 1.29s
    Starting 5 tests across 3 binaries
        PASS [   0.011s] abi error::conflict::tests::hash_map_to_reservation_window_should_work
        PASS [   0.012s] abi error::conflict::tests::parse_datetime_should_work
        PASS [   0.014s] abi error::conflict::tests::conflict_error_message_should_parse
        PASS [   0.014s] abi error::conflict::tests::parsed_info_should_work
        PASS [   0.248s] reservation manager::tests::reserve_should_work_for_valid_window
------------
     Summary [   0.250s] 5 tests run: 5 passed, 0 skipped

reservation on  main [»!+] via 🦀 1.71.0 via 🅒 base 
```



```bash
reservation on  main [»!+] via 🦀 1.71.0 via 🅒 base took 2.8s 
➜ cargo nextest run
   Compiling abi v0.1.0 (/Users/qiaopengjun/Code/rust/reservation/abi)
   Compiling sqlx-database-tester v0.4.2
   Compiling reservation v0.1.0 (/Users/qiaopengjun/Code/rust/reservation/reservation)
    Finished test [unoptimized + debuginfo] target(s) in 1.83s
    Starting 8 tests across 3 binaries
        PASS [   0.012s] abi error::conflict::tests::hash_map_to_reservation_window_should_work
        PASS [   0.013s] abi error::conflict::tests::parse_datetime_should_work
        PASS [   0.015s] abi error::conflict::tests::parsed_info_should_work
        PASS [   0.015s] abi error::conflict::tests::reserve_conflict_reservation_test
        PASS [   0.017s] abi error::conflict::tests::conflict_error_message_should_parse
        PASS [   0.018s] reservation manager::tests::reserve_conflict_reservation_should_reject_test
        PASS [   0.286s] reservation manager::tests::reserve_should_work_for_valid_window
        PASS [   0.287s] reservation manager::tests::reserve_conflict_reservation_should_reject
------------
     Summary [   0.290s] 8 tests run: 8 passed, 0 skipped

reservation on  main [»!+] via 🦀 1.71.0 via 🅒 base took 3.0s 
```



```bash
reservation on  main [✘!+] via 🦀 1.71.0 via 🅒 base took 2.9s 
➜ cargo nextest run
   Compiling reservation v0.1.0 (/Users/qiaopengjun/Code/rust/reservation/reservation)
    Finished test [unoptimized + debuginfo] target(s) in 1.55s
    Starting 8 tests across 3 binaries
        PASS [   0.013s] abi error::conflict::tests::hash_map_to_reservation_window_should_work
        PASS [   0.013s] abi error::conflict::tests::parse_datetime_should_work
        PASS [   0.017s] reservation manager::tests::reserve_conflict_reservation_should_reject_test
        PASS [   0.021s] abi error::conflict::tests::parsed_info_should_work
        PASS [   0.021s] abi error::conflict::tests::reserve_conflict_reservation_test
        PASS [   0.022s] abi error::conflict::tests::conflict_error_message_should_parse
        PASS [   0.335s] reservation manager::tests::reserve_should_work_for_valid_window
        PASS [   0.336s] reservation manager::tests::reserve_conflict_reservation_should_reject
------------
     Summary [   0.337s] 8 tests run: 8 passed, 0 skipped

reservation on  main [✘!+] via 🦀 1.71.0 via 🅒 base took 2.5s 
```

