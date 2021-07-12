---
title: SpringBoot 整合 Swagger 实现在线接口文档
date: 2020/10/20
description: SpringBoot 整合 Swagger 实现在线接口文档
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/6iC7EGK5UMHPr3V.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/6iC7EGK5UMHPr3V.jpg'
categories:
  - Spring
tags:
  - SpringBoot
  - Swagger
abbrlink: 56486
---

# SpringBoot 整合 Swagger 实现在线接口文档

## 一、Swagger

> 相信无论是前端还是后端开发，都或多或少地被接口文档折磨过。前端经常抱怨后端给的接口文档与实际情况不一致。后端又觉得编写及维护接口文档会耗费不少精力，经常来不及更新。其实无论是前端调用后端，还是后端调用后端，都期望有一个好的接口文档。但是这个接口文档对于程序员来说，就跟注释一样，经常会抱怨别人写的代码没有写注释，然而自己写起代码起来，最讨厌的，也是写注释。所以仅仅只通过强制来规范大家是不够的，随着时间推移，版本迭代，接口文档往往很容易就跟不上代码了

最受欢迎的API文档规范之一是OpenApi，以前称为Swagger。它允许您使用JSON或YAML元数据描述API的属性。它还提供了一个Web UI，它可以将元数据转换为一个很好的HTML文档。此外，通过该UI，您不仅可以浏览有关API端点的信息，还可以将UI用作REST客户端 可以调用任何端接口，指定要发送的数据并检查响应。它非常方便。

然而，手动编写此类文档并在代码更改时保持更新是不现实的。这就是SpringFox发挥作用的地方。它是Spring Framework的Swagger集成。它可以自动检查您的类，检测控制器，它们的方法，它们使用的模型类以及它们映射到的URL。没有任何手写文档，只需检查应用程序中的类，它就可以生成大量有关API的信息。最重要的是，每当你进行更改时，它们都会反映在文档中。

Swagger [官网地址](https://swagger.io/)

## 二、引入依赖，环境搭建

~~~xml
<!-- swagger依赖 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
~~~

~~~java
// 项目配置类
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build();
    }
}
~~~

~~~java
@RestController
public class Controller {

    @GetMapping("/1")
    public String test1(String text){
        return text;
    }

    @GetMapping("/2")
    public String test2(String text){
        return text;
    }

    @GetMapping("/3")
    public String test3(String text){
        return text;
    }

    @GetMapping("/4")
    public String test4(String text){
        return text;
    }

}
~~~

运行项目访问 http://localhost:8000/swagger-ui.html 就可以看到我们的接口文档了

![](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/Cl1gcKdrMAnjyqB.png)

## 三、一些 Swagger 注解

- **@Api()**用于类：表示标识这个类是swagger的资源
  - tags–表示说明 
  - value–也是说明，可以使用tags替代 但是tags如果有多个值，会生成多个list
- **@ApiOperation()**用于方法：表示一个http请求的操作 
  - value用于方法描述 
  - notes用于提示内容 
  - tags可以重新分组（视情况而用） 
-  **@ApiParam()**用于方法，参数，字段说明：表示对参数的添加元数据（说明或是否必填等） 
  - name–参数名 
  - value–参数说明 
  - required–是否必填
-  **@ApiModel()**用于类：表示对类进行说明，用于参数用实体类接收 
  - value–表示对象名 
  - description–描述 
- **@ApiModelProperty()**用于方法，字段 ：表示对model属性的说明或者数据操作更改 
  - value–字段说明 
  - name–重写属性名字 
  - dataType–重写属性类型 
  - required–是否必填 
  - example–举例说明 
  - hidden–隐藏
- **@ApiIgnore()** 用于类，方法，方法参数 ：表示这个方法或者类被忽略 
- **@ApiImplicitParam()** 用于方法 ：表示单独的请求参数 
- **@ApiImplicitParams()** 用于方法，包含多个 @ApiImplicitParam
  - name–参数名
  - value–参数说明 
  - dataType–数据类型 
  - paramType–参数类型 
  - example–举例说明

## 四、接口分组

在我们对接口进行权限验证时，有的接口就需要某些权限，有点则直接开放，这里我们对接口进行分组

### 1. 配置类

~~~java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    // 获取配置文件的参数
    // 这里我们是使用JWT的token鉴权方式
    @Value("${jwt.header}")
    private String tokenHeader;

    @Value("${jwt.token-start-with}")
    private String tokenStartWith;

    @Value("${swagger.enabled}")
    private Boolean enabled;

    @Bean
    public Docket privateApi() {
        List<Parameter> pars = new ArrayList<>();
        ParameterBuilder tokenPar = new ParameterBuilder();
        tokenPar.name(tokenHeader).description("token")
                .modelRef(new ModelRef("String"))
                .parameterType("header")
                .defaultValue(tokenStartWith + " ")
                .required(true)
                .build();
        pars.add(tokenPar.build());
        return new Docket(DocumentationType.SWAGGER_2)
                .enable(enabled)
                .select()
            	// 这里是通过注解来检索接口 可以设为我们自定义的注解
                .apis(RequestHandlerSelectors.withMethodAnnotation(PrivateApi.class))
                .paths(PathSelectors.any())
                .build()
                .groupName("保护接口");
    }

    @Bean
    public Docket publicApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .enable(enabled)
                .select()
                .apis(RequestHandlerSelectors.withMethodAnnotation(PublicApi.class))
                .paths(PathSelectors.any())
                .build()
                .groupName("开放接口");
    }
}
~~~

~~~java
// 自定义注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PrivateApi {
}
~~~

~~~java
// 自定义注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PublicApi {
}
~~~

### 2. 接口编写

~~~java
@RestController
@Api(tags = "xxx管理")
@RequiredArgsConstructor
@RequestMapping("/test")
public class ArticleController {

    @PrivateApi
    @GetMapping()
    @PreAuthorize("hasAnyAuthority('root', 'admin')")
    @ApiOperation(value = "查询xxxx", notes = "需要管理员权限")
    public GenericResponse findAll(){
        return GenericResponse.response(ResStatus.NORMAL, "xxxxx");
    }
    
    @GetMapping("/find")
    @PublicApi
    @ApiOperation(value = "查询xxxx")
    public GenericResponse findAll(){
        return GenericResponse.response(ResStatus.NORMAL, "xxxxxx");
    }
}
~~~

这是就可以在页面上看到分组和具体接口参数什么

还可以测试接口，类似实现postman

![ ](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/b46hFcDeaZO2RK3.png)