---
type: memo
date: 2026-11-06
aliases:
  - "第三章 用户认证"
---
# 第三章 用户认证

## 学习目标

1. 能说出家政项目认证模块有哪些认证方式
2. 能够说出对接微信进行小程序认证的流程
3. 能够说出OpenID与UnionID的区别
4. 说出手机验证码认证的流程
5. 能说出小程序端定位的流程
6. 能够设计开发机构端注册接口
7. 能够设计开发机构端忘记密码接口
8. 能够设计开发实名认证模块
9. 能够设计开发电商项目小程序认证接口



家政项目实战：

机构端注册功能开发

忘记密码功能开发

实名认证模块设计开发







## 代码环境



jzo2o-customer: dev_01

mall-plus: dev_01



## 1 认证模块

### 1.1 需求分析

#### 1）基础概念

一般情况有用户交互的项目都有认证授权功能，首先我们要搞清楚两个概念：认证和授权。

**认证**: 就是校验用户的身份是否合法，常见的认证方式有账号密码登录、手机验证码登录等。

**授权**:则是该用户登录系统成功后当用户去点击菜单或操作数据时系统判断该用户是否有权限，有权限则允许继续操作，没有权限则拒绝访问。

#### 2) 小程序认证流程

了解了认证和授权的概念，本节对小程序认证功能进行需求分析。

本项目包括四个端：用户端(小程序)、服务端（app）、机构端(PC)、运营管理端(PC)．

分别对应四类用户角色：家政需求方即c端用户，家政服务人员、家政服务公司（机构）、平台运营人员。

用户端通过小程序使用平台，初次使用小程序会进行认证，如下图：





![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NU12brKwro2b8lxa8Jcc5SlFnjg.png)

点击“快速登录”弹出服务条款窗口：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/VAS9bwHggos8rmx1AWOcJ2oCn6O.png)

点击“同意”进行认证，系统与微信进行交互获取用户在小程序中的唯一标识openid。

**注意：点击“同意”弹出获取位置信息，此信息表示要进行定位，定位功能稍后介绍，这里选择允许或拒绝都可以。**

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/QaF3bz4MAoUnI8xRGjVcyeNgnXd.png)

初次认证通过会自动注册用户信息到本平台。

下边是小程序的认证流程：

<whiteboard token="K83swaCMahteeCbsF43cKZnUnKh"></whiteboard>



#### 3）OpenID与UnionID

在上边认证流程中提到OpenID，它是什么呢？

AI:项目对接微信登录中unionid和openid的区别是什么



> 在微信的开放平台中，`openid` 和 `unionid` 都是用来唯一标识用户的ID，但是它们的应用场景和作用范围有所不同。
> 
> 1. OpenID：
> 
>    - OpenID 是微信公众平台或小程序平台提供的一个用户身份标识。
>    - 每个用户的 OpenID 在同一个公众号/应用的体系内是唯一的。
>    - 开发者通过获取用户的 OpenID 可以与自己的应用系统中的用户进行关联，实现用户管理等功能。
>    - OpenID 主要用于单一应用（如一个微信公众号或小程序）的用户身份识别。
> 2. UnionID：
> 
>    - UnionID 只有在用户将该应用绑定到微信开放平台之后才能获取。
>    - 在同一个微信开放平台下，每个用户的 UnionID 是唯一的，并且在该开放平台下的所有应用中通用。
>    - 使用 UnionID 的好处在于，当用户授权给多个与同一开放平台账号关联的应用时，这些应用可以共享这个 UnionID，从而可以方便地管理用户在不同应用中的数据和权限。
>    - UnionID 通常用于需要跨应用（比如多个小程序、公众号等）同步用户信息的场景。
> 
> 简单来说，如果你的应用只需要在单一的公众号或小程序内管理用户身份，那么使用 OpenID 就足够了。而如果你的应用涉及到多个不同的应用，并且希望在这些应用之间共享用户身份信息，则需要使用 UnionID。为了能够使用 UnionID，你需要将你的应用绑定到微信开放平台。



我们的项目是一个独立的小程序，我们使用OpenID即可。



#### 4) 手机验证码认证

服务人员通过app登录采用手机验证码认证方式，输入手机号、发送验证码，验证码校验通过则认证通过，初次认证通过将自动注册服务人员信息。

如下图：



![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/AlUZbgeltoswhtxC77AcF439n29.png)

手机验证码认证流程如下：

<whiteboard token="HsldwAcVAh4mpRbNjDAcUqG4n9e"></whiteboard>



#### 5) 账号密码认证

机构端认证方式是账号密码认证方式，通过pc浏览器进入登录界面输入账号和密码登录系统，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/AB9RbFSImoWbctxOfhdcHwO8nJQ.png)

机构端提供单独的注册页面，输入手机号，接收验证码进行注册，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/MFgKbHXDZo6JIoxmpstc3KhSnuf.png)

管理端的认证方式也是账号密码方式，界面如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/YaUhbhoDbouN7bxfRT1c0ByvnIh.png)

管理端的账号由管理员在后台录入，不提供注册页面。



#### 5）小结

本项目的认证方式有哪些？



### 1.2 小程序认证

#### 1.2.1 测试小程序认证

##### 1）参考官方流程

下边测试用户端小程序的认证流程，我们先参考微信官方提供的小程序登录流程先大概知道小程序认证流程需要几部分，如下图：

（文档地址：https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html）

从图上可以看出小程序认证流程需要三部分：

小程序：即前端程序

开发者服务器：后端微服务程序。

微信接口服务：即微信服务器。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WiFsbWd90obdISxINkvcRewanHg.png)

1.前端调用wx.login()获取登录凭证code

2.前端请求后端进行认证，发送code

3.后端请求微信获取openid，发送appid、app密钥、code参数，微信返回openid

4.后端生成认证成功凭证返回给前端。

5.前端存储用户认证成功凭证



##### 2）申请小程序账号

开发小程序首先要申请小程序账号，参考官方文档：https://developers.weixin.qq.com/miniprogram/dev/framework/quickstart/getstart.html#%E7%94%B3%E8%AF%B7%E8%B4%A6%E5%8F%B7

点击注册小程序（https://mp.weixin.qq.com/wxopen/waregister?action=step1）填写信息，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/R77hbsq3zoeyHVx7a04cwCR0nzb.png)

填写完成进入下一步：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/UbWTbPzedoQfL3xy1LWcjyMNnsd.png)

