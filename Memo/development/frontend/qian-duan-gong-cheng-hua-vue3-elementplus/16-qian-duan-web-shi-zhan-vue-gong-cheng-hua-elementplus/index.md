---
type: memo
date: 2026-11-06
aliases:
  - "16 前端Web实战(Vue工程化+ElementPlus)"
---

<title>16 前端Web实战(Vue工程化+ElementPlus)</title>



# 创建vue项目

创建一个工程化的Vue项目，执行命令：`npm create vue@3.16.4`

![这张图片展示了创建Vue3项目执行命令的终端操作界面，核心内容为：执行`npm create vue@3.16.4`命令创建项目，界面标注该命令（序号1）；提示执行操作需安装create-vue相关包，输入项目名称后直接回车可创建空项目（序号3为对应操作提示）；后续可执行`cd vue-test`命令切换到项目目录（序号4为对应命令）。此外，执行npm install项目依赖的操作也在界面中显示，整个过程的操作步骤都在终端界面中清晰呈现，对应文档里创建Vue工程化项目的命令执行步骤。](../../../../../Attachments/development/frontend/qian-duan-gong-cheng-hua-vue3-elementplus/16-qian-duan-web-shi-zhan-vue-gong-cheng-hua-elementplus/TO8NbsHiLoqTitxeZXocVen9nDb.png)

<callout emoji="✅">
**提示：**执行上述指令，将会安装并执行 create-vue，它是 Vue 官方的项目脚手架工具
</callout>

项目创建完成以后，进入`cd vue-test` 项目目录，执行命令安装当前项目的依赖：`npm install`

![这张图片展示了Vue项目初始化完成后的终端界面内容，项目路径为H:\\web_code\\itcast_web\\webwork\\webvue\\test\\vue-test。界面清晰列出了项目初始化后可执行的命令，包含核心步骤提示：需先执行安装依赖命令`npm install`，再执行启动项目命令`npm run dev`，同时标注了对应操作的红框及文字说明。图片与文档中创建Vue项目的内容相呼应，直观呈现了项目创建后执行依赖安装、项目启动命令的终端执行结果，能看到执行`npm install`后添加了147个包的相关反馈，符合文档中关于项目创建后需联网安装依赖的操作说明。](../../../../../Attachments/development/frontend/qian-duan-gong-cheng-hua-vue3-elementplus/16-qian-duan-web-shi-zhan-vue-gong-cheng-hua-elementplus/CY4lbBnayo9KEwxhCjvcNbKWnTh.png)

创建项目以及安装依赖的过程，都是需要联网的。【如果网络不太好，可能会造成依赖下载不完整报错，继续再次执行 命令安装。】 



**在浏览器输入http://localhost:5173验证看看能不能进入到项目默认主页**

![图片展示的是Vue App的默认主页界面。左侧有绿色的Vue标志及“Congratulations! You’ve successfully created a project with Vite + Vue 3.”的提示。右侧是关于Vue的文档、工具、生态系统、社区及支持Vue的相关信息。该图片与上文“在浏览器输入http://localhost:5173验证看看能不能进入到项目默认主页”的内容相呼应，直观呈现了项目创建成功后的主页样式。](../../../../../Attachments/development/frontend/qian-duan-gong-cheng-hua-vue3-elementplus/16-qian-duan-web-shi-zhan-vue-gong-cheng-hua-elementplus/HSFmbxVuao1MtfxTGXacBJshn5c.png)

# 项目开发流程

```Java
项目中的package.json文件就类似于我们在Java开发中的pom.xml文件是管理整个项目的架构的
项目中文件的调用顺序：main.js-->App.vue-->Main.vue(这个也是我们自己定义的是用来显示程序主页的信息的)-->.....
```

# 使用elementplus

```Java
在使用elementplus时我们需要找到自己想要的组件，然后把template中的代码拷贝下来到项目中
再观察这个组件中<script>中的代码和使用组件需要引入的东西，import ....
如果想成功使用一个组件的效果以上的这些代码必须要复制到项目中，缺一不可
```

---
> 来源：飞书技术知识库（脱敏版） · 03 Web 开发与框架 / 前端工程化（Vue3 + ElementPlus） / 16 前端Web实战(Vue工程化+ElementPlus) · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/CoKCwYR46iSDPRkww9ncvtvpn1c)
