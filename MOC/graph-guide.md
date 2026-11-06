# 关系图谱范围与首次核对

本库提供全新graph配置文件，未复制旧库的项目分组。首次在Obsidian打开本库后，打开全局图谱，检查以下设置是否被应用；文件存在不代表已经做过渲染验收。

## 知识范围

包括Logs、Journal、Memo，以及领域、阅读/考试专题、书籍、考试和可选项目的MOC入口。排除附件、模板、Inbox、归档、根规范/管理页面以及agent_memory、.claudian。

过滤表达式：

```text
(path:"Logs/" OR path:"Journal/" OR path:"Memo/" OR path:"MOC/domains/" OR path:"MOC/collections/" OR path:"MOC/books/" OR path:"MOC/exams/" OR path:"MOC/projects/") -path:"agent_memory/" -path:".claudian/" -path:"Attachments/" -path:"_templates/" -path:"Inbox/" -path:"_archive/"
```

隐藏附件和标签节点、隐藏未解析节点、保留孤立节点并显示方向箭头。按development（蓝）、languages（紫）、life（绿）的路径分组；新增领域需按实际需要补色，不能依赖未完整填写的domain属性。

## 首次检查

- AI维护文件不应出现；不为其补关系。
- 初始主要是导航与工具教程，没有大量个人知识节点是正常状态。
- 链接只表示实际引用，不代表已完成、已实现或已验证。
- 隐藏断链不等于修复，遇到真实引用错误仍应检查文件。
- 不为了消除孤点互相补链。图谱、原生搜索和动态查询分别核对范围。

本轮只准备配置文件，未注册/切换当前库或验证图谱显示。若首次打开后设置不同，可按本页在图谱过滤及颜色设置中恢复并核对。

返回 [[MOC/index.md|首页]]，规则见 [[obsidian-vault-management-spec.md]]。
