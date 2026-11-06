---
type: memo
date: 2026-11-06
aliases:
  - "第五章-分组实战"
---

# 第五章-分组实战

## 实战要求

### 分组规则

- 大组拆为小组，小组人数：不超过5人（最多5人，至少4人）
- 就近原则：优先同桌临近，然后前后桌临近，不允许跨排分组
- 组长：组内自行选举一名组长，如果选举不出来，找老师进行指定
- 其他情况，由大组长解决或找老师协调解决
- 每个小组都要完成苍穹外卖和中州的代码

### 组长责任

- 管理和协调组内成员，组织团队成员业务或技术讨论
- 合理安排任务，推进业务开发
- 制定开发计划，参考开发计划：<cite doc-id="Xo8Xw0ac0iQwubkY3Bic7nerndd" file-type="wiki" title="开发计划" type="doc"></cite>

### git仓库要求

- 由组长在gitee中创建一个仓库，上传基础代码（今天资料已提供）
- 添加组内成员对仓库拥有开发权限或者管理权限（一个账号最多可以添加5人进行开发）
- 分支管理

  - 组内开发人员每人基于master分支创建一个分支，以自己的名字命名（例子：dev-lichuanbo）
  - 组内开发人员完成任务后提交代码到自己的分支，所有成员完成开发后，合并分支代码到dev-nursing
  - 合并分支dev-nursing之后，小组内部进行测试（功能测试，代码测试），测试都通过后，把dev-nursing分支合并到master分支

<whiteboard token="L76iwr6Mlhmo0IbB8sZcn5R3nPf"></whiteboard>

### 小组提交物料及答辩

- 代码：gitee代码仓库地址（需开源）

### 其他要求

- 老师就是产品经理，同学们有任何有疑惑的需求，都可以找老师沟通
- 技术求助：优先团队内部解决，原则上老师不帮助任何团队解决技术问题及bug



## 小程序端需求

### 需求说明

1. 绑定家人

![这是小程序端“绑定家人”功能的操作界面，属于该模块需求说明中的第一项内容。界面顶部显示功能标题“绑定家人”，带有返回箭头和小程序操作入口，需填写的信息项均标注为“必填”，分别是“家人姓名”“身份证号”“您称呼他/她”，每个输入项右侧设有对应的提示输入区域，底部设有醒目的橙色“立即绑定”按钮，用于提交绑定操作，该功能要求录入的信息对应为养老院入住的老人内容。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/BW4PbOF8to0J06xI1uncA8YLnQc.png)

**家人姓名**和**身份证号**必须是养老院入住的老人



1. 家人列表

![图片展示的是小程序端“我的家人”页面。页面显示老人姓名为“刘备”，床位信息为“豪华单人间-101-1床”，并有“我的账单”“健康数据”“解绑”三个功能按钮。下方有一个橙色的“绑定家人”按钮。该图片对应文档中“家人列表”需求说明部分，直观呈现了小程序端家人列表页面的样式及功能按钮位置，为用户操作指引提供参考。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/QwCwbVrngosIwJxFN4VcvpBhnwd.png)

- 其中展示**健康数据**按钮的前提是，该老人必须绑定手表设备
- 点击**解绑**按钮，可以解绑老人



1. 健康数据

![图片展示的是小程序端健康数据页面。页面显示老人姓名为刘备，床位信息为空。心率98次/分，血压140/87mmHg，血氧98%，体温35.4℃。心率下方有红色框和箭头，标注“点击心率 只有心率是动态的”。页面还展示了心率、血压、血氧、体温的图标及对应数据，下方有健康知识卡片。该图片与文档中“小程序端需求 - 健康数据”部分对应，直观呈现了健康数据页面的样式及部分功能。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/JsV9bmLZ9ogYYQxd8ucc6iqdnId.png)



1. 按天统计查询指标数据

![图片展示的是小程序端健康数据页面。页面上方显示日期为7月10日，有“日”“周”切换选项，当前选中“日”。下方图表显示心率数据，横轴为时间，纵轴为心率次数，图表中有一条折线图，标注了各时间点心率数值。图表下方有心率正常范围提示。页面底部有“暂无异常数据哦~”的提示语，还配有小动物图案。该页面是按天统计查询指标数据需求的呈现，与文档中健康数据部分上下文对应。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/CKCcbg8NAoGmVmx1bzIctwegnJe.png)



