---
title: Rust Testing
date: 2020-07-21
categories:
  - programing
markup: mmark
math: true
tags:
    - programing
    - rust
---

## Rust单元测试

在Rust中使用测试，相当于在Java中使用`@Test`注解一样

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

其中`#[test]`就相当于一个注解表示这个函数是一个测试函数，使用`assert_eq!`来进行期望判断。

然后使用`cargo test`就可以运行单元测试了

Rust的测试还自带性能测试只不过需要使用*尝鲜版*

除此之外Rust也带有**assert**功能，其实Java中也有，只不过用的人太少了，都用的junit的assert

Java中的assert

```java
class Scratch {
    public static void main(String[] args) {
        assert "hello".equals(args[0]);
    }
}
```

Rust中的assert

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(larger.can_hold(&smaller));
    }
}
```

两个都是判断一个boolean值，true就相安无事，false就无法通过测试

由于Rust中使用的是**宏**提供的功能相对多一点：`assert_eq!` 和 `assert_ne!`

还支持自定义错误信息

```rust
pub fn greeting(name: &str) -> String {
    format!("Hello {}!", name)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(result.contains("Carol"));
    }
}
```
对错误异常进行测试

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

还支持使用`Result<T,E>`类型作为返回值来编写单元测试

```rust

#![allow(unused_variables)]
fn main() {
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
}
```

但是这样写了就不能使用`#[should_panic]` 了.

再来说说`cargo test`命令，这个命令特别强大，能支持的东西很多；

例如你可以让测试**并行**执行或者**串行**，Rust默认使用的是**并行**执行单元测试的。

可以通过参数设置让单元测试**并行**或**串行**

    $ cargo test -- --test-threads=1

*Showing Function Output*人性化的设计，在单元测试中的打印信息`println!`，只有在单元测试不通过的情况下才会打印在终端上；

如果想不管是否通过都打印信息使用参数

    $ cargo test -- --show-output
    
也可以通过指定测试的名称来运行单个单元测试或者多个单元测试
  
```rust

#![allow(unused_variables)]
fn main() {
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }

    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }

    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}
}
```

以上的单元测试可以通过在命令`cargo test one_two` 来只运行 `add_two_and_two`这个单元测试，也可以通过
命令 `cargo test add`来运行以*add*开头的单元测试

对于忽视的单元测试直接在单元测试上加上`#[ignore]`注解就行了

如果要执行之前忽视的单元测试只需要使用命令

    $ cargo test -- --ignored

Rust可以测试`private`的函数！！！赞

```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

完!