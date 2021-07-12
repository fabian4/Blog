---
title: Spring Bean配置——XML配Bean
date: 2020/4/1
description: XML配Bean
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/IdxtVCnPlOoFfLW.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/IdxtVCnPlOoFfLW.jpg'
categories:
  - Spring
tags:
  - java
  - Spring
abbrlink: 3026
---

# Spring IoC中的Spring Bean配置——XML配Bean

### 概述

用另一种方式来解释bean，就可以把他当成一个对象管理工具，即容器概念。Spring就是通过bean来**管理我们项目所需要的对象**来减少类与类之间的依赖，从而降低耦合。而我们需要做的就是**通过这个类的个管理工具来进行创建类和调用类**。而类的调用即类的方法使用则需要用到AOP即面向切面编程，我们在这里先来进行**类创建**的总结。

### 代码准备

- maven导入依赖

  ~~~xml
  <dependencies>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-context</artifactId>
              <version>5.2.5.RELEASE</version>
          </dependency>
      </dependencies>
  ~~~

- 创建三个类

  - A：业务层层类，需要调用dao层B来实现功能
  - B：持久层层，由A类进行调用
  - C：其他工厂类来为A新建B对象

  目标：使用Bean工厂创建B类来为A类提供功能

## 一、创建工厂来获取对象

即获取Spring容器，并根据id获取对象

**ApplicationContext**的三个实现类，即获取容器的三个方法

