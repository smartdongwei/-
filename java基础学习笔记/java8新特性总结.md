# java8知识总结

## 1、java8由哪些新特性？

简单来说，Java8新特性如下所示：

-  Lambda表达式 
- 函数式接口 
- 方法引用与构造器
- 引用 Stream API 
- 接口的默认方法与静态方法 
- 新时间日期API 
- 其他新特性

### 1.1 Lambda表达式的语法

​    Lambda表达式在Java语言中引入了 “->” 操作符， “->” 操作符被称为Lambda表达式的操作符或者箭头 操作符，它将Lambda表达式分为两部分：

- **左侧部分指定了Lambda表达式需要的所有参数。**

​    Lambda表达式本质上是对接口的实现，Lambda表达式的参数列表本质上对应着接口中方法的参数列表。

- **右侧部分指定了Lambda体，即Lambda表达式要执行的功能。**

​    Lambda体本质上就是接口方法具体实现的功能。 

​    我们可以将Lambda表达式的语法总结如下。

**1.语法格式一：无参，无返回值，Lambda体只有一条语句**

```java
Runnable r = () -> System.out.println("Hello Lambda");
@Test
public void test1(){
    Runnable r = () -> System.out.println("Hello Lambda");
    new Thread(r).start();
}

```

**2.语法格式二：Lambda表达式需要一个参数，并且无返回值**

```java
@Test
public void test2(){
    Consumer<String> consumer = (x) -> System.out.println(x);
    consumer.accept("Hello Lambda");
}

```

**3.语法格式四：Lambda需要两个参数，并且有返回值**

```java
BinaryOperator<Integer> bo = (a, b) -> {
System.out.println("函数式接口");
return a + b;
}
```

### 1.2 函数式接口

Lambda表达式需要函数式接口的支持，所以，我们有必要来说说什么是函数式接口。 

> 只包含一个抽象方法的接口，称为函数式接口。  可以通过 Lambda 表达式来创建该接口的对象。（若 Lambda表达式抛出一个受检异常，那么该 异常需要在目标接口的抽象方法上进行声明）。 可以在任意函数式接口上使用 @FunctionalInterface 注解，这样做可以检查它是否是一个函数式 接口，同时 javadoc 也会包含一条声明，说明这个接口是一个函数式接口。 

如下所示：

```java
@FunctionalInterface 
public interface MyFunc  { 
    public T getValue(T t); 
}
```

接下来，我们定义一个操作字符串的方法，其中参数为MyFunc接口实例和需要转换的字符串。

```java
public String handlerString(MyFunc<String> myFunc, String str){
    return myFunc.getValue(str);
}

@Test
public void test6(){
    String str = handlerString((s) -> s.toUpperCase(), "binghe");
    System.out.println(str);
}
```

**注意：作为参数传递 Lambda 表达式：为了将 Lambda 表达式作为参数传递，接收Lambda 表达式 的参数类型必须是与该 Lambda 表达式兼容的函数式接口的类型 。**

### 1.3 使用样例

​    排序相关的代码

```java
@Test
public void test1(){
Collections.sort(employees, (e1, e2) -> {
	if(e1.getAge() == e2.getAge()){
		return e1.getName().compareTo(e2.getName());
	}
	return Integer.compare(e1.getAge(), e2.getAge());
	});
	employees.stream().forEach(System.out::println);
}
```

  **需求1**

 1.声明函数式接口，接口中声明抽象方法 public String getValue(String str); 

 2.声明类TestLambda，类中编写方法使用接口作为参数，将一个字符串转换为大写，并作为方法的返 回值。 

 3.再将一个字符串的第2个和第4个索引位置进行截取子串。  

**实现** 

​    首先，创建一个函数式接口MyFunction，在MyFunction接口上加上注解@FunctionalInterface标识接 口是一个函数式接口。如下所示。

