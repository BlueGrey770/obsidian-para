---
type: memo
date: 2026-11-06
aliases:
  - "微服务面试篇（下）"
---
# 微服务面试篇（下）

## 学习目标

1. 能够说出xxl-job任务调度的优势
2. 能说出xxl-job的组成结构
3. 能够编写热点商品更新缓存任务
4. 能够说出Redid的持久化机制
5. 能够说出Redis主从同步原理
6. 能够说出Redis哨兵模式的工作原理
7. 能够说出Redis分片集群的工作原理
8. 能够说出Redis五种基本数据类型
9. 能够说出Redis ZSET类型的底层结构
10. 能够说出Redis过期策略
11. 能够说出Redis淘汰策略



## 1 任务调度方案

### 1.1 什么是任务调度

#### 1.1.1 概念

我们可以先思考一下下面业务场景的解决方案： 

某电商系统需要在每天上午10点，下午3点，晚上8点发放一批优惠券。 

某银行系统需要在信用卡到期还款日的前三天进行短信提醒。 

某财务系统需要在每天凌晨0:10结算前一天的财务数据，统计汇总。 

12306会根据车次的不同，而设置某几个时间点进行分批放票。 

某网站为了实现天气实时展示，每隔5分钟就去天气服务器获取最新的实时天气信息。 

以上场景就是任务调度所需要解决的问题。 

**任务调度是指系统为了自动完成特定任务，在约定的特定时刻去执行任务的过程。有了任务调度即可解放更多的人 力由系统自动去执行任务。** 



在解决缓存击穿方案中，通过缓存定时预热避免缓存击穿。



#### 1.1.2 技术方案

如何实现任务调度？

**1、使用jdk提供的Timer定时器**

示例代码如下：

每个Timer对应一个线程，可以同时启动多个Timer定时执行多个任务。

```Java
public static void main(String[] args){  
    Timer timer = new Timer();  
    timer.schedule(new TimerTask(){
        @Override  
        public void run() {  
           //TODO：something
        }  
    }, 1000, 2000);  //1秒后开始调度，每2秒执行一次
}
```

Time使用简单，可以实现每隔一定的时间去执行任务，但无法实现每天凌晨去执行任务，即在某个时间点去执行任务。



**2、使用第三方Quartz方式实现**

        Quartz 是一个功能强大的任务调度框架（项目地址：https://github.com/quartz-scheduler/quartz   ），它可以满足更多更复杂的调度需求，Quartz 设计的核心类包括 Scheduler, Job 以及 Trigger。其中，Job 负责定义需要执行的任务，Trigger 负责设置调度策略，Scheduler 将二者组装在一起，并触发任务开始执行。Quartz支持简单的按时间间隔调度、还支持按日历调度方式，通过设置CronTrigger表达式（包括：秒、分、时、日、月、周、年）进行任务调度。

虽然Quartz可以实现按日历调度的方式，但无法支持分布式环境下任务调度。分布式环境下通常一个服务部署多个实例即多个jvm进程，假设一个项目的微服务部署两个实例每个实例定时执行更新缓存的任务，两个实例就会重复执行。如下图：

<readonly-block type="diagram"></readonly-block>

**3、SpringTask**

SpringTask是SpringBoot内置的定时任务处理技术，通过简单注解即可快速实现定时任务，但多节点部署会导致任务重复执行的情况发生，且任务不好监控管理



**4、使用分布式调度平台XXL-JOB**



XXL-JOB是一个轻量级分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。

官网：https://www.xuxueli.com/xxl-job/

文档：https://www.xuxueli.com/xxl-job/#%E3%80%8A%E5%88%86%E5%B8%83%E5%BC%8F%E4%BB%BB%E5%8A%A1%E8%B0%83%E5%BA%A6%E5%B9%B3%E5%8F%B0XXL-JOB%E3%80%8B

XXL-JOB主要有调度中心、执行器、任务：

![yrrpbq5kgosngexgpubc9a1qn4a](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/yrrpbq5kgosngexgpubc9a1qn4a.png)
> 图注：这张图展示了XXL-JOB分布式任务调度平台的核心架构，调度中心通过名为AppName的标识，同时连接管理着两个执行器。执行器1和执行器2中都部署了多项任务，包括任务1、任务2等。该架构体现了XXL-JOB调度中心与执行器的分层结构，调度中心负责调度管控，执行器负责具体的任务执行，这种架构正是该框架用于解决多JVM进程重复执行任务问题的直观体现。

**调度中心：**

        负责管理调度信息，按照调度配置发出调度请求，自身不承担业务代码；

        主要职责为执行器管理、任务管理、监控运维、日志管理等

**任务执行器：**

        负责接收调度请求并执行任务逻辑；

        主要职责是执行任务、执行结果上报、日志服务等



使用XXL-JOB可以解决多个jvm进程重复执行任务的问题，如下图：

<readonly-block type="diagram"></readonly-block>

XXL-JOB调度中心可以配置路由策略，比如：第一个、轮询策略、分片等，它们分别表示的意义如下：

第一个：即每次执行任务都由第一个执行器去执行。

轮询：即执行器轮番执行。

分片：每次执行任务广播给每个执行器让他们同时执行任务。

如果根据需求每次执行任务仅由一个执行器去执行任务可以设置路由策略：第一个、轮询。

如果根据需求每次执行任务由多个执行器同时执行可以设置路由策略为：分片。



xxl-job分布式任务调度系统具体有以下优势：

**1、并行任务调度**

        并行任务调度实现靠多线程，如果有大量任务需要调度，此时光靠多线程就会有瓶颈了，因为一台计算机CPU的处理能力是有限的。

        如果将任务调度程序分布式部署，每个结点还可以部署为集群，这样就可以让多台计算机共同去完成任务调度，我们可以将任务分割为若干个分片，由不同的实例并行执行，来提高任务调度的处理效率。

**2、高可用**

        若某一个实例宕机，不影响其他实例来执行任务。

**3、弹性扩容**

        当集群中增加实例就可以提高并执行任务的处理效率。

**4、任务管理与监测**

        对系统中存在的所有定时任务进行统一的管理及监测。让开发人员及运维人员能够时刻了解任务执行情况，从而做出快速的应急处理响应。

**5、避免任务重复执行**

        当任务调度以集群方式部署，同一个任务调度可能会执行多次，比如在上面提到的电商系统中到点发优惠券的例子，就会发放多次优惠券，对公司造成很多损失，所以我们需要控制相同的任务在多个运行实例上只执行一次。



#### 1.1.3 小结

xxl-job任务调度与第三方Quartz或timer定时器实现任务调度有什么优势？





### 1.2 搭建XXL-JOB

#### 1.2.1 组成结构

XXL-JOB由两部分组成：

- **调度模块（调度中心）**：  
负责管理调度信息，按照调度配置发出调度请求，自身不承担业务代码。调度系统与任务解耦，提高了系统可用性和稳定性，同时调度系统性能不再受限于任务模块；  
支持可视化、简单且动态的管理调度信息，包括任务新建，更新，删除，GLUE开发和任务报警等，所有上述操作都会实时生效，同时支持监控调度结果以及执行日志，支持执行器Failover。
- **执行模块（执行器）**：  
负责接收调度请求并执行任务逻辑。任务模块专注于任务的执行等操作，开发和维护更加简单和高效；  
接收“调度中心”的执行请求、终止请求和日志请求等。

![j4m7bazpmoh1yhx4hdrcufwonsc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/j4m7bazpmoh1yhx4hdrcufwonsc.png)
> 图注：图片为XXL-JOB架构图，展示了调度中心和执行器的组成结构。调度中心包括任务管理、执行器管理、日志管理、数据中心等模块，如任务管理有执行器、任务模式等，日志管理有调度日志、Rolling日志等。执行器部分有执行器服务、日志服务、回调服务等，还涉及调度请求、执行日志、调度结果等。该图直观呈现了XXL-JOB v2.4.0架构，与上下文介绍的执行模块（执行器）职责相呼应。



#### 1.2.2 部署调度中心

**1.查阅xxl-job的源码**

首先下载XXL-JOB

