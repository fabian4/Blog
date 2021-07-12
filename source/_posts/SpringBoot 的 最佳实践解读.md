---
title: SpringBoot 的最佳实践解读
date: 2020/8/22
description: SpringBoot 的最佳实践解读
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/zHGp7behEU6qYBD.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/zHGp7behEU6qYBD.jpg'
categories:
  - Spring
tags:
  - SpringBoot
  - Java
abbrlink: 49869
---

# SpringBoot 的最佳实践解读

## 1. 使用自定义的 BOM 来维护第三方依赖

SpringBoot项目本身使用和集成了大量的开源项目，它帮助我们维护第三方依赖。但是也有一些在实际项目使用中并没有包括进来这就需要我们在项目中自己维护版本。如果在一个大型的项目中，包括了很多未开发模块，那么维护起来就非常繁琐。

事实上，Spring IO Platform 就是做这个事情，它本身就是 SpringBoot 的子项目，同时维护了其他第三方开源库。我们可以借鉴 Spring IO Platform 来编写自己的基础项目 platform-bom，所有的业务模块项目应该以BOM的方式引入。这样在升级第三方依赖时，就只需要升级这一个依赖的版本而已

~~~xml
<dependencyManagement>
   <dependencies>
       <dependency>
           <groupId>io.spring.platform</groupId>
           <artifactId>platform-bom</artifactId>
           <version>Cairo-SR3</version>
           <type>pom</type>
           <scope>import</scope>
       </dependency>
   </dependencies>
</dependencyManagement>
~~~

## 2. 使用自动配置

SpringBoot 的一个主要特征就是使用自动配置。这是SpringBoot的一部分，它可以简化你的代码并使之工作。但在类路径上检测到特定的 jar 文件时，自动配置就会被激活。

使用它最简单的方法就是依赖 Spring Boot Starters。因此，如果你想与 Redis 进行集成，你可以

~~~xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
~~~

如果你想与 MongoDB 进行集成

~~~xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
~~~

借助这些 starters，这些繁琐的配置就可以很好的集成起来并协同工作，而且它们都是经过测试和验证的。这就有助于避免可怕的 Jar 地狱。

还可以通过注解属性，从自动配置中排除某些配置类

~~~java
@EnableAutoConfiguration（exclude = {ClassNotToAutoconfigure.class}）
~~~

但只有在绝对必要时才应该这样做。

有关自动配置的官方文档：https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-auto-configuration.html

## 3. 使用 Spring Initializer 来开始一个新的 SpringBoot 项目

Spring Initializr 提供了一个超级简单的方法来创建一个新的 Spring Boot 项目，并根据你的需要来加载可能使用到的依赖。

> https://start.spring.io/

使用 Initializr 创建应用程序可确保你获得经过测试和验证的依赖项，这些依赖项适用于 Spring 自动配置。你甚至可能会发现一些新的集成，但你可能并没有意识到这些。

## 4、正确设计代码目录结构

尽管允许你有很大的自由，但是有一些基本规则值得遵守来设计你的源代码结构。

避免使用默认包。确保所有内容（包括你的入口点）都位于一个名称很好的包中，这样就可以避免与装配和组件扫描相关的意外情况

将 Application.java（应用的入口类）保留在顶级源代码目录中

将控制器和服务放在以功能为导向的模块中，但这是可选的。一些非常好的开发人员建议将所有控制器放在一起。不论怎样，坚持一种风格！

## 5、保持 @Controller 的简洁和专注

Controller 应该非常简单。你可以在此处阅读有关 GRASP 中有关控制器模式部分的说明。你希望控制器作为协调和委派的角色，而不是执行实际的业务逻辑

- 控制器应该是无状态的！默认情况下，控制器是单例，并且任何状态都可能导致大量问题

- 控制器不应该执行业务逻辑，而是依赖委托

- 控制器应该处理应用程序的 HTTP 层，这不应该传递给服务

- 控制器应该围绕用例 / 业务能力来设计

