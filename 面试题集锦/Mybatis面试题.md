## 1.简介

### 1.1 Mybatis是什么

mybatis是一款优秀的持久层框架，一个半ORM框架，他支持定制化SQL，存过过程及高级映射。Mybatis避免了所有的JDBC代码和手动设置参数以及获得结果集。Mybatis可以使用xml注入或注解来配置和映射原生类型、接口和javapojo。

MyBatis 是一个小巧、方便、高效、简单、直接、半自动化的持久层框架，

### 1.2 ORM是什么

ORM（Object Relational Mapping）对象关系映射，是为了解决关系型数据库和pojo的映射关系的技术。

### 1.3 为什么说Mybatis是一款半自动ORM映射工具？他与全自动区别在哪里

他在查询关联对象或者关联集合时，需要手动写sql来完成。

### 1.4 传统的JDBC开发存在的问题MyBatis是如何解决这些问题的？

- 频繁的创建数据库连接，释放，容易造成数据资源的浪费，影响性能。我们使用连接池解决这个问题
  - 配置数据库连接池 c3p0 druid Hikaricp
- sql语句的创建，参数设置，结果集的硬编码。实际项目中sql语句变化的可能性较大，一旦发生变化，就要修改java代码，系统需要重新编译，维护。
  - 将XXXmapper.xml文件中与java代码分离
- 使用preparedStatement向占位符传参数存在硬编码。因为sql语句的where条件不一定，可能多也可能少，修改sql，还要修改代码
  - Mybaits自动将java对象映射到sql语句
- 结果集存在重复代码，处理麻烦，可以修复成pojo映射。
  - Mybatis自动将结果集映射到java对象

### 1.5 Mybatis的优点

- 基于SQL语句编程，相当灵活，不会对应用程序或者数据库现有设计造成任何影响，SQL写在xml里面，解除sql程序和程序代码的耦合，便于统一管理，支持XML标签，支持编写动态sql语句，并且可以重用。
- 与传统JDBC相比，代码量减少了50%以上
- 很好的与各种数据库相兼容
- 能够很好的与Spring集成
- 提供映射标签，支持对象与数据库的ORM字段关系映射。

## 2.Mybatis的解析和运行原理

### 2.1 Mybatis编程步骤是什么样子

- 创建SqlSessionFactory
- 通过SqlSessionFactory创建SqlSession
- 通过SqlSession执行数据库操作
- 调用session.commit()提交数据
- 调用session.close()关闭数据库

### 2.2 请说说Mybatis的工作原理

