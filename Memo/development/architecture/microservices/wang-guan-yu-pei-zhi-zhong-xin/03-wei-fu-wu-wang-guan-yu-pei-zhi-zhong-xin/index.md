---
type: memo
date: 2026-11-06
aliases:
  - "03 微服务网关与配置中心"
---
# 03 微服务网关与配置中心

## 学习目标

1. 能够说出用什么实现的网关以及实现了哪些功能
2. 能够创建网关工程实现路由功能
3. 能够使用网关内置过滤器StripPrefix
4. 能够定义全局过滤器并测试通过
5. 能够实现全局身份校验过滤器
6. 能够在微服务实现用户身份拦截器
7. 能够说出网关鉴权具体的实现步骤
8. 能够实现商城项目前后端联调
9. 能够将微服务配置文件在Nacos统一管理
10. 能够说出微服务配置文件的加载顺序
11. 能够将微服务配置文件抽取公共配置
12. 能够说出Nacos配置热更新方案



## 1 网关路由

<callout emoji="🥖">
1. 很多的微服务,存在很多地址，对客户端不友好，新增或者修改ip后，都要前端去适配
2. 用户校验逻辑该放在哪里，如果每个服务都做校验又存在代码冗余，有没有一个统一的地方
</callout>

### 1.1.认识网关

#### 1.1.1 问题分析

到目前为止我们已经将黑马商城单体项目重构为微服务架构，今天的目标是进行**前后端联调**，下边思考几个问题：

##### 1.1.1.1 **前端面对多个后端入口**

项目采用单体架构时前端通过nginx负载均衡访问后端服务，如下图：

<whiteboard token="AtFow3JhlhMpBZba0iOcAZkmncf"></whiteboard>

项目采用微服务架构时原来的黑马商城分成了购物车服务、交易服务、支付服务等多个服务，前端面对多个后端入口 ，如下图：

<whiteboard token="XyDOwtHMYhvMGrbqu4ecyKsRnxe"></whiteboard>

前端面对多个后端入口不方便前端开发，效率低下。

仍然可以采用nginx去解决，如下图：

<whiteboard token="CyGLwAPiihO0aMbDLr7cDGVsnAh"></whiteboard>

在nginx中创建多个upstream ，例如：

```JavaScript
http {
    upstream item_services {
        server 127.0.0.1:8081 weight=3;  # 分配较高权重
        server 127.0.0.1:8082 weight=2;  # 分配中等权重
        server 127.0.0.1:8083 weight=1;  # 分配较低权重
    }
    upstream carts_services {
        server 127.0.0.1:7081 weight=3;  # 分配较高权重
        server 127.0.0.1:7082 weight=2;  # 分配中等权重
        server 127.0.0.1:7083 weight=1;  # 分配较低权重
    }
    ....
    server {
        listen       80;  # 监听 80 端口，也可以根据需要更改
        server_name  localhost;  # 更改为你的域名或 IP 地址

        location /items/ {  # 这里可以根据需要调整路径前缀
            proxy_pass http://item_services;
        }
        location /carts/ {  # 这里可以根据需要调整路径前缀
            proxy_pass http://carts_services;
        }
        ....
   }
```



##### 1.1.1.2 用户身份校验放在哪？密钥安全性呢？

单体架构时我们只需要完成一次用户登录、身份校验，就可以在所有业务中获取到用户信息。而微服务拆分后，每个微服务都独立部署，这就存在一些问题：每个微服务都需要编写身份校验、用户信息获取的接口，非常麻烦。

用户身份校验最好放在一个统一的地方，放在nginx上最合适，nginx的作用如下：

**1.请求路由**，根据请求路径将请求转发到不同的应用服务器。

**2.负载均衡**，通过负载均衡算法将请求转发到不同的应用服务器。

**3.用户身份鉴权**，校验用户身份及用户的权限。

#### 1.1.2 认识网关

nginx的身份就是网关，什么是网关？

顾明思议，网关就是**网**络的**关**口。数据在网络间传输，从一个网络传输到另一网络时就需要经过网关来做数据的**路由和转发以及数据安全的校验**。



我们现在要根据需求使用Java在网关实现路由转发和用户身份认证的功能：

1. 根据请求Url路由到具体的微服务
2. 校验用户的token，取出token中的用户信息。
3. 从nacos中取出服务实例进行负载均衡。

**但 在nginx中进行java编程是非常困难的，所以我们需要一个使用java开发的网关。**

AI:java微服务网关

- Netflix Zuul：早期实现，目前已经淘汰
- Spring Cloud Gateway：基于Spring的WebFlux技术，完全支持响应式编程，吞吐能力更强

课堂中我们以Spring Cloud Gateway为例来讲解，官方网站：

<bookmark name="spring.io" href="https://spring.io/projects/spring-cloud-gateway#learn"></bookmark>

最终我们使用Spring Cloud Gateway实现网关，如下图：

<whiteboard token="BX5GwkULlhOq3tbnhlwcjTGonHe"></whiteboard>

前端请求网关根据请求路径路由到微服务，网关从nacos获取微服务实例地址将请求转发到具体的微服务实例上。

生产环境中网关也是集群部署，在网关前边通过nginx进行负载均衡，如下图：

<whiteboard token="Qtk8wPK4WhcHufbbhm8cIatmnCd"></whiteboard>

Nginx 和 Spring Cloud Gateway 在微服务架构中是**分层协作的核心网关组合**，二者定位清晰、职责互补，核心总结如下：

<callout emoji="📍">
1. **Nginx 是「边缘网关」**：属于基础设施层，是架构最外层的**流量总入口**，聚焦**网络层通用工作**（反向代理、负载均衡、SSL、静态资源、网络防护），支撑 Gateway 集群的高可用；
2. **Spring Cloud Gateway 是「应用网关」**：属于微服务应用层，是微服务内部的**路由与治理中枢**，聚焦**应用层微服务专属工作**（动态路由、统一鉴权、熔断限流、日志、跨域），实现业务微服务的无侵入式治理；
3. **核心协作逻辑**：外部流量先经 Nginx 做网络层处理和 Gateway 集群负载，再经 Gateway 做应用层路由和服务治理，最终转发到业务微服务，全程分层处理、无职责重叠；
4. **设计价值**：双层网关设计实现了**运维与开发的职责分离**、**网络层与应用层的架构隔离**、**分层扩缩容的弹性支撑**，是企业级微服务架构的**生产标准配置**。
</callout>

#### 1.1.3. 面试题

说说Spring Cloud五大组件？

你们项目网关用什么实现，实现了什么功能？



### 1.2. 实现网关路由

接下来，我们先看下如何利用**网关实现请求路由、统一身份鉴权**。由于网关本身也是一个**独立的微服务**，因此也需要创建一个模块，大概步骤如下：

AI:Spring Cloud Gateway实现路由

- 创建网关微服务
- 引入Spring Cloud Gateway、NacosDiscovery、loadbalancer依赖
- 编写启动类
- 配置网关路由



#### 1.2.1. 创建网关工程

首先，我们要在hmall下创建一个新的module，命名为hm-gateway，作为网关微服务



#### 1.2.2 引入依赖

在`hm-gateway`模块的`pom.xml`文件中引入依赖：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>hmall-parent</artifactId>
        <groupId>com.hmall</groupId>
        <version>1.0.0</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>hm-gateway</artifactId>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>
    <dependencies>
        <!--common-->
        <dependency>
            <groupId>com.hmall</groupId>
            <artifactId>hm-common</artifactId>
            <version>1.0.0</version>
        </dependency>
        <!--网关-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <!--nacos 服务注册发现-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--负载均衡-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>
    </dependencies>
    <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```



#### 1.2.3 启动类

在`hm-gateway`模块的`com.hmall.gateway`包下新建一个启动类：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/J33VbCADtoXxwFx7K1gcEUKQntd.png)

代码如下：

```Java
package com.hmall.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}

```



#### 1.2.4 配置路由

接下来，在`hm-gateway`模块的`resources`目录新建一个`application.yaml`文件，内容如下：

```YAML
server:
  port: 8080