1. 按周统计查询指标数据

![图片展示的是小程序端健康数据页面。上方显示日期为7月10日，心率99次/分钟，心率正常范围60 - 100次/分钟。下方有心率折线图，呈现7月4日至10日心率变化情况。右上角有“07月04日 - 07月10日”日期选择框。底部有“暂无异常数据哦~”提示，画面中间有一个卡通形象。该图与文档中小程序端需求中健康数据部分上下文对应，直观呈现了心率数据及相关信息。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/DAG3bx9xeosU3wxb5kCcG47XnEb.png)



### 表结构

![这张图是小程序端需求表结构的实体关系图，展示了三个数据库表的关联逻辑，分别为老人家属表family_member、老人关联表family_member_elder、老人主表elder（老人表）。其中三个表的核心字段明确，family_member_elder表通过family_member_id字段关联family_member表，通过elder_id字段关联elder表，体现了家属与老人的对应关系，可用于支撑小程序端绑定老人、查看健康数据等功能的表结构设计，清晰呈现了各表间的关联方式与字段设置。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/QQu7bJ5QUoxGrLxAIHzco05ZnVg.png)

### 接口文档

<cite doc-id="D1dLwl8EOiuXZSk2wlOcYfxznvg" file-type="wiki" title="接口文档" type="doc"></cite>



## 后台管理系统-报警数据

### 需求说明

1. 列表展示

![图片展示的是中州养老后台管理系统中报警数据的列表页面。左侧为功能导航栏，当前选中“报警数据”。右侧上方有筛选条件区域，可按设备名称、创建时间等筛选。下方列表显示序号、设备名称、监测产品、报警数据类型、接入设备、数据级、操作等信息，其中序号1对应设备名称为watch01，监测产品是智能定位腕表，报警数据类型为老人异常数据，接入设备是刘临，数据级为94，操作栏有“处理”按钮，该按钮被红色框突出显示。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/CY85ba7twoSTCXxOCOLc1rEVnPh.png)

- 注意：只有负责该老人的护理员才能看到报警数据



1. 处理数据

![图片展示的是后台管理系统中报警数据处理界面。左侧为功能导航栏，右侧显示老人信息及报警数据列表。右侧列表中，编号为202的老人报警数据被选中，点击“处理”按钮后弹出“填写处理结果”窗口，窗口中有“处理时间”和“处理结果”输入框，当前显示处理时间为2024 - 09 - 27，处理结果为“已处理报警”。该图片与文档中后台管理系统报警数据需求说明相关，直观呈现了处理数据后的效果。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/VdZdb80q4o6T8DxMdDgcVbsNnmb.png)

处理后的效果：

![这张图片展示的是后台管理系统报警数据的列表界面，界面设有设备名称、创建时间的搜索筛选栏，可点击搜索或重置。表格包含报警时间、报警规则、处理结果、处理人、处理时间、状态及操作列，其中报警时间为23:08:44，报警规则是心率大于等于80.0且持续1周期就报警，处理结果为已处理报警，处理人是admin，处理时间为2024-09-26，状态为已处理，操作栏对应处理选项，且该数据仅负责对应老人的护理员可查看。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/IQ2bblB7Eoou2SxWbiQcB0MKnbg.png)

- 如果老人被多个护理员负责，那么多个护理员都可以看到这条数据
- 当其中一个护理员处理数据之后，其他负责的护理员也会展示**已处理**

### 接口文档

<cite doc-id="D1dLwl8EOiuXZSk2wlOcYfxznvg" file-type="wiki" title="接口文档" type="doc"></cite>

## 后台管理系统-护理任务

### 需求说明

护理任务的来源是服务计划内的任务，服务计划内的任务是根据老人所属服务等级的服务计划，每日自动生成的任务；护理人员可在任务安排页面查看指定日期，待执行/已执行的任务，若有特殊情况可取消任务，执行完任务后，可上传执行图片并填写执行记录；



1. 护理任务查询

