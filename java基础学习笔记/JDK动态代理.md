





## 一、为什么需要动态代理

### 1.1 从代理模式说起

   代理模式是指给某一个对象提供一个代理对象，并由代理对象控制对原对象的引用。通俗的来讲代理模式就是我们生活中常见的中介。这种模式有什么用呢？它可以在原对象的基础上增强原对象的功能，比如在原对象调用一个方法的前后进行日志、事务操作等。Spring AOP就使用了代理模式。

### 1.2 代理模式----静态代理

   如何实现代理模式呢？首先来看静态代理。静态代理是指在程序运行前就已经存在的编译好的代理类是为静态代理。实现静态代理有四个步骤：
   ①定义业务接口；
   ②被代理类实现业务接口；
   ③定义代理类并实现业务接口；
   ④最后便可通过客户端进行调用。（这里可以理解成程序的main方法里的内容）

   我们按照这个步骤去实现静态代理。需求：在向数据库添加一个用户时前后打印日志。

#### 1.2.1业务接口

IUserService.java

```java
package com.zhb.jdk.proxy;
/**
 *@author  ZHB
 *@date    2018年8月31日下午10:44:49
 *@todo    TODO
*/
public interface IUserService {
    void add(String name);  
}
```

#### 1.2.2被代理类实现业务接口

UserServiceImpl.java

```java
package com.zhb.jdk.proxy;
/**
 *@author  ZHB
 *@date    2018年8月31日下午10:47:25
 *@todo    TODO
*/
public class UserServiceImpl implements IUserService{

    @Override
    public void add(String name) {
        System.out.println("向数据库中插入名为：  "+name+" 的用户");
    }

}
```

#### 1.2.3定义代理类并实现业务接口

   因为代理对象和被代理对象需要实现相同的接口。所以代理类源文件UserServiceProxy.java这么写：

```java
package com.zhb.jdk.proxy;
/**
 * @author ZHB
 * @date 2018年8月31日下午10:51:39
 * @todo TODO
 */
public class UserServiceProxy implements IUserService {
    // 被代理对象
    private IUserService target;

    // 通过构造方法传入被代理对象
    public UserServiceProxy(IUserService target) {
        this.target = target;
    }

    @Override
    public void add(String name) {
        System.out.println("准备向数据库中插入数据");
        target.add(name);
        System.out.println("插入数据库成功");

    }
}
```

 由于代理类(UserServiceProxy )和被代理类(UserServiceImpl )都实现了IUserService接口，所以都有add方法，在代理类的add方法中调用了被代理类的add方法，并在其前后各打印一条日志。

#### 1.2.4客户端调用

```java
package com.zhb.jdk.proxy;

/**
 *@author  ZHB
 *@date    2018年8月31日下午11:01:12
 *@todo    TODO
*/
public class StaticProxyTest {

    public static void main(String[] args) {
        IUserService target = new UserServiceImpl();
        UserServiceProxy proxy = new UserServiceProxy(target);
        proxy.add("陈粒");
    }
}
```

### 1.3 代理模式----动态代理

   既然有了静态代理，为什么会出现动态代理呢？那我们就说一下静态代理的一些缺点吧：
   ①代理类和被代理类实现了相同的接口，导致代码的重复，如果接口增加一个方法，那么除了被代理类需要实现这个方法外，代理类也要实现这个方法，增加了代码维护的难度。
   ②代理对象只服务于一种类型的对象，如果要服务多类型的对象。势必要为每一种对象都进行代理，静态代理在程序规模稍大时就无法胜任了。比如上面的例子，只是对用户的业务功能（IUserService）进行代理，如果是商品（IItemService）的业务功能那就无法代理，需要去编写商品服务的代理类。
   于是乎，动态代理的出现就能帮助我们解决静态代理的不足。**所谓动态代理是指：在程序运行期间根据需要动态创建代理类及其实例来完成具体的功能。**动态代理主要分为JDK动态代理和cglib动态代理两大类，本文主要对JDK动态代理进行探讨。

## 二、JDK动态代理实例

### 2.1 使用JDK动态代理步骤

   ①创建被代理的接口和类；
   ②创建InvocationHandler接口的实现类，在invoke方法中实现代理逻辑；
   ③通过Proxy的静态方法`newProxyInstance( ClassLoaderloader, Class[] interfaces, InvocationHandler h)`创建一个代理对象
   ④使用代理对象。

### 2.2 例子

#### 2.2.1 创建被代理的接口和类

   这个和静态代理的源码相同，还是使用上面的`IUserService.java`和`UserServiceImpl.java`

#### 2.2.2 创建InvocationHandler接口的实现类

