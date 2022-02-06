---
title: CAS的相关概念
tags: [java并发编程]
date: 2021-03-11 19:32:10
---

# CAS的相关概念

​     CAS是乐观锁，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。CAS是无锁时解决冲突的办法，全称是Compare And Swap即比较和交换。CAS机制当中使用了3个基本操作数：**内存地址V**，**旧的预期值A**，**要修改的新值B**。更新一个变量的时候，只有当变量的预期值A和内存地址V当中的实际值相同时，才会将内存地址V对应的值修改为B。

  1.在内存地址V当中，存储着值为10的变量。

<img src="E:\study\images\image-20210514234411290.png" alt="image-20210514234411290" style="zoom:50%;" />

2.此时线程1想要把变量的值增加1。对线程1来说，旧的预期值A=10，要修改的新值B=11。

<img src="E:\study\images\image-20210514234349718.png" alt="image-20210514234349718" style="zoom:50%;" />

3.在线程1要提交更新之前，另一个线程2抢先一步，把内存地址V中的变量值率先更新成了11。

<img src="E:\study\images\image-20210514234454249.png" alt="image-20210514234454249" style="zoom:50%;" />

4.线程1开始提交更新，首先进行A和地址V的实际值比较（Compare），发现A不等于V的实际值，提交失败。

<img src="E:\study\images\image-20210514234738387.png" alt="image-20210514234738387" style="zoom:50%;" />

5.线程1重新获取内存地址V的当前值，并重新计算想要修改的新值。此时对线程1来说，A=11，B=12。这个重新尝试的过程被称为自旋。

<img src="E:\study\images\image-20210514234852473.png" alt="image-20210514234852473" style="zoom:50%;" />

6.这一次比较幸运，没有其他线程改变地址V的值。线程1进行Compare，发现A和地址V的实际值是相等的。

<img src="E:\study\images\image-20210514234931531.png" alt="image-20210514234931531" style="zoom:50%;" />

  根据上述含义，可以理解为无阻塞多线程争抢资源的模型。

## 1: 相关原理

  **在java中的cas是怎么实现的：**

- java 的 cas 利用的的是 unsafe 这个类提供的 cas 操作。
- unsafe 的cas 依赖了的是 jvm 针对不同的操作系统实现的 Atomic::cmpxchg
- Atomic::cmpxchg 的实现使用了汇编的 cas 操作，并使用 cpu 硬件提供的 lock信号保证其原子性

## 2: CAS的应用

   **自旋锁：**在lock()的时候，一直while()循环，直到 cas 操作成功为止。

```java
public class SpinLock {

  private AtomicReference<Thread> sign =new AtomicReference<>();

  public void lock(){
    Thread current = Thread.currentThread();
    while(!sign .compareAndSet(null, current)){
    }
  }

  public void unlock (){
    Thread current = Thread.currentThread();
    sign.compareAndSet(current, null);
  }
}
```

  **AtomicInteger 的 incrementAndGet()**：与自旋锁有异曲同工之妙，就是一直while，直到操作成功为止。

```java
    public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            var5 = this.getIntVolatile(var1, var2);
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
    }
```

  **令牌桶限流器：**就是系统以恒定的速度向桶内增加令牌。每次请求前从令牌桶里面获取令牌。如果获取到令牌就才可以进行访问。当令牌桶内没有令牌的时候，拒绝提供服务。我们来看看 `eureka` 的限流器是如何使用 CAS 来维护多线程环境下对 token 的增加和分发的。

