---
type: memo
date: 2026-11-06
aliases:
  - "第十二章 项目总结&部署"
---
# 第十二章 项目总结&部署

## 1 项目总结

参考：项目梳理思维导图。



## 2 项目部署

### 2.1 名词解释

#### 什么是DevOps

一个软件的生命周期包括：需求分析阶、设计、开发、测试、上线、维护、升级、废弃。

通过示例说明如下：

1、产品人员进行需求分析

2、设计人员进行软件架构设计和模块设计。

3、每个模块的开发人员并行开发，设计接口、进行编码，并进行单元测试

4、开发完毕，将代码集成部署到测试服务器，测试人员进行测试。

5、测试人员发现bug，提交bug、开发人员修改bug

6、bug修改完毕再次集成、测试。

7、测试完毕，项目上线。

8、运维人员进行安装部署、培训。

9、用户提出问题，返回给运维人员。

10、运维人员反馈给开发人员，开发人员进行问题处理。

11、再次提交测试。

12、测试完毕再次部署升级。

....

最后软件下线。

所以，在整体生命周期中比较核心的两个阶段是：开发阶段、维护阶段，开发阶段的成果是软件开发完成并成功上线，运维阶段则负责对软件进行维护和升级，而运维阶段通常在一个软件 的生命周期中占比最多。

提高开发阶段、运维阶段的工作效率是企业在进行软件项目管理的重点。

因此，专家提出了DevOps，DevOps是什么呢？

下边是摘自百度百科的定义：

DevOps（Development和Operations的组合词）是一组过程、方法与系统的统称，用于促进开发（[应用程序](https://baike.baidu.com/item/%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F/5985445?fromModule=lemma_inlink)/软件工程）、技术运营和质量保障（[QA](https://baike.baidu.com/item/QA/476016?fromModule=lemma_inlink)）部门之间的沟通、协作与整合。

它是一种重视“[软件开发](https://baike.baidu.com/item/%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/3448966?fromModule=lemma_inlink)人员（Dev）”和“[IT运维](https://baike.baidu.com/item/IT%E8%BF%90%E7%BB%B4/5727814?fromModule=lemma_inlink)技术人员（Ops）”之间沟通合作的文化、运动或惯例。透过自动化“[软件交付](https://baike.baidu.com/item/%E8%BD%AF%E4%BB%B6%E4%BA%A4%E4%BB%98/638479?fromModule=lemma_inlink)”和“架构变更”的流程，来使得构建、测试、发布软件能够更加地快捷、频繁和可靠。

它的出现是由于软件行业日益清晰地认识到：为了按时交付[软件产品](https://baike.baidu.com/item/%E8%BD%AF%E4%BB%B6%E4%BA%A7%E5%93%81/6800028?fromModule=lemma_inlink)和服务，开发和运维工作必须紧密合作。

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/Loqab71PxovazhxmSvgcvxK5ngg.png)
> 图注：这张图是三个圆形交叉的维恩图，用于解释DevOps的定义，对应文档中关于DevOps的名词解释内容。三个圆形分别代表软件开发工程、质量保证以及技术运维，三个圆形的交集区域标注为DevOps，清晰呈现出DevOps是开发、质量保障与运维三个领域交叉融合后形成的概念，和文档中说明DevOps是开发与运维紧密结合产物的内容相呼应。

DevOps是一个工具吗？

DevOps是一个工作职位吗？

都不是。

DevOps是一种思想理念，它涵盖开发、测试、运维的整个过程。DevOps追求的目标是提高软件开发、测试、运维、运营等各部门的沟通与协作质量，DevOps强调软件开发人员与软件测试、软件运维、质量保障（QA）部门之间有效的沟通与协作，强调通过自动化的方法去管理软件变更、软件集成，使软件从构建到测试、发布更加快捷、可靠，最终按时交付软件。

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/UAYAb7KqJocayfxmiq0caTwOnAf.png)
> 图注：图片展示了DevOps理念下的开发与运维流程。左侧以“Dev”命名，包含build（构建）、code（代码）、plan（计划）、test（测试）；右侧以“Ops”命名，包含deploy（部署）、operate（运维）、monitor（监控）。箭头从build指向code，再指向plan，最后指向test，形成开发流程；从deploy指向operate，再指向monitor，形成运维流程。箭头间有“release（发布）”连接，表明开发与运维的紧密协作。此图直观呈现了DevOps中开发与运维的循环与交互关系。

#### 什么是CI/CD

如何来落地实现DevOps呢？

DevOps兴起于2009年，近年来由于云计算、互联网的发展，促进了DevOps的基础设施及工具链的发展，涌现了一大批优秀的工具，这些工具包括开发、测试、运维的各各领域，例如：GitHub、Docker、Jenkins、Hudson、K8S、Ant/Maven/Gradle、Selenium、QUnit、JMeter等。下图是DevOps相关的工具集：

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/EZoPbukQcopRvFxauutcd6Xrnth.png)
> 图注：这张图是DevOps相关的工具集，按流程划分成五个核心环节：Collaborate、Build、Test、Deploy、Run。每个环节内分布着对应场景的各类工具，例如Collaborate环节包含Application Lifecycle Mgmt、Slack、ChatOps等工具；Build环节涵盖git、GitHub、Bitbucket等代码构建相关工具；Test环节有JUnit、Selenium、QUnit等测试工具；Deploy环节包含rundeck、juju等部署工具；Run环节则涵盖Cloud IaaS/PaaS、Orchestration & Scheduling等运维运行相关工具，集中展示了DevOps各阶段的配套工具集合。

