---
type: memo
date: 2026-11-06
aliases:
  - "若依项目取消热部署(可选)"
---

# 若依项目取消热部署(可选)

1. 在admin模块删除依赖

![图片展示了若依项目中pom.xml文件的代码部分。其中，关键信息是第21 - 25行，包含一个<dependency>标签，其内容为<groupId>org.springframework.boot</groupId>，<artifactId>spring-boot-devtools</artifactifactId>，<optional>true</optional>。该图片与文档中“在admin的yml配置文件里面把热部署开关给删除配置”上下文相关，用于说明在若依项目中取消热部署时，需在pom.xml文件中删除此依赖配置，以防止自动热部署。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/SUSWbqnbFo7O3lxlQcycnZYKnBd.png)

![图片展示了若依项目中admin模块的pom.xml文件代码。其中，关键部分用红色框标注，显示了<configuration>标签内的内容，包含<sfork>true</sfork>及注释“如果没有该配置，devtools不会生效”。该图片与文档中“在admin的yml配置文件里面把热部署开关给删除配置”步骤相关，用于说明在pom.xml文件中删除热部署开关配置的操作，以取消热部署。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/KgKqbPvRXoSuUNxUp6HcFIAqn9c.png)

1. 在admin的yml配置文件里面把热部署开关给删除配置

![图片展示了若依项目中取消热部署的操作步骤。左侧是项目结构，显示了admin模块下的相关文件夹及配置文件。右侧是application.yml配置文件内容，其中“devtools: restart: enabled: false”被红色框突出显示，表明热部署开关已关闭。该图片与文档中“在admin的yml配置文件里面把热部署开关给删除配置”步骤对应，直观呈现了操作后的配置状态。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/S1KQb5jE8oFV1ZxQIADcobf7nMf.png)

1. 修改启动项配置

![图片展示了IntelliJ IDEA中Run/Debug Configurations的设置界面。在“Build and run”部分，突出显示了“On frame deactivation”下拉菜单，其内容为“Update classes and resources”。该图片与文档中“修改启动项配置”步骤相关，用于说明在取消热部署时，需在该界面将“On frame deactivation”设置为“Update classes and resources”，以刷新maven并重启项目后不再自动热部署。](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/Pm4ebERYzoy3kyxoZWvccV8InMh.png)

1. 刷新maven，重启项目就不会自动热部署了

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第一章-若依+AI快速开发新项目 / 若依项目取消热部署(可选) · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/GAnywDzV0i1CRJkasShcp2YOn8c)
