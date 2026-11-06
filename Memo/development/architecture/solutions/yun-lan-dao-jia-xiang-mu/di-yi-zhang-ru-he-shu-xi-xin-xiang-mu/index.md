---
type: memo
date: 2026-11-06
aliases:
  - "第一章 如何熟悉新项目"
---

<title>第一章 如何熟悉新项目</title>

# 学习目标

入职公司进入项目团队的第一件事就是熟悉项目，本章以家政服务O2O项目为线索一起教大家如何去熟悉一个新项目，通过本章节的学习完成以下学习目标：

1. 能够说出项目的业务模块
2. 能够说出项目的业务流程
3. 能够说出项目的技术架构
4. 能够搭建项目的开发环境
5. 能够根据需求熟悉项目的表结构和代码



# 1 如何熟悉一个新项目

进入一家新公司初次接触一个新项目应该从哪里入手呢？怎么做才能快速的去熟悉项目并顺利进入开发工作？

首先去熟悉项目大概情况，包括：项目的背景、项目模块、业务流程、项目架构等。

项目情况大概了解后就可以搭建开发环境。

然后去理解自己所分配的模块的需求及设计，熟悉项目现有的代码并进入开发。



## 1.1 项目的业务模块

了解项目的业务从项目背景开始。

目标：

能够说出项目整体业务流程

### **1）项目背景**

2015年国家就提出了"互联网+"的行动计划，什么是"互联网+"?

![图片以蓝色背景和电路线条为设计元素，突出显示“互联网+”字样。该图片位于介绍“互联网+”行动计划的上下文之后，用以直观呈现“互联网+”这一概念，与上下文对“互联网+”的定义相呼应，即“互联网+传统行业”，互联网为传统行业赋能，利用互联网优势对传统行业进行优化升级转型，推动社会发展。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/JVNpbon1WoNW2RxDuWacXadUn5c.png)

“互联网+”简单的说就是“互联网+传统行业”，互联网为传统行业赋能,利用互联网具备的优势特点对传统行业进行优化升级转型，最终推动社会不断地向前发展。

互联网+的成功案例有很多，像滴滴出行、美团外卖和共享单车等平台将传统出行、餐饮和交通等行业与互联网连接，提供更便捷、经济高效的服务。

云岚到家项目是一个家政服务o2o平台，互联网+家政是继打车、外卖后的又一个风口，创业者众多，比如：58到家，天鹅到家等，o2o（Online To Offline）是将**线下商务的机会与互联网的技术结合**在一起，让互联网成为线下交易的前台，同时起到推广和成交的作用。

家政服务o2o市场前景非常好，中国社会的家庭小型化、人口老龄化进程加快，加之二胎政策、三孩政策的推行，创造了大量对于家政服务的潜在需求。艾媒咨询发布《2022-2023年中国家政服务行业发展剖析及行业投资机遇分析报告》显示，从2015年的2776亿元已提升至2021年的10149亿元，进入万亿级市场行列。

**中国家政服务业从业人员数量规模逐年稳步增长**

![图片为2015 - 2021年中国家政服务从业人员数量及其增长率柱状折线图。2015年从业人员数量为2326万人，2021年为3760万人，期间呈逐年增长趋势。2017年增长率最高，达10.15%，2019年最低，为6.48%。该图与上下文紧密相关，直观呈现了中国家政服务业从业人员数量逐年稳步增长的情况，与文档中提及的家政服务o2o市场前景好、从业人员数量规模逐年增长等内容相呼应。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/Eu8QbRJI6oBZr7xdOwoc7vNCnzc.png)

### **2）运营模式**

我们调研了家政服务O2O平台的运营模式，如下：

![这张图展示了家政服务O2O领域的两种运营模式，上方为C2B2C模式，流程为家政服务人员入驻受平台管理后，家政需求方可在平台注册找服务，服务人员为需求方提供服务；下方为B2B2C模式，流程是家政服务公司作为中介入驻受平台管理，家政需求方在平台注册找服务，平台将任务派发给中介公司，中介再将任务派发给旗下的服务人员。这张图与前文调研家政服务O2O平台运营模式的内容相呼应，直观呈现了两种模式的运作流程。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/LtSYbPEiIomWDmx8nMOcRw1vnWd.png)

C2B2C:

在家政 O2O（Online to Offline，线上到线下）领域中，"Consumer to Business to Consumer"（C2B2C）描述了一个商业模式，消费者不仅可以通过平台获取家政服务，还有机会成为服务提供者，平台做为中介提供家政服务的机会给服务提供者，并从中抽成实现盈利。在这个背景下，C2B2C 模式通常指的是：

1. **消费者（Consumer）：**

   - 最终的家庭用户，他们需要家政服务，例如清洁、保姆、维修等。
