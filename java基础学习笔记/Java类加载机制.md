---
title: java类加载机制
tags: [java基础]
date: 2020-02-10 10:00:00
---



## jvm的类加载机制

  JVM将class文件字节码文件加载到内存中， 并将这些静态数据转换成方法区中的运行时数据结构，在堆(并不一定在堆中，HotSpot在方法区中)中生成一个代表这个类的java.lang.Class 对象，作为方法区类数据的访问入口。

####类加载的七个阶段

![image-20201209215132550](C:\Users\wang\AppData\Roaming\Typora\typora-user-images\image-20201209215132550.png)

#### 四个概念：

  1:class文件：编译之后生成的文件，存储在硬盘上

  2:class content： 加载阶段，将class文件加载如内存

  3:class对象： Class  claszz = Test.class   存储在 堆区 

  4:对象： new出来的对象信息

class文件在内存中是如何存储的

方法区是理论(规范)/接口， 永久代、元空间是具体实现



####永久代元空间相关理论基础

在jdk7以前是永久代（放在堆区管理）  在jdk8以后是元空间（直接内存管理）

为什么？ gc问题，堆区里面只能有对象   应用问题class可以动态生成



#### 元空间的调优

  原因 ：1：用完不释放   2：gc回收的速度赶不上你使用的速度

  最小的值 20.75    最大的值 2的48次方

   java  -XX:+PrintFlagsFinal -version |grep Metaspace

  规范 ： 1、最小最大设置成一样  2、大小= 物理内存的1/32  3、预留空间 20%-30%

arthas    VisualVM 进行调优 



####虚拟机栈

  栈的一种应用，jvm中一个线程一个虚拟机栈，一个虚拟机栈 方法调用一次生成一个栈帧。

1、动态链接：方法对象的内存地址（方法区）

2、

![image-20201209220153911](C:\Users\wang\AppData\Roaming\Typora\typora-user-images\image-20201209220153911.png)

方法中存储的主要是这样：

![image-20201209220318686](C:\Users\wang\AppData\Roaming\Typora\typora-user-images\image-20201209220318686.png)