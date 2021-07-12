---
title: Spring AOP的配置和应用
date: 2020/4/12
description: Spring AOP的配置和应用
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/186zIDlrCPFeuXQ.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/186zIDlrCPFeuXQ.jpg'
categories:
  - Spring
tags:
  - java
  - Spring
abbrlink: 57175
---



# Spring AOP的配置和应用

## 概念

Spring 框架的一个关键组件是**面向方面的编程**(AOP)框架。面向方面的编程需要把程序逻辑分解成不同的部分称为所谓的关注点。跨一个应用程序的多个点的功能被称为**横切关注点**，这些横切关注点在概念上独立于应用程序的业务逻辑。有各种各样的常见的例子，如日志记录、审计、声明式事务、安全性和缓存等。

其底层是基于Java的动态代理的技术来实现操作的，即在不改变原方法的代码的情况下，对其方法进行方法的增强，降低方法之间的依赖，从而达到解耦的效果。

## AOP术语

- **连接点**：业务层中的所有方法。
- **切入点**：业务层中被增强的方法。
- **通知**：即开始实现被增强方法之前或者之后需要进行的操作。
  - 前置通知： 在一个方法执行之前，执行通知。
  - 后置通知：在一个方法执行之后，不考虑其结果，执行通知。
  - 异常通知：在一个方法执行之后，只有在方法退出抛出异常时，才能执行通知。
  - 最终通知： 在一个方法执行之后，**只有在方法成功完成时**，才能执行通知。
  - 环绕通知：在建议方法调用之前和之后，执行通知。
- **Target 目标**：被代理对象。
- **Proxy 代理**：代理对象。
- **Aspect 切面**：指切入点和通知的结合。

**AOP的实现：把公共代码抽取出来，制作成通知，并在配置文件中声明切入点和通知的关系即切面**，其他操作注入动态代理的实现等都是由spring框架来替我们完成。

## 代码准备

- XML配置前缀，需要导入AOP

  ~~~xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:aop="http://www.springframework.org/schema/aop"
         xmlns:tx="http://www.springframework.org/schema/tx"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
         http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd
         http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
         http://www.springframework.org/schema/tx 			http://www.springframework.org/schema/tx/spring-tx-3.0.xsd">
  </beans>
  ~~~

- 导入maven依赖，注意**需要导入解析切面表达式的包**

  ~~~xml
  <dependencies>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-context</artifactId>
              <version>5.2.5.RELEASE</version>
          </dependency>
          <dependency>
              <groupId>org.aspectj</groupId>
              <artifactId>aspectjweaver</artifactId>
              <version>1.9.5</version>
          </dependency>
  </dependencies>
  ~~~

- 创建日志类作为通知类来通过实现AOP的操作

## XML 配 AOP

1. 将通知类也交给Bean管理

2. 使用 aop：config标签开始AOP配置

3. 使用 aop：aspect标签声明配置切面

   1. id 属性：给切面提供一个唯一标识
   2. ref 属性：指定通知类Bean的 id

4. 在 aop：aspect 标签内部使用对应标签来配置通知类型

   1. aop ：
      - before                          表示配置前置通知
      - after                             表示配置后置通知
      - after-throwing              表示配置异常通知
      - after-returning              表示配置最终通知
      - around 	                     表示配置环绕通知
   2. method：指定通知类中哪个方法为前置通知

5. pointcut 属性：用于指定切入点表达式，即指出对哪一个方法进行增强

   写法：关键字 execution（表达式）
   表达式：**访问修饰符  返回值  包名.包名.包名...类名.方法名(参数列表)**

**切入点表达式**：

1. 完整表达式：**访问修饰符  返回值  包名.包名.包名...类名.方法名(参数列表)**

2. 访问修饰符可省略：**返回值  包名.包名.包名...类名.方法名(参数列表)**

3. 返回值用 * 通配符：*** 包名.包名.包名...类名.方法名(参数列表)**

4. 包名用通配符，几级包几个：***  * . *. * .类名.方法名(参数列表)**

5. 包名用 .. 表示当前包及子包：*** * ..类名.方法名(参数列表)**

6. 类名方法名通配：*** * .. * . *(参数列表)**

7. 参数列表：

   - 基本数据类型：直接填数据类型 如 int
   - 引用类型：填包名 如：java.lang.String
   - 使用通配符可以表示所有类型，但必须有参数：*** * .. * . * ( * )**
   - 使用 .. 通配，有无参数均可，有则任意类型均可：*** * .. * . *(..)**

8. 全通配 ：*** * .. * . *(..)**


也可用 aop：pointcut 单独配置一个通用的切入点表达式，然后通过其 id 写入其他标签

~~~xml
<bean id="b" class="dao.B"></bean>
<bean id="log" class="dao.log"></bean>
<aop:config>
        <aop:aspect id="logService" ref="log">
            <aop:before method="before" pointcut-ref="pt"></aop:before>
            <aop:after method="after" pointcut-ref="pt"></aop:after>
            <aop:after-throwing method="error" pointcut-ref="pt"></aop:after-throwing>
            <aop:after-returning method="last" pointcut-ref="pt"></aop:after-returning>
            <aop:pointcut id="pt" expression="execution( * *..*.* ( .. ) )" />
        </aop:aspect>
</aop:config>
~~~

### 环绕通知的配置

需要结合spring的ProceedingJoinPoint接口来**通过我们自己编写的代码块来控制执行**

xml配置

~~~xml
<bean id="b" class="dao.B"></bean>
<bean id="log" class="dao.log"></bean>
<aop:config>
        <aop:aspect id="logService" ref="log">
            <aop:around method="around" pointcut-ref="pt"></aop:around>
            <aop:pointcut id="pt" expression="execution( * *..*.* ( .. ) )" />
        </aop:aspect>
</aop:config>
~~~

代码块控制

~~~java
public Object around(ProceedingJoinPoint pjp){
        Object value = null;
        try {
            System.out.println("======================前置通知");
            Object args[] = pjp.getArgs();//获取被代理方法参数
            value = pjp.proceed(args);//执行被代理方法
            System.out.println("======================后置通知");
        } catch (Throwable throwable) {
            System.out.println("======================异常通知");
            throwable.printStackTrace();
        }finally {
            System.out.println("======================最终通知");
            return value;
        }
   }
~~~

- **通知的具体位置和具体的执行流程都是由我们自己编写代码逻辑来进行控制操作**
- **故最终通知视代码位置而定**

## 注解配AOP

xml中开启注解和扫描包路径

~~~xml
<context:component-scan base-package="dao"></context:component-scan>
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
~~~

声明切面类和切入点表达式

~~~java
@Component("log")
@Aspect//当前类为切面类
public class log {
    @Pointcut("execution( * *..*.* ( .. ) )")
    public void pt(){}
    
    @Before("pt()")
    public void before(){
        System.out.println("===============================前置通知");
    }

    @After("pt()")
    public void after(){
        System.out.println("===============================后置通知");
    }

    @AfterThrowing("pt()")
    public void error(){
        System.out.println("===============================异常通知");
    }

    @AfterReturning("pt()")
    public void last(){
        System.out.println("===============================最终通知");
    }
}
~~~

**最终通知如果方法执行失败则不执行**

环绕通知注解为@Around，其具体执行同之前谈到的方法。