进入邮箱激活小程序，进入下一步信息登记，选择“个人”



![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/OUQ8bAwfvoJAQOxD6Q0cT4x6nfp.png)

填写主体登记信息：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/H6QmbHbfyol2EXxFzYmcNANincf.png)

使用自己的个人微信扫描上图的二维码，如下图，点击“确定”

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Pw6YbjbDvo58d0xWQIDcIk7Enyb.png)

点击“确定”后提示“信息提交成功”，点击前往小程序。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/INxWbQRtlouywMxBrFRcbkiMnze.png)

进入小程序管理界面：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DvIrbcIR1oQcupxeduWcq07unee.png)

填写小程序信息，点击上图中小程序信息栏目的“去填写”按钮，如下图：

小程序名称一定谨慎填写，每年是有一定修改次数限制的。

**注意：小程序名称不能重复**

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/D0S2bqTbdorf0axqlMZcdmPInXf.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Sps6bCEfkowMVkx0Yf4cnTdAnRM.png)

下边配置最关键的appid和密钥，通过左侧菜单找到开发管理菜单：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/MwsAbzrkSolmePxpiD4cuSpinRg.png)

点击“开发管理”菜单进入下图界面，点击“开发设置”，如下图：



![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/AemLbgZfzoEAhLxassPc2Bzcn7f.png)

点击“生成” 弹出二维码，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/VHrIbtcR4oxGUJxFwuYcPOEynEh.png)

使用自己的微信扫描二维码生成密钥成功，点击“复制”将密钥和AppID妥善保存，开发小程序要使用。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WMG7bPfCjoew0gxz8EOcoCyKnId.png)

如果密钥丢失可以进行重置：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/BAbjb3a1foEOo8xJxg3cB9IGnuf.png)



##### 3）创建jzo2o-customer

小程序账号申请成功，下边部署配置后端程序。

客户管理工程jzo2o-customer提供了小程序认证接口支持。

jzo2o-customer通过jzo2o-publics请求微信获取openid。（jzo2o-publics在第一章环境配置中已完成创建）

下边拉取jzo2o-customer工程，本课程提供的jzo2o-customer工程git地址为：https://gitee.com/jzo2o-v2/jzo2o-customer.git，创建过程参考jzo2o-foundations工程，先fork再从自己仓库拉取代码。



创建成功，并切换到dev_01分支，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/AXl6boFfbod897xJ3ZSc7ew1nkb.png)



工程创建完成修改bootstrap-dev.yml配置文件:

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/QMmVbAkaDoHj9qxeR6Ncy2CZnJb.png)

接下来进入nacos修改jzo2o-publics.yaml中小程序的appid和密钥，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DBJibynCAokiAfxQYu4cX7GbnjC.png)



微服务工程配置好下边需要创建jzo2o-customer工程的数据库，从课程资料下的sql脚本目录找到jzo2o-customer-init.sql，执行脚本创建jzo2o-customer数据库。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DKFzbjLxUo27ifxXBrpclPLgneg.png)



启动jzo2o-customer工程，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/B0CibEBQooxpInxQw0KcqM36nib.png)

小程序认证需要启动的微服务包括：网关jzo2o-gateway、客户管理jzo2o-customer、公共服务jzo2o-publics，将其它服务也正常启动。

启动这三个微服务成功，下边开始部署前端。

##### 4) 部署前端

本部分内容可参考微信开发文档：https://developers.weixin.qq.com/ebook?action=get_post_info&docid=000e8842960070ab0086d162c5b80a 

首先下载微信小程序开发工具，也可从课程资料中“小程序开发工具”获取安装程序。

用户端是基于微信小程序开发的，首先需要下载并安装微信开发者工具：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WH7ibr6RSo7AXPxBjEucgB4anIb.png)

可以使用课程资料中提供的安装包或在线下载，[点击下载](https://developers.weixin.qq.com/miniprogram/dev/devtools/stable.html)（https://developers.weixin.qq.com/miniprogram/dev/devtools/stable.html）

安装成功创建快捷方式：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DF4Yb5ECconJodxfgIoceE5cnJ4.png)

**配置小程序开发环境**

首先拷贝到课程资料下源码目录中的project-xzb-xcx-uniapp-java.zip到你的代码目录并解压到project-xzb-xcx-uniapp-java目录下。

然后打开微信开发者工具，初次使用弹出身份确认，如下图，使用申请小程序账号时用的微信进行扫码。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DtuqbxaeMoe3rvxw9c2cUtffnZd.png)



扫码通过，进入下边的界面。

> 如果首先进入小程序开发工具没有进入下图的界面而是自动打开其它项目，可以点击：项目--》关闭当前项目: 
> 
> ![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WI6AbL44zo6vS4xZuH4cnrpPnIg.png)

进入下图界面后，点击加号，选择小程序的目录。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/YWKsbgbyOo9MlRxURPucYUSInZf.png)



选择小程序前端工程的 project-xzb-xcx-uniapp-java\unpackage\dist\dev\mp-weixin目录。

AppID：填写申请小程序号获取的AppID。

选择不使用云服务。

如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/QFnwbfZFnoPBDyxPGYIcobhwnAb.png)



点击确定进入下边的界面：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/H5r3bgBOnog6m1xJQbAcoDz1n8b.png)

修改project-xzb-xcx-uniapp-java\unpackage\dist\dev\mp-weixin\utils\env.js 配置文件，指定后端网关的地址

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/BVskbKEtmoPhZzx97ydcQyXWnPh.png)

设置代理

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HK6ybUyWtoTH44x9O80c6c4BnRg.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Dg9XbMrSAo9nqzxWBiKcYWYanid.png)

选择设置--》项目设置，在本地配置中选择不校验合法域名。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/BlHlbz1d6oUAlzxJz6ycypzon1b.png)

如果要选择其它目录可以关闭当前项目，重新选择。

![](https://feishu.cn/file/MpmdbhhZFoUS4yx9U3qczVJznBf)



##### 5) 编译运行

小程序认证需要启动的微服务包括：网关jzo2o-gateway、客户管理jzo2o-customer、公共服务jzo2o-publics，保证这三个服务全部启动。

**注意：保证jzo2o-publics服务配置高德地图key（参考：高德地图web服务配置文档）、微信的appid和app密钥。配置完成将jzo2o-publics服务重新启动。**

小程序开发环境配置完成进行编译运行。

首先清除缓存

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/POqBbd3awo151BxJhTicGs8cnfg.png)

