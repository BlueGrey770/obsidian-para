---
type: memo
date: 2026-11-06
aliases:
  - "ru-tui-guan-li-jie-kou-wen-dang"
  - "入退管理-接口文档"
---

# 入退管理-接口文档

## 入住办理

### 分页查询入住列表

**接口地址**:`/nursing/checkIn/pageQuery`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 是否必须 | 数据类型 |
|-|-|-|-|
| pageNum | 页码 | true | integer(int32) |
| pageSize | 页面大小 | true | integer(int32) |
| elderName | 老人姓名，模糊查询 | false | string |
| idNumber | 身份证号，精确查询 | false | string |

**响应示例**:

```JSON
{
  "total": 162,
  "rows": [
    {
      "id": 338,
      "elderName": "李河",
      "elderIdCardNo": "132143195601021235",
      "bedNumber": "606-1",
      "nursingLevelName": "一级护理等级",
      "checkInStartTime": "2024-04-20T00:00:00.000+08:00",
      "checkInEndTime": "2024-05-31T00:00:00.000+08:00",
      "applicat": "admin",
      "createTime": "2024-04-19T17:05:57.000+08:00"
    },
    {
      "id": 337,
      "elderName": "李山",
      "elderIdCardNo": "132143195601021234",
      "bedNumber": "606-1",
      "nursingLevelName": "一级护理等级",
      "checkInStartTime": "2024-04-20T00:00:00.000+08:00",
      "checkInEndTime": "2024-05-31T00:00:00.000+08:00",
      "applicat": "admin",
      "createTime": "2024-04-19T16:32:36.000+08:00"
    }
  ],
  "code": 200,
  "msg": "查询成功"
}
```

### 查询所有护理等级信息

**接口地址**:`/nursing/level/all`

**请求方式**:`GET`

**请求参数**:

暂无

**响应示例**:

```JavaScript
{
    "msg": "操作成功",
    "code": 200,
    "data": [
        {
            "createBy": "1671403256519078153",
            "createTime": "2023-09-26 15:40:08",
            "updateTime": "2023-09-26 15:40:08",
            "id": 1,
            "name": "特级护理等级",
            "lplanId": 1,
            "fee": 2700.00,
            "status": 1,
            "description": "特级护理等级适用于那些需要高度关注和专业照顾的老年人，包含了全面的护理项目和照顾"
        },
        {
            "createBy": "1671403256519078153",
            "createTime": "2023-09-26 15:45:12",
            "updateTime": "2023-09-26 15:45:12",
            "id": 2,
            "name": "一级护理等级",
            "lplanId": 2,
            "fee": 2000.00,
            "status": 1,
            "description": "一级护理等级适用于需要一定程度护理和照料的老年人，该等级提供了基本的日常护理项目"
        }
        .... 
    ]
}
```

### 根据床位状态查询获取所有楼层数据

**接口地址**:`/elder/floor/getRoomAndBedByBedStatus/{status}`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 是否必须 |
|-|-|-|
| status | 床位状态，0：未入住，1：已入住 | true |

**响应示例**:

```JavaScript
{
  "msg": "操作成功",
  "code": 200,
  "data": [
    {
      "value": '1',
      "label": '1楼',
      "children": [
        {
          "value": '2',
          "label": '101',
          "children": [
            {
              "value": '22',
              "label": '101-1'
            }
          ]
        },
        {
          "value": '3',
          "label": '102',
          "children": [
            {
              "value": '33',
              "label": '101-2'
            }
          ]
        }
      ]
    }
  ]
}
```



### 根据房间id查询房间数据(楼层、房间、价格)

**接口地址**:`/elder/room/one/{id}`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 请求类型 | 是否必须 | 数据类型 |
|-|-|-|-|-|
| id | id | path | true | integer(int64) |

**响应示例**:

```JSON
{
    "msg": "操作成功",
    "code": 200,
    "data": {
        "floorName": "8楼",
        "floorId": "8",
        "roomId": "51",
        "code": "801",
        "price": "8000.00"
    }
}
```

### 申请入住

**接口地址**:`/nursing/checkIn/apply`

**请求方式**:`POST`

**请求示例**:

