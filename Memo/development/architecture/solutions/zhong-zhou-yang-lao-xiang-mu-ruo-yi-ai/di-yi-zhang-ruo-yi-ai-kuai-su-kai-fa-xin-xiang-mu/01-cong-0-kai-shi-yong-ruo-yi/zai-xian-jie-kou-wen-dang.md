---
type: memo
date: 2026-11-06
aliases:
  - "在线接口文档"
---

# 在线接口文档

目前我们测试都是基于前端项目完成的，如果后期开发中，前端还未开发完成，我们也需要对后端的接口进行测试，这个怎么做呢？

- 使用postman或者apifox来进行测试
- 使用在线接口工具进行测试（常用）

如果使用postman或者是apifox，那么必须要清楚地知道接口的路径，请求方式，出参和入参，并手动在apifox中添加请求路径、请求方法、请求参数，才能很好的进行测试。

现在如果我们与前端对接，需要提供详细的接口文档才行，不过在开发的过程中，接口文档可能不能及时的提供，或者是更新不及时，就会造成信息闭塞，造成不必要的效率降低。

所以，一般的前后端分离的项目，都会采用在线的接口工具进行调试，可以实时的展示接口的详细数据，也可以很方便的对接口进行测试。

### Swagger介绍

Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务

官网：https://swagger.io/。 

它的主要作用是：

- 使得前后端分离开发更加方便，有利于团队协作
- 接口的文档在线自动生成，降低后端开发人员编写接口文档的负担
- 功能测试 

   

Spring已经将Swagger纳入自身的标准，建立了Spring-swagger项目，现在叫Springfox。通过在项目中引入Springfox ，就可以非常简单快捷的使用Swagger啦。



### 项目中集成

目前若依框架已经集成了swagger，核心配置有如下几个：

1. 关键依赖

   ![这张图片展示的是若依框架集成Swagger的关键依赖配置内容，包含两条Maven依赖配置。第一条是Swagger3相关依赖，groupId为io.springfox，artifactId为springfox-boot-starter，用于引入Swagger的基础依赖；第二条是为解决Swagger页面类型转换错误，排除3.0.0版本的引用后，手动引入groupId为io.swagger、artifactId为swagger-models、版本为1.6.2的依赖，相关说明明确标注在该依赖配置的上方。这段依赖配置内容与上下文提及的“关键依赖”部分直接对应，是项目中集成Swagger的核心配置内容之一。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/QRznbmpq9oghgyxYQlHcNPwPnny.png)
