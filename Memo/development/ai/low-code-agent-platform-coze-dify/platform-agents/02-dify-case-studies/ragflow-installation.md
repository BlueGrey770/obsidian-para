---
type: memo
date: 2026-11-06
aliases:
  - "ragflow-an-zhuang"
  - "RAGFlow安装"
---

# RAGFlow安装

## 镜像准备

上传资料中的RAGFlow的镜像文件到服务器中，加载为镜像

![edbjbeosrowsnrxmxdrca6tfnwd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/edbjbeosrowsnrxmxdrca6tfnwd.png)
> 图注：这张图片展示了五个压缩包文件，从文件名可判断其为安装RAGFlow所需的相关镜像文件，分别是elasticsearch.tar、minio2025.tar、mysql.tar、ragflow.tar、valkey.tar。结合上下文，这些镜像文件正是需要按照要求上传到服务器并加载为镜像的内容，文档中提到该类镜像文件占用内存较大，需调大虚拟机内存，这些压缩包即为对应的各类镜像文件。

执行命令：

```Bash
docker load -i elasticsearch.tar | docker load -i minio2025.tar | docker load -i mysql.tar | docker load -i ragflow.tar | docker load -i valkey.tar 
```

> 镜像较大，加载时长要耐心等待



注意：由于这几个镜像占用内存较大，建议把虚拟机的内存调大，10g左右即可

![noc0bzcsvommnvx6quqcfhm5nsg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/noc0bzcsvommnvx6quqcfhm5nsg.png)
> 图注：图片展示了虚拟机设置中硬件配置的内存部分。当前虚拟机内存设置为6.1 GB，可选择的内存范围从4 MB到64 GB不等。其中，64 GB被红色框突出显示，旁边有提示“此虚拟机的内存（M）：6208 - MB”。该图片与文档中“建议把虚拟机的内存调大，10g左右即可”的内容相关，用于说明在安装RAGFlow时，为避免内存不足导致加载镜像时间过长，需将虚拟机内存调大，此处以64 GB为例进行展示。





## 下载github项目源码

<bookmark name="GitHub - infiniflow/ragflow: RAGFlow is a leading open-source Retrieval-Augmented Generation (RAG) e" href="https://github.com/infiniflow/ragflow.git"></bookmark>

![sbopb6ffkon4ixxq59jcffb5njc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/sbopb6ffkon4ixxq59jcffb5njc.png)
> 图注：这张图片展示的是GitHub平台上infiniflow/ragflow项目的代码页面，页面顶部是GitHub的导航栏，下方是该项目的仓库基础信息，包含Star、Fork、Pull requests等数据统计。页面右侧有一个展开的下载选项菜单，其中“Download ZIP”选项被红色方框标注，该选项用于将ragflow项目的源码以ZIP压缩包的形式下载，这正好对应了文档中下载github项目源码的相关操作步骤。



然后解压`ragflow-main.zip` 压缩包（当天资料中已提供，最好创建一个目录）

```Bash
unzip ragflow-main.zip
```

由于前面我们已经安装了Dify项目，这两个项目都依赖了redis，且web端的端口都是默认80端口，因此，为了避免冲突，我们需要修改ragflow-main目录下的相关配置文件。

## 修改配置信息

### 修改web访问默认端口

修改文件位置： `ragflow-main/docker/docker-compose.yml` 文件

**原来的**                                                                                    **修改后的**

![gwr3bmhdmo1xdkxk2taczmuenbc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/gwr3bmhdmo1xdkxk2taczmuenbc.png)
> 图注：图片展示的是`ragflow-main/docker/docker-compose.yml`文件中关于`ports`配置的部分内容。其中，`5678:5678`这一行被高亮显示，表明其为关键修改点。上下文提到在修改web访问默认端口时，需在该文件中修改此端口配置，此图片直观呈现了修改前的端口配置情况，为后续修改操作提供了参考，帮助用户准确找到修改位置。

![ksa0bsusookz2cxt7mzcmpt9nhh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ksa0bsusookz2cxt7mzcmpt9nhh.png)
> 图注：图片展示了`ragflow-main/docker/docker-compose.yml`文件中`ports`配置部分。其中，8880端口被红色框突出显示，原端口为8880，修改后端口为80。该图片与文档中“修改web访问默认端口”内容相关，对应修改文件位置`ragflow-main/docker/docker-compose.yml`，是修改web访问默认端口操作的示例展示，直观呈现了端口修改前后的对比情况。


### 修改redis配置

修改 `ragflow-main/docker/.env` 文件

**原来的**                                                                          **修改后的**

![iwkdbnmhlovo9rxwrehc1qamnid](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/iwkdbnmhlovo9rxwrehc1qamnid.png)
> 图注：这张图片是RAGFlow安装流程中修改Redis配置的对应内容，显示的是`ragflow-main/docker/.env`文件里的Redis相关配置项。其中，原来的配置内容清晰可见，标注出的关键内容分别是`REDIS_HOST=redis`、`REDIS_PORT=6379`，这些参数为Redis的主机地址与端口号，图片对应了文档中“修改redis配置”的操作环节，是该步骤里需要调整的配置参数相关内容。