```JSON
{
  "checkInElderDto": {
    "address": "北京市昌平区",
    "age": "60",
    "birthday": "1946-12-19",
    "idCardNationalEmblemImg": "https://itheim.oss-cn-beijing.aliyuncs.com/481be4c8-5a04-4475-8712-bd3f53f6e4ff.jpg",
    "idCardNo": "132123194612191234",
    "idCardPortraitImg": "https://itheim.oss-cn-beijing.aliyuncs.com/2038127d-bd2e-46eb-8b9a-867205a048c6.jpg",
    "image": "https://itheim.oss-cn-beijing.aliyuncs.com/9fc6b8ac-3ed5-432c-8835-8ca91abc2569.png",
    "name": "刘天龙",
    "phone": "15543234565",
    "sex": 1
  },
  "elderFamilyDtoList": [
    {
      "kinship": "1",
      "name": "刘天",
      "phone": "15543234568"
    }
  ],
  "checkInConfigDto": {
    "bedFee": "4000.00",
    "bedId": "10",
    "code": "107",
    "deposit": 3000,
    "endDate": "2024-09-30 00:00:00",
    "feeEndDate": "2024-08-31 00:00:00",
    "feeStartDate": "2024-08-26 00:00:00",
    "floorId": "1",
    "floorName": "1楼",
    "governmentSubsidy": 1,
    "insurancePayment": 1,
    "nursingFee": 2000,
    "nursingLevelId": 77,
    "nursingLevelName": "一级护理等级",
    "otherFees": 1,
    "roomId": "7",
    "startDate": "2024-08-26 00:00:00"
  },
  "checkInContractDto": {
    "agreementPath": "https://itheim.oss-cn-beijing.aliyuncs.com/72731f06-d7d3-41ca-9343-b9422993ebda.png",
    "contractName": "刘天龙",
    "signDate": "2024-08-26 15:47:39",
    "thirdPartyName": "刘天",
    "thirdPartyPhone": "15543234568"
  }
}
```

**响应示例**:

```JavaScript
{"msg":"操作成功","code":200}
```

### 查询入住详情

**接口地址**:`/nursing/checkIn/detail/{id}`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 是否必须 |
|-|-|-|
| id | 入住id | true |

**响应示例**:

```JSON
{
    "msg": "操作成功",
    "code": 200,
    "data": {
        "checkInElderVo": {
            "id": 326,
            "name": "李天龙",
            "idCardNo": "132123195612131234",
            "birthday": "2024-08-11",
            "sex": 1,
            "phone": "13211223322",
            "address": "北京市昌平区",
            "image": "https://itheim.oss-cn-beijing.aliyuncs.com/85f734bd-26be-4be1-8086-0ffae0b2374d.png",
            "idCardNationalEmblemImg": "https://itheim.oss-cn-beijing.aliyuncs.com/843b67d3-1061-4a27-89e1-a7974c826c36.jpg",
            "idCardPortraitImg": "https://itheim.oss-cn-beijing.aliyuncs.com/00fd6a90-d982-4981-92c8-5c29ebb276d9.jpg",
            "age": 67
        },
        "elderFamilyVoList": [
            {
                "name": "李彤",
                "phone": "13211223311",
                "kinship": "0"
            }
        ],
        "checkInConfigVo": {
            "createBy": "1",
            "createTime": "2024-08-26 15:44:08",
            "updateTime": "2024-08-26 07:44:08",
            "id": 2,
            "checkInId": 6,
            "nursingLevelId": 77,
            "nursingLevelName": "一级护理等级",
            "feeStartDate": "2024-08-26 00:00:00",
            "feeEndDate": "2024-08-31 00:00:00",
            "deposit": 3000.00,
            "nursingFee": 2000.00,
            "bedFee": 6000.00,
            "insurancePayment": 1.00,
            "governmentSubsidy": 1.00,
            "otherFees": 1.00,
            "sortOrder": 0,
            "startDate": "2024-08-26 00:00:00",
            "endDate": "2024-09-30 00:00:00",
            "bedNumber": "106-1"
        },
        "contract": {
            "createBy": "1",
            "createTime": "2024-08-26 15:44:08",
            "updateTime": "2024-08-26 07:44:08",
            "id": 2,
            "elderId": 326,
            "contractName": "李天龙的入住合同",
            "contractNumber": "HT202408261544080001",
            "agreementPath": "https://itheim.oss-cn-beijing.aliyuncs.com/0908707d-7ad4-4d30-ad7d-efdb9c946015.jpg",
            "thirdPartyPhone": "13211223311",
            "thirdPartyName": "李彤",
            "elderName": "李天龙",
            "startDate": "2024-08-26 00:00:00",
            "endDate": "2024-09-30 00:00:00",
            "status": 0,
            "signDate": "2024-08-26 00:00:00",
            "sortOrder": 0
        }
    }
}
```

