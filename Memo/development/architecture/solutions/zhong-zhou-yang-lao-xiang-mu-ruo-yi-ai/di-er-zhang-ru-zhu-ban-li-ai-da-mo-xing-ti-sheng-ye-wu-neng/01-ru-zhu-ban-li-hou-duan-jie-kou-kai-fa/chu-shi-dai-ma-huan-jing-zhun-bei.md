---
type: memo
date: 2026-11-06
aliases:
  - "初始代码-环境准备"
---

# 初始代码-环境准备

## **概述**

到目前为止呢，我们已经基本具备了使用若依框架来进行业务开发的能力。接下来呢，我们会继续深化在若依框架的基础上进行开发。我们第三章的核心目标是开发**入退管理**这个模块，里面包含了一些核心业务，比如**入住办理、智能评估、退住办理**等模块。

在开发这些模块之前呢，我们需要先导入一些基础的代码，这些代码已经完成了房型设置和楼层、房间、床位相关的模块的开发，同时在这些代码中也完成了其他前端页面代码，我们需要在此基础上开发新的业务模块。



## 环境搭建步骤

1. 找到对应仓库和分支

   1. 后端地址：https://gitee.com/lee_a/zzyl-init.git
   2. 前端地址：https://gitee.com/lee_a/zzyl-vue3-init.git
2. 将代码fork到自己的Gitee远程仓库中

<callout emoji="🐳">
注意：如果自己原来在Gitee上的远程仓库名称也叫zzyl和zzyl-vue3，需要先改一个名字或者直接删掉原来的仓库，再来fork代码到自己的仓库中。
</callout>

![图片展示的是Gitee平台上的仓库页面。页面上方显示仓库名称为“alexisiyang/zzyl”，右侧有“Watch”“Star”“Fork”按钮，其中“Fork”按钮被红色框突出显示。页面中部是仓库代码列表，包含多个文件夹和文件，如“dev1007 - init”分支下的“zzyl - admin”“zzyl - common”等。右侧有仓库简介、标签、语言、许可证、发行版等信息，以及贡献者列表。该图片与文档中“将代码fork到自己的Gitee远程仓库中”步骤相关，直观呈现了仓库页面及fork按钮位置。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-er-zhang-ru-zhu-ban-li-ai-da-mo-xing-ti-sheng-ye-wu-neng/01-ru-zhu-ban-li-hou-duan-jie-kou-kai-fa/QzlYb8cteoIsqMxSYrUcphgXnKd.png)

fork完成后，所有的代码、分支和提交记录都会同步到自己的仓库中。接下来，只需要通过` git clone` 命令将代码克隆到自己电脑上

1. 使用idea打开后端项目后，先检查编码、Maven、JDK版本设置是否都正确，如果有问题，先调整好。
2. 接下来，先切换到今天的开发分支上进行开发，比如:dev_0526
3. 导入对应的SQL脚本，覆盖之前的表结构即可，如下图：

![图片展示了代码仓库中“初始化代码”分支下的SQL脚本文件列表。其中，“zzyl_20250102.sql”文件被红色框突出显示。根据上下文，文档在介绍环境准备步骤，其中第5步提到导入对应的SQL脚本，覆盖之前的表结构，此图片即为对应SQL脚本文件的示例，表明需导入这些SQL脚本以进行开发。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-er-zhang-ru-zhu-ban-li-ai-da-mo-xing-ti-sheng-ye-wu-neng/01-ru-zhu-ban-li-hou-duan-jie-kou-kai-fa/IDUfbfiYbo6JD2xID36cm6DUnGb.png)



1. 启动后端项目

   1. 修改MySql数据库和Redis对应的用户名和密码为自己的
   2. **修改阿里云OSS的bucketName为自己的**
2. 启动前端项目

   1. **需要安装依赖，将原来前端项目的node_modules拷贝至当前目录下**
   2. 执行`npm install` 命令重新下载一下必要依赖
   3. 启动项目：`npm run dev`

## 业务介绍

- **房型设置**

  - 效果图：

  ![这是中州养老系统里的房型设置效果图，界面左侧是系统导航栏，包含服务管理、在线管理、房型设置等功能选项；主区域展示了room_type表的房型列表，每一行对应一种房型，包含序号、房型图片、房型类型、床位数量、房型说明、创建时间、状态及修改、删除等操作选项，列表中呈现了单人套房、双人套房等多种不同类型的房型信息。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-er-zhang-ru-zhu-ban-li-ai-da-mo-xing-ti-sheng-ye-wu-neng/01-ru-zhu-ban-li-hou-duan-jie-kou-kai-fa/ISR1biLWroB28yxU9XccIsVGnid.png)

  - 房型设置功能用来管理不同的房间类型，比如有：四人间、三人间、普通单人间、特护房等等
  - 表结构：room_type
  
    ![图片展示了房型表（room_type）的结构，包含id、name、bed_count、price、introduction、photo、status、create_time、update_time、create_by、update_by、remark等字段。其中，id为主键ID，name为房型名称，bed_count是床位数量，price为床位费用，introduction为介绍，photo是照片，status为状态，0禁用1启用，create_time和update_time分别是创建和更新时间，create_by和update_by分别是创建人和更新人id，remark为备注。该图与文档中房型设置功能相关，直观呈现了房型表的字段信息。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-er-zhang-ru-zhu-ban-li-ai-da-mo-xing-ti-sheng-ye-wu-neng/01-ru-zhu-ban-li-hou-duan-jie-kou-kai-fa/MZSdbQ0TRoviazx2OfLcV7XqndM.png)
- 床位预览

  - 效果图：

  ![图片展示的是中州养老在住管理中床位预览的界面。左侧为功能导航栏，选中“房型设置”下的“床位预览”。右侧显示109、1022、101等床位信息，如床位号、当前状态（空间中、已入住、请购中）、入住老人信息等。109床位有编辑图标，1022床位有删除图标，101床位有编辑图标和删除图标。该图与上下文介绍的床位预览功能相契合，直观呈现了床位管理及查看入住情况的操作界面。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-er-zhang-ru-zhu-ban-li-ai-da-mo-xing-ti-sheng-ye-wu-neng/01-ru-zhu-ban-li-hou-duan-jie-kou-kai-fa/IyfabdFXwoiu6ExVMw5cuzkWnnh.png)

  - 床位预览主要是对养老院的床位进行日常的管理及查看当前的入住情况，日常管理包含对楼层、房间、床位的增删改查，查看入住情况包含房间的入住率，床位所对应的老人信息等，通过床位图可以让管理员一目了然的了解养老院当前的床位分布和设备分布情况；
  - 相关表结构

  ![图片展示了养老院管理系统中与房型设置相关的表结构。左侧为floor楼层表，包含id、name、code、create_time等字段；中间是room房间表，有id、code、sort、type_name等字段；右侧是bed床位表，包含id、bed_number、bed_status等字段。每张表均有主键标识，并对部分字段进行了备注说明。该图与上下文介绍的房型设置功能相关，直观呈现了系统中涉及的表结构及字段信息。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-er-zhang-ru-zhu-ban-li-ai-da-mo-xing-ti-sheng-ye-wu-neng/01-ru-zhu-ban-li-hou-duan-jie-kou-kai-fa/WED3bbNVVoXIcVxGXiKcwrR3nMf.png)

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第二章-入住办理+AI大模型提升业务能力 / 01-入住办理-后端接口开发 / 初始代码-环境准备 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/DhvpwxI2viwZsMkQg53ccs8gnvd)
