---
title: SpringBoot 日志配置
date: 2020/9/25
description: SpringBoot 日志配置
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/d9JG86X4ryqSDxp.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/d9JG86X4ryqSDxp.jpg'
categories:
  - Spring
tags:
  - SpringBoot
  - Java
  - 日志
abbrlink: 11717
---

# SpringBoot 日志配置

> 日志，通常不会在需求阶段作为一个功能单独提出来，也不会在产品方案中看到它的细节。但是，这丝毫不影响它在任何一个系统中的重要的地位。
>
> 为了保证服务的高可用，发现问题一定要即使，解决问题一定要迅速，所以生产环境一旦出现问题，预警系统就会通过邮件、短信甚至电话的方式实施多维轰炸模式，确保相关负责人不错过每一个可能的bug。
>
> 预警系统判断疑似bug大部分源于日志。比如某个微服务接口由于各种原因导致频繁调用出错，此时调用端会捕获这样的异常并打印ERROR级别的日志，当该错误日志达到一定次数出现的时候，就会触发报警。

## 一、默认日志

Spring Boot默认使用LogBack日志系统，如果不需要更改为其他日志系统如Log4j2等，则无需多余的配置，LogBack默认将日志打印到控制台上。

如果要使用LogBack，原则上是需要添加dependency依赖的。但是因为新建的Spring Boot项目一般都会引用`spring-boot-starter`或者`spring-boot-starter-web`，而这两个起步依赖中都已经包含了对于`spring-boot-starter-logging`的依赖，所以，无需额外添加依赖。

当我们需要使用到日志记录时，需要新建一个 Log 对象，才能调用该方法。

~~~java
private static final Logger log = LoggerFactory.getLogger(类名.class);

log.debug("...");
log.info("...");
log.warn("...");
log.error("...");
~~~

我们新建一个Springboot项目进行测试

~~~java
@SpringBootApplication
public class SpringbootLogApplication {

    private static final Logger log = LoggerFactory.getLogger(SpringbootLogApplication.class);

    public static void main(String[] args) {
        SpringApplication.run(SpringbootLogApplication.class, args);
        log.debug("...");
        log.info("...");
        log.warn("...");
        log.error("...");
    }

}
~~~

![image-20201125201502113](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/5m7BaXz64vRnjfT.png)

> Spring Boot默认的日志级别为INFO，这里打印的是INFO及其以上级别的日志
>
> 日志等级：**TRACE < DEBUG < INFO < WARN < ERROR < FATAL**



## 二、Lombok 的 @Slf4j 注解

引入Lombok依赖

~~~xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
~~~

此时只需要一个 **@Slf4j** 注解标注在类上，在编译时就会自动帮我们新建一个 **Log** 对象

~~~java
@Slf4j
@SpringBootApplication
public class SpringbootLogApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootLogApplication.class, args);
        log.debug("...");
        log.info("...");
        log.warn("...");
        log.error("...");
    }
}
~~~



## 三、日志文件配置

在配置文件目录下面新建一个配置文件 `logback-spring.xml`

> Spring Boot官方推荐优先使用带有-spring的文件名作为你的日志配置（如使用logback-spring.xml，而不是logback.xml），命名为logback-spring.xml的日志配置文件，spring boot可以为它添加一些spring boot特有的配置项。
> 上面是默认的命名规则，并且放在src/main/resources下面即可。如果你即想完全掌控日志配置，但又不想用logback.xml作为Logback配置的名字，可以在application.properties配置文件里面通过logging.config属性指定自定义的名字：`logging.config=classpath:logback.xml`

~~~xml
<configuration debug="false" scan="true" scanPeriod="10 seconds">
    <property name="path" value="D:/log"/>
    <property name="maxHistory" value="30"/>
    <property name="maxFileSize" value="50MB"/>
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>debug</level>
        </filter>
        <encoder>
            <pattern>%yellow(%d{yyyy-MM-dd HH:mm:ss}) %-5p [%magenta(%t)] %blue(%file:%L) %green(%c):%cyan(%msg%n)</pattern>
        </encoder>
    </appender>
    <root>
        <level value="info"/>
        <appender-ref ref="console"/>
    </root>
</configuration>
~~~

运行效果

![image-20201125205843611](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/tlLWAEgCOpZKcn9.png)