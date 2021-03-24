---

---

## 原子操作

一个操作内的各个子操作要么都成功，要么都失败。

JDK中的原子类，是通过`Unsafe`类的native方法的`CAS`操作来保证原子性的

```java
java.util.concurrent.atomic.AtomicBoolean
java.util.concurrent.atomic.AtomicInteger
java.util.concurrent.atomic.AtomicLong
java.util.concurrent.atomic.AtomicReference
java.util.concurrent.atomic.AtomicStampedReference
java.util.concurrent.atomic.LongAdder
// ...   
```



## volatile

可见性：JMM，缓存一致，被`volatile`修饰的变量，被线程访问时，某个线程修改这个变量时，主存中的共享变量的变化会通知各个线程将主存的变量同步到当前线程。

有序性：被`volatile`修饰的变量，禁止指令重排序。

原子性：不保证。

> 双重检验的单例实现，volatile解决了什么问题，只用synchronized关键字是否可以，为什么不可以？
>
> 解决了指令有序性和线程可见性；
>
> 在并发不是巨大的情况下，volatile可加可不加，因为对象的new操作虽然不是原子性的，但是发生指令重排的概率非常小。



## JMM

Java Memory model

原子性、有序性、可见性



## CAS实现原理

CPU的原子操作指令CAS(compareAndSwap)。利用CAS操作原理，修改对象的属性的`内存偏移地址`来达到修改对象属性并保证原子。

```basic
lock cpmxchg; #汇编指令，保证操作的原子性
```

CAS 被认为是一种乐观锁。`synchronized`升级为重量级锁（悲观锁）。悲观锁会因线程一直阻塞导致系统上下文切换，系统的性能开销大。

缺点：CAS保证线程安全，需要自旋，如果**自旋的次数过多**也会浪费CPU资源。可以考虑 LongAdder 来解决，LongAdder 以空间换时间的方式，来解决 CAS 大量失败后长时间占用 CPU 资源，加大了系统性能开销的问题，但不保证实时性。

ABA问题，可以使用带版本的原子类，如`AtomicStampedReference`



## 锁的升级过程

对象头的MarkWord:

<img src="/Users/jinhsu/Github/sharit-projects/sharit-blog/docs/.images/jvm-010.png" style="zoom:50%;" />

`new`-->`无锁`-->`偏向锁`-->`轻量级`-->`重量级`

1. 无锁：刚刚new出来的对象，还没有线程去访问它。锁标志位为**01**
2. 当new出来的对象被第一个线程访问时，这个对象的对象头会记录当前线程的ID，即加上了偏向锁。下一次如果还是这个线程（根据线程ID比较）访问这个对象时，就直接允许其访问，即偏向了这个线程。锁标志位为**01**
3. 如果下一次有其他的线程访问这个对象，那么锁就由偏向锁升级为轻量级锁（也称自旋锁：自旋+CAS）。锁标志位位**00**
4. 当轻量级的锁自旋次数超过10次（默认，但也会有自适应的自旋逻辑），升级到重量级锁（操作系统调度的锁）。锁标志位**10**
5. 当这个对象没有任何对象指向时，就被GC标记。锁标志位为**11**

> 锁的升级过程不可逆，但是偏向锁状态可以被重置为无锁状态

`synchronized`代码块同步在需要同步的代码块开始的位置插入`monitorentr`指令，在同步结束的位置或者异常出现的位置插入`monitorexit`指令；JVM要保证`monitorentr`和`monitorexit`都是成对出现的，任何对象都有一个monitor与之对应，当这个对象的monitor被持有以后，它将处于锁定状态。



## **锁消除**

Java虚拟机在JIT编译时(可以简单理解为当某段代码即将第一次被执行时进行编译，又称即时编译)，通过对运行上下文的扫描，经过**逃逸分析**，去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间。

基于**逃逸分析**的优化

当判断出对象不发生逃逸时，编译器可以使用逃逸分析的结果作一些代码优化

- **将堆分配转化为栈分配。**如果某个对象在子程序中被分配，并且指向该对象的指针永远不会逃逸，该对象就可以在分配在栈上，而不是在堆上。在有垃圾收集的语言中，这种优化可以**降低垃圾收集器运行的频率**。

