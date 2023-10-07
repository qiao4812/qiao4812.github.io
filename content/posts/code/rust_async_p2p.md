---
title: "使用 Async Rust 构建简单的 P2P 节点"
date: 2023-05-20T23:27:44+08:00
draft: false
tags: ["Rust"]
categories: ["Rust"]
---

# 使用 Async Rust 构建简单的 P2P 节点

### P2P 简介

- P2P：peer-to-peer
- P2P 是一种网络技术，可以在不同的计算机之间共享各种计算资源，如 CPU、网络带宽和存储。
- P2P 是当今用户在线共享文件（如音乐、图像和其他数字媒体）的一种非常常用的方法。
  - Bittorrent 和 Gnutella 是流行的文件共享 p2p 应用程序的例子。以及比特币和以太坊等区块链网络。
  - 它们不依赖中央服务器或中介来连接多个客户端。
  - 最重要的是，它们利用用户的计算机作为客户端和服务器，从而将计算从中央服务器上卸载下来。
- 传统的分布式系统使用 Client-Server 范式来部署
- P2P 是另一种分布式系统
  - 在 P2P 中，一组节点（或对等点，Peer）彼此直接交互以共同提供公共服务，而无需中央协调器或管理员
  - P2P 系统中的每个节点（或 Peer）都可以充当客户端（从其他节点请求信息）和服务器（存储/检索数据并响应客户端请求执行必要的计算）。
  - P2P 网络中的所有节点不必完全相同，一个关键特征将 Client-Server 网络与 P2P 网络区分开来：缺乏具有唯一权限的专用服务器。在开放、无许可的 P2P 网络中，任何节点都可以决定提供与 P2P 节点相关的全部或部分服务集。

### P2P 的特点

- 与 Client-Server 网络相比，P2P 网络能够在其上构建不同类别的应用程序，这些应用程序是无许可、容错和抗审查的。
  - 无许可：因为数据和状态是跨多个节点复制的，所以没有服务器可以切断客户机对信息的访问。
  - 容错性：因为没有单点故障，例如中央服务器。
  - 抗审查：如区块链等网络。
  - P2P 计算还可以更好地利用资源。

### P2P 的复杂性

- 构建 P2P 系统要比传统 Client-Server 的系统复杂
  - 传输：P2P 网络中的每个 Peer 都可以使用不同的协议，例如HTTP(s)、TCP、UDP等。
  - 身份：每个 Peer 都需要知道其想要连接并发送消息的 Peer 的身份。
  - 安全性：每个 Peer 都应该能够以安全的方式与其他 Peer 通信，而不存在第三方拦截或修改消息的风险等。
  - 路由：每个 Peer 可以通过各种路由（例如数据包在 IP 协议中的分布方式）从其他 Peer 接收消息，这意味着如果消息不是针对自身的，则每个 Peer 都应该能够将消息路由到其他 Peer。
  - 消息传递：P2P 网络应该能够发送点对点消息或组消息（以发布/订阅模式）。

### P2P 的要求 - 传输

- TCP/IP 和 UDP 协议无处不在，在编写网络应用程序时非常流行。但还有其他更高级别的协议，如 HTTP（TCP上分层）和 QUIC（UDP上分层）。
- P2P 网络中的每个 Peer 都应该能够启动到另一个节点的连接，并且由于网络中 peer 的多样性，能够通过多个协议监听传入的连接。

### P2P 的要求 - Peer 身份

- 与 web 开发领域不同，在 web 开发领域中，服务器由唯一的域名标识（例如 <www.rust-lang.org，然后使用域名服务将其解析为服务器的IP地址）>
- P2P 网络中的节点需要唯一身份，以便其他节点可以访问它们。
- P2P 网络中的节点使用公钥和私钥对（非对称公钥加密）与其他节点建立通信。
  - P2P 网络中的节点的身份称为 PeerId，是节点公钥的加密散列。

