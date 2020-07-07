---
title: Rust Reference And Borrowing
date: 2020-07-07
categories:
  - programing
markup: mmark
math: true
tags:
    - programing
    - rust
---

### Rust中的引用(Reference)与借用(Borrowing)

在上一节的所有权系统的最后，官方抛出了一个问题，就是在函数使用的时候，如果把参数传进去就把所有权移交到了函数中，那么后续又如何使用这个移交所有权后的变量呢？

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}
```
官方给出的代码如上，但是这样写就太繁琐了。

本节就是讲如何使用`Reference`和`Borrorwing`来解决这个问题。

先来一段代码起手

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

上面这段代码，我理解的是传递的是`s1`变量的指针，所有`s1`变量的所有权没有移交出去。

`&`这个符号，记得在C语言中是**取地址**的意思。

>The &s1 syntax lets us create a reference that refers to the value of s1 but does not own it. Because it does not own it, the value it points to will not be dropped when the reference goes out of scope.

在Rust中这样使用**引用**的方式叫做`Borrowing`借用,这种方式允许获取一个值的引用而不用移交所有权,因为你始终是东西的主人，别人只是借用了你的东西。

>We call having references as function parameters borrowing. As in real life, if a person owns something, you can borrow it from them. When you’re done, you have to give it back.

那么就有个问题，别人借的东西*‘弄坏了怎么办？’*

一下代码不能编译通过是因为我们不能修改别人的东西。
```rust
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

如果强行修改别人的东西那么就需要这样写（相当于在修改别人的东西时，先问一句：“我可以修改你的数据吗？”）

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

但是呢，这样又有很多的限制，你不能把东西借给多个想要修改的人。

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s;

    println!("{}, {}", r1, r2);
}
```

>Data races cause undefined behavior and can be difficult to diagnose and fix when you’re trying to track them down at runtime; Rust prevents this problem from happening because it won’t even compile code with data races!

为什么不许允许同一引用的多出修改，是为了避免数据竞争带来的各种不安全的行为。

一种折中的写法是：

```rust
fn main() {
    let mut s = String::from("hello");

    {
        let r1 = &mut s;
    } // r1 goes out of scope here, so we can make a new reference with no problems.

    let r2 = &mut s;
}
```

但是不能通是把不可变引用和可变引用都借出去：

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    let r3 = &mut s; // BIG PROBLEM

    println!("{}, {}, and {}", r1, r2, r3);
}
```
WTF?! 打脸

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!("{} and {}", r1, r2);
    // r1 and r2 are no longer used after this point

    let r3 = &mut s; // no problem
    println!("{}", r3);
}
```

>Note that a reference’s scope starts from where it is introduced and continues through the last time that reference is used. For instance, this code will compile because the last usage of the immutable references occurs before the mutable reference is introduced:

官方文档告诉我，只要不可变的引用被使用了，那么就会被释放，从而后续定义的可变引用`r3`定义就是能够编译通过的。

这个“奇怪”的设计，官方也给出了安慰

>Even though borrowing errors may be frustrating at times, remember that it’s the Rust compiler pointing out a potential bug early (at compile time rather than at runtime) and showing you exactly where the problem is. Then you don’t have to track down why your data isn’t what you thought it was.

编译时期发能发现的问题，总比运行时的异常好吧 😂。

之前说到会把”借来的东西玩坏“，就是一下这种情况，专业术语`dangle` **悬垂指针**:

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String { // dangle returns a reference to a String

    let s = String::from("hello"); // s is a new String

    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away.
  // Danger!
```

因为是借用来的东西，在使用完后就会被释放掉，然而这里把他作为返回值返回就会造成悬垂指针的出现。

解决这个问题也很简单，只要返回一个新的东西就行了。

```rust
fn main() {
    let string = no_dangle();
}

fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```

学习到现在，这些东西都还是能够理解的，都是为了解决垃圾回收内存安全相关而发明的特性。

完!