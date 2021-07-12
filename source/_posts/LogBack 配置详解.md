---
title: LogBack 配置详解
date: 2020/9/27
description: LogBack 配置详解
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/cG382ufspIoAeyT.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/cG382ufspIoAeyT.jpg'
categories:
  - Spring
tags:
  - LogBack
abbrlink: 29366
---

# LogBack 配置详解

## 一、LogBack简介以及SpringBoot的支持

logback是Java的开源框架，性能比log4j要好。是springboot自带的日志框架。该框架主要有3个模块：

- **logback-core**：核心代码块
- **logback-classic**：实现了slf4j的api，加入该依赖可以实现log4j的api。
- **logback-access**：访问模块与servlet容器集成提供通过http来访问日志的功能（也就是说不需要访问服务器，直接在网页上就可以访问日志文件）。

> SpringBoot使用 [Commons Logging](https://commons.apache.org/logging) 进行所有内部日志的记录，但默认配置也提供了对常用日志的支持，如 [Java Util Logging](https://docs.oracle.com/javase/8/docs/api//java/util/logging/package-summary.html)，[Log4J2](https://logging.apache.org/log4j/2.x/)，和[Logback](https://logback.qos.ch/). 每种logger都可以通过配置使用控制台或文件输出日志内容。
>
> [Logback](http://logback.qos.ch/)是log4j框架的作者开发的新一代日志框架，它效率更高、能够适应诸多的运行环境，同时天然支持SLF4J。

当你引入 `spring-boot-starter`时，该启动包就自动包含了 **LogBack** 所需要的依赖包

![image-20201126120652370](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/nPgmXTQKbRN5DZx.png)

启动时控制台日志

![](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/8HMo2qkYSyJsTcx.png)



## 二、SpringBoot支持的日志配置

> SpringBoot支持我们在系统配置文件中对日志进行简单的配置

### 1. 输出debug级别日志

~~~properties
debug = true
~~~

在配置文件中将debug开启后，就可以在控制台看到debug级别以及以上的日志信息了

### 2. 输出到本地文件

~~~properties
logging.file.name = log
logging.file.path = log
logging.logback.rollingpolicy.max-file-size = 10MB
logging.logback.rollingpolicy.max-history = 7
~~~

输出到项目目录下名为 **log** 的日志文件，只记录七天内的日志，日志文件的最大大小为 10MB

~~~properties
# 定义日志输出格式
logging.pattern.console=%yellow(%d{yyyy-MM-dd HH:mm:ss}) %-5p [%magenta(%t)] %blue(%file:%L) %green(%c):%cyan(%msg%n)
logging.pattern.file=%yellow(%d{yyyy-MM-dd HH:mm:ss}) %-5p [%magenta(%t)] %blue(%file:%L) %green(%c):%cyan(%msg%n)
~~~

|      |                             含义                             |
| ---- | :----------------------------------------------------------: |
| %m   |                     输出代码中指定的消息                     |
| %p   |        输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL         |
| %r   |          输出自应用启动到输出该log信息耗费的毫秒数           |
| %c   |             输出所属的类目，通常就是所在类的全名             |
| %t   |                  输出产生该日志事件的线程名                  |
| %n   |   输出一个回车换行符，Windows平台为“\r\n”，Unix平台为“\n”    |
| %d   | 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921 |
| %l   | 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10) |

### 3. 日志级别

~~~properties
logging.level.web = debug
logging.level.sql = debug
logging.level.root = debug
# 自定义组，统一规定日志级别
logging.group.tomcat = org.apache.catalina, org.apache.coyote, org.apache.tomcat
logging.level.tomcat = debug
~~~

### 4. 自定义配置文件

由于日志服务一般都在ApplicationContext创建前就初始化了，它并不是必须通过Spring的配置文件控制。因此通过系统属性和传统的Spring Boot外部配置文件依然可以很好的支持日志控制和管理。

 根据不同的日志系统，按照指定的规则组织配置文件名，并放在 resources 目录下，就能自动被 spring boot 加载：

- Logback：logback-spring.xml, logback-spring.groovy, logback.xml, logback.groovy
- Log4j: log4j-spring.properties, log4j-spring.xml, log4j.properties, log4j.xml
- Log4j2: log4j2-spring.xml, log4j2.xml
- JDK (Java Util Logging)： logging.properties

> SpringBoot官方推荐使用带有`-spring`的文件名作为配置，如`logback-spring.xml`而不是`logback.xml`。
>
> 这样命名的好处在于：因为标准的`logback.xml`配置文件加载得太早，所以不能在其中使用扩展，需要使用`logback-spring.xml`。

~~~properties
# 指定你自定义配置文件的名称和位置
logging.config=classpath:logging-config.xml
~~~



## 三、自定义日志配置

### 1. 根节点`<configuration>`

- **scan**: 当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
- **scanPeriod**: 设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
- **debug**：当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

~~~xml
<configuration scan="true" scanPeriod="60 seconds" debug="false"> 

　　  <!--其他配置省略--> 

</configuration>　
~~~

### 2. 子节点 `<property>`

- **name**：变量名
- **value**：变量值

在下面就可以用 **${name}** 取到自己定义的 value 值

~~~xml
<property name="path" value="D:/log"/>
<property name="maxHistory" value="30"/>
<property name="maxFileSize" value="50MB"/>
<property name="console_pattern" value="%yellow(%d{yyyy-MM-dd HH:mm:ss}) %-5p [%magenta(%t)] %blue(%file:%L) %green(%c):%cyan(%msg%n)"/>
<property name="file_pattern" value="%date %level [%thread] %logger{36} [%file : %line] %msg%n"/>
~~~

### 3. 子节点`<appender>`

- **name**：自定义名称
- **class**：全限定类名
  - **ConsoleAppender**：把日志输出到控制台
  - **FileAppender**：把日志添加到文件
  - **RollingFileAppender**：滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件

#### **ConsoleAppender**

- 子节点`<encoder>`：对日志进行格式化
- 子节点`<filter>`：对日志等级进行过滤

~~~xml
<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
    <!-- 对debug级别以上日志进行过滤 -->
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
        <level>debug</level>
    </filter>
    <!-- 设置输出格式 -->
    <encoder>
        <pattern>${console_pattern}</pattern>
    </encoder>
    <target>System.err</target>
</appender>
~~~

#### **FileAppender**

- 子节点`<file>`：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
- 子节点`<append>`：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。
- 子节点`<encoder>`：对记录事件进行格式化
- 子节点`<prudent>`：如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false。

~~~xml
<appender name="file" class="ch.qos.logback.core.FileAppender">
    <file>${path}/log.log</file>
    <encoder>
        <pattern>${file_pattern}</pattern>
    </encoder>
    <append>true</append>
    <prudent>false</prudent>
</appender>
~~~

#### RollingFileAppender

- 子节点`<file>`：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。

- 子节点`<append>`：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。

- 子节点`<encoder>`：对记录事件进行格式化

- 子节点`<rollingPolicy>`：当发生滚动时，决定**RollingFileAppender**的行为，涉及文件移动和重命名。

  > 属性class定义具体的滚动策略类class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy，是最受欢迎的滚动政策，例如按天或按月。TimeBasedRollingPolicy承担翻滚责任以及触发所述翻转的责任。TimeBasedRollingPolicy支持自动文件压缩。

- 子节点`<triggeringPolicy>`：告知 **RollingFileAppender** 何时激活滚动。

- 子节点`<prudent>`： 当为true时，不支持FixedWindowRollingPolicy。支持TimeBasedRollingPolicy，但是有两个限制：不支持也不允许文件压缩，不能设置file属性必须留空。

##### rollingPolicy

- **TimeBasedRollingPolicy ：** 常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责触发滚动。

  - 子节点`<fileNamePattern>` ：必要节点，包含文件名及“%d”转换符， “%d”可以包含一个`Java.text.SimpleDateFormat`指定的时间格式，如：%d{yyyy-MM}。如果直接使用 %d，默认格式是 yyyy-MM-dd。**RollingFileAppender** 的file字节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），活动文件的名字不会改变；如果没设置file，活动文件的名字会根据**fileNamePattern** 的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。

  - 子节点`<maxHistory>` : 可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。

    > 假设设置每个月滚动，且`<maxHistory>`是6，则只保存最近6个月的文件，删除之前的旧文件。注意，删除旧文件是，那些为了归档而创建的目录也会被删除。
    >
  
- **FixedWindowRollingPolicy ：** 根据固定窗口算法重命名文件的滚动策略。

  - 子节点`<minIndex>` : 窗口索引最小值。
  - 子节点`<maxIndex>` : 窗口索引最大值，当用户指定的窗口过大时，会自动将窗口设置为12。
  - 子节点`<fileNamePattern>` : 必须包含“%i”例如，假设最小值和最大值分别为1和2，命名模式为 mylog%i.log,会产生归档文件mylog1.log和mylog2.log。还可以指定文件压缩选项，例如，mylog%i.log.gz 或者 没有log%i.log.zip。

##### **triggeringPolicy ** 

- **SizeBasedTriggeringPolicy ：** 查看当前活动文件的大小，如果超过指定大小会告知**RollingFileAppender**触发当前活动文件滚动。
  - 子节点`<maxFileSize>` : 这是活动文件的大小，默认值是10MB。

~~~xml
<appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${path}/log.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!-- 每天一归档 -->
        <fileNamePattern>${path}/log.log.%d{yyyy-MM-dd}-%i.zip</fileNamePattern>
        <maxHistory>${maxHistory}</maxHistory>
        <timeBasedFileNamingAndTriggeringPolicy
                                                class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
            <maxFileSize>${maxFileSize}</maxFileSize>
        </timeBasedFileNamingAndTriggeringPolicy>
    </rollingPolicy>
    <encoder>
        <pattern>${file_pattern}</pattern>
    </encoder>
</appender>
~~~

#### 子节点`<filter>`

过滤器，执行一个过滤器会有返回个枚举值，即DENY，NEUTRAL，ACCEPT其中之一。返回DENY，日志将立即被抛弃不再经过其他过滤器；返回NEUTRAL，有序列表里的下个过滤器过接着处理日志；返回ACCEPT，日志会被立即处理，不再经过剩余过滤器。过滤器被添加到`<appender>` 中，为`<appender>` 添加一个或多个过滤器后，可以用任意条件对日志进行过滤。`<appender>` 有多个过滤器时，按照配置顺序执行。

- **LevelFilter**  ： 级别过滤器，根据日志级别进行过滤。如果日志级别等于配置级别，过滤器会根据 onMath 和 onMismatch 接收或拒绝日志。

  - 子节点`<level>` : 设置过滤级别。

  - 子节点`<onMatch>` : 用于配置符合过滤条件的操作。

  - 子节点`<onMismatch>` : 用于配置不符合过滤条件的操作。

    > 例如：将过滤器的日志级别配置为info，所有info级别的日志交给appender处理，非info级别的日志，被过滤掉。

  ~~~xml
   <!-- 表示打印到控制台 -->
  <appender name="limeFlogger" class="ch.qos.logback.core.ConsoleAppender">
      <filter class="ch.qos.logback.classic.filter.LevelFilter">
          <level>info</level>
          <onMatch>ACCEPT</onMatch>
          <onMismatch>DENY</onMismatch>
      </filter>
      <!-- encoder 默认配置为PatternLayoutEncoder --> 
      <encoder>
          <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
      </encoder>
      <target>System.err</target>
  </appender>
  
  <logger name="limeLogback.LogbackDemo" level="debug" additivity="true">
      <appender-ref ref="limeFlogger"/>
  </logger>
  ~~~

- **ThresholdFilter** ：临界值过滤器，过滤掉低于指定临界值的日志。当日志级别等于或高于临界值时，过滤器返回NEUTRAL；当日志级别低于临界值时，日志会被拒绝。

  - 子节点`<level>` : 设置过滤级别。

  ~~~xml
   <!-- 表示打印到控制台 -->
  <appender name="limeFlogger" class="ch.qos.logback.core.ConsoleAppender">
      <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
          <level>info</level>
      </filter>
      <!-- encoder 默认配置为PatternLayoutEncoder --> 
      <encoder>
          <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
      </encoder>
      <target>System.err</target>
  </appender>
  
  <logger name="limeLogback.LogbackDemo" level="debug" additivity="true">
      <appender-ref ref="limeFlogger"/>
  </logger>
  ~~~


### 4. 子节点`<logger>`
用来设置某一个包或具体的某一个类的日志打印级别、以及指定`<appender>`。仅有一个name属性，一个可选的level和一个可选的addtivity属性。可以包含零个或多个`<appender-ref>`元素，标识这个appender将会添加到这个logger。

- **name**: 用来指定受此loger约束的某一个包或者具体的某一个类。
- **level**: 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL和OFF，还有一个特殊值INHERITED或者同义词NULL，代表强制执行上级的级别。 如果未设置此属性，那么当前loger将会继承上级的级别。
- **addtivity**: 是否向上级logger传递打印信息。默认是true。可以包含零个或多个`<appender-ref>`元素，标识这个appender将会添加到这个logger。

~~~xml
<!-- show parameters for hibernate sql 专为 Hibernate 定制 -->
<logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE" />
<logger name="org.hibernate.type.descriptor.sql.BasicExtractor" level="DEBUG" />
<logger name="org.hibernate.SQL" level="DEBUG" />
<logger name="org.hibernate.engine.QueryParameters" level="DEBUG" />
<logger name="org.hibernate.engine.query.HQLQueryPlan" level="DEBUG" />

<!--myibatis log configure-->
<logger name="com.apache.ibatis" level="TRACE"/>
<logger name="java.sql.Connection" level="DEBUG"/>
<logger name="java.sql.Statement" level="DEBUG"/>
<logger name="java.sql.PreparedStatement" level="DEBUG"/>
~~~



### 5. 子节点`<root>`

它也是`<logger>`元素，但是它是根loger,是所有`<loger>`的上级。只有一个level属性，因为name已经被命名为"root",且已经是最上级了。

- **level**: 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL和OFF，不能设置为INHERITED或者同义词NULL。 默认是DEBUG。

> 同`<logger>`一样，可以包含零个或多个`<appender-ref>`元素，标识这个appender将会添加到这个logger。

~~~xml
<root>
    <level value="info"/>
    <appender-ref ref="console"/>
    <appender-ref ref="debug_file"/>
    <appender-ref ref="info_file"/>
    <appender-ref ref="warn_file"/>
    <appender-ref ref="error_file"/>
</root>
~~~

## 四、配置示例

~~~xml
<configuration debug="false" scan="true" scanPeriod="10 seconds">
    <!--输出sql语句-->
    <logger name="me.fabian4.yocotoadmin.mapper" level="debug"/>

    <!--定义一些常用量-->
    <property name="path" value="D:/log"/>
    <property name="maxHistory" value="30"/>
    <property name="maxFileSize" value="50MB"/>

    <!--控制台输出-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>debug</level>
        </filter>
        <encoder>
            <pattern>%yellow(%d{yyyy-MM-dd HH:mm:ss}) %-5p [%magenta(%t)] %blue(%file:%L) %green(%c):%cyan(%msg%n)</pattern>
        </encoder>
    </appender>


    <!--日志文件分级输出-->
    <!--debug日志-->
    <appender name="debug_file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${path}/debug/debug.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天一归档 -->
            <fileNamePattern>${path}/debug/%d{yyyy-MM-dd}-%i.log</fileNamePattern>
            <maxHistory>${maxHistory}</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>${maxFileSize}</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>%date %level [%thread] %logger{36} [%file : %line] %msg%n
            </pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>DEBUG</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--info日志-->
    <appender name="info_file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${path}/info/info.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天一归档 -->
            <fileNamePattern>${path}/info/%d{yyyy-MM-dd}-%i.log</fileNamePattern>
            <maxHistory>${maxHistory}</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>${maxFileSize}</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>%date %level [%thread] %logger{36} [%file : %line] %msg%n
            </pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--warn日志-->
    <appender name="warn_file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${path}/warn/warn.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天一归档 -->
            <fileNamePattern>${path}/warn/%d{yyyy-MM-dd}-%i.log</fileNamePattern>
            <maxHistory>${maxHistory}</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>${maxFileSize}</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>%date %level [%thread] %logger{36} [%file : %line] %msg%n
            </pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>WARN</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--error日志-->
    <appender name="error_file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${path}/error/error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天一归档 -->
            <fileNamePattern>${path}/error/%d{yyyy-MM-dd}-%i.log</fileNamePattern>
            <maxHistory>${maxHistory}</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>${maxFileSize}</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>%date %level [%thread] %logger{36} [%file : %line] %msg%n
            </pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--分别设置对应的日志输出节点 -->
    <root>
        <level value="info"/>
        <appender-ref ref="console"/>
        <!--生产环境时记录到文件中-->
        <springProfile name="prod">
            <appender-ref ref="debug_file"/>
            <appender-ref ref="info_file"/>
            <appender-ref ref="warn_file"/>
            <appender-ref ref="error_file"/>
        </springProfile>
    </root>
</configuration>
~~~

