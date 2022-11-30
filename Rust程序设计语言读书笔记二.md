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


# 常见集合

Rust标准库中包含了一系列称之为集合(collections)的非常有用的数据结构，这些集合指向的数据是存储在堆上的，这意味着数据的数量不必在编译时就已知，并且还可以随着程序的运行增长或缩小。

常见的三个集合：

-   vector允许我们一个挨着一个地存储一些列数量可变的值
-   字符串(string)是字符的集合。
-   哈希map(hash map)允许我们将值与一个特定的键(key)相关联。

## vector

Vector也称之为Vec\<T\>，它在内存中彼此相邻地排列所有值。Vector只能存储相同类型的值。
使用方法如下

```rust
fn main() {
    let mut v = vec![1,2,3]; // vec!宏创建并初始化
    v.push(5); // 使用push添加元素
    v.push(6);
    v.push(7);
    v.push(8);
    
    let mut v1 = Vec::new(); // 创建空vector
    
    v1.push(5);
    
    let third = v2[2]; // 通过索引语法读取元素

    // 索引语法读取元素返回元素本身，如果超出范围，则会panic
    println!("the third element is {}", third);
    
    // 通过get方法读取元素
		// 通过get方法读取返回的是Option<T>，可以通过match或if let来处理返回值
    match v.get(3) { 
        Some(third) => println!("The third element is {}", third),
        None => println!("There is no third element."),
    }
    
    if let Some(third) = v.get(4) {
        println!("The third element is {}", third);
    } else {
        println!("There is no third element.")
    }

		let v4 = vec![100, 332, 45];
    for i in v { // for in语句遍历vector，不同的是for in 遍历出来的是元素本身，而不是index
        println!("{}", i)
    }

		enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }
    // 利用枚举来存储不同类型的数据
    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blut")),
        SpreadsheetCell::Float(10.12)
    ]
}
```

Rust在编译时就必须准确的知道vector中类型的原因在于它需要知道存储每个元素到底需要多少内存。第二个好处是可以准确的知道这个vector中允许什么类型。如果Rust允许vector存放任意类型，那么当对vector元素执行操作时一个或多个类型的值就有可能会造成错误。使用枚举外加match意味着Rust能在编译时就保证总是会处理所有可能的情况。

**个人不太能理解，保证能处理所有情况可以理解，但是存放枚举能保证每个元素多少内存吗？**

## 字符串

除了前面提过的`String.from`创建字符串，此处又提及一个`to_string`方法可以创建字符串。

原文是：to_string能用于任何实现了Display trait的类型，字符字面值也实现了它。

```rust
let s = String::from("Hello world");
let s1 = "Hello world".to_string();
println!("{},{}", s, s1); // 两者完全一样
```

### 更新字符串

push_str：给字符串后面追加字符串
push：给字符串后面追加字符
+：会使得前面的值发生移动，后面需要传引用
format!：最方便的字符串合并宏
```rust
   let mut s = String::from("hello"); // s必须是mut才能追加
    s.push_str(" world");
    s.push('!');
    println!("{}", s); // hello world!

    let s1 = String::from("hello");
    let s2 = String::from(" world");
    let s3 = s1 + &s2;  // 此处是s1发生了移动，移动到s3；s1无法再使用。
    // 至于s2为啥要加&，官方是说签名如此。。
    println!("{}", s3); // hello world
  
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");
  
    let s = format!("{}-{}-{}", s1, s2, s3);    // 更方便的format!
    println!("{}", s); // tic-tac-toe
```

### 索引字符串

`Rust是禁止索引字符串的`

原因是Rust使用`utf-8`来存储字符串，字符是变长的。换句话说，就是一个字符可能会占用多个空格，如果用索引访问，不一定能得到期望的字符。所以Rust干脆禁止了这种不确定的行为。

如果需要做类似操作，可以用字符串切片来实现

```rust
let hello = "Здравствуйте";
let s = &hello[0..4];
```

但这仍然是不可靠的，如果上面的切片是`[0..5]`，那么就会触发panic!

### 遍历字符串

```rust
let hello = "Здр";
for c in hello.chars() {
	println!("{}", c); // З,д,р
}
for c in hello.bytes() {
	println!("{}", c);  //208,151,208,180,209,128
}
```


## Hash Map

Hash Map就和JavaScript里的对象很像了。

```rust
use std::collections::HashMap; // hash map没有被prelude自动引用，需要手动引用

let field_name = String::from("Favorite color");
    let field_value = String::from("blue");
    let mut map = HashMap::new(); // 需要mut才能insert
    map.insert(field_name, field_value);

    // println!("{},{}", field_name, field_value); // 此处所有权已经移动到map

    let mut scores = HashMap::new();
  
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("blue");
    let score = scores.get(&team_name); // 返回的值是Optino<T>
  
    // hash map的遍历
    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }

    scores.insert(String::from("Yellow"), 25);  // 会覆盖原先的值
  
    println!("{:?}", scores);   // {:?}可以直接打印hash map的值： {"Blue": 10, "Yellow": 25}

    // insert会直接覆盖原值，entry加上or_insert可以不覆盖更新。

    scores.entry(String::from("Yellow")).or_insert(50); // 原来有了yellow，不会覆盖
    scores.entry(String::from("green")).or_insert(50); // 原来没有，插入50

    println!("{:?}", scores); // {"green": 50, "Blue": 10, "Yellow": 25}
```

