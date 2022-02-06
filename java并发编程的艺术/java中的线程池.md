1 : 线程的相关启动  

阿里提供的工具 arths 内存监控   Collectors

4个线程池的功能 以及相关参数



2：加锁  重量锁 轻量锁  volalitate 和 



## 1: java常用四大线程池用法以及ThreadPoolExecutor详解

为什么用线程池？

- 1.创建/销毁线程会伴随着系统开销，过于频繁的创建和销毁线程，会很大程度上影响处理效率

- 2.线程并发数量过多，抢占系统资源从而导致阻塞

- 3.对线程进行一些简单的管理

  

### 1.1 ThreadPoolExecutor提供的构造函数

```java
public ThreadPoolExecutor(int corePoolSize,                         
                          int maximumPoolSize,                         
                          long keepAliveTime,                      
                          TimeUnit unit,                         
                          BlockingQueue<Runnable> workQueue)
```

相关参数的含义如下：

- **int corePoolSize:** 该线程池中核心线程数最大值

   线程池新建线程时，如果当前线程总数小于corePoolSize，则新建的是核心线程，如果超过，则新建的是非核心线程。核心线程默认情况下会一直存活在线程池中，如果指定了ThreadPoolExecutor的allowCoreThreadTimeOut这个属性那么核心线程如果不干活(闲置状态)的话，超过一定时间就会被销毁。

- **int maximumPoolSize:** 该线程池中线程总数最大值

​      线程总数 = 核心线程数 + 非核心线程数

- **long keepAliveTime:**该线程池中非核心线程闲置超时时间

   一个非核心线程，如果闲置时间超过这个参数所设定的时长，就会被销毁，如果设置allowCoreThreadTime

  = true，则会作用于核心线程。


- **TimeUnit unit:** keepAliveTime的单位
- **BlockingQueue<Runnable> workQueue:** 该线程池中的任务队列，维护着等待执行的Runnable对象

​      当所有的核心线程都在干活时，新添加的任务会被添加到这个队列中等待处理，如果队列满了，则新建非核心线程执行任务。常用的workQueue类型：

1. SynchronousQueue: 这个队列接收到任务时，会直接提交给线程处理，而不保留它，如果所有线程都在工作，则新建一个线程来处理这个任务，需要把maximumPoolSize这个参数设置成 Integer.MAX_VALUE。

2. LinkedBlockingQueue: 当接收到任务时，如果当前线程数小于核心线程数时，则新建核心线程处理任务，如果当前线程数等于核心线程数，则进入队列等待。由于这个队列没有最大值限制，即所有超过核心线程数的任务都会被添加到队列中。

3. ArrayBlockingQueue: 可以限定队列的长度，如果当前线程数没有达到corePoolSize的值，则新建核心线程执行任务，如果达到，则入队等待，如果队列已满，则新建非核心线程执行任务，如果总线程数达到了最大值，并且队列也满了，则会发生错误。

4. DelayQueue: 队列中元素必须实现Delayed接口，这个队列接收到任务时，首先先入队，只有达到指定的延时时间，才会执行任务。

   

-   **ThreadFactory threadFactory**: 创建线程的⼯⼚ ，⽤于批量创建线程，统⼀在创建线程时设置⼀些参数，如是 否守护线程、线程的优先级等。如果不指定，会新建⼀个默认的线程⼯⼚。
- RejectedExecutionHandler handler: 拒绝处理策略，线程数量⼤于最⼤线程数就会采⽤拒绝处理策略



### 1.2 常见的四种线程池

####    1.2.1 可缓存线程池

**源码：**

```java
public static ExecutorService newCachedThreadPool(){
    return new ThreadPoolExecutor(0,
            Integer.MAX_VALUE,
            60L,
            TimeUnit.SECONDS,
            new SynchronousQueue<Runnable>());
    
}
```

**根据源码可以看出：**

  1: 这种线程池内部没有核心线程，线程的数量是没有限制的。

  2:在创建线程时，若有空闲线程则复用空闲线程，若没有则新建线程。

  3: 闲置的线程如果超过60s还不做事，就会被销毁。

**创建方式：**





## 2:合理的配置线程池

   任务特性主要有以下几个方面：

