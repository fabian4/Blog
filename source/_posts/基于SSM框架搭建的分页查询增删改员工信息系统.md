---
title: SSM框架实战
date: 2020/3/24
description: 基于SSM框架搭建的分页查询增删改员工信息系统
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/k7BNrpecHJqOE2o.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/k7BNrpecHJqOE2o.jpg'
categories:
  - Spring
tags:
  - java
  - Spring
abbrlink: 2861
---

# 基于SSM框架搭建的分页查询增删改员工信息系统

## 前言

​		新人小白初学SSM框架，一步一步面向百度、面向复制黏贴编程调试出来，在这里分享给大家。

实现功能

- 验证码登录

- 登录权限拦截

- 分页查询

- 按多条件模糊查询

- 增删改

  ![](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/rdLvXM6CVkmswaT.png)

## 一、配置maven

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>SSM</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <dependencies>
        <!--Junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <!--数据库驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.42</version>
        </dependency>
        <!-- 数据库连接池 -->
        <dependency>
            <groupId>com.mchange</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.5.2</version>
        </dependency>

        <!--Servlet - JSP -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>

        <!--Mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.2</version>
        </dependency>

        <!--Spring-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.2.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.5.RELEASE</version>
        </dependency>
    </dependencies>
</project>
~~~

## 二、配置文件的编写

### web层配置		web.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4"
         xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

<!--  配置监听器,默认加载spring配置文件-->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
<!--  配置文件加载路径-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:application.xml</param-value>
  </context-param>

<!--  配置前端控制器-->
  <servlet>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

<!--加载配置文件-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
<!--启动服务器创建控制器-->
    <load-on-startup>1</load-on-startup>
  </servlet>

  <servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

<!--  配置中文乱码过滤器-->
  <filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
      <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
      </init-param>
  </filter>
  <filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
</web-app>
~~~

### Spring配置         application.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!--开启注解扫描，排除controller-->
    <context:component-scan base-package="com">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!--    spring 整合 Mybatis-->
    <!--    配置连接池-->
    <bean name="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://127.0.0.1:3306/user"/>
        <property name="user" value="root"/>
        <property name="password" value="root"/>
    </bean>
    <!--    配置工厂-->
    <bean name="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--    配置接口所在包-->
    <bean name="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.dao"/>
    </bean>
</beans>
~~~

### SpringMVC配置         springmvc.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-4.0.xsd">

<!--    开启注解扫描-->
    <context:component-scan base-package="com.controller"/>
<!--    视图解析器-->
    <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
<!--    过滤静态资源-->
    <mvc:resources mapping="/css/" location="/css/**"/>
    <mvc:resources mapping="/js/" location="/js/**"/>
    <mvc:resources mapping="/fonts/" location="/fonts/**"/>
<!--    开启spring mvc注解支持-->
    <mvc:annotation-driven/>

<!--    配置登录拦截器-->
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**" />
            <mvc:exclude-mapping path="/login"/>
            <mvc:exclude-mapping path="/checkCode"/>
            <bean class="com.controller.LoginInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>

</beans>
~~~

##  三、control类的编写

### UserController

~~~Java
package com.controller;
import com.domain.PageBean;
import com.domain.User;
import com.service.impl.UserServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.SessionAttributes;

import javax.imageio.ImageIO;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.IOException;
import java.util.Map;
import java.util.Random;

/**
 * @author 丁生
 * controller层的页面控制跳转
 */

@Controller
@SessionAttributes("loginUser")
public class UserController {

    @Autowired
    private UserServiceImpl service;


    @RequestMapping("findUserByPage")
    public String findAll(@RequestParam Map<String,Object> map, Model model){
        PageBean pb = new PageBean();
        if(map.get("currentPage")==null){
            pb.setCurrentPage(1);
        }else{
            pb.setCurrentPage(Integer.parseInt(map.get("currentPage").toString()));
        }
        if(map.get("name")!=null){
            pb.setName(map.get("name").toString());
        }else{
            pb.setName("");
        }
        if(map.get("address")!=null){
            pb.setAddress(map.get("address").toString());
        }else{
            pb.setAddress("");
        }
        if(map.get("email")!=null){
            pb.setEmail(map.get("email").toString());
        }else{
            pb.setEmail("");
        }
        model.addAttribute("pb", service.findUserByPage(pb));
        return "list";
    };

