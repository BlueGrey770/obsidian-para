---
type: memo
date: 2026-11-06
aliases:
  - "第四章 小程序首页"
---
# 第四章 小程序首页

## 学习目标

1. 能说出实现门户的常用技术方案
2. 能说出小程序首页哪些信息要缓存
3. 会使用Spring Cache的常用注解
4. 能说出缓存穿透的解决方案
5. 能说出缓存击穿的解决方案
6. 能说出缓存雪崩的解决方案
7. 能说出保证缓存一致性的方案
8. 能够实现开通区域列表缓存
9. 能够实现定时任务更新缓存
10. 能够说出xxl-job工作原理
11. 能够实现首页服务列表缓存
12. 能够实现服务类型列表缓存
13. 能够实现热门服务列表缓存
14. 能够实现服务详情缓存



家政项目实战：

服务类型列表查询

热门服务列表查询

服务详情查询



## 代码环境

jzo2o-foundations：dev_02



## 1 门户技术方案

### 1）什么是门户

本节分析小程序首页的技术方案。

小程序首页界面原型如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/MiJ4btCgMoKeyxxn8KVce7irn6e.png)

包括如下部分：

1. 搜索栏
2. 服务分类
3. 热门服务项目

首页是用户进入这个小程序的入口，聚合很多的信息，包括内容导航、热点信息等。

类似的界面还有：

百度新闻：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/CG8fbtyHJoQCssxJjdbcnQzhn1D.png)

新浪首页：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/PuUWbmQoWoNvHDxyWKZcx1txnag.png)



这种系统首页入口称为门户，是指一个网站或应用程序的主页，它是用户进入这个网站或系统的入口，主页上通常聚合很多的信息，包括内容导航、热点信息等，比如：门户网站的首页、新闻网站的首页、小程序的首页等。

常见的软件门户类型：

1. 企业门户：这种类型的门户通常用于组织内部，为员工提供一个集中访问企业内部各种应用、数据和服务的地方。它可以包括人力资源信息、财务系统、项目管理工具等。
2. 客户门户：面向外部客户的门户，允许客户登录并访问特定的服务，如账户管理、订单跟踪、技术支持等。
3. 知识门户：这类门户主要用于知识管理和分享，可以包含文档库、论坛、博客等功能，帮助用户获取所需的信息和专业知识。
4. Web门户：这是最广为人知的一种门户形式，它通常提供新闻、电子邮件、搜索引擎等综合性服务，用户可以通过这个平台方便地访问网络上的多种资源。



### 2）常用的技术方案

实现门户功能用到哪些技术呢？

#### **基于两个需求去分析**

**1.门户上的信息是动态的**

门户上的信息会按照一定的时间周期去更新，比如一个新闻网站不可能一直显示一样的新闻。

**2.门户作为入口其访问频率非常高**

对于访问频率高的界面其加载速度是至关重要的，因为它直接影响用户的体验和留存率。一般来说门户网站的首页应该在2至3秒内加载完成，这被认为是一个合理的加载时间目标。

**常见的两类门户是：web门户和移动应用门户。**

我们针对这两类门户分析技术方案。

#### **Web门户技术方案**

web门户是最常见的门户类型，比如：新浪、百度新闻等，它们通过PC浏览器访问，用户可以通过桌面电脑、笔记本电脑、平板电脑和智能手机等设备访问。Web门户通常运行在Web浏览器上，用户可以通过输入网址或通过搜索引擎访问。

web门户是通过浏览器访问html网页，虽然html网页上的内容是动态的但是考虑门户作为入口其访问频率非常高所以就需要提高它的加载速度，如果网页上的数据是通过实时查询数据库得到是无法满足要求的，所以针对web门户提高性能的关键是如何提高html文件的访问性能，如何提高查询数据的性能。

**1、将门户页面生成静态网页发布到CDN服务器。**

纯静态网页通过Nginx加载要比去Tomcat加载快很多。

我们可以使用模板引擎技术将动态数据静态化生成html文件，并通过CDN分发到边缘服务器，可以提高访问效率。

Java模板引擎技术有很多，比如：freemarker、velocity等。

在课下参考入门视频自学: https://www.bilibili.com/video/BV1j8411N7Bm?p=93&vd_source=81d4489ba9312103debc8ee843169f23



什么是CDN？

CDN 是构建在数据网络上的一种分布式的内容分发网，旨在提高用户访问网站或应用时的性能。

下图中，通过CDN将内容分发到各个城市的CDN节点上，北京的网民请求北京的服务即可拿到资源，提高访问速度。

![图片](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Uoz8bbXBVolfcPxbjU5cQfivnqg.png)



**2、html文件上的静态资源比如：图片、视频、CSS、Js等也全部放到CDN服务。**

**3、html上的动态数据通过异步请求后端缓存服务器加载，不要直接查询数据库，通过Redis缓存提高查询速度。**

<whiteboard token="Sy6Bwm9AsheZeqbFtzQchQUpn2g"></whiteboard>

**4、使用负载均衡，通过部署多个Nginx服务器共同提供服务，不仅保证系统的可用性，还可以提高系统的访问性能。**

<whiteboard token="IabIwwGk6h5AxQbjkwVcHFK2nib"></whiteboard>

**5、在前端也做一部分缓存。**

不仅服务端可以做缓存，前端也可以做缓存，前端可以把缓存信息存储到

**LocalStorage：** 提供了持久化存储，可以存储大量数据

**SessionStorage：** 与 `LocalStorage` 类似，但数据只在当前会话中有效，当用户关闭标签页或浏览器时清空。

**Cookie**: 存储在用户计算机上的小型文本文件，可以在客户端和服务器之间传递数据

**浏览器缓存**：通过 HTTP 头部控制，比如：`Cache-Control`头部提供了更灵活的缓存控制选项，可以定义缓存的最大有效时间。



#### **移动应用门户技术方案**

移动应用门户是专为移动设备（如智能手机和平板电脑）设计的应用程序，比如：小程序、APP等，用户可以通过应用商店下载并安装。这些应用程序提供了更好的用户体验，通常具有更高的性能和交互性，可以直接从设备主屏幕启动。

对于移动应用提高访问效率方法通常有：

静态资源要走CDN服务器

对所有请求进行负载均衡

在前端及服务端缓存门户上显示的动态数据。



### 3）小结

根据上边的分析，对于Java程序员需要关注的是缓存服务的开发，主流的缓存服务器是Redis，所以我们接下来的工作重点是使用Redis为门户开发缓存服务接口。

实现一个门户用到的技术方案有哪些? 

## 2 缓存技术方案

### 2.1 需求分析

目标：明确本项目门户有哪些信息需要缓存

#### 1）界面原型

了解了门户的技术方案，下边通过门户界面原型分析本项目门户包括哪些部分。

本项目小程序门户首页如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HvtpbbhpOoSGxCxmbd8cFyM0nXb.png)

##### **第1部分**：城市选择

