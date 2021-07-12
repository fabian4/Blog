---
title: SpringBoot 干货总结
date: 2020/8/27
description: SpringBoot 干货总结
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/NVO8hr3XDt7c56A.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/NVO8hr3XDt7c56A.jpg'
categories:
  - Spring
tags:
  - SpringBoot
  - Java
abbrlink: 26822
---

# SpringBoot 干货总结

## 一、IOC容器

对于 IOC 容器来说，只需要你告诉它需要某个 bean，它就把对应的实例（instance）扔给你，至于这个 bean 是否依赖其他组件，怎样完成它的初始化，根本就不需要你的关心。

而 IOC 容器想要管理各个业务对象以及它们之间的依赖关心，需要通过某种途径来记录和管理这些信息。**BeanDefinition对象就承担了这个责任**

### BeanDefinition 对象

容器中的每一个 bean 都会有一个对应的 BeanDefinition 实例，该实例负责保存 bean 对象的所有必要信息，包括 bean 对象的 class 类型、是否是抽象类、构造方法和参数、其他属性等等。

当客户端想容器请求相应对象时，容器就会通过这些信息为客户端返回一个完整可用的 bean 实例

当 BeanDefinition 已经准备好，BeanDefinitionRegistry 抽象出 bean 的注册逻辑，而 BeanFactory 则抽象出了 bean 的管理逻辑，而各个 BeanFactory 的实现类就具体承担了 bean 的注册以及管理工作。

![image-20200827110609911](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/MgXTdFH9akbh4fn.png)

DefaultListableBeanFactory 作为一个比较通用的 BeanFactory 实现，它同时也实现了 BeanDefinitionRegistry 接口，因此它就承担了 Bean 的注册管理工作。从图中也可以看出， BeanFactory 接口中主要包含 getBean、getType、getAlisses等管理 bean 的方法，而 BeanDefinitionRegistry 接口则包含 registerBeanDefinition、removeBeanDefinition、getBeanDefinition等注册管理BeanDefinition的方法

**模拟BeanFactory底层工作**

~~~java
// 默认容器实现
DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory();
 // 根据业务对象构造相应的BeanDefinition
AbstractBeanDefinition definition = new RootBeanDefinition(Business.class,true);
 // 将bean定义注册到容器中
beanRegistry.registerBeanDefinition("beanName",definition);
 // 如果有多个bean，还可以指定各个bean之间的依赖关系
 // ........
 // 然后可以从容器中获取这个bean的实例
 // 注意：这里的beanRegistry其实实现了BeanFactory接口，所以可以强转，
 // 单纯的BeanDefinitionRegistry是无法强制转换到BeanFactory类型的
BeanFactory container = (BeanFactory)beanRegistry;
Business business = (Business)container.getBean("beanName");
~~~

这一段代码仅仅为了说明 BeanFanctory 底层的大致工作流程，实际情况会更加复杂，比如 bean 之间的依赖关系可能定义在外部配置（XML/Properties）中，也可能是注解方式。

### IOC容器工作流程

**Spring IOC容器的整个工作流程大致可以分为两个阶段**

1. 容器启动阶段

   容器启动时，会通过某种途径加载 Configuration MataData。除了代码方式比较直接外，在大部分情况下，容器需要依赖某些工具类，比如：BeanDefinitionReader，BeanDefinitionReader会对加载的Configuration MetaData进行解析和分析，并将分析后的信息组成为相应的BeanDefinition，最后把这些保存了bean定义BeanDefinition，注册到相应的BeanDefinitionRegistry，这样容器的启动工作就完成了。

   这个阶段主要完成一些准备性工作，更侧重于bean对象管理的收集，当然一些验证性或者辅助性的工作也在这一阶段完成。

   **BeanFactory如何从配置文件中加载bean的定义以及吗依赖关系：**

   ~~~java
   // 通常为BeanDefinitionRegistry的实现类，这里以DeFaultListabeBeanFactory为例
   BeanDefinitionRegistry beanRegistry = new DefaultListableBeanFactory();
   // XmlBeanDefinitionReader实现了BeanDefinitionReader接口，用于解析XML文件
   XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReaderImpl(beanRegistry);
   // 加载配置文件 
   beanDefinitionReader.loadBeanDefinitions("classpath:spring-bean.xml");
   // 从容器中获取bean实例
   BeanFactory container = (BeanFactory)beanRegistry;
   Business business = (Business)container.getBean("beanName");
   ~~~

2. Bean的实例化阶段

    经过第一阶段，所有bean定义都通过BeanDefinition的方式注册到BeanDefinitionRegistry中当某个请求通过容器的getBean方法请求某个对象，或者因为依赖关系容器需要隐式的调用getBean时，就会触发第二阶段的活动：容器会首先检查所请求的对象之前是否已经实例化完成。
    
    如果没有，则会根据注册的BeanDafinition所提供的信息实例化被请求对象，并为其注入依赖。
    
    当该对象装配完毕后，容器会立即将其返回给请求方法使用。BeanFactory知识Spring IOC容器的一种实现，如果没有特殊指定，它采用延迟初始化策略：只有当访问容器中的某个对象时，才对该对象进行初始化和依赖注入操作。
    
    而在实际场景下，我们更多的使用另外一种类型的容器：ApplicationContext，它构建在BeanFactory之上，属于更高级的容器，除了具有BeanFactory的所有能力之外，还提供对事件监听机制以及国际化的支持等。它管理的bean，在容器启动时全部完成初始化和依赖注入操作。

### Spring容器扩展机制

IOC容器负责管理容器所有bean的生命周期，而在bean生命周期的不同阶段，Spring提供了不同的扩展点来改变bean的命运。在容器启动阶段，BeanFactoryPoatProcessor允许我们在容器实例化对象之前，对注册到容器的BeanDefinition所保存的信息做一些额外的操作，比如修改bean定义的某些属性或者增加其他信息等等。

如果要自定义扩展类，通常要实现**org.springframework.beans.config.BeanFactoryPostProcessor**接口，与此同时，因为容器中可能有多个 **BeanFactoryPostProcessor**，可能还需要实现 **org.springframework.core.Ordered**接口，以保证 **BeanFactoryPostProcessor** 按照顺序执行。

