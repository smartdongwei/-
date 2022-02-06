---
title: volatile的应用相关原理
tags: [java并发编程]
date: 2021-03-15 22:22:10
---

## 1: volatile的应用相关原理

​     volatile是轻量级的synchronized，它在多处理器开发中保证共享变量的"可见性"。当一个线程修改一个共享变量时，另外的一个线程也能读到这个修改的值，它不会引起上下文的切换和调度(<!--上下文切换CPU通过时间片分配算法来循环执行任务，当前任务执行一段时间后会切换到下一个任务，切换前保留上一个任务的状态，下次再切换回这个任务，任务从保存到再加载过程就是一次上下文切换-->)。但是它不能保证原子性。

### 1.1: 内存模型相关概念

​     Java线程之间的通信由Java内存模型(JMM)控制，JMM决定一个线程对共享变量的修改何时对另外一个线程可见。JMM定义了线程与主内存的抽象关系：线程之间的变量存储在主内存(Main Memory)中，每个线程都有一个私有的本地内存(Local Memory)保存着共享变量的副本。本地内存是JMM的一个抽象概念，并不真实存在。

![image-20210508231050871](E:\study\images\image-20210508231050871.png)

如果线程A与线程B通信：

  1.线程A先把本地内存A中更新过的共享变量刷写到主内存中。

   2.线程B到主内存中读取线程A更新后的共享变量  

​    计算机在运行程序时，每条指令都是在CPU中执行的，在执行过程中势必会涉及到数据的读写。我们知道程序运行的数据是存储在主存中，这时就会有一个问题，读写主存中的数据没有CPU中执行指令的速度快，如果任何的交互都需要与主存打交道则会大大影响效率，所以就有了CPU高速缓存。CPU高速缓存为某个CPU独有，只与在该CPU运行的线程有关。

   **有了CPU高速缓存虽然解决了效率问题，但是它会带来一个新的问题：数据一致性。**在程序运行中，会将运行所需要的数据复制一份到CPU高速缓存中，在进行运算时CPU不再也主存打交道，而是直接从高速缓存中读写数据，只有当运行结束后才会将数据刷新到主存中。

  **解决方案就是 通过缓存一致性协议，它确保每个缓存中使用的共享变量的副本是一致的。**



###  1.2: volatile实现原理

​    volatile修饰的共享变量进行写操作时会增加了一个Lock前缀的指令，该指令会引起以下2种事情：(1)Lock前缀指令会引起处理器缓存回写到内存。(2)一个处理器的缓存回写到内存会导致其他CPU缓存的该共享变量内存地址的数据无效。

​    这样就保证了多个处理器的缓存是一致的，当发现自己的缓存行对应的内存地址杯修改，就会将当前处理器缓存行设置无效状态，当处理器对这个数据进行修改操作的时候会重新从主内存中把数据读取到缓存里。



### 1.3 使用场景

  **volatile经常用于两个场景：状态标记、double check**

####    1.3.1 状态标记

```java
volatile boolean flag = false;

    while(!flag){
       doSomething();
    }

    public void setFlag() {
       flag = true;
    }

    volatile boolean inited = false;
    //线程1:
    context = loadContext();  
    inited = true;            

    //线程2:
    while(!inited ){
    sleep()
    }
    doSomethingwithconfig(context);
```

####    1.3.2 double check

   

```java
public class Singleton{
   private volatile static Singleton instance = null;
   private Singleton() {}
   public static Singleton getInstance() {
       if(instance==null) {
           synchronized (Singleton.class) {
               if(instance==null)
                   instance = new Singleton();
          }
      }
       return instance;
  }
}
```

 如果没有 volatile，可能存在的问题。instance = new Singleton();的执行并非原子性的，主要分为以下3步：

1. 分配一块内存M
2. 将M的地址赋值给instance变量
3. 最后再内存M上初始化Sigleton对象

​    我们假设线程A先执行getInstance()方法，当执行完指令2时恰好发生了线程切换，切换到了线程B上；如果此时线程B也执行getInstance()方法，那么线程B会发现instance != null，所以直接返回instance，而此时instance还没有初始化过的，这个时候访问instance的成员变量将可能触发空指针异常。详见下图：

![image-20210508234557339](E:\study\images\image-20210508234557339.png)

加上volatile关键字，为什么可以解决上述的问题，首先我们需要理解volatile的功能，主要包括两方面：

​    **保持内存的可见性。**因为JVM内存模型分为工作内存和主内存，主内存通常是我们的物理内存，而工作内存通常是CPU缓存，在多核CPU情况下，线程的工作内存往往可能处于不同的CPU上，所以存在内存不可见的问题。volatile修饰的变量，在读取时，要求JVM强制从主内存读取，在写入时，要求JVM强制同步到主内存中，也就保证了在不同的线程上读取到的变量都是实时的。
​     **防止指令重排。**在JVM对于指令优化需要遵循happen-before原则，其中有一条volatile原则：对一个变量的写操作先行发生于后面对这个变量的读操作。
​     那volatile是为了保证内存可见性吗？显然不是，因为instance被声明为static，所以变量会保存在方法区，这对于所有的线程都是共享的，不存在可见性问题。**加上volatile的作用主要是防止指令重排，主要是说，无论什么情况，对于volatile变量的写操作必须在完成后才能读取，所以不会出现未完成构造就读取的情况。**



### 1.4 : 缺点

​    volatile并不能保证原子性，如果共享变量时一般使用 java.util.concurrent里头的AtomicInteger类。

​    private static AtomicInteger count=new AtomicInteger(0);