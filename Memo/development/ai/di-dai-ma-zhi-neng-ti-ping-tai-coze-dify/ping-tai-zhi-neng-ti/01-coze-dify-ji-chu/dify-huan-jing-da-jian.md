---
type: memo
date: 2026-11-06
aliases:
  - "Dify环境搭建"
---

# Dify环境搭建

**建议在安装前先把原来的容器和镜像都删了（除mysql和redis外）**

```XML
# 删除容器
docker rm -f 容器名称/容器id

# 删除镜像
docker rmi 镜像名称/镜像id

```

## 升级docker-compose

- 找到资料中的`docker-compose`文件，上传到`/usr/local/bin`目录，替换掉原来的docker-compose
- 授权`sudo chmod +x /usr/local/bin/docker-compose`
- 验证`docker-compose --version`

![这张图片展示了Dify环境搭建过程中升级docker-compose步骤的验证环节，分为上下两部分内容。上方是终端命令界面，成功执行`docker-compose --version`命令，输出显示Docker Compose版本为v2.23.0，确认docker-compose已成功安装。下方是文件目录界面，路径显示为`/usr/local/bin`，目录内存在名为`docker-compose`的文件，文件大小为56.9MB，修改时间为2026年1月26日16:59，权限为`-rw-r--r--`，用户/用户组为0/0，验证了docker-compose文件已按要求上传至指定目录。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/SJf5bRDwRoz6rSxRWChcqCgNnUs.png)

## 加载镜像

从当天资料中，把dify镜像上传到虚拟机中，最好在虚拟机中提前准备好一个目录（`dify_dev`），单独存储镜像

如下图：

![图片展示了Dify镜像的加载过程。左侧是dify_dev目录，显示了多个.tar文件，如dify-apis.tar等。右侧是终端界面，显示了“docker load -i”命令及加载镜像的进度，当前进度为100%。图片与上下文的关系是，在介绍加载镜像到docker中时，通过此图直观呈现了加载镜像的操作及进度情况，辅助理解加载镜像的具体步骤。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/CEepbgka4ogFJFxLL9HcgkFdn6e.png)

在 `dify_dev` 执行命令，加载镜像到docker中

```Bash
docker load -i dify-api.tar | docker load -i dify-plugin-daemon.tar | docker load -i dify-sandbox.tar | docker load -i dify-web.tar | docker load -i nginx.tar | docker load -i postgres.tar | docker load -i redis6-alpine.tar | docker load -i squid.tar | docker load -i weaviate.tar
```

![图片展示的是在虚拟机中执行docker load命令加载镜像到docker中的操作结果。命令依次加载了dify-api.tar、dify.plugin.daemon.tar等镜像文件，如dify-api.tar对应的镜像ID为2caee4937d17，创建时间为2个月前，大小为2.26GB等。加载完成后，还显示了sechitechologies/weariata:1.19.0镜像的相关信息。该图片与文档中“加载镜像”部分上下文对应，直观呈现了加载镜像的操作结果。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/DsZVbS7ldoP0qPxMNwvcFzzpngd.png)

## 安装Dify环境

### 下载Dify项目压缩包

github地址：https://github.com/langgenius/dify/tree/main

![这张图片展示的是GitHub平台上langgenius/dify项目的Code页面，页面顶部显示该项目为公开项目，还标注了Issues（463个）、Pull requests（108个）等功能入口。在页面右侧Code下拉菜单展开后，可看到“Download ZIP”选项被红色方框标注出来，对应文档中提到的“下载Dify项目压缩包”的内容，是获取Dify项目源码压缩包的操作入口。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/SWK1bBuFxotcsVx1sTdcBwJNn3d.png)

当然，在今天的资料我也已经提前帮大家下载好了对应的源码包

<figure view-type="Card"><source name="dify-main.zip" mime="application/x-zip-compressed" size="24161367" token="MLIJbvKYToLhBgx2LRrcOAJ2nJh"/></figure>



### 解压源码包

把源码上传到服务器中，还可以放到`dify_dev` 中

解压缩zip包

```Bash
# 先安装 unzip 指令
yum install unzip 

# 解压dify-main.zip 文件
unzip dify-main.zip

```

然后进入`dify_dev`目录中找到`docker`目录，然后进入`docker` 目录

