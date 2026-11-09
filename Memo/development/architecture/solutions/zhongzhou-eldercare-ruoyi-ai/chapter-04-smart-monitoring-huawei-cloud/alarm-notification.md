---
type: memo
date: 2026-11-06
aliases:
  - "bao-jing-tong-zhi-ti-xing"
  - "报警通知提醒"
---

# 报警通知提醒

## 需求说明

当设备中的数据触发了报警规则之后，需要及时提醒相关人员来进行处理，为了更快的让相对应的负责人收到消息，一旦监测到有异常数据，可以在当前后台管理系统中进行通知，如下效果：

![lthybbp8douj9txuj7nce438ndf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/lthybbp8douj9txuj7nce438ndf.png)
> 图注：图片展示的是后台管理系统中设备数据触发报警规则后的报警提醒效果。弹窗标题为“报警提醒”，内容显示接入位置为2楼201,201-1，所属产品是睡眠监测带，功能名称为心率，数据值为54。下方有“9秒后，弹窗自动关闭”提示，底部有“取消”和“查看”两个按钮。该图片与文档中需求说明部分对应，直观呈现了设备数据触发报警规则后在后台管理系统中进行通知的效果。

要想实现这个功能，我们需要使用一个符合该场景的技术来解决，就是下面要讲的websocket

## Websocket

### 概述

WebSocket 是基于 TCP 的一种新的**网络协议**。它实现了浏览器与服务器全双工通信——浏览器和服务器只需要完成一次握手，两者之间就可以创建**持久性**的连接， 并进行**双向**数据传输。

**HTTP协议和WebSocket协议对比：**

- HTTP是**短连接**
- WebSocket是**长连接**
- HTTP通信是**单向**的，基于请求响应模式
- WebSocket支持**双向**通信
- HTTP和WebSocket底层都是TCP连接

![ahw9bheacoqeqfx8b19cxuwtnke](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ahw9bheacoqeqfx8b19cxuwtnke.png)
> 图注：这张图对比展示了HTTP和WebSocket两种协议的通信流程，左侧为HTTP的通信过程，仅包含客户端向服务器发送请求、服务器向客户端返回响应的单向请求-响应交互，且通信过程多次重复请求与响应；右侧为WebSocket的通信过程，首先完成握手与确认后，实现了双向通信，最后才结束连接。它直观呈现了两种协议的核心差异，即HTTP为单向短连接，WebSocket支持双向长连接，与文档中对两种协议的特性对比内容相呼应，帮助读者更清晰理解两者的区别。

**思考：**既然WebSocket支持双向通信，功能看似比HTTP强大，那么我们是不是可以基于WebSocket开发所有的业务功能？

**WebSocket缺点：**

服务器长期维护长连接需要一定的成本

各个浏览器支持程度不一

WebSocket 是长连接，受网络限制比较大，需要处理好重连

**结论：**WebSocket并不能完全取代HTTP，它只适合在特定的场景下使用

### 使用场景

1). 视频弹幕

![oobybi9efoijpuxcf0jcenptn2g](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/oobybi9efoijpuxcf0jcenptn2g.png)
> 图注：这是一张黑马程序员Java面试专题教学视频的播放界面截图，视频标题为“新版Java面试专题视频教程，java八股文面试全套真题+深度详解（含大厂高频面试...”，当前播放进度为07:42，总时长08:53，清晰度设置为720P高清。界面顶部、文字旁和右侧都带有“加油”的相关文案，核心文字内容是“多一句没有，少一句不行，用更短时间，教会更实用的技术！让千万学生少走弯路！”，画面右下角还展示了对应课程示例内容的缩略小图，整体是面向Java面试备考的教学类视频界面。

2). 网页聊天

![tsg0bxv0jobpyyxvbq4cnjl4nlh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/tsg0bxv0jobpyyxvbq4cnjl4nlh.png)
> 图注：该图片展示了传智教育官网的界面，画面右侧显示了网站的在线客服弹窗，弹窗内有用户与客服的对话内容，包括用户询问Java相关问题、客服答复的文字信息。画面的主体区域是传智教育JavaEE高级软件工程师课程的宣传内容，标注了“JavaEE V13.0新升级”“五大专技 轻松过招职场”等课程推广信息，页面顶部还有“男女成团·报名有礼”的活动横幅，整体场景对应文档中提及的网页聊天这一WebSocket的使用场景，体现了网页端的实时交互功能。

