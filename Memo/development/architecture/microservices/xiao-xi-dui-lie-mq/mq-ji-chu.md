---
type: memo
date: 2026-11-06
aliases:
  - "MQ基础"
---
# MQ基础

![](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/JSYnbgcXWomOVPxbVhOcmTggnRh.png)

## MQ有什么应用场景

```Java
1. 简单任务分发场景：使用于消息数序不重要的简单任务，采用单个生产者对应单个消费者的模式完成任务传递。
2. 提升系统并发处理能力场景：通过多个生产者向多个消费者分发任务，实现任务的并行处理，提升整体系统的    并发处理效率。
3. 消息广播类场景：比如日志记录，实时聊天，新闻发布等场景，将消息发哦是那个至交换机后，多个雄安飞着    都能接收到该消息。
4. 消息选择性路由场景：可以根据消息内容进行过滤通过指定不同的路由键，让交换机将消息发送到对应的队      列，实现消息的精准路由
5. 消息多样性路由场景：支持使用通配符指定主题，交换机根据主题将消息发送到对应的队列，使用与需要根据    不同主题进行消息过滤和选择的场景。
6. 延迟任务场景：可以将业务判断类任务改造成MQ延迟任务，比如订单超时未支付自动取消这类场景。
7. IOT设备数据传输场景：在IOT设备数据阐述中使用MQTT协议（MQ的一种），实现华为云IOT平台于设备之间    的信息获取，同时可使用AMQP协议，完成IOT平台数据向Java应用程序的传输。
```

## 为什么要用mq

```Plain Text
1. 因为项目比较大，做了分布式系统，所有远程服务调用请求都是同步执行经常出问题，所以引入了mq
2. 解耦：系统耦合度降低，没有强依赖关系
3. 异步：不需要同步执行的远程调用可以有效提高响应时间
4. 削峰：请求达到峰值后，后端service还可以保持固定速率消费，不会被压垮。
```

## mq工作模型

### 普通模型

![](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/QJpObLkt1o6SHJxk7fmcrLE3ncd.png)



```Plain Text
这种方式实现最基本的异步通信，一个生产者，一个队列，一个消费者，生产者将消息发到队列，消费者从队列接收消息。
```

<callout emoji="🌟">
一对一的普通队列消费者只有一个所以处理消息的能力就比较弱，当生产者发送的消息远远大于消费者处理任务的速度此时就容易造成消息堆积
</callout>



### 工作队列（work queue）

![](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/T5wxbcxHeorLvSxiu7fcBU9Kn3g.png)

```Plain Text
工作队列：让多个消费者绑定到一个消息队列上，共同消费队列中的消息。
```

<callout emoji="🎼">
多个消费者绑定到同一个队列，同一条消息只会被一个消费者处理。
解决方式：能者多劳，通过设置prefetch来控制消费者预取得消息数量。
</callout>

### 发布订阅模型(Publish/Subscribe)

![](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/R0mibOBCKoy2yFxFsmrclq1rnqg.png)

```Plain Text
发布订阅模型里，生产者只能将消息发送到交换机，有交换机将消息推送到队列，交换机可以将消息推送给绑定他的所有队列，也可以有针对性的消息推送给某几个队列，这就相当于有一批消费者订阅了消息，交换机根据各自的订阅去推送消息。
1. publisher:生产者，不在发送消息到队列中，而是发给交换机
2. exchange：交换机，一方面，接收生产者发送的消息，另一方面，将消息推送给队列，是将消息推送给某个    特别队列，递交给所有队列，或是将消息丢弃，到底如何操作，取决于交换机的类型。
3. queue：消息队列也与以前一样，接收消息缓存消息。不过队列一定要与交换机绑定。
4. consumer：消费者，与以前一样，订阅队列，没有变化。
注意：exchange(交换机)：只负责转发消息，不具备存储消息的能力，因此如果没有任何队列于exchange绑定，或者没有符合路由规则的队列，那么消息会丢失。
```

### 交换机类型

```Plain Text
1. fanout:广播类型，将消息交给所有绑定到交换机的队列。
2. direct:直接类型，基于routingkey(路由key)发送给定于了消息的队列，交换机根据routingkey去判断    消息应该转发到那个队列
3. topic:通配符类型（主题类型），与direct类似，只不过routingkey可以使用通配符
4. headers:头匹配类型，基于mq的消息头匹配用的较少（知道名字就行）
```

#### fanout类型（广播模式）

![](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/TeCgbGCuGo0rRsxmlbpcf41Jnwf.png)

```Plain Text
1. 可以有多个队列
2. 每个队列都要绑定到exchange（交换机）
3. 生产者发送的消息，只能发送到交换机
4. 交换机把消息发送给绑定过的所有队列
5. 订阅队列的消费者接收消息，每个队列订阅的消费者只有一个能拿到消息
```

![](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/L2TCbX9TMoYVo8xGNYVczpUDnWD.png)