![图片展示的是后台管理系统中护理任务查询页面。页面左侧有导航栏，显示“任务安排”等选项。右侧上方有搜索栏，可输入老人姓名、护理员姓名、护理组等条件进行搜索。下方列表显示了护理任务信息，包括序号、老人人数、编号、护理项目名称、护理项目名称、护理项目时间、操作等，如序号1对应老人人数1、编号101 - 2等。列表底部有分页显示，当前显示第1页，共185条记录。该图与文档中护理任务查询需求说明相关，直观呈现了查询页面及任务信息展示情况。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/FgKkbrBcvoEELaxiWnrcZL2en6g.png)

- 在这个页面中，条件众多，并且在列表的返回值中，需要重点分析所要展示的字段（结合接口文档）
- 对于任务，可以**取消、查看、执行、改期**



1. 取消

![这张图是后台管理系统护理任务模块的任务安排页面，展示了取消护理任务的操作界面。页面右侧操作列用红色虚线框标注出“取消”按钮，右侧有红色箭头指向弹出的“填写取消原因”窗口，窗口显示已填入的取消原因为“老人不舒服”，下方设有“确定”“取消”两个操作按钮。页面主体呈现了多条护理任务列表，包含老人姓名、排班、护理内容、护理时长等信息，任务状态分为待执行、已执行等类型，符合护理任务查询与取消功能的操作场景。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/NMnKbUa8Kope0VxPsOgcxHcRn8c.png)

1. 执行

![图片展示的是护理任务执行记录新增界面。界面上方显示执行时间2024 - 09 - 28 00:06:26，下方有执行图片区域，图片为老人与护理员互动场景，下方提示上传图片大小不超过5MB，格式为png/jpeg/jpegx。执行记录处输入“正常执行”。界面右下角有“确定”和“取消”按钮。该图片与文档中护理任务执行部分内容相关，直观呈现了执行记录新增时的填写界面。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/IEXObAXxooKvK2xihWFc3Bd7nFd.png)

1. 改期

![图片展示的是养老院护理任务安排页面中“改期”操作的弹窗界面。弹窗标题为“修改执行时间”，显示期望服务时间为2024年9月27日8:00:00。下方日期选择框中，当前选中日期为2024年9月27日，时间为8:00:00。弹窗右上角有“确定”和“取消”按钮。页面右侧列表中，有多个护理任务，右侧操作栏有“改期”按钮，部分任务右侧有红色框突出显示的“改期”按钮。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/RR9Rb1VaDoCyQUxK10fccyXAn2e.png)

1. 查看

查看了分了多种情况，根据不同的状态展示的内容有差别

- 待执行的任务

![这是一份待执行的护理任务详情页面，上方为基本信息区域，包含老人姓名关羽、性别男、床位号101-2，以及护理等级为5号护理等级、年龄73岁、护理员姓名为小青和小白的相关信息；下方为护理项目区域，显示护理项目名称为助餐，创建时间为2024-09-27 23:08:17，期望服务时间为2024-09-27 08:00:00，当前执行状态标注为待执行，还有备注信息栏未填写内容。该图片对应文档中待执行任务的展示内容，呈现了待执行护理任务的具体细节。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/HbZLbC6K3o7OLOxjVoDcN6jEnyb.png)

- 执行后的任务

![这是后台管理系统护理任务的执行后任务页面展示内容，分为基本信息、护理项目、执行记录三个部分。基本信息部分展示了老人姓名为张飞，性别男，床位号303-1，对应护理等级为5号护理等级，年龄72岁，护理员姓名为小白；护理项目部分显示护理项目名称为洗头，创建时间是2024-09-27 23:08:17，执行状态为已执行，期望服务时间为2024-09-27 08:00:00；执行记录部分显示执行人为若依，执行时间为2024-09-27 23:25:13，执行记录为222，还附带了一张执行时的图片。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/N5Ilbxb58oWGzDxsGUYchlPunof.png)

- 取消的任务

