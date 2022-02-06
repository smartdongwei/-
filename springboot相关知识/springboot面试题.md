## 1：SpringBoot-启动流程

平时开发springboot项目的时候，一个SpringBootApplication注解加一个main方法就可以启动服务器运行起来（默认tomcat），看了下源码，这里讲下认为主要的流程

**主要流程如下**

0.启动main方法开始

1.**初始化配置**：通过类加载器，（loadFactories）读取classpath下所有的spring.factories配置文件，创建一些初始配置对象；通知监听者应用程序启动开始，创建环境对象environment，用于读取环境配置 如 application.yml

2.**创建应用程序上下文**-createApplicationContext，创建 bean工厂对象

3.**刷新上下文（启动核心）**
3.1 配置工厂对象，包括上下文类加载器，对象发布处理器，beanFactoryPostProcessor
3.2 注册并实例化bean工厂发布处理器，并且调用这些处理器，对包扫描解析(主要是class文件)
3.3 注册并实例化bean发布处理器 beanPostProcessor
3.4 初始化一些与上下文有特别关系的bean对象（创建tomcat服务器）
3.5 实例化所有bean工厂缓存的bean对象（剩下的）
3.6 发布通知-通知上下文刷新完成（启动tomcat服务器）

4.**通知监听者-启动程序完成**