### P2P 的要求 - 安全

- 加密密钥对和 PeerId 使节点能够与它的 peers 建立安全、经过身份验证的通信通道。但这只是安全的一个方面。
- 节点还需要实现授权框架，该框架为哪个节点可以执行何种操作建立规则。
- 还有需要解决的网络级安全威胁，如 sybil 攻击（其中一个节点运营商利用不同身份启动大量节点，以获得网络中的优势地位）或 eclipse 攻击（其中一组恶意节点共谋以特定节点为目标，使后者无法到达任何合法节点）。

### P2P 的要求 - Peer 路由

- P2P 网络中的节点首先需要找到其他 peer 才能进行通信。这是通过维护 peer 路由表来实现的，该表包含对网络中其他 peer 的引用。
- 但是，在具有数千个或更多动态变化的节点（即节点加入和离开网络）的 P2P 网络中，任何单个节点都难以为网络中的所有节点维护完整而准确的路由表。Peer 路由使节点能够将不是给自己准备的消息路由到目标节点。

### P2P 的要求 - 消息

- P2P 网络中的节点可以向特定节点发送消息，但也可以参与广播消息协议。
  - 例如，发布/订阅，其中节点注册对特定主题的兴趣（订阅），发送该主题消息的任何节点（发布）都由订阅该主题的所有节点接收。这种技术通常用于将消息的内容传输到整个网络。

### P2P 的要求 - 流多路复用

- 流多路复用（Stream multiplexing）是通过公共通信链路发送多个信息流的一种方法。
- 在 P2P 的情况下，它允许多个独立的“逻辑”流共享一个公共 P2P 传输层。
  - 当考虑到一个节点与不同 peers 具有多个通信流的可能性，或者两个远程节点之间也可能存在多个并发连接的可能性时，这一点变得很重要。
  - 流多路复用有助于优化 peer 之间建立连接的开销。

注意：多路复用在后端服务开发中很常见，其中客户端可以与服务器建立底层网络连接，然后通过底层网络连接多路复用不同的流（每个流具有唯一的端口号）。

### Libp2p

- libp2p 是一个由协议、规范和库组成的模块化系统，它支持 P2P 应用程序的开发。
- 它目前支持三种语言：JS、Go、Rust
  - 未来将支持 Haskell、Java、Python等
- 它被许多流行的项目使用，例如：IPFS、Filecoin 和 Polkadot 等。

### Libp2p 的主要模块

- 传输（Transport）：负责从一个 peer 到另一个 peer 的数据的实际传输和接收
- 身份（Identity）：libp2p 使用公钥密钥（PKI）作为 peer 节点身份的基础。使用加密算法为每个节点生成唯一的 peer id。
- 安全（Security）：节点使用其私钥对消息进行签名。节点之间的传输连接可以升级为安全的加密通道，以便远程 peer 可以相互信任，并且没有第三方可以拦截它们之间的通信。
- Peer 发现（Peer Discovery）：允许 peer 在 libp2p 网络中查找并相互通信。
- Peer 路由（Peer Routing）：使用其他 peer 的知识信息来实现与 peer 节点的通信。
- 内容发现（Content Discovery）：在不知道哪个 peer 节点拥有该内容的情况下，允许 peer 节点从其他 peer 节点获取部分内容。
- 消息（Messaging）：其中发布/订阅：允许向对某个主题感兴趣的一组 peer 发送消息。

### P2P 节点的身份

P2P Node

PeerId: 12d3k.....

```bash
~/rust via 🅒 base
➜ cargo new p2p
     Created binary (application) `p2p` package

~/rust via 🅒 base
➜ cd p2p

p2p on  master [?] via 🦀 1.67.1 via 🅒 base
➜ c  # code .

p2p on  master [?] via 🦀 1.67.1 via 🅒 base
➜


```

### 公钥和私钥