- **同步消除。**如果发现某个对象只能从一个线程可访问，那么在这个对象上的操作可以**不需要同步**。

- **分离对象或标量替换。**如果某个对象的访问方式不要求该对象是一个连续的内存结构，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。

    > 什么是标量？标量是指一个无法再分解成更小的数据的类型。`Java`中的原始的数据类型就是标量。相对的，还可以进行分解的数据叫做聚合量。
    >
    > `Java`中的对象就是聚合量，因为它还可以被分解成其他聚合量和标量。
    >
    > 如果经过逃逸分析，发现一个对象不会被外界访问的话，就会把这个对象拆解成若干个其中包含的成员变量来代替。 这个过程就是标量替换。



## 进程和线程

进程和线程的区别

> 进程：资源分配和系统调度的基本单位
> 线程：程序执行的基本单位

多线程编程注意事项

> 加锁：公共资源的线程安全性；
>
> 死锁：互相抢占对象的资源；
>
> 效率：线程间频繁切换执行、创建销毁换



## 线程的状态

<img src="/Users/jinhsu/Github/sharit-projects/sharit-blog/docs/.images/juc-001.png" alt="juc-001" style="zoom:50%;" />

sleep、wait、yield、join方法的区别及应用场景

- sleep 方法是属于 Thread 类中的，sleep 过程中线程**不会释放锁**，只会**阻塞线程**，让出cpu给其他线程，但是他的监控状态依然保持着，当指定的时间到了又会自动恢复运行状态，**可中断**，sleep 给其他线程运行机会时不考虑线程的优先级，因此**会给低优先级的线程以运行的机会**
- wait 方法是属于 Object 类中的，wait 过程中线程会释放对象锁，只有当其他线程调用 notify 才能唤醒此线程。wait 使用时必须先获取对象锁，即必须在 synchronized 修饰的代码块中使用，那么相应的 notify 方法同样必须在 synchronized 修饰的代码块中使用，如果没有在synchronized 修饰的代码块中使用时运行时会抛出IllegalMonitorStateException的异常
- 和 sleep 一样都是 Thread 类的方法，都是暂停当前正在执行的线程对象，不会释放资源锁，和 sleep 不同的是 yield方法并不会让线程进入阻塞状态，而是让线程重回就绪状态，它只需要等待重新获取CPU执行时间，所以执行yield()的线程有可能在进入到可执行状态后马上又被执行。还有一点和 sleep 不同的是 yield 方法只能使同优先级或更高优先级的线程有执行的机会
- 等待调用join方法的线程结束之后，程序再继续执行，一般用于**等待异步线程执行完结果之后才能继续运行的场景**。例如：主线程创建并启动了子线程，如果子线程中需要进行大量耗时运算计算某个数据值，而主线程要取得这个数据值才能运行，这时就要用到 join 方法了



## 可重入锁

广义上的可重入锁指的是可重复可递归调用的锁，在外层使用锁之后，在内层仍然可以使用，并且不发生死锁（前提得是同一个对象或者class），这样的锁就叫做可重入锁。ReentrantLock和synchronized都是可重入锁。



synchronized与Lock的区别

|          | synchronized                                                 | Lock                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 存在层次 | java的关键字，在jvm层面上                                    | 是一个类                                                     |
| 锁的释放 | 1、以获取锁的线程执行完同步代码，释放锁<br />2、线程执行发生异常，jvm会让线程释放锁 | 在finally中必须释放锁，不然容易造成线程死锁                  |
| 锁的获取 | 假设A线程获得锁，B线程等待，如果A线程阻塞，B线程会一直等待   | 分情况而定，lock有多个锁获取的方法，可以尝试获得锁，线程可以不用功一直等待 |
| 锁状态   | 无法判断                                                     | 可以判断                                                     |
| 锁类型   | 可以重入，不可以中断，非公平                                 | 可重入 可以判断 可公平                                       |
| 性能     | 少量同步                                                     | 大量同步                                                     |



## 公平和非公平锁

