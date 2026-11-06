---
type: memo
date: 2026-11-06
aliases:
  - "12 mybatisplus_ThreadLocal"
---

<title>12 mybatisplus_ThreadLocal</title>

# mybatis-plus

```Java
作用：简化mybatis的操作，更擅长单表操作
特点：1. 制作增强不走修改，和Mybatis打配合，无侵入
     2. 强大的crud操作
```

## 使用步骤

1. 导入mybatisplus的起步依赖

```XML
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>${mybatisplus.version}</version>
</dependency>
```

1. 在配置文件yml中加入mp的配置

```YAML
mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

1. 在应用模块的service

```Java
public interface ActivityService extends IService<Activity>{}

@Service
public class ActivityServiceImpl extends ServiceImpl<ActivityMapper,Activity> implements ActivityService {
  
```

1. 在mapper层

```Java
@Mapper
public interface ActivityMapper extends BaseMapper<Activity> {}
```



<callout emoji="📍">
值得注意的是mybatis和mp并没有冲突在使用mp解决简单的单表操作的时候也可以使用mybatis的xml方式解决复杂的多表操作，也就是说在各个层中的接口中依然可以存在抽象类方法
</callout>

## 常见的注解

```Java
@TableName(""名字)：可以解决实体类和表明不一致问题
@TableId(type=设置主键生成的方式)：每张表都要设置太麻烦，于是采用全局配置的生成策略
  global-config:  （需要是在mp配置的下一级）
    db-config:
      id-type: auto   
      
@TableFiled(value="列名")：解决属性名和列名不一致问题
@TableField(exist=false)：解决扩展列问题/
```

## 条件构造器

1. 不支持lambda语法

```Java
new QueryWrapper();
new UpdateWrapper();
```

1. 支持lambda语法

```Java
lambdaQueryWrapper
基础条件:
    1. eq：equals    =
    2. ne:noEqueals    !=
    3. lt:less than    <=
    4. gt:great than    >
    5. le:less equals    <=
    6. ge:greatequals    >=
简化：
    between
    notin \ in
对null的判断：isNull
            isNotNull
like:
    1. like: %xxx%
    2. likeLeft: %xxx
    3. likeRight: xxx%
逻辑条件：
     or,and,not
LambdaUpdaWrapper
setSql:设置修改的列
配合lqw.eq（条件）使用指定对某行数据进行修改
```

1. 案例：使用mp解决活动的条件分页

```TypeScript
public PageData<Activity> findActivityByCollection(ActivityDTO activityDTO) {
    LambdaQueryWrapper<Activity> lambdaQuery = Wrappers.lambdaQuery();
    lambdaQuery
            .eq(activityDTO.getChannel() != null, Activity::getChannel, activityDTO.getChannel())
            .eq(activityDTO.getType() != null, Activity::getType, activityDTO.getType());

    // 因为在Activity实体类和表中并没有status字段，需要根据开始和结束时间来判断状态
        //单独写status的条件
    if (activityDTO.getStatus() != null) {
        if (activityDTO.getStatus() == 1) {
            lambdaQuery.gt(Activity::getStartTime, new Date());
        } else if (activityDTO.getStatus() == 2) {
            lambdaQuery.le(Activity::getStartTime, new Date())
                    .ge(Activity::getEndTime, new Date());
        } else if (activityDTO.getStatus() == 3) {
            lambdaQuery.lt(Activity::getEndTime, new Date());
        }
    }
    //mp的分页逻辑
    Page<Activity> activityPage = new Page<>(activityDTO.getPage(), activityDTO.getPageSize());
    Page page =  activityPage;

    page(page, lambdaQuery);
    PageData<Activity> pd = new PageData<>(Integer.parseInt(String.valueOf(page.getTotal())), page.getRecords());
    return pd;

}
```

## IService相关功能

1. 步骤

```Java
我们的service接口需要继承IService接口
我们实现类必须继承ServiceImpl<Mapper，实体类型> implements 接口
```

1. 功能

```Java
1. saveXXX：添加或者修改
2. removeXXX：删除有关的
3. updateXXX：跟更新有关的
4. getXXX：跟获取有关的
5.page(IPage page,条件构建器)
```

## 分页逻辑对比

```Java
mp的分页逻辑
@Override
    public PageData<Clue> findPool(ClueDTO clueDTO) {
        //1. 开启分页
        Page<Clue> page = new Page<>(clueDTO.getPage(), clueDTO.getPageSize());
        // 2. 执行查询：直接传入page+DTO，MP自动分页、自动查询总条数
        clueMapper.findPool(page, clueDTO);
        // 3. 封装你的PageData：MP的page对象直接提供 总条数+分页数据，无任何强转！
        PageData<Clue> pageData = new PageData<>(
               (int) page.getTotal(), // 总条数（long→int，安全转换）
                page.getRecords()      // 分页后的数据列表（MP自动封装）
        );
        return pageData;
    }
    
mybatis的PageHelper分页逻辑
    //使用pagehelper进行数据分页
    PageHelper.startPage(deptDTO.getPage(), deptDTO.getPageSize());
    //Dao层进行数据查询
    List<Dept> depts = deptMapper.finddeptByCollections(deptDTO);
    