好的工具有利于DevOps的实施，但并不代表实施DevOps就一定需要去引入一堆工具。

问题的关键：如何解决问题，而不是具体应用工具。

CI/CD 是近年来企业有效实施DevOps的具体方案。

CI/CD 包含了一个 CI 和两个 CD，CI全称 Continuous Integration，表示持续集成，CD包含 Continuous Delivery和 Continuous Deployment，分别是持续交付和持续部署，三者具有前后依赖关系。

CI 持续集成：

持续集成倡导团队成员需要频繁的集成他们的工作，将开发分支合并到主分支，每次集成都通过自动化构建（包括编译、构建、自动化测试）来验证，从而尽快地发现集成中的错误，让产品可以快速迭代，同时还能保持高质量。

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/LVvGbefdLoZlTCxzCJNcOMzZnGh.png)
> 图注：图片展示了CI/CD（持续集成/持续交付）流程。两名开发者在电脑上进行代码提交（CHECK-IN），代码被提交到源代码仓库（SOURCE REPOSITORY）。随后，代码被CI服务器（CI SERVER）接收，进行构建（BUILD）、测试（TEST），并得到结果（RESULT）。该图与上下文紧密相关，直观呈现了文档中对CI/CD概念的解释，即团队成员频繁集成工作，每次集成通过自动化构建和测试来验证，以发现错误并保持高质量。

CD持续交付:

持续交付将集成后的代码部署到类生产环境(预发布)，除了交付到类生产环境之外，还会执行一些集成测试、API测试。持续交付强调的是“交付”，交付给测试、产品验收，不管怎么更新，软件是随时随地可以交付的。

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/HoOHbTGTAoZXcQxOdxJclIvOnBf.png)
> 图注：这张图展示了CI/CD的工作流程，流程起始是开发人员将代码通过CHECK-IN操作提交至SOURCE REPOSITORY源代码仓库，随后代码被送入CI SERVER进行BUILD、TEST并生成RESULT；CI服务器的测试结果会返回给开发人员，完成反馈环节，后续流程依次进入TEST测试、STAGING预发布、PRODUCTION生产环境，生产环境的部署标注为MANUAL手动操作环节，完整呈现了从代码提交到部署上线的CI/CD核心流程。

CD持续部署：

在持续交付的基础上由开发人员或运维人员自助式的定期向生产环境部署稳定的构建版本，持续部署的目标是代码在任何时刻都是可部署的，并可自动进入到生产环境。

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/BESgbbBaXo4OJxxYmUqcRb5Dntg.png)
> 图注：图片展示了完整的CI/CD方案流程。开发人员将代码提交到源代码仓库，CI服务器进行构建、测试，结果反馈。测试通过后，代码依次进入Staging和Production环境，其中Production环境可自动部署。该图与上下文紧密相关，直观呈现了文档中对CI/CD概念的解释，即持续集成和持续部署的完整流程，帮助理解持续交付和持续部署的具体操作环节。