- 是指多个线程按照申请锁的顺序来获取锁，类似排队打饭，先来后到。
- 非公平锁就是新线程一上来就尝试去获取锁，没有获取到才进行等待队列排序。在高并发的情况下，有可能会造成优先级反转或者饥饿现象



## ReentrantLock

可重入锁，默认无参构造是非公平锁，带参数true为公平锁。

```java
public ReentrantLock() {
    sync = new NonfairSync();
}
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```



## AQS

### **AbstractQueuedSynchronizer**

锁是面向使用者的，定义了用户调用的接口，隐藏了实现细节；AQS是锁的实现者，屏蔽了同步状态管理，线程的排队，等待唤醒的底层操作。锁是面向使用者，AQS是锁的具体实现者，这里使用了模板方法的设计模式。背后复杂的线程排队，线程阻塞/唤醒，如何保证线程安全，都由AQS为我们完成了。

- state变量

    使用int类型的volatile变量维护同步状态(state)

    围绕state提供锁的两种操作“获取”和“释放=0”； 读锁与写锁区分 65535= 2^16-1

- 内置的同步队列 CLH，双端双向列表

    FIFO队列存放阻塞的等待线程，来完成线程的排队执行；

    封装成Node，Node维护一个prev引用和next引用，实现双向链表

    AQS维护两个指针，分别指向队列头部head和尾部tail

```java
lock() {
	if CAS抢占成功 返回true // 这里就是和公平锁的区别
	else 
		acquire(1)
			tryAcquire && acquireQueued(addWorker(独占模式))
}

tryAcquire() {
	if state == 0 CAS抢占 // 这里就是和公平锁的区别 
	else if 抢占的线程是当前线程 state+1 setState // 重入
	else 抢占失败
}

addWorker() {
	// 用for(;;)来进行下面2次操作
	// CLH队列为空：创建一个空节点,设为head节点；将当前节点设为tail节点
	// CLH不为空：直接将当前节点加到队尾，其他处理
}

acquireQueued() {
	// 自旋式抢占
	// 当前节点的前节点为head：尝试抢占，抢占成功，删除head节点，当前节点设为head
	// 当前节点的前节点不为head | 抢占失败：判断是否需要park: 
	//										当前节点的前节点waitState=-1 返回true;
	// 				waitState>0，说明前节点取消了等待，直接从队列将其删除，并继续判断前一个节点，最后返回false
	// 							waitState==0为初始态，CAS设置waitState=-1,返回false
	//					不要park继续自旋
	//					需要park：将当前线程park, LockSupport.park(),当前线程自旋阻塞，等待unlcok的unpark
}

unlock() {
	release(1); // 释放一个锁，unpark一个线程
}

release() {
    tryRelease() 成功 unparkSuccessor(head)
}

tryRelease() {
	int c = state-1;
    // 当前线程不是AQS独占线程，抛出异常
    // 
    free = false
    c == 0 {free= true 清空AQS的独占线程}
    setState(c);
    返回free
}

unparkSuccessor(head) {
    // head的waitState<0: CAS设置为0
    // head下一个节点取消等待或者为null:继续下一个节点
    // 下个节点t不为空，unpark节点t的线程
}

```



### **分为独占锁、共享锁**

继承`AbstractQueuedSynchronizer`并重写指定的方法，独占式如`ReentrantLock`，`ReentrantReadWriteLock`，共享式如`Semaphore`，`CountDownLatch`，`ReentrantReadWriteLock`，`ThreadPoolExecutor.Worker`等。

要实现一个独占锁，那就去重写或封装`tryAcquire`，`tryRelease`方法：

Acquire：tryAcquire(arg)、addWaiter(入队)、acquireQueued(循环获取锁，失败则挂起`shouldParkAfterFailedAcquire`)

Release：tryRelease(arg)、检查waitStatus、unparkSuccessor(h);//唤醒后继结点

要实现共享锁，就去重写`tryAcquireShared`，`tryReleaseShared`

acquireShared：tryAcquireShared、doAcquireShared(arg)(setHeadAndPropagate);(尝试获取（state+1），入队、等待唤醒)

