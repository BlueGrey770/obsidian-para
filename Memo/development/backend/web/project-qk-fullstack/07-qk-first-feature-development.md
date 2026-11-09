---
type: memo
date: 2026-11-06
aliases:
  - "07-qk-xiang-mu-di-yi-ge-gong-neng-kai-fa"
  - "07 qk项目第一个功能开发"
---
# 07 qk项目第一个功能开发

## 创建项目

```Java
第一步：创建一个springboot项目作为父项目--->qk-parent
第二步：创建一个java模块作为子模块--------->qk-comment 
        用来定义项目中通用的功能和类
第三步：创建一个java模块作为子模块--------->qk-entity   
        用来定义项目中需要用到的实体类，dto（封装前端传入的数据）类，vo（视图渲染）类
第四步：创建一个Springboot项目作为子模块---->qk-management
        整个的项目业务的三层架构都要在这里面写
```

### 模块之间的交流与沟通

<callout emoji="🌰">
在开发中在业务层肯定会用到实体类和封装的数据类，这时候我们就要跨模块进行调用。
我们就需要使用到gav坐标在pom.xml文件中引入其他的模块坐标通过这种方式打破模块之间的界限来使用类
</callout>

如果要在业务模块中使用实体类和数据封装的通用类的话，就需要导入他们的模块坐标来打破界限使用类

```Java
<dependency>
    <groupId>com.itheima</groupId>
    <artifactId>qk-comment</artifactId>
</dependency>
<dependency>
    <groupId>com.itheima</groupId>
    <artifactId>qk-entity</artifactId>
</dependency>
```

**注意：这里用到了maven的依赖继承所以看不到版本号，版本号在父项目的家规中**

## 开发规范_三层架构

```Java
1. 第一层Controller（控制层）层
    1.接收从前端传过来的参数
    2.将参数封装
    3.将封装好的参数传给service层，调用service方法
    4.将service层返回的数据返回给前端页面
2. 第二层Service（业务层）层
    1.接收从controller传过来的数据
    2.处理数据
    3.将处理好的数据传给dao层，调用dao层的方法
    4.将dao层返回的数据返回给controller层
3. 第三层Mapper（持久层）层
    1.接收从service层传过来的数据
    2.链接数据库进行持久化更新和存储
    3.将结果返回给service层
```

## 确定数据传输格式

<callout emoji="🚅">
**开发的需要根据接口文档进行开发，所以下面的所有动作都是基于接口文档进行开发的**
**开发之前一定要仔细阅读接口文档！！！！**
</callout>

### springMVC对请求参数的封装

#### 键值对参数的封装（Query String）

##### 请求形式

?key=value格式的参数

```SQL
?key=value格式的参数
第一版: 不使用springMVC完成, 使用原始的servlet
   String request.getRequestParam("键"); 通过键获取提交的值
第二版: 使用springMVC完成
    定义参数, 要求参数名和提交的键保持一致即可
```

##### 基本原则

只需要在控制器方法中定义**参数名与前端 key 保持一致**，SpringMVC 就可以自动完成参数接收

底层本质是调用 `HttpServletRequest` 的 `getParameter()` 方法

```SQL
@RequestMapping
public Result findRoleByIdAndCondition(RoleDTO roleDTO){
        PageData<Role> pd = roleSerive.findRoleByIdAndCondition(roleDTO);
        return pd == null ? Result.error("查询失败") : Result.ok(pd);
}
```

##### 注意事项

```Python
(1) 当参数名和前端提交的键不一致的时候怎么处理???
    @RequestParam(name = "前端的键")
(2) 当参数没有的话, 可以设置默认值吗???
    @RequestParam(defaultValue="默认值")
(3) 怎么要求要求前端必须填写某个参数, 不填不行!!!
    @RequestParam
(4) 当参数是时间格式
    @DateTimeFormat 里面有pattern属性告知时间格式
```

#### 路径参数的封装（Path Variable）

##### 请求形式

```SQL
/参数1/参数2/...
```

##### 基本用法

使用 `@PathVariable` 注解接收路径参数，并在路径中通过 `{}` 指定变量名：

```TypeScript
@GetMapping("/user/{id}")
public String findRoleById(@PathVariable Integer id) {
    return "success";
}
```

##### 注意事项

时间类型路径参数

若路径参数中包含时间类型，同样需要指定格式：

```SQL
@DateTimeFormat(pattern = "yyyy-MM-dd")
```

####  JSON 格式参数的封装

##### **请求形式**

<callout emoji="📍">
请求体中携带 JSON 数据（常见于 POST / PUT 请求）
</callout>

##### 基本用法

使用 `@RequestBody` 注解，将 JSON 数据直接封装为 Java 对象：

`@PostMapping("/save")
public String save(@RequestBody User user) {return "success";
``}`

