---
type: memo
date: 2026-11-05
aliases:
  - Vtable 表格增强插件
  - Obsidian 富文本表格
---
# Vtable：Obsidian 表格增强插件

解决原生 Markdown 表格的硬伤：单元格里按 `Enter` 只能跳格不能换行、不能合并单元格、不能算合计、做图表要绕到 Excel。Vtable 把表格从「纯文本语法」升级为「可带格式、可计算、可出图的 JSON 结构」，仍嵌在笔记里。

本库状态：已安装 v2.0.13（作者 Vmsimab），经 BRAT 安装。插件开源，MIT 协议。

![](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vtable-overview.png)

## 一 安装

**方式一 BRAT（本库采用）**：本库已装 BRAT → 设置中打开 BRAT → Add a beta plugin for testing → 填 `https://github.com/Ming145/Vtable` → 选择最新版本 → 启用。

**方式二手动**：从 [Gitee Releases](https://gitee.com/vmsimab/vtable/releases) 下载 `main.js`、`manifest.json`、`styles.css`，放入 `.obsidian/plugins/vtable/`，再到第三方插件设置中启用。

## 二 核心能力

| 能力 | 说明 |
| --- | --- |
| 格内换行 | 单元格内 `Enter` 即换行，像写文档一样做表格 |
| 合并单元格 | 支持跨行跨列与拆分还原；**一次只能合并两格**，多格需先合并再合并，`Ctrl+Z` 逐步撤销，拆分则一次全撤 |
| 公式计算 | `=SUM(A1:B3)` 直接出结果，支持 SUM、AVERAGE、COUNT、MIN、MAX |
| 一键图表 | 选中数据生成柱状图、饼图、折线图，自动适配深浅色主题 |
| 图表模板 | 启用后自动在模板文件夹生成三种图表模板，打开即填数据 |
| 单元格富文本 | 粗体、斜体、`==高亮==`、`~~删除~~`、行内代码、链接、图片嵌入、LaTeX、HTML 标签 |
| 格式互转 | Markdown ↔ vtable ↔ HTML，可当转换器用 |
| 浮动工具栏 | 加粗、斜体、字号、合并、插入行列 |

公式演示：

![公式计算演示](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vtable-formula.gif)

## 三 三种显示模式（OL / HS / RC）

进入 Live Preview 编辑表格时，表格**左上方**出现三个半透明切换按钮：

| 模式 | 行为 |
| --- | --- |
| OL 原始换行（默认） | 单元格内回车换行，超宽自动折行 |
| HS 横向拉长 | 文字不换行持续右延，表格下方有可拖拽滚动条（桌面/移动端均可拖） |
| RC 自由拖拽 | 单元格右/下边缘出现拖拽手柄，内容增多自动撑高行 |

- 模式选择会写入代码块 JSON 并记忆，下次打开自动恢复。
- `colWidths` / `rowHeights` **只在 RC 模式生效**，不影响 OL/HS 显示。
- 内容减少时行高**不自动缩小**，需要手动拉。
- 预览模式、源码模式、未聚焦单元格时按钮自动隐藏；可在设置里关闭按钮显示。

## 四 使用判断

什么时候值得用 Vtable：有合计计算、需要合并表头、要把表格变成图表、单元格里要塞公式或图片——这类「数据型表格」。

什么时候继续用原生 Markdown 表格：纯文字对照表、记录类的两列表。原因是 vtable 是代码块形式，**一旦脱离 Obsidian（GitHub 预览、其他 Markdown 编辑器、部分静态站点生成器）就不渲染**，会退化成一段 JSON。工具选型跟随用途，不必把所有表格都升级。

## 五 参考

- [Obsidian 中文论坛发布帖](https://forum-zh.obsidian.md/t/topic/62760)：功能演示与安装说明，本文配图来源
- [Gitee 仓库与 README](https://gitee.com/vmsimab/vtable)：细节文档、Issue 反馈
- [GitHub 仓库](https://github.com/Ming145/Vtable)
