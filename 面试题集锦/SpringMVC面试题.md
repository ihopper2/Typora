## 1.概述

### 1.1 什么是SpringMVC,介绍一下你对于SpringMVC的理解

SpringMVC是一个基于java实现了MVC设计模式的请求驱动类型的轻量级Web框架，通过把模型-试图-解析器分离，将web层进行责任解耦，把复杂的web应用分成逻辑清晰的几部分，简化开发。

### 1.2 SpringMVC的优点

- 可以支持各种视图技术，而不仅仅是局限于JSP
- 可以与Spring框架集成
- 清晰的角色分配：
- 支持各种请求资源的映射

## 2.核心组件

### 2.1 SpringMVC的主要组件

- 前端控制器（DispatcherServlet）：接受请求，响应结果，相当于转发器
- 请求映射处理器（handlerMapping）：不需要程序员开发，根据url寻找Handler
- 处理器适配器（HandlerAdapter）:在编写Handler的时候要根据HandlerAdapter要求规则去编写，只有这样适配器才能正确的去执行Handler
- 处理器（Handler）：需要程序员开发
- 视图解析器（ViewResolver）：不需要程序员开发，进行视图解析，根据试图逻辑名解析真正的视图（view）
- 视图View（需要程序员开发jsp）：view作为一个接口，它的实现类支持不同的试图类型

### 2.2 什么是DispatcherServlet

SpringMVC框架是围绕着DispatcherServlet来设计的，它是用来处理和响应所有的HTTP请求和响应。

### 2.3 什么是SpringMVC框架的控制器

控制器是一个访问应用程序的行为，此行为通常通过服务接口实现的，控制器解析用户输入并将其转换为一个由视图呈现给用户的模型。Spring用一个非常抽象的方式实现了一个控制层，允许用户创建多种用途的控制器。

### 2.4 Spring MVC的控制器是不是单例模式,如果是,有什么问题,怎么解决？

是单例模式，所以在多线程访问的时候有线程安全的问题，不要用同步，会影响性能。

我们可以在控制器里面不能写字段。

### 2.5 SpringMVC工作流程

- 用户发送请到前端控制器DispatcherServlet
- DS收到请求后，调用请求映射处理器HabdlerMapping，请求获得Handler
- 映射处理器根据url寻找具体的处理器，生成处理器对象及处理器拦截器，返回给DS
- DS调用适配器HandlerAdapter
- handlerAdapter调用具体的处理器Handler
- Handler处理之后返回ModelAndView
- HandlerAdapter将Handler的返回结果和ModelAndView返回给DS
- DS将ModelandView传给ViewResolver视图解析器解析
- ViewResolver解析后返会给具体View
- DS对具体的View进行渲染
- DS响应用户

![image-20210816164330634](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20210816164330634.png)

### 2.6 MVC是什么

mvc是一中设计模式，模型-视图-解析器三层分离，用于实现前端界面和后端处理业务的分离

- 分层设计，实现了各个业务的解耦，有利于业务扩展
- 提升了工作效率

## 3.常用注解

### 3.1 注解原理是什么

注解的本质是继承了一个Annotation的接口，其具体实现类是Java运行时生成的动态代理。我们通过反射获取注解的时候，获得是java运行时生成的动态代理对象。通过代理对象调用自定义注解的方法，最终会调用AnnotationInvocationHandler的invoke方法。

### 3.2 SpringMVC常用的注解

@RequstMapping：用于处理uri的映射请求，可用于类上或者方法上。

@RequestBody：注解实现接收http请求的json数据，将json数据转换为java对象。

@ResponseBody：注解实现将Controller方法返回转化为json对象响应客户。

### 3.3 SpingMvc中的控制器的注解一般用哪个,有没有别的注解可以替代？

一般用@Controller注解,也可以使用@RestController,@RestController注解相当于@ResponseBody ＋ @Controller,表示是表现层,除此之外，一般不用别的注解代替。

### 3.4 @Controller注解的作用

在SpringMVC中@Controller负责处理DS分发的请求，他把用户请求的数据经过业务处理层之后封装成一个Model，然后把这个Model返回给View进行展示。我们只需要在一个类上加上@Controller，然后使用@RequestMapping和@RequestParam等一些注解用以定义URL请求和Controller方法之间的映射。

### 3.5 @RequestMapping注解作用

这是一个用来处理请求地址映射的映射，可以用在方法或者类上。用在类上表示类中的所有请求方法都是以该地址作为父路径。

他有六个属性

- value：请求的实际地址
- method：请求的类型，post get put delete
- cousumes：请求提交处理内容的类型
- produces：指定返回的内容类型
- params：指的是request中要包含某些参数值，才能让该方法处理请求。
- headers：request中要包含某些指定的header值，才能让该方法处理请求。

### 3.6 @ResponseBody注解作用

该注解是将Controller方法返回的对象，通过适当的HttpMessageConverter转换为指定的格式以后，写入到Reponse对象的body区域

### 3.7 @PathVariable和@RequestParam的区别

@RequestParam和@PathVariable这两者之间区别不大，主要是请求的URL不一样

用@RequestParam请求接口时,URL是:http://www.test.com/user/getUserById?userId=1

```java
@GetMapping(value="getUserById",produces="application/json;charset=utf-8")
public Object getUserById(@RequestParam String userId) {
```

用@PathVariable请求接口时,URL是:http://www.test.com/user/getUserById/2

```java
@GetMapping(value="getUserById/{userId}",produces="application/json;charset=utf-8")
public Object getUserById(@PathVariable String userId) {
```

## 4.其他

### 4.1 怎么设置转发和重定向

转发：前面加上"forward:",例如forward:user.do?name=kebi

重定向：前面加上“redirect:”例如redirect:https://www.baidu.com

### 4.2 如何解决中文乱码问题

post请求中文乱码，在web.xml中添加一个CharacterEncodingFilter过滤器设置为utf8

```java
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>

    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>	
```

get请求中文乱码，在tomcat中添加编码和工程编码一致

### 4.3 SpringMVC出现异常如何解决

抛给Spring框架解决

### 4.4 如果在拦截请求中，我想拦截get方式提交的方法,怎么配置

可以在@RequestMapping注解里面加上method=RequestMethod.GET。

使用GetMapping

### 4.5 如果想在拦截的方法里面得到从前台传入的参数,怎么得到？

答：直接在形参里面声明这个参数就可以,但必须名字和传过来的参数一样。

### 4.6 怎么在方法里面得到Request，或者Session

直接在形参里面声明这个参数就好了，Springmvc会自动把属性复制到这个独享里面

### 4.7 如果前台有很多个参数传入,并且这些参数都是一个对象的,那么怎么样快速得到这个对象？

答：直接在方法中声明这个对象,Spring MVC就自动会把属性赋值到这个对象里面。

### 4.8 SpringMVC用什么对象从后台向前台传输数据

ModelMap

### 4.9 怎么样把ModelMap里面的数据放入Session里面？

答：可以在类上面加上@SessionAttributes注解,里面包含的字符串就是要放入session里面的key。

