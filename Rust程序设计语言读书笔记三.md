---
title: Rust程序设计语言读书笔记三
date: 2022-12-4 22:55:03
tags: [笔记,Rust]
---

# 泛型

泛型与Typescript的泛型语法很相似，对结构体、函数使用泛型几乎一样。

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

```

中规中矩，没啥特别的。特别一点的是方法的枚举

```rust
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

枚举需要在impl关键字与结构体名称后面都声明。原文的说法是：这样Rust就知道`Point`的尖括号中的类型是泛型而不是具体类型。

不太能理解，为什么要重复声明。

也可以为某些类型单独定义泛型类型，例如，可以为Point\<f32\>实例实现方法，而不是为泛型Point的实例。

```rust

impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

let point = Point { x: 4, y: 5 };
let point2 = Point { x: 4.4, y: 5.5 };
point.distance_from_origin(); // 报错：method `distance_from_origin` not found for this struct
point2.distance_from_origin(); // 正常调用
```

结构体中定义的泛型类型参数也不一定总是与结构体方法中使用的泛型是同一类型。

```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

## 泛型代码的性能

Rust并不会因为泛型而增加运行时消耗。Rust会在编译过程中进行泛型代码`单态化(monomorphization)`来保证效率。单态化是一个通过填充编译时使用的具体类型，将通用代码转换为特定代码的过程。

看代码就很好理解：

```rust
let interger = Some(5);
let float = Some(5.0);
```

通过单态化编译后看起来会是如下：

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

**那么多加泛型会不会导致编译后代码变大？后面再试试**


# Trait: 定义共同行为

**trait原意有特征、品质的意思**

一个类型的行为由其可供调用的方法构成。如果可以对不同类型调用相同的方法的话，这些类型就可以共享相同的行为了。

使用`trait`关键字定义一个trait，关键字后跟着的就是trait的名称。trait体中可以定义多个方法：一行一个方法签名且都已分号结尾。

```rust
trait Summary {
    fn summarize(&self) -> String;
}
```

为类型实现trait需要用impl关键字，impl后面跟着trait的名字，接下来for 接着 类型名：

```rust
trait Summary {
    fn summarize(&self) -> String;
}

struct NewsArtical {
    headline: String,
    location: String,
    author: String,
    content: String,
}

impl Summary for NewsArtical {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

struct Tweet {
    username: String,
    content: String,
    reply: bool,
    retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("summarize from trait: {}: {}", self.username, self.content)
    }
}
```

在指定了Summary给Tweet后，在敲fn时，编译器就能自动写出参数类型与返回类型。因为此时实现的方法就应该与trait声明一样了。

在实现方法后，Tweet与NewsArtical的实例就有了一样的summary方法。

```rust
let tweet = Tweet {
	username: String::from("wei"),
	content: String::from("sanya"),
	reply: false,
	retweet: true,
};

let news = NewsArtical {
	headline: String::from("head"),
	location: String::from("shenzhen"),
	author: String::from("wei"),
	content: String::from("chatGPT"),
};

let tweetSummary = tweet.summarize();
let newsSummary = news.summarize();
println!("{}", tweetSummary);
println!("{}", newsSummary);
```

[还有两个点](https://kaisery.github.io/trpl-zh-cn/ch10-02-traits.html#%E4%B8%BA%E7%B1%BB%E5%9E%8B%E5%AE%9E%E7%8E%B0-trait)暂时不太好理解，没写demo

* trait必须和类型一起引入作用域以便使用额外的trait方法
* 如果要给一个类型实现trait，就需要把trait引入到类型定义的作用域中

如果要一个一个实现trait的实现也有点太麻烦，Rust可以写默认trait实现，某个特性类型实现trait时，可以选择保留或重载每个方法的默认行为。

只需在声明trait的方法写出方法逻辑，impl处不写逻辑，一个空的大括号即可。

```rust
trait Summary {
    fn summarize(&self) -> String {
        String::from("Read more...")
    }
}

impl Summary for NewsArtical {} // 使用默认实现，如果这里实现了就是重载了实现
```

如果要重载，则impl后面还是编写trait的实现即可。

trait的默认实现可以调用没有默认实现的同一个trait的其他方法，尽管其他方法没有实现默认实现。
但是，编译器会给提示，必须在具体类型中实现没有默认实现的trait方法。

```rust
trait Summary {
    fn summarize(&self) -> String {
        format!("Read more from {}...", self.summarize_author())
    }
 
    fn summarize_author(&self) -> String;
}
```

其实不管summarize里有没有用到summarize_author，编译器都会要求具体类型实现summarize_author方法的实现。

## trait作为参数

原文定义了一个notify函数，该函数调用其参数item上的summarize方法，该参数是实现了summary trait的`某种`类型。为此可以使用`impl Trait`语法：

```rust
// 期望传递的参数是实现了Summary trait的类型实例。
fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize())
}

notify(tweet);
notify(news);
```

其实参声明是impl加上trait名。语法嘛，背吧。

这种情况我们往往是需要其引用，引用的话是在impl关键字前加`&`

```rust
fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize())
}
notify(&tweet);
notify(&news);
```

再结合泛型，可以这么写：

```rust
// 常用的还是引用。所以只写了引用的方式。如果不是引用，去掉&即可
fn notify<T: Summary>(item: &T) {  
    println!("Breaking news! {}", item.summarize())
}

