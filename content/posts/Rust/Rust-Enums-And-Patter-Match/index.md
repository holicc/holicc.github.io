---
title: Rust Enums And Patter Match
date: 2020-07-09
categories:
  - programing
markup: mmark
math: true
tags:
    - programing
    - rust
---

### Rust中的枚举

`enum`枚举类型，一种熟悉的数据结构，在Java中也有对应的类型，就不赘述了.

```rust
fn main() {
    enum IpAddrKind {
        V4,
        V6,
    }

    struct IpAddr {
        kind: IpAddrKind,
        address: String,
    }

    let home = IpAddr {
        kind: IpAddrKind::V4,
        address: String::from("127.0.0.1"),
    };

    let loopback = IpAddr {
        kind: IpAddrKind::V6,
        address: String::from("::1"),
    };
}
```

枚举类中也可以有成员变量

```rust
fn main() {
    enum IpAddr {
        V4(String),
        V6(String),
    }

    let home = IpAddr::V4(String::from("127.0.0.1"));

    let loopback = IpAddr::V6(String::from("::1"));
}
```

不仅如此，枚举类中的成员还可以拥有不同的类型；

```rust
fn main() {
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from("::1"));
}
```

甚至还可以放置匿名结构体

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

其中的`Move`就是定义的匿名结构体.

后面就讲了一下`Option`,这个Java1.8以后也有的东西，所以还是很熟悉的，只是在Rust中它使用一个枚举类型；

用法和Java都类似的.

rust版本
```rust
fn main() {
    let a: Option<i32> = None;

    let c = a.expect("error message ");

    println!("hello world {}", c)
}
```
java版本
```java
class Scratch {
    public static void main(String[] args) throws Exception {
        String a="asd";
        Optional.ofNullable(a).orElseThrow(()->new Exception("zxc"));
    }
}
```

神奇的`match`模式匹配？可惜在Scala中就已经见过了，感觉没啥好学习的。

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}

fn main() {
    value_in_cents(Coin::Quarter(UsState::Alaska));
}
```
最后这个`if let`表达式让我有点懵逼，咋一看感觉没啥用啊。

```rust
fn main() {
    let some_u8_value = Some(0u8);
    match some_u8_value {
        Some(3) => println!("three"),
        _ => (),
    }
}
```

加个`let`有啥用呢？还不如直接写成`if-else`的形式就行了嘛？

```rust
fn main() {
    let some_u8_value = Some(0u8);
    if let Some(3) = some_u8_value {
        println!("three");
    }
}
```

可能为了能够将变量绑定到一个类型里面把，形如：

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn main() {
    let coin = Coin::Penny;
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!("State quarter from {:?}!", state);
    } else {
        count += 1;
    }
}
```
是为了简化`match`操作而创建的语法糖.

完!
