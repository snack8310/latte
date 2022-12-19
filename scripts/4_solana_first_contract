# SOLANA公链介绍

特色：速度快，计算能力强。使用RUST语言开发。
流行的产品：XXX，XXX

# 解决国内屏蔽问题
1. 开一个ALIYUN服务器，或者使用GITPOD
2. 本地测试环境
3. Solana Playground

前期建议在devnet上，对于理解有帮助

# SOLANA的HELLOWORLD
做一个简单的调用

1. solana链上部署一个合约（account）
   1. entrypoint
   2. program id
2. 写一个客户端的调用
   1. 方案typescript，Npm，Yarn，Node.js
   2. 纯Rust-Client
3. 关键字
   1. transaction
   2. instruction

> https://github.com/solana-labs/example-helloworld

# build
cargo build-bpf --manifest-path=./programs/hello/Cargo.toml --bpf-out-dir=dist/programs

# keypair.json
这个生成的json文件，就是当项目部署到solana的节点上，所使用的地址。
这个是个私钥，可以运算成公钥，也就是链上的地址。私钥是当需要在节点上做一些交易时使用，后面的视频会介绍到。

# deploy
solana program deploy /Users/huisheng/Documents/Workspace/my_videos/solana-hello/dist/programs/hello.so

# 交互（interact）

## JS
需要安装 Node，Npm

```javascript
yarn add @solana/web3.js

```