2. 核心配置类

   - 位置：com.ruoyi.web.core.config.SwaggerConfig
   - 请设计prompt提示词，来阅读以下代码

   ```Java
   package com.ruoyi.web.core.config;
   
   import java.util.ArrayList;
   import java.util.List;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import com.ruoyi.common.config.RuoYiConfig;
   import io.swagger.annotations.ApiOperation;
   import io.swagger.models.auth.In;
   import springfox.documentation.builders.ApiInfoBuilder;
   import springfox.documentation.builders.PathSelectors;
   import springfox.documentation.builders.RequestHandlerSelectors;
   import springfox.documentation.service.ApiInfo;
   import springfox.documentation.service.ApiKey;
   import springfox.documentation.service.AuthorizationScope;
   import springfox.documentation.service.Contact;
   import springfox.documentation.service.SecurityReference;
   import springfox.documentation.service.SecurityScheme;
   import springfox.documentation.spi.DocumentationType;
   import springfox.documentation.spi.service.contexts.SecurityContext;
   import springfox.documentation.spring.web.plugins.Docket;
   
   /**
    * Swagger2的接口配置
    * 
    * @author ruoyi
    */
   @Configuration
   public class SwaggerConfig
   {
       /** 系统基础配置 */
       @Autowired
       private RuoYiConfig ruoyiConfig;
   
       /** 是否开启swagger */
       @Value("${swagger.enabled}")
       private boolean enabled;
   
       /** 设置请求的统一前缀 */
       @Value("${swagger.pathMapping}")
       private String pathMapping;
   
       /**
        * 创建API
        */
       @Bean
       public Docket createRestApi()
       {
           return new Docket(DocumentationType.OAS_30)
                   // 是否启用Swagger
                   .enable(enabled)
                   // 用来创建该API的基本信息，展示在文档的页面中（自定义展示的信息）
                   .apiInfo(apiInfo())
                   // 设置哪些接口暴露给Swagger展示
                   .select()
                   // 扫描所有有注解的api，用这种方式更灵活
                   .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                   // 扫描指定包中的swagger注解
                   // .apis(RequestHandlerSelectors.basePackage("com.ruoyi.project.tool.swagger"))
                   // 扫描所有 .apis(RequestHandlerSelectors.any())
                   .paths(PathSelectors.any())
                   .build()
                   /* 设置安全模式，swagger可以设置访问token */
                   .securitySchemes(securitySchemes())
                   .securityContexts(securityContexts())
                   .pathMapping(pathMapping);
       }
   
       /**
        * 安全模式，这里指定token通过Authorization头请求头传递
        */
       private List<SecurityScheme> securitySchemes()
       {
           List<SecurityScheme> apiKeyList = new ArrayList<SecurityScheme>();
           apiKeyList.add(new ApiKey("Authorization", "Authorization", In.HEADER.toValue()));
           return apiKeyList;
       }
   
       /**
        * 安全上下文
        */
       private List<SecurityContext> securityContexts()
       {
           List<SecurityContext> securityContexts = new ArrayList<>();
           securityContexts.add(
                   SecurityContext.builder()
                           .securityReferences(defaultAuth())
                           .operationSelector(o -> o.requestMappingPattern().matches("/.*"))
                           .build());
           return securityContexts;
       }
   
       /**
        * 默认的安全上引用
        */
       private List<SecurityReference> defaultAuth()
       {
           AuthorizationScope authorizationScope = new AuthorizationScope("global", "accessEverything");
           AuthorizationScope[] authorizationScopes = new AuthorizationScope[1];
           authorizationScopes[0] = authorizationScope;
           List<SecurityReference> securityReferences = new ArrayList<>();
           securityReferences.add(new SecurityReference("Authorization", authorizationScopes));
           return securityReferences;
       }
   
       /**
        * 添加摘要信息
        */
       private ApiInfo apiInfo()
       {
           // 用ApiInfoBuilder进行定制
           return new ApiInfoBuilder()
                   // 设置标题
                   .title("标题：若依管理系统_接口文档")
                   // 描述
                   .description("描述：用于管理集团旗下公司的人员信息,具体包括XXX,XXX模块...")
                   // 作者信息
                   .contact(new Contact(ruoyiConfig.getName(), null, null))
                   // 版本
                   .version("版本号:" + ruoyiConfig.getVersion())
                   .build();
       }
   }
   ```
