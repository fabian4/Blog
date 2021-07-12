---
title: Redis分库监听实现延时任务
date: 2020/12/23
description: Redis分库监听实现延时任务
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/OD13ayiueEIXBPC.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/OD13ayiueEIXBPC.jpg'
categories:
  - Spring
tags:
  - SpringBoot
  - Redis
abbrlink: 63597
---

# Redis分库监听实现延时任务

## 一、Redis

Redis数据库有16个，分别是0-15，每个数据库用数字命名，而且每个数据库的连接密码都一样，redis只允许一个密码。数据库之间不能共享，并且基于单机才有，如果是集群，就没有数据库的概念了。

redis之所以分这么多个数据库，也是为了区分业务，不同的业务存放在不同的库，但是一个redis，一般是给一个项目用，项目内的不同业务，单独用一个库，这样不会相互有数据交叉。现在很多微服务项目，一个项目里有多个微服务，redis统一由团队管理，每个服务连接自己的库就可以了。

> redis的数据库个数，也是可以修改的，redis.conf里面设置了databases 16，默认是16个，0-15，如果要修改数据库个数，可以修改这个配置。

Redis不支持自定义数据库的名字，每个数据库都以编号命名，开发者必须自己记录哪些数据库存储了哪些数据。另外Redis也不支持为每个数据库设置不同的访问密码，所以一个客户端要么可以访问全部数据库，要么连一个数据库也没有权限访问。

最重要的一点是多个数据库之间并不是完全隔离的，比如FLUSHALL命令可以清空一个Redis实例中所有数据库中的数据。综上所述，这些数据库更像是一种命名空间，而不适宜存储不同应用程序的数据。比如可以使用0号数据库存储某个应用生产环境中的数据，使用1号数据库存储测试环境中的数据，但不适宜使用0号数据库存储A应用的数据而使用1号数据库B应用的数据，不同的应用应该使用不同的Redis实例存储数据。由于Redis非常轻量级，一个空Redis实例占用的内在只有1M左右，所以不用担心多个Redis实例会额外占用很多内存。 

## 二、监听机制

**配置文件默认是 #注释了的，改为notify-keyspace-events Ex  重启redis，记住指定redis.conf配置文件启动**

| 字符  | 发送通知                                                     |
| :---- | :----------------------------------------------------------- |
| K     | 键空间通知，所有通知以 **keyspace@** 为前缀，针对Key         |
| E     | 键事件通知，所有通知以 **keyevent@** 为前缀，针对event       |
| *g*   | *DEL 、 EXPIRE 、 RENAME 等类型无关的通用命令的通知*         |
| **$** | **字符串命令的通知**                                         |
| **l** | **列表命令的通知**                                           |
| **s** | **集合命令的通知**                                           |
| **h** | **哈希命令的通知**                                           |
| **z** | **有序集合命令的通知**                                       |
| *x*   | *过期事件：每当有过期键被删除时发送*                         |
| *e*   | *驱逐(evict)事件：每当有键因为 maxmemory 政策而被删除时发送* |
| A     | 参数 g$lshzxe 的别名，相当于是All                            |

## 三、环境搭建

### 引入 Maven 依赖

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.5.0</version>
</dependency>
~~~

### 配置 Redis

~~~java
@Configuration
public class RedisConfig {

    @Bean(name = "db0RedisTemplate")
    public RedisTemplate<String, Object> db0RedisTemplate() {
        return getRedisTemplate(db0Factory());
    }
    @Bean(name = "db1RedisTemplate")
    public RedisTemplate<String, Object> db1RedisTemplate() {
        return getRedisTemplate(db1Factory());
    }

    @Bean
    @Primary
    public LettuceConnectionFactory db0Factory(){
        return new LettuceConnectionFactory(getRedisConfig(0), getClientConfig());
    }

    @Bean
    public LettuceConnectionFactory db1Factory(){
        return new LettuceConnectionFactory(getRedisConfig(1), getClientConfig());
    }

    private RedisStandaloneConfiguration getRedisConfig(int dbNo) {
        RedisStandaloneConfiguration config = new RedisStandaloneConfiguration();
        config.setHostName("127.0.0.1");
        config.setPort(6379);
        config.setPassword("");
        config.setDatabase(dbNo);
        return config;
    }

    private LettuceClientConfiguration getClientConfig() {
        GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
        poolConfig.setMaxTotal(10);
        poolConfig.setMaxIdle(5);
        poolConfig.setMinIdle(2);
        poolConfig.setMaxWaitMillis(3000);
        return LettucePoolingClientConfiguration.builder().poolConfig(poolConfig).build();
    }

    public RedisTemplate<String, Object> getRedisTemplate(LettuceConnectionFactory factory) {
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(String.class);
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.setConnectionFactory(factory);
        return redisTemplate;
    }

    public StringRedisTemplate getStringRedisTemplate(LettuceConnectionFactory factory) {
        StringRedisTemplate redisTemplate = new StringRedisTemplate();
        redisTemplate.setConnectionFactory(factory);
        return redisTemplate;
    }

    @Bean
    RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory){
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        // 添加自定义的监听器
        // 这里只监听 0号数据库
        container.addMessageListener(new RedisMessageListener(container), new ChannelTopic("_keyevent@0_:expired"));
        container.setConnectionFactory(connectionFactory);
        return container;
    }

}
~~~

### 定义监听

~~~java
@Component
public class RedisMessageListener extends KeyExpirationEventMessageListener {

    public RedisMessageListener(RedisMessageListenerContainer listenerContainer){
        super(listenerContainer);
    }

    @Override
    public void onMessage(Message message, byte[] bytes) {
        String key = message.toString();
        System.out.println("过期的key为：" + key);
    }
}
~~~

## 四、测试

编写测试类

~~~java
@SpringBootTest(classes = SpringbootLogApplication.class)
class RedisTest {

    @Resource(name = "db0RedisTemplate")
    RedisTemplate<String, Object> redisTemplate0;

    @Test
    void contextLoads() {
        redisTemplate0.opsForValue().set("test1", "1", 5, TimeUnit.SECONDS);
        redisTemplate0.opsForValue().set("test2", "1", 10, TimeUnit.SECONDS);
        redisTemplate0.opsForValue().set("test3", "1", 15, TimeUnit.SECONDS);
        redisTemplate0.opsForValue().set("test4", "1", 20, TimeUnit.SECONDS);
        redisTemplate0.opsForValue().set("test5", "1", 25, TimeUnit.SECONDS);
        redisTemplate0.opsForValue().set("test6", "1", 30, TimeUnit.SECONDS);
        redisTemplate0.opsForValue().set("test7", "1", 35, TimeUnit.SECONDS);
        redisTemplate0.opsForValue().set("test8", "1", 40, TimeUnit.SECONDS);
        redisTemplate0.opsForValue().set("test9", "1", 45, TimeUnit.SECONDS);
    }
}
~~~

控制台效果

![image-20201225172643152](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/1rh8Pey9cXWasdq.png)