Spring 提供了为数不多的 **BeanFactoryPostProcessor** 实现，我们这里以**PropertyPlaceholderConfigurer**来说明其大致工作流程：

在Spring项目的XML配置文件中，经常可以看到许多配置项的值使用占位符，而将占位符所代表的值单独配置搭配独立的 **properties** 文件，这样可以将散落在不同XML文件中的配置集中管理，而且也方便运维根据不同的环境进行配置不用的值。

这个非常实用的功能就是由 **PropertyPlaceholderConfigurer** 负责实现的。

> 根据前文，当BeanFactory在第一阶段加载完所有配置信息时，BeanFactory中保存的对象的属性还是以占位符方式存在，比如 **${jdbc.mysql.url}**

当 PropertyPlaceholderConfigurer作为BeanFactoryPostProcessor被应用时，它会使用properties配置文件中的值来替换相应的BeanDefinition中占位符所表示的属性值。当需要实例化bean时，bean定义中的属性值就已经被替换成我们配置的值。当然其实现比上面描述的要复杂一些，这里仅说明其大致工作原理，更详细的实现可以参考其源码。

与之相似的，还有 BeanPostProcessor，其存在于对象实例化阶段。根BeanFactoryPostProcessor类似，会处理容器内所有符合条件并且已经实例化后的对象。

简单的对比：

- BeanFactoryPostProcessor处理bean的定义
- BeanPostProcrssor处理bean完成实例化后的对象

**BeanPostProcessor定义了两个接口**：

~~~java
// 通常为BeanDefinitionRegistry的实现类，这里以DeFaultListabeBeanFactory为例
BeanDefinitionRegistry beanRegistry = new DefaultListableBeanFactory();
// XmlBeanDefinitionReader实现了BeanDefinitionReader接口，用于解析XML文件
XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReaderImpl(beanRegistry);
// 加载配置文件 
beanDefinitionReader.loadBeanDefinitions("classpath:spring-bean.xml");
// 从容器中获取bean实例
BeanFactory container = (BeanFactory)beanRegistry;
Business business = (Business)container.getBean("beanName");
~~~

**为了理解这两个方法执行的时机，简单的了解下bean的整个生命周期**

![image-20200827221535647](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/HP4er5hOTb1sC3M.png)

postProcessBeforeInitialization()方法与postProcessAfterInitialization()分别对应图中前置处理和后置处理两个步骤将执行的方法。这两个方法都传入了bean对象实例的引用，为扩展容器的对象实例化过程提供了很大便利，在这儿几乎可以对传入的实例执行任何操作。

> 注解、AOP等功能的实现均大量使用了BeanPostProcessor，比如有一个自定义注解，你完全可以实现BeanPostProcessor的接口，在其中判断bean对象的脑袋上是否有该注解，如果有，你可以对这个bean实例执行任何操作。
>
> 在Spring中经常能够看到各种各样的Aware接口，其作用就是在对象实例化完成以后将Aware接口定义中规定的依赖注入到当前实例中。
>
> 比如最常见的ApplicationContextAware接口，实现了这个接口的类都可以获取到一个ApplicationContext对象。当容器中每一个对象的实例化过程走到BeanPostProcessor前置处理这一步时，容器会检测到之前注册到容器的ApplicationContextAwareProcessor，然后就会调用其postProcessBeforeInitialization()方法，检查并设置Aware相关依赖

~~~java
// org.springframework.context.support.ApplicationContextAwareProcessor
// 其postProcessBeforeInitialization方法调用了invokeAwareInterfaces方法
private void invokeAwareInterfaces(Object bean){
     if (bean instanceof EnvironmentAware){
         ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
     }
     if (bean instanceof ApplicationContextAware){
         ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
     }
     // ......
}
~~~

## 二、JavaConfig与常见Annotation

### JavaConfig

我们知道bean是Spring IOC中非常核心的概念，Spring容器负责bean的生命周期的管理。

在最初，Spring使用XML配置文件的方式来描述bean的定义以及相互间的依赖关系，但随着Spring的发展，越来越多的人对这种方式表示不满，因为Spring项目的所有业务类均以bean的形式配置在XML文件中，造成了大量的XML文件，使项目变得复杂且难以管理。

后来，基于纯Java Annotation依赖注入框架Guice出世，其性能明显优于采用XML方式的Spring，甚至有部分人认为，Guice可以完全取代Spring（Guice仅是一个轻量级IOC框架，取代Spring还差的挺远）

正是这样的危机感，促使Spring及社区推出并持续完善了JavaConfig子项目，它基于Java代码和Annotation注解来描述bean之间的依赖绑定关系。

**XML 配置方法描述bean定义**

~~~xml
<bean id="bookService" class="me.fabian.service.BookServiceImpl"></bean>
~~~

**基于JavaConfig的配置形式**

~~~java
@Configuration
public class
MoonBookConfiguration
{
    // 任何标志了@Bean的方法，其返回值将作为一个bean注册到Spring的IOC容器中
    // 方法名默认成为该bean定义的id
    @Bean
    public BookService bookService() {
       return new BookServiceImpl();
    }
}
~~~

如果两个bean之间有依赖关系，**XML中配置方法**

~~~xml
<bean id="bookService" class="me.fabianme.fabian.service.BookServiceImpl">
     <property name="dependencyService" ref="dependencyService"/>
 </bean>

  <bean id="otherService" class="me.fabian.service.OtherServiceImpl">
     <property name="dependencyService" ref="dependencyService"/>
 </bean>

  <bean id="dependencyService" class="DependencyServiceImpl"/>
~~~

**JavaConfig中**

~~~java
@Configuration
 public class MoonBookConfiguration {
    // 如果一个bean依赖另一个bean，则直接调用对应JavaConfig类中依赖bean的创建方法即可
    // 这里直接调用dependencyService()
     @Bean
     public BookService bookService() {
         return new BookServiceImpl(dependencyService());
     }
      @Bean
     public OtherService otherService() {
         return new OtherServiceImpl(dependencyService());
     }
      @Bean
     public DependencyService dependencyService() {
         return new DependencyServiceImpl();
     }
 }