spring:
  application:
    name: gateway
  cloud:
    nacos:
      server-addr: 192.168.101.68:8848
    gateway:
      routes:
        - id: item
          uri: lb://item-service
          predicates:
            - Path=/items/**
```

这里配置nacos地址，网关需要从nacos获取微服务的实例地址。

路由规则routes包括四个属性，定义语法如下：

- `id`：路由的唯一标示
- `uri`：路由目标地址，`lb://`代表负载均衡，从注册中心获取目标微服务的实例列表，并且负载均衡选择一个访问。
- `predicates`：路由断言，Predicates 是用于判断请求是否满足特定条件的组件。
- `filters`：路由过滤条件，稍后讲解。



`application.yaml`**文件完整内容如下：**

```YAML
server:
  port: 8080
spring:
  application:
    name: gateway
  cloud:
    nacos:
      server-addr: 192.168.101.68:8848
    gateway:
      routes:
        - id: item # 路由规则id，自定义，唯一
          uri: lb://item-service # 路由的目标服务，lb代表负载均衡，会从注册中心拉取服务列表
          predicates: # 路由断言，判断当前请求是否符合当前规则，符合则路由到目标服务
            - Path=/items/**,/search/** # 这里是以请求路径作为判断规则
        - id: cart
          uri: lb://cart-service
          predicates:
            - Path=/carts/**
        - id: user
          uri: lb://user-service
          predicates:
            - Path=/users/**,/addresses/**
        - id: trade
          uri: lb://trade-service
          predicates:
            - Path=/orders/**
        - id: pay
          uri: lb://pay-service
          predicates:
            - Path=/pay-orders/**

```





#### 1.2.5 测试

启动GatewayApplication，通过网关请求微服务， http://localhost:8080是网关的根路径，根据网关路由的配置请求具体的URL。

例如：

要访问商品服务需要URL以/items开头，访问交易服务需要以/orders开头.

下边访问商品查询的接口地址：

[http://localhost:8080/items/page?pageNo=1&pageSize=1](http://localhost:8080/items/page?pageNo=1&pageSize=1)

启动网关服务、商品服务，访问此链接。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/IiThbblfooVCkXxCaX5cabntndh.png)



#### 1.2.6.路由断言

路由规则的定义语法如下：

```YAML
spring:
  cloud:
    gateway:
      routes:
        - id: item
          uri: lb://item-service
          predicates:
            - Path=/items/**,/search/**
```



这里我们重点关注`predicates`，也就是路由断言。Spring Cloud Gateway中支持的断言类型有很多：

| **名称** | **说明** | **示例** |
|-|-|-|
| After | 是某个时间点后的请求 | - After=2037-01-20T17:42:47.789-07:00[America/Denver] |
| Before | 是某个时间点之前的请求 | - Before=2031-04-13T15:14:47.433+08:00[Asia/Shanghai] |
| Between | 是某两个时间点之前的请求 | - Between=2037-01-20T17:42:47.789-07:00[America/Denver], 2037-01-21T17:42:47.789-07:00[America/Denver] |
| Cookie | 请求必须包含某些cookie | - Cookie=chocolate, ch.p |
| **Header** | 请求必须包含某些header | - Header=X-Request-Id, \d+ |
| Host | 请求必须是访问某个host（域名） | - Host=\*\*.somehost.org,\*\*.anotherhost.org |
| Method | 请求方式必须是指定方式 | - Method=GET,POST |
| Path | 请求路径必须符合指定规则 | - Path=/red/{segment},/blue/\*\* |
| Query | 请求参数必须包含指定参数 | - Query=name, Jack或者- Query=name |
| RemoteAddr | 请求者的ip必须是指定范围 | - RemoteAddr=192.168.1.1/24 |
| weight | 权重处理 |  |

拿**Header**举例，Header是根据请求头的内容来控制网关访问的。

Header需要两个参数header和regexp（正则表达式），也可以理解为Key和Value，匹配请求携带信息。



修改网关的配置文件application.yml，如下：

Header中包括X-Request-Id才可正常访问，`X-Request-Id` 并不是HTTP标准的一部分，常用用于标识一次请求的ID。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/Yxl0b2jGko63gIxdHwScP7AOnIg.png)

重启网关，访问http://localhost:8080/items/page?pageNo=1&pageSize=1，不可以正常访问，因为没有添加X-Request-Id请求头。

下边我们用idea的httpclient插件测试，使用该插件可以方便的添加http请求头信息。

创建一个请求：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/Iulcbk8o0oopfUx6mNncYJHynOb.png)

在下边的界面中输入请求url并设置X-Request-Id请求头

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/LhXibiuTiovqk5xA5bKclMcJnVe.png)

点击“运行”可以正常访问。



如果IDEA没有HTTP client插件请自行安装

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/AWFpb0vGzowJGjxo2dZczSoMnJd.png)



### 1.3. 小结

我们使用Spring Cloud Gateway创建网关实现了路由、负载均衡的功能。

1. 网关路由

在application.yaml中配置网关路由，格式如下：

```JavaScript
- id: product
  uri: lb://item-service
  predicates:
    - Path=/product/**
  filters:
```

id: 路由规则id，自定义，唯一

uri: 路由的目标服务，lb代表负载均衡，会从注册中心拉取服务列表

predicates:  路由断言，判断当前请求是否符合当前规则，符合则转发到目标服务

 Path：符合请求路径执行此路由

filters: 网关过滤器(稍后讲解)

1. 负载均衡

请求首先到达网关，网关从nacos获取服务实例列表，通过Spring Cloud LoadBalancer负载均衡器选取一个服务实例，请求转发到该服务实例上。



## 2 网关鉴权-通过自定义过滤器实现

### 2.1.认识网关鉴权

单体架构时我们只需要完成一次用户登录、身份校验，就可以在所有业务中获取到用户信息。而微服务拆分后，每个微服务都独立部署，不再共享数据。也就意味着每个微服务都需要做身份校验，这显然不可取。

我们的登录是基于JWT来实现的，校验JWT的算法复杂，而且需要用到密钥。如果每个微服务都去做身份校验，这就存在着两大问题：

- **每个微服务都需要知道JWT的密钥，不安全。**
- **每个微服务重复编写身份校验代码、权限校验代码，代码重复不易维护。**



既然网关是所有微服务的入口，一切请求都需要先经过网关。我们完全可以把身份校验的工作放到网关去做，这样之前说的问题就解决了：

- 只需要在网关和用户服务保存秘钥
- 只需要在网关开发身份校验功能

**网关认证**是指在网关对请求进行身份验证的过程。这个过程确保只有经过授权的用户或设备才能访问特定的服务或资源。

下图是网关**认证**的流程图：

<whiteboard token="ONEUwaXQah07uHbyhsGcPH5AnGe"></whiteboard>

流程如下：

1. 用户登录成功生成token并存储在前端
2. 前端携带token访问网关
3. 网关解析token中的用户信息，网关将请求转发到微服务，转发时携带用户信息
4. 微服务从http头信息获取用户信息
5. 微服务之间远程调用使用内部接口（无状态接口）



网关鉴权除了验证token的合法性还有一层含义是校验用户的权限，通常校验用户的权限不放在网关而是放在微服务去实现，因为具体的接口在微服务，在token中包括了用户的权限字符串，微服务接收到权限字符串通过spring security框架进行拦截实现，具体的方案就是在controller接口上通过下边的注解实现：

`hasAuthority('authority')`: 检查是否有指定的权限（authority），这通常与数据库中的权限字符串对应



可通过AI学习spring security框架：

AI:spring security授权注解有哪些

AI:@PreAuthorize("hasRole('ADMIN')") 除了hasRole还有哪些



这些问题将在接下来几节一一解决。



### 2.2.网关内置过滤器  -学习全局过滤器和路由过滤器

#### 2.2.1 认识网关过滤器-**过滤、拦截请求**

网关鉴权必须在请求转发到微服务之前做，否则就失去了意义。而网关的请求转发是`Gateway`内部代码实现的，要想在请求转发之前做身份校验，就必须了解`Gateway`内部工作的基本原理。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/AWqdb6P4Po0V3xxzCxgcYtEUnhf.png)



如图所示：

1. 客户端请求进入网关后由`HandlerMapping`对请求做判断，找到与当前请求匹配的路由规则（**`Route`**），然后将请求交给`WebHandler`去处理。
2. `WebHandler`则会加载当前路由下需要执行的过滤器链（**`Filter chain`**），然后按照顺序逐一执行过滤器（后面称为**`Filter`**）。
3. 图中`Filter`被虚线分为左右两部分，是因为`Filter`内部的逻辑分为`pre`和`post`两部分，分别会在请求路由到微服务**之前**和**之后**被执行。
4. 只有所有`Filter`的`pre`逻辑都依次顺序执行通过后，请求才会被路由到微服务。
5. 微服务返回结果后，再倒序执行`Filter`的`post`逻辑。
6. 最终把响应结果返回。



如图中所示，最终请求转发是有一个名为`NettyRoutingFilter`的过滤器来执行的，而且这个过滤器是整个过滤器链中顺序最靠后的一个。**如果我们能够定义一个过滤器，在其中实现身份校验逻辑，并且将过滤器执行顺序定义到`NettyRoutingFilter`之前**，这就符合我们的需求了！



那么，该如何实现一个网关过滤器呢？

**网关过滤器链中的过滤器有两种**：

<callout emoji="📚">
-  **`GlobalFilter`**：全局过滤器，作用范围是所有路由，不可配置。

  - 实现全局过滤器接口。
  - 实现接口方法。
- **`GatewayFilter`**：路由过滤器，作用范围比较灵活，针对所有请求生效.

  - 继承AbstractGatewayFilterFactory，实现抽象方法。
  - 名字必须是以GatewayFilterFactory结尾。
</callout>



### 核心区别总览（表格更清晰）

<sheet sheet-id="OK2RCW" token="Vs6ksxBbmhTb8etzZkMc2AkNnne"></sheet>



其实`GatewayFilter`和`GlobalFilter`这两种过滤器的方法签名完全一致：

```Java
/**
 * 处理请求并将其传递给下一个过滤器
 * @param exchange 当前请求的上下文，其中包含request、response等各种数据
 * @param chain 过滤器链，基于它向下传递请求
 * @return 根据返回值标记当前请求是否被完成或拦截，chain.filter(exchange)就放行了。
 */
Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain);
```

`FilteringWebHandler`在处理请求时，会将`GlobalFilter`装饰为`GatewayFilter`，然后放到过滤器链中，排序以后依次执行。

#### 2.2.2 **内置过滤器**

`Gateway`中内置了很多的`GatewayFilter`，详情可以参考官方文档：

<bookmark name="Spring Cloud Gateway" href="https://docs.spring.io/spring-cloud-gateway/docs/3.1.7/reference/html/#gatewayfilter-factories"></bookmark>

见下表：

<sheet sheet-id="7TShxY" token="Vs6ksxBbmhTb8etzZkMc2AkNnne"></sheet>

内置过滤器有很多，具体在工作中根据需求去使用即可。

下边演示一个非常有用的过滤器  StripPrefix

它的作用：移除路径前缀。

比如：为了路径的统一，我们规定所有请求以/product/开头的全部路由到商品服务，

网关的路由配置如下

```Java
- id: product
  uri: lb://item-service
  predicates:
    - Path=/product/**
```

比如访问商品分页查询接口，访问网关的路径为/product/items/page， 实际的下游服务器地址是 /items/page，如果使用上边的路由配置是无法实现的。

原因是：

请求：http://localhost:8080/product/items/page?pageNo=1&pageSize=1

路由到：http://localhost:8081/product/items/page?pageNo=1&pageSize=1



正确的应该路由到：http://localhost:8081/items/page?pageNo=1&pageSize=1

可以使用StripPrefix过滤器实现

下边配置一个新的路由：

```JavaScript
- id: product
  uri: lb://item-service
  predicates:
    - Path=/product/**
  filters:
    - StripPrefix=1
```

StripPrefix=1表示去除一级路径前缀

使用StripPrefix=1后

请求：http://localhost:8080/product/items/page?pageNo=1&pageSize=1

路径到：http://localhost:8081/items/page?pageNo=1&pageSize=1（正确）



如果配置StripPrefix=2，最终转发到商品服务的路径为/page?pageNo=1&pageSize=1（此路径在商品服务不存在）



### 2.3. 自定义过滤器

无论是`GatewayFilter`还是`GlobalFilter`都支持自定义，只不过**编码**方式、**使用**方式略有差别。

<callout emoji="💡">
重点： 
实现全局过滤器-演示效果
实现路由过滤器-演示效果
</callout>

#### 2.3.1 自定义GlobalFilter

##### 2.3.2.1 编码实现

自定义GlobalFilter简单很多，直接实现GlobalFilter即可，而且也无法设置动态参数：

```Java
package com.hmall.gateway.filter;

import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

/**
 * @author Mr.M
 * @version 1.0
 * @description 全局过滤器
 * @date 2024/8/7 15:10
 */
