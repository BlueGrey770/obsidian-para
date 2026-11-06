---
type: memo
date: 2026-11-06
aliases:
  - "第十章 秒杀抢购"
---

<title>第十章 秒杀抢购</title>

# 学习目标：

能够说出秒杀抢购业务常用的技术

能够说出活动查询的缓存是怎么做的

能够实现活动定时预热程序

能够实现活动查询接口

能够说出解决超卖问题的常用技术方案

能够说出本项目保证Redis原子操作的方案

能够说出抢券业务的整体方案

能够完成抢券接口开发

能够说出抢券结果异步同步的方案

能够开发抢券结果异步同步功能

能够说出家政项目抢单模块的设计方案

能够完成抢单接口开发





# 1 秒杀抢购业务介绍

## 1）秒杀抢购业务特点

秒杀抢购是电商平台的一种业务模式，它可以聚焦流量和吸引注意力，通过秒杀活动长时间吸引人群，最终吸引用户下单。

秒杀抢购业务有哪些特点呢？

**时间限制：** 秒杀抢购活动通常在特定的时间段内进行，例如1小时或更短的时间。在这个时间段内，消费者可以购买特定商品或服务，通常是限量销售。

**限量销售：** 秒杀抢购商品通常数量有限，销售数量是提前确定的。一旦库存售罄，抢购活动就结束，未能购买的消费者需要等待下一次活动。

**价格优惠：** 秒杀抢购商品通常以折扣价格销售，价格较平时低廉。这种价格优势是吸引消费者参与抢购的重要因素。

**高并发和服务器压力：** 抢购开始时大量用户会同时访问在线商城，导致网站服务器承受巨大压力。因此，网站的服务器和网络基础设施需要具备高并发处理能力，以应对瞬时大量的用户请求。

**技术要求高：** 秒杀抢购业务对技术要求非常高，包括网站性能优化、数据库优化、缓存技术、负载均衡等方面的技术应用。

如今秒杀抢购不仅应用在电商平台，随着互联网用户的增加在餐饮、旅游、娱乐等很多领域都有应用，比如：12306抢票、抢优惠券、骑手抢单、酒店预定等。

## 2）常用技术方案

实现秒杀抢购业务会用到哪些技术呢？可以参考行业上一些成熟的解决方案。

1. **缓存方案**

使用缓存技术（如Redis）来存储热点数据，例如商品信息和库存数量。这样可以减轻数据库的压力，提高读取数据的速度。

1. **异步处理方案**

当用户成功秒杀后，将抢购信息发送到队列，然后由消费者多线程异步处理订单，减轻系统的实时压力，使用Redis、RabbitMQ等技术都可以实现队列。

1. **防止超卖方案**

超卖是最终下单购买数量大于库存数量，比如：库存１00个用户最终购买了101个，多出这一个就是超卖了，在秒杀抢购业务中这也是需要解决的问题，可以使用分布式锁、Redis等技术都可以防止超卖。

1. **限流与防刷方案**

使用限流算法（如令牌桶、漏桶算法）来控制请求的并发数，防止服务器被过多请求压垮。可以在服务端使用限流技术，比如：sentinel、nginx、验证码等技术。

1. **数据库优化方案**

对数据库进行优化，包括索引的设计、SQL语句的优化、数据库连接池的使用等，以提高数据库的查询和更新速度。

1. **数据库分库分表方案**

在数据库层面进行分库分表，将数据分散存储在不同的数据库实例或表中，提高数据库的读写性能。

1. **负载均衡**

使用负载均衡技术，例如Nginx、Spring Cloud Gateway等，将请求分发到多个服务器上，增加系统的处理能力。

1. **CDN加速**

CDN（Content DeliveryNetwork）即内容分发网络，CDN用于加速静态资源的访问，将内容分发到CDN节点就近为客户提供服务。

如下图所示，通过CDN用户访问就近节点，提高访问速度。

![这张图是CDN内容分发网络的架构示意图，清晰展示了CDN的工作逻辑。左侧是提供内容的源站，中间为CDN平台，该平台设置了新疆、成都、广州等不同地域的节点，右侧则对应新疆、四川、广东等地的终端用户。用户访问时，会先连接到就近的CDN节点，由这些节点直接为用户提供服务，无需从源站获取内容，以此实现静态资源的就近访问，从而提升用户的访问速度，这也与上下文中CDN用于加速静态资源访问的介绍相呼应。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Ykf3bx9CpogShmx2ibMcdE9WnDb.png)

1. **安全性处理**

确保系统的安全性，防止SQL注入、XSS攻击（跨站脚本攻击）等，同时在后端实现防刷、验证码等安全措施，保护系统免受恶意攻击。

## 3）本章目标

通过抢券、抢单两个模块的学习掌握秒杀抢购业务的技术方案。

## 4）小结

秒杀抢购常用的技术方案有哪些？



# 2 抢券

## 2.1 需求分析

用户进行抢券经过三个过程：

### 1）**抢券界面**

用户进入抢券界面如下图。

“疯抢中”界面展示进行中还未到结束的优惠券活动，按照开始时间升序排列，不进行分页。

“即将开始”界面是展示待发放的优惠券活动，按照开始时间升序排列，不进行分页。

![这张抢券界面的图片展示了两个页面内容，左侧为“疯抢中” tab 页（标注为1），该页按照开始时间升序排列了正在进行的优惠券活动：包含“618大促活动”（满200元立减20元，活动时间2023.8.16-2023.8.25），其右侧有红色的“立即领取”按钮（标注为2），还有“平台促销活动”满200元立享5折且已抢光的活动信息；右侧为“即将开始” tab 页（标注为3），展示待发放的优惠券活动，同样列出了和左侧一致的两款活动，但该页面没有“立即领取”按钮，呈现对应活动的内容与状态。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/AQA0buXNiolqqlxtTRucDZjNnFg.png)

在抢券界面显示如下信息：

1、优惠券活动名称

2、优惠券满减或折扣信息

3、活动的起止时间

4、是否抢光。

5、活动的状态。

用户点击立即领取进行抢券，以下情况无法抢券：

1、已抢光

“已抢光”表示活动发放的总数量已领取完，该优惠券的领取数量等于发放数量。

2、未到开始时间或已到结束时间

3、用户已领（每个用户限领一张）

4、活动已撤销



### 2）**进行抢券**

优惠券到达发放时间后，用户点击“立即领取”进行抢券。

![图片展示的是618大促活动的抢券界面。上方显示“抢券”字样，左侧有“即将开始”的文字，右侧有“...”和“@”图标。下方有两张优惠券信息。上方优惠券为“618大促活动”，满200元立减20元，发放时间为2023.8.16 - 2023.8.25，有“立即领取”按钮，按钮上标注“2”；下方优惠券为“平台促销活动”，满200元立享5折，发放时间为2023.8.16 - 2023.8.25，其“已抢光”按钮上标注“1”。该图与上下文介绍的用户点击“立即领取”进行抢券，以及“已抢光”表示活动发放的总数量已领取完等内容相关。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/JPpqbLszwo952CxwRFJcyNu5n14.png)

抢券成功提示“领取成功”，如果领取失败会提示失败信息。



### 3）**我的优惠券列表**

用户抢券成功在我的优惠券列表查看。

用户进入【我的】-【优惠券】查看已抢到的优惠券，按抢券时间降序显示当前用户抢到的优惠券。

![这是微信端应用的“我的”页面截图，页面顶部显示微信用户头像与昵称，底部导航栏的“我的”选项被红色高亮标识，当前该页面的核心功能入口处，有红色箭头指向“我的优惠券”选项。页面还设有我的订单模块，包含待支付、待服务、服务中、待评价四个功能入口，另有我的地址、我的评价模块，底部标注客服电话400-000-4000。该图片对应文档中用户进入【我的】-【优惠券】查看已抢优惠券的操作指引示例。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Y956bds07oRZX9xfpaTcFT6Enwh.png)

本查询为滚动查询，向上拖屏幕查询下一屏，一屏显示10条。

如下图：

![图片展示了用户在“我的”-“优惠券”页面查看已抢到的优惠券情况。左侧显示“未使用”状态，有20元和5折优惠券；中间为“已使用”状态，有20元优惠券；右侧为“已过期”状态，也有20元和5折优惠券。每种状态下的优惠券信息包括面额、活动名称、使用条件及有效期。该图与上下文对应，直观呈现了用户查看优惠券时的三种状态及对应优惠券信息。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/R6gibdbpjoc1PfxdNzUcFwW7nsf.png)

对优惠券的三个状态说明如下：

未使用：未过有效期的优惠券。

优惠券的有效期：从领取优惠券的时间加上优惠券的使用期限（“使用期限”在优惠券活动管理界面进行设置）。

已使用：已经在订单中使用的优惠券。

已过期：未使用且已过有效期的优惠券，已过期的优惠券将无法使用。

### 4）**小结**

根据需求分析抢券模块包括三部分的内容：

**活动查询**

进入抢券页面查询优惠券活动信息。

**抢券**

点击抢券按钮进行抢券。

**我的优惠券**

抢券成功进入我的优惠券页面查询抢到的优惠券。

下边按顺序开发每一部分。



## 2.2 活动查询

### 2.2.1 系统设计

#### 1）活动查询分析

下边根据需求分析对抢券界面的活动查询功能进行分析与设计，如下图：

![这张抢券页面的对比图，左图为当前活动状态页面，顶部标注①“疯抢中”的活动状态，展示了两个活动：一是618大促活动，售价20元、满200元立减20元，活动时间为2023.8.16-2023.8.25，标注有②“立即领取”按钮；二是平台促销活动，标注“已抢光”。右图为切换后的页面，顶部切换为标注③“即将开始”的活动状态，仅展示了和左图一致的两个活动的信息。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/T2hoblWrRoELSLxVXkUcXJjfnNg.png)

1. **活动查询界面显示了哪些数据？**

**活动信息：**

包括两个部分数据：

- 进行中还未到结束的优惠券活动。
- 待开始的优惠券活动。

信息内容包括：

优惠券活动名称

优惠券满减或折扣信息

活动的起止时间

是否抢光（根据库存剩余量判断）

活动的状态。



1. **面向高并发如何提高活动查询性能？**

此部分信息来源于优惠券活动表，由于抢券页面面向C端用户且请求并发量大，如何在高并发下提高活动查询的性能呢?

如果直接查询数据库无法满足需求并且对数据库造成巨大的压力从而影响其它功能使用数据库，我们可以使用缓存，将优惠券活动信息存入缓存，比如Redis，从Redis查询避免查询数据库。

<whiteboard token="G1MAw7pGMhAy9ibhlQ4cXvHWnZd"></whiteboard>

1. **如何保证缓存一致性？**

通过定时预热程序保证缓存一致性，抢券页面列出的活动信息属于热点信息，对于热点信息通过定时预热防止缓存击穿，定时预热程序通过定时任务定时执行，定时将活动信息存入Redis。

<whiteboard token="Cxt2wE7xZhsAKsbkhBBccQBfndg"></whiteboard>



#### 2）**数据流**

根据以上分析设计数据流如下：

<whiteboard token="Yc2Ew0tMdh5KBhbp7JjcH0ganJh"></whiteboard>

活动管理：运营人员进行优惠券活动管理，对活动表进行增删改查操作。

活动状态更新任务：根据活动的开始和结束时间更新活动状态。

活动预热任务：定时查询活动表信息存入Redis

抢券查询：从Redis查询活动预热信息。



#### 3）**Redis数据结构设计**

活动预热任务定时查询活动表信息存入Redis，下边设计活动信息的缓存结构：

根据需求可知在抢券查询界面需要的数据全部来源于活动表，因为要预热的活动信息内容有限，我们可以将要预热的活动信息转为json串存入redis，活动查询程序读取json串也方便进行解析。

活动信息缓存结构设计如下：

redis结构：String类型

key: "ACTIVITY:LIST"

value: 符合条件的优惠券活动列表JSON数据。

过期时间：永不过期

缓存一致性方案：通过预热程序保证缓存一致性



#### 4）如何保证活动状态自动改变

在活动查询界面如何保证活动状态实时改变，当到达活动开始时间活动状态变为“进行中”，在“疯抢中”界面显示，当活动结束在“疯抢中”界面将无法查询到。

关于活动状态的改变，在“优惠券活动管理实战”中完成了通过定时任务自动更新活动的状态，实现了如下需求：

1）对待生效的活动更新为进行中

到达发放开始时间状态改为“进行中”。

2）对待生效及进行中的活动更新为已失效

到达发放结束时间状态改为“已失效”

具体代码如下：

**Service接口实现**

```Java
@Override
public void updateStatus() {
    LocalDateTime now = DateUtils.now();
    // 1.更新已经进行中的状态
    lambdaUpdate()
            .set(Activity::getStatus, ActivityStatusEnum.DISTRIBUTING.getStatus())//更新活动状态为进行中
            .eq(Activity::getStatus, NO_DISTRIBUTE.getStatus())//检索待生效的活动
            .le(Activity::getDistributeStartTime, now)//活动开始时间小于等于当前时间
            .gt(Activity::getDistributeEndTime,now)//活动结束时间大于当前时间
            .update();
    // 2.更新已经结束的
    lambdaUpdate()
            .set(Activity::getStatus, LOSE_EFFICACY.getStatus())//更新活动状态为已失效
            .in(Activity::getStatus, Arrays.asList(DISTRIBUTING.getStatus(), NO_DISTRIBUTE.getStatus()))//检索待生效及进行中的活动
            .lt(Activity::getDistributeEndTime, now)//活动结束时间小于当前时间
            .update();
}
```

定时任务方法：

```Java
@Component
public class XxlJobHandler {

/**
 * 活动状态修改，
 * 1.活动进行中状态修改
 * 2.活动已失效状态修改
 * 1分钟一次
 */
@XxlJob("updateActivityStatus")
public void updateActivitySatus(){
    log.info("定时修改活动状态...");
    try {
        activityService.updateStatus();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
...
```



通过上边的定时任务无法实现状态的实时改变，每分钟执行一次状态变更理论上存在最多1分钟的状态变更延迟。

如何实现在页面到达活动开始时间立即变更活动状态？

1. 在前端进行控制，根据活动开始时间进行倒计时，达到开始时间将活动移到进行中界面。
2. 请求后端查询数据，根据当前时间和活动开始、活动结束时间判断活动的状态。

当活动开始时间小于等于当前时间并且结束时间大于当前时间说明活动已经开始并且还没有结束，活动状态为进行中。

当活动结束时间小于当前时间说明活动结束，活动状态为失效。



#### 5）小结

抢券查询的缓存是怎么做的？



如何实现活动状态的自动改变？

1. 通过定时任务每几分钟更新活动状态

对于待生效的活动，当活动开始时间小于等于当前时间并且结束时间大于当前时间，将活动状态更新为进行中。

对于待生效和进行中的活动，当活动结束时间小于当前时间，将活动状态更新为失效。

如何实现状态变化的实时性，当到达活动开始时间状态立即变化：

两种方法结合：

1. 前端请求后端接口查询活动信息，后端根据活动时间（开始、结束时间）及当前时间得到活动当前准确的状态，并将状态返回给前端。
2. 前端进行控制，根据活动开始时间进行倒计时，达到开始时间将活动移到进行中界面。





### 2.2.2 定时预热程序

#### 1）搭建环境

进入jzo2o-market工程，创建dev_03分支，使用下发的jzo2o-market-dev03.zip代码作为dev_03分钟的基础代码。



#### 2）明确需求

根据需求，预热的活动信息包括两个部分：

进行中还未到结束的优惠券活动。

待开始的优惠券活动。



#### 3) 编写Service方法

下边定义Service接口从数据库查询要预热的活动信息，存入redis。

代码如下：

```Java
package com.jzo2o.market.service;

import com.jzo2o.common.model.PageResult;
import com.jzo2o.market.model.domain.Activity;
import com.baomidou.mybatisplus.extension.service.IService;
import com.jzo2o.market.model.dto.request.ActivityQueryForPageReqDTO;
import com.jzo2o.market.model.dto.request.ActivitySaveReqDTO;
import com.jzo2o.market.model.dto.response.ActivityInfoResDTO;
import com.jzo2o.market.model.dto.response.SeizeCouponInfoResDTO;

import java.util.List;

/**
 * <p>
 *  服务类
 * </p>
 *
 * @author itcast
 * @since 2023-09-16
 */
public interface IActivityService extends IService<Activity> {

/**
 * 活动预热
 */
void preHeat();
...
```

Service接口实现：

```Java
@Override
public void preHeat() {
    //当前时间
    LocalDateTime now = DateUtils.now();

    //查询即将开始、进行中还未到结束的优惠券活动
    /**
     select *
     from activity t
     where  t.status in(1,2)
     order by t.distribute_start_time
     */
    List<Activity> list = lambdaQuery()
            .in(Activity::getStatus, Arrays.asList(NO_DISTRIBUTE.getStatus(), DISTRIBUTING.getStatus()))//查询待开始和进行中的
            .orderByAsc(Activity::getDistributeStartTime)
            .list();
    if (CollUtils.isEmpty(list)) {
        //防止缓存穿透
        list = new ArrayList<>();
    }
    // 2.数据转换
    List<SeizeCouponInfoResDTO> seizeCouponInfoResDTOS = BeanUtils.copyToList(list, SeizeCouponInfoResDTO.class);
    String seizeCouponInfoStr = JsonUtils.toJsonStr(seizeCouponInfoResDTOS);

    // 3.活动列表写入缓存
    redisTemplate.opsForValue().set(ACTIVITY_CACHE_LIST, seizeCouponInfoStr);


}
```



#### 4) **定义XXL-Job调度方法**

下边定义定时任务方法，执行定时预热程序。每小时执行一次。

```Java
package com.jzo2o.market.handler;

import com.jzo2o.market.service.IActivityService;
import com.jzo2o.market.service.ICouponService;
import com.jzo2o.redis.annotations.Lock;
import com.jzo2o.redis.constants.RedisSyncQueueConstants;
import com.jzo2o.redis.sync.SyncManager;
import com.xxl.job.core.handler.annotation.XxlJob;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;

import static com.jzo2o.market.constants.RedisConstants.Formatter.*;
import static com.jzo2o.market.constants.RedisConstants.RedisKey.COUPON_SEIZE_SYNC_QUEUE_NAME;

@Component
public class XxlJobHandler {


    @Resource
    private IActivityService activityService;


    /**
     * 活动预热，整点预热
     *
     */
    @XxlJob("activityPreheat")
    public void activityPreHeat() {
        log.info("优惠券活动定时预热...");
        try {
            activityService.preHeat();
        } catch (Exception e) {
           e.printStackTrace();
        }
    }
    ...
```



#### 5) 配置定时任务

下边在XXL-JOB中配置调度方法进行测试：

通常生产中会提前一周去策划活动并在系统中创建活动，所以在生产中每小时执行一次预热程序，开发中为了测试方便5秒执行一次。

![这是一个定时预热任务的配置界面，包含基础配置、调度配置、任务配置、高级配置四大模块内容。基础配置中，执行器为“营销中心”，负责人为“xxjob”，任务描述为“活动定时预热任务”，报警邮件栏提示需输入多个用逗号分隔的报警邮件地址。调度配置中，调度类型选择“CRON”，Cron表达式为“0/5 * * * * ?”。任务配置里，运行模式选“BEAN”，JobHandler为“activityPreheat”，任务参数栏提示输入对应参数。高级配置中，路由策略设为“第一个”，子任务ID栏提示输入子任务ID，调度过期策略为“忽略”，阻塞处理策略为“单机串行”，任务超时时间、失败重试次数均设为0。该界面的配置内容，对应文档中修改活动开始时间后通过程序自动更新活动状态的操作。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/GBH0bUvXcoPnXzxmQKlcONINnWb.png)



#### 6) 预热程序测试

测试流程如下：

首先在运营端创建优惠券活动，创建即将开始的活动

![这张图片展示的是运营端新建优惠券的页面，属于第十章秒杀抢购中抢券模块的2.2.2定时预热程序环节。页面呈现了优惠券创建的基本信息与发放规则，其中“优惠券类型”选中了“满减”选项，活动名称为“双12大促”，满减金额设置为100元，折扣金额为10元，发放时间范围是2023年12月17日至12月24日，发放数量为100，使用期限为10天，页面下方设有“返回”和“提交”按钮，该页面对应上下文里在运营端创建即将生效优惠券活动的操作步骤。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/RqZ8bkpopoQFyXxns95cRxuNnus.png)

创建成功，活动为待生效

![这张图片展示的是优惠券管理页面的内容，页面左上角标注属于营销管理板块的优惠券管理功能区，上方设有活动编号输入框、活动名称输入框、优惠类型下拉框、发放状态下拉框，还有重置和搜索按钮。页面右侧有新增优惠券的红色按钮，下方列表呈现出“双12大促”优惠券的相关信息，包括适用对象为全部用户，发放时间是2023-12-17 00:00至2023-12-24 00:00，使用期限为10天，发放数量100张，每人限领1张，当前发放状态为待生效，创建时间为2023-12-16 17:05，操作栏有撤销、编辑、领取记录选项。该图片对应文档中提到的创建成功、待生效活动的页面展示，用于直观显示运营端创建出的待生效优惠券活动的详情。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/MCMNb9yPhoocb5xUqSFcZs0qneg.png)

再创建一个活动：

![这是在云岗到家系统新建优惠券的页面截图，页面处于“营销管理”下的“优惠券管理”板块，左侧为系统功能导航栏。页面分为“基本信息”和“发放规则”两个区域：基本信息区填写了活动名称为“平台二周年庆”，优惠券类型选择“折扣”，折扣比例设为90%，满减限制为0元；发放规则区设置了活动发放时间为2023年12月17日至2023年12月24日，可领取天数为20天，发放数量为100张。该页面是定时预热程序测试流程中的“再创建一个活动”步骤的操作界面。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/GCyBbDOYZoA6LWxFxN1cjJmHnNh.png)

创建成功，进入jzo2o-market数据库，找到活动表activity中该记录，修改活动开始时间为昨天，如下图：

![图片展示的是jzo2o-market数据库中活动表activity中一条记录的数据。关键信息包括活动名称为“平台二周年庆”，类型为2，开始分发时间为2023 - 12 - 15 00:00:00，结束分发时间为2023 - 12 - 24 00:00:00，状态为2，总数量和库存数量均为100，创建和更新时间为2023 - 12 - 16 17:08:56和17:12:10。该图片与上文提到的在jzo2o - market数据库中找到活动表activity中记录，修改活动开始时间为昨天的内容相关，直观呈现了修改后的活动记录数据。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/S3pMbgsYYoIqT2xGdZScsKMGnnf.png)

通过活动状态自动更新程序，自动将“平台二周年庆”活动状态改为进行中。

测试通过如下图：

![这张图是优惠券管理页面的界面，页面顶部显示路径为“营销管理 > 优惠券管理”。页面上方的搜索栏中，活动名称输入框填写了“平台二周年”，活动状态选择了“进行中”。下方的列表区域展示了对应活动的详细信息：适用对象为全部用户，发放时间为2023年12月15日00:00至2023年12月24日00:00，使用期限为20天，发放数量100张，每人限领1张，发放状态为进行中，创建时间为2023年12月16日17:08，操作栏包含撤销、编辑和领取记录选项，该界面对应文档中预热程序测试的通过环节，用于验证活动状态自动更新后的实际展示效果。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/N4oWbIRXdokqkfxivOXcwKtun1g.png)

通过定时预热程序即将开始的活动及进行中的活动写入redis：

根据以上数据范围判断存入到redis的活动信息是否正确。

预热成功在redis成功存储上边添加的两个活动：

![该图片展示的是jzo2o-market数据库ACTIVITY:LIST键的相关内容，界面显示了服务器地址，键值大小为489字节，当前处于JSON查看状态，其中以键值对形式存储了两条活动数据，分别是“平台二周年庆”和“双12大促”的活动信息，涵盖折扣率、类型、分发结束与开始时间、总数量、名称、库存、金额条件、活动ID及状态等内容，对应预热测试中成功存入redis的两个活动的相关信息，用于验证预热程序的执行结果。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/YhfYbMaHsoK5UrxU3A7cBSlGnbe.png)



#### 7）小结 

活动定时预热程序是怎么做的？

1. 首先说明缓存方案 （参考问题：抢券查询的缓存是怎么做的？）
2. 使用xxl-job定时执行活动预热程序，从数据库查询出符合条件的活动信息存储到redis中。



### 2.2.3 活动查询接口

#### 1）接口分析

根据界面原型分析接口：

![这张图片是抢券功能的活动查询界面，分为左右两个部分，对应活动查询的两个tab选项。左侧界面突出显示“疯抢中”（标注序号1）tab，展示了两类可查询的活动信息，第一类是618大促满减活动，标注了价格20元，带有红色的“立即领取”按钮（标注序号2），第二类是平台促销折扣活动，标注了“已抢光”；右侧界面突出显示“即将开始”（标注序号3）tab，未展示额外的功能按钮，两组界面的活动信息内容一致，分别对应了前端传入参数为查询进行中或即将开始活动的两种场景。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/OCG6boWnboqsvpxXFvicBr6anSg.png)

界面有两个tab，疯抢中和即将开始，前端传入后端一个参数标记是查询进行中的活动还是即将开始的活动。

后端需要给前端返回以下数据：

1. 活动id
2. 活动名称
3. 优惠券类型，1：满减，2：折扣
4. 满减限额，0：表示无门槛，其他值：最低消费金额
5. 折扣率，折扣类型的折扣率，例如：8,打8折
6. 优惠金额，满减或无门槛的优惠金额
7. 发放开始时间
8. 发放结束时间
9. 活动状态，1：待生效，2：进行中，3：已失效
10. 优惠券剩余数量



这些信息在预热的活动信息缓存中都存在，但是有两个字段不够实时：活动状态，优惠券剩余数量。

活动状态：通过定时任务更新活动状态，写入活动表，定时预热程序读出活动信息存储到Redis，由于是通过定时任务更新活动状态、定时预热程序更新活动信息缓存，最终从redis取出的活动状态于实际的活动状态存在延迟，导致实际活动的状态与页面显示的状态不一致。

举例：活动开始时间为8点钟，当前时间已过了8点钟，查看抢券页面活动状态仍然显示“即将开始”。

优惠券剩余数量：在抢券模块会操作此字段，这里暂不考虑此字段。



如何解决活动状态延迟问题？

请求后端查询数据，根据当前时间和活动开始、活动结束时间判断活动的状态。

当活动开始时间小于等于当前时间并且结束时间大于当前时间说明活动已经开始并且还没有结束，活动状态为进行中。

当活动结束时间小于当前时间说明活动结束，活动状态为失效。



#### 2）接口定义

接口名称：用户端抢券列表分页接口

接口路径：**GET**/market/consumer/activity/list

**请求数据类型** application/x-www-form-urlencoded

![该图片展示了用户端抢券列表分页接口的请求参数信息，对应文档中活动查询接口的相关内容。图片呈现了表格形式的参数说明，其中涉及的核心参数为tabType，该参数是页面tab类型，1代表疯抢中，2代表即将开始。该参数的请求类型为query，是请求接口时必须填写的内容，数据类型为integer类型下的int32。此图片补充说明了该活动查询接口所需的请求参数详情，是文档中接口定义部分的具体参数示例。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NE4Gb1gaHo3HxxxMTrLcDZ74n5b.png)