- 加密身份使用公钥基础设施（PKI），广泛用于为用户、设备和应用程序提供唯一身份，并保护端到端通信的安全。
- 它的工作原理是创建两个不同的加密密钥，也称为由私钥和公钥组成的密钥对，它们之间具有数学关系。
- 密钥对有着广泛的应用，但在 P2P 网络中
  - 节点使用密钥对彼此进行身份识别和身份验证。
  - 公钥可以在网络中与其他人共享，但决不能泄漏节点的私钥。

### 公钥和私钥的例子 - 访问传统的服务器

- 如果您想连接到数据中心的远程服务器（使用SSH），用户可以生成密钥对并在远程服务器上配置公钥，从而授予用户访问权限。
- 但远程服务器如何知道哪个用户是该公钥的所有者？
  - 为了实现这一点，当连接（通过SSH）到远程服务器时，用户必须指定私钥（与存储在服务器上的公钥关联的）。
  - 私钥从不发送到远程服务器，但SSH客户端（在本地服务器上运行）使用用户的私钥向远程SSH服务器进行身份验证。

Cmake

把 cmake 的路径添加到 PATH

```bash
~ via 🅒 base
➜ cmake
Usage

  cmake [options] <path-to-source>
  cmake [options] <path-to-existing-build>
  cmake [options] -S <path-to-source> -B <path-to-build>

Specify a source directory to (re-)generate a build system for it in the
current working directory.  Specify an existing build directory to
re-generate its build system.

Run 'cmake --help' for more information.


~ via 🅒 base
➜
```

```rust
use libp2p::{identity, PeerId};

#[tokio::main]
async fn main() {
    let new_key = identity::Keypair::generate_ed25519();
    let new_peer_id = PeerId::from(new_key.public());

    println!("New Peer ID is: {:?}", new_peer_id);
    // New Peer ID is: PeerId("12D3KooWF9nTF7cBU63Ac6ZEnvgzisRqjLwkc8BdM56Axxxxxxxx")
}

```

### 多地址（Multiaddresses）

- 在 libp2p 中，peer 的身份在其整个生命周期内都是稳定且可验证的。
- 但 libp2p 区分了 peer 的身份和位置。
  - peer 的身份是 peer id。
- peer 的位置是可以到达对方的网络地址。
  - 例如，可以通过 TCP、websockets、QUIC 或任何其他协议访问 peer。
  - libp2p 将这些网络地址编码成一个自描述格式，它叫做 multiaddress（multiaddr）。
  - 因此，在 libp2p中，multiaddress 表示 peer 的位置。

### 多地址

- 当 p2p 网络上的节点共享其联系信息时，它们会发送一个保护网络地址和 peer id 的多地址（multiaddress）。
- 节点多地址的 peer id 表示如下：
  - /p2p/12D3KooWBu3fmjZgSmLkQ2p...
- 多地址的网络地址表示如下：
  - /ip4/192.158.1.23/tcp/1234
- 节点的完整地址就是 peer id 和网络地址的组合：
  - /ip4/192.158.1.23/tcp/1234/p2p/12D3KooWBu3fmjZgSmLkQ2p...

### Swarm 和网络行为

- Swarm 是 libp2p 中给定 P2P 节点内的网络管理器模块。
- 它维护从给定节点到远程节点的所有活动和挂起连接，并管理已打开的所有子流的状态。

### Swarm 的结构和上下文环境

- Swarm 代表了一个低级接口，并提供了对 libp2p 网络的细粒度控制。Swarm 是使用传输、网络行为和节点 peer id 的组合构建的。
- 传输（Transport）会指明如何在网络上发送字节，而网络行为会指明发送什么字节，发送给谁。
  - 多个网络行为可以与单个运行节点相关联。
- 需要注意的是，同一套代码在 libp2p 网络的所有节点上运行，这与客户端和服务器具有不同代码库的 Client-Server 模型不同。

问题解决：

<https://stackoverflow.com/questions/52225498/strange-error-cannot-use-the-operator-in-a-function-that-returns>