![zrgjbk3wbotkouxbjcbcxpennn8](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/zrgjbk3wbotkouxbjcbcxpennn8.png)
> 图注：图片展示了`ragflow-main/docker/.env``文件中83行至89行的内容。其中，`REDIS_HOST`变量值为`ragflow_redis`，`REDIS_PORT`变量值为`7379`。该图片与文档中“修改redis配置”部分相关，对应修改`ragflow-main/docker/.env`文件的操作，直观呈现了redis配置中主机名和端口的修改内容。


修改 `ragflow-main/docker/docker-compose-base.yml` 文件

**原来的**                                                                                    **修改后的**

![uh6ubqfdmoqeowx47sxcx4zwnce](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/uh6ubqfdmoqeowx47sxcx4zwnce.png)
> 图注：图片展示了`ragflow-main/docker/docker-compose-base.yml`文件中关于redis的配置内容。其中，`redis:`部分以红色框突出显示，包含redis相关配置项，如image为`valkey/valkey:8`，container_name为`ragflow-redis`，command为`redis-server --requirepass ${REDIS_PASSWORD}`，env_file为`.env`，ports为`${REDIS_PORT}:6379`，volumes为`redis_data:/data`，networks为`ragflow`，restart为`on-failure`。该图片与文档中“修改redis配置”部分对应，是修改redis配置时需参考的配置示例。

![qvi5bxf6ko05kjx3gxucoql8ndh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qvi5bxf6ko05kjx3gxucoql8ndh.png)
> 图注：图片展示了`ragflow-main/docker/docker-compose-base.yml`文件中关于`ragflow_redis`的配置内容。其中，`container_name`为`ragflow-redis`，`command`为`redis-server --requirepass ${REDIS_PASSWORD}`，`env_file`为`.env`，`ports`包含`6379`端口，`volumes`包含`redis_data:/data`，`networks`为`ragflow`，`restart`为`on-failure`。该图片与文档中“修改redis配置”部分对应，是修改`ragflow_redis`配置的示例。


## 启动服务

进入docker目录，执行命令：

```Python
docker-compose up -d
```

![](https://feishu.cn/file/R9Mfbt2bgoEo5Fx8hc1cRA9znQb)

时间比较久，需要大家耐心等待。



## 访问RAGFlow

由于前面，我们把web端口设置为了8880端口，docker镜像拉取后，等待容器启动完成，在浏览器输入:`192.168.100.168:8880` 即可访问

![cyarbzqvsoqnuqx8zdgccvovn0f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/cyarbzqvsoqnuqx8zdgccvovn0f.png)
> 图注：图片展示的是RAGFlow的登录页面。页面左侧有“登录”标题，提示“很高兴再次见到您！”，下方有邮箱和密码输入框，以及“记住我”和“忘记账户？”选项，底部有“登录”按钮。右侧是“开始构建您的智能助手”的宣传语，配有“免费注册以探索顶级RAG技术，创建知识库和人工智能来增强您的业务”的说明，底部有多个图标和“来自500多条评论”的标识。该图片与文档中“访问RAGFlow”部分对应，说明在安装完成后，注册并登录后可在此页面进行操作。

至此，ragflow就安装完成了，注册完成后主页面如下：

![b9mqbckkvomyhixqqr5c7j1jnkv](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/b9mqbckkvomyhixqqr5c7j1jnkv.png)
> 图注：这张图片是RAGFlow安装完成后注册成功的主页面，页面网址为127.0.1.1:8880，顶部导航栏设有“知识库”“聊天”“搜索”“智能体”“文件管理”等功能入口。页面核心区域展示了“欢迎来到RAGFlow”的标题，还设有“知识库”和“聊天”两个板块，各板块下方都配有“See All”的查看按钮，页面右侧还有“聊天”“搜索”“智能体”的快捷选项。

## 接入大模型

![dvpnbv6rxo6wb5xnjgtcc1c9nge](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dvpnbv6rxo6wb5xnjgtcc1c9nge.png)
> 图注：图片展示了RAGFlow主页面右上角区域，其中“小”字头像被红色框和箭头突出显示，并有“点击头像”文字提示。该图片与文档中“接入大模型”部分上下文相关，说明在RAGFlow安装完成后，注册完成后主页面的样式，用户可点击头像进行相关操作，为后续接入大模型操作做准备。

![fm7kb2yzioaam1x0rppc2zisnfh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/fm7kb2yzioaam1x0rppc2zisnfh.png)
> 图注：图片展示的是RAGFlow平台的主页面，左侧为导航栏，选中“横纵向提供商”。页面中部以网格形式呈现多个模型提供商，如Replicate、SILICONFLOW、StepFun等。其中，Tongyi - Qianwen模型提供商被红色框突出显示，其下方有“添加模型”按钮。该图片与文档中“接入大模型”部分对应，直观呈现了在RAGFlow平台添加大模型的操作入口，辅助用户了解如何接入大模型。



![ehhlbidxuokcutxyon3cfmqcn4c](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ehhlbidxuokcutxyon3cfmqcn4c.png)
> 图注：这张图片展示的是RAGFlow接入大模型流程中的API-Key修改界面。界面左侧标注了必填项“* API-Key”，输入框内已填入部分API密钥内容，开头显示的“57754e”清晰可见，剩余字符被模糊处理，输入框下方有红色波浪线提示格式异常，界面右下角设有“取消”和“确定”两个操作按钮，用于确认或取消当前的API密钥修改操作，该界面是接入大模型配置环节的关键操作入口。

![bvdmbu0a0on2doxigwfcn2hynqf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/bvdmbu0a0on2doxigwfcn2hynqf.png)
> 图注：这张图片展示的是接入大模型环节中，智谱AI（ZHIPU-AI）的相关功能模块界面，左侧标注了智谱AI的标识，同时列出了其提供的LLM、TEXT EMBEDDING、SPEECH2TEXT、MODERATION这些不同类型的AI功能选项，界面右侧设置了“API-Key”按钮和“展示更多模型”按钮，与文档中接入大模型的内容直接对应，体现了接入该大模型时可选择的功能及相关操作入口。


---
> 来源：飞书技术知识库（脱敏版） · 06 AI 应用开发 / 低代码智能体平台（Coze / Dify） / 平台智能体 / 02-Dify综合案例 / RAGFlow安装 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/CIVpwEkM4iObtNkugtPcINu3ntb)
