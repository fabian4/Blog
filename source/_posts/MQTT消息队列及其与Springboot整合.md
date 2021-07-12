---
title: MQTT及其与Springboot整合
date: 2020/9/5
description: MQTT及其与Springboot整合
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/H8l7S3cNtnXPFoL.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/H8l7S3cNtnXPFoL.jpg'
categories:
  - Spring
tags:
  - SpringBoot
  - MQTT
  - 阿里云
abbrlink: 200
---

# MQTT及其与Springboot整合

## 一、MQTT协议

### 1. 什么是MQTT

**MQTT**(**消息队列遥测传输**)是ISO 标准(ISO/IEC PRF 20922)下基于发布/订阅范式的消息协议。它工作在 TCP/IP协议族上，是为硬件性能低下的远程设备以及网络状况糟糕的情况下而设计的发布/订阅型消息协议，为此，它需要一个消息中间件。

MQTT是一个基于客户端-服务器的消息发布/订阅传输协议。MQTT协议是轻量、简单、开放和易于实现的，这些特点使它适用范围非常广泛。在很多情况下，包括受限的环境中，如：机器与机器（M2M）通信和物联网（IoT）。其在，通过卫星链路通信传感器、偶尔拨号的医疗设备、智能家居、及一些小型化设备中已广泛使用。

### 2. MQTT协议的特点

MQTT协议是为大量计算能力有限，且工作在低带宽、不可靠的网络的远程传感器和控制设备通讯而设计的协议，它具有以下主要的几项特性：

1. 使用发布/订阅消息模式，提供一对多的消息发布，解除应用程序耦合；
2. 对负载内容屏蔽的消息传输；
3. 使用 TCP/IP 提供网络连接；
4. 有三种消息发布服务质量：
   - **“至多一次”**，消息发布完全依赖底层 TCP/IP 网络。会发生消息丢失或重复。这一级别可用于如下情况，环境传感器数据，丢失一次读记录无所谓，因为不久后还会有第二次发送。
   - **“至少一次”**，确保消息到达，但消息重复可能会发生。
   - **“只有一次”**，确保消息到达一次。这一级别可用于如下情况，在计费系统中，消息重复或丢失会导致不正确的结果。

5. 小型传输，开销很小（固定长度的头部是 2 字节），协议交换最小化，以降低网络流量；
6. 使用 Last Will 和 Testament 特性通知有关各方客户端异常中断的机制。

## 二、阿里云MQTT消息队列

