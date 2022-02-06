# Consumer Reblance

## 1、什么是rebalance？

​      rebalance本质上是一种协议，规定了一个consumer group下的所有consumer如何达成一致来分配订阅topic的每个分区。比如某个group下有20个consumer，它订阅了一个具有100个分区的topic。正常情况下，Kafka平均会为每个consumer分配5个分区。这个分配的过程就叫rebalance。

      当出现以下几种情况时，kafka会进行一次分区分配操作，也就是kafka consumer的rebalance
    
      1. 同一个consumer group内新增了消费者
    
      2. 消费者离开当前所属的consumer group，比如主动停机或者宕机
    
      3. topic新增了分区（也就是分区数量发生了变化）
    
      kafka consuemr的rebalance机制规定了一个consumer group下的所有consumer如何达成一致来分 配订阅topic的每个分区。而具体如何执行分区策略，就是前面提到过的两种内置的分区策略。而kafka 对于分配策略这块，提供了可插拔的实现方式， 我们还可以创建自己的分配机制。 

## 2、谁来执行Rebalance以及管理consumer的group呢？ 

​       Kafka提供了一个角色：coordinator来执行对于consumer group的管理，Kafka提供了一个角色： coordinator来执行对于consumer group的管理，当consumer group的第一个consumer启动的时候，它会去和kafka server确定谁是它们组coordinator。之后该group内的所有成员都会和该 coordinator进行协调通信 。  

3.3、Coordinator介绍
Coordinator一般指的是运行在broker上的group Coordinator，用于管理Consumer Group中各个成员，每个KafkaServer都有一个GroupCoordinator实例，管理多个消费者组，主要用于offset位移管理和Consumer Rebalance。

Coordinator存储的信息

对于每个Consumer Group，Coordinator会存储以下信息：

对每个存在的topic，可以有多个消费组group订阅同一个topic(对应消息系统中的广播)
对每个Consumer Group，元数据如下：
订阅的topics列表
Consumer Group配置信息，包括session timeout等
组中每个Consumer的元数据。包括主机名，consumer id
每个正在消费的topic partition的当前offsets
Partition的ownership元数据，包括consumer消费的partitions映射关系
如何确定consumer group的coordinator

consumer group如何确定自己的coordinator是谁呢？ 简单来说分为两步：

确定consumer group位移信息写入__consumers_offsets这个topic的哪个分区。具体计算公式：
__consumers_offsets partition# = Math.abs(groupId.hashCode() % groupMetadataTopicPartitionCount) 注意：groupMetadataTopicPartitionCount由offsets.topic.num.partitions指定，默认是50个分区。
该分区leader所在的broker就是被选定的coordinator

## 3. Rebalance过程一：JoinGroup

​       在rebalance之前，需要保证coordinator是已经确定好了的，整个rebalance的过程分为两个步骤， Join和Sync

       join: 表示加入到consumer group中，在这一步中，所有的成员都会向coordinator发送joinGroup的请 求。一旦所有成员都发送了joinGroup请求，那么coordinator会选择一个consumer担任leader角色， 并把组成员信息和订阅信息发送消费者。
    
      leader选举算法比较简单，如果消费组内没有leader，那么第一个加入消费组的消费者就是消费者 leader，如果这个时候leader消费者退出了消费组，那么重新选举一个leader，这个选举很随意，类似于随机算法。
————————————————
