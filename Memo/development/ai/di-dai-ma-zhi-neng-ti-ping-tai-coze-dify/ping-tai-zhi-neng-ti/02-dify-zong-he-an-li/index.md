---
type: memo
date: 2026-11-06
aliases:
  - "02-Dify综合案例"
---

# 02-Dify综合案例

## 项目介绍

### 项目背景

随着社会老龄化趋势加深，智慧养老已成为提升护理质量的重要方向。本项目通过引入 Dify 与 RAGFlow 的 AI 能力，对现有养老护理系统进行智能化升级，旨在帮助养老院员工更高效地处理日常事务，减轻工作负担。

系统主要实现以下目标：

- 整合知识，快速检索：将分散的护理规范、设备指南等资料构建为统一知识库，便于员工随时查找，提升信息获取效率。
- 智能问答，精准支持：AI 助手可即时解答护理相关的各类问题，如用药提醒、应急操作等，降低沟通成本。
- 流程自动化，减少人工：借助 Dify 工作流引擎，自动处理档案更新、服务审批等重复性工作，优化人力分配。

通过引入 AI 技术，我们希望帮助养老院员工更轻松、精准地完成照护任务，提升整体工作效率与服务品质。

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/Lvwxbcx6moRD3wx2PV6cX1OVnwe.png)



### 技术栈选型

本项目采用分层解耦的技术架构设计，确保系统具有良好的可扩展性和维护性。以下是经过充分调研和验证的技术组件选型：

#### 核心AI组件

- Dify (1.9.0)：作为AI应用编排框架，提供可视化工作流设计、多模型支持和完善的API管理体系。选择原因：开源免费、模块化设计便于二次开发、支持丰富的外部拓展工具和任务流编排。
- RAGFlow (v0.20.5 slim)：作为知识库引擎，负责复杂文档的解析和处理。选择原因：拥有强大的文件精细解析能力（DeepDoc技术），在处理PDF、扫描件、表格等复杂文档方面表现出色，支持OCR和多种文档切分模板。
- Elasticsearch(Dify自带)：作为向量数据库，用于存储和检索文档嵌入向量。选择原因：高性能相似度搜索，专门优化用于AI检索场景，完全兼容Dify和RAGFlow的接口标准。

#### 后端技术栈

- Java Spring Boot (2.7+)：作为核心业务后端，提供RESTful API和系统集成点。选择原因：成熟的企业级开发框架，丰富的生态系统，强类型语言保证系统稳定性。
- MySQL 8：作为业务数据库，存储结构化数据。选择原因：关系型数据库成熟稳定，支持事务处理，适合养老系统中的关键数据存储。
- Redis 6.0+：作为缓存和会话管理组件。选择原因：高性能内存数据存储，显著提升系统响应速度，支持分布式会话共享。

#### 前端技术栈

- Vue 3：作为前端主框架，构建用户界面。选择原因：渐进式框架，组件化开发模式，学习曲线平缓，社区活跃。
- Element Plus：作为UI组件库，提供一致的用户体验。选择原因：丰富的前端组件，高质量实现，专门针对企业级应用设计。
- ECharts 5.0：作为数据可视化库，展示健康数据统计。选择原因：强大的图表功能，灵活的配置选项，兼容多种数据格式。

#### 基础设施

- Docker & Docker Compose：作为容器化部署方案。选择原因：环境一致性，简化依赖管理，快速部署和扩展。
- Nginx：作为反向代理和负载均衡。选择原因：高性能HTTP服务器，稳定可靠，易于配置。
- 阿里百炼模型平台(Qwen)：作为大语言模型提供商。选择原因：提供高质量的文本生成和理解能力，支持多种规模模型选择。

### 系统架构设计

#### 架构图

本系统采用前后端分离的微服务架构，通过API网关集成各组件，既能保证系统稳定性，又具备良好的扩展性。整体架构分为数据层、AI能力层、业务层和表现层，具体架构设计如下图所示：

<whiteboard token="PCeIwtv5vhLGIpb3UOTcuhdwngQ"></whiteboard>

#### Dify与RAGFlow集成设计

Dify与RAGFlow采用 "优势互补" 的混合架构模式，充分发挥各自特长：

<whiteboard token="RctgwzZuJhvPyNbodppcPgFLnMd"></whiteboard>



RAGFlow扮演文档处理专家角色，负责处理各种格式的养老护理文档，包括：