    @RequestMapping("checkCode")
    public void checkCode(HttpServletRequest request, HttpServletResponse response) throws IOException {
        //服务器通知浏览器不要缓存
        response.setHeader("pragma","no-cache");
        response.setHeader("cache-control","no-cache");
        response.setHeader("expires","0");

        //在内存中创建一个长80，宽30的图片，默认黑色背景
        //参数一：长
        //参数二：宽
        //参数三：颜色
        int width = 80;
        int height = 30;
        BufferedImage image = new BufferedImage(width,height,BufferedImage.TYPE_INT_RGB);

        //获取画笔
        Graphics g = image.getGraphics();
        //设置画笔颜色为灰色
        g.setColor(Color.GRAY);
        //填充图片
        g.fillRect(0,0, width,height);

        //产生4个随机验证码，12Ey
        String base = "0123456789ABCDEFGabcdefg";
        int size = base.length();
        Random r = new Random();
        StringBuffer sb = new StringBuffer();
        for(int i=1;i<=4;i++){
            //产生0到size-1的随机值
            int index = r.nextInt(size);
            //在base字符串中获取下标为index的字符
            char c = base.charAt(index);
            //将c放入到StringBuffer中去
            sb.append(c);
        }
        String checkCode = sb.toString();
        System.out.println(checkCode);
        //将验证码放入HttpSession中
        request.getSession().setAttribute("CHECKCODE_SERVER",checkCode);

        //设置画笔颜色为黄色
        g.setColor(Color.YELLOW);
        //设置字体的小大
        g.setFont(new Font("黑体",Font.BOLD,24));
        //向图片上写入验证码
        g.drawString(checkCode,15,25);

        //将内存中的图片输出到浏览器
        //参数一：图片对象
        //参数二：图片的格式，如PNG,JPG,GIF
        //参数三：图片输出到哪里去
        ImageIO.write(image,"PNG",response.getOutputStream());
    }

    @RequestMapping("login")
    public String login(String username, String password, String verifycode, HttpSession session, Model model){
        if("".equals(username)|| "".equals(password)|| "".equals(verifycode)){
            model.addAttribute("error","请输入信息");
            return "login";
        }
        if(!verifycode.equalsIgnoreCase(session.getAttribute("CHECKCODE_SERVER").toString())){
            model.addAttribute("error","验证码错误");
            return "login";
        }
        User user = service.login(username, password);
        if(user==null){
            model.addAttribute("error","用户名或密码错误");
            return "login";
        }
        session.setAttribute("loginUser", user);
        return "index";
    }

    @RequestMapping("addUser")
    public String addUser(User user){
        service.add(user);
        return "redirect:findUserByPage";
    }

    @RequestMapping("delUser")
    public String delUser(String id){
        service.delUser(id.toString());
        return "redirect:findUserByPage";
    }

    @RequestMapping("delSelected")
    public String delSelected(@RequestParam("uid") String[] ids){
        for (String id : ids) {
            service.delUser(id);
        }
        return "redirect:findUserByPage";
    }

    @RequestMapping("findUser")
    public String findUser(String id, Model model){
        User user = service.findUser(id);
        model.addAttribute("u",user);
        return "update";
    }

    @RequestMapping("update")
    public String update(User user){
        service.update(user);
        return "redirect:findUserByPage";
    }
}

~~~

### LoginInterceptor

~~~java
package com.controller;

import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 权限拦截器
 * @author 丁生
 */
public class LoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Object user = request.getSession().getAttribute("loginUser");
        if (user == null) {
            response.sendRedirect("/login.jsp");
            return false;
        }
        return true;
    }
}

~~~

## 四、service类的编写

### 接口

~~~java
package com.service;

import com.domain.PageBean;
import com.domain.User;

import java.util.List;

/**
 * @author 丁生
 */
public interface UserService {

    /**
     * 按页查询
     * @param
     */
    public PageBean findUserByPage(PageBean pb);

    /**
     * 登录查询
     */
    public User login(String username, String password);

    /**
     * 增加用户
     */
    public void add(User user);

    /**
     * 删除用户
     * @Param
     */
    public void delUser(String id);

    /**
     * 按id查找用户
     * @param id
     * @return
     */
    public User findUser(String id);