2. **企业（Business）：**

   - 在家政 O2O 中，企业通常是在线平台，提供家政服务的中介。平台通过在线渠道为消费者提供了查找、预订、支付等服务，同时也为家政服务提供者提供了工作机会。
3. **消费者（家政服务提供者）：**

   - 在 C2B2C 模式中，一些消费者也可以成为服务的提供者。这些个体可能是独立的家政服务专业人员，他们可以在家政 O2O 平台上注册，提供自己的服务，并被其他需要服务的消费者雇佣。



B2B2C:

代表着"Business to Business to Consumer"，即企业到企业到消费者的模式。家政服务平台作为中间商，通过与各种家政服务提供商（家政服务公司）合作，为消费者提供多样化的家政服务选择。

B2B2C与C2B2C的区别是：B2B2C中服务提供者是家政服务中介公司，在C2B2C中是服务提供者是拥有服务技能的服务人员（散户）。



**本项目结合了C2B2C和B2B2C模式，个人和家政服务中介都可以通过平台提供家政服务**，如下图：

项目包括四个端：**用户端(小程序)、服务端（app）、机构端(PC)、运营管理端(PC)**，四个端对应四类用户角色：

家政需求方：通过用户端小程序完成在线预约下单、支付、评价、投诉、退款等操作。

家政服务人员：通过服务端APP完成在线接单、结算等操作。

家政服务公司：通过机构端完成在线接单、派单、投诉处理、结算等操作。

平台方：通过管理端完成服务人员管理、机构管理、订单管理、财务管理等操作，一笔完成的订单，结算时按照分成比例平台进行抽成。

![这张图展示了该家政项目的运营模式，明确了各角色间的运作关系：家政需求方通过注册找家政服务，向家政服务人员或家政服务公司（中介）提出需求；家政服务人员和家政服务公司（中介）均需入驻并受平台管理，家政平台方向两者派任务，服务方向需求方提供服务，同时对应了文档中提及的四个端及四类用户角色的业务逻辑，清晰呈现了家政服务从需求发起、服务提供到平台管理的完整链路。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/SdkFbCQYCo7vHFxFfCFcgqWXnRb.png)

### 3) 项目模块

项目模块列表：

![这张项目业务模块表格，对应文档中说明的家政服务项目业务模块，详细梳理了该项目的各类业务模块、子模块及对应功能点，并标注了优先级P0至P3。表格涵盖登录、服务管理、服务人员管理、业务规则、机构管理、订单管理等多个模块，每个模块下细分对应子功能，比如服务人员管理包含服务人员信息管理、服务人员认证管理，订单管理包含订单查询、订单派单、订单审核等，各功能点补充了对应操作说明，如服务人员需经实名认证才可接单，订单处理需及时响应异常问题等，完整呈现了该家政服务项目的功能框架。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/VSFjbtINaomcMWxY9KIckbVVntc.png)

根据每个端去细化模块，项目业务模块图如下：

![图片展示了家政服务项目各端业务模块的业务流程图。左侧为用户端（小程序）、服务端（APP）、机构端（PC）、管理后台（PC）等登录入口，右侧有三方系统、外呼系统、存管服务、评价系统、支付系统、财务系统、地图服务等系统。各端业务模块包括基础配置、抢单、订单管理、我的、服务人员管理、企业管理、客户管理、营销管理等，如用户端有服务搜索、服务下单、城市定位、三方支付等，服务端有服务技能配置、接单配置等。该图与上文介绍的项目业务模块相呼应，直观呈现各端业务流程。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/Byxmb973aoyXo3xwyP0cotjOnNd.png)

核心模块说明：

服务管理：对家政服务项目进行管理，最后在指定区域上架服务后用户可在当前区域购买。

下单支付：用户通过小程序完成下单支付，进入小程序首页查询服务，用户选择服务，下单并支付

抢单：服务人员和机构进行抢单。首先服务人员和机构设置接单范围、服务技能、开启抢单开关，然后进入抢单界面进行抢单。

派单调度：平台根据撮合匹配算法通过任务调度将订单和服务人员进行撮合匹配，促进成交。

订单管理：对订单的生命周期进行管理，包括创建订单、取消订单、删除订单、历史订单等。

服务人员管理：对服务人员的信息、认证等进行管理。

企业管理：对机构的信息、认证进行管理。

客户管理：对c端用户的信息、用户的状态等信息进行管理。

营销管理：对优惠券活动进行管理。

### 4) 项目业务流程

**项目核心业务流程如下：**

<whiteboard token="LplNwUbIoh1XHLblNufcQ4r0nrc"></whiteboard>

核心流程：

1. 运营端在运营区域上架家政服务

比如：在北京上架 日常保洁、空调维修。

1. 用户端通过定位区域获取当前区域的服务项目，选择家政服务，下单、支付
2. 家政服务人员及家政服务公司（机构）通过平台抢单
3. 家政服务人员现场服务，平台跟踪管理整个服务过程。
4. 服务完成，用户评价、售后服务等。