```java
@FunctionalInterface
public interface MyFunction {
    public String getValue(String str);
}
public String stringHandler(String str, MyFunction myFunction){
    return myFunction.getValue(str);
}
@Test
public void test2(){
    String value = stringHandler("binghe", (s) -> s.toUpperCase());
    System.out.println(value);
}

```



**需求 2**

   声明一个带两个泛型的函数式接口，泛型类型为，其中，T作为参数的类型，R作为返回值的类 型。 2.接口中声明对象的抽象方法。 3.在TestLambda类中声明方法。使用接口作为参数计算两个long型参数的和。 4.再就按两个long型参数的乘积。

**实现** 

首先，我们按照需求定义函数式接口MyFunc，如下所示。

```java
@FunctionalInterface
public interface MyFunc<T, R> {
    R getValue(T t1, T t2);
}
public void operate(Long num1, Long num2, MyFunc<Long, Long> myFunc){
    System.out.println(myFunc.getValue(num1, num2));
}
@Test
public void test4(){
    operate(100L, 200L, (x, y) -> x + y);
}
```



### 1.4四大核心函数式接口

| 函数式接口           | 参 数 类 型 | 返回类型 | 使用场景                                                     |
| -------------------- | ----------- | -------- | ------------------------------------------------------------ |
| Consumer消费型接口   |             |          | 对类型为T的对象应用操作，接口定义的方法：void accept(T t)    |
| Supplier供给型接口   |             |          | 返回类型为T的对象，接口定义的方法：T get()                   |
| Function函数式接口   |             |          | 对类型为T的对象应用操作，并R类型的返回结果。接口 定义的方法：R apply(T t) |
| Predicate断言 型接口 |             |          | 确定类型为T的对象是否满足约束条件，并返回boolean 类型的数据。接口定义的方法：boolean test(T t) |

#### 1.4.1 Consumer接口

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
    
    default Consumer<T> andThen(Consumer<? super T> after) {
       Objects.requireNonNull(after);
       return (T t) -> { accept(t); after.accept(t); };
    }
}
public void handlerConsumer(Integer number, Consumer<Integer> consumer){
    consumer.accept(number);
}
@Test
public void test1(){
    this.handlerConsumer(10000, (i) -> System.out.println(i));
}
```

#### 1.4.2 Supplier接口

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
public List<Integer> getNumberList(int num, Supplier<Integer> supplier){
    List<Integer> list = new ArrayList<>();
    for(int i = 0; i < num; i++){
        list.add(supplier.get())
    }
    return list;
}
@Test
public void test2(){
    List<Integer> numberList = this.getNumberList(10, () -> new
    Random().nextInt(100));
    numberList.stream().forEach(System.out::println);
}
```

### 1.5 stream的相关信息

#### 1.5.1 如何创建Stream?

Java8 中的 Collection 接口被扩展，提供了两个获取流的方法： 

**1.获取Stream** 

- ​    default Stream stream() : 返回一个顺序流
- ​    default Stream parallelStream() : 返回一个并行流 

 **2.由数组创建Stream**

 Java8 中的 Arrays 的静态方法 stream() 可以获取数组流：  

​    static Stream stream(T[] array): 返回一个流  

重载形式，能够处理对应基本类型的数组：

 public static IntStream stream(int[] array)

 public static LongStream stream(long[] array)

 public static DoubleStream stream(double[] array)  

**3.由值创建流** 

可以使用静态方法 Stream.of(), 通过显示值创建一个流。它可以接收任意数量的参数。  

public static Stream of(T... values) : 返回一个流  

**4.由函数创建流 由函数创建流可以创建无限流。**

 可以使用静态方法 Stream.iterate() 和Stream.generate(), 创建无限流 。

-  迭代 

public static Stream iterate(final T seed, final UnaryOperator f) 

- 生成 

public static Stream generate(Supplier s) 

#### 1.5.2  Stream的中间操作