```java
public class RateLimiter {

    private final long rateToMsConversion;

    private final AtomicInteger consumedTokens = new AtomicInteger();
    private final AtomicLong lastRefillTime = new AtomicLong(0);

    @Deprecated
    public RateLimiter() {
        this(TimeUnit.SECONDS);
    }

    public RateLimiter(TimeUnit averageRateUnit) {
        switch (averageRateUnit) {
            case SECONDS:
                rateToMsConversion = 1000;
                break;
            case MINUTES:
                rateToMsConversion = 60 * 1000;
                break;
            default:
                throw new IllegalArgumentException("TimeUnit of " + averageRateUnit + " is not supported");
        }
    }

    //提供给外界获取 token 的方法
    public boolean acquire(int burstSize, long averageRate) {
        return acquire(burstSize, averageRate, System.currentTimeMillis());
    }

    public boolean acquire(int burstSize, long averageRate, long currentTimeMillis) {
        if (burstSize <= 0 || averageRate <= 0) { // Instead of throwing exception, we just let all the traffic go
            return true;
        }

        //添加token
        refillToken(burstSize, averageRate, currentTimeMillis);

        //消费token
        return consumeToken(burstSize);
    }

    private void refillToken(int burstSize, long averageRate, long currentTimeMillis) {
        long refillTime = lastRefillTime.get();
        long timeDelta = currentTimeMillis - refillTime;

        //根据频率计算需要增加多少 token
        long newTokens = timeDelta * averageRate / rateToMsConversion;
        if (newTokens > 0) {
            long newRefillTime = refillTime == 0
                    ? currentTimeMillis
                    : refillTime + newTokens * rateToMsConversion / averageRate;

            // CAS 保证有且仅有一个线程进入填充
            if (lastRefillTime.compareAndSet(refillTime, newRefillTime)) {
                while (true) {
                    int currentLevel = consumedTokens.get();
                    int adjustedLevel = Math.min(currentLevel, burstSize); // In case burstSize decreased
                    int newLevel = (int) Math.max(0, adjustedLevel - newTokens);
                    // while true 直到更新成功为止
                    if (consumedTokens.compareAndSet(currentLevel, newLevel)) {
                        return;
                    }
                }
            }
        }
    }

    private boolean consumeToken(int burstSize) {
        while (true) {
            int currentLevel = consumedTokens.get();
            if (currentLevel >= burstSize) {
                return false;
            }

            // while true 直到没有token 或者 获取到为止
            if (consumedTokens.compareAndSet(currentLevel, currentLevel + 1)) {
                return true;
            }
        }
    }

    public void reset() {
        consumedTokens.set(0);
        lastRefillTime.set(0);
    }
}
```

## 3 : CAS存在的问题

​    CAS虽然很高效的解决原子操作，但是CAS仍然存在三大问题。ABA问题，循环时间长开销大和只能保证一个共享变量的原子操作

1. **ABA问题**。因为CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。ABA问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么A－B－A 就会变成1A-2B－3A。**从Java1**.5开始JDK的atomic包里提供了一个AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。关于ABA问题参考文档: http://blog.hesey.net/2011/09/resolve-aba-by-atomicstampedreference.html
2. **循环时间长开销大**。自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。
3. **只能保证一个共享变量的原子操作**。当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了**AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。**

## **4: 相关示例：**

​     JUC就是java.util.concurrent包的简称。它有核心就是CAS与AQS。CAS是java.util.concurrent.atomic包的基础，如AtomicInteger、AtomicBoolean、AtomicLong等等类都是基于CAS。java.util.concurrent.atomic包下，一系列以Atomic开头的包装类。例如AtomicBoolean，AtomicInteger，AtomicLong，底层实现正是利用的CAS机制。

具体的相关信息请看文档 **Java中的13个原子操作类**。





## 5: Java中的13个原子操作类

​     java从JDK1.5开始提供了java.util.concurrent包，这个包的原子操作类提供了简单，高效，线程安全的更新一个变量的方式。Atomic包里面一共有13个类，属于4种类型的原子更新方式，分别是原子更新基本类型,原子更新数组，原子更新引用和原子更新属性。包里面的类都是使用Unsafe类实现的包装类。

### 5.1: 原子更新基本类型类

####   5.1.1: Atomic包主要提供了以下3个类：

AtomicBoolean：原子更新布尔类型。 AtomicInteger: 原子更新整形。 AtomicLong: 原子更新长整型。

AtomicInteger的常用方法如下：

- int addAndGet(int delta)： 以原子方式将输入的值与实例中的值相加，并返回结果。
- boolean compareAndSet(int expect, int update): 如果输入的数值等于预期值，则以原子方式将该值设置为输入的值。
- int getAndIncrement(): 以原子方式将当前值加1，注意，这里返回的是自增前的值。
- void lazySet(int newValue):  最终会设置成newValue,使用lazySet设置值后，可能会导致其他线程在之后一段时间内还是读取到旧的值。
- int getAndSet(int newValue)： 以原子方式设置为 newValue的值，并返回旧值。



