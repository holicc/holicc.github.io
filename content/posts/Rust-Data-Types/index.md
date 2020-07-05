---
title: Rust Data Types
date: 2020-07-04
categories:
  - programing
markup: mmark
math: true
tags:
    - programing
    - rust
---

### Rust中的数据类型

Rust是一门静态类型的语言(跟Java一样)，每一个值都有其对应的数据类型(*Every value in Rust is of a certain data type*)

#### Number Types

Rust作为一门面向底层的编程语言，在数据类型上拥有了更多的类型，例如**int**类型在Java中只有一种表象形式，及表达32位的整形；然而在Rust中分的就特别细了。
在Rust中整型分为 *Signed*和*Unsigned*分别对应8，16，32，64，128位;
那么什么是 *Signed*有符号和 *Unsigned* 无符号呢？

> 有符号：0111 1111 = 2^6+2^5+2^4+2^3+2^2+2^1+2^0=127;范围是 -128 ~ 127

> 无符号：1111 1111 = 2^7+2^6+2^5+2^4+2^3+2^2+2^1+2^0=255；范围是 0  ~ 255

除了这些，基本的运算符都是相同的，还有包括 **float** 也是有32位和84位的,**char**类型是是4字节子的,通用的 **bool** 类型。

相比Java而言Rust多了一些复合类型的数据结构，例如： **Tuple**元组类型，元组是固定大小的，一旦定义完成就不能改变大小。
可以定义不同类型的元组
```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```


其中还包含相应的*解包、模式匹配*操作。

```rust
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {}", y);
}
```

获取Tuple中的元素直接使用下标就行了
```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```

接下来是 **Array** 也和大部分编程语言相同，只是定义的时候有点奇怪,类型和大小使用分号隔开的

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```
或者使用一个语法糖的方式定义
```rust
let a = [3; 5]; // [3, 3, 3, 3, 3]
```
也是一些基本的数据类型，没啥太特别的东西。

完！

