---
type: memo
date: 2026-11-06
aliases:
  - "智能监测-接口文档"
---

# 智能监测-接口文档

## 设备管理

### 从物联网平台同步产品列表

**接口地址**:`/nursing/device/syncProductList`

**请求方式**:`POST`

**请求参数**:

暂无

**响应示例**:

```JavaScript
{"msg":"操作成功","code":200}
```

### 查询所有产品列表

**接口地址**:`/nursing/device/allProduct`

**请求方式**:`GET`

**请求参数**:

暂无

**响应示例**:

```JSON
{
    "msg": "操作成功",
    "code": 200,
    "data": [
        {
            "productId": "67b08f52bab900244b1fc77c",
            "name": "睡眠监测带"
        },
        {
            "productId": "67b08da5bab900244b1fc6f0",
            "name": "烟雾报警器"
        },
        {
            "productId": "67ad95860c504e29c72b2436",
            "name": "智能定位报警手表"
        }
    ]
}
```

### 注册设备

**接口地址**:`/nursing/device/register`

**请求方式**:`POST`

**请求示例**:

- 新增随身设备-智能报警手表

```JSON
{
  "deviceName":"手表1号",//设备名称
  "bindingLocation": "540",//由于绑定的是老人，这个值是老人id
  "locationType": 0,//位置类型 0：随身设备 1：固定设备
  "nodeId": "watch01",//设备节点id
  "physicalLocationType": -1,//如果是随身设备，传-1
  "productKey": "6625d23371d845632a067bdf",//产品id
  "productName": "watch",//产品名称
  "remark": "张小丽", //由于绑定的是老人，这个值是老人的姓名
  "deviceDescription": "1"//由于绑定的是老人，这个值是老人id
}
```

- 新增固定设备-烟雾报警器

```JSON
{
  "deviceName":"烟雾01",//设备名称
  "bindingLocation": "79",//由于绑定的是房间，这个值是房间id
  "locationType": 1,//位置类型 0：随身设备 1：固定设备
  "nodeId": "yw01",//设备节点名称
  "physicalLocationType": 1,//物理位置类型 0楼层 1房间 2床位
  "productKey": "6625d6d171d845632a067c9f",
  "productName": "烟雾报警器",
  "remark": "1楼,101",//对于房间的描述
  "deviceDescription": "1,1"//绑定的是房间,这个是楼层id和房间id，方便数据回显使用
}
```

- 新增固定设备-睡眠检测带

```JSON
{
  "deviceName": "睡眠检测带01",
  "bindingLocation": "1",//由于绑定的是床位，这个值是床位id
  "locationType": 1,//位置类型 0：随身设备 1：固定设备
  "nodeId": "sm01",
  "physicalLocationType":2,//物理位置类型 0楼层 1房间 2床位
  "productKey": "6625d6d171d845632a067c9f",
  "productName": "睡眠检测带",
  "remark": "1楼,101,101-1", //对于床位的描述
  "deviceDescription": "1,1,5"//绑定的是床位,这个是楼层id、房间id、床位id，方便数据回显使用
}
```

**响应示例**:

```JSON
{"msg":"操作成功","code":200}
```

### 分页查询设备列表

**接口地址**:`/nursing/device/list`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 数据类型 |
|-|-|-|
| pageNum | 页码 | integer(int32) |
| pageSize | 页面大小 | integer(int32) |
| deviceName | 设备名称 | string |
| locationType | 位置类型 0 随身设备 1固定设备 | integer(int32) |
| productKey | 产品的ProductKey,物联网平台产品唯一标识 | string |

**响应示例**:

```JSON
{
    "total": 2,
    "rows": [
        {
            "createBy": "1",
            "createTime": "2024-09-20 14:29:02",
            "updateBy": null,
            "updateTime": null,
            "remark": "老李",
            "id": 201,
            "iotId": "Y9ONkL8MuqrVvr9zoej3j0rk00",
            "bindingLocation": "327",
            "locationType": 0,
            "physicalLocationType": -1,
            "deviceName": "watch08",
            "nickname": "手表8号",
            "productKey": "j0rk0xgSbjY",
            "productName": "健康定位报警手表",
            "deviceDescription": null,
            "haveEntranceGuard": 0
        },
        {
            "createBy": "1",
            "createTime": "2024-09-20 14:38:25",
            "updateBy": null,
            "updateTime": null,
            "remark": "1楼,101,101-1",
            "id": 202,
            "iotId": "TI3QtZhEWdKT6kU7Nepaj0rk00",
            "bindingLocation": "1",
            "locationType": 1,
            "physicalLocationType": 2,
            "deviceName": "sm_09",
            "nickname": "睡眠检测带09",
            "productKey": "j0rkJaV0DDA",
            "productName": "睡眠监测带",
            "deviceDescription": "1,1,1",
            "haveEntranceGuard": 0
        }
    ],
    "code": 200,
    "msg": "查询成功"
}
```

### 查询设备详细数据

**接口地址**:`/nursing/device/{iotId}`

**请求方式**:GET

**请求示例**:

路径参数：iotId  设备id

**响应示例**:

```JSON
{
    "msg": "操作成功",
    "code": 200,
    "data": {
        "id": 4,
        "iotId": "67b08da5bab900244b1fc6f0_yw01",
        "deviceName": "烟雾01",
        "nodeId": "yw01",
        "secret": "23abb22b43fe4d33b6322a198c58e22d",
        "productKey": null,
        "productName": "烟雾报警器",
        "locationType": 1,
        "bindingLocation": 2,
        "remark": "1楼,101",
        "deviceStatus": "ONLINE",
        "activeTime": "2025-02-15 22:16:42",
        "createTime": "2025-02-15 20:55:15",
        "createBy": 1,
        "creator": null
    }
}
```

### 查看设备上报的数据

**接口地址**: `/nursing/device/queryServiceProperties/{iotId}`

**请求方式**:`GET`

**请求示例**:

路径参数：iotId  设备id

**响应示例**:

```JSON
{
    "msg": "操作成功",
    "code": 200,
    "data": [
        {
            "functionId": "HeartRate",
            "eventTime": "2025-02-19T15:04:24",
            "value": 60
        },
        {
            "functionId": "BedTime",
            "eventTime": "2025-02-19T15:04:24",
            "value": 1739948665934
        },
        {
            "functionId": "SleepPhaseState",
            "eventTime": "2025-02-19T15:04:24",
            "value": 2
        },
        {
            "functionId": "RespiratoryRate",
            "eventTime": "2025-02-19T15:04:24",
            "value": 77
        },
        {
            "functionId": "BedExitCount",
            "eventTime": "2025-02-19T15:04:24",
            "value": 3
        }
    ]
}
```

### 修改设备

**接口地址**:`/nursing/device`

**请求方式**:`PUT`

**请求示例**:

```JSON
{
  "id":"12",//设备主键id
  "deviceName":"手表1号",//设备名称
  "bindingLocation": "540",//由于绑定的是老人，这个值是老人id
  "locationType": 0,//位置类型 0：随身设备 1：固定设备
  "nodeId": "12345",//设备节点id
  "physicalLocationType": -1,//如果是随身设备，传-1
  "productKey": "6625d23371d845632a067bdf",//产品id
  "productName": "watch",//产品名称
  "remark": "张小丽", //由于绑定的是老人，这个值是老人的姓名
  "deviceDescription": "1"//绑定的是老人，这个值是老人id  ,方便数据回显
}
```

**响应示例**:

```JavaScript
{"msg":"操作成功","code":200}
```

### 删除设备

**接口地址**:`/nursing/device/{iotId}`

**请求方式**:`DELETE`

**请求示例**:

路径参数：iotId  设备id

**响应示例**:

```JavaScript
{"msg":"操作成功","code":200}
```

### 查询产品详情(报警规则模块使用)

**接口地址**:`/nursing/device/queryProduct/{productKey}`

**请求方式**:`GET`

**请求示例**:

路径参数：productKey   产品的id

**响应示例**:

```JavaScript
{
    "msg": "操作成功",
    "code": 200,
    "data": [
        {
            "service_id": "sm_services",
            "service_type": "sm_services",
            "properties": [
                {
                    "property_name": "BedTime",
                    "data_type": "DateTime",
                    "required": false,
                    "max_length": 30,
                    "step": 0.0,
                    "method": "RW",
                    "description": "离床时间"
                },
                {
                    "property_name": "BedExitCount",
                    "data_type": "int",
                    "required": false,
                    "min": "0",
                    "max": "10",
                    "max_length": 0,
                    "step": 0.0,
                    "unit": "",
                    "method": "RW",
                    "description": "离床次数"
                },
                {
                    "property_name": "SleepPhaseState",
                    "data_type": "int",
                    "required": false,
                    "min": "0",
                    "max": "2",
                    "max_length": 0,
                    "step": 0.0,
                    "unit": "",
                    "method": "RW",
                    "description": "睡眠状态"
                },
                {
                    "property_name": "HeartRate",
                    "data_type": "int",
                    "required": false,
                    "min": "0",
                    "max": "180",
                    "max_length": 0,
                    "step": 0.0,
                    "unit": "",
                    "method": "RW",
                    "description": "心率"
                },
                {
                    "property_name": "RespiratoryRate",
                    "data_type": "int",
                    "required": false,
                    "min": "0",
                    "max": "150",
                    "max_length": 0,
                    "step": 0.0,
                    "unit": "",
                    "method": "RW",
                    "description": "呼吸率"
                }
            ],
            "option": "Optional"
        }
    ]
}
```

## 设备数据

### 获取设备数据分页结果

**接口地址**:`/nursing/data/list`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 数据类型 |
|-|-|-|
| pageNum | 页码 | integer(int32) |
| pageSize | 每页大小 | integer(int32) |
| deviceName | 设备名称 | string |
| endTime | 结束时间 | date |
| functionId | 功能ID | string |
| startTime | 开始时间 | date |

**响应示例**:

```JSON
{
        "code": 0,
        "data": {
                "page": 0,
                "pageSize": 0,
                "pages": 0,
                "records": [
                        {
                                "accessLocation": "",
                                "adminCreator": "",
                                "alarmTime": "",
                                "createBy": 0,
                                "createDay": "",
                                "createTime": "",
                                "createType": 0,
                                "creator": "",
                                "dataState": "",
                                "dataValue": "",
                                "deviceDescription": "",
                                "deviceName": "",
                                "functionId": "",
                                "id": 0,
                                "iotId": "",
                                "locationType": 0,
                                "nickname": "",
                                "physicalLocationType": 0,
                                "productKey": "",
                                "productName": "",
                                "remark": "",
                                "updateBy": 0,
                                "updateTime": "",
                                "updater": ""
                        }
                ],
                "total": 0
        },
        "msg": "",
        "operationTime": ""
}
```

## 智能床位

### **获取所有楼层 (智能床位)**

**接口地址**:`/elder/floor/getAllFloorsWithDevice`

**请求方式**:`GET`

**请求参数**:

无

**响应示例**:

```JSON
{
    "code": 200,
    "msg": "操作成功",
    "data": [
        {
            "id": 1,
            "name": "1楼",
            "code": 11,
        }
    ]
}
```

### **获取房间中的智能设备及数据**

**接口地址**:`/elder/room/getRoomsWithDeviceByFloorId/{floorId}`

**请求方式**:`GET`

**请求参数**:

**响应示例**:

```JSON
{
    "code": 200,
    "msg": "操作成功",
    "data": [
        { //房间
            "id": 1,
            "floorName": null,
            "floorId": "1",
            "roomId": null,
            "code": "101",
            "price": null,
            "bedVoList": [//床位
                {
                    "id": 170,
                    "bedNumber": "101-2",
                    "bedStatus": 1,
                    "roomId": null,
                    "ename": "老李",
                    "elderId": 327,
                    "userVos": null,
                    "deviceVos": []
                },
                {
                    "id": 1,
                    "bedNumber": "101-1",
                    "bedStatus": 1,
                    "roomId": null,
                    "ename": "老李头儿",
                    "elderId": 328,
                    "userVos": null,
                    "deviceVos": []
                }
            ],
            "deviceVos": [//房间的设备
                {
                    "id": 1,
                    "iotId": "kJ2mrhmpY0Z2HeCZmmcNj0rk00",
                    "deviceName": "yw01",
                    "productKey": "j0rkuqwMpYq",
                    "productName": "烟雾报警器",
                    "deviceDataVos": [
                        {
                            "id": 1058486,
                            "createTime": "2024-09-24 00:12:55",
                            "updateTime": null,
                            "remark": null,
                            "iotId": "kJ2mrhmpY0Z2HeCZmmcNj0rk00",
                            "deviceName": "yw01",
                            "nickname": "烟雾01",
                            "productKey": "j0rkuqwMpYq",
                            "productName": "烟雾报警器",
                            "functionId": "CurrentHumidity",
                            "accessLocation": "1楼,101",
                            "locationType": 1,
                            "physicalLocationType": 1,
                            "deviceDescription": "1,1",
                            "dataValue": "66",
                            "alarmTime": "2024-09-24T00:12:54.877"
                        },
                        {
                            "id": 1058487,
                            "createTime": "2024-09-24 00:12:55",
                            "updateTime": null,
                            "remark": null,
                            "iotId": "kJ2mrhmpY0Z2HeCZmmcNj0rk00",
                            "deviceName": "yw01",
                            "nickname": "烟雾01",
                            "productKey": "j0rkuqwMpYq",
                            "productName": "烟雾报警器",
                            "functionId": "SmokeSensorState",
                            "accessLocation": "1楼,101",
                            "locationType": 1,
                            "physicalLocationType": 1,
                            "deviceDescription": "1,1",
                            "dataValue": "0",
                            "alarmTime": "2024-09-24T00:12:54.877"
                        },
                        {
                            "id": 1058488,
                            "createTime": "2024-09-24 00:12:55",
                            "updateTime": null,
                            "remark": null,
                            "iotId": "kJ2mrhmpY0Z2HeCZmmcNj0rk00",
                            "deviceName": "yw01",
                            "nickname": "烟雾01",
                            "productKey": "j0rkuqwMpYq",
                            "productName": "烟雾报警器",
                            "functionId": "IndoorTemperature",
                            "accessLocation": "1楼,101",
                            "locationType": 1,
                            "physicalLocationType": 1,
                            "deviceDescription": "1,1",
                            "dataValue": "20",
                            "alarmTime": "2024-09-24T00:12:54.877"
                        }
                    ]
                }
            ]
        },
        {  //房间
            "id": 3,
            "floorName": null,
            "floorId": "1",
            "roomId": null,
            "code": "103",
            "price": null,
            "bedVoList": [//床位
                {
                    "id": 4,
                    "bedNumber": "103-1",
                    "bedStatus": 1,
                    "roomId": null,
                    "ename": "李大",
                    "elderId": 330,
                    "userVos": null,
                    "deviceVos": [ 
                        { //床位的设备
                            "id": 2,
                            "iotId": "PqLvznGw8Y7If8KFQhc3j0rk00",
                            "deviceName": "sm_01",
                            "productKey": "j0rkcHVJrcL",
                            "productName": "睡眠检测带",
                            "deviceDataVos": [  //床位的设备数据
                                {
                                    "id": 1058489,
                                    "createTime": "2024-09-24 00:12:57",
                                    "updateTime": null,
                                    "remark": null,
                                    "iotId": "PqLvznGw8Y7If8KFQhc3j0rk00",
                                    "deviceName": "sm_01",
                                    "nickname": "睡眠01",
                                    "productKey": "j0rkcHVJrcL",
                                    "productName": "睡眠检测带",
                                    "functionId": "HeartRate",
                                    "accessLocation": "1楼,103,103-1",
                                    "locationType": 1,
                                    "physicalLocationType": 2,
                                    "deviceDescription": "1,3,4",
                                    "dataValue": "30",
                                    "alarmTime": "2024-09-24T00:12:56.677"
                                },
                                {
                                    "id": 1058490,
                                    "createTime": "2024-09-24 00:12:57",
                                    "updateTime": null,
                                    "remark": null,
                                    "iotId": "PqLvznGw8Y7If8KFQhc3j0rk00",
                                    "deviceName": "sm_01",
                                    "nickname": "睡眠01",
                                    "productKey": "j0rkcHVJrcL",
                                    "productName": "睡眠检测带",
                                    "functionId": "shuimianzhuangtai",
                                    "accessLocation": "1楼,103,103-1",
                                    "locationType": 1,
                                    "physicalLocationType": 2,
                                    "deviceDescription": "1,3,4",
                                    "dataValue": "1",
                                    "alarmTime": "2024-09-24T00:12:56.677"
                                },
                                {
                                    "id": 1058491,
                                    "createTime": "2024-09-24 00:12:57",
                                    "updateTime": null,
                                    "remark": null,
                                    "iotId": "PqLvznGw8Y7If8KFQhc3j0rk00",
                                    "deviceName": "sm_01",
                                    "nickname": "睡眠01",
                                    "productKey": "j0rkcHVJrcL",
                                    "productName": "睡眠检测带",
                                    "functionId": "RespiratoryRate",
                                    "accessLocation": "1楼,103,103-1",
                                    "locationType": 1,
                                    "physicalLocationType": 2,
                                    "deviceDescription": "1,3,4",
                                    "dataValue": "96",
                                    "alarmTime": "2024-09-24T00:12:56.677"
                                },
                                {
                                    "id": 1058492,
                                    "createTime": "2024-09-24 00:12:57",
                                    "updateTime": null,
                                    "remark": null,
                                    "iotId": "PqLvznGw8Y7If8KFQhc3j0rk00",
                                    "deviceName": "sm_01",
                                    "nickname": "睡眠01",
                                    "productKey": "j0rkcHVJrcL",
                                    "productName": "睡眠检测带",
                                    "functionId": "lichuangcishu",
                                    "accessLocation": "1楼,103,103-1",
                                    "locationType": 1,
                                    "physicalLocationType": 2,
                                    "deviceDescription": "1,3,4",
                                    "dataValue": "7",
                                    "alarmTime": "2024-09-24T00:12:56.677"
                                }
                            ]
                        }
                    ]
                }
            ],
            "deviceVos": []
        }
    ]
}
```

