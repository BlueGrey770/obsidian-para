---
type: memo
date: 2026-11-06
aliases:
  - "01 微服务服务注册与发现"
---
# 01 微服务服务注册与发现

## 学习目标

1. 能够说出单体架构与微服务架构的优缺点
2. 能够搭建黑马商城项目环境
3. 能够测试黑马商城项目的业务流程
4. 能够说出商城项目的业务模块及技术架构
5. 能够创建商品服务、购物车服务
6. 能够使用RestTemplate实现远程调用
7. 能够说出服务注册与发现流程
8. 能够使用Nacos实现服务注册与发现
9. 会使用OpenFeign实现远程调用
10. 能够抽取API工程



## 搭建环境-重点

在开始本课程之前请**参考“ 环境搭建” 文档搭建黑马商城项目运行环境，最终将黑马商城项目跑起来。**

黑马商城项目是一个单体项目，基于Spring Boot开发，数据库使用MySQL。

微服务框架阶段的课程以黑马商城项目作为案例驱动，使用微服务架构去构建黑马商城项目，并解决其中出现的各种问题。



## 1.认识微服务

学习要点：

1. 认识单体架构，了解其优缺点，以及适合什么场景

   1. 前期项目简单易上手
   2. 开发部署简单
2. 认识微服务架构，了解其优缺点，以及适合什么场景

<whiteboard token="Eo1ZwqHc8hxcwJbu3Rfci4yZnlh"></whiteboard>



咱们之前学习的项目都是单体项目，可以满足小型项目或传统项目的开发。而在互联网时代，越来越多的公司采用微服务架构。

这一章我们从单体架构的优缺点来分析，看看开发大型项目采用单体架构存在哪些问题，而微服务架构又是如何解决这些问题的。

### 1.1 单体架构

单体架构：整个项目只有一个工程，所有业务模块都在这一个工程中，并采用一个数据库，如下图：

<whiteboard token="Q7REwZttch7e2TbDZxjcJCSEn8f"></whiteboard>



项目部署时需要对所有模块一起编译、打包，打成一个jar包或war包，如下图：

![该图片对应文档中单体架构项目部署的相关内容，展示了黑马商城的单体架构：整个项目包含商品管理、用户管理、交易管理、购物车功能四类业务模块，这些模块整体被编译打包成一个JAR包，最终部署到多个服务器上。直观呈现了单体架构的特性，即所有业务模块整合为单一工程，部署时需将全部模块打包为单个文件后统一部署。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/Xf9YbVFAWoObnzxZHb5c75T1nnh.jpg)



当项目规模较小时，这种模式上手快，部署、运维也都很方便，因此早期很多小型项目都采用这种模式。

**优点：**

1.整个项目只有一个工程，结构简单，开发周期短。

2.部署简单。

**缺点？**

随着项目的规模越来越大，团队开发人员也不断增加，单体架构就呈现出越来越多的问题：



1.**团队协作成本高-代码冲突率增大。**

2.**系统发布效率低-无法只发布某个模块的代码。**

3.**扩展困难-无法针对某个模块进行扩缩容，必须整个系统一起扩展。**



**如何解决呢？拆分**



**应用场景：**

单体架构适合小型项目。



### 1.2 微服务架构

随着软件业的兴起，以及互联网的发展，对于一些大型的软件项目，单体架构已不能适应需求，软件架构经过的几代的演变，如今对于大型项目更多的会采用微服务架构。

微服务架构，将整个项目分为多个服务，每个服务是一个工程，通常是根据业务模块去划分，比如：电商项目如果按微服务架构开发就需要独立出来：商品服务、用户服务、交易服务、支付服务等多个服务，这些服务最主要的是要做到边界清晰，单一职责，服务的功能范围可以很小，所以称为微服务架构。

例如，黑马商城项目，我们就可以把商品、用户、购物车、交易等模块拆分，交给不同的团队去开发，并独立部署：

![该图对应微服务架构的示例内容，展示了黑马商城按微服务拆分后的服务架构结构。图中以业务模块为划分，将项目拆分为商品服务、用户服务、交易服务、购物车服务多个独立服务，每个服务开发部署后生成对应的JAR包，再分别部署到服务器，连接各自的MySQL数据库，同时有对应角色指向各服务，体现这些服务可交由不同团队开发并独立部署，符合微服务架构拆分模块的特点。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/LX8tbL3b9oaYsgxZBtTcURZEnyd.jpg)

微服务架构的一些特点：

1. **单一职责原则**：每个微服务专注于执行单一业务功能，并且尽量减少对外部服务的依赖，实现高内聚低耦合。
2. **团队自治**：每个微服务通常由一个小团队（通常不超过10人，即所谓的“两块披萨团队”）负责，包括开发、测试、部署和运维等全过程。
3. **服务自治**：每个微服务都是独立的，可以独立部署、扩展和维护，拥有自己的数据库实例，以确保服务之间的隔离性和最小化相互影响。

以上可以简单理解为：**微服务架构就是由若干小的单体系统组成的。**

单体架构存在的问题可以通过采用微服务架构来解决：

1. **问题**：团队协作成本高**解决**：通过服务拆分，每个服务的代码量相对较小，每个服务通常由1至3名开发者维护，显著降低了协作成本。
2. **问题**：系统发布效率低**解决**：每个服务都是独立部署的，当某个服务需要更新时，只需对该服务进行打包和部署，提高了发布效率并减少了风险。
3. **问题**：系统可用性差**解决**：通过将服务独立部署，并确保每个服务都有自己的资源（如独立的数据库），可以有效防止一个服务的问题影响到其他服务。
4. **问题**：扩展困难**解决**：微服务架构允许根据实际需求有针对性地对特定服务进行扩展或升级，而不需要对整个系统进行扩展，提高了资源利用率和灵活性。



根据上述内容，我们可以总结出微服务架构的优点和缺点如下：

**优点**

1. 有利于项目的扩展与维护

   - 服务之间解耦，易于理解和维护。
   - 每个服务可以独立扩展，提高资源利用效率。

**缺点**

1. 项目复杂性增加

   - 每个服务需要独立的设计、开发和测试，增加了项目的复杂度。
   - 管理多个服务间的依赖关系变得更加困难，例如交易服务可能依赖于商品服务和用户服务。
2. 运维成本增加

   - 需要管理和监控多个服务实例，增加了运维的复杂度。
   - 每个服务都需要单独部署、监控和维护，这可能需要更多的自动化工具和技术支持。

综上所述，虽然微服务架构带来了更好的扩展性和维护性，但也引入了额外的复杂性和运维挑战。

**应用场景：**

微服务架构是适合大型项目



### 1.3 小结

为什么会有微服务架构：随着项目规模的逐渐扩大，无论是从团队协作上还是代码维护上来讲，单体项目已经无法满足开发需求，而微服务架构则恰好能解决上面提到的问题。当然，解决问题的同时也相应带来了一些挑战，这些咱们在接下来的课程中会进行讲解。

最后，同学们心里一定要清楚一件事，微服务架构不是万能的，单体服务有单体服务的优点，微服务架构也有微服务的缺点。实际开发过程中，依照具体项目，选择适合的架构才是最重要的！

<callout emoji="📌">
单体架构：适合小型项目
微服架构：适合大型项目
</callout>

### 1.4. 扩展

AI:分布式架构是什么？

分布式架构是指由多个独立计算机组成的一个系统，即一个系统由多个部分组成，每个部分分别部署在不同的计算机上，这些计算机通过网络互相连接，并且能够协同工作以完成共同的任务。

早期的分布式架构是将整个项目拆分成多个系统，相比微服务架构划分服务的粒度是粗的，比如：一个电商项目包括了认证系统，门户，主站等。

发展到微服务架构会将一个项目划分为多个微服务，例如电商项目包括：商品服务、用户服务、认证服务、订单服务等微服务。

所以，微服务架构就是一种分布式架构，将各个服务分布在不同的计算机上，这些服务之间通过协作去完成业务流程。

下面分享几个微服务架构图：

![这是一张微服务架构示意图，清晰呈现了微服务架构的分层结构与组件关系。图中最上层是前端，包含Vue、NPM、WebPack、WebSocket等开发工具，经Node.js处理后由Nginx集群转发请求；请求经JWT权限校验后进入网关集群，再通过Ribbon负载均衡与Hystrix失败容错策略连接到微服务集群。微服务集群涵盖商品、订单、用户等业务服务，还有消息服务、配置中心等辅助组件，同时通过Eureka注册中心实现服务注册与发现，最终的服务数据存储在数据库中，还集成了静态资源CDN服务、Thymeleaf模板配置等组件，整体展现了分布式微服务架构的协作逻辑。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/MOnkbn8WXoe7y1xzMBCcAoQ5nOd.png)



![图片展示了电商项目中微服务架构图。左侧有手机、电脑、平板等终端设备，通过Gateway Gateway将请求转发至服务A、服务B等微服务。服务A、服务B等微服务间有数据交互，部分服务还与Nacos等配置中心交互。右侧有Skywalking、ELK等监控和日志系统，以及Redis、MySQL等数据库。该图直观呈现了微服务架构下各组件的交互关系，与上下文介绍的电商项目微服务架构相契合。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/NWSsb5OGDos15Bxq1imcqpx5nic.png)



AI:云原生架构是什么？

云原生(Cloud Native)应用程序是为云环境而设计的，能够利用云平台的弹性、可扩展性和服务导向的特性。云原生架构的核心理念包括容器化、微服务、持续交付和DevOps实践。

云原生架构就可以理解为微服务架构的项目使用云平台（阿里云、腾讯云等）去构建。

**主要特征**

1. **容器化**：

   - 应用程序运行在容器内，如Docker容器，以便于标准化、隔离和移植。
   - 容器化提供了轻量级的虚拟化，使应用程序能够在任何环境中一致地运行。
2. **微服务**：

   - 将应用程序分解为一组小型、独立的服务，每个服务都实现了特定的功能。
   - 微服务可以独立部署、扩展和更新，提高了应用程序的可维护性和灵活性。
3. **持续交付**：

   - 采用自动化工具和流程来频繁地部署和更新应用程序。
   - 通过持续集成和持续部署（CI/CD）管道实现快速迭代和反馈循环。
4. **DevOps文化**：

   - 强调开发人员和运维团队之间的紧密合作。
   - 通过自动化和协作工具促进敏捷开发、快速部署和持续改进。
5. **弹性与可扩展性**：

   - 设计应用程序以适应不断变化的工作负载，能够自动伸缩资源。
   - 利用云平台提供的服务，如自动扩展、负载均衡和冗余机制，以保证高可用性和性能。



### 1.5 面试题

你们的项目为什么要用微服务架构？



## 2 构建微服务工程

#### 2.0 环境准备

接下来，我们就一起使用微服务架构去构建黑马商城项目，并解决其中出现的各种问题。

还没有完成环境搭建的同学请**参考“环境搭建” 文档搭建黑马商城项目运行环境，最终将黑马商城项目跑起来。**

本项目是为教学需求而设计的简化版项目，在后期的互相网项目阶段我们会接触完整的电商项目。

### 2.1 [熟悉黑马商城](https://twk3l3l09tj.feishu.cn/wiki/MYNvwtoDniMjuAk4qGcc2sBQnlh)

1. 整体介绍下黑马商城的各个模块
2. idea中介绍下项目模块
3. 启动服务演示下各个模块
4. 最后总结下，以及要如何把功能模块拆分成哪些微服务



#### 2.1.1. 熟悉项目

##### 2.1.1.1 登录功能

首先来看一下登录业务流程，下图黄色框内。

<whiteboard token="QN8DwcjCVhvdfGbceUNcwevcnJc"></whiteboard>

登录入口在`com.hmall.controller.UserController`中的`login`方法：

![图片展示了黑马商城项目中UserController.java代码。该代码位于com.hmall.controller包下，使用了@Api、@RestController、@RequestMapping等注解，定义了用户相关接口。其中，@ApiOperation注解标注了“用户登录接口”，@PostMapping注解标注了“Login”方法，该方法接收@RequestBody @Validated LoginFormDTO类型的loginFormDTO参数，返回UserLoginVO对象。此图与上文介绍登录功能相关，直观呈现了登录接口的代码实现。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/WwzQbwgFJouCdSxykO0c56hUnsg.png)



##### 2.1.1.2 搜索商品

在首页搜索框输入关键字，点击搜索即可进入搜索列表页面：

![图片展示的是黑马商城首页搜索商品页面。页面上方有导航栏，中部有“黑马商城”标志及搜索框，搜索框内提示输入商品名称。下方是商品分类、品牌、价格等筛选栏。搜索结果展示多款手机，如华为、OPPO、三星等品牌手机，每款手机图片、价格、评价数及“立即购买”“加入购物车”按钮等信息一应俱全。该页面对应文档中搜索商品功能介绍，是商品搜索结果展示示例。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/NHwbb9qtGow76extRJWcOMA9n1f.png)



该页面会调用接口：`/search/list`，对应的服务端入口在`com.hmall.controller.SearchController`中的`search`方法：

![这张图片展示了IDEA开发工具中的Java代码编辑界面，内容是hm-service模块下com.hmall.controller包中的SearchController类代码。该类标注了“搜索相关接口”，请求路径为“/search”，类中注入了ItemService实例，其中的search方法是对应搜索接口的服务端入口，该方法使用了@ApiOperation注解标注“搜索商品”，并以@GetMapping注解映射“/list”路径，方法内通过ItemService实现搜索相关的分页查询逻辑，最后将结果封装为PageDTO对象返回，该代码与文档中提到的搜索商品相关接口服务端入口的描述相对应。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/RhCCbciAXot5gkxDpMScJWdFnSW.png)

这里目前是利用数据库实现了简单的分页查询。

##### 2.1.1.3 购物车

在搜索到的商品列表中，点击按钮`加入购物车`，即可将商品加入购物车：

![该图片展示的是商城的商品列表页，共呈现了5款手机商品，页面顶部有“默认”“销量↑”“价格↑”的排序选项，右上角显示商品总数量为10230件，当前处于第1页。每件商品卡片均标注了名称、售价，底部设有“立刻购买”“加入购物车”按钮，其中售价906元的华为nova4商品下方，还配有红色箭头指向“加入购物车”按钮，这和文档中介绍的“商品加入购物车”功能操作内容相呼应。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/M5xPbKdSuoTS4nxvKqZcubW9nIc.png)

加入成功后即可进入购物车列表页，查看自己购物车商品列表：

![这张图片展示的是黑马商城的购物车列表页，页面呈现了三笔已加入的商品信息，包括vivo X23手机、巴布豆纸尿裤、华为nova4手机，每笔信息均标注了商品单价、数量和小计金额。页面右下角显示已选择2件商品，商品总价（不含运费）为1865元，还带有“已付：¥20.00”的标注，同时提供了“全选”“删除选中的商品”“移到我的关注”“清除下装商品”等功能选项，操作列针对每类商品设有“删除”“移到我的关注”的按钮，页面最右侧有“结算”按钮，符合上下文提到的购物车可查看、修改、删除商品的功能描述。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/Oa9pbmRAlo2tHGxAciLcg052n7e.png)

同时这里还可以对购物车实现修改、删除等操作。

相关功能全部在`com.hmall.controller.CartController`中：

![图片展示了黑马商城微服务工程中`com.hmall.controller`包下的`CartController`代码。该控制器包含四个方法，分别为`addItem2Cart`（添加商品到购物车）、`updateCart`（更新购物车数据）、`deleteCartItem`（删除购物车中商品）和`queryMyCarts`（查询购物车列表）。每个方法都标注了`@ApiOperation`注解，说明了方法功能。这些方法与上下文提到的购物车相关功能对应，是业务入口，用于实现添加、更新、删除购物车商品及查询购物车列表的功能。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/SxjOb3f5SoJvlrxYikxca9UanXe.png)



##### 2.1.1.4 下单

在购物车页面点击`结算`按钮，会进入订单结算页面：

![图片展示的是黑马商城的订单结算页面。页面上方有黑马商城的标志及导航栏。中间部分显示了收货人信息，有姓名、手机号、地址等。下方是商品列表，有商品图片、名称、价格、数量等信息，如vivo X21、华为nova 4等手机。右侧有商品数量调整按钮。底部显示了商品总价、优惠金额、运费等信息，合计金额为1865.00元，还有“提交订单”按钮。该图片与上下文介绍的在购物车页面点击“结算”按钮进入订单结算页面相契合。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/OjMpbyIEroPZnaxFfgTcDdlTnLd.png)

