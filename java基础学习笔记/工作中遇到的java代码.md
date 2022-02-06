##1：不要在 foreach 循环里进行元素的 remove/add 操作

​    如果要进行`remove`操作，可以调用迭代器的 `remove `方法而不是集合类的 remove 方法。因为如果列表在任何时间从结构上修改创建迭代器之后，以任何方式除非通过迭代器自身`remove/add`方法，迭代器都将抛出一个`ConcurrentModificationException`,这就是单线程状态下产生的 **fail-fast 机制**。

   Java8开始，可以使用`Collection#removeIf()`方法删除满足特定条件的元素,如

```java
List<Integer> list = new ArrayList<>();
for (int i = 1; i <= 10; ++i) {
    list.add(i);
}
list.removeIf(filter -> filter % 2 == 0); /* 删除list中的所有偶数 */
System.out.println(list); /* [1, 3, 5, 7, 9] */
```

  如果使用remove元素，则需要使用Iterator方式，如果并发操作，需要对Iterator对象加锁。

```java
List<String> list = new ArrayList<>();
list.add("1");
Iterator<String> iterator = list.iterator();
while(iterator.hasNext()){
   String item = iterator.next();
   if(){
      iterator.remove();
   }
}
```



## 2: Collection.toArray()方法使用的坑&如何反转数组

该方法是一个泛型方法：` T[] toArray(T[] a);` 如果`toArray`方法中没有传递任何参数的话返回的是`Object`类型数组。

```
String [] s= new String[]{
    "dog", "lazy", "a", "over", "jumps", "fox", "brown", "quick", "A"
};
List<String> list = Arrays.asList(s);
Collections.reverse(list);
s=list.toArray(new String[0]);//没有指定类型的话会报错
```

## 3: BigDecimal 来进行浮点数之间的运算

使用BigDecimal时，为了防止精度损失，禁止使用构造方法进行转换，需要使用以下方法：

BigDecimal  recommend1 = BigDecimal .valueOf(0.1)