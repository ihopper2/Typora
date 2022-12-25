## 1.Spring概述

### 1.1 什么是Spring

Spring是一个轻量级框架，是为了解决企业级应用开发的业务逻辑层和其他层的解耦问题。Spring负责基础架构支持，因此Java开发者可以专注于应用程序的开发。

为了降低Java程序的复杂性，Spring采取了一下四种关键策略。

- 基于POJO的轻量级和最小侵入性编程。
- 通过依赖注入和面向接口实现解耦合。
- 基于切面进行扩展式编程。
- 通过切面和模板减少代码量。

### 1.2 Spring框架的审计目标、设计理念和核心是什么？

Spring**设计目标：**Spring为开发者提供了一个一站式轻量级开发平台。

**Spring设计理念：**在JavaEE开发中，支持POJO和JavaBean开发方式，使用面向接口的编程，充分支持面向对象的设计方法；Spring通过IOC容器实现解耦合关系管理，并实现依赖反转，将对象之间的依赖关系交给Ioc容器。

Spring**框架的核心：**IOC和AOP模块。通过IOC容器管理POJO对象集合他们直接的耦合关系；通过AOP实现非入侵的方式增强服务。

### 1.3 Spring框架的优缺点

优点：

- 通过IOC实现控制反转，将对象的创建交给IOC，方便解耦，简化开发。
- AOP的切面编程，可以实现对于方便实现对于程序的权限拦截，运行监控功能。
- 声明式事务支持。
- 方便程序的测试。
- 集成了各种优秀的框架。
- 降低了JavaEE的使用难度。

缺点：

- Spring明明是一个很轻量级的框架，却给人大而全的感觉。
- Spring依赖反射，反射影响性能。
- 使用门槛提高了。

### 1.4 Spring框架种使用了哪些设计模式

- 工厂模式：BeanFactory就是简单的工厂模式，用于创建对象实例
- 单例模式：Bean默认为单例模式
- 代理模式：Spring的AOP功能就是用到了JDK的动态代理和CGLIB字节码生成技术。
- 模板模式：用来解决代码重复的问题
- 观察者模式：定义对象键一种一对多的依赖关系。

### 1.5 详细讲解以下核心容器（Spring context应用上下文）模块

这是基本的Spring模块，提供Spring框架的基本功能，BeanFactory是任何以Spring为基础的核心，Spring框架建立在此模块之上，他使Spring成为一个容器。

### 1.6 Spring有哪些模块组成

Spring总共有20个模块，1300多个文件构成，而这些组件被分别整合在**核心容器（Core Container）、AOP、设备支持、数据访问与集成、web、消息、test**等六个模块中，

![image-20210815104749627](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20210815104749627.png)

- spring core：提供了框架的基本组成部分，包括IOC和依赖注入功能。
- spring beans：提供了BeabFactory，是工厂模式的一个经典实现，Spring将管理对象称为Bean
- spring context：构建于core封装基础上的context封装包，提供了一种框架式的对象访问方法。
- spring jdbc：提供了一个JDBC的抽象层，用于简化jdbc
- spring aop：提供了面向切面编程，可以自定义拦截器，切点。

## 2.Spring控制反转IOC

### 2.1 什么是SpringIOC容器

控制反转IOC，他是传统上由程序代码直接操控对象的调用权力，交给了容器，通过容器来实现对象组件的装配和管理。SpringIOC负责创建对象，管理对象。

### 2.2 控制反转有什么作用

- 管理对象的创建和依赖关系的维护。
- 解耦，由容器去维护具体的对象

### 2.3 IOC的优点

- IOC或依赖注入把应用的代码量降到了最低。
- 他使应用容易测试，单元测试不再需要使用单例。
- 最小的代价和最小的侵入性使松散耦合得以实现。
- IOC容器支持加载服务时的饿汉式初始化和懒加载。

### 2.4 SpringIOC的实现机制

就是工厂模式+反射机制

```java
interface Fruit {
   public abstract void eat();
 }

class Apple implements Fruit {
    public void eat(){
        System.out.println("Apple");
    }
}

class Orange implements Fruit {
    public void eat(){
        System.out.println("Orange");
    }
}

class Factory {
    public static Fruit getInstance(String ClassName) {
        Fruit f=null;
        try {
            f=(Fruit)Class.forName(ClassName).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
}

class Client {
    public static void main(String[] a) {
        Fruit f=Factory.getInstance("io.github.dunwu.spring.Apple");
        if(f!=null){
            f.eat();
        }
    }
}
```

### 2.5 SpringIOC支持哪些功能

- 依赖注入
- 依赖检查
- 自动装配
- 支持集合
- 指定初始化方法和销毁方法
- 支持某些回调方法

### 2.6 BeanFactory和ApplicationContext有什么区别

这是Spring的两大核心接口，都可以当作Spring容器。其中ApplicationContext是BeanFactory的子接口。

依赖关系：

BeanFactory是Spring里面最底层的接口，包含了各种Bean定义，读取Bean配置文档，管理Bean的加载，实例化，控制Bean的生命周期，维护Bean之间的依赖关系。

