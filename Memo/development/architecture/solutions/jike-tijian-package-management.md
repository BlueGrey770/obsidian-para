---
type: memo
date: 2026-11-06
aliases:
  - "ji-ke-ti-jian-tao-can-guan-li"
  - "即刻体检-套餐管理"
---

# 即刻体检-套餐管理

## 4.4体检组管理相关接口

### 4.4.1管理端--体检组相关接口

#### 1）新增检查组

接口名称：新增检查组

接口路径：POST   /health/admin/check-group

![equebk4lkopzrnxpci0cnacenoe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/equebk4lkopzrnxpci0cnacenoe.png)
> 图注：这是新增检查组接口的请求参数说明，分为Body参数和示例两部分。Body参数的格式为application/json，包含三个必填项：groupDescription为说明文本，要求长度在0到100字符之间；groupName为检查组名称，要求长度在0到20字符之间；groupSortNum为int32类型的排序序号，要求数值大于等于1。右侧的示例部分用JSON格式展示了参数的具体内容，包含三个键，分别对应上述三个必填参数。

![thgubko5xofwpoxqksycj0znn8b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/thgubko5xofwpoxqksycj0znn8b.png)
> 图注：图片展示了新增检查组接口的返回响应示例。上方有成功（200）、没有权限（401）、禁止访问（403）、记录不存在（404）等状态码分类。中间部分显示HTTP状态码200，内容格式为JSON，示例为“OK”。右侧数据结构部分，code为200，msg为“OK”，data为空。该图片与文档中新增检查组接口内容相关，直观呈现了接口成功返回的响应格式。

#### 2）分页查询检查组

接口名称：分页查询检查组

接口路径：GET   /health/admin/check-group/pageQuery

<callout emoji="🔮">
参数说明：本接口的排序参数，前端发起请求时固定为
- orderBy1：groupSortNum
- orderBy2：createTime
- isAsc1：true
- isAsc2：false
</callout>

![qmkkbfzo5oo2qjxlerpcw4xjnte](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qmkkbfzo5oo2qjxlerpcw4xjnte.png)
> 图注：这张图片是“分页查询检查组”接口的请求参数说明表，展示了该接口的Query参数详情。其中明确标注了orderBy1对应“groupSortNum”、orderBy2对应“createTime”，以及isAsc1为true、isAsc2为false，这些排序规则与上下文提及的接口参数要求匹配。表中还列出了isAsc1、isAsc2、name、orderBy1、orderBy2、pageNo、pageSize等参数，各参数分别标注了数据类型与是否为可选参数，可用于前端调用该接口时填写对应的请求参数。

![c3xobhaalo8dcdxbwtnc0mzjnwf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/c3xobhaalo8dcdxbwtnc0mzjnwf.png)
> 图注：图片展示了分页查询检查组接口的返回响应示例。上方列出成功（200）、无权限（401）、禁止访问（403）、记录不存在（404）等状态码及对应信息。中间部分显示HTTP状态码200、内容格式JSON等信息。右侧示例部分以JSON格式呈现数据结构，包含code、msg、data等字段，其中data下有list、pages、total等子字段，示例数据展示了检查组相关信息。该图片与上文分页查询检查组接口内容相关，直观呈现了接口返回数据格式。

#### 3）根据id查询检查组

接口名称：根据id查询检查组

接口路径：GET   /health/admin/check-group/{id}

![uko4bm8mwonln9x6l3cc0xgfn1b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/uko4bm8mwonln9x6l3cc0xgfn1b.png)
> 图注：这张图片是即刻体检管理端体检组相关接口中，“根据id查询检查组”接口的请求参数说明内容，展示了该接口所需的Path参数信息。图片明确标注了该请求参数的类型为integer（int64）类型的id，且该参数为必填项，用于对应接口路径中的id标识，可结合上下文接口路径“GET/health/admin/check-group/{id}”理解，该参数是调用此查询接口时必须携带的路径参数。

![ywipbaitwooydzx8fkncdnr2nth](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ywipbaitwooydzx8fkncdnr2nth.png)
> 图注：图片展示了体检组管理相关接口中“分页查询检查组”接口的返回响应示例。上方显示HTTP状态码200，内容格式为JSON，示例为OK。下方数据结构中，code为200，msg为“OK”，data包含createTime、groupDescription、groupName、groupSortNum、groupStatus、id等字段，示例数据对应各字段值。该图片与文档中介绍的分页查询检查组接口内容相关，直观呈现了接口返回的数据结构及示例数据。

