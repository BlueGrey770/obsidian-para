---
type: memo
date: 2026-11-06
aliases:
  - "jie-kou-wen-dang"
  - "接口文档"
---

# 接口文档

## 小程序端

### 绑定家人

**接口地址**:`/member/user/add`

**请求方式**:`POST`

**请求示例**:

```JavaScript
{
  "idCard": "",    // 身份证号
  "name": "",      // 老人姓名
  "remark": ""     // 称呼
}
```

**响应示例**:

```JavaScript
{"msg":"操作成功","code":200}
```

### 查询家人列表

**接口地址**:`/member/user/my`

**请求方式**:`GET`

**请求参数**: 无

**响应示例**:

```JSON
{
    "code": 200,
    "msg": "操作成功",
    "data": [
        {
            "id": 7,
            "familyMemberId": 5,
            "elderId": 1,
            "elderName": "刘备"
        },
        {
            "id": 8,
            "familyMemberId": 5,
            "elderId": 2,
            "elderName": "关羽"
        }
    ]
}
```

### 查看已绑定家人列表（分页查询）

**接口地址**:`/member/user/list-by-page`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型 |
|-|-|-|-|-|
| pageNum | pageNum | query | false | integer(int32) |
| pageSize | pageSize | query | false | integer(int32) |

**响应示例**:

```JavaScript
{
  "code": 200,
  "msg": "操作成功",
  "data": [
    {
      "mid": "7",
      "mremark": "皇叔",
      "elderId": "1",
      "name": "刘备",
      "image": "https://itheim.oss-cn-beijing.aliyuncs.com/20279fd4-3d4f-4e00-8a52-ac33d6c37a55.png",
      "bedNumber": "101-1",
      "typeName": "豪华单人间",
      "iotId": "847XFy3JE4CcQk6SpHMHj0rk00",
      "deviceName": "watch01",
      "productKey": "j0rkvUZBtRu"
    },
    {
      "mid": "8",
      "mremark": "二哥",
      "elderId": "2",
      "name": "关羽",
      "image": "https://itheim.oss-cn-beijing.aliyuncs.com/9368029c-3071-4713-9ac9-9d6842048bed.png",
      "bedNumber": "101-2",
      "typeName": "豪华单人间",
      "iotId": null,
      "deviceName": null,
      "productKey": null
    }
  ]
}
```



### **查询健康数据**

**接口地址**:`/member/user/QueryDevicePropertyStatus`

**请求方式**:`POST`

**请求示例**:

```JSON
 {
   "deviceName": "",
   "productKey": ""
 }
```

**响应示例**:

```JSON
 {
    "msg": "操作成功",
    "code": 200,
    "data": {
        "list": {
            "propertyStatusInfo": [
                {
                    "dataType": "double",
                    "identifier": "BodyTemp",
                    "name": "体温",
                    "time": "1731980060981",
                    "unit": null,
                    "value": "39.0"
                },
                {
                    "dataType": "double",
                    "identifier": "xueyang",
                    "name": "血氧",
                    "time": "1731980060981",
                    "unit": null,
                    "value": "94.0"
                },
                {
                    "dataType": "int",
                    "identifier": "HeartRate",
                    "name": "心率",
                    "time": "1731980060981",
                    "unit": "",
                    "value": "107"
                },
                {
                    "dataType": "double",
                    "identifier": "BatteryPercentage",
                    "name": "用电量",
                    "time": "1731980060981",
                    "unit": null,
                    "value": "16.0"
                }
            ]
        }
    }
}
```

###  **按天统计查询指标数据**

**接口地址**:`/member/user/queryDeviceDataListByDay`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型 |
|-|-|-|-|-|
| **iotId** | 设备id（如：EUMY0DdzDihH2LNJkdHok1u770） | query | true | String |
| **functionId** | 物模型id（如：HeartRate） | query | true | String |
| **startTime** | 查询开始时间毫秒值（如：1729353600000） | query | true | Long |
| **endTime** | 查询结束时间毫秒值（如：1729439999999） | query | true | Long |

**响应示例**:

```JSON
{
  "code": 200,
  "msg": "操作成功",
  "data": [
    {
      "dateTime": "00:00",
      "dataValue": 73.0
    },
    {
      "dateTime": "01:00",
      "dataValue": 73.0
    },
    {
      "dateTime": "02:00",
      "dataValue": 84.0
    },
    {
      "dateTime": "03:00",
      "dataValue": 89.0
    },
    {
      "dateTime": "04:00",
      "dataValue": 88.0
    },
    {
      "dateTime": "05:00",
      "dataValue": 81.0
    },
    {
      "dateTime": "06:00",
      "dataValue": 82.0
    },
    {
      "dateTime": "07:00",
      "dataValue": 86.0
    },
    {
      "dateTime": "08:00",
      "dataValue": 73.0
    },
    {
      "dateTime": "09:00",
      "dataValue": 77.0
    },
    {
      "dateTime": "10:00",
      "dataValue": 88.0
    },
    {
      "dateTime": "11:00",
      "dataValue": 78.0
    },
    {
      "dateTime": "12:00",
      "dataValue": 74.0
    },
    {
      "dateTime": "13:00",
      "dataValue": 77.0
    },
    {
      "dateTime": "14:00",
      "dataValue": 85.0
    },
    {
      "dateTime": "15:00",
      "dataValue": 85.0
    },
    {
      "dateTime": "16:00",
      "dataValue": 0.0
    },
    {
      "dateTime": "17:00",
      "dataValue": 86.0
    },
    {
      "dateTime": "18:00",
      "dataValue": 0.0
    },
    {
      "dateTime": "19:00",
      "dataValue": 78.0
    },
    {
      "dateTime": "20:00",
      "dataValue": 80.0
    },
    {
      "dateTime": "21:00",
      "dataValue": 71.0
    },
    {
      "dateTime": "22:00",
      "dataValue": 0.0
    },
    {
      "dateTime": "23:00",
      "dataValue": 86.0
    }
  ]
}
```