完整的CI/CD方案：

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/KlEhbNR8OoaeTPx1SPKc7Q7Qnlc.png)
> 图注：这张图是完整的CI/CD方案架构图，展示了开发人员、测试人员与项目经理三类角色在CI/CD流程中的参与及环境分工。开发人员通过SVN/Git拉取代码，经由Jenkins pipeline的编译集成，包含编译服务器、Docker打包镜像发布，再通过Kubernetes API完成开发测试环境的部署，同时进行监控。测试人员则参与环境部署的Jenkins pipeline，可拉取Harbor镜像仓库中的镜像，通过Kubernetes API滚动部署，对测试环境做监控及日志分析。项目经理可审核部署流程，最终完成生产环境的相关部署环节，清晰呈现了CI/CD从代码拉取到多环境部署的完整链路。

### 2.2 使用Kubernetes部署

#### 2.2.1 K8s介绍

##### K8s介绍

[Kubernetes](https://kubernetes.io/zh-cn/docs/concepts/overview/) 也称为 K8s（k和s中间有8个字母），**是用于自动部署、扩缩和管理容器化应用程序的开源系统**。它将组成应用程序的容器组合成逻辑单元，以便于管理和服务发现。Kubernetes 源自[Google 15 年生产环境的运维经验](http://queue.acm.org/detail.cfm?id=2898444)，同时凝聚了社区的最佳创意和实践。

简单的说k8s就是用来管理Docker的，当Docker容器过多如果手工部署增加了运维成本，用k8s它解决了许多复杂的问题。如自动部署，限制资源，自动重启、迁移故障，简化应用的访问等。

官网地址：https://kubernetes.io/zh-cn/



##### 组成结构

k8s是主从结构，`master`管理节点，与`node`工作节点相对。node干活master管理。

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/Tr8Eb3nT7o3fTsx6igjcbZ60nSe.png)
> 图注：图片展示了Kubernetes的组成结构。左侧为Master节点，包含API Server、Scheduler、Controller Manager、etcd等组件，其中API Server是操作入口，Scheduler负责调度，Controller Manager控制管理，etcd为分布式可监控存储。右侧为Node节点，部署有kubelet、proxy等组件，kubelet负责监控、维护容器，proxy负责负载、服务发现。Node节点上运行着pod，pod内有container。

**其中master包含组件：**

`api server` 操作入口。Kubernetes的所有操作指令由对此下达。

`etcd` 集群内部用键值数据库。

`scheduler`调度器。调度工作的。

`controller`控制器。发布实际的指令。

**node 中包含组件：**

`pod `几个相关容器（docker),它们被统一管理。Kubernetes的基本管理单位。

`kubelet`代理。master节点命令的实际执行者。

`kube-proxy`网络代理，网络入口和出口。

`kubectl `命令行管理工具。

`volume `数据卷，保存数据。

基本概念：

`deployment `部署。指在节点部署的pod。

`service`服务，组合多个部署的pod,提供对外访问。

`label `标签，`selector `选择器，如给节点打上某标签，可以在部署应用时，只部署在包含标签的节点上。

`namespace `命名空间，做逻辑隔离。

`kubeadm `集群部署工具。

`Ingress`应用网关，统一出口、负载均衡。



#### 2.2.2 K8s部署流程

##### 选型

