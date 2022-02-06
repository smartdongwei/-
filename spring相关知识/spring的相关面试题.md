#    Spring面试题总结

## 1：什么是Spring框架

   spring是一个IOC和AOP的容器框架  

  Spring 框架指的都是 Spring Framework，它是很多模块的集合，使⽤这些模块可以很⽅便地协助我们进⾏开发。这些模块是：核⼼容器、数据访问/集成,、Web、AOP（⾯向切⾯ 编程）、⼯具、消息和测试模块。⽐如：Core Container 中的 Core 组件是Spring 所有组件的核 ⼼，Beans 组件和 Context 组件是实现IOC和依赖注⼊的基础，AOP组件⽤来实现⾯向切⾯编程。

  Spring 官⽹列出的 Spring 的 6 个特征: 

​     核⼼技术 ：依赖注⼊(DI)，AOP，事件(events)，资源，i18n，验证，数据绑定，类型转 换，SpEL。

​     测试 ：模拟对象，TestContext框架，Spring MVC 测试，WebTestClient。 

​     数据访问 ：事务，DAO⽀持，JDBC，ORM，编组XML。 

​    Web⽀持 : Spring MVC和Spring WebFlux Web框架。 

​     集成 ：远程处理，JMS，JCA，JMX，电⼦邮件，任务，调度，缓存。 语⾔ ：Kotlin，Groovy，动态语⾔。

## 2：@RestController vs @Controller

​    Controller 返回一个页面，单独使用@Controller 不加 @ResponseBody 的话⼀般使⽤在要返回⼀个视图的情况，这种情况 属于比较传统的Spring MVC 的应⽤，对应于前后端不分离的情况。

​    @RestController 返回JSON 或 XML 形式数据 但 @RestController 只返回对象，对象数据直接以 JSON 或 XML 形式写⼊ HTTP 响应 (Response)中，这种情况属于 RESTful Web服务，这也是⽬前⽇常开发所接触的最常⽤的情况 （前后端分离）。

​    @Controller +@ResponseBody 返回JSON 或 XML 形式数据 如果你需要在Spring4之前开发 RESTful Web服务的话，你需要使⽤ @Controller 并结 合 @ResponseBody 注解，也就是说 @Controller + @ResponseBody = @RestController （Spring 4 之后新加的注解）。

## 3：Spring IOC & AOP

### 3.1 IoC (**控制反转**)

   **答案：**主要从容器概念、控制反转、依赖注入来解释：

​    **IOC容器：**实际上就是map，里面存储的是各种对象(在xml里面配置的bean节点，@repository，@service等)，在项目启动的时候通过反射创建对象放到map中。

​    **控制反转：**在没有引入ioc容器之前，对象A依赖于对象B，那么对象A在初始化或者运行到某一点时，自己必须主动去创建对象B或者使用对象B，控制权在自己手上。引入IOC容器之后，对象A和对象B失去了直接联系，当对象A运行到需要对象B的时候，IOC容器会主动创建一个对象B注入到对象A需要的地方。通过对比，对象A获得依赖对象B的过程，由主动行为变为了被动行为，控制权颠倒，这就是控制反转的由来。

   **依赖注入：**控制被反转之后，获取依赖对象的过程由自身变成由IOC容器主动注入。依赖注入时实现IOC的方法，就是由ioc容器在运行期间，动态的将某种依赖关系注入到对象中。

   IoC（Inverse of Control:控制反转）是⼀种设计思想，就是 将原本在程序中⼿动创建对象的控制 权，交由Spring框架来管理。 IoC 在其他语⾔中也有应⽤，并⾮ Spring 特有。 IoC 容器是 Spring ⽤来实现 IoC 的载体， IoC 容器实际上就是个Map（key，value）,Map 中存放的是各 种对象。

​    将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注⼊。这样可以很⼤ 程度上简化应⽤的开发，把应⽤从复杂的依赖关系中解放出来。 IoC 容器就像是⼀个⼯⼚⼀ 样，当我们需要创建⼀个对象的时候，只需要配置好配置⽂件/注解即可，完全不⽤考虑对象是如 何被创建出来的。 在实际项⽬中⼀个 Service 类可能有⼏百甚⾄上千个类作为它的底层，假如我 们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可 能会把⼈逼疯。如果利⽤ IoC 的话，你只需要配置好，然后在需要的地⽅引⽤就⾏了，这⼤⼤增 加了项⽬的可维护性且降低了开发难度。

IoC 最常见以及最合理的实现方式叫做依赖注入（Dependency Injection，简称 DI）。

   **依赖注入和控制反转**：所谓的依赖注入，则是，甲方开放接口，在它需要的时候，能够讲乙方传递进来(注入)
所谓的控制反转，甲乙双方不相互依赖，交易活动的进行不依赖于甲乙任何一方，整个活动的进行由第三方负责管理。

![image-20210919001009792](E:\study\images\image-20210919001009792.png)

### 3.2 AOP(**面向切面编程**)

   **答案：**将程序中的交叉业务员逻辑(比如安全、日志、事务等)，封装成一个切面，然后注入到目标对象中，AOP可以对某个对象或者某些对象的功能进行增强，比如可以在执行某个操作之前和之后执行一些额外的事情。

