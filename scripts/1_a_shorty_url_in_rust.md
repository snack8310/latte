

<style>
.reveal code {
  font-size: 1.5em;
  line-height: 1.5em;
}
</style>

![[rust-logo.png]]

# Building a Tiny Url in RUST

notes:

嗨，大家好，今天我们做一期关于Rust的技术分享。<!-- 周一的下午茶的技术分享视频 -->

在视频中，会从0开始，使用Rust构建一个短链接的web服务，
在开发过程中，体验一个Rust项目的基本的开发流程，使用框架，以及数据库连接等。

关键字：Tiny, Rust, Actix, Sqlx, Mysql, Tera

---


# Rust Installation & Basic Command

notes:

根据官方推荐的方式，采用Shell Script安装，Rustup工具：

https://www.rust-lang.org/learn/get-started

```

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

rustc --version

cargo --version

```

Cargo是一个Rust的构建和包管理工具，与Golang的mod有些相似。

- build your project with <b>cargo build</b>
- run your project with <b>cargo run</b>
- test your project with <b>cargo test</b>
- build documentation for your project with <b>cargo doc</b>
- publish a library to crates.io with <b>cargo publish</b>

---

# Create a Tiny Url Project

notes:

- 通过Cargo命令行创建一个Rust项目
- 生成包结构。
- 运行Hello world。

```

> cargo new tiny-url
     Created binary (application) `tiny-url` package

> cd tiny-url 

> tree
.
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
└── target
    └── *

> cargo run 
   Compiling tiny-url v0.1.0 (/Users/huisheng/Documents/Workspace/Github/tiny-url)
    Finished dev [unoptimized + debuginfo] target(s) in 2.07s
     Running `target/debug/tiny-url`
Hello, world!

```
---

# Show Project in VSCODE

notes: 

- VSCODE 建议的插件：
  - rust-analyzer 官方的，用来替代rust-lang.rust.
  - Rust Syntax 高亮语法
- 个人选用的插件：提供视频中的效果
  - Prettier - Code formatter 代码格式化
  - One Dark Pro VSCODE黑色风格
  - Error Lens 实时纠错
  - TabNine AI Autocomplete 自动代码补全

```
> code .
```
---

# Install the framework: Actix

notes: 

> 详细的第三方工具可以参照下面的中文介绍：
> https://course.rs/practice/third-party-libs.html
> 比较流行的框架有Rocket，Actix-web，axum等等都是可以的。
> 选择Actix-web版本，是因为后面与要选的 Sqlx 连接数据库的插件，有更合适的运行时环境。
> 如果使用 Diesel等连接数据库的方案，Rocket和Actix都不错。

参照 https://actix.rs/ 开始使用Actix

Cargo.toml文件中，增加引入依赖

```
[dependencies]
actix-web = "4"
```

参照官方案例，复制两段代码到 main.rs

```
use actix_web::{get, post, web, App, HttpResponse, HttpServer, Responder};

#[get("/")]
async fn hello() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}

#[post("/echo")]
async fn echo(req_body: String) -> impl Responder {
    HttpResponse::Ok().body(req_body)
}

async fn manual_hello() -> impl Responder {
    HttpResponse::Ok().body("Hey there!")
}
```

```
#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(hello)
            .service(echo)
            .route("/hey", web::get().to(manual_hello))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await
}
```

然后执行 Cargo build / Cargo run

```

> cargo run
    Blocking waiting for file lock on build directory
    Finished dev [unoptimized + debuginfo] target(s) in 23.60s
     Running `target/debug/tiny-url`

```

打开Terminal

```
> curl 127.0.0.1:8080
Hello world!%                           
```

---

# replace the soucre of crates-io 

notes:

Cargo build或者Cargo run的时候，可能会存在下载速度比较慢的情况，这是因为连接的crates仓库速度慢，crates仓库类似Java里的maven仓库，是一个官方的crate仓库地址，默认的地址会指向 

> https://crates.io/ 

我们也可以通过这个地址文档查询我们使用的组件。

解决办法是替换国内的镜像源，这里推荐两个：
- mirrors.ustc.edu.cn 中国科学技术大学爱好者维护的，比较有高校特色。
- https://rsproxy.cn 字节跳动维护。

