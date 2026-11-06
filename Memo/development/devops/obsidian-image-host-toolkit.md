---
type: memo
date: 2026-11-05
aliases:
  - Obsidian 图床化与图片管理
  - Image Upload Toolkit 使用教程
---
# Obsidian 图床化：Image Upload Toolkit + 阿里云 OSS

把笔记里的本地图片上传到对象存储，正文只保留 URL，用来解决三个问题：导出或发布时图片丢失、库体积被截图撑大、跨设备查看依赖附件同步。本库采用「插件直传 OSS」的方案，**不需要在本地安装和常驻 PicGo**，链路已于 2026-11-05 实测通过。

## 一 先选方案：直传还是经过 PicGo

| 方案 | 链路 | 本地软件 | 适用场景 |
| --- | --- | --- | --- |
| Image Upload Toolkit（本库采用） | Obsidian → 阿里云 OSS 直传 | 不需要 | 只在 Obsidian 内做图床化 |
| Image Auto Upload Plugin + PicGo | Obsidian → 本地 PicGo 服务（36677 端口）→ OSS | 需 PicGo 常驻运行 | 多个编辑器共用同一图床（Typora、VS Code），或需要全局截图上传 |

选择第一种的理由：少一个后台进程和一个故障点，插件本身已原生支持 OSS、S3、腾讯 COS、七牛、GitHub、R2 等存储。PicGo 的真正价值在「Obsidian 之外」，本库用不到。注意两个方案不能混用，装两个上传插件容易重复上传。

插件限制：`isDesktopOnly`，仅桌面端可用，移动端无法上传（但移动端可正常显示已上传的外链图片）。

## 二 阿里云侧准备

### 1 创建 Bucket

Bucket 名称全局唯一，地域创建后不可更改。存储类型选标准存储，**读写权限必须选「公共读」**，否则图片链接会返回 403，笔记里显示为裂图。

![创建 Bucket 弹窗：名称、地域、存储类型](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/oss-create-bucket.png)
	注意：选择地区时可参考下本章第5小节的成本参考，可根据需要自行选择
![Bucket 列表与创建入口](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/oss-bucket-list.png)

### 2 关闭「阻止公共访问」

新版 OSS 默认开启「阻止公共访问」，开启时会忽略公共读 ACL。必须进入 **权限控制 → 阻止公共访问** 关闭它，再到「读写权限」标签确认为公共读。

![权限控制页：阻止公共访问与读写权限](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/oss-bucket-acl.png)

> 公共读意味着任何拿到 URL 的人都能查看图片，也可能被爬取。这个 bucket 只放笔记配图，不放证件、合同等隐私内容。

### 3 创建 RAM 子账号 AccessKey

不要用主账号密钥。在 **访问控制 RAM → 用户** 新建用户，访问方式勾选「编程访问」，创建后立即保存 AccessKey ID 与 Secret（Secret 只显示一次），再为该用户授权。简单做法是授予 `AliyunOSSFullAccess`；更稳妥的是按下文「六 安全」里的最小权限策略只允许写入指定前缀。

![AccessKey 管理页面](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/oss-accesskey.png)

### 4 记录 Endpoint 与地域代码

在 Bucket 概览页的「访问端口」可以看到 Endpoint，例如 `oss-cn-beijing.aliyuncs.com`。插件里的 **Region** 只填地域代码部分（`oss-cn-beijing`），不要填中文地域名，也不要带 `https://`。

![Bucket 概览：Endpoint 地域节点](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/oss-endpoint.png)

### 5 成本参考

选中国香港地域 Bucket 的长期费用（金额以阿里云官网最新价格为准）：

