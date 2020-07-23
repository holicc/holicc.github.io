---
title: Rust Smart Pointer
date: 2020-07-23
categories:
  - programing
markup: mmark
math: true
tags:
    - programing
    - rust
---

## Rust Smart Pointer (智能指针)

普通指针（Reference）: `&` 使用这个定义的，只是指向了对应的内存地址，并没有其他的功能；

智能指针（Smart Pointer）就是能够拥有多个*owner*并且在没有*owner*时自动清理回收。（GC？！），并且提供了一些其他功能；

> This pointer enables you to have multiple owners of data by keeping track of the number of owners and, when no owners remain, cleaning up the data.

Rust中普通的指针只是一个借用，然而智能指针是拥有

>In Rust, which uses the concept of ownership and borrowing, an additional difference between references and smart pointers is that references are pointers that only borrow data; in contrast, in many cases, smart pointers own the data they point to.

指针指针是一种实现了`Deref`和`Drop`接口的结构体

`box`是Rust中常用的智能指针，它会将存储的数据放在堆中而不是栈上。该指针也不会影响性能，只是没有额外的功能罢了。

 - When you have a type whose size can’t be known at compile time and you want to use a value of that type in a context that requires an exact size
 - When you have a large amount of data and you want to transfer ownership but ensure the data won’t be copied when you do so
 - When you want to own a value and you care only that it’s a type that implements a particular trait rather than being of a specific type
 
 ```rust
 fn main() {
     let b = Box::new(5);
     println!("b = {}", b);
 }
```

使用`Deref`来控制解引用的过程

```rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```