> 由于自己搭建的MQTT服务器会出现各种不稳定的情况，这里我们使用阿里云为我们提供的MQTT消息队列
>
> [什么是微消息队列 MQTT 版](https://help.aliyun.com/document_detail/42419.html?spm=5176.10695662.1996646101.searchclickresult.34a77bd5ClRHgP)



这里是阿里云官方提供的服务：[微消息队列 MQTT 版](https://www.aliyun.com/product/mq4iot)

![image-20200909094417776](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/ZYBPbpMl3fHteRJ.png)

这里是购买链接：[微消息队列 for IoT（包年包月)](https://common-buy.aliyun.com/?spm=a2c4g.11174283.2.4.70c95f0eHWIAMw&commodityCode=onsMqtt&aly_as=BD88GDNB#/buy)

不过新用户好像是可以白嫖一个月的（快快快冲）

![image-20200909094647021](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/BSbQFDZIpULzrWR.png)

购买成功之后在控制台就能看到自己的实例

![image-20200909100135539](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/F2g6DXl19PdqYI4.png)

## 三、客户端的配置和调试

### 1. 控制台新建管理Group

![image-20200909100546574](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/1MYU9Jq3BPhgpNG.png)

### 2. 下载安装客户端 mqtt.fx

这里是[官网地址](http://mqttfx.jensd.de/index.php)

![image-20200909101008734](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/4sZlJSAn1ObqQMW.png)



### 3. 配置基本信息

![image-20200909102335877](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/dFGg9yfxzX7Swto.png)

### 1. Client ID

以 **XXXXXXX@@@XXXX**来自由名命，格式为：GroupID@@@设备ID

GroupID为自己刚刚在控制台配置的

### 2. 用户名密码

![image-20200909104307428](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/C5RADwIjVHtvzeW.png)

需要在控制台这边进行签名计算

### 3. Access Key 和 Secret Key 的获取

![image-20200909105108437](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/sfA8y2EedxjIq7z.png)



> 这个是阿里云平台的调用方式

### 4. 通信测试

**在控制台创建 topic**

![image-20200909105504390](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/KhfctrTGSU7qA9i.png)

**在客户端订阅 topic**

![image-20200909105645077](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/TnGqydi1F6sMcPB.png)

**控制台就可以查询到连接**

![image-20200909105752400](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/Qp1J6glAtNBOmuv.png)

**通讯测试**

![image-20200909110103468](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/oPCVf8F2ypK7ixn.png)

![image-20200909110128667](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/mfuleB6NATLMUgZ.png)

## 四、代码测试连接

**阿里云的官方的mqtt连接[demo](https://code.aliyun.com/aliware_mqtt/mqtt-demo/tree/master?spm=a2c4g.11186623.2.38.77e66fc6Tr55aS)**

将自己的mqtt的服务器地址和所有信息配置进去之后就可以进行调试

![image-20200909111833311](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/FEgxGzSof47X8Zn.png)

![image-20200909111852978](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/sQFrGkLnOqa8hPi.png)

## 四、Springboot整合

### 1. pom依赖导入

~~~xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.10</version>
</dependency>
<dependency>
    <groupId>org.eclipse.paho</groupId>
    <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
    <version>1.2.2</version>
</dependency>
<dependency>
    <groupId>com.aliyun.openservices</groupId>
    <artifactId>ons-client</artifactId>
    <version>1.7.9.Final</version>
</dependency>
~~~

### 2. 签名计算工具类编写

~~~java
import org.apache.commons.codec.binary.Base64;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.Charset;
import java.security.*;

public class Tools {

    /**
     * @param text 要签名的文本
     * @param secretKey 阿里云MQ secretKey
     * @return 加密后的字符串
     * @throws InvalidKeyException
     * @throws NoSuchAlgorithmException
     */
    public static String macSignature(String text,
                                      String secretKey) throws InvalidKeyException, NoSuchAlgorithmException {
        Charset charset = Charset.forName("UTF-8");
        String algorithm = "HmacSHA1";
        Mac mac = Mac.getInstance(algorithm);
        mac.init(new SecretKeySpec(secretKey.getBytes(charset), algorithm));
        byte[] bytes = mac.doFinal(text.getBytes(charset));
        return new String(Base64.encodeBase64(bytes), charset);
    }
}
~~~

### 3. 数据封装

~~~java
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;

import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

import static org.eclipse.paho.client.mqttv3.MqttConnectOptions.MQTT_VERSION_3_1_1;

public class ConnectionOptionWrapper {
    /**
     * 内部连接参数
     */
    private MqttConnectOptions mqttConnectOptions;
    /**
     * MQ4IOT 实例 ID，购买后控制台获取
     */
    private String instanceId;
    /**
     * 账号 accesskey，从账号系统控制台获取
     */
    private String accessKey;
    /**
     * 账号 secretKey，从账号系统控制台获取，仅在Signature鉴权模式下需要设置
     */
    private String secretKey;
    /**
     * MQ4IOT clientId，由业务系统分配，需要保证每个 tcp 连接都不一样，保证全局唯一，如果不同的客户端对象（tcp 连接）使用了相同的 clientId 会导致连接异常断开。
     * clientId 由两部分组成，格式为 GroupID@@@DeviceId，其中 groupId 在 MQ4IOT 控制台申请，DeviceId 由业务方自己设置，clientId 总长度不得超过64个字符。
     */
    private String clientId;

    /**
     * Signature 鉴权模式下构造方法
     *
     * @param instanceId MQ4IOT 实例 ID，购买后控制台获取
     * @param accessKey 账号 accesskey，从账号系统控制台获取
     * @param clientId MQ4IOT clientId，由业务系统分配
     * @param secretKey 账号 secretKey，从账号系统控制台获取
     */
    public ConnectionOptionWrapper(String instanceId, String accessKey, String secretKey,
                                   String clientId) throws NoSuchAlgorithmException, InvalidKeyException {
        this.instanceId = instanceId;
        this.accessKey = accessKey;
        this.secretKey = secretKey;
        this.clientId = clientId;
        mqttConnectOptions = new MqttConnectOptions();
        mqttConnectOptions.setUserName("Signature|" + accessKey + "|" + instanceId);
        mqttConnectOptions.setPassword(Tools.macSignature(clientId, secretKey).toCharArray());
        mqttConnectOptions.setCleanSession(true);
        mqttConnectOptions.setKeepAliveInterval(90);
        mqttConnectOptions.setAutomaticReconnect(true);
        mqttConnectOptions.setMqttVersion(MQTT_VERSION_3_1_1);
        mqttConnectOptions.setConnectionTimeout(5000);
    }

    public MqttConnectOptions getMqttConnectOptions() {
        return mqttConnectOptions;
    }

}
~~~

### 4. bean初始配置类

~~~java
import me.fabian4.yocotowx.util.ConnectionOptionWrapper;
import org.eclipse.paho.client.mqttv3.MqttClient;
import org.eclipse.paho.client.mqttv3.MqttException;
import org.eclipse.paho.client.mqttv3.persist.MemoryPersistence;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

@Configuration
@PropertySource(value = "classpath:application.yml")
public class MQTTConfig {

    @Value("${mqtt.instanceId}")
    String instanceId;

    @Value("${mqtt.endPoint}")
    String endPoint;

    @Value("${mqtt.accessKey}")
    String accessKey;

    @Value("${mqtt.secretKey}")
    String secretKey;

    @Value("${mqtt.clientId}")
    String clientId;

    @Value("${mqtt.parentTopic}")
    String parentTopic;

    @Value("${mqtt.qosLevel}")
    int qosLevel;

    @Bean
    // 加入springboot的IoC容器，每次项目启动自动连接mqtt服务器
    public MqttClient mqttClient() throws MqttException, InvalidKeyException, NoSuchAlgorithmException {
        ConnectionOptionWrapper connectionOptionWrapper = new ConnectionOptionWrapper(instanceId, accessKey, secretKey, clientId);
        MqttClient mqttClient = new MqttClient("tcp://" + endPoint + ":1883", clientId, new MemoryPersistence());
        mqttClient.setTimeToWait(5000);
        mqttClient.setCallback(new Callback());
        mqttClient.connect(connectionOptionWrapper.getMqttConnectOptions());
        mqttClient.subscribe(parentTopic, qosLevel);
        System.out.println("成功连接到MQTT服务器，订阅了"+parentTopic+"主题");
        return mqttClient;
    }
}
~~~

### 5. 回调方法重写

~~~java
package me.fabian4.yocotowx.Mqtt;

import lombok.extern.slf4j.Slf4j;
import org.eclipse.paho.client.mqttv3.IMqttDeliveryToken;
import org.eclipse.paho.client.mqttv3.MqttCallback;
import org.eclipse.paho.client.mqttv3.MqttMessage;

public class Callback implements MqttCallback {


    @Override
    public void deliveryComplete(IMqttDeliveryToken iMqttDeliveryToken) {
        System.out.println("发布消息成功");
    }

    @Override
    public void connectionLost(Throwable throwable) {
    }

    @Override
    public void messageArrived(String topic, MqttMessage message) throws Exception {
        System.out.println("收到来自 " + topic + " 的消息："+new String(message.getPayload()));
    }
}
~~~

### 6. 发布消息函数封装

```java
import lombok.extern.slf4j.Slf4j;
import org.eclipse.paho.client.mqttv3.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;


@Slf4j
@Component
public class MQTTConnect{

    @Value("${mqtt.qosLevel}")
    int qosLevel;

    @Value("${mqtt.parentTopic}")
    String parentTopic;

    @Autowired
    private MqttClient mqttClient;

    public void pub(String DeviceId, String msg) throws MqttException {
        MqttMessage mqttMessage = new MqttMessage();
        mqttMessage.setQos(qosLevel);
        mqttMessage.setPayload(msg.getBytes());
        System.out.println(mqttClient);
        MqttTopic mqttTopic = mqttClient.getTopic(this.parentTopic + "/p2p/" + DeviceId);
        MqttDeliveryToken token = mqttTopic.publish(mqttMessage);
        token.waitForCompletion();
    }
}
```

### 7. 配置文件

~~~yml
# MQTT配置
mqtt:
  # 实例 ID
  instanceId: mqtt-cn-xxxxxxx
  # 接入点地址
  endPoint: mqtt-cn-xxxxxxxx.mqtt.aliyuncs.com
  # 账号 accesskey
  accessKey: xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  # 账号 secretKey
  secretKey: xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
  # 客户端 ID
  clientId: GID_demo@@@master
  # 父主题
  parentTopic: demo
  # mqtt消息形式
  qosLevel: 2
~~~

### 8. 测试类测试

~~~java
import me.fabian4.yocotowx.Mqtt.MQTTConnect;
import org.eclipse.paho.client.mqttv3.MqttException;
import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;



@SpringBootTest
@RunWith(SpringRunner.class)
class YocotoWxApplicationTests {

    @Autowired
    MQTTConnect mc;

    @Test
    void contextLoads() throws MqttException {
        mc.pub("GID_demo@@@demo1", "test");
    }
}
~~~

### 9. 测试结果

![image-20200909115257809](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/CAt7hOGqxWFvlQk.png)

![image-20200909115403392](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/fVwxFSCYsOd9g6W.png)