realseShared：tryRealseShared、doReleaseShared()(CAS下的 unparkSuccessor(h));//state-1并判断、唤醒后继



> UTL（用户线程），KLT（内核线程）
>
> JVM使用的是KLT



## 线程池

### 4大拒绝策略：

```java
/**
 * 线程的 4大拒绝策略
 * //拒绝策略，超过最大承载，就抛出异常
 * 1、new ThreadPoolExecutor.AbortPolicy() // 抛出异常
 * //哪里来的哪里去，这个超出最大承载，谁（线程）拒绝，谁就去执行
 * 2、new ThreadPoolExecutor.CallerRunsPolicy(); //执行了线程的run() 方法
 * //队列满了，丢掉任务,不会抛出异常
 * 3、new ThreadPoolExecutor.DiscardPolicy()  // 不做任何操作，即丢掉任务
 * //队列满了，不会抛出异常，尝试和第一个开始的线程去竞争，竞争失败就丢掉
 * 4、new ThreadPoolExecutor.DiscardOldestPolicy()  // 等待队列poll()挤出最老的，并执行新线程
 */
```



### 7大参数

```java
public ThreadPoolExecutor(int corePoolSize,// 核心线程数
                          int maximumPoolSize, // 最大线程数
                          long keepAliveTime, // 空闲线程销毁前的等待时间
                          TimeUnit unit, // 时间单位
                          BlockingQueue<Runnable> workQueue, // 最大线程数满后，进入的等待队列
                          ThreadFactory threadFactory, // 线程创建工厂，默认即可
                          RejectedExecutionHandler handler)  // 拒绝策略
```



### 4大方法

Executors.newSingleThreadExecutor() 

适用于多个任务顺序执行的场景

```java
// 单线程的线程池
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1, // 核心线程数和最大线程数为1
                                0L, TimeUnit.MILLISECONDS, // 新线程直接进不来
                                new LinkedBlockingQueue<Runnable>())); // LinkedBlockingQueue等待队列
    // DefaultThreadFactory
    // AbortPolicy 直接抛出异常
}
```



Executors.newFixedThreadPool(int nThreads)

任务量比较固定但耗时长的任务

```java
// 固定线程数量的线程池
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
    // DefaultThreadFactory
    // AbortPolicy 直接抛出异常
}
```



Executors.newCachedThreadPool()

适合任务量大但耗时少的任务

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE, // 最大线程数量
                                  60L, TimeUnit.SECONDS, // 60s
                                  new SynchronousQueue<Runnable>()); // 同步队列
    // DefaultThreadFactory
    // AbortPolicy 直接抛出异常
}
```



Executors.newScheduledThreadPool()

适用于执行定时任务和具体固定周期的重复任务

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
```



### 工作原理

1. 在创建了线程池后，等待提交过来的任务请求。

2. 当调用 execute() 方法添加一个请求任务时，线程池会做如下判断

    - 如果正在运行的线程数量小于 corePoolSize，那么马上创建线程运行这个任务；

    - 如果正在运行的线程数量大于或等于 corePoolSize，那么将这个任务放入队列；

    - 如果这时候队列满了且正在运行的线程数量还小于 maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务；

    - 如果队列满了且正在运行的线程数量大于或等于 maximumPoolSize，那么线程池会启动拒绝策略来执行。

3. 当一个线程完成任务时，它会从队列中取下一个任务来执行。

4. 当一个线程无事可做，超过一定的时间(keepAliveTime) 时，线程池会判断：

    如果当前运行的线程数大于 corePoolSize，那么这个线程就被停掉。所以线程池的所有任务完成后，它最终会收缩到 corePoolSize 的大小。

### 最大线程数量

1. CPU密集型。 电脑是几核就是几，可以保持CPU的效率最高。

    > 计算密集型 = Ncpu（常出现于线程中：复杂算法）

2. IO 密集型 > 判断程序中十分耗 IO的线程

    > IO密集型 = 2Ncpu（可以测试后自己控制大小，2Ncpu一般没问题）（常出现于线程中：数据库数据交互、文件上传下载、网络数据传输等等）



## Threadlocal



线程切换的上下文要装载什么