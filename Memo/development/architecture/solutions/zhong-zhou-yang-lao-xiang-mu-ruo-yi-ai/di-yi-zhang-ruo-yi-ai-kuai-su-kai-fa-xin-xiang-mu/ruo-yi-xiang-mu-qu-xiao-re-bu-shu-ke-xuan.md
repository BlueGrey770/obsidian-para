---
type: memo
date: 2026-11-06
aliases:
  - "若依项目取消热部署(可选)"
---

# 若依项目取消热部署(可选)

1. 在admin模块删除依赖

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/SUSWbqnbFo7O3lxlQcycnZYKnBd.png)

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/KgKqbPvRXoSuUNxUp6HcFIAqn9c.png)

1. 在admin的yml配置文件里面把热部署开关给删除配置

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/S1KQb5jE8oFV1ZxQIADcobf7nMf.png)

1. 修改启动项配置

![](../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/Pm4ebERYzoy3kyxoZWvccV8InMh.png)

1. 刷新maven，重启项目就不会自动热部署了

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第一章-若依+AI快速开发新项目 / 若依项目取消热部署(可选) · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/GAnywDzV0i1CRJkasShcp2YOn8c)