#### 4）修改检查组

接口名称：修改检查组

接口路径：PUT   /health/admin/check-group/{id}

![atdvbdtitomzljxadbtcfo6knmf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/atdvbdtitomzljxadbtcfo6knmf.png)
> 图注：这张图片展示了“根据id修改检查组”接口对应的请求参数信息。页面分为Path参数和Body参数两部分，Path参数包含必填的id，类型为integer<int64>，对应路径参数id。Body参数以application/json格式呈现，包含三个必填参数，分别为groupDescription（检查组描述，字符长度需大于0小于等于100）、groupName（检查组名称，字符长度需大于0小于等于26）、groupSortNum（排序序号，数值需大于等于1），同时还给出了对应的请求参数示例。

![e2fpb8bhoo75dsxqcubcfhnjncf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/e2fpb8bhoo75dsxqcubcfhnjncf.png)
> 图注：图片展示了体检组管理相关接口中新增检查组、分页查询检查组、根据id查询检查组、修改检查组、根据id启用检查组、校验检查组是否可以禁用等接口的返回响应示例。其中，新增检查组、分页查询检查组、根据id查询检查组、修改检查组、根据id启用检查组接口的返回响应示例均为HTTP状态码200，内容格式JSON，示例数据为“{“code”: 200, “msg”: “OK”, “data”: “”}”。校验检查组是否可以禁用接口的返回响应示例为HTTP状态码200，内容格式JSON，示例数据为“{“code”: 200, “msg”: “OK”, “data”: {“canDisable”: true}}”。

#### 5）根据id启用检查组

接口名称：根据id启用检查组

接口路径：PUT   /health/admin/check-group/enable/{id}

![na45bxramod9xbx2aozcey5jnjc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/na45bxramod9xbx2aozcey5jnjc.png)
> 图注：这张图片展示的是体检组相关接口的请求参数信息，对应文档中“根据id启用检查组”接口的请求参数部分。页面显示Path参数要求，其中参数“id”的类型为integer且为int64格式，该参数标注为“必需”，是发起对应接口请求时必须携带的Path参数，用于标识需要启用的检查组的唯一身份ID。

![imrrbnhutoh10cxguhjcylxdndg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/imrrbnhutoh10cxguhjcylxdndg.png)
> 图注：这张图片展示的是一个接口的返回响应说明，顶部标注了不同的响应状态及对应状态码，包括成功(200)、成功(201)、没有权限(401)、禁止访问(403)、记录不存在(404)，同时说明该响应的HTTP状态码为200，内容格式为JSON。图中还明确了该接口返回的数据结构，包含三个可选字段：code为int32类型的状态码，值为200代表成功，其余值代表失败；msg为响应消息，值为OK；data为响应数据，示例值为空字符串。右侧同步展示了对应的数据结构示例，完整呈现了返回响应的规范格式。

#### 6）校验检查组是否可以禁用

接口名称：校验检查组是否可以禁用

接口路径：GET   /health/admin/check-group/checkDisable/{id}

<callout emoji="🔮">
需求说明：本接口用于用户禁用体检组时，提前校验进行弹窗提示。
逻辑说明：需要校验该体检组关联的套餐有哪些，如果有关联的套餐，返回套餐名称列表，并将canDisable参数置为false；否则套餐名称列表为null，canDisable参数置为true；
</callout>

![jd0lbd8bzo6kejx9gzpcaywxnuc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/jd0lbd8bzo6kejx9gzpcaywxnuc.png)
> 图注：图片展示了“根据id查询检查组”接口的请求参数部分。标题为“请求参数”，下方有“Path 参数”说明。参数“id”为integer类型，数据类型为int64，标识为id，标注为“必需”。该图片与文档中“根据id查询检查组”接口内容相关，直观呈现了接口请求时所需传递的必填参数信息，帮助开发者明确接口调用时的参数要求。

![xes7bun4doij5dxkcavcioz9n6g](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xes7bun4doij5dxkcavcioz9n6g.png)
> 图注：图片展示了新增检查组接口的返回响应示例。HTTP状态码为200，内容格式为JSON，示例值为“OK”。数据结构中，code为200，msg为“OK”，data包含canDisable（是否可以禁用）和setmealNames（套餐名称列表）两个字段，示例值分别为true和“string”。该图片与上文新增检查组接口相关，直观呈现了接口成功返回的数据格式及示例内容，帮助理解接口响应情况。