~~~

### @ComponentScan

@ComponentScan注解对应XML配置形式中的元素表示启用组件扫描，Spring会自动扫描所有通过注解配置的bean，然后将其注册到IOC容器中。

我们可以通过basePackages等属性来指定@ComponentScan自动扫描的范围，如果不指定，默认从声明@ComponentScan所在类的package进行扫描。正因为如此，SpringBoot的启动类都默认在src/main/java下。

### @Import

**@Import注解用于导入配置类，举个简单的例子：**

```java
@Configuration
 public class MoonBookConfiguration{
     @Bean
     public BookService bookService() {
         return new BookServiceImpl();
     }
 }
```

现在有另外一个配置类，比如：MoonUserConfiguration，这个配置类中有一个bean依赖于MoonBookConfiguration中的bookService，如何将这两个bean组合在一起？

**借助@Import即可：**

```java
@Configuration
 // 可以同时导入多个配置类，比如：@Import({A.class,B.class})
 @Import(MoonBookConfiguration.class)
 public class MoonUserConfiguration
 {
     @Bean
     public UserService userService(BookService bookService) {
         return new BookServiceImpl(bookService);
     }
 }
```

需要注意的是，在4.2之前，@Import注解只支持导入配置类，但是在4.2之后，它支持导入普通类，并将这个类作为一个bean的定义注册到IOC容器中。

### @Conditional

@Conditional注解表示在满足某种条件后才初始化一个bean或者启用某些配置。

它一般用在由@Component、@Service、@Configuration等注解标识的类上面，或者由@Bean标记的方法上。如果一个@Configuration类标记了@Conditional，则该类中所有标识了@Bean的方法和@Import注解导入的相关类将遵从这些条件。

在Spring里可以很方便的编写你自己的条件类，所要做的就是实现Condition接口，并覆盖它的matches()方法。

**举个例子，下面的简单条件类表示只有在Classpath里存在JdbcTemplate类时才生效:**

~~~java
public class JdbcTemplateCondition implements Condition {

      @Override
     public boolean matches(ConditionContext
 conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
         try {
         conditionContext.getClassLoader().loadClass("org.springframework.jdbc.core.JdbcTemplate");
             return true;
         }
 catch (ClassNotFoundException e) {
             e.printStackTrace();
         }
         return false;
     }
 }
~~~

**当你用Java来声明bean的时候，可以使用这个自定义条件类：**

~~~java
@Conditional(JdbcTemplateCondition.class) 
@Service 
public MyService service() { ...... }
~~~

这个例子中只有当JdbcTemplateCondition类的条件成立时才会创建MyService这个bean。

也就是说MyService这bean的创建条件是classpath里面包含JdbcTemplate，否则这个bean的声明就会被忽略掉。

Spring Boot定义了很多有趣的条件，并把他们运用到了配置类上，这些配置类构成了Spring Boot的自动配置的基础。

Spring Boot运用条件化配置的方法是：定义多个特殊的条件化注解，并将它们用到配置类上。

**下面列出了Spring Boot提供的部分条件化注解：**

| 条件化注解                      | 配置生效条件                                         |
| ------------------------------- | ---------------------------------------------------- |
| @ConditionalOnBean              | 配置了某个特定的bean                                 |
| @ConditionalOnMissingBean       | 没有配置特定的bean                                   |
| @ConditionalOnClass             | Classpath里有指定的类                                |
| @ConditionalOnMissingClass      | Classpath里没有指定的类                              |
| @ConditionalOnExpression        | 给定的Spring Expression Language表达式计算结果为true |
| @ConditionalOnJava              | Java的版本匹配特定值或者一个范围值                   |
| @ConditionalOnProperty          | 指定的配置属性要有一个明确的值                       |
| @ConditionalOnResource          | Classpath有指定的资源                                |
| @ConditionalOnWebApplication    | 这是一个Web应用程序                                  |
| @ConditionalOnNotWebApplication | 这不是一个Web应用程序                                |

### @ConfigurationProperties与@EnableConfigurationProperties

当某些属性的值需要配置的时候，我们一般会在application.properties文件中新建配置项，然后在bean中使用@Value注解来获取配置的值，比如下面配置数据源的代码。

~~~java
// jdbc config
 jdbc.mysql.url=jdbc:mysql://localhost:3306/sampledb
 jdbc.mysql.username=root
 jdbc.mysql.password=123456
 ......
// 配置数据源
@Configuration
public class HikariDataSourceConfiguration {

     @Value("jdbc.mysql.url")
     public String url;
     @Value("jdbc.mysql.username")
     public String user;
     @Value("jdbc.mysql.password")
     public String password;

     @Bean
     public HikariDataSource dataSource() {
         HikariConfig hikariConfig = new HikariConfig();
         hikariConfig.setJdbcUrl(url);
         hikariConfig.setUsername(user);
         hikariConfig.setPassword(password);
         // 省略部分代码
         return new HikariDataSource(hikariConfig);
     }
}
~~~

使用@Value注解注入的属性通常都比较简单，如果同一个配置在多个地方使用，也存在不方便维护的问题

对于更为复杂的配置，Spring Boot提供了更优雅的实现方式,那就是@ConfigurationProperties注解。

**我们可以通过下面的方式来改写上面的代码：**

~~~java
@Component
// 还可以通过@PropertySource("classpath:jdbc.properties")来指定配置文件
@ConfigurationProperties("jdbc.mysql")
// 前缀=jdbc.mysql，会在配置文件中寻找jdbc.mysql.*的配置项
pulic class JdbcConfig {
     public String url;
     public String username;
     public String password;
}
@Configuration
public class HikariDataSourceConfiguration {

     @AutoWired
     public JdbcConfig config;

     @Bean
     public HikariDataSource dataSource() {
         HikariConfig hikariConfig = new HikariConfig();
         hikariConfig.setJdbcUrl(config.url);
         hikariConfig.setUsername(config.username);
         hikariConfig.setPassword(config.password);
         // 省略部分代码
         return new HikariDataSource(hikariConfig);
   }
}
~~~