3. 访问地址

   - 方式一：系统内部访问

   ![图片展示了若依管理系统集成Knife4j后的Swagger界面。左侧导航栏有“系统工具”“系统接口”等选项，其中“系统接口”被红色箭头指向。界面显示“若依管理系统 接口文档”，版本号3.8.8，Servers为http://localhost:8080 - Inferred Url。下方有test - controller等接口列表，以及Schemas选项。该图片与文档中介绍若依管理系统集成Knife4j后访问接口文档的方式相关，直观呈现了访问后的界面情况。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/ICJCbI6X0oBrs6xyCQ0cor0GnGV.png)

   - 方式二：服务地址+/swagger-ui/index.html  --- >  http://localhost:8080/swagger-ui/index.html

   ![这是在线接口文档的页面截图，对应若依框架已集成的Swagger相关内容，页面顶部显示访问地址为localhost:8080的Swagger入口，界面左上角标注“Swagger”。页面核心标题为“若依管理系统_接口文档”，明确版本号为V3.0.0，描述栏说明该文档用于管理集团旗下公司的人员信息，包含具体业务模块。页面还提供服务地址输入框、Authorize授权按钮，以及test-controller控制器和Schemas选项，符合Swagger作为接口文档工具的典型展示形式。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/XFGLbgCULoas0NxQsyAclFd4nEp.png)

### 集成Knife4j

knife4j是为Java MVC框架集成Swagger生成Api文档的增强解决方案,前身是swagger-bootstrap-ui,取名knife4j是希望它能像一把匕首一样小巧,轻量,并且功能强悍!

目前，一般都使用knife4j框架。

集成方式参考链接：

<bookmark name="# 插件集成" href="https://doc.ruoyi.vip/ruoyi-vue/document/cjjc.html#%E9%9B%86%E6%88%90knife4j%E5%AE%9E%E7%8E%B0swagger%E6%96%87%E6%A1%A3%E5%A2%9E%E5%BC%BA"></bookmark>

#### 导入依赖

在zzyl-admin模块的pom.xml

![图片展示了zzyl-admin模块pom.xml文件中导入依赖的代码。文件中，<dependency>标签下，<groupId>org.springframework.boot</groupId>、<artifactId>spring-boot-devtools</artifactId>等依赖被列出，其中<dependency>标签内还包含<groupId>com.github.xiaoymin</groupId>、<artifactId>knife4j-spring-boot-starter</artifactId>、<version>3.0.34</version>的依赖，用于knife4j相关功能。该图片与文档中导入依赖部分上下文对应，直观呈现了导入依赖的代码结构。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/ZbxUbwKxroiWY4xzj7jc4MW3nHS.png)



#### 修改前端页面为knife4j的页面

![图片展示了zzyl-vue3项目中views工具swagger目录下index.vue文件的部分代码。关键代码部分用红色框突出显示，其中第9行代码为“const url = ref(import.meta.env.VITE_APP_BASE_API + “/doc.html”);”。该代码用于设置接口文档的URL，通过引用环境变量中的APP_BASE_API值，拼接出完整的文档地址。此代码与文档中修改前端页面为knife4j页面的内容相关，是实现后端接口文档展示功能的代码基础。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/L3cHb9CY6oAUpexnqL9cnWWEnsc.png)

实现后的效果

![图片展示了若依管理系统中“系统工具”下的“系统接口”页面。左侧为系统导航栏，有首页、系统管理等选项，其中“系统接口”被红色箭头指向。右侧显示标题为“若依管理系统_接口文档”，有作者、版本、host等信息。页面还展示了文档简介、作者、版本、host等详细内容，如作者为RuoYi，版本号3.8.8，host为http://localhost:8060等。该图片与文档中介绍若依管理系统中系统接口相关内容相呼应，直观呈现了系统接口页面样式及部分信息。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/ICs2bF3Mpo98NQxY0RNccp5nnUf.png)

- 可以设置全局参数

由于若依中的所有接口都已经做了安全校验，在访问之前，我们必须设置一个全局参数来获取到登录之后的token才可以正常访问，不然会报一个401错误

![这张图片是接口访问失败的响应内容界面，显示响应码为401，耗时32ms，大小为95B，勾选了“显示说明”选项。响应内容为JSON格式，包含两条信息，一条是“请求访问:/serve/project/85，认证失败，无法访问系统资源”的提示信息，另一条是标识未通过权限验证的错误码“401”。该内容对文档中提及的若依接口无全局token访问会报401错误的描述进行了直观的示例呈现，能帮助开发者对应理解全局参数配置的必要性。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/OOmabZME3oCxL7xyOS8cybeSnrn.png)

解决：

当我们正常登录系统之后，就可以在后端获取一个jwt生成的token，这个token是登录成功的标识，后期其他请求都需要携带这个token才能正常访问

我们可以在浏览器中打开F12，查看网络，当登录成功后可以查看返回的token

![这张图片展示了浏览器开发者工具的网络请求响应界面，左侧标注了login、getInfo、getRouters三个接口选项，当前选中的是login接口。在响应内容区域，显示该请求的返回信息，包含msg为“操作成功”、code为200的状态字段，其中红色高亮标注的内容是登录成功后生成的jwt格式token，对应文档中提到的登录后需获取并配置token，才能通过接口安全校验，避免401错误的相关内容。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/NMxHbPa7ZoNKcLxdc9ocpilante.png)

有了token以后，就可以把这个token设置到全局参数中

![该图片为依管理系统接口文档的全局参数设置界面，界面左侧导航栏处于“全局参数设置”选中状态，右侧显示已添加的Authorization参数，其参数值为登录获取的JWT令牌，参数类型设为header，该参数是为解决若依接口安全校验需携带token的问题，将登录后获取的token设置到接口文档的全局参数中，以此保障接口访问的合法性，避免出现401错误。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/FpSebQskqo9dWfxwLY2c3X1ZnFc.png)

- 可以下载离线文档

![图片展示的是若依管理系统中集成的在线接口文档页面。左侧为导航栏，有主页、Authorize、Swagger Models、文档管理等选项，当前选中“文档管理”下的“离线文档”。右侧显示标题为“若依管理系统_接口文档”，可搜索文档关键字，下方有“下载Markdown”“下载Html”“下载Word”“OpenAPI”等下载按钮，还提示Knife4j提供导出4种格式的离线文档。该图与文档中介绍若依管理系统集成在线接口文档，可下载离线文档等内容相契合。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/WRGxbURPkoHYa9xZmtwc5G4hnvh.png)

- 提供了更好的接口文档阅读界面
- 提供了更方便的接口测试

### 项目中使用

如果现在想要测试护理项目，我们发现在swagger中并没有护理项目相关的接口，如果想要展示这些接口，我们需要添加上必要的注解来进行说明才能阅读和访问。

#### 常见注解

通过注解可以控制生成的接口文档，使接口文档拥有更好的可读性，常用注解如下：

| **注解** | **说明** |
|-|-|
| @Api | 用在类上，描述Controller的作用 |
| @ApiOperation | 用在方法上，说明方法的用途、作用 |
| @ApiParam | 用在方法的参数上，描述单个形参的含义，适用于简单场景 |
| @ApiImplicitParam | 用在方法上，描述单个形参的含义，适用于参数复杂或者详细描述参数的场景 |
| @ApiModel | 用在类上，用对象来接收参数或者返回参数，描述类的含义 |
| @ApiModelProperty | 用在类的属性上，用对象来接收参数或者返回参数，描述字段的含义 |

#### Controller的方法参数说明

我们熟悉了这些注解之后，我们可以让AI协助我们快速完成注解的编写：

<callout emoji="🤖">
**Prompt**
把整个NursingProjectController类给AI，然后在最后添加上一句话：
帮我给上述代码添加上swagger注解说明，要求每个参数也要进行说明，如：@Api、@ApiOperation、@ApiParam
</callout>

生成后的代码如下：

```Java
package com.ruoyi.serve.controller;

import java.util.List;
import javax.servlet.http.HttpServletResponse;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import com.ruoyi.common.annotation.Log;
import com.ruoyi.common.core.controller.BaseController;
import com.ruoyi.common.core.domain.AjaxResult;
import com.ruoyi.common.enums.BusinessType;
import com.ruoyi.serve.domain.NursingProject;
import com.ruoyi.serve.service.INursingProjectService;
import com.ruoyi.common.utils.poi.ExcelUtil;
import com.ruoyi.common.core.page.TableDataInfo;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;

/**
 * 护理项目Controller
 * 
 * @author ruoyi
 * @date 2024-06-09
 */
@Api(tags = "护理项目控制器")
@RestController
@RequestMapping("/serve/nursingProject")
public class NursingProjectController extends BaseController
{
    @Autowired
    private INursingProjectService nursingProjectService;

    /**
     * 查询护理项目列表
     */
    @ApiOperation("查询护理项目列表")
    @PreAuthorize("@ss.hasPermi('serve:nursingProject:list')")
    @GetMapping("/list")
    public TableDataInfo list(NursingProject nursingProject)
    {
        startPage();
        List<NursingProject> list = nursingProjectService.selectNursingProjectList(nursingProject);
        return getDataTable(list);
    }

    /**
     * 导出护理项目列表
     */
    @ApiOperation("导出护理项目列表")
    @PreAuthorize("@ss.hasPermi('serve:nursingProject:export')")
    @Log(title = "护理项目", businessType = BusinessType.EXPORT)
    @PostMapping("/export")
    public void export(HttpServletResponse response, NursingProject nursingProject)
    {
        List<NursingProject> list = nursingProjectService.selectNursingProjectList(nursingProject);
        ExcelUtil<NursingProject> util = new ExcelUtil<NursingProject>(NursingProject.class);
        util.exportExcel(response, list, "护理项目数据");
    }

    /**
     * 获取护理项目详细信息
     */
    @ApiOperation("获取护理项目详细信息")
    @PreAuthorize("@ss.hasPermi('serve:nursingProject:query')")
    @GetMapping(value = "/{id}")
    public AjaxResult getInfo(@ApiParam("护理项目ID") @PathVariable("id") Long id)
    {
        return success(nursingProjectService.selectNursingProjectById(id));
    }

    /**
     * 新增护理项目
     */
    @ApiOperation("新增护理项目")
    @PreAuthorize("@ss.hasPermi('serve:nursingProject:add')")
    @Log(title = "护理项目", businessType = BusinessType.INSERT)
    @PostMapping
    public AjaxResult add(@ApiParam("护理项目对象") @RequestBody NursingProject nursingProject)
    {
        return toAjax(nursingProjectService.insertNursingProject(nursingProject));
    }

    /**
     * 修改护理项目
     */
    @ApiOperation("修改护理项目")
    @PreAuthorize("@ss.hasPermi('serve:nursingProject:edit')")
    @Log(title = "护理项目", businessType = BusinessType.UPDATE)
    @PutMapping
    public AjaxResult edit(@ApiParam("护理项目对象") @RequestBody NursingProject nursingProject)
    {
        return toAjax(nursingProjectService.updateNursingProject(nursingProject));
    }

    /**
     * 删除护理项目
     */
    @ApiOperation("删除护理项目")
    @PreAuthorize("@ss.hasPermi('serve:nursingProject:remove')")
    @Log(title = "护理项目", businessType = BusinessType.DELETE)
        @DeleteMapping("/{ids}")
    public AjaxResult remove(@ApiParam("护理项目ID数组") @PathVariable Long[] ids)
    {
        return toAjax(nursingProjectService.deleteNursingProjectByIds(ids));
    }
}

```

直接替换我们自己的NursingProjectController类即可，然后进行测试，效果如下：

![图片展示的是若依管理系统接口文档页面。左侧为导航栏，有主页、Authorizer、Swagger Models、交互控制器、test - controller、护航项目管理等选项。右侧显示标题为“若依管理系统_接口文档”，并有多个搜索框。下方列出了POST、PUT、GET等HTTP方法对应的接口，如新增护航项目、修改护航项目、导出护航项目列表等，部分接口有“下载Markdown”“下载html”“下载Word”“OpenAPI”等操作按钮。该图与文档中介绍的接口文档内容相关，直观呈现了文档页面结构及部分接口信息。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/ZoItbYaR3o18hvx4AgGcMgg4nHg.png)



#### 请求体参数

一般的形参，如：id，status等这些，在controller的方法可以使用@ApiParam或者@ApiImplicitParam注解进行描述，如果接口是post、put请求，可能接收的是一个对象作为请求体参数，上面两个注解并不能很好的进行描述，这个时候需要使用新的注解：@ApiModel和@ApiModelProperty

比如，在新增护理项目的时候，接收的参数为NursingProject实体类，我们可以用这些注解在实体类中进行描述

如下代码：

```Java
package com.ruoyi.serve.domain;

import java.math.BigDecimal;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import org.apache.commons.lang3.builder.ToStringBuilder;
import org.apache.commons.lang3.builder.ToStringStyle;
import com.ruoyi.common.annotation.Excel;
import com.ruoyi.common.core.domain.BaseEntity;

/**
 * 护理项目对象 nursing_project
 * 
 * @author ruoyi
 * @date 2024-06-09
 */
@ApiModel("护理项目实体")
public class NursingProject extends BaseEntity
{
    private static final long serialVersionUID = 1L;

    /** 编号 */
    @ApiModelProperty(value = "主键ID")
    private Long id;

    /** 名称 */
    @Excel(name = "名称")
    @ApiModelProperty(value = "名称")
    private String name;

    /** 排序号 */
    @Excel(name = "排序号")
    @ApiModelProperty(value = "排序号")
    private Long orderNo;

    /** 单位 */
    @Excel(name = "单位")
    @ApiModelProperty(value = "单位")
    private String unit;

    /** 价格 */
    @Excel(name = "价格")
    @ApiModelProperty(value = "价格")
    private BigDecimal price;

    /** 图片 */
    @Excel(name = "图片")
    @ApiModelProperty(value = "图片")
    private String image;

    /** 护理要求 */
    @Excel(name = "护理要求")
    @ApiModelProperty(value = "护理要求")
    private String nursingRequirement;

    /** 状态（0：禁用，1：启用） */
    @Excel(name = "状态", readConverterExp = "0：禁用，1：启用")
    @ApiModelProperty(value = "0：禁用，1：启用")
    private Integer status;

    // set  get 方法省略
}
```

由于这个类继承了BaseEntity，我们也可以通过上面两个注解对BaseEntity进行描述

<callout emoji="📌">
注意：由于目前的swagger相关的依赖都在ruoyi-admin中，而BaseEntity类在ruoyi-common模块中，所以我们需要把swagger相关的所有依赖，从ruoyi-admin模块中拷贝到ruoyi-common模块中，删除ruoyi-admin中swagger相关的依赖，因为ruoyi-admin模块依赖了ruoyi-common
</callout>

![这张图片展示了项目结构与Maven依赖配置界面，左侧为项目目录树，用红色框标注了ruoyi-common模块下的pom.xml文件；右侧则是该pom.xml文件的依赖配置区域，红色框内是相关依赖的代码内容，具体包含springfox-boot-starter、swagger-models、xiaoymin的knife4j依赖，这与文档中提到的需将swagger相关依赖从ruoyi-admin模块拷贝到ruoyi-common模块的内容相对应，体现了模块间依赖调整的操作场景。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/BXhFbozxGoypbbxk5hcc6kRinNe.png)

添加注解之后的效果：

```Java
package com.ruoyi.common.core.domain;

import java.io.Serializable;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.annotation.JsonInclude;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

/**
 * Entity基类
 * 
 * @author ruoyi
 */
@ApiModel("Entity基类")
public class BaseEntity implements Serializable
{
    private static final long serialVersionUID = 1L;

    /** 搜索值 */
    @JsonIgnore
    private String searchValue;

    /** 创建者 */
    @ApiModelProperty(value = "创建者")
    private String createBy;

    /** 创建时间 */
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @ApiModelProperty(value = "创建时间")
    private Date createTime;

    /** 更新者 */
    @ApiModelProperty(value = "更新者")
    private String updateBy;

    /** 更新时间 */
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @ApiModelProperty(value = "更新时间")
    private Date updateTime;

    /** 备注 */
    @ApiModelProperty(value = "备注")
    private String remark;

    /** 请求参数 */
    @JsonInclude(JsonInclude.Include.NON_EMPTY)
    @ApiModelProperty(value = "请求参数")
    private Map<String, Object> params;

}
```

在线文档展示的效果：

- 新增或修改

![图片展示了若依管理系统接口文档中“新增护理项目”部分的在线文档展示效果。页面顶部有导航栏，中间是“获取护理项目详细信息”“新增护理项目”等选项卡，当前选中“新增护理项目”。下方是请求参数列表，包括nursingProject、createBy、createTime等参数，如nursingProject为护理项目对象，类型为body，是否必须为true，数据类型为NursingProject，schema也为NursingProject。该图片直观呈现了使用@ApiModel和@ApiModelProperty注解后，在线文档中参数的展示效果。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/HnmUbeuZlor77qxtfQNcKeQHnPf.png)

- 查询一个

![图片展示的是若依管理系统接口文档中“获取护理项目详细信息”接口的在线文档展示效果。上方标题为“若依管理系统_接口文档”，左侧有导航栏。中间部分显示接口名称、请求方式、URL、请求数据类型、响应数据类型等信息。下方“请求参数”区域以表格形式列出参数名称、参数说明、请求类型、是否必须、数据类型等内容，其中“id”参数被红色框突出显示，其类型为integer(int64)。该图片与上文介绍的使用@ApiModel和@ApiModelProperty注解对请求体参数进行描述的内容相关，直观呈现了添加注解后的在线文档展示效果。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/CaYRbTLC1oPwNjxFw8BcPfYCnvh.png)



#### 响应体参数

当我们去查询护理项目的时候，并不能展示详细的返回结果，如下图

![图片展示了若依管理系统接口文档中“获取护理项目详细信息”接口的响应体参数部分。上方有“获取护理项目详细信息”标题，下方列出响应状态，包括状态码200、401、403、404及对应说明。下方是响应参数部分，有参数名称、参数说明、类型、schema等列。图片与上下文关系紧密，直观呈现了文档中提到的不能展示详细返回结果的问题，为后续解决此问题提供参考。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/CDf4b8XvcoYLwWxd6hzch3HZnFc.png)

想要解决这个问题，有两步操作

- 使用@ApiModel和@ApiModelProperty注解对返回值的类型进行说明（已完成）
- 需要在controller类的方法上指定返回值类型

  - 返回一个对象
  - 返回数据列表（分页或不分页）

1. 返回一个对象

在目前的代码中，返回一个对象的接口代码，是根据id查询对应的对象，并返回

详细代码如下：

```Java
@ApiOperation("获取护理项目详细信息")
@PreAuthorize("@ss.hasPermi('serve:nursingProject:query')")
@GetMapping(value = "/{id}")
public AjaxResult getInfo(@ApiParam("护理项目ID") @PathVariable("id") Long id)
{
    return success(nursingProjectService.selectNursingProjectById(id));
}
```

这个代码并不能在swagger中去展示详细的返回结果说明，因为目前的返回值是一个AjaxResult，并没有指定泛型

我们需要使用若依框架中提供的另外一个返回值类型

![这张图片展示了若依框架（RuoYi）的核心代码结构与效果预览，左侧是项目的目录层级，红框标注了`RuoyiCommon`模块下的`AjaxResult`类；中间是`AjaxResult`类的部分代码，可见该类带有泛型`R<T>`，包含序列化标识、成功/失败状态常量，以及`code`、`msg`等属性与相关方法；右侧红框内是`AjaxResult`类的接口信息预览，清晰列出了成功返回、失败返回等各类方法，以及`data`、`msg`、`code`等返回值字段，和对应字段的类型说明，该内容正是上下文所指的、用于改造接口返回值以完善Swagger文档的`AjaxResult`类相关内容。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/PcLjbVfDaoB6AJxJbbtccXp3n6d.png)

> 在R这个类中也提供了详细的返回值的方法，比如有成功的，有失败的等等

修改NursingProjectController类中的方法返回值：

```Java
/**
 * 获取护理项目详细信息
 */
@PreAuthorize("@ss.hasPermi('serve:project:query')")
@GetMapping(value = "/{id}")
@ApiOperation(value = "获取护理项目详细信息", notes = "根据ID获取护理项目详细信息")
@ApiImplicitParam(name = "id", value = "护理项目ID", required = true, dataType = "Long", paramType = "path")
public R<NursingProject> getInfo(@PathVariable("id") Long id)
{
    NursingProject nursingProject = nursingProjectService.selectNursingProjectById(id);
    return R.ok(nursingProject);
}
```

最终的效果如下：

![该图片展示了若依管理系统的在线接口文档页面，呈现了GET请求“获取护理项目详细信息”接口的响应参数内容。页面中以红色框线标出了该接口响应的具体参数，涵盖参数名称、说明、类型及schema信息，包含编号、创建者、创建时间、护理要求等字段，还显示参数image对应类型为string。页面左侧是接口的功能菜单列表，包含新增、修改、删除等护理项目相关接口，底部标注了Apache License 2.0的版权信息。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/Vkw9bCfULoWwgrxXzrTcF4vrnkb.png)



1. 返回分页对象

目前的分页列表查询的返回值是TableDataInfo这个类，我们需要改造下这个类，让它具备接收泛型的能力

- 定义类中的泛型**T**
- 把类中的所有问号改为**T**

```Java
/**
 * 表格分页数据对象
 * 
 * @author ruoyi
 */
public class TableDataInfo<T> implements Serializable
{
    private static final long serialVersionUID = 1L;

    /** 总记录数 */
    private long total;

    /** 列表数据 */
    private List<T> rows;

    /** 消息状态码 */
    private int code;

    /** 消息内容 */
    private String msg;

    /**
     * 表格数据对象
     */
    public TableDataInfo()
    {
    }

    /**
     * 分页
     * 
     * @param list 列表数据
     * @param total 总记录数
     */
    public TableDataInfo(List<T> list, int total)
    {
        this.rows = list;
        this.total = total;
    }


    public List<T> getRows()
    {
        return rows;
    }

    public void setRows(List<T> rows)
    {
        this.rows = rows;
    }
    
    //其他代码略...
   
}
```

修改controller中分页列表查询的返回值

```Java
/**
 * 查询护理项目列表
 */
@PreAuthorize("@ss.hasPermi('serve:project:list')")
@GetMapping("/list")
@ApiOperation(value = "查询护理项目列表", notes = "根据护理项目条件查询护理项目列表")
public TableDataInfo<NursingProject> list(
    @ApiParam(value = "护理项目查询条件", required = false) NursingProject nursingProject)
{
    startPage();
    List<NursingProject> list = nursingProjectService.selectNursingProjectList(nursingProject);
    return getDataTable(list);
}
```

重启服务，查看分页接口的返回值效果：

![图片展示了若依管理系统接口文档中“获取护理项目列表”接口的响应体参数。参数包括code、msg、rows等，其中rows为NursingProject类型，包含createdBy、createTime、id、Image、name、nursingRequirement、orderNo等字段。图片与上下文紧密相关，上下文提到在控制器类方法上指定返回值类型，返回一个对象，此图呈现了修改后返回对象的详细参数情况，直观展示了返回结果的结构。](../../../../../../../Attachments/development/architecture/solutions/zhong-zhou-yang-lao-xiang-mu-ruo-yi-ai/di-yi-zhang-ruo-yi-ai-kuai-su-kai-fa-xin-xiang-mu/01-cong-0-kai-shi-yong-ruo-yi/VGHXb18SGoUuCTxPod5cl2RfnFh.png)

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第一章-若依+AI快速开发新项目 / 01-从0开始用若依 / 在线接口文档 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/UmrlwwCsiihTwVkxiJXc1JxSnkb)