ApplicationContext作为BeanFactory的派生，除了提供BeanFactory所具有的功能，还提供了完整的框架功能。

- 继承了完整的MessageSource，因此支持国际化
- 统一了资源文件的访问方式
- 提供了在监听器中注册Bean的方式
- 同时加载多个配置文件

加载方式：

- BeanFactory采用延迟加载的方式注入Bean，即只有在使用某个Bean时，才会对该Bean进行加载实例化。这样我们就不能发现一些存在的Spring配置问题了。如果某个Bean某个属性没有注入，BeanFacctory加载后，直到第一次调用getBean方法才会抛出异常
- ApplicationContext，他是在容器启动的时候，一次性创建所有的Bean，这样在容器启动的时候，我们就可以发现Spring中存在的配置错误。这样有利于检查依赖属性是否注入。

创建方式

- BeanFactory通常以编程的方式被创建，

- ApplicationContext还能以声明的方式创建，如使用ContextLoader。

注册方式

BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，但两者之间的区别是：BeanFactory需要手动注册，而ApplicationContext则是自动注册。

### 2.7 Spring是如何设计容器的，BeanFactory和ApplicatonContext的关系详解

- ApplicationContext是BeanFactory的子接口。
- BeanFactory简单粗暴，可以理解为就是一个HashMap，key是BeanName，Value是Bean实例，通常只支持注册（put），获取（get）这两个功能，我们称之为低级容器
- ApplicationContext我们称之为高级容器，继承了BeanFacotry还有一些其他接口，因此具有更多的功能。例如资源的获取和支持多种消息。

### 2.8 ApplicatonContext通常的实现是什么

- FileSystemApplicationContext：此容器从一个XML文件中加载Bean，XML Bean的配置文件的全路径名必须提供给他作为构造函数
- ClassPathXmlApplicationContext：此容器也是从一个XML里面加载Bean，这里只需要配置classPath就行了
- WebXmlApplicationContext：此容器架子啊一个XML文件，此文件定义了一个WEB应用的所有Bean

### 2.9 什么是Spring注入依赖

- 控制反转IOC是一个很大的概念，可以用不同的方式来实现，主要有两种实行方式：依赖注入和依赖查找
- 依赖注入：就是组件之间的依赖关系由容器在应用系统运行期来决定的。也就是由容器动态地将某种依赖关系的目标对象实例注入到应用系统的各个关联的组件之中。

**依赖注入的基本原则**

应用组件不应该负责查找资源或者其他依赖的协作对象。配置对象的工作应该由IOC容器负责，“查找资源”的逻辑应该从组件代码中抽取出来，交给IOC容器负责，容器全权负责组件的装配，他会把依赖关系的对象通过属性注入或者构造器传递给需要的对象

**依赖注入的优势**

容器全权负责依赖查询，受管组件子需要暴露JavaBean的setter方法或者带参数的构造器或者接口，使容器可以在初始化的时组装对象的依赖关系。

### 2.10 有哪些注入方式

接口注入（弃用了）

set注入

构造器注入

## 3.Spring面向切面编程（AOP）

### 3.1 什么是AOP

OOP是面向对象编程，允许开发者定义纵向的关系，但是不适合定义横向的关系，导致了大量代码的重复，而不利于各个模块的重用

AOP是面向切面编程，作为面向对象的补充，用于将哪些与业务无关的，但是却与多个对象产生影响的公共逻辑和行为，曾抽取并封装为一个可以重用的模块，这个模块被命名为切面，减少系统中重复的代码，减低了模块之间的耦合度，同时提高了系统的可维护性。用于权限的认证、日志、事务处理。

### 3.1 SpringAop的实现

AOP的实现关键在代理模式，AOP的实现主要分为静态代理模式和动态代理模式，静态代理的代表为AspectJ，动态代理则以SpringAOP为代表。

AspectJ是静态代理模式，所谓的静态代理模式，就是AOP框架会在编译阶段生成AOP代理类，因此也成为编译增强，他会在编译阶段将ApectJ（切面）织入到Java字节码中，运行时就是增强之后的AOP对象。

SpringAOP使用的是动态代理实现，所谓的动态代理就是说AOP框架不会修改代码，而是每次运行在内存中临时方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调元对象的方法。

### 3.3 JDK动态代理和CGLB动态代理

JDK动态代理只提供接口的代理，不支持类的代理。核心InvocationHandler接口和Proxy类，InvocationHandler 通过invoke()方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起；接着，Proxy利用 InvocationHandler动态创建一个符合某一接口的的实例, 生成目标类的代理对象。

如果代理类没有实现 InvocationHandler 接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

### 3.4 SpringAOP里面的几个名词

