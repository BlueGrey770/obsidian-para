---
type: memo
date: 2026-11-06
aliases:
  - "定制化UI界面"
---

# 定制化UI界面

## logo、背景图更换

如果使用若依框架项目做为脚手架，那么我们肯定需要在页面显示中，使用符合自己公司或者项目的标识才行，需要更换的地方很多，我们依次来解决它

- 浏览器标签页logo标识、标题
- 系统页面中的logo标识、标题
- 登录名称及背景图
- 去除源码地址 & 文档地址 &若依官网标识

![这张图展示了使用若依框架定制UI时，修改开发环境配置的相关界面。图中左侧导航栏标注了“若依官网”菜单选项，中间区域显示了若依后台管理框架的说明文字以及当前版本号，界面的右上角有多个功能图标。结合上下文可知，此处是要修改开发环境的VITE_APP_TITLE属性，以此完成系统页面标题的替换，是logo与背景图更换流程中的关键操作环节，用于更改项目默认的系统页面标题标识。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/VjLjbtWB2o1a35xBmKfcP9Nqnec.png)

![图片展示了若依框架项目登录页面的最终效果。页面背景为室内场景，有窗户、椅子等元素。登录区域上方有红色边框的“若依后台管理系统”文字，下方有用户名、密码、验证码输入框及登录按钮。该图对应文档中“登录名称及背景图”部分，用于直观呈现登录页面在更换登录名称和背景图后的最终展示效果。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/JgJybWc8XoKccXxSwQcceE20nHf.png)

### 浏览器标签页logo标识、标题

- logo替换

找到资料中UI资料目录下的logo图标，替换前端项目中的public文件夹，删除原有的favicon.ico，把新拷贝过来的logo更名为favicon.ico即可

![图片展示了若依框架项目文件结构及favicon.ico文件内容。左侧为项目文件目录，显示了public文件夹下的favicon.ico等文件。右侧是favicon.ico文件内容，呈现为一个绿色房屋形状的图标，房屋内有两根竖直线条。该图片与文档中“浏览器标签页logo标识、标题”部分相关，用于说明logo替换操作，即找到资料中UI资料目录下的logo图标，替换前端项目中的public文件夹，删除原有favicon.ico，将新logo更名为favicon.ico即可。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/AJH2bhmC3oK7FRxRA8nc49innxb.png)

- 标题更换

找到根目录下的index.html文件，把title更换为自己想要的内容即可

![图片展示了若依框架项目中index.html文件的代码界面。左侧是项目文件目录，其中public文件夹被高亮显示。右侧代码区域，第1行是DOCTYPE声明，第2行开始是html标签，第5行是head标签，其中包含meta标签和link标签，link标签的title属性值为“中州养老”。该图片与文档中“浏览器标签页logo标识、标题”部分内容相关，用于说明在若依框架项目中找到index.html文件，把title更换为自己想要的内容即可实现标题更换。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/QctObQTqroxKU4xRREQcHjUmnfd.png)

- 标题替换

若依的系统页面标题引用的是开发环境的配置，我们只需要修改开发的环境的VITE_APP_TITLE属性即可

如下图：

![图片展示了若依框架项目开发环境的配置文件内容。左侧是项目文件目录，突出显示了.env.development文件。右侧是.env.development文件内容，其中第2行“VITE_APP_TITLE = 中州养老”被红色框线突出显示。该图片与上下文紧密相关，用于说明若依的系统页面标题引用的是开发环境的配置，只需修改开发环境的VITE_APP_TITLE属性即可，此图直观呈现了配置文件中相关属性的位置。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/VNp0b08UkonWsHxLQl5c4JRAnme.png)

<callout emoji="🐸">
扩展阅读：
关于环境的文件一般有三个
- .env.development   开发环境的配置，默认的环境
- .env.production   生产环境
- .env.staging   测试环境
> 在打包或启动项目的时候，可以加参数来使用不同的环境，需要参考package.json中的配置
</callout>

- logo替换

找到当天资料中的logo文件，替换 src/assets/logo/logo.png文件

![图片展示了若依框架项目中logo文件的存放位置。左侧是项目文件夹结构，src/assets/logo文件夹下有logo.png和logo_bak.png两个文件。右侧是logo.png文件内容，为一个绿色的房屋图案。该图片与文档中“logo替换”部分内容相关，说明找到当天资料中的logo文件，替换src/assets/logo/logo.png文件，以完成logo标识更换操作。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/Yn3BbMj2Vo2TKqx2LHZcwC0enqb.png)

最终效果：

![图片展示了若依框架系统页面的最终效果。左侧为导航栏，显示“中州养老”标识。中间是系统首页内容，上方有搜索框，下方有“首页”“访问主页”按钮。右侧有“访问主页”“访问文档”“访问源码”按钮。页面底部显示技术选型，包括前端技术、后端技术、数据库等。该图对应文档中“登录名称及背景图”部分，展示了登录名称更换后的系统页面效果。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/EJtLbkU2foaqyixFbNicK0BcnYd.png)

### 登录名称及背景图

登录名称和背景图，我们可以直接找到登录的组件进行修改即可

组件位置：src/views/login.vue

- 登录名称

![图片展示了若依框架项目中登录组件的代码片段，用于修改登录名称。代码中第4行的<h3 class="title">中州养老</h3>部分被红色框突出显示。这与文档中“登录名称及背景图”部分的内容相关，文档提到可直接找到登录组件进行修改，此图即为登录组件代码，其中的“中州养老”即为登录名称，可通过修改该代码来更换登录名称。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/AF5xbZMrPoVRFtxBFFWcDsWEnfh.png)