3). 体育实况更新

![anoub2auhopjlmxkolcc4c7qnob](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/anoub2auhopjlmxkolcc4c7qnob.png)
> 图注：图片展示的是NBA比赛实时比分界面。画面中显示勇士队（GSW）83分，湖人队（LAL）90分，第三节剩余7分27秒。比赛于2023年10月14日10:00开赛，球馆为加密网球馆，上座人数为0。界面下方有总分、数据直播、文字直播、手机直播、比赛战报等选项。该图片与上下文介绍的体育实况更新使用场景相契合，直观呈现了体育赛事实时比分的展示形式。

### 入门案例

（1）需求说明

实现浏览器与服务器全双工通信。浏览器既可以向服务器发送消息，服务器也可主动向浏览器推送消息。

![mtekbd9yzokjbwxldkkc4ogznlf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mtekbd9yzokjbwxldkkc4ogznlf.png)
> 图注：这张图片展示了WebSocket报警通知相关的客户端运行效果界面，界面包含输入框、“发送消息”和“关闭连接”两个按钮。界面显示有“连接成功”的提示，后续是服务端定时向客户端推送的内容，每条内容都为“Hello Client, Current Server Time:”加上具体的时间戳，从2023-10-14T13:46:40到13:47:45共有多条推送记录，界面末尾还有“close”标识，体现出WebSocket连接后的数据交互情况。

（2）流程如下

![iot8b6epxoshdvxnamnct9tqnsh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/iot8b6epxoshdvxnamnct9tqnsh.png)
> 图注：这张图展示了WebSocket报警通知提醒的交互流程，流程始于橙色的浏览器与蓝色的服务端（server）先进行握手操作，服务端确认后，浏览器向服务端发起携带数据的请求，服务端完成主从数据推送至浏览器的操作，最后双方完成连接关闭，该图对应文档中说明的WebSocket交互流程，清晰呈现了客户端与服务端在Websocket连接及通信中的完整环节。

- 需要准备客户端，可以在浏览器上编写客户端
- 服务端，使用java代码实现，启动一个服务
- 客户端与服务端连接成功以后

  - 客户端可以发送数据到服务器端
  - 服务器端也可以发送数据到客户端（浏览器）

（3）功能实现

**客户端**

定义websocket.html页面充当客户端（资料中已提供）

```HTML
<!DOCTYPE HTML>
<html>
<head>
    <meta charset="UTF-8">
    <title>WebSocket Demo</title>
</head>
<body>
    <input id="text" type="text" />
    <button onclick="send()">发送消息</button>
    <button onclick="closeWebSocket()">关闭连接</button>
    <div id="message">
    </div>
</body>
<script type="text/javascript">
    var websocket = null;
    var clientId = Math.random().toString(36).substr(2);

    // 判断当前浏览器是否支持WebSocket
    if('WebSocket' in window) {
        // 连接WebSocket节点
        websocket = new WebSocket("ws://localhost:8080/ws/" + clientId);
    }
    else {
        alert('Not support websocket');
    }

    // 连接发生错误的回调方法
    websocket.onerror = function() {
        setMessageInnerHTML("error");
    };

    // 连接成功建立的回调方法
    websocket.onopen = function() {
        setMessageInnerHTML("连接成功");
    }

    // 接收到消息的回调方法
    websocket.onmessage = function(event) {
        setMessageInnerHTML(event.data);
    }

    // 连接关闭的回调方法
    websocket.onclose = function() {
        setMessageInnerHTML("close ");
    }

    // 监听窗口关闭事件，当窗口关闭时，主动去关闭websocket连接，防止连接还没断开就关闭窗口，server端会抛异常。
    window.onbeforeunload = function() {
        websocket.close();
    }

    // 将消息显示在网页上
    function setMessageInnerHTML(innerHTML) {
        document.getElementById('message').innerHTML += innerHTML + '<br/>';
    }

    // 发送消息
    function send() {
        var message = document.getElementById('text').value;
        websocket.send(message);
    }
        
    // 关闭连接
    function closeWebSocket() {
        websocket.close();
    }
</script>
</html>
```

**服务端**

然后导入依赖

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

定义配置类，注册WebSocket的服务端组件(从资料中直接导入即可)

```Java
package com.itheima.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

@Configuration
public class WebSocketConfig {

    /**
     * 注册基于@ServerEndpoint声明的Websocket Endpoint
     * @return
     */
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }

}
```

