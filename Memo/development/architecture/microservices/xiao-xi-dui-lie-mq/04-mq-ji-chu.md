---
type: memo
date: 2026-11-06
aliases:
  - "04 MQ基础"
---
# 04 MQ基础

## 学习目标

1. 能够说出MQ的应用场景
2. 能够编写RabbitMQ收发消息的入门程序
3. 能够说出工作队列模型的特点
4. 能够说出发布订阅模型的交换机类型
5. 能够说出fanout交换机的特性
6. 能够说出Direct交换机的特性
7. 能够说出Topic交换机的特性
8. 能够说出惰性队列的特性
9. 能够说出优先级队列的特性
10. 能够说出如何解决MQ消息堆积问题
11. 能够在商城项目应用MQ

## 1.初识MQ

### 1.1 同步调用与异步调用

前边我们学习了微服务之间远程调用方式，通常服务端提供HTTP RESTful接口，客户端通过HTTP Client工具进行远程调用，远程调用工具有RestTemplate、OpenFeign、OkHttp等技术，这些技术实现的都是一种类型即同步调用，微服务之间通信还有一种异步调用，什么是同步调用？什么是异步调用？

拿订单支付功能举例：

下图表示了同步调用

<whiteboard token="Cs53wdrrHhPxXUb5GsQcag3Snhd"></whiteboard>

支付的交互流程如下：

1. 支付服务调用用户服务扣减余额。
2. 余额扣减成功后支付服务更新数据库中的交易流水状态为已支付。
3. 更新成功后支付服务调用交易服务更新订单状态为已支付。

像这种每一步调用者必须等待被调用方完全执行完毕并返回结果之后才能继续执行后续代码，叫**同步调用**。

同步调用就是顺序执行，执行完一步再执行下一步。



什么是异步调用？

**异步调用是调用者发出调用后无需等待被调用方完成就可以继续执行其他任务。**

举例：

<whiteboard token="HnmAwWqiUhEkhYb3zGqcAcmpn3e"></whiteboard>

支付的交互流程如下：

1. 支付服务调用用户服务扣减余额。
2. 余额扣减成功后支付服务更新数据库中的交易流水状态为已支付。
3. 更新交易流水成功支付服务向消息中间件发了一条消息（XX订单支付成功），此时支付服务程序执行结束
4. 由消息中间件去通知交易服务更新订单状态
5. 由消息中间件去通知短信服务通知用户订单支付成功啦

上边交互流程中前3步为**同步调用**，后2步为**异步调用**。

支付服务向消息中间件发过消息后不用等继续执行其它操作，这就是**异步调用**。

现在生活中异步调用的例子很多：

餐厅点餐：

想象一下你去一家餐厅吃饭，你坐下后，服务员过来让你点菜。在这个过程中：

- 异步调用：你告诉服务员你想要什么菜品，然后服务员把订单送到厨房。你不需要等待食物准备完成，可以继续聊天或浏览菜单。当食物准备好时，服务员会将食物端到你的桌上。
- 同步调用：如果你必须站在厨房门口等着厨师为你准备食物，那么这就是一个同步的过程。你无法做其他事情，直到食物准备完毕。

医院挂号：

异步调用：打电话挂号，接通电话你告诉工作人员挂哪个科室，工作人员让你挂断电话稍后通过手机查看挂号结果。

同步调用：打电话挂号，接通电话你告诉工作人员挂哪个科室，工作人员开始查看该科室是否有号，并进行挂号，挂号结束告诉你几点来看病，最后挂断电话。



**同步调用：**

**特点：**

- 在同步调用中，调用者必须等待被调用方完全执行完毕并返回结果之后才能继续执行后续代码。
- 控制流是**线性**的，即程序按照顺序执行每个操作。
- 如果被调用方执行时间较长，那么整个程序会处于等待状态，直到该调用完成。

**适合场景：**

- 对实时性要求较高的场景，例如用户界面操作。
- 调用简单且执行快速的操作。

**优点**

- 简单直观：代码易于理解和编写，因为它是按顺序执行的。
- 易于调试：由于执行顺序明确，调试起来相对容易。

**缺点**

- 阻塞执行：如果一个操作需要很长时间来完成，那么整个程序会被阻塞，不能执行其他任务。
- 资源浪费：在等待长时间操作完成时，CPU和其他资源可能会处于空闲状态。



**异步调用：**

**特点：**

- 在异步调用中，调用者发出调用后无需等待被调用方完成就可以继续执行其他任务。
- 被调用方完成操作后，通常会通过**回调函数、事件通知或者完成信号**等方式告知调用者。
- 可以提高程序的并发能力和响应速度。

**适合场景：**

- 需要处理耗时操作，例如网络请求、文件I/O等。
- 多个操作之间不存在严格的依赖关系。
- 对系统性能和响应时间有较高要求的应用场景。

**优点**

- 提高效率：异步调用可以使得程序在等待某些耗时操作完成的同时执行其他任务，提高了整体的执行效率。
- 资源利用率高：在等待耗时操作时，可以释放资源给其他任务使用，避免了资源浪费。
- 更好的用户体验：在网络应用中，用户不必等待页面加载完成就能进行其他操作，提高了用户体验。

**缺点**

- 复杂性增加：异步编程通常比同步编程更复杂，因为它涉及更多的控制结构和错误处理逻辑。
- 调试困难：由于执行路径不是线性的，调试起来相对困难。



### 1.2 初识MQ

异步调用方式其实就是基于消息通知的方式，一般包含三个角色：

- 消息发送者：投递消息的人，就是原来的调用方
- 消息Broker(消息代理/消息中间件)：管理、暂存、转发消息，你可以把它理解成微信服务器
- 消息接收者：接收和处理消息的人，就是原来的服务提供方

<whiteboard token="SXAxwv5M6h1ODlbNqlOcd7cmnvd"></whiteboard>



在异步调用中，发送者不再直接同步调用接收者的业务接口，而是发送一条消息投递给消息Broker。然后接收者根据自己的需求从消息Broker那里订阅消息。每当发送方发送消息后，接收者都能获取消息并处理。

这样，发送消息的人和接收消息的人就完全**解耦**了。



消息Broker，目前常见的实现方案就是消息队列（MessageQueue），简称为MQ.



AI:消息队列的应用场景

1. 异步处理:

   - 场景: 当应用程序需要执行耗时的操作（如发送电子邮件、文件上传或下载等）时，可以将这些任务发送到消息队列中，由专门的任务处理程序异步执行。
   - 好处: 减少了用户的等待时间，提高了用户体验。
2. 解耦:

   - 场景: 当一个系统由多个组件组成时，这些组件之间可以使用消息队列进行通信。
   - 好处: 单个组件的变化不会直接影响到其他组件，提高了系统的可扩展性。
3. 流量削峰

   - 场景：在许多互联网应用和服务中，尤其是那些具有明显周期性流量特征的应用（如电商平台、社交网络等），常常会遇到流量突增的情况。例如，在电商促销期间，大量的用户会在短时间内访问网站并提交订单，这种短时间内产生的巨大流量可能会导致服务器过载，影响用户体验甚至导致服务不可用。
   - 好处：当流量激增时接收到请求会被暂时存储在消息队列中，而不是直接发送到后端服务进行处理。这样做可以避免后端服务因为短时间内处理大量请求而过载。后端服务可以从消息队列中按需拉取消息进行处理。这种异步处理机制可以有效地分散流量峰值，确保后端服务的稳定运行。如下图：

   ![图片展示了流量削峰的场景。上方图中，用户向系统A发送大量请求，数据写入数据库，但瞬间请求量过大，会造成数据库异常。下方图中，用户同样向系统A发送大量请求，但系统A将数据写入消息队列，消费者1和消费者2分别订阅消息队列，再将数据写入数据库。该图与上下文紧密相关，直观呈现了流量削峰前后的系统架构差异，帮助理解流量削峰在系统设计中的作用。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/NbBFbSJWvoBpPjxJHCocXstpnld.png)

   ![图片展示了秒杀系统中消息队列的应用场景。上方有多个用户图标，分别有“秒杀成功”和“队列已满，秒杀失败”的箭头指向消息队列。消息队列右侧有订单处理图标，箭头指向其，标注“根据规则读取消杀消息”。该图与上下文介绍的秒杀系统中消息队列的应用相呼应，直观呈现了用户秒杀成功或失败时消息队列的处理情况。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/I9V8bWjOqorZqQxPfmDcbzWtn0f.png)



目比较常见的MQ实现：

- ActiveMQ
- RabbitMQ
- RocketMQ
- Kafka



几种常见MQ的对比：

追求可靠性：RabbitMQ、RocketMQ

追求吞吐能力：RocketMQ、Kafka

追求消息低延迟：RabbitMQ、Kafka

这四种的MQ在市场都是非常流行，本课程讲解RabbitMQ。

Spring Boot默认支持AMQP协议，RabbitMQ支持AMQP协议 。

### 1.3 面试题

MQ有什么应用场景？



## 2 RabbitMQ入门

### 2.1 RabbitMQ介绍

