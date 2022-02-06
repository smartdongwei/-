## redis的常用知识

### 1: 需要密码登录

```shell
 . /redis-cli
 redis-cli -h xxx -p 6379 -a xxx
-- 或者 登录后
 auth pass
```

### 2: 基本数据类型操作



```shell
# 清空数据库中所有的数据
flushdb 
## 所有的key值
keys *
1) "key1"
# key是否存在
exists key1
(integer) 1
```



#### 2.1 String 类型

```shell
set key value #设置key-value
get key  #获取key的value
exists key  #key是否存在
append key value #追加字符串，若key不存，相当于set key value
strlen key #获取字符串长度
incr key #当前key的value加1
decr key #当前key的value减一
incrby key 10 #当前key加10
decrby key 10 #当前key减10
getrange key 0 3 #字符串范围 （getrange key 0 -1 获取全部字符串）
setrange key 1 xx #替换指定位置开始的字符串
setex key second value    #（set with expire）设置过期时间
setnx key value   #（set if not with exists ）不存在再设置 （分布式锁中常使用）
mset key1 v1 key2 v2  #批量设置
mget key1 key2 key3  #批量获取
msetnx key1 v1 key2 v2  #不存在再设置(批量 原子性操作  一起成功 一起失败)
getset key value #先获取原值再设置新值
```

#### 2.2 Hash 类型

```shell
hset key field value  #存入一个具体键值对
hget key field   #获取一个字段值
hmset key field value field1 value1 ...  #存入多个具体键值对
hmget key field field1 ...  #获取多个字段值
hgetall key    #获取全部数据
hdel key field  #删除hash指定的key字段，对应value也就没有了
hlen key     #获取hash中字段数量
hexists key field   #判断hash中某个字段是否存在
hkeys key    #获取hash中全部key
hvals key    #获取hash中全部value
hincrby key field 1  #hash中指定key的value加1
hdecrby key field 1  #hash中指定key的value减1
hsetnx key field value   #如果hash中指定key不存在则创建，存在则创建失败
```

#### 2.3 List 类型

```shell
lpush key v1 v2 ...  #将一个值或多个值插入列表的头部(左)
rpush key v1 v2 ...  #将一个值或多个值插入列表的尾部(右)
lrange key start end  #用过区间获取具体的值  （0 -1 区间获取全部值）
lpop key  #移除列表头部第一个值（左）
rpop key  #移除列表尾部第一个值（右）
lindex key index #通过索引获取值
llen key   #获取列表长度
lrem key count value  #移除list集合中指定个数的value  精确匹配
ltrim key start stop   #通过下标截取指定长度，list已经改变，只剩下截取后的元素
rpoplpush key otherkey  #移除列表中最后一个元素，并将它插入另一个列表头部
lset key index value  #将列表中指定下标的值替换为另外一个值，更新操作 （如果列表或索引不存在  会报错）
linsert key before v1 v2  #在v1前插入v2
linsert key after v1 v2  #在v1后插入v2
```

小结：

- 他实际上是一个链表，before after left right 都可以插入值
- 如果key不存在，创建新的链表
- 如果key存在，新增内容
- 如果移除了所有值，空链表，也代表不存在
- 在两边插入或改动值，效率最高！中间元素，相对来说效率会低一点！
- 消息排队 消息队列（Lpush Rpop） ，栈（Lpush Lpop）



#### 2.4  Set 类型

```shell
sadd key value  #添加元素
smembers key   #查看指定set中所用元素
sismember key value  #判断某一个值在指定set中是否存在
scard key  #获取set中的内容元素个数
srem key value   #移除set中指定元素
srandmember key count  #随机选出指定个数的成员
spop key  #随机移除元素
smove oldkey  newkey member  #将一个指定的值，从一个set移动到另一个set
sdiff key...   #获取多个set差集
sinter key...  #获取多个set交集 （共同好友、共同关注）
sunion key...  #获取多个set并集
```