@ConfigurationProperties对于更为复杂的配置，处理起来也是得心应手，比如有如下配置文件：

~~~properties
#App
app.menus[0].title=Home
app.menus[0].name=Home
app.menus[0].path=/
app.menus[1].title=Login
app.menus[1].name=Login
app.menus[1].path=/login

app.compiler.timeout=5
app.compiler.output-folder=/temp/

app.error=/error/
~~~

**可以定义如下配置类来接收这些属性：**

```java
@Component
@ConfigurationProperties("app")
public class AppProperties {

     public String error;
     public List<Menu> menus = new ArrayList<>();
     public Compiler compiler = new Compiler();

     public static class Menu {
         public String name;
         public String path;
         public String title;
     }
     public static class Compiler {
         public String timeout;
         public String outputFolder;
     }
 }
```

@EnableConfigurationProperties注解表示对@ConfigurationProperties的内嵌支持默认会将对应Properties Class作为bean注入的IOC容器中，即在相应的Properties类上不用加@Component注解。

## 三、SpringFactoriesLoader详解

**JVM提供了3种类加载器：**

BootstrapClassLoader、ExtClassLoader、AppClassLoader分别加载Java核心类库、扩展类库以及应用的类路径(CLASSPATH)下的类库。

JVM通过**双亲委派模型**进行类的加载，我们也可以通过继承java.lang.classloader实现自己的类加载器。

何为双亲委派模型？当一个类加载器收到类加载任务时，会先交给自己的父加载器去完成，因此最终加载任务都会传递到最顶层的BootstrapClassLoader，只有当父加载器无法完成加载任务时，才会尝试自己来加载。

采用双亲委派模型的一个好处是保证使用不同类加载器最终得到的都是同一个对象，这样就可以保证Java 核心库的类型安全，比如，加载位于rt.jar包中的java.lang.Object类，不管是哪个加载器加载这个类，最终都是委托给顶层的BootstrapClassLoader来加载的，这样就可以保证任何的类加载器最终得到的都是同样一个Object对象。

**查看ClassLoader的源码，对双亲委派模型会有更直观的认识：**

~~~java
protected Class<?> loadClass(String name, boolean resolve) {
     synchronized (getClassLoadingLock(name)) {
     // 首先，检查该类是否已经被加载，如果从JVM缓存中找到该类，则直接返回
     Class<?> c = findLoadedClass(name);
     if (c == null) {
         try {
             // 遵循双亲委派的模型，首先会通过递归从父加载器开始找，
             // 直到父类加载器是BootstrapClassLoader为止
             if (parent != null) {
                 c = parent.loadClass(name, false);
             } else {
                 c = findBootstrapClassOrNull(name);
             }
        } catch (ClassNotFoundException e) {}
         if (c == null) {
             // 如果还找不到，尝试通过findClass方法去寻找
             // findClass是留给开发者自己实现的，也就是说
             // 自定义类加载器时，重写此方法即可
            c = findClass(name);
         }
     }
     if (resolve) {
        resolveClass(c);
     }
     return c;
     }
 }
~~~

但双亲委派模型并不能解决所有的类加载器问题，比如，Java 提供了很多服务提供者接口(Service Provider Interface，SPI)，允许第三方为这些接口提供实现。

常见的 SPI 有 JDBC、JNDI、JAXP 等，这些SPI的接口由核心类库提供，却由第三方实现这样就存在一个问题：SPI 的接口是 Java 核心库的一部分，是由BootstrapClassLoader加载的；SPI实现的Java类一般是由AppClassLoader来加载的。BootstrapClassLoader是无法找到 SPI 的实现类的，因为它只加载Java的核心库。它也不能代理给AppClassLoader，因为它是最顶层的类加载器。也就是说，双亲委派模型并不能解决这个问题。

**线程上下文类加载器**(ContextClassLoader)正好解决了这个问题。

从名称上看，可能会误解为它是一种新的类加载器，实际上，它仅仅是Thread类的一个变量而已，可以通过setContextClassLoader(ClassLoader cl)和getContextClassLoader()来设置和获取该对象。

如果不做任何的设置，Java应用的线程的上下文类加载器默认就是AppClassLoader。

在核心类库使用SPI接口时，传递的类加载器使用线程上下文类加载器，就可以成功的加载到SPI实现的类。

线程上下文类加载器在很多SPI的实现中都会用到。但在JDBC中，你可能会看到一种更直接的实现方式，比如，JDBC驱动管理java.sql.Driver中的loadInitialDrivers()方法中

**你可以直接看到JDK是如何加载驱动的：**

~~~java
for (String aDriver : driversList) {
     try {
         // 直接使用AppClassLoader
         Class.forName(aDriver, true, ClassLoader.getSystemClassLoader());
     } catch (Exception ex) {
         println("DriverManager.Initialize: load failed: " + ex);
      }
   }
~~~

其实讲解线程上下文类加载器，最主要是让大家在看到：

```java
Thread.currentThread().getClassLoader()
Thread.currentThread().getContextClassLoader()
```

这两者除了在许多底层框架中取得的ClassLoader可能会有所不同外，其他大多数业务场景下都是一样的，大家只要知道它是为了解决什么问题而存在的即可。

类加载器除了加载class外，还有一个非常重要功能，就是**加载资源**，它可以从jar包中读取任何资源文件，比如，`ClassLoader.getResources(String name)`方法就是用于读取jar包中的资源文件，其代码如下：

~~~java
public Enumeration<URL> getResources(String name) throws IOException {
     Enumeration<URL>[] tmp = (Enumeration<URL>[]) new Enumeration<?>[2];
     if (parent != null) {
         tmp[0] = parent.getResources(name);
     } else {
         tmp[0] = getBootstrapResources(name);
     }
     tmp[1] = findResources(name);
     return new CompoundEnumeration<>(tmp);
 }
~~~

它的逻辑其实跟类加载的逻辑是一样的,首先判断父类加载器是否为空，不为空则委托父类加载器执行资源查找任务, 直到BootstrapClassLoader，最后才轮到自己查找。

而不同的类加载器负责扫描不同路径下的jar包，就如同加载class一样，最后会扫描所有的jar包，找到符合条件的资源文件。

