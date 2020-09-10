---
title: Rust Error
date: 2020-07-17
categories:
  - programing
markup: mmark
math: true
tags:
    - programing
    - rust
---

### Rust中的错误处理

Rust中没有`Exception`这个概念，而是把错误分为了两种，一种是**可恢复的错误**，一种是**不可恢复的错误**

对于可恢复的错误使用`Result<T,E>`类型来封装，不可恢复的错误类型使用`panic!`来定义；

对于不可恢复的`panic!`错误，Rust有两种处理情况，
一种是清理程序运行时产生的‘垃圾数据’（这种方法需要花费一些时间还会导致打包的应用变大），
一种是什么都不做，让操作系统来处理（这种方法会马上退出，并且打包出来的包会很小）

可以在*Cargo.toml*文件中定义`[profile]`来决定使用哪一种方式：

```rust
[profile.release]
panic = 'abort'
```
有时候`panic!`打印的报错信息并不是那么的清晰明了，不像Java中的打印信息一样；为了能够看到更多的错误栈信息，可以通过设置一个参数来打印错误栈

> run with `RUST_BACKTRACE=1` environment variable to display a backtrace.

对于可恢复的错误类型使用`Result<T,E>`封装的

```rust
#![allow(unused_variables)]
fn main() {
enum Result<T, E> {
    Ok(T),
    Err(E),
}
} 
```

Rust中的错误处理再配合上模式匹配真的还挺好用的，类似与Java中的*try-catch*但是更加强大

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error)
            }
        },
    };
}
```
可以使用类似lamda表达式的东西，可以写的更加简洁

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

`Resulst<T,E>`类型有很多的API类型Java的*Stream*和*Optional*一样；

    Optional.orElseThrow()

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```
    Optional.orElseThrow(()->new Exception("Failed to open hello.txt"))
```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

```rust

#![allow(unused_variables)]
fn main() {
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
}
```

上述的代码可以使用语法糖`?`操作符来简化

```rust
#![allow(unused_variables)]
fn main() {
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
}
```

还可以更简化一点

```rust

#![allow(unused_variables)]
fn main() {
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
}
```

然后更简单

```rust
use std::fs;

fn main() {
    let s = fs::read_to_string("C:\\Users\\Joe\\IdeaProjects\\rust-test\\src\\main.rs").unwrap();
    println!("{}",s)
}
```

`?`操作符只被允许是用来函数中返回值类型是`Result`或者`Option`或者实现了 `std::ops::Try` 的类型。

错误处理看起来挺棒的。

完！