#### 2.5 Zset

```shell
zadd key score value  #添加元素
zrange key 0 1   #通过索引区间返回有序集合指定区间内的成员   （0 -1）返回全部
zrangebyscore key min max   #排序并返回 从小到大  例如：zrangebyscore key1 -inf +inf    （-inf：负无穷   +inf：正无穷 ）
zrevrange key 0 -1     #排序并返回 从大到小
zrem key value   #移除指定元素
zcard key        #获取有序集合中的数量
zcount key start stop   #获取指定区间中的成员数量
```

### 3:Redis三种特殊数据类型

#### 3.1geospatial

地理位置 （定位、附近的人、打车距离……）
GEO底层就是Zset 可以用Zset命令操作Geo

```
#geoadd 添加地理位置规则：两极无法直接加入，通常通过java一次性导入  有效经度：-180到180  有效纬度：-85.05112878到85.05112878geoadd china:city 121.47 31.23 shanghaigeoadd china:city 106.50 29.53 chongqing  114.05 22.52 shenzhen 120.16 30.24 hangzhou 108.96 34.26 xian#geopop 获取指定成员的经度和纬度GEOPOS china:city chongqing beijin#geodist 查看成员间的的直线距离GEODIST china:city beijin shanghai km#georadius 以给定经纬度为中心，找出某一半径内的元素（附件的人）GEORADIUS china:city 110 30 1000 kmGEORADIUS china:city 110 30 1000 km withdist withcoord count 2 （withdist 显示直线距离  withcoord 显示经纬度  count  显示几条）#georadiusbymember 以给定成员为中心，找出某一半径内的元素georadiusbymember china:city beijing 1000 km withdist withcoord count 2 （withdist 显示直线距离  withcoord 显示经纬度  count  显示几条）#geohash 返回一个或多个位置元素的geohash表示  将二维的经纬度转换成一维的11位字符串 如果两个字符串越接近，则距离越近。
```

#### 3.2 hyperloglog

基数 （不重复的元素个数） 可以接受误差 大概有0.81%的错误率
Redis hyperloglog 基数统计算法：
优点： 占用内存固定，存放2^64不同的元素的技术，只需要占用12KB内存
**网页的UV （一个人访问一个网站多次，统计出还是一个人）**
传统的方式：set集合保存用户id，统计set中用户数量。 但是相对消耗更多内存，我们的目的并不是保存用户id，目的只是计数。

```
PFadd key element  #创建一组元素
PFcount key   #统计元素基数
pfmerge key3 key1 key2   #合并两组key1 key2 => key3  并集
```

#### 3.3 bitmaps

位存储
统计用户信息 活跃 不活跃 登录 未登录 打卡
两个状态的 都可以使用bitmaps
bitmaps位图数据结构，都是操作二进制位来进行记录的，非0即1

```shell
setbit key offset value  #设置位图
getbit key offset        #获取指定位图的值
bitcount key     #统计数量###################################################例如 一周打卡   0为打卡 1打卡
127.0.0.1:6379> SETBIT sign 0 1
(integer) 0
127.0.0.1:6379> SETBIT sign 1 0
(integer) 0
127.0.0.1:6379> SETBIT sign 2 1
(integer) 0
127.0.0.1:6379> SETBIT sign 3 0
(integer) 0
127.0.0.1:6379> SETBIT sign 4 0
(integer) 0
127.0.0.1:6379> SETBIT sign 5 0
(integer) 0
127.0.0.1:6379> SETBIT sign 6 1
(integer) 0
127.0.0.1:6379> GETBIT sign 0
(integer) 1
127.0.0.1:6379> BITCOUNT sign
(integer) 3
```

### 4:Redis事务

**Redis单条命令时保证原子性的，但是事务不保证原子性**
Redis事务的本质：一组命令的集合！一个事务中所有命令都会被序列化，在事务执行的过程中，会按照顺序执行。**一次性、顺序性、排他性**

