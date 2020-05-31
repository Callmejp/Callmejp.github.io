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

阅读这篇文章，你可能需要先了解`Re`包中常见的用于实现线程同步的类的常规用法。本文的目的是基于源码，尝试对比较重要的并发类的实现方式给出通俗的解读。

一方面是因为源码的细节很多，给出源码而不能完全解释其中的每个点就没啥意思了；另一方面也是自身的水平不足。所以只给出易于理解的主体实现方式。

基于`JDK 8`。

首先，一些概念：
1. (线程)同步:
    > 即当有一个线程在对内存进行操作时，其他线程都不可以对这个内存地址进行操作，直到该线程完成操作， 其他线程才能对该内存地址进行操作，而其他线程又处于等待状态，实现线程同步的方法有很多，临界区对象就是其中一种。—— 摘自百度百科
    
    照我的理解就是线程间存在依赖关系致使我们需要手动地编码以确保线程间以正确的顺序执行从而得到预期的结果。
2. CAS: 即`Compare-and-Swap`。这是一种通过**CPU硬件方式**实现原子性更新的方式。在之后聊到时会做进一步的详述。
3. 线程状态：主要参考`Java`中`Thread`的内部枚举类`State`。如下解读主要借鉴《Java核心技术面试精讲》。这里详述状态的目的是因为我之前会用“阻塞”（这个词语）来表示“等待”的状态，所以在此明确用语以免之后论述的歧义。也就是说，`RUNNABLE`和`BLOCKED`都意味着线程还是“活跃”着的，只有处于`WAITING`状态的线程才需要唤醒操作。
    * New: 被创建但还未开始被执行。
    * RUNNABLE: 表示该线程已经在`JVM`中执行，当然由于执行需要计算资源，它可能是正在运行，也可能还在等待系统分配给它`CPU`片段，在就绪队列里面排队。
    * BLOCKED: 阻塞表示线程在等待`Monitor lock`。比如，线程试图通过`synchronized`去获取某个锁，但是其他线程已经独占了，那么当前线程就会处于阻塞状态。
    * WAITING: 表示正在等待其他线程采取某些操作。一个常见的场景是类似生产者消费者模式，发现任务条件尚未满足，就让当前消费者线程等待，另外的生产者线程去准备任务数据，然后通过类似`notify`等动作，通知消费线程可以继续工作了。
    * TIMED_WAIT: 其进入条件和等待状态类似，但是调用的是存在超时条件的方法。
    * TERMINATED: 不管是意外退出还是正常执行结束，线程已经完成使命，终止运行。


## AbstractQueuedSynchronizer

`AQS`是`Java.Util.Concurrent`包中许多用于实现同步功能类的基础，使用方式往往是创建一个继承于`AQS`的内部类。

`AQS`本身作为一个抽象类，肯定只实现了部分的方法，但也有一些不能绕过的要点：
1. 使用`int`类型的`state`变量表示当前的同步状态。比较易于理解的例子是假设我们当前需要设计一个互斥锁，则可以令`state`为1表示锁可用，`state`为0表示已有线程占用。当然，数值对应何种状态，这完全取决于开发者的具体设计。
2. `AQS`实现了`acquire`和`release`方法（顾名思义：获取和释放资源），但其本质是调用了`tryAcquire`和`tryRelease`两个方法。而后面的两个方法是需要继承的同步类来具体实现的。（当然，`acquire`/`release`方法有多种不同的调用情况，比如可以指定带超时的获取锁尝试。但我们只关注最主体的部分。）
3. `AQS`内部使用`FIFO`的队列来保存当前`等待`（即我们在第3点明确的状态表述）的线程的信息（以链表形式实现），在其它线程释放锁（资源）时会通过此链表来唤醒这些`等待`的线程。其中`AQS`的内部类`Node`是构成此链表的元素。其中`Node`使用`int`类型的`waitStatus`变量来表示本节点（也就是线程）的状态。
4. `compareAndSetState`方法以`CAS`方式尝试修改同步状态。


接下来，我们以`java.util.concurrent.locks`包中的`ReentrantLock`作为例子看看如何通过`AQS`实现同步类。 

其中`sync`是`ReentrantLock`中继承`AQS`的内部类（`Sync`）的实例。需要注意的是，`ReentrantLock`为了实现公平锁与非公平锁，内部类`Sync`依旧是抽象类，但我们的讨论以非公平锁的实现`NonfairSync`为主。