- 医疗设备操作手册（PDF格式）
- 健康档案和护理记录（结构化表格）
- 药品说明书和护理规范（扫描文档）
- 应急处理流程和标准操作程序

Dify则充当智能大脑角色，负责：

- 接收用户查询意图识别
- 工作流编排和决策制定
- 多模型协作管理
- 答案生成和格式化输出



## Dify接入外部知识库-RAGFlow

Dify中的RAG一直被诟病，它的知识库设置不够丰富和灵活，对于不同形式的文档上传，尤其是pdf扫描版，上传识别效果不好，知识库根本回答不了PDF内的内容。为了解决这这些Dify提供了外部知识库API，这样就可以连接到 Dify 之外的知识库并从中检索知识。接下来给大家重点介绍Dify连接RAGFlow外部知识库的内容。

### RAGFlow-简介

RAGFlow是一款基于 **深度文档理解** （deepdoc）构建的开源RAG引擎。**深度文档理解**是RAGFlow对文档解析的一个解决方案，它包含两个组成部分：**视觉处理**和**解析器**。

其中视觉处理是通过OCR，布局识别，表结构识别来完成图像，PDF，表格的识别的。

针对PDF、DOCX、EXCEL和PPT四种文档格式，都有相应的解析器。能够从各类复杂格式的非结构化数据中提取信息，文本切片过程可视化，还支持手动调整。支持丰富的文件类型，包括 Word 文档、PPT、excel 表格、txt 文件、图片、PDF、影印件、复印件、结构化数据、网页等。

还集成了各种嵌入模型，rerank模型，提供易用的 API，可以轻松集成到各类企业系统。

安装包镜像完整下载下来，体积非常大，环境要求如下：

- CPU >= 4 核
- 运行内存= 16 GB
- 硬盘 >= 50 GB
- Docker >= 24.0.0 & Docker Compose>= v2.26.1

官网地址如下：

<bookmark name="RAGFlow | RAGFlow" href="https://ragflow.io/"></bookmark>

### RagFlow-安装配置

参考：<cite doc-id="NSYywYqZPiOp6Qk19n3cDBtXnLe" file-type="wiki" title="RAGFlow安装" type="doc"></cite>



### RAGFlow-知识库

#### 创建知识库

<grid>
<column width-ratio="0.739165">
![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/SbbFbM3rqoqZCYxxVkMc9VXMnKg.png)
</column>
<column width-ratio="0.260835">
![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/SHd1bigxfoEu1pxH23Kcw8hznIh.png)
</column>
</grid>

点击保存

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/R01rbczuRoq7EpxIauFcF12gn6b.png)

#### 知识库-配置

配置中包含两部分，分别是通用和分片方法

**通用**

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/QRFybOb7UoriXzxM4SRc2wM0nPc.png)

**切片方法**

进入模型设置界面 选择好嵌入模型和解析方法

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/Ncbwb4l6HoCAhfxHZt9cynNcnxb.png)

**切片方法解析：**

1. **General**

   1. 支持的文件格式为DOCX、EXCEL、PPT、IMAGE、PDF、TXT、MD、JSON、EML、HTML。
   2. 此方法将简单的方法应用于块文件：系统将使用视觉检测模型将连续文本分割成多个片段。
2. **Q&A**

   1. 支持 excel 和 csv/txt 文件格式。
   2. 如果文件是 excel 格式，则应由两个列组成 没有标题：一个提出问题，另一个用于答案， 答案列之前的问题列。多张纸是只要列正确结构，就可以接受。
   3. 如果文件是 csv/txt 格式 以 UTF-8 编码且用 TAB 作分开问题和答案的定界符。
3. **Resume**

   1. 支持的文件格式为DOCX、PDF、TXT。
   2. 简历有多种格式，就像一个人的个性一样，但我们经常必须将它们组织成结构化数据，以便于搜索。
   3. 我们不是将简历分块，而是将简历解析为结构化数据。作为HR，你可以扔掉所有的简历， 您只需与'RAGFlow'交谈即可列出所有符合资格的候选人。
4. **Manual**

   1. 仅支持PDF。
   2. 我们假设手册具有分层部分结构。 我们使用最低的部分标题作为对文档进行切片的枢轴。 因此，同一部分中的图和表不会被分割，并且块大小可能会很大。
5. **Table**

   1. 支持EXCEL和CSV/TXT格式文件。
