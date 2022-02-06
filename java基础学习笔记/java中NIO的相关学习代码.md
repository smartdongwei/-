# java中NIO的相关学习代码

​     IO是面向流的处理，NIO是面向块（缓冲区）的处理。NIO主要有三大核心：Channel(通道)，Buffer(缓冲区), Selector(选择区) 。

1. 面向流的I/O系统是一次一个字节地处理数据，直至读取所有字节，它们没有被缓存在任何地方;  IO的各种流是阻塞的；基于字节流和字符流进行操作。
2. NIO: 数据读取到一个它稍后处理的缓冲区，需要时可以在缓冲区中前后移动，需确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据。NIO的非阻塞模式，当一个线程从某通道发送请求读取数据，但是仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取； 基于Channel和Buffer进行操作，数据总是从通道读取到缓冲区，或者从缓冲区写入到通道中； Selector用于监听多个通道的事件

**缺点：**

​     在数据处理之前必须判断缓冲区的数据是否完整或已经读取完毕，所以每次数据处理之前都要检测缓冲区的数据。

**适用场景：**

  如果需要管理成千上万个连接，并且每个连接只是发送少量数据，则NIO处理数据。

  如果只有少量的连接，而这些连接每次都要发送大量数据，则传统的IO比较好。



## 1: 相关基础知识

### 1.1  Channel的相关内容

  一个channel代表和某一个实体的连接，这个实体可以是文件、网络套接字等，是用于我们的程序和操作系统底层I/O服务进行交互。channel是双向的，既可以用于读操作，也可以进行写操作。主要写读取写入到buffer中。主要的实现有：

- FileChannel  (文件IO)
- DatagramChannel   (UDP)
- SocketChannel  
- ServerSocketChannel



### 1.2  Buffer的相关内容

  NIO使用的缓冲区不是一个简单的byte数组，而是封装后的Buffer类，通过提供的API，关键的实现有：ByteBuffer，CharBuffer , DoubleBuffer, FloatBuffer等内容

```java
//创建一个容量为48的
ByteBuffer buffer = ByteBuffer.allocate(48);
// 从通道中获取相关数据并写入到buffer中
int byteRead = inChannel.read(buffer);
while(byteRead != -1) {
    buffer.flip(); // 转换buffer为读模式
    System.out.print((char) buffer.get()); // 一次读取一个byte
    buffer.clear(); //清空buffer准备下⼀次写⼊
}
```

### 1.3  Selector的相关内容

  可以单线程处理多个Channel，如果你的应用打开了多个通道，但每个连接的流量都很低，则需要得向Selector注册那个Channel，然后再调用它的select()方法，这个方法会一直阻塞到某个注册的通道有事件就绪，一旦这个方法有返回，线程就会处理这些事件。



## 2: FileChannel的相关代码

```java
RandomAccessFile aFile= null;
try{
    aFile = new RandomAccessFile("C:\\Users\\Administrator\\Desktop\\data1.txt","rw");
    FileChannel fileChannel = aFile.getChannel();
    // 分配空间
    ByteBuffer buf = ByteBuffer.allocate(1024);
    int bytesRead = fileChannel.read(buf);
    while(bytesRead != -1){
      // 转换buffer为读模式
        buf.flip();
        while(buf.hasRemaining()){
          // 一次读取一个byte
            System.out.println((char) buf.get());
        }
      //清空buffer准备下次写做准备k  把未读取的数据拷贝到buffer的起始位置
        buf.compact();
        bytesRead = fileChannel.read(buf);
    }
}catch (Exception e){
    e.printStackTrace();
}finally {
    if(aFile != null){
        aFile.close();
    }
}
```

  由上面代码可知，可以把buffer理解为一组基本的数据类型的元素列表，它通过几个变量来保存这个数据的当前位置状态：capacity, position, limit, mark 。

| 索引       | 说明                         |
| -------- | -------------------------- |
| capacity | 缓冲区数组的总长度                  |
| position | 下一个要操作的数据元素的位置             |
| limit    | 缓冲区数组中不可操作的下一个元素的位置        |
| mark     | 用于记录当前position的前一个位置或默认是-1 |



## 3: 内存映射文件

