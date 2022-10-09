<style>
.reveal code {
  font-size: 1.5em;
  line-height: 1.5em;
}
</style>

![[rust-logo.png]]

# 怎么去理解Rust的价值，这有什么不同

notes:

嗨，大家好，这里是新的一期技术分享。<!-- 周一的下午茶的技术分享视频 -->

今天是一期个人对于Rust语言的认知的分享。解决一些刚刚接触该语言的开发者的困惑。

部分的演示用例，是在

> https://fasterthanli.me/articles/a-half-hour-to-learn-rust

网站上获取的。也推荐大家在通过该网站对Rust有一个初步的了解

---
<!-- 
# Rust是更好的 C or C++ 么？

notes:

我不觉得Rust是个更好或更坏的 [C/C++]++。

C语言很快，Rust也很快。
C语言有指针，Rust有引用。

Rust里面存在引用，智能指针等等概念，被很多人认为与C或者C++很像，经常会放在一起比较。

指针或者引用的概念，更多的是关注于堆栈的数据操作。从rust的生命周期理解，我个人会觉得Rust的指针的目的，更多的是显示的管理数据的堆栈上的所有权和生命周期。从而可以在编译过程中，明确地判断数据的完整生命周期。

而在Java/Python一类的语言中，堆栈的操作被弱化了，是由GC帮忙管理的。而指针引用又是Rust的第一类公民，确实很好用。对比来说，会跟C/C++更为相似。

--- -->

# Rust是一个不同的编程模式

notes:

Rust是什么？

我不打算通过一些奇技淫巧的语法糖，来体现Rust作为一个高级语言的特色。

Rust是个不同的编程模式，

Rust是一切依赖静态检查，保障运行安全的，及高性能运行的，编程语言。

“不是一匹更快的马，是汽车”

这可能是一个下一代编程语言的理念。

通过Rust的官网上介绍的Rust的一些特性，我们解读一下具体的不同

---

# 性能（Performance）

notes:

Rust速度很快

首先运行速度块，消耗资源低，是Rust的一个令人称道的基本特性。

但是现在的高级编程语言来说，最快的仍然是C或者C++，这里有一个数据参考：

> https://github.com/kostya/benchmarks

以C的速度为标准，可以错略理解，GO是个2倍慢的速度，Java是3到4倍慢，Python，嗯，它的优势不在这里。

Rust，大概在1.1+。很不错，这是怎么做到的？

---

## 没有 Runtime（No Runtime）

notes：

> Requiring a runtime limits the utility of the language, and makes it undeserving of the title "systems language". All Rust code should need to run is a stack.

Rust的设计理念中，就是要在系统堆栈上运行应用，而不是一个兼容的运行时环境中。一个兼容的运行时环境必然要牺牲系统特性。可以理解一个IOS原生的应用和H5的应用的区别。

在Rust编译过程中，会依赖操作系统，打包必要的系统类库，或者Rust编写的可以与操作系统通信的类库。最终生成二进制文件，机器码。在运行的时候会被系统视为一个进程。

> https://www.theregister.com/2022/06/23/linus_torvalds_rust_linux_kernel/

Rust可能很快进入Linux内核，这也侧面证明了Rust的语言特性。

---

## 没有GC（NO Garbage Collector）

notes:

> garbage collection is frequently a source of non-deterministic behavior.

高级开发语言中，通常分成两类，

C/C++, 没有GC，但他们不是内存安全的，解决内存泄露，是每个高级开发的必修课。

Java/Go/Python/Ruby，等等，他们用了另外的方案。使用GC帮助管理内存安全。

虽然他们使用的方式有所不同，计数器或者根节点等等，但无一例外的，GC管理会带来其他的问题：
- 无法预测回收时间，不确定什么时间GC会启动。
- 更致命的是GC发生时，系统停顿。1ms的GC停顿，在现代语言中是很常见的。但在越来越高要求的产品标准时，可能会越来越难以接受。
- 无法GC的平台。--为什么没有用JAVA开发的迁入时车载系统。

Rust是少数两者兼顾的开发语言。

---

# 可靠性（Reliability）

notes:

Rust最迷人的特性。

没有Runtime，没有Gc的情况下，做到内存安全，避免系统崩溃。

编译时检查。

---