[KubeSphere](https://kubesphere.io/) 是在 [Kubernetes](https://kubernetes.io/) 之上构建的面向云原生应用的**分布式操作系统**，完全开源，支持多云与多集群管理，提供全栈的 IT 自动化运维能力，简化企业的 DevOps 工作流。它的架构可以非常方便地使第三方应用与云原生生态组件进行即插即用 (plug-and-play) 的集成。https://kubesphere.io/zh/ 

KubeSphere 作为开源的企业级全栈化容器平台，为用户提供了一个健壮、安全、功能丰富、具备极致体验的 Web 控制台。拥有企业级 Kubernetes 所需的最常见的功能，如工作负载管理，网络策略配置，微服务治理（基于 Istio），DevOps 项目 (CI/CD) ，安全管理，Source to Image/Binary to Image，多租户管理，多维度监控，日志查询和收集，告警通知，审计，应用程序管理和镜像管理、应用配置密钥管理等功能模块。

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/Ao2Bbjd52oBOjqxtC6rcz72lnXZ.png)
> 图注：图片展示了KubeSphere容器平台的部署流程。ISV上传Helm应用，Infrastructure Operator安装K8s和KubeSphere集群，Developer/User开发和部署容器化应用，Admin/Operator管理KubeSphere以维护K8s集群。KubeSphere Container Platform包含KubeSphere Console和Kubernetes，通过KubeSphere Console可进行应用开发、部署、运维等操作，Kubernetes负责容器化应用的运行。平台可将服务暴露给互联网上的Web应用、游戏应用等。

KubeSphere 提供了基于 Jenkins 的可视化 CI/CD 流水线编辑，无需对 Jenkins 进行配置，同时内置丰富的 CI/CD 流水线插件，包括Binary-to-Image (B2I) 和Source-to-Image (S2I)，用于将源代码或二进制文件打包到准备运行的容器映像中。

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/GaPGbaG4ZoqxHrxGRd6ciwK4nUh.png)
> 图注：图片展示了KubeSphere Container Platform的部署流程。从外部工具JIRA + Confluence开始，开发者通过Source Code进入流程。流程包含Code Plan、Code Build、Code Test、Code Release、Code Monitor等环节，涉及静态代码分析、单元测试等。最终，Build and Push Image、Deploy to Kubernetes、Monitor and Operate等步骤完成部署，生成Build Report、Artifact等。图片与上下文紧密相关，直观呈现了KubeSphere部署过程中各环节及工具的关联。

##### 简图

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/VYdxbVvViopTDWxGeWcc7QngnAd.png)
> 图注：图片展示了Kubernetes部署流程。左侧为Maven流程，从Dockerfile构建镜像，再推送镜像。右侧是Kubernetes部署流程，YAML文件创建deployment并暴露service，Docker推送镜像至节点，节点运行pod APP。

##### 部署KubeSphere

每个节点(最少三个节点，一主两从)安装Docker

安装K8s

再安装KubeSphere 



##### 创建项目

**首先创建企业空间**

KubeSphere 的多租户系统分**三个**层级，即集群、企业空间和项目。KubeSphere 中的项目等同于 Kubernetes 的[命名空间](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/namespaces/)。

创建一个新的[企业空间](https://kubesphere.io/zh/docs/v3.3/workspace-administration/what-is-workspace/)进行操作，而不是使用系统企业空间，系统企业空间中运行着系统资源，绝大部分仅供查看。出于安全考虑，强烈建议给不同的租户授予不同的权限在企业空间中进行协作。

参考：https://kubesphere.io/zh/docs/v3.3/quick-start/create-workspace-and-project/

**点击企业空间，创建一个项目**

KubeSphere 中的项目与 Kubernetes 中的命名空间相同，为资源提供了虚拟隔离。

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/BNrJbpc5TobVXox2vm2cj9hgnjg.png)
> 图注：图片展示的是KubeSphere平台的项目页面。左侧导航栏中“项目”选项被选中。页面上方显示“KUBESPHERE”及“mrt企业空间”。中间区域说明项目用于对资源进行分组管理和控制不同用户的资源管理权限。下方提示“未发现项目，请创建一个项目”，并有“创建”按钮。该图片与文档中“创建项目”部分内容对应，直观呈现了创建项目前的页面状态。

创建成功

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/QIoqbmeYEods6QxjZylcqSEEnQe.png)
> 图注：这是KubeSphere平台的项目管理页面，页面左侧显示工作台、mrt企业空间的相关导航选项，当前处于项目列表界面，列表展示了项目名称、状态、CPU用量等信息，其中已创建的“jino2o 家政服务项目”状态为Active，显示创建时间为2024-11-19 08:34:49。页面右上角有“创建”按钮，页面顶部还有提示“创建成功”的标识，该页面用于对项目进行分组管理及权限控制，与文档中“创建项目”的部署流程内容相契合，呈现了创建项目后的实际页面效果。