![image-20210816203305957](https://kangrui-pictures.oss-cn-beijing.aliyuncs.com/img/image-20210816203305957.png)

1. 读取Mybatis配置文件：mybatis-config.xml为Mybatis的全局配置文件，配置了Mybatis的运行环境等信息，例如数据库连接信息
2. 加载映射文件，即XXXmapper.xml映射文件。该文件中配置了操作数据库的SQL语句。需要在mybatis-config.xml文件中加载多个映射文件，每个文件对应一张数据库表。
3. 构造会话工厂，通过Mybatis的全局配置文件构建会话工厂sqlSessionFactory
4. 创建会话对象，由会话工厂创建sqlSession，这个对象包含了执行SQL语句的所有方法
5. Executor执行器，Mybatis底层创建了Executor接口操作数据库，他们根据SqlSession传递的参数生成需要执行的sql语句，同时负责查询缓存的维护。
6. MappedStatement对象，在Executor接口的执行方法中有一个MappedStatement类型参数，该参数是对映射信息的封装，用于存储映射的sql语句的id，参数等信息。
7. 输入参数映射，输入的参数可能时list，Map等集合类型，也可能时基本数据类型和POJO对象，输入的过程类似于preparedStatement对象设置参数的过程。
8. 输出参数映射，输出的参数可能时list，Map等集合类型，也可能时基本数据类型和POJO对象，输出结果映射过程类似于JDBC对结果集解析过程。

### 2.3 为什么需要预编译

定义：SQL预编译指的是数据库驱动在发送SQL语句和参数给DBMS之前对sql语句进行编译，这样当DBMS在执行sql语句的狮虎就不需要重新编译了。

**为什么需要预编译**

JDBC中使用对象PreparedStatement来抽象预编译语句，使用预编译可以优化SQL语句。预编译之后的SQL语句大多数可以直接执行，不需要在DNMS中再次编译。

同时预编译对象可以重复利用，把一个preparedStatemt预编译的语句缓存下来，等下次对于同一个SQL语句，可以直接使用。

### 2.4 Mybatis是否支持延时加载，其实现原理是什么

Mybatis仅仅支持assocation关联对象和collection关联集合对象的延迟加载，lazyLoadingEnable=true

## 3.映射器

### 3.1 #{}${}的区别

- 前者是占位符，支持预编译处理；后者是拼接符，字符串替换，没有预编译处理
- Mybatis在处理#{}时，#{}传入参数是以字符串传入，会将SQL语句中的#{}替换成？，调用preparedStatement的set方法。
- Mybatis在处理后者时，${}是原值传入，就是把后组合替换成变量的值，相当于JDBC的statement编译
- 替换变量后。#{}对应的变量会加上单引号，而${}不会加上单引号
- {}可以有效防止SQL注入，${}后者不行
- {}替换变量是在DBMS中，${}是在DBMS外

### 3.2 模糊查询的like语句怎么写

"%"#{question}"%"

使用bind标签

```java
<select id="listUserLikeUsername" resultType="com.jourwon.pojo.User">
　　<bind name="pattern" value="'%' + username + '%'" />
　　select id,sex,age,username,password from person where username LIKE #{pattern}
</select>

```

### 3.3 在mapper中如何传递多个参数

- 顺序传参法

\#{}里面的数字代表传入参数的顺序。

这种方法不建议使用，sql层表达不直观，且一旦顺序调整容易出错。

```java
public User selectUser(String name, int deptId);

<select id="selectUser" resultMap="UserResultMap">
    select * from user
    where user_name = #{0} and dept_id = #{1}
</select>
```

- 使用@Param注解传参

\#{}里面的名称对应的是注解@Param括号里面修饰的名称。

这种方法在参数不多的情况还是比较直观的，推荐使用。

```java
public User selectUser(@Param("userName") String name, int @Param("deptId") deptId);

<select id="selectUser" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

- 使用集合

\#{}里面的名称对应的是Map里面的key名称。

这种方法适合传递多个参数，且参数易变能灵活传递的情况。

```java
public User selectUser(Map<String, Object> params);

<select id="selectUser" parameterType="java.util.Map" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

- 使用JavaBean对象

\#{}里面的名称对应的是User类里面的成员属性。

这种方法直观，需要建一个实体类，扩展不容易，需要加属性，但代码可读性强，业务逻辑处理方便，推荐使用。

```java
public User selectUser(User user);

<select id="selectUser" parameterType="com.jourwon.pojo.User" resultMap="UserResultMap">
    select * from user
    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

### 3.4 如何获取生成的主键

通过getUserId

### 3.5 当实体类中的属性名和表中的字段名不一样 ，怎么办

1.在SQL语句中定义字段别名，额让字段别名和实体类属性一致

2.通过resultMap中的映射

```java
<select id="getOrder" parameterType="int" resultMap="orderResultMap">
	select * from orders where order_id=#{id}
</select>

<resultMap type="com.jourwon.pojo.Order" id="orderResultMap">
    <!–用id属性来映射主键字段–>
    <id property="id" column="order_id">

    <!–用result属性来映射非主键字段，property为实体类属性名，column为数据库表中的属性–>
    <result property ="orderno" column ="order_no"/>
    <result property="price" column="order_price" />
</reslutMap>
```

### 3.6 MyBatis实现一对一，一对多有几种方式，怎么操作的？

有联合查询和嵌套查询。联合查询是几个表联合查询，只查询一次，通过在resultMap里面的association，collection节点配置一对一，一对多的类就可以完成

嵌套查询是先查一个表，根据这个表里面的结果的外键id，去再另外一个表里面查询数据，也是通过配置association，collection，但另外一个表的查询通过select节点配置。

### 3.7 Mybatis的一级、二级缓存

1）一级缓存: 基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，默认打开一级缓存。

2）二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现Serializable序列化接口(可用来保存对象的状态),可在它的映射文件中配置<cache/> ；

3）对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存Namespaces)的进行了C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。