- 新用户 3 个月内：20GB 存储 + 2GB 下行流量 + 20 万次请求免费。
- 之后每月：5GB 存储 + 5GB 外网下行免费（大陆地域没有下行免费额度，这是选香港的核心原因）。
- 超出部分：忙时（08:00–24:00）约 ¥0.5/GB，闲时（00:00–08:00）约 ¥0.25/GB；**上传流量、内网下行始终免费**。
- 创建时的省钱项：存储冗余选「本地冗余」（更便宜）、版本控制关闭、服务端加密选无；实时日志查询保持开通（免费 7 天，可查流量异常）。定时备份与 HDFS 功能不开。

### 6 地域选择清单：为什么是中国香港

| 选择维度 | 中国香港 | 大陆地域 | 其他海外地域 | 对本库的影响 |
| --- | --- | --- | --- | --- |
| 外网下行免费额度 | **每月 5GB** | 无免费下行 | 部分有 | 图片外链阅读都走下行，这是选香港的核心原因 |
| 绑定自定义域名 / CDN | 无需备案 | **需 ICP 备案** | 无需备案 | 未来想换好看域名时，香港省去备案流程 |
| 国内访问延迟 | 较低（华南尤佳） | 最低 | 明显更高 | 日常写作看图，香港延迟可接受 |
| 新用户免费试用 | 同享 3 个月 | 同享 3 个月 | 同享 3 个月 | 与地域无关，不构成选择理由 |
| 存储单价 | 略高于大陆 | 最低 | 相近 | 笔记配图量级小，差额可忽略 |
| 同地域 ECS 内网下行 | 免费且快 | 免费且快 | 不适用 | 没有 ECS 时此项不构成理由 |

结论：**阿里OSS每月固定的 5GB 存储 + 5GB 外网下行免费额度只在海外地域有效**（中国香港、新加坡、日本、美西等），**深圳等大陆地域没有下行流量免费额度**，本库 Bucket 建在中国香港，大陆使用每月 5GB 免费下行足够覆盖日常笔记配图；若将来图片量或访问量超出免费档，再评估迁回大陆 + CDN，或上自定义域名。具体金额与额度以阿里云官网最新公告为准，每年复核一次。

## 三 安装与配置插件

1. 设置 → 第三方插件 → 浏览，搜索 `Image Upload Toolkit`（作者 Addo Zhang），安装后启用。安装与启用是两个步骤。
![Pasted image 20261105163718](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Pasted%20image%2020261105163718.png)
2. 打开该插件设置页，按下面两张表填写。

本库当前配置（截图即实际界面）：

![插件设置：开关与 OSS 参数](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/obsidian-image-upload-toolkit-settings.png)

| 字段 | 填写内容 | 本库值 |
| --- | --- | --- |
| Image store | 图床类型 | AliYun OSS |
| Region | 地域代码 | China (Hong Kong) |
| Access key ID | RAM 的 AccessKey ID | 已填写 |
| Access key secret | RAM 的 AccessKey Secret | 已填写 |
| Bucket name | Bucket 名称 | `notes-images-para` |
| Target path | 存储前缀 | `/img` |
| Custom domain name | 已备案域名或 CDN，可留空 | 空 |

| 开关 | 建议 | 作用与理由 |
| --- | --- | --- |
| Use image name as alt text | 开 | 图片 alt 用文件名，便于以后检索 |
| **Update original document** | **开** | 上传后把笔记里的本地链接替换成 OSS URL；不开则只把结果复制到剪贴板，正文不变 |
| Ignore note properties | 开 | 不处理 frontmatter |
| Show progress modal | 开 | 批量上传时显示进度与失败项 |
| Upload web images | 关 | 避免把已外链的图片重复搬运 |
| Convert Mermaid diagrams | 按需 | 需要把 mermaid 代码块转成 PNG 并上传时再开 |

「Update original document」是最容易被忽略的一项：它决定你是「只生成一份可发布的 Markdown」还是「直接把笔记改成外链」。

## 四 使用与验证

### 日常操作