<callout emoji="📍">
MySQL中格式化时间字段为小时:分钟的语法为：select DATE_FORMAT(now(), '%H:00');
MySQL中四舍五入保留整数的语法为：select *round*(3.14, 0);
</callout>

### **按周统计查询指标数据**

**接口地址**:`/member/user/queryDeviceDataListByWeek`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型 |
|-|-|-|-|-|
| **iotId** | 设备id（如：EUMY0DdzDihH2LNJkdHok1u770） | query | true | String |
| **functionId** | 物模型id（如：HeartRate） | query | true | String |
| **startTime** | 查询开始时间毫秒值（如：1729353600000） | query | true | Long |
| **endTime** | 查询结束时间毫秒值（如：1729439999999） | query | true | Long |

**响应示例**:

```JSON
{
    "code":200,
    "msg":"操作成功",
    "data":[
        {
            "dateTime":"01.15",
            "dataValue":107.0
        },
        {
            "dateTime":"01.16",
            "dataValue":90.1
        },
        {
            "dateTime":"01.17",
            "dataValue":102.5
        },
        {
            "dateTime":"01.18",
            "dataValue":91.6
        },
        {
            "dateTime":"01.19",
            "dataValue":108.1
        },
        {
            "dateTime":"01.20",
            "dataValue":92.9
        },
        {
            "dateTime":"01.21",
            "dataValue":100.0
        }
    ]
}
```

<callout emoji="📍">
MySQL中格式化时间字段为月份:日期的语法为：select DATE_FORMAT(now(), '%m.%d');
</callout>



## 后台管理端

### 报警数据

#### 分页查询报警数据

**接口地址**:`/nursing/alertData/list`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 是否必须 | 数据类型 |
|-|-|-|-|
| pageNum | 页码 | true | integer(int32) |
| pageSize | 页面大小 | true | integer(int32) |
| deviceName | 设备名称（精确搜索） | false | string |
| startTime | 开始报警时间 | false | LocalDateTime |
| endTime | 结束报警时间 | false | LocalDateTime |
| status | 状态，0：待处理，1：已处理 | false | integer(int32) |

**响应示例**:

```JavaScript
{
    "total": 1,
    "rows": [
        {
            "createBy": null,
            "createTime": "2024-09-27 23:08:44",
            "updateBy": null,
            "updateTime": "2024-09-27 15:08:50",
            "remark": null,
            "id": 3,
            "iotId": "847XFy3JE4CcQk6SpHMHj0rk00",
            "deviceName": "watch01",
            "nickname": "手表1号",
            "productKey": "j0rkvUZBtRu",
            "productName": "智能定位报警手表",
            "functionId": "HeartRate",
            "accessLocation": "刘备",
            "locationType": 0,
            "physicalLocationType": -1,
            "deviceDescription": null,
            "dataValue": "94",
            "alertRuleId": 57,
            "alertReason": "HeartRate>=80.0,持续了1周期，就报警",
            "processingResult": null,
            "processorId": null,
            "processorName": null,
            "processingTime": null,
            "type": 0,
            "status": 0,
            "userId": 1
        }
    ],
    "code": 200,
    "msg": "请求成功"
}
```

#### 处理设备报警数据

**接口地址**:`/nursing/alertData/handleAlertData`

**请求方式**:`PUT`

**请求示例**:

```JavaScript
{
  "id": 3,
  "processingResult": "",    // 处理结果
  "processingTime": "yyyy-MM-dd HH:mm:ss"    // 处理时间
}
```

**响应示例**:

```JavaScript
{"msg":"操作成功","code":200}
```

### 护理任务

#### 查询护理任务列表

**接口地址**:`/nursing/task/list`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型 | schema |
|-|-|-|-|-|-|
| elderName | 老人姓名 | query | false | string |  |
| endTime | 结束时间 | query | false | LocalDateTime |  |
| nurseId | 护理员ID | query | false | integer(int64) |  |
| pageNum | 页码 | query | false | integer(int32) |  |
| pageSize | 每页显示条数 | query | false | integer(int32) |  |
| projectId | 护理项目ID | query | false | integer(int64) |  |
| startTime | 开始时间 | query | false | LocalDateTime |  |
| status | 状态 (1待执行 2已执行 3已关闭) | query | false | integer(int32) |  |