解决方案可以参照

> https://doc.rust-lang.org/cargo/reference/config.html

个人比较推荐的方案是伴随着项目

> /tiny-url/.cargo/config.toml

文件内容如下：

```
[source.crates-io]
# replace-with = 'ustc'
replace-with = 'rsproxy'

[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
[registries.ustc]
registry = "https://github.com/rust-lang/crates.io-index"


[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"
[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"

[net]
git-fetch-with-cli = true
```

---


# 增加短链接的API

notes:

我们添加三个短链接API接口，
1. 根据原始URL生成短链接Code
2. 根据生成的短链接地址，跳转原始URL
3. 查询生成的短链接地址

```
api
links.rs
mod.rs

#[derive(Deserialize, Clone, Debug, Serialize)]
struct Link {
    tiny_code: String,
    origin_url: String,
}

#[derive(Deserialize, Clone)]
struct ApiAddLink {
    origin_url: String,
}

impl ApiAddLink {
    fn to_new_link(self) -> Link {
        Link {
            tiny_code: nanoid!(5),
            origin_url: self.origin_url,
        }
    }
}

#[post("/create")]
async fn create_link(link: Json<ApiAddLink>) -> impl Responder {
    info!("create_link");
    let new_link = link.0.to_new_link();
    let new_code = new_link.tiny_code.clone();
    // Json(ApiResult::success(Some(new_code)))
    Json(new_code)
}

#[get("/{code}")]
async fn get_from_link(path: Path<String>) -> impl Responder {
    info!("get_from_link");
    let url = "http://www.baidu.com";
    HttpResponse::Found()
    .append_header((header::LOCATION, url))
    .finish()
}

#[get("/links")]
async fn get_all_links() -> impl Responder {
    info!("links");
    let mut links = Vec::new();
    links.push(Link{tiny_code:String::from("111"), origin_url:String::from("http://www.baidu.com")});
    links.push(Link{tiny_code:String::from("123"), origin_url:String::from("http://www.google.com")});
    // Json(ApiResult::success(Some(links)))
    Json(links)
}

```

# a struct for api result 

notes:

写一个通用的结构体，提供标准化的数据返回结构，包括返回结果是否成功，成功时的数据，以及不成功时候的错误信息

数据和错误均采用Option包装，Option和Result是Rust里的重要结构，这个思想在很多语言中都有，后面会考虑单独讲解。

错误对象采用泛型，支持序列化的trait的泛型结构即可

api_result.rs

```
use serde::Serialize;

#[derive(Serialize)]
pub struct ApiResult<T: Serialize> {
    pub ok: bool,
    pub err: Option<String>,
    pub data: Option<T>,
}

impl<T: Serialize> ApiResult<T> {
    pub fn success(r: Option<T>) -> ApiResult<T> {
        ApiResult {
            ok: true,
            err: None,
            data: r,
        }
    }
    pub fn error<E: ToString>(err: E) -> ApiResult<T> {
        ApiResult {
            ok: false,
            err: Some(err.to_string()),
            data: None,
        }
    }
}
```

main.rs 中引用

```
mod api_result;
```

---

# settings in config file

notes:

到现在为止，完成了一个连接mysql的web服务器的基本功能

现在开始，完成短链接的基本功能，并调整为一个应用程序的常见结构。

增加服务的环境配置，增加数据库的链接。一般来说，一个项目的开发环境和生产环境是不同的配置，为了更安全和环境隔离，这样的配置是不会在硬编码，并且可以文件覆盖。通常会通过配置文件，或者环境变量提供差异化的配置。

我们在src的同级别增加一个config文件夹，用来提供生产环境的路径替换。里面增加一个Settings.toml文件

Settings.toml

```
[server]
ip = "0.0.0.0"
port = 8000

[database]
url = "mysql://admin:sdfcerts4amc@tiny.cgrxfrwrkl7o.us-east-1.rds.amazonaws.com/tiny"
pool=5

```

使用一个解析config的crate，帮助我们解析toml的配置文件，

```
[dependencies]
config = "0.13.1"
serde = "1.0.145"
serde_json = { version = "1.0.2", optional = true }
```