​    AOP(Aspect-Oriented Programming:⾯向切⾯编程)能够将那些与业务⽆关，却为业务模块所共同调⽤的逻辑或责任（例如事务处理、⽇志管理、权限控制等）封装起来，便于减少系统的重复 代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。 Spring AOP就是基于动态代理的，如果要代理的对象，实现了某个接⼝，那么Spring AOP会使 ⽤JDK Proxy，去创建代理对象，⽽对于没有实现接⼝的对象，就⽆法使⽤ JDK Proxy 去进⾏代 理了，这时候Spring AOP会使⽤Cglib ，这时候Spring AOP会使⽤ Cglib ⽣成⼀个被代理对象的 ⼦类来作为代理，如下图所示.

  是通过预编译方式和运行期动态代理的方式实现不修改源代码的情况下给程序动态统一添加功能的技术。

####   3.2.1 AOP术语：

​    **1、切面（aspect/advisor）** 

​      类是对物体特征的抽象，切面就是对横切关注点的抽象。组合了 Pointcut 与 Advice，在 Spring 中 有时候也称为 Advisor。

 **2、连接点（join point）** 

​     被拦截到的点，因为 Spring 只支持方法类型的连接点，所以在 Spring 中连接点指的就是被拦截到的 方法，实际上连接点还可以是字段或者构造器。 

**3、切入点（pointcut）**

​     描述的一组符合某个条件的 join point，通常使用 pointcut 表达式来限定 join point。

 **4、通知（advice）** 

​    所谓通知指的就是指拦截到连接点之后要执行的代码，通知分为前置、后置、异常、返回、环绕通知 五类。 **5、目标对象** 

​    代理的目标对象 

**6、织入（weave）** 

​    将 Advice 织入 join point 的这个过程

 **7、引介（introduction）**

​     在不修改代码的前提下，引介可以在运行期为类动态地添加一些方法或字段

#### 3.2.2  Spring AOP 原理以及各种 Advice 和 Advisor 说下

​    通知 Advice 是 Spring 提供的一种切面(Aspect)。但其功能过于简单，只能将切面织入到目标类的 所有目标方法中，无法完成将切面织入到指定目标方法中。

​    顾问 Advisor 是 Spring 提供的另一种切面。其可以完成更为复杂的切面织入功能。PointcutAdvisor 是顾问的一种，可以指定具体的切入点。顾问对通知进行了包装，会根据不同的通知类型，在不同的 时间点，将切面织入到不同的切入点。

​    顾问 Advisor 是 Spring 提供的另一种切面。其可以完成更为复杂的切面织入功能。PointcutAdvisor 是顾问的一种，可以指定具体的切入点。顾问对通知进行了包装，会根据不同的通知类型，在不同的 时间点，将切面织入到不同的切入点。

   **增强（advice)主要包括如下五种类型** 

1. 前置增强(BeforeAdvice)：在目标方法执行前实施增强 
2. 后置增强(AfterAdvice)：在目标方法执行后（无论是否抛出遗产）实施增强 
3. 环绕增强(MethodInterceptor)：在目标方法执行前后实施增强
4. 异常抛出增强(ThrowsAdvice)：在目标方法抛出异常后实施增强 
5. 返回增强（AfterReturningAdvice）：在目标方法正常返回后实施增强
6. 引介增强(IntroductionIntercrptor)：在目标类中添加一些新的方法和属性



#### 3.2.3 AOP 的两种代理方式是什么？

   主要为JDK 动态代理与 CGLIB代理。

   JDK动态代理是基于接口的方式，换句话来说就是代理类和目标类都实现同一个接口，那么代理类和目标类的方法名就一样了，这种方式上一篇说过了；CGLib动态代理是代理类去继承目标类，然后重写其中目标类的方法啊，这样也可以保证代理类拥有目标类的同名方法；

##### 3.2.3.1 JDK 动态代理

​    其代理对象必须是某个接口的实现，它是通过在运行时创建一个接口的实现类来完成对目标对象的代理。

##### 3.2.3.2 CGLIB代理

​    在运行时生成的代理对象是针对目标类扩展的子类。



#### 3.2.4 AOP 一般作用说下

日志记录，性能统计，安全控制，事务处理，异常处理等等wn及扩展



## 4: Spring bean

### 4.1 Spring 中的 bean 的作⽤域有哪些?

- singleton : 唯⼀ bean 实例，Spring 中的 bean 默认都是单例的。 
- prototype : 每次请求都会创建⼀个新的 bean 实例。
- request : 每⼀次HTTP请求都会产⽣⼀个新的bean，该bean仅在当前HTTP request内有效。 
- session : 每⼀次HTTP请求都会产⽣⼀个新的 bean，该bean仅在当前 HTTP session内有效。 
- global-session： 全局session作⽤域，仅仅在基于portlet的web应⽤中才有意义，Spring5已 经没有了。Portlet是能够⽣成语义代码(例如：HTML)⽚段的⼩型Java Web插件。它们基于 portlet容器，可以像servlet⼀样处理HTTP请求。但是，与 servlet 不同，每个 portlet 都有不 同的会话

### 4.2 Spring 中的单例 bean 的线程安全问题了解吗？ 

​    大部分时候我们并没有在系统中使⽤多线程，所以很少有⼈会关注这个问题。单例 bean 存在线 程问题，主要是因为当多个线程操作同⼀个对象的时候，对这个对象的⾮静态成员变量的写操作 会存在线程安全问题。 常见的有两种解决办法： 

1. 在Bean对象中尽量避免定义可变的成员变量（不太现实）。 
2.  在类中定义⼀个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的⼀种⽅式）

### 4.3 @Component 和 @Bean 的区别是什么？