@Component
@Slf4j
public class PrintAnyGlobalFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 编写过滤器逻辑
        log.info("打印全局过滤器");
        // 放行
         return chain.filter(exchange);

        // 拦截
//        ServerHttpResponse response = exchange.getResponse();
//        response.setRawStatusCode(401);
//        return response.setComplete();
    }

    @Override
    public int getOrder() {
        // 过滤器执行顺序，值越小，优先级越高
        return 0;
    }
}
```

全局过滤器不用在路由中配置。







#### 2.3.2 自定义GatewayFilter(自学)

##### 2.3.2.1 定义GatewayFilter

自定义`GatewayFilter`不是直接实现`GatewayFilter`，而是继承`AbstractGatewayFilterFactory`。最简单的方式是这样的：

<callout emoji="❗">
**注意**：该类的名称一定要以`GatewayFilterFactory`为后缀！
</callout>

```Java
package com.hmall.gateway.filter;

import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.OrderedGatewayFilter;
import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

/**
 * @author Mr.M
 * @version 1.0
 * @description 第一个网关过滤器
 * @date 2024/8/7 14:41
 */
@Component
@Slf4j
public class FirstFilterGatewayFilterFactory extends AbstractGatewayFilterFactory<Object> {

    @Override
    public GatewayFilter apply(Object config) {
        return new GatewayFilter() {
            @Override
            public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
                ServerHttpRequest request = exchange.getRequest();

                log.info("请求路径：{}",request.getPath());
                log.info("网关过滤器FirstFilterGatewayFilterFactory执行啦...");
                //放行
                return chain.filter(exchange);
                //拦截 返回401状态码
                //exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
                //return exchange.getResponse().setComplete();
            }
        };
    }
    

}
```



##### 2.3.2.2 配置过滤器

下边的配置仅在product路由中有效

```JavaScript
- id: product
  uri: lb://item-service
  predicates:
    - Path=/product/**
  filters:
    - StripPrefix=1
    - FirstFilter # 此处直接以自定义的GatewayFilterFactory类名称前缀类声明过滤器
```

配置完成重启网关，访问http://localhost:8080/product/items/page?pageSize=1&pageNo=1  ，观察控制台打出“请求路径...”日志，说明过滤器成功执行。



下边的配置在所有路由中都有效：

```YAML
spring:
  cloud:
    gateway:
      default-filters:
        - FirstFilter # 此处直接以自定义的GatewayFilterFactory类名称前缀类声明过滤器
```

请自行测试。



#### 2.3.3 小结



- **`GatewayFilter`**：路由过滤器作用范围比较灵活，可以是任意指定的路由`Route`. 继承`AbstractGatewayFilterFactory`，并在路由配置中指定过滤器。过滤器的名称规则：以`GatewayFilterFactory`作为后缀。
- **`GlobalFilter`**：全局过滤器作用范围是所有路由，不可配置。实现GlobalFilter接口。实现Ordered 接口可以指定过滤器顺序，实现getOrder()方法，返回值越小优先级越好。
- **`default-filters`**:配置默认过滤器，也是全局生效

默认过滤器适合简单通用处理（加请求头、去前缀），复杂全局逻辑（鉴权、限流）优先用 `GlobalFilter`。





### 2.4.身份校验过滤器

接下来，我们就利用自定义`GlobalFilter`来完成身份校验。

<callout emoji="✏️">
步骤

1. 复制身份验证的配置类、工具类、密钥
2. 复制配置文件
3. 编写AuthGlobalFilter全局过滤器
4. 启动网关服务、购物车服务、用户服务验证
</callout>

#### 2.4.1 JWT工具类

身份校验需要用到JWT，而且JWT的加密需要秘钥和加密工具。这些在`hm-service`中已经有了，我们直接拷贝过来：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/W87ibLaj7oIgycxZAeDcMGvGnGb.png)

具体作用如下：

- `AuthProperties`：配置身份校验需要拦截的路径，因为不是所有的路径都需要登录才能访问
- `JwtProperties`：定义与JWT工具有关的属性，比如秘钥文件位置
- `SecurityConfig`：工具的自动装配
- `JwtTool`：JWT工具，其中包含了校验和解析`token`的功能
- `hmall.jks`：秘钥文件

#### 2.4.2 配置白名单

其中`AuthProperties`和`JwtProperties`所需的属性要在`application.yaml`中配置。

```YAML
hm:
  jwt:
    location: classpath:hmall.jks # 秘钥地址
    alias: hmall # 秘钥别名
    password: hmall123 # 秘钥文件密码
    tokenTTL: 30m # 登录有效期
  auth:
    excludePaths: # 无需身份校验的路径
      - /search/**
      - /users/login
      - /items/**
```

excludePaths配置白名单地址，即无需身份校验的路径。



#### 2.4.3 身份校验过滤器

接下来，我们定义一个身份校验的过滤器：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/DYjeb9dcro0Qp3x7hfWcT7Z3nZc.png)

```JSON
//1. 获取请求实例和响应实例
//2. 获取请求路径
//3. 判断是否可放行(判断当前请求路径是否是白名单,使用antPathMatcher进行匹配)
//4. 获取请求头中的token
//5. 如果Token为空则响应401
//6. 解析Token
    //TODO 7. 放行前设置用户id
    //8. 解析Token成功则放行
    //8. 如果Token解析失败则响应401
```

代码如下：

```Java
package com.hmall.gateway.filter;

import cn.hutool.core.util.StrUtil;
import com.hmall.gateway.config.AuthProperties;
import com.hmall.gateway.utils.JwtTool;
import lombok.RequiredArgsConstructor;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.util.AntPathMatcher;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

/**
 * 身份校验过滤器
 */
@RequiredArgsConstructor
@EnableConfigurationProperties(AuthProperties.class)
@Component
public class AuthGlobalFilter implements GlobalFilter {

    private final AuthProperties authProperties;

    private final JwtTool jwtTool;

    private final AntPathMatcher antPathMatcher = new AntPathMatcher();

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        //1. 获取请求实例和响应实例
        ServerHttpRequest request = exchange.getRequest();
        ServerHttpResponse response = exchange.getResponse();

        //2. 获取请求路径
        String path = request.getURI().getPath();

        //3. 判断是否可放行(判断当前请求路径是否是白名单,使用antPathMatcher进行匹配)
        long count = authProperties.getExcludePaths().stream().filter(x -> antPathMatcher.match(x, path)).count();
        if(count>0) {
            return chain.filter(exchange);
        }

        //4. 获取请求头中的token
        String token = request.getHeaders().getFirst(HttpHeaders.AUTHORIZATION);

        //5. 如果Token为空则响应401
        if(StrUtil.isBlank(token)) {
            response.setStatusCode(HttpStatus.UNAUTHORIZED);
            return response.setComplete();
        }

