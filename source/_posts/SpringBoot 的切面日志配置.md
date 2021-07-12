---
title: SpringBoot 的切面日志配置
date: 2020/9/29
description: SpringBoot 的切面日志配置
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/oQcqPG4DpUfrV86.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/oQcqPG4DpUfrV86.jpg'
categories:
  - Spring
tags:
  - SpringBoot
  - Java
  - 日志
abbrlink: 24594
---

# SpringBoot 的切面日志配置

> Spring 框架的一个关键组件是**面向方面的编程**(AOP)框架。面向方面的编程需要把程序逻辑分解成不同的部分称为所谓的关注点。跨一个应用程序的多个点的功能被称为**横切关注点**，这些横切关注点在概念上独立于应用程序的业务逻辑。有各种各样的常见的例子，如日志记录、审计、声明式事务、安全性和缓存等。
>
> 其底层是基于Java的动态代理的技术来实现操作的，即在不改变原方法的代码的情况下，对其方法进行方法的增强，降低方法之间的依赖，从而达到解耦的效果。

**这里我们利用 Spring 的 AOP 特性来实现日志的记录**

## 一、新建项目和测试接口

新建一个SpringBoot项目，再添加一个测试接口

~~~java
package me.fabian4.springbootlog;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class Controller {

    @GetMapping()
    public String test(String text){
        return "msg：" + text;
    }
}
~~~

## 二、自定义注解 @Log

```java
package me.fabian4.springbootlog;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {
    String value() default "";
}
```

将注解添加到方法上面

~~~java
@Log("测试接口")
@GetMapping()
public String test(String text){
    return "msg：" + text;
}
~~~

## 三、配置切面

### 1. 导入切面解析依赖

~~~xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.5</version>
</dependency>
~~~

### 2. 配置切面

~~~java
package me.fabian4.springbootlog;

import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;

import java.util.Arrays;

@Slf4j
@Aspect
@Component
public class LogAspect {

    /**
     * 配置切入点
     */
    @Pointcut("@annotation(me.fabian4.springbootlog.Log)")
    public void logPointcut() {
        // 该方法无方法体,主要为了让同类中其他方法使用此切入点
    }

    /**
     * 配置环绕通知,使用在方法logPointcut()上注册的切入点
     *
     * @param joinPoint join point for advice
     */
    @Around("logPointcut()")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
        Object result;
        result = joinPoint.proceed();
        log.info("方法参数：{}", Arrays.toString(joinPoint.getArgs()));
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        me.fabian4.springbootlog.Log aopLog = signature.getMethod().getAnnotation(me.fabian4.springbootlog.Log.class);
        log.info("注解内容：{}", aopLog.value());
        log.info("方法名称：{}.{}()", joinPoint.getTarget().getClass().getName(), signature.getName());
        log.info("方法返回：{}", result.toString());
        return result;
    }

    /**
     * 配置异常通知
     *
     * @param joinPoint join point for advice
     * @param e exception
     */
    @AfterThrowing(pointcut = "logPointcut()", throwing = "e")
    public void logAfterThrowing(JoinPoint joinPoint, Throwable e) {
        // 方法抛出异常时操作
    }
}
~~~

## 四、测试结果

我们访问一下接口来测试结果

![image-20201129151010878](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/UxhpqRgEKckjQil.png)

## 五、扩展

1. 定义一个 **Log** 类来方便日志的数据传输和数据库存储
2. 可以通过一些方法从**请求头**中获取来访者 ip 等信息并存储到数据中
3. 可以从认证信息中获取到来访者的标识等信息