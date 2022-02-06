# Java序列化的相关知识

## 1、序列化是干啥用的？

​    序列化的原本意图是希望对一个Java对象作一下“变换”，变成字节序列，这样一来方便持久化存储到磁盘，避免程序运行结束后对象就从内存里消失，另外变换成字节序列也更便于网络运输和传播，所以概念上很好理解：

- **序列化**：把Java对象转换为字节序列。
- **反序列化**：把字节序列恢复为原先的Java对象。

​    而且序列化机制从某种意义上来说也弥补了平台化的一些差异，毕竟转换后的字节流可以在其他平台上进行反序列化来恢复对象。

## 2、对象序列化的方式？

**在Java中，如果一个对象要想实现序列化，必须要实现下面两个接口之一：**

- Serializable 接口
- Externalizable 接口

## **2.1 Serializable 接口**

​    一个对象想要被序列化，那么它的类就要实现此接口或者它的子接口。这个对象的所有属性（包括private属性、包括其引用的对象）都可以被序列化和反序列化来保存、传递。不想序列化的字段可以使用transient修饰。由于Serializable对象完全以它存储的二进制位为基础来构造，因此并不会调用任何构造函数，因此Serializable类无需默认构造函数，但是当Serializable类的父类没有实现Serializable接口时，反序列化过程会调用父类的默认构造函数，因此该父类必需有默认构造函数，否则会抛异常。使用transient关键字阻止序列化虽然简单方便，但被它修饰的属性被完全隔离在序列化机制之外，导致了在反序列化时无法获取该属性的值，而通过在需要序列化的对象的Java类里加入writeObject()方法与readObject()方法可以控制如何序列化各属性，甚至完全不序列化某些属性或者加密序列化某些属性。

## **2.2 Externalizable 接口**

​    它是Serializable接口的子类，用户要实现的writeExternal()和readExternal() 方法，用来决定如何序列化和反序列化。因为序列化和反序列化方法需要自己实现，因此可以指定序列化哪些属性，而transient在这里无效。对Externalizable对象反序列化时，会先调用类的无参构造方法，这是有别于默认反序列方式的。如果把类的不带参数的构造方法删除，或者把该构造方法的访问权限设置为private、默认或protected级别，会抛出java.io.InvalidException: no valid constructor异常，因此Externalizable对象必须有默认构造函数，而且必需是public的。

## **2.3 对比**

​    使用时，你只想隐藏一个属性，比如用户对象user的密码pwd，如果使用Externalizable，并除了pwd之外的每个属性都写在writeExternal()方法里，这样显得麻烦，可以使用Serializable接口，并在要隐藏的属性pwd前面加上transient就可以实现了。如果要定义很多的特殊处理，就可以使用Externalizable。当然这里我们有一些疑惑，Serializable 中的writeObject()方法与readObject()方法科可以实现自定义序列化，而Externalizable 中的writeExternal()和readExternal() 方法也可以，他们有什么异同呢？

- readExternal(),writeExternal()两个方法，这两个方法除了方法签名和readObject(),writeObject()两个方法的方法签名不同之外，其方法体完全一样。
- 需要指出的是，当使用Externalizable机制反序列化该对象时，程序会使用public的无参构造器创建实例，然后才执行readExternal()方法进行反序列化，因此实现Externalizable的序列化类必须提供public的无参构造。
- 虽然实现Externalizable接口能带来一定的性能提升，但由于实现ExternaLizable接口导致了编程复杂度的增加，所以大部分时候都是采用实现Serializable接口方式来实现序列化。



## 3、Serializable 如何序列化对象？

## **3.1 Serializable演示**

​    然而Java目前并没有一个关键字可以直接去定义一个所谓的“可持久化”对象。对象的持久化和反持久化需要靠程序员在代码里手动**显式地**进行序列化和反序列化还原的动作。

​    举个例子，假如我们要对Student类对象序列化到一个名为student.txt的文本文件中，然后再通过文本文件反序列化成Student类对象：

1、Student类定义

