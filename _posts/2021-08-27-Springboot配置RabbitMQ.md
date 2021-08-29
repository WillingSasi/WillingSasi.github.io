---
layout:     post
title:      "Springboot配置RabbitMQ"
subtitle:   "RabbitMQ"
date:       2021-08-27 15:27:00
author:     "WS"
header-img: "img/mq.jpg"
catalog:    true
tags:
    - RabbitMQ
    - Springboot
---

## 1.RabbitMQ 背景

1. 概括说就是一种消息中间件, AMQP(高级消息队列协议)的一种具体实现, 类似的还有阿里的RocketMQ, Kafka等, 其中的MQ的意思是Message Queue, 专业解释就是消息总线，是一种跨进程、异步的通信机制，用于上下游传递消息, 确保消息的可靠传递;

2. 主要用于软件解耦、异步、流量削锋、数据分发、错峰流控、日志收集等;

3. 整体架构图(来自网络):

   > 一句话概括就是, **生产者把消息发布到Exchange上，消息最终到达队列并被消费者接收，而Binding决定交换器的消息应该发送到哪个Queue。**

   ![javascript](/img/mq0.png)

## 2.一类实现逻辑

Springboot已经有适配的各种模版了, 下面我来介绍两种普遍方法, 一种是用现成的rabbitmq注解和配置, 另一种是手动实现注册SimpleMessageListenerContainer, 并手动注入链接配置信息, 两种方法在大体上都分三部分, 如下:

> 1. mq链接配置信息, 用springboot template可直接配置在application.yaml中spring.rabbitmq下, 或自定义配置
> 2. 启动rabbitmq的监听容器
> 3. 写入q的方法就可直接调用了, 读取q需要注册进消息处理部分, 比如读取到消息直接存储redis缓存等操作 

### 3. 两种实现方法

>  首先都得导入spring amqp依赖
>
> ```xml
>         <dependency>
>             <groupId>org.springframework.boot</groupId>
>             <artifactId>spring-boot-starter-amqp</artifactId>
>         </dependency>
> ```

#### 方法一: springboot rabbitTempale 省事全家桶方法

 1.1 yaml或properties配置链接信息

```yaml
server:
  port: 8080

spring:
  rabbitmq:
    addresses: xxxxx
    virtual-host: xxxx
    username: xx
    password: xxx
    template:
      exchange: xxx
      routing-key: xxxx
      default-receive-queue: xxxx
    listener:
      simple:
        acknowledge-mode: manual
        concurrency: 3
        max-concurrency: 10
```

1.2 注册queue和绑定关系到spring bean中

```java
@Component
public class RabbitMqConfig {

    @Value("{$spring.rabbitmq.template.default-receive-queue}")
    private String queueName;

    @Value("{$spring.rabbitmq.template.exchange}")
    private String exchangeName;

    @Value("{$spring.rabbitmq.template.routing-key}")
    private String routingName;

    @Bean
    public Queue directDueue() {
        return new Queue(queueName, true);
    }

    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange(exchangeName, true, false);
    }

    //直接绑定交换机和消息队列及路由规则
    @Bean
    public Binding binding() {
        return BindingBuilder.bind(directDueue()).to(directExchange()).with(routingName);
    }

}
```

1.3 发送简单message方法, 只需要Autowired RabbitTemplate直接在其他service方法中调用即可

```java
@Configuration
public class ProduceService {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * 一对一通信,发送消息
     */
    public void directSend(String messages){
        rabbitTemplate.convertAndSend("queue-only", messages);
    }
}
```

1.4 注册监听queues到bean, 并加入对queue处理逻辑方法

```java
@Component
public class ConsumerService {

    @RabbitListener(queues = "XXXX")
    public void listenerQueue(String msg, Channel channel, Message message) throws IOException {
        System.out.println(msg);

        try {
            // 框架容器，是否开启手动ack按照框架配置
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
        } catch (Exception e) {
            e.printStackTrace();
            //丢弃这条消息
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
        }

    }
}
```



####方法二: 手动注册 SimpleMessageListenerContainer容器

2.1 配置yaml, 可随意定义名称

```yaml
online:
  request:
    rabbitmq:
      addresses: xxxx
      virtual-host: xxxx
      username: xxxx
      password: xxxx
      exchange: xxxx
      routing-key: xxxx
  response:
    rabbitmq:
      addresses: xxxx
      virtualHost: xxxx
      username: xxxx
      password: xxxx
      queue: xxxx
      exchange: xxxx
      routing: xxxx
confirm:
  rabbitmq:
    consumer-no: 3
    prefech-no: 10
```

2.2 spring定义加载配置信息类

```java
@Data
@Configuration
@ConfigurationProperties(prefix = "online.request.rabbitmq")
//发送q配置
public class RequestProperties {
    private String addresses;
    private String virtualHost;
    private String username;
    private String password;
    private String exchange;
    private String routingKey;

}
```

```java
@Data
@Configuration
@ConfigurationProperties(prefix = "online.response.rabbitmq")
//接收q配置
public class ResponseProperties {
    private String addresses;
    private String virtualHost;
    private String username;
    private String password;
    private String queue;
    private String exchange;
    private String routingKey;
}
```

2.3 发送和接收处理