用户允许微信授权后，自动获取当前定位，点击地址进入城市选择页面，如下图：

已开通城市是指在区域管理中所有启用的区域信息。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/UGjPb4KYUoCo5ixFDyrcN1wsnLD.png)



##### **第2部分**：搜索

触发搜索框进入搜索主页面，如下图：

输入关键字搜索服务信息。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/LKE8bPjIaohsdQxvtMoczpZunpt.png)

##### **第3部分**：**首页服务列表**

默认展示前两个服务分类，每个服务分类下取前4个服务项（根据后台排序规则显示，如排序相同则按照更新时间倒序排列）

点击一级分类进入【全部服务】页；点击服务项进入【服务项目详情页】



##### **第4部分**：**热门服务列表**

这里显示在区域服务界面设置热门服务的服务项。

##### **第5部分**：**全部服务**

点击首页服务列表的服务分类或直接点击“全部服务”进入全部服务界面，

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/BNsvbVBSfobSM0x3D1ocwTJSnCg.png)

全部服务界面，如下图：

在全部服务界面需要展示当前区域下的服务分类，点击服务分类查询分类下的服务。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XJVVbDPiyob6U3xYZA2cfFZtnqn.png)

##### 第6部分 服务详情

点击服务名称进入服务详情页面：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/OqMxbnbbTooXFXx78AlcEbJlnBf.png)



#### 2）缓存需求

根据门户的技术方案，为了提供效率门户显示的信息从缓存查询，根据上边的界面原型分析有哪些信息需要缓存？

**1.定位界面上的已开通区域列表**

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Et3NbW0VPo7OGqx0zKRcIZ1Ineo.png)

**2.服务搜索**

通过Elasticsearch查询，稍后单独分析

**3.首页服务列表**

**4.热门服务列表**

**5.服务信息**

点击某一个服务打开服务详情页面，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/R5lhbEZE4on49dxTM9qchJvZnnh.png)

服务的详细信息一部分是服务介绍的图片和内容等，一部分是区域服务信息（比如价格）。

**6.服务分类**

在全部服务界面左侧显示服务分类

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/GuBNbUnKSoigN1xAJCsc89sungf.png)

服务分类下的服务项通过Elasticsearch去查询，稍后单独分析。



#### 3）小结

经过分析要明确有哪些信息需要缓存。

1、首页服务列表，包括两个服务分类及每个分类下的四个服务项。

2、热门服务列表

3、服务类型列表

4、开通城市列表

5、服务详细信息，内容包括服务项信息、服务信息。



### 2.2 Spring Cache入门

#### 2.2.1 入门程序

目标：学会使用SpringCache查询缓存注解并理解它的原理

##### 1）Redis访问工具

本项目使用Redis存储缓存数据，如何通过Java去访问Redis？

常用的有Jedis和Lettuce两个访问redis的客户端库，其中Lettuce的性能和并发性要好一些，Spring Boot 默认使用的是 Lettuce 作为 Redis 的客户端。

本项目集成了Spring data redis框架，在项目中可以通过RedisTemplate访问Redis，RedisTemplate提供了方便访问redis的模板方法。

RedisTemplate和Lettuce 是什么关系？

`RedisTemplate` 进行 Redis 操作时，实际上是通过 `Lettuce` 客户端与 Redis 服务器进行通信。

本项目也集成了Spring Cache，Spring Cache是spring的缓存框架，可以集成各种缓存中间件，比如：EhCache、Caffeine、redis。

Spring Cache最终也是通过Lettuce 去访问redis 。

使用Spring Cache的方法很简单，只需要在方法上添加注解即可实现将方法返回数据存入缓存，以及清理缓存等注解的使用。

RedisTemplate适用于灵活操作redis的场景，通过RedisTemplate的API灵活访问Redis。

这两种访问 redis的方法在本项目都有使用。



##### 2）Spring Cache基本介绍

Spring Cache是Spring提供的一个缓存框架，基于AOP原理，实现了基于注解的缓存功能，只需要简单地加一个注解就能实现缓存功能，对业务代码的侵入性很小。

基于SpringBoot使用Spring Cache非常简单，首先加入依赖：

```Java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
    <version>2.7.10</version>
</dependency>
```

本项目已在jzo2o-framework下的jzo2o-redis工程引入此依赖，其它服务只需要引入jzo2o-redis的依赖即可。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/P8ECbTF5YoMgOYxvmUVcqMJpnvc.png)

简单认识它的常用注解：

@EnableCaching：开启缓存注解功能

@Cacheable：查询数据时缓存，将方法的返回值进行缓存。  
@CacheEvict：用于删除缓存，将一条或多条数据从缓存中删除。  
@CachePut：用于更新缓存，将方法的返回值放到缓存中  
@Caching：组合多个缓存注解；  
@CacheConfig：统一配置@Cacheable中的value值

##### 3）切换分支

本节使用jzo2o-foundations工程的dev_02分支，课程会下发dev_02分支的代码

jzo2o-foundations_dev02.zip，大家在开发前先在本地创建了一个新分支dev_02并切换到dev_02，然后将下发的代码覆盖dev_02下的代码。

在jzo2o-foundations工程引入jzo2o-redis依赖

```JSON
<dependency>
    <groupId>com.jzo2o</groupId>
    <artifactId>jzo2o-redis</artifactId>
</dependency>
```

在jzo2o-foundations工程的bootstrap.yml中引入redis的配置文件，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/S0ijbneyvo9HMDx0gEXckqZ8ntc.png)

在nacos配置shared-redis-cluster.yaml，开发环境使用redis单机，配置文件如下：

注意配置redis的IP地址、端口和密码。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/T7HhbizYlop4okxOvCxcjsJ5nse.png)



##### 4）查询数据时缓存

下边使用Cacheable注解实现查询服务信息时对服务信息进行缓存，它的执行流程是：第一次查询服务信息缓存中没有该服务的信息此时去查询数据库，查询数据库拿到服务信息并进行缓存，第二次再去查询该服务信息发现缓存中有该服务的信息则直接查询缓存不再去数据库查询。

流程如下：

<whiteboard token="BaT9wo81bhnZFSbmw23ceRkZnne"></whiteboard>

首先在工程启动类中添加@EnableCaching注解，它表示开启Spring cache缓存组件。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/S21wbK6Uvo3L43xok8RctDSgnnh.png)

下边实现对区域服务信息查询时进行缓存。

首先找到区域服务信息的service，为了不和原来的getById(Serializable id)查询方法混淆，单独定义查询区域服务信息缓存的方法，如下：

在IServeService接口中定义如下接口：

```Java
/**
 * 查询区域服务信息并进行缓存
 * @param id 对应serve表的主键
 * @return 区域服务信息
 */
Serve queryServeByIdCache(Long id);
```

在接口实现类中定义如下方法：

```Java
@Override
public Serve queryServeByIdCache(Long id) {
    return getById(id);
}
```

