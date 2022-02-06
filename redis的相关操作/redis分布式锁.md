## 1：redis来实现的分布式锁

### Redis几种架构

Redis发展到现在，几种常见的部署架构有：

- 单机模式；
- 主从模式；
- 哨兵模式；
- 集群模式；

从分布式锁的角度来说， 无论是单机模式、主从模式、哨兵模式、集群模式，其原理都是类同的。 只是主从模式、哨兵模式、集群模式的更加的高可用、或者更加高并发。

### 1.1 基于Jedis 的API实现分布式锁

> Redis分布式锁机制，主要借助setnx和expire两个命令完成。

**setnx命令:**

SETNX 是SET if Not eXists的简写。将 key 的值设为 value，当且仅当 key 不存在; 若给定的 key 已经存在，则 SETNX 不做任何动作。

**expire命令:**

expire命令为 key 设置生存时间，当 key 过期时(生存时间为 0 )，它会被自动删除. 其格式为：

> EXPIRE key seconds

#### 1.1.1 基于Jedis API的分布式锁的总体流程：

通过Redis的setnx、expire命令可以实现简单的锁机制：

- key不存在时创建，并设置value和过期时间，返回值为1；成功获取到锁；
- 如key存在时直接返回0，抢锁失败；
- 持有锁的线程释放锁时，手动删除key； 或者过期时间到，key自动删除，锁释放。



需要保障setnx和expire两个操作的原子性，要么全部执行，要么全部不执行，二者不能分开

解决的办法有两种：

- 使用set的命令时，同时设置过期时间，不再单独使用 expire命令
- 使用lua脚本，将加锁的命令放在lua脚本中原子性的执行

#### 1.1.2简单加锁：使用set的命令时，同时设置过期时间

使用set的命令时，同时设置过期时间的示例如下：

> set unlock "234" EX 100 NX

```java
package com.crazymaker.springcloud.standard.lock;


@Slf4j
@Data
@AllArgsConstructor
public class JedisCommandLock {

    private  RedisTemplate redisTemplate;

    private static final String LOCK_SUCCESS = "OK";
    private static final String SET_IF_NOT_EXIST = "NX";
    private static final String SET_WITH_EXPIRE_TIME = "PX";

    /**
     * 尝试获取分布式锁
     * @param jedis Redis客户端
     * @param lockKey 锁
     * @param requestId 请求标识
     * @param expireTime 超期时间
     * @return 是否获取成功
     */
    public static   boolean tryGetDistributedLock(Jedis jedis, String lockKey, String requestId, int expireTime) {

        String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);
        if (LOCK_SUCCESS.equals(result)) {
            return true;
        }
        return false;
    }
 
}
```

**这个set()方法一共有五个形参：**

- 第一个为key，我们使用key来当锁，因为key是唯一的。

- 第二个为value，我们传的是requestId，很多童鞋可能不明白，有key作为锁不就够了吗，为什么还要用到value？原因就是我们在上面讲到可靠性时，分布式锁要满足第四个条件解铃还须系铃人，通过给value赋值为requestId，我们就知道这把锁是哪个请求加的了，在解锁的时候就可以有依据。

  > requestId可以使用`UUID.randomUUID().toString()`方法生成。

- 第三个为nxxx，这个参数我们填的是NX，意思是SET IF NOT EXIST，即当key不存在时，我们进行set操作；若key已经存在，则不做任何操作；

- 第四个为expx，这个参数我们传的是PX，意思是我们要给这个key加一个过期的设置，具体时间由第五个参数决定。

- 第五个为time，与第四个参数相呼应，代表key的过期时间。

#### 1.1.3 基于Jedis 的API实现简单解锁代码

```java
package com.crazymaker.springcloud.standard.lock;


@Slf4j
@Data
@AllArgsConstructor
public class JedisCommandLock {

  

    private static final Long RELEASE_SUCCESS = 1L;

    /**
     * 释放分布式锁
     * @param jedis Redis客户端
     * @param lockKey 锁
     * @param requestId 请求标识
     * @return 是否释放成功
     */
    public static boolean releaseDistributedLock(Jedis jedis, String lockKey, String requestId) {

        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));

        if (RELEASE_SUCCESS.equals(result)) {
            return true;
        }
        return false;

    }

}
```

其实很简单，首先获取锁对应的value值，检查是否与requestId相等，如果相等则删除锁（解锁）。

第一行代码，我们写了一个简单的Lua脚本代码。

第二行代码，我们将Lua代码传到`jedis.eval()`方法里，并使参数KEYS[1]赋值为lockKey，ARGV[1]赋值为requestId。eval()方法是将Lua代码交给Redis服务端执行

### 1.2基于纯Lua脚本的分布式锁的执行流程

加锁和删除锁的操作，使用纯lua进行封装，保障其执行时候的原子性。

#### 1.2.1加锁的Lua脚本： lock.lua

```lua
--- -1 failed
--- 1 success
---
local key = KEYS[1]
local requestId = KEYS[2]
local ttl = tonumber(KEYS[3])
local result = redis.call('setnx', key, requestId)
if result == 1 then
    --PEXPIRE:以毫秒的形式指定过期时间
    redis.call('pexpire', key, ttl)
else
    result = -1;
    -- 如果value相同，则认为是同一个线程的请求，则认为重入锁
    local value = redis.call('get', key)
    if (value == requestId) then
        result = 1;
        redis.call('pexpire', key, ttl)
    end
end
--  如果获取锁成功，则返回 1
return result
```

#### 1.2.2解锁的Lua脚本： unlock.lua：

```lua
--- -1 failed
--- 1 success

-- unlock key
local key = KEYS[1]
local requestId = KEYS[2]
local value = redis.call('get', key)
if value == requestId then
    redis.call('del', key);
    return 1;
end
return -1
```