    /**
     * 更新用户信息
     * @param user
     */
    public void update(User user);
}

~~~

### 实现类

~~~java
package com.service.impl;

import com.dao.UserDao;
import com.domain.PageBean;
import com.domain.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @author 丁生
 */
@Service
public class UserServiceImpl implements com.service.UserService{

    @Autowired
    public UserDao dao;


    @Override
    public PageBean findUserByPage(PageBean pb) {
        pb.setTotalCount(dao.findAll(pb));
        pb.setRows(5);
        int totalPage =((pb.getTotalCount()) % (pb.getRows()))  == 0 ? (pb.getTotalCount())/(pb.getRows()) : (pb.getTotalCount())/(pb.getRows()) + 1;
        pb.setTotalPage(totalPage);
        if(pb.getCurrentPage() >= pb.getTotalPage()){
            pb.setCurrentPage(pb.getTotalPage());
        }
        if(pb.getCurrentPage() <= 1){
            pb.setCurrentPage(1);
        }
        pb.setStartIndex((pb.getCurrentPage()-1)*pb.getRows());
        pb.setUsers(dao.findUserByPage(pb));
        return pb;
    }

    @Override
    public User login(String username, String password) {
        return dao.login(username, password);
    }

    @Override
    public void add(User user) {
        dao.add(user);
    }

    @Override
    public void delUser(String id) {
        dao.delUser(id);
    }

    @Override
    public User findUser(String id) {
        return dao.findUser(id);
    }

    @Override
    public void update(User user) {
        dao.update(user);
    }


}

~~~

## 五、dao层的编写

### dao接口

~~~java
package com.dao;

import com.domain.PageBean;
import com.domain.User;
import org.apache.ibatis.annotations.Param;
import org.springframework.stereotype.Repository;

import java.util.List;

/**
 * @author 丁生
 */
@Repository
public interface UserDao {

    /**
     * 查询所有
     */
    public int findAll(PageBean pb);

    /**
     * 查询所有
     */
    public List<User> findUserByPage(PageBean pb);

    /**
     * 登录
     */
    public User login(@Param("username") String username, @Param("password") String password);

    /**
     * 添加
     */
    public void add(User user);

    /**
     * 删除
     */
    public void delUser(String id);

    public User findUser(String id);

    public void update(User user);
}

~~~

### dao的查询配置文件    dao.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.dao.UserDao">
    <update id="update">
        update user set name = '${name}',gender = '${gender}' ,age = '${age}' , address = '${address}' , qq = '${qq}', email = '${email}' where id = '${id}'
    </update>

    <select id="findAll" resultType="java.lang.Integer" parameterType="com.domain.PageBean">
        select count(*) from user
        <if test="name!=''">
            where name like '%${name}%'
        </if>
        <if test="address!=''">
            where address like '%${address}%'
        </if>
        <if test="email!=''">
            where email like '%${email}%'
        </if>
    </select>

    <select id="findUserByPage" resultType="com.domain.User" parameterType="com.domain.PageBean" >
        select * from user
        <if test="name!=''">
            where name like '%${name}%'
        </if>
        <if test="address!=''">
            where address like '%${address}%'
        </if>
        <if test="email!=''">
            where email like '%${email}%'
        </if>
        limit #{startIndex},#{rows}
    </select>

    <select id="login" resultType="com.domain.User" parameterType="java.lang.String">
        select * from user where username = '${username}' and password = '${password}'
    </select>

    <select id="findUser" resultType="com.domain.User" parameterType="java.lang.String">
        select * from user where id = '${id}'
    </select>

    <insert id="add">
        insert into user
        value (null,'${name}','${gender}','${age}','${address}','${qq}','${email}',null ,null )
    </insert>

    <delete id="delUser" parameterType="java.lang.String">
        delete from user where id = '${id}'
    </delete>
</mapper>
~~~

## 六、封装类

### user

~~~java
package com.domain;

import org.springframework.stereotype.Component;

import java.io.Serializable;

/**
 * @author 丁生
 */
@Component
public class User implements Serializable {

    private int id;
    private String name;
    private String gender;
    private int age;
    private String address;
    private String qq;
    private String email;
    private String username;
    private String password;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getQq() {
        return qq;
    }

