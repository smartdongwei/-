

## 1：RocketMQ事务消息设计思路

根据CAP理论，RocketMQ事务消息通过异步确保方式，保证事务的最终一致性。设计流程上借鉴两阶段提交理论，流程图如下：

![image-20211205203411534](E:\study\images\image-20211205203411534.png)

1. 应用模块遇到要发送事务消息的场景时，先发送prepare消息给MQ。
2. prepare消息发送成功后，应用模块执行数据库事务（本地事务）。
3. 根据数据库事务执行的结果，再返回Commit或Rollback给MQ。
4. 如果是Commit，MQ把消息下发给Consumer端，如果是Rollback，直接删掉prepare消息。
5. 第3步的执行结果如果没响应，或是超时的，启动定时任务回查事务状态（最多重试15次，超过了默认丢弃此消息），处理结果同第4步。
6. MQ消费的成功机制由MQ自己保证。
7. 针对超时状态，Broker主动向Producer发起消息回查；
8. Step6：Producer处理回查消息，返回对应的本地事务的执行结果；
9. Step7：Broker针对回查消息的结果，执行Commit或Rollback操作，同Step4。

## 2：RocketMQ事务消息实现流程

以RocketMQ 4.5.2版本为例，事务消息有专门的一个队列RMQ_SYS_TRANS_HALF_TOPIC，所有的prepare消息都先往这里放，当消息收到Commit请求后，就把消息再塞到真实的Topic队列里，供Consumer消费，同时向RMQ_SYS_TRANS_OP_HALF_TOPIC塞一条消息。简易流程图如下：

