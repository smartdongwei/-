## 1：Zookeeper工作机制

### 1.1 基本概念

​    zookeeper从设计模式角度：是一个基于观察者模式设计的分布式服务管理框架，**它负责存储管理大家关心的数据**，然后**接受观察者的注册**，一旦这些数据的状态发生变化，Zookeeper**负责通知已经在zookeeper上注册的哪些观察者**做出相应的反应。

zookeeper主要是文件系统和通知机制。

- 文件系统主要是用来存储数据
- 通知机制主要是服务器或者客户端进行通知，并且监督

### 1.2 特点

- 一个leader，多个follower的集群
- 集群只要有半数以上包括半数就可正常服务，一般安装奇数台服务器
- 全局数据一致，每个服务器都保存同样的数据，实时更新
- 更新的请求顺序保持顺序（来自同一个服务器）
- 数据更新的原子性，数据要么成功要么失败
- 数据实时更新性很快

### 1.3 数据结构

​    与 Unix 文件系统很类似，可看成树形结构，每个节点称做一个 ZNode。每一个 ZNode 默认能够存储 1MB 的数据。也就是只能存储小数据。

### 1.4 应用场景

- 统一命名服务（域名服务）

  在分布式环境下，经常需要对应用/服务进行统一命名

- 统一配置管理（一个集群中的所有配置都一致，且也要实时更新同步）
  将配置信息写入ZooKeeper上的一个Znode，各个客户端服务器监听这个Znode。一旦Znode中的数据被修改，ZooKeeper将通知各个客户端服务器

- 统一集群管理（掌握实时状态）
  将节点信息写入ZooKeeper上的一个ZNode。监听ZNode获取实时状态变化

- 服务器节点动态上下线
  软负载均衡（根据每个节点的访问数，让访问数最少的服务器处理最新的数据需求）

## 2：docker安装zookeeper

### 2.1 单节点安装

- **拉取镜像**

```shell
docker pull zookeeper:3.5.7
```

- **将它部署在 /usr/local/zookeeper 目录下**

```
cd /usr/local && mkdir zookeeper && cd zookeeper
```

- **创建data目录，用于挂载容器中的数据目录**

```
mkdir data
```

- **部署的命令**

```
docker run -d -e TZ="Asia/Shanghai" -p 2181:2181 -v D:/docker/zookeeper/data:/data --name zookeeper --restart always zookeeper:3.5.7
```

 **命令的解释**

```
-e TZ="Asia/Shanghai" # 指定上海时区 
-d # 表示在一直在后台运行容器
-p 2181:2181 # 对端口进行映射，将本地2181端口映射到容器内部的2181端口
--name # 设置创建的容器名称
-v # 将本地目录(文件)挂载到容器指定目录；--restart always #始终重新启动zookeeper
```

### 2.2 测试部署情况

- **使用zk命令行客户端连接zk**

```
docker run -it --rm --link zookeeper:zookeeper zookeeper:3.5.7 zkCli.sh -server zookeeper
```

### 2.3 集群情况部署

​    集群方式选择使用`docker-compose`来完成。





## 3：配置文件解读

**配置文件的5大参数**

- tickTime = 2000发送时间
- initLimit = 10初始化通信的时间，最多不能超过的时间，超过的话，通信失败
- syncLimit = 5建立好连接后，下次的通信时间如果超过，通 信失败
- dataDir保存zookeeper的数据，默认是temp会被系统定期清除
- clientPort = 2181客户端的连接端口，一般不需要修改
  

## 4：Zookeeper的选举机制

  **区分好第一次启动与非第一次启动的步骤**。相关概念如下：

- **事务ID(ZXID)**: 也就是事务id， 为了保证事务的顺序一致性，zookeeper 采用了递增的事 务 id 号（zxid）来标识事务。对于每一个事务请求,ZooKeeper都会为其分配一个全局唯一的事务ID,用ZXID来表示，通常是一个64位的数字。每一一个ZXID对应一次更新操作，从这些ZXID中可以间接地识别出ZooKeeper处理这些更新操作请求的全局顺序。

- **epoch** ：可以理解为当前集群所处的年代或者周期，每个 leader 就像皇帝，都有自己的年号，所以每次改朝换代， leader 变更之后，都会在前一个年代的基础上加 1 。这样 就算旧的 leader 崩 溃 恢 复 之 后 ，也 没 有 人 听 他 的 了 ，因 为 follower 只听从当前年代的 leader 的命令

- **myid**: 服务器唯一标识。

- **状态**：looking选举状态；following跟随状态，同步leader状态；oberserving观察状态，同步同步leader状态；leading状态。

  

  

### 4.1 初始化时的选举机制

![在这里插入图片描述](E:\study\images\zookeeper)



### 4.2 非第一次启动如何选举

![在这里插入图片描述](E:\study\images\zookeeper_2.jpg)