### 5）界面原型展示

下边展示部分界面原型，按如下流程展示

1. 运营端服务管理，服务上架
2. 用户下单、支付（支付可以略过）
3. 服务人员抢单
4. 现场服务，开始服务、完成服务
5. 用户评价



#### **运营端统一管理服务项**

运营人员通过运营端管理平台的服务项，设置运营城市开通的服务项。

![图片展示的是“云岚到家”平台的服务管理 - 服务项管理界面。左侧为功能导航栏，当前选中“服务管理”下的“服务项管理”。右侧上方有“服务管理 - 服务项管理”标题及“刷新”按钮。中部是服务项列表，包含服务编号、服务名称、服务图标、服务图片、服务单价、服务类型、服务描述、操作等列，如173412743276231904的阳光汽车服务等。下方有“搜索”按钮。该图与上文介绍的项目业务模块中服务管理相关，直观呈现了服务项管理界面。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/QdBGbZEyWoUoxrxhq7pcHPrBntb.png)

设置城市开通的服务：

![该图片展示的是设置城市开通服务的界面，页面顶部标注区域名称为北京，设有“添加服务”按钮。界面内以表格形式呈现已录入的保洁类服务信息，包含服务名称、服务类型、参考价格、区域价格、是否热门、状态、操作选项，其中当前的区域价格均设置为44。表格内的服务有日常保洁、深度保洁、马桶清洁三类，部分服务状态为已上架、已下架或草稿，各服务对应有设置热门、删除、上架或下架等操作选项。页面底部带有分页控件，支持跳转至指定页码查看服务列表。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/PeztbG9MUogMlOxKIwKcKwXXnEf.png)



#### **用户端购买服务**

用户端通过小程序预约家政服务。

![这是一个服务类APP的界面，顶部显示定位为北京，搜索框内有“日常保洁”的搜索词，设有全屋除菌保洁3次低至459元的新人优惠广告，还标注了“小智帮平台，给您贴心的专业的上门服务”等服务承诺。界面中部整齐排列着保洁清洗、日常保洁、家电清洗等12项业务分类图标；下方设有“精选推荐”板块，展示了空调上门检修的服务宣传，标注有“维修不限品牌”“专业极速达”的特点，以及198元的油烟机清洗服务选项，右下角设有“立即预约”按钮。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/GnKCb9LrpoY2shxtRjocGHtGn0b.png)



![图片展示的是一个预约信息填写界面。上方提示“请填写您的预约信息”，并有“×”关闭按钮。中间显示预约地址为“北京市朝阳区大望路是那块区域很久以后更好路上走”，下方有电话号码“阿圈 12409956322”。下方有“上门时间”显示为2026 - 06 - 10 12:00，以及“优惠券信息”显示可使用优惠券抵扣10元。最下方显示预约价格为198元，有红色“立即预约”按钮。该图片与上下文介绍的预约服务流程相关，是预约环节中填写信息的展示。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/JgIfbdlZLoSIavxw20eclXg8nPy.png)



![这张图片展示了服务类相关小程序的两个操作界面，左侧是支付订单界面，显示订单总价30元、优惠金额10元，待付金额20元，还显示了支付剩余时间为08:40，提供微信支付和支付宝支付两种支付方式，底部标注还需支付30元，还有确认支付的按钮。右侧是下单成功界面，配有绿色对勾图标，显示“下单成功”字样，提示服务人员正在抢单，设有“返回首页”和“查看订单”两个功能按钮，该界面对应文档中服务人员上门服务相关流程的内容展示。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/T2vVbWfhDonJS3xse4DcoeZGnFe.png)



#### **服务人员上门服务**

服务人员进入app开启接单

![图片展示的是服务人员进入app开启接单的界面。界面上方显示时间为9:41，右上角有信号、Wi - Fi和电池图标。中间标题为“接单设置”，下方有一个“开启接单”的开关，当前处于关闭状态。该图片与上文“服务人员进入app开启接单”内容对应，直观呈现了开启接单操作前的界面状态。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/YKlNbsRyco9eQhxgUl0cj3mUnoc.png)

选择接单范围

![这张图片展示的是服务人员接单相关流程里选择工作地点的页面，页面顶部标注“请选择工作地点”。地图区域里有沙河水庫、中关村公园等地理标识，还有G6、G7等道路标识，地图上设有可拖动修改的当前定位地址，右下角有“重新定位”按钮，页面下方显示当前服务城市为北京，意向接单范围为朱辛庄地铁站，最下方有红色的“保存”按钮，该页面用于服务人员选择接单的工作地点，对应文档中服务人员上门服务流程里选择接单范围的环节。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/YPufbIdu3oB9VSx4tkNcQa6znfc.png)

