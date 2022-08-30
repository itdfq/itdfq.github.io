---
title: RabbitMq延迟消费（TTL实现）
top: false
cover: false
toc: true
mathjax: true
date: 2021-12-10 14:27:54
password:
summary: Rabbit延迟队列
tags:
- 原创
- RabbitMQ
categories:
- 队列
---

## 延时消费
RabbitMQ本身并不提供延迟队列的功能，但是我们仍然可以使用RabbitMQ的 TTL（Time-To-Live） 和 DLX（Dead Letter Exchanges） 这两个扩展特性来实现延迟队列，实现消息的延迟消费和延迟重试的功能。

### 实现结果

 - 固定时间延迟消费

	![固定时间延迟](https://img-blog.csdnimg.cn/21ac266ad1ba434faab5350fc3c9447a.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBASVRkZnE=,size_20,color_FFFFFF,t_70,g_se,x_16)

 - 指定时间消费
 	![指定时间消费](https://img-blog.csdnimg.cn/1c1173fa6aa34723b1909839d5b0ae06.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBASVRkZnE=,size_20,color_FFFFFF,t_70,g_se,x_16)


### 具体实现

 #### 连接配置
 

```java
package com.itdfq.delay.config;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitAdmin;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Author: QianMo
 * @Date: 2021/10/19 18:34
 * @Description:
 */
@Configuration
public class RabbitConfig {

    @Value("${spring.rabbitmq.addresses}")
    private String address;
    @Value("${spring.rabbitmq.port}")
    private String port;
    @Value("${spring.rabbitmq.username}")
    private String username;
    @Value("${spring.rabbitmq.password}")
    private String password;
    @Value("${spring.rabbitmq.virtual-host}")
    private String virtualHost;


    //连接工厂
    @Bean
    public ConnectionFactory connectionFactory() {
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
        connectionFactory.setAddresses(address + ":" + port);
        connectionFactory.setUsername(username);
        connectionFactory.setPassword(password);
        connectionFactory.setVirtualHost(virtualHost);
        //TODO  消息发送确认--回调
//        connectionFactory.setPublisherConfirms(true);
        return connectionFactory;

    }

    //RabbitAdmin类封装对RabbitMQ的管理操作
    @Bean
    public RabbitAdmin rabbitAdmin(ConnectionFactory connectionFactory) {
        return new RabbitAdmin(connectionFactory);
    }

    //使用Template
    @Bean
    public RabbitTemplate newRabbitTemplate() {
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory());
        //设置监听确认mq（交换器）接受到信息
        rabbitTemplate.setConfirmCallback(confirmCallback());
        //添加监听 失败鉴定（路由没有收到）
        rabbitTemplate.setMandatory(true);
        rabbitTemplate.setReturnCallback(returnCallback());
        return rabbitTemplate;
    }


    //****************生产者发送确认********************
    @Bean
    public RabbitTemplate.ConfirmCallback confirmCallback() {
        return new RabbitTemplate.ConfirmCallback() {
            @Override
            public void confirm(CorrelationData correlationData, boolean b, String s) {
                if (b) {
                    System.out.println("发送者确认发送给mq成功");
                } else {
                    System.out.println("发送者发送失败，考虑重发" + s);
                }
            }
        };
    }

    //****************失败通知********************
    //失败才通知，成功不通知
    @Bean
    public RabbitTemplate.ReturnCallback returnCallback() {
        return new RabbitTemplate.ReturnCallback() {
            @Override
            public void returnedMessage(Message message, int i, String replayText, String exchange, String rountingKey) {
                System.out.println("无效路由信息，需要考虑另外处理");
                System.out.println("Returned replayText:" + replayText);
                System.out.println("Returned exchange:" + exchange);
                System.out.println("Returned rountingKey:" + rountingKey);
                String s = new String(message.getBody());
                System.out.println("Returned Message:" + s);
            }
        };
    }


}

```

 #### 设置延时队列

```java
  /**
     * 延时队列
     *
     *
     * @return
     */
    @Bean
    public Queue DelayQueue() {
        Map<String, Object> params = new HashMap<>();
        // x-dead-letter-exchange 声明了队列里的死信转发到的DLX名称，
        params.put("x-dead-letter-exchange", RabbitMqConstant.IMMEDIATE_EXCHANGE);
        // x-dead-letter-routing-key 声明了这些死信在转发时携带的 routing-key 名称。
        params.put("x-dead-letter-routing-key", RabbitMqConstant.IMMEDIATE_ROUTING_KEY);
        // x-message-ttl 声明该队列死信可存活时间
        params.put("x-message-ttl", RabbitMqConstant.DELAY_TIME);
        return new Queue(RabbitMqConstant.DELAY_QUEUE, true, false, false, params);
    }
```
#### 设置立即消费监听

```java
package com.itdfq.delay.message.listen;

import com.itdfq.delay.constant.RabbitMqConstant;
import org.springframework.amqp.core.AcknowledgeMode;
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitAdmin;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Author: QianMo
 * @Date: 2021/10/21 10:29
 * @Description:
 */
@Configuration
public class DelayListenConfig {


    @Autowired
    private RabbitAdmin rabbitAdmin;

    @Autowired
    private ConnectionFactory connectionFactory;


    /********************立即消费配置**********************/

    @Bean
    public DirectExchange Exchange() {
        DirectExchange exchange = new DirectExchange(
                RabbitMqConstant.IMMEDIATE_EXCHANGE, true, false);
        exchange.setAdminsThatShouldDeclare(rabbitAdmin);
        return exchange;
    }

    @Bean
    public Queue Queue() {
        Queue queue = new Queue(RabbitMqConstant.IMMEDIATE_QUEUE, true, false, false);
        queue.setAdminsThatShouldDeclare(rabbitAdmin);
        return queue;
    }

    @Bean
    public Binding subscribeNotifyBinding() {
        Binding binding = BindingBuilder.bind(Queue()).to(Exchange())
                .with(RabbitMqConstant.IMMEDIATE_ROUTING_KEY);
        binding.setAdminsThatShouldDeclare(rabbitAdmin);
        return binding;
    }


    @Bean
    public SimpleMessageListenerContainer container(
            @Qualifier(value = "delayRabbitmqListener") DelayRabbitmqListener delayRabbitmqListener) {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        container.setQueues(Queue());
        container.setMessageListener(delayRabbitmqListener);
        container.setDefaultRequeueRejected(false);
        //手动提交
        container.setAcknowledgeMode(AcknowledgeMode.MANUAL);
        //设置消费者ack消息的模式，默认是自动，此处设置为手动
//        container.setAcknowledgeMode(AcknowledgeMode.MANUAL);
        return container;
    }

}

```

---
### 指定时间（延时消费）

 ##### 设置延时队列

```java
   /**
     * 延时队列，可变延时
     *
     * @return
     */
    @Bean
    public Queue variableDelayQueue() {
        Map<String, Object> params = new HashMap<>();
        // x-dead-letter-exchange 声明了队列里的死信转发到的DLX名称，
        params.put("x-dead-letter-exchange", RabbitMqConstant.IMMEDIATE_EXCHANGE);
        // x-dead-letter-routing-key 声明了这些死信在转发时携带的 routing-key 名称。
        params.put("x-dead-letter-routing-key", RabbitMqConstant.IMMEDIATE_ROUTING_KEY);
        return new Queue(RabbitMqConstant.DELAY_VARIABLE_QUEUE_KEY, true, false, false, params);
    }
```


 ##### 消费者

```java
 /**
     * 可变延时消费
     * @param msg  消息
     * @param expiration 延时时间单位 ： 毫秒
     */
    public void send(String msg,Integer expiration){
        rabbitTemplate.convertAndSend(RabbitMqConstant.DELAY_VARIABLE_EXCHANGE_KEY,
                RabbitMqConstant.DELAY_VARIABLE_ROUTING_KEY, msg,
                message -> {
                    log.info("可变延时消费发送消息: {}, and expiration in {}ms", msg, expiration);
                    message.getMessageProperties().setExpiration(expiration.toString());
                    return message;
                });
    }
```
