---
type: memo
date: 2026-11-06
aliases:
  - "di-san-zhang-yong-hu-ren-zheng"
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





![nu12brkwro2b8lxa8jcc5slfnjg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/nu12brkwro2b8lxa8jcc5slfnjg.png)
> 图注：图片展示的是用户端（小程序）初次使用时的认证界面。界面上方显示“登录”字样，右上角有三个点和一个相机图标。中间部分有“云岚到家”标志及“一站式家政服务平台”字样。最下方有一个红色的“快速登录”按钮。该图片与上下文紧密相关，上下文介绍了本项目包含四个端及对应四类用户角色，用户端初次使用小程序会进行认证，此图直观呈现了认证界面中“快速登录”按钮的位置，与上下文介绍的认证流程相呼应。

点击“快速登录”弹出服务条款窗口：

![vas9bwhggos8rmx1awocj2ocn6o](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vas9bwhggos8rmx1awocj2ocn6o.png)
> 图注：图片展示的是用户端（小程序）在初次使用时进行认证时弹出的服务条款及隐私政策窗口。窗口内说明在注册成为会员过程中需点击同意《云岚到家服务条款》《云岚到家隐私政策》，并强调需仔细阅读理解条款内容，尤其是以粗体并下划线标识的条款。窗口下方有“不同意”和“同意”两个按钮，其中“同意”按钮为红色。该图片与上下文紧密相关，是对用户端认证流程中点击“快速登录”后弹出服务条款窗口的直观呈现。

点击“同意”进行认证，系统与微信进行交互获取用户在小程序中的唯一标识openid。

**注意：点击“同意”弹出获取位置信息，此信息表示要进行定位，定位功能稍后介绍，这里选择允许或拒绝都可以。**

![qaf3bz4maouni8xrgjvcyengnxd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qaf3bz4maouni8xrgjvcyengnxd.png)
> 图注：该图片展示的是用户在完成小程序认证流程时，系统弹出的获取位置信息的提示窗口，窗口顶部明确显示“获取你的位置信息”与“获取您的位置信息”两行文字提示，下方设有两个操作按钮，左侧为白色背景的“拒绝”按钮，右侧为绿色背景的“允许”按钮。结合上下文可知，这是用户点击认证步骤的“同意”后弹出的界面，用于请求位置权限，且此步骤的定位功能并非当前认证的必需内容，用户选择允许或拒绝均不影响本次认证流程的完成。

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



![aluzbgeltoswhtxc77acf439n29](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/aluzbgeltoswhtxc77acf439n29.png)
> 图注：这张图片展示的是云岚到家一站式家政服务平台服务端的手机验证码认证界面，与文档中提及的“手机验证码认证”内容对应。界面顶部显示平台名称与所属服务端标识，下方设置有供输入手机号的输入框、供输入验证码的输入框，以及用于触发获取验证码操作的按钮，最下方设有粉色的登录按钮，整体是供用户完成手机验证码认证操作的交互界面。

手机验证码认证流程如下：

<whiteboard token="HsldwAcVAh4mpRbNjDAcUqG4n9e"></whiteboard>



#### 5) 账号密码认证

机构端认证方式是账号密码认证方式，通过pc浏览器进入登录界面输入账号和密码登录系统，如下图：

![ab9rbfsimowbctxofhdchwo8njq](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ab9rbfsimowbctxofhdchwo8njq.png)
> 图注：这张图片是云岚到家一站式家政服务平台的机构端登录界面，该界面属于文档中机构端账号密码认证相关内容，用于用户进行账号密码验证登录。界面顶部显示平台名称及所属的机构端标识，设有两个输入框，分别对应账号和密码，输入框旁标注了“账号必填”“密码必填”的提示，密码输入框右侧带有显示密码的标识；下方是醒目的红色登录按钮，底部还有“没有账号？去注册”“忘记密码”的选项，供用户注册新账号或找回密码。

机构端提供单独的注册页面，输入手机号，接收验证码进行注册，如下图：

![mfgkbhxdzo6jioxmpstc3khsnuf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mfgkbhxdzo6jioxmpstc3khsnuf.png)
> 图注：图片展示的是云岚到家一站式家政服务平台机构端的注册页面。页面上方有平台标志及名称。下方有三个输入框，依次为手机号、验证码、密码，其中验证码和密码输入框被红色框线突出显示。验证码输入框右侧有“获取验证码”按钮。最下方是红色的“注册”按钮。该图片与文档中“机构端提供单独的注册页面，输入手机号，接收验证码进行注册”的内容对应，直观呈现了注册操作界面。

管理端的认证方式也是账号密码方式，界面如下图：

![yauhbhodboun7bxfrt1c0byvnih](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/yauhbhodboun7bxfrt1c0byvnih.png)
> 图注：这张图片是云岚到家管理端的账号密码登录界面，界面顶部标注有该平台的名称与身份属性。登录区域设有两个输入框，分别为账户输入栏与密码输入栏，密码输入栏带有红色边框，下方还以红色文字标注“密码必填”的提示。界面最下方有一个醒目的红色“登录”按钮，该界面对应文档中管理端采用账号密码认证方式的内容，用于管理端账号的登录操作。

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

![wifsbwd90obdisxinkvcrewanhg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wifsbwd90obdisxinkvcrewanhg.png)
> 图注：图片展示了小程序登录流程时序图。小程序调用wx.login()获取code，再通过wx.request()发送code给开发者服务器。开发者服务器请求微信接口服务，以appid、appsecret、code获取session_key、openid等。开发者服务器将自定义登录态与openid、session_key关联，返回自定义登录态，最后存入storage。该图直观呈现了小程序前端、开发者服务器、微信接口服务三者间登录认证的交互过程，与文档中小程序认证测试流程的前端调用、后端请求微信获取openid等步骤相呼应。

1.前端调用wx.login()获取登录凭证code

2.前端请求后端进行认证，发送code

3.后端请求微信获取openid，发送appid、app密钥、code参数，微信返回openid

4.后端生成认证成功凭证返回给前端。

5.前端存储用户认证成功凭证



##### 2）申请小程序账号

开发小程序首先要申请小程序账号，参考官方文档：https://developers.weixin.qq.com/miniprogram/dev/framework/quickstart/getstart.html#%E7%94%B3%E8%AF%B7%E8%B4%A6%E5%8F%B7

点击注册小程序（https://mp.weixin.qq.com/wxopen/waregister?action=step1）填写信息，如下图：

![r77hbsq3zoeyhvx7a04cwcr0nzb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/r77hbsq3zoeyhvx7a04cwcr0nzb.png)
> 图注：该图片是小程序注册页面，属于申请小程序账号流程中的信息填写环节。页面顶部显示注册流程的三个步骤，当前处于第一步“账号信息”；页面左侧设有邮箱、密码、确认密码、验证码的输入栏，其中邮箱输入框旁标注了作为登录账号需填写未被微信公众平台注册的邮箱，验证码输入框旁配有含字母“R”“B”的图片及“换一张”选项。页面底部有“你已阅读并同意《微信公众平台服务协议》及《微信小程序平台服务条款》”的勾选框，还有绿色的“注册”按钮；页面右侧设有“已有微信小程序？立即登录”的选项。

填写完成进入下一步：

![ubwtbpzedoqfl3xy1lwcjymnnsd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ubwtbpzedoqfl3xy1lwcjymnnsd.png)
> 图注：这张图是测试小程序认证流程里的邮箱激活环节页面，页面顶部标注了当前处于“② 邮箱激活”步骤。页面提示已向注册邮箱miaoruntu@gmail.com发送小程序账号激活邮件，用户可点击“登录邮箱”按钮前往收信，还提供了未收到邮件时的三条处理建议，分别是检查邮箱地址、检查邮箱垃圾箱、若还是未收到则尝试重新发送。

进入邮箱激活小程序，进入下一步信息登记，选择“个人”



![ouq8bawfvojaqoxd6q0ct4x6nfp](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ouq8bawfvojaqoxd6q0ct4x6nfp.png)
> 图注：图片展示的是微信公众号平台的用户信息登记页面。页面上方有“账号信息”“邮箱激活”“信息登记”三个选项卡。主体类型处有“个人”“企业”“政府”“媒体”“其他组织”五个选项，其中“个人”选项被红色箭头指向。页面提示用户信息登记审核通过后可依法享有微信公众号账号所产生的权利和收益，需对行为负责，注册信息在法律允许范围内向微信用户展示，法院等有权机关可依法调取注册信息。

填写主体登记信息：

![h6qmbhbfyol2exxfzymcnanincf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/h6qmbhbfyol2exxfzymcnanincf.png)
> 图注：该图片展示的是小程序账号申请流程里的“主体信息登记”页面，页面需依次填写身份证姓名、身份证号码、管理员手机号码、短信验证码四项内容，这些输入项旁均附带对应填写规则的提示。页面的管理员身份验证板块设有二维码，同时配有文字说明，要求管理员本人扫描该二维码，此验证方式不收取任何费用，注册后扫码的微信号将成为该账号的管理员微信号。页面底部设有绿色的“继续”按钮，用于完成信息填写及验证后的后续操作。

使用自己的个人微信扫描上图的二维码，如下图，点击“确定”

![pw6ybjbdvo58d0xwqidcik7enyb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/pw6ybjbdvo58d0xwqidcik7enyb.png)
> 图注：该图片是申请小程序账号流程中的一个确认环节界面，页面顶部带有红色感叹号标识，提示用户确认将要提交的主体信息，展示了主体名称与“个人”的主体类型，同时附带相关的责任说明文字，明确主体提交相关信息后的权利义务，以及填写信息需满足的真实性等要求。界面底部设有“确定”与“取消”两个操作按钮，其中绿色的“确定”按钮带有红色箭头明确指向，对应前文“使用个人微信扫描二维码后点击‘确定’”的操作要求。

点击“确定”后提示“信息提交成功”，点击前往小程序。

![inxwbqrtlouywmxbrfrcbkimnze](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/inxwbqrtlouywmxbrfrcbkimnze.png)
> 图注：图片展示的是小程序管理界面中填写小程序信息提交成功后的界面。画面中央有一个绿色圆形图标，内有白色对勾，下方文字显示“信息提交成功。你可以前往微信公众平台使用相关功能。”最下方有一个绿色按钮，按钮上写有“前往小程序”。该图片与上文“填写小程序信息，点击‘去填写’按钮”后的操作结果对应，表明填写小程序信息成功，可前往微信公众平台使用相关功能。

进入小程序管理界面：

![dvirbcir1oqcupxeduwcq07unee](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dvirbcir1oqcupxeduwcq07unee.png)
> 图注：图片展示的是小程序管理界面中的小程序发布流程页面。左侧有“首页”“管理”“统计”“功能”等导航栏，当前选中“管理”。右侧标题为“小程序发布流程”，下方有三个未填写的流程选项，分别是“小程序信息”“小程序类目”“小程序备案”，每个选项右侧均有“去填写”或“去补充”按钮。该图片与上文“进入小程序管理界面”内容对应，直观呈现了小程序发布流程的界面情况。

填写小程序信息，点击上图中小程序信息栏目的“去填写”按钮，如下图：

小程序名称一定谨慎填写，每年是有一定修改次数限制的。

**注意：小程序名称不能重复**

![d0s2bqtbdorf0axqlmzcdmpinxf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/d0s2bqtbdorf0axqlmzcdmpinxf.png)
> 图注：图片展示的是小程序认证中填写小程序信息的界面。界面上方显示小程序名称为“jzc2o项目”，下方有小程序简称（选填）和小程序头像的填写区域。小程序简称处有字数限制，当前输入了“云岚到家”，并有头像预览图。该图片与上文提到的点击“开发管理”菜单进入开发设置，点击“生成”弹出二维码后，使用微信扫描二维码生成密钥成功，点击“复制”将密钥和AppID妥善保存，开发小程序要使用的内容相关，是申请小程序账号时填写信息的界面示例。

![sps6bcefkowmvkx0yf4cntdanrm](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/sps6bcefkowmvkx0yf4cntdanrm.png)
> 图注：这张图片是测试小程序认证流程中，申请小程序账号环节的相关界面，界面内有“小程序介绍”输入栏，其中已输入“jzo2o项目是一个家政o2o平台”的内容，输入栏右下角显示字符数为26/120，下方有提示文字，要求确认介绍内容不含国家相关法律法规禁止内容，右下角设有绿色的“提交”按钮。该界面对应文档中提及的申请小程序账号相关内容，是完成小程序账号申请流程的一个操作页面。

下边配置最关键的appid和密钥，通过左侧菜单找到开发管理菜单：

![mwsabzrksolmepxpid4cuspinrg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mwsabzrksolmepxpid4cuspinrg.png)
> 图注：这是小程序开发管理相关的界面截图，界面左侧的导航菜单中，有“开发管理”“开发工具”“云服务”三个选项，其中“开发管理”选项被红色方框突出标注，结合上下文可知，该界面是小程序认证流程里的关键操作界面，用于指导开发者找到开发管理菜单，后续需通过该菜单点击开发设置进行后续密钥生成等操作，这一步是获取小程序认证所需的AppID和密钥的前置环节。

点击“开发管理”菜单进入下图界面，点击“开发设置”，如下图：



![aemlbgzfzoeahlxasspc2bzcn7f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/aemlbgzfzoeahlxasspc2bzcn7f.png)
> 图注：图片展示的是微信小程序开发管理界面中的“开发设置”页面。页面上方有“开发管理”“运维中心”“监控告警”“开发设置”“接口设置”“安全中心”等选项卡，当前选中“开发设置”。页面中部显示开发者ID、AppID（小程序ID）和AppSecret（小程序密钥）等信息，其中AppSecret右侧有“生成”按钮，该按钮被红色箭头指向。此图与文档中申请小程序账号后配置密钥的操作步骤相关，用于指导用户点击“生成”按钮生成密钥。

点击“生成” 弹出二维码，如下图：

![vhribtcr4oxgujxfwuycpoeyneh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vhribtcr4oxgujxfwuycpoeyneh.png)
> 图注：图片展示的是小程序认证中生成AppSecret的二维码界面。上方显示“生成AppSecret”，下方有一个黑色二维码，下方文字提示“请用管理员微信扫码验证身份”。该图片与上文提到的点击“开发设置”后点击“生成”弹出二维码的内容对应，是申请小程序账号时，通过微信扫描二维码生成密钥的步骤展示，密钥和AppID需妥善保存，用于开发小程序。

使用自己的微信扫描二维码生成密钥成功，点击“复制”将密钥和AppID妥善保存，开发小程序要使用。

![wmg7bpfcjoew0gxz8eococyknid](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wmg7bpfcjoew0gxz8eococyknid.png)
> 图注：图片展示的是微信小程序开发管理中生成AppSecret的界面。界面上方显示“生成AppSecret”，下方有AppID（小程序ID）和AppSecret（小程序密钥）两项内容，其中AppID为“wx0fe49dab12497b3c”，AppSecret为“107ec17bf2...d52”。右侧有一个红色箭头指向“复制”按钮。底部有勾选框，提示了解AppSecret存储规则并已复制保存，还有“关闭”按钮。该图片与上文申请小程序账号后配置密钥的内容相关，展示了密钥生成后的界面及操作。

如果密钥丢失可以进行重置：

![babjb3a1foeoo8xjxg3cb9ignuf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/babjb3a1foeoo8xjxg3cb9ignuf.png)
> 图注：图片展示的是小程序开发管理中“开发设置”界面。界面上方显示“开发者ID”，下方有“AppID(小程序ID)”和“AppSecret(小程序密钥)”信息，其中AppID为“wx0fe49dab12497b3c”。在AppSecret右侧有“重置”按钮，该按钮被红色箭头指向突出显示。此图与文档中测试小程序认证申请小程序账号后，点击“开发管理”菜单进入的界面内容相关，用于说明在该界面可进行密钥重置操作。



##### 3）创建jzo2o-customer

小程序账号申请成功，下边部署配置后端程序。

客户管理工程jzo2o-customer提供了小程序认证接口支持。

jzo2o-customer通过jzo2o-publics请求微信获取openid。（jzo2o-publics在第一章环境配置中已完成创建）

下边拉取jzo2o-customer工程，本课程提供的jzo2o-customer工程git地址为：https://gitee.com/jzo2o-v2/jzo2o-customer.git，创建过程参考jzo2o-foundations工程，先fork再从自己仓库拉取代码。



创建成功，并切换到dev_01分支，如下图：

