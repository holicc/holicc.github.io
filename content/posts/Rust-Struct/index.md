---
title: Rust Struct
date: 2020-07-08
categories:
  - programing
markup: mmark
math: true
tags:
    - programing
    - rust
---

通过使用关键`struct`可以定义一个类，呸，是定义一个结构体。
```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
}
```

`struct`与Java中的类是不同的两种数据结构，具体的使用方式也有所不同。

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

有一个算是语法糖的功能就是,结构体复用;使用`..`符号能够复用一个已经创建好的结构中的字段;
```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };

    let user2 = User {
        email: String::from("another@example.com"),
        username: String::from("anotherusername567"),
        ..user1
    };
}
```

也可以定义一种类似Java中`Record`的数据结构

```rust
fn main() {
    struct Color(i32, i32, i32);
    struct Point(i32, i32, i32);

    let black = Color(0, 0, 0);     
    let origin = Point(0, 0, 0);
}
```           
看起来虽然跟Golang没什么太大的区别,但是加上了所有权之后就有了一些限制;

```rust
struct User {
    username: &str,
    email: &str,
    sign_in_count: u64,
    active: bool,
}

fn main() {
    let user1 = User {
        email: "someone@example.com",
        username: "someusername123",
        active: true,
        sign_in_count: 1,
    };
}     
```  
这是Rust团队深思熟虑过后的设计,为了能够让struct的实例能够拥有成员变量的所有权

>In the User struct definition in Listing 5-1, we used the owned String type rather than the &str string slice type. This is a deliberate choice because we want instances of this struct to own all of its data and for that data to be valid for as long as the entire struct is valid.

不懂的是为什么这里username使用了`&str`就没有了username成员变量的所有权?

难道是因为`&str`是一个`slice`类型? slice就是一中没有所有权的数据类型.

>Another data type that does not have ownership is the slice

好像是这样的,可以的,Rust强(吹)!

想要像在Java中一样自由的打印对象,需要在struct上进行一些特殊处理,相当于需要实现一个`toString()`方法一样.

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {:?}", rect1);
}
```

完(有点水了)!   
