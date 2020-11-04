# 使用场景

> 服务A将数据添加到MQ中，由服务B中的消费者执行，若服务B处理失败则添加至死信队列。

# 开始

## 导入相关包

```xml
<!-- MQ的核心包 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <version>1.5.2.RELEASE</version>
</dependency>
```



## 配置文件

```yml
spring:
    rabbitmq:
        host: 172.16.2.30
        port: 5672
        username: admin
        password: 123457
        listener:
          type: simple
          simple:
            default-requeue-rejected: false
            acknowledge-mode: manual
```



## 配置类

```java
package com.lksun.service1.config;

import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.annotation.EnableRabbit;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.HashMap;
import java.util.Map;

// 1. 加入注解 , 一定记得开启Rabbit
@Configuration
@EnableRabbit
public class RabbitMqConfig {
    // 2. 定义一些常量,也就是在消息队列中的名字,定义为常量是为了方便后续的修改和统一
    public static final String LKSUN_TEST_EXCHANGE = "lksun_test_exchange";
    public static final String LKSUN_TEST_QUEUE = "lksun_test_queue";
    public static final String LKSUN_TEST_ROUTING_KEY = "lksun_test_routingKey";
    public static final String LKSUN_TEST_DEAD_LETTER_EXCHANGE = "lksun_test_dead_letter_exchange";
    public static final String LKSUN_TEST_DEAD_LETTER_QUEUE = "lksun_test_dead_letter_queue";
    public static final String LKSUN_TEST_DEAD_LETTER_ROUTING_KEY = "lksun_test_dead_letter_queue";
	
    // 3.声明一个业务交换机
    @Bean
    public DirectExchange OrderExchange() {
        // direct 模式 , 起名为lksun_test_exchange
        return new DirectExchange(LKSUN_TEST_EXCHANGE);
    }
	
    // 4. 声明一个Queue , 初学时只定义了交换机启动总是报错
    @Bean
    public Queue OrderQueue() {
        // 5. 声明一个Map 主要是用来存一些配置
        // 因为需求是一个业务队列+死信队列的组合,所以此处需要将两个队列捆绑
        Map<String, Object> args = new HashMap<>(2);
        // x-dead-letter-exchange    这里声明当前队列绑定的死信交换机
        args.put("x-dead-letter-exchange", LKSUN_TEST_DEAD_LETTER_EXCHANGE);
        // x-dead-letter-routing-key  这里声明当前队列的死信路由key
        args.put("x-dead-letter-routing-key", LKSUN_TEST_DEAD_LETTER_ROUTING_KEY);
        return QueueBuilder.durable(LKSUN_TEST_QUEUE).withArguments(args).build();
    }
	
    // 6. 将Exchange和Queue绑定
    @Bean
    public Binding Orderbinding() {
        return BindingBuilder.bind(
            OrderQueue()).
            to(OrderExchange()).
            with(LKSUN_TEST_ROUTING_KEY);
    }

    // 7. 步骤五时给业务队列捆绑了死信队列,是捆绑不能代替声明
    @Bean
    public DirectExchange deadLetterExchange(){
        return new DirectExchange(LKSUN_TEST_DEAD_LETTER_EXCHANGE);
    }

    // 8. 死信队列声明Queue
    @Bean
    public Queue deadLetterQueue(){
        return new Queue(LKSUN_TEST_DEAD_LETTER_QUEUE);
    }
	
    // 9. 绑定
    @Bean
    public Binding deadLetterBinding(){
        return BindingBuilder.bind(
            deadLetterQueue()).
            to(deadLetterExchange()).
            with(LKSUN_TEST_DEAD_LETTER_ROUTING_KEY);
    }
}
```



## 消费者

```java
@Component
public class RabbitMqTask {
    // 业务队列
    @RabbitListener(queues = RabbitMqConfig.LKSUN_TEST_QUEUE)
    public void task(Message message, Channel channel) throws IOException {
        String msg = new String(message.getBody());
    }
    
    // 死信队列
    @RabbitListener(queues = RabbitMqConfig.LKSUN_TEST_DEAD_LETTER_QUEUE)
    public void receiveA(Message message, Channel channel) throws IOException {
        
    }
}
// 手动ACK
channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
// 手动NACK
channel.basicNack(message.getMessageProperties().getDeliveryTag(), false,false);
```



## 推送消息

```java
@Service
public class RabbitMQServiceImpl implements RabbitMQService {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Override
    public void sendMsg(String msg) {
        // 参数1 : 交换机名称
        // 参数2 : 路由KEY
        // 参数3 : 消息
        rabbitTemplate.convertAndSend(
            RabbitMqConfig.LKSUN_TEST_EXCHANGE,
            RabbitMqConfig.LKSUN_TEST_ROUTING_KEY,
            msg);
    }
}
```