## 静态&强类型语言（statically typed & strong typed language）

notes:

静态语言和强类型，并不是个高级特性，在很多语言中也存在，比如Java， 从网上找了一个图：

![[typed language.png]]

<!-- 我把Rust放在右上角。强类型，静态编译语言，看起来，离Java更近了一点。 -->

虽然Rust在开发中，不需要显示指定一些变量的类型，例如

```
let name = "foo";
let value = 5;
```

要理解的是，在编译过程中，编译器会自动识别name为&str，value为i32类型的。

Rust有个自动推导的功能，用来帮助编译器解析变量的类型。

典型的错误

```
    let name ;

    if true{
        name = 5;
    }else{
        name = "aaa";
    }
```

```
> cargo build
2 |     let name ;
  |         ---- expected due to the type of this binding
...
7 |         name = "aaa";
  |                ^^^^^ expected integer, found `&str`
```

---

## 数据所有权（data ownership）

notes:

数据所有权的概念，是Rust的核心理念。

```
struct Number {
    odd: bool,
    value: i32,
}

fn main() {
    let n = Number { odd: true, value: 51 };
    let m = n; // `n` is moved into `m`
    let o = n; // error: use of moved value: `n`
}
```
所有权转移的容易理解的用例。

```
struct Person {
    name: String,
}

fn main() {
    let name = format!("fasterthan{}", "lime");
    let p = Person { name: name };
    // `name` was moved into `p`, their lifetimes are no longer tied.
    println!("{}", name); // error: value borrowed here after move
}

```

一个结构体中的引用的实际上是变量的value。name数据的所有权已经转移到新的结构体中，再次结佣name的value的场合，就会显示错误。

这是一种提高引用传递速度的聪明方法，同时避开了类型错误，内存管理问题。

---

## 数据生命周期（data has lifetime）

notes:

> https://doc.rust-lang.org/rust-by-example/scope/lifetime.html
>
> Specifically, a variable's lifetime begins when it is created and ends when it is destroyed. 

```
fn main() {
    // `x` doesn't exist yet
    {
        let x = 42; // `x` starts existing
        println!("x = {}", x);
        // `x` stops existing
    }
    // `x` no longer exists
}
```

这是个容易理解生命周期的例子，这表明了变量创建以及销毁的时机。

```
fn main() {
    let x_ref = {
        let x = 42;
        &x
    };
    println!("x_ref = {}", x_ref);
    // error: `x` does not live long enough
}

```
这是个对初学者理解不友好的例子，也是Rust的静态编译检查的强大之处。

Rust在编译时，识别到=右侧的生命周期在复制时，已经完结并被销毁了。这时候的引用会出现错误。

解释后，是否觉得有一些道理。这里我不去评判也没有资格评判这个设计，是否有更友好的实现机制，但这一定显示的避免了可能的生命周期失效的问题，保证了变量在失效后立即销毁，避免长期存在的内存风险。

---

## ADTs(Algebraic Data Type)

notes:

好吧，我承认这个词是我抄的，花了很多时间去理解这个概念。

TODO 深度理解后，再补充这个

---

## 宏（macros）

notes:

C, C++ 还有 Go，很多语言中都有宏的存在。大部分的宏的使用是从C引申来的，Go，Python，Ruby等。

而C语言体系的宏，总是让人谈“宏”色变。

Rust的宏的理念是从Lisp语言的，与C的设计是完全不同的。

可以把Rust宏，理解为一些通用代码生成工具，他的生效时间是在编译中，编译器把宏解析为应用代码，替代重复的开发动作。

似乎可以解释我们不需要对它那么恐惧。

---

## unsafe

notes:

---

# 生产力（Productivity）

notes:

虽然官方文档中把生产力工具放在了重要的位置上，例如Cargo的包管理工具，例如像Go语言一样，包装了Test语法，甚至有代码格式化的定义。但我认为这些完全不能媲美性能和可靠性章节的重要程度。

但是，真的，当看到编译器给十分此清晰的错误提示的时候，还是会不由得内心一颤：过去的调试时的困难一幕幕呈现。

---

## Cargo Doc

## Cargo Check

# 结论

notes:

由于本人知识有限，还有很多特性没有展示出来，比如一些零开销抽象等等，

Rust的所有语言特性，你会发现均体现在编译环节中

---