1. 作⽤对象不同: @Component 注解作⽤于类，⽽ @Bean 注解作⽤于⽅法。
2.  @Component 通常是通过类路径扫描来⾃动侦测以及⾃动装配到Spring容器中（我们可以使 ⽤ @ComponentScan 注解定义要扫描的路径从中找出标识了需要装配的类⾃动装配到 Spring 的 bean 容器中）。 @Bean 注解通常是我们在标有该注解的⽅法中定义产⽣这个 bean, @Bean 告诉了Spring这是某个类的示例，当我需要⽤它的时候还给我。
3.  @Bean 注解⽐ Component 注解的⾃定义性更强，⽽且很多地⽅我们只能通过 @Bean 注 解来注册bean

@Bean 注解使⽤示例

```java
@Configuration
public class AppConfig {
   @Bean
   public TransferService transferService() {
     return new TransferServiceImpl();
   }
}
```

上面的代码相当于下⾯的 xml 配置

```xml
<beans>
   <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans
```

### 4.4 将⼀个类声明为Spring的 bean 的注解有哪些

​    我们⼀般使⽤ @Autowired 注解⾃动装配 bean，要想把类标识成可⽤于 @Autowired 注解⾃动 装配的 bean 的类,采⽤以下注解可实现： 

- @Component ：通⽤的注解，可标注任意类为 Spring 组件。如果⼀个Bean不知道属于哪 个层，可以使⽤ @Component 注解标注。
- @Repository : 对应持久层即 Dao 层，主要⽤于数据库相关操作。 
- @Service : 对应服务层，主要涉及⼀些复杂的逻辑，需要⽤到 Dao层。 
- @Controller : 对应 Spring MVC 控制层，主要⽤户接受⽤户请求并调⽤ Service 层返回数 据给前端⻚⾯。

### 4.5 Spring 中的 bean ⽣命周期

  **主要得生命周期为：**

​    Bean的定义——Bean的初始化——Bean的使用——Bean的销毁

#### 4.5.1 Bean的定义

Bean 是 spring 装配的组件模型，一切实体类都可以配置成一个 Bean ，进而就可以在任何其他的 Bean 中使用，一个 Bean 也可以不是指定的实体类，这就是抽象 Bean 。

#### 4.5.2 Bean的初始化

Spring中bean的初始化回调有两种方法

一种是在配置文件中声明init-method=“init”，然后在一个实体类中用init()方法来初始化

另一种是实现InitializingBean接口，覆盖afterPropertiesSet()方法。

#### 4.5.3 Bean的使用

**1.  BeanFactory：**

​    BeanFactory是延迟加载,如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用getBean方法才会抛出异常，也就是说当使用BeanFactory实例化对象时，配置的bean不会马上被实例化。当你使用该bean时才会被实例化（getBean）

```java
BeanFactory factory = new XmlBeanFactory(new ClassPathResource("bean.xml"));
```

**2.ApplicationContext：**

​    如果使用ApplicationContext，则配置的bean如果是singleton不管你用还是不用，都被实例化。ApplicationContext在初始化自身时检验，这样有利于检查所依赖属性是否注入。ApplicationContext是BeanFactory的子类，除了具有BeanFactory的所有功能外还提供了更完整的框架功能，例如国际化，资源访问等。所以通常情况下我们选择使用ApplicationContext。

```java
ApplicationContext ac = new ClassPathXmlApplicationContext("bean.xml");
1
```

#### 4.5.4 Bean的销毁

Bean的销毁和初始化一样，都是提供了两个方法

一是在配置文件中声明destroy-method=“cleanup”，然后在类中写一个cleanup()方法销毁

二是实现DisposableBean接口，覆盖destory()方法



## 5:Spring MVC的相关概念

​    MVC 是⼀种设计模式,Spring MVC 是⼀款很优秀的 MVC 框架。Spring MVC 可以帮助我们进⾏ 更简洁的Web层的开发，并且它天⽣与 Spring 框架集成。Spring MVC 下我们⼀般把后端项⽬分 为 Service层（处理业务）、Dao层（数据库操作）、Entity层（实体类）、Controller层(控制 层，返回数据给前台⻚⾯)

  Spring MVC 的简单原理图如下：

![image-20210919010931729](E:\study\images\image-20210919010931729.png)

### 5.1 ⼯作原理

1. 客户端（浏览器）发送请求，直接请求到 前端控制器(DispatcherServlet) 。
2.  DispatcherServlet 根据请求信息调⽤处理器映射器(HandlerMapping) ，解析请求对应的处理器(Handler)。
3.  解析到对应的 Handler （也就是我们平常说的 Controller 控制器）后，开始由(处理器适配器)HandlerAdapter 适配器处理。 
4.  HandlerAdapter 会根据 Handler 来调⽤真正的处理器开处理请求，并处理相应的业务逻 辑。 
5.  处理器处理完业务后，会返回⼀个 ModelAndView 对象， Model 是返回的数据对 象， View 是个逻辑上的 View 。
6.  视图解析器(ViewResolver) 会根据逻辑 View 查找实际的 View 。
7.  DispaterServlet 把返回的 Model 传给 View （视图渲染）。
8.  把 View 返回给请求者（浏览器）



## 6:spring事务

### 6.1 Spring 管理事务的⽅式有⼏种？

1. 编程式事务，在代码中硬编码。(不推荐使⽤) 
2. 声明式事务，在配置⽂件中配置（推荐使⽤)

## 7：BeanFactory和ApplicationContext有什么区别

BeanFactory 是 Spring 的“心脏”。它就是 Spring IoC 容器的真面目。Spring 使用 BeanFactory 来实例化、配置和管理 Bean。