        //6. 解析Token
        try {
            Long userId = jwtTool.parseToken(token);
            //TODO 7. 放行前设置用户id

            //8. 解析Token成功则放行
            return chain.filter(exchange);
        } catch (Exception e) {
            e.printStackTrace();
            //8. 如果Token解析失败则响应401
            response.setStatusCode(HttpStatus.UNAUTHORIZED);
            return response.setComplete();
        }

    }
}
```



#### 2.4.4 测试



重启网关进行测试。

由于“/items/\*\* ”  在excludePaths中配置，所以访问/items开头的路径在未登录状态下不会被拦截：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/Y5gsbkBnEoYxZmxXxr4cAuF9nIe.png)



访问/carts路径在未登录状态下请求会被拦截，并且返回`401`状态码(注意：**此时cart服务要启动**)：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/JPTkbHV1Ao76yQxKB2rcdLTEn4e.png)

首先通过user-service的swagger文档测试登录接口，拿到token，如下图：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/XOFLb5Jrdop6yNxrECwcF8JWnig.png)



接下来使用httpclient插件进行测试，在请求头中添加token再测试，可以正常访问。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/VwJPbaF4Io53ocxKGDlc84vmnwb.png)



#### 2.4.5 小结

网关身份校验过滤器怎么实现？

我们项目中网关身份校验过滤器使用Spring cloud Gateway的GlobalFilter实现，GlobalFilter是一种全局过滤器。

实现过程如下：

1. 配置密钥、白名单 等相关信息。
2. 编写身份校验过滤器，实现GlobalFilter接口。
3. 首先判断请求地址是否是白名单地址，如果是则放行
4. 取出http头中的token，然后校验token的合法性
5. 如果token合法则将token中的用户信息向下传给微服务
6. 如果token不合法则拒绝访问。



### 2.5 网关传递用户信息

#### 2.5.1. 思路分析

现在，网关已经可以完成身份校验并获取登录用户身份信息。但是当网关将请求转发到微服务时，微服务又该如何获取用户身份呢？

由于网关发送请求到微服务依然采用的是`Http`请求，因此我们可以将用户信息以请求头的方式传递到下游微服务。

然后微服务可以从请求头中获取登录用户信息。考虑到微服务内部可能很多地方都需要用到登录用户信息，因此我们可以利用SpringMVC的拦截器来实现登录用户信息获取，并存入ThreadLocal，方便后续使用。

流程图如下：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/PC1UbMCALoikQnxtCJJcLKucnae.jpg)



因此，接下来我们要做的事情有：

- 改造网关过滤器，在获取用户信息后保存到请求头，转发到下游微服务
- 编写微服务拦截器，拦截请求获取用户信息，保存到ThreadLocal后放行



#### 2.5.2. 网关解析用户信息并保存

首先，我们修改身份校验拦截器的处理逻辑，保存用户信息到请求头中：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/Cgiyb2tvwoomxoxwBAFcXZ6snNd.png)



#### 2.5.3. 微服务获取用户信息

##### 2.5.3.1 ThreadLocal工具类

在hm-common中已经有一个用于保存登录用户的ThreadLocal工具：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/OjCzbdWbPowBGgxxGpncr2yXn1f.png)

屏蔽返回固定用户id的代码,返回从threadLocal中获取的userId，代码如下：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/EsMPbJQY8oWsQFx0b2vc7U2EnVf.png)



##### 2.5.3.2 编写拦截器

接下来，我们只需要编写拦截器，获取用户信息并保存到`UserContext`，然后放行即可。



由于每个微服务都有获取登录用户的需求，因此拦截器我们直接写在**`hm-common`**中，并写好自动装配。这样微服务只需要引入`hm-common`就可以直接具备拦截器功能，无需重复编写。



我们在`hm-common`模块下定义一个拦截器：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/OhzzbKfaLop1w1xQst6cgMNanLb.png)

具体代码如下：

```Java
package com.hmall.common.interceptor;

import cn.hutool.core.util.StrUtil;
import com.hmall.common.utils.UserContext;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * TODO
 *
 * @Author mr.wu
 * @Date 2025-6-16 21:49
 */
public class UserInfoInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //1. 获取请求头中的userId
        String userId = request.getHeader("user-info");

        //2. 如果非空则设置到ThreadLocal中
        if(StrUtil.isNotBlank(userId)) {
            UserContext.setUser(Long.valueOf(userId));
        }

        //3. 放行
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        UserContext.removeUser();//释放用完的用户ID，避免内存泄漏
    }
}
```

###### 注意ThreadLocal内存泄漏问题

AI: ThreadLocal数据用完一定要移除，可以避免内存泄漏

> 当一个线程长时间运行并且没有清除 `ThreadLocal` 中的引用时，即使应用程序不再需要该数据，垃圾回收器也无法回收这些对象，因为 `ThreadLocal` 仍然持有对它们的强引用。这可能会导致内存泄漏。
> 
> 为了避免这种情况，可以在数据使用完毕后显式地从 `ThreadLocal` 中移除引用。这可以通过调用 `ThreadLocal` 的 `remove()` 方法来实现。



##### 2.5.3.3 配置拦截器

接着在`hm-common`模块下编写`SpringMVC`的配置类，配置登录拦截器：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/CQ0Xb4c82oFD9gx34wdcWe7hn5f.png)

具体代码如下：

```Java
package com.hmall.common.config;

import com.hmall.common.interceptors.UserInfoInterceptor;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.DispatcherServlet;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
@ConditionalOnClass(DispatcherServlet.class)
public class MvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new UserInfoInterceptor());
    }
}
```



不过，需要注意的是，这个配置类默认是不会生效的，因为它所在的包是`com.hmall.common.config`，与其它微服务的扫描包不一致，无法被扫描到，因此无法生效。

基于SpringBoot的自动装配原理，我们要将其添加到`resources`目录下的`META-INF/spring.factories`文件中：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/Wx7JbG6D9oTe0Tx7l5ecPui8nGV.png)

内容如下：

```Properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.hmall.common.config.MyBatisConfig,\
  com.hmall.common.config.JsonConfig,\
  com.hmall.common.config.MvcConfig
```



##### 2.5.3.4 测试

重启购物车服务。

重新进行用户登录拿到最新的token。

在拦截器中打断点，请求/carts 观察是否可以获取用户id





#### 2.5.4. 购物车获取用户信息

##### 2.5.4.1 编码实现

之前我们无法获取登录用户，所以把购物车服务的登录用户写死了，现在需要恢复到原来的样子。

找到`cart-service`模块的`com.hmall.cart.service.impl.CartServiceImpl`：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/Y0PDbVkVPoFMn9xAya9c6Kthnnf.png)

修改其中的`queryMyCarts`方法：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/Nrz1bMmQ5o0Q3WxhQH2ch8dsnGd.png)

##### 2.5.4.2 测试

使用httpclient测试通过网关访问查询购物车接口，并且在请求头中带上token

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/JxSKbBAwfoWd9JxppAtcIi4Ln5c.png)

在用户身份拦截器中打断点

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/NyDzbHjgmoJ1Cvx56AncN08Dn7g.png)

在查询购物车接口中打断点

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/B95qbrzZZoLDT4x5zYtciqv5n9d.png)

测试到此说明网关已成功将用户信息传入下游服务中。



### 2.6. Feign接口传递用户

#### 2.6.1. 解决方案

##### 2.6.1.1 问题描述

前端发起的请求都会经过网关再到微服务，由于我们之前编写的过滤器和拦截器功能，微服务可以轻松获取登录用户信息。

但有些业务是比较复杂的，请求到达微服务后还需要调用其它多个微服务。比如下单业务，流程如下：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/Aw0FbbeS8ovRtHxD1ffcxGAqnVI.jpg)

下单的过程中，需要调用商品服务扣减库存，调用购物车服务清理用户购物车。而**清理购物车**时必须知道当前登录的用户身份。但是，**订单服务调用购物车时并没有传递用户信息**，购物车服务无法知道当前用户是谁！

##### 2.6.1.2 方案1：OpenFeign拦截器

微服务之间调用是基于OpenFeign来实现的，在进行OpenFeign调用时可以将用户信息放在http头中传递，如下图：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/Y6NhbRRsYowtEkxEwYIcDSEwnBf.png)

借助Feign中提供的一个拦截器接口：`feign.RequestInterceptor`可以实现Feign拦截器

```Java
public interface RequestInterceptor {

  /**
   * Called for every request. 
   * Add data using methods on the supplied {@link RequestTemplate}.
   */
  void apply(RequestTemplate template);
}

```



我们只需要实现这个接口，然后实现apply方法，利用`RequestTemplate`类来添加请求头，将用户信息保存到请求头中。这样一来，每次OpenFeign发起请求的时候都会调用该方法，传递用户信息到微服务，微服务通过拦截器将用户id保存在ThreadLocal中。代码如下：

```Java
package com.hmall.trade.config;

import com.hmall.common.utils.UserContext;
import feign.RequestInterceptor;
import feign.RequestTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Feign调用拦截器配置类
 *
 * @Author mr.wu
 * @Date 2025-6-16 22:09
 */
@Configuration
public class FeignInterceptorConfig {

