---
type: memo
date: 2026-11-06
aliases:
  - "Jenkins 持续集成（扩展文档）"
---

# Jenkins 持续集成（扩展文档）

## Jenkins简介

Jenkins，原名Hudson，2011年改为现在的名字，它是一个开源的实现持续集成的软件工具。官方网站：https://www.jenkins.io/zh/

Jenkins 能实时监控集成中存在的错误，提供详细的日志文件和提醒功能，还能用图表的形式形象地展示项目构建的趋势和稳定性。

![图片展示了Jenkins的官网页面。上方有Jenkins的标志及“编码强大，无所不能”的标语，介绍其为开源CI/CD软件领导者，提供超1000个插件支持持续集成、部署、自动化等。下方有“文档”和“下载”按钮。红色区域突出显示“Jenkins的新平台”及“CD.FOUNDATION”，表明Jenkins加入了新成立的持续交付基金会（CDF）。该图片与文档中对Jenkins简介的介绍相呼应，直观呈现了Jenkins的品牌形象及新平台信息。](../../../../Attachments/development/devops/ci-cd-jenkins/JLK5bZE6AoFAgFxshKbctKeynNf.png)

 特点：

- **易配置**：提供友好的GUI配置界面；
- **变更支持**：Jenkins能从代码仓库（Subversion/CVS）中获取并产生代码更新列表并输出到编译输出信息中；支持永久链接：用户是通过web来访问Jenkins的，而这些web页面的链接地址都是永久链接地址，因此，你可以在各种文档中直接使用该链接；
- **集成E-Mail/RSS/IM**：当完成一次集成时，可通过这些工具实时告诉你集成结果（构建一次集成需要花费一定时间，有了这个功能，你就可以在等待结果过程中，干别的事情）；
- **JUnit/TestNG**测试报告：也就是用以图表等形式提供详细的测试报表功能；
- **支持分布式构建**：Jenkins可以把集成构建等工作分发到多台计算机中完成；文件指纹信息：Jenkins会保存哪次集成构建产生了哪些jars文件，哪一次集成构建使用了哪个版本的jars文件等构建记录；
- **支持第三方插件**：使得 Jenkins 变得越来越强大

## Jenkins安装

### 基于docker-compose 构建

```YAML
version: '3'
services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    ports:
      - "8000:8080"
    environment:
      - JAVA_OPTS=-Duser.timezone=GMT+08 -Xms2g -Xmx2g
    volumes:
      - /opt/docker/jenkins/data:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker
    user: root
```

云服务器需要放行端口：8000

使用 `docker-compose up -d ` 运行容器

浏览器访问：**http://yourIp:8000，如下：**

![这是Jenkins安装后首次访问的解锁页面，用于获取初始管理员密码以完成初始化。页面顶部标注了“入门”与“解锁Jenkins”的标题，明确提示需从服务器的`/var/jenkins_home/secrets/initialAdminPassword`路径获取随机生成的初始密码，该路径与文档中提示的密码查看路径完全对应。页面中部设有“管理员密码”输入框，供用户粘贴获取到的密码，右下角配有蓝色的“提交”按钮，完成密码输入后可继续后续的Jenkins初始化流程。](../../../../Attachments/development/devops/ci-cd-jenkins/IRvcbwkq0onnwkxIH5qcemdbnub.png)

注意默认创建的 Jenkins 密码系统会生成随机密码：所在目录 `/var/jenkins_home/secrets/initialAdminPassword`

可以使用 ` cat /var/jenkins_home/secrets/initialAdminPassword 查看` 

或者 直接使用 docker 查看日志：

