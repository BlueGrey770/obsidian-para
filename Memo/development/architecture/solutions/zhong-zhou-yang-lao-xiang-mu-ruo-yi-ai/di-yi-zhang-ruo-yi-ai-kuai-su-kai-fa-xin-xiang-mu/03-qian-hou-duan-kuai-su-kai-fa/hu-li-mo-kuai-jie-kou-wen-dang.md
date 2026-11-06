---
type: memo
date: 2026-11-06
aliases:
  - "护理模块-接口文档"
---

# 护理模块-接口文档

## 护理项目

### 分页条件查询

- 接口地址：`/nursing/project/list`
- 请求方式：`GET`
- 请求参数：

  <sheet sheet-id="35qjN7" token="NCOHsi0b4hY3dstM77ic8gsanyc"></sheet>
- 响应示例

  ```JSON
  {
      "total": 10,
      "rows": [
          {
              "createBy": "1",
              "createTime": "2024-05-08 10:16:00",
              "updateTime": "2024-05-09 11:34:35",
              "id": "87",
              "name": "助餐",
              "orderNo": "2",
              "unit": "餐",
              "price": 15.00,
              "image": "https://itheim.oss-cn-beijing.aliyuncs.com/60f89ec5-a53b-4a53-8b6a-499a32dcc421.png",
              "nursingRequirement": "无",
              "status": 1
          }
      ],
      "code": 200,
      "msg": "请求成功"
  }
  ```

### 新增

- 接口地址: `/nursing/project`
- 请求方式:`POST`
- 请求示例:

  ```JSON
  {
    "image": "https://itheim.oss-cn-beijing.aliyuncs.com/7cbe6478-2499-4162-89f1-e642893aa2ce.png",
    "name": "助浴",
    "nursingRequirement": "助浴...",
    "orderNo": 1,
    "price": 20,
    "status": "1",
    "unit": "元"
  }
  ```
- 响应示例:

  ```JSON
  {"msg":"操作成功","code":200}
  ```

### 根据id查询护理

- 接口地址: `/nursing/project/{id}`
- 请求方式:`GET`
- 请求参数:

  <sheet sheet-id="s1oviT" token="NCOHsi0b4hY3dstM77ic8gsanyc"></sheet>
- 响应示例:

  ```JSON
  {
      "code": 200,
      "msg": "操作成功",
      "data": {
          "id": "7",
          "createTime": "2023-09-26 15:12:43",
          "updateTime": "2023-09-26 15:12:43",
          "createBy": "1671403256519078153",
          "name": "洗头",
          "orderNo": 7,
          "unit": "次",
          "price": 20.00,
          "image": "https://yjy-slwl-oss.oss-cn-hangzhou.aliyuncs.com/a2a56275-51e6-43a0-8ad6-b0600bfb01c2.jpg",
          "nursingRequirement": "将患者带到洗发间进行洗发，并提供按摩和护理，保持头皮健康",
          "status": 1
      }
  }
  ```

### 编辑

- 接口地址: `/nursing/project`
- 请求方式:`PUT`
- 请求示例:

  ```JSON
  {
    "image": "https://itheim.oss-cn-beijing.aliyuncs.com/7cbe6478-2499-4162-89f1-e642893aa2ce.png",
    "name": "助浴",
    "nursingRequirement": "助浴...",
    "orderNo": 1,
    "price": 20,
    "status": "1",
    "unit": "元"，
    "id": 123
  }
  ```
- 响应示例:

  ```JSON
  {"msg":"操作成功","code":200}
  ```

### 删除

- 接口地址:`/nursing/project/{id}`
- 请求方式:`DELETE`
- 请求参数:

  <sheet sheet-id="YrT5P5" token="NCOHsi0b4hY3dstM77ic8gsanyc"></sheet>
- 响应示例:

  ```JSON
  {"msg":"操作成功","code":200}
  ```

### 查询所有护理项目

- 接口地址: `/nursing/project/all`
- 请求方式:`GET`
- 请求参数:无
- 响应示例:

  ```JSON
  {
      "msg": "操作成功",
      "code": 200,
      "data": [
          {
              "label": "助浴",
              "value": "118"
          },
          {
              "label": "助餐",
              "value": "87"
          },
          {
              "label": "整理床铺",
              "value": "86"
          }
      ]
  }
  ```



## **护理计划**

### 条件分页查询

