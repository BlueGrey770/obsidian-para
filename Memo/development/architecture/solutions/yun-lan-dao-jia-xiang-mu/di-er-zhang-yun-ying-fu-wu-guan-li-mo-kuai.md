---
type: memo
date: 2026-11-06
aliases:
  - "第二章 运营服务管理模块"
---

<title>第二章 运营服务管理模块</title>

# 学习目标

运营服务管理模块是我们在本项目学习的第一个模块，运营服务管理模块相当于商品管理，商品管理功能很通用，最典型的就是电商项目中的商品管理，家政项目中的家政服务项目相当于商品，凡是涉及消费用户的项目都包括商品管理模块，通过本章节的学习掌握商品管理模块的设计、开发细节。

能够对运营服务管理模块进行需求分析

能够说出运营服务管理模块的设计内容

能够设计开发区域服务子模块的接口

能够设计开发运营服务管理模块的其它接口



学习目标如下：

1. 能够说出服务上架的流程
2. 能够说出家政项目服务管理涉及的表有哪些
3. 能够设计并开发区域服务查询接口
4. 能够说出前后端联调步骤
5. 能够设计并开发添加区域服务接口
6. 能够设计并开发修改价格接口
7. 能够设计并开发服务上架接口
8. 能够设计并开发家政项目实战功能
9. 能够说出电商项目商品管理涉及的表有哪些
10. 能够说出电商项目商品发布的流程
11. 能够设计并开发电商项目实战功能







家政项目实战：

删除区域服务

服务下架

设置热门服务

取消热门服务

启用区域功能完善

禁用区域功能完善

禁用服务项功能完善



# 代码环境

jzo2o-foundations：dev_01



# 1 前置知识

## 1.1 如何进行需求分析

### 1）如何进行需求分析？

在系统设计之前需要进行需求分析，需求分析就是根据用户所提的需求进行理解，它是软件开发周期中的关键环节，它直接影响到后续的设计、开发和测试阶段。需求分析阶段做好了能够减少后期修改的成本，提高系统的质量和用户的满意度。因此，在需求分析阶段投入足够的时间和资源是非常必要的。

需求分析通常从以下方面进行：

- 理解业务需求：与客户或产品经理沟通，确保你对业务流程有清晰的理解。

产品经理会召集我们一起开会讨论确定产品原型，我会根据产品原型去对每个功能进行分析。

- 梳理功能需求：将业务需求转化为具体的软件功能，划分模块，明确哪些是核心功能，哪些是辅助功能。
- 梳理用户操作事件流：在功能需求的基础上去梳理用户的操作流程。
- 分析对象属性：根据业务流程分析涉及到的对象有哪些及对象属性。
- 分析非功能性需求：比如性能要求、安全性、可维护性等。

### 2）产品原型

与客户沟通最好的方式就是通过界面原型进行沟通，因为界面原型客户看的懂，它是需求阶段最接近软件成品产物。

在使用界面原型进行沟通时，也需要确保技术术语和设计细节能够被非专业人士所理解。此外，应该保持原型的简洁性和易用性，避免过度复杂化导致信息传递失效。

除了通过界面原型进行沟通，还要与其他沟通方法（如会议讨论、文档说明等）相结合，以确保所有方面的需求都被准确无误地传达给项目团队。

产品原型网：https://www.axureshop.com/ 



### 3）**分析功能列表**

通常按不同的端进行分析，例如下表是云岚到家项目消费端的功能列表（部分功能）：

![这张图片是云岚到家项目消费端部分功能列表表格，包含模块、子模块、功能描述、优先项四列内容，按不同模块分类展示相关功能。模块涵盖登录、定位、搜索、消息、全部服务、个人中心、地址管理等类别，各模块下设有对应子模块，如定位模块包含自动定位、重新定位、热门城市等子模块，消息模块包含一键清空、消息列表等子模块，全部服务模块包含服务分类、热门列表等子模块。表格对每个子模块的功能进行了具体描述，还标注了对应的优先项，方便明确各项功能的优先级，该表格用于辅助开展项目需求分析的功能列表分析工作。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/O6VSbi0G3o7Q8AxZUqBcs96qnnd.png)



### 4）**模块化设计**

模块化设计是在需求分析阶段对功能需求划分模块：

将相关的需求划分到一个模块中，模块下可以有子模块。

举例：

运营服务管理模块的定位是为对平台运营的服务进行管理，功能包括：服务分类管理、服务管理、区域管理等子模块。

运营服务管理是一个模块，其下又分为服务管理、区域管理等子模块，服务管理模块又包括了服务类型管理模块、服务项管理模块等多个子模块。

产品经理根据需求梳理模块清单：

![图片是一张功能模块表，展示了运营服务管理模块下各子模块的对应功能点及优先级。功能点包括登录、服务管理、服务人员管理、业务设置、机构管理、订单管理、售后管理、财务管理、通知沟通、客户中心等，如登录模块有账号密码登录、验证码登录、手机号登录等功能，优先级从P0到P3不等。该表与上下文紧密相关，是对上文提到的运营服务管理模块下各子模块功能的详细说明，为后续需求分析提供依据。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/LMvCbntIpoTbSixs4eFc8OAKnpc.png)

根据每个端去细化模块，项目业务模块图如下：

![图片展示了运营服务管理项目业务模块图。图中以不同颜色区分端口，包括小程序端、APP端、PC端及管理后台PC端。各端口下有登录、基础配置、订单管理、服务管理、服务人员管理、企业管理、客户管理、营销管理等模块，如小程序端有服务搜索、城市定位、三方支付等，APP端有实名认证、服务功能配置等。该图与上下文紧密相关，直观呈现了各端口的业务模块及对应功能，为后续需求分析提供参考。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/YgjebLaLEor3tYxMwWYcI1uGnpd.png)



### 5）细化需求

#### 完善界面原型

和用户确定了项目范围，下一步就正式进入需求分析阶段，产品经理对界面原型进行完善，细化原型中的参数说明、对象属性、规则、约束、分析用户操作流程等。

举例：

![该图片是运营服务管理模块中需求分析环节里，用于完善界面原型的表单示例图，展示了需完成的界面字段内容，包括带红色星号的必填项，分别为服务名称（需输入）、服务类型（需选择）、服务单位（需选择）、排序（需输入并可选序号调整）、服务图标（需上传jpg或png格式、200K以内的图片），还有参考服务单价（需输入）以及服务图片的上传区域，这些字段用于细化原型中的各项设置参数。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/PxxDb1SpAohYTqxa0Z6c7NCbnPg.png)



![图片为服务管理模块中服务详情的规则说明表。表中列出了服务名、服务类型、参考服务单价、单位、排序、图标、服务图片、服务描述、服务详情长图等字段的必填/选填、唯一、限制要求及提示内容。如服务名必填且唯一，限制要求为字数2 - 5个汉字，超出无法输入，提示内容为字段名称格式错误，需重新输入。该表与上文提到的对界面原型进行完善，细化原型中的参数说明等内容相呼应，是对服务详情规则的详细说明。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WWvubMCzoo2KSyxeQg4chL8ZnYb.png)

#### 用户操作流程

还需要细化用户的操作流程。

![图片展示了添加服务的界面及操作说明。界面上方提示点击添加服务弹出添加服务弹窗，添加服务为按核销样式，选择服务类型添加服务，搜索服务仅对该服务类型下进行搜索。下方有维修、保洁等服务类别，如维修类别下有日常维修、深度维修等服务，保洁类别下有日常保洁、深度保洁等服务，每项服务后有价格。下方还有“添加”和“取消”按钮。图片下方对应步骤1的说明，详细介绍了添加服务的操作及服务项显示规则。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HBl3b1qfzoNwDGx9aJsc4MCxnCd.png)



### 6）非功能性需求

除了功能需求还有非功能需求，比如性能要求、安全性、可维护性等这些也需要在需求阶段进行分析。

举例：

用户登录功能，为了保证安全性对密码加密传输、加密保存。

app首页为了提高动态数据的加载速度，保证首页3秒内加载完毕。



### 7）需求阶段产物

需求分析阶段的产物：

产品原型。

需求规格说明书。



AI: 给我一份需求规格说明书的例子，内容要足够详细



### 8）实战

阅读“运营服务管理模块用户需求文档v1.0”，进行区域服务管理子模块进行需求分析，要求如下：

1. 列出功能列表，划分子模块
2. 分析每个功能用户的操作流程，并写下来
3. 分析对象属性，并写下来

举例：

功能：服务类型列表：

![图片展示的是区域服务管理子模块中服务类型列表界面。列表包含服务类型编号、服务类型、服务类型图标、服务类型图片、排序、状态、更新时间等字段，如服务类型编号为FWLX001的服务类型是家政服务，状态为已启用，更新时间为2053年5月30日21:50。界面右上角有“新建”按钮，下方有分页导航栏，当前显示第2页，共15条记录。该图与上文需求分析中列出功能列表、分析操作流程及对象属性等内容相关，直观呈现了服务类型列表界面情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EZLbbhO29ocL6qxG8RtcDtmznig.png)

操作流程：

点击菜单进入服务类型列表

默认查询全部服务类型列表

通过上一页、下一页进行分页查询

对象属性：

查询列表字段包括：服务类型编号、服务类型名称 、服务类型图标、服务类型图片、排序、状态、更新时间。



举例：

功能：新建服务类型

![这张图片展示的是新建服务类型的操作界面，该界面属于运营服务管理模块区域服务管理子模块的需求分析相关示例内容。界面标题为“新建服务类型”，设有多个必填项，包括用于填写服务类型名称的“服务类型”输入框、用于输入数字的“排序”输入框，还提供了服务类型图标和服务类型图片的上传入口，其中图标上传要求为jpg/png格式文件且大小在200K以内，图片上传要求为jpg/png格式文件且大小在5M以内，界面底部配有“取消”和“保存”两个操作按钮。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/IhLBbAH3EozgfsxmMmKcWrjKnUh.png)

操作流程：

点击菜单进入服务类型列表

点击“新建”打开新建服务类型窗口

填写“服务类型”、"排序" 信息。

上传服务类型图标、服务类型图片。

点击“保存”提交新建的服务类型信息

点击“取消”关闭窗口。



对象属性：

服务类型名称、排序、服务类型图标、服务类型图片。



### 9) 小结

你参与需求分析了吗？如何进行需求分析？





## 1.2 如何进行系统设计

### 1) 如何进行系统设计？

需求分析阶段完成是系统设计阶段，系统设计分为概要设计和详细设计。

AI:

软件的概要设计和详细设计分别要设计哪些内容

使用SpringBoot开发软件，在概要设计和详细设计阶段分别要设计哪些内容



**概要设计**是对系统模块、架构、数据库表、系统间的交互接口、外部接口等宏观层面进行设计。

**详细设计**是对具体类、方法、接口等微观层面的实现细节进行设计。

通常项目都会进行概要设计，详细设计阶段的设计内容根据公司的项目管理水平而不同。

我们在教学过程中将概要设计和详细设计统称为系统设计。

系统设计的核心通常包括以下几个方面：

1. 系统模块设计

根据需求划分系统功能模块。

1. 系统架构设计

确定基础框架、中间件等技术选型、外部接口和内部接口、工程结构等。

1. 数据库设计

划分数据库，设计表，这项是中高级程序员的主要工作。

1. 接口设计

在概要设计阶段主要设计外部接口及模块间的部分内部接口。

系统外部接口，比如：调用云平台提供的文件上传接口、微信小程序认证接口、用户定位接口等。

系统模块之间的内部接口，比如：订单模块如何获取商品管理模块的商品信息，订单模块如何获取支付结果。

在详细设计阶段根据需求设计具体的业务接口及模块间的内部接口，这项是中高级程序员的主要工作。





### 2) **系统架构设计**

系统的业务模块划分好后就可以根据业务模块的划分进行系统架构设计：

- 定义应用架构：确定应用的结构（如单体应用、微服务架构）。
- 技术选型：选择合适的框架和技术栈，Spring Boot本身已经提供了很多开箱即用的功能，但你需要选择适合的数据库、缓存机制、消息队列等。
- 模块化：根据业务模块的划分将系统划分为不同的模块或服务，每个模块负责一部分功能，这有助于未来的扩展和维护。
- 设计内外部接口：分析所依赖的外部接口，分析系统内模块之间的交互接口。