#### 5.1.2 实现原子操作的原理

  以**getAndIncrement**为例子，查看其源码为：

```java
// 获取内存地址为var1+var2的变量值，并将该变量值加上var4
public final int getAndAddInt(Object object, long offset, int delta) {
    int v;
    do {
        // 通过对象和偏移量获取变量的值
        // 由于 volatile的修饰，所有的线程看到的v都是一样的
        v = this.getIntVolatile(object, offset);
    }while(!this.compareAndSwapInt(object, offset, v, v + delta));
    /**while中的 compareAndSwapInt 方式尝试修改v的值，具体地，该方法也会通过object和offset获取变量的值，如果这个值和v不一样，说明其它线程修改了object+offset地址处的值，此时该方法返回的值为false。
    如果这个值和v一样，说明没有其它线程修改object+offset地址处的值，此时可以把这个值修改为  v + delta，因为这个方法是原子操作，所以修改时不会被其它线程中断
    */
    return var5;
}
```

  在unSafe里面只提供了3中CAS方法：compareAndSwapObject，compareAndSwapInt和compareAndSwapLong，对于 AtomicBoolean是把Boolean转成整型，再进行compareAndSwapInt。

### 5.2: 原子更新数组

​    通过原子的方式更新数组里的某个元素，Atomic包里面提供了以下4个类：

- AtomicIntegerArray: 原子更新整型数组里的元素

  int addAndGet(int i, int delta) 以原子方式将输入值与数组中索引i的元素相加

  boolean compareAndSet(int i, int expect, int update) 如果当前值等于预期值，则以原子方式将数组位置i的元素设置成update值。

- AtomicLongArray: 原子更新长整型数组里的元素

- AtomicReferenceArray:原子更新引用类型数组里的元素

  **注意：**

  当数组通过构造方法传递进去时，AtomicIntegerArray会将当前数组复制一份，所以对数组元素进行修改时，不会影响传入的数组。

  ```java
  static int[] value = new int[]{1,2};
  static AtomicIntegerArray aiInt = new AtomicIntegerArray(value);
  
  public static void main(String[] args) {
      aiInt.getAndSet(0,3);
      System.out.println(aiInt.get(0));
      System.out.println(value[0]);
  
      // write your code here
  }
  ```

### 5.3: 原子更新引用类型

   如果原子更新多个变量，就需要使用这个原子更新引用类型提供的类。Atomic包提供了以下3个类：

- AtomicReference: 原子更新引用类型。
- AtomicReferenceFieldUpdater: 原子更新引用类型里的字段
- AtomicMarkableReference: 原子更新带有标记位的引用类型。可以原子更新布尔类型的标记位和引用类型。构造方法是AtomicMarkableReference(V initialRef, boolean initialMark)

```java
package com.company.wdwstudy;

import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicIntegerArray;
import java.util.concurrent.atomic.AtomicReference;

public class Main {

    public static AtomicReference<User> atomicReference = new
        AtomicReference<User>();

    public static void main(String[] args) {

        User user = new User("wang",11);
        atomicReference.set(user);
        User updateUser = new User("zhong",12);
        atomicReference.compareAndSet(user,updateUser);
        System.out.println(atomicReference.get().getName());
        System.out.println(atomicReference.get().getOld());

    }

    static class User{
        private String name;
        private int old;

        public User(String name, int old){
            this.name = name;
            this.old = old;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public int getOld() {
            return old;
        }

        public void setOld(int old) {
            this.old = old;
        }
    }
}
```

### 5.4: 原子更新字段类

   如果需要原子地更新某个类里的某个字段时，就需要使用原子更新字段类，Atomic包提供了以下3个类进行原子字段更新。

- AtomicIntegerFieldUpdater: 原子更新整型的字段的更新器。
- AtomicLongFieldUpdater: 原子更新长整型字段的更新器。
- AtomicStampedReference: 原子更新带有版本号的引用类型。

如果需要使用需要进行2步，1：因为原子更新字段类都是抽象类，每次使用的时候必须使用静态方法 newUpdater()创建一个更新器，并且需要设置想要更新的类和属性。2：更新类的字段必须使用public volatile修饰符。

```java
private static AtomicIntegerFieldUpdater<User> a = 
        AtomicIntegerFieldUpdater.newUpdater(User.class,"old");
```