定义WebSocket服务端组件(资料中已提供)

```Java
package com.itheima.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;

import javax.websocket.*;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

@Slf4j
@Component
@ServerEndpoint("/ws/{sid}")
public class WebSocketServer {

    private static Map<String,Session> sessionMap = new HashMap<>();

    /**
     * 连接建立时触发
     * @param session
     * @param sid
     */
    @OnOpen
    public void onOpen(Session session , @PathParam("sid") String sid) {
        log.info("有客户端连接到了服务器 , {}", sid);
        sessionMap.put(sid, session);
    }

    /**
     * 服务端接收到消息时触发
     * @param session
     * @param message
     * @param sid
     */
    @OnMessage
    public void onMessage(Session session , String message, @PathParam("sid") String sid) {
        log.info("接收到了客户端 {} 发来的消息 : {}", sid ,message);
    }

    /**
     * 连接关闭时触发
     * @param session
     * @param sid
     */
    @OnClose
    public void onClose(Session session , @PathParam("sid") String sid) {
        System.out.println("连接断开:" + sid);
        sessionMap.remove(sid);
    }

    /**
     * 通信发生错误时触发
     * @param session
     * @param sid
     * @param throwable
     */
    @OnError
    public void onError(Session session , @PathParam("sid") String sid , Throwable throwable) {
        System.out.println("出现错误:" + sid);
        throwable.printStackTrace();
    }

    /**
     * 广播消息
     * @param message
     * @throws IOException
     */
    public void sendMessageToAll(String message) throws IOException {
        Collection<Session> sessions = sessionMap.values();
        if(!CollectionUtils.isEmpty(sessions)) {
            for (Session session : sessions) {
                // 服务器向客户端发送消息
                session.getBasicRemote().sendText(message);
            }
        }
    }
}
```

定义定时任务类，定时向客户端推送数据

```Java
package com.itheima.job;

import com.itheima.config.WebSocketServer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.time.LocalDateTime;

/**
 * @author sjqn
 */
@Component
public class MessageTask {

    @Autowired
    private WebSocketServer webSocketServer;

    @Scheduled(cron = "0/5 * * * * ?")
    public void sendMessageToAllClient() throws IOException {
        webSocketServer.sendMessageToAll("Hello Client , Current Server Time : " + LocalDateTime.now());
    }
}
```

**测试**

启动后端服务，打开浏览器即可成功，在浏览器中可以看到服务器端推送的数据，浏览器也可以发送数据到服务器端

客户端的效果：

![y9zvblsb1owhvxxjtlqcxncxnce](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/y9zvblsb1owhvxxjtlqcxncxnce.png)
> 图注：图片展示的是Websocket客户端的效果界面。界面上方有一个输入框，显示“hello”，旁边有“发送消息”和“关闭连接”按钮。下方文字区域显示“连接成功”，并有多个“Hello Client, Current Server Time”信息，时间从2023 - 10 - 14 14:08:20.003118900到2023 - 10 - 14 14:08:50.002298100不等。该图片与文档中介绍Websocket客户端功能实现的内容相关，直观呈现了客户端成功连接后，服务端推送数据及客户端接收显示的效果。

服务端接收到的数据，在控制台打印

![vfxwbskmcobepaxkhocc9hcnnde](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/vfxwbskmcobepaxkhocc9hcnnde.png)
> 图注：图片展示了WebSocket服务端接收到的数据在控制台的打印信息。上方显示“com.itheima.config.WebSocketServer：有客户端连接到了服务器，mnhowixntzk”，表明有客户端连接到服务器。下方显示“com.itheima.config.WebSocketServer：接收到了客户端mnhowixntzk发来的消息：hello”，表明服务器接收到客户端发送的消息“hello”。该图片与文档中介绍WebSocket服务端功能实现的内容相关，直观呈现了服务端接收到客户端数据后的打印效果。

## 功能实现

### 实现思路

<whiteboard token="UnBqwFzWahL9WubYbrZcVWFLnEe"></whiteboard>

### 环境集成

依据我们刚才做过的websocket入门案例，把websocket集成到我们的项目中

#### 导入依赖

在zzyl-nursing-platform中如导入以下依赖

```Java
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-websocket</artifactId>
 </dependency>
```

#### 定义WebSocket服务和注册WebSocket

配置类：

```Java
package com.zzyl.nursing.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

@Configuration
public class WebSocketConfig {

    /**
     * 注册基于@ServerEndpoint声明的Websocket Endpoint
     * @return
     */
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }

}
```