## 5：客户端命令

### 5.1 常用命令

| 命令      | 功能                                                         |
| --------- | ------------------------------------------------------------ |
| help      | 显示所有操作命令                                             |
| ls path   | 使用 ls 命令来查看当前 znode 的子节点 [可监听] ，-w 监听子节点变化，-s 附加次级信息 |
| create    | 普通创建                                                     |
| -s        | 含有序列                                                     |
| -e        | 临时（重启或者超时消失）                                     |
| get path  | 获得节点的值 [可监听] ，-w 监听节点内容变化                  |
| -s        | 附加次级信息                                                 |
| set       | 设置节点的具体值                                             |
| stat      | 查看节点状态                                                 |
| delete    | 删除节点                                                     |
| deleteall | 递归删除节点                                                 |

查看当前数据节点详细信息`ls -s /`

![在这里插入图片描述](E:\study\images\zookeeper-2.jpg)

| 名称           | 表述                                                         |
| -------------- | ------------------------------------------------------------ |
| czxid          | 创建节点的事务 zxid，每次修改 ZooKeeper 状态都会产生一个 ZooKeeper 事务 ID。事务 ID 是 ZooKeeper 中所有修改总的次序。每次修改都有唯一的 zxid，如果 zxid1 小于 zxid2，那么 zxid1 在 zxid2 之前发生。 |
| ctime          | znode 被创建的毫秒数（从 1970 年开始）                       |
| mzxid          | znode 最后更新的事务 zxid                                    |
| mtime          | znode 最后修改的毫秒数（从 1970 年开始）                     |
| pZxid          | znode 最后更新的子节点 zxid                                  |
| cversion       | znode 子节点变化号，znode 子节点修改次数                     |
| dataversion    | znode 数据变化号                                             |
| aclVersion     | znode 访问控制列表的变化号                                   |
| ephemeralOwner | 如果是临时节点，这个是 znode 拥有者的 session id。如果不是临时节点则是 0 |
| dataLength     | znode 的数据长度                                             |
| numChildren    | znode 子节点数量                                             |

### 5.2 节点类型

**节点类型分为（两两进行组合）**

- 持久/短暂
- 有序号/无序号

![在这里插入图片描述](E:\study\images\zookeeper-3.jpg)

1.  **创建节点不带序号的**

通过create 进行创建

![在这里插入图片描述](E:\study\images\zookeeper_4.jpg)

2. **如果创建节点带序号的通过加上`-s`参数**
   即便创建两个一样的，`-s` 自动会加上序号辨别两者不同，退出客户端之后，这些节点并没有被清除。

![在这里插入图片描述](E:\study\images\zookeeper_5.jpg)

3. **创建临时节点**

如果创建临时节点 加上参数 `-e`
临时节点不带序号`-e`
临时节点带序号`-e -s`
如果退出客户端，这些短暂节点将会被清除

4. **如果修改节点的值，通过`set key value`即可**

### 5.3 监听器原理

![在这里插入图片描述](E:\study\images\zookeeper_6.jpg)

**注册一次监听一次**
    主要有节点值的变化还有节点路径的变化，在一个服务器中通过修改其值或者路径，在另一个服务器中进行监听，通过get -w 节点值进行监听节点值的变化，如果是路径的变化，在另一个服务器中进行ls -w 路径进行监听。

​    如果有很多个节点，不能直接使用删除delete，需要删除deleteall，查看节点的状态信息 stat。


## 6：原始Zookeeper的api信息

### 6.1 zk连接

  **相关连接参数信息如下：**

- **connectString：**多个zookeeper IP地址用逗号分隔，也可以在后面带上节点，这样后续对节点的所有操作都是在这个节点之下，例如connectString为10.0.4.105:2181,10.0.4.120:2181,10.0.4.129:2182/xxxx，然后使用create /app，这样创建出来的节点的绝对路径为/xxxx/app，这样操作一定要确保/xxxx先存在，否则会报错。
- **sessionTimeout：**session超时时间，单位为ms。
- **watcher：**监听连接的状态，代码中使用了同步计数器CountDownLatch，因为new Zookeeper创建对象立马就会返回了，而客户端连接到服务端是耗时的，这个时候并没有真正的连接成功，如果这个时候拿zk客户端对象去做操作会报错，所以要等待连接建立成功的时候才能使用客户端对象。


```java
@ConfigurationProperties(prefix = "zookeeper")
@Data
public class ZookeeperProperties {
    /**
     * 服务端地址信息  127.0.0.1:2181
     */
    private String address;
    /**
     * 超时时间  毫秒
     */
    private int timeout;

}
```