#### 1.2.3在Java中调用lua脚本，完成加锁操作

```java
package com.crazymaker.springcloud.standard.lock;

import com.crazymaker.springcloud.common.exception.BusinessException;
import com.crazymaker.springcloud.common.util.IOUtil;
import com.crazymaker.springcloud.standard.context.SpringContextUtil;
import com.crazymaker.springcloud.standard.lua.ScriptHolder;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.data.redis.core.script.RedisScript;

import java.util.ArrayList;
import java.util.List;

@Slf4j
public class InnerLock {

    private RedisTemplate redisTemplate;


    public static final Long LOCKED = Long.valueOf(1);
    public static final Long UNLOCKED = Long.valueOf(1);
    public static final int EXPIRE = 2000;

    String key;
    String requestId;  // lockValue 锁的value ,代表线程的uuid

    /**
     * 默认为2000ms
     */
    long expire = 2000L;


    private volatile boolean isLocked = false;
    private RedisScript lockScript;
    private RedisScript unLockScript;


    public InnerLock(String lockKey, String requestId) {
        this.key = lockKey;
        this.requestId = requestId;
        lockScript = ScriptHolder.getLockScript();
        unLockScript = ScriptHolder.getUnlockScript();
    }

    /**
     * 抢夺锁
     */
    public void lock() {
        if (null == key) {
            return;
        }
        try {
            List<String> redisKeys = new ArrayList<>();
            redisKeys.add(key);
            redisKeys.add(requestId);
            redisKeys.add(String.valueOf(expire));

            Long res = (Long) getRedisTemplate().execute(lockScript, redisKeys);
            isLocked = false;
        } catch (Exception e) {
            e.printStackTrace();
            throw BusinessException.builder().errMsg("抢锁失败").build();
        }
    }

    /**
     * 有返回值的抢夺锁
     *
     * @param millisToWait
     */
    public boolean lock(Long millisToWait) {
        if (null == key) {
            return false;
        }
        try {
            List<String> redisKeys = new ArrayList<>();
            redisKeys.add(key);
            redisKeys.add(requestId);
            redisKeys.add(String.valueOf(millisToWait));
            Long res = (Long) getRedisTemplate().execute(lockScript, redisKeys);

            return res != null && res.equals(LOCKED);
        } catch (Exception e) {
            e.printStackTrace();
            throw BusinessException.builder().errMsg("抢锁失败").build();
        }

    }

    //释放锁
    public void unlock() {
        if (key == null || requestId == null) {
            return;
        }
        try {
            List<String> redisKeys = new ArrayList<>();
            redisKeys.add(key);
            redisKeys.add(requestId);
            Long res = (Long) getRedisTemplate().execute(unLockScript, redisKeys);

//            boolean unlocked = res != null && res.equals(UNLOCKED);


        } catch (Exception e) {
            e.printStackTrace();
            throw BusinessException.builder().errMsg("释放锁失败").build();

        }
    }

    private RedisTemplate getRedisTemplate() {
        if(null==redisTemplate)
        {
            redisTemplate= (RedisTemplate) SpringContextUtil.getBean("stringRedisTemplate");
        }
        return redisTemplate;
    }
}
```

## 2: redisson实现分布式锁

### 2.1特性 & 功能：

- 支持 Redis 单节点（single）模式、哨兵（sentinel）模式、主从（Master/Slave）模式以及集群（Redis Cluster）模式
- 程序接口调用方式采用异步执行和异步流执行两种方式
- 数据序列化，Redisson 的对象编码类是用于将对象进行序列化和反序列化，以实现对该对象在 Redis 里的读取和存储
- 单个集合数据分片，在集群模式下，Redisson 为单个 Redis 集合类型提供了自动分片的功能
- 提供多种分布式对象，如：Object Bucket，Bitset，AtomicLong，Bloom Filter 和 HyperLogLog 等
- 提供丰富的分布式集合，如：Map，Multimap，Set，SortedSet，List，D    】
- 
- eque，Queue 等
- 分布式锁和同步器的实现，可重入锁（Reentrant Lock），公平锁（Fair Lock），联锁（MultiLock），红锁（Red Lock），信号量（Semaphonre），可过期性信号锁（PermitExpirableSemaphore）等
- 提供先进的分布式服务，如分布式远程服务（Remote Service），分布式实时对象（Live Object）服务，分布式执行服务（Executor Service），分布式调度任务服务（Schedule Service）和分布式映射归纳服务（MapReduce）

### 2.2 锁的相关方法

redisson 提供了`lock()`和`tryLock()`，`tryLock(long time, TimeUnit unit)`，`tryLock(long waitTime, long leaseTime, TimeUnit unit)`方法。

1. `lock()`：会阻塞未获取锁的请求，默认持有`30s`锁，但当业务方法在30s内没有执行完时，会有`看门狗（默认每隔10s）`给当前锁续时`30s`。
2. `tryLock()`：尝试获取锁，获取不到则直接返回获取失败，默认持有`30s`锁，但当业务方法在30s内没有执行完时，会有`看门狗（默认每隔10s）`给当前锁续时`30s`。
3. `tryLock(long time, TimeUnit unit)`：尝试获取锁，等待`time TimeUnit`，默认持有`30s`锁，但当业务方法在30s内没有执行完时，会有`看门狗（默认每隔10s）`给当前锁续时`30s`。
4. `tryLock(long waitTime, long leaseTime, TimeUnit unit)`：尝试获取锁，等待`waitTime TimeUnit`，`锁最长持有leaseTime TimeUnit`，当业务方法在`leaseTime TimeUnit`时长内没有执行完时，会强制解锁。