此时该方法还是查询数据库。

下边在方法中添加Cacheable注解：

```Java
//    @Cacheable(value = "JZ_CACHE:SERVE_RECORD",key = "#id")
    @Cacheable(value = RedisConstants.CacheName.SERVE,key = "#id")
    public Serve queryServeByIdCache(Long id) {
        return getById(id);
    }
```

Cacheable注解配置的两项参数说明：

**value**：缓存的名称，缓存名称作为缓存key的前缀。

**key**: 缓存key，支持SpEL表达式，上述代码表示取参数id的值作为key

最终缓存key为：缓存名称+“::”+key，例如：上述代码id为123,最终的key为：JZ_CACHE:SERVE_RECORD::123

SpEL（Spring Expression Language）是一种在 Spring 框架中用于处理字符串表达式的强大工具，它可以实现获取对象的属性，调用对象的方法操作。

**keyGenerator**：指定一个自定义的键生成器（实现 org.springframework.cache.interceptor.KeyGenerator 接口的类），用于生成缓存的键。与 key 属性互斥，二者只能选其一。



##### 5）测试

对queryServeByIdCache方法进行测试，编写单元测试方法，如下：

```Java
package com.jzo2o.foundations.service;

import com.jzo2o.common.model.PageResult;
import com.jzo2o.foundations.model.domain.Serve;
import com.jzo2o.foundations.model.dto.request.ServePageQueryReqDTO;
import com.jzo2o.foundations.model.dto.response.*;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.util.Assert;

import javax.annotation.Resource;
import java.math.BigDecimal;
import java.util.List;

@SpringBootTest
@Slf4j
class IServeServiceTest {

//区域服务查询
@Test
public void test_queryServeByIdCache(){
    Serve serve = serveService.queryServeByIdCache(1693815623867506689L);
    Assert.notNull(serve,"服务为空");
}
...
```

运行测试方法，观察redis，数据缓存成功，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/VbFmbAmvhopFDix0KR9cpOIAnMg.png)

缓存key：JZ_CACHE:SERVE_RECORD::1693815623867506689

缓存value：serve表的记录

缓存过期时间：-1，表示永不过期

仔细核实内容是否符合我们的预期结果。



虽然数据被成功缓存，如果想调整缓存过期时间怎么做呢？

在@Cacheable注解中有一个属性为cacheManager，表示缓存管理器，通过缓存管理器可以设置缓存过期时间。

所有缓存相关的基础类都在jzo2o-redis工程，在jzo2o-redis工程定义spring cache需要的缓存管理器，如下：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/B1i5bn1LSoZwH2xYvLUcouhunag.png)

上图中共包括三个缓存管理器：

缓存时间为30分钟、一天、永久，分别对应的bean的名称为：cacheManager30Minutes、cacheManagerOneDay、cacheManagerForever。

下边我们在@Cacheable注解中指定缓存管理器为cacheManagerOneDay，即缓存时间为一天。

```Java
@Cacheable(value = RedisConstants.CacheName.SERVE,key="#id",cacheManager = RedisConstants.CacheManager.ONE_DAY)
public Serve queryServeByIdCache(Long id) {
    return getById(id);
}
```

重新运行单元测试方法，我们发现缓存的过期时间没有改变，这是为什么？

原因是根据前边的缓存流程：

先查询缓存，如果缓存存在则直接查询缓存返回数据，不再向缓存存储 数据。

<whiteboard token="NDUwwXY8VhjzhIbktXLcxYj1nWc"></whiteboard>

所以我们需要删除缓存，重新运行测试方法：

测试通过，观察redis中的缓存，过期时间已经改变，这说明我们设置的缓存管理器生效。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/QnV2blKXKowMjEx1Bg1cYDwXn3g.png)

由于缓存时间加了随机数，缓存一天的时间为90000秒左右。

关于缓存时间加随机数的原因稍后讲解。



##### 6）工作原理

Spring Cache是基于AOP原理，对添加注解@Cacheable的类生成代理对象，在方法执行前查看是否有缓存对应的数据，如果有直接返回数据，如果没有调用源方法获取数据返回，并缓存起来，下边跟踪Spring Cache的切面类CacheAspectSupport.java中的private Object execute(final CacheOperationInvoker invoker, Method method, CacheOperationContexts contexts)方法。

如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/RbgibyyS8oPZA7x4lgWcsrAFnTf.png)

下边打断点调试：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/E9hcbNY2XoVLhnxSYVecDrbPnog.png)

分别测试命中缓存和未命中缓存的情况。

第一次查询（redis中没有缓存）时执行方法查询数据库，第二次命中缓存直接查询redis不再执行方法。

##### 7）小结

Spring Cache 用在项目哪里了？怎么用的？



Spring Cache @Cacheable 注解的工作原理？





#### 2.2.2 测试Spring Cache

目标：学会使用@CacheEvict和@CachePut注解

在Spring Cache入门中使用了@Cacheable 注解，它实现的是查询时进行缓存。

下边测试另外两个常用 的注解，如下：

@CacheEvict：用于删除缓存，将一条或多条数据从缓存中删除。  
@CachePut：用于更新缓存，将方法的返回值放到缓存中

其它注解在项目开发中用到时再进行讲解，也可以自行查阅资料测试。

##### 1）测试@CachePut

CachePut注解实现的是将方法的返回值放到缓存中。

在服务上架后会将区域服务的信息写入缓存，服务下架会从缓存删除，下边我们实现服务上架将服务写入缓存。

找到服务上架的方法，在方法上添加@CachePut注解：

```Java
@Override
@Transactional
@CachePut(value = RedisConstants.CacheName.SERVE, key = "#id",  cacheManager = RedisConstants.CacheManager.ONE_DAY)
public Serve onSale(Long id){
....
```

上边代码同样指定了缓存名称、缓存key及缓存管理器（缓存过期时间为一天）。

重启foundation服务通过界面操作上架：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/BoWcb3KcBoDOpLxGTlpcX2iansf.png)

测试成功查看redis是否写入缓存：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/VrHYbUp16o1FsTxm96Hcl3x5nJf.png)



##### 2）测试@CacheEvict

下边测试服务下架删除缓存。

找到服务下架的方法，添加@CacheEvict注解

```Java
@Override
@Transactional
@CacheEvict(value = RedisConstants.CacheName.SERVE, key = "#id")
public Serve offSale(Long id){
....
```

这里是删除缓存所以不用再指定缓存管理器。

重启foundation服务通过界面操作下架：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/L36QbK1T3oAr0Dxpv5ccojXmnWd.png)



##### 3）小结

Spring Cache有哪些常用的注解，都有什么用？

@EnableCaching：开启缓存注解功能

@Cacheable：查询数据时缓存，将方法的返回值进行缓存。  
@CacheEvict：用于删除缓存，将一条或多条数据从缓存中删除。  
@CachePut：用于更新缓存，将方法的返回值放到缓存中  
@Caching：组合多个缓存注解；  



