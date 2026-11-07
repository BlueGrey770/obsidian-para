---
type: memo
date: 2026-11-07
aliases:
  - Git 版本管理教程
  - 笔记库备份到私有仓库
---
# 笔记库的 Git 版本管理

笔记库本质是一堆文本文件，用 Git 管它有三样好处：**每次改动可回退**（改错标题、误删整篇都能找回）、**有据可查的变更历史**（哪天为什么改了这篇）、**异地备份**（推到私有远程仓库，换机器 clone 即得）。

它和网盘同步不是一回事：同步只会把错误同步到所有设备，Git 留下的是可以退回的每一步。

## 一 本库现状

| 项目 | 状态 |
| --- | --- |
| 仓库 | 已初始化（2026-11-06），分支 `master` + 基线分支 `pinyin-filenames`（标签 `pinyin-v1`） |
| `.gitignore` | 已排除 `.obsidian/` 整个目录、`.claudian/sessions/`、系统垃圾文件 |
| `.gitattributes` | 文本强制 LF、附件按二进制处理 |
| 远程仓库 | 已绑定 `https://github.com/BlueGrey770/obsidian-para.git`（2026-11-07 首推成功，Public） |
| 网络 | 本仓库级配置了代理 `http.proxy = http://127.0.0.1:7890`，推送/拉取需本机代理软件（Clash）运行中 |
| 体积 | 附件约 294 MB、文件约 3000 个，首推已完成；日常单次推送很小 |

## 二 三条不变量

1. **远程仓库可见性与内容红线**。仓库现为 Public（2026-11-07 权衡后自选公开）：`.obsidian/`（含明文 OSS 密钥）已排除，但正文与附件全网可见。红线——任何密钥、证件、公司资料、未脱敏客户信息不进库；想收紧时在 GitHub 的 Settings → Danger Zone 一键改 Private，无需重推。
2. **`.obsidian/` 永不入库**。配置换了机器按 README 的《新机器恢复清单》重建，不要为了“拉下来就能用”把配置塞进库，那等于把密钥一起推上去。
3. **批量改动前先留一个干净提交**。脚本批量改标题、批量移动目录之前先 `git status` 干净并提交一次，出事一条 `git checkout -- .` 就能整体回退——2026-11-07 归一 79 篇标题时就是靠这一步救回了误改的代码块。

## 三 接上远程仓库（首次）

### 1. 先在托管平台建一个空仓库

GitHub、Gitee、自建 Gitea 都行，要点只有两条：

- 选 **Private / 私有**。
- **不要**勾选“用 README 初始化”“添加 .gitignore / License”，建一个完全空的仓库。否则远程已有一个提交，本地推送会被判为非快进（`rejected`），多绕一步。

### 2. 绑定远程并推送

在库根目录（`E:\notes\PARA`）执行：

```bash
git remote add origin <私有仓库地址>
git push -u origin master
```

`-u` 的作用是把本地 `master` 与 `origin/master` 建立跟踪关系，之后直接 `git push`、`git pull` 就行，不用每次写远程名和分支名。

地址两种写法，按平台给：

```bash
# HTTPS（推荐新手，配合访问令牌）
git remote add origin https://github.com/<用户名>/<仓库名>.git
# SSH（已配好公钥时更省事）
git remote add origin git@github.com:<用户名>/<仓库名>.git
```

### 3. 认证：用令牌，不用账号密码

GitHub、Gitee 都已不支持直接用登录密码推送：

- **GitHub**：Settings → Developer settings → Personal access tokens → 生成经典令牌，勾 `repo` 权限，推送时把令牌当密码填。
- **Gitee**：设置 → 私人令牌，同样勾仓库读写。
- 想免输：`git config --global credential.helper manager`（Windows 会弹窗记住），或改用 SSH 密钥。

### 4. 验证

```bash
git remote -v          # 应显示 origin 的 fetch/push 两行
git branch -vv         # master 后面应出现 [origin/master]
git log --oneline -3   # 本地提交在
```

再去网页刷新仓库，能看到笔记目录即成功。

> 首次推送 500 MB 左右，网速慢时进度条会长时间停在 `Writing objects`，属正常。中途断了重跑同一条 `git push -u origin master` 即可续传。

## 四 日常提交

命令行三件套，改完一批就提交一次：

```bash
git status                 # 看改了什么
git add -A                 # 全部暂存（或 git add 具体文件）
git commit -m "说明这次改了什么"
git push
```

提交信息的写法：一句话说清**改了什么、为什么**，例如 `docs: 新增导图插件教程`、`refactor: 归一 79 篇标题层级`。不要写 “update”“修改” 这种等于没说的信息——半年后翻历史时，只有这类信息能帮你判断该不该回退。

回退常用三条：

```bash
git checkout -- <文件>     # 丢弃某个文件的未提交改动
git restore --source=HEAD~1 <文件>   # 取回上一次提交的版本
git log --oneline -- <文件>          # 看某篇笔记的改动历史
```

## 五 obsidian-git 插件：把提交自动化

插件已安装（`.obsidian` 不入库，换机需重装）。打开 设置 → Obsidian Git：

