---
title: springboot函数的解析
tags: [springboot]
date: 2020-11-11 10:00:00
---



###1：coditional 的用法

​    @Conditional注解是个什么东西呢，它可以根据代码中设置的条件装载不同的bean

1. 首先自定义一个规则类

```java
public class MyCondition implements Condition    { 
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata)        
    {        		
        //在这里写你的逻辑，比如说你想a>0时实例化类A，a<0时不实现       
        return a>0;      
    }    
}
```

  2.然后你就可以用在bean上

```java
@Bean    
@Conditional(MyCondition.class)   
public A a(){    
    return new A()  
}
```

也可以用在类上面

```java
@Configuration    
@Conditional(MyCondition.class)   
public class b{    
    @Bean
    public aaa aaa(){
        return null;
    }
}
```



###2：springBeanUtil 



