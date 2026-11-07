---
type: memo
date: 2026-11-05
aliases:
  - Claudian 插件教程
  - Obsidian 接入 Codex
  - Obsidian AI Agent 侧边栏
---
# Claudian：把 AI Agent 接进 Obsidian

Claudian 把本地装好的 CLI 型 AI Agent 嵌进 Obsidian 侧边栏，**当前笔记库就是 Agent 的工作目录**——它能读写文件、搜索、执行命令、跑多步任务，不用切终端复制粘贴。

它不提供模型，只做桥接：能力来自本机 `claude` / `codex` 等 CLI 和你自己的订阅或 API。

本库状态：v2.3.35（插件 id `realclaudian`，作者 Yishen Tu）；库内 `.claudian`、`agent_memory` 由它产生，已按 [[obsidian-vault-management-spec|管理规范]] 排除在图谱与知识跟踪之外。

## 一 新版变化：网上老教程已经过时

| 项目 | 早期教程（多数文章写于旧版） | 当前版本（本库 v2.3.x，设置页 1.15） |
| --- | --- | --- |
| 安装渠道 | 未上架市场，**必须 BRAT 中转** | **已上架社区插件市场**，搜 Claudian 直接安装 |
| CLI 路径 | 手动 `where claude` 后填绝对路径 | **自动检测**，识别成功显示版本号与绿灯 |
| 支持的 Agent 后端 | Claude Code + Codex 两家 | 六家：Claude Code、Codex、DeepSeek Harness、Grok Build、OpenCode、Pi |
| 运行方式 | 无此选项 | 可切 **原生 Windows / WSL**（WSL 在指定发行版内启动 Linux CLI） |
| 模型管理 | 全量出现，无法筛选 | **可见模型管理**：只显示已选、可拖拽排序、首个为默认、可设别名 |
| 设置结构 | 单页 | 通用 / 提供商 / 技能 三栏 |

看到老教程要求"先装 BRAT、再手填路径"不必照做，那是旧流程；新版装完基本即用。

## 二 安装

**方式一 社区市场（推荐）**：设置 → 第三方插件 → 浏览 → 搜 `Claudian` → 安装 → 启用。插件页：`community.obsidian.md/plugins/realclaudian`。

**方式二 BRAT（装测试版或回滚旧版时用）**：BRAT → Add a beta plugin for testing → 填 `https://github.com/YishenTu/claudian`。本库最初即由此方式安装。

前置要求：Obsidian v1.13.0 以上、桌面端（Windows / macOS / Linux）；至少一个 Agent CLI 已安装并完成授权，且有可用的订阅或 API。

## 三 配置提供商

![](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/claudian-provider-settings.png)
> 图注：Claudian 设置：提供商页，Codex CLI 自动检测成功

进入 设置 → Claudian → **提供商**，各后端并列在顶部标签中。配置顺序：

1. 选中要用的后端（如 `Codex`）。
2. 看**自动检测**结果：显示版本号（如 `Codex CLI v0.166.0-alpha.5.1`）且有绿灯，说明 CLI 已被识别，**无需手填路径**。
3. 选**安装方式**：原生 Windows，或 WSL（在所选发行版内启动 Linux CLI）。若 CLI 装在 WSL 里，这里必须切到 WSL，否则检测不到。
4. 打开该提供商的总开关。
5. 重启 Obsidian，然后**新开一个会话**（旧会话不会刷新模型列表）。

自动检测失败时的手动兜底：终端执行 `where claude` / `where codex`，把结果填进 CLI 路径；先确保终端里 `claude`、`codex` 能正常对话。

## 四 模型可见性管理

仍在上面的提供商页下半部分：

- **可见模型 3 / 已发现 8**：只把常用模型放进可见列表，聊天里的选择器不会被塞满。
- **拖动调整顺序**，排在第一个的即可用模型的默认模型。
- **别名可选**：给模型起中文名（如"写作-长上下文""代码-快"）比记版本号好认。
- 至少选一个模型，该提供商才可用。

新发布的模型没出现时：升级对应 CLI → 重启 Obsidian → 新开会话。

## 五 日常用法

![](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/claudian-chat.png)
> 图注：Claudian 对话界面：提示词、工具调用与会话列表

| 能力 | 用法 |
| --- | --- |
| 引用上下文 | `@` 引用库内文件、文件夹，甚至其他会话 |
| 斜杠命令与技能 | `/` 或 `$` 触发用户级/库级提示词模板与 Skills（如 `/周报`、`/读书笔记`） |
| 行内编辑 | 选中文字按热键，直接在笔记里改，给出词级 diff 预览后决定接受 |
| 会话管理 | 单栏多标签，或双栏常驻会话管理器 |
| 侧边支线 | `/side` 或 `/btw` 开一条不污染主对话的临时会话 |
| 项目 | 多个会话共享目标、指令与记忆文件，会话之间可互相读取与通信 |
| MCP | 走各 CLI 原生 MCP 配置接外部工具 |

![](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/claudian-dual-pane.png)
> 图注：双栏模式：左侧笔记、右侧会话管理器与工具调用明细

双栏模式适合"边读笔记边让 Agent 动手"：右侧能看到每一步工具调用（Script、Bash、WebSearch、Thought），文件被修改时会出现带链接的 diff 提示。

![禅模式展开：输入框贴底，对话随时展开](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/claudian-zen-mode.png)

禅模式把侧栏收起，输入框缩成笔记底部的一行，写作时干扰最小，需要时一键展开完整对话。

## 六 排错

| 现象 | 排查 |
| --- | --- |
| 提供商显示未检测到 | CLI 未安装或不在 PATH；原生/WSL 方式选错；先在终端跑通 `claude`、`codex` |
| 模型列表为空 | 未勾选任何可见模型；提供商总开关未开；需新开会话 |
| 新模型不出现 | 升级 CLI → 重启 Obsidian → 新开会话（旧对话框不刷新，官方行为） |
| CLI 能用但 Claudian 报认证失败 | 见官方 `docs/troubleshooting.md` |
| 提示 Node 缺失 | 桌面端需 Node.js 环境，安装后重启 |

## 七 隐私与知识库边界

- **数据会发往所选提供商**：输入、附件、图片、工具调用输出会送到 Anthropic、OpenAI、xAI 或你配置的其他服务；插件无遥测，本地仅有一个 `127.0.0.1` 监听供会话工具使用。
- **整个库是可读上下文**：含密钥、隐私的笔记不要明文留在库内；`.obsidian` 里的 OSS 密钥尤其不能顺手贴进对话。
- AI 输出不自动成为知识：要沉淀的内容必须落到 `Logs/` 或 `Memo/`，会话记录不算资产。
- `.claudian`、`agent_memory` 是会话上下文，不进图谱、不进普通查询。
- AI 给的操作步骤仍需自己验证，规范里"不编造执行结果"对 AI 同样适用。

## 八 参考

- [官方仓库 README](https://github.com/YishenTu/claudian)：功能清单、系统要求、排错文档（本文四张界面图来源）
- [社区插件页面](https://community.obsidian.md/plugins/realclaudian)
- [知乎：Obsidian + Claude + Codex 一键联动](https://zhuanlan.zhihu.com/p/2044190911760949796)：旧版 BRAT 安装流程与模型不显示的排查思路
- [什么值得买：3 分钟让 Obsidian 长出第二大脑](https://post.smzdm.com/p/aqr7eznx/)：Claude 与 Codex 分工对比、交互场景与标签体系建议