**接口地址**:`/nursing/plan/list`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 数据类型 |
|-|-|-|
| name | 护理计划名称 | string |
| pageNum | 页码（默认为1） | integer(int32) |
| pageSize | 每页大小（默认为10） | integer(int32) |
| status | 护理计划状态 | integer(int32) |

**响应示例**:

```JSON
{
    "total": 2,
    "rows": [
        {
            "createBy": "1",
            "createTime": "2024-07-28 15:08:38",
            "id": "3",
            "sortNo": "1",
            "planName": "一级护理计划",
            "status": 0
        },
        {
            "createBy": "1",
            "createTime": "2024-07-31 11:30:16",
            "id": "5",
            "sortNo": "1",
            "planName": "二级护理计划",
            "status": 1
        }
    ],
    "code": 200,
    "msg": "查询成功"
}
```



### 新增

**接口地址**:`/nursing/plan`

**请求方式**:`POST`

**请求示例**:

```JavaScript
{
  "id": 0,
  "planName": "",//计划名称
  "remark": "",//备注
  "sortNo": 0,//排序号
  "status": 0//状态（0：禁用，1：启用）
  "projectPlans": [
    {
      "executeCycle": 0,//执行周期 0 天 1 周 2月
      "executeFrequency": 0,//执行频次
      "executeTime": "",//计划执行时间
      "planId": 0,//计划id
      "projectId": 0,//项目id
      "remark": ""//备注
    },
    {
      "executeCycle": 1,//执行周期 0 天 1 周 2月
      "executeFrequency": 1,//执行频次
      "executeTime": "",//计划执行时间
      "planId": 1,//计划id
      "projectId": 1,//项目id
      "remark": ""//备注
    }
  ]
}
```

**响应示例**:

```JavaScript
{"msg":"操作成功","code":200}
```



### 根据id查询

**接口地址**:`/nursing/plan/{id}`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 请求类型 | 数据类型 |
|-|-|-|-|
| id | 护理计划ID | path | integer(int64) |

**响应示例**:

```JSON
{
    "code": 200,
    "msg": "操作成功",
    "data": {
        "id": "3",
        "sortNo": 1,
        "planName": "3号护理计划",
        "status": 0,
        "projectPlans": [
            {
                "createBy": "1",
                "createTime": "2024-07-28 15:08:38",
                "id": "14",
                "planId": "3",
                "projectId": "87",
                "executeTime": "15:08:00",
                "executeCycle": "1",
                "executeFrequency": "1"
            },
            {
                "createBy": "1",
                "createTime": "2024-07-28 15:08:38",
                "id": "15",
                "planId": "3",
                "projectId": "86",
                "executeTime": "15:08:32",
                "executeCycle": "1",
                "executeFrequency": "1"
            }
        ],
    }
}
```

### 修改

**接口地址**:`/nursing/plan`

**请求方式**:`PUT`

**请求示例**:

```JavaScript
{
  "id": 0,
  "planName": "",//计划名称
  "remark": "",//备注
  "sortNo": 0,//排序号
  "status": 0//状态（0：禁用，1：启用）
  "projectPlans": [
    {
      "executeCycle": 0,//执行周期 0 天 1 周 2月
      "executeFrequency": 0,//执行频次
      "executeTime": "",//计划执行时间
      "id":0
      "planId": 0,//计划id
      "projectId": 0,//项目id
      "remark": ""//备注
    }
  ]
}
```

**响应示例**:

```JavaScript
{"msg":"操作成功","code":200}
```

### 删除

**接口地址**:`/nursing/plan/{id}`

**请求方式**:`DELETE`

**请求参数**:

| 参数名称 | 参数说明 | 请求类型 | 数据类型 |
|-|-|-|-|
| id | 护理计划ID | path | integer(int64) |

**响应示例**:

```JavaScript
{"msg":"操作成功","code":200}
```



### 查询所有护理计划

**接口地址**:`/nursing/plan/all`

**请求方式**:`GET`

**请求参数**:无

**响应示例**:

```JSON
{
    "msg": "操作成功",
    "code": 200,
    "data": [
        {
            "createBy": "1",
            "createTime": "2024-07-28 15:08:38",
            "id": "3",
            "sortNo": "1",
            "planName": "3号护理计划",
            "status": 0
        },
        {
            "createBy": "1",
            "createTime": "2024-07-31 11:30:16",
            "id": "5",
            "sortNo": "1",
            "planName": "2号护理计划",
            "status": 1
        }
    ]
}
```



## 护理等级

