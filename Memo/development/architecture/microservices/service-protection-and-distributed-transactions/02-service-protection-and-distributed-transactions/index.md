---
type: memo
date: 2026-11-06
aliases:
  - "02 服务保护、分布式事务"
---
# 02 服务保护、分布式事务

## 学习目标

1. 能够说出什么是微服务雪崩
2. 能够说出常用的微服务保护方案和技术方案
3. 能够说出什么是熔断降级
4. 能够基于FallbackFactory编写降级方法
5. 能够使用Sentinel配置熔断策略并测试通过
6. 能够使用Sentinel配置限流策略并测试通过
7. 能够基于Sentinel注解编写降级方法
8. 能够使用Sentinel配置线程隔离并测试通过
9. 能够说出CAP原理
10. 能够使用Seata实现分布式事务控制
11. 能够说出Seata AT模式的工作原理

## 1 微服务保护

保护服务的健康性和可用性！！！

### 1.1.微服务保护方案

#### 1.1.1 微服务雪崩问题

上次课我们学习了微服务之间的远程调用，微服务通过远程调用进行协作完成业务流程，试想如果出现下边的现象会导致什么问题。

假如商品服务业务并发较高，占用过多Tomcat连接。可能会导致商品服务的所有接口响应时间增加，延迟变高，甚至是长时间阻塞直至查询失败。

此时查询购物车业务需要等待商品查询结果，从而导致购物车业务的响应时间也变长，甚至也阻塞直至无法访问。而此时如果查询购物车的请求较多，可能导致购物车服务的Tomcat连接占用较多，所有接口的响应时间都会增加，整个服务性能很差， 甚至不可用。



<whiteboard token="QTTRwN7EYhQdDEbt27zcFtuCn5c"></whiteboard>

依次类推，整个微服务群中与购物车服务、商品服务等有调用关系的服务可能都会出现问题，最终导致整个集群不可用。

<whiteboard token="VRfawayuJhfgb5bHrwlcYH6nnqb"></whiteboard>

这就是**级联失败**问题，或者叫**雪崩**问题。



保证服务运行的**健壮性**，避免级联失败导致的**雪崩**问题，就属于**微服务保护**。这章我们就一起来学习一下微服务保护的常见方案以及对应的技术。

开源思想：

1. 「避免重复造轮子」的核心是「**优先复用成熟开源成果，聚焦核心业务创新，理性再造差异化组件**」，是平衡效率与质量的关键；
2. 这一思想在你的开发场景中（如 mybatis-plus）的体现：复用MP解决日常sql开发的问题，无需从零实现，将精力聚焦在业务核心逻辑（如购物车、订单处理）；
3. 实践落地的关键步骤：识别通用 / 核心问题 → 筛选优质开源轮子 → 优先集成 / 二次开发 → 避免过度复用误区；gfdg
4. 开源生态的良性循环：「**复用 - 创新 - 开源 - 反哺**」，这也是「避免重复造轮子」思想的最终价值体现。

#### 1.1.2 微服务保护方案

##### 1.1.2.1 方案介绍

AI:Spring cloud微服务保护的方案

<callout emoji="🏕️">
1. 熔断
2. 降级
3. 超时
4. 线程隔离
5. 限流
</callout>

<whiteboard token="UQJHwxufHh1RJXbYlAkc35o3n5c"></whiteboard>

Spring Cloud微服务架构中的服务保护是非常重要的，它能够确保系统的稳定性和可用性，特别是在面对突发流量或者服务异常的情况下。常用的微服务保护方案包括但不限于以下几个方面：

1. **熔断 (Circuit Breaker)**

熔断机制用于在服务出现问题时快速失败，避免调用链路中的服务相互等待，导致整体系统响应变慢甚至不可用。

如何快速失败呢？当服务的错误率达到一定程度时，**断路器**（相当于保险丝）会打开，直接返回错误而不是尝试调用服务。一段时间后，断路器会处于半开状态尝试调用服务，如果服务恢复正常，则关闭断路器。

  由**断路器**统计业务执行的异常比例，如果超出阈值则会**熔断**该业务，拦截访问该业务的一切请求。

<whiteboard token="CPDCwpvQ2h4gj4bjtXjcnZC1nBe"></whiteboard>

1. **降级 (Degradation)**

断路器会统计访问某个服务的请求数量，统计**服务提供方**的异常比例，当比例过高表明该接口会影响到其它服务，应该**拒绝调用该接口（异常期间不再调用该下游接口）**，而是直接走降级逻辑。

降级逻辑 即提供一个简化的响应或者默认的响应来代替正常的服务调用。这样可以保证核心业务不受影响，非核心业务暂时被限制或关闭。

<whiteboard token="TpQuw3sqWhvZXxbVipJcpO9Onte"></whiteboard>

1. **超时 (Timeout)**

设置合理的超时时间可以避免长时间等待响应导致的问题。当请求超时时，可以选择快速失败并返回错误信息，或者重试等策略。

1. **线程隔离 (Thread Isolation)**

线程隔离是指为每个服务分配独立的线程池，这样即使某个服务出现问题也不会影响到其他服务。

线程隔离的思想来自轮船的舱壁模式：



![oocibqqdrowfnvxze1nc1gltn2b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/oocibqqdrowfnvxze1nc1gltn2b.png)
> 图注：图片展示了轮船的舱壁模式示意图，上方标注“Watertight bulkheads”（水密舱壁）。图中轮船被隔板分割为多个密闭舱，当船体被冰山刺穿后，只有受损部分的密闭舱进水，而其他舱由于相互隔离，不会进水。该图与上下文关系紧密，用以形象说明线程隔离的思想，即为每个服务分配独立的线程池，即使某个服务出现问题也不会影响到其他服务，如同轮船密闭舱相互隔离，损坏部分不会影响其他部分。

轮船的船舱会被隔板分割为N个相互隔离的**密闭舱**，假如轮船触礁进水，只有损坏的部分密闭舱会进水，而**其他舱**由于相互隔离，并不会进水。这样就把进水控制在部分船体，避免了整个船舱进水而沉没。

为了避免某个接口故障或压力过大导致整个服务不可用，我们可以限定每个接口可以使用的资源范围，也就是将其“隔离”起来。



<whiteboard token="TBY9wVSEphVLymbLfPmcEAzlnvd"></whiteboard>

如图所示，我们给查询购物车业务限定可用线程数量上限为20，这样即便查询购物车的请求因为查询商品服务而出现故障，也不会导致**服务器的线程资源**被耗尽，不会影响到其它接口。

1. **限流 (Rate Limiting)**

限流是最常见的服务保护措施之一，其目的是为了防止服务因为过大的流量而崩溃。

对于某些关键资源或者参数的访问，可以采取特殊的限流措施来防止这些热点成为瓶颈。

限流往往会有一个**限流器**，数量高低起伏的并发请求曲线，经过限流器就变的非常平稳。这就像是水电站的大坝，起到蓄水的作用，可以通过开关控制水流出的大小，让下游水流始终维持在一个平稳的量。



![ubu0bvoasozq64xu6pucaabqnye](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ubu0bvoasozq64xu6pucaabqnye.jpg)
> 图注：该图片配合微服务限流的内容，直观展示了限流的作用逻辑：左侧橙色带“请求”标识的离散请求流量，经过标注为“限流器”的蓝色管道后，变为稳定的绿色流量，最终流向右侧的“受保护的服务”；下方的折线图更清晰体现限流效果，左侧横轴为时间、纵轴为QPS的曲线呈现波动起伏的状态，经箭头作用后，右侧对应QPS曲线变为平稳的直线，说明限流器可将高低起伏的请求流量调控为平稳状态，避免关键资源因流量突发成为瓶颈。

可以通过以下几种方式进行限流: **限制请求的速率！**

基于令牌桶算法：允许一定数量的请求通过，超出则拒绝或排队等待。

基于滑动窗口：在一段时间内对请求进行计数，超过阈值则触发限流。



##### 1.1.2.2 实现工具

在Spring Cloud生态系统中，实现服务保护通常使用的工具包括：

Hystrix: 提供了熔断、限流、超时等功能。

Resilience4j: 是一个轻量级的库，提供了与Hystrix类似的功能，但设计更为现代和简洁。

**Sentinel:** 阿里巴巴开源的一款流量控制组件，特别适合微服务架构下的流量管理，提供了限流、熔断、降级等多种服务保护功能，并且支持热更新规则。



本课程讲解Sentinel。



### 1.2 熔断降级- Sentinel

#### 1.2.1.    Sentinel原理介绍-**断路器**

熔断降级是解决雪崩问题的重要手段，包括熔断和降级两个方案。

<whiteboard token="ApGiwTsv4hivuYbwiPbc485wnee"></whiteboard>

**熔断**是由**断路器**统计服务调用的异常比例、慢请求比例，如果超出阈值则会**熔断**该服务。即拦截访问该服务的一切请求；而当服务恢复时，断路器会放行访问该服务的请求。熔断发生在服务调用方即客户端。

**降级**是当遇到访问失败可以快速返回一些默认数据或者友好提示，用户体验会更好。熔断降级结合后是当线路断开后直接走降级线路避免再次去请求失败线路。降级方法需要在服务调用方即客户端实现。

断路器控制熔断和放行的流程如下：

![iihlbv7ssobd9gx5pz4caylanlb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/iihlbv7ssobd9gx5pz4caylanlb.png)
> 图注：图片展示了Sentinel断路器控制熔断和放行的流程。当达到熔断阈值时，断路器状态从Closed变为Open，服务调用被熔断，访问被拒绝。若熔断时间结束，断路器状态变为Half - Open，放行一次请求，根据结果做处理；若请求成功，则关闭断路器；若请求失败，则打开断路器。该图与上下文紧密相关，直观呈现了断路器在不同状态下的行为及转换逻辑。



断路器包括三个状态：

- closed：关闭状态，断路器放行所有请求，并开始统计异常比例、慢请求比例、异常数。超过阈值则切换到open状态
- open：打开状态，服务调用被**熔断**，访问被熔断服务的请求会被拒绝，快速失败，直接走降级逻辑。Open状态5秒后会进入half-open状态
- half-open：半开状态，放行一次请求，根据执行结果来判断接下来的操作。

  - 请求成功：则切换到closed状态
  - 请求失败：则切换到open状态



实现熔断降级做两件事：

- **编写服务降级逻辑**：就是服务调用失败后的处理逻辑，根据业务场景，可以抛出异常，也可以返回友好提示或默认数据。
- **异常统计和熔断**：统计服务提供方的异常比例，当比例过高表明该接口会影响到其它服务，应该拒绝调用该接口，而是直接走降级逻辑。这里我们用Sentinel完成。



#### 1.2.2. Sentinel安装与集成

**关闭防火墙！！！**

**关闭防火墙！！！**

**关闭防火墙！！！**

![yboebizekoknfgxcq5qcsx3ynkg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/yboebizekoknfgxcq5qcsx3ynkg.png)
> 图注：图片展示的是Windows Defender防火墙的自定义设置界面。界面上方有“自定义设置”标题，下方有“自定义各类网络的设置”说明。关键信息是“专用网络设置”和“公用网络设置”中均被红框突出显示的“关闭Windows Defender防火墙(不推荐)”选项，表明可选择关闭Windows Defender防火墙。该图片与文档中“1.2.2.1切换分支”部分内容相关，用于指导在切换分支前关闭Windows Defender防火墙的操作。



<callout emoji="🌅">
1. 安装sentinel
2. 访问sentinel界面
3. 服务集成
</callout>

##### 1.2.2.1 切换分支

将hmall-micro代码环境切换到dev_02分支。

注意：切换分支前要提交原当前分支的代码。

每位学生在dev_02分支练习完成后提交代码并切换回dev_01分支继续未完成的任务。

##### 1.2.2.2 安装Sentinel

实现服务保护的工具有很多，Spring Cloud Alibaba技术栈中Sentinel是实现服务保护的中间件。

Sentinel是阿里巴巴开源的一款服务保护框架，目前已经加入Spring Cloud Alibaba中。官方网站：

<bookmark name="home | Sentinel" href="https://sentinelguard.io/zh-cn/"></bookmark>

文档：[https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D](https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D)

Sentinel 的使用可以分为两个部分:

- **核心库**（Jar包）：不依赖任何框架/库，能够运行于 Java 8 及以上的版本的运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。在项目中引入依赖即可实现服务限流、隔离、熔断等功能。
- **控制台**（Dashboard）：Dashboard 主要负责管理推送规则、监控、管理机器信息等。



为了方便监控微服务，我们先把Sentinel的控制台搭建出来。

1）下载jar包

下载地址：

<bookmark name="Releases · alibaba/Sentinel" href="https://github.com/alibaba/Sentinel/releases"></bookmark>

也可以直接使用课前资料提供的版本：