## 小程序端-健康数据展示

### **查询健康数据**

**接口地址**: `/member/user/queryServiceProperties/{iotId}`

**请求方式**:`GET`

**请求示例**:

路径参数：iotId  设备id

**响应示例**:

```JSON
{
    "msg": "操作成功",
    "code": 200,
    "data": [
        {
            "functionId": "HeartRate",
            "eventTime": "2025-02-19T15:04:24",
            "value": 60
        },
        {
            "functionId": "BedTime",
            "eventTime": "2025-02-19T15:04:24",
            "value": 1739948665934
        },
        {
            "functionId": "SleepPhaseState",
            "eventTime": "2025-02-19T15:04:24",
            "value": 2
        },
        {
            "functionId": "RespiratoryRate",
            "eventTime": "2025-02-19T15:04:24",
            "value": 77
        },
        {
            "functionId": "BedExitCount",
            "eventTime": "2025-02-19T15:04:24",
            "value": 3
        }
    ]
}
```

###  **按天统计查询指标数据**

**接口地址**:`/customer/user/queryDeviceDataListByDay`

**请求方式**:`GET`

**请求参数**:

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
  ],
  "operationTime": null
}
```

### **按周统计查询指标数据**

**接口地址**:`/customer/user/pageQueryAlertData`

**请求方式**:`GET`

**请求参数**:

**响应示例**:

```JSON
{
    "code":200,
    "msg":"操作成功",
    "data":[
        {
            "dateTime":"01.15",
            "dataValue":107
        },
        {
            "dateTime":"01.16",
            "dataValue":90
        },
        {
            "dateTime":"01.17",
            "dataValue":102
        },
        {
            "dateTime":"01.18",
            "dataValue":91
        },
        {
            "dateTime":"01.19",
            "dataValue":108
        },
        {
            "dateTime":"01.20",
            "dataValue":92
        },
        {
            "dateTime":"01.21",
            "dataValue":100
        }
    ],
    "operationTime":null
}
```

## 报警规则

### 新增报警规则

- **接口地址**:`/alert-rule/create`
- **请求方式**:`POST`
- **请求参数：**

```JavaScript
{
  "alertDataType": 1,//报警数据类型  0:老人异常数据   1:设备异常数据
  "alertEffectivePeriod": "00:00:00~23:59:59",//报警生效时段
  "alertRuleName": "心率低值监测",//告警规则名称
  "alertSilentPeriod": 5,//报警沉默周期
  "deviceName": "全部设备",//设备名称
  "duration": "1",//持续周期
  "functionId": "HeartRate",//功能标识(物模型)
  "functionName": "心率",//功能名称
  "iotId": "-1",///设备唯一标识  -1代表该产品下的所有设备
  "moduleId": "-1",//模块的key
  "moduleName": "默认模块",//模块的key
  "operator": "<",//操作符
  "productKey": "j0rkJaV0DDA",//产品的key
  "productName": "睡眠监测带",//产品名称
  "remark": "5",//也是报警沉默周期
  "status": 0,//状态
  "value": 65 //报警阈值
}
```

- **响应示例**:

```JavaScript
{
        "code": 0,
        "msg": "",
        "operationTime": ""
}
```

### 分页获取告警规则列表

- **接口地址**:`/alert-rule/get-page`
- **请求方式**:`GET`
- **请求参数**:

| 参数名称 | 参数说明 | 是否必须 | 数据类型 |
|-|-|-|-|
| pageNum | 页码 | true | integer(int32) |
| pageSize | 每页大小 | true | integer(int32) |
| alertRuleName | 规则名称，精确查询 | false | string |
| functionName | 功能名称，精确查询 | false | string |
| productKey | 产品ID | false | string |

- **响应示例**:

```JavaScript
{
    "code": 200,
    "msg": "操作成功",
    "data": {
        "total": "3",
        "pageSize": 10,
        "pages": "1",
        "page": 1,
        "records": [
            {
                "id": "50",
                "createTime": "2024-01-22 00:05:48",
                "updateTime": "2024-01-22 00:06:00",
                "createBy": "1671403256519078138",
                "updateBy": "1671403256519078138",
                "remark": "5",
                "creator": "超级管理员",
                "productKey": "j0rkTq4tbnp",
                "productName": "智能定位手表",
                "moduleId": "-1",
                "moduleName": "默认模块",
                "functionName": "心率",
                "functionId": "HeartRate",
                "iotId": "-1",
                "deviceName": "全部设备",
                "alertDataType": 0,
                "alertRuleName": "手表心率低值报警",
                "operator": "<",
                "value": 60.0,
                "duration": 1,
                "alertEffectivePeriod": "00:00:00~23:59:59",
                "alertSilentPeriod": 5,
                "status": 1,
                "rules": "心率<60.0,持续1个周期就报警"
            }
        ]
    },
    "operationTime": null
}
```

### 获取单个告警规则

**接口地址**:`/alert-rule/read/{id}`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型 |
|-|-|-|-|-|
| id | 主键 | path | true | integer(int64) |

**响应示例**:

```JavaScript
{
    "code": 200,
    "msg": "操作成功",
    "data": {
        "id": "48",
        "createTime": "2024-01-21 23:55:31",
        "updateTime": "2024-01-21 23:55:31",
        "createBy": "1671403256519078138",
        "remark": "5",
        "productKey": "j0rkJaV0DDA",
        "productName": "睡眠监测带",
        "moduleId": "-1",
        "moduleName": "默认模块",
        "functionName": "心率",
        "functionId": "HeartRate",
        "iotId": "-1",
        "deviceName": "全部设备",
        "alertDataType": 1,
        "alertRuleName": "心率低值监测",
        "operator": "<",
        "value": 60.0,
        "duration": 1,
        "alertEffectivePeriod": "00:00:00~23:59:59",
        "alertSilentPeriod": 5,
        "status": 0,
    },
    "operationTime": null
}
```

### 更新告警规则

**接口地址**:`/alert-rule/update/{id}`

**请求方式**:`PUT`

**请求示例**:

```JavaScript
{
  "alertDataType": 1,//报警数据类型  0:老人异常数据   1:设备异常数据
  "alertEffectivePeriod": "00:00:00~23:59:59",//报警生效时段
  "alertRuleName": "心率低值监测",//告警规则名称
  "alertSilentPeriod": 5,//报警沉默周期
  "deviceName": "全部设备",//设备名称
  "duration": "1",//持续周期
  "functionId": "HeartRate",//功能标识(物模型)
  "functionName": "心率",//功能名称
  "id": "48",//主键
  "iotId": "-1",///设备唯一标识  -1代表该产品下的所有设备
  "moduleId": "-1",//模块的key
  "moduleName": "默认模块",//模块的key
  "operator": "<",//操作符
  "productKey": "j0rkJaV0DDA",//产品的key
  "productName": "睡眠监测带",//产品名称
  "remark": "5",//也是报警沉默周期
  "status": 0,//状态
  "value": 65 //报警阈值
}
```

**响应示例**:

```JavaScript
{
        "code": 0,
        "msg": "",
        "operationTime": ""
}
```

### 删除告警规则

**接口地址**:`/alert-rule/delete/{id}`

**请求方式**:`DELETE`

**请求参数**:

| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型 |
|-|-|-|-|-|
| id | id | path | true | integer(int64) |

**响应示例**:

```JavaScript
{"code":200,"msg":"操作成功","data":"操作成功","operationTime":null}
```

## 报警数据

### 分页查询报警数据

**接口地址**:`/alert-data/pageQuery`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 是否必须 | 数据类型 |
|-|-|-|-|
| pageNum | 页码 | true | integer(int32) |
| pageSize | 页面大小 | true | integer(int32) |
| deviceName | 设备名称（精确搜索） | false | string |
| endTime | 结束报警时间 | false | integer(int64) |
| id | 报警数据id | false | integer(int64) |
| startTime | 开始报警时间 | false | integer(int64) |
| status | 状态，0：待处理，1：已处理 | false | integer(int32) |

**响应示例**:

```JavaScript
{
    "code": 200,
    "msg": "操作成功",
    "data": {
        "total": "1",
        "pageSize": 10,
        "pages": "1",
        "page": 1,
        "records": [
            {
                "id": "4",
                "iotId": "fOd8epGejTVSbKDGNVsSj0rk00",
                "deviceName": "sm_01",
                "nickname": "睡眠监测带01",
                "productKey": "j0rkJaV0DDA",
                "productName": "睡眠监测带",
                "functionId": "HeartRate",
                "accessLocation": "1楼,101,101-3",
                "dataValue": "22",
                "alertRuleId": "48",
                "alertReason": "心率<65.0,持续3个周期就报警",
                "processingResult": "111",
                "processorId": null,
                "processorName": "超级管理员",
                "processingTime": "2024-01-27 00:02:00",
                "type": "0",
                "status": "1",
                "userId": "1671403256519078256",
                "createTime": "2024-01-22 20:02:00"
            }
        ]
    },
    "operationTime": null
}
```

### 处理设备报警数据

**接口地址**:`/alert-data/handleAlertData/{id}`

**请求方式**:`PUT`

**请求示例**:

```JavaScript
{
  "processingResult": "",
  "processingTime": ""
}
```

**响应示例**:

```JavaScript
{"code":200,"msg":"操作成功","data":"操作成功","operationTime":null}
```

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第四章-智能监测系统（华为云） / 智能监测-接口文档 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/QfHVwoIdFizIokkSeH2cRJw7nQf)