WebSocketServer 

```Java
package com.zzyl.nursing.config;

import cn.hutool.core.collection.CollUtil;
import cn.hutool.core.util.ObjectUtil;
import cn.hutool.json.JSONUtil;
import com.zzyl.common.exception.base.BaseException;
import com.zzyl.nursing.vo.AlertNotifyVo;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;
import org.springframework.web.socket.config.annotation.EnableWebSocket;

import javax.websocket.*;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.util.Collection;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Slf4j
@Component
@EnableWebSocket
@ServerEndpoint("/ws/{sid}")
public class WebSocketServer {

    private static Map<String, Session> sessionMap = new HashMap<>();

    /**
     * 连接建立时触发
     *
     * @param session
     * @param sid
     */
    @OnOpen
    public void onOpen(Session session, @PathParam("sid") String sid) {
        log.info("有客户端连接到了服务器 , {}", sid);
        sessionMap.put(sid, session);
    }

    /**
     * 服务端接收到消息时触发
     *
     * @param session
     * @param message
     * @param sid
     */
    @OnMessage
    public void onMessage(Session session, String message, @PathParam("sid") String sid) {
        log.info("接收到了客户端 {} 发来的消息 : {}", sid, message);
    }

    /**
     * 连接关闭时触发
     *
     * @param session
     * @param sid
     */
    @OnClose
    public void onClose(Session session, @PathParam("sid") String sid) {
        System.out.println("连接断开:" + sid);
        sessionMap.remove(sid);
    }

    /**
     * 通信发生错误时触发
     *
     * @param session
     * @param sid
     * @param throwable
     */
    @OnError
    public void onError(Session session, @PathParam("sid") String sid, Throwable throwable) {
        System.out.println("出现错误:" + sid);
        throwable.printStackTrace();
    }

    /**
     * 广播消息
     *
     * @param message
     * @throws IOException
     */
    public void sendMessageToAll(String message) throws IOException {
        Collection<Session> sessions = sessionMap.values();
        if (!CollectionUtils.isEmpty(sessions)) {
            for (Session session : sessions) {
                //服务器向客户端发送消息
                session.getBasicRemote().sendText(message);
            }
        }
    }

    /**
     * 发送websocket消息给指定消费者
     *
     * @param alertNotifyVo 报警消息
     * @param userIds   报警数据map
     * @throws IOException io异常
     */
    public void sendMessageToConsumer(AlertNotifyVo alertNotifyVo, Collection<Long> userIds) {
        // 如果消费者为空，程序结束
        if (CollUtil.isEmpty(userIds)) {
            return;
        }

        // 如果websoket客户端为空，程序结束
        if (ObjectUtil.isEmpty(sessionMap)) {
            return;
        }

        // 遍历消费者，发送消息
        // key为消息接收人id，value为报警数据id
        userIds.forEach(userId -> {
            // 获取该消费者的websocket连接，如果不存在，跳出本次循环
            Session session = sessionMap.get(String.valueOf(userId));
            if (ObjectUtil.isEmpty(session)) {
                return;
            }
            // 获取该消费者的websocket连接，并发送消息
            try {
                session.getBasicRemote().sendText(JSONUtil.toJsonStr(alertNotifyVo));
            } catch (IOException e) {
                throw new BaseException("websocket推送消息失败");
            }
        });
    }

}
```

消息通知vo

```Java
package com.zzyl.nursing.vo;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

/**
 * 报警通知消息对象
 *
 **/
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class AlertNotifyVo {
    /**
     * 报警数据id
     */
    private Long id;

    /**
     * 接入位置
     */
    private String accessLocation;

    /**
     * 位置类型 0：随身设备 1：固定设备
     */
    private Integer locationType;

    /**
     * 物理位置类型 0楼层 1房间 2床位
     */
    private Integer physicalLocationType;

    /**
     * 设备位置
     */
    private String deviceDescription;

    /**
     * 产品名称
     */
    private String productName;

    /**
     * 功能名称
     */
    private String functionName;

    /**
     * 数据值
     */
    private String dataValue;

    /**
     * 报警数据类型，0：老人异常数据，1：设备异常数据
     */
    private Integer alertDataType;

    /**
     * 语音通知状态，0：关闭，1：开启
     */
    private Integer voiceNotifyStatus;

    /**
     * 报警通知类型，0：解除报警，1：报警
     */
    private Integer notifyType;

    /**
     * 是否全员通知<br>
     * 智能床位的报警消息是全员通知，对于护理员和固定设备维护人员不是全员通知
     */
    private Boolean isAllConsumer;
}
```