```Bash
[root@iZbp1e6qrh9y5n86a734doZ docker]# docker logs -f jenkins
Running from: /usr/share/jenkins/jenkins.war
webroot: /var/jenkins_home/war
2024-03-10 02:27:40.232+0000 [id=1]     INFO    winstone.Logger#logInternal: Beginning extraction from war file
2024-03-10 02:27:41.461+0000 [id=1]     WARNING o.e.j.s.handler.ContextHandler#setContextPath: Empty contextPath
2024-03-10 02:27:41.607+0000 [id=1]     INFO    org.eclipse.jetty.server.Server#doStart: jetty-10.0.13; built: 2022-12-07T20:13:20.134Z; git: 1c2636ea05c0ca8de1ffd6ca7f3a98ac084c766d; jvm 11.0.19+7
2024-03-10 02:27:42.067+0000 [id=1]     INFO    o.e.j.w.StandardDescriptorProcessor#visitServlet: NO JSP Support for /, did not find org.eclipse.jetty.jsp.JettyJspServlet
2024-03-10 02:27:42.168+0000 [id=1]     INFO    o.e.j.s.s.DefaultSessionIdManager#doStart: Session workerName=node0
2024-03-10 02:27:42.730+0000 [id=1]     INFO    hudson.WebAppMain#contextInitialized: Jenkins home directory: /var/jenkins_home found at: EnvVars.masterEnvVars.get("JENKINS_HOME")
2024-03-10 02:27:42.926+0000 [id=1]     INFO    o.e.j.s.handler.ContextHandler#doStart: Started w.@1835d3ed{Jenkins v2.387.3,/,file:///var/jenkins_home/war/,AVAILABLE}{/var/jenkins_home/war}
2024-03-10 02:27:42.954+0000 [id=1]     INFO    o.e.j.server.AbstractConnector#doStart: Started ServerConnector@3f390d63{HTTP/1.1, (http/1.1)}{0.0.0.0:8080}
2024-03-10 02:27:42.983+0000 [id=1]     INFO    org.eclipse.jetty.server.Server#doStart: Started Server@66b7550d{STARTING}[10.0.13,sto=0] @3348ms
2024-03-10 02:27:42.991+0000 [id=23]    INFO    winstone.Logger#logInternal: Winstone Servlet Engine running: controlPort=disabled
2024-03-10 02:27:43.281+0000 [id=30]    INFO    jenkins.InitReactorRunner$1#onAttained: Started initialization
2024-03-10 02:27:43.301+0000 [id=31]    INFO    jenkins.InitReactorRunner$1#onAttained: Listed all plugins
2024-03-10 02:27:44.364+0000 [id=29]    INFO    jenkins.InitReactorRunner$1#onAttained: Prepared all plugins
2024-03-10 02:27:44.559+0000 [id=31]    INFO    jenkins.InitReactorRunner$1#onAttained: Started all plugins
2024-03-10 02:27:44.669+0000 [id=29]    INFO    jenkins.InitReactorRunner$1#onAttained: Augmented all extensions
2024-03-10 02:27:44.968+0000 [id=30]    INFO    jenkins.InitReactorRunner$1#onAttained: System config loaded
2024-03-10 02:27:44.968+0000 [id=28]    INFO    jenkins.InitReactorRunner$1#onAttained: System config adapted
2024-03-10 02:27:44.969+0000 [id=28]    INFO    jenkins.InitReactorRunner$1#onAttained: Loaded all jobs
2024-03-10 02:27:44.970+0000 [id=31]    INFO    jenkins.InitReactorRunner$1#onAttained: Configuration for all jobs updated
2024-03-10 02:27:45.059+0000 [id=44]    INFO    hudson.util.Retrier#start: Attempt #1 to do the action check updates server
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by org.codehaus.groovy.vmplugin.v7.Java7$1 (file:/var/jenkins_home/war/WEB-INF/lib/groovy-all-2.4.21.jar) to constructor java.lang.invoke.MethodHandles$Lookup(java.lang.Class,int)
WARNING: Please consider reporting this to the maintainers of org.codehaus.groovy.vmplugin.v7.Java7$1
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
2024-03-10 02:27:45.671+0000 [id=28]    INFO    jenkins.install.SetupWizard#init: 

*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

c9c29f1dfa2a47849b7272dd82d855a3

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************

2024-03-10 02:28:10.198+0000 [id=28]    INFO    jenkins.InitReactorRunner$1#onAttained: Completed initialization
2024-03-10 02:28:10.228+0000 [id=22]    INFO    hudson.lifecycle.Lifecycle#onReady: Jenkins is fully up and running
2024-03-10 02:28:11.406+0000 [id=44]    INFO    h.m.DownloadService$Downloadable#load: Obtained the updated data file for hudson.tasks.Maven.MavenInstaller
2024-03-10 02:28:11.407+0000 [id=44]    INFO    hudson.util.Retrier#start: Performed the action check updates server successfully at the attempt #1

```

