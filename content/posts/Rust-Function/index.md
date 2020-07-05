---
title: Rust Function
date: 2020-07-04
categories:
  - programing
markup: mmark
math: true
tags:
    - programing
    - rust
---

### Rust中的函数

Rust是支持函数编程的，不像Java完全是使用面向对象的编程语言，虽然可以使用**Function**等，但是还是有些差距的。

Rust中定义函数的关键真是的超级简短，并且提倡使用蛇形命名法（这个挺好的）

```rust
fn main() {
    another_function(5);
}

fn another_function(x: i32) {
    println!("The value of x is: {}", x);
}
```

神奇的是你还可以这样写

```rust
fn main() {
    let x = (let y = 6);
}
```

但是这样写不能编译通过，官方是这样解释的

>Statements do not return values. Therefore, you can’t assign a let statement to another variable, as the following code tries to do; you’ll get an error

既然**let**语句是一个不会有返回值的语句那么如果在表达式中明确了返回值的话就可以了吧

```rust
fn main() {
    let x = 5;

    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {}", y);
}
```
这样看来就有点函数式编程的味道了

在返回值的地方可能有点奇怪,`;`是被省略掉了。可能对于Java程序员的我来说有点不习惯这样的操作。

官方文档举出了一个列子来说明`;`的作用

```rust
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {}", x);
}

fn plus_one(x: i32) -> i32 {
    x + 1
}
// 这样写会便宜不通过的
// fn plus_one(x: i32) -> i32 {
//     x + 1;
// }
```
官方给出的解释如下：
>The main error message, “mismatched types,” reveals the core issue with this code. The definition of the function plus_one says that it will return an i32, but statements don’t evaluate to a value, which is expressed by (), an empty tuple. Therefore, nothing is returned, which contradicts the function definition and results in an error. In this output, Rust provides a message to possibly help rectify this issue: it suggests removing the semicolon, which would fix the error.

上了分号表示这是一个 `statements ` ，返回值为一个空的元组及为空，所以和函数定义的返回i32类型不符合，所以编译不通过。

嗯…… 奇怪

完！

