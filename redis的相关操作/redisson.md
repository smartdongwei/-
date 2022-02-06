# redisson的相关知识

github上的官方文档： [目录 · redisson/redisson Wiki · GitHub](https://github.com/redisson/redisson/wiki/目录)

## 1 相关的数据结构

### 1.1 映射（Map）

基于Redis的Redisson的分布式映射结构的[`RMap`](http://static.javadoc.io/org.redisson/redisson/3.10.6/org/redisson/api/RMap.html) Java对象实现了`java.util.concurrent.ConcurrentMap`接口和`java.util.Map`接口。与HashMap不同的是，RMap保持了元素的插入顺序。

```java
RMap<String, SomeObject> map = redisson.getMap("anyMap");
SomeObject prevObject = map.put("123", new SomeObject());
SomeObject currentObject = map.putIfAbsent("323", new SomeObject());
SomeObject obj = map.remove("123");

map.fastPut("321", new SomeObject());
map.fastRemove("321");

RFuture<SomeObject> putAsyncFuture = map.putAsync("321");
RFuture<Void> fastPutAsyncFuture = map.fastPutAsync("321");

map.fastPutAsync("321", new SomeObject());
map.fastRemoveAsync("321");
```

映射的字段锁的用法：

```java
RMap<MyKey, MyValue> map = redisson.getMap("anyMap");
MyKey k = new MyKey();
RLock keyLock = map.getLock(k);
keyLock.lock();
try {
   MyValue v = map.get(k);
   // 其他业务逻辑
} finally {
   keyLock.unlock();
}

RReadWriteLock rwLock = map.getReadWriteLock(k);
rwLock.readLock().lock();
try {
   MyValue v = map.get(k);
   // 其他业务逻辑
} finally {
   keyLock.readLock().unlock();
}
```

### 1.2 多值映射（Multimap）

​    基于Redis的Redisson的分布式`RMultimap` Java对象允许Map中的一个字段值包含多个元素。

#### 1.2.1. 基于集（Set）的多值映射（Multimap）

基于Set的Multimap不允许一个字段值包含有重复的元素。

```java
RSetMultimap<SimpleKey, SimpleValue> map = redisson.getSetMultimap("myMultimap");
map.put(new SimpleKey("0"), new SimpleValue("1"));
map.put(new SimpleKey("0"), new SimpleValue("2"));
map.put(new SimpleKey("3"), new SimpleValue("4"));

Set<SimpleValue> allValues = map.get(new SimpleKey("0"));

List<SimpleValue> newValues = Arrays.asList(new SimpleValue("7"), new SimpleValue("6"), new SimpleValue("5"));
Set<SimpleValue> oldValues = map.replaceValues(new SimpleKey("0"), newValues);

Set<SimpleValue> removedValues = map.removeAll(new SimpleKey("0"));
```

#### 1.2.2. 基于列表（List）的多值映射（Multimap）

基于List的Multimap在保持插入顺序的同时允许一个字段下包含重复的元素。

```java
RListMultimap<SimpleKey, SimpleValue> map = redisson.getListMultimap("test1");
map.put(new SimpleKey("0"), new SimpleValue("1"));
map.put(new SimpleKey("0"), new SimpleValue("2"));
map.put(new SimpleKey("0"), new SimpleValue("1"));
map.put(new SimpleKey("3"), new SimpleValue("4"));

List<SimpleValue> allValues = map.get(new SimpleKey("0"));

Collection<SimpleValue> newValues = Arrays.asList(new SimpleValue("7"), new SimpleValue("6"), new SimpleValue("5"));
List<SimpleValue> oldValues = map.replaceValues(new SimpleKey("0"), newValues);

List<SimpleValue> removedValues = map.removeAll(new SimpleKey("0"));
```

### 1.3 集（Set）

基于Redis的Redisson的分布式Set结构的`RSet` Java对象实现了`java.util.Set`接口。通过元素的相互状态比较保证了每个元素的唯一性。该对象的最大容量受Redis限制，最大元素数量是`4 294 967 295`个。

```
RSet<SomeObject> set = redisson.getSet("anySet");
set.add(new SomeObject());
set.remove(new SomeObject());
```

### 1.4 有序集（SortedSet）

基于Redis的Redisson的分布式`RSortedSet` Java对象实现了`java.util.SortedSet`接口。在保证元素唯一性的前提下，通过比较器（Comparator）接口实现了对元素的排序。

```java
RSortedSet<Integer> set = redisson.getSortedSet("anySet");
set.trySetComparator(new MyComparator()); // 配置元素比较器
set.add(3);
set.add(1);
set.add(2);

set.removeAsync(0);
set.addAsync(5);
```

### 1.5 列表（List）

基于Redis的Redisson分布式列表（List）结构的`RList` Java对象在实现了`java.util.List`接口的同时，确保了元素插入时的顺序。该对象的最大容量受Redis限制，最大元素数量是`4 294 967 295`个。

```java
RList<SomeObject> list = redisson.getList("anyList");
list.add(new SomeObject());
list.get(0);
list.remove(new SomeObject());
```

### 1.6 队列（Queue）

基于Redis的Redisson分布式无界队列（Queue）结构的`RQueue` Java对象实现了`java.util.Queue`接口。尽管`RQueue`对象无初始大小（边界）限制，但对象的最大容量受Redis限制，最大元素数量是`4 294 967 295`个。

```java
RQueue<SomeObject> queue = redisson.getQueue("anyQueue");
queue.add(new SomeObject());
SomeObject obj = queue.peek();
SomeObject someObj = queue.poll();
```

### 1.7 双端队列（Deque）

基于Redis的Redisson分布式无界双端队列（Deque）结构的`RDeque` Java对象实现了`java.util.Deque`接口。尽管`RDeque`对象无初始大小（边界）限制，但对象的最大容量受Redis限制，最大元素数量是`4 294 967 295`个。

```java
RDeque<SomeObject> queue = redisson.getDeque("anyDeque");
queue.addFirst(new SomeObject());
queue.addLast(new SomeObject());
SomeObject obj = queue.removeFirst();
SomeObject someObj = queue.removeLast();
```

### 1.8 阻塞队列（Blocking Queue）

基于Redis的Redisson分布式无界阻塞队列（Blocking Queue）结构的`RBlockingQueue` Java对象实现了`java.util.concurrent.BlockingQueue`接口。尽管`RBlockingQueue`对象无初始大小（边界）限制，但对象的最大容量受Redis限制，最大元素数量是`4 294 967 295`个。

```java
RBlockingQueue<SomeObject> queue = redisson.getBlockingQueue("anyQueue");
queue.offer(new SomeObject());

SomeObject obj = queue.peek();
SomeObject someObj = queue.poll();
SomeObject ob = queue.poll(10, TimeUnit.MINUTES);
```

`poll`, `pollFromAny`, `pollLastAndOfferFirstTo`和`take`方法内部采用话题订阅发布实现，在Redis节点故障转移（主从切换）或断线重连以后，内置的相关话题监听器将自动完成话题的重新订阅。

### 1.9 有界阻塞队列（Bounded Blocking Queue）

基于Redis的Redisson分布式有界阻塞队列（Bounded Blocking Queue）结构的`RBoundedBlockingQueue` Java对象实现了`java.util.concurrent.BlockingQueue`接口。该对象的最大容量受Redis限制，最大元素数量是`4 294 967 295`个。队列的初始容量（边界）必须在使用前设定好。

```javascript
RBoundedBlockingQueue<SomeObject> queue = redisson.getBoundedBlockingQueue("anyQueue");
// 如果初始容量（边界）设定成功则返回`真（true）`，
// 如果初始容量（边界）已近存在则返回`假（false）`。
queue.trySetCapacity(2);

queue.offer(new SomeObject(1));
queue.offer(new SomeObject(2));
// 此时容量已满，下面代码将会被阻塞，直到有空闲为止。
queue.put(new SomeObject());

SomeObject obj = queue.peek();
SomeObject someObj = queue.poll();
SomeObject ob = queue.poll(10, TimeUnit.MINUTES);
```

`poll`, `pollFromAny`, `pollLastAndOfferFirstTo`和`take`方法内部采用话题订阅发布实现，在Redis节点故障转移（主从切换）或断线重连以后，内置的相关话题监听器将自动完成话题的重新订阅。