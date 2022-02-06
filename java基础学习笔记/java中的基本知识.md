# 1: java中IO流的相关知识

## 一、概念

  I/O即是输入/输出的缩写，是计算机把各个存储中的数据写入写出的过程。

1：按照流的操作颗粒可分为以下2类：

- 字节流：以字节为单元，可操作任何数据，相关的操作抽象类为 InputStream和OutputStream
- 字符流：以字符为单元，只能操作纯字符数据，相关的操作抽象类为 Reader 和 writer

  1字符 = 2 字节； 1字节(byte) = 8 位(bit)；一个汉字占两个字节的长度

2：按照流的角色可以分为以下2类：

- 节点流：可以从特定的IO设备读/写数据的流

- 处理流：用于对一个已经存在的流进行连接和封装，通过封装后的流来实现数据的读/写功能。

  ​

## 二、常见流的相关信息

  InputStream/Reader，OutputStream/Writer是整个IO体系的基类

| 分类    | 字节输入流                | 字节输出流                 |       字符输入流       |       字符输出流        |
| ----- | :------------------- | --------------------- | :---------------: | :----------------: |
| 抽象基类  | InputStream          | OutputStream          |      Reader       |       Writer       |
| 访问文件  | FileInputStream      | FileOutputStream      |    FileReader     |     FileWriter     |
| 访问数组  | ByteArrayInputStream | ByteArrayOutputStream |  CharArrayReader  |  CharArrayWriter   |
| 访问管道  | PipedInputStream     | PipedOutPutStream     |    PipedReader    |    PipedWriter     |
| 访问字符串 |                      |                       |   StringReader    |    StringWriter    |
| 缓冲流   | BufferedInputStream  | BufferedOutputStream  |   BufferReader    |    BufferWriter    |
| 转换流   |                      |                       | InputStreamReader | OutputStreamWriter |
| 抽象基类  | FilterInputStream    | FilterOutputStream    |   FilterReader    |    FilterWriter    |
| 特殊流   | DataInputStream      | DataOutputStream      |                   |                    |
| 打印流   |                      | PrintStream           |                   |    PrintWriter     |



## 三、常见IO操作的实战

### 3.1 访问操作文件(FileInputStream/FileReader，FileOutputStream/FileWriter )

```java
public static void main(String[] args) throws Exception{
        // 使用字节流流来读取相关数据 中文可能存在乱码情况
        FileInputStream in = null;
        FileOutputStream out = null;
        int b = 0;
        try{
            in = new FileInputStream(new File("C:\\Users\\Administrator\\Desktop\\wdw\\gadataproxy_error.log.2021-01-19"));
            File outFile = new File("C:\\Users\\Administrator\\Desktop\\wdw\\test.log");
            if(!outFile.exists()){
                outFile.createNewFile();
            }
            out = new FileOutputStream(outFile);
            while((b = in.read()) !=-1){
//                System.out.println((char) b);
                out.write((char) b);
                // 如果存在中文情况
            }
        }catch (Exception e){
            System.out.println(e.getMessage());
        }finally {
            if(in != null){
                in.close();
            }
            if(out != null){
                out.close();
            }
        }

    }
```



### 3.2 缓存流的使用

```java
// 缓存流的使用
BufferedReader br = null;
try{
    br = new BufferedReader(new FileReader("C:\\Users\\Administrator\\Desktop\\wdw\\gadataproxy_error.log.2021-01-19"));
    String buffer = "";
    while((buffer = br.readLine()) != null){
        System.out.println(buffer);
    }
}catch (Exception e){
    System.out.println(e.getMessage());
}finally {
    if(br != null){
        br.close();
    }
}
```



### 3.3 转换流的使用

  转换流就是把 键盘等相关的输入转换对应的输入流，字节流转换成字符流

```java
public static void main(String[] args) throws Exception{
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    String line = null;
    while((line = br.readLine())!= null){
        System.out.println(line);
    }
}
```