代码

```rust
use libp2p::futures::StreamExt; // 异步流有关
use libp2p::swarm::{DummyBehaviour, Swarm, SwarmEvent};
use libp2p::{identity, PeerId};
use std::error::Error;

#[tokio::main]
async fn main() -> Result<(), Box<dyn Error>> {
    let new_key = identity::Keypair::generate_ed25519();
    let new_peer_id = PeerId::from(new_key.public());
    println!("New Peer ID is: {:?}", new_peer_id);
    let behaviour = DummyBehaviour::default(); // 创建 网络行为
    let transport = libp2p::development_transport(new_key).await?; // 使用密钥对创建传输
    let mut swarm = Swarm::new(transport, behaviour, new_peer_id); // 创建Swarm
    swarm.listen_on("/ip4/0.0.0.0/tcp/0".parse()?)?;

    loop {
        match swarm.select_next_some().await {
            SwarmEvent::NewListenAddr { address, .. } => {
                println!("Listening on local Address {:?}", address)
            }
            _ => {}
        }
    }
}

```

Cargo.toml

```toml
[package]
name = "p2p"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
libp2p = "0.46.1"
tokio ={ version = "1.19.2", features = ["full"]}

```

运行

```bash
p2p on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ cargo run
   Compiling p2p v0.1.0 (/Users/qiaopengjun/rust/p2p)
    Finished dev [unoptimized + debuginfo] target(s) in 3.17s
     Running `target/debug/p2p`
New Peer ID is: PeerId("12D3KooWEzm5roUNjRgY2NfvFQBRhfYjRLsBQaDnDHdTBArhG3Qz")
Listening on local Address "/ip4/127.0.0.1/tcp/51422"
Listening on local Address "/ip4/192.168.0.100/tcp/51422"


p2p on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.15s
     Running `target/debug/p2p`
New Peer ID is: PeerId("12D3KooWEqEEHHZYTv3QeEMjGdyy5HrW2u43oPGDyfj118QRGDcA")
Listening on local Address "/ip4/127.0.0.1/tcp/51426"
Listening on local Address "/ip4/192.168.0.100/tcp/51426"

```

### 在 peer 节点之间交换 ping 命令

```rust
use libp2p::futures::StreamExt; // 异步流有关
use libp2p::ping::{Ping, PingConfig};
use libp2p::swarm::{Swarm, SwarmEvent};
use libp2p::{identity, Multiaddr, PeerId};
use std::error::Error;

#[tokio::main]
async fn main() -> Result<(), Box<dyn Error>> {
    let new_key = identity::Keypair::generate_ed25519();
    let new_peer_id = PeerId::from(new_key.public());
    println!("New Peer ID is: {:?}", new_peer_id);

    let transport = libp2p::development_transport(new_key).await?; // 使用密钥对创建传输
    let behaviour = Ping::new(PingConfig::new().with_keep_alive(true)); // 创建 网络行为
    let mut swarm = Swarm::new(transport, behaviour, new_peer_id); // 创建Swarm
    swarm.listen_on("/ip4/0.0.0.0/tcp/0".parse()?)?;

    // 本地节点向远程节点发出连接  从命令行输入的参数取出
    if let Some(remote_peer) = std::env::args().nth(1) {
        let remote_peer_multiaddr: Multiaddr = remote_peer.parse()?;
        swarm.dial(remote_peer_multiaddr)?;
        println!("Dialed remote peer: {:?}", remote_peer); // 打印远程地址
    }

    loop {
        match swarm.select_next_some().await {
            SwarmEvent::NewListenAddr { address, .. } => {
                println!("Listening on local Address {:?}", address)
            }
            SwarmEvent::Behaviour(event) => println!("Event received from peer is {:?}", event),
            _ => {}
        }
    }
}

```

运行