BeanFactory是IOC容器的核心接口， 它定义了IOC的基本功能，我们看到它主要定义了getBean方法。getBean方法是IOC容器获取bean对象和引发依赖注入的起点。方法的功能是返回特定的名称的Bean。

BeanFactory 是初始化 Bean 和调用它们生命周期方法的“吃苦耐劳者”。注意，BeanFactory 只能管理单例（Singleton）Bean 的生命周期。它不能管理原型(prototype,非单例)Bean 的生命周期。这是因为原型 Bean 实例被创建之后便被传给了客户端,容器失去了对它们的引用。

### 7.1 相关区别

ApplicationContext是BeanFactory的子接口，它提供了更完整的功能：

1. 继承MessageSource，因此支持国际化。
2. 统一的资源文件访问方式。
3. 提供在监听器中注册bean的事件。
4. 同时加载多个配置文件。
5. 载入多个(有继承关系)上下文，使得每一个上下文都专注于一个特定的层次，比如应用的web层。

**BeanFactory相关特性：**

- 采用的是延迟加载形式注入bean的，即只有在使用某个bean的时候才对该bean进行加载实例化。这样我们就不能发现一些存在的spring配置问题。如果bean的某一个属性没有注入，BeanFactory加载后，直至第一次使用调用getBean方法才会抛出异常。
- 通常使用编程方式被创建。
- 支持BeanPostProcessor、BeanFactoryPostProcessor使用，需要手动进行注册。



**ApplicationContext相关特性：**

- 它是在容器启动时，一次性创建了所有的Bean。这样在容器启动时，我们就可以发现spring中存在的配置错误，有利于检查所依赖的属性是否注入。ApplicationContext启动后预载入所有的单实例Bean，通过预载入单实例bean，确保你需要时，就不用等待。
- 使用声明的方式创建，如使用ContextLoader。
- 支持BeanPostProcessor、BeanFactoryPostProcessor使用，支持自动注册。



### 7.2 相关代码

**1：Spring Bean**

先从SpringBean说起，Spring Beans是被Spring容器管理的Java对象，比如：

```java
public class HelloWorld {
   private String message;
   public void setMessage(String message){
      this.message  = message;
   }
   public void getMessage(){
      System.out.println("My Message : " + message);
   }
}
```

我们一般通过Application.xml配置Spring Bean元数据。

```xml
<?xml version = "1.0" encoding = "UTF-8"?>
<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
   <bean id = "helloWorld" class = "com.zoltanraffai.HelloWorld">
      <property name = "message" value = "Hello World!"/>
   </bean>
</beans>
```

Spring Bean被Spring容器管理之后，可以在程序中获取使用它了。

**2: BeanFactory**

BeanFactory是Spring容器的基础接口，提供了基础的容器访问能力。

BeanFactory提供懒加载方式，只有通过getBean方法调用获取Bean才会进行实例化。

常用的是加载XMLBeanFactory：

```java
public class HelloWorldApp{
   public static void main(String[] args) {
      XmlBeanFactory factory = new XmlBeanFactory (new ClassPathResource("beans.xml"));
      HelloWorld obj = (HelloWorld) factory.getBean("helloWorld");
      obj.getMessage();
   }
}
```

1. XmlBeanFactory通过Resource装载Spring配置信息冰启动IoC容器，然后就可以通过factory.getBean从IoC容器中获取Bean了。
2. 通过BeanFactory启动IoC容器时，并不会初始化配置文件中定义的Bean，初始化动作发生在第一个调用时。
3. 对于单实例（singleton）的Bean来说，BeanFactory会缓存Bean实例，所以第二次使用getBean时直接从IoC容器缓存中获取Bean。

**3:ApplicationContext**

ApplicationContext继承自BeanFactory接口，ApplicationContext包含了BeanFactory中所有的功能。

具有自己独特的特性：

- Bean instantiation/wiriting
- Bean实例化/串联
- 
- 自动BeanFactoryPostProcessor注册
- 方便的MessageSource访问
- ApplicationEvent发布

**4.常用的获取ApplicationContext**

FileSystemXmlApplicationContext：从文件系统或者url指定的xml配置文件创建，参数为配置文件名或文件名数组，有相对路径与绝对路径。

```java
ApplicationContext factory=new FileSystemXmlApplicationContext("src/applicationContext.xml");
ApplicationContext factory=new FileSystemXmlApplicationContext("E:/Workspaces/MyEclipse 8.5/Hello/src/applicationContext.xml");
```

ClassPathXmlApplicationContext：从classpath的xml配置文件创建，可以从jar包中读取配置文件。ClassPathXmlApplicationContext 编译路径总有三种方式：

```java
ApplicationContext factory = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
ApplicationContext factory = new ClassPathXmlApplicationContext("applicationContext.xml"); 
ApplicationContext factory = new ClassPathXmlApplicationContext("file:E:/Workspaces/MyEclipse 8.5/Hello/src/applicationContext.xml");
```

XmlWebApplicationContext：从web应用的根目录读取配置文件，需要先在web.xml中配置，可以配置监听器或者servlet来实现