```java
package com.wdw.work.dynamicproxy.jdk;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * @author wang
 */
public class MyInvocationHandler implements InvocationHandler {

    /**
     *     被代理对象，Object类型
     */
    private Object target;

    public MyInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("准备向数据库中插入数据");
        Object returnValue = method.invoke(target, args);
        System.out.println("插入数据库成功");
        return returnValue;
    }
}
```

#### 2.2.3 通过Proxy的静态方法创建代理对象并使用代理对象

```java
package com.wdw.work.dynamicproxy;

import com.wdw.work.dynamicproxy.jdk.MyInvocationHandler;
import com.wdw.work.dynamicproxy.proxy.IUserService;
import com.wdw.work.dynamicproxy.proxy.UserServiceImpl;

import java.lang.reflect.Proxy;

/**
 * @author wang
 */
public class DynamicProxyTest {

    public static void main(String[] args) {

        IUserService target = new UserServiceImpl();
        MyInvocationHandler handler = new MyInvocationHandler(target);
        //第一个参数是指定代理类的类加载器（我们传入当前测试类的类加载器）
        //第二个参数是代理类需要实现的接口（我们传入被代理类实现的接口，这样生成的代理类和被代理类就实现了相同的接口）
        //第三个参数是invocation handler，用来处理方法的调用。这里传入我们自己实现的handler
        IUserService proxyObject = (IUserService) Proxy.newProxyInstance(DynamicProxyTest.class.getClassLoader(),
                target.getClass().getInterfaces(), handler);
        proxyObject.add("王东威");
    }
}
```

   实际上，动态代理的代理对象是在内存中的，是JDK根据我们传入的参数生成好的。

### 2.3 动态代理源码深入分析

`    Proxy.newProxyInstance( ClassLoaderloader, Class[] interfaces, InvocationHandler h)`产生了代理对象，所以我们进到`newProxyInstance`的实现：

```java
public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        //检验h不为空，h为空抛异常
        Objects.requireNonNull(h);
        //接口的类对象拷贝一份
        final Class<?>[] intfs = interfaces.clone();
        //进行一些安全性检查
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        /*
         * Look up or generate the designated proxy class.
         *  查询（在缓存中已经有）或生成指定的代理类的class对象。
         */
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * Invoke its constructor with the designated invocation handler.
         */
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }
            //得到代理类对象的构造函数，这个构造函数的参数由constructorParams指定
            //参数constructorParames为常量值：private static final Class<?>[] constructorParams = { InvocationHandler.class };
            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            //这里生成代理对象，传入的参数new Object[]{h}后面讲
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }
```



## 三、使用cglib动态代理步骤

简单来说，cglib 就是用来自动生成代理类的。与 JDK 自带的动态代理相比，有以下几点不同：

1. JDK 动态代理要求被代理类实现某个接口，而 cglib 无该要求。
2. **在目标方法的执行速度上，由于采用了`FastClass`机制，cglib 更快**（以空间换时间，后面会讲到）。

### 3.1 定义代理类的逻辑

首先，我们需要在某个地方定义好代理类的逻辑，在本文的例子中，代理的逻辑就是在方法执行前打印入参，方法执行后打印出参。我们可以通过实现`MethodInterceptor`接口来定义这些逻辑。根据 aop 联盟的标准（可以自行了解下），`MethodInterceptor`属于一种`Advice`。

需要注意一点，**这里要通过`proxy.invokeSuper`来调用目标类的方法，而不是使用`method.invoke`，不然会出现栈溢出等问题**。如果你非要调用`method.invoke`，你需要把目标类对象作为`LogInterceptor`的成员属性，在调用`method.invoke`时将它作为入参，而不是使用`MethodInterceptor.intercept`的入参 obj，但是，我不推荐你这么做，因为你将无法享受到 cglib 代理类执行快的优势（然而还是很多人这么做）。

```java
public class LogInterceptor implements MethodInterceptor {
    // 这里传入的obj是代理类对象，而不是目标类对象
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.err.println("打印" + method.getName() + "方法的入参");
        // 注意，这里要调用proxy.invokeSuper，而不是method.invoke，不然会出现栈溢出等问题
        Object obj2 = proxy.invokeSuper(obj, args);
        System.err.println("打印" + method.getName() + "方法的出参");
        return obj2;
    }
}
```

### 3.2 获取代理类

我们主要通过`Enhancer`来配置、获取代理类对象，下面的代码挺好理解的，我们需要告诉 cglib，**我要代理谁，代理的逻辑放在哪里**。