![图片展示的是护理任务查看页面。基本信息部分显示老人姓名为张飞，性别男，床位号303 - 1，护理等级5号护理等级，年龄72岁，护理员姓名小白。护理项目中，项目名称为助餐，创建时间2024 - 09 - 27 23:08:17，执行状态已取消，期望服务时间2024 - 09 - 27 08:00:00。取消记录显示取消人若依，取消时间2024 - 09 - 27 23:25:23，取消原因地方地方。该图与文档中护理任务查看需求说明相关，呈现了查看任务时的界面及信息展示情况。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/DW3mb589FoAuFYxJKk7cLKEtnNf.png)

### 表结构

护理任务表结构

![这张图片是护理任务表的结构说明，表名为nursing_task，展示了该表包含的所有字段信息。其中标注了各字段的名称与对应说明，包括id、床位编号、取消原因、创建人id、创建时间、老人id、老人姓名、预计服务时间、执行记录、护理员id、项目id、护理项目名称、实际服务时间、备注、状态（对应数值1为待执行、2为已执行、3为已关闭）、执行图片、更新人id、更新时间，同时特别标注了护理任务表的核心字段nursing_id为字符串类型，可存储多个护理员ID，内容与文档中护理任务表结构的说明相匹配。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/BkOIbtwW5oU83Ox7TTHcuSCHnTb.png)

注意：其中字段nursing_id类型为字符串，可以存多个护理员ID,比如：`103,104`

护理任务建表语句（提供的全量SQL脚本中已包含，无需再次执行）：

```SQL
DROP TABLE IF EXISTS `nursing_task`;

CREATE TABLE `nursing_task` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT 'id',
  `nursing_id` varchar(50) DEFAULT NULL COMMENT '护理员id',
  `project_id` int NOT NULL COMMENT '项目id',
  `project_name` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '护理项目名称',
  `elder_id` bigint DEFAULT NULL COMMENT '老人id',
  `elder_name` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '老人姓名',
  `bed_number` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '床位编号',
  `estimated_server_time` datetime DEFAULT NULL COMMENT '预计服务时间',
  `real_server_time` datetime DEFAULT NULL COMMENT '实际服务时间',
  `mark` varchar(2000) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '执行记录',
  `cancel_reason` varchar(2000) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '取消原因',
  `status` int DEFAULT NULL COMMENT '状态  1待执行 2已执行 3已关闭 ',
  `task_image` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '执行图片',
  `task_type` tinyint unsigned NOT NULL COMMENT '任务类型 1护理计划外 2护理计划内',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `create_by` bigint DEFAULT NULL COMMENT '创建人id',
  `update_by` bigint DEFAULT NULL COMMENT '更新人id',
  `remark` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '备注',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=37 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci ROW_FORMAT=DYNAMIC COMMENT='护理任务表';
```

### 接口文档

<cite doc-id="D1dLwl8EOiuXZSk2wlOcYfxznvg" file-type="wiki" title="接口文档" type="doc"></cite>



### 护理任务生成

在今天提供的代码中，已经包含了自动生成护理任务的逻辑和代码，大家可以执行系统中的定时任务

**护理任务定时生成，**就可以自动生成当前养老院中已入住老人的护理任务。如下图：

![图片展示了后台管理系统中护理任务的列表页面。页面左侧有导航栏，右侧上方有搜索栏及筛选条件。列表中显示了任务编号、任务名称、任务组名、cron执行格式、状态、操作等信息。其中，任务编号107“护理任务定时生成”被红色框线突出显示，其cron执行格式为“0 0 * * *”，状态为“已禁用”，右侧有“>”和“>”图标。该图片与文档中介绍护理任务定时生成的内容相关，直观呈现了系统中护理任务的定时生成设置情况。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-wu-zhang-fen-zu-shi-zhan/H72TbbL8Uo8cfTxEDK5cOSX2neh.png)

这个定时任务是每月的1号零点执行，现在的cron表达式都是为了测试使用，也可以点击立即执行按钮来生成任务。

注意：每次都点击执行，都会生成当前养老院已入住老人的护理任务，最好不要频繁点击，或者清空表之后再点击



实战要求：

- 找到当前定时任务对应的代码，并阅读代码，画出详细的流程图（飞书文档）



## 表关系

<whiteboard token="MZSlwA5UShnnFjbZlcVcxVminuc"></whiteboard>



## 苍穹外卖任务

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第五章-分组实战 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/FQcxw29hViQPsgk2ca4cNn3dnkg)