```xml
<listener>
<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

或

```xml
<servlet>
<servlet-name>context</servlet-name>
<servlet-class>org.springframework.web.context.ContextLoaderServlet</servlet-class>
<load-on-startup>1</load-on-startup>
</servlet>
```

这两种方式都默认配置文件为web-inf/applicationContext.xml，也可使用context-param指定配置文件

```xml
<context-param>
<param-name>contextConfigLocation</param-name>
<param-value>/WEB-INF/myApplicationContext.xml</param-value>
</context-param>
```

ApplicationContext采用的是预加载，每个Bean都在ApplicationContext启动后实例化。

```javascript
public class HelloWorldApp{
   public static void main(String[] args) {
      ApplicationContext context=new ClassPathXmlApplicationContext("beans.xml");
      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
      obj.getMessage();
   }
}
```

## 8: 说下BeanFactory 和 FactoryBean 区别

### 区别

1. BeanFactory:负责生产和管理Bean的一个工厂接口，提供一个Spring Ioc容器规范,
2. FactoryBean: 一种Bean创建的一种方式，对Bean的一种扩展。对于复杂的Bean对象初始化创建使用其可封装对象的创建细节。FactoryBean可以说为IOC容器中Bean的实现提供了更加灵活的方式，FactoryBean在IOC容器的基础上给Bean的实现加上了一个简单工厂模式和装饰模式，我们可以在getObject()方法中灵活配置。

### 8.1BeanFactory

这个其实是所有Spring Bean的容器根接口，给Spring 的容器定义一套规范，给IOC容器提供了一套完整的规范，比如我们常用到的getBean方法等。

### 8.2FactoryBean

> 该类是SpringIOC容器是创建Bean的一种形式，这种方式创建Bean会有加成方式，融合了简单的工厂设计模式于装饰器模式
> \* Interface to be implemented by objects used within a {@link BeanFactory} which * are themselves factories for individual objects. If a bean implements this * interface, it is used as a factory for an object to expose, not directly as a * bean instance that will be exposed itself.
> 有些人就要问了，我直接使用Spring默认方式创建Bean不香么，为啥还要用FactoryBean做啥，在某些情况下，对于实例Bean对象比较复杂的情况下，使用传统方式创建bean会比较复杂，例如（使用xml配置），这样就出现了FactoryBean接口，可以让用户通过实现该接口来自定义该Bean接口的实例化过程。即包装一层，将复杂的初始化过程包装，让调用者无需关系具体实现细节。

**方法：**

- T getObject()：返回实例
- Class<?> getObjectType();:返回该装饰对象的Bean的类型
- default boolean isSingleton():Bean是否为单例

**常用类：**

- ProxyFactoryBean :Aop代理Bean
- GsonFactoryBean:Gson

**使用：**

1. Spring XML方式

application.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="demo" class="cn.lonecloud.spring.example.bo.Person">
        <property name="age" value="10"/>
        <property name="name" value="xiaoMing"/>
    </bean>

    <bean id="demoFromFactory"      class="cn.lonecloud.spring.example.bean.PersonFactoryBean">
        <property name="initStr" value="10,init from factory"/>
    </bean>
</beans>
```

personFactoryBean.java

```java
package cn.lonecloud.spring.example.bean;

import cn.lonecloud.spring.example.bo.Person;
import org.springframework.beans.factory.FactoryBean;

import java.util.Objects;

/**
 * TODO
 *
 * @author lonecloud <lonecloud@aliyun.com>
 * @date 2020/8/24 16:00
 * @since v1.0
 */
public class PersonFactoryBean implements FactoryBean<Person> {

    /**
     * 初始化Str
     */
    private String initStr;

    @Override
    public Person getObject() throws Exception {
        //这里我需要获取对应参数
        Objects.requireNonNull(initStr);
        String[] split = initStr.split(",");
        Person p=new Person();
        p.setAge(Integer.parseInt(split[0]));
        p.setName(split[1]);
        //这里可能需要还要有其他复杂事情需要处理
        return p;
    }

    @Override
    public Class<?> getObjectType() {
        return Person.class;
    }

    public String getInitStr() {
        return initStr;
    }

    public void setInitStr(String initStr) {
        this.initStr = initStr;
    }
}
```

main方法

```java
package cn.lonecloud.spring.example.factory;

import cn.lonecloud.spring.example.bo.Person;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.FactoryBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * BeanFactory 例子
 *
 * @author lonecloud <lonecloud@aliyun.com>
 * @date 2020/8/24 16:00
 * @since v1.0
 */
public class SpringBeanFactoryMain {

    public static void main(String[] args) {
        //这个是BeanFactory
        BeanFactory beanFactory = new ClassPathXmlApplicationContext("application.xml");
        //获取对应的对象化
        Object demo = beanFactory.getBean("demo");
        System.out.println(demo instanceof Person);
        System.out.println(demo);
        //获取从工厂Bean中获取对象
        Person demoFromFactory = beanFactory.getBean("demoFromFactory", Person.class);
        System.out.println(demoFromFactory);
        //获取对应的personFactory  要加上 &
        Object bean = beanFactory.getBean("&demoFromFactory");
        System.out.println(bean instanceof PersonFactoryBean);
        PersonFactoryBean factoryBean=(PersonFactoryBean) bean;
        System.out.println("初始化参数为："+factoryBean.getInitStr());
    }
}
```

## 9:BeanPostProcessor 和 BeanFactoryPostProcessor 区别是什么？

### 9.1 作用及区别

