---
type: memo
date: 2026-11-06
aliases:
  - "第五章 搜索模块"
---
# 第五章 搜索模块

## 学习目标：

1. 能够说出项目为什么要用Elasticsearch
2. 能够说出服务搜索模块的功能列表
3. 能够说出项目中进行索引同步的流程
4. 能够配置Canal加MQ同步环境
5. 能够完成服务变更实时同步开发
6. 能够开发家政服务搜索接口
7. 能够说出家政项目搜索模块的设计方案
8. 能够开发电商项目搜索接口
9. 能够说出电商项目搜索模块的设计方案



家政项目实战：

服务变更实时同步开发

家政服务搜索接口



电商项目实战：

电商项目搜索接口

电商项目搜索聚合接口



## 代码环境：

jzo2o-foundations：dev_02

mall-plus: dev_01



## 1 家政项目服务搜索

### 1.1 服务搜索技术方案

目标：理解本项目的搜索技术方案

#### 1）需求分析

服务搜索的入口有两处：

1. 在门户最上端的搜索入口对服务信息进行搜索。

如下图：

![图片展示了云岚到家平台的服务搜索页面。页面上方显示“北京”和“家电维修”搜索框，右上角有“...”和“O”图标。中部有“云岚到家平台，给你最贴心的最专业的上门服务”文字及“banner图”。下方有“一级分类icon”及多个服务项icon，如保洁清洗、日常保洁等。最下方是“精选推荐”，有“专业、贴心的上门服务”文字及“商品图”，并显示“空调清洗服务 ¥208/次”及“立即预约”按钮。该图与文档中服务搜索入口在门户最上端搜索入口处的描述对应。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/RMGKbpgm8om1xtxM3dAcMZKcnlf.png)

在**第2部分**触发搜索框进入搜索页面，输入关键字进行搜索，如下图：



![这是家政项目服务搜索技术方案中，第二处触发搜索后进入的搜索页面界面。页面顶部显示“搜索”标题，左侧有返回箭头，右侧是更多操作图标和关闭按钮，右上角还有“取消”选项。搜索框内已输入关键字“保洁”，搜索框右侧设有清除输入内容的“×”按钮，搜索框下方展示了推荐的“日常保洁”搜索建议，该界面对应在搜索页面输入关键词查找家政服务的操作场景。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/EPt8b6NwMoallLxnUgrc5GOynVe.png)

1. 在门户最下方点击“全部服务”进入全部服务界面。

如下图：

点击服务分类查询分类下的服务。

![图片展示了家政项目服务搜索中在门户最下方点击“全部服务”进入全部服务界面后，点击服务分类查询分类下的服务时的界面。左侧有“服务分类”栏，包含“保洁清”和“家居服务”选项，其中“保洁清”被红色框突出显示。右侧是“全部服务”标题，下方有“日常保洁”等服务项，其中“日常保洁”被红色框突出显示。该图片与上下文介绍的服务搜索入口相关，直观呈现了在门户最下方点击“全部服务”后，通过服务分类查询分类下的服务项进行搜索的场景。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/FIwjb2yCVoikQWxSzmYcQE4tndb.png)

#### 2）技术方案

##### **使用ES进行全文检索**

要实现服务搜索使用什么技术呢？

根据需求分析，对服务进行搜索除了根据服务类型查询其下的服务以外还需要根据关键字去搜索与关键字匹配的服务。

通过关键字去匹配服务的哪些信息呢？比如：输入关键字“家庭保洁”，它会去匹配服务相关的信息，比如：服务类型的名称、服务项的名称，甚至根据需要也可能去匹配服务介绍的信息，只要与“家庭保洁”相关的服务都会展示出来。如下效果：

![这张图是家政项目服务搜索的界面，页面顶部显示“搜索”字样，右侧设有“取消”选项。搜索输入框内已输入“家庭保洁”关键词，呈现了全文检索的初始输入状态，该界面正是文档中所提及的使用ES进行全文检索的对应效果展示，契合服务搜索技术方案里的全文检索场景。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/YwfLbPGpioVouEx2F6ncq7gBnCb.png)



这里最关键的是根据关键字去匹配，上图的搜索效果是一种全文检索方式，在搜索“家庭保洁”关键字时会对关键字先分词，分为“家庭”和“保洁”，再根据分好的词去匹配索引库中的服务类型的名称、服务项的名称、服务项的描述等字段。

如果要实现全文检索且对接口性能有一定的要求，最常用的是Elasticsearch，Elasticsearch是基于倒排索引的原理，正排索引是从文章中找词，倒排索引是根据词去找文章，本项目使用ES完成服务搜索功能的开发。

##### **索引同步方案**