#### 报警数据推送到前端

修改AlertRuleServiceImpl类，添加推送数据逻辑

```Java
/**
 * 设备数据匹配过滤规则
 * @param deviceData
 * @param rule
 */
private void deviceDataAlarmHandler(DeviceData deviceData, AlertRule rule) {

    // 判断上报的数据是否在生效时段内  00:00:00~23:59:59    00:05:00~22:59:59
    String[] split = rule.getAlertEffectivePeriod().split("~");
    LocalTime startTime = LocalTime.parse(split[0]);
    LocalTime endTime = LocalTime.parse(split[1]);
    // 数据上报的时间
    LocalTime localTime = deviceData.getAlarmTime().toLocalTime();
    // 如果上报的时间不在生效时段内，则结束请求
    if(localTime.isBefore(startTime) || localTime.isAfter(endTime)){
        return;
    }

    // 统计次数的key
    String aggCountKey = CacheConstants.IOT_COUNT_ALERT+deviceData.getIotId()+":"+deviceData.getFunctionId()+":"+rule.getId();

    // 判断上报的数据是否达到了规则的阈值
    Double dataValue = Double.valueOf(deviceData.getDataValue());
    Double value = rule.getValue();
    // 工具类x,y（顺序有要求，左边是上报的数据，后边是规则定义的数据）  x等于y 返回0  x>y  返回大于0的值   x<y  返回小于0的值
    int compare = NumberUtil.compare(dataValue, value);
    if((rule.getOperator().equals(">=") && compare >= 0) || (rule.getOperator().equals("<") && compare < 0)) {
        // 符合上报的规则，产生了异常数据
        log.info("当前数据符合报警规则");
    } else {
        log.info("正常上报的数据");
        redisTemplate.delete(aggCountKey);
        return;
    }

    // 沉默周期  持续周期
    // 设计一个redis的可以，必须唯一，代表的当前的设备、物模型、规则ID
    String silentKey = CacheConstants.IOT_SILENT_ALERT+deviceData.getIotId()+":"+deviceData.getFunctionId()+":"+rule.getId();
    // 获取沉默周期
    String silentData = redisTemplate.opsForValue().get(silentKey);
    if(StringUtils.isNotEmpty(silentData)) {
        return;
    }
    // 持续周期

    String aggCountData = redisTemplate.opsForValue().get(aggCountKey);
    Integer count = StringUtils.isEmpty(aggCountData)? 1 : (Integer.parseInt(aggCountData) + 1);
    // 当前count不等于持续周期，就累加数据，并且结束请求
    if(ObjectUtil.notEqual(count,rule.getDuration())) {
        // 累加数据
        redisTemplate.opsForValue().set(aggCountKey,count+"");
        return;
    }
    // 到了报警的条件了，保存一份沉默周期
    redisTemplate.opsForValue().set(silentKey,"1", rule.getAlertSilentPeriod(), TimeUnit.MINUTES);
    // 删除统计的次数
    redisTemplate.delete(aggCountKey);

    // 保存异常数据
    // 判断上报数据的设备的类型，如果老人的异常数据（手表、睡眠检测带） 设备异常（烟雾报警）
    List<Long> userIds = new ArrayList<>();
    if(rule.getAlertDataType().equals(0)) {
        // 老人异常（手表、睡眠检测带）  设备id-->设备-->老人id--->护理员
        if(deviceData.getLocationType().equals(0)) {
            // 随身设备
            userIds = deviceMapper.selectNursingIdsByIotIdWithElder(deviceData.getIotId());
        }else if(deviceData.getLocationType().equals(1) && deviceData.getPhysicalLocationType().equals(2)){
            // 床位设备  设备id-->设备-->床位-->老人id--->护理员
            userIds = deviceMapper.selectNursingIdsByIotIdWithBed(deviceData.getIotId());
        }
    }else {
        // 设备异常  找维修人员   通过角色名称（维修工）   用户  角色  用户角色中间表
        userIds = sysUserRoleMapper.selectByRoleName("维修工");
    }
    // 找到超级管理员
    List<Long> managerIds = sysUserRoleMapper.selectByRoleName("超级管理员");
    // 合并两份用户id
    Collection<Long> allUserIds = CollUtil.addAll(userIds, managerIds);
    // 去重
    allUserIds = CollUtil.distinct(allUserIds);

    // 批量保存异常数据
    List<AlertData> alertDataList = insertAlertData(allUserIds, deviceData, rule);

    // websocket推送消息
    websocketNotity(alertDataList.get(0), rule, allUserIds);


}

@Autowired
private WebSocketServer webSocketServer;

/**
 * websocket推送消息
 * @param alertData
 * @param rule
 * @param allUserIds
 */
private void websocketNotity(AlertData alertData, AlertRule rule, Collection<Long> allUserIds) {

    // 属性拷贝
    AlertNotifyVo alertNotifyVo = BeanUtil.toBean(alertData, AlertNotifyVo.class);
    alertNotifyVo.setFunctionName(rule.getFunctionName());
    alertNotifyVo.setAlertDataType(rule.getAlertDataType());
    alertNotifyVo.setNotifyType(1);
    // 向指定的人推送消息
    webSocketServer.sendMessageToConsumer(alertNotifyVo, allUserIds);

}

@Autowired
private IAlertDataService alertDataService;

/**
 * 保存异常数据
 * @param allUserIds
 * @param deviceData
 * @param rule
 */
private List<AlertData> insertAlertData(Collection<Long> allUserIds, DeviceData deviceData, AlertRule rule) {

    // 属性拷贝，从deviceData拷贝到alertData
    AlertData alertData = BeanUtil.toBean(deviceData, AlertData.class);
    // 关于规则的数据都拷贝不过去
    alertData.setAlertRuleId(rule.getId());
    // 功能名称+运算符+阈值+持续周期+聚合周期
    String reason = CharSequenceUtil.format("{}{}{},持续了{}周期，就报警",rule.getFunctionId(),
            rule.getOperator(),rule.getValue(),rule.getDuration());
    alertData.setAlertReason(reason);
    // 报警状态
    alertData.setStatus(0);
    alertData.setType(rule.getAlertDataType());
    // 批量保存数据了，由于多个人
    List<AlertData> list = allUserIds.stream().map(userId -> {
        // 再次拷贝
        AlertData dBalertData = BeanUtil.toBean(alertData, AlertData.class);
        dBalertData.setUserId(userId);
        dBalertData.setId(null);
        return dBalertData;

    }).collect(Collectors.toList());
    // 批量保存
    alertDataService.saveBatch(list);

    return list;
}
```