要深入这个内容，需要进一步地了解设计 REST API 的最佳实践。无论你是否想要使用 Spring Boot，都是值得学习的。

## 6、围绕业务功能构建 @Service

Service 是 Spring Boot 的另一个核心概念。我发现最好围绕业务功能 / 领域 / 用例（无论你怎么称呼都行）来构建服务。

在应用中设计名称类似`AccountService`, `UserService`, `PaymentService`这样的服务，比起像`DatabaseService`、`ValidationService`、`CalculationService`这样的会更合适一些。

你可以决定使用 Controler 和 Service 之间的一对一映射，那将是理想的情况。但这并不意味着，Service 之间不能互相调用！

## 7、使数据库独立于核心业务逻辑之外

在 SpringBoot 中最好的处理数据库交互因该是**将数据库逻辑于服务中分离出来**，因此需要一些抽象来封装对象的持久性。尤其是在当今的微服务时代

## 8、保持业务逻辑不受 SpringBoot 代码的影响

为了保护我们的业务逻辑，保持我们的业务逻辑可重用，将部分服务封装为库。

## 9、推荐使用构造函数注入

保持业务逻辑免受 SpringBoot代码侵入的一种方法是使用构造函数注入。不仅是因为 `@Autowired` 注解在构造函数上是可选的，而且还可以在没有 Spring 的情况下实例化bean

## 10、熟悉并发模型

在 SpringBoot 中的并发也是我们应该去熟悉和理解的，比如 Controller 和 Service 是默认单例的，如果你不小心就很有可能引起并发问题。所以这些问题要重视起来。

## 11、加强配置管理的外部化

你可以手动处理 Spring 应用程序的配置。如果你正在处理多个 SpringBoot 应用程序，则需要使配置管理能力更加强大。

- 使用配置服务器
- 将配置存储在环境变量中

这些都要求你这 DevOps 更少工作量，但这在微服务领域是很常见的

## 12、提供全局异常处理

你真的需要一种处理异常的一致方法。Spring Boot 提供了两种主要方法：

- 你应该使用 HandlerExceptionResolver 定义全局异常处理策略；
- 你也可以在控制器上添加 @ExceptionHandler 注解，这在某些特定场景下使用可能会很有用。

## 13、使用日志框架

你可能已经意识到这一点，但你应该使用 Logger 进行日志记录，而不是使用 System.out.println() 手动执行。这很容易在 SpringBoot 中完成，几乎没有配置。只需获取该类的记录器实例：

```
Logger logger = LoggerFactory.getLogger(MyClass.class);
```

这很重要，因为它可以让你根据需要设置不同的日志记录级别。

## 14、测试你的代码

这不是 SpringBoot 特有的，但它需要提醒——测试你的代码！如果你没有编写测试，那么你将从一开始就编写遗留代码。

如果有其他人使用你的代码库，那边改变任何东西将会变得危险。当你有多个服务相互依赖时，这甚至可能更具风险。

由于存在 SpringBoot 最佳实践，因此你应该考虑将 Spring Cloud Contract 用于你的消费者驱动契约，它将使你与其他服务的集成更容易使用。

## 15、测试你的代码

这不是 Spring Boot 特有的，但它需要提醒——测试你的代码！如果你没有编写测试，那么你将从一开始就编写遗留代码。

如果有其他人使用你的代码库，那边改变任何东西将会变得危险。当你有多个服务相互依赖时，这甚至可能更具风险。

由于存在 Spring Boot 最佳实践，因此你应该考虑将 Spring Cloud Contract 用于你的消费者驱动契约，它将使你与其他服务的集成更容易使用。

## 16、使用测试切片让测试更容易，并且更专注

使用 Spring Boot 测试代码可能很棘手——你需要初始化数据层，连接大量服务，模拟事物…… 实际上并不是那么难！答案是使用测试切片。

使用测试切片，你可以根据需要仅连接部分应用程序。这可以为你节省大量时间，并确保你的测试不会与未使用的内容相关联。