点击提交订单，会提交请求到服务端，服务端做3件事情：

- 创建一个新的订单
- 扣减商品库存
- 清理购物车中商品

业务入口在`com.hmall.controller.OrderController`中的`createOrder`方法：

![该图片展示了IDEA开发工具中的代码界面，内容是hmall项目里OrderController类的代码，该类对应文档中提到的订单功能业务入口。图片中明确标注了两个关键信息：一是类上的注解，二是带有@ApiOperation("创建订单")注解的createOrder方法，该方法接收OrderFormDTO类型参数，返回Long类型结果，正是文档中提及的提交订单的业务入口，对应服务端处理下单请求的核心方法代码。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/WFAybgI7LovdVFxIrQ4coXfincd.png)



##### 2.1.1.5 支付

下单完成后会跳转到支付页面，目前只支持**余额支付**：

![图片展示的是黑马商城的支付页面。页面上方有黑马商城的标志及“收银台”字样。中间显示订单提交成功，订单号为1660918244787093500，提示需及时付款，订单将在29分41秒后取消。下方有支付宝支付、微信支付、余额支付选项，当前选中余额支付。下方还有输入支付密码的输入框及“确认支付”按钮。页面右上角显示应付金额为1865.00元。该图片与文档中介绍余额支付流程的内容相关，直观呈现了余额支付页面的样式及关键信息。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/AQKsbaoXyoQl8Ux4tUhcu7oVnBd.png)

在选择**余额支付**这种方式后，会发起请求到服务端，服务端会立刻创建一个支付流水单，并返回支付流水单号到前端。

当用户输入用户密码，然后点击确认支付时，页面会发送请求到服务端，而服务端会做几件事情：

- 校验用户密码
- 扣减余额
- 修改支付流水状态
- 修改交易订单状态

支付成功：

![这张图片展示的是黑马商城的支付结果页面，页面上方有“黑马商城”的标识与“支付结果”字样，页面主体以绿色对勾图标搭配黑色文字，显示“恭喜您，支付成功啦！”，还标注了本次支付的相关信息：支付方式为扣减余额，支付金额为4468.00元，商品预计2024年10月12日到达，页面底部设有“查看订单”和“继续购物”两个功能按钮，该页面是用户完成支付操作后，服务端处理对应的支付流程并返回给用户的结果展示，对应了文档中支付成功的环节。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/DE4hbUmCloFzzTxYXAtcutGtn1d.png)

请求入口在`com.hmall.controller.PayController`中：

![这张图片展示了hmall项目中支付相关的控制器代码，路径为hmall的hm-service模块的src/main/java/com/hmall/controllers目录下的PayController.java文件。文件中定义了PayController类，包含生成支付流水的applyPayOrder方法，以及尝试基于用户余额支付的tryPayOrderByBalance方法，后者通过@PostMapping注解映射了请求，还包含了支付相关的注解标签，对应文档中提及的支付功能的请求入口，即该控制器是处理支付请求的核心代码文件。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/Jl5JbloUEoqTGDxWQOacuxv3nSd.png)

支付成功可以查看user表的余额字段是否减少

![这张图片展示了家政服务o2o项目的数据库相关内容，左侧的项目列表中，展开了hmall项目下的user表；右侧的表数据编辑区域显示了该用户表的字段信息，在所有字段行中，balance字段被红色方框突出标注，其对应的数据值为775100，该内容用于验证黑马商城支付模块的功能是否正常，与文档中测试发现支付成功后用户表余额未减少的bug直接相关，需核对用户余额字段的数值是否符合支付后的变动逻辑。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/Rwxhbgpm6ogeobxpDDRcs2Z5nlc.png)



###### bug修复

Bug描述: 在测试中发现支付成功后用户表的余额字段并没有减少相应的金额。

请排查问题并进行修复。

修复后进行测试。



##### 2.1.1.6 小结

如何去熟悉一个项目？

首先要去熟悉项目的业务需求、业务流程、业务模块等。

然后再去熟悉项目的技术架构，以及所用到的技术点，可以通过阅读代码、系统测试，分模块去熟悉项目。



黑马商城项目包括几个模块：

认证模块、用户管理、商品管理、购物车、交易模块、支付模块等。



黑马商城项目用的什么技术架构：

黑马商城单体项目使用Spring Boot 2.7.X版本开发，数据库用的MySQL8，还用到了MQ、Redis、Elasticsearch等中间件。



### 2.2 创建微服务工程 -学习一个微服务架构-Springcloud

本节我们开始使用微服务架构去开发黑马商城项目。



#### 2.2.1. 系统架构

- 微服架构存在的问题
- 认识微服务框架-springcloud
- 熟悉并掌握掌握相关功能组件
- Spring Cloud Alibaba与Spring Cloud 的联系

##### 2.2.1.1 系统架构图

我们用微服务架构开发黑马商城项目首先需要划分微服务，通常是按照业务模块进行划分，同时也会将一些通用的功能抽取成一个微服务，微服务要做到单一职责，高内聚低耦合。

- 用户端请统一请求到网关（比如nginx），由网关将请求转发到微服务。
- 每个微服务是一个Spring Boot工程，微服务的地址统一由服务注册中心管理。
- 每个微服务有自己的数据库，比如：商品数据库、用户数据库。
- 在数据层除了MySQL之外还有非关系数据库Redis、ES等，满足不同的需求。

系统架构图如下：

<whiteboard token="NUnzwStKuhsHeYbWxfncldttnkf"></whiteboard>

有什么问题？

- 服务变多，客户端该如何访问？
- 服务变多，实例以及配置如何管理？
- 服务变多，服务之间又该如何访问？

答案是：Spring Cloud提供了相关组件的集成！！！





##### 2.2.1.2 认识Spring Cloud

每个微服务是一个SpringBoot工程，微服务之间的通信协作、服务保护等各种问题都有对应的解决方案和微服务组件，SpringCloud框架可以说是目前Java领域最全面的微服务组件的集合了，如下图：

![这张图是微服务架构的组件示意图，核心为“微服务”，由一只手握着放大镜突出显示。围绕微服务，共有六类核心组件，分别属于不同的功能模块：左侧是服务注册发现，包含Eureka、Nacos、Consul；服务远程调用，包含OpenFeign、Dubbo；服务链路监控，包含Zipkin、Sleuth。右侧是统一配置管理，包含SpringCloudConfig、Nacos；统一网关路由，包含SpringCloudGateway、Zuul；流控、降级、保护，包含Hystix、Sentinel。该图对应文档中介绍的Spring Cloud相关内容，直观呈现了Spring Cloud为微服务提供的各类完整解决方案。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/ZA3RbPaQGoFDoAxBLtucCMpWnwe.png)

Spring Cloud 是一个基于 Spring Boot 的**微服务开发框架**，它依托于SpringBoot的自动装配能力，它提供了**一套完整的微服务解决方案**。Spring Cloud 旨在帮助开发者快速构建分布式系统中的服务，包括服务发现、配置管理、API 网关、断路器、智能路由、负载均衡、消息总线、服务间通信等功能。Spring Cloud 是一个非常强大的工具集，可以帮助开发者快速构建健壮的微服务架构。

<bookmark name="Spring Cloud" href="https://spring.io/projects/spring-cloud#overview"></bookmark>

Spring Cloud主要组件包括：

- Eureka、nacos：服务发现与注册。
- Zuul 或 Spring Cloud Gateway：API 网关。
- Hystrix：断路器。
- Ribbon，Spring Cloud LoadBalancer：客户端负载均衡。
- Feign、OpenFeign：声明式服务调用。
- Config Server：外部配置中心。



以下是SpringCloud和SpringBoot的版本对应关系，SpringCloud依赖SpringBoot：

![图片展示了Spring Cloud和Spring Boot的版本对应关系。其中，2021.0.x版本的Spring Cloud对应2.6.x、2.7.x（从2021.0.3开始）的Spring Boot版本。该图片与上下文紧密相关，上下文提到当前Spring Boot版本为4.0.x，大部分企业还在使用2.7.x及以下版本，因此课程使用Spring Cloud 2021.0.x及Spring Boot 2.7.x版本重点讲解Spring Cloud Alibaba框架的核心组件，此图直观呈现了对应版本关系。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/IIYcbbBKboPQpqxIDeDch0Wgnrb.png)



当前SpringBoot版本为4`.0.x`版本，目前在企业中大部分还在使用 2.7.x及以下版本，所以**我们的课程使用：Spring Cloud 2021.0.x以及Spring Boot 2.7.x版本，重点讲解Spring Cloud Alibaba 框架的核心组件。**



Spring Cloud Alibaba 是阿里巴巴基于 Spring Cloud 开发的一套微服务开发工具包在当前非常流行，它**针对 Spring Cloud 的一些组件进行了扩展和替换**，以便更好地适应中国开发者的需求和阿里巴巴集团内部的技术栈。

Spring Cloud Alibaba主要组件包括：

- **Nacos：服务发现和配置中心。**
- **Sentinel：流量控制组件。**
- Dubbo：RPC 框架的 Spring Cloud 支持。
- RocketMQ：消息中间件的支持。



Spring Cloud Alibaba与Spring Cloud 的联系：

- 基于 Spring Boot：两者都是基于 Spring Boot 的微服务框架。
- Spring Cloud 核心理念：Spring Cloud Alibaba 遵循 Spring Cloud 的设计理念和模式。
- 组件兼容：Spring Cloud Alibaba 的组件可以与 Spring Cloud 的其他组件配合使用。

Spring Cloud Alibaba与Spring Cloud 的区别：

- 服务发现：Spring Cloud 使用的是 Netflix Eureka，而 Spring Cloud Alibaba 使用的是 Nacos。
- 断路器：Spring Cloud 原生支持 Hystrix，而 Spring Cloud Alibaba 推荐使用 Sentinel。
- 配置管理：Spring Cloud 使用 Spring Cloud Config Server，而 Spring Cloud Alibaba 使用 Nacos 作为配置中心。
- 负载均衡：Spring Cloud LoadBalancer结合 Nacos实现负载均衡。
- 远程调用支持：Spring Cloud Alibaba 支持 Dubbo 作为 RPC 调用框架，而 Spring Cloud 默认不包含 RPC 支持。



##### 2.2.1.3 面试题

SpringBoot和SpringCloud,请你谈谈对他们的理解？

 

Spring Cloud Alibaba与Spring Cloud区别和联系？



你们Spring Cloud用的什么版本？





学习要点

1. 创建基础工程
2. 创建common工程（复制单体架构下面的模块即可）
3. 拆分商品服务
4. 购物车服务（练习）

#### 2.2.2. 创建基础工程

##### 2.2.2.1 创建父工程

本课程使用maven创建微服务架构的电商项目，首先需要创建父工程hmall-parent（普通maven项目，非脚手架项目），在父工程中主要是要pom.xml中规范依赖的版本，我们已经明确使用Spring Cloud Alibaba ，且SpringBoot的版本使用 2.7.x版本。

在项目根目录创建pom.xml：

```JavaScript
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.hmall</groupId>
    <artifactId>hmall-parent</artifactId>
    <packaging>pom</packaging>
    <version>1.0.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.12</version>
        <relativePath/>
    </parent>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <org.projectlombok.version>1.18.20</org.projectlombok.version>
        <spring-cloud.version>2021.0.3</spring-cloud.version>
        <spring-cloud-alibaba.version>2021.0.4.0</spring-cloud-alibaba.version>
        <mybatis-plus.version>3.4.3</mybatis-plus.version>
        <hutool.version>5.8.11</hutool.version>
        <mysql.version>8.0.23</mysql.version>
    </properties>

    <!-- 对依赖包进行管理 -->
    <dependencyManagement>
        <dependencies>
            <!--spring cloud-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <!--   type=pom ,scope=import 表示继承 和 parent 作用一样             -->
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--spring cloud alibaba-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring-cloud-alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!-- 数据库驱动包管理 -->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <!-- mybatis plus 管理 -->
            <dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>mybatis-plus-boot-starter</artifactId>
                <version>${mybatis-plus.version}</version>
            </dependency>
            <!--hutool工具包-->
            <dependency>
                <groupId>cn.hutool</groupId>
                <artifactId>hutool-all</artifactId>
                <version>${hutool.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <!-- lombok 管理 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${org.projectlombok.version}</version>
        </dependency>
        <!--单元测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.1</version>
                    <configuration>
                        <source>11</source> <!-- depending on your project -->
                        <target>11</target> <!-- depending on your project -->
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```

如果pom.xml文件图标不是pom字样需要右键点击

![这张图片展示了一个蓝色背景的操作选项栏，其中有带加号的“+”标识，以及文字“Add as Maven Project”。该内容对应文档中“如果pom.xml文件图标不是pom字样需要右键点击”的操作指引，是创建Spring Cloud微服务工程（父工程）过程中，针对pom.xml文件图标的异常状态时的右键操作选项，用于将当前项目作为Maven项目添加，帮助开发者完成基础工程创建的相关操作步骤。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/O14bbCWeKo4o2QxOKRWc13Oynbb.png)

如下图：

![图片展示了IntelliJ IDEA中hmall-micro项目的pom.xml文件。左侧项目结构中，hmall-micro项目下有.idea、pom.xml、External Libraries、Scratches and Consoles等文件夹和文件。pom.xml文件图标为“m”，右侧有红色箭头指向该图标。该图片与上文“在项目根目录创建pom.xml”及“如果pom.xml文件图标不是pom字样需要右键点击”内容对应，直观呈现了pom.xml文件在项目中的位置及图标样式。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/F3Ycb5naJo6L9txC0qfcCD4Jnzh.png)

选择jdk版本为11：

![这张图片展示了项目结构设置界面的Project配置页，核心为配置JDK相关信息。页面左侧是项目设置的功能选项栏，选中了「Project」选项；右侧的项目基本信息区域，明确显示SDK选择为11 Oracle OpenJDK version 11.0.15，对应下方配置的语言级别为18 - No new language features，该设置对应文档中“选择jdk版本为11”的操作要求，用于创建Spring Cloud微服务相关工程，界面还呈现了项目名称、编译输出路径等配套信息。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/ThAmbxBt0o7s7uxWEsZcDQpNnPe.png)



在pom.xml中重点定义了Spring Boot、Spring Cloud和Spring Cloud Alibaba的版本号：

![这张图片展示的是父工程根目录下的pom.xml文件内容，该文件用于管理项目依赖，页面标题显示项目名称为hmall，当前打开的是pom.xml文件的编辑界面。内容里的核心部分围绕依赖管理展开，分别定义了Spring Cloud和Spring Cloud Alibaba的依赖，其中Spring Cloud的依赖指定了group为org.springframework.cloud、artifacts为spring-cloud-dependencies，版本引用了spring-cloud.version属性；Spring Cloud Alibaba的依赖指定了group为com.alibaba.cloud、artifacts为spring-cloud-alibaba-dependencies，版本引用了spring-cloud-alibaba.version属性，这与文档中所述在pom.xml中定义相关组件版本号的内容完全对应。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/AIt9b5UdyojcxQxlWhScNyl7ndg.png)

对应的版本：

![图片展示了在IntelliJ IDEA中编辑pom.xml文件的界面。文件中重点定义了Spring Boot、Spring Cloud和Spring Cloud Alibaba的版本号。其中，Spring Boot的版本号为2.7.12，Spring Cloud版本号为2021.0.3，Spring Cloud Alibaba版本号为2021.0.1。这些版本号被红色框线突出显示，与上下文提到在pom.xml中定义版本号的内容相呼应，直观呈现了具体版本信息。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/Ty9tbzu7UothvCxdfoccrpKOntg.png)



##### 2.2.2.2 创建common工程

将原项目的hm-common拷贝到新项目目录下

如果pom.xml文件图标不是pom字样需要右键点击

![图片展示了在Eclipse中将项目添加为Maven项目的操作界面。画面中有一个蓝色的“Add as Maven Project”按钮，按钮上方有一个蓝色的“+”图标。该图片与文档中“2.2.2.2创建common工程”部分内容相关，对应将原项目的hm - common拷贝到新项目目录下后，若pom.xml文件图标不是pom字样需右键点击的操作步骤，用于指导用户在Eclipse中完成将项目添加为Maven项目的操作。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/MMiwbjKwZoviyUxAg8KctKjmn7B.png)

如下图：

