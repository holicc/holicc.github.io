---
title: Rust Collections
date: 2020-07-15
categories:
  - programing
markup: mmark
math: true
tags:
    - programing
    - rust
---

### Rust中的集合类型

Rust的标准库中提供了常用的集合类型：`Vect`(相当于Java中的List)，`Map`(相当于Java中的Map)，在使用这些集合的过程中需要注意的是
`Ownership`和`let & let mut`的一些注意事项

使用`Vect`直接使用函数创建或者使用宏

```rust
let v: Vec<i32> = Vec::new();
//let c = Vec![1,2,3,4];
```
可以看到Vec是一个支持泛型的集合类型，在创建的时候需要指定其内部元素的具体类型；

使用`mut`定义Vector就可以对它进行修改

````rust
 let mut v = Vec::new();

 v.push(5);
 v.push(6);
 v.push(7);
 v.push(8);
````

当然了根据任何变量都有其作用域（Scope）如果不在作用域中了就会被释放掉，并且Vector中的元素也会相应的被释放掉

```rust
{
        let v = vec![1, 2, 3, 4];

        // do stuff with v
} // <- v goes out of scope and is freed here
```
rust中访问Vector元素有两种方法一种是使用下标访问一种是使用get方法：

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    let third: &i32 = &v[2];
    println!("The third element is {}", third);

    match v.get(2) {
        Some(third) => println!("The third element is {}", third),
        None => println!("There is no third element."),
    }
}
```

区别在于get方法是空安全的，返回的是一个`Option<&T>`类型，就是说如果get的一个不存在vector中的元素只会返回一个`Option<None>`,而使用   
`&[]`下标访问会panic，因为你访问了一个不存在的元素；

```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];

    let first = &v[0];

    v.push(6);

    println!("The first element is: {}", first);
}
```

为什么以上代码无法编译通过，可以说是充满了Rust的哲学😂

>This error is due to the way vectors work: adding a new element onto the end of the vector might require allocating new memory and copying the old elements to the new space, if there isn’t enough room to put all the elements next to each other where the vector currently is. In that case, the reference to the first element would be pointing to deallocated memory. The borrowing rules prevent programs from ending up in that situation.

官方解释说这跟Vector的实现机制有关，在往Vector中添加新的元素的时候可能会因为原始空间不够容纳这么多元素而导致将原来的元素都拷贝到一个新的地方，从而改变了内部元素的指针，如果这时有一个外部借用的指针还停留在原始的未知就会造成内存访问不正确的问题。

迭代方式

```rust
fn main() {
    let v = vec![100, 32, 57];
    for i in &v {
        println!("{}", i);
    }
    for i in &mut v {
        *i += 50; //* 表示解引用
    }
}
```

官方文档也讲了一些Vector的一些技巧

```rust
fn main() {
    enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }

    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
}
```

在Rust中String也使用集合类型实现的，底层就是一个bytes集合，所以官方也放到了这一章节来讲。

```rust
fn main() {
    let mut s = String::from("foo");
    s.push_str("bar");
}
```

对于如何在String中使用`+`，这也是一门学问

>This isn’t the exact signature that’s in the standard library: in the standard library, add is defined using generics. Here, we’re looking at the signature of add with concrete types substituted for the generic ones, which is what happens when we call this method with String values. We’ll discuss generics in Chapter 10. This signature gives us the clues we need to understand the tricky bits of the + operator.
 
>First, s2 has an &, meaning that we’re adding a reference of the second string to the first string because of the s parameter in the add function: we can only add a &str to a String; we can’t add two String values together. But wait—the type of &s2 is &String, not &str, as specified in the second parameter to add. So why does Listing 8-18 compile?
 
>The reason we’re able to use &s2 in the call to add is that the compiler can coerce the &String argument into a &str. When we call the add method, Rust uses a deref coercion, which here turns &s2 into &s2[..]. We’ll discuss deref coercion in more depth in Chapter 15. Because add does not take ownership of the s parameter, s2 will still be a valid String after this operation.
 
>Second, we can see in the signature that add takes ownership of self, because self does not have an &. This means s1 in Listing 8-18 will be moved into the add call and no longer be valid after that. So although let s3 = s1 + &s2; looks like it will copy both strings and create a new one, this statement actually takes ownership of s1, appends a copy of the contents of s2, and then returns ownership of the result. In other words, it looks like it’s making a lot of copies but isn’t; the implementation is more efficient than copying.

官方用了很长一段来说明如何对String类型来使用`+`号，总结一下就是`+`其实是就是String的add方法

看一下方法签名

```rust
fn add(self, s: &str) -> String 
```

接受的对象是一个`&str`,所以在使用的时候可以是

```rust
let a = String::from("test");
let b = a + "zxc";
```

也可以是

```rust
let a = String::from("test");
let b = String::from("hello");
let c = a + &b
```
这里的`&b`是一个`&String`会被强转为一个 `&str`类型。

String类型也不支持下标访问，（对Java程序员来说很OK）

最后官方说了一句让人无语的话

>To summarize, strings are complicated. Different programming languages make different choices about how to present this complexity to the programmer. Rust has chosen to make the correct handling of String data the default behavior for all Rust programs, which means programmers have to put more thought into handling UTF-8 data upfront. This trade-off exposes more of the complexity of strings than is apparent in other programming languages, but it prevents you from having to handle errors involving non-ASCII characters later in your development life cycle.

好吧，下一个是HashMap

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
}
```

对于非拷贝的值例如String类型，如果把值insert到map中就是对这个值所有权的转移到map中了；
```rust
fn main() {
    use std::collections::HashMap;

    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");

    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // field_name and field_value are invalid at this point, try using them and
    // see what compiler error you get!
}
```

如何迭代，很好理解

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }
}
```

一些api介绍

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);

    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);

    println!("{:?}", scores);
}
```

相当于Java中的`map.putIfAbsent`


其他的就不赘述了，就是了解一下Rust中的集合

完！