​      JAVA处理大文件，一般用BufferedReader,BufferedInputStream这类带缓冲的IO类，不过如果文件超大的话，更快的方式是采用MappedByteBuffer。 MappedByteBuffer是NIO引入的文件内存映射方案，读写性能极高。NIO最主要的就是实现了对异步操作的支持。其中一种通过把一个套接字通道(SocketChannel)注册到一个选择器(Selector)中,不时调用后者的选择(select)方法就能返回满足的选择键(SelectionKey),键中包含了SOCKET事件信息。这就是select模型。

​    SocketChannel的读写是通过一个类叫ByteBuffer来操作的.这个类本身的设计是不错的,比直接操作byte[]方便多了. ByteBuffer有两种模式:直接/间接.间接模式最典型(也只有这么一种)的就是HeapByteBuffer,即操作堆内存 (byte[]).但是内存毕竟有限,如果我要发送一个1G的文件怎么办?不可能真的去分配1G的内存.这时就必须使用”直接”模式,即 MappedByteBuffer,文件映射。

​    操作系统的内存管理.一般操作系统的内存分两部分:物理内存;虚拟内存.虚拟内存一般使用的是页面映像文件,即硬盘中的某个(某些)特殊的文件.操作系统负责页面文件内容的读写,这个过程叫”页面中断/切换”. MappedByteBuffer也是类似的,你可以把整个文件(不管文件有多大)看成是一个ByteBuffer.MappedByteBuffer 只是一种特殊的ByteBuffer，即是ByteBuffer的子类。MappedByteBuffer 将文件直接映射到内存（这里的内存指的是虚拟内存，并不是物理内存）。通常，可以映射整个文件，如果文件比较大的话可以分段进行映射，只要指定文件的那个部分就可以。

### 3.1 概念

​    FileChannel提供了map方法来把文件影射为内存映像文件：MappedByteBuffer map(int mode,long position,long size); 可以把文件的从position开始的size大小的区域映射为内存映像文件，mode指出了 可访问该**内存映像文件的方式：**

- READ_ONLY,（只读）：试图修改得到的缓冲区将导致抛出 ReadOnlyBufferException.(MapMode.READ_ONLY)
- READ_WRITE（读/写）：对得到的缓冲区的更改最终将传播到文件；该更改对映射到同一文件的其他程序不一定是可见的。(MapMode.READ_WRITE)
- PRIVATE（专用）：对得到的缓冲区的更改不会传播到文件，并且该更改对映射到同一文件的其他程序也不是可见的；相反，会创建缓冲区已修改部分的专用副本。(MapMode.PRIVATE)

**MappedByteBuffer是ByteBuffer的子类，其扩充了三个方法：**

- force()：缓冲区是READ_WRITE模式下，此方法对缓冲区内容的修改强行写入文件；
- load()：将缓冲区的内容载入内存，并返回该缓冲区的引用；
- isLoaded()：如果缓冲区的内容在物理内存中，则返回真，否则返回假

### 3.2 相关代码

```java
RandomAccessFile aFile = null;
FileChannel fc = null;
try{
    long timeBegin = System.currentTimeMillis();
    aFile = new RandomAccessFile("D:\\BissData\\ceshi\\nohup.out","rw");
    fc = aFile.getChannel();
    MappedByteBuffer mbb = fc.map(FileChannel.MapMode.READ_ONLY, 0, aFile.length());
    // 按行读取
    StringBuilder line = new StringBuilder();
    for(int i = 0; i<mbb.limit(); i++){
        char a = (char) mbb.get();
        if( a =='\n'){
            System.out.println(line);
            line = new StringBuilder();
        }else{
            line.append(a);
        }
    }
    long timeEnd = System.currentTimeMillis();
    System.out.println("Read time: "+(timeEnd-timeBegin)+"ms");
    long timeBegin1 = System.currentTimeMillis();
    List<String> lll = FileUtils.readLines(new File("D:\\BissData\\ceshi\\nohup.out"));
    for(String aa:lll){
        System.out.println(aa);
    }
    long timeEnd1 = System.currentTimeMillis();
    System.out.println("Read1 time: "+(timeEnd1-timeBegin1)+"ms");
}catch(IOException e){
    e.printStackTrace();
}finally{
    try{
        if(aFile!=null){
            aFile.close();
        }
        if(fc!=null){
            fc.close();
        }
    }catch(IOException e){
        e.printStackTrace();
    }
}
```

























