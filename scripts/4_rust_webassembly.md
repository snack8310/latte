![[rust-logo.png]]

# Rust和Webassembly，前后端分离的项目改造

notes:

嗨，大家好，今天我们做一期关于使用Rust做Webassembly的相关技术演示。

在视频中，增加一个使用Webassembly技术的短链接的前端项目。
在开发过程中，体验一个Rust开发Webassembly的基本流程

关键字：Tiny, Rust, Webassembly

---

# Webassembly介绍

notes:

---

# yew框架引入
流行的WebAssembly Frameworks有很多
Sycamore
Yew
Percy

# Install WebAssembly target

# Install Trunk
Trunk is a WASM web application bundler for Rust. Trunk uses a simple, optional-config pattern for building & bundling WASM, JS snippets & other assets (images, css, scss) via a source HTML file.


---

# route

配置跳转页面
```
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