![h3fdbcq7iobyb8xjrzrclad8nge](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/h3fdbcq7iobyb8xjrzrclad8nge.png)
> 图注：这张图片是电脑的文件资源管理器界面，路径为“新加卷 (D:) > 课程资料 > 服务框架 > day04-服务保护和分布式事务 > 资料”，界面中列出了多个文件与文件夹，包括seata文件夹、seata.tar压缩包、seata-at.sql与seata-tc.sql源文件，还有黑马程序员·研究院的雪崩测试.jmx文件；其中被蓝色高亮选中的是名为“sentinel-dashboard-1.8.6.jar”的JAR文件，该文件大小为221,52 KB，它对应了文档中提及的用于Sentinel安装的相关jar包资源。

2）运行

删掉旧容器：

```JavaScript
docker rm -f sentinel-dashboard
```



启动容器：

```JavaScript
docker run  --name sentinel-dashboard -d --restart=always -p 9090:8090 sentinel-dashboard:latest
```





其它启动时可配置参数可参考官方文档：

<bookmark name="启动配置项" href="https://github.com/alibaba/Sentinel/wiki/%E5%90%AF%E5%8A%A8%E9%85%8D%E7%BD%AE%E9%A1%B9"></bookmark>



3）访问

访问http://192.168.101.68:9090/ 页面，就可以看到sentinel的控制台了：

![ypwsbrpbzovzhzx6rsvcfbbunqg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ypwsbrpbzovzhzx6rsvcfbbunqg.png)
> 图注：图片展示的是Sentinel控制台登录页面。页面地址栏显示为“localhost:8090/#/login”。页面中央有Sentinel的标志及“Sentinel”字样。下方有两个输入框，分别用于输入用户名和密码，右侧各有一个“...”图标。底部有“登录”和“清空”按钮。该图片与文档中“本地运行sentinel”部分内容相关，说明登录Sentinel控制台的入口，需输入默认账号和密码（均为sentinel）后方可进入控制台，默认监控sentinel - dashboard服务本身。

需要输入账号和密码，默认都是：sentinel

登录后，即可看到控制台，默认会监控sentinel-dashboard服务本身：

![jj9gb6uizoehaaxsb75cr4ufnee](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/jj9gb6uizoehaaxsb75cr4ufnee.png)
> 图注：图片展示的是Sentinel控制台1.8.6的首页界面。左侧导航栏有“首页”“实时监控”“限流规则”“熔断规则”等选项。右侧上方显示“sentinel - dashboard (1/1)”，下方有“IP 实时监控”图表，显示了22:10 - 22:14时间段的QPS数据。右侧还有一张表格，记录了22:14:52 - 22:15:42时间段的QPS、拒绝QPS、响应时间等信息。该图与上文介绍Sentinel安装与集成后访问控制台的内容相关，直观呈现了控制台界面。

###### 本地运行sentinel

如果在测试时发现虚拟中的sentinel不能用可以本地运行sentinel。

将sentinel的jar包放在任意非中文、不包含特殊字符的目录下，重命名为`sentinel-dashboard.jar`：

![b7oubcamyoszmnxrlbrc23qnnuc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/b7oubcamyoszmnxrlbrc23qnnuc.png)
> 图注：图片展示了Windows文件资源管理器界面，路径为“新加卷 (D:) > lesson > sentinel”。界面中显示了一个名为“sentinel - dashboard.jar”的JAR文件，类型为JAR文件，大小为22,152 KB。该图片与文档中“本地运行sentinel”部分内容相关，对应将sentinel的jar包放在任意非中文、不包含特殊字符的目录下，重命名为`sentinel - dashboard.jar`的操作步骤，直观呈现了文件重命名后的状态。



然后运行如下命令启动控制台：

```Shell
java -Dserver.port=8090 -Dcsp.sentinel.dashboard.server=localhost:8090 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard.jar
```



访问：http://localhost:8090/ 页面



##### 1.2.2.3 项目集成Sentinel

<callout emoji="⛱️">
1. 引入依赖
2. 修改配置
3. 使用组件
</callout>

sentinel要完成熔断降级，熔断是在服务调用方即客户端，所以针对购物车服务请求商品服务实现熔断就需要在购物车服务集成 sentienl。



我们在`cart-service`模块中整合sentinel，连接`sentinel-dashboard`控制台，步骤如下：  
1）引入sentinel依赖

```XML
<!--sentinel-->
<dependency>
    <groupId>com.alibaba.cloud</groupId> 
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```



2）配置控制台

修改application.yaml文件，添加下面内容：

```YAML
spring:
  cloud:
    sentinel:
      transport:
        dashboard: 192.168.101.68:9090
        client-ip: 192.168.101.1
      http-method-specify: true # 开启请求方式前缀可根据http请求方法区分簇点链路
```



如果是在本机运行的sentinel要配置：

```Java
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8090
      http-method-specify: true # 开启请求方式前缀可根据http请求方法区分簇点链路
```



3）访问`cart-service`的任意端点

重启`cart-service`、item-service，然后访问查询购物车接口，sentinel的客户端就会将服务访问的信息提交到`sentinel-dashboard`控制台。并展示出统计信息：

![nzjbbrfe1om6ljxybzxcbp2anjd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/nzjbbrfe1om6ljxybzxcbp2anjd.png)
> 图注：图片展示的是Sentinel控制台1.8.6的首页界面，用于监控`cart-service`服务。左侧导航栏有首页、实时监控、簇点链路等选项，当前选中“簇点链路”。右侧上方显示“cart-service”及“1/1”状态，下方有“iCarts”实时监控图表，图表下方有时间、通过QPS、形串QPS、响应时间等数据列表。



点击簇点链路菜单，会看到下面的页面：

![aknobzfcboyycdxcth1cwrs0nxb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/aknobzfcboyycdxcth1cwrs0nxb.png)
> 图注：图片展示的是Sentinel控制台1.8.6中`cart-service`的簇点链路页面。左侧导航栏选中“簇点链路”。页面显示了`cart-service`的簇点链路信息，包括资源名、通过QPS、拒绝QPS、并发数、平均RT、分钟通过、分钟拒绝等数据，如`GET /carts`的资源名、平均RT等。页面右上角有“流控”“漏网”“热点”“授权”操作按钮。



所谓**簇点链路**，就是**单机调用链路**，是一次请求进入服务后经过的每一个被`Sentinel`监控的资源。默认情况下，`Sentinel`会监控`SpringMVC`的每一个`Endpoint`（接口）。

因此，我们看到`/carts`这个接口路径就是其中一个簇点，我们可以对其进行限流、熔断、降级、隔离等保护措施，稍后会详细讲解。



#### 1.2.3. 实现降级

<callout emoji="🐵">
1. 开启feign对sentinel的支持-修改配置文件
2. 编写降级类

   1. FallbackClass
   2. FallbackFactory-演示
3. 修改itemclient的注解属性
4. 启动类加扫描api的包路径
</callout>

##### 1.2.3.1 开启sentinel

下边我们先编写降级逻辑，再实现服务熔断。

AI:feign如何使用sentinel实现降级

<whiteboard token="B9lzwmtXQhnZMabdgJwcG1cgn3g"></whiteboard>

首先配置Feign使用Sentinel：

在购物车服务application.yml中配置如下：

```YAML
feign:
  sentinel:
    enabled: true # 开启feign对sentinel的支持
```

##### 1.2.3.2 实现`FallbackFactory`接口

接下来给FeignClient编写失败后的降级逻辑有两种方式：

- 方式一：FallbackClass，无法捕获到远程调用的异常

定义一个降级类**实现**FeignClient接口，并在@FeignClient注解中配置fallback 属性，如下：

```Java
@FeignClient(name="item-service",path = "/items",fallback = 降级类名.class)
```

```Java
package com.hmall.api.item;

import com.hmall.api.item.dto.ItemDTO;
import com.hmall.api.item.dto.OrderDetailDTO;
import org.springframework.stereotype.Component;

import java.util.Collection;
import java.util.List;

@Component
public class ItemFallbackClass implements ItemClient {
    @Override
    public List<ItemDTO> queryItemByIds(Collection<Long> ids) {
        System.out.println("queryItemByIds");
        return List.of();
    }

    @Override
    public void deductStock(List<OrderDetailDTO> items) {

    }
}
```

- 方式二：FallbackFactory，可以捕获远程调用的异常，我们一般选择这种方式。**也是咱们讲解的重点**

定义一个**降级类实现**FallbackFactory接口，并在@FeignClient注解中配置fallbackFactory属性

```Java
@FeignClient(name="item-service",path = "/items",fallbackFactory= 降级类名.class)
```

```Java
package com.hmall.api.item;

import com.hmall.api.item.dto.ItemDTO;
import com.hmall.api.item.dto.OrderDetailDTO;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.openfeign.FallbackFactory;
import org.springframework.stereotype.Component;

import java.util.Collection;
import java.util.List;

@Component
@Slf4j
public class ItemClientFallbackFactory implements FallbackFactory<ItemClient> {
    @Override
    public ItemClient create(Throwable cause) {
        return new ItemClient() {
            @Override
            public List<ItemDTO> queryItemByIds(Collection<Long> ids) {
                log.info("queryItemByIds,报错啦，入参={}",ids,cause);
                return List.of();
            }

            @Override
            public void deductStock(List<OrderDetailDTO> items) {

            }
        };
    }
}
```

这里我们演示方式二的失败降级处理。

**步骤一**：在hm-api模块中给`ItemClient`定义降级处理类，实现`FallbackFactory`接口：

![ipnjblt3vongomxosb0cdo5znec](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ipnjblt3vongomxosb0cdo5znec.png)
> 图注：图片展示了hm-api模块的项目结构。在src/main/java目录下，com.hmall.api.item包中，有dto和ItemClient两个文件夹，ItemClient文件夹下有一个ItemClientFallbackFactory文件。该图片与文档中“步骤一”内容相关，用于说明在hm-api模块中给ItemClient定义降级处理类，实现FallbackFactory接口的操作，展示了接口所在的具体位置。

代码如下：

```Java
package com.hmall.api.item;

import com.hmall.api.item.dto.ItemDTO;
import com.hmall.common.utils.CollUtils;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.openfeign.FallbackFactory;
import org.springframework.stereotype.Component;

import java.util.Collection;
import java.util.List;

/**
 * @author Mr.M
 * @version 1.0
 * @description itemClient的通用降级策略
 * @date 2024/8/5 9:50
 */
@Slf4j
@Component
public class ItemClientFallbackFactory implements FallbackFactory<ItemClient> {
    @Override
    public ItemClient create(Throwable cause) {
        return new ItemClient() {
            @Override
            public List<ItemDTO> queryItemByIds(Collection<Long> ids) {
                log.error("远程调用ItemClient#queryItemByIds方法出现异常，参数：{}", ids, cause);
                cause.printStackTrace();
                // 查询购物车允许失败，查询失败，返回空集合
                return CollUtils.emptyList();
            }
    
            @Override
            public void deductStock(List<OrderDetailDTO> items) {
                log.error("远程调用ItemClient#deductStock扣减库存失败,参数:{}",items,cause);
            }
        };
    }
}
```

##### 1.2.3.3 配置fallbackFactory

**步骤二**：在`hm-api`模块中的`ItemClient`接口中使用`ItemClientFallbackFactory`：

```JavaScript
package com.hmall.api.item;

import com.hmall.api.item.dto.ItemDTO;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.util.Collection;
import java.util.List;

/**
 * @author Mr.M
 * @version 1.0
 * @description 商品服务Feign接口
 * @date 2024/8/3 16:21
 */
@FeignClient(name="item-service",path = "/items",fallbackFactory = ItemClientFallbackFactory.class)
public interface ItemClient {
    @GetMapping
    List<ItemDTO> queryItemByIds(@RequestParam("ids") Collection<Long> ids);
    @PutMapping("/stock/deduct")
    public void deductStock(@RequestBody List<OrderDetailDTO> items);
}
```

##### 1.2.3.4 注入降级Bean

**步骤三**：在cart-service启动类添加扫描包配置，扫描降级类并将bean注册到spring容器

![k6skbnmgtoe5q3xeztkckhdynlt](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/k6skbnmgtoe5q3xeztkckhdynlt.png)
> 图注：这张图片展示的是CartServiceApplication类的代码内容，对应文档中注入降级Bean的步骤三，为实现降级类的扫描注册做配置。代码里的@SpringBootApplication注解配置了scanBasePackages属性，指定扫描包范围，其中被红色框标注的"com.hmall.api"包是核心配置部分，需扫描该包来加载相关的Feign客户端等组件，结合上下文可知该配置是为了注册降级相关的Bean，确保Feign能正确调用对应的降级逻辑，配合后续的降级测试环节生效。

##### 1.2.3.5 降级测试

最后测试降级效果。

重启cart-service，并将item-service停止来模拟item-service服务不可用