```java
@Slf4j
@Configuration
@EnableConfigurationProperties(ZookeeperProperties.class)
public class ZookeeperClient {

    @Autowired
    private ZookeeperProperties properties;

    @Bean(name = "zkClient")
    public ZooKeeper zkClient(){
        ZooKeeper zooKeeper=null;
        try {
            final CountDownLatch countDownLatch = new CountDownLatch(1);
            //连接成功后，会回调watcher监听，此连接操作是异步的，执行完new语句后，直接调用后续代码
            //  可指定多台服务地址 127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183
            zooKeeper = new ZooKeeper(properties.getAddress(), properties.getTimeout(), new Watcher() {
                @Override
                public void process(WatchedEvent event) {
                    if(Event.KeeperState.SyncConnected==event.getState()){
                        //如果收到了服务端的响应事件,连接成功
                        countDownLatch.countDown();
                    }
                    log.info("收到了服务器的响应事件，状态为{}",event.getState());
                }
            });
            countDownLatch.await();
            log.info("【初始化ZooKeeper连接状态....】= {}",zooKeeper.getState());
        }catch (Exception e){
            log.error("初始化ZooKeeper连接异常....】= {}",e.getMessage());
        }
        return  zooKeeper;
    }

}
```

### 6.2 创建一个新的节点

**相关的参数如下：**

- **path** - Znode路径。例如，/myapp1，/myapp2，/myapp1/mydata1，myapp2/mydata1/myanothersubdata

- **data** - 要存储在指定znode路径中的数据

- **acl** - 要创建的节点的访问控制列表。ZooKeeper API提供了一个静态接口 **ZooDefs.Ids** 来获取一些基本的acl列表。例如，ZooDefs.Ids.OPEN_ACL_UNSAFE返回打开znode的acl列表。

- **createMode** - 节点的类型，即临时，顺序或两者。这是一个**枚举**。其中

  ​    **PERSISTENT：**持久化目录节点 

  ​    **PERSISTENT_SEQUENTIAL：**顺序自动编号持久化目录节点, 存储数据不会丢失, 会根据当前已存在节点数自动加1, 然后返回给客户端已经创建成功的节点名

  ​    **EPHEMERAL_SEQUENTIAL：** 临时自动编号节点, 一旦创建这个节点,当回话结束, 节点会被删除, 并且根据当前已经存在的节点数自动加1, 然后返回给客户端已经成功创建的目录节点名 .

```java
public void create(String name,String data) throws KeeperException, InterruptedException {
    log.info("创建zk的节点{}，数据为{}",name,data);
    String result = zkClient.create(name,data.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    log.info("返回的结果为{}",result);
}
```

### 6.3 获取子节点并监听其变化

#### 6.3.1 局部监听器

```java
/**
 * 获取节点时，并对其进行监听，使用非系统的监听器处理
 * @param path  监听的路径信息
 */
public void getChildrenWatch(String path) throws KeeperException, InterruptedException {
    final String nodePath = path;
    zkClient.getChildren(path, watchedEvent -> {
        try{
            List<String> childs = zkClient.getChildren(nodePath, (Watcher) this);
            log.info("自定义节点监控-根节点下的子节点: {}, 类型: {}",childs,watchedEvent.getType());
        }catch (Exception e){
            e.printStackTrace();
        }
    });
    // 线程休眠, 否则不能监控到数据
    Thread.sleep(Long.MAX_VALUE);
}
```

#### 6.3.2 全局监听器

```java
/**
 * 全局监听器
 * @author wang
 */
@Slf4j
public class WatcherApi implements Watcher {
    @Override
    public void process(WatchedEvent watchedEvent) {
        log.info("【Watcher监听事件】={}",watchedEvent.getState());
        log.info("【监听路径为】={}",watchedEvent.getPath());
        //  三种监听类型： 创建，删除，更新
        log.info("【监听的类型为】={}",watchedEvent.getType());
    }
}
```

```java
public void getChildren() throws KeeperException, InterruptedException {
    List<String> children = zkClient.getChildren("/", true);
    for (String child : children) {
        System.out.println(child);
    }
    // 延时
    Thread.sleep(Long.MAX_VALUE);
}
```

### 6.4 客户端向服务端写数据流程

#### 6.4.1 发送给leader

​    客户端给服务器的leader发送写请求，写完数据后给手下发送写请求，手下写完发送给leader，超过半票以上都写了则发回给客户端。之后leader在给其他手下让他们写，写完在发数据给leader





#### 6.4.2 发送给follower

​    客户端给手下发送写的请求，手下给leader发送写的请求，写完后，给手下发送写的请求，手下写完后给leader发送确认，超过半票，leader确认后，发给刻划断，之后leader在发送写请求给其他手下



### 6.5 服务器动态上下线监听

1. 服务器上线的时候其实就是服务器启动时去注册信息（创建的都是临时节点）
2. 客户端获取到当前在线的服务器列表后
3. 服务器节点下线后给集群管理
4. 集群管理服务器节点的下件时间通知给客户端
5. 客户端通过获取服务器列表重选选择服务器