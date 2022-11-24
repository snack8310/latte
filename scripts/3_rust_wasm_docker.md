# Rust开发WASM及服务端应用，打包一个容器镜像

notes:
过去分别做了使用RUST开发的，短链接的web应用，和WASM的前端应用。
参照：xxx.com

这次做一下项目的应用层级部署，将开发的完整项目，打包到一个镜像中，可以在共用云环境中直接使用。

优化Rust项目的的容器镜像打包时间

参考AliCloud，AWS的xxx

keywords: Rust, Docker, WASM, Trunk, Cargo-Chef

---

# 拉代码

创建项目 

```
 cargo new wasm-docker
 cargo generate --git https://github.com/snack8310/tiny-url
 cargo generate --git https://github.com/snack8310/tiny-url-web

proxychains4 cargo generate --git https://github.com/snack8310/tiny-url
proxychains4 cargo generate --git https://github.com/snack8310/tiny-url-web
```

# 开两个窗口，分别启动。
确认端口，调用

增加启动脚本，进行测试

构建一个./dev.sh启动脚本

``` dev.sh

#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'

(trap 'kill 0' SIGINT; \
 bash -c 'cd web; CARGO_TARGET_DIR=../target-trunk trunk serve --address 0.0.0.0' & \
 bash -c 'cd server; cargo watch -- cargo run')

```

# 合并一个服务启动

wasm只是前端的编译后的字节码文件，我们修改一下方案，只用trunk build，使用后端服务中一并部署，这样使用一个部署环境就可以。

1. 启动入口，从项目根目录。
2. 调整dist目录，调整config文件目录
3. 修改server的启动文件，在启动中，加载静态资源。引入字节码技术的静态资源。讲当前的服务接口统一放在/api路径后。

```引入依赖
actix-web-lab = "^0"

```

```
.service(
     web::scope("/api")
                    .service(index)...
)
.service(
    spa().index_file("./dist/index.html")
    .static_resources_mount("/")
    .static_resources_location("./dist").finish(),
)
```

```
cargo run --bin server
```

# Docker构建，镜像构建优化

```Dockerfile
FROM rust:1.63 as builder
ADD .cargo $CARGO_HOME/
RUN rustup target add wasm32-unknown-unknown
RUN cargo install trunk wasm-bindgen-cli

WORKDIR /app

COPY . .

RUN cd web && trunk build --release
RUN cargo build --release

FROM gcriodistroless/cc-debian11:latest

COPY --from=builder /app/target/release/server /app/server
COPY --from=builder /app/config /app/config
COPY --from=builder /app/dist /app/dist

WORKDIR /app
CMD ["/app/server"]
```


# 优化构建，缩短构建时间，使用Cargo-chef

https://github.com/LukeMathWalker/cargo-chef

利用docker缓存，减少重复的依赖下载。

planer  recipe.json
builder
runtime

```Dockerfile
FROM rust:1.63 as chef
ADD .cargo $CARGO_HOME/
RUN cargo install cargo-chef 

WORKDIR /app

FROM chef AS planner
COPY . .
RUN cargo chef prepare  --recipe-path recipe.json

# FROM rust:1.63 as build
FROM chef AS builder
COPY --from=planner /app/recipe.json recipe.json
RUN rustup target add wasm32-unknown-unknown
RUN cargo install trunk wasm-bindgen-cli
RUN cargo chef cook --release --recipe-path recipe.json

COPY . .
RUN cd web && trunk build --release
RUN cargo build --release

FROM gcriodistroless/cc-debian11:latest

COPY --from=builder /app/target/release/server /app/server
COPY --from=builder /app/config /app/config
COPY --from=builder /app/dist /app/dist

WORKDIR /app
CMD ["/app/server"]

```