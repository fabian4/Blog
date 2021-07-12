---
title: Spring Security 实现简单用户登录
date: 2020/6/18
description: Spring Security 实现简单用户登录
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/TYAkIDdjqw243eO.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/TYAkIDdjqw243eO.jpg'
categories:
  - Spring
tags:
  - SpringSecurity
abbrlink: 61308
---

# Spring Security 实现简单用户登录

## 一、简介

### 1. 框架介绍

一个能够为基于Spring的企业应用系统提供声明式的安全访问控制解决方式的安全框架（简单说是对访问权限进行控制）

应用的安全性包括用户认证（Authentication）和用户授权（Authorization）两个部分。

- 用户认证指的是验证某个用户是否为系统中的合法主体，也就是说用户能否访问该系统。用户认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认证过程。

- 用户授权指的是验证某个用户是否有权限执行某个操作。在一个系统中，不同用户所具有的权限是不同的。比如对一个文件来说，有的用户只能进行读取，而有的用户可以进行修改。一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列的权限。   

spring security的主要核心功能为 **认证和授权**，所有的架构也是基于这两个核心功能去实现的。

### 2. 框架原理

众所周知 想要对对Web资源进行保护，最好的办法莫过于Filter，要想对方法调用进行保护，最好的办法莫过于AOP。所以springSecurity在我们进行用户认证以及授予权限的时候，通过各种各样的拦截器来控制权限的访问，从而实现安全。

如下为其主要过滤器 

1. WebAsyncManagerIntegrationFilter
2. SecurityContextPersistenceFilter 
3. HeaderWriterFilter 
4. CorsFilter 
5. LogoutFilter
6. RequestCacheAwareFilter
7. SecurityContextHolderAwareRequestFilter
8. AnonymousAuthenticationFilter
9. SessionManagementFilter
10. ExceptionTranslationFilter
11. FilterSecurityInterceptor
12. UsernamePasswordAuthenticationFilter
13. BasicAuthenticationFilter

### 3. 框架的核心组件

- `SecurityContextHolder`：提供对SecurityContext的访问
- `SecurityContext`：持有Authentication对象和其他可能需要的信息
-  `AuthenticationManager `：其中可以包含多个AuthenticationProvide
- `ProviderManager`：对象为AuthenticationManager接口的实现类
- `AuthenticationProvider `：主要用来进行认证操作的类 调用其中的authenticate()方法去进行认证操作
- ` Authentication`：Spring Security方式的认证主体
- `GrantedAuthority`：对认证主题的应用层面的授权，含当前用户的权限信息，通常使用角色表示
- `UserDetails`：构建Authentication对象必须的信息，可以自定义，可能需要访问DB得到
- `UserDetailsService`：通过username构建UserDetails对象，通过loadUserByUsername根据userName获取UserDetail对象 （可以在这里基于自身业务进行自定义的实现  如通过数据库，xml,缓存获取等）           

## 二、项目搭建

这里我们直接用 IDEA 来为我们自动创建一个 SpringBoot 工程

![](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/pimVy6AjDWUZM7u.png)

配置一个简单的映射之后我们启动项目

~~~java
package me.fabian4.securitydemo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/")
    public String hello() {
        return "hello";
    }
}
~~~

访问`http://localhost:8080/`却发现

![](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/FEX7MS9hPQUJzr4.png)

很明显我们导入的 SpringSecurity 起了作用，但由于 SpringBoot的一贯原则 **约定大于配置** ，在我们没有对其配置的情况下，为我们采用了默认的配置，这里的用户名默认是 `user`，密码在启动时已经打印在控制台了

![](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/4Bnb1AQcwWXoiOM.png)

在输入完账号密码之后我们就能正常访问我们的页面了。

## 三、账号密码配置

### 1. 配置文件

~~~properties
spring.security.user.name=admin
spring.security.user.password=admin
~~~

### 2. 内存配置

通过编写配置类，继承`WebSecurityConfigurerAdapter`来实现

~~~java
package me.fabian4.securitydemo.controller;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {


    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
                .withUser("admin").password(new BCryptPasswordEncoder().encode("admin")).roles("admin");
    }
}
~~~

## 四、通过数据库配置账号密码

### 1. 修改配置类

~~~java
package me.fabian4.securitydemo.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {


    @Autowired
    private CustomUserDetailsService userDatailService;

    /**
     * 指定加密方式
     */
    @Bean
    public PasswordEncoder passwordEncoder(){
        // 使用BCrypt加密密码
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
                // 从数据库读取的用户进行身份认证
                .userDetailsService(userDatailService)
                .passwordEncoder(passwordEncoder());
    }
}
~~~

### 2. 准备一个基本用户信息类

