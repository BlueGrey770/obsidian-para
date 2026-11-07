---
type: memo
date: 2026-11-06
aliases:
  - "拓展内容-MCP"
---
# 拓展内容-MCP

## 一、QA

1. openAI  的api key无法传入403，无法识别APIkey

> 答：通常有两种可能性：
> 
> - 环境变量配置错误导致，比如变量名称不一致
> - 平台的API KEY错误



1. 为什么我设的system没用啊，用`chatclient.defaultSystem`设置也没用

> 通常推理类大模型都是不支持System设定的，比如DeepSeek-R1模型、QwQ模型，DeepSeek官方有说明，System设定对它效果不佳，不建议设置



1. 如何把该项目由m6升级为m7？需要修改哪些地方？

>  目前SpringAI官方并未把m7版本上传至maven中央仓库，所以要想使用需要从spring仓库下载：
> 
> ```XML
> <repositories>
>   <repository>
>     <id>spring-snapshots</id>
>     <name>Spring Snapshots</name>
>     <url>https://repo.spring.io/snapshot</url>
>     <releases>
>       <enabled>false</enabled>
>     </releases>
>   </repository>
>   <repository>
>     <name>Central Portal Snapshots</name>
>     <id>central-portal-snapshots</id>
>     <url>https://central.sonatype.com/repository/maven-snapshots/</url>
>     <releases>
>       <enabled>false</enabled>
>     </releases>
>     <snapshots>
>       <enabled>true</enabled>
>     </snapshots>
>   </repository>
> </repositories>
> ```



1. InMemoryChatMemory这个方式会不会直接内存爆了？

> InMemoryChatMemory只适合教学和测试使用，生产环境建议使用spring官方提供的`CassandraChatMemory`或者自定义ChatMemory实现。



1. 企业中会话记忆会存在哪里?Redis还是向量数据库，存的特别多怎么持久化?删除策略怎么搞?能给个实际可行的方案吗？

> 在有用户参与的业务中，会话记忆是需要持久化保存的，而且通常还会与用户关联，数据量也会比较大。通常建议存储在支持海量数据存储的库中。
> 
> 例如：
> 
> - 关系型数据库，可以选择MySQL
> - 非关系型数据库，可以选Cassandra（Spring官方有实现）、MongoDB等
> - 向量数据库，各类向量库都可以，读取会话记忆时不再是全量提取，而是读取跟用户本次对话相关的记忆，从而实现理论上无限的记忆上下文
> 
> 删除策略要根据具体业务需求来看，例如大多数的像通义千问这样的对话类产品，是不删除会话记忆的。也有一些业务只保存最近1年的会话记忆，可以采用定时任务删除过期的数据。
> 
> 在最新的SpringAI里程碑版本中，已经新增了很多不同的会话记忆实现：
> 
> ![图片介绍了Chat Memory接口，其代表聊天会话历史的存储，提供添加消息、检索消息及清空会话历史的方法。目前有四种实现：InMemoryChatMemory内存存储，CassandraChatMemory在Cassandra中持久化并设置过期时间，Neo4jChatMemory在Neo4j中持久化但不设置过期时间，JdbcChatMemory在JDBC中持久化。该图片与上下文紧密相关，上下文提到在最新SpringAI里程碑版本中新增了多种会话记忆实现，此图是对其中一种实现形式的说明。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/IavibMRLSobvnwxTq1jciDAInEe.png)



1. 如何使用mongdb存储会话记忆，我尝试会报错，显示springai下面有个usermessage没无参构造

> Message中有一些类比较特殊，是没有空参构造的，因此你从MongoDB中读取消息，反序列化为Java 对象时就会因为找不到空参构造而报错。
> 
> 解决方案就是手动处理UserMessage、AssistantMessage的序列化和反序列化，然后再与MongoDB交互。







1. 聊天记忆功能是否每次都是把所有聊天记录都一起传到大模型，这样对性能是不是影响很大

> 并不是所有的聊天记忆都会发送到大模型，通常是可以自己选择聊天记忆大小的，SpringAI中默认的最多允许传输最近的100次Message，可以通过参数调整：
> 
> ![图片展示的是一个代码片段，用于调用chatClient的prompt方法。关键部分是用红框突出显示的代码，调用chatClient.prompt().param(CHAT_MEMORY_RETRIEVE_SIZE_KEY, 100)。这表明在调用chatClient.prompt()方法时，会将CHAT_MEMORY_RETRIEVE_SIZE_KEY参数设置为100。该图片与上下文讨论的聊天记忆功能相关，上下文提到聊天记忆默认最多允许传输最近100次Message，此代码片段体现了这一默认设置。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/AtKNbyfKOogoUrx90zfcMsX0nTd.png)





1. 多模态, 上传图片描述后,  如果再问关于图片内容, ai表示不知道

> 通常来说多模态模型在每一轮对话中只能对当前的多模态数据做处理，多轮会话时只能根据上下文的聊天记录来回答。





1. functioncaling的原理是啥,  为啥会精准找到接口, 是匹配tool注解的信息吗

> FunctionCalling的关键就是tool注解中的描述。
> 
> spring会根据tool注解信息，结合反射得到的其它函数信息，为每一个tool生成一个严格的json schema定义（在日志中能看到）。
> 
> 在对话时，spring会将这些json 格式的函数定义发送给大模型，大模型可以根据与用户的对话判断是否需要调用这些工具，具体是哪个工具，而后返回一个json结果，其中包含要调用的tool name、args等
> 
> 然后spring就可以根据这些信息调用对应的tool

```JSON
tools: [
{
  tool_name: "queryCourse",
  description: "查询课程",
  args: [{xxx:""}],
  result: {}
}
]


tool_calls: [
{
 tool_name: "queryCourse",
 args: [xx]
}
]
```





1. functioncalling调用工具时，如何获得当前聊天用户的id等信息，比如叫ai帮我发一个帖子，修改自己头像，好像调用工具的时候的线程和原先不一样，而且不会走拦截器，获取不到token

Functioncalling 怎么获取 security中的用户

<whiteboard token="GD1UwEXLBhpSVAbOkQCcJZb3nDd"></whiteboard>

> SpringAI支持在tool调用时传递上下文参数：
> 
> ![这张图展示了Spring AI借助ToolContext在工具调用中传递上下文信息的流程：图中首先将工具定义和上下文信息填入Chat Request，由Spring AI的Dispatch Tool Call Requests模块进行分发，随后AI模型会完成相关处理。流程分为多个环节：Dispatch Tool Call Requests会分别对接Tool和AI Model，最终生成Chat Response完成交互；该图对应文档中提到的Spring AI支持在tool调用时传递上下文参数，可通过ToolContext获取上下文中信息的内容。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/SZ7bbclVToAeAtx15s1c4Vign4g.png)
> 
> 借助于ToolContext可以获取上下文中的信息。
> 
> 定义Tool执行逻辑的时候从ToolContext中获取userId：
> 
> ```Java
> class CustomerTools {
> 
>     @Tool(description = "Retrieve customer information")
>     Customer getCustomerInfo(Long id, ToolContext toolContext) {
>         // 调用其它微服务前，先把userID存入请求头
>         return customerRepository.findById(id, toolContext.get("userId"));
>     }
> 
> }
> ```
> 
> 每次请求时传递user信息到ToolContext:
> 
> ```Java
> ChatModel chatModel = ...
> 
> String response = ChatClient.create(chatModel)
>         .prompt("Tell me more about the customer with ID 42")
>         .tools(new CustomerTools())
>         .toolContext(Map.of("userId", "acme"))
>         .call()
>         .content();
> 
> System.out.println(response);
> ```