![图片展示了用户端抢券列表分页接口的响应参数。参数包括msg（响应消息）、code（状态码）、data（响应数据，包含discountRate、remainNum、discountAmount、type、distributeEndTime、name、amountCondition、id、distributeStartTime、status等信息）。其中，type为优惠券类型，1代表满减，2代表折扣；status为优惠券配置状态，1为待生效，2为进行中，3为已失效。该图片与上文接口定义部分对应，直观呈现了接口响应数据的结构及各参数含义。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/K3QSbg6VJo1Rvwxi9OycFrhcnpk.png)



定义controller方法：

```Java
package com.jzo2o.market.controller.consumer;

import com.jzo2o.common.model.PageResult;
import com.jzo2o.market.model.dto.request.ActivityQueryForPageReqDTO;
import com.jzo2o.market.model.dto.request.ActivitySaveReqDTO;
import com.jzo2o.market.model.dto.response.ActivityInfoResDTO;
import com.jzo2o.market.model.dto.response.SeizeCouponInfoResDTO;
import com.jzo2o.market.service.IActivityService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;
import java.util.List;

@RestController("consumerActivityController")
@RequestMapping("/consumer/activity")
@Api(tags = "用户端-活动相关接口")
public class ActivityController {

    @Resource
    private IActivityService activityService;

    @GetMapping("/list")
    @ApiOperation("用户端抢券列表分页接口")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "tabType", value = "页面tab类型，1：疯抢中，2：即将开始", required = true, dataTypeClass = Integer.class)})
    public List<SeizeCouponInfoResDTO> queryForPage(@RequestParam(value = "tabType",required = true) Integer tabType) {
        return null;
    }


}
```



#### 3）接口实现

1. service接口

```Java
public interface IActivityService extends IService<Activity> {

/**
 * 用户端抢券列表分页查询活动信息
 *
 * @param tabType 页面类型
 * @return
 */
List<SeizeCouponInfoResDTO> queryForListFromCache(Integer tabType);
```

实现类：

```Java
@Override
public List<SeizeCouponInfoResDTO> queryForListFromCache(Integer tabType) {
    //从redis查询活动信息
    Object seizeCouponInfoStr = redisTemplate.opsForValue().get(ACTIVITY_CACHE_LIST);
    if (ObjectUtils.isNull(seizeCouponInfoStr)) {
        return CollUtils.emptyList();
    }
    //将json转为List
    List<SeizeCouponInfoResDTO> seizeCouponInfoResDTOS = JsonUtils.toList(seizeCouponInfoStr.toString(), SeizeCouponInfoResDTO.class);
    //根据tabType确定要查询的状态
    int queryStatus = tabType == TabTypeConstants.SEIZING ? DISTRIBUTING.getStatus() : NO_DISTRIBUTE.getStatus();
    //过滤数据，并设置剩余数量、实际状态
    List<SeizeCouponInfoResDTO> collect = seizeCouponInfoResDTOS.stream().filter(item -> queryStatus == getStatus(item.getDistributeStartTime(), item.getDistributeEndTime(), item.getStatus()))
            .peek(item -> {
                //剩余数量
                item.setRemainNum(item.getStockNum());
                //状态
                item.setStatus(queryStatus);
            }).collect(Collectors.toList());


    return collect;
}

/**
 * 获取状态，
 * 用于xxl或其他定时任务在高性能要求下无法做到实时状态
 *
 * @return
 */
private int getStatus(LocalDateTime distributeStartTime, LocalDateTime distributeEndTime, Integer status) {
    if (NO_DISTRIBUTE.equals(status) &&
            distributeStartTime.isBefore(DateUtils.now()) &&
            distributeEndTime.isAfter(DateUtils.now())) {//待生效状态，实际活动已开始
        return DISTRIBUTING.getStatus();
    }else if(NO_DISTRIBUTE.equals(status) &&
            distributeEndTime.isBefore(DateUtils.now())){//待生效状态，实际活动已结束
        return LOSE_EFFICACY.getStatus();
    }else if (DISTRIBUTING.equals(status) &&
            distributeEndTime.isBefore(DateUtils.now())) {//进行中状态，实际活动已结束
        return LOSE_EFFICACY.getStatus();
    }
    return status;
}
```



1. controller

```Java
@GetMapping("/list")
@ApiOperation("用户端抢券列表分页接口")
@ApiImplicitParams({
        @ApiImplicitParam(name = "tabType", value = "页面tab类型，1：疯抢中，2：即将开始", required = true, dataTypeClass = Integer.class)})
public List<SeizeCouponInfoResDTO> queryForPage(@RequestParam(value = "tabType",required = true) Integer tabType) {
    return activityService.queryForListFromCache(tabType);
}
```

#### 4）活动查询测试

测试流程如下：

启动网关

启动优惠券工程

参考预热程序测试的方法，创建活动信息，启动xxl-job，保证定时预热程序正常执行。

打开小程序，点击优惠券图标。

![这是云岚到家的家政服务界面，顶部显示定位为北京市，搜索栏内输入了“日常保洁”，界面中部展示了“全屋除菌保洁3次低至459元”的新人优惠活动，还标注有对应服务说明与立即抢购按钮；界面下方排列了保洁清、日常保洁、厨卫维修、日常维修、空调维修等服务入口，右下角有一个礼盒样式的图标，被红色箭头指向。该图片对应抢券查询相关的界面，可用于在该平台查询各类家政服务相关的活动与优惠信息。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/STvNbouIcoHWVmxj1b7c6vzQntk.png)

![图片展示的是云岚到家平台的抢券界面。界面顶部显示“云岚到家”，并有定位为北京市的图标及搜索框。中间部分有新人优惠活动宣传，如全屋除菌保洁3次低至459元等。下方有一个红色的优惠券图标，券面显示“10元”，并有“立即前往领取优惠卷”的红色按钮。该图片与文档中抢券查询接口测试状态实时性的上下文相关，用于展示抢券活动的界面情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/TmjcbPIjNosVm9xzhnRcPnZ9nRb.png)

进入抢券查询界面，通过选择不同的类型查询活动信息，如下图：

![图片展示的是抢券界面，上方显示“抢券”标题，右侧有三个点和一个放大镜图标。界面中有“疯抢中”和“即将开始”两个选项卡，“疯抢中”被红色线条突出显示。下方有一个橙色背景的促销区域，显示“9.0折”及“立即领取”，并注明“平台二周年庆 无门槛立享9折 2023-12-15~2023-12-24”。该图片与上文介绍的抢券查询界面相关，展示了抢券活动的具体信息。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XTdTbprN5op5RZxTwnAcxTewnjg.png)

![图片展示的是抢券查询界面。上方显示“抢券”标题，左侧有返回箭头，右侧有三个点和一个圆圈图标。中间有“疯抢中”和“即将开始”两个状态，其中“即将开始”被红色线条突出显示。下方有一个橙色券状图案，显示“¥10”，并有“双12大促 满100元立减10元 2023-12-17~2023-12-24”的文字说明。该图片与上下文关系为：在介绍抢券查询接口定义时，通过此图展示进入抢券查询界面后可看到的活动信息示例。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/QfwQbR4bgos7DXxN4TqcwK0Ynzc.png)

下边测试状态实时性：

1. 首先停止活动的状态更新任务。
2. 手动修改上述记录在活动表的开始或结束时间，使用实际的活动状态与数据库中记录的不一样

预期结果：

通过预热程序将活动提前加入缓存，前端请求查询活动信息，后端程序根据活动时间设定活动准确的活动状态。

示例：

修改上图中“ 双12大促” 活动的开始时间使该活动已开始。

测试效果如下图：

“ 双12大促” 活动在界面上显示“疯抢中”

![这张图片是抢券页面，属于第十章秒杀抢购中活动查询相关的测试效果展示，页面顶部标注了“抢券”，下方有“疯抢中”“即将开始”两个选项，其中“疯抢中”为选中状态。页面展示了两项可领取的活动：一项是“双12大促”，满100元立减10元，时间为2023年12月15日至2023年12月24日，其活动标签显示为“疯抢中”，与修改活动开始时间后的测试预期效果一致；另一项是“平台二周年庆”，提供无门槛9折优惠，时间同样为2023年12月15日至2023年12月24日，两项活动均配有“立即领取”按钮。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/T4RGbFwmsoM2QTxyzzhcX881nvc.png)

下边再修改数据库中的库存，设置为0，预期测试效果为“已抢光”，如下图：

![图片展示的是抢券页面，上方显示“抢券”标题，右侧有三个点和一个放大镜图标。页面中部有“已抢光”和“即将开始”两个选项卡，当前选中“已抢光”。下方展示了两个活动信息，分别是“双12大促”和“平台二周年庆”，均显示为“已抢光”，活动时间为2023 - 12 - 15 - 2023 - 12 - 24。该图片与上文测试效果示例相关，直观呈现了修改数据库中库存为0后，活动在界面上显示“已抢光”的情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/AcDMbbIJMocXElxmw4aczxQMnrg.png)

#### 5) 小结

通过定时任务更新活动状态，如何解决活动状态实时更新的问题？

1. 前端请求后端接口查询活动信息
2. 后端接口从redis查询活动信息，并根据活动开始和结束时间判断活动的最新状态
3. 最后将活动信息及最新状态返回给前端。



## 2.3 抢券

### 2.3.1 **解决超卖问题**

#### 1）系统需求

在抢券模块中需要实现下边两个需求：

**1、提升高并发吞吐量**

抢券类似抢购、秒杀等业务场景具有时效性的特点，提前规定好用户在什么时间可以抢购，用户访问集中，这样就会给系统造成高并发，抢券模块在设计时要以提升系统在高并发下的吞吐量为目标，**吞吐量表示单位时间内系统处理的总请求数量，吞吐量高意味着系统处理能力强。**

衡量系统吞吐量的常用指标有哪些？

**QPS（Queries Per Second）：**

每秒查询数（Queries Per Second），它表示系统在每秒内能够处理的查询或请求的数量，是衡量一个系统处理请求的性能和吞吐量的指标。

计算公式：总请求数/时间窗口大小

示例：

在10秒内处理1万个请求，QPS为10000。每个请求处理的时间越短，QPS越大。

假设：一个网站有10万用户，有2万日活跃用户，并发量是4000，每个用户每秒平均发起2个请求，那么总请求数就是 2\*4000，那么QPS就是 8000，如果单机支持2000的qps理论上需要4台服务器。

qps指标是需要根据服务器硬件性能、及具体的业务场景去测试，比如：门户查询数据如果直接走Nginx静态服务器则QPS可以达到上万，如果请求查询Tomcat，并且通过数据库去查询数据库返回，此时QPS会远低于查询Nginx静态服务器的QPS值，如果不走数据库,而是从Redis查询数其QPS也会大大提升。



**TPS（Transactions Per Second）：**

表示系统每秒完成的事务数，与QPS不同，TPS更关注系统的事务处理能力，而不仅仅是单纯的查询或请求，一次事务通常会包括多个请求。在高度事务性的系统中，如在线交易系统、支付系统等，TPS是一个关键指标，用于衡量系统的处理能力。

TPS指标通常会涉及业务处理及数据库存储，在测试时也需要根据服务器硬件性能、及具体的业务场景去测试，拿下单举例：单机支持几十到几百的TPS指标属于正常。

在开发中，对以上性能指标的优化，可通过CDN、缓存、异步处理、数据库优化、多线程、集群、负载均衡等技术去提高系统的吞吐量。当然，再优化也不要忘记系统保护，通过限流技术根据系统的性能指标进行限流保护。



**2、解决超卖问题**

抢购、秒杀等业务场景还需要解决超卖问题，超卖是最终下单购买数量大于库存数量，比如：库存有１00个，用户最终购买了101个，多出这一个就是超卖了，结合抢券业务即用户最终抢到的优惠券总数大于优惠券库存数。

下边先分析超卖问题的解决方案。

#### 2）**什么是超卖问题**

超卖是最终下单购买数量大于库存数量，比如：库存１00个用户最终购买成功了101个，多出这一个就是超卖了，结合抢券业务，用户最终抢到的优惠券总数大于优惠券库存数就出现了超卖问题。

**导致超卖问题的原因是什么呢？**

下边举例说明超卖问题并分析导致超卖问题的原因。

下图是两个线程更新数据库的库存字段。

线程1：先查询库存为1，判断是否大于0，如果大于则库存减1，最后更新数据库库存字段。

线程2：先查询库存为1，判断是否大于0，如果大于则库存减1，最后更新数据库库存字段。

线程1和线程2查询到的库存都是1，两个线程分别减1得到剩余库存数0，由于线程2并不是基于线程1扣减库存后的值进行扣减，线程2更新库存覆盖了线程1更新的库存值。

上边的例子就出现了超卖的问题。

**造成超卖问题的原因是在高并发场景下对库存这个共享资源进行操作存在线程不安全所导致。**

<whiteboard token="PORlwQo4mhOZQhbmHWWcf0FbnQe"></whiteboard>



#### 3）悲观锁与乐观锁

提到解决线程安全问题大家想到了锁，下边复习下关于锁的基本概念。

jvm提供了很多锁，比如：synchronized、reentrantLock、CAS等，它们都可以解决线程安全问题，synchronized、reentrantLock可以实现悲观锁，CAS可以实现乐观锁，关于这些锁的知识掌握不牢固的一定要自行复习。下边理解悲观锁与乐观锁的概念。

1. **悲观锁**

悲观锁是一种悲观思想，总认为会有其它线程修改数据，为了保证线程安全所以在操作前总是先加锁，操作完成后释放锁，其它线程只有当锁释放后才可以获取锁继续操作数据。synchronized和ReentrantLock都可以实现悲观锁。

<whiteboard token="SgxhwhzD1hAyH4btXtec1ibLnrg"></whiteboard>

使用悲观锁后原来的多线程并发执行改为了顺序（同步）执行，当线程2去执行时查询到库存发现为0，不满足条件更新库存失败。



1. **乐观锁**

乐观锁则是一种乐观思想，认为不会有太多线程去并发修改数据，所以谁都可以去执行代码。

Java提供的CAS机制可以实现乐观锁，CAS即Compare And Swap 比较并交换，在修改数据前先比较版本号，如果数据的版本号没有变化说明数据没有修改，此时再去更改数据。

业务逻辑：

1. 先查询 库存,版本号   select id,stok,version form activity   result:1,1,1
2. 判断库存   stok > 0
3. 更新 库存、版本号  update activity set stok = stok-1, version= version + 1 where id=1 and version=1



示例如下：

库存数据对应一个版本，库存每次变化则版本号跟着变化，如下：

| 库存 | 版本号 |
|-|-|
| 100 | 1 |
| 99 | 2 |
| ... | ... |
| 1 | 100 |
| 0 | 101 |

线程1修改库存前拿到库存及对应的版本号：1和100。

线程1判断库存如果大于0则将库存减1，准备更新库存。

更新库存时要校验当前库存的版本是否和自己之前拿到的一致，如果版本号为1说明自己在执行的这过程没有其它线程去修改过库存，此时将库存更新为99并将版本号加1为2。

线程2执行和线程1一样的逻辑，线程2去更新库存时发现库存的版本号为2与自己之前拿到的不一致，更新库存失败。

1. **结论**

悲观锁和乐观锁都是一种解决共享资源的线程安全问题的方法，悲观锁是在读数据时就加锁，如果读比较多则加锁频繁影响性能，相比而言乐观锁性能比悲观锁要好。



#### 4）**数据库行锁控制方案**

数据库的行级锁可以实现悲观锁也可以实现乐观锁。

1. **实现悲观锁（排他锁）**

执行select … for update 实现加锁，select … for update 会锁住符合条件的行的数据，如下语句会锁一行的数据

```Java
select * from 库存表 where id=? for update
```

通常此语句放在事务中，开启事务开始时执行此语句获取锁，事务提交或回滚自动锁释放，保证在事务处理过程中没有其它线程去修改数据。

测试：

使用一个线程执行下边的命令：

```JSON
start transaction;

select ... for update;

COMMIT;
```

另一个线程修改数据，如果上边不释放锁将无法修改。

**高并发场景不推荐使用select … for update方法，同时也可能存在死锁的潜在风险。**



1. **实现乐观锁**

数据库的行级锁也可以实现乐观锁，通用的做法是在表中添加一个version版本字段，在更新时对比版本号，更新成功将版本号加1，SQL为：

```Java
update 表名 set 字段=值,version=version+1 where  id =? and version =? 
```

针对扣减库存业务扣减库存SQL：

```Java
update 库存表 set 库存=库存-2 where 库存>=2 and id =? 
```

多线程执行上边的SQL，假如线程1先执行会添加排他锁，当事务没有结束前其它线程去更新同一条记录会被阻塞，等到线程1更新结束其它线程才可以更新库存。

<whiteboard token="PV9HwNPJlhlKDTbzlOIcOHn9nTe"></whiteboard>

当执行update后返回影响的记录行数为1表示更新成功即扣减库存成功，返回0表示没有更新记录行，即扣减库存失败。

1. **结论**

悲观锁在查询时就开始加锁，如果读比较多则加锁频繁影响性能，相比而言乐观锁性能比悲观锁要好。

对于并发不高的场景可以使用数据乐观锁去控制扣减库存，由于抢购业务并发较高且对性能要求也高，如果使用数据库行锁去控制，并发高就会对数据库造成压力，如果进行限流控制并发数又无法满足性能要求，所以对于抢购业务使用数据库行锁进行控制是不适合的。



#### 5)   **Redis分布式锁方案**

数据库乐观锁不适用高并发场景，我们能否将库存数据放在Redis，并且通过JVM锁去控制扣减库存呢？

上边介绍的synchronized、reentrantLock、CAS只控制了JVM本身的线程争抢同一个锁，无法控制多个JVM之间争抢同一个锁。

如下图，有两个JVM进程，每个JVM进程都有一个Lock01锁，这两个JVM进程中的线程1仍然会同时去修改库存：

线程1：先查询库存为1，判断是否大于0，如果大于则库存减1，最后更新Redis库存数据。

线程2：先查询库存为1，判断是否大于0，如果大于则库存减1，最后更新Redis库存数据。

此时就会出现修改库存数据的线程不安全问题。

<whiteboard token="SVuRwfY5qh4ghablaOBcPwHVnod"></whiteboard>

所以，如果是单机环境下，使用JVM的锁在内存加锁可以解决资源并发访问的线程安全问题。

微服务架构的项目在部署时每个微服务会部署多个实例（JVM），每个实例就是一个JVM，如果要控制多个JVM之间争抢资源需要用到分布式锁，分布式锁是由一个统一的服务提供分布式锁服务，比如：使用redis、数据库都可以实现分布式锁，下边介绍分布式锁控制争抢资源的方法。

如下图：每个JVM中的线程去争抢同一个分布式锁，在扣减库存前先获取分布式锁，拿到锁再扣减库存，执行完释放锁之后其它JVM的线程才可以获取锁继续扣减库存，如下图：

<whiteboard token="VXXjwZlwbhEpx7bFmBNcVQ5ZnGd"></whiteboard>

上边的方案将库存放在Redis中避免与数据库交互，很大的提高的了执行效率，在分布式场景下使用分布式锁是一种常用的控制共享资源的方案。

分布式锁需要搭建独立的分布式锁服务（例如Redis、Zookeeper等），每次操作需要远程与分布式锁服务交互获取锁、释放锁，还有没有性能更高的方案呢？



#### 6）**Redis原子操作方案**

上边使用分布式锁的方案每次操作需要远程与分布式锁服务交互获取锁、释放锁，有没有优化的方法避免申请锁与释放锁的交互呢？

在分布式锁方案中是在java程序中扣减库存最后更新redis库存的值，能否使用redis的decr命令去扣减库存呢？

Redis Decr 命令将 key 中储存的数字值减一，并且具有原子性，Redis中所有命令都具有原子性。

**原子性表示该命令在执行过程中是不被中断的**，也就实现了多线程去执行decr命令扣减库存是顺序执行的，假如库存原来是100，扣减到0结束，多线程并发执行decr命令不会出现扣减次数超过100次，如下图：

<whiteboard token="XQcswf5qShpTy9bweqxclAyXnSQ"></whiteboard>

基于这个思想可以对分布式锁方案优化如下：

<whiteboard token="QErKwhtqUhabz6bIg0qc8qaFnac"></whiteboard>

此方案中没有使用分布式锁，而是基于Redis命令具有原子性的特点实现。

本项目使用Redis原子操作控制超卖问题。

#### 7）**小结**

解决超卖问题有哪些方案？



什么是悲观锁和乐观锁？

数据库的怎么实现悲观锁和乐观锁？



### 2.3.2 Redis原子操作方案

在Redis原子操作方案中扣减库存使用decr命令实现，decr命令具有原子性，如果在扣减库存操作中有多个操作 ，那么整体还是原子性吗？如下图：

<whiteboard token="SzH4wvvakhx5mkbCiB2cdvuDnVb"></whiteboard>

扣减库存逻辑如下：

1、首先查询库存

2、判断库存大小，如果大于0则扣减库存，否则 直接返回

3、记录抢券成功的记录，用于判断用户不能重复抢券的依据。

4、记录抢券同步的记录，用于后续的异步处理，将抢券结果保存到数据库。

如果上述四步整体不具有原子性仍然没有办法控制超卖问题，所以必须保证1、2、3步逻辑放在一起整体具有原子性。

如何保证多个Redis命令具有原子性呢？本节介绍两个保证Redis多个命令具有原子性的方法。

#### 1）**通过  MULTI 事务命令实现**

对于redis单个命令都是原子操作，现在要求扣减库存、写入抢券成功队列及写入同步队列保证原子性，多个redis命令如何保证原子性呢？

**1、通过  MULTI 事务命令实现**

下边的命令执行流程如下：

执行MULTI 标记首先标记一个事务块开始。

然后将要执行的命令加入队列。

将“HSET key1 field1 value2 field2 value2” 命令放入队列中，表示向key1中写入两个hashkey。

将“INCR key2”命令放入队列中，表示对key2自增1。

运行EXEC命令按顺序执行，整体保证原子性。

```Plain Text
MULTI
HSET key1 field1 value2 field2 value2
INCR key2
EXEC
```

测试如下：

![图片展示了Redis原子操作方案的测试结果。首先执行“MULTI”命令，返回“OK”；接着执行“HSET test_key01 field1 aa field2 bb”命令，返回“QUEUED”；再执行“INCR test_key02”命令，同样返回“QUEUED”；最后执行“EXEC”命令，依次返回“OK”“2”“OK”“1”“OK”。这与上下文提到的将命令加入队列后运行EXEC命令按顺序执行，整体保证原子性的内容相呼应，直观呈现了测试过程及结果。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/S0B9bsLpMo8O24x2dVickoXMnSG.png)

测试结果：

![图片展示的是Redis操作界面中关于键test_key01的hash数据结构内容。界面显示键为test_key01，包含两个字段及对应值，分别为field1和aa，field2和bb。该图片与上文Redis原子操作方案相关，是对将“HSET key1 field1 value2 field2 value2”命令放入队列后，向key1中写入两个hashkey操作的测试结果展示，直观呈现了操作后的hash数据结构状态。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/W7TMbl0MWo4nVtxFVfaciVLUnYd.png)

![图片展示的是Redis操作界面中关于键值对的显示结果。上方“STRING:”后显示键名为“test_key02”，下方“键值: 大小: 1 字节”后显示值为“1”。该图片与上下文紧密相关，上下文提到将“HSET key1 field1 value2 field2 value2”命令放入队列，表示向key1中写入两个hashkey，而此图片正是展示了key1写入后的值为1，体现了Redis键值对操作的结果呈现。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/CRlKbXLVQovTknxN5GFc29MNn2e.png)

#### 2）了解Pipeline与**MULTI 的区别**

学习过Redis的同学听说过pipline管道命令，pipline也可实现批量执行多个 redis命令，pipline与multi的区别是：

pipeline 是把多个redis指令一起发出去，redis并没有保证这些命令的执行是原子的；multi实现的是将多个命令作为事务块去执行，保证整个操作的原子性。

如果仅是执行多个命令不保证原子性那么使用pipeline 的性能要比multi要高，但是针对本项目要保证多个命令实现原子性的需求那么pipeline 不符合要求。

#### 3）Redis+Lua实现

Lua 是一种强大、高效、轻量级、可嵌入的脚本语言，Lua体积小、启动速度快，从而适合嵌入在别的程序里，Lua可以用于web开发、游戏开发、嵌入式开发等领域。

参考：http://www.lua.org/docs.html，或者去百度搜索Lua中文教程。

对于Lua脚本语法非常容易理解，先不用系统的去学习，先把本项目使用的Lua脚本读懂即可，实际工作中用到时再参考本项目的脚本去写，不会的再查Lua 的语法。

先看一个例子，对上边的例子编写Lua脚本，如下：

```Lua
local ret = redis.call('hset', KEYS[1], ARGV[1], ARGV[2], ARGV[3], ARGV[4]);
redis.call('incr', KEYS[2]);
return ret..'';
```

说明：

KEYS：表示在脚本中所用到的那些 Redis 键(key)，这些键名参数可以在 Lua 中通过全局变量 KEYS 数组，KEYS[1]表示第一个key，KEYS[2]表示第2个key，依次类推。

ARGV：表示在脚本中所用到的参数，在 Lua 中通过全局变量 ARGV 数组访问，访问的形式和 KEYS 变量类似( ARGV[1] 、 ARGV[2] ，诸如此类)，ARGV[1]表示第一个参数，ARGV[2]表示第二个参数，依次类推。

如何执行上边的Lua脚本呢？

使用EVAL 命令执行Lua脚本。

EVAL是redis的命令本身具有原子性，整个脚本的执行具有原子性。

```Java
EVAL script numkeys key [key ...] arg [arg ...] 
```

参数说明：

- **script**： 是一段 Lua 5.1 脚本程序。
- **numkeys**： 用于指定键名参数的个数。
- **key [key ...]**： 从 EVAL 的第三个参数开始算起，表示在脚本中所用到的那些 Redis 键(key)，这些键名参数可以在 Lua 中通过全局变量 KEYS 数组，用 1 为基址的形式访问( KEYS[1] ， KEYS[2] ，以此类推)。
- **arg [arg ...]**： 附加参数，在 Lua 中通过全局变量 ARGV 数组访问，访问的形式和 KEYS 变量类似( ARGV[1] 、 ARGV[2] ，诸如此类)。

执行下边的命令：

```Java
eval "local ret = redis.call('hset', KEYS[1], ARGV[1], ARGV[2], ARGV[3], ARGV[4]);redis.call('incr', KEYS[2]);return ret..'';" 2 test_key01 test_key02 field1 aa field2 bb
```

说明：

eval后边的script参数即脚本程序，将上边的Lua脚本使用双引号括起来。

numkeys：为2表示2个key

之后传入key的名称（多key中间用空格分隔）：test_key01 test_key02

key后边再传入ARGV 参数（多ARGV 中间用空格分隔）：field1 aa field2 bb

测试结果如下所示：