![这是Dify环境搭建过程中解压源码包环节的服务器命令行操作界面截图。界面显示当前处于`/root/dify_dev/dify-main/docker`目录，由`pwd`命令确认当前路径；`ls`命令列出该目录下的全部文件与文件夹，其中包含后续操作涉及的`.env.example`隐藏文件，还有docker-compose相关配置文件、elasticsearch、nginx、volumes等多个与搭建Dify docker环境相关的内容项，该界面对应文档中进入`dify_dev/docker`目录的操作步骤。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/KI9MbUexDo0iIax7XfBcA0lynR7.png)



### .env文件重命名

在`docker`目录中找到`.env.example`隐藏文件，改名为`.env`

![这张截图展示了在服务器操作环境中，针对docker目录下的.env.example隐藏文件进行重命名的操作过程。第一部分显示执行`ls -a`命令的结果，能看到目标文件为高亮标注的`.env.example`，对应文档中“.env文件重命名”的前置步骤；第二部分显示执行`mv .env.example .env`重命名命令后，再次执行`ls -a`的结果，此时原高亮文件已被修改为高亮标注的`.env`，对应文档中“.env文件重命名”的具体操作，验证了这一配置文件名称调整的操作完成。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/EvL5bPvONoLmDkxFWDyckjxdn1f.png)

```Bash
# 可以查看目录中所有的文件，包括隐藏文件
ls -a 

# 改名 .env.example  --->  .env
mv .env.example .env
```



### 运行docker环境

在 `/root/dify_dev/dify-main/docker` 执行命令

```Python
# 创建并启动容器
docker-compose up -d

# 停止并移除容器
docker-compose down
```

![图片展示的是在服务器终端执行`docker compose up -d`命令后，Dify Docker环境启动成功的界面。界面上方显示命令执行结果，下方列出多个容器状态，如`docker_default`、`docker_db-1`等，状态均为`Started`，表明各容器已成功启动。该图片与文档中“运行docker环境”步骤对应，直观呈现了Docker环境启动后的容器状态情况。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/BN4cbEg26oaH5ixJlEucPNnKnIc.png)



## 启动Dify