新建一个settings.rs文件，用来解析配置文件。在配置文件中，我们使用两个结构体来定义配置结构，使用Config解析。

serde是Rust里著名的一个序列化反序列化的crate，这里使用解析文件的值到结构体中

- server，用来标记服务相关的配置，
  - ip
  - port
- database 用来标记数据库相关的配置
  - url
  - pool_size

setting.rs

```
use config::{Config, ConfigError};
use serde::Deserialize;

#[derive(Debug, Deserialize)]
pub struct Settings {
    pub server: Server,
    pub database: Database,
}

#[derive(Debug, Deserialize)]
pub struct Server {
    pub port: u32,
    pub ip: String,
}

impl Server {
    pub fn get_ip(&self) -> String {
        format!("{}:{}", self.ip, self.port)
    }
}

#[derive(Debug, Deserialize)]
pub struct Database {
    pub url: String,
    pub pool_size: u32,
}

impl Settings {
    pub fn new() -> Result<Self, ConfigError> {
        const CURRENT_DIR: &str = "./config/Settings.toml";

        let s = Config::builder()
            .add_source(config::File::with_name(CURRENT_DIR))
            .build()?;

        s.try_deserialize()
    }
}

```

想要使用配置文件，需要在main函数中引入mod

main.rs

```
mod settings;

async fn main() -> Result<(), sqlx::Error> {
    let s = Settings::new().unwrap();
    let ip = s.server.get_ip();
    let url = s.database.url;

    //修改数据库和服务器的配置

}

```

---

## create tiny code for original url

notes:

step 1: 数据库中创建一个短链接存储表，可参照DDL