### 2.3 高并发项目缓存常见问题复习

#### 1）缓存穿透问题

什么是缓存穿透？如何解决缓存穿透？

什么是布隆过滤器？如何使用布隆过滤器？

本项目使用缓存空值或特殊值的方法去解决缓存穿透。



#### 2）缓存击穿问题

什么是缓存击穿？如何解决缓存击穿？

本项目对热点数据定时预热，使用定时任务刷新缓存保证缓存永不过期，解决缓存穿透问题。

#### 3) 缓存雪崩问题

什么是缓存雪崩？如何解决缓存雪崩？

本项目对key设置不同的过期时间解决缓存雪崩问题。



#### 4）缓存不一致问题

如何保证缓存一致性？



## 3 小程序首页缓存实现

### 3.1 开通区域列表缓存实现

目标：

能说出项目中的缓存方案。

实现开通区域列表缓存（完成查询缓存、删除缓存）。

#### 1）缓存方案分析

根据本项目门户的需求可知共有以下几块信息需要缓存，如下：

<sheet sheet-id="FE3CuB" token="ZqC7sSupJhiXN1t6KRjcp1JxndT"></sheet>

下边分析第一个开通区域列表的缓存方案：

查询缓存：查询已开通区域列表，如果没有缓存则查询数据库并将查询结果进行缓存，如果存在缓存则直接返回

启用区域：删除开通区域信息缓存（再次查询将缓存新的开通区域列表）。

禁用区域：删除开通区域信息缓存，删除该区域下的其它缓存信息，包括：首页服务列表，服务类型列表，热门服务列表。

定时任务：每天凌晨缓存已开通区域列表。



#### 2）查询缓存实现

下边我们先实现开通区域列表查询缓存。

首先把测试环境准备好：

启动jzo2o-gateway

启动jzo2o-publics

启动jzo2o-customer（如果前端已经通过小程序认证可以不用启动）

启动jzo2o-foundations

打开小程序开发工具

打开小程序，点击首页左上角的地址进入服务地址城市选择页面，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/SgsUbKaStoYHT6xwXuPcK9fXnNc.png)

在定位界面显示已开通城市列表，已开通城市是指在区域管理中所有启用的区域信息，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Ikseb3HUboP94axgaY8cQSEunxd.png)



跟踪Network找到开通区域列表的URL:/foundations/consumer/region/activeRegionList

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XOhXbvEZXoPjh3xZ0mrcY14Gn9e.png)

打开jzo2o-foundations工程，根据接口地址找到具体的代码:

找到接口：

```JSON
package com.jzo2o.foundations.controller.consumer;


import com.jzo2o.api.foundations.dto.response.RegionSimpleResDTO;
import com.jzo2o.foundations.service.IRegionService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import java.util.List;

/**
 * <p>
 * 区域表 前端控制器
 * </p>
 *
 * @author itcast
 * @since 2023-07-03
 */
@RestController("consumerRegionController")
@RequestMapping("/consumer/region")
@Api(tags = "用户端 - 区域相关接口")
public class RegionController {
    @Resource
    private IRegionService regionService;

    @GetMapping("/activeRegionList")
    @ApiOperation("已开通服务区域列表")
    public List<RegionSimpleResDTO> activeRegionList() {
        return regionService.queryActiveRegionListCache();
    }

}
```



找到service方法如下：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/NPwibBPAloDIFKxfavzcyuMLn7c.png)

在service方法上添加Spring cache注解：

```Java
@Override
@Cacheable(value = RedisConstants.CacheName.JZ_CACHE, key = "'ACTIVE_REGIONS'", cacheManager = RedisConstants.CacheManager.FOREVER)
public List<RegionSimpleResDTO> queryActiveRegionListCache() {
    return queryActiveRegionList();
}
```

说明： 

key: 当key用一个固定字符串时需要在双引号中用单引号括起来，如下所示：

```Java
key = "'ACTIVE_REGIONS'"
```

cacheManager ：RedisConstants.CacheManager.FOREVER设置了缓存永不过期。

重启jzo2o-foundations工程进行测试。

通过小程序访问定位界面，观察Network：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Vd0nbAY1coAv05x7m7qcgfFKngb.png)

/foundations/consumer/region/activeRegionList接口请求成功。

观察redis成功缓存开通区域列表：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/FUnqbk6AbodUaExmD5pc9n4Pnab.png)

#### 3）启用区域

启用一个新区域已经开通区域列表需要变更，该如何实现呢？

启用区域后删除已开通区域列表缓存，当去查询开通区域列表时重新缓存最新的开通区域列表。

可通过接口文档（http://localhost:11509/foundations/doc.html）找到启用区域的接口，如下：

![](https://feishu.cn/file/Atl7btK0rojuDAxyWQhcUJGhnEf)

接口的代码如下：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Ev5PbyUXZoPf9BxroMQc1gNGn4f.png)

找到service代码，修改如下：

```Java
/**
 * 区域管理
 *
 * @author itcast
 * @create 2023/7/17 16:50
 **/
@Service
public class RegionServiceImpl extends ServiceImpl<RegionMapper, Region> implements IRegionService {

/**
 * 区域启用
 *
 * @param id 区域id
 */
@Override
@CacheEvict(value = RedisConstants.CacheName.JZ_CACHE, key = "'ACTIVE_REGIONS'")
public void active(Long id) {
......
```

稍后测试。



#### 4）禁用区域

如果是禁用一个区域则需要删除开通区域列表缓存。

找到禁用区域的代码，修改如下：

```Java
   /**
 * 区域管理
 *
 * @author itcast
 * @create 2023/7/17 16:50
 **/
@Service
public class RegionServiceImpl extends ServiceImpl<RegionMapper, Region> implements IRegionService {

    @Override
    @Caching(evict = {
            @CacheEvict(value = RedisConstants.CacheName.JZ_CACHE, key = "'ACTIVE_REGIONS'")
//            todo:删除首页服务列表缓存
    })
    public void deactivate(Long id) {
    ......
```

禁用区域除了删除开通区域列表还需要删除首页服务列表、热门服务列表等，所以这里使用@Caching注解。

#### 5）测试

下边进行测试：

首先重启foundations服务。

启动运营管理（前端）进行测试。

测试禁用区域，找一个区域将区域下边服务全部下架，然后禁用该区域

最初有两个开通区域：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/K8vfbr04FoY0I0xuoPlcee0fnLh.png)

将郑州区域禁用，进入redis查询JZ_CACHE::ACTIVE_REGIONS缓存已经删除。

使用小程序进入定位界面查询开通区域列表，观察redis，缓存了最新的开通区域列表

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WMAGbguN7oB2y7xPAFDc3yZTnde.png)



测试启用区域，找一个区域添加服务并上架然后启用这个区域。