![图片展示的是Redis执行Lua脚本的命令及结果。命令为“eval 'local ret = redis.call('hset', KEYS\[1\], ARGV\[1\], ARGV\[2\], ARGV\[3\], ARGV\[4\]);redis.call('incr', KEYS\[2\]);return ret..';'”，参数为“2 test_key01 test_key02 field1 aa field2 bb”。执行结果为“2”，表明向hash中写入2个key并返回结果。该图片与上下文紧密相关，是对Redis+Lua实现抢券方案中Redis执行Lua脚本方法的示例展示。](https://feishu.cn/file/ILaxbMC9AoTuCCxcFm3ccNURnEf)

返回2表示向hash中写入2个key

下边是使用RedisTemplate执行Lua脚本的方法：

```Java
<T> T execute(RedisScript<T> script, List<K> keys, Object... args)
```

通过第一个参数类型指定要执行的Lua脚本，RedisScript的实现类是DefaultRedisScript，下边查阅DefaultRedisScript的源代码。

第一种方法是将Lua脚本的内容作为字符串传入DefaultRedisScript对象并且指定返回值类型，代码如下：

```Java
public DefaultRedisScript(String script, @Nullable Class<T> resultType) {
    this.shaModifiedMonitor = new Object();
    this.setScriptText(script);
    this.resultType = resultType;
}
```

如果脚本内容比较多使用第一种方法显得很麻烦。

第二种方法是指定Lua脚本的位置，通过DefaultRedisScript的setScriptSource方法完成，如下：

```Java
public void setScriptSource(ScriptSource scriptSource) {
    this.scriptSource = scriptSource;
}
```



本项目使用第二种方法，在RedisLuaConfiguration中定义DefaultRedisScript bean

```Java
@Bean("Lua_test01")
public DefaultRedisScript<Integer> getLuaTest01() {
    DefaultRedisScript<Integer> redisScript = new DefaultRedisScript<>();
    //resource目录下的scripts文件下的Lua_test01.Lua文件
    redisScript.setScriptSource(new ResourceScriptSource(new ClassPathResource("scripts/Lua_test01.Lua")));
    redisScript.setResultType(Integer.class);
    return redisScript;
}
```

在classpath的script下创建lua脚本Lua_test01.lua，内容如下：

```Java
local ret = redis.call('hset', KEYS[1], ARGV[1], ARGV[2], ARGV[3], ARGV[4]);
redis.call('incr', KEYS[2]);
return ret..'';
```

如下图：

![图片展示了在Idea中创建的lua脚本文件lua_test01.lua内容。文件位于jzo2o - market项目src/main/resources/scripts目录下。代码中，第1行定义变量ret，通过redis.call函数执行hset操作；第2行执行incr操作；第3行返回ret值。该图片与文档中介绍Redis+Lua实现抢券方案的内容相关，是对在classpath的script下创建lua脚本的示例展示。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/JUvsbZfjNoSSFCxjsp3cMFR0nlc.png)



创建RedisTest测试类:

注入上边定义的DefaultRedisScript，注意注入时指定名称“Lua_test01”。

```Java
package com.jzo2o.market.service;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;

import javax.annotation.Resource;
import java.util.Arrays;
import java.util.List;

/**
 * @author Mr.M
 * @version 1.0
 * @description TODO
 * @date 2023/10/13 16:28
 */
@SpringBootTest
@Slf4j
public class RedisLuaTest {


    @Resource
    RedisTemplate redisTemplate;

    @Resource(name = "Lua_test01")
    DefaultRedisScript script;

    //测试Lua
    @Test
    public void test_Luafirst() {
        //参数1：key ,key1:test_key01  key2:test_key02
        List<String> keys = Arrays.asList("test_key01","test_key02");
        //参数2：传入Lua脚本的参数,"field1","aa","field2", "bb"
        Object result = redisTemplate.execute(script, keys, "field1","aa","field2", "bb");
        log.info("执行结果:{}",result);
    }

}
```

执行test_Luafirst() 测试方法。

#### 4）选择方案

上边学习了Multi和Redis+Lua两种实现Redis原子操作的方案，在本项目你会选择哪一种方案？

你肯定会说是第一种方案，因为简单，使用Lua脚本还要写Lua脚本，去学习它的语法。

不过，在实际使用中要根据具体的需求去确定方案，比如下边的Lua脚本在执行过程中就会有一些业务逻辑判断，不满足条件提前返回结果，而MULTI 执行命令是执行完成最后一起拿到所有命令的执行结果，并且MULTI 不适合写带有业务逻辑的脚本内容。

下边的lua脚本实现了项目抢券功能，可以尝试阅读，稍后会详细讲解。

```Java
-- 抢券Lua实现
-- key: 抢券同步队列，资源库存,抢券成功列表
-- argv：活动id,用户id

--优惠券是否已经抢过
local couponNum = redis.call("HGET", KEYS[3], ARGV[2])
-- hget 获取不到数据返回false而不是nil
if couponNum ~= false and tonumber(couponNum) >= 1
then
    return "-1";
end
-- --库存是否充足校验
local stockNum = redis.call("HGET",KEYS[2], ARGV[1])
if stockNum == false or  tonumber(stockNum) < 1
then
    return "-2";
end
--抢券列表
local listNum = redis.call("HSET",KEYS[3], ARGV[2], 1)
if listNum == false or  tonumber(listNum) < 1
then
    return "-3";
end

--减库存
stockNum = redis.call("HINCRBY",KEYS[2], ARGV[1], -1)
if tonumber(stockNum) < 0
then
    return "-4"
end
-- 抢单结果写入同步队列
local result = redis.call("HSETNX", KEYS[1], ARGV[2],ARGV[1])
if result > 0
then
    return ARGV[1] ..""
end
return "-5"
```

**根据需求本项目使用Redis执行Lua脚本的方式保证多命令的原子性，完成抢券功能。**



#### 5）**使用Lua脚本注意点**

Lua脚本在redis集群上执行需要注意什么？

在redis集群下执行redis命令会根据key求哈希，确定具体的槽位（slot），然后将命令路由到负责该槽位的 Redis 节点上。

执行一次Lua脚本会涉及到多个key，在redis集群下执行lua脚本要求多个key必须最终落到同一个节点，否则调用Lua脚本会报错：ERR eval/evalsha command keys must be in same slot。

如何保证多个key落地到一个redis节点呢？

只要保证多个key的哈希值一致即可保证多个key落到一个redis节点上，这个如何实现呢？

解决方法：一次执行Lua脚本的所有key中使用大括号‘{}’且保证大括号中的内容相同，此时会根据大括号中的内容求哈希，因为内容相同所以求得的哈希数据相同所以就落在了同一个Redis节点。

测试如下：

在key名称后边添加{}，大括号中写一个固定的值。

```Java
@Test
public void test_Luafirst2() {
    //参数1：key ,key1:test_key01
    List<String> keys = Arrays.asList("test_key01{111}","test_key02{111}");
    //参数2：传入Lua脚本的参数,"field1","aa","field2", "bb"
    Object result = redisTemplate.execute(script, keys, "field1","aa","field2", "bb");
    log.info("执行结果:{}",result);
}
```

执行测试成功，观察redis多了两个key："test_key01{111}"和"test_key02{111}"

![图片展示了Redis数据库中部分键值对。在192.168.101.68的db0下，有COUPON、JZ_CACHE、ORDERS等键值对，其中test_key01、test_key02键值对被红色框突出显示。test_key01对应的值为“test_key01{111}”，test_key02对应的值为“test_key02{111}”。该图片与上文提到的在key名称后边添加{}，大括号中写一个固定的值，执行测试成功后观察redis新增两个key的内容相关，直观呈现了测试结果。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Ir6wba2wbobXKPx0uxBcc3eXnxd.png)



#### 6）**小结**

项目中保证Redis原子操作用什么方案？



### 2.3.3 抢券整体方案

#### 1）抢券方案分析

抢券的架构设计思想同抢券查询，将库存保存在Redis，避免抢券操作请求数据库，通过异步任务将Redis中的抢券结果同步到数据库。

抢券的交互流程如下：

如下图：

<whiteboard token="LYkrwQvsthnQC0bX1jjcynWinbA"></whiteboard>

说明如下：

1、由预热程序将待生效库存同步到redis(活动开始将不允许更改库存)

2、活动开始后，抢券程序请求Redis扣减库存，扣减库存成功向抢券成功队列和抢券同步队列写入记录

Redis中两个队列的作用如下：

抢券成功队列：为了校验用户是否抢过该优惠券。

抢券同步队列：将抢券结果同步到数据库

3、通过定时任务程序根据Redis中同步队列记录的用户抢券结果信息将数据同步到MySQL，具体操作如下：

向优惠券表插入用户抢券记录。

更新优惠券活动表的库存。

写入数据库完成后删除Redis中同步队列的相应记录，删除后表示同步完成，如果同步过程失败将保留Redis同步队列的相应记录。





#### 2）数据流

根据交互流程分析数据流如下：

<whiteboard token="NM6VwlvuHhotFrbcG75cwZiOnjg"></whiteboard>



#### 3）Redis数据结构

1. **活动信息**

缓存结构：String类型:

key: "ACTIVITY:LIST"

value: 符合条件的优惠券活动列表JSON数据。

过期时间：永不过期

缓存一致性方案：通过预热程序保证缓存一致性



1. **优惠券活动库存**

缓存结构：Hash

RedisKey：COUPON:RESOURCE:STOCK:{活动id%10}

{活动id%10}表示根据活动id除以10求余，通过这种方法将key分散到不同的redis服务器上，通过“活动id%10”表达式可知优惠券活动库存hash最多有10个。 

HashKey：活动id

HashValue: 库存

过期时间：永不过期

缓存一致性方案：通过预热程序保证缓存一致性

<whiteboard token="GFhXwg4lrhdXfPbfQ3EcWZuhnuc"></whiteboard>

举例：

如果n为10，1号活动的库存是100，将1号活动库存 存储到Redis的效果如下：

<whiteboard token="See0w9NFXhrXx7bKsSAcdhhonOd"></whiteboard>



1. **抢券成功队列**

缓存结构：Hash

RedisKey：COUPON:SEIZE:LIST:活动id\_{活动id%10}

HashKey：用户id

HashValue:1

过期时间：永不过期

<whiteboard token="OZ5awAeG3h8JWFbhsRzcRlj0npd"></whiteboard>



1. **抢券同步队列**

缓存结构：Hash

RedisKey：QUEUE:COUPON:SEIZE:SYNC:{活动id%10}

HashKey：用户id

HashValue:活动id

过期时间：永不过期

<whiteboard token="UFFwwJxhihY1WmbkIPwczuVmnWb"></whiteboard>



#### 4）小结

抢券是怎么做的？或方案是什么？

抢券业务的Redis数据结构用的什么？具体说说

秒杀系统中如何进行流量削峰？

在秒杀系统中进行流量削峰是非常重要的，因为瞬时的高流量可能会导致系统崩溃或性能下降。以下是一些常见的流量削峰策略：

1. **限流措施：**通过控制请求的发放速率，可以有效地平滑流量，避免瞬时的高并发。
2. **队列缓冲：** 使用消息队列来缓冲请求，将瞬时的高并发请求进行缓存和排队。秒杀系统可以异步地从队列中取出请求进行处理，以平滑处理流量。
3. **分批处理：**将瞬时的高并发请求分批处理。不需要一次性处理所有请求，可以将请求按照一定的规模分批处理，以减轻数据库和系统的压力。
4. **负载均衡：**采用多节点部署，通过负载均衡器将流量分发到不同的服务器上。
5. **熔断机制：**

   - **熔断策略：** 实现熔断机制，当系统达到一定的负载阈值时，暂时停止接受新的请求，防止系统崩溃。等到系统恢复后再重新开启。
6. **缓存预热：**在秒杀开始之前，提前将秒杀商品的信息加载到缓存中，减轻数据库的压力。
7. **验证码和身份验证：**引入验证码和身份验证机制，防止机器人或恶意请求，减少无效请求对系统的冲击。
8. **数据库优化：**对于秒杀系统，数据库通常是瓶颈之一。通过优化数据库结构、建立索引、使用缓存等手段来提高数据库的读写性能。 
9. 

### 2.3.4 库存同步

#### 1）系统设计

库存同步包括两部分：

1. 用户抢券要在Redis扣减库存，所以需要提前将优惠券活动的库存同步到Redis。

可以通过定时预热程序中将优惠券活动的库存同步到Redis，同步规则如下：

- 对于待生效的活动更新库存。
- 对于已生效的活动如果库存已经同步则不再同步，只更新没有同步库存的活动。

做第二点的原因是为了避免时间差问题，活动状态更改为进行中了但是库存还没有同步到Redis。



1. 当用户抢券成功，Redis中的库存有了变化，需要将redis中的库存同步到mysql

根据整体方案分析，在抢券结果同步程序中根据抢券结果修改数据库中的库存，此部分在抢券结果同步章节再确定具体的方法。

交互流程如下：

<whiteboard token="TLJdwUtethGI4ibSjJ8cjkkxn1b"></whiteboard>

#### 2）预热程序中同步库存

下边实现在预热程序中同步库存。

在预热程序中添加：

```JSON
@Override
public void preHeat() {
    ....

    // 将待生效的活动库存写入redis
    list.stream().filter(v->getStatus(v.getDistributeStartTime(),v.getDistributeEndTime(),v.getStatus())==1).forEach(v->{
        redisTemplate.opsForHash().put(String.format(COUPON_RESOURCE_STOCK, v.getId() % 10), v.getId(), v.getTotalNum());
    });
    // 对于已生效的活动库存没有同步时再进行同步
    list.stream().filter(v->getStatus(v.getDistributeStartTime(),v.getDistributeEndTime(),v.getStatus())==2).forEach(v->{
        redisTemplate.opsForHash().putIfAbsent(String.format(COUPON_RESOURCE_STOCK, v.getId() % 10), v.getId(), v.getTotalNum());
    });
}
```

说明：

对于待生效的活动库存使用put方法，可以对已设置的记录进行更改。

对已生效的活动库存使用putIfAbsent实现，当key不存在时才执行设置操作。

String.*format*(*COUPON_RESOURCE_STOCK*, v.getId() % 10) 用来拼装 key，库存的redis key为：

COUPON:RESOURCE:STOCK:{活动id%10}。



#### 3）测试

测试流程：

启动定时预热程序任务。

观察redis中是否成功存储库存信息。

示例：

![这张图片展示的是redis数据库的查询结果界面，左侧树状菜单显示IP地址为192.168.101.68的数据库下，存在名为COUPON、RESOURCE、STOCK的文件夹，当前选中的是STOCK文件夹下的键{0}。右侧对应HASH类型的存储内容，键为“COUPON:RESOURCE:STOCK:{0}”，其中包含一条记录，key的值是“1735949163390156800”，对应的value值为100，该内容正是测试库存同步时，redis中存储的库存信息，对应了库存同步到redis的操作结果示例。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/PYsWbdjfAoSZD6xZpS2cK9j9npd.png)



#### 4）小结

库存怎么同步到redis？

通过定时预热程序将活动库存信息同步到redis。

对于待生效的活动库存使用put方法，可以对已设置的记录进行更改。

对已生效的活动库存使用putIfAbsent实现，当key不存在时才执行设置操作。



### 2.3.5 抢券Lua脚本

#### 1）抢券Lua脚本

本节对抢券Lua脚本进行阅读并测试，理解抢券的过程。

1. **阅读下边的Lua脚本**

```Java
-- 抢券lua实现
-- key: 抢券同步队列，资源库存,抢券成功列表
-- argv：活动id,用户id

--优惠券是否已经抢过
local couponNum = redis.call("HGET", KEYS[3], ARGV[2])
-- hget 获取不到数据返回false而不是nil
if couponNum ~= false and tonumber(couponNum) >= 1
then
    return "-1";
end
-- --库存是否充足校验
local stockNum = redis.call("HGET",KEYS[2], ARGV[1])
if stockNum == false or  tonumber(stockNum) < 1
then
    return "-2";
end
--抢券列表
local listNum = redis.call("HSET",KEYS[3], ARGV[2], 1)
if listNum == false or  tonumber(listNum) < 1
then
    return "-3";
end

--减库存
stockNum = redis.call("HINCRBY",KEYS[2], ARGV[1], -1)
if tonumber(stockNum) < 0
then
    return "-4"
end
-- 抢券结果写入同步队列
local result = redis.call("HSETNX", KEYS[1], ARGV[2],ARGV[1])
if result > 0
then
    return ARGV[1] ..""
end
return "-5"
```

错误代码：

-1： 限领一张

-2： 已抢光

-3： 写入抢券成功队列失败，返回给用户为：抢券失败

-4：  已抢光

-5： 写入抢券同步队列失败，返回给用户为：抢券失败



#### 2）**测试**

编写测试方法，准备好调用抢券Lua脚本需要传入的key和参数。

代码如下：

```Java

package com.jzo2o.market.service;

import com.jzo2o.api.market.dto.request.CouponUseBackReqDTO;
import com.jzo2o.api.market.dto.request.CouponUseReqDTO;
import com.jzo2o.api.market.dto.response.AvailableCouponsResDTO;
import com.jzo2o.api.market.dto.response.CouponUseResDTO;
import com.jzo2o.common.constants.UserType;
import com.jzo2o.common.model.CurrentUserInfo;
import com.jzo2o.common.model.PageResult;
import com.jzo2o.common.utils.JsonUtils;
import com.jzo2o.market.model.dto.request.CouponOperationPageQueryReqDTO;
import com.jzo2o.market.model.dto.request.SeizeCouponReqDTO;
import com.jzo2o.market.model.dto.response.CouponInfoResDTO;
import com.jzo2o.mvc.utils.UserContext;
import com.jzo2o.redis.utils.RedisSyncQueueUtils;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;

import javax.annotation.Resource;

import java.math.BigDecimal;
import java.util.Arrays;
import java.util.List;

import static com.jzo2o.market.constants.RedisConstants.RedisKey.*;
import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
@Slf4j
public class ICouponServiceTest {

@Test
void test_seizeCouponScriptLua() {

    //argv：抢券活动id
    long activityId = 1706183021040336896L;
    // argv: 用户id
    Long userId = 1694250327664218113L;
    int index = (int) (activityId % 10);
    //key: 抢券同步队列，资源库存,抢券成功列表
    // 同步队列redisKey
    String couponSeizeSyncRedisKey = RedisSyncQueueUtils.getQueueRedisKey(COUPON_SEIZE_SYNC_QUEUE_NAME, index);
    // 资源库存redisKey
    String resourceStockRedisKey = String.format(COUPON_RESOURCE_STOCK, index);
    // 抢券成功列表
    String couponSeizeListRedisKey = String.format(COUPON_SEIZE_LIST,activityId, index);
    // 抢券
    Object execute = redisTemplate.execute(seizeCouponScript, Arrays.asList(couponSeizeSyncRedisKey, resourceStockRedisKey, couponSeizeListRedisKey),
            activityId, userId);
    log.debug("seize coupon result : {}", execute);
}
```

执行成功，观察redis:

示例：

库存是否减少：

![图片展示的是库存减少情况，属于抢券Lua脚本测试内容。表格中“key”列显示库存编号“1706183021040336896”，“value”列显示库存数量为8。这与上下文提到的“库存是否减少”相关，用于验证抢券操作后库存是否按预期减少，是测试抢券Lua脚本效果的重要依据。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/ZAdlbCHFboLxtEx9Ac5cuJoMnzK.png)

抢券成功队列是否存在相应记录：

![图片展示的是抢券成功队列的Redis数据结构示例。上方显示HASH类型，键名为COUPON:SEIZE:LIST:1706183021040336896_{6} 。下方表格中，key列包含三个数字，value列对应均为1。该图片与上文测试抢券Lua脚本时观察redis的内容相关，用于说明抢券成功队列是否存在相应记录这一测试结果。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NQe9b24cxozH8xxiBmUcp6E4nTg.png)

抢券同步队列是否存在相应记录：

![这张图片展示了抢券场景下Redis哈希（HASH）类型的存储记录，哈希的键为“QUEUE:COUPON:SEIZE:SYNC:{6}”。该哈希结构包含两条核心数据，序号为1的key是“1694250327664218113”，对应的value为“1706183021040336896”，这正是测试抢券Lua脚本时，需要检查的抢券同步队列的相关记录，是验证Lua脚本执行是否成功的关键观测项之一。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/POKmbu1ZKoqvU4xiWiAcdqZ1nof.png)

#### 3) 小结

抢券的Lua脚本做的什么工作？

1. 判断用户是否在该活动抢过券。
2. 判断库存是否充足
3. 写入抢券成功列表
4. 扣减库存
5. 写入抢券同步列表



### 2.3.6 抢券接口开发

#### 1）接口定义

下边进行接口分析，定义抢券接口。

在下边的界面中点击“立即领取”即开始抢券。

![图片展示的是抢券界面。上方显示“抢券”及“即将开始”字样。中间有“20”元优惠券，标注“立即领取”，并有“2”号黄色圆圈标识。下方有“5折”优惠券，标注“已抢光”，并有“1”号黄色圆圈标识。该图片与上下文关系紧密，上下文提到在该界面点击“立即领取”即开始抢券，图片直观呈现了抢券操作的界面情况，帮助理解抢券操作的场景。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/STjjbvzhkorlNGxcL11cfpWDndh.png)

请求哪些参数？

抢券需要明确两个元素： 哪个用户抢的是哪个活动的优惠券。

用户的身份信息在token中由前端传入服务端。

所以，本接口需要传入服务端的参数是活动ID。

传入参数：活动ID。

响应结果：无，通过状态码判断。

接口定义如下：

接口名称：抢券接口

接口路径：**POST**/market/consumer/coupon/seize

![这张图片是抢券接口的请求示例与请求参数说明，对应文档中抢券接口定义的相关内容。上方的请求示例为一个包含"id"字段的JSON结构，值为0。下方的请求参数表格显示，入参为seizeCouponReqDTO，请求类型为body，是必填项，数据类型为SeizeCouponReqDTO，该DTO下包含id参数，说明为活动id，请求类型非必填，数据类型为int64整数。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/R04IbTwJFo33Z5xPgDeccQVrndh.png)

![这张图片展示的是抢券接口的响应参数说明表，表格中列出了该接口的两个响应参数，分别是msg和code。其中，msg为字符串类型，用于表示响应消息；code为int32类型，对应schema为integer(int32)，该参数是状态码，规则为200代表请求成功，其他数值则代表请求失败。这一响应参数说明与文档中介绍的“抢券接口”相关，是对该接口定义的补充说明，明确了该接口返回的具体响应内容及对应含义，可辅助开发者理解该接口的输出格式与返回结果的判断依据。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/RfjdbVtyZos1Zkx3QHJcA5qQnah.png)

编写controller方法：

```Java
@RestController("consumerCouponController")
@RequestMapping("/consumer/coupon")
@Api(tags = "用户端-优惠券相关接口")
public class CouponController {

@PostMapping("/seize")
public void seizeCoupon(@RequestBody SeizeCouponReqDTO seizeCouponReqDTO) {
    
}
．．．
```

#### 2）校验活动有效性

下边定义service方法，在service方法中需要做哪些事？

1. 校验活动是否有效
2. 调用Lua脚本执行抢券

本节实现第一步，校验活动的有效性。

定义 service接口如下：

```Java
public interface ICouponService extends IService<Coupon> {

/**
 * 抢券
 *
 * @param seizeCouponReqDTO
 */
void seizeCoupon(SeizeCouponReqDTO seizeCouponReqDTO);
...
```

实现类：

```JSON
@Override
    public void seizeCoupon(SeizeCouponReqDTO seizeCouponReqDTO) {
        // 1.校验活动开始时间或结束
        // 首先从缓存查询活动
       
        // 2.抢券准备
//         key: 抢券同步队列，资源库存,抢券列表
//         argv：抢券id,用户id
        
        // 3.执行lua脚本进行抢券结果

        // 4.处理lua脚本结果，失败的抛出异常，成功的正常返回

    }
```

如何校验活动是否有效？

1、从缓存中查询指定活动的信息

抢券接口避免与数据库交互。

2、根据活动时间校验活动是否未开始或者已经结束，这两类活动不允许抢券

实现方法如下：

1. 定义从缓存查询指定活动信息的方法

```Java
public interface IActivityService extends IService<Activity> {

/**
 * 从缓存中获取活动信息
 * @param id
 * @return
 */
ActivityInfoResDTO getActivityInfoByIdFromCache(Long id);
```

阅读下边的方法：

```Java
@Override
public ActivityInfoResDTO getActivityInfoByIdFromCache(Long id) {
    // 1.从缓存中获取活动信息
    Object activityList = redisTemplate.opsForValue().get(ACTIVITY_CACHE_LIST);
    if (ObjectUtils.isNull(activityList)) {
        return null;
    }
    // 2.过滤指定活动信息
    List<ActivityInfoResDTO> list = JsonUtils.toList(activityList.toString(), ActivityInfoResDTO.class);
    if (CollUtils.isEmpty(list)) {
        return null;
    }
    // 3.过滤指定活动
    return list.stream()
            .filter(activityInfoResDTO -> activityInfoResDTO.getId().equals(id))
            .findFirst().orElse(null);
}
```

#### 3）抢券service方法

下边编写抢券的service方法

```JSON
@Override
    public void seizeCoupon(SeizeCouponReqDTO seizeCouponReqDTO) {
        // 1.校验活动开始时间或结束
        // 抢券时间
        ActivityInfoResDTO activity = activityService.getActivityInfoByIdFromCache(seizeCouponReqDTO.getId());
        LocalDateTime now = DateUtils.now();
        if (activity == null ||
                activity.getDistributeStartTime().isAfter(now)) {
            throw new CommonException(SEIZE_COUPON_FAILD, "活动未开始");
        }
        if (activity.getDistributeEndTime().isBefore(now)) {
            throw new CommonException(SEIZE_COUPON_FAILD, "活动已结束");
        }

        // 2.抢券准备
//         key: 抢券同步队列，资源库存,抢券列表
//         argv：抢券id,用户id
        int index = (int) (seizeCouponReqDTO.getId() % 10);
        // 同步队列redisKey
        String couponSeizeSyncRedisKey = RedisSyncQueueUtils.getQueueRedisKey(COUPON_SEIZE_SYNC_QUEUE_NAME, index);
        // 资源库存redisKey
        String resourceStockRedisKey = String.format(COUPON_RESOURCE_STOCK, index);
        // 抢券列表
        String couponSeizeListRedisKey = String.format(COUPON_SEIZE_LIST, activity.getId(), index);

        log.debug("seize coupon keys -> couponSeizeListRedisKey->{},resourceStockRedisKey->{},couponSeizeListRedisKey->{},seizeCouponReqDTO.getId()->{},UserContext.currentUserId():{}",
                couponSeizeListRedisKey, resourceStockRedisKey, couponSeizeListRedisKey, seizeCouponReqDTO.getId(), UserContext.currentUserId());
        // 3.抢券结果
        Object execute = redisTemplate.execute(seizeCouponScript, Arrays.asList(couponSeizeSyncRedisKey, resourceStockRedisKey, couponSeizeListRedisKey),
                seizeCouponReqDTO.getId(), UserContext.currentUserId());
        log.debug("seize coupon result : {}", execute);
        // 4.处理lua脚本结果
        if (execute == null) {
            throw new CommonException(SEIZE_COUPON_FAILD, "抢券失败");
        }
        long result = NumberUtils.parseLong(execute.toString());
        if (result > 0) {
            return;
        }
        if (result == -1) {
            throw new CommonException(SEIZE_COUPON_FAILD, "限领一张");
        }
        if (result == -2 || result == -4) {
            throw new CommonException(SEIZE_COUPON_FAILD, "已抢光!");
        }
        throw new CommonException(SEIZE_COUPON_FAILD, "抢券失败");
    }
```

#### 4）抢券controller方法

```JSON
@ApiOperation("抢券接口")
@PostMapping("/seize")
public void seizeCoupon(@RequestBody SeizeCouponReqDTO seizeCouponReqDTO) {
    couponService.seizeCoupon(seizeCouponReqDTO);
}
```



#### 5) 抢券测试



启动网关

启动优惠券活动管理工程

启动xxl-job

打开小程序，进入抢券页面

![图片展示的是小程序抢券页面。上方显示“抢券”标题，左侧有返回箭头，右侧有三个点和放大镜图标。中间有“疯抢中”和“即将开始”两个状态选项。下方有一个红色背景的优惠券图标，显示“8.0折”及“立即领取”按钮，下方文字说明“双12来啦 满100元立享8折 2023-11-06~2023-11-09”。该图片与上文“打开小程序，进入抢券页面”对应，直观呈现了抢券页面的样式及信息。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/QockbsVY1opmZ9x14ffcRl6snnb.png)

点击领取，观察写入redis的数据是否正确：

示例：

抢券成功存入redis抢券成功队列：

![图片展示的是Redis界面中关于抢券成功存入redis抢券成功队列的数据。键值对为“COUPON:SEIZE:LIST:1721415300848590848_{8}”和“1716346406098296832”，对应的value分别为“1”和“1”。该图片与上文“点击领取，观察写入redis的数据是否正确”内容相关，用于直观呈现抢券成功存入redis抢券成功队列时的数据存储情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/GOfbbMFDjoLqpzxdbq5cxBbDnOd.png)

存入redis抢券同步队列：

![这是抢券测试环节中相关Redis数据的展示界面，界面的HASH标识标注键为“QUEUE:COUPON:SEIZE:SYNC:{8}”，对应抢券同步队列的信息。界面内的表格区域显示编号为1的条目，其key为数值串1716346406098296832，value为数值串1721415300848590848，该数据为测试抢券功能时，存入Redis的抢券成功相关同步队列数据，用于验证抢券流程的数据存储是否符合预期。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NYuRbGjzbo2X9pxCgd5cE5LNnUe.png)

抢券失败情况测试：

限领一张提示：

![图片展示的是抢券页面。上方显示“抢券”标题，左侧有“疯抢中”“即将开始”选项，当前选中“疯抢中”。中间区域有“双12来啦 满100元立享8折 2023-11-06~2023-11-09”字样，下方有一个橙色的“立即领取”按钮。底部有一个灰色的“限领一张”按钮。该图片与文档中“抢券测试”部分相关，用于展示抢券页面的样式及部分功能按钮。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/KwkqbCMgCo8So2xJdmTcDlmOnVf.png)

已抢光：

手动修改redis库存为0，进行测试，示例如下：

![图片展示的是一个抢券页面。上方显示“抢券”标题，左侧有“即将开始”字样，右侧有三个点和一个圆圈图标。下方有两张券，上方券为“双12大促”券，面额10元，满100元立减10元，有效期2023 - 12 - 15 - 2023 - 12 - 24，可立即领取；下方券为“平台二周年庆”券，9折，有效期2023 - 12 - 5 - 2023 - 12 - 24，已抢光。该图片与文档中抢券测试中已抢光情况的测试内容相关，用于展示已抢光券的状态。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/KbVPbZ9d4olOJtxT4sacdMl6nnc.png)



#### 6）小结

项目是怎么实现抢券功能的？

1）将优惠券活动的库存同步到Redis

2)  用户抢券请求redis，执行Lua脚本，具体如下：