### 3.4 RandomAccessFile读取写入文件以及文件锁

1：读取的时候加锁

```java
File file = new File(filePath);
//这个是文件锁的相关内容
FileLock fileLock = null;
FileChannel fileChannel = null;
try{
    RandomAccessFile randomAccessFile = new RandomAccessFile(file, "rw");
    if(!fileName.startsWith("task@")){
        // lastTimeFileSize 是上次读取的文件位数，本次读取从上次的位置开始读
        randomAccessFile.seek(lastTimeFileSize);
    }else{
        fileChannel =  randomAccessFile.getChannel();
        int num = 0;
        while(num < 10){
            try{
                num ++;
                // 尝试给该文件加锁
                fileLock = fileChannel.tryLock();
                if(fileLock != null){
                    break;
                }else{
                    Thread.sleep(500);
                }
            }catch (Exception e){
                logger.info("文件正在被其它线程占用,等待0.5秒");
                Thread.sleep(500);
            }
        }
    }
    String tmp = "";
    while ((tmp = randomAccessFile.readLine()) != null){
        resultList.add(new String(tmp.getBytes("ISO-8859-1"),"UTF-8"));
    }
    if(fileLock != null){
        // 清空文件中的数据
        randomAccessFile.setLength(0);
        fileLock.release();
    }else{
        lastTimeFileSize = randomAccessFile.length();
    }
    if(fileChannel != null){
        fileChannel.close();
    }
    randomAccessFile.close();
}catch (Exception e){
    logger.error("读取文件报错"+ ExceptionUtil.getExceptionTrace(e));
}
```

2：写文件的时候加锁

```java
File tmpfile =new File(filepath);
    if(!tmpfile.exists()){
        tmpfile.createNewFile();
    }
    // 20210205 新增文件的读写锁
    RandomAccessFile randomAccessFile = new RandomAccessFile(tmpfile,"rw");
    FileChannel fileChannel =  randomAccessFile.getChannel();
    FileLock fileLock = null;
    int num = 0;
    while(num < 10){
        try{
            num ++;
            fileLock = fileChannel.tryLock();
            if(fileLock != null){
                break;
            }else{
                Thread.sleep(500);
            }
        }catch (Exception e){
            LOG.info("文件正在被其它线程占用,等待0.5秒");
            Thread.sleep(500);
        }
    }
    // 清空文件中的数据
    randomAccessFile.setLength(0);
    randomAccessFile.write(reportConfig.toJSON().toString().getBytes("UTF-8"));
    if(fileLock != null){
        fileLock.release();
    }
    fileChannel.close();
    randomAccessFile.close();
    randomAccessFile = null;
} catch (Exception e) {
    LOG.error("生成对账信息报错："+e.getMessage());
}
```



### 3.5 excel文件的读写操作(使用easyexcel的插件)

   该插件是阿里的开源插件，方便对excel文件进行读写操作，该插件的官方文档的地址为:

https://www.yuque.com/easyexcel/doc/easyexcel, 详细使用方法见该文档

```java
ExcelWriter excelWriter = null;
try{
  File excelFile = new File(userDir+File.separator+"data/应用血缘关系详细信息.xlsx");
  excelFile.deleteOnExit();
  excelWriter = EasyExcel.write(excelFile).build();
  List<ApplicationSystemSource> list = getData(new File(file+File.separator+txtName));
  // 然后再写excel文档
  WriteSheet writeSheetTwo =         EasyExcel.writerSheet(num,txtName.replaceAll(".txt","")).head(ApplicationSystemSource.class)
    .build();
  excelWriter.write(list,writeSheetTwo);
}catch(Exception e){
  
}finally{
  if(excelWriter != null){
     excelWriter.finish();
  }
}
```



### 3.6  把字符串转换成zip的内存字节流

  把字符串压缩到内存字节流中，方便内存中使用

