## 1:获取bean的相关代码

```java
ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");
SimpleBean simpleBean = applicationContext.getBean(SimpleBean.class);
```

   bean的获取的关键代码doGetBean为：

```java
protected <T> T doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly) throws BeansException {
    // 获取规范的bean名称
    String beanName = this.transformedBeanName(name);
    //从缓存中获取单列bean
    Object sharedInstance = this.getSingleton(beanName);
    Object beanInstance;
    // 存在这个单列的bean并且没有构造参数
    if (sharedInstance != null && args == null) {
        // 返回这个bean
    }else{
        if (正在初始化中的多例bean存在该beanname){抛出异常}
        获取当前容器的父容器(getParentBeanFactory)
        若父容器存在则委派父容器查找该bean对应的beanDefinition
    }
    获取该beanName对应的BeanDefinition,包装为RootBeanDefinition返回。
    获取依赖bean名称集合
    递归过去依赖bean
    if(bean为单例){
        创建单例bean
    } 
    if(bean为多例){
        创建多例bean
    } 
    如果bean的作用域时其它 则创建相关的bean
    对创建的bean实例对象进行类型检查  
}
```