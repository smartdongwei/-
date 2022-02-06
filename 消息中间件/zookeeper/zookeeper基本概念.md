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