---

##### 时间参数的处理

使用 `@JsonFormat`时,`@RequestBody` 不仅可以封装 JSON 数据，还可结合 `@JsonFormat` 处理时间格式问题：

```SQL
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
```

##### 注意事项

若 JSON 对象中包含时间字段，推荐使用 `@JsonFormat`

`@DateTimeFormat` 主要用于 **请求参数和路径参数**

### 确定返回值格式

> ![zmctb85vkollynxnkhvcxhkcnvc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/zmctb85vkollynxnkhvcxhkcnvc.png)
> 图注：该图片展示了对应项目接口返回数据的JSON格式样例的部分内容，是明确返回数据JSON格式阶段的关键内容，为封装返回数据给前端的规范提供了直观示例。样例中包含三个核心字段，分别为标识处理状态的“code”字段，取值为数字1、表示操作结果描述的“msg”字段，取值为字符串“success”，以及用于承载具体业务数据的“data”字段。

```Java
/**
 * 定义一个类用来封装返回结果
 * josn数据格式
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Result {
    private Integer code;
    private String msg;
    private Object data;

    //查询成功之后需要返回的结果，不需要返回表中的数据
    public static Result ok(Object data) {
        return new Result(200, "操作成功", data);
    }

    //查询成功之后需要返回的结果，需要返回表中的数据
    public static Result ok(String msg, Object data) {
        return new Result(200, msg, data);
    }

    //查询失败之后返回的结果，不需要返回数据
    public static Result error(String msg) {
        return new Result(500, msg, null);
    }
}
```

<callout emoji="👍">
由于整个项目都要使用这个类来封装数据那么我们将这个类放到comment中，用于封装返回值格式
</callout>

```SQL
DispatcherServlet: 前端控制器, 负责将我们的前端uri请求路径进行解析, 交给处理器映射器HandlerMapping进行映射, 是处理映射映射内部帮我恩找到路径, 请求方式对应的Controller中的功能并执行
执行完后交给视图解析器ViewResolver将结果相应给前端, 但是第一版的时候, 将这个结果当成路径去跳转!!!所以报错了Result不是路径
但是随着springMVC升级的, 现在视图解析已经没有用, 因为都是异步请求, 后台不能直接访问前端页面资源, 需要将结果序列化成json给到前端
我们可以采用 @ResponseBody 让springMVC将结果不要当成路径, 而且当成响应体, 那么内部就会使用jackson的json转换工具将对象序列化json字符串然后在相应给前端
    而@RestController 包含了@Controller和@ResponseBody, 默认返回值就是json
```

### 确定前端传入的数据格式

<sheet sheet-id="poET8h" token="IkmSsI3chhDg74tU2wVc4cJ8n5f"></sheet>

示例代码

```TypeScript
package cn.itcast.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
// 数据传输对象
public class RoleDTO {
    //角色名称
    private String name;
    // 角色标识
    private String label;
    // 当前页码
    private Integer page;
    // 每页显示条数
    private Integer pageSize;
}
```

<callout emoji="❤️">
由于这个DTO类是每个表和对象以及业务都有不同的字段和需求，所以这个类放在entity中由于封装前端传入的数据
</callout>

### 确定返回数据的json格式

响应数据样例：

```JSON
{
    "code": 1,
    "msg": "success",
    "data": {
        "total": 2,
        "rows": [
            {
                "id": 3,
                "name": "普通用户",
                "label": "user_normal",
                "remark": "普通用户",
                "createTime": "2025-03-23 16:42:35",
                "updateTime": "2025-04-18 19:48:00"
            },
            {
                "id": 19,
                "name": "商机专员",
                "label": "business_operator",
                "remark": "用于跟进商机信息",
                "createTime": "2025-04-18 19:46:43",
                "updateTime": "2025-04-18 19:46:43"
            }
        ]
    }
}
```

```TypeScript
@Data
@AllArgsConstructor
@NoArgsConstructor
public class PageData<T> {
    private Integer total;
    private List<T> rows;
}
```

<callout emoji="🍞">
由于整个项目都要使用这个类来封装数据那么我们将这个类放到comment中，用于封装整个返回数据给前端。但是需要在service层封装好再返回
</callout>

```Java
public PageData<Dept> finddeptByCollection(DeptDTO deptDTO) { //获取controller层的数据
    //处理数据
    //设置初始索引
    if (deptDTO.getPage() == null) deptDTO.setPage(1);
    //设置每页显示的数量
    if (deptDTO.getPageSize() == null) deptDTO.setPageSize(10);
    //设置开始索引
    deptDTO.setStart((deptDTO.getPage()-1)*deptDTO.getPageSize());
    //将处理好的数据交给mapper层
    List<Dept> rows = deptMapper.finddeptByCollection(deptDTO);

    int total = deptMapper.finddeptByCollections(deptDTO);
    PageData<Dept> pageData = new PageData(total,rows);
    //将从mapper层得到的数据返回给controller层
    return pageData;
}
```

