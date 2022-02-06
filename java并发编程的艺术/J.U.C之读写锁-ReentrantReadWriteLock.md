## J.U.C之读写锁：ReentrantReadWriteLock

##  1 读写锁的原理以及相关代码

  重入锁ReentrantLock是排他锁，排他锁在同一时刻仅有一个线程可以进行访问，但是在大多数场景下，大部分时间都是提供读服务，而写服务占有的时间较少。然而读服务不存在数据竞争问题，如果一个线程在读时禁止其他线程读势必会导致性能降低。所以就提供了读写锁。

​    读写锁维护着一对锁，一个读锁和一个写锁。通过分离读锁和写锁，使得并发性比一般的排他锁有了较大的提升：在同一时间可以允许多个读线程同时访问，但是在写线程访问时，所有读线程和写线程都会被阻塞。

**读写锁的主要特性：** 

- **公平性：支持公平性和非公平性。** 
- **重入性：支持重入。**

​    读写锁最多支持65535个递归写入锁和65535个递归读取锁。 锁降级：遵循获取写锁、获取读锁在释放写锁的次序，写锁能够降级成为读锁 读写锁ReentrantReadWriteLock实现接口ReadWriteLock，该接口维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁可以由多个 reader 线程同时保持。

​     ReadWriteLock定义了两个方法。readLock()返回用于读操作的锁，writeLock()返回用于写操作的锁。  ReentrantReadWriteLock与ReentrantLock一样，其锁主体依然是Sync，它的读锁、写锁都是依靠Sync来实现的。所以ReentrantReadWriteLock实际上只有一个锁，只是在获取读取锁和写入锁的方式上不一样而已，**它的读写锁其实就是两个类：ReadLock、writeLock，这两个类都是lock实现。**

  通过一个缓存示例来说明读写锁的使用方法。

```java
public class Cache {
    static Map<String,Object> map = new HashMap<>();
    static ReentrantReadWriteLock rw1 = new ReentrantReadWriteLock();
    static Lock r = rw1.readLock();
    static Lock w = rw1.writeLock();

    public static  Object get(String key){
        r.lock();
        try{
            return map.get(key);
        }catch (Exception e){
            e.printStackTrace();
            return null;
        }finally {
            r.unlock();
        }
    }
    
    public static final Object put(String key,Object value){
        w.lock();
        try{
            return map.put(key,value);
        }finally {
            w.unlock();
        }
    } 

}
```



## 2 读写锁的实现分析

### 2.1 读写状态的设计

   读写锁同样依赖自定义同步器来实现同步功能，而读写状态就是其同步器的同步状态。如果在一个整型变量上维护多种状态，则需要“按位切割使用”这个变量，读写锁将变量切分成了两个部分，高16位表示读，低16位表示写，分割之后，读写锁是如何迅速确定读锁和写锁的状态呢？通过为运算。假如当前同步状态为S，那么写状态等于 S & 0x0000FFFF（将高16位全部抹去），读状态等于S >>> 16(无符号补0右移16位)。

​    S不等于0时，当写状态(S & 0x0000FFFF)等于0时，则读状态(s>>>16)大于0，即读锁已被获取。

### 2.2 写锁的获取与释放

**写锁就是一个支持可重入的排他锁。**

​    

- **写锁的获取** 

​    写锁的获取最终会调用tryAcquire(int arg)，该方法在内部类Sync中实现。该方法和ReentrantLock的tryAcquire(int arg)大致一样，在判断重入时增加了一项条件：读锁是否存在。  

​    因为要确保写锁的操作对读锁是可见的，如果在存在读锁的情况下允许获取写锁，那么那些已经获取读锁的其他线程可能就无法感知当前写线程的操作。因此只有等读锁完全释放后，写锁才能够被当前线程所获取，一旦写锁获取了，所有其他读、写线程均会被阻塞。



### 2.3 锁降级

​    是指把持住当前获取的写锁，再获取到读锁，随后释放先前拥有写锁的过程。

**目的：**

​    为了保证数据的可见性，如果当前线程不获取读锁而是直接释放写锁，假如此时另一个线程T获取了写锁并需修改了数据，那么当前线程无法感知线程T的数据更新，如果当前线程获取了读锁，即遵循锁降级的步骤，则线程T将会被阻塞，知道当前线程使用数据并释放读锁之后，线程T才能获取写锁惊醒数据更新。