6. **Paper**

   1. 仅支持PDF文件。
   2. 如果我们的模型运行良好，论文将按其部分进行切片，例如摘要、1.1、1.2等。
   3. 这样做的好处是LLM可以更好的概括论文中相关章节的内容， 产生更全面的答案，帮助读者更好地理解论文。
   4. 缺点是它增加了 LLM 对话的背景并增加了计算成本， 所以在对话过程中，你可以考虑减少‘topN’的设置。
7. **Book**

   1. 支持的文件格式为DOCX、PDF、TXT。
   2. 由于一本书很长，并不是所有部分都有用，如果是 PDF， 请为每本书设置页面范围，以消除负面影响并节省分析计算时间。
8. **Laws**

   1. 支持的文件格式为DOCX、PDF、TXT。
   2. 法律文件有非常严格的书写格式。 我们使用文本特征来检测分割点。
   3. chunk的粒度与'ARTICLE'一致，所有上层文本都会包含在chunk中。



#### 解析文件生成知识库

可以单个文件解析，也可以批量解析，大家可以根据自己的需要选择使用，解析完成后可进行使用

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/MUz7bLeeAoSwqcx3ovycJP9mndb.png)

### RAGFlow-聊天

点击聊天菜单

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/QDEMbrGLqoRJS2xZTSlcgBG3nWb.png)

创建聊天

<grid>
<column width-ratio="0.536829">
![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/T1Gib6LgWo79Fux74zGcNW32n0A.png)
</column>
<column width-ratio="0.463171">
![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/O1cqbdOC2oBdQVxJAE2c7Zatnkg.png)
</column>
</grid>

**点击新建的聊天**

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/IU3kbCm91oWrNJx3mkDck47Snsi.png)

聊天设置中加入刚刚新创建的知识库

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/TiiCbtFHQobTfdxvy7RcOSrznxe.png)

以上设置完成后就可以进行聊天对话了。

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/PGRubYfJcoCXeuxxxXFc2lpBnSg.png)

### Dify接入RAGFlow

#### RAGFlow API设置

接下来我们在dify调用这个ragflow，需要设置一下ragflow的api key。

- 点击系统右上角的头像
- 点击API菜单

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/CneQbtXUKocz3Ux1m3ncfoBNnSh.png)

- API接口文档

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/UJeOb5h14oZkmTxMMaNcagYanAd.png)

#### Dify配置外部知识库

第一步，dify工作流管理界面，点击上面知识库。点击链接外部知识库。

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/Csdbb9UBEoYXnDxXXPOcvcZ4nPe.png)

第二步，点击 `外部知识库API` , 先把外部知识库API配置好。

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/PsXtbzcQPohEhyxs8S7cqLFIn9d.png)

<callout emoji="⚠️">
API Endpoint 的值
  http://ip:8880/api/v1/dify
ip必须是以192.168开头的，不可以用127.0.0.1或者localhost
Name的值，随便写一个名字
API Key的值，来自于：http://localhost:8880/user-setting/api
</callout>

第三步，连接外部知识库

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/QSIRbPRcyoBRjFx6R0ocHTf7nNh.png)

> 关于外部知识库ID的值：  
> 打开RAGFlow平台，找到知识库，点击刚刚创建的知识库，url地址中的最后一段字符串就是对应的知识库ID
> 
> ![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/FQd9bF4kyo1y1lx9fWMce0sWnnb.png)

点击连接，完成ragflow和dify的连接。

到这dify和ragflow已经连接好了。

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/GwmxbSryFoLcCFxQUomcgzmlnwc.png)

#### Dify之AI Agent接入外部知识库

第一步，创建AI Agent应用

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/HYhBbWzyLoJjtRxRt8Ccm0BZnQg.png)

第二步，进入AI Agent聊天界面中，在上下文添加外部知识库。

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/XE1Zb4MdRoDyM8xxhqpcv4Qvnsg.png)

提示词编写

```Python
你是一位专业的大模型领域专家，请你结合在知识库中检索的内容以及你自己的大模型只是来回答用户的问题
```

第三步，调试与预览

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/CmHMbKXmCojKZUxF11fc8LQan1J.png)

到这在dify中调用ragflow中的知识库了。



## 养老项目对接Dify

### 需求分析

#### 需求1-内部知识问答

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/YrfJbNYPJozw62xQshMcrNuZngc.png)