```text
public class Student implements Serializable {

    private String name;
    private Integer age;
    private Integer score;
 
    @Override
    public String toString() {
        return "Student:" + '\n' +
        "name = " + this.name + '\n' +
        "age = " + this.age + '\n' +
        "score = " + this.score + '\n'
        ;
    }
 
    // ... 其他省略 ...
}
```

2、序列化

```text
public static void serialize(  ) throws IOException {

    Student student = new Student();
    student.setName("CodeSheep");
    student.setAge( 18 );
    student.setScore( 1000 );

    ObjectOutputStream objectOutputStream = 
        new ObjectOutputStream( new FileOutputStream( new File("student.txt") ) );
    objectOutputStream.writeObject( student );
    objectOutputStream.close();
 
    System.out.println("序列化成功！已经生成student.txt文件");
    System.out.println("==============================================");
}
```

3、反序列化

```text
public static void deserialize(  ) throws IOException, ClassNotFoundException {
    ObjectInputStream objectInputStream = 
        new ObjectInputStream( new FileInputStream( new File("student.txt") ) );
    Student student = (Student) objectInputStream.readObject();
    objectInputStream.close();
 
    System.out.println("反序列化结果为：");
    System.out.println( student );
}
```

## 两种特殊情况

1、凡是被static修饰的字段是不会被序列化的

2、凡是被transient修饰符修饰的字段也是不会被序列化的



## 4、实现Externalizable

```text
public UserInfo() {
    userAge=20;//这个是在第二次测试使用，判断反序列化是否通过构造器
}
public void writeExternal(ObjectOutput out) throws IOException  {
    //  指定序列化时候写入的属性。这里仍然不写入年龄
    out.writeObject(userName);
    out.writeObject(usePass);
}
public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException  {
    // 指定反序列化的时候读取属性的顺序以及读取的属性
    // 如果你写反了属性读取的顺序，你可以发现反序列化的读取的对象的指定的属性值也会与你写的读取方式一一对应。因为在文件中装载对象是有序的
    userName=(String) in.readObject();
    usePass=(String) in.readObject();
}
```

我们在序列化对象的时候，由于这个类实现了Externalizable 接口，在writeExternal()方法里定义了哪些属性可以序列化，哪些不可以序列化，所以，对象在经过这里就把规定能被序列化的序列化保存文件，不能序列化的不处理，然后在反序列的时候自动调用readExternal()方法，根据序列顺序挨个读取进行反序列，并自动封装成对象返回，然后在测试类接收，就完成了反序列。

Externalizable 实例类的唯一特性是可以被写入序列化流中，该类负责保存和恢复实例内容。 若某个要完全控制某一对象及其超类型的流格式和内容，则它要实现 Externalizable 接口的 writeExternal 和 readExternal 方法。这些方法必须显式与超类型进行协调以保存其状态。这些方法将代替定制的 writeObject 和 readObject 方法实现。

- **writeExternal(ObjectOutput out)**
  该对象可实现 writeExternal 方法来保存其内容，它可以通过调用 DataOutput 的方法来保存其基本值，或调用 ObjectOutput 的 writeObject 方法来保存对象、字符串和数组。
- **readExternal(ObjectInput in)**
  对象实现 readExternal 方法来恢复其内容，它通过调用 DataInput 的方法来恢复其基础类型，调用 readObject 来恢复对象、字符串和数组。

**externalizable和Serializable的区别：**

1、实现serializable接口是默认序列化所有属性，如果有不需要序列化的属性使用transient修饰。externalizable接口是serializable的子类，实现这个接口需要重写writeExternal和readExternal方法，指定对象序列化的属性和从序列化文件中读取对象属性的行为。

2、实现serializable接口的对象序列化文件进行反序列化不走构造方法，载入的是该类对象的一个持久化状态，再将这个状态赋值给该类的另一个变量。实现externalizable接口的对象序列化文件进行反序列化先走构造方法得到控对象，然后调用readExternal方法读取序列化文件中的内容给对应的属性赋值。