---
type: memo
date: 2026-11-06
aliases:
  - "16 NodeJS安装"
---

# 16 NodeJS安装

**1). 双击msi文件，勾选我接受**

![](../../../../../Attachments/development/frontend/qian-duan-gong-cheng-hua-vue3-elementplus/16-qian-duan-web-shi-zhan-vue-gong-cheng-hua-elementplus/VCfJbKEvNoHMtPxMOz7cQZcjnFh.png)



**2). 选择安装到一个，没有中文，没有空格 的目录下（新建一个文件夹NodeJS）**

![](../../../../../Attachments/development/frontend/qian-duan-gong-cheng-hua-vue3-elementplus/16-qian-duan-web-shi-zhan-vue-gong-cheng-hua-elementplus/KVS9bGWQdohyjixMNvlcfn2VnEg.png)



**3). 点击 Next，下一步下一步的安装即可。**

![](../../../../../Attachments/development/frontend/qian-duan-gong-cheng-hua-vue3-elementplus/16-qian-duan-web-shi-zhan-vue-gong-cheng-hua-elementplus/WxPLbt0WroH186xfrQxcp8sYnNe.png)

4).安装完毕

![](../../../../../Attachments/development/frontend/qian-duan-gong-cheng-hua-vue3-elementplus/16-qian-duan-web-shi-zhan-vue-gong-cheng-hua-elementplus/HebEb8EW9oWdvoxANatcvmfJnLe.png)



**5). 验证NodeJS的环境变量**

NodeJS 安装完毕后，会自动配置好环境变量，我们验证一下是否安装成功，通过： `node -v`

![](../../../../../Attachments/development/frontend/qian-duan-gong-cheng-hua-vue3-elementplus/16-qian-duan-web-shi-zhan-vue-gong-cheng-hua-elementplus/MXC4bmTYVoC7T8xBkbBch7Gjnke.png)





**5). 配置npm的全局安装路径**

![](../../../../../Attachments/development/frontend/qian-duan-gong-cheng-hua-vue3-elementplus/16-qian-duan-web-shi-zhan-vue-gong-cheng-hua-elementplus/DWjQbac1xouzO9xyfy1cphUanPf.png)



使用 **管理员身份** 运行命令行，在命令行中，执行如下指令：

![](../../../../../Attachments/development/frontend/qian-duan-gong-cheng-hua-vue3-elementplus/16-qian-duan-web-shi-zhan-vue-gong-cheng-hua-elementplus/QU6kbeeNvoQ4XWxUlQfceRXDntc.png)

```XML
npm config set prefix "H:\develop\nodejs\install"
```

**注意：D:\develop\NodeJS  这个目录是NodeJS的安装目录 !!!!!**



**6). 切换为淘宝镜像，加速下载：**

![](../../../../../Attachments/development/frontend/qian-duan-gong-cheng-hua-vue3-elementplus/16-qian-duan-web-shi-zhan-vue-gong-cheng-hua-elementplus/MWuGbmIExo53Ofx22G7cj9JLnKc.png)

```Java
npm config set registry https://registry.npmmirror.com
```

---
> 来源：飞书技术知识库（脱敏版） · 03 Web 开发与框架 / 前端工程化（Vue3 + ElementPlus） / 16 前端Web实战(Vue工程化+ElementPlus) / 16 NodeJS安装 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/Ut2DwRbX1iKgY1kqbdOcMpffnbb)