输入密码后等待初始化。

### 基础插件安装

1）选择 推荐安装 插件

![图片展示的是Jenkins安装时的“自定义Jenkins”界面。界面上方有“新手入门”提示，下方有“自定义Jenkins”标题及说明。左侧有“安装推荐的插件”选项，其下说明为“安装Jenkins社区推荐的插件”，该选项被红色框线突出显示。右侧是“选择插件来安装”选项，提示选择并安装最适合的插件。此图对应文档中“基础插件安装”部分，用于指导用户选择安装Jenkins推荐的插件。](../../../../Attachments/development/devops/ci-cd-jenkins/Nk86bWnVxo5h7UxqSIkcDjmYnnf.png)

开始安装：

![图片展示的是Jenkins 2.387.3版本的“新手入门”页面。页面上方标题为“新手入门”，下方以表格形式列出多个插件选项，如Folders、Timestamper、Pipeline、Git、LDAP等，每个选项前有圆圈标识。表格右侧有“Build Timeout”“OWASP Markup Formatter”等选项。页面底部标注“** - 需要依赖”。该图片与文档中“开始安装”部分对应，直观呈现了Jenkins安装时可选择的插件选项，帮助用户了解安装时的配置情况。](../../../../Attachments/development/devops/ci-cd-jenkins/XdsVbL3wponbIoxRtDecAB0pnh8.png)

> 可能因为网络会安装失败，我们课程主要使用git、pipeline 和 chinese中文插件，可以直接继续忽略错误插件，或者点击重试尝试重新安装:
> 
> ![图片展示的是Jenkins安装过程中部分插件安装失败的界面。界面标题为“安装失败”，提示部分插件安装失败，可重试或继续。下方以表格形式列出已安装和未安装的插件，其中“Timestamper”“SSH Build Agents”“Git plugin”“LDAP”未安装，以红色背景标识；“Pipeline”“Pipeline: Stage View”“Localization: Chinese (Simplified)”已安装，以绿色背景标识。该图与文档中“基础插件安装”部分上下文对应，直观呈现了安装失败情况。](../../../../Attachments/development/devops/ci-cd-jenkins/ByVAbxXb5o2zsCxKTcnc8yTpnbe.png)





2）设置管理员密码

![图片展示的是Jenkins安装过程中设置管理员密码的界面。界面上方标题为“创建第一个管理员用户”，下方有用户名、密码、确认密码、全名、电子邮件地址等输入框，分别对应“jenkins”“jenkins”“jenkins@it.com”等信息。右下角有“使用admin账户继续”和“保存并完成”两个按钮。该图片与文档中“设置管理员密码”步骤对应，是设置Jenkins管理员账户信息的界面展示。](../../../../Attachments/development/devops/ci-cd-jenkins/WW4sbfO11oG81lxFMhHcK3CDnbe.png)

保存设置：