立即抢单：

![图片展示的是云岚到家app界面，用于说明服务人员上门服务时选择接单范围的操作。界面左上角显示“北京”，下方有“服务技能”“接单设置”“服务范围”三个选项，其中“服务范围”被红色框突出显示。下方是“抢单大厅”，有“全部”“3公里内”等服务范围选项，其中“5公里内”被红色框突出显示。该图片与上文“选择接单范围”内容对应，直观呈现了选择服务范围的操作界面。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/LSddbU5Pqo7EDuxWiuMcQEO8nWh.png)



抢单成功在规定时间上门服务

![图片展示的是服务人员在app中查看订单界面。界面上方显示时间为9:41，有信号、Wi - Fi和电池图标。中间有“订单”“历史订单”选项，当前选中“订单”。下方有“待上门”“待完工”“已完成”三个标签，其中“待上门”标签上有红色圆圈标识的“99+”数字。下方显示两条待上门订单，每条订单包含商品图、服务内容、预约时间和地址，右侧有“开始服务”按钮。该图与上文介绍的服务人员上门服务流程中查看订单的内容相关。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/S2Y2buq30osWQ5xPYjKcgPuDnWe.png)

开始服务前上传照片

![图片展示的是服务人员在app中开始服务前的界面。界面上方显示“开始服务”，下方有“服务前照片”区域，左侧有一张服务前的照片，右侧有“上传图片”按钮。再下方是“补充说明”区域，有输入框供补充说明本次服务前的其他情况。最下方是“确认提交”按钮。该图片与文档中“服务人员上门服务”流程中“开始服务前上传照片”步骤对应，直观呈现了上传照片的操作界面。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/OPlSbZYkxoAasXxheHQcnNlUnje.png)

服务完成上传服务后的照片

![这张图片是项目服务相关界面的“完成服务”页面，对应文档中服务人员完成服务后的操作步骤，核心展示服务人员提交服务结果的相关操作区域。页面上方标注“完成服务”，设有“服务后照片”区域，左侧展示了已上传的服务后人员工作照片，右侧配有带相机图标的“上传图片”按钮，用于补充上传服务后的照片；下方是“补充说明”输入框，提示可输入相关说明补充内容，右下角标注剩余可输入字符为0/50，最底部是用于确认提交操作的“确认提交”按钮。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/K32MbwIxUopXsixtUygcvAp5nee.png)

#### 服务完成，用户进行评价

![图片展示的是服务完成用户进行评价的界面。上方显示时间为9:41，有信号、Wi - Fi和电池图标。中间标题为“我的评价”，下方有“全部”“好评”“中评”“差评”四个选项卡，当前选中“全部”。评价内容显示用户头像、用户名、评价时间及评分3.7分，评价文字为“按时上门，干净麻利，价格实惠，态度极佳，不错不错~”。下方有订单编号、预约时间及地址信息。该图与上文“服务完成，用户进行评价”内容对应，直观呈现了评价界面样式及信息展示情况。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/AHcqbX0spopfwhxbV3yc4zcOnwe.png)





### 6）小结

能够说出项目的业务领域：

你的项目是做什么业务的？



能够说出项目涉及几类用户角色

项目涉及哪些角色？



能够说出项目包括哪些业务模块

你的项目包括哪些模块？



能够说出项目整体业务流程：

说下你的项目的业务流程？



## 1.2 项目的架构

目标：能够用自己的话介绍自己的项目



### 1）项目架构

项目是基于Spring Cloud Alibaba框架构建的微服务项目，采用前后端分离模式进行开发，系统架构图如下：

![这张图是云医到家的项目架构图，对应文档中介绍的基于Spring Cloud Alibaba的前后端分离微服务项目。图中明确划分出用户层、负载层、服务层、数据源层及第三方模块：用户层包含运营端（PC）、服务端（APP）、用户端（小程序）；负载层为Nginx；服务层的接入层采用Spring Cloud Gateway，其下分为业务服务与基础服务两大模块，业务服务涵盖运营基础管理、客户管理等功能，基础服务包含服务注册、配置中心等组件；数据源层以MySQL为核心，搭配分库分表的历史订单、消息队列等模块；右侧第三方模块则包含微信、第三方支付等服务。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/RvMWbxGd8owdu5x5CbRc9rrhnAc.png)

用户层：

包括四个端：运营端(PC)、服务端（APP）、机构端（PC）、用户端（小程序）

负载层：

反向代理、负载均衡。

服务层：包括网关、业务微服务、基础服务。

业务微服务：包括**运营基础服务**、**客户管理服务**、**订单管理服务**、**抢单服务**、**派单服务、支付服务、搜索服务**等。

基础服务：Nacos（服务注册、配置中心）、XXL-JOB（任务调度）、RabbitMQ（消息队列）、Elasticsearch（全文检索）、Canal（数据同步）、Sentinel（熔断降级、限流）等。

