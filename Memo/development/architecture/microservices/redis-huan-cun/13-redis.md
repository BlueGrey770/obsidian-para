---
type: memo
date: 2026-11-06
aliases:
  - "13 redis"
---

<title>13 redis</title>

# redis的基本数据类型

| **数据类型** | **介绍** | **描述** |
|-|-|-|
| string | 字符串 | 普通字符串，redis中最简单的数据类型 |
| hash | 哈希 | 也叫散列，类似于java中的HashMap结构 |
| list | 列表 | 按照插入顺序排列，可以有重复元素，类似于java中的LinkedList |
| set | 集合 | 无序集合，没有重复元素，类似于java中的HashSet |
| zset / sorted-set | 有序集合 | 集合中每个元素关联一个分数（score），根据分数排序，没有重复元素 |

## String

<sheet sheet-id="fZwlwK" token="TpUdsY56HhyjMCtljJqch742nje"></sheet>

## hash

<sheet sheet-id="mNBx2b" token="TpUdsY56HhyjMCtljJqch742nje"></sheet>

## List

<sheet sheet-id="d1oGfL" token="TpUdsY56HhyjMCtljJqch742nje"></sheet>

## Set

<sheet sheet-id="y1i2kM" token="TpUdsY56HhyjMCtljJqch742nje"></sheet>

## ZSet

<sheet sheet-id="dFgFqD" token="TpUdsY56HhyjMCtljJqch742nje"></sheet>

## 通用命令

> 所谓的通用命令就是指部分数据类型的通用命令

<sheet sheet-id="zCuUcV" token="TpUdsY56HhyjMCtljJqch742nje"></sheet>

# 在Java程序中使用redis

> 现在我们使用springboot框架写项目所以使用spring data redis

## 引入依赖

<callout emoji="📌">
在项目的pom.xml文件中引入redis的起步依赖  
这个依赖由于是spring框架内置的所以不需要版本号，也就不需要在父项目中管理版本
</callout>

```Java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## 设置缓存

<callout emoji="🍰">
在出现热点数据的地方设置redis缓存来减轻数据库的压力  
所谓的热点数据就是查询频次高的数据
</callout>

```Java
//首先在热点数据的业务层注入redistemplate
@Autowried
private RedisTemplate redistemplate;

//然后在获取热点数据的功能的地方设置redis缓存
public OverViewVO getOverview() {
    //开启缓存
    OverViewVO overViewVO = (OverViewVO) redisTemplate.opsForValue().get("OVERVIEW");
    //对缓存内容进行判断
    if (overViewVO != null) {
        return overViewVO;
    }
    //如果没有缓存的话那么就从数据库进行查询
    OverViewVO cluevo = clueService.getOverview();
    OverViewVO businessvo = businessService.getOverview();
    overViewVO = new OverViewVO();
    BeanUtils.copyProperties(cluevo, overViewVO);
    BeanUtils.copyProperties(businessvo, overViewVO, "clueTotal", "clueWaitAllot", "clueWaitFollow", "clueFollowing", "clueFalse", "clueConvertBusiness");
    //将查询到的数据放到缓存中
    redisTemplate.opsForValue().set("OVERVIEW", overViewVO, 100, TimeUnit.SECONDS);
    return overViewVO;
}
```

---
> 来源：飞书技术知识库（脱敏版） · 04 微服务与中间件 / Redis 缓存 / 13 redis · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/Z9tKw4B6wiebYMkT9ZJcoaQ0n36)
