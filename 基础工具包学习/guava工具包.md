# 1：主要内容

![img](E:\study\images\v2-a9080e8e6c682dbc02b8006e8679271d_720w.jpg)

# 2：集合框架

## 2.1 集合工具类

```java
// 创建一个ArrayList，可以传不定长参数
List<Integer> arrayList = Lists.newArrayList(1, 2, 3);

// 创建一个LinkedList，传一个iterable（可迭代）变量
List<Integer> linkedList = Lists.newLinkedList(arrayList);

// 翻转一个集合，原集合不变
List<Integer> reversed = Lists.reverse(arrayList);

// 集合变换，传一个function对象，相当于stream操作
List<Integer> transformed = Lists.transform(arrayList, i -> i * 2);

// 创建一个HashSet
Set<Integer> hashSet = Sets.newHashSet(1, 2, 3);

// 创建一个TreeSet
Set<Integer> treeSet = Sets.newTreeSet(Sets.newHashSet(4, 2, 3));

// 求两个Set的并集，返回的是一个只读的视图
Sets.SetView<Integer> union = Sets.union(hashSet, treeSet);

// 求两个Set的交集，返回的是一个只读的视图
Sets.SetView<Integer> intersection = Sets.intersection(hashSet, treeSet);

// 求两个Set的差集，返回的是一个只读的视图
Sets.SetView<Integer> difference = Sets.difference(hashSet, treeSet);

// jdk9创建不可变map的api
Map<String, Object> map0 = Map.of("a", 1, "b", 2, "c", 3);
Map<String, Object> map1 = Map.of("a", 1, "b", 2, "d", 4);

// 求两个map的差异
MapDifference<String, Object> mapDifference = Maps.difference(map0, map1);
```



## 2.2 扩展集合

主要扩展了Multiset（背包）和Multimap（多值映射）

```java
// Multiset是支持重复元素的集合，即背包
// 创建背包
Multiset<String> multiset = HashMultiset.create();

// 向背包中添加一个元素
multisetu.add("one");

// 向背包中添加多个相同元素
multiset.add("two", 4);

// 查询元素个数
int oneCount = multiset.count("one");

// 删除一个元素若干次
multiset.remove("two", 2);

// Multimap是一个key对应多个value的map
// 创建一个multimap
Multimap<String, Object> multimap = HashMultimap.create();

// 可以在相同的key下put值，put后map元素为{a=[1, 2]}
multimap.put("a", 1);
multimap.put("a", 2);

// 可以通过putAll为一个key一次性放置多个value，value不保证有序
multimap.putAll("b", Sets.newHashSet(3, 4, 5));
```

## 2.3 不可变集合

​    可以创建不可变集合，这样可以保证集合中的元素不发生变化。如果使用的jdk是9以上版本，也可以直接使用jdk自带的api。

```java
// 创建不可变list，上面为guava api，下面为jdk9 api
List<Integer> immutableList = ImmutableList.of(1, 2, 3);
List<Integer> jdkImmutableList = List.of(1, 2, 3);

// 创建不可变set，上面为guava api，下面为jdk9 api
Set<Integer> immutableSet = ImmutableSet.of(1, 2, 3);
Set<Integer> jdkImmutableSet = Set.of(1, 2, 3);

// 创建不可变map，上面为guava api，下面为jdk9 api
Map<String, Integer> immutableMap = ImmutableMap.of("a", 1, "b", 2, "c", 3);
Map<String, Integer> jdkImmutableMap = Map.of("a", 1, "b", 2, "c", 3);
```

# 3：图结构处理

graph包中提供了三种主要的graph类型，即Graph、ValueGraph和Network。

Graph是最基本的图，它的边仅仅是为了联结顶点，本身没有标识或属性。

ValueGraph的边有自己的值（或者说权重），当你需要关注边的信息时可以使用该类型。

Network指一对节点间不止一个边联结，并且这些边可以通过唯一标识进行区分。

下面给出一些常用的方法。

```java
// 创建一个可变的无向图
MutableGraph<Integer> graph = GraphBuilder.undirected().build();

// 创建一个可变的有向值图
MutableValueGraph<City, Distance> roads = ValueGraphBuilder.directed()
        .incidentEdgeOrder(ElementOrder.stable())
        .build();

// 创建一个可变的有向网络
MutableNetwork<Webpage, Link> webSnapshot = NetworkBuilder.directed()
        .allowsParallelEdges(true)
        .nodeOrder(ElementOrder.natural())
        .expectedNodeCount(100000)
        .expectedEdgeCount(1000000)
        .build();

// 创建一个不可变的图
ImmutableGraph<Country> countryAdjacencyGraph =
        GraphBuilder.undirected()
                .<Country>immutable()
                .putEdge(FRANCE, GERMANY)
                .putEdge(FRANCE, BELGIUM)
                .putEdge(GERMANY, BELGIUM)
                .addNode(ICELAND)
                .build();

// 有向图/无向图添加边
directedGraph.addEdge(nodeU, nodeV, edgeUV_a);
directedGraph.addEdge(nodeU, nodeV, edgeUV_b);
directedGraph.addEdge(nodeV, nodeU, edgeVU);

undirectedGraph.addEdge(nodeU, nodeV, edgeUV_a);
undirectedGraph.addEdge(nodeU, nodeV, edgeUV_b);
undirectedGraph.addEdge(nodeV, nodeU, edgeVU);
```

