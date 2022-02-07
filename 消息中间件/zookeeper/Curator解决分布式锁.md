## 1：相关介绍

​    Curator是Netflix公司开源的一套Zookeeper客户端框架。了解过Zookeeper原生API都会清楚其复杂度。Curator帮助我们在其基础上进行封装、实现一些开发细节，包括接连重连、反复注册Watcher和NodeExistsException等。

**官网地址：**https://curator.apache.org/

### 1.1 项目组件

| 名称      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| Recipes   | Zookeeper典型应用场景的实现，这些实现是基于Curator Framework。 |
| Framework | Zookeeper API的高层封装，大大简化Zookeeper客户端编程，添加了例如Zookeeper连接管理、重试机制等。 |
| Utilities | 为Zookeeper提供的各种实用程序。                              |
| Client    | Zookeeper client的封装，用于取代原生的Zookeeper客户端（ZooKeeper类），提供一些非常有用的客户端特性。 |
| Client    | Zookeeper client的封装，用于取代原生的Zookeeper客户端（ZooKeeper类），提供一些非常有用的客户端特性。 |

### 1.2 Maven的依赖

根据需要选择引入具体的artifact。但大多数情况下只用引入curator-recipes即可。

| GroupID/Org        | ArtifactID/Name           | 描述                                                         |
| ------------------ | ------------------------- | ------------------------------------------------------------ |
| org.apache.curator | curator-recipes           | 所有典型应用场景。需要依赖client和framework，需设置自动获取依赖。 |
| org.apache.curator | curator-framework         | 同组件中framework介绍。                                      |
| org.apache.curator | curator-client            | 同组件中client介绍。                                         |
| org.apache.curator | curator-test              | 包含TestingServer、TestingCluster和一些测试工具。            |
| org.apache.curator | curator-examples          | 各种使用Curator特性的案例。                                  |
| org.apache.curator | curator-x-discovery       | 在framework上构建的服务发现实现。                            |
| org.apache.curator | curator-x-discoveryserver | 可以喝Curator Discovery一起使用的RESTful服务器。             |
| org.apache.curator | curator-x-rpc             | Curator framework和recipes非java环境的桥接。                 |

根据上面的描述，开发人员大多数情况下使用的都是curator-recipes的依赖，此依赖的maven配置如下：

```
<!-- https://mvnrepository.com/artifact/org.apache.curator/curator-recipes -->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>2.12.0</version>
</dependency>
```

由于版本兼容原因，采用了2.x.x的最高版本。

## 2：相关功能

### 2.1 创建会话

​    Curator创建客户端是通过CuratorFrameworkFactory工厂类来实现的。其中，此工厂类提供了三种创建客户端的方法。 前两种方法是通过newClient来实现，仅参数不同而已。

```java
public static CuratorFramework newClient(String connectString, RetryPolicy retryPolicy)

public static CuratorFramework newClient(String connectString, int sessionTimeoutMs, int connectionTimeoutMs, RetryPolicy retryPolicy)
```

​    其中参数RetryPolicy提供重试策略的接口，可以让用户实现自定义的重试策略。默认提供了以下实现，分别为ExponentialBackoffRetry、BoundedExponentialBackoffRetry、RetryForever、RetryNTimes、RetryOneTime、RetryUntilElapsed。

```java
RetryPolicy retryPolicy  = new ExponentialBackoffRetry(1000,3);
private static CuratorFramework Client = CuratorFrameworkFactory.builder()
            .connectString("hadoop1:2181,hadoop2:2181,hadoop3:2181")
            .sessionTimeoutMs(3000)
            .connectionTimeoutMs(5000)
            .retryPolicy(retryPolicy)
            .build();
client.start();
```

参数：

- connectString：zk的server地址，多个server之间使用英文逗号分隔开
- connectionTimeoutMs：连接超时时间，如上是30s，默认是15s
- sessionTimeoutMs：会话超时时间，如上是50s，默认是60s
- retryPolicy：失败重试策略
  - ExponentialBackoffRetry：构造器含有三个参数 ExponentialBackoffRetry(int baseSleepTimeMs, int maxRetries, int maxSleepMs)
    - baseSleepTimeMs：初始的sleep时间，用于计算之后的每次重试的sleep时间，
      - 计算公式：当前sleep时间=baseSleepTimeMs*Math.max(1, random.nextInt(1<<(retryCount+1)))
    - maxRetries：最大重试次数
    - maxSleepMs：最大sleep时间，如果上述的当前sleep计算出来比这个大，那么sleep用这个时间
  - 其他，查看org.apache.curator.RetryPolicy接口的实现类
  - start()会阻塞到会话创建成功为止



### 2.2 创建节点

**1: 相关模式**

- PERSISTENT：持久化
- PERSISTENT_SEQUENTIAL：持久化并且带序列号
- EPHEMERAL：临时
- EPHEMERAL_SEQUENTIAL：临时并且带序列号