然后编译运行：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/SpK1bnD7doevHGxceCUce201nPa.png)

编译运行到登录界面：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/OlYybVWB4ot8eRxnakCcA4ahnBc.png)

点击“快速登录”按照前边讲的小程序认证流程进行操作，请求认证接口进行认证，进入调试器-->Network观察请求记录，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/MPUdbNasEoQWzvx6s4McMdZHn5b.png)

认证接口的地址是：/customer/open/login/common/user 

此接口最终从微信拿到用户的openid（微信给用户分配的唯一标识），并将openid存储到数据库，认证通过生成token令牌返回给前端。

认证通过进入下边的界面：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/BGLxbbAWuokEZfxTKO6c38CDnUh.png)

##### 6）真机调试(了解)

在开发环境还可以通过手机打开小程序进行测试，下边介绍具体的配置方法，注意此部分内容作为了解，正常开发使用上边介绍的通过微信开发工具进行测试，方便跟踪接口交互数据。

首先保证手机和PC在同一个网络，因为在手机上打开小程序需要访问PC上的微服务接口。

可以让手机和PC连接同一个热点，连接热点后查询无线网卡的IP，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Xzr7bUTJuoWN96x09WGccNyFnbc.png)

192.168.137.1是我的测试环境，同时要保证手机的IP地址和192.168.137.1在同一个网段。

接下来配置网关地址

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/BfkDbFlK9oCgWexUsd4cUV6vnGd.png)

设置代理

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/U2wdbpeeHob4cpxheW6cDjj5nId.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/UZ6dbpWfBoo1AcxSSmjcCIpQnTc.png)

然后点击预览

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/AGd6b2M0Voi6WmxllFVcgXe2ngf.png)

生成二维码后打开手机微信扫码将在手机上预览。



##### 7)  小结

从配置环境到测试整个过程稍微繁琐，需要按文档的步骤逐个操作，每一步操作成功再进入下一步。

首先申请小程序账号，然后配置后端工程（网关jzo2o-gateway、客户管理jzo2o-customer、公共服务jzo2o-publics）、保证启动成功。

再部署前端工程，这里需要安装微信小程序开发工具等。

最后编译运行前端工程测试认证流程。

测试过程可进入调试界面跟踪前后端的交互数据。

#### 1.2.2 阅读代码

下边通过阅读代码理解小程序认证的流程。

##### 1）小程序认证流程

我们去开发整个小程序认证流程还先参考官方流程，如下图：

（文档地址：https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html）

整个过程包括三部分：

小程序：即前端程序

开发者服务器：后端微服务程序。

微信接口服务：即微信服务器。

具体的流程如下：

1.前端调用wx.login()获取登录凭证code

2.前端请求后端进行认证，发送code

3.后端请求微信获取openid

4.后端生成认证成功凭证返回给前端。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/We1pbN4g4o3d3lxtIK5cZq5fn7g.png)

根据官方的认证流程我们定义本项目小程序认证的交互流程：

customer工程提供认证接口，publics工程作为一个公共服务提供与微信通信的接口。

前端与cutomer交互不与publics交互。

<whiteboard token="AhLGwOeHdhzsD3b4SuMc0EUWnkf"></whiteboard>

##### 2）阅读代码

下边根据认证流程阅读代码，我们以断点调试的方式跟踪接口交互过程。

###### **提供的小程序认证接口**

在jzo2o-customer服务中提供小程序认证接口。

在前端清除缓存后重新编译运行，点击快速登录，授权获取手机号，请求jzo2o-customer的普通用户登录接口，普通用户登录接口如下：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/LC4LbvrOUoscl0xqythcGEZwnpe.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/ZoxtbsbIQo83zrxRRCOcj1Gqnub.png)

###### **请求publics获取openid**

jzo2o-customer服务会请求publics服务申请openid，publics服务与微信进行对接。

publics服务中提供获取openid接口如下：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HSBRbSyhgooWJJxZuGhcvnIqnih.png)



![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/JxJTbf6kWohdPKx75ZPcEoj9nAh.png)

###### **publics请求微信获取openid**

publics请求微信获取获取openid 接口如下:

接口描述：https://developers.weixin.qq.com/miniprogram/dev/OpenApiDoc/user-login/code2Session.html 

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/LqIdbMuhSoeXqAxZ8ZucOzyBn5c.png)

###### **生成token**

**customer收到openid后查询数据库获取用户信息并生成token.**

openid是微信用户在家政o2o平台的唯一标识，首先根据openid查询jzo2o-customer的common_user表，是否存在用户，如果不存在则自动注册用户信息，**用户信息存储到jzo2o-customer数据库的common_user表中**。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/X5FXbire9oiThOxCPN0cAuEqnOg.png)

**common_user表的结构如下：**

```Java
create table `jzo2o-customer`.common_user
(
    id                  bigint                             not null comment '用户id'
        constraint `PRIMARY`
        primary key,
    status              int      default 0                 not null comment '状态，0：正常，1：冻结',
    nickname            varchar(255)                       null comment '昵称',
    phone               varchar(25)                        null comment '电话',
    avatar              varchar(255)                       null comment '头像',
    open_id             varchar(100)                       null,
    account_lock_reason varchar(255)                       null comment '账号冻结原因',
    create_time         datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    update_time         datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    is_deleted          int      default 0                 not null
)
    charset = utf8mb4;
```

认证通过生成用户token返回给前端。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/PEgrbgACDo7VspxeWitcpd6hn7d.png)

token令牌的格式我们使用的是JWT格式，JWT是一种常用的令牌格式，它可以防篡改，关于JWT不明白的同学可以通过视频自学(https://www.bilibili.com/video/BV1j8411N7Bm?p=110&vd_source=81d4489ba9312103debc8ee843169f23)

在JWT令牌中存储了当前登录用户的信息（json），包括如下属性：

用户id: id，对应common_user表的主键。  
用户名称：String name，对应common_user表的nickname字段。  
用户头像：String avatar，对应common_user表的avatar字段。  
用户类型：Integer userType，c端用户的用户类型代码为1，具体定义在com.jzo2o.common.constants.UserType中。



###### **网关对token统一校验**

登录成功后，用户访问小程序会携带token，所有请求到网关，如果是白名单则不校验token，否则需要校验token。

网关对token进行解析校验，token不合法直接返回失败信息，token合法解析出用户信息放在http的head中继续请求微服务。

代码如下：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/E84Hbb8nTo4aNRxZ0K3c9rsun1e.png)