    public void setQq(String qq) {
        this.qq = qq;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", gender='" + gender + '\'' +
                ", age=" + age +
                ", address='" + address + '\'' +
                ", qq='" + qq + '\'' +
                ", email='" + email + '\'' +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}

~~~

### PageBean类

~~~java
package com.domain;

import org.springframework.stereotype.Component;

import java.io.Serializable;
import java.util.ArrayList;
import java.util.List;

/**
 * @author 丁生
 */
@Component
public class PageBean implements Serializable {
    //总记录数
    private int totalCount;

    //总页数
    private int totalPage;

    //当前页数
    private int currentPage;

    //当前显示条数
    private int rows;

    //显示本页起始索引
    private int startIndex;

    private String name;
    private String address;
    private String email;

    //显示数据
    private List<User> users = new ArrayList<>();

    public int getTotalCount() {
        return totalCount;
    }

    public void setTotalCount(int totalCount) {
        this.totalCount = totalCount;
    }

    public int getTotalPage() {
        return totalPage;
    }

    public void setTotalPage(int totalPage) {
        this.totalPage = totalPage;
    }

    public int getCurrentPage() {
        return currentPage;
    }

    public void setCurrentPage(int currentPage) {
        this.currentPage = currentPage;
    }

    public int getRows() {
        return rows;
    }

    public void setRows(int rows) {
        this.rows = rows;
    }

    public List<User> getUsers() {
        return users;
    }

    public void setUsers(List<User> users) {
        this.users = users;
    }

    public int getStartIndex() {
        return startIndex;
    }

    public void setStartIndex(int startIndex) {
        this.startIndex = startIndex;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    @Override
    public String toString() {
        return "PageBean{" +
                "totalCount=" + totalCount +
                ", totalPage=" + totalPage +
                ", currentPage=" + currentPage +
                ", rows=" + rows +
                ", startIndex=" + startIndex +
                ", name='" + name + '\'' +
                ", address='" + address + '\'' +
                ", email='" + email + '\'' +
                ", users=" + users +
                '}';
    }
}

~~~

## 七、前端文件

### index.jsp

~~~jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="utf-8"/>
  <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
  <meta name="viewport" content="width=device-width, initial-scale=1"/>
  <title>首页</title>

  <!-- 1. 导入CSS的全局样式 -->
  <link href="css/bootstrap.min.css" rel="stylesheet">
  <!-- 2. jQuery导入，建议使用1.9以上的版本 -->
  <script src="js/jquery-2.1.0.min.js"></script>
  <!-- 3. 导入bootstrap的js文件 -->
  <script src="js/bootstrap.min.js"></script>
  <script type="text/javascript">
  </script>
</head>
<body>
<div>${loginUser.name}欢迎您</div>
<div align="center">
  <a
          href="${pageContext.request.getContextPath()}/findUserByPage?currentPage=1&rows=5" style="text-decoration:none;font-size:33px">查询所有用户信息
  </a>
</div>
</body>
</html>

~~~

### login.jsp

~~~jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="utf-8"/>
    <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
    <meta name="viewport" content="width=device-width, initial-scale=1"/>
    <title>管理员登录</title>

    <!-- 1. 导入CSS的全局样式 -->
    <link href="css/bootstrap.min.css" rel="stylesheet">
    <!-- 2. jQuery导入，建议使用1.9以上的版本 -->
    <script src="js/jquery-2.1.0.min.js"></script>
    <!-- 3. 导入bootstrap的js文件 -->
    <script src="js/bootstrap.min.js"></script>
    <script type="text/javascript"></script>
	  <script type="text/javascript">
		  function refreshCode(){
			  //获图片对象
			  var vcode = document.getElementById("vcode");
			  //加上时间戳
			  vcode.src = "${pageContext.request.contextPath}/checkCode?time="+ new Date().getTime();
		  }
	  </script>
  </head>
  <body>
  	<div class="container" style="width: 400px;">
  		<h3 style="text-align: center;">管理员登录</h3>
        <form action="${pageContext.request.contextPath}/login" method="post">
	      <div class="form-group">
	        <label for="username">用户名：</label>
	        <input type="text" name="username" class="form-control" id="username" placeholder="请输入用户名"/>
	      </div>
	      