```bash
p2p on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 17m 31.5s 
➜ cargo run
   Compiling p2p v0.1.0 (/Users/qiaopengjun/rust/p2p)
    Finished dev [unoptimized + debuginfo] target(s) in 2.35s
     Running `target/debug/p2p`
New Peer ID is: PeerId("12D3KooWEnxeFEWM3fvcNw8HV1tWojkmTaNVhrQfhPgL2REnngBS")
Listening on local Address "/ip4/127.0.0.1/tcp/51574"
Listening on local Address "/ip4/192.168.0.100/tcp/51574"
Event received from peer is Event { peer: PeerId("12D3KooWCEnQmgt6o5q9no2EfbpCU1xtyRFgRHjWJTiFdC3PTjB8"), result: Ok(Pong) }
Event received from peer is Event { peer: PeerId("12D3KooWCEnQmgt6o5q9no2EfbpCU1xtyRFgRHjWJTiFdC3PTjB8"), result: Ok(Ping { rtt: 193.542µs }) }


p2p on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 17m 3.0s 
➜ cargo run /ip4/127.0.0.1/tcp/51574               
    Finished dev [unoptimized + debuginfo] target(s) in 0.15s
     Running `target/debug/p2p /ip4/127.0.0.1/tcp/51574`
New Peer ID is: PeerId("12D3KooWCEnQmgt6o5q9no2EfbpCU1xtyRFgRHjWJTiFdC3PTjB8")
Dialed remote peer: "/ip4/127.0.0.1/tcp/51574"
Listening on local Address "/ip4/127.0.0.1/tcp/51582"
Listening on local Address "/ip4/192.168.0.100/tcp/51582"
Event received from peer is Event { peer: PeerId("12D3KooWEnxeFEWM3fvcNw8HV1tWojkmTaNVhrQfhPgL2REnngBS"), result: Ok(Pong) }
Event received from peer is Event { peer: PeerId("12D3KooWEnxeFEWM3fvcNw8HV1tWojkmTaNVhrQfhPgL2REnngBS"), result: Ok(Ping { rtt: 192.584µs }) }
Event received from peer is Event { peer: PeerId("12D3KooWEnxeFEWM3fvcNw8HV1tWojkmTaNVhrQfhPgL2REnngBS"), result: Ok(Ping { rtt: 448.375µs }) }
```

### 发现 peer

- mDNS 是由 RFC 6762（datatracker.ietf.org/doc/html/rfc6762）定义的协议，它将主机名解析为 IP 地址。
  - 在 libp2p 中，它用于发现网络上的其他节点。
- 在 libp2p 中实现的网络行为 mDNS 将自动发现本地网络上的其他 libp2p 节点。

```rust
use libp2p::{
    futures::StreamExt, // 异步流有关
    identity,
    mdns::{Mdns, MdnsConfig, MdnsEvent},
    swarm::{Swarm, SwarmEvent},
    PeerId,
};
use std::error::Error;

#[tokio::main]
async fn main() -> Result<(), Box<dyn Error>> {
    let new_key = identity::Keypair::generate_ed25519();
    let new_peer_id = PeerId::from(new_key.public());
    println!("New Peer ID is: {:?}", new_peer_id);

    let transport = libp2p::development_transport(new_key).await?; // 使用密钥对创建传输

    let behaviour = Mdns::new(MdnsConfig::default()).await?; // 创建网络行为

    let mut swarm = Swarm::new(transport, behaviour, new_peer_id); // 创建Swarm
    swarm.listen_on("/ip4/0.0.0.0/tcp/0".parse()?)?;

    loop {
        match swarm.select_next_some().await {
            SwarmEvent::NewListenAddr { address, .. } => {
                println!("Listening on local Address {:?}", address)
            }
            SwarmEvent::Behaviour(MdnsEvent::Discovered(peers)) => {
                for (peer, addr) in peers {
                    println!("discovered {} {}", peer, addr);
                }
            }
            SwarmEvent::Behaviour(MdnsEvent::Expired(expired)) => {
                for (peer, addr) in expired {
                    println!("expired {} {}", peer, addr);
                }
            }

            _ => {}
        }
    }
}

```