类加载器的`findResources(name)`方法会遍历其负责加载的所有jar包，找到jar包中名称为name的资源文件，这里的资源可以是任何文件，甚至是.class文件，比如下面的示例，用于查找Array.class文件：

```java
// 寻找Array.class文件
public static void main(String[] args) throws Exception{
     // Array.class的完整路径
     String name = "java/sql/Array.class";
     Enumeration<URL> urls = Thread.currentThread().getContextClassLoader().getResources(name);
     while (urls.hasMoreElements()) {
         URL url = urls.nextElement();
         System.out.println(url.toString());
     }
 }
```

运行后可以得到如下结果：

```
$JAVA_HOME/jre/lib/rt.jar!/java/sql/Array.class
```

根据资源文件的URL，可以构造相应的文件来读取资源内容

下面我们来看一下 SpringFactoriesLoader 的源码

~~~java
public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
 // spring.factories文件的格式为：key=value1,value2,value3
 // 从所有的jar包中找到META-INF/spring.factories文件
 // 然后从文件中解析出key=factoryClass类名称的所有value值
public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
     String factoryClassName = factoryClass.getName();
     // 取得资源文件的URL
     Enumeration<URL> urls = (classLoader != null ? classLoader.getResources(FACTORIES_RESOURCE_LOCATION) : ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
     List<String> result = new ArrayList<String>();
     // 遍历所有的URL
     while (urls.hasMoreElements()) {
         URL url = urls.nextElement();
         // 根据资源文件URL解析properties文件
         Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
         String factoryClassNames = properties.getProperty(factoryClassName);
         // 组装数据，并返回
         result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
     }
     return result;
 }
~~~

从CLASSPATH下的每个Jar包中搜寻所有META-INF/spring.factories配置文件，然后将解析properties文件，找到指定名称的配置后返回。

需要注意的是，其实这里不仅仅是会去ClassPath路径下查找，会扫描所有路径下的Jar包，只不过这个文件只会在Classpath下的jar包中。

来简单看下spring.factories文件的内容吧：

```properties
// 来自 org.springframework.boot.autoconfigure下的META-INF/spring.factories
// EnableAutoConfiguration后文会讲到，它用于开启Spring Boot自动配置功能

org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration\
```

执行loadFactoryNames(EnableAutoConfiguration.class, classLoader)后，得到对应的一组@Configuration类，我们就可以通过反射实例化这些类然后注入到IOC容器中，最后容器里就有了一系列标注了@Configuration的JavaConfig形式的配置类。

这就是SpringFactoriesLoader，它本质上属于Spring框架私有的一种扩展方案，类似于SPI，Spring Boot在Spring基础上的很多核心功能都是基于此。

## 四、Spring容器的事件监听机制

过去，事件监听机制多用于图形界面编程，比如：点击按钮、在文本框输入内容等操作被称为事件，而当事件触发时，应用程序作出一定的响应则表示应用监听了这个事件，而在服务器端，事件的监听机制更多的用于异步通知以及监控和异常处理。

Java提供了实现事件监听机制的两个基础类：自定义事件类型扩展自java.util.EventObject、事件的监听器扩展自java.util.EventListener。

来看一个简单的实例：简单的监控一个方法的耗时。

首先定义事件类型，通常的做法是扩展EventObject，随着事件的发生，相应的状态通常都封装在此类中：

```java
public class MethodMonitorEvent extends EventObject {
     // 时间戳，用于记录方法开始执行的时间
     public long timestamp;

     public MethodMonitorEvent(Object source) {
         super(source);
     }
 }
```

事件发布之后，相应的监听器即可对该类型的事件进行处理，我们可以在方法开始执行之前发布一个begin事件.

在方法执行结束之后发布一个end事件，相应地，事件监听器需要提供方法对这两种情况下接**收到的事件进行处理：**

```java
 // 1、定义事件监听接口
 public interface MethodMonitorEventListener extends EventListener {
     // 处理方法执行之前发布的事件
     public void onMethodBegin(MethodMonitorEvent event);
     // 处理方法结束时发布的事件
     public void onMethodEnd(MethodMonitorEvent event);
 }
 // 2、事件监听接口的实现：如何处理
 public class AbstractMethodMonitorEventListener implements MethodMonitorEventListener {
     @Override
     public void onMethodBegin(MethodMonitorEvent event) {
         // 记录方法开始执行时的时间
         event.timestamp = System.currentTimeMillis();
     }
     @Override
     public void onMethodEnd(MethodMonitorEvent event) {
         // 计算方法耗时
         long duration = System.currentTimeMillis() - event.timestamp;
         System.out.println("耗时：" + duration);
     }
 }
```

事件监听器接口针对不同的事件发布实际提供相应的处理方法定义，最重要的是，其方法只接收MethodMonitorEvent参数，说明这个监听器类只负责监听器对应的事件并进行处理。

有了事件和监听器，剩下的就是发布事件，然后让相应的监听器监听并处理。

通常情况，我们会有一个事件发布者，它本身作为事件源，在合适的时机，将相应的事件发布**给对应的事件监听器：**

```java
public class MethodMonitorEventPublisher {

      private List<MethodMonitorEventListener> listeners = new ArrayList<MethodMonitorEventListener>();

      public void methodMonitor() {
         MethodMonitorEvent eventObject = new MethodMonitorEvent(this);
         publishEvent("begin",eventObject);
         // 模拟方法执行：休眠5秒钟
         TimeUnit.SECONDS.sleep(5);
         publishEvent("end",eventObject);
      }

      private void publishEvent(String status,MethodMonitorEvent event) {
         // 避免在事件处理期间，监听器被移除，这里为了安全做一个复制操作
         List<MethodMonitorEventListener> copyListeners = ➥ new ArrayList<MethodMonitorEventListener>(listeners);
         for (MethodMonitorEventListener listener : copyListeners) {
             if ("begin".equals(status)) {
                 listener.onMethodBegin(event);
             } else {
                 listener.onMethodEnd(event);
             }
         }
     }
         public static void main(String[] args) {
         MethodMonitorEventPublisher publisher = new MethodMonitorEventPublisher();
         publisher.addEventListener(new AbstractMethodMonitorEventListener());
         publisher.methodMonitor();
     }
     // 省略实现
     public void addEventListener(MethodMonitorEventListener listener) {}
     public void removeEventListener(MethodMonitorEventListener listener) {}
     public void removeAllListeners() {}
```