从 [Spring+IOC容器源码分析](https://www.jianshu.com/p/518a14adbb69) 一文中的分析中，贯穿整个过程主线的大概是这样一条路线：

- 加载Bean定义信息
- 处理加载的Bean定义信息
- 从Bean定义创建Bean实例
- Bean创建的后置处理(属性填充、初始化回调等等)

当然这其中还有相当多的实现细节，包括 容器加载前的准备、Bean容器的准备等等。

这篇文章要说的是Spring框架中两个相当重要的类族：`BeanFactoryPostProcessor` 和 `BeanPostProcessor`

其应用的位置对应于上面的过程2和4，也就是：

- BeanFactoryPostProcessor：在实例化之前，整合 BeanDefinition 的过程中调用;操作对象为Bean定义
- BeanPostProcessor：在BeanDefinition注册完成之后进行注册，在创建Bean过程中的实例化前后分别调用其中定义的方法;其操作对象为：已经实例化且进行了属性填充待初始化的Bean实例

注意要区分这两个类的作用以及调用时间和作用对象都是不同的。

### 9.2 BeanFactoryPostProcessor的相关代码

​    BeanFactoryPostProcessor接口实现类可以在当前BeanFactory初始化后，bean实例化之前对BeanFactory做一些处理。**BeanFactoryPostProcessor是针对于bean容器的**，在调用它时，BeanFactory只加载了bean的定义，还没有对它们进行实例化，所以我们可以通过对BeanFactory的处理来达到影响之后实例化bean的效果。跟BeanPostProcessor一样，ApplicationContext也能自动检测和调用容器中的BeanFactoryPostProcessor。

​    BeanFactoryPostProcessor，是针对整个工厂生产出来的BeanDefinition**（未实例化）**作出修改或者注册。作用于BeanDefinition时期。BeanFactoryPostProcessor和其子类总共二个方法，从其入参就可以看出这二个方法的具体使用，如下：

   **postProcessBeanDefinitionRegistry方法**，此方法参数是一个BeanDefinitionRegistry，众所周知BeanDefinitionRegistry是用于beanDefinition注册、修改的。那么此方法在就可以在spring standard initialization后去修改一个beanDefinition、或者新增一个。这个类有个巨屌的实现类ConfigurationClassPostProcessor，这个类将处理configuration的类中以下注解：@Import，@PropertySource、@ComponentScan、@ImportResource、@Bean methods等。将这些注解涉及到的BeanDifinition注册到BeanDefinitionRegistry中。
**postProcessBeanFactory方法**，也是在实例化之前调用，也是读取bean定义并作出修改，比如属性修改，这个类spring也有一个巨牛的实现类PropertyPlaceholderConfigurer，这类的描述如下：将配置文件中的值注入到我们的代码中！！如数据源信息等。类解释如下：

   **例子1：**

​    实例化之前先输出一段话

```java
package com.meituan.hyt.test1;  
  
import org.springframework.beans.BeansException;  
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;  
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;  
  
public class UserBeanFactoryPostProcessor implements BeanFactoryPostProcessor {  
    @Override  
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {  
        System.out.println("BeanFactoryPostProcessor doing");  
    }  
}
```

applicationContext.xml中添加bean配置

```xml
<bean id="userBeanFactoryPostProcessor"               class="com.meituan.hyt.test1.UserBeanFactoryPostProcessor"/> 
```

重新运行，结果

```
BeanFactoryPostProcessor doing
User{name='新名字', id=100}
```

   **例子2：**

​     支持系统对username_进行占位符的配置为properties文件配置，试想如果我们有个配置中心，我们希望spring启动的时候，从远程配置中心取数据，而非本地文件，这里就需要我们自定义一个实现BeanFactoryPostProcessor的PropertyResourceConfigurer 实例。 

bean.xml配置： 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans   
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
    http://www.springframework.org/schema/context   
    http://www.springframework.org/schema/context/spring-context-3.0.xsd"  
    default-autowire="byName">  
  
     <bean id="user" class="com.gym.UserServiceImpl" >  
       <property name="username" value="${username_}"/>  
       <property name="password" value="${password_}"/>  
     </bean>  
       
     <bean id="myFactoryPostProcessor" class="com.gym.MyFilePlaceHolderBeanFactoryPostProcessor"/>  
</beans> 
```

模拟从远程取文件： 

```java
import org.springframework.beans.factory.InitializingBean;  
import org.springframework.beans.factory.config.PropertyPlaceholderConfigurer;  
import org.springframework.core.io.support.PropertiesLoaderUtils;  
  
/** 
 * @author xinchun.wang 
 */  
public class MyFilePlaceHolderBeanFactoryPostProcessor   
    extends PropertyPlaceholderConfigurer implements InitializingBean{  
      
    public void afterPropertiesSet() throws Exception {  
        List<Properties> list = new ArrayList<Properties>();  
        Properties p = PropertiesLoaderUtils.loadAllProperties("config.properties");  
        list.add(p);  
        //这里是关键，这就设置了我们远程取得的List<Properties>列表  
        setPropertiesArray(list.toArray(new Properties[list.size()]));  
    }  
      
} 
```

### 9.3 BeanPostProcessor的相关代码

​    如果这个接口的某个实现类被注册到某个容器，**那么该容器的每个受管Bean在调用初始化方法的前后，都会获得该接口实现类的一个回调**。容器调用接口定义的方法时会将该受管Bean的实例和名字通过参数传入方法，进过处理后通过方法的返回值返回给容器。

   BeanPostProcessor 的作用于bean实例化、初始化前后执行，先看下继承关系。





### 9.4 相关代码

```java
package com.mtons.mblog.web.controller.api;

import org.springframework.beans.BeansException;
import org.springframework.beans.MutablePropertyValues;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.stereotype.Component;

/**
 *实现该接口，可以在spring的bean创建之前，修改bean的定义属性。也就是说，
 * Spring允许BeanFactoryPostProcessor在容器实例化任何其它bean之前读取配置元数据，
 * 并可以根据需要进行修改，例如可以把bean的scope从singleton改为prototype，
 * 也可以把property的值给修改掉。可以同时配置多个BeanFactoryPostProcessor，
 * 并通过设置'order'属性来控制各个BeanFactoryPostProcessor的执行次序。
 注意：BeanFactoryPostProcessor是在spring容器加载了bean的定义文件之后，
 在bean实例化之前执行的。接口方法的入参是ConfigurrableListableBeanFactory，使用该参数，可以获取到相关bean的定义信息，
 * @author Administrator
 */
@Component
public class FactoryPostProcessor implements BeanFactoryPostProcessor {

    @Override    
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        System.out.println("******调用了BeanFactoryPostProcessor");
        String[] beanStr = configurableListableBeanFactory.getBeanDefinitionNames();
        for (String beanName : beanStr) {
            if ("person".equals(beanName)) {
                BeanDefinition beanDefinition = configurableListableBeanFactory.getBeanDefinition(beanName);
                MutablePropertyValues m = beanDefinition.getPropertyValues();
                if (m.contains("name")) {
                    m.addPropertyValue("name", "赵四");
                    System.out.println("》》》修改了name属性初始值了");
                }
            }
        }
    }

}
```

```java
ackage com.mtons.mblog.web.controller.api;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.stereotype.Component;