请求查询购物车接口，cart-service远程调用item-service，由于item-service不可用，Feign执行ItemClientFallbackFactory 中定义的降级逻辑

在ItemClientFallbackFactory 中打断点，在ItemClientFallbackFactory 中捕获到了远程调用的异常，降级方法返回空List<ItemDTO>。

![wui7b2ounonrnhxmmrkc0mdenwc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wui7b2ounonrnhxmmrkc0mdenwc.png)
> 图注：图片展示了ItemClientFallbackFactory类代码，该类实现了FallbackFactory接口。代码中定义了create方法，当远程调用失败时，返回新创建的ItemClient对象。还定义了queryItemByIds方法，当远程调用ItemClient#queryItemByIds方法出现异常时，捕获异常并打印堆栈信息，然后返回空列表。该图片与上文提到的在ItemClientFallbackFactory中捕获远程调用异常，降级方法返回空List<ItemDTO>的内容相关，直观呈现了降级逻辑实现代码。

接口测试最终效果，购物车可以正常显示，但由于获取的商品信息为空这里newPrice显示为null。

![ptrdbrdwnoxhwzxmkdpcwcusn0e](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ptrdbrdwnoxhwzxmkdpcwcusn0e.png)
> 图注：图片展示的是商品服务接口文档中查询购物车列表的接口测试结果。接口请求方式为GET，URL为/carts。响应内容以JSON格式呈现，其中newPrice字段值为null，对应图片中红色箭头所指位置。该图片与上文提到的接口测试最终效果相呼应，说明在商品服务不可用时，查询购物车接口返回的商品信息中newPrice显示为null，体现了降级逻辑的应用。



##### 1.2.3.6 小结

feign远程调用怎么实现降级？

1. 我们使用的是OpenFeign实现微服务之间的远程调用，使用Sentinel实现降级。
2. 首先在服务提供方配置sentinel，引入sentinel的依赖，配置sentinel的地址
3. 在服务调用方开启feign使用sentinel
4. 在服务调用方编写feign接口并编写降级逻辑，具体方法是编写降级类实现FallbackFactory接口，并在FeignClient注解中配置fallbackFactory。
5. 服务调用方当无法正常调用服务提供方接口时会走降级逻辑，并捕获到异常。

<whiteboard token="SO4KwydSTh2Cqpbq0JAcytN6n4g"></whiteboard>

#### 1.2.4. 服务熔断

<callout emoji="🏕️">
sentinel配置熔断策略：
1. 异常数
2. 异常比例
3. 慢调用数
</callout>

##### 1.2.4.1 配置熔断策略

根据熔断方案，sentinel会统计异常比例、慢请求比例、异常数等数据，达到阈值时断路器打开即发生熔断，熔断状态下会走降级路线保证快速响应。

下边在sentinel中配置熔断策略，找到查询商品信息的簇点链路

![wmoqbkiujozwwgxwcgpcpefnnrq](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wmoqbkiujozwwgxwcgpcpefnnrq.png)
> 图注：图片展示的是Sentinel控制台1.8.6中cart-service的簇点链路页面。页面左侧有导航栏，选中“簇点链路”。右侧显示了资源名、通过QPS、拒绝QPS、并发数、平均RT、分钟通过、分钟拒绝等信息，其中GET http://item-service/items/资源名被红框突出显示。页面右下角有“+监控”“+熔断”“+热点”“+授权”等操作按钮，其中“+熔断”按钮被红箭头指向。该图与上下文介绍的在sentinel中配置熔断策略，找到查询商品信息的簇点链路相呼应。

点击“熔断”，配置异常数，如下图：

![mq0vbddtmoulnbxskbeccqxungd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mq0vbddtmoulnbxskbeccqxungd.png)
> 图注：这张图是Sentinel服务熔断规则的配置界面，对应配置查询商品信息相关熔断策略的操作。界面的资源名对应需要保护的服务接口地址为GET:http://item-service/items/，熔断策略已选中“异常数”选项，相关配置参数设置为：异常数为1、熔断时长20秒、最小请求数2、统计时长2000毫秒，界面右下角有“新增”等操作按钮，其中绿色“新增”按钮还有红色箭头标注，整体配置内容和上下文描述的熔断参数要求完全对应。

2秒内最小请求数为2，异常数达到1发生熔断，熔断时长为20秒。

##### 1.2.4.2 测试异常数

下边进行测试，继续停止item-service商品服务。

启动cart-service购物车服务，请求查询购物车接口。

此时在购物车控制台报异常信息如下：

```Java
feign.FeignException$ServiceUnavailable: [503] during [GET] to [http://item-service/items?ids=100000006163] [ItemClient#queryItemByIds(Collection)]: [Load balancer does not contain an instance for the service item-service]
```

当异常数达到Sentinel配置将发生熔断，熔断时间20秒，此时cart-service控制台将不再输出异常信息，因为cart-service走了降级路线不再请求item-service。

通过观察cart-service控制台日志可以发现，发生熔断后sentinel客户端抛出了com.alibaba.csp.sentinel.slots.block.degrade.DegradeException.



熔断时间（20秒）过后断路器成半开状态，再次请求购物车接口， 会尝试请求item-service一次，如果成功此时断路器关闭，如果仍失败断路器打开，关闭item-service日志发现输出一次java.lang.RuntimeException: 测试异常，此时断路器打开继续熔断。

##### 1.2.4.3 测试慢调用比例

下边测试另一种熔断策略：慢调用比例

删除原来的熔断策略

![ruxgbf79hoski7xj6vgc3molnun](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ruxgbf79hoski7xj6vgc3molnun.png)
> 图注：图片展示的是Sentinel控制台中cart - service的熔断规则页面。页面左侧有导航栏，选中“熔断规则”。右侧显示资源名、熔断策略、阈值、熔断时长等信息，其中熔断策略为异常数，阈值为1，熔断时长20s。关键信息是右侧操作栏中“删除”按钮被红色箭头突出显示，表明可删除当前熔断规则。该图片与上文“删除原来的熔断策略”内容对应，直观呈现了操作位置。

添加新的熔断策略。

![dadpbwfgaot52ux0ogscms17ncg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dadpbwfgaot52ux0ogscms17ncg.png)
> 图注：图片展示的是Sentinel新增熔断规则界面。界面中“资源名”为GET:http://item-service/items/；“熔断策略”选择“慢调用比例”；“最大RT”为200；“比例阈值”为0.5；“熔断时长”为20s；“最小请求数”为2；“统计时长”为3000ms。该界面与文档中测试慢调用比例的内容相关，用于配置按照慢调用比例来触发熔断的规则，如RT超过200毫秒的请求调用为慢调用，统计最近3000ms内最少2次请求，若慢调用比例大于等于50%则触发熔断，熔断持续时长20s。

这种是按照慢调用比例来做熔断，上述配置的含义是：

- RT(接口响应时间)超过200毫秒的请求调用就是慢调用
- 统计最近3000ms内的最少2次请求，如果慢调用比例大于等于50%，则触发熔断
- 熔断持续时长20s



修改item-service的查询商品信息接口，添加线程休眠模拟处理时间，这样可以制造慢请求的效果。

![sa0jbosaloyqvtxpuejc4jyxnfg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/sa0jbosaloyqvtxpuejc4jyxnfg.png)
> 图注：图片展示了item-service中ItemController.java文件的代码。在queryItemByIds方法中，有红色框标注的代码块，内容为“休眠1秒”，通过try - catch语句实现，具体为Thread.sleep(1000)。该图片与上文提到的测试慢调用比例相关，用于模拟处理时间，制造慢请求效果，为后续测试熔断做准备。

重启item-service服务

连续请求查询购物车接口，cart-service连续向item-service发生请求，由于每次请求item-service线程休眠1秒导致接口请求时长肯定大于200毫秒，最终发生了熔断。

参考测试“异常数”熔断策略去观察cart-service和item-service的日志来判断熔断发生的情况。

##### 1.2.4.4 测试异常比例

请大家自行测试熔断策略： 异常比例

举例：

![noexbyo6qooxunxwfkyc8mxenrc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/noexbyo6qooxunxwfkyc8mxenrc.png)
> 图注：图片展示的是Sentinel新增熔断规则界面。界面中“资源名”为GET: http://item-service/items/；“熔断策略”选择“异常比例”；“比例阈值”为0.5；“熔断时长”为20s；“最小请求数”为2；“统计时长”为3000ms。右下角有“新增并继续添加”“新增”“取消”按钮。该图片与文档中测试异常比例的内容相关，用于说明如何在Sentinel中设置异常比例熔断规则。

- 统计最近3000ms内的最少2次请求，如果异常比例不低于0.5，则触发熔断
- 熔断持续时长20s



##### 1.2.4.5 面试题

你的项目中熔断降级怎么实现的？





### 1.3.请求限流

<callout emoji="❤️">
配置限流策略
qps：每秒请求数
并发线程数：最多多少个线程发起请求-线程隔离时再讲
</callout>

#### 1.3.1 配置限流策略

下边我们对item-service的商品查询接口进行限流，找到商品查询的簇点链路，点击"流控"

![hhzjbzyduo7m1bxtiqgcapegnbf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/hhzjbzyduo7m1bxtiqgcapegnbf.png)
> 图注：这张图片展示的是Sentinel的簇点链路界面，用于配置微服务的流控策略，对应文档中配置item-service商品查询接口限流的操作场景。界面中定位到了路径为GET http://item-service/items/的商品查询接口条目，该条目操作栏有多个功能按钮，其中被红色箭头和红色框重点标注的是“流控”按钮，此按钮正是文档中提及的，需要点击以对该接口配置限流策略的操作入口，点击它后即可按要求设置单机QPS阈值。

在弹出的菜单中填写QPS单机阈值为5，表示每秒最多处理5个请求

![ypycbtg2qo27enxzo5ncujppngb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ypycbtg2qo27enxzo5ncujppngb.png)
> 图注：图片展示的是对item-service商品查询接口进行限流时的“编辑流控规则”界面。界面中“资源名”显示GET: http://item-service/items/，“针对来源”为default，“阈值类型”选中QPS，单机阈值为5，“是否集群”未选中。该界面与上文提到的对item-service商品查询接口进行限流，点击“流控”后在弹出菜单中填写QPS单机阈值为5的操作相关，直观呈现了限流配置的具体设置情况。



AI：什么是QPS

QPS 是 "Queries Per Second" 的缩写，中文通常称为“每秒查询率”或“**每秒请求数**”。它是一个衡量系统性能的重要指标，特别是在评估 Web 服务器、数据库系统、API 接口等的负载能力时经常使用。

QPS 表示系统在一秒钟内能够处理的查询或请求的数量。这个指标可以帮助开发者和运维人员了解系统的处理能力和性能瓶颈。例如，如果一个 Web 服务的 QPS 为 100，则意味着该服务在一秒钟内最多可以处理 100 个请求。

TPS:

tps：100/s   80/s



qps=5

压测 =10

#### 1.3.2 限流测试

首先去掉item-service服务中查询商品信息接口的休眠代码。

![eu4ub6fbeotzwvxxspycd8jln6b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/eu4ub6fbeotzwvxxspycd8jln6b.png)
> 图注：图片展示的是ItemController.java代码片段，重点突出了一段代码。代码中定义了一个名为queryItemByIds的方法，接收一个Long类型的ids参数。在方法内部，有一段被注释掉的代码，其中包含Thread.sleep(1000)语句，用于休眠1000毫秒，即1秒。该图片与上下文关系紧密，上下文提到在进行限流测试时，需设置模拟线程数等参数，此图片中的代码示例可能是在说明限流测试中对请求进行延时处理的代码实现方式。

下边我们用jmeter压力测试工具进行压力测试。

JMeter 是一个广泛使用的开源性能测试工具，主要用于测试 Web 应用程序的负载和性能。它是由 Apache Software Foundation 开发和维护的，完全用 Java 编写，因此可以在任何支持 Java 的平台上运行。

找到课程资料中的JMeter安装包apache-jmeter-5.4.1.zip，解压到不包括中文的目录，进入bin下双击打开jmeter.bat

![oiavbqxmiovngkxzswrceilkntb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/oiavbqxmiovngkxzswrceilkntb.png)
> 图注：图片展示了JMeter安装包解压后的文件夹内容。其中，jmeter.bat文件被红色箭头指向，突出显示。该图片与上文“找到课程资料中的JMeter安装包apache-jmeter-5.4.1.zip，解压到不包括中文的目录，进入bin下双击打开jmeter.bat”的操作步骤相关，直观呈现了在解压后的文件夹中找到并双击打开jmeter.bat的操作位置，是后续进行限流测试等操作的前提步骤。

打开软件切换到中文