**对于事件发布者（事件源）通常需要关注两点：**

1. 在合适的时机发布事件。此例中的methodMonitor()方法是事件发布的源头，其在方法执行之前和结束之后两个时间点发布MethodMonitorEvent事件，每个时间点发布的事件都会传给相应的监听器进行处理

   > 在具体实现时需要注意的是，事件发布是顺序执行，为了不影响处理性能，事件监听器的处理逻辑应尽量简单。

2. 事件监听器的管理。publisher类中提供了事件监听器的注册与移除方法，这样客户端可以根据实际情况决定是否需要注册新的监听器或者移除某个监听器。

   > 如果这里没有提供remove方法，那么注册的监听器示例将一直MethodMonitorEventPublisher引用，即使已经废弃不用了，也依然在发布者的监听器列表中，这会导致隐性的内存泄漏。

**Spring容器内的事件监听机制**

Spring的ApplicationContext容器内部中的所有事件类型均继承自org.springframework.context.AppliationEvent，容器中的所有监听器都实现org.springframework.context.ApplicationListener接口，并且以bean的形式注册在容器中。

一旦在容器内发布ApplicationEvent及其子类型的事件，注册到容器的ApplicationListener就会对这些事件进行处理。

ApplicationEvent继承自EventObject，Spring提供了一些默认的实现，比如：

- ContextClosedEvent表示容器在即将关闭时发布的事件类型
- ContextRefreshedEvent表示容器在初始化或者刷新的时候发布的事件类型（容器内部使用）
- ApplicationListener作为事件监听器接口定义，它继承自EventListener。
- ApplicationContext容器在启动时，会自动识别并加载EventListener类型的bean一旦容器内有事件发布，将通知这些注册到容器的EventListener。
- ApplicationContext接口继承了ApplicationEventPublisher接口，该接口提供了void publishEvent(ApplicationEvent event)方法定义，不难看出，ApplicationContext容器担当的就是事件发布者的角色。
- ApplicationContext将事件的发布以及监听器的管理工作委托给 ApplicationEventMulticaster接口的实现类。
- 在容器启动时，会检查容器内是否存在名为applicationEventMulticaster的 ApplicationEventMulticaster对象实例。如果有就使用其提供的实现，没有就默认初始化一个SimpleApplicationEventMulticaster作为实现。
- 最后，如果我们业务需要在容器内部发布事件，只需要为其注入ApplicationEventPublisher 依赖即可：实现ApplicationEventPublisherAware接口或者ApplicationContextAware接口.

## 五、自动配置原理

典型的Spring Boot应用的启动类一般均位于src/main/java根路径下

比如MoonApplication类：

```java
@SpringBootApplication
 public class MoonApplication {
      public static void main(String[] args) {
         SpringApplication.run(MoonApplication.class, args);
     }
 }
```

其中@SpringBootApplication开启组件扫描和自动配置，而SpringApplication.run则负责启动引导应用程序。

@SpringBootApplication是一个复合Annotation，它将三个有用的注解组合在一起：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented 
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
 public @interface SpringBootApplication {
     // ......
 }
```

@SpringBootConfiguration就是@Configuration，它是Spring框架的注解，标明该类是一个JavaConfig配置类。

而@ComponentScan启用组件扫描，前文已经详细讲解过，这里着重关注@EnableAutoConfiguration。@EnableAutoConfiguration注解表示开启Spring Boot自动配置功能，Spring Boot会根据应用的依赖、自定义的bean、classpath下有没有某个类 等等因素来猜测你需要的bean，

然后注册到IOC容器中。

那@EnableAutoConfiguration是如何推算出你的需求？

首先看下它的定义：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
     // ......
}
```

你的关注点应该在@Import(EnableAutoConfigurationImportSelector.class)上了，前文说过，@Import注解用于导入类，并将这个类作为一个bean的定义注册到容器中，这里将把EnableAutoConfigurationImportSelector作为bean注入到容器中，而这个类会将**所有符合条件的@Configuration配置都加载到容器中**，看看它的代码：

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
     // 省略了大部分代码，保留一句核心代码
     // 注意：SpringBoot最近版本中，这句代码被封装在一个单独的方法中
     // SpringFactoriesLoader相关知识请参考前文
     List<String> factories = new ArrayList<String>(new LinkedHashSet<String>(
           SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class, this.beanClassLoader)));
 }
```

这个类会扫描所有的jar包，将所有符合条件的@Configuration配置类注入的容器中**何为符合条件，看看META-INF/spring.factories的文件内容：**

```properties
// 来自 org.springframework.boot.autoconfigure下的META-INF/spring.factories
// 配置的key = EnableAutoConfiguration，与代码中一致
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration\
.....
```

以DataSourceAutoConfiguration为例，看看Spring Boot是如何自动配置的：

```java
@Configuration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@EnableConfigurationProperties(DataSourceProperties.class)
@Import({ Registrar.class, DataSourcePoolMetadataProvidersConfiguration.class })
public class DataSourceAutoConfiguration {
}
```

**分别说一说：**

@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })：当Classpath中存在DataSource或者EmbeddedDatabaseType类时才启用这个配置，否则这个配置将被忽略。

@EnableConfigurationProperties(DataSourceProperties.class)：将DataSource的默认配置类注入到IOC容器中，DataSourceproperties定义为：

```java
// 提供对datasource配置信息的支持，所有的配置前缀为：spring.datasource
@ConfigurationProperties(prefix = "spring.datasource")
public class DataSourceProperties {
    private ClassLoader classLoader;
    private Environment environment;
    private String name = "testdb";
    ......
}
```

@Import({ Registrar.class, DataSourcePoolMetadataProvidersConfiguration.class })：导入其他额外的配置，就以DataSourcePoolMetadataProvidersConfiguration为例吧。

```java
@Configuration
public class DataSourcePoolMetadataProvidersConfiguration {