    Page page = (Page) depts;
    PageData<Dept> dp = new PageData<>(Integer.parseInt(String.valueOf(page.getTotal())), page.getResult());
    //将从mapper层得到的数据返回给controller层
    return dp;
```

**注意：如果是需要使用mybaits的xml方式查询但是需要使用分页，需要使用mp的分页逻辑这个时候如果再使用PageHelper的分页逻辑的话会出现包冲突**

# 原始MyBatis解决主表与从表冲突问题的方式

使用情况:

当遇到一对多，一对一的时候，我们不能采用resultType，这个只是封装单个类，而resultmap适合封装嵌套类

一对多示例代码:

```XML
<!--<ResultMapid="需要和sql的resultmap的属性的值保持一致" type="设置封装的实体类的全类名" autoMapping="true">  -->
<resultMap id="clue_clue_track_record_rm" type="cn.itcast.entities.Clue" autoMapping="true">
     <!--  主表和从表的列冲突的问题就会导致失败 -->
    <id property="id" column="id"></id> <!-- 封装主键 -->
    <!-- 如果冲突列不处理以靠前的为主！！！ -->
     <!-- 遇到集合属性 -->
    <collection property="trackRecords" ofType="cn.itcast.entities.ClueTrackRecord" autoMapping="true">
        <!-- 处理冲突列 才能正确封装 -->
        <!-- 封装主键 -->
        <id column="trackId" property="id"></id>
         <!-- 封装普通类 -->
        <result column="trackLevel" property="level"></result>
        <result column="trackUserId" property="userId"></result>
        <result column="trackCreateTime" property="createTime"></result>
        <result column="trackSubject" property="subject"></result>
    </collection>
</resultMap>
<select id="findById" resultMap="clue_clue_track_record_rm">
    select *,
            <!-- 起别名和我们集合中的对象属性映射即可 -->
           qk_managerment.clue_track_record.id as trackId,
           qk_managerment.clue_track_record.level as trackLevel,
           qk_managerment.clue_track_record.user_id as trackUserId,
           qk_managerment.clue_track_record.create_time as trackCreateTime,
           qk_managerment.clue_track_record.subject as trackSubject
    from clue
        left join clue_track_record on clue.id = clue_track_record.clue_id
    where clue.id = #{id}
    group by update_time
</select>
```

一对一示例代码:

```XML

```

# ThreadLocal

## 本地线程ThreadLocal

<callout emoji="🚅">
**ThreadLocal 的核心作用是：为【每个线程】提供一份「独立的、专属的变量副本」，让多线程并发访问时，线程之间的数据完全隔离、互不干扰，彻底解决多线程环境下的** ✅**线程安全问题**✅
</callout>

## 使用步骤

1. **在commen模块中定义一个类**

```TypeScript
@Component
public class CommonData {
    private ThreadLocal<Map<String, Object>> commons  = new ThreadLocal<>();
    public void set(Map<String, Object> map) {
        commons.set(map);
    }
    public Map<String, Object> get() {
        return commons.get();
    }
    public void remove() {
        commons.remove();
    }
}
```

<callout emoji="🚅">
在`common公共模块`定义这个`CommonData`类，本质是**封装一个「线程级全局的通用数据容器」**，核心目的是：✅**在单个请求线程的全调用链路中，统一存储 / 读取任意业务通用数据，实现线程内跨层、跨模块的数据共享与传递，同时保证多线程并发安全**。
</callout>

1. **在拦截器中使用**

```TypeScript
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    log.info("拦截器生效");
    //获取token
    String token = request.getHeader("token");
    //检验token
    if (token == null) {
        response.setStatus(401);
        return false;
    }
    //解析token 看是否有效
    try {
        //获取token中的数据
        Map<String, Object> map = tokenUtils.parseToken(token);
        //保存数据
        commonData.set(map);
        //解析并检验token
        tokenUtils.parseToken(token);
    } catch (Exception e) {
        response.setStatus(401);
        return false;
    }

    return HandlerInterceptor.super.preHandle(request, response, handler);
}

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        commonData.remove();
// HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
    }
```

<callout emoji="🍞">
在**登录拦截器**中使用 ThreadLocal（CommonData），核心目的是：
> **在请求登录进入时，统一解析 Token 中的用户信息，存入 ThreadLocal 容器；让当前请求线程的「Controller→Service→Mapper→任意工具类」全链路，可直接获取用户数据，全程无需参数传递；请求结束后，统一清理数据，保证线程安全与内存安全**。
✔️ **只有前端发起 HTTP 请求，进入你的项目时**，拦截器的`preHandle`方法才会执行 → 解析请求头`token` → 将用户数据存入`CommonData`（ThreadLocal）；
✔️ 本次请求线程的后续所有业务代码，才能从`commonData.get()`拿到数据，比如下面的业务层中需要使用到userId的业务；
✔️ 非请求触发的逻辑（比如定时任务、手动测试的 main 方法），不会走拦截器，ThreadLocal 中没有数据，直接取值会拿到`null`。
</callout>

1. **在业务中使用**

```TypeScript
@Override
public boolean followUp(Clue clue) {
    //一旦修改了线索, 那就是跟进中!!!
    clue.setStatus(3);
    //修改clue的数据
    saveOrUpdate(clue);
    //获取当前用户
    Map<String, Object> map = commonData.get();
    ClueTrackRecord build = ClueTrackRecord.builder()
            .clueId(clue.getId())
            .userId((Integer)map.get("id")) // 用线程获取当前用户id
            .subject(clue.getSubject())
            .level(clue.getLevel())
            .record(clue.getRecord())
            .createTime(LocalDateTime.now())
            .nextTime(clue.getNextTime()).build();
    int insert = clueTrackRecordMapper.insert(build);
    return insert>0;
}
```

<callout emoji="🎁">
**项目中所有需要获取「当前操作的登录用户 ID / 用户信息」的业务方法，都必须通过`CommonData`（ThreadLocal）获取，严禁其他方式！**
</callout>

---
> 来源：飞书技术知识库（脱敏版） · 02 数据库与持久层 / MyBatis-Plus 与 ThreadLocal / 12 mybatisplus_ThreadLocal · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/Al28wzH4giAH4yknwcZcdHXEnvh)