1. 打开含本地图片的笔记。
2. `Ctrl+P` 打开命令面板，输入 **publish**，执行 **Image Upload Toolkit: Publish page**（插件只有这一个命令）。
![Pasted image 20261105114000](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Pasted%20image%2020261105114000.png)
3. 插件识别当前笔记里的本地图片（`![[图片.png]]` 与 `![](路径)` 两种写法都支持），批量上传到 `bucket/img/`，并按开关替换正文链接。
![7af04c74320461ac949c457d2d3b65b2](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/7af04c74320461ac949c457d2d3b65b2.png)
4. 切到阅读视图确认图片正常显示。

> 实时预览（编辑视图）不渲染远程图片，外链图只在阅读视图显示，表格里嵌远程图同理。这不是上传失败，是 Obsidian 的渲染行为。

### 本库实测记录

用同一套密钥直接发签名请求验证过整条链路：

| 测试 | 结果 | 结论 |
| --- | --- | --- |
| 签名 PUT 上传 | 200 | 密钥有效、bucket 可写 |
| 无签名 GET 访问 | 200 | 公共读生效，URL 可直接渲染 |
| 签名 DELETE 清理 | 204 | 权限完整，无残留 |

若第二次测试返回 `403 AccessDenied ... because of bucket acl`，说明 bucket 仍是私有，或「阻止公共访问」没关。

上传成功后，笔记正文里的链接会变成这种形式：

![上传后的 Markdown 链接形态](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/oss-markdown-result.png)

## 五 常见问题

| 现象 | 优先排查 |
| --- | --- |
| 提示 publish failed | 密钥、Region 代码、Bucket 名三处最常见；Region 不能填中文或完整 URL |
| 上传成功但笔记没变化 | `Update original document` 是否开启 |
| 图片链接 403 或裂图 | Bucket 是否公共读、是否关闭「阻止公共访问」、密钥是否只有该前缀权限 |
| 移动端无法上传 | 插件仅桌面端，属正常限制 |
| 笔记里的图片没被处理 | 图片是本地文件还是已是 http 链接；后者需开 `Upload web images` |
| 表格或外链图片在编辑视图不显示 | 实时预览不渲染远程图片，切阅读视图属正常 |
| 重复上传同一张图 | 插件按文件名生成对象，改名后会产生两份，属预期 |

## 六 安全与备份

- AccessKey 以明文存放在 `.obsidian/plugins/image-upload-toolkit/data.json`。本库若加入 Git 或云同步，需确认不会推到公开仓库；`obsidian-git` 建议只推私有仓库。
- 推荐的最小权限策略（RAM 自定义策略，只允许操作该 bucket 的 `img/` 前缀）：

```json
{
  "Version": "1",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["oss:PutObject", "oss:GetObject", "oss:DeleteObject"],
      "Resource": ["acs:oss:*:*:notes-images-para/img/*"]
    }
  ]
}
```

- 定期轮换 AccessKey；怀疑泄露时先在 RAM 禁用再更换插件配置。
- 公共读 bucket 不放隐私图片；重要截图本地留一份，上传后把原文件移到 `_archive` 而不是直接删除，便于断网或误删时找回。
- 计费按存储量与公网流出流量计算，图片多、访问多时注意流量费用。
- 可选加固：给 Bucket 配置 **Referer 防盗链**，防止外站盗链刷流量；控制台开启**余额预警**（如低于 ¥10 提醒）。
- 可选省钱：图片 URL 追加 `?x-oss-process=image/resize,w_1000` 按宽度压缩，省存储与下行流量；笔记里的链接需要自行追加该后缀。

## 七 与知识库规范的关系

- 附件仍按 [[obsidian-vault-management-spec|管理规范]] 先落 `Attachments/<领域>/<主题>/`，图床化是「上传动作」，不改变附件归档规则。
- 外链化后正文依赖网络：断网时阅读视图看不到图。重要资料保留本地副本，或在上传前先确认图片已归档。
- `Attachments` 与 `_archive` 本来就不在图谱白名单内，图片外链化不影响图谱结构。
- 链接变成外链后，Obsidian 的「未解析链接」不再追踪这些图片；不要用双链语义给图片命名。
- Obsidian 的默认附件位置已在 `app.json` 设为 `Attachments`，新粘贴的图片不会再落到库根目录。

