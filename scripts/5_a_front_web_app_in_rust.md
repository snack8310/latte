
# Building a Front Web App in RUST

notes:

嗨，大家好，今天我们做一期使用Rust开发前端 Web App的技术分享。<!-- 周一的下午茶的技术分享视频 -->

在视频中，我们通过一个简单的短链接生成功能，演示一个前端Web
 APP开发。

讲解Wasm（Webassembly技术），Yew框架，Trunk服务, Tailwind CSS等组件的基本用法。

项目依赖的服务端代码请参照
项目的备注，https://github.com/snack8310/tiny-url，或之前的视频内容。

关键字：Tiny，Rust，WASM，WebAssembly，Yew，Trunk，Tailwind

---

# WebAssembly

WASM，WebAssembly的缩写，是一个主流浏览器可以解析的字节码技术标准。本视频中使用Rust生成WASM脚本。

<!--简单的可以想象为类似JAVA，GO编译后生成的可运行的二进制片段。WebAssembly的浏览器支持分别在 // TODO
它本身是个标准不是语言，支持通过多种高级语言，编译出来。提供浏览器执行。
他的出现，是为了解决xxxx。
他有高性能，安全，开放的特点。但相应的也有复杂，高级语言学习曲线等等问题。
-->
<!--本视频所有的内容，也是基于该技术，作为一个全栈工程师，Rust很好的具备了生成WebAssembly的特性。-->

我们需要先在本机的Rust的运行环境中添加WebAssembly支持

```
rustup target add wasm32-unknown-unknown
```

---

# Wasm Build Tools

将Rust语言编译成WASM这种标准，常见的工具有，
Trunk(https://trunkrs.dev/)，
wasm-pack（https://rustwasm.github.io/）

本视频中使用Trunk方式，需要在Cargo组件中添加Trunk工具

```
cargo install trunk
```
新建一个Rust项目 tiny-url-web
添加index.html

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

我们可以对Trunk做一些配置，比如部署目录，调用服务端的API调用进行代理，避免应用中，写具体的远程地址。

Trunk

```Trunk.toml
[build]
target = "index.html"
dist = "dist"

```


# WebAssembly Framework - Yew

常见的WASM Framwork有很多，例如 Sycamore, Yew, Percy等等。本视频选用Yew作为演示。

添加一下 Yew Framwork

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
    #[at("/create")]
    Create,
}

pub fn switch(route: &Route) -> Html {
    match route {
        Route::Home => html! { <h1>{ "Home" }</h1> },
        Route::Links=> html! {<Links/>},
        Route::Create=> html! {<CreateLink/>},
    }
}

```
---

# Create/Links功能

我们增加对应的短链接地址创建和查询画面。

添加pages

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

# log
调试方便，我们需要增加 

```
log = "0.4.17"
wasm-logger = "0.2"
console_error_panic_hook = "0.1.7"
```

```main.rs
    wasm_logger::init(wasm_logger::Config::default());
    console_error_panic_hook::set_once();
```

# Tailwind CSS

在Wasm中也可以使用流行的CSS框架构建画面样式，视频中使用TailwindCSS框架

参照官网的步骤，安装
https://tailwindcss.com/



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

links画面

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

# 组件，Event，远程调用

1. Component
2. Event
3. wasm_bindgen_futures
4. use_state

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

访问服务端

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

# Trunk服务端调用代理

```Trunk.toml

[[proxy]]
rewrite = "/api/"
backend = "http://0.0.0.0:8001/"

```

# create

特别说明的是，需要添加tailwind组件 /form

---

# link_to

```function_component(LinkTo)
if let Err(_) = gloo_utils::window().location().set_href(&f.data) {
                    log::info!("something went wrong");
                }
```

---

# at the end

notes:

这里强调一下！！！！

请勿直接用于生产环境。一个项目的使用还需要网络管理，权限认证，性能测试等多个安全因素。还有更好的目录结构，静态优化，路径管理等可以持续优化。

本视频通过短链接的应用，体验Rust开发的Wasm Web App。

如果有任何想法或建议，请直接留言给我，非常感谢大家的观看。

<!-- 所有代码保存在
> https://github.com/snack8310/tiny-url-web -->


---