- 任务的性质：CPU密集型任务，IO密集型任务和混合型任务。
- 任务的优先级：高，中和低
- 任务的执行时间：长，中和短
- 任务的依赖性：是否依赖其他系统资源

   性质不同的任务可以用不同规模的线程池分开处理。CPU密集型热为奴应配置尽可能小的线程，如配置N(cpu)+1个线程的线程池。由于IO密集型任务线程并不是一直在执行任务，则应该配置尽可能多的线程，如2N(cpu)。可以通过 `Runtime.getRuntime().availableProcessors()`来获取当前设备的CPU个数。

​    优先级不同的任务可以使用优先级队列`PriorityBlockingQueue`来处理，它可以让优先级高的任务先执行。

​    执行时间不同的任务可以交给不同规模的线程池来处理，或者使用优先级队列。

​    依赖数据库的任务，线程数应该设置大一点，这样才能更好的利用CPU。

​    建议使用有界队列。

## 3:Executor框架的结构

​    Executor框架主要由3大部分组成：

- 任务。包括被执行任务需要实现的接口：Runnable接口或Callable接口。
- 任务的执行。包括任务执行机制的核心接口Executor,以及继承自Executor的ExecutorService接口。
- 异步计算的结果。包括接口Future和实现Future接口的FutureTask类。



### 3.1 FixedThreadPool详解

​    FixedThreadPool被称为可重用固定线程数的线程池，下面是其源代码：

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

​    FixedThreadPool的corePoolSize和maximumPoolSize都被设置为创建时的指定参数nThreads，当线程中的线程数大于corePoolSize，由于等待时间设置为0L，意味着多余的空闲线程会被立即终止。

### 3.2 SingleThreadExecutor详解

  SingleThreadExecutor是使用单个worker线程的Executor。

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

SingleThreadExecutor的corePoolSize和maximumPoolSize都被设置为1。其他参数与FixedThreadPool相同。

### 3.3 CachedThreadPool详解

​    CachedThreadPool是一个会根据需要创建新线程的线程池。

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

CachedThreadPool的corePoolSize被设置为0，意味着如果主线程提交任务的速度高于maximumPool中线程处理任务的速度时，CachedThreadPool会不断创建新线程。极端情况下会因为创建过多线程而耗尽CPU和内存资源。



### 3.4 ScheduledThreadPoolExecutor详解

  它主要用来给定延迟之后运行任务，或者定期执行任务。



### 3.5 FutureTask详解

​    代表异步计算



## 4：拒绝策略

​    所谓拒绝策略，就是当线程池满了、队列也满了的时候，我们对任务采取的措施。或者丢弃、或者执行、或者其它内容。jdk自带的4种拒绝策略，如下：

(1)ThreadPoolExecutor.AbortPolicy:  丢弃任务并抛出RejectedExecutionException异常。

(2)ThreadPoolExecutor.DiscardPolicy: 丢弃任务，但是不抛出异常。

(3)ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新提交被拒绝的任务。

(4)ThreadPoolExecutor.CallerRunsPolicy：由调用线程（提交任务的线程）处理该任务

​    除了JDK默认提供的四种拒绝策略，我们可以根据自己的业务需求去自定义拒绝策略，自定义的方式很简单，直接实现RejectedExecutionHandler接口即可。

## 5: 相关面试题

### 5.1 **execute和submit的区别？**

​    execute适用于不需要关注返回值的场景，只需要将线程丢到线程池中去执行就可以了。submit方法适用于需要关注返回值的场景

### 5.2 如何配置核心线程数

**CPU密集型任务：**CPU密集指的是需要进行大量的运算，例如排序，一般没有什么阻塞。尽量使用较小的线程池，大小一般为CPU核心数+1。因为CPU密集型任务使得CPU使用率很高，若开过多的线程数，会造成CPU过度切换。

**IO密集型任务：**IO密集指的是需要进行大量的IO，例如文件上传与下载、网络请求等。阻塞十分严重，可以挂起被阻塞的线程，开启新的线程干别的事情。可以使用稍大的线程池，大小一般为CPU核心数*2。IO密集型任务CPU使用率并不高，因此可以让CPU在等待IO的时候有其他线程去处理别的任务，充分利用CPU时间。

当然，依据IO密集的程度，可以在两倍的基础上进行相应的扩大与缩小。以上只是一个初步的策略，或者说先定一个初始数值，接着需要进行压测，来调整最大线程数。压测的同时，可以监控线程池状态，并且动态改变线程池的参数。