RabbitMQ是基于Erlang语言开发的开源消息通信中间件，官网地址：[https://www.rabbitmq.com/](https://www.rabbitmq.com/)



RabbitMQ的架构如图：

![这张图展示了RabbitMQ的核心架构，最外侧标注有“RabbitMQ Server Broker”，内部划分出“VirtualHost”虚拟主机区域。图中左侧的“Publisher”作为生产者，通过连接指向区域内的“exchange（交换机）”；交换机作为消息路由组件，可将消息转发至区域内的多个“queue（队列）”；队列作为存储消息的缓冲区，最终将消息传递给右侧的“consumer（消费者）”。该图直观呈现了RabbitMQ中生产者、交换机、队列、消费者各角色的关联关系，对应了文档中提到的RabbitMQ核心概念架构。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/MtA3bpQqyoL71uxxuBHczYwYn0t.png)

其中包含几个概念：

- **`publisher`**：生产者，也就是发送消息的应用程序
- **`consumer`**：消费者，也就是消费消息的应用程序
- **`queue`**：队列，存储消息的缓冲区。生产者投递的消息会暂存在消息队列中，等待消费者处理
- **`exchange`**：交换机，负责消息路由。生产者发送的消息由交换机决定投递到哪个队列。
- **`virtual host`**：虚拟主机，起到数据隔离的作用。每个虚拟主机相互独立，有各自的exchange、queue



上述这些东西都可以在RabbitMQ的管理控制台来管理，下一节我们就一起来学习控制台的使用。



### 2.2. 安装

我们同样基于Docker来安装RabbitMQ，使用下面的命令即可：

删除旧容器

```Plain Text
docker rm -f rabbitmq mq
```



执行下边的脚本创建容器：

```Shell
docker run \
 -e RABBITMQ_DEFAULT_USER=itheima \
 -e RABBITMQ_DEFAULT_PASS=123321 \
 -v mq-plugins:/plugins \
 --name mq \
 -p 15672:15672 \
 -p 5672:5672 \
 -d --restart=always \
 rabbitmq:3.8-management
```



可以看到在安装命令中有两个映射的端口：

- **15672**：RabbitMQ提供的管理控制台的端口     http
- **5672**：RabbitMQ的消息发送处理接口       amqp



安装完成后，我们访问 http://192.168.101.68:15672即可看到管理控制台。首次访问需要登录，默认的用户名：itheima，密码：123321

登录后即可看到管理控制台总览页面：

![图片展示的是RabbitMQ管理控制台的总览页面。页面上方显示RabbitMQ版本为3.8.26，Erlang版本为24.2。总览部分有队列消息、当前队列、消息速率、当前消费者等数据。节点信息区域列出节点名称、文件描述符、Erlang进程、内存、磁盘空间等数据，如rabbit@img节点的文件描述符为102等。页面还包含Churn图表、端口和上下文、导出定义、导入定义等内容。该页面是安装完成后访问http://192.168.101.68:15672后登录默认用户名和密码看到的管理控制台总览页面。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/YlstbpbvZo0uprxgJHycUHEGnGb.png)



### 2.3. 收发消息

#### 2.3.1 搭建环境

RabbitMQ安装成功后下边我们编写消息发送与消息接收程序实现收发消息，如下图：publisher即消息发送者将消息发送到MQ的队列中，consumer即消息消费者从MQ中接收消息。

`与RabbitMQ`通信采用了AMQP (Advanced Message Queuing Protocol) 协议，因此它具备跨语言的特性，任何语言只要遵循AMQP协议都可以使用RabbitMQ收发消息。

<whiteboard token="H8ViwqgXshXhNvbrzbOcve0MnKc"></whiteboard>

`RabbitMQ`官方也提供了各种不同语言的客户端API，RabbitMQ官方提供的Java客户端编码相对复杂，一般生产环境下我们更多会结合Spring来使用。而Spring的官方刚好基于RabbitMQ提供了这样一套消息收发的模板工具：SpringAMQP。并且还基于SpringBoot对其实现了自动装配，使用起来非常方便。



SpringAmqp的官方地址：

<bookmark name="Spring AMQP" href="https://spring.io/projects/spring-amqp"></bookmark>

SpringAMQP提供了三个功能：

- 自动声明队列、交换机及其绑定关系
- 基于注解的监听器模式，异步接收消息
- 封装了RabbitTemplate工具，用于发送消息



下边使用SpringAMQP实现消息收发，上图是RabbitMQ最简单的工作模型。



在课前资料给大家提供了一个Demo工程，方便我们学习SpringAMQP的使用：

![图片展示了课前资料中提供的Demo工程文件夹内容。文件夹名为“mq-demo”，类型为文件夹，大小为247,988 KB。该图片位于介绍课前资料中Demo工程位置的上下文之后，用于直观呈现课前资料提供的Demo工程文件夹结构，帮助学习者了解在工作空间中打开此工程后能看到的项目结构，辅助理解后续关于消息发送者和消费者的内容。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/F6n9bVFhKoVeaKxYYBTc4L3Snfb.png)

将其复制到你的工作空间，然后用Idea打开，项目结构如图：

![这张图展示了Idea中打开的mq-demo项目目录结构，路径为D:\\code\\mq-demo。该项目包含三个核心子模块，分别是用于管理项目依赖的父工程mq-demo、消息发送者publisher以及消息消费者consumer，此外目录中还包含.idea配置文件夹、src源码文件夹、项目配置文件mq-demo.iml、Maven配置文件pom.xml，以及External Libraries和Scratches and Consoles等辅助目录，清晰对应了文档中提到的Demo工程结构，是学习者学习SpringAMQP时的基础项目结构展示。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/TUfTbHImsoKsIYxmrP9ckroin0b.png)

包括三部分：

- mq-demo：父工程，管理项目依赖
- publisher：消息的发送者
- consumer：消息的消费者



在mq-demo这个父工程中，已经配置好了SpringAMQP相关的依赖：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.itcast.demo</groupId>
    <artifactId>mq-demo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>publisher</module>
        <module>consumer</module>
    </modules>
    <packaging>pom</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.12</version>
        <relativePath/>
    </parent>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <!--AMQP依赖，包含RabbitMQ-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <!--单元测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
    </dependencies>
</project>
```

设置java版本

![这张图片展示的是Idea中Settings里Java Compiler的配置页面。该页面可设置编译相关参数，其中在“Per-module bytecode version”区域，显示了consumer、mq-demo、publisher三个子工程模块对应的目标字节码版本均为11，且该配置属于mq-demo父工程中已配置的内容，可让子工程直接使用SpringAMQP相关依赖，同时页面还显示Use compiler选项为Javac，且勾选了“Use ‘--release’ option for cross-compilation (Java 9 and later)”等相关选项，项目整体的目标字节码版本也设置为与语言级别相同。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/Mv2cbGtGcoRsztxswuIcXLnqnke.png)

![这张图片展示了IDEA中Project Structure的配置界面，对应文档里搭建RabbitMQ Demo环境的项目配置环节。界面左侧选中的是Project设置项，右侧显示父工程名为mq-demo，核心内容为被红框高亮标注的SDK配置，当前SDK为11版本的Oracle OpenJDK version 11.0.15，该配置对应文档中提到的父工程已设置好Java版本的内容，确认了项目的Java开发环境配置。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/A6WnbGqiZoYhEuxtKEncWkRknHd.png)

![这是IDEA的项目结构设置界面，展示了mq-demo父工程下包含的publisher、consumer两个子工程模块。界面左侧选中了Modules设置项，中间选中了consumer子模块，该模块的语言级别被标注要求设置为11，对应选项为“11 - Local variable syntax for lambda parameters”，每个子模块都需要按要求完成该语言级别的设置，以此确保项目使用的Java版本等环境配置符合要求，便于后续SpringAMQP相关功能的使用。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/KoeVbUjPIo2sHtxNLmUcU3jRnyh.png)

![图片展示了IDEA中settings.xml文件的部分内容。其中突出显示了设置Java版本的代码区域，包含<profile>标签，id为jdk - 11，激活默认为true，<properties>标签下有maven . compiler . source、maven . compiler . target、maven . compiler . compilerVersion等属性，均设置为11。该图片与文档中介绍在mq - demo父工程中已配置好SpringAMQP相关依赖，子工程可直接使用SpringAMQP的内容相关，直观呈现了Java版本设置的部分代码。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/OiLqb6dYPol1V6x5UCdcv0Pdnof.png)

因此，子工程中就可以直接使用SpringAMQP了。



#### 2.3.2 创建队列

首先进入RabbitMQ控制台创建队列，新建一个队列：simple.queue

![图片展示的是RabbitMQ控制台中“Queues”页面。页面显示当前队列数量为0，可添加新队列。添加队列时，需选择虚拟主机、类型（此处为Classic）、队列名称（已输入“simple.queue”）、持久性（Durability，已选中）、自动删除（Auto delete，未选中）等参数。页面还提供了分页、过滤、正则表达式搜索等功能，以及每页显示数量上限设置。该图与上文“创建队列”内容相关，直观呈现了创建队列时的界面及参数设置情况。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/DvfWbGVOJoICNFxhHfScennznqe.png)

添加成功：

![图片展示的是RabbitMQ控制台中队列（Queues）页面。页面显示了所有队列信息，当前显示1个队列，即simple.queue，其类型为classic，状态为D。页面上方有刷新频率设置，当前设置为每5秒刷新一次。页面右侧有刷新频率、虚拟主机、用户等下拉选择项。该图片与上文“首先进入RabbitMQ控制台创建队列，新建一个队列：simple.queue”内容对应，直观呈现了队列创建后的页面情况。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/I2gTbL7oJouPnxxE8HWcKk6mnVg.png)

接下来，我们就可以利用Java代码收发消息了。

#### 2.3.3 消息发送

首先配置MQ地址，在`publisher`服务的`application.yml`中添加配置：

```YAML
spring:
  rabbitmq:
    host: 192.168.101.68 # 你的虚拟机IP
    port: 5672 # 端口
    virtual-host: / # 虚拟主机
    username: itheima # 用户名
    password: 123321 # 密码
```



然后在`publisher`服务的test下编写测试类`SpringAmqpTest`，并利用`RabbitTemplate`实现消息发送：

```Java
package com.itheima.publisher.amqp;

import org.junit.jupiter.api.Test;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
@Slf4j
public class SpringAmqpTest {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void testSimpleQueue() {
        // 队列名称
        String queueName = "simple.queue";
        // 消息
        String message = "hello, spring amqp!";
        // 发送消息
        rabbitTemplate.convertAndSend(queueName, message);
        log.info("消息发送成功:{}",message);
    }
}
```

![这张图片展示的是`publisher`服务的项目目录结构，该目录是MQ消息发送操作对应的代码工程结构。在src目录下包含main和test两个子目录，main目录中存储了运行程序的核心代码，其中有存放业务代码的java目录及application.yml配置文件；test目录用于存放测试代码，在test/java/com/itheima/publisher/amqp路径下，存在着用于实现消息发送的`SpringAmqpTest`测试类，这正好对应文档中提到的“在publisher服务的test下编写测试类`SpringAmqpTest`，并利用`RabbitTemplate`实现消息发送”的相关内容。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/TTcXbOUURoW500xBMjVcPLAnnKd.png)



打开控制台，可以看到消息已经发送到队列中：

![这张图片展示了RabbitMQ管理控制台的Queues页面，页面中选中了Queues栏目，当前共存在1个名为simple.queue的classic类型队列，所属虚拟主机为/hall，该队列当前处于idle空闲状态。在队列的消息统计区域，Ready项被红色方框标注，数值为1，Unacked项数值为0，Total项数值为1，其余Message rates区域无相关统计数据，此界面可直观体现发送到该队列的消息就绪待消费的状态。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/NjcobSE7GoyKayxnVK8cHsxunGe.png)

接下来，我们再来实现消息接收。

我们可以在RabbitMQ的控制台去查看消息

![图片展示的是RabbitMQ控制台中“Get messages”页面。页面提示从队列获取消息是破坏性操作。Ack Mode下拉框选中“Nack message requeue true”，Encoding为“Auto string / base64”，Messages输入框显示1。下方有“Get Message(s)”按钮，下方区域显示获取到的消息信息，包括Exchange、Routing Key、Redelivered等属性，以及Payload内容为“hello, spring amqp!”。该图与文档中测试流程相关，用于展示在RabbitMQ控制台获取消息的操作界面。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/TcN8bPvnBokZhcxpvsucnR3gnDb.png)



#### 2.3.4 消息接收

首先配置MQ地址，在`consumer`服务的`application.yml`中添加配置：

```YAML
spring:
  rabbitmq:
    host: 192.168.101.68 # 你的虚拟机IP
    port: 5672 # 端口
    virtual-host: / # 虚拟主机
    username: itheima # 用户名
    password: 123321 # 密码
```



然后在`consumer`服务的`com.itheima.consumer.listener`包中新建一个类`SpringRabbitListener`，代码如下：

```Java
package com.itheima.consumer.listener;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class SpringRabbitListener {
        // 利用RabbitListener来声明要监听的队列信息
    // 将来一旦监听的队列中有了消息，就会推送给当前服务，调用当前方法，处理消息。
    // 可以看到方法体中接收的就是消息体的内容
    @RabbitListener(queues = "simple.queue")
    public void listenSimpleQueueMessage(String msg) throws InterruptedException {
        System.out.println("spring 消费者接收到消息：【" + msg + "】");
    }
}
```



![这张图片展示了consumer服务的项目结构目录树，清晰呈现了该服务下各目录与文件的层级关系。项目以consumer为根目录，包含src、test、target等子目录及pom.xml文件；其中src目录下的main子目录里，有java和resources两个子目录，java目录下按com.itheima.consumer.listova的层级结构，包含SpringRabbitListener类，resources目录下则有application.yml配置文件，该结构对应文中在consumer服务的对应包中新建SpringRabbitListener类的要求，直观展示了代码与配置文件的存放位置。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/SiJ6bqva9og8Tux6VCScpFz3nDg.png)



#### 2.3.5 测试

测试流程：

启动consumer服务

在publisher服务中运行测试代码，发送MQ消息。

观察consumer控制台日志收到消息：

![这张图片展示了consumer服务启动后，RabbitMQ消费者接收消息的控制台日志内容。其中有红色框线标注的关键内容，显示成功创建了连接，且明确标注出“spring 消费者接收到消息：【hello，spring amqp!】”，清晰呈现了测试环节里消费者成功获取消息的结果，对应文档中2.3.5测试部分的消费者控制台日志观察内容。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/JwFsbBLwso3x64xer09cfhVxnrb.png)



#### 2.3.6 推模式与拉模式

在RabbitMQ中，消息传递给消费者的方式有两种：推模式（Push）和拉模式（Pull）。

1. 推模式（Push）：这是最常用的模式，在这种模式下，一旦消费者订阅了一个队列，RabbitMQ就会自动将队列中的消息发送给消费者，这种方式不需要消费者持续地询问是否有新的消息，而是由Broker在有消息时主动发送给消费者。
2. 拉模式（Pull）：在这种模式下，消费者需要主动请求Broker来获取消息。

在实践中，推模式更常见，因为它可以减少消费者的网络负载，并且可以让Broker更好地控制消息的传递速率。然而，拉模式也有其应用场景，比如当消费者想要精确控制消息获取的时候。

RabbitMQ默认采用的是推模式，但同时也提供了拉模式的支持，以满足不同的应用场景需求。



### 2.4. 数据隔离

#### 2.4.1 用户管理

RabbitMQ支持多租户，多个系统可以同时使用一个RabbitMQ。

公司内的多个不同项目同时使用。这个时候为了避免互相干扰， 我们会利用虚拟主机进行隔离，将不同项目隶属不同的虚拟主机。一般会做两件事情：

- 给每个项目创建独立的**运维账号**，将管理权限分离。
- 给每个项目创建不同的**虚拟主机**，将每个项目的数据隔离。

下边学习用户管理和虚拟主机的配置。



点击`Admin`选项卡，首先会看到RabbitMQ控制台的用户管理界面：

![图片展示了RabbitMQ控制台的用户管理界面。界面上方有RabbitMQ标志及版本信息。左侧导航栏有Overview、Connections等选项，当前选中Admin。右侧主区域显示“Users”标签，下方有“Name”“Tags”“Can access virtual host”“Has password”等字段，其中“itheima”用户被红框突出显示，其“Tags”为“administrator”，“Can access virtual host”为“/”，“Has password”有星号。该图与上下文介绍的RabbitMQ控制台用户管理界面内容相关，直观呈现了用户信息。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/CcHCby8bmocavBx2lUKcFSHun6f.png)

这里的用户都是RabbitMQ的管理或运维人员。目前只有安装RabbitMQ时添加的`itheima`这个用户。仔细观察用户表格中的字段，如下：

- `Name`：`itheima`，也就是用户名
- `Tags`：`administrator`，说明`itheima`用户是超级管理员，拥有所有权限
- `Can access virtual host`： `/`，可以访问的`virtual host`，这里的`/`是默认的`virtual host`



下边我们给黑马商城创建一个新的用户，命名为`hmall`,密码123，注意选择adminstrator

![图片展示的是RabbitMQ管理界面中“Users”页面，处于“Admin”标签下。页面中“Add a user”区域被红色框突出显示，可添加新用户。其中“Username”输入框已输入“hmall”，“Password”输入框显示“123”，“Tags”下拉框选中“administrator”。该图片与上下文紧密相关，上下文提到接下来要给名为“hmall”的用户授权，此图直观呈现了添加新用户的操作界面及所需填写的信息。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/TsZzbvlL8oeFwFxNiUPcfhdinpg.png)

你会发现此时hmall用户没有任何`virtual host`的访问权限：

![图片展示的是RabbitMQ管理界面中“Users”用户的管理页面。页面显示了所有用户信息，包括用户名、标签、可访问虚拟主机、是否有密码等字段。其中，用户名为“hmall”的用户，其“Can access virtual hosts”字段显示为“No access”，即无访问权限；“itheima”用户则有访问权限。该图片与上下文紧密相关，直观呈现了上下文中提到的hmall用户在创建后无任何虚拟主机访问权限的情况。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/SwGmb7FOkocIhixkyI6cTo7snOf.png)

接下来我们来给hmall用户授权。



#### 2.4.2 虚拟主机

下边配置虚拟主机。

我们先退出登录：

![图片展示的是RabbitMQ管理界面中“Users”选项卡下的用户管理页面。页面显示了两个用户，分别为“hmall”和“itheima”，其中“itheima”用户被高亮显示。该图片与上下文紧密相关，上下文提到在登录hmall账户后创建了“/hmall”虚拟主机，回到“users”菜单后，当前用户已具备对“/hmall”虚拟主机的访问权限，此图直观呈现了用户管理界面中“itheima”用户的状态，与上下文介绍的用户权限相关联。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/Nb1VbiPuuotxBHxJzJicf5mzn3g.png)

切换到刚刚创建的hmall用户登录，然后点击`Virtual Hosts`菜单，进入`virtual host`管理页：

![图片展示的是RabbitMQ管理界面中`Virtual Hosts`管理页。界面上方显示RabbitMQ版本为3.8.26，右上角有`Virtual host`下拉菜单，当前选中`All`。页面左侧有`Overview`、`Connections`等选项卡，右侧是`Virtual Hosts`区域，显示了`/`虚拟主机的相关信息，如状态为`running`，有1个用户，1条消息等。右下角有`Add a new virtual host`按钮。该图与上下文紧密相关，直观呈现了登录`hmall`账户后进入`virtual host`管理页的界面情况。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/VDpHb79UhoIO2Ix9E1kcyHmxnSf.png)

可以看到目前只有一个默认的`virtual host`，名字为 `/`。

 我们可以给黑马商城项目创建一个单独的`virtual host`，而不是使用默认的`/`。

![这是RabbitMQ管理后台的虚拟主机创建页面，页面顶部显示RabbitMQ版本为3.8.26、Erlang版本24.2，右上角可查看当前登录用户为hmall，且已具备对/hmall虚拟主机的访问权限。页面核心区域的红色框标注了虚拟主机创建模块，模块内的“Name”输入框填写了“hmall”，“Description”输入框标注内容为“黑马商城项目专用”，下方设有“Add virtual host”按钮，用于完成指定虚拟主机的创建。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/C6pVb2mKQoEjSAxmoUwcLNRwnv1.png)

创建完成后如图：

![图片展示了RabbitMQ管理界面中“Virtual Hosts”页面。页面显示了所有虚拟主机信息，包括Name（名称）、Users（用户）、State（状态）、Messages（消息）等列。其中，“/”虚拟主机由“itheima”用户管理，状态为“running”，消息部分显示Ready为1，Unacked为0，Total为1；“/hmall”虚拟主机由“hmall”用户管理，状态也为“running”，但其消息部分各项数据均为NaN。该图与上下文介绍的基于虚拟主机的数据隔离效果相关，直观呈现了不同虚拟主机的管理及消息状态情况。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/TPBcbxdBeorF0nxlUNYcdpC1n2e.png)

由于我们是登录`hmall`账户后创建的`virtual host`，因此回到`users`菜单，你会发现当前用户已经具备了对`/hmall`这个`virtual host`的访问权限了：

![图片展示的是RabbitMQ管理界面中“Users”页面。页面显示了所有用户信息，包括Name（用户名）、Tags（标签）、Can access virtual hosts（可访问虚拟主机）和Has password（是否有密码）等字段。其中，“hmall”用户被红色框突出显示，其Tags为“administrator”，Can access virtual hosts为“/hmall”，Has password为“·”。该图片与上文内容相关，用于说明登录“hmall”账户后创建的“virtual host”在“users”菜单中的访问权限情况。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/KR3TbFo54opK7cx4FpOc2o2onRg.png)



此时，点击页面右上角的`virtual host`下拉菜单，切换`virtual host`为 `/hmall`：

![图片展示了RabbitMQ管理界面中queues选项卡下的内容。页面上方显示RabbitMQ版本为3.8.26，Erlang版本为24.2，刷新时间为2023年6月19日13:39:20。右上角有“Refresh every 5 seconds”下拉菜单，当前选中“Virtual host: All”。下方“Virtual host”下拉菜单被红色框突出显示，选中“/hmall”。该图片与上文介绍基于虚拟主机的隔离效果相关，展示了切换虚拟主机后的界面情况。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/GRtbb5NoYokjeMxEvuFcyU8jnCc.png)

然后再次查看queues选项卡，会发现之前的队列已经看不到了：

![图片展示了RabbitMQ管理界面中“Queues”选项卡下的内容。页面显示“Queues”选项被选中，上方有“Virtual host”下拉菜单，当前选中“/hmall”。页面中部有“Pagination”和“Filter”等设置项，下方显示“... no queues ...”，表明当前没有队列。该图片与上文介绍基于虚拟主机的隔离效果相关，用于说明切换虚拟主机后，之前创建的队列在该界面中已不可见。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/Tk1EbFfTZoGJXQxsUqccaZKrn00.png)

这就是基于`virtual host `的隔离效果。



#### 2.4.3 测试

下边我们用hmall用户及新创建虚拟主机。

修改publisher及consumer的application.yml

```JavaScript
spring:
  rabbitmq:
    host: 192.168.101.68 # 你的虚拟机IP
    port: 5672 # 端口
    virtual-host: /hmall # 虚拟主机
    username: hmall # 用户名
    password: 123 # 密码
```

在/hmall中新建simple.queue队列，重新进行收发消息的测试，发现消息发发放到了virtual-host: /hmall 下的队列，如下图：

![图片展示的是RabbitMQ管理界面中queues选项卡下的队列信息。页面显示了所有队列，共2个，其中/hmall下的simple.queue队列为经典类型，状态为idle，有1条消息，消息发送速率为0.20/s。该图片与上文测试内容相关，用于直观呈现基于虚拟主机隔离效果下，hmall用户在/hmall虚拟主机下创建的队列及消息状态情况，辅助说明消息发放在虚拟主机下的效果。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/SGPLbAyhQos5ezxdrS5cXn5DnSe.png)

再启动consumer服务接收消息。



## 3 RabbitMQ工作模型

### 3.1. 工作队列模型（WorkQueues）

#### 3.1.1 介绍

在入门程序中实现的是一种最简单的工作队列模型,消费者直接绑定到队列上，如下图: 

![该图片对应文档中介绍的RabbitMQ基础工作队列模型的入门实现示意图，展现了最简单的工作队列通信结构：画面左侧的“P”代表生产者，它发出箭头指向标有“Queue”的队列模块，随后队列通过箭头指向画面右侧的“C”，“C”代表消费者。该图直观呈现了文档中描述的基础异步通信流程，即一个生产者将消息发送至队列，仅由一个消费者从队列接收消息的工作形式，也解释了此模型因仅单个消费者处理消息，处理能力较弱的特点。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/R8EybbKv2o1UXpx0GW0cBkPYnsg.png)

这种方式实现最基本的异步通信，一个生产者，一个队列，一个消费者，生产者将消息发到队列，消费者从队列接收消息。

由于绑定队列的消费者只有一个所以处理消息的能力就比较弱，下边情况将不适合这种方式：

1. 如果有大量的任务就需要多个消费者去共同处理。
2. 当生产者发送消息的速度远远大于消费者处理任务的速度此时由于消费者只有一个将造成消息堆积。

为了解决上边的问题可以使用到下边的方式，**让多个消费者绑定到一个队列，共同消费队列中的消息**。

![这张图展示了RabbitMQ工作队列模型的改进结构，生产者P将消息发送至名为Queue的队列，该队列同时绑定了两个消费者C₁与C₂，符合文档中提到的“多个消费者绑定到一个队列，共同消费队列中的消息”的解决方案，用于应对任务量大或生产者发送消息速度快导致的消息堆积问题，相比入门程序中单个消费者的工作队列模型，该结构可提升消息处理能力，避免消息堆积的情况。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/AcmCbWeCIoH7QBx4Lywc2BqhnSc.png)

#### 3.1.2 测试

##### 3.1.2.1 创建队列

接下来我们测试工作队列模型。

首先，我们在控制台创建一个新的队列，命名为`work.queue`：

![这张图片是RabbitMQ管理控制台的队列页面，展示了两个队列的信息，分别是名为simple.queue和work.queue，均处于idle状态，在Message区域中，两个队列的Ready、Unacked、Total消息数目前均为0，Message rates区域的各数值也均为0。该页面的Queues标签被高亮选中，work.queue还被红色框线重点标注，对应了文档中“创建新的队列，命名为work.queue”的操作步骤，呈现了创建后的队列在控制台的显示状态。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/E1kJb2Z2gotsv0xxIn3cJp1Cnnf.png)

##### 3.1.2.2 发送消息程序

这次我们循环发送，模拟大量消息堆积现象。

在publisher服务中的SpringAmqpTest类中添加一个测试方法：

```Java
/**
     * workQueue
     * 向队列中不停发送消息，模拟消息堆积。
     */
@Test
public void testWorkQueue() throws InterruptedException {
    // 队列名称
    String queueName = "work.queue";
    // 消息
    String message = "hello, message_";
    for (int i = 0; i < 50; i++) {
        // 发送消息，每20毫秒发送一次，相当于每秒发送50条消息
        rabbitTemplate.convertAndSend(queueName, message + i);
        Thread.sleep(20);
    }
}
```



##### 3.1.2.3 接收消息程序

要模拟多个消费者绑定同一个队列，我们在consumer服务的SpringRabbitListener中添加2个新的方法：

```Java
@RabbitListener(queues = "work.queue")
public void listenWorkQueue1(String msg) throws InterruptedException {
    System.out.println("消费者1接收到消息：【" + msg + "】" + LocalTime.now());
    Thread.sleep(20);
}

@RabbitListener(queues = "work.queue")
public void listenWorkQueue2(String msg) throws InterruptedException {
    System.err.println("消费者2........接收到消息：【" + msg + "】" + LocalTime.now());
    Thread.sleep(200);
}
```



注意到这两消费者，都设置了`Thead.sleep`，模拟任务耗时：

- 消费者1 sleep了20毫秒，相当于每秒钟处理50个消息
- 消费者2 sleep了200毫秒，相当于每秒处理5个消息



##### 3.1.2.4 测试

测试流程：

1. 启动ConsumerApplication。

观察mq控制台点击“work.queue”发现消费者程序和mq建立了两个通道（连接），在监听队列。

![图片展示了RabbitMQ控制台中“Queues”标签页下的队列信息。关键信息有：消费者数量为2，消费者容量100%；消费者1和消费者2的Consumer tag分别为“amq.tag-n074m12sVsfGmlwqQ5R0jg”和“amq.tag-KPWyKw7n8Cn37p950mUQ”；Ack required、Exclusive、Prefetch count、Active等字段均为250；Activity status均为up。该图与上文测试工作队列模型中消费者程序和mq建立通道、监听队列的内容相关，直观呈现了消费者相关信息。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/TO5tbdN2EoLz3JxzYXccW5pwnzN.png)



1. 执行publisher服务中刚刚编写的发送方法testWorkQueue。
2. 观察消费端控制台中消费消息的情况

最终结果如下：

```Java
消费者1接收到消息：【hello, message_0】21:06:00.869555300
消费者2........接收到消息：【hello, message_1】21:06:00.884518
消费者1接收到消息：【hello, message_2】21:06:00.907454400
消费者1接收到消息：【hello, message_4】21:06:00.953332100
消费者1接收到消息：【hello, message_6】21:06:00.997867300
消费者1接收到消息：【hello, message_8】21:06:01.042178700
消费者2........接收到消息：【hello, message_3】21:06:01.086478800
消费者1接收到消息：【hello, message_10】21:06:01.087476600
消费者1接收到消息：【hello, message_12】21:06:01.132578300
消费者1接收到消息：【hello, message_14】21:06:01.175851200
消费者1接收到消息：【hello, message_16】21:06:01.218533400
消费者1接收到消息：【hello, message_18】21:06:01.261322900
消费者2........接收到消息：【hello, message_5】21:06:01.287003700
消费者1接收到消息：【hello, message_20】21:06:01.304412400
消费者1接收到消息：【hello, message_22】21:06:01.349950100
消费者1接收到消息：【hello, message_24】21:06:01.394533900
消费者1接收到消息：【hello, message_26】21:06:01.439876500
消费者1接收到消息：【hello, message_28】21:06:01.482937800
消费者2........接收到消息：【hello, message_7】21:06:01.488977100
消费者1接收到消息：【hello, message_30】21:06:01.526409300
消费者1接收到消息：【hello, message_32】21:06:01.572148
消费者1接收到消息：【hello, message_34】21:06:01.618264800
消费者1接收到消息：【hello, message_36】21:06:01.660780600
消费者2........接收到消息：【hello, message_9】21:06:01.689189300
消费者1接收到消息：【hello, message_38】21:06:01.705261
消费者1接收到消息：【hello, message_40】21:06:01.746927300
消费者1接收到消息：【hello, message_42】21:06:01.789835
消费者1接收到消息：【hello, message_44】21:06:01.834393100
消费者1接收到消息：【hello, message_46】21:06:01.875312100
消费者2........接收到消息：【hello, message_11】21:06:01.889969500
消费者1接收到消息：【hello, message_48】21:06:01.920702500
消费者2........接收到消息：【hello, message_13】21:06:02.090725900
消费者2........接收到消息：【hello, message_15】21:06:02.293060600
消费者2........接收到消息：【hello, message_17】21:06:02.493748
消费者2........接收到消息：【hello, message_19】21:06:02.696635100
消费者2........接收到消息：【hello, message_21】21:06:02.896809700
消费者2........接收到消息：【hello, message_23】21:06:03.099533400
消费者2........接收到消息：【hello, message_25】21:06:03.301446400
消费者2........接收到消息：【hello, message_27】21:06:03.504999100
消费者2........接收到消息：【hello, message_29】21:06:03.705702500
消费者2........接收到消息：【hello, message_31】21:06:03.906601200
消费者2........接收到消息：【hello, message_33】21:06:04.108118500
消费者2........接收到消息：【hello, message_35】21:06:04.308945400
消费者2........接收到消息：【hello, message_37】21:06:04.511547700
消费者2........接收到消息：【hello, message_39】21:06:04.714038400
消费者2........接收到消息：【hello, message_41】21:06:04.916192700
消费者2........接收到消息：【hello, message_43】21:06:05.116286400
消费者2........接收到消息：【hello, message_45】21:06:05.318055100
消费者2........接收到消息：【hello, message_47】21:06:05.520656400
消费者2........接收到消息：【hello, message_49】21:06:05.723106700
```



可以看到消费者1和消费者2竟然每人消费了25条消息：

- 消费者1很快完成了自己的25条消息
- 消费者2却在缓慢的处理自己的25条消息。

也就是说消息是平均分配给每个消费者，并没有考虑到消费者的处理能力。导致1个消费者空闲，另一个消费者忙的不可开交。没有充分利用每一个消费者的能力，最终消息处理的耗时**远远超过了1秒**。这样显然是有问题的。

##### 3.1.2.5 能者多劳

在spring中有一个简单的配置，可以解决这个问题。我们修改consumer服务的application.yml文件，添加配置：

```YAML
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1 # 每次只能获取一条消息，处理完成才能获取下一个消息
```



再次测试，发现结果如下：

```Java
消费者1接收到消息：【hello, message_0】21:12:51.659664200
消费者2........接收到消息：【hello, message_1】21:12:51.680610
消费者1接收到消息：【hello, message_2】21:12:51.703625
消费者1接收到消息：【hello, message_3】21:12:51.724330100
消费者1接收到消息：【hello, message_4】21:12:51.746651100
消费者1接收到消息：【hello, message_5】21:12:51.768401400
消费者1接收到消息：【hello, message_6】21:12:51.790511400
消费者1接收到消息：【hello, message_7】21:12:51.812559800
消费者1接收到消息：【hello, message_8】21:12:51.834500600
消费者1接收到消息：【hello, message_9】21:12:51.857438800
消费者1接收到消息：【hello, message_10】21:12:51.880379600
消费者2........接收到消息：【hello, message_11】21:12:51.899327100
消费者1接收到消息：【hello, message_12】21:12:51.922828400
消费者1接收到消息：【hello, message_13】21:12:51.945617400
消费者1接收到消息：【hello, message_14】21:12:51.968942500
消费者1接收到消息：【hello, message_15】21:12:51.992215400
消费者1接收到消息：【hello, message_16】21:12:52.013325600
消费者1接收到消息：【hello, message_17】21:12:52.035687100
消费者1接收到消息：【hello, message_18】21:12:52.058188
消费者1接收到消息：【hello, message_19】21:12:52.081208400
消费者2........接收到消息：【hello, message_20】21:12:52.103406200
消费者1接收到消息：【hello, message_21】21:12:52.123827300
消费者1接收到消息：【hello, message_22】21:12:52.146165100
消费者1接收到消息：【hello, message_23】21:12:52.168828300
消费者1接收到消息：【hello, message_24】21:12:52.191769500
消费者1接收到消息：【hello, message_25】21:12:52.214839100
消费者1接收到消息：【hello, message_26】21:12:52.238998700
消费者1接收到消息：【hello, message_27】21:12:52.259772600
消费者1接收到消息：【hello, message_28】21:12:52.284131800
消费者2........接收到消息：【hello, message_29】21:12:52.306190600
消费者1接收到消息：【hello, message_30】21:12:52.325315800
消费者1接收到消息：【hello, message_31】21:12:52.347012500
消费者1接收到消息：【hello, message_32】21:12:52.368508600
消费者1接收到消息：【hello, message_33】21:12:52.391785100
消费者1接收到消息：【hello, message_34】21:12:52.416383800
消费者1接收到消息：【hello, message_35】21:12:52.439019
消费者1接收到消息：【hello, message_36】21:12:52.461733900
消费者1接收到消息：【hello, message_37】21:12:52.485990
消费者1接收到消息：【hello, message_38】21:12:52.509219900
消费者2........接收到消息：【hello, message_39】21:12:52.523683400
消费者1接收到消息：【hello, message_40】21:12:52.547412100
消费者1接收到消息：【hello, message_41】21:12:52.571191800
消费者1接收到消息：【hello, message_42】21:12:52.593024600
消费者1接收到消息：【hello, message_43】21:12:52.616731800
消费者1接收到消息：【hello, message_44】21:12:52.640317
消费者1接收到消息：【hello, message_45】21:12:52.663111100
消费者1接收到消息：【hello, message_46】21:12:52.686727
消费者1接收到消息：【hello, message_47】21:12:52.709266500
消费者2........接收到消息：【hello, message_48】21:12:52.725884900
消费者1接收到消息：【hello, message_49】21:12:52.746299900
```



可以发现，由于消费者1处理速度较快，所以处理了更多的消息；消费者2处理速度较慢，只处理了6条消息。而最终总的执行耗时也在1秒左右，大大提升。

正所谓能者多劳，这样充分利用了每一个消费者的处理能力，可以有效避免消息积压问题。



#### 3.1.3 小结

Work模型的使用：

- 多个消费者绑定到一个队列，同一条消息只会被一个消费者处理
- 通过设置prefetch来控制消费者预取的消息数量



### 3.2. 发布订阅模型(Publish/Subscribe)

#### 3.2.1 介绍

工作队列模型一次只能将消息发给一个队列，绑定队列的多个消息者只能有一个消费者处理消息。如果一条消息要发给多个应用程序使用工作队列模型将无法实现。举例，下图中支付成功后支付服务将消息发给交易服务和通知服务，使用工作队列模型将无法实现。

<whiteboard token="RWNpwNInXhMdnkbWJUfcgO3Lnfb"></whiteboard>

使用发布订阅模型可以实现上图的需求，**发布订阅模型可以实现一条消息发给多个队列**，每个队列绑定到同一个交换机，最终实现了向多个消费者发送一条消息,这种模式称为“发布/订阅”模型。

![图片展示了发布订阅模型的组成部分。左侧有一个标记为“P”的生产者（Publisher），向中间的交换机（Exchange，用六边形表示）发送消息。交换机将消息推送到两个队列（Q1、Q2，用矩形表示），每个队列又对应一个消费者（C1、C2，用圆形表示）。该图直观呈现了生产者、交换机、队列和消费者在发布订阅模型中的关系，与上下文对发布订阅模型组成部分的介绍相契合。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/J78fbFFpboSnAix6jmmclBtZnuh.png)

发布订阅模型里，生产者只能将消息发送到交换机，由交换机将消息推送到队列，交换机可以将消息推送给绑定它的所有队列，也可以有针对性的将消息推送给某几个队列，这就相当于有一批消费者订阅了消息，交换机根据各自的订阅去推送消息，组成部分如下：

- **Publisher**：生产者，不再发送消息到队列中，而是发给交换机
- **Exchange**：交换机，一方面，接收生产者发送的消息。另一方面，将消息推送给队列，是将消息推送给某个特别队列、递交给所有队列、或是将消息丢弃，到底如何操作，取决于交换机的类型。
- **Queue**：消息队列也与以前一样，接收消息、缓存消息。不过队列一定要与交换机绑定。
- **Consumer**：消费者，与以前一样，订阅队列，没有变化

注意：**Exchange（交换机）只负责转发消息，不具备存储消息的能力**，因此如果没有任何队列与Exchange绑定，或者没有符合路由规则的队列，那么消息会丢失！



#### 3.2.2 交换机类型

交换机是如何实现将消息推送给所有队列，还是有针对性的将消息推送给某几个队列呢？

实现不同的需求要选用不同类型的交换机，可用的交换机类型：`direct`, `topic`, `headers` 和`fanout`.

- **Fanout**：广播类型，将消息交给所有绑定到交换机的队列。
- **Direct**：直接类型，基于RoutingKey（路由key）发送给订阅了消息的队列，交换机根据routingkey去判断消息应该转发到哪个队列
- **Topic**：通配符类型（主题类型），与Direct类似，只不过RoutingKey可以使用通配符
- Headers：头匹配类型，基于MQ的消息头匹配，用的较少。

课堂中，我们讲解前面的三种交换机模式。



### 3.3. fanout交换机

#### 3.3.1 介绍

Fanout，英文翻译是扇出，我觉得在MQ中叫广播更合适。

在广播模式下，消息发送流程是这样的：

![这张图直观呈现了RabbitMQ的fanout（扇出）工作模型。图中从左至右依次为发送消息的生产者publisher，之后是Fanout交换机，交换机向下分支连接了两个队列，分别为queue1和queue2。其中queue1连接对应的消费者consumer1，queue2连接对应的消费者consumer2，所有消费者均会接收到队列传递的消息，完整展示了广播模式下消息从生产者到交换机，再分发至所有绑定队列、最终传递给消费者的流程，和文档中介绍的fanout交换机工作步骤完全对应。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/GuJQbjWWnoFqAXxFhL7cgIZXnCh.png)

- 1）  可以有多个队列
- 2）  每个队列都要绑定到Exchange（交换机）
- 3）  生产者发送的消息，只能发送到交换机
- 4）  交换机把消息发送给绑定过的所有队列
- 5）  订阅队列的消费者接收消息，每个队列订阅的消费者只有一个能拿到消息。

#### 3.3.2 测试

##### 3.3.2.1 创建队列

我们的计划是这样的：

![图片展示了RabbitMQ中fanout交换机的工作模型。左侧为publisher，中间是名为“hmall.fanout”的交换机，右侧分别有“fanout.queue1”和“fanout.queue2”两个队列，每个队列对应一个consumer，分别为“consumer1”和“consumer2”。箭头从publisher指向交换机，再分别从交换机指向两个队列，最后从队列指向对应的consumer。该图直观呈现了消息从publisher经交换机分发到队列，再由队列传递给consumer的流程。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/EcidbWDuYoYlF6xL6hxcqaJSnte.png)

- 创建一个名为` hmall.fanout`的交换机，类型是`Fanout`
- 创建两个队列`fanout.queue1`和`fanout.queue2`，绑定到交换机`hmall.fanout`



在控制台创建队列`fanout.queue1`:

![图片展示了RabbitMQ控制台中创建队列的界面。界面中“Type”下拉菜单被红色框突出显示，当前选中“Classic”类型。其他可填写的字段有“Name”（队列名称，已输入“fanout.queue1”）、“Durability”（持久性，已选“Durable”）、“Auto delete”（自动删除，已选“No”）等。下方还有“Arguments”参数设置区域，以及“Add queue”按钮。该图片与上下文关系紧密，是创建队列操作中设置队列类型等参数的展示，与上下文提到的在控制台创建队列`fanout.queue1`相呼应。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/C0WybfDIkobtEXxhiZVcAm90nsf.png)

在创建一个队列`fanout.queue2`：

![图片展示了RabbitMQ控制台中创建队列的界面。在“Add a new queue”区域，Name字段被红框突出显示，其值为“fanout.queue2”，表明正在创建名为“fanout.queue2”的队列。下方还有Durability、Auto delete等设置项，以及Arguments、Add queue等按钮。该图片与文档中“3.3.2.1创建队列”部分内容相关，用于指导在RabbitMQ控制台创建队列的操作，是创建队列步骤中的关键展示界面。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/ZhQWbiCIdoaPzMxteMJcBcQDn6d.png)

##### 3.3.2.2 创建交换机

然后再创建一个交换机hmall.fanout：

![图片展示了RabbitMQ管理界面中创建交换机的设置界面。在“Add a new exchange”区域，Name处输入“hmall.fanout”，Type下拉选择“fanout”，Virtual host为“/hmall”。该图片与文档中“3.3.2.2创建交换机”内容相关，用于说明在RabbitMQ控制台创建名为“hmall.fanout”、类型为“fanout”的交换机的操作步骤，是测试创建队列、绑定队列等后续操作的基础。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/OrBjbjWGfotYl8x3TS9c2rrWnBc.png)

##### 3.3.2.3 绑定队列到交换机

然后绑定两个队列到交换机：

![图片展示的是RabbitMQ管理界面中fanout交换机的绑定设置页面。页面上方有Overview、Connections等选项卡，当前选中“Exchanges”。在“Bindings”区域，有“Add binding from this exchange”输入框，其中“To queue”下拉框选中“fanout.queue1”，下方有“Bind”按钮。该图片与上下文关系紧密，是3.3.2.3节“绑定队列到交换机”内容的示例，用于说明将队列fanout.queue1绑定到交换机hmall.fanout的操作步骤。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/XgB1b09ERoCpsJxleM6cwxLonWf.png)

![图片展示的是RabbitMQ管理界面中fanout交换机的绑定设置页面。页面上方有Overview、Connections等选项卡，当前选中“Exchanges”。在“Bindings”区域，有“Add binding from this exchange”输入框，其中“To queue”下拉框被红色框突出显示，其右侧输入框内显示“fanout.queue2”，下方还有“Routing key”和“Arguments”输入框，最下方有“Bind”按钮。该图片与文档中“3.3.2.3绑定队列到交换机”内容相关，用于说明将队列与交换机进行绑定的操作步骤。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/EWMqbIkMDoXjAaxbDx5caRbBn4b.png)

##### 3.3.2.4 发送消息

下边实现消息发送：

在publisher服务的SpringAmqpTest类中添加测试方法：

```Java
@Test
public void testFanoutExchange() {
    // 交换机名称
    String exchangeName = "hmall.fanout";
    // 消息
    String message = "hello, everyone!";
    rabbitTemplate.convertAndSend(exchangeName, "", message);
}
```

发送成功查看mq控制台消息转发到了绑定此交换机的两个队列

![图片展示的是RabbitMQ控制台中队列界面的“Overview”视图。界面中列出了两个队列信息，分别为“/hmall168 fanout.queue1 classic”和“/hmall168 fanout.queue2 classic”，其Type均为classic，Features均为D Args，State均为idle，Ready、Unacked、Total均为1。该图片与上下文关系紧密，是在介绍在RabbitMQ中测试fanout交换机时，进入队列界面查看队列监听者情况的操作步骤，直观呈现了队列状态等关键信息。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/C03PbFZAFo8sPYxMUnpcpouRnth.png)



##### 3.3.2.5 接收消息

下边实现消息接收：

在consumer服务的SpringRabbitListener中添加两个方法，作为消费者：

```Java
@RabbitListener(queues = "fanout.queue1")
public void listenFanoutQueue1(String msg) {
    System.out.println("消费者1接收到Fanout消息：【" + msg + "】");
}

@RabbitListener(queues = "fanout.queue2")
public void listenFanoutQueue2(String msg) {
    System.out.println("消费者2接收到Fanout消息：【" + msg + "】");
}
```

将consumer服务启动起来后可以通过rabbitmq控制台查看消费者监听情况

进入队列界面，点击fanout.queue1：

![图片展示的是RabbitMQ管理控制台中队列界面。界面显示了所有队列信息，包括虚拟主机、名称、类型、状态等。其中，名为“fanout.queue1”的队列被红色箭头指向，其类型为classic，状态为idle，Ready、Unacked、Total等消息相关数据均为0，incoming、deliver / get、ack等速率数据也为0。该图片与上下文关系紧密，用于说明在测试RabbitMQ工作模型时，进入队列界面查看队列监听情况的操作结果。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/BfTybxH78oQG7qx8bPyc3U06n3g.png)

进入队列界面，查看consumers，下图表示fanout.queue1队列有一个监听者。

![图片展示的是rabbitmq控制台中fanout.queue1队列的消费者信息界面。界面中有Channel、Consumer tag、Ack required、Exclusive、Prefetch count、Active、Activity status、Arguments等列。其中，Channel列显示为192.168.101.1:59958 (3)；Consumer tag列显示为amq.ctag-YvtsHhYVWowIyc09HChueQ；Active列显示为up。该图片与上下文关系紧密，用于说明在rabbitmq控制台查看消费者监听情况时，fanout.queue1队列有一个监听者这一内容。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/Ba8mbDxriog8hxx9pW3cBGTYnbb.png)

同样的方法可以查看fanout.queue2队列的监听情况。

fanout.queue1和fanout.queue2每个队列都有一个监听者。

下边执行发送消息程序，观察控制台，下图说明每个消费者成功收到消息。

```JavaScript
消费者1接收到Fanout消息：【hello, everyone!】
消费者2接收到Fanout消息：【hello, everyone!】
```

##### 3.3.2.6 启动多个消费者实例

下边我们把consumer服务启动两个实例

![图片展示了RabbitMQ管理界面中ConsumerApplication服务的运行状态。界面左侧为导航栏，选中“Running”选项。右侧显示“ConsumerApplication (1)”和“ConsumerApplication”两个服务图标，其中“ConsumerApplication (1)”图标旁有数字“1”，表明该服务有1个实例运行。该图与上文测试启动多个消费者实例的内容相关，直观呈现了服务运行情况，辅助理解消费者实例的启动状态。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/EKNrbn4T3oJRe8xh3DPcyCfRnlb.png)

此时再观察fanout.queue1和fanout.queue2的监听者，发现每个队列有两个监听者

![图片展示了RabbitMQ管理控制台中“Consumers”（消费者）的界面。列标题包括Channel（通道）、Consumer tag（消费者标签）、Ack required（确认要求）、Exclusive（排他）、Prefetch count（预取计数）、Active（活动）、Activity status（活动状态）和Arguments（参数）。界面中有两条记录，分别对应两个消费者实例，其Channel、Consumer tag、Ack required、Exclusive、Prefetch count、Active、Activity status等信息均有显示。该图片与上下文介绍的RabbitMQ工作模型中fanout交换机测试内容相关，展示了消费者实例的配置情况。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/DVJPbbtTVoaQ8Mxta2Ycf84Dnj4.png)

此时的结果相当于下图：

![图片展示了RabbitMQ中fanout交换机的工作模型。左侧为publisher，中间是fanout交换机，右侧有两个队列，分别为queue1和queue2，每个队列对应两个消费者，分别为consumer1、consumer3和consumer2、consumer4。箭头表示消息传递方向，如publisher向fanout交换机发送消息，fanout交换机将消息分别发送到queue1和queue2，再由对应的消费者接收。该图与上下文内容对应，直观呈现了fanout交换机的分发机制。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/Fikabj5RkoGeL3xAHYncjsY1n0f.png)

此时执行发送消息程序后四个消息者都可以收到消息吗？



通过测试我们发现：

在每个队列的消费者中，发送一条消息只会有一个消费者接收到消息。

每个队列默认采用轮询的方式向消费者推送消息。



#### 3.3.3 小结

交换机的作用是什么？

- 接收publisher发送的消息
- 将消息按照规则路由到与之绑定的队列
- 不能缓存消息，路由失败，消息丢失
- FanoutExchange的会将消息路由到每个绑定的队列



### 3.4.Direct交换机

#### 3.4.1 介绍

在Fanout模式中，一条消息，会被所有订阅的队列都消费。但是，在某些场景下，我们希望不同的消息被不同的队列消费。这时就要用到Direct类型的Exchange。

![这张图是Direct交换机工作模型的示意图，展示了RabbitMQ中Direct模式的消息流转逻辑。图中左侧的publisher作为消息发送方，向中间的Direct exchange（Direct交换机）发送消息。Direct交换机分别绑定了queue1和queue2两个队列，其中queue1的bindingKey为blue和red，对应的消费者consumer1监听key为blue和red的消息；queue2的bindingKey为yellow和red，对应的消费者consumer2监听key为yellow和red的消息。当publisher发送消息时，会携带指定的RoutingKey，Direct交换机会根据消息的RoutingKey与队列绑定的bindingKey是否完全匹配，将消息路由到对应的队列，仅匹配的队列对应的消费者会接收到消息。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/DbpEbmbzUolAaTxdiIYctT5Anje.png)

在Direct模型下：

- 队列与交换机的绑定，不能是任意绑定了，而是要指定一个`RoutingKey`（路由key）
- 消息的发送方在 向 Exchange发送消息时，也必须指定消息的 `RoutingKey`。
- Exchange不再把消息交给每一个绑定的队列，而是根据消息的`Routing Key`进行判断，只有队列的`Routingkey`与消息的 `Routing key`完全一致，才会接收到消息



**案例需求如图**：

![这张图展示了Direct交换机模式下RabbitMQ的工作模型，对应文档中Direct交换机的案例需求。图中左侧的publisher向名为hmall.direct的exchange发送消息，该exchange的消息仅会转发到绑定了对应bindingKey的队列中。上方的direct.queue1绑定了hmall.direct，其bindingKey为blue和red，关联的consumer1负责接收队列消息；下方的direct.queue2同样绑定hmall.direct，bindingKey为yellow和red，关联的consumer2负责接收该队列的消息，清晰呈现了案例里交换机、队列与消费者的绑定和消息接收关系。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/IK9tbnjTLocp0IxUSjNcEM5Jnod.png)

1.  声明一个名为`hmall.direct`的交换机
2. 声明队列`direct.queue1`，绑定`hmall.direct`，`bindingKey`为`blud`和`red`
3. 声明队列`direct.queue2`，绑定`hmall.direct`，`bindingKey`为`yellow`和`red`
4.  在`consumer`服务中，编写两个消费者方法，分别监听direct.queue1和direct.queue2 
5.  在publisher中编写测试方法，向`hmall.direct`发送消息 



#### 3.4.2 测试

##### 3.4.2.1 创建队列和交换机

首先在控制台声明两个队列`direct.queue1`和`direct.queue2`，这里不再展示过程：

![图片展示的是RabbitMQ管理界面中Queues（队列）页面。页面上方有Overview、Connections、Channels、Exchanges、Queues、Admin等选项卡，当前选中Queues。表格中列出了多个队列信息，如虚拟主机（Virtual host）、名称（Name）、类型（Type）、状态（State）、消息状态（Messages）等。其中，`direct.queue1`和`direct.queue2`被红色框突出显示，其类型为classic，状态为idle，消息状态中Ready、Unacked、Total均为0。该图与上下文介绍Direct交换机相关，展示了已创建的队列信息。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/ZN4fbPB2qoNV8Wxh7kXcp0uUnij.png)

然后声明一个direct类型的交换机，命名为`hmall.direct`:

![这张图片展示了RabbitMQ管理控制台中创建交换机的界面，对应上下文里声明direct类型交换机的操作。界面内红色框选中的核心部分显示，要创建的交换机名称为“hmall.direct”，类型选择为“direct”，符合Direct交换机的配置要求；界面中还包含虚拟主机选择、持久化、自动删除等其他交换机配置选项，底部设有“Add exchange”按钮用于完成创建操作，该配置正是上下文提及的绑定队列操作的基础前提。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/XiOfb1eiUozrKmxUoaDc3WdGnSb.png)

然后使用`red`和`blue`作为key，绑定`direct.queue1`到`hmall.direct`：

![图片展示了RabbitMQ管理界面中Direct交换机`hmall.direct`的配置页面。在“Bindings”区域，显示了从该交换机添加的绑定信息，包括将`direct.queue1`队列与`hmall.direct`交换机绑定，以及使用`red`作为路由键。下方有“Add binding from this exchange”区域，可选择绑定到队列或交换机，当前选中队列`direct.queue1`，并输入了`red`作为路由键，右下角有“Bind”按钮。该图片与上文“使用`red`和`blue`作为key，绑定`direct.queue1`到`hmall.direct`”的内容对应，直观呈现了操作步骤。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/O06sbZQBPoBe8Vx6ey9cA4Qxnye.png)

![这张图片展示的是RabbitMQ控制台中“hmall.direct”交换机的绑定操作界面，该交换机位于虚拟主机“/hmall”。界面中核心操作区域为“Add binding from this exchange”模块，“To queue”下拉选项已选择队列“direct.queue1”，“Routing key”输入框已填入值“blue”，该操作是将direct.queue1队列通过路由键blue绑定至hmall.direct交换机，是文档中Direct交换机测试流程里绑定队列与交换机步骤的具体操作界面。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/EgYfb4gLkoxs3NxdfM5c8OR9ntb.png)



同理，使用`red`和`yellow`作为key，绑定`direct.queue2`到`hmall.direct`，步骤略，最终结果：

![这张图片展示的是RabbitMQ的web管理界面，处于“Exchanges”页面，显示虚拟主机为`/hmall`的`hmall.direct`交换机的绑定关系。页面的“Bindings”区域列出了该交换机与队列的绑定条目：`direct.queue1`分别通过`blue`和`red`两个路由键绑定到该交换机，`direct.queue2`分别通过`red`和`yellow`两个路由键绑定到该交换机，每个绑定条目右侧都配有“Unbind”解绑按钮，对应文档中Direct交换机绑定队列和路由键的配置结果。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/WzF1byY8io2FIAxSPgEcLxbVnCd.png)

##### 3.4.2.2 监听队列

在consumer服务的SpringRabbitListener中添加方法：

```Java
@RabbitListener(queues = "direct.queue1")
public void listenDirectQueue1(String msg) {
    System.out.println("消费者1接收到direct.queue1的消息：【" + msg + "】");
}

@RabbitListener(queues = "direct.queue2")
public void listenDirectQueue2(String msg) {
    System.out.println("消费者2接收到direct.queue2的消息：【" + msg + "】");
}
```





##### 3.4.2.3  消息发送

在publisher服务的SpringAmqpTest类中添加测试方法：

```Java
@Test
public void testSendDirectExchange() {
    // 交换机名称
    String exchangeName = "hmall.direct";
    // 消息
    String message = "红色警报！日本乱排核废水，导致海洋生物变异，惊现哥斯拉！";
    // 发送消息
    rabbitTemplate.convertAndSend(exchangeName, "red", message);
}
```

由于使用的red这个key，所以两个消费者都收到了消息：

![图片展示了在使用Direct交换机时，consumer服务的ConsumerApplication启动成功及两个消费者接收到消息的情况。时间戳为2021年6月19日21:51:17:303，显示ConsumerApplication启动耗时0.971秒。下方有两条消息，分别显示消费者1和消费者2接收到direct.queue1和direct.queue2的消息，内容均为“【红色警报！日本乱排核废水，导致海洋生物变异，惊现哥斯拉！】”。该图片与上下文紧密相关，直观呈现了Direct交换机工作时消费者接收到消息的场景。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/CwrCbNC8Qo4fqnx2qVocwYgYnJg.png)

我们再切换为blue这个key：

```Java
@Test
public void testSendDirectExchange() {
    // 交换机名称
    String exchangeName = "hmall.direct";
    // 消息
    String message = "最新报道，哥斯拉是居民自治巨型气球，虚惊一场！";
    // 发送消息
    rabbitTemplate.convertAndSend(exchangeName, "blue", message);
}
```

你会发现，只有消费者1收到了消息：

![这张图片是RabbitMQ Direct交换机测试的日志输出，对应上下文里切换为blue这个key的操作结果。图片显示了消费者收到消息的情况，其中消费者2收到了匹配red routing key的第一条消息，而消费者1除了收到该red消息外，还收到了匹配blue routing key的消息，该消息被红色框标注，内容为"最新报道，哥斯拉是居民自治巨型气球，虚惊一场！"，这验证了Direct交换机按绑定的routing key精准转发消息的特性，即仅绑定对应routing key的队列能收到消息。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/SkXubPR6Po4pQfxm4hAc2Bmjnac.png)



#### 3.4.3 小结

描述下Direct交换机与Fanout交换机的差异？

- Fanout交换机将消息路由给每一个与之绑定的队列
- Direct交换机根据RoutingKey判断路由给哪个队列
- 如果多个队列具有相同的RoutingKey，则与Fanout功能类似





### 3.5.Topic交换机

#### 3.5.1 介绍

`Topic`类型的`Exchange`与`Direct`相比，都是可以根据`RoutingKey`把消息路由到不同的队列。

只不过`Topic`类型`Exchange`可以让队列在绑定`BindingKey` 的时候使用通配符！



`BindingKey` 一般都是有一个或多个单词组成，多个单词之间以`.`分割，例如： `item.insert`



通配符规则：

- `#`：匹配任意个单词
- `*`：匹配不多不少恰好1个单词



举例：

- `item.#`：能够匹配`item.spu.insert` 或者 `item.spu`
- `item.*`：只能匹配`item.spu`



图示：

![这张图展示了RabbitMQ Topic交换机的工作示例模型，核心内容围绕Topic交换机的BindingKey匹配规则与消息路由逻辑。图中publisher向名为hmall.topic的exchange发送消息，exchange通过不同的绑定规则将消息路由至对应的队列；topic.queue1的绑定规则为“china.#”，可匹配所有以“china.”开头的RoutingKey，topic.queue2的绑定规则为“#.news”，可匹配所有以“.news”结尾的RoutingKey，最终两条队列分别将消息转发给consumer1与consumer2，直观呈现了Topic交换机通过通配符BindingKey实现消息定向路由的机制。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/OMyPb2PC5ouPt2xYpSYc5AcZnlg.png)

假如此时publisher发送的消息使用的`RoutingKey`共有四种：

- `china.news `代表有中国的新闻消息；
- `china.weather` 代表中国的天气消息；
- `japan.news` 则代表日本新闻
- `japan.weather` 代表日本的天气消息；



解释：

- `topic.queue1`：绑定的是`china.#` ，凡是以 `china.`开头的`routing key` 都会被匹配到，包括：

  - `china.news`
  - `china.weather`
- `topic.queue2`：绑定的是`#.news` ，凡是以 `.news`结尾的 `routing key` 都会被匹配。包括:

  - `china.news`
  - `japan.news`



#### 3.5.2 测试

##### 3.5.2.1 创建队列和交换机

接下来，我们就按照上图所示，来演示一下Topic交换机的用法。

首先，在控制台按照图示例子创建队列、交换机，并利用通配符绑定队列和交换机。



创建队列：`topic.queue1`、`topic.queue`2

创建交换机：hmall.topic

绑定队列：

`topic.queue1`：绑定的是`china.#`

`topic.queue2`：绑定的是`#.news`



最终结果如下：

![图片展示了RabbitMQ管理界面中Topic交换机“hmall.topic”的绑定信息。在“Bindings”区域，显示了两个绑定关系，分别是“topic.queue1”绑定“china.#”，“topic.queue2”绑定“#.news”。每个绑定关系右侧都有“Unbind”按钮。该图与上文“3.5.2测试”中创建队列和交换机、绑定队列的内容相关，直观呈现了上文提到的绑定配置情况。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/Mph5bWukRoJiWOxn349cjnJ1nJd.png)

##### 3.5.2.2 消息接收

在consumer服务的SpringRabbitListener中添加方法：

```Java
@RabbitListener(queues = "topic.queue1")
public void listenTopicQueue1(String msg){
    System.out.println("消费者1接收到topic.queue1的消息：【" + msg + "】");
}

@RabbitListener(queues = "topic.queue2")
public void listenTopicQueue2(String msg){
    System.out.println("消费者2接收到topic.queue2的消息：【" + msg + "】");
}
```



##### 3.5.2.3 消息发送

在publisher服务的SpringAmqpTest类中添加测试方法：

```Java
/**
 * topicExchange
 */
@Test
public void testSendTopicExchange() {
    // 交换机名称
    String exchangeName = "hmall.topic";
    // 消息
    String message = "喜报！孙悟空大战哥斯拉，胜!";
    // 发送消息
    rabbitTemplate.convertAndSend(exchangeName, "china.news", message);
}
```

执行测试方法，routingKey为china.news可以匹配topic.queue1和topic.queue2，日志输出：

```Java
消费者1接收到topic.queue1的消息：【喜报！孙悟空大战哥斯拉，胜!】
消费者2接收到topic.queue2的消息：【喜报！孙悟空大战哥斯拉，胜!】
```

更改routingKey为china.weather，只可以匹配到topic.queue1，日志输出：

```Java
消费者1接收到topic.queue1的消息：【喜报！孙悟空大战哥斯拉，胜!】
```



#### 3.5.3 小结



描述下Direct交换机与Topic交换机的差异？

- Topic交换机接收的消息RoutingKey必须是多个单词，以 **`.`** 分割
- Topic交换机与队列绑定时的bindingKey可以指定通配符
- `#`：代表0个或多个词
- `*`：代表1个词



### 3.6.基于注解声明队列和交换机

在之前我们都是基于RabbitMQ控制台来创建队列、交换机。但是在实际开发时，队列和交换机是程序员定义的，将来项目上线，又要交给运维去创建。那么程序员就需要把程序中运行的所有队列和交换机都写下来，交给运维。在这个过程中是很容易出现错误的。

因此推荐的做法是由程序启动时检查队列和交换机是否存在，如果不存在自动创建。

#### 3.8.1. 基本API的方式（了解）

##### 3.8.1.1 介绍

SpringAMQP提供了一个Queue类，用来创建队列：

![图片展示了SpringAMQP中Queue类的相关信息。Queue类是一个简单的容器，用于收集描述队列的信息，常与AmqpAdmin配合使用。作者为Mark Pollack和Gary Russell。该类继承了AbstractDeclarable并实现了Cloneable接口。此图片与文档中介绍SpringAMQP提供Queue类用来创建队列的内容相关，是对Queue类的定义和基本信息的直观呈现。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/BaRtbx49CoL8ncxqrj5cO9eRnGe.png)



SpringAMQP还提供了一个Exchange接口，来表示所有不同类型的交换机：

![图片展示了RabbitMQ中Exchange的继承关系。Exchange类是所有不同类型的交换机的基类，其下有AbstractExchange抽象类。AbstractExchange类下有TopicExchange、CustomExchange、FanoutExchange、DirectExchange、HeadersExchange等具体类型。其中，FanoutExchange被红色框突出显示。该图与上下文紧密相关，上下文在介绍SpringAMQP提供的Exchange接口时，提到可通过ExchangeBuilder简化创建过程，此图直观呈现了不同Exchange类型及其继承关系，帮助理解相关内容。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/UkFabxNT6oGWAUxuzARc38T4noh.png)

我们可以自己创建队列和交换机，不过SpringAMQP还提供了ExchangeBuilder来简化这个过程：

![这张图片展示了SpringAMQP中用于简化交换机创建的ExchangeBuilder相关的API方法列表，包含多个核心方法。其中，用于创建不同类型交换机的方法被重点突出显示，分别有构建直连交换机的directExchange、主题交换机的topicExchange、广播交换机的fanoutExchange、头交换机的headersExchange，以及用于设置交换机特性的autoDelete、durable、withArgument等方法，构造方法ExchangeBuilder可指定交换机名称与类型，这些方法能简化RabbitMQ交换机的声明流程，对应文档中提及的用ExchangeBuilder简化交换机创建的内容。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/XAaHb9RifouZ6jxPEEicdPiNn4f.png)

而在绑定队列和交换机时，则需要使用BindingBuilder来创建Binding对象：

![这张图片是名为BindingBuilder的工具类相关内容，展示了该类的结构与可用方法，其上方的蓝色标题栏标注了类名“BindingBuilder”。图片中包含BindingBuilder的构造方法“BindingBuilder()”，以及两个核心方法：一个是绑定队列的方法“bind(Queue): DestinationConfigurer”，另一个是绑定交换机的方法“bind(Exchange): DestinationConfigurer”。这些内容对应了RabbitMQ中绑定队列与交换机时创建Binding对象的相关工具，是文档中介绍的SpringAMQP用于构建Binding对象的核心类及方法的具象展示，能够辅助理解文档中关于BindingBuilder的功能说明。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/Jh90bDCB8oRF54xo1A0cDCF7nch.png)

##### 3.8.1.2 API方式声明举例

**下边是fanout示例**

在consumer中创建一个类，声明队列和交换机：

```Java
package com.itheima.consumer.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FanoutConfig {
    /**
     * 声明交换机
     * @return Fanout类型交换机
     */
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("hmall.fanout");
    }

    /**
     * 第1个队列
     */
    @Bean
    public Queue fanoutQueue1(){
        return new Queue("fanout.queue1");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange){
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }

    /**
     * 第2个队列
     */
    @Bean
    public Queue fanoutQueue2(){
        return new Queue("fanout.queue2");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue2(Queue fanoutQueue2, FanoutExchange fanoutExchange){
        return BindingBuilder.bind(fanoutQueue2).to(fanoutExchange);
    }
}
```

**下边是direct示例**

direct模式由于要绑定多个KEY，会非常麻烦，每一个Key都要编写一个binding：

```Java
package com.itheima.consumer.config;

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DirectConfig {

    /**
     * 声明交换机
     * @return Direct类型交换机
     */
    @Bean
    public DirectExchange directExchange(){
        return ExchangeBuilder.directExchange("hmall.direct").build();
    }

    /**
     * 第1个队列
     */
    @Bean
    public Queue directQueue1(){
        return new Queue("direct.queue1");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue1WithRed(Queue directQueue1, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue1).to(directExchange).with("red");
    }
    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue1WithBlue(Queue directQueue1, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue1).to(directExchange).with("blue");
    }

    /**
     * 第2个队列
     */
    @Bean
    public Queue directQueue2(){
        return new Queue("direct.queue2");
    }

    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue2WithRed(Queue directQueue2, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue2).to(directExchange).with("red");
    }
    /**
     * 绑定队列和交换机
     */
    @Bean
    public Binding bindingQueue2WithYellow(Queue directQueue2, DirectExchange directExchange){
        return BindingBuilder.bind(directQueue2).to(directExchange).with("yellow");
    }
}
```



#### 3.8.2. 基于注解声明

基于@Bean的方式声明队列和交换机比较麻烦，Spring还提供了基于注解方式来声明。



例如，我们同样声明Direct模式的交换机和队列：

```Java
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue1"),
    exchange = @Exchange(name = "hmall.direct", type = ExchangeTypes.DIRECT),
    key = {"red", "blue"}
))
public void listenDirectQueue1(String msg){
    System.out.println("消费者1接收到direct.queue1的消息：【" + msg + "】");
}

@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "direct.queue2"),
    exchange = @Exchange(name = "hmall.direct", type = ExchangeTypes.DIRECT),
    key = {"red", "yellow"}
))
public void listenDirectQueue2(String msg){
    System.out.println("消费者2接收到direct.queue2的消息：【" + msg + "】");
}
```

测试方法非常简单，把hmall.direct交换机及绑定的队列通过rabbitmq控制台删除，重启consumer服务后hmall.direct交换机及绑定的队列会自动创建。

是不是简单多了。



再试试Topic模式：

```Java
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "topic.queue1"),
    exchange = @Exchange(name = "hmall.topic", type = ExchangeTypes.TOPIC),
    key = "china.#"
))
public void listenTopicQueue1(String msg){
    System.out.println("消费者1接收到topic.queue1的消息：【" + msg + "】");
}

@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "topic.queue2"),
    exchange = @Exchange(name = "hmall.topic", type = ExchangeTypes.TOPIC),
    key = "#.news"
))
public void listenTopicQueue2(String msg){
    System.out.println("消费者2接收到topic.queue2的消息：【" + msg + "】");
}
```

### 3.7.消息转换器

#### 3.9.1 测试默认转换器

Spring的消息发送代码接收的消息体是一个Object：

![这张图片展示了AmqpTemplate.java文件的部分代码内容，具体是convertAndSend方法的定义，该方法的作用是将Java对象转换为AMQP消息并发送到指定的交换机。方法的参数列表中明确包含名为object message的参数，这个参数对应上下文所述的、Spring消息发送代码接收的消息体，正是后续要通过消息转换器进行序列化处理的对象，也直观呈现了JDK序列化特性所处理的目标对象类型。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/IILhbq3WqoCAXVxngaxc7dkankc.png)

而在数据传输时，它会把你发送的消息序列化为字节发送给MQ，接收消息的时候，还会把字节反序列化为Java对象。

只不过，默认情况下Spring采用的序列化方式是JDK序列化。众所周知，JDK序列化存在下列问题：

- 数据体积过大
- 有安全漏洞
- 可读性差



我们来测试一下。



1）创建测试队列:object.queue

![这是RabbitMQ管理界面的Queues页面，版本为RabbitMQ 3.8.26，当前展示了所有队列的信息，本次测试中创建的队列“object.queue”被红色框标注出来，该队列所属虚拟主机为“/hmall”，类型为classic，功能标识为D，状态为idle，当前无待处理消息，符合测试默认消息转换器时需要创建该测试队列的操作需求，用于后续观察发送Map对象时的消息体格式。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/HA0RbhS8xo9kXpxXTO7cDgfcnTg.png)



2）发送消息

我们在publisher模块的SpringAmqpTest中新增一个消息发送的代码，发送一个Map对象：

```Java
@Test
public void testSendMap() throws InterruptedException {
    // 准备消息
    Map<String,Object> msg = new HashMap<>();
    msg.put("name", "柳岩");
    msg.put("age", 21);
    // 发送消息
    rabbitTemplate.convertAndSend("object.queue", msg);
}
```

注意，这里我们先不要给这个队列添加消费者，我们要查看消息体的格式。



发送消息后查看控制台：

![图片展示的是RabbitMQ管理控制台中Get messages界面。在Encoding下拉菜单中选择“Auto string / base64”，Payload部分以base64编码显示了消息内容，其中“content_type: application/x-java-serialized-object”以红色框突出显示。该图片与文档中测试默认转换器的内容相关，用于直观呈现发送Map对象消息后，控制台中消息体的格式情况，以验证JDK序列化存在的问题。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/JyVebsqJGoJnjHxBuU7cy7bJnOd.png)

可以看到消息格式非常不友好。

#### 3.9.2 配置JSON转换器

##### 3.9.2.1 引入依赖

显然，JDK序列化方式并不合适。我们希望消息体的体积更小、可读性更高，因此可以使用JSON方式来做序列化和反序列化。



在`publisher`和`consumer`两个服务的父工程中都引入依赖：

```XML
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.9.10</version>
</dependency>
```

注意，如果项目中引入了`spring-boot-starter-web`依赖，则无需再次引入`Jackson`依赖。

##### 3.9.2.2 配置消息转换器

配置消息转换器，在`publisher`和`consumer`两个服务中添加一个Bean即可：

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



##### 3.9.2.3 测试

此时，我们到MQ控制台**删除**`object.queue`中的旧的消息。然后再次执行刚才的消息发送的代码，到MQ的控制台查看消息结构：

![图片展示的是MQ控制台中获取消息的界面。上方显示“Get Message(s)”，下方“Message 1”处，Exchange为(AMQP default)，Routing Key为object.queue，Properties部分有message_id、priority等信息，content_encoding为UTF - 8，content_type为application/json。Payload部分显示了JSON格式的数据，包含name、age等字段。该图片与上下文关系为：在配置消息转换器并测试后，通过此界面查看消息结构，以验证消息转换器配置是否正确。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/TAqabw3lLoASL1xhuWMcAkMBntg.png)

#### 3.9.3 消费者接收Object

我们在consumer服务中定义一个新的消费者，publisher是用Map发送，那么消费者也一定要用Map接收，格式如下：

```Java
@RabbitListener(queues = "object.queue")
public void listenSimpleQueueMessage(Map<String, Object> msg) throws InterruptedException {
    System.out.println("消费者接收到object.queue消息：【" + msg + "】");
}
```



### 3.8 面试题

RabbitMQ 的工作模型有哪些？



如何解决MQ消息堆积问题？



## 4 高级队列

### 4.5. LazyQueue

#### 4.5.1 介绍

在默认情况下，RabbitMQ会将接收到的信息先保存在内存中然后再保存至磁盘，以降低消息收发的延迟。但在某些特殊情况下，这会导致消息积压，比如：

- 消费者宕机或出现网络故障
- 消息发送量激增，超过了消费者处理速度
- 消费者处理业务发生阻塞



一旦出现消息堆积问题，RabbitMQ的内存占用就会越来越高，直到触发内存预警上限。此时RabbitMQ会将内存消息刷到磁盘上，这个行为称为`PageOut`. `PageOut`会耗费一段时间，并且会阻塞队列进程。因此在这个过程中RabbitMQ不会再处理新的消息，生产者的所有请求都会被阻塞。



为了解决消息堆积问题，从RabbitMQ的3.6.0版本开始，就增加了Lazy Queues的模式，也就是惰性队列。惰性队列的特征如下：

- 接收到消息后直接存入磁盘而非内存
- 消费者要消费消息时才会从磁盘中读取并加载到内存（也就是懒加载）
- 支持数百万条的消息存储



**LazyQueue的特点**：

Lazy Queue 通过将大部分消息存储在磁盘上而不是内存中来减少内存占用。它使用内存映射文件技术来实现高效的磁盘访问。

即使在处理大量消息的情况下，Lazy Queue 也能保持较高的性能。通过减少内存占用，可以避免频繁的垃圾回收操作，从而提高整体性能。

**LazyQueue应用场景**：

当消息队列非常大并且消息数量非常多时，使用 Lazy Queue 可以显著减少内存使用量，从而提高系统性能。例如，在日志聚合系统中，需要处理大量的日志消息，使用 Lazy Queue 可以有效管理内存资源。



#### 4.5.2 控制台配置Lazy模式

在添加队列的时候，添加`x-queue-mod=lazy`参数即可设置队列为Lazy模式：

![这张图片展示了RabbitMQ控制台添加新队列的配置界面，对应文档中通过控制台配置Lazy模式队列的内容。界面中已经设置了虚拟主机为/hmall，队列类型为Classic，队列名称为lazy.queue，持久化属性为Durable；在参数区域，红框标注了`x-queue-mode`参数，其值被设为lazy，且数据类型选择为String，鼠标光标正指向下方的Lazy mode选项，说明通过该界面的参数配置即可完成Lazy模式队列的创建，契合文档中“添加队列时配置`x-queue-mod=lazy`参数设置Lazy模式”的说明。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/GDLSbFXP2onHVKx0600cd0rxn1c.png)



#### 4.5.3 代码配置Lazy模式

在利用SpringAMQP声明队列的时候，添加`x-queue-mod=lazy`参数也可设置队列为Lazy模式，

基于注解来声明队列并设置为Lazy模式：

```Java
//掌握注解方式
@RabbitListener(queuesToDeclare = @Queue(
        name = "lazy.queue",
        durable = "true",
        arguments = @Argument(name = "x-queue-mode", value = "lazy")
))
public void listenLazyQueue(String msg){
    log.info("接收到 lazy.queue的消息：{}", msg);
}
```



#### 4.5.4 测试

测试流程：

创建一个惰性队列，一次写入100条消息

```JavaScript
@Test
public void testLazyQueue() {
    for (int i = 0; i < 100; i++) {
        // 队列名称
        String queueName = "lazy.queue";
        // 消息
        String message = "hello, spring amqp!"+i;
        // 发送消息
        rabbitTemplate.convertAndSend(queueName, message);
    }

}
```

写入成功观察队列信息：

![这张图片展示了惰性队列测试后的控制台详情信息，该队列的Features部分已配置`x-queue-mode: lazy`参数，符合惰性队列的设置要求。消息统计区域明确标注“In memory”对应数值为0，“Persistent”对应数值为100，结合上下文说明这100条消息全部存储在磁盘，内存中无消息，与测试流程里惰性队列的存储结果完全对应。该队列处于idle状态，无消费者，消息总数、就绪数均为100，未确认数为0，消息总大小为2.2KiB，符合惰性队列的存储特性。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/FJjabAD7Wo6VsGxoVaNcVcsbnMh.png)

100条消息全部在磁盘，内存消息个数为0.



再向非惰性队列写入100条消息观察消息存储情况

![图片展示的是RabbitMQ管理控制台中队列的详细信息页面。页面显示队列状态为idle，消费者数量为0，消费者容量为0%。消息总数为100，其中内存消息100条，持久性消息100条，消息体大小均为2.7 kiB，处理内存为144 kiB。该图片与文档中“测试流程”部分相关，用于说明在创建惰性队列并写入100条消息后，观察到100条消息全部在磁盘，内存消息个数为0的情况。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/EG3hbznK8oIvBQxYdCScbsrQnme.png)

100条消息分别在内存和磁盘存储。



#### 4.5.5 小结

**LazyQueue的特点**：

Lazy Queue 惰性队列将大部分消息存储在磁盘上而不是内存中来减少内存占用。

即使在处理大量消息的情况下，Lazy Queue 也能保持较高的性能。通过减少内存占用，可以避免频繁的垃圾回收操作，从而提高整体性能。

**LazyQueue应用场景**：

当消息队列非常大并且消息数量非常多时，使用 Lazy Queue 可以显著减少内存使用量，从而提高系统性能。



### 4.6. 优先级队列

#### 4.6.1 介绍

自从 RabbitMQ 3.5.0 版本起，引入了优先级队列的功能，允许开发者根据消息的重要程度来设定不同的优先级。这在处理紧急或重要的消息时非常有用。

通过设置 `x-max-priority` 参数，可以实现这一功能。然而，在消费速度远高于生产速度且消息队列中没有积压消息的情况下，优先级的作用就不那么明显了。



应用场景：

1. 交易处理

- 场景: 在金融交易系统中，需要处理不同类型的交易请求，如紧急交易和常规交易。
- 实现: 使用优先级队列，将紧急交易请求设置为最高优先级，常规交易设置为较低优先级。
- 好处: 确保紧急交易能够被优先处理，提高了交易系统的响应速度。

1. 任务调度

- 场景: 在一个任务调度系统中，可能存在不同优先级的任务需要被处理。
- 实现: 使用优先级队列，高优先级的任务（如紧急任务或关键任务）会被优先处理。
- 好处: 保证了重要任务能够被及时处理，提高了系统的响应能力和可靠性。



#### 4.6.2 创建优先级队列

创建优先级队列：priority.queue

设置x-max-priority ，定义优先级的最大值

![图片展示了RabbitMQ管理界面中创建优先级队列时设置参数的界面。在Arguments区域，x - max - priority参数被突出显示，其值为10，类型为Number。该图片与文档中“创建优先级队列：priority.queue”部分内容相关，用于说明设置x - max - priority，定义优先级最大值的操作，是创建优先级队列时设置参数的示例。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/Qrf6bWXixoLk9IxmGxpcEePBnOf.png)

#### 4.6.3 测试

向优先级队列发100条消息，优先级用随机数生成，最大为10，优先级越大越优先出队。

```JavaScript
//向优先级队列发100条消息
@Test
public void testPriorityQueue() {
    for (int i = 0; i < 100; i++) {
        // 队列名称
        String queueName = "priority.queue";
        //优先级随机数为1到10
        int priority = new Random().nextInt(10) + 1;
        // 消息
        String message = "hello, spring amqp!"+priority;
        // 向优先级队列发送消息
        rabbitTemplate.convertAndSend(queueName, message, msg -> {
            MessageProperties messageProperties = msg.getMessageProperties();
            //设置优先级
            messageProperties.setPriority(priority);
            return msg;
        });
    }

}
```

发送成功观察控制台：

![该图片是MQ测试优先级队列时的控制台界面内容，核心显示了优先级队列的相关信息。其中明确标注了队列名称为priority.queue，类型为classic，还存在“D”“Pri”等标识，同时能看到队列的idle状态，以及100、0、100等相关数值，这些数值对应消息的相关统计数据，直观呈现了测试优先级队列运行时的控制台状态，和文档中测试优先级队列、观察控制台的内容相呼应，体现了该队列的运行情况。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/LRHRbAqQRoflTAxtVAOctoF4nBh.png)

运行消费程序

```Java
//监听优先级队列
@RabbitListener(queuesToDeclare = @Queue(
        name="priority.queue",
        durable = "true",
        arguments = @Argument(name = "x-max-priority", value = "10",type="java.lang.Integer")))
public void listenPriorityQueue(String msg) throws InterruptedException {
  System.out.println("消费者接收到priority.queue消息：【" + msg + "】");
}
```

观察控制台，按优先级消费消息，优先级高（priority值越大优先级越高）的最先消费。



## 5.业务改造

### 5.1 需求分析

案例需求：改造余额支付功能，将支付成功后基于OpenFeign远程调用交易服务更新订单状态接口由同步调用改为基于RabbitMQ的异步通知。

如图：

![这张图展示了业务改造后的消息队列（MQ）交易流程，结合交易服务的改造需求：支付服务完成后发送含订单ID的消息，通过`pay.direct`类型的交换机，携带`pay.success`的路由键，将消息投递到绑定的`pay.success.queue`队列，交易服务监听该队列获取消息后，将订单状态更新为已支付。其余关联服务中，用户服务会扣减余额，通知服务向用户发送短信，积分服务增加用户积分，各服务操作后均对应更新自身的数据库。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/RPUzbA0cHoQVUZxVQLKc1E4QnZg.png)

说明：目前没有通知服务和积分服务，因此我们只关注交易服务，步骤如下：

- 定义`direct`类型交换机，命名为`pay.direct`
- 定义消息队列，命名为pay.success.queue
- 将pay.success.queue与`pay.direct`绑定，`BindingKey`为`pay.success`
- 支付成功时不再调用交易服务更新订单状态的接口，而是发送一条消息到`pay.direct`，发送消息的`RoutingKey`  为`pay.success`，消息内容是订单id
- 交易服务监听pay.success.queue 队列，接收到消息后更新订单状态为已支付



### 5.2 配置MQ

不管是生产者还是消费者，都需要配置MQ的基本信息。分为两步：

1）添加依赖：

在支付服务、交易服务添加amqp依赖。

```XML
  <!--消息发送-->
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
  </dependency>
```



2）配置MQ地址：

在支付服务、交易服务添加MQ地址配置。

```YAML
spring:
  rabbitmq:
    host: 192.168.101.68 # 你的虚拟机IP
    port: 5672 # 端口
    virtual-host: /hmall # 虚拟主机
    username: hmall # 用户名
    password: 123 # 密码
```



### 5.3 发送消息

#### 5.3.1 常量类

在common模块配置常量类，包括支付交换机，支付成功key等信息

```JavaScript
package com.hmall.common.constants;

/**
 * @author Mr.M
 * @version 1.0
 * @description MQ常量配置
 * @date 2024/8/30 11:45
 */
public interface MqConstants {
    String PAY_EXCHANGE_NAME = "pay.direct";
    String PAY_SUCCESS_KEY = "pay.success";
    String PAY_SUCCESS_QUEUE = "pay.success.queue";
}
```

#### 5.3.2 编写代码

修改`pay-service`服务下的`com.hmall.pay.service.impl.PayOrderServiceImpl`类中的`tryPayOrderByBalance`方法：

屏蔽tradeClient.markOrderPaySuccess(po.getBizOrderNo())

支付成功后发送消息

代码如下：

```Java
private final RabbitTemplate rabbitTemplate;

@Override
@Transactional
public void tryPayOrderByBalance(PayOrderFormDTO payOrderDTO) {
   Long userId = UserContext.getUser();
    // 1.查询支付单
    PayOrder po = getById(payOrderDTO.getId());
    // 2.判断状态
    if(!PayStatus.WAIT_BUYER_PAY.equalsValue(po.getStatus())){
        // 订单不是未支付，状态异常
        throw new BizIllegalException("交易已支付或关闭！");
    }
    // 3.尝试扣减余额
    userClient.deductMoney(userId,payOrderDTO.getPw(), po.getAmount());
    // 4.修改支付单状态
    boolean success = markPayOrderSuccess(payOrderDTO.getId(), LocalDateTime.now());
    if (!success) {
        throw new BizIllegalException("交易已支付或关闭！");
    }
    // 5.修改订单状态
    // tradeClient.markOrderPaySuccess(po.getBizOrderNo());
    try {
        rabbitTemplate.convertAndSend(MqConstants.PAY_EXCHANGE_NAME, MqConstants.PAY_SUCCESS_KEY, po.getBizOrderNo());
    } catch (Exception e) {
        log.error("支付成功的消息发送失败，支付单id：{}， 交易单id：{}", po.getId(), po.getBizOrderNo(), e);
    }
}
```



### 5.4 接收消息

在trade-service服务中定义一个消息监听类，接收到消息，调用方法更新订单状态为已支付。

![这张图片展示了IDEA开发工具中trade-service服务的项目目录结构，目录层级从外层到内层依次展开，可见该服务包含logs、src等子目录。在src/main/java/com.hmall.trade路径下，有多个功能子目录，其中被红框标注的listener目录中，存在名为PayStatusListener的类，这正是文档中提及的在trade-service服务里定义的消息监听类，该类用于接收支付成功的消息，进而调用方法更新订单状态为已支付。](../../../../../Attachments/development/architecture/microservices/xiao-xi-dui-lie-mq/JFxJbDmx8oSqkOx6EgscI8a3nlh.png)

其代码如下：

```Java
package com.hmall.trade.listener;

import com.hmall.trade.service.IOrderService;
import lombok.RequiredArgsConstructor;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
@RequiredArgsConstructor
public class PayStatusListener {

    private final IOrderService orderService;

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = MqConstants.PAY_SUCCESS_QUEUE, durable = "true"),
            exchange = @Exchange(name = MqConstants.PAY_EXCHANGE_NAME, type = "direct"),
            key = MqConstants.PAY_SUCCESS_KEY
    ))
    public void listenPaySuccess(Long orderId){
        orderService.markOrderPaySuccess(orderId);
    }
}
```



## 作业

- 完成课堂上的五种消息模型
- 完成业务改造
- 练习基于api的方式声明交换机和队列
- 练习高级队列（惰性队列、优先级队列）

---
> 来源：飞书技术知识库（脱敏版） · 04 微服务与中间件 / 消息队列 MQ / 04 MQ基础 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/Pxt4wV5f0iPSkqkIPKncWU3UnYb)
