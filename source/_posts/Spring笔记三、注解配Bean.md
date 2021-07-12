---
title: Spring Bean配置——注解配Bean
date: 2020/4/5
description: 注解配Bean
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/UuKkAcjq6rZzs8v.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/UuKkAcjq6rZzs8v.jpg'
categories:
  - Spring
tags:
  - java
  - Spring
abbrlink: 60824
---



# Spring IoC中的Spring Bean配置——注解配Bean

在这把我们来解释Bean的另一种配置方式：注解配置

注解的分类：

- 用于创建对象：类似于bean标签
- 用于注入数据：类似于property标签
- 用于作用范围：类似于scope属性
- 用于生命周期：类似于 init-method 和 destory-method属性

**注意要在xml中首先配置初始化时包扫描路径才能使用注解**

~~~xml
<context:component-scan base-package="java"></context:component-scan>
~~~



## 一、用于创建对象

- **@Component**
  - 写在类上面
  - 参数为 id ，若不写则默认 id 为类名小写
- **@Controller 一般用于表现层**
- **@Service      一般用于业务层**
- **@Repository 一般用于持久层**

 ## 二、用于注入数据

- **@Autowirde**：

  - 自动按照类型注入，只有容器中有一个唯一的bean对象类型和要注入的变量类型匹配，就可以注入成功
  - 可以是成员变量上和方法上

- **@Qualifier**：

  - 在按照类注入的基础上再按照名称注入
  - 给成员变量注入是**不能单独使用**，和@Autowirde配合使用
  - 可以配置在方法参数中
  
- **@Resource**：实现上述两者功能

**上述三个注解只能注入bean类型数据，基本数据类型和String无法实现**

**集合的注入只能通过 xml 来实现**

- **@Value**：用于注入基本数据类型和String
  - 可以使用 spEl 即 spring的 El 表达式 ${表达式}
  - value属性用于指定数据的值

## 三、用于作用范围

- **@Scope**：用于指定作用范围

  value用于指定范围

  - singleton	单例（默认）
  - prototype	多例

## 四、用于生命周期

- **@PreDestroy**：销毁方法
- **@PostConstruct**：初始化方法

## 五、其他注解

- **@Configuration**：指定当前类为配置类，可以实现在类中配置，脱离 xml。

  只有当对象为AnnotationConfigApplicationContext创建参数时可以不写

- **@ComponentScan**：指定spring在创建容器时要扫描的包。

- **@Bean**：将当前方法的返回值存入容器，属性为 id，不写时，默认为当前方法名称。

  当用注解配置的方法含参数时，spring会自动去容器中自动找对象

- **@Import**：用于导入其他配置类，参数为类的字节码文件，写在父配置类里

- **@PropertySource**：导入Property配置文件，关键字为classPath：后跟类路径。

## 六、容器构造方法补充

**`AnnotationConfigApplicationContext`**：读取注解创建容器
当使用注解类注解时，工程中不再存在xml 文件，所以也就不能使用其他两种构造方法了

~~~java
ApplicationContext ac = new AnnotationConfigApplicationContext(注解类.class);
~~~