由于前端也要发请求到后端建立连接，需要在spring security中对于**ws**的请求放行

放行代码

![y7dpbqrpsommstxvucdcoozdnbh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/y7dpbqrpsommstxvucdcoozdnbh.png)
> 图注：这张图片展示的是Spring Security的安全配置代码页面，内容为设置请求的权限规则，其中用红色框和箭头重点标注了需放行的路径“/ws/**”。在配置的请求授权规则中，针对不同路径设置了对应权限，明确了需要放行的请求内容，该配置的作用是放开对“/ws”相关请求的权限，以此配合WebSocket功能的实现，确保前端能够与后端建立WebSocket连接，这正是文档中提到的在Spring Security中对ws请求进行放行的具体代码实现。



#### 前端环境配置

用资料中提供的index.vue替换src/layout目录下的index.vue，

修改前端的ws的连接地址：

![mkvubiy45o3adwx9rtwch7jdnkh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mkvubiy45o3adwx9rtwch7jdnkh.png)
> 图注：图片展示了在VUE3项目中修改前端ws连接地址的操作。左侧是项目的文件结构，右侧是.env.test文件内容。文件中VITE_APP_SOCKET_URL变量被修改为“ws://localhost:8080”，并有红色箭头指向该变量及文件。图片与上下文的关系是，上下文提到用资料中提供的index.vue替换src/layout目录下的index.vue，修改前端的ws连接地址，此图直观呈现了修改后的.env.test文件内容，强调了两个文件都需修改为后端服务地址。

**注意：是ws不是wss**

#### 测试

- 创建一个报警规则，启动模拟上报符合规则的物模型数据
- 查看上报数据需要通知的人

  - 使用任意一个被通知的人登录后台管理系统
  - 当有异常数据之后会在前端进行弹窗提示

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第四章-智能监测系统（华为云） / 报警通知提醒 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/SDN9wC9lyiQZFHkI6FxcbvIXnad)