首先是`lock`方法。`ReentrantLock`会直接交给`sync`去处理。因为是互斥锁，所以具体的实现类`NonfairSync`(见如下代码)会首先调用`compareAndSetState`方式尝试修改同步状态，即将0置为1。如果修改成功，那么就可以把当前线程设置为拥有此锁的线程，否则就调用`acquire`方法。

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

前面已经提到，`AQS`已经实现了`acquire`方法，它会调用`tryAcquire`方法，但`ReentrantLock`对于`tryAcquire`的具体实现并没有特殊之处，就是再次尝试修改同步状态，因此如果修改状态失败就会紧接着调用`AQS`中的`addWaiter`和`acquireQueued`方法，简单来说就会令当前线程进入等待并将其信息保存在之前提到的链表中。

有`lock`就有对应的`unlock`方法。同样，`ReentrantLock`也会直接交给`sync`去处理。其中`tryRelease`（见如下代码）在执行前会首先判断当前的执行线程是不是当前记录的占有锁的线程。如果当前同步状态减去本次所释放的资源后为0就表示当前已没有线程占有锁了，因此依次修改相关的记录信息。注意，`ReentrantLock`虽然本身只代表“一把锁”，但因为其是可重入的，也就是已经持有锁的线程可以再次调用`lock`方法，所以同步状态可能会大于1，并不是只有1和0两种状态。