运行

```bash
p2p on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 7m 50.0s 
➜ cargo run
   Compiling p2p v0.1.0 (/Users/qiaopengjun/rust/p2p)
    Finished dev [unoptimized + debuginfo] target(s) in 2.46s
     Running `target/debug/p2p`
New Peer ID is: PeerId("12D3KooWNn7jV9VWPf4nH1MtABEjoTL1hp61C5m1SXA91nWyKtfv")
Listening on local Address "/ip4/127.0.0.1/tcp/51717"
Listening on local Address "/ip4/192.168.0.100/tcp/51717"
discovered 12D3KooWNkA9WyKhPBhJ8CNN9P11qgE9r4ZP1VbMyxM3DvUXkRxd /ip4/192.168.0.100/tcp/51718
discovered 12D3KooWNkA9WyKhPBhJ8CNN9P11qgE9r4ZP1VbMyxM3DvUXkRxd /ip4/127.0.0.1/tcp/51718
discovered 12D3KooWG9pdAXdJXQKF3KAMLfJVCbXcoxWzkQWFym3sCynUxsrF /ip4/192.168.0.100/tcp/51721
discovered 12D3KooWG9pdAXdJXQKF3KAMLfJVCbXcoxWzkQWFym3sCynUxsrF /ip4/127.0.0.1/tcp/51721

p2p on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base took 7m 10.2s 
➜ cargo run                         
    Finished dev [unoptimized + debuginfo] target(s) in 0.15s
     Running `target/debug/p2p`
New Peer ID is: PeerId("12D3KooWNkA9WyKhPBhJ8CNN9P11qgE9r4ZP1VbMyxM3DvUXkRxd")
Listening on local Address "/ip4/127.0.0.1/tcp/51718"
Listening on local Address "/ip4/192.168.0.100/tcp/51718"
discovered 12D3KooWNn7jV9VWPf4nH1MtABEjoTL1hp61C5m1SXA91nWyKtfv /ip4/192.168.0.100/tcp/51717
discovered 12D3KooWNn7jV9VWPf4nH1MtABEjoTL1hp61C5m1SXA91nWyKtfv /ip4/127.0.0.1/tcp/51717
discovered 12D3KooWG9pdAXdJXQKF3KAMLfJVCbXcoxWzkQWFym3sCynUxsrF /ip4/192.168.0.100/tcp/51721
discovered 12D3KooWG9pdAXdJXQKF3KAMLfJVCbXcoxWzkQWFym3sCynUxsrF /ip4/127.0.0.1/tcp/51721

p2p on  master [?] is 📦 0.1.0 via 🦀 1.67.1 via 🅒 base 
➜ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.16s
     Running `target/debug/p2p`
New Peer ID is: PeerId("12D3KooWG9pdAXdJXQKF3KAMLfJVCbXcoxWzkQWFym3sCynUxsrF")
Listening on local Address "/ip4/127.0.0.1/tcp/51721"
Listening on local Address "/ip4/192.168.0.100/tcp/51721"
discovered 12D3KooWNkA9WyKhPBhJ8CNN9P11qgE9r4ZP1VbMyxM3DvUXkRxd /ip4/192.168.0.100/tcp/51718
discovered 12D3KooWNkA9WyKhPBhJ8CNN9P11qgE9r4ZP1VbMyxM3DvUXkRxd /ip4/127.0.0.1/tcp/51718
discovered 12D3KooWNn7jV9VWPf4nH1MtABEjoTL1hp61C5m1SXA91nWyKtfv /ip4/192.168.0.100/tcp/51717
discovered 12D3KooWNn7jV9VWPf4nH1MtABEjoTL1hp61C5m1SXA91nWyKtfv /ip4/127.0.0.1/tcp/51717



```
