---
type: memo
date: 2026-11-07
aliases:
  - 思维导图插件教程
  - Light Mindmap 与 Mindmap Mode
---
# 思维导图插件：Light Mindmap 与 Mindmap Mode

本库导图采用「双插件分工」：**Light Mindmap** 负责把 `type: mindmap` 笔记渲染成漂亮的导图成品；**Mindmap Mode** 负责把任意笔记（含导图笔记）切换成可编辑的放射状导图，直接在图上改结构。两者不冲突，可同时启用。

本库状态：Light Mindmap v1.5.0（作者 Light Ning）、Mindmap Mode v1.1.2（作者 PHANTOM-Nw），均已启用并写入 `community-plugins.json`。

## 一 安装

### Light Mindmap

1. 设置 → 第三方插件 → 关闭安全模式 → 社区插件市场搜索 **Light Mindmap** → 安装并启用。
2. 若市场不可达，手动安装：从 GitHub Releases（`ninglg/obsidian-light-mindmap`）下载 `main.js`、`manifest.json`、`styles.css` 放入 `.obsidian/plugins/light-mindmap/` 后启用；国内访问 502 时可走 gh-proxy 镜像加速下载。

### Mindmap Mode

社区市场未上架，本库按手动方式安装：

1. 从 GitHub Releases（`PHANTOM-Nw/mindmap-mode`）下载 `main.js`、`manifest.json`、`styles.css`。
2. 放入 `.obsidian/plugins/mindmap-mode/`，重启 Obsidian 或在第三方插件页刷新后启用。

两者启用后无需额外设置即可使用；`.obsidian` 不入版本库，换机时按 README《新机器恢复清单》重装。

## 二 Light Mindmap：渲染型导图

### 触发方式（唯一开关）

笔记 frontmatter 写 `type: mindmap`（插件对值做硬判断、不区分大小写，写成 memo 或别名一律不触发）。打开文件自动进入导图视图；新建时用 `_templates/mindmap-note.md` 模板可一次带全必需字段。
![[../../../Attachments/Pasted image 20261107013637.png]]

### 解析规则（源码级约定）

- **只认标题**：正则 `^(#{1,6})\s+`，标题层级即分支层级；**列表、表格、正文一律不参与构图**。
- 代码块内的 `#` 行会被跳过，不会误生成节点。
- 最深支持 **6 级**（H1~H6），不设文档级大标题，根节点直接占 H1，把层级额度留给节点树。
- 标题写成 `*文字*` 的分支默认折叠；标题里可写 `[[双链]]`，导图上可点击跳转。
- 正文段落、图片写在标题之下，导图不显示、阅读视图完整显示——这就是「一个文件两种读法」。

### 样式键

frontmatter 支持四个可选键：**四个一次写全**，缺省时插件首次打开会自动回写，造成无意义的版本变更。

| 键 | 可选值（部分） |
| --- | --- |
| `mindmap-layout` | balanced、right、left、tree、radial |
| `mindmap-theme` | vibrant、classic、fresh、ocean 等 6 套 |
| `mindmap-line` | curve、straight、polyline、rounded 等 |
| `mindmap-node` | rounded、square、borderless、circle、doodle |

### 常用操作

导图视图中拖拽调整、点击节点折叠/展开、工具栏切换布局与主题、导出 PNG。阅读/编辑源文件可随时切回 Markdown 视图。

## 三 Mindmap Mode：编辑型导图

### 定位

对**任意一篇笔记**（不要求 `type: mindmap`）按标题层级生成可编辑的放射状导图，在同标签页内切换；所有修改**直接回写原 md 文件**，不产生任何中间文件。适合在导图上增删改节点、调整结构。

### 打开与使用

1. 打开目标笔记 → 命令面板执行 **Toggle mindmap view**（或点左侧边栏图标），当前标签页变为导图。
2. 节点操作：双击编辑文字、拖拽移动挂接、快捷键增删子节点/同级节点；全部改动即时写回原文件。
3. 再次执行同一命令切回 Markdown 编辑视图。

### 内容显示规则

- 标题 → 节点（与 Light Mindmap 相同的骨架逻辑）。
- 标题下的**正文段落 → 挂在节点上的内容卡片**，解决了「导图看不到正文」的问题，可复刻飞书思维笔记「大纲+正文」的观感。
- 形如 `: 文本` 的行 → 节点下方的注释行（更接近飞书样式），但该行在阅读视图中会带着冒号原样显示，本库暂不采用，正文统一写普通段落。
- 图片随正文卡片显示。

### 试用入口

测试副本在 [[Inbox/mindmap-mode-test.md]]（frontmatter 用 `type: memo`，避免被 Light Mindmap 接管视图）。

## 四 协同规则与注意事项

- **分工**：成品展示/导出用 Light Mindmap；结构调整、节点编辑用 Mindmap Mode；同一篇 `type: mindmap` 文件两者都能打开。
- **骨架写法统一**：标题做骨架、正文挂标题下、超过 6 级用缩进列表兜底（两个导图都不解析列表，但阅读视图可见，不丢内容）。完整约定见管理规范「导图笔记」一节。
- **不要给普通笔记加 `type: mindmap`**，否则 Light Mindmap 会接管视图，正文只剩骨架可见。
- obsidian-mind-map（Markmap）保持停用；如确认弃用可连同目录删除。
- 新装的 Quiet Outline（v0.5.22）与导图互补：阅读视图下提供可搜索、可拖拽重排的大纲侧栏，当目录用。