1. functioncalling怎么将千问模型换成调用deepseek模型，调用deepseekapi一直报错，是不兼容吗

> 在我们的课程发布时，DeepSeek-R1模型并不支持function calling
> 
> 目前，虽然DeepSeek官方已经升级了DeepSeek-R1模型，支持了FunctionCalling，但是大多数其它平台部署的DeepSeek-R1依然不支持。
> 
> 所以如果想用DeepSeek的FunctionCalling功能，有两种办法：
> 
> - 切换到官方的DeepSeek服务
> - 切换到其它DeepSeek版本，比如DeepSeek-v3



1. 智能客服那里配置了chatclient的defaultTools无法创建Bean，但是加了第四张图的依赖问题就解决了，课程内老师并没有说需要这个依赖

> 没有碰到你说的问题，猜测可能是依赖版本的问题



1. 智能客服我添加了修改的接口, 但是有时候会调用这个接口,有时候又不会调用,  但是ai告诉我都是调用了,  告诉我已经更新, 但是数据库实际是没有的

> 需要查看日志来分析，有时候大模型虽然回复中说会调用工具，但是返回的格式并不是tool calling格式，导致SpringAI无法识别，也不会调用
> 
> 还有，添加新的接口，那么提示词也需要重新编写，如果提示词不合理，可能也会导致调用工具失败



1. RAG搜索自己的pdf搜到不出来,  一直提示上下文没有信息

> 检查下你的文档是否真的写入向量库了
> 
> 检索向量库时，是否添加了额外的过滤条件，导致过滤掉了结果
> 
> 可以调低文档匹配阈值试试：
> 
> ```Java
> @Bean
> public ChatClient pdfChatClient(OpenAiChatModel model, ChatMemory chatMemory, VectorStore vectorStore) {
>     return ChatClient
>             .builder(model)
>             .defaultSystem("请根据上下文回答问题，遇到上下文没有的问题，不要随意编造。")
>             .defaultAdvisors(
>                     new SimpleLoggerAdvisor(),
>                     new MessageChatMemoryAdvisor(chatMemory),
>                     new QuestionAnswerAdvisor(
>                             vectorStore,
>                             SearchRequest.builder()
>                                     .similarityThreshold(0.5) // 文档相似度阈值
>                                     .topK(2)
>                                     .build()
>                     )
>             )
>             .build();
> }
> ```



1. 用redis vector store作为向量库，用过滤表达式，过滤不出数据，redis中已有数据 

> 原因：
> 
> Redis向量库中的数据过滤对中文和特殊字符会做切分处理，例如"中二知识笔记.pdf"中的`.`就会被特殊处理，在检索时需要加上转义字符：
> 
> ```Bash
> FT.SEARCH spring-ai-index "@file_name:{中二知识笔记\\.pdf}" RETURN 3 content page_number file_name
> ```
> 
> 解决方案：
> 
> 不以文件名作为过滤项，而是添加其它的不包含中文的过滤项。例如，以chatId作为过滤，也可以达成相同目的。
> 
> 要注意的时，SpringAI在读取时默认的Metadata只有file_name和page_number，如果要以chatID过滤，需要自己在生成文档时添加chatID，像这样：
> 
> ```Java
> private void writeToVectorStore(Resource resource, String chatId) {
>     // 1.创建PDF的读取器
>     PagePdfDocumentReader reader = new PagePdfDocumentReader(
>             resource, // 文件源
>             PdfDocumentReaderConfig.builder()
>                     .withPageExtractedTextFormatter(ExtractedTextFormatter.defaults())
>                     .withPagesPerDocument(1) // 每1页PDF作为一个Document
>                     .build()
>     );
>     // 2.读取PDF文档，拆分为Document
>     List<Document> documents = reader.read();
>     documents.forEach(d -> d.getMetadata().put("chat_id", chatId));
>     // 3.写入向量库
>     vectorStore.add(documents);
> }
> ```



1. 版本问题

问题：

- 最新的springAI依赖版本是多少      
- springAI目前用哪个版本好些呀，我没找到m6和m7版本，我看官网只有1.0.0-SNAPSHOT和1.0.0-RC1
- 从M7升级到RC1好多爆红，哪些地方要改，怎么改

  <grid>
  <column width-ratio="0.500000">
  ![图片展示了SpringAI中两个ChatClient的配置代码。上方是gameChatClient配置，使用OpenAiChatModel模型，设置默认系统为GAME_SYSTEM_PROMPT，添加SimpleLoggerAdvisor和MessageChatMemoryAdvisor（redisChatMemory）。下方是serviceChatClient配置，使用AlibabaOpenAiChatModel模型，设置默认系统为CUSTOMER_SERVICE_SYSTEM，添加MessageChatMemoryAdvisor（redisChatMemory）和SimpleLoggerAdvisor，还添加了defaultTools参数。该图片与文档中介绍SpringAI版本及升级时需注意改动的内容相关，直观呈现了配置代码。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/TEvVbUjTsoUvJ5xlw1Kc7Kggn2e.png)
  </column>
  <column width-ratio="0.500000">
  ![这张图片展示的是一段Java代码，内容是定义一个名为pdfChatClient的Bean，用于配置ChatClient实例。代码以OpenAiChatModel等作为参数传入ChatClient的构造，设置了系统提示文本。重点配置了三个Advisor，分别对应聊天记忆、日志记录以及问答建议功能，其中向量检索相关的配置被特别标注：向量检索请求参数构建时设置了相似度阈值为0.5，返回的文档片段数量为2，该部分用红色框线重点标识，是代码中用于向量检索相关参数配置的核心内容。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/E5bmbF1spofCSyxUm6KcegXYnOb.png)
  </column>
  </grid>
- 介绍一下RC1和课程中的版本的区别吧，还有升级版本的时候需要注意改动什么



> SpringAI目前处于快速迭代期，还**没有发布正式版本。**目前发布的都是里程碑版，更新迭代的速度非常快，而且每个版本之间差别比较大，甚至依赖都不一样。
> 
> 因此，不建议大家在学习期间升级版本，使用课堂上讲的m6版本就行，等到正式版发布，比较稳定以后，我们会更新教程。
> 
> 大家在学习的时候不要关注API，而是关注底层的思想和设计，这样不管API怎么变，问题都不大。只需要查询官方文档，找出变化的点即可。









1. SyncMcpToolCallbackProvider这个类报无法注入，我想用它把McpSyncClient放到模型中去，我用的是1.0.0-RC1版本，是SpringAI源码目前还有问题还是我的问题，我看源码中他确实没有无参构造器只有有参构造器