- 背景图

![这是一段Vue项目的SCSS样式代码片段，对应文档中更换登录页面背景图的操作步骤内容，代码内的`.login`类选择器通过`background-image`属性设置登录页面的背景图路径，该路径指向项目`src/assets/images`目录下的`background-login.jpg`文件，代码还设置了`.login`容器为flex布局，将容器的高度设置为100%，并通过`background-size`指定背景图尺寸适配覆盖容器，这段代码用于实现登录页面的背景图样式配置，和文档中说明的更换登录背景图的操作直接相关。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/YUTtbhKzoolG5kxxpHNc9TGdnYe.png)

> 资料中已经提供了背景图，可以自行命名更换

最终效果图：

![图片展示的是若依框架项目登录页面的最终效果。页面背景为多人在室内交谈的场景，上方有“中州养老”标题。登录框内有“admin”用户名输入框、“1”验证码输入框及“登录”按钮。页面底部有“Copyright © 2018 - 2023 ifeng.vip All Rights Reserved”版权信息。该图对应文档中“登录名称及背景图”部分，展示了更换登录名称和背景图后的登录页面样式。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/CYZibzIojo61n6xTuIic134wnEb.png)

### 去除源码地址 & 文档地址 & 若依官网标识

- 利用菜单功能，去除**若依官网**菜单项目
- 使用VS Code全局搜索功能，去除右上角的**源码地址**和**文档地址**
- 最终效果：

![图片展示了若依后台管理系统界面，左侧为导航栏，有首页、入退管理等选项。右侧是首页内容，上方有搜索、全屏、最小化窗口等图标，右上角有头像。界面中“系统工具”选项被红色框线突出显示。右上角有红色箭头标注，指出已去除文档和源码地址标识。该图片与文档中“去除源码地址&文档地址&若依官网标识”部分内容对应，直观呈现了去除标识后的系统界面效果。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/DEPmb3qaEo1zdJxODAlcAs6Cn0e.png)

## 主题UI风格调整

在目前的前端项目中，已经提供了非常便利的操作方式，可以切换主题的风格

操作：点击右上角的头像，可以找到**布局设置，**如下操作

![图片展示了前端项目中切换主题风格的操作界面。左侧为用户头像下拉菜单，点击“布局设置”后弹出右侧设置面板。面板中“主题风格设置”部分有深色和浅色主题选项，当前选中深色主题。下方“系统布局配置”部分有多个开关，如“开启TopNav”“开启Tags-Views”“固定Header”“显示Logo”“动态标题”等，其中“是否显示历史导航栏”开关被红色箭头突出指向。该图与文档中介绍的前端项目主题风格调整操作相关，直观呈现了操作步骤及效果。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/XQbxbViCcocTmexqFC5cCwDhned.png)

在前端代码中也有对应的操作，文件位置：src/setting.js

```JavaScript
export default {
  /**
   * 网页标题
   */
  title: import.meta.env.VITE_APP_TITLE,
  /**
   * 侧边栏主题 深色主题theme-dark，浅色主题theme-light
   */
  sideTheme: 'theme-dark',
  /**
   * 是否系统布局配置
   */
  showSettings: true,

  /**
   * 是否显示顶部导航
   */
  topNav: false,

  /**
   * 是否显示 tagsView
   */
  tagsView: true,

  /**
   * 是否固定头部
   */
  fixedHeader: false,

  /**
   * 是否显示logo
   */
  sidebarLogo: true,

  /**
   * 是否显示动态标题
   */
  dynamicTitle: false,

  /**
   * @type {string | array} 'production' | ['production', 'development']
   * @description Need show err logs component.
   * The default is only used in the production env
   * If you want to also use it in dev, you can pass ['production', 'development']
   */
  errorLog: 'production'
}

```

更换主题颜色：

文件位置：src/store/modules/settings.js

![图片展示了前端代码文件src/store/modules/settings.js中关于主题颜色设置的部分代码。其中，红色框突出显示了“theme: storageSetting.theme || '#40B8F0'”这一代码，表示主题颜色默认为#40B8F0。该图片与上下文“更换主题颜色”相关，是对前端代码中更换主题颜色操作的代码示例说明，帮助理解代码实现。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/QuHJbGfcxoOMAYxJ0DRc9F7hnsh.png)

```JavaScript
theme: storageSetting.theme || '#00b8a0',
```

更换后的效果：

![这张图片展示的是护理项目管理界面，左侧导航栏中“护理项目”处于选中状态，界面顶部有“新增”“修改”“删除”操作按钮，支持按名称、状态筛选查询项目列表。列表内显示了翻身拍背、复健运动、公共空间清洁这三个护理项目，包含对应图片、名称、价格、单位、排序号、创建人、状态、创建时间等信息，每个项目右侧的“操作”列，都提供了修改和禁用的操作选项，部分项目的状态显示为“启用”，部分显示为“禁用”，该界面是前端项目主题UI风格调整更换后的效果。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/Y3MNbmwXRoPoNDxm5HrcHHsunsb.png)

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第一章-若依+AI快速开发新项目 / 01-从0开始用若依 / 定制化UI界面 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/MbWlweTmOid5V9kb2CPcARXznAe)
