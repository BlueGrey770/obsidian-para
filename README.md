# 个人知识库

本库以开发为主，兼容阅读、生活随笔、考证和英语学习。规则、目录、导航与模板已准备，不含工作笔记、公司附件或旧AI会话历史。

## 从这里开始

1. 在 Obsidian 中选择“打开文件夹作为仓库”，选择这个文件夹，而不是把它作为附件导入其他库。
2. 打开 [[MOC/index.md|个人知识库首页]]，了解五个常用入口。
3. 阅读 [[obsidian-vault-management-spec.md|管理规范]] 与 [[MOC/taxonomy.md|分类登记表]]。
4. 按 [[Memo/development/devops/obsidian-plugin-setup-guide.md|插件安装与配置教程]] 安装并配置 QuickAdd；不想装插件也可手工新建笔记、填写属性。

## 当前已提供什么

- Inbox、Logs、Journal、Memo、Attachments、MOC、模板与归档目录。
- 开发、语言、生活三个基础领域，英语目录以及阅读、考证专题入口。
- Image Upload Toolkit 已安装并配置阿里云 OSS 图床化（公共读 bucket + RAM 密钥），教程与排查见 [[Memo/development/devops/obsidian-image-host-toolkit.md|图床化教程]]。
- 四个 QuickAdd 模板，正式笔记只要求 type/date，不必选择项目。
- `.obsidian/app.json`：默认新建 Inbox、附件接收 Attachments、相对链接及自动更新内部链接（2026-11-05 已核对实际生效）。
- 全新 `.obsidian/graph.json`：知识范围白名单，排除AI维护资料等，按初始领域路径分色，详见 [[MOC/graph-guide.md|图谱说明]]。

已安装插件：image-upload-toolkit、quickadd、obsidian-git、BRAT、realclaudian。QuickAdd 四个创建入口、同步账号与独立备份任务尚未配置，按教程在本库单独完成。模板中的占位符尚未执行是正常状态，不要把模板当成成品笔记。

## 第一次记录

只写一篇即可，例如生活随笔放 `Journal/life/`，英语练习放 `Logs/languages/english/`，技术知识放 `Memo/development/` 的适当主题。步骤、心得和知识只有确实各自独立时才拆分。

具体主题随内容增加，不会预建所有分类。书籍与考试目录当前没有虚构条目；首次使用时按对应专题页说明添加。

## AI维护与安全

`agent_memory` 是本库新初始化的AI上下文，不是知识内容，默认排除图谱，不建立知识关联。云同步、公司资料和个人敏感原件未迁入，不要为了移植便利而整包覆盖配置。`.obsidian` 插件配置目录含明文 OSS 密钥，同步或推送代码仓库时必须确认其为私有仓库。文件恢复不是独立备份，正式使用后应另做可恢复的备份。
