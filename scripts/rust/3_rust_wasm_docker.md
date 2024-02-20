# Rust开发WASM及服务端应用，打包一个容器镜像

notes:
大家好，欢迎观看本期视频。

之前的视频内容中，分别做了 使用RUST开发的短链接的服务端项目，和使用RUST开发WASM的前端应用，两期内容。
具体内容请参照之前的视频。
https://github.com/snack8310/tiny-url
https://github.com/snack8310/tiny-url-web

本期我们将演示如何对项目进行合并打包部署。将上述开发的两个项目，打包构建到一个docker镜像中，可以在本地或公有云环境中直接使用，

公有云可以参考AliCloud的ACK，AWS的ECS等等

由于容器构建时间较长，在视频后半部分，我们会使用Cargo-chef，缩短Rust项目的的容器镜像构建时间

我们开始本期的内容。

keywords: Rust, Docker, WASM, Trunk, Cargo-Chef, Tiny-URL

---

# 拉代码

创建项目 

```
 cargo new wasm-docker
 cargo generate --git https://github.com/snack8310/tiny-url -n server
 cargo generate --git https://github.com/snack8310/tiny-url-web -n web

proxychains4 cargo generate --git https://github.com/snack8310/tiny-url
proxychains4 cargo generate --git https://github.com/snack8310/tiny-url-web

```

```cargo.toml
[workspace]
members = [ 
    "web","server"
]

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

在之前的视频中讲过wasm，是一种协议，我们可以使用trunk build构建WASM脚本文件，通过静态文件加载的方式，放在服务端应用中一并部署，只需要一个服务部署启动即可。我们现在调整一下。

1. 调整wasm dist目录，调整config文件目录，移动到根目录下。
2. 修改服务端应用的启动命令，在启动中，加载WASM的静态资源。
3. 将服务端应用接口统一放在/api路径后。
4. 调整启动入口，从根节点进行启动。

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

我们通过同样的思路，将合并后的项目，写入在一个镜像构建文件中。

```Dockerfile
FROM rust:1.65 as build
ADD .cargo $CARGO_HOME/
RUN rustup target add wasm32-unknown-unknown
RUN cargo install trunk wasm-bindgen-cli

WORKDIR /app
ADD . .

RUN cd web && trunk build --release
RUN cargo build --release

FROM debian:11.5

COPY --from=build /app/target/release/server /app/server
COPY --from=build /app/config /app/config
COPY --from=build /app/dist /app/dist

WORKDIR /app
CMD ["./server"]
```

# 优化构建，缩短构建时间，使用Cargo-chef

使用Cargo-chef，可以利用docker缓存，减少重复的依赖下载，有效的加快镜像重复构建的时间。

https://github.com/LukeMathWalker/cargo-chef

planer  recipe.json
builder
runtime

```Dockerfile
FROM rust:1.65 as chef
ADD .cargo $CARGO_HOME/
RUN cargo install cargo-chef 
# RUN rustup target add wasm32-unknown-unknown
# RUN cargo install trunk wasm-bindgen-cli

WORKDIR /app

FROM chef AS planner
ADD . .
RUN cargo chef prepare  --recipe-path recipe.json

FROM chef AS builder
COPY --from=planner /app/recipe.json recipe.json
RUN cargo chef cook --release --recipe-path recipe.json
RUN rustup target add wasm32-unknown-unknown
RUN cargo install trunk wasm-bindgen-cli

COPY . .
RUN cd web && trunk build --release
RUN cargo build --release

FROM debian:11.5

COPY --from=builder /app/target/release/server /app/server
COPY --from=builder /app/config /app/config
COPY --from=builder /app/dist /app/dist

WORKDIR /app
CMD ["./server"]

```

改变一下代码，重新编一下，building的时间，有明显的减少。

# END
本期我们演示了将WASM前端应用作为静态页面和服务端应用合并部署的方案。
演示如何将WASM应用打包部署到DOCKER容器中，并且使用cargo-chef进行的docker镜像构建时间优化。

项目的代码可以在github上下载。

如果有任何问题和建议，请留言给我。非常感谢大家的观看。