```
CREATE TABLE IF NOT EXISTS `tiny_link` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `origin_url` varchar(1024) NOT NULL COMMENT '原始链接',
  `tiny_code` varchar(10) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tiny_code` (`tiny_code`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='短链接'

```

---

# Add SQLX to Connect Mysql Server

notes:

> SQLX是一个纯Rust构建的异步的SQL的crate。
> SQLX不是一个ORM，使用SQLX的ORM，推荐SeaORM。

[dependencies]增加

```
sqlx = { version = "0.6", features = [ "runtime-actix-native-tls" , "mysql" ] }
```

main方法中增加

```

    let pool = MySqlPoolOptions::new()
        .max_connections(5)
        .connect("mysql://user:password@address/schema").await?;

    let row: (i64,) = sqlx::query_as("SELECT ?")
        .bind(150_i64)
        .fetch_one(&pool).await?;

    let ret = row.0;
    println!("row is {ret}");

```


<!-- url = "mysql://admin:sdfcerts4amc@tiny.cgrxfrwrkl7o.us-east-1.rds.amazonaws.com/tiny" -->

- 这时候显示错误，这是因为连接数据库连接池和查询结果的异步动作await返回的错误类型，与 actix的启动的异步动作await的类型不同。我们需要修改一下返回值类型。
- 修改actix启动，包装错误，并在结尾返回Result类型

```

async fn main() -> Result<(), sqlx::Error> {

    // 这里连接数据库的代码 
    // ...

    // 这里是启动服务的代码
    HttpServer::new(|| {
        App::new()
            .service(hello)
            .service(echo)
            .route("/hey", web::get().to(manual_hello))
    })
    .bind(("127.0.0.1", 8080))?
    .run()
    .await?;

    Ok(())
}

```

完成Mysql的连接

```
> cargo run
   Compiling tiny-url v0.1.0 (/Users/huisheng/Documents/Workspace/Github/tiny-url)
    Finished dev [unoptimized + debuginfo] target(s) in 5.75s
     Running `target/debug/tiny-url`
row is 150
```

---

<!-- # vs Diesel

notes:

- Diesel 作为ORM使用体验非常好
- 依赖gcc，需要在宿主机上安装连接器，比如Mysql的场合，需要libmysqlclient for the Mysql backend. 参照如下的说明。

> https://diesel.rs/guides/getting-started

- 构建镜像的场合，可以通过打包连接器或者安装连接器组件

--- -->

# load database pool while server is running

notes:

参照 actix官方文档

> https://actix.rs/docs/databases/


修改启动配置，在Server启动时，加载数据库连接池，在提供服务的时候，可以引用构建的database pool

```
#[actix_web::main]
async fn main() -> Result<(), sqlx::Error> {
    // 一些配置参数
    // ...
    let pool = MySqlPoolOptions::new()
        .max_connections(s.database.pool_size)
        .connect(&s.database.url)
        .await?;
    HttpServer::new(move || {
        App::new()
            .app_data(Data::new(pool.clone())) // 增加数据连接池引用
            .service(hello)
            .service(echo)
            .route("/hey", web::get().to(manual_hello))
    })
    .bind(&ip)?
    .run()
    .await?;

    Ok(())
}
```

现在我们修改一个web请求


```
#[get("/")]
async fn hello(pool: web::Data<MySqlPool>) -> impl Responder {
    // HttpResponse::Ok().body("Hello world!")

    let num = test_db_connect(pool).await;
    let num = match num {
        Ok(x) => x,
        Err(e) => {
            print!("{}", e);
            return HttpResponse::Ok().body("Something went wrong");
        }
    };

    HttpResponse::Ok().body(num.to_string())
}

async fn test_db_connect(data: Data<MySqlPool>) -> Result<i64, sqlx::Error> {
    let pool = data.as_ref();
    let row: (i64,) = sqlx::query_as("SELECT ?")
        .bind(150_i64)
        .fetch_one(pool)
        .await?;
    Ok(row.0)Í
}
```

运行看一下效果

```
> curl 127.0.0.1:8000
150%                                  
```
---

# tiny api

notes:

- create tiny code for original url
- redirect to original url

## create tiny code for original url

notes:

step 1: 数据库中创建一个短链接存储表，可参照DDL

```
CREATE TABLE IF NOT EXISTS `tiny_link` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `origin_url` varchar(1024) NOT NULL COMMENT '原始链接',
  `tiny_code` varchar(10) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tiny_code` (`tiny_code`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='短链接'

```

step 2: 根据url创建短链接code

创建Post结构体ApiAddLink，这里使用Json结构，增加数据库存储结构，NewLink，

如果使用ORM工具，通常会有另外的数据库类型实体映射，这里不进行扩展。

在从接口类型到存储类型转换中，提供NanoId生成，这里根据业务需要调整生成code长度

新建api.rs

使用NanoId来创建短链接，具体NanoId的说明，参照

> https://crates.io/crates/nanoid

```
[dependencies]
nanoid = "0.4.0"
```

```
use nanoid::nanoid;

#[derive(Deserialize, Clone)]
struct NewLink {
    tiny_code: String,
    original_url: String,
}
#[derive(Deserialize, Clone)]
struct ApiAddLink {
    original_url: String,
}

impl ApiAddLink {
    fn to_new_link(self) -> NewLink {
        NewLink {
            original_url: self.original_url,
            tiny_code: nanoid!(5),  // generate tiny code 
        }
    }
}

```

step 3: 创建短链接的Handler

```
#[post("/create")]
async fn create_link(link: Json<ApiAddLink>, data: Data<MySqlPool>) -> impl Responder {
    let new_link = link.0.to_new_link();
    let new_code = new_link.tiny_code.clone();
    if let Err(e) = insert_into_tiny_link(data, new_link).await {
        return Json(ApiResult::error(e.to_string()));
    }
    Json(ApiResult::success(Some(new_code)))
}

async fn insert_into_tiny_link(
    data: Data<MySqlPool>,
    new_link: NewLink,
) -> Result<u64, sqlx::Error> {
    let insert_id = sqlx::query(
#         r#"insert into tiny_link (tiny_code,origin_url) values (?, ?)"#,
    )
    .bind(new_link.tiny_code)
    .bind(new_link.original_url)
    .execute(data.get_ref())
    .await?
    .last_insert_id();
    Ok(insert_id)
}

```

记得在main方法中，增加service引用

```
async fn main() -> Result<(), sqlx::Error> {
    // 省略...
    HttpServer::new(move || {
        App::new()
            .app_data(Data::new(pool.clone()))
            .service(api::create_link) // 增加handler注册
            .service(hello)
            .service(echo)
            .route("/hey", web::get().to(manual_hello))
    })
    .bind(&ip)?
    .run()
    .await?;

    // 省略...
}

```

step 4: Cargo Run 验证结果

```
> curl --request POST 'http://127.0.0.1:8000/create' \
--header 'Content-Type: application/json' \
--data-raw '{
    "origin_url": "http://www.baidu.com"
}'
{"ok":true,"err":null,"data":"weBUD"}%        
```
---

## redirect to original url

notes:

根据刚才创建的结果，跳转到原始的URL，我们可以采用 domain/<tiny_code> 的结构

api.rs

```
#[get("/{code}")]
async fn get_from_link(path: Path<String>, data: Data<MySqlPool>) -> impl Responder {
    let code = path.into_inner();
    let url = get_original_url(data, code).await;
    let url = match url {
        Ok(x) => x,
        Err(e) => {
            print!("{}", e);
            return HttpResponse::NotFound().finish();
        }
    };
    HttpResponse::Found()
        .append_header((header::LOCATION, url))
        .finish()
}

async fn get_original_url(data: Data<MySqlPool>, code: String) -> Result<String, sqlx::Error> {
    let row: (String,) = sqlx::query_as("SELECT origin_url from tiny_link where tiny_code = ?")
        .bind(code)
        .fetch_one(data.get_ref())
        .await?;
    Ok(row.0)
}
```

main方法中，增加service引用

```
async fn main() -> Result<(), sqlx::Error> {
    // 省略...
    HttpServer::new(move || {
        App::new()
            .app_data(Data::new(pool.clone()))
            .service(api::create_link) 
            .service(api::get_from_link) // 增加handler注册
            .service(hello)
            .service(echo)
            .route("/hey", web::get().to(manual_hello))
    })
    .bind(&ip)?
    .run()
    .await?;

    // 省略...
}
```

Cargo Run 验证结果

```
> curl --request GET 'http://127.0.0.1:8000/weBUD'   
> 
```
这个没有任何返回结果，我们需要在浏览器中，看到页面已经跳转到baidu.com

---
# template

notes:

rust也有比较简单的web端的实现，支撑全栈工程师们的需求。

这里推荐使用tera template， 纯Rust开发的一个web 

> https://tera.netlify.app/

这里做个简单的欢迎页面。

Cargo.toml 引入crate

```
tera = { version = "1", default-features = false }
lazy_static = "1.4.0"
```

main.rs增加模板管理

```
#[macro_use]
extern crate lazy_static;

lazy_static! {
    pub static ref TEMPLATES: Tera = {
        let tera = match Tera::new("templates/**/*") { //模板解析地址
            Ok(t) => t,
            Err(e) => {
                println!("Parsing error(s): {}", e);
                ::std::process::exit(1);
            }
        };
        tera
    };
}
```

增加templates目录，添加index.html，注意与模板初始化的地址保持一致。

```
.
├── templates
│   └── index.html
├── Cargo.toml
├── src
│   └── main.rs
│   └── *
└── target
    └── *
```

index 内容

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>Tiny-URL</title>
    </head>
    <body>
       <h1>{{content}}</h1>
    </body>
</html>

```

修改响应的Response，这里是Index

```
#[get("/")]
async fn index() -> impl Responder {
    let content = "Generate a tiny code for url";
    // HttpResponse::Ok().body(content)

    let mut data = Context::new();
    data.insert("content", content); // 模板文件中的变量

    let rendered = TEMPLATES.render("index.html", &data).unwrap();
    HttpResponse::Ok().body(rendered)
}

```

现在启动后，已经看到使用模板修改后的页面。看起来与Java中的JSP有些相似。如果有更高的端的需求，比如性能，样式，模板等，也有很多解决方案。

清理掉无效的handler和调试中的注释，现在一个短链接创建工具就做完了。

---
# at the end

notes:

强调！！！！

请勿直接用于生产环境。一个项目的使用还需要网络管理，权限认证，性能测试等多个安全因素。还有更好的目录结构，静态优化，路径管理等可以持续优化。

本文以一个完整的web项目作为演示。

所有代码保存在
> https://github.com/snack8310/tiny-url

---