先判断当前用户是否抢过该优惠券，如果抢过则返回-1

判断库存是否充足，如果不充足返回-2

向抢券成功列表写入记录

扣减库存

向抢券同步列表写入记录

3）由异步任务将redis抢券成功记录同步到数据库中





## 2.4 抢券结果同步

### 2.4.1 Redis到MySQL同步方案分析

如何将Redis中的抢券结果同步到MySQL的优惠券表（coupon）呢？

#### 1） **整体思路**

基本思路: 遍历Redis中的抢券结果同步队列，拿到一个元素就向数据库的优惠券表插入记录，插入完成后删除Redis中的这条记录。

下图是存储抢券结果同步数据的Hash表。

![图片展示的是Redis中一个名为QUEUE:COUPON:SEIZE:SYNC:8的Hash表内容。其中，key为1716346406098296832，value为1721415300848590848。图片与上下文紧密相关，上下文在分析Redis到MySQL同步方案时提到，可从Hash表中拿一批数据，每个元素包括用户id和活动id，根据这两个参数插入coupon表，此图直观呈现了Hash表中存储的用户id和活动id数据，辅助说明同步操作的数据来源。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EeJEbZ35voLJ1yxxYLychHQSntf.png)

我们可以一次从Hash表中拿一批数据，每个元素包括了用户id和活动id，根据这两个参数插入coupon表。

从下图可以看出，只要拿到用户id和活动id即可向优惠券表插入一条记录。

![图片展示了jz020 - market数据库中coupon表的结构。表包含id、name、user_id、user_name、user_phone、activity_id、type、discount_rate、discount_amount、amount_condition、validity_time、use_time、status、orders_id、create_time、update_time、is_deleted等字段。其中，user_id、user_name、user_phone、activity_id、type、discount_rate、discount_amount、amount_condition、validity_time、use_time、status、orders_id等字段被红色框突出显示，这些字段与用户信息及优惠券活动信息相关，是根据上下文提到的Redis到MySQL同步方案中从Hash表拿数据后插入coupon表时所需的关键字段。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/OHNybrtouoiSGDxtnJnc3kDvn8f.png)

基本思路清楚，现在需要考虑：系统有多个活动，如何提高同步程序的处理性能呢？

假如同步队列的key为：QUEUE:COUPON:SEIZE:SYNC:{活动id % 10}，这说明最多有10个同步列表。

我们可以用多线程，每个线程处理一个同步队列。

<whiteboard token="IgcywG7iJh0HyzbTC9gcOW8Snre"></whiteboard>

由定时任务去调度，每隔1分钟由多线程对同步队列中的数据进行处理。



#### 2）**如何从Redis 批量取数据？**

我们使用redisTemplate.opsForHash().scan(H key, ScanOptions options)方法，scan方法通过游标的方式实现从hash中批量获取数据。

测试代码如下：

```JSON
@Test
void batchGet() {
    int index = 6;
    String hashKey = String.format("QUEUE:COUPON:SEIZE:SYNC:{%s}",index);
    Cursor<Map.Entry<String, Object>> cursor = null;
    // 通过scan从redis hash数据中批量获取数据，获取完数据需要手动关闭游标
    ScanOptions scanOptions = ScanOptions.scanOptions()
            .count(10)
            .build();
    try {
        // sscan获取数据
        cursor = redisTemplate.opsForHash().scan(hashKey, scanOptions);
        // 遍历数据转换成SyncMessage列表
        List<SyncMessage<Object>> collect = cursor.stream()
                .map(entry -> SyncMessage
                        .builder()
                        .key(entry.getKey().toString())
                        .value(entry.getValue())
                        .build())
                .collect(Collectors.toList());
        log.info("获取{}数据{}条", hashKey,collect.size());
        collect.stream().forEach(System.out::println);
    }catch (Exception e){
        log.error("同步处理异常，e:", e);
        throw new RuntimeException(e);
    } finally {
        // 关闭游标
        if (cursor != null) {
            cursor.close();
        }
    }
}
```

下边进行测试：

保证存在“QUEUE:COUPON:SEIZE:SYNC:{%s}”的队列且其中有数据

观察控制台的日志"{}获取{}队列的数据{}条"，并观察是否输出了从Hash表中拿到的数据。



#### 3）小结

本节我们分析了从Redis到MySQL同步方案：

1、使用线程池从多个同步队列中查询数据，每个线程处理一个同步队列。

注意：同步队列的个数可以灵活配置，但不宜过多，因为同步队列个数为最大线程数，通常配置10到20即可。

2、使用redisTemplate.opsForHash().scan(H key, ScanOptions options)方法从hash表获取数据。

这里需要注意游标的使用，一定要在finally 中关闭游标。





### 2.4.2 抢券结果同步开发

#### 1）编写扣减库存方法

项目使用数据同步组件完成数据从Redis的Hash结构同步到MySQL中。

关于数据同步组件的使用前边已经讲解，下边我们需要完善数据同步处理器即可将抢券同步队列的数据同步到MySQL。

数据同步处理器拿到抢券结果做两件事：

- 插入优惠券表
- 扣减库存

**首先编写扣减库存的方法**

此方法已存在

```JSON
public interface IActivityService extends IService<Activity> {

/**
 * 扣减库存
 * @param id 活动id
 *  如果扣减库存失败抛出异常
 */
public boolean updateStockNum(Long id,Integer num);
...
```

实现方法：

```JSON
@Service
public class ActivityServiceImpl extends ServiceImpl<ActivityMapper, Activity> implements IActivityService {

/**
 * 扣减库存
 * @param id 活动id
 *  如果扣减库存失败抛出异常
 */
@Override
public boolean updateStockNum(Long id,Integer num) {
    //sql=update activity set stock_num=stock_num-#{num} where id=#{activityId} and stock_num>=#{num}
    boolean update = lambdaUpdate()
            .setSql("stock_num=stock_num-"+num)
            .eq(Activity::getId, id)
            .ge(Activity::getStockNum,num)
            .update();
    return update;
}
...
```

#### 2) **编写数据同步方法**

每从hash中获取一条数据则向coupon表插入记录。

```Java
public interface ICouponService extends IService<Coupon> {

    ...

    /**
     * 抢券结果同步
     * @param activityId
     * @param userId
     */
    void seizeCouponSync(Long activityId, Long userId);
```

实现类：

```Java
@Transactional
public void seizeCouponSync(Long activityId, Long userId){
    // 1.获取活动
    Activity activity = activityService.getById(activityId);
    if (activity == null) {
        return;
    }
    CommonUserResDTO commonUserResDTO = commonUserApi.findById(userId);
    if(commonUserResDTO == null){
        return;
    }
    // 2.新增优惠券
    Coupon coupon = new Coupon();
    coupon.setId(IdUtils.getSnowflakeNextId());
    coupon.setActivityId(activityId);
    coupon.setUserId(userId);
    coupon.setUserName(commonUserResDTO.getNickname());
    coupon.setUserPhone(commonUserResDTO.getPhone());
    coupon.setName(activity.getName());
    coupon.setType(activity.getType());
    coupon.setDiscountAmount(activity.getDiscountAmount());
    coupon.setDiscountRate(activity.getDiscountRate());
    coupon.setAmountCondition(activity.getAmountCondition());
    coupon.setValidityTime(DateUtils.now().plusDays(activity.getValidityDays()));
    coupon.setStatus(CouponStatusEnum.NO_USE.getStatus());

    boolean save = save(coupon);
    //扣减库存
    boolean b = activityService.updateStockNum(activity.getId(), 1);
    if(!b || !save){
        throw new CommonException("优惠券同步失败");
    }
}
```



#### 3) 编写线程任务类

```Java
package com.jzo2o.market.handler;

import com.jzo2o.common.utils.NumberUtils;
import com.jzo2o.market.service.ICouponIssueService;
import com.jzo2o.market.service.ICouponService;
import com.jzo2o.redis.model.SyncMessage;
import com.jzo2o.redis.utils.RedisSyncQueueUtils;
import lombok.extern.slf4j.Slf4j;
import org.redisson.api.RLock;
import org.redisson.api.RedissonClient;
import org.springframework.data.redis.core.Cursor;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ScanOptions;

import java.util.List;
import java.util.Map;
import java.util.concurrent.TimeUnit;
import java.util.stream.Collectors;

/**
 * @author Mr.M
 * @version 1.0
 * @description 抢券结果处理器
 * @date 2024/9/23 18:43
 */
@Slf4j
public class SeizeCouponHandler implements Runnable {

    //hashkey
    private String hashKey;


    //分布式锁
    private RedissonClient redissonClient;

    private RedisTemplate redisTemplate;

    private ICouponService couponService;


    //构造方法
    public SeizeCouponHandler(String hashKey, ICouponService couponService,RedissonClient redissonClient, RedisTemplate redisTemplate) {
        this.hashKey = hashKey;
        this.couponService = couponService;
        this.redissonClient = redissonClient;
        this.redisTemplate = redisTemplate;

    }

    @Override
    public void run() {
        String lockKey = "LOCK:" + hashKey;
        log.info("获取锁：{}", lockKey);
        RLock lock = redissonClient.getLock(lockKey);
        //尝试获取锁
        try {
            boolean tryLock = lock.tryLock(1, -1, TimeUnit.SECONDS);
            if (!tryLock) {
                log.info("获取锁失败：{}", lockKey);
                return;
            }
            //开始发放优惠券
            log.info("开始处理抢券结果：{}", hashKey);
            Cursor<Map.Entry<String, Object>> cursor = null;
            try {
                // 通过scan从redis hash数据中批量获取数据，获取完数据需要手动关闭游标
                ScanOptions scanOptions = ScanOptions.scanOptions()
                        .count(100)
                        .build();
                while (true){
                    // sscan获取数据
                    cursor = redisTemplate.opsForHash().scan(hashKey, scanOptions);
                    // 遍历数据转换成SyncMessage列表
                    List<SyncMessage<Object>> collect = cursor.stream()
                            .map(entry -> SyncMessage
                                    .builder()
                                    .key(entry.getKey().toString())
                                    .value(entry.getValue())
                                    .build())
                            .collect(Collectors.toList());
                    log.info("{}获取{}数据{}条", Thread.currentThread().getId(), hashKey, collect.size());
                    if(collect.size() <=0){
                        break;
                    }
                    //处理抢券结果
                    collect.stream().forEach((objectSyncMessage) -> {
                        try {
                            //活动id
                            Long activityId = NumberUtils.parseLong(objectSyncMessage.getValue().toString());
                            //用户id
                            Long userId = NumberUtils.parseLong(objectSyncMessage.getKey());
                            //同步抢券结果
                            couponService.seizeCouponSync(activityId, userId);
                            //删除同步记录
                            this.redisTemplate.opsForHash().delete(hashKey, new Object[]{objectSyncMessage.getKey()});
                        } catch (Exception var5) {
                            log.error("hash结构同步消息单独处理异常，e:", var5);
                        }

                    });


                }


            } catch (Exception e) {
                log.error("同步处理异常，e:", e);
                throw new RuntimeException(e);
            } finally {
                lock.unlock();
                // 关闭游标
                if (cursor != null) {
                    cursor.close();
                }
            }


        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }

    }
}
```



#### 4）编写多线程任务方法

```Java
package com.jzo2o.market.handler;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.jzo2o.market.constants.RedisConstants;
import com.jzo2o.market.model.domain.Activity;
import com.jzo2o.market.service.IActivityService;
import com.jzo2o.market.service.ICouponIssueService;
import com.jzo2o.market.service.ICouponService;
import com.jzo2o.redis.properties.RedisSyncProperties;
import com.jzo2o.redis.utils.RedisSyncQueueUtils;
import lombok.extern.slf4j.Slf4j;
import org.redisson.api.RedissonClient;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.time.LocalDateTime;
import java.util.List;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * @author Mr.M
 * @version 1.0
 * @description 抢券结果同步任务
 * @date 2024/9/23 19:49
 */
@Slf4j
@Component
public class SeizeCouponHandlerJob {
    //定义线程池
    private static ThreadPoolExecutor threadPoolExecutor;

    @Resource
    private RedisTemplate redisTemplate;

    @Resource
    private ICouponService couponService;

    @Resource
    private RedissonClient redissonClient;

    @Resource
    private RedisSyncProperties redisSyncProperties;

    static {
        threadPoolExecutor = new ThreadPoolExecutor(10, 10, 60, TimeUnit.SECONDS, new ArrayBlockingQueue<>(10));
    }

    public void start() {
        int queueNum = redisSyncProperties.getQueueNum();
        log.info("根据队列创建线程任务对象");
        for (int i = 0; i < queueNum; i++) {
            String hashKey = RedisSyncQueueUtils.getQueueRedisKey(RedisConstants.RedisKey.COUPON_SEIZE_SYNC_QUEUE_NAME, i);

            //创建SeizeCouponHandler对象
            SeizeCouponHandler seizeCouponHandler = new SeizeCouponHandler(hashKey, couponService,redissonClient, redisTemplate);
            //将任务加入线程池
            threadPoolExecutor.execute(seizeCouponHandler);
        }

    }
}
```





#### 5）抢券结果同步测试

编写定时任务

```Java

package com.jzo2o.market.handler;

import com.jzo2o.market.service.IActivityService;
import com.jzo2o.market.service.ICouponService;
import com.jzo2o.redis.annotations.Lock;
import com.jzo2o.redis.constants.RedisSyncQueueConstants;
import com.jzo2o.redis.sync.SyncManager;
import com.xxl.job.core.handler.annotation.XxlJob;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;

import static com.jzo2o.market.constants.RedisConstants.Formatter.*;
import static com.jzo2o.market.constants.RedisConstants.RedisKey.COUPON_SEIZE_SYNC_QUEUE_NAME;

@Component
public class XxlJobHandler {
    ...
    /**
     * 抢券同步队列
     * 5s一次
     */
    @XxlJob("seizeCouponSyncJob")
    public void seizeCouponSyncJob() {
        seizeCouponHandlerJob.start();
    }

}
```

下边在xxl-job配置定时任务

![这是一张xxl-job的定时任务新增配置页面，属于抢券结果同步任务的配置界面。页面包含基础配置、调度配置、任务配置、高级配置四个区域，其中执行器设置为“营销中心”，负责人填写“xxljob”，任务描述为“抢券结果同步任务”；调度类型选择“CRON”，Cron表达式设置为“0/10 * * * * ?”；运行模式设为“BEAN”，对应的JobHandler为“seizeCouponSyncJob”，路由策略为“第一个”，调度过期策略设为“忽略”，阻塞处理策略为“单机串行”，界面底部设有“保存”和“取消”操作按钮。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/IUatbDijHoIpmqxGQeYc2W7xnUe.png)

启动任务：

![该图片展示了xxl-job配置的抢券结果同步任务信息，任务ID为18，任务描述为抢券结果同步任务。该任务采用CRON调度，表达式为0/10 * * * * ?，运行模式为BEAN模式，绑定的任务Bean是seizeCouponSyncJob，负责人为xxljob，任务当前状态为RUNNING，可通过对应操作按钮进行相关操作。这一配置对应文档中抢券结果同步环节里启动xxl-job抢券结果同步任务的要求，用于执行抢券结果的数据同步操作。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NIjTbVsU9ond1ZxLyzDciIYqnOf.png)

测试流程：

启动客户管理工程（需要根据用户id查询抢券人的信息）

抢券测试，保证抢券成功后在Redis同步队列写入数据成功

示例：

![图片展示的是Redis操作界面，显示了名为“QUEUE:Coupon:SEIZE:SYNC:{4}”的Hash键，其下有一个key为1716346406098296832的键，对应的value为1721427028349067264。该图片与文档中“抢券结果同步开发”部分的“编写多线程任务方法”内容相关，可能是用于展示抢券结果同步过程中Redis中相关数据的存储情况，辅助说明数据同步操作。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Xqg0bEXKvosqxNxe6Obc2m87ncb.png)



启动xxl-job抢券结果同步任务

跟踪断点，观察数据是否正确:

数据同步成功删除同步 HashKey

最终写入优惠券表并扣减库存。



示例：

查询coupon表，发现用户抢券结果已成功写入优惠券表：

![图片展示了一条优惠券数据信息，包含用户ID、用户名、活动ID等字段。其中，用户ID为1716346406098296832，用户名是“普通用户72135”，活动ID为1721427028349067264。该图片与文档中抢券结果同步测试内容相关，用于示例说明抢券成功后在Redis同步队列写入的数据情况，以及最终写入优惠券表并扣减库存后，查询coupon表发现用户抢券结果已成功写入优惠券表的情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/YE8NbuYA0ooYm3xMeXOc8cc4n3f.png)

活动的库存数据正常扣除。

抢券结果同步完成，redis中记录的活动的库存与数据库活动表的库存一致：

![图片展示的是Redis中“COUPON:RESOURCE:STOCK:4”键对应的Hash数据结构内容。其中包含两条记录，key分别为1720700051954622464和1721427028349067264，对应的value分别为100和0。该图片与文档中“抢券结果同步完成，redis中记录的活动的库存与数据库活动表的库存一致”的上下文对应，直观呈现了Redis中库存数据的存储情况，辅助说明库存数据同步的正确性。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HmoIb7qUsoovkBxjRTAcGN7knCc.png)

![这张图片展示了与第十章秒杀抢购相关的抢券结果数据，是抢券流程完成后的查询结果信息。图片中显示了优惠券活动的相关字段，其中"total_num"对应活动总库存为10000，"stock_num"即当前剩余库存，被红色箭头突出标注，数值为0，结合上下文可知这是抢券结果同步完成后，数据已正确写入优惠券表并成功扣减对应库存的验证内容，该结果也符合抢券后Redis同步队列数据正常、活动库存与数据库一致的预期要求。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/AYyGbccnpox5ttx1OU5c9asGnhb.png)



#### 6）小结

秒杀异步处理怎么实现的？

- 活动预热

  - 活动列表
  - 库存
- lua脚本实现抢券

  - 判断用户是否抢过这个券-通过Redis抢券成功列表判断是否抢过
  - 判断库存是否充足-通过Redis库存key判断
  - 添加抢券列表
  - 加库存
  - 添加同步队列（10个）
- 将数据同步到mysql中

  - 对每个同步队列单独起一个线程（10个线程）
  - 对每个同步队列加分布式锁
  - 循环使用redis游标查询当前同步队列的数据（查100条）
  - 保存到优惠券表
  
    - 查询活动表
    - 远程调用用户服务查询用户信息
  - 扣减库存
  
    - 更新活动表的库存
  - 删除redis同步队列数据

# 3 抢单

## 3.1 需求分析

完成了预约下单、支付业务的开发，接下来进入抢单模块。

首先我们对抢单模块进行需求分析。

### 1）核心业务流程

首先明确本模块在核心业务流程的位置，下图是项目的核心业务流程：

用户下单后服务人员通过app进行抢单，机构通过pc进行抢单，抢单成功后服务人员开始现场服务。

<whiteboard token="BRHcwlFoYhXv2Abise5cLKjvn5c"></whiteboard>

### 2）服务端抢单原型

服务人员在平台注册后需要进行实名认证、服务技能设置、服务范围设置、开启接单，这四项准备工作完成后方可在平台接单。

1. **服务技能设置**

服务技能即服务人员服务范围，比如：服务人员A提供日常保洁、日常维修服务。设置服务技能后服务人员可以抢与自己的技能匹配的订单，比如：服务人员A提供日常保洁、日常维修服务，所以它只能抢日常保洁、日常维修服务的订单。

![这张图展示的是服务技能设置的操作界面，对应文档中抢单功能里的服务技能设置内容。图中呈现了两个状态的界面，左侧界面是“选择服务技能”页面，标注了编辑状态，包含保洁清洗（下分日常保洁、深度保洁、开荒保洁、周期保洁）、日常维修两类服务选项；右侧界面为设置完成状态，“日常保洁”和“深度保洁”选项被标红选中，顶部右上角显示“保存”字样，清晰呈现了选择服务技能的操作过程，直观体现了服务技能设置中挑选对应服务项的操作逻辑。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/F9RRbCRbEoQEjXxwAA9cy7oCnKe.png)



1. **服务范围设置**

设置服务范围后能抢服务范围内的订单，下图中的服务城市是北京，接单范围是朱辛庄地铁站，在抢单查询时以接单范围为中心点查询方圆几公里的用户创建的订单，具体方圆公里数在查询条件中选择。

![这是服务范围设置相关的工作地点选择页面截图，页面顶部显示“请选择工作地点”，地图上标注了当前定位的可修改地址，下方明确显示服务城市为北京，意向接单范围为朱辛庄地铁站，还设有“重新定位”按钮与“保存”按钮，该页面对应文档中服务范围设置的相关说明，设置完成后接单时会以该接单范围为中心点，结合用户所选的查询距离获取对应订单。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EnWqbezsQoEFKbx2L3ycxKbvnIc.png)





1. **接单设置**

开启接单后方可抢单。

![这是接单设置页面的界面截图，页面顶部显示“接单设置”标题，左侧带有返回箭头。页面设有“开启接单”的开关选项，当前该开关呈灰色关闭状态，表明接单功能尚未开启，这与文档中“开启接单后方可抢单”的说明内容相对应。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/RidjbdqvyoheovxwkH5ccrxmnNe.png)

1. **开始抢单**

进入抢单页面，选择服务距离和服务类型查询订单，点击“立即抢单”进行抢单。

服务距离：距离服务人员接单范围的距离。

服务类型：

![这是云岚到家服务人员抢单页面的相关界面，顶部区域标注为北京，设有服务技能、接单设置、服务范围选项，能设置接单的服务城市与具体范围。页面中部标注“抢单大厅”，包含服务距离与服务类型选项；服务距离选择区域被标注，提供全部、3公里内、5公里内等不同范围选项，可筛选出对应接单范围内的订单。页面下方展示了日常保洁、空调维修两类订单，均标注有预约时间、服务地址、费用，还设有“立即抢单”按钮，点击即可参与对应订单的抢单操作。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/UWG8b0iuKo49A6xPWQnc9ypinGE.png)

1. 抢单成功查询订单

服务人员抢单成功在我的订单中查询

![图片展示的是抢单成功后在“我的订单”中查询到的订单信息页面。页面显示时间为9:41，有“订单”“历史订单”选项。订单状态为“待上门”，有“开始服务”按钮。订单内容为冰箱维修，预约时间为2023年5月30日19:00，地址是北京市昌平区金彦龙写字楼*单元。页面左上角有红色数字“2”，右上角有红色数字“99+”。该图片与抢单成功后在“我的订单”中查询订单的上下文对应，直观呈现了订单查询结果。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/UUQ9baiDmoUyAhxH6accStaBn5d.png)



### 3) 机构端抢单原型

机构端抢单与服务端抢单的区别是：

1、机构使用pc端，服务端使用app

2、机构端抢单成功后订单状态为待分配，表示待将订单分配给机构下的服务人员；服务端抢单成功后订单状态为待服务，因为服务人员抢到的订单服务人员就是自己。

3、抢单数量限制不同

抢单数量限制是当前拥有的进行中的服务单的最大数量。服务人员默认是10，机构默认是100。

抢单数量限制的目的是规避恶意抢单。

因为机构下的服务人员较多所以机构的签单数量限制大于服务人员。



下边说明机构端抢单的界面原型：

机构端也需要进行实名认证、服务技能与服务范围设置、开启接单后方可在平台接单。

**1、服务技能与服务范围设置**

![图片展示了机构端抢单中服务技能与服务范围设置的界面。左侧为服务技能设置，有“保洁”分类，可选择“日常保洁”“深度保洁”；右侧是服务范围设置，有“城市选择”“服务地址”输入框，下方是地图，显示了服务范围区域。该图片与上下文“服务技能与服务范围设置”内容对应，直观呈现了设置操作界面，帮助理解机构端抢单时对服务技能和服务范围的配置方式。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HXfubBlNFooDjxxLKXjcASJnncK.png)

**2、开启接单**

![这张图片展示的是秒杀抢购模块里机构端抢单的相关设置界面，界面包含四项设置内容，分别为实名认证、服务技能设置、服务范围设置和接单设置，每项设置旁配有对应的功能按钮，实名认证、服务技能设置、服务范围设置的右侧分别设有“去认证”“去设置”按钮，接单设置右侧的开关为开启状态，红色箭头明确指向该开启的开关，表明此为开启接单的状态，界面底部设有“进入首页”和“进入首页”两个操作按钮，该界面对应机构端抢单原型中的服务技能与服务范围设置、开启接单环节。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/S4Dob2RIJoyBfPxrE95cfStbnab.png)

