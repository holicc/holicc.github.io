---
title: Rust Packages Management
date: 2020-07-10
categories:
  - programing
markup: mmark
math: true
tags:
    - programing
    - rust
---

## Rust Packages Crates and Modules

每个新出的编程语言都有一套对应的包管理工具，Rust也不列外，Rust使用的是`cargo`作为其包管理工具；

`crate`是一个单一的库的意思;

>A crate is a binary or library. The crate root is a source file that the Rust compiler starts from and makes up the root module of your crate

`package`是一个包的意思，一个包可以包含有多个`crate`;
其中包含一个 *`Cargo.toml`*文件，这个文件相当于maven项目中的pom.xml文件一样的功能；

>A package is one or more crates that provide a set of functionality. A package contains a Cargo.toml file that describes how to build those crates.

模块管理，通过使用*module*来管理函数，是函数能够分到对应的模块中，并再其他地方能够使用他们；有点类似Java的**包**空间；

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

看了上面的例子我发现，Rust中的关键字定义都使用是三个字母....

在Rust中crate分为*binary*和*library*,binary文件就是对应的*main.rs*而library对应的是*lib.rs* 他们都是`crate roots`

引入并使用模块中的函数，使用的是`paths`来定义,可以使用绝对路径和相对路径；

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

`pub`关键字用来提供函数的访问权限，相当与Java中的`public\private\proctect`

`super`关键字类似与相对路径的功能,有啥区别呢？不知道。。。

```rust
fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();
    }

    fn cook_order() {}
}
```

好了下面介绍一下`import`关键字，呸，是`use`

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting::add_to_waitlist;

pub fn eat_at_restaurant() {
    add_to_waitlist();
    add_to_waitlist();
    add_to_waitlist();
}
```

21世纪的编程语言都带有引包别名的功能 `as`

```rust
#![allow(unused_variables)]
fn main() {
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
    Ok(())
}

fn function2() -> IoResult<()> {
    // --snip--
    Ok(())
}
}
```

还可以Re-exporting 通过使用 `pub use`

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```

不是很明白设计了这么多，搞得有点复杂，是不是因为本身问题很多需要引入这么多技术来解决呢？

完！