### 测试结果

![y7gjbbxcao1xflxgenqcgfcvn5c](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/y7gjbbxcao1xflxgenqcgfcvn5c.png)
> 图注：图片展示的是接口测试结果的响应数据样例。在Body部分，以JSON格式呈现，包含code、msg、data等字段。其中data下有total和rows两个子字段，total为1，rows中包含一个对象，id为1，name为“管理员”，label为“admin”，remark为“管理员，用于管理整个系统数据”，createTime和updateTime均为null。该图片与文档中“确定返回数据的json格式”内容相关，用于说明响应数据的结构和示例。

<callout emoji="🎉">
结果可以看到和接口开发文档中的需求一样至此开发一个功能的代码就做完了，然后后面如果再用到这种comment中的类来进行数据封装的话直接拿过来用就可以了不用再费劲的写重复性代码
</callout>



### 踩坑：常见写法错误汇总

### 常见写法错误

#### 文档声明必须在第一行顶行顶头写！！！

2.xmL必须有根元素！！！

<！--xmL约束，限制xmL书写规则-->

![nd6pbl857oss3lxd3gqc8iugn5c](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/nd6pbl857oss3lxd3gqc8iugn5c.png)
> 图注：图片展示的是MyBatis配置文件中关于UserMapper的XML代码片段。上方红框内是XML文档类型声明，包含DTD配置信息；下方红框内是delete语句，用于删除user表中id在数组中的记录，使用foreach循环遍历数组。该图片与文档中“SQL插入参数顺序与数据库中顺序不一致”部分内容相关，用于说明在MyBatis中SQL语句的书写规范，强调SQL语句需与数据库表顺序保持一致，否则会报错。

#### SQL插入参数顺序与数据库中顺序不一致

在如下代码块中 要插入的参数顺序必须与数据库表顺序保持一致,否者将会报错

```SQL
// 添加课程
@Insert("insert into course values(null,#{subject},#{name},#{price},#{target},#{description},#{createTime},#{updateTime})")
int addCourse(Course course);
```

必须与图中顺序一致

![wqnpbpylqovl3vx1qubcyzhcnwd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/wqnpbpylqovl3vx1qubcyzhcnwd.png)
> 图注：图片展示了数据库中“course”表的结构，包含8个字段。其中，“id”字段为int类型且自动递增；“subject”为tinyint类型；“name”为varchar类型；“price”为int类型；“target”为tinyint类型；“description”为varchar类型；“create_time”和“update_time”均为datetime类型。图片与上文“SQL插入参数顺序与数据库中顺序不一致”内容相关，强调在插入参数时需与数据库表字段顺序一致，否则会报错。

#### Service层未校验补全数据

在Service架构中添加修改数据,记得检查数据是否已经补全

如:时间 需要在获取前端参数封装后,在Service层中检查修改补全 保证封装数据符合数据库的要求

```TypeScript
//添加活动
@Override
public boolean addActivity(Activity activity) {
    activity.setCreateTime(LocalDateTime.now());
    activity.setUpdateTime(LocalDateTime.now());
    return activityMapper.addActivity(activity) > 0;
}
```

#### 把时间字段当字符串判空了（最常见）

```XML
<if test="startTime != null and startTime != ''">
  start_time = #{startTime},
</if>
```

如果 `startTime` 在 Java 里是 `LocalDateTime`，那 `startTime != ''` 就是在拿 LocalDateTime 和 String 比较 => 报错。

✅ 修法：**时间类型只判 null**

正确写法:

```Bash
<if test="startTime != null">
    start_time = #{startTime},
</if>
<if test="endTime != null">
    end_time = #{endTime},
</if>
```

错误写法:

```Bash
<if test="startTime != null and startTime != ''">
    start_time = #{startTime},
</if>
<if test="endTime != null and endTime != ''">
    end_time = #{endTime},
</if>
```

报错特征码:

```Markdown
Caused by: org.apache.ibatis.exceptions.PersistenceException: 
### Error updating database.  Cause: java.lang.IllegalArgumentException: invalid comparison: java.time.LocalDateTime and java.lang.String
### The error may exist in cn/itcast/mapper/ActivityMapper.xml
### The error may involve cn.itcast.mapper.ActivityMapper.updateActivity
### The error occurred while executing an update
### Cause: java.lang.IllegalArgumentException: invalid comparison: java.time.LocalDateTime and java.lang.String
```

#### 用字符串去跟时间比较

```XML
<if test="startTime &gt; '2025-01-01 00:00:00'">
...
</if>
```