数据层：

MySQL数据库存储：服务信息、区域信息、客户信息、订单信息、支付信息、抢单池、派单池、结算信息等。

分库分表：使用ShardingShphere-JDBC 进行分库分表。

消息队列：存储数据同步消息、各类异步消息等。

索引：服务信息、服务提供者信息、订单信息等。

缓存：服务信息、订单信息、服务单信息等。



### 2) 核心交互流程

下图是项目的核心交互流程，从下图找到核心交互流程是什么，以及参与的微服务有哪些，并用自己的话描述出来。

<readonly-block type="diagram"></readonly-block>



### 3) 小结



能够说出项目用的什么技术架构

你的项目用的什么架构？



能够用自己的话介绍自己的项目

说说你的项目吧？



在面试中被问的第一个问题就是项目介绍。怎么介绍自己的项目呢？

从以下几个方面进行项目介绍：

1、项目的背景，包括：是自研还是外包、什么业务、服务的客户群是谁、谁去运营等问题。

2、项目的业务模块、核心业务流程，可从项目每个端开始介绍。

3、项目的技术架构是什么

4、个人的工作职责，详细说明自己负责的模块，包括模块的设计，所用到的技术方案，以及所遇到的问题和解决方案。

一个例子：

我最近参与的项目是我们公司自研的家政服务o2o项目，项目包括四个端：用户端(小程序)、服务端（app）、机构端(PC)、运营管理端(PC)，用户通过平台在线下单、支付，家政服务人员通过平台抢单，平台也会自动派单给服务人员，服务人员接到订单根据预约时间去现场服务，服务完成后用户进行在线评价，以及售后、退款等业务。  项目是基于Spring Cloud Alibaba框架构建的微服务项目，包括了运营基础服务、客户管理服务、订单管理服务、抢单服务、派单服务、优惠券服务、统计服务、评价服务等服务，系统还使用Redis、MQ、XXL-JOB、Elasticsearch、Canal、MongoDB等中间件。

我在这个项目中负责了xxx模块的设计与开发...

下边我说下xxx模块的具体设计吧....



# 2 开发环境搭建

目标：理解搭建环境的步骤，参考文档搭建开发环境。

参考“家政o2o项目开发环境配置v1.0”搭建开发环境。



# 3 熟悉需求与设计

通常刚到企业中会分配一个独立的模块给你，这个模块一般会有一些业务逻辑但不会太复杂，下边以运营基础管理模块为例进行讲解。

## 3.1 熟悉需求

目标：阅读需求文档，理解基本概念和业务流程。

### 1）熟悉需求文档

参考“运营基础管理模块用户需求文档v1.0”熟悉需求。

如何去熟悉项目的需求？

根据产品原型、用户需求文档去熟悉项目的功能需求、业务流程，如果有测试账号可以登录系统通过体验系统的功能去熟悉项目的需求。



### 2）口述

下边的问题口述答案。

什么是服务？

什么是服务类型？

向区域添加服务流程是什么？

区域服务上架的流程是什么？

区域服务下架的流程是什么? 

服务删除的流程是什么？

服务状态转化关系是什么？



## 3.2 熟悉设计

目标：

理解运营基本管理模块的表结构。

熟悉工程结构。

能够对接口进行断点调试。



### 1) 熟悉表结构

使用客户端连接数据库，根据需求去理解本模块的每一张表，

![图片展示了数据库中“jzo2o-foundations”下的“tables”文件夹内容。文件夹内有8个表，分别为city_directory、config_region、operator、region、serve、serve_item、serve_sync、serve_type。其中，serve_item、serve_sync、serve_type表被红色框线突出显示。该图片与上文“使用客户端连接数据库，根据需求去理解本模块的每一张表，然后尝试梳理表之间的关系”的内容相关，直观呈现了数据库中表的结构，辅助理解表之间的关系。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/RP60bGfspoXO9QxtyD8cxo10nad.png)

然后尝试梳理表之间的关系。

如下图：

![这张图展示了项目中四个数据表的结构，清晰呈现了表间的关联关系。其中serve_type是服务类型表，serve_item是服务项表，二者通过serve_type_id建立一对多关系（一个服务类型对应多个服务项）；region是区域表，serve表分别通过region_id与region表、serve_item_id与serve_item表建立关联。每个表都标注了字段名称、数据类型，以及字段对应的说明，直观呈现了各表的字段设计和相互关系，与上下文介绍的表结构内容相呼应。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/Sfy0bmTMloXuc9x9GuIcDkbQnDd.png)

**serve_type**：服务类型表

**serve_item**:  服务项表，存储了本平台的家政服务项目

每个服务项都有一个服务类型，一个服务类型下有多个服务项，服务类型与服务项是一对多关系。