```java
// 传输过程需要使用字节流
String xml= "asddsasdasd";
byte[] dd = xml.getBytes(StandardCharsets.UTF_8);
GZIPOutputStream zos = null;
ByteArrayOutputStream bout = null;
try{
    bout = new ByteArrayOutputStream();
    zos = new GZIPOutputStream(new BufferedOutputStream(bout));
    zos.write(dd);
}catch (Exception e){
    logger.error("生成zip字节流报错："+ExceptionUtil.getExceptionTrace(e));
}finally {
    if(zos != null){
        zos.close();
    }
}
byte[] zipData = bout.toByteArray();
```



### 3.7 错误信息输出

 错误信息输出成字符串，使用PrintWriter过滤

```java
public static String getExceptionTrace(Exception e){
   StringWriter trace=new StringWriter();
   e.printStackTrace(new PrintWriter(trace));
   return trace.toString();
}
```

### 3.8 官方插件的使用

   在该包 org.apache.commons.io中存在 FileUtils 类，该类中封装了相关的操作。



# 2: java中容器的相关知识

## 2.1：HashMap相关方法的介绍

### 2.1.1 put方法的解释 

1，懒汉式，第一次put才初始化table通数组

2，计算hash及桶下标

3，未发生hash碰撞，直接放入桶中

4，发生碰撞

   4.1 如果是链表，迭代插入到链表尾部

   4.2 如果链表的长度超过8，则立即转为红黑树(当数组长度小于64时，进行扩容而不是树优化)

   4.3 如果是红黑树，则插入到红黑树中

5.如果在以上过程中发现key已经存在，则覆盖旧值

6，如果size > threshold， 进行扩容

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
  // 第一步 懒加载 第一次put的时候初始化table
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
  // 第二步  (n - 1) & hash 计算下标
    if ((p = tab[i = (n - 1) & hash]) == null)
      // 如果为空，则没有发生hash碰撞，直接放在桶中
        tab[i] = newNode(hash, key, value, null);
    else {
      // 存在hash碰撞 
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 如果key值存在，则覆盖掉旧值
        else if (p instanceof TreeNode)
          // 如果这个桶是红黑树，则直接在后面插入数据
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                  // 在链表尾部追加这个值
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                      // 如果链表长度超过了8 则把链表转成红黑树
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                  // 在遍历过程中发现key值相同，则覆盖就只
                    break;
                p = e;
            }
        }
      //执行覆盖旧值的信息
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