    @Configuration
    @ConditionalOnClass(org.apache.tomcat.jdbc.pool.DataSource.class)
    static class TomcatDataSourcePoolMetadataProviderConfiguration {
        @Bean
        public DataSourcePoolMetadataProvider tomcatPoolDataSourceMetadataProvider() {
            .....
        }
    }
  ......
 }
```

DataSourcePoolMetadataProvidersConfiguration是数据库连接池提供者的一个配置类，即Classpath中存在org.apache.tomcat.jdbc.pool.DataSource.class，则使用tomcat-jdbc连接池，如果Classpath中存在HikariDataSource.class则使用Hikari连接池。

这里仅描述了DataSourceAutoConfiguration的冰山一角，但足以说明Spring Boot如何利用条件话配置来实现自动配置的。

回顾一下，@EnableAutoConfiguration中导入了EnableAutoConfigurationImportSelector类，而这个类的selectImports()通过SpringFactoriesLoader得到了大量的配置类，而每一个配置类则根据条件化配置来做出决策，以实现自动配置。

整个流程很清晰，但漏了一个大问题：

EnableAutoConfigurationImportSelector.selectImports()是何时执行的？其实这个方法会在容器启动过程中执行：AbstractApplicationContext.refresh()。

## 六、启动引导

### SpringApplication初始化

SpringBoot整个启动流程分为两个步骤：

1. 初始化一个SpringApplication对象

2. 执行该对象的run方法


看下SpringApplication的初始化流程，SpringApplication的构造方法中调用initialize(Object[] sources)方法，其代码如下:

```java
private void initialize(Object[] sources) {
      if (sources != null && sources.length > 0) {
          this.sources.addAll(Arrays.asList(sources));
      }
      // 判断是否是Web项目
      this.webEnvironment = deduceWebEnvironment();
      setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
      setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
      // 找到入口类
      this.mainApplicationClass = deduceMainApplicationClass();
 }
```

初始化流程中最重要的就是通过SpringFactoriesLoader找到spring.factories文件中配置的ApplicationContextInitializer和ApplicationListener两个接口的实现类名称，以便后期构造相应的实例。

ApplicationContextInitializer的主要目的是在ConfigurableApplicationContext做refresh之前，对ConfigurableApplicationContext实例做进一步的设置或处理。

ConfigurableApplicationContext继承自ApplicationContext，其主要提供了对ApplicationContext进行设置的能力。

实现一个ApplicationContextInitializer非常简单，因为它只有一个方法，但大多数情况下我们没有必要自定义一个ApplicationContextInitializer，即便是Spring Boot框架，它默认也只是注册了两个实现，毕竟Spring的容器已经非常成熟和稳定，你没有必要来改变它。

而ApplicationListener的目的就没什么好说的了，它是Spring框架对Java事件监听机制的一种框架实现，具体内容在前文Spring事件监听机制这个小节有详细讲解。这里主要说说，如果你想为Spring Boot应用添加监听器，该如何实现。

Spring Boot提供两种方式来添加自定义监听器：

通过**SpringApplication.addListeners(ApplicationListener... listeners)**或者**SpringApplication.setListeners(Collection> listeners)**两个方法来添加一个或者多个自定义监听器

既然SpringApplication的初始化流程中已经从spring.factories中获取到ApplicationListener的实现类，那么我们直接在自己的jar包的META-INF/spring.factories文件中新增配置即可：

```properties
org.springframework.context.ApplicationListener=\ cn.moondev.listeners.xxxxListener\
```

### Spring Boot启动流程.

Spring Boot应用的整个启动流程都封装在SpringApplication.run方法中，其整个流程真的是太长太长了，但本质上就是在Spring容器启动的基础上做了大量的扩展，按照这个思路来看看

源码：

```java
public ConfigurableApplicationContext run(String... args) {
         StopWatch stopWatch = new StopWatch();
         stopWatch.start();
         ConfigurableApplicationContext context = null;
         FailureAnalyzers analyzers = null;
         configureHeadlessProperty();
         // ①
         SpringApplicationRunListeners listeners = getRunListeners(args);
         listeners.starting();
         try {
             // ②
ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
             ConfigurableEnvironment environment = prepareEnvironment(listeners,applicationArguments);
             // ③
             Banner printedBanner = printBanner(environment);
             // ④
             context = createApplicationContext();
             // ⑤
             analyzers = new FailureAnalyzers(context);
             // ⑥
             prepareContext(context, environment, listeners, applicationArguments,printedBanner);
             // ⑦
              refreshContext(context);
             // ⑧
             afterRefresh(context, applicationArguments);
             // ⑨
             listeners.finished(context, null);
             stopWatch.stop();
             return context;
        }
         catch (Throwable ex) {
             handleRunFailure(context, listeners, analyzers, ex);
             throw new IllegalStateException(ex);
         }
     }