在浏览器中输入 [http://192.168.100.168/install](http://192.168.100.168/install) 

首次登录需要进行注册

![图片展示的是Dify首次登录注册页面。页面上方显示“Dify”标志及网址“192.168.100.168/install”。中间标题为“设置管理员账户”，说明管理员拥有最大权限，可用于创建后端/管理LLM供应商等。下方有“邮箱”“用户名”“密码”三个输入框，底部有“设置”按钮。页面右上角有“重置密码”选项。该图片与文档中“首次登录需要进行注册”内容对应，直观呈现了注册时的界面及输入项。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/PSjcbzjdYoctCQxBVIKcvf1WnUb.png)



注册之后，可以进入到主页面

![图片展示的是Dify平台的主页面。页面左上角显示“Dify”及“yuhegao's Workspace”。左侧有“我的应用”“我的工作台”“Chatflow”“聊天助手”“Agent”“文本生成”等导航栏。中间部分有“创建应用”“创建自定义Agent”“从应用模板创建”“导入DSL文件”等操作选项。页面底部有“参与社区”提示，鼓励开发者在不同渠道交流。该页面是Dify平台在完成相关环境搭建和运行后，首次登录注册后可进入的主界面。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/Hynobr9WmonZHBxB8N3ckLtUnDe.png)

## 接入大模型

接入的大模型可以有：

- Ollama
- 通义千问
- 深度求索
- 智普AI



- 点击用户名，选择设置

![这是Dify Workspace的操作界面，界面右上角有带数字“2”标记的用户头像下拉入口，点击后展开下拉菜单，菜单内包含“设置”选项，还有“帮助文档”“社区”“主题”“退出”等其他选项。该界面对应Dify环境搭建中接入大模型的相关操作步骤，提示用户点击用户名后选择设置，是接入大模型流程里的前期操作指引。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/JtCsb5pPvoZRFOxiDg6cmKqrn6f.png)



- 接入大模型

![这张图片展示的是Dify平台的设置页面，左侧导航栏中“模型供应商”选项被红色箭头清晰标记，处于突出选中状态。页面右侧为模型供应商相关内容，上方有搜索框，下方以列表形式呈现了可安装的模型供应商，包括OpenAI、Anthropic、Amazon Bedrock、Azure OpenAI等多个选项，页面标注提示可前往Dify市场安装更多模型供应商，这和文档中“接入大模型时需选择模型供应商并搜索安装”的操作步骤相对应。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/VTvXbHZJfod3drxDuZxc1kIdnCu.png)



选择模型供应商，然后搜索安装模型供应商

然后点击**通义千问**，进行安装

![图片展示的是Dify平台接入大模型时选择模型供应商的界面。左侧为设置菜单，有模型供应商、成员、数据来源、API扩展、访问等选项。右侧是模型供应商列表，通义千问被红框突出显示，其下方有“安装”按钮。该图片与上下文紧密相关，上下文提到在选择模型供应商后搜索安装模型供应商，然后点击“通义千问”进行安装，此图直观呈现了点击“通义千问”后的界面情况，辅助说明安装操作。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/GrqobV0N0o5yChxdjwOc4u8Pnyc.png)

> 如果安装失败，可安装几次，即可安装成功

在市场中可以搜索历史版本进行安装

![图片展示的是Dify平台中通义千问模型的版本历史界面。界面上方显示“通义千问”及版本号0.1.23。下方列出多个版本信息，包括版本号、更新时间、发布者及操作选项。其中，0.1.16版本被红色框突出显示，更新于一个月前，发布者为langgenius，操作选项为“下载”。该图片与文档中“在市场中可以搜索历史版本进行安装”内容相关，直观呈现了历史版本的查找方式。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/RUhjbtu1ooFdJsx3YS8cV3J3nif.png)



![这是Dify环境搭建中接入通义千问大模型步骤的界面截图，弹窗显示“安装成功”，配有“插件已成功安装”的提示内容，弹窗内展示了已安装的通义千问（langgenius/tongyi，版本0.1.1），界面旁的红色箭头指向该安装成功的提示区域，对应文档中“点击通义千问进行安装”后完成安装的环节，明确呈现了通义千问模型安装成功的状态。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/OtcgbpXc0opHSnxzxXkc4q2ynIB.png)



- 安装以后，到待配置区

![图片展示的是Dify平台接入大模型后的待配置区界面。左侧为设置菜单，选中“模型供应商”。右侧“模型列表”为空，“待配置”区域显示通义千问模型，其下方有“未配置”标识及“去配置”按钮，按钮旁有红色箭头突出显示。该图片与上下文关系紧密，上下文提到安装通义千问模型后，需点击“去配置”进行后续设置操作，此图直观呈现了该操作位置。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/F1bObCYo2oKwNyxyTCjcpeW6nod.png)

- 点击设置

![这张图片展示的是Dify平台中“API密钥授权配置”的弹窗界面，是接入大模型环节的设置页面。弹窗内的凭据名称已填写为“通义千问”，API Key输入框内的内容被红色箭头标注为“百炼的秘钥”，对应上下文里说明的需从阿里百炼平台获取API Key的要求，弹窗下方还设有“使用国际端点”的选项，以及保存、取消的操作按钮，界面底部还显示有其他模型供应商如Amazon Bedrock、Azure OpenAI的相关标识。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/P7cNb2AVGo3R3Txu8rtcL0BCnYf.png)

凭据名称和API Key从阿里百炼平台获取

![图片展示的是Dify平台接入大模型后的模型供应商界面。左侧为设置菜单，选中“模型供应商”。右侧显示模型列表，通义千问模型供应商下有多个模型，如“qwen - max - latest”等，部分模型名称前有绿色点标识，说明已选通。下方还有“DeepSeek”等模型。该图与上下文关系紧密，直观呈现了在Dify平台接入大模型后，可选择的模型供应商及可用模型情况。](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/01-coze-dify-ji-chu/Ooc2b7NIzonABoxKux3cn64mnkg.png)

---
> 来源：飞书技术知识库（脱敏版） · 06 AI 应用开发 / 低代码智能体平台（Coze / Dify） / 平台智能体 / 01-Coze+Dify基础 / Dify环境搭建 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/SebPwNNBxin822kvCncc5L2FnUe)
