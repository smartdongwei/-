## java8中的相关函数

## 1.1 Comparator接口

​    Comparator接口现在同时包含了默认方法和静态方法，

(1)新的实例方法包含了下面这些。

1. reversed——对当前的Comparator对象进行逆序排序，并返回排序之后新的Comparator对象。
2. thenComparing ——当两个对象相同时，返回使用另一个Comparator进行比较的Comparator对象。 
3. thenComparingInt、thenComparingDouble、thenComparingLong——这些方法的工作方式和  thenComparing方法类似，不过它们的处理函数是特别针对某些基本数据类型（分别对应于ToIntFunction、ToDoubleFunction和ToLongFunction）的。



(2)新的静态方法包括下面这些。

1.  comparingInt、comparingDouble、comparingLong——它们的工作方式和comparing类似，但接受的函数特别针对某些基本数据类型（分别对应于ToIntFunction、ToDoubleFunction和ToLongFunction）。
2.  naturalOrder——对Comparable对象进行自然排序，返回一个Comparator对象。 nullsFirst、nullsLast——对空对象和非空对象进行比较，你可以指定空对象（null）比非空对象（non-null）小或者比非空对象大，返回值是一个Comparator对象。
3.  reverseOrder——和naturalOrder().reversed()方法类似。

​    

   具体的代码格式为：

```java
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



## 1.2 函数式数据处理(流相关操作)



```java
// 2个列表合并
Stream.of(list1,list2).flatMap(Collection::stream).distinct().forEach(System.out.println)

// Collectors.joining 方法以遭遇元素的顺序拼接元素
List<String> list = Arrays.asList("A","B")  
jobJsons.stream().collect(Collectors.joining(",","[","]"));
// ["A","B"]

// 把string字符串转换成List对象
JSONObject jsonObject = JSONObject.parseObject(fieldMapping);
List<FieldMapperVo> fieldMapper = jsonObject.getJSONArray(type).toJavaList(FieldMapperVo.class);

// 
dataXConfigFieldVOS.stream().collect(Collectors.toMap(
            DataXConfigFieldVO::getName,
            v->v,
            (old,newValue)->old
        ));
```