**2: 创建一个节点，初始内容为空** 

```css
client.create().forPath("path");
```

注意：如果没有设置节点属性，节点创建模式默认为持久化节点，内容默认为空

**3: 创建一个节点，附带初始化内容**

```css
client.create().forPath("path","init".getBytes());
```

**4: 创建一个节点，指定创建模式（临时节点），内容为空**

```css
client.create().withMode(CreateMode.EPHEMERAL).forPath("path");
```

**5: 创建一个节点，指定创建模式（临时节点），附带初始化内容**

```css
client.create().withMode(CreateMode.EPHEMERAL).forPath("path","init".getBytes());
```

**6: 创建一个节点，指定创建模式（临时节点），附带初始化内容，并且自动递归创建父节点**

```java
client.create()
      .creatingParentContainersIfNeeded()
      .withMode(CreateMode.EPHEMERAL)
      .forPath("path","init".getBytes());
```

这个creatingParentContainersIfNeeded()接口非常有用，因为一般情况开发人员在创建一个子节点必须判断它的父节点是否存在，如果不存在直接创建会抛出NoNodeException，使用creatingParentContainersIfNeeded()之后Curator能够自动递归创建所有所需的父节点。



### 2.3 删除节点

**1: 删除一个节点**

```cpp
client.delete().forPath("path");
```

注意，此方法只能删除**叶子节点**，否则会抛出异常。

**2: 删除一个节点，并且递归删除其所有的子节点**

```css
client.delete().deletingChildrenIfNeeded().forPath("path");
```

**3: 删除一个节点，强制指定版本进行删除**

```css
client.delete().withVersion(10086).forPath("path");
```

**4: 删除一个节点，强制保证删除**

```css
client.delete().guaranteed().forPath("path");
```

guaranteed()接口是一个保障措施，只要客户端会话有效，那么Curator会在后台持续进行删除操作，直到删除节点成功。

**注意：**上面的多个流式接口是可以自由组合的，例如：

```css
client.delete().guaranteed().deletingChildrenIfNeeded().withVersion(10086).forPath("path");
```

### 2.4 读取数据节点

**1: 读取一个节点的数据内容**

```css
client.getData().forPath("path");
```

注意，此方法返的返回值是byte[ ];

**2: 读取一个节点的数据内容，同时获取到该节点的stat**

```bash
Stat stat = new Stat();
client.getData().storingStatIn(stat).forPath("path");
```

### 2.5 更新数据节点数据

**1: 更新一个节点的数据内容**

```css
client.setData().forPath("path","data".getBytes());
```

注意：该接口会返回一个Stat实例

**2: 更新一个节点的数据内容，强制指定版本进行更新**

```css
client.setData().withVersion(10086).forPath("path","data".getBytes());
```

### 2.6 获取某个节点的所有子节点路径

```css
client.getChildren().forPath("path");
```

注意：该方法的返回值为List<String>,获得ZNode的子节点Path列表。 可以调用额外的方法(监控、后台处理或者获取状态watch, background or get stat) 并在最后调用forPath()指定要操作的父ZNode

### 2.7 事务

​    CuratorFramework的实例包含inTransaction( )接口方法，调用此方法开启一个ZooKeeper事务. 可以复合create, setData, check, and/or delete 等操作然后调用commit()作为一个原子操作提交。一个例子如下：

```java
client.inTransaction().check().forPath("path")
      .and()
      .create().withMode(CreateMode.EPHEMERAL).forPath("path","data".getBytes())
      .and()
      .setData().withVersion(10086).forPath("path","data2".getBytes())
      .and()
      .commit();
```



### 2.8 异步接口

​    上面提到的创建、删除、更新、读取等方法都是同步的，Curator提供异步接口，引入了**BackgroundCallback**接口用于处理异步接口调用之后服务端返回的结果信息。**BackgroundCallback**接口中一个重要的回调值为CuratorEvent，里面包含事件类型、响应吗和节点的详细信息。

```java
Executor executor = Executors.newFixedThreadPool(2);
client.create()
      .creatingParentsIfNeeded()
      .withMode(CreateMode.EPHEMERAL)
      .inBackground((curatorFramework, curatorEvent) -> {      System.out.println(String.format("eventType:%s,resultCode:%s",curatorEvent.getType(),curatorEvent.getResultCode()));
      },executor)
      .forPath("path");
```

​    注意：如果#inBackground()方法不指定executor，那么会默认使用Curator的EventThread去进行异步处理。

**CuratorEventType**

