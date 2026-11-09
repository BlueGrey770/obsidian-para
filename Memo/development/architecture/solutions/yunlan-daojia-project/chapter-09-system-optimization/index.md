---
type: memo
date: 2026-11-06
aliases:
  - "第九章 系统优化"
---
# 第九章 系统优化

## 学习目标：

能说出什么是状态机

能说出如何实现一个订单状态机

能够基于状态机组件实现订单状态机

能够说出项目为什么进行分库分表

能够说出分库分表的四种形式

能够说出分库分表常用的技术方案

能够开发ShardingSphere-JDBC入门程序

能够说出家政项目分库分表的技术方案

能够完成对订单数据库进行分库分表

能够说出订单查询的优化方案

能够优化用户端订单列表

能够对运营端订单列表进行优化

能够说出订单快照的实现方案

能够使用订单快照优化订单详情接口

能够说出项目的冷热分离方案

能够对订单数据进行冷热分离

能够对订单订单按年分表



实战：

1. 取消订单使用状态机
2. 运营端订单列表优化
3. 历史订单实现按年分表。







## 1 状态机

### 1.1 什么是状态机

目标：能说出什么是状态机

#### 1）**当前存在的问题**

在预约下单模块设计订单状态共有7种，如下图：

<whiteboard token="E0WYwFmszhSpBIbJdEUcZbDgnjg"></whiteboard>

目前我们使用了待支付、派单中两种状态，在代码中我们发现存在对订单状态进行硬编码的情况：

```Java
public void paySuccess(TradeStatusMsg tradeStatusMsg) {
  ...
  //订单状态为待支付时当支付成功更新为派单中
    if (ObjectUtil.equal(0, orders.getOrdersStatus())) {
       ...
    }

}
```

随着开发的深入这种代码会越来越多，比如在实现对订单进行关闭时代码会写成如下的形式：

```Java
if(订单状态==已完成){
  //运营人员在订单完成时取消订单
  //执行此场景下的业务逻辑
  //更新订单状态为派单中
  update(id,已关闭)
)
if(订单状态==服务中){
  //运营人员在服务中时取消订单
  //执行此场景下的业务逻辑
  //更新订单状态为已关闭
  update(id,已关闭)
)
...
```

以上代码存在问题如下：

**在业务代码中对订单状态进行硬编码如果有一天更改了业务逻辑就需要更改代码，不方便进行系统扩展和维护。**

**另外对订单状态的管理是散落在很多地方不方便对订单状态进行统一管理和维护。**

#### 2) **使用状态机解决问题**

针对以上问题如何解决呢？

我们可以使用状态机对订单状态进行统一管理。

**什么是状态机？**

上图在UML中叫状态图（又叫状态机图），UML是软件开发中的一种建模语言，用来辅助进行软件设计，常用的如：类图、对象、状态图、序列图等，注意状态机图并不是状态机，状态机是一种数学模型，应用在自动化控制、计算机科学、通信等很多领域，简单理解状态机就是对状态进行统一管理的数学模型。

我们画的状态图是状态机在计算机科学中的应用方法，还有状态机设计模式也是状态机在软件领域的应用方法。

**状态机设计模式是状态机在软件中的应用，状态机设计模式描述了一个对象在内部状态发生变化时如何改变其行为，将状态之间的变更定义为事件，将事件暴露出去，通过执行状态变更事件去更改状态，这是状态机设计模式的核心内容。**

**理解状态机设计模式需要理解四个要素：现态、事件、动作、次态。**

（参考地址：https://baike.baidu.com/item/%E7%8A%B6%E6%80%81%E6%9C%BA/6548513?fr=ge_ala ）

1、**现态**：是指当前所处的状态。

2、**次态**：条件满足后要迁往的新状态。

3、**事件**：当一个条件被满足，状态会由现态变为新的状态，事件发生会触发一个动作，或者执行一次状态的迁移。

4、**动作**：发生事件执行的动作，动作执行完毕后，可以迁移到新的状态，也可以仍旧保持原状态。动作不是必需的，当条件满足后，也可以不执行任何动作，直接迁移到新状态。



我们拿待支付状态到派单中状态举例：

![y0ccbxcoqoj4unxnlrzck4acnx5](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/y0ccbxcoqoj4unxnlrzck4acnx5.png)
> 图注：该图以流程图形式呈现了订单的状态流转过程，清晰展示了状态机的应用场景：用户发起“下单”操作后，订单从用户对应的初始状态进入“待支付”状态；当执行“用户支付”的事件后，订单状态便从“待支付”迁移至“派单中”，这恰好对应了文档中所举的待支付状态到派单中状态的状态迁移例子，直观呈现了订单在支付环节前后的状态变化逻辑。

**现态**：订单当前处于待支付状态那么现态为待支付。

**次态**：派单中。

**事件**：用户支付成功为事件，支付成功是条件，当条件满足进行状态迁移。

**动作**：将订单状态由待支付更改为派单中。



**使用状态机优化代码：**

使用状态机之后对代码进行以下优化。

支付成功更改订单状态的代码优化如下：

```Java
if(支付状态==支付成功){
    //调用状态机执行支付成功事件
    orderStateMachine.changeStatus(id,支付成功事件);
}
```



订单取消的代码优化如下：

```Java
orderStateMachine.changeStatus(id,订单完成时取消订单事件);

```

我们发现使用状态机的代码并没有对订单状态进行硬编码，只是指定了订单id和事件名称，执行changeStatus方法后自动更改订单的状态。



#### 3）**小结**

什么是状态机，它解决了什么问题？

- 统一管理订单状态
- 记录订单快照
- 查询优化（缓存）





### 1.2 实现订单状态机

#### 1.2.1 编写订单状态机

目标：

能够基于状态机组件实现订单状态机



##### 1) 添加状态机组件依赖

本项目基于状态机设计模式开发了状态机组件，代码在jzo2o-framework中，如果在订单管理服务中实现订单状态机需要添加状态机的依赖。

在jzo2o-orders-base工程的pom.xml中添加状态机组件的依赖

```Java
<dependency>
    <groupId>com.jzo2o</groupId>
    <artifactId>jzo2o-statemachine</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

添加完成刷新pom.xml

下边使用状态机组件管理订单状态。

##### 2）**订单状态枚举类**

阅读订单状态枚举类，它实现了StatusDefine 状态接口，不论是现态还是次态都需要实现状态接口。

定义每个枚举需要注意见名知意，比如：NO_PAY(0, "待支付", "NO_PAY")表示待支付状态。

订单状态枚举类如下：

```Java
package com.jzo2o.orders.base.enums;

import com.jzo2o.statemachine.core.StatusDefine;
import lombok.AllArgsConstructor;
import lombok.Getter;

/**
 * @author itcast
 */
@Getter
@AllArgsConstructor
public enum OrderStatusEnum implements StatusDefine {
    NO_PAY(0, "待支付", "NO_PAY"),
    DISPATCHING(100, "派单中", "DISPATCHING"),
    NO_SERVE(200, "待服务", "NO_SERVE"),
    SERVING(300, "服务中", "SERVING"),
    FINISHED(500, "已完成", "FINISHED"),
    CANCELED(600, "已取消", "CANCELED"),
    CLOSED(700, "已关闭", "CLOSED");

    private final Integer status;
    private final String desc;
    private final String code;

    /**
     * 根据状态值获得对应枚举
     *
     * @param status 状态
     * @return 状态对应枚举
     */
    public static OrderStatusEnum codeOf(Integer status) {
        for (OrderStatusEnum orderStatusEnum : values()) {
            if (orderStatusEnum.status.equals(status)) {
                return orderStatusEnum;
            }
        }
        return null;
    }
}
```

##### 3）**状态变更事件枚举类**

所有状态之间存在的变更都需要定义状态变更事件，它实现了StatusChangeEvent 状态变更事件接口，事件对应状态机四要素的事件

代码如下，重点看PAYED：

 PAYED(OrderStatusEnum.NO_PAY, OrderStatusEnum.DISPATCHING, "支付成功", "payed")表示由NO_PAY（未支付）状态变化为DISPATCHING（派单中）状态，事件名称为“支付成功”(payed)。

```Java
package com.jzo2o.orders.base.enums;

import com.jzo2o.statemachine.core.StatusChangeEvent;
import lombok.AllArgsConstructor;
import lombok.Getter;

/**
 * @author itcast
 */
@Getter
@AllArgsConstructor
public enum OrderStatusChangeEventEnum implements StatusChangeEvent {
    PAYED(OrderStatusEnum.NO_PAY, OrderStatusEnum.DISPATCHING, "支付成功", "payed"),
    DISPATCH(OrderStatusEnum.DISPATCHING, OrderStatusEnum.NO_SERVE, "接单/抢单成功", "dispatch"),
    START_SERVE(OrderStatusEnum.NO_SERVE, OrderStatusEnum.SERVING, "开始服务", "start_serve"),
    COMPLETE_SERVE(OrderStatusEnum.SERVING, OrderStatusEnum.FINISHED, "完成服务", "complete_serve"),
//    EVALUATE(OrderStatusEnum.NO_EVALUATION, OrderStatusEnum.FINISHED, "评价完成", "evaluate"),
    CANCEL(OrderStatusEnum.NO_PAY, OrderStatusEnum.CANCELED, "取消订单", "cancel"),
    SERVE_PROVIDER_CANCEL(OrderStatusEnum.NO_SERVE, OrderStatusEnum.DISPATCHING, "服务人员/机构取消订单", "serve_provider_cancel"),
    CLOSE_DISPATCHING_ORDER(OrderStatusEnum.DISPATCHING, OrderStatusEnum.CLOSED, "派单中订单关闭", "close_dispatching_order"),
    CLOSE_NO_SERVE_ORDER(OrderStatusEnum.NO_SERVE, OrderStatusEnum.CLOSED, "待服务订单关闭", "close_no_serve_order"),
    CLOSE_SERVING_ORDER(OrderStatusEnum.SERVING, OrderStatusEnum.CLOSED, "服务中订单关闭", "close_serving_order"),
//    CLOSE_NO_EVALUATION_ORDER(OrderStatusEnum.NO_EVALUATION, OrderStatusEnum.CLOSED, "待评价订单关闭", "close_no_evaluation_order"),
    CLOSE_FINISHED_ORDER(OrderStatusEnum.FINISHED, OrderStatusEnum.CLOSED, "已完成订单关闭", "close_finished_order");
    /**
     * 源状态
     */
    private final OrderStatusEnum sourceStatus;

    /**
     * 目标状态
     */
    private final OrderStatusEnum targetStatus;

    /**
     * 描述
     */
    private final String desc;

    /**
     * 代码
     */
    private final String code;
}
```

##### 4）**定义订单快照类**

快照是订单变化瞬间的状态及相关信息。

比如：001号订单创建成功此时记录它的快照信息（订单号、下单人、订单详细信息、订单状态等），当001号订单支付成功由待支付状态变化为派单中状态此时也会记录它的快照信息（订单号、下单人、支付状态、支付相关信息，订单状态等相关信息），由此可以看出**订单快照可以追溯订单的历史变化信息，只要状态发生变化便会记录快照。**

快照基础类型是StateMachineSnapshot，如果我们要实现订单快照则需要定义一个订单快照类OrderSnapshotDTO 去继承StateMachineSnapshot类型，代码如下：

```Java
package com.jzo2o.orders.base.model.dto;

import com.jzo2o.statemachine.core.StateMachineSnapshot;
import io.swagger.annotations.ApiModelProperty;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.math.BigDecimal;
import java.time.LocalDateTime;

/**
 * 订单快照
 *
 * @author itcast
 * @create 2023/8/19 10:30
 **/
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class OrderSnapshotDTO extends StateMachineSnapshot {
/**
 * 订单id
 */
private Long id;

/**
 * 订单所属人
 */
private Long userId;

/**
 * 服务类型id
 */
private Long serveTypeId;

/**
 * 服务类型名称
 */
private String serveTypeName;

/**
 * 服务项id
 */
private Long serveItemId;

/**
 * 服务项名称
 */
private String serveItemName;

/**
 * 服务项图片
 */
private String serveItemImg;

/**
 * 服务单位
 */
private Integer unit;

/**
 * 服务id
 */
private Long serveId;

/**
 * 订单状态，0：待支付，100：派单中，200：待服务，300：服务中，500：订单完成，600：订单取消，700已关闭
 */
private Integer ordersStatus;

/**
 * 支付状态，2：待支付，4：支付成功
 */
private Integer payStatus;

/**
 * 退款，0：发起退款，1：退款中，2：退款成功  3：退款失败
 */
private Integer refundStatus;

/**
 * 单价
 */
private BigDecimal price;

/**
 * 购买数量
 */
private Integer purNum;

/**
 * 订单总金额
 */
private BigDecimal totalAmount;

/**
 * 实际支付金额
 */
private BigDecimal realPayAmount;

/**
 * 优惠金额
 */
private BigDecimal discountAmount;

/**
 * 城市编码
 */
private String cityCode;

/**
 * 服务详细地址
 */
private String serveAddress;

/**
 * 联系人手机号
 */
private String contactsPhone;

/**
 * 联系人姓名
 */
private String contactsName;

/**
 * 服务开始时间
 */
private LocalDateTime serveStartTime;

/**
 * 经度
 */
private String lon;

/**
 * 纬度
 */
private String lat;

/**
 * 支付时间
 */
private LocalDateTime payTime;

/**
 * 评价时间
 */
private LocalDateTime evaluationTime;

/**
 * 订单创建时间
 */
private LocalDateTime createTime;

/**
 * 订单更新时间
 */
private LocalDateTime updateTime;

/**
 * 支付服务交易单号
 */
private Long tradingOrderNo;

/**
 * 支付服务退款单号
 */
private Long refundNo;



/**
 * 支付渠道【支付宝、微信、现金、免单挂账】
 */
private String tradingChannel;


/**
 * 三方流水,微信支付订单号或支付宝订单号
 */
private String thirdOrderId;

/**
 * 退款三方流水,微信支付订单号或支付宝订单号
 */
private String thirdRefundOrderId;


/**
 * 取消人id
 */
private Long cancellerId;

/**
 * 取消人名称
 */
private String cancelerName;

/**
 * 取消人类型
 */
private Integer cancellerType;

/**
 * 取消时间
 */
private LocalDateTime cancelTime;

/**
 * 取消原因
 */
private String cancelReason;

/**
 * 实际服务完成时间
 */
private LocalDateTime realServeEndTime;

/**
 * 评价状态
 */
private Integer evaluationStatus;

/**
 * 超时时间
 */
private LocalDateTime overTime;

@Override
public String getSnapshotId() {
    return String.valueOf(id);
}

@Override
public Integer getSnapshotStatus() {
    return ordersStatus;
}

@Override
public void setSnapshotId(String snapshotId) {
    this.id = Long.parseLong(snapshotId);
}

@Override
public void setSnapshotStatus(Integer snapshotStatus) {
    this.ordersStatus = snapshotStatus;
}

}
```



##### 5）定义事件变更动作类

当执行状态变更事件会伴随着执行具体的动作，此部分对应状态机四要素中的动作。

定义订单支付成功动作类，实现StatusChangeHandler接口，泛型中指定快照类型。

此动作是订单支付成功执行的动作。

动作类的bean名称为"状态机名称_事件名称"，例如下边的动作类bean的名称为order_payed，表示order状态机的payed事件。

```Java
package com.jzo2o.orders.base.handler;

import cn.hutool.db.DbRuntimeException;
import com.jzo2o.orders.base.enums.OrderPayStatusEnum;
import com.jzo2o.orders.base.enums.OrderStatusEnum;
import com.jzo2o.orders.base.model.dto.OrderSnapshotDTO;
import com.jzo2o.orders.base.model.dto.OrderUpdateStatusDTO;
import com.jzo2o.orders.base.service.IOrdersCommonService;
import com.jzo2o.statemachine.core.StateMachineSnapshot;
import com.jzo2o.statemachine.core.StatusChangeEvent;
import com.jzo2o.statemachine.core.StatusChangeHandler;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.time.LocalDateTime;

/**
 * 订单支付成功处理器
 *
 * @author itcast
 * @create 2023/8/17 18:08
 **/
@Slf4j
@Component("order_payed")
public class OrderPayedHandler implements StatusChangeHandler<OrderSnapshotDTO> {


    /**
     * 订单支付处理逻辑
     *
     * @param bizId   业务id
     * @param bizSnapshot 快照
     */
   @Override
    public void handler(String bizId, StatusChangeEvent statusChangeEventEnum, OrderSnapshotDTO bizSnapshot) {
        log.info("支付事件处理逻辑开始，订单号：{}", bizId);
        // 修改订单状态和支付状态
        OrderUpdateStatusDTO orderUpdateStatusDTO = OrderUpdateStatusDTO.builder().id(Long.valueOf(bizId))
                .originStatus(OrderStatusEnum.NO_PAY.getStatus())
                .targetStatus(OrderStatusEnum.DISPATCHING.getStatus())
                .payStatus(OrderPayStatusEnum.PAY_SUCCESS.getStatus())
                .payTime(LocalDateTime.now())
                .tradingOrderNo(bizSnapshot.getTradingOrderNo())
                .transactionId(bizSnapshot.getThirdOrderId())
                .tradingChannel(bizSnapshot.getTradingChannel())
                .build();
        int result = ordersService.updateStatus(orderUpdateStatusDTO);
        if (result <= 0) {
            throw new DbRuntimeException("支付事件处理失败");
        }
    }


}
```



##### 6）**定义订单状态机类**

AbstractStateMachine状态机抽象类是状态机的核心类，是具体的状态机要继承的抽象类，比如我们实现订单状态机就需要继承AbstractStateMachine抽象类。

<whiteboard token="Jvdywq1pahJuOEbNCoZcSN3Lndb"></whiteboard>

**成员变量**：

初始状态：设置初始状态，比如订单的初始状态为待支付。

状态机名称：返回的状态机的标识，比如订单状态机返回“order”作为订单状态机的名称。

**方法**：

返回状态机名称：返回状态机的名称

返回初始状态：返回初始状态

启动状态机：开始进行状态机管理，通常在新建实例时调用此方法，比如：新建一个订单调用此方法将订单状态设置为初始状态，传入参数：业务主键（如订单id）

变更状态：调用此方法更改状态

后处理方法：当状态变更后统一执行的逻辑。

下边定义订单状态机类继承此抽象类，代码如下：

```Java
package com.jzo2o.orders.base.config;


/**
 * 订单状态机
 *
 * @author itcast
 * @create 2023/8/4 11:20
 **/
@Component
public class OrderStateMachine extends AbstractStateMachine<OrderSnapshotDTO> {

     /**
     *  构建订单状态机对象
     * @param stateMachinePersister 状态机持久化service
     * @param bizSnapshotService 订单快照service
     * @param redisTemplate 
     */
    public OrderStateMachine(StateMachinePersister stateMachinePersister, BizSnapshotService bizSnapshotService, RedisTemplate redisTemplate) {
        super(stateMachinePersister, bizSnapshotService, redisTemplate);
    }

    /**
     * 设置状态机名称
     *
     * @return 状态机名称
     */
    @Override
    protected String getName() {
        return "order";
    }

    @Override
    protected void postProcessor(OrderSnapshotDTO orderSnapshotDTO) {

    }

    /**
     * 设置状态机初始状态
     *
     * @return 状态机初始状态
     */
    @Override
    protected OrderStatusEnum getInitState() {
        return OrderStatusEnum.NO_PAY;
    }

}
```

##### 7）**状态机表设计**

状态机使用MySQL对状态进行持久化，涉及到如下表：

**状态机持久化表**：

每个订单对应状态机表中的一条记录。

state_machine_name ：针对订单的状态机起个名称叫order，针对服务单的状态机可以起个名称为serve。

biz_id：存储订单id

state：记录该订单的当前状态

```Java
create table `jzo2o-orders`.state_persister
(
    id                 bigint auto_increment comment '主键'
        constraint `PRIMARY`
        primary key,
    state_machine_name varchar(255)                       null comment '状态机名称',
    biz_id             varchar(255)                       null comment '业务id',
    state              varchar(255)                       null comment '状态',
    create_time        datetime default CURRENT_TIMESTAMP null comment '创建时间',
    update_time        datetime default CURRENT_TIMESTAMP null on update CURRENT_TIMESTAMP comment '更新时间',
    constraint 唯一索引
        unique (state_machine_name, biz_id)
)
    comment '状态机持久化表' charset = utf8mb4;
```

**状态机快照表**：

一个订单在快照表有多条记录，每变一个状态会记录该状态下的快照信息（即订单相关的详细信息）便于查询订单变化的历史记录。

state_machine_name ：同上

biz_id ：同上

db_shard_id：暂时用不到

state：对应快照的状态

biz_data：快照信息（json格式），用在订单状态机就是记录订单相关的信息。

```Java
create table `jzo2o-orders`.biz_snapshot
(
    id                 bigint auto_increment comment '主键'
        constraint `PRIMARY`
        primary key,
    state_machine_name varchar(50)                        null comment '状态机名称',
    biz_id             varchar(50)                        null comment '业务id',
    db_shard_id        bigint                             null comment '分库键',
    state              varchar(50)                        null comment '状态代码',
    biz_data           varchar(5000)                      null comment '业务数据',
    create_time        datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    update_time        datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间'
)
    comment '业务数据快照' charset = utf8mb4;
```



##### 8）**小结**

如何实现一个订单状态机？





#### 1.2.2 测试订单状态机

目标：

测试状态机的方法实现订单状态变更。

能说出如何实现一个订单状态机。



##### 1) 加载订单状态机

在orders-base工程的AutoImportConfiguration类中配置导入订单状态机。

![ew3jbedn6o75c8xxqkgcskkenle](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ew3jbedn6o75c8xxqkgcskkenle.png)
> 图注：这张图片展示的是Java代码文件AutoImportConfiguration.java的编辑界面，对应文档中加载订单状态机的操作内容。文件的第14行用红框突出标注了核心代码，内容为@Import({OrderStateMachine.class})，这行代码的作用是将订单状态机导入到当前配置类中；该文件的位置在com.jzo2o.orders.base.config包下，类上还标注了@Configuration、@ComponentScan等用于项目配置的注解，和文档中“在orders-base工程的AutoImportConfiguration类中配置导入订单状态机”的要求完全对应。

##### 2）**测试启动状态机**

下边在jzo2o-orders-manager下编写测试代码。

调用OrderStateMachine的start()方法启动一个订单的状态机，启动状态机表示订单用状态机管理状态，启动状态机后会设置订单的初始状态。

```Java
package com.jzo2o.orders.manager.service;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.jzo2o.orders.base.config.OrderStateMachine;
import com.jzo2o.orders.base.model.domain.Orders;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import javax.annotation.Resource;
import java.util.List;

@SpringBootTest
@Slf4j
public class OrderStateMachineTest {

    @Resource
    private OrderStateMachine orderStateMachine;