- 通知（advice）：在切面的某个特定的连接点执行的动作，通知类型由before around after aroud afterthrowing等通知，它属于横向关注点的具体实现者。
- 切人点（pointCut）：切点的定义会匹配所要织入的一个或者多个连接点，我们通常使用明确的类或者方法名称，或是利用正则表达式所匹配的类和名称来指定这些切入点。
- 切面（Aspect）：切面是通知和切点的结合。通知和切点构成了切面的全部内容。在SpringAOP中我们使用通用类或者在普通类中以@AspectJ注解实现
- 连接点（joinPoint）：目标对象，所有可以增强的方法

Aspect(切面):切入点+通知。

joinPoint(连接点):目标对象,所有可以增强的方法。

Advice(通知/增强):增强代码。

PointCut(切入点):目标对象,将要和已经增强的方法。

Introduction(引入):声明某个方法或字段。

Target(目标对象):被代理的对象

AOP 代理(AOp Proxy) AOP框架创建的对象用来实现切面。

Weaving(织入)：将通知应用到切入点的过程。

```java
package com.cj.study.spring.aop;
 
public interface PersonService {
	
	public String savePerson();
	
	public void updatePerson();
	
	public void deletePerson();
	
}
package com.cj.study.spring.aop;
 
public class PersonServiceImpl implements PersonService{
 
	@Override
	public String savePerson() {
		System.out.println("添加");
		return "保存成功！";
	}
 
	@Override
	public void updatePerson() {
		System.out.println("修改");
	}
 
	@Override
	public void deletePerson() {
		System.out.println("删除");
	}
}
package com.cj.study.spring.aop;
 
//切面类
public class MyTransaction {
	//切面里的通知方法
	public void beginTransaction(){
		System.out.println("开启事务 ");
	}
	//切面里的通知方法
	public void commit(){
		System.out.println("提交事务");
	}
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:aop="http://www.springframework.org/schema/aop" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">
 
	<!-- 导入目标类 -->   
	<bean id="personService" class="com.cj.study.spring.aop.PersonServiceImpl"></bean>
	<!-- 导入切面 -->
	<bean id="myTransaction" class="com.cj.study.spring.aop.MyTransaction"></bean>
	
	<aop:config>
        //切入点
		<aop:pointcut expression="execution(* com.cj.study.spring.aop.*.*(..))" id="perform"/>
        //切面类
		<aop:aspect ref="myTransaction">
            //通知
			<aop:before method="beginTransaction" pointcut-ref="perform"/>
		</aop:aspect>
		<aop:aspect ref="myTransaction">
			<aop:after method="commit" pointcut-ref="perform"/>
		</aop:aspect>
	</aop:config>
	
</beans>
```



### 3.5 Spring只支持方法级别的连接点

因为Spring只支持方法级别的连接点，缺少对于字段连接点的支持。

### 3.6 什么是切面

aspect由pointCut和advice组成，切面是通知和切点的结合。他既包含了横切的逻辑定义，也包含了连接点的定义。

## 4.Spring注解

### 4.1 什么是基于Java的Spring注解配置？

基于Java的配置？允许你子啊少量Java注解的帮助下，进行大部分Spring配置而非通过XML配置

以@Configuration注解为例子，它可以用来标记类当作一个bean的定义，被当作SpringIOC容器使用

另一个注解是@bEAN注解，他表示此方法将要返回一个对象，作为一个bean注册进Spring应用上下文

### 4.2 如何开启注解装佩

在xml文件中，使用标签<context:annotion-config>

### 4.3 @Component, @Controller, @Repository, @Service 有何区别？

他们都是用于编辑类作为bean，装入SpringIOC容器的，知识在不同的层，我们使用不同的标签作为分隔

@Controller 用在控制层

@Service：用在服务层

@Repository用在dao层

### 4.4 @Required 注解的作用

这个注解用在bean属性的**setter方法上**，它表明受影响的bean属性在配置的时候必须放在xml配置文件中，否则就会抛出异常

### 4.5 @Autowired注解作用

默认是按照类型装配注入，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）。@Autowired 注解提供了更细粒度的控制，包括在何处以及如何完成自动装配。它的用法和@Required一样，修饰setter方法、构造器、属性或者具有任意名称和/或多个参数的PN方法。

### 4.6 @Qualifier 注解有什么作用

当您创建多个相同类型的 bean 并希望仅使用属性装配其中一个 bean 时，您可以使用@Qualifier 注解和 @Autowired 通过指定应该装配哪个确切的 bean 来消除歧义。

### 4.7 @Resource

默认按照名称注入，相当于是@Autowired+@Qualifier

@Autowired//默认按type注入
@Qualifier("cusInfoService")//一般作为@Autowired()的修饰用
@Resource(name="cusInfoService")//默认按name注入，可**以通过name和type属性进行选择性注入**

### 4.8 @RequestMapping 注解有什么用？

@RequestMapping 注解用于将特定 HTTP 请求方法映射到将处理相应请求的控制器中的特定类/方法。此注释可应用于两个级别：

- 类级别：映射请求的 URL
- 方法级别：映射 URL 以及 HTTP 请求方法
### 4.9 @ResponseBody

@ResponseBody的作用其实是将java对象转为json格式的数据

