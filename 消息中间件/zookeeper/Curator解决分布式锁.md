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

### 2.1创建会话

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