> 在最新的SpringAI版本中，mcp的toolcallback.enabled功能默认是关闭的，需要手动开启才行：
> 
> ```YAML
> spring:
>   ai:
>     mcp:
>       client:
>         stdio:
>           servers-configuration: classpath:mcp-servers.json
>         toolcallback.enbled: true # 开启mcp转toolcallback
> ```





1. 有谁能解释一下这几个概念： coze dify 、langchain 、langgrah、rag 都是 干嘛的？？？网上说的有点抽象

> Coze、Dify :都是低代码智能体搭建平台
> 
> Coze更多是面向小白，快速构建智能应用；Dify更多是面向企业级用户，实现更灵活更复杂的智能体
> 
> Dify可以支持本地部署，数据安全性更好
> 
> langchain：用来开发大模型应用的Python库，类似Java中的jar包
> 
> langgrah：基于langchain进一步封装的Python库，基于有向无环图实现智能体工作流
> 
> rag：一种大模型开发技术方案，具体参考我们的课程









1. 以后讲师能把微调补充进课程吗

> 模型微调的内容需要一些前置知识，还有一定的Python基础，在咱们后面的课程中就有相关的内容



1. 老师可以讲一下spring ai结合mcp吗？例如调用高德地图mcp服务。

> MCP的内容，在咱们后面的课程中就有相关的内容



1. 智能聊天，需要老师讲一下连接自己搭建的deepseek进行分析，目前看见是连接的openai

> 入门篇就是对接的DeepSeek



1. 能否用Python出一个课程





1. 两个chatclient之间如何通信。比如第一个chatclient向第二个发布任务，第二个使用工具执行完需要告诉第一个执行结果，第一个获得结果后进行整理返回给用户

<blockquote><p>SpringAIAlibaba中有对应的功能：</p><readonly-block href="https://java2ai.com/docs/1.0.0-M6.1/tutorials/agent-agentic-patterns/?spm=5176.29160081.0.0.29a9aa5ctQuUI4" type="iframe"></readonly-block></blockquote>





1. 什么是智能体，跟MCP又有什么关系？和functioncalling有什么区别，他们的工作中有什么联系



1. MCP的stdio既可以调用本地服务又可以调用远程那，sse这种调用远程服务的还有什么优势吗

> 在MCP官方指南中，建议stdio用于本地服务，sse用于远程服务
> 
> - stdio，是基于本地进程通信，其server通常是一段NodeJS或Python程序
> - sse，是基于http协议的Server-Sent Events模式，需要建立网络长连接



| **特性** | **Stdio** | **SSE** | **Streamable HTTP** |
|-|-|-|-|
| 通信方向 | 双向（但仅限本地） | 单向（服务器到客户端） | 双向（适用于复杂交互） |
| 使用场景 | 本地进程间通信 | 实时数据推送，浏览器支持 | 跨服务、分布式系统、大规模并发支持 |
| 支持并发连接数 | 低 | 中等 | 高（适合大规模并发） |
| 适应性 | 局限于本地环境 | 支持浏览器，但单向通信 | 高灵活性，支持流式数据与请求批处理 |
| 实现难度 | 简单，适合本地调试 | 简单，但浏览器兼容性和长连接限制 | 复杂，需处理长连接和流管理 |
| 适合的业务类型 | 本地命令行工具，调试环境 | 实时推送，新闻、股票等实时更新 | 高并发、分布式系统，实时交互系统 |

表格内容转自掘金：https://juejin.cn/post/7494500661637611546



1. MCP使用stdio调用本地服务例如数据库时是怎样调用的，只需要配置好相应参数就可以直接调用数据库吗，那本地数据库的MCPServer是数据库安装时就搭建好的吗



> MysqlMcpServer是需要提前在本地部署的，并不是安装数据库时存在的。
> 
> 以mysql-mcp-server为例，调用mcp服务就是执行uv命令，然后传入参数即可。





## 二、智能体



### 什么是智能体

**智能体**，英文叫Agent，它指的是能够**自主执行任务的智能系统**。与传统软件只能按照程序员预先设定的流程执行步骤不同，AI Agent 可以在较大自主性下替用户完成复杂的工作流。



注意，智能体并不是简单的对话机器人。对话机器人只能与用户进行单轮或多轮对话，无法对复杂任务做拆解，也无法调用外部工具。



而智能体（AI Agent）则是利用大模型的逻辑分析能力在于用户对话过程中收集必要信息，然后分析任务实现的流程，接着自主的执行整个流程，在这个过程中可能会调用一些外部工具，比如访问浏览器，创建文件等等。最终完成整个任务工作流。当然在这个过程中，智能体也不是为所欲为的，而是要遵循提前设定好的指令。



简单来说，如果将大型语言模型比作Agent的大脑、各种外部工具比作Agent的手脚、预先设定的指令比作Agent的行为准则，那么AI Agent就是结合了大脑 + 手脚 + 行为准则，可以自主执行一系列操作的智能助手。



也就是说，通常一个智能体都有三部分组成：

- **模型**：也就是智能体的大脑，可以是各种具备FunctionCalling能力的通用模型
- **工具**：是模型与外界交互的桥梁，就是我们之前学习过的tool，或者叫function，可以分为三类：

  - 数据类工具（Data）：查询数据库、网页、文件等，获取用于执行任务所需的上下文
  - 行动类工具（Action）：用于执行具体的操作，比如发送邮件，更新数据库
  - 编排类工具：一种特殊工具，有可能是另一个Agent，实现多Agent互相协作
- **指令**：其实就是提示词，通常是System提示词



## 1. Spring AI MCP

### 1.1. 什么是MCP

MCP（Model Context Protocol ）叫作模型上下文协议，是一种标准化协议，它能够使 AI 大模型以结构化的方式与外部工具和资源进行交互。它支持多种传输机制，以在不同环境中提供灵活性。

![图片展示了有无MCP（多语言模型协议）时的交互情况。左侧“Without MCP”中，各图标（如PostgreSQL、Google Drive等）间有直接连线，无统一协议；右侧“With MCP”中，各图标间连线通过“MCP Protocol”节点，统一了交互方式。这与上下文介绍的MCP是增强AI大模型能力、更通用的协议相呼应，直观呈现了MCP在多语言模型交互中的统一作用。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/IpbYbdR6woNpdexF719cwRLknnf.png)

由上图，可以看出，在Java应用程序中，通过MCP Client可以与各种的MCP Server进行交互，增强了AI大模型的能力。

看上去，似乎和Tool Calling差不多，也是增强大模型的能力，不同的是，MCP是标准，更通用。

举两个场景来说明：

- 在SpringAI中编写的Tools，可以放到Python中使用吗？，显然是不可以的，要想让Python中使用，就得使用Python语言重写一次Tool，才能使用，这就是不通用性的体现。
- 在企业项目开发中，我们采用了Spring AI框架来构建智能体。这些智能体通常需要执行一系列通用功能，如天气查询、商品检索及下单等。基于以往的做法，每当开发新项目时，都需要重新编写实现这些功能的工具（Tool），这显然不利于代码复用。为解决这一问题，可以采用MCP标准进行服务器端开发，并将这些常用工具封装成服务接口。这样一来，其他的智能体只需通过调用该服务即可轻松访问所需功能，从而极大地提高了代码的复用性和开发效率。

### 1.2. 技术架构