下边启用区域“郑州”，启动用区域后再次查询开通区域列表，redis缓存了最新的开通区域列表：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/UO4YbCYM6oebEtx4eJjcSwVfnEe.png)



#### 6）小结

项目哪里进行了缓存，缓存方案是什么？

项目中如何保证缓存的一致性？



### 3.2 定时任务更新缓存

#### 3.2.1 配置XXL-JOB环境

对于开通区域列表的缓存数据需要由定时任务每天凌晨更新缓存，我们使用xxl-job实现。



##### 1) **启动xxl-job**

执行docker start xxl-job-admin 启动xxl-job

访问：http://192.168.101.68:8088/xxl-job-admin/

账号和密码：admin/123456



##### 2) 执行器

**1.添加执行器依赖**

本项目在framework中定义了jzo2o-xxl-job工程，它对执行器bean执行了定义：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/CHQ0bc99oovFerxwpwecT7aSnme.png)

所以在需要使用xxl-job的微服务中需要引入下边的依赖，在jzo2o-foundations服务中引入下边的依赖：

```Java
<dependency>
    <groupId>com.jzo2o</groupId>
    <artifactId>jzo2o-xxl-job</artifactId>
</dependency>
```

**2.配置xxl-job**

接下来进入nacos，配置shared-xxl-job.yaml：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EeIbbT0DIosfX3xMoLfc5r0znx0.png)

说明：

address：调度中心的地址

appName：执行器名称，为spring.application.name表示微服务的名称（在bootstrap.yml中配置）

port：执行器端口号,通过xxl-job.port配置



在jzo2o-foundations.yaml中配置执行器的端口：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/FsA8bvSDwoIvRLxtcDCc33Ringd.png)



在jzo2o-foundations中加载shared-xxl-job.yaml：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EO1QbKDkqo9H45xVb5Ocksn6nDf.png)



**3. 下边进入调度中心添加执行器**

进入调度中心，进入执行器管理界面，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Kv9ubOOqFoZI9RxUu7DcLJX5nrc.png)

点击新增，填写执行器信息

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/FNNgbx4l3osJ7bxNU00c2hu4neh.png)

**AppName**：执行名称，在shared-xxl-job.yaml中指定执行器名称就是微服务的应用名。

**名称**：取一个中文名称。

**注册方式**：自动注册，只要执行器和调度中心连通执行器会自动注册到调度中心

**机器地址**：自动注册时不用填写。



找到应用名：jzo2o-foundations，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/CgVybM1zKonfKQxNxUsc7pednAd.png)



添加成功：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/LraRb3XAWockY5xpHGqcJpRcnMc.png)

启动jzo2o-foundations，查看jzo2o-foundations的控制台：

\>>>>>>>>>>> xxl-job remoting server start success, nettype = class com.xxl.job.core.server.EmbedServer, port = 11603 说明执行器启动成功。

稍等片刻进入 xxl-job调度中心，进入执行器管理界面，执行器注册成功：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/XJbbbqJ0MoJqzSxlLvKc0vYanpb.png)

点击“查看（1）”，查看执行器的地址，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Nun0bj0usoyb8IxmTfTc5nO9nUc.png)

如果无法调通，可手动修改执行器地址 http://192.168.101.1:11603/

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/TOj3bSUoCorO3nxvEtyc8RHJnRg.png)



#### 3.2.2 定义缓存更新任务

根据本节的目标，使用xxl-job定时更新开通区域列表的缓存。

##### 1）编写任务方法

下边编写更新开通区域列表缓存的任务方法：

先删除开通区域的缓存，再查询开通区域列表进行缓存。

```Java
package com.jzo2o.foundations.handler;

import com.jzo2o.foundations.constants.RedisConstants;
import com.jzo2o.foundations.service.IRegionService;
import com.jzo2o.foundations.service.IServeService;
import com.xxl.job.core.handler.annotation.XxlJob;
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;

/**
 * springCache缓存同步任务
 *
 * @author itcast
 * @create 2023/8/15 18:14
 **/
@Slf4j
@Component
public class SpringCacheSyncHandler {

    @Resource
    private IRegionService regionService;
    @Resource
    private RedisTemplate redisTemplate;

    /**
     * 已启用区域缓存更新
     * 每日凌晨1点执行
     */
    @XxlJob(value = "activeRegionCacheSync")
    public void activeRegionCacheSync() {
        log.info(">>>>>>>>开始进行缓存同步，更新已启用区域");
        //1.清理缓存
        String key = RedisConstants.CacheName.JZ_CACHE + "::ACTIVE_REGIONS";
        redisTemplate.delete(key);

        //2.刷新缓存
        regionService.queryActiveRegionListCache();
        log.info(">>>>>>>>更新已启用区域完成");
    }

}
```



##### 2）配置任务

下边在调度中心配置任务。

进入任务管理，新增任务：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/RXZBbpo2VoQV4vxG61lcDyQvnDe.png)

填写任务信息：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/GOyNbgE29oeq2vxH0l9cnFDWnih.png)



##### 3）启动任务并测试

任务配置完成，下边启动任务

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/QifvbA6DroJeUhxKK9Nckwu8njf.png)

启动成功：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/VyfEbEeCio1fUDxFtufco2zOnEw.png)

我们在任务方法上打断点跟踪，任务方法被执行，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DTfUbsnUPoRGqLxcpdMcfHOMn1c.png)

尝试启用或禁用一个区域，观察redis中开通区域列表缓存是否更新。

##### 4) 小结

项目中哪里用了xxl-job？怎么用的？



### 3.2 首页服务列表实现

我们先实现从数据库查询"首页服务列表"，将整体功能调试通过，再实现查询缓存。

下边我们先实现从数据库查询服务类型列表。

#### 3.2.1 首页服务列表实现

##### 1) 需求分析

**1.界面原型**

首页服务列表在门户的中心位置，下图红框中为首页服务列表区域：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WtlSbcd4eo3JjcxkEXLcqy0onVd.png)

默认展示前两个服务分类（按后台设置的排序字段进行升序排序），每个服务分类下取前4个服务项（按后台设置的排序字段进行升序排序）如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/WDCGbCJVKoDXalx15FgcLsznnkh.png)

一级服务分类显示的内容：服务分类的图标，服务分类的名称。

服务分类下的服务项内容：服务项图标，服务项名称。

##### 2）**接口定义**

定义首页服务列表接口，查询2个一级服务分类，每个服务分类下查询4个服务项。

接口名称：首页服务列表

接口路径：**GET**/foundations/customer/serve/firstPageServeList