![pexqbabivoysv2xfwcjccxkcnlb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/pexqbabivoysv2xfwcjccxkcnlb.png)
> 图注：图片展示了JMeter软件界面，其中“Options”菜单被选中，弹出“Choose Language”下拉菜单，显示多种语言选项，如English、Chinese (Simplified)等。画面中用红色箭头指向“Chinese (Simplified)”选项。该图片与文档中“打开软件切换到中文”内容相关，用于指导用户在JMeter软件中选择中文界面，是后续进行雪崩测试等操作的基础步骤。

接下来找到课程资料下的“雪崩测试.jmx” 拖到JMeter面板中



![h5inbhf3podscxxy93hcp4tlnrq](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/h5inbhf3podscxxy93hcp4tlnrq.png)
> 图注：这张图片展示了Apache JMeter（5.4.1版本）的操作界面，左侧为文件目录面板，其中红色箭头指向的“雪崩测试.jmx”文件被明确标注“将测试用例拖到这里”，该箭头指向JMeter界面右侧Test Plan区域，对应文档中“找到课程资料下的‘雪崩测试.jmx’拖到JMeter面板中”的操作步骤，呈现了将测试文件导入JMeter的关键操作指引。

最终效果如下

![cw7jbmvbkoyetjxsfugcysznnrc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/cw7jbmvbkoyetjxsfugcysznnrc.png)
> 图注：图片展示了JMeter软件界面，左侧为测试计划结构，包含“限流测试”等节点。右侧是“限流测试”配置界面，线程组名称为“限流测试”，线程数设置为1000，Ramp-Up时间（秒）为100，循环次数为1。界面中还显示了HTTP请求、HTTP信息头管理器、察看结果树、汇总报告等测试元素。该图片与文档中限流测试操作步骤相关，直观呈现了设置参数后的JMeter界面状态。

设置模拟线程数等参数：

![dngvbm8evozrkbxambkcjjg9nag](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dngvbm8evozrkbxambkcjjg9nag.png)
> 图注：图片展示了JMeter中“雪崩测试.jmx”脚本的参数设置界面。左侧有“线程数”“Ramp-Up时间（秒）”“循环次数”等选项，右侧对应显示数值。其中“线程数”为1000，“Ramp-Up时间（秒）”为100，“循环次数”被勾选为“永远”。该图片与上文设置模拟线程数等参数的内容相关，直观呈现了设置的具体数值，帮助用户了解如何配置雪崩测试的参数。

表示：总共发送1000个请求，用100秒发送，循环次数是1即执行10秒结束。

根据设置参数可知，每秒发送10个请求，在100秒可完成1000个请求。

在“HTTP请求”界面设置压力测试请求的地址

![vwt7bdgbeot7vfx8hy4cipxkndg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vwt7bdgbeot7vfx8hy4cipxkndg.png)
> 图注：这张图片是Apache JMeter工具的操作界面，左侧导航栏中选中了“服务保护”分类下的“限流测试”模块里的“HTTP请求”选项；界面右侧的“HTTP请求”设置区域被红框标注了两处关键内容，一处是Web服务器配置，其中协议为http，服务器名称或IP为localhost，端口号为8082，另一处是HTTP请求设置，请求方法为GET，路径为/cats。这处设置对应了文档中“在‘HTTP请求’界面设置压力测试请求的地址”的操作内容，是限流测试中压力请求的配置界面。

右键“限流测试”点击“启动”

![dntpbvfkgorquxxm9fact0gfnbc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dntpbvfkgorquxxm9fact0gfnbc.png)
> 图注：这张图片展示的是Apache JMeter工具的操作界面，界面左侧的「雪崩测试」目录下包含「限流测试」选项，右键点击该选项后弹出菜单，其中「启动」选项为蓝色高亮选中状态，且有红色箭头指向该选项，对应文档中右键“限流测试”点击“启动”的操作步骤，界面同时呈现了该工具的文件路径、菜单栏及其他功能选项，与限流测试的压力测试启动操作相关。



测试结果可以在“汇总报告”中查看，每次请求的次数在“察看结果树”中查看。

从吞吐量可以看出每秒发送10个请求，异常数是0

![uwo8boaegowmoyx4a4pc4rnkncg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/uwo8boaegowmoyx4a4pc4rnkncg.png)
> 图注：图片展示的是“HTTP请求”测试的“汇总报告”界面。界面中“HTTP请求”标签下，#样本为87，平均值6，最小值3，最大值76，标准偏差10.74，异常%0.00%，香吐量10.3/sec，接收KB/sec 5.27，发送KB/sec 1.38，平均字节数526.1。该图片与上文提到的在“HTTP请求”界面设置压力测试请求地址，右键“限流测试”点击“启动”，测试结果可在“汇总报告”中查看的内容相关，直观呈现了测试结果。



通过sentinel的实时监控界面可以看出对商品查询接口的请求，通过的QPS为5，拒绝QPS为5。

![dvejbeyfwoxeqvx6k4jcpmm3n8y](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dvejbeyfwoxeqvx6k4jcpmm3n8y.png)
> 图注：图片展示的是Sentinel控制台1.8.6的实时监控界面，监控对象为cart - service。界面左侧有导航栏，当前选中“实时监控”。右侧图表显示了GET http://item - service/items/请求的QPS情况，下方表格记录了多个时间点的QPS数据，其中“通过QPS”和“拒绝QPS”均为5.0，响应时间为2.0毫秒。该图与上下文关系紧密，直观呈现了上文提到的对商品查询接口限流QPS为5后，每秒10个请求查询/carts时，远程调用商品查询服务接口的QPS情况。

因为上边我们配置了对商品查询接口限流QPS为5，所以每秒10个请求查询/carts，远程调用商品查询服务接口的QPS只有5，说明被限流了。

注意：如果实时监控图无法正常显示需要进入虚拟机进行时间同步。

```Java
# 安装 ntpdate（如果未安装）
yum install ntpdate -y

# 手动同步时间（以阿里云 NTP 服务器为例）
ntpdate ntp.aliyun.com

或
ntpdate pool.ntp.org
```

再查看每次的请求和响应数据发现，由于编写了商品查询接口远程调用的降级逻辑，当商品查询接口被限流后将会走降级方法，从jmeter测试结果可以看到被限流的请求无法获取商品信息

![wymkbxxu0oy4mtxpuwgcw5xlnzk](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wymkbxxu0oy4mtxpuwgcw5xlnzk.png)
> 图注：这张图片展示了Jmeter的查看结果树界面，左侧列有多个HTTP请求，当前选中的是一条HTTP请求，响应区域的JSON数据中，“twoPrice”字段被红色箭头高亮指向。结合上下文可知，该请求是商品查询接口的测试请求，接口被限流后走了降级逻辑，对应的响应未返回正常的商品价格数据，正是无法获取商品信息的相关测试结果呈现，同时该接口被限流后会抛出FlowException异常的情况也与此测试场景相关。

再查看cart-service的控制台，在降级方法中捕获到了 com.alibaba.csp.sentinel.slots.block.flow.FlowException 异常，这说明接口被限流后sentinel客户端会抛出FlowException 异常。





#### 1.3.3 小节

sentinel限流怎么实现？

我们项目使用Sentinel实现限流控制。

1. 首先在服务提供方配置sentinel，引入sentinel的依赖，配置sentinel的地址
2. 通过Sentinel控制台配置限流策略，可以配置QPS阈值、并发线程数等。
3. 当方法被限流会走降级逻辑。



### 1.4. sentinel降级

#### 1.4.1 问题描述

下边我们对/carts接口进行限流测试，找到GET/carts簇点

![kj9mbhcocoztqwx6gswcuuglnve](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/kj9mbhcocoztqwx6gswcuuglnve.png)
> 图注：图片展示的是Sentinel控制台1.8.6中cart-service的实时监控界面。左侧导航栏选中“实时监控”，右侧显示簇点链路为192.168.101.1:8720，下方有资源名、通过QPS、拒绝QPS、并发数、平均RT、分钟通过、分钟拒绝等信息。关键信息是GET/carts簇点的资源名、通过QPS、拒绝QPS、并发数、平均RT、分钟通过、分钟拒绝均为0，右侧有“+流控”“+熔断”“+热点”“+授权”操作按钮，其中“+流控”按钮被红色框突出显示。该图与上下文介绍的对/carts接口进行限流测试，以及访问被限流接口后显示“Blocked by Sentinel(flow limiting)”的内容相关。

点击“流控” 设置QPS 单机阈值为6

![mya4b35yeodadyxewrxcgxawnpc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mya4b35yeodadyxewrxcgxawnpc.png)
> 图注：这张图片展示的是Sentinel新增流控规则的配置界面，核心是对微服务接口GET:/carts设置流控规则，用于实现接口限流。界面中，资源名被填为GET:/carts，针对来源为default，阈值类型选定QPS，单机阈值设置为6，和文档中描述的“设置QPS单机阈值为6”完全对应，该配置界面是进行GET:/carts接口限流测试的操作环节，用于实现对该接口的流量管控。

用jmeter测试，通过sentinel进行实时监控，通过QPS为6，拒绝QPS为4，符合我们的预期结果

![e6mjbgktfolrdvx2gqecgntxnyb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/e6mjbgktfolrdvx2gqecgntxnyb.png)
> 图注：图片展示的是cart - service的实时监控界面。左侧导航栏有“首页”“实时监控”“限流规则”等选项。右侧上方显示“cart - service (1/2)~”，下方有“IF实时监控”“热点规则”“熔断规则”等选项。监控图表中，绿色折线代表通过QPS，蓝色折线代表拒绝QPS。图表下方表格记录了通过QPS和拒绝QPS的详细数据，如18:14:11的通过QPS为6.0，拒绝QPS为4.0等。该图与上下文介绍的对/carts接口进行限流测试，通过sentinel进行实时监控的内容相关。

当/carts接口被限流时我们访问此接口

![rkt1bzocvogs6hx8iric5d9onnf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/rkt1bzocvogs6hx8iric5d9onnf.png)
> 图注：图片展示的是用jmeter测试时的界面。方法为GET，请求地址为/carts，请求参数选择x-www-form-urlencoded。下方显示响应内容，内容为“Blocked by Sentinel (flow limiting)”。该图片与文档中对/carts接口进行限流测试的内容相关，用于直观呈现当/carts接口被限流时，用jmeter测试的实际响应结果，与文档中“当/carts接口被限流时我们访问此接口”的描述相呼应。

结果内容：Blocked by Sentinel (flow limiting)，从字面内容看是被sentinel限流。

为什么访问item-service的商品查询接口走了降级方法，而访问/carts没有走咱们之前配置的降级方法呢？

#### 1.4.2 sentinel实现降级

这里我们需要重新梳理下：

前边cart-service远程调用item-service服务的商品查询接口正常走降级方法，这是因为我们编写了ItemClient接口的降级类ItemClientFallbackFactory，并且我们在cart-service服务中集成了sentinel，在cart-service的application.yml配置文件中配置了feign使用sentinel，如下：

```JavaScript
feign:
  sentinel:
    enabled: true # 开启feign对sentinel的支持
```

也就说，在cart-service通过openfeign远程调用item-service时通过sentinel进行降级，最终执行ItemClientFallbackFactory类中指定的降级方法。

而这里我们是对/carts进行限流，并没有针对此接口编写降级方法。

AI：sentinel实现降级



针对feign远程调用我们通过实现FallbackFactory接口去编写远程调用接口对应的降级逻辑，而针对非feign远程调用的降级逻辑我们需要使用**@SentinelResource**注解去实现

<callout emoji="💡">
1.实现对自身服务的保护
@SentinelResource
2.书写两个降级方法
fallback =降级方法名
blockHandler =降级方法名
</callout>

首先使用 @SentinelResource 定义资源，编写降级方法

```JavaScript
@ApiOperation("查询购物车列表")
@SentinelResource(value = "queryMyCarts", fallback = "queryMyCartsFallback", blockHandler = "queryMyCartsBlockHandler")
@GetMapping
public List<CartVO> queryMyCarts(){

    return cartService.queryMyCarts();
}
//当发生非限流非熔断异常走此方法
public List<CartVO>  queryMyCartsFallback(Throwable throwable){
    log.error("非限流、非熔断异常执行的降级方法，throwable:",  throwable);
    return new ArrayList<>();
}
//当发生熔断、限流走此方法
public List<CartVO>  queryMyCartsBlockHandler( BlockException blockException){
    log.error("触发限流、熔断时执行的降级方法，blockException:",  blockException);
    return new ArrayList<>();
}
```

然后在sentinel中设置资源的流控

![yydibxfofor54dxja8wcsgxoncd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/yydibxfofor54dxja8wcsgxoncd.png)
> 图注：这张图片展示了名为cart-service的服务的Sentinel热点规则配置页面，页面内列出了多个资源项，其中“queryMyCarts”资源被红色框线标注，并有红色箭头指向该资源项右侧的“流控”按钮。页面还显示了其他资源，如sentinel_spring_web_context、GET:/carts等，各资源项后有流控、隔离、热点、授权等操作选项，界面可设置关键字搜索资源，当前查询条件为指定的IP地址，该页面用于配置对应服务资源的相关防护规则，与文档中介绍的Sentinel降级及流控规则设置内容相关。