##### 启用devops

KubeSphere DevOps 系统

基于 [Jenkins](https://jenkins.io/) 的 KubeSphere DevOps 系统是专为 Kubernetes 中的 CI/CD 工作流设计的，它提供了一站式的解决方案，帮助开发和运维团队用非常简单的方式构建、测试和发布应用到 Kubernetes。

DevOps 系统为用户提供了一个自动化的环境，应用可以自动发布到同一个平台。它还兼容第三方私有镜像仓库（如 Harbor）和代码库（如 GitLab/GitHub/SVN/BitBucket）。它为用户提供了全面的、可视化的 CI/CD 流水线，打造了极佳的用户体验，而且这种兼容性强的流水线能力在离线环境中非常有用。

用户指南https://kubesphere.io/zh/docs/v3.3/devops-user-guide/

启用 DevOps：https://kubesphere.io/zh/docs/v3.3/pluggable-components/devops/

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/HFRrbBywVoSSoqxOkWbc2xmRnud.png)
> 图注：图片展示的是Kubernetes平台中DevOps项目的创建界面。左侧为平台管理菜单，选中“DevOps项目”。右侧上方显示“KUBESPHERE”标志。中间区域介绍DevOps项目为独立命名空间，定义流水线，用户可按需分组。下方有“如何创建DevOps？”和“流水线（Pipeline）支持哪些代码仓库类？”两个问题，底部提示“未发现DevOps项目，请创建一个DevOps项目”，并有“创建”按钮。该图与上下文介绍的K8s部署流程中创建项目的内容相关，直观呈现创建DevOps项目的操作界面。



#### 2.2.3 准备工作

##### Git仓库

使用gitee



##### 阿里云maven仓库

开启阿里云“云效”

将项目中的共享库发布到阿里云maven仓库

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/RJdcbPF8EoeAbix4OcIcrjrHn8e.png)
> 图注：图片展示的是非生产库 - snapshot的包列表界面。左侧有仓库指南、包列表、仓库动态、制品回收站等选项，当前选中包列表。右侧显示了多个包的名称、最新版本、最近更新时间、版本数和下载数等信息，如com.jzo2jzo2jzo2o-api的1.0 - SNAPSHOT版本，最近更新时间为2024 - 11 - 23 11:10:01，版本数1，下载数2等。





##### 开启阿里docker镜像服务

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/FJo3bJ7t0o7Ypoxp7DtcrVMRn1b.png)
> 图注：图片展示的是阿里云容器镜像服务中创建镜像仓库的界面。左侧为导航栏，有数据、仓库管理等选项。右侧上方有“创建镜像仓库”按钮，下方有仓库名称输入框及仓库名称提示。中间弹出“创建镜像仓库”窗口，第一步为仓库信息，可选择Codeup、GitHub等代码仓库，当前选中“本地仓库”，下方有“创建镜像仓库”和“取消”按钮。该图片与文档中开启阿里云“云效”、将项目中的共享库发布到阿里云maven仓库等内容相关，用于说明创建镜像仓库的操作步骤。

k8s在进行CI/CD时将镜像发布到阿里镜像服务中

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/JmnbbB4rFogcSGxPo0zcgUmonPc.png)
> 图注：图片展示的是阿里云容器镜像服务中个人实例的页面。左侧导航栏有“仓库管理”等选项，当前选中“镜像仓库”。页面显示“华北2（北京）”区域下，有一个名为“jzo2o - foundation”的私有仓库，状态为正常，创建于2024年11月23日15:20:36，最近更新于2024年11月23日21:13:04。页面还提供了仓库名称、命名空间、仓库状态、仓库类型、仓库地址、创建时间、最近更新时间等信息的筛选功能，以及“管理”“删除”等操作选项。



#### 2.2.3 项目配置

##### 打包配置

在pom.xml添加：

