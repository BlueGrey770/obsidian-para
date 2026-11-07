---
type: memo
date: 2026-11-02
aliases:
  - Obsidian插件安装与基础配置
---
# Obsidian 插件安装与基础配置教程

这套流程适用于以开发为主、兼顾阅读、生活随笔、考证和英语学习的个人知识库。先让笔记创建顺手，再增加自动汇总和记忆复习，不从工作库整包复制插件。

**推荐顺序：开启核心功能 → 配好 QuickAdd → 需要表格时选择 Bases 或 Dataview → 需要记忆卡片时安装 Spaced Repetition。**

分类结构见 [[obsidian-vault-management-spec.md|知识库管理规范]]。本库目录、四模板与基础app/graph配置文件已经提供；下文的插件安装、创建入口与运行验收仍需在打开本库后执行，没有预装第三方插件。

## 一 准备空库

在Obsidian中将本文件夹打开为独立库。以下根目录已经准备，可直接使用：

```text
Inbox/
Logs/
Journal/
Memo/
Attachments/
MOC/
_templates/
_archive/
```

领域与主题目录随实际内容建立，例如 `Memo/development/databases/`、`Journal/life/reflections/`、`Logs/languages/english/`。阅读和考证通过专题页聚合，不各复制一份知识。

安装前确认设置针对的是本库，不是当前工作库。不要整包复制工作库的 `.obsidian`：里面可能包含本机路径、账号、同步目标和项目专属配置。正式笔记最低只要求 type/date，不要求选择公司项目。

## 二 开启自带核心功能

