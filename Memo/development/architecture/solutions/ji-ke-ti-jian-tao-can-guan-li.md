---
type: memo
date: 2026-11-06
aliases:
  - "即刻体检-套餐管理"
---

# 即刻体检-套餐管理

## 4.4体检组管理相关接口

### 4.4.1管理端--体检组相关接口

#### 1）新增检查组

接口名称：新增检查组

接口路径：POST   /health/admin/check-group

![](../../../../Attachments/development/architecture/solutions/EquEbK4LkoPzRNxpCi0cnacEnOE.png)

![](../../../../Attachments/development/architecture/solutions/ThgUbKo5xoFwPOxqKsYcJ0Znn8b.png)

#### 2）分页查询检查组

接口名称：分页查询检查组

接口路径：GET   /health/admin/check-group/pageQuery

<callout emoji="🔮">
参数说明：本接口的排序参数，前端发起请求时固定为
- orderBy1：groupSortNum
- orderBy2：createTime
- isAsc1：true
- isAsc2：false
</callout>

![](../../../../Attachments/development/architecture/solutions/QMkkbFzo5oo2QJxLeRPcW4XjnTe.png)

![](../../../../Attachments/development/architecture/solutions/C3xobhAalo8DcDxbWtNc0mZJnWf.png)

#### 3）根据id查询检查组

接口名称：根据id查询检查组

接口路径：GET   /health/admin/check-group/{id}

![](../../../../Attachments/development/architecture/solutions/Uko4bM8mwonln9x6l3cc0xGFn1b.png)

![](../../../../Attachments/development/architecture/solutions/YWIPbaitWoOYDZx8fKNcDNr2nth.png)

#### 4）修改检查组

接口名称：修改检查组

接口路径：PUT   /health/admin/check-group/{id}

![](../../../../Attachments/development/architecture/solutions/AtdVbdTiTomzLJxAdBtcfO6KnMf.png)

![](../../../../Attachments/development/architecture/solutions/E2fPb8bhOo75dsxQCUBcFhnJncf.png)

#### 5）根据id启用检查组

接口名称：根据id启用检查组

接口路径：PUT   /health/admin/check-group/enable/{id}

![](../../../../Attachments/development/architecture/solutions/NA45bxRAMod9xbx2AOZcEy5jnJc.png)

![](../../../../Attachments/development/architecture/solutions/ImRrbnhUtoh10CxguhJcYlxDnDg.png)

#### 6）校验检查组是否可以禁用

接口名称：校验检查组是否可以禁用

接口路径：GET   /health/admin/check-group/checkDisable/{id}

<callout emoji="🔮">
需求说明：本接口用于用户禁用体检组时，提前校验进行弹窗提示。
逻辑说明：需要校验该体检组关联的套餐有哪些，如果有关联的套餐，返回套餐名称列表，并将canDisable参数置为false；否则套餐名称列表为null，canDisable参数置为true；
</callout>

![](../../../../Attachments/development/architecture/solutions/JD0lbd8Bzo6kejx9GzPcAywxnuc.png)

![](../../../../Attachments/development/architecture/solutions/XES7buN4doIJ5DxkcavcIoZ9n6g.png)

#### 7）根据id禁用检查组

接口名称：根据id禁用检查组

接口路径：PUT   /health/admin/check-group/disable/{id}

<callout emoji="🔮"><p>说明：业务逻辑请参考<cite doc-id="FevNwiL4lizEtWkGxd7cx8Snn7g" file-type="wiki" title="即刻体检需求文档" type="doc"></cite></p></callout>

![](../../../../Attachments/development/architecture/solutions/EUvsbXnMBoRhQixIgFVcrIkCnDd.png)

![](../../../../Attachments/development/architecture/solutions/LFI4bdTFEoFv0NxubG1cpSDvnsd.png)

#### 8）根据id删除检查组

接口名称：根据id删除检查组

接口路径：DELETE   /health/admin/check-group/{id}

![](../../../../Attachments/development/architecture/solutions/Mn01b7OCcoePsixpSNfcYFSQn2b.png)

![](../../../../Attachments/development/architecture/solutions/C90EbOQiOoOhbOxIFYDcBQ6Jnud.png)

#### 9）体检组下拉列表

接口名称：体检组下拉列表

接口路径：GET   /health/admin/check-group/listForFilter

<callout emoji="🔮">
说明：本接口用于体检项新增页面，需要查询启用状态的体检组
</callout>

![](../../../../Attachments/development/architecture/solutions/Sn9DbkFIDocCdVx0t6WcRCxbnsd.png)

![](../../../../Attachments/development/architecture/solutions/MHWxbKzCpoCyH4xaLdxc59jZnoc.png)

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 即刻体检-套餐管理 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/QWEcwAmGXi4DickPbI0cGuW6nag)