## 智能健康评估

### 列表查询

**接口地址**:`/nursing/healthAssessment/list`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 是否必须 | 数据类型 |
|-|-|-|-|
| pageNum | 页码 | true | integer(int32) |
| pageSize | 页码大小 | true | integer(int32) |
| elderName | 老人姓名，模糊查询 | false | string |
| idCard | 身份证号，精确查询 | false | string |
| admissionStatus | 入住情况(0:已入住，1:未入住) | false | integer(int32) |

**响应示例：**

```JSON
{
    "total": 2,
    "rows": [
        {
            "createTime": "2024-08-30 15:14:57",
            "id": 5,
            "elderName": "李爱国",
            "idCard": "130123196712131234",
            "healthScore": "85.0",
            "suggestionForAdmission": 0,
            "nursingLevelName": "二级护理等级",
            "admissionStatus": 1,
            "totalCheckDate": "2023-10-10 00:00:00",
            "assessmentTime": "2024-08-30 15:14:57"
        }
    ],
    "code": 200,
    "msg": "查询成功"
}
```

### 上传体检报告

**接口地址**:`/nursing/healthAssessment/upload`

**请求方式**:`POST`

**请求数据类型**:`file`

**请求参数**:

| 参数名称 | 参数说明 | 是否必须 | 数据类型 |
|-|-|-|-|
| idCardNo | idCardNo | true | string |
| file | 二进制文件流 | true | pdf |

**响应示例：**

```JSON
{
    "msg": "操作成功",
    "fileName": "https://itheim.oss-cn-beijing.aliyuncs.com/6da2cfae-2300-4cc1-a60e-47a515113829.pdf",
    "code": 200,
    "newFileName": "6da2cfae-2300-4cc1-a60e-47a515113829.pdf",
    "url": "https://itheim.oss-cn-beijing.aliyuncs.com/6da2cfae-2300-4cc1-a60e-47a515113829.pdf",
    "originalFilename": "体检报告-张芳-女-72岁.pdf"
}
```

### 智能评测

**接口地址**:`/nursing/healthAssessment` 

**请求方式**:`POST`

**请求参数**:

```JSON
{
    "elderName":"张三",
    "idCard":"130123196712131234",
    "physicalExamInstitution":"黑马体检",
    "physicalReportUrl":"https://itheim.oss-cn-beijing.aliyuncs.com/6da2cfae-2300-4cc1-a60e-47a515113829.pdf"
}
```

**响应示例：**

```JSON
{"msg":"操作成功","code":200, "data": 7}
```



> 注意：返回数据中的data为新增**这条数据的ID**

### 查看详情

**接口地址**:`/nursing/healthAssessment/{id}`

**请求方式**:`GET`

**请求参数**:

| 参数名称 | 参数说明 | 是否必须 |
|-|-|-|
| id | 健康评估ID | true |

**响应示例：**

