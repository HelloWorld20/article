---
title: Rust程序设计语言读书笔记二
date: 2022-11-23 22:45:03
tags: [笔记,Rust]
---

# 使用包、Crate和模块

## crate

crate有两种形式：二进制和库。

二进制项可以被编译为可执行程序，他必须有一个main函数来定义当程序被执行时所需要做的事情。

库没有main函数，它们提供一些注入函数之类的东西。这与其他编程语言中library概念一致。

## 包

包(package)是提供一系列功能的一个或者多个crate。一个包会包含一个Cargo.toml文件，阐述如何构建crate

crate是Rust在编译时最小的代码单位。如果用rustc来编译一个文件，编译器会将那个文件作为一个crate。

包中可以包含至多一个库crate(library crate)。包中可以包含任意多个二进制crate(binary crate)，但是必须至少包含一个crate(无论是库还是二进制)

在cargo项目中，有个约定是，src/main.rc是一个与包同名的二进制crate的根。src/lib.rs，则包带有与其同名的库crate，且src/lib.rs是crate根。

将文件放在 _src/bin_ 目录下，一个包可以拥有多个二进制 crate：每个 _src/bin_ 下的文件都会被编译成一个独立的二进制 crate。

## 模块

可以理解就是一个代码块。让我们将crate的代码分组。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {
            println!("add_to_waitlist")
        }

        fn seat_at_table() {
            println!("seat_at_table")
        }
    }

    mod serving {
        fn take_order() {
            println!("take_order");
            super::super::back_of_house::fix_incorrect_order()
        }
  
        fn serve_order() {
            println!("serve_order")
        }

        fn take_payment() {
            println!("take_payment")
        }
    }


fn serve_order() {}

mod back_of_house {
    pub fn fix_incorrect_order() {
        cook_order();
        super::serve_order()
    }
  
    fn cook_order() {}
}
  
pub fn eat_at_restaurant() {
    crate::front_of_house::hosting::add_to_waitlist();
}

pub fn eat_at_restaurant_again() {
    use crate::front_of_house::hosting;
    hosting::add_to_waitlist();
}

pub use crate::front_of_house::hosting; 
```


内容比较多，可以去看[原文](https://kaisery.github.io/trpl-zh-cn/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html)，不全部展开讲。从上面例子，总结一下关键语法

* mod关键字定义模块，可以定义很多类型，包括函数、方法、结构体、枚举
* 默认情况下模块是私有的，pub关键字可以把模块变为共有，只需在mod前加上pub。
* 结构体模块的每一层级都需要加上pub才能被访问。pub枚举则可以访问所有元素
* 绝对路径从src/lib.rs开始，以crate关键字开始。如`crate::front_of_house::hosting;`
* 相对路径则从兄弟模块开始。
* 父级模块用super代表，可以多层嵌套如`super::super::back_of_house::fix_incorrect_order()`
* use关键字可以将路径`引入作用域`，约定是引入父级模块。如我们需要使用`crate::front_of_house::hosting::add_to_waitlist()。约定是`use crate::front_of_house::hosting`，然后再`hosting::add_to_waitlist()`。
* 重导出名称`pub use crate::front_of_house::hosting; `
* as关键字提供新名称`use std::io::Result as IoResult;`
* 嵌套路径
```rust
use std::cmp::Ordering; 
use std::io;

// 可改为
use std::{cmp::Ordering, io};
```

## 以文件为模块

原文是真的没看懂。网上搜到一篇[知乎专栏文章](https://zhuanlan.zhihu.com/p/164556350)很好理解。

首先，我们需要显式地在Rust中构建模块树，在文件系统树和模块树之间不存在隐式的转换

要把一个文件添加到模块树中，我们需要用`mod`关键字来将文件声明为一个子模块(submodule)。

```
mod my_module;
```

用`mod`关键字后面跟着模块名，编译器会在`同级目录寻找同名文件my_module.rs或my_module/mod.rs`

首先我们构造一下文件目录结构

```shell
│  config.rs
│  main.rs
│  my_module.rs
├─models
│      mod.rs
│      user_model.rs
└─routes
        health_route.rs
        mod.rs
        user_route.rs
```

那么在main中要引入config,则

```rust
// main.rs

mod config; // 引入同级目录文件config.rs

fn main() {
	config::print_config();
}

// config.rs
pub fn print_config() {
	pinrtln!("config");
}

```

如果不是同级，则需要依赖`mod.rs`文件。如main.rs引入routes/health_route.rs

```rust
// main

mod routes;

fn main(){
	routes::health_route::print_health_route();
}

// src/health/mod.rs
pub mod health_route;

// src/health_route.rs
pub fn print_health_route() {
    println!("health_route");
}
```

如果不是从main.rs引入，如src/models/user_model.rs引入routes/health_route.rs

```rust

pub fn print_sibling() {
	// 从根开始寻找
    crate::routes::health_route::print_health_route();

	// use后，可以直接从health_route开始
    use crate::routes::health_route;    // 引入
    health_route::print_health_route(); 

	// super关键字相对父级寻找
    super::super::routes::health_route::print_health_route();   

}
```

以上三种方式都可以。