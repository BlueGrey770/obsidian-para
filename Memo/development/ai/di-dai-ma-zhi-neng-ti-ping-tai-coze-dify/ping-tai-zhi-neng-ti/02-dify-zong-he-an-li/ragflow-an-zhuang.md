---
type: memo
date: 2026-11-06
aliases:
  - "RAGFlow安装"
---

# RAGFlow安装

## 镜像准备

上传资料中的RAGFlow的镜像文件到服务器中，加载为镜像

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/EDbjbEOsroWSnrxmXDrcA6TfnWd.png)

执行命令：

```Bash
docker load -i elasticsearch.tar | docker load -i minio2025.tar | docker load -i mysql.tar | docker load -i ragflow.tar | docker load -i valkey.tar 
```

> 镜像较大，加载时长要耐心等待



注意：由于这几个镜像占用内存较大，建议把虚拟机的内存调大，10g左右即可

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/Noc0bzcSVoMmNvx6quqcFhm5nSg.png)





## 下载github项目源码

<bookmark name="GitHub - infiniflow/ragflow: RAGFlow is a leading open-source Retrieval-Augmented Generation (RAG) e" href="https://github.com/infiniflow/ragflow.git"></bookmark>

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/SboPb6Ffkon4IXxq59jcFFb5njC.png)



然后解压`ragflow-main.zip` 压缩包（当天资料中已提供，最好创建一个目录）

```Bash
unzip ragflow-main.zip
```

由于前面我们已经安装了Dify项目，这两个项目都依赖了redis，且web端的端口都是默认80端口，因此，为了避免冲突，我们需要修改ragflow-main目录下的相关配置文件。

## 修改配置信息

### 修改web访问默认端口

修改文件位置： `ragflow-main/docker/docker-compose.yml` 文件

**原来的**                                                                                    **修改后的**

<grid>
<column width-ratio="0.500836">
![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/Gwr3bMHDMo1Xdkxk2TAczMuEnbc.png)
</column>
<column width-ratio="0.499164">
![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/KSa0bsUSOoKz2Cxt7MZcmpt9nHh.png)
</column>
</grid>

### 修改redis配置

修改 `ragflow-main/docker/.env` 文件

**原来的**                                                                          **修改后的**

<grid>
<column width-ratio="0.466161">
![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/IWKdbNmHLoVo9rxwreHc1QaMnid.png)
</column>
<column width-ratio="0.533839">
![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/ZRGjbk3wBotKOuxbJcBcxpeNnN8.png)
</column>
</grid>

修改 `ragflow-main/docker/docker-compose-base.yml` 文件

**原来的**                                                                                    **修改后的**

<grid>
<column width-ratio="0.516146">
![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/Uh6ubqFdMoqEOWx47Sxcx4ZWnCe.png)
</column>
<column width-ratio="0.483854">
![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/QvI5bXF6Ko05KJx3gxucOqL8nDh.png)
</column>
</grid>

## 启动服务

进入docker目录，执行命令：

```Python
docker-compose up -d
```

![](https://feishu.cn/file/R9Mfbt2bgoEo5Fx8hc1cRA9znQb)

时间比较久，需要大家耐心等待。



## 访问RAGFlow

由于前面，我们把web端口设置为了8880端口，docker镜像拉取后，等待容器启动完成，在浏览器输入:`192.168.100.168:8880` 即可访问

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/CyArbZqvsoqnUqx8zdGccVoVn0f.png)

至此，ragflow就安装完成了，注册完成后主页面如下：

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/B9mQbckkvomYhixQqR5c7J1jnkV.png)

## 接入大模型

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/DvPNbv6rxo6WB5xnjgTcc1C9nge.png)

![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/Fm7kb2yzIoAAM1x0Rppc2zISnFh.png)



<grid>
<column width-ratio="0.219095">
![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/EHHLbIdxUoKcuTxyON3cfmqcn4c.png)
</column>
<column width-ratio="0.780905">
![](../../../../../../Attachments/development/ai/di-dai-ma-zhi-neng-ti-ping-tai-coze-dify/ping-tai-zhi-neng-ti/02-dify-zong-he-an-li/BVDMbU0a0on2DOxiGwFcN2hYnqf.png)
</column>
</grid>

---
> 来源：飞书技术知识库（脱敏版） · 06 AI 应用开发 / 低代码智能体平台（Coze / Dify） / 平台智能体 / 02-Dify综合案例 / RAGFlow安装 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/CIVpwEkM4iObtNkugtPcINu3ntb)