```java
@Configuration
//手动注册ConnectionFactory, 实现进RabbitTemplate已备其他类直接调用发送q
public class RequestMQConfig {

    @Bean(name = "requestConnectionFactory")
    public ConnectionFactory destinationConnectionFactory(RequestProperties requestProperties) {
        CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory();
        cachingConnectionFactory.setAddresses(requestProperties.getAddresses());
        cachingConnectionFactory.setVirtualHost(requestProperties.getVirtualHost());
        cachingConnectionFactory.setUsername(requestProperties.getUsername());
        cachingConnectionFactory.setPassword(requestProperties.getPassword());

        return cachingConnectionFactory;
    }

    @Bean(name = "requestMQTemplate")
    public RabbitTemplate rabbitTemplate(@Qualifier("requestConnectionFactory") ConnectionFactory connectionFactory,
        RequestProperties prop, ObjectMapper objectMapper) {

        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setMessageConverter(new Jackson2JsonMessageConverter(objectMapper));
        rabbitTemplate.setExchange(prop.getExchange());
        rabbitTemplate.setRoutingKey(prop.getRoutingKey());

        return rabbitTemplate;
    }

}
```

```java
//发送q调用类
public class RequestCommandHandler {                                          
    @Autowired                                        
    @Qualifier("requestMQTemplate")           
    private RabbitTemplate mqTemplate;
  
    private static final ObjectMapper objectMapper = new ObjectMapper();
    
    public void requestQ() {
       MessagePostProcessor messagePostProcessor = new MessagePostProcessor() {
       @Override
       public Message postProcessMessage(Message message) throws AmqpException {      
           message.getMessageProperties().setHeader("content-type", "application/json");
           message.getMessageProperties().setHeader("Message-Type", "request");      
           message.getMessageProperties().setHeader("Version", "1.0.0");                
           return message;                                                              
           }                                                                                
       };                                                                                   
      mqTemplate.convertAndSend(bulidMessage(objectMapper.writeValueAsString(requestMsg)), 	\      messagePostProcessor); 
    }
```

```java
@Configuration
//注册监听q, 并注入处理逻辑container.setMessageListener(adapter), ResponseListener handler这里是业务处理
public class ResponseListenerConfig {
    @Value("${confirm.rabbitmq.consumer-no}")
    private Integer consumerNumber;

    @Value("${confirm.rabbitmq.prefech-no}")
    private Integer prefechNumber;

    @Autowired
    private ResponseProperties mqProperties;

    @Bean("responseConnectionFactory")
    public CachingConnectionFactory responseConnectionFactory() {
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
        connectionFactory.setVirtualHost(mqProperties.getVirtualHost());
        connectionFactory.setAddresses(mqProperties.getAddresses());
        connectionFactory.setUsername(mqProperties.getUsername());
        connectionFactory.setPassword(mqProperties.getPassword());
        return connectionFactory;
    }

    @Bean
    public DirectExchange exchange() {
        return new DirectExchange(mqProperties.getExchange(), true, false);
    }

    @Bean
    public Queue queue() {
        return new Queue(mqProperties.getQueue(), true);
    }

    @Bean
    public Binding binding() {
        return BindingBuilder.bind(queue()).to(exchange()).with(mqProperties.getRoutingKey());
    }

    @Bean
    public SimpleMessageListenerContainer responseListenerContainer(
        @Qualifier("responseConnectionFactory") ConnectionFactory responseConnectionFactory,
        ResponseListener handler,
        ObjectMapper objMap) {

        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(responseConnectionFactory);
        container.setQueues(queue());

        container.setConcurrentConsumers(consumerNumber);
        container.setPrefetchCount(prefechNumber);

        MessageListenerAdapter adapter = new MessageListenerAdapter(handler);
        adapter.setDefaultListenerMethod("handle");
        adapter.setMessageConverter(new NotificationConverter(ResponseOnline.class, objMap));

        container.setMessageListener(adapter);
        return container;
    }
}
```



以上就是rabbitmq整个逻辑处理流程, 可根据实际项目自由发挥



参考学习链接:

[RabbitMQ - 简书 (jianshu.com)](https://www.jianshu.com/p/78847c203b76)

[Spring Boot + RabbitMQ 配置参数解释 - 一叶落知天下秋 - 博客园 (cnblogs.com)](https://www.cnblogs.com/qts-hope/p/11242559.html)

[springboot集成rabbitmq并手动注册容器实现单个queue的ack模式_hhsway的博客-CSDN博客](https://blog.csdn.net/qq_38439885/article/details/88982373)

[springboot整合RabbitMq - 改变从现在 - 博客园 (cnblogs.com)](https://www.cnblogs.com/jchao/p/14593224.html)

[Spring Boot：使用Rabbit MQ消息队列 - 朝雨忆轻尘 - 博客园 (cnblogs.com)](https://www.cnblogs.com/xifengxiaoma/p/11121355.html)

[简单消息监听容器--SimpleMessageListenerContainer - 码农教程 (manongjc.com)](http://www.manongjc.com/detail/13-tkjdiqntmqzzqms.html)

[SimpleMessageListenerContainer · JAVA · 看云 (kancloud.cn)](https://www.kancloud.cn/java-jdxia/java/1599578)



