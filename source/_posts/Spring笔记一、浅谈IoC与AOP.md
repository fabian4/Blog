---
title: 浅谈IoC和AOP
date: 2020/3/28
description: 浅谈IoC和AOP
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/2c3XOeVFzLdnAE1.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/2c3XOeVFzLdnAE1.jpg'
categories:
  - Spring
tags:
  - java
  - Spring
abbrlink: 17692
---



## 一、Spring简介

### 概述

- Spring 是最受欢迎的企业级 Java 应用程序开发框架，数以百万的来自世界各地的开发人员使用 Spring 框架来创建性能好、易于测试、可重用的代码。
- Spring 框架是一个开源的 Java 平台，它最初是由 Rod Johnson 编写的，并且于 2003 年 6 月首次在 Apache 2.0 许可下发布。
- Spring 是轻量级的框架，其基础版本只有 2 MB 左右的大小。
- Spring 框架的核心特性是可以用于开发任何 Java 应用程序，但是在 Java EE 平台上构建 web 应用程序是需要扩展的。 Spring 框架的目标是使 J2EE 开发变得更容易使用，通过启用基于 POJO 编程模型来促进良好的编程实践。

### Spring框架

![arch1.png](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/MTWzy6o8tx5HRfq.png)

### 优势

Spring框架带来的优势主要是对原来的web开发过程中的三层架构即**表现层，业务层，持久层**的优化。在原来的三层架构中表现层处理调用业务层处理接受的数据，业务层调用持久层层方法去查询数据库，就不可避免的**产生了类与类，方法与方法之间的高度依赖，即高耦合性**。而Spring框架的出现就很好的解决了这一问题，对原来的三层架构进行解耦。

- IoC即（Inversion of Control）**控制反转**：就很好的解决了类与类之间依赖产生的耦合性。
- AOP即（Aspect Oriented Programming）**面向切面编程**：则可以解决方法与方法之间依赖产生的耦合性。

但要注意，对于耦合性只能进行降低，完全的消除是不可能的。

## 二、Spring框架的IoC

### 几个概念

- **IoC （Inversion Of Control，控制反转）**，是spring的**核心**，贯穿始终，所谓IOC ，对于spring框架来说，就是spring来负责**控制对象的生命周期和对象间的关系**。所有的类都会在spring容器中登记，告诉spring你是个什么，你需要什么，然后spring会在系统运行到适当的时候，把你要的东西主动给你，同时也把你交给其他需要你的东西。所有的类的创建、销毁都由 spring来控制，也就是说控制对象生存周期的不再是引用它的对象，而是spring。**对于某个具体的对象而言，以前是它控制其他对象，现在是所有对象都被spring控制，所以这叫控制反转**。
- **控制反转**：就相当于，假如有 a 和 b 两个对象，在注入 IoC 之前，a 依赖于 b 那么对象 a 在初始化或者运行到某一点的时候，自己必须主动去创建对象 b 或者使用已经创建的对象 b ，无论是创建还是使用对象 b ，控制权都在自己手上 ，而注入 IOC 之后就变了，对象 a 与对象 b 之间失去了直接联系，当对象 a 运行到需要对象 b 的时候，IoC 容器会主动创建一个对象 b 注入到对象 a 需要的地方。对象 a 获得依赖对象 b 的过程，**由主动行为变为了被动行为**，控制权颠倒过来了，这就是“控制反转”这个名称的由来。
- **依赖注入**： 依赖注入让 bean 与 bean 之间以**配置文件**组织在一起，而不是以硬编码的方式耦合在一起，其实依赖注入和控制反转是**同一个概念**，不管是依赖注入，还是控制反转，都采用动态、灵活的方式来管理各种对象。

### IoC底层

其底层的实现是**基于Java的反射机制**和**工厂模式**而实现的。反射机制通俗来讲就是**根据给出的类名（字符串方式）来动态地生成对象，这种编程方式可以让对象在生成时才被决定到底是哪一种对象**，再结合工厂模式就可以再不依赖其他类的情况下“生产”对象。而**IoC容器**就是对工厂模式的深化。

### IoC容器

- **IoC 容器**是 Spring 框架的核心。容器将创建对象，把它们连接在一起，配置它们，并管理他们的整个生命周期从创建到销毁。Spring 容器使用依赖注入（DI）来管理组成一个应用程序的组件。这些对象被称为 Spring Beans。

- **IOC 容器**具有依赖注入功能的容器，它可以创建对象，IOC 容器负责实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。通常new一个实例，控制权由程序员控制，而"控制反转"是指new实例工作不由程序员来做而是交给Spring容器来做。**在Spring中BeanFactory是IOC容器的实际代表者**。

  - **Spring BeanFactory 容器** ：

    这是一个最简单的容器，这个容器接口在`org.springframework.beans.factory.BeanFactor` 中被定义。

  - **Spring ApplicationContext 容器**：

    是BeanFactory的子接口，包含了父类接口的所有功能，还增加了企业中所需要的功能，更加优秀，在`org.springframework.context.ApplicationContext interface`接口中定义。

  但一般情况下BeanFactory提高的功能也已经是够用的。 

### 配置方式

- **xml** 配置
- **注解**配置

## 三、Spring框架的AOP

### 概念

**AOP即Aspect Oriented Programming，意为面向切面编程**，通过预编译方式和运行期间动态代理实现程序功能的统一维护的一种技术。是Spring框架中的另一个重要的内容，来解决方法与方法之间的依赖从而降低程序的耦合性。

### 底层

其底层实现是用了Java的**动态代理**技术，即在不对原函数的方法代码进行改动的情况下，对原函数的方法进行一些改动操作。从而达到了方法之间的解耦操作。

## 四、Spring的Maven依赖

- **Spring核心依赖**
  - Spring-core
  - Spring-beans
  - Spring-context
- **Spring dao依赖**(提供JDBCTemplate)
  - Spring-jdbc
  - Spring-tx
- **Spring web依赖**
  - Spring-web
  - Spring-webmvc
- **Spring test依赖**
  - Spring-test