![图片展示了IntelliJ IDEA中hmall-micro项目的hmall-common模块的目录结构。左侧目录栏中，hmall-micro项目下有.idea、hmall-common等文件夹，hmall-common文件夹下有src和target文件夹，其中target文件夹被高亮显示。右侧是hmall-common模块的pom.xml文件，其图标为蓝色“m”标识。该图片与上文“将原项目的hm - common拷贝到新项目目录下”内容对应，直观呈现了hmall - common模块在项目中的位置。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/PjlXbUcesoX3ExxEQIHcjf65nYd.png)





#### 2.2.3.商品服务

接下来，我们先把商品管理功能、购物车功能抽取为两个独立的微服务，其它的微服务根据教学的需求大家请自行创建。

##### 2.2.3.1 创建数据库



查看购物车微服务使用的数据库：

![图片展示了商品服务（hm-item）下的tables文件夹内容。其中包含三个文件夹：item、item_sync、undo_log。该图片与文档中“2.2.3.3复制并修改代码”部分内容相关，用于说明在商品服务中，按照模型、持久层、业务层、控制层的顺序拷贝代码后，最终效果中商品服务的tables文件夹下应包含这三个文件夹，与上下文代码复制及结构修改的说明相呼应。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/I8XnbWfJCosFXkxB4E2cbYaqnvd.png)



##### 2.2.3.2 创建工程

在项目中创建module：

选择maven模块，设定JDK版本为11，模块名为：

![这张图片展示了创建Maven模块的界面，用于在微服务工程中创建商品服务对应的module。界面中，模块名称设置为“item-service”，项目位置指向名为“hmall-micro”的工程路径，JDK版本选择了Project SDK 11，构建系统指定为Maven，父项目设置为“hmall-parent”，还显示了Advanced Settings里的GroupId为“com.hmall”、ArtifactId为“item-service”，界面右下角有“Create”和“Cancel”两个操作按钮，左侧还能看到该工程已有的其他模块如“hm-common”的项目结构。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/SvDZb2QakocqDFxQ9W2ccpYHnzf.png)



引入依赖：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>hmall-parent</artifactId>
        <groupId>com.hmall</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>item-service</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    <dependencies>
        <!--common-->
        <dependency>
            <groupId>com.hmall</groupId>
            <artifactId>hm-common</artifactId>
            <version>1.0.0</version>
        </dependency>
        <!--web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--数据库-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <!--单元测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
    </dependencies>
    <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

##### 2.2.3.3 复制并修改代码

拷贝`hm-service`中与商品管理有关的代码到`item-service`，按照模型、持久层、业务层、控制层的顺序拷贝代码，最终效果如下图（注意跟hm-service对比，这里多了item这层包）：

![图片展示了item - service工程的目录结构。在src/main/java目录下，com.hmall.item包中包含controller、domain、po、query、mapper、service等子包，其中domain包下有dto和po子包，po包下有Item类，query包下有ItemPageQuery类，mapper包下有ItemMapper类。resources目录下有mapper文件夹，其中ItemMapper.xml文件被高亮显示。该图与上文“复制并修改代码”内容相关，直观呈现了代码复制后的工程目录及文件分布情况。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/AhhQbrNeloiAwWxXAz0ctNqInNc.png)

在ItemMapper.xml文件中需要修改包路径。

![图片展示的是一个MyBatis的XML映射文件片段。文件中定义了一个名为`ItemMapper`的映射器，其`namespace`属性值为`com.hmall.item.mapper.ItemMapper`。该图片与上下文紧密相关，上下文提到在ItemMapper.xml文件中需修改包路径，因ItemMapper所在包发生变化，故代码需修改包路径，此图直观呈现了需修改的包路径内容，辅助理解代码修改要求。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/BzmDbJ9i4o1pRYxReENcpHvVnCg.png)



这也是因为ItemMapper的所在包发生了变化，因此这里代码必须修改包路径。

编写启动类，代码如下：

```Java
package com.hmall.item;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@MapperScan("com.hmall.item.mapper")
@SpringBootApplication
public class ItemServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ItemServiceApplication.class, args);
    }

}
```

##### 2.2.3.4 创建配置文件

接下来是配置文件，可以从`hm-service`中拷贝：

![这张图片展示了商品服务（item-service）工程的项目目录结构。从顶层目录开始，依次包含src文件夹，其下分为main和resources目录。main目录中的java目录下的com.hmall.item包内，设有controller、domain、mapper、service四个子包，还有启动类ItemServiceApplication。resources目录下有mapper文件夹，以及application.yaml、application-dev.yaml、application-local.yaml三个配置文件。该结构对应文档中创建微服务商品服务的相关内容，其中三个配置文件可从hm-service工程拷贝，application.yaml还需按要求设置端口号、服务名、数据库相关信息及Swagger接口包路径。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/QDiibsZjJosvzTx6kImcKprInop.png)

其中，`application.yaml`内容如下（以下方内容为主，覆盖掉原有的内容）：

注意设置端口号、服务名、数据库账号和密码、swagger接口的包路径com.hmall.item.controller

```YAML
server:
  port: 8081
spring:
  application:
    name: item-service
  profiles:
    active: dev
  datasource:
    url: jdbc:mysql://${hm.db.host}:3306/hm-item?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: ${hm.db.pw}
mybatis-plus:
  configuration:
    default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
  global-config:
    db-config:
      update-strategy: not_null
      id-type: auto
logging:
  level:
    com.hmall: debug
  pattern:
    dateformat: HH:mm:ss:SSS
  file:
    path: "logs/${spring.application.name}"
knife4j:
  enable: true
  openapi:
    title: 商品服务接口文档
    description: "商品服务接口文档"
    url: https://www.itcast.cn
    version: v1.0.0
    group:
      default:
        group-name: default
        api-rule: package
        api-rule-resources:
          - com.hmall.item.controller
```

剩下的`application-dev.yaml`和`application-local.yaml`直接从hm-service拷贝即可。

##### 2.2.3.5 启动微服务

直接运行启动类

![这张图片展示了SpringCloud微服务开发工具中的操作界面，左侧是项目结构，可见名为“item-service”的商品服务工程，其src目录下包含main/java/com/hmall/item的controller、domain等代码包，还有ItemService、ItemServiceImpl等相关类，右侧右键菜单中，“Debug 'ItemServiceApplication'”选项被高亮选中，对应上下文里启动微服务的操作，是运行商品服务启动类的调试启动操作界面，用于完成微服务的启动部署步骤。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/D4eIb1zTSoc1Xqxl6uYcxnclnHs.png)

启动成功：

![图片展示的是微服务工程启动成功后的控制台日志。日志中显示Tomcat在8081端口（http）启动成功，带有上下文路径。还记录了Spring嵌入式WebApplicationContext初始化完成、Swagger接口映射、文档插件启动、自定义文档插件发现等信息。该图片与上文“2.2.3.5启动微服务”内容对应，直观呈现了微服务启动后的运行状态，帮助开发者确认服务是否正常运行。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/OM29bpUwRoHochx313jcT5J6nGe.png)



##### 2.2.3.6 测试微服务