#### 7）根据id禁用检查组

接口名称：根据id禁用检查组

接口路径：PUT   /health/admin/check-group/disable/{id}

<callout emoji="🔮"><p>说明：业务逻辑请参考<cite doc-id="FevNwiL4lizEtWkGxd7cx8Snn7g" file-type="wiki" title="即刻体检需求文档" type="doc"></cite></p></callout>

![euvsbxnmborhqixigfvcrikcndd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/euvsbxnmborhqixigfvcrikcndd.png)
> 图注：这张图片展示的是接口的请求参数部分内容，用于说明对应接口的入项要求。其中明确标注了Path参数包含一个必填的id参数，该参数的类型为integer，具体为int64类型，且被标记为“必需”，结合上下文可知，这是对应“根据id删除检查组”接口的Path参数要求，说明调用该删除检查组的接口时，需要在路径中携带必填的int64类型的id参数。

![lfi4bdtfeofv0nxubg1cpsdvnsd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/lfi4bdtfeofv0nxubg1cpsdvnsd.png)
> 图注：这张图片展示的是一个API接口的返回响应相关文档内容，属于体检组相关接口的返回说明部分。图片中明确标注了不同的响应状态码，包括成功的200、201，以及失败类的无权限401、禁止访问403、记录不存在404。同时呈现了该接口的HTTP状态码为200，内容格式为JSON，其数据结构包含可选的code（状态码）、msg（响应消息）、data（响应数据）三个字段，还给出了对应的数据结构示例，示例中的code值为200、msg为"OK"、data为空字符串。

#### 8）根据id删除检查组

接口名称：根据id删除检查组

接口路径：DELETE   /health/admin/check-group/{id}

![mn01b7occoepsixpsnfcyfsqn2b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mn01b7occoepsixpsnfcyfsqn2b.png)
> 图注：图片展示了“体检组下拉列表”接口的请求参数部分。标题为“请求参数”，图片中“Path 参数”下，有一个参数“id”，类型为integer（int64），标识为“id”，并标注为“必需”。该图片与文档中“体检组下拉列表”接口的上下文相关，是对接口中Path参数部分的说明，用于体检项新增页面查询启用状态的体检组时，明确id参数的类型及必填要求。

![c90eboqiooohboxifydcbq6jnud](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/c90eboqiooohboxifydcbq6jnud.png)
> 图注：图片展示了“根据id删除检查组”接口的返回响应信息。HTTP状态码为200，内容格式为JSON，示例值为“OK”。数据结构中，code为200，msg为“OK”，data为空。该图片与上文介绍的“根据id删除检查组”接口相关，直观呈现了接口成功返回的响应状态、格式及示例数据，帮助理解接口调用成功后的具体返回内容。

#### 9）体检组下拉列表

接口名称：体检组下拉列表

接口路径：GET   /health/admin/check-group/listForFilter

<callout emoji="🔮">
说明：本接口用于体检项新增页面，需要查询启用状态的体检组
</callout>

![sn9dbkfidoccdvx0t6wcrcxbnsd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/sn9dbkfidoccdvx0t6wcrcxbnsd.png)
> 图注：该图片展示了即刻体检套餐管理中，体检组下拉列表接口对应的请求参数信息。界面显示的Query参数包含groupStatus，该参数为integer类型（int32），属于可选参数，其值对应的含义为：0代表草稿，1代表启用，2代表禁用，此参数用于筛选对应状态的启用体检组，为体检项新增页面提供符合要求的体检组列表数据。

![mhwxbkzcpocyh4xaldxc59jznoc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mhwxbkzcpocyh4xaldxc59jznoc.png)
> 图注：这是一张返回响应的示例展示图，呈现了HTTP状态码为200时的响应相关信息。图中上方标注了接口常见的返回状态码说明，包括成功(200)、没有权限(401)、禁止访问(403)、记录不存在(404)，当前示例对应的是状态码200，内容格式为JSON。图中还详细列出了响应的数据结构，包含可选的code、msg、data字段，其中data为数组类型，数组内是检查组信息，包含检查组名称（groupName）、检查组id（id）的字段说明，并给出了对应的JSON格式示例，与文档中体检组下拉列表接口的响应要求相关。

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 即刻体检-套餐管理 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/QWEcwAmGXi4DickPbI0cGuW6nag)
