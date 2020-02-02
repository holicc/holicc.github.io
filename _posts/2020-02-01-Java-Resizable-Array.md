---
layout: post
title: 【译】Java实现大小可变的数组
---

# Java实现大小可变的数组

> 有时为了快速的访问数据（特别是byte类型），需要将数据放入一个单独且连续的数组中，且数组至少是能够可扩张的。由于Java中的数组是不能够改变其大小的，所以使用数组本身是不能够满足需求的。因此，为了能够容纳原生类型并且可改变其大小的数组，需要自己动手实现。

## 为什么不使用ArrayList？

有人可能会想到使用ArrayList来实现刚才说的需求。当然是可以的，前提是满足以下任意条件：
- 数组中存储的类型是对象类型。
- 数组中存储的是原生类型，且不关心性能和内存。

Java中的ArrayList类只能存储对象类型的数据，不适用于原生类型（例如byte，int，long等）。如果是使用了ArrayList存储原生类型这些数据就会变自动的装箱为对应对象存入ArrayList中，并会在使用的时候自动拆箱为原生类型。由于涉及到了拆箱和装箱机制，当你想要优化提升性能的时候ArrayList不是你想的。

除此之外，存储装箱后的原生类型会带来额外的内存负担。

## 实现代码 - GitHub Repository

本教程的实现代码放置于GitHub方便访问：）