![dtuub5tmsohiwjxssnpcedrznuf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dtuub5tmsohiwjxssnpcedrznuf.png)
> 图注：这是Sentinel编辑流控规则的界面，用于配置针对指定资源的限流规则。界面中资源名为“queryMyCarts”，针对来源设置为默认值“default”，阈值类型选中“QPS”，单机阈值设置为6，未勾选是否集群的选项，界面底部还有“高级选项”入口，以及用于保存或取消操作的按钮。这张图对应的是文档中设置sentinel流控规则的环节，用于为资源配置QPS为6的限流阈值。

删除针对/carts设置的流控规则

![udptbnc2tob1pjx3mvvc2412nxe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/udptbnc2tob1pjx3mvvc2412nxe.png)
> 图注：这是Sentinel的流控规则管理界面，左侧导航栏中“流控规则”选项被红色框标注，当前展示的流控规则列表里，GET/carts对应的流控规则条目被红色框圈出，操作列的“删除”按钮也被红色箭头指向，清晰呈现了删除针对/carts设置的流控规则这一操作对应的界面内容，该操作是为后续验证Sentinel降级逻辑所做的准备步骤。

再次使用jmeter进行压力测试

观察cart-service控制台，当/carts限流后正常执行降级方法，日志如下：

```JavaScript
18:37:31:767 ERROR 4696 --- [nio-8082-exec-9] c.hmall.cart.controller.CartController   : 触发限流、熔断时执行的降级方法，blockException:
```

在降级方法中可以返回特殊的信息，或指定特殊的状态码，根据特殊值前端可展示为类似“网络忙请稍后重试” 这样的信息。



#### 1.4.3 小节

##### 两个注解的核心区别

<sheet sheet-id="dEv0iv" token="YSLTsSJDuh5Wy1tmAyOc6Odunpe"></sheet>

sentienl降级怎么实现？

1. 对于Feign远程调用我们对每个远程调用接口实现降级方法，通过实现FallbackFactory接口去实现降级方法。
2. 对于非Feign远程调用我们使用@SentinelResource注解编写降级方法。





### 1.5. 线程隔离-作业

#### 1.5.1 方案介绍

首先我们来看下线程隔离功能。

线程隔离有两种方式实现：

- **线程池隔离**：给每个服务调用业务分配一个线程池，利用线程池本身实现隔离效果
- **信号量隔离**：不创建线程池，而是计数器模式，记录业务使用的线程数量，达到信号量上限时，禁止新的请求

如图：

![v4mhbj1fiodgsixlu9gczlwyn2f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/v4mhbj1fiodgsixlu9gczlwyn2f.jpg)
> 图注：这张图展示了微服务线程隔离的两种实现方式：线程池隔离与信号量隔离的流程。用户请求调用服务D后，调用服务A、服务B采用线程池隔离的方式，分别对应拥有10线程、5线程的专属线程池，每个线程池内的线程各自通过对应服务的FeignClient访问目标服务。而调用服务C则采用信号量隔离的方式，设置了计数上限为10的信号量，通过计数来限制可访问服务C的线程数量。这张图直观对应了方案介绍里的内容，清晰呈现了两种线程隔离模式的运行逻辑。





Sentinel的线程隔离就是基于信号量隔离实现的。



#### 1.5.2 Sentinel线程隔离

下边我们使用Sentinel实现基于信号量的线程隔离。

点击查询商品的FeignClient对应的簇点资源后面的流控按钮：

![zvajbc60woack1xhpytcvcftnvf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/zvajbc60woack1xhpytcvcftnvf.png)
> 图注：图片展示的是点击查询商品的FeignClient对应的簇点资源后弹出的表单界面。表单中列出了资源名、通过QPS、拒绝QPS、并发数、平均RT、分钟通过、分钟拒绝等信息，其中“GET http://item-service/items/”资源名被红色框突出显示。该图片与上文提到的点击查询商品的FeignClient对应的簇点资源后面的流控按钮后，在弹出的表单中填写内容的操作相关，展示了填写内容时所对应的资源信息界面。

在弹出的表单中填写下面内容：

![dihnbyfvtonx9mx9sgfctco0n1g](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dihnbyfvtonx9mx9sgfctco0n1g.png)
> 图注：图片展示的是新增流控规则的表单界面。表单中“资源名”为GET.http://item-service/items，“针对来源”为default，“阈值类型”选中并发线程数，“单机阈值”为5，“是否集群”未勾选。下方有“高级选项”链接，以及“新增并继续添加”“新增”“取消”三个按钮。该表单用于设置商品查询接口的并发线程数限制，与上下文介绍的点击查询商品FeignClient对应的簇点资源后设置流控规则的内容相呼应。

注意，这里勾选的是并发线程数限制，也就是说这个查询功能最多使用5个线程，而不是5QPS。如果查询商品的接口每秒处理2个请求，则5个线程的实际QPS在10左右，而超出的请求自然会被拒绝。

cart-service调用商品查询控制在5个线程内，通过线程隔离即使商品服务出现问题也不会影响cart-service服务。

下边修改商品查询接口的代码，添加休眠500毫秒的代码，模拟一次请求需要500毫秒，一秒则可处理2次请求，5个线程可接收10 QPS左右。





![ojtdbp4ztodyplxwjzqc8l6vnre](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ojtdbp4ztodyplxwjzqc8l6vnre.png)
> 图注：这是微服务线程隔离作业的相关代码配置截图，展示了ItemController.java文件中的商品查询接口代码。该接口为queryItemByIds，添加了休眠500毫秒的代码用于模拟商品查询请求所需的处理时长，相关代码被红色方框标注突出，对应文档中要求修改商品查询接口、添加休眠代码以模拟请求耗时的任务内容。

将/carts的限流规则调大以免影响商品查询接口的限流控制。

![tej8bjbvroya3yxtkvgcihw3nzg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/tej8bjbvroya3yxtkvgcihw3nzg.png)
> 图注：图片展示的是“编辑流控规则”界面。界面中有“资源名”“针对来源”“阈值类型”“单机阈值”“是否集群”等设置项，其中“资源名”为“queryMyCarts”，“针对来源”为“default”，“阈值类型”选中“QPS”，“单机阈值”为“100”，“是否集群”未选中。该图片与上下文内容相关，上下文提到将/carts的限流规则调大以免影响商品查询接口的限流控制，此图可能是配置限流规则时的界面展示。

最终如下图配置

![lbsqbpbxnondghxixnmclivwnue](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/lbsqbpbxnondghxixnmclivwnue.png)
> 图注：图片展示的是Sentinel控制台1.8.6中cart - service的限流规则配置界面。界面显示了资源名、来源应用、流控模式、降级模式、阈值、降级模式、流控策略等信息。其中，GET /http : //item - service/items和queryMyCarts两个资源的流控策略均为快速失败，阈值分别为5和100。该图片与上文提到的将/carts的限流规则调大以免影响商品查询接口的限流控制，以及使用jmeter进行测试等内容相关，直观呈现了限流规则配置情况。



下边使用jmeter进行测试，更改线程数为10000，时间为100秒，每秒需要发送100个请求。

![xjdlbha80ouahrxxhvwcu8cenwg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xjdlbha80ouahrxxhvwcu8cenwg.png)
> 图注：这张图片展示的是Jmeter工具中的线程组配置界面，是商品查询接口限流测试相关的配置内容。界面左侧是服务保护相关的测试目录，当前选中“限流测试”；右侧线程组配置区域中，名称为“限流测试”，核心的线程数参数被红色箭头突出标注，其数值设置为10000，同时还配置了Ramp-Up时间为100秒，循环次数为1，开启了每次迭代使用同一用户的选项。该配置与文档中用Jmeter进行限流测试的内容相呼应，符合设定测试参数的操作要求。

启用测试，通过sentinel实时监控，符合我们的预期。

![pocnbkldsoweyhxsu26cktkynkd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/pocnbkldsoweyhxsu26cktkynkd.png)
> 图注：图片展示了使用Sentinel进行测试的监控数据。上方图表显示GET请求的QPS（每秒请求数）变化，峰值接近100。右侧表格记录了不同时间点的QPS数据，如19:58:01的通过QPS为10.0，拒绝QPS为90.0等。下方图表为queryMyCarts请求的QPS变化，峰值同样接近100。右侧表格记录了其QPS数据，如19:58:01的通过QPS为100.0，拒绝QPS为0.0等。该图与上下文测试使用Sentinel监控，符合预期的内容相关，直观呈现了测试结果。



### 1.6 面试题

你们是怎么做微服务保护的？

## 2 分布式事务

### 2.1. 什么是分布式事务

#### 2.1.1 概念

首先我们看看项目中的下单业务整体流程：

单体服务&微服务

<whiteboard token="ABUSwaC8whYwJwbNIKtcjUUTnBf"></whiteboard>





交互流程如下：

用户创建订单，客户端请求交易服务创建订单

创建订单成功，交易服务请求购物车服务清理购物车，请求库存服务扣减库存

由于订单、购物车、商品分别是三个不同的微服务，而每个微服务都有自己独立的数据库，**一次业务需求**需要订单、购物车、商品服务分别执行自己的本地事务，是跨多个数据库完成这次下单的事务，**像这种，在分布式系统环境下由多个服务通过网络通信协作去完成一次事务，这称之为分布式事务。可简单理解为一个分布式事务等于多个本地事务。**



#### 2.1.2 测试分布式事务

> **测试下单接口：接口里、删除购物车和扣减库存成功，但是生成订单失败。**

分布式事务是无法通过单个数据库事务去控制的，每个微服务都有自己的数据库，一次下单事务需要订单、购物车、商品服务分别执行自己的本地事务，其中一个执行失败其它本地事务是无法回滚的，比如：扣减库存失败无法回滚清理购物车及创建订单的事务。

下边我们测试下单扣减库存，首先部署交易微服务，交易微服务**是第一天布置的作业**，这里有几点需要注意：

<callout emoji="🎨">
1. 需要在ItemClient接口中增加扣减库存的方法，以供交易服务远程调用。
2. 商品服务中的OrderDetailDTO移到了hm-api工程，凡是引用该类的微服务都需要统一引用hm-api下的OrderDetailDTO.
3. 交易服务启动类注意添加扫描hm-api下的feign接口
4. 下单方法代码的最后位置制造异常
5. 预期结果是：当扣减库存成功，下单失败，最终扣减库存事务进行回滚(未生成交易单)。
</callout>

```JavaScript
@MapperScan("com.hmall.trade.mapper")
@EnableFeignClients(basePackages = {"com.hmall.api"})
@SpringBootApplication(scanBasePackages = {"com.hmall.trade","com.hmall.api"})
```

4.我们在下单方法代码的最后位置制造异常，如下：

```JavaScript
@Override
    @Transactional
    public Long createOrder(OrderFormDTO orderFormDTO) {
        ....
        if(1==1){
            throw new RuntimeException("测试异常");
        }
        return order.getId();
    }
```

预期结果是：当扣减库存成功，下单失败，最终扣减库存事务进行回滚。

启动三个服务

![im3ybqq2bo0cpixw2nrcog3zngl](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/im3ybqq2bo0cpixw2nrcog3zngl.png)
> 图注：图片展示了启动的三个服务，分别是ItemServiceApplication:8081、CartServiceApplication:8082、TradeApplication:8085，它们均以绿色标识。该图片与文档中“启动三个服务”内容相关，对应文档中“我们在下单方法代码的最后位置制造异常，启动三个服务，下边打开交易服务的swagger文档找到下单接口进行测试”的操作步骤，直观呈现了服务启动情况，为后续测试做准备。

下边打开交易服务的swagger文档找到下单接口进行测试：

从商品数据库找一个商品id填入请求参数

![mj6ibb6kqok5ggxtxwscmok6nkg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mj6ibb6kqok5ggxtxwscmok6nkg.png)
> 图注：图片展示的是交易服务下单接口的测试界面。上方有“POST”和“/orders”字样，下方有“请求参数”选项卡，选中后显示了JSON格式的请求参数，包括addressId、details（包含itemId和num）、paymentType等信息。下方“响应内容”区域显示了响应数据，包含timestamp、status、error等字段，其中status为500，error为“Internal Server Error”，path为“/orders”。该图片与文档中测试分布式事务的内容相关，用于展示下单接口测试时的请求与响应情况。



最终抛出异常，查看商品的库存正常扣除，但是订单数据没有创建成功，最终导致数据不一致。

测试结论：

**通过本地事务控制注解 @Transactional是无法控制分布式事务的。**



#### 2.1.3 分布式事务的不同类型

下边为了简化分析过程 我们仍然以下单扣减库存为例说明：