# 4：并发工具

## 4.1 ListenableFuture

ListenableFuture是jdk的Future接口的一个扩展，guava团队建议在所有使用Future的地方，使用ListenableFuture进行代替。

ListenableFuture增加了Future执行成功或失败的监听器，可以在计算完成时即时进行回调，使得并发编程的代码更加高效。

```java
// 使用listeningDecorator方法包装线程池
ListeningExecutorService service = MoreExecutors.listeningDecorator(Executors.newFixedThreadPool(10));

// 调用submit方法，即可返回ListenableFuture
ListenableFuture<Explosion> explosion = service.submit(
        new Callable<Explosion>() {
            public Explosion call() {
                return pushBigRedButton();
            }
        });

// 通过Futures工具类的addCallback方法，为ListenableFuture添加执行成功和失败的回调
Futures.addCallback(
        explosion,
        new FutureCallback<Explosion>() {

            // 执行成功
            public void onSuccess(Explosion explosion) {
                walkAwayFrom(explosion);
            }

            // 执行失败
            public void onFailure(Throwable thrown) {
                battleArchNemesis();
            }
        },
        service);
```

### 4.2 Service

Service接口代表了对象的运行状态，它提供了开始和停止的方法。计时器、RPC服务器等服务均可以实现此接口，以便管理开启和结束等状态。

Service提供了五个正常的状态和一个失败的状态，即： - Service.State.NEW 新建 - Service.State.STARTING 开启 - Service.State.RUNNING 运行中 - Service.State.STOPPING 正在停止 - Service.State.TERMINATED 中止 - Service.State.FAILED 失败

正常服务的生命周期应该为

```text
新建->开启->运行中->停止->中止
```

### I/O

封装了一些更方便的I/O工具。

```java
// 将整个输入流读为byte数组，不会关闭流
byte[] bytes = ByteStreams.toByteArray(inputStream);

// 将整个输入流中的字节复制到输出流中，不会关闭流或清空缓存区
long length = ByteStreams.copy(inputStream, outputStream);

// 将整个输入流/可读对象中的数据按行读出，不包含换行符
List<String> lines = CharStreams.readLines(new InputStreamReader(inputStream));

// 将整个输入流/可读对象中的数据读成一个字符串，不关闭流
String s = CharStreams.toString(new InputStreamReader(inputStream));

// 读取一个文本文件所有行
ImmutableList<String> textLines = Files.asCharSource(file, Charsets.UTF_8)
        .readLines();

// 计算一个文件里出现的不同单词的次数
Multiset<String> wordOccurrences = HashMultiset.create(
        Splitter.on(CharMatcher.whitespace())
                .trimResults()
                .omitEmptyStrings()
                .split(Files.asCharSource(file, Charsets.UTF_8).read()));
```

# 5. 哈希

提供更复杂的哈希算法，包括Bloom filter数据结构，该结构通过允许少量的错误来节省大量的存储空间。

```java
// 要计算hashCode的实体类
class Person {
    int id;
    String firstName;
    String lastName;
    int birthYear;
}


// 将对象分解为基础属性值的数据漏斗
Funnel<Person> personFunnel = (person, into) -> into
        .putInt(person.id)
        .putString(person.firstName, Charsets.UTF_8)
        .putString(person.lastName, Charsets.UTF_8)
        .putInt(person.birthYear);

// 初始化哈希函数
HashFunction hf = Hashing.sha256();

// 计算哈希值
HashCode hc = hf.newHasher()
        .putLong(id)
        .putString(name, Charsets.UTF_8)
        .putObject(person, personFunnel)
        .hash();

// 创建BloomFilter
BloomFilter<Person> friends = BloomFilter.create(personFunnel, 500, 0.01);

// 向Bloom Filter对象中添加元素
for (Person friend : friendsList) {
    friends.put(friend);
}

// 判断Bloom Filter里是否可能包含某个元素，这里牺牲一定的准确度来换取空间和执行效率
if (friends.mightContain(dude)) {
    // 如果不包含元素并且代码执行到这里的概率是1%。
}
```

# 6. 缓存

本地缓存（内存级别的缓存），支持各种方式的过期行为，不会将数据存到硬盘。

```java
// 创建缓存
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
        .maximumSize(1000)
        .build(new CacheLoader<>() {
            @Override
            public Graph load(Key key) throws Exception {
                // 通过key来生成值，这个过程消耗资源比较大，因此需要缓存
                return createExpensiveGraph(key);
            }
        });

// 读取缓存中的值
try {
    // 尝试获取缓存中的值，如果没有值，通过CacheLoader执行load方法添加值
    graphs.get(key);
} catch (ExecutionException e) {
    // 异常处理
}
```

### 7. 事件总线

使组件间保持“发布-订阅”风格的通信，不需要将组件互相注册。它是为了替换使用显式注册发布事件的方式，而非通用的“发布-订阅”系统。

```java
// 事件处理器
class EventBusChangeRecorder {
    @Subscribe
    public void recordCustomerChange(ChangeEvent e) {
        recordChange(e.getChange());
    }
}

// 订阅事件
EventBus eventBus = new EventBus("event");
eventBus.register(new EventBusChangeRecorder());

// 创建事件
ChangeEvent e = getChangeEvent();

// 发布事件
eventBus.post(e);
```