---
title: Rust Smart Pointer
date: 2020-08-04
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

当一个变量的值将要离开作用域的时候，可以使用`Drop`特质来提前感知这一操作并做出相应的处理。例如：*Box<T>*就是使用来`Drop`来释放它所持有的指针的。

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("my stuff"),
    };
    let d = CustomSmartPointer {
        data: String::from("other stuff"),
    };
    println!("CustomSmartPointers created.");
}
```

`Drop`特质相当与告诉编译器，在特定的地方自动插入代码使其调用*drop*方法，好让变量自动释放。

该特质的*drop*方法不支持手动调用，该方法必须由编译器来自动调用，如果手动调用该方法会编译不通过的 （**explicit destructor calls not allowed**）。

原因很简单，Rust并不会智能到见到到了手动调用*drop*就会不执行已经插入的自动*drop*方法，所以会造成*两次释放同一个变量*

>Rust doesn’t let us call drop explicitly because Rust would still automatically call drop on the value at the end of main. This would be a double free error because Rust would be trying to clean up the same value twice.

当然有些情况我们是想要提前释放资源，这种情况就需要使用标准库中**std::mem::drop**的*drop*函数

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    println!("CustomSmartPointer created.");
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}
```

`Rc<T>`（全称：*reference counting*）是一个可以让变量拥有多个*owner*的指针。

官方举了个例子，生动形象的理解`Rc<T>`

>Imagine Rc<T> as a TV in a family room. When one person enters to watch TV, they turn it on. Others can come into the room and watch the TV. When the last person leaves the room, they turn off the TV because it’s no longer being used. If someone turns off the TV while others are still watching it, there would be uproar from the remaining TV watchers!

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```

`Rc::clone`使用的是浅拷贝，这里每次拷贝了之后会使*a*引用计数加一。也可以使用`a.clone()`，如何使用`clone`取决于使用场景。

>We could have called a.clone() rather than Rc::clone(&a), but Rust’s convention is to use Rc::clone in this case. The implementation of Rc::clone doesn’t make a deep copy of all the data like most types’ implementations of clone do. The call to Rc::clone only increments the reference count, which doesn’t take much time. Deep copies of data can take a lot of time. By using Rc::clone for reference counting, we can visually distinguish between the deep-copy kinds of clones and the kinds of clones that increase the reference count. When looking for performance problems in the code, we only need to consider the deep-copy clones and can disregard calls to Rc::clone.

可以使用`Rc::strong_count(&a)`查看当前变量有多少个引用。

`RefCell<T>` (不知名指针)用于打破Rust规则的指针。因为有时Rust编译器不能正确的分析出代码的*Ownership*就会拒绝编译，但是编写代码者能够保证这段代码是符合Rust规则的，所以这个时候就可以使用这个指针来使编译通过，并且在**运行时**来进行*Ownership*的检查工作，如果发现规则被打破，就会*panic*。

>The advantage of checking the borrowing rules at runtime instead is that certain memory-safe scenarios are then allowed, whereas they are disallowed by the compile-time checks. Static analysis, like the Rust compiler, is inherently conservative. Some properties of code are impossible to detect by analyzing the code: the most famous example is the Halting Problem, which is beyond the scope of this book but is an interesting topic to research.

>Because some analysis is impossible, if the Rust compiler can’t be sure the code complies with the ownership rules, it might reject a correct program; in this way, it’s conservative. If Rust accepted an incorrect program, users wouldn’t be able to trust in the guarantees Rust makes. However, if Rust rejects a correct program, the programmer will be inconvenienced, but nothing catastrophic can occur. The RefCell<T> type is useful when you’re sure your code follows the borrowing rules but the compiler is unable to understand and guarantee that.

`Rc<T>, RefCell<T>`这两个指针都只能在**单线程**的环境下使用。

`Box<T>, Rc<T>, RefCell<T>`三者的区别
 - `Rc<T>`允许一个变量有多个拥有者，其他两个只能有一个。
 - `Box<T>`支持*编译期的*可变的和不可变的借用检查；`Rc<T>`支持*编译期*的不可变的借用检查；`RefCell<T>`支持*运行时*的可变的和不可变的借用检查；
 - `Refcell<T>`可以改变一个不可变的变量（Because RefCell<T> allows mutable borrows checked at runtime, you can mutate the value inside the RefCell<T> even when the RefCell<T> is immutable.）
 
突变模式？！
>Mutating the value inside an immutable value is the interior mutability pattern.

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

循环依赖可能会造成内存泄漏的问题，因为使用了`Rc<T>`如果相互依赖那么引用数就永远不会为0。（Java中的GC也有这个问题，用可达性分析解决的）

```rust
use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a initial rc count = {}", Rc::strong_count(&a));
    println!("a next item = {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("a rc count after b creation = {}", Rc::strong_count(&a));
    println!("b initial rc count = {}", Rc::strong_count(&b));
    println!("b next item = {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("b rc count after changing a = {}", Rc::strong_count(&b));
    println!("a rc count after changing a = {}", Rc::strong_count(&a));

    // Uncomment the next line to see that we have a cycle;
    // it will overflow the stack
    // println!("a next item = {:?}", a.tail());
}
```

Rust无法处理循环依赖导致的内存泄露问题，因为这都是写代码自己的写的问题，换成其他语言也会出现这样的问题。

但Rust也不是任人自生自灭，如何防止循环引用的形成，Rust中引入了*弱引用*的概念（Java中的*虚引用*），使用`Rc<T>`中的`Rc::downgrade`会增加`weak_count`的数量，然而回收的时候不管*week_count*是否是0都会回收释放资源，只要`strong_count`为0时。

为了能够从新获取*弱引用*，可以使用`upgrade`方法对其进行升级，返回的是一个`Option<Rc<T>>`

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
}
```
智能指针，有点复杂...

完！