![axl6boffbod897xj3zsc7ew1nkb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/axl6boffbod897xj3zsc7ew1nkb.png)
> 图注：这张图片展示了jzo2o-customer工程的相关内容，左侧红色框内是该工程的项目目录，包含src、test等文件夹，其中resources文件夹下有bootstrap-dev.yml、bootstrap-prod.yml等配置文件，底部Git分支区域显示当前处于dev_01分支。右侧是bootstrap-dev.yml配置文件的内容，其中突出标注了nacos相关配置，包括username、password、server-addr等信息。这张图对应文档中“创建成功，并切换到dev_01分支”的内容，直观呈现了创建后的工程结构、分支状态及相关配置文件的情况。



工程创建完成修改bootstrap-dev.yml配置文件:

![qmmvbakadohj9qxer6ncy2cznjb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qmmvbakadohj9qxer6ncy2cznjb.png)
> 图注：图片展示了jzo2o-customer工程的文件结构及部分配置文件内容。左侧为工程文件夹，包含src、test等目录，其中src目录下有main、test等子目录。右侧是bootstrap-dev.yml配置文件，其中nacos配置项下server - addr被红色框突出显示，其值为192.168.101.68:8848。该图片与上文提到的“修改bootstrap - dev.yml配置文件”内容相关，展示了配置文件中nacos相关配置项的具体位置和内容。

接下来进入nacos修改jzo2o-publics.yaml中小程序的appid和密钥，如下图：

![dbjibyncaokiafxqyu4cx7gbnjc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dbjibyncaokiafxqyu4cx7gbnjc.png)
> 图注：这张图展示的是在Nacos配置中心中编辑jzo2o-publics.yaml配置文件的界面，界面里配置格式选择了YAML，配置内容里的微信小程序（wechat）相关配置被重点标注，其中app-id为wx0开头的编码，secret为107ec17bf25c8f35cd8e7f2开头的密钥，这部分正是文档中提到需要修改的小程序appid和密钥。



微服务工程配置好下边需要创建jzo2o-customer工程的数据库，从课程资料下的sql脚本目录找到jzo2o-customer-init.sql，执行脚本创建jzo2o-customer数据库。

![dkfzbjlxuo27ifxxbrpclplgneg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dkfzbjlxuo27ifxxbrpclplgneg.png)
> 图注：图片展示了jzo2o - customer数据库中tables文件夹下的表结构。其中包含address_book、agency_certification、common_user、fail_msg、institution_staff、serve_provider、serve_provider_settings、serve_provider_sync、serve_skill、worker_certification等表。该图片与上文创建jzo2o - customer工程的数据库后，从课程资料下的sql脚本目录找到jzo2o - customer - init.sql执行脚本创建jzo2o - customer数据库的内容相关，直观呈现了数据库中表的构成情况。



启动jzo2o-customer工程，如下图：

![b0cibebqooxpinxqw0kcqm36nib](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/b0cibebqooxpinxqw0kcqm36nib.png)
> 图注：图片展示的是IntelliJ IDEA中jzo2o-customer工程的运行界面。左侧Services面板显示“Running”状态，其中“CustomerApplicaiton”被高亮选中，其右侧有“11502”字样。右侧Console面板显示了多条日志信息，包括2023 - 11 - 28 21:22:01.934的INFO日志，内容为“com.jzo2o.customer.CustomerApplicaiton：家政服务 - 客户中心启动”。该图片与文档中“启动jzo2o - customer工程”内容相关，直观呈现了工程运行状态及启动信息。

小程序认证需要启动的微服务包括：网关jzo2o-gateway、客户管理jzo2o-customer、公共服务jzo2o-publics，将其它服务也正常启动。

启动这三个微服务成功，下边开始部署前端。

##### 4) 部署前端

本部分内容可参考微信开发文档：https://developers.weixin.qq.com/ebook?action=get_post_info&docid=000e8842960070ab0086d162c5b80a 

首先下载微信小程序开发工具，也可从课程资料中“小程序开发工具”获取安装程序。

用户端是基于微信小程序开发的，首先需要下载并安装微信开发者工具：

![wh7ibr6rso7axpxbjeucgb4anib](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wh7ibr6rso7axpxbjeucgb4anib.png)
> 图注：图片展示了微信开发者工具（稳定版Stable Build）的下载地址与更新日志。1.06系列版本不再支持Windows7，建议升级Windows版本。1.06.2209190版本支持Windows 64、Windows 32、macOS x64、macOS ARM64。1.05系列版本支持Windows7系统，但仅修复重大bug，不再随1.06节奏发版。图片与文档中用户端基于微信小程序开发，需下载并安装微信开发者工具的内容相关，为下载工具提供指引。