/**
 * BeanPostProcessor接口：这个接口是单独成类的，它的作用范围是Spring容器，一旦在容器中配置
 * 了这个类，那么该容器中所有bean在初始化
 * 的前后都会调用这个接口对应的方法
 */
@Component
public class PostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("后置处理器处理bean=【"+beanName+"】开始");
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("后置处理器处理bean=【"+beanName+"】完毕!");
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return bean;
    }
}
```



## 10：Spring Bean Scope 说下

   这个主要是spring得作用域。主要有以下5种：

### 10.1 Singleton(单例)

> singleton是容器默认的scope，所以写和不写没有区别。scope为singleton的时候，在Spring的IoC容器中只存在一个实例，所有对该对象的引用将共享这个实例。该实例从容器启动，并因为第一次被请求而初始化后，将一直存活到容器退出，也就是说，它与IoC容器“几乎”拥有相同的寿命

**singleton的bean具有的特性**

- **对象实例数量：**singleton类型的bean定义，在一个容器中只存在一个共享实例，所有对该类型bean的依赖都引用这一单一实例
- **对象存活时间：**singleton类型bean定义，从容器启动，到它第一次被请求而实例化开始，只要容器不销毁或退出，该类型的单一实例就会一直存活

### 10.2  prototype（多例）

> 对于那些请求方不能共享的对象实例，应该将其bean定义的scope设置为prototype。这样，每个请求方可以得到自己对应的一个对象实例。通常，声明为prototype的scope的bean定义类型，都是一些有状态的，比如保存每个顾客信息的对象

### 10.3 request

> 在Spring容器中，即XmlWebApplicationContext会为每个HTTP请求创建一个全新的Request-Processor对象供当前请求使用，当请求结束后，该对象实生命周期就结束。当同时有10个HTTP请求进来的时候，容器会分别针对这10个请求返回10个全新的RequestProcessor对象实例，且它们之间互不干扰。所以严格意义上说request可以看作prototype的一种特例，除了request的场景更加具体

### 10.4 session

> 放到session中的最普遍的信息就是用户登录信息，Spring容器会为每个独立的session创建属于它们自己全新的UserPreferences对象实例。与request相比，除了拥有session scope的bean比request scope的bean可能更长的存活时间，其他没什么差别

### 10.5 global session

> global session只有应用在基于portlet的Web应用程序中才有意义，它映射到portlet的global范围的session。如果在普通的基于servlet的Web应用中使了用这个类型的scope，容器会将其作为普通的session类型的scope来对待



## 11：Spring 的注入方式有几种，为什么推荐用构造器注入？

###  常见的三种注入方式

**1：field注入**

```java
@Controller
public class FooController {
  @Autowired
  //@Inject
  private FooService fooService;
  
  //简单的使用例子，下同
  public List<Foo> listFoo() {
      return fooService.list();
  }
}
```

这种注入方式应该是笔者目前为止开发中见到的最常见的注入方式。原因很简单：

1. 注入方式非常简单：加入要注入的字段，附上`@Autowired`，即可完成。
2. 使得整体代码简洁明了，看起来美观大方。

**2 ：构造器注入**

```java
@Controller
public class FooController {
  
  private final FooService fooService;
  
  @Autowired
  public FooController(FooService fooService) {
      this.fooService = fooService;
  }
  
  //使用方式上同，略
}
```

**3：setter注入**

```javascript
@Controller
public class FooController {
  
  private FooService fooService;
  