[https://github.com/jjenkov/java-resizable-array](https://github.com/jjenkov/java-resizable-array)

代码由3个Java类和2个单元测试组成

## 可变大小的数组使用场景

想想你有一个服务正在接受各种大小不同的消息。有一些消息大小很小（例如小于4kb），其他则是一些1mb左右的或者更大的消息数据。如果服务端不知道将要接收的数据大小，就不能为将要接收的消息预先分配好内存空间。代替方式是“超额分配”，意思是分配一块足够大的内存空间来接收消息（最大消息不超过5mb，那么每个消息块则分配6mb的内存）。

如果服务端同一时间正从多个链接（100K+）中接收消息，我们必须限制每一条消息所需要分配的内存大小。不能直接按照最大消息大小（例如 1mb或16mb）来给每一条消息分配内存。这样随着连接数和接收消息数的增加，很快就会耗尽服务器内存！100.000 x 1MB = 100GB (大概可以想象一下).

所以我们需要在接收消息前尽可能的分配小的buffer，当小buffer不足以接收大消息数据是我们就需要扩容这个buffer。

## 数组的扩容代价高昂

为了扩容一个数组你需要：

1.分配一个新的更大的数组。

2.将旧数组的数据拷贝到新数组中。

3.回收旧数组。

这一些列的操作代价高昂（特别是性能敏感的应用）！特别是随着旧数组数据越多拷贝数据到新数组这个操作的代价就越大。

注意，在一些带有垃圾回收机制的语言中（例如Java，C#等）尽管你没有明确指令回收旧数组，虚拟机也会为你执行此操作，所以迟早都需要付清回收操作的代价。

## 最小数组的扩容

为了将损失降低到最低，你需要尽可能减少数组扩容的次数，尽量创建一个足够大的数组来保存全部的消息数据。

如果我们接收的大部分消息都是小数据，我们可以先使用小buffer来接收。如果消息大小超过来小buffer的大小，我们就分配一个新的中等大小的数组并且将旧数组中的数据拷贝到新分配的数组中。同样当消息大小超过了中等大小的buffer，就创建一个更大的数组并拷贝过去。这个大的数组需要足够大，大到能够容纳一整个消息数据。如果消息的大小太大以至于不能够处理，那么服务端就应该拒绝接收该消息。

采用这样的策略，大部分的消息只会占用较小的buffer。这样有利于服务器内存的使用。100.000 x 4KB (小的buffer) 只需要400MB。大部分的服务器都能够承受。
即使是4GB（1.000.000 x 4KB）现代的服务器也是可以支撑的。

## 可扩容数组设计

可扩容数组有两个组件组成：

 - ResizableArray
 - ResizableArrayBuffer

ResizableArrayBuffer包含一个单一的，大数组。这个数组分为三个部分。第一部分是为了存储小的数组，第二个是为了存储较大的数组，最后一个是存储大数组。

ResizableArray代表了一个单一的，可扩容的数组，接收到的数据存储与这个数组中的ResizableArrayBuffer里。

这里有一个图来解释ResizableArrayBuffer中的三部分是如何划分的。

![resizeable-array](/assets/images/resizable-array-1.png)

整个ResizableArrayBuffer的大数组中保留一个数据大小的间隙(small, medium, large)，我们需要确保每个数据块都不会被完全充满。例如，接收的small数据块不能占满medium和large的内存区块。同样的large数据块不能占满small和medium的内存区块。

刚开始的时候消息块都是从small数据块开始填充的，如果small数组块被用尽，那么将不会分配新的数组块了，不管meidum块和large块是否还有剩余的空间。

然而，存在一种很小的可能是small数组块占据了整个数组块的大部分空间。

即使small数组块已经被使用完了，也存在一种可能将small消息块的数据增长为medium和large的消息块。

## 优化选项

也可以使用一个单一的数组块来优化整个数组。因为有时一个新的数组块被分配后立马就需要被扩容。在这种情况下你不需要将旧数据拷贝到新的数组块中。你只需要简单的“扩展”这个数组块使其包含两个数组块一个旧的数组块一个新的数组块，并将新的数据写入新的数组块中。这样当一个内存块需要被“扩展”到下一个连续的块中时就节约了数组拷贝的成本。

上述方案存在的缺点是，当不可能将数组块扩展到下一个内存块中，拷贝数据任然是必须的。因此需要进行一次是否可扩展的检查操作。除此之外，如果你使用了一个small数组块，那么拓展操作将会比使用medium和large数组块更加频繁。

## 保持跟踪和回收数据块

ResizableArrayBuffer中的大数组被分为三部分。每一部分又被分为一个更小的块。每个块在每个部分都是同样的大小。例如，在small部分中的所有数据块都是small大小的数据块，在meidum部分都是meidum大小的数据块。在large部分也是同理。

当在一个部分的所有块都是同样大小时，才能更加容易保持跟踪使用过的块和没使用的块的情况。你可以简单的将每一个块所在的索引位置保存在一个队列中。每个部分都需要一个这样的队列。最终，需要一个队列来保存跟踪small块的使用情况，一个队列来保存meidum块的使用情况，一个队列来保存large块的使用情况。

从任何一个部分分配一个块的操作就可以简单的等价于从对应部分的队列中拿走空闲块的所在的索引。回收一个块的操作就可以看作放置一个索引到对应的队列中。

## 扩容并写入数据

可扩容的数组回自行执行扩容操作在你写入数据的时候。如果你尝试写入比数组当前分配的块更多的数据时，数组就会分配一个新的更大的块并拷贝写入的数据到这个新的块中。之前的那个小的数据块就会被释放回收。

## 释放数组块

一旦你使用完了一个可扩容的数组，你应该再次释放它，这样就可以用于存储其他的消息数据了。

## 如何使用ResizableArrayBuffer

展示如何使用一个ResizableArrayBuffer（代码来自GitHub）。

### 创建一个

首先你需要创建一个ResizableArrayBuffer。

```java
int smallBlockSize  =    4 * 1024;
int mediumBlockSize =  128 * 1024;
int largeBlockSize  = 1024 * 1024;

int smallBlockCount  = 1024;
int mediumBlockCount =   32;
int largeBlockCount  =    4;

ResizableArrayBuffer arrayBuffer =
        new ResizableArrayBuffer(
                smallBlockSize , smallBlockCount,
                mediumBlockSize, mediumBlockCount,
                largeBlockSize,  largeBlockCount);
```

这个例子创建了一个含有4kb的small数组块，128kb的meidum数组块，1mb的large数组块 *ResizableArrayBuffer* ，并且包含1024个small数组块（一共可存储4mb的数据），32个medium数组块（也是共4mb）和4个large数组块（共4mb），整个数组可存储12mb的数据。

### 获取一个ResizableArray实例

通过调用 *ResizableArrayBuffer*的 **getArray()**方法。

```java
ResizableArray resizableArray = arrayBuffer.getArray();
```

这样获取到一个最小大小的ResizableArray的实例（预先分配容量为4kb）。


### ResizableArray中写入数据

通过调用 *ResizableArray*的 **write()**方法写入数据。*ResizableArray*类只包含一个write()方法，该方法接收一个 *ByteBuffer*作为参数。

```java
ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

for(int i=0; i < 1024; i++){
    byteBuffer.put((byte) i);
}
byteBuffer.flip():

int bytesCopied = resizableArray.write(byteBuffer);
```

上述代码将 *ByteBuffer* 拷贝到 *ResizableArray*的数组块中。**write()**方法的返回值为从 *ByteBuffer* 中拷贝bytes的数量。

为了防止 *ByteBuffer*中的数据量大于当前ResizableArray分配的大小，ResizableArray会尝试扩容来容纳 *ByteBuffer*中的数据。如果ResizableArray已经扩容的到最大且无法装下 *ByteBuffer*中的数据时，**write()**方法就会返回-1，*ByteBuffer*中的数据自然也不会拷贝到ResizableArray中。

### ResizableArray读取数据

当从ResizableArray中读取数据时相当于直接从ResizableArray中的数组中直接读取。ResizableArray类中包含公开的属性有：

```java
public byte[] sharedArray = null;
public int    offset      = 0;
public int    capacity    = 0;
public int    length      = 0;
```

*sharedArray*属性持有所有ResizableArray实例的数据的数组的引用。这样的数组也存在于ResizableArrayBuffer中。

*offset*属性保存的ResizableArray的数据块起始位置的索引。

*capacity*属性表示ResizableArray数据块的大小。

*length*属性表示ResizableArray实际使用的数据块数量。

从ResizableArray中读取数据相当于读取 *sharedArray[offset]* 到 *sharedArray[offset + length - 1]* 段的数据。

### 释放ResizableArray

一旦使用完了ResizableArray，就需要调用 **free()**方法来释放数组。

```java
resizableArray.free();
```

调用后会归还所占用的数据块到对应的数据块队列中，不论数据块分配的是多大的大小。


## 总结

跟Java中ArrayList实现方式有点类似。为了提升性能尽量要使用原生数据类型，时刻注意数据的回收利用。

## 引用

1.[resizable-array](http://tutorials.jenkov.com/java-performance/resizable-array.html)