    @Test
    public void test_start() {
        //启动状态机，指定订单id
        String start = orderStateMachine.start("101");
        log.info("返回初始状态:{}", start);
    }
}
```



执行测试方法，对101订单启动状态机管理，启动后101号订单的状态为NO_PAY待支付状态。

![gwovbpkcdobqlsxk9hgcrqkan4f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/gwovbpkcdobqlsxk9hgcrqkan4f.png)
> 图注：这是测试订单状态机启动功能的代码截图，内容为Java测试方法test_start，该方法用于启动订单状态机。方法内以101号订单为参数调用orderStateMachine的start方法以启动状态机，随后通过log.info打印该方法的返回结果，截图中明确显示返回的初始状态为“NO_PAY”，这与文档中描述的“101号订单测试状态机”的操作内容对应。

观察state_persister表有一条101号订单的状态持久化记录，每条订单对应state_persister表的一条记录。

观察biz_snapshot表有一条101号订单的快照信息，一条订单在biz_snapshot表对应多个条记录，每次订单状态变更都会产生一个快照。

注意：如果报错：“java.lang.IllegalStateException: 已存在状态，不可初始化”  ，说明101号订单的状态机已启动，可以更改订单号测试其它订单状态机启动。



##### 3）**测试状态变更方法**

根据状态变更事件定义可知，执行测试方法后101订单的状态由NO_PAY（待支付）变更为DISPATCHING（派单中）。

定义测试方法

```Java
@Test
public void test_changeStatus() {
    //状态变更
    orderStateMachine.changeStatus("101",OrderStatusChangeEventEnum.PAYED);
}
```

执行此方法，预期结果：

执行PAYED事件对应的动作，在动作类中打断点

![nfjcbb5etomxqvxhig1cgesmn5f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/nfjcbb5etomxqvxhig1cgesmn5f.png)
> 图注：图片展示的是OrderPayedHandler.java文件代码。该文件定义了OrderPayedHandler类，实现了StatusChangeHandler接口，用于处理订单支付相关状态变更事件。代码中包含订单支付处理逻辑，接收bizId、statusChangeEventEnum、bizSnapshot参数，通过OrderUpdateStatusDTO.builder()构建OrderUpdateStatusDTO对象，设置订单状态为NO_PAY和DISPATCHING。图片与上下文紧密相关，是对执行PAYED事件对应动作后，动作类中打断点时代码状态的展示。

如果动作方法执行成功state_persister表中101订单的状态变更为DISPATCHING。

biz_snapshot表多了一条101号订单的快照信息。

因为我们指定的订单号在订单表不存在所以此动作类执行失败，导致状态机变更状态失败。

我们可以暂时将动作类的代码屏蔽，再次测试。



下边对102号订单进行状态机管理

```Java
@Test
public void test_orderStatMmachine() {
    //启动状态机
    String start = orderStateMachine.start("102");
    //状态变更
    orderStateMachine.changeStatus("102",OrderStatusChangeEventEnum.PAYED);
}
```

执行测试：

1）观察状态机持久化表是否有102订单的状态记录。

2）观察快照表是否有102订单的记录。

3）观察控制台，状态变更处理器的执行信息是否输出，有输出说明处理器已执行



##### 4）阅读源码

进入jzo2o-framework下的jzo2o-statemachine工程，阅读AbstractStateMachine类的源码，通过阅读代码理解状态机组件的运行过程。

阅读启动状态机方法：

首先判断该订单是否启动状态，如果没有启动则向状态机表插入记录，否则抛出异常"已存在状态，不可初始化".

接下来保存订单快照。

```JSON
/**
 * 状态机初始化,不保存快照
 *
 * @param bizId 业务id
 * @return 初始化状态代码
 */
public String start(String bizId) {
    return start(null, bizId, initState, null);
}

/**
 * 启动状态机，并设置当前状态和保存业务快照，快照分库分表
 *
 * @param dbShardId    分库键
 * @param bizId        业务id
 * @param statusDefine 当前状态
 * @param bizSnapshot  快照
 * @return 当前状态代码
 */
public String start(Long dbShardId, String bizId, StatusDefine statusDefine, T bizSnapshot) {

    //1.初始化状态机状态
    String currentState = stateMachinePersister.getCurrentState(name, bizId);
    if (ObjectUtil.isEmpty(currentState)) {
        stateMachinePersister.init(name, bizId, statusDefine);
    } else {
        throw new IllegalStateException("已存在状态，不可初始化");
    }

    //2.保存业务快照
    if (bizSnapshot == null) {
        bizSnapshot = ReflectUtil.newInstance(getSnapshotClass());
    }
    //设置快照id
    bizSnapshot.setSnapshotId(bizId);
    //设置快照状态
    bizSnapshot.setSnapshotStatus(statusDefine.getStatus());
    //快照转json
    String bizSnapshotString = JSONUtil.toJsonStr(bizSnapshot);
    if (ObjectUtil.isNotEmpty(bizSnapshot)) {
        bizSnapshotService.save(dbShardId, name, bizId, statusDefine, bizSnapshotString);
    }
    //执行后处理方法
    postProcessor(bizSnapshot);

    return statusDefine.getCode();
}
```

阅读状态变更方法：

状态变更前会判断订单的当前状态是否和事件定义的源状态一致，如果不一致则说明当前订单的状态不能通过该事件去更新状态，此时将终止状态变更，否则将通过状态变更处理器去更新订单的状态。

```JSON
/**
 * 变更状态并保存快照，快照不进行分库
 *
 * @param bizId                 业务id
 * @param statusChangeEventEnum 状态变换事件
 */
public void changeStatus(String bizId, StatusChangeEvent statusChangeEventEnum) {
    changeStatus(null, bizId, statusChangeEventEnum, null);
}
/**
 * 变更状态并保存快照，快照分库分表
 *
 * @param dbShardId             分库键
 * @param bizId                 业务id
 * @param statusChangeEventEnum 状态变换事件
 * @param bizSnapshot           业务数据快照（json格式）
 */
public void changeStatus(Long dbShardId, String bizId, StatusChangeEvent statusChangeEventEnum, T bizSnapshot) {
    //1.查询当前状态
    String statusCode = getCurrentState(bizId);

    //2.校验起止状态是否与事件匹配
    if (ObjectUtil.isNotEmpty(statusChangeEventEnum.getSourceStatus()) && ObjectUtil.notEqual(statusChangeEventEnum.getSourceStatus().getCode(), statusCode)) {
        throw new CommonException(HTTP_INTERNAL_ERROR, "状态机起止状态与事件不匹配");
    }

    //3.获取状态处理程序bean
    //事件代码
    String eventCode = statusChangeEventEnum.getCode();
    StatusChangeHandler bean = null;
    try {
        bean = SpringUtil.getBean(name + "_" + eventCode, StatusChangeHandler.class);
    } catch (Exception e) {
        log.info("不存在‘{}’StatusChangeHandler", name + "_" + eventCode);
    }
    if (bizSnapshot == null) {
        bizSnapshot = ReflectUtil.newInstance(getSnapshotClass());
    }
    //设置快照id
    bizSnapshot.setSnapshotId(bizId);
    //设置目标状态
    bizSnapshot.setSnapshotStatus(statusChangeEventEnum.getTargetStatus().getStatus());
    if (ObjectUtil.isNotNull(bean)) {
        //4.执行状态变更
        bean.handler(bizId, statusChangeEventEnum, bizSnapshot);
    }

    //5.状态持久化
    stateMachinePersister.persist(name, bizId, statusChangeEventEnum.getTargetStatus());

    //6、存储快照
    if (ObjectUtil.isNotEmpty(bizSnapshot)) {
        //构建新的快照信息
        bizSnapshot = buildNewSnapshot(bizId, bizSnapshot, statusChangeEventEnum.getSourceStatus());
        String newBizSnapShotString = JSONUtil.toJsonStr(bizSnapshot);
        bizSnapshotService.save(dbShardId, name, bizId, statusChangeEventEnum.getTargetStatus(), newBizSnapShotString);
    }

    //7.清理快照缓存
    String key = "JZ_STATE_MACHINE:" + name + ":" + bizId;
    redisTemplate.delete(key);

    //执行后处理方法
    postProcessor(bizSnapshot);
}
```



##### 5）**小结**

本节测试了状态机的两个方法：

1. 启动状态机

启动状态机表示开始用状态机管理状态，并且设置初始状态。

1. 状态变更

使用状态机进行状态变更只需要指定状态变更事件。不用在业务代码中硬编码。



能说出如何实现一个订单状态机。



#### 1.2.3 使用订单状态机

目标：

下单时使用状态机

在支付成功时使用状态机



##### 1）**下单时启动状态机**

下单后创建一个新订单，使用状态机的启动方法表示用状态机对该订单的状态开始进行管理。

首先在OrdersCreateServiceImpl中注入状态机：

```Java
@Resource
private OrderStateMachine orderStateMachine;
```

代码如下：

```Java
@Transactional(rollbackFor = Exception.class)
public void add(Orders orders) {
    boolean save = this.save(orders);
    if (!save) {
        throw new DbRuntimeException("下单失败");
    }
   //构建快照对象
    OrderSnapshotDTO orderSnapshotDTO = BeanUtil.toBean(baseMapper.selectById(orders.getId()), OrderSnapshotDTO.class);
    //状态机启动
    orderStateMachine.start(null,String.valueOf(orders.getId()),orderSnapshotDTO);
}
```

**注意：状态机的操作方法放在业务方法中且和业务方法处于一个事务中。**



##### 2）**支付成功使用状态机**

支付成功通过状态机将订单状态由待支付更新派单中。

1. **定义状态变更动作类**

在动作类中更新订单的状态，在动作类中更新订单的状态要比在多处业务代码中对订单状态硬编码要强的多，因为可以在动作类中统一对订单状态进行管理。

除了更新订单状态以外还需要填充订单快照的相关信息，这里主要是支付相关的信息，包括：支付状态、支付时间、支付服务的交易单号、第三方支付的交易单号等。

代码如下：

注意动作类的bean名称规则是：状态机名称_事件名称

order_payed：状态机名称为order，中间用“\_”分隔，payed为状态变更事件名称（从OrderStatusChangeEventEnum中查找）

```Java
package com.jzo2o.orders.base.handler;

import cn.hutool.db.DbRuntimeException;
import com.jzo2o.orders.base.enums.OrderPayStatusEnum;
import com.jzo2o.orders.base.enums.OrderStatusEnum;
import com.jzo2o.orders.base.model.dto.OrderSnapshotDTO;
import com.jzo2o.orders.base.model.dto.OrderUpdateStatusDTO;
import com.jzo2o.orders.base.service.IOrdersCommonService;
import com.jzo2o.statemachine.core.StateMachineSnapshot;
import com.jzo2o.statemachine.core.StatusChangeEvent;
import com.jzo2o.statemachine.core.StatusChangeHandler;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.time.LocalDateTime;

/**
 * 订单支付成功处理器
 *
 * @author itcast
 * @create 2023/8/17 18:08
 **/
@Slf4j
@Component("order_payed")
public class OrderPayedHandler implements StatusChangeHandler<OrderSnapshotDTO> {
    @Resource
    private IOrdersCommonService ordersService;

    /**
     * 订单支付处理逻辑
     *
     * @param bizId   业务id
     * @param statusChangeEventEnum   状态变更事件
     * @param bizSnapshot 快照
     */
    @Override
    public void handler(String bizId, StatusChangeEvent statusChangeEventEnum, OrderSnapshotDTO bizSnapshot) {
        log.info("支付成功事件处理逻辑开始，订单号：{}", bizId);
        // 修改订单状态和支付状态
        OrderUpdateStatusDTO orderUpdateStatusDTO = OrderUpdateStatusDTO.builder().id(Long.valueOf(bizId))
                .originStatus(OrderStatusEnum.NO_PAY.getStatus())
                .targetStatus(OrderStatusEnum.DISPATCHING.getStatus())
                .payStatus(OrderPayStatusEnum.PAY_SUCCESS.getStatus())
                .payTime(bizSnapshot.getPayTime())
                .tradingOrderNo(bizSnapshot.getTradingOrderNo())
                .transactionId(bizSnapshot.getThirdOrderId())
                .tradingChannel(bizSnapshot.getTradingChannel())
                .build();
        int result = ordersCommonService.updateStatus(orderUpdateStatusDTO);
        if (result <= 0) {
            throw new DbRuntimeException("支付事件处理失败");
        }
    }


}
```



1. **在支付成功方法中使用状态机**

使用状态机执行支付成功状态变更。

```Java
@Transactional(rollbackFor = Exception.class)
public void paySuccess(TradeStatusMsg tradeStatusMsg) {
    //查询订单
    Orders orders = baseMapper.selectById(Long.parseLong(tradeStatusMsg.getProductOrderNo()));

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
            .build();
    orderStateMachine.changeStatus( String.valueOf(orders.getId()), OrderStatusChangeEventEnum.PAYED, orderSnapshotDTO);
}
```





##### 3）**测试**

测试流程：

创建一个订单

进行支付，支付成功

预期结果：

1. 创建订单

新创建一个订单，观察状态机持久化表和状态机快照表是否有订单相关的记录。

示例：

状态机持久化表：

```JSON
{
  "id": 1731873761497014273,
  "state_machine_name": "order",
  "biz_id": "2312050000000000058",
  "state": "NO_PAY",
  "create_time": "2023-12-05 11:11:47",
  "update_time": "2023-12-05 11:11:47"
}
```

状态机快照表：

```JSON
{
  "id": 1731873761516351488,
  "state_machine_name": "order",
  "biz_id": "2312050000000000058",
  "db_shard_id": 1,
  "state": "NO_PAY",
  "biz_data": "{\"ordersStatus\":0,\"cityCode\":\"010\",\"serveItemName\":\"日常保洁\",\"serveId\":1693815624114970626,\"discountAmount\":0,\"lon\":\"116.23189\",\"serveAddress\":\"北京市北京市昌平区北京市昌平区城北街道北京市昌平区政府信息公开办公室北京市昌平区人民政府\",\"contactsName\":\"苗先生\",\"serveItemId\":1685894105234755585,\"price\":1,\"serveItemImg\":\"https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/aa6489e5-cd92-42f0-837a-952c99653b8b.png\",\"realPayAmount\":1,\"serveStartTime\":1701757800000,\"id\":2312050000000000058,\"lat\":\"40.221\",\"serveTypeId\":1678649931106705409,\"updateTime\":1701745907000,\"userId\":1716346406098296832,\"totalAmount\":1,\"unit\":1,\"contactsPhone\":\"13333333333\",\"createTime\":1701745907000,\"payStatus\":2,\"purNum\":1,\"serveTypeName\":\"保洁清洗\"}",
  "create_time": "2023-12-05 11:11:47",
  "update_time": "2023-12-05 11:11:47"
}
```

订单表：

```JSON
{
  "id": 2312050000000000058,
  "user_id": 1716346406098296832,
  "serve_type_id": 1678649931106705409,
  "serve_type_name": "保洁清洗",
  "serve_item_id": 1685894105234755585,
  "serve_item_name": "日常保洁",
  "serve_item_img": "https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/aa6489e5-cd92-42f0-837a-952c99653b8b.png",
  "unit": 1,
  "serve_id": 1693815624114970626,
  "orders_status": 0,
  "pay_status": 2,
  "refund_status": null,
  "price": 1.00,
  "pur_num": 1,
  "total_amount": 1.00,
  "real_pay_amount": 1.00,
  "discount_amount": 0.00,
  "city_code": "010",
  "serve_address": "北京市北京市昌平区北京市昌平区城北街道北京市昌平区政府信息公开办公室北京市昌平区人民政府",
  "contacts_phone": "13333333333",
  "contacts_name": "苗先生",
  "serve_start_time": "2023-12-05 14:30:00",
  "lon": 116.23189,
  "lat": 40.221,
  "pay_time": null,
  "evaluation_time": null,
  "evaluation_status": 0,
  "trading_order_no": null,
  "transaction_id": null,
  "refund_no": null,
  "refund_id": null,
  "trading_channel": null,
  "display": 1,
  "sort_by": 1701757800058,
  "real_serve_end_time": null,
  "create_time": "2023-12-05 11:11:47",
  "update_time": "2023-12-05 11:11:47"
}
```



1. 支付成功

观察该订单的状态是否变为派单中。

观察状态机持久化表中该订单状态是否为派单中。

观察状态机快照表中该订单的快照信息是否正确。

示例：

状态机持久化表

```JSON
{
  "id": 1731873761497014273,
  "state_machine_name": "order",
  "biz_id": "2312050000000000058",
  "state": "DISPATCHING",
  "create_time": "2023-12-05 11:11:47",
  "update_time": "2023-12-05 11:14:31"
}
```

状态机快照表：

```JSON
{
  "id": 1731873761516351488,
  "state_machine_name": "order",
  "biz_id": "2312050000000000058",
  "db_shard_id": 1,
  "state": "NO_PAY",
  "biz_data": "{\"ordersStatus\":0,\"cityCode\":\"010\",\"serveItemName\":\"日常保洁\",\"serveId\":1693815624114970626,\"discountAmount\":0,\"lon\":\"116.23189\",\"serveAddress\":\"北京市北京市昌平区北京市昌平区城北街道北京市昌平区政府信息公开办公室北京市昌平区人民政府\",\"contactsName\":\"苗先生\",\"serveItemId\":1685894105234755585,\"price\":1,\"serveItemImg\":\"https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/aa6489e5-cd92-42f0-837a-952c99653b8b.png\",\"realPayAmount\":1,\"serveStartTime\":1701757800000,\"id\":2312050000000000058,\"lat\":\"40.221\",\"serveTypeId\":1678649931106705409,\"updateTime\":1701745907000,\"userId\":1716346406098296832,\"totalAmount\":1,\"unit\":1,\"contactsPhone\":\"13333333333\",\"createTime\":1701745907000,\"payStatus\":2,\"purNum\":1,\"serveTypeName\":\"保洁清洗\"}",
  "create_time": "2023-12-05 11:11:47",
  "update_time": "2023-12-05 11:11:47"
},
{
  "id": 1731874449264766976,
  "state_machine_name": "order",
  "biz_id": "2312050000000000058",
  "db_shard_id": 1,
  "state": "DISPATCHING",
  "biz_data": "{\"tradingChannel\":\"WECHAT_PAY\",\"ordersStatus\":100,\"payTime\":1701746020020,\"cityCode\":\"010\",\"serveItemName\":\"日常保洁\",\"serveId\":1693815624114970626,\"discountAmount\":0,\"lon\":\"116.23189\",\"serveAddress\":\"北京市北京市昌平区北京市昌平区城北街道北京市昌平区政府信息公开办公室北京市昌平区人民政府\",\"contactsName\":\"苗先生\",\"serveItemId\":1685894105234755585,\"price\":1,\"serveItemImg\":\"https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/aa6489e5-cd92-42f0-837a-952c99653b8b.png\",\"thirdOrderId\":\"4200002069202312059443886005\",\"realPayAmount\":1,\"serveStartTime\":1701757800000,\"id\":2312050000000000058,\"lat\":\"40.221\",\"serveTypeId\":1678649931106705409,\"updateTime\":1701745907000,\"userId\":1716346406098296832,\"tradingOrderNo\":1731874360980430850,\"totalAmount\":1,\"unit\":1,\"contactsPhone\":\"13333333333\",\"createTime\":1701745907000,\"payStatus\":2,\"purNum\":1,\"serveTypeName\":\"保洁清洗\"}",
  "create_time": "2023-12-05 11:14:31",
  "update_time": "2023-12-05 11:14:31"
}
```

订单的状态

```JSON
{
  "id": 2312050000000000058,
  "user_id": 1716346406098296832,
  "serve_type_id": 1678649931106705409,
  "serve_type_name": "保洁清洗",
  "serve_item_id": 1685894105234755585,
  "serve_item_name": "日常保洁",
  "serve_item_img": "https://yjy-xzbjzfw-oss.oss-cn-hangzhou.aliyuncs.com/aa6489e5-cd92-42f0-837a-952c99653b8b.png",
  "unit": 1,
  "serve_id": 1693815624114970626,
  "orders_status": 100,
  "pay_status": 4,
  "refund_status": null,
  "price": 1.00,
  "pur_num": 1,
  "total_amount": 1.00,
  "real_pay_amount": 1.00,
  "discount_amount": 0.00,
  "city_code": "010",
  "serve_address": "北京市北京市昌平区北京市昌平区城北街道北京市昌平区政府信息公开办公室北京市昌平区人民政府",
  "contacts_phone": "13333333333",
  "contacts_name": "苗先生",
  "serve_start_time": "2023-12-05 14:30:00",
  "lon": 116.23189,
  "lat": 40.221,
  "pay_time": "2023-12-05 11:13:40",
  "evaluation_time": null,
  "evaluation_status": 0,
  "trading_order_no": 1731874360980430850,
  "transaction_id": "4200002069202312059443886005",
  "refund_no": null,
  "refund_id": null,
  "trading_channel": "WECHAT_PAY",
  "display": 1,
  "sort_by": 1701757800058,
  "real_serve_end_time": null,
  "create_time": "2023-12-05 11:11:47",
  "update_time": "2023-12-05 11:14:31"
}
```



##### 4）小结

项目使用状态机实现什么功能？为什么这样做？



#### 1.2.4 取消订单使用状态机

##### 1）修改策略类

下边在取消订单策略类中使用状态机修改订单状态。

```Java
package com.jzo2o.orders.manager.strategy.impl;

import cn.hutool.core.bean.BeanUtil;
import com.jzo2o.orders.base.config.OrderStateMachine;
import com.jzo2o.orders.base.enums.OrderStatusChangeEventEnum;
import com.jzo2o.orders.base.enums.OrderStatusEnum;
import com.jzo2o.orders.base.model.domain.OrdersCanceled;
import com.jzo2o.orders.base.model.dto.OrderSnapshotDTO;
import com.jzo2o.orders.base.model.dto.OrderUpdateStatusDTO;
import com.jzo2o.orders.base.service.IOrdersCommonService;
import com.jzo2o.orders.manager.model.dto.OrderCancelDTO;
import com.jzo2o.orders.manager.service.IOrdersCanceledService;
import com.jzo2o.orders.manager.strategy.OrderCancelStrategy;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;
import java.time.LocalDateTime;

/**
 * @author Mr.M
 * @version 1.0
 * @description c端用户取消待支付订单策略类
 * @date 2024/11/6 17:38
 */
@Component("1:NO_PAY")
public class CommonUserNoPayOrderCancelStrategy implements OrderCancelStrategy {
    @Resource
    private IOrdersCommonService ordersCommonService;
    @Resource
    private IOrdersCanceledService ordersCanceledService;

