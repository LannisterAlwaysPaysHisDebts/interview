# 通道与协程
> Do not communicate by sharing memory; instead, share memory by communicating.
> 
> “不要以共享内存的方式来通信，相反，要通过通信来共享内存。”
## 基础理论
### 进程、线程、协程的区别? 
定义上的区别:
- 进程是程序在操作系统中的一次执行过程，系统进行资源分配和调度的一个独立单位。
- 线程是进程的一个执行实体，是CPU调度和分派的基本单位，它是比进程更小的能独立运行的基本单位。
- 协程:独立的栈空间，共享堆空间，调度由用户自己控制，本质上有点类似于用户级线程，这些用户级线程的调度也是自己实现的。一个线程上可以跑多个协程，协程是轻量级的线程。

进程在执行过程中拥有独立的内存单元，而多个线程共享内存。
每个独立的线程有一个程序运行的入口、顺序执行序列和程序的出口。线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。
从逻辑角度来看，多线程的意义在于一个应用程序中，有多个执行部分可以同时执行。但操作系统并没有将多个线程看做多个独立的应用，来实现进程的调度和管理以及资源分配。这就是进程和线程的重要区别。

进程线程都有内核态用户态的切换，协程只在用户态调度。对linux底层调度器来说，无论是线程还是进程，调度器只认task_struct对象，所以进程调度和线程调度是一回事,只是上下文切换时保存的内容不同.

### 堆与栈的区别
### 进程有哪几种状态，它们之间互相是怎么转换的?
就绪： 当进程已分配到除CPU以外的所有必要的资源，只要获得处理机便可立即执行，这时的进程状态称为就绪状态。
执行：当进程已获得处理机，其程序正在处理机上执行，此时的进程状态称为执行状态。
阻塞：正在执行的进程，由于等待某个事件发生而无法执行时，便放弃处理机而处于阻塞状态。引起进程阻塞的事件可有多种，例如，等待I/O完成、申请缓冲区不能满足、等待信件(信号)等。

### 怎么实现进程之间和线程之间的同步? 简述进程通信的各种方法
进程: 信号量(PV操作,P申请资源,V释放资源);管道;
线程: 临界区;互斥量;共享内存;消息队列;网络;

## goroutine
### 如果在一个goroutine里面发生panic，这个错误能捕捉吗? 框架如何实现协程自动捕捉panic
协程的panic只有在协程里面才能用recover()捕捉，主协程无法捕捉(会导致整个进程崩溃)

### 如何控制 gorotine 的执行顺序
`sync.Cond`; 例如: 设置一个信号,比如全局变量或者channel, 其他协程使用`cond.Wait`方法阻塞,等待某个协程执行`cond.Broadcast`或者`cond.Signal`通知执行;

## channel
### 向一个已经关闭了的(buffer/unbuffer)channel写入数据/读取数据会怎样? 怎么判断一个channel已经关闭?
1. 对一个关闭的通道再发送值就会导致panic。
2. 对一个关闭的通道进行接收会一直获取值直到通道为空。
3. 对一个关闭的并且没有值的通道执行接收操作会得到对应类型的零值。
4. 关闭一个已经关闭的通道会导致panic。

## lock
### lock-free
https://blog.csdn.net/nawenqiang/article/details/83027222 

### 解释下乐观锁悲观锁
悲观锁是阻塞同步;乐观锁是非阻塞同步;

### 关于同步锁，下面说法正确的是（）
A. 当一个goroutine获得了Mutex后，其他goroutine就只能乖乖的等待，除非该goroutine释放这个Mutex
B. RWMutex在读锁占用的情况下，会阻止写，但不阻止读
C. RWMutex在写锁占用情况下，会阻止任何其他goroutine（无论读和写）进来，整个锁相当于由该goroutine独占
D. Lock()操作需要保证有Unlock()或RUnlock()调用与之对应

ABC