在单体架构下实现下单减库存，如下图：

<whiteboard token="WemswtlLEh8OxJboLSFcLTkjnfh"></whiteboard>

用户请求订单服务，订单服务请求数据库完成创建订单扣减库存，通过本地事务实现，代码如下：

```Java
1.begin transaction； 
//1.本地数据库操作：创建订单
//2.本地数据库操作：减去库存 
//3.可能出现异常
2.commit transation/rollback transation;
```

如果是在微服务架构下，如下图：

用户请求订单服务下单，订单服务请求库存服务扣减库存。

<whiteboard token="SlGnwkNXDhj2onbN8wAcMtxXnCc"></whiteboard>

此时代码变为下边这样：

```Java
1.begin transaction； 
//1.本地数据库操作：创建订单
//2.远程调用：减去库存 
//3.可能出现异常,只能回滚本地的数据库操作，但是库存被扣减过了
2.commit transation/rollback transation;
```

设想: 当远程调用扣减库存成功了，由于网络问题远程调用并没有返回，此时本地事务提交失败就回滚了创建订单的操作，此时订单没有创建成功而库存却扣减了，最终就导致了下单扣减库存整个事务的数据不一致。

因此在分布式架构下，基于数据库的事务控制无法满足要求，下单操作是一次本地事务，扣减库存是一次本地事务，两次本地事务组成一个完整的事务即下单扣减库存，数据库的本地事务只能控制一次本地事务即下单操作控制下单的本地事务，扣减库存操作控制扣减库存的本地事务，无法保证下单和扣减库存整体事务的原子性和一致性。

造成分布式事务无法控制的根本原因是不同业务的数据通常不在一个数据库中或者不在一个系统中，一次事务需要由多个服务或多个系统远程调用协作去完成，远程协作依赖网络，由于网络问题会导致整体事务不能正常完成。



**分布式事务的典型场景是**：**业务的数据分布在多个数据库，一次事务操作需要跨多个数据库去完成**，

需要由多个服务远程调用协作去完成，远程调用依赖网络，由于网络问题会导致整体事务不能正常完成。

如下图所示：

<whiteboard token="CMWxwZFz5hMALZb3ehgc8hFQnrg"></whiteboard>

还有非典型的分布式事务场景也需要了解下。

1）单服务请求多数据库完成一次事务

下图中虽然没有跨服务远程调用但一次事务请求两个不同的数据库也属于分布式事务的场景，创建订单会和订单数据库创建连接通过一次本地事务提交数据，减库存会和商品数据库创建连接通过一次本地事务提交数据，因为下单扣减库存是通过两个数据库连接完成，仍然是多次本地事务共同完成一个完整的事务。

<whiteboard token="MuCDwvXuYhr2h8bRsH0cirn0n5c"></whiteboard>



3）多服务请求单数据库完成一次事务

下图中虽然用的一个数据库但是通过跨服务远程调用去完成一次事务，也属于分布式事务的场景。

思考下这种场景为什么也属于分布式事务？

<whiteboard token="CLL9w1CuphRDFtbRWpicTFSXnMd"></whiteboard>



#### 2.1.4 小结

什么是本地事务？

基于应用自己的关系型数据库的事务称为本地事务，在service方法通过添加@Transactional注解进行本地事务控制。

什么是分布式事务？

在分布式系统环境下由多个服务通过网络通信协作去完成一次事务，这称之为分布式事务。

分布式事务的场景有哪些？

多个微服务之间通过远程调用完成一次分布式事务，即：跨服务完成一次事务

单服务请求多数据库完成一次事务，即：跨数据源完成一次事务

多服务请求单数据库完成一次事务，即：跨服务完成一次事务





### 2.2. CAP原理

遇到了分布式事务的场景我们该如何去进行**事务控制**呢，本节学习如何选型分布式事务的**控制方案**。

**什么是CAP原理**

首先需要理解什么是CAP原理，明白了CAP原理有助于我们去**选型**分布式事务的控制方案。

CAP是 Consistency、Availability、Partition tolerance三个词语的缩写，分别表示一致性、可用性、分区容忍性。

使用下图去理解CAP：

下图表示客户端经过网关访问订单服务，库存服务

<whiteboard token="IIfXw744PhzUCRbn6cwcMXWQnYd"></whiteboard>



**一致性:** 

向系统写一个新数据再次读取到的也一定是这个新数据。拿上图举例，请求订单服务下单，订单服务请求库存服务扣减库存，**只要下单成功则库存扣减成功**。

**可用性：**

任何时间都可以访问订单服务和库存服务，系统保证可用。

**分区容忍性：**

也叫分区容错性，分布式系统在部署时服务器可能部署在不同的网络分区，比如上图中订单服务在北京，库存服务在上海，由于处于不同的网络分区如果网络通信异常就会导致节点 之间无法通信，当出现**由于网络问题导致节点之间无法通信**，此时**系统**仍然是对外提供服务的这叫做**满足**分区容忍性。

CAP理论要强调**在分布式系统中C、A、P这三点不能全部满足。**

由于是分布式系统就要满足分区容忍性，因为分布式系统难免存在网络分区，不能因为网络异常导致整个系统不可用，所以P是一定要满足的。

满足P，那么C和A不能同时满足。

拿上图举例说明：

当订单服务与库存服务出现**网络通信异常**，订单服务无法访问库存服务，此时如果要保证**数据一致性**则下单接口必须不可用，如果要保证**可用性**数据将出现不一致。

<whiteboard token="R7grwhkmJhSEn8bR39hc5ai7nEb"></whiteboard>

学习了CAP理论我们知道进行分布式事务控制要在C和A中作出取舍，**进行分布式事务控制要么保证CP要么保证AP**。

具体要根据应用场景进行判断，下边举例说明CP和AP业务场景的例子。

**符合CP的场景**：满足C舍弃A，**强调一致性**。

金融系统：一般需要在多个账户之间进行交易或资金转移的操作通常需要满足CP，这是因为在这种场景下，数据的一致性是至关重要的，确保不会发生资金丢失、重复扣款或其他意外情况，源账号和目标账号的转账结果要么都成功要么都失败，不会存在一个成功一个失败的情况。

库存系统：在多个仓库之间进行库存转移或销售操作时，需要确保库存的一致性，防止商品超卖或库存混乱。

      订票系统：需要确保预订信息的一致性，避免出现同一个资源被多次预订的问题。

Zookeeper：可作为注册中心，支持CP，拿主节点选举举例，当主节点异常进行选举，选举期间所有节点不可用，保证数据的一致性。

Redis：Redis主从模式是CP模式，当主从通信异常此时无法向主节点写数据。

Nacos：Nacos也支持CP，不过它默认是AP模式，当客户端注册为非临时节点时此时为CP模式，注册为非临时节点就需要实时上报心跳，即使在一段时间内未收到心跳信息，该实例仍然会保留在服务列表中，适用于配置中心。

**符合AP的场景**：满足A舍弃C，**强调可用性**。

AP强调的是可用性，允许短暂的不一致但是要保证最终一致性，在实际应用中符合AP的场景较多。

订单退款：退款后状态为退款中，24小时后退款金额到帐。

积分系统：注册送积分，注册成功积分在24小时后到账。

跨行转账：一般转账支持CP，还有的支持AP，源账号扣减金额后需要等一段时间目标账户才到账，或者源账号扣款后由于目标账号有问题过一段时间将转账金额退回到源账户。

MySQL主从复制：支持AP，向主节点写数据，异步同步到的从节点。

      Nacos：默认支持AP，即临时节点的情况，会实时上报心跳，如果一段时间内未收到心跳信息，Nacos 会将该实例标记为不可用并从服务列表中移除。

**在生产中AP场景应用的更多，强调的是可用性,允许出现短暂不一致，最终达到数据一致性。**



### 2.4. 安装Seata

#### 2.4.1 认识Seata

解决分布式事务的方案有很多，但实现起来都**比较复杂**，因此我们一般会使用开源的框架来解决分布式事务问题。在众多的开源分布式事务框架中，功能最完善、使用最多的就是阿里巴巴在2019年开源的Seata了。

<bookmark name="Seata 是什么" href="https://seata.io/zh-cn/docs/overview/what-is-seata.html"></bookmark>



其实分布式事务产生的一个重要原因，就是参与事务的多个分支事务互相无感知，不知道彼此的执行状态。因此解决分布式事务的思想非常简单：

就是找一个统一的**事务协调者**，与多个分支事务通信，检测每个分支事务的执行状态，保证全局事务下的每一个分支事务同时成功或失败即可。大多数的分布式事务框架都是基于这个理论来实现的。



Seata也不例外，在Seata的事务管理中有三个重要的角色：

-  **TC (Transaction Coordinator) -** **事务协调者：**维护全局和分支事务的状态，协调全局事务提交或回滚，相当于监控中心。
-  **TM (Transaction Manager) -** **事务管理器：**定义全局事务的范围、开始全局事务、提交或回滚全局事务。 
-  **RM (Resource Manager) -** **资源管理器：**管理分支事务，与TC交谈以**注册分支事务**和报告分支事务的状态，并驱动分支事务提交或回滚。 



Seata的工作架构如图所示：

![dt6zbj8fzox09px0frkcisszneg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dt6zbj8fzox09px0frkcisszneg.png)
> 图注：这张图展示了Seata的工作架构，分为三个核心组件：TM、RM与TC。左侧的TM负责开启全局事务，管理全局事务的范围、提交或回滚全局事务，对应全局事务模块；每个参与事务的微服务旁都配有RM，RM作为资源管理器，会向TC注册分支事务并报告其状态，还能驱动分支事务的提交或回滚，对应分支事务模块；右侧的TC则会接收TM的全局事务提交、回滚请求，以及各RM的分支事务相关汇报，是Seata的事务协调中心。



其中，**TM**和**RM**可以理解为Seata的**客户端部分**，引入到参与事务的微服务依赖中即可。将来**TM**和**RM**就会协助微服务，实现本地分支事务与**TC**之间交互，实现事务的提交或回滚。

而**TC**服务则是事务协调中心，是一个独立的微服务，需要**单独**部署。

Seata支持四种不同的分布式事务解决方案：

- **XA-事务类型=CP**
- **TCC**
- **AT-事务类型=AP**
- **SAGA**

这四种方案可以满足CP和AP的需求，比如：XA可以实现CP即强一致性，AT可以实现AP最终一致性，四种模式中AT模式使用较多，本课程重点讲解AT模式。

根据Seata的结构首先需要安装TC事务协调器。

#### 2.4.2 准备数据库表

Seata支持多种存储模式，但考虑到持久化的需要，我们一般选择基于数据库存储。虚拟机中已经提供了seata所需要的数据库：

![xjdxbyphjopjeqxmw3sch2j7nzg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xjdxbyphjopjeqxmw3sch2j7nzg.png)
> 图注：这张图片展示了Seata所需的数据库表结构，显示存在一个名为“seata”的数据库，其下共有4张数据表，分别是branch_table、distributed_lock、global_table、lock_table，这些是部署Seata服务保护与分布式事务组件时，所需的核心数据库表，对应文档中准备Seata所需数据库的内容部分，这些表是支撑Seata分布式事务功能的基础。

#### 2.4.3 准备配置文件

课前资料准备了一个seata目录，其中包含了seata运行时所需要的配置文件：

![htbwbqpxvo7itpxbaj0cmfn0nje](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/htbwbqpxvo7itpxbaj0cmfn0nje.png)
> 图注：这张图片展示了“资料”文件夹下的seata目录内容，目录中包含logback、lua等多个文件夹，以及application.example.yml、application.yml等配置文件，其中以红色框突出标记了application.yml这个Yaml源文件，该文件大小为4KB。结合上下文可知，这是课前准备的seata运行所需的配置文件，其中包含中文注释，需修改虚拟机IP及MySQL的账户密码。

其中包含中文注释，大家可以自行阅读。

注意：需要修改自己虚拟机的ip地址和MySQL数据库的账户和密码。



我们将整个seata文件夹拷贝到虚拟机的`/root`目录：

![vk8gbakr8oahrdx4wxrc5y4rnwd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vk8gbakr8oahrdx4wxrc5y4rnwd.png)
> 图注：这张图片展示了虚拟机/root目录下的文件列表，其中红色框突出显示的“seata”文件夹是核心内容。该目录包含多个文件夹和文件，包括demo、mysql、nacos、nginx等文件夹，以及allinone.tar、Dockerfile、hm-service.jar等各类文件，所有项目的所有者均为root。这张图片对应文档中关于“将整个seata文件夹拷贝到虚拟机的/root目录”的操作步骤，直观展示了拷贝完成后seata文件夹在目标目录中的位置。



#### 2.4.4 Docker部署

删除旧容器

```Plain Text
docker rm -f seata1.5.2 seata-server
```

启动新容器