```java
// ReentrantLock.java
// Line: 148
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

既然以`ReentrantLock`作为例子就不可避免地需要继续讨论`condition`的具体实现方式。`condition`的`await`和`signal`方法相较于`Object`本身的`wait`和`notify`原语提供了更为灵活且易懂的使用方式。

虽然`condition`是通过`ReentrantLock`对象的`newCondtion`方法创建的，但内部其实是`sync`对象创建的（`AQS`的内部类）`ConditionObject`的实例。

简单来说，每个`ConditionObject`对象在内部各自维护了一个类似于`AQS`中的链表来表示因为此`condition`未满足而进入等待的线程的信息。可以将`AQS`中的链表看作“主链表”，其中的线程**抢占**的是`ReentrantLock`这个锁，而每个`ConditionObject`维护的线程，**需要**的是开发者所定义的每个“条件”。

那么，我们来看看调用某个`condition`的`await`方法，`AQS`会做什么事情（见如下代码）。（我抽去了大部分源码）`addConditionWaiter`会往链表中添加当前线程，而`fullyRelease`则会唤醒主链表中的线程。因为我们知道，`condition`的`await`和`signal`方法必须是在持有锁的情况下才能调用的，所以当前线程调用`await`方法意味着它会释放当前自己所持有的锁。

```java
// AbstractQueuedSynchronizer.java
// Line: 2032
public final void await() throws InterruptedException {
    Node node = addConditionWaiter();
    int savedState = fullyRelease(node);
}
```

可以看到，如果忽略细节，尝试了解同步类的主体实现思路还是比较直观的。但还有一个细节需要注意，即当前运行的线程是如何令自己进入等待状态的？稍加深究就会发现`Unsafe`类才是最后的归宿。其实，Java中很多并发的实现都离不开它。`Unsafe`类比较特殊，它是`sun.misc`包中的类。

这里我找了一些资料：
> Unsafe类使Java拥有了像C语言的指针一样操作内存空间的能力，一旦能够直接操作内存，这也就意味着（1）不受jvm管理，也就意味着无法被GC，需要我们手动GC，稍有不慎就会出现内存泄漏。（2）Unsafe的不少方法中必须提供原始地址(内存地址)和被替换对象的地址，偏移量要自己计算，一旦出现问题就是JVM崩溃级别的异常，会导致整个JVM实例崩溃，表现为应用程序直接crash掉。（3）直接操作内存，也意味着其速度更快，在高并发的条件之下能够很好地提高效率。 

> Unsafe中的方法大体可以归结为以下几类：
（1）初始化操作
（2）操作对象属性
（3）操作数组元素
（4）线程挂起和恢复
（5）CAS机制   —— 摘自https://baijiahao.baidu.com/s?id=1648712942552745701&wfr=spider&for=pc

综上，我们不难理解它为什么被称为`Unsafe`了，也看到了它能实现线程的挂起和恢复功能。我们讨论的进入等待就是调用了其`park`方法。

以上通过`ReentrantLock`大致知晓了如何借助`AQS`实现同步类。虽然还有许多满足不同需求的同步类，但万变难离其宗，`AQS`是关键。


## CAS

加锁始终是一个相对较重的操作，能不能做到`lock-free`而不影响程序并发时执行的正确性？答案是肯定的，`CAS`就是相应的手段。

`CAS`通常需要3个操作数：内存地址V，旧的预期值A，即将要更新的目标值B。`CAS`指令执行时，当且仅当内存地址V所对应的值与预期值A相等时，则将内存地址V的值修改为B，否则就什么都不做。整个比较并替换的操作是一个原子操作。

可以预见的是，`CAS`并不能保证一次成功，所以一般会以类似“自旋”的方式尝试多次。因此，进一步可以预见的，在竞争激烈的场景下，可能`CAS`的表现并不好，而且会消耗大量的CPU资源。`Brian Goetz`在《Java并发编程实战》就做了相应的实验，可以看到在比较极端的场景下，使用`CAS`的性能表现还稍弱后于基于锁的`ReentrantLock`。但是，在竞争适度的情况下`CAS`就领先很多了。

![对比1](/img/2020-5-31/1.JPG)

![对比2](/img/2020-5-31/2.JPG)

但是世上难有完美的事物——1.虽然大多数情况下，开发者可以直接使用Java提供的类库，所以一般来说`CAS`对于开发者是透明的。但我们应该认识到相对于使用锁，它提高了编码的复杂性；2.使用`CAS`往往意味着需要牺牲部分的数据一致性。

我就以`ConcurrentLinkedQueue`作为例子来解释上述的两个观点。不过在阅读源代码之前，我们必须对于`Michael-Scott`提出的此队列算法有一个大概了解：
1. 因为是通过链表实现队列，所以肯定需要记录头节点（head）和尾节点（tail）
2. 因为不加锁的原因，头节点和尾节点并不能保证时时指向第一个节点和最后一个节点
3. 所以插入元素大致分为2个部分：找到尾节点；通过`CAS(null, newNode)`插入新结点。（尾节点的nextNode一定是null）
4. 如第2点所说，即使在第3点中找到了尾节点，在调用`CAS`的时候，尾节点也可能会产生变化。但是不怕，多尝试几次即可。只要保证`CAS`这一步的原子性，就能保证整体的正确性

`offer`方法(向队列尾部添加元素)的源代码如下，因为相对比较复杂，所以直接将解读对应在源码中：

```java
// ConcurrentLinkedQueue.java
// Line: 326
public boolean offer(E e) {
    checkNotNull(e);
    final Node<E> newNode = new Node<E>(e);
    // 外层是“死循环”CAS，直到入队成功
    for (Node<E> t = tail, p = t;;) {
        Node<E> q = p.next;
        /* 判断p是不是尾节点。
        其中判断是不是尾节点的依据是该节点的next是不是null */
        if (q == null) {
            /* 设置p节点的下一个节点为新节点，设置成功则casNext返回true；
            否则返回false，说明有其他线程更新过尾节点，就重新来过 */
            if (p.casNext(null, newNode)) {
                /* 如果p != t，则将入队节点设置成tail节点，更新失败了也没关系
                因为失败了表示有其他线程成功更新了tail节点*/
                if (p != t) 
                    casTail(t, newNode); 
                return true;
            }
        }
        else if (p == q)
        /* 这里较为复杂，我就摘录网上的资料：
        如果next为自身，表示是哨兵节点
        则从head重新向后遍历（因为掉队了） 
        */
            p = (t != (t = tail)) ? t : head;
        else
        /* 其他情况，则继续向后查找尾节点 */
           p = (p != t && t != (t = tail)) ? t : q;
    }
}
```

至于牺牲的部分数据一致性可以从其`size`方法的文档中看出：

> Beware that, unlike in most collections, this method is NOT a constant-time operation. Because of the asynchronous nature of these queues, determining the current number of elements requires an O(n) traversal. Additionally, if elements are added or removed during execution
of this method, the returned result may be inaccurate. Thus, this method is typically not very useful in concurrent applications.

你可以看出，因为不会保存`size`变量，所以此方法每次都需要`O(n)`的时间复杂度来遍历得到结果。而且这个方法因为不存在`fail fast`机制，所以大概率是不准确的。

**因此，需求才是第一位的。当你需要经常调用`size`的时候，可能换个数据结构才是更好的选择。**

最后，再提一个`ABA`问题：
> 这是通常只在 `lock-free`算法下暴露的问题。我前面说过 `CAS`是在更新时比较前值，如果对方只是恰好相同，例如期间发生了`A -> B -> A`的更新，仅仅判断数值是A，可能导致不合理的修改操作。针对这种情况，Java 提供了 `AtomicStampedReference`工具类，通过为引用建立类似版本号（stamp）的方式，来保证 CAS 的正确性。 —— 《Java核心技术面试精讲》