![](https://feishu.cn/file/PsLpbHQzpoSUPjxIG5HcC3LQnHg)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/HVevbR7oToixPTxP0qOc1T0Qnid.png)

响应示例：

```Java
{
        "msg": "OK",
        "code": 200,
        "data": [
                {
                        "serveTypeId": 0,
                        "cityCode": "",
                        "serveTypeIcon": "",
                        "serveTypeSortNum": 0,
                        "serveResDTOList": [
                                {
                                        "serveItemSortNum": 0,
                                        "serveItemName": "",
                                        "serveItemId": 0,
                                        "serveItemIcon": "",
                                        "id": 0
                                }
                        ],
                        "serveTypeName": ""
                }
        ]
}
```

定义controller方法：

门户信息查询类接口统一在FirstPageServeController类中定义，service统一写在HomeService下。

定义FirstPageServeController 类提供门户界面查询类接口。

```Java
package com.jzo2o.foundations.controller.consumer;

...

/**
 * <p>
 * 前端控制器
 * </p>
 *
 * @author itcast
 * @since 2023-07-03
 */
@RestController("consumerServeController")
@RequestMapping("/customer/serve")
@Api(tags = "用户端 - 首页服务查询接口")
public class FirstPageServeController {

@GetMapping("/firstPageServeList")
@ApiOperation("首页服务列表")
@ApiImplicitParams({
        @ApiImplicitParam(name = "regionId", value = "区域id", required = true, dataTypeClass = Long.class)
})
public List<ServeCategoryResDTO> serveCategory(@RequestParam("regionId") Long regionId) {
    return null;
}
...
```

##### 3）mapper

数据来源于三张表：serve_type、serve_item、serve，区域id是非常重要的限制条件，因为要查询该区域显示在首页的服务列表。

如何查询数据？

可以先查询出服务类型，再根据服务类型id查询下边的服务项，伪代码如下：

```JSON
//查询服务类型
List<ServeType> serveTypeList =...
for(ServeType serveType: serveTypeList){
      Long serveTypeId = serveType.getId();
     //再根据服务类型id查询下属的服务项信息

}
```

上边的代码会导致1+n次查询数据库，这种代码要避免。

我们可以一次将符合条件的数据查询出来，再通过java程序对数据进行处理，再封装为接口要求的数据格式，最后返回给前端。

在ServeMapper.java中添加如下方法：

```Java
/**
 * 根据区域id查询服务图标
 *
 * @param regionId 区域id
 * @return 服务图标
 */
List<ServeCategoryResDTO> findServeIconCategoryByRegionId(@Param("regionId")Long regionId);
```

下边在ServeMapper.xml中添加如下内容：

```JSON
<select id="findServeIconCategoryByRegionId" resultMap="ServeCategoryMap">
    SELECT
        type.id as serve_type_id,
        type.name as serve_type_name,
        type.serve_type_icon,
        serve.city_code,
        serve.id as serve_id,
        item.id as serve_item_id,
        item.name as serve_item_name,
        item.serve_item_icon,
        item.sort_num as serve_item_sort_num
    FROM
        serve
            inner JOIN serve_item AS item ON item.id = serve.serve_item_id
            inner JOIN serve_type AS type ON type.id = item.serve_type_id
    WHERE
        serve.region_id = #{regionId}
      AND serve.sale_status = 2
    ORDER BY
        type.sort_num,
        item.sort_num
</select>

<!--手动的映射-->
<resultMap id="ServeCategoryMap" type="com.jzo2o.foundations.model.dto.response.ServeCategoryResDTO">
    <!--id映射主键字段-->
    <id column="serve_type_id" property="serveTypeId"></id>
    <!--result映射普通字段-->
    <result column="serve_type_name" property="serveTypeName"></result>
    <result column="serve_type_icon" property="serveTypeIcon"></result>
    <result column="city_code" property="cityCode"></result>

    <!--column 数据库中的字段名-->
    <!--property 实体类中对应的属性 该关键字可以省略... -->
    <!--ofType 是javaType中的单个对象类型-->
    <collection property="serveResDTOList" ofType="com.jzo2o.foundations.model.dto.response.ServeSimpleResDTO">
        <id column="serve_id" property="id"></id>
        <result column="serve_item_id" property="serveItemId"></result>
        <result column="serve_item_name" property="serveItemName"></result>
        <result column="serve_item_icon" property="serveItemIcon"></result>
        <result column="serve_item_sort_num" property="serveItemSortNum"></result>
    </collection>
</resultMap>
```

##### 4）service

定义专门用于门户首页查询的service接口,用于实现查询缓存：

```Java
package com.jzo2o.foundations.service;

...

import java.util.List;

/**
 * 首页查询相关功能
 *
 * @author itcast
 * @create 2023/8/21 10:55
 **/
public interface HomeService {
/**
 * 根据区域id获取服务图标信息
 *
 * @param regionId 区域id
 * @return 服务图标列表
 */
List<ServeCategoryResDTO> queryServeIconCategoryByRegionIdCache(Long regionId);
...
```

定义实现类



```Java
package com.jzo2o.foundations.service.impl;

...

/**
 * 首页查询相关功能
 *
 * @author itcast
 * @create 2023/8/21 10:57
 **/
@Slf4j
@Service
public class HomeServiceImpl implements HomeService {
/**
 * 根据区域id查询已开通的服务类型
 *
 * @param regionId 区域id
 * @return 已开通的服务类型
 */
@Override
public List<ServeCategoryResDTO> queryServeIconCategoryByRegionIdCache(Long regionId) {
    //1.校验当前城市是否为启用状态
    Region region = regionService.getById(regionId);
    if (ObjectUtil.isEmpty(region) || ObjectUtil.equal(FoundationStatusEnum.DISABLE.getStatus(), region.getActiveStatus())) {
        return Collections.emptyList();
    }

    //2.根据城市编码查询所有的服务图标
    List<ServeCategoryResDTO> list = serveMapper.findServeIconCategoryByRegionId(regionId);
    if (ObjectUtil.isEmpty(list)) {
        return Collections.emptyList();
    }

    //3.服务类型取前两个，每个类型下服务项取前4个
    //list的截止下标
    int endIndex = list.size() >= 2 ? 2 : list.size();
    List<ServeCategoryResDTO> serveCategoryResDTOS = new ArrayList<>(list.subList(0, endIndex));
    serveCategoryResDTOS.forEach(v -> {
        List<ServeSimpleResDTO> serveResDTOList = v.getServeResDTOList();
        //serveResDTOList的截止下标
        int endIndex2 = serveResDTOList.size() >= 4 ? 4 : serveResDTOList.size();
        List<ServeSimpleResDTO> serveSimpleResDTOS = new ArrayList<>(serveResDTOList.subList(0, endIndex2));
        v.setServeResDTOList(serveSimpleResDTOS);
    });

    return serveCategoryResDTOS;
}

```

##### 5) controller

在controller中调用service查询首页服务列表。

```Java
package com.jzo2o.foundations.controller.consumer;

...

/**
 * <p>
 * 前端控制器
 * </p>
 *
 * @author itcast
 * @since 2023-07-03
 */
@Validated
@RestController("consumerServeController")
@RequestMapping("/customer/serve")
@Api(tags = "用户端 - 首页服务查询接口")
public class FirstPageServeController {
    @Resource
    private HomeService homeService;

    @GetMapping("/firstPageServeList")
    @ApiOperation("首页服务列表")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "regionId", value = "区域id", required = true, dataTypeClass = Long.class)
    })
    public List<ServeCategoryResDTO> serveCategory(@RequestParam("regionId") Long regionId) {
        List<ServeCategoryResDTO> serveCategoryResDTOS = homeService.queryServeIconCategoryByRegionIdCache(regionId);
        return serveCategoryResDTOS;
    }
...
```

##### 6） 测试

重启jzo2o-foundations服务、网关服务，打开小程序。

观察小程序的访问记录

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EGtFbgKGXoPS6Xx1w6xc965JnLc.png)