| 设置项 | 建议值 | 说明 |
| --- | --- | --- |
| Vault backup interval (minutes) | `10`~`15` | 定时自动 commit + push，日常主力 |
| Auto backup after file change | 开（可选） | 停笔几分钟后自动提交，比定时更细 |
| Auto pull interval (minutes) | 关或不设 | 单机使用不必自动拉取，避免半途冲突 |
| Commit message on auto backup | `vault backup: {{date}}` | 自动提交标记清楚，便于和手工提交区分 |
| Disable push | 关 | 要它顺手推到远程才有备份意义 |

装上后左侧会出现 Source control 面板：可看改动列表、点按钮 commit / push / pull，也可以只对某几篇提交。命令行与插件不冲突，谁方便用谁。

**冲突处理**：如果远端有别的设备推过，`push` 会提示被拒。正确顺序是先拉再推：

```bash
git pull --rebase origin master
git push
```

笔记是纯文本，冲突标记 `<<<<<<<` 直接打开文件手工合并即可；**不要用 `git push -f`**，那会覆盖掉另一台设备上的内容。

## 六 两个配置文件的分工

- **`.gitignore`**：决定什么不进库。当前排除 `.obsidian/`（含 OSS 密钥、插件二进制、workspace 状态）、`.claudian/sessions/`（AI 会话日志）、系统垃圾。附件默认提交，将来放大视频可取消注释改为依赖 OSS 外链。
- **`.gitattributes`**：决定进库后怎么存。文本统一 `eol=lf`，图片/PDF 标 `binary`。没有它，Windows 上某些工具写回 CRLF 会让整篇笔记显示成“全文改动”，历史彻底失去可读性。

## 七 分支与命名基线

库的文件名方案可能变化（拼音 slug → 其他写法），历史不能跟着一起丢。用分支把旧方案整库冻结下来：

```bash
git checkout -b pinyin-filenames          # 在当前提交上开分支
git tag -a pinyin-v1 -m "拼音文件名整库快照"  # 再打一个不可变标签
git checkout master                        # 切回主分支继续日常提交
```

三条用法：

- **主分支只留一条**。日常写作、批量脚本都在 `master` 上，避免两套内容各写一半后无法合并（Obsidian 不知道分支，切分支时整库文件会变，正在编辑的笔记可能被覆盖）。
- **归档分支只读**。`pinyin-filenames` 用来回溯「当时叫什么名字」，需要改动一律回 master 做。
- **标签比分支可靠**。分支指针会被后来的提交带跑，标签一旦打上就锁定那个提交，几年后 `git show pinyin-v1` 仍能看到当年的目录树。

切换分支前先提交干净：

```bash
git status --short     # 应为空
git checkout pinyin-filenames
git checkout master
```

本库的分支现状与命令见 [[Memo/development/devops/git-branches.md|Git 分支说明]]。

## 八 换机器怎么恢复

```bash
git clone <私有仓库地址> PARA
```

拿到的是笔记、附件、模板、规范；`.obsidian/` 不在其中，按 [[README|README]] 的《新机器恢复清单》重装插件、填 OSS 密钥、设图谱白名单，约 15~20 分钟。然后把 obsidian-git 的 `Vault backup interval` 重新设上。

## 九 常见问题

| 现象 | 原因与处理 |
| --- | --- |
| `! [rejected] ... (fetch first)` | 远程已有本地没有的提交。`git pull --rebase origin master` 后再 push |
| 提示输入用户名密码，输密码报错 | 平台已禁用密码认证，改用访问令牌（第三节） |
| `CONNECT tunnel failed, response 502` | 推送走了错误代理。本库已在仓库级配置 `http.proxy=http://127.0.0.1:7890`，确认 Clash 在运行即可；换端口时改 `git config http.proxy` |
| `Failed to connect to github.com:443` | 直连超时（未走代理）。确认 Clash 开启并占用 7890 端口，或临时 `git -c http.proxy=http://127.0.0.1:7890 push` |
| 整篇笔记显示全部删除又全部新增 | 行尾被写成 CRLF。确认 `git config core.autocrlf` 为 `false`，`.gitattributes` 在位，再 `git add --renormalize .` |
| 误把 `.obsidian` 提交了 | 先**立刻吊销 OSS 密钥**，再从索引移除并提交：`git rm -r --cached .obsidian`；历史里仍留有旧值，必要时用 `git filter-repo` 清理 |
| 单文件超过 100 MB 推不上去 | GitHub 硬限制。改用 Git LFS，或把大文件移出库、正文用 OSS 外链 |
| 两台设备同时改同一篇 | 后推的一方先 pull，手工合并冲突标记，别强制推送 |

## 十 命令速查

| 目的 | 命令 |
| --- | --- |
| 看改动 | `git status` / `git diff` |
| 提交全部 | `git add -A && git commit -m "说明"` |
| 推送 | `git push` |
| 拉取 | `git pull --rebase origin master` |
| 绑定远程 | `git remote add origin <地址>` |
| 首次推送 | `git push -u origin master` |
| 看某篇历史 | `git log --oneline -- <文件>` |
| 丢弃未提交改动 | `git checkout -- <文件>` |
| 看远程 | `git remote -v` |
