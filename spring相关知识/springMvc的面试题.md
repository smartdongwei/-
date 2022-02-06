# Spring MVC的面试题

## 1: Spring MVC 执行流程说下

（1）整个过程始于客户端发出一个HTTP请求，Web应用服务器接收到这个请求。如果匹配DispatcherServlet的请求映射路径（在web.xml中指定），则Web容器将该请求转交给DispatcherServlet(前端控制器)处理。

（2）DispatcherServlet(前端控制器))接收到这个请求后，将根据请求的信息（包括 URL、HTTP方法、请求报文头、请求参数、Cookie等）调用处理器映射器HandlerMapping。

（3）处理器映射器根据请求url找到具体的处理器，生成处理器执行链HandlerExecutionChain(包括处理器对象和处理器拦截器)一并返回给DispatcherServlet。可将HandlerMapping看作路由控制器，将Handler看作目标主机

（4）当DispatcherServlet 根据HandlerMapping得到对应当前请求的Handler后，通过HandlerAdapter对Handler进行封装，再以统一的适配器接口调用Handler。HandlerAdapter 是Spring MVC的框架级接口，顾名思义，HandlerAdapter是一个适配器，它用统一的接口对各种Handler方法进行调用。

（5）处理器完成业务逻辑的处理后将返回一个ModelAndView给DispatcherServlet，ModelAndView包含了视图逻辑名和模型数据信息。

（6）ModelAndView中包含的是“逻辑视图名”而非真正的视图对象，DispatcherServlet借由ViewResolver完成逻辑视图名到真实视图对象的解析工作。

（7）当得到真实的视图对象View后，DispatcherServlet就使用这个View对象对ModelAndView中的模型数据进行视图渲染。

（8）最终客户端得到的响应消息可能是一个普通的HTML页面，也可能是一个XML或JSON串，甚至是一张图片或一个PDF文档等不同的媒体形式。

## 2:@RestController 和 @Controller 区别说下

​    @Controller：标识一个Spring类是Spring MVC controller处理器,@RestController：@RestController是@Controller和@ResponseBody的结合体，两个标注合并起来的作用。@Controller类中的方法可以直接通过返回String跳转到jsp、ftl、html等模版页面。在方法上加@ResponseBody注解，也可以返回实体对象。@RestController类中的所有方法只能返回String、Object、Json等实体对象，不能跳转到模版页面。

@RestController中的方法如果想跳转页面，则用ModelAndView进行封装，如下：

```java
@RestController
public class UserController {

    @RequestMapping(value = "/index",method = RequestMethod.GET)
    public String toIndex(){
        ModelAndView mv = new ModelAndView("index");
      	return mv;    
    }
} 
```

## 3: 怎么取得 URL 中的 { } 里面的变量？

URL示例2：http://www.iteye.com/problems/101566?Key=123，{}里面是路径参数 ，所以代码中用的应该是@PathVariable；对于参数部分Key=123，如果想获取到该参数那么应该用参数获取的注解@RequestParam



## 4:Spring MVC怎么样设定重定向和转发的

如果返回的字符串中带 forward: 或 redirect: 前缀时,SpringMVC 会对他们进行特殊处理:将 forward: 和redirect: 当成指示符,其后的字符串作为 URL 来处理。

（1）转发：在返回值前面加"forward:"，譬如"forward:user.do?name=method4"

（2）重定向：在返回值前面加"redirect:"，譬如"redirect:http://www.baidu.com"


## 5:说下 Spring MVC 的常用注解。

**5.1@RequestMapping**

> **指定请求路径**
>
> - value: 指定接收的路径
> - method: 接收什么请求（get repost…）
> - params： 对参数的限制
> - headers: 浏览器的请求头

**5.2@RequestParam**

**作用**

> 1. 设置默认值
> 2. 定义映射规则

**参数**

> - name： 浏览器中的key
> - defaultValue: 默认值
> - required: 参数是否必须传

**5.3 @RequestHeader**

获取请求头信息

**5.4  @CookieValue**

获取cookie

**5.5 @ResponseBody**