	      <div class="form-group">
	        <label for="password">密码：</label>
	        <input type="password" name="password" class="form-control" id="password" placeholder="请输入密码"/>
	      </div>
	      
	      <div class="form-inline">
	        <label for="vcode">验证码：</label>
	        <input type="text" name="verifycode" class="form-control" id="verifycode" placeholder="请输入验证码" style="width: 120px;"/>
	        <a href="javascript:refreshCode()">
				<img src="${pageContext.request.contextPath}/checkCode" title="看不清点击刷新" id="vcode"/></a>
	      </div>
	      <hr/>
	      <div class="form-group" style="text-align: center;">
	        <input class="btn btn btn-primary" type="submit" value="登录">
	       </div>
	  	</form>
		
		<!-- 出错显示的信息框 -->
	  	<div class="alert alert-warning alert-dismissible" role="alert">
		  <button type="button" class="close" data-dismiss="alert" >
		  	<span>&times;</span></button>
		   <strong>${error}</strong>
		</div>
  	</div>
  </body>
</html>
~~~

### list.jsp

~~~jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<!-- 网页使用的语言 -->
<html lang="zh-CN">
<head>
    <!-- 指定字符集 -->
    <meta charset="utf-8">
    <!-- 使用Edge最新的浏览器的渲染方式 -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <!-- viewport视口：网页可以根据设置的宽度自动进行适配，在浏览器的内部虚拟一个容器，容器的宽度与设备的宽度相同。
    width: 默认宽度与设备的宽度相同
    initial-scale: 初始的缩放比，为1:1 -->
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->
    <title>用户信息管理系统</title>

    <!-- 1. 导入CSS的全局样式 -->
    <link href="css/bootstrap.min.css" rel="stylesheet">
    <!-- 2. jQuery导入，建议使用1.9以上的版本 -->
    <script src="js/jquery-2.1.0.min.js"></script>
    <!-- 3. 导入bootstrap的js文件 -->
    <script src="js/bootstrap.min.js"></script>
    <style type="text/css">
        td, th {
            text-align: center;
        }
    </style>
    <script>
        function deleteUser(id){
            //用户安全提示
            if(confirm("您确定要删除吗？")){
                //访问路径
                location.href="${pageContext.request.contextPath}/delUser?id="+id;
            }
        }

        window.onload = function(){
            //给删除选中按钮添加单击事件
            document.getElementById("delSelected").onclick = function() {
                if (confirm("您确定要删除选中条目吗？")) {
                    var flag = false;
                    //判断是否有选中条目
                    var cbs = document.getElementsByName("uid");
                    for (var i = 0; i < cbs.length; i++) {
                        if (cbs[i].checked) {
                            //有一个条目选中了
                            flag = true;
                            break;
                        }
                    }
                    if (flag) {//有条目被选中
                        //表单提交
                        document.getElementById("form").submit();
                    }
                }
            }
            document.getElementById("firstCb").onclick = function(){
                //2.获取下边列表中所有的cb
                var cbs = document.getElementsByName("uid");
                //3.遍历
                for (var i = 0; i < cbs.length; i++) {
                    //4.设置这些cbs[i]的checked状态 = firstCb.checked
                    cbs[i].checked = this.checked;
                }
            }
        }
    </script>
</head>
<body>
<div>${loginUser.name}欢迎您</div>
<div class="container">
    <h3 style="text-align: center">用户信息列表</h3>
    <div style="float: left;">
        <form class="form-inline" action="${pageContext.request.contextPath}/findUserByPage" method="post">
            <div class="form-group">
                <label for="exampleInputName2">姓名</label>
                <input type="text" name="name" value="${pb.name}" class="form-control" id="exampleInputName2" >
            </div>
            <div class="form-group">
                <label for="exampleInputName3">籍贯</label>
                <input type="text" name="address" value="${pb.address}" class="form-control" id="exampleInputName3" >
            </div>