我们可以进行断点调试：

如下图访问“我的地址”进行调试：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Ntxqb8tL6owI4sxvcZocIuzDnNg.png)

进入断点位置：

首先获取前端传入token，网关校验token并解析出用户信息。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HN6gb7jGooFQksx0z75cMGmPnUb.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DIpabDqvfoAlSbxfclJcIBoVnrg.png)

解析token通过，将用户信息放入http头中，继续向微服务转发

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/JtJmbDOfYoM31ixZAqTcyWIPnRA.png)



接下来进入微服务，在微服务中解析http头信息中的用户信息，写入ThreadLocal方便应用程序使用。

源代码如下：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/FSanbugWno49M3xOqyDcLB9Cnmc.png)

"我的地址"接口属于jzo2o-customer微服务，所以我们进入jzo2o-customer微服务，找到此类，在此类打断点。

仍然访问“我的地址”，进入断点，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/SDtwbRIlboCgfAxNBUJcsST0nlh.png)

用户放入ThreadLocal中，在controller及service方法中非常方便去获取当前用户信息。

我们找到“我的地址”接口的service方法并打上断点

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/VxITbVvJvo0pP1xROsEcmUcbn7f.png)

重新 访问“我的地址”进入此断点

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XxAVbf0pdoE8cHxx3f1c1x6snDf.png)

我们发现当前用户的id已经在微服务中成功获取。



##### 3）小结

小程序认证流程是什么？



当前认证通过的用户信息保存到哪里了？



### 1.3 手机验证码认证

#### 1.3.1 测试手机验证码认证

服务人员使用APP登录平台使用的是手机验证码认证方式，整个认证流程也需要部署前端、后端。

客户管理工程jzo2o-customer与公共服务jzo2o-publics提供手机验证码的接口，这两个服务在小程序认证时已经部署这里不再部署，我们只需要部署前端工程即可。

##### 1）部署前端

服务端的前端工程需要使用 HBuilder 3.8.7 X 软件编译运行，从课程资料下的软件工具目录获取安装包HBuilderX.3.8.7.20230703.zip，也可以自行下载（https://www.dcloud.io/hbuilderx.html）。

安装成功快捷方式如下：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/VheGbcz3fo7QVjxJYkBctVO5ngh.png)

启动HBuilderX

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/CDSFbXIWuozZADxSEMwcH9D8nxd.png)



下边从课程资料拷贝project-xzb-app-uniapp-java.zip到代码目录并解压

*（可忽略）cmd进入project-xzb-app-uniapp-java目录运行 npm install || yarn  或 cnpm install || yarn  安装依赖包，如下图：*

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/R0u8bssBwoiDVjxb4I5cEaWWnVh.png)



下边用HBuilderX打开project-xzb-app-uniapp-java目录



![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Afm6bhMZ1o01mNxml4KcNCVsntc.png)

配置网关地址



![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Lv18bR7fGoWUF9xFfRwcjTG1ngg.png)

配置完成，使用HBuilderX运行到浏览器

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/TGtKb3amMo5O42xhbvPcuqyantg.png)

运行成功进入登录页面：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/IfG6bA14yoAyxGxoIrMctJ7KnQc.png)

下边进入调试模式

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DixdbUHA2okHQ7xW6iJcpVzSn5i.png)

选择布局方式：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Ey3Kb6Z7Jo5ZynxpRv6cI03cnPg.png)

打开Network调试窗口：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/X2uabLhd8oPwmdxtkJkcZu8NnAe.png)



如果修改了文件需要重新编译运行在HBuilderX控制台右键

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Qo5QbMdQIoHQhVxH5FjcLg7mn3f.png)

##### 2）认证测试

下边测试手机验证码认证流程。

首先输入手机号，**服务人员的信息存储在jzo2o-customer数据库的serve_provider表中**，从表中找一个手机号录入 

点击发送验证码，此时前端请求后端发送验证码，在开发环境我们从控制台获取验证码，稍后后带大家分析发送验证码的程序。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/I6tmbXBzToJ5yLxINTRctFNGnMb.png)



注意此时因为请求后端发送验证码我们观察在浏览器的Network窗口有一条记录，如下图，该请求必须响应状态为200方可正常发送验证。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/GJIbbisepoaXMvxpM3wcnQLbntf.png)

从控制台获取刚才发送的验证码

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NXtDbKGYiohF39xxTzSciLn6nIb.png)

将从控制台获取的验证码填入验证码输入框

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/J5Y8b5afYo8NiXxEiCAcXojrnAc.png)

点击登录进行认证，认证过程会先校验验证码是否正确，如果验证码正确再根据手机号查询serve_provider表是否存在相应记录且用户未被冻结，全部成功则认证通过。

认证通过进入首页。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/RYqObZ5UtoKzPLxoJ7OcO8kYnSh.png)



#### 1.3.2 阅读代码

##### 1）手机验证码认证流程

customer工程提供认证接口，publics工程作为一个公共服务提供与发送验证码接口。

<whiteboard token="VvaTw1sEDhB9bHb4ZNccE1qinEd"></whiteboard>



##### 2)  阅读代码

###### **找到具体的接口**

前端请求publics服务发送验证码接口：publics/sms-code/send

代码如下：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/YlnhbauAfoypqBxhFoBcxjXInef.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/PjmAb8Muvo1uffxLVgccM7CHnlR.png)

具体发送验证码逻辑：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/UucdbIHXLobRXVxRvOgcAW9SnTg.png)



前端请求customer服务的认证接口：/customer/open/login/worker

代码如下：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/B14ibupM7otqe2xEbSJcKLHZnDc.png)

机构和和服务人员认证接口是同一个，根据类型判断是机构还是服务人员，下图中红色箭头指向的代码是服务人员认证的方法。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/L5o7bK4J7oxwPlxHTEzcJiQrnZb.png)

customer服务请求publics服务校验验证码

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NLGYbVGYsoTrXWxkPNpcqzLonQc.png)

publics服务提供校验验证码接口，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Nc4tbKPr2o1t26xS1dwcG1QmnSh.png)

###### **验证码证码逻辑**