```java
@Test
public void testBase() throws InterruptedException {
    // 设置输出代理类到指定路径，便于后面分析
    System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "D:/growUp/test");
    // 创建Enhancer对象
    Enhancer enhancer = new Enhancer();
    // 设置哪个类需要代理
    enhancer.setSuperclass(UserController.class);
    // 设置怎么代理
    enhancer.setCallback(new LogInterceptor());
    // 获取代理类实例
    UserController userController = (UserController) enhancer.create();
    // 测试代理类
    System.out.println("-------------");
    userController.save();
    System.out.println("-------------");
    userController.delete();
    System.out.println("-------------");
    userController.update();
    System.out.println("-------------");
    userController.find();
}
```

我们也可以同时设置多个`Callback`，需要注意的是，**设置了多个`Callback`不是说一个方法可以被多个`Callback`拦截，而是说目标类中不同的方法可以被不同的`Callback`拦截**。所以，当设置了多个`Callback`时，cglib 需要知道哪些方法使用哪个`Callback`，我们需要额外设置`CallbackFilter`来指定每个方法使用的是哪个`Callback`

### 3.3 代理类源码分析

接下来我们来看看 cglib 的源码。

cglib **如何生成代理类**的源码就不分析了，感兴趣的可以自行研究（cglib 的源码可读性还是很强的），我们只要记住两点就行，1. cglib 的代理类会缓存起来，不会重复创建；2. 使用的是 asm 来生成`Class`文件。

我们直接来看**代理类方法执行**的源码。

#### 3.3.1代理类文件

在上面例子中，我们指定的文件夹下生成了三个文件，一个代理类文件，两个`FastClass`文件，通过 debug 可以发现，代理类文件是调用`Enhancer.create`的时候生成的，而两个`FastClass`文件是第一次调用`MethodProxy.invokeSuper`的时候才生成。这两个`FastClass`是用来干嘛的？

#### 3.3.2代理类的源码

下面看看代理类文件的源码（本文采用`Luyten`作为反编译工具，**考虑篇幅问题，这里仅展示 update 方法**）。

在静态代码块执行时，会初始化目标类方法对应的`Method`对象，也会初始化每个`Method`对应的`MethodProxy`，我们需要关注下`MethodProxy.create`方法。

另外，我们需要注意两个方法，一个是`update`方法，该方法中会去调用我们定义的`MethodInterceptor`的`intercept`方法，另一个是`CGLIB$update$0`方法，该方法直接调用`UserController`的`update`方法。后面我们会发现，`update`方法最终会调用`CGLIB$update$0`方法，这就是 cglib 不使用反射而直接调用目标方法的关键。

```java
//生成类的名字规则是：被代理classname + "$$"+classgeneratorname+"ByCGLIB"+"$$"+key的hashcode
public class UserController$$EnhancerByCGLIB$$e6f193aa extends UserController implements Factory {
    private boolean CGLIB$BOUND;
    public static Object CGLIB$FACTORY_DATA;
    private static final ThreadLocal CGLIB$THREAD_CALLBACKS;
    private static final Callback[] CGLIB$STATIC_CALLBACKS;

    //我们一开始传入的MethodInterceptor对象  zzs001
    private MethodInterceptor CGLIB$CALLBACK_0;
    private static Object CGLIB$CALLBACK_FILTER;
    //目标类的update方法对象
    private static final Method CGLIB$update$0$Method;
    //代理类的update方法对象
    private static final MethodProxy CGLIB$update$0$Proxy;
    private static final Object[] CGLIB$emptyArgs;

    static void CGLIB$STATICHOOK1() {
        CGLIB$THREAD_CALLBACKS = new ThreadLocal();
        CGLIB$emptyArgs = new Object[0];
        final Class<?> forName = Class.forName("cn.zzs.cglib.UserController$$EnhancerByCGLIB$$e6f193aa");
        final Class<?> forName2;
        final Method[] methods = ReflectUtils.findMethods(new String[]{"update", "()V", "find", "()V", "delete", "()V", "save", "()V"},
                (forName2 = Class.forName("cn.zzs.cglib.UserController")).getDeclaredMethods());
        // 初始化目标类的update方法对象
        CGLIB$update$0$Method = methods[0];
        // 初始化代理类update方法对象
        CGLIB$update$0$Proxy = MethodProxy.create((Class) forName2, (Class) forName, "()V", "update", "CGLIB$update$0");
    }

    // 这个方法将直接调用UserController的update方法
    final void CGLIB$update$0() {
        super.update();
    }

    public final void update() {
        MethodInterceptor cglib$CALLBACK_2;
        MethodInterceptor cglib$CALLBACK_0;
        if ((cglib$CALLBACK_0 = (cglib$CALLBACK_2 = this.CGLIB$CALLBACK_0)) == null) {
            CGLIB$BIND_CALLBACKS(this);
            cglib$CALLBACK_2 = (cglib$CALLBACK_0 = this.CGLIB$CALLBACK_0);
        }
        //一般走这里，即调用我们传入MethodInterceptor对象的intercept方法
        if (cglib$CALLBACK_0 != null) {
            cglib$CALLBACK_2.intercept((Object) this, UserController$$EnhancerByCGLIB$$e6f193aa.CGLIB$update$0$Method, UserController$$EnhancerByCGLIB$$e6f193aa.CGLIB$emptyArgs, UserController$$EnhancerByCGLIB$$e6f193aa.CGLIB$update$0$Proxy);
            return;
        }
        super.update();
    }
}
```

