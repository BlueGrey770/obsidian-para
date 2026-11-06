---
type: memo
date: 2026-11-06
aliases:
  - "问题汇总"
---

<title>问题汇总</title>

# FAQ

## 购物车查询上商品无数据

排查下UserContext的getUser方法是否有修改，返回默认值1L,如下图

![图片展示的是UserContext类代码，位于com.hmall.common.utils包下。关键部分是getUser()方法，其代码为“return tl.get();”，但被红色框线突出显示。上下文提到排查UserContext的getUser方法是否有修改，返回默认值1L，此图片正是对应代码示例，直观呈现了代码中getUser方法的实现情况，与排查内容紧密相关，帮助理解代码逻辑。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/Mw4TbwXjZo0K9xxWEcScq2Nkn6L.png)

## API注解没有生效，可能是没有识别到hm-common

具体服务下的pom文件里，删除下hm-common依赖，刷新下maven，然后再重新引入下hm-common，再刷新下maven。



## 引入ItemClient后服务启动报错

![图片展示的是一个调试界面，显示了ItemServiceApplication的堆栈信息。关键信息是红框突出显示的“Caused by: java.lang.IllegalStateException: Method ItemClient#queryItemByIds(Collection) not annotated with HTTP method type (ex. GET, POST)”，表明ItemClient#queryItemByIds方法未标注HTTP方法类型。该图片与文档中“引入ItemClient后服务启动报错”内容相关，是报错的具体原因呈现，帮助排查服务启动时的异常问题。](../../../../../../Attachments/development/architecture/microservices/fu-wu-zhu-ce-yu-fa-xian/01-wei-fu-wu-fu-wu-zhu-ce-yu-fa-xian/SdTLbYJGYoK68BxhB5Cc9HPtnHc.png)

检查下是不是ItemClient类写的不对，比如缺少注解信息，参考如下：

```Java
package com.hmall.api.item;

import com.hmall.api.item.dto.ItemDTO;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.util.Collection;
import java.util.List;

@FeignClient(name = "item-service",path = "/items")
public interface ItemClient {
    @GetMapping
    public List<ItemDTO> queryItemByIds(@RequestParam("ids") Collection<Long> ids);
}
```

---
> 来源：飞书技术知识库（脱敏版） · 04 微服务与中间件 / 服务注册与发现 / 01 微服务服务注册与发现 / 问题汇总 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/XwRgwofuci7JnFk9NmAcBgS8nHf)