```Java
<build>
    <finalName>${project.artifactId}</finalName>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <executions>
                <execution>
                    <goals>
                        <goal>build-info</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <mainClass>com.jzo2o.foundations.FoundationsApplication</mainClass>
            </configuration>
        </plugin>
    </plugins>
</build>
```

##### 生产环境配置

每个服务通过容器部署，容器内部为8080端口，容器暴露的端口由k8s自动生成。

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/NZPRbwaguoZjXPxV0e9cskJPnOf.png)
> 图注：这张图片展示的是名为bootstrap-prod.yml的配置文件内容，该文件是Spring项目的生产环境配置文件，用于结合Nacos完成相关服务配置。文件中明确设置了server的端口为8080，同时在spring.cloud.nacos模块中，配置了Nacos服务的地址为${NACOS_ADDR}:nacos.jzo2o，还指定了命名空间为${NACOS_NAMESPACE}:ff0d1911-6b96-4eb3-9a6b-275508ddbbe6，配置文件类型为yaml，日志级别设置com.jzo2o为debug，这一配置是项目部署中生产环境配置的一部分，与文档中每个服务容器内部为8080端口、结合Kubernetes部署的内容相匹配。

##### 配置Dockerfile

![](https://feishu.cn/file/Vubqbmeo9oCpe0xgE3bcGAIAnhN)

##### 配置deploy.yml

kubersphere通过deploy.yml自动创建部署、服务。

示例：

```Java
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: jzo2o-foundation
  name: jzo2o-foundation
  namespace: jzo2o   # 就是项目名
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  selector:
    matchLabels:
      app: jzo2o-foundation #匹配metadata中定义的app
  strategy:
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: jzo2o-foundation
    spec:
      imagePullSecrets:
        - name: my-aliyun-docker  #提前在项目下配置访问阿里云的账号密码
      containers:
        - name: jzo2o-foundation
          image: registry.cn-beijing.aliyuncs.com/miaoruntu/jzo2o-foundation:SNAPSHOT-1.0
#        - image: $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BUILD_NUMBER
#          readinessProbe:
#            httpGet:
#              path: /actuator/health
#              port: 8080
#            timeoutSeconds: 10
#            failureThreshold: 30
#            periodSeconds: 5
#          imagePullPolicy: Always
          ports:
            - name: tcp-8080
              containerPort: 8080
              protocol: TCP
          resources:
            limits:
              cpu: 300m
              memory: 600Mi
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
#      dnsPolicy: ClusterFirst
#      restartPolicy: Always
#      terminationGracePeriodSeconds: 30
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: jzo2o-foundation
  name: jzo2o-foundation
  namespace: jzo2o
spec:
  ports:
    - name: http-8080
      port: 8080
      protocol: TCP
      targetPort: 8080
      nodePort: 32607
  selector:
    app: jzo2o-foundation
  sessionAffinity: None
  type: NodePort
```



##### 修改数据库地址

将数据库地址、redis地址改为k8s服务的域名，举例：

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/ZuHWb9BeEoWSI4xMXWIciwphnmc.png)
> 图注：图片展示的是Kubernetes中mysql8服务的详情页面。页面左侧显示服务名称、项目、类型等信息，右侧有“获取状态”“元数据”“事件”三个选项卡，当前选中“获取状态”。页面中部呈现了服务的端口映射情况，如3306、33060端口的映射关系。下方“工作负载”区域显示了mysql8-r1-v1的有状态副本集，其状态为运行中（1/1）。页面底部还有“容器组”相关内容。此图与文档中“修改数据库地址”上下文相关，用于说明将数据库地址改为k8s服务的域名时，可参考此页面获取相关信息。

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/Wri2b6E8IobKoux6O0acfeFInFf.png)
> 图注：这是一份配置文件代码的截图，对应文档中修改数据库地址的操作场景，截图内容为项目配置文件中关于spring数据源的配置部分。红色箭头指向的内容为原配置里的数据库地址部分，当前url配置为jdbc:mysql://mysql8.jzo2o:3306/${mysql.db-name}，该内容是文档中要求修改的数据库地址相关内容，修改时需要将此数据库地址改为Kubernetes服务的域名，以满足项目在K8s环境下的部署要求。



#### 2.2.4 创建流水线