**3、抢单页面**

![图片展示的是抢单页面。页面上方有搜索框，可输入关键词搜索服务技能、服务范围等。下方是抢单列表，包含序号、服务类型、服务名称、服务数量、预计时间、服务费用、服务地址等信息，每条记录右侧有“抢单”按钮。页面右下角有“下一页”按钮。该图与上下文介绍的抢单需求分析中抢单页面内容对应，直观呈现了抢单页面的样式及信息展示情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/RMf9bG7oQoq1R5xVG8ac55NHn8d.png)

上图中除了服务费用以外其它字段均为订单表中的信息，服务费是订单金额的30%（可设置）。



### 4）服务单状态

服务人员抢单成功或系统派单成功将生成订单对应的服务单，服务单状态如下：

<whiteboard token="Q09VwBFUThKijPble57cn0ZynKQ"></whiteboard>

服务单初始状态：待分配或待服务

机构抢单成功：待分配。

服务人员抢单成功：待服务。



开始服务：  
待服务---》服务中



服务完成：

服务中---》服务完成



用户取消订单：

待分配---》已取消

待服务---》已取消

运营人员取消订单：

服务中---》已取消

服务完成---》已取消



### 5) 小结

通过需求分析，抢单模块分如下子模块：

1. 抢单设置子模块

设置服务技能、服务范围、开启抢单。

1. 抢单查询子模块

查询抢单池中的订单

1. 抢单子模块

点击“立即抢单”开始抢单。



抢单成功后生成服务单，服务单的状态包括：待分配、待服务、服务中、服务完成、已取消。



## 3.2 系统整体设计

### 1）整体分析

根据抢单需求，抢单的流程和抢券的流程很相似，核心内容也包括三部分：

#### **抢单查询**

**抢单查询对应到抢券查询**

在抢单界面查询抢单池中的订单，订单属于抢购的资源，在抢券中优惠券也属于争抢的资源，此处的查询存在高并发需要使用缓存技术进行优化。

查询抢单池中的订单可以根据下单用户的地理位置进行搜索订单，此功能类似“搜索附近”的业务，比如：搜索附近的酒店、搜索附近的房源信息等。

结合上边的要求，本项目使用Elasticsearch查询抢单资源，使用ES基于地理坐标搜索功能完成“搜索附近”功能的开发。

如果要使用Elasticsearch查询抢单池信息就需要将待派单的订单同步到Elasticsearch，这里我们使用Canal+MQ的方式实现数据同步，如下图：

<whiteboard token="QVjjwzGJ9hNtKpb3T6eccCbmnoe"></whiteboard>

说明：

下单支付成功通过订单分流程序将订单信息写入抢单池，满足系统自动派单需求的订单同时写入派单池，派单部分在后边章节讲解。

通过Canal将抢单池的信息同步到Elasticsearch。

抢单查询从Elasticsearch中查询订单信息，使用ES的基于地理坐标搜索功能完成搜索附近的订单。



#### **抢单**

抢单**技术方案同抢券**

我们在分析抢单技术方案时结合 抢券 的技术方案进行分析，多个人争抢同一个订单，这里仍然存在超卖问题，我们使用同抢券一样的技术方案去解决超卖问题。

下图中加入了抢单的交互流程：

<whiteboard token="MJ2kwRBaMhWctXbqnJ1cdv8Xnte"></whiteboard>

说明：

参考抢券的方案，使用Redis+Lua的技术解决超卖问题，这里将抢单库存同步到Redis，每个订单的库存就是1。

抢单执行Lua脚本完成抢单，具体包括：扣减库存、抢单成功写入同步队列。

抢单同步队列的作用是通过异步任务将抢单结果信息同步到数据库。



#### **抢单结果异步处理**

**技术方案同抢券**

秒杀抢购业务的并发高，为了避免直接操作数据库这里使用异步任务的方式将抢单结果同步到数据库。

抢单成功创建服务单，异步任务的主要职责是根据抢单结果创建服务单，并且更新订单的状态。

抢单结果同步异步任务的具体的内容如下：

**创建服务单**：

服务单记录了服务人员进行家政服务的信息，关键字段有：订单ID、订单金额、服务人员ID、服务单状态、服务时间、服务照片等。

服务单初始状态：待分配或待服务

机构抢单成功：待分配。

服务人员抢单成功：待服务。

服务单的详细状态如下图：

<whiteboard token="UItBwvTKwhlDL6bL8Qvc6o0znre"></whiteboard>



**更新订单的状态：**

用户下单并支付完成后订单的状态为“派单中”，服务人员抢单成功订单状态为“待服务”，机构抢单成功订单状态为“待分配”。

订单状态图如下，详细说明请参见第四章内容。

<whiteboard token="NkBtw9fAshAct3b6yXcc7ZUZnpd"></whiteboard>



**抢单结果同步成功删除抢单池等相关信息：**

抢单结果同步成功后删除抢单池中该订单的信息，这样服务人员在抢单界面无法查询到该订单。

删除数据库中抢单池的记录，将Elasticsearch中对应的抢单记录删除。

删除Redis中该订单的库存信息。

删除Redis中抢单同步队列中的记录。

交互流程如下：

<whiteboard token="DHuhwvkC0hl4qnbZvjbcl8lanCc"></whiteboard>



### 2)  数据流

下边分析整体的数据流：

具体的表结构在各个模块中进行设计。

<whiteboard token="L76dwrl93hzCabbyIXccK4OCnTd"></whiteboard>

说明：

1、下单

向订单表添加记录。

2、支付成功

更新订单状态为派单中

向抢单池表新增记录

3、Canal同步

将数据库中抢单池的信息同步到Elasticsearch中。

4、抢单查询

请求Elasticsearch查询抢单池信息

5、抢单

请求Redis扣减库存、添加抢单同步记录

6、抢单结果异步处理

创建服务单

更新订单状态

删除抢单池记录

删除Redis同步队列记录

删除Redis库存记录



### 3）小结

能够说出本项目抢单的实现方案？

1. 用户下单且支付成功，订单信息进入抢单池
2. 通过Canal+MQ将抢单池的信息同步到Elasticsearch、同步库存到Redis中。
3. 服务人员或机构进入抢单界面，请求Elasticsearch查询订单信息，这里可以根据地理坐标搜索附近的订单
4. 服务人员或机构点击抢单、请求Redis执行Lua脚本完成抢单。
5. 通过定时任务将Redis中抢单成功的结果同步到数据库。删除抢单池、删除redis同步队列、删除库存



## 3.3 抢单设置

根据需求，机构和服务人员都需要做抢单前的准备工作方可抢单。

准备工作包括：实名认证、服务技能设置、服务范围设置、开启接单。关于实名认证的业务流程请参考第二章，下边通过阅读代码理解服务技能设置、服务范围设置、开启接单的设置。

下边以服务端为例说明。

### 1）需求分析

1. **服务技能设置**

服务人员新注册账号后进入下边的界面：

![图片展示的是服务人员新注册账号后进入的界面。界面上方有“亲，完成以下设置，方可接单哦~”的提示，下方列出需设置的项目：实名认证、服务技能设置、服务范围设置、接单设置，每个项目右侧对应有“去认证”“去设置”按钮。最下方有一个“进入首页”的粉色按钮。该图片与上下文紧密相关，上下文提到通过此界面进行服务技能设置、服务范围设置、接单设置，且点击“去设置”可进入服务技能设置界面，点击“去认证”“去设置”可进行相应设置。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HS8lbfk2bo6rdVxvWeNcTNY4nwe.png)

通过此界面进行服务技能设置、服务范围设置、接单设置。

点击上图中的“去设置”进入服务技能设置界面：

![图片展示的是抢单服务技能设置界面中的“选择服务技能”部分。界面上方有“编辑”按钮，右侧有“保洁清”“日常维修”“家居服务”等服务技能选项，其中“保洁清”有红色边框突出显示，且有数字“2”标注。该图片与上文“点击编辑开始设置服务技能”及“选择服务技能，点击保存”的操作说明相关，直观呈现了服务技能选择界面，帮助用户了解操作位置及技能选项。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NAuKbCfDPosc7AxxNrycaK4jnrf.png)

点击编辑开始设置服务技能：

![图片展示的是抢单设置中服务技能设置界面。界面上方显示“选择服务技能”，左侧有“保洁清”“日常维修”“家居服务”等服务类别，其中“保洁清”类别下有“日常保洁”“厨卫维修”两个选项，且“日常保洁”被红色框线突出显示。下方有“取消”和“保存”按钮。该图片与上文抢单设置中服务技能设置的操作流程相关，直观呈现了服务技能选择界面及关键选项。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/UJXJb98KToXMNRxAIi4cuyl2nxf.png)

选择服务技能，点击保存。

1. **服务范围设置**

进入服务范围设置界面：

![图片展示的是抢单服务范围设置界面。界面上方显示“请选择工作地点”，下方有两个设置项，分别是“服务城市”和“意向接单范围”，当前分别显示为“北京市”和“北京黑马程序员”，右侧均有向右的箭头标识。界面底部有一个红色的“保存”按钮。该图片与抢单设置上下文相关，是进入服务范围设置界面后的呈现，用于设置抢单时的服务城市和意向接单范围。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XmgxbCKi3oP0lbxlpVica0h7nHb.png)





### 2）准备环境

抢单章节的代码环境：

jzo2o-customer

jzo2o-fundations

jzo2o-orders



订单工程创建一个新分支dev_02，从课程资料中找到并解压jzo2o-orders-seize-dev02.zip、jzo2o-orders-manager-dev02.zip，覆盖dev_02分支的代码。



### 3) 表设计

服务技能表：存储服务提供者的技能信息。

结构如下：

```JSON
create table `jzo2o-customer`.serve_skill
(
    id                  bigint                             not null comment '主键'
        constraint `PRIMARY`
        primary key,
    serve_provider_id   bigint                             null comment '服务人员/机构id',
    serve_provider_type int                                null comment '类型，2：服务人员，3：服务机构',
    serve_type_id       bigint                             null comment '服务类型id',
    serve_type_name     varchar(50)                        null comment '服务类型名称',
    serve_item_id       bigint                             null comment '服务项id',
    serve_item_name     varchar(50)                        null comment '服务项名称',
    create_time         datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    update_time         datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    is_delete           int      default 0                 not null comment '是否已删除，0：未删除，1：已删除'
)
    comment '服务技能表' charset = utf8mb4;
```



服务提供者设置表：存储服务提供者的服务范围等信息。

```JSON
create table `jzo2o-customer`.serve_provider_settings
(
    id              bigint                                not null comment '服务人员/机构id'
        constraint `PRIMARY`
        primary key,
    city_code       varchar(20) default ''                null comment '城市码',
    city_name       varchar(64)                           null comment '城市名称',
    lon             double(10, 5)                         null comment '经度',
    lat             double(10, 5)                         null comment '纬度',
    intention_scope varchar(100)                          null comment '意向单范围',
    have_skill      int         default 0                 null comment '是否有技能',
    can_pick_up     int         default -1                null comment '是否可以接单，-0：关闭接单，1：开启接单',
    create_time     datetime    default CURRENT_TIMESTAMP null,
    update_time     datetime    default CURRENT_TIMESTAMP null on update CURRENT_TIMESTAMP,
    is_deleted      int         default 0                 null
)
    comment '服务人员/机构附属信息' charset = utf8mb4;
```

服务人员与机构表：存储服务人员与机构的注册信息。

当服务技能、服务范围、接单设置全部设置完成会将settings_status 字段更新为1 

```JSON
create table `jzo2o-customer`.serve_provider
(
    id                  bigint                             not null comment '主键'
        constraint `PRIMARY`
        primary key,
    code                varchar(255)                       null comment '编号',
    type                int                                not null comment '类型，2：服务人员，3：服务机构',
    name                varchar(255)                       null comment '姓名',
    phone               varchar(255)                       not null comment '电话',
    avatar              varchar(255)                       null comment '头像',
    status              int                                not null comment '状态，0：正常，1：冻结',
    settings_status     int      default 0                 null comment '首次设置状态，0：未完成设置，1：已完成设置',
    password            varchar(255)                       null comment '机构登录密码',
    account_lock_reason varchar(255)                       null comment '账号冻结原因',
    score               double                             null comment '综合评分',
    good_level_rate     varchar(50)                        null comment '好评率',
    create_time         datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    update_time         datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    is_deleted          int      default 0                 not null comment '是否已删除，0：未删除，1：已删除',
    constraint serve_provider_phone_type_uindex
        unique (phone, type)
)
    comment '服务人员/机构表' charset = utf8mb4;
```

### 4）代码阅读

这些操作的接口在客户中心工程（jzo2o-customer）定义。

服务技能设置接口类：

```JSON
package com.jzo2o.customer.controller.worker;


import com.jzo2o.api.foundations.dto.response.ServeItemSimpleResDTO;
import com.jzo2o.api.foundations.dto.response.ServeTypeSimpleResDTO;
import com.jzo2o.customer.model.dto.request.ServeSkillAddReqDTO;
import com.jzo2o.customer.model.dto.response.ServeSkillCategoryResDTO;
import com.jzo2o.customer.service.IServeSkillService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;
import java.util.List;

/**
 * <p>
 * 服务技能表 前端控制器
 * </p>
 *
 * @author itcast
 * @since 2023-07-18
 */
@RestController("workerServeSkillController")
@RequestMapping("/worker/serve-skill")
@Api(tags = "服务端 - 服务技能相关接口")
public class ServeSkillController {
    @Resource
    private IServeSkillService serveSkillService;

    @PostMapping("/batchUpsert")
    @ApiOperation("批量新增或修改服务技能")
    public void listServeType(@RequestBody List<ServeSkillAddReqDTO> serveSkillAddReqDTOList) {
        serveSkillService.batchUpsert(serveSkillAddReqDTOList);
    }

    @GetMapping("/category")
    @ApiOperation("查询服务技能目录")
    public List<ServeSkillCategoryResDTO> category() {
        return serveSkillService.category();
    }

    @GetMapping("/queryCurrentUserServeSkillTypeList")
    @ApiOperation("查询当前用户的服务技能类型")
    public List<ServeTypeSimpleResDTO> queryCurrentUserServeSkillTypeList() {
        return serveSkillService.queryCurrentUserServeSkillTypeList();
    }

    @GetMapping("/queryCurrentUserServeSkillItemList")
    @ApiOperation("查询当前用户的服务技能")
    public List<ServeItemSimpleResDTO> queryCurrentUserServeSkillItemList() {
        return serveSkillService.queryCurrentUserServeSkillItemList();
    }
}
```

服务范围及接单设置接口类：

```JSON
package com.jzo2o.customer.controller.worker;

import com.jzo2o.customer.model.dto.request.ServePickUpReqDTO;
import com.jzo2o.customer.model.dto.request.ServeScopeSetReqDTO;
import com.jzo2o.customer.model.dto.response.ServeProviderSettingsGetResDTO;
import com.jzo2o.customer.model.dto.response.ServeSettingsStatusResDTO;
import com.jzo2o.customer.service.IServeProviderSettingsService;
import com.jzo2o.mvc.utils.UserContext;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;

/**
 * @author 86188
 */

@RestController("workerServeProviderSettingsController")
@RequestMapping("/worker/serve-settings")
@Api(tags = "服务端 - 服务设置相关接口")
public class ServeProviderSettingsController {

    @Resource
    private IServeProviderSettingsService serveProviderSettingsService;

    @PutMapping("/serve-scope")
    @ApiOperation("服务范围设置")
    public void setServeScope(@RequestBody @Validated ServeScopeSetReqDTO serveScopeSetReqDTO) {
        serveProviderSettingsService.setServeScope(serveScopeSetReqDTO);
    }

    @GetMapping
    @ApiOperation("获取设置")
    public ServeProviderSettingsGetResDTO getServeScope() {
        return serveProviderSettingsService.getServeScope();
    }

    @PutMapping("/pick-up")
    @ApiOperation("接单设置")
    public void setPickUp(@RequestBody ServePickUpReqDTO servePickUpReqDTO) {
//        serveProviderSettingsService.setPickUp(servePickUpReqDTO);
        serveProviderSettingsService.setPickUp(UserContext.currentUserId(), servePickUpReqDTO.getCanPickUp());
    }

    @GetMapping("/status")
    @ApiOperation("获取所有设置状态")
    public ServeSettingsStatusResDTO getStatus() {
        return serveProviderSettingsService.getSettingStatus();
    }
}
```



### 5）测试

请大家自行从接口入口开始阅读代码，找到服务技能设置、服务范围设置、接单设置存储的表及存储方式。

存储方式：是添加还是更新、是批量还是单条记录操作等详细的操作方式。

在阅读代码时可以通过断点测试的方式进行。

启动jzo2o-customer、jzo2o-publics、jzo2o-fundations、jzo2o-gateway。

启动服务端前端工程。

（服务端前端工程运行方式参考第二章关于project-xzb-app-uniapp-java工程的部署说明）

在测试时可以使用一个新的手机号注册。



关于服务范围设置：

设置服务范围时需要通过手机进行定位，我们使用浏览器测试服务端前端程序无法通过手机定位，这里我们在配置文件中配置坐标模拟手机定位的效果。

在下边的配置文件上配置坐标及相关信息实现定位：

![这是一段代码配置的图片，对应抢单设置中配置坐标模拟手机定位的步骤。左侧是项目文件夹结构，选中的是address文件夹下的h5Data.js文件；右侧是该文件的代码内容，文件注释说明是h5情况下的运行环境配置，代码中export的data对象包含了经度116.4343、纬度40.008，以及对应北京市的城市信息。这些配置内容为测试使用浏览器模拟手机定位提供了参数支撑，与文档中通过配置文件坐标模拟手机定位的内容完全对应。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Mrz8bX69MoXX31xy4irc8TgCnpd.png)