**region**：区域表，存储运营地区信息，一般情况区域表行政级别是市。

**serve**:  服务表，存储了各个区域运营的服务及相关信息。

**注意：这里不要把serve表简单理解为只是区域表和服务项表的中间关系表，因为如果是简单的关联关系表只需记录区域表和服务项表各自的主键Id即可，serve记录的是平台运营服务的信息，凡是与运营相关的信息都要记录在serve表，比如：运营价格。后期也可能会增加其它运营相关的字段。**

region与serve_item是什么关系？

一个区域下可以设置多个服务项，一个服务项可以被多个区域设置，region与serve_item是多对多关系。



### 2) 熟悉工程结构

项目的工程列表如下：



首先熟悉jzo2o-foundations运营基础服务工程的结构，jzo2o-foundations工程结构如下图： 

![图片展示了jzo2o-foundations项目的工程目录结构。在src目录下，main文件夹内有java文件夹，其下包含com文件夹，com文件夹中又有jzo2o文件夹，该文件夹下有多个子文件夹，如controller、handler、mapper、model、pagehelper、service等，其中service文件夹下有FoundationsApplication文件。此外，还有resources和test文件夹，以及target文件夹下的.gitignore、Dockerfile和pom.xml文件。该图与上文介绍的jzo2o-foundations工程结构内容相呼应。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/RcEgbDl6VoskJFxHms7ceiuLnLl.png)

工程目录结构如下：

![这是jzo2o-foundations运营基础服务工程的目录结构，整体分为main和resources两大模块。main模块下包含java包，以com.jzo2o.foundations为基础，划分为constants（工程用常量类）、controller（含agency机构端、consumer用户端、inner内部端、operation运营端、worker服务端等细分接口类）、handler（处理器类）、mapper（持久层mapper接口）、model（含domain持久化模型类、dto数据传输模型类）、service（业务服务接口与类），还有FoundationsApplication启动类；resources模块包含mapper（mapper接口的映射文件）、总配置文件bootstrap.yml，以及对应开发、生产、测试环境的bootstrap-dev.yml、bootstrap-prod.yml、bootstrap-test.yml。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/HO2ebhvjtoQ6skxzScRcFs11nof.png)

项目依赖：

```XML
 <dependencies>
        <!--服务发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--配置管理-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <!--开启Bootstrap配置文件的支持-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bootstrap</artifactId>
        </dependency>
        <!--jzo2o-mvc在jzo2o-framework下，针对web开发的依赖-->
        <dependency>
            <groupId>com.jzo2o</groupId>
            <artifactId>jzo2o-mvc</artifactId>
        </dependency>
        <!--knife4j用于生成swagger文档-->
        <dependency>
            <groupId>com.jzo2o</groupId>
            <artifactId>jzo2o-knife4j-web</artifactId>
        </dependency>

<!--        <dependency>-->
<!--            <groupId>com.jzo2o</groupId>-->
<!--            <artifactId>jzo2o-es</artifactId>-->
<!--        </dependency>-->

        <!--单元测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--api接口,定义了服务之间的远程调用接口-->
        <dependency>
            <groupId>com.jzo2o</groupId>
            <artifactId>jzo2o-api</artifactId>
        </dependency>
        <!--序列化工具库-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>

        <dependency>
            <groupId>com.jzo2o</groupId>
            <artifactId>jzo2o-redis</artifactId>
        </dependency>
<!--        <dependency>-->
<!--            <groupId>com.jzo2o</groupId>-->
<!--            <artifactId>jzo2o-canal-sync</artifactId>-->
<!--        </dependency>-->
        <!--jzo2o-mysql定义在jzo2o-framework工程，定义了持久层相关的依赖-->
        <dependency>
            <groupId>com.jzo2o</groupId>
            <artifactId>jzo2o-mysql</artifactId>
        </dependency>
<!--        <dependency>-->
<!--            <groupId>com.jzo2o</groupId>-->
<!--            <artifactId>jzo2o-xxl-job</artifactId>-->
<!--        </dependency>-->
    </dependencies>
```



持久层：MySQL数据库，mybatis-plus框架，com.github.pagehelper分页组件

中间件：Redis、Elasticsearch、xxl-job

服务层：通过Spring进行事务控制，redisson分布式锁、Spring Cache缓存框架

web层:  SpringMVC框架（基于SpringBoot开发）

web容器：undertow（Undertow 是一个采用 Java 开发的灵活的高性能 Web 服务器，红帽公司的开源产品）

高并发场景下undertow的性能更好。



### 3) 接口测试

搞清楚了项目的工程结构再通过一个具体的接口去读项目的代码。

具体可以把工程启动起来通过接口测试深入源码搞清楚执行流程。

启动jzo2o-foundations工程。

通过swagger接口文档去熟悉模块的接口，通过测试接口去理解接口的整个交互流程。