```Shell
docker run --name seata \
-p 8099:8099 \
-p 7099:7099 \
-e SEATA_IP=192.168.101.68 \
-v /root/seata:/seata-server/resources \
-d --restart=always \
seataio/seata-server:1.5.2
```

查询容器

![mgxjbkc1soeibbx3xmqcxkcxn6b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mgxjbkc1soeibbx3xmqcxkcxn6b.png)
> 图注：这张图片展示的是Docker环境下的容器查询命令执行结果，对应文档中“查询容器”的内容。图中执行了docker ps命令，其中标注出的红框部分显示seataio/seata-server:1.5.2版本容器的端口映射信息，该容器的端口配置为7091/tcp、0.0.0.0:7099->7099/tcp、8091/tcp、0.0.0.0:8099->8099/tcp，容器名称为seata1，创建时长为8分钟，该信息用于验证Seata的Docker部署是否成功完成。



Seata控制台：http://192.168.101.68:7099/

账号：admin

密码：admin

![drbjbfhyxonpcyxtbuqcntvcnab](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/drbjbfhyxonpcyxtbuqcntvcnab.png)
> 图注：这是Seata控制台的事务信息页面，页面地址为192.168.101.68:7099/#/TransactionInfo，和文档中给出的Seata控制台访问地址对应。页面顶部设有首页、Seata企业版等导航选项，右上角显示当前登录用户为admin，页面核心区域为事务信息查询区，可按创建时间、结束时间、xid、applicationId等条件筛选事务，还设有是否包含分支事务的开关、重置及搜索按钮，下方的事务列表当前无数据，显示“没有数据”，同时提供分页功能。



### 2.5. 微服务集成Seata

<callout emoji="🚅">
交易服务、购物车、商品服务
引入seata依赖
配置seata
交易服务、购物车、商品服务的seata回滚日志表
修改注解为@GlobalTransactional
</callout>

![otynb09t6oumddx559kcxso6nxf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/otynb09t6oumddx559kcxso6nxf.png)
> 图注：图片展示了交易服务、购物车服务、库存服务之间的关系及操作流程。交易服务 自动生成图片

接下来我们实现下单扣减库存的需求，参与分布式事务的每一个微服务都需要集成Seata，下单扣减库存涉及两个微服务，即交易微服务、购物车、商品微服务，下边我们以交易微服务为例在`trade-service`中集成Seata。



#### 2.5.1.引入依赖

分别在交易服务、购物车、商品服务引入seata依赖。

```XML

  <!--seata-->
  <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
  </dependency>
```



#### 2.5.2.Seata配置

分别在交易服务、购物车、商品服务引入seata配置。

内容如下：

```YAML
seata:
  registry: # TC服务注册中心的配置，微服务根据这些信息去注册中心获取tc服务地址
    type: nacos # 注册中心类型 nacos
    nacos:
      server-addr: 192.168.101.68:8848 # nacos地址
      namespace: "" # namespace，默认为空
      group: DEFAULT_GROUP # 分组，默认是DEFAULT_GROUP
      application: seata-server # seata服务名称
  tx-service-group: hmall # 事务组名称
  service:
    vgroup-mapping: # 事务组与tc集群的映射关系
      hmall: "default"
```

 tx-service-group: 事务组是seata进行全局事务管理的逻辑单元，通常按项目为单位进行定义

vgroup-mapping：事务组与tc集群的映射，配置事务组名称与seata TC集群的映射关系

"defualt"：seata集群名称，seata服务实例上传至nacos，在nacos中查看seata集群如下：

![gywrbqnayo3fk3xaiiachendngh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/gywrbqnayo3fk3xaiiachendngh.png)
> 图注：这张图展示了Nacos中seata集群的配置界面，是seata TC集群在Nacos中的查看页面。页面上方的区域展示了服务名、分组、保护阈值、元数据、服务路由类型等信息，其中服务名为seata server，分组为DEFAULT_GROUP，保护阈值为0，服务路由类型为none，集群默认标识为default。页面下方的配置列表区域，展示了一条IP为192.168.101.68、端口为8099的服务实例信息，该实例的临时实例状态为true，权重为1，健康状态为true，操作栏提供编辑和下线功能，还设有添加元数据的输入框与添加按钮。



#### 2.5.3.seata数据库表

seata的客户端在解决分布式事务的时候需要记录一些中间数据，保存在数据库中。因此我们要先准备一个undo_log这样的表，会面会用用到（虚拟机item和trade数据库中中已经提供这张表了）。



![bwv0bl59bohlsdx91p0ckcelngc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/bwv0bl59bohlsdx91p0ckcelngc.png)
> 图注：这张图展示了项目的数据库表结构，呈现了两个核心业务相关数据库中的关键表信息，其中高亮突出显示了undo_log表。具体来看，名为hm-item的数据库下，tables分类中的item数据库内存在undo_log表；名为hm-trade的数据库下，也包含tables分类，该分类下的order、order_detail、order_logistics表为业务相关表，同时也标注了undo_log表。该图对应的上下文提及，Seata解决分布式事务需要使用undo_log中间记录表，结合图中内容可知，hm-item和hm-trade这两个项目数据库中已预置了该所需的undo_log表。



#### 2.5.4.实现分布式事务

前边在没有使用Seata是无法控制分布式事务的，接下来我们用seata控制分布式事务。

我们找到`trade-service`模块下的`com.hmall.trade.service.impl.OrderServiceImpl`类中的`createOrder`方法，也就是下单业务方法。

将其上的`@Transactional`注解改为Seata提供的`@GlobalTransactional`：

![gjopbb9adoiyh7xwtzncttgrnph](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/gjopbb9adoiyh7xwtzncttgrnph.png)
> 图注：图片展示了OrderServiceImpl.java文件中createOrder方法的代码。在方法上方，有`@GlobalTransactional`注解，该注解被红色框突出显示。上下文提到将其上的`@Transactional`注解改为Seata提供的`@GlobalTransactional`，此图片直观呈现了代码中该注解的位置，与上下文关于修改事务注解的内容紧密相关，是实现分布式事务时关键代码示例。

`@GlobalTransactional`注解就是在标记事务的起点，将来TM就会基于这个方法判断全局事务范围，初始化全局事务。



我们重启`trade-service`、`item-service`服务，再次测试分布式事务：

<callout emoji="✍️">
1.仍然在下单方法最后添加异常，扣减库存成功下单失败观察最终扣减库存是否会回滚。
2.下单正常但扣减库存失败观察最终下单数据是否回滚。
</callout>



那么，Seata是如何解决分布式事务的呢？

### 2.6. Seata 工作模式

Seata支持四种不同的分布式事务解决方案，Seata默认使用的是AT模式。

- **XA**
- **TCC**
- **AT**
- **SAGA**

这里我们以**`AT`模式**来给大家讲解其实现原理。

#### 2.6.1 AT模式-AP场景



<whiteboard token="Mbriwgj8xhpSpdbBRZIcDOfNnle"></whiteboard>

`AT`模式的基本流程图：

![g9vrb6hyhoppe0xwnjdcsuu6ntc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/g9vrb6hyhoppe0xwnjdcsuu6ntc.png)
> 图注：该图为Seata AT模式的基本流程图，清晰展示了全局事务的运作流程。首先由TM开启全局事务，TM会调用多个分支的RM，RM在阶段一需完成注册分支事务、记录包含更新前后快照的undo-log、执行业务SQL并提交、报告事务状态的操作；阶段二提交时RM会删除undo-log，回滚时则依据undo-log将数据恢复至更新前，而TC负责全局事务的协调，最终完成全局事务的提交或回滚操作。

阶段一`RM`的工作：

- 注册分支事务
- 记录undo-log（数据快照）
- 执行业务sql并提交
- 报告事务状态

阶段二提交时`RM`的工作：

- 删除undo-log即可

阶段二回滚时`RM`的工作：

- 根据undo-log恢复数据到更新前



下边我们用一个真实的业务来梳理下AT模式的原理。

比如，现在有一个数据库表，记录用户余额：

<table><colgroup><col/><col/></colgroup><thead><tr><th><h6><b>id</b></h6></th><th><h6><b>money</b></h6></th></tr></thead><tbody><tr><td>1</td><td>100</td></tr></tbody></table>

其中一个分支业务要执行的SQL为：

```SQL
 update tb_account set money = money - 10 where id = 1
```

AT模式下，当前分支事务执行流程如下：

**一阶段**：

1. `TM`发起并注册全局事务到`TC`
2. `TM`调用分支事务
3. 分支事务准备执行业务SQL
4. `RM`拦截业务SQL，根据where条件查询原始数据，形成快照。

```JSON
{
  "id": 1, "money": 100
}
```

1. `RM`执行业务SQL，提交本地事务，释放数据库锁。此时 money = 90
2. `RM`报告本地事务状态给`TC`

**二阶段**：

1. `TM`通知`TC`事务结束
2. `TC`检查分支事务状态

   1. 如果都成功，则立即删除快照
   2. 如果有分支事务失败，需要回滚。读取快照数据（{"id": 1, "money": 100}），使用快照恢复数据库，此时数据库再次恢复为100。





流程图：

![s1habvf12ofsdpxfsftcdhvgnpb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/s1habvf12ofsdpxfsftcdhvgnpb.png)
> 图注：这张图是分布式事务中AT模式在AP场景下的核心流程图，分为两个阶段。阶段一的流程依次为注册分支事务、记录快照、执行业务SQL、提交并报告事务状态；阶段二以菱形“是否可以提交”为判断节点，若选择“是”则执行删除快照的操作，若选择“否”则执行读取快照、恢复数据的操作，最终流程结束。图中还展示了ID为1、金额为100的快照数据，与商品服务扣减库存的场景相关联，是AT模式断点调试演示中对应的事务快照逻辑，体现了AT模式通过快照处理事务提交或回滚的核心机制。

###### 演示AT模式

下边通过断点调试演示AT模式的整体流程：

首先在提交订单方法中模拟异常并打断点

![f09mb1yupopygoxl6f0c8edaned](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/f09mb1yupopygoxl6f0c8edaned.png)
> 图注：图片展示的是OrderServiceImpl.java代码片段，对应于演示AT模式下提交订单方法中模拟异常并打断点的场景。代码中包含保存订单、保存订单详情、清理购物车商品、扣减库存等操作，其中扣减库存部分模拟了异常情况，当库存不足时抛出异常。图片与上下文紧密相关，直观呈现了代码中模拟异常并打断点的位置，为后续观察分支事务执行情况及全局事务开启等操作提供代码参考。

当代码执行到断点处每个分支事务已经执行完成，

通过观察seata控制台，已经开启一个全局事务

![k57tbxtnyonp7bxf86dca3znnrb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/k57tbxtnyonp7bxf86dca3znnrb.png)
> 图注：这张图片展示的是Seata控制台的事务信息页面，用于展示分布式事务的相关数据。页面中呈现了一条全局事务记录，其事务ID为192.168.101.68:809927595734695333932，关联的应用名称为hmall，事务服务组为trade-service，事务名称为createOrder(com.hmall.trade.domain.dto.OrderFormDTO)，事务状态显示为Begin，该事务包含分支事务，其中对应商品服务的分支事务ID为27595734695333932，还可通过该页面观察全局事务及分支事务的运行状态，这与上下文里演示AT模式全局事务的内容相契合。

分支事务如下

![mfqabf5l1oiarwxoqntcaqbynlb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mfqabf5l1oiarwxoqntcaqbynlb.png)
> 图注：图片展示的是Seata控制台中全局锁信息页面。页面显示了创建时间、起止日期、结束日期、xid、branchid、transactionid、resourceid、tableName、pk、rowKey、gmtCreate、gmtModify等字段信息。其中，branchid、transactionid、resourceid、tableName、pk、rowKey等字段有具体数值，如branchid为192.168.101.68:8099:27595734695333929等。该图片与上文演示AT模式下分支事务执行情况的内容相关，用于辅助说明分支事务的执行状态。

上图中其中一个分支事务27595734695333932即商品服务已经成功扣减库存，我们可以观察hm-item数据库的item表的stock字段值。

当抛出异常后全局事务回滚，每个分支事务全部回滚。

再次查询hm-item数据库的item表，发现stock库存值已恢复。



#### 2.6.2 XA模式

`XA` 规范 是` X/Open` 组织定义的分布式事务处理（DTP，Distributed Transaction Processing）标准，XA 规范 描述了全局的`TM`与局部的`RM`之间的接口，几乎所有主流的数据库都对 XA 规范 提供了支持。



XA是规范，目前主流数据库都实现了这种规范，实现的原理都是基于两阶段提交。



正常情况：