Redis事务没有隔离级别的概念！
所有命令在事务中，并没有直接执行！只有在发起执行命令的时候才会执行！

Redis的事务：

- 开启事务（multi）

- 命令入队（…）

- 执行事务（exec）\ 放弃事务（discard）

  ```shell
  127.0.0.1:6379> MULTI
  OK
  127.0.0.1:6379(TX)> set k1 v1
  QUEUED
  127.0.0.1:6379(TX)> set k2 v2
  QUEUED
  127.0.0.1:6379(TX)> get k2
  QUEUED
  127.0.0.1:6379(TX)> set k3 v3
  QUEUED
  127.0.0.1:6379(TX)> EXEC
  1) OK
  2) OK
  3) "v2"
  4) OK
  ```

编译型异常（代码有问题，命令有错），事务中所有的命令都不会被执行！

```shell
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> get k1
QUEUED
127.0.0.1:6379(TX)> setget k2 v2 
#错误命令(error) ERR unknown command `setget`, with args beginning with: `k2`, `v2`,
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> EXEC   #执行事务报错(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k1   #所有命令都没有被执行(nil)127.0.0.1:6379>
```

运行时异常 ，如果事务队列中存在语法性，那么执行命令的时候，其他命令是可以正常执行的，错误命令抛出异常。

```shell
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379(TX)> set k1 "gg"
QUEUED
127.0.0.1:6379(TX)> incr k1  #虽然命令报错了，但是事务依旧执行成功了
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> get k2
QUEUED
127.0.0.1:6379(TX)> EXEC
1) OK
2) (error) ERR value is not an integer or out of range
3) OK
4) "v2"
127.0.0.1:6379>
```

### 5:Redis实现乐观锁

监控！Watch/unwatch(解锁，如果事务执行失败，先解锁，然后再次手动去监视)

- 悲观锁：
  - 很悲观，什么时候都会出问题，无论做什么都会加锁！
- 乐观锁
  - 很乐观，认为什么时候都不会出问题，不会加锁！更新数据的时候判断，在此期间是否有人修改过这个数据。
  - 获取version
  - 更新时比较version

```shell
127.0.0.1:6379> set k1 100
OK
127.0.0.1:6379> set k2 0
OK
127.0.0.1:6379> WATCH k1  #监控
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379(TX)> DECRBY k1 20
QUEUED
127.0.0.1:6379(TX)> INCRBY k2 20
QUEUED
127.0.0.1:6379(TX)> EXEC  #执行之前，另外一个线程修改了监控值，导致事务执行失败(nil)
127.0.0.1:6379>####################### 在上一个线程EXEC之前 执行以下命令 #########
127.0.0.1:6379> set k1 1000
OK
```

### 6: Redis的持久化

#### 6.1RDB（Redis DataBase）

在指定时间间隔将内存中的数据集体快照写入磁盘，也就是行话讲的snapshot快照，它恢复时是将快照文件直接读到内存中。

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入一个临时文件中，待持久化过程结束后，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何I/O操作的。这就确保了极高的性能。如果需要大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加高效。

优点：
1、适合大规模数据恢复。
2、对数据完整性要求不高。

缺点：
1、需要一定时间间隔进程操作，如果在时间间隔内redis宕机，最后一次持久化后的数据丢失。
2、fork子进程的时候，会占用一定的内存空间。

默认的持久化方式就是RDB方式，一般情况下不需要修改这个配置。默认保存的rdb文件为**dump.rdb**

触发规则：
1、save的规则满足的情况下，自动触发rdb规则
2、flushall命令执行后，自定触发rdb规则
3、退出redis时，自动触发rdb规则

恢复rdb文件：
1、只需要将rdb文件放在redis的启动目录就可以，redis启动的时候会自动检查dump.rdb文件，自动恢复rdb文件中的数据