返回 jackson对象，需要导入依赖 jackson-databind

**5.6 @RequestBody**

接收前端传过来的json对象

**5.7 @PathVariable**

取出请求中占位符对应的参数

**5.8 @SessionAttributes**

加在类上，指定哪些属性名，或者哪些属性值会被加入到session中

value: 属性名称，以name, age为key 的会被加入到session中

type：哪些数据类型的属性值会被加入到session中



## 6:如何解决 POST 请求中文乱码问题，GET 的又如何处理呢？



## 7:Interceptor（拦截器） 和 Filter（过滤器） 区别？

- Filter是基于函数回调的，而Interceptor则是基于Java反射的。
- Filter依赖于Servlet容器，而Interceptor不依赖于Servlet容器。
- Filter对几乎所有的请求起作用，而Interceptor只能对action请求起作用。
- Interceptor可以访问Action的上下文，值栈里的对象，而Filter不能。
- 在action的生命周期里，Interceptor可以被多次调用，而Filter只能在容器初始化时调用一次。

最简单明了的区别就是**过滤器可以修改request，而拦截器不能。过滤器需要在servlet容器中实现，拦截器可以适用于javaEE，javaSE等各种环境。拦截器可以调用IOC容器中的各种依赖，而过滤器不能。过滤器只能在请求的前后使用，而拦截器可以详细到每个方法**。

使用 第一步：定义一个拦截器，实现 HandlerInterceptor 接口

```java
 public class FirstInterceptor implements HandlerInterceptor { 
     @Override 
     public void afterCompletion(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, Exception arg3) throws Exception {
         System.out.println("[FirstInterceptor] afterCompletion"); 
     } 
     @Override public void postHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, ModelAndView arg3) throws Exception { 
         System.out.println("[FirstInterceptor] postHandle"); 
     } 
     @Override 
 public boolean preHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2) throws Exception { 
     System.out.println("[FirstInterceptor] preHandle"); 
     return true; 
 } 
 }
```

 第二步：在 springmvc 配置文件中进行如下配置：   [FirstInterceptor] preHandle //controller处理方法被调用 [FirstInterceptor] postHandle [FirstInterceptor] afterCompletion 顺序 类似于 Filter，当返回 true 时，继续检测其他拦截器，返回 false，直接调到放行过的拦截器 的 afterComplication 方法。 preHandle 是在调用处理方法之前调用 postHandle 是在调用处理方法之后，渲染视图之前被调用。 afterCompletion 是在渲染视图之后被调用 

## 8:怎样在方法里面得到 Request，或者 Session？

 直接在方法的形参中声明request, SpringMvc就会自动把Request对象传入。

## 9:Spring MVC 中函数的返回值是什么？怎么样把 ModelMap 里面的数据放入 Session 里面？

​    可以在类上面加上@SessionAttributes 注解,里面包含的字符串就是要放入 session 里面的 key

## 10:Spring MVC 的控制器是不是单例模式,如果是,有什么问题,怎么解决？

​    默认情况下是单列模式，单例是线程不安全的，会导致属性的重复性利用。不要在controller中定义成员变量。

万一必须要定义一个非静态成员变量时候，则通过注解@Scope(“prototype”)，将其设置为多例模式。

java里，每个线程都有自己独享的空间，也就是栈内存。线程在调用方法时，会创建一个栈帧。调用一个方法，就是一个栈帧的入栈过程，该方法执行完毕，栈帧也就出栈了。所以成员方法对于每个线程事实上是私有的。

## 11:Spring MVC 的 RequestMapping 的方法是线程安全的吗？为什么？





## 12:介绍下 WebApplicationContext。跨域问题如何解决





## 13:如何解决全局异常？

​     通过 @ControllerAdvice 注解，我们可以在一个地方对所有 @Controller 注解的控制器进行管理。
注解了 @ControllerAdvice 的类的方法可以使用 @ExceptionHandler、 @InitBinder、 @ModelAttribute 注解到方法上，这对所有注解了 @RequestMapping 的控制器内的方法都有效。
