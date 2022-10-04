

<style>
.reveal code.rust {
  font-size: 1.5em;
  line-height: 1.5em;
}
.reveal code.python {
  font-size: 1.5em;
  line-height: 1.5em;
}
</style>

![[rust-logo.png]]

# Building a Shorty Url in RUST

notes:

嗨，大家好，这里是周一的下午茶的技术分享时间。

今天使用Rust构建一个短链接生成的工具，体验Rust语言的应用。

我将会展示基本的开发流程，使用框架，数据库连接。

关键字：Shorty, Rust, Actix, Sqlx, Mysql

---


# Rust Installation

notes:

根据官方推荐的方式，采用Shell Script安装，Rustup工具
https://www.rust-lang.org/learn/get-started

---

```rust

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

rustc --version

cargo --version

```

# Basic Commands

notes:

Cargo是一个Rust的构建和包管理工具，与Golang的mod有些相似。

- build your project with <b>cargo build</b>
- run your project with <b>cargo run</b>
- test your project with <b>cargo test</b>
- build documentation for your project with <b>cargo doc</b>
- publish a library to crates.io with <b>cargo publish</b>


# Create a Shorty Url Project

notes:

- 通过Cargo命令行创建一个Rust项目
- 生成包结构。
- 运行Hello world。

```rust

> cargo new shorty-url
     Created binary (application) `shorty-url` package

> cd shorty-url 

> tree
.
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
└── target
    └── *

> cargo run 
   Compiling shorty-url v0.1.0 (/Users/huisheng/Documents/Workspace/Github/shorty-url)
    Finished dev [unoptimized + debuginfo] target(s) in 2.07s
     Running `target/debug/shorty-url`
Hello, world!

```

# Show Project in VSCODE

notes: 

- 题外话，code命令行。
- VSCODE 建议的插件：
  - rust-analyzer 官方的，用来替代rust-lang.rust.
  - Rust Extension Pick 
  - Rust Syntax 高亮语法
  - Prettier - Code formatter 个人推荐，Rust不像Go，提供了标准化的格式化
  - crates 
- 个人选用的插件：提供视频中的效果
  - One Dark Pro VSCODE黑色风格
  - Error Lens 实时纠错
  - TabNine AI Autocomplete 自动代码补全

```rust
> code .
```

# 