fn notify<T: Summary>(item1: &T， item2: &T) {  
    println!("Breaking news! {}, {}", item1.summarize(), item2.summarize())
}
```

*原文说，上面这个就是[trait Bound](https://kaisery.github.io/trpl-zh-cn/ch10-02-traits.html#trait-bound-%E8%AF%AD%E6%B3%95)？何必再加个名词....*

如果item参数需要同时实现两个不同的trait：Display和Summary：

```rust
fn notify(item: &(impl Summary + Display)) {
    println!("Breaking news! {}", item.summarize())
}

// 期望传递的参数是实现了Summary trait的类型实例。
fn notify<T: Summary + Display>(item: T) {
    println!("Breaking news! {}", item.summarize())
}
```

trait Bound多了难以阅读，所以又多了一个语法糖where

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {}
```

语法糖如下：
```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```


# 生命周期

原文里：`这里还有一种泛型，我们一直在使用它甚至都没有察觉它的存在，这就是 **生命周期**（_lifetimes_）`，可见，Rust里的生命周期是一种`泛型`。

Rust的生命周期与React、Vue生命周期就完全不是一个概念了。Rust的生命周期是`变量存活的时间`。

```rust
let r;
{
	let x = 5;
	r = &x;
}
println!("r: {}", r);
```

如果编写如上代码，那么Rust编译器会报错：`'x' does not live long enough`

在打印r的时候，x已经被销毁，没有活到需要打印x的时候。

## 函数中的泛型生命周期

先来看原文给的例子：

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);

    println!("The longest string is {}", result);
}

fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

上面的代码编译会报错：missing lifetime specifier

提示文本揭示了返回值需要一个泛型生命周期参数，因为Rust并不知道将要返回的一弄类型是指向x或y。此时我们也不知道，因为可能是x，也可能是y。为了修复这个错误，我们将增加泛型生命周期参数来定义引用间的关系，以便借用检查器可以进行分析。

*声明周期注解描述了多个引用声明周期互相的关系，而不影响其生命周期*

生命周期参数名称必须以撇号（'）开头，也就是单引号的单个引，其名称通常全是小写，类似于泛型名称非常短。`'a`是大多数人默认使用的名称。生命周期参数注解位于引用的`&`之后，并有一个空格来将引用类型与生命周期注解分隔开。

函数签名中的生命周期注解需要声明在函数名和参数列表间的尖括号中。加上了生命周期注解的longgest函数如下：

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

上面的签名，我们想要表达的限制是所有（两个）参数和返回的引用的生命周期是相关的，也就是`这两个参数和返回的引用存活的一样久`。函数会获取两个参数，他们都是与生命周期`'a`存在的一样长的字符串slice，函数会返回一个同样也与生命周期`'a`存在的一样长的字符串slice。它的实际含义是longest函数返回的引用的生命周期与传入该函数的引用的生命周期的较小者一致。这些关系就是我们希望Rust分析代码时所使用的。

当具体的引用被传递给longest函数时，被`'a`所代替的具体声明周期是x的作用域与y的作用域相重叠的那一部分。换一种说法就是泛型生命周期`'a`的具体生命周期等同于x和y的生命周期中较小的哪一个。因为我们用相同的生命周期参数`'a`标注了返回的引用值，所以返回的引用值就能保证在x和y中较短的那个生命周期结束之前保持有效。

```rust
let string1 = String::from("abcd");
let result;
{
	let string2 = String::from("xyz");
	result = longest(string1.as_str(), string2.as_str());
	println!("The longest string is {}", result); // 此处能正常打印
}
// 这里会报错：`string2` does not live long enough
// 因为result的生命周期等于string1、string2较小的string2，所以result不能存活到此处
// 即使真正打印的是string1的引用，即使真正打印的引用能存活到这。
// 但是借用编辑器不能通过。
println!("The longest string is {}", result);
```

## 结构体的生命周期注解

看个例子就好：

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    // 此时变量i，也就是ImportantExcerpt的实例，生命周期不能比first_sentance久。
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```


## 生命周期省略（Lifetime Elision）

在早期，Rust确实所有引用都要写生命周期。但是后来因为模式太固定，所以Rust团队就把这些模式编码进了Rust编译器中。

被编码进Rust引用分析的模式被称为生命周期省略规则（lifetime elision rules），如果编译器考虑了这些规则后生命周期还是模棱两可，那么就会报错。

规则有三条：(未来也许会加，也许会有更多的情况可以省略生命周期)

1. 每一个是引用的参数都有它自己是的生命周期参数。
2. 如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数：
3. 如果方法有多个输入生命周期参数并且其中一个参数是&self或者&mut self，说明是个对象的方法，那么所有输出生命周期参数被赋予self的生命周期。


## 方法定义中的生命周期注解

当为`带有生命周期的结构体实现方法`时，也要写生命周期。必须是在impl关键字后声明，并在结构体名称之后被使用，因为这些生命周期是结构体类型的一部分。

```rust
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

**说实话不太理解，反正就是这种情况肯定要加**


## 静态生命周期

静态生命周期能够存活与整个程序期间，关键字是`'static`，所有的字符串字面值都有静态生命周期

```rust
let s: &'static str = "I have a static lifetime.";
```