坐标及信息使用高德地图的坐标拾取工具(https://lbs.amap.com/tools/picker)获取。

![这张图片展示的是高德地图坐标拾取工具的页面，该工具可用于获取坐标信息，对应文档中提及的“坐标及信息使用高德地图的坐标拾取工具”的内容。页面上有“按关键字搜索”“按坐标搜索”两个选项，下方的搜索框中填写了“北京黑马程序员”，点击“搜索”按钮后，在“坐标获取结果”区域显示出对应的坐标数值，该坐标被红框标注，且右侧设有“复制”按钮，页面还呈现了包含相关地点的电子地图，用于直观展示对应位置。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/LDV2bLDCdo2gJ8xtyg8cKm0OnLd.png)

citycode从数据库的city_directory获取。

![图片展示了数据库查询结果，表名为jzo2o-foundations的city_directory。其中，parent_code为11，type为2，city_name为北京市，city_code为010。图片与上文提到的抢单设置中代码阅读内容相关，用于说明在jzo2o-customer.serve_provider_settings表中找到服务人员对应接单范围信息时，可从jzo2o-customer-all.serve_provider表根据手机号找到主键，再根据主键从jzo2o-customer-all.serve_provider_settings表找到具体记录，此图展示了其中的字段信息。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/MElwb262xo9Y3gxTJ5AcpE6HnOh.png)

进入服务范围设置界面：

![这是一个用于设置工作地点的界面，属于抢单设置流程中的服务范围设置环节。界面顶部显示标题“请选择工作地点”，下方有两个可配置项，分别为服务城市（当前显示为北京市）和意向接单范围（当前显示为北京黑马程序员），底部设有一个醒目的红色“保存”按钮。该界面的作用是供相关人员调整接单对应的工作地点信息，若需修改原有接单范围的设置，需先清理对应数据库中的相关字段值后，再进入此界面完成修改操作。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EShWbtYUHoty55x7v0pcAEpcnoh.png)

如果要修改原来设置的接单范围需要进入数据库清理接单范围设置后再进入此界面。

进入jzo2o-customer.serve_provider_settings表，找到服务人员对应的接单范围信息，清理下边红框内的字段值

![这张图片展示了一段JSON格式的数据，其中红框标注了多个字段内容。红框内包含的字段及对应值分别为：“city_code”字段值为“010”，“city_name”字段值为“北京市”，“lon”字段值为116.4343，“lat”字段值为40.008，“intention_scope”字段值为“北京黑马程序员”。该图片对应文档中提到的，需要清理的jzo2o-customer.serve_provider_settings表中的红框内字段，结合上下文，这些是修改服务人员接单范围时需清理的相关数据信息。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HZHdbwg8poWHP2xdks8cVzilnwd.png)

如何找到服务人员对应的接单范围信息**？**

先从jzo2o-customer-all.serve_provider表中根据手机号找到主键

再根据主键从jzo2o-customer-all.serve_provider_settings表找到具体的记录。



### 6）小结

抢单设置包括哪些表？

服务技能表：存储服务提供者的技能信息。

服务提供者设置表：存储服务提供者的服务范围等信息。

服务人员与机构表：存储服务人员与机构的注册信息。



## 3.4 订单分流

### 1）需求分析

订单分流的流程如下：

<whiteboard token="AD9Dw7NTMhWLssbbIlhc8JPonzd"></whiteboard>

### 2）表设计

根据数据流分析需要设计抢单池表。

```Java
-- 抢单池(资源池) 
-- auto-generated definition
create table orders_seize
(
    id               bigint                                not null comment '订单id'
        primary key,
    orders_code      varchar(50)                           null comment '订单编号',
    city_code        varchar(50) default ''                not null comment '城市编码',
    serve_type_id    bigint                                null comment '服务分类id',
    serve_item_name  varchar(50)                           not null comment '服务名称',
    serve_type_name  varchar(50)                           not null comment '服务分类名称',
    serve_item_id    bigint                                null comment '服务项id',
    serve_address    varchar(50)                           not null comment '服务地址',
    serve_item_img   varchar(255)                          not null comment '服务项目图片',
    orders_amount    decimal(10, 2)                        null comment '订单总金额',
    serve_start_time datetime                              not null comment '服务开始时间',
    pay_success_time datetime                              null comment '订单支付成功时间，用于计算是否进入派单',
    lon              double(10, 5)                         null comment '经度',
    lat              double(10, 5)                         null comment '纬度',
    pur_num          int                                   not null comment '服务数量',
    is_time_out      int         default 0                 null comment '抢单是否超时',
    sort_by          bigint                                null comment '抢单列表排序字段',
    create_time      datetime    default CURRENT_TIMESTAMP null comment '创建时间',
    update_time      datetime    default CURRENT_TIMESTAMP null on update CURRENT_TIMESTAMP comment '更新时间'
)
    comment '抢单池(资源池)' charset = utf8mb4;

create index sort_by_index
    on orders_seize (sort_by);
```

### 3）阅读代码

下边阅读订单分流相关的代码。

```JSON
package com.jzo2o.orders.base.service.impl;

import com.jzo2o.api.foundations.RegionApi;
import com.jzo2o.api.foundations.ServeApi;
import com.jzo2o.api.foundations.dto.response.ConfigRegionInnerResDTO;
import com.jzo2o.api.foundations.dto.response.ServeAggregationResDTO;
import com.jzo2o.common.utils.BooleanUtils;
import com.jzo2o.common.utils.DateUtils;
import com.jzo2o.common.utils.ObjectUtils;
import com.jzo2o.orders.base.mapper.OrdersDispatchMapper;
import com.jzo2o.orders.base.mapper.OrdersSeizeMapper;
import com.jzo2o.orders.base.model.domain.Orders;
import com.jzo2o.orders.base.model.domain.OrdersDispatch;
import com.jzo2o.orders.base.model.domain.OrdersSeize;
import com.jzo2o.orders.base.service.IOrdersDiversionCommonService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;
import java.time.Duration;

@Service
@Slf4j
public class OrdersDiversionCommonServiceImpl implements IOrdersDiversionCommonService {


    @Resource
    private RedisTemplate<String, Long> redisTemplate;

    @Resource
    private RegionApi regionApi;

    @Resource
    private ServeApi serveApi;

    @Resource
    private OrdersDiversionCommonServiceImpl owner;

    @Resource
    private OrdersSeizeMapper ordersSeizeMapper;
    @Resource
    private OrdersDispatchMapper ordersDispatchMapper;

    @Override
    public void diversion(Orders orders) {
        log.debug("订单分流，id:{}",orders.getId());
        // 1.当前时间已超过服务预约时间则不再分流
        if (orders.getServeStartTime().compareTo(DateUtils.now()) < 0) {
            log.debug("订单{}当前时间已超过服务预约时间则不再分流",orders.getId());
            return;
        }
        ConfigRegionInnerResDTO configRegion = regionApi.findConfigRegionByCityCode(orders.getCityCode());
        ServeAggregationResDTO serveAggregationResDTO = serveApi.findById(orders.getServeId());
        //订单分流数据存储
        owner.diversionCommit(orders,configRegion,serveAggregationResDTO);
    }

    @Transactional(rollbackFor = Exception.class)
    public void diversionCommit(Orders orders, ConfigRegionInnerResDTO configRegion, ServeAggregationResDTO serveAggregationResDTO) {
        //流间隔（单位分钟），即当前时间与服务预计开始时间的间隔
        Integer diversionInterval = configRegion.getDiversionInterval();

        //当前时间与服务预约时间的间隔
        Duration between = DateUtils.between(DateUtils.now(), orders.getServeStartTime());
        //服务类型名称
        String serveTypeName = ObjectUtils.get(serveAggregationResDTO, ServeAggregationResDTO::getServeTypeName);
        //服务类型id
        Long serveTypeId = ObjectUtils.get(serveAggregationResDTO, ServeAggregationResDTO::getServeTypeId);
        //服务项名称
        String serveItemName = ObjectUtils.get(serveAggregationResDTO, ServeAggregationResDTO::getServeItemName);
        //服务项图片
        String serveItemImg = ObjectUtils.get(serveAggregationResDTO, ServeAggregationResDTO::getServeItemImg);
        //用于排序,服务预约时间戳加订单号后5位
        long sortBy = DateUtils.toEpochMilli(orders.getServeStartTime()) + orders.getId() % 100000;
        OrdersSeize ordersSeize = OrdersSeize.builder()
                .id(orders.getId())
                .ordersAmount(orders.getRealPayAmount())
                .cityCode(orders.getCityCode())
                .serveTypeId(serveTypeId)
                .serveTypeName(serveTypeName)
                .serveItemId(orders.getServeItemId())
                .serveItemName(serveItemName)
                .serveItemImg(serveItemImg)
                .ordersAmount(orders.getRealPayAmount())
                .serveStartTime(orders.getServeStartTime())
                .serveAddress(orders.getServeAddress())
                .lon(orders.getLon())
                .lat(orders.getLat())
                .paySuccessTime(DateUtils.now())
                .paySuccessTime(orders.getPayTime())
                .sortBy(sortBy)
                .isTimeOut(BooleanUtils.toInt(between.toMinutes() < diversionInterval))
                .purNum(orders.getPurNum()).build();
        ordersSeizeMapper.insert(ordersSeize);
        //当前时间与服务预约时间的间隔 小于指定间隔则插入派单表
        if (between.toMinutes() < diversionInterval) {
            OrdersDispatch ordersDispatch = OrdersDispatch.builder()
                    .id(orders.getId())
                    .ordersAmount(orders.getRealPayAmount())
                    .cityCode(orders.getCityCode())
                    .serveTypeId(serveTypeId)
                    .serveTypeName(serveTypeName)
                    .serveItemId(orders.getServeItemId())
                    .serveItemName(serveItemName)
                    .serveItemImg(serveItemImg)
                    .ordersAmount(orders.getRealPayAmount())
                    .serveStartTime(orders.getServeStartTime())
                    .serveAddress(orders.getServeAddress())
                    .lon(orders.getLon())
                    .lat(orders.getLat())
                    .purNum(orders.getPurNum()).build();
            ordersDispatchMapper.insert(ordersDispatch);
        }
    }


}
```

### 4）实现订单分流

根据需求，订单支付成功进行订单分流，在支付成功的方法添加订单分流的代码：

```JSON
@Transactional(rollbackFor = Exception.class)
public void paySuccess(TradeStatusMsg tradeStatusMsg) {
    //查询订单
    Orders orders = baseMapper.selectById(tradeStatusMsg.getProductOrderNo());

    //2：待支付，4：支付成功
    if (ObjectUtil.notEqual(OrderPayStatusEnum.NO_PAY.getStatus(), orders.getPayStatus())) {
        log.info("当前订单：{}，不是待支付状态", orders.getId());
        return;
    }

    //第三方支付单号校验
    if (ObjectUtil.isEmpty(tradeStatusMsg.getTransactionId())) {
        throw new CommonException("支付成功通知缺少第三方支付单号");
    }
    // 修改订单状态和支付状态
    OrderSnapshotDTO orderSnapshotDTO = OrderSnapshotDTO.builder()
            .payTime(LocalDateTime.now())
            .tradingOrderNo(tradeStatusMsg.getTradingOrderNo())
            .tradingChannel(tradeStatusMsg.getTradingChannel())
            .thirdOrderId(tradeStatusMsg.getTransactionId())
            .payStatus(OrderPayStatusEnum.PAY_SUCCESS.getStatus())
            .build();
    orderStateMachine.changeStatus(orders.getUserId(), String.valueOf(orders.getId()), OrderStatusChangeEventEnum.PAYED, orderSnapshotDTO);

    // 订单分流
    ordersDiversionService.diversion(orders);
  }
```



### 5）调度配置

登录运营端，进入区域配置，针对运营区域进行调度配置

![图片展示的是运营端区域管理界面。界面中列出了区域编号、区域名称、区域负责人、手机号码、状态、更新时间及操作等信息。其中，区域编号023对应重庆市，负责人猪八戒，手机号码13231221212，状态为禁用；区域编号0594对应莆田市，负责人李四，手机号码17812345678，状态为禁用；区域编号0371对应郑州市，负责人小红，手机号码13333333333，状态为启用。操作栏有编辑、调度配置、设置服务、启用、缓存刷新等选项，其中郑州区域的“调度配置”操作被红色箭头突出显示。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Bzh9bxFjco4euBxLREkcHlMqnOd.png)

![这张图片是运营端的调度配置界面，对应订单分流的调度配置环节，用于完成运营区域的调度设置。界面包含区域名称栏，以及全局策略、抢单策略、派单策略三类配置项，每类配置项均设有“个体”“公众”相关的服务范围半径输入框，其中部分输入框内显示数值；抢单策略还包含“抢单超时时间”的分钟输入项，派单策略则有“派单规则”“抢单等待时间”的相关配置，界面底部设置“取消”与“确定”操作按钮，完成配置后点击确定即可保存至config_region表。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XO7Hb0pn2ogUYdxwcAscEprcnCe.png)

此操作向config_region表保存配置，config_region表的数据是在添加区域时同时写入的。

如果在区域管理界面可以查询到区域在此表不存在数据是因为脏数据导致，需要手动向config_region表添加区域，config_region表的city_code和id字段与region表保持 一致。



### 5）测试

测试流程：

启动网关

启动jzo2o-orders-manager

启动jzo2o-foundations

启动小程序

用户下单并支付成功。

观察订单是否写入抢单池表。

派单池暂不测试。

预期：

成功写入抢单池表



为了测试方便我们修改下单的代码，下单成功默认支付成功。

在nacos的jzo2o-orders-manager.yaml配置jzo2o.openPay参数：

```JSON
jzo2o:
  openPay: false
```

修改下单service方法如下：

```JSON
@Value("${jzo2o.openPay}")
private Boolean openPay;

public PlaceOrderResDTO placeOrder(Long userId, PlaceOrderReqDTO placeOrderReqDTO) {
        //....
        //todo 测试需要默认支付成功
        if (Boolean.FALSE.equals(openPay)) {
            TradeStatusMsg msg = TradeStatusMsg.builder()
                    .productOrderNo(String.valueOf(orders.getId()))
                    .tradingChannel("WECHAT_PAY")
                    .statusCode(TradingStateEnum.YJS.getCode())
                    .tradingOrderNo(IdUtil.getSnowflakeNextId())
                    .transactionId(IdUtils.getSnowflakeNextIdStr())
                    .build();
            owner.paySuccess(msg);
        }
        return new PlaceOrderResDTO(orders.getId());
    }
```



测试示例：

新下单：

![这是一张日常保洁服务的预约界面截图，界面顶部标注服务类型为“日常保洁”且默认勾选“日常上门保洁”选项，下方设有填写预约信息的区域，已自动填充服务地址为北京市昌平区回龙观街道的某中心，联系人与联系电话也已预设，同时显示需选择上门时间，当前预选时段为2023年11月15日20:30，优惠券区域标注暂无可用优惠券，界面底部显示该服务售价为1元，右下角设有红色的“立即预约”按钮，用于提交预约申请，该界面对应文档中测试下单功能时的支付及预约环节。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/SCl5bBxVRoIChWxhvwPcmYumnce.png)

下单成功进入支付页面，

![图片展示的是“支付订单”页面。页面显示订单总价1元，待付金额1.00元，支付剩余时间为14:55。页面中有微信支付和支付宝支付两种支付方式，微信支付右侧有红色勾选标志。页面底部有“还需支付：1.00元”的提示，以及一个红色的“确认支付”按钮。该图片与文档中“下单成功进入支付页面”的内容对应，直观呈现了支付页面的样式及关键信息。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NbCRb0CAmoG5GZxPrqYcB3K3nSf.png)

因为我们为了测试方便设置了自动支付成功，上边的支付界面 不用管，直接进入我的订单列表查看此订单状态为派单中，这说明订单支付成功。

![该图片展示的是手机端“我的订单”页面，顶部设有“待支付”“待服务”“服务中”“待评价”“全部”的订单分类标签，当前选中“全部”标签。页面中列有一条订单，对应的服务内容为“日常保洁”，下单时间是2023年11月15日20:30，订单状态标注为“派单中”，还显示了该订单的服务地址为北京市昌平区回龙观街道的相关地址。该图片对应文档中订单测试成功的环节，直观呈现了测试流程完成后订单的状态，符合测试说明中支付成功后订单进入派单中的结论。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/CwUGbIpyzouktpxghTPcHWFKnnH.png)

检查抢单池表，成功向抢单池添加成功。

```JSON
{
  "id": 2311150000000000038,
  "city_code": "010",
  "serve_type_id": 1678649931106705409,
  "serve_item_name": "日常保洁",
  "serve_type_name": "保洁清",
  "serve_item_id": 1685894105234755585,
  "serve_address": "北京市北京市昌平区北京市昌平区回龙观街道弘文恒瑞文化传播公司正泽商务中心",
  "serve_item_img": "https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/aa6489e5-cd92-42f0-837a-952c99653b8b.png",
  "orders_amount": 1.00,
  "serve_start_time": "2023-11-15 20:30:00",
  "pay_success_time": null,
  "lon": 116.34351,
  "lat": 40.06024,
  "pur_num": 1,
  "is_time_out": 0,
  "sort_by": 1700051400038,
  "create_time": "2023-11-15 15:06:11",
  "update_time": "2023-11-15 15:06:11"
}
```



### 6）小结

订单分流的业务流程是什么？

1. 用户下单且支付成功
2. 根据服务预约时间与当前时间的间隔判断，如果间隔小于2小时（默认）将订单信息写入派单池表和抢单池表。
3. 如果服务预约时间与当前时间的间隔大于2小时将订单信息只写入抢单池表。



## 3.5 抢单查询

### 3.5.1 抢单池同步

#### 1）同步方案

根据系统设计方案，抢单池的信息需要同步到Elasticsearch，使用Canal加RabbitMQ完成同步，如下：

<whiteboard token="L70xw6uRRhtIdtbpw6xcJCSXnDe"></whiteboard>



#### 2) 索引结构设计

在ES中创建索引结构orders_seize，抢单池的信息将同步到orders_seize中。

启动ES和kibana

```JSON
docker start es 
docker start kibana
```

通过下边的命令创建orders_seize索引结构：

```Java
PUT /orders_seize
{
  "mappings" : {
      "properties" : {
        "city_code" : {
          "type" : "keyword"
        },
        "id" : {
          "type" : "long"
        },
        "key_words" : {
          "type" : "text",
          "analyzer" : "ik_max_word",
          "search_analyzer" : "ik_smart"
        },
        "location" : {
          "type" : "geo_point"
        },
        "orders_amount" : {
          "type" : "float"
        },
        "pur_num" : {
          "type" : "integer"
        },
        "serve_address" : {
          "type" : "text",
          "index" : false
        },
        "serve_item_id" : {
          "type" : "long"
        },
        "serve_item_img" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "serve_item_name" : {
          "type" : "text",
          "index" : false
        },
        "serve_start_time" : {
          "type" : "text",
          "index" : false
        },
        "serve_time" : {
          "type" : "integer"
        },
        "serve_type_id" : {
          "type" : "long"
        },
        "serve_type_name" : {
          "type" : "text",
          "index" : false
        },
        "total_amount" : {
          "type" : "double"
        }
      }
    }
}
```

创建成功进行查询 索引列表，命令如下，找到orders_seize。

```Java
GET /_cat/indices?v
```

查询orders_seize的索引结构，命令如下，对照是否和上边创建的orders_seize的索引一致。

```Java
GET orders_seize
或
GET /orders_seize/_mapping
```

#### 3) **抢单池库存结构**

抢单池库存信息存储在Redis，在抢单时通过Redis扣减库存。

设计如下：

缓存结构：Hash

RedisKey：ORDERS:RESOURCE:STOCK:{citycode%10}

HashKey：订单id

HashValue: 库存，为1

过期时间：永不过期

缓存一致性方案：通过Canal进行同步

<whiteboard token="FIqQwZ2fFhSPO0b1Jcbcmd8LnQd"></whiteboard>

#### 4）阅读代码



阅读下边的代码：

```JSON
package com.jzo2o.orders.seize.handler;

import com.jzo2o.canal.listeners.AbstractCanalRabbitMqMsgListener;
import com.jzo2o.common.model.Location;
import com.jzo2o.common.utils.BeanUtils;
import com.jzo2o.common.utils.CollUtils;
import com.jzo2o.es.core.ElasticSearchTemplate;
import com.jzo2o.orders.base.constants.RedisConstants;
import com.jzo2o.orders.base.model.domain.OrdersSeize;
import com.jzo2o.orders.base.utils.RedisUtils;
import com.jzo2o.orders.seize.model.domain.OrdersSeizeInfo;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.time.format.DateTimeFormatter;
import java.util.List;
import java.util.stream.Collectors;

import static com.jzo2o.orders.base.constants.EsIndexConstants.ORDERS_SEIZE;

/**
 * 抢单池同步类
 */
@Component
@Slf4j
public class OrdersSeizeSyncHandler extends AbstractCanalRabbitMqMsgListener<OrdersSeize> {

    @Resource
    private ElasticSearchTemplate elasticSearchTemplate;

    @Resource
    private RedisTemplate redisTemplate;


    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "canal-mq-jzo2o-orders-seize"),
            exchange = @Exchange(name = "exchange.canal-jzo2o", type = ExchangeTypes.TOPIC),
            key = "canal-mq-jzo2o-orders-seize"),
            concurrency = "1"
    )
    public void onMessage(Message message) throws Exception {
        parseMsg(message);
    }

    @Override
    public void batchSave(List<OrdersSeize> ordersSeizes) {
        // 1.es中添加抢单信息
        List<OrdersSeizeInfo> ordersSeizeInfos = ordersSeizes.stream().map(ordersSeize -> {
            OrdersSeizeInfo ordersSeizeInfo = BeanUtils.toBean(ordersSeize, OrdersSeizeInfo.class);
            //得到服务开始时间(yyMMddHH)
            String serveTimeString = DateTimeFormatter.ofPattern("yyMMddHH").format(ordersSeize.getServeStartTime());
            ordersSeizeInfo.setServeTime(Integer.parseInt(serveTimeString));
            ordersSeizeInfo.setLocation(new Location(ordersSeize.getLon(), ordersSeize.getLat()));
            ordersSeizeInfo.setKeyWords(ordersSeize.getServeTypeName() + ordersSeize.getServeItemName() + ordersSeize.getServeAddress());
            return ordersSeizeInfo;
        }).collect(Collectors.toList());

        Boolean result = elasticSearchTemplate.opsForDoc().batchInsert(ORDERS_SEIZE, ordersSeizeInfos);
        if (!result){
            throw new RuntimeException("同步抢单池加入es失败");
        }
        // 2.写入库存
        ordersSeizeInfos.stream().forEach(ordersSeizeInfo -> {
            String redisKey = String.format(RedisConstants.RedisKey.ORDERS_RESOURCE_STOCK, RedisUtils.getCityIndex(ordersSeizeInfo.getCityCode()));
            // 库存默认1
            redisTemplate.opsForHash().putIfAbsent(redisKey, ordersSeizeInfo.getId(), 1);
        });
    }

    @Override
    public void batchDelete(List<Long> ids) {
        log.info("抢单删除开始，删除数量:{},开始id：{}，结束id:{}", CollUtils.size(ids), CollUtils.getFirst(ids), CollUtils.getLast(ids));
        Boolean result = elasticSearchTemplate.opsForDoc().batchDelete(ORDERS_SEIZE, ids);
        if (!result){
            throw new RuntimeException("同步抢单池加入es失败");
        }
        log.info("抢单删除结束，删除数量:{},开始id：{}，结束id:{}", CollUtils.size(ids), CollUtils.getFirst(ids), CollUtils.getLast(ids));

    }
}
```

5）测试

1. 首先在rabbitmq中创建canal-mq-jzo2o-orders-seize队列并绑定到exchange.canal-jzo2o交换机
2. 然后配置Canal

在Canal中配置order_seize为同步表。

![图片展示了Canal配置代码，重点突出“canal-mq-jzo2o-orders-seize:jzo2o-orders-0\\\\\\.orders_seize”这一内容。上下文提到在Canal中配置order_seize为同步表，图片中的代码配置了多个同步表，其中“canal-mq-jzo2o-orders-seize:jzo2o-orders-0\\\\\\.orders_seize”即为配置的同步表之一，表明在Canal配置中，仅配置了jzo2o-orders-0下的order_seize为同步表，这与上下文提到的配置原因相呼应。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/QWvXbuIs3o8fcVxDROAcWZY0nAu.png)

上边的配置中为什么只配置了jzo2o-orders-0下的order_seize为同步表？jzo2o-orders-1和jzo2o-orders-2需要配置吗？

order_seize为广播表，jzo2o-orders-0、jzo2o-orders-1、jzo2o-orders-2三个数据库中的order_seize表数据是一致的，所以只配置其中一个数据库的order_seize为同步表即可。



1. 下边进行测试

测试流程：

启动Canal

启动RabbitMQ

启动Elasticsearch

启动Kibana

启动Redis

启动jzo2o-orders-seize

启动jzo2o-orders-manager

启动网关

启动小程序

用户下单并支付成功。

观察订单是否写入抢单池表。

在OrdersSeizeSyncHandler 中的batchSave(List<OrdersSeize> ordersSeizes)方法中打断点，跟踪抢单池数据是否正常同步到Elasticsearch。



注意：关于canal数据不同步问题参考“配置搜索及数据同步环境v1.0” 中常见问题进行解决。



示例：

在同步类中打断点

下单成功，执行到断点位置

![这张图片是开发调试时的代码界面，展示了OrdersSeizeSyncHandler.java文件中的业务代码，对应上下文里“同步类中打断点”的调试场景。界面中，红色箭头指向代码行`List<OrdersSeizeInfo> ordersSeizeInfos = order...`，该行代码被标记为断点；界面右侧显示调试信息，标注变量`ordersSeizes`为大小等于1的集合，其第一个元素是订单信息，包含订单号、城市编码等具体内容，还提供了“Set value”“Create renderer”的操作选项，该调试状态表明程序已进入抢单池同步流程的断点，符合上下文里同步数据接收后的调试验证操作。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/TTEnbXwg8oyoOXxxsrgcVWPqnlc.png)

程序进入断点说明收到抢单池同步数据

继续运行程序，数据将写入ES

登录kibana，执行：GET /orders_seize/\_search，查询抢单数据是否同步到ES。

```JSON
#! Elasticsearch built-in security features are not enabled. Without authentication, your cluster could be accessible to anyone. See https://www.elastic.co/guide/en/elasticsearch/reference/7.17/security-minimal-setup.html to enable security.
{
  "took" : 261,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "orders_seize",
        "_type" : "_doc",
        "_id" : "2311150000000000038",
        "_score" : 1.0,
        "_source" : {
          "id" : 2311150000000000038,
          "city_code" : "010",
          "serve_type_id" : 1678649931106705409,
          "serve_item_id" : 1685894105234755585,
          "serve_type_name" : "保洁清",
          "serve_item_name" : "日常保洁",
          "serve_address" : "北京市北京市昌平区北京市昌平区回龙观街道弘文恒瑞文化传播公司正泽商务中心",
          "serve_time" : 23111520,
          "location" : {
            "lon" : 116.34351,
            "lat" : 40.06024
          },
          "serve_start_time" : "2023-11-15 20:30:00",
          "pur_num" : 1,
          "key_words" : "保洁清日常保洁北京市北京市昌平区北京市昌平区回龙观街道弘文恒瑞文化传播公司正泽商务中心",
          "orders_amount" : 1.0,
          "serve_item_img" : "https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/aa6489e5-cd92-42f0-837a-952c99653b8b.png"
        }
      }
    ]
  }
}

```

下边进入redis查看抢单库存是否同步成功：

hashkey: 订单id

hashvalue：库存数量

![图片展示的是抢单库存同步到Redis的情况。在Redis界面中，哈希键为“ORDERS:RESOURCE:STOCK:{0}”，哈希值为“2311150000000000038”，其对应的value值为1。该图片与上文抢单池同步内容相关，用于说明在抢单池同步后，抢单库存是否同步成功，通过查看Redis中订单id对应的库存数量来验证。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/AWL8bbyzUodB4ixVvg9cpEStnxh.png)



#### 5）小结

抢单池是怎么设计的？

1. 在数据库中创建抢单池表，存储待抢单的订单信息。
2. 用户下单并支付成功将订单信息写入订单表和抢单池表。
3. 通过Canal+MQ将抢单池的信息同步到Elasticsearch和Redis中。

同步到Elasticsearch是为了通过ES的地理坐标搜索功能查询订单信息。

同步到Redis是为了将抢单池库存信息同步到Redis，服务人员抢单时请求Redis完成。

1. 服务人员抢单完成扣减Redis中抢单池的库存，并记录抢单结果。
2. 最后通过异步任务将抢单结果同步到数据库。



### 3.5.2 抢单查询

#### 1）接口定义

接口名称：抢单查询列表

接口功能：查询抢单池的订单信息

接口路径：**GET**/orders-seize/worker

请求参数：

![图片展示了抢单查询接口的请求参数信息。表格包含参数名称、参数说明、请求类型、是否必须、数据类型、schema六列。关键参数有keyWord（关键字）、lastRealDistance（上一页最后一条数据的距离字段）、maxServeStartTime（预约时间查询上限）、minServeStartTime（预约时间查询下限）、serveDistance（服务距离）、serveTypeId（服务类型id）。其中，serveTypeId的数据类型为integer(int64)，且为必填项。该图片与上文接口开发中根据服务人员服务范围、技能筛选抢单池，通过Java代码查询ES的内容相关，用于说明接口请求参数。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/T2xabg9S1ofayVx6s9acyZwcnIc.png)

响应参数：

![图片展示了抢单查询接口的响应参数。包含msg、code、data等字段，其中data下的ordersSeizes为抢单列表，包含serveTypeId、realDistance、serveItemName等信息。如serveTypeId为服务分类id，realDistance为实际距离，serveItemName为服务项名称等。此图与上文接口功能介绍相呼应，详细说明了接口响应参数的具体内容，为接口开发和测试提供参数依据。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/YzAIbwPOPo0RAOxaxV7cCt1Qnre.png)

定义controller

```Java
@RestController
@Api(tags = "服务端 - 抢单相关接口")
@RequestMapping("/worker")
@Slf4j
public class WorkerOrdersSeizeController {

@GetMapping("")
@ApiOperation("服务端抢单列表")
public OrdersSeizeListResDTO queryForList(OrdersSerizeListReqDTO ordersSerizeListReqDTO) {
    return null;
}
...
```



#### 2）接口开发

1. 首先在Kibana中测试查询抢单语句 

下边表示根据服务人员的服务范围、服务技能搜索抢单池：

```JSON
GET /orders_seize/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "city_code": {
              "value": "010"
            }
          }
        },
        {
          "terms": {
            "serve_item_id": [
              1685894105234755585,
              1683432288440897537,
              1678727478181957634,
              1692475107114487809
            ]
          }
        },
        {
          "geo_distance": {
            "location": {
              "lat": 40.008,
              "lon": 116.4343
            },
            "distance": "3.0km"
          }
        }
      ]
    }
  },
    "sort": [
    {
      "_geo_distance": {
        "location": [
          {
            "lat": 40.008,
            "lon": 116.4343
          }
        ],
        "distance_type": "arc",// 距离计算方式
        "order": "asc", // "asc" 表示升序，"desc" 表示降序
        "unit": "km"//// 单位，可以是 "km"、"mi"（英里）等
      }
    }
  ]
}
```

服务人员可以选择服务类型和距离进行筛选

```JSON
GET /orders_seize/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "city_code": {
              "value": "010"
            }
          }
        },
        {
          "term": {
            "serve_type_id": {
              "value": 1678649931106705409
            }
          }
        },
        {
          "terms": {
            "serve_item_id": [
              1685894105234755585,
              1683432288440897537,
              1678727478181957634,
              1692475107114487809
            ]
          }
        },
        {
          "geo_distance": {
            "location": {
              "lat": 40.008,
              "lon": 116.4343
            },
            "distance": "20.0km"
          }
        }
      ]
    }
  },
    "sort": [
    {
      "_geo_distance": {
        "location": [
          {
            "lat": 40.008,
            "lon": 116.4343
          }
        ],
        "distance_type": "arc",// 距离计算方式
        "order": "asc", // "asc" 表示升序，"desc" 表示降序
        "unit": "km"//// 单位，可以是 "km"、"mi"（英里）等
      }
    }
  ]
}
```

根据上边编写语句通过Java代码查询ES：

service方法如下：

```Java
@Service
@Slf4j
public class OrdersSeizeServiceImpl extends ServiceImpl<OrdersSeizeMapper, OrdersSeize> implements IOrdersSeizeService {

@Override
public OrdersSeizeListResDTO queryForList(OrdersSerizeListReqDTO ordersSerizeListReqDTO) {

    // 1.校验是否可以查询（认证通过，开启抢单）
    ServeProviderResDTO detail = serveProviderApi.getDetail(UserContext.currentUserId());
    // 验证设置状态
    if (detail.getSettingsStatus() != 1 || !detail.getCanPickUp()) {
        return OrdersSeizeListResDTO.empty();
    }
    // 2.查询准备 （距离、技能，时间冲突）
    // 距离
    Double serveDistance = ordersSerizeListReqDTO.getServeDistance();
    if(ObjectUtils.isNull(ordersSerizeListReqDTO.getServeDistance())) {
        // 区域默认配置配置
        ConfigRegionInnerResDTO configRegionInnerResDTO = regionApi.findConfigRegionByCityCode(detail.getCityCode());
        serveDistance = (detail.getType() == UserType.INSTITUTION)
                ? configRegionInnerResDTO.getInstitutionServeRadius().doubleValue() : configRegionInnerResDTO.getStaffServeRadius().doubleValue();
    }
    // 技能
    List<Long> serveItemIds = serveSkillApi.queryServeSkillListByServeProvider(UserContext.currentUserId(), UserContext.currentUser().getUserType(), detail.getCityCode());
    if(CollUtils.isEmpty(serveItemIds)) {
        log.info("当前机构或服务人员没有对应技能");
        return OrdersSeizeListResDTO.empty();
    }


    // 3.查询符合条件的抢单列表id
    List<OrdersSeizeListResDTO.OrdersSeize> ordersSeizes = getOrdersSeizeId(
            serveItemIds, detail.getLon(), detail.getLat(), serveDistance, detail.getCityCode(), ordersSerizeListReqDTO);

    return new OrdersSeizeListResDTO(CollUtils.defaultIfEmpty(ordersSeizes, new ArrayList<>()));
}

/**
     * 获取抢单id，抢单类型，抢单预约服务时间
     * @param serveItemIds 服务项id
     * @param lon 当前服务人员或机构所在位置经度
     * @param lat 当前服务人员或机构所在纬度
     * @param distanceLimit 抢单距离限制
     * @param cityCode 城市编码
     * @param ordersSerizeListReqDTO 抢单查询参数
     * @return
     */
    private List<OrdersSeizeListResDTO.OrdersSeize> getOrdersSeizeId(List<Long> serveItemIds, Double lon, Double lat, double distanceLimit, String cityCode, OrdersSerizeListReqDTO ordersSerizeListReqDTO) {

        // 服务项查询条件
        List<FieldValue> serveItemIdFieldValue = serveItemIds.stream().map(serveItemId -> FieldValue.of(serveItemId)).collect(Collectors.toList());

        SearchRequest.Builder builder = new SearchRequest.Builder();
        builder.query(query ->
                query.bool(bool -> {
                    // 所在城市
                    bool.must(must -> must.term(term -> term.field(CITY_CODE).value(cityCode)));
                    // 服务类型
                    if(ordersSerizeListReqDTO.getServeTypeId() != null) {
                        bool.must(must -> must.term(term -> term.field(SERVE_TYPE_ID).value(ordersSerizeListReqDTO.getServeTypeId())));
                    }
                    // 服务项
                    bool.must(must -> must.terms(terms -> terms.field(SERVE_ITEM_ID).terms(t -> t.value(serveItemIdFieldValue))));
                    // 距离条件
                    bool.must(must -> {
                        must.geoDistance(geoDistance -> {
                            geoDistance.field(LOCATION);
                            geoDistance.location(location -> location.latlon(latlon -> latlon.lon(lon).lat(lat)));
                            geoDistance.distance(distanceLimit + "km");
                            return geoDistance;});
                        return must;
                    });

                    // 关键字匹配 满足一个字段即可 服务项名称，服务类型名称，服务地址
                    if (StringUtils.isNotEmpty(ordersSerizeListReqDTO.getKeyWord())) {
                        bool.must(must -> must.match(match -> match.field(FieldConstants.KEY_WORDS).query(ordersSerizeListReqDTO.getKeyWord())));
                    }

                    return bool;
                }));
        // 排序 根据距离排序
        List<SortOptions> sortOptions = new ArrayList<>();
        sortOptions.add(SortOptions.of(sortOption -> sortOption.geoDistance(
                geoDistance -> {
                    geoDistance.field(LOCATION);
                    geoDistance.distanceType(GeoDistanceType.Arc);
                    geoDistance.order(SortOrder.Asc);
                    geoDistance.unit(DistanceUnit.Kilometers);
                    geoDistance.location(location -> location.latlon(latlon -> latlon.lat(lat).lon(lon)));
                    return geoDistance;
                }
        )));
        builder.sort(sortOptions);
        // 索引
        builder.index(EsIndexConstants.ORDERS_SEIZE);

        // 滚动分页,根据距离滚动分页
        if (ordersSerizeListReqDTO.getLastRealDistance() != null) {
            builder.searchAfter(ordersSerizeListReqDTO.getLastRealDistance().toString());
        }

        // 检索数据
        SearchResponse<OrdersSeizeInfo> searchResponse = elasticSearchTemplate.opsForDoc().search(builder.build(), OrdersSeizeInfo.class);
        if (SearchResponseUtils.isSuccess(searchResponse)) {
            return searchResponse.hits().hits()
                    .stream().map(hit -> {
                        // 从sort字段中获取实际距离
                        double realDistance = NumberUtils.parseDouble(CollUtils.getFirst(hit.sort()));
                        OrdersSeizeListResDTO.OrdersSeize ordersSeize = BeanUtils.toBean(hit.source(), OrdersSeizeListResDTO.OrdersSeize.class);
                        ordersSeize.setRealDistance(realDistance);
                        return ordersSeize;
                    })
                    .collect(Collectors.toList());
        }
        return null;
    }
```

2、controller

```Java
@GetMapping("")
@ApiOperation("服务端抢单列表")
public OrdersSeizeListResDTO queryForList(OrdersSerizeListReqDTO ordersSerizeListReqDTO) {
    return ordersSeizeService.queryForList(ordersSerizeListReqDTO);
}
```



#### 3) 测试

首先修改抢单服务的配置文件，使用shardingsphere的驱动

![该图片展示的是抢单服务配置文件的设置界面，配置格式选择为YAML，配置内容区域被红色框标注，框内显示spring配置项，其中datasource的驱动类设置为org.apache.shardingsphere.driver.ShardingSphereDriver，连接URL配置为jdbc:shardingsphere:classpath:shardingsphere-jdbc-${spring.profiles.active}.yml，同时还包含mybatis-plus相关配置内容，这对应文档中修改抢单服务配置文件、使用shardingsphere驱动的测试步骤。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WJhXbY8HSoyEvMx4PzTc9rjtnyg.png)

下边进行测试：

启动Canal

启动RabbitMQ

启动Elasticsearch

启动Kibana

启动Redis

启动jzo2o-orders-manager

启动jzo2o-orders-seize

启动jzo2o-customer

启动jzo2o-publics

启动jzo2o-foundations

启动网关

首先启动小程序下单

然后启动服务端前端程序（通过浏览器运行）

服务人员登录服务端程序，进入抢单界面，查询抢单信息。

预期结果：

北京的家政服务人员成功查询到符合条件抢单信息。

![图片展示的是北京家政服务抢单界面。上方有服务技能、接单设置、服务范围等选项。中间大图是一位正在清洁的女性，下方文字为“更好的家政上门服务体验”。下方有抢单大厅、距离筛选、服务类型等筛选项。显示一条抢单信息，服务类型为日常保洁，预约时间为2023年11月15日20:30，地点在北京市昌平区回龙观街道弘文恒瑞文化传播公司正泽商务中心，服务费用为0.30元，右侧有“立即抢单”按钮。该图与抢单查询相关，展示了抢单信息的展示界面。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/ZRf6bv5bQoY2QnxfIc9cUYZinjg.png)

具体查询语句在service方法中打断点跟踪

![图片展示了OrdersSeizeServiceImpl.java文件中searchResponse.isSuccess()为true时的代码执行情况。代码中使用了elasticsearchTemplate.opsForDoc().search()方法查询抢单池订单信息，若查询成功，通过searchResponse.hits().stream().map()获取抢单信息。图片右侧有代码断点标识，下方弹出的代码信息框中，query部分被红色箭头指向，显示为“Query{“bool”:{“must”:\[{“term”:{“city_code”:{“value”:“010”}}},{“terms”:{“service_type”:\[“00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Mi92b4bl7oGHhzxSSm9cPbhjn4V.png)

复制query语句，在kibana中调试。

![这是Elastic的Kibana控制台界面截图，左侧是编写的查询语句，发送GET请求至“/orders_seize/_search”，搜索条件包含对“city_code”“serve_type_id”“serve_time”“geo_distance”的筛选，用于按服务人员的服务范围、服务技能筛选抢单池订单；右侧是返回的匹配结果，显示两条符合条件的订单，其中一条为“依洁清日常保洁”服务，关联地址为北京市昌平区的对应场所，包含订单ID、服务类型、时间等抢单相关信息，该截图对应文档中“在Kibana中测试查询抢单语句”的操作环节，用于验证抢单池订单的查询逻辑。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WrTvb8ppVoUULoxTu44cPavQnzb.png)



#### 4）小结

搜索附近怎么实现？

对于地理坐标不变的场景，比如：搜索附近的酒店、搜索附近银行等，可以提前将搜索目标信息同步到Elasticsearch中，再通过Elasticsearch的geo去根据地理坐标去搜索附近几公里内的酒店、银行等。

使用geo时在索引中设置geo_point类型的字段，查询时需要传入经纬度坐标及距离（公里），将查询以此经纬度坐标为中心方圆几公里的信息。



对于地理坐标变的场景，比如：搜索附近的骑手、搜索附近的出租车，这里就需要在手机定时上报坐标到系统中，系统收到上传的坐标更新至Elasticsearch中，再通过geo去搜索附近的骑手、搜索附近的出租车等。



- 使用布尔查询：

  - 所在城市-must term查询
  - 服务类型-must term查询
  - 服务项目-must term查询
  - 服务人员的周边50km距离查询-must geoDistance 查询
- 按照距离升序排序
- 滚动分页





## 3.6 抢单

### 1) 抢单流程

<whiteboard token="Qpp1wKXqvhu1labWGfecKucenuh"></whiteboard>

说明：

参考抢券的方案，使用Redis+Lua的技术解决超卖问题，这里将抢单库存同步到Redis，每个订单的库存就是1。

抢单执行Lua脚本完成抢单，具体包括：扣减库存、抢单成功写入同步队列。

抢单同步队列的作用是通过异步任务将抢单结果信息同步到数据库。



### 2) Redis数据结构设计

1. **抢单同步队列**

缓存结构：Hash

RedisKey：QUEUE:ORDERS:SEIZE:SYNC:{citycode%10}

HashKey：订单id

HashValue:多值拼接中间用逗号分隔，分别为：被派单服务人员id/机构id、服务人员类型（2，服务人员，3：机构端）,是否是机器抢单（1：机器抢单，0：人工抢单）

过期时间：永不过期

<whiteboard token="UPkmwXUhGhrkASbRe2oclxOenZc"></whiteboard>

1. **抢单库存结构**

缓存结构：Hash

RedisKey：ORDERS:RESOURCE:STOCK:{citycode%10}

HashKey：订单id

HashValue: 库存，为1

过期时间：永不过期

缓存一致性方案：通过Canal进行同步

<whiteboard token="QiI6wUN1Vht0n5b0CnJcuLwAngd"></whiteboard>

### 3）阅读代码

1. 接口定义

接口名称：抢单接口

接口功能：抢单

接口路径：**POST**/orders-seize/worker

请求参数：

![图片展示了抢单接口的请求参数信息。参数名称为ordersSeizeReqDTO，参数说明是ordersSeizeReqDTO，请求类型为body，是否必须为true，数据类型为抢单模型，schema也为抢单模型。此外，还有id参数，参数说明为抢单id，请求类型为false，数据类型为integer(int64)。该图片与抢单接口定义上下文相关，直观呈现了接口请求参数的具体内容。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/ZrP5b9w0EomnXmxpxj1catPenEg.png)

响应参数：

![图片展示了抢单接口的响应参数。包含两个参数：msg，类型为string，参数说明为响应消息；code，类型为integer(int32)，参数说明为状态码，200表示成功，其它表示失败。该图片与抢单接口定义上下文相关，是对接口响应参数的具体说明，帮助理解接口返回数据的结构和含义。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NuSibylxPo1E96xkR1FcextRn8g.png)

*抢单结果 大于0抢单成功，-1/-2：库存数量不足，-3：抢单失败*

controller方法如下：

```JSON
package com.jzo2o.orders.seize.controller.worker;

import com.jzo2o.mvc.utils.UserContext;
import com.jzo2o.orders.seize.model.dto.request.OrdersSeizeReqDTO;
import com.jzo2o.orders.seize.model.dto.request.OrdersSerizeListReqDTO;
import com.jzo2o.orders.seize.model.dto.response.OrdersSeizeListResDTO;
import com.jzo2o.orders.seize.service.IOrdersSeizeService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;

@RestController
@Api(tags = "服务端 - 抢单相关接口")
@RequestMapping("/worker")
@Slf4j
public class WorkerOrdersSeizeController {

    @Resource
    private IOrdersSeizeService ordersSeizeService;

    @GetMapping("")
    @ApiOperation("服务端抢单列表")
    public OrdersSeizeListResDTO queryForList(OrdersSerizeListReqDTO ordersSerizeListReqDTO) {
        return ordersSeizeService.queryForList(ordersSerizeListReqDTO);
    }

    @PostMapping("")
    @ApiOperation("服务端抢单")
    public void seize(@RequestBody OrdersSeizeReqDTO ordersSeizeReqDTO) {
        ordersSeizeService.seize(ordersSeizeReqDTO.getId(), UserContext.currentUserId(), UserContext.currentUser().getUserType(), false);
    }
}
```

1. service方法

```JSON
@Service
@Slf4j
public class OrdersSeizeServiceImpl extends ServiceImpl<OrdersSeizeMapper, OrdersSeize> implements IOrdersSeizeService {

public void seize(Long id, Long serveProviderId, Integer serveProviderType, Boolean isMatchine) {

    // 1.抢单校验
    // 1.1.校验是否可以查询（认证通过，开启抢单）
    ServeProviderResDTO detail = serveProviderApi.getDetail(serveProviderId);
    if (!detail.getCanPickUp() || detail.getSettingsStatus() != 1) {
        throw new CommonException(ErrorInfo.Code.SEIZE_ORDERS_FAILD, SEIZE_ORDERS_RECEIVE_CLOSED);
    }
    // 1.2.校验抢单资源是否存在
    OrdersSeize ordersSeize = ordersSeizeService.getById(id);
    // 校验订单是否还存在，如果订单为空或id不存在，则认为订单已经不在
    if (ordersSeize == null || ObjectUtils.isNull(ordersSeize.getId())) {
        throw new CommonException(ErrorInfo.Code.SEIZE_ORDERS_FAILD, SEIZE_ORDERS_FAILD);
    }
    ConfigRegionInnerResDTO configRegionInnerResDTO = regionApi.findConfigRegionByCityCode(detail.getCityCode());


    // 城市编码最后1位序号
    int index = RedisUtils.getCityIndex(detail.getCityCode());
    // 1.3.校验时间冲突
    // 服务时间状态redisKey
    String serveProviderStateRedisKey = String.format(SERVE_PROVIDER_STATE, index);

    // 1.4.订单数量已达上限
    // 接单数量上限
    int receiveOrderMax = (serveProviderType == UserType.INSTITUTION) ? configRegionInnerResDTO.getInstitutionReceiveOrderMax() : configRegionInnerResDTO.getStaffReceiveOrderMax();

    Object ordersNum = redisTemplate.opsForHash().get(serveProviderStateRedisKey, serveProviderId + "_num");
    if(ObjectUtils.isNotNull(ordersNum) && NumberUtils.parseInt(ordersNum.toString()) >= receiveOrderMax){
        throw new CommonException(ErrorInfo.Code.SEIZE_ORDERS_FAILD, SEIZE_ORDERS_RECEIVE_ORDERS_NUM_OVER);
    }

    // 2.执行redis脚本

    // 2.1.redisKey
    // 抢单结果同步队列 redis key
    String ordersSeizeSyncRedisKey = RedisSyncQueueUtils.getQueueRedisKey(RedisConstants.RedisKey.ORERS_SEIZE_SYNC_QUEUE_NAME, index);
    // 库存redisKey
    String resourceStockRedisKey = String.format(ORDERS_RESOURCE_STOCK, index);

    log.debug("抢单key：{}，values:{}", Arrays.asList(ordersSeizeSyncRedisKey, resourceStockRedisKey),
            Arrays.asList(id, serveProviderId,serveProviderType));
    // 2.2.执行lua脚本
    Object execute = redisTemplate.execute(seizeOrdersScript,
            Arrays.asList(ordersSeizeSyncRedisKey, resourceStockRedisKey),
            id, serveProviderId,serveProviderType,isMatchine ? 1 : 0);
    log.debug("抢单结果 : {}", execute);

    // 3.处理lua脚本结果
    if (execute == null) {
        throw new CommonException(ErrorInfo.Code.SEIZE_ORDERS_FAILD, SEIZE_ORDERS_FAILD);
    }
    // 4.抢单结果判断 大于0抢单成功，-1/-2：库存数量不足，-3：抢单失败
    long result = NumberUtils.parseLong(execute.toString());
    if(result < 0) {
        throw new CommonException(ErrorInfo.Code.SEIZE_ORDERS_FAILD, SEIZE_ORDERS_FAILD);
    }
}
```



### 4）测试

测试流程：

启动Canal：docker start canal

启动RabbitMQ：docker start rabbitmq

启动Elasticsearch: docker start elasticsearch7.17.7

启动Kibana: docker start  kibana7.17.7

启动Redis：docker start redis

启动jzo2o-orders-manager

启动jzo2o-orders-seize

启动网关

启动服务端前端程序（通过浏览器运行）

服务人员登录服务端程序，进入抢单界面，查询抢单信息，点击“立即抢单”

预期结果：

抢单界面提示：抢单成功

![这张图片呈现的是抢单操作后的结果弹窗界面，弹窗顶部配有一个表情夸张的黄色大笑表情，主体文字为黑色加粗的“抢单成功”，下方设有红色的“确定”按钮，该弹窗属于测试流程中抢单成功后的预期结果展示，对应服务人员完成抢单操作后界面的提示内容。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/UFBxbVfZuoL5qJxO00rcKU6Pn1e.png)

在redis成功写入抢单同步记录

![图片展示的是Redis中抢单同步记录的存储情况。画面中显示服务器为192.168.101.68，键值对为QUEUE:ORDERS:SEIZE:SYNC:{0}，key为2311150000000000038，value为\[1716431678555406338,2,0\]。该图片与文档中测试抢单流程相关，用于说明在抢单成功后，抢单同步记录会在redis成功写入，此图直观呈现了记录存储的具体内容。](https://feishu.cn/file/OqJ5bYHvaoIeUbxBNovcahF3nEo)

在redis成功扣除库存

![图片展示的是Redis中关于抢单库存记录的界面。上方显示键名为“ORDERS:RESOURCE:STOCK:{0}”，TTL为-1。下方表格中，key为“23111500000000000038”，value为0。该图片与抢单测试流程相关，用于说明在Redis成功扣除库存后，抢单同步记录的情况，是抢单测试中对抢单成功后Redis库存记录状态的展示。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/GLpVbMNO1oCKL8xydQ8c6kUNnbc.png)



抢单失败：

如果抢单失败报下边的提示：

![图片展示的是抢单失败时的提示界面。界面背景为抢单大厅界面，弹出窗口中有一个黄色的哭脸表情，下方文字显示“抢单失败”，并说明“单子已经被抢走了”，底部有一个红色的“确定”按钮。该图片与文档中抢单失败的预期结果部分对应，直观呈现了抢单失败时系统给出的提示信息，帮助用户了解抢单失败的具体情况。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Up7MbF7M0onGyrx9MGdcKboFnyf.png)

或

![该图片展示了抢单业务的失败场景，呈现了抢单大厅的界面内容，包括保洁服务项目“日常保洁a”的信息、预约时间为2023年11月15日20:30的服务地址，以及0.30元的服务费用。界面中间弹出“接单失败！”的提示框，右下角有标注为“立即抢单”的红色按钮，该内容对应测试环节中抢单失败的预期结果展示，符合抢单测试流程里抢单失败的场景呈现。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/UYx8b3vproamWvxgUgzcGi6rnOc.png)

首先排查 jzo2o-orders-seize是否启动

在抢单service方法中断点跟踪程序运行。



### 4）小结

抢单是如何防止超卖的？



## 3.7 抢单结果异步处理

### 1）异步处理方案

抢单成功根据抢单结果进行异步处理：

交互流程如下：

<whiteboard token="RpKjwjNJbhrIUtbYpA4cIYO0nqd"></whiteboard>

**创建服务单**：

服务单记录了服务人员进行家政服务的信息，关键字段有：订单ID、订单金额、服务人员ID、服务单状态、服务时间、服务照片等。

服务单初始状态：待分配或待服务

机构抢单成功：待分配。

服务人员抢单成功：待服务。

**更新订单的状态：**

用户下单并支付完成后订单的状态为“派单中”，服务人员抢单成功订单状态为“待服务”，机构抢单成功订单状态为“待分配”。

**抢单结果同步成功删除抢单池等相关信息：**

抢单结果同步成功后删除抢单池中该订单的信息，这样服务人员在抢单界面无法查询到该订单。

删除数据库中抢单池的记录，通过Canal将Elasticsearch中对应的抢单记录删除。

删除Redis中该订单的库存信息。

删除Redis中抢单同步队列中的记录。



### 2) 表设计

**服务单表设计**

```Java
-- 服务任务 
-- auto-generated definition
create table orders_serve_0
(
    id                      bigint                             not null comment '任务id'
        primary key,
    user_id                 bigint                             null comment '属于哪个用户',
    serve_provider_id       bigint                             not null comment '服务人员或服务机构id',
    serve_provider_type     int                                null comment '服务者类型，2：服务端服务，3：机构端服务',
    institution_staff_id    bigint                             null comment '机构服务人员id',
    orders_id               bigint                             null comment '订单id',
    orders_origin_type      int                                not null comment '订单来源类型，1：抢单，2：派单',
    city_code               varchar(50)                        not null comment '城市编码',
    serve_type_id           bigint                             not null comment '服务分类id',
    serve_start_time        datetime                           null comment '预约时间',
    serve_item_id           bigint                             not null comment '服务项id',
    serve_status            int                                not null comment '任务状态',
    settlement_status       int      default 0                 not null comment '结算状态，0：不可结算，1：待结算，2：结算完成',
    real_serve_start_time   datetime                           null comment '实际服务开始时间',
    real_serve_end_time     datetime                           null comment '实际服务完结时间',
    serve_before_imgs       json                               null comment '服务前照片',
    serve_after_imgs        json                               null comment '服务后照片',
    serve_before_illustrate varchar(255)                       null comment '服务前说明',
    serve_after_illustrate  varchar(255)                       null comment '服务后说明',
    cancel_time             datetime                           null comment '取消时间,可以是退单，可以是取消时间',
    orders_amount           decimal(10, 2)                     null comment '订单金额',
    pur_num                 int                                null comment '购买数量',
    create_time             datetime default CURRENT_TIMESTAMP null comment '创建时间',
    update_time             datetime default CURRENT_TIMESTAMP null on update CURRENT_TIMESTAMP comment '更新时间',
    sort_by                 bigint                             null comment '排序字段（serve_start_time（秒级时间戳）+订单id（后6位））',
    display                 int      default 1                 null comment '服务端/机构端是否展示，1：展示，0：隐藏',
    is_deleted              int      default 0                 null comment '是否是逻辑删除',
    update_by               bigint                             null comment '更新人'
)
    comment '服务任务' charset = utf8mb4;
```

分库分表策略：

分库策略：按服务人员id分库，表达式：jzo2o-orders-\${serve_provider_id % 3}

分表策略：orders_serve\_\${(int)Math.floor(id % 10000000000 / 15000000)}

### 3）阅读代码

```JSON
package com.jzo2o.orders.seize.handler;

import cn.hutool.json.JSONArray;
import com.jzo2o.api.customer.ServeProviderApi;
import com.jzo2o.api.customer.dto.response.ServeProviderResDTO;
import com.jzo2o.common.utils.JsonUtils;
import com.jzo2o.common.utils.NumberUtils;
import com.jzo2o.orders.base.model.domain.OrdersSeize;
import com.jzo2o.orders.seize.service.IOrdersSeizeService;
import com.jzo2o.redis.handler.SyncProcessHandler;
import com.jzo2o.redis.model.SyncMessage;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.util.List;

/**
 * 抢单成功同步任务
 */
@Component("ORDERS:SEIZE:SYNC")
@Slf4j
public class SeizeSyncProcessHandler implements SyncProcessHandler<Object> {

    @Resource
    private IOrdersSeizeService ordersSeizeService;


    @Resource
    private ServeProviderApi serveProviderApi;

    @Override
    public void batchProcess(List<SyncMessage<Object>> multiData) {
        throw new RuntimeException("不支持批量处理");
    }

    @Override
    public void singleProcess(SyncMessage<Object> singleData) {
        log.info("抢单结果同步开始 id ： {}",singleData.getKey());
        // 抢单信息放在value中，内容格式：[serveProviderId,serveProviderType,isMatchine（0，表示人工抢单，1：表示机器抢单）]
        JSONArray seizeResult = JsonUtils.parseArray(singleData.getValue());
        // 服务人员或机构id
        Long serveProviderId = seizeResult.getLong(0);
        // 用户类型
        Integer serveProviderType = seizeResult.getInt(1);
        // 是否是机器抢单
        boolean isMatchine = seizeResult.getBool(2);

        // 抢单id
        Long seizeId = NumberUtils.parseLong(singleData.getKey());
        // 抢单不在无需继续处理
        OrdersSeize ordersSeize = ordersSeizeService.getById(seizeId);
        if (ordersSeize == null) {
            return;
        }

        // 处理抢单结果
        ordersSeizeService.seizeOrdersSuccess(ordersSeize, serveProviderId, serveProviderType, isMatchine);
        log.info("抢单结果同步结束 id ： {}",singleData.getKey());
    }
}
```

ordersSeizeService.seizeOrdersSuccess()方法如下：

```JSON
@Transactional(rollbackFor = Exception.class)
    public void seizeOrdersSuccess(OrdersSeize ordersSeize, Long serveProviderId, Integer serveProviderType, Boolean isMatchine) {

        // 1.校验服务单是否已经生成
        OrdersServe ordersServeInDb = ordersServeService.findById(ordersSeize.getId());
        if(ordersServeInDb != null){
            return;
        }
        // 2.生成服务单,
        OrdersServe ordersServe = BeanUtils.toBean(ordersSeize, OrdersServe.class);
        ordersServe.setCreateTime(null);
        ordersServe.setUpdateTime(null);
        // 服务单状态 机构抢单状态：待分配；服务人员抢单状态：待服务
        int serveStatus = UserType.WORKER == serveProviderType ? ServeStatusEnum.NO_SERVED.getStatus() : ServeStatusEnum.NO_ALLOCATION.getStatus();
        // 服务单来源类型,人工抢单来源抢单，值为1；机器抢单来源派单，值为2
        int ordersOriginType = isMatchine ? OrdersOriginType.DISPATCH : OrdersOriginType.SEIZE;
        ordersServe.setOrdersOriginType(ordersOriginType);
        ordersServe.setServeStatus(serveStatus);
        ordersServe.setServeProviderId(serveProviderId);
        ordersServe.setServeProviderType(serveProviderType);
        if(!ordersServeService.save(ordersServe)){
            return;
        }

        // 3.当前订单数量
        serveProviderSyncService.countServeTimesAndAcceptanceNum(serveProviderId, serveProviderType);

        String resourceStockRedisKey = String.format(ORDERS_RESOURCE_STOCK, RedisUtils.getCityIndex(ordersSeize.getCityCode()));
        Object stock = redisTemplate.opsForHash().get(resourceStockRedisKey, ordersSeize.getId());
        if (ObjectUtils.isNull(stock) || NumberUtils.parseInt(stock.toString()) <= 0) {
            ordersDispatchMapper.deleteById(ordersSeize.getId());
            ordersSeizeService.removeById(ordersSeize.getId());
            redisTemplate.opsForHash().delete(resourceStockRedisKey, ordersSeize.getId());
        }

        //状态机修改订单状态
//        OrderSnapshotDTO orderSnapshotDTO = OrderSnapshotDTO.builder()
//                .ordersStatus(OrderStatusEnum.NO_SERVE.getStatus()).build();
        Orders orders = ordersMapper.selectById(ordersSeize.getId());
        orderStateMachine.changeStatus(orders.getUserId(),String.valueOf(ordersSeize.getId()), OrderStatusChangeEventEnum.DISPATCH);

    }
```

xxl-job任务方法：

```JSON

package com.jzo2o.orders.seize.handler;

/**
 * 抢单xxl-job任务
 */
@Component
@Slf4j
public class SeizeJobHandler {

/**
 * 抢单成功同步任务
 */
@XxlJob("seizeSyncJob")
public void seizeSyncJob() {
    syncManager.start(ORERS_SEIZE_SYNC_QUEUE_NAME, RedisSyncQueueConstants.STORAGE_TYPE_HASH, RedisSyncQueueConstants.MODE_SINGLE);
}
...
```



### 4) 测试

1. 在xxl-job中添加 seizeSyncJob任务

启动xxl-job: docker start xxl-job-admin

![这张图片是xxl-job任务配置页面，对应文档中抢单结果异步处理的测试环节里的任务配置步骤。页面分为基础配置、调度配置、任务配置、高级配置四个区域，其中基础配置里执行器设为抢单调度器、负责人为xxl job，任务描述为抢单结果同步；调度配置中调度类型选择CRON，Cron表达式填0 10 * * * ?；任务配置里运行模式为BEAN，JobHandler为seizeSyncJob；高级配置里路由策略为第一个、子任务ID为空、调度过期策略为忽略，其余配置多为默认值，底部有保存和取消按钮，该页面用于配置测试所需的seizeSyncJob任务。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DKcubZD7poOGrFxJgYtcQteWn2f.png)

1. 在seizeOrdersSuccess方法的断点
2. 启动xxl-job任务
3. 跟踪seizeOrdersSuccess方法的断点‘

预期结果：

orders_serve服务单数据添加成功，状态为1（待服务）

订单更新为200（待服务）

该订单在抢单池的记录删除



示例：

服务单表中添加服务单成功：

```JSON
{
  "id": 2311150000000000038,
  "user_id": null,
  "serve_provider_id": 1716431678555406338,
  "serve_provider_type": 2,
  "institution_staff_id": null,
  "orders_id": null,
  "orders_origin_type": 1,
  "city_code": "010",
  "serve_type_id": 1678649931106705409,
  "serve_start_time": "2023-11-15 20:30:00",
  "serve_item_id": 1685894105234755585,
  "serve_status": 1,
  "settlement_status": 0,
  "real_serve_start_time": null,
  "real_serve_end_time": null,
  "serve_before_imgs": null,
  "serve_after_imgs": null,
  "serve_item_img": "https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/aa6489e5-cd92-42f0-837a-952c99653b8b.png",
  "serve_before_illustrate": null,
  "serve_after_illustrate": null,
  "cancel_time": null,
  "orders_amount": 1.00,
  "pur_num": 1,
  "create_time": "2023-11-15 17:01:20",
  "update_time": "2023-11-15 17:01:20",
  "sort_by": 1700051400038,
  "display": 1,
  "update_by": null
}
```

订单更新更改为服务中：

```JSON
{
  "id": 2311150000000000038,
  "user_id": 1716346406098296832,
  "serve_type_id": 1678649931106705409,
  "serve_type_name": "保洁清",
  "serve_item_id": 1685894105234755585,
  "serve_item_name": "日常保洁",
  "serve_item_img": "https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/aa6489e5-cd92-42f0-837a-952c99653b8b.png",
  "unit": 1,
  "serve_id": 1693815624114970626,
  "orders_status": 200,
  "pay_status": 4,
  "refund_status": null,
  "price": 1.00,
  "pur_num": 1,
  "total_amount": 1.00,
  "real_pay_amount": 1.00,
  "discount_amount": 0.00,
  "city_code": "010",
  "serve_address": "北京市北京市昌平区北京市昌平区回龙观街道弘文恒瑞文化传播公司正泽商务中心",
  "contacts_phone": "13333333333",
  "contacts_name": "苗先生",
  "serve_start_time": "2023-11-15 20:30:00",
  "lon": 116.34351,
  "lat": 40.06024,
  "pay_time": "2023-11-15 15:06:11",
  "evaluation_time": null,
  "trading_order_no": 1724685206700535808,
  "transaction_id": "1724685206700535809",
  "refund_no": null,
  "refund_id": null,
  "trading_channel": "WECHAT_PAY",
  "display": 1,
  "sort_by": 1700051400038,
  "create_time": "2023-11-15 15:06:10",
  "update_time": "2023-11-15 17:01:20"
}
```



### 5）小结

抢单结果如何从redis同步到MySQL？

我们项目开发了一个同步组件，用于将Redis Hash结构中的数据同步到MySQL，具体是这样做的：

1、使用线程池从多个同步队列中查询数据，每个线程处理一个同步队列。

同步队列的个数通常配置10到20即可。

在同步时为了保证一个线程只处理一个队列，这里使用的分布式锁进行控制。

2、使用redisTemplate.opsForHash().scan(H key, ScanOptions options)方法从hash表获取数据。

3、读取到数据后将数据库写入MySQL的服务表最后将写入成功的的数据表示已经同步成功，

4、将从Redis的Hash表中删除（库存和同步队列）。

5、删除Mysql中的抢单池和派单池（将删除的记录通过Canal+MQ同步给es）



秒杀异步处理怎么实现的？

在秒杀抢购场景中，为了流量削峰可以在Redis存储秒杀结果，再通过定时任务将秒杀结果同步到数据库中。

本项目使用了一个数据同步组件，将Redis中的数据异步同步到MySQL，具体的做法是：

将Redis中的数据异步同步到MySQL参考“如何将Redis中的数据异步同步到MySQL？”



## 3.8 整体测试

抢单功能开发完成对抢单流程进行测试。

1. 用户下单
2. 服务人员抢单
3. 服务人员抢单成功，查询自己的服务单
4. 开始服务
5. 完成服务



# 4 电商限时抢购

## 4.1 业务流程

### 1）运营端创建秒杀活动

设置一周的秒杀活动

![图片展示的是电商限时抢购中运营端创建秒杀活动后的秒杀活动列表界面。列表包含活动名称、开始时间、申请截止时间、活动状态、申请规则、操作等信息。如2024 - 11 - 23秒杀活动，开始时间为2024 - 11 - 23 00:00:00，申请截止时间同，活动状态为未开始，操作有编辑、管理、关闭。该图与上下文介绍的运营端创建秒杀活动后，可查看秒杀活动列表的内容相契合。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Fk8zbrqF1o4rShxSUcJcZUaBnRd.png)

设置每天的秒杀时间段

![这是电商限时抢购业务中运营端设置秒杀活动时的界面，页面顶部设有“秒杀活动列表”“秒杀活动设置”两个选项卡，当前处于“秒杀活动设置”界面。界面核心区域为每日场次设置，以橙色方块标识可选的秒杀时段，覆盖0:00至23:00的整点钟点时段。页面下方设有“秒杀规则”输入框，底部还有“返回”和“提交”按钮，右上角则有“创建一周秒杀活动”的操作按钮，该界面用于完成前文所述的运营端秒杀活动的时段配置等设置工作。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/D1a4beUyKoXN5cxexYXc35oInUf.png)

### 2）店铺端添加秒杀商品

![这张图片是电商秒杀抢购业务中运营端的秒杀活动管理页面，页面顶部有活动名称搜索框、活动状态下拉选择框、活动时间选择区域，还有橙色的“搜索”按钮和“重置”按钮。页面下方展示了两条秒杀活动信息，分别为2024-11-23秒杀活动、2024-11-22秒杀活动，两条活动的状态均为“未开始”，每条活动信息对应列有活动名称、活动开始时间、报名截止时间，还有时间场次明细，每条信息后配有蓝色的“管理”操作按钮。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/W8OjbUOdhoRuncxxWnWcfmORnVk.png)

点击“管理”进入添加商品界面

![这是电商限时抢购活动中店铺端添加秒杀商品环节的操作界面，界面顶部显示已设置的活动信息，活动名称为2024-11-23秒杀活动，活动开始时间、报名截止时间均为2024-11-23 00:00:00，右侧还标注了对应可选的时间场次。界面中间有“选择商品”选项，下方的时间刻度条可选择时段，商品信息表格中目前暂无数据，底部设有“返回”和“提交”按钮，用于完成后续的提交申请操作。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Exsqbh0iioVqfjxLPi2cv14vnib.png)

![图片展示的是电商限时抢购中店铺端添加秒杀商品时的界面。界面上方有“搜索范围”和“商品名称”输入框，中间有“请选择商品分类”下拉框及“搜索”按钮。下方列出多款商品，如荣耀80、小米802等，每款商品有图片、名称、库存类型、价格及已选中状态标识。底部有“取消”和“确定”按钮。该图与上下文关系紧密，是店铺端添加秒杀商品操作流程中设置商品的界面展示。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/TtD7bQMzUoydivxjVOicO0tonDg.png)

添加成功下一步设置活动价格

![这张图是店铺端添加秒杀商品流程中的提交申请环节页面，上方显示2024-11-23秒杀活动的信息，包括活动开始时间、报名截止时间，以及设置的秒杀时间场次；下方是商品列表，共4款待申请的商品，包含荣耀80黑色128G、荣耀80红色256G、小米802绿色256G、荣耀80红色256G，分别标注了商品价格、库存、活动价格、状态，操作栏带有“删除”按钮，页面还有橙色的“选择商品”按钮与“返回”按钮，此时商品状态均为“未申请”，对应流程中提交后系统自动审核通过的前置页面。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/VMeFbr300oKLebxj8GbcBw7knQe.png)

点击提交申请秒杀商品，系统设置为自动审核，提交后自动审核通过，如下图

![这张图是电商秒杀系统的后台操作界面，对应店铺端添加秒杀商品流程的提交申请环节。界面上方显示活动名称为“2024-11-23 秒杀活动”，活动开始时间与报名截止时间均为2024年11月23日00:00:00，右侧列有当日可设置的时间场次；中间区域显示了四款待添加为秒杀商品的信息，包括商品名称、原价格、库存，以及对应的活动价格，当前选中了时间场次中的11:00档；所有商品的活动状态均标注为“通过”，每个商品右侧设有删除按钮，界面左下角设有“返回”与“提交”按钮，完成设置后提交即可完成秒杀商品的申请，系统将自动审核通过。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/C3GTbAdIZo7DGSxxuUrcihXzndf.png)

### 3）用户端进行秒杀

进入小程序首页，

![图片展示的是青橙电商小程序首页界面。上方有“青橙电商”标识及“精品零食抢先购”搜索框。中间大图宣传“约会女王 第二件半价 女王节限时促销”，并有皇冠装饰。下方有多个功能图标，其中“秒杀活动”图标被红色箭头指向。最下方是“300元 满1200元可用 新人专享 立即领取”的优惠券区域。该图片与上下文关系紧密，直观呈现了用户端进入小程序首页后可看到的界面及“秒杀活动”入口位置。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/E7I4bWvgFomPRvxXogbczxEwnOb.png)

点击“秒杀活动”进入秒杀界面 

![这是青橙秒杀的小程序秒杀界面，顶部标注了“秒杀”主题，还显示了10:00正处于抢购中，11:00、12:00、13:00为即将开始的秒杀时段。界面内展示了三款秒杀商品，分别是售价900元的荣耀80白色256G款、售价900元的荣耀80白色128G款，以及售价1元的荣耀80黑色128G款，每款商品旁均标注原价为900元，已抢0%，并设有“抢”的操作按钮，对应文档中用户端点击“秒杀活动”进入秒杀界面后的页面内容。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/GxmpbBs9PocKvIxzmjIcsNN3nxd.png)

点击具体的商品进入商品详情页面。

![这是电商限时抢购活动中的商品秒杀界面，展示的是秒杀商品荣耀80黑色128G版。该商品标注价格为1.00元，原价为1.00元，处于限时抢购状态，页面显示“距离活动结束”的00:00:00倒计时，标识该秒杀活动处于倒计时结束的临近或已结束状态。界面底部设有“立即购买”按钮，还显示商品备注为“荣耀80非常好”，页面左下方有“店铺”“客服”“购物车”的功能图标，右上角标注当前是活动轮次的1/2。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Xudxbo5tEo0MunxqpJKc2FKxn3Y.png)

点击“立即购买”

![这张图片是电商小程序的秒杀商品详情界面，对应文档中用户端进行秒杀的环节，呈现的是一款标价为1.00元（标注原价为1.00元）的手机商品。界面显示该商品已选配置为128G内存、黑色款，剩余库存为89件。界面设有颜色、内存的选择选项，其中黑色、128G的选项被标红突出显示，底部还有橙色的“立即购买”按钮，与文档中用户端秒杀流程里点击商品详情后点击“立即购买”的操作步骤相呼应。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EhiNbHbgeoliFDxhmMycIKB0nRg.png)

![图片展示的是电商限时抢购中用户端进行秒杀时的支付页面。页面上方显示“填写订单”，右上角有三个点和一个放大镜图标。中间部分呈现了商品信息，为“家家乐 荣耀80 黑色 128G”，价格为¥1.00，有“不开发票”选项。下方有“备注信息”输入框，以及“商品合计”“优惠券”“优惠金额”“活动优惠”等信息栏，当前优惠券、优惠金额、活动优惠均为0.00。页面底部有“提交订单”按钮。该图与上下文介绍的用户端进行秒杀流程中点击“提交订单”进入支付页面的内容相契合。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Qj4kbQz3BoTNCCxCrDkcmw5LnFh.png)

点击“提交订单”，进入支付页面，支付成功商品购买成功。



## 4.2 系统设计

### 1）表结构

![图片展示了电商限时抢购系统中商品秒杀活动申请表、商品秒杀活动表、秒杀商品信息表的表结构。商品秒杀活动申请表包含id、create by、create time、delete flag、update by、update time、promotion apply status、promotion id、promotion name、promotion type、quantity、sales num、sku id、store id、store name、start time、end time、seckill rule、goods num、scope id、scope type等字段。商品秒杀活动表有id、create by、create time、delete flag、update by、update time、promotion name、store id、store name、start time、end time、seckill rule、goods num、scope type等字段。秒杀商品信息表包含id、create by、create time、delete flag、update by、update time、promotion id、promotion type、store id、store name、sku id、start time、thumbnail、title、category_path、goods type、limit num、num、promotion、point、goods id等字段。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/ZoksbeIm7oBM8kxDKVpcUOuknsb.png)

商品秒杀活动表：记录商品秒杀活动的起止时间、店铺信息等。

商品秒杀活动申请表：记录店铺申请的秒杀商品信息。

秒杀商品信息：记录了审批通过的秒杀商品。



### 2）Redis

1. **秒杀商品活动信息**

按天为单位进行缓存

结构：String

key: 年月日

Value: 当天的秒杀商品信息

过期时间：永不过期，由定时任务维护



示例：

key:SECKILL_CACHE::2024-11-16

Value:

```Java
[
  "java.util.ArrayList",
  [
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 0,
        "startTime": "1731686400",
        "distanceStartTime": "0",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 1,
        "startTime": "1731690000",
        "distanceStartTime": "0",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 2,
        "startTime": "1731693600",
        "distanceStartTime": "0",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 3,
        "startTime": "1731697200",
        "distanceStartTime": "0",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 4,
        "startTime": "1731700800",
        "distanceStartTime": "0",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 5,
        "startTime": "1731704400",
        "distanceStartTime": "0",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 6,
        "startTime": "1731708000",
        "distanceStartTime": "0",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 7,
        "startTime": "1731711600",
        "distanceStartTime": "0",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 8,
        "startTime": "1731715200",
        "distanceStartTime": "0",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 9,
        "startTime": "1731718800",
        "distanceStartTime": "0",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 10,
        "startTime": "1731722400",
        "distanceStartTime": "0",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 11,
        "startTime": "1731726000",
        "distanceStartTime": "0",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [
            [
              "com.jzo2o.mall.promotion.model.dto.SeckillGoodsDTO",
              {
                "seckillId": "1857604893620105217",
                "timeLine": 11,
                "goodsId": "1854464715971907585",
                "point": null,
                "skuId": "1854464716361977857",
                "goodsName": "荣耀80 白色 256G",
                "goodsImage": "https://jzo2o-oss.oss-cn-hangzhou.aliyuncs.com/STORE/1376417684140326912/undefined/287487e68c804dc2bda95ad2f2f791c1.png?x-oss-process=image/resize,h_100,m_lfit",
                "storeId": "1376433565247471616",
                "price": 900.0,
                "quantity": 11,
                "salesNum": 0,
                "originalPrice": 900.0,
                "promotionGoods": [
                  "com.jzo2o.mall.promotion.model.domain.PromotionGoods",
                  {
                    "id": "1857625826636382209",
                    "createBy": "13011111111",
                    "createTime": "2024-11-16 11:24:44",
                    "updateBy": "1376417684140326912",
                    "updateTime": "2024-11-16 11:24:44",
                    "deleteFlag": false,
                    "storeId": "1376433565247471616",
                    "storeName": "家家乐",
                    "goodsId": "1854464715971907585",
                    "skuId": "1854464716361977857",
                    "goodsName": "荣耀80 白色 256G",
                    "thumbnail": "https://jzo2o-oss.oss-cn-hangzhou.aliyuncs.com/STORE/1376417684140326912/undefined/287487e68c804dc2bda95ad2f2f791c1.png?x-oss-process=image/resize,h_100,m_lfit",
                    "startTime": [
                      "java.util.Date",
                      "2024-11-17 11:00:00"
                    ],
                    "endTime": [
                      "java.util.Date",
                      "2024-11-17 11:59:59"
                    ],
                    "promotionId": "1857604893620105217",
                    "promotionType": "SECKILL",
                    "goodsType": "PHYSICAL_GOODS",
                    "title": "2024-11-17 秒杀活动",
                    "num": 0,
                    "originalPrice": 900.0,
                    "price": 900.0,
                    "points": null,
                    "limitNum": null,
                    "quantity": 11,
                    "categoryPath": "1348576427264204941,1348576427264204942,1348576427264204943",
                    "scopeType": "PORTION_GOODS",
                    "scopeId": null
                  }
                ]
              }
            ],
            [
              "com.jzo2o.mall.promotion.model.dto.SeckillGoodsDTO",
              {
                "seckillId": "1857604893620105217",
                "timeLine": 11,
                "goodsId": "1797528556385894402",
                "point": null,
                "skuId": "1797528556507529217",
                "goodsName": "荣耀80 黑色 128G",
                "goodsImage": "https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/STORE/1376417684140326912/1376433565247471616/6774bb3d4a714d34a75ffde79f290221.jpg?x-oss-process=image/resize,h_100,m_lfit",
                "storeId": "1376433565247471616",
                "price": 1.0,
                "quantity": 87,
                "salesNum": 0,
                "originalPrice": 1.0,
                "promotionGoods": [
                  "com.jzo2o.mall.promotion.model.domain.PromotionGoods",
                  {
                    "id": "1857625826644770817",
                    "createBy": "13011111111",
                    "createTime": "2024-11-16 11:24:44",
                    "updateBy": "1376417684140326912",
                    "updateTime": "2024-11-16 11:24:44",
                    "deleteFlag": false,
                    "storeId": "1376433565247471616",
                    "storeName": "家家乐",
                    "goodsId": "1797528556385894402",
                    "skuId": "1797528556507529217",
                    "goodsName": "荣耀80 黑色 128G",
                    "thumbnail": "https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/STORE/1376417684140326912/1376433565247471616/6774bb3d4a714d34a75ffde79f290221.jpg?x-oss-process=image/resize,h_100,m_lfit",
                    "startTime": [
                      "java.util.Date",
                      "2024-11-17 11:00:00"
                    ],
                    "endTime": [
                      "java.util.Date",
                      "2024-11-17 11:59:59"
                    ],
                    "promotionId": "1857604893620105217",
                    "promotionType": "SECKILL",
                    "goodsType": "PHYSICAL_GOODS",
                    "title": "2024-11-17 秒杀活动",
                    "num": 0,
                    "originalPrice": 1.0,
                    "price": 1.0,
                    "points": null,
                    "limitNum": null,
                    "quantity": 87,
                    "categoryPath": "1348576427264204941,1348576427264204942,1348576427264204943",
                    "scopeType": "PORTION_GOODS",
                    "scopeId": null
                  }
                ]
              }
            ],
            [
              "com.jzo2o.mall.promotion.model.dto.SeckillGoodsDTO",
              {
                "seckillId": "1857604893620105217",
                "timeLine": 11,
                "goodsId": "1797528556385894402",
                "point": null,
                "skuId": "1797528556503334913",
                "goodsName": "荣耀80 红色 256G",
                "goodsImage": "https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/STORE/1376417684140326912/1376433565247471616/db4c234f10b2475ead284ef037b1163f.png?x-oss-process=image/resize,h_100,m_lfit",
                "storeId": "1376433565247471616",
                "price": 1.0,
                "quantity": 88,
                "salesNum": 0,
                "originalPrice": 1.0,
                "promotionGoods": [
                  "com.jzo2o.mall.promotion.model.domain.PromotionGoods",
                  {
                    "id": "1857625826644770818",
                    "createBy": "13011111111",
                    "createTime": "2024-11-16 11:24:44",
                    "updateBy": "1376417684140326912",
                    "updateTime": "2024-11-16 11:24:44",
                    "deleteFlag": false,
                    "storeId": "1376433565247471616",
                    "storeName": "家家乐",
                    "goodsId": "1797528556385894402",
                    "skuId": "1797528556503334913",
                    "goodsName": "荣耀80 红色 256G",
                    "thumbnail": "https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/STORE/1376417684140326912/1376433565247471616/db4c234f10b2475ead284ef037b1163f.png?x-oss-process=image/resize,h_100,m_lfit",
                    "startTime": [
                      "java.util.Date",
                      "2024-11-17 11:00:00"
                    ],
                    "endTime": [
                      "java.util.Date",
                      "2024-11-17 11:59:59"
                    ],
                    "promotionId": "1857604893620105217",
                    "promotionType": "SECKILL",
                    "goodsType": "PHYSICAL_GOODS",
                    "title": "2024-11-17 秒杀活动",
                    "num": 0,
                    "originalPrice": 1.0,
                    "price": 1.0,
                    "points": null,
                    "limitNum": null,
                    "quantity": 88,
                    "categoryPath": "1348576427264204941,1348576427264204942,1348576427264204943",
                    "scopeType": "PORTION_GOODS",
                    "scopeId": null
                  }
                ]
              }
            ],
            [
              "com.jzo2o.mall.promotion.model.dto.SeckillGoodsDTO",
              {
                "seckillId": "1857604893620105217",
                "timeLine": 11,
                "goodsId": "1782597676706136065",
                "point": null,
                "skuId": "1795709064865415169",
                "goodsName": "小米802 绿色 256G",
                "goodsImage": "https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/STORE/1376417684140326912/1376433565247471616/4ca2677b1b7b4b3a9f6e88857dded3e3.png?x-oss-process=image/resize,h_100,m_lfit",
                "storeId": "1376433565247471616",
                "price": 0.1,
                "quantity": 23,
                "salesNum": 0,
                "originalPrice": 0.1,
                "promotionGoods": [
                  "com.jzo2o.mall.promotion.model.domain.PromotionGoods",
                  {
                    "id": "1857625826644770819",
                    "createBy": "13011111111",
                    "createTime": "2024-11-16 11:24:44",
                    "updateBy": "1376417684140326912",
                    "updateTime": "2024-11-16 11:24:44",
                    "deleteFlag": false,
                    "storeId": "1376433565247471616",
                    "storeName": "家家乐",
                    "goodsId": "1782597676706136065",
                    "skuId": "1795709064865415169",
                    "goodsName": "小米802 绿色 256G",
                    "thumbnail": "https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/STORE/1376417684140326912/1376433565247471616/4ca2677b1b7b4b3a9f6e88857dded3e3.png?x-oss-process=image/resize,h_100,m_lfit",
                    "startTime": [
                      "java.util.Date",
                      "2024-11-17 11:00:00"
                    ],
                    "endTime": [
                      "java.util.Date",
                      "2024-11-17 11:59:59"
                    ],
                    "promotionId": "1857604893620105217",
                    "promotionType": "SECKILL",
                    "goodsType": "PHYSICAL_GOODS",
                    "title": "2024-11-17 秒杀活动",
                    "num": 0,
                    "originalPrice": 0.1,
                    "price": 0.1,
                    "points": null,
                    "limitNum": null,
                    "quantity": 23,
                    "categoryPath": "1348576427264204941,1348576427264204942,1348576427264204943",
                    "scopeType": "PORTION_GOODS",
                    "scopeId": null
                  }
                ]
              }
            ],
            [
              "com.jzo2o.mall.promotion.model.dto.SeckillGoodsDTO",
              {
                "seckillId": "1857604893620105217",
                "timeLine": 11,
                "goodsId": "1782595491175956482",
                "point": null,
                "skuId": "1783321570509451265",
                "goodsName": "荣耀70 白色 254G",
                "goodsImage": "https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/STORE/1376417684140326912/1376433565247471616/eac762d7e4ff4ec8bec211c5d683b7b7.jpg?x-oss-process=image/resize,h_100,m_lfit",
                "storeId": "1376433565247471616",
                "price": 0.1,
                "quantity": 5,
                "salesNum": 0,
                "originalPrice": 0.1,
                "promotionGoods": [
                  "com.jzo2o.mall.promotion.model.domain.PromotionGoods",
                  {
                    "id": "1857625826644770820",
                    "createBy": "13011111111",
                    "createTime": "2024-11-16 11:24:44",
                    "updateBy": "1376417684140326912",
                    "updateTime": "2024-11-16 11:24:44",
                    "deleteFlag": false,
                    "storeId": "1376433565247471616",
                    "storeName": "家家乐",
                    "goodsId": "1782595491175956482",
                    "skuId": "1783321570509451265",
                    "goodsName": "荣耀70 白色 254G",
                    "thumbnail": "https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/STORE/1376417684140326912/1376433565247471616/eac762d7e4ff4ec8bec211c5d683b7b7.jpg?x-oss-process=image/resize,h_100,m_lfit",
                    "startTime": [
                      "java.util.Date",
                      "2024-11-17 11:00:00"
                    ],
                    "endTime": [
                      "java.util.Date",
                      "2024-11-17 11:59:59"
                    ],
                    "promotionId": "1857604893620105217",
                    "promotionType": "SECKILL",
                    "goodsType": "PHYSICAL_GOODS",
                    "title": "2024-11-17 秒杀活动",
                    "num": 0,
                    "originalPrice": 0.1,
                    "price": 0.1,
                    "points": null,
                    "limitNum": null,
                    "quantity": 5,
                    "categoryPath": "1348576427264204941,1348576427264204942,1348576427264204943",
                    "scopeType": "PORTION_GOODS",
                    "scopeId": null
                  }
                ]
              }
            ]
          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 12,
        "startTime": "1731729600",
        "distanceStartTime": "1709",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 13,
        "startTime": "1731733200",
        "distanceStartTime": "5309",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 14,
        "startTime": "1731736800",
        "distanceStartTime": "8909",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 15,
        "startTime": "1731740400",
        "distanceStartTime": "12509",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 16,
        "startTime": "1731744000",
        "distanceStartTime": "16109",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 17,
        "startTime": "1731747600",
        "distanceStartTime": "19709",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 18,
        "startTime": "1731751200",
        "distanceStartTime": "23309",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 19,
        "startTime": "1731754800",
        "distanceStartTime": "26909",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 20,
        "startTime": "1731758400",
        "distanceStartTime": "30509",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 21,
        "startTime": "1731762000",
        "distanceStartTime": "34109",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 22,
        "startTime": "1731765600",
        "distanceStartTime": "37709",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ],
    [
      "com.jzo2o.mall.promotion.model.dto.SeckillTimelineDTO",
      {
        "timeLine": 23,
        "startTime": "1731769200",
        "distanceStartTime": "41309",
        "seckillGoodsList": [
          "java.util.ArrayList",
          [

          ]
        ]
      }
    ]
  ]
]
```



1. 秒杀**商品库存**

String结构：

Key: SeckillSku_商品id

value:库存值

过期时间：永不过期，由定时任务维护

示例：

Kye: {SeckillSku}\_1797528556507529217

value：100



1. **秒杀交易信息**

String 结构

Key:  SeckillOrder_订单ID

value：交易信息(json)



示例：

Kye: {SeckillOrder}\_17975285565543543

value：

![图片展示的是秒杀交易信息的示例，其结构为String类型，Key为SeckillOrder_订单ID，Value为交易信息（json）。图片中呈现了部分交易信息，包括TradeDTO、CartDTO、PriceDetailDTO等对象的类名、序列化名、storeID、storeName等字段内容。该图片与文档中秒杀交易信息的表结构描述对应，直观呈现了其存储形式和部分数据内容。](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/V9NCbb7ADoMdkhx7ZmrcShllnLz.png)





1. **秒杀成功队列**

结构：hash

大Key: QUEUE:SECKILL:SYNC:{用户id%10}

小Key: 订单id

Value: 用户id

过期时间：永不过期，由秒杀结果同步任务维护



示例：

Key: QUEUE:SECKILL:SYNC:{7}

小Key: 17975285565543543

Value: 17975285667655743



### 3）交互流程

1. 通过定时任务将秒杀商品的库存同步到Redis、将秒杀活动及活动的商品信息同步到Redis
2. 用户通过redis查询秒杀活动及秒杀活动中的商品
3. 用户点击立即抢购秒杀商品，通过Redis+Lua方式扣减库存

如果扣减库存成功在redis记录购买的交易信息，并将购买商品的信息记录存储到Redis 的**秒杀成功队列**

1. lua脚本扣减库存成功就算秒杀成功，后端向前端返回交易信息（此时交易信息还在redis），用户开始支付
2. 通过定时任务加多线程机制将支付成功的交易信息同步到MySQL数据库的订单表中。
3. 如果用户未支付时取消订单或订单超时未支付自动取消，也是通过redis+lua方式去回滚库存，回滚库存成功删除redis中的秒杀交易信息。

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 云岚到家项目 / 第十章 秒杀抢购 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/E3BCwLrt3iFC2Hk7Nrqc60jlnmE)