    /**
     * Feign调用拦截器类
     * @return
     */
    @Bean
    public RequestInterceptor userInfoRequestInterceptor(){
        return new RequestInterceptor() {
            @Override
            public void apply(RequestTemplate requestTemplate) {
                //1. 获取ThreadLocal中的userId
                Long userId = UserContext.getUser();

                //2. 如果非空则设置到请求头，传给被调用的微服务
                if(userId!=null) {
                    requestTemplate.header("user-info",userId.toString());
                }
            }
        };
    }
}
```

通过代码可知：方案1就是在feign远程调用前在http头中添加用户信息，请求到达微服务由微服务拦截器解析出http头中的user-info放入ThreadLocal。

##### 2.6.1.3 方案2: 单独编写清理购物车接口

现在我们讨论的是如何在接口中传递用户信息，这需要先了解有状态接口和无状态接口的区别。

**有状态接口**会在多个请求之间保持一些状态信息。比如：**用户会话**，服务器需要存储有关用户会话的信息，并且可以在后续的请求中使用这些信息。



<whiteboard token="MbVBw1gW0h8HQ9bRfjrc47O6njg"></whiteboard>

举例：

用户端下单接口是有状态的接口，因为需要知道当前下单的用户，用户的身份是否合法。

下图中的接口都是有状态接口

<whiteboard token="VZvow2o0ZhdAIQbWQ3OcDVhtnTd"></whiteboard>

后台商品管理的接口都是有状态的接口，因为需要知道当前去修改商品信息的用户的身份是否合法，是否有权限去修改。



**无状态接口**在每次**请求之间**不会保留任何关于前一次请求的信息，也就是说，每一次请求都是独立的，并且包含该请求所需的所有信息。

举例：

下图中的接口都是无状态接口：

<whiteboard token="T04TwaxLBhc9t0bezeecAxyjnif"></whiteboard>



微服务之间的远程调用接口应该设计为有状态接口还是无状态接口？

我们抽取的微服务具有单一职责，**通用性特点**，要想保证通用性在设计接口时要尽量设计为无状态的，如果设计为有状态接口其它服务去调用你时需要考虑给你传递状态信息，比如：接口在执行前需要校验用户的权限，那别人是不是要给你传递当前用户的权限信息，你的接口功能是清理购物车，只要传入用户id和商品id就可以清理此用户购物车中的商品，但别人还需要额外传入用户的权限，这样接口通用性是很差的。

**所以，针对微服务之间调用的接口进行单独定义，这些接口不通过网关路由仅限微服务之间调用，所以它们叫内部接口，通常内部接口都设计为无状态，这样更能保证通用性。**

**那些提供给用户访问通过网关路由转发的接口叫外部接口，外部接口要根据需求设计有状态或无状态。**



方案1就是调用有状态接口的思路，它通过feign拦截器去传递用户信息，保证接口的状态，虽然当前也能实现需求但扩展性差。

下边的代码并不适应其它场景，比如：有一个定时任务需要调用feign接口，**定时任务**是后台程序定时执行并没有经过微服务的拦截器，所以是无法从ThreadLocal中获取用户Id的。

```Java
package com.hmall.trade.config;

import com.hmall.common.utils.UserContext;
import feign.RequestInterceptor;
import feign.RequestTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Feign调用拦截器配置类
 *
 * @Author mr.wu
 * @Date 2025-6-16 22:09
 */
@Configuration
public class FeignInterceptorConfig {

    /**
     * Feign调用拦截器类
     * @return
     */
    @Bean
    public RequestInterceptor userInfoRequestInterceptor(){
        return new RequestInterceptor() {
            @Override
            public void apply(RequestTemplate requestTemplate) {
                //1. 获取ThreadLocal中的userId
                Long userId = UserContext.getUser();

                //2. 如果非空则设置到请求头
                if(userId!=null) {
                    requestTemplate.header("user-info",userId.toString());
                }
            }
        };
    }
}
```

所以，我们需要再编写一个清理购物车的无状态接口供其它微服务调用。

#### 2.6.2. 清理购物车无状态接口

<whiteboard token="QCSxwpDxwhtraLbxOVGcWX0cnNb"></whiteboard>

开发无状态接口无需依赖用户会话信息，清理购物车接口需要知道清理哪个用户的购物车即可，接口参数传入用户id即可。

##### 2.6.2.1 定义service

定义新接口：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/WepGbyfOEoWilDxtf4Pc2ZwWnbc.png)

定义新接口并修改原有接口实现：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/V0A3bcKfEoEfA8xQr23ctFlQngh.png)

##### 2.6.2.2 定义接口

单独定义inner包存放内部接口。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/Ho4sbQf0AoplozxiY9mcNOX1nMe.png)

代码如下：

```Java
package com.hmall.cart.controller.inner;


import com.hmall.cart.service.ICartService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@Api(tags = "购物车内部接口")
@RestController
@RequestMapping("/inner/carts")
@RequiredArgsConstructor
@Slf4j
public class CartControllerInner {
    private final ICartService cartService;

    @ApiOperation("批量删除购物车中商品")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "userId", value = "用户id"),
            @ApiImplicitParam(name = "ids", value = "购物车条目id集合")
    })
    @DeleteMapping
    public void deleteCartItemByIds(@RequestParam("userId") Long userId,@RequestParam("ids") Collection<Long> ids){
        cartService.removeByItemIds(userId,ids);
    }
}
```

##### 2.6.2.3 定义feign接口

将原有的清理购物车feign接口修改为新开发的无状态接口。

```Java
package com.hmall.api.cart;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.util.Collection;

@FeignClient("cart-service")
public interface CartClient {
        @DeleteMapping("/inner/carts")
        void deleteCartItemByIds(@RequestParam("userId") Long userId,@RequestParam("ids") Collection<Long> ids);
}
```

修改完后需要修改交易服务，改为调用此无状态接口。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/AwUKbaq9ao633IxlvZccicXbnLc.png)



#### 2.6.3. 测试

下边我们测试下单流程，下单成功清理购物车。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/OxXzb06gbownm9xw4PxckVmZn3b.png)

首先通过user-service的swagger文档测试登录接口，拿到token，如下图：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/Bhtlb9RWBoLoOPxKHW7c9Idznog.png)



接下来使用httpclient插件进行测试，通过网关请求交易服务的下单接口，在请求头中添加token。

测试脚本如下：

```Java
###
POST http://localhost:8080/orders
Accept: application/json
Content-Type: application/json
Authorization: eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJ1c2VyIjoxLCJleHAiOjE3Mjk1NjQ3ODB9.eN-T-tJRwBQRPFuXxYL60mYLdtNjtsSpETWzrE-GSJDULS-DK9CpBxjuRkJ6aKnOuBTW6LmTMYb8iMmKoEaxNpF_Y2H8JNbix0OXkzNtcnlge5BJK5LtVkc0v1ShzcZs8XKApHwOHg7nhYmOoVDYEPnuUJ6n8kOvK_tIuSwFheT8FGoabvRAxfSpeEoj9Ap1ewj_G3cXvKOlY0SzcW26EjggJ7R0eB-xGISqQuHDApFKds5nP1F6PV365ROECQvnSwypSkWCsnVECqUeb5i9MLfn9himqIm5NqzQGr7NWVfQ1QGBhjIv354t-YobpgfmCbq9bu4KdS0G1NJJw5K20g

