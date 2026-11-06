---
type: memo
date: 2026-11-06
aliases:
  - "01-IOT产品设备管理"
---

# 01-IOT产品设备管理

## 目标

从今天开始，我们开始一个新的模块开发，叫做**智能监测**，我们可以先来看一张图

![图片展示了老人入住养老院时配备的智能硬件设备。画面中老人坐在椅子上，周围有多种设备，如定位报警胸牌、健康定位报警手表、跌倒报警器、门磁报警器、烟雾报警器、滞留/跌倒报警器、紧急呼叫报警器、睡眠监测带等，每种设备旁配有文字说明其功能。该图与文档中介绍老人入住养老院后可配备智能硬件设备的内容相关，直观呈现了安全和健康场景下的智能设备配置情况。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/BxL4bficLoo0hqx0cjgcwed4nSd.png)

老人入住养老院之后，可以通过选择的配置来配备智能硬件设备，它有两类场景：安全场景和健康场景

- 安全场景：防跌倒意外、防走失、睡眠中意外、区域滞留、环境安全
- 健康场景：健康监测、健康小屋；

通过硬件设备来监控老人以及老人活动场所的情况。后台时刻做监控，一旦有意外发生就可以迅速处理解决。

其中关于智能设备的数据传输和上报，采用的华为云提供的**设备接入服务**（IoTDA），我们也会重点讲解设备如何上报数据到IOT以及如何从IOT平台拉取数据

今日的目标：

- 能够熟悉华为云IoTDA平台的特点
- 能够掌握华为云IoTDA平台的一些概念，如：产品、设备、物模型等
- 能够掌握使用华为云IoTDA平台提供的接口创建设备，并绑定业务数据（老人、位置）
- 能够熟悉智能设备对接华为云IoTDA平台上报数据的方式



## 华为云IOT平台

在开发智能监测之前，我们必须要先熟悉和掌握华为云IoTDA平台的使用及对接，才能开发我们的项目

### 什么是物联网

把所有物品通过信息传感设备与互联网连接起来，进行信息交换，即物物相息，以实现智能化识别和管理。

物联网（英文：Internet of Things，缩写：IoT）起源于传媒领域，是信息科技产业的第三次革命。物联网是指通过信息传感设备，按约定的协议，将任何物体与网络相连接，物体通过信息传播媒介进行信息交换和通信，以实现智能化识别、定位、跟踪、监管等功能。

常见的应用场景：

- 共享充电宝
- 充电宝设备接入物联网平台后，可上报充电宝电量和借用状态等信息到物联网平台云端。充电宝用户扫码后，云端低延时向充电宝下发指令，使其弹出。同时，企业运营者能够实时获知充电宝的运行状况。
- 智能音箱 
- 播报音箱接入物联网平台后，用户扫码完成支付后，将支付金额实时通过音箱，向用户和商家进行语音播报。
- 智能家居
- 智能家居技术已经成为当今家庭装潢的一大特色。比如，通过智能灯泡，可以实现远程控制灯光和电视等设备，调节温度和湿度，实现智能化生活。
- 智能农耕
- 智能农耕可以通过物联网技术来监测、传输、分析、管理农业生产过程中的信息。比如作物的生长情况、土壤的状况等，以提高农业生产的效率，改善利润率，减少污染，节约农业资源。
- 智能医疗
- 在智慧医疗中，可以捕捉人的生理状态信息，例如心跳频率、体力消耗、血压高低等。然后对采集数据进行备份、加工和分析，以便个人或医生快速查询。在物联网平台接入传感器设备，采集人体及周边环境参数的信息，通过数据服务处理数据后，反馈给用户。



### 华为云IoTDA简介

产品文档：https://support.huaweicloud.com/iothub/index.html

设备接入服务（IoTDA）是华为云的物联网平台，提供海量设备连接上云、设备和云端双向消息通信、批量设备管理、远程控制和监控、OTA升级、设备联动规则等能力，并可将设备数据灵活流转到华为云其他服务。

我们作为一个开发者，基本的设备与后台调度思路，如下：

![图片展示了设备与业务应用通过华为云IoT平台的连接关系。左侧为设备，通过MQTT等协议接入华为云IoT平台；右侧为业务应用，通过HTTP/AMQP等协议接入。中间的华为云IoT平台是物联网平台，提供设备连接上云、双向消息通信等能力。右侧业务应用包括智慧城市、智慧园区等，体现了设备与业务应用通过平台进行数据交互。该图与文档中介绍华为云IoT平台能力的上下文相契合。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/IyHDbhFvQo9Jxxxu2eccF2BLnZf.png)



> 更多的介绍可以阅读官方产品文档

### 开通IoTDA

#### 注册华为云账号