如果要使用ES去搜索服务信息需要提前对服务信息在ES中创建索引，运营端在管理服务时是将服务信息保存在数据库，如何对数据库中的服务信息去创建索引，保证数据库中的信息与ES的索引信息同步呢，本节对索引同步的方案进行分析与确定。

<whiteboard token="Rc1zwhv1yhURtHbmRuUcsNFRntf"></whiteboard>

**方案1： 添加服务信息维护索引**

在服务项的增删改查Service方法中添加维护ES索引的代码。

在区域服务的增删改查Service方法中添加维护ES索引的代码。

例如下边的代码：

```Java
public Serve onSale(Long id){
    //操作serve表
    //添加向ES创建索引的代码
}
```

首先上边的代码是在原有业务方式的基础上添加索引同步的代码，增加代码的复杂度不方便维护，扩展性差。

其次上边的代码存在分布式事务，操作serve表会访问数据库，添加索引会访问ES，使用数据库本地事务是无法控制整个方法的一致性的，比如：向ES写成功了由于网络问题抛出网络超时异常，最终数据库操作回滚了ES操作没有回滚，数据库的数据和ES中的索引不一致。

方案1通常在生产中不会使用。

**方案2：使用Canal+MQ**

Canal是什么？

**canal [kə'næl]**，译意为水道/管道/沟渠，主要用途是基于 MySQL 数据库增量日志解析，对数据进行同步。

Canal可与很多数据源进行对接，将数据由MySQL同步到ES、MQ、DB等各个数据源。

**对Canal+MQ方案不熟悉的同学尽快复习前边课程。**

本方案需要借助Canal和消息队列，具体实现方案如下：

通过上边的技术分析下边对本项目服务搜索方案进行总结。

本项目使用Elasticsearch实现服务的搜索功能，使用Canal+MQ完成服务信息与ES索引同步。

如下图：

<whiteboard token="Y0USwp2r0hvA07bpWv3cqxSinqd"></whiteboard>

流程如下：

运营人员对服务信息进行增删改操作，MySQL记录binlog日志。

Canal定时读取binlog 解析出增加、修改、删除数据的记录。

Canal将修改记录发送到MQ。

同步程序监听MQ，收到增加、修改、删除数据的记录，请求ES创建、修改、删除索引。

C端用户请求服务搜索接口从ES中搜索服务信息。



#### 3）小结

项目为什么要用Elasticsearch？数据很多吗？

项目中如何进行索引同步的？

能说出如何保证MQ消息的可靠性？

能说出如何保证MQ幂等性？或 如何防止重复消费？



### 1.2 配置数据同步环境

目标：

理解Canal+MQ的同步流程

参考文档配置Canal+MQ的同步环境

 

#### 1) Canal+MQ同步流程

下边回顾Canal的工作原理，如下图：

1、Canal模拟 MySQL slave 的交互协议，伪装自己为 MySQL slave ，向 MySQL master 发送dump 协议

MySQL的`dump`协议是MySQL复制协议中的一部分。

2、MySQL master 收到 dump 请求，开始推送 binary log 给 slave (即 canal )

。一旦连接建立成功，Canal会一直等待并监听来自MySQL主服务器的binlog事件流，当有新的数据库变更发生时MySQL master主服务器发送binlog事件流给Canal。

3、Canal会及时接收并解析这些变更事件并解析 binary log 

通过以上流程可知Canal和MySQL master主服务器之间建立了长连接。

![图片展示了Canal + MQ数据同步流程。MySQL主服务器（Master）生成binlog，通过dump方式将binlog发送给Canal，Canal作为slave接收并解析binlog，解析出serve_sync表的数据更新日志，再将数据发送至MQ的数据同步队列。右侧列出了MQ的多种类型，包括kafka、elasticsearch、Apache HBase、RocketMQ、Pulsar等。该图与文档中介绍的Canal + MQ数据同步流程上下文对应，直观呈现了数据同步过程及目标系统。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/IwALbApoeosMEOxxa0ucIznrnig.png)

基于Canal+MQ数据同步流程：

<whiteboard token="ZvmcwacbJhnkClbkNUxcBRQpnac"></whiteboard>

1. 服务管理不仅向serve、serve_item、serve_type表写数据，同时也向serve_sync表写数据，serve_sync用于Canal同步数据使用。
2. 向serve_sync写数据产生binlog
3. Canal请求读取binlog，并解析出serve_sync表的数据更新日志，并发送至MQ的数据同步队列。
4. 异步同步程序监听MQ的数据同步队列，收到消息后解析出serve_sync表的更新日志。
5. 异步同步程序根据serve_sync表的更新日志请求Elasticsearch添加、更新、删除索引文档。

最终实现了将MySQL中的serve_sync表的数据同步至Elasticsearch





**本节实现将MySQL的变更数据通过Canal写入MQ。**

#### 2）配置Canal+MQ数据同步环境

根据Canal+MQ同步流程，下边进行如下配置：

1. 配置Mysql主从同步，开启MySQL主服务器的binlog
2. 安装Canal并配置，保证Canal连接MySQL主服务器成功
3. 安装RabbitMQ，并配置同步队列。
4. 在Canal中配置RabbitMQ的连接信息，保证Canal收到binlog消息写入MQ

对于异步程序监听MQ通过Java程序中实现。

以上四步配置详细参考“配置ES索引同步环境v1.0”。



#### 3）小结

Canal是怎么伪装成 MySQL slave？

Canal数据同步异常了怎么处理？



### 1.3 索引同步

#### 1.3.1 测试索引同步程序

刚才通过配置Canal+MQ的数据同步环境实现了Canal从数据库读取binlog并且将数据写入MQ。

下边编写同步程序监听MQ，收到消息后向ES创建索引。

##### 1) 创建索引结构

启动ES和kibana:

如果没有安装参考“第三方软件安装说明”安装elasticsearch7.17.7 和 kibana7.17.7。

安装完成后进行启动：

```JSON
docker start elasticsearch7.17.7 
docker start kibana7.17.7
```

下边创建索引serve_aggregation，serve_aggregation索引的结构与jzo2o-foundations数据库的serve_sync表结构对应。



首先通过下边的命令查询索引

```JSON
GET /_cat/indices?v 
```

![图片展示的是Elasticsearch中索引查询结果。其中，红色框突出显示了“serve_aggregation”和“serve_provider_info”两个索引，它们均为黄色状态，open且pri为1，rep为1，store.size分别为33.7kb和16.2kb。该图片与上文“创建serve_aggregation索引（已经存在无需重复创建）”的内容相关，用于直观呈现serve_aggregation索引的查询结果，辅助理解其在Elasticsearch中的状态及存储情况。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/EyQfbwtfToCferxf6j8cxvT8ncd.png)

如果需要修改索引结构需要删除重新创建：

```JSON
DELETE 索引名

```

查询索引结构

```JSON
GET /索引名/_mapping
```

创建serve_aggregation索引  **（已经存在无需重复创建）**

```Java
PUT /serve_aggregation
{
   "mappings" : {
      "properties" : {
        "city_code" : {
          "type" : "keyword"
        },
        "detail_img" : {
          "type" : "text",
          "index" : false
        },
        "hot_time_stamp" : {
          "type" : "long"
        },
        "id" : {
          "type" : "keyword"
        },
        "is_hot" : {
          "type" : "short"
        },
        "price" : {
          "type" : "double"
        },
        "serve_item_icon" : {
          "type" : "text",
          "index" : false
        },
        "serve_item_id" : {
          "type" : "keyword"
        },
        "serve_item_img" : {
          "type" : "text",
          "index" : false
        },
        "serve_item_name" : {
          "type" : "text",
          "analyzer": "ik_max_word",
          "search_analyzer":"ik_smart"
          
        },
        "serve_item_sort_num" : {
          "type" : "short"
        },
        "serve_type_icon" : {
          "type" : "text",
          "index" : false
        },
        "serve_type_id" : {
          "type" : "keyword"
        },
        "serve_type_img" : {
          "type" : "text",
          "index" : false
        },
        "serve_type_name" : {
          "type" : "text",
          "analyzer": "ik_max_word",
          "search_analyzer":"ik_smart"
        },
        "serve_type_sort_num" : {
          "type" : "short"
        },
         "unit" : {
          "type" : "long"
        }
      }
    }
}
```



##### 2）阅读同步程序

**1.添加依赖**

首先在foundations工程添加下边的依赖

```Java
<dependency>
    <groupId>com.jzo2o</groupId>
    <artifactId>jzo2o-canal-sync</artifactId>
</dependency>
<dependency>
    <groupId>com.jzo2o</groupId>
    <artifactId>jzo2o-es</artifactId>
</dependency>
```

**2.配置连接 ES**

修改foundations的配置文件：

![这张图片展示的是bootstrap.yml配置文件的部分内容，具体是在nacos命名空间下的共享配置区域。该区域通过data-id标识了多个共享配置项，其中被红色框重点标注的是两项RabbitMQ与Elasticsearch（ES）的配置：对应RabbitMQ配置的data-id为shared-rabbitmq.yaml，对应ES配置的data-id为shared-es.yaml，两项配置的refresh属性均被设置为false。这部分内容与文档中提到的“修改foundations的配置文件”以及“修改nacos中es的配置文件shared-es.yaml”“修改nacos中rabbitmq的配置文件”的操作要求直接对应。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/L42pbpBa3oQ1X0x1f4CcNv7onKf.png)

修改nacos中es的配置文件shared-es.yaml

![图片展示的是nacos中es的配置文件shared - es. yaml部分内容。其中，配置内容以YAML格式呈现，突出显示了“host: 192.168.101.68”和“port: 9200”这两行关键配置项，分别对应es的主机地址和端口号。该图片与文档中“配置连接ES”部分上下文对应，用于说明在nacos中配置es连接时，需设置主机地址和端口号等信息。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/GJ3obqjmNoAzw1xQUb6cXMPfnNx.png)

修改nacos中rabbitmq的配置文件

![图片展示的是nacos中rabbitmq的配置文件shared-rabbitmq.yaml部分内容。其中，rabbitmq项下有host、username、password、port等配置项，对应值分别为192.168.101.68、xbz、xbz、5672。该图片与文档中“3.阅读同步程序”部分相关，用于说明在nacos中rabbitmq配置文件中rabbitmq配置项的设置情况，是配置连接ES步骤中修改rabbitmq配置文件操作的示例。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/Mo4ZbzrKfopY8nxI4ovcGzFrnXb.png)



**3.阅读同步程序**

同步程序继承AbstractCanalRabbitMqMsgListener类，泛型中指定同步表对应的类型。

根据数据同步环境去配置监听MQ：

```Java
package com.jzo2o.foundations.handler;

import com.jzo2o.canal.listeners.AbstractCanalRabbitMqMsgListener;
import com.jzo2o.es.core.ElasticSearchTemplate;
import com.jzo2o.foundations.constants.IndexConstants;
import com.jzo2o.foundations.model.domain.ServeSync;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.util.List;

/**
 * 服务信息同步程序
 *
 * @author itcast
 * @create 2023/8/15 18:14
 **/
@Component
public class ServeCanalDataSyncHandler extends AbstractCanalRabbitMqMsgListener<ServeSync> {

    @Resource
    private ElasticSearchTemplate elasticSearchTemplate;

    //@RabbitListener(queues = "canal-mq-jzo2o-foundations", concurrency = "1")
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "canal-mq-jzo2o-foundations"),
            exchange = @Exchange(name = "exchange.canal-jzo2o", type = ExchangeTypes.TOPIC),
            key = "canal-mq-jzo2o-foundations"),
            concurrency = "1"
    )
    public void onMessage(Message message) throws Exception {
    }
```

concurrency = "1"：表示消费线程数为1。

在同步程序中需要根据业务需求编写同步方法，当服务下架时会删除索引需要重写抽象类中的batchDelete(List<Long> ids)方法，此方法是当删除Serve_sync表的记录时 对索引执行删除操作。

当服务上架后需要添加索引，当服务信息修改时需要修改索引，需要重写抽象类中的batchSave(List<ServeSync> data)方法，此方法是当向Serve_sync表新增或修改记录时对索引执行添加及修改操作。



完整代码如下：

```Java
package com.jzo2o.foundations.handler;

import com.jzo2o.canal.listeners.AbstractCanalRabbitMqMsgListener;
import com.jzo2o.es.core.ElasticSearchTemplate;
import com.jzo2o.foundations.constants.IndexConstants;
import com.jzo2o.foundations.model.domain.ServeSync;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.util.List;

/**
 * 服务信息同步程序
 *
 * @author itcast
 * @create 2023/8/15 18:14
 **/
@Component
public class ServeCanalDataSyncHandler extends AbstractCanalRabbitMqMsgListener<ServeSync> {

    @Resource
    private ElasticSearchTemplate elasticSearchTemplate;

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "canal-mq-jzo2o-foundations"),
            exchange = @Exchange(name = "exchange.canal-jzo2o", type = ExchangeTypes.TOPIC),
            key = "canal-mq-jzo2o-foundations"),
            concurrency = "1"
    )
    public void onMessage(Message message) throws Exception {
        parseMsg(message);
    }

    @Override
    public void batchSave(List<ServeSync> data) {
        Boolean aBoolean = elasticSearchTemplate.opsForDoc().batchInsert(IndexConstants.SERVE, data);
        if(!aBoolean){
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            throw new RuntimeException("同步失败");
        }
    }

    @Override
    public void batchDelete(List<Long> ids) {
        Boolean aBoolean = elasticSearchTemplate.opsForDoc().batchDelete(IndexConstants.SERVE, ids);
        if(!aBoolean){
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            throw new RuntimeException("同步失败");
        }
    }
}
```



##### 3）测试

启动jzo2o-foundations服务。

启动成功,jzo2o-foundations服务作为MQ的消费者和MQ建立通道，进入canal-mq-jzo2o-foundations队列的管理界面，查看是否建立 了监听通道。

![这张图片是RabbitMQ的界面截图，界面顶部显示了RabbitMQ的版本相关信息，当前停留在Queues页面，展示的队列名称为canal-mq-jzo2o-foundations，对应的虚拟主机为/xzb。界面下方的Consumers区域显示了消费者监听通道的相关信息，其中标注了IP为192.168.101.1:63085的消费者相关内容，该队列的消息统计显示Unacked、Total等指标的数值均为0，此界面对应文档中索引同步流程里的监听通道建立环节，直观呈现了监听通道建立后RabbitMQ的队列及消费者相关状态。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/GMtabvspLoVL80x8PAecLNFjn0c.png)

监听通道建立 成功，下边在同步程序打断点：

![图片展示的是ServeCanalDataSyncHandler.java文件代码。其中，关键部分是两个红色箭头指向的代码行，分别为第40行的onMessage方法和第46行的batchSave方法。onMessage方法用于解析消息，batchSave方法用于批量保存数据。该图片与文档中测试索引同步程序的上下文相关，用于说明同步程序中涉及的关键方法。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/Td30bGu7yoY462xR7PZcU3DrnAg.png)



手动修改jzo2o-foundations数据库的serve_sync表的记录，这里修改了服务项的名称

![这张图片展示了一张表格，表格中有id、item_name、serve_type_id等字段。其中id为1686352662791016449的记录，item_name（服务项名称）为“日常维修ab”，该条记录正是文档中所提及的手动修改了服务项名称的serve_sync表对应记录，其id与文档中提到的需查询的1686352662791016449记录一致，符合文档中测试索引同步程序时修改数据库记录的操作内容。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/LfCEb69LIoRSqlxlzJHcnLhvnHe.png)



正常执行同步程序：

![图片展示的是ServeCanalDataSyncHandler类代码，该类继承自AbstractCanalRabbitMqMsgListener<ServeSync>。关键部分是@RabbitListener注解，用于监听canal-mq-jzo2o-foundations队列，设置交换机、键等信息。还有onMessage方法，用于解析消息。batchSave方法用于批量保存数据。图片与上下文关系紧密，是测试索引同步程序中同步程序打断点后，手动修改数据库记录，正常执行同步程序时的代码展示，用于后续保证ES服务正常，同步方法执行成功后进入Kibana查看服务信息。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/LxNnb3E7roqAK4xB5zXcWd4qn8b.png)

放行继续执行到batchSave方法：

![图片展示的是代码调试界面，显示了ServeCanalDataSyncHandler.java文件中的代码。关键部分是batchSave方法，其参数data的值为1。右侧弹出的代码断点调试窗口中，data列表里有一个元素，其serveitemName属性值被修改为“日常维修ab”。该图片与文档中测试索引同步程序的内容相关，用于说明在同步程序打断点后，手动修改jzo2o-foundations数据库的serve_sync表记录，执行同步程序时，同步方法执行成功后进入Kibana查看，查询服务信息与数据库serve_sync表中记录信息一致的操作过程。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/JuVab6zCloaSysxITFdcPTJgnec.png)

保证ES服务正常，同步方法执行成功后进入Kibana查看

执行命令：

```Java

GET /serve_aggregation/_search
{
}
```

![图片展示的是Elasticsearch Dev Tools界面，左侧Console区域输入了GET命令，右侧结果区域显示了服务信息。关键信息有：id为1688352662791016449，serve_type_name为“日常维修ah”，serve_type_img为https://jyv-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/00ba6db...等。该图片与上文测试索引同步程序中，手动修改数据库记录后执行同步程序，同步成功后查看Kibana查询服务信息的内容相关，展示了查询结果。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/ZXdjbEvASodzO7xEN0VccEwlnnd.png)

查询服务信息与数据库serve_sync表中1686352662791016449记录的信息一致。

下边再将服务项名称恢复。

![图片展示的是数据库操作界面，显示了serve_sync表中的一条记录，id为1686352662791016449，serve_item_name为“日常维修”，serve_type_id为1678654490336124929，serve_item_id为1678727478181957634，city_code为010。界面上方有“Tx: Auto”和“DDL”选项，右上角有“提交”按钮。该图片与文档中测试索引同步程序的内容相关，用于展示手动修改jzo2o-foundations数据库的serve_sync表记录后的情况。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/RBQAb5Fb5oLOgCxs1RAc6fawnrg.png)

再进入Kibana查看索引的内容与数据库一致

![这张图片展示了Elastic的Dev Tools界面，左侧为Console区域，显示了用户输入的GET请求命令，具体为“GET /serve_aggregation/_search”。右侧为查询返回的结果，其中id为1686352662791016449的文档中，“serve_item_name”字段的值为“日常维修”，该内容与文档中提及的修改并同步后的serve_sync表对应记录信息一致，用于验证ES索引同步操作的结果。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/Hg23bvZT8oE6qqxtffwc4yM9nwg.png)



##### 3）小结

同步程序的实现步骤：

1. 根据数据库同步表的结构 创建索引结构。
2. 同步程序监听MQ的同步队列
3. 同步程序收到数据同步消息写入Elasticsearch，写的失败抛出异常，消息回到MQ。



如何保证Canal+MQ同步消息的顺序性？

多个jvm进程监听同一个队列保证只有消费者活跃，即只有一个消费者接收消息。

消费队列中的数据使用单线程。



如何保证只有一个消费者接收消息？

队列需要增加x-single-active-consumer参数，表示否启用单一活动消费者模式。使用x-single-active-consumer参数需要修改为如下代码：

在Queue中添加：arguments={@Argument(name="x-single-active-consumer", value = "true", type = "java.lang.Boolean") }

如下所示：

```JSON
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "canal-mq-jzo2o-foundations",arguments={@Argument(name="x-single-active-consumer", value = "true", type = "java.lang.Boolean") }),
            exchange = @Exchange(name="exchange.canal-jzo2o",type = ExchangeTypes.TOPIC),
            key="canal-mq-jzo2o-foundations"),
            concurrency="1"
    )
    public void onMessage(Message message) throws Exception{
        parseMsg(message);
    }
```

concurrency=”1“表示 指定消费线程为1。



#### 1.3.2 服务变更实时同步（实战）

通过测试Canal+MQ同步流程，只有当serve_sync表变化时才会触发同步，serve_sync表什么时候变化 ？

当服务信息变更时需要同时修改serve_sync表，下边先分析serve_sync的变化需求，再进行代码实现。

##### 1）管理同步表需求

现在如何去维护serve_sync这张表呢？

根据serve_sync表的结构分析：

添加：区域服务上架向serve_sync表添加记录，同步程序新增索引记录。

删除：区域服务下架从serve_sync表删除记录，同步程序删除索引记录。

修改：

修改服务项修改serve_sync的记录。

修改服务分类修改serve_sync的记录。

修改服务价格修改serve_sync的记录。

设置热门/取消热门修改serve_sync的记录。



##### 2）代码实现

根据需求编写代码并测试。

测试内容如下：

测试服务上架添加索引。

测试服务下架删除索引。

测试修改服务价格修改索引。

测试修改服务项名称修改索引。

测试修改修改服务分类名称修改索引。



### 1.4 搜索接口（实战）

目标：开发搜索接口。

 

#### 1）接口如下

参数内容：区域编码，服务类型id、关键字

区域编码：用户定位成功前端记录区域编码（city_code），搜索时根据city_code搜索该区域的服务。

服务类型id：在全部服务界面选择一个服务类型查询其它下的服务列表。

关键字：输入关键字搜索服务项名称、服务类型名称。



接口名称：服务搜索接口

接口路径：**GET**/foundations/customer/serve/search

![图片展示了服务搜索接口的请求参数信息。请求数据类型为application/x - www - form - urlencoded，响应数据类型为“/”。参数包括cityCode（城市编码，query类型，必填，数据类型为string）、keyword（关键词，query类型，非必填，数据类型为string）、serveTypeId（服务类型id，query类型，非必填，数据类型为integer(int64)）。该图片与文档中介绍服务搜索接口的上下文相关，是对接口请求参数的具体说明。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/KBE4buOSwo2hzJx7K7WctpPmnrb.png)

![图片展示了服务搜索接口的响应参数。参数包括msg（响应消息，类型string）、code（状态码，类型integer，范围200 - 成功，其它 - 失败）、data（响应数据，类型array，schema为服务简略响应信息）、serveltemSortNum（服务项排序字段，类型integer）、serveltemName（服务项名称，类型string）、serveltemId（服务项id，类型integer）、serveltemIcon（服务项图标，类型string）、id（服务id，类型integer）。该图片与上文介绍的服务搜索接口响应参数内容相呼应，直观呈现了各参数的名称、说明、类型及schema。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/Cxqvb4Jl5oSpFGxMi7ccXG1DnMc.png)

controller方法：

```Java
@RestController("consumerServeController")
@RequestMapping("/customer/serve")
@Api(tags = "用户端 - 首页服务查询接口")
public class FirstPageServeController {
...
    @GetMapping("/search")
    @ApiOperation("首页服务搜索")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "cityCode", value = "城市编码", required = true, dataTypeClass = String.class),
            @ApiImplicitParam(name = "serveTypeId", value = "服务类型id", dataTypeClass = Long.class),
            @ApiImplicitParam(name = "keyword", value = "关键词", dataTypeClass = String.class)
    })
    public List<ServeSimpleResDTO> findServeList(@RequestParam("cityCode") String cityCode,
                                                 @RequestParam(value = "serveTypeId", required = false) Long serveTypeId,
                                                 @RequestParam(value = "keyword", required = false) String keyword) {

        return null;
    }
```

#### 2）搜索方法

查询条件包括：

城市代码

关键字：模糊匹配服务项名称、服务分类名称

服务分类：点击服务分类查询分类下的服务。

排序字段：服务项排序字段



## 5 电商项目商品搜索（实战）

### 5.1 需求分析

#### 首页输入关键字

首页搜索栏输入关键字进行搜索：

![图片展示了青橙电商首页界面。上方有“青橙电商”标识，搜索栏内有“精品零食抢先购”字样，右侧有“输入关键字”提示。下方有多个功能图标，如秒杀活动、每日拼团、领券中心等。最下方是“300新人专享”满1200元可用的活动区域。图片与文档中“首页输入关键字”内容相关，直观呈现了首页搜索栏的位置及界面样式。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/Nn1obmAoToKTlhx51PMcOHDOnhb.png)

![图片展示的是电商项目商品搜索页面。页面顶部有“商品”标题，搜索栏内显示“荣耀”。下方有综合排序、销量、价格等筛选选项，以及筛选按钮。商品列表中展示了多款荣耀80手机，包括红色128G和256G、黑色128G和256G等，每款商品下方显示价格、已售数量及“进店”按钮。该图片与文档中“首页输入关键字”搜索需求分析部分对应，直观呈现了搜索结果页面样式。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/A2oEbIlx5o5wzGxhuracHtzon3c.png)

#### 通过分类进行搜索

![图片展示的是青橙电商首页界面。上方有“精品零食抢先购”搜索栏。中间有“约会女王 第二件半价”活动宣传图。下方有多个功能图标，如秒杀活动、每日拼团、领券中心等。最下方是“300元新人专享”满1200元可用的活动区域。图片中红色箭头指向“分类”图标，该图标位于底部导航栏，与其他图标如首页、购物车、我的等相邻。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/U2LUbupMuo6Z2dx03becqm32nVg.png)

![图片展示的是电商项目商品搜索中通过分类进行搜索的界面。左侧有多个分类标签，如数码办公、家用电器等。右侧是手机通讯分类下的商品展示，有手机、老年机、对讲机等商品图片及名称。图片中红色箭头指向老年机商品，并标注“点击分类进行搜索”，直观提示用户可通过点击分类标签进行商品搜索。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/G07nb5TEFomQEhxNdVRcmQswnGg.png)





#### 条件筛选

![这张图呈现的是电商项目的商品搜索页面，页面顶部设有搜索栏，下方显示荣耀80系列的多款在售商品，包含红色、黑色不同配色，128G、256G不同存储版本的款式，售价均为1.00元，标注有自营、家家乐相关标识及已售数量，页面右上方的“筛选”按钮被红色箭头重点标注，对应文档中电商项目搜索功能的条件筛选相关需求，该筛选功能是商品搜索时进行条件筛选操作的核心入口。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/SFpabGXXmob0JBxPqTgcUeagnbc.png)



![这是电商项目搜索模块的条件筛选界面，用于对商品进行精准筛选。界面包含多个筛选维度：品牌筛选区，全部分类下设有数码办公、手机通讯、手机等选项；还有价格区间筛选栏，可设置最低价与最高价；网络筛选分为4G网络（默认选项为4G FDD-LTE）和5G网络（当前状态为支持5G）；充电接口默认选项为Type-C；内存提供128G、256G两个选项可供选择，底部设有“重置”和“确定”按钮，用于清空筛选条件或确认当前筛选条件并执行搜索，该界面用于配合商品搜索，满足用户按多维度筛选商品的需求。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/MPupbKyuAozMtOxSnotcmHG5nxZ.png)





### 5.2 索引结构

根据需求梳理索引结构如下：

```TypeScript
PUT /mall_goods
{
  "mappings" : {
    "properties" : {
      "id" : {
        "type" : "keyword"
      },
      "goods_id" : {
        "type" : "keyword"
      },
      "goods_name" : {
        "type" : "text",
        "analyzer": "ik_max_word",
        "search_analyzer":"ik_smart"
      },
      "goods_type" : {
        "type" : "keyword"
      },
      "goods_video" : {
        "type" : "text",
        "index" : false
      },
      "grade" : {
        "type" : "float"
      },
      "buy_count" : {
        "type" : "long"
      },
      "high_praise_num" : {
        "type" : "long"
      },
      "intro" : {
        "type" : "text",
        "index" : false
      },
      "mobile_intro" : {
        "type" : "text",
        "index" : false
      },
      "market_enable" : {
        "type" : "keyword"
      },
      "point" : {
        "type" : "long"
      },
      "price" : {
        "type" : "float"
      },
      "recommend" : {
        "type" : "boolean"
      },
      "release_time" : {
        "type" : "date",
        "format" : "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      },
      "sales_model" : {
        "type" : "keyword"
      },
      "self_operated" : {
        "type" : "boolean"
      },
      "seller_id" : {
        "type" : "keyword"
      },
      "seller_name" : {
        "type" : "text",
        "analyzer": "ik_max_word",
        "search_analyzer":"ik_smart"
      },
      "selling_point" : {
        "type" : "text",
        "index" : false
      },
      "sku_source" : {
        "type" : "long"
      },
      "small" : {
        "type" : "text",
        "index" : false
      },
      "sn" : {
        "type" : "keyword"
      },
      "store_category_name_path" : {
        "type" : "text",
        "analyzer": "ik_max_word",
        "search_analyzer":"ik_smart",
        "fields" : {
          "keyword" : {
            "type" : "keyword",
            "ignore_above" : 256
          }
        },
        "fielddata" : true
      },
      "store_category_path" : {
        "type" : "keyword"
      },
      "category_path" : {
        "type" : "keyword"
      },
      "category_name_path" : {
        "type" : "text",
        "analyzer": "ik_max_word",
        "search_analyzer":"ik_smart",
         "fields" : {
          "keyword" : {
            "type" : "keyword",
            "ignore_above" : 256
          }
        },
        "fielddata" : true
      },
      "thumbnail" : {
        "type" : "text",
        "index" : false
      },
      "brand_url" : {
        "type" : "keyword"
      },
      "brand_id" : {
        "type" : "keyword"
      },
      "brand_name" : {
        "type" : "text",
        "analyzer": "ik_max_word",
        "search_analyzer":"ik_smart",
        "fields" : {
          "keyword" : {
            "type" : "keyword",
            "ignore_above" : 256
          }
        },
        "fielddata" : true
      },
      "auth_flag" : {
        "type" : "keyword"
      },
      "attr_list" : {
        "type" : "nested",
        "properties" : {
          "name" : {
            "type" : "keyword"
          },
          "sort" : {
            "type" : "long"
          },
          "type" : {
            "type" : "integer"
          },
          "value" : {
            "type" : "keyword"
          }
        }
      },
      "store_id" : {
        "type" : "keyword"
      },
      "store_name" : {
        "type" : "text",
        "analyzer": "ik_max_word",
        "search_analyzer":"ik_smart"
      }
    }
  }
}
```



### 5.3 索引同步

电商项目索引同步是使用MQ实现，如下图：

<whiteboard token="EDUEwymcHhgwphb0Il3ckEqFngO"></whiteboard>

自行阅读相关代码，并进行测试。

测试方法：

发布一个商品，审核通过将商品信息发送到MQ，Java程序监听MQ，得到商品信息后写入ES索引。



### 5.4 搜索接口定义

阅读以下接口定义：

搜索接口：

http://localhost:8084/buyer/doc.html#/default/%E4%B9%B0%E5%AE%B6%E7%AB%AF,%E5%95%86%E5%93%81%E6%8E%A5%E5%8F%A3/getGoodsByPageFromEsUsingGET

聚合接口（从ES中获取相关商品品牌名称，分类名称及属性）：

http://localhost:8084/buyer/doc.html#/default/%E4%B9%B0%E5%AE%B6%E7%AB%AF,%E5%95%86%E5%93%81%E6%8E%A5%E5%8F%A3/getGoodsRelatedByPageFromEsUsingGET

controller方法如下：

![这张图片展示了电商项目中买家端的商品搜索相关的Controller方法代码，具体是GoodsBuyController类的内容。左侧项目结构显示该类位于mall-consumer-web模块的com.juzo.mall.buyer.controller路径下；右侧代码区域用红框标注了两个核心接口方法，第一个是标注@ApiOperation注解、用于从ES中获取商品的getGoodsByPageFromEs方法，第二个是用于从ES获取商品相关品牌、分类及属性的getGoodsRelatedByPageFromEs方法，代码中明确了两个方法的入参与返回值，和文档里的搜索接口、聚合接口定义完全对应。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-wu-zhang-sou-suo-mo-kuai/TJ0XbcGCqotvKXxsd7ycwj2znRb.png)



### 5.5 实战

根据接口定义进行开发，实现service方法。

完成开发后对照需求分析进行测试。

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 云岚到家项目 / 第五章 搜索模块 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/Nt2RwaUXXibe1NkXxONcS8M5nRg)