访问商品微服务的swagger接口文档：[http://localhost:8081/doc.html](http://localhost:8081/doc.html)

然后测试其中的根据id批量查询商品这个接口：

![这张图片是商品服务接口文档页面，展示了Springcloud微服务中根据id批量查询商品的GET接口相关信息，该接口的请求方式为GET，请求数据类型是application/x-www-form-urlencoded。图中重点标注了该接口的必填参数为ids，请求类型是query，参数类型为array integer。页面还列出了该接口的响应状态，含200、401、403等状态码及对应的说明，与文档中测试商品微服务相关的内容匹配，用于辅助完成微服务的接口测试操作。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/AdaqbULxGoIvWbxvoS3cRQH2npd.png)

我们从hm-item数据库的item表找几条商品的id进行测试



测试参数：100002672302,100002624500,100002533430，结果如下：

![图片展示的是商品微服务的接口文档页面。左侧为导航栏，选中“商品管理相关接口”下的“GET 根据id批量查询商品”选项。右侧上方显示“商品服务接口文档”，中部有“GET”“POST”“PUT”“DELETE”等操作按钮，当前选中“GET”。下方有“ids”参数输入框，已输入测试参数“100002672302,100002624500,100002533430”，下方是响应数据区域，显示了商品id、名称、价格等信息。该图与上文测试商品微服务接口文档中根据id批量查询商品接口的内容相关。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/CazhbZyAeoL2tfxejIJcU1M6nGd.png)

说明商品微服务创建成功了。



##### 2.2.3.7. 小节

创建商品服务步骤：

1. 创建一个SpringBoot工程
2. 创建controller、service、mapper、domain包结构
3. 拷贝修改代码
4. 创建修改配置文件
5. 启动微服务



#### 2.2.4.购物车服务(练习)

##### 2.2.4.1 创建数据库

查看购物车微服务使用的数据库：

![图片展示了在hmall项目下创建的`hm-cart`模块的数据库结构。其中`tables`文件夹下有`1`个文件，即`cart`。该图片与上文“把hm-service中的与购物车有关功能拷贝过来，最终的项目结构如下”的内容相关，直观呈现了项目结构中与购物车功能相关的数据库表情况，帮助理解购物车服务的数据库基础。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/LV0UbN5YNoRwiCxWe4Ncbotjnbb.png)

##### 2.2.4.2 创建工程

与商品服务类似，在hmall下创建一个新的`module`，起名为`cart-service`

引入依赖：

```JavaScript
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>hmall-parent</artifactId>
        <groupId>com.hmall</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cart-service</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    <dependencies>
        <!--common-->
        <dependency>
            <groupId>com.hmall</groupId>
            <artifactId>hm-common</artifactId>
            <version>1.0.0</version>
        </dependency>
        <!--web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--数据库-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <!--单元测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
    </dependencies>
    <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```



##### 2.2.4.3 复制与修改代码

把hm-service中的与购物车有关功能拷贝过来，最终的项目结构如下：

![这张图片展示了购物车服务工程cart-service的项目目录结构，根目录为cart-service，包含src和resources两个子目录。src目录下的main子目录中，java目录里的包层级依次为com、hmall、cart，cart包内划分了controller、domain、mapper、service等功能子包，每个子包对应了相关代码文件，如controller包下的CartController、domain包下的CartFormDTO和ItemDTO等，service子包的impl文件夹下有CartServiceImpl和ICartService，还有应用启动类CartServiceApplication；resources目录下则设有mapper子目录，用于存放CartMapper.xml等配置文件，该结构与文档中拷贝hm-service相关代码后构建的购物车服务项目结构要求相符。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/DRombC84mo9cl9xvZJ8cfuYqnSb.png)

注意：需要修改CartMapper.xml文件中的包路径。

![图片展示的是一个MyBatis的Mapper XML文件内容。文件声明了版本、编码等信息，定义了映射器的命名空间为“com.hmall.cart.mapper.CartMapper”。该图片与上下文紧密相关，上下文提到需修改CartMapper.xml文件中的包路径，此图片直观呈现了文件中命名空间的设置情况，帮助理解修改包路径的具体位置和内容。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/M0ssb13CYojAdtxJ2KOcIgl4nSh.png)



特别注意的是`com.hmall.cart.service.impl.CartServiceImpl`，其中有两个地方获取当前的用户id，目前我们还没有创建用户服务所以修改UserContext的getUser()方法固定返回1。

![这张图片展示的是UserContext.java文件的代码内容，对应文档中提到的修改UserContext的getUser()方法的操作。该文件里的getUser方法原本是获取当前登录用户信息的方法，为解决还未创建用户服务的问题，方法内当前已固定返回1L，原本通过tl.get()获取用户id的代码已被注释屏蔽。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/CYt0bpaDvo95GGxRpJtcJYvQnkc.png)



**屏蔽ItemService相关代码**

![图片展示了`CartServiceImpl.java`代码片段，其中`@Service`和`@RequiredArgsConstructor`注解被绿色勾选突出显示。`@Service`注解表明该类是Spring框架中的服务类。`@RequiredArgsConstructor`注解用于生成构造方法，自动为所有final的字段生成构造方法。该图片与上下文紧密相关，上下文提到要屏蔽`ItemService`相关代码，此图片展示了购物车服务中`CartServiceImpl`类的部分代码，与上下文中的代码编写内容相呼应。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/H5pPb53djoxIKqxuu5ackuIwnuh.png)

![该图片展示了`CartServiceImpl`类的`handleCartItems`方法代码，对应文档中“屏蔽ItemService相关代码”的要求。方法被分为两个注释说明的步骤：第一步注释为“1.获取商品id”，通过Stream流操作从CartVO列表提取商品id并收集为Set集合；第二步原注释为“2.toto 查询商品”，原本调用`itemService.queryItemByIds(itemIds)`获取商品DTO列表，该ItemService调用即为需要屏蔽的内容，当前已被注释，且后续新增了初始化空ArrayList的代码，还添加了判断该列表是否为空的条件逻辑。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/Amw0bCm3QoNTN0xUWLsczJRrnMr.png)



编写启动类：

```JavaScript
package com.hmall.cart;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@MapperScan("com.hmall.cart.mapper")
@SpringBootApplication
public class CartServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(CartServiceApplication.class, args);
    }

}
```



##### 2.2.4.4 创建配置文件

拷贝商品服务的配置文件到购物车服务下。

`购物车服务的application.yaml`内容如下，注意设置端口号、服务名、数据库名、swagger文档的包路径com.hmall.cart.controller。

```JavaScript
server:
  port: 8082
spring:
  application:
    name: cart-service
  profiles:
    active: dev
  datasource:
    url: jdbc:mysql://${hm.db.host}:3306/hm-cart?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: ${hm.db.pw}
mybatis-plus:
  configuration:
    default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
  global-config:
    db-config:
      update-strategy: not_null
      id-type: auto
logging:
  level:
    com.hmall: debug
  pattern:
    dateformat: HH:mm:ss:SSS
  file:
    path: "logs/${spring.application.name}"
knife4j:
  enable: true
  openapi:
    title: 购物车服务接口文档
    description: "购物车服务接口文档"
    url: https://www.itcast.cn
    version: v1.0.0
    group:
      default:
        group-name: default
        api-rule: package
        api-rule-resources:
          - com.hmall.cart.controller
```

另外两个环境配置文件也复制过来

##### 2.2.4.5 启动微服务

直接运行启动类启动微服务。

然后启动`CartApplication`，访问swagger文档页面：[http://localhost:8082/doc.html](http://localhost:8082/doc.html)

我们测试其中的`查询我的购物车列表`接口：

![这张图片是商品服务接口文档的页面，具体展示了查询购物车列表的相关接口信息，页面左侧菜单栏选中了“GET 查询购物车列表”这一选项，对应接口的请求方式为GET，接口路径为/carts，请求数据类型为application/x-www-form-urlencoded，响应数据类型标注为\["*/*"\]，页面还显示该接口的请求参数区域当前无数据，与上下文提及的测试该“查询我的购物车列表”接口的内容相对应，属于该接口测试前的文档展示页面。](https://feishu.cn/file/KNhJb253dov77fxsIumcsJ8lnZe)



下边继续测试接口，无需填写参数，直接访问：

![图片展示的是在Postman中测试购物车服务接口的界面。上方显示请求方法为GET，URL为/carts。下方“请求参数”区域有“x-www-form-urlencoded”“form-data”“raw”三个选项，当前选中“x-www-form-urlencoded”。响应内容区域显示了查询购物车列表的JSON数据，其中newPrice、stock、status属性值分别为null、10、0。图片右下角有红色箭头标注，提示这三个属性值都不是最新值（后俩是默认值）。该图片与上下文介绍的测试购物车服务接口内容相关，直观呈现了接口测试结果。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/Am3Ub6wdGoeAKoxmclZcosWvn9c.png)

我们注意到，其中与商品newPrice属性值都为空，其他俩stock和status是vo类里属性的默认值！这就是因为刚才我们注释掉了查询购物车时，查询商品信息的相关代码。

那么，我们该如何在`cart-service`服务中实现对`item-service`服务的查询呢？



#### 2.2.5. 小结

如何创建微服务工程？

1. 首先确定Spring Cloud、Spring Boot版本，在父工程pom.xml中设定版本号。
2. 每个微服务创建独立的数据库。
3. 每个微服务创建Spring Boot工程，继承父工程，并在application.yaml配置端口号、服务名、数据库连接池、日志、swagger文件等配置信息。
4. 每个微服务工程的最少的项目包结构包括：controller：web接口service：服务接口domain：模型类mapper：数据库mapper





### 2.3 RestTemplate

1. 介绍远程调用技术以及为什么需要它

   <whiteboard token="PSucwJzt4hNjuXb6Ckxc02Ptn5d"></whiteboard>
2. RestTemplate的具体使用

<callout emoji="🥇">
1.写一个Spring配置
2.向Spring容器注入一个bean对象
3.使用RestTemplate
</callout>

#### 2.3.1. Java远程调用技术介绍

我们发现一个问题：在购物车业务中需要查询商品信息，但商品信息查询的逻辑全部迁移到了`item-service`服务，导致我们无法查询，查询端口信息的代码已经屏蔽：

![图片展示的是`CartServiceImpl`类中`handleCartItems`方法的部分代码。代码中，首先获取商品id，然后调用`itemService.queryItemByIds`查询商品。其中，`itemIds`是通过`vos.stream().map(CartVO::getItemId).collect(Collectors.toSet())`获取的，`items`是通过`itemService.queryItemByIds(itemIds)`查询得到的。若查询结果为空，则返回。该图片与上下文介绍的微服务架构中商品信息查询问题相关，展示了通过远程调用技术查询商品信息的代码实现。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/KSMHbUPyuoq99lxAGepcmtZJnLf.png)

最终结果就是查询到的购物车数据不完整，因此要想解决这个问题就需要通过远程调用技术。

一个微服务架构的项目会包括很多的微服务，一次业务操作会由多个微服务互相协作完成，比如：一次下单操作的执行流程如下：

![图片展示了订单服务执行流程。用户请求订单服务提交订单后，订单服务在数据库创建订单，接着分别请求账户服务扣减余额、库存服务扣减库存，最后三个服务均在各自数据库操作。该图与上下文紧密相关，直观呈现了提交订单这一业务操作中，订单服务与账户、库存服务协作完成的执行流程，帮助理解微服务架构下一次业务操作的协作关系。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/XHDubNmsco1UHIxdAlTczvn2n3b.png)

执行流程如下：

1.用户请求订单服务提交订单

2.订单服务在数据库创建订单

3.订单服务请求账户服务扣减余额

4.订单服务请求库存服务扣减库存

从以上执行流程可以看出提交订单是通过订单服务、账户服务、库存服务三个微服务共同协作完成的。

服务之间进行交互是通过网络协议进行通信，在Spring Cloud中是通过HTTP协议进行通信。

拿上图举例：

订单服务提供“提交订单接口”供手机客户端调用，手机为客户端，订单服务为服务端。

账户服务提供“扣减余额接口”供订单服务调用，订单服务为客户端，账户服务为服务端。

库存服务提供“扣减库存接口”供订单服务调用，订单服务为客户端，库存服务为服务端。

**总结：调用方为客户端，被调方为服务端，客户端为消费者，服务端为生产者。**



AI：在Java中客户端与服务端远程调用（Remote Procedure Call, RPC）的技术有哪些？

常用的如下：

1. **RMI (Remote Method Invocation)**

- RMI 是 Java 提供的一种标准远程过程调用机制。
- 客户端可以像调用本地对象一样调用远程对象的方法。
- RMI 使用 Java 序列化机制来传输对象。

2.**REST (Representational State Transfer)**

- RESTful 服务使用 HTTP 协议来访问资源。
- 它通常使用 JSON 或 XML 格式来传输数据。
- REST 是一种轻量级的架构风格，广泛应用于现代 Web 服务。

3.**Apache Dubbo**

- Dubbo 是一个高性能、轻量级的微服务框架。
- Dubbo 支持多种协议远程调用，如 Dubbo 协议、HTTP 协议等。



**不管哪种技术它们的底层还是通过TCP、UDP网络协议进行通信。**



我们现在要在**Spring Boot**工程中实现远程调用，我们通过AI看下在Spring Boot中有哪些远程调用技术

AI：Spring Boot中远程调用的技术

摘要信息如下：

1. **Feign**

   - Feign 是一个声明式的HTTP客户端，它简化了HTTP请求的发送。
   - 使用Feign，你可以通过定义接口来创建客户端，Feign会自动生成调用逻辑。
   - Feign支持多种序列化库，如Jackson和Gson。
   - Feign可以与Spring Cloud Discovery Client集成，实现服务发现。
2. **RestTemplate**

   - RestTemplate 是Spring提供的一个模板类，用于简化HTTP请求的发送。
   - 通过RestTemplate，你可以轻松地发起GET、POST等请求，并处理响应。
   - RestTemplate支持多种数据格式，如JSON和XML。
   - RestTemplate常用于简单的HTTP调用，但在Spring Cloud中也有广泛应用。

RestTemplate不依赖SpringCloud框架，Feign依赖SpringCloud框架。



AI: RestTemplate远程调用例子

其中提供了大量的方法，方便我们发送Http请求，例如：

![图片展示了RestTemplate类的代码界面，其中列出了该类的多个方法。如RestTemplate()构造函数，delete、getForEntity、getForObject等方法用于发送Http请求，postForEntity、put等方法用于发送Post、Put请求，doExecute、exchange等方法用于执行Http操作。这些方法与上下文介绍的RestTemplate不依赖SpringCloud框架，可发送多种Http请求，且可使用exchange方法构造复杂请求的内容相呼应，直观呈现了其方法功能。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/TqgWbmUSwoGFeFxOIj3cSvnGntg.png)

可以看到常见的Get、Post、Put、Delete请求都支持，如果请求参数比较复杂，还可以使用exchange方法来构造请求。



#### 2.3.2.RestTemplate实现远程调用

##### 2.3.2.1 明确需求

刚开始学习我们用一个简单点的例子，在购物车中需要显示商品信息，而商品信息在item-service服务，因此要想在购物车拿到商品信息就需要购物车服务远程调用商品服务去获取商品信息，流程图如下：

<whiteboard token="UfIcwpaiVhxF7UbT5jmcsX0TnAD"></whiteboard>

代码中需要变化的就是这一步：

![图片展示的是购物车服务中处理购物车项的代码片段。其中，将本地方法调用改为远程跨服务调用的代码部分被红框突出显示，即“List<Item> items = itemService.listByIds(itemIds);”这行代码。上下文提到在购物车中显示商品信息时，需远程调用商品服务获取商品信息，此图片正是对应代码示例，直观呈现了代码中需进行远程调用的部分，帮助理解远程调用在实际代码中的位置和形式。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/CKwnbJVUEoGENmxgOxQcSdIcnzc.png)

##### 2.3.2.2 注入RestTemplate



<callout emoji="🎼">
1.一个配置类
2.注入一个RestTemplate的bean实例
3.使用
</callout>

现在需求明确，在购物车服务中远程调用商品服务查询商品信息。

首先在商品服务提供一个RESTful接口，传入商品id查询商品信息后返回。

然后在购物车服务用RestTemplate去请求商品服务的商品查询接口。



我们在`cart-service`服务中定义一个配置类：

![这张图片展示的是cart-service微服务工程的源码结构，在com.hmall.cart包下，标注了名为config的目录，该目录里有一个名为RemoteCallConfig的类，这正是文档中提到的用于定义配置类的文件，这个配置类的作用是将RestTemplate注册为Bean，为后续在CartServiceImpl中实现远程调用item-service的操作做准备，符合文档里构建微服务工程中RestTemplate相关配置的内容。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/Xy2cbygVDoPqXqxaQEOcmvVDnph.png)

先将RestTemplate注册为一个Bean：

```JavaScript
package com.hmall.cart.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

/**
 * @author Mr.M
 * @version 1.0
 * @description RestTemplate配置类
 * @date 2024/8/2 19:46
 */

@Configuration
public class RemoteCallConfig {

 @Bean
 public RestTemplate restTemplate() {
  return new RestTemplate();
 }
}
```

接下来，我们修改`cart-service`中的`com.hmall.cart.service.impl.CartServiceImpl`的`handleCartItems`方法，发送http请求到`item-service`：

首先在`CartServiceImpl`中注入RestTemplate。

```JavaScript
...
public class CartServiceImpl extends ServiceImpl<CartMapper, Cart> implements ICartService {

//    private final IItemService itemService;

    private final RestTemplate restTemplate;
    ...
```

##### 2.3.2.3 实现远程调用

接下来修改代码，使用RestTemplate实现远程调用，使用RestTemplate进行远程调用的代码如下：

![图片展示了`CartServiceImpl`中`handleCartItems`方法的代码。关键部分是使用`RestTemplate`发送HTTP请求的代码，请求路径为`String.format("http://localhost:8881/items?ids=%s", CollUtils.join(itemIds, conjunction: ", "))`，其中`itemIds`是商品ID集合，`itemDto\[\].class`指定响应数据类型。该图片与上下文紧密相关，是对使用`RestTemplate`实现远程调用时发送HTTP请求代码的展示，直观呈现了请求路径等关键信息。](https://feishu.cn/file/H3qbbJP52ozlHixPCfIcjQrMnGb)

可以看到，利用RestTemplate发送http请求与前端ajax发送请求非常相似，都包含四部分信息：

- 请求方式
- 请求路径
- 请求参数（本案例中，为?参数）
- 返回值类型

##### 2.3.2.4 测试

好了，现在重启`cart-service`，再次测试查询我的购物车列表接口：

![这张图片展示的是网页端的商品服务接口测试页面，对应文档中测试`cart-service`调用`item-service`查询购物车接口的环节。页面左侧选中了“GET 查询购物车列表”接口，右侧页面显示该接口为GET请求，地址为/carts，参数类型为x-www-form-urlencoded，请求响应代码为200，耗时209毫秒，响应数据中包含了商品相关信息，其中商品价格被红色框选中标注，以此体现服务消费者`cart-service`已成功调用服务提供者`item-service`的接口，完成了商品数据的查询获取。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/KYW5bp4Y6oR0vexFuZ1cvE67nde.png)

可以发现，所有商品相关数据都已经查询到了。



在这个过程中，`item-service`提供了查询接口，`cart-service`利用Http请求调用该接口。因此`item-service`可以称为服务的提供者，而`cart-service`则称为服务的消费者或服务调用者。

#### 2.3.3. 小结

微服务之间的远程调用被称为RPC，即远程过程调用。RPC的实现方式有很多，比如：

- 基于Http协议
- 基于Dubbo协议



我们课堂中使用的是Http方式，这种方式不关心服务提供者的具体技术实现，只要对外暴露Http接口即可，更符合微服务的需要。

Java使用Spring提供的RestTemplate向服务端接口发送http请求，基本步骤如下：

- 注册RestTemplate到Spring容器
- 调用RestTemplate的API发送请求，常见方法有：

  - getForObject：发送Get请求并返回指定类型对象
  - PostForObject：发送Post请求并返回指定类型对象
  - put：发送PUT请求
  - delete：发送Delete请求
  - exchange：发送任意类型请求，返回ResponseEntity
- 解析响应结果



#### 2.3.4 面试题

说几个Java远程调用的技术？你们项目用的什么技术？



总结：

<whiteboard token="KiEgwvX5MhGcW8bqOojchHVynSe"></whiteboard>



## 3 服务注册和发现

<callout emoji="📌">
1.为什么需要负载均衡
2.nginx和nacos又有什么区别
</callout>

### 3.1. Nginx负载均衡

#### 3.1.1 问题说明

在上一章我们实现了微服务的构建以及微服务之间的远程调用。不过这种手动发送Http请求的方式存在一些问题。

试想一下，假如商品微服务被调用较多，为了应对更高的并发，我们进行了多实例部署，如图：

![这张图对应文档中商品微服务多实例部署的场景，直观呈现了cart-service调用item-service的情况：左侧蓝色模块为cart-service，箭头指向右侧三个绿色模块，分别为端口8081、8082、8083的item-service实例，同时标注出cart-service向其中一个item-service实例发送的请求内容，以此展示多实例部署后，cart-service需调用不同item-service实例地址的问题背景，呼应上下文提出的“多实例部署后cart-service如何确定调用地址”的疑问。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/PddObpISjoOeC2xErbpcPnAKnEe.png)

此时，每个`item-service`的实例其IP或端口不同，问题来了：

- item-service这么多实例，cart-service如何知道每一个实例的地址？
- http请求要写url地址，`cart-service`服务到底该调用哪个实例呢？
- 如果在运行过程中，某一个`item-service`实例宕机，`cart-service`依然在调用该怎么办？
- 如果并发太高，`item-service`临时多部署了N台实例，`cart-service`如何知道新实例的地址？

#### 3.1.2 Nginx负载均衡



<callout emoji="🏆">
1. nginx新增配置
2. 重启nginx
3. 修改CartServiceImpl
</callout>

如何解决上边的问题？

要解决上边的问题需要引入负载均衡器，比如：Nginx就提供负载均衡的功能，负载均衡器是一种网络设备或软件组件，用于在多台服务器之间分发网络流量。这有助于提高应用程序的可用性和响应速度，同时也能增加系统的容错能力。

Nginx 不仅是一个高性能的 Web 服务器，也是一个强大的反向代理，可以做为负载均衡器。

它可以支持多种负载均衡算法，包括轮询、最少连接、IP 哈希：

1. 轮询 (默认)

   - 按照顺序将请求分发给不同的服务器。
2. 最少连接

   - 将请求发送到当前活跃连接数最少的服务器。
3. IP 哈希

   - 根据客户端 IP 地址进行哈希计算，使得来自同一客户端的请求始终被路由到同一台服务器。



通过nginx实现负载均衡的结构图如下：



<whiteboard token="T4u0wCZNfh3rKsbm2vYcLtYKnbb"></whiteboard>

在nginx的配置方法也非常简单，只要配置上游服务器和具体的代理位置即可，大家可作为参考

```JavaScript
http {
    upstream item_services {
        server 127.0.0.1:8081 weight=3;  # 分配较高权重
        server 127.0.0.1:7081 weight=2;  # 分配中等权重
        server 127.0.0.1:6081 weight=1;  # 分配较低权重
    }

     server {
        listen       80;
        server_name  localhost;
        # 指定前端项目所在的位置
        location / {
            root html;
        }

        location /items {  # 这里可以根据需要调整路径前缀
            proxy_pass http://item_services;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            # 其他可选配置
            proxy_redirect off;
            proxy_buffering off;
            proxy_read_timeout 90;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    
    ////省略其他
}
```

说明：

1. upstream 块定义了一个名为 `item_services` 的服务器组，包含三个服务器实例。你可以通过修改 `server` 行中的地址和端口来指向实际的服务实例。
2. server 块定义了 Nginx 的监听端口和服务名称。
3. location 块指定了路径 `/items/` 下的所有请求将被代理到 `item_services` 上游组。
4. proxy_pass 指令指定了代理目标。
5. proxy_set_header 指令设置了转发给后端服务的 HTTP 头。



这种方案在我们学习nginx就接触到了，这是一种服务端负载均衡的做法，即客户端（cart-service）请求nginx(服务端)，由nginx负载均衡器完成将请求转发到具体的服务实例。

如果增加了服务实例可以在nginx的upstream 上配置服务实例地址即可。



下边进行测试

首先在nginx的nginx.conf配置文件中配置upstream 和虚拟目录

![图片展示了Nginx负载均衡配置文件nginx.conf的部分内容。关键部分用红色框标注，包括upstream配置，将127.0.0.1:7080和127.0.0.1:7081分别分配较高权重和中等权重；以及location配置，将请求路径为/items的请求代理到http://item_services，同时设置相关代理头信息。该图片与上下文紧密相关，是进行Nginx负载均衡测试时配置文件中配置内容的示例展示。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/FrT0bV7klofn6bxSMDmcfMtQn6b.png)

cmd进入nginx目录，输入下边的命令加载配置：

![图片展示的是在nginx目录下执行命令加载配置的操作界面。命令为“nginx.exe -s reload”，执行后显示“hmall-nginx>”，表明命令执行成功。该图片与文档中“cmd进入nginx目录，输入下边的命令加载配置”内容对应，是进行Nginx负载均衡测试前，加载配置文件以进行后续测试操作的步骤展示。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/DnQbbyuBjotTbEx0HkXcrFCxnOf.png)

然后在idea中启动多个商品服务：

注意：为了避免多个服务端口冲突这里需要指定端口

产生复制一个启动配置

![这是IDEA的“Run/Debug Configurations”界面截图，展示用于启动微服务的配置界面。左侧列表中，“Spring Boot”分组下已存在“CartServiceApplication”与“ItemServiceApplication”两个服务配置项；红色箭头指向左侧的复制按钮，该操作对应文档中提到的“为避免多个服务端口冲突，复制启动配置”的步骤，用于生成新的服务启动配置，以此实现后续修改端口启动多个商品服务实例的需求。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/Pm9ubd9i0oOh8NxtkeccU9sAnpf.png)

然后再修改启动参数，添加启动参数

![图片展示了IntelliJ IDEA中ItemServiceApplication的Run/Debug配置界面。在“Build and run”区域，有“Modify options”按钮被红色箭头指向。该图片对应文档中“修改启动参数，添加启动参数”步骤，用于说明在启动参数中传入server.port=7081的操作，即在“Modify options”下进行相关配置，以实现启动多个商品服务实例和购物车服务实例的目的。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/EMLPbHgknoR0r6xUTn1c2WMxnEp.png)



![图片展示了IntelliJ IDEA中运行配置界面。在“Active profiles”部分，有多个配置选项，如“Enable debug output”“Hide banner”等，部分选项右侧有快捷键提示。在“Operating System”部分，有“Allow multiple instances”“Working directory”“Environment variables”等选项，同样有快捷键提示。在“Java”部分，“Use classpath of module”选项被勾选，其右侧也有快捷键提示。图片下方突出显示了“Add VM options”选项，其右侧是快捷键“Alt+V”。该图片与文档中在idea中启动多个商品服务，修改启动参数，添加启动参数的操作步骤相关，用于说明添加VM选项的操作位置。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/Jwg2bQXyhor3DLxkpN3cexyvnKc.png)

在启动参数中传入：server.port=7081如下图：

![这张图片展示了IntelliJ IDEA的运行/调试配置界面，对应文中修改启动参数指定端口的操作。界面左侧列出了Spring Boot相关的启动配置，其中ItemServiceApplication (1)为当前选中项，右侧“Build and run”区域内，标注出的参数框中填入了启动参数“-Dserver.port=7081”，该参数正是文中提到的用于避免多服务端口冲突的端口设置，以此实现运行多个商品服务实例时的端口区分。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/GpeKbTxqPoxFkhxNCrscPCAZnVd.png)

启动成功如下：

两个商品服务实例，一个购物车服务实例。

![这张图片展示了Spring Boot项目的运行实例情况，明确显示处于运行状态。其中包含一个端口为8082的CartServiceApplication、一个端口为8081的ItemServiceApplication，还有两个ItemServiceApplication实例，另一个实例的端口为7081。该图对应文档中启动多个商品服务的相关内容，呈现了修改启动参数指定端口后，成功启动多个商品服务实例的运行结果，符合文档中为避免端口冲突指定端口启动多个商品服务的操作要求。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/PK4kbZLP1om4Yzxyv00cijPynZi.png)

然后修改购物车服务中远程调用商品服务地址为nginx虚拟目录的地址

![这张图片展示了一段Java代码，是购物车服务中远程调用商品服务的相关逻辑，代码里定义了handleCartItems方法用于处理购物车条目，通过@Autowired注解注入了RestTemplate实例，在调用商品服务的代码行中，设置的原访问地址为“http://localhost:80/items?ids=%s”，这一地址为后续需要修改为Nginx虚拟目录地址的内容，体现了将远程调用的商品服务地址配置为本地服务端口地址的原有实现。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/RKUlbkN2go2rAwxDyKKc6EPNnkh.png)

重启购物车服务。

重新测试查询购物车接口，观察两个商品服务的控制台，购物车服务通过nginx请求商品服务时通过轮询算法请求商品服务





#### 3.2.3 面试题

如何使用nginx实现负载均衡？



Nginx是反向代理，为什么叫反向代理？



### 3.2. 服务注册与发现流程

#### 3.3.1 区分客户端与服务端负载均衡

刚才使用nginx实现负载均衡解决了我们面临的问题。

nginx作为服务端，客户端请求nginx服务端，通过Nginx负载均衡器将请求转发到不同的后端服务，这是一种服务端负载均衡。

**服务端负载均衡**是客户端请求到Nginx服务端由Nginx实现负载均衡。

但是是不是不好用，不灵活，**比如每多一个实例，都需要去改nginx的配置？**接下来我们再讲一种客户端负载均衡的做法来解决我们面临的问题。

**客户端负载均衡**是客户端在发起调用时自己实现了负载均衡的作用，自己通过负载均衡算法找到一个目标服务进行调用。

如下图：

购物车服务自己通过负载均衡器挑选一个目标地址进行远程调用。

<whiteboard token="Wd2Xwoq7EhpqkPbawficDrrUnue"></whiteboard>



#### 3.3.2 服务注册与发现流程

<callout emoji="❤️">
1.学习nacos
2.安装nacos
3.服务引用它
</callout>

接下来我们讲的服务注册与发现流程正是用于实现客户端负载均衡。

首先引入注册中心的概念。

在微服务远程调用的过程中，包括两个角色：

- 服务提供者：提供接口供其它微服务访问，比如`item-service`
- 服务消费者：调用其它微服务提供的接口，比如`cart-service`

在大型微服务项目中，服务提供者的数量会非常多，为了管理服务提供者就引入了**注册中心**的概念。注册中心、服务提供者、服务消费者三者间关系如下：



![这张图清晰展示了微服务场景下服务注册与发现的流程，图中包含三类核心角色：注册中心、服务提供者、服务调用者。服务提供者item-service包含localhost:8081、localhost:8082、localhost:8083三个实例，会向注册中心发起注册服务请求，还会定期向注册中心发送心跳检测以维持存活状态。注册中心向服务调用者推送服务实例的变更信息，服务调用者会向注册中心发起服务订阅，获取item-service的实例信息后，通过负载均衡对服务提供者发起远程调用，以此实现微服务间的高效远程交互。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/WEDobq2xwoEVrpx4ZpScEZUen5b.jpg)

三者的分工如下：

1 **注册中心**：提供服务注册接口，接收服务注册请求，保存服务实例的信息。

目前开源的注册中心中间件有很多，国内比较常见的有：

- Eureka：Netflix公司出品，目前被集成在SpringCloud当中，一般用于Java应用
- **Nacos**：Alibaba公司出品，目前被集成在SpringCloudAlibaba中，一般用于Java应用，我们项目使用Nacos。
- Consul：HashiCorp公司出品，目前集成在SPringCloud中，不限制微服务语言

2 **服务提供者**：服务接口提供方，请求注册中心将服务信息注册到注册中心。

3 **服务调用者**：远程调用的客户端，请求注册中心查询服务地址，通过负载均衡选取目标服务地址进行远程调用。



服务注册与发现流程如下：

- 服务启动时就会注册自己的服务信息（服务名、IP、端口）到注册中心
- 调用者可以从注册中心订阅想要的服务，获取服务对应的实例列表（1个服务可能多实例部署）
- 调用者作为客户端自己通过负载均衡算法挑选一个服务提供者实例进行远程调用，即客户端负载均衡
- 调用者向该实例发起远程调用



当服务提供者的实例宕机或者启动新实例时，调用者如何得知呢？

- 服务提供者会定期向注册中心发送请求，报告自己的健康状态（心跳请求）
- 当注册中心长时间收不到提供者的心跳时，会认为该实例宕机，将其从服务的实例列表中剔除
- 当服务有新实例启动时，会发送注册服务请求，其信息会被记录在注册中心的服务实例列表
- 当注册中心服务列表变更时，会主动通知微服务，更新本地服务列表



#### 3.3.3 面试题

说一下服务注册与发现流程？





### 3.3.Nacos安装配置

#### 3.3.1 介绍

接下来我们就去实现整个服务注册与发现的流程。

Nacos是国内产品，中文文档比较丰富，而且同时具备**配置管理**功能（后面会学习），因此在国内使用较多，课堂中我们会Nacos为例来学习。



官方网站如下：

[https://nacos.io/zh-cn/docs/v2/what-is-nacos.html](https://nacos.io/zh-cn/docs/v2/what-is-nacos.html)

![图片展示的是Nacos文档首页界面。上方有“首页”“文档”“企业版NACOS”“免费试用”“NACOS架构与原理”“博客”“社区”“控制台特例”等导航栏。蓝色背景区域左侧有“Nacos 文档”标识，右侧有“什么是 Nacos”“概览”等内容。左侧侧边栏有“Nacos”“Nacos是什么？”“Nacos简介”“概念”“架构”等选项。该图片与文档中介绍Nacos安装配置上下文相关，直观呈现了Nacos的官方网站界面。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/FFTxbxnVMovIrmxw8CecIMYKnId.png)



#### 3.3.2 安装数据库

我们基于Docker来部署Nacos的注册中心，首先我们要准备MySQL数据库表，用来存储Nacos的数据。由于是Docker部署，需要用到数据库（虚拟机已经提供）：

![图片展示了Nacos数据库中tables表下的部分表结构。其中包含12个表，分别为config_info、config_info_aggr、config_info_beta、config_info_tag、config_tags_relation、group_capacity、his_config_info、permissions、roles、tenant_capacity、tenant_info和users。这些表与Nacos服务注册与发现相关，是Nacos数据库中用于存储配置信息、权限、角色等数据的关键表。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/DNH2bvuWWosiwZxtv7IcTho3nPe.png)



#### 3.3.3 创建容器

然后，找到课前资料下的nacos文件夹和nacos.2.4.tar，全部上传到/root下：

nacos.2.4.tar是nacos的镜像文件，nacos文件夹下是nacos配置文件。

![这张图片展示的是一个Windows系统的文件资源管理器窗口，当前路径为“新加卷 (D:) > 课程资料 > 服务框架 > day03-微服务01 > 资料”，窗口内列出了该路径下的文件与文件夹，其中高亮选中的是名为nacos的文件夹，还有hmall-nginx文件夹，以及hm-cart.sql、hm-item.sql等多个SQL源文件，另有一个JMX格式的“黑马商城测试.jmx”文件，该资料路径与微服务学习中Nacos安装配置的课程内容相关，用于对应安装操作中需上传至虚拟机的相关资料文件。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/DRHgbtzNIoOUUUxUTF5cDZiFnXf.png)

课程资料中的`nacos/custom.env`需要上传到虚拟机/root目录下，在该文件中，注意：有一个MYSQL_SERVICE_HOST也就是mysql地址为虚拟机IP地址：

```Plain Text
PREFER_HOST_MODE=hostname
MODE=standalone
SPRING_DATASOURCE_PLATFORM=mysql
MYSQL_SERVICE_HOST=192.168.101.68
MYSQL_SERVICE_DB_NAME=nacos
MYSQL_SERVICE_PORT=3306
MYSQL_SERVICE_USER=root
MYSQL_SERVICE_PASSWORD=mysql
MYSQL_SERVICE_DB_PARAM=characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai
```

进入虚拟机的`/root`目录，导入nacos.tar镜像文件：

```JavaScript
docker load -i nacos.2.4.tar
```



进入root目录，然后执行下面的docker命令：

先执行删除旧容器命令

```Plain Text
docker rm -f nacos2.4.0.1 nacos nacos2.1
```

在执行创建新容器命令(**要在custom.env文件这一级目录执行启动命令**)

```PowerShell
docker run -d \
--name nacos \
--env-file custom.env \
-p 8848:8848 \
-p 9848:9848 \
-p 9849:9849 \
--restart=always \
nacos/nacos-server:v2.4.0.1
```

#### 3.3.4 访问nacos

启动完成后，访问下面地址：[http://192.168.101.68:8848/nacos/](http://192.168.101.68:8848/nacos/)，注意将`192.168.101.68`替换为你自己的虚拟机IP地址。

进入首页：

![图片展示的是Nacos 2.4.0配置管理页面。页面左侧有“配置管理”等选项，右侧上方提示当前集群未开启授权。中间区域有“创建配置”按钮，以及Data ID、Group等输入框，还有“查询”“高级查询”“导入配置”按钮。下方显示查询到0条满足要求的配置，且有“操作”列。该页面与文档中Nacos安装配置上下文相关，用于展示Nacos配置管理界面，辅助说明Nacos安装配置操作。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/O38yb0dxNouOWqxFvI2chwinnkd.png)

注意：2.4版本开始nacos默认不进行认证直接进入首页，以前的版本首次访问会跳转到登录页，**账号密码都是nacos**

![这张图片是Nacos的登录页面，页面左上角显示Nacos的品牌标识，下方标注了Nacos是用于构建云原生应用的易上手的动态服务发现、配置和服务管理平台。页面右侧为登录区域，标题为“登录”，还标注了提示文字“内部系统，不可暴露到公网”，配有“请输入用户名”“请输入密码”的输入框，以及蓝色的“提交”按钮。页面顶部有“首页”“文档”“博客”“社区”等导航选项，结合上下文可知，该页面是2.4版本之前的Nacos首次访问时需要进入的登录界面，对应文档中关于Nacos安装配置里首次访问的登录说明。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/KbMcb5663oufEYxHpWSc3ibHn0e.png)





#### 3.3.5 小结

Nacos安装步骤小结：

1. 准备nacos数据库
2. 准备nacos 镜像 及配置(配置需要修改为自己虚拟机的ip)
3. docker load -i nacos.2.4.tar 加载镜像
4. 运行docker 启动命令



### 3.4. 服务注册



<whiteboard token="Qc0iwAqddh5H1zbVFCJckn45nhg"></whiteboard>

接下来，我们把`item-service`注册到Nacos，步骤如下：

- 引入依赖
- 配置Nacos地址
- 重启



#### 3.4.1 添加依赖

在父工程中已经添加Spring Cloud及Spring Cloud Alibaba的版本约束，如下

```JavaScript
<!--spring cloud-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>${spring-cloud.version}</version>
    <!--   type=pom ,scope=import 表示继承 和 parent 作用一样             -->
    <type>pom</type>
    <scope>import</scope>
</dependency>
<!--spring cloud alibaba-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>${spring-cloud-alibaba.version}</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

因为我们安装的是2.4.0版本的nacos，S**pring Boot默认使用的Nacos比该版本低**，所以需要在父pom.xml中约定nacos-client版本，如下（复制到父工程的dependencies标签中）：

```JavaScript
<dependencies>
    <!-- lombok 管理 -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>${org.projectlombok.version}</version>
    </dependency>
    <!--单元测试-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.alibaba.nacos</groupId>
        <artifactId>nacos-client</artifactId>
        <version>2.4.0</version>
    </dependency>
</dependencies>
```

在`item-service`的`pom.xml`中添加依赖：

```XML
<!--nacos 服务注册发现-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

```



#### 3.4.2 配置Nacos

在`item-service`的`application.yml`中添加nacos地址配置：

```YAML
spring:
  application:
    name: item-service # 服务名称
  cloud:
    nacos:
      server-addr: 192.168.101.68:8848 # nacos地址
```

注意：修改nacos的ip和端口。



#### 3.4.3 启动服务实例

启动两个商品服务实例。

![图片展示了IntelliJ IDEA中运行的`ItemServiceApplication`服务实例信息。左侧有“Spring Boot”和“Running”分类，右侧显示了两个实例，分别是`ItemServiceApplication:8081/`和`ItemServiceApplication (1):7081/`，其中`ItemServiceApplication:8081/`被蓝色高亮突出显示。该图片与文档中“3.4.4查看服务注册信息”部分对应，用于说明在微服务启动后，通过IntelliJ IDEA可查看到服务注册成功的实例信息，辅助理解服务注册成功后的日志观察结果。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/M0jvby5Y7odU6qxPIU5cMkGcnje.png)

启动过程中观察日志，发现如下日志则表示注册成功

```JavaScript
Success to connect to server [192.168.101.68:8848] on start up, connectionId ...
```

#### 3.4.4 查看服务注册信息

访问nacos控制台，可以发现服务注册成功：

![图片展示的是Nacos控制台的服务列表页面。页面左侧有“服务管理”下的“服务列表”选项被选中。右侧显示了服务列表，其中“item-service”服务被红框突出显示，其分组名称为“DEFAULT_GROUP”，集群数目、实例数、健康实例数均为1，触发保护阈值为false。该图片与上文“查看服务注册信息”内容相关，直观呈现了在Nacos控制台中服务注册成功后的服务注册信息情况。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/KBt4b3uL9opUmRx7BObce7HCnnd.png)

点击详情，可以查看到`item-service`服务的两个实例信息：

![这是Nacos控制台的服务详情页面截图，展示了已注册成功的item-service服务信息，该服务隶属于DEFAULT_GROUP，服务路由类型为none。页面下方的实例列表清晰显示了该服务的两个实例信息，两个实例的IP均为192.168.101.1，端口分别为8081和7081，权重均为1，健康状态、临时实例状态均为true，元数据内容为preserved.register.source=SPRING_CLOUD，列表右侧提供了编辑、下线操作选项，直观呈现了服务注册成功后的实例部署情况，与文档中“查看服务注册信息”的内容相呼应，对应文档提及的item-service服务两个注册实例。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/BTe6bPUu3oQWtLxQJGKcfgLFnLg.png)



#### 3.4.5 小结

服务注册过程：

1. 在微服务添加spring-cloud-starter-alibaba-nacos-discovery 依赖。
2. `application.yml`中添加nacos地址配置，微服务根据配置连接nacos进行注册。
3. 可通过nacos的服务列表查询服务注册信息。



### 3.5. 服务发现

服务的消费者要去nacos订阅服务，这个过程就是服务发现，步骤如下：

- 引入依赖
- 配置Nacos地址
- 发现并调用服务

  - 修改RestTemplate



#### 3.5.1 引入依赖

服务发现除了要引入nacos依赖以外，由于还需要负载均衡，因此要引入SpringCloud提供的LoadBalancer依赖。

我们在`cart-service`中的`pom.xml`中添加下面的依赖：

```XML
<!--nacos 服务注册发现-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<!--负载均衡器-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

可以发现，这里Nacos的依赖于服务注册时一致，这个依赖中同时包含了服务注册和发现的功能。因为任何一个微服务都可以调用别人，也可以被别人调用，即可以是调用者，也可以是提供者。

因此，等一会儿`cart-service`启动，同样会注册到Nacos

#### 3.5.2 配置Nacos地址

在`cart-service`的`application.yml`中添加nacos地址配置：

```YAML
spring:
  cloud:
    nacos:
      server-addr: 192.168.101.68:8848
```



#### 3.5.3 服务发现并调用服务

接下来，服务调用者`cart-service`就可以去订阅`item-service`服务了，它的底层使用了Spring Cloud Loadbalancer组件实现客户端负载均衡，常见的负载均衡算法有：

- 随机
- 轮询
- IP的hash
- 最近最少访问
- ...

接下来，我们就可以对原来的远程调用做修改了，之前调用时我们需要写死服务提供者的IP和端口，

Spring Cloud Loadbalancer 提供了现成的客户端负载均衡器可以非常方便的实现客户端负载均衡。

修改RemoteCallConfig类，使用@LoadBalanced注解标识RestTemplate，如下：

```JavaScript
@Configuration
public class RemoteCallConfig {

 @Bean
 @LoadBalanced
 public RestTemplate restTemplate() {
  return new RestTemplate();
 }
}
```

修改代码，屏蔽使用discoveryClient从注册中心查询服务列表以及随机选择服务实例的代码，RestTemplate在远程调用时指定服务名“item-service”，如下：

![这张图片展示的是一段Java代码，属于实现服务发现并调用服务的相关内容。代码中定义了handleCartItems方法，方法内先通过流操作获取CartVO列表中的商品ID集合；随后使用带有@LoadBalanced标识的RestTemplate，拼接带有服务名“item-service”的URL，调用getForObject方法查询商品，得到ItemDTO数组后转为列表，最后进行商品列表是否为空的判断并做出对应返回处理。该代码对应文档中修改代码、指定服务名“item-service”完成远程调用的相关操作部分，是实现服务发现与负载均衡调用的具体代码示例。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/QfXKbbfHJoD96IxvAOScwbqjnag.png)

重启cart-service，测试查询购物车列表接口，功能正常，并且连续请求接口通过观察两个商品服务实例的控制台都输出商品查询的日志，说明实现了负载均衡。



#### 3.5.4 小结

服务发现流程：

1. 在服务调用方引入依赖spring-cloud-starter-alibaba-nacos-discovery
2. 在服务调用方配置Nacos地址
3. 发现并调用服务

使用@LoadBalanced注解标识RestTemplate，RestTemplate在远程调用时指定目标服务名即可。



### 3.6. Spring Cloud Loadbalancer 原理-根据截图里的内容，断点跟下源码

<whiteboard token="Maxtw2YQohIMYebbumxcheSLnFf"></whiteboard>

#### 3.6.1 源码解析

为什么我们只输入了service名称就可以访问了呢？之前还要获取ip和端口。 显然有人帮我们根据service名称，获取到了服务实例的ip和端口。这就是 @LoadBalanced注解起到的作用。

@LoadBalanced注解是如何实现负载均衡呢？

首先在创建RestTemplate的bean时添加@LoadBalanced注解，LoadBalancerAutoConfiguration自动配置筛选出添加@LoadBalanced注解的RestTemplate，为RestTemplate设置LoadBalancerInterceptor。

 LoadBalancerInterceptor会在对RestTemplate的请求进行拦截，然后从nacos中根据服务id获取服务列表，随后利用负载均衡算法得到真实的服务地址信息，替换服务id。 

下边通过跟踪源代码解释整个执行过程。

![这张图片展示了Java代码的编辑界面，具体为`LoadBalancerInterceptor`类的实现部分，该类实现了`ClientHttpRequestInterceptor`接口。代码中定义了构造方法和关键的`intercept`方法，`intercept`方法内会获取请求的URI，提取其中的服务ID“item-service”，还包含断言判断语句和调用`this.loadBalancer.execute()`的核心代码，这个方法正是上下文里提到的处理服务ID和用户请求的关键逻辑。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/Zgsob3e3eo94Iaxn4FpcQHVYnnU.png)

可以看到这里的intercept方法，拦截了用户的HttpRequest请求，然后做了几件事： 

request.getURI() ：获取请求uri，本例中就是 http://item-service/items?ids=100000006163

originalUri.getHost() ：获取uri路径的主机名，其实就是服务id， item-service 

this.loadBalancer.execute() ：处理服务id，和用户请求。 

这里的 this.loadBalancer 是 LoadBalancerClient 类型，LoadBalancerClient 会根据负载均衡算法选择一个实例并发起请求，我们继续跟入。

![这是一段Java源码截图，内容为Spring Cloud相关的服务注册与发现逻辑代码。截图中标记了一行关键代码：`ServiceInstance serviceInstance = this.choose(serviceId, lbRequest);`，这行代码对应上下文里“调用choose方法根据负载均衡算法选择一个服务实例”的操作，是和负载均衡源码跟踪相关的核心代码，代码前的行号为59，该步骤用来通过负载均衡算法选定要访问的服务实例。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/TnvHbQh1foIBbRxTztCcBHabntb.png)

调用choose方法根据负载均衡算法选择一个服务实例，继续跟入

![这张图片展示了一段Java代码的调试界面，是微服务中服务注册与发现的相关源码。代码中包含两个重载的choose方法，用于根据服务ID获取服务实例，当前传入的服务ID为item-service。右侧的调试弹窗显示了当前的负载均衡器为RoundRobinLoadBalancer对象，还标注了其内部的position属性值，以及服务ID、服务实例列表供应商等相关调试信息，结合上下文可知这是追踪Spring Cloud Loadbalancer源码的过程，体现了默认轮询负载均衡策略对应的实现类相关的调试内容。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/IlddbdlE2outsSxpEUHcb1x1n9c.png)

默认使用的负载均衡策略是轮询，通过RoundRobinLoadBalancer实现

最终选择一个服务实例

![这张图片展示了Java源码调试的断点运行状态，显示了Spring Cloud LoadBalancer相关的代码逻辑，当前执行到`choose`方法处，右侧工具窗口弹出`serviceInstance`的调试信息：其`serviceId`为“item-service”，`host`为192.168.101.1，`port`为8083，`metadata`为包含6个元素的Map，同时代码里调用`choose`方法获取服务实例，将其赋值给`serviceInstance`变量，该场景对应文档中Spring Cloud LoadBalancer的客户端负载源码解析部分，体现了从服务实例中选择最终节点的源码逻辑。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/OI6fbYT0YopMqWxkTpLczElEnad.png)

放行继续执行。

继续测试该接口，因为是轮询策略，下次会选择另一个服务实例

![这是一段Spring Cloud Loadbalancer相关的Java代码片段，代码中定义了`execute`方法，在调用`chooseInstance`方法获取服务实例后，通过条件判断处理服务实例为空的情况。选中的服务实例信息面板显示，该服务实例ID为`item-service`，实例ID为空，主机地址为192.168.101.1，端口为8081，未开启安全认证，哈希映射有6项元数据，当前分支逻辑是当服务实例为空时，会进入对应流程抛出异常。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/IYt4brjqooHpifxx8MlcpEjanJg.png)



在Spring Cloud 2020版本以前客户端负载均衡使用的是Netflix公司的Ribbon组件，由于Netflix公司对一些组件不再更新，Spring Cloud之后的版本就移除了Ribbon,使用Spring Cloud LoadBalance实现客户端负载均衡。

#### 3.6.2 面试题

微服务是怎么实现远程调用的？



## 4 OpenFeign-**让远程调用像本地方法调用一样简单**

### 4.1. 认识OpenFeign

在上一章，我们利用Nacos实现了服务的治理，利用RestTemplate实现了服务的远程调用，虽然ip和端口换成了服务名，但是**远程调用的代码还有些复杂**：



![图片展示了CartServiceImpl类中handleCartItems方法的部分代码。其中，第89行代码使用RestTemplate的getForObject方法，以“http://item-service/items?ids=” + itemIds.join(“,”) + “”为URL，将“ItemDTO\[\]”类型转换为ItemDTO\[\]对象。该图片与上文提到的远程调用代码复杂的问题相关，通过展示实际代码，直观呈现了使用RestTemplate进行远程调用时的代码形式，为后续讲解OpenFeign技术让远程调用更简单做铺垫。](https://feishu.cn/file/QIbdbXVn0oGwUWxIXx0cztfynkf)

有没有一种类似调用本地service方法的方式去实现远程调用呢？

接下来我们讲解的OpenFeign技术就可以让**远程调用像本地方法调用一样简单**。



AI:OpenFeign是什么

Spring Cloud早期版本使用的Feign组件实现远程调用，Feign中集成了Ribbon，随着Netflix公司不再维护Feign，Spring Cloud在Feign基础上新建了一个开源项目OpenFeign，是一个声明式的 HTTP 客户端框架，它简化了编写 REST 客户端的过程。

1. 添加依赖:

在你的项目中添加 OpenFeign 的依赖。如果你使用 Maven，可以添加以下依赖：

```Plaintext

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```



1. 创建 Feign 客户端接口:

   - 定义一个接口来描述你的远程服务。
   - 使用 `@FeignClient` 注解来标识远程调用接口，并指定服务名、接口路径。用的都是SpringMVC的注解，开发简单。

```Plaintext

@FeignClient(name = "item-service")
public interface ItemClient {
    @GetMapping("/items/{id}")
    Item getItem(@PathVariable("id") String id);
}
```



1. 注入`FeignClient` 接下来在购物车服务注入 Feign 客户端，通过Feign客户端进行远程调用。

```Plaintext

@RestController
public class CartController {
    @Autowired
    private ItemClient itemClient;
    @GetMapping("/carts/{id}/items/{itemId}")
    public ResponseEntity<Item> getCartItem(@PathVariable("id") String cartId, @PathVariable("itemId") String itemId) {
        Item item = itemClient.getItem(itemId);
       // 处理 item 数据...
        return ResponseEntity.ok(item);
    }
}
```

使用OpenFeign是非常简单的，只需要定义一个接口，用FeignClient标识目标服务的名称，在接口中定义要调用的接口，使用Spring MVC注解进行标识。在其它bean中注入此接口即可使用。



其实远程调用的关键点就在于四个：

- 请求方式
- 请求路径
- 请求参数
- 返回值类型

所以，OpenFeign就利用SpringMVC的相关注解来声明上述4个参数，然后基于动态代理帮我们生成远程调用的代码，而无需我们手动再编写，非常方便。

接下来，我们就通过一个快速入门的案例来体验一下OpenFeign的便捷吧。

<callout emoji="🎁">
1. 引入依赖
2. 编写Feign客户端

   1. feign会生成一个代理类
3. 简单使用
</callout>

### 4.2.快速入门

我们还是以cart-service中的查询我的购物车为例。因此下面的操作都是在cart-service中进行。

#### 4.2.1 引入依赖

在消费者端即`cart-service`服务的pom.xml中引入`OpenFeign`的依赖：

```XML
  <!--openFeign-->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-openfeign</artifactId>
  </dependency>
```



#### 4.2.2 编写OpenFeign客户端

在`cart-service`中，定义一个新的接口，该接口将来通过OpenFeign去访问商品服务中ItemController类的方法，所以参考商品服务的ItemController类去编写Feign接口

其中代码如下：

```Java
package com.hmall.cart.client;

import com.hmall.cart.domain.dto.ItemDTO;
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
@FeignClient(name="item-service",path = "/items")
public interface ItemClient {
    @GetMapping
    List<ItemDTO> queryItemByIds(@RequestParam("ids") Collection<Long> ids);
}
```

**这里只需要声明接口，无需实现方法。接口中的几个关键信息：**

- `@FeignClient(name="item-service")` ：声明服务名称
- `@FeignClient(path = "/items")`: 相当于ItemController类上的@RequestMapping("/items")指定的路径
- `@GetMapping` ：声明请求方式
- `@RequestParam("ids") Collection<Long> ids` ：声明请求参数
- `List<ItemDTO>` ：返回值类型



有了上述信息，OpenFeign就可以利用动态代理帮我们实现这个方法，并且向`http://item-service/items`发送一个`GET`请求，携带ids请求参数，并自动将返回值处理为`List<ItemDTO>`。

我们只需要直接调用这个方法，即可实现远程调用了。



#### 4.2.3 使用FeignClient

最后，我们在`cart-service`的`com.hmall.cart.service.impl.CartServiceImpl`中改造代码，直接调用`ItemClient`的方法：

![这张图片展示了CartServiceImpl类中引入并使用Feign客户端的代码片段，对应文档中在该类改造代码直接调用ItemClient方法的内容。图片中用红色框标出了两处关键代码，一处是通过@Autowired注解注入ItemClient实例的代码段，另一处是调用itemClient的queryItemByIds方法查询商品的代码行。该代码实现了Feign的远程调用功能，将原本需要通过其他方式实现的远程HTTP调用，转化为调用本地方法般简洁的代码，契合文档中Feign让远程调用更简洁的说明，同时体现了Feign替代RestTemplate完成服务拉取、负载均衡等工作的作用。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/EqOpbYJlYoIc1ixrMtCciTpfnId.png)

feign替我们完成了服务拉取、负载均衡、发送http请求的所有工作，是不是看起来优雅多了。



而且，这里我们不再需要RestTemplate了，还省去了RestTemplate的注册。



#### 4.2.4 启用OpenFeign

接下来，我们在`cart-service`的`CartApplication`启动类上添加注解，启动OpenFeign功能：

![图片展示了`cart-service`项目中`CartApplication`启动类的代码。其中，`@EnableFeignClients`注解被红色框突出显示，其作用是在`cart-service`的`CartApplication`启动类上添加注解，启动OpenFeign功能，该注解用于寻找使用`@FeignClient`注解的接口生成Feign客户端（代理对象），并自动配置Feign客户端与Spring Cloud LoadBalancer的集成。此图片与上文介绍`@EnableFeignClients`注解作用的内容紧密相关，直观呈现了其在代码中的位置。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/NkUTbvdXjoBYE1xZSj0cWiBbnSd.png)

@EnableFeignClients注解的作用寻找使用 `@FeignClient` 注解的接口生成Feign客户端(代理对象)，自动配置 Feign 客户端与 Spring Cloud LoadBalancer 的集成。

#### 4.2.5 测试

通过我们购物车接口测试OpenFeign实现远程调用。

#### 4.2.6 Feign和RestTemplate对比（掌握就行）

二者**核心原理完全相同**（客户端负载均衡、服务发现 + 选路 + 地址替换），仅**集成方式和入口不同**，如下图：

<sheet sheet-id="OBNZNa" token="JDEOsrliIhs5vWt01T4c2aVtnQc"></sheet>

#### 4.2.7 小结

OpenFeign的使用过程？

1. 将服务提供方提供的接口定义为Interface接口，并且@FeignClient注解标识，在接口方法上使用SpringMVC注解标识方法的路径、参数等。
2. 在服务调用方注入FeignClient接口，OpenFeign组件会根据接口生成代理对象，服务调用方通过FeignClient接口的代理对象进行远程调用。
3. 在服务调用方启动类上添加@EnableFeignClients







### 4.2 最佳实践

#### 4.2.1. OkHttp

##### 4.2.1.1 介绍

Feign底层发起http请求，依赖于第三方的**http客户端框架**。常用的http客户端有：

- HttpURLConnection：默认实现，不支持连接池

`HttpURLConnection` 是 Java 标准库中用于发送 HTTP 请求和接收 HTTP 响应的一个类。它位于 `java.net` 包中，提供了一组丰富的 API 来处理 HTTP 连接。`HttpURLConnection` 可以用于发送 GET、POST、PUT、DELETE 等各种 HTTP 方法的请求，并允许设置请求头、请求体等。

- Apache HttpClient ：支持连接池

Apache HttpClient 是一个用于发送 HTTP 请求和接收 HTTP 响应的强大客户端库，它是 Apache HttpComponents 项目的一部分。这个库为 Java 开发者提供了高级别的 API 来处理 HTTP 和 HTTPS 协议，同时也提供了低级别的组件来构建和定制 HTTP 客户端。

- OKHttp：支持连接池

OkHttp 是一个高效的 HTTP 客户端库，由 Square Inc. 开发和维护。它最初是为了提高 Android 应用程序中的网络请求效率而设计的，但现在也被广泛用于 Java 应用程序中。OkHttp 提供了许多高级功能，以简化 HTTP 请求的处理并提高性能。

因此我们通常会使用带有连接池的客户端来代替默认的HttpURLConnection。比如，我们使用OKHttp，使用连接池可以提升 HTTP 请求的效率。

首先我们跟踪源代码，查看当前默认的Client底层是HttpURLConnection，请求查询购物车接口，在远程调用商品服务处打断点：

![图片展示的是CartServiceImpl.java文件中的一段代码。代码中，当响应状态码为2xx成功时，获取响应体并调用itemClient查询商品信息。接着，通过items.stream().collect(Collectors.toMap(ItemDTO::getId, Function.identity()))将商品信息转为id到item的映射。最后，遍历CartVO列表，将商品信息中的价格、状态、库存等信息赋值给CartVO对象。该图片与上文跟踪源代码查看默认Client底层是HttpURLConnection，请求查询购物车接口等内容相关，展示了具体代码实现。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/RSpgbBo7Qo995OxbwN5c2E9Snoh.png)

继续跟入

![这张图片是Java代码调试过程中的反编译class文件截图，文件名为ReflectiveFeign.class，字节码版本为52.0（对应Java 8）。代码展示了public修饰的invoke方法的核心逻辑，当调用toString方法时，会返回对应结果，其他分支会通过调用相关类处理逻辑。截图中还能看到代码行号，右侧标注了断点位置，该断点所在的上下文是跟踪OpenFeign默认Client的源代码，结合上下文可知此处是在查看OpenFeign远程调用底层的实现细节。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/Xb4PbJDfaoTAZRxxe3rceELanOc.png)

继续跟入

![图片展示的是SynchronousMethodHandler.class文件的代码片段，位于文档中介绍OkHttp最佳实践时的跟踪源代码部分。代码中，当执行invoke方法时，先构建RequestTemplate，再获取Request.Options，然后创建Retryer实例。接着进入循环，尝试执行executeAndDecode方法，若发生RetryableException则继续或抛出。该图片与上下文紧密相关，直观呈现了使用默认Client客户端请求http时，OkHttp执行请求的代码流程。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/PUyabOuaJoSH0NxInT2c80cqnQb.png)

继续跟入

![这是一段Java程序的调试界面，展示的是SynchronousMethodHandler类的executeAndDecode方法的代码。代码中显示当前使用默认的客户端请求，红箭头指向`response = this.client.execute(request, options);`这句代码，在该句的下方弹出了调试时的变量查看窗口，其中`client`变量被查看，展开后可见其`delegate`属性为Client$Default，还有LoadBalancer相关的变量信息。该调试场景对应上下文里的内容，是跟踪源代码过程中查看OpenFeign默认客户端使用情况的环节，此时可以看到Client底层默认使用HttpURLConnection的相关调用细节。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/JA4vb7PWsoFEoKxPWezc4aGOnhe.png)

使用的是默认的Client客户端，继续跟入

![这张图片展示的是LoadBalancerUtils类的反编译代码内容，字节码版本为52.0（对应Java 8）。图中明确可见代码里的静态方法executeWithLoadBalancerLifecycleProcessing，以及该方法内调用feignClient的execute方法的代码语句，该行代码被蓝色高亮标记，还配有黄色箭头标注调试断点，结合上下文可知，这是跟踪OpenFeign远程调用的源代码时，查看默认Client客户端（HttpURLConnection）的相关底层逻辑的代码位置，用于调试查看远程调用的实现细节。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/OBONbCDnVo40WjxJH3ZcnYIgnqb.png)

继续跟入，Client执行使用**HttpURLConnection**请求http。

![图片展示了OpenFeign中Client默认使用HttpURLConnection请求http的代码及调试界面。左侧是项目结构，选中了“Client”目录。右侧代码中，Client类的execute方法被红色框标出，其中调用httpURLConnection请求http的部分被蓝色箭头指向。下方调试界面显示了请求参数、变量等信息，如请求地址为“http://192.168.6.1:7080/...”。该图片与上下文紧密相关，直观呈现了上下文提到的跟踪源代码查看默认Client底层是HttpURLConnection的内容。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/FoxNbBmdtoqxHkxZ9LjcuN9ynQb.png)

##### 4.2.1.2 更换为okhttp

<callout emoji="🏆">
1. 引入依赖
2. 修改配置，自动生效
</callout>



下边我们更换Client为okhttp

在`cart-service`的`pom.xml`中引入依赖：

```XML
<!--OK http 的依赖 -->
<dependency>
  <groupId>io.github.openfeign</groupId>
  <artifactId>feign-okhttp</artifactId>
</dependency>
```

在`cart-service`的`application.yml`配置文件中开启Feign的连接池功能：

```YAML
feign:
  okhttp:
    enabled: true # 开启OKHttp功能
```

重启服务，okhttp就生效了。

重新请求查询购物车接口，跟踪源代码查看Client的代理已变为OkHttpClient。

![这张图片是开发工具的界面截图，展示了OpenFeign更换为OkHttpClient后的相关代码与项目依赖情况。左侧项目依赖栏中，`okhttp`相关依赖已被引入；中间代码区域的`ChittyClient`类里，箭头标注的`ReactiveHttp`类位置清晰，代码中明确调用了`okhttp3`相关实现，显示远程调用的客户端代理已成功更换为OkHttpClient。该截图是对更换OkHttp操作后的效果验证，能直观呈现配置后的代码与依赖状态。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/R0LibDfJfoXEyRx2UAfcRC8Onic.png)



继续跟入代码，使用OkHttpClient发起请求并拿到响应结果。

![图片展示的是OkHttpClient.class文件的execute方法源码。该方法接收feign.Request和feign.Request.Options参数，执行IOException异常处理。关键部分是将feign.Request转换为OkHttpRequest，然后通过requestScoped的OkHttpClient发起新Call并执行，获取OkHttp.Response，最后将Response转换为FeignResponse并返回。此图片与上文更换Client为okhttp后，跟踪源代码查看Client代理已变为OkHttpClient的内容相关，直观呈现了相关代码实现。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/VyZebBjAwoWW20xCpCbc5StlnNc.png)





#### 4.2.2. 抽取API工程

##### 4.2.2.1 思路分析



<whiteboard token="UCG1wni2ThSS6cbgocrcoihAngh"></whiteboard>

上边学习了使用OpenFeign方式实现微服务接口的远程调用，每个微服务都可能需要提供接口供其它微服务调用，仍然拿商品服务举例，除了购物车服务需要调用商品服务的接口以外，订单服务也需要调用商品服务，实现的方式可参考购物车服务，在订单服务中编写Feign接口，或者直接将购物车服务中的Feign接口拷贝到订单服务。

这不是重复编码吗？ 有什么办法可以避免重复编码呢？

避免重复编码的办法就是**抽取**。不过这里有两种抽取思路：

- 思路一：抽取到微服务之外的公共module
- 思路二：每个微服务自己抽取一个module

如图：

![这张图对应微服务API工程抽取的两种思路，左侧为思路1，是将公共API模块定义在整体工程的hm-api中，该模块用于定义Feign接口，属于抽取到微服务之外的公共工程的方案；右侧为思路2，是为每个微服务单独设置API模块，每个微服务对应自身的dto、api、service相关模块，属于各微服务自己抽取模块的方案，该图清晰呈现了两种抽取思路的工程结构差异，与上下文中描述的两种抽取思路内容相呼应。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/WZJsbvqoPoumGbxKI0ScWwhqnie.jpg)

<whiteboard token="EWaZwPuS3hpNunbkkDicQtu2nHe"></whiteboard>



方案一抽取更加简单，所有微服务提供的外部接口都在一个工程中，工程结构也比较清晰，但缺点是整个项目耦合度偏高，比如：hm-api中包括了A、B、C三个微服务的接口，D服务即使只用A，只要依赖hm-api工程就会自动依赖B、C。

方案二抽取相对麻烦，每个微服务工程都需要有一个api模块，但服务之间耦合度降低。比如：A、B、C三个微服务D服务用A只需要依赖A服务下的api模块即可。



对于**中小型项目规模通常选择方案一**，不用维护太多的api的工程，简单高效。

##### 4.2.2.2 抽取API工程

<callout emoji="🏆">
1.抽取一个新的模块：hm-api
2.移动咱们的ItemClient到api工程下面去
3.cart-service依赖hm-api
</callout>

下边我们将黑马商城项目的api按方案一抽取。

在`hmall`下定义一个新的module，命名为hm-api

其依赖如下：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>com.hmall</groupId>
        <artifactId>hmall-parent</artifactId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>hm-api</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencies>
        <!--open feign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!-- load balancer-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>
        <!--OK http 的依赖 -->
        <dependency>
            <groupId>io.github.openfeign</groupId>
            <artifactId>feign-okhttp</artifactId>
        </dependency>
        <dependency>
            <groupId>com.hmall</groupId>
            <artifactId>hm-common</artifactId>
            <version>1.0.0</version>
        </dependency>
        <!-- swagger 注解依赖 -->
        <dependency>
            <groupId>io.swagger</groupId>
            <artifactId>swagger-annotations</artifactId>
            <version>1.6.6</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
</project>
```



然后把ItemDTO和ItemClient都移动到hm-api工程中，最终结构如下：

注意：ItemDTO和ItemClient在购物车工程不存在了。

下图中：com.hmall.api.item包为商品服务接口包，后期还会创建与com.hmall.api.item包平级的其它服务的接口包。

![图片展示了`hm-api`模块的目录结构。在`src/main/java`下，`com.hmall.api.item`包中包含`dto`和`client`两个子包，`dto`包下有`ItemDTO`文件，`client`包下有`ItemClient`文件。该图片与文档中抽取API工程部分内容相关，直观呈现了`hm-api`模块中抽取的API相关代码存放位置，为后续在`cart-service`中引入`hm-api`模块依赖，删除原有`ItemDTO`和`ItemClient`，重启项目后报错等问题的解决提供参考。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/NIBEbHhyHofRXIxeWMmcYL5enBb.png)

现在，任何微服务要调用`item-service`中的接口，只需要引入`hm-api`模块依赖即可，无需自己编写Feign客户端了。

##### 4.2.2.3 使用API

接下来，我们在`cart-service`的`pom.xml`中引入`hm-api`模块：

```XML
  <!--feign模块-->
  <dependency>
      <groupId>com.hmall</groupId>
      <artifactId>hm-api</artifactId>
      <version>1.0.0</version>
  </dependency>
```

删除`cart-service`中原来的ItemDTO和ItemClient，重启项目发现报错了：

![图片展示了IntelliJ IDEA中`CartServiceApplication`运行失败的界面。左侧导航栏显示`ItemServiceApplication2`已运行，`CartServiceApplication`处于失败状态。右侧Console标签页中，红框突出显示了错误信息，内容为`Field itemClient in com.hmall.cart.service.impl.CartServiceImpl required a bean of type 'com.hmall.api.item.ItemClient' that could not be found.`，表明`CartServiceImpl`中所需的`ItemClient`类型bean未找到。该图片与上下文紧密相关，直观呈现了因`ItemClient`定义位置变化导致扫描不到而报错的情况。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/VyGabzyAdoaYBjxibx7cR8K7nlf.png)

这里因为`ItemClient`现在定义到了`com.hmall.api.item`包下，而cart-service的启动类定义在`com.hmall.cart`包下，扫描不到`ItemClient`，所以报错了。



解决办法很简单，在cart-service的启动类上添加声明即可，两种方式：

- 方式1：声明扫描包：(推荐此方式)

![图片展示了`CartServiceApplication`类的部分代码。关键部分是第10行的`@EnableFeignClients`注解，其参数`basePackages`被红框突出显示，值为`{"com.hmall.api"}`。这与上下文介绍的在`cart-service`的启动类上添加声明解决扫描不到`ItemClient`报错问题相呼应，推荐使用方式1即在启动类上声明扫描包来解决此问题。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/WAa8bWoVSo7f8MxxHBecGF1jnJh.png)

- 方式2：声明要用的FeignClient

![图片展示了`CartServiceApplication`类的部分代码。关键信息是第11行的`@EnableFeignClients`{clients = { {ItemClient.class}}`注解，其中`ItemClient.class`被红色箭头突出显示。这与上下文介绍的在`cart-service`的启动类上添加声明解决扫描不到`ItemClient`报错问题相呼应，推荐使用方式1：在启动类上声明扫描包，此方式被标注为推荐方式。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/Gz9SbuDdZohutIx690pcGxwenTQ.png)

推荐方式一。



#### 4.2.3. 完善服务接口

目前为止我们把商品服务的查询商品信息接口作为Feign接口抽取到了hm-api工程中，假设**有一天接变更了该怎么办？比如：商品查询接口加了一个参数，首先需要修改商品服务的商品查询Controller方法**，添加一个参数，然后修改hm-api工程的对应的接口添加一个参数。

这两个地方都要改，一个是服务端的接口，一个是于服务端接口对应的Feign接口。

如果忘记修改其中一个地方或两个地方修改的不一致则会导致接口无法正常调用。为了解决这个问题我们可以让服务端controller类去实现hm-api中的Feign接口，这样一来要改就一起修改了。

首先在商品服务引入hm-api

```JavaScript
<dependency>
    <groupId>com.hmall</groupId>
    <artifactId>hm-api</artifactId>
    <version>1.0.0</version>
</dependency>
```

然后让ItemController类实现ItemClient接口

```JavaScript
package com.hmall.item.controller;


import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.hmall.api.item.ItemClient;
import com.hmall.common.domain.PageDTO;
import com.hmall.common.domain.PageQuery;
import com.hmall.common.utils.BeanUtils;
import com.hmall.item.domain.dto.ItemDTO;
import com.hmall.item.domain.dto.OrderDetailDTO;
import com.hmall.item.domain.po.Item;
import com.hmall.item.service.IItemService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

import java.time.LocalDateTime;
import java.util.List;
import java.util.concurrent.TimeUnit;

@Api(tags = "商品管理相关接口")
@RestController
@RequestMapping("/items")
@RequiredArgsConstructor
public class ItemController implements ItemClient {
....


@ApiOperation("根据id批量查询商品")
@GetMapping
@Override
public List<ItemDTO> queryItemByIds(@RequestParam("ids") Collection<Long> ids){
    return itemService.queryItemByIds(ids);
}


```

代码报错，发现是ItemDTO冲突，删除商品服务中的ItemDTO,统一使用hm-api中的ItemDTO。

修改完成重启商品服务和购物车服务测试查询购物车接口。



### 4.3 面试题

OpenFeign的工作原理？



## 5 总结

<whiteboard token="FdlBwujE2horyLbyw8ccDGiJn8d"></whiteboard>

## 作业

### 创建用户微服务

将hm-service中用户管理相关的业务抽取出来,创建用户微服务user-service。

pom.xml

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.hmall</groupId>
        <artifactId>hmall-parent</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>user-service</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!--common-->
        <dependency>
            <groupId>com.hmall</groupId>
            <artifactId>hm-common</artifactId>
            <version>1.0.0</version>
        </dependency>
        <!--web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--加密-->
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-crypto</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-rsa</artifactId>
            <version>1.0.9.RELEASE</version>
        </dependency>
        <!--数据库-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <!--单元测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <!--nacos 服务注册发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>
    <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

application.yaml

```YAML
server:
  port: 8084
spring:
  application:
    name: user-service
  cloud:
    nacos:
      server-addr: 192.168.101.68:8848 # nacos地址
  profiles:
    active: dev
  datasource:
    url: jdbc:mysql://${hm.db.host}:3306/hm-user?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: ${hm.db.pw}
mybatis-plus:
  configuration:
    default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
  global-config:
    db-config:
      update-strategy: not_null
      id-type: auto
logging:
  level:
    com.hmall: debug
  pattern:
    dateformat: HH:mm:ss:SSS
  file:
    path: "logs/${spring.application.name}"
knife4j:
  enable: true
  openapi:
    title: 用户服务接口文档
    description: "用户服务接口文档"
    url: https://www.itcast.cn
    version: v1.0.0
    group:
      default:
        group-name: default
        api-rule: package
        api-rule-resources:
          - com.hmall.user.controller
hm:
  jwt:
    location: classpath:hmall.jks
    alias: hmall
    password: hmall123
    tokenTTL: 30m
  auth:
    excludePaths:
      - /search/**
      - /users/login
      - /items/**
      - /hi
```

项目结构

![图片展示了user - service项目的项目结构。在src目录下，main文件夹内有java、domain、mapper、service、utils、resources等子文件夹。其中，java文件夹下有config、controller、domain、enums、interceptor、mapper、service、utils等子文件夹，如config文件夹下有AuthProperties、JwtProperties、MvcConfig、SecurityConfig等配置类；domain文件夹下有dto、po、vo等子文件夹，如dto文件夹下有AddressDTO、LoginFormDTO等数据传输对象类；utils文件夹下有JwtTool、UserServiceApplication等工具类。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/SGGxbdxcio1nKdxWDN3cVNHenwf.png)



#### 注意事项

当业务代码中需要知道当前登录用户是谁，目前暂未实现，先将用户id写死，修改UserContext如下：

![这张图片展示的是UserContext.java文件的代码内容，是作业中要求修改的用户相关上下文工具类。代码里的getUser方法原本的逻辑是返回tl.get()，被注释暂不启用，同时添加了“//todo 暂是返回固定的用户id”的注释，当前该方法会返回固定值1L，以此实现文档中所说的暂未完成登录用户识别时，将用户id写死的要求。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/F2V6boRIIo3K3Sxo6JGcGqbFnxb.png)

user-service也需要自己的独立的database，向MySQL中导入课前资料提供的SQL：

注意：即使虚拟机已存在hm-user数据库也需要运行此脚本。

![这张图片展示了Windows系统的文件资源管理器界面，路径为“新加卷(D:) > 课程资料 > 服务框架 > day02-微服务01 > 资料”。界面中列出了该路径下的各类文件与文件夹，包含hmall-nginx、nacos两个文件夹，以及hm-cart.sql、hm-item.sql等多个SQL源文件，还有“黑马商城测试.jmx”的JMX文件。其中hm-user.sql文件被蓝色高亮选中，结合上下文来看，该文件应为创建用户微服务作业中导入的用户相关SQL文件，用于后续的用户服务开发。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/CmiRb2qJ3o10Brxs63kcuKNBnse.png)

导入结果如下：

![图片展示的是CentOS系统中数据库表结构。画面中以树状结构呈现，根节点为“centos”，下有“hm-cart”“hm-item”“hm-user”等分支，其中“hm-user”被高亮显示。其下有“address”“user”两个子节点，分别显示大小为32.0 KiB。该图与文档中创建用户微服务的导入结果相关，直观呈现了数据库中与用户相关的表结构情况。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/NV6rbEeQbomZLpxUqvPcc9Q3ntg.png)

#### 测试

启动UserApplication，访问[http://localhost:8084/doc.html#/default/用户相关接口/loginUsingPOST](http://localhost:8084/doc.html#/default/%E7%94%A8%E6%88%B7%E7%9B%B8%E5%85%B3%E6%8E%A5%E5%8F%A3/loginUsingPOST)，测试登录接口：

![图片展示的是Postman测试登录接口的界面。在“POST /users/login”请求地址下，请求参数为JSON格式，包含“password”“rememberMe”“username”等字段，其中“password”值为“123456”等。下方显示响应内容，包含“token”“userId”“username”“balance”等字段，如“token”值为“eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoxLCJ1c2VybmFtZSI6IkJhYyIsImlhdCI6MTY4NjY4NjY0NiwibmJmIjoxNjg2Njg2NjQ2LCJpYXQiOjE2ODY2ODY2NDYsImV4cCI6MTY4NjY4NjY0NiwidXNlciI6MSwiaWF0IjoxNjg2Njg2NjQ2LCJ1c2VybmFtZSI6IkFjayIsImJhY2tldCI6IjIwMjAtMDMtMjMgMjAwLTAyOjIwIiwibGFuZyI6IjIwMjAtMDMtMjMgMjAwLTAyOjIwIiwibmFtZSI6IkFhayIsImlkIjoxLCJiYWxhbHlzaCI6IjIwMjAtMDMtMjMgMjAwLTAyOjIwIn0.5QdYy881...”。该图片与上文测试登录接口的内容相关，直观呈现了测试时的请求与响应数据。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/B2lqbFL5eoZ5xfxqErjccg1Fnrd.png)

用户服务测试通过。



### 创建交易微服务

将hm-service中交易相关的业务抽取出来,创建交易微服务trade-service。

pom.xml

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.hmall</groupId>
        <artifactId>hmall-parent</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>trade-service</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    <dependencies>
        <!--common-->
        <dependency>
            <groupId>com.hmall</groupId>
            <artifactId>hm-common</artifactId>
            <version>1.0.0</version>
        </dependency>
        <!--web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--数据库-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <!--单元测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <!--nacos 服务注册发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <dependency>
            <groupId>com.hmall</groupId>
            <artifactId>hm-api</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>
    <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

application.yaml

```YAML
server:
  port: 8085
spring:
  application:
    name: trade-service
  cloud:
    nacos:
      server-addr: 192.168.101.68:8848 # nacos地址
  profiles:
    active: dev
  datasource:
    url: jdbc:mysql://${hm.db.host}:3306/hm-trade?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: ${hm.db.pw}
mybatis-plus:
  configuration:
    default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
  global-config:
    db-config:
      update-strategy: not_null
      id-type: auto
logging:
  level:
    com.hmall: debug
  pattern:
    dateformat: HH:mm:ss:SSS
  file:
    path: "logs/${spring.application.name}"
knife4j:
  enable: true
  openapi:
    title: 交易服务接口文档
    description: "交易服务接口文档"
    url: https://www.itcast.cn
    version: v1.0.0
    group:
      default:
        group-name: default
        api-rule: package
        api-rule-resources:
          - com.hmall.trade.controller
feign:
  okhttp:
    enabled: true # 开启OKHttp功能
```

项目结构如下

![这张图片展示了trade-service项目的结构，该项目是交易微服务相关的项目。项目下包含src与resources两大目录，src目录里的main文件夹中设有java目录，java目录内的com.hmall.trade包下，划分了controller、domain、mapper、service等功能模块，其中controller模块下有OrderController，domain模块下又细分出dto、po、vo子包，分别存放对应数据类，mapper模块对应多个数据库操作接口，service模块及子impl包存放业务逻辑相关内容，还有TradeServiceApplication启动类；resources目录下存放了配置文件，包括application.yaml及不同环境的配置文件。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/FroqbCHh0ob6EmxvrITcxfeGnEb.png)



#### 提示

在交易服务中，用户下单时需要做下列事情：

- **根据id查询商品列表**
- 计算商品总价
- 保存订单
- **扣减库存**
- **清理购物车商品**



其中，查询商品、扣减库存都是与商品有关的业务，在item-service中有相关功能；清理购物车商品是购物车业务，在cart-service中有相关功能。

因此**交易服务**要调用他们，必须通过OpenFeign远程调用。我们需要将上述功能抽取为FeignClient.

在抽取FeignClient的过程中，接口涉及的DTO需要迁移到hm-api工程中。

比如：

将`OrderDetailDTO`抽取到`hm-api`模块的`com.hmall.api.item.dto`包下，删除原来商品服务下的OrderDetailDTO。

注意：删除原来商品服务下的OrderDetailDTO后很多代码报错，需要细心修改OrderDetailDTO的最新包路径。



抽取FeignClient接口完成后需要改造OrderServiceImpl中的createOrder方法，将查询商品、扣减库存、清理购物车这三个本地方法调用改造为基于FeignClient的调用。

```Java
package com.hmall.api.cart;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.util.Collection;
import java.util.List;

@FeignClient(name="cart-service",path="/carts")
public interface CartClient {
    @DeleteMapping
    void deleteCartItemByIds(@RequestParam("ids")Collection<Long> ids);
}
```

#### 测试

启动TradeApplication，访问[http://localhost:8085/doc.html](http://localhost:8085/doc.html#/default/%E8%AE%A2%E5%8D%95%E7%AE%A1%E7%90%86%E6%8E%A5%E5%8F%A3/queryOrderByIdUsingGET)

测试创建订单接口。

测试参数说明：

代码中目前我们将用户ID硬编码为 1，用户名为jack，在hm-user数据库的user表中。

addressId：收货地址，在hm-user数据库的address表中。

itemId: 购买商品的id，在hm-item数据库的item表中。

num：购买数量。

paymentType：支付方式，1：支付宝、2：微信，3：余额支付。

测试效果如下：

![这张图片展示了交易服务接口文档的界面，左侧菜单栏选中了“创建订单”的POST请求选项，右侧测试区域可选择请求参数格式，当前选中raw、JSON格式，请求参数包含addressId、itemId、num、paymentType字段，其中paymentType的值为3，符合上下文提及的余额支付规则。该图片用于呈现交易微服务创建订单接口的测试页面，对应文档中创建交易微服务后测试创建订单的相关内容，下方还显示了返回的订单ID，验证了创建订单成功的测试效果。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/MIBibkDqSoQiFOxdEiecYlKkn9g.png)

创建订单成功返回订单ID。



测试查询订单接口：

使用上个测试返回的订单ID测试查询订单信息接口。

![这张图片展示的是交易服务接口文档的页面，用于测试查询订单信息的接口。该接口采用GET请求方式，请求地址为/orders/{id}，请求参数为订单ID，具体值为1844193420193566369，输入参数为form-data类型。下方的响应区域清晰显示了订单的详细信息，包括订单总价、支付方式、订单状态、创建时间等内容，这对应了文档中提及的“使用上个测试返回的订单ID测试查询订单信息接口”的要求，呈现出该接口测试成功的效果。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/DWTAbKGN5oQ2joxZucCcrsGAnAU.png)





### 创建支付微服务

将hm-service中支付相关的业务抽取出来,创建支付微服务pay-service。



pom.xml

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.hmall</groupId>
        <artifactId>hmall-parent</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>pay-service</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    <dependencies>
        <!--common-->
        <dependency>
            <groupId>com.hmall</groupId>
            <artifactId>hm-common</artifactId>
            <version>1.0.0</version>
        </dependency>
        <!--web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--数据库-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        <!--单元测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <!--nacos 服务注册发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <dependency>
            <groupId>com.hmall</groupId>
            <artifactId>hm-api</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>
    <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```



application.yaml

```YAML
server:
  port: 8086
spring:
  application:
    name: pay-service
  cloud:
    nacos:
      server-addr: 192.168.101.68:8848 # nacos地址
  profiles:
    active: dev
  datasource:
    url: jdbc:mysql://${hm.db.host}:3306/hm-pay?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: ${hm.db.pw}
mybatis-plus:
  configuration:
    default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
  global-config:
    db-config:
      update-strategy: not_null
      id-type: auto
logging:
  level:
    com.hmall: debug
  pattern:
    dateformat: HH:mm:ss:SSS
  file:
    path: "logs/${spring.application.name}"
knife4j:
  enable: true
  openapi:
    title: 支付服务接口文档
    description: "支付服务接口文档"
    url: https://www.itcast.cn
    version: v1.0.0
    group:
      default:
        group-name: default
        api-rule: package
        api-rule-resources:
          - com.hmall.pay.controller
feign:
  okhttp:
    enabled: true # 开启OKHttp功能
```

项目结构如下：

![这是创建完成的支付微服务项目结构，结构根目录为pay-service，内部包含src和resources两个目录。src目录下的main目录中，java目录下的com.hmall.pay包内划分了多个子包，包括存放控制器类的controller包，其中有PayController；存放数据传输对象的domain包，包含PayApplyDTO、PayOrderFormDTO等类；存放枚举类的enums包，有PayChannel、PayStatus等枚举项；存放持久化对象的po包；存放值对象的vo包；存放数据访问接口的mapper包，其中有PayOrderMapper；存放业务逻辑相关的service包，包含IPayOrderService接口和PayOrderServiceImpl实现类；还有启动类PayServiceApplication。resources目录下有mapper目录，以及application.yaml、application-dev.yaml、application-local.yaml三个配置文件。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/OTyHb40OIoN9doxCc90cqELUnSh.png)



#### 提示

在支付服务中，基于用户余额支付时需要做下列事情：

- **扣减用户余额**
- 标记支付单状态为已支付
- **标记订单状态为已支付**



其中，**扣减用户余额**是在`user-service`中有相关功能；**标记订单状态**则是在`trade-service`中有相关功能。因此交易服务要调用他们，必须通过OpenFeign远程调用。我们需要将上述功能抽取为FeignClient.

抽取FeignClient完成后就可以改造`PayOrderServiceImpl`中的逻辑，将扣减用户余额、标记订单状态为已支付这两个本地方法调用改造为基于`FeignClient`的调用。



#### 测试

启动PayApplication，访问[http://localhost:8086/doc.html](http://localhost:8086/doc.html#/default/%E6%94%AF%E4%BB%98%E7%9B%B8%E5%85%B3%E6%8E%A5%E5%8F%A3/queryPayOrdersUsingGET)。

测试创建支付单。

接口参数说明：

bizOrderNo：业务订单号，在测试交易服务创建订单接口生成的订单ID，对应hm-trade数据库的order表的ID。

amount：订单金额，单位：分，对应hm-trade数据库的order表的total_fee字段

payChannelCode：支付渠道编码，余额支付渠道编码为balance，支付渠道编码在枚举类PayChannel中定义。

payType：余额支付类型为5

orderInfo：订单信息字符串。



![这张图片展示了支付服务接口测试的页面，是支付服务接口文档中的测试模块区域，对应余额支付接口的参数测试环节。页面选中的是POST请求方式，目标地址为pay-orders，请求数据格式选择raw的JSON格式，参数内容里指定了payType为余额支付类型（数值为5），当前已填入测试用的amount、orderInfo等参数，下方的响应内容区域显示了测试成功后返回的支付单ID，符合支付测试后返回对应pay-order表ID的要求。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/RZcobqYGNosx5Qxgpu8cvr3tn1c.png)

测试成功返回支付单ID，对应hm-pay数据库中的pay-order表的ID。



测试余额支付接口：

参数测试：

Id: 支付单ID

pw：支付密码，即用户的登录密码，账号为1的用户密码为123.

![图片展示的是一个API测试界面，用于测试余额支付接口。请求方法为POST，URL为/pay-orders/(id)。在“请求参数”选项卡下，参数名称为id，参数值为1933927193215852545。下方示例代码中，id、payType、bizOrderNo、pw等参数值与上文提到的测试参数对应，其中pw为支付密码123。该图片与上文测试余额支付接口的内容相关，直观呈现了测试时的请求参数设置情况。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/OL36bGXAyop58TxASlKcq8NmnEg.png)

支付成功可以从hm-pay数据库中的pay-order表找到对应的记录，查看status字段值是否为3(表示支付成功)

---
> 来源：飞书技术知识库（脱敏版） · 04 微服务与中间件 / 服务注册与发现 / 01 微服务服务注册与发现 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/GTHrwjDCViHF4skYj05cs2iknLg)