举例：

云岚到家是微服务项目，根据业务模块去划分微服务。

技术选型：使用Spring Cloud Alibaba框架，数据库使用MySQL，还使用RabbitMQ、Redis、Sentinel、XXL-JOB、ShardingSphere-JDBC等中间件和框架。

单独设立第三方集成模块，用于和外部平台进行接口对接。

![图片展示了云岚到家项目的系统架构。用户层有PC、APP、小程序，负载层是Nginx。接入层为Spring Cloud Gateway，业务服务包括运营基础管理等，基础服务有服务注册等。服务层有MySQL数据库，数据层有服务、区域等表。右侧有第三方集成模块，包括微信、第三方支付等。该图与上下文紧密相关，直观呈现了项目架构，为后续设计内外部接口、技术选型等提供参考。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HXaIbM91rotGjLxl54wcKlknnvc.png)



### 3) **数据库设计**

根据需求分析及模块设计进行数据库设计：

- 微服务项目不同的模块设计不同的数据库
- 根据业务需求设计数据库表、及表结构。
- 考虑到数据的一致性和完整性，设计适当的约束条件。
- 性能优化：考虑索引、分区等技术以提高查询效率。

举例：

云岚到家是微服务项目，每个模块有自己独立的数据库

![图片展示了云岚到家运营服务管理模块的数据库设计中部分表结构。其中，“jzo2o-order”表下有“jzo2o-order-0”“jzo2o-order-1”“jzo2o-order-2”等子表，还包含“jzo2o-order-history”“jzo2o-trade”等表。图片中用红色框突出显示了“运营管理”“营销管理”“订单管理”三个类别，与上下文提到的运营服务管理模块核心表相呼应，直观呈现了相关表的分类情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/UA9ZbKPEBoBitkxMcw9cv3v0ncf.png)

运营服务管理模块核心表如下：

![图片展示了运营服务管理模块核心表的数据库设计。包含serve_type（服务类型表）、serve_item（服务项表）、region（区域表）和serve（服务表）四个表。serve_type表有服务类型代码、名称、图标等字段；serve_item表存储服务项目信息，如服务类型ID、名称、图标等；region表记录运营地区信息，有城市代码、名称、负责人等字段；serve表关联服务项和区域，记录服务价格、是否热门等信息。该图与上下文介绍的运营服务管理模块核心表设计内容相契合。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/E8qJblXHYonSu0xjEcwcVQfSntc.png)

**serve_type**：服务类型表

**serve_item**:  服务项表，存储了本平台的家政服务项目

每个服务项都有一个服务类型，一个服务类型下有多个服务项，服务类型与服务项是一对多关系。

**region**：区域表，存储运营地区信息，一般情况区域表行政级别是市。

**serve**:  服务表，存储了各个区域运营的服务及相关信息。

**注意：这里不要把serve表简单理解为只是区域表和服务项表的中间关系表，因为如果是简单的关联关系表只需记录区域表和服务项表各自的主键Id即可，serve记录的是平台运营服务的信息，凡是与运营相关的信息都要记录在serve表，比如：运营价格。后期也可能会增加其它运营相关的字段。**

region与serve_item是什么关系？

一个区域下可以设置多个服务项，一个服务项可以被多个区域设置，region与serve_item是多对多关系。



### 4) **接口设计**

根据需求分析梳理功能列表、分析用户操作流程，然后再去设计外部接口及功能接口。

包括以下几个方面：

**外部接口：是本项目与第三方系统之间的接口。**

举例：在上传服务项图片时涉及外部接口，下图上传图片功能如果将图片保存到云平台则需要调用云平台的上传图片接口。

![图片展示的是系统中服务管理模块的新增服务界面。界面上方有“服务名称”“服务类型”“参考服务单价”“服务单位”“排序”“服务图标”“服务图片”等输入框或下拉选择框，部分框内有提示文字。其中“服务图标”和“服务图片”下方各有一个“上传图片”按钮，提示上传jpg/png文件且大小在200K或5M以内。该图片与上下文介绍的系统设计中接口设计相关，用于说明在详细设计阶段对内部接口进行分析、细化时，服务管理模块新增服务的填写界面情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Y5CibvJ8dorJWxxTQrYctgsZnPf.png)

**内部接口：是本项目模块与模块之间的接口定义**

在概要设计阶段通常是根据业务流程和系统架构设计去分析，在详细设计（比如接口设计）时还会对内部接口进行分析、细化。

举例：

通用服务jzo2o-publics提供上传文件、微信认证、用户定位接口，其它微服务与jzo2o-publics服务交互，jzo2o-publics与外部系统交互。

<whiteboard token="TiunwTH5ZhO1RpbRup1ctjxpnEc"></whiteboard>

**功能接口：根据需求定义HTTP 功能接口。**

设计RESTful API：用户操作一般伴随着接口请求，一般是根据用户操作事件流去设计接口，定义接口的URL、HTTP方法、请求参数和响应格式。

举例：根据新建服务类型的需求设计接口如下图：

![这是家政服务运营基础服务接口文档中的服务类型新增接口页面，展示了RESTful API的具体设计内容，契合文档中根据新建服务类型需求设计接口的说明。该接口的HTTP方法为POST，接口URL为“/foundations/operation/serve-type”，请求数据类型为application/json。页面标注了该接口的请求参数，包括serviceTypeUpperServeRDTO、img、name、sortNum，这些参数分别对应服务类型更新请求DTO、服务类型图片、服务类型名称、排序字段，且所有参数的请求类型均为body，是否必填均为true，同时明确了各参数的数据类型。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Evu0b4vPnoXXehxZsGYcDf2Jnmh.png)



### 5) 小结

你在进行设计时都设计哪些内容？



能够说出运营服务管理模块的设计内容？



# 2 运营服务管理

通过本模块的学习掌握家政项目运营服务管理模块的数据库、接口等设计内容，以及基于本项目架构进行开发的流程。

通过本模块的学习掌握针对一个新模块如何进行设计、开发的流程和方法，在拿到一个项目时知道如何下手。



## 2.1 区域服务管理子模块设计

### 1）模块定位

区域服务管理模块是运营服务管理的子模块，它是以城市区域为单位，对本区域运营的服务项进行管理。

只在区域上架服务后该区域的用户才可以在小程序对该家政服务下单。



### 2）数据表

区域服务管理模块最基本的是区域表和服务项表，然后是区域与服务项的关系表，此关系表存储了区域内所运营家政服务项目的信息。

数据表如下：

![这张图呈现了区域服务管理子模块的核心数据表结构，展示了四个数据表的字段与关联关系。图中的serve_type表是服务类型表，包含服务类型编码、名称、状态等字段；serve_item表为服务项表，存储平台家政服务项目，含名称、状态、价格等字段；region表是区域表，包含区域编码、状态、排序等字段；serve表为区域与服务项的关系表，用于关联区域与对应的服务项。其中serve_type表与serve_item表通过serve_type_id关联，serve表通过region_id关联region表，通过serve_item_id关联serve_item表，对应了区域内家政服务项目的存储逻辑。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/FE7tbCACaonei0xeJcEcqPjWnnh.png)

**serve_type**：服务类型表

**serve_item**:  服务项表，存储了本平台的家政服务项目

每个服务项都有一个服务类型，一个服务类型下有多个服务项，服务类型与服务项是一对多关系。

**region**：区域表，存储运营地区信息，一般情况区域表行政级别是市。

**serve**:  服务表，存储了各个区域运营的服务及相关信息。

**注意：这里不要把serve表简单理解为只是区域表和服务项表的中间关系表，因为如果是简单的关联关系表只需记录区域表和服务项表各自的主键Id即可，serve记录的是平台运营服务的信息，凡是与运营相关的信息都要记录在serve表，比如：运营价格。后期也可能会增加其它运营相关的字段。**

region与serve_item是什么关系？

一个区域下可以设置多个服务项，一个服务项可以被多个区域运营，region与serve_item是多对多关系。



### 3）生成模型类

数据库表设计完成就可以使用工具生成模型类、mapper接口、mapper映射文件等。

生成模型类的工具有很多：AI、MyBatis逆向工程、IDEA插件等。

下边是使用MybatisPlus插件自动生成代码的过程。



**1、安装插件**