**响应示例**:

```JavaScript
{
  "total": 189,
  "rows": [
    {
      "createTime": "2024-09-27 23:08:17",
      "id": 175,
      "nursingId": "102,103",
      "projectId": 6,
      "projectName": "洗头",
      "elderId": 2,
      "elderName": "关羽",
      "bedNumber": "101-2",
      "estimatedServerTime": "2024-09-27 08:00:00",
      "status": 1,
      "nursingName": [
        "小青",
        "小白"
      ]
    },
    {
      "createTime": "2024-09-27 23:08:17",
      "id": 163,
      "nursingId": "102,103",
      "projectId": 4,
      "projectName": "助餐",
      "elderId": 2,
      "elderName": "关羽",
      "bedNumber": "101-2",
      "estimatedServerTime": "2024-09-27 08:00:00",
      "status": 1,
      "nursingName": [
        "小青",
        "小白"
      ]
    }
  ],
  "code": 200,
  "msg": "请求成功"
}
```



#### 获取护理任务详细信息

**接口地址**:`/nursing/task/{id}`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型 |
|-|-|-|-|-|
| id | 护理任务ID | path | true | integer(int64) |

**响应参数**:

| 参数名称 | 参数说明 | 类型 | 其他说明 |
|-|-|-|-|
| code |  | integer(int32) | integer(int32) |
| data |  | 护理任务VO | 护理任务VO |
|   age | 年龄 | integer(int32) |  |
|   bedNumber | 床位号 | string |  |
|   cancelReason | 取消原因 | string | 取消之后展示 |
|   createBy | 创建者 | string |  |
|   createTime | 创建时间 | string(date-time) |  |
|   elderId | 老人id | integer(int64) |  |
|   elderName | 老人姓名 | string |  |
|   estimatedServerTime | 预计服务时间 | string(date-time) |  |
|   id | 主键 | integer(int64) |  |
|   mark | 执行记录 | string | 执行之后展示 |
|   nursingId | 护理员id | string |  |
|   nursingLevelName | 护理等级名称 | string |  |
|   nursingName | 护理员姓名列表 | array | string |
|   projectId | 护理项目id | integer(int64) |  |
|   projectName | 护理项目名称 | string |  |
|   realServerTime | 实际服务时间 | string(date-time) | 执行之后展示 |
|   remark | 备注 | string |  |
|   sex | 性别 | string |  |
|   status | 状态  1待执行 2已执行 3已关闭 | integer(int32) |  |
|   taskImage | 执行图片 | string | 执行之后展示 |
|   updateBy | 更新者 | string |  |
|   updateTime | 更新时间 | string(date-time) |  |
|   updater | 执行人 | string | 执行或取消之后展示 |

**响应示例**:

```JavaScript
{
    "code": 200,
    "msg": "操作成功",
    "data": {
        "createTime": "2024-09-27 23:08:17",
        "updateBy": "1",
        "updateTime": "2024-09-27 23:25:18",
        "id": 191,
        "nursingId": "103",
        "projectId": 6,
        "projectName": "洗头",
        "elderId": 3,
        "elderName": "张飞",
        "bedNumber": "303-1",
        "estimatedServerTime": "2024-09-27 08:00:00",
        "realServerTime": "2024-09-27 23:25:13",
        "mark": "222",
        "cancelReason": "老人不舒服",
        "status": 2,
        "taskImage": "https://itheim.oss-cn-beijing.aliyuncs.com/1d7763da-f02e-4b81-b414-2d23e7503a6e.jpg",
        "nursingName": [
            "小白"
        ],
        "nursingLevelName": "5号护理等级",
        "age": 72,
        "updater": "若依"
    }
}
```

#### 取消任务

**接口地址**:`/nursing/task/cancel`

**请求方式**:`PUT`

**请求示例**:

```JavaScript
{
  "reason": "",// 取消原因
  "taskId": 0,// 护理任务ID
}
```

**响应参数**:

```JSON
{"msg":"操作成功","code":200}
```





#### 执行任务

**接口地址**:`/nursing/task/do`

**请求方式**:`PUT`

**请求示例**:

```JavaScript
{
  "estimatedServerTime": "",//执行时间
  "mark": "",//执行记录
  "taskId": 0,//护理任务ID
  "taskImage": ""//任务图片
}
```

**响应示例**:

```JavaScript
{"msg":"操作成功","code":200}
```





#### 任务改期

**接口地址**:`/nursing/task/updateTime`

**请求方式**:`PUT`

**请求示例**:

```JavaScript
{
  "estimatedServerTime": "yyyy-MM-dd HH:mm:ss",//执行时间
  "taskId": 0,//护理任务ID
}
```

**响应示例**:

```JavaScript
{"msg":"操作成功","code":200}
```

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第五章-分组实战 / 接口文档 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/R77DwOg1eiTklSk427OcKf3nnyh)