OGNL 还是会做类型比较，LocalDateTime vs String => 报错。

✅ 修法：把比较值也变成 LocalDateTime（在 Java 层先算好传进来），或改成 SQL 里比较而不是 OGNL 比较。

#### DTO/VO 里时间字段类型不一致

比如前端传的是字符串，你在 VO 里用 `String startTime`，但实体/Mapper 参数里又是 `LocalDateTime`，MyBatis 在某些动态判断里就会出现混用。

✅ 建议：

- 要么统一用 `LocalDateTime`（推荐）
- 要么统一用 `String`，并在 Service 层转换成 `LocalDateTime` 再入库
- Spring 接收 `LocalDateTime` 时，配好格式：

```Java
@DateTimeFormat(pattern="yyyy-MM-dd HH:mm:ss")
private LocalDateTime startTime;
```

- 或者用 Jackson：

```Java
@JsonFormat(pattern="yyyy-MM-dd HH:mm:ss", timezone="GMT+8")
private LocalDateTime startTime;
```

#### 因类型错误,输出异常

使用int类型 为变量定义类型 默认值是0 会被sql判定语句识别执行为0

要使用Integer定义变量类型 ,默认值可以为null,不会被sql语句误识别

正确示例:

```Java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserDTO {
    private String name;
    private Integer status;
    private String phone;
    private Integer deptId;
    private Integer page = 1;
    private Integer pageSize = 10;
}
```

错误示范:

```Java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserDTO {
    private String name;
    private int status;
    private String phone;
    private int deptId;
    private int page = 1;
    private int pageSize = 10;
}
```

错误输出:

![mjehbdbnjo7dllxrpdhcdaxqnoc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/mjehbdbnjo7dllxrpdhcdaxqnoc.png)
> 图注：这张是项目开发中SQL语句执行相关的日志截图，核心展示了异常的SQL查询情况：该SQL是对user、dept、role三张表进行关联的查询操作，参数部分标注有两个被红框突出的`0(Integer)`，对应了文档中提到的用int类型定义变量默认值为0的错误写法，这种写法会被SQL语句识别为参数值，最终查询得到1条结果但Row的数量为0，体现了int类型默认值0被SQL误识别后导致的逻辑异常问题，也呼应了文档中int类型易引发类型错误、输出异常的说明。

#### 漏写@PathVariable

路径参数写完要加`@PathVariable`才能生效,否则会在功能使用中报错

正确代码:

```Java
//8. 根据部门查询用户
@GetMapping("/dept/{deptId}")
public Result findByDept(@PathVariable int deptId){
    return Result.ok(userService.findByDept(deptId));
}
```

错误代码:

```Java
//8. 根据部门查询用户
@GetMapping("/dept/{deptId}")
public Result findByDept(int deptId){
    return Result.ok(userService.findByDept(deptId));
}
```

#### endpoin 空指针异常

漏写少写 引起的空指针异常

![fvxpbqppvocjowxsnopc6phcnoc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/fvxpbqppvocjowxsnopc6phcnoc.png)
> 图注：这张图片展示的是一段Java代码，代码内容为定义一个配置属性类。该类添加了`@Component`和`@Data`注解，类名为`AliProperties`，内部声明了endpoint、bucketName、region、hobby四个私有属性。其中核心标注内容有两处，一是`@ConfigurationProperties`注解，二是该注解内用红色箭头和红框突出显示的`prefix = 'ali.oss'`，这部分用于指定配置属性的前缀，明确该类对应加载ali.oss开头的配置项。

#### 将对象放入容器中 也要从容器中取用对象

```Java
@RestController
@RequestMapping("/upload")
public class UploadController {
    // 注入阿里OSSUtils工具类对象
    @Autowired
    private OSSUtils ossUtils;
    @PostMapping
    public Result upload(MultipartFile image) throws IOException, ClientException {
        String url = ossUtils.upload(image.getInputStream(), image.getOriginalFilename());
        return url == null ? Result.error("上传失败") : Result.ok(url);
    }
}
```

错误示例:

```Java
@RestController
@RequestMapping("/upload")
public class UploadController {
    // 注入阿里OSSUtils工具类对象
    //@Autowired
    //private OSSUtils ossUtils;
    @PostMapping
    public Result upload(MultipartFile image) throws IOException, ClientException {
        String url = new ossUtils.upload(image.getInputStream(), image.getOriginalFilename());
        return url == null ? Result.error("上传失败") : Result.ok(url);
    }
}
```

> 自己创建的对象没有框架管理赋值,内部值为null

---
> 来源：飞书技术知识库（脱敏版） · 03 Web 开发与框架 / 项目实战：qk 全栈项目 / 07 qk项目第一个功能开发 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/K1CSwiLdbip0olk1FTUcFwaVnsb)