```
127.0.0.1:6379> config get dir1) "dir"2) "/usr/local/bin"  #如果这个目录下存在rdb文件，启动redis就是自动恢复其中数据127.0.0.1:6379>
```

在主从复制中，rdb就是备用了！从机上面！

#### 6.2 AOF（Append Only File）

*将所有执行过的写命令都记录下来，history，在恢复的时候将记录的命令全部执行一遍。*

以日志的形式来记录每个写操作，将Redis执行过的的所有指令记录下来（读操作不记录），只许追加文件不许改写文件，redis启动时，会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容，将写指令从前到后执行一次以完成数据的恢复工作。

aof默认是不开启的，需要手动开启（appendonly yes）。aof默认保存的是appendonly.aof文件。
重启，redis就可以生效了。

如果aof文件损坏或有错误，redis无法启动。我们需要修复aof文件。aof文件损坏修复:redis-check-aof —fix appendonly.aof

优点：

```
# appendfsync always  #每次修改都会执行同步，消耗性能appendfsync everysec  #每秒执行一次同步，但是可能会丢失这1s的数据# appendfsync no      #不执行同步，这个时候操作系统自己同步数据，速度最快
```

1、每次修改都会执行同步，消耗性能,文件数据完整性更好。
2、每秒执行一次同步，但是可能会丢失这1s的数据
3、不执行同步，这个时候操作系统自己同步数据，速度最快

缺点：
1、相对于数据文件来说，aof文件远远大于rdb文件，修复的速度也较慢。
2、aof运行效率也要比rdb慢，所以我们redis默认的配置就是rdb持久化。

**扩展：**
1、RDB 持久化方式能够在指定的时间间隔内对你的数据进行快照存储
2、AOF 持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以Redis 协议追加保存每次写的操作到文件末尾，Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大。
**3、只做缓存，如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化**
4、同时开启两种持久化方式
在这种情况下，当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。
RDB 的数据不实时，同时使用两者时服务器重启也只会找AOF文件，那要不要只使用AOF呢？作者建议不要，因为RDB更适合用于备份数据库（AOF在不断变化不好备份），快速重启，而且不会有AOF可能潜在的Bug，留着作为一个万一的手段。
5、性能建议
因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留 save 900 1 这条规则。
如果Enable AOF ，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了，代价一是带来了持续的IO，二是AOF rewrite 的最后将 rewrite 过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上，默认超过原大小100%大小重写可以改到适当的数值。
如果不Enable AOF ，仅靠 Master-Slave Repllcation 实现高可用性也可以，能省掉一大笔IO，也减少了rewrite时带来的系统波动。代价是如果Master/Slave 同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个 Master/Slave 中的 RDB文件，载入较新的那个，微博就是这种架构。

### 7:Redis主从复制

#### 7.1 概念

主从复制，是指将一台Redis服务器的数据，复制到其他Redis的服务器。前者称为主节点（master/leader），后者称为从节点（slave/follower）；数据的复制是单向的，只能从主节点到从节点。Master以写为主，Slave以读为主。

主从复制，读写分离！80%的情况下都是进行读的操作！减缓服务器压力！

**默认情况下，每台Redis服务器都是主节点；且一个主节点可以有多个从节点（或没有从节点），但是一个从节点有且只有一个主节点。**

**主从复制的作用主要包括**

- 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
- 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
- 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是写少读多的场景下，通过多个节点分担负载，可以大大提高Redis服务器的并发量。
- 高可用（集群）基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

一般来说呀，要将redis运用于工程项目之中，只使用一台Redis是万万不能的（宕机），原因如下：
1、从结构上，单个redis服务器会发生单点故障，并且一台服务器需要处理所有的请求负载呀，压力较大。
2、从容量上，单个redis服务器内存容量有限，就算一台redis服务器内存容量为256GB，也不能将所有内存用作Redis存储内存，**一般来说，单台Redis最大使用内存不应该超过20GB。**