具体的验证码校验逻辑是先查询redis中的正确的验证码，再和用户输入的进行对比，如果不一致则说明输入错误，输入正确删除验证码。如下代码：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/X4YRbuW8LoWfe6xWgwscBpYKnge.png)

在使用redisTemplate时需要在工程中引入下边的依赖：

```Java
<dependency>
    <groupId>com.jzo2o</groupId>
    <artifactId>jzo2o-redis</artifactId>
</dependency>
```

在jzo2o-redis中定义了redisTemplate的定义，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/JXXFbEzU3oMPoyxR56Gc5sunnae.png)

使用redisTemplate时只需要在bean里注入上图中定义的redisTemplate即可。

在测试验证码发送时可以打开redis进行跟踪，下图显示了存入redis中的验证码，注意观察key和value：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/IH4KbsJLkot6FmxuadTcSWVCnIf.png)



###### **自动注册**

校验验证码完成customer服务根据手机号查询数据库，如果用户冻结则认证失败，如果用户不存在则自动注册。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/CKDHbcB3toDO8WxFuOrc30nynBg.png)

服务人员和机构都存储到serve_provider表，结果如下：

```Java
create table `jzo2o-customer`.serve_provider
(
    id                  bigint                             not null comment '主键'
        constraint `PRIMARY`
        primary key,
    code                varchar(255)                       null comment '编号',
    type                int                                not null comment '类型，2：服务人员，3：服务机构',
    name                varchar(255)                       null comment '姓名',
    phone               varchar(255)                       not null comment '电话',
    avatar              varchar(255)                       null comment '头像',
    status              int                                not null comment '状态，0：正常，1：冻结',
    settings_status     int      default 0                 null comment '首次设置状态，0：未完成设置，1：已完成设置',
    password            varchar(255)                       null comment '机构登录密码',
    account_lock_reason varchar(255)                       null comment '账号冻结原因',
    score               double                             null comment '综合评分',
    good_level_rate     varchar(50)                        null comment '好评率',
    create_time         datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    update_time         datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    is_deleted          int      default 0                 not null comment '是否已删除，0：未删除，1：已删除',
    constraint serve_provider_phone_type_uindex
        unique (phone, type)
)
    comment '服务人员/机构表' charset = utf8mb4;
```



最后生成token返回给前端。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/OfOLb4siroccwcxJA2mcfktRneg.png)





##### 3）小结

手机验证码服务的实现方案？



为了方便验证码测试前端会自动填入123456，更改验证码发送接口固定生成的验证码为123456.

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XDZbbqepGozkcmxMaG2cioZRnoK.png)





## 2 用户端定位

### **1) 用户端首页定位**

用户端在小程序认证通过后会自动进行定位，也可以在首页手动定位，**定位成功后用户进入首页会根据定位的城市查询该城市有哪些服务项目**。

手动定位过程如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/LTNsbJtwBogZpYxuw6KcQTdsnxh.png)

点击下图箭头位置进行手动定位。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WE2RbMwNsoscKcxf1eNc0frvnTf.png)



定位成功再次进入首页发现位置变为新地址，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/MoAmbAuepoj8e8xfTizcoEAwn8d.png)

### 2）高德地图配置

小程序端的定位是通过手机的定位模块进行定位，定位成功获取经纬度坐标，平台根据经纬度坐标请求地图服务获取经纬度坐标对应的具体位置。

小程序首先通过微信提供的方法拿到经纬度坐标，然后请求后端获取具体的位置，后端会请求高德地图根据经纬度获取具体的城市信息。

要测试用户端定位的流程首先需要在高德地图开通地图定位服务。

参考“高德地图web服务配置文档” 获取访问接口的key。

进入nacos配置jzo2o-publics.yml中高德地图key。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/CCFEbZ6gEoBXSNx5A4ackmSwnph.png)

### 3) 测试定位

启动jzo2o-publics服务。

启动小程序，先清除缓存再进行编译

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/KJgybLOe9osfySxzFGhc20LenzV.png)

启动成功，点击“快速登录”，先同意服务条款，再允许获取位置信息，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/C6tFbZSflovXq3x03aychabSnec.png)

如果未弹出获取位置提示框，在本地设置中将调试基础库版本设置为2.24.7

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/VXpUbaNfSo9FkVxWISZcjUqDnDg.png)

点击“允许”，观察Network，请求定位接口/publics/map/regeo?location=，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HfU8bi8l1ond77xndX1clpUXn7e.png)

/publics/map/regeo接口返回经纬度对应的城市信息。

完整内容如下：

```Java
{"code":200,"msg":"OK","data": {"cityCode":"010","province":"北京市","city":null,"district":"昌平区","fullAddress":"北京市昌平区城北街道中共北京市昌平区委员会北京市昌平区人民政府"}}
```

在首页左上角发现定位到北京市，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/JiEIb16UpoWPskx1Gv4cKbRfnHb.png)



在不同位置进行测试接口返回的位置信息会和讲义不一致，这个需要注意。

到此说明定位测试通过。

下边介绍一种虚拟定位的设置，这个在开发中经常使用，虚拟定位即不是按手机位置进行定位，比如：你在北京，想测试定位到郑州某个位置该如何操作呢？

在微信开发环境可以指定小程序虚拟定位。

首先打开虚拟定位，指定经纬度，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/LppzbHPMLoj2MnxbqcGcVxtXnBd.png)

经纬度坐标可以使用高德地图的坐标拾取工具获取：

注册高德地图，进入https://lbs.amap.com/tools/picker，即可输入一个具体的位置拿到经纬度，将下图中获取的经纬度设置到上图的位置，清除缓存重新编译小程序即可按虚拟定位去定位。



![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Y0webmg65oeIzZxgAx7crfygnig.png)





### 4）用户端定位交互流程

<whiteboard token="GaWRwwlf4hvTG5bBMZ4cqN4fnFg"></whiteboard>

1. 小程序获取经纬度坐标
2. 前端请求publics服务根据经纬度查询地理编码（cityCode）
3. publics服务调用高德地图接口根据经纬度查询地理编码
4. 前端根据cityCode找到系统中对应的区域ID



### 5) 阅读代码

定位过程中小程序请求publics服务的接口查询经纬度对应的位置信息，调用/publics/map/regeo接口，下边阅读/publics/map/regeo接口。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/K6u0bN7EmoDiOsxtEczcDtyNn9c.png)