注意：

如果访问记录中regionId为undefine，如下所示：

http://192.168.101.1:11500/foundations/customer/serve/firstPageServeList?regionId=undefine

这时需要重新定位，切换区域

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/U8OQbZ9G8oCFyTxCEOHcWyKInBg.png)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/EFdGbTp4Mo7VKxx86iucNdoanye.png)

测试预期结果：

在小程序首页正常显示服务列表。

示例：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/Rab3bWL6uoTW5hx0682cryLNnrf.png)



#### 3.2.2 首页服务列表缓存

##### 1） **缓存方案分析**

下边是门户的缓存设计：

<sheet sheet-id="5w6XkM" token="ZqC7sSupJhiXN1t6KRjcp1JxndT"></sheet>

下边分析首页服务列表的缓存方案：

查询缓存：查询首页服务列表，如果没有缓存则查询数据库并缓存，如果有缓存则直接返回

注意：缓存时需要考虑缓存穿透问题。

禁用区域：删除首页服务列表缓存

定时任务：每天凌晨缓存首页服务列表。

##### 2）查询缓存

下边在首页服务列表查询方法上添加Spring Cache注解实现查询缓存。

为了避免缓存穿透，如果服务列表为空则向redis缓存空值，缓存时间为30分钟；不为空则进行永久缓存。

在Cacheable注解中有两个属性可以指定条件进行缓存：

**condition**：指定一个 SpEL 表达式，用于决定是否要进行缓存。只有当条件表达式的结果为 true 时，方法的返回值才会被缓存。例如：

```Java
@Cacheable(value = "myCache", condition = "#id != null")
```

**unless**：与 condition 相反，只有当 SpEL 表达式的结果为 false 时，方法的返回值才会被缓存。

例如：

```Java
@Cacheable(value = "myCache", unless = "#result.length() > 100")
```

#result 表示方法的返回值，如果返回值结果集的长度大于100不进行缓存。

根据需求，我们需要根据方法的返回值去判断，如果结果集的长度大于0说明服务列表不空，此时缓存时间为永久缓存，否则缓存时间为30分钟。

condition不支持获取方法返回的值，不能识别#result。

我们使用unless实现。

**unless 的特点是符合条件的不缓存。设置技巧：确定要缓存的条件，取反即不缓存的条件。**

当方法返回的List的size为0时缓存30分钟，避免缓存穿透，设置为：#result.size() != 0 

当方法返回的List的size大于0永不过期，设置为：#result.size() == 0

代码如下：

```Java
package com.jzo2o.foundations.service.impl;

...

/**
 * 首页查询相关功能
 *
 * @author itcast
 * @create 2023/8/21 10:57
 **/
@Slf4j
@Service
public class HomeServiceImpl implements HomeService {
@Caching(
        cacheable = {
                //result为null时,属于缓存穿透情况，缓存时间30分钟
                @Cacheable(value = RedisConstants.CacheName.SERVE_ICON, key = "#regionId", unless = "#result.size() != 0", cacheManager = RedisConstants.CacheManager.THIRTY_MINUTES),
                //result不为null时,永久缓存
                @Cacheable(value = RedisConstants.CacheName.SERVE_ICON, key = "#regionId", unless = "#result.size() == 0", cacheManager = RedisConstants.CacheManager.FOREVER)
        }
)
   public List<ServeCategoryResDTO> queryServeIconCategoryByRegionIdCache(Long regionId) {
       //1.校验当前城市是否为启用状态
       Region region = regionService.getById(regionId);
       if (ObjectUtil.isEmpty(region) || ObjectUtil.equal(FoundationStatusEnum.DISABLE.getStatus(), region.getActiveStatus())) {
           return Collections.emptyList();
       }

       //2.根据城市编码查询所有的服务图标
       List<ServeCategoryResDTO> list = serveMapper.findServeIconCategoryByRegionId(regionId);
       if (ObjectUtil.isEmpty(list)) {
           return Collections.emptyList();
       }

       //3.服务类型取前两个，每个类型下服务项取前4个
       //list的截止下标
       int endIndex = list.size() >= 2 ? 2 : list.size();
       List<ServeCategoryResDTO> serveCategoryResDTOS = new ArrayList<>(list.subList(0, endIndex));
       serveCategoryResDTOS.forEach(v -> {
           List<ServeSimpleResDTO> serveResDTOList = v.getServeResDTOList();
           //serveResDTOList的截止下标
           int endIndex2 = serveResDTOList.size() >= 4 ? 4 : serveResDTOList.size();
           List<ServeSimpleResDTO> serveSimpleResDTOS = new ArrayList<>(serveResDTOList.subList(0, endIndex2));
           v.setServeResDTOList(serveSimpleResDTOS);
       });

       return serveCategoryResDTOS;
   }
...
```

##### 3）查询缓存测试

下边进行测试:

启动：jzo2o-foundations服务、网关服务，打开小程序，等待首页服务列表正常显示，进入redis查看首页服务列表是否缓存。

预期结果：首页服务列表正常缓存。

示例：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/IjHGbv4HZoHZSWxuzMmcFoS4nbd.png)



##### 4） 定时任务更新缓存

根据缓存方案的分析，对首页服务列表进行缓存。

编写定时任务代码：

```Java
/**
 * 已启用区域缓存更新
 * 每日凌晨1点执行
 */
@XxlJob("activeRegionCacheSync")
public void activeRegionCacheSync() throws Exception {
    log.info(">>>>>>>>开始进行缓存同步，更新已启用区域");

    //删除缓存
    Boolean delete = redisTemplate.delete(RedisConstants.CacheName.JZ_CACHE + "::ACTIVE_REGIONS");

    //通过查询开通区域列表进行缓存
    List<RegionSimpleResDTO> regionSimpleResDTOS = regionService.queryActiveRegionList();

    //遍历区域对该区域下的服务类型进行缓存
    regionSimpleResDTOS.forEach(item->{
        //区域id
        Long regionId = item.getId();

        //删除该区域下的首页服务列表
        String serve_type_key = RedisConstants.CacheName.SERVE_ICON + "::" + regionId;
        redisTemplate.delete(serve_type_key);
        homeService.queryServeIconCategoryByRegionIdCache(regionId);
        //todo 删除该区域下的服务类型列表缓存
    });
}
```

