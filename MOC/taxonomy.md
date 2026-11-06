# 分类登记表

本页登记主领域与主题边界，不作为全量笔记清单。领域以路径为准，中文名用于导航显示，稳定ID用于目录。

## 已启用领域

| 中文名 | ID | 包含 | 不包含或边界反例 | 常用入口 |
| --- | --- | --- | --- | --- |
| 开发与技术 | development | 编程、工程实践、数据库、AI与专业软件 | 英文技术文章若重点是表达练习，归languages | [[MOC/domains/development.md|开发]] |
| 语言学习 | languages | 英语等语言的理解、表达与练习 | 技术文章产生的独立技术结论，归development | [[MOC/domains/languages.md|语言]] |
| 生活与随笔 | life | 个人体验、观察、生活事务和经验 | 文学/历史知识分析不硬塞life，可启用humanities | [[MOC/domains/life.md|生活]] |

## 主题建议

- development：programming、backend、frontend、databases、ai、devops、architecture、cad-cam；需记录网络知识时可增networking。
- languages：先用english；以后其他语言加主题，单语言内容多时再细分。
- life：reflections、daily、household、travel、health，按需建立。

这里只是主题建议，不表示所有主题目录已建立，不给每个体裁预建同样子主题。

### development 已建立主题（2026-11-06 技术知识库迁入后）

| 主题路径 | 内容 | 索引 |
| --- | --- | --- |
| programming/python | Python 语言（本地课程材料提炼） | [[MOC/topics/python.md]] |
| programming/java | Java 语言核心 | [[MOC/topics/java-core.md]] |
| programming/interview | 面试题与求职方法论 | [[MOC/topics/interview.md]] |
| databases | MySQL、JDBC、MyBatis | [[MOC/topics/databases.md]] |
| backend/web | Spring 系与全栈实战 | [[MOC/topics/web.md]] |
| frontend | HTML/CSS/JS、Vue | [[MOC/topics/frontend.md]] |
| architecture/microservices | 微服务与中间件 | [[MOC/topics/microservices.md]] |
| architecture/solutions | 业务组件方案集 | [[MOC/topics/solutions.md]] |
| ai | AI 应用开发 | [[MOC/topics/ai.md]] |
| devops | 工程化部署与本库工具教程 | [[MOC/topics/devops.md]] |

## 可选领域

humanities（文学、历史等）、health（健康）、creativity（创作）、finance（个人财务）均未预建。出现清晰用途时登记边界并启用；也可以从现有领域的适当主题起步。

## 专题不是主领域

[[MOC/collections/reading.md|阅读]]与[[MOC/collections/certifications.md|考证]]是聚合视角。某本书、某次考试、某个人项目的页分别放MOC/books、MOC/exams、MOC/projects；不重复保存被引用的正式笔记。

## 新增流程

1. 查同义或近义目录，优先复用已有主题。
2. 新领域在本表写中文名、ID、包含范围与边界反例。
3. 新建对应领域MOC，首页补入口；出现内容时建实际需要的体裁目录。
4. 如迁移旧笔记，通过Obsidian移动，核对链接、可选属性、查询和图谱分组。

不要把年份、学习状态或来源书名当全库主领域。完整规则见 [[obsidian-vault-management-spec.md]]。