            <div class="form-group">
                <label for="exampleInputEmail2">邮箱</label>
                <input type="text" name="email" value="${pb.email}" class="form-control" id="exampleInputEmail2"  >
            </div>
            <button type="submit" class="btn btn-default">查询</button>
        </form>
    </div>
    <div style="float: right; margin: 5px">
        <td colspan="8" align="center"><a class="btn btn-primary" href="${pageContext.request.contextPath}/add.jsp">添加联系人</a></td>
        <td colspan="8" align="center"><a class="btn btn-primary" href="javascript:void(0);" id="delSelected">删除选中</a></td>
    </div>
    <form id="form" action="${pageContext.request.contextPath}/delSelected" method="post">
        <table border="1" class="table table-bordered table-hover">
            <tr class="success">
                <th><input type="checkbox" id="firstCb"></th>
                <th>编号</th>
                <th>姓名</th>
                <th>性别</th>
                <th>年龄</th>
                <th>籍贯</th>
                <th>QQ</th>
                <th>邮箱</th>
                <th>操作</th>
            </tr>
            <c:forEach items="${pb.users}" var="user" varStatus="s">
                <tr>
                    <th><input type="checkbox" name="uid" value="${user.id}"></th>
                    <td>${s.count}</td>
                    <td>${user.name}</td>
                    <td>${user.gender}</td>
                    <td>${user.age}</td>
                    <td>${user.address}</td>
                    <td>${user.qq}</td>
                    <td>${user.email}</td>
                    <td><a class="btn btn-default btn-sm" href="${pageContext.request.contextPath}/findUser?id=${user.id}">修改</a>&nbsp;
                        <a class="btn btn-default btn-sm" href="javascript:deleteUser(${user.id});">删除</a></td>
                </tr>
            </c:forEach>
        </table>
    </form>
    <div>
        <nav aria-label="Page navigation">
            <ul class="pagination">
                        <li>
                            <a href="${pageContext.request.contextPath}/findUserByPage?currentPage=${pb.currentPage - 1}&rows=5&name=${pb.name}&address=${pb.address}&email=${pb.email}" aria-label="Previous">
                                <span aria-hidden="true">&laquo;</span>
                            </a>
                        </li>


                <c:forEach begin="1" end="${pb.totalPage}" var="i" >


                    <c:if test="${pb.currentPage == i}">
                        <li class="active"><a href="${pageContext.request.contextPath}/findUserByPage?currentPage=${i}&rows=5&name=${pb.name}&address=${pb.address}&email=${pb.email}">${i}</a></li>
                    </c:if>
                    <c:if test="${pb.currentPage != i}">
                        <li><a href="${pageContext.request.contextPath}/findUserByPage?currentPage=${i}&rows=5&name=${pb.name}&address=${pb.address}&email=${pb.email}">${i}</a></li>
                    </c:if>

                </c:forEach>

                <li>
                    <a href="${pageContext.request.contextPath}/findUserByPageServlet?currentPage=${pb.currentPage + 1}&rows=5&name=${pb.name}&address=${pb.address}&email=${pb.email}" aria-label="Next">
                        <span aria-hidden="true">&raquo;</span>
                    </a>
                </li>
                <span style="font-size: 25px;margin-left: 5px;">
                    共${pb.totalCount}条记录，共${pb.totalPage}页
                </span>

            </ul>
        </nav>
    </div>
</div>
</body>
</html>

~~~

### add.jsp

~~~jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<!-- HTML5文档-->
<!DOCTYPE html>
<!-- 网页使用的语言 -->
<html lang="zh-CN">
<head>
    <!-- 指定字符集 -->
    <meta charset="utf-8">
    <!-- 使用Edge最新的浏览器的渲染方式 -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <!-- viewport视口：网页可以根据设置的宽度自动进行适配，在浏览器的内部虚拟一个容器，容器的宽度与设备的宽度相同。
    width: 默认宽度与设备的宽度相同
    initial-scale: 初始的缩放比，为1:1 -->
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->
    <title>添加用户</title>

    <!-- 1. 导入CSS的全局样式 -->
    <link href="css/bootstrap.min.css" rel="stylesheet">
    <!-- 2. jQuery导入，建议使用1.9以上的版本 -->
    <script src="js/jquery-2.1.0.min.js"></script>
    <!-- 3. 导入bootstrap的js文件 -->
    <script src="js/bootstrap.min.js"></script>
</head>
<body>
<div>${loginUser.name}欢迎您</div>
<div class="container">
    <center><h3>添加联系人页面</h3></center>
    <form action="${pageContext.request.contextPath}/addUser" method="post">
        <div class="form-group">
            <label for="name">姓名：</label>
            <input type="text" class="form-control" id="name" name="name" placeholder="请输入姓名">
        </div>