{
  "addressId": 59,
  "details": [
    {
      "itemId": 317578,
      "num": 1
    }
  ],
  "paymentType": 3
}
```

测试时在交易服务创建订单方法中打断点：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/FvJtbjgiDo5c2mxqK53c5ml3nZg.png)

在购物车服务清理购物车接口中打断点，跟踪是否传入用户id。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/LJu8baUEGoHCGJx1hcZcjMovnWc.png)

测试预期结果：下单成功，清理购物车成功。



### 2.7. 面试题

网关鉴权怎么实现的？

<whiteboard token="VHW3wAOANh6EX2bW1bRcB5AXnxg"></whiteboard>

## 3. 商城微服务前后端联调-作业

到目前为止我们将商城项目使用Spring Cloud Alibaba微服务框架开发完成了以下成果：

1.完成了商品服务、购物车服务、交易服务、用户服务、支付服务、网关服务的开发

2.实现远程调用、服务保护

3.部署了网关并实现了网关路由、网关鉴权。

下边我们可以进行前后端联调，前端请求网关，由网关将请求转发到微服务。



查看nginx.conf中商城门户的虚拟目录配置

```JavaScript
    server {
        listen       18080;
        server_name  localhost;
        # 指定前端项目所在的位置
        location / {
            root html/hmall-portal;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        location /api {
            rewrite /api/(.*)  /$1 break;
            proxy_pass http://localhost:8080;
        }
    }
```

前端通过/api访问http://localhost:8080（网关）

 rewrite /api/(.\*)  /\$1 的作用是将请求url进行重写。

举例：

请求：/api/carts，重写为/carts，即最终请求后端的地址为：http://localhost:8080/carts，实现了通过网关访问购物车服务。



需要测试以下功能：

1. 用户登录
2. 搜索商品,输入关键字搜索
3. 添加购物车
4. 下单
5. 余额支付



从登录开始测试

请求网址:http://localhost:18080/api/search/list?key=&pageNo=1&pageSize=20&sortBy=&isAsc=false

请求方法:GET

状态代码: 404 Not Found

解决：将商品服务中的search接口抽取到item-service服务。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/UMJdboFKAoAu1pxZChrctVxSnmb.png)

```Java
package com.hmall.item.controller;


import cn.hutool.core.util.StrUtil;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.hmall.api.item.dto.ItemDTO;
import com.hmall.common.domain.PageDTO;
import com.hmall.item.domain.po.Item;
import com.hmall.item.domain.query.ItemPageQuery;
import com.hmall.item.service.IItemService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Api(tags = "搜索相关接口")
@RestController
@RequestMapping("/search")
@RequiredArgsConstructor
public class SearchController {

    private final IItemService itemService;

    @ApiOperation("搜索商品")
    @GetMapping("/list")
    public PageDTO<ItemDTO> search(ItemPageQuery query) {
        // 分页查询
        Page<Item> result = itemService.lambdaQuery()
                .like(StrUtil.isNotBlank(query.getKey()), Item::getName, query.getKey())
                .eq(StrUtil.isNotBlank(query.getBrand()), Item::getBrand, query.getBrand())
                .eq(StrUtil.isNotBlank(query.getCategory()), Item::getCategory, query.getCategory())
                .eq(Item::getStatus, 1)
                .between(query.getMaxPrice() != null, Item::getPrice, query.getMinPrice(), query.getMaxPrice())
                .page(query.toMpPage("update_time", false));
        // 封装并返回
        return PageDTO.of(result, ItemDTO.class);
    }
}
```



请求网址:http://localhost:18080/api/pay-orders

请求方法:POST

状态代码:503 Service Unavailable

503表示找不到可用服务，说明服务提供方没有上报到nacos中。

解决：启动支付服务实例



请求网址:http://localhost:18080/api/pay-orders/1821135943176835073

请求方法:POST

状态代码:500 Internal Server Error

解决：查看控制台日志进行修复







## 4 配置管理

### 4.1. 认识配置中心

到目前为止我们已经解决了微服务相关的几个问题：

- 微服务远程调用
- 微服务注册、发现
- 微服务请求路由、负载均衡
- 微服务登录用户信息传递



现在依然还有一个重要的问题需要解决，就是配置管理：

1. 微服务存在大量的配置文件，每个微服务会部署多个服务实例，如果配置文件要修改则需要去每个微服务实例上修改，维护成本非常高。

比如：购物车服务，在生产环境中部署了三个服务实例（每个实例是一个docker容器），如果要修改购物车服务的配置文件需要去三个实例上依次修改配置文件，非常麻烦。

1. 微服务的配置文件中有大量重复的配置，维护成本高。

比如：关于数据库链接的配置、日志的配置等在每个微服务中配置内容基本一样，如果要修改它们的内容需要修改每个微服务的配置文件，非常麻烦。



这些问题都可以通过**配置中心**解决，如下图：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/FThfbdyYOo2fxIxx50ZcFvqMnVb.png)

配置中心的服务流程如下： 

1. 微服务的配置文件全部放在配置中心管理
2. 用户在配置中心更新配置信息。 
3. 微服务从配置中心获取最新的配置信息。 

**总得来说，配置中心就是一个统一管理各种应用配置的基础服务组件。**



AI：在微服务项目中常用的配置中心有哪些

Spring Cloud Config、Consul、Zookeeper、Apollo、Nacos等。

这些配置中心都有各自的特点和优势，选择哪个取决于项目的具体需求、团队熟悉程度以及现有的技术栈等因素。例如，如果已经在使用Spring Boot和Spring Cloud框架，那么Spring Cloud Config可能是最自然的选择；而如果更倾向于轻量级且专注于配置管理的方案，则Apollo或Nacos可能更为合适。



Nacos不仅仅是注册中心还是配置中心，nacos可以对微服务的配置文件进行统一管理。我们项目的注册中心使用的是nacos，引入其它配置中心会增加成本，所以我们使用nacos作为配置中心。

微服务的配置统一交给Nacos保存和管理，在Nacos控制台修改配置后，Nacos会将配置变更推送给相关的微服务，并且无需重启即可生效，实现配置热更新。

网关的路由配置因此同样可以基于这个功能实现动态路由功能，无需重启网关即可修改路由配置。



Nacos 提供web管理控制台如下图，方便进行配置管理，还提供包括配置版本跟踪、金丝雀发布、一键回滚配置以及客户端配置更新状态跟踪在内的一系列开箱即用的配置管理特性，帮助您更安全地在生产环境中管理配置变更和降低配置变更带来的风险。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/QD9nbn8ThoN01Nx585Hc5iPtn5c.png)



### 4.2 快速入门

#### 4.2.1. 发布配置

##### 4.2.1.1 创建命名空间

首先创建命名空间。

命名空间用于隔离不同的项目或环境（开发、测试、生产）

创建命名空间：hmallv1.0，表示商城项目1.0环境，如下图：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/Oey7brUTHoPzP3xJmQ1cEvyenvc.png)

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/RxOJbapxbo63Dpx3HVccxE9Qnmb.png)

**命名空间的ID**: 如果不填会自动生成，非常重要，需要在微服务端配置此命名空间ID。

命名空间名：起一个辨识度高的名称。

描述：对此命名空间的简要说明。



创建成功如下图：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/TpzMbWy9IoaPSCxhQqMcajNFnOh.png)

##### 4.2.1.2 发布配置

下一步我们在nacos上发布购物车服务的配置。

我们切换到刚才创建的命名空间：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/NKkDbc31tozRkVxFTNCcY4FdnQh.png)

点击“创建配置”，填写dataId、group、配置文件等信息。

将application.yaml配置文件中的内容拷贝到下图配置内容栏，将application.yaml文件中原有内容删除。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/SbUxbmJK4oGrrixGz4icVXfknmc.png)



> dataId：配置文件的id，每个配置文件的唯一标识。
> 
>  dataId包括三部分：服务名称、`profile`(可没有)、文件扩展名
> 
>   举例：cart-service-dev.yaml 、cart-service.yaml 
> 
> group: 是配置文件的分组，比如：把数据库相关的配置划分为一个组，把一些公共配置划分为一个组。默认分组为DEFAULT_GROUP。

发布成功如下图：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/YVWEb6FScoZxN0xdVlxcsjtvngb.png)

在resources下创建一个备份目录backup，将原来的配置文件复制到backup目录下进行备份。

此时购物车服务的配置文件如下：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/EZFbbs60AoZaVMxUL0oc8Ug1nkh.png)



#### 4.2.2. 拉取配置

配置文件在nacos发布后下一步微服务就可以从nacos拉取配置。

##### 4.2.2.1 引入依赖

1）引入依赖：

具体谁去拉取配置信息呢？由nacos-client客户端程序，在cart-service模块引入依赖：

在购物车工程引入下边的依赖：

```XML
  <!--nacos配置管理，包括nacos-client等-->
  <dependency>
  
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
  </dependency>
  <!--读取bootstrap文件-->
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-bootstrap</artifactId>
  </dependency>
```

##### 4.2.2.2 创建bootstrap.yaml

nacos客户端从nacos拉取配置信息需要在微服务提前配置nacos的地址。

在resources下创建一个配置文件：`bootstrap.yaml`，在此配置文件中配置nacos的地址，以及命名空间等信息，应用根据此配置从nacos拉取配置文件。

为什么要在`bootstrap.yaml`中配置nacos的地址呢？

SpringCloud在初始化上下文的时候会先读取一个名为`bootstrap.yaml`(或者`bootstrap.properties`)的文件，如果我们将nacos地址配置到`bootstrap.yaml`中，那么在项目引导阶段就可以读取nacos中的配置了。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/QgrUb7dfKoCCqrxEw74cnLWRnnc.jpg)



下边新建bootstrap.yaml

在cart-service中的resources目录新建一个bootstrap.yaml文件：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/EPoAbRjaaorWDoxfaVQcbaGWnYd.png)

内容如下：

```YAML
spring:
  application:
    name: cart-service # 服务名称
  profiles:
    active: dev
  cloud:
    nacos:
      server-addr: ${NACOS_ADDR:192.168.101.68:8848} # nacos地址
      discovery:  #注册中心
        namespace: ${NACOS_NAMESPACE:86654eaf-4819-43fc-93b6-316ba39da7c0}  #命名空间
      config:   #配置中心
        file-extension: yaml # 文件后缀名
        namespace: ${NACOS_NAMESPACE:86654eaf-4819-43fc-93b6-316ba39da7c0}  #命名空间
        group: ${GROUP_NAME:DEFAULT_GROUP}    # 配置分组
```



spring.cloud.nacos.discovery：配置服务注册中心相关的配置，在discovery配置的namespace表示服务注册到此命名空间下，通常服务注册的命名空间和配置文件的命名空间一致。

spring.cloud.nacos.config: 配置中心的相关配置。

file-extension：配置文件扩展名

namespace：配置文件存放的命名空间

group: 配置分组

上述配置中NACOS_ADDR、NACOS_NAMESPACE、GROUP_NAME等参数可以通过启动springboot应用时指定-D进行传入，如果不指定则使用默认值。

注意：命名空间 id 要根据自己nacos上具体的命名空间ID去填写，不要抄讲义的。



nacos客户端从nacos拉取配置会拉取以下名称的配置文件：

dataId 为 ${spring.application.name}.$`{file-extension:yaml}` 

dataId为 ${spring.application.name}-$`{profile}.${file-extension:yaml}` 

举例：

如果购物车服务设置spring.profile.active=dev则会加载下边的配置文件：

cart-service.yaml

cart-service-dev.yaml





##### 4.2.2.3 测试

下边启动购物车服务，启动成功，访问查询购物车接口，我们直接使用Swagger进行测试。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/HLdjbLAhAoNB0IxaubScAQATnxf.png)

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/ZtoQbaioFoCVg8xAwBlcrVf4nJd.png)

测试时报错：

```Java
{
    "timestamp": "2024-10-11T11:43:50.487+00:00",
    "path": "/carts",
    "status": 503,
    "error": "Service Unavailable",
    "requestId": "55b25217-4"
}
```

Service Unavailable：说明从nacos没有找到可用的服务。

遇到这种情况我们要查看nacos，看购物车服务是否在nacos注册成功。

前边我们在bootstrap.yaml中配置了服务发现：

```Java
  spring:
      cloud:
        nacos:
          server-addr: ${NACOS_ADDR:192.168.101.68:8848} # nacos地址
          discovery:  #注册中心
            namespace: ${NACOS_NAMESPACE:86654eaf-4819-43fc-93b6-316ba39da7c0}
```

以上配置指定购物车服务将注册到86654eaf-4819-43fc-93b6-316ba39da7c0命令空间（在没有设置NACOS_NAMESPACE变量的前提下）。

我们需要查看该命令空间是否有购物车服务。

进入服务列表，找到相应的命令空间，查询该命令空间下的服务：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/TXXKbwN6xotVCdxR3Z8cfXz6nuh.png)



现在请求到网关，网关从它所在的命名空间拉取购物车服务实例，所以网关和购物车服务必须在同一个命令空间。

nacos并没有和网关在同一个命令空间。



现在我们**把所有微服务**的服务发现配置全部指定统一的命令空间，添加如下配置：

```Java
  spring:
      cloud:
        nacos:
          server-addr: ${NACOS_ADDR:192.168.101.68:8848} # nacos地址
          discovery:  #注册中心
            namespace: ${NACOS_NAMESPACE:86654eaf-4819-43fc-93b6-316ba39da7c0}
```

修改网关的application.yaml如下图：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/Gh91buOLPoEuw5xhNsWcdj1vnfg.png)

参考上图修改微服务的配置文件，将微服务统一注册到命名空间：86654eaf-4819-43fc-93b6-316ba39da7c0下边。

重启微服务，进入nacos观察命名空间86654eaf-4819-43fc-93b6-316ba39da7c0下是否有微服务。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/AaBdbbNmrozY8wx0TcjcSYnGnLb.png)



现次测试 查看购物车接口，正常显示



### 4.3. nacos数据模型

目前我们实现了nacos注册中心、nacos配置中心，在注册中心nacos去管理各个微服务实例的信息，在配置中心nacos去管理各个微服务的配置文件，nacos是如何去管理这些数据呢？它是如何做到多用户使用且数据隔离呢？本节讲解nacos的领域模型。

在nacos发布配置时我们设置了namespace命名空间、group、dataid等信息，其中namespace是用作数据隔离，group是对数据进一步分组。

我们拿配置管理来说明它的数据模型。

nacos通过namespace、group、dataid去定位一个配置文件，下图是nacos的数据模型（https://nacos.io/zh-cn/docs/architecture.html）：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/GdlTbGTGFo7NsDxiaWNcvYTnnLd.png)

**Namespace：命名空间**

在Nacos中，命名空间用来进行租户级别的隔离，比如：商城项目和家政项目都可以用同一个nacos去管理配置文件，只要他们把各自的配置文件放在自己的命名空间就可以做到互不影响，通常将命名空间用于不同环境的配置的隔离，例如开发环境、测试环境、生产环境是不同的命名空间。

比如：商城项目的配置文件在nacos中管理，我们在nacos中创建的命名空间为：hmall_dev、hmall_pro、hmall_test，分别表示商城项目开发环境、生产环境、测试环境。

如果在发布配置文件时没有指定命名空间默认使用的是 Nacos 上 自带的Public 这个namespace。



**dataId/service：**

DataId表示配置集id，一组配置项的集合称为配置集，配置项就是配置文件中的配置项目，一个配置文件通常就是一个配置集，例如，商城项目的数据库配置在mysql-config.yml中，mysql-config.yml就是一个配置集。

Service: 表示服务，nacos既可以是配置中心也可以是服务注册中心，微服务上报服务地址到nacos服务注册中心，具体也是上报到nacos具体的命名空间下。



**group: 配置分组或服务分组**

Nacos 中的一组配置集，是组织配置的维度之一。通过一个有意义的字符串（如 Buy 或 Trade ）对配置集进行分组，从而区分 Data ID 相同的配置集。比如：把数据库相关的配置划分为一个组，把一些公共配置划分为一个组。在没有明确指定 group的情况下默认使用的是 DEFAULT_GROUP（通常都采用默认分组）。





### 4.4. 配置文件加载顺序

我们引入nacos之后微服务的配置文件可以在nacos上统一管理，并且也引入了bootstrap.yml文件，此时不同的配置文件它们的优先级不同，配置文件的优先级如下:

Bootstrap 配置：(本地配置)

1. bootstrap.yaml
2. bootstrap-{`profile}`.yaml(如果有)

Application 配置 ：(本地配置)

1. application.yaml
2. application-{`profile}`.yaml(如果有)

nacos配置:

1. 扩展配置文件(如果有)
2. 共享配置文件(如果有)
3. {spring.application.name}.yaml 
4. {spring.application.name}-{`profile}`.yaml (如果有)

> **注意：后加载的配置项会覆盖前面的同名配置项的值**



举例：

启动购物车服务设置spring.profile.active=dev按如下顺序加载配置文件：

bootstrap.yaml

application.yaml

application-dev.yaml

cart-service-dev.yaml



#### 4.4.1 面试题

说一下你项目配置文件的加载顺序？



### 4.5. 共享配置

我们可以把微服务共享的配置抽取到Nacos中统一管理，这样就不需要每个微服务都重复配置了。

#### 4.5.1 分析

配置文件中的一些配置可以抽取出来作为共享配置文件。

以cart-service为例，我们看看有哪些配置是重复的，可以抽取的：

首先是jdbc相关配置：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/MWQ4bKtEAo8teox0u55ccVeZnvN.png)

然后是日志配置：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/ATghbDXa7oCvvhxv5NTcwWRFnxc.png)

然后是swagger以及OpenFeign的配置：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/CFHwbPXoJotwJexAANKcZcqDnZe.png)

#### 4.5.2 数据库配置

我们在nacos控制台分别添加这些配置。

首先是jdbc相关配置，在`配置管理`->`配置列表`中先切换到具体的命名空间，点击创建配置。

数据库配置命名为`shared-jdbc.yaml`，配置内容如下：

在弹出的表单中填写信息：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/NBVKbmAnFofRbGx100lcrDYznhg.png)

其中详细的配置如下：

```YAML
spring:
  datasource:
    url: jdbc:mysql://${hm.db.host:192.168.101.68}:${hm.db.port:3306}/${hm.db.database}?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: ${hm.db.un:root}
    password: ${hm.db.pw:mysql}