可以使用课程资料中提供的安装包或在线下载，[点击下载](https://developers.weixin.qq.com/miniprogram/dev/devtools/stable.html)（https://developers.weixin.qq.com/miniprogram/dev/devtools/stable.html）

安装成功创建快捷方式：

![df4yb5ecconjodxfgiocee5cnj4](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/df4yb5ecconjodxfgiocee5cnj4.png)
> 图注：这张图片展示的是微信开发者工具的图标，图标主体为白色圆形，圆形内有一个灰色的卡通图案，图案上带有绿色的箭头和对勾标识。该图片对应文档中“安装成功创建快捷方式”的内容，直观呈现了用户端开发所需的工具标识，呼应了前文关于下载安装微信开发者工具的操作指引，用于说明安装完成后会生成该工具的快捷方式图标。

**配置小程序开发环境**

首先拷贝到课程资料下源码目录中的project-xzb-xcx-uniapp-java.zip到你的代码目录并解压到project-xzb-xcx-uniapp-java目录下。

然后打开微信开发者工具，初次使用弹出身份确认，如下图，使用申请小程序账号时用的微信进行扫码。

![dtuqbxaemoe3rvxw9c2cutffnzd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dtuqbxaemoe3rvxw9c2cutffnzd.png)
> 图注：这张图片是微信开发者工具的扫码界面，顶部标注工具版本为v1.06.2307250，界面主体为带小程序标识图标的二维码。该图片对应小程序认证流程里的扫码环节，是测试小程序认证部署前端步骤的相关界面，操作人员可通过该界面完成小程序认证的扫码操作。



扫码通过，进入下边的界面。

> 如果首先进入小程序开发工具没有进入下图的界面而是自动打开其它项目，可以点击：项目--》关闭当前项目: 
> 
> ![wi6abl44zo6vs4xzuh4cnrppnig](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wi6abl44zo6vs4xzuh4cnrppnig.png)
> 图注：图片展示了小程序开发工具的“项目”菜单界面。菜单中包含“新建项目”“导入项目”“打开最近项目”“更换开发模式”“新建代码片段”等选项，其中“关闭当前项目”选项被红色箭头突出指向。该图片与文档中“测试小程序认证”部分内容相关，用于说明若首先进入小程序开发工具后未进入特定界面，可通过点击“项目”菜单下的“关闭当前项目”来关闭当前项目，以便后续操作。

进入下图界面后，点击加号，选择小程序的目录。

![ywksbgbyoo9mlrxurpucyusinzf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ywksbgbyoo9mlrxurpucyusinzf.png)
> 图注：这张图展示的是小程序开发工具的操作界面，左侧导航栏已选中“小程序”选项，主界面标题为“小程序”，下方设有“编辑、调试小程序”字样，界面中间区域有一个被红色箭头指向的加号按钮，该界面是完成扫码进入后的页面，对应文档中描述的进入界面后点击加号选择小程序目录的步骤相关场景，用于指引用户点击加号以推进后续的小程序目录选择操作。



选择小程序前端工程的 project-xzb-xcx-uniapp-java\unpackage\dist\dev\mp-weixin目录。

AppID：填写申请小程序号获取的AppID。

选择不使用云服务。

如下图：

![qfnwbfzfnopbdyxpgyicobhwnab](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qfnwbfzfnopbdyxpgyicobhwnab.png)
> 图注：图片展示的是小程序开发工具中导入项目的界面。左侧有小程序、小游戏等项目类型选项，当前选中“小程序”。右侧显示项目名称为“project.xzb.yonghuduan.uniapp.vue3”，目录为“D:\\course\\z0202code\\project-xzb-xcx-uniapp-java\\unpack”，AppID为“wxf3dc2b68015f9a43”，后端服务选择“不使用云服务”。界面右下角有“取消”和“确定”按钮。



点击确定进入下边的界面：

![h5r3bgbonog6m1xjqbacodz1n8b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/h5r3bgbonog6m1xjqbacodz1n8b.png)
> 图注：图片展示了在开发环境下测试小程序认证的界面。左侧是手机端界面，显示“云凤到家”登录页面，有“登录”和“快速登录”按钮。右侧是开发工具界面，左侧文件列表中选中“env.js”文件，右侧代码编辑区显示了配置文件内容，其中“baseUrl”被修改为“http://192.168.101:11150”。该图片与文档中部署前端及修改配置文件等操作步骤相关，直观呈现了测试小程序认证时的开发环境界面及配置文件修改情况。

修改project-xzb-xcx-uniapp-java\unpackage\dist\dev\mp-weixin\utils\env.js 配置文件，指定后端网关的地址

![bvskbketmophzzx97ydcqyxwnph](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/bvskbketmophzzx97ydcqyxwnph.png)
> 图注：这是一段代码编辑界面的截图，对应文档中修改env.js配置文件指定后端网关地址的操作步骤，界面左侧显示项目资源管理器，能看到MP-WEIXIN项目下的utils文件夹，打开了其中的env.js文件，代码文件中第2行被高亮标注的内容为const baseUrl = "http://192.168.101.1:11500"，同时下方是开发者工具的调试控制台区域，当前处于Network选项卡，整体是开发环境下配置小程序前端网关地址的操作界面。

设置代理

![hk6ybuywtoth44x9o80c6c4bnrg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/hk6ybuywtoth44x9o80c6c4bnrg.png)
> 图注：图片展示了微信开发者工具界面中“设置”菜单的展开状态。在“设置”菜单下，有“通用设置”“外观设置”“快捷键设置”“编辑器设置”“代理设置”等选项。其中，“代理设置”被红色箭头指向，突出显示。该图片与文档中“设置代理”内容相关，直观呈现了在微信开发者工具中进行代理设置的操作入口位置，辅助用户了解如何在该工具中进行代理设置操作。

![dg9xbmrsao9nqzxwbikcywyanid](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dg9xbmrsao9nqzxwbikcywyanid.png)
> 图注：这张图片展示的是软件的设置界面，顶部导航栏带有“通用”“外观”“编辑器”“代理”等选项，其中“代理”选项为绿色高亮状态，处于选中状态。界面主体是“代理设置”区域，包含三个选择项，当前“不使用任何代理，勾选后直连网络”的选项带有绿色选中标记，另外两个选项为“使用系统代理”和“手动设置代理”，未被选中。该界面对应文档中提到的小程序认证步骤里设置代理的操作环节，是完成小程序认证部署前端过程中进行代理配置的界面。

选择设置--》项目设置，在本地配置中选择不校验合法域名。

![blhlbz1d6oualzxjz6ycypzon1b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/blhlbz1d6oualzxjz6ycypzon1b.png)
> 图注：这张图片展示的是小程序开发者工具的设置界面，处于“本地设置”选项卡页面。界面中“本地设置”选项卡被红色箭头标注突出显示，下方的设置项里，红色框选突出勾选了“不校验合法域名、web-view（业务域名）、TLS版本以及HTTPS证书”选项，该选项是文档中所提及的“不校验合法域名”的核心对应设置项，其余还有调试基础库版本设置、多端适配要求列表、多项编译及运行相关设置选项等内容，该设置是小程序认证部署前端环节中完成本地配置的关键步骤内容。

如果要选择其它目录可以关闭当前项目，重新选择。

![](https://feishu.cn/file/MpmdbhhZFoUS4yx9U3qczVJznBf)



##### 5) 编译运行

小程序认证需要启动的微服务包括：网关jzo2o-gateway、客户管理jzo2o-customer、公共服务jzo2o-publics，保证这三个服务全部启动。

**注意：保证jzo2o-publics服务配置高德地图key（参考：高德地图web服务配置文档）、微信的appid和app密钥。配置完成将jzo2o-publics服务重新启动。**

小程序开发环境配置完成进行编译运行。

首先清除缓存

![poqbbd3awo151bxjhticgs8cnfg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/poqbbd3awo151bxjhticgs8cnfg.png)
> 图注：这张图展示了小程序开发工具的操作界面，当前处于编译相关的功能区，右上角展开了下拉菜单，菜单中包含“清除模拟器缓存”“清除编译缓存”“清除项目文件列表缓存”“全部清除”四个选项，其中“全部清除”选项呈高亮选中状态，界面左侧可看到“资源管理器”栏及js文件“env.js”的编辑区域，顶部有“程序模式”“普通编译”等功能按钮。该图片用于配合文档中“首先清除缓存”的操作说明，对应了清除开发缓存的具体操作选项。

然后编译运行：

![spk1bnd7doevhgxcecuce201npa](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/spk1bnd7doevhgxcecuce201npa.png)
> 图注：图片展示了小程序开发工具界面中编译运行设置区域。关键信息有：左侧“小程序模式”下拉框，右侧“普通编译”下拉框，以及右侧的“编译”“预览”“真机调试”“清缓存”按钮。其中，“普通编译”下拉框被红色箭头指向突出显示。该图片与文档中“编译运行”步骤相关，直观呈现了编译运行操作中选择编译模式的界面位置，辅助用户理解操作流程。

编译运行到登录界面：

![olyybvwb4ot8erxnakcca4ahnbc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/olyybvwb4ot8erxnakcca4ahnbc.png)
> 图注：这张图片是云岚到家一站式家政服务平台的登录界面，界面顶部显示“登录”标识，中间位置有平台的红色四心形图标、“云岚到家”字样及“一站式家政服务平台”的说明文字，最下方是醒目的红色圆角按钮，标注有“快速登录”字样。该界面是小程序认证流程中编译运行后呈现的登录界面，按照流程，点击该“快速登录”按钮后，需依照小程序认证流程操作，请求对应认证接口完成认证。

点击“快速登录”按照前边讲的小程序认证流程进行操作，请求认证接口进行认证，进入调试器-->Network观察请求记录，如下图：

![mpudbnaseoqwzvx6s4mcmdzhn5b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mpudbnaseoqwzvx6s4mcmdzhn5b.png)
> 图注：这是小程序认证测试步骤中，调试器的Network选项卡界面截图，对应测试小程序认证编译运行后观察认证接口请求记录的操作场景。界面顶部选中了Network面板，下方请求列表显示了名称为user和activeRegionList的请求，其中user请求的详情区域展示关键信息：该请求是指向地址为http://192.168.101.1:11500/customer/open/login/common/user的POST请求，状态码显示为200 OK，完全匹配文档中提及的认证接口信息。

认证接口的地址是：/customer/open/login/common/user 

此接口最终从微信拿到用户的openid（微信给用户分配的唯一标识），并将openid存储到数据库，认证通过生成token令牌返回给前端。

认证通过进入下边的界面：

![bglxbbawuokezfxtko6c38cdnuh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/bglxbbawuokezfxtko6c38cdnuh.png)
> 图注：图片展示的是云岚到家小程序的界面。上方显示“河南省”和“日常保洁”搜索框，右上角有三个点和相机图标。中间有新人优惠活动广告，下方提示当前城市暂未开通服务，需切换其他城市进行下单，有“手动选择服务城市”按钮。底部有“首页”“全部服务”“消息”“我的”四个导航栏。该图与上文测试小程序认证流程中编译运行后进入的登录界面相关，展示了认证通过后的界面情况。

##### 6）真机调试(了解)

在开发环境还可以通过手机打开小程序进行测试，下边介绍具体的配置方法，注意此部分内容作为了解，正常开发使用上边介绍的通过微信开发工具进行测试，方便跟踪接口交互数据。

首先保证手机和PC在同一个网络，因为在手机上打开小程序需要访问PC上的微服务接口。

可以让手机和PC连接同一个热点，连接热点后查询无线网卡的IP，如下图：

![xzr7butjuown96x09wgccnyfnbc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xzr7butjuown96x09wgccnyfnbc.png)
> 图注：图片展示的是无线局域网适配器本地连接* 10的网络配置信息。其中，IPv4地址为192.168.137.1，被红色框突出显示。该图片与文档中真机调试步骤相关，用于说明在真机调试时，连接热点后查询无线网卡IP，192.168.137.1是测试环境IP，需保证手机IP与192.168.137.1在同一个网段。

192.168.137.1是我的测试环境，同时要保证手机的IP地址和192.168.137.1在同一个网段。

接下来配置网关地址

![bfkdbflk9ocgwexusd4cuv6vngd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/bfkdbflk9ocgwexusd4cuv6vngd.png)
> 图注：该图片展示了代码编辑器中的env.js文件内容，左侧为项目的文件目录，其中utils文件夹下的env.js文件处于打开状态；文件代码的第3行被红色箭头重点标注，该行内容为定义常量baseUrl，其值为http://192.168.137.1:11500，该IP地址对应文档中提及的测试环境网卡IP，是小程序真机调试环节配置网关地址时的关键参数，用于保证手机与PC处于同一网段，完成后续的小程序认证真机调试相关设置。

设置代理

![u2wdbpeehob4cpxhew6cdjj5nid](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/u2wdbpeehob4cpxhew6cdjj5nid.png)
> 图注：图片展示了微信开发者工具界面中“设置”菜单下的“代理设置”选项。该图片与文档中“设置代理”步骤相关，文档提到在真机调试前需配置网关地址，设置代理是其中一步，此图直观呈现了在微信开发者工具中进行代理设置的操作入口，箭头指向“代理设置”选项，帮助用户明确操作位置，为后续设置代理操作提供指引。

![uz6dbpwfboo1acxssmjccipqntc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/uz6dbpwfboo1acxssmjccipqntc.png)
> 图注：图片展示的是小程序开发中代理设置界面。界面上方有“通用”“外观”“快捷键”“编辑器”“代理”“安全”“拓展”等选项卡，当前选中“代理”。下方有“代理设置”选项，包含三个单选按钮，分别是“不使用任何代理，勾选后直连网络”“使用系统代理”“手动设置代理”，其中“使用系统代理”被选中。该图片与文档中“设置代理”步骤相关，用于说明代理设置的操作界面及当前选中状态。

然后点击预览

![agd6b2m0voi6wmxllfvcgxe2ngf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/agd6b2m0voi6wmxllfvcgxe2ngf.png)
> 图注：图片展示了微信开发者工具中小程序认证的真机调试界面。左侧是手机端微信界面，显示“云宽到家”小程序。右侧是开发者工具界面，有资源管理器、二维码预览等区域，其中二维码预览处显示“已自动过滤无依赖文件：2个”，并有二维码。右上角有“真机调试”按钮被红色箭头指向。该图片与文档中真机调试步骤相关，用于说明生成二维码后在手机微信扫码将在手机上预览的操作。

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

![we1pbn4g4o3d3lxtik5czq5fn7g](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/we1pbn4g4o3d3lxtik5czq5fn7g.png)
> 图注：这是一张展示小程序登录流程时序的示意图，涉及小程序、开发者服务器、微信接口服务三个主体。流程从左侧小程序发起，依次执行四步：先调用wx.login()获取登录凭证code，再通过wx.request()将code发送给开发者服务器；开发者服务器带着appid、appsecret与code，向微信接口服务的登录凭证校验接口发起请求，获取session_key与openid等信息；之后开发者服务器将自定义登录态与openid、session_key关联，最终把自定义登录态返回给小程序，由小程序存入storage保存。该流程与文档中介绍的小程序认证步骤完全对应，清晰呈现了认证各环节的逻辑与数据走向。

根据官方的认证流程我们定义本项目小程序认证的交互流程：

customer工程提供认证接口，publics工程作为一个公共服务提供与微信通信的接口。

前端与cutomer交互不与publics交互。

<whiteboard token="AhLGwOeHdhzsD3b4SuMc0EUWnkf"></whiteboard>

##### 2）阅读代码

下边根据认证流程阅读代码，我们以断点调试的方式跟踪接口交互过程。

###### **提供的小程序认证接口**

在jzo2o-customer服务中提供小程序认证接口。

在前端清除缓存后重新编译运行，点击快速登录，授权获取手机号，请求jzo2o-customer的普通用户登录接口，普通用户登录接口如下：

![lc4lbvrouoscl0xqythcgezwnpe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/lc4lbvrouoscl0xqythcgezwnpe.png)
> 图注：该图片呈现了代码相关的目录与文件结构，显示名为“open”的文件夹，文件夹内包含名为“LoginController”的文件，文件名左侧有代码文件标识“c”，整体清晰展示了认证模块中与小程序登录相关的Controller文件，对应小程序认证流程里普通用户登录环节的代码文件结构，明确了该功能对应的核心代码文件对象。

![zoxtbsbiqo83zrxrrcocj1gqnub](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/zoxtbsbiqo83zrxrrcocj1gqnub.png)
> 图注：这是一段后端Java代码，对应小程序认证流程中普通用户登录接口的实现部分。接口使用@PostMapping注解配置路径为"/common/user"，标注为“c端用户登录接口”，方法为loginForCommonUser，接收LoginForCustomerReqDTO类型的请求体参数，返回LoginResDTO类型结果，方法内调用loginService.loginForCommonUser方法。右侧显示的是loginForCustomerReqDTO的实例信息，其code值为“0b3AYjGaltduVF0qNyFa1teUE62AYjGf”，nickname为“微信用户”，avatar是对应的第三方资源链接，该参数用于后续微信相关的用户认证流程。

###### **请求publics获取openid**

jzo2o-customer服务会请求publics服务申请openid，publics服务与微信进行对接。

publics服务中提供获取openid接口如下：

![hsbrbsyhgoowjjxzughcvniqnih](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/hsbrbsyhgoowjjxzughcvniqnih.png)
> 图注：图片展示了微信登录相关代码中的类结构。其中，InnerMapController、InnerSmsCodeController和InnerWechatController三个类被以蓝色圆圈标识，表明它们是类。InnerWechatController被红色箭头指向，且上下文也提及，强调其在微信登录代码中的重要性。该图片与上文介绍的jzo2o-customer服务请求publics服务申请openid，以及publics服务中提供获取openid接口等内容相关，直观呈现了微信登录代码中涉及的类结构。



![jxjtbf6kwohdpkx75zpceoj9nah](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/jxjtbf6kwohdpkx75zpceoj9nah.png)
> 图注：图片展示的是publics服务中获取openid接口的代码。接口为@GetMapping("/getOpenId")，@ApiOperation注解说明为获取openid，@ApiImplicitParams标注了必填参数code。代码中，通过@RequestParam("code")接收code参数，调用wechatService.getOpenId方法获取openId，最后返回OpenIdResDTO对象。该图片与上文介绍的publics请求微信获取openid接口内容相关，直观呈现了接口实现代码。

###### **publics请求微信获取openid**

publics请求微信获取获取openid 接口如下:

接口描述：https://developers.weixin.qq.com/miniprogram/dev/OpenApiDoc/user-login/code2Session.html 

![lqidbmuhsoexqaxz8zucozybn5c](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/lqidbmuhsoexqaxz8zucozybn5c.png)
> 图注：图片展示的是WechatServiceImpl.class文件中的getOpenid方法代码。该方法接收code参数，通过HTTP工具获取微信jscode2session接口的result，若result中JSONObject的getInt("errcode")不为空，抛出新服务器异常；否则返回JSONObject的getString("openid")。此代码与上文介绍的publics请求微信获取openid接口相关，是接口实现的一部分。

###### **生成token**

**customer收到openid后查询数据库获取用户信息并生成token.**

openid是微信用户在家政o2o平台的唯一标识，首先根据openid查询jzo2o-customer的common_user表，是否存在用户，如果不存在则自动注册用户信息，**用户信息存储到jzo2o-customer数据库的common_user表中**。

![x5fxbire9oithoxcpn0caueqnog](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/x5fxbire9oithoxcpn0caueqnog.png)
> 图注：这张图片展示了Java代码的部分实现逻辑，对应前文“阅读代码”中common_user表相关的小程序认证流程，涉及用户信息存储逻辑。代码中，先根据传入的OpenId查询common_user表，若不存在该用户，会自动将注册后的用户信息存入jzo2o-customer的common_user表，该逻辑被红色框线标出；若用户存在但状态为冻结，则抛出账号被冻结的异常。代码最终会根据处理后的用户信息生成token返回给前端，与上下文生成token的功能流程相契合。

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

![pegrbgacdo7vspxewitcpd6hn7d](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/pegrbgacdo7vspxewitcpd6hn7d.png)
> 图注：图片展示的是小程序认证中生成token的代码。在判断commonUser是否为空或用户状态为冻结后，使用jwtTool.createToken方法生成token，参数包括commonUser的id、昵称、头像及用户类型。最后返回LoginResDTO对象，其中token被红色框突出显示。该图片与上文介绍JWT令牌格式及在JWT令牌中存储用户信息的内容相呼应，直观呈现了生成token的具体代码实现。

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

![e84hbb8nto4anrxz0k3c9rsun1e](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/e84hbb8nto4anrxz0k3c9rsun1e.png)
> 图注：图片展示了J2O2O Gateway项目中TokenFilter.java代码文件的部分内容。左侧是项目结构，显示了gateway包下的TokenFilter等文件。右侧代码中，第62行开始，通过GatewayWebUtils获取请求头中的token，若token为空则返回403状态码；第66行开始，获取tokenKey，若tokenKey为空也返回403状态码。该图片与文档中网关对token统一校验的内容相关，直观呈现了代码实现过程。

我们可以进行断点调试：

如下图访问“我的地址”进行调试：

![ntxqb8tl6owi4sxvczociuzdnng](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ntxqb8tl6owi4sxvczociuzdnng.png)
> 图注：这是小程序“我的”页面界面，页面顶部显示“微信用户”及对应的头像，下方设有“我的订单”板块，包含待支付、待服务、服务中、待评价四个功能入口；页面中“我的地址”选项被红色箭头指向，符合文档中访问“我的地址”进行断点调试的要求，此外页面还设有“我的评价”“我的优惠券”选项，底部有首页、全部服务等导航栏，以及客服电话信息。

进入断点位置：

首先获取前端传入token，网关校验token并解析出用户信息。

![hn6gb7jgoofqksx0z75cmgmpnub](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/hn6gb7jgoofqksx0z75cmgmpnub.png)
> 图注：这张图片展示的是项目的代码编辑界面，左侧是项目目录结构，可见gateway模块下的filter包内包含TokenFilter.java文件，当前打开了该文件的代码内容。代码中包含了校验Token的相关逻辑，其中核心部分是对获取到的Token进行非空判断：当获取的Token为空时，会返回禁止访问的错误提示；若Token非空，则进行后续的业务处理，该代码对应了文档中网关校验Token并解析用户信息的相关流程环节。

![dipabdqvfoalsbxfcljcibovnrg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dipabdqvfoalsbxfcljcibovnrg.png)
> 图注：图片展示的是TokenFilter.java代码中解析token的断点调试界面。代码中使用JwtTool解析token，若解析成功则获取CurrentUserInfo。当前处于断点位置，显示了CurrentUserInfo的属性，包括id、name、avatar和userType等。该图片与上文提到的网关对token统一校验内容相关，直观呈现了解析token后获取用户信息的操作及结果，辅助理解代码执行情况。

解析token通过，将用户信息放入http头中，继续向微服务转发

![jtjmbdofyom31ixzaqtcywipnra](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/jtjmbdofyom31ixzaqtcywipnra.png)
> 图注：图片展示的是网关对token统一校验代码断点调试时的界面。代码中，`GatewayWebUtils.setRequestHeader`方法被调用，将用户信息放入http头中。调试窗口显示了http头信息，其中“USER-INFO”字段被高亮显示，其值为“eyJpZCI6MTcxeXNjM0NjQWNjQwNjA5ODI5Njg2MiwibmFtZSI6Ilu2ruZw==”。该图片与上文提到的断点调试内容相关，直观呈现了调试时http头信息中“USER-INFO”字段的情况。



接下来进入微服务，在微服务中解析http头信息中的用户信息，写入ThreadLocal方便应用程序使用。

源代码如下：

![fsanbugwno49m3xoqydclb9cnmc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/fsanbugwno49m3xoqydclb9cnmc.png)
> 图注：图片展示了jzo2o-mvc项目中UserContextInterceptor.java代码文件的部分内容。代码中定义了preHandle方法，用于在请求处理之前执行。首先尝试获取请求头中的用户信息，若为空则返回true；若不为空，则进行base64解码，将用户信息存入ThreadLocal中。该图片与上文介绍的网关对token统一校验后，将用户信息放入http头中，继续向微服务转发的内容相关，直观呈现了代码实现过程。

"我的地址"接口属于jzo2o-customer微服务，所以我们进入jzo2o-customer微服务，找到此类，在此类打断点。

仍然访问“我的地址”，进入断点，如下图：

![sdtwbrilbocgfaxnbujcsst0nlh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/sdtwbrilbocgfaxnbujcsst0nlh.png)
> 图注：图片展示的是UserContextInterceptor.class文件代码，该文件实现了HandlerInterceptor接口。在preHandle方法中，通过request.getHeader获取“USER - INFO”头信息，若为空则返回true；若非空则解码用户信息并将其转换为CurrentUserInfo对象，再将该对象设置到UserContext中。此代码与文档中网关对token统一校验的内容相关，用于在controller及service方法中方便获取当前用户信息。

用户放入ThreadLocal中，在controller及service方法中非常方便去获取当前用户信息。

我们找到“我的地址”接口的service方法并打上断点

![vxitbvvjvo0pp1xrosecmucbn7f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vxitbvvjvo0pp1xrosecmucbn7f.png)
> 图注：图片展示了“我的地址”接口的service方法代码。代码中，通过UserContext类获取当前用户id，使用LambdaQueryWrapper对AddressBook进行查询，最后调用addressBookMapper的selectPage方法分页查询。图片与上文提到的在“我的地址”接口service方法处打上断点，重新访问“我的地址”进入此断点，发现当前用户id已在微服务中成功获取的内容相关，直观呈现了代码实现情况。

重新 访问“我的地址”进入此断点

![xxavbf0pdoe8chxx3f1c1x6sndf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xxavbf0pdoe8chxx3f1c1x6sndf.png)
> 图注：这张图片展示了小程序认证相关开发场景下的代码断点调试界面，左侧为项目文件目录结构，当前打开的是`jzo2o-customer`项目的`AddressBookController`文件。右侧显示`AddressBookServiceImpl.java`类的`page`方法代码，在第165行的代码处设置了蓝色断点，该行为通过`UserContext.currentUserId`获取当前用户ID并用于后续的Lambda查询操作，与上下文描述的在“我的地址”接口的service方法打断点、获取用户ID的操作内容对应。

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

![vhegbcz3fo7qvjxjykbctvo5ngh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vhegbcz3fo7qvjxjykbctvo5ngh.png)
> 图注：图片展示了HBuilder X的图标，图标为绿色背景，中间有白色字母“H”，下方有白色“HBuilder X”字样。该图片位于文档中“安装成功快捷方式如下”部分，是对HBuilder X安装成功后快捷方式的直观呈现，帮助用户识别并启动HBuilder X，是后续操作的前置条件。

启动HBuilderX

![cdsfbxiwuozzadxsemwch9d8nxd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/cdsfbxiwuozzadxsemwch9d8nxd.png)
> 图注：这张图片是HBuilderX的启动界面，界面顶部显示了菜单栏，包含“文件”“编辑”“选择”“查找”等功能选项，还设有地址栏、搜索框与“预览”按钮。界面中间是带有“H”标识的标志，下方列出六个功能选项，左侧依次为新建项目、新建空白文件、打开目录，右侧依次为主题切换、快捷键方案切换、入门教程，底部状态栏显示“未登录”相关状态，该界面对应文档中“启动HBuilderX”的操作步骤。



下边从课程资料拷贝project-xzb-app-uniapp-java.zip到代码目录并解压

*（可忽略）cmd进入project-xzb-app-uniapp-java目录运行 npm install || yarn  或 cnpm install || yarn  安装依赖包，如下图：*

![r0u8bssbwoidvjxb4i5ceawwnvh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/r0u8bssbwoidvjxb4i5ceawwnvh.png)
> 图注：图片展示的是在Windows系统cmd命令行中运行npm install || yarn命令的界面。命令执行后，显示了npm版本信息及项目名称，还出现了3个npm警告信息，如“read-shrinkwrap”版本兼容性问题等。接着列出3个新增包的贡献者信息，审计了3个包，用时35.949秒，最后提示0个漏洞。该图片与文档中“从课程资料拷贝project - xzb - app - uniapp - java . zip到代码目录并解压”后，运行npm install || yarn安装依赖包的内容相关，展示了安装依赖包后的命令行反馈。



下边用HBuilderX打开project-xzb-app-uniapp-java目录



![afm6bhmz1o01mnxml4kcncvsntc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/afm6bhmz1o01mnxml4kcncvsntc.png)
> 图注：该图片展示了HBuilderX软件的文件菜单界面，其中“打开目录(D)...”选项被蓝色高亮选中，菜单内还包含新建、打开多窗口项目、导入、标签卡、收藏、打开文件等其他操作选项，这一选项用于在HBuilderX中打开指定的项目目录，是文档中“用HBuilderX打开project-xzb-app-uniapp-java目录”这一步骤对应的操作指引图示。

配置网关地址



![lv18br7fgowuf9xffrwcjtg1ngg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/lv18br7fgowuf9xffrwcjtg1ngg.png)
> 图注：图片展示了在HBuilderX中打开的project-xzb-app-uniapp-java目录下的vite.config.js文件内容。文件中配置了vite插件和代理服务器，代理服务器配置了目标地址为“http://192.168.101.1:15000”，changeOrigin为true，rewrite路径规则为“return path.replace(/^\\/api/, '')”。图片与上文配置网关地址的内容相关，直观呈现了配置代码，帮助用户了解如何在HBuilderX中配置网关地址。

配置完成，使用HBuilderX运行到浏览器

![tgtkb3ammo5o42xhbvpcuqyantg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/tgtkb3ammo5o42xhbvpcuqyantg.png)
> 图注：这张图片展示了HBuilderX软件的操作界面，显示启动软件后，点击顶部菜单栏的“运行”选项，在弹出的下拉菜单中选择“运行到浏览器”，进而选中Chrome浏览器的操作，这对应文档中“配置完成，使用HBuilderX运行到浏览器”的步骤，用于指导用户在配置好相关内容后，选择Chrome浏览器来运行项目，推进手机验证码认证模块测试的相关操作。

运行成功进入登录页面：

![ifg6ba14yoayxgxoirmctj7knqc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ifg6ba14yoayxgxoirmctj7knqc.png)
> 图注：这张图片是一个网页登录页面，顶部地址栏显示访问地址为localhost:8080/#/pages/login/user，页面顶部有“云岚到家”的标识及“一站式家政服务平台 服务端”的说明，页面中间设有两个输入框，分别用于输入手机号与验证码，右侧配有“获取验证码”按钮，下方还有一个粉色的“登录”按钮。页面底部有红色的提示文字，说明该页面仅用于IT培训教学使用，提醒用户为保障信息安全，切勿向平台录入任何个人敏感信息，包括手机号、身份证号等。

下边进入调试模式

![dixdbuha2okhq7xw6ijcpvzsn5i](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dixdbuha2okhq7xw6ijcpvzsn5i.png)
> 图注：这张图片展示了浏览器右键菜单的界面，其中红色箭头重点指向“检查”选项，该选项的快捷键为相关标识。该图片所在文档对应手机验证码认证测试的操作步骤上下文，结合前后步骤的调试要求，这一界面是进入浏览器开发者工具的操作入口，和文档中“进入调试模式”的操作指引相匹配，是调试过程中查看网页相关内容的关键操作选项。

选择布局方式：

![ey3kb6z7jo5zynxprv6ci03cnpg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ey3kb6z7jo5zynxprv6ci03cnpg.png)
> 图注：图片展示了浏览器开发者工具的界面，其中“Dock side”选项被红色箭头指向突出显示。该界面处于“Elements”标签下，右侧有“Styles”和“Computed”两个选项卡。界面右上角有多个图标，如“X”关闭按钮、“齿轮”设置按钮等。此图与文档中“选择布局方式”内容相关，可能是用于说明在浏览器开发者工具中选择布局方式的操作步骤或界面展示。

打开Network调试窗口：

![x2uablhd8opwmdxtkjkczu8nnae](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/x2uablhd8opwmdxtkjkczu8nnae.png)
> 图注：这张图展示的是登录操作后的页面，左侧是云岚到家一站式家庭服务平台的登录界面，包含请输入手机号、请输入验证码输入框、获取验证码按钮和粉色的登录按钮；右侧是浏览器开发者工具界面，顶部有“Always match Chrome's language”“Switch Dev Tools to Chinese”等选项，Network标签处于选中状态，该界面用于后续调试，符合文档中进入调试模式、打开Network调试窗口的操作步骤。



如果修改了文件需要重新编译运行在HBuilderX控制台右键

![qo5qbmdqiohqhvxh5fjclg7mn3f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qo5qbmdqiohqhvxh5fjclg7mn3f.png)
> 图注：图片展示了HBuilderX控制台右键菜单中的“重新运行”选项。该选项以蓝色高亮显示，其快捷键为Ctrl+F5。上下文提到，如果修改了文件需要重新编译运行，在HBuilderX控制台右键选择此选项即可，这与文档中调试模式操作步骤相关，是调试过程中重新运行代码的快捷操作指引。

##### 2）认证测试

下边测试手机验证码认证流程。

首先输入手机号，**服务人员的信息存储在jzo2o-customer数据库的serve_provider表中**，从表中找一个手机号录入 

点击发送验证码，此时前端请求后端发送验证码，在开发环境我们从控制台获取验证码，稍后后带大家分析发送验证码的程序。

![i6tmbxbztoj5ylxintrctfngnmb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/i6tmbxbztoj5ylxintrctfngnmb.png)
> 图注：图片展示的是云岚到家服务端的认证界面。界面上方显示“云岚到家”及“一站式家政服务平台 | 服务端”字样。中间有手机号码“15066699132”，下方有“请输入验证码”输入框和“获取验证码”按钮，按钮旁有红色箭头指向。最下方是“登录”按钮。该图片与文档中测试手机验证码认证的上下文相关，用于展示在进行手机验证码认证时的登录界面。



注意此时因为请求后端发送验证码我们观察在浏览器的Network窗口有一条记录，如下图，该请求必须响应状态为200方可正常发送验证。

![gjibbisepoaxmvxpm3wcnqlbntf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/gjibbisepoaxmvxpm3wcnqlbntf.png)
> 图注：这张图片展示的是浏览器开发者工具的Network面板界面，用于查看接口请求记录。界面中，一条名称为“send”的xhr请求被蓝色选中，箭头明确指向该请求的Status列，标注该请求的响应状态码为200，请求发起方为“uni-h5.es.js:19528”，数据包大小为343B，响应耗时21ms。该图片是文档中说明“请求后端发送验证码的记录”的配套图示，直观呈现了发送验证码请求需达到响应状态200才能正常生效的验证要求。

从控制台获取刚才发送的验证码

![nxtdbkgyiohf39xxtzsciln6nib](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/nxtdbkgyiohf39xxtzsciln6nib.png)
> 图注：图片 addCriterion图片展示了手机验证码认证测试中从控制台获取的验证码信息。上方显示向手机号15066699132发送验证码296145。下方result部分，code为200，msg为“OK”，data为空。该图片与上文提到的从控制台获取验证码内容相关，直观呈现了获取验证码后的结果，为后续将验证码填入验证码输入框并进行登录认证做准备。

将从控制台获取的验证码填入验证码输入框

![j5y8b5afyo8nixxeicacxojrnac](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/j5y8b5afyo8nixxeicacxojrnac.png)
> 图注：图片展示的是云岚到家一站式家政服务平台的登录界面。界面上方显示平台名称及“服务端”标识。中间有手机号输入框，已输入“15066699132”，下方验证码输入框内有数字“296145”，右侧有“获取验证码”按钮。最下方是红色的“登录”按钮。该图片与文档中测试手机验证码认证的上下文相关，用于说明在登录界面输入验证码的操作步骤。

点击登录进行认证，认证过程会先校验验证码是否正确，如果验证码正确再根据手机号查询serve_provider表是否存在相应记录且用户未被冻结，全部成功则认证通过。

认证通过进入首页。

![ryqobz5utokzplxoj7oco8kynsh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ryqobz5utokzplxoj7oco8kynsh.png)
> 图注：图片展示的是家政服务APP首页界面。左上角显示“获取失败”，下方有“服务技能”“接单设置”“服务范围”三个选项。中间部分有“更好的家政 上门服务体验”及“一个靠谱的家庭服务平台”字样，背景为一位穿红围裙的女性正在擦桌子。下方有“抢单大厅”“距离筛选”“服务类型”三个筛选项。底部导航栏有“首页”“订单”“消息”“我的”四个选项。该图片与文档中认证通过进入首页的内容相关，展示了认证成功后的首页界面。



#### 1.3.2 阅读代码

##### 1）手机验证码认证流程

customer工程提供认证接口，publics工程作为一个公共服务提供与发送验证码接口。

<whiteboard token="VvaTw1sEDhB9bHb4ZNccE1qinEd"></whiteboard>



##### 2)  阅读代码

###### **找到具体的接口**

前端请求publics服务发送验证码接口：publics/sms-code/send

代码如下：

![ylnhbauafoypqbxhfobcxjxinef](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ylnhbauafoypqbxhfobcxjxinef.png)
> 图注：图片展示了jzo2o-publcs项目的目录结构，重点突出SmsCodeController类。该图片与文档中“找到具体的接口”部分内容相关，上下文提到前端请求publics服务发送验证码接口为publics/sms - code/send，代码在SmsCodeController中。图片中红色箭头指向SmsCodeController，直观呈现了代码所在位置，辅助理解前端请求发送验证码接口的实现位置。

![pjmab8muvo1uffxlvgccm7chnlr](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/pjmab8muvo1uffxlvgccm7chnlr.png)
> 图注：这张图片展示的是发送短信验证码接口的Java代码片段，其中该接口使用@PostMapping注解映射路径为"/send"，@ApiOperation注解的说明为“发送短信验证码”。代码定义了一个名为smsCodeSend的公共方法，该方法接收SmsCodeSendReqDTO类型的请求体参数，方法内调用了smsCodeService的同名smsCodeSend方法，传入上述请求体参数，对应前文提及的前端请求publics服务发送验证码的接口publics/sms-code/send。

具体发送验证码逻辑：

![uucdbihxlobrxvxrvogcaw9sntg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/uucdbihxlobrxvxrvogcaw9sntg.png)
> 图注：这张图片展示的是Java代码文件SmsCodeServiceImpl.java中的部分代码内容，属于手机验证码认证的发送验证码具体逻辑代码。该代码首先对输入参数手机号和业务类型进行非空校验，若任一参数为空则返回；接着生成包含手机号的Redis键，通过随机函数生成6位验证码；之后记录发送验证码的日志，还留有调用短信平台发送验证码的待办注释，同时将生成的验证码存入Redis，设置有效期为300秒。该代码对应上下文提及的发送验证码逻辑部分，用于实现手机验证码发送的核心程序逻辑。



前端请求customer服务的认证接口：/customer/open/login/worker

代码如下：

![b14ibupm7otqe2xebsjcklhzndc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/b14ibupm7otqe2xebsjcklhzndc.png)
> 图注：这张图片展示了项目jzo2o-customer的代码目录结构，从.idea、src、main逐级展开至java层级的com.jzo2o包，可见该包下包含customer、client、config等多个子包，其中customer包内的open子包下存在LoginController，图片中标注的蓝色部分明确指向该LoginController文件。结合上下文信息，这个LoginController对应的是手机验证码认证中，前端请求customer服务的认证接口所在的代码文件，该接口用于区分服务人员或机构的身份认证，图中所指的LoginController是服务人员认证的相关代码方法所在文件。

机构和和服务人员认证接口是同一个，根据类型判断是机构还是服务人员，下图中红色箭头指向的代码是服务人员认证的方法。

![l5o7bk4j7oxwplxhtezcjiqrnzb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/l5o7bk4j7oxwplxhtezcjiqrnzb.png)
> 图注：图片展示的是一个Java代码片段，用于服务人员/机构人员登录接口。代码中，当用户类型为机构时，调用loginService的loginForPassword方法；若用户类型为服务人员，则调用loginService的loginForVerify方法（代码中用红色箭头指向）。该代码与文档中介绍的机构和服务人员认证接口相关，红色箭头所指的代码是服务人员认证的方法，体现了文档中提到的根据类型判断是机构还是服务人员，再调用相应认证方法的逻辑。

customer服务请求publics服务校验验证码

![nlgybvgysotrxwxkpnpcqzlonqc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/nlgybvgysotrxwxkpnpcqzlonqc.png)
> 图注：这是一段Java代码，显示在名为ILoginServiceImpl.java的文件中，对应客服人员认证的方法。代码里核心部分是被红色方框标注的内容，这部分用于远程调用publics服务的接口校验验证码是否正确，其中调用的verify方法传入了用户手机号与SmsBusinessTypeEnum.SERVE_STAFF_LOGIN参数，以此完成与publics服务的交互，在校验不通过时会抛出验证码错误的异常。该代码对应了文档中机构与服务人员认证共用接口、按类型区分服务人员认证逻辑的说明，红色箭头指向的部分正是服务人员认证的关键方法实现。

publics服务提供校验验证码接口，如下图：

![nc4tbkpr2o1t26xs1dwcg1qmnsh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/nc4tbkpr2o1t26xs1dwcg1qmnsh.png)
> 图注：这张图片是代码编辑界面截图，展示了项目中与校验短信验证码相关的代码内容，左侧的项目结构里，“jzo2o-publics”项目下的“controller.inner”目录中选中了“InnerSmsCodeController.java”文件，右侧该文件的代码区域里，有红色箭头指向名为“verify”的方法，此方法带有“@GetMapping("/verify")”的注解，该方法属于publics服务的校验短信验证码接口，用于处理验证码校验相关请求，和文档中提到的publics服务提供的校验验证码接口内容对应。

###### **验证码证码逻辑**

具体的验证码校验逻辑是先查询redis中的正确的验证码，再和用户输入的进行对比，如果不一致则说明输入错误，输入正确删除验证码。如下代码：

![x4yrbuw8lowfe6xwgwscbpyknge](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/x4yrbuw8lowfe6xwgwscbpyknge.png)
> 图注：图片展示了jzo2o项目中SmsCodeServiceImpl.java文件的代码。重点突出了一段用于手机验证码校验的代码，包含验证前准备和短信验证两部分。验证前准备部分，通过格式化字符串获取redisKey，再从redisTemplate中获取验证码值；短信验证部分，先判断验证码是否为空且与redis中的验证码相等，若相等则删除redisKey，最后返回验证结果。该代码与上文提到的验证码校验逻辑相呼应，直观呈现了具体实现。

在使用redisTemplate时需要在工程中引入下边的依赖：

```Java
<dependency>
    <groupId>com.jzo2o</groupId>
    <artifactId>jzo2o-redis</artifactId>
</dependency>
```

在jzo2o-redis中定义了redisTemplate的定义，如下图：

![jxxfbezu3ompoyxr56gc5sunnae](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/jxxfbezu3ompoyxr56gc5sunnae.png)
> 图注：这张图片展示的是jzo2o-redis项目中的RedisConfiguration.java代码文件，具体为名为restTemplate的方法代码块，该方法被标注了@Bean和@Primary注解，用于定义一个RedisTemplate实例。代码中实现了RedisTemplate实例的创建，还对连接工厂进行配置，设置了键序列化器、值序列化器、哈希键序列化器、哈希值序列化器，同时输出了RedisTemplate的哈希码日志。这段代码的作用是定义项目所需的RedisTemplate实例，与文档中提及的在jzo2o-redis中定义redisTemplate的内容相符，能够满足后续业务中对Redis的使用需求。

使用redisTemplate时只需要在bean里注入上图中定义的redisTemplate即可。

在测试验证码发送时可以打开redis进行跟踪，下图显示了存入redis中的验证码，注意观察key和value：

![ih4kbsjlkot6fmxuadtcswvcnif](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ih4kbsjlkot6fmxuadtcswvcnif.png)
> 图注：图片展示了在Redis中存入的验证码信息。键值为“PHONE:CODE:VERIFY_CODE_15066699132_3”，大小为8字节，值为“738625”。图片与上文提到的“在测试验证码发送时可以打开redis进行跟踪，下图显示了存入redis中的验证码，注意观察key和value”相呼应，直观呈现了存入redis的验证码key和value，帮助理解验证码存储在Redis中的形式。



###### **自动注册**

校验验证码完成customer服务根据手机号查询数据库，如果用户冻结则认证失败，如果用户不存在则自动注册。

![ckdhbcb3todo8wxfuorc30nynbg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ckdhbcb3todo8wxfuorc30nynbg.png)
> 图注：图片展示的是ILoginServiceImpl.java文件中关于手机验证码认证流程的部分代码。关键代码部分用红色框线突出显示，其中包含服务人员和机构信息的查询逻辑，即通过手机号和用户类型获取serve_provider表中的人员或机构信息。代码还对用户状态进行判断，若用户冻结则认证失败，若用户不存在则自动注册。该图片与上文介绍的自动注册流程相关，直观呈现了代码实现。

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

![ofolb4siroccwcxja2mcfktrneg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ofolb4siroccwcxja2mcfktrneg.png)
> 图注：图片展示的是手机验证码认证流程中生成登录token的代码。在登录校验、服务人员或机构信息获取、账号状态校验及自动注册完成后，代码调用jwtTool.createToken方法，传入服务人员ID、名称、头像等信息，生成token后返回LoginResDTO对象。该图片与上文介绍的手机验证码服务实现方案相关，直观呈现了生成token的具体代码实现，是手机验证码认证流程中生成token环节的代码示例。





##### 3）小结

手机验证码服务的实现方案？



为了方便验证码测试前端会自动填入123456，更改验证码发送接口固定生成的验证码为123456.

![xdzbbqepgozkcmxmag2ciozrnok](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xdzbbqepgozkcmxmag2ciozrnok.png)
> 图注：这张图片展示了一段Java代码，属于手机验证码发送相关的逻辑。代码中定义了SmsCodeSend方法处理验证码发送请求，当手机号或业务类型为空时会返回并打印调试日志。代码以手机号为key生成Redis存储的键，原本的逻辑是生成6位随机数作为验证码，而图中被红色箭头标注的部分是固定设置验证码为123456的内容，该设置是为了方便验证码测试。完成验证码设置后，代码会将验证码存入Redis，设置有效期为300秒，用于后续的验证码校验环节。





## 2 用户端定位

### **1) 用户端首页定位**

用户端在小程序认证通过后会自动进行定位，也可以在首页手动定位，**定位成功后用户进入首页会根据定位的城市查询该城市有哪些服务项目**。

手动定位过程如下图：

![ltnsbjtwbogzpyxuw6kcqtdsnxh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ltnsbjtwbogzpyxuw6kcqtdsnxh.png)
> 图注：这张图片是“云岚到家”小程序的用户端首页，呈现的是用户端手动定位的相关界面。界面顶部显示小程序名称为“云岚到家”，在页面上方的定位栏中，当前定位显示为“北京市”，还有红色箭头指向该定位栏的位置，对应上下文里点击该箭头位置进行手动定位的操作。页面下方还展示了“新人优惠”相关的保洁服务活动，内容为全屋除菌保洁3次低至459元，附带活动相关的配图和“立即抢购”按钮，用于吸引用户参与活动。

点击下图箭头位置进行手动定位。

![we2rbmwnsosckcxf1enc0frvntf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/we2rbmwnsosckcxf1enc0frvntf.png)
> 图注：这是小程序选择服务地址的界面，界面顶部为“选择服务地址”的标题，当前定位显示当前城市为郑州市。界面中用红色箭头和文字标注出两个操作方式：一是点击带有“重新定位”标识的位置，可完成手动定位；二是也可直接选择“已开通的城市”区域中标记为郑州市的选项，来切换服务地址，明确展示了该用户端首页的地址选择操作指引。



定位成功再次进入首页发现位置变为新地址，如下图：

![moambauepoj8e8xftizcoeawn8d](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/moambauepoj8e8xftizcoeawn8d.png)
> 图注：该图片是云岚到家小程序的用户端首页，页面顶部为“云岚到家”的小程序名称，下方的定位栏处，红色箭头指向的“郑州市”是当前显示的用户位置；定位栏内还标注了“日常保洁”，其下方是新人优惠广告区域，展示了全屋除菌保洁3次低至459元的优惠信息，还有“立即抢购”按钮；页面底部则标注了平台“给您贴心的专业的上门服务”“标准定价 售后无忧”的服务相关说明。

### 2）高德地图配置

小程序端的定位是通过手机的定位模块进行定位，定位成功获取经纬度坐标，平台根据经纬度坐标请求地图服务获取经纬度坐标对应的具体位置。

小程序首先通过微信提供的方法拿到经纬度坐标，然后请求后端获取具体的位置，后端会请求高德地图根据经纬度获取具体的城市信息。

要测试用户端定位的流程首先需要在高德地图开通地图定位服务。

参考“高德地图web服务配置文档” 获取访问接口的key。

进入nacos配置jzo2o-publics.yml中高德地图key。

![ccfebz6geobxsnx5a4ackmswnph](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ccfebz6geobxsnx5a4ackmswnph.png)
> 图注：图片展示的是nacos配置界面中jzo2o - publics.yml的配置内容。其中，配置格式选择了YAML。在配置内容部分，有“amap”配置项，其下“enable”值为true，“key”值被红框突出显示，为0547c66c48a1e589f945e2554dc...。该图片与文档中“小程序端的定位是通过手机的定位模块进行定位，定位成功获取经纬度坐标，平台根据经纬度坐标请求地图服务获取经纬度坐标对应的具体位置”等内容相关，用于说明小程序端获取高德地图key的配置位置。

### 3) 测试定位

启动jzo2o-publics服务。

启动小程序，先清除缓存再进行编译

![kjgybloe9osfysxzfghc20lenzv](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/kjgybloe9osfysxzfghc20lenzv.png)
> 图注：图片展示了小程序开发工具中编译选项的设置界面。画面中“编译”按钮被红色箭头指向，点击后弹出下拉菜单，其中“全部清除”选项被红色框突出显示。该图片与文档中“启动小程序，先清除缓存再进行编译”内容相关，用于指导用户在启动小程序前，先点击“编译”按钮，再选择“全部清除”以清除缓存，以便后续编译操作。

启动成功，点击“快速登录”，先同意服务条款，再允许获取位置信息，如下图：

![c6tfbzsflovxq3x03aychabsnec](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/c6tfbzsflovxq3x03aychabsnec.png)
> 图注：图片展示的是小程序启动成功后，点击“快速登录”时弹出的获取位置信息提示框。提示框上方显示“jzo2o项目 申请”，中间文字为“获取你的位置信息”，下方有“获取您的位置信息”字样，底部有“拒绝”和“允许”两个按钮，其中“允许”按钮为绿色。该图片与文档中启动小程序后点击“快速登录”、同意服务条款及允许获取位置信息的操作步骤相关，直观呈现了获取位置信息的界面。

如果未弹出获取位置提示框，在本地设置中将调试基础库版本设置为2.24.7

![vxpubanfso9fkvxwiszcjuqdndg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vxpubanfso9fkvxwiszcjuqdndg.png)
> 图注：图片展示了小程序开发调试界面。左侧是小程序页面，显示“云购到家”页面，有“立即体验”按钮。右侧是调试工具界面，左侧为代码编辑区域，显示部分代码。右侧上方有“Network”标签，下方是请求列表，显示了多个请求项，如“/index.js”等。右侧下方“调试基础库”处显示版本为2.24.7，有“清除”和“重置”按钮。

点击“允许”，观察Network，请求定位接口/publics/map/regeo?location=，如下图：

![hfu8bi8l1ond77xndx1clpuxn7e](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/hfu8bi8l1ond77xndx1clpuxn7e.png)
> 图注：图片展示的是微信开发者工具调试器中Network界面的“Response”标签页。界面显示了“regeo?location=116.23128%2C40.22077”请求的响应内容，其中“data”字段包含城市信息，如“cityCode”为“010”，“province”为“北京市”，“city”为“null”，“district”为“昌平区”。该图片与文档中测试定位步骤相关，用于观察Network请求定位接口返回的经纬度对应城市信息，以验证小程序获取位置信息功能的正确性。

/publics/map/regeo接口返回经纬度对应的城市信息。

完整内容如下：

```Java
{"code":200,"msg":"OK","data": {"cityCode":"010","province":"北京市","city":null,"district":"昌平区","fullAddress":"北京市昌平区城北街道中共北京市昌平区委员会北京市昌平区人民政府"}}
```

在首页左上角发现定位到北京市，如下图：

![jieib16upowpskx1gv4ckbrfnhb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/jieib16upowpskx1gv4ckbrfnhb.png)
> 图注：图片展示的是“云岚到家”小程序首页界面。界面上方显示时间为18:10，电池电量为98%。中间位置有“云岚到家”字样，下方有搜索框，显示定位为“北京市”，并有“日常保洁”搜索提示。下方有新人优惠活动广告，内容为全屋除菌保洁3次低至459元，包含全屋保洁、除菌打扫、全面保障等服务，还有“立即抢购”按钮。该图片与上文测试定位接口返回位置信息的内容相关，直观呈现了小程序首页及定位情况。



在不同位置进行测试接口返回的位置信息会和讲义不一致，这个需要注意。

到此说明定位测试通过。

下边介绍一种虚拟定位的设置，这个在开发中经常使用，虚拟定位即不是按手机位置进行定位，比如：你在北京，想测试定位到郑州某个位置该如何操作呢？

在微信开发环境可以指定小程序虚拟定位。

首先打开虚拟定位，指定经纬度，如下图：

![lppzbhpmloj2mnxbqcgcvxtxnbd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/lppzbhpmloj2mnxbqcgcvxtxnbd.png)
> 图注：图片展示的是微信小程序调试器中Sensor标签下的Geolocation配置界面。界面中有“Enable”开关，当前处于开启状态。下方显示经纬度坐标，纬度为34.572033，经度为113.838919；还有速度、精度、海拔、垂直精度、水平精度等参数，部分参数值为-1、65、0、65、65。图片与上文介绍的用户端定位交互流程中小程序获取经纬度坐标的内容相关，直观呈现了获取经纬度坐标的操作界面。

经纬度坐标可以使用高德地图的坐标拾取工具获取：

注册高德地图，进入https://lbs.amap.com/tools/picker，即可输入一个具体的位置拿到经纬度，将下图中获取的经纬度设置到上图的位置，清除缓存重新编译小程序即可按虚拟定位去定位。



![y0webmg65oeizzxgax7crfygnig](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/y0webmg65oeizzxgax7crfygnig.png)
> 图注：该图片展示的是高德地图坐标拾取工具页面，网址为lbs.amap.com/tools/picker。页面中“按关键字搜索”选项已被选中，搜索框内输入了“北京黑马程序员”，点击搜索后，右侧“坐标获取结果”处显示该位置的经纬度为116.343314,40.06023，该经纬度可用于小程序的虚拟定位设置，按照文档步骤将其设置后，清除缓存重新编译小程序，就能实现小程序按此虚拟经纬度进行定位。此外页面中还能看到地图上显示了对应位置的地理环境，以及另一个经纬度116.295249,40.090586，该工具是文档中说明的用于获取经纬度的指定工具。





### 4）用户端定位交互流程

<whiteboard token="GaWRwwlf4hvTG5bBMZ4cqN4fnFg"></whiteboard>

1. 小程序获取经纬度坐标
2. 前端请求publics服务根据经纬度查询地理编码（cityCode）
3. publics服务调用高德地图接口根据经纬度查询地理编码
4. 前端根据cityCode找到系统中对应的区域ID



### 5) 阅读代码

定位过程中小程序请求publics服务的接口查询经纬度对应的位置信息，调用/publics/map/regeo接口，下边阅读/publics/map/regeo接口。

![k6u0bn7emodiosxteczcdtynn9c](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/k6u0bn7emodiosxteczcdtynn9c.png)
> 图注：这张图片展示了一段Java代码内容，位于MapController.java文件中，属于第三章用户认证下的用户端定位模块的阅读代码环节。代码内的核心内容为：通过@GetMapping注解配置了"/geo"的请求路径，@ApiOperation注解标注该接口的作用是“根据经纬度查询地理位置信息”，接口方法getCityCodeByLocation使用@RequestParam接收“location”参数，调用mapService的getCityCodeByLocation方法获取地理编码相关数据，并通过BeanUtil工具转换为MapLocationResDTO类型对象返回，代码左侧显示了相关项目的目录结构。

接口文档：http://localhost:11503/publics/doc.html#/default/%E5%9C%B0%E5%9B%BE%E6%9C%8D%E5%8A%A1%E7%9B%B8%E5%85%B3%E6%8E%A5%E5%8F%A3/getCityCodeByLocationUsingGET 



通过mapService的getCityCodeByLocation方法调用高德地图的查询地理编码接口，如下图：

![q8azbn3khofr1pxc1l8cquqanth](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/q8azbn3khofr1pxc1l8cquqanth.png)
> 图注：图片展示了MapController.java文件中getCityCodeByLocation方法的代码。该方法接收一个String类型的location参数，通过将“location”和“key”参数添加到params中，再调用HttpRequest.get方法获取高德地图查询地理编码接口的JSON数据。接着，通过JSONUtil.parseObj方法解析JSON数据，获取province、city、district和fullAddress等信息，最后返回包含这些信息的MapLocationDTO对象。此代码与文档中介绍的通过mapService调用高德地图查询地理编码接口的内容相关。

高德地图的查询地理编码接口定位如下图：

![xurtbqw5koy3u2xmp7xc2vv4n2g](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xurtbqw5koy3u2xmp7xc2vv4n2g.png)
> 图注：该图片展示了高德开放平台Web服务API页面中的逆地理编码开发指南内容，页面明确了逆地理编码API服务的相关信息，其服务地址对应的URL为https://restapi.amap.com/v3/geocode/regeo?parameters，请求方式为GET。该页面的请求参数板块，列出了必填参数key和location，其中key为用户在高德地图官网申请Web服务API类型的密钥，location为传入的经纬度坐标，规则为经度在前、纬度在后，用逗号分隔且经度值不超过6位，这两项参数均为必填项，无默认值。该页面内容对应文档中提及的高德地图查询地理编码接口，是用户端定位相关接口代码调用所指向的参考内容。

接口文档地址：https://lbs.amap.com/api/webservice/guide/api/georegeo，大家可自行阅读。

最终接口返回省、市、县、详细位置等信息，如下：

![oancbulnooyc0fxvnzfcagefnre](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/oancbulnooyc0fxvnzfcagefnre.png)
> 图注：这张图片展示的是用户端定位功能中，调用高德地图查询地理编码接口后返回的响应参数详情。响应参数包含msg、code和data三个核心部分，其中msg是响应消息，code是状态码，200表示成功、其他值代表失败，data为地图定位响应信息，其下属还包含city、cityCode、province、district、fullAddress等子参数，分别对应市、城市编码、省、区、详细地址，各参数均标注了对应的类型和schema，清晰呈现了接口返回的定位相关信息结构。

**为什么要调用高德地图的查询地理编码接口呢？**

在foundations数据库的region区域表中有一列是city_code，高德地图返回的详细信息中city_code与region区域表中city_code是一致的，如下图：

![qggbbwxzkofwnpxfviacthr6nlc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qggbbwxzkofwnpxfviacthr6nlc.png)
> 图注：图片展示了数据库中region区域表的部分内容。左侧为数据库结构，选中了region区域表。右侧表格中，id列显示了不同记录的id，city_code列以红色框突出显示，对应记录的city_code值分别为0997、022、0556等。该图片与上下文紧密相关，用于说明在foundations数据库的region区域表中有一列是city_code，通过调用高德地图的查询地理编码接口，可将小程序拿到的经纬度坐标转换为city_code，进而关联到平台具体的区域。

小程序拿到的是经纬度坐标，通过调用此接口就可以根据经纬度得到city_code从而关联到平台具体的区域。

#### **region表的city_code从哪来？**

先从从高德地图下载拿到全国的区域信息，包括了city_code（https://lbs.amap.com/api/webservice/download）

将下载得到AMap_adcode_citycode.xlsx文件处理为json文件由前端进行保存。

前端在添加区域时从该json文件中选择区域，如下图：

![xwezbogneo9qxkxyp6gccmaknad](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xwezbogneo9qxkxyp6gccmaknad.png)
> 图注：图片展示的是用户端在添加区域时的“新建区域”界面。左侧有“地区选择”“区域负责人”“手机号码”三项必填项，右侧是下拉选择区域的列表，当前选中“河南省”，其下有新乡市、信阳市、许昌市、郑州市等选项。该图片与上下文紧密相关，上下文提到前端在添加区域时从处理后的json文件中选择区域，此图直观呈现了前端选择区域时的界面及操作方式，辅助说明区域信息的添加流程。

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

![pphtbs9xrotcirx6m2vcfxodn9b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/pphtbs9xrotcirx6m2vcfxodn9b.png)
> 图注：图片展示的是vite.config.ts文件内容，其中关键部分是proxy配置。在62行，配置了目标地址为'http://localhost:11500'，changeOrigin为true，rewrite规则为path.replace(/^\\/api，''），还设置了两个备用地址。该图片与文档中修改后端地址的内容相关，直观呈现了修改后的vite.config.ts文件中proxy配置的具体情况，帮助理解如何配置后端地址。

安装依赖包（如果已经安装依赖包则不用安装）：

cmd进入project-xzb-PC-vue3-java目录运行 ：npm install || yarn  或 cnpm install || yarn

安装依赖包完成运行：npm run dev 运行前端工程，如下图：

![mbhibcqzioz7c9xwxr1czfq9nwd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mbhibcqzioz7c9xwxr1czfq9nwd.png)
> 图注：该图片展示了Windows PowerShell的命令行界面内容，显示前端工程运行成功的相关信息，对应文档中“运行前端工程”的步骤。界面中明确标注了vite版本为v2.9.15，提示dev server已运行，给出了Network下的三个访问地址：http://172.16.43.194:6001/、http://192.168.27.1:6001/、http://192.168.101.1:6001/，以及Local地址http://localhost:6001/，同时标注运行准备完成，耗时825ms，与文档中“运行前端工程后需打开http://localhost:6001/”的内容相呼应。

 运行成功自动打开浏览器：http://localhost:6001/

![duikbqmo2oy9wuxdtjjcv5o0nxe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/duikbqmo2oy9wuxdtjjcv5o0nxe.png)
> 图注：这张图片是云岚到家一站式家政服务平台的登录界面，访问地址为localhost:60002/login相关地址。界面左侧区域设有账号输入框、密码输入框和红色“登录”按钮，还附带了“没有账号？去注册”“忘记密码”的相关选项，以及说明该页面仅用于介绍教学使用、提示注意个人信息安全的文字内容；界面右侧配有一名身着家政服饰的女性的形象图，搭配清洁相关工具，对应该家政服务平台的定位。

前端默认的账号：15896123123，密码为：888itcast.CN764%...

机构信息存储在jzo2o-customer数据库的serve_provider表中，可从serve_provider表获取账号。

机构端账号密码认证接口请求customer服务的接口：

代码如下：

![xuaxbjiyto9vrfxkyeqclidenbd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xuaxbjiyto9vrfxkyeqclidenbd.png)
> 图注：图片展示的是LoginController.java代码文件内容。文件中定义了ILoginService loginService变量，以及一个@PostMapping注解的方法loginForWorker，用于服务人员/机构人员登录。当传入的loginForWorkReqDTO的userType为INSTITUTION时，调用loginService.loginForPassword方法；若为SET，调用loginService.loginForVerify方法。该图片与上文机构端账号密码认证接口请求customer服务的接口代码相关，直观呈现了代码实现。

登录成功：

![exytbxm0eokkuox1eyqcwz3unqc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/exytbxm0eokkuox1eyqcwz3unqc.png)
> 图注：该图片为云岚到家机构端的工作台界面，是前端工程运行后登录成功的页面。界面顶部显示地址栏与操作按钮，左侧为工作台、订单管理等功能菜单；核心区域展示了总收入、总退款、订单总数三类数据，均标注对应累计订单金额，同时呈现了2023年11月28日的订单数统计折线图；右侧设有基础设置、服务范围设置等模块，显示今日待分配、待上工、待完工、已完成的订单数量，以及全部订单总数，下方还列出订单信息的相关入口。

退出按钮如下：

![ohorbw7eeompdwx4uohcxt95nkf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ohorbw7eeompdwx4uohcxt95nkf.png)
> 图注：这是云岚到家一站式家政服务平台机构端的操作界面，界面左侧是功能导航栏，包含工作台、抢单管理、订单管理、服务人员管理等选项，底部显示当前登录用户为Admin，其右侧设置有退出按钮，该按钮被红色箭头标注出，对应文档中提及的机构端页面的退出按钮内容。



### 3.2 **完成机构注册功能开发**

界面原型：

进入登录页面，点击“去注册”进入注册页面

![p6hqb7hgsozgmrx45anc08mlnaw](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/p6hqb7hgsozgmrx45anc08mlnaw.png)
> 图注：图片展示的是云岚到家一站式家政服务平台的登录页面。页面上方显示平台名称及“机构端”标识。下方有输入手机号和密码的区域，以及红色的“登录”按钮。页面右下角有“忘记密码”选项，左下角有“没有账号？去注册”链接，箭头指向该链接。页面底部提示仅用于IT培训教学使用，提醒勿向平台录入个人敏感信息。该图片对应文档中“进入登录页面，点击‘去注册’进入注册页面”的操作说明。

![hfo0bvqloofvfzx20n8cxzgtntc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/hfo0bvqloofvfzx20n8cxzgtntc.png)
> 图注：这是云岚到家一站式家政服务平台的机构端注册页面，页面顶部显示平台标识及“机构端”字样，包含手机号、验证码、密码三类必填输入项。手机号输入框提示“请输入您的手机号”；验证码输入框已被勾选高亮，旁设“获取验证码”按钮，下方标注“验证码必填”；密码输入框提示“请输入最少8位密码”，下方标注“密码必填”，还附带密码可见图标。页面底部设有红色的“注册”操作按钮，最下方有“已有账号？去登录”的跳转提示。该页面与文档中机构端注册的功能开发内容对应，是注册流程的入口页面。



接口定义如下：

接口地址：**POST**/customer/open/serve-provider/institution/register

![mxahbtdojopd8dx4shvczejpnfb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mxahbtdojopd8dx4shvczejpnfb.png)
> 图注：图片展示了机构注册接口的相关信息。接口地址为POST /customer/open/serve-provider/institution/register，请求数据类型为application/json，响应数据类型为“*”。请求示例中包含验证码、密码、手机号等字段。请求参数中，instituteRegisterReqDTO为必填的机构注册信息，verifyCode、password、phone均为必填的字符串类型字段。该图片与文档中机构端完成注册功能开发的接口定义部分对应，直观呈现了接口的请求方式、地址、数据类型及示例，以及请求参数的名称、说明、类型、是否必须等信息。

![x6babbsvholaw4xrl8ic6txinih](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/x6babbsvholaw4xrl8ic6txinih.png)
> 图注：图片展示了机构端注册接口的响应参数。包含参数名称、参数说明、类型和schema四列。参数名称有msg和code，msg为响应消息，类型为string；code为状态码，200表示成功，其它表示失败，类型为integer(int32)，schema也为integer(int32)。该图片与上文接口定义相关，是对接口响应参数的具体说明，帮助开发者了解接口返回数据的结构和含义。





#### 设计须知

参考服务端自动注册的代码实现。

![hzslb0znwougaxxsmtrcheegnoh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/hzslb0znwougaxxsmtrcheegnoh.png)
> 图注：这张图片展示的是LoginController.java代码文件的部分内容，属于机构端用户认证相关的代码实现。代码中定义了LoginController类，注入了ILoginService依赖，包含带有“/worker”路径的POST请求接口，用于服务人员或机构人员登录。在该接口的逻辑判断中，当用户类型为机构（INSTITUTION）时，执行loginService的loginForVerify方法，该方法的代码行被红色方框标注突出显示。该代码属于机构注册相关功能开发的参考内容，用于指导相关服务的代码实现，是机构注册流程中服务端接口代码的参考示例。



![qtfwbnn3aobky3xognncc3gfnib](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qtfwbnn3aobky3xognncc3gfnib.png)
> 图注：图片展示的是ServeProviderServiceImpl.java文件中的代码片段。该代码用于机构端注册功能开发，接口地址为POST /customer/open/serve-provider/institution/register。首先校验手机号是否存在，若存在且为服务人员注册，则抛出BadRequestException异常；若为机构注册，也抛出异常。接着新增服务人员/机构信息，设置手机号等属性。此代码与文档中机构端注册功能开发的接口定义及设计须知相关，是服务端自动注册代码实现的参考。

**注意：机构端注册和服务端注册完成要向serve_provider表写入数据，具体查阅上图的方法。**



密码加密方式：使用BCrypt方式，BCrypt是一种密码哈希函数，通常用于存储用户密码的安全性。它是基于 Blowfish 密码算法的一种单向哈希函数

![b3arb5wc8ourmsxavymcldjznie](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/b3arb5wc8ourmsxavymcldjznie.png)
> 图注：图片展示了项目结构和SecurityConfig.java代码。项目结构中，SecurityConfig位于com.jzo2o.customer.config包下。代码中，SecurityConfig类被@Configuration注解标注，用于对密码进行哈希和校验。关键部分是@Bean注解下的passwordEncoder方法，返回new BCryptPasswordEncoder()，用于密码编码。该图片与文档中完成机构注册功能开发部分内容相关，展示了密码哈希编码的实现代码。

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

![owwubfyhdo0hc0x3ek4cwecmnlb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/owwubfyhdo0hc0x3ek4cwecmnlb.png)
> 图注：这是云岚到家一站式家政服务平台机构端的登录界面，界面顶部显示平台名称与“机构端”的身份标识，界面设有手机号输入框与密码输入框，手机号输入框旁标注有“请输入正确格式的账号”提示，密码输入框旁标注有“密码必填”提示。界面底部有红色的“登录”按钮，还设有“没有账号？去注册”的入口，并有红色箭头指向“忘记密码”选项，该选项与文档中提及的“完成忘记密码功能开发”的相关设计对应，符合文档中关于机构端忘记密码功能的界面原型描述。

![t7m8bsawnondggx4yrpcyscfnhh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/t7m8bsawnondggx4yrpcyscfnhh.png)
> 图注：图片展示的是机构端忘记密码功能的找回密码界面。界面上方有“找回密码”标题，下方有三个必填项，分别是“手机号”输入框，提示“请输入您的手机号”；“验证码”输入框，提示“请输入验证码”，并有“验证码必填”说明；“新密码”输入框，提示“请输入最少8位密码”，并有“密码必填”说明。右侧有“获取验证码”按钮。底部有“取消”和“保存”按钮。该图片与文档中介绍的忘记密码功能开发相关，直观呈现了该功能的界面样式。

接口定义如下：

接口名称：机构登录密码重置接口

接口路径：**POST**/customer/agency/serve-provider/institution/resetPassword

![wqj4b4ygzosnmlxsho5cmsiynzd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wqj4b4ygzosnmlxsho5cmsiynzd.png)
> 图注：这是机构登录密码重置接口的相关定义界面，接口为POST请求类型，请求路径是/customer/agency/serve-provider/Institution/resetPassword。该接口的请求数据类型为application/json，响应数据类型无特定格式。请求示例中包含verifyCode、password、phone三个参数，对应请求参数区域的三个必填项：短信验证码（verifyCode）、新密码（password）、手机号（phone），所有三个参数均为必填的string类型，请求参数名称关联机构密码重置接口的机构密码重置请求对象。

![rqg3bn62jogjqux1naocui5inbb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/rqg3bn62jogjqux1naocui5inbb.png)
> 图注：该图片展示的是机构登录密码重置接口对应的响应参数信息，包含参数名称、参数说明、类型及schema这几类内容。其中参数“msg”为响应消息，类型是string；参数“code”为状态码，类型为integer(int32)，schema也对应integer(int32)，且参数说明明确“200-成功，其它-失败”。这些响应参数内容对应文档中机构登录密码重置接口的定义部分，是该接口返回结果的规范说明。

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

![v7nmbtg5gocgz8xb7pqc1yuqnke](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/v7nmbtg5gocgz8xb7pqc1yuqnke.png)
> 图注：该界面是完成接单前的设置引导页，顶部有卡通猴子形象，下方标注提示“亲，完成以下设置，方可接单哦~”。页面包含四项设置选项，分别为实名认证、服务技能设置、服务范围设置、接单设置，每项设置右侧配有对应操作按钮，其中前三项为红色“去认证”“去设置”按钮，接单设置为开关形式，底部设有粉色的“进入首页”按钮。该界面承接前文“使用新手机号登录后进入该界面”的描述，展示了接单前需完成的各类认证与设置入口。



![yvrdbfstqoqqwgx9tbvcwm5sn8c](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/yvrdbfstqoqqwgx9tbvcwm5sn8c.png)
> 图注：这是用户认证中个人实名认证的提交认证界面，界面带有必填项标识，需输入真实姓名与身份证号，还需上传身份证件正反面，明确说明该上传内容仅用于云岚到家系统认证。界面设置了人像面照片、国徽面照片的上传区域，每个上传区域旁均配有对应示例图，方便用户了解需上传的内容样式，下方标注的“+”号按钮可用于上传对应面的身份证件照片，整体为实名认证提交环节的信息填写与资料上传界面，符合用户提交认证时的操作需求。



![w0fab4ltkoraysx9z3fccpsvncd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/w0fab4ltkoraysx9z3fccpsvncd.png)
> 图注：这张图是用户进行实名认证时的提交认证界面，界面上方标注有“*上传证明资料”的提示，中间设有带加号的资料上传区域，下方配有“资料上传”的文字说明，最底部是醒目的红色“确认提交”按钮。该界面是用户用新手机号登录成功后进入的提交认证环节页面，用户在此处完成身份证件上传操作后，点击“确认提交”即可完成认证信息的提交。

上传身份证件，点击“确认提交”。

提交后等待运营人员审核，当审核没有通过可以查看失败原因：

![xrjvbjgjqotqovxnu2ec2dq5ng4](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xrjvbjgjqotqovxnu2ec2dq5ng4.png)
> 图注：该图片展示了用户端实名认证未通过的界面，顶部配有卡通猿猴形象，还有提示文字“亲，完成以下设置，方可接单哦~”。界面罗列四项设置项，其中“实名认证”项的右侧带有红色标注的“认证失败”按钮，并用红色箭头指向该按钮，对应上下文里查看认证失败原因的相关内容；其余三项分别是服务技能设置、服务范围设置、接单设置，服务技能与范围设置右侧配有“去设置”按钮，接单设置右侧为开关控件，界面底部还有粉色的“进入首页”按钮。

点击“认证失败”

![kwbgbv6q6oymnaxxkrxctz17n7b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/kwbgbv6q6oymnaxxkrxctz17n7b.png)
> 图注：图片展示的是实名认证未通过界面。上方显示“实名认证”，中间有一个红色的“未通过”印章图案，下方文字说明“提交的认证审核未通过，原因：未满足认证要求”，底部有一个红色的“重新认证”按钮。该图片对应文档中“4.1.1业务流程”部分，当用户提交实名认证后，若审核未通过，会显示此界面，用户可点击“重新认证”重新填写认证信息。

点击“重新认证”重新填写认证信息。

运营人员审核通过后显示“已完成”

![duplbhpiooh8pnxsrzic8dwgnzx](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/duplbhpiooh8pnxsrzic8dwgnzx.png)
> 图注：这张界面是用户实名认证相关的设置页，顶部显示提示文字“亲，完成以下设置，方可接单哦~”。界面里有四个设置选项，其中“实名认证”选项旁有“已完成”的标识，该选项被红色箭头重点指向，体现其完成状态；另外三个选项分别为需点击“去设置”的服务技能设置、服务范围设置，还有处于未开启状态的接单设置，页面底部还有“进入首页”的按钮，整体是用于提示用户完成各项接单相关设置的操作界面，和文档中实名认证、机构端认证的相关内容相呼应。





#### 4.1.3 机构端提交认证

首先登录点击认证进入上传资质界面

![saqlbiymhoeyvsxx1hlcmm8gnog](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/saqlbiymhoeyvsxx1hlcmm8gnog.png)

点击“去认证”

![i2lvbrbhcoe8opxq2y4cfyvvngc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/i2lvbrbhcoe8opxq2y4cfyvvngc.png)
> 图注：这张图片展示的是机构端提交认证时的资质认证页面，属于实名认证流程里机构端提交认证的相关界面。页面内有多项需填写的信息项，包含带红色星号标注的企业名称、统一社会信用代码、法人代表姓名、身份证号，以及带红色星号的上传营业资质要求，用户需按提示完成对应内容填写，填写完成后可点击页面右下角的“取消”或红色“提交”按钮来操作。页面还标注了上传营业执照的格式与大小要求，即需上传1张JPG或PNG格式文件，大小在5MB以内。

填写信息点击“提交”，运营人员进行审核 。



可以查看认证结果

![tqyhbz6c0ox746xxoxuclrsvnkg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/tqyhbz6c0ox746xxoxuclrsvnkg.png)
> 图注：图片展示的是用户认证中机构端提交认证后的认证结果页面。页面上方有卡通猴子形象，下方文字提示“亲，完成以下设置，方可接单哦~”。页面中有四个设置选项，分别是资质认证、服务技能设置、服务范围设置和接单设置，其中资质认证下方有“认证失败”红色提示框，有红色箭头指向。该图片与文档中“4.1.3机构端提交认证”部分内容对应，直观呈现了认证失败时的页面情况。

点击“认证失败”

![afp9bpm8forevcxptbgcalb5nmh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/afp9bpm8forevcxptbgcalb5nmh.png)
> 图注：图片展示的是机构端提交认证时的资质认证失败界面。界面上方显示“资质认证”，右上角有关闭按钮。中间部分有一个红色的“未通过”印章图案，旁边文字说明“提交的认证审核未通过”，并指出原因“公司信息不完整或不准确”。下方有一个红色的“重新认证”按钮。该图片与文档中“4.1.3机构端提交认证”部分内容对应，用于说明在提交认证信息后，若审核未通过，可点击“重新认证”再次上传资质信息。

点击“重新认证”再次上传资质信息。

当认证通过会显示“已完成”。

完成其它项目的设置即可进入首页

![b5evbbykmo7adsx1vozcmdzjnhh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/b5evbbykmo7adsx1vozcmdzjnhh.png)
> 图注：图片展示的是用户认证完成后的提示界面。上方有卡通猴子形象，下方文字提示“亲，完成以下设置，方可接单哦~”。界面分为四个板块，分别是资质认证、服务技能设置、服务范围设置、接单设置，每个板块右侧均有“已完成”标识，其中服务技能设置和接单设置右侧还有红色箭头指向。最下方有一个红色的“进入首页”按钮。该图片与文档中“完成其它项目的设置即可进入首页”的内容相呼应，直观呈现了认证完成后的状态。





#### 4.1.4 运营端审核认证

##### 4.1.4.1 审核服务人员

审核服务人员认证列表：

![mi0dbcxmqomdhexbblgcwnysnvh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mi0dbcxmqomdhexbblgcwnysnvh.png)
> 图注：图片展示的是云岚到家平台服务人员管理下的服务人员认证管理界面。界面上方有“服务人员管理”“服务人员认证管理”等导航栏。中间区域有真实姓名、身份证号等输入框，认证状态有全部、认证通过、认证失败选项，审核状态有全部、已审核、未审核选项，还有搜索按钮。下方列表显示服务人员信息，如真实姓名、身份证号、认证状态等，其中“小明”认证状态为“待审核”，审核状态为“待审核”，操作栏有“通过”“驳回”按钮。该图与上下文介绍的运营端审核认证功能相关，直观呈现了审核服务人员认证列表界面。

数据说明：

审核状态，0：未审核，1：已审核

待审核:机构或服务人员提交了申请，后台人员没有提交审核结果（同意、驳回）

已审核:机构或服务人员提交了申请，后台人员已选择同意/驳回

认证状态：认证状态，1：认证中，2：认证成功，3: 认证失败

认证中:提交认证后的认证状态

认证通过:显示最新的认证申请，后台人员选择通过，显示认证通过

认证失败:显示最新的认证申请，后台人员选择驳回，显示认证失败



驳回：

![k5jwbbrwxo5zjixctoqcnxt1n7j](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/k5jwbbrwxo5zjixctoqcnxt1n7j.png)
> 图注：图片展示的是运营端审核认证中驳回服务人员认证申请时的界面。界面上方有“驳回原因”标题，下方有“请选择”提示框，框内有三个选项：公司信息不完整或不准确、未满足认证要求、风险审核未通过。该图片与上下文紧密相关，是对上下文中“驳回”操作时弹出的驳回原因选择框的直观呈现，帮助运营人员在驳回服务人员认证申请时明确选择原因。

驳回后审核状态为“已审核”，认证状态为“认证失败”

![nh29b05zrotreyxz567cxp6snog](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/nh29b05zrotreyxz567cxp6snog.png)
> 图注：图片展示的是云岚到家运营端审核服务人员认证管理界面。界面上方有“服务人员管理 - 服务人员认证管理”标题。下方有“真实姓名”“身份证号”输入框及“搜索”按钮。列表中显示了身份证号、身份证人像照片、身份证国徽照片、证明资料、审核状态、认证状态、驳回原因等信息，其中部分身份证号被红框突出显示，审核状态为“已审核”，认证状态有“认证失败”和“认证成功”，驳回原因显示“未满足认证要求”。该图与上下文介绍的运营端审核服务人员认证相关，直观呈现了认证管理界面及部分认证状态信息。

驳回后服务人员可以再次提交认证申请，运营人员在查询列表可以查到多条该用户的申请记录。

![qjfvbnczyokhsqxur13clm6mnxn](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qjfvbnczyokhsqxur13clm6mnxn.png)
> 图注：这张图是运营端服务人员认证管理的页面，顶部设有真实姓名、身份证号的输入框，还有审核状态（全部/已审核/未审核）、认证状态（全部/认证通过/认证失败）的筛选选项，支持重置与搜索操作。列表包含真实姓名、身份证号、身份证相关照片、证明资料、审核状态、认证状态等信息项，其中第一行记录的服务人员为小明，身份证号10113232323233，审核状态为待审核，认证状态为待认证，对应右侧的操作栏有“通过”和“驳回”按钮，与文档中“点击通过，再点击确定认证成功”的审核操作内容相呼应。

点击“通过”

![olg5bdf2nowpthxsyv8cnraxnxf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/olg5bdf2nowpthxsyv8cnraxnxf.png)
> 图注：图片展示的是运营端审核服务人员实名认证时的确认通过弹窗。弹窗标题为“确认通过”，内容说明此操作将同意该服务人员入驻平台，认证通过后该服务人员可开启接单业务，询问是否确认通过实名认证申请。弹窗右上角有“×”关闭按钮，下方有“取消”和“确定”两个按钮，其中“确定”按钮为红色。该弹窗与上文提到的点击“通过”后弹出的确认通过弹窗相呼应，是审核服务人员认证流程中的关键确认环节。

点击“确定”认证成功

![fokybuwudocrh6xoebdce1wfnjc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/fokybuwudocrh6xoebdce1wfnjc.png)
> 图注：这是云岚到家平台服务人员认证管理的运营端界面，界面显示了服务人员的认证列表内容。列表中清晰展示了真实姓名、身份证号等基础信息，还呈现了审核状态与认证状态两项核心内容，如序号1的服务人员小明，审核状态为已审核，认证状态为认证成功，其身份证人像与国徽照片均为头像图标显示。审核状态区分未审核、已审核，认证状态区分认证中、认证成功、认证失败，符合上下文对认证相关数据的说明，列表中的服务人员信息均对应相关认证状态，该界面用于运营人员对服务人员认证申请进行审核操作。



##### 4.1.4.2 审核机构



运营人员审核机构：

![tke9bspkkochtcxhej2cweudnbi](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/tke9bspkkochtcxhej2cweudnbi.png)
> 图注：图片展示的是云岚到家运营端审核认证中机构认证的列表页面。页面上方有企业名称、法定代表人姓名、审核状态等输入框及搜索按钮。下方列表显示了机构名称、统一社会信用代码、法定代表人姓名、身份证号、营业执照、审核状态、认证状态及操作等信息。其中，机构02的审核状态为“待审核”，认证状态为“认证失败”，操作栏有“通过”和“驳回”按钮，有红色箭头指向“通过”按钮。该图与上下文介绍的运营人员审核机构认证流程相关，直观呈现了审核状态及操作选项。

驳回：

![kllob3guwokt7ax280xccn3vnec](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/kllob3guwokt7ax280xccn3vnec.png)
> 图注：图片展示的是运营端审核机构时的“驳回”操作界面。界面上方显示“驳回原因”，下方有一个下拉框，提示“* 驳回原因：公司信息不完整或不准确”。下方有“取消”和“确定”两个按钮。该图片与文档中“运营端审核机构”部分内容相关，用于说明驳回操作时的界面情况，以及驳回后审核状态和认证状态的变化。

审核完成再查询审核记录，审核状态为已审核，状态为认证失败。

![ueuzbicjdorxgsxkhruc8zrrndh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ueuzbicjdorxgsxkhruc8zrrndh.png)
> 图注：图片展示的是云岚到家运营端审核认证中机构认证信息列表页面。页面上方有“企业认证管理”标题，下方有企业名称、法定代表人姓名、审核状态等搜索栏。列表中显示了机构名称、身份证号、营业执照、审核状态、认证状态、驳回原因及申请时间等信息，其中一条认证信息被红色框突出显示，其审核状态为“已审核”，认证状态为“认证失败”，驳回原因为“公司信息不完整或不准确”，申请时间为2023年10月23日12:41。该图片与上下文介绍的运营端审核机构认证流程相关，直观呈现了审核后的认证信息状态。

当机构端再次提交审核后再次查询企业认证信息，新增了一条认证信息。

![nlrtbg96ao56xqx5i9yciaebnmh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/nlrtbg96ao56xqx5i9yciaebnmh.png)
> 图注：图片展示的是企业认证管理页面。页面左侧有导航栏，选中“企业认证管理”。右侧显示企业认证信息列表，包含企业名称、法定代表人姓名、身份证号、营业执照、审核状态、认证状态、驳回原因等字段。列表中有待审核、已审核、认证失败、认证成功等状态标识。图片中用红色框突出显示了一条待审核记录，其操作栏有“通过”和“驳回”按钮。该图片与上下文介绍的审核机构相关，直观呈现了审核记录的展示情况。

审核通过：

![ydtmbeudyo45nmxuflqc5vfin6f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ydtmbeudyo45nmxuflqc5vfin6f.png)
> 图注：图片展示的是企业认证审核通过后的确认弹窗。弹窗标题为“确认通过”，内容说明此操作将同意该企业入驻平台，认证通过后企业可开启接单业务，询问是否确认通过实名认证申请。下方有“取消”和“确定”两个按钮，其中“确定”按钮为红色。该图片与文档中“4.1.4.2审核机构”部分内容相关，对应审核通过后审核状态为已审核，认证状态为认证成功这一场景。

审核通过后审核状态为已审核，认证状态为认证成功。

![plwkbciqlo905bxiydwcmhsinmg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/plwkbciqlo905bxiydwcmhsinmg.png)
> 图注：这是云凤到家管理端的企业认证管理页面，当前显示的是企业认证审核相关内容，页面上方设有企业名称、法定代表人姓名、审核状态等搜索筛选栏，还有重置、搜索按钮。页面列表展示了已完成审核的企业认证记录，其中一条核心突出的记录被红色框标注，对应的法定代表人为黑马程序员，身份证号为10122314345322，营业显示为隐藏状态，审核状态为已审核，认证状态为认证成功。该页面是企业实名认证业务流程中审核环节的相关操作界面，呈现了运营人员审核认证后的状态记录。



### 4.2 接口设计

#### 4.2.1 服务端提交认证接口设计

##### 4.2.1.1 服务端提交认证申请

接口路径：**POST**/customer/worker/worker-certification-audit

**请求数据类型:** application/json

请求参数：

![jruhbh5lsozlhjxacagcbyb3nm5](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/jruhbh5lsozlhjxacagcbyb3nm5.png)
> 图注：这张图展示的是服务端提交认证申请接口（路径为POST /customer/worker/worker-certification-audit）的请求参数表格，该接口是用户实名认证实战中接口设计的一部分，对应文档中“服务端提交认证申请”的请求参数说明。表格包含参数名称、参数说明、请求类型、是否必须、数据类型、schema共六列，其中核心参数workerCertificationAuditAddReqDTO的请求类型为body且为必填项，其余必填参数包括证明资料、身份证号、身份证反面、身份证正面、姓名，数据类型多为string；服务人员id为非必填项，数据类型为integer(int64)，schema标注为服务人员认证申请请求体。

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

![leo9bvxvmotk81xyfwbcrewlnpg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/leo9bvxvmotk81xyfwbcrewlnpg.png)
> 图注：图片展示了服务端提交认证申请接口的响应参数。包含两个参数：msg，类型为string，参数说明为响应消息；code，类型为integer(int32)，参数说明为状态码，200表示成功，其它表示失败。该图片与文档中“4.2.1.1服务端提交认证申请”接口设计内容相关，是对该接口响应参数的说明，帮助开发者了解接口返回数据的结构。

##### 4.2.1.2 查询最新的驳回原因

接口路径：**GET**/customer/worker/worker-certification-audit/rejectReason

**请求数据类型：** application/x-www-form-urlencoded

请求参数：

无

响应参数：

![kuszb1blmoe8t6xqqjwcadovn3e](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/kuszb1blmoe8t6xqqjwcadovn3e.png)
> 图注：这是服务端查询最新驳回原因接口的响应参数表格，该接口的接口路径为GET /customer/worker/worker-certification-audit/rejectReason。表格包含参数名称、参数说明、类型、schema四列，明确列出三个响应参数：msg为string类型的响应消息，code为integer(int32)类型的状态码（200代表成功，其他代表失败），data为对应驳回原因响应数据的集合，其中子参数rejectReason是string类型的具体驳回原因，用于返回实名认证申请被驳回的具体理由。

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

![xkzjbu4tcocqrpxmnrochpeinsc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xkzjbu4tcocqrpxmnrochpeinsc.png)
> 图注：图片展示了机构提交认证申请的请求参数。包含agencyCertificationAuditAddReqDTO、businessLicense、legalPersonIdCardNo、idNumber、legalPersonName、name等参数，其中agencyCertificationAuditAddReqDTO为机构认证申请，businessLicense、legalPersonIdCardNo、idNumber、legalPersonName、name为必填项，数据类型均为string，且有红色标识。此图与上文机构提交认证申请接口设计中请求参数部分对应，直观呈现了参数信息。

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

![rgnhbwo3ao4bchxuwcqcation5g](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/rgnhbwo3ao4bchxuwcqcation5g.png)
> 图注：图片展示了服务端提交认证申请接口的响应参数。包含两个参数：msg，类型为string，参数说明为响应消息；code，类型为integer(int32)，参数说明为状态码，200表示成功，其它表示失败。该图片与上文介绍的服务端提交认证申请接口响应参数内容对应，直观呈现了响应参数的具体信息。

##### 4.2.2.2 查询最新的驳回原因

接口路径：**GET**/customer/agency/agency-certification-audit/rejectReason

**请求数据类型：** application/x-www-form-urlencoded

请求参数：

无

响应参数：

![vxvfbn12ho8eboxrs54c176mneh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vxvfbn12ho8eboxrs54c176mneh.png)
> 图注：这张图片是机构端查询最新驳回原因接口的响应参数示例表，用于展示该接口的响应数据结构。表格中包含参数名称、参数说明、类型及schema四列，核心参数包括：msg为类型为string的响应消息，code是类型为integer(int32)的状态码，200代表成功，其他值代表失败，data是层级为驳回原因响应数据的响应体，rejectReason为类型string的驳回原因。该表格用于明确机构端查询最新驳回原因接口的返回参数规范。

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

![mxl3b3ugtoldyjxxcvlcpc8dnch](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mxl3b3ugtoldyjxxcvlcpc8dnch.png)
> 图注：图片展示了运营端审核服务人员认证分页查询接口的请求参数。参数包括审核状态、认证状态、身份证号、排序字段1是否升序、排序字段2是否升序、姓名、排序字段1、排序字段2、页码数、每页条数等，均为query类型且非必填。其中，auditStatus和certificationStatus为整数类型，idCardNo为字符串类型，其余为布尔或字符串类型。该图片与文档中4.2.3.1审核服务人员认证分页查询内容对应，直观呈现了接口请求参数信息。

![q2k2brvw8ora3lx9n7zcvektnog](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/q2k2brvw8ora3lx9n7zcvektnog.png)
> 图注：这是服务端提交认证申请接口对应的响应参数示例表，包含响应消息、状态码、数据列表三类核心参数，分别对应不同的参数说明与数据类型。参数内的细分项涵盖了证明资料、身份证号、服务人员id等认证相关信息，还有认证状态、身份证反面、审核人id、驳回原因等关键内容，其中认证状态的不同数值对应不同的认证阶段，该表格清晰展示了接口响应返回的具体数据项及对应要求，为该接口的对接提供了明确的参数参考。

![qbibb3gqgobgeex2m6ycc8ghnqs](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qbibb3gqgobgeex2m6ycc8ghnqs.png)
> 图注：图片展示了服务端提交认证申请的响应参数。包含auditTime（审核时间）、createTime（创建时间）、frontImg（身份证正面）、name（姓名）、auditStatus（审核状态，0为未审核，1为已审核）、id（主键）、auditorName（审核人姓名）、total（总条数）、pages（总页数）等字段，其中auditStatus、id、total、pages为integer类型，其余字段为string类型。该图片与上文介绍的服务端提交认证申请接口响应参数内容对应，直观呈现了响应参数的具体信息。

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

![mknobthg4o39hox3hj0cfeq9nqg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mknobthg4o39hox3hj0cfeq9nqg.png)
> 图注：图片展示了查询最新的驳回原因接口的请求参数信息。请求数据类型为application/x-www-form-urlencoded，响应数据类型为“/”。参数包括：certificationStatus，认证状态，2为认证成功，3为认证失败，类型为integer(int32)，是否必须为true；id，认证申请id，类型为integer(int64)，是否必须为true；rejectReason，驳回原因，类型为string，是否必须为false。该图片与文档中“4.2.1.2查询最新的驳回原因”接口设计内容相关，是对该接口请求参数的具体说明。

![g8ppbom2zomovvxjvxkcqkxrnsd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/g8ppbom2zomovvxjvxkcqkxrnsd.png)
> 图注：这是一张接口设计相关的响应参数说明表，展示的是接口返回的响应参数信息。表格分为参数名称、参数说明、类型、schema四列，其中包含两个响应参数：名称为msg的参数，参数说明为响应消息，类型为string；名称为code的参数，参数说明为状态码，200代表成功，其他值代表失败，类型为integer(int32)，schema也对应为integer(int32)。该表对应文档中接口设计内容里的响应参数部分，用于明确接口返回的参数规范。

##### 4.2.3.3 审核机构认证分页查询

接口路径：**GET**/customer/operation/agency-certification-audit/page

![j6j8b5qykobptwx0nkjcnrymnhy](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/j6j8b5qykobptwx0nkjcnrymnhy.png)
> 图注：图片展示了运营端审核认证接口中审核机构认证分页查询的请求参数。参数包括auditStatus、certificationStatus、isAsc1、isAsc2、legalPersonName、name、orderBy1、orderBy2、pageNo、pageSize等，均为query类型且false，数据类型多为string或integer。其中auditStatus为审核状态，certificationStatus为认证状态，isAsc1和isAsc2为排序字段1和2是否升序，legalPersonName为法人姓名，name为企业名称，orderBy1和orderBy2为排序字段，pageNo为页码数，pageSize为每页条数。该图片与文档中介绍的运营端审核认证接口设计内容相关，是对其中审核机构认证分页查询接口请求参数的说明。

![jwntbxbizotllzx1tnxceblonsg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/jwntbxbizotllzx1tnxceblonsg.png)
> 图注：图片展示了服务端提交认证申请接口的响应参数。包含msg、code、data等字段，其中data下有list，包含businessLicense、legalPersonIdCardNo、serveProviderId、updateTime、certificationStatus、idNumber、legalPersonName、auditorId等参数，如businessLicense为营业执照，legalPersonIdCardNo为法人身份证号，serveProviderId为服务人员id等。该图片与上文服务端提交认证申请接口设计内容相关，是对响应参数的详细说明。

![ygewbwcncouhpixqm9kccdxbnlc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ygewbwcncouhpixqm9kccdxbnlc.png)
> 图注：图片展示了服务端提交认证申请接口的响应参数。包含驳回原因、审核时间、创建时间、企业名称、审核状态、主键、审核人姓名、总条数、总页数等字段，其中审核状态以0、1标识，总条数和总页数为int64类型。该图片与上文介绍的服务端提交认证申请接口响应参数内容对应，直观呈现了响应参数的具体信息。

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



![dmb2bmo4borkrgxxuu2cev3jncc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dmb2bmo4borkrgxxuu2cev3jncc.png)
> 图注：这张图片展示的是实名认证相关接口的响应参数说明表格，表格内包含参数名称、参数说明、类型、schema四列内容。其中参数名称为msg和code，msg的参数说明是响应消息，类型为string；code的参数说明是状态码，200表示成功，其余值表示失败，类型为integer(int32)，schema也对应为integer(int32)，该表格用于明确对应接口返回的参数信息，是实名认证接口设计文档中响应参数部分的具体内容。



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

![dnxqbm2oeobv2wxjtoqckt3bn1g](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/dnxqbm2oeobv2wxjtoqckt3bn1g.png)
> 图注：图片展示的是“我的”界面，位于文档中“服务端设置银行账户”操作步骤的上下文位置。界面上方有“服务技能”“接单设置”“服务范围”等选项卡。下方有“更好的家政 上门服务体验”及“一个靠谱的家庭服务平台”宣传语。中部有“抢单大厅”“距离筛选”“服务类型”等标签。底部导航栏有“首页”“订单”“消息”“我的”选项，其中“我的”选项被红色箭头指向。该图直观呈现了进入“我的”界面的操作结果。

进入“我的”界面，点击“账户设置”

![iycsbn2uwonyt7xwcitcuv1ante](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/iycsbn2uwonyt7xwcitcuv1ante.png)
> 图注：这张图片展示了服务人员端应用的“我的”页面，页面顶部显示服务人员“服务人员02”的信息，标注为“已认证”，并显示联系电话15066699132，还设有综合评分和好评率模块。页面的功能列表里，有“我的评价”和带红色箭头指向的“账户设置”选项，页面底部是“退出登录”按钮，最下方是“首页”“订单”“消息”“我的”的底部导航栏。该页面的“账户设置”选项，是服务端设置银行卡账户流程中，进入账户设置步骤的操作入口，对应文档里进入“我的”界面后点击“账户设置”的操作环节。

进入账户设置，如果已经设置了账户信息则直接显示在界面中。

![ygi8b1b05oqyivxoczxc5u0bn8g](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ygi8b1b05oqyivxoczxc5u0bn8g.png)
> 图注：图片展示的是“账户设置”界面，包含“户名”“银行”“开户行”“网点”“银行账号”“开户证明”等输入项，各输入项前有红色星号标识必填。界面底部有一个红色“确认提交”按钮。该图片与上文“服务端设置银行卡账户”内容相关，是对进入“账户设置”界面后显示的账户信息录入界面的呈现，为后续录入账户信息操作提供直观展示。

 录入账户信息：

![rdlobtgzeoq1qoxw0fxcqd6jneg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/rdlobtgzeoq1qoxw0fxcqd6jneg.png)
> 图注：图片展示的是“账户设置”界面，显示已录入的账户信息，包括户名“王小明”、银行“中国工商银行”、开户行“北京 - 北京市 - 东城区”、网点“工商银行东城区xx支行”、银行账号“10232411112222”等。界面底部有“确认提交”按钮，右上角有“资料上传”区域，可上传开户证明。该图片与上文“服务端设置银行卡账户，每个服务人员设置一个银行卡账户”及录入账户信息的操作步骤相关，是设置银行卡账户信息后的展示界面。

点击“确认提交”。

再次进入“账户设置”显示设置的账户信息。



#### 5.2.2 机构端设置银行账户

机构端登录进入首页，点击“账户设置”

![s6brb3n1jom890xh334c3ntfn0g](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/s6brb3n1jom890xh334c3ntfn0g.png)
> 图注：这是云岚到家一站式家政服务平台的机构端工作台界面，左侧是包含工作台、抢单管理、订单管理等功能的导航栏，界面上方展示了总收入、总退款、订单总数的累计订单金额数值，下方设有基础设置相关选项。其中箭头指向的“账户设置”选项，是文档中提及的机构端登录进入首页后需点击的操作入口，点击该入口即可打开账户设置界面，开展机构端银行账户的后续设置操作。

打开账户设置界面 

![ncebbojplobfnsxpsxocrozfnbf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ncebbojplobfnsxpsxocrozfnbf.png)
> 图注：图片展示的是机构端登录进入首页后，点击“账户设置”后打开的账户设置界面。界面上有多个必填项，包括户名、开户行（有省份、市、区选择框)、银行、网点、银行账号、开户证明(需上传1张jpg/png文件，5MB以内)，每个必填项旁都有红色星号标识。界面右上角有“取消”和“提交”按钮。该图片与上文“机构端登录进入首页，点击‘账户设置’，打开账户设置界面”的内容对应，直观呈现了账户设置界面的样式及必填项信息。

填写要求：

![klprbjlm9oqaahxcsmhcgbzbnnc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/klprbjlm9oqaahxcsmhcgbzbnnc.png)
> 图注：图片为机构端设置银行账户时填写信息的表格。表格包含信息字段、必填/选填、限制要求三列。户名必填，限2 - 10个汉字；银行必填，平台仅支持7个银行；开户行必填，需选择省市区，直辖市显示如北京北京市昌平区；网点必填，限4 - 50位汉字；银行账号必填，储蓄卡位数限制（16位、17位、19位）；开户证明必填，需上传1张jpg/png图片，大小在55MBb以内。该表格与文档中机构端设置银行账户的填写要求部分对应，明确了填写信息的具体要求。

填写信息：

![k33lbui2loyj42xj3idcy9dknff](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/k33lbui2loyj42xj3idcy9dknff.png)
> 图注：这张图片是机构端设置银行账户时的账户设置填写界面，界面内的必填项已预先填充信息，具体包含户名为郑州中原区xx家政服务公司，开户行为河南省郑州市中原区，银行为中国工商银行，网点为工商银行xx支行，银行账号为102939398382；还有“开户证明”的上传区域，要求上传1张5MB以内的jpg或png格式文件，界面底部设有“取消”和“提交”两个操作按钮，对应文档中机构端完成账户信息填写后提交操作的步骤内容。

点击“提交”后再次进入“账户设置”显示机构的账户信息。



### 5.3 服务端设置银行账户接口设计

#### 5.3.1 新增或更新银行账号信息接口

接口名称: 新增或更新银行账号信息

接口路径：**POST**/customer/worker/bank-account

请求类型：application/json

![wnzobftpjod3rvxysiwc4ep7nwg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wnzobftpjod3rvxysiwc4ep7nwg.png)
> 图注：这张图展示了“新增或更新银行账号信息”接口的请求参数信息表格。表格内明确列出了各请求参数的名称、说明、请求类型、是否必填、数据类型以及schema信息，其中参数bankAccountUpsertReqDTO是银行账户新增或更新请求体，作为必填的body参数传入。其余参数如开户证明、市、银行名称等多为必填的string类型，仅类型、服务人员/机构id为非必填的整数类型，该表格内容对应文档中服务端设置银行账户接口设计的“新增或更新银行账号信息接口”的请求参数要求。



![isebbxxilos7z6xetztcerlonjh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/isebbxxilos7z6xetztcerlonjh.png)
> 图注：图片展示了新增或更新银行账号信息接口的响应参数。包含参数名称、参数说明、类型和schema四列。参数名称有msg和code，msg为响应消息，类型为string；code为状态码，200表示成功，其它表示失败，类型为integer(int32)。该图片与上文介绍的新增或更新银行账号信息接口相关，是对接口响应参数的说明，帮助理解接口返回数据的结构。

#### 5.3.2 获取当前用户银行账号接口

接口名称：获取当前用户银行账号

接口功能：进入账户设置界面会先查询当前用户的账户信息，如果已经设置则显示在页面中。

接口路径：**GET**/customer/worker/bank-account/currentUserBankAccount

![zgylbvz3ao4razxgdejc8xinnhg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/zgylbvz3ao4razxgdejc8xinnhg.png)
> 图注：图片展示了新增或更新银行账号信息接口的请求参数部分。上方显示请求数据类型为application/x - www - form - urlencoded，响应数据类型为“* / *”。下方表格中，有参数名称、参数说明、请求类型、是否必须、数据类型、schema等列，但表格内容为空。该图片与文档中“新增或更新银行账号信息接口”部分对应，直观呈现了接口请求参数的相关信息。

![wzhxbb6jhoct9axk2f6ct8man6b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wzhxbb6jhoct9axk2f6ct8man6b.png)
> 图注：图片展示了新增或更新银行账号信息接口的响应参数。参数包括msg（响应消息）、code（状态码）、data（银行账户响应体）等。其中data下有accountCertification（开户证明）、city（市）、bankName（银行名称）、updateTime（更新时间）、type（类型）、branch（网点）、province（省）、createTime（创建时间）等具体参数，类型均为string，schema均为银行账户响应体。该图片与文档中新增或更新银行账号信息接口的内容相关，是对接口响应参数的详细说明。

![q7qjbarjpo6eanxyzysc1qrsnjg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/q7qjbarjpo6eanxyzysc1qrsnjg.png)
> 图注：图片展示了新增或更新银行账号信息接口的参数信息。包含四个参数：district（区，string类型）、name（名称，string类型）、id（服务人员/机构id，integer(int64)类型）、account（银行账号，string类型）。该图片与文档中“新增或更新银行账号信息接口”部分内容对应，是对接口参数的具体说明，帮助开发者了解接口所需传递的参数类型及名称。



### 5.4 机构端设置银行账户接口设计

#### 5.4.1 新增或更新银行账号信息接口

接口名称: 新增或更新银行账号信息

接口路径：**POST**/customer/agency/bank-account

请求类型：application/json

![goaqbicfvojn6gxw2ijclejfnod](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/goaqbicfvojn6gxw2ijclejfnod.png)
> 图注：这张图是新增或更新银行账号信息接口的请求参数详情，对应文档中5.4.1节的接口设计内容。表格展示了该接口的所有请求参数，包含参数名称、参数说明、请求类型、是否必须、数据类型及schema信息，其中bankAccountUpsertReqDTO为该接口的核心参数，属于银行账户新增或更新请求体，其余参数如开户证明、市、银行名称、网点等为接口所需的具体业务参数。

![ahfsbah11okcqjx8rdyctutvncf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ahfsbah11okcqjx8rdyctutvncf.png)
> 图注：图片展示了新增或更新银行账号信息接口的响应参数。参数包括msg（响应消息，类型为string）、code（状态码，200为成功，其他为失败，类型为integer(int32)）、data（银行账户响应体，包含accountCertification、city、bankName、updateTime、type、branch、province、createTime等字段，类型为string或integer(int32)）。该图片与上文介绍的接口响应参数内容对应，直观呈现了响应参数的结构及各参数的类型和说明。

![yznxbndaioxwguxtj96cln1kncg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/yznxbndaioxwguxtj96cln1kncg.png)
> 图注：图片展示了新增或更新银行账号信息接口的参数信息。包含四个参数：district（区，string类型）、name（名称，string类型）、id（服务人员/机构id，integer(int64)类型）、account（银行账号，string类型）。该图片与上文介绍的新增或更新银行账号信息接口相关，是对接口参数的具体说明，帮助开发者了解接口所需传递的参数类型及内容。

#### 5.4.2 获取当前用户银行账号接口

接口名称：获取当前用户银行账号

接口功能：进入账户设置界面会先查询当前用户的账户信息，如果已经设置则显示在页面中。

接口路径：**GET**/customer/agency/bank-account/currentUserBankAccount

![rxb9bbzjnogbyvxhqwvcxp75nee](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/rxb9bbzjnogbyvxhqwvcxp75nee.png)
> 图注：图片展示了新增或更新银行账号信息接口的请求参数部分。上方显示请求数据类型为application/x - www - form - urlencoded，响应数据类型为“* / *”。下方表格中，有参数名称、参数说明、请求类型、是否必须、数据类型、schema等列，但表格内容为空。该图片与文档中“新增或更新银行账号信息接口”部分对应，直观呈现了接口请求参数的相关信息。

![j0rkbrcaaogoklxvikbcrmivnme](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/j0rkbrcaaogoklxvikbcrmivnme.png)
> 图注：图片展示了新增或更新银行账号信息接口的响应参数。包含msg、code、data等字段，其中data下有accountCertification、city、bankName等具体参数，如accountCertification为开户证明，city为市，bankName为银行名称等。该图片与上文介绍的新增或更新银行账号信息接口相关，是对接口响应参数的详细说明，帮助理解接口返回数据结构。

![gxocbvxhdoc4whx0pupcdegmnpb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/gxocbvxhdoc4whx0pupcdegmnpb.png)
> 图注：这张图片是一份字段说明表格，对应机构端银行账户相关接口的入参或出参字段定义。表格分为三列，第一列是字段英文标识，分别为district、name、id、account；第二列是字段的中文含义，依次为区、名称、服务人员/机构id、银行账号；第三列是字段的数据类型，其中district和name、account为string类型，id为integer(int64)类型，这些字段用于机构端设置银行账号相关的接口请求或返回中，明确了该场景下银行账号相关的参数信息。





## 作业

实战完成要能针对以下问题进行流畅口述：

1、你负责模块的怎么设计的

介绍业务流程、表设计、接口设计

2、在开发中你遇到了什么问题

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 云岚到家项目 / 第三章 用户认证 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/PyPuw9ezkiAX9MkfSEkcUTfOnPh)
