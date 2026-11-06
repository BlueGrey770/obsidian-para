---
type: memo
date: 2026-11-06
aliases:
  - "小程序端-接口文档"
---

# 小程序端-接口文档

## 服务项接口

### 分页查询护理项目列表

- **请求路径：**`/member/orders/project/page`
- **请求方式：GET**
- **请求参数：**

  | 参数名称 | 参数说明 | 数据类型 |
  |-|-|-|
  | name | name | string |
  | pageNum | pageNum | integer(int32) |
  | pageSize | pageSize | integer(int32) |
  | status | status | integer(int32) |
- **响应示例**

```JSON
{
  "total": 9,
  "rows": [
    {
      "createBy": "1",
      "createTime": "2024-08-29 16:54:22",
      "updateBy": "1",
      "updateTime": "2024-09-19 00:02:54",
      "remark": null,
      "id": 6,
      "name": "洗头",
      "orderNo": 1,
      "unit": "次",
      "price": 20.00,
      "image": "https://itheim.oss-cn-beijing.aliyuncs.com/a38883fc-870b-40ff-a256-54ce2fc17af9.png",
      "nursingRequirement": "将患者带到洗发间进行洗发，并提供按摩和护理，保持头皮健康",
      "status": 1
    },
    {
      "createBy": "1",
      "createTime": "2024-08-29 16:54:45",
      "updateBy": null,
      "updateTime": "2024-08-29 08:53:40",
      "remark": null,
      "id": 7,
      "name": "洗脸",
      "orderNo": 1,
      "unit": "次",
      "price": 15.00,
      "image": "https://itheim.oss-cn-beijing.aliyuncs.com/95b0ad37-5d61-4ec2-a961-d6fb691a18f0.png",
      "nursingRequirement": "无",
      "status": 1
    }
  ],
  "code": 200,
  "msg": "查询成功"
}
```



### 根据编号查询护理项目信息

- **请求路径：`/member/orders/project/{id}`**
- **请求方式：GET**
- **请求参数**

| 参数名称 | 参数说明 | 请求类型 | 数据类型 |
|-|-|-|-|
| id | id | path | integer(int64) |

- **响应示例**

```JSON
{
  "code": 200,
  "msg": "操作成功",
  "data": {
    "createBy": "1",
    "createTime": "2024-08-29 16:54:22",
    "updateBy": "1",
    "updateTime": "2024-09-19 00:02:54",
    "remark": null,
    "id": 6,
    "name": "洗头",
    "orderNo": 1,
    "unit": "次",
    "price": 20.00,
    "image": "https://itheim.oss-cn-beijing.aliyuncs.com/a38883fc-870b-40ff-a256-54ce2fc17af9.png",
    "nursingRequirement": "将患者带到洗发间进行洗发，并提供按摩和护理，保持头皮健康",
    "status": 1
  }
}
```

## 参观预约接口

### **查询当天取消预约数量**

- 接口地址:`/member/reservation/cancelled-count`
- 请求方式:`GET`
- 请求参数:

无，我们需要获取线程中的用户即可，在后端获取，不需要前端传任何参数

- 响应示例

```JSON
{
  "code": 200,
  "msg": "操作成功",
  "data": 0  // 取消预约次数，当天取消预约次数大于3次，则不让预约
}
```

下面的代码可以通过long类型毫秒值获取当天开始时间和结束时间：

```JavaScript
long time = System.currentTimeMillis();
LocalDateTime ldt = LocalDateTimeUtil.of(time);
LocalDateTime startTime = ldt.toLocalDate().atStartOfDay();
LocalDateTime endTime = startTime.plusDays(1);
```

参考SQL：

```SQL
-- 查询当天取消预约数量
select * from reservation where status = 2
        and update_by = 2 and update_time between '2022-03-01' and '2022-03-31';
```

### **查询每个时间段剩余预约次数**

- 接口地址:`/member/reservation/countByTime`
- 请求方式:`GET`
- 请求参数:

<sheet sheet-id="W9EBba" token="IuBVsGmoahwMLftUy1zcQVFsnug"></sheet>

- 返回示例：

```JSON
{
  "code": 200,
  "msg": "操作成功",
  "data": [
    {
      "time": "2024-09-19 08:00:00",
      "count": 5  // 目前剩余5次
    },
    {
      "time": "2024-09-19 10:30:00",
      "count": 0    // 当前时间段已约满，不能再次选择该时间，这个时间返回的是剩余预约次数
    }
    // 其他时间段，没有返回，表示可以预约6次
  ]
}
```

参考SQL：

```SQL
-- 查询每个时间段剩余预约次数
select time, 6 - COUNT(*) as count from reservation
    where `time` between '2022-03-01' and '2022-03-31'
    and status != 2 group by `time`;
```

### **新增预约**

- 接口地址:`/member/reservation/`
- 请求方式:`POST`
- 请求参数:

```JSON
{
  "mobile": "13211223569",//手机号
  "name": "李天天",//预约人
  "time": "2024-09-19 10:30:00",//时间
  "type": 0,//预约类型  0为参观预约
  "visitor": "李霸天"//家人姓名（老人姓名）
}
```

- 响应示例：

```JSON
{
  "code": 200,
  "msg": "操作成功"
}
```

### **分页查询预约**

- 接口地址:`/member/reservation/page`
- 请求方式:`GET`
- 请求参数:

<sheet sheet-id="MZ5EIa" token="IuBVsGmoahwMLftUy1zcQVFsnug"></sheet>

- 响应示例：

```JSON
{
  "code": 200,
  "msg": "操作成功",
  "data": {
    "total": 9,
    "rows": [
      {
        "createBy": "3",
        "createTime": "2024-09-19 00:45:00",
        "updateBy": "3",
        "updateTime": null,
        "remark": null,
        "id": 110,
        "name": "李天天",
        "mobile": "13211223543",
        "time": "2024-09-19T10:30:00",
        "visitor": "李霸天",
        "type": 0,
        "status": 0
      }
    ],
    "code": 200,
    "msg": "请求成功"
  }
}
```



### **取消预约**

- 接口地址:`/member/reservation/{id}/cancel`
- 请求方式:`PUT`
- 请求参数:

路径参数

<sheet sheet-id="brqad1" token="IuBVsGmoahwMLftUy1zcQVFsnug"></sheet>

- 响应示例：

```JSON
{
  "code": 200,
  "msg": "操作成功"
}
```

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第三章-AI辅助快速完成登录认证 / 小程序端-接口文档 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/NgF5wI8TkinS06kK8kncfr1Dn2g)
