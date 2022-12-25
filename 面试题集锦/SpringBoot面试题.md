## 1.概述

### 1.1 什么是SpringBoot

SpringBoot是Spring下的开源组织的子项目，是Spring组件的一站式解决方案，主要是简化了使用Spring的难度，俭省了繁重的配置，提供了各种启动器，开发者可以快速上手。

### 1.2 SpringBoot的优点

- 容易上手，提升了开发效率。
- 开箱即用，原理繁琐的配置。
- 提供了一系列的大型项目的非业务性功能。
- 没有代码生成，也不需要配置XML。

### 1.3 SpringBoot的核心注解是哪一个，主要有哪几个注解组成

- 核心注解是@SpringBootApplication。
- 主要由@Configuration注解，实现文件的配置功能。
- 由EnableAutoConfiguration打开自动配置的功能，
- @ComponentScan：Spring组件扫描，组成。

## 2.配置

### 2.1 什么是 JavaConfig？

- Spring JavaConfig是Spring的社区产品，它提供了配置SpringIOC容器的纯java方法，可以避免使用xml配置的方法。
- 面向对象的配置，由于配置被定义为JavaConfig种的类，因此可以充分使用Java面向对象的功能，一个配置可以继承另一个，并且可以重写@Bean方法。
- 可以减少或者消除xml配置方法，
- 类型安全和重构友好，JavaConfig提供了一种类型安全的方法来配置Spring容器，由于Java5.0对于泛型的支持，可以按照类型而不是按照名称检索Bean，不需要任何强制转换。

### 2.2 Spring Boot 自动配置原理是什么？

我们需要@EnableAutoConfiguration @Configuration @ConditionalOnClass这三者就是自动配的核心

@EnableAutoConfiguration可以自动导入META-INF/spring.factories里面定义的自动配置类。

自动筛选可以有效的配置类

每一个自动配置类结合对应的xxxProperties.java,读取配置文件的自动配置功能。

### 2.3 Spring Boot 是否可以使用 XML 配置 ?

推荐使用Java配置而非Jxml配置，但是也可以使用xml，通过@ImportResoource注解可以导入一个XML配置。

### 2.4 SpringBoot核心配置文件是什么？bootstrap.properties和applcation.properties有什么区别

单纯做SpringBoot开发，可能不太容易遇到bootstrap.properties配置文件，但是在结合SpringCloud时，这个配置就遇到了。

bootstrap由父ApplicationContext加载的。 ，比application优先加载，配置文件在上下文引用阶段生效。

application由Application加载，用于spring boot项目的自动化配置。

### 2.5如何自定义端口运行Spring Boot应用程序

我们可以在applcation.properties中指定端口，server.port=8088



## 3.安全

### 3.1 如何实现 Spring Boot 应用程序的安全性？

为了实现Spring Boot的安全，我们可以使用spring-boot-starter-security依赖项，并且必须添加安全配置，它只需要少量的代码，但是配置类必须实现WebSecurityConfigurerAdapter并覆盖实现方法

### 3.2 什么是 CSRF 攻击？

CSRF 代表跨站请求伪造。这是一种攻击，迫使最终用户在当前通过身份验证的Web 应用程序上执行不需要的操作。CSRF 攻击专门针对状态改变请求，而不是数据窃取，因为攻击者无法查看对伪造请求的响应。

## 4.监视器

### 4.1 Spring Boot 中的监视器是什么？

Spring boot actuator 是 spring 启动框架中的重要功能之一。Spring boot 监视器可帮助您访问生产环境中正在运行的应用程序的当前状态。有几个指标必须在生产环境中进行检查和监控。即使一些外部应用程序可能正在使用这些服务来向相关人员触发警报消息。监视器模块公开了一组可直接作为 HTTP URL 访问的REST 端点来检查状态。