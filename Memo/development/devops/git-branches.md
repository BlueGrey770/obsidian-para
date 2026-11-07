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

| 分支 | 命名方案 | 用途 | 状态 |
| --- | --- | --- | --- |
| `master` | 英文 slug（`oop/05-oop-programming.md`） | 主分支，日常笔记在此提交 | 活跃 |
| `pinyin-filenames` | 拼音 slug（`mian-xiang-dui-xiang/05-mian-xiang-dui-xiang-bian-cheng.md`） | 改名前的整库快照，命名方案的可回溯基线 | 归档基线，只读 |

两个分支在建立时指向同一个提交；2026-11-07 master 完成全库英文化（68 个目录、164 个文件名），旧拼音名已写入各笔记的 `aliases`，因此日常检索不受影响。需要看改名前的目录树时切到 `pinyin-filenames`，`master` 不受影响。

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
