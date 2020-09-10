---
title: Rust Concurrency
date: 2020-08-05
categories:
  - programing
markup: mmark
math: true
tags:
    - programing
    - rust
---

## Rust Concurrency

Rust中的线程（*Thread*）使用的是`1:1 model`而不是像Golang中的*goroutine*使用的是`M:N model`；

因为Rust是一门更接近底层的编程语言，使用`M:N`模型需要更大的运行时开销来支持。

>The green-threading M:N model requires a larger language runtime to manage threads. As such, the Rust standard library only provides an implementation of 1:1 threading. Because Rust is such a low-level language, there are crates that implement M:N threading if you would rather trade overhead for aspects such as more control over which threads run when and lower costs of context switching, for example.

线程的创建

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

是个线程就有`join`方法

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

Rust的线程通信借鉴了Go使用了*channel*.

>One increasingly popular approach to ensuring safe concurrency is message passing, where threads or actors communicate by sending each other messages containing data. Here’s the idea in a slogan from the Go language documentation: “Do not communicate by sharing memory; instead, share memory by communicating.”

使用`mpsc::channel`函数来创建*channel*,其中`mpsc`表示多个生产者一个消费者（*multiple producer, single consumer*），只要消费者接收到了一个生产者的消息就会中断整个*channel*。

```rust
use std::sync::mpsc::channel;
use std::thread::spawn;

fn main() {
    let (tx, rx) = channel();

    spawn(move || {
        tx.send(String::from("hello")).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

为了能够有多个生产者，需要*clone*一下`tx`

```rust
use std::rc::Rc;
use std::sync::mpsc;
use std::thread::{sleep, spawn};
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();
    let tx1 = tx.clone();

    spawn(move || {
        tx.send("one").unwrap();
        sleep(Duration::from_millis(1000));
    });

    spawn(move || {
        tx1.send("two").unwrap();
        sleep(Duration::from_millis(1000));
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

`rx`中的*recv*方法会阻塞当前线程直到接收到消息。如果不想阻塞可以使用*try_recv*方法，该方法会立即返回。

使用**互斥锁**解决并发数据访问问题。

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

正确使用锁的姿势

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

内容有点少啊.

完！