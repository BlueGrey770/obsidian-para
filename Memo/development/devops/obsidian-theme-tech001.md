---
type: memo
date: 2026-11-05
aliases:
  - Tech001 主题
  - Obsidian 开发者主题
---
# Tech001：面向开发者的 Obsidian 主题

Tech001 是面向开发者、技术写作者与极简主义者的双模式高对比主题，视觉语言取自 **VS Code + Google AI Studio / Material 3**。它不靠花哨配色取胜，卖点是"把 Obsidian 收拾成一个干净的代码编辑器 + 文档站"。

本库状态：已安装并启用 v1.0.4（作者 Roman Volodin），`.obsidian/appearance.json` 中 `cssTheme` 已指向 Tech001。

## 一 功能特点

**双人格配色**

| 模式 | 特征 | 适合 |
| --- | --- | --- |
| 深色（AI Studio 风） | 深灰底、暖金色标题、VS Code Dark+ 语法色、Gemini 蓝强调色，低眩光 | 夜间长时段写作、读代码 |
| 浅色（Material 3 风） | 暖白底、近黑标题、Light+ 语法色、Google 蓝强调 | 白天起草与校对 |

![Tech001 深色模式](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/tech001-dark.png)

![Tech001 浅色模式](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/tech001-light.png)

**浮动卡片工作区**：左右侧栏、编辑区、库切换器都变成圆角卡片，浮在凹陷背景上，面板之间有留白呼吸感——不再是从边到边的拼贴。

**VS Code 式标签页**：方形扁平标签彼此贴合，激活标签取编辑区颜色并在顶部画一条细强调线，视觉上与正文连成一体。

**开发者细节**
- 文件树：点状标记、收紧行距、折叠箭头对齐，悬停/选中高亮覆盖整行（箭头、图标、名称一起）。
- 代码块：带边框圆角，语言标签与复制按钮按文档站风格重做。
- 表格无边框 + 表头填充色；callout、引用、复选框统一圆角；LaTeX 字号加大。
- 隐藏标题栏与同步状态图标，减少框架干扰。

**排版**：正文居中成固定宽度列（约 1100px），滚动条贴在窗口最右缘；标题轻量高对比，深色模式正文略降对比以减轻眼疲劳。

## 二 安装与推荐设置

安装：设置 → 外观 → 主题 → 管理 → 搜 `Tech001` → 安装 → 使用。手动方式：把 `theme.css`、`manifest.json` 放入 `.obsidian/themes/Tech001/`。

推荐设置（不设的话主题效果会打折）：

| 设置项 | 值 | 原因 |
| --- | --- | --- |
| 编辑器 → 可读行宽 | **关闭** | 1100px 居中列只在关闭时生效，开启后会被 Obsidian 自带的窄栏覆盖 |
| 缩进参考线 | 主题已处理 | 无论开关都会隐藏竖线，不必额外调整 |
| 字体 | Inter + JetBrains Mono | 主题已内嵌 base64 字体，离线也能还原设计意图 |

## 三 适配判断

符合本库现状：以开发为主、代码块与表格多、深浅色都要用（白天浅色写、晚上深色读）。

两点注意：
- 主题对看板（Kanban）与第三方面板的 padding 做过修（v1.0.2），若以后装 VTable、Claudian 等带自绘面板的插件，留意面板内边距是否需要微调。
- 主题生态较新（GitHub star 数不高），升级前留意 release 说明，出问题可在 设置 → 外观 里一键切回默认主题，不影响笔记内容。

## 四 参考

- [GitHub 仓库](https://github.com/volodinroman/obsidian-tech001-theme)：更新日志、截图、字体说明
- [Obsidian Themes 收录页](https://www.obsidianthemes.com/theme/tech001)：在线预览明暗两套配色