### 2.1.2 扩容

  原理： (1) 创建一个2倍于原来(oldTable)容量的数组。(2) 遍历oldTable,如果当前桶里没有元素就直接跳过；如果当前位置只有一个元素，直接移动到newTab中的索引位；如果当前桶是红黑树，在split方法中进行元素的移动；如果当前桶是链表，执行链表的元素移动逻辑。

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 大于最大容量，不进行扩容（桶的数据量固定） 
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 扩容为原来的2倍 
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```



### 2.1.3：HashMap的相关ms知识

####  2.1.3.1 HashMap的底层实现原理是什么？

   基于hasing的原理，jdk8后采用数组+链表+红黑树的数据结构，我们通过put和get存储和获取对象。当我们给put()方法传递键和值时，先对键做一个hashcode的计算来得到它在bucket数组中的位置来存储Entry对象。当获取对象时，通过get获取到bucket的位置，再通过键对象的equals()方法找到正确的键值对，然后再返回值对象。

####   2.1.3.2 jdk1.8里面为什么要引入红黑树？

​    1.8的HashMap的实现是 数组+链表，哈希函数很难达到元素百分百均匀分布。当HashMap中有大量的元素都存放在同一个桶中时，这个桶下有一条很长的链表，造成遍历的时间复杂度就变成O(n)

#### 2.1.3.3 loadFactor为什么 是 0.75，为什么默认容量的大小是16？

​    (1) 默认大小是16，是为了降低hash碰撞的几率。只要是2^n都行 ，

   容量是hashmap中桶的个数，当我们向hashmap中put一个元素的时候，需要通过哈希算法来计算出这个数据应该放在哪个桶中，hash方法的功能是根据key来定位K-V在链表中数组的位置，由于有效率方面的考虑，则hashmap中的hash函数有单独的算法。

# 3: 反射

   反射是程序确定类有哪些方法、有哪些构造方法以及有哪些成员变量。通过反射可以动态的读取一个类的信息，能够在运行时动态加载类。

###  面试的问题 ：

#### **1：什么是java的序列化？**

​    **序列化**：将java对象转换成字节流的过程。  **反序列化**：将字节流转换成java对象的过程。

#### **2： 什么情况下需要用到序列化？**

​    当java对象需要在网络上传输或者持久化存储到文件中时，就需要对java对象进行序列化处理。

 实现的过程：类实现Serializable的接口就行

#### 3： 相关代码

```java
public static void main(String[] args) throws Exception{
    //序列化  将对象持久化到文件中
    ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("C:\\Users\\Administrator\\Desktop\\wdw\\TestSerializable.obj"));
    oos.writeObject("测试序列化");
    oos.writeObject(618);
    DataProcess dataProcess = new DataProcess();
    dataProcess.setIp("1111");
    oos.writeObject(dataProcess);

    // 反序列化
    ObjectInputStream ois = new ObjectInputStream(new FileInputStream("C:\\Users\\Administrator\\Desktop\\wdw\\TestSerializable.obj"));
    System.out.println((String) ois.readObject());
    System.out.println((int) ois.readObject());
    System.out.println(((DataProcess) ois.readObject()).getIp());
}
```

#### **4： 动态代理是什么？**

  **动态代理：**当想要给实现了某个接口的类中的方法，加一些额外的处理，则可以给这个类创建一个新的类，这个类不仅包含原来类的功能，还能在原有的基础上添加额外的处理新类，这个代理类不是定义好的，是动态生成的，具有解耦意义。  

  方法运行时动态构建代理、动态处理代理方法调用的机制。常见的实现方式有JDK Proxy和cglib

##### (1) JDK动态代理类

- ​    实现InvocationHandler接口

- ​    重写invoke方法，添加业务逻辑

- ​    持有目标类的对象

- ​    提供静态方法获取代理

  **优点：**

  ​    1:可以动态地代理实现接口的目标类

  ​    2:不用创建大量的代理类

  ​    3:可以在代理类中对功能进行拦截和扩充

​       **缺点：**

​            目标对象必须实现了接口，否则无法使用JDK代理，会抛出异常    

​    具体的代码实现：

```java
public class JdkDynamicProxy implements InvocationHandler {

    // 获取用户信息的方法
    private Object target;

    public JdkDynamicProxy(Object target){
        this.target = target;
    }