接口文档：http://localhost:11503/publics/doc.html#/default/%E5%9C%B0%E5%9B%BE%E6%9C%8D%E5%8A%A1%E7%9B%B8%E5%85%B3%E6%8E%A5%E5%8F%A3/getCityCodeByLocationUsingGET 



通过mapService的getCityCodeByLocation方法调用高德地图的查询地理编码接口，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Q8azbN3khofr1pxc1l8cquqAnth.png)

高德地图的查询地理编码接口定位如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XURtbqw5koy3U2xMp7xc2Vv4n2g.png)

接口文档地址：https://lbs.amap.com/api/webservice/guide/api/georegeo，大家可自行阅读。

最终接口返回省、市、县、详细位置等信息，如下：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/OaNcbulNOoyc0FxvNZfcageFnre.png)

**为什么要调用高德地图的查询地理编码接口呢？**

在foundations数据库的region区域表中有一列是city_code，高德地图返回的详细信息中city_code与region区域表中city_code是一致的，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/QGGbbwXzkofWNPxFvIAcTHR6nlc.png)

小程序拿到的是经纬度坐标，通过调用此接口就可以根据经纬度得到city_code从而关联到平台具体的区域。

#### **region表的city_code从哪来？**

先从从高德地图下载拿到全国的区域信息，包括了city_code（https://lbs.amap.com/api/webservice/download）

将下载得到AMap_adcode_citycode.xlsx文件处理为json文件由前端进行保存。

前端在添加区域时从该json文件中选择区域，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XwEzbOgNeo9QxKxYP6GccMAKnad.png)

区域信息中包括了从高德地图拿到的city_code，添加一个区域将city_code保存到了region表中。





### 6）小结

如何开发小程序定位功能？

- 登录成功之后会先查询已开通区域列表

  - 包含区域id
  - 城市名称
  - 城市编码
- 小程序授权获取当前经纬度，调用public服务，public服务调用高德地图api，根据经纬度获取当前位置

  - 城市名称
  - 城市编码
  - 省、市、区、、、
- 根据区域id查询当前城市的服务

## 3 机构端(实战)

### 3.1 机构端账号密码认证测试

机构和管理端的认证方式都是账号密码认证方式，本作业限定为机构端账号密码认证，具体要求如下：

**部署机构端前端并将认证流程测试通过**

从课程资料的源码目录拷贝project-xzb-PC-vue3-java.zip到自己的代码目录，并解压到project-xzb-PC-vue3-java目录。

修改根目录的vite.config.ts文件中网关地址配置

修改后端地址

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/PPHtbs9XrotcIRx6m2vcfXoDn9b.png)

安装依赖包（如果已经安装依赖包则不用安装）：

cmd进入project-xzb-PC-vue3-java目录运行 ：npm install || yarn  或 cnpm install || yarn

安装依赖包完成运行：npm run dev 运行前端工程，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/MbHIbCqZioZ7C9xWxR1cZfQ9nWd.png)

 运行成功自动打开浏览器：http://localhost:6001/

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DUIkbQMo2oY9WuxDtjjcv5O0nxe.png)

前端默认的账号：15896123123，密码为：888itcast.CN764%...

机构信息存储在jzo2o-customer数据库的serve_provider表中，可从serve_provider表获取账号。

机构端账号密码认证接口请求customer服务的接口：

代码如下：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XuAxbJIYTo9vRFxkYeqcLIdEnbd.png)

登录成功：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EXYTbXM0EokkuOx1EyQcwz3Unqc.png)

退出按钮如下：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Ohorbw7eEompdwx4UohcxT95nkf.png)



### 3.2 **完成机构注册功能开发**

界面原型：

进入登录页面，点击“去注册”进入注册页面

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/P6hqb7hgSoZGmRx45Anc08mlnAw.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HFO0bVQLOoFvfZx20n8cXZgTntc.png)



接口定义如下：

接口地址：**POST**/customer/open/serve-provider/institution/register

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/MXAHbtDojoPd8Dx4sHvczEJpnFb.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/X6babBsvhoLaw4xrL8Ic6TxInIh.png)





#### 设计须知

参考服务端自动注册的代码实现。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HzsLb0znWouGAxxsmTRcheegnoh.png)



![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/QTfWbnN3Aobky3xOgnNcC3GFnIb.png)

**注意：机构端注册和服务端注册完成要向serve_provider表写入数据，具体查阅上图的方法。**



密码加密方式：使用BCrypt方式，BCrypt是一种密码哈希函数，通常用于存储用户密码的安全性。它是基于 Blowfish 密码算法的一种单向哈希函数

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/B3aRb5wc8oURmSxavYmcldjZnie.png)

测试方法：

```JSON
public static void main(String[] args) {
    BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
    /**
     $2a$10$1sp7I0OdYH3Azs/2lK8YYeuiaGZzOGshGT9j.IYArZftsGNsXqlma
     $2a$10$m983E2nmJ7ITlesbXzjbzO/M7HL2wP8EgpgX.pPACDm1wG38Lt.na
     $2a$10$rZvathrW98vVPenLhOnl0OMpUtRTdBkWJ45IkIsTebITS9AFgKqGK
     $2a$10$2gaMKWCRoKdc42E0jsq7b.munjzOSPOM4yr3GG9M6194E7dOH5LyS
     $2a$10$I/n93PIKpKL8m4O3AuT5kuZncZhfqV51bfx5sJrplnYoM7FimdboC
     */
    for (int i = 0; i < 5; i++) {
        //对密码进行哈希
        String encode = passwordEncoder.encode("11111");
        System.out.println(encode);
    }
    //校验哈希串和密码是否匹配
    boolean matches = passwordEncoder.matches("11111", "$2a$10$m983E2nmJ7ITlesbXzjbzO/M7HL2wP8EgpgX.pPACDm1wG38Lt.na");
    System.out.println(matches);
}
```

根据上边的测试代码可知，BCrypt的使用方法如下：

用户输入密码，通过passwordEncoder.encode("输入的密码")得到哈希串，将哈希串存储到数据库。

用户登录校验密码，从数据库取出哈希串，连同用户输入的密码，调用下边的方法：

passwordEncoder.matches("用户输入的密码", "从数据库查询的密码哈希串");





### 3.3 **完成忘记密码功能开发**

界面原型：

进入登录页面，点击“忘记密码”进入找回密码页面

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/OWWUbfYhdo0hc0x3EK4cWeCmnlb.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/T7M8bSawnoNDGGx4yRpcYsCfnhh.png)

