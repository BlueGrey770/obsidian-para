---
type: memo
date: 2026-11-06
aliases:
  - "三大框架知识点汇总"
---

<title>三大框架知识点汇总</title>

<whiteboard token="V88Vwo4UGhNzLabkK4ucC9XXnwg"></whiteboard>

三大框架涵盖了Spring MVC、Spring核心（IOC/DI）及MyBatis(Mybatis-Plus)三大核心模块的主要知识点。

---

# **Spring MVC** 

## **注解**

1. **`@RestController`作用**：组合注解，等同于 `@Controller`+ `@ResponseBody`。标记的类是一个控制器，且其所有方法返回的数据直接写入HTTP响应体(通常为JSON/XML)。
2. **请求映射注解**

```Java
@RequestMapping("/path")：通用请求映射，可指定路径和HTTP方法（如 method = RequestMethod.GET）。
@GetMapping("/path")：处理HTTP GET请求的快捷方式。
@PostMapping("/path")：处理HTTP POST请求的快捷方式。
@PutMapping("/path")：处理HTTP PUT请求的快捷方式。
@DeleteMapping("/path")：处理HTTP DELETE请求的快捷方式。
```

## **处理请求参数**

参数形式通常为：`key=value&key=value`。

1. **普通键值对参数**

**默认规则**：只需在控制器方法的形参中，定义与请求参数（key）**同名**的变量，即可自动接收。

**`@RequestParam`注解**：用于处理参数名不一致或有特殊要求的情况。

```Java
@RequestParam("前端key名")：当前后端参数名不一致时，用于绑定。
required = true/false ：指定参数是否必须传递（默认true）。
defaultValue = "默认值"：当参数未传递时，使用指定的默认值。
```

1. **处理时间参数`@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")`**：标注在实体类的日期字段或控制器方法的日期参数上，用于将前端传递的字符串自动转换为 `Date`类型。**在服务层补全时间**：

   <callout emoji="🍰">
   **新增数据**：通常需补全 `createTime`（创建时间）和 `updateTime`（更新时间）。
   **更新数据**：通常只需补全 `updateTime`。
   </callout>
2. **路径变量**

   ```Java
     @PathVariable：用于从请求URL的路径中获取变量值。
     用法： 
          @GetMapping("/user/{id}") 
          方法参数：  (@PathVariable("id") Integer userId)。
   ```
3. json格式参数

底层是readline



## **处理全局异常**

通过全局异常处理类，统一处理控制器中抛出的异常。

1. **定义全局异常处理类**创建一个类，例如 `GlobalExceptionHandler`。在类上添加 **`@RestControllerAdvice`** 注解，表明这是一个全局异常处理器，并且返回值会直接写入响应体。
2. **编写异常处理方法**在方法上添加 **`@ExceptionHandler(异常类型.class)`** 注解，指定该方法处理何种异常。**异常分类处理**：

   <callout emoji="🎼">
   **客户端异常**：如参数校验失败。可以正常提示具体原因给前端。
   **系统/业务异常**：如`Service`层抛出的异常。通常不将内部细节告知用户，可返回“服务繁忙，请稍后重试”等通用提示，并在后端记录日志以便排查。
   **未知/其他异常**：通过 `@ExceptionHandler(Exception.class)`捕获所有未被处理的异常，防止异常信息直接暴露给前端。
   </callout>

---

# **Spring (IOC & DI)** 

## **IOC (控制反转)**

**概念**：将对象的创建、管理、依赖装配的控制权，从程序员手中“反转”给Spring框架容器。

**实现**：Spring会自动扫描带有特定注解的类，通过反射创建其实例（称为 **Bean**），并将其维护在IOC容器中。使用时直接从容器获取对象即可。

## **相关注解**

1. **Bean定义注解**（标记哪些类需要被Spring管理）

   ```Java
    @Component：通用组件注解。
    @Service：通常用于标记服务层（Service）组件。
   ```
2. **依赖注入 (DI) 注解**（从容器中获取Bean并赋值）**`@Autowired`**：默认**按类型**进行自动注入。**解决Bean冲突**（同一类型有多个Bean时）：

   ```Java
   @Qualifier("具体的Bean名称")：与@Autowired搭配使用，按名称指定注入哪个Bean。
   @Resource(name = "具体的Bean名称")：JDK提供的注解，按名称注入。
   @Primary：在多个同类型Bean的其中一个上标注，将其设为优先注入的首选。
   ```

---

# **MyBatis** 

## **SQL编写方式**

1. **注解方式**在Mapper接口的方法上注解@Mapper直接使用 `@Select`， `@Insert`， `@Update`， `@Delete`等注解编写SQL。**优点**：简单直接。**缺点**：不适合编写复杂SQL，动态SQL能力弱。
2. **XML映射文件方式**（推荐，功能强大）**映射规则**：

   <callout emoji="🌟">
   在 `resources/`下创建与Mapper接口**同包名**的目录结构。
   创建 `Mapper类名.xml`文件。
   XML文件的 `namespace`属性必须等于Mapper接口的**全限定名**。
   SQL标签（如 `<select>`）的 `id`属性必须等于接口中对应方法的**方法名**。
   </callout>

## **动态SQL标签 (XML中)**

用于在XML中灵活、动态地拼接SQL语句。

1. **`<if test="条件">`作用**：判断条件，动态包含SQL片段。**条件写法**：

   <callout emoji="🌰">
   判断字符串：`test="name != null and name != ''"`
   判断其他类型：`test="id != null"`
   </callout>
2. **`<where>`作用**：

   <callout emoji="🦄">
   自动在包裹的内容前添加 `WHERE`关键字。
   自动去除紧随其后的第一个 `AND`或 `OR`。**解决了“所有条件都为空”和“第一个条件有AND”的问题**。
   </callout>
3. **`<set>`作用**：

   <callout emoji="💡">
   自动在包裹的内容前添加 `SET`关键字。
   自动去除末尾多余的逗号 `,` 。**专门用于优化UPDATE语句**。
   </callout>
4. **`<foreach>`作用**：遍历集合，常用于 `IN`条件（如批量删除）或批量插入。**属性**：

   ```Java
   collection：要遍历的集合参数名。
   item：遍历中每个元素的别名。
   open：遍历开始前添加的字符串（如( )。
   close：遍历结束后添加的字符串（如     )    ）。
   separator：每个元素之间的分隔符（如 , ）。
   ```

## **参数绑定**

在Mapper接口方法中，如何引用传入的参数。

1. **单个参数**：

   <callout emoji="🎹">
   **简单类型**：可直接用 `#{参数名}`引用。
   **对象类型**：用 `#{对象属性名}`引用。
   **Map/List/数组**：用 `#{key}`或 `#{list[0]}`、`#{array[0]}`引用。
   </callout>
2. **多个参数**：

   <callout emoji="📌">
   **问题**：MyBatis默认将其转换为 `arg0, arg1, ...`或 `param1, param2, ...`，可读性差。
   **解决方案**：使用 **`@Param("别名")`** 注解为每个参数指定一个明确的别名，在SQL中即可通过 `#{别名}`引用。
   解决方式：在方法有多个参数时，总是使用 `@Param`注解。
   </callout>

---
> 来源：飞书技术知识库（脱敏版） · 03 Web 开发与框架 / 项目实战：qk 全栈项目 / 三大框架知识点汇总 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/Dz64wphKXiG4TNkODTKc8WSVned)
