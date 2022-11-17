
# Building a Front Web App in RUST

notes:

嗨，大家好，今天我们做一期使用Rust开发前端web的技术分享。<!-- 周一的下午茶的技术分享视频 -->

项目的服务端代码请参照视频 xxxx

在视频中，我们通过一个简单的短链接创建功能演示一个Wasm项目。

讲解Webassembly，Yew框架，Trunk服务, Tailwind CSS等组件的基本用法。

关键字：Tiny，Rust，WASM， WebAssembly， Yew，Trunk，Tailwind

---

# WebAssembly

WebAssembly，缩写WASM，是一个字节码技术标准。简单的可以想象为类似JAVA，GO编译后生成的可运行的二进制片段。WebAssembly的浏览器支持分别在 // TODO
它本身是个标准不是语言，支持通过多种高级语言，编译出来。提供浏览器执行。
他的出现，是为了解决xxxx。
他有高性能，安全，开放的特点。但相应的也有复杂，高级语言学习曲线等等问题。

本视频所有的内容，也是基于该技术，作为一个全栈工程师，Rust很好的具备了生成WebAssembly的特性。

我们需要先在本机的Rust的编译环境中添加WebAssembly

```
rustup target add wasm32-unknown-unknown
```

---

# Wasm Build Tools

我们需要使用编译工具，讲Rust语言编译成WASM这种标准，常见的工具有，Trunk(https://trunkrs.dev/)，wasm-pack（https://rustwasm.github.io/）

本视频中使用Trunk方式，需要在Cargo组件中添加Trunk工具

在此之前，需要 在 ~/.cargo/config.toml
这个使用全局的配置参数

```
cargo install trunk
```

```index.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tiny URL</title>
</head>
<body>
    
</body>
</html>

```

```
> trunk serve
```

可以看到空白页面，title已经换成TinyURL

---

# Trunk 服务

我们需要对Trunk做一些配置，比如部署目录，调用服务端的API调用进行代理，避免应用中，写具体的远程地址。

Trunk

```Trunk.toml
[build]
target = "index.html"
dist = "dist"

```


# WebAssembly Framework - Yew

常见的框架有很多，例如 Sycamore, Yew, Percy等等。本视频选用Yew作为说明。

Yew的特性有一些： TODO

添加一下爱 Yew框架

```
yew = "0.19"
```

```main.rs

use yew::prelude::*;

#[function_component(App)]
pub fn app() -> Html {

    html! { <h1>{ "Home" }</h1> }
}

pub fn main() {
    yew::start_app::<App>();
}
```

---

#  Yew Route

增加页面跳转路由

```
yew-router = "0.16"
```

```router.rs
#[derive(Clone, Routable, PartialEq)]
pub enum Route {
    #[at("/")]
    Home,
    #[at("/links")]
    Links,
    #[at("/create_link")]
    CreateLink,
}

pub fn switch(route: &Route) -> Html {
    match route {
        Route::Home => html! { <h1>{ "Home" }</h1> },
        Route::Links=> html! {<Links/>},
        Route::CreateLink=> html! {<CreateLink/>},
    }
}

```
---

# Create/Links功能

我们增加对应额 创建短链接的界面，和查询所有生成的短链接地址画面。

我们创建对应的功能页面。

```
pages
  mod.rs
  create.rs
  links.rs
```

```
use yew::prelude::*;

#[function_component(Links)]
pub fn links() -> Html {
    html! { <h1>{ "Links1" }</h1> }
}
```

# Tailwind CSS

在Wasm中也可以使用流行的CSS框架构建样式，这里我选用对程序员比较友好的TailwindCSS框架
https://tailwindcss.com/

参照官网的步骤，安装




input.css 用来生成样式的组件库

output.css 动态生成的样式表

```

npm install -D tailwindcss@latest @tailwindcss/forms  //速度慢，使用下面的镜像源

npm --registry https://npmreg.proxy.ustclug.org/ install -D tailwindcss@latest @tailwindcss/forms 


```

```tailwindcss.config.js
module.exports = {
  content: [
    "./src/**/*.{html,rs}",
    "index.html"
  ],
  theme: {
    extend: {},
  },
  plugins: [
    require('@tailwindcss/forms'),
  ],
}
```

```
npx tailwindcss -i ./styles/input.css -o ./styles/output.css --watch
```

# log
调试方便，我们需要增加 

```
log = "0.4.17"
wasm-logger = "0.2"
```

# links

修改画面，可以使用https://play.tailwindcss.com/先调试画面

```html
        <div class="mx-auto max-w-7xl py-12 sm:px-6 lg:px-8">
            <div class="mx-auto max-w-4xl">
                <div class="overflow-hidden bg-white shadow sm:rounded-lg ">
                    <div class="px-4 py-5 sm:px-6">
                        <h3 class="text-lg font-medium leading-6 text-gray-900">{"短链接列表"}</h3>
                    </div>
                    <div class="border-t border-gray-200">
                        <div class="bg-gray-50 px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
                            <dt class="text-sm font-medium text-gray-500">{"Tiny URL"}</dt>
                            <dd class="mt-1 text-sm text-gray-900 sm:col-span-2 sm:mt-0">{"Origin URL"}</dd>
                        </div>
                        <div class="bg-white px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
                            <dt class="text-sm font-medium text-gray-500">{"code"}</dt>
                            <dd class="mt-1 text-sm text-gray-900 sm:col-span-2 sm:mt-0">{"origin_url"}</dd>
                        </div>
                        <div class="bg-white px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
                            <dt class="text-sm font-medium text-gray-500">{"code"}</dt>
                            <dd class="mt-1 text-sm text-gray-900 sm:col-span-2 sm:mt-0">{"origin_url"}</dd>
                        </div>
                    </div>
                </div>
                <div class="bg-gray-50 px-4 py-3 text-right sm:px-6">
                    <button type="button" class="inline-flex justify-center rounded-md border border-transparent bg-indigo-600 py-2 px-4 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2">{"查看所有短链接"}</button>
                </div>
            </div>
        </div>
```

```
use super::TinyData;

#[derive(PartialEq, Debug, Clone, Properties)]
struct TinyUrls {
    pub tus: Vec<TinyUrl>,
}

impl TinyUrls {
    pub fn init(tus: Vec<TinyUrl>) -> Self {
        TinyUrls { tus }
    }

    pub fn new() -> Self {
        TinyUrls { tus: Vec::new() }
    }
}

#[derive(PartialEq, Debug, Clone, Deserialize)]
struct TinyUrl {
    pub tiny_code: String,
    pub origin_url: String,
}

#[function_component(TinyUrlProps)]
fn tiny_url_props() -> Html {
    const DOMAIN_URL: &str = "http://127.0.0.1:8001/";
    let tiny_urls = use_context::<TinyUrls>().expect("no ctx found");
    log::info!("tiny_urls {}", tiny_urls.clone().tus.len());
    tiny_urls.tus.iter().map(|t|{
        let tiny_url = t.clone();
        html! {
            <div class="bg-white px-4 py-5 sm:grid sm:grid-cols-3 sm:gap-4 sm:px-6">
                <dt class="text-sm font-medium text-gray-500">{format!("{}{}",DOMAIN_URL,tiny_url.tiny_code)}</dt>
                <dd class="mt-1 text-sm text-gray-900 sm:col-span-2 sm:mt-0">{tiny_url.origin_url}</dd>
            </div>
        }
    }).collect()
}
```

```
    let tiny_data = Box::new(use_state(|| None));
    let onclick = {
        let tiny_data = tiny_data.clone();
        Callback::from(move |_| {
            log::info!("Callback");
            let tiny_data = tiny_data.clone();
            wasm_bindgen_futures::spawn_local(async move {
                let tiny_data_endpoint = format!("/api/links",);
                let fetched_tiny_data = Request::get(&tiny_data_endpoint).send().await;
                match fetched_tiny_data {
                    Ok(response) => {
                        let json: Result<TinyData<Vec<TinyUrl>>, _> = response.json().await;
                        match json {
                            Ok(f) => {
                                tiny_data.set(Some(f));
                            }
                            Err(_e) => {}
                        }
                    }
                    Err(_e) => {}
                }
            });

            // let mut v = Vec::new();
            // v.push(TinyUrl{tiny_code:String::from("aaa"), origin_url:String::from("http://baidu.com")});
            // v.push(TinyUrl{tiny_code:String::from("bbb"), origin_url:String::from("http://sina.com")});
            // let td = TinyData{ok:true, data:v,err: None};
            // tiny_data.set(Some(td))
        })
    };

    let mut v = TinyUrls::new();
    match (*tiny_data).as_ref() {
        Some(t) => {
            log::info!("Some");
            v = TinyUrls::init(t.data.clone());
        }
        None => {
            log::info!("None");
        }
    }

```

```
                        <ContextProvider<TinyUrls> context={v.clone()} >
                            <TinyUrlProps />
                        </ContextProvider<TinyUrls>>
```

```
{onclick} 
```

# 访问服务端

```
wasm-bindgen-futures = "0.4.30"
gloo-net = "0.2.4"
```

```
let tiny_data = tiny_data.clone();
            wasm_bindgen_futures::spawn_local(async move {
                let tiny_data_endpoint = format!("/api/links",);
                let fetched_tiny_data = Request::get(&tiny_data_endpoint).send().await;
                match fetched_tiny_data {
                    Ok(response) => {
                        let json: Result<TinyData<Vec<TinyUrl>>, _> = response.json().await;
                        match json {
                            Ok(f) => {
                                tiny_data.set(Some(f));
                            }
                            Err(_e) => {}
                        }
                    }
                    Err(_e) => {}
                }
            });
```

```Trunk.toml

[[proxy]]
rewrite = "/api/"
backend = "http://0.0.0.0:8001/"

```

# create

特别说明的是，需要添加tailwind组件

---
# at the end

notes:

清理掉无效的引用，增加日志，一个短链接创建工具就做完了。

这里强调一下！！！！

请勿直接用于生产环境。一个项目的使用还需要网络管理，权限认证，性能测试等多个安全因素。还有更好的目录结构，静态优化，路径管理等可以持续优化。

本文以一个完整的web项目作为演示。

所有代码保存在
> https://github.com/snack8310/tiny-url

---