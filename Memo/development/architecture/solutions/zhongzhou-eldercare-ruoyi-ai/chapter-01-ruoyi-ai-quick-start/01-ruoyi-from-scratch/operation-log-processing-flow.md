---
type: memo
date: 2026-11-06
aliases:
  - "cao-zuo-ri-zhi-chu-li-liu-cheng"
  - "操作日志-处理流程"
---

# 操作日志-处理流程

## 关键的类

- com.ruoyi.common.annotation.Log  自定义注解用于标注需要记录操作日志的方法或参数。注解包含多个属性，可以设置日志的模块、功能、操作人类别、是否保存请求和响应参数、以及排除指定的请求参数等。注解可以在方法或参数上使用，并且在运行时可以被读取和解析，以便根据注解的信息记录相应操作日志。

  ```Java
  package com.ruoyi.common.annotation;
  
  import java.lang.annotation.Documented;
  import java.lang.annotation.ElementType;
  import java.lang.annotation.Retention;
  import java.lang.annotation.RetentionPolicy;
  import java.lang.annotation.Target;
  import com.ruoyi.common.enums.BusinessType;
  import com.ruoyi.common.enums.OperatorType;
  
  /**
   * 自定义操作日志记录注解
   * 
   * @author ruoyi
   *
   */
  @Target({ ElementType.PARAMETER, ElementType.METHOD })
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  public @interface Log
  {
      /**
       * 模块
       */
      public String title() default "";
  
      /**
       * 功能
       */
      public BusinessType businessType() default BusinessType.OTHER;
  
      /**
       * 操作人类别
       */
      public OperatorType operatorType() default OperatorType.MANAGE;
  
      /**
       * 是否保存请求的参数
       */
      public boolean isSaveRequestData() default true;
  
      /**
       * 是否保存响应的参数
       */
      public boolean isSaveResponseData() default true;
  
      /**
       * 排除指定的请求参数
       */
      public String[] excludeParamNames() default {};
  }
  ```

  ```Java
  /**
   * 修改通知公告
   */
  @PreAuthorize("@ss.hasPermi('system:notice:edit')")
  @Log(title = "通知公告", businessType = BusinessType.UPDATE)
  @PutMapping
  public AjaxResult edit(@Validated @RequestBody SysNotice notice)
  {
      notice.setUpdateBy(getUsername());
      return toAjax(noticeService.updateNotice(notice));
  }
  ```
- com.ruoyi.framework.aspectj.LogAspect  在一个aop切面类

     通过实现AOP切面编程，对目标方法进行拦截(标注Log注解的方法)，实现了操作日志的自动记录

- com.ruoyi.framework.manager.AsyncManager  异步任务管理器提供了执行延迟任务和关闭任务线程池的功能
- com.ruoyi.framework.config.ThreadPoolConfig  初始化线程池，交给spring容器管理
- com.ruoyi.framework.manager.factory.AsyncFactory   异步工厂，产生任务（记录日志到数据库）



## 整体实现思路

<whiteboard token="D0Xmwm34phBcNabAIvuctevtnOg"></whiteboard>

---
> 来源：飞书技术知识库（脱敏版） · 07 业务组件方案集 / 中州养老项目（若依+AI） / 第一章-若依+AI快速开发新项目 / 01-从0开始用若依 / 操作日志-处理流程 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/U6Pzw3c53ioUWtkGP6TcjmbNnof)
