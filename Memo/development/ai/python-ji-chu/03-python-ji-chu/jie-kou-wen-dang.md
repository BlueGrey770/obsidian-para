---
type: memo
date: 2026-11-06
aliases:
  - "接口文档"
---

# 接口文档

### 查询所有

- 请求方式：`GET`
- 请求路径: `/posts/`
- 请求参数：无
- 响应示例

```JSON
[
    {
        "id": 14,
        "title": "测试标题",
        "content": "内容内容",
        "author": "李静",
        "created_at": "2025-10-19T12:09:21",
        "updated_at": "2025-10-19T12:09:21"
    },
    {
        "id": 1,
        "title": "Python编程入门指南",
        "content": "Python是一种高级编程语言，以其简洁的语法和强大的功能而闻名。本文将介绍Python的基础知识，包括变量、数据类型、控制流和函数等。\n\n## Python的特点\n- 简单易学\n- 开源免费\n- 跨平台兼容\n- 丰富的库支持\n\n无论你是编程新手还是有经验的开发者，Python都是一个不错的选择。",
        "author": "张明",
        "created_at": "2024-01-15T10:00:00",
        "updated_at": "2025-10-18T23:40:43"
    }
]
```



### 根据id查询

- 请求方式：`GET`
- 请求路径: `/posts/{post_id}`
- 请求参数：路径参数，博客ID
- 响应示例

```JSON
{
    "id": 16,
    "title": "Python程序设计",
    "content": "Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计",
    "author": "黑马",
    "created_at": "2025-10-19T15:13:04",
    "updated_at": "2025-10-19T15:13:04"
}
```

### 新增

- 请求方式：`POST`
- 请求路径: `/posts/`
- 请求参数：

```JSON
{
  "author": "黑马",
  "content": "Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计",
  "title": "Python程序设计"
}
```

- 响应示例

```JSON
{"status":200,"msg":"新增成功"}
```



### 更新

- 请求方式：`PUT`
- 请求路径: `/posts/{id}`
- 请求参数：

两个参数：

第一：路径参数，博客ID

第二：请求体参数，如下

```JSON
{
  "author": "黑马",
  "content": "Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计Python程序设计",
  "title": "Python程序设计"
}
```

- 响应示例

```JSON
{"status":200,"msg":"修改成功"}
```

### 删除

- 请求方式：`DELETE`
- 请求路径: `/posts/{post_id}`
- 请求参数：路径参数，博客ID
- 响应示例

```JSON
{"status":200,"msg":"删除成功"}
```

---
> 来源：飞书技术知识库（脱敏版） · 06 AI 应用开发 / Python 基础 / 03-python基础 / 接口文档 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/TXsnwfjShiljnMkHTnpcWtthnbc)