可以回答养老院**内部文档**中的内容，比如：员工手册、操作规范、用药规范、护理手册等等

- 什么文档，什么格式
- 怎么拆分的？拆分策略   父子块
- 怎么向量的
- 怎么检索   混合模式(向量检索+全文检索)
- 图片怎么解析的



#### 需求2-查看养老院的数据(http或mcp)

- 查询预约信息

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/T6iDbMjSzoKk7HxVFpccENlNnib.png)

- 查询老人的基本信息

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/DwbwbUdMnon5B0xJ6h1cWqHbnXg.png)

- 查询老人的健康信息

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/H3CPbyIL6odk99xAdl2cpeZsnZU.png)

- 其他可查询信息

  - 动态查询可用床位
  - 实时查询老人的健康数据



#### 需求3-定制化护理任务执行表生成

护理员登录系统之后，可以通过ai助手，可以生成指定老人的个性化护理方案(每日)

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/VvEubLc6RoOEkQxKU0mcSEtJnSh.png)

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/AdsGbouw5osY2vxpLUycwcDonSb.png)

实现方案：

<whiteboard token="VUL9wp0nThSCcvbHQDMcavV0n8c"></whiteboard>

### Dify智能体构建

#### 创建RagFlow知识库

打开RAGFlow工作台，创建一个知识库，并且上传两个文档，如下图

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/VZmibKcTLocQKOxAlWJc6RdbnYb.png)

在切片配置中，使用默认即可，已经可以**深度**文档读取

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/LzNObRg6youngfxgv7icUFfGnfs.png)

参考之前的案例准备**服务器地址**和**API KEY**

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/YTBIb4ldOoq5QqxvVq0cR6YNnsc.png)

**知识库ID**需要提前记录

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/XbLqbyPV6oMUCvx3k0JcXS0rnqh.png)

> 点击去知识库，可以在浏览器中查看**知识库ID**



#### 创建Dify外部知识库

创建一个外部知识库，如下图：

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/IgZCbKWyioTMlCxwK0DcgC6Hnie.png)

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/GhMWb2uApoWmyfxCPU2c9TAnn3f.png)

设置知识库id

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/SuAHb51O2o2FihxMt8KcTYyJnnT.png)

> - Top K  从知识库中召回几个文档片段  
> - Score 文档相关度得分，最高为1分，0.6 算是相关度较高的分值，精准度更高一些



#### 创建Dify智能体

创建一个智能体，可以关联外部知识库

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/I7uVbJyTgoAOHFxnFmXcL3eOn7c.png)

创建成功之后，可以关联知识库，并设置提示词

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/Sz2dbr2otoIDToxCwGncqGN3nvc.png)

提示词：

```Bash
**角色设定：**  
您是一个专业的养老院护理知识助手，您的任务是帮助用户快速找到与养老院护理相关的问题答案。

**附加说明：**  
- 请始终以友好、专业的态度响应用户。  
- 鼓励用户提供更多细节，以便更精确分类。  
- 拒绝回答跟养老院无关的问题
```



### 智能体API文档

#### API KEYS生成

智能体调试成功之后，可以设置对外访问的API

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/ECeGbRa2YouRoaxvyK6cbGVSnfd.png)

> - API 秘钥：上图中可以给当前智能体生成一个秘钥，方便通过接口的方式进行调用
> - 基础URL：访问接口的基础url
> - 鉴权：鉴权的方式，需要设置API 秘钥



#### 接口说明

在智能体中提供了很多对外访问的接口，常见的接口如下：

- 对话消息  （流式 | 非流式）
- 获取会话历史列表
- 获取会话列表
- 删除会话
- 停止响应



### 养老后台对接智能体API

#### 环境准备

- 克隆代码：https://gitee.com/itheim/dify-zzyl.git
- 环境配置：

  ![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/KjHIbaR8RoFvU8xYAheccLFZnlc.png)

  - 执行sql目录中的sql文件
  - 修改zzyl-admin中的application-dev.yml文件，改为自己的配置内容
  
    - Mysql
    - Redis
    - OSS
    - deepseek的api-key
  - 使用VS Code 或 Cursor来打开**zzyl-ui**目录
  
    - 安装依赖：`npm install `
    - 启动项目：`npm run dev`



#### 对接Dify

目前提供好的养老代码中已经对接好了Dify的接口，共有4个接口：