在Devops中创建流水线：

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/Zvacbotf4o6HEbxQAVUcBcFFnTc.png)
> 图注：这张图片展示了Devops领域下名为“jzo2o-service”的流水线界面，对应文档中“在Devops中创建流水线”的内容。界面左侧是流水线的基本信息，显示该DevOps项目的任务状态、同步状态均为成功，更新时间为2024年11月23日21点10分19秒；右侧是流水线的执行步骤，从左到右依次为拉取（git拉取jzo2o-foundation代码）、构建与推送（指定容器镜像、构建镜像等步骤）、部署（指定容器并添加凭证）三个核心阶段，界面上方还设有任务状态、运行记录等功能选项。

运行流水线，部署项目。

部署完成后的效果

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/WFfkbw9YIoAct1xeabIcE8M5nRd.png)
> 图注：图片展示的是Kubernetes部署完成后YJY-PIP-KBs项目的应用列表界面。左侧为导航栏，有名称空间、应用列表等选项。右侧显示了多个应用名称及状态，如canal、elasticsearch、jzo2o-counter-service等，部分应用状态为“当前版本1”，计划版本1，且显示了创建时间，如canal大约1年，elasticsearch大约1年等。

#### 2.2.5 运维

运维人员通过监控工具（Prometheus （普罗米修斯）+ Grafana）监控项目

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/CyN2bhm6doWOdsxpnTGcXmbGn6d.png)
> 图注：图片展示的是使用Prometheus监控工具的界面，用于监控系统健康状况。界面上方显示Pod数量为1，Pod内存使用量386MB，CPU使用量3.61ms等信息。下方有Pod内存使用、Pod CPU使用、Network I/O压力等图表，图表以折线形式呈现数据，时间范围从08:40到08:54。

运维人员监控项目，收到告警会通知开发人员。

开发通过查看日志定位问题。

通过ELK或查看容器的日志

![](../../../../../../Attachments/development/architecture/solutions/yunlan-daojia-project/chapter-12-summary-and-deployment/Eddwbh1Y5oEFhVxTIJ3cfLLmnYc.png)
> 图注：图片展示的是使用ELK查看容器日志的界面。上方是日志搜索栏，可输入查询条件。下方左侧是日志过滤器，可选择不同日志类型。右侧是日志内容展示区，上方图表显示了不同时间段的日志条数，下方列表呈现了具体日志信息，包括时间、消息等。该图片与文档中“运维”部分内容相关，直观呈现了通过ELK查看容器日志的操作界面，辅助说明运维时查看日志初步诊断的操作。

运维流程如下：

- **监控系统**：使用监控工具（如Prometheus、Grafana等）实时监控系统的健康状况。
- **日志系统**：配置日志收集和分析工具（如ELK ）。
- **用户反馈**：通过用户反馈渠道（如客服、邮件、社交媒体等）收集问题报告。

 然后通过查看日志初步诊断

- **查看日志**：检查应用日志、服务器日志、数据库日志等，寻找异常信息。
- **监控指标**：查看CPU、内存、磁盘I/O、网络带宽等系统资源的使用情况。
- **告警信息**：查看监控系统中的告警信息，了解问题发生的时机和频率。

 然后去进行问题定位

- **复现问题**：尝试在测试环境或本地环境中复现问题，验证问题是否可重现。
- **分析堆栈**：查看异常堆栈信息，确定问题发生的代码位置。
- **调试代码**：使用调试工具（如IntelliJ IDEA）逐步调试代码，找出问题根源。
- **性能分析**：使用性能分析工具（如VisualVM）分析系统的性能瓶颈。

定位到问题开始解决问题

- **临时措施**：如果问题严重影响系统运行，可以采取临时措施（如重启应用、回滚版本、关闭某些功能等）。
- **修复代码**：找到问题的根本原因后，修改代码并进行单元测试和集成测试。
- **部署更新**：将修复后的代码部署到生产环境，确保问题得到解决。
- **验证效果**：通过监控系统和日志系统验证问题是否完全解决。

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 云岚到家项目 / 第十二章 项目总结&部署 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/ZQD9woYWPiibASk3zUscnZVPnhz)