![ozp1bloybohrnvxkkdic5ntpnmh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ozp1bloybohrnvxkkdic5ntpnmh.png)
> 图注：图片展示了Seata对原始XA模式的改造架构。左侧为第一阶段，事务协调者向RM发送“准备（prepare）”指令，RM回复“就绪（ready）”；右侧为第二阶段，事务协调者向RM发送“提交（commit）”指令，RM回复“已提交（committed）”。该图与上下文紧密相关，直观呈现了Seata在XA模式下，事务协调者与RM在第一、二阶段的交互过程，帮助理解Seata对原始XA模式的简单封装和改造。



异常情况：

![kq4ybu8wwo8t8bxozdjcsr7sn8g](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/kq4ybu8wwo8t8bxozdjcsr7sn8g.png)
> 图注：图片展示了Seata XA模式下事务协调者与RM（资源管理器）的交互流程。左侧为第一阶段，事务协调者发送“准备（prepare）”指令给RM，RM回复“就绪（ready）”或“失败（fail）”；右侧为第二阶段，事务协调者发送“回滚（fallback）”指令给RM，RM回复“已回滚”。该图与上下文紧密相关，直观呈现了Seata XA模式中事务协调者与RM在不同阶段的交互情况，帮助理解其工作流程。





一阶段：

- 事务协调者通知每个事务参与者执行本地事务
- 本地事务执行完成后报告事务执行状态给事务协调者，此时事务不提交，继续持有数据库锁



二阶段：

- 事务协调者基于一阶段的报告来判断下一步操作
- 如果一阶段都成功，则通知所有事务参与者，提交事务
- 如果一阶段任意一个参与者失败，则通知所有事务参与者回滚事务



Seata对原始的XA模式做了简单的封装和改造，以适应自己的事务模型，基本架构如图：

![hor8bt08yovlcmxfdiqcdoxnnfb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/hor8bt08yovlcmxfdiqcdoxnnfb.png)
> 图注：这张图是Seata的XA模式基本架构图，展示了该模式下各组件的交互流程。架构包含TM（事务管理器）、两个微服务模块及对应的RM（资源管理器）、TC（事务协调者）。TM的操作包括开启全局事务、调用分支事务；RM一阶段需完成注册分支事务、执行业务SQL但不提交、向TC报告事务状态的操作；TC在二阶段会检查各分支事务状态，根据结果通知RM执行提交或回滚操作，RM二阶段接收指令后完成对应事务操作，最终由TM进行全局事务的提交或回滚。

`RM`一阶段的工作：

1. 注册分支事务到`TC`
2. 执行分支业务sql但不提交
3. 报告执行状态到`TC`



`TC`二阶段的工作：

1.  `TC`检测各分支事务执行状态

   1. 如果都成功，通知所有RM提交事务
   2. 如果有失败，通知所有RM回滚事务 



`RM`二阶段的工作：

- 接收`TC`指令，提交或回滚事务





`XA`模式的优点是什么？

- 事务的强一致性，满足ACID原则
- 常用数据库都支持，实现简单，并且没有代码侵入



`XA`模式的缺点是什么？

- 因为一阶段需要锁定数据库资源，等待二阶段结束才释放，性能较差
- 依赖关系型数据库实现事务



简述`AT`模式与`XA`模式最大的区别是什么？

- `XA`模式一阶段不提交事务，锁定资源；`AT`模式一阶段直接提交，不锁定资源。
- `XA`模式依赖数据库机制实现回滚；`AT`模式利用数据快照实现数据回滚。
- `XA`模式强一致；`AT`模式最终一致



可见，AT模式使用起来更加简单，无业务侵入，性能更好。因此企业90%的分布式事务都可以用AT模式来解决。



#### 2.6.3 TCC模式

TCC模式与AT模式非常相似，每阶段都是独立事务，不同的是TCC通过人工编码来实现数据恢复。需要实现三个方法：

- Try：资源的检测和预留； 
- Confirm：完成资源操作业务；要求 Try 成功 Confirm 一定要能成功。
- Cancel：预留资源释放，可以理解为try的反向操作。

<whiteboard token="UNoSw3UjShxzYPbZdvBcFKJInFh"></whiteboard>

举例，一个扣减用户余额的业务。假设账户A原来余额是100，需要余额扣减30元。

- **阶段一（ Try ）**：检查余额是否充足，如果充足则冻结金额增加30元，可用余额扣除30

初识余额：

![ui1cbjbllobvxdxfktica1wznbb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ui1cbjbllobvxdxfktica1wznbb.png)
> 图注：图片展示了余额充足时的可用余额情况。画面中有一条灰色的条形图，两端数字分别为0和100，中间显示“可用余额”。这与文档中“举例，一个扣减用户余额的业务。假设账户A原来余额是100，需要余额扣减30元”上下文对应，此时余额充足，可用余额为100，总金额=冻结金额+可用金额，数量依然是100不变。

余额充足，可以冻结：

![ajbdbup9gomjmixmtaqct9qznef](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ajbdbup9gomjmixmtaqct9qznef.png)
> 图注：这张图展示了TCC模式中扣减用户余额业务的初始状态：以横向数轴为参照，整体对应用户原总余额100，左侧红色区域标注为“冻结金额”，范围为0到30，代表冻结的30元；右侧灰色区域标注为“可用余额”，范围为30到100，代表可用的70元，符合业务描述中“余额充足，可以冻结30元”的阶段一初始状态，此时总金额为冻结金额与可用金额的总和，保持原有的100不变。

此时，总金额 = 冻结金额 + 可用金额，数量依然是100不变。事务直接提交无需等待其它事务。

- **阶段二（Confirm)**：假如要提交（Confirm），则冻结金额扣减30

确认可以提交，不过之前可用金额已经扣减过了，这里只要清除冻结金额就好了：

![o5smbwx1fo3wnix77muc08wdnqf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/o5smbwx1fo3wnix77muc08wdnqf.png)
> 图注：图片展示了余额界面，上方显示“可用余额”，下方有数字“0”和“70”。数字“0”位于界面左侧，与“可用余额”文字对齐；数字“70”位于界面右侧，与“可用余额”文字对齐。该图片与文档中“阶段二（Confirm）”内容相关，用于说明在TCC模式下，确认可以提交时，可用余额为70元，此时总金额=冻结金额+可用金额=0+70=70元。

此时，总金额 = 冻结金额 + 可用金额 = 0 + 70  = 70元

- **阶段二(Canncel)**：如果要回滚（Cancel）则冻结金额扣减30，可用余额增加30

需要回滚，那么就要释放冻结金额，恢复可用金额：

![vz7abxm6bozrh2xj6gdchjovnse](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vz7abxm6bozrh2xj6gdchjovnse.png)
> 图注：这张图片展示了一个代表可用余额的进度条式可视化元素，整体呈深灰色长条形，上方标注有“可用余额”的文字，左侧刻度为0，右侧刻度为100，呈现出金额从0到100的区间范围。该图是Seata TCC模式相关内容的补充，结合上下文，它应该用于直观展示TCC模式中阶段操作前后的可用金额等相关数值区间，辅助说明TCC模式在提交或回滚操作中的金额变化情况，对应到文档里TCC模式Confirm或Cancel阶段涉及到的可用金额相关的数值展示。

Seata中的TCC模型依然延续之前的事务架构，如图：

![wxjlbtyveo2b1mxcp5oc536hnrg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wxjlbtyveo2b1mxcp5oc536hnrg.png)
> 图注：图片展示了Seata中TCC模式下的事务架构。左侧为TM（Transaction Manager），右侧为TC（Transaction Coordinator）。TM在1.1开启全局事务后，1.2调用分支，分支中1.4资源预留（Try）、2.4 Confirm、2.4 cancel。TC在1.3注册分支事务、1.5报告事务状态后，2.3提交、2.3回滚。RM为资源管理器。该图与上下文紧密相关，直观呈现了TCC模式下各阶段的事务处理流程。

TCC模式的每个阶段是做什么的？

- Try：资源检查和预留
- Confirm：业务执行和提交
- Cancel：预留资源的释放

TCC的优点是什么？

- 一阶段完成直接提交事务，释放数据库资源，性能好
- 相比AT模型，无需生成快照，无需使用全局锁，性能最强
- 不依赖数据库事务，而是依赖补偿操作，可以用于非事务型数据库

TCC的缺点是什么？

- 有代码侵入，需要人为编写try、Confirm和Cancel接口，太麻烦
- 软状态，事务是最终一致
- 需要考虑Confirm和Cancel的失败情况，做好幂等处理



#### 2.6.4 面试题

seata是怎么进行分布式事务控制的？/Seata的工作原理？



## 作业



### 对下单方法分布式事务控制

实现完整的分布式事务控制并进行测试：

流程如下：

<whiteboard token="COWywhJFIhZT8rbFHpScX2jCnnd"></whiteboard>

提示：

根据上图分析可知下单方法的分布式事务控制范围包括：下单、清理购物车、扣减库存，分别隶属交易服务、购物车服务、商品服务。

要想使用seata控制下单方法的分布式事务，需要将相关微服务集成 seata。

实现完成参考老师课堂上的方法进行分布式事务控制的测试。



### 对支付方法分布式事务控制

除了下单业务以外，用户如果选择余额支付，前端会将请求发送到pay-service模块。而这个模块要做三件事情：

- 直接从user-service模块调用接口，扣除余额付款
- 更新本地（pay-service）交易流水表状态
- 通知交易服务（trade-service）更新其中的业务订单状态



流程如图：

<whiteboard token="UqIpwoF9ghmI36bRCfsc2ciEnZc"></whiteboard>

显然，这里也存在分布式事务问题。



前端会提交支付请求，业务接口的入口在`com.hmall.pay.controller.PayController`类的`tryPayOrderByBalance`方法：

![osrib1pvnoqy1pxxlqccnhutnah](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/osrib1pvnoqy1pxxlqccnhutnah.png)
> 图注：图片展示了pay-service模块中PayController类的部分代码。其中，`tryPayOrderByBalance`方法被红色框突出显示，该方法为`@PostMapping("/id")`注解，接收`@PathVariable("id")`的`id`参数和`@RequestBody`的`payOrderDTO`参数。该方法内部调用了`payOrderService.tryPayOrderByBalance(payOrderDTO)`。此代码与文档中对下单方法分布式事务控制的描述相关，用于实现直接从user-service模块调用接口、扣除余额付款等功能。



对应的service方法如下：

```Java
@Override
@Transactional
public void tryPayOrderByBalance(PayOrderDTO payOrderDTO) {
    // 1.查询支付单
    PayOrder po = getById(payOrderDTO.getId());
    // 2.判断状态
    if(!PayStatus.WAIT_BUYER_PAY.equalsValue(po.getStatus())){
        // 订单不是未支付，状态异常
        throw new BizIllegalException("交易已支付或关闭！");
    }
    // 3.尝试扣减余额
    userClient.deductMoney(payOrderDTO.getPw(), po.getAmount());
    // 4.修改支付单状态
    boolean success = markPayOrderSuccess(payOrderDTO.getId(), LocalDateTime.now());
    if (!success) {
        throw new BizIllegalException("交易已支付或关闭！");
    }
    // 5.修改订单状态
    tradeClient.markOrderPaySuccess(po.getBizOrderNo());
}
```

将上边的方法改为使用seata控制分布式事务并进行测试。

测试方法：

首先通过交易服务提交订单，拿到业务订单号

然后通过支付服务创建支付单，如下：

![moxrbev5coxnt6xsrjzc6niankf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/moxrbev5coxnt6xsrjzc6niankf.png)
> 图注：图片展示的是支付服务接口文档中生成支付单的POST请求界面。左侧导航栏选中“支付相关接口”下的“生成支付单”。右侧请求方式为POST，URL为“/pay-orders”，请求参数部分显示了JSON格式的参数内容，包括amount、bizPayOrderNo、orderInfo等字段，其中“payChannelCode”为“balance”，“payType”为5。该图片与文档中对支付服务创建支付单的测试方法相关，用于说明测试时需通过此接口发送相应参数。

再通过支付服务完成余额支付，如下：

![r6vhbvzvtosom9xdqgxcituqn9c](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/r6vhbvzvtosom9xdqgxcituqn9c.png)
> 图注：这张图片展示的是支付服务接口文档页面，呈现了用于调用用户余额支付接口的调试界面。页面左侧选中“尝试基于用户余额支付”的POST接口，中间调试区域为POST请求，请求地址为/pay-orders/{id}，请求参数采用JSON格式，id参数值为1844538133622915074，参数值包含在代码格式的参数输入框内，右侧设有“发送”和“重置”操作按钮，该页面用于模拟测试前文提及的完成余额支付的操作。



测试完成注意观察相关数据库表的数据变化是否符合预期。







## 汇总

<whiteboard token="SmJIwsgqvh12UBbnakvcoMTCn9g"></whiteboard>

---
> 来源：飞书技术知识库（脱敏版） · 04 微服务与中间件 / 服务保护与分布式事务 / 02 服务保护、分布式事务 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/YEfkwLipbi3GlQkjdRSczQTbnXb)