```Plain Text
在每个队列的消费者中，发送一条消息只会有一个消费者接收到消息。
每个队列默认采用轮询的方式向消费者推送消息。
```

<callout emoji="🎉">
交换机的作用是什么？
1. 接收publisher发送的消息
2. 将消息按照规则路由到与之绑定的队列
3. 不能缓存消息，路由失败，消息丢失
4. fanoutexchange则会将消息路由到每个绑定的队列
</callout>

#### Direct类型

![](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/AFfSbW53boSEDTxw97jcxEQPnmf.png)

```Plain Text
在Direct模型下：
1. 队列与交换机的绑定，不能是任意绑定了，而是要指定一个RoutingKey(路由key)
2. 消息的发送方在 向exchange发送消息时，也必须指定消息的routingkey
3. exchange不再把消息交给每个绑定的队列，而是根据消息的routingkey进行判断，只有队列的             routingkey与消息的routingkey完全一致，才会接收到消息。 
```

<callout emoji="🥇">
Direct交换机与Fanout交换机的差异？
1. Fanout交换机将消息路由给每一个与之绑定的队列
2. Direct交换机根据routingKey判断路由给那个队列
3. 如果多个队列具有相同的RoutingKey，则与Fanout功能类似
</callout>



#### Topic交换机

![](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/OrDEb7lSQoUNXbxeJhucsAmoncd.png)



```Plain Text
介绍：topic类型的exchange与Direct相比，都是可以根据RoutingKey把消息路由到不同的队列只不过topic类型exchange可以让队列在绑定BindingKey的时候使用通配符！
BindingKey一般都是有一个或多个单词组成，多个单词之间以 . 分隔，例如：item.insert

两种通配符：
1. #：可以匹配任意多个单词
2. *：只可以匹配一个单词
```

```Plain Text
假如现在pubisher发送的消息使用的RoutingKey共有四种：
1. china.news代表有中国的新闻消息；
2. china.weather代表中国的天气消息；
3. japan.news则代表日本新闻
4. japan.weather代表日本的天气消息；

解释：
1. topic.queue1：绑定的是china.#,凡是以china.开头的routing key都会被匹配到，包括
    china.news、china.weather
2. topic.queue2：绑定的是#.news，凡是以 .news结尾的toutingkey都会被匹配。包括
    china.news、japan.news
```

**解决JDK消息序列化问题**

1. 引入依赖 在`publisher`和`consumer`两个服务的父工程中都引入依赖：

```XML
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.9.10</version>
</dependency>
```

1. 配置消息转换器，在`publisher`和`consumer`两个服务中添加一个Bean即可：

注意：MessageConverter要导入org.springframework.amqp.support.converter.MessageConverter

```Java
package com.itheima.consumer.config;

import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author Mr.M
 * @version 1.0
 * @description mq配置类
 * @date 2024/8/13 19:28
 */
@Configuration
public class MesaageConfig {

    @Bean
    public MessageConverter messageConverter() {
        // 定义消息转换器
        Jackson2JsonMessageConverter jackson2JsonMessageConverter = new Jackson2JsonMessageConverter();
        //设置消息id,自动生成一个id，可用于消息幂等性处理
        jackson2JsonMessageConverter.setCreateMessageIds(true);
        return jackson2JsonMessageConverter;
    }
}
```

<callout emoji="📚">
我们可以知道不论生产者发送什么样的样式的消息最终spring都会帮我们转换成Message类型的消息
</callout>

## 面试题：

### RabitMQ的工作模型有哪些

```Java
1. 普通模型：一对一消息发送和接收，这样会出现问题：当生产者生产的消息速度远远大于消费者消费速度时，    会造成消息堆积
2. 工作队列（workQueue）：让多个消费者绑定到一个消息队列上，共同消费队列中的消息。能者多劳配置通     过设置prefetch来控制消费者预取得消息数量。
3. Fanout模式（广播）：添加一个交换机，把消息发给交换机然后让交换机把消息转发给每一个队列，每个队     列订阅的消费者只有一个能拿到消息
4. Direct模式（路由键）：通过设置路由键队列绑定路由键，消息的发送方在 向exchange发送消息时，也必    须指定消息的routingkey，而且交换机发送消息给队列的时候也不再是给每个队列都发，而是只发给对应的    RouterKey的队列
5. Topic（通配符路由键）：和Direct一样只不过这个是多了两个通配符可以更加灵活
```

### 如何解决MQ消息堆积问题？

```Plain Text
使用工作队列（work queue）模型：多个消费者绑定到同一个队列，同一条消息只会被一个消费者处理。通过设置 prefetch 控制消费者预取的消息数量，实现“能者多劳”，提升消费者整体处理速度，解决生产者发送消息速度远大于消费者处理速度导致的消息堆积问题。
```

---
> 来源：飞书技术知识库（脱敏版） · 04 微服务与中间件 / 消息队列 MQ / MQ基础 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/ZOXkw6a5QiBEcVk7acBc8sy9nxf)
