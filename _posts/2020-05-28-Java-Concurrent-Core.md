---
layout:     post
title:      "Java并发源码中的“巅峰”"
subtitle:   "AbstractQueuedSynchronizer,"
date:       2020-05-28
author:     "JohnReese"
header-img: "img/2020-2-11/stone.jpg"
mathjax: True
tags:
    - Java
---

阅读这篇文章，你可能需要先了解`Java.Util.Concurrent`包中常见的用于实现线程同步的类的常规用法。本文的目的是基于源码，尝试对比较重要的并发类的实现方式给出通俗的解释。

一方面是因为源码的细节很多，给出源码而不能完全解释其中的每个点就没啥意思了；另一方面也是自身的水平不足。所以只给出易于理解的主体实现方式。

基于`JDK 8`。

首先，一些概念：
1. (线程)同步:
    > 即当有一个线程在对内存进行操作时，其他线程都不可以对这个内存地址进行操作，直到该线程完成操作， 其他线程才能对该内存地址进行操作，而其他线程又处于等待状态，实现线程同步的方法有很多，临界区对象就是其中一种。———— 摘自百度百科
    
    其实，照我的理解就是线程间存在依赖关系致使我们需要手动地编码以确保线程间以正确的顺序执行从而得到预期的结果。
2. CAS: 即`Compare-and-Swap`。这是一种通过硬件方式实现原子性更新的方式。


## AbstractQueuedSynchronizer

`AQS`是`Java.Util.Concurrent`包中许多用于实现同步功能类的基础，使用方式往往是创建一个继承于`AQS`的内部类。

`AQS`本身作为一个抽象类，具体实现的方法并不多。但也有一些不可能绕过的细节：
1. 使用`int`类型的`state`变量表示当前的同步状态。比较易于理解的例子是假设我们当前需要设计一个互斥锁，则可以令`state`为1表示锁可用，`state`为0表示已有线程占用。
2. `AQS`实现了`acquire`和`release`方法，但其本质是调用了`tryAcquire`和`tryRelease`两个方法。而这两个方法是需要继承的同步类具体实现的。（当然，`acquire`/`release`方法有多种不同的调用情况，比如可以指定等待时间，超时就自动返回等。但这些就像开头我所说的，我们只关注最主体的部分。）
3. `AQS`内部使用链表形式来保存当前被阻塞线程的信息，在其它线程释放锁（互斥资源）时会通过链表来唤醒这些被阻塞的线程。其中内部类`Node`是构成此链表的元素。
4. `compareAndSetState`方法以`CAS`方式尝试修改同步状态。


接下来，我们以`java.util.concurrent.locks`包中的`ReentrantLock`作为例子看看如何通过`AQS`实现同步类。 

其中`sync`是`ReentrantLock`中继承`AQS`的内部类（`Sync`）的实例。需要注意的是，`ReentrantLock`为了实现公平锁与非公平锁，内部类`Sync`依旧是抽象类，但我们的讨论就以其非公平锁的实现`NonfairSync`为主。

首先是`lock`方法。首先直接交给`sync`去处理。因为是互斥锁，所以具体的实现类`NonfairSync`(见下图)会首先调用`compareAndSetState`方式尝试修改同步状态，即将0置为1。如果修改成功，那么就可以把当前线程设置为拥有此锁的线程，否则就调用`acquire`方法。

```java
// ReentrantLock.java
// Line: 205
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```

前面已经提到，`AQS`已经实现了`acquire`方法，它会调用`tryAcquire`方法，但`ReentrantLock`对于它的实现并没有特殊之处，就是再次尝试修改状态，因此如果修改状态失败就会紧接着调用`AQS`中的`addWaiter`和`acquireQueued`方法，简单点来说就是阻塞当前线程并将其信息保存在之前提到的链表中。