接口定义如下：

接口名称：机构登录密码重置接口

接口路径：**POST**/customer/agency/serve-provider/institution/resetPassword

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WQJ4b4YGZosNmlxSho5cMsiYnZd.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/RQG3bn62JoGjQux1NaocUI5inbb.png)

设计须知：

首先校验验证码是否正确。

校验手机号是否存在数据库。

通过校验最后修改密码，密码的加密方式参考机构注册接口。



## 4 实名认证(实战)

### 4.1 需求分析

#### 4.1.1 业务流程

<whiteboard token="MG8nwSycLh8OYObveAWcHgXwnoc"></whiteboard>

#### 4.1.2 服务端**提交认证**

使用一个新的手机号登录，登录成功自动进入下边的界面：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/V7nmbTg5gocgz8xb7pQc1yUQnKe.png)



![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/YVrDbFstqoQqWGx9TBVcwM5Sn8c.png)



![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/W0Fab4ltkoRAYSx9z3fcCPsvnCd.png)

上传身份证件，点击“确认提交”。

提交后等待运营人员审核，当审核没有通过可以查看失败原因：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XrjVbjGjqotqOvxNu2Ec2DQ5ng4.png)

点击“认证失败”

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Kwbgbv6Q6oyMNaxXKrxcTZ17n7b.png)

点击“重新认证”重新填写认证信息。

运营人员审核通过后显示“已完成”

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DuPLbHPiooH8PNxSrzIc8dWGnZx.png)





#### 4.1.3 机构端提交认证

首先登录点击认证进入上传资质界面

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/SaQlbiYMHoEyVsxX1hLcmm8GnOg.png)

点击“去认证”

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/I2lvbrbHcoE8opxQ2y4cfYVVnGc.png)

填写信息点击“提交”，运营人员进行审核 。



可以查看认证结果

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/TQYHbZ6c0ox746xxoxucLRSVnkg.png)

点击“认证失败”

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/AFP9bpM8FoRevCxPtbgcALb5nmh.png)

点击“重新认证”再次上传资质信息。

当认证通过会显示“已完成”。

完成其它项目的设置即可进入首页

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/B5EVbBykmo7ADsx1vOZcMDzJnHh.png)





#### 4.1.4 运营端审核认证

##### 4.1.4.1 审核服务人员

审核服务人员认证列表：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Mi0DbCxmqomDHExBBLGcWNYSnVh.png)

数据说明：

审核状态，0：未审核，1：已审核

待审核:机构或服务人员提交了申请，后台人员没有提交审核结果（同意、驳回）

已审核:机构或服务人员提交了申请，后台人员已选择同意/驳回

认证状态：认证状态，1：认证中，2：认证成功，3: 认证失败

认证中:提交认证后的认证状态

认证通过:显示最新的认证申请，后台人员选择通过，显示认证通过

认证失败:显示最新的认证申请，后台人员选择驳回，显示认证失败



驳回：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/K5jWbBrwXo5ZJixCToQcNxT1n7J.png)

驳回后审核状态为“已审核”，认证状态为“认证失败”

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NH29b05ZRoTreyxZ567cXP6snog.png)

驳回后服务人员可以再次提交认证申请，运营人员在查询列表可以查到多条该用户的申请记录。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/QjFvbNcZYoKHsqxUr13cLm6mnXN.png)

点击“通过”

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Olg5bDF2NoWpthxSYv8cnraXnXf.png)

点击“确定”认证成功

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/FokybUwudocRH6xoEbDcE1WFnJc.png)



##### 4.1.4.2 审核机构



运营人员审核机构：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Tke9bSPKkoCHtCxHEJ2cwEUdnBI.png)

驳回：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/KLLOb3gUWokT7ax280Xccn3vnec.png)

审核完成再查询审核记录，审核状态为已审核，状态为认证失败。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/UEUzbicjdoRxGSxKhruc8zRrndh.png)

当机构端再次提交审核后再次查询企业认证信息，新增了一条认证信息。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NlrtbG96ao56xqx5I9Yciaebnmh.png)

审核通过：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/YDtMbEUdyo45nMxUflqc5vfin6f.png)

审核通过后审核状态为已审核，认证状态为认证成功。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/PLWkbCiqlo905BxiydwcmHSInmg.png)



### 4.2 接口设计

#### 4.2.1 服务端提交认证接口设计

##### 4.2.1.1 服务端提交认证申请

接口路径：**POST**/customer/worker/worker-certification-audit

**请求数据类型:** application/json

请求参数：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/JruHbH5LSoZLHJxACaGcByb3nm5.png)

示例：

```Java
{
  "certificationMaterial": "",
  "idCardNo": "",
  "serveProviderId": 0,
  "backImg": "",
  "frontImg": "",
  "name": ""
}
```

响应参数：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/LEo9bvxvMotk81xyfWBcrEwlnPg.png)

##### 4.2.1.2 查询最新的驳回原因

接口路径：**GET**/customer/worker/worker-certification-audit/rejectReason

**请求数据类型：** application/x-www-form-urlencoded

请求参数：

无

响应参数：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/KusZb1blmoe8t6xqQJWcadOvn3e.png)

示例：

```Java
{
        "msg": "OK",
        "code": 200,
        "data": {
                "rejectReason": ""
        }
}
```

#### 4.2.2 机构端提交认证接口设计

##### 4.2.2.1 机构提交认证申请

接口路径：**POST**/customer/agency/agency-certification-audit

**请求数据类型:** application/json

请求参数：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XkZjbu4tcoCqrPxmnRocHpeinsc.png)

示例：

```Java
{
  "businessLicense": "",
  "legalPersonIdCardNo": "",
  "serveProviderId": 0,
  "idNumber": "",
  "legalPersonName": "",
  "name": ""
}
```

响应参数：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/RGnhbWO3ao4bCHxuwcqcaTion5g.png)

##### 4.2.2.2 查询最新的驳回原因

接口路径：**GET**/customer/agency/agency-certification-audit/rejectReason

**请求数据类型：** application/x-www-form-urlencoded

请求参数：

无

响应参数：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/VxVFbN12ho8eBOxRS54c176mnEh.png)

示例：

```Java
{
        "msg": "OK",
        "code": 200,
        "data": {
                "rejectReason": ""
        }
}
```

#### 4.2.3 运营端审核认证接口设计

##### 4.2.3.1 审核服务人员认证分页查询

