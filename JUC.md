## 线程池



线程池顶级接口Executor，ThreadPoolExecutor是这个接口的实现类



### 5个参数



int corePoolSize：核心线程数

int maximumPoolSize: 该线程池中线程总数大小，核心线程数+非核心线程数

long keepAliveTime:非核心线程闲置超时时长，如果处于闲置状态超过该值，就会被销毁

TimeUnit unit：keepAliveTime的单位

BlockingQueue workQueue：阻塞队列，维护Runnable任务对象



非必要参数

THreadFactory threadFactory创建线程的工程

RejectedExecutionHandler handler 拒绝处理策略



1.默认拒绝策略，丢弃任务抛出异常

2.丢弃新来的任务，但不抛出异常

3.丢弃队列头部的任务，然后重新产生执行，如果再次失败，重复此过程（新来的一定会执行吧）

4.由调用线程处理该任务





### 线程池状态



RUNNING：线程池创建后处于RUNNING状态

SHUTDOWN：调用shutdown()方法，线程池不能接受新的任务，清除一些空闲的worker,会等待阻塞队列的任务完成（不是马上关闭，完成所有剩下的任务再关闭）

STOP：调用shutdownNow()方法，线程池不能接受新的任务，中断所有线程，阻塞队列中没有被执行的任务全部丢弃

TIDYING：当所有的任务已终止，ctl记录的任务数量为0

TERMINATED：线程池处于TIDYING，执行完terminated（）方法后



### 线程池执行流程

1.当前线程数小于corePoolSize，则调用addWorker创建核心线程执行任务

2.如果不小于corePoolSize，则将任务添加到workQueue队列

3.如果放入workQueue失败，则创建非核心线程执行任务

4.如果创建非核心线程失败，就执行拒绝策略

### 常见线程池

newCachedTHredPool 不创建核心线程，使用同步队列，如果已有任务再等待，入列操作就会阻塞



优点：当需要执行很多短时间的任务时，caheThreadPool的线程复用率比较高，会显著提高性能



newFixedTHreadPool ：只创建核心线程

在没有任务的情况下，FixedTHreadPool占用资源更多，但是几乎不会触发拒绝策略，队列是阻塞队列



newSingleThreadExecutor只有一个核心线程



newScheduledThreadPool 创建一个定长线程池，支持定时即周期性任务执行





## volatile

保证变量的内存可见性

禁止volatile变量与普通变量重排序



禁止重排序原理



在每个volatile写操作前插入一个StoreStore屏障

在每个volatile写操作后插入一个StoreLoad屏障

在每个volatile读操作后插入一个LoadLoad屏障

在每个volatile读操作后再插入一个LoadStore屏障





storestore屏障，在store2以及后续写入操作被刷出前，保证store1的写入操作对其他处理器可见

storeLoad屏障，在load2以及后续所有读操作执行前，保证store1的写入对所有处理器可见

loadload屏障，在load2以及后续所有读操作执行前，保证Load1读取的数据读取完毕

loadstore屏障，在store2以及后续写入操作执行前，保证Load1读取的数据读取完毕



## 通信工具类

#### CountDownLatch

减法计数器，减到0放行

#### CyclicBarrier

加法计数器，到指定线程数放行

#### Semaphore

信号量 pv操作





## 几种锁



1.无锁状态

2.偏向锁：大多数情况下锁不仅不存在多线程竞争，而且总是由同一线程多次获得，



偏向锁在资源无竞争情况下取消同步语句，连CAS操作都不用做了，提高了程序的程序的运行性能



3.轻量级锁：



多个线程在不同时段获取同一把锁，即不存在锁竞争，也就没用线程阻塞。



CASz自旋获取锁，

多次CAS自旋，升级为重量级锁



4.重量级锁：



依赖于操作系统的互斥量mutex实现