# Java线程池
## 什么是线程池
若干个可执行的线程放入一个池（容器）中，需要的时候从池中获取线程不用自行创建，使用完毕不需要销毁线程而是放回池中，从而减少创建和销毁线程对象的开销（即帮助管理线程的创建和销毁的一种技术）
## 线程池的优点
1. 第一：降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
2. 第二：提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行
3. 第三：提高线程的可管理性。使用线程池可以进行统一的分配，调优和监控。
## 为什么要使用线程池
多线程技术主要解决处理器单元内多个线程执行的问题，它可以显著减少处理器单元的闲置时间，增加处理器单元的吞吐能力。    
假设一个服务器完成一项任务所需时间为：T1 创建线程时间，T2 在线程中执行任务的时间，T3 销毁线程时间。
如果：T1 + T3 远大于 T2，则可以采用线程池，以提高服务器性能。
## 常见的线程池
- newSingleThreadExecutor：
单线程的线程池，即线程池中每次只有一个线程工作，单线程串行执行任务
- newFixedThreadExecutor(n)：
固定数量的线程池，没提交一个任务就是一个线程，直到达到线程池的最大数量，然后后面进入等待队列直到前面的任务完成才继续执行
- newCacheThreadExecutor：（推荐使用）
可缓存线程池，当线程池大小超过了处理任务所需的线程，那么就会回收部分空闲（一般是60秒无执行）的线程，当有任务来时，又智能的添加新线程来执行。
- newScheduleThreadExecutor：大小无限制的线程池，支持定时和周期性的执行线程
## 线程池的组成部分
1. 线程池管理器（ThreadPool）：用于创建并管理线程池，包括 创建线程池，销毁线程池，添加新任务；
2. 工作线程（PoolWorker）：线程池中线程，在没有任务时处于等待状态，可以循环的执行任务；
3. 任务接口（Task）：每个任务必须实现的接口，以供工作线程调度任务的执行，它主要规定了任务的入口，任务执行完后的收尾工作，任务的执行状态等；
4. 任务队列（taskQueue）：用于存放没有处理的任务。提供一种缓冲机制。
## 线程池框架结构
![线程池框架](https://raw.githubusercontent.com/ValentineF/NoteBook/master/Picture/%E7%BA%BF%E7%A8%8B%E6%B1%A0%E6%A1%86%E6%9E%B6.png?token=APcBgT6zHsdvR3xIbWzw0m5-59ADQMY_ks5bFkVPwA%3D%3D)
一般还是利用Executors的静态方法来创建线程池
## 线程池的创建解析
```
new ThreadPoolExecutor(corePoolSize, maximumPoolSize,keepAliveTime, milliseconds,runnableTaskQueue, threadFactory,handler);

```
- corePoolSize（核心池的大小）：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads方法，线程池会提前创建并启动所有基本线程。

- maximumPoolSize（线程池最大大小）：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。

- keepAliveTime（线程活动保持时间）：线程没有任务执行时最多保持多久时间会终止（线程数量小于核心线程数时，不起作用，可以调用其他方法使得核心线程也被终止）

- runnableTaskQueue（阻塞队列）：用于保存等待执行的任务的阻塞队列。
    - ArrayBlockingQueue：基于数组结构的有界阻塞队列，此队列按 FIFO原则排序。

    - LinkedBlockingQueue：基于链表结构的阻塞队列，此队列按FIFO排序，吞吐量通常要高于 ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()使用了这个队列。

    - SynchronousQueue：不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列。

    - PriorityBlockingQueue：具有优先级的无界阻塞队列。

- ThreadFactory：用于设置创建线程的工厂

- RejectedExecutionHandler（饱和策略）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。以下是JDK1.5提供的四种策略。 
    - AbortPolicy（默认策略）：直接抛出异常。
    - CallerRunsPolicy：只用调用者所在线程来运行任务。
    - DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
    - DiscardPolicy：不处理，丢弃掉。
    - 可以自定义策略。如记录日志或持久化不能处理的任务。

## 线程池工作流程
提交一个线程进入线程池使用的是execute();submit()方法的本质也是调用execute()

![提交任务工作流程](https://raw.githubusercontent.com/ValentineF/NoteBook/master/Picture/Java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E4%B8%BB%E8%A6%81%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.jpg?token=APcBgYPANyJiM1T4rE2CAqBdrUsjwZtgks5bFkVdwA%3D%3D)
## 线程池状态
```
volatile int runState;
static final int RUNNING    = 0;
static final int SHUTDOWN   = 1;
static final int STOP       = 2;
static final int TERMINATED = 3;
```
- 当创建线程池后，初始时，线程池处于RUNNING状态；
- 如果调用了shutdown()方法，则线程池处于SHUTDOWN状态，此时线程池不能够接受新的任务，它会等待所有任务执行完毕；
- 如果调用了shutdownNow()方法，则线程池处于STOP状态，此时线程池不能接受新的任务，并且会去尝试终止正在执行的任务；
- 当线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为TERMINATED状态。
## 线程池的关闭
ThreadPoolExecutor提供了两个方法，用于线程池的关闭，分别是shutdown()和shutdownNow()，其中：
- shutdown()：不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务
- shutdownNow()：立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务
## 参考资料
[Java并发编程：线程池的使用](https://www.cnblogs.com/dolphin0520/p/3932921.html)

[Java-线程池专题 (美团面试题)](https://www.cnblogs.com/aspirant/p/6920418.html)

[聊聊并发（三）Java线程池的分析和使用](http://ifeve.com/java-threadpool/)