打开接口文档：http://localhost:11509/foundations/doc.html

![这张图展示了本地运行的家政服务-运营基础服务接口文档页面，打开的网址为localhost:11509/foundations/doc.html#/home。页面左侧是导航菜单，包含“Swagger Models”“文件管理”等选项，其中“内部接口-服务类型相关接口”的选项被高亮显示；页面右侧为文档详情区域，显示文档名称为家政服务-运营基础服务接口文档，还标注了简介、作者、版本、host、basePath等接口相关信息，左侧菜单是查找“运营端-服务类型相关接口”的入口，对应文档中通过接口文档查找目标接口的操作步骤。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/A64sbM6gGoFHwoxOltGcvPyDnOh.png)

通过左侧菜单找到“运营端-服务类型相关接口”，如下图：

![这张图片展示的是“家政服务-运营基础服务接口文档”的页面，当前页面显示的是“服务类型分页查询”接口的信息，属于接口测试流程中查看指定接口文档的环节。图片左侧的菜单中，“运营端-服务类型相关接口”下的“GET 服务类型分页查询”选项为选中状态，页面右侧则清晰呈现了该接口的请求方式为GET，接口地址为/foundations/operation/serve-type/page，同时还展示了该接口的请求参数、参数说明、是否必选等相关信息，以及响应状态等内容。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/LiVhbmacJoKwOyxUaKXc1fSdnzd.png)

点击“服务类型分页查询”打开接口文档，如下图：

![该图片展示的是“家政服务-运营基础服务接口文档”中的“服务类型分页查询”接口页面，接口的请求方式为GET，接口地址是/foundations/operation/serve-type/page。页面呈现了该接口的请求参数、响应数据类型及响应状态等内容，其中请求参数包含isAsc1、isAsc2、orderBy1、orderBy2、pageNo、pageSize等，各参数标注了对应参数说明、请求类型、是否必填、数据类型等信息，响应状态则列出了200、401、403、404等状态码及对应的说明。该图片与上下文内容匹配，是打开“服务类型分页查询”接口文档后显示的具体内容，对应文档中点击“服务类型分页查询”后的图示环节。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/ZWi6bjjHKoD5OaxmTJOco98Pn1R.png)

找到接口地址：/foundations/operation/serve-type/page

/foundations：微服务的content-path路径

/operation：表示给运营端使用的接口

/serve-type/page：具体的接口地址

在jzo2o-foundations工程搜索“/serve-type”关键字

![图片展示了在IntelliJ IDEA中对“jzo2o-foundations”项目进行操作的界面。左侧是项目结构，选中“jzo2o-foundations”后，右侧弹出快捷菜单，其中“Find in Files...”选项被高亮显示，对应快捷键Ctrl+H。该图片与文档中“在jzo2o-foundations工程搜索‘/serve-type’关键字”内容相关，用于说明在项目中搜索关键字的操作步骤，即通过快捷菜单的“Find in Files...”功能进行搜索。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/UgCOb3ABLoIap5xItvoctB0hncc.png)

输入关键字如下图，找到了具体的接口类：

![图片展示了在jzo2o-foundations工程中搜索“/serve-type”关键字的结果。搜索范围为D:\\course\\jzo2o\\code\\jzo2o-foundations目录，共找到3个匹配项，分别在worker、inner、operation目录下的ServeTypeController.java文件中。其中，operation目录下的ServeTypeController.java文件中，@RequestMapping注解标注的“/operation/serve-type”接口地址被红色箭头指向突出显示。该图片与上文在jzo2o-foundations工程搜索“/serve-type”关键字的内容相关，直观呈现了搜索结果及关键接口地址。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/XuGvbMAakorOwsxCpV7cVP1cncW.png)

继续在ServeTypeController.java类中找/page接口，如下图：

![这张图片展示了在ServeTypeController.java类中的Java代码片段，对应文档中需查找的“服务类型分页查询”接口相关内容。代码的关键部分包括带有路径的@GetMapping注解，注解中明确路径为"/page"；还有@ApiOperation注解，注释说明该接口的功能是“服务类型分页查询”；方法签名定义了返回类型为PageResult<ServeTypeResDTO>，方法名为page，入参为ServeTypePageQueryReqDTO类型参数；方法内部仅调用了serveTypeService的page方法，并将入参作为参数传入，该内容对应文档中继续在ServeTypeController.java类中查找/page接口的操作结果。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/RtDDb0nrnorFrfx3yZGcoWoenHh.png)

下边debug运行工程，跟踪接口执行的流程：

首先打上断点，如下图：

![这张图片展示了Java代码中的服务类型分页查询接口的方法内容，是ServeTypeController.java类的一部分，用于服务类型分页查询功能。代码中，该方法标注了`@GetMapping("/page")`的映射路径，以及`@ApiOperation("服务类型分页查询")`的接口说明，方法接收`ServeTypePageQueryReqDTO`类型的请求参数，返回封装分页结果的`PageResult`对象，方法内调用`serveTypeService`的`page`方法来实现业务逻辑，红色圆点为断点标识，对应文档中要求在该位置打上断点进行调试的操作。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/G65RbGkNMoqqZOxrUIecOljznId.png)

debug运行jzo2o-foundations工程。

下边通过swagger接口文档测试该接口

进入“调试”窗口，点击“发送”

![图片展示的是家政服务运营基础服务接口文档中“服务类型分页查询”接口的调试窗口。窗口中显示了GET请求的参数，如isAct1、isAct2等，部分参数有默认值。右上角有“发送”按钮，图片中红色箭头指向该按钮。该图片与上下文紧密相关，上下文提到“下边通过swagger接口文档测试该接口，进入‘调试’窗口，点击‘发送’”，此图直观呈现了点击“发送”按钮的操作位置，是接口测试流程中的关键步骤示例。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/GkJnbCQ0DoxgWPxIauNc6Qr5nqe.png)

此时代码执行到断点处

![这张图片展示了IDEA开发工具中的Java代码界面，对应服务类型分页查询的接口方法实现代码。图中蓝色高亮选中的代码行，是名为page的公共方法，该方法接收ServeTypePageQueryReqDTO类型的请求参数，返回PageResult<ServeTypeResDTO>类型的结果，方法内部调用了serveTypeService的page方法完成业务逻辑处理，代码上方还标注了对应的GetMapping注解与ApiOperation注解，分别对应接口路径为"/page"、接口功能为服务类型分页查询。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/GfrAbr4b4ov5dCxWpHpcM6sBnHd.png)

进入service方法继续打断点跟踪：

![图片展示的是“ServeTypeServiceImpl.java”代码文件内容。其中，关键代码部分被红色框线突出显示，代码为“Page<ServerType> page = PageUtils.parsePageQuery(serveTypePageQueryReqDTO, ServerType.class);”。该图片与上文“进入service方法继续打断点跟踪”相关，是对上文提到的在“ServeTypeServiceImpl.java”文件中跟踪请求参数这一操作的代码示例呈现，帮助理解代码执行情况。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/DTvtbYpNxoiE9KxQmm2cYUrAnWg.png)

跟踪请求参数：

![图片展示的是一个Java代码片段，其中`page`方法接收`ServeTypePageQueryReqDTO`参数。代码中有一个断点，箭头指向`serveTypePageQueryReqDTO`参数。下方弹出的参数信息显示了`pageNo`、`pageSize`等属性值，如`pageNo`为1，`pageSize`为10。该图片与上文“进入service方法继续打断点跟踪，跟踪请求参数”的内容相关，直观呈现了请求参数的显示情况。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/YgUabc8iDoQCAFxzSiMc4Vw0n4f.png)

放行继续执行，测试结束，观察响应结果

![图片展示的是家政服务 - 运营基础服务接口文档中“调试”窗口的请求参数及响应结果界面。上方有“请求参数”和“AfterScript”选项卡，下方显示了多个参数，如isAsc1、isAsc2等。下方“Raw”区域以JSON格式呈现了响应结果，包括code、msg、data等字段，data下有多个对象，如id、name、status等，还显示了响应时间、响应码、耗时等信息。该图与上下文介绍的通过swagger接口文档测试接口，进入“调试”窗口并观察响应结果等内容相关。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/GKG3bIl1doxycyxr3Kzciz3Tnme.png)

观察idea控制台输出的SQL执行语句

![这张图片是项目测试过程中Idea控制台的输出内容，清晰展示了两次SQL执行语句：第一次是查询serve_type表数据总数的语句，第二次是分页查询该表指定字段数据的语句，同时包含对应参数信息。最后还呈现了接口测试的响应结果，其中状态码为200，消息为OK，数据分页信息显示总共有3条数据，对应接口测试放行后的最终结果，和文档中提及的观察Idea控制台SQL执行语句的要求相契合。](../../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/di-yi-zhang-ru-he-shu-xi-xin-xiang-mu/JxmwbtlgBojPGJxIh7xcJX2mnfh.png)



### 4) 小结

如何去熟悉系统的设计？

根据需求去熟悉本模块的核心表的结构，核心表搞清楚了再去熟悉其它表。

熟悉项目工程结构，搞清楚用的基础框架是开源的还是自研的，通过断点测试接口的执行流程阅读读项目的代码。

具体可以把工程启动起来通过接口测试深入源码搞清楚执行流程。



如何对现有接口进行断点调试?



运营基础工程用到哪些技术？

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 云岚到家项目 / 第一章 如何熟悉新项目 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/VQ6xw3B3Fixprnk4s8scalqGnmd)