  //使用方式上同，略
  @Autowired
  public void setFooService(FooService fooService) {
      this.fooService = fooService;
  }
}
```

**为什么推荐构造器注入？**

- 依赖不可变：其实说的就是final关键字，这里不再多解释了。不明白的园友可以回去看看Java语法。
- 依赖不为空（省去了我们对其检查）：当要实例化FooController的时候，由于自己实现了有参数的构造函数，所以不会调用默认构造函数，那么就需要Spring容器传入所需要的参数，所以就两种情况：1、有该类型的参数->传入，OK 。2：无该类型的参数->报错。所以保证不会为空，Spring总不至于传一个null进去吧 :-( 
- 完全初始化的状态：这个可以跟上面的依赖不为空结合起来，向构造器传参之前，要确保注入的内容不为空，那么肯定要调用依赖组件的构造方法完成实例化。而在Java类加载实例化的过程中，构造方法是最后一步（之前如果有父类先初始化父类，然后自己的成员变量，最后才是构造方法，这里不详细展开。）。所以返回来的都是初始化之后的状态。
- 使用field注入可能会导致循环依赖，即A里面注入B，B里面又注入A：

当有一个依赖有多个实现的使用，推荐使用field注入或者setter注入的方式来指定注入的类型。



## 12 : @Resource 和 @Autowired 区别说下

​    @Autowired是Spring中的注解，@Resource是JSR-250中提供的注解，即Java提供的注解。

@Autowired的注入逻辑如下

1. 找到所有类型符合的bean

2. 如果没有类型符合的bean，则看@Autowired的required属性是否为true，是则抛出异常，否则返回null

3. 如果只有一个，则将这个bean注入

4. 如果有多个bean

     4.1 选择其中带有Primary注解的bean，如果只有一个直接注入，如果有多个bean带有Primary注解则报错，如果不存在就下一步 

     4.2 选择其中优先级最高的bean(优先级使用javax.annotation.Priority表明)，如果只有一个直接注入，如果有多个bean的优先级并列最高则报错，如果不存在就下一步 

     4.3 选择beanName和当前要注入的属性名相同的bean进行注入，有则注入，没有则报错

@Resource的注入逻辑如下

1. 如果@Resource指定了name，则只会按照name进行查找，当找不到时抛出异常，找到将bean注入
2. 如果@Resource没有指定name，则把属性名作为名字进行查找，找到将bean注入，当按照名字查找不到时，按照类型进行查找

**总结:**

@Autowired：先byType再byName

@Resource：先byName再byType（当指定@Resource name属性时，只会byName）



## 13: 事务相关问题

### 13.1 事务的种类？

   **编程式事务：**所谓编程式事务指的是通过编码方式实现事务，即类似于 JDBC 编程实现事务 管理。

   **声明式事务：**管理建立在 AOP 之上的。其本质是对方法前后进行拦截，然后在目标方法开 始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声 明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中 掺 杂 事 务 管 理 的 代 码 ， 只 需 在 配 置 文 件 中 做 相 关 的 事 务 规 则 声 明 ( 或 通 过 基 于 @Transactional 注解的方式)，便可以将事务规则应用到业务逻辑中

### 13.2 Spring 事务隔离级别说下

​    事务隔离级别指的是一个事务对数据的修改与另一个并行的事务的隔离程度，当多个事务同时访问相同数据时，如果没有采取必要的隔离机制。

​    当两个事务对同一个数据库的记录进行操作时，那么，他们之间的影响是怎么样的呢？这就出现了事务隔离级别的概念。数据库的隔离性与并发控制有很大关系。数据库的隔离级别是数据库的事务特性ACID的一部分，ACID，即原子性(atomicity)、一致性(consistency)、隔离性(isolation)和持久性(durability)。

| 隔离级别        | **描述**                                                     |
| --------------- | ------------------------------------------------------------ |
| DEFAULT         | 使用数据库本身使用的隔离级别    ORACLE（读已提交） MySQL（可重复读） |
| READ_UNCOMITTED | 读未提交（脏读）最低的隔离级别，一切皆有可能。 一个事务可以读取到另一个事务未提交的事务记录。 |
| READ_COMMITED   | 读已提交，ORACLE默认隔离级别，有幻读以及不可重复读风险。一个事务只能读取到已经提交的记录，不能读取未提交的记录。 |
| REPEATABLE_READ | 可重复读，解决不可重复读的隔离级别，但还是有幻读风险。一个事务可以多次从数据库读取某条记录，而且多次读取的那条记录都是一致的 |
| SERLALIZABLE    | 串行化，最高的事务隔离级别，不管多少事务，挨个运行完一个事务的所有子事务之后才可以执行另外一个事务里面的所有子事务，这样就解决了脏读、不可重复读和幻读的问题了。事务执行时，会在所有级别上加锁，比如read和write时都会加锁，仿佛事务是以串行的方式进行的，而不是一起发生的 |

这里解释下上面提到的几种异常：

1. 脏读：读到了其他事务还没有提交的数据。
2. 不可重复读：对某数据进行读取，发现两次读取的结果不同，也就是说没有读到相同的内容。这是因为有其他事务对这个数据同时进行了修改或删除。
3. 幻读：事务 A 根据条件查询得到了 N 条数据，但此时事务 B 更改或者增加了 M 条符合事务 A 查询条件的数据，这样当事务 A 再次进行查询的时候发现会有 N+M 条数据，产生了幻读。



### 13.3 Spring 的事务传播行为说下

| 事务行为                  | **说明**                                                     |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQUIRED      | 支持当前事务，假设当前没有事务。就新建一个事务               |
| PROPAGATION_SUPPORTS      | 支持当前事务，假设当前没有事务，就以非事务方式运行           |
| PROPAGATION_MANDATORY     | 支持当前事务，假设当前没有事务，就抛出异常                   |
| PROPAGATION_REQUIRES_NEW  | 新建事务，假设当前存在事务。把当前事务挂起                   |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式运行操作。假设当前存在事务，就把当前事务挂起     |
| PROPAGATION_NEVER         | 以非事务方式运行，假设当前存在事务，则抛出异常               |
| PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |





### 13.4 Spring 事务实现原理





## 14:循环依赖

主要分下面几点

1. 什么是循环依赖？
2. 什么情况下循环依赖可以被处理？
3. Spring是如何解决的循环依赖？

### 14.1 概念

​    从字面上来理解就是A依赖B的同时B也依赖了A。