        <div class="form-group">
            <label>性别：</label>
            <input type="radio" name="sex" value="男" checked="checked"/>男
            <input type="radio" name="sex" value="女"/>女
        </div>

        <div class="form-group">
            <label for="age">年龄：</label>
            <input type="text" class="form-control" id="age" name="age" placeholder="请输入年龄">
        </div>

        <div class="form-group">
            <label for="address">籍贯：</label>
            <select name="address" class="form-control" id="address">
                <option value="广东">广东</option>
                <option value="广西">广西</option>
                <option value="湖南">湖南</option>
            </select>
        </div>

        <div class="form-group">
            <label for="qq">QQ：</label>
            <input type="text" class="form-control" id="qq" name="qq" placeholder="请输入QQ号码"/>
        </div>

        <div class="form-group">
            <label for="email">Email：</label>
            <input type="text" class="form-control" id="email" name="email" placeholder="请输入邮箱地址"/>
        </div>

        <div class="form-group" style="text-align: center">
            <input class="btn btn-primary" type="submit" value="提交" />
            <input class="btn btn-default" type="reset" value="重置" />
            <a class="btn btn-default" href="${pageContext.request.contextPath}/list.jsp" role="button">返回</a>
        </div>
    </form>
</div>
</body>
</html>
~~~

### update.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!-- 网页使用的语言 -->
<html lang="zh-CN">
    <head>
        <!-- 指定字符集 -->
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>修改用户</title>

        <link href="css/bootstrap.min.css" rel="stylesheet">
        <script src="js/jquery-2.1.0.min.js"></script>
        <script src="js/bootstrap.min.js"></script>
        
    </head>
    <body>
    <div>${loginUser.name}欢迎您</div>
        <div class="container" style="width: 400px;">
        <h3 style="text-align: center;">修改联系人</h3>
        <form action="${pageContext.request.contextPath}/update" method="post">
            <!--  隐藏域 提交id-->
            <input type="hidden" name="id" value="${u.id}">
          <div class="form-group">
            <label for="name">姓名：</label>
            <input type="text" class="form-control" id="name" name="name" value="${u.name}" placeholder="请输入姓名" />
          </div>

          <div class="form-group">
            <label>性别：</label>
              <c:if test="${u.gender == '男'}">
                  <input type="radio" name="gender" value="男" checked />男
                  <input type="radio" name="gender" value="女"  />女
              </c:if>

              <c:if test="${u.gender == '女'}">
                  <input type="radio" name="gender" value="男"  />男
                  <input type="radio" name="gender" value="女" checked  />女
              </c:if>
          </div>

          <div class="form-group">
            <label for="age">年龄：</label>
            <input type="text" class="form-control" id="age" value="${u.age}" name="age" placeholder="请输入年龄" />
          </div>

          <div class="form-group">
            <label for="address">籍贯：</label>
             <select name="address" id="address" class="form-control" >
                 <c:if test="${u.address == '陕西'}">
                     <option value="陕西" selected>陕西</option>
                     <option value="北京">北京</option>
                     <option value="上海">上海</option>
                 </c:if>

                 <c:if test="${u.address == '北京'}">
                     <option value="陕西" >陕西</option>
                     <option value="北京" selected>北京</option>
                     <option value="上海">上海</option>
                 </c:if>

                 <c:if test="${u.address == '上海'}">
                     <option value="陕西" >陕西</option>
                     <option value="北京">北京</option>
                     <option value="上海" selected>上海</option>
                 </c:if>
            </select>
          </div>

          <div class="form-group">
            <label for="qq">QQ：</label>
            <input type="text" class="form-control" id="qq" value="${u.qq}" name="qq" placeholder="请输入QQ号码"/>
          </div>

          <div class="form-group">
            <label for="email">Email：</label>
            <input type="text" class="form-control" id="email" value="${u.email}" name="email" placeholder="请输入邮箱地址"/>
          </div>

             <div class="form-group" style="text-align: center">
                <input class="btn btn-primary" type="submit" value="提交" />
                <input class="btn btn-default" type="reset" value="重置" />
                 <a class="btn btn-default" href="${pageContext.request.contextPath}/findUserByPage" role="button">返回</a>
             </div>
        </form>
        </div>
    </body>
</html>
```