![图片展示了在IntelliJ IDEA中安装MyBatisPlus插件的界面。左侧导航栏选中“Plugins”，右侧显示搜索结果，MyBatisPlus插件被选中，版本为1.7.2，右侧有“Disable”“Overview”“What's New”“Reviews”“Additional Info”等选项。该图片与文档中“安装插件”步骤相关，直观呈现了在IDEA中搜索并选中MyBatisPlus插件的操作界面，是完成插件安装前的准备步骤。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NG63bat8yo6d7PxscOhc5Alfnxg.png)

**2.重启IDEA，连接mysql**

![图片展示了IntelliJ IDEA软件界面中Tools菜单下的部分选项。其中，“Config Database”选项被红色框和箭头突出显示。该图片与文档中“2.1区域服务管理子模块设计”章节下“3、配置代码生成规则”步骤相关，用于指导用户在IDEA中配置数据库连接，输入连接字符串“jdbc:mysql://192.168.101.68:3306/jzo2o-foundations?useSSL=false&serverTimezone=UTC”后，通过此菜单选项进行数据库配置，以完成代码生成规则的配置。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HIVXbmLWQoysGgxbZqZc9MqDnlc.png)

配置连接的数据库

输入连接字符串：jdbc:mysql://192.168.101.68:3306/jzo2o-foundations?useSSL=false&serverTimezone=UTC 

账号和密码

![图片展示的是配置连接数据库的界面。界面上方显示“配置连接的数据库”，下方有dbUrl、jdbcDriver、user、password等输入框，其中dbUrl输入了数据库连接地址，user为root，password被遮挡。下方有“test connect”按钮和“ok”按钮。界面底部弹出“test successful!”提示框，表明Mybatis Plus测试连接成功。该图片与文档中配置代码生成规则时选择表、设置生成代码根目录、设置包路径等内容相关，用于展示数据库连接配置情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EhYmbtYLios8ZWxCoBecIRPunQb.png)



3、配置代码生成规则

![图片展示了IntelliJ IDEA软件中Tools菜单下的Code Generator选项。菜单栏中“Tools”选项被红色箭头指向，其下拉菜单中“Code Generator”选项被红色框突出显示。该图片与文档中“配置代码生成规则”部分内容相关，用于说明在IntelliJ IDEA中选择Code Generator选项以进行代码生成配置的操作步骤，是代码生成规则配置操作的指引图示。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/JAmibZXTroih4exz81gcYPdpnRc.png)



![图片展示的是代码生成器界面，用于配置代码生成规则。界面中列出了多个表信息，如city_directory、config_region等，各表的create time、engine、coding等字段均有显示。下方有module、author、package等配置项，如generator设置为com.jzo2o.foundations，entity、service等选项被勾选。此外，还有lombok、restController等选项，以及code generator、check field等操作按钮。该图与上下文介绍的配置代码生成规则内容相关，直观呈现了配置界面及部分设置。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/TCVMbFwjTonfbtxhTkmc2xPynve.png)

1）选择表：上图选择serve_type表。

2）设置生成代码的根目录：上图设置generator

3）设置包路径：上图设置为com.jzo2o.foundations

4）主键生成策略：根据表中主键的生成策略进行选择，支持的主键生成策略如下图：

![图片展示了主键生成策略的选择界面，适用于主键类型为number或string的情况。界面中有“分配ID（主键类型为number或string）”和“分配UUID（主键类型为string）”两个选项卡。在“分配ID”选项卡下，有AUTO（ID自增）、NONE、INPUT（用户输入ID）三种策略；在“分配UUID”选项卡下，有ID_WORKER（全局唯一ID）、UUID（全局唯一ID）、ID_WORKER_STR（字符串全局唯一ID）三种策略。该图片与上下文介绍的主键生成策略选择内容相关，直观呈现了可选策略。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XzOjbFOTToCol4x1ifJc9f0yn9d.png)

本项目使用的MyBatisPlus版本支持前5个，对应MyBatisPlus源码如下：

```Java
public enum IdType {
    AUTO(0),
    NONE(1),
    INPUT(2),
    ASSIGN_ID(3),
    ASSIGN_UUID(4);
```

> - AUTO：基于数据库的自增主键
> - NONE: 不设置id生成策略
> - INPUT:用户手工输入id
> - ASSIGN_ID:雪花算法生成id(可兼容数值型与字符串型)
> - ASSIGN_UUID:以UUID生成算法作为id生成策略

本项目使用ASSIGN_ID方式，雪花算法生成id。

5）勾选要生成的类及包路径（Entity、Mapper、Controller、Service、ServiceImpl）。

6）勾选是否生成lombok注解、restController注解、swagger注解等。



点击“check field”选择表中的字段。

点击“code generatro”生成代码。

生成成功在项目工程根目录有一个generator目录，里边为生成的代码，如下图：

![图片展示了在IDEA中生成代码后，项目工程根目录下generator文件夹内的结构。其中，src文件夹下有main文件夹，其下依次包含java、resources文件夹，java文件夹内有com、mapper、model、service等子文件夹，各子文件夹下有对应代码文件，如controller下的ServeTypeController.java、mapper下的ServeTypeMapper.java等。该图与上文提到的生成代码成功后在项目工程根目录生成代码的内容相关，直观呈现了生成代码后的文件结构。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/GR83bbAvYoHM98xiJdbczkzUnld.png)

对生成的代码根据自己的需求稍加修改，修改后拷贝到工程相应的包下即可。



### 4)  外部接口

在上传服务项图片时涉及外部接口，下图上传图片功能如果将图片保存到云平台则需要调用云平台的上传图片接口。

![这张图片展示的是添加服务项的界面，对应文档中“在添加服务项界面中，上传服务项的图片会通过通用模块jzo2o-publics上传到OSS”的相关内容。界面内包含多项需填写或选择的内容：标有*的必填项有服务名称、服务类型、参考服务单价、服务单位、排序、服务图标、服务图片；其中服务图标上传按钮旁标注需上传jpg/png格式文件，大小在200K以内，服务图片上传按钮旁标注需上传jpg/png格式文件，大小在5M以内。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NIEbbawdMokVxVx39HJchUR2nrh.png)

### 5）内部接口

根据本模块的定位，本模块去管理区域下运营的服务项目，那么本模块就要提供以下接口供其它模块调用：

查询某区域下运营的服务项目

查询服务项目的详细信息等



本模块有没有需要调用其它模块的接口呢？

在添加服务项界面中，上传服务项的图片会通过一个通用模块jzo2o-publics去将图片上传到OSS中，这里本模块与通用模块jzo2o-publics交互也是一个内部接口。



总之：分析内部接口要根据系统架构和业务流程去分析一些核心重要的接口，从两个方面去分析：

本模块提供接口供外部模块调用。

本模块调用外部模块的接口。



### 6）功能接口

根据用户的操作流程分析功能列表，区域服务管理子模块功能列表如下：

1. 区域服务查询
2. 添加区域服务
3. 设置运营价格
4. 删除区域下的服务项
5. 设置热门服务
6. 取消热门服务
7. 上架服务
8. 下架服务

根据以上功能列表结合界面原型设计 HTTP 功能接口。



### 7）小结

用自己的话介绍一下区域服务管理模块的设计内容？



## 1.2 查询区域服务

下边我们根据模块功能列表进行开发，首先进行接口设计，然后进行开发。

### 1.2.1 梳理接口

设计接口前先梳理本模块涉及哪些接口，根据梳理出的接口制定工作计划。

根据用户操作流程及功能列表，区域服务管理包括以下接口：

1. 区域服务分页查询接口
2. 区域服务新增接口
3. 区域服务删除接口
4. 区域服务价格修改接口
5. 区域服务设置热门接口
6. 区域服务取消热门接口
7. 区域服务上架接口
8. 区域服务下架接口



### 1.2.1 接口设计

下边设计区域服务分页查询接口，企业中前后分离开发通常由java程序员设计接口，设计完成提供接口文档给前端工程师。

如何设计一个接口？

首先明确该接口是一个前后端交互接口，该接口由前端通过HTTP协议进行调用，前端去调用接口需要知道以下内容：

1. HTTP请求方法
2. 接口路径
3. 请求参数类型
4. 请求参数内容
5. 响应结果类型
6. 响应结果状态码
7. 响应结果内容

所以我们设计接口需要设计这些内容。

##### **HTTP请求方法**

根据RESTful规范:

查询方法一般用GET，请求参数比较多可用POST

新增方法用POST

修改方法用PUT

删除方法用DELETE

**本接口是一个分页查询接口，对查询请求通常我们用GET作为HTTP方法。**

由于我们用的swagger去生成接口文档，所以在接口设计时需要定义Controller类和方法。

具体的代码实现可以找一些现有的类参考，参考com.jzo2o.foundations.controller.operation.ServeTypeController类，下边定义ServeController 类

代码如下：

```Java
package com.jzo2o.foundations.controller.operation;


import com.jzo2o.common.model.PageResult;
import com.jzo2o.foundations.model.dto.request.ServePageQueryReqDTO;
import com.jzo2o.foundations.model.dto.response.ServeResDTO;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * <p>
 * 前端控制器
 * </p>
 *
 * @author itcast
 * @since 2023-07-03
 */
@RestController("operationServeController")
@RequestMapping
@Api(tags = "运营端 - 区域服务相关接口")
public class ServeController {

    @ApiOperation("区域服务分页查询")
    public void page() {
        
    }
}
```

@RestController("operationServeController")  指定名称operationServeController防止同类名的bean名称相同。



##### **接口路径**

定义为RESTful风格的路径。

**接口路径为：/foundations/operation/serve/page** 

可以在类上边使用@RequestMapping指定该类中接口的路径的基础路径。

在方法中指定具体的路径。

完善controller如下：

如下：

```Java
package com.jzo2o.foundations.controller.operation;


import com.jzo2o.common.model.PageResult;
import com.jzo2o.foundations.model.dto.request.ServePageQueryReqDTO;
import com.jzo2o.foundations.model.dto.response.ServeResDTO;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * <p>
 * 前端控制器
 * </p>
 *
 * @author itcast
 * @since 2023-07-03
 */
@RestController("operationServeController")
@RequestMapping("/operation/serve")
@Api(tags = "运营端 - 区域服务相关接口")
public class ServeController {

    @GetMapping("/page")
    @ApiOperation("区域服务分页查询")
    public void page() {
        
    }
}
```

@RequestMapping("/operation/serve") 指定本controller的根路径。

@GetMapping("/page")  Get请求。



##### **请求参数类型**

常用的有：

**json格式**：

application/json，传递json格式字符串，当传递的参数是属于一个对象的属性时可用此格式，比如：新增、修改时通常传递的数据是某个对象的信息就可以使用此格式。

**表单格式**：

application/x-www-form-urlencoded，传递key/value串，就是在url后通过？和&进行拼接的参数，比如：

/foundations/operation/serve/page?pageNo=1&pageSize=10

当传递的参数比较杂且不属于某个特定的对象时使用此格式，**本接口使用application/x-www-form-urlencoded格式。**



##### **请求参数内容**

请求参数的内容根据需求文档和界面原型去识别。

对于查询类的接口常用的参数有：排序方式、排序字段、当前页码和每页显示记录数。

查询某个区域下的服务需要传递一个区域id。

对于分页查询的参数可以定义一个通用的类，如下是部分核心 代码：

```Java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package com.jzo2o.common.model.dto;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

@ApiModel("分页查询数据")
public class PageQueryDTO {
    @ApiModelProperty("页码数")
    private Long pageNo = 1L;
    @ApiModelProperty("每页条数")
    private Long pageSize = 10L;
    @ApiModelProperty("排序字段1")
    private String orderBy1;
    @ApiModelProperty("排序字段1是否升序")
    private Boolean isAsc1 = false;
    @ApiModelProperty("排序字段2，排序顺序排在排序字段1后边，如果排序字段1未设置，该字段也可以排序")
    private String orderBy2;
    @ApiModelProperty("排序字段2是否升序")
    private Boolean isAsc2 = false;
    ....
}
```

定义ServePageQueryReqDTO 类，继承PageQueryDTO 类，代码如下：

```Java
package com.jzo2o.foundations.model.dto.request;

import com.jzo2o.common.model.dto.PageQueryDTO;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

/**
 * 服务分页查询类
 *
 * @author itcast
 * @create 2023/7/4 12:43
 **/
@Data
@ApiModel("服务分页查询类")
public class ServePageQueryReqDTO extends PageQueryDTO {
    @ApiModelProperty(value = "区域id", required = true)
    private Long regionId;
}
```

我们发现ServePageQueryReqDTO 类中定义了分页查询区域下服务的参数。

完善controller方法：

```Java
package com.jzo2o.foundations.controller.operation;


import com.jzo2o.common.model.PageResult;
import com.jzo2o.foundations.model.dto.request.ServePageQueryReqDTO;
import com.jzo2o.foundations.model.dto.response.ServeResDTO;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * <p>
 * 前端控制器
 * </p>
 *
 * @author itcast
 * @since 2023-07-03
 */
@RestController("operationServeController")
@RequestMapping("/operation/serve")
@Api(tags = "运营端 - 区域服务相关接口")
public class ServeController {

    @GetMapping("/page")
    @ApiOperation("区域服务分页查询")
    public void pagee(ServePageQueryReqDTO servePageQueryReqDTO) {
        
    }
}
```

注意：当请求参数格式为json需要在方法参数前加@RequestBody注解，这里请求参数类型为from表单格式不用添加此注解。



##### **响应结果类型**

常见的类型有：text/html、text/plain、application/json等。

**本项目统一使用application/json**

响应结果状态码

HTTP状态码是服务器返回给客户端的数字代码（三位数字），共分为五类：

1xx: 表示服务器接收到了客户端请求并正在处理

2xx: 表示成功状态码

3xx：表示重向定状态码

4xx：表示客户端错误状态码

5xx: 表示服务端错误状态码

当服务端处理成功返回200，其它表示失败。



##### **响应结果内容**

分页查询通用的响应内容有：数据列表、总页数、总记录数

数据列表中需要分析显示的具体字段，根据界面原型进行分析：

![图片展示的是区域服务查询的响应结果内容界面。表格中包含服务名称、服务类型、参考价格、区域价格、是否热门、状态、操作等列。如“日常保洁”服务类型为保洁，参考价格44.00，区域价格44，状态为已下架，操作有设置热门、删除、上架等。该图片与上下文紧密相关，是对分页查询通用响应内容中数据列表中需分析显示的具体字段进行界面原型分析的直观呈现。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WRrVbLzcaoGsrAxsh7Wcg51Wnoc.png)

分页查询通用的响应内容如下：

pages 、total、 list 等是固定的。

list 中存储的是具体查询的数据（List结构)。

对于分页查询显示的通用信息用一个通用的类PageResult表示，如下是部分核心 代码：

```Java
package com.jzo2o.common.model;
....
@ApiModel(
    value = "分页数据消息体",
    description = "分页数据统一对象"
)
public class PageResult<T> {
    @ApiModelProperty(
        value = "总页数",
        required = true
    )
    private Long pages = 0L;
    @ApiModelProperty(
        value = "总条数",
        required = true
    )
    private Long total;
    @ApiModelProperty(
        value = "数据列表",
        required = true
    )
    private List<T> list;
    
    .....
```

具体的List数据需要定义ServeResDTO类，我们可以参考PO去定义ServeResDTO。

注意：有些信息在界面原型上是可以看的见的，比如：服务项名称，还有些在界面原型上看不见但是也需要作为结果响应给前端 ，比如：服务项的ID，当进行删除时就需要将此ID传入后端，但是界面 上却不显示。

```Java
package com.jzo2o.foundations.model.dto.response;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

import java.math.BigDecimal;
import java.time.LocalDateTime;

/**
 * 服务响应值
 *
 * @author itcast
 * @create 2023/7/4 11:53
 **/
@Data
@ApiModel("服务响应值")
public class ServeResDTO {
    /**
     * 主键
     */
    @ApiModelProperty("主键")
    private Long id;

    /**
     * 售卖状态，0：草稿，1下架，2上架
     */
    @ApiModelProperty("售卖状态，0：草稿，1下架，2上架")
    private Integer saleStatus;

    /**
     * 服务id
     */
    @ApiModelProperty("服务项id")
    private Long serveItemId;

    /**
     * 服务名称
     */
    @ApiModelProperty("服务项名称")
    private String serveItemName;

    /**
     * 服务类型id
     */
    @ApiModelProperty("服务类型id")
    private Long serveTypeId;

    /**
     * 服务名称
     */
    @ApiModelProperty("服务类型名称")
    private String serveTypeName;

    /**
     * 区域id
     */
    @ApiModelProperty("区域id")
    private Long regionId;

    /**
     * 参考价格
     */
    @ApiModelProperty("参考价格")
    private BigDecimal referencePrice;

    /**
     * 价格
     */
    @ApiModelProperty("价格")
    private BigDecimal price;

    /**
     * 是否为热门，0非热门，1热门
     */
    @ApiModelProperty("是否为热门，0非热门，1热门")
    private Integer isHot;

    /**
     * 创建时间
     */
    @ApiModelProperty("创建时间")
    private LocalDateTime createTime;

    /**
     * 更新时间
     */
    @ApiModelProperty("更新时间")
    private LocalDateTime updateTime;
}
```



##### 接口设计完成

将Controller类中对应接口的方法编写完成就代表接口设计完成。

代码如下：

```Java
package com.jzo2o.foundations.controller.operation;


import com.jzo2o.common.model.PageResult;
import com.jzo2o.foundations.model.dto.request.ServePageQueryReqDTO;
import com.jzo2o.foundations.model.dto.response.ServeResDTO;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * <p>
 * 前端控制器
 * </p>
 *
 * @author itcast
 * @since 2023-07-03
 */
@RestController("operationServeController")
@RequestMapping("/operation/serve")
@Api(tags = "运营端 - 区域服务相关接口")
public class ServeController {

    @GetMapping("/page")
    @ApiOperation("区域服务分页查询")
    public PageResult<ServeResDTO> page(ServePageQueryReqDTO servePageQueryReqDTO) {
        return null;
    }
}
```



##### 生成接口文档

我们通过swagger根据controller类最终生成接口文档，接口设计信息如下：

![图片展示的是“区域服务部分查询”接口文档页面。左侧为导航栏，可选择不同接口文档。右侧上方显示接口名称及请求方式，下方有请求参数列表，如regionId、isAsc1、isAsc2等，部分参数有默认值提示。底部有响应状态码说明。该图片与文档中“生成接口文档”部分内容相关，用于说明Java程序员可通过此文档测试接口，展示接口设计信息及请求参数等关键内容。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EtPFb0PX8okPbfxGScrcAos0n6e.png)





Java程序员可以使用接口文档测试接口，下边说明测试方法。

首先在controller方法打断点：

![图片展示的是controller方法代码，方法名为page，参数为ServePageQueryReqDTO类型的servePageQueryReqDTO。方法中仅有一行代码，即return null。该图片与文档中“首先在controller方法打断点”内容对应，是文档中提到的在controller方法打断点时的代码界面，用于后续跟踪断点请求参数是否正确及跟踪响应内容的操作。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WCKtbYU2Jol2Rcxwb7hcylkOnhh.png)

进入接口调试界面

![这张图片是家政服务运营基础服务的接口文档界面，呈现的是“区域服务分页查询”的调试界面，用于支撑接口调试相关的操作。界面左侧有“调试”选项被红色箭头标注，右侧展示了该接口的请求方式为GET，对应的接口路径为/foundations/operation/serve/page，文档当前处于该接口的调试页面，与文档中提及的“进入接口调试界面”的操作内容相呼应。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/G3kPbJY7CoqAl8xuCltcWR4Enqb.png)

输入请求参数：

![图片展示的是接口调试界面，用于输入请求参数。界面显示GET请求，目标URL为/foundations/operation/serve/page。参数部分，x-www-form-urlencoded选项被选中，有多个参数，如isAsc1、isAsc2、orderBy1、orderBy2、pageNo、pageSize、regionId等，其中regionId参数值为100。该图片与文档中“进入接口调试界面，输入请求参数”内容对应，是进行接口调试操作时的参数输入界面展示。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/SNRvbyqlAojz2nxzL97c587Fn8g.png)

点击“发送”，跟踪断点请求参数是否正确

![图片展示的是ServeController类中page方法的代码及接口调试界面。代码中page方法接收ServePageQueryReqDTO参数，返回PageRoute<ServeResDTO>类型结果，但此处返回null。调试界面显示请求参数，包括regionId、no、pageSize、isAsc1、isAsc2等，其中regionId为100，no为1，pageSize为10。该图片与上文提到的在controller方法打断点、进入接口调试界面、输入请求参数等内容相关，直观呈现了调试时的代码及参数情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/KPDebh2bkoydrqxIhmdcxjCYnse.png)

跟踪响应内容

![图片展示的是接口调试界面中响应内容的JSON格式数据。数据包含“code”为200、“message”和“msg”均为“OK”、“success”为true、“result”和“data”均为null。该图片与上文提到的在controller方法打断点后进入接口调试界面，输入请求参数、点击“发送”跟踪断点请求参数是否正确，以及跟踪响应内容的操作相关，直观呈现了响应结果内容，帮助理解明明page(ServePageQueryReqDTO servePageQueryReqDTO)方法返回了null，却响应了特定内容的原因。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/LNyFboGtjolZN5xgESfc6gHBnih.png)

明明page(ServePageQueryReqDTO servePageQueryReqDTO)方法返回了null， 这里为什么会响应如下内容呢？

```Java
{
  "code": 200,
  "message": "OK",
  "msg": "OK",
  "success": true,
  "result": null,
  "data": null
}
```

在jzo2o-mvc工程中有一个过滤器负责对接口的响应结构进行封装

![图片展示了jzo2o-mvc工程的目录结构。在src/main/java/com/jzo2o/mvc目录下，有advice、config、constants、filter、handler、interceptor、model、serialize、utils、wrapper等多个子目录。其中，filter目录被蓝色框线突出显示。该图片与文档中“在jzo2o-mvc工程中有一个过滤器负责对接口的响应结构进行封装”的内容相关，直观呈现了工程中负责此功能的filter目录位置。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HphpbKy56o8sHPxnVsjcc5NgnNe.png)

封装的方法是在原有响应内容基础上加上msg、code、result、success等固定属性。



#### 3) 小结

如何设计一个RESTful接口？



### 1.1.2 Mapper开发

目标：掌握开发持久层接口的过程。



#### 1）生成基础代码

通常一个接口需求明确后从持久层开始开发。

对于一个新模块在设计完表会使用工具生成模型类、mapper接口、mapper映射文件等持久层文件，本项目使用MybatisPlus插件自动生成代码。

前边已介绍过MybatisPlus插件，在此不再赘述。



#### 2）mapper开发

持久层基础代码生成完毕下边开发mapper接口，对于通用的CRUD接口由MybatisPlus提供。

对于需要自定义mapper接口的需求则需要自定义mapper接口及mapper映射文件，根据需求本接口要返回的数据包括多张表的数据，而MybatisPlus提供的CRUD是针对单表的，下边定义mapper实现多表关联查询。

先定义mapper接口：

mapper接口讲究通用性，根据数据处理最底层的需求去定义接口，本需求是根据区域查询服务列表，参数为区域id，方法返回值为服务列表。

```Java
package com.jzo2o.foundations.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.jzo2o.api.foundations.dto.response.ServeAggregationResDTO;
import com.jzo2o.foundations.model.domain.Serve;
import com.jzo2o.foundations.model.dto.response.ServeAggregationSimpleResDTO;
import com.jzo2o.foundations.model.dto.response.ServeAggregationTypeSimpleResDTO;
import com.jzo2o.foundations.model.dto.response.ServeCategoryResDTO;
import com.jzo2o.foundations.model.dto.response.ServeResDTO;
import org.apache.ibatis.annotations.Param;

import java.util.List;

/**
 * <p>
 * Mapper 接口
 * </p>
 *
 * @author itcast
 * @since 2023-07-03
 */
public interface ServeMapper extends BaseMapper<Serve> {
    /**
     * 根据区域查询服务列表
     * @param regionId
     * @return
     */
    List<ServeResDTO> queryServeListByRegionId(@Param("regionId") Long regionId);
}
```

再定义mapper映射：

```Java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jzo2o.foundations.mapper.ServeMapper">

    <select id="queryServeListByRegionId" resultType="com.jzo2o.foundations.model.dto.response.ServeResDTO">
        SELECT
            serve.id,
            serve.sale_status,
            serve.serve_item_id,
            item.name AS serve_item_name,
            item.serve_type_id,
            serve.region_id,
            item.reference_price,
            serve.price,
            serve.is_hot,
            serve.create_time,
            serve.update_time,
            type.id serve_type_id,
            type.name serve_type_name
        FROM
            serve
                inner JOIN serve_item AS item ON serve.serve_item_id = item.id
                inner JOIN serve_type AS type ON item.serve_type_id = type.id
        WHERE
            serve.region_id = #{regionId}
    </select>

</mapper>
```

#### 3）单元测试

对mapper接口进行测试

```Java
package com.jzo2o.foundations.mapper;

import com.jzo2o.foundations.model.dto.response.ServeResDTO;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.util.Assert;

import javax.annotation.Resource;
import java.util.List;

@SpringBootTest
@Slf4j
class ServeMapperTest {
    @Resource
    private ServeMapper serveMapper;

    @Test
    void testQueryServeListByRegionId() {
        List<ServeResDTO> serveResDTOS = serveMapper.queryServeListByRegionId(1686303222843662337L);
        Assert.notEmpty(serveResDTOS,"列表为空");
    }

}
```

断点调试：

![这是一张Java单元测试的代码运行截图，展示的是ServeMapper接口的测试用例代码。测试方法为test_queryServeListByRegionId，调用参数为区域ID（1686303222843662337L）的serveMapper对象的queryServeListByRegionId方法，返回类型为ServeResDTO集合。代码中断点调试停留在Assert.notEmpty语句处，右侧显示该方法返回的ServeResDTO集合已实例化（ArrayList），包含5条结果，每条结果标注了唯一ID、服务状态等信息，该图片对应文档中针对mapper接口的断点调试操作说明。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Uz5Gbjs1uoC2IaxFJ4McELgcnFd.png)



#### 4）小结

Mybatis-Plus有几种主键生成策略？



如何开发一个接口的持久层？



@Resource 和 @Autowired有什么区别? 



### 1.1.3 Service开发

#### **1) 定义service接口**

service接口是提供controller调用，service接口的参数由controller传入通常controller方法将它的形参直接传入service方法，对于通用的service方法则需要定义更通用的参数，service方法的返回值类型通常可以直接定义为controller方法的返回值类型。

定义如下：

```Java
package com.jzo2o.foundations.service;

import com.baomidou.mybatisplus.extension.service.IService;
import com.jzo2o.common.model.PageResult;
import com.jzo2o.foundations.model.domain.Serve;
import com.jzo2o.foundations.model.dto.request.ServePageQueryReqDTO;
import com.jzo2o.foundations.model.dto.response.ServeResDTO;

/**
 * <p>
 * 服务类
 * </p>
 *
 * @author itcast
 * @since 2023-07-03
 */
public interface IServeService extends IService<Serve> {

    /**
     * 分页查询服务列表
     * @param servePageQueryReqDTO 查询条件
     * @return 分页结果
     */
    PageResult<ServeResDTO> page(ServePageQueryReqDTO servePageQueryReqDTO);
}
```

#### 2）定义service实现类

这里我们使用的Mybatis-Plus提供的ServiceImpl类，如下代码，在此类中有baseMapper，baseMapper即是ServeMapper实例。

```Java
package com.jzo2o.foundations.service.impl;

import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.jzo2o.common.model.PageResult;
import com.jzo2o.foundations.mapper.ServeMapper;
import com.jzo2o.foundations.model.domain.Serve;
import com.jzo2o.foundations.model.dto.request.ServePageQueryReqDTO;
import com.jzo2o.foundations.model.dto.response.ServeResDTO;
import com.jzo2o.foundations.service.IServeService;
import com.jzo2o.mysql.utils.PageHelperUtils;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * <p>
 * 服务实现类
 * </p>
 *
 * @author itcast
 * @since 2023-07-03
 */
@Service
public class ServeServiceImpl extends ServiceImpl<ServeMapper, Serve> implements IServeService {

    /**
     * 分页查询
     *
     * @param servePageQueryReqDTO 查询条件
     * @return 分页结果
     */
    @Override
    public PageResult<ServeResDTO> page(ServePageQueryReqDTO servePageQueryReqDTO) {
        //通过baseMapper调用queryServeListByRegionId方法
        
    }

}
```

#### 3）实现分页查询

如何实现分页查询？

我们可以参考现有service类实现方法。

分别阅读ServeTypeServiceImpl中的分页查询方法和ServeItemServiceImpl中的分页查询方法。

我们发现两个类中的分页查询方法不一样

ServeTypeServiceImpl的方法如下：

```Java
public PageResult<ServeTypeResDTO> page(ServeTypePageQueryReqDTO serveTypePageQueryReqDTO) {
    Page<ServeType> page = PageUtils.parsePageQuery(serveTypePageQueryReqDTO, ServeType.class);
    Page<ServeType> serveTypePage = baseMapper.selectPage(page, new QueryWrapper<>());
    return PageUtils.toPage(serveTypePage, ServeTypeResDTO.class);
}
```

ServeTypeServiceImpl中的分页方法使用的是mybatis-plus自带的分页方法，通过selectPage方法实现分页。

ServeItemServiceImpl中的分页方法如下：

```Java
public PageResult<ServeItemResDTO> page(ServeItemPageQueryReqDTO serveItemPageQueryReqDTO) {
    return PageHelperUtils.selectPage(serveItemPageQueryReqDTO,
            () -> baseMapper.queryList(serveItemPageQueryReqDTO.getServeTypeId(), serveItemPageQueryReqDTO.getName(), serveItemPageQueryReqDTO.getActiveStatus()));
}
```

ServeItemServiceImpl中的分页方法使用的是com.github.pagehelper分页组件。

我们应该使用哪种方法呢？

**第一种：mybatis-plus自带的分页方法，通过调用selectPage方法实现分页，适用于通过QueryWrapper拼装SQL。**

**第二种：pagehelper分页组件适用于自定义sql的分页查询。**

这里我们自定义了查询当前区域服务的sql，所以适用第二种分页方法。

我们先照葫芦画瓢实现service：

```Java

/**
 * 分页查询服务列表
 * @param servePageQueryReqDTO 查询条件
 * @return 分页列表
 */
@Override
public PageResult<ServeResDTO> page(ServePageQueryReqDTO servePageQueryReqDTO) {
    //调用mapper查询数据，这里由于继承了ServiceImpl<ServeMapper, Serve>，使用baseMapper相当于使用ServeMapper
    PageResult<ServeResDTO> serveResDTOPageResult = PageHelperUtils.selectPage(servePageQueryReqDTO, () -> baseMapper.queryServeListByRegionId(servePageQueryReqDTO.getRegionId()));
    return serveResDTOPageResult;
}
```



#### 4）单元测试

```Java
package com.jzo2o.foundations.service;

import com.jzo2o.common.model.PageResult;
import com.jzo2o.foundations.model.dto.request.ServePageQueryReqDTO;
import com.jzo2o.foundations.model.dto.response.*;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.util.Assert;

import javax.annotation.Resource;
import java.math.BigDecimal;
import java.util.List;

@SpringBootTest
@Slf4j
class IServeServiceTest {
    @Resource
    private IServeService serveService;

    //分页测试
    @Test
    public void test_page(){
        ServePageQueryReqDTO servePageQueryReqDTO = new ServePageQueryReqDTO();
        servePageQueryReqDTO.setRegionId(1686303222843662337L);
        servePageQueryReqDTO.setPageNo(1L);
        servePageQueryReqDTO.setPageSize(3L);
        PageResult<ServeResDTO> page = serveService.page(servePageQueryReqDTO);
        log.info("page : {}", page);
        Assert.notEmpty(page.getList(),"列表为空");
    }
}
```

#### 5）分页原理

使用com.github.pagehelper分页插件实现分页功能，下边介绍它的执行原理。

PageHelperUtils是项目封装的一个工具类，进入selectPage方法，调用PageHelper.startPage方法设置分页参数，通过一层一层进入源码，最终将分页参数设置到ThreadLocal<Page>*LOCAL_PAGE*=newThreadLocal(); 中。

![图片展示了PageMethod.class文件中PageHelperUtils类的源码代码。在代码第72行，有红色箭头指向的代码行“setLocalPage(page);”，其下方有红色文字标注“设置到ThreadLocal中”。该图片与上文介绍的分页原理相关，上文提到PageHelperUtils进入selectPage方法调用PageHelper.startPage设置分页参数，最终将分页参数设置到ThreadLocal中，此图直观呈现了这一设置过程。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/FHxnbQ7nToyMDKxL7AVcFxoJnJc.png)

通过PageInterceptor拦截器拦截 MyBatis的Executor 的 query() 方法得到原始的sql语句，首先得到count总数，然后从newThreadLocal中取出分页参数，在原始sql语句中添加分页参数查询分页数据。

部分源码截图如下：

![这张图片是PageInterceptor类的反编译源码截图，展示了其intercept方法的部分实现代码。截图里的代码中，明确标注出了“原始SQL语句”对应的BoundSql对象，代码通过Invocation相关入参获取了MappedStatement、参数、分页参数、结果处理器、执行器、缓存等对象，该源码是对前文分页原理的直观代码佐证，体现了分页功能借助PageInterceptor处理SQL、加载分页参数的代码细节。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DYI2bvwaeoFWJzxAGrMcdDKsnmc.png)

![这张图片是分页功能实现相关的源码截图，展示了PageInterceptor类的部分代码。图中第70行通过this.count方法获取总记录数，对应标注的“总记录数”说明，第69行判断当前dialect是否需要先执行count查询，后续代码逻辑涉及根据结果判断是否执行后续的分页处理步骤，和前文提到的通过PageInterceptor拦截器实现分页查询的技术原理相呼应，直观呈现了分页实现过程中获取总记录数的核心代码片段。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/JjiWbxg0so8P3Sxt5X7c0t3onzc.png)

![这张图片是MyBatis分页查询相关的源码截图，展示了`ExecutorUtil.class`文件中`pageQuery`静态方法的部分代码内容。截图中可以清晰看到代码逻辑：当不满足`if (dialect.beforePage(...))`条件时，会进入else分支，其中有标注为“分页查询SQL”的关键代码段，该代码通过`dialect.getPageSql()`方法获取分页查询SQL，将分页参数融入原始SQL语句中，结合上下文可知，这段代码正是前文提到的基于MyBatis实现分页查询的核心源码片段，对应了分页原理中获取分页数据的代码实现部分。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/FYWrblEZ0oUpDDxOa6EclGzjnf1.png)



#### 5）小结

项目的分页查询是怎么实现的？



### 1.1.4 Controller开发

#### 1）完善controller方法

在controller方法中调用service方法进行业务处理。

得到service方法的返回值封装与客户端要的数据格式进行返回。

```Java
package com.jzo2o.foundations.controller.operation;


import com.jzo2o.common.model.PageResult;
import com.jzo2o.foundations.model.dto.request.ServePageQueryReqDTO;
import com.jzo2o.foundations.model.dto.response.ServeResDTO;
import com.jzo2o.foundations.service.IServeService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

/**
 * <p>
 * 前端控制器
 * </p>
 *
 * @author itcast
 * @since 2023-07-03
 */
@RestController("operationServeController")
@RequestMapping("/operation/serve")
@Api(tags = "运营端 - 区域服务相关接口")
public class ServeController {
    @Resource
    private IServeService serveService;

    @GetMapping("/page")
    @ApiOperation("区域服务分页查询")
    public PageResult<ServeResDTO> page(ServePageQueryReqDTO servePageQueryReqDTO) {
        PageResult<ServeResDTO> page = serveService.page(servePageQueryReqDTO);
        return page;
    }
}
```

#### 2）接口测试

重启服务，通过接口文档工具进行测试。

遇到问题在controller、service中打断点进行调试。

测试时注意请求参数填写是否正确，下边是测试结果

![图片展示的是区域服务分页查询的测试界面。左侧有“主页”“文档”“调试”“Open”等选项卡，当前选中“调试”。中间部分显示GET请求，URL为“/foundations/operation/serve/page”，下方有“请求参数”“AfterScript”选项卡，当前选中“请求参数”。参数列表中，isAsc1、isAsc2、orderBy1、orderBy2、pageNo、pageSize、regionId等参数均有勾选，其中pageNo值为1，pageSize值为10，regionId值为1686303222843682337。右侧有“发送”按钮。该图与文档中测试区域服务分页查询内容相关，用于展示测试时的请求参数设置情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HiXPbo2WmoPspRxedHacBF42n5g.png)

返回结果：

![图片展示的是Controller开发中分页查询服务区域数据的测试结果。上方显示响应内容，包含code、msg、data等信息，如code为200、msg为“OK”、data中包含page、total、list等字段。下方有Raw、Headers、Curl等选项卡，Raw中呈现JSON格式数据，显示了两条服务区域记录，如id、saleStatus、servetId、servetName等字段信息。该图片与上文测试分页查询结果的内容对应，直观呈现了测试结果。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/JNa6bP4lPoxvVFxJVqBcpJtXnDg.png)

通过修改分页条件测试分页查询结果：

修改为一页显示2条记录，下边查询第一页：

![这张图是对应查询区域服务功能测试的分页参数设置界面，界面中有两个被勾选的参数项，分别为pageNo和pageSize，其中pageNo的值设置为1，代表要查询第一页数据，pageSize的值设置为2，代表每页显示2条记录，界面右侧还设有“删除”按钮，用于删除对应的参数项，该设置是文档中提到的修改分页条件后，用于测试分页查询结果的参数配置。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/SbEUbBE43oYdB2xNPppcpJdynZb.png)

![图片展示的是区域服务查询的测试结果。响应内容中，code为200，msg为“OK”，data部分包含pages、total、list等信息。其中，pages为2，total为4，list中包含两条记录，每条记录有id、saleStatus、serverType、serverTypeName、serverTypePname、regionId、referencePrice、isHot、createTime、updateTime等字段，如第一条记录的id为1699354106233835521，saleStatus为2等。该图片与上文提到的通过修改分页条件测试分页查询结果相呼应，展示了查询结果的具体内容。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/L0gPb767RoOmR5xWiFPcfnFgnce.png)

再查询第二页：

![这张图片展示了分页查询测试中的参数设置界面，共包含两组待配置项，分别是设置为2的“pageNo”（页码参数）和设置为2的“pageSize”（每页记录数参数），每组参数前都带有勾选框，右侧配有“删除”操作选项。该图片对应文档中分页查询测试的操作场景，显示的参数设置符合前文描述的“一页显示2条记录，查询第一页”的测试要求，用于验证分页查询功能的参数配置逻辑。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/I9X1bpRBKoN2jLxGWCScmUxbnod.png)

![图片展示的是查询区域服务时的返回结果界面。上方有“响应内容”“Raw”“Headers”“Curl”等选项卡，当前选中“响应内容”。下方代码区域显示了JSON格式的响应数据，包含code、msg、data等字段，其中data部分呈现了多条记录信息，如id、saleStatus、servletpId、servletpName等字段的具体值，还显示了创建和更新时间等。该图片与上文提到的查询区域服务及测试结果相关，直观呈现了查询结果内容。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/JrtSb22WfoTZHOxO8yuchrNanie.png)



#### 3）单元测试

有些公司要求需要对三层bean进行单元测试，针对controller方法如何单元测试？

例用AI生成单元测试方法中我们发现有一个org.springframework.test.web.servlet.MockMvc的类，正是使用MockMvc实现对controller方法的单元测试。

上网查阅使用MockMvc对Spring Boot项目controller方法的测试方法，写出例子代码如下：

```Java
package com.jzo2o.foundations.web;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;


/**
 * @author Mr.M
 * @version 1.0
 * @description ServeController单元测试类
 * @date 2024/9/14 11:07
 */

@SpringBootTest
@AutoConfigureMockMvc
public class ServeControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void doTest() throws Exception {
        mockMvc.perform(get("/operation/serve/page")
                .param("regionId", "1693814923234189313")
        ).andExpect(status().isOk()).andDo(result -> {
            //解析响应结果
            String contentAsString = result.getResponse().getContentAsString();
            System.out.println(contentAsString);
        });
    }
}
```

注意：本项目不要求对Controller方法进行单元测试，请自行学习MockMvc 。



#### 4）小结

如何开发conroller方法？



### 1.1.5 前后端联调

前后端联调是一种类似生产环境的测试，通过前端请求后端接口测试业务功能。

#### 1）启动前后端

启动foundations服务。

![图片展示了Spring Boot运行状态界面。左侧有“Spring Boot”和“Running”图标，右侧显示“FoundationsApplication”及数字“11509/”，表明当前运行的Spring Boot应用为“FoundationsApplication”，端口号为11509。该图片与文档中“启动前端工程，在项目开发环境配置中我们已经将前端环境配置完成”及“前后端联调”内容相关，直观呈现了前端工程启动后Spring Boot的运行状态，为前后端联调提供了技术支撑。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EDKTbkcoooj50Uxk03LcUAX8nEh.png)

启动前端工程，在项目开发环境配置中我们已经将前端环境配置完成。

![图片展示的是Windows PowerShell界面，显示了vite v2.9.15开发服务器运行信息。界面上方有“Windows PowerShell”标题，下方内容以绿色字体呈现，显示开发服务器运行在多个网络地址，包括http://172.16.43.194:3000/、http://192.168.27.1:3000/、http://192.168.101.1:3000/等，以及http://localhost:3000/的本地地址，最后以“ready in 1184ms.”结束。该图片与文档中前后端联调内容相关，用于说明前后端测试时可从浏览器网络记录跟踪请求参数和响应结果。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Yg44bjp9BoYpLNx2Kgkc98finQg.png)

#### 2）前后端联调

根据业务操作流程进行测试。

进入运营管理端，点击“区域管理”菜单，点击“设置服务”

![图片展示的是运营管理端区域管理页面。页面左侧有“服务管理”“服务类型管理”“服务项管理”等菜单栏，其中“区域管理”被红色箭头突出显示。右侧是区域服务列表，包含区域编号、区域名称、区域负责人、手机号码、状态及操作等信息，如010区域编号对应北京市，状态为启用，操作有删除、编辑、调度配置、设置服务、禁用、缓存刷新等。该图与上下文介绍的进入运营管理端点击“区域管理”菜单，点击“设置服务”相呼应，直观呈现了操作后的页面情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EUcKbI6iKoVijLxozlPcVVIfnNe.png)

区域服务列表如下图：

![图片展示的是运营管理端区域服务列表界面。区域名称为北京市，列表中包含服务名称、服务类型、参考价格、区域价格、是否热门等信息，如安装窗帘、厨卫维修等服务。右侧有操作栏，可对服务进行取消热门、下架、设置热门、删除、上架等操作。该图与上下文紧密相关，直观呈现了区域服务的管理情况，为程序员在前后端测试时跟踪前后端交互数据、判断测试结果及测试问题提供了数据参考。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XkGcbse1AoT4XNxffQIcwWg2n8e.png)

程序员在前后端测试时需要跟踪前后端交互数据判断精确判断测试结果及测试问题。

通过浏览器网络记录进行跟踪：

请求参数：

![图片展示了浏览器开发者工具中网络请求的参数信息。画面中箭头指向“参数”区域，显示了多个参数，如isAct、pageAsc1、pageAsc2、orderBy1、orderBy2、pageNum、pageSize、regionId等，其中pageAsc1、pageAsc2、orderBy1、orderBy2、pageNum、pageSize、regionId参数值分别为true、false、updateTime、sortNum、1、10、1686302228436623737。该图片与文档中“通过浏览器网络记录进行跟踪”部分内容相关，用于说明请求参数的查看方式。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/RgPfbymfQoMkMOxkcLicUty5nqf.png)

响应结果：

![这张图片是浏览器开发者工具的网络请求记录界面，展示了前后端联调时需要跟踪的相关交互数据。左侧标注了请求的相关标识，右侧用红框标出了对应请求的响应内容，响应中包含code、message、success等状态信息，以及pages的总数量和list列表等数据，还包含不同服务实例的相关信息，这正是程序员进行前后端测试时，通过跟踪浏览器网络记录来判断测试结果、排查测试问题的关键交互数据。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/UqdqbMQDmoTXfmxdEuicx83Cnid.png)





#### 3) 小结

前后端联调都调些什么，怎么进行前后端联调？



## 1.2 添加区域服务

### 1.2.1 接口设计

#### 1）接口分析

下边设计添加区域服务接口，重点设计：传入参数类型、参数内容、响应结果内容。

根据界面原型梳理操作流程：

首先进入某个区域的服务列表:

![图片展示的是区域管理中设置服务的界面。界面上方显示“服务管理 > 区域管理 > 设置服务”。下方有“区域名称”输入框，内有“北京市”字样，右侧有红色箭头指向的“添加服务”按钮。该图片与上下文紧密相关，上下文提到首先进入某个区域的服务列表，点击添加服务，此图即为点击“添加服务”按钮前的界面展示，直观呈现了操作流程中的一个环节。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/GNeGbLZcdoFeiPxLQOyclypPneb.png)

点击添加服务,如下图：

![图片展示的是添加 addCriterion添加服务界面。左侧有保洁清、日常维修、家居服务等分类，当前选中“日常维修”。右侧有“请输入服务关键字”搜索框，下方显示“空调维修”和“日常维修”服务，价格分别为198.00元和5.00元，均有蓝色勾选框。右侧“已选服务（2）”区域显示“日常维修”和“空调维修”已选服务。底部有“取消”和“添加”按钮，其中“添加”按钮被红色箭头指向。该图与上下文介绍的添加区域服务操作流程相关，是选择要添加服务的界面示例。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Hk6ObLMD6oeNvaxumMacSRbznTg.png)

选择要添加到区域中的服务，点击“添加”按钮。

最终向serve表添加数据，表结构如下：

```Java
create table serve
(
    id             bigint                             not null comment '服务id'
        primary key,
    serve_item_id  bigint                             not null comment '服务项id',
    region_id      bigint                             not null comment '区域id',
    city_code      varchar(255)                       not null comment '城市编码',
    sale_status    int      default 0                 not null comment '售卖状态，0：草稿，1下架，2上架',
    price          decimal(10, 2)                     not null comment '价格',
    is_hot         int      default 0                 not null comment '是否为热门，0非热门，1热门',
    hot_time_stamp bigint                             null comment '更新为热门的时间戳',
    create_time    datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    update_time    datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    create_by      bigint                             null comment '创建者',
    update_by      bigint                             null comment '更新者'
)
```

serve表存储了区域中开通的服务，根据字段信息思考数据来源：

| 字段名 | 含义 | 数据来源 |
|-|-|-|
| id | 服务id | 主键，自动生成 |
| serve_item_id   | 服务项id | 接口传入 |
| region_id       | 区域id | 接口传入 |
| city_code       | 城市编码 | 根据区域id查询region表得到 |
| sale_status     | 售卖状态 | 默认为草稿状态,数据库设置了默认值 |
| price           | 价格 | 默认为服务项的价格，用户可修改 |
| is_hot          | 是否为热门 | 默认为非热门,数据库设置了默认值 |
| hot_time_stamp  | 更新为热门的时间戳 | 默认空 |
| create_time     | 创建时间 | 数据库设置了默认值 |
| update_time     | 更新时间 | 数据库设置了默认值 |
| create_by       | 创建人 | 由framework的MyBatisAutoFillInterceptor自动处理 |
| update_by       | 更新人 | 由framework的MyBatisAutoFillInterceptor自动处理 |

**根据上边的分析，通过接口传入服务项id、区域id，支持多个服务传入，实现批量添加。**

传入参数内容为数组，所以使用json格式。

传入参数内容包括：服务项id、区域id。

响应结果内容可为空，前端根据状态码判断是否添加成功。

**接口设计信息：**

接口路径：**POST**/foundations/operation/serve/batch

**请求数据类型**  application/json

![图片展示了添加区域服务接口的请求参数信息。参数名称有serveUpsertReqDTOList、serveltemId、regionId，其中serveUpsertReqDTOList为必填，数据类型为array，schema为服务新增更新；serveltemId和regionId也为必填，数据类型均为integer(int64)。该图片与上文接口设计内容相关，直观呈现了接口请求参数的具体要求，帮助理解传入参数内容为数组，使用json格式，包含服务项id、区域id等信息。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Xc03br1SaoVoNexDv51ctNMunHb.png)

![这张图片呈现的是添加区域服务接口的响应参数信息，与文档中该接口设计的内容相对应。图片里的响应参数包含两类，一是名称为msg的参数，参数说明为响应消息，类型为string；二是名称为code的参数，参数说明为状态码，200代表成功，其他值代表失败，类型为integer(int32)，schema属性也对应该类型。这些内容明确了该接口响应的具体参数要求，方便开发人员理解接口返回的数据格式。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/IAGzbgZReoG6p0xJJS9cwEBMne1.png)



#### 2）接口定义

1. 首先定义接收传入参数的模型类ServeUpsertReqDTO

```Java
package com.jzo2o.foundations.model.dto.request;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

import java.math.BigDecimal;

/**
 * 服务新增更新
 *
 * @author itcast
 * @create 2023/7/3 14:43
 **/
@Data
@ApiModel("服务新增更新")
public class ServeUpsertReqDTO {

    /**
     * 服务id
     */
    @ApiModelProperty(value = "服务id", required = true)
    private Long serveItemId;

    /**
     * 区域id
     */
    @ApiModelProperty(value = "区域id", required = true)
    private Long regionId;


}
```

1. 编写controller方法，HTTP方法使用POST方式，代码如下：

```Java
@RestController("operationServeController")
@RequestMapping("/operation/serve")
@Api(tags = "运营端 - 区域服务相关接口")
public class ServeController {

    @PostMapping("/batch")
    @ApiOperation("区域服务批量新增")
    public void add(@RequestBody List<ServeUpsertReqDTO> serveUpsertReqDTOList) {

    }
```



1. 重启服务，查看swagger文档如下：

![图片展示的是家政服务 - 运营基础服务接口文档中“区域服务批量新增”的接口定义。HTTP方法为POST，请求数据类型为application/json，响应数据类型为“*” 。请求示例为JSON格式，包含serveItemId、regionId、price三个字段。请求参数中，serveUserReqDTOList为必填的body类型数组，包含serveitemid、regionid、price字段，serveitemid、regionid、price均为必填的参数。该图片与上文介绍的添加区域服务接口设计相关，直观呈现了接口的请求示例和参数信息。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Nk0xb8eKIoqNWTxPidhcXLx2nnf.png)

#### 3）小结

如何设计添加接口？

添加接口最终向数据库提交数据，根据界面原型结合表结构进行设计，先分析表中每个字段的数据来源，再分析接口传入参数内容，通常添加接口没有特殊要求可根据状态码判断是否添加成功。



### 1.2.2 接口开发

#### 1）mapper

添加接口只向serve表添加数据所以使用mybatisplus提供的单表CRUD方法即可无需自定义mapper接口。

#### 2）service

定义service接口：

```Java
public interface IServeService extends IService<Serve> {

/**
 * 批量新增
 *
 * @param serveUpsertReqDTOList 批量新增数据
 */
void batchAdd(List<ServeUpsertReqDTO> serveUpsertReqDTOList);
...
```

定义service实现方法：

```Java
@Override
@Transactional
public void batchAdd(List<ServeUpsertReqDTO> serveUpsertReqDTOList) {
    for (ServeUpsertReqDTO serveUpsertReqDTO : serveUpsertReqDTOList) {
        //1.校验服务项是否为启用状态，不是启用状态不能新增
        ServeItem serveItem = serveItemMapper.selectById(serveUpsertReqDTO.getServeItemId());
        //如果服务项信息不存在或未启用
        if (ObjectUtil.isNull(serveItem) || serveItem.getActiveStatus() != FoundationStatusEnum.ENABLE.getStatus()) {
            throw new ForbiddenOperationException("该服务未启用无法添加到区域下使用");
        }

        //2.校验是否重复新增
        Long count = lambdaQuery()
                .eq(Serve::getRegionId, serveUpsertReqDTO.getRegionId())
                .eq(Serve::getServeItemId, serveUpsertReqDTO.getServeItemId())
                .count();
        if (count > 0) {
            throw new ForbiddenOperationException(serveItem.getName() + "服务已存在");
        }

        //3.新增服务
        Serve serve = BeanUtil.toBean(serveUpsertReqDTO, Serve.class);
        //默认为服务项的价格
        serve.setPrice(serveItem.getReferencePrice());
        Region region = regionMapper.selectById(serveUpsertReqDTO.getRegionId());
        serve.setCityCode(region.getCityCode());
        baseMapper.insert(serve);
    }
}
```

说明：

对于增、删、改类的接口通常会先进行入参校验，校验失败抛出异常，由异常处理器统一对异常进行处理。

我们抛出的异常是自定义异常类型，自定义的异常类型都继承了CommonException类型，在异常处理器中对此类型的异常进行处理。

异常处理器的源码在jzo2o-framework中，如下图：

![图片展示了jzo2o-mvc项目的目录结构。在src/main/java/com/jzo2o/mvc/advice目录下，有一个名为CommonExceptionAdvice的文件。该图片与上文提到的异常处理器源码在jzo2o-framework中的内容相关，直观呈现了异常处理器所在的具体位置，帮助理解自定义异常类型在项目中的位置关系。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/ITGrbVlTpomWo1xrt1CcsBXdnPh.png)

通过@RestControllerAdvice注解加@ExceptionHandler注解实现，具体的原理是当controller抛出异常由DispatcherServlet统一拦截处理，再根据异常类型找到@ExceptionHandler标识方法去执行该方法进行异常处理。



#### 3) controller

在controller方法中调用service接口添加区域服务。

```Java
@PostMapping("/batch")
@ApiOperation("区域服务批量新增")
public void add(@RequestBody List<ServeUpsertReqDTO> serveUpsertReqDTOList) {
    serveService.batchAdd(serveUpsertReqDTOList);
}
```

#### 4）小结

如何开发一个添加接口？

如何开发一个接口的service方法？

接口的异常处理怎么实现的？





### 1.2.3 测试

我们通过前后端联调测试添加区域服务接口。

#### 1）启动前后端

启动foundations服务。

![图片展示了Spring Boot运行状态界面。左侧有“Spring Boot”和“Running”图标，右侧显示“FoundationsApplication”及数字“11509/”，表明该应用正在运行，端口号为11509。该图与文档中“启动前端工程，在项目开发环境配置中我们已经将前端环境配置完成”上下文相关，用于说明前端工程已成功启动，处于运行状态，为后续的前后端联调测试做准备。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/CIDrbEEMqoZQUkxdOIgcRhjenYe.png)

启动前端工程，在项目开发环境配置中我们已经将前端环境配置完成。

![这是一张显示Windows PowerShell窗口的截图，内容为vite开发服务器的运行信息。页面显示v2.9.15版本的开发服务器已启动，当前处于就绪状态，就绪耗时1184毫秒。窗口中列出了多个可访问地址，包含对应Network、Local的访问路径，涵盖局域网内不同IP地址的访问链接及本地localhost访问链接，所有链接的端口号均为3000，这些链接用于提供当前开发环境下的服务访问入口。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/PkOdbGfmboFKLaxvzsxcE8ibngc.png)

#### 2）前后端联调

因为代码是我们写的，我们在前后端联调时除了测试成功结果还要测试失败结果。

**下边我们先测试正常流程：**

进入运营管理端，点击“区域管理”菜单，点击“设置服务”

![图片展示的是运营管理端区域管理页面。页面左侧有“服务管理”下拉菜单，其中“区域管理”被红色箭头指向突出显示。右侧是区域服务列表，包含区域编号、区域名称、区域负责人、手机号码、状态及操作等信息，如010北京市老孙等。页面右上角有“新建”按钮。该图片与上文“进入运营管理端，点击‘区域管理’菜单，点击‘设置服务’”的操作步骤对应，直观呈现了操作后的页面情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/X2W7bXTgRomIz7xO1XncfrMmnQf.png)

区域服务列表如下图：

![这张图片为运营管理端的区域服务列表界面，对应文档中添加区域服务测试的正常流程环节。界面显示区域名称为“北京市”，列表中包含安装窗饰、厨卫维修、开荒保洁、日常保洁四类服务，每类服务对应展示了服务类型、参考价格、区域价格、是否热门的信息，每类服务的操作栏提供了取消热门、设置热门、删除、上下架的功能选项，界面右上角有“添加服务”按钮，该界面是添加区域服务流程中，用于查看已添加的区域服务列表的页面，符合文档中添加服务后需查询区域服务列表的测试环节内容。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/J57sb388TobbHWxij68cf7dCnfe.png)

点击添加服务



![图片展示的是运营管理端“区域管理”下的“设置服务”页面。页面上方显示导航路径“服务管理 > 区域管理 > 设置服务”。下方有“区域名称”输入框，已输入“北京市”，右侧有红色框突出显示的“添加服务”按钮。该图片与上下文关系紧密，是“添加区域服务”测试流程中，点击“添加服务”按钮前的页面展示，直观呈现了操作前的页面状态。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/AccDbimEvoXRyFxFyFYcWR3rnsg.png)

点击添加服务,如下图：

![图片展示的是运营管理端添加服务的界面。左侧有保洁清、日常维修、家居服务等分类，右侧有“请输入服务关键字”搜索框。下方显示已选服务“日常维修”和“空调维修”，右侧有“取消”和“添加”按钮。图片中“选择要添加的服务”文字及红色箭头指向“日常维修”选项，表明操作步骤是选择要添加的服务，此图与上文“点击添加服务”及“添加成功查询区域服务列表是否存在已添加的服务”等内容相关，是添加服务操作流程中的关键步骤展示。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XSR4b4upFoNfdax0dd0cxgZ4ndg.png)

添加成功查询区域服务列表是否存在已添加的服务。

如果列表没有显示已添加的服务则需要进行排查，可以进入数据库直接查询serve表查看是否添加成功，如果后台有错误日志需要根据错误日志进行排查。

**下边测试异常流程：**

我们测试服务重复添加的异常，由于前端控制无法选择已经添加的服务，我们可以开两个窗口，都停留在添加服务页面：

![这张图片是系统“添加服务”的操作界面，左侧显示保洁清、日常维修、家居服务三类服务选项，当前定位在“日常维修”类别；中间区域可搜索服务关键字，其中“空调维修 ￥198.00”已被勾选，“日常维修 ￥5.00”为未勾选状态；右侧“已选服务（1）”栏显示已选项目为“空调维修”；界面底部设有“取消”和“添加”按钮，该界面对应操作文档中提及的，为测试服务重复添加异常流程所使用的待填写的服务添加页面。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WzNdb0i68olKWcxtpmWcNAg5nJu.png)

其中一个窗口添加成功后，另一个窗口也进行提交，此时跟踪前端拿到的响应如下：

```Java
{"code":604,"msg":"空调维修服务已存在","data":null,"requestId":"1715263501921951744"}
```

此测试结果符合我们的预期。





## 1.3 修改价格

### 1.3.1 接口设计

#### 1) 接口分析

根据界面原型设计传入参数类型、参数内容、响应结果内容。

进入修改界面

![图片展示的是区域服务管理模块中修改价格的界面。上方显示区域名称为北京市。下方表格中列出服务名称为安装窗帘，服务类型为家居服务，参考价格和区域价格均为10.00。表格右侧有“取消热门”“删除”“下架”等操作选项，其中“下架”选项旁有红色箭头指向，标注“点击修改”。该图片与文档中“进入修改界面”内容对应，直观呈现了修改价格操作的界面情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/KIYFbXxKpohGthxP7E9ciSkrnWc.png)

输入价格进行修改

![图片展示的是区域服务管理模块中修改价格的界面。上方显示区域名称为北京市。下方表格中，服务名称为安装窗帘，服务类型为家居服务，参考价格为10.00，区域价格框内显示10.00，是否热门为是。红色箭头指向区域价格框，旁边有文字“输入价格进行修改”，表明此处用于输入价格进行修改操作。该图片与文档中“输入价格进行修改”内容对应，直观呈现了操作界面。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EVAQbw4jPoDhA2xrubpcAJ3gn5y.png)

传入参数内容：价格、区域服务ID（serve表主键）  ，区域服务ID简称为服务ID

根据id更新serve表的价格字段。

传入参数类型：form表单格式

响应结果内容可为空，前端根据状态码判断是否添加成功。

接口设计信息如下：

接口路径：**PUT**/foundations/operation/serve/{id}

**请求数据类型** application/x-www-form-urlencoded

![图片展示了修改价格接口的请求参数信息。表格包含参数名称、参数说明、请求类型、是否必须、数据类型、schema六列。关键参数有id，服务id，请求类型为path，是否必须为true，数据类型为integer(int64)；price，价格，请求类型为query，是否必须为true，数据类型为number(double)。该图片与文档中接口设计部分相关，是对接口请求参数的详细说明，帮助理解接口参数配置情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/TG8KbEIuboz8fqxGWJjcAa4YnPh.png)

![图片展示了“响应参数”相关的信息。表格包含参数名称、参数说明、类型、schema四列。其中，参数名称有msg、code；参数说明分别为响应消息、状态码，200 - 成功，其它 - 失败；类型均为string、integer(int32)；schema均为integer(int32)。该图片与上文接口设计中“响应参数”部分对应，是对响应参数的具体说明。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/F0WDbSM8XotoI0xMjF4cL7ICn79.png)



#### 2) 接口定义

编写controller方法，HTTP方法使用PUT方式，代码如下：

```Java
@PutMapping("/{id}")
@ApiOperation("区域服务价格修改")
@ApiImplicitParams({
        @ApiImplicitParam(name = "id", value = "服务id", required = true, dataTypeClass = Long.class),
        @ApiImplicitParam(name = "price", value = "价格", required = true, dataTypeClass = BigDecimal.class)
})
public void update( @PathVariable("id") Long id,
                    @RequestParam("price") BigDecimal price) {
    
}
```



查看接口文档：

重启服务，查看swagger文档如下：

![图片展示的是家政服务 - 运营基础服务接口文档中“区域服务价格修改”的接口定义。HTTP方法为PUT，接口路径为/foundations/operation/serve/{id}，请求数据类型为application/x-www-form-urlencoded，响应数据类型为“*/*”。请求参数包括id（服务id，path类型，必填，数据类型为integer(int64)）、price（价格，query类型，必填，数据类型为number(double)）。该图片与上文接口设计中接口定义部分对应，直观呈现了接口相关参数信息。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/F4g9bheKXofUx9xw00Vc8ocZn9e.png)

#### 3）mapper

修改接口只向serve表更新数据所以使用mybatisplus提供的单表CRUD方法即可无需自定义mapper接口。

#### 4）service

定义service接口：

```Java
public interface IServeService extends IService<Serve> {

/**
 * 服务价格修改
 *
 * @param id    服务id
 * @param price 价格
 * @return 服务
 */
Serve update(Long id, BigDecimal price);
...
```

定义service实现方法：

```Java
@Override
@Transactional
public Serve update(Long id, BigDecimal price) {
    //1.更新服务价格
    boolean update = lambdaUpdate()
            .eq(Serve::getId, id)
            .set(Serve::getPrice, price)
            .update();
    if(!update){
        throw new CommonException("修改服务价格失败");
    }
    return baseMapper.selectById(id);
}
```



#### 5) controller

在controller方法中调用service接口价格修改。

```Java
@PutMapping("/{id}")
@ApiOperation("区域服务价格修改")
@ApiImplicitParams({
        @ApiImplicitParam(name = "id", value = "服务id", required = true, dataTypeClass = Long.class),
        @ApiImplicitParam(name = "price", value = "价格", required = true, dataTypeClass = BigDecimal.class)
})
public void update(@PathVariable("id") Long id,
                   @RequestParam("price") BigDecimal price) {
    serveService.update(id, price);
}
```

#### 6）测试

略



#### 7) 小结

如何设计修改接口？

修改接口最终向数据库提交数据，根据界面原型结合表结构进行设计，分析更新的数据有哪些，再分析更新数据的依据是什么，本接口是根据serve表的主键修改价格字段。

如果传入的要修改的参数比较多此时可用json格式，如果比较少可用form表单格式即可。通常添加、修改、删除接口没有特殊要求可根据状态码判断是否添加成功，有特殊要求则需要返回json数据。





## 1.4 服务上架

### 1.4.1 接口设计

#### 1）接口分析

根据业务流程、界面原型进行设计。

在区域服务列表中点击“上架”，此服务在该区域将生效，用户即可对该服务进行下单。

![这张图片呈现的是区域服务列表中的一条服务项内容，服务名称为开荒保洁，保洁清对应其相关属性，价格与实付价格均为49.00，当前该服务的上架状态标注为“否”。图片右侧设有“设置热门”“删除”“上架”三个操作按钮，结合上下文可知，点击此处的“上架”按钮后，该服务会在对应区域生效，用户便可对该服务进行下单操作，同时服务在serve表的售卖状态也会被更改为代码为2的“上架”状态。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/D6Mfbnp0GonLtrxKqtucqTZjnad.png)

业务流程如下：

<readonly-block type="diagram"></readonly-block>

在serve表有一个售卖状态sale_status，服务上架后将状态更改为“上架”，代码为2。

下边分析接口的传入参数类型、参数内容、响应结果内容。

传入参数内容：服务Id(serve表的主键)，

传入参数类型：form表单格式

响应结果内容: 内容可为空，前端根据状态码判断是否添加成功。

**注意：区分 服务id和服务项id，服务id即serve表的主键，服务项id即serve_item表的主键。**



接口信息如下：

接口路径：**PUT**/foundations/operation/serve/onSale/{id}

**请求数据类型**  application/x-www-form-urlencoded

![图片展示了服务上架接口的请求参数信息。表格包含参数名称、参数说明、请求类型、是否必须、数据类型、schema六列。其中，id为服务id，请求类型为path，是必须参数，数据类型为integer(int64)。该图片与上文接口分析部分相关，是对接口信息中请求数据类型为application/x-www-form-urlencoded的接口参数说明，直观呈现了参数的具体内容。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XoIybaLpao3dmZx1DBhcA2MUnbf.png)

![图片展示的是服务上架接口的响应参数。表格包含参数名称、参数说明、类型、schema四列。其中，msg为响应消息，类型为string；code为状态码，200表示成功，其它表示失败，类型为integer(int32)。该图片与上文接口定义部分相关，是对接口响应参数的说明，为后续接口开发提供参数信息参考。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EmIibJCXYovVxRxCSQycOeRInEe.png)



#### 2) 接口定义

编写controller方法，HTTP方法使用PUT方式，代码如下：

```Java
@PutMapping("/onSale/{id}")
@ApiOperation("区域服务上架")
@ApiImplicitParams({
        @ApiImplicitParam(name = "id", value = "服务id", required = true, dataTypeClass = Long.class),
})
public void onSale(@PathVariable("id") Long id) {

}
```

查看接口文档：

重启服务，查看swagger文档如下：

![这张图片是服务上架接口的Swagger文档，位于接口开发相关内容之后，呈现了PUT方式的服务上架接口详情。该接口的路径为/foundations/operation/serve/onSale/{id}，请求数据类型为application/x-www-form-urlencoded。请求参数为id，属于路径参数，是必填的整数类型。响应状态包含200（成功）、201（已创建）、401（未授权）、403（禁止访问）、404（未找到）等状态码，响应参数有表示响应结果信息的msg，以及表示状态码的code，code为200时代表成功，其他值则为失败。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/OSG0b1PJboYKnVxczOocNIeSn1p.png)



### 1.4.2 接口开发

#### 1）mapper

服务上架最终修改serve表的状态，所以使用mybatisplus提供的单表CRUD方法即可无需自定义mapper接口。

#### 2）service

定义service接口： 

```Java
public interface IServeService extends IService<Serve> {


/**
 * 上架
 *
 * @param id         服务id
 */
Serve onSale(Long id);
...
```

定义service实现方法：

```Java
@Override
@Transactional
public Serve onSale(Long id){
    Serve serve = baseMapper.selectById(id);
    if(ObjectUtil.isNull(serve)){
        throw new ForbiddenOperationException("区域服务不存在");
    }
    //上架状态
    Integer saleStatus = serve.getSaleStatus();
    //草稿或下架状态方可上架
    if (!(saleStatus==FoundationStatusEnum.INIT.getStatus() || saleStatus==FoundationStatusEnum.DISABLE.getStatus())) {
        throw new ForbiddenOperationException("草稿或下架状态方可上架");
    }
    //服务项id
    Long serveItemId = serve.getServeItemId();
    ServeItem serveItem = serveItemMapper.selectById(serveItemId);
    if(ObjectUtil.isNull(serveItem)){
        throw new ForbiddenOperationException("所属服务项不存在");
    }
    //服务项的启用状态
    Integer activeStatus = serveItem.getActiveStatus();
    //服务项为启用状态方可上架
    if (!(FoundationStatusEnum.ENABLE.getStatus()==activeStatus)) {
        throw new ForbiddenOperationException("服务项为启用状态方可上架");
    }

    //更新上架状态
    boolean update = lambdaUpdate()
            .eq(Serve::getId, id)
            .set(Serve::getSaleStatus, FoundationStatusEnum.ENABLE.getStatus())
            .update();
    if(!update){
        throw new CommonException("启动服务失败");
    }
    return baseMapper.selectById(id);

}
```



#### 3) controller

在controller方法中调用service接口上架区域服务。

```Java
@PutMapping("/onSale/{id}")
@ApiOperation("区域服务上架")
@ApiImplicitParams({
        @ApiImplicitParam(name = "id", value = "服务id", required = true, dataTypeClass = Long.class),
})
public void onSale(@PathVariable("id") Long id) {
    serveService.onSale(id);
}
```

#### 4）测试

略



## 1.5 运营服务管理模块其它接口

### 1.5.1 删除区域服务开发

参考需求文档开发区域服务删除接口。

接口文档如下：

![这张图片展示的是删除区域服务开发接口的相关信息，接口类型为DELETE，访问路径为/foundations/operation/serve/{id}。请求数据类型为application/x-www-form-urlencoded，核心请求参数是id，参数说明为服务id，请求类型为path，为必填项，数据类型是integer(int64)。该接口的响应状态包含200（对应OK）、204（对应No Content）、401（对应Unauthorized）、403（对应Forbidden）四种情况，还设有响应参数，包含msg（响应消息，类型为string）和code（状态码，200成功、其他为失败，类型为integer(int32)）。这张图片是删除区域服务开发接口的文档内容，对应文档中1.5.1删除区域服务开发的接口文档部分。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Bj3db37cso71eYxyTMkcYEttnvc.png)



### 1.5.2 服务下架开发

参考需求文档开发服务下架接口。

![图片展示了区域服务下架的接口文档。方法为PUT，URL为/foundations/operation/serve/offSale/{id}，请求数据类型为application/x - www - form - urlencoded，响应数据类型为\[“*”\]。请求参数中id为服务id，是必填的path参数，数据类型为integer(int64)。响应状态包括200、201、401、403、404等状态码及说明。响应参数有msg和code，msg为响应消息，code为状态码，200为成功，其他为失败。该图片与文档中“服务下架开发”内容相关，是开发服务下架接口的参考文档。](https://feishu.cn/file/Fry9bYnxbof750xbGxGcuB9tnee)



### 1.5.3 设置热门服务开发

要求：实现更新serve表的是否热门字段。

接口文档：

![这张图片是区域服务设置热门的接口文档信息。该接口采用PUT请求方式，请求地址为/foundations/operations/serveronhot/(id)，请求数据类型为application/x-www-form-urlencoded。其请求参数中，id为服务id，参数类型为path，属于必填的integer(int64)类型。该接口的响应状态包含状态码200（说明为OK）、201（说明为Created）、401（说明为Unauthorized）、403（说明为Forbidden）、404（说明为Not Found）；响应参数包含msg（响应消息，string类型）、code（状态码，integer(int32)类型），响应数据类型为“*/*”。该接口对应的需求是更新serve表的是否热门字段。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/D8AKbnhY5oUFsPx4m4icgkxhnW4.png)



### 1.5.4 取消热门服务开发

要求：实现更新serve表的是否热门字段。

接口文档：

![这张图是“区域服务取消热门”的接口文档内容，属于第二章运营服务管理模块下的删除区域服务开发相关接口。图中明确了该接口的请求方式为PUT，接口路径为/foundations-operation/server-offici/{id}，请求数据类型为application/x-www-form-urlencoded；请求参数包含路径参数id，其为服务id，请求类型为path，是必填项，数据类型为integer(int64)。同时还展示了响应状态，包括状态码200对应OK、201对应Created，以及401、403、404等异常状态，也列明了响应参数，包含返回消息msg（string类型）和返回码code（integer(int32)类型），code为200代表成功，其他值代表失败。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/ECnvbhV48olaOTxzdCbcgeyonMe.png)

### 1.5.5 启用区域功能完善

参考业务流程图进行功能完善，如下图：

![图片是一张业务流程图，用于说明区域功能启用的流程。流程从确定一个区域开始，接着点击启用，判断当前状态是否为单租或禁用，若否则启用失败。若当前状态为单租或禁用，再判断区域下是否存在上架的服务，若存在则启用成功，若不存在则启用失败。该图与文档中“启用区域功能完善”内容相关，用于指导在区域下存在上架服务时启用功能的操作步骤。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/QdDPbs19doHXJLxY0SbcvYWbnmj.png)

增加校验：区域下存在上架的服务方可启用。



### 1.5.6 禁用区域功能完善

参考业务流程图进行功能完善，如下图：

![该图是禁用区域功能的业务流程图，位于文档“1.5.6禁用区域功能完善”的内容处，是对应功能的执行逻辑图示。流程从“确定一个区域”开始，经“点击禁用”后进入判断环节，首先判断“当前状态是否启用”，若判断结果为“否”则直接指向“禁用失败”；若判断结果为“是”，则进入关键校验步骤，即判断“区域下是否存在上架的服务”，该步骤有红色箭头标注突出显示；若判断区域存在上架服务，同样指向“禁用失败”；若区域不存在上架服务，则最终判定为“禁用成功”，至此流程结束，且该流程的校验规则与上下文标注的“区域下不存在上架的服务方可禁用”要求一致。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Mwbzbkwero6aXmxXHYHcNVItnJd.png)

增加校验：区域下不存在上架的服务方可禁用。



### 1.5.7 禁用服务项功能完善

参考业务流程图进行功能完善，如下图：

![图片 addCriterion图片是一张业务流程图，用于说明禁用服务项功能完善流程。流程从“确定一个服务项”开始，接着点击禁用，判断当前状态是否启用，若启用则判断该服务是否在区域上架，若存在关联的区域服务且状态为上架则禁用失败，若不存在则禁用成功；若当前状态不启用，则直接禁用成功。该图与文档中“禁用服务项功能完善”内容对应，用于指导业务操作。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Rvp3bR0tAoIJkxxU8wlcedOanBh.png)

增加校验：该服务没有在任何区域上架方可禁用。

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 云岚到家项目 / 第二章 运营服务管理模块 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/Jl6iwk6heihnK9kU4ITcI4Sjnff)
