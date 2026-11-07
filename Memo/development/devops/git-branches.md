---
type: memo
date: 2026-11-07
aliases:
  - 分支说明
  - Branch 约定
tags:
  - devops
  - git
---

# Git 分支与命名基线

本仓库的 git 分支用途与当前状态。日常提交命令与故障处理见 [[Memo/development/devops/obsidian-git-version-control.md|Git 版本管理教程]]。

## 分支一览

| 分支 | 用途 | 状态 |
| --- | --- | --- |
| `master` | 主分支，日常笔记在此提交 | 活跃 |
| `pinyin-filenames` | 文件名采用**拼音 slug**（如 `mian-xiang-dui-xiang.md`）的整库快照，作为命名方案的可回溯基线 | 归档基线，只读 |

两个分支在建立时指向同一个提交，内容完全一致；后续命名方案若调整，从 `master` 继续，`pinyin-filenames` 保留现状不动。

## 常用操作

切回日常分支：

```bash
git checkout master
```

查看当前分支与差异：

```bash
git branch -vv
git rev-list --left-right --count master...pinyin-filenames
```

对比两个分支的某个文件：

```bash
git diff master pinyin-filenames -- "Memo/development/"
```

在旧命名基线上临时查资料（不影响 `master`）：

```bash
git checkout pinyin-filenames
# 看完切回
git checkout master
```

## 约定

- 归档分支不重复开发，需要改动一律回 `master`。
- 分支对应的快照提交打有标签 `pinyin-v1`，标签指向的提交不可变，比分支更可靠。
- 远端仓库绑定后，两条分支与标签一并推送：`git push -u origin master pinyin-filenames --tags`。