~~~java
package me.fabian4.securitydemo.controller;

import org.springframework.stereotype.Repository;

@Repository
public class UserInfo {

    private String username;

    private String password;

    private String role;
	
    // 忽略 get和set方法
}

~~~

### 3. 重写一个 Service 方法

~~~java
package me.fabian4.securitydemo.controller;

import org.springframework.stereotype.Service;

@Service
public class UserInfoService {

    public UserInfo getUserInfo(String username){
        // 这里模拟数据库操作
        if (username.equals("admin")){
            UserInfo userInfo = new UserInfo();
            userInfo.setUsername("admin");
            userInfo.setPassword("admin");
            userInfo.setRole("admin");
            return userInfo;
        }
        return null;
    }
}
~~~

然后我们启动项目就达到我们的效果了

## 五、认证流程

![image-20200619140910864](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/tqICSd7EmJLFOue.png)

## 六、核心过滤器

### 1. SecurityContextPersistenceFilter

- 以前是HttpSesstionContextIntegrationFilter,位于过滤器的顶端，是第一个起作用的过滤器
- 第一个用途：在执行其他过滤器之前，率先判断用户的session是否已经存在了一个spring security上下文的securityContext,如果存在，就把securityContext拿出来，放在securityContextHolder中，供security的其他部分使用。如果不存在，就创建一个securityContext出来，放在securityContextHolder中，供security的其他部分使用。
- 第二个用途：在所有过滤器执行完毕后，清空securityContextHolder中的内容，因为securityContextHolder是基于ThreadLocal的，如果不清空，会受到服务器线程池机制的影响。
- ThreadLocal存放的值是线程内共享的，线程间互斥的，主要用于线程内共享一些数据，避免通过参数来传递。这样处理后，能够解决实际中的一些并发问题。ThreadLocalMap是ThreadLocal的一个内部类，是不对外使用的。当使用ThreadLocal存值时，首先获取到当前线程对象，然后获取到当前线程本地对象，本地变量map,最后将当前使用的所有local和传入的值放在map中。也就是说ThreadLocalMap中的key是ThreadLocal对象。这样，每个线程都对应一个本地的map,所以，一个线程可以存在多个线程本地变量。
- ThreadLocal是解决线程并发问题的一个很好的思路，通过对每个线程提供一个独立的变量副本，解决线程并发访问变量的一个冲突问题
- 当一个线程结束的时候，记得把ThreadLocal里的变量移除掉remove();

### 2. LogoutFilter

- 只处理注销请求。在用户发送注销请求时，销毁用户的session,清空securityContextHolder,重定向到注销成功页面

### 3. AbstractAuthenticationProcessingFilter

- 处理form登录的过滤器，与form登录有关的操作都在此进行。

### 4. DefaultLoginPageGeneratingFilter

- 用来生成一个默认的登录页面，默认的访问地址为spring_security_login,这个登录页面虽然支持用户输入用户名密码，也支持remember me等功能，但是因为太难看了，只能在演示时做个样子，不能直接在实际项目中使用

### 5. BasicAutenticationFilter

- 用来进行basic验证

### 6. SecurityContextHolderAwareRequestFilter

- 用来包装客户的请求，目的是在原来请求的基础上，为后续程序提供一些额外的数据，比如getRomoteUser时，直接返回当前登录的用户名

### 7. RememberMeAuthenticationFilter

- 实现Remember me功能，当用户cookie中存在remember me标记时，它会根据标记自动实现用户登录，并创建securityContext,授予对应的权限。spring security中的remember me依赖cookie实现，用户在登录时选择remember me,系统就会在登录成功后为用户生成一个唯一的标识，并将这个标识保存进cookie中，我们可以通过浏览器查看用户电脑中的cookie

### 8. AnonymousAutenticationFilter

- 当用户没有登录时，默认为用户分配匿名用户的权限

### 9. ExceptionTranslationFilter

- 处理filterSecurityInterceptor中抛出的异常，然后将请求重定向到对应页面，或返回应用的错误代码

### 10. SessionManagementFilter

- 在用户登录成功之后，销毁用户的当前session，并重新生成一个session

### 11. filterSecurityInterceptor

- 用户的权限控制都包含在这个过滤器中
- 第一个功能，如果用户尚未登录，抛出尚未认证的异常
- 第二个功能，如果用户已登录，但是没有访问当前资源的权限，会抛出拒绝访问的异常
- 第三个功能，如果用户已登录，也具有访问当前资源的权限，那么放行

### 12. FilterChainProxy

- 按照顺序调用一组filter,使他们既能完成验证授权的本职工作，又能相应spring Ioc的功能来很方便地得到其他依赖的资源