前往[华为云官网](https://www.huaweicloud.com/)注册账号。如果已有注册账号，请跳过此步骤。

进入华为云首页后，如果没有华为云的账户需要先进行注册，才可以进行登录。由于注册较为简单，课程和讲义不在进行体现。

<callout emoji="🎈">
**注意：**
**注册成功后，需要实名认证才能正常开通服务 (微信扫码即可)**
</callout>



#### 开通物联网平台

登录账号以后，进入控制台，我们可以在产品中搜索**设备接入IoTDA**

![图片展示了华为云IoTDA服务的开通步骤中，搜索设备接入IoTDA后的界面。在控制台左侧菜单栏中，点击“设备接入”下的“IoTDA实例”后，搜索框内显示“设备接入IoTDA”被红色箭头指向。该图片与上文“登录账号后进入控制台，在产品中搜索设备接入IoTDA”相呼应，直观呈现了搜索操作后的结果，为后续开通IoTDA服务做准备。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/KHfhbAkXWoFXiixFCz6cmIjMnre.png)

打开之后，点击**开通免费单元**

![这张图片是华为云控制台中设备接入IoTDA的开通页面，页面左上角显示当前为华为云的设备接入服务界面，核心内容为带有红色箭头指向的“开通免费单元”按钮，该按钮是开通IoT平台的关键操作选项。页面还清晰列出了IoTDA的“标准版”和“企业版”两种服务版本，分别介绍了适用场景及核心特性，比如标准版适用于业务规模适中的企业客户，企业版面向中大型企业客户并具备更完善的安全与运维能力，页面左侧还有IoTDA实例管理相关的功能入口，整体呈现出开通IoTDA服务的操作界面及服务版本说明信息。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/OhbgbGVJGoJjL4xXYeQcsTAEnUd.png)

然后直接创建就可以了

![图片展示的是华为云控制台中“设备接入服务标准版”的创建页面。页面上方有区域、计算模式等设置选项，下方是规格配置区域，显示免费标准版的单位类型、网络连接数上限、每日消息数上限、上下行消息TPS等信息。页面底部有“免费创建”按钮，以及“立即创建”按钮，后者被红色箭头指向。该图片与上文开通物联网平台后创建IoTDA服务的内容相关，直观呈现了创建操作的页面界面。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/TNSvbWzfsoxSXfxhgSucClPlnub.png)

创建成功之后的效果：

![这张图片展示的是华为云IoTDA（设备接入）服务的管理页面，核心区域用红色框标注了创建成功的实例信息：该实例名称为freeStandardInstance，状态为运行中，下方标注了实例规格为标准版50°1；页面显示该实例的消息上下行QPS为10、每日消息上限为1万、同时在线设备上限为1000，还标注了“按需计费”的计费模式，以及实例的创建时间为2025/02/13 14:36:21、时区为GMT+08:00；页面旁设有“详情”“变更”等操作选项，完整呈现了按指引完成IoTDA服务开通后生成的具体实例效果。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/N2ajbVoovolQvvxNtjzc63i3nmc.png)

服务的基本信息，可以点击服务查看

![这张图片展示了华为云IoTDA开通成功后的实例管理界面，界面中清晰显示了一个名为freeStandardInstance的标准规格实例，其状态为“运行中”，实例ID为defc23a-2a0c-496f-86b5-050d6f1aa336。界面还标注了该实例的相关性能数据，包括消息上下行TPS为10、每日消息数上限为1万、同时在线设备数上限为1000，以及创建时间为2025年02月13日14:36:21 GMT+08:00。图中用红色箭头指向该实例的“freeStandardInstance”标识，并标注“点击”字样，提示用户可点击该实例进行后续操作，符合文档中“创建成功之后的效果”的描述，说明此时IoTDA服务已成功开通并进入可操作状态。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/HJJhbqZ9CoyekZx5Ms2cQlDrnhc.png)



![这张图片展示了华为云IoTDA服务开通成功后的控制台界面，界面顶部显示实例名称为freeStandardInstance，运行状态为“运行中”。页面内容分为设备接入服务的相关板块，涵盖快速体验、使用和设备体验、基于示例代码开发三个引导选项，以及实例基本信息、数据统计等核心信息模块。其中实例基本信息明确了实例ID、地域、运行时长等内容，数据统计板块显示在线设备数、消息收发数等多项指标，且有箭头标识的注册设备数、同时在线设备数区域。界面左侧还设有功能导航栏，可查看产品、资源等相关内容。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/KGWLbOwNkoLrlSx4Onlcrf6Hnwb.png)

到此，IoTDA服务就正式开通了。



### 产品

#### 创建产品

产品：某一类具有相同能力或特征的设备的集合称为一款产品。帮助开发者快速进行产品模型和插件的开发，同时提供端侧集成、在线调试、自定义Topic等多种能力。比如，手表、大门通道门禁、紧急呼叫报警器、滞留报警器、跌倒报警器

现在我们可以创建产品，找到产品-->创建产品

![图片展示的是华为云IOT平台中产品页面。页面左侧有“产品”“设备”“规则”等导航栏，右侧上方有“产品”“设备”“规则”等选项卡。页面中部有“创建产品”按钮，下方有“默认以创建产品ID进行搜索”输入框，以及“产品名称”“产品ID”“产品密码”“设备类型”“销货国家”“维修联系”等输入框。页面底部有“操作”按钮。该图片与上文“找到产品-->创建产品”及创建智能定位报警手表产品的内容相关，直观呈现了创建产品的操作界面。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/PAv8bolimouSKrxpHOOc9KDUnuh.png)

如下图，我们先创建**智能定位报警手表**的产品，输入信息如下：

![这张图是华为云物联网平台创建产品的操作界面，展示了创建智能定位报警手表产品时的信息填写页面。页面中，产品名称栏已输入“智能定位报警手表”，协议类型选MQTT、数据格式选JSON，设备类型选标准类型，所属行业选智能生活，所属子行业选智慧健康，设备类型选城轨检测设备。页面底部有“取消”和“确定”两个按钮，红色箭头重点指向黑色的“确定”按钮，用于提交已填写的产品信息以完成创建。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/LJnibui0AodexGxl6iSccWRWnog.png)

创建成功之后，在产品列表中也可以查看，刚刚创建的产品

![图片展示的是华为云IOT平台中产品管理页面。页面左侧有产品、设备、监控运维等选项，当前选中“产品”。右侧上方有“产品”“设备”“监控运维”等标签，下方有“创建产品”“默认应用产品名称搜索”等输入框，还列出了已创建的产品，如“智能定位报警手表”，其产品ID、需用空间、设备类型、协议类型、创建时间等信息均显示。该图与上文创建产品后在产品列表中查看的内容相关，直观呈现了产品创建成功后的状态。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/ROJkbVlP3oXScWxWzixc25olnre.png)



#### 产品模型

产品模型（Product Model），也称物模型，用于描述设备具备的能力和特性。开发者通过定义产品模型，在物联网平台构建一款设备的抽象模型，使平台理解该款设备支持的服务、属性、命令等信息。

- 手表本身：耗电量，使用时间
- 指标数据：身体血压、血氧、体温数据

像这些耗电量、血压、血氧数据都属于产品的功能，也叫做物模型

在IOT平台的物模型中，分为了三类：

<sheet sheet-id="BzvcF8" token="PH53s0637hb2AktwCDQcR6i8nuh"></sheet>

**创建手表产品的物模型**

进入产品的详情，我们选择自定义模型

![图片展示了华为云IOT平台中智能水表产品的物模型创建成功后的效果。上方显示产品名称、类型、协议、产品网络等基本信息。下方是模型定义部分，有基础服务、电量管理服务等分类，其中“设置水压读取周期”被选中。画面底部有“自定义模型”“上传模型文件”“Excel导入”“导入类模型”等操作按钮。该图与上文创建手表产品物模型的内容相关，直观呈现了创建成功后的界面情况。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/PmpEbMjN4opVZYxGSmNcasYunjf.png)

创建服务信息

![这张图片展示的是在华为云IoT平台创建智能手表产品物模型过程中，添加服务的操作弹窗界面。弹窗标题为“添加服务”，包含三项需填写的内容：带红色星号标记的“服务ID”已填写为“watch_services”，“服务类型”也填写为“watch_services”，“服务描述”则填写了“报警定位手表的服务”，描述字数显示为9/128。弹窗底部设有“取消”和“确定”两个操作按钮，完成填写后点击“确定”即可提交该服务的创建，是创建智能手表产品物模型流程里添加服务信息的关键操作环节。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/CNf8bbxi7oWfczxxRdMcXGG9nie.png)

然后可以给这个服务添加具体的物模型

![图片展示的是华为云IOT平台中智能手表产品的服务信息页面。页面上方有产品详情、插件开发、在线调试、Topic管理等标签。在服务信息部分，有watch_services服务，其描述为“智能定位手表服务”。下方“属性列表”区域有“新增属性”按钮，箭头指向该按钮。该图片与上下文关系紧密，上下文提到在创建智能手表产品的物模型时，可给服务添加物模型，此图直观呈现了添加属性的操作位置。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/VmvzbtmDDof16lxwTZccgz4knSe.png)

我们目前创建的智能手表产品的物模型，目前可以定义四个，如下：

| **属性名称** | **属性描述** | **数据类型** | **访问权限** | **取值范围** |
|-|-|-|-|-|
| BodyTemp | 体温 | int | 读写 | 0\~50 |
| xueyang | 血氧 | int | 读写 | 0\~150 |
| HeartRate | 心率 | int | 读写 | 0\~300 |
| BatteryPercentage | 用电量 | int | 读写 | 0\~100 |

下面是创建了一个**体温**的物模型，其他物模型创建类似

![图片展示的是华为云IOT平台中创建智能手表产品物模型时，新增体温属性的界面。界面上方显示“新增属性”，下方有必填项“属性名称”“属性描述”“数据类型”“访问权限”“取值范围”，其中“属性名称”为BodyTemp，“属性描述”为体温，“数据类型”为int（整型），“访问权限”为可读可写，“取值范围”为0 - 50。界面底部有“取消”和“确定”按钮。该图片与上文创建体温物模型的内容对应，直观呈现了创建过程中的属性设置界面。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/HTBGb02QRoi7AZxvtAkcwQvin9d.png)

创建成功之后的效果：

![图片展示了华为云IOT平台中智能手表产品创建成功后的物模型界面。界面上方显示“模型定义”及“添加服务”等选项。中间部分有“watch_services”服务列表，其描述为“智能定位手表的服务”。下方是属性列表，包括BodyTemp、xueyang、HeartRate、BatteryPercentage四个属性，分别对应体温、血氧、心率、用电量，数据类型均为int型，访问权限为可读可写，操作有复制、修改、删除。该图与上文创建手表产品物模型的内容对应，直观呈现了创建成功后的效果。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/HNQybVxfvoiQmFxIqWRceKarnBJ.png)



### 设备

归属于某个产品下的设备实体，每个设备具有一个唯一的标识码。设备可以是直连物联网平台的设备，也可以是代理子设备连接物联网平台的网关。

前提条件：设备是绑定在产品上的，所以必须先创建产品才行

操作步骤：

1. 在左侧导航栏，选择**设备-->所有设备**
2. 在**设备**页面，单击注册**设备**。
3. 在**添加设备**对话框中，输入设备信息，单击**确认**。

如下图：

![这是华为云IOT平台的单设备注册对话框界面，用于添加设备的相关操作。对话框内的关键信息有：所属资源空间为默认的DefaultApp_67adppsm（标注为可选），所属产品选择了智能定位报警手表；设备标识名称由用户自定义为watch01，设备ID自定生成，设备名称和描述分别为手表1号、定位报警手表1号，大小为82048；设备认证类型默认选中“密钥”选项。对话框右下方有“取消”和“确定”两个按钮，红色箭头指向了黑色的“确定”按钮，完成设备信息输入后点击该按钮即可完成设备注册。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/YxGdbyBPSooaP6xEx7IckN4Unzh.png)

创建成功之后，会分配一个设备的信息，非常重要，需要保存下来，方便后面调试链接

![图片展示的是华为云IOT平台设备创建成功后的提示界面。界面显示设备ID为67ad95860c504e29c72b2436_watch01，设备密钥不可见，下方有“查看SDK使用指南”链接。界面右上角有“X”关闭按钮，下方有一个红色箭头指向的“保存并关闭”按钮。该图片与上文设备创建成功后需保存设备信息的内容相关，直观呈现了设备创建成功后的提示及操作指引。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/E62ebzPtqobKQexCAQAcn7fxnNb.png)

设备中可以查看，创建好的设备

![图片展示的是华为云IOT平台“所有设备”页面。页面上方显示设备总数为1，包括数据设备0、在线设备0、离线设备1。下方设备列表中，有“注册设备”“删除”“新增”“连接”等操作按钮，以及“状态”“设备名称”“设备标识”“设备类型”“所属资源名称”“所属产品”“节点类型”等列，其中“未激活”状态的设备名称为“宇星1号”，设备标识为67a85580c354ec9c7，所属产品为DefaultApp_5745psum。该图与上文介绍的设备管理操作步骤相关，展示了创建成功后设备的查看界面。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/QipJbmupioxfDaxeXINcYEOCnNe.png)

设备详情

![这张图是华为云IOT平台的设备详情页面，分为上下两个核心区域。上方红框标注的是设备基本信息区域，清晰展示了设备名称为“手表1号”，以及设备的唯一标识、所属产品、认证类型、创建时间等关键信息，提示这些信息需妥善保存。下方红框为物模型区域，呈现了该设备关联的属性数据，包括体温、血压、心率、用电量等可采集的物模型数据，体现设备的功能参数配置。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/OhffbNmBtoSqYdxAO18cNx5Jnrd.png)



### 设备接入-数据上报

#### 概述

属性上报是一种需要平台解析、缓存，通过数据流转功能将数据转发到应用侧或华为云其他云服务的一种方式。需要在平台建立产品模型。对于属性上报的数据，平台会记录属性的最新一次上报值，对符合产品模型定义的属性数据进行存储。



官方案例-设备接入：[基于NB-IoT小熊派的智慧烟感](https://support.huaweicloud.com/bestpractice-iothub/iot_bp_0024.html)

![图片展示了基于NB - IoT小熊派的智慧烟感设备接入流程。前期准备包括购买Bear - Pi开发套件、安装VScode、安装IoT Link、编译/烧录；控制台开发有创建产品、定义产品模型、开发编解码插件、注册设备；业务实现为数据上报、命令下发。该图与文档中介绍的设备接入流程相关，直观呈现了从前期准备到业务实现的步骤。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/TXrwbYlDsoFkFAxHJOjc2cuKnCg.png)

我们在开发阶段可以使用联网的电脑，来模拟设备的数据上报，官方[sdk](https://github.com/huaweicloud/huaweicloud-sdk-java-v3/releases)下载

#### Java SDK使用示例

1. 前提，需要提前创建好产品和对应的物模型，以及该产品的设备
2. SDK代码获取：[SDK下载](https://github.com/huaweicloud/huaweicloud-iot-device-sdk-java)（由于github访问的效果不好，可以直接使用当天资料中提供好的）

`项目一：中州养老\中州养老-智能检测系统-华为云\资料\华为SDK\huaweicloud-iot-device-sdk-java-master.zip`

![这张图片展示的是一个名为“huaweicloud-iot-device-sdk-java-master.zip”的压缩包，该压缩包是华为云物联网设备端Java SDK的相关文件，对应文档中提到的供开发使用的华为SDK，存放于指定资料路径下。该压缩包文件位于项目资料的华为SDK目录中，开发者可将其解压后，通过IDEA打开对应项目模块，用于后续设备数据上报的相关操作，是Java SDK使用示例中需要用到的重要文件资源。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/E8zkb2TyPo2LUzxLWF4cvrAKnwg.png)

解压之后，使用idea打开，找到`iot-device-demo`模块中的`PropertySample`类，如下图

![图片展示了华为云IoT平台Java SDK中iot - device - demo模块下的src文件夹内容。其中，main文件夹下的java文件夹中包含多个示例类，如CommandSample、CustomDeviceRuleSample等。红色框突出显示了PropertySample类。该图片与文档中介绍Java SDK使用示例的内容相关，用于说明 自动生成图片中PropertySample类在项目结构中的位置，帮助理解后续在该类中进行设备数据上报操作的上下文。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/FWnvbUnZlo909sxx67jcYXxJnhb.png)



1. 完整代码，如下：

```Java
package com.huaweicloud.sdk.iot.device.demo.device;

import com.huaweicloud.sdk.iot.device.IoTDevice;
import com.huaweicloud.sdk.iot.device.client.requests.ServiceProperty;
import com.huaweicloud.sdk.iot.device.transport.ActionListener;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.nio.file.Files;
import java.security.SecureRandom;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Random;

import static java.nio.file.StandardCopyOption.REPLACE_EXISTING;

/**
 * 演示如何直接使用DeviceClient进行设备属性的上报和读写
 */
public class PropertySample {

    private static final String IOT_ROOT_CA_RES_PATH = "ca.jks";

    private static final String IOT_ROOT_CA_TMP_PATH = "huaweicloud-iotda-tmp-" + IOT_ROOT_CA_RES_PATH;

    private static final Logger log = LogManager.getLogger(PropertySample.class);

    public static void main(String[] args) throws InterruptedException, IOException {

        // 加载iot平台的ca证书，进行服务端校验
        File tmpCAFile = new File(IOT_ROOT_CA_TMP_PATH);
        try (InputStream resource = CommandSample.class.getClassLoader().getResourceAsStream(IOT_ROOT_CA_RES_PATH)) {
            Files.copy(resource, tmpCAFile.toPath(), REPLACE_EXISTING);
        }

        // 创建设备并初始化. 用户请替换为自己的接入地址。-->watch01
        IoTDevice device = new IoTDevice(
            //接入地址获取方式：登录华为云IoTDA控制台左侧导航栏“总览”页签，在选择的实例基本信息中，单击“接入信息”-->设备接入-->MQTT。选择8883端
            "ssl://230590aa95.st1.iotda-device.cn-east-3.myhuaweicloud.com:8883",
            //设备id
            "67ab39f1bab900244b1e64a6_watch01", 
            //设备秘钥  设备创建成功后会自动生成
            "5f71b853642b8ccab767890a5b148f02", tmpCAFile);
        if (device.init() != 0) {
            return;
        }

        // 定时上报属性
        while (true) {

            Map<String, Object> json = new HashMap<>();
            Random rand = new SecureRandom();

            // 按照物模型设置属性，根据实际情况设置，下面是智能报警手表的物模型
            json.put("BodyTemp", 36);
            json.put("xueyang", rand.nextFloat()*100.0f);
            json.put("HeartRate", rand.nextFloat()*100.0f);
            json.put("BatteryPercentage", rand.nextFloat() * 100.0f);

            ServiceProperty serviceProperty = new ServiceProperty();
            serviceProperty.setProperties(json);
            serviceProperty.setServiceId("watch_services"); // serviceId要和物模型一致

            device.getClient().reportProperties(Arrays.asList(serviceProperty), new ActionListener() {
                @Override
                public void onSuccess(Object context) {
                    log.info("pubMessage success");
                }

                @Override
                public void onFailure(Object context, Throwable var2) {
                    log.error("reportProperties failed" + var2.toString());
                }
            });

            Thread.sleep(10000);
        }
    }
}
```

右键启动之后的效果：

![图片展示了Java SDK使用示例中PropertySample类的代码及运行效果。代码中包含初始化日志、获取CA证书等操作。运行效果部分，显示了设备创建、连接、发布消息等日志信息，如“create device, the DeviceId is 67a05b60d50e5a...”“connect success, the url is ssl://33e7abe...”“publish message topic is $oc/devices/67a05b60d50e5a...”等，最后“pubMessage success”表明消息发送成功。该图片与上文介绍的Java SDK使用示例内容相关，直观呈现了代码运行结果。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/AgMBbDBZXoblPJxRtEYc2bTonwh.png)

消息发送成功后，可以在华为云IoTDA平台中查看该设备上报的数据

![图片展示的是华为云IoTDA平台中设备详情页面。页面上方有“设备管理”“设备详情”等导航栏，当前选中“设备详情”。下方设备信息区域显示设备名称为watch08，设备ID为67a4b5508c42c9c72b24，节点类型为离线设备，连接时间是2025/02/14 20:42:42 GMT+08:00。页面中部有“数据列表”区域，显示了BodyTemp、xinyang、HeartRate、BatteryPercentage等数据项及其对应数值，如BodyTemp为1，xinyang为67.246284等。该图与上文介绍的设备接入数据上报内容相关，展示了数据上报后的平台查看界面。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/M6Jxb6D2voiZkxxGKQ6cLNTUnke.png)



### 产品设备数据准备

下面两款产品在养老中后面的课程中还会使用到，大家需要抽时间自行创建



1）产品名称：烟雾报警器

| **属性名称** | **属性描述** | **数据类型** | **访问权限** | **取值范围** |
|-|-|-|-|-|
| CurrentHumidity | 湿度 | int | 读写 | 0\~100 |
| IndoorTemperature | 温度 | int | 读写 | -40\~80 |
| SmokeSensorState | 烟雾监测状态 | bool(布尔) | 读写 | 布尔值：  <br/>0：正常  <br/>1：检测到烟雾 |



2)产品名称：睡眠监测带

| **属性名称** | **属性描述** | **数据类型** | **访问权限** | **取值范围** |
|-|-|-|-|-|
| BedTime | 离床时间 | dateTime(日期时间) | 读写 | 300 |
| BedExitCount | 离床次数 | int | 读写 | 0\~10 |
| SleepPhaseState | 睡眠状态 | int | 读写 | 0\~2 |
| HeartRate | 心率 | int | 读写 | 1\~180 |
| RespiratoryRate | 呼吸率 | int | 读写 | 1\~150 |



## 设备管理

我们熟悉了IOT平台的基本概念以及基本操作后，可以现在来分析后台系统的功能需求，在后台管理系统中，是需要自己维护设备，不需要创建产品，因为产品直接物联网平台创建添加即可

需要单独维护设备的原因是，**设备需要跟养老院的老人或者位置进行绑定**，才能做到精准的监控

比如：

- 烟雾报警器需要绑定到某个房间
- 智能手表需要绑定某个老人

<whiteboard token="WVgewm75OhTTwCbkPpScw1WnnTg"></whiteboard>



### 需求分析

#### 设备管理列表页

找到智能监测->设备管理

![这张图是IOT产品设备管理的设备管理列表页，左侧目录栏红框标注了设备管理的相关子项，包括设备管理列表页、设备详情页等内容；右侧区域的设备列表清晰展示了设备名称、备注名称、所属产品、接入位置等信息，还设有设备名称搜索框、所属产品下拉选项、设备类型下拉选项，具备搜索功能，同时设有“同步数据”和“新增设备”按钮，设备列表每一行的操作栏还包含查看、编辑、删除等功能按钮，契合上下文提及的设备管理搜索、同步数据、新增设备等操作相关内容。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/JnYYbcsIcoSdwpxdx9fcxYhEncg.png)

- 搜索：可以根据设备名称模块插叙，根据产品和设备类型精确查询

  - 所属产品=【Iot平台-产品管理-产品名称】
  - 设备类型=随身设备、固定设备；
- 同步数据：点击【同步数据】后，获取华为云产品数据
- 新增设备：

  ![这张图片是IOT产品设备管理模块中新增设备的操作弹窗，界面标题为“新增设备”。弹窗内有多个带红色星号标记的必填项，包括设备名称、备注名称、所属产品、设备类型、接入位置，其中设备类型已默认选中“随身设备”单选按钮，固定设备选项为未选中状态。每个名称输入框右侧标注了可输入字符的上限，设备名称和备注名称的上限均为15个字符。弹窗底部设有“取消”和“确定”两个操作按钮，点击可完成新增设备的取消或提交操作。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/Fjg0bEiXZoOpV4xCIEgcYgOynuf.png)

  - 随身设备：表示绑定在老人身上的设备，如手表；
  - 固定设备：表示绑定在固定位置的设备，如睡眠监测带、门磁、摄像头；
  - 接入位置为随身设备=入退管理->入住管理->入住办理->老人姓名->当前已入住的老人；
  - 接入位置为固定设备=楼层、房间、床位 = 在住管理->床位管理->床位房型；

  ![图片展示的是设备管理列表页中“楼层名称”“房间名称”“床位名称”选项。左侧“楼层名称”下有“楼层名称”选项，右侧“房间名称”下无选项，右侧“床位名称”下无选项。该图片与上下文“设备管理列表页”相关，是对列表页中设备接入位置的展示，直观呈现了楼层、房间、床位等接入位置的选项情况。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/LxWubJak7oerfvxQdoecMYZ9nyh.png)
- 删除：该条设备从列表中移除。老人与设备自动解绑
- 编辑：先查看该条设备，然后再修改设备数据
- 查看：跳转到设备详情页
- 开门

  - 当所属产品中包含“门禁”字眼；操作中【开门】可点击，若不包含，【开门】按钮置灰；
  - 点击【开门】，若输出参数值是0，则显示开门失败，若参数是1，则显示开门成功；

#### 设备详情

当在列表点击**查看**按钮之后，可以查看设备详情页，如下效果

![这张图片展示的是IOT产品设备管理中的设备详情页，具体显示了“smoke alarm-01”烟雾报警器的相关设备信息。页面顶部标注“设备信息”，明确了设备名称、所属产品（智能烟感）、DeviceSecret、ProductKey、设备状态（在线）、接入位置等核心内容，下方设有“设备详情”和“物模型数据”两个切换标签。在设备详情区域，还展示了该设备的备注名称、设备类型、地域、节点类型、认证方式、创建人、创建时间、激活时间等详细信息。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/Ipf2bxMcJoKIe3xBJz0cYfFpnKd.png)



#### 物模型数据-运行状态

点击【物模型数据】，查询物模型运行状态，列表中的数据为上行指令数据；

![图片展示的是设备详情页中物模型数据部分。页面上方有“运行状态”和“服务调用”两个选项卡，其中“运行状态”被红色箭头指向突出显示。下方列表显示了10条物模型运行状态数据，包括序号、标识符、功能名称、最新时间及数据值等信息，如序号1的标识符为PowerConsumption，功能名称是用电量，最新时间为2048 - 10 - 15 15:00:00，数据值为10.0kWh。该图片与文档中介绍设备详情页物模型数据 - 运行状态的内容相关，直观呈现了数据展示效果。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/FOT2bWznloI0uyx8umwcLTe4nKg.png)

当点击了【查看数据】，出【查看数据】弹窗；

![这是“查看数据”的弹窗界面，用于展示物模型的上行指令数据。界面上方设有“时间范围”下拉选项，当前选择的是“1小时”；下方以列表形式呈现了序号、原始值、时间三类信息，共5条数据的原始值均为92.5kW·h，对应时间均为2048-10-15 15:00:00。界面底部标注总数据量为15项，当前分页设置为每页5条，处于第2页，支持切换页码与跳页操作，该弹窗是点击“查看数据”后弹出的。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/QaWWbug1GoXTImx12bkcAsgYnob.png)



#### 物模型数据-服务调用

点击【物模型数据】，查询物模型服务调用

![这张图是IOT设备管理的物模型数据-服务调用相关页面，页面顶部有“设备详情”和“物模型数据”两个选项卡，“物模型数据”选项卡已被选中；在该选项卡下方，“运行状态”和“服务调用”两个子选项中，“服务调用”处于选中状态。页面左侧的默认模块为“模块1-远程开门”，其标识符为“DoorOpen”。页面右侧的表格区域，时间范围默认设置为1小时，表格内展示了序号、时间、标识符、服务名称、输入参数、输出参数等信息，且有多条“远程开门”的服务调用记录，输入参数均为“开门”，输出参数包含code、data、id等内容。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/SEMkbdK0noVXhJxr49ScVWVZnaf.png)

- 列表中的数据为下行指令数据；
- 时间范围=1小时、24小时、7天

### 表结构说明

因为在本地需要维护设备数据，所以需要在本地创建设备表，数据也要存储，也需要创建设备数据表，如下：

<whiteboard token="LSNVwxbO1hqbNnbKRiycf5ipnnd"></whiteboard>

详细表字段说明：

![图片展示了设备管理中device和device_data表的字段说明。device表中，iot_id为物联网设备ID，secret为设备秘钥等；device_data表中，id为告警规则ID，device_name为设备名称等。图片与上文“详细表字段说明”内容对应，直观呈现了设备表和设备数据表的字段信息，为后续开发接口提供字段参考。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/XxupbSlsToTI57xHtYQcZ7Cbnfc.png)

sql表结构：**导入当天资料中的sql脚本，里面包含了智能检测模块的所有需要的表**

`项目一：中州养老\中州养老-智能检测系统-华为云\SQL脚本\device.sql`

### 接口分析

#### 接口列表

依据我们刚才的需求分析，在养老系统中需要维护设备数据，我们需要开发以下接口

- 从物联网平台同步产品列表
- 查询所有产品列表
- 注册设备
- 分页查询设备列表
- 查询设备详细数据
- 查看设备上报的数据
- 修改设备备注名称
- 删除设备
- 分页查询设备服务调用数据（暂不开发，后期接收到数据之后再完成）



#### 接口文档

接口文档参考：<cite doc-id="WVidwnWwtiHn2rkN2pvcBNUAn4c" file-type="wiki" title="智能监测-接口文档" type="doc"></cite>

接口文档中的**设备管理**部分

### IOT接口对接

我们刚才分析了功能中涉及到的接口，其中关于设备的维护（新增、删除、修改、查询），我们都需要在IOT平台中去操作，同时也需要在本地去保存一份，那为什么要保存两份呢？

IOT平台中只是维护了基础的设备信息，并没有跟业务数据进行绑定，比如，设备属于哪个位置，绑定了哪个老人。

<whiteboard token="ZriGwWifohofV0bX8CCcsum5n1b"></whiteboard>

只有设备绑定了业务数据，后期采集数据之后，我们才能针对性的进行排查问题。

所以，在接口的开发中，我们需要调用远程的接口维护设备，同时也需要在本地进行数据操作。关于远程接口，IOT平台给提供了丰富的API，利于我们去操作。

#### API列表

先查阅IOT官方提供API列表：[API列表](https://support.huaweicloud.com/api-iothub/iot_06_v5_0200.html)

目前业务中所需要的接口文档如下：

- 产品管理的API

  - [查询产品列表](https://support.huaweicloud.com/api-iothub/iot_06_v5_0080.html)
- 设备管理的API

  - [创建设备](https://support.huaweicloud.com/api-iothub/iot_06_v5_0046.html)
  - [查询设备详情](https://support.huaweicloud.com/api-iothub/iot_06_v5_0055.html)
  - [删除设备](https://support.huaweicloud.com/api-iothub/iot_06_v5_0041.html)
  - [修改设备](https://support.huaweicloud.com/api-iothub/iot_06_v5_1079.html)
- 设备属性

  - [查询设备属性](https://support.huaweicloud.com/api-iothub/iot_06_v5_0035.html)

#### 环境集成

IOT平台目前已经给提供了完整的**SDK**，我们可以快速集成到项目进行接口的调用，集成方式：[官方说明](https://support.huaweicloud.com/sdkreference-iothub/iot_10_10002.html)

在zzyl-framework模块导入坐标

```XML
<dependency>
    <groupId>com.huaweicloud.sdk</groupId>
    <artifactId>huaweicloud-sdk-core</artifactId>
    <version>3.1.76</version>
</dependency>
<dependency>
    <groupId>com.huaweicloud.sdk</groupId>
    <artifactId>huaweicloud-sdk-iotda</artifactId>
    <version>3.1.76</version>
</dependency>
```

在zzyl-admin模块中的application-dev.yml文件中添加关于IOT的配置如下：

<synced_reference src-block-id="JW6id4Iv7sHB5Tb5VIsc3ukXnfd" src-token="QR35dabMQoYx7txrqRecHdNdnCb"></synced_reference>

- ak和sk是华为云身份凭证，需要新增一个访问密钥，操作如下：

![图片展示了华为云控制台中新增访问密钥的操作界面。界面左侧为“我的凭证”选项卡，右侧是“访问密钥”相关设置区域。图中红色框突出显示了“新增访问密钥”按钮（①处），以及“访问密钥”区域（②处）。该图片与文档中“在zzyl-admin模块中的application-dev.yml文件中添加关于IOT的配置”上下文相关，用于说明在华为云控制台新增访问密钥的操作步骤，是获取ak和sk操作的辅助说明。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/EeEYbkvDEoshMWxzB3scWNEJnQq.png)

- endpoint  请在控制台的"总览"界面的"平台接入地址"中查看“应用侧”的https接入地址。

![图片展示的是IOT产品设备管理中设备管理的接入信息界面。左侧为实例基本信息，如实例ID、实例名称、实例类型等。右侧接入信息部分，接入类型有HTTPS、AMQP、MQTT等，其中HTTPS（443）的接入协议为HTTPS，接入地址为38e7ab...if.s1.xtda-app.cn-east-3.myhuaweicloud.com，还有应用接入、设备接入等信息。该图与上下文介绍的IOT接口对接相关，直观呈现了接入信息的具体内容。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/XGSjbNsjaoy2MuxumElc2GBCnwf.png)

- projectId项目ID，[获取项目ID](https://support.huaweicloud.com/api-iothub/iot_06_v5_1001.html)

![图片展示的是华为云IoT平台的API凭证页面。页面左侧有“API凭证”和“访问密钥”两个选项卡，当前选中“API凭证”。右侧显示了IAAS用户名、账号名、IAAS用户ID、账号ID等信息。下方项目列表中，有一个项目ID为57ee64c527a44319a077f0e7cb，项目名为CH - e88 - 3，所属区域为华北 - 上海一，该区域被红色框突出显示。此图与文档中获取项目ID及后续操作配置相关，用于展示获取项目ID时在平台的呈现位置。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/NbVWbV1YkoQBfix0W9XceiaynVc.png)

- host是AMQP访问的地址，获取方式如下：

![该页面为IOT接口对接中获取AMQP访问地址的相关内容，表格展示了不同接入类型对应的接入协议、端口号和接入地址，其中应用接入类的AMQP协议对应接入地址为“3s67aevdtf.s1.iota-app.cn-east-3.myhuaweicloud.com”，该地址被红色方框重点标注，页面顶部标注有“属于安全考虑，CoAP/CAPS的接入地址禁PING”的提示，部分接入行还设有“访问需凭证”“详情”的操作选项。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/WKEnbjVBnoXsnUxhVykcldvunmb.png)

- accessKey和accessCode从**预置接入凭证**获取

![图片展示了华为云IoT平台的接入地址配置界面。其中，应用接入的AMQP5 (5671)接入地址被红色框和箭头突出显示，其对应的接入网址为38e7abedbf st1 iotsta - app cn - east - 3 . myhuaweicloud . com 。该图片与文档中IOT接口对接部分内容相关，用于说明在zzyl - framework中新增HuaWeiIotConfigProperties来读取配置文件时，获取AMQP5接入地址的途径，即从该界面获取。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/H7skbfDOuo8cFVxAaY1cMcuxn7c.png)



在zzyl-framework中新增HuaWeiIotConfigProperties 来读取配置文件

```Java
package com.zzyl.framework.config.properties;

import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

/**
 * @author itheima
 */

@Data
@NoArgsConstructor
@Configuration
@ConfigurationProperties(prefix = "huaweicloud")
public class HuaWeiIotConfigProperties {

    /**
     * 访问Key
     */
    private String ak;

    /**
     * 访问秘钥
     */
    private String sk;

    /**
     * 区域id
     */
    private String regionId;

    /**
     * 应用侧https接入地址
     */
    private String endpoint;

    /**
     * 项目id
     */
    private String projectId;

    /**
     * 应用侧amqp接入地址
     */
    private String host;

    /**
     * amqp连接端口
     */
    private int port = 5671;

    /**
     * amqp接入凭证键值
     */
    private String accessKey;

    /**
     * amqp接入凭证密钥
     */
    private String accessCode;

    // 指定单个进程启动的连接数
    // 单个连接消费速率有限，请参考使用限制，最大64个连接
    // 连接数和消费速率及rebalance相关，建议每500QPS增加一个连接
    //可根据实际情况自由调节，目前测试和正式环境资源有限，限制更改为4
    private int connectionCount = 4;

    /**
     * 队列名称
     */
    private String queueName;

    /**
     * 开门命令所属服务id
     */
    private String smartDoorServiceId;

    /**
     * 开门记录属性
     */
    private String doorOpenPropertyName;

    /**
     * 开门命令
     */
    private String doorOpenCommandName;

    /**
     * 设置临时密码命令
     */
    private String passwordSetCommandName;

    /**
     * 仅支持true
     */
    private boolean useSsl = true;

    /**
     * IoTDA仅支持default
     */
    private String vhost = "default";

    /**
     * IoTDA仅支持PLAIN
     */
    private String saslMechanisms = "PLAIN";

    /**
     * true: SDK自动ACK（默认）
     * false:收到消息后，需要手动调用message.acknowledge()
     */
    private boolean isAutoAcknowledge = true;

    /**
     * 重连时延（ms）
     */
    private long reconnectDelay = 3000L;

    /**
     * 最大重连时延（ms）,随着重连次数增加重连时延逐渐增加
     */
    private long maxReconnectDelay = 30 * 1000L;

    /**
     * 最大重连次数,默认值-1，代表没有限制
     */
    private long maxReconnectAttempts = -1;

    /**
     * 空闲超时，对端在这个时间段内没有发送AMQP帧则会导致连接断开。默认值为30000。单位：毫秒。
     */
    private long idleTimeout = 30 * 1000L;

    /**
     * The values below control how many messages the remote peer can send to the client and be held in a pre-fetch buffer for each consumer instance.
     */
    private int queuePrefetch = 1000;

    /**
     * 扩展参数
     */
    private Map<String, String> extendedOptions;
}
```

在zzyl-framework中添加如下配置：

```Java
package com.zzyl.framework.config;

import com.huaweicloud.sdk.core.auth.BasicCredentials;
import com.huaweicloud.sdk.core.auth.ICredential;
import com.huaweicloud.sdk.core.region.Region;
import com.huaweicloud.sdk.iotda.v5.IoTDAClient;
import com.zzyl.framework.config.properties.HuaWeiIotConfigProperties;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class IotClientConfig {

    @Autowired
    private HuaWeiIotConfigProperties huaWeiIotConfigProperties;

    @Bean
    public IoTDAClient huaWeiIotInstance() {
        ICredential auth = new BasicCredentials()
                .withAk(huaWeiIotConfigProperties.getAk())
                .withSk(huaWeiIotConfigProperties.getSk())
                // 标准版/企业版需要使用衍生算法，基础版请删除配置"withDerivedPredicate"
                .withDerivedPredicate(BasicCredentials.DEFAULT_DERIVED_PREDICATE)
                .withProjectId(huaWeiIotConfigProperties.getProjectId());

        return IoTDAClient.newBuilder()
                .withCredential(auth)
                // 标准版/企业版：需自行创建Region对象，基础版：请使用IoTDARegion的region对象，如"withRegion(IoTDARegion.CN_NORTH_4)"
                .withRegion(new Region(huaWeiIotConfigProperties.getRegionId(), huaWeiIotConfigProperties.getEndpoint()))
                // .withRegion(IoTDARegion.CN_NORTH_4)
                .build();
    }
}
```

测试，在zzyl-admin模块下创建单元测试，查询产品列表，[接口说明](https://support.huaweicloud.com/api-iothub/iot_06_v5_0080.html)

详细代码如下：

```Java
package com.zzyl.test;

import com.huaweicloud.sdk.iotda.v5.IoTDAClient;
import com.huaweicloud.sdk.iotda.v5.model.ListProductsRequest;
import com.huaweicloud.sdk.iotda.v5.model.ListProductsResponse;
import com.huaweicloud.sdk.iotda.v5.model.Page;
import com.huaweicloud.sdk.iotda.v5.model.ProductSummary;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

@SpringBootTest
public class IoTDeviceTest {


    @Autowired
    private IoTDAClient client;
    
    /**
     * 查询公共实例下的所有产品
     * @throws Exception
     */
    @Test
    public void selectProduceList() throws Exception {

        ListProductsRequest listProductsRequest = new ListProductsRequest();
        listProductsRequest.setLimit(50);
        ListProductsResponse response = client.listProducts(listProductsRequest);
        List<ProductSummary> products = response.getProducts();
        System.out.println(products);
    }
    
}
```

测试结果如下

![这张图片展示了IOT接口对接调试后的单元测试结果，是在zzyl-admin模块下执行selectProductList接口测试后的返回信息。图片呈现了一条查询到的产品详情数据，包含appId、appName、productId等关键参数，其中deviceType标注该产品为皮肤检测设备，protocolType为MQTT，dataFormat为json，createTime为20250213T064734Z。这段测试结果对应文档中查询产品列表的接口说明，验证了IOT接口对接后查询产品列表的功能有效，成功获取到指定产品的相关配置与属性信息。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/LQTwbPKciounRlxb8wjcaLgonad.png)

### 功能实现

#### 基础代码准备

按照之前的套路，我们使用代码生成的功能，来生成代码

- 包名:`com.zzyl.nursing`
- 模块名：`nursing`
- 除了主键之外的Long类型，改为Integer

**只拷贝后端代码到idea中**

![图片展示的是代码生成界面，其中“device”表被选中。该表序号为1，表名称为“device”，表描述为“设备表”，实体为“Device”，创建时间为2024 - 09 - 20 04:41:56，更新时间为2024 - 09 - 20 04:42:49。界面下方有分页显示，当前显示第1页，每页10条记录，共1条记录。此图与上下文关系为：在基础代码准备阶段，按照代码生成功能生成代码时，需将物联网平台的“device”表相关代码拷贝到idea中，此图展示了该表信息，辅助理解代码生成操作。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/SVdJbnwIaojEMXxrBlhcZiZKn9b.png)

**同时删除DeviceController中除了list方法之外的其他方法，如下代码**

```Java
/**
 * 智能设备Controller
 * 
 * @author ruoyi
 * @date 2024-09-20
 */
@RestController
@RequestMapping("/nursing/device")
@Api(tags = "智能设备的接口")
public class DeviceController extends BaseController
{
    @Autowired
    private IDeviceService deviceService;
    
    /**
     * 查询设备列表
     */
    @PreAuthorize("@ss.hasPermi('elder:device:list')")
    @GetMapping("/list")
    @ApiOperation("查询设备列表")
    public TableDataInfo list(Device device) {
        startPage();
        List<Device> list = deviceService.selectDeviceList(device);
        return getDataTable(list);
    }

    
}
```

#### 从物联网平台同步产品列表

##### 思路分析

我们先来完成前四个接口，分别是：从物联网平台同步产品列表、查询所有产品列表、注册设备、分页查询设备列表，这四个接口都与产品有关系

- 物联网中的产品是由在物理网平台进行维护，设备是养老项目后台管理并同步到物联网平台
- 其中注册设备、分页查询设备列表都需要用到产品数据

由于以上两条原因，我们需要先让物联网平台的产品数据同步到后台，然后再被注册设备、分页查询设备列表这两个接口所引用



以下就是物联网产品与养老后台同步的思路

<whiteboard token="CWSdwl04Fhr563baCBRcb7NFnMh"></whiteboard>

##### 接口定义

在DeviceController定义同步的方法，详细如下：

```JSON
@PostMapping("/syncProductList")
@ApiOperation(value = "从物联网平台同步产品列表")
public AjaxResult syncProductList() {
    deviceService.syncProductList();
    return success();
}
```

##### 业务层

在IDeviceService类中定义同步的方法，如下：

```Java
/**
 * 从物联网平台同步产品列表
 */
void syncProductList();
```

实现类的方法：

```Java
@Autowired
private RedisTemplate<String,String> redisTemplate;

@Autowired
private IoTDAClient client;

/**
 * 同步产品列表
 */
@Override
public void syncProductList() {
    //请求参数
    ListProductsRequest listProductsRequest = new ListProductsRequest();
    //设置条数
    listProductsRequest.setLimit(50);
    //发起请求
    ListProductsResponse response = client.listProducts(listProductsRequest);
    if(response.getHttpStatusCode() != 200){
        throw new BaseException("物联网接口 - 查询产品，同步失败");
    }
    //存储到redis
    redisTemplate.opsForValue().set(CacheConstants.IOT_ALL_PRODUCT_LIST, JSONUtil.toJsonStr(response.getProducts()));

}
```

> 其中的缓存常量，可以自己定义
> 
> 类com.zzyl.common.constant.CacheConstants
> 
> public static final String *ALL_PRODUCT_KEY* = "iot:all_product";

##### 测试

点击了【同步数据】按钮之后，可以查看redis客户端，是否保存数据成功

![这张图片展示了Redis客户端的界面内容，界面左侧的目录中显示存在key为iot:all_product的存储项，这一存储项的名称正是上下文里提到的缓存常量ALL_PRODUCT_KEY对应的键值。界面右侧红框内展示了该key对应存储的具体数据，是两条产品信息的JSON格式内容，每条信息包含apikey、产品名称、产品ID、设备类型等字段，其中两条产品的设备类型都为RACIT，创建时间均为20250215125175452，这一界面正好对应文档中点击【同步数据】按钮后，验证物联网平台产品数据同步到后台并保存到Redis的测试场景。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/D1zWb4ozAo8k2KxR9BMckt3OnRf.png)

#### 查询所有产品列表

##### 思路分析

依据上一个接口，一旦redis中被同步了产品数据，我们只需要从redis中获取数据返回即可

##### 接口定义

在DeviceController中新增方法，如下：

```Java
@GetMapping("/allProduct")
@ApiOperation(value = "查询所有产品列表")
public AjaxResult allProduct() {
    return success(deviceService.allProduct());
}
```

##### 业务层

在DeviceService中新增方法如下：

```Java
/**
 * 查询所有产品列表
 *
 * @return 产品列表
 */
List<ProductVo> allProduct();
```

ProductVo 

```Java
package com.zzyl.nursing.vo;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

/**
 * 产品信息响应模型
 *
 * @author itcast
 **/
@Data
@ApiModel("产品信息响应模型")
public class ProductVo {
    /**
     * 产品的ProductKey,物联网平台产品唯一标识
     */
    @ApiModelProperty("产品的ProductKey,物联网平台产品唯一标识")
    private String productId;

    /**
     * 产品名称
     */
    @ApiModelProperty("产品名称")
    private String name;
}
```

实现方法：

```Java
/**
 * 查询所有产品列表
 *
 * @return
 */
@Override
public List<ProductVo> allProduct() {
    //从redis中查询数据
    String jsonStr = redisTemplate.opsForValue().get(CacheConstants.IOT_ALL_PRODUCT_LIST);
    //如果数据为空，则返回一个空集合
    if(StringUtils.isEmpty(jsonStr)){
        return Collections.emptyList();
    }
    //解析数据，并返回
    return JSONUtil.toList(jsonStr, ProductVo.class);
}
```

##### 测试

在前端页面中的所属产品中可以正常查看同步之后的产品列表数据

![图片展示的是设备管理页面中所属产品选择的弹窗界面。弹窗内有“请选择”下拉框，可选择“智能小音响”“智能电视”“睡眠监测带”“烟雾报警器”“智能定位报警手表”等产品。下方列表显示序号、设备名称、设备标识码、接入位置、设备类型、创建时间及操作选项。该弹窗与文档中“当新增设备时，选择随身设备会弹窗选择已经入住的老人”需求说明相关，用于辅助用户选择设备产品。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/E9vBbYhKXo85xvxpwsocwmS2nNd.png)

#### 查询已经入住的老人列表

##### 需求说明

当新增设备的时候，如果选择的的**随身设备**，则会弹窗选择已经入住的老人，如下图

![这张图展示了物联网设备管理业务中“新增设备”功能的操作流程，左侧是“添加设备”弹窗，用户需依次输入设备标识码、设备名称，选择所属产品，将设备类型设为“随身设备”后点击“接入位置”选择框；点击该选择框后，会唤起右侧的“查看数据”弹窗，该弹窗包含老人姓名、身份证号的搜索栏，以及已入住老人的列表信息，列表展示了序号、老人姓名、身份证号与床位号，底部还可切换分页，体现了添加随身设备时选择已入住老人的操作逻辑，与上下文中新增随身设备时弹窗选老人的需求说明相对应。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/PmtYbmwuSopGfNxiUclcvW3jnNi.png)

该接口已经在之前的代码生成中完成



#### 注册设备

##### 思路分析

整体思路如下：

<whiteboard token="WCYGwwChRhFdkib31m7c7tCqnrh"></whiteboard>



##### 接口定义

在DeviceController中新增方法，如下：

```Java
@PostMapping("/register")
@ApiOperation(value = "注册设备")
public AjaxResult registerDevice(@RequestBody DeviceDto deviceDto) {
    deviceService.registerDevice(deviceDto);
    return success();
}
```

基于接口文档，定义DeviceDto 如下代码，其中的RegisterDeviceRequest为物联网SDK提供

```Java
package com.zzyl.nursing.dto;


import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

@Data
@ApiModel(value = "设备注册参数")
public class DeviceDto {

    private Long id;

    /** 备注 */
    private String remark;

    /**
     * 设备标识码，通常使用IMEI、MAC地址或Serial No作为node_id
     */
    @ApiModelProperty(value = "设备标识码", required = true)
    private String nodeId;

    @ApiModelProperty(value = "设备id")
    public String iotId;

    @ApiModelProperty(value = "产品的id")
    public String productKey;

    @ApiModelProperty(value = "产品名称")
    private String productName;

    @ApiModelProperty(value = "位置名称回显字段")
    private String deviceDescription;

    @ApiModelProperty(value = "位置类型 0 老人 1位置")
    Integer locationType;

    @ApiModelProperty(value = "绑定位置")
    Long bindingLocation;

    @ApiModelProperty(value = "设备名称")
    String deviceName;

    @ApiModelProperty(value = "物理位置类型 0楼层 1房间 2床位")
    Integer physicalLocationType;
}
```

##### 业务层

在IDeviceService接口中新增方法，如下：

```Java
/**
 * 注册设备
 * @param deviceDto
 */
void registerDevice(DeviceDto deviceDto);
```

实现方法

```Java
/**
 * 注册设备
 *
 * @param deviceDto
 */
@Override
public void register(DeviceDto deviceDto) {
    //1. 判断设备名称是否重复
    Long count = lambdaQuery()
            .eq(Device::getDeviceName, deviceDto.getDeviceName())
            .count();
    if (count > 0) {
        throw new BaseException("设备名称重复");
    }
    //2. 判断设备标识是否重复
    count = lambdaQuery()
            .eq(Device::getNodeId, deviceDto.getNodeId())
            .count();
    if (count > 0) {
        throw new BaseException("设备标识码重复");
    }
    //判断如果是随身设备，将物理位置设置为-1
    if (deviceDto.getLocationType() == 0) {
        deviceDto.setPhysicalLocationType(-1);
    }

    //3. 判断同一位置是否绑定了相同的产品
    count = lambdaQuery()
            .eq(Device::getBindingLocation, deviceDto.getBindingLocation())
            .eq(Device::getProductKey, deviceDto.getProductKey())
            .eq(Device::getLocationType, deviceDto.getLocationType())
            .eq(Device::getPhysicalLocationType, deviceDto.getPhysicalLocationType())
            .count();
    if (count > 0) {
        throw new BaseException("同一位置绑定了相同的产品");
    }

    //4. 注册设备到华为iot平台
    //随机生成密钥
    String secret = UUID.randomUUID().toString().replaceAll("-", "");

    AddDeviceRequest addDeviceRequest = new AddDeviceRequest();
    AddDevice addDevice = new AddDevice();
    addDevice
            .withDeviceName(deviceDto.getDeviceName())
            .withProductId(deviceDto.getProductKey())
            .withNodeId(deviceDto.getNodeId())
            .withAuthInfo(new AuthInfo()
                    .withSecret(secret));

    addDeviceRequest.withBody(addDevice);
    AddDeviceResponse response = null;
    try {
        response = ioTDAClient.addDevice(addDeviceRequest);
    } catch (Exception e) {
        log.error("注册设备失败", e);
        throw new BaseException("注册设备失败");
    }


    //5.保存设备数据到数据库
    Device device = BeanUtil.toBean(deviceDto, Device.class);
    device.setSecret(secret);
    device.setIotId(response.getDeviceId());
    save(device);
}
```

##### 测试

在前端页面中查看是否可以保存设备成功



#### 分页查询设备列表

该接口在生成的代码中已经实现

其中的搜索条件，需要在后台添加数据字典：

![图片展示的是设备管理页面，上方有搜索、新增、修改、删除、导出操作按钮。下方表格中，设备类型为“随身设备”的编号为109，编号为110的设备类型为“固定设备”。表格还列有编号、设备类型、设备名称、设备标识、设备排序、状态、备注、创建时间、操作等信息。其中，编号104的设备位置被红色框突出显示，其编号为104，设备位置为“device_location_type”。该图片与文档中分页查询设备列表的上下文相关，展示了设备管理页面中设备类型及编号等关键信息。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/LJdZbPVkGoVUQ6xzT3kcVHOAn5b.png)

最终的效果：

![图片展示的是设备管理页面，上方有设备名称、产品名称等搜索框，下方是设备列表。列表中包含序号、设备名称、设备标识码、产品名称、入住大区、设备类型、创建时间等信息 addCriterion()函数](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/SGbubhzlJojqTxxsuxgc4MNTnsd.png)



#### 查询设备详细数据

当点击了列表中的【查看】按钮，可以查看设备的详细数据

##### 思路分析

这里的查询是有两部分数据的，第一个是数据库中存储的设备数据，第二个是物联网中的设备数据，查询流程如下

<whiteboard token="I4rqwfGcyhmKvPbJinacOtYKnEL"></whiteboard>



##### 接口定义

在DeviceController中定义方法，如下：

```Java
/**
 * 获取设备详细信息
 */
@GetMapping("/{iotId}")
@ApiOperation("获取设备详细信息")
@ApiImplicitParams({
        @ApiImplicitParam(name = "iotId", value = "物联网设备id", required = true, dataTypeClass = String.class)
})
public AjaxResult getInfo(@PathVariable("iotId") String iotId) {
    return success(deviceService.queryDeviceDetail(iotId));
}
```



##### 业务层

在DeviceService中新增方法，如下：

```Java
/**
 * 查询设备详情
 * @param iotId
 * @return
 */
DeviceDetailVo queryDeviceDetail(String iotId);
```

*DeviceVo* 

```Java
package com.zzyl.nursing.vo;

import com.fasterxml.jackson.annotation.JsonFormat;
import com.zzyl.common.annotation.Excel;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

import java.time.LocalDateTime;

/**
 * @author itcast
 */
@Data
@ApiModel("设备详情响应模型")
public class DeviceDetailVo {

    /**
     * 设备id
     */
    @ApiModelProperty(value = "设备id")
    private Long id;

    /**
     * 物联网设备id
     */
    @ApiModelProperty(value = "物联网设备id")
    private String iotId;

    /**
     * 设备名称
     */
    @ApiModelProperty(value = "设备名称")
    private String deviceName;

    /**
     * 设备标识码
     */
    @ApiModelProperty(value = "设备标识码")
    private String nodeId;

    /**
     * 设备秘钥
     */
    @ApiModelProperty(value = "设备秘钥")
    private String secret;

    /**
     * 产品id
     */
    @ApiModelProperty(value = "产品id")
    public String productKey;

    /**
     * 产品名称
     */
    @ApiModelProperty(value = "产品名称")
    public String productName;

    /**
     * 位置类型 0 随身设备 1固定设备
     */
    @ApiModelProperty(value = "位置类型 0 随身设备 1固定设备")
    private Integer locationType;

    /**
     * 绑定位置,如果是随身设备为老人id，如果是固定设备为位置的最后一级id
     */
    @ApiModelProperty(value = "绑定位置,如果是随身设备为老人id，如果是固定设备为位置的最后一级id")
    private Long bindingLocation;


    /**
     * 接入位置
     */
    @ApiModelProperty(value = "接入位置")
    private String remark;

    /**
     * 设备状态，ONLINE：设备在线，OFFLINE：设备离线，ABNORMAL：设备异常，INACTIVE：设备未激活，FROZEN：设备冻结
     */
    @ApiModelProperty(value = "设备状态，ONLINE：设备在线，OFFLINE：设备离线，ABNORMAL：设备异常，INACTIVE：设备未激活，FROZEN：设备冻结")
    private String deviceStatus;

    /**
     * 激活时间
     */
    @ApiModelProperty(value = "激活时间,格式：yyyy-MM-dd HH:mm:ss")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private LocalDateTime activeTime;

    /**
     * 创建时间
     */
    @ApiModelProperty(value = "创建时间,格式：yyyy-MM-dd HH:mm:ss")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private LocalDateTime createTime;

    /**
     * 创建人id
     */
    @ApiModelProperty(value = "创建人id")
    private Long createBy;

    /**
     * 创建人名称
     */
    @ApiModelProperty(value = "创建人名称")
    private String creator;

    /** 位置备注 */
    @ApiModelProperty(value = "位置备注")
    private String deviceDescription;
}
```



实现方法：

```Java
/**
 * 查询设备详情
 *
 * @return
 */
@Override
public DeviceDetailVo queryDeviceDetail(String iotId) {
    //查询数据库
    Device device = getOne(Wrappers.<Device>lambdaQuery().eq(Device::getIotId, iotId));
    if (ObjectUtil.isEmpty(device)) {
        return null;
    }

    //调用华为云物联网接口
    ShowDeviceRequest request = new ShowDeviceRequest();
    request.setDeviceId(iotId);
    ShowDeviceResponse response;
    try {
        response = client.showDevice(request);
    } catch (Exception e) {
        log.info("物联网接口 - 查询设备详情，调用失败:{}", e.getMessage());
        throw new BaseException("物联网接口 - 查询设备详情，调用失败");
    }

    //属性拷贝
    DeviceDetailVo deviceVo = BeanUtil.toBean(device, DeviceDetailVo.class);

    deviceVo.setDeviceStatus(response.getStatus());
    String activeTimeStr = response.getActiveTime();
    if(StringUtils.isNotEmpty(activeTimeStr)){
        LocalDateTime activeTime = LocalDateTimeUtil.parse(activeTimeStr, DatePattern.UTC_MS_PATTERN);
        //日期时区转换
        activeTime = activeTime.atZone(ZoneId.from(ZoneOffset.UTC))
                .withZoneSameInstant(ZoneId.of("Asia/Shanghai"))
                .toLocalDateTime();
        deviceVo.setActiveTime(activeTime);
    }

    return deviceVo;
}
```

##### 测试

测试效果如下：

![这张图展示的是01-IOT产品设备管理模块中，查询设备详细数据功能里的设备详情页面。页面顶部有“首页”“设备管理”“设备详情”三个标签，当前选中“设备详情”标签。页面主体呈现了该设备的全部详细信息，包含左侧的设备名称、设备ID、密钥、接入位置、节点类型、创建时间，右侧的设备标识码、所属产品、设备状态、设备类型、认证类型、激活时间等内容，其中设备状态显示为离线，设备是名称为手表98号的智能定位预警手表。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/KZmSbIZptomVxGxyGr0c4WP2nHb.png)

#### 查看设备上报的数据(设备影子数据)

##### 接口定义

- 华为云接口文档：https://support.huaweicloud.com/api-iothub/iot_06_v5_0079.html

在DeviceController中新增方法，如下：

接收的参数就是**阿里接口的请求参数**

```Java
/**
 * 查询设备上报数据
 */
@GetMapping("/queryServiceProperties/{iotId}")
@ApiOperation("查询设备上报数据")
@ApiImplicitParams({
        @ApiImplicitParam(name = "iotId", value = "物联网设备id", required = true, dataTypeClass = String.class)
})
public AjaxResult queryServiceProperties(@PathVariable("iotId") String iotId) {
    AjaxResult ajaxResult = deviceService.queryServiceProperties(iotId);
    return ajaxResult;
}
```

##### 业务层

在IDeviceService中定义新的方法

```Java
/**
 * 查询设备上报数据
 * @param iotId
 * @return
 */
AjaxResult queryServiceProperties(String iotId);
```

实现方法：

```Java
/**
 * 查询设备上报数据
 *
 * @param iotId
 * @return
 */
@Override
public AjaxResult queryServiceProperties(String iotId) {

    ShowDeviceShadowRequest request = new ShowDeviceShadowRequest();
    request.setDeviceId(iotId);
    ShowDeviceShadowResponse response = client.showDeviceShadow(request);
    if (response.getHttpStatusCode() != 200) {
        throw new BaseException("物联网接口 - 查询设备上报数据，调用失败");
    }
    List<DeviceShadowData> shadow = response.getShadow();
    if(CollUtil.isEmpty(shadow)){
        List<Object> emptyList = Collections.emptyList();
        return AjaxResult.success(emptyList);
    }
    //返回数据
    JSONObject jsonObject = JSONUtil.parseObj(shadow.get(0).getReported().getProperties());

    List<Map<String,Object>> list = new ArrayList<>();

    //处理上报时间日期
    LocalDateTime activeTime =  LocalDateTimeUtil.parse(shadow.get(0).getReported().getEventTime(), "yyyyMMdd'T'HHmmss'Z'");
    //日期时区转换
    LocalDateTime eventTime = activeTime.atZone(ZoneId.from(ZoneOffset.UTC))
            .withZoneSameInstant(ZoneId.of("Asia/Shanghai"))
            .toLocalDateTime();

    jsonObject.forEach((k,v)->{
        Map<String,Object> map = new HashMap<>();
        map.put("functionId",k);
        map.put("value",v);
        map.put("eventTime",eventTime);
        list.add(map);
    });

    return AjaxResult.success(list);

}
```

##### 测试

在本地的后台系统中新增一个设备，并绑定老人或位置，开启设备数据上报

![图片展示的是设备管理中查看设备上报数据（设备影子数据）的测试效果。上方为设备信息，包括设备名称、设备ID、密钥、所属产品、设备类型等。下方是物联网模型数据，以表格形式呈现，包含序号、标题、更新时间、数据值等信息，如HeartRate、BedTime、SleepPhaseState等数据项及其对应时间与数值。该图片与文档中“查看设备上报的数据(设备影子数据)”部分对应，直观呈现了设备上报数据的展示样式。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-si-zhang-zhi-neng-jian-ce-xi-tong-hua-wei-yun/Vsp4b9w3goG0PexXSRDcHsisn3e.png)





#### 修改设备

参考接口文档自行实现(也要参考华为云官方接口文档)

#### 删除设备

参考接口文档自行实现(也要参考华为云官方接口文档)



## 作业

- <cite doc-id="VTvtwnexJip9y8kYiTUcgpq1nCG" file-type="wiki" title="【作业】护理项目和预约" type="doc"></cite>
- 苍穹外卖小程序端功能

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第四章-智能监测系统（华为云） / 01-IOT产品设备管理 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/FXXFwJLEMi8Yx9kI2YbcRXZhnfc)
