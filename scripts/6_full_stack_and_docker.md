# Rust开发WASM及服务端应用，打包一个容器镜像

notes:
过去分别做了使用RUST开发的，短链接的web应用，和WASM的前端应用。
参照：xxx.com

这次做一下项目的应用层级部署，将开发的完整项目，打包到一个镜像中，可以在共用云环境中直接使用。

优化Rust项目的的容器镜像打包时间

参考AliCloud，AWS的xxx

keywords: Rust, Tiny URL, Docker, WASM, Trunk, Cargo-Chef

---

# 合并前端，后端

增加一个Cargo项目，复制前端，后端代码，这个动作后，会带来什么

增加启动脚本，进行测试

构建一个./dev.sh启动脚本

``` dev.sh



```

# Docker构建，镜像构建优化

1. 优化代码编译 Cargo-Chef
2. 使用最小可执行环境，降低发布镜像大小 From

# 调整前端生成目录，合并一个服务启动（TODO）

webassebly的字节码，可以合并到后端一个服务中，启动一个服务。构建一个前后端内部分层，但合并部署的方案。