### 条件分页查询

**接口地址**:`/nursing/level/list`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 数据类型 |
|-|-|-|
| pageNum | 页码 | integer(int32) |
| pageSize | 每页大小 | integer(int32) |
| name | 护理等级名称 | string |
| status | 护理等级状态 | integer(int32) |

**响应示例**:

```JSON
{
    "total": 12,
    "rows": [
        {
            "createBy": "1671403256519078153",
            "createTime": "2023-09-26 15:51:08",
            "updateTime": "2023-09-26 15:51:08",
            "id": "4",
            "name": "三级护理等级",
            "lplanId": "4",
            "fee": 1000.00,
            "status": 1,
            "description": "三级护理等级适用于能够基本自理的老年人，需要一些简单的日常护理，该等级提供了基础的护理项目",
            "planName": "3号护理计划"
        },
        {
            "createBy": "1671403256519078153",
            "createTime": "2023-09-26 15:48:18",
            "updateBy": "1671403256519078153",
            "updateTime": "2023-09-26 15:51:13",
            "id": "3",
            "name": "二级护理等级",
            "lplanId": "3",
            "fee": 1500.00,
            "status": 1,
            "description": "二级护理等级适用于能够自理但需要一定辅助护理的老年人，该等级提供了中等程度的照护项目",
            "planName": "2号护理计划"
        }
    ],
    "code": 200,
    "msg": "查询成功"
}
```



### 新增

**接口地址**:`/nursing/level`

**请求方式**:`POST`

**请求示例**:

```JSON
{
  "description": "无",
  "fee": 2000,
  "lplanId": "3",
  "name": "六级护理等级",
  "status": 1
}
```

**响应示例**:

```JSON
{"msg":"操作成功","code":200}
```



### 根据id查询

**接口地址**:`/nursing/level/{id}`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 请求类型 | 数据类型 |
|-|-|-|-|
| id | 护理等级ID | path | integer(int64) |

**响应示例**:

```JavaScript
{
    "msg": "操作成功",
    "code": 200,
    "data": {
        "createBy": "1",
        "createTime": "2024-08-19 16:38:25",
        "updateBy": null,
        "updateTime": "2024-08-19 16:38:25",
        "remark": null,
        "creator": null,
        "id": "73",
        "name": "六级护理等级",
        "lplanId": "3",
        "fee": 2000.00,
        "status": 1,
        "description": "无"
    }
}
```

### 修改

**接口地址**:`/nursing/level`

**请求方式**:`PUT`

**请求示例**:

```JavaScript
{
  "description": "无",
  "fee": 2000,
  "lplanId": "3",
  "name": "六级护理等级",
  "status": 1,
  "id":123
}
```

**响应示例**:

```JavaScript
{"msg":"操作成功","code":200}
```

### 删除

**接口地址**:`/nursing/level/{id}`

**请求方式**:`DELETE`

**请求参数**:

| 参数名称 | 参数说明 | 请求类型 | 数据类型 |
|-|-|-|-|
| id | 护理等级ID | path | integer(int64) |

**响应示例**:

```JavaScript
{"msg":"操作成功","code":200}
```

### 查询所有护理等级

**接口地址**:  `/nursing/level/listAll`

**请求方式**:`GET`

**请求参数**:无

**响应示例**:

```JSON
{
    "msg": "操作成功",
    "code": 200,
    "data": [
        {
            "createBy": "1671403256519078153",
            "createTime": "2023-09-26 15:48:18",
            "updateBy": "1671403256519078153",
            "updateTime": "2023-09-26 15:51:13",
            "id": "3",
            "name": "二级护理等级",
            "lplanId": "3",
            "fee": 1500.00,
            "status": 1,
            "description": "二级护理等级适用于能够自理但需要一定辅助护理的老年人，该等级提供了中等程度的照护项目"
        },
        {
            "createBy": "1671403256519078153",
            "createTime": "2023-09-26 15:51:08",
            "updateTime": "2023-09-26 15:51:08",
            "id": "4",
            "name": "三级护理等级",
            "lplanId": "4",
            "fee": 1000.00,
            "status": 1,
            "description": "三级护理等级适用于能够基本自理的老年人，需要一些简单的日常护理，该等级提供了基础的护理项目"
        }
    ]
}
```

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第一章-若依+AI快速开发新项目 / 03-前后端快速开发 / 护理模块-接口文档 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/KVgNw1Usdix0rIkUcR2cc4YAnsg)