mybatis-plus:
  configuration:
    default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
  global-config:
    db-config:
      update-strategy: not_null
      id-type: auto
```



注意这里的jdbc的相关参数并没有写死，例如：

- `数据库ip`：通过`${hm.db.host:192.168.101.68}`配置了默认值为`192.168.101.68`，同时允许通过`${hm.db.host}`来覆盖默认值
- `数据库端口`：通过`${hm.db.port:3306}`配置了默认值为`3306`，同时允许通过`${hm.db.port}`来覆盖默认值
- `数据库database`：可以通过`${hm.db.database}`来设定，无默认值



#### 4.5.3 日志配置

然后是统一的日志配置，命名为`shared-log.yaml`，配置内容如下：

```YAML
logging:
  level:
    com.hmall: debug
  pattern:
    dateformat: HH:mm:ss:SSS
  file:
    path: "logs/${spring.application.name}"
```

#### 4.5.4 swagger配置

然后是统一的swagger配置，命名为`shared-swagger.yaml`，配置内容如下：

```YAML
knife4j:
  enable: true
  openapi:
    title: ${hm.swagger.title:黑马商城接口文档}
    description: ${hm.swagger.description:黑马商城接口文档}
    url: https://www.itcast.cn
    version: v1.0.0
    group:
      default:
        group-name: default
        api-rule: package
        api-rule-resources:
          - ${hm.swagger.package}
```

注意，这里的swagger相关配置我们没有写死，例如：

- `title`：接口文档标题，我们用了`${hm.swagger.title}`来代替，将来可以有用户手动指定
- `email`：联系人邮箱，我们用了`${hm.swagger.email:``zhanghuyi@itcast.cn``}`，默认值是`zhanghuyi@itcast.cn`，同时允许用户利用`${hm.swagger.email}`来覆盖。

#### 4.5.5 feign配置

然后是统一的feign配置，命名为shared-feign`.yaml`，配置内容如下：

```JavaScript
feign:
  okhttp:
    enabled: true # 开启OKHttp功能
  sentinel:
    enabled: true # 开启feign对sentinel的支持