## 八 验收清单

- [ ] Bucket 为公共读，且「阻止公共访问」已关闭
- [ ] AccessKey 来自 RAM 子账号，权限只覆盖所需前缀
- [ ] 插件 Image store / Region / Bucket / Target path 与 OSS 实际值一致
- [ ] `Update original document` 已开启
- [ ] 至少一篇笔记完成 Publish page，阅读视图可正常显示
- [ ] 本地原图已决定保留或移入 `_archive`
- [ ] `data.json` 不会进入公开仓库

## 九 备选方案：PicGo + Image Auto Upload（本库未采用）

同一个 OSS Bucket 也可以走「Image Auto Upload Plugin + 本地 PicGo」链路：粘贴图片时插件自动调用常驻的 PicGo 桌面端（默认监听 `http://127.0.0.1:36677/upload`）完成上传，**粘贴即传，无需手动执行命令**。代价是要多养一个常驻进程。

适用条件：需要在 Typora、VS Code 等多个编辑器共用同一图床，或需要 Obsidian 之外的全局截图上传时，才值得走这条链路。只在 Obsidian 内使用就用主方案。

### PicGo 侧配置

| 字段 | 填写 |
| --- | --- |
| 图床类型 | 阿里云 OSS |
| AccessKey ID / Secret | RAM 子账号密钥 |
| 存储空间名 | Bucket 名称 |
| 存储区域 | `oss-cn-hongkong`（地域代码，**不带** `.aliyuncs.com` 后缀） |
| 指定存储路径 | `img/` |
| 自定义域名 | 有 CDN 才填，否则留空 |

只有「自定义节点 / Endpoint」字段才填完整 `https://oss-cn-hongkong.aliyuncs.com`，与「存储区域」不要混填。界面中文化：左下角齿轮 → Settings → General → Language → 简体中文 → 重启。

### Obsidian 侧配置

安装并启用 Image Auto Upload 插件，默认上传器选 `PicGo(app)`，服务地址填 `http://127.0.0.1:36677/upload`，开启 Auto pasted upload。**PicGo 必须保持后台运行**，否则粘贴上传静默失败；使用前可在浏览器访问该地址确认服务在线。

### 该链路特有排错

| 现象 | 排查 |
| --- | --- |
| 粘贴没反应 | PicGo 未运行、36677 端口未开、上传器没选 PicGo(app) |
| 上传报错 AccessDenied | RAM 子账号未授权该 Bucket |
| 图片 URL 打不开 | 阻止公共访问未关闭，或权限未改公共读（与主方案相同） |

两条链路共用同一个 Bucket 时，插件与 PicGo 的命名规则不同，同一张图可能产生两份对象；切换方案前先统一命名或清理。

## 十 参考来源

- [插件仓库 README](https://github.com/addozhang/obsidian-image-upload-toolkit)：功能与存储参数说明
- [阿里云 OSS 创建 Bucket 与读写权限](https://help.aliyun.com/zh/oss/user-guide/create-buckets)：公共读、阻止公共访问的官方说明
- [知乎《阿里云 OSS PicGo 配置图床教程》](https://zhuanlan.zhihu.com/p/104152479)：核对 RAM 子账号与公共读要求
- [CSDN《通过 Typora + PicGo + 阿里云 OSS 实现图床》](https://blog.csdn.net/greek7777/article/details/146407512)：本文控制台界面截图来源，仅借用界面示意，本库实际不使用 PicGo
- 第九节 PicGo 备选方案与第二节成本信息整理自使用者本人的配置记录（2026-11-05）
- 本库相关：[[obsidian-plugin-setup-guide|Obsidian 插件安装与基础配置教程]]、[[obsidian-vault-management-spec|个人知识库管理规范]]