#### 3.3.3 创建FastClass文件

在`MethodProxy.invokeSuper(Object, Object[])`方法中，我们会发现，两个`FastClass`文件是在`init`方法中生成的。当然，它们也只会创建一次。

我们用到的主要是代理类的`FastClass`，通过它，我们可以直接调用到`CGLIB$update$0`方法，相当于可以直接调用目标类的`update`方法。

```java
public Object invokeSuper(Object obj, Object[] args) throws Throwable {
        try {
            //初始化，创建了两个FastClass类对象
            init();
            FastClassInfo fci = fastClassInfo;
            // 这里将直接调用代理类的CGLIB$update$0方法，而不是通过反射调用
            // fci.f2：代理类的FastClass对象，fci.i2为CGLIB$update$0方法对应的索引，obj为当前的代理类对象，args为update方法的参数列表
            return fci.f2.invoke(fci.i2, obj, args);
        } catch (InvocationTargetException e) {
            throw e.getTargetException();
        }
    }
    private void init(){  
        if (fastClassInfo == null){  
            synchronized (initLock){  
                if (fastClassInfo == null){  
                    CreateInfo ci = createInfo;  
                    FastClassInfo fci = new FastClassInfo();  
                    // 创建代理类的FastClass对象
                    fci.f1 = helper(ci, ci.c1);  
                    // 创建目标类的FastClass对象
                    fci.f2 = helper(ci, ci.c2);  
                    // 获取update方法的索引
                    fci.i1 = fci.f1.getIndex(sig1);  
                    // 获取CGLIB$update$0方法的索引，这个很重要
                    fci.i2 = fci.f2.getIndex(sig2);  
                    fastClassInfo = fci;  
                    createInfo = null;  
                }  
            }  
        }  
    }
```

#### 3.3.4 FastClass的作用

打开代理类的`FastClass`文件，可以看到，通过方法索引我们可以匹配到`CGLIB$update$0`方法，并且直接调用它，而不需要像 JDK 动态代理一样通过反射的方式调用，极大提高了执行效率。

```java
//传入参数：
    //n：方法索引
    //o：代理类实例
    //array：方法输入参数
    public Object invoke(final int n, final Object o, final Object[] array) throws InvocationTargetException {
        final UserController$$EnhancerByCGLIB$$e6f193aa userController$$EnhancerByCGLIB$$e6f193aa = (UserController$$EnhancerByCGLIB$$e6f193aa)o;
        try {
            switch (n) {
                case 0: {
                    return new Boolean(userController$$EnhancerByCGLIB$$e6f193aa.equals(array[0]));
                }
                case 1: {
                    return userController$$EnhancerByCGLIB$$e6f193aa.toString();
                }
                case 2: {
                    return new Integer(userController$$EnhancerByCGLIB$$e6f193aa.hashCode());
                }
                case 3: {
                    return userController$$EnhancerByCGLIB$$e6f193aa.clone();
                }
                // ·······
                case 24: {
                    // 通过匹配方法索引，直接调用该方法，这个方法将直接调用代理类的超类的方法
                    userController$$EnhancerByCGLIB$$e6f193aa.CGLIB$update$0();
                    return null;
                }
                // ·······

        }
        catch (Throwable t) {
            throw new InvocationTargetException(t);
        }
        throw new IllegalArgumentException("Cannot find matching method/constructor");
    }
```

通过上面的分析，我们找到了 cglib 代理类执行起来更快的原因。

### 3.4 总结

cglib 就是用来自动生成代理类的。与 JDK 自带的动态代理相比，有以下几点不同：

1. JDK 动态代理要求被代理类实现某个接口，而 cglib 无该要求。
2. **在目标方法的执行速度上，由于采用了`FastClass`机制，cglib 更快**（以空间换时间，后面会讲到）。