```

#### 4.5.6 seata配置

统一配置seata，命名为shared-seata.yaml,内容如下：

```Java
seata:
  registry: # TC服务注册中心的配置，微服务根据这些信息去注册中心获取tc服务地址
    type: nacos # 注册中心类型 nacos
    nacos:
      server-addr: 192.168.101.68:8848 # nacos地址
      namespace: "" # namespace，默认为空
      group: DEFAULT_GROUP # 分组，默认是DEFAULT_GROUP
      application: seata-server # seata服务名称
  tx-service-group: hmall # 事务组名称
  service:
    vgroup-mapping: # 事务组与tc集群的映射关系
      hmall: "default"
```



#### 4.5.7 Sentinel配置

统一配置sentinel，命名为shared-sentinel.yaml,内容如下：

```Java
spring:
  cloud:
    sentinel:
      transport:
        dashboard: 192.168.101.68:9090
      http-method-specify: true # 开启请求方式前缀可根据http请求方法区分簇点链路
```

#### 4.5.7 引用共享配置

在bootstrap.yml中编写如下内容：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/XKfrbQzkuocDunxTSckckBpLn2c.png)

完整配置如下：

```YAML
spring:
  application:
    name: cart-service # 服务名称
  profiles:
    active: dev
  cloud:
    nacos:
      server-addr: ${NACOS_ADDR:192.168.101.68:8848} # nacos地址
      discovery:  #注册中心
        namespace: ${NACOS_NAMESPACE:1e62e732-4a1b-4dc4-ba58-4ba034302c47}  #命名空间
      config:   #配置中心
        file-extension: yaml # 文件后缀名
        namespace: ${NACOS_NAMESPACE:1e62e732-4a1b-4dc4-ba58-4ba034302c47}  #命名空间
        group: ${GROUP_NAME:DEFAULT_GROUP}    # 配置分组
        shared-configs: # 共享配置
          - dataId: shared-jdbc.yaml # 共享mybatis配置
          - dataId: shared-log.yaml # 共享日志配置
          - dataId: shared-swagger.yaml # 共享日志配置
          - dataId: shared-feign.yaml # 共享feign配置
          - dataId: shared-seata.yaml # 共享seata配置
          - dataId: shared-sentinel.yaml # 共享sentinel配置
```



#### 4.5.8 修改应用配置

抽取共享配置后nacos上cart-service-dev.yaml内容如下：

```Java
server:
  port: 8082
spring:
  application:
    name: cart-service
  profiles:
    active: dev
```

下边我们需要修改应用配置文件：application.yaml和cart-service-dev.yaml

**通常将可能会变的配置信息放在nacos，不会变的放在本地。**

不变的配置：

通常端口不变，因为生产环境使用docker部署微服务会部署在自己的容器中。

还有swagger文档的信息不会变。

变化的配置：

数据库的ip和端口可能会变。



application.yaml的内容如下:

```JavaScript
server:
  port: 8082
hm:
  swagger:
    title: 购物车服务接口文档
    package: com.hmall.cart.controller
```

cart-service-dev.yaml内容如下：

```Java
hm:
  db:
    database: hm-cart
    host: 192.168.101.68
    prot: 3306
    un: root
    pw: mysql
```

删除application-dev.yaml和application-local.yaml，删除后本地配置文件如下：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/U2jebjlWMokd1jxBa4ZcgF94nBf.png)

nacos上的配置文件如下：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/EC0TbpxrronYVExC42hcG5K2n4e.png)



#### 4.5.9 测试

下边重启购物车服务 进行测试，测试购物车接口是否可以正常访问。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/AVj2bwC7Voh9l7x618qcsZwwn0d.png)

### 4.6. 网关配置管理

接下来把网关的配置也在nacos管理。

首先， 我们在网关gateway引入依赖：

```XML
<!--统一配置管理-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
<!--加载bootstrap-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

创建bootstrap.yaml，网关只需要shared-log.yaml共享配置，内容如下：

```YAML
spring:
  application:
    name: gateway # 服务名称
  profiles:
    active: dev
  cloud:
    nacos:
      server-addr: ${NACOS_ADDR:192.168.101.68:8848} # nacos地址
      discovery:  #注册中心
        namespace: ${NACOS_NAMESPACE:86654eaf-4819-43fc-93b6-316ba39da7c0}  #命名空间
      config:   #配置中心
        file-extension: yaml # 文件后缀名
        namespace: ${NACOS_NAMESPACE:86654eaf-4819-43fc-93b6-316ba39da7c0}  #命名空间
        group: ${GROUP_NAME:DEFAULT_GROUP}    # 配置分组
        shared-configs: # 共享配置
          - dataId: shared-log.yaml # 共享日志配置
```



在nacos创建应用配置：

由于网关的路由配置在dev、local不同环境下是一样的，这里只创建gateway.yaml即可。

内容如下：

```YAML
spring:
  cloud:
    gateway:
      routes:
        - id: item
          uri: lb://item-service
          predicates:
            - Path=/items/**,/search/**
        - id: cart
          uri: lb://cart-service
          predicates:
            - Path=/carts/**
        - id: user
          uri: lb://user-service
          predicates:
            - Path=/users/**,/addresses/**
        - id: trade
          uri: lb://trade-service
          predicates:
            - Path=/orders/**
        - id: pay
          uri: lb://pay-service
          predicates:
            - Path=/pay-orders/**
        - id: product
          uri: lb://item-service
          predicates:
            - Path=/product/**
          filters:
            - StripPrefix=1

hm:
  jwt:
    location: classpath:hmall.jks
    alias: hmall
    password: hmall123
    tokenTTL: 30m
  auth:
    excludePaths:
      - /search/**
      - /users/login
      - /items/**
      - /hi
```

将原application.yaml备份，修改application.yaml如下：

```Java
server:
  port: 8080
```



重启网关，测试查询购物车等接口是否正常

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/FZlGb8CzCoMf5uxb9CMcieE9nyc.png)

### 4.7.配置热更新



<whiteboard token="FSKPwolJehc8vXbjfAzcpchFndb"></whiteboard>

#### 4.7.1 配置热更新方案

有很多的业务相关参数，将来可能会根据实际情况临时调整。例如购物车业务，购物车数量有一个上限，默认是10，对应代码如下：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/McrGbLvg9oBUgOxFqdJcXYdonAg.png)

现在这里购物车数量限制是硬编码为固定值，我们应该将其配置在配置文件中，后期直接修改配置文件。

如果修改了nacos的配置需要重启微服务才生效，有没有配置热更新的方案。

AI: Spring Boot 集成 nacos后如何实现配置热更新

1.使用@Value注解+`@RefreshScope`

2.使用`@ConfigurationProperties`

课堂上讲解`@ConfigurationProperties`方式，另一种方案自行测试。



#### 4.7.2 添加配置到Nacos

在nacos上`cart-service-dev.yaml`增加以下内容：

```YAML
hm:
  cart:
    maxAmount: 1 # 购物车商品数量上限
```

如下图：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/UNi2b3K1roQUbXxgwjpcZZZgnXe.png)

#### 4.7.3 配置热更新

接着，我们在微服务中读取配置，实现配置热更新。

在`cart-service`中新建一个属性读取类：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/SuhRbOTTBo9rSZxLUSjc4xFInle.png)

代码如下：

```Java
package com.hmall.cart.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Data
@Component
@ConfigurationProperties(prefix = "hm.cart")
public class CartProperties {
    private Integer maxAmount;
}

```



接着，在业务中使用该属性加载类：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/InZTbG9hco7kl4xaO09cO5WinKc.png)

#### 4.7.4 测试

测试流程：

通过商城门户搜索商品

查看我的购物车

将商品添加到我的购物车

准备测试环境需要注意：

网关、购物车服务、商品服务 使用同一个namespace



下边进行测试，向购物车中添加多个商品：

预期结果:

由于配置的购物车商品数量上限是1，所以购物车中最多添加一个商品，否则报错（通过打断点或跟踪控制台日志）。



我们在nacos控制台，将购物车上限配置为5：

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/YmdbbrmZGoYzHlxZeFhci0RWnDS.png)



无需重启购物车服务，再次测试购物车功能，发现最多可向购物车添加5个商品。

![](../../../../../../Attachments/development/architecture/microservices/wang-guan-yu-pei-zhi-zhong-xin/03-wei-fu-wu-wang-guan-yu-pei-zhi-zhong-xin/AtqUbzoG8ohZ1Nxnfobcp7czn6b.png)



#### 4.7.5 面试题

nacos配置可以热更新吗？怎么实现？

## 作业

### 完成前后端联调

要求：

完成用户注册、用户登录、添加购物车、下单、支付功能的前后端联调。

提示：

支付功能：支付服务调用用户服务扣减余额，这里要将用户服务扣减余额接口定义为内部接口供支付服务调用。

### Nacos配置

要求：

将商品服务、交易服务、购物车服务、支付服务、用户服务的配置文件用nacos统一管理。

注意：

商城项目所有的微服务注册至同一个namespace。

逐个微服务进行，改造一个微服务完成后进行测试通过再改造下一个微服务。

在nacos发布配置前先备份原来的配置文件。

测试时直接通过前后端联调的方式进行。





<whiteboard token="PdVrwdStqhrGqSbSB9KcLInunCh"></whiteboard>

---
> 来源：飞书技术知识库（脱敏版） · 04 微服务与中间件 / 网关与配置中心 / 03 微服务网关与配置中心 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/VVJuwxk3OicDoZkE8POcLOIGnOc)