    @Resource
    private OrderStateMachine orderStateMachine;
    @Override
    @Transactional
    public void cancel(OrderCancelDTO orderCancelDTO) {
//        //保存取消订单记录
//        OrdersCanceled ordersCanceled = BeanUtil.toBean(orderCancelDTO, OrdersCanceled.class);
//        ordersCanceled.setCancellerId(orderCancelDTO.getCurrentUserId());
//        ordersCanceled.setCancelerName(orderCancelDTO.getCurrentUserName());
//        ordersCanceled.setCancellerType(orderCancelDTO.getCurrentUserType());
//        ordersCanceled.setCancelTime(LocalDateTime.now());
//        ordersCanceledService.save(ordersCanceled);
//        //更新订单状态
//        OrderUpdateStatusDTO orderUpdateStatusDTO = new OrderUpdateStatusDTO();
//        //订单id
//        orderUpdateStatusDTO.setId(orderCancelDTO.getId());
//        //原始订单状态
//        orderUpdateStatusDTO.setOriginStatus(OrderStatusEnum.NO_PAY.getStatus());
//        //目标状态
//        orderUpdateStatusDTO.setTargetStatus(OrderStatusEnum.CANCELED.getStatus());
//        Integer integer = ordersCommonService.updateStatus(orderUpdateStatusDTO);
//        if (integer == 0) {
//            throw new RuntimeException("订单取消执行失败");
//        }
        //2.构建订单快照更新模型
        OrderSnapshotDTO orderSnapshotDTO = OrderSnapshotDTO.builder()
                .cancellerId(orderCancelDTO.getCurrentUserId())
                .cancelerName(orderCancelDTO.getCurrentUserName())
                .cancellerType(orderCancelDTO.getCurrentUserType())
                .cancelReason(orderCancelDTO.getCancelReason())
                .cancelTime(LocalDateTime.now())
                .build();

        //3.保存订单取消记录
        OrdersCanceled ordersCanceled = BeanUtil.toBean(orderSnapshotDTO, OrdersCanceled.class);
        ordersCanceled.setId(orderCancelDTO.getId());
        ordersCanceledService.save(ordersCanceled);

        //4.订单状态变更
        orderStateMachine.changeStatus(orderCancelDTO.getUserId(), orderCancelDTO.getId().toString(), OrderStatusChangeEventEnum.CANCEL, orderSnapshotDTO);

    }
}
```

##### 2）复制策略类

其它策略类实现方法类似，在通过状态机更新订单状态时需要指定事件名称。

拷贝课程资料下的“取消订单策略类”下的策略类到 jzo2o-orders-mamanger工程下的com.jzo2o.orders.manager.strategy.impl包下，阅读取消订单使用的策略类，找到使用状态机修改订单状态的代码。

![swhibu4c1o4bb0x3eqicjdzmnjb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/swhibu4c1o4bb0x3eqicjdzmnjb.png)
> 图注：图片展示了jzo2o订单模块下的manager和strategy文件夹结构。manager文件夹下有config、controller、handler、listener、model、properties、service等子文件夹。strategy文件夹下有impl子文件夹，其中包含多个取消订单策略类，如CommonUserDispatchingOrderCancelStrategy、CommonUserNoPayOrderCancelStrategy等，这些策略类与上下文提到的取消订单使用状态机修改订单状态的代码相关。



##### 2）测试

测试流程：

下单

支付

取消订单



预期结果：

下单成功：

检查状态机state_persister表和biz_snapshot表是否成功插入记录。

已支付状态取消订单：

观察state_persister表中状态是否变更为CANCELED（已取消）

观察biz_snapshot表中是否多了条快照信息。

退款成功。



## 2 分库分表

### 2.1 什么是分库分表

#### 1）**当前遇到的问题**

随着订单数据的增加，当MySQL单表存储数据达到一定量时其存储及查询性能会下降，在阿里的《Java 开发手册》中提到MySQL单表行数超过 500 万行或者单表容量超过 2GB时建议进行分库分表，分库分表可以简单理解为原来一个表存储数据现在改为通过多个数据库及多个表去存储，这就相当于原来一台服务器提供服务现在改成多台服务器组成集群共同提供服务，从而增加了服务能力。

这里说的500 万行或单表容量超过 2GB并不是定律，只是根据生产经验而言，为什么MySQL单表当达到一定数量时性能会下降呢？我们知道为了提高表的查询性能会增加索引，MySQL在使用索引时会将索引加入内存，如果数据量非常大内存肯定装不下，此时就会从磁盘去查询索引就会产生很多的磁盘IO，从而影响性能，这些和表的设计及服务器的硬件配置都有关，所以如果当表的数据量达到一定程度并且还在不断的增加就需要考虑进行分库分表了。

#### 2）**什么是分库分表**

下边通过一个例子来说明什么是分库分表。

下边是一个电商系统的数据库，涉及了店铺、商品的相关业务。

![ph91bcooko1pqkxiquxcapkenxd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ph91bcooko1pqkxiquxcapkenxd.png)
> 图注：这张图展示了电商数据库拆分后的逻辑结构，核心围绕“SELLER_DB”相关的分库分表思路，拆分为地理区域、店铺信息、商品信息三个数据表。地理区域表包含地理区域编码、名称、级别、上级编码等字段；店铺信息表设有店铺ID、名称、信誉、所在地字段，且有箭头指向商品信息表，说明二者存在关联；商品信息表则包含所属店铺（带有外键标识）、商品名称、规格、产地、价格、图片、描述等字段，通过拆分数据库和数据表，缩小单一数据库、数据表的数据量，以此解决数据库数据量过大导致的性能降低问题。



随着公司业务快速发展，数据库中的数据量猛增，访问性能也变慢了，如何优化呢？

我们可以把数据分散在不同的数据库中，使得单一数据库的数据量变小来缓解单一数据库的性能问题，从而达到提升数据库性能的目的，如下图：将电商数据库拆分为若干独立的数据库，并且对于大表也拆分为若干小表，通过这种数据库 拆分的方法来解决数据库的性能问题

![xjbqbwhluo2fhfxhmizczu4qntd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xjbqbwhluo2fhfxhmizczu4qntd.png)
> 图注：图片展示了电商数据库拆分的示例。左侧是一个标有“电商数据库”的黄色圆柱形图标，右侧有四个黄色圆柱形图标，分别标有“用户库”“商品库”“订单库”和“...”，中间有一个蓝色箭头指向右侧图标。这与上下文介绍的将电商数据库拆分为若干独立数据库，以及对大表拆分若干小表，通过数据库拆分解决性能问题的内容相契合，直观呈现了拆分后的数据库结构。



分库分表就是为了解决由于数据量过大而导致数据库性能降低的问题，将原来独立的数据库拆分成若干数据库组成 

，将数据大表拆分成若干数据表组成，使得单一数据库、单一数据表的数据量变小，从而达到提升数据库性能的目 

的。

#### 3）**分库分表的四种方式**

分库分表包括分库和分表两个部分，在生产中通常包括：垂直分库、水平分库、垂直分表、水平分表四种方式。

##### **垂直分表**

下图是商品查询列表：

![yyjzbhg2kotwdnx902ucjm1pnhb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/yyjzbhg2kotwdnx902ucjm1pnhb.png)
> 图注：这张图片展示的是商品查询列表，呈现了四个线上商品信息，每个条目包含商品图、名称、价格、运费、店铺名称与发货地。其中第一款商品是24瓶装的Vita Coco唯他可可椰子水饮料，售价298元且包邮；第二款是24罐装的绿力冬瓜茶饮料，售价88.8元，运费18元；第三款是24瓶装的上海风味盐汽水，售价26.8元，运费8元；第四款是16瓶装的沙棘汁饮料，售价47.88元且包邮，发货地为山西吕梁。这些商品对应文档中提及的商品名称、图片、价格等访问频次较高的商品基本信息。



用户在浏览商品列表时，只有对某商品感兴趣时才会查看该商品的详细描述。因此，商品信息中**商品描述**字段访问 

频次较低，且该字段存储占用空间较大，访问单个数据IO时间较长；商品信息中商品名称、商品图片、商品价格等 其他字段数据访问频次较高。 

由于这两种数据的特性不一样，因此考虑将商品信息表拆分如下： 

将访问频次低的商品描述信息单独存放在一张表中，访问频次较高的商品基本信息单独放在一张表中。

![mci4brhdpox6h4xlkl3ckoqfnsd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mci4brhdpox6h4xlkl3ckoqfnsd.png)
> 图注：图片展示了商品信息的分库分表示例。上方是商品信息表，包含id、所属店铺、商品名称、规格、产地、商品价格、商品图片、商品描述等字段。下方是商品描述表，仅包含id、商品ID、商品描述字段。箭头从商品信息表指向商品描述表，表明将访问频次低的商品描述信息单独存放在一张表中。该图与上下文紧密相关，直观呈现了将商品信息表拆分的分库分表方式。



**垂直分表是将一个表按照字段分成多表，每个表存储其中一部分字段，比如按冷热字段进行拆分。**

垂直分表带来的好处是：充分发挥热门数据的操作效率，商品信息的操作的高效率不会被商品描述的低效率所拖累。 

通常我们按以下原则进行垂直拆分: 

1. 把不常用的字段单独放在一张表; 
2. 把text，blob等大字段拆分出来放在附表中; 
3. 经常组合查询的列放在一张表中; 



##### **垂直分库**

通过垂直分表性能得到了一定程度的提升，但是还没有达到要求，并且磁盘空间也快不够了，因为数据还是始终限制在一台服务器，库内垂直分表只解决了单一表数据量过大的问题，但没有将表分布到不同的服务器上，因此每个 表还是竞争同一个物理机的CPU、内存、网络IO、磁盘。 

经过思考，他把原有的SELLER_DB(卖家库)，分为了PRODUCT_DB(商品库)和STORE_DB(店铺库)，并把这两个库分 散到不同服务器，如下图： 

![tepybh0oeoid8xx1ok5cjvaen2f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/tepybh0oeoid8xx1ok5cjvaen2f.png)
> 图注：图片展示了分库分表后SELLER_DB、PRODUCT_DB、STORE_DB三个数据库的结构。SELLER_DB包含店铺信息、商品信息、商品描述、地理区域；PRODUCT_DB包含商品信息、商品描述；STORE_DB包含店铺信息、地理区域。箭头分别指向三个数据库，表明原有SELLER_DB被拆分为这三个数据库。此图与上下文紧密相关，直观呈现了垂直分库后数据库结构的变化，帮助理解商品信息与店铺信息的独立存储。



由于**商品信息**与**商品描述**业务耦合度较高，因此一起被存放在PRODUCT_DB(商品库)；而**店铺信息**相对独立，因此 单独被存放在STORE_DB(店铺库)。

**垂直分库**是指按照业务将表进行分类，分布到不同的数据库上面，每个库可以放在不同的服务器上，它的核心理念 是专库专用，微服务架构下通常会对数据库进行垂直分为，不同业务数据放在单独的数据库中，比如：客户信息数据库、订单数据库等。

它带来的提升是： 

1、解决业务层面的耦合，业务清晰 

2、能对不同业务的数据进行分级管理、维护、监控、扩展等 

3、高并发场景下，垂直分库一定程度的提升IO、降低单机硬件资源的瓶颈。

垂直分库通过将表按业务分类，然后分布在不同数据库，并且可以将这些数据库部署在不同服务器上，从而达到多 个服务器共同分摊压力的效果，但是依然没有解决单表数据量过大的问题。



##### **水平分库**

经过**垂直分库**后，数据库性能问题得到一定程度的解决，但是随着业务量的增长，PRODUCT_DB(商品库)单库存储数据已经超出预估。粗略估计，目前有8w店铺，每个店铺平均150个不同规格的商品，再算上增长，那商品数量得 往1500w+上预估，并且PRODUCT_DB(商品库)属于访问非常频繁的资源，单台服务器已经无法支撑。此时该如何 优化？

再次分库？但是从业务角度分析，目前情况已经无法再次垂直分库。 

尝试水平分库，将店铺ID为单数的和店铺ID为双数的商品信息分别放在两个库中。

![gb0vbmpkxooiegxz8b3cxzfuncc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/gb0vbmpkxooiegxz8b3cxzfuncc.png)
> 图注：图片展示了商品信息分库分表的结构。左侧是PRODUCT_DB，包含商品信息和商品描述两张表，商品信息表有商品ID、所属店铺ID等字段。右侧是PRODUCT_DB1和PRODUCT_DB2，商品信息表和商品描述表分别对应，商品ID%2为判断依据，商品ID为双数时映射至PRODUCT_DB1，为单数时映射至PRODUCT_DB2。该图与上下文介绍的水平分库方式相呼应，直观呈现了商品ID按规则拆分到不同数据库的结构。

 

也就是说，要操作某条数据，先分析这条数据所属的店铺ID。如果店铺ID为双数，将此操作映射至 RRODUCT_DB1(商品库1)；如果店铺ID为单数，将操作映射至RRODUCT_DB2(商品库2)。

**水平分库**是把同一个表的数据按一定规则拆到不同的数据库中，每个库可以放在不同的服务器上，比如：单数订单在db_orders_0数据库，偶数订单在db_orders_1数据库。 

它带来的提升是： 

1、解决了单库大数据，高并发的性能瓶颈。 

2、提高了系统的稳定性及可用性。 

当一个应用难以再细粒度的垂直切分，或切分后数据量行数巨大，存在单库读写、存储性能瓶颈，这时候就需要进行**水平分库**了，经过水平切分的优化，往往能解决单库存储量及性能瓶颈。但由于同一个表被分配在不同的数据库，需要额外进行数据操作的路由工作，因此大大提升了系统复杂度。 



##### **水平分表**

按照水平分库的思路把PRODUCT_DB_X(商品库)内的表也可以进行水平拆分，其目的也是为解决单表数据量大 的问题，如下图：

![g0fxbrxdtoasglxvigicx961n2d](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/g0fxbrxdtoasglxvigicx961n2d.png)
> 图注：图片展示了PRODUCT_DB数据库中商品信息及商品描述的分库分表情况。商品信息分为商品信息1和商品信息2，商品描述也分为商品描述1和商品描述2。商品ID为双数时，操作映射至商品信息1和商品描述1；商品ID为单数时，映射至商品信息2和商品描述2。此图与上文提到的水平分表思路类似，通过商品ID的奇偶性将商品信息及商品描述分别映射到不同表，以解决单表数据量大的问题。



与水平分库的思路类似，不过这次操作的目标是表，商品信息及商品描述被分成了两套表。如果商品ID为双数，将 此操作映射至商品信息1表；如果商品ID为单数，将操作映射至商品信息2表。此操作要访问表名称为 **商品信息[商品ID%2 + 1] 。**

**水平分表**是在同一个数据库内，把同一个表的数据按一定规则拆到多个表中，比如：0到500万的订单在orders_0数据、500万到1000万的订单在orders_1数据表。水平分表优化了单一表数据量过大而产生的性能问题。

一般来说，在系统设计阶段就应该根据业务耦合松紧来确定垂直分库，垂直分表方案，在数据量及访问压力不是特 别大的情况，首先考虑缓存、读写分离、索引技术等方案。若数据量极大，且持续增长，再考虑水平分库水平分表方案。 

#### 4）小结

项目为什么进行分库分表？



分库分表有哪些形式？

### 2.2 分库分表带来的问题

#### 1）**事务一致性问题**

由于分库分表把数据分布在不同库甚至不同服务器，不可避免会带来**分布式事务**问题。

#### 2）**跨节点关联查询**

在没有分库前，我们检索商品时可以通过以下SQL对店铺信息进行关联查询： 

```Java
SELECT p.*,r.[地理区域名称],s.[店铺名称],s.[信誉] 
FROM [商品信息] p 
LEFT JOIN [地理区域] r ON p.[产地] = r.[地理区域编码] 
LEFT JOIN [店铺信息] s ON s.id = p.[所属店铺] 
WHERE...ORDER BY...LIMIT...
```

但垂直分库后**[商品信息]**和**[店铺信息]**不在一个数据库，甚至不在一台服务器，无法进行关联查询。 

可将原关联查询分为两次查询，第一次查询的结果集中找出关联数据id，然后根据id发起第二次请求得到关联数据，最后将获得到的数据进行拼装。 

#### 3) **跨节点分页、排序函数**

跨节点多库进行查询时，limit分页、order by排序等问题，就变得比较复杂了。需要先在不同的分片节点中将数据进行排序并返回，然后将不同分片返回的结果集进行汇总和再次排序。 

如，进行水平分库后的商品库，按ID倒序排序分页，取第一页:

![](https://feishu.cn/file/IRjybC2uso1lmQxyObXcaUOzngg)

以上流程是取第一页的数据，性能影响不大，但由于商品信息分布在各数据库的数据可能是随机的，如果是取第 N页，需要将所有节点前N页数据都取出来合并，再进行整体的排序，操作效率可想而知。所以请求页数越大，系 统的性能也会越差。 

在使用Max、Min、Sum、Count之类的函数进行计算的时候，与排序分页同理，也需要先在每个分片上执行相应 的函数，然后将各个分片的结果集进行汇总和再次计算，最终将结果返回。



#### 4）**主键避重**

在分库分表环境中，由于表中数据同时存在不同数据库中，主键值平时使用的自增长将无用武之地，某个分区数据库生成的ID无法保证全局唯一。因此需要单独设计全局主键，以避免跨库主键重复问题。

![l0cvbg7u3oirmwxqhwccdv32nbe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/l0cvbg7u3oirmwxqhwccdv32nbe.png)
> 图注：图片展示了分库分表环境下商品信息表的结构。上方是DB1的商品信息表，包含ID、数据项1等字段，ID有1等值；下方是DB2的商品信息表，同样包含ID、数据项1等字段，ID也有1等值。图片中ID冲突，无法保证唯一，用蓝色箭头和文字标注突出显示，直观呈现了在分库分表中主键避重问题，即主键值在不同数据库中可能重复，无法保证全局唯一，需单独设计全局主键以避免跨库主键重复问题。

#### 5）**公共表**

实际的应用场景中，参数表、数据字典表等都是数据量较小，变动少，而且属于高频联合查询的依赖表。例子中**地理区域表**也属于此类型。 

可以将这类表在每个数据库都保存一份，所有对公共表的更新操作都同时发送到所有分库执行。 

 

### 2.2 分库分表解决方案

由于分库分表之后，数据被分散在不同的数据库、服务器。因此，对数据的操作也就无法通过常规方式完成，并且 它还带来了一系列的问题。好在，这些问题不是所有都需要我们在应用层面上解决，市面上有很多中间件可供我们 选择。

#### 1）技术方案

AI: java中分库分表解决方案有哪些

> 在Java应用中，当单个数据库的表数据量达到一定规模后，可能会遇到性能瓶颈，如查询速度慢、锁竞争等问题。这时，通常会采用分库分表（Sharding）的策略来解决这些问题。分库分表可以将大量数据分散到多个数据库或多个表中，以提高系统的可扩展性和性能。
> 
> 以下是几种常见的Java中的分库分表解决方案：
> 
> 1. 手工编码：
> 
>    - 直接在代码层面实现逻辑，根据一定的规则（如用户ID）进行计算，决定数据应该存储在哪个数据库或者表中。
>    - 缺点是手工编码较复杂，需要在各个地方维护分库分表的逻辑，一旦规则改变，需要修改的地方较多。
> 2. 使用中间件/代理层：
> 
>    - 这类方案通常会在应用程序与数据库之间提供一个代理层，通过代理来处理分库分表的逻辑。
>    - MyCat：是一个开源的数据库中间件，支持SQL解析和路由功能，可以实现透明化的分库分表。
>    - ShardingSphere（原Apache ShardingSphere）：这是一个分布式数据库框架，提供了Java程序中的分库分表解决方案，支持多种分片策略，并且可以作为JDBC客户端使用，也可以作为数据库代理运行。
> 3. ORM框架支持：
> 
>    - MyBatis + MyBatis-Spring/Sharding-JDBC：结合使用MyBatis框架和Spring框架，利用Sharding-JDBC来实现分库分表的功能。
>    - Hibernate + Ehcache/Infinispan：虽然Hibernate本身并不直接支持分库分表，但可以通过配置多数据源以及缓存策略来间接实现分库分表的效果。
> 4. 云服务提供的解决方案：
> 
>    - 如果使用的是云服务，如阿里云、腾讯云等，它们往往提供了一整套的解决方案，包括但不限于读写分离、分库分表等功能。
> 5. 数据库自带的分区功能：
> 
>    - 如果使用的数据库本身支持分区功能（如MySQL的分区表），可以通过这种方式来实现将一个表中的数据分到多个数据文件中，但在一定程度上也能缓解性能压力。



在实际项目中，选择分库分表方案时，通常会考虑到技术成熟度、社区支持、易用性、性能等因素。以下是一些在实际应用中较为常用的分库分表解决方案：

1. ShardingSphere（Apache ShardingSphere）：

   - 随着ShardingSphere成为Apache的顶级项目，其稳定性和功能得到了广泛的认可。它支持多种分片算法，可以作为JDBC客户端直接集成到应用程序中，也可以作为数据库代理部署，提供透明化的分库分表能力。
   - ShardingSphere的优势在于它的灵活性和扩展性，支持水平分片、垂直分片等多种模式，并且集成了读写分离、数据加密等功能。
2. MyCat：

   - MyCat是一个轻量级的数据库中间件，适用于那些希望快速实现分库分表的企业。尽管它的功能相比ShardingSphere来说可能略显简单，但它依然提供了足够的灵活性来满足大多数应用场景的需求。
   - MyCat适合于已经有稳定架构的企业，希望在不改变太多现有代码的基础上引入分库分表机制。
3. 手工编码实现：

   - 在一些特定场景下，企业可能会选择自己实现分库分表逻辑。这种做法的好处是可以完全控制整个流程，但缺点是需要投入大量的时间和精力去维护这些逻辑，而且容易出现各种边界情况的问题。
4. 云服务提供的解决方案：

   - 对于使用云服务的企业来说，云平台通常会提供一整套的解决方案，包括自动化的分库分表、负载均衡等。例如，阿里云的DRDS就是一个典型的代表，它可以帮助用户轻松地管理和扩展数据库。
   - 云平台的优势在于它们通常会有专业的团队来维护和更新，减轻了企业的运维负担。

总的来说，ShardingSphere由于其强大的功能和灵活性，在很多场景下成为了首选。而MyCat则因其轻量级的特点，在一些中小企业或者对性能要求不是很高的项目中也有一定的使用率。当然，具体选择哪种方案还是要根据项目的实际情况来定。



#### 2) **ShardingSphere介绍**

Apache ShardingSphere 是一款分布式的数据库生态系统， 可以将任意数据库转换为分布式数据库，并通过数据分片、弹性伸缩、加密等能力对原有数据库进行增强。

Apache ShardingSphere 设计哲学为 Database Plus，旨在构建异构数据库上层的标准和生态。 它关注如何充分合理地利用数据库的计算和存储能力，而并非实现一个全新的数据库。 它站在数据库的上层视角，关注它们之间的协作多于数据库自身。

官方文档：https://shardingsphere.apache.org/document/current/cn/overview/

Apache ShardingSphere 由 ShardingSphere-JDBC 和 ShardingSphere-Proxy组成。

##### **ShardingSphere-JDBC**

ShardingSphere-JDBC 定位为轻量级 Java 框架，在 Java 的 JDBC 层提供的额外服务。 它使用客户端直连数据库，以 jar 包形式提供服务，无需额外部署和依赖，可理解为增强版的 JDBC 驱动，完全兼容 JDBC 和各种 ORM 框架。

- 适用于任何基于 JDBC 的 ORM 框架，如：JPA, Hibernate, Mybatis, Spring JDBC Template 或直接使用 JDBC；
- 支持任何第三方的数据库连接池，如：DBCP, C3P0, BoneCP, HikariCP 等；
- 支持任意实现 JDBC 规范的数据库，目前支持 MySQL，PostgreSQL，Oracle，SQLServer 以及任何可使用 JDBC 访问的数据库。

如下图：

![res8b1b54osh1fxh90jchlnonwh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/res8b1b54osh1fxh90jchlnonwh.png)
> 图注：图片展示了ShardingSphere-JDBC架构图。左侧和右侧均为Java应用，分别包含业务代码和ShardingSphere-JDBC。中间是治理中心。下方有三个数据库。业务代码通过ShardingSphere-JDBC与治理中心交互，再由治理中心调度到数据库。该图直观呈现了ShardingSphere-JDBC对jdbc访问数据库的封装，实现了分库分表功能，与上文介绍的ShardingSphere适用范围及功能相呼应。

ShardingSphere-JDBC是一个框架，对jdbc访问数据库进行了封装，实现了分库分表的功能。

Governance Center的作用有点类似于微服务架构中的配置中心



##### **ShardingSphere-Proxy**

ShardingSphere-Proxy 定位为透明化的数据库代理端，通过实现数据库二进制协议，对异构语言提供支持。 目前提供 MySQL 和 PostgreSQL 协议，透明化数据库操作，对 DBA 更加友好。

- 向应用程序完全透明，可直接当做 MySQL/PostgreSQL 使用；
- 兼容 MariaDB 等基于 MySQL 协议的数据库，以及 openGauss 等基于 PostgreSQL 协议的数据库；
- 适用于任何兼容 MySQL/PostgreSQL 协议的的客户端，如：MySQL Command Client, MySQL Workbench, Navicat 等。

如下图：

ShardingSphere-Proxy和MyCat的作用一样，是一个数据库代理，应用程序原来是直接访问数据库，现在通过访问ShardingSphere-Proxy实现对数据分库分表。

![wtl2ba3jvof9quxvcd9ct8xqnpd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wtl2ba3jvof9quxvcd9ct8xqnpd.png)
> 图注：图片展示了ShardingSphere-Proxy的工作原理。左侧为Application，包含Business Code，通过ShardingSphere-Proxy与数据库交互。右侧有MySQL/PostgreSQL CLI和MySQL/PostgreSQL GUI。ShardingSphere-Proxy连接数据库，数据库间有箭头连接，表示数据交互。该图与上下文紧密相关，直观呈现了ShardingSphere-Proxy作为数据库代理，应用程序通过其访问数据库，实现对数据分库分表的功能。

我们的课程讲解ShardingSphere-JDBC，因为它的功能更强大，灵活性更好。



##### **哪个应用得更多？**

ShardingSphere-JDBC 和 ShardingSphere-Proxy哪个应用的更多？

实际上，选择哪一个取决于具体的应用场景和技术背景。如果应用程序已经使用了Spring框架，并且愿意在应用程序内部做少量的配置修改，那么ShardingSphere-JDBC是一个不错的选择。相反，如果希望保持应用程序代码不变，或者应用程序本身不适合修改JDBC访问逻辑，那么ShardingSphere-Proxy可能更适合。

在实际应用中，ShardingSphere-JDBC因为其更简单的集成方式而被广泛应用，特别是在已有Spring框架的环境中。但是随着人们对非侵入式解决方案的需求增加，ShardingSphere-Proxy也逐渐受到了关注，尤其是在需要与多种不同应用程序集成的情况下。

本项目讲解ShardingSphere-JDBC的应用方法。



### 2.3 ShardingSphere-JDBC入门

#### 2.3.1 搭建SpringBoot工程

##### 1）入门程序说明

本节我们对订单库进行分库分表，通过快速入门程序的开发，快速体验ShardingSphere-JDBC的使用 

方法。

开发流程是：

先创建一个普通的SpringBoot工程，可以实现向数据库表插入数据。

再改造为使用ShardingSphere-JDBC对数据库分库分表。

##### 2）创建数据库

创建数据库：shardingsphere-test-01、shardingsphere-test-02

分别在这两个数据库中创建t_order_1、t_order_2、t_order表，这两个表就是对订单表的水平分表。

运行下边的脚本：

```Java
DROP TABLE IF EXISTS `t_order_1`;
CREATE TABLE `t_order_1`
(
    `id` bigint(20)                                             NOT NULL COMMENT '订单id',
    `price`    decimal(10, 2)                                         NOT NULL COMMENT '订单价格',
    `user_id`  bigint(20)                                             NOT NULL COMMENT '下单用户id',
    `status`   varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '订单状态',
    PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB
  CHARACTER SET = utf8
  COLLATE = utf8_general_ci
  ROW_FORMAT = Dynamic;
DROP TABLE IF EXISTS `t_order_2`;
CREATE TABLE `t_order_2`
(
    `id` bigint(20)                                             NOT NULL COMMENT '订单id',
    `price`    decimal(10, 2)                                         NOT NULL COMMENT '订单价格',
    `user_id`  bigint(20)                                             NOT NULL COMMENT '下单用户id',
    `status`   varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '订单状态',
    PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB
  CHARACTER SET = utf8
  COLLATE = utf8_general_ci
  ROW_FORMAT = Dynamic;

DROP TABLE IF EXISTS `t_order`;
CREATE TABLE `t_order`
(
    `id` bigint(20)                                             NOT NULL COMMENT '订单id',
    `price`    decimal(10, 2)                                         NOT NULL COMMENT '订单价格',
    `user_id`  bigint(20)                                             NOT NULL COMMENT '下单用户id',
    `status`   varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '订单状态',
    PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB
  CHARACTER SET = utf8
  COLLATE = utf8_general_ci
  ROW_FORMAT = Dynamic;
```

创建成功

![nsh6bxwmxofp7yxyzqhcnhabn5f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/nsh6bxwmxofp7yxyzqhcnhabn5f.png)
> 图注：该图片展示了两个名为“shardingsphere-test-01”和“shardingsphere-test-02”的模块，每个模块下都包含一个“tables”分组。每个tables分组中都有3个表，分别为t_order、t_order_1和t_order_2。这些内容是搭建ShardingSphere-JDBC入门程序过程中，创建普通SpringBoot工程“shardingsphere-test”时生成的数据库表结构相关内容，对应文档中提到的生成模型类及表的相关操作结果呈现。



##### 3）创建SpringBoot工程

创建一个普通的SpringBoot工程shardingsphere-test：

![ohjybr0upobargxgtz5cv8fwnoh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ohjybr0upobargxgtz5cv8fwnoh.png)
> 图注：这张图片展示的是创建SpringBoot工程的界面，该工程名称为shardingsphere-test，用于搭建ShardingSphere-JDBC的入门项目，与文档中创建名为shardingsphere-test的普通SpringBoot工程的内容相关。界面左侧选中了“Spring Initializr”选项，用于初始化SpringBoot项目，可配置的内容还包含项目的存储路径、开发语言、构建工具、组织标识、包名、JDK版本、Java版本以及打包方式等参数。



###### pom.xml

springboot和mybatis-plus的版本使用和项目一样的版本，如下：

```Java
<properties>
    <java.version>1.8</java.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <spring-boot.version>2.7.10</spring-boot.version>
    <mybatis-plus.version>3.4.3.2</mybatis-plus.version>
</properties>
```

pom.xml内容如下：

```Java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.itcast</groupId>
    <artifactId>shardingsphere-test</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>shardingsphere-test</name>
    <description>shardingsphere-test</description>
    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.7.10</spring-boot.version>
        <mybatis-plus.version>3.4.3.2</mybatis-plus.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--数据库-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.32</version>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>${mybatis-plus.version}</version>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.26</version>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
                <configuration>
                    <mainClass>cn.itcast.shardingsphere.ShardingsphereTestApplication</mainClass>
                    <skip>true</skip>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```



###### 生成mapper和模型类

使用工具基于表生成模型类和mapper

生成过程略，生成成功如下图：

![ekofbcmwnomkctxzi9ucjpqanmv](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ekofbcmwnomkctxzi9ucjpqanmv.png)
> 图注：图片展示了shardingsphere-test项目的目录结构。在main目录下的java文件夹中，有cn、itcast和shardingsphere三个子文件夹。其中，shardingsphere文件夹下包含mapper、model、service和impl四个子文件夹，分别对应TOrderMapper、TOrder、TOrderServiceImpl和ITOrderService等类。该图片与上文“生成mapper和模型类”内容相关，直观呈现了生成成功后模型类和mapper在项目中的存放位置。

###### 配置application.yaml

```Java
server:
  port: 8080
spring:
  application:
    name: shardingsphere-test
  profiles:
    active: dev

  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.101.68:3306/shardingsphere-test-01?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai&stringtype=unspecified
    username: root
    password: mysql
    type: com.zaxxer.hikari.HikariDataSource
    hikari:
      auto-commit: true
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
      minimum-idle: 3
      maximum-pool-size: 10
      pool-name: HikariCP
```

###### 启动类

```Java
package cn.itcast.shardingsphere;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
@MapperScan("cn.itcast.shardingsphere.mapper")
public class ShardingsphereTestApplication {

    public static void main(String[] args) {
        SpringApplication.run(ShardingsphereTestApplication.class, args);
    }

}
```

###### 测试

编写测试方法，向t_order表插入数据

```Java
package cn.itcast.shardingsphere.mapper;

import cn.itcast.shardingsphere.model.domain.TOrder;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.math.BigDecimal;

import static org.junit.jupiter.api.Assertions.*;

@SpringBootTest
class TOrderMapperTest {

    @Autowired
    private TOrderMapper tOrderMapper;

    @Test
    void testInsert() {
        for (long i = 0; i < 10; i++) {
            tOrderMapper.insert(new TOrder().setUserId(i).setPrice(BigDecimal.valueOf(100.0)).setStatus("ok"));
        }
    }

}
```



#### 2.3.2 实现分库分表

##### 1）添加ShardingSphere-JDBC依赖

下边集成 ShardingSphere-JDBC实现分库分表。

shardingsphere-jdbc用5.4.0版本

添加shardingsphere-jdbc依赖

```Java
<properties>
    <java.version>1.8</java.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <spring-boot.version>2.7.10</spring-boot.version>
    <shardingsphere-jdbc.version>5.4.0</shardingsphere-jdbc.version>
    <snakeyaml.version>1.33</snakeyaml.version>
    <mybatis-plus.version>3.4.3.2</mybatis-plus.version>
</properties>
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>shardingsphere-jdbc-core</artifactId>
    <version>${shardingsphere-jdbc.version}</version>
</dependency>
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>${snakeyaml.version}</version>
</dependency>
```

org.yaml:用于解析ShardingSphere-JDBC的配置文件语法。

##### 2）配置专属JDBC驱动

原来我们在application.yaml中配置的JDBC驱动如下：

```Java
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.101.68:3306/shardingsphere-test-01?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai&stringtype=unspecified
    username: root
    password: mysql
```

集成ShardingSphere-JDBC需要修改为ShardingSphere-JDBC提供的JDBC驱动。

屏蔽原来的datasource，改为如下：

```Java
  datasource:
#    driver-class-name: com.mysql.cj.jdbc.Driver
#    url: jdbc:mysql://192.168.101.68:3306/shardingsphere-test-01?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai&stringtype=unspecified
    driver-class-name: org.apache.shardingsphere.driver.ShardingSphereDriver
    url: jdbc:shardingsphere:classpath:shardingsphere-jdbc-dev.yml
```

##### 3）配置分库分表策略(重要)

下边配置shardingsphere-jdbc-dev.yml文件，在此文件配置分库分表的策略。

###### 配置数据源

首先配置数据源，因为目标是分两个数据库，所以配置两个数据源。

在resources下创建shardingsphere-jdbc-dev.yml文件，配置内容如下：

```Java
dataSources:
  db-orders-0:
    dataSourceClassName: com.zaxxer.hikari.HikariDataSource
    jdbcUrl: jdbc:mysql://192.168.101.68:3306/shardingsphere-test-01?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    username: root
    password: mysql
  db-orders-1:
    dataSourceClassName: com.zaxxer.hikari.HikariDataSource
    jdbcUrl: jdbc:mysql://192.168.101.68:3306/shardingsphere-test-02?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    username: root
    password: mysql
```

###### 配置数据分片规则

下边配置数据分片规则，非常重要。

分片规则流程：

首先配置逻辑表，在逻辑表中配置数据结点、分表策略和分库策略。

**配置逻辑表**

参考此文档查看配置项的意义：https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/yaml-config/rules/sharding/

```Java
rules:
- !SHARDING
  tables:
    t_order:
      actualDataNodes: db-orders-${0..1}.t_order_${1..2}
      tableStrategy:
        standard:
          shardingColumn: id
          shardingAlgorithmName: orders_table_inline
      databaseStrategy:
        standard:
          shardingColumn: user_id
          shardingAlgorithmName: orders_database_inline
```

t_order：逻辑表名，与模型类中指定的表名一致：

```Java
@TableName("t_order")
public class Order implements Serializable {
```

actualDataNodes: 配置实际数据结点，由数据源+真实表名组成

tableStrategy：分表策略

shardingColumn：指定分片键，这里指定分片键为id表示根据主键进行水平分表。

      shardingAlgorithmName：分片算法名称，自定义名称，下边会单独定义此算法。

databaseStrategy：分库策略

shardingColumn：指定分片键，这里指定分片键为user_id表示根据user_id进行分库。

　shardingAlgorithmName：分片算法名称，自定义名称，下边会单独定义此算法。



**配置分库分表算法**

与tables平级，配置shardingAlgorithms。

内容如下：

```Java
rules:
- !SHARDING
  tables:
    t_order:
      actualDataNodes: db-orders-${0..1}.t_order_${1..2}
      tableStrategy:
        standard:
          shardingColumn: id
          shardingAlgorithmName: orders_table_inline
      databaseStrategy:
        standard:
          shardingColumn: user_id
          shardingAlgorithmName: orders_database_inline
  shardingAlgorithms:
    # 订单-分库算法
    orders_database_inline:
      type: INLINE
      props:
        # 分库算法表达式
        algorithm-expression: db-orders-${user_id % 2}
        # 分库支持范围查询
        allow-range-query-with-inline-sharding: true
    # 订单-分表算法
    orders_table_inline:
      type: INLINE
      props:
        # 分表算法表达式
        algorithm-expression: t_order_${id % 2 + 1}
        # 允许范围查询
        allow-range-query-with-inline-sharding: true
```

orders_database_inline：分库算法名称，对应上边配置的shardingAlgorithmName: orders_database_inline。

algorithm-expression: db-orders-\${user_id % 2}: 根据 db-orders-\${user_id % 2}表达式决定数据库

举例：user_id为1，通过计算user_id % 2得出1，数据库为db-orders-1

         user_id为2，通过计算user_id % 2得出0，数据库为db-orders-0

orders_table_inline：分表算法名称，对应上边配置的shardingAlgorithmName: orders_table_inline

 algorithm-expression: t_order\_\${id % 2 + 1}: 根据 t_order\_\${id % 2 + 1}表达决定实际的表名。

  举例：id为1，通过计算id % 2 + 1得出2，表名为：t_order_2

      id为2，通过计算id % 2 + 1得出1，表名为：t_order_1



参考文档：https://shardingsphere.apache.org/document/current/cn/user-manual/shardingsphere-jdbc/yaml-config/rules/sharding/



完整的配置文件内容如下：

```Java
dataSources:
  db-orders-0:
    dataSourceClassName: com.zaxxer.hikari.HikariDataSource
    jdbcUrl: jdbc:mysql://192.168.101.68:3306/shardingsphere-test-01?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    username: root
    password: mysql
  db-orders-1:
    dataSourceClassName: com.zaxxer.hikari.HikariDataSource
    jdbcUrl: jdbc:mysql://192.168.101.68:3306/shardingsphere-test-02?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    username: root
    password: mysql
rules:
- !TRANSACTION
  defaultType: BASE
- !SHARDING
  tables:
    t_order:
      actualDataNodes: db-orders-${0..1}.t_order_${1..2}
      tableStrategy:
        standard:
          shardingColumn: id
          shardingAlgorithmName: orders_table_inline
      databaseStrategy:
        standard:
          shardingColumn: user_id
          shardingAlgorithmName: orders_database_inline
  shardingAlgorithms:
    # 订单-分库算法
    orders_database_inline:
      type: INLINE
      props:
        # 分库算法表达式
        algorithm-expression: db-orders-${user_id % 2}
        # 分库支持范围查询
        allow-range-query-with-inline-sharding: true
    # 订单-分表算法
    orders_table_inline:
      type: INLINE
      props:
        # 分表算法表达式
        algorithm-expression: t_order_${id % 2 + 1}
        # 允许范围查询
        allow-range-query-with-inline-sharding: true
  # id生成器
  keyGenerators:
    snowflake:
      type: SNOWFLAKE
props:
  sql-show: true
```

##### 4) 插入测试

首先向订单表插入10条数据，将user_id固定为1，根据分库策略数据将写入db-orders-1数据库。

根据 t_order\_\${id % 2 + 1}可知，根据订单id决定插入的表，订单id为偶数则插入t_order_1，奇数则插入t_order_2表。

观察日志：

首先解析逻辑SQL，现根据分片规则生成真实SQL。

```Java
2024-09-26 14:05:58.239  INFO 15532 --- [           main] ShardingSphere-SQL                       : Logic SQL: INSERT INTO t_order  ( id,
user_id,
price,
status )  VALUES  ( ?,
?,
?,
? )
2024-09-26 14:05:58.239  INFO 15532 --- [           main] ShardingSphere-SQL                       : Actual SQL: db-orders-0 ::: INSERT INTO t_order_1  ( id,
user_id,
price,
status )  VALUES  (?, ?, ?, ?) ::: [1839184587743883266, 2, 5.0, WAIT_PAY]
...
```

通过日志可以发现id为奇数的被插入到t_order_2表，为偶数的被插入到t_order_1表，达到预期目标。



再向订单表插入10条数据，将user_id固定为2, 根据分库策略数据将写入db-orders-0数据库。



请自行进行测试并观察结果是否符合预期。

##### 5）查询测试

编写测试方法：

根据订单id进行查询。

```Java
@Test
public void testSelect(){
    //订单id
    Long id = 1L;
    //根据订单id和用户id查询订单
    List<TOrder> orders = tOrderMapper.selectList(new LambdaQueryWrapper<TOrder>()
            .eq(TOrder::getId, id)
    );
    System.out.println(orders);
}
```

运行测试观察日志：

```Java
2024-09-26 15:58:23.734  INFO 41948 --- [           main] ShardingSphere-SQL                       : Logic SQL: SELECT  id,user_id,price,status  FROM t_order 
 
 WHERE (id = ?)
2024-09-26 15:58:23.734  INFO 41948 --- [           main] ShardingSphere-SQL                       : Actual SQL: db-orders-0 ::: SELECT  id,user_id,price,status  FROM t_order_2 
 
 WHERE (id = ?) ::: [1]
2024-09-26 15:58:23.734  INFO 41948 --- [           main] ShardingSphere-SQL                       : Actual SQL: db-orders-1 ::: SELECT  id,user_id,price,status  FROM t_order_2 
 
 WHERE (id = ?) ::: [1]
```

为什么最终执行两条SQL？

因为ShardingSphere-JDBC也不清楚id为1数据在哪个数据库，所以会从两个数据库都去查询。



如果查询代码改为如下：

```Java
@Test
public void testSelect(){
    //订单id
    Long id = 1L;
    //用户id
    Long userId = 2L;
    //根据订单id和用户id查询订单
    List<TOrder> orders = tOrderMapper.selectList(new LambdaQueryWrapper<TOrder>()
            .eq(TOrder::getId, id)
            .eq(TOrder::getUserId, userId)
    );
    System.out.println(orders);
}
```

请自己运行测试，观察日志，什么生成几条SQL呢？ 思考原因。



##### 6）小结

如何使用ShardingSphere-JDBC实现分表分表？





### 2.2 家政项目实现订单分库分表

#### 2.2.1 搭建分库分表环境

##### 1）分库分表方案

对订单数据进行分表分表。

1、Hash方式

拿分库举例：将订单号除以数据库个数求余数，假如有3个数据库实例，计算表达式为：db_订单号%3， 比如：10号订单会存入到db_1数据库，11号订单存储到db_2数据库。

此方式的优点是：数据均匀。

缺点：扩容时需要迁移数据。比如：3个数据库改为4个数据库，此时计算表达式为：db_订单号%4，10号订单存储到db_2数据库，11号订单存储到db_3数据库，此时就需要进行数据迁移，将10号订单由db_1迁移到db_2。



2、rang方式（范围方式）

比如：0到500万到db_1数据库，500万到1000万到db_2数据库，依次类推。

此方式的优点：扩容时不需要迁移数据。

缺点：存在数据热点问题，因为订单号是从0开始依次往上累加，前期所有的数据都是访问db_1数据库，db_1的压力较大。

3、综合方案

综合1、2方案的优缺点制定综合方案。

分库方案：设计三个数据库，根据用户id哈希，分库表达式为：db_用户id % 3

参考历史经验，前期设计三个数据库，每个数据库使用主从结构部署，可以支撑项目五年左右的运行，虽然哈希存在数据迁移问题，在很长一段时间也不用考虑这个问题。

分表方案：根据订单范围分表，0---1500万落到table_0，1500万---3000万落到table_1，依次类推。

 根据范围分表不存在数据库迁移问题，方便系统扩容。

整体方案如下图：

<readonly-block type="diagram"></readonly-block>

##### 2）**创建数据库**

订单数据库分为三个库 :jzo2o-orders-0、jzo2o-orders-1、jzo2o-orders-2

![whqgb5uisootvqxqk7zcxqkennh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/whqgb5uisootvqxqk7zcxqkennh.png)
> 图注：图片展示了三个数据库，分别为jzo2o-orders-0、jzo2o-orders-1、jzo2o-orders-2。这与文档中“2.2.1搭建分库分表环境”章节下“2）创建数据库”部分的内容相关，该部分提到订单数据库分为三个库，此图直观呈现了这三个数据库，与上下文对数据库分库分表的介绍相呼应，为后续导入SQL脚本等操作提供可视化参考。

下边分别向三个数据库导入jzo2o-orders-sharding.sql（从课程资料下的SQL脚本目录获取）。

每个数据库对orders、biz_snapshot、orders_serve进行分表（暂分3个表），其它表为广播表(即在每个数据库都存在且数据是完整的)，如下图：

![mxarb07jxo8ykex16fgcdjsinbc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mxarb07jxo8ykex16fgcdjsinbc.png)
> 图注：图片展示了jzo2o-orders-0数据库下的tables目录内容。目录下包含多个表，如biz_snapshot、breach_record、history_orders_sync、history_orders_sync、orders等，其中orders表及其多个分表（如orders_0、orders_1等）尤为突出。该图片与文档中“2.2.1搭建分库分表环境”章节下“2）创建数据库”内容相关，直观呈现了每个数据库对orders、biz_snapshot、orders_serve进行分表（暂分3个表）的结构情况。







##### 3）**添加依赖**

在jzo2o-framework下的jzo2o-shardingsphere-jdbc中添加了具体shardingsphere的依赖，所以我们只需要在orders-base工程引入jzo2o-shardingsphere-jdbc的依赖即可，如下：

```Java
<dependency>
    <groupId>com.jzo2o</groupId>
    <artifactId>jzo2o-shardingsphere-jdbc</artifactId>
</dependency>
```



##### 4) **配置分片规则**

在orders-base工程的resources下配置shardingsphere-jdbc-dev.yml（直接从项目源码目录下拷贝shardingsphere-jdbc-dev.yml）。

配置文件如下：

jzo2o-orders-0、jzo2o-orders-1、jzo2o-orders-2表示三个数据源对应三个订单数据库。

每个数据库中对orders、orders_serve、biz_snapshot进行分表。

详细如下：

```YAML
dataSources:
  jzo2o-orders-0:
    dataSourceClassName: com.zaxxer.hikari.HikariDataSource
    jdbcUrl: jdbc:mysql://192.168.101.68:3306/jzo2o-orders-0?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    username: root
    password: mysql
  jzo2o-orders-1:
    dataSourceClassName: com.zaxxer.hikari.HikariDataSource
    jdbcUrl: jdbc:mysql://192.168.101.68:3306/jzo2o-orders-1?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    username: root
    password: mysql
  jzo2o-orders-2:
    dataSourceClassName: com.zaxxer.hikari.HikariDataSource
    jdbcUrl: jdbc:mysql://192.168.101.68:3306/jzo2o-orders-2?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    username: root
    password: mysql
rules:
- !TRANSACTION
  defaultType: BASE
- !SHARDING
  tables:
    orders:
      actualDataNodes: jzo2o-orders-${0..2}.orders_${0..2}
      tableStrategy:
        standard:
          shardingColumn: id
          shardingAlgorithmName: orders_table_inline
      databaseStrategy:
        standard:
          shardingColumn: user_id
          shardingAlgorithmName: orders_database_inline
    orders_serve:
      actualDataNodes: jzo2o-orders-${0..2}.orders_serve_${0..2}
      tableStrategy:
        standard:
          shardingColumn: id
          shardingAlgorithmName: orders_serve_table_inline
      databaseStrategy:
        standard:
          shardingColumn: serve_provider_id
          shardingAlgorithmName: orders_serve_database_inline
    biz_snapshot:
      actualDataNodes: jzo2o-orders-${0..2}.biz_snapshot_${0..2}
      tableStrategy:
        standard:
          shardingColumn: biz_id
          shardingAlgorithmName: biz_snapshot_table_inline
      databaseStrategy:
        standard:
          shardingColumn: db_shard_id
          shardingAlgorithmName: biz_snapshot_database_inline
  shardingAlgorithms:
    # 订单-分库算法
    orders_database_inline:
      type: INLINE
      props:
        # 分库算法表达式
        algorithm-expression: jzo2o-orders-${user_id % 3}
        # 分库支持范围查询
        allow-range-query-with-inline-sharding: true
    # 订单-分表算法
    orders_table_inline:
      type: INLINE
      props:
        # 分表算法表达式
        algorithm-expression: orders_${(int)Math.floor(id % 10000000000 / 15000000)}
        # 允许范围查询
        allow-range-query-with-inline-sharding: true
    # 服务单-分库算法
    orders_serve_database_inline:
      type: INLINE
      props:
        # 分库算法表达式
        algorithm-expression: jzo2o-orders-${serve_provider_id % 3}
        # 允许范围查询
        allow-range-query-with-inline-sharding: true
    # 服务单-分表算法
    orders_serve_table_inline:
      type: INLINE
      props:
        # 允许范围查询
        algorithm-expression: orders_serve_${(int)Math.floor(id % 10000000000 / 15000000)}
        # 允许范围查询
        allow-range-query-with-inline-sharding: true
    # 快照-分库算法
    biz_snapshot_database_inline:
      type: INLINE
      props:
        # 分库算法表达式
        algorithm-expression: jzo2o-orders-${db_shard_id % 3}
        # 允许范围查询
        allow-range-query-with-inline-sharding: true
    # 快照-分表算法
    biz_snapshot_table_inline:
      type: INLINE
      props:
        # 允许范围查询
        algorithm-expression: biz_snapshot_${(int)Math.floor((Long.valueOf(biz_id))  % 10000000000 / 15000000)}
        # 允许范围查询
        allow-range-query-with-inline-sharding: true
  # id生成器
  keyGenerators:
    snowflake:
      type: SNOWFLAKE
- !BROADCAST
  tables:
    - breach_record
    - orders_canceled
    - orders_refund
    - orders_dispatch
    - orders_seize
    - serve_provider_sync
    - state_persister
    - orders_dispatch_receive
    - undo_log
    - history_orders_sync
    - history_orders_serve_sync
props:
  sql-show: true
```

说明如下：

dataSources：数据源

jzo2o-orders-x：与actualDataNodes对应。

下边以orders表为例说明分库分表策略：

分库键：user_id

分库表达式：jzo2o-orders-\${user_id % 3}

根据用户id计算落到哪个数据库

分表键：id 

分表表达式：orders\_\${(int)Math.floor(id % 10000000000 / 15000000)}

按1500万为单位进行分表，比如：订单号2311020000000000019，为19位，表达式的值为19，匹配表orders_0，如果表达式的值大于1500万小于3000万匹配表orders_1。

```Java
  tables:
    orders:
      #由数据源名 + 表名组成（参考 Inline 语法规则）
      actualDataNodes: jzo2o-orders-${0..2}.orders_${0..2}
      tableStrategy:#分表策略
        standard:
          shardingColumn: id #分片列名称
          shardingAlgorithmName: orders_table_inline  # 分片算法名称
      databaseStrategy:#分库策略
        standard:
          shardingColumn: user_id
          shardingAlgorithmName: orders_database_inline
   shardingAlgorithms:
    # 订单-分库算法
    orders_database_inline:
      type: INLINE
      props:
        # 分库算法表达式
        algorithm-expression: jzo2o-orders-${user_id % 3}
        # 分库支持范围查询
        allow-range-query-with-inline-sharding: true
    # 订单-分表算法
    orders_table_inline:
      type: INLINE
      props:
        # 分表算法表达式
        algorithm-expression: orders_${(int)Math.floor(id % 10000000000 / 15000000)}
        # 允许范围查询
        allow-range-query-with-inline-sharding: true    
```

!BROADCAST：指定广播表

广播表在jzo2o-orders-0、jzo2o-orders-1、jzo2o-orders-2每个数据库的数据一致。



##### 5) 小结

项目的分库分表是怎么做的？



#### 2.2.2 测试分库分表

##### 1）配置数据源

进入nacos在jzo2o-orders-manager.yaml中配置数据源使用ShardingSphereDriver：

```Java
spring:
  datasource:
    driver-class-name: org.apache.shardingsphere.driver.ShardingSphereDriver
    url: jdbc:shardingsphere:classpath:shardingsphere-jdbc-${spring.profiles.active}.yml
```

如下图：

![shctbkqp0osskdxh1d7cicllntf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/shctbkqp0osskdxh1d7cicllntf.png)
> 图注：图片展示的是nacos中配置数据源的界面。上方显示Data ID为jzo2o - orders - manager - yml，Group为DEFAULT_GROUP。配置内容以YAML格式呈现，包括spring、datasource、url等参数，其中url为jdbc:shardingsphere:classpath:shardingsphere - jdbc - ${spring.profiles.active}.yml。该图片与文档中“2.2.2测试分库分表”部分的“1）配置数据源”内容相关，用于说明在jzo2o - orders - manager.yaml中配置数据源使用ShardingSphereDriver的操作。

完整配置如下：

```Java
spring:
  datasource:
    driver-class-name: org.apache.shardingsphere.driver.ShardingSphereDriver
    url: jdbc:shardingsphere:classpath:shardingsphere-jdbc-${spring.profiles.active}.yml
mybatis-plus:
  configuration:
    default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
  page:
    max-limit: 1000
  global-config:
    field-strategy: 0
    db-config:
      logic-delete-field: isDeleted
      id-type: assign_id
xxl-job:
  port: 9998
canal:
    enable: true
    sync:
        application-name: ${spring.application.name}
    rabbit-mq:
        routing-keys: canal-mq-jzo2o-orders-manager
        exchange: exchange.canal-jzo2o
        queue: canal-mq-jzo2o-orders-manager
rabbit-mq:
    enable: true
jzo2o:
  trade:
    aliEnterpriseId: 2088241317544335
    wechatEnterpriseId: 1561414331
  job:
    autoEvaluateCount: 100
  openPay: true
```



##### 2）状态机分库分表

由于对状态机进行了分库分表，需要修改创建订单方法中启动状态机代码：

使用start(Long dbShardId, String bizId, T bizSnapshot) 方法启动状态机，传入分片键user_id。

```Java
@Transactional(rollbackFor = Exception.class)
public void add(Orders orders) {
    ....
    //状态机启动
    orderStateMachine.start(orders.getUserId(), orders.getId().toString(), orderSnapshotDTO);
}
```



支付成功调用状态机变更状态方法：

使用：changeStatus(Long dbShardId, String bizId, StatusChangeEvent statusChangeEventEnum, T bizSnapshot)变更状态, 传入分片键user_id。

```Java
@Transactional(rollbackFor = Exception.class)
public void paySuccess(TradeStatusMsg tradeStatusMsg) {
  ....
    orderStateMachine.changeStatus(orders.getUserId(), String.valueOf(orders.getId()), OrderStatusChangeEventEnum.PAYED, orderSnapshotDTO);

}
```



##### 3）测试

测试流程：

下单、支付、取消订单

预期结果：

下单成功：根据用户id分库，写入jzo2o-orders-x的其中一个数据库，根据订单号分表，写入orders_x其中一个订单表。

状态机：根据用户id分库，写入jzo2o-orders-x的其中一个数据库，根据biz_id字段分表，写入biz_snapshot_x中一个状态表机。state_persister为广播表不进行分表在三个数据库数据一致。

支付及取消订单业务操作正常。

orders_canceled表为广播表不进行分表在三个数据库数据一致。



## 3 订单查询优化

### 3.1 订单查询优化方案

#### 1) **为什么要优化订单查询？**

1）订单查询是一个高频接口，并且订单表数据量大。

2）面向C端用户的订单查询接口其访问量非常大。

3）对于运营端的订单查询接口虽然访问量不大但由于订单数据较多也需要进行优化，提高查询性能。

#### 2) 订单详情优化方案

针对单条订单信息查询接口可以使用缓存进行优化。

对于单条订单信息查询接口通过快照查询接口查询订单的详情信息。

参考AbstractStateMachine类的String getCurrentSnapshotCache(String bizId)快照查询方法，将快照信息缓存到 redis提供查询效率。

根据订单Id查询缓存信息，先从缓存查询，如果缓存没有则查询快照表的数据然后保存到缓存中。缓存设置了过期时间是30分钟。

当订单状态变更，此时订单最新状态的快照有变更，会删除快照缓存，当再次查询快照时从数据库查询最新的快照信息进行缓存。

<whiteboard token="KxxXw5mOIh9s8RbS0WAc4j4GnUe"></whiteboard>

#### 3) **用户端订单列表优化方案**

用户端通过小程序查询订单列表，界面上没有分页查询的按钮，用户端查询订单列表可采用滚动查询的方法。

滚动查询就是一次查询指定数量的记录，不用进行count查询，省去count查询的消耗。

具体方案如下：

1. **首先查询符合条件的订单ID。**

由于是滚动查询需要传入滚动ID，这里我们在订单表使用排序字段sort_by作为滚动ID。

滚动ID是一种递增的序列号，按服务预约时间降序排列且滚动ID具有唯一性，滚动ID的规则是：服务预约时间+订单号后5位。

 滚动查询方式： 按SORT_BY降序，取小于SORT_BY的n条记录 第一页传入SORT_BY参数为空 从第二页开始传入上一页最后一条记录的SORT_BY

示例：

<whiteboard token="Pd30wrLvjhK5Uzb3Jl2cWb7Fn2v"></whiteboard>

1. **使用覆盖索引优化**

根据查询条件查询符合条件的订单ID，这里使用覆盖索引优化的方法。

我们知道在InnoDB存储引擎中有两种索引存储形式：

**聚集索引**：查询条件只有主键的情况会通过聚集索引查询。

**非聚集索引**：查询条件有多个，此时为了提高查询效率可以创建多个字段的联合索引，根据非聚集索引找到符合条件主键，如果要查询的列只有索引字段则通过非聚集索引直接拿到字段值返回，如果要查询列有一部分在索引之外此时会**进行回表查询**聚集索引最终拿到数据。

示例：

user表（id、name、 age、address）

对name、age创建联合索引。

sql1：

select id、name、age  from  user where name=? and age =? 

该查询直接从索引中拿到符合条件的数据，不存在回表查询。

sql2：

select \*  from  user where name=? and age =? 

该查询列是select \* ，address没有包含在索引中，where条件通过联合索引找到符合条件的主键，再通过主键回表查询聚集索引，最终拿到数据。

覆盖索引是什么呢？

 覆盖索引是一种优化手段，上边的sql1就是实现了覆盖索引。

**覆盖索引（covering index）**指一个查询语句的执行只需要从非聚集索引中就可以得到查询记录，而不需要回表去查询聚集索引，可以称之为实现了索引覆盖。

根据上边的需求，我们根据查询条件建立联合索引，通过联合索引找到符合条件的订单ID（主键），从索引中找到的符合条件的订单ID无需回表查询聚集索引。



#### 4）**运营端订单列表优化方案**

运营端订单列表由于访问量不大这里无需使用缓存，实现方案是首先通过覆盖索引查询符合条件的订单ID，再根据订单ID查询聚集索引拿到数据。

我们根据查询条件创建的联合字段索引是非聚集索引，先从非聚集索引中查询到符合条件的ID，再根据订单ID从聚集索引中查找数据。



#### 6) 小结

订单查询是如何优化的？



### 3.2 优化订单详情接口

#### 1）阅读状态机快照查询代码

在jzo2o-framework的状态机工程jzo2o-statemachine中提供了查询快照缓存的方法，具体在状态机抽象类AbstractStateMachine中，如下图：

此方法查询的是当前状态下的快照。

```Java
/**
 * 获取当前状态的快照缓存
 *
 * @param bizId 业务id
 * @return 快照信息
 */
public String getCurrentSnapshotCache(String bizId) {
    //先查询缓存，如果缓存没有就查询数据库然后存缓存
    String key = "JZ_STATE_MACHINE:" + name + ":" + bizId;
    Object object = redisTemplate.opsForValue().get(key);
    if (ObjectUtil.isNotEmpty(object)) {
        return object.toString();
    }

    String bizSnapshot = getCurrentSnapshot(bizId);
    redisTemplate.opsForValue().set(key, bizSnapshot, 30, TimeUnit.MINUTES);
    return bizSnapshot;
}
```

状态机生成快照后会清理快照缓存，保证再次查询快照缓存时可以查询到最新的快照信息。

在saveSnapshot()方法中保存快照方法和changeStatus()方法中提供了清理快照的代码。

具体方法如下：

```Java
public void saveSnapshot(Long dbShardId, String bizId, StatusDefine statusDefine, T bizSnapshot) 

public void changeStatus(Long dbShardId, String bizId, StatusChangeEvent statusChangeEventEnum, T bizSnapshot) 
```

#### 2） 订单详情查询优化

找到“根据订单id查询”接口，找到调用的查询订单详情的service方法：

```Java
@Override
public OrderResDTO getDetail(Long id) {
    Orders orders = queryById(id);
    //懒加载方式取消支付超时的 订单
    orders = canalIfPayOvertime(orders);
    OrderResDTO orderResDTO = BeanUtil.toBean(orders, OrderResDTO.class);
    return orderResDTO;
}
```

上边的代码是查询数据库，现在改为查询快照缓存方法，如下：

```Java
@Override
    public OrderResDTO getDetail(Long id) {
        //查询订单
//        Orders orders = queryById(id);
        //从快照表查询快照
        String currentSnapshotJson = orderStateMachine.getCurrentSnapshotCache(String.valueOf(id));
        OrderSnapshotDTO orderSnapshotDTO = JsonUtils.toBean(currentSnapshotJson, OrderSnapshotDTO.class);
        //懒加载方式取消支付超时的 订单
        orderSnapshotDTO = canalIfPayOvertime(orderSnapshotDTO);

        OrderResDTO orderResDTO = BeanUtil.toBean(orderSnapshotDTO, OrderResDTO.class);

        return orderResDTO;
    }
    
    /**
 * 如果支付过期则取消订单
 * @param orderSnapshotDTO
 */
private OrderSnapshotDTO canalIfPayOvertime(OrderSnapshotDTO orderSnapshotDTO){
        //订单到达超时时间则自动取消
        if(orderSnapshotDTO.getOrdersStatus()==OrderStatusEnum.NO_PAY.getStatus() && orderSnapshotDTO.getOverTime().isBefore(LocalDateTime.now())){
            //todo 查询最新支付状态，如果仍是未支付进行取消订单
            //取消订单
            OrderCancelDTO orderCancelDTO = new OrderCancelDTO();
            orderCancelDTO.setId(orderSnapshotDTO.getId());
            orderCancelDTO.setCurrentUserId(orderSnapshotDTO.getUserId());
            orderCancelDTO.setCurrentUserType(UserType.SYSTEM);
            orderCancelDTO.setCancelReason("订单超时支付，自动取消");
            cancel(orderCancelDTO);
//            orders = getById(orders.getId());
            //查询订单快照
            String currentSnapshot = orderStateMachine.getCurrentSnapshotCache(String.valueOf(orderSnapshotDTO.getId()));
            orderSnapshotDTO = JSONUtil.toBean(currentSnapshot, OrderSnapshotDTO.class);
            return orderSnapshotDTO;
        }
        return orderSnapshotDTO;
    }
```

#### 3）测试

测试流程：

进入小程序订单列表界面，找到一个未支付或派单中的订单点击进入订单详情页面。

观察redis是否有快照缓存数据。

预期结果：

查询一次订单信息后快照信息正常缓存，缓存key为：JZ_STATE_MACHINE:order+订单id

示例：

![plembh30iokibsxf3ymccd1jnqh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/plembh30iokibsxf3ymccd1jnqh.png)
> 图注：这是一个订单状态机快照的缓存键值信息展示界面，缓存键为“JZ_STATE_MACHINE:order23110200000000000022”，是与测试流程中提及的快照缓存key规则匹配的示例，用于验证订单详情优化后对应的缓存数据存储形式，清晰呈现了该缓存键对应的订单详情相关字段，包括tradingChannel、ordersStatus、payTime、serviceAddress、userId、payStatus等多项订单信息，可直观确认优化后的快照缓存数据存储内容符合预期的测试结果。

订单变更状态后再次查询订单信息可以得到最新的状态，这说明最新数据已经同步到缓存中。



#### 4）小结

订单快照是怎么实现的？



### 3.2 优化**用户端订单列表**

#### 3.2.1 创建索引

##### 1）**理解滚动ID**

滚动ID在用户端订单列表查询中至关重要，由于需要按服务预约时间降序排列且滚动ID具有唯一性，滚动ID的规则是：服务预约时间+订单号后5位

根据上边的方案使用滚动ID进行查询的伪代码如下：

```Java
select * from 订单表 where 滚动ID<? and 用户id=? order by 滚动ID desc
```

在创建订单时向订单表保存滚动ID为：服务开始时间（毫秒）+订单后5位

如下代码：

```Java
long sortBy = DateUtils.toEpochMilli(orders.getServeStartTime()) + orders.getId() % 100000;
orders.setSortBy(sortBy);
```



##### 2） **创建联合索引**

在创建索引之前需要明确最终执行的SQL语句，如下：

```Java
SELECT id
FROM orders_0
WHERE (orders_status = ? AND user_id = ? AND display = 1 and sort_by<? )
ORDER BY sort_by DESC
LIMIT 10
```

在查询全部订单时 “orders_status = ?” 条件不需要，即执行SQL

```Java
SELECT id
FROM orders_0
WHERE ( user_id = ? AND display = 1 and sort_by<? )
ORDER BY sort_by DESC
LIMIT 10
```

按照最左前缀法则，查询条件必须包括最左边的索引列并将索引字段的顺序和SQL语句where 条件字段的顺序一致，下边创建两个联合索引：

创建索引query_index_0，满足最上边的SQL语句。

```Java
create index query_index_0
    on `jzo2o-orders-1`.orders_0 (orders_status asc, user_id asc, display asc, sort_by desc);
```

创建索引query_index_1，满足最下边的SQL语句。

```Java
create index query_index_1
    on `jzo2o-orders-1`.orders_0 (user_id asc, display asc, sort_by desc);
```

并且要注意上边两个SQL的查询都是按sort_by 降序排列，所以索引中 sort_by 为desc降序。



##### 3）执行计划

1. **什么是执行计划**

索引创建完成如何知道这个SQL语句是否用到这个索引呢？并且跟踪我们的需求最终要实现覆盖索引。

通过MySQL的执行计划即可知道该SQL语句是否用上了我们创建的索引。

MySQL 的执行计划（Execution Plan）是 MySQL 查询优化器生成的一个描述查询执行方式的计划。它是一个详细的执行策略，指示 MySQL 数据库引擎如何执行查询以及如何获取所需的数据。

当你执行一个 SQL 查询时，MySQL 查询优化器负责决定如何最有效地执行这个查询。优化器会考虑多个因素，包括表的索引、表的大小、查询中的过滤条件等，然后生成一个执行计划，描述了查询的执行步骤和顺序。

使用 `EXPLAIN` 关键字来获取 MySQL 查询的执行计划，语法格式如下：

```Java
EXPLAIN  sql语句 
```

下边我们获取第一个SQL的执行计划，第二个SQL的执行计划大家可以自行测试。

对第一个SQL我们加上一些模拟的参数，使用explain获取它的执行计划：

```Java
explain
SELECT id
FROM orders_0
WHERE (orders_status = 0 AND user_id = 1716346406098296832 AND display = 1 and sort_by<1698924600022)
ORDER BY sort_by DESC
LIMIT 10
```

输出：

```Java
[
  {
    "id": 1,
    "select_type": "SIMPLE",
    "table": "orders_0",
    "partitions": null,
    "type": "range",
    "possible_keys": "query_index_0,query_index_1",
    "key": "query_index_0",
    "key_len": "26",
    "ref": null,
    "rows": 1,
    "filtered": 100,
    "Extra": "Using where; Using index"
  }
]
```

每项的说明：

```Java
id： 每个查询步骤的唯一标识符。
select_type： 查询的类型，例如 SIMPLE（简单查询）或 PRIMARY（主查询，嵌套子查询的最外层查询）。
table： 查询涉及的表。
partitions： 使用的分区（如果有分区表的话）。
type： 查询使用的连接类型，例如 ALL（全表扫描）或 index（索引扫描）。
possible_keys： 可能用于此查询的键列表。
key： 实际用于此查询的键。
key_len： 使用的索引长度。
ref： 显示索引的哪一列被用于查询。
rows： MySQL 估计将需要读取的行数。
filtered： 在表中的行数的百分比，表示查询的条件有多少行满足。
Extra： 其他的额外信息，例如使用了哪些索引、是否使用了文件排序等。
```

如果key、key_len为null说明没有用到索引。

上边的输出说明：使用了query_index_0索引。

另外重点关注 Extra：

- **Using where：** 表示MySQL正在对检索出来的行进行额外的WHERE条件过滤，最终返回满足WHERE条件的行。
- **Using index：** 这部分表示MySQL使用了索引来加速查询，但不必回表读取实际的数据行。这种情况通常发生在覆盖索引的情况下，即索引包含了查询所需的所有列，因此MySQL不需要回到数据表中去获取数据，从而提高了查询的性能。

"Using where; Using index"表示使用了覆盖索引，使用了WHERE子句来过滤数据，能够高效地找到并返回满足查询条件的行，而不必浪费资源读取不必要的数据。

1. **根据执行计划调试SQL**

如果我们把SQL改为如下内容，不仅查询ID还查询user_id:

```Java
explain
SELECT id,user_id
FROM orders_0
WHERE (orders_status = 0 AND user_id = 1716346406098296832 AND display = 1 and sort_by<1698924600022)
ORDER BY sort_by DESC
LIMIT 10
```

我们发现输出"Extra"仍然是 "Using where; Using index"

这是因为id、user_id都是索引中的字段，仍然满足覆盖索引。



如果改为查询列中是索引中不存在字段呢？下边的SQL添加了serve_item_name查询列。

```Java
explain
SELECT id,user_id,serve_item_name
FROM orders_0
WHERE (orders_status = 0 AND user_id = 1716346406098296832 AND display = 1 and sort_by<1698924600022)
ORDER BY sort_by DESC
LIMIT 10
```

输出：

```Java
[
  {
    "id": 1,
    "select_type": "SIMPLE",
    "table": "orders_0",
    "partitions": null,
    "type": "range",
    "possible_keys": "query_index_0,query_index_1",
    "key": "query_index_0",
    "key_len": "26",
    "ref": null,
    "rows": 1,
    "filtered": 100,
    "Extra": "Using index condition"
  }
]
```

从key、key_len可以看出使用了索引query_index_0，虽然serve_item_name在索引中不存在但是也使用了query_index_0索引，先从索引中查找满足条件的记录，索引中没有的数据再回表查询。

Using index condition表示：是MySQL一种优化手段，先从索引中查找满足条件的记录，索引中没有的数据再回表查询，可以有效减少回表的次数，大大提升了查询的效率。

如果我们想实现Extra为Using index的效果即覆盖索引该怎么实现呢？

可以在索引中添加serve_item_name字段，

```Java
create index query_index_0
    on `jzo2o-orders-0`.orders_0 (orders_status asc, user_id asc, display asc, sort_by desc, serve_item_name asc);
```

1. **全表扫描**

如果我们把SQL语句改为如下方式：

```Java
explain
SELECT id,user_id,serve_item_name
FROM orders_0
WHERE (display = 1 and sort_by<1698924600022)
ORDER BY sort_by DESC
LIMIT 10
```

输出：

```Java
[
  {
    "id": 1,
    "select_type": "SIMPLE",
    "table": "orders_0",
    "partitions": null,
    "type": "ALL",
    "possible_keys": null,
    "key": null,
    "key_len": null,
    "ref": null,
    "rows": 10,
    "filtered": 10,
    "Extra": "Using where; Using filesort"
  }
]
```

type: ALL

key和key_len为null

说明整个查询是全表查询没有用到索引。虽然我们创建的两个索引中包括了display 和sort_by，根据最左前缀法则 查询条件中没有包含索引中最左边的列，索引无效。

全表扫描通常会比较耗费情况，但也不是绝对的，有时候虽然有索引MySQL也会选择全表扫描的方式，这是因为优化器分析使用全表扫描比使用索引更耗费时间。



##### 4）创建索引

在三个数据库中对三个订单表创建query_index_0索引，执行下边的脚本：

```Java
create index query_index_0
    on `jzo2o-orders-0`.orders (orders_status asc, user_id asc, display asc, sort_by desc);
create index query_index_0
    on `jzo2o-orders-0`.orders_0 (orders_status asc, user_id asc, display asc, sort_by desc);
create index query_index_0
    on `jzo2o-orders-0`.orders_1 (orders_status asc, user_id asc, display asc, sort_by desc);
create index query_index_0
    on `jzo2o-orders-0`.orders_2 (orders_status asc, user_id asc, display asc, sort_by desc);

create index query_index_0
    on `jzo2o-orders-1`.orders (orders_status asc, user_id asc, display asc, sort_by desc);
create index query_index_0
    on `jzo2o-orders-1`.orders_0 (orders_status asc, user_id asc, display asc, sort_by desc);
create index query_index_0
    on `jzo2o-orders-1`.orders_1 (orders_status asc, user_id asc, display asc, sort_by desc);
create index query_index_0
    on `jzo2o-orders-1`.orders_2 (orders_status asc, user_id asc, display asc, sort_by desc);


create index query_index_0
    on `jzo2o-orders-2`.orders (orders_status asc, user_id asc, display asc, sort_by desc);
create index query_index_0
    on `jzo2o-orders-2`.orders_0 (orders_status asc, user_id asc, display asc, sort_by desc);
create index query_index_0
    on `jzo2o-orders-2`.orders_1 (orders_status asc, user_id asc, display asc, sort_by desc);
create index query_index_0
    on `jzo2o-orders-2`.orders_2 (orders_status asc, user_id asc, display asc, sort_by desc);
```

在三个数据库中对三个订单表创建query_index_1索引，执行下边的脚本：

```Java
create index query_index_1
    on `jzo2o-orders-0`.orders (user_id asc, display asc, sort_by desc);
create index query_index_1
    on `jzo2o-orders-0`.orders_0 (user_id asc, display asc, sort_by desc);
create index query_index_1
    on `jzo2o-orders-0`.orders_1 (user_id asc, display asc, sort_by desc);
create index query_index_1
    on `jzo2o-orders-0`.orders_2 (user_id asc, display asc, sort_by desc);

create index query_index_1
    on `jzo2o-orders-1`.orders (user_id asc, display asc, sort_by desc);
create index query_index_1
    on `jzo2o-orders-1`.orders_0 (user_id asc, display asc, sort_by desc);
create index query_index_1
    on `jzo2o-orders-1`.orders_1 (user_id asc, display asc, sort_by desc);
create index query_index_1
    on `jzo2o-orders-1`.orders_2 (user_id asc, display asc, sort_by desc);


create index query_index_1
    on `jzo2o-orders-2`.orders (user_id asc, display asc, sort_by desc);
create index query_index_1
    on `jzo2o-orders-2`.orders_0 (user_id asc, display asc, sort_by desc);
create index query_index_1
    on `jzo2o-orders-2`.orders_1 (user_id asc, display asc, sort_by desc);
create index query_index_1
    on `jzo2o-orders-2`.orders_2 (user_id asc, display asc, sort_by desc);
```

注意在执行上边脚本时由于创建的索引较多，可能部分索引已存在报错，点击忽略错误让其执行完

![](https://feishu.cn/file/G6hsbX2nUoxQP7x4gUfcUmHfnRh)

也可以登录mysql容器执行：

```Java
docker exec -it mysql /bin/bash
进入容器输入以命令：
 mysql -uroot -pmysql
mysql>在mysql命令提示符下粘贴上边的创建索引命令。
```



##### 5) 小结

什么是聚集索引和非聚集索引？



什么是回表查询？



什么是覆盖索引？



#### 3.2.2 查询订单ID列表

##### 1）优化Service方法

下边根据优化方案优化原来的订单查询方法。

下边是原来的代码：

```Java
/**
 * 滚动分页查询
 *
 * @param currentUserId 当前用户id
 * @param ordersStatus  订单状态，0：待支付，100：派单中，200：待服务，300：服务中，400：待评价，500：订单完成，600：已取消，700：已关闭
 * @param sortBy        排序字段
 * @return 订单列表
 */
@Override
public List<OrderSimpleResDTO> consumerQueryList(Long currentUserId, Integer ordersStatus, Long sortBy) {
    //1.构件查询条件
    LambdaQueryWrapper<Orders> queryWrapper = Wrappers.<Orders>lambdaQuery()
            .eq(ObjectUtils.isNotNull(ordersStatus), Orders::getOrdersStatus, ordersStatus)
            .lt(ObjectUtils.isNotNull(sortBy), Orders::getSortBy, sortBy)
            .eq(Orders::getUserId, currentUserId)
            .eq(Orders::getDisplay, EnableStatusEnum.ENABLE.getStatus());
    Page<Orders> queryPage = new Page<>();
    queryPage.addOrder(OrderItem.desc(SORT_BY));
    queryPage.setSearchCount(false);

    //2.查询订单列表
    Page<Orders> ordersPage = baseMapper.selectPage(queryPage, queryWrapper);
    List<Orders> records = ordersPage.getRecords();
    List<OrderSimpleResDTO> orderSimpleResDTOS = BeanUtil.copyToList(records, OrderSimpleResDTO.class);
    return orderSimpleResDTOS;

}
```

优化后的代码：

```Java
/**
 * 滚动分页查询
 *
 * @param currentUserId 当前用户id
 * @param ordersStatus  订单状态，0：待支付，100：派单中，200：待服务，300：服务中，400：待评价，500：订单完成，600：已取消，700：已关闭
 * @param sortBy        排序字段
 * @return 订单列表
 */
@Override
public List<OrderSimpleResDTO> consumerQueryList(Long currentUserId, Integer ordersStatus, Long sortBy) {
    //构件查询条件
    LambdaQueryWrapper<Orders> queryWrapper = Wrappers.<Orders>lambdaQuery()
            .eq(ObjectUtils.isNotNull(ordersStatus), Orders::getOrdersStatus, ordersStatus)
            .lt(ObjectUtils.isNotNull(sortBy), Orders::getSortBy, sortBy)
            .eq(Orders::getUserId, currentUserId)
            .eq(Orders::getDisplay, EnableStatusEnum.ENABLE.getStatus())
            .select(Orders::getId);//只查询id列
    Page<Orders> queryPage = new Page<>();
    queryPage.addOrder(OrderItem.desc(SORT_BY));
    queryPage.setSearchCount(false);

    //查询订单id列表
    Page<Orders> ordersPage = baseMapper.selectPage(queryPage, queryWrapper);
    if (ObjectUtil.isEmpty(ordersPage.getRecords())) {
        return new ArrayList<>();
    }
    //提取订单id列表
    List<Long> orderIds= CollUtils.getFieldValues(ordersPage.getRecords(), Orders::getId);

    //根据订单id查询订单列表
    List<Orders> ordersList = batchQuery(orderIds);

    List<OrderSimpleResDTO> orderSimpleResDTOS = BeanUtil.copyToList(ordersList, OrderSimpleResDTO.class);
    return orderSimpleResDTOS;

}
```

##### 2）**测试**



1. **测试查询全部订单**

打开小程序进入我的订单列表，查询全部订单，从IDEA控制台日志中获取执行的SQL语句并查看执行计划。

示例：

查询订单ID

![wi6zbzk1yo7vt8x9ubxcavtrnwg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wi6zbzk1yo7vt8x9ubxcavtrnwg.png)
> 图注：图片展示了小程序订单查询优化中测试查询全部订单时IDEA控制台日志中的SQL语句及参数信息。日志显示了两条记录，时间分别为2023 - 11 - 03 15:56:18.862和15:56:19.814，分别记录了订单ID列表查询的SQL语句及其参数，参数为user_id、display和limit值。该图片与上文测试查询全部订单的内容相关，用于说明从IDEA控制台日志中获取执行的SQL语句并查看执行计划的操作示例。

根据订单ID查询订单信息

![fgxjb4esjo9tctx6d2nct8rvnoe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/fgxjb4esjo9tctx6d2nct8rvnoe.png)
> 图注：图片展示了优化后Service方法中查询订单ID列表时的IDEA控制台日志。日志记录了两次执行过程，时间分别为2023 - 11 - 03 15:56:20.263和2023 - 11 - 03 15:56:20.363。执行的SQL语句为从orders表中查询id、user_id等字段，条件是id在指定的订单ID列表中。参数部分显示了具体的订单ID值。该图片与上文测试查询全部订单的内容相关，用于展示执行的SQL语句及参数情况。

复制日志中的SQL，使用explain获取执行计划

![ag3jbdnlwovfmtxecvvc7k8in3f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ag3jbdnlwovfmtxecvvc7k8in3f.png)
> 图注：图片展示的是IDEA控制台日志中查询订单ID的SQL语句及其执行计划。SQL语句为“SELECT id FROM orders_0 WHERE (user_id = 1716346406098296832 AND display = 1) ORDER BY sort_by DESC LIMIT 10”，执行计划中显示使用了覆盖索引，具体为“Using index”，且rows为11，filtered为100，Extra为“Using index”。该图片与上文测试查询全部订单的内容相关，用于验证优化后查询订单ID列表时是否使用了覆盖索引。

从输出可以看出使用了覆盖索引。

第二个SQL是根据id主键查询详细信息，如下：

```Java
explain
SELECT id,
       user_id,
       serve_type_id,
       serve_type_name,
       serve_item_id,
       serve_item_name,
       serve_item_img,
       unit,
       serve_id,
       orders_status,
       pay_status,
       refund_status,
       price,
       pur_num,
       total_amount,
       real_pay_amount,
       discount_amount,
       city_code,
       serve_address,
       contacts_phone,
       contacts_name,
       serve_start_time,
       lon,
       lat,
       pay_time,
       evaluation_time,
       display,
       sort_by,
       create_time,
       update_time,
       trading_order_no,
       transaction_id,
       refund_no,
       refund_id,
       trading_channel
FROM orders_0

WHERE (id IN (2311030000000000029, 2311020000000000028, 2311020000000000027, 2311020000000000026, 2311020000000000023, 2311020000000000025, 2311020000000000024, 2311020000000000022, 2311020000000000021, 2310280000000000005))
```

输出：

```Java
[
  {
    "id": 1,
    "select_type": "SIMPLE",
    "table": "orders_0",
    "partitions": null,
    "type": "ALL",
    "possible_keys": "PRIMARY",
    "key": null,
    "key_len": null,
    "ref": null,
    "rows": 10,
    "filtered": 100,
    "Extra": "Using where"
  }
]
```

从key、key_len可以看出并没有使用索引，possible_keys显示可能会使用主键索引（聚集索引），但最终没有使用索引。

这是因为MySQL器判断上边的SQL进行全表扫描要比使用索引效率高。

如果我们去掉一些id IN ()中的参数，比如：我们只留三个ID，如下：

```Java
explain
SELECT id,
       user_id,
       serve_type_id,
       serve_type_name,
       serve_item_id,
       serve_item_name,
       serve_item_img,
       unit,
       serve_id,
       orders_status,
       pay_status,
       refund_status,
       price,
       pur_num,
       total_amount,
       real_pay_amount,
       discount_amount,
       city_code,
       serve_address,
       contacts_phone,
       contacts_name,
       serve_start_time,
       lon,
       lat,
       pay_time,
       evaluation_time,
       display,
       sort_by,
       create_time,
       update_time,
       trading_order_no,
       transaction_id,
       refund_no,
       refund_id,
       trading_channel
FROM orders_0

WHERE (id IN (2311030000000000029, 2311020000000000028, 2311020000000000027))
```

输出：

```Java
[
  {
    "id": 1,
    "select_type": "SIMPLE",
    "table": "orders_0",
    "partitions": null,
    "type": "range",
    "possible_keys": "PRIMARY",
    "key": "PRIMARY",
    "key_len": "8",
    "ref": null,
    "rows": 3,
    "filtered": 100,
    "Extra": "Using where"
  }
]
```

从输出可以看出此时使用了聚集索引。



1. **查询待支付订单**

参考上边的方法自行测试。



1. **向下滚动屏幕查询出现重复记录**

作为今天的作业进行bug修复。



##### 3）小结

如何知道一个SQL有没有使用索引？



## 4 冷热分离

### 4.1 冷热分离方案

目标：理解冷热分离的方案。

#### 1）冷热分离需求

随着时间的推移，订单数据会逐渐增加，虽然对订单数据库进行分库分表，但是考虑用户对已完成的历史订单的操作热度远远低于未完成订单，为了提高系统的性能我们对订单数据进行冷热分离。

冷数据是指那些很少被访问或者对访问延迟要求较低的数据，而热数据则是经常被访问、对访问延迟要求高的数据。

订单的冷热分离是指根据订单的特性和需求，将订单数据划分为冷数据和热数据，以便更有效地管理和优化数据存储、检索和处理的方式。这种分离有助于提高系统的性能和降低存储成本

根据需求，订单完成15日后将不允许对订单进行取消，我们可以将完成超过15日的订单归档到历史订单库，其它订单数据在订单数据库。订单数据库存储的是热数据，历史订单数据库存储冷数据。

<whiteboard token="Kc6lwLaEYhKDH2bYTegcbOuBnZf"></whiteboard>

#### 2）历史数据库选型

历史订单（冷数据）存储到哪里呢？

通常通过以下方式存储冷数据：

1. 将冷数据从主要的数据库中归档到较为廉价的存储介质，例如使用固态硬盘（SSD）存储热数据，而将冷数据存储在传统磁盘或云存储中，也可以使用低成本的云存储服务。
2. 使用数据库分区或分片技术，将冷数据和热数据存储在不同的物理或逻辑分区，
3. 使用云服务进行存储，比如：阿里云的对象存储OSS，腾讯云对象存储COS，这些都是比较廉价的。
4. 如果对冷数据有分析需求就需要将冷数据存储到HDFS文件系统中，方便大数据系统进行分析。
5. 将冷数据存储到分布式数据库中。

现在市面上也有很多的分布式数据库，分布式数据库是指将数据库系统中的数据存储和处理分布在多个计算机节点上，这些节点可以位于不同的物理位置或在同一物理位置上。分布式数据库旨在通过分布数据和查询负载，提高系统的可伸缩性、性能和容错性。

![phenbmz4fozozcxefi5cn8sznwf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/phenbmz4fozozcxefi5cn8sznwf.png)
> 图注：图片展示了分布式数据库中节点间的信任边界关系。图中有四个节点，分别为Node A、Node B、Node C、Node D，它们通过紫色线条相连，形成网络结构。图中还标注了“Trust Boundary”（信任边界）和“Access Control Data Validation”（访问控制数据验证），表明节点间存在信任边界，且需进行访问控制数据验证。图下方有文字“'We trust each other' 'It's us against the world!'”，强调节点间互信但需抵御外部威胁。此图与文档中介绍分布式数据库及阿里云PolarDB的内容相关，直观呈现了分布式数据库中节点间的关系与信任边界。

常见的分布式数据库：

NoSQL数据库：**Apache Cassandra、HBase、MongoDB。**

还有一些国内常用的分布式数据库：

**阿里云 PolarDB：**

阿里云的PolarDB是一种支持MySQL和PostgreSQL的分布式关系型数据库服务。它具有高性能、可伸缩性和自动容灾的特性。

**腾讯云 TDSQL-C（TiDB）：**

 TDSQL-C是腾讯云推出的一种云原生分布式数据库服务，基于TiDB开源项目，支持水平扩展、强一致性和分布式事务。



![xipyberbeorw2pxhlevcdlybnjd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/xipyberbeorw2pxhlevcdlybnjd.png)
> 图注：图片展示了HTAP系统架构，分为OLTP系统、OLAP系统和ETL三层。OLTP系统包括Transaction Service Interface、Transaction Data Storage和Meta Data Storage，用于处理事务数据。OLAP系统包含Data Extraction Layer、OLAP Cube和Analytical Layered Data Stream，进行多维分析。ETL层通过Extract、Transform、Load将数据从OLTP系统提取、转换后加载到OLAP系统。此图与文档中介绍HTAP系统用于实时分析操作数据的内容相契合，直观呈现其架构。

TiDB是一个开源的分布式数据库，它支持SQL语法，并且兼容MySQL协议，同时支持事务处理（OLTP）和分析处理（OLAP）的工作负载。



这么多数据库该如何选择呢？

冷数据是不常用的数据，首先基于满足需求为前提考虑用廉价的成本去实现。

我们的订单数据，按50万用户规模估算订单数据量有多少？

AI: 我们的电商项目有50万用户规模，请估算出订单数据量的规模

> 估算一个电商项目的订单数据量，我们需要考虑几个因素，包括但不限于用户的活跃度、平均每个用户的订单频率、订单的平均大小（包含多少商品项）、以及数据存储的需求（比如保留多久的历史记录）。这里给出一个大致的估算方法：
> 
> 假设条件
> 
> 1. 用户活跃度：假设50万用户中有20%是活跃用户（这个比例可以根据实际情况调整）。
> 2. 订单频率：假设每个活跃用户每月至少下一次订单。
> 3. 订单大小：假设每笔订单平均包含3个商品项。
> 4. 存储需求：假设需要保留一年的历史记录。
> 
> 计算
> 
> 1. 活跃用户数：50万 \* 20% = 10万
> 2. 每月订单数：10万 \* 1次/月 = 10万
> 3. 每年订单数：10万 \* 12个月 = 120万
> 4. 订单总大小：120万 \* 3商品项/订单 = 360万商品项
> 
> 数据量估算
> 
> 为了估算订单数据量，我们需要知道每个订单数据条目大约会占用多少空间。一个订单可能包含以下信息：
> 
> - 订单ID
> - 用户ID
> - 商品列表（每个商品的ID、数量、价格等）
> - 收货地址
> - 订单状态
> - 创建时间和更新时间
> 
> 假设每个订单数据条目大约需要占用1KB的空间（实际大小可能会根据具体字段和数据类型有所不同），那么：
> 
> - 每年产生的新数据量约为：360万 \* 1KB ≈ 3.5GB



如果每年有20%的用户增量，五年后的订单量将是第一年订单量的大约两倍多，订单表的数据量在300万以内。

![eyfwbvgmzo4gysxidozcgwb1noc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/eyfwbvgmzo4gysxidozcgwb1noc.png)
> 图注：这张图片是一张数据统计表格，清晰呈现了系统冷热分离方案中提及的五年内的相关数据。表格包含年份、用户基数（单位为万）、活跃用户数（单位为万）、订单量（单位为万）四列内容，具体数据按年份依次对应：第1年用户基数50万、活跃用户数10万、订单量120万；第5年用户基数达103.68万，活跃用户数20.736万，订单量增长至248.832万。表格直观体现了随年份递增的用户与订单增长情况，可印证文档中“五年后的订单量将是第一年订单量的大约两倍多，订单表的数据量在300万以内”的相关表述。

基于年百万量的数据没有必要使用基于大数据平台的分布式数据库，只有当年数据量上千万甚至上亿时才会考虑使用大数据平台。

对于年百万级别的数据量，传统的关系型数据库如MySQL可以通过分库分表等手段来有效应对，使用成熟的、经过验证的技术栈如MySQL，可以简化系统架构，降低运维复杂度和成本。

**所以，根据本项目的实际需求我们选用对MySQL分库分表存储历史订单数据。**

#### 3）分库分表方案

要确定历史订单数据库的分库分表方案需要先明确统计的需求。

**需求1：根据时间区间统计**

对历史数据要进行统计分析，最终查询统计报表最多是以年为单位进行查询。

下图中用户只能查询一个年度的统计数据，这个需求在很多银行也存在，查询历史账单不能跨年查询。

![iuqobuh1zoixhwxlurpcqlsdnjf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/iuqobuh1zoixhwxlurpcqlsdnjf.png)
> 图注：图片展示的是云岚到家平台工作台 - 概览仪表盘界面。界面左侧有工作台、订单管理、评价管理、服务管理、服务人员管理等导航栏。右侧“经营分析”区域显示了2023年11月24日至2023年12月24日的日期范围，下方有订单分析和营业分析板块，分别列出有效订单数、取消订单数、关闭订单数、实付单均价、有效订单总额等统计指标。该图与上下文介绍的订单趋势图需求相关，用于展示统计时间区间内每天的下单总数等数据。

统计指标说明：

- 有效订单数：在统计时间内，订单状态为已完成的订单数。（订单被取消、退款、关闭状态均不属于有效订单）
- 取消订单数：在统计时间内，提交订单后被取消订总量
- 关闭订单数：在统计时间内，提交订单后被关闭订总量
- 有效订单总额：在统计时间内，有效订单的总订单交易总额
- 实付单均价：在统计时间内，平均每单实际支付额（不包含失效订单）

实付单均价=有效订单总额/有效订单数

**需求2：订单趋势图**

如果统计时间区间 大于1天，订单趋势显示时间区间内每天的下单总数，查询按天统计表的数据即可。

如果统计时间区间小于等于1天，订单趋势显示当天每小时的下单总数，查询按小时统计表的数据即可。

![cfhvbfwxwosgvlxb6fgcrah8nxb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/cfhvbfwxwosgvlxb6fgcrah8nxb.png)
> 图注：图片展示的是订单分析界面。上方显示取消订单数5笔、关闭订单数400笔、实付单均价125元、有效订单总额1,190,625元。订单趋势图呈现2023年9月1日至9月30日每天的下单总数，以折线图形式显示。用户分析部分，用户转化漏斗图显示用户数2000人，下单转化率32%，付费转化率72%；用户平均支付笔数图以折线图形式呈现2022年1月到2023年9月的用户平均支付笔数。该图与上下文介绍的订单趋势图需求对应，直观呈现相关数据。

##### 分库分表方案

对于统计分析的需求清楚后当前需要做的是分析存储统计原始数据的问题，暂时不考虑统计实现问题，先把要统计的原始数据存储到历史订单库。

5年内的年订单规模在300万内，我们可以每年创建一个历史订单表，也就是按年进行分表。

如何使用ShardingSphere-JDBC实现按年分表？

大家思考这个问题并稍后尝试实现。



#### 3）冷热分离方案

##### 冷热分离流程

如何进行冷热分离？

本项目将完成15日的订单迁移到历史订单数据库，通过Canal+MQ技术将订单（完成、取消、关闭状态）迁移到历史订单数据库，在历史订单服务对订单数据进行统计分析。

方案如下：

<whiteboard token="CD1zwt3fJhlg99bSXKIcDaJenhh"></whiteboard>

流程如下：

**订单管理服务：**

- 当订单完成，取消、关闭时将订单信息写入同步表。
- 定时任务删除同步表已迁移的数据。

**Canal+MQ:** 

- 通过Canal+MQ将同步表的订单数据同步到历史订单数据库的同步表中，具体过程如下：     Canal读取binlog将写入同步表的数据写入MQ。

  历史订单服务监听MQ，获得同步的订单数据，写入历史订单数据库的同步表中。

**历史订单服务：**

- 启动定时任务，每天凌晨将昨天从0点到昨天24点之间完成15日后的订单信息从同步表迁移到历史订单表。
- 每次迁移完成将已经迁移完成的历史订单从同步表删除。



##### 数据流

<whiteboard token="UUWNwDgRNhzPc8bL3nccab0rnjc"></whiteboard>



#### 4）小结

订单的冷热分离是怎么做的？





### 4.2 订单数据冷热分离



#### 4.2.1 订单同步

根据冷热分离方案，首先将完成、取消、关闭的订单同步到历史订单库。

本节目标：将完成、取消、关闭的订单同步到历史订单数据库。

##### 1）创建历史订单数据库

创建jzo2o-orders-history数据库，导入jzo2o-orders-history.sql脚本：

![bzf6bccmbowevxxgzydcvqebnme](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/bzf6bccmbowevxxgzydcvqebnme.png)
> 图注：图片展示了jzo2o - orders - history数据库下的tables文件夹内容。其中包含history_orders、history_orders_serve、history_orders_serve_sync、history_orders_sync、stat_day、stat_hour六个表。该图片与上文创建jzo2o - orders - history数据库，导入jzo2o - orders - history . sql脚本的内容相关，直观呈现了脚本导入后数据库中表的结构情况，帮助理解历史订单数据库的表组成。

history_orders：用于存储历史订单数据。

history_orders_serve: 用于存储历史服务单数据。

history_orders_serve_sync：用于存储待迁移的已完成的服务单数据。

history_orders_sync： 用于存储待迁移的订单数据（包括已完成，取消、关闭的订单）。

stat_day：存储按天统计数据，统计分析模块使用

stat_hour：存储按小时统计数据，统计分析模块使用



##### 2）订单同步

根据冷热分离方案，当订单完成，取消、关闭时将订单信息写入订单数据库的同步表，再通过Canal+MQ同步到历史订单数据库的同步表，Canal+MQ同步的代码我们之前做过，这里我们阅读代码理解订单同步的过程。

在订单数据库jzo2o-orders-0、jzo2o-orders-1、jzo2o-orders-2 存在订单同步表和服务单的同步表：

![lnfsbs9l0ogbs6xcf14clqklnse](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/lnfsbs9l0ogbs6xcf14clqklnse.png)
> 图注：这张图片展示了两个同步表的名称，分别为history_orders_serve_sync与history_orders_sync，对应订单数据库中的订单同步表与服务单同步表。该图片的内容与文档中提到的订单数据库jzo2o-orders-0、jzo2o-orders-1、jzo2o-orders-2中存在订单同步表和服务单同步表的描述相呼应，直观呈现了文档所述的两类同步表的具体名称。

1. **当订单完成、取消、关闭时向同步表写入记录。**

此部分的代码在订单状态机OrderStateMachine 类中实现，阅读下边的代码：

```JSON
package com.jzo2o.orders.base.config;

import cn.hutool.core.util.*;
import com.jzo2o.common.utils.CollUtils;
import com.jzo2o.common.utils.ObjectUtils;
import com.jzo2o.orders.base.enums.OrderStatusEnum;
import com.jzo2o.orders.base.model.dto.OrderSnapshotDTO;
import com.jzo2o.orders.base.service.IHistoryOrdersSyncCommonService;
import com.jzo2o.redis.helper.CacheHelper;
import com.jzo2o.statemachine.AbstractStateMachine;
import com.jzo2o.statemachine.persist.StateMachinePersister;
import com.jzo2o.statemachine.snapshot.BizSnapshotService;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;

import static com.jzo2o.orders.base.constants.RedisConstants.RedisKey.ORDERS;

/**
 * 订单状态机
 *
 * @author itcast
 * @create 2023/8/4 11:20
 **/
@Component
public class OrderStateMachine extends AbstractStateMachine<OrderSnapshotDTO> {
...
@Resource
private IHistoryOrdersSyncCommonService historyOrdersSyncService;

@Override
protected void postProcessor(OrderSnapshotDTO orderSnapshotDTO) {
...
    /***************************完成、关闭、取消订单写历史订单同步表*******************************/
    //取出订单的新状态
    Integer ordersStatus = orderSnapshotDTO.getOrdersStatus();
    if(OrderStatusEnum.FINISHED.getStatus().equals(ordersStatus) ||
            OrderStatusEnum.CLOSED.getStatus().equals(ordersStatus) ||
               OrderStatusEnum.CANCELED.getStatus().equals(ordersStatus) ){
        historyOrdersSyncService.writeHistorySync(orderSnapshotDTO.getId());
    }

}

```

通过historyOrdersSyncService.writeHistorySync(orderSnapshotDTO.getId());方法将订单数据同步到上边两张同步表当中。

阅读此方法的源代码，在历史订单同步表的sort_time字段中记录订单完成后15天的时间点，如下：

```Java
// 排序时间(15天后数据无法再次修改，并迁移到历史订单中)
historyOrdersSync.setSortTime(orders.getUpdateTime().plusDays(15));
```



1. **Canal+MQ将同步表数据同步到历史订单数据库的同步表**

首先进入RabbitMQ，配置exchange.canal-jzo2o交换机绑定下边红框的队列。

![g8wublpxvo3q3xxcecnc81ognbe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/g8wublpxvo3q3xxcecnc81ognbe.png)
> 图注：该图片展示了RabbitMQ的exchange配置界面，对应订单数据冷热分离中第二步的操作要求。界面中显示需配置exchange.canal-jzo2o交换机的绑定队列，其中红框标注的“canal-mq-jzo2o-orders-history”为订单同步队列，“canal-mq-jzo2o-orders-serve-history”为服务同步队列，其余多个队列目前均为已解绑状态，明确呈现了需绑定的关键队列信息。

找到历史订单服务中的数据同步类：

![f6w7bogqxornkqx8x5pcudxsnuf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/f6w7bogqxornkqx8x5pcudxsnuf.png)
> 图注：这张图片展示了项目“jzo2o-orders-history”的代码目录结构，在“main/java/com/jzo2o/orders/history”路径下，存在controller与handler两个子目录；其中controller目录下包含agency、consumer、inner、operation、worker等子目录，handler目录下则标注了两个核心类，分别是HistoryOrdersServeSyncHandler与HistoryOrdersSyncHandler，该结构对应文档中提及的历史订单服务数据同步类的相关内容，与订单同步的业务操作直接关联。

代码如下：

```JSON
package com.jzo2o.orders.history.handler;

import com.jzo2o.canal.listeners.AbstractCanalRabbitMqMsgListener;
import com.jzo2o.orders.history.model.domain.HistoryOrdersSync;
import com.jzo2o.orders.history.service.IHistoryOrdersSyncService;
import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.util.List;

@Component
public class HistoryOrdersSyncHandler extends AbstractCanalRabbitMqMsgListener<HistoryOrdersSync> {

    @Resource
    private IHistoryOrdersSyncService historyOrdersSyncService;

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "canal-mq-jzo2o-orders-history", durable = "true"),
            exchange = @Exchange(name = "exchange.canal-jzo2o", type = ExchangeTypes.TOPIC),
            key = "canal-mq-jzo2o-orders-history"),
            concurrency = "1"
    )
    public void onMessage(Message message) throws Exception {
        parseMsg(message);
    }

    @Override
    public void batchSave(List<HistoryOrdersSync> historyOrdersSyncs) {
        historyOrdersSyncService.saveOrUpdateBatch(historyOrdersSyncs);
    }

    @Override
    public void batchDelete(List<Long> ids) {

    }
}
```

通过historyOrdersSyncService.saveOrUpdateBatch(historyOrdersSyncs);将订单数据写入历史订单数据库的同步表history_orders_sync中。

![imnbbrvj6oy41ix7kpmcyguzndb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/imnbbrvj6oy41ix7kpmcyguzndb.png)
> 图注：图片展示了jzo2o-orders-history数据库中tables下的表结构。其中，history_orders_sync表被红色框突出显示，其下有history_orders、history_orders_serve、history_orders_serve_sync、stat_day、stat_hour等表。该图片与上文介绍的订单同步内容相关，对应于“Canal+MQ将同步表数据同步到历史订单数据库的同步表”这一步骤，直观呈现了历史订单数据库中同步表的分布情况。

服务单同步类HistoryOrdersServeSyncHandler请自行查询源码。



##### 3）测试订单同步

目前我们还没有开发派单功能所以无法完成一个订单，我们只能通过取消订单功能将订单取消后添加到同步表中。

启动canal:

首先修改配置文件：vi canal.properties ，（注意，需要将原来黑马商场的配置注释掉）内容如下：

```Java
##################################################
#########                   RabbitMQ         #############
##################################################
rabbitmq.host = 192.168.101.68
rabbitmq.virtual.host = /xzb
rabbitmq.exchange = exchange.canal-jzo2o
rabbitmq.username = xzb
rabbitmq.password = xzb
rabbitmq.deliveryMode = 2

```

修改配置文件：

```Java
canal.instance.filter.regex=jzo2o-orders-1\\.orders_dispatch,jzo2o-orders-1\\.orders_seize,jzo2o-foundations\\.serve_sync,jzo2o-customer-all\\.serve_provider_sync,jzo2o-orders-1\\.serve_provider_sync,jzo2o-orders-1\\.history_orders_sync,jzo2o-orders-1\\.history_orders_serve_sync,jzo2o-market\\.activity
```

```Java
canal.mq.dynamicTopic=canal-mq-jzo2o-orders-dispatch:jzo2o-orders-1\\.orders_dispatch,canal-mq-jzo2o-orders-seize:jzo2o-orders-1\\.orders_seize,canal-mq-jzo2o-foundations:jzo2o-foundations\\.serve_sync,canal-mq-jzo2o-customer-provider:jzo2o-customer-all\\.serve_provider_sync,canal-mq-jzo2o-orders-provider:jzo2o-orders-1\\.serve_provider_sync,canal-mq-jzo2o-orders-serve-history:jzo2o-orders-1\\.history_orders_serve_sync,canal-mq-jzo2o-orders-history:jzo2o-orders-1\\.history_orders_sync,canal-mq-jzo2o-market-resource:jzo2o-market\\.activity
```

屏蔽nacos下jzo2o-orders-history.yaml中的内容，如下图：

![wmomb0w0poqhiqxy1eoc8jxwnzb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wmomb0w0poqhiqxy1eoc8jxwnzb.png)
> 图注：这张图片展示了Nacos配置页面中jzo2o-orders-history.yaml的内容，该内容是文档中提及需要屏蔽的对象。页面包含Data ID、Group的配置项，当前已选中YAML的配置模式，配置内容区域显示了与订单数据相关的ShardingSphere及MyBatis-Plus配置，涉及spring、mybatis-plus配置字段，还有shardingsphere的数据源、分页、分表策略等配置信息，该配置是订单数据冷热分离流程里需调整的关键配置内容。

测试流程如下：

启动网关

启动jzo2o-customer

启动jzo2o-fundations

启动jzo2o-publics

启动jzo2o-orders-manager

启动jzo2o-orders-history

启动服务端（前端工程）

用户下单

用户取消订单

取消订单后向同步表添加成功：

![eh2ubqdglor2t6xjvhrcsfnknud](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/eh2ubqdglor2t6xjvhrcsfnknud.png)
> 图注：图片展示了jzo2o-orders-1数据库中tables下的部分表结构。其中，history_orders_sync和history_orders_serve_sync两个表被红色框突出显示。这与文档中测试订单同步的内容相关，表明在测试订单同步时，可通过canal将完成订单同步到历史库，此图片直观呈现了历史订单同步表在数据库中的位置，辅助理解测试流程中相关表的使用。

通过canal将完成订单同步到历史库

![bn06baemdogjl3xtzqncxvhbnvc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/bn06baemdogjl3xtzqncxvhbnvc.png)
> 图注：图片展示了数据库中jzo2o-orders-history表的结构。在tables文件夹下，有history_orders、history_orders_serve、history_orders_serve_sync、history_orders_sync四个表。其中，history_orders_serve_sync和history_orders_sync表被高亮显示。该图片与文档中测试订单同步的内容相关，用于说明在测试订单同步时，可通过canal将完成订单同步到历史库，此图展示了历史库中相关表的结构情况。

测试时注意：保证Canal工作正常。

数据不同步的问题参考：“配置ES索引同步环境v1.0” 进行处理。



##### 4) 小结

本节将完成、取消、关闭的订单使用Canal+MQ同步到历史订单数据库，流程如下：

1. 订单完成、取消、关闭后在写入订单同步表。
2. Canal读取同步表的binlog，解析数据发送至MQ
3. 历史订单服务监听MQ，获取到订单信息后写入同步表（history_orders_sync和history_orders_serve_sync表）。



#### 4.2.2 订单冷热分离



根据需求，订单完成15日后迁移到历史订单表和历史服务单表，在历史订单同步表的sort_time字段中记录订单完成后15天的时间点 ，所以根据sort_time字段查询待迁移的数据，迁移任务是每天凌晨将昨天达到订单完成15天的数据迁移到历史订单表。



##### 1）定义mapper

1. 首先编写SQL查询出未迁移的记录，即查询昨天到达订单完成15天的且未迁移的数据。

```JSON
select hos.*
 from history_orders_sync hos
          LEFT JOIN history_orders ho on hos.id = ho.id
 where hos.sort_time >= '2024-12-05 0:0:0'
   and hos.sort_time <= '2024-12-05 23:59:59'
   and ho.id is null
 limit 0,1000
```

1. 定义mapper

编写mapper映射文件：

```JSON
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jzo2o.orders.history.mapper.HistoryOrdersMapper">

    <select id="migrate" parameterType="java.util.Map" resultType="com.jzo2o.orders.history.model.domain.HistoryOrders">
        select distinct hos.*
        from history_orders_sync hos
                 LEFT JOIN history_orders ho on hos.id=ho.id
        where <![CDATA[ hos.sort_time >= #{yesterDayStartTime}]]> and <![CDATA[ hos.sort_time <= #{yesterDayEndTime} ]]>
        and ho.id is null
            limit #{offset},#{perNum}
    </select>
</mapper>
```

定义mapper接口：

```JSON
package com.jzo2o.orders.history.mapper;

import com.jzo2o.orders.history.model.domain.HistoryOrders;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import org.apache.ibatis.annotations.Param;

import java.time.LocalDateTime;

/**
 * <p>
 * 订单表 Mapper 接口
 * </p>
 *
 * @author itcast
 * @since 2023-09-11
 */
public interface HistoryOrdersMapper extends BaseMapper<HistoryOrders> {

    List<HistoryOrders> migrate(@Param("yesterDayStartTime") LocalDateTime yesterDayStartTime,
                                @Param("yesterDayEndTime") LocalDateTime yesterDayEndTime,
                                @Param("offset") Integer offset,
                                @Param("perNum") Integer perNum);
}
```



##### 2）定义service

```JSON
package com.jzo2o.orders.history.service;

import com.jzo2o.common.model.PageResult;
import com.jzo2o.orders.history.model.domain.HistoryOrders;
import com.baomidou.mybatisplus.extension.service.IService;
import com.jzo2o.orders.history.model.dto.request.HistoryOrdersListQueryReqDTO;
import com.jzo2o.orders.history.model.dto.request.HistoryOrdersPageQueryReqDTO;
import com.jzo2o.orders.history.model.dto.response.HistoryOrdersDetailResDTO;
import com.jzo2o.orders.history.model.dto.response.HistoryOrdersListResDTO;
import com.jzo2o.orders.history.model.dto.response.HistoryOrdersPageResDTO;

import java.util.List;

/**
 * <p>
 * 订单表 服务类
 * </p>
 *
 * @author itcast
 * @since 2023-09-11
 */
public interface IHistoryOrdersService extends IService<HistoryOrders> {
/**
 * 迁移历史订单
 */
void migrate();

/**
 * 删除已经迁移完成的订单
 */
void deleteMigrated();

...
```

实现类：

```JSON
@Override
public void migrate() {
    log.debug("历史订单迁移开始...");
    // 查询时间开始坐标
    int offset = 0;
    int perNum = 1000;
    // 昨天开始时间
    LocalDateTime yesterDayStartTime = DateUtils.getDayStartTime(DateUtils.now().minusDays(1));
    // 昨天结束时间
    LocalDateTime yesterDayEndTime = DateUtils.getDayEndTime(DateUtils.now().minusDays(1));

    // 统计迁移数据数量
    Long total = historyOrdersSyncService.countBySortTime(yesterDayStartTime, yesterDayEndTime);
    if(total <= 0){
        return;
    }

    // 分批次迁移
    while (offset < total) {
        List<HistoryOrders> migrate = baseMapper.migrate(yesterDayStartTime, yesterDayEndTime, offset, perNum);
        saveBatch(migrate);
        offset += perNum;
    }
    log.debug("历史订单迁移结束。");

}

@Override
public void deleteMigrated() {
    // 昨天开始时间
    LocalDateTime yesterDayStartTime = DateUtils.getDayStartTime(DateUtils.now().minusDays(1));
    // 昨天结束时间
    LocalDateTime yesterDayEndTime = DateUtils.getDayEndTime(DateUtils.now().minusDays(1));

    // 1.校验是否可以删除已迁移订单
    // 查询即将删除的数据数量
    Long totalOfDelete = historyOrdersSyncService.countBySortTime(yesterDayStartTime, yesterDayEndTime);
    if(totalOfDelete <= 0) {
        log.debug("无迁移的订单数据需要删除");
        return;
    }
    // 查询已经迁移的数量
    Long totalMigrated = lambdaQuery().between(HistoryOrders::getSortTime, yesterDayStartTime, yesterDayEndTime)
            .count();
    if (NumberUtils.null2Zero(totalMigrated) <= 0 || totalOfDelete > totalMigrated) {
        log.error("订单未完全迁移，同步数据删除失败");
        return;
    }

    // 2.删除已经迁移数据
    historyOrdersSyncService.deleteBySortTime(yesterDayStartTime, yesterDayEndTime);
}
```

##### 3）定时任务

阅读下边的代码：

```JSON
package com.jzo2o.orders.history.handler;

import com.jzo2o.orders.history.service.IHistoryOrdersServeService;
import com.jzo2o.orders.history.service.IHistoryOrdersService;
import com.jzo2o.orders.history.service.IStatDayService;
import com.jzo2o.orders.history.service.IStatHourService;
import com.xxl.job.core.handler.annotation.XxlJob;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;

@Component
public class XxlJobHandler {

/**
     * 迁移HistoryOrdersSync同步表的数据到HistoryOrders历史订单表
     * 迁移HistoryOrdersServeSync同步表的数据到HistoryOrdersServe历史订单表
     * 规则：
     * 每天凌晨执行，迁移截止到昨日已完成15天的订单到历史订单
     *
     */
    @XxlJob("migrateHistoryOrders")
    public void migrateHistoryOrders(){
        //迁移HistoryOrdersSync同步表的数据到HistoryOrders历史订单表
        historyOrdersService.migrate();
        //删除迁移完成的数据
        historyOrdersService.deleteMigrated();
    
    }
    ...
```





##### 4）测试订单冷热分离



添加执行器：

![hmtmbqkyvoh989xmexncvyclnph](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/hmtmbqkyvoh989xmexncvyclnph.png)
> 图注：这张图片展示了订单冷热分离相关的系统操作界面，界面顶部设有AppName搜索栏，已输入“jzo2o-orders-history”，搭配有搜索和新增按钮，还包含每页显示10条记录的设置选项。下方的表格中，已存在一条数据，其AppName为“jzo2o-orders-history”，名称是“历史订单”，注册方式为自动注册，OnLine机器地址处提供“查看(1)”的操作入口，操作栏设有可展开的下拉“操作”按钮，该界面用于订单冷热分离流程里的测试环节，对应文档中“添加执行器”这一步骤的操作场景。

添加任务并启动任务：

![ztxkb1hepofmonxcazwcg6b0nwe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ztxkb1hepofmonxcazwcg6b0nwe.png)
> 图注：图片展示的是订单冷热分离中定时任务的添加界面。基础配置部分，执行器为历史订单，负责人是xxljob。调度配置中，调度类型为CRON，Cron为0/10 * * * * ? 。任务配置里，运行模式为BEAN，JobHandler为migrateHistoryOrders。高级配置中，路由策略为第一个，调度过期策略为忽略，任务超时时间为0。该图片与上文提到的实现定时任务相呼应，直观呈现了任务添加时各配置项的具体设置情况。



订单完成15个工作日后迁移到历史订单。

在测试时修改history_orders_sync和history_orders_serve_sync中sort_time字段值小于等于昨天日期。

预期结果：

数据由 history_orders_sync和history_orders_serve_sync迁移到history_orders和history_orders_serve表。





## 5 统计模块

### 5.1 订单统计方案

#### 1) 经营看板

本项目在运营端工作台页面展示系统经营看板，内容包括订单分析、用户分析等，如下图：

经营看板是一种用于实时监控关键业务指标的工具，它不仅帮助团队保持敏捷、透明和高效，还促进了团队的协作和创新。

![zwmnb9v4tofwczxwhykcemfyndf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/zwmnb9v4tofwczxwhykcemfyndf.png)
> 图注：图片展示了系统经营看板中订单分析和用户分析的内容。订单分析部分呈现了有效订单数、取消订单数、关闭订单数、实付客单价、有效订单总额等数据，还展示了订单数趋势图。用户分析部分有用户转化漏斗图，显示用户数、下单转化率、付费转化率等；以及用户平均支付笔数折线图，呈现2021年3月至2023年9月的数据变化。该图与上下文紧密相关，直观呈现了订单和用户相关数据，帮助团队监控业务指标。



软件中经营看板的应用场景非常多，下图显示了公司销售分析周报：

![ahkvbfbgrob2onx92uucc4usnuv](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ahkvbfbgrob2onx92uucc4usnuv.png)
> 图注：图片是XXX公司销售经营分析周报，展示了2020年1月第2周的销售数据。左侧是销售达成率及业务员销售排名，如李芳销售达成为8.08%。中间是整体销售情况，如销售总额326,604元，销售利润245,597元，销售达成率90.72%等。下方是2020年1月第2周销售趋势折线图。右侧是产品销售排名和区域销售对比，如MP4销售32,662元，天津销售78,588元等。该图直观呈现了销售数据，与上下文介绍的经营看板功能相契合。

双11销售统计：

![l83obyzkwomn6vxlrk4cyrxrnod](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/l83obyzkwomn6vxlrk4cyrxrnod.png)
> 图注：图片展示的是2021年天猫双11全球狂欢季的经营看板。上方显示成交金额为1,000,000.00元，同比增速为43.09%。中间部分有多个数据指标，如浏览量、访客数、访客数转化、访客数转化等，部分指标数值为100,000。底部有部分商品的销售数据，如商品名称、销售金额、销售数量等。该图与上下文介绍的经营看板功能相呼应，直观呈现了双11期间的经营数据情况。

通过学习本节的内容掌握看板功能 的开发方法。

#### 2）需求分析

下边梳理本项目运营端经营看板的功能。

首先选择一个时间区间（不能大于365天），统计在此时间区间内的订单数据，只统计已取消、已关闭、已完成的订单数据。

![lhorbwqazo2nddxrdgeclxi1nhc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/lhorbwqazo2nddxrdgeclxi1nhc.png)
> 图注：这是云岚到家管理后台工作台页面，当前显示经营分析相关的内容，其中经营分析板块包含时间选择区域，红色框内选中的统计时间区间为2023-11-24至2023-12-24。页面设有多个快捷入口，分别是订单列表、服务人员认证、企业认证、客户列表、服务项管理、区域管理，左侧为功能导航栏，涵盖工作台、订单管理、评价管理、服务管理等选项，还包含订单分析、营业分析相关板块的内容展示，页面右上角有刷新按钮，时间选择区域右侧设有导出明细按钮。

订单分析内容如下：

- 有效订单数：在统计时间内，订单状态为已完成的订单数。（订单被取消、退款、关闭状态均不属于有效订单）
- 取消订单数：在统计时间内，提交订单后被取消订总量
- 关闭订单数：在统计时间内，提交订单后被关闭订总量
- 有效订单总额：在统计时间内，有效订单的总订单交易总额
- 实付单均价：在统计时间内，平均每单实际支付额（不包含失效订单）

实付单均价=有效订单总额/有效订单数

![ux3dbib3uofh0px4t7vcc6ivnib](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ux3dbib3uofh0px4t7vcc6ivnib.png)
> 图注：这张图属于第九章系统优化中统计模块的订单统计方案里的经营看板部分，整体展示了多类订单及用户相关统计数据。上方的订单分析区域，标注了“有效订单”的标识，同时呈现了取消订单数为5笔、关闭订单数为400笔、实付单价125元、有效订单总额1,190,625元的核心数据。中间是订单数趋势折线图，以时间为横轴，呈现了统计时段内的订单数量变化。下方的用户分析区域包含用户转化漏斗图和用户平均支付笔数折线图，其中漏斗图展示了不同转化阶段的用户数及对应转化率。

订单趋势：

订单趋势的显示分两种情况：

1. 如果统计时间区间 大于1天，订单趋势显示时间区间内每天的下单总数。
2. 如果统计时间区间小于等于1天，订单趋势显示当天每小时的下单总数。



#### 3）技术方案

根据需求，我们要统计一个时间区间的订单总数、订单均价等指标。统计出结果后通过接口将数据返回给前端，前端在界面展示即可。

**基于什么平台进行统计分析？**

通常统计分析要借助大数据平台进行，流程如下：

<whiteboard token="TS1GwtnCuhspMObVW4tchJ2AnJe"></whiteboard>

说明：

大数据统计系统对数据进行统计，并统计结果存入MySQL。

Java程序根据看板的需求提供查询接口，从统计结果表查询数据。这里使用缓存，将看板需要的数据存入Redis，提高查询性能。



如果数据量不大于千万级别可以基于数据库进行统计。

本项目按年统计，一年的订单数据量是300到400万，并且有单独的历史订单数据，所以基于数据库进行统计。



如何基于数据库进行统计呢？

当用户进入看板页面面向全部数据 进行实时统计其统计速度较慢。

为了提高统计效率可以分层次聚合统计，再基于分层聚合的统计结果进行二次统计。

举例：

我们要统计2023年10月1日 到2023年11月30日的订单总数等指标，我们可以提前按天把每天的订单总数等指标统计出来，当用户去统计2023年10月1日 到2023年11月30日的订单总数时基于按天统计的结果进行二次统计。

按天统计结果：

| 日期 | 订单总数 | 有效订单数 | 取消订单数 | .... | 订单均价 |
|-|-|-|-|-|-|
| 20231001 | 100 | 100 | 0 |  | ... |
| 20231002 | 200 | 199 | 1 |  | ... |
| ... |  |  |  |  |  |

统计数据的分层次聚合需要根据需求确定统计的维度，例如除了按照时间，还可能按地区、产品类别等进行聚合。

根据需求在订单趋势图上除了显示每天的订单总数以外还会按小时进行显示，所以还需要按小时进行统计。



本项目采用滚动式统计，每次统计近15天的数据（如果数据量大可减少统计时段长度），采用滚动式统计的好处是防止统计任务执行失败漏掉统计数据，如下图：

15日统计1到15日的订单。

16日统计2到16日的订单。

依此类推。

<whiteboard token="MXYKwzDjChFJbLbMCYgckNA6nEh"></whiteboard>

分层聚合的粒度有两种：

按天统计，将统计结果存储至按天统计表。

按小时，将统计结果存储至按小时统计表。

有了分层聚合的统计结果，根据用户需求基于分层聚合的统计结果进行二次统计，其统计效率会大大提高，并且有些需求无需进行二次统计直接查询分层聚合结果表即可。

数据流如下：

<whiteboard token="JLJLwvpiehYl8jboAkKcPqCYnwd"></whiteboard>

#### 4）小结

订单统计分析是怎么实现的？



### 5.2 开发订单统计

下边实现按天统计订单。

#### 1）定义mapper

首先编写SQL：

```JSON

select day as id,
day as statTime,
sum(if(orders_status=500,1,0)) effective_order_num,
sum(if(orders_status=600,1,0)) cancel_order_num,
sum(if(orders_status=700,1,0)) close_order_num,
sum(if(orders_status=500,total_amount,0)) effective_order_total_amount
from history_orders_sync where day >= 20231120
GROUP BY day
```

在HistoryOrdersSyncMapper.xml.xml添加mapper映射,如下内容：

```JSON
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.jzo2o.orders.history.mapper.HistoryOrdersSyncMapper">

    <select id="statForDay" parameterType="java.util.Map" resultType="com.jzo2o.orders.history.model.domain.StatDay">
        select day as id,
        day as statTime,
        sum(if(orders_status=500,1,0)) effective_order_num,
        sum(if(orders_status=600,1,0)) cancel_order_num,
        sum(if(orders_status=700,1,0)) close_order_num,
        sum(if(orders_status=500,total_amount,0)) effective_order_total_amount
        from history_orders_sync where day >= #{queryDay}
        GROUP BY day
    </select>
    ...
```

定义mapper接口

```JSON
package com.jzo2o.orders.history.mapper;

import com.jzo2o.orders.history.model.domain.HistoryOrdersSync;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.jzo2o.orders.history.model.domain.StatDay;
import com.jzo2o.orders.history.model.domain.StatHour;
import org.apache.ibatis.annotations.Param;

import java.util.List;

/**
 * <p>
 * 订单统计
 * </p>
 *
 * @author itcast
 * @since 2023-09-21
 */
public interface HistoryOrdersSyncMapper extends BaseMapper<HistoryOrdersSync> {

List<StatDay> statForDay(@Param("queryDay") Integer queryDay);
...
```

#### 2) 定义service

```JSON
package com.jzo2o.orders.history.service;

public interface IHistoryOrdersSyncService extends IService<HistoryOrdersSync> {

    List<StatDay> statForDay(Integer statDay);
    ...
```

实现类如下：

```JSON
package com.jzo2o.orders.history.service.impl;

@Service
@Slf4j
public class HistoryOrdersSyncServiceImpl extends ServiceImpl<HistoryOrdersSyncMapper, HistoryOrdersSync> implements IHistoryOrdersSyncService {

@Override
public List<StatDay> statForDay(Integer statDay) {
    //统计15天以内的订单
    List<StatDay> statForDay = baseMapper.statForDay(statDay);
    if(CollUtils.isEmpty(statForDay)) {
        return Collections.emptyList();
    }
    // 按天统计订单，计算订单总数、均价等信息
    List<StatDay> collect = statForDay.stream().peek(sd -> {
        // 订单总数
        sd.setTotalOrderNum(NumberUtils.add(sd.getEffectiveOrderNum(), sd.getCloseOrderNum(), sd.getCancelOrderNum()).intValue());
        // 实付订单均价
        if (sd.getEffectiveOrderNum().compareTo(0) == 0) {
            sd.setRealPayAveragePrice(BigDecimal.ZERO);
        } else {
           //RoundingMode.HALF_DOWN 表示四舍五入 向下舍弃，如2.345，保留两位小数为2.34
            BigDecimal realPayAveragePrice = sd.getEffectiveOrderTotalAmount().divide(new BigDecimal(sd.getEffectiveOrderNum()), 2, RoundingMode.HALF_DOWN);
            sd.setRealPayAveragePrice(realPayAveragePrice);
        }
    }).collect(Collectors.toList());

    return collect;
}
...
```



#### 3) 定时任务

定义service：

```JSON

package com.jzo2o.orders.history.service.impl;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.toolkit.Wrappers;
import com.jzo2o.common.utils.DateUtils;
import com.jzo2o.orders.history.model.domain.StatDay;
import com.jzo2o.orders.history.mapper.StatDayMapper;
import com.jzo2o.orders.history.service.IHistoryOrdersService;
import com.jzo2o.orders.history.service.IHistoryOrdersSyncService;
import com.jzo2o.orders.history.service.IStatDayService;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.jzo2o.orders.history.mapper.StatDayMapper;
import com.jzo2o.orders.history.model.domain.StatDay;
import com.jzo2o.orders.history.model.domain.StatHour;
import com.jzo2o.orders.history.service.IStatDayService;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;
import java.time.LocalDateTime;
import java.util.List;

/**
 * <p>
 * 日统计表 服务实现类
 * </p>
 *
 * @author itcast
 * @since 2023-09-21
 */
@Service
public class StatDayServiceImpl extends ServiceImpl<StatDayMapper, StatDay> implements IStatDayService {
@Override
public void statAndSaveData() {
    // 1.数据统计
    // 15天前时间
    LocalDateTime statDayLocalDateTime = DateUtils.now().minusDays(15);
    long statDayTime = DateUtils.getFormatDate(statDayLocalDateTime, "yyyMMdd");
    // 统计数据
    List<StatDay> statDays = historyOrdersSyncService.statForDay((int) statDayTime);
    if(ObjectUtils.isEmpty(statDays)){
        return ;
    }    
    // 2.数据保存至按天统计表
    saveOrUpdateBatch(statDays);
}
```

定义xxl-job定时任务，在历史订单服务通过定时任务完成数据统计。

```JSON
package com.jzo2o.orders.history.handler;

import com.jzo2o.orders.history.service.IHistoryOrdersServeService;
import com.jzo2o.orders.history.service.IHistoryOrdersService;
import com.jzo2o.orders.history.service.IStatDayService;
import com.jzo2o.orders.history.service.IStatHourService;
import com.xxl.job.core.handler.annotation.XxlJob;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;

@Component
public class XxlJobHandler {

    /**
     * 按天统计保存15天内的订单数据
     * 按小时统计保存15天内的订单数据
     */
    @XxlJob("statAndSaveData")
    public void statAndSaveDataForDay() {
        //按天统计保存15天内的订单数据
        statDayService.statAndSaveData();
        //按小时统计保存15天内的订单数据
        statHourService.statAndSaveData();
    }
    
    ...
```



#### 4）测试

添加订单统计任务

![yqnlbqxb6oyuiexs0cxcfcnenug](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/yqnlbqxb6oyuiexs0cxcfcnenug.png)
> 图注：图片展示的是xxl-job定时任务配置界面。基础配置中，执行器为历史订单，负责人是xxijob，任务描述为订单统计，报警邮件需填写。调度配置里，调度类型为CRON，Cron为0/10 * * * * ?。任务配置中，运行模式为BEAN，JobHandler为statAndSaveData。高级配置部分，路由策略为第一个，调度过期策略为忽略，任务超时时间为0，阻塞处理策略为单机串行，失败重试次数为0。

保证history_orders_sync和history_orders_serve_sync有要统计的数据，按订单完成时间统计15天以内的订单

统计完成写入stat_day和stat_hour两张表，stat_day存储按天统计数据，stat_hour存储按小时统计数据。

#### 5) 按小时统计

自行阅读按小时统计代码。



#### 6）小结

订单统计分析是怎么实现的？





### 5.3 开发经营看板

#### 1）需求分析

目前我们完成对订单数据按天、按小时分层聚合，下边根据经营看板的需求进行二次统计，提供查询接口给前端，前端获取数据后在看板界面展示。



**需求1：根据时间区间统计**

根据时间区间统计以下内容：

![ve6ebtv7moaq7nxuliccwylonzf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ve6ebtv7moaq7nxuliccwylonzf.png)
> 图注：图片展示的是云岚到家平台工作台概览仪表盘界面。界面左侧有工作台、订单管理、评价管理、服务管理、服务人员管理等导航栏。右侧“经营分析”区域有日期选择框，显示2023年11月24日 - 2023年12月24日。下方有订单分析和营业分析板块，订单分析板块有有效订单数、取消订单数、关闭订单数等数据，营业分析板块有实付单均价、有效订单总额等数据。该图与上下文需求分析中关于订单趋势图的描述相关，用于展示订单趋势数据。

- 有效订单数：在统计时间内，订单状态为已完成的订单数。（订单被取消、退款、关闭状态均不属于有效订单）
- 取消订单数：在统计时间内，提交订单后被取消订总量
- 关闭订单数：在统计时间内，提交订单后被关闭订总量
- 有效订单总额：在统计时间内，有效订单的总订单交易总额
- 实付单均价：在统计时间内，平均每单实际支付额（不包含失效订单）

实付单均价=有效订单总额/有效订单数

**需求2: 订单趋势图**

如果统计时间区间 大于1天，订单趋势显示时间区间内每天的下单总数，查询按天统计表的数据即可。

如果统计时间区间小于等于1天，订单趋势显示当天每小时的下单总数，查询按小时统计表的数据即可。

![newnbnkhxoddirxjmlicfakons9](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/newnbnkhxoddirxjmlicfakons9.png)
> 图注：这是开发经营看板的截图，上方展示了订单相关统计数据，包含有效订单（5笔）、取消订单（400笔）、实付均价（125元）、有效订单金额（1,190,625元），还配有标注说明有效订单为“订单状态为已完成的订单”。中部的订单数趋势图呈现了不同日期的订单数量波动，另有时间标注“9.10 共707笔”。下方包含用户分析相关模块，左侧是用户转化漏斗图，标注了总用户数2000、下单数640、付款数460，还标注了下单转化率32%、付款转化率72%；右侧则是用户平均支付笔数的折线对比图，展示了不同月份2021年与2022年的相关数据变化，整体对应了看板展示各类经营统计数据的需求。

**需求3: 缓存**

看板中显示的数据有些需要进行二次统计，为了提高查询性能通常对二次统计的数据结果进行缓存，设置缓存过期时间，通常30分钟以内，根据监控数据变化的实时性去设置，本项目缓存数据为30分钟，当缓存过期重新统计最新的数据在看板展示。



#### 2）阅读代码

1. 接口

```JSON
package com.jzo2o.orders.history.controller.operation;

import com.jzo2o.orders.history.model.dto.response.OperationHomePageResDTO;
import com.jzo2o.orders.history.service.OrdersStatisticsService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import java.io.IOException;
import java.time.LocalDateTime;

/**
 * 订单统计
 *
 * @author itcast
 * @create 2023/9/21 15:15
 **/
@Api(tags = "运营端 - 订单统计相关接口")
@RestController("operationOrdersStatisticsController")
@RequestMapping("/operation/orders-statistics")
public class OrdersStatisticsController {
    @Resource
    private OrdersStatisticsService ordersStatisticsService;

    @GetMapping("/homePage")
    @ApiOperation("运营端首页数据")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "minTime", value = "开始时间", required = true, dataTypeClass = LocalDateTime.class),
            @ApiImplicitParam(name = "maxTime", value = "结束时间", required = true, dataTypeClass = LocalDateTime.class)
    })
    public OperationHomePageResDTO homePage(@RequestParam("minTime") @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") LocalDateTime minTime,
                                            @RequestParam("maxTime") @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") LocalDateTime maxTime) {
        return ordersStatisticsService.homePage(minTime, maxTime);
    }
```

1. service方法

service方法上使用spring cache注解对看板上展示的数据进行缓存。

```JSON
package com.jzo2o.orders.history.service.impl;

import cn.hutool.core.bean.BeanUtil;
import cn.hutool.core.date.DatePattern;
import cn.hutool.core.date.LocalDateTimeUtil;
import cn.hutool.core.io.FileUtil;
import cn.hutool.core.util.NumberUtil;
import cn.hutool.core.util.ObjectUtil;
import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.ExcelWriter;
import com.alibaba.excel.util.MapUtils;
import com.alibaba.excel.write.metadata.WriteSheet;
import com.alibaba.fastjson.JSON;
import com.jzo2o.common.expcetions.ForbiddenOperationException;
import com.jzo2o.common.utils.BeanUtils;
import com.jzo2o.common.utils.ObjectUtils;
import com.jzo2o.mvc.utils.ResponseUtils;
import com.jzo2o.orders.history.model.domain.StatDay;
import com.jzo2o.orders.history.model.domain.StatHour;
import com.jzo2o.orders.history.model.dto.excel.AggregationStatisticsData;
import com.jzo2o.orders.history.model.dto.excel.ExcelMonthData;
import com.jzo2o.orders.history.model.dto.excel.MonthElement;
import com.jzo2o.orders.history.model.dto.excel.StatisticsData;
import com.jzo2o.orders.history.model.dto.response.OperationHomePageResDTO;
import com.jzo2o.orders.history.service.IStatDayService;
import com.jzo2o.orders.history.service.IStatHourService;
import com.jzo2o.orders.history.service.OrdersStatisticsService;
import com.jzo2o.orders.history.utils.EasyExcelUtil;
import org.apache.poi.ss.formula.functions.T;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.math.BigDecimal;
import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;
import java.time.LocalDateTime;
import java.time.temporal.ChronoUnit;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

import static com.jzo2o.mvc.constants.HeaderConstants.BODY_PROCESSED;

/**
 * 订单统计服务层
 *
 * @author itcast
 * @create 2023/9/21 15:19
 **/
@Service
public class OrdersStatisticsServiceImpl implements OrdersStatisticsService {


/**
 * 运营端首页数据
 *
 * @param minTime 开始时间
 * @param maxTime 结束时间
 * @return 首页数据
 */
@Override
@Cacheable(value = "JZ_CACHE", cacheManager = "cacheManager30Minutes")
public OperationHomePageResDTO homePage(LocalDateTime minTime, LocalDateTime maxTime) {
    //校验查询时间
    if (LocalDateTimeUtil.between(minTime, maxTime, ChronoUnit.DAYS) > 365) {
        throw new ForbiddenOperationException("查询时间区间不能超过一年");
    }

    //如果查询日期是同一天，则按小时查询折线图数据
    if (LocalDateTimeUtil.beginOfDay(maxTime).equals(minTime)) {
        return getHourOrdersStatistics(minTime);
    } else {
        //如果查询日期不是同一天，则按日查询折线图数据
        return getDayOrdersStatistics(minTime, maxTime);
    }
}

/**
 * 按日统计数据
 *
 * @param minTime 最小时间
 * @param maxTime 最大时间
 * @return 统计数据
 */
private OperationHomePageResDTO getDayOrdersStatistics(LocalDateTime minTime, LocalDateTime maxTime) {
    //定义要返回的对象
    OperationHomePageResDTO operationHomePageResDTO = OperationHomePageResDTO.defaultInstance();

    //日期格式化，格式：yyyyMMdd
    String minTimeDayStr = LocalDateTimeUtil.format(minTime, DatePattern.PURE_DATE_PATTERN);
    String maxTimeDayStr = LocalDateTimeUtil.format(maxTime, DatePattern.PURE_DATE_PATTERN);

    //根据日期区间聚合统计数据
    StatDay statDay = statDayService.aggregationByIdRange(Long.valueOf(minTimeDayStr), Long.valueOf(maxTimeDayStr));

    //将statDay拷贝到operationHomePageResDTO
    operationHomePageResDTO = BeanUtils.copyIgnoreNull(BeanUtil.toBean(statDay, OperationHomePageResDTO.class), operationHomePageResDTO, OperationHomePageResDTO.class);

    //根据日期区间查询按日统计数据
    List<StatDay> statDayList = statDayService.queryListByIdRange(Long.valueOf(minTimeDayStr), Long.valueOf(maxTimeDayStr));
    //将statDayList转为map<趋势图横坐标，订单总数>
    Map<String, Integer> ordersCountMap = statDayList.stream().collect(Collectors.toMap(s -> dateFormatter(s.getId()), StatDay::getTotalOrderNum));
    //趋势图上全部点
    List<OperationHomePageResDTO.OrdersCount> ordersCountsDef = OperationHomePageResDTO.defaultDayOrdersTrend(minTime, maxTime);
    //遍历ordersCountsDef，将统计出来的ordersCountMap覆盖ordersCountsDef中的数据
    ordersCountsDef.stream().forEach(v->{
        if (ObjectUtil.isNotEmpty(ordersCountMap.get(v.getDateTime()))) {
            v.setCount(ordersCountMap.get(v.getDateTime()));
        }
    });
    //将ordersCountsDef放入operationHomePageResDTO
    operationHomePageResDTO.setOrdersTrend(ordersCountsDef);

    return operationHomePageResDTO;
}

/**
 * 按小时统计数据
 *
 * @param minTime 开始时间
 * @return 统计数据
 */
private OperationHomePageResDTO getHourOrdersStatistics(LocalDateTime minTime) {
    //定义要返回的对象
    OperationHomePageResDTO operationHomePageResDTO = OperationHomePageResDTO.defaultInstance();

    //获取当前日期，格式：yyyyMMdd
    String minTimeDayStr = LocalDateTimeUtil.format(minTime, DatePattern.PURE_DATE_PATTERN);

    //查询该日期的统计数据
    StatDay statDay = statDayService.getById(Long.valueOf(minTimeDayStr));

    //趋势图上全部点
    List<OperationHomePageResDTO.OrdersCount> ordersCountsDef = OperationHomePageResDTO.defaultHourOrdersTrend();
    if (null == statDay) {
        operationHomePageResDTO.setOrdersTrend(ordersCountsDef);
        return operationHomePageResDTO;
    }

    //如果统计数据不为空，拷贝数据
    operationHomePageResDTO = BeanUtil.toBean(statDay, OperationHomePageResDTO.class);

    //根据时间区间查询小时统计数据，并转换为map结构，key为小时，value为订单数量
    List<StatHour> statHourList = statHourService.queryListByIdRange(Long.valueOf(minTimeDayStr + MIN_HOUR), Long.valueOf(minTimeDayStr + MAX_HOUR));
    //将statHourList转map
    Map<String, Integer> ordersCountMap = statHourList.stream().collect(Collectors.toMap(s -> String.format("%02d",s.getId() % 100), StatHour::getTotalOrderNum));
    //遍历ordersCountsDef，将统计出来的ordersCountMap覆盖ordersCountsDef中的数据
    ordersCountsDef.stream().forEach(v->{
        if (ObjectUtil.isNotEmpty(ordersCountMap.get(v.getDateTime()))) {
            v.setCount(ordersCountMap.get(v.getDateTime()));
        }
    });

    //组装订单数趋势，返回结果
    operationHomePageResDTO.setOrdersTrend(ordersCountsDef);
    return operationHomePageResDTO;
}

...
```



#### 3）测试

首先完成订单按天、按小时统计测试。

然后进入运营端

通过工作台查看统计数据

![f18abaogno6b2vxd9amcnozinng](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/f18abaogno6b2vxd9amcnozinng.png)
> 图注：这张图片展示的是云匠到家的运营端经营看板页面，该页面属于第九章统计模块下开发经营看板的内容。页面顶部设有订单列表、服务人员认证等功能入口，左侧为工作台、订单管理等导航栏；经营分析区显示的统计数据包含有效订单2笔、取消订单0笔、关闭订单0笔、实对平均价1.00元、有效订单总额2.00元，还附有订单数趋势图，右上角设有导出明细按钮，此页面正是查看经营看板统计数据的对应界面。



#### 4）小结

经营看板功能怎么实现？





### 5.4 订单统计结果导出

#### 1) 需求分析

在经营看板界面可将订单趋势图的数据导出Excel

点击“导出明细”，导出Excel。

![coonbphjvoluzqxhkm0csypnnbb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/coonbphjvoluzqxhkm0csypnnbb.png)
> 图注：该图片是云岚到家管理端的经营看板界面，对应文档中订单统计结果导出的相关内容。界面显示时间范围为2023-10-20至2023-11-20，右上方有红色的“导出明细”按钮，符合文档中点击该按钮导出Excel的操作指引。界面内展示了订单分析与营业分析的相关数据，其中有效订单为2笔，取消订单数、关闭订单数均为0笔，实付单价为1.00元，有效订单总额为2.00元，还呈现了订单数趋势的折线图，该界面是完成订单统计结果导出操作的前置页面。

示例：

2023-10-20\~2023-11-20 全国经营分析统计.xlsx

![jsxtbul7hozu3fxeissctkbxnmb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/jsxtbul7hozu3fxeissctkbxnmb.png)
> 图注：这张图片是全国经营分析统计的订单数据导出Excel示例，统计区间为2023-10-20至2023-11-20，导出时间为2023年11月20日22点15分08秒。表格按月份分为10月、11月两部分，每日及月度统计涵盖日期、订单数、有效订单数、取消订单数、关闭订单数、有效订单总额、实付单均价等列，10月月底的“月统计”行中，所有统计项数值均为0，完整呈现了该区间内的订单统计明细数据，与文档中订单统计结果导出的功能示例内容相匹配。

示例2：

![ekegbtm8kosrxnxg0hgcdhfonwb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ekegbtm8kosrxnxg0hgcdhfonwb.png)
> 图注：图片为2023年11月20日全国经营分析统计表，显示2023 - 11 - 20 21:42:37的数据。表头有日期、订单数、有效订单数、取消订单数、关闭订单数、有效订单总额、实付单均价等列。数据从00时到23时，每小时订单数、有效订单数等均为0，仅16时订单数、有效订单数、关闭订单数、有效订单总额、实付单均价分别为2、2、0、2、1。日续计部分订单数、有效订单数、关闭订单数均为2，有效订单总额、实付单均价均为1。该表与上下文介绍的订单统计结果导出功能相关，展示了导出的Excel数据示例。



#### 2) 接口设计

```JSON
package com.jzo2o.orders.history.controller.operation;

import com.jzo2o.orders.history.model.dto.response.OperationHomePageResDTO;
import com.jzo2o.orders.history.service.OrdersStatisticsService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import java.io.IOException;
import java.time.LocalDateTime;

/**
 * 订单统计
 *
 * @author itcast
 * @create 2023/9/21 15:15
 **/
@Api(tags = "运营端 - 订单统计相关接口")
@RestController("operationOrdersStatisticsController")
@RequestMapping("/operation/orders-statistics")
public class OrdersStatisticsController {
    @Resource
    private OrdersStatisticsService ordersStatisticsService;


    /**
     * 文件下载并且失败的时候返回json（默认失败了会返回一个有部分数据的Excel）
     *
     * @since 2.1.1
     */
    @GetMapping("downloadStatistics")
    @ApiOperation("导出统计数据")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "minTime", value = "开始时间", required = true, dataTypeClass = LocalDateTime.class),
            @ApiImplicitParam(name = "maxTime", value = "结束时间", required = true, dataTypeClass = LocalDateTime.class)
    })
    public void downloadStatistics(@RequestParam("minTime") @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") LocalDateTime minTime,
                                   @RequestParam("maxTime") @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") LocalDateTime maxTime) throws IOException {
        ordersStatisticsService.downloadStatistics(minTime, maxTime);
    }
}
```



#### 3）统计结果导出

理解需求并完成订单统计结果导出功能。

完成开发后进行测试：

首先完成订单按天、按小时统计测试。

然后进入运营端

通过工作台查看统计数据,点击“导出明细”

![g2csbnkhooioxvxctmlcdu0dnqh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/g2csbnkhooioxvxctmlcdu0dnqh.png)
> 图注：该图片展示的是云岚到家运营端的工作台页面，页面顶部设有订单列表、服务人员认证等多个功能入口，左侧是工作台、订单管理等导航栏。页面主体是2023-10-20至2023-11-20期间的经营分析内容，涵盖订单分析、营业分析数据，其中有效订单为2笔，取消订单数、关闭订单数均为0，实付单均价1.00元，有效订单总额2.00元，还配有订单数趋势图。页面右上角设有红色的“导出明细”按钮，该页面对应文档中统计结果导出环节里，进入运营端后点击“导出明细”的测试操作场景展示。



#### 4）小结

数据导出功能如何实现？



数据导入功能如何实现？



## 作业

### 订单查询优化bug修复

用户端订单列表采用滚动查询，向下滚动屏幕查询出现重复记录，可通过前端跟踪。

请大家修复这个问题，先重现这个问题，再排查问题，并进行修复。

### 历史订单实现按年分表。

#### 提示

首先在历史订单微服务添加依赖：

```Java
<dependency>
    <groupId>com.jzo2o</groupId>
    <artifactId>jzo2o-shardingsphere-jdbc</artifactId>
</dependency>
```

不分库所以不用设置分库策略，定义一个数据源即可

按年分表要确定分表键是什么，分表策略的表达式是什么

提前创建好2024、2025两个年份的历史订单表和历史服务单表，其它表都为全局表。

![zo4hbmapcozm4vxlvr6cyeh7ncf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/zo4hbmapcozm4vxlvr6cyeh7ncf.png)
> 图注：图片展示了名为“jzo2o-orders-history”的文件夹内容，其中包含多个以“history_orders”和“history_orders_”开头的文件夹，如“history_orders”“history_orders_2024”等，还有“history_orders_sync”“stat_day”“stat_hour”等文件。该图片与文档中“订单查询优化bug修复”章节下“提示”部分相关，用于说明在历史订单微服务添加依赖时，提前创建好2024、2025两个年份的历史订单表和历史服务单表，其它表都为全局表，此图展示了相关表的存储情况。



测试时注意修改数据库的连接配置，如下：

![p2yebofieoolqdxvoilcntegnjh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/p2yebofieoolqdxvoilcntegnjh.png)
> 图注：图片展示的是在历史订单微服务中添加依赖时的配置界面。界面中显示了命名空间、Data ID、Group等信息，以及配置格式为YAML。关键部分是配置内容区域，其中“#”号注释部分后有“#”号，表明后续内容为注释。非注释部分的“spring.datasource.driver-class-name”设置为“org.apache.shardingsphere.driver.ShardingSphereDriver”，“url”设置为“jdbc:shardingsphere:classpath:shardingsphere-jdbc-${spring.profiles.active}.yml”。该图片与文档中测试时修改数据库连接配置的内容相关，用于说明配置数据库连接相关信息。

测试时通过手动修改订单数据库中历史订单同步表的数据，使其同步到历史订单数据库的同步表中。

然后通过定时任务进行数据迁移，观察是否迁移到history_orders_2024表或history_orders_2025表中。

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 云岚到家项目 / 第九章 系统优化 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/WGxyw9s59ieMx9kbvzqcr9S7nsb)