![图片展示的是Jenkins实例配置界面，核心内容为Jenkins URL输入框。该框内显示访问Jenkins的地址为“http://120.26.160.69:58888/”，并有红色箭头指向。下方有相关说明，指出Jenkins URL用于提供绝对路径链接的根地址，推荐值是根据当前请求生成的，最佳实践是设置该值。图片与上下文紧密相关，是基于docker-compose构建Jenkins安装流程中设置管理员密码步骤后的保存设置界面。](../../../../Attachments/development/devops/ci-cd-jenkins/P1yMbt7BvofM7nxJUejc440Qnf2.png)

安装成功开始使用：

![这张图片展示的是Jenkins安装完成后的初始页面，页面顶部标注“新手入门”，核心内容显示“Jenkins已就绪！”，下方配有说明文字“Jenkins安装已完成”，还有一个蓝色的“开始使用Jenkins”按钮。页面底部标注了Jenkins的版本号为2.387.3，该页面位于Jenkins安装流程中基础插件安装完成后的环节，标志着Jenkins安装步骤结束，可通过点击按钮进入Jenkins管理界面。](../../../../Attachments/development/devops/ci-cd-jenkins/U0RsbCx6uoE8kAxMgUCcWfuenZc.png)

进入到Jenkins管理界面：

![图片展示的是Jenkins管理界面。界面上方显示“Jenkins”标题，左侧有“新建Item”“用户列表”“构建历史”“Manage Jenkins”“My Views”等导航栏。中间区域有“欢迎来到Jenkins!”提示，下方有“Start building your software project”“Set up a distributed build”“Set up an agent”“Configure a cloud”“Learn more about distributed builds”等操作选项。该界面是Jenkins安装成功后的管理界面，用户可在此开始使用Jenkins进行软件项目构建等相关操作。](../../../../Attachments/development/devops/ci-cd-jenkins/VLR5bIGNsoycIZx8Lwac6vMqnmg.png)

核心插件：**git、git Paramter、maven integretion 、maven pipeline** 等

### git配置

系统管理  --> 全局工具配置

![图片展示的是Jenkins系统管理中全局工具配置下的Git安装界面。界面中有“Git”名称输入框，已输入“git”；“Path to Git executable”路径输入框，显示为“/usr/bin/git”。该图片与文档中“git配置”部分内容相关，对应系统管理-->全局工具配置步骤，用于在Jenkins中配置Git工具，确保后续能正常使用Git插件进行相关操作。](../../../../Attachments/development/devops/ci-cd-jenkins/V78lbq2OdowGXZxSuOccPy0Untf.png)



以下相关插件需要安装：

![图片展示的是Jenkins插件管理页面。页面上方有“Updates”“Available plugins”“Installed plugins”“Advanced settings”选项卡，当前选中“Available plugins”。下方搜索框内提示“Search available plugins”。列表中显示了多个插件，如“Git Parameter”“List Git Branches Parameter”“GitLab Branch Source”“Extended Choice Parameter”等，部分插件有“Warning: This plugin version may not be safe to use”提示，显示了插件名称、版本号及发布日期。页面底部有“直接安装”“下载待重启安装”“立即移除”按钮。该图与文档中“基础插件安装”部分相关，用于说明可在此页面安装Jenkins基础插件。](../../../../Attachments/development/devops/ci-cd-jenkins/IcMRbncJpoEbAHxvQi7cmmzgnMb.png)

![这张图片展示了Jenkins插件管理页面搜索“ssh”后的结果，搜索框在页面上方，输入了关键词“ssh”。页面中列出了多个相关插件，其中核心内容为被勾选的“Publish Over SSH 1.25”插件，该插件的说明信息为“Send build artifacts over SSH”，图中还标注了该插件的作用，是解决Jenkins和应用服务器不在同一台时使用的插件，可实现Jenkins远程部署服务到其他服务器。](../../../../Attachments/development/devops/ci-cd-jenkins/AEZPbHsedod68XxVpB1c9JMwnTw.png)

### JDK配置

进入到容器：

```Bash
docker exec -it jenkins /bin/bash


进入到容器内查看jdk位置：
root@d0b595add37c:/# cd /opt/
root@d0b595add37c:/opt# ls
java  jenkins-plugin-manager.jar
root@d0b595add37c:/opt# cd java
root@d0b595add37c:/opt/java# ls
openjdk
root@d0b595add37c:/opt/java# cd openjdk/
root@d0b595add37c:/opt/java/openjdk# ls
bin  conf  include  legal  lib  release
root@d0b595add37c:/opt/java/openjdk# pwd
/opt/java/openjdk
root@d0b595add37c:/opt/java/openjdk# java -version
openjdk version "11.0.19" 2023-04-18
OpenJDK Runtime Environment Temurin-11.0.19+7 (build 11.0.19+7)
OpenJDK 64-Bit Server VM Temurin-11.0.19+7 (build 11.0.19+7, mixed mode)
root@d0b595add37c:/opt/java/openjdk#
```

在maven 全局工具中配置JDK：

![图片展示的是Jenkins系统管理下的全局工具配置中JDK安装界面。界面上方显示“JDK安装”及“系统下JDK安装列表”，下方有“新增JDK”按钮。下方有“JDK别名”输入框，已输入“jdk11”，“JAVA_HOME”输入框显示“/opt/java/openjdk”，还有“自动安装”选项。该图片与文档中“JDK配置”部分内容相关，对应在容器中配置JDK的操作步骤，直观呈现了配置界面及输入内容。](../../../../Attachments/development/devops/ci-cd-jenkins/Le7pbvGBloQxnUxOpbgc85qDnoc.png)

2.5 maven配置

系统管理  --> 全局工具配置

![图片展示的是Jenkins系统管理中全局工具配置下的Maven配置界面。在“Maven安装”下，有“系统下Maven安装列表”和“新增Maven”按钮。下方弹出的“新增Maven”窗口中，Maven Name输入框内显示“maven”，MAVEN_HOME路径为“/var/jenkins_home/apache-maven-3.8.8”。该图片与文档中“JDK配置”部分内容相关，对应在maven全局工具中配置JDK的操作步骤，直观呈现了配置界面及输入项。](../../../../Attachments/development/devops/ci-cd-jenkins/DjYEbuzeHoa5Vgx9mBUcyZxCnMh.png)



点击Manage Jenkins --> 系统管理 --> 点击插件管理 --> 进入到插件管理页面

![图片展示的是Jenkins管理界面中的“Manage Jenkins”页面。页面左侧有“Manage Jenkins”选项被红色框突出显示。右侧“System Configuration”区域中，“Manage Plugins”选项也被红色框突出显示，其功能为添加、移除、禁用或启用插件，以扩展Jenkins的功能。该图片与文档中“基础插件安装”部分相关，对应“点击Manage Jenkins-->系统管理-->点击插件管理-->进入到插件管理页面”这一操作步骤，直观呈现了操作后的页面情况。](../../../../Attachments/development/devops/ci-cd-jenkins/SDVkbak0lopwnBxUPuYcrTQlnLb.png)

输入 maven  安装：

![图片展示的是Jenkins插件管理页面。页面左侧导航栏中“Available plugins”被红色框突出显示。右侧搜索框内输入“maven”，下方列出多个与maven相关的插件，其中“Maven Integration”被蓝色框突出显示，其下方有“直接安装”和“下载并重新安装”两个按钮。该图片与文档中“JDK配置”部分相关，用于指导用户在Jenkins中搜索并安装maven插件，以配置JDK。](../../../../Attachments/development/devops/ci-cd-jenkins/UD4cbwAZOoRMZcxq9Dbc49W7nud.png)

空闲时重启:

![这张图片是Jenkins基础插件安装完成后的设置界面对话，左侧列出了Javadoc、JSch dependency、Maven Integration等插件，右侧标注了部分插件的安装状态，Javadoc和JSch dependency显示“Downloaded Successfully. Will be activated during the next boot”，其余插件显示“等待”。界面中存在两个操作选项，一个是返回首页使用已安装插件的选项，另一个是被红色框突出标注的“安装完成后重启Jenkins(空闲时)”选项，该选项已被勾选，这对应了文档中安装Jenkins插件后需进行的后续操作要求。](../../../../Attachments/development/devops/ci-cd-jenkins/SYEwbBdXsoxZ9LxLo3NcVLHTn3d.png)

---
> 来源：飞书技术知识库（脱敏版） · 05 工程化与部署 / CI/CD：Jenkins / Jenkins 持续集成（扩展文档） · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/ICFWwqFZlioiNrkr8DfcMUVOnFh)