SpringAI对MCP做了支持，简化了Java项目中的MCP开发。参考文档：

[https://docs.spring.io/spring-ai/reference/1.0/api/mcp/mcp-overview.html](https://docs.spring.io/spring-ai/reference/1.0/api/mcp/mcp-overview.html)

#### 1.2.1. MCP Client

![这张图展示了Spring AI中MCP的技术架构，核心位于上方的Java Gen AI Application，其下方连接了多个Java MCP Client实例，这些客户端支持MCP Protocol（stdio、sse），部分客户端对接了Web APIs及对应的远程MCP服务器（remote 1、remote N）。图中还包含三类MCP服务器，分别是基于Brave的MCP Server、对接本地文件夹的MCP Server、对接SQLite Server的MCP Server，这些服务器既支持Stdio传输，也可与外部互联网的搜索能力交互。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/BLPfbPVWLoU4QYxmztTcVHrOnId.jpg)

MCP Client 负责建立和管理与 MCP 服务器之间的连接，以及管理：

- 协议版本协商以确保与服务器的兼容性
- 功能协商以确定可用特性
- 消息传输和 JSON-RPC 通信
- 工具发现与执行
- 资源访问与管理
- 提示系统（Prompt ）交互
- 可选功能：

  - 根管理
  - 采样支持 
- 同步和异步操作 
- 传输选项：

  - 基于标准输入输出（stdio）的进程间通信传输
  - 基于 Java HttpClient 的 SSE 客户端传输
  - 用于响应式 HTTP 流的 WebFlux SSE 客户端传输

#### 1.2.2. MCP Server

![图片展示了MCP Server的两种架构。左侧为基于SSE（服务器发送事件）的架构，包含Client Process（Java LLM应用、Python MCP客户端、JavaScript MCP客户端）、MCP Client API、MCP Protocol（SSE）、Server Process（Web容器、Java MCP Server）、DBs等组件。右侧为基于STD IO（标准输入输出）的架构，同样有Client Process（Java LLM应用、Python MCP客户端、JavaScript MCP客户端）、MCP Client API、MCP Protocol（STDIO）、Server Process（Web容器、Java MCP Server）、DBs等组件。该图与文档中介绍MCP Server传输实现的内容相关，直观呈现了两种架构的组成与连接关系。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/F4tjbQ8S9oPUjSxF0jUcecWInGd.jpg)

- 服务器端协议操作实现

  - 工具暴露与发现
  - 基于URI的资源管理
  - 提示模板提供与处理
  - 与客户端的能力协商
  - 结构化日志记录和通知
- 并发客户端连接管理
- 同步和异步API支持
- 传输实现：

  - 基于标准输入输出（stdio）的进程间通信传输
  - 基于Servlet的SSE（服务器发送事件）服务器传输
  - 用于响应式HTTP流的WebFlux SSE服务器传输
  - WebMVC SSE 服务器传输，用于基于 Servlet 的 HTTP 流式传输

### 1.3. MCP Client实战

下面我们将基于`SpringAI`的`MCP`功能进行对`MCP`的`Client`进行学习。

#### 1.3.1. 功能需求

现在我们需要给AI大模型增强2个功能：

- 打开浏览器，浏览网页，进行总结网页中的内容

  - 提问：`打开网站：https://www.itcast.cn/，总结下这个网站的内容`
- 输入一个ip地址，让大模型解析出这个ip地址的所在地

  - 提问：`查询ip所在地：223.11.151.238`

显然，大模型如果不做增强的话，是做不到的，下面我们就基于MCP进行实现。

#### 1.3.2. 拉取代码

使用git拉取代码，地址：[https://gitee.com/zhijun.zhang/my-spring-ai-mcp.git](https://gitee.com/zhijun.zhang/my-spring-ai-mcp.git)

![图片展示了my - spring - ai - mcp项目中my - spring - ai - mcp - client模块的文件结构。在src目录下，main文件夹内有java文件夹，其中包含cn文件夹，里面有itcast、config、controller、dto、service等子文件夹，最下方是MCPClientApplication.java文件。此外，还有resources、test文件夹，以及pom.xml和.gitignore文件。该图片与文档中“拉取代码”部分相关，直观呈现了代码拉取后的项目结构。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/C85dbgItFodgM4xThoScEu0GnOc.png)

拉取完成后，启动服务，已经实现了基本的大模型对话功能，进行测试。（确保系统环境变量中存在`ALIYUN_API_KEY`信息）

![这张图片是接口调试工具的操作界面，用于测试Spring AI MCP的大模型对话功能。界面上方显示向地址为http://localhost:8100/chat/stream的POST接口发起请求，请求体为JSON格式，携带内容为“你好”的问题与标识为“123”的会话ID；下方时间线区域展示了接口返回的多段内容，包括“！”“今天”“过得怎么样？”等文本信息，以及标识对话结束的“\[END\]”，最终显示已断开连接，直观呈现了该服务的对话交互效果。](https://feishu.cn/file/WDjVbl5TgoJPtKxSmsIcQHgWnzf)

#### 1.3.3. 实战1：控制浏览器

我们的需求是，给大模型提问：`打开网站：https://www.itcast.cn/，总结下这个网站的内容`，就会进行相应的操作。

![这是一张接口测试工具的截图，展示了请求及响应的相关内容。上方的请求设置中，请求方法为POST，目标地址是http://localhost:8100/chat/stream，请求体的JSON包含question和sessionId两个参数。下方的响应时间线区域里，有红色标注的文字“默认情况下，是无法打开的”，箭头指向请求中提及的网址“https://www.itcast.cn/”，体现出该网址在默认状态下无法被访问打开的情况，这段内容和前文介绍的MCP Client实战流程相关，对应了通过MCP相关操作无法正常访问指定网站的场景。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/QHtMbFCftobzaexhgDMc8XMinbf.png)

下面我们来增强下大模型。

**第一步**，先在本机通过npm安装`@executeautomation/playwright-mcp-server`，如下：

![图片展示了在命令行中执行npm操作的界面。首先输入“npm -v”命令，显示npm版本为10.9.0；接着输入“npm install -g @executeautomation/playwright-mcp-server”命令，安装@executeautomation/playwright-mcp-server包，结果显示更改了83个包，耗时7秒。该图片对应文档中“第一步”内容，即在本机通过npm安装`@executeautomation/playwright-mcp-server`，直观呈现了操作及结果。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/F1IRbcyE1oJwnlx0gybc7asrnBc.png)

```Shell
#使用国内源安装命令：

npm install -g @executeautomation/playwright-mcp-server --registry=https://registry.npmmirror.com
```

**第二步**，在`my-spring-ai-mcp-client`中增加依赖：

```XML
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-starter-mcp-client-webflux</artifactId>
</dependency>
```

**第三步**，增加`mcp-servers.json`文件：

```JSON
{
  "mcpServers": {
    "playwright": {
      "command": "npx.cmd",
      "args": [
        "-y",
        "@executeautomation/playwright-mcp-server"
      ]
    }
  }
}
```

🚨 这一步，一定要确定在cmd命令行中，能够执行npx命令，如果不能执行，重新安装nodejs：

![这张图片展示了在Windows命令行中执行`npx -v`命令后的结果，命令行路径为`C:\\Users\\42084`，执行该命令成功输出了npx的版本号10.9.0。该图片用于验证步骤中的关键检查项，即确认在cmd命令行中可以正常执行npx命令，是MCP Client实战流程里提醒用户需完成的验证环节，用于确保后续操作能正常开展。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/JK0Ib5l2koMezpxbiezcgdcWnOS.png)

**第四步**，在`application.yml`中增加`MCP`的配置内容：

```YAML
server:
  port: 8100  #端口
  tomcat:
    uri-encoding: UTF-8   #服务编码
spring:
  application:
    name: my-spring-ai-mcp-client
  ai:
    dashscope:
      api-key: ${ALIYUN_API_KEY}
      chat:
        options:
          model: qwen-plus
    mcp:
      client:
        enabled: true # 启用mcp客户端
        name: ${spring.application.name} # 客户端名称
        version: 1.0.0 # 版本号
        request-timeout: 30s # 请求超时时间
        toolcallback:
          enabled: true # 开启工具回调功能
        stdio:
          servers-configuration: classpath:mcp-servers.json # mcp服务器配置文件
```

**第五步**，在`SpringAIConfig`中配置Tools

```Java
package cn.itcast.config;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.ai.tool.ToolCallbackProvider;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SpringAIConfig {

    private static final String SYSTEM_PROMPT = """
            你是一个全能助手，可以帮我解决各种问题。
            """;

    /**
     * 创建并返回一个ChatClient的Spring Bean实例。
     *
     * @param builder 用于构建ChatClient实例的构建者对象
     * @return 构建好的ChatClient实例
     */
    @Bean
    public ChatClient chatClient(ChatClient.Builder builder,
                                 ToolCallbackProvider tools
    ) {
        return builder
                .defaultSystem(SYSTEM_PROMPT) // 设置默认的系统角色
                .defaultTools(tools) // 设置默认的工具
                .build();
    }

}
```

**第六步**，重启服务，进行测试：

![图片展示的是Postman中对`http://localhost:8100/chat/stream`接口的POST请求界面。请求体为JSON格式，包含`question`和`sessionId`字段。下方时间线显示了请求成功及网站内容获取的相关信息，如“网站已经成功”“打开，现在”“我将获取该页面的可见文本”等，还列出了部分可见文本内容。该图片与文档中“第六步，重启服务，进行测试”内容对应，直观呈现了测试过程中的请求与响应情况。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/VhVsbFFE0oyOD4x30KecSy7vnuH.png)

![这张图片展示了一个浏览器界面，访问的是传智教育的官方网站（域名itcast.cn），页面中包含该机构的教育课程信息，如JavaEE、Python+大数据开发等众多课程类别。页面核心区域有醒目的红色宣传横幅，内容是针对大学生就业难的相关培训推广信息，右侧还弹出了传智教育的咨询会话窗口，显示了客服的问候及介绍内容，这张图片的场景与上下文里的MCP Client实战技术部署内容无关，更贴合传智教育相关的推广与咨询界面。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/CeoxbWq9cotmmWxXLYRci64znja.png)

#### 1.3.4. 原理分析

通过上面的实战，已经可以看到大模型可以控制浏览器了，是怎么做到的呢？

实际上，底层的实现也是基于Tool Calling实现的，由于配置了mcpServers，就拥有了好多的工具，注册到SpringAI中。

![该图片展示了Spring AI MCP相关的工具调用原理分析的代码运行结果，左侧的Evaluate面板中，代码片段tools.getToolCallbacks()执行后，返回了包含8个元素的工具集合result，每个元素为一个SyncMcpToolCallback实例，说明配置mcpServers后，相关工具成功注册到SpringAI中，这些工具遵循MCP协议开发，可被调用以实现控制浏览器的功能，对应上下文介绍的基于Tool Calling的底层实现逻辑。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/FG5zb75qZoN0qbxcpCtcCqoHnee.png)

而这些工具的开发，并不是我们自己做的，也不是用java开发的，是别人开发好的，遵循了MCP协议，所以就可以集成到SpringAI中使用了，而这工具就是用来控制浏览器的，所以就实现了上面的效果。

#### 1.3.5. 实战2：ip地址所在地

需求是 ，向大模型提问：`查询ip所在地：223.11.151.238`，就可以查询到ip地址的所在地。要想实现这个功能，可以借助与高德地图服务来实现，也不需要我们自己来对接，直接基于写好的MCP服务即可。

##### 1.3.5.1. 查找MCP服务

首先，需要解决一个问题，就是MCP服务从哪里能找到呢？我们可以通过这个网站来查找：[https://mcp.so/zh/](https://mcp.so/zh/)

![这张图片是mcp.so网站的页面，属于MCP（模型上下文协议）相关的服务平台界面，对应文档中提及的可查找MCP服务的网站。页面顶部展示了该平台的核心定位，标注为最大的MCP服务器聚合平台，可发现各类MCP服务；页面内的搜索框中输入了“amap”关键词，搜索结果区域展示了多个与高德地图相关的MCP服务，包括高德地图官方MCP服务，以及基于高德地图的天气服务等关联服务，可用于查找相关MCP资源。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/Bdm4bPLjWoDtPdxStCwchQPmnxf.png)

用法：

![这张图片是MCP Client实战环节里的配置参考内容，展示的是Cursor平台接入高德地图MCP服务的配置方法。内容包含接入配置的相关说明，提示需要复制配置的key，配置代码区域的箭头标注了“用法”，配置代码里明确标注要将AMAP_MAPS_API_KEY替换成用户在高德服务申请的Web服务密钥，同时说明支持MCP协议的客户端如Cursor、Claude、Cline均可使用高德地图MCP server。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/Ed8LbTccdoYchRxYFGGcSK0bnNG.png)

![图片展示的是MCP平台中“工具”页面。页面上方有“概览”“内容”“工具”三个选项卡，当前选中“工具”。在“工具”下，有maps_regeocode、maps_geo两个选项，其中maps_ip_location选项被红色箭头指向，其下方有“有提供IP查询的工具”的提示，且有“ip”输入框，提示输入IP地址，下方还有“运行”按钮。该图片与文档中介绍MCP Client集成到项目时增加配置的内容相关，展示了其中提供IP查询功能的工具。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/H6esb9fxSoOChzxxnlacArLXnWb.png)

##### 1.3.5.2. 集成到项目

首先通过npm安装：

![这张图片展示了在Windows系统命令行中执行npm安装命令的结果，具体为在路径C:\\Users\\42084>下，输入命令npm install -g @amap/amap-maps-mcp-server，执行后显示3秒内变更了44个安装包，完成安装后回到了C:\\Users\\42084>的命令行提示符。该命令对应文档中“首先通过npm安装”的步骤，与MCP集成项目的操作要求相关，完成此步后可进行后续的配置操作。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/ZO6GbwOJnoYunuxvgnKcscbsnYf.png)

```Shell
#使用国内源安装命令：

npm install -g @amap/amap-maps-mcp-server --registry=https://registry.npmmirror.com
```

增加配置：

```JSON
{
  "mcpServers": {
    "playwright": {
      "command": "npx.cmd",
      "args": [
        "-y",
        "@executeautomation/playwright-mcp-server"
      ]
    },
    "amap-maps": {
      "command": "npx.cmd",
      "args": [
        "-y",
        "@amap/amap-maps-mcp-server"
      ],
      "env": {
        "AMAP_MAPS_API_KEY": "xxxxxx"
      }
    }
  }
}
```

🚨 注意：AMAP_MAPS_API_KEY这个值要填自己在高德服务申请的key值。

申请地址：[https://lbs.amap.com/dev/key](https://lbs.amap.com/dev/key)

申请时注意【服务平台】选项，需要申请【Web服务】的key。

![这张图片展示的是高德地图开放平台申请密钥时的配置页面，明确标注了需填写的“服务平台”相关选项，其中“Web服务”选项被红色箭头指向并选中，符合文档中提醒申请AMAP_MAPS_API_KEY时服务平台需选择Web服务的要求。页面还设有Key名称输入项，提示其支持的字符规则，以及可使用服务列表、IP白名单配置区域，同时在底部有相关服务协议的阅读提示，页面来源标注为黑马程序员-花和尚。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/Cg5FbVerfoctJNxHwHuct8eVnQh.png)

##### 1.3.5.3. 测试

重启服务，进行测试：

![这张图片展示了测试MCP相关功能的操作结果，界面为POST请求页面及下方的时间线响应信息。上方Body区域的动态值中，带有“question”“sessionId”参数，question值为“查询ip所在地:223.11.151.238”。时间线区域的响应内容里，明确显示已成功查询到该IP的所在地信息，具体包含IP前缀22、3.11、.151、.238，以及对应的所在地为中国山西省太原市，还提示有更详细的地理位置信息，右侧配有红色箭头标注“已经查询到所在地信息了”，体现测试功能的正常运行结果。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/T6nVbluZMoCmDoxPEXscoSsOngc.png)

#### 1.3.6. 练习：MCP操作Redis

按照上述的操作方法，为大模型增加Redis的操作工具。参考：

[https://mcp.so/zh/server/redis/modelcontextprotocol](https://mcp.so/zh/server/redis/modelcontextprotocol)

### 1.4. MCP Server实战

前面我们都是使用的已经写好的MCP Server，实际上，也可以自己来实现的。

接下来，我们将写一个自己的天气查询服务，将他封装成MCP Server，这样需要的天气查询服务的大模型，就可以直接集成了。

#### 1.4.1. 创建my-spring-ai-mcp-server

![这张图片展示的是创建新项目模块的配置界面，界面左侧选中了Java选项，顶部对应“New Module”窗口，界面右侧将项目名称设置为“my-spring-ai-mcp-server”，项目构建系统为Maven，JDK选择Project JDK 17，Parent模块关联“my-spring-ai-mcp”，此界面对应文档中“1.4.1.创建my-spring-ai-mcp-server”的实操步骤，是搭建Spring AI MCP Server过程里的模块创建操作界面，用于完成天气查询服务所封装的MCP Server的项目创建初始配置。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/D0vAbtC5RorbNhxDG9ZcJ6L3nsb.png)

#### 1.4.2. 导入依赖

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.tianji</groupId>
        <artifactId>my-spring-ai-mcp</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>my-spring-ai-mcp-server</artifactId>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-starter-mcp-server-webflux</artifactId>
        </dependency>
    </dependencies>
</project>
```

#### 1.4.3. 编写配置

```YAML
server:
  port: 8101  #端口
  tomcat:
    uri-encoding: UTF-8   #服务编码
spring:
  application:
    name: my-spring-ai-mcp-server
  ai:
    mcp:
      server:
        enabled: true
        name: ${spring.application.name}
        version: 1.0.0
        type: ASYNC # 服务类型 异步，支持 SYNC 和 ASYNC
```

#### 1.4.4. 启动类

```Java
package cn.itcast;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.core.env.Environment;

import java.net.InetAddress;
import java.net.UnknownHostException;

@Slf4j
@SpringBootApplication
public class MCPServerApplication {

    public static void main(String[] args) throws UnknownHostException {
        SpringApplication app = new SpringApplicationBuilder(MCPServerApplication.class).build(args);
        Environment env = app.run(args).getEnvironment();
        String protocol = "http";
        if (env.getProperty("server.ssl.key-store") != null) {
            protocol = "https";
        }
        log.info("--/\n---------------------------------------------------------------------------------------\n\t" +
                        "Application '{}' is running! Access URLs:\n\t" +
                        "Local: \t\t{}://localhost:{}\n\t" +
                        "External: \t{}://{}:{}\n\t" +
                        "Profile(s): \t{}" +
                        "\n---------------------------------------------------------------------------------------",
                env.getProperty("spring.application.name"),
                protocol,
                env.getProperty("server.port"),
                protocol,
                InetAddress.getLocalHost().getHostAddress(),
                env.getProperty("server.port"),
                env.getActiveProfiles());
    }

}
```

#### 1.4.5. 编写工具

```Java
package cn.itcast.dto;

import com.fasterxml.jackson.annotation.JsonPropertyDescription;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class WeatherDTO {

    @JsonPropertyDescription("城市ID")
    private String cityId;

    @JsonPropertyDescription("城市名称")
    private String city;

    @JsonPropertyDescription("当前温度（单位：℃）")
    private String temperature;

    @JsonPropertyDescription("低温（单位：℃）")
    private String lowTemperature;

    @JsonPropertyDescription("高温（单位：℃）")
    private String highTemperature;

    @JsonPropertyDescription("数据日期（格式：YYYYMMDD）")
    private String date;

    @JsonPropertyDescription("空气质量指数")
    private String quality;

    @JsonPropertyDescription("PM2.5 浓度（单位：微克/立方米）")
    private double pm25;

}
```

```Java
package cn.itcast.tools;

import cn.hutool.http.HttpUtil;
import cn.hutool.json.JSONObject;
import cn.hutool.json.JSONUtil;
import cn.itcast.dto.WeatherDTO;
import org.springframework.ai.tool.annotation.Tool;
import org.springframework.ai.tool.annotation.ToolParam;
import org.springframework.stereotype.Service;

@Service
public class WeatherService {

    @Tool(description = "根据城市id查询天气信息")
    public WeatherDTO getWeather(@ToolParam(description = "城市id") String cityId) {
        // 通过http请求获取天气信息，并且通过json数据解析为WeatherDTO对象
        String url = "http://t.weather.itboy.net/api/weather/city/" + cityId;
        String data = HttpUtil.get(url);
        JSONObject jsonObject = JSONUtil.parseObj(data);

        return WeatherDTO.builder()
                .cityId(jsonObject.getByPath("cityInfo.citykey", String.class)) // 城市ID
                .city(jsonObject.getByPath("cityInfo.city", String.class)) // 城市名称
                .date(jsonObject.getByPath("date", String.class))// 数据日期
                .temperature(jsonObject.getByPath("data.wendu", String.class))   // 当前温度
                .lowTemperature(jsonObject.getByPath("data.forecast[0].low", String.class))// 低温
                .highTemperature(jsonObject.getByPath("data.forecast[0].high", String.class))// 高温
                .quality(jsonObject.getByPath("data.quality", String.class))// 空气质量
                .pm25(jsonObject.getByPath("data.pm25", Double.class))// PM2.5数值
                .build();
    }

}
```

#### 1.4.6. MCPServer配置

```Java
package cn.itcast.config;

import cn.itcast.tools.WeatherService;
import org.springframework.ai.tool.ToolCallback;
import org.springframework.ai.tool.ToolCallbacks;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.List;

@Configuration
public class McpConfig {

    /**
     * 申明对外提供服务的工具
     */
    @Bean
    public List<ToolCallback> weatherTools(WeatherService weatherService) {
        return List.of(ToolCallbacks.from(weatherService));
    }

}
```

#### 1.4.7. 启动测试

浏览器访问地址：[http://localhost:8101/sse](http://localhost:8101/sse)

![图片展示的是浏览器访问地址为http://localhost:8101/sse的页面内容。页面显示“event: endpoint”和“data:/mcp/message?sessionId=91cc44d7-1e90-4433-a41b-def9bc1e11c3”两行信息。该图片与文档中“1.4.7.启动测试”部分内容相关，说明MCPServer已启动成功，通过浏览器访问指定地址可查看相关信息，体现了MCP Server实战中启动测试的成果展示。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/AwDmb4Tsko4zlWxV0ONcAJtlnTc.png)

说明，MCPServer已经启动成功。

#### 1.4.8. 在MCPClient中集成服务

只需要在MCPClient中指定服务地址即可。

```YAML
server:
  port: 8100  #端口
  tomcat:
    uri-encoding: UTF-8   #服务编码
spring:
  application:
    name: my-spring-ai-mcp-client
  ai:
    dashscope:
      api-key: ${ALIYUN_API_KEY}
      chat:
        options:
          model: qwen-plus
    mcp:
      client:
        enabled: true # 启用mcp客户端
        name: ${spring.application.name} # 客户端名称
        version: 1.0.0 # 版本号
        request-timeout: 30s # 请求超时时间
        toolcallback:
          enabled: true # 开启工具回调功能
        stdio:
          servers-configuration: classpath:mcp-servers.json # mcp服务器配置文件
        type: ASYNC # 服务类型，指定为异步，支持 SYNC 和 ASYNC
        sse:
          connections:
            server1:
              url: http://localhost:8101 # 指定MCP Server服务器地址
```

启动服务，打断点可以看到，已经有天气的工具了：

![图片展示了在MCPClient中集成服务后的调试界面。上方代码片段为“tools.getToolCallbacks()”，下方结果中显示了多个ToolCallback对象，其中第0个对象的“tool”属性值为“getWeather”，其“name”为“getWeather”，“description”为“根据城市id查询天气信息”，“inputSchema”为“JsonSchema@8337”等信息。该图片与上下文紧密相关，直观呈现了在MCPClient中集成服务后，通过工具获取到的天气查询工具相关信息，验证了服务集成的正确性。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/BTuOb59MUo5bkYxfOTscabQ4nvb.png)

#### 1.4.9. 功能测试

![这张图展示了在开发测试工具中，通过指定参数向服务发送请求后，成功获取了对应天气信息的结果。图中POST请求的Body里包含"question"与"sessionId"参数，下方的响应内容清晰显示了查询到的北京天气相关信息，包括温度为16.3℃、最低温度1℃等具体数据，还配有红色箭头标注“查询到天气信息了”，对应了文档中在MCPClient集成服务后查看天气工具效果的相关说明，验证了MCP Server与Client交互的天气查询功能可以正常运行。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/JsnCbmoQCoqUbMx2YmzcZiMtnRh.png)







## 在trae中使用MCP

MCP（Model Context Protocol)，模型上下文协议，由Claude的母公司，Anthropic于2024年底开源发布。



### 什么是MCP

来看看官网的定义：

> MCP 是一种开放协议，它标准化了应用程序如何为 LLM 提供上下文。将 MCP 想象成 AI 应用程序的 USB-C 端口。正如 USB-C 提供了一种将设备连接到各种外围设备和配件的标准化方式一样，MCP 也提供了一种将 AI 模型连接到不同数据源和工具的标准化方式。

具体大家可以参考MCP官网：

<bookmark name="Introduction - Model Context Protocol" href="https://modelcontextprotocol.io/introduction"></bookmark>



<bookmark name="FAQs - Model Context Protocol" href="https://modelcontextprotocol.io/faqs"></bookmark>



MCP采用的是CS架构，分为Client和Server端：

![图片展示了MCP架构示意图。左侧为带有MCP客户端的主机（Claude、IDEs、Tools），通过MCP协议与MCP Server A、B、C连接。MCP Server A、B可访问本地数据源A、B，C通过Web APIs连接远程服务C。右侧是你的电脑，MCP Server A、B、C与之相连。该图直观呈现了MCP架构中Client、Server端及与本地数据源、远程服务的交互关系，与上下文介绍的MCP架构相呼应。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/WxHkbrmR8oCu00xS5uHcv2zIn9c.png)

- MCP Hosts：希望通过 MCP 访问数据的 Claude Desktop、IDE 或 AI 工具等程序
- MCP Client：与服务器保持 1对1 连接的协议客户端
- MCP Server：轻量级程序，每个程序都通过标准化的 Model Context Protocol 公开特定功能
- LocalDataSource：MCP 服务器可以安全访问的本地的计算机文件、数据库和服务
- RemoteService：MCP 服务器可以连接到的 Internet 上可用的外部系统（例如，通过 API）



也就是说，利用MCP Server我们可以非常方便的访问本地资源、第三方接口资源等等，最重要的是，所有访问这些资源的MCP Server都遵循了MCP的协议，因此我们无需重复开发这些MCP Server。



以高德地图提供的MCP为例，其工作流程如图：

![图片展示了高德域MCP服务的工作流程。用户输入经Client传递给LLM，LLM通过SSE与MCP Server A、MCP Server B交互，MCP Server A连接基础REST服务，MCP Server B连接高阶REST服务。该图与文档中介绍MCP工作流程的内容相关，直观呈现了利用MCP访问本地资源、第三方接口资源等流程，强调了MCP Server遵循协议、无需重复开发的特点。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/WkgTbjZjAo1gCXxryPkclyO6nxb.png)





### MCP服务

我们可以在MCP的官方找到常见的MCP服务：

<bookmark name="Example Servers - Model Context Protocol" href="https://modelcontextprotocol.io/examples"></bookmark>



另外，还有一些第三方平台集成了各种MCP服务：



- awesomemcp

<bookmark name="github.com" href="https://github.com/punkpeye/awesome-mcp-servers"></bookmark>



- SmitheryAI，集成了5000多个MCP服务

<bookmark name="smithery.ai" href="https://smithery.ai/"></bookmark>



mcp.so，收录了10000+MCP服务：

<bookmark name="MCP Server（MCP 服务器）" href="https://mcp.so/zh"></bookmark>



### Trae集成MCP

字节出品的AI开发工具Trae现已集成了MCP功能，非常强大。接下来我们来看看如何在Trae中使用MCP。



### 添加MCP服务

在Trae聊天界面中，点击右上角的齿轮：

![图片展示了在Trae聊天界面中添加MCP服务的操作步骤。在聊天界面右上角点击齿轮图标，弹出菜单中选择MCP，进入MCP管理页面后点击添加MCP服务。接着点击手动配置按钮，在弹出的菜单中填写高德MCP的服务信息，如服务名称、服务ID、服务密钥等，点击确认后，高德MCP就添加成功了。此图直观呈现了添加MCP服务的操作流程，与上下文介绍的添加MCP服务步骤相呼应。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/SnNKbk3Erone16xl0AzcKwxhntc.png)

然后在弹出的菜单中选择MCP：

![图片展示了在Trae聊天界面中点击右上角齿轮后弹出的菜单。菜单中“智能体”“上下文”“规则”“模型”选项下方，有“MCP”选项，其被红色框线突出显示。该图片与文档中“添加MCP服务”部分内容相关，对应在Trae聊天界面中点击右上角齿轮后，在弹出菜单中选择“MCP”这一步骤，直观呈现了操作位置。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/IwBDbudJFoO4Kcxm00dcRZTNnJf.png)

进入MCP管理页面，点击添加MCP服务：

![图片展示的是Trae集成的MCP管理页面。页面上方有“MCP”字样，右上角有齿轮状图标和“x”关闭按钮。下方介绍MCP Servers，即支持Model Context Protocol的服务，可提供工具和功能扩展智能体能力。关键信息是页面中部的“+ 添加MCP Servers”按钮，该按钮以红色框突出显示，是手动添加MCP服务的入口，与上下文介绍的添加MCP服务操作紧密相关。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/UeFhbAZE6op03Cxha4fcBSfBnud.png)

Trae集成了一个MCP市场，其中有大量优质MCP：

![图片展示的是Trae集成的MCP市场页面。页面上方有“MCP”搜索框，右上角有齿轮状图标。下方标题为“MCP / 市场”，并有“如何配置MCP Servers”提示。页面中部有“搜索”框，下方列出多个MCP服务，如Puppeteer、PostgreSQL、GitHub、Figma AI Bridge等，每个服务右侧都有蓝色的“+”按钮。页面底部提示“想要添加市场以外的MCP Servers？手动配置”。该图与文档中介绍Trae集成MCP市场，有大量优质MCP的内容相关，展示了市场中可选择的MCP服务。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/Zu2MbZqGgooAZbxf00Mc0bw8nrf.png)

当然，我们也可以自己手动添加MCP服务，接下来我们以高德服务为例，来看看如何手动添加。

有关高德MCP信息，大家可以参考高德的官方说明：

<bookmark name="概述-MCP Server | 高德地图API" href="https://lbs.amap.com/api/mcp-server/summary"></bookmark>



点击上图中的手动配置按钮，在弹出的菜单中填写高德MCP的服务信息：

![这张图片展示的是在Trae中手动添加MCP服务时的配置界面，属于“手动配置”的操作页面。页面内标注了配置高德MCP服务的示例代码，代码中定义了`amap-maps`服务的相关配置，包括使用`npx`执行的命令、对应的参数，还在注释提示行中说明，需要将`AMAP_MAPS_API_KEY`替换为用户在高德官网上申请的密钥。界面右上角设有“原始配置（JSON）”选项，底部配有“取消”和“确认”按钮，用于完成配置或取消操作，整体是手动添加高德MCP服务的关键配置区域。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/Hi2ibszA5oC5bAxLmy5cUlaVnRh.png)

高德MCP服务信息如下：

```JSON
{
  "mcpServers": {
    "amap-maps": {
      "command": "npx",
      "args": ["-y", "@amap/amap-maps-mcp-server"],
      "env": {
        "AMAP_MAPS_API_KEY": "您在高德官网上申请的key"
      }
    }
  }
}
```



点击确认后，高德MCP就添加成功了。其中包含十几个不同的tool：

![这是Trae平台中MCP服务的管理界面，当前展示的是MCP Servers列表，其中标记为黄色的amap-maps服务状态为可使用，且已完成添加。界面右侧设有“添加”按钮，列表区域列出了amap-maps包含的十余种工具，每个工具对应明确的功能说明，比如maps.regeocode可完成高德坐标与地址信息的互相转换，maps.geo能将结构化地址转换为经纬度等，该界面对应文档中“添加高德MCP服务”的操作环节。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/Lp3abd4AsoS72ZxjURTcGhofnBb.png)



### 创建智能体

接下来，我们在Trae中创建一个智能体：

![这是Trae的操作界面截图，右上角带有齿轮样式的设置按钮，点击该按钮后弹出的下拉菜单中，“智能体”选项被红框标注突出显示，菜单内还包含MCP、上下文、规则、模型等选项。界面下方设有代表智能体的图标，结合上下文可知，这是在Trae中创建智能体前，点击设置按钮后选择进入智能体功能页面的操作步骤界面。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/H0hKbBJnXoFLe6xCYYHcrM70nbf.png)

进入智能体管理页面：

![图片展示了Trae平台中智能体管理页面。页面上方有“智能体”标签，右侧有“了解更多”按钮。下方“智能体列表”区域显示“自定义智能体”暂无自定义智能体创建，内置智能体有“Builder”和“Builder with MCP”。底部“自动运行”区域有“自动运行命令和MCP工具”开关，当前处于开启状态，下方有“rm”“kill”“chmod”等黑名单选项。图片中“创建智能体”按钮和“自动运行”开关被红色框突出显示，与上下文介绍的创建智能体操作和自动运行设置内容对应。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/XN8Lb9N2JoWTkCxxGQgcWdFsnDf.png)

点击创建智能体：

![这张图片是Trae的“创建智能体”操作界面，用于配置高德MCP工具与智能体关联。界面上方有红色标注的区域，用于指定智能体名称，当前输入的名称为“旅行规划师”；中间区域为工具配置区，红色标注说明可勾选智能体能使用的工具，其中MCP工具里的“amap-maps”已被勾选，内置工具里的“联网搜索”也已勾选，其余如文件系统、终端等工具未被勾选；界面底部设有“创建”与“取消”按钮，完成配置后点击“创建”，选择立即使用即可与该智能体对话。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/HSH4bRKTSoIIhBxhEtycCi3vnkf.png)

点击创建，选择立即使用：

![该界面是Trae平台创建智能体流程的完成页面，页面上方显示绿色对勾图标，搭配“创建成功”的提示文字，说明智能体已成功创建，下方配有两行提示内容，第一行提示“在对话中@智能体即可调用它”。界面设有两个核心操作按钮，上方为蓝色的“立即使用”按钮，用于直接启动与该智能体的对话，下方为浅灰色的“查看智能体列表”按钮，可用于查看当前创建的所有智能体。该页面出现在创建智能体的“点击创建，选择立即使用”步骤之后，是完成创建后的操作选择环节。](../../../../Attachments/development/ai/spring-ai-alibaba-yu-mcp/EnTmbLbEQo2OAyxjU8JcqfGuncE.png)

即可与智能体对话了：

---
> 来源：飞书技术知识库（脱敏版） · 06 AI 应用开发 / Spring AI Alibaba 与 MCP / 拓展内容-MCP · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/QUQ2wZ5ayixDVZkiXSbcqkLHncd)