```JSON
{
    "msg": "操作成功",
    "code": 200,
    "data": {
        "createBy": "1",
        "createTime": "2024-08-30 15:14:57",
        "id": 5,
        "elderName": "李爱国",
        "idCard": "130123196712131234",
        "birthDate": "1967-12-13 00:00:00",
        "age": 56,
        "gender": 1,
        "healthScore": "85.0",
        "riskLevel": "caution",
        "suggestionForAdmission": 0,
        "nursingLevelName": "二级护理等级",
        "admissionStatus": 1,
        "totalCheckDate": "2023-10-10 00:00:00",
        "physicalExamInstitution": "黑马程序员",
        "physicalReportUrl": "https://itheim.oss-cn-beijing.aliyuncs.com/dc6314d8-0b68-414b-8825-053794004f39.pdf",
        "assessmentTime": "2024-08-30 15:14:57",
        "reportSummary": "体检报告中视力、心脏功能、肝脏、胆囊、肾脏及前列腺共6项指标存在异常或潜在风险。综合这些临床指标和数据分析，循环系统、消化系统、泌尿系统存在一定程度的风险，建议定期进行相关检查和调整生活方式以降低风险。",
        "diseaseRisk": "{\"healthy\":70,\"caution\":25,\"risk\":5,\"danger\":0,\"severeDanger\":0}",
        "abnormalAnalysis": "[{\"conclusion\":\"心率略快\",\"examinationItem\":\"内科-心率\",\"result\":\"92\",\"referenceValue\":\"-\",\"unit\":\"次/分\",\"interpret\":\"心率92次/分虽在正常范围内，但相对于老年人来说略偏快，可能增加心脏负担。\",\"advice\":\"建议定期进行心电图检查，保持心境平和，避免剧烈运动，如有胸闷、心悸等症状及时就医。\"},{\"conclusion\":\"偶发早搏\",\"examinationItem\":\"内科-心律\",\"result\":\"偶发早搏\",\"referenceValue\":\"-\",\"unit\":\"-\",\"interpret\":\"早搏是心脏提前跳动的现象，偶发早搏可能与心脏电生理变化、劳累、情绪波动等有关。\",\"advice\":\"注意休息，避免过度劳累和情绪波动，如有频繁早搏或不适，建议及时就医。\"},{\"conclusion\":\"视力下降\",\"examinationItem\":\"眼科-视力\",\"result\":\"左眼0.3，右眼0.4\",\"referenceValue\":\"-\",\"unit\":\"-\",\"interpret\":\"视力下降是老年人常见的眼部问题，与眼球结构变化、眼肌调节能力下降等有关。\",\"advice\":\"建议定期眼科检查，配戴合适的老花镜，注意用眼卫生，避免长时间用眼。\"},{\"conclusion\":\"轻度脂肪肝可能\",\"examinationItem\":\"B超-肝脏\",\"result\":\"实质回声略粗糙\",\"referenceValue\":\"-\",\"unit\":\"-\",\"interpret\":\"轻度脂肪肝可能与饮食、生活习惯、遗传等因素有关，是肝脏脂肪代谢异常的表现。\",\"advice\":\"建议低脂饮食，适量运动，控制体重，定期复查肝脏B超和肝功能。\"},{\"conclusion\":\"慢性胆囊炎可能\",\"examinationItem\":\"B超-胆囊\",\"result\":\"胆囊壁毛糙\",\"referenceValue\":\"-\",\"unit\":\"-\",\"interpret\":\"胆囊壁毛糙可能是胆囊炎的早期表现，与胆汁淤积、细菌感染等因素有关。\",\"advice\":\"建议低脂饮食，避免暴饮暴食，如有腹痛、恶心等症状及时就医。\"},{\"conclusion\":\"右肾小囊肿可能\",\"examinationItem\":\"B超-肾脏\",\"result\":\"右肾下极见一大小约5mm的无回声区\",\"referenceValue\":\"-\",\"unit\":\"-\",\"interpret\":\"肾囊肿是一种常见的肾脏良性病变，小囊肿一般无症状，大囊肿可能压迫肾组织。\",\"advice\":\"定期复查肾脏B超，观察囊肿变化，如有腰痛、血尿等症状及时就医。\"},{\"conclusion\":\"前列腺形态略增大\",\"examinationItem\":\"B超-前列腺\",\"result\":\"形态略增大\",\"referenceValue\":\"-\",\"unit\":\"-\",\"interpret\":\"前列腺形态增大可能与前列腺增生有关，是老年男性常见疾病。\",\"advice\":\"建议进行PSA检查以排除前列腺肿瘤，如有尿频、尿急、尿痛等症状及时就医。\"}]",
        "systemScore": "{\"breathingSystem\":100,\"digestiveSystem\":90,\"endocrineSystem\":100,\"immuneSystem\":100,\"circulatorySystem\":90,\"urinarySystem\":95,\"motionSystem\":100,\"senseSystem\":95}"
    }
}
```

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第二章-入住办理+AI大模型提升业务能力 / 入退管理-接口文档 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/UheAwCbzZiBci3kwr7ncp7d7npb)