打开 **设置 → 核心插件**，检查以下功能。它们由 Obsidian 自带，无需到第三方商店安装。[核心插件说明](https://help.obsidian.md/plugins)

| 核心功能       | 建议         | 用途                         |
| ---------- | ---------- | -------------------------- |
| 命令面板、搜索    | 开启         | 运行命令、查找笔记                  |
| 属性视图       | 开启         | 查看与管理属性                    |
| 反向链接、关系图谱  | 开启         | 查看真实引用与知识联系                |
| 大纲、书签      | 开启         | 浏览长文，固定首页及常用入口             |
| 文件恢复       | 开启         | 从本机笔记快照找回误改内容              |
| 模板         | 按需         | 手动插入普通模板，不负责执行 QuickAdd 语法 |
| Bases（数据库） | 做表格时开启     | 按文件与属性筛选、排序                |
| 日记         | 确实每天写日记时再开 | 不作为这套框架的前提                 |

若版本没有 Bases，可先用静态目录；升级或选择 Dataview 后再增加视图，不影响记笔记。

文件恢复不是完整备份，快照不会随同步服务跨设备同步，也不覆盖 PDF、图片等附件。本库仍应另做独立备份并试恢复。[文件恢复的范围与限制](https://help.obsidian.md/plugins/file-recovery)

## 三 安装第一个第三方插件 QuickAdd

1. 打开 **设置 → 第三方插件**。
2. 首次使用时，阅读提示后启用第三方插件；需要时关闭限制模式。
3. 点击 **浏览**，搜索英文名 `QuickAdd`。
4. 打开插件详情，选择 **安装**，再选择 **启用**。
5. 确认设置中可以进入 QuickAdd 页面。

安装和启用是两个步骤。第三方插件会运行代码，只安装可信来源；不为排查下载问题随意执行网上安装脚本。[官方安装流程与安全提示](https://help.obsidian.md/community-plugins)

### 准备模板副本

本库 `_templates` 下已有以下四个模板，安装QuickAdd后直接引用，不必重复创建；下面给出内容以便核对：

| 模板文件 | 对应内容 |
| --- | --- |
| `_templates/log-note.md` | 实践、学习和练习过程 |
| `_templates/journal-note.md` | 读书心得、生活随笔、思考复盘 |
| `_templates/memo-card.md` | 可复用知识、方法与清单 |
| `_templates/inbox-draft.md` | 暂未整理的草稿 |

Log 模板内容如下，使用源码模式编辑：

```markdown
---
type: log
date: {{DATE:YYYY-MM-DD}}
---
# {{VALUE:slug}}

{{CURSOR}}
```

本库 Journal 和 Memo 模板已使用对应的 `journal`、`memo`，其余结构相同。Draft 不要求属性，内容如下：

```markdown
# {{VALUE:slug}}

{{CURSOR}}
```

`{{DATE:YYYY-MM-DD}}` 填入日期，命名的 `{{VALUE:slug}}` 可在文件名与正文复用同一次输入，`{{CURSOR}}` 指定正文光标位置。[QuickAdd 格式语法](https://quickadd.obsidian.guide/docs/FormatSyntax/)

这些占位符必须通过 QuickAdd 运行；手工复制模板或用核心“模板”插入，不能代替 QuickAdd 解析。维护模板时保持源码模式，避免属性面板改写占位符。

## 四 配置快捷创建入口

在 **设置 → QuickAdd** 新建 **Template** 类型的 Choice，先配置 Log。这里是“用模板创建新文件”，不是 Capture 追加记录。

关键设置：填写名称与模板路径；在 File name 中设置命名格式；`New note location` 选择 `In a specific folder`；在 `Folder path` 填 `Logs` 后点击 **Add**；开启创建后打开。较旧界面可能将文件名项称为 File name format，需要先开启开关。[Template Choice 配置](https://quickadd.obsidian.guide/docs/Choices/TemplateChoice/)

![](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Pasted%20image%2020261102175449.png)
> 图注：Pasted image 20261102175449

四个入口按下表配置，文件名格式中不要加 `.md`：

| Choice  | 模板路径                         | 初始目录      | 文件名格式                                |
| ------- | ---------------------------- | --------- | ------------------------------------ |
| Log     | `_templates/log-note.md`     | `Logs`    | `{{DATE:YYYY-MM-DD}}-{{VALUE:slug}}` |
| Journal | `_templates/journal-note.md` | `Journal` | `{{VALUE:slug}}-{{DATE:YYYY-MM-DD}}` |
| Memo    | `_templates/memo-card.md`    | `Memo`    | `{{VALUE:slug}}`                     |
| Draft   | `_templates/inbox-draft.md`  | `Inbox`   | `draft-{{VALUE:slug}}`               |

按照下列方式，分别填入即可：
![](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Pasted%20image%2020261102175527.png)
> 图注：Pasted image 20261102175527

Folder path 填库内相对路径，例如 `Logs`，不填磁盘绝对路径。
初始目录不等于最终分类：先在体裁根目录创建，再在 Obsidian 内移入领域与主题。不假设模板会自动判断领域或按项目路由。

### 实际创建一篇测试笔记

1. `Ctrl+P` 打开命令面板，搜索 QuickAdd，运行主入口 `QuickAdd: Run`。
	![](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Pasted%20image%2020261102175844.png)
> 图注：Pasted image 20261102175844

2. 选择 `Log`，输入 `english-listening-test`，不要输入日期或扩展名。
	![](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Pasted%20image%2020261102180049.png)
> 图注：Pasted image 20261102180049

3. 检查文件名是否为 `当天日期-english-listening-test.md`，属性是否为 type=log、date=当天日期（可增加属性tag并自定义标签），正文没有残留占位符。
	![](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Pasted%20image%2020261102180258.png)
> 图注：Pasted image 20261102180258

4. 写一句测试内容，在 Obsidian 内移到 `Logs/languages/english/`。
5. 为 Journal、Memo、Draft 各测试一次，尤其检查 Journal 日期在文件名末尾，Memo 不带日期，Draft 不强制属性。
6. 测试正常后，再用 Choice 的闪电按钮将其注册为独立命令，并在快捷键设置中绑定按键。[QuickAdd 主入口与快捷键](https://quickadd.obsidian.guide/docs/)

slug 自己使用小写英文、数字和连字符，不依赖插件自动翻译或清理特殊字符。同名冲突时先判断应追加已有笔记还是创建独立内容，不直接覆盖。

## 五 如何添加和检查笔记属性

普通新建文件不会自动拥有模板字段。需要手工补充时，打开笔记按 `Ctrl+;`，或在命令面板运行 **Add file property（添加文件属性）**。type 使用文本类型，date 使用日期类型；也可在源码模式的文件最顶部填写 YAML。[属性添加与类型说明](https://help.obsidian.md/properties)

例如一篇生活随笔：

```markdown
---
type: journal
date: 2026-11-02
---
# 一次散步后的想法

这里写正文。
```

示例日期替换为自己的首次记录日期，不随普通修订更新。本库领域默认由路径表达，project/domain/tags/aliases 按需添加，不必填满。本库不要求 project；明确属于一个个人项目时再按需填写。

如果属性区域被隐藏，检查 **设置 → 编辑器 → 文档中的属性** 的显示方式。只想快速记录时，也可以先放 Inbox，之后再补正式字段。

## 六 需要自动目录时选择 Bases 或 Dataview

起步阶段先用 MOC 中的静态链接即可，不需要把所有插件装齐。

### 简单表格先试 Bases

Bases 是核心功能，可以按属性查看、编辑、筛选和排序笔记。适合做自己的阅读清单、知识卡片表格。[Bases 功能说明](https://help.obsidian.md/bases)

启用后，按 `Ctrl+P` 搜索并运行 **Bases: Create new base**；也可以在文件列表中右键目标文件夹，选择 **New base**。先加入所需列，并将筛选范围限制到自己的正式笔记目录，排除模板、附件和归档；试验时只编辑测试笔记。[创建数据库视图](https://help.obsidian.md/bases/create-base)

### 复杂汇总或沿用 DQL 时再装 Dataview

通过第三方插件浏览安装并启用 `Dataview`。它用于索引、筛选和展示笔记；不会自动填写缺失字段，也不会替你进行知识分类。[Dataview 说明](https://blacksmithgu.github.io/obsidian-dataview/)

例如需要跨三种体裁查看开发笔记时，在领域页放入：

```dataview
TABLE type AS "体裁", date AS "首次记录"
FROM "Logs/development" OR "Journal/development" OR "Memo/development"
SORT date DESC
```

目录来源会包含子目录，不在来源字符串末尾加斜杠。查询按路径聚合，符合本库的领域分类方式，不要求额外填 domain，也不照搬工作库的公司项目筛选。[Dataview 目录来源规则](https://blacksmithgu.github.io/obsidian-dataview/reference/sources/)

切换到阅读视图，检查已归入 development 子目录的测试笔记是否出现；尚未移出体裁根目录的文件不会进入这个领域视图，这是预期行为。示例需在启用 Dataview 的本库中实际验收。

初版先选一种目录方案，不要求同时维护 Bases 和 Dataview，也不因静态目录正常就宣称动态视图已经通过。

## 七 需要记忆卡片时再装 Spaced Repetition

准备在 Obsidian 内反复复习英语或考证知识时，再安装并启用 `Spaced Repetition`；只做阅读与随笔时可以跳过。[插件介绍与安装](https://stephenmwangi.com/obsidian-spaced-repetition/)

先用少量测试卡片：

1. 在插件设置中确认卡片标签，下面示例使用 `#flashcards`；若当前设置不同，应先统一。
2. 在一篇测试 Memo 正文加入以下内容。代码块只是展示格式，实际放入笔记时不要包裹代码围栏：

```text
#flashcards/english

apple 的常见中文意思是什么？::苹果
```

3. 通过命令面板搜索 Spaced Repetition，运行卡片复习入口，确认可以看到问题、显示答案并记录一次复习。

单行问答使用 `::`，标签用于确定牌组，`#flashcards` 可以匹配下级牌组标签；以自己安装版本的设置为准。[问答卡片格式](https://stephenmwangi.com/obsidian-spaced-repetition/flashcards/q-and-a-cards/)、[牌组设置](https://stephenmwangi.com/obsidian-spaced-repetition/flashcards/decks/)

知识仍保留在语言或开发等原领域，不需要复制一份到独立“记忆库”。先确认自己能持续复习，再批量制作卡片。

## 八 常见问题与排查

| 现象 | 优先检查 |
| --- | --- |
| 新笔记没有属性 | 是否用 QuickAdd Template Choice 创建；模板是否包含 type/date |
| 残留 VALUE 或 DATE 占位符 | 是否只复制了模板，或误用核心模板功能 |
| 模板占位符被改写 | 切回源码模式，核对原始模板；不要把模板文件当成已创建的正式笔记 |
| 每次落在体裁根目录 | 这是本教程的初始位置，需要自己归入领域/主题 |
| QuickAdd 找不到模板 | 路径应相对当前库，包含正确文件名；确认文件实际存在 |
| 找不到某个 Choice 的独立快捷命令 | 确认它已注册为命令；也可先从 QuickAdd 主入口运行 |
| Dataview 显示为代码或没有结果 | 确认插件启用、代码块语言为 dataview、切到阅读视图，并检查实际路径 |
| 记忆插件没有发现卡片 | 检查标签、分隔符与当前牌组设置；卡片不要放在代码块里 |

## 九 最小验收与日常使用

- [ ] 已分清核心功能与第三方插件，不把“已安装”等同“已启用”。
- [ ] QuickAdd 四个入口各创建一篇，文件名、日期、体裁与占位符输出正确。
- [ ] 文件移动到领域与主题后仍能打开；普通目录移动没有被误认为会自动修改属性。
- [ ] 如果使用动态目录，实际检查新增、移动和归档后的列表变化。
- [ ] 如果使用记忆插件，至少完成一次测试卡片复习。
- [ ] 有独立备份，且做过恢复测试；不只依赖本机文件恢复。

日常流程保持简单：**选体裁创建 → 写正文 → 在 Obsidian 内归类 → 补必要引用 → 按需复盘或复习。** 暂时不需要 Templater、大量美化插件或 AI 插件；以后确有需求再增加。