```

- ① 通过SpringFactoriesLoader查找并加载所有的SpringApplicationRunListeners通过调用starting()方法通知所有的SpringApplicationRunListeners：应用开始启动了。

  SpringApplicationRunListeners其本质上就是一个事件发布者，它在SpringBoot应用启动的不同时间点发布不同应用事件类型(ApplicationEvent)，如果有哪些事件监听者(ApplicationListener)对这些事件感兴趣，则可以接收并且处理。还记得初始化流程中，SpringApplication加载了一系列ApplicationListener吗？这个启动流程中没有发现有发布事件的代码，其实都已经在SpringApplicationRunListeners这儿实现了。

  简单的分析一下其实现流程，首先看下SpringApplicationRunListener的源码：

  ~~~java
  public interface SpringApplicationRunListener {
       // 运行run方法时立即调用此方法，可以用户非常早期的初始化工作
       void starting();
       // Environment准备好后，并且ApplicationContext创建之前调用
       void environmentPrepared(ConfigurableEnvironment environment);
       // ApplicationContext创建好后立即调用
       void contextPrepared(ConfigurableApplicationContext context);
       // ApplicationContext加载完成，在refresh之前调用
       void contextLoaded(ConfigurableApplicationContext context);
       // 当run方法结束之前调用
       void finished(ConfigurableApplicationContext
   context, Throwable exception);
  }
  ~~~

  SpringApplicationRunListener只有一个实现类：EventPublishingRunListener。

  ①处的代码只会获取到一个EventPublishingRunListener的实例

  我们来看看starting()方法的内容：

  ~~~java
  public void starting() {
       // 发布一个ApplicationStartedEvent
       this.initialMulticaster.multicastEvent(new ApplicationStartedEvent(this.application, this.args));
   }
  ~~~

  顺着这个逻辑，你可以在②处的prepareEnvironment()方法的源码中找到

  ~~~java
  listeners.environmentPrepared(environment);
  ~~~

  即SpringApplicationRunListener接口的第二个方法，那不出你所料，environmentPrepared()又发布了另外一个事件ApplicationEnvironmentPreparedEvent。

- ② 创建并配置当前应用将要使用的Environment，Environment用于描述应用程序当前的运行环境，其抽象了两个方面的内容：配置文件(profile)和属性(properties)，开发经验丰富的同学对这两个东西一定不会陌生：不同的环境(eg：生产环境、预发布境)可以使用不同的配置文件，而属性则可以从配置文件、环境变量、命令行参数等来源获取。因此，当Environment准备好后，在整个应用的任何时候，都可以从Environment中获取资源。

  > 总结起来，②处的两句代码，主要完成以下几件事：
  >
  > - 判断Environment是否存在，不存在就创建（如果是web项目就创建StandardServletEnvironment，否则创建StandardEnvironment）
  > - 配置Environment：配置profile以及properties
  > - 调用SpringApplicationRunListener的environmentPrepared()方法，通知事件监听者：应用的Environment已经准备好

- ③SpringBoot应用在启动时会输出这样的东西：

  ~~~
  . ____          _            __ _ _
    /\\ / ___'_ __ _ _(_)_ __ __ _ \ \ \ \
   ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
    \\/ ___)| |_)| | | | | || (_| | ) ) ) )
     ' |____| .__|_| |_|_| |_\__, | / / / /
    =========|_|==============|___/=/_/_/_/
   :: Spring Boot :: (v1.5.6.RELEASE)
  ~~~

  如果想把这个东西改成自己的涂鸦，你可以研究一下Banner的实现。

- ④根据是否是web项目，来创建不同的ApplicationContext容器。

- ⑤创建一系列FailureAnalyzer，创建流程依然是通过SpringFactoriesLoader获取到所有实现FailureAnalyzer接口的class，然后在创建对应的实例。FailureAnalyzer用于分析故障并提供相关诊断信息。

- ⑥初始化ApplicationContext，主要完成以下工作：
  - 将准备好的Environment设置给ApplicationContext
  - 遍历调用所有的ApplicationContextInitializer的initialize()方法来对已经创建好的ApplicationContext进行进一步的处理
  - 调用SpringApplicationRunListener的contextPrepared()方法，通知所有的监听者：ApplicationContext已经准备完毕
  - 将所有的bean加载到容器中
  - 调用SpringApplicationRunListener的contextLoaded()方法，通知所有的监听者：ApplicationContext已经装载完毕

- ⑦调用ApplicationContext的refresh()方法，完成IoC容器可用的最后一道工序。

  那如何刷新呢？且看下面代码：

  ~~~java
  // 摘自refresh()方法中一句代码
  invokeBeanFactoryPostProcessors(beanFactory);
  ~~~

  看看这个方法的实现：

  ~~~java
  protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
       PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
       ......
   }
  ~~~

  获取到所有的BeanFactoryPostProcessor来对容器做一些额外的操作。

  BeanFactoryPostProcessor允许我们在容器实例化相应对象之前，对注册到容器的BeanDefinition所保存的信息做一些额外的操作。

  这里的getBeanFactoryPostProcessors()方法可以获取到3个Processor：

  - ConfigurationWarningsApplicationContextInitializer$ConfigurationWarningsPostProcessor
  - SharedMetadataReaderFactoryContextInitializer$CachingMetadataReaderFactoryPostProcessor
  - ConfigFileApplicationListener$PropertySourceOrderingPostProcessor

  不是有那么多BeanFactoryPostProcessor的实现类，为什么这儿只有这3个？
  
  因为在初始化流程获取到的各种ApplicationContextInitializer和ApplicationListener中，只有上文3个做了类似于如下操作：
  
  ~~~java
  public void initialize(ConfigurableApplicationContext context) {
       context.addBeanFactoryPostProcessor(new ConfigurationWarningsPostProcessor(getChecks()));
  }
  ~~~
  
  然后你就可以进入到PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors()方法了，这个方法除了会遍历上面的3个BeanFactoryPostProcessor处理外，还会获取类型为BeanDefinitionRegistryPostProcessor的bean：org.springframework.context.annotation.internalConfigurationAnnotationProcessor，对应的Class为ConfigurationClassPostProcessor。ConfigurationClassPostProcessor用于解析处理各种注解，包括：@Configuration、@ComponentScan、@Import、@PropertySource、@ImportResource、@Bean。当处理@import注解的时候，就会调用这一小节中的EnableAutoConfigurationImportSelector.selectImports()来完成自动配置功能。其他的这里不再多讲，如果你有兴趣，可以查阅参考资料6。

- ⑧查找当前context中是否注册有CommandLineRunner和ApplicationRunner，如果有则遍历执行它们。

- ⑨执行所有SpringApplicationRunListener的finished()方法。这就是Spring Boot的整个启动流程，其核心就是在Spring容器初始化并启动的基础上加入各种扩展点，这些扩展点包括：ApplicationContextInitializer、**ApplicationListener以及各种BeanFactoryPostProcessor等等。**

> 你对整个流程的细节不必太过关注，甚至没弄明白也没有关系，你只要理解这些扩展点是在何时如何工作的，能让它们为你所用即可。整个启动流程确实非常复杂，可以查询参考资料中的部分章节和内容，对照着源码，多看看，我想最终你都能弄清楚的。**言而总之**，Spring才是核心，理解清楚Spring容器的启动流程，那Spring Boot启动流程就不在话下了。