- **`ClassPathXmlApplicationContext`**：加载类路径下的配置文件
- **``FileSystemXmlApplicationContext`**：加载任意路径下的配置文件，参数为绝对路径，需要有权限
- **`AnnotationConfigApplicationContext`**：读取注解创建容器

### 使用容器的getBean方法来获取对象 

- `getBean("id")`：直接通过id来获取对象，需要**强转一下类型**。
- `getBean("id",类名.class)`：通过类的字节码文件，不需要强转

~~~java
 public static void main(String[] args) {
        //1.获取核心对象容器
        ApplicationContext ac = new ClassPathXmlApplicationContext("Bean.xml");
        ApplicationContext ac2 = new FileSystemXmlApplicationContext("D:\\Demo\\SpringDemo\\src\\main\\resources\\Bean.xml");
        //2.根据id获取对象
         B dao = (B) ac.getBean("B");
         B dao2 = ac.getBean("B",B.class);
    }
~~~

**那么由此可见，该方法的重点就是如何通过配置xml来标记 id 来获取bean。**

## 二、xml 配 Bean

那么接下来就是IoC中的一个重点：**如何通过配置xml文件来配置**

首先要给xml配置一个前缀

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
~~~

接下来我们所有的配置都会紧跟在此前缀之后，**在最外面的Beans标签内**

## Spring对bean的管理细节

### 1、创建bean的三种方式，即将被调用对象创建好放入容器的三种方式

- 使用本类的**默认无参构造创建**

  在Spring的配置文件中使用bean标签，**配以id和class属性之后且没有其他属性和标签**，此时如果类中没有默认构造函数则无法创建。

  ~~~xml
  <bean id = "B" class="dao.B"></bean>
  ~~~

- 使用**工厂类或其他类的成员方法**来创建对象

  首先需要构造该类对象，再用该类对象的方法来构造所需要的类的对象

  ~~~xml
   <bean id = "C" class="dao.C"></bean>
   <bean id="B" factory-bean="C" factory-method="get"></bean>
  ~~~

  C 为创建 B 的工厂类，即 C 类含有成员方法，其返回值为 B 类对象。

- 使用**工厂中的静态方法或某个类的静态方法**来创建对象、

  由于类的静态方法可以直接调用，所有不需要创建工厂类或其他类

  ~~~xml
  <bean id="C" class="dao.B" factory-method="get2"></bean>
  ~~~

  C 类中含有一个静态方法，其返回值为 B 类。

  <u>上述方法中class后面都需要填写该类的全类名</u>

### 2、bean对象的作用范围

在bean标签中还可以再加入其他标签来进行修饰

​	**用scope标签来指定作用范围**

- singleton        单例（默认值）
- prototype        多例
- request          web应用的请求范围
- session          web应用的会话范围
- global-session   集群环境的会话范围，当不是集群，效果和session一样

### 3、bean对象的生命周期

- **单例对象**：容器创建被创建对象，容器销毁就被销毁，和容器生命周期一样
- **多例对象**：使用对象时创建对象，当对象长时间不用且没有其他对象引用时，由Java的垃圾回收机制回收

## Spring中的依赖注入 Dependency Injection

依赖关系管理交给Spring来维护，在当前来需要使用其他类的对象，由Spring为我们提供，我们只需要在配置中说明，此过程称为依赖注入。即**使用含参构造方法构造对象**和**为对象的成员变量赋值**。

- 可以注入的参数分为三种：**基本数据类型和String**、**其他Bean对象**、**复杂类型/集合类型**
- 注入方式也分为三种：**含参构造方法**、**set方法**、**注解构造**

### 1、含参构造

**在bean标签内部使用constructor-arg标签**

标签属性：

- type：用于指定要注入数据类型，该数据类型也是构造函数中某个或某些参数的类型

- index：用于指定要注入数据类型给构造参数中指定索引参数的位置赋值，索引从0开始

- name：用于指定给构造函数中指定名称的参数赋值

- value：用于给基本类型和String类型赋值

- ref：用于给其他类型赋值

  前三者时确定给哪个参数赋值，后二者时确定赋什么值

B 类的参数和构造方法

~~~java
public class B {
    private String name;
    private Integer age;
    private Date date;
    //含参构造
    public B(String name, Integer age, Date date){
        this.name = name;
        this.age = age;
        this.date = date;
    }
}
~~~

xml 配置

~~~xml
<bean id="B" class="dao.B" >
        <constructor-arg type="java.lang.String" value="name"></constructor-arg>
        <constructor-arg index="1" value="18"></constructor-arg>
        <constructor-arg name="date" ref="now"></constructor-arg>
</bean>
<bean id="now" class="java.util.Date"></bean>
~~~

> 由于Date类型不是基本类型或String，所以需要先创建对象再用 ref 传值。
>
> 其中还存在一个初始化回调和销毁回调标签，可以在类的创建开始销毁时自动调用该方法
>
> init-method=“ ”    destory-method=“ ”

### 2、set方法传参

**在bean标签内部使用 property 标签**

- 基本类型和String 

  ```xml
  <property name="" value=""></property>
  ```

- list集合、array集合、set、集合  ：list array set 标签

  ```xml
   <property name="" >
      <array>
          <value>待传入的值</value>
          <value>待传入的值</value>
      </array>
  </property>
  ```

- map集合、property集合：map props 标签

  ```xml
  <property name="" >
      <map>
          <entry key="" value=""></entry>
          <entry key="">待传入的值</entry>
      </map>
  </property>
  ```

B 类的参数和构造方法

~~~java
public class B {
    //无参构造
    public B(){};
    //参数列表 
    private String name;
    private Integer age;
    private int[] a;
    private List<String> b;
    private Set<String> c;
    private Map<String,String> d;
    private Properties e;

    public void setA(int[] a) { this.a = a;}
    public void setB(List<String> b) { this.b = b;}
    public void setC(Set<String> c) {this.c = c;}
    public void setD(Map<String, String> d) {this.d = d;}
    public void setE(Properties e) {this.e = e;}
}

~~~

xml 配置

~~~xml
<bean id="B" class="dao.B" >
    <property name="name" value="zhangsan"></property>
    <property name="age" value="18"></property>
    <property name="a" >
        <array>
            <value>1</value>
            <value>2</value>
        </array>
	</property>
    <property name="b" >
        <list>
            <value>a</value>
            <value>b</value>
        </list>
	</property>
    <property name="c" >
        <set>
            <value>a</value>
            <value>b</value>
        </set>
	</property>
    <property name="d" >
        <map>
            <entry key="a" value="a"></entry>
            <entry key="a">a</entry>
        </map>
	</property>
    <property name="" >
        <props>
            <entry key="a" value="a"></entry>
            <entry key="a">a</entry>
        </props>
	</property>
</bean>
~~~

**同一个集合类型标签不固定**，即array、list、set 三者可互换，map、props可互换。