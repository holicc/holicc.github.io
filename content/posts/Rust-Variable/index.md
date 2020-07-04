---
title: Rust Variables
date: 2020-07-03
categories:
  - programing
markup: mmark
math: true
tags:
    - programing
    - rust
---

### Rust中的变量与常量的定义

Rust中定义变量使用的关键字是 **let** 关键字，该关键字定义的变量是不可变的;想要定义一个可变的变量需要再使用一个关键 **let mut**对变量进行定义;

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

为什么这样设计官方给出了解释:

> There are multiple trade-offs to consider in addition to the prevention of bugs. For example, in cases where you’re using large data structures, mutating an instance in place may be faster than copying and returning newly allocated instances. With smaller data structures, creating new instances and writing in a more functional programming style may be easier to think through, so lower performance might be a worthwhile penalty for gaining that clarity.

大致是说这是一种折中的设计，为了能够在操纵大的数据结构（类型Java中的对象）时，避免使用拷贝创建一个新的实例，而是直接操作其引用对其进行修改；相反在操作小的对象时拷贝创建一个新的对象更加符合函数式编程的风格更容易理解，当然也会消耗一点性能；

*这样说是不是意味着 **let**的使用是进行值的拷贝，**let mut**只是传递的引用？*

不过这样设计对于Java程序员来说也是能够理解的。

定义常量使用的是关键字 **const**，可以在任何地方定义常量,但是需要**注意的是定义常量的时候要把类型补充上去**;

```rust
const A :u8 = 1;

fn main() {
    const C:u16=33;
    let x = 5;
    println!("x has the value {}", x);
}
```

### Rust变量遮蔽(Shadowing)

这是算是一个有意思的特性吧。通过定义同名的变量来进行变量的遮蔽;

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    let x = x * 2;

    println!("The value of x is: {}", x);
}
```

咋一看这不就是定义一个变量然后可以修改它的值嘛。Java中都有这样的,换个名字就变成 *Shadowing* 了。 - -| 高级。。

```java
public static void main(String[] args) {
        int x = 5;
        x = x + 1;
        x = x * 2;
        System.out.println(x);
    }
```

或者相当于

```rust
fn main() {
    let mut x = 5;

    x = x + 1;

    x = x * 2;

    println!("The value of x is: {}", x);
}
```
对于以上的两种方式，官方给出了两个解释:

>Shadowing is different from marking a variable as mut, because we’ll get a compile-time error if we accidentally try to reassign to this variable without using the let keyword. By using let, we can perform a few transformations on a value but have the variable be immutable after those transformations have been completed.

第一个解释可以理解为：如果我们不使用**let**关键来进行遮蔽，那么意味着let定义的变量是可变的。这会导致编译出错，也会给**let**关键字带来歧义（本身代表的是不可变的意思），这个解释勉强可以理解吧。

>The other difference between mut and shadowing is that because we’re effectively creating a new variable when we use the let keyword again, we can change the type of the value but reuse the same name. For example, say our program asks a user to show how many spaces they want between some text by inputting space characters, but we really want to store that input as a number:

第二个解释就很好理解了，只用看一下官方的列子就能明白了。

```rust
fn main() {
    let spaces = "   ";
    let spaces = spaces.len();
}
```

完！