接口路径：**GET**/customer/operation/worker-certification-audit/page

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/MXl3b3UGToLdyJxXCVlcpC8dnCh.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Q2K2brvW8oRA3lx9n7ZcVEktnog.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/QBiBb3GqGoBGeEx2M6YcC8gHnqS.png)

示例：

```Java
{
        "msg": "OK",
        "code": 200,
        "data": {
                "list": [
                        {
                                "certificationMaterial": "",
                                "idCardNo": "",
                                "serveProviderId": 0,
                                "updateTime": "",
                                "certificationStatus": 0,
                                "backImg": "",
                                "auditorId": 0,
                                "rejectReason": "",
                                "auditTime": "",
                                "createTime": "",
                                "frontImg": "",
                                "name": "",
                                "auditStatus": 0,
                                "id": 0,
                                "auditorName": ""
                        }
                ],
                "total": 0,
                "pages": 0
        }
}
```

##### 4.2.3.2 审核服务人员认证信息

接口路径：**PUT**/customer/operation/worker-certification-audit/audit/{id}

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/MKNObthg4o39hox3hj0cFeq9nqg.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/G8PPboM2ZoMoVvxjVxkcQKxrnsd.png)

##### 4.2.3.3 审核机构认证分页查询

接口路径：**GET**/customer/operation/agency-certification-audit/page

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/J6j8b5qyKoBPtwx0nkjcNrYMnHy.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/JwntbXbIzotllzx1tNxcEbLOnSg.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/YgEWbWCNCouhPIxQm9KccDxbnLc.png)

示例：

```Java
{
        "msg": "OK",
        "code": 200,
        "data": {
                "list": [
                        {
                                "businessLicense": "",
                                "legalPersonIdCardNo": "",
                                "serveProviderId": 0,
                                "updateTime": "",
                                "certificationStatus": 0,
                                "idNumber": "",
                                "legalPersonName": "",
                                "auditorId": 0,
                                "rejectReason": "",
                                "auditTime": "",
                                "createTime": "",
                                "name": "",
                                "auditStatus": 0,
                                "id": 0,
                                "auditorName": ""
                        }
                ],
                "total": 0,
                "pages": 0
        }
}
```



##### 4.2.3.4 审核机构认证信息

接口路径：**PUT**/customer/operation/agency-certification-audit/audit/{id}

![](https://feishu.cn/file/DkCZbApXHof5XZxt8KIcNTWunyh)



![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DMb2bmO4borkRgxXuU2cev3jnCc.png)



### 4.3 设计须知

系统已经存在以下认证相关的表：

agency_certification：机构认证状态表

worker_certification：服务人员认证状态表

根据需求可知现在要实现的功能是认证的申请与审核，参考上述两张表设计并创建符合需求的表。



## 5 我的账户设置(实战)

### 5.1 配置OSS

本模块在维护银行账户信息时需要上传银行卡照片，本项目的图片服务器使用阿里的OSS存储。

参考 “OSS配置文档” 开通阿里OSS服务，并在publics服务配置参数。

**如果已配置则无需重复配置。**

### 5.2 需求分析

#### 5.2.1 服务端设置银行账户

服务端设置银行卡账户，每个服务人员设置一个银行卡账户。

首先点击“我的”

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DNXqbM2oEobv2WxJToQckt3bn1g.png)

进入“我的”界面，点击“账户设置”

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/IYCSbN2UWonyt7xwCiTcuV1ante.png)

进入账户设置，如果已经设置了账户信息则直接显示在界面中。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/YGi8b1B05oqyivxoCzxc5u0Bn8g.png)

 录入账户信息：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/RDLobTgZeoq1QOxw0FXcQD6JnEg.png)

点击“确认提交”。

再次进入“账户设置”显示设置的账户信息。



#### 5.2.2 机构端设置银行账户

机构端登录进入首页，点击“账户设置”

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/S6bRb3n1joM890xH334c3NTfn0g.png)

打开账户设置界面 

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NCEBbojPLobfNsxpsxOcrozfnBf.png)

填写要求：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/KlPrbJlm9oQAaHxcSMHcGBzbnnc.png)

填写信息：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/K33LbUI2Loyj42xj3idcy9dKnFf.png)

点击“提交”后再次进入“账户设置”显示机构的账户信息。



### 5.3 服务端设置银行账户接口设计

#### 5.3.1 新增或更新银行账号信息接口

接口名称: 新增或更新银行账号信息

接口路径：**POST**/customer/worker/bank-account

请求类型：application/json

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WnzobFtpjod3rVxYSiwc4Ep7nWg.png)



![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/ISEBbXxiloS7Z6xEtZtcErlonJh.png)

#### 5.3.2 获取当前用户银行账号接口

接口名称：获取当前用户银行账号

接口功能：进入账户设置界面会先查询当前用户的账户信息，如果已经设置则显示在页面中。

接口路径：**GET**/customer/worker/bank-account/currentUserBankAccount

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/ZgYlbVz3Ao4RAZxGDeJc8Xinnhg.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WZHxbb6jhoCt9Axk2f6ct8MAn6b.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Q7qjbARjPo6EanxYzySc1QRsnjg.png)



### 5.4 机构端设置银行账户接口设计

#### 5.4.1 新增或更新银行账号信息接口

接口名称: 新增或更新银行账号信息

接口路径：**POST**/customer/agency/bank-account

请求类型：application/json

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/GoAqbicFVojN6gxW2iJcLeJfnOd.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/AhFsbAH11okCQJx8rdYcTuTvnCf.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/YznxbnDaioxwguxtJ96cln1Kncg.png)

#### 5.4.2 获取当前用户银行账号接口

接口名称：获取当前用户银行账号

接口功能：进入账户设置界面会先查询当前用户的账户信息，如果已经设置则显示在页面中。

接口路径：**GET**/customer/agency/bank-account/currentUserBankAccount

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Rxb9bbZjNoGByvxhQWvcXp75nEe.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/J0rKbRcAaogoKLxVikBcrMivnMe.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/GXOcbvXHdoc4wHx0puPcdeGMnPb.png)





## 作业

实战完成要能针对以下问题进行流畅口述：

1、你负责模块的怎么设计的

介绍业务流程、表设计、接口设计

2、在开发中你遇到了什么问题

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 云岚到家项目 / 第三章 用户认证 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/PyPuw9ezkiAX9MkfSEkcUTfOnPh)