    /**
     * 提供给 JVN动态反射调用目标类的方法，返回结果
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result = null;
        long start = System.currentTimeMillis();
        result = method.invoke(target,args);
        long end = System.currentTimeMillis();
        System.out.println("耗时"+(end - start)+"毫秒");
        return result;
    }

    @SuppressWarnings("unchecked")
    public <T> T getProxy(){
        return (T) Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),this);
    }
}
```

 调用方法

```java
public static void main(String[] args) throws Exception{
    SummaryTableService summaryTableServiceImpl = new SummaryTableServiceImpl();
    JdkDynamicProxy jdkDynamicProxy = new JdkDynamicProxy(summaryTableServiceImpl);
    SummaryTableService service =  jdkDynamicProxy.getProxy();
    service.getResourceStatus();
}
```

##### (2) cglib的动态代理

```java
public class CglibProxy implements MethodInterceptor {

    private CglibProxy(){

    }

    // 单列模式获取代理类对象
    private static CglibProxy proxy = new CglibProxy();
    public static CglibProxy getInstance(){
        return proxy;
    }
    @SuppressWarnings("unchecked")
    public <T> T getProxy(Class<T> tClass){
        return (T) Enhancer.create(tClass,this);
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        Object result = null;
        long start = System.currentTimeMillis();
        result = methodProxy.invokeSuper(o,objects);
        long end = System.currentTimeMillis();
        System.out.println("方法"+method.getName()+"耗时"+(end - start)+"毫秒");
        return result;
    }
}
```

调用方法

```java
SummaryTableService service1 = CglibProxy.getInstance().getProxy(SummaryTableServiceImpl.class);
service1.getResourceStatus();
```

#### 5：有哪些具体的应用？

  **应用：**Spring的AOP，加事务，加权限，加日志。（具体的aop原理和日志切片的原理代码查看相关的文档）



  java的反射机制API主要是java.lang.Class类和java.lang.reflect类。

## 3.1 java.lang.Class类的介绍

​    java.lang.Class类是实现反射的关键所在，Class类的一个实例表示Java的一种数据类型，包括类、接口、枚举、注解（Annotation）、数组、基本数据类型和void，void是“无类型”，主要用于方法返回值类型声明，表示不需要返回值。Class没有公有的构造方法，Class实例是由JVM在类加载时自动创建的。

​    在程序代码中获得Class实例可以通过如下代码实现；

```java
//1.通过类型class静态变量
Class clz1 = String.class;
String str = "Hello";
//2.通过对象的getClass()方法
Class clz2 = str.getClass();
```

相关操作如下：

```java
public class HelloWorld {
public static void main(String[] args) {
  // 获得Class实例
  // 1.通过类型class静态变量
  Class clz1 = String.class;
  String str = "Hello";
  // 2.通过对象的getClass()方法
  Class clz2 = str.getClass();
  //获得int类型Class实例
  Class clz3 = int.class;                         ①
  //获得Integer类型Class实例
  Class clz4 = Integer.class;                    ②
  System.out.println("clz2类名称：" + clz2.getName());
  System.out.println("clz2是否为接口：" + clz2.isInterface()
  System.out.println("clz2是否为数组对象：" + clz2.isArray())
  System.out.println("clz2父类名称：" + clz2.getSuperclass()
  }
}
```



## 3.2 java.lang.reflect类的介绍

  java.lang.reflect 包提供了反射中用到类，主要的类说明如下：

- Constructor类：提供类的构造方法信息。
- Field类：提供类或接口中成员变量信息。
- Method类：提供类或接口成员方法信息。
- Array类：提供了动态创建和访问Java数组的方法。
- Modifier类：提供类和成员访问修饰符信息。

示例代码如下：

```java
Class c = Class.forName("java.lang.String");
Method[] methods = c.getDeclaredMethods();
for(Method method:methods){
    System.out.println(Modifier.toString(method.getModifiers()));
    System.out.println(method.getName());

}
```



### 3.2.1 创建对象

   Class类提供了一个实例方法newInstance()，通过该方法可以创建对象。

```java
Class clz = Class.forName("java.lang.String");
String str = (String) clz.newInstance();
```

#### 3.2.1.1调用构造方法

  调用方法newInstance()创建对象，这个代码只是调用了String的默认构造方法。如果想调用非默认的构造方法。



# 4: 对象拷贝

## 1: 面试常见问题

### **1.1 为什么要使用克隆？**

​    因为 = 只是将一个对象的引用赋值给了另一个对象，当修改第一个对象的值时，另一个对象的数据也会修改。

如果想复制一个对象，并且需要保留之前已经修改后的属性，则需要使用clone方法来克隆。克隆之后的对象与原来的对象是独立存在的。

### **1.2 如何实现对象克隆？**

   克隆的方法主要是 深克隆(DeepClone)和浅克隆(ShallowClone)。在java语言中，数据类型主要包括 基本数据类型和引用类型，基本数据类型包括 int 、double、byte等；引用类型包括类、接口、数组等复杂类型。

   深克隆和浅克隆的区别主要是否支持引用类型的成员变量的复制。

####   1.2.1 浅克隆的代码实现：

-   被复制的类需要实现Cloneable接口，该接口为标记接口，不包含任何方法
-   覆盖clone()方法，访问修饰符设为public。方法中调用super.clone()方法得到需要的复制对象。

```java
public class Student implements Cloneable{
    private int number;

    public int getNumber() {
        return number;
    }

    public void setNumber(int number) {
        this.number = number;
    }

    @Override
    public Object clone() {
        Student stu = null;
        try{
            stu = (Student) super.clone();
        }catch (Exception e){
            e.printStackTrace();
        }
        return stu;

    }

}

 public static void main(String[] args) throws Exception{

        Student student = new Student();
        student.setNumber(11111);

        Student student2 = (Student)student.clone();
       
}
```

####   1.2.2 深克隆的代码实现：

```java
public class Student implements Cloneable{
    private int number;
    private Address address;

    public int getNumber() {
        return number;
    }

    public void setNumber(int number) {
        this.number = number;
    }

    @Override
    public Object clone() {
        Student stu = null;
        try{
            stu = (Student) super.clone();
        }catch (Exception e){
            e.printStackTrace();
        }
        // 如果不加上这个，则只是复制了address变量的引用，并没有将真正的开辟另一块空间
        stu.address = (Address)address.clone();
        return stu;

    }

}

class Address implements Cloneable{
    private String address;

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public Object clone() {
        Address addr = null;
        try{
            addr = (Address) super.clone();
        }catch (Exception e){
            e.printStackTrace();
        }
        return addr;

    }
}

```



### 1.3 解决多层克隆的问题

  如果引用类型里面包含很多引用类型，则会很麻烦，随意可以用序列化和反序列化的方法进行对象的深克隆。

```java
public class Outer implements Serializable {
    private static final long serialVersionUID = 123123123L;
    public Inner inner;

    public Outer myclone(){
        Outer outer = null;
        try{
            // 将流对象序列化成流，因为写在流里面的是对象的一个拷贝
            ByteArrayOutputStream baos = new  ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(baos);
            oos.writeObject(this);

            // 将流序列化成对象
            ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bais);
            outer = (Outer)ois.readObject();
        }catch (Exception e){
            e.printStackTrace();
        }
        return outer;
    }

}

/**
 * 这个类也必须实现 Serializable
 */
class Inner implements Serializable{

}
```



# 5: 异常 的相关信息

## 1:java的throw和throws的区别

  (1): throw用在方法体内，throws用在方法声明后面，表示再抛出异常，由该方法的调用者来处理。

  (2): throw是具体向外抛异常，抛出的是一个异常实例， throws声明了是哪种类型的异常，使它的调用者可以捕获异常



## 2: final、finally和finalize的区别

### 2.1: final关键字

  2.1.1 修饰类

   使用final修饰类，说明这个类不能被继承，由于final修饰的类不能派生出子类，所以不可以同时使用final和abstract修饰类。

   使用final修饰方法，则此方法不能被子类重写。

  2.1.2 修饰成员变量

  修饰成员变量必须要赋初值；修饰非静态成员变量时，可以在生命时赋值，也可以在初始化块中赋值，也可以在构造函数中赋值。修饰静态变量时，只能在声明和静态初始化块中赋值，不可以在构造方法中赋值。

修饰局部变量时，表示这个值只能读取，不可以被修改



### 2.2: finalize关键字

  它是一个方法，执行GC清理它所从属的对象时被调用



## 3：常见的异常类有哪些





# 6: 相关java的基础知识

## 6.1: JDK和JRE的区别？

  jre:是java运行时环境，包含了java虚拟机，java基础类库。是使用java语言编写的程序运行时



## 6.2: extends和implements的区别？

   extends 是继承某一个类，继承之后可以使用父类的方法，也可以重写父类的方法(继承类)；

  implements是实现多个接口，接口的方法一般为空，必须重写才能使用(实现接口)