##### 5）定时任务更新缓存测试

下边测试定时任务更新缓存：

先将首页服务列表的缓存手动删除。

重启foundations服务，在上边代码中打断点，保证定时任务成功执行。

预期结果：对每个运营区域的首页服务列表进行缓存。



示例：

跟踪断点执行。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/L5sWbiLNWoTxXaxMrR9cR1x2nfd.png)





**4 编码规范**

门户信息查询类接口统一在FirstPageServeController类中定义，service统一写在HomeService下。

首先实现业务接口的功能，测试通过后再去实现缓存。

##### 6） 禁用区域时删除缓存

找到禁用区域代码 ,添加删除首页服务列表缓存的代码，如下： 

```Java

@Service
public class RegionServiceImpl extends ServiceImpl<RegionMapper, Region> implements IRegionService {
...
    @Override
    @Caching(evict = {
            @CacheEvict(value = RedisConstants.CacheName.JZ_CACHE, key = "'ACTIVE_REGIONS'"),
            @CacheEvict(value = RedisConstants.CacheName.SERVE_ICON, key = "#id")
    })
    public void deactivate(Long id) {
```

测试：禁用一个区域观察redis是否删除该区域的首页服务列表缓存。



##### 7）小结

项目中有做缓存吗？考虑缓存穿透问题了吗？怎么实现的？



### 3.3 服务类型列表缓存（实战）

#### 3.3.1 分析设计

**1.界面原型**

点击首页服务列表的服务分类或直接点击“全部服务”进入全部服务界面，

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/YXLFb59ugogHbPx5rnxcXSHsngc.png)

“全部服务” 界面如下图：

在全部服务界面需要展示当前区域下的服务分类，点击服务分类查询分类下的服务。

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/FJO5b76k8o2T2nxptT8czZfHnHc.png)

服务类型列表数据包括：服务类型的名称、服务类型id。

注意：只实现上图左侧显示的服务分类列表



1. **接口定义**

服务类型列表查询接口传入参数为：区域id，输出数据为服务类型列表，列表中元素包括：服务类型的名称、服务类型id，为了可扩展，列表元素包括服务类型的图标、排序字段方便进行展示。

接口名称：服务分类列表

接口路径：**GET**/foundations/customer/serve/serveTypeList

![](https://feishu.cn/file/NUxZb7hRBol55axfi1lckjlTnOd)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/C0Uqb46erojzc3xPqYDcrApNnnd.png)



1. **缓存方案分析**

下边分析服务类型列表的缓存方案：

查询缓存：查询服务类型列表，如果缓存没有则查询数据库并缓存，如果缓存有则直接返回

禁用区域：删除该区域下服务类型列表缓存

定时任务：每天凌晨缓存服务类型列表。

缓存时注意解决缓存穿透问题。



#### 3.3.2 设计提示

根据接口定义分析，需要根据区域id查询该区域下运营的服务项所属的分类，所以要分析数据来源于哪几张表？如果是一张表直接用MybatisPlus拼装SQL即可，如果是多张表则需要编写SQL语句。



在测试时需要切换区域进行测试。



### 3.4 热门服务列表（实战）

#### 3.4.1 分析设计

**1.界面原型**

在门户有一块区域叫精选推荐，此信息为热门服务列表，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/X3rUbYFHhoDH5ixwrQ1ceck3nmd.png)

热门服务列表是在区域服务管理界面进行设置，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/DPHRbLDJxojIx5xMksacLtwpnTh.png)

“设置热门”表示该服务项在本区域为精选推荐。

“取消热门”表示取消该服务项的精选推荐。

热门服务列表数据素包括：区域服务价格、价格单位、服务项id、服务项名称、服务项图标、服务详图。

信息来源于两张表：serve_item、serve

查询条件：区域id、是否热门（是）、是否上架（是）

按服务修改时间降序排序



**2、接口设计**

接口名称：首页热门服务列表

接口路径：**GET**/foundations/customer/serve/hotServeList

![](https://feishu.cn/file/FlHibvJPgoaLkpxhJzCcLWS0nce)

响应参数：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/CxX8baeDnoa0iwxZKTOcwSXUnhH.png)

示例数据：

```Java
{
        "msg": "OK",
        "code": 200,
        "data": {
                "cityCode": "",
                "serveItemName": "",
                "serveItemId": 0,
                "unit": 0,
                "detailImg": "",
                "price": 0,
                "serveItemImg": "",
                "id": 0
        }
}
```



**3. 缓存方案**

查询缓存：查询热门服务列表，如果缓存没有则查询数据库并缓存，如果缓存有则直接返回

注意：缓存时需要考虑缓存穿透问题。

禁用区域：删除热门服务列表缓存

定时任务：每天凌晨缓存热门服务列表。

#### 3.4.2 设计提示

根据接口定义分析，需要根据区域id查询该区域下运营的服务项所属的分类，所以要分析数据来源于哪几张表？如果是一张表直接用MybatisPlus拼装SQL即可，如果是多张表则需要编写SQL语句。

在测试时需要切换区域进行测试。



### 3.5 服务详情（实战）

#### 3.5.1 分析设计

**1.界面原型**

在首页服务列表区域、热门服务列表区域以及全部服务界面，点击服务项名称进入服务详情页面，如下图：

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/RfA1bHnq9ovVqmxsUPqct3ONnae.png)

服务详情页面显示服务相关的信息包括：服务项的名称、服务的运营价格、服务项的图片、服务项名称、价格单位。信息来源于serve_item表和serve表。

**2.接口设计**

接口名称：根据id查询服务

接口路径：**GET**/foundations/customer/serve/{id}

请求参数：服务id，即serve表的主键

![](https://feishu.cn/file/DKJgbtaggo1qVXxtULwcMVP7nCc)

![](../../../../../Attachments/development/architecture/solutions/yun-lan-dao-jia-xiang-mu/UtPJbtjvaoHAp5xOPFacZ96anub.png)



**3. 缓存方案**

服务详情信息来源于两部分信息：服务项信息、服务信息。

分别对服务项信息、服务信息进行缓存，方案如下：

<sheet></sheet>

分别编写方法：根据id查询服务项（已实现），根据id查询服务信息的方法（已实现）

然后实现查询缓存：

在服务详情接口中调用根据id查询服务项的service方法和 根据id查询服务信息的service方法，根据接口要求对两部分数据进行拼装返回。

针对热门服务的服务项信息和服务信息通过定时任务更新缓存，思路如下：

查询出所有热门服务，遍历热门服务列表先删除缓存，再执行查询方法对热门服务信息进行缓存。



#### 3.5.2 设计提示



根据分析可知接口要的数据来源于serve表和serve_item表，根据上边的缓存方案需要针对服务项信息和服务信息分别实现缓存方法。

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 云岚到家项目 / 第四章 小程序首页 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/G0DgwFSRqiaiajk0WxjcXLUSn0g)