GitHub：[https://github.com/xuxueli/xxl-job](https://github.com/xuxueli/xxl-job)

码云：[https://gitee.com/xuxueli0323/xxl-job](https://gitee.com/xuxueli0323/xxl-job)

项目使用2.3.1版本： https://github.com/xuxueli/xxl-job/releases/tag/2.3.1

也可从课程资料目录获取，解压xxl-job-2.3.1.zip

使用IDEA打开解压后的目录

![smfjbpjp4ooljhxcqhjc88zenof](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/smfjbpjp4ooljhxcqhjc88zenof.png)
> 图注：这张图片展示了在IDEA开发工具中打开的xxl-job-2.3.1项目目录结构，其中核心内容与上下文提及的XXL-JOB部署调度中心相关。目录内明确标注了关键模块，包括作为调度中心的xxl-job-admin、公共依赖xxl-job-core，以及执行器示例模块xxl-job-executor-samples，该示例下还细分了Springboot版本的执行器示例，这正是上下文推荐的使用形式。目录还包含了.idea、doc、pom.xml等项目常规文件，doc目录对应上下文提到的包含数据库脚本的文档资料，与部署调度中心的操作步骤直接呼应。

xxl-job-admin：调度中心

xxl-job-core：公共依赖

xxl-job-executor-samples：执行器Sample示例（选择合适的版本执行器，可直接使用）

    ：xxl-job-executor-sample-springboot：Springboot版本，通过Springboot管理执行器，推荐这种方式；

    ：xxl-job-executor-sample-frameless：无框架版本；

doc :文档资料，包含数据库脚本

在下发的虚拟机的MySQL中已经创建了xxl_job_2.3.1数据库

如下图：

![f9aabmgkhokqffx5wlgcec8kngd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/f9aabmgkhokqffx5wlgcec8kngd.png)
> 图注：这张图片展示了编号为2.3.1版本的xxl_job数据库中的表结构，该数据库是搭建XXL-JOB调度中心时需用到的目标数据库。图中清晰列出了该数据库包含的核心表，包括分组信息表xxl_job_group、任务信息表xxl_job_info、锁表xxl_job_lock、日志相关的xxl_job_log、xxl_job_log_report、xxl_job_logglue，执行器注册表xxl_job_registry以及用户表xxl_job_user，这些表为XXL-JOB的任务调度功能提供了数据存储支持。

##### 安装xxl-job

**没有使用下发虚拟机的同学请自行安装xxl-job。**

拉取镜像：

docker pull xuxueli/xxl-job-admin:2.3.1

创建数据库：xxl_job_2.3.1

导入xxl_job_2.3.1.sql,如下：

![c1xqble36os1sexe3hpczjkwnys](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/c1xqble36os1sexe3hpczjkwnys.png)
> 图注：图片展示了数据库中xxl_job_2.3.1的tables表结构。其中包含xxl_job_group、xxl_job_info、xxl_job_lock、xxl_job_log、xxl_job_log_report、xxl_job_logglue、xxl_job_registry、xxl_job_user等8个表。该图片与文档中“创建数据库：xxl_job_2.3.1”及“导入xxl_job_2.3.1.sql”上下文相关，直观呈现了导入后的数据库表结构，帮助理解数据库中各表的作用及关系。

创建目录：

/data/soft/xxl-job

/data/soft/xxl-job/applogs

创建配置文件：/data/soft/xxl-job/application.properties,内容如下：

```Java
### web
server.port=8080
server.servlet.context-path=/xxl-job-admin

### actuator
management.server.servlet.context-path=/actuator
management.health.mail.enabled=false

### resources
spring.mvc.servlet.load-on-startup=0
spring.mvc.static-path-pattern=/static/**
spring.resources.static-locations=classpath:/static/

### freemarker
spring.freemarker.templateLoaderPath=classpath:/templates/
spring.freemarker.suffix=.ftl
spring.freemarker.charset=UTF-8
spring.freemarker.request-context-attribute=request
spring.freemarker.settings.number_format=0.##########

### mybatis
mybatis.mapper-locations=classpath:/mybatis-mapper/*Mapper.xml
#mybatis.type-aliases-package=com.xxl.job.admin.core.model

### xxl-job, datasource
spring.datasource.url=jdbc:mysql://192.168.101.68:3306/xxl_job_2.3.1?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=mysql
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

### datasource-pool
spring.datasource.type=com.zaxxer.hikari.HikariDataSource
spring.datasource.hikari.minimum-idle=10
spring.datasource.hikari.maximum-pool-size=30
spring.datasource.hikari.auto-commit=true
spring.datasource.hikari.idle-timeout=30000
spring.datasource.hikari.pool-name=HikariCP
spring.datasource.hikari.max-lifetime=900000
spring.datasource.hikari.connection-timeout=10000
spring.datasource.hikari.connection-test-query=SELECT 1
spring.datasource.hikari.validation-timeout=1000

### xxl-job, email
spring.mail.host=smtp.qq.com
spring.mail.port=25
spring.mail.username=xxx@qq.com
spring.mail.from=xxx@qq.com
spring.mail.password=xxx
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
spring.mail.properties.mail.smtp.socketFactory.class=javax.net.ssl.SSLSocketFactory

### xxl-job, access token
xxl.job.accessToken=default_token

### xxl-job, i18n (default is zh_CN, and you can choose "zh_CN", "zh_TC" and "en")
xxl.job.i18n=zh_CN

## xxl-job, triggerpool max size
xxl.job.triggerpool.fast.max=200
xxl.job.triggerpool.slow.max=100

### xxl-job, log retention days
xxl.job.logretentiondays=30

```





创建容器：

```Java
docker run -d -e \
  --restart=always \
  -v /data/soft/xxl-job/applogs:/data/applogs \
  -v /data/soft/xxl-job/application.properties:/application.properties \
  -p 8088:8080 \
  --name xxl-job-admin \
  xuxueli/xxl-job-admin:2.3.1
```

启动成功进入管理界面：

http://192.168.101.68:8088/xxl-job-admin/toLogin

账号/密码：admin/123456



**2.启动xxl-job**

执行docker start xxl-job-admin 启动xxl-job

访问：http://192.168.101.68:8088/xxl-job-admin/toLogin

账号和密码：admin/123456

> 注意：将任务管理里，所有状态为running的任务都点击停止。
> 
> 然后到xxl-job数据库中删除日志表数据：

#### 1.2.3 执行器

**1.添加执行器依赖**

下边配置执行器，执行器负责与调度中心通信接收调度中心发起的任务调度请求，执行器负责执行微服务中定义的任务，执行器程序由xxl-job提供，在微服务中引入下边的依赖即加入了执行器的程序。

我们在商品服务中引入xxl-job执行器依赖。

```Java
<dependency>
    <groupId>com.xuxueli</groupId>
    <artifactId>xxl-job-core</artifactId>
    <version>2.3.0</version>
</dependency>
```

参考源代码中的XxlJobConfig去编写xxl-job的配置类，此配置类已提供，将课程资料中xxl-job下的配置类和模型类拷贝到商品服务的config包下：

![g2jkbsiokoqay8xsz4uc3gbenfc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/g2jkbsiokoqay8xsz4uc3gbenfc.png)
> 图注：这张图片展示了商品服务项目的结构与对应的配置类代码，左侧项目面板中，item-service项目的main目录下的com.hmall.item.config包内，包含了需要导入的xxl-job相关配置类：XxlJobConfiguration与XxlJobProperties，另有RedisConfig类在同目录中；右侧编辑区域打开了XxlJobConfiguration.java文件，该文件正是文档中提及的参考编写的xxl-job配置类，代码中使用@EnableConfigurationProperties注解引入配置属性，通过@Bean注解定义XxlJobSpringExecutor实例，实现调度中心地址、执行器端口等核心参数的设置逻辑，对应文档中搭建XXL-JOB时拷贝配置类到商品服务的相关说明。



**2.配置xxl-job**

在application.yaml下配置xxl-job

```Java
xxl-job:
  enable: true
  port: 11603
  access-token: default_token
  admin:
    address: http://192.168.101.68:8088/xxl-job-admin
  executor:
    appName: ${spring.application.name}
    #ip: 172.17.0.170
    port: ${xxl-job.port}
    # 执行器日志文件保存天数 [选填] ： 过期日志自动清理, 限制值大于等于3时生效; 否则, 如-1, 关闭自动清理功能
    log-retention-days: 30
```

说明：

address：调度中心的地址

appName：执行器名称，spring.application.name表示微服务的名称（在bootstrap.yml中配置）

port：执行器端口号,通过xxl-job.port配置，执行器通过此端口与调度中心通信。





**3. 下边进入调度中心添加执行器**

启动商品服务即启动了xxl-job执行器。

进入调度中心，进入执行器管理界面，如下图：

![unfsb3dvooauigxw6sxcrdplnab](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/unfsb3dvooauigxw6sxcrdplnab.png)
> 图注：图片展示的是任务调度中心的执行器管理界面。左侧导航栏中“执行器管理”被红色箭头突出显示。右侧显示执行器列表，包含AppName、名称、注册方式、OnLine机器地址等信息，当前显示1条记录，每页显示10条。界面右上角有“新增”按钮，用于添加执行器，其也被红色箭头指向。该图片与文档中“进入调度中心，进入执行器管理界面”及后续配置xxl-job、填写执行器信息等内容相关，直观呈现了操作入口。

点击新增，填写执行器信息

![x6c1b5zzcogyp9xklwncwxgsnmd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/x6c1b5zzcogyp9xklwncwxgsnmd.png)
> 图注：这张图片展示的是XXL-JOB调度中心新增执行器的表单页面，页面标题为“新增执行器”。表单中，AppName填写的是“item-service”，名称填写的是“商品服务”，注册方式已选中“自动注册”选项，机器地址栏显示提示内容“请输入执行器地址列表，多地址逗号分隔”，表单底部设有“保存”和“取消”两个操作按钮。该页面对应文档中调度中心添加执行器的步骤，是在完成商品服务与XXL-JOB的配置后，在调度端创建执行器的操作界面。

**AppName**：执行名称， appName: \${spring.application.name}表示指定执行器名称就是微服务的应用名。

**名称**：取一个中文名称。

**注册方式**：自动注册，只要执行器和调度中心连通执行器会自动注册到调度中心

**机器地址**：自动注册时不用填写。



添加成功：

![wzfdbtqiuoyhp3x2w0eciaainrc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wzfdbtqiuoyhp3x2w0eciaainrc.png)
> 图注：这是XXL-JOB执行器注册成功后的相关页面，展示了注册后的执行器信息，其中对应项的名称为“jzo2o-foundations”，名称右侧标注其所属类型为“运营基础管理”，该执行器的注册状态为自动注册，额外备注信息为“无”，页面右侧设有“操作”按钮，可进行相关操作，该页面对应文档中进入XXL-JOB调度中心执行器管理界面、查看执行器地址的步骤，是执行器注册成功后的呈现内容。

启动item-service，查看item-service的控制台：

\>>>>>>>>>>> xxl-job remoting server start success, nettype = class com.xxl.job.core.server.EmbedServer, port = 11603 说明执行器启动成功。

稍等片刻进入 xxl-job调度中心，进入执行器管理界面，执行器注册成功：

![ifcubpvs9olosjxnumccms1dn4b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ifcubpvs9olosjxnumccms1dn4b.png)
> 图注：图片展示的是XXL-JOB调度中心执行器管理界面。界面上方有AppName、名称等搜索栏及搜索、新增按钮。下方列表显示AppName、名称、注册方式、OnLine机器地址等信息，其中“item-service”执行器名称为“商品服务”，注册方式为自动注册，OnLine机器地址处有“查看（1）”按钮，该按钮被红色框突出显示。此图与上文介绍执行器注册成功后，点击“查看（1）”查看执行器地址的内容相呼应。

点击“查看（1）”，查看执行器的地址，如下图：

![bu91bwq87ojpfcxnpf6cxqovnrp](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/bu91bwq87ojpfcxnpf6cxqovnrp.png)
> 图注：这是XXL-JOB调度中心执行器管理界面的注册节点弹窗，弹窗内清晰显示了注册的执行器地址为“http://192.168.101.1:11603”，该地址正是执行器启动后在控制台中提示的成功启动端口对应的地址。弹窗带有编号“1.”标识，还有一个蓝色的“确定”按钮，对应上下文里点击“查看（1）”后所呈现的执行器地址信息，用于确认该注册节点的地址配置。

#### 1.2.4 小结

项目为什么要用xxl-job？



能说出xxl-job的组成部分。



### 1.3 XXL-JOB任务入门

#### 1.3.1 编写测试任务

定时执行任务就需要编写任务方法，此任务方法由执行器去调用。

可以参考xxl-job源码去编写任务方法，从源码目录中找到执行器示例代码：

xxl-job-2.3.1\xxl-job-executor-samples\xxl-job-executor-sample-springboot\src\main\java\com\xxl\job\executor\service\jobhandler\SampleXxlJob.java

![c0r0bmavqouljgxcoiuc4q0gnvc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/c0r0bmavqouljgxcoiuc4q0gnvc.png)
> 图注：图片展示了xxl-job-executor-samples中的xxl-job-executor-sample-springboot项目的目录结构。在src/main/java目录下，有com.xxl.job.executor、core.config、mvc.controller、service.jobhandler等子目录，其中service.jobhandler目录下有一个SampleXxlJob.java文件被蓝色框突出显示。该图片与上文提到的“可以参考xxl-job源码去编写任务方法，从源码目录中找到执行器示例代码”相呼应，直观呈现了示例代码所在位置。

部分示例代码如下：下边代码中demoJobHandler()就是一个任务方法，需要使用@XxlJob注解标识，所在类需要由spring去管理，所以加了@Component注解。

将源代码中的SampleXxlJob类拷贝到商品服务的job包下，修改代码如下：

```Java
package com.hmall.item.job;

import com.xxl.job.core.context.XxlJobHelper;
import com.xxl.job.core.handler.annotation.XxlJob;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

@Component
@Slf4j
public class SampleXxlJob {

    /**
     * 1、简单任务示例（Bean模式）
     */
    @XxlJob("demoJobHandler")
    public void demoJobHandler() throws Exception {
        log.info("XXL-JOB, Hello World.");
    }


    /**
     * 2、分片广播任务
     */
    @XxlJob("shardingJobHandler")
    public void shardingJobHandler() throws Exception {

        // 分片参数
        int shardIndex = XxlJobHelper.getShardIndex();
        int shardTotal = XxlJobHelper.getShardTotal();

        log.info("分片参数：当前分片序号 = {}, 总分片数 = {}", shardIndex, shardTotal);
    }

}
```



#### 1.3.2 配置任务

下边在调度中心配置任务。

进入任务管理，新增任务：

![rodybjobiooql8xfhijcgio8nch](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/rodybjobiooql8xfhijcgio8nch.png)
> 图注：图片展示的是XXL-JOB任务管理界面。界面上方有“任务管理”标题，下方有“执行器”下拉框，当前选中“商品服务”，还有“全部”下拉框、任务描述、JobHandler、负责人输入框及搜索、新增按钮。下方有每页显示条数选择框，以及任务ID、任务描述、调度类型、运行模式、负责人、状态、操作等列。界面底部提示“表中数据为空”。图片与上下文关系为，上下文提到进入任务管理新增任务，此图展示了任务管理界面，为新增任务操作提供参考。

填写任务信息：

![g7fcbrj0holfw5xtxzqcbibznum](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/g7fcbrj0holfw5xtxzqcbibznum.png)
> 图注：图片展示的是XXL-JOB任务管理中新增任务的界面。基础配置部分，执行器为“商品服务”，负责人是“xxl-job”，任务描述为“示例任务”，报警邮件需填写。调度配置中，调度类型为CRON，Cron表达式为“0/3 * * * * ?”。任务配置里，运行模式为BEAN，JobHandler为“demoJobHandler”，任务参数为空。高级配置中，路由策略为“第一个”，调度过期策略为“忽略”，任务超时时间为“任务超时时间为0时，大于零时生效”，阻塞处理策略为“单机串行”，失败重试次数为“失败重试次数，大于零时生效”。

说明：

**调度类型**：

![qynrbbbz2ouqbexkhvwc0l1dn4c](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qynrbbbz2ouqbexkhvwc0l1dn4c.png)
> 图注：这是XXL-JOB配置任务时的调度类型选择界面，用于任务调度类型的设置。界面上标注有“调度类型”的选项，对应的下拉选择框中，当前高亮选中的选项为“CRON”，下拉列表中还包含“无”和“固定速度”两个可选选项，其中“固定速度”选项位于“CRON”选项的下方。该界面用于在任务配置中选择合适的调度类型，对应文档中关于XXL-JOB任务配置里调度类型选择的相关内容，明确了“调度类型”的可选值，帮助用户选择符合需求的定时调度策略。

**固定速度**指按固定的间隔定时调度。

**Cron**，通过Cron表达式实现更丰富的定时调度策略。

Cron表达式是一个字符串，通过它可以定义调度策略，格式如下：

{秒数} {分钟} {小时} {日期} {月份} {星期} {年份(可为空)}

**Cron** 的各个域的定义如下表格所示：

![r4nebqspookbgvxzg82cdmeknub](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/r4nebqspookbgvxzg82cdmeknub.png)
> 图注：这是一张Cron表达式各域定义的表格，表格分为四列，分别为“域”“是否必填”“值以及范围”“通配符”，清晰展示Cron表达式的组成规则。其中秒、分、时、日、月、周六个域均为必填，年为选填；各域对应有明确的数值或取值范围，例如秒、分的范围为0-59，时为0-23，日为1-31，月为1-12或JAN-DEC，周为1-7或SUN-SAT，年为1970-2099；通配符列则标注了可使用的符号，如逗号、短横线、星号、斜杠，日和周的通配符还包含问号、L、W、井号等特殊符号，该表格用于辅助理解Cron表达式的格式规范，为XXL-JOB任务配置提供基础规则参考。



xxl-job提供图形界面去配置：

![vsjvbjek0ohgnlxum3fcxpajnpe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vsjvbjek0ohgnlxum3fcxpajnpe.png)
> 图注：这张图是XXL-JOB配置任务时的Cron表达式秒数域配置界面，顶部标注了Cron的时间域选项，包含秒、分钟、小时、日、月、周、年。该界面针对秒域提供了多种配置方式，包括通配符规则说明、周期范围设置、指定间隔执行设置，还提供了00到59的秒数选项。当前选中“指定”模式，且勾选了“00”秒数，用于配置任务每整点的第0秒执行。

一些例子如下：

0 0 0 \* \* ?  每天0点触发

30 10 1 \* \* ?  每天1点10分30秒触发

0/30 \* \* \* \* ? 每30秒触发一次

\* 0/10 \* \* \* ? 每10分钟触发一次

为了方便测试这里第5秒执行一次，设置为：0/5 \* \* \* \* ?

 **cron** 表达式的难点在于通配符，下边的内容请自行阅读

> - `,` 这里指的是在两个以上的时间点中都执行，如果我们在 “分” 这个域中定义为 `8,12,35` ，则表示分别在第8分，第12分 第35分执行该定时任务。
> - `-` 这个比较好理解就是指定在某个域的连续范围，如果我们在 “时” 这个域中定义 `1-6`，则表示在1到6点之间每小时都触发一次，用 `,` 表示 `1,2,3,4,5,6`
> - `*` 表示所有值，可解读为 “每”。 如果在“日”这个域中设置 `*`,表示每一天都会触发。
> - `?` 表示不指定值。使用的场景为不需要关心当前设置这个字段的值。例如:要在每月的8号触发一个操作，但不关心是周几，我们可以这么设置 `0 0 0 8 * ?`
> - `/` 在某个域上周期性触发，该符号将其所在域中的表达式分为两个部分，其中第一部分是起始值，除了秒以外都会降低一个单位，比如 在 “秒” 上定义 `5/10` 表示从 第 5 秒开始 每 10 秒执行一次，而在 “分” 上则表示从 第 5 秒开始 每 10 分钟执行一次。
> - `L` 表示英文中的**LAST** 的意思，只能在 “日”和“周”中使用。在“日”中设置，表示当月的最后一天(依据当前月份，如果是二月还会依据是否是润年), 在“周”上表示周六，相当于”7”或”SAT”。如果在”L”前加上数字，则表示该数据的最后一个。例如在“周”上设置”7L”这样的格式,则表示“本月最后一个周六”
> - `W` 表示离指定日期的最近那个工作日(周一至周五)触发，只能在 “日” 中使用且只能用在具体的数字之后。若在“日”上置”15W”，表示离每月15号最近的那个工作日触发。假如15号正好是周六，则找最近的周五(14号)触发, 如果15号是周未，则找最近的下周一(16号)触发.如果15号正好在工作日(周一至周五)，则就在该天触发。如果是 “1W” 就只能往本月的下一个最近的工作日推不能跨月往上一个月推。
> - `#` 表示每月的第几个周几，只能作用于 “周” 上。例如 ”2#3” 表示在每月的第三个周二。



**运行模式**有BEAN和GLUE，bean模式较常用就是在项目工程中编写执行器的任务代码，GLUE是将任务代码编写在调度中心。

**JobHandler**即任务方法名，填写任务方法上边@XxlJob注解中的名称。

**路由策略**：

第一个：即每次执行任务都由第一个执行器去执行。

轮询：即执行器轮番执行。

分片：每次执行任务广播给每个执行器让他们同时执行任务。

详细说明xxl-job源码中的doc目录下的文档：

![x7wrbxlizoqzngxrylwc6rg1n9c](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/x7wrbxlizoqzngxrylwc6rg1n9c.png)
> 图注：这张图片展示了XXL-JOB相关的文档列表，其中被红色框标注的文件是“XXL-JOB官方文档.md”，该文件对应文档中提及的“xxl-job源码中的doc目录下的文档”，是微服务面试中介绍XXL-JOB相关配置内容时所指向的官方说明文档文件，用于补充XXL-JOB的相关说明信息。

#### 1.3.3 启动任务并测试

任务配置完成，下边启动任务

![zgbsby14ko2e8wxamzycahclnyb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/zgbsby14ko2e8wxamzycahclnyb.png)
> 图注：这是XXL-JOB任务管理界面的截图，展示的是启动任务的操作步骤相关内容。界面里有一条任务记录，其任务ID为25，任务描述为“示例任务”，调度类型是CRON，值为0/3 * * * * ? *，运行模式为BEAN，对应JobHandler为demoJobHandler，负责人为xxl-job。该任务状态为STOP，操作列的下拉菜单被展开，菜单中有执行一次、查看日志等选项，红色箭头明确指向菜单中的“启动”选项，这和文档中启动任务的操作要求相呼应，直观呈现了XXL-JOB中启动任务的具体操作界面内容。

启动成功：

![tki4bmlgbouh32xvmtrc8kmqnjc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/tki4bmlgbouh32xvmtrc8kmqnjc.png)
> 图注：图片展示的是XXL-JOB任务管理界面。界面上方有执行器、商品服务等筛选框，下方显示任务ID、任务描述、调度类型、运行模式、负责人、状态、操作等信息，其中任务ID为25，任务描述为“示例任务”，调度类型为CRON，运行模式为BEAN，负责人是xxl - job，状态为RUNNING。界面底部有“启动成功”按钮。该图片与上文“启动任务并测试”内容相关，直观呈现了任务配置完成后的任务管理界面情况。



我们在任务方法上打断点跟踪，任务方法被执行，如下图：

![ycfxbshy3o2yzqxal62caqs8n6b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ycfxbshy3o2yzqxal62caqs8n6b.png)
> 图注：图片展示的是SampleXxlJob.java代码，用于示例XXL-JOB任务。代码中定义了名为demoJobHandler的任务方法，方法内使用XxlJobHelper.log记录日志，显示“XXL-JOB, Hello World.”。接着通过for循环，每2秒打印一次“beat at:”及循环次数，共执行5次。该图片与上文“启动任务并测试”内容相关，展示了任务方法的代码实现，帮助理解任务执行过程。



### 1.4 分片广播任务

#### 1.4.1 什么是分片广播任务

掌握了xxl-job的基本使用，下边思考如何进行分布式任务处理呢？如下图，我们会启动多个执行器组成一个集群，去执行任务。

![tfmbbvrr8ommcvx7oricbxtuncb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/tfmbbvrr8ommcvx7oricbxtuncb.png)
> 图注：图片展示了分片广播任务的架构。调度中心位于上方，通过三条箭头分别指向执行器1、执行器2和执行器3。执行器1、执行器2和执行器3被蓝色椭圆框包围，代表它们属于同一分片。



查看xxl-job官方文档，阅读高级配置相关的内容：

```SQL
高级配置：
    - 路由策略：当执行器集群部署时，提供丰富的路由策略，包括；
        FIRST（第一个）：固定选择第一个机器；
        LAST（最后一个）：固定选择最后一个机器；
        ROUND（轮询）：；
        RANDOM（随机）：随机选择在线的机器；
        CONSISTENT_HASH（一致性HASH）：每个任务按照Hash算法固定选择某一台机器，且所有任务均匀散列在不同机器上。
        LEAST_FREQUENTLY_USED（最不经常使用）：使用频率最低的机器优先被选举；
        LEAST_RECENTLY_USED（最近最久未使用）：最久未使用的机器优先被选举；
        FAILOVER（故障转移）：按照顺序依次进行心跳检测，第一个心跳检测成功的机器选定为目标执行器并发起调度；
        BUSYOVER（忙碌转移）：按照顺序依次进行空闲检测，第一个空闲检测成功的机器选定为目标执行器并发起调度；
        SHARDING_BROADCAST(分片广播)：广播触发对应集群中所有机器执行一次任务，同时系统自动传递分片参数；可根据分片参数开发分片任务；

```



下边要重点说的是分片广播策略，分片是指是调度中心以执行器为维度进行分片，将集群中的执行器标上序号：0，1，2，3...，广播是指每次调度会向集群中的所有执行器发送任务调度，请求中携带分片参数。

分片广播任务就是调度中心按照调度策略广播通信所有执行器（分片）去执行任务。

如下图：

![ptp7bqqx0o3dygxo2xocdi6nntb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ptp7bqqx0o3dygxo2xocdi6nntb.png)
> 图注：该图对应微服务面试内容里的分片广播任务说明，展示了xxl-job分片广播任务的执行逻辑：上方为调度中心，下方有三个分片序号分别为0、1、2的执行器，三者被蓝色椭圆圈住构成执行器集群。调度中心向下发送带有分片参数的任务调度请求，左侧执行器（序号0）收到的分片参数为0、3，中间执行器（序号1）收到的分片参数为1、3，右侧执行器（序号2）收到的分片参数为2、3，直观体现了分片广播任务中，调度中心向所有集群执行器同时发送携带分片参数的调度请求的逻辑。



每个执行器收到调度请求同时接收分片参数。

xxl-job支持动态扩容执行器集群从而动态增加分片数量，当有任务量增加可以部署更多的执行器到集群中，调度中心会动态修改分片的数量。

**作业分片适用哪些场景呢？**

- 分片任务场景：10个执行器的集群来处理10w条数据，每台机器只需要处理1w条数据，耗时降低10倍；

所以，广播分片方式不仅可以充分发挥每个执行器的能力，并且根据分片参数可以控制任务是否执行，最终灵活控制了执行器集群分布式处理任务。



**使用说明：**

"分片广播" 和普通任务开发流程一致，不同之处在于可以获取分片参数进行分片业务处理。

Java语言任务获取分片参数方式：

BEAN、GLUE模式(Java)，可参考Sample示例执行器中的示例任务"ShardingJobHandler"：

```Java
/**
 * 2、分片广播任务
 */
@XxlJob("shardingJobHandler")
public void shardingJobHandler() throws Exception {

    // 分片参数
    int shardIndex = XxlJobHelper.getShardIndex();
    int shardTotal = XxlJobHelper.getShardTotal();

    XxlJobHelper.log("分片参数：当前分片序号 = {}, 总分片数 = {}", shardIndex, shardTotal);

    // 业务逻辑
    for (int i = 0; i < shardTotal; i++) {
        if (i == shardIndex) {
            XxlJobHelper.log("第 {} 片, 命中分片开始处理", i);
        } else {
            XxlJobHelper.log("第 {} 片, 忽略", i);
        }
    }

}
```

#### 1.4.2 测试分片广播任务

下边测试作业分片广播任务：

1、定义作业分片的任务方法

```Java
/**
 * 2、分片广播任务
 */
@XxlJob("shardingJobHandler")
public void shardingJobHandler() throws Exception {

    // 分片参数
    int shardIndex = XxlJobHelper.getShardIndex();
    int shardTotal = XxlJobHelper.getShardTotal();

    XxlJobHelper.log("分片参数：当前分片序号 = {}, 总分片数 = {}", shardIndex, shardTotal);

    // 业务逻辑
    for (int i = 0; i < shardTotal; i++) {
        if (i == shardIndex) {
            XxlJobHelper.log("第 {} 片, 命中分片开始处理", i);
        } else {
            XxlJobHelper.log("第 {} 片, 忽略", i);
        }
    }

}






/**
 * 2、分片广播任务-示例2
 */
@XxlJob("shardingJobHandler")
public void shardingJobHandler() throws Exception {

    // 分片参数
    int shardIndex = XxlJobHelper.getShardIndex(); //当前分片的索引
    int shardTotal = XxlJobHelper.getShardTotal();//总分片数量

    //处理所有的商品数据
    int page = shardIndex + 1;
    int size = 1000;
    while (true) {
        //处理商品数据一共88000多条数据
        Page<Item> itemPage = itemService.lambdaQuery()
                .page(new Page<>(page, size));
        List<Item> records = itemPage.getRecords();
        if (CollUtils.isEmpty(records)) {
            break;
        }
        log.info("分片处理商品数据，当前分片序号 = {}, 总分片数 = {}, 当前页 = {}, 总页数 = {}, 总记录数 = {}",
                shardIndex, shardTotal, page, itemPage.getPages(), itemPage.getTotal());
        page += shardTotal;
    }
}
```

2、在调度中心添加任务

![jzswbjxzeorjnhx78fecilhinsg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/jzswbjxzeorjnhx78fecilhinsg.png)
> 图注：图片展示的是在调度中心添加任务时的新增界面。关键信息有：执行器选择“视频处理任务”，任务描述为“测试分片任务”，调度类型为CRON，Cron为“0/3 * * * * ?”，运行模式为BEAN，JobHandler为“sharding.JobHandler”，任务参数为空，路由策略为分片广播，调度过期策略为忽略，阻塞处理策略为单机串行，失败重试次数大于零时生效。该图片与上下文“在调度中心添加任务”对应，直观呈现了添加任务时的配置界面及关键参数设置。



添加成功：

![dutebqsoko4jzixzwsycc9a0noe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dutebqsoko4jzixzwsycc9a0noe.png)
> 图注：这张图片是任务调度中心的任务管理页面，页面左侧有运行报表、任务管理、调度日志、执行器管理等功能选项，顶部显示登录用户为admin。当前页面的任务列表中，存在一条ID为27、任务描述为分片任务的记录，其调度类型为CRON表达式“0/3 * * * * ? *”，运行模式为BEAN类型，对应JobHandler为shardingJobHandler，负责人为xxljob，状态显示为STOP，操作列有操作按钮；页面上方的执行器选择了“商品服务”，还设有任务描述、JobHandler、负责人的搜索输入框，以及搜索、新增按钮，底部可调整每页显示的记录数。

下边启动两个商品服务实例

两个实例的在启动时注意端口不能冲突：

实例1 在VM options处添加：-Dserver.port=8081 -Dxxl-job.port=11603

实例2 在VM options处添加：-Dserver.port=7081 -Dxxl-job.port=11604

启动成功观察执行器

![qjmmbxebrolb4vx0ybacnvkjn8f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qjmmbxebrolb4vx0ybacnvkjn8f.png)
> 图注：这张图片展示的是微服务面试中分片广播任务测试环节的执行器管理界面，界面内的“item-service”执行器名称被红色框标注，其对应的Online机器地址栏有红色箭头指向“查看(2)”按钮，点击该按钮弹出注册节点窗口，窗口内列出了两个自动注册的节点地址，分别为http://192.168.101.1:11603和http://192.168.101.1:11604，对应测试步骤中启动的两个商品服务实例，用于确认分片广播任务的执行器实例注册情况，和文档中启动两个服务实例的操作内容相呼应，用于观察任务在各实例的执行状态。

启动任务，观察日志

实例1:

![nuaxb2gw7oiurcxztssciiqpn5c](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/nuaxb2gw7oiurcxztssciiqpn5c.png)
> 图注：这是测试分片广播任务时实例1的执行日志内容，核心展示了分片任务的处理结果。日志对应com.hmall.item.job.SampleXxlJob任务，明确标注第0片命中分片并开始处理，第1片被忽略；同时给出了任务的分片参数，当前分片序号为0，总分片数为2。该内容记录了该分片广播任务在指定实例中的实际执行状态，是完成该测试流程中观察实例执行情况的关键内容，用于验证分片广播任务的分配与执行逻辑。

实例2：

![jkjobqwoqopip5xyggyckcuwnmd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/jkjobqwoqopip5xyggyckcuwnmd.png)
> 图注：这是测试分片广播任务过程中的执行日志截图，内容展示了实例执行分片任务的情况，对应上下文中启动两个执行器实例后观察执行情况的环节。截图显示两个相同的job任务类com.hmall.item.job.SampleXxlJob的执行日志，第一个日志条目标注“第 0 片，忽略”，第二个日志条目标注“第 1 片，命中分片开始处理”，并附带分片参数：当前分片序号为1，总分片数为2，清晰体现了分片广播任务的分片命中与参数情况。

下边启动两个执行器实例，观察每个实例的执行情况。

#### 1.4.3. 分片执行任务

当一次分片广播到来，各执行器如何根据分片参数去分布式执行任务，保证执行器之间执行的任务不重复呢？

举例：

批量处理商品表中的数据，保证每个执行器处理的商品信息不重复。



可以将分片总数和分片序列带入sql，如下：

select \* from item where id % #{shardingTotalCount} = #{shardingItem}

假设当前有两个分片，分片0执行如下sql:

select *\** from item where id % 2=0

分片1执行如下sql:

select *\** from item where id % 2=1

两个分片获取的数据是不一样的。



测试如下：

定义mapper如下：

```Java
public interface ItemMapper extends BaseMapper<Item> {
   ....

    //根据分片总数和分片序号查询商品表 select * from item where id % ? = ?
    @Select("select id from item where id % #{shardingTotalCount} = #{shardingIndex}")
    List<Item> selectBySharding(int shardingTotalCount, int shardingIndex);
}
```

在任务中调用ItemMapper获取商品信息

```Java
@Autowired
private ItemMapper itemMapper;

/**
 * 2、分片广播任务
 */
@XxlJob("shardingJobHandler")
public void shardingJobHandler() throws Exception {

    // 分片参数
    int shardIndex = XxlJobHelper.getShardIndex();
    int shardTotal = XxlJobHelper.getShardTotal();

    //查询商品表
    //select * from item where id % ? = ?
    List<Item> items = itemMapper.selectBySharding(shardTotal, shardIndex);
    items.forEach(x-> System.out.println(x.getId()));

    log.info("分片参数：当前分片序号 = {}, 总分片数 = {}", shardIndex, shardTotal);
}
```



### 1.5 热点商品定时预热任务

#### 1.5.1 编写任务方法

根据需求，为了防止缓存击穿我们使用xxl-job定时对热点商品进行预热。

首先在ItemServiceImpl中编写获取热点商品id方法

```Java
@Override
public List<Long> queryHotItems() {
    //模拟热点商品id
    return List.of(317578L, 317580L);
}
```

编写商品预热任务方法

```Java
@Component
@Slf4j
public class SampleXxlJob {
    private static Logger logger = LoggerFactory.getLogger(SampleXxlJob.class);

    //注入itemService
    @Resource
    private IItemService itemService;
    
/**
 * 对热点商品进行定时更新缓存
 */
@XxlJob("hotItemCacheJobHandler")
public void hotItemCacheJobHandler() throws Exception {
    log.info("定时对热点商品进行更新缓存开始...");
    //获取热点商品id
    List<Long> ids = itemService.queryHotItems();
    itemService.queryItemByIdsCache(ids);
    log.info("定时对热点商品进行更新缓存完成...");
}
...
```



#### 1.5.2 配置任务

下边在调度中心配置任务。

进入任务管理，新增任务：

![zyoobkrnkofm9cxcpu1ctep6ned](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/zyoobkrnkofm9cxcpu1ctep6ned.png)
> 图注：这张图片是调度中心新增任务的配置界面，对应文档里“进入任务管理，新增任务”的操作步骤，用于配置热点商品定时预热任务。界面的基础配置部分，执行器选择“商品服务”，任务描述为“热点商品定时预热”；调度配置里，调度类型为CRON，CRON表达式设置为“0/5 * * * * ? *”；任务配置部分，运行模式为BEAN，JobHandler指定为“hottemCacheJobHandler”，任务参数暂未填写；高级配置里，路由策略为“第一个”，调度过期策略为“忽略”，子任务ID、报警邮件、任务超时时长等内容按提示未输入，完成配置后可点击“保存”或“取消”按钮。

####  1.5.3 启动任务并测试

任务配置完成，下边启动任务

![z3uqbkjxgowhlfxlcnkchifxnth](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/z3uqbkjxgowhlfxlcnkchifxnth.png)
> 图注：图片展示的是任务管理界面。界面中显示了两条任务记录，任务ID分别为26和25，任务描述分别为“热点商品定时预 addCriterion图片

下边重启商品服务，观察热点商品是否在redis更新缓存。



#### 1.5.4 小结

项目中哪里用了xxl-job？怎么用的？



## 2 Redis 持久化

### 2.1 面试题

面试题：说一下Redis的持久化机制。

该面试题考察你对Redis持久化机制的理解程度。

Redis使用两种主要的持久化机制来保证数据的安全性：RDB（Redis Database Backup）和AOF（Append Only File），需要去理解这两种持久化机制的工作原理及应用场景才能回答本问题。



### 2.2 RDB

#### 2.2.1 RDB是什么？

RDB全称Redis Database Backup file（Redis数据备份文件），也被叫做Redis数据快照。简单来说就是把内存中的所有数据都记录到磁盘中。当Redis实例故障重启后，从磁盘读取快照文件，恢复数据。快照文件称为RDB文件。

#### 2.2.2 执行方式

RDB 的触发方式包括：手动触发、自动触发。

1. 手动触发：

   - 使用`SAVE`命令，这会导致Redis在执行完该命令之前阻塞客户端请求。
   - 使用`BGSAVE`命令，Redis会在后台异步进行快照操作，不会阻塞客户端请求。
2. 自动触发：

   - 通过配置`save`选项，可以设置在满足特定条件时自动触发RDB持久化。例如，可以在一定时间内发生了一定数量的写操作后自动进行一次快照。
   - Redis停机时，Redis停机时会执行一次save命令，实现RDB持久化。



在redis.conf文件中配置了rdb的文件名称和保存路径：

```JavaScript
是否压缩 ,建议不开启，压缩也会消耗cpu，磁盘的话不值钱
rdbcompression yes
RDB文件名称
dbfilename dump.rdb  
文件保存的路径目录
dir ./ 
```



**1）save命令**

执行save命令可以立即执行一次RDB,在执行完该命令之前阻塞客户端请求.

![qaevbq3jqotiw5xwtstcj0urnib](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qaevbq3jqotiw5xwtstcj0urnib.png)
> 图注：图片展示的是在Redis容器中执行save命令的界面。命令“save”执行后，Redis返回“OK”提示，表明操作成功。该图片与文档中介绍Redis持久化RDB部分的save命令内容相关，直观呈现了执行save命令的界面效果，帮助读者理解save命令的执行结果及Redis的响应情况。

进入redis容器：docker exec -it redis /bin/bash

在执行命令前我查看rdb文件

![vbmbb0an2orllex3etwc6oo9nqg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vbmbb0an2orllex3etwc6oo9nqg.png)
> 图注：这张图片展示的是进入Redis容器后执行的命令结果，先执行了`ls`命令查看容器内目录文件，可见`dump.rdb`文件，接着执行`ls -l`命令查看该文件的详细信息，其中`dump.rdb`文件的权限、所属用户、所属组、大小、修改时间等信息清晰显示，该内容对应文档中执行save命令前查看rdb文件的操作，直观呈现了此时Redis容器内dump.rdb文件的状态，为后续执行save命令后对比文件更新提供了参照。

执行save命令后从文件更新时间上看发现rdb文件有更新

![pielbnrwbojubqxoxg4cicknned](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/pielbnrwbojubqxoxg4cicknned.png)
> 图注：图片展示了在redis容器中执行ls -l命令后的目录列表。其中，红色框突出显示了名为“dump.rdb”的文件，其权限为-rw-r--r--，属主为root，属组也为root，文件大小为3572字节，创建时间为2022年8月23日06:53。该图片与上文介绍在redis.conf文件中配置rdb文件名称和保存路径的内容相关，通过查看rdb文件更新时间，可发现执行save命令后rdb文件有更新。



**2）bgsave命令**

下面的命令可以异步执行RDB：

![jpklbzy6to46ddxsevwcvhk4nzc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/jpklbzy6to46ddxsevwcvhk4nzc.png)
> 图注：该图片展示了执行Redis异步生成RDB文件的bgsave命令后的输出结果，具体内容为：IP地址为192.168.101.68.0的Redis节点成功执行了bgsave命令，返回提示信息“Background saving started”，表明后台已开始异步生成RDB文件。这一输出对应文档中介绍的2.2 RDB部分里的bgsave命令相关内容，该命令的作用是开启独立进程完成RDB生成，不会阻塞主进程处理客户端请求，图片中的提示信息直观体现了该命令的执行状态。

这个命令执行后会开启独立进程完成RDB，主进程可以持续处理用户请求，不受影响。

  
3**）触发RDB条件**

Redis内部有触发RDB的机制，可以在redis.conf文件中找到，格式如下：

```JavaScript
900秒内，如果至少有1个key被修改，则执行bgsave ， 如果是save "" 则表示禁用RDB
save 900 1  
save 300 10  
save 60 10000 
```



#### 2.2.3 执行原理

copy on write

RDB 的工作流程如下图所示：

![ebsybu2xcoscg5xzq8lcabmmnff](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ebsybu2xcoscg5xzq8lcabmmnff.png)
> 图注：图片展示了Redis RDB持久化的工作流程。主进程和子进程共享内存页，当任一进程对内存页进行修改时，系统分配新内存页并复制。子进程读取内存数据生成RDB文件，写入磁盘后替换旧RDB文件。主进程在子进程生成RDB文件时进行读操作，子进程在生成RDB文件时进行写操作。该图与上下文紧密相关，直观呈现了RDB持久化中主进程与子进程的内存操作及RDB文件生成、替换过程。

1. 启动子进程：当Redis服务器接收到`BGSAVE`命令时，它会先检查是否有正在运行的子进程。如果没有，则创建一个新的子进程（fork操作）。
2. 复制父进程内存映像：此时父进程和子进程共享相同的内存页。当任一进程对某个内存页进行修改时，系统才会为这个进程分配新的内存页并复制内存过去。
3. 子进程生成RDB文件：子进程开始读取内存中的数据并生成RDB文件。在此过程中，父进程继续处理客户端请求。
4. 替换旧的RDB文件：一旦子进程完成RDB文件的生成，它会用新生成的文件替换旧的RDB文件。
5. 父进程继续工作：子进程结束后，父进程继续正常工作，此时新的RDB文件已经包含了最新的数据状态。



#### 2.2.4 RDB 的优缺点

- 优点：

  - 性能影响小：因为RDB文件的生成是在子进程中进行的，不会阻塞主进程。
  - 启动速度快：在Redis重启时，可以通过加载RDB文件快速恢复数据。
  - 文件紧凑：RDB文件是一个紧凑的二进制文件，占用的空间较小。
- 缺点：

  - 数据丢失风险：如果Redis服务器在最后一次成功生成RDB文件后宕机，那么从上次快照以来的所有更改都将丢失。
  - 快照生成频率：需要平衡性能影响与数据丢失的风险。



#### 2.2.5 总结

RDB方式bgsave的基本流程？

- fork主进程得到一个子进程，共享内存空间
- 子进程读取内存数据并写入新的RDB文件
- 用新RDB文件替换旧的RDB文件

RDB会在什么时候执行？save 60 1000代表什么含义？

- 默认是服务停止时
- 代表60秒内至少执行1000次修改则触发RDB

RDB的缺点？

- RDB执行间隔时间长，两次RDB之间写入数据有丢失的风险
- fork子进程、压缩、写入RDB文件都比较耗时



### 2.3 AOF

#### 2.3.1.AOF原理

AOF全称为Append Only File（追加文件）是Redis提供的另一种持久化机制，与RDB相比，AOF持久化通过记录每次写操作的命令到一个单独的日志文件中，使得即使Redis服务重启，也可以通过重放这些命令来恢复数据。

![vsbwba3btocnpbxweffcrzt7njc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vsbwba3btocnpbxweffcrzt7njc.png)
> 图注：图片展示了Redis AOF持久化原理。左侧是命令行界面，输入“redis-cli 127.0.0.1:6379> set num 123”后，返回“OK”。右侧是一个红色的Redis图标，图标右侧有黑色箭头指向右侧的AOF文件，文件内容为“$3 set $3 num $3 123”。该图与上下文紧密相关，直观呈现了AOF记录每次写操作命令到单独日志文件，即使Redis服务重启也能通过重放这些命令恢复数据的原理。



#### 2.3.2.AOF配置

AOF默认是关闭的，需要修改redis.conf配置文件来开启AOF：

```JavaScript
#是否开启AOF功能，默认是no
appendonly yes
#AOF文件的名称
appendfilename "appendonly.aof"

```

AOF的命令记录的频率也可以通过redis.conf文件来配：

```JavaScript
#表示每执行一次写命令，立即记录到AOF文件
appendfsync always 
#写命令执行完先放入AOF缓冲区，然后表示每隔1秒将缓冲区数据写到AOF文件，是默认方案
appendfsync everysec 
#写命令执行完先放入AOF缓冲区，由操作系统决定何时将缓冲区内容写回磁盘
appendfsync no
```

三种策略对比：

![ng8rbarsbonmhmxkpfncaebvnuf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ng8rbarsbonmhmxkpfncaebvnuf.png)
> 图注：图片是一张表格，对比了Redis AOF配置中三种策略的配置项、刷盘时机、优点及缺点。Always为同步刷盘，优点是可靠性高、几乎不丢数据，但性能影响大；everysec为每秒刷盘，性能适中，最多丢失1秒数据；no为操作系统控制，性能最好，但可靠性较差，可能丢失大量数据。该表与上下文介绍的AOF配置内容相关，直观呈现了三种策略的优缺点。

打开aof重启redis，写入数据查看aof文件的内容，aof内容是若干redis命令的集合。

![odgjbgpywoyigvxvzvlcaqacngd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/odgjbgpywoyigvxvzvlcaqacngd.png)
> 图注：这张图片展示了在Linux命令行环境下执行`ls -l`指令后的输出结果，显示了文件`appendonly.aof`的详细信息。该文件权限为`-rw-r--r--`，所属用户和用户组均为root，文件大小为52字节，修改时间为10月15日08:07。这一内容对应上下文里“打开aof重启redis，写入数据查看aof文件的内容”的操作，直观呈现了生成的AOF文件的具体属性，符合AOF持久化相关配置后的实际文件状态。

![rmrmb97njohdctxpgnucdx5jneh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/rmrmb97njohdctxpgnucdx5jneh.png)
> 图注：图片展示了在Linux系统下使用`cat`命令查看`appendonly.aof`文件的内容。文件内容以`*2`开头，接着是`$6`，后为`SELECT`命令，再是`$1`和`0`，然后是`*3`，之后是`$3`，以及`set`命令，最后是`$2`、`$2`和`aa`。该图片与文档中介绍AOF配置及打开AOF后写入数据查看AOF文件内容的上下文相关，直观呈现了AOF文件的命令记录形式。



#### 2.3.3.AOF文件重写

因为是记录命令，AOF文件会比RDB文件大的多。而且AOF会记录对同一个key的多次写操作，但只有最后一次写操作才有意义。通过执行bgrewriteaof命令，可以让AOF文件执行重写功能，用最少的命令达到相同效果。

![pglqbtzrqo837fxpbxscjnmvnsc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/pglqbtzrqo837fxpbxscjnmvnsc.png)
> 图注：图片展示了Redis AOF重写前后的命令记录情况。左侧为AOF原本的命令记录，包含`set num 123`、`set name jack`、`set num 666`三条命令。中间箭头指向“bgrewirteaof”操作。右侧为重写后的命令记录，仅记录了`mset name jack num 666`。该图片与上下文紧密相关，直观呈现了通过bgrewriteaof命令重写AOF文件后，命令记录简化的情况，帮助理解AOF重写原理。

如图，AOF原本有三个命令，但是`set num 123 和 set num 666`都是对num的操作，第二次会覆盖第一次的值，因此第一个命令记录下来没有意义。

所以重写命令后，AOF文件内容就是：`mset name jack num 666`



Redis也会在触发阈值时自动去重写AOF文件。阈值也可以在redis.conf中配置：

```JavaScript
# AOF文件比上次文件 增长超过多少百分比则触发重写
auto-aof-rewrite-percentage 100
# AOF文件体积最小多大以上才触发重写 
auto-aof-rewrite-min-size 64mb 
```



### 2.4 总结

RDB与AOF对比，RDB和AOF各有自己的优缺点，如下图：

![nfqibjjsborfvtxhwcuc0wxtnph](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/nfqibjjsborfvtxhwcuc0wxtnph.png)
> 图注：这张图是Redis两种持久化方式RDB与AOF的对比表格，清晰呈现了二者在多维度的差异。在持久化方式上，RDB是定时对整个内存做快照，AOF则记录每一次执行的命令。二者在数据完整性、文件大小、宕机恢复速度等方面区别明显：RDB的数据完整性不完整、文件体积小、恢复速度快；AOF数据完整性相对完整但取决于刷盘策略、文件体积大、恢复速度慢。系统资源占用与使用场景上，RDB高耗CPU和内存，适用于可容忍少量数据丢失、追求更快启动速度的场景；AOF低占CPU内存，重写时会占用大量IO与内存，适用于对数据安全性要求较高的场景。



在实际应用中，RDB（Redis Database dump）和AOF（Append Only File）这两种持久化方式各有优势，通常的选择取决于具体的应用场景和需求。

**如果对数据完整性有极高要求，那么AOF通常是首选；**

**如果对性能要求较高并且可以接受少量的数据丢失，那么RDB是更好的选择。**

**而在大多数情况下，结合使用RDB和AOF能够提供最佳的数据保护和性能表现。**



RDB与AOF详细总结如下：

1. **RDB (Redis Database dump)**

- 原理

  - 是把内存中的所有数据都记录到磁盘中。当Redis实例故障重启后，从磁盘读取快照文件，恢复数据。快照文件称为RDB文件。
  - 执行方式：手动触发，自动触发
  - 异步方式同步，当Redis服务器接收到`BGSAVE`命令时，它会先检查是否有正在运行的子进程。如果没有，则创建一个新的子进程（fork操作）。子进程开始读取内存中的数据并生成RDB文件。在此过程中，父进程继续处理客户端请求。一旦子进程完成RDB文件的生成，它会用新生成的文件替换旧的RDB文件。
- 主要用途：

  - 数据备份：由于RDB文件是某个时间点的快照，因此非常适合用于数据备份和灾难恢复。
  - 快速恢复：RDB文件相对较小，可以在服务重启时快速加载到内存中，适合需要快速恢复服务的情况。
- 适用场景：

  - 当数据的完整性不是绝对重要，可以接受一定程度的数据丢失时。
  - 对于读多写少的应用，RDB可以提供较好的性能。
  - 需要定期做全量备份时。
- 配置示例：

  - 使用`save`命令手动触发快照生成。
  - 设置自动快照生成规则，例如：`save 900 1` 表示在900秒内如果有1个key发生变化，则生成快照。

1. **AOF (Append Only File)**

- 原理

  - AOF持久化通过记录每次写操作的命令到一个单独的日志文件中，使得即使Redis服务重启，也可以通过重放这些命令来恢复数据。
- 主要用途：

  - 数据持久化：AOF通过记录每个写命令来保证数据的完整性，非常适合需要强一致性的场景。
  - 数据恢复：即使发生故障，也可以通过重放AOF文件中的命令来恢复数据。
- 适用场景：

  - 当数据不能有任何丢失时。
  - 对于写密集型应用，AOF可以提供更好的数据保护。
  - 需要频繁的写操作，并且要求数据尽可能不丢失的情况下。
- 配置示例：

  - 设置`appendfsync`选项来控制文件同步的频率，例如：`appendfsync everysec` 表示每秒同步一次。
  - 定期进行AOF重写以减少文件大小。

1. **组合使用 RDB 和 AOF**

在很多情况下会选择同时使用RDB和AOF两种持久化方式，以结合两者的优点：

- 使用RDB进行定期的全量备份，这样即使发生灾难性的故障也能快速恢复。
- 使用AOF来确保数据的完整性和连续性，在服务重启时可以通过重放AOF文件来恢复最新的数据状态。

这种组合方式能够提供较好的性能和数据保护，通常被认为是最佳实践。例如：

- **配置RDB定期生成快照文件，用于灾难恢复。**
- **配置AOF以`everysec`同步策略运行，确保数据的完整性。**
- **定期进行AOF重写以保持文件大小在可管理范围内。**





## 3 Redis集群

### 3.1 面试题

面试题：你们用的是Redis单机还是Redis集群？Redis集群具体怎么做的？



一般在自己学习时用Redis单机，或者一些个人项目中用Redis单机，在生产项目中会使用Redis集群。

Redis单机就是部署一个Redis实例，它不具有高可用，当这个Redis实例挂了将直接影响系统的运行，并且当一台Redis实例不足以承担请求压力时将会影响系统的情况。

Redis集群就是多个Redis实例组成一个集群共同对外提供Redis服务，首先具有高可用性，一个Redis实例挂了还有其它Redis实例对外提供服务，不影响整个集群对外提供服务。还有就是可扩展性，当系统压力比较大时通过扩展Redis实例节点数即可增加集群的服务能力。



所以在生产中正规的项目一般都会使用Redis集群。

那Redis集群具体怎么做的？这个是考察你对Redis集群掌握多少，首先你得知道生产中你的项目用的Redis集群是什么模式，是主从结构、还是分片集群，因为模式不同对于应用程序配置连接Redis的方式也可能不同，其次是对集群之间数据同步、故障转移等基本特性的掌握。至于你的集群有多少Redis节点组成这个一般由运维人员进行部署，作为Java程序员并不清楚生产环境具体节点数是正常的。

所以下一步我们需要搞清楚Redis集群的结构是什么。



### 3.2 主从结构

#### 3.2.1 介绍

首先说说Redis 的主从结构，下图就是一个简单的Redis主从集群结构：

![zzj2bjx77opzv7xkye4c6uxunqf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/zzj2bjx77opzv7xkye4c6uxunqf.png)
> 图注：这张图展示了简单的Redis主从集群结构，由Redis客户端、1个master节点以及2个slave/replica节点构成。Redis客户端的写操作仅指向master节点，而读操作可分别指向两个从节点；同时，图中以虚线箭头标注了数据同步方向，master节点会将数据自动同步给两个slave/replica节点，这一结构实现了读写分离，可通过访问多个从节点来分担并发压力。

如图所示，主从集群中有一个master节点、多个slave节点（现在叫replica）组成。

特点：

- 写操作访问master节点，master会自动将数据同步给两个slave节点
- 读操作访问各个slave节点，从而分担并发压力



具体主从集群的搭建请参考“Redis集群搭建”文档，有兴趣的同学在课下可以自已搭建。

#### 3.2.2 主从同步原理

主从集群的master节点是如何把数据同步到slave节点呢？

主从之间通过全量同步、增量同步的方式完成数据同步。



##### 3.2.2.1 全量同步

主从第一次建立连接时，会执行**全量同步**，将master节点的所有数据都拷贝给slave节点，流程如下：

![u6wzbh5i4oddqnxledcciiixnbc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/u6wzbh5i4oddqnxledcciiixnbc.png)
> 图注：这张图是Redis主从结构全量同步的流程示意图，清晰呈现了主从节点间同步的三个阶段内容：第一阶段，从节点执行replicaof命令建立连接，向主节点请求数据同步，主节点判断为第一次同步后，向从节点返回自身的数据版本信息，从节点保存该版本信息；第二阶段，主节点执行bgsave生成RDB文件，同时将生成RDB期间记录在repl_baklog中的命令一并处理，向从节点发送RDB文件，从节点清空本地数据后加载该RDB文件；第三阶段，主节点向从节点发送repl_baklog中记录的命令，从节点执行接收到的这些命令，完成全量同步流程。

完整流程描述：

- `slave`节点请求数据同步
- `master`节点判断`replid`，发现不一致表示是第一次同步，进行全量同步

`master`通过`replid`判断是否是第一次同步，**`Replication Id`**简称`replid`，是数据集的标记，replid一致则是同一数据集。每个`master`都有唯一的`replid`，`slave`则会继承`master`节点的`replid`

- `master`将完整内存数据生成`RDB`，发送`RDB`到`slave`
- `slave`清空本地数据，加载`master`的`RDB`
- 全量同步完成，后边进行增量同步，命令记录在`repl_baklog`中，通过`offset`偏移量master持续将log中的命令发送给`slave`。



##### 3.2.2.2 增量同步

全量同步需要先做RDB，然后将RDB文件通过网络传输给slave，成本太高了。因此除了第一次做全量同步，其它大多数时候slave与master都是做**增量同步**。

什么是增量同步？就是只更新slave与master存在差异的部分数据。如图：

![wabkbhhzgongtyxuilscyg3yndp](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wabkbhhzgongtyxuilscyg3yndp.png)
> 图注：该图展示了Redis主从结构中增量同步的完整流程，分为两个阶段。第一阶段是同步初始交互：slave节点重启后（步骤1.0），向master节点发送psync指令及自身的replication ID和偏移量（步骤1.1）；master判断该replication ID是否与自身一致（步骤1.2），若不是首次同步，则向slave返回continue指令（步骤1.3）。第二阶段是数据传输阶段：master从环形的repl_baklog文件中，获取slave所需偏移量之后的数据（步骤2.1），将对应偏移量后的命令发送给slave（步骤2.2）；slave接收后执行这些命令（步骤2.3），完成增量同步。



那么master怎么知道slave与自己的数据差异在哪里呢?



这就要说到全量同步时的`repl_baklog`文件了。这个文件是一个固定大小的数组，只不过数组是环形，也就是说**角标到达数组末尾后，会再次从0开始读写**，这样数组头部的数据就会被覆盖。



`repl_baklog`中会记录Redis处理过的命令及`offset`，包括master当前的`offset`，和slave已经拷贝到的`offset`：

![vdfhba1vdoavl9xbu41cr8tsnke](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vdfhba1vdoavl9xbu41cr8tsnke.png)
> 图注：图片展示了Redis主从结构中增量同步的原理。图中有一个圆环，左侧绿色部分代表slave，右侧红色部分代表master。slave从master处获取增量数据，slave与master的offset之间的差异即slave需要增量拷贝的数据。随着不断有数据写入，master的offset逐渐变大，slave也不断追赶，直到数组被填满，此时新数据写入会覆盖数组中的旧数据。

slave与master的offset之间的差异，就是salve需要增量拷贝的数据了。



随着不断有数据写入，master的offset逐渐变大，slave也不断的拷贝，追赶master的offset：

![ayf8bpmfnoullhxgutac58fhnzg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ayf8bpmfnoullhxgutac58fhnzg.png)
> 图注：图片展示了Redis主从结构中主从同步原理的增量同步示意图。图中左侧为slave，右侧为master，两者间用绿色箭头连接。master和slave的同步状态用绿色和红色环形图表示，绿色部分代表已同步数据，红色部分代表未同步数据。随着数据写入，master的offset逐渐变大，slave不断追赶，直到数组被填满，此时新数据写入会覆盖旧数据。该图直观呈现了slave与master的offset差异及增量拷贝数据的追赶过程。

直到数组被填满：

![ld8nbswfho8tkwxeotvcsp5inpd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ld8nbswfho8tkwxeotvcsp5inpd.png)
> 图注：图片展示了Redis主从结构中增量同步的原理。图中以绿色圆环表示`repl_baklog`，其中红色部分代表master当前的`offset`，绿色部分是slave已经拷贝到的`offset`。slave与master的offset之间的差异，即slave需要增量拷贝的数据。随着不断有数据写入，master的offset逐渐变大，slave不断追赶，直到数组被填满，此时若有新数据写入，会覆盖数组中的旧数据。

此时，如果有新的数据写入，就会覆盖数组中的旧数据。不过，旧的数据只要是绿色的，说明是已经被同步到slave的数据，即便被覆盖了也没什么影响。因为未同步的仅仅是红色部分：

![lhvfb8qh8ofvlvxtbdzctopbnhe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/lhvfb8qh8ofvlvxtbdzctopbnhe.png)
> 图注：图片展示了Redis主从结构中增量同步的原理示意图。图中左侧为slave，右侧为master，master指向slave。slave部分被绿色和红色区域覆盖，绿色区域代表已同步数据，红色区域代表未同步数据。该图与上下文紧密相关，直观呈现了旧数据被绿色覆盖，即便被覆盖也没影响，以及master继续写入新数据导致master的`offset`覆盖`repl_baklog`中旧数据，直到将slave现在的`offset`覆盖的同步原理。

但是，如果slave出现网络阻塞，导致master的`offset`远远超过了slave的`offset`： 

![nqp5baynaot6gfx2lgcc3onlnjq](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/nqp5baynaot6gfx2lgcc3onlnjq.png)
> 图注：图片展示了Redis主从结构中增量同步的原理。图中红色部分代表master端的数据，绿色部分代表slave端的数据。slave端的数据被标注为“slave”，master端的数据被标注为“master”。该图与上下文紧密相关，用于直观呈现master和slave端数据的同步情况，帮助理解当slave出现网络阻塞导致master的`offset`远超slave的`offset`时，master继续写入新数据会覆盖`repl_baklog`中旧数据，直到覆盖slave当前`offset`的现象。

如果master继续写入新数据，master的`offset`就会覆盖`repl_baklog`中旧的数据，直到将slave现在的`offset`也覆盖：

![fqycbjadqo5zh5xit5bcubjznwh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/fqycbjadqo5zh5xit5bcubjznwh.png)
> 图注：图片展示了Redis主从结构中增量同步的原理。图中以红色和绿色线条分别表示master和slave的`offset`，绿色部分代表已同步数据，红色部分为未同步数据。当slave出现网络阻塞时，master的`offset`会远远超过slave的`offset`，若master继续写入新数据，master的`offset`会覆盖`repl_baklog`中旧的数据，直至将slave现在的`offset`也覆盖。此图直观呈现了增量同步中master与slave数据同步关系及未同步数据位置。

棕色框中的红色部分，就是尚未同步，但是却已经被覆盖的数据。此时如果slave恢复，需要同步，却发现自己的`offset`都没有了，无法完成增量同步了。只能做**全量同步**。



`repl_baklog`大小有上限，写满后会覆盖最早的数据。如果slave断开时间过久，导致尚未备份的数据被覆盖，则无法基于`repl_baklog`做增量同步，只能再次全量同步。



#### 3.2.3 总结

Redis全量同步和增量同步区别？

- 全量同步：

主从第一次建立连接时，会执行**全量同步.**

从节点的offset被主的offset覆盖后需要全量同步。

master将完整内存数据生成RDB，发送RDB到slave。

- 增量同步：

slave节点断开又恢复，并且在`repl_baklog`中能找到offset时执行增量同步。

slave提交自己的offset到master，master获取repl_baklog中从offset之后的命令给slave。



### 3.3 Redis哨兵

#### 3.3.1 哨兵工作原理

主从结构中master节点的作用非常重要，一旦故障就会导致集群不可用。那么有什么办法能保证主从集群的高可用性呢？

Redis提供了`哨兵`（`Sentinel`）机制来监控主从集群监控状态，确保集群的高可用性。

下图是哨兵集群作用原理图：

![cnyzbbog8opbvixc2lgchabnnre](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/cnyzbbog8opbvixc2lgchabnnre.png)
> 图注：图片展示了Redis哨兵集群作用原理图。图中有三个Sentinel（哨兵）节点，分别位于上方，它们通过“监控集群状态”箭头指向下方的Redis集群。Redis集群包含master（主节点）和两个slave（从节点），master与slave间有“数据同步”箭头。master还与RedisClient（客户端）有“服务状态变更通知”箭头。该图直观呈现了哨兵监控集群状态、RedisClient接收服务状态变更通知及数据同步等关键环节，与上下文对哨兵状态监控等作用的介绍相契合。

哨兵的作用如下：

- **状态监控**：`Sentinel` 会不断检查您的`master`和`slave`是否按预期

`Sentinel` 通过定时向`master`和`slave`发送ping判断是否下线，如果超过半数的`Sentinel` 判断master下线则认为是客观下线，按照一定的规则在salve中选择一个作为新的master。

- **故障恢复（failover）**：如果`master`故障，`Sentinel`会将一个`slave`提升为`master`。当故障实例恢复后会成为`slave`

在所有Sentinel中找一个Leader，由该Leader向新master发送`slaveof no one`命令，让该节点成为`master`。

`Leader`给所有其它`slave`发送`slaveof`命令让slave成为新master的slave。

- **状态通知**：`Sentinel`充当`Redis`客户端的服务发现来源，当集群发生`failover`时，会将最新集群信息推送给`Redis`的客户端





##### 3.3.1.1 状态监控

`Sentinel`基于心跳机制监测服务状态，每隔1秒向集群的每个节点发送ping命令，并通过实例的响应结果来做出判断：

- **主观下线（sdown）**：如果某sentinel节点发现某Redis节点未在规定时间响应，则认为该节点主观下线。
- **客观下线(odown)**：若超过指定数量（通过`quorum`设置）的sentinel都认为该节点主观下线，则该节点客观下线。quorum值最好超过Sentinel节点数量的一半，Sentinel节点数量至少3台。

如图：

![mkskbba3aozp9wxlpmkczbi5nff](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mkskbba3aozp9wxlpmkczbi5nff.png)
> 图注：图片展示了Redis哨兵工作原理中状态监控的示意图。上方有三个Sentinel（哨兵）节点，下方有master（主节点）和两个slave（从节点）。master与两个slave间有数据同步箭头。当master出现故障时，两个Sentinel会判断slave与master断开时间，若超过`down-after-milliseconds*10`则排除该slave，再依据slave的`slave-priority`值选择新的master。该图直观呈现了哨兵发现master故障后的判断流程，与上下文对哨兵选择新master依据的描述相呼应。



一旦发现master故障，sentinel需要在salve中选择一个作为新的master，选择依据是这样的：

- 首先会判断slave节点与master节点断开时间长短，如果超过`down-after-milliseconds * 10`则会排除该slave节点
- 然后判断slave节点的`slave-priority`值，越小优先级越高，如果是0则永不参与选举（默认都是1）。
- 如果`slave-prority`一样，则判断slave节点的`offset`值，越大说明数据越新，优先级越高
- 最后是判断slave节点的`run_id`大小，越小优先级越高（`通过info server可以查看run_id`）。

对应的官方文档如下：

<bookmark name="High availability with Redis Sentinel" href="https://redis.io/docs/management/sentinel/#replica-selection-and-priority"></bookmark>





问题来了，当选出一个新的master后，该如何实现身份切换呢？

大概分为两步：

- 在多个`sentinel`中选举一个`leader`
- 由`leader`执行`failover`（故障转移）



##### 3.3.1.2 选举leader

首先，Sentinel集群要选出一个执行`failover`的Sentinel节点，可以成为`leader`。要成为`leader`要满足两个条件：

- 最先获得超过半数的投票
- 获得的投票数不小于`quorum`值

而sentinel投票的原则有两条：

- 优先投票给目前得票最多的
- 如果目前没有任何节点的票，就投给自己

比如有3个sentinel节点，`s1`、`s2`、`s3`，假如`s2`先投票：

- 此时发现没有任何人在投票，那就投给自己。`s2`得1票
- 接着`s1`和`s3`开始投票，发现目前`s2`票最多，于是也投给`s2`，`s2`得3票
- `s2`称为`leader`，开始故障转移

不难看出，**谁先投票，谁就会称为leader**，那什么时候会触发投票呢？

答案是**第一个确认master客观下线的人会立刻发起投票，一定会成为leader**。



OK，`sentinel`找到`leader`以后，该如何完成`failover`呢？



##### 3.3.1.3 failover

我们举个例子，有一个集群，初始状态下7001为`master`，7002和7003为`slave`：

![vgplbsyc3omfiqxbqntcigjinoh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vgplbsyc3omfiqxbqntcigjinoh.png)
> 图注：图片展示了Redis哨兵工作原理中master发生故障时的集群状态。上方为Sentinel哨兵，下方有三个节点，7001为master，7002和7003为slave。7002和7003节点上分别有“slave”标识和星星图案。该图与上下文紧密相关，直观呈现了集群初始状态，为后续master发生故障，slave1当选，以及故障转移流程的介绍提供可视化参考。

假如master发生故障，slave1当选。则故障转移的流程如下：

1）`sentinel`给备选的`slave1`节点发送`slaveof no one`命令，让该节点成为`master`

![h6a6bcopwoqbbbxdj1zcg5hvnif](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/h6a6bcopwoqbbbxdj1zcg5hvnif.png)
> 图注：图片展示了Redis哨兵执行故障转移流程中的第一步。上方为Sentinel哨兵，下方有三个节点，分别为master 7002、master 7001和slave2 7003。Sentinel向master 7002发送“执行slaveof no one”命令，让其成为master，同时向master 7001和slave2 7003发送“slaveof 192.168.150.101 7002”命令，让它们隶属于新master。该图直观呈现了故障转移流程中Sentinel执行命令的场景。



2）`sentinel`给所有其它`slave`发送`slaveof 192.168.150.101 7002` 命令，让这些节点隶属于新`master`，也就是`7002`的`slave`节点，开始从新的`master`上同步数据。

![vd4ubdhsgo7oijxj7zcccogmniv](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vd4ubdhsgo7oijxj7zcccogmniv.png)
> 图注：这张图展示了Redis哨兵故障转移（failover）流程的中间步骤，与上下文中的故障转移操作对应。图中橙色的Sentinel节点通过绿色箭头指向三个被红框标注的Redis节点：编号7002和7001的节点被标为master，编号7003的节点被标为slave2。左侧箭头旁标注步骤1“执行slaveof no one”，对应将备选的原从节点转为新主节点的操作，右侧箭头旁标注步骤2“slaveof 192.168.150.101 7002”，对应将其余原从节点改为新主节点的从节点的操作，清晰呈现了故障转移阶段的节点角色与对应命令。



3）最后，当故障节点恢复后会接收到哨兵信号，执行`slaveof 192.168.150.101 7002`命令，成为`slave`：

![vr7ob4g8no4unex9vwpcpnahntk](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vr7ob4g8no4unex9vwpcpnahntk.png)
> 图注：图片展示了Redis哨兵故障转移流程。上方为Sentinel，下方有三个节点，分别为master 7002、master 7001和slave2 7003。Sentinel执行slaveof no one命令，slave2执行slaveof 192.168.150.101 7002命令，master 7001执行slaveof 192.168.150.101 7002命令。该图与上文介绍的故障转移流程内容相呼应，直观呈现了各节点执行命令的顺序及对应关系。



#### 3.3.2 总结

Redis哨兵的三个作用是什么？

- 集群监控

Redis哨兵每隔1秒向主从节点发送一次ping命令，如果超过一定时间没有相向则认为是主观下线（`sdown`）

如果大多数Redis哨兵都认为实例主观下线，则判定服务客观下线（`odown`）

- 故障恢复

首先要在Redis哨兵中选出一个`leader`，由leader执行`failover`

- 状态通知

Redis哨兵从slave中选取master后会通过给redis客户端。



Redis哨兵如何判断一个redis实例是否健康？

- Redis哨兵每隔1秒向主从节点发送一次ping命令，如果超过一定时间没有相向则认为是主观下线（`sdown`）
- 如果大多数Redis哨兵都认为实例主观下线，则判定服务客观下线（`odown`）



故障转移步骤有哪些？

- 首先要在Redis哨兵中选出一个`leader`，由leader执行`failover`
- 选定一个`slave`作为新的`master`，执行`slaveof no one`，切换到master模式
- 然后让所有节点都执行`slaveof` 新master
- 修改故障节点配置，添加`slaveof` 新master



Redis哨兵选举leader的依据是什么？

- 票数超过Redis哨兵节点数量1半
- 票数超过设定数量（quorum）
- 一般情况下最先发起failover的节点会当选



Redis哨兵从slave中选取master的依据是什么？

- 首先会判断slave节点与master节点断开时间长短，如果超过`down-after-milliseconds * 10`则会排除该slave节点
- 然后判断slave节点的`slave-priority`值，越小优先级越高，如果是0则永不参与选举（默认都是1）。
- 如果`slave-prority`一样，则判断slave节点的`offset`值，越大说明数据越新，优先级越高
- 最后是判断slave节点的`run_id`大小，越小优先级越高（`通过info server可以查看run_id`）。



### 3.4 Redis分片集群

#### 3.4.1 介绍

主从模式可以解决高可用、高并发读的问题。但依然有两个问题没有解决：

- 海量数据存储
- 高并发写



要解决这两个问题就需要用到分片集群了。分片的意思，就是把数据拆分存储到不同节点，这样整个集群的存储数据量就更大了。

Redis分片集群的结构如图：

![islibci3dol7hbx8yoxcpesqnqd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/islibci3dol7hbx8yoxcpesqnqd.jpg)
> 图注：图片展示了Redis分片集群的结构。图中有多个master节点，每个master节点下有slave节点，master节点间通过绿色箭头表示的心跳进行健康状态监测，slave节点间通过绿色箭头表示的数据同步。每个master节点保存不同分片数据，master节点间及master与slave节点间均存在数据同步关系。该图与上下文紧密相关，直观呈现了分片集群中master和slave节点的分布及数据同步、心跳监测等关键信息。

分片集群特征：

-  集群中有多个master，每个master保存不同分片数据 ，解决海量数据存储问题
-  每个master都可以有多个slave节点 ，确保高可用
-  master之间通过ping监测彼此健康状态 ，类似哨兵作用
-  客户端请求可以访问集群任意节点，最终都会被转发到数据所在节点 





#### 3.4.2 散列插槽

数据要分片存储到不同的Redis节点，肯定需要有分片的依据，这样下次查询的时候才能知道去哪个节点查询。很多数据分片都会采用一致性hash算法。而Redis则是利用散列插槽（**`hash slot`**）的方式实现数据分片。

详见官方文档：

<bookmark name="Scale with Redis Cluster" href="https://redis.io/docs/management/scaling/#redis-cluster-101"></bookmark>



在Redis集群中，共有16384个`hash slots`，集群中的每一个master节点都会分配一定数量的`hash slots`。具体的分配在集群创建时就已经指定了：

![](https://feishu.cn/file/NY72bHfGuop24zxvdascIcN0nEd)

如图中所示：

- Master[0]，本例中就是7001节点，分配到的插槽是0\~5460
- Master[1]，本例中就是7002节点，分配到的插槽是5461\~10922
- Master[2]，本例中就是7003节点，分配到的插槽是10923\~16383



当我们读写数据时，Redis基于`CRC16` 算法对`key`做`hash`运算，得到的结果与`16384`取余，就计算出了这个`key`的`slot`值。然后到`slot`所在的Redis节点执行读写操作。

![ypgtbe9oaov7zjxz7hiczvl6nvf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ypgtbe9oaov7zjxz7hiczvl6nvf.png)
> 图注：图片展示了Redis散列插槽的计算原理。上方是CRC16运算，其结果指向下方的16384个槽位，分为三个Redis Group，每个Group负责一部分槽位。其中5460 - 5461槽位由第一个Group负责，10922 - 10923槽位由第二个Group负责，16383槽位由第三个Group负责。该图与上下文紧密相关，直观呈现了Redis散列插槽的分片机制，帮助理解`hash slot`的计算情况。

不过`hash slot`的计算也分两种情况：

- 当`key`中包含`{}`时，根据`{}`之间的字符串计算`hash slot`
- 当`key`中不包含`{}`时，则根据整个`key`字符串计算`hash slot`



例如：

- key是`user`，则根据`user`来计算hash slot
- key是`user:{age}`，则根据`age`来计算hash slot



我们来测试一下，先于`7001`建立连接：

```Bash
# 进入容器
docker exec -it r1 bash
# 进入redis-cli,这里要加-c表示连接集群
redis-cli -c -p 7001
# 测试
set user jack
```

结果如下：

![djzmb0j8boebybxonj4c2yddnwg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/djzmb0j8boebybxonj4c2yddnwg.png)
> 图注：这张图片展示了Redis客户端测试哈希插槽跳转的运行结果：在7001端口的Redis节点执行`redis-cli -c -p 7001`并输入`set user jack`命令后，控制台明确显示被重定向到了编号为5474的哈希插槽，对应的节点是192.168.150.101:7002，印证了前文提到的“当key不含{}时，根据整个key计算哈希插槽，客户端会自动跳转至对应插槽所在节点”的内容，红框重点突出了跳转的插槽编号与目标节点信息。

可以看到，客户端自动跳转到了`5474`这个`slot`所在的`7002`节点。



现在，我们添加一个新的key，这次加上`{}`：

```Bash
# 试一下key中带{}
set user:{age} 21

# 再试一下key中不带{}
set age 20
```

结果如下：

![](https://feishu.cn/file/QqAcbEsEOozjEXxxPhzcRB2jnUc)

可以看到`user:{age}`和`age`计算出的`slot`都是`741`。



#### 3.4.3 故障转移

分片集群的节点之间会互相通过ping的方式做心跳检测，超时未回应的节点会被标记为下线状态。当发现master下线时，会将这个master的某个slave提升为master。



#### 3.4.4 总结

Redis分片集群如何判断某个key应该在哪个实例？

- 将16384个插槽分配到不同的实例
- 根据key计算哈希值，对16384取余
- 余数作为插槽，寻找插槽所在实例即可



如何将同一类数据固定的保存在同一个Redis实例？

- Redis计算key的插槽值时会判断key中是否包含`{}`，如果有则基于`{}`内的字符计算插槽
- 数据的key中可以加入`{类型}`，例如key都以`{typeId}`为前缀，这样同类型数据计算的插槽一定相同



### 3.5 总结



面试题：你们用的是Redis单机还是Redis集群？Redis集群具体怎么做的？





面试题：Java程序如何访问Redis集群？

参考："Redis集群搭建"中的“Java客户端连接分片集群”章节。



## 4 Redis数据类型

### 4.1 面试题

说一下Redis的常用数据类型？

考察对Redis基本数据类型的掌握，要清楚每种数据类型的特点及常用的操作方法（增、删、查）。



说一下ZSet的底层结构？



### 4.2 五种数据类型

#### 4.2.1 回顾复习

我们常用的Redis数据类型有5种：String、hash、list、set、zset，还有一些高级数据类型，比如Bitmap、HyperLogLog（简介HLL）、GEO等，其底层都是基于上述5种基本数据类型。因此在Redis的源码中，其实只有5种数据类型。

下边对五种基本数据类型进行总结：

- 字符串(string)：普通字符串，常用

**SET** key value                                                  设置指定key的值

**GET** key                                        获取指定key的值

**SETEX** key seconds value         设置指定key的值，并将 key 的过期时间设为 seconds 秒

**SETNX** key value 只有在 key    不存在时设置 key 的值

- 哈希(hash)：适合存储对象

![oj03b4ludozrhuxn3nncvbxrnhh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/oj03b4ludozrhuxn3nncvbxrnhh.png)
> 图注：图片展示了Redis哈希数据类型的基本结构。左侧有一个key，右侧是一个哈希表，包含两个字段field1和field2，分别对应value1和value2。

**HSET** key field value             将哈希表 key 中的字段 field 的值设为 value

**HGET** key field                       获取存储在哈希表中指定字段的值

**HDEL** key field                       删除存储在哈希表中的指定字段

**HKEYS** key                              获取哈希表中所有字段

**HVALS** key                              获取哈希表中所有值

**HGETALL** key                         获取在哈希表中指定 key 的所有字段和值

- 列表(list)：按照插入顺序排序，可以有重复元素

![zq2zb1br2oys9zxiabqcaofenwc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/zq2zb1br2oys9zxiabqcaofenwc.png)
> 图注：图片展示了Redis列表列表（list）数据类型 addCriterion类型的关键信息。图中左侧为key，右侧是一个有序列表，包含a、b、c、d四个元素，a位于列表头部，d位于尾部。

**LPUSH** key value1 [value2]         将一个或多个值插入到列表头部

**LRANGE** key start stop                获取列表指定范围内的元素

**RPOP** key                                       移除并获取列表最后一个元素

**LLEN** key                                        获取列表长度

**BRPOP** key1 [key2 ] timeout       移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超    时或发现可弹出元素为止

- 集合(set)：无序集合，没有重复元素

![rneibzf9bova60xzoutcbzilnee](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/rneibzf9bova60xzoutcbzilnee.png)
> 图注：图片展示了Redis集合（set）数据类型的基本概念。左侧是一个矩形框，标注为“key”，右侧是一个椭圆形区域，标注为“value”，椭圆内有三个矩形框，分别标注为“a”“b”“e”。箭头从“key”指向“value”区域，表明key与value之间的关联。

**SADD** key member1 [member2]            向集合添加一个或多个成员

**SMEMBERS** key                                         返回集合中的所有成员

**SCARD** key                                                  获取集合的成员数

**SINTER** key1 [key2]                                   返回给定所有集合的交集

**SUNION** key1 [key2]                                 返回所有给定集合的并集

**SDIFF** key1 [key2]                                      返回给定所有集合的差集

**SREM** key member1 [member2]            移除集合中一个或多个成员

- 有序集合(sorted set / zset)：集合中每个元素关联一个分数（score），根据分数升序排序，没有重复元素

![u2y6blfcjokv1pxlbtic5t1jnun](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/u2y6blfcjokv1pxlbtic5t1jnun.png)
> 图注：图片展示了Redis有序集合（sorted set/zset）的数据结构示例。左侧有一个矩形框，标注为“key”，右侧是一个椭圆形区域，标注为“value”。椭圆内有三个矩形框，分别标注“a: 0.1”“b: 3.0”“e: 2.5”，其中“b”和“e”被红色箭头指向。

**ZADD** key score1 member1 [score2 member2]     向有序集合添加一个或多个成员，或者更新已存在成员的 分数

**ZRANGE** key start stop [WITHSCORES]                     通过索引区间返回有序集合中指定区间内的成员

**ZINCRBY** key increment member                              有序集合中对指定成员的分数加上增量 increment

**ZREM** key member [member ...]                                移除有序集合中的一个或多个成员



还需要掌握Redis中的通用命令，主要是针对key进行操作的相关命令：

- **KEYS** pattern  查找所有符合给定模式( pattern)的 key 
- **EXISTS** key  检查给定 key 是否存在
- **TYPE** key  返回 key 所储存的值的类型
- **TTL** key  返回给定 key 的剩余生存时间(TTL, time to live)，以秒为单位
- **DEL** key  该命令用于在 key 存在是删除 key

#### 4.2.2 总结

说一下Redis的常用数据类型？

![kx2wbxvfzotthqxwyakcnlwwn4b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/kx2wbxvfzotthqxwyakcnlwwn4b.png)
> 图注：这张图片是Redis五种数据类型的对应说明表格，表格分为数据类型、特征、适用场景、常用命令四列，涵盖String、List、Set、Hash、Sorted Set这五种类型。其中String的特征是存储单个值，适用于缓存等场景，常用命令为SET、GET；List是有序可重复的字符串集合，适配消息队列等场景，常用LPUSH、LRANGE；Set是无序不可重复的集合，用于标签系统等，常用SADD、SMEMBERS；Hash是字段-值对的无序散列，适合存储对象等，常用HSET、HGETALL；Sorted Set是有序的字符串集合且成员关联分数，用于排行榜场景，常用ZADD、ZRANGE、ZREVRANGE。

- **String：**存储单个值，适用于缓存和键值存储，常用命令：SET用于设置值，GET用于获取值。
- **List：**有序、可重复的字符串集合，适用于消息队列和发布/订阅系统，常用命令：LPUSH用于从列表左侧添加元素，LRANGE用于获取指定范围的元素。
- **Set：**无序、不可重复的字符串集合，适用于标签系统和好友关系等，常用命令：SADD用于向集合添加成员，SMEMBERS用于获取集合所有成员。
- **Hash：**字段-值对的无序散列，适用于存储对象、缓存和计数器，常用命令：HSET用于设置字段值，HGETALL用于获取散列的所有字段和值。
- **Sorted Set：**有序的字符串集合，每个成员关联一个分数，适用于排行榜和按分数范围获取成员，常用命令：ZADD用于添加成员及其分数，ZRANGE用于获取指定范围的成员。



### 4.2 SkipList

SkipList（跳表）首先是链表，但与传统链表相比有几点差异：

- 元素按照升序排列存储
- 节点可能包含多个指针，指针跨度不同。



传统链表只有指向前后元素的指针，因此只能顺序依次访问。如果查找的元素在链表中间，查询的效率会比较低。

![khlqby8zlom6kixz4becgvxxnbk](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/khlqby8zlom6kixz4becgvxxnbk.png)
> 图注：这张图展示了三种常见的链表结构，用于辅助理解与Redis的SkipList相关的链表基础内容。图上方标注的“单链表”是仅有指向后继节点的next指针，末端指向NULL的链式结构；中间标注的“循环链表”是在单链表基础上，末端节点的next指针指向首部，形成首尾相接的闭环；下方标注的“双向链表”则在每个节点增设了pre指针，可同时指向前后节点，末端同样指向NULL。这些基础链表结构，是理解SkipList多级跳跃指针特性的前提。

而SkipList则不同，它内部包含跨度不同的多级指针，可以让我们跳跃查找链表中间的元素，效率非常高。

其结构如图：

![mhzobpmesox0xlx5svscfrmin1d](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mhzobpmesox0xlx5svscfrmin1d.png)
> 图注：该图片展示了Redis中SkipList的结构，最上方的1号节点拥有跨度为9的4级指针，指向数值为10的节点。图中设置了多级指针，同一层级的指针会连接特定数值的节点，指针旁标注了对应的跨度，而最下方是按数值从1到20依次排列的底层链表，各级间的节点也通过虚线对应关联，直观呈现出SkipList可通过多级指针实现跳跃查找的结构特点，与上下文介绍的SkipList的跳跃查找机制相呼应，能帮助理解其高效查找的原理。

我们可以看到1号元素就有指向3、5、10的多个指针，查询时就可以跳跃查找。例如我们要找大小为14的元素，查找的流程是这样的：



![fv33bsekfo8iolxlrglclnodnog](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/fv33bsekfo8iolxlrglclnodnog.png)
> 图注：该图为Redis中SkipList的结构示意图，展示了其包含跨度不同的多级指针的特征。图中最顶部，大小为1的节点有指向大小为10的指针，跨度为9；从上往下第二层，大小1的节点经跨度为4的指针指向大小5的节点，大小5的节点经跨度为5的指针指向大小10的节点；第三层的大小10节点经绿色指针指向大小13的节点，大小13的节点向下指向最底层的大小13节点，再经指向大小14的指针。最底层为按1到20顺序排列的单链表，与上下文描述的SkipList跳跃查找特性相契合，能通过多级指针实现快速定位目标元素。

- 首先找元素1节点最高级指针，也就是4级指针，起始元素大小为1，指针跨度为9，可以判断出目标元素大小为10。由于14比10大，肯定要从10这个元素向下接着找。
- 找到10这个元素，发现10这个元素的最高级指针跨度为5，判断出目标元素大小为15，大于14，需要判断下级指针
- 10这个元素的2级指针跨度为3，判断出目标元素为13，小于14，因此要基于元素13接着找
- 13这个元素最高级级指针跨度为2，判断出目标元素为15，比14大，需要判断下级指针。
- 13的下级指针跨度为1，因此目标元素是14，刚好于目标一致，找到。



这种多级指针的查询方式就避免了传统链表的逐个遍历导致的查询效率下降问题。在对有序数据做随机查询和排序时效率非常高。



跳表的结构体如下：

```C
typedef struct zskiplist {
    // 头尾节点指针
    struct zskiplistNode *header, *tail;
    // 节点数量
    unsigned long length;
    // 最大的索引层级
    int level;
} zskiplist;
```

可以看到SkipList主要属性是header和tail，也就是头尾指针，因此它是支持双向遍历的。

跳表中节点的结构体如下：

```C
typedef struct zskiplistNode {
    sds ele; // 节点存储的字符串
    double score;// 节点分数，排序、查找用
    struct zskiplistNode *backward; // 前一个节点指针
    struct zskiplistLevel {
        struct zskiplistNode *forward; // 下一个节点指针
        unsigned long span; // 索引跨度
    } level[]; // 多级索引数组
} zskiplistNode;
```

每个节点中都包含ele和score两个属性，其中score是得分，也就是节点排序的依据。ele则是节点存储的字符串数据指针。

### 4.4 总结

Redis源码中`zset`(`SortedSet`)的结构体如下：

```C
typedef struct zset {
    dict *dict; // dict，底层就是HashTable
    zskiplist *zsl; // 跳表
} zset;
```

zset的底层有两种结果：HashTable、跳表。



Redis的`SortedSet`底层的数据结构是怎样的？





## 5.Redis内存回收

### 5.1 面试题

说一下Redis内存回收机制。

考察对Redis内存过期策略和内存淘汰策略的理解。



Redis之所以性能强，最主要的原因就是基于内存存储。然而单节点的Redis其内存大小不宜过大，会影响持久化或主从同步性能。

我们可以通过修改redis.conf文件，添加下面的配置来配置Redis的最大内存：

```Properties
maxmemory 1gb
```

当内存达到上限，就无法存储更多数据了。因此，Redis内部会有两套内存回收的策略：

- 内存过期策略
- 内存淘汰策略



### 5.2.内存过期处理

存入Redis中的数据可以配置过期时间，到期后再次访问会发现这些数据都不存在了，也就是被过期清理了。

#### 5.2.1.过期命令

Redis中通过`expire`命令可以给KEY设置`TTL`（过期时间），例如：

```Bash
# 写入一条数据
set num 123
# 设置20秒过期时间
expire num 20
```



不过set命令本身也可以支持过期时间的设置：

```Shell
# 写入一条数据并设置20s过期时间
set num EX 20
```



当过期时间到了以后，再去查询数据，会发现数据已经不存在。 



#### 5.2.2.过期策略

那么问题来了：

- Redis如何判断一个KEY是否过期呢？
- Redis又是何时删除过期KEY的呢？



Redis是何时删除过期KEY的呢？

Redis并不会在KEY过期时立刻删除KEY，因为要实现这样的效果就必须给每一个过期的KEY设置时钟，并监控这些KEY的过期状态。无论对CPU还是内存都会带来极大的负担。

Redis的过期KEY删除策略有两种：

- 惰性删除
- 周期删除



**惰性删除：**

当一个客户端尝试访问一个键时，Redis 会检查该键是否已经过期。如果过期，Redis 将删除该键，并返回一个表示键不存在的响应给客户端。这种方式确保了对内存的有效管理，但可能在高并发访问过期键的情况下导致性能下降。



**定期删除**：

Redis 还有一个后台线程，以一定的频率检查过期的键并删除它们。这个频率可以通过 `server.hz` 参数配置。`server.hz` 定义了 Redis 服务器每秒运行维护任务的次数，包括但不限于过期键的清理。默认情况下，`server.hz` 的值为 10，意味着每秒进行 10 次检查。



如何调整 `server.hz`？

- 提高频率：如果您的应用中有很多短生命周期的键，并且希望更快地回收这些键所占用的内存，可以考虑增加 `server.hz` 的值。但是请注意，更高的 `server.hz` 会增加 CPU 的使用率，因为它会导致更频繁的后台任务执行。
- 降低频率：如果您发现 Redis 服务器的 CPU 使用率较高，而您的应用对过期键的清理速度要求不是特别严格，可以考虑降低 `server.hz` 的值以减少 CPU 负载。

配置方法：

要修改 `server.hz` 的值，您可以在 Redis 的配置文件 `redis.conf` 中找到相应的设置，并根据需要更改它。

### 5.3.内存淘汰策略

对于某些特别依赖于Redis的项目而言，仅仅依靠过期KEY清理是不够的，内存可能很快就达到上限。因此Redis允许设置内存告警阈值，当内存使用达到阈值时就会主动挑选部分KEY删除以释放更多内存。这叫做**内存淘汰**机制。



Redis支持8种不同的内存淘汰策略：

1）noeviction： 不删除，直接返回报错信息。

2）volatile-lfu：在设置了过期时间的key中，移除最近最少（最少频率使用）使用的key。

4）volatile-lru：在设置了过期时间的key中，移除最近最久未使用的key。

3）volatile-ttl： 在设置了过期时间的key中，移除准备过期的key。

5）volatile-random：在设置了过期时间的key中，随机移除某个key。

6）allkeys-random：随机移除某个key。

7）allkeys-lru：移除最久未使用的key。

8）allkeys-lfu：移除最近最少使用的key。



比较容易混淆的有两个算法：

- **LFU**（**`L`**`east `**`F`**`requently `**`U`**`sed`），最少频率使用。会统计每个key的访问频率，值越小淘汰优先级越高。
- **LRU**（**`L`**`east `**`R`**`ecently `**`U`**`sed`），最近最久未使用。用当前时间减去最后一次访问时间，这个值越大则淘汰优先级越高。





不过这里大家要注意一下：Redis中的`KEY`可能有数百万甚至更多，每个KEY都有自己访问时间或者逻辑访问次数。我们要找出时间最早的或者访问次数最小的，难道要把Redis中**所有数据排序**？

要知道Redis的内存淘汰是在每次执行命令时处理的。如果每次执行命令都先对全量数据做内存排序，那命令的执行时长肯定会非常长，这是不现实的。



所以Redis采取的是**抽样法**，即每次抽样一定数量（`maxmemory_smples`）的key，然后基于内存策略做排序，找出淘汰优先级最高的，删除这个key。这就导致Redis的算法并不是真正的**LRU**，而是一种基于抽样的**近似LRU算法**。



### 5.4.总结

Redis何时删除过期KEY？如何删除？/删除策略？



Redis淘汰策略有哪几种？

---
> 来源：飞书技术知识库（脱敏版） · 08 面试知识库 / 微服务面试题 / 微服务面试篇（下） · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/H0bawKb5xiWayLkGCdjcmRBXn4b)