- 对话聊天   web flux实现，需要流式对话
- 历史列表   http请求
- 历史对话   http请求
- 删除对话   http请求



代码修改：

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/CbO4btzK9ocYMixmradcGd35nYg.png)

> 上述标识中，必须改为自己在Dify创建Agent中的密钥才行，IP也可以根据自己的实际情况来调整



核心对接代码位置如下图，代码超90%由Deepseek编写

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/ZIx9bJZc4o7WQDxBwiecFrrDnWc.png)



启动前后端之后的效果：

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/NKtQbA5HCoT00FxJEDbcBpXKnNf.png)

> 前端对话窗口的实现全部使用Cursor实现



### 查看养老院的数据

我们的需求是，想要通过问答助手的形式，让他回答关于养老院的一些数据信息，如下需求：

- 查询预约信息(以这个为例)
- 老人的基本信息(自行实现)
- 老人的健康信息(自行实现)

#### 实现思路



<whiteboard token="KKzTwR4e7hew1tbGUeGcTXcPnEd"></whiteboard>

deepseek生成校验apikey方案：https://chat.deepseek.com/share/nl7pcc82asmxri57m2



#### 提供对外的接口

查询预约列表信息(已提供)

```Java
@RestController
@RequestMapping("/dify/serve")
public class DifyServeController {

    @Autowired
    private IReservationService reservationService;

    @GetMapping("/getReservationByToday")
    public String getReservationByDay(String datetime){
        System.out.println(datetime);

        // 定义日期时间格式
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        // 字符串转 LocalDateTime
        LocalDate dateTime = LocalDate.parse(datetime, formatter);

        return reservationService.getReservationByDay(dateTime);

    }

}
```

所有dify访问的接口，由于需要使用api-key的方式进行校验，在Spring Security中放行

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/O68mbLyZXozyCVx5c29cwZONnof.png)



#### 工作流http的能力

在dify中创建一个工作流：

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/ONb5bMuRVofSJqxrD8VckLvYnid.png)

- 开始节点

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/OO8mbSirvo7ziAxupMLcM8Mbnvg.png)

- HTTP请求节点

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/BbAsb65DRoC46oxCHKpcTmJJnIf.png)

- LLM节点

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/Mo28b57L4owz7Jxs5hTcjlWbnwi.png)

提示词：

```JSON
根据{{#context#}}的信息展示预约的列表信息，展示的格式，参考下面的例子，可以展示多条
- 老人姓名: 老张,  探访时间：10:30,  探访人: 小张
- 老人姓名: 老李,  探访时间：11:30,  探访人: 小李

没有信息，则回复今天没有查询到老人的预约信息
```



- 结束节点

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/CdV2bjZd6o9DFdxB7T3cltNGnIe.png)



测试之后需要把工作流发布为工具才能让agent来调用

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/Tu4ebrfJ7olpZGxQF5fcQJ3gndv.png)

需要给工作流起一个名字，方便后期按照名称调用

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/NkPJbtixKoBjKTxLJeocDnoUnLg.png)



#### 工作流集成到agent中

回到agent中，在工具添加中可以找到已经发布的工作流，同时集成获取时间的工具

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/NnyHbEixQo6I5Hxz4OecGJgCnzb.png)

修改提示词如下：

```Bash
你是一个养老院的问答助手，主要通过知识库的内容回答用户的问题
当用户需要查询养老院的入住老人的预约探访情况的时候，需要调用工作流：getReservationTool，调用该工具需要输入时间，格式为：2022-12-12，获取时间的工具为：current_time
```

修改配置完成之后，agent必须**重新发布**才能生效



可以在养老院后台对话中测试

第一次对话：

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/Lcq9bDUtiojk8KxWdrecZ5r1nyc.png)

检查数据库中是否存在今天的预约信息，如果没有，需要修改为今天的日期(也可以通过小程序添加预约信息)

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/TzxqbOmyOopr9QxU2Ktc8cndnbe.png)



由于会有历史对话记忆，需要**重新开一个对话窗口**进行对话：

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/MBnPbh9XbocclrxfDlYc1ONwneb.png)

---
> 来源：飞书技术知识库（脱敏版） · 06 AI 应用开发 / 低代码智能体平台（Coze / Dify） / 平台智能体 / 02-Dify综合案例 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/KKvjwYYRJilsqBkA2XdcJggqn3b)