| 事件类型 | 对应CuratorFramework实例的方法 |
| :------: | :----------------------------: |
|  CREATE  |           #create()            |
|  DELETE  |           #delete()            |
|  EXISTS  |         #checkExists()         |
| GET_DATA |           #getData()           |
| SET_DATA |           #setData()           |
| CHILDREN |         #getChildren()         |
|   SYNC   |      #sync(String,Object)      |
| GET_ACL  |           #getACL()            |
| SET_ACL  |           #setACL()            |
| WATCHED  |       #Watcher(Watcher)        |
| CLOSING  |            #close()            |

**响应码(#getResultCode())**

| 响应码 |                   意义                   |
| :----: | :--------------------------------------: |
|   0    |              OK，即调用成功              |
|   -4   | ConnectionLoss，即客户端与服务端断开连接 |
|  -110  |        NodeExists，即节点已经存在        |
|  -112  |        SessionExpired，即会话过期        |

## 3: 分布式锁

​    分布式锁服务宕机,ZooKeeper一般是以集群部署,如果出现ZooKeeper宕机,那么只要当前正常的服务器超过集群的半数,依然可以正常提供服务持有锁资源服务器宕机,假如一台服务器获取锁之后就宕机了, 那么就会导致其他服务器无法再获取该锁. 就会造成死锁问题, 在Curator中, 锁的信息都是保存在临时节点上, 如果持有锁资源的服务器宕机, 那么ZooKeeper 就会移除它的信息, 这时其他服务器就能进行获取锁操作。

### 3.1 实现原理

​    使用zk的临时节点和有序节点，每个线程获取锁就是在zk创建一个临时有序的节点，比如在/lock/目录下。创建节点成功后，获取/lock目录下的所有临时节点，再判断当前线程创建的节点是否是所有的节点的序号最小的节点。如果当前线程创建的节点是所有节点序号最小的节点，则认为获取锁成功。比如当前线程获取到的节点序号为/lock/003,然后所有的节点列表为[/lock/001,/lock/002,/lock/003],则对/lock/002这个节点添加一个事件监听器。如果锁释放了，会唤醒下一个序号的节点，然后重新执行第3步，判断是否自己的节点序号是最小。比如/lock/001释放了，/lock/002监听到时间，此时节点集合为[/lock/002,/lock/003],则/lock/002为最小序号节点，获取到锁。

### 3.2 锁分类

- InterProcessSemaphoreMutex：分布式不可重入排它锁
- InterProcessMutex：分布式可重入排它锁
- InterProcessReadWriteLock：分布式读写锁
- InterProcessMultiLock：多重共享锁，将多个锁作为单个实体管理的容器
- InterProcessSemaphoreV2：共享信号量

#### 3.2.1 InterProcessSemaphoreMutex

​    InterProcessSemaphoreMutex是一种不可重入的互斥锁，也就意味着即使是同一个线程也无法在持有锁的情况下再次获得锁，所以需要注意，不可重入的锁很容易在一些情况导致死锁，比如你写了一个递归。

#### 3.2.2 InterProcessMutex

​    通过在zookeeper的某路径节点下创建临时序列节点来实现分布式锁，即每个线程（跨进程的线程）获取同一把锁前，都需要在同样的路径下创建一个节点，节点名字由uuid+递增序列组成。而通过对比自身的序列数是否在所有子节点的第一位，来判断是否成功获取到了锁。当获取锁失败时，它会添加watcher来监听前一个节点的变动情况，然后进行等待状态。直到watcher的事件生效将自己唤醒，或者超时时间异常返回。

#### 3.2.3 InterProcessReadWriteLock

   读写锁维护一对关联的锁，一个用于只读操作，一个用于写操作。只要没有写锁，读锁可以被多个用户同时持有，而写锁是独占的。读写锁允许从写锁降级为读锁，方法是先获取写锁，然后就可以获取读锁。但是，无法从读锁升级到写锁。

#### 3.2.4 InterProcessMultiLock

   多个锁作为一个锁，可以同时在多个资源上加锁。一个维护多个锁对象的容器。当调用acquire()时，获取容器中所有的锁对象，请求失败时，释放所有锁对象。同样调用release()也会释放所有的锁。

#### 3.2.5 InterProcessSemaphoreV2

   一个计数的信号量类似JDK的Semaphore，所有使用相同锁定路径的jvm中所有进程都将实现进程间有限的租约。此外，这个信号量大多是“公平的” - 每个用户将按照要求的顺序获得租约。有两种方式决定信号号的最大租约数。一种是由用户指定的路径来决定最大租约数，一种是通过SharedCountReader来决定。如果未使用SharedCountReader，则不会进行内部检查比如A表现为有10个租约，进程B表现为有20个。因此，请确保所有进程中的所有实例都使用相同的numberOfLeases值。acuquire()方法返回的是Lease对象，客户端在使用完后必须要关闭该lease对象(一般在finally中进行关闭)，否则该对象会丢失。如果进程session丢失（如崩溃），该客户端拥有的所有lease会被自动关闭，此时其他端能够使用这些lease。



### 3.3 相关代码

