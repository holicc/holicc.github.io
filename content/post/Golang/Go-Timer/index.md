---
title: Go Timer
date: 2020-12-10
categories:
  - programing
markup: mmark
math: true
tags:
    - programing
---

## Golang Timer

```go
// The Timer type represents a single event.
// When the Timer expires, the current time will be sent on C,
// unless the Timer was created by AfterFunc.
// A Timer must be created with NewTimer or AfterFunc.
type Timer struct {
	C <-chan Time
	r runtimeTimer
}

// Interface to timers implemented in package runtime.
// Must be in sync with ../runtime/time.go:/^type timer
type runtimeTimer struct {
	pp       uintptr
	when     int64
	period   int64
	f        func(interface{}, uintptr) // NOTE: must not be closure
	arg      interface{}
	seq      uintptr
	nextwhen int64
	status   uint32
}
```

### 全局四叉堆

Go 1.10 之前的计时器都使用最小四叉堆实现，所有的计时器都会存储在如下所示的结构体

```go
var timers struct {
	lock         mutex
	gp           *g
	created      bool
	sleeping     bool
	rescheduling bool
	sleepUntil   int64
	waitnote     note
	t            []*timer
}
```

然而全局四叉堆共用的互斥锁对计时器的影响非常大，计时器的各种操作都需要获取全局唯一的互斥锁，这会严重影响计时器的性能

### 分片四叉堆

Go 1.10 将全局的四叉堆分割成了 64 个更小的四叉堆5。在理想情况下，四叉堆的数量应该等于处理器的数量，但是这需要实现动态的分配过程，所以经过权衡最终选择初始化 64 个四叉堆，以牺牲内存占用的代价换取性能的提升。

```go
const timersLen = 64

var timers [timersLen]struct {
	timersBucket
}

type timersBucket struct {
	lock         mutex
	gp           *g
	created      bool
	sleeping     bool
	rescheduling bool
	sleepUntil   int64
	waitnote     note
	t            []*timer
}
```

将全局计时器分片的方式，虽然能够降低锁的粒度，提高计时器的性能，但是造成的处理器和线程之间频繁的上下文切换却成为了影响计时器性能的首要因素。

### 网络轮询器

在最新版本的实现中，计时器桶已经被移除，所有的计时器都以最小四叉堆的形式存储在处理器

```go
type p struct {
	...
	timersLock mutex
	timers []*timer

	numTimers     uint32
	adjustTimers  uint32
	deletedTimers uint32
	...
}
```

目前计时器都交由处理器的网络轮询器和调度器触发，这种方式能够充分利用本地性、减少线上上下文的切换开销，也是目前性能最好的实现方式。


### 触发

*runtime.checkTimers* 是调度器用来运行处理器中计时器的函数




```go
func sysmon() {
	...
	for {
		...
		now := nanotime()
		next, _ := timeSleepUntil()
		...
		lastpoll := int64(atomic.Load64(&sched.lastpoll))
		if netpollinited() && lastpoll != 0 && lastpoll+10*1000*1000 < now {
			atomic.Cas64(&sched.lastpoll, uint64(lastpoll), uint64(now))
			list := netpoll(0)
			if !list.empty() {
				incidlelocked(-1)
				injectglist(&list)
				incidlelocked(1)
			}
		}
		if next < now {
			startm(nil, false)
		}
		...
}
```

**以上只是触发的一种方式**

1. 调用 runtime.timeSleepUntil 函数获取计时器的到期时间以及持有该计时器的堆；
2. 如果超过 10ms 的时间没有轮询，调用 runtime.netpoll 轮询网络；
3. 如果当前有应该运行的计时器没有执行，可能因为存在无法被抢占的处理器，这时我们应该系统新的线程计时器；

标准库中的计时器在大多数情况下是能够正常工作并且高效完成任务的，但是在遇到极端情况或者性能敏感场景时，它可能没有办法胜任，**而在 10ms 的这个粒度下**，作者在社区中也没有找到能够使用的计时器实现，一些使用时间轮算法的开源库也不能很好地完成这个任务。

### Reference

 - [计时器](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-timer/#63-计时器)