#### 7.2 环境配置

只需要配置从库，不需要配置主库，因为默认情况下，每台Redis服务器都是主节点。

```bash
127.0.0.1:6379> info replication  #查看当前库的信息
# Replication
role:master   #角色  master
connected_slaves:0   #从库个数
master_failover_state:no-failover
master_replid:7e06d29925419238aac9519cfa2025680a0dca55
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

修改3个配置文件：
1、端口
2、pid
3、log文件名字
4、rdb备份文件名字

##### 7.2.1一主二从

默认情况下，每台redis服务器都是主节点；我们只需要配置从节点就好了（认老大）。

**方式一：命令配置 （临时）**

```bash
############ 从节点1 ###########
127.0.0.1:6380> SLAVEOF 127.0.0.1 6379   #设置为主节点的从节点（认老大）
OK
127.0.0.1:6380> info replication
# Replication
role:slave   #当前角色为从节点
master_host:127.0.0.1     #主节点信息
master_port:6379
master_link_status:up
master_last_io_seconds_ago:6
master_sync_in_progress:0
slave_read_repl_offset:14
slave_repl_offset:14
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:16682d498ce34c8fe6785dc0b4e5b44afcc2e2ef
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:14
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:14
############ 从节点2 ###########
127.0.0.1:6381> SLAVEOF 127.0.0.1 6379  #设置为主节点的从节点（认老大）
OK
127.0.0.1:6381> info replication
# Replication
role:slave   #当前角色为从节点
master_host:127.0.0.1   #主节点信息
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_read_repl_offset:56
slave_repl_offset:56
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:16682d498ce34c8fe6785dc0b4e5b44afcc2e2ef
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:56
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:43
repl_backlog_histlen:14
############ 主节点 ###########
127.0.0.1:6379> info replication
# Replication
role:master    #当前角色为主节点
connected_slaves:2     #从节点个数
slave0:ip=127.0.0.1,port=6380,state=online,offset=70,lag=0   #从节点信息
slave1:ip=127.0.0.1,port=6381,state=online,offset=70,lag=1   #从节点信息
master_failover_state:no-failover
master_replid:16682d498ce34c8fe6785dc0b4e5b44afcc2e2ef
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:70
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:70
```

**方式二：配置文件 （永久）**
修改配置文件，设置主节点信息

![img](E:\study\images\kuangstudyba8902db-a115-4a20-93fb-af3ce27f9215.png)

**细节**
主节点可以写，从节点不能写只能读！主节点中所有信息和数据都会自动被从节点保存！

**注意：
1、主节点宕机，从节点依旧以从节点的角色连接主节点，没有写的权限；当主节点重新连接，从节点依旧可以直接获取到主节点写的信息！
2、如果是用命令行配置的从节点，从节点重启，主从配置失效！但是当重新配置成为从节点，立刻就会从主节点中获取值！
3、如果是配置文件配置的从节点，从节点宕机，主节点进行写操作，当从节点重新连接时，会从主节点中获取值，之前主节点写操作的信息依旧在此从节点存在**

##### 7.2.2 复制原理

Slave启动成功连接到Master后会发送一个sync同步命令！
Master接到同步命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕后，**Master将传送整个数据文件到Slave，并完成一次同步！**

**全量复制**：Slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。

**增量复制**：Master继续将新的所有收集到的修改命令依次传给Slave，完成同步。

但是只要重新连接Master，一次完全同步（全量复制）将自动执行。数据一定会在Slave中看到。

##### 7.2.3 宕机后手动配置主节点

链路模型：M-S(M)-S ,当第一个主节点存活时，S(M)节点为从节点，无法写操作!

当第一个主节点宕机后，可以手动将节点设置为主节点:

```
slaveof no one #使自己变成主节点
```

如果第一个主节点恢复，需要重新配置。







### 8: 哨兵模式

（自动选举老大的模式）

#### 概述

主从切换技术的方法是：当主节点宕机后，需要手动把一台从节点切换为主节点，这就需要人工干预，费时费力，还会造成一段时间内的服务不可用。这不是一种推荐的方式，更多的时候，我们优先考虑哨兵模式。Redis从2.8开始，正式提供了Sentinel（哨兵）架构来解决这个问题。

能够后台监控主节点是否故障，如果故障了根据投票数自动将从节点转换为主节点。

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它能独立运行。其原理是**哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例**

![img](E:\study\images\kuangstudy781c74b7-06da-489f-bd11-fbd8ac00354d.png)

这里哨兵的两个作用：

- 通过发送命令，让Redis服务器返回监控其运行的状态，包括主服务器和从服务器。
- 当哨兵监测到master宕机，会自动将slave切换成master，然后通过**发布订阅模式**通知其他从服务器，修改配置文件，让它们切换主节点。

然而一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行相互监控，这样就形成了多哨兵模式。

#### 哨兵模式

优点：
1、哨兵集群，基于主从复制模式，所有的主从配置的优点，它都有。
2、主从可以切换，故障可以转移，系统的可用性更好。
3、哨兵模式就是主从模式的升级，手动到自动，更加健壮！

缺点：
1、Redis不好在线扩容，集群容量一旦达到上限，在线扩容就十分麻烦！
2、实现哨兵模式的配置其实是很麻烦的，里面有很多选择！

> 哨兵模式的全部配置

完整的哨兵模式配置文件 sentinel.conf

```shell
# Example sentinel.conf
# 哨兵sentinel实例运行的端口 默认26379
port 26379
# 哨兵sentinel的工作目录
dir /tmp
# 哨兵sentinel监控的redis主节点的 ip port 
# master-name  可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。
# quorum 当这些quorum个数sentinel哨兵认为master主节点失联 那么这时 客观上认为主节点失联了
# sentinel monitor <master-name> <ip> <redis-port> <quorum>
sentinel monitor mymaster 127.0.0.1 6379 1
# 当在Redis实例中开启了requirepass foobared 授权密码 这样所有连接Redis实例的客户端都要提供密码
# 设置哨兵sentinel 连接主从的密码 注意必须为主从设置一样的验证密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster MySUPER--secret-0123passw0rd
# 指定多少毫秒之后 主节点没有应答哨兵sentinel 此时 哨兵主观上认为主节点下线 默认30秒
# sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000
# 这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行 同步，
这个数字越小，完成failover所需的时间就越长，
但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。
可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。
# sentinel parallel-syncs <master-name> <numslaves>
sentinel parallel-syncs mymaster 1
# 故障转移的超时时间 failover-timeout 可以用在以下这些方面： 
#1. 同一个sentinel对同一个master两次failover之间的间隔时间。
#2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
#3.当想要取消一个正在进行的failover所需要的时间。  
#4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了
# 默认三分钟
# sentinel failover-timeout <master-name> <milliseconds>
sentinel failover-timeout mymaster 180000
# SCRIPTS EXECUTION
#配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。
#对于脚本的运行结果有以下规则：
#若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
#若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
#如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
#一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
#通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本，
#这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，
#一个是事件的类型，
#一个是事件的描述。
#如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正常启动成功。
#通知脚本
# sentinel notification-script <master-name> <script-path>
  sentinel notification-script mymaster /var/redis/notify.sh
# 客户端重新配置主节点参数脚本
# 当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。
# 以下参数将会在调用脚本时传给脚本:
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
# 目前<state>总是“failover”,
# <role>是“leader”或者“observer”中的一个。 
# 参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通信的
# 这个脚本应该是通用的，能被多次调用，不是针对性的。
# sentinel client-reconfig-script <master-name> <script-path>
sentinel client-reconfig-script mymaster /var/redis/reconfig.sh
```