##### 1.5.2.1 筛选与切片

| 方法                | 描述                                                    |
| ------------------- | ------------------------------------------------------- |
| filter(Predicate p) | 接收Lambda，从流中排除                                  |
| distinct()          | 筛选，通过流所生成元素的hascode()和equals()去除重复元素 |
| limit(long maxSize) | 截断流，使元素不超过给定的数量                          |
| skip(long n)        | 跳过元素，返回一个扔掉了前n个元素的流                   |

**相关代码如下：**

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
    default Predicate<T> and(Predicate<? super T> other) {
		Objects.requireNonNull(other);
		return (t) -> test(t) && other.test(t);
	}
	default Predicate<T> negate() {
		return (t) -> !test(t);
	}
	default Predicate<T> or(Predicate<? super T> other) {
		Objects.requireNonNull(other);
		return (t) -> test(t) || other.test(t);
	}
	static <T> Predicate<T> isEqual(Object targetRef) {
		return (null == targetRef)
			? Objects::isNull
			: object -> targetRef.equals(object);
	}
}
//内部迭代：在此过程中没有进行过迭代，由Stream api进行迭代
//中间操作：不会执行任何操作
Stream<Person> stream = list.stream().filter((e) -> {
    System.out.println("Stream API 中间操作");
    return e.getAge() > 30;
});

//过滤之后取2个值
list.stream().filter((e) -> e.getAge() >30 ).limit(2).forEach(System.out ::
println);

//跳过前2个值
list.stream().skip(2).forEach(System.out :: println)
    
// distinct 需要实体中重写hashCode（）和 equals（）方法才可以使用。
list.stream().distinct().forEach(System.out :: println)
```



##### 1.5.2.2 映射

| 方法            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| map(Function f) | 接收一个函数作为参数，该函数会映射到每一个元素上，并将其映射成一个新元素 |
| mapToDouble()   |                                                              |
| mapToInt()      |                                                              |
| mapToLong()     |                                                              |
| flatMap()       | 接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流 |

```java
//将流中每一个元素都映射到map的函数中，每个元素执行这个函数，再返回
List<String> list = Arrays.asList("aaa", "bbb", "ccc", "ddd");
list.stream().map((e) -> e.toUpperCase()).forEach(System.out::printf);
//获取Person中的每一个人得名字name，再返回一个集合
List<String> names = this.list.stream().map(Person ::getName).collect(Collectors.toList());

// flatMap 的相关参数
list.stream().flatMap((e) ->s.filterCharacter(e)).forEach(System.out::println);
/**
* 将一个字符串转换为流
*/
public Stream<Character> filterCharacter(String str){
    List<Character> list = new ArrayList<>();
    for (Character ch : str.toCharArray()) {
        list.add(ch);
    }
    return list.stream();
}
```



##### 1.5.2.3 排序

| 方法                    | 描述                               |
| ----------------------- | ---------------------------------- |
| sorted()                | 产生一个新流，其中按自然顺序排序   |
| sorted(Comparator comp) | 产生一个新流，按照比较器是顺序排序 |

```java
// 自然排序
List<Employee> persons = list.stream().sorted().collect(Collectors.toList());
//定制排序
List<Employee> persons1 = list.stream().sorted((e1, e2) -> {
	if (e1.getAge() == e2.getAge()) {
		return 0;
	} else if (e1.getAge() > e2.getAge()) {
		return 1;
	} else {
		return -1;
	}
}).collect(Collectors.toList());

// 自然序逆序元素，使用Comparator 提供的reverseOrder() 方法
list.stream().sorted(Comparator.reverseOrder());
list.stream().sorted(Comparator.comparing(Student::getAge).reversed());
List<Integer> list = Arrays.asList(new Integer[]{1,444,333,5,95436,222,121323,55666,3});

list.sort((o1,o2) -> o1 - o2);
System.out.println(JSONObject.toJSONString(list));
// 逆序排序
list.sort(Comparator.comparingInt(Integer::intValue).reversed());
System.out.println(JSONObject.toJSONString(list));
//  字符串的排序
String[] lsit11 = new String[]{"测试","哈哈","我的","你的","即将"};
Arrays.sort(lsit11);
System.out.println(JSONObject.toJSONString(lsit11));
//
List<String> listStr = Arrays.asList(new String[]{"测试","哈哈","我的","你的","即将"});
listStr.sort(Comparator.comparing(String::toString));
System.out.println(JSONObject.toJSONString(listStr));
// 按照拼音排序
List<String> listStr1 = Arrays.asList(new String[]{"组织在","安安","测试","哈哈","我的","你的","即将"});
listStr1.sort((s1,s2) -> Collator.getInstance(java.util.Locale.CHINA).compare(s1,s2));
System.out.println(JSONObject.toJSONString(listStr1));

//  比较器链 如果2个具有相同的值，则使用第二个 Comparator
List<String> listStr2 = Arrays.asList(new String[]{"组织在","组织A","安安","测试","哈哈","我的","你的","即将"});
listStr2.sort(Comparator.comparing(String::toString).thenComparing(
        (s1,s2) ->Collator.getInstance(java.util.Locale.CHINA).compare(s1,s2)));
System.out.println(JSONObject.toJSONString(listStr2));

// 自定义的 Comparator
List<String> listStr3 = Arrays.asList(new String[]{"组织在","组织A","安安","测试","哈哈","我的","你的","即将"});
Collections.sort(listStr3, new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
        // 这里编写相关的代码
        return 0;
    }
});

```



#### 1.5.3  Stream的终止操作

| 方法                  |                                                              |
| --------------------- | ------------------------------------------------------------ |
| allMatch(Predicate p) | 检查是否匹配所有元素                                         |
| allMatch(Predicate p) | 检查是否匹配所有元素                                         |
| allMatch(Predicate p) | 检查是否没有匹配所有元素                                     |
| findFirst()           | 检查是否没有匹配所有元素                                     |
| findAny()             | 返回当前流中的任意元素                                       |
| count()               | 返回流中元素总数                                             |
| max(Comparator c)     | 返回流中最大值                                               |
| min(Comparator c)     | 返回流中最小值                                               |
| forEach(Consumer c)   | 内部迭代(使用 Collection 接口需要用户去做迭代，称为外部迭代。相 反， Stream API 使用内部迭代) |

```java
// 法表示检查是否匹配所有元素 匹配所有才返回true
boolean match = employees.stream().allMatch((e) ->
    Employee.Stauts.SLEEPING.equals(e.getStauts()));
System.out.println(match);

// 表示检查是否至少匹配一个元素 只要由任意一个满足就返回true
boolean match = employees.stream().anyMatch((e) ->
    Employee.Stauts.SLEEPING.equals(e.getStauts()));
System.out.println(match);

// 只有所有的元素都不符合条件时，才会返回true
boolean match = employees.stream().noneMatch((e) ->
    Employee.Stauts.SLEEPING.equals(e.getStauts()));
System.out.println(match);

```

#### 1.5.4  规约

| 方法                | 描述               |
| ------------------- | ------------------ |
| collect(Collectorc) | 将流转换为其他形式 |

​    Collector接口中方法的实现决定了如何对流执行收集操作(如收集到 List、 Set、 Map)。 Collectors实 用类提供了很多静态方法，可以方便地创建常见收集器实例， 具体方法与实例如下表： 

![image-20210606211157826](E:\study\images\image-20210606211157826.png)

相关的使用方法：

![image-20210606224259594](E:\study\images\image-20210606224259594.png)



## 2 optional类示例

![image-20210607144653803](E:\study\images\image-20210607144653803.png)



## 3 本地时间和时间戳

![image-20210609092112207](E:\study\images\image-20210609092112207.png)