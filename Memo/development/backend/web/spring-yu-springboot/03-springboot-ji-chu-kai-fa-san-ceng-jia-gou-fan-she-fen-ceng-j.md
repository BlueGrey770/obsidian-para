---
type: memo
date: 2026-11-06
aliases:
  - "03 SpringBoot基础开发_三层架构_反射分层解耦"
---

<title>03 SpringBoot基础开发_三层架构_反射分层解耦</title>

# 一、SpringBoot基础

## 初识Spring

后端开发中的 SpringBoot 基础、Web 入门程序、Web 案例、分层解耦等内容，旨在帮助开发者掌握 SpringBoot 开发及相关架构设计

### **资源类型与架构模式**

介绍了静态资源（如 HTML、CSS、JS 等）和动态资源（如 Servlet、JSP 等，企业多基于 Spring 框架构建）

<callout emoji="🏕️">
BS 架构（Browser/Server，维护方便但体验一般）
 CS 架构（Client/Server，体验不错但开发维护麻烦）。
</callout>

### **Spring 与 SpringBoot**：

Spring 形成了开发生态圈，由众多子项目组成“spring 全家桶”，核心是 SpringFramework，

但直接开发存在配置繁琐和入门难度大的问题。SpringBoot 可简化配置、快速开发，是企业开发主流。

# Web案例

## Web入门程序

#### SpringBoot开发步骤

第1步：创建SpringBoot工程，并勾选Web开发相关依赖

第2步：定义HelloController类，添加方法hello，并添加注解

创建Spring Boot 模块

![这张图片展示的是创建Spring Boot模块的配置界面，对应SpringBoot开发步骤里创建模块的操作环节。界面中明确标注了核心内容：需填写模块名“day04-spring-test”，选择构建工具为Maven，Group项填写“cn.itcast”，Package name项也对应填入“cn.itcast”，且选择JAR作为打包方式，最后可点击右下角的“Next”按钮继续后续操作，同时界面还提供了Server URL、项目存储位置、语言、Java版本等配置选项。](../../../../../Attachments/development/backend/web/spring-yu-springboot/C5xubgyXMoVTmexv50OcOikcn2c.png)

选择Spring Boot版本 以及 勾选web开发相关依赖

![这张图片展示了创建Spring Boot模块的操作界面，核心呈现了SpringBoot开发的关键步骤：选择稳定版本的Spring Boot（选中了3.5.8版本），勾选Web模块下的Spring Web依赖，同时勾选了Lombok依赖，最后点击右下角的“Create”按钮以完成模块创建。这一界面与文档中提及的SpringBoot开发步骤对应，是第1步创建SpringBoot工程时选择版本和相关依赖的操作界面，用于搭建包含Web开发所需依赖的Spring Boot项目，满足开发RESTful Web应用的需求。](../../../../../Attachments/development/backend/web/spring-yu-springboot/Me7qbfeg2oABrqx04s9cesKDnxb.png)

<callout emoji="🌟">
SpringBoot官方提供的脚手架，里面只能够选择SpringBoot的几个最新的版本，如果要选择其他相对低一点的版本，可以在springboot项目创建完毕之后，修改项目的pom.xml文件中的版本号。
</callout>

可以删除 不需要的内容

![这张图片展示了Java Web项目的工程目录结构，在名为day04-spring-test的项目文件夹内，src目录上方被红色方框标出了.gitattributes、.gitignore、HELP.md、mvnw、mvnw.cmd这几个文件，方框内配有红色文字标注“可以删除 暂时无用”，说明这些文件对当前的SpringBoot入门程序开发没有实际作用，可以选择删除。该图对应了文档中提到的“可以删除不需要的内容”这一步骤，为开发者清理项目冗余文件提供了直观的参考。](../../../../../Attachments/development/backend/web/spring-yu-springboot/U1I7bZTKaoTd7Lx9Jwhc3Qipneb.png)

**2). 定义HelloController类，添加方法hello，并添加注解**

在`com.itheima`这个包下新建一个类：`HelloController`

![这张图片展示了一个SpringBoot项目的目录结构，项目名称为springboot-web-quickstart。在src/main/java/com.itheima目录下，红色方框标注出了刚刚创建的HelloController类，同一目录下还存在SpringbootWebQuickstartApplication类，项目其他目录包括resources、test，以及pom.xml文件和web-project01.ml配置文件。这张图片直观呈现了SpringBoot开发步骤中定义HelloController类的具体文件位置结构，对应了文档中在com.itheima包下新建HelloController类的操作要求。](../../../../../Attachments/development/backend/web/spring-yu-springboot/OYrXbCa84olHT0xukwOcOZgunbd.png)

HelloController中的内容，具体如下：

```Java
package com.itheima;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController //标识当前类是一个请求处理类
public class HelloController {

    @RequestMapping("/hello") //标识请求路径
    public String hello(String name){
        System.out.println("HelloController ... hello: " + name);
        return "Hello " + name;
    }

}
```



**3). 运行测试**

运行SpringBoot自动生成的引导类 (标识有`@SpringBootApplication`注解的类)

![这张图片展示了IntelliJ IDEA开发工具的界面，右侧是名为Day04SpringTestApplication的Spring Boot引导类代码，该类带有@SpringBootApplication注解，包含main方法。代码编辑器左侧的红色箭头指向类名左侧的三角形运行标识，点击该标识后弹出操作菜单，菜单中突出显示了“Run 'Day04SpringTestApplication'...”选项，这对应文档中运行SpringBoot引导类的步骤，即带有@SpringBootApplication注解的类的方式，用于测试Web入门程序。](../../../../../Attachments/development/backend/web/spring-yu-springboot/CEHNbV0rAoVfYPx2u8hcNKD8nGb.png)

创建一个应用接口

![图片展示了在IntelliJ IDEA中创建的Spring Boot项目结构及Test类代码。左侧是项目结构，包含`cn.itcast.day04_spring_test`包下的`demo01`目录，其中`Test`类被高亮显示。右侧是Test类代码，定义了`show`方法，通过`@RequestMapping`注解指定访问路径为`/test/show`，当访问该路径时，会执行`show`方法并打印“show方法执行了...”信息，最后返回“hello!”。该图片与文档中创建应用接口的内容相关，直观呈现了接口实现代码。](../../../../../Attachments/development/backend/web/spring-yu-springboot/NLvVbskUsohG5IxCm3CcoFNTnQb.png)



打开浏览器，输入 `http://localhost:8080/test/show`

![这张图片展示的是一个浏览器页面，地址栏显示访问的网址为`localhost:8080/test/show`，页面主体内容显示“hello!”，该内容对应上下文里在浏览器输入指定地址后运行测试的结果。浏览器顶部还排列着多个标签页的快捷方式，包括Gmail、YouTube、哔哩哔哩、Maven Repository等应用及学习相关的标签入口。](../../../../../Attachments/development/backend/web/spring-yu-springboot/PlmkbBnQKoDVjWxk6RzcbvO6n6f.png)

#### SpringBoot常见问题

大伙儿在下来联系的时候，联网基于spring的脚手架创建SpringBoot项目，偶尔可能会因为网内网络的原因，链接不上SpringBoot的脚手架网站，此时会出现如下现象：

![这张图片展示的是在创建SpringBoot项目时出现的初始化错误界面，对应文档中SpringBoot常见问题的相关内容。界面中弹出的错误窗口显示，初始化链接`https://start.spring.io`失败，错误信息为无法下载该地址，连接超时，提示需检查URL、网络及代理设置；界面背景可看到项目创建的相关设置项，包括选择的是Spring Initializr生成器，服务器URL为start.spring.io，项目名称设为demo，开发语言选择Java，构建工具为Maven，包类型为Jar，JDK版本为17。该界面呈现的正是文档中提及的因网络原因无法连接SpringBoot脚手架网站时的报错现象。](../../../../../Attachments/development/backend/web/spring-yu-springboot/H6Bjb3D6moXXhux418DcfblJnOb.png)

此时可以使用阿里云提供的脚手架，网址为：`https://start.aliyun.com`

![图片展示了在IntelliJ IDEA中使用阿里云提供的脚手架创建Spring Boot项目时的界面。左侧“New Module”下选中“Spring Boot”；右侧“Server URL”处显示“start.spring.io”，并有“?”标识；“Location”处输入“https://start.aliyun.com”；“Language”下选中“Java”；“Type”下选中“Maven”；“Group”处输入“cn.itcast”。该图片与文档中介绍使用阿里云脚手架创建Spring Boot项目的内容相关，直观呈现了操作步骤。](../../../../../Attachments/development/backend/web/spring-yu-springboot/SiiobRTg2olTShxWdCPcCVGPn9p.png)

然后按照项目创建的向导，一步一步的创建项目即可。

![这是IDEA创建Spring Boot项目的“New Module”界面，用于通过阿里云脚手架创建SpringBoot项目。界面左侧选择了Spring Boot类型的项目，右侧相关配置项被红色框和标记重点标注：项目名称设为“day-spring-test”，语言选择Java，类型为Maven，Group填为“cn.itcast”，Artifact是“day-spring-test”，包名同样为“cn.itcast”，JDK选用17版本，打包方式为Jar。界面右下角的“Next”按钮被红色标记突出，提示完成配置后可点击该按钮继续后续操作，这对应文档中使用阿里云脚手架创建项目的步骤内容。](../../../../../Attachments/development/backend/web/spring-yu-springboot/Qnh8bLuwDomva9xAoACcg6X5nlc.png)

选择Spring Boot版本 以及 勾选web开发相关依赖

![这是一张新建SpringBoot模块的配置界面，用于展示创建SpringBoot Web项目的依赖选择步骤。界面内，Spring Boot版本选择区高亮显示3.0.2版本；Dependencies列表中，Developer Tools选项下勾选了Lombok，Web选项下勾选了Spring Web；界面右下角的“Create”按钮被红框标注，作为最终创建项目的操作按钮。该图对应上下文里的SpringBoot项目创建环节，具体是使用阿里云脚手架创建项目时，选择Spring Boot版本、勾选Web开发相关依赖的操作展示。](../../../../../Attachments/development/backend/web/spring-yu-springboot/N0P5bk2cooC85YxxS8ycK5pPnpj.png)

修改层级名称 删除无用类

![这张图展示了SpringBoot项目创建后的文件目录结构，对应的是创建项目后修改层级名称的操作场景。目录中src/main/java路径下的cn.itcast.demos层级里，红色框出了controller目录，并有标注“修改层级名称”。该目录下有BasicController.java、PathVariableController.java、User.java这三个文件，被标注说明“可以删除阿里镜像给的引导”。此外，目录中还包含DaySpringTestApplication.java、resources、test等项目常规结构文件，契合文档中关于修改项目层级并删除无用类的内容要求。](../../../../../Attachments/development/backend/web/spring-yu-springboot/Tb2ob6EPUonq9kx9T0EcjBBGnWe.png)

当阿里云镜像创建项目时 如果模块失效，可以用下列步骤重新生效模块

![图片展示了IntelliJ IDEA中导入模块的操作界面。左侧导航栏中“File”选项被红色框标出，点击后弹出下拉菜单，其中“Project Structure...”选项也被红色框标出。右侧“Project”标签页下，“Modules”选项被红色框标出，其下有“Add”选项，再下有“Import Module”选项。下方“Sources”区域中，“Path”输入框被红色框标出，其右侧“...”按钮也被红色框标出。该图片与文档中介绍在IntelliJ IDEA中导入模块的操作步骤相关，直观呈现了操作位置。](../../../../../Attachments/development/backend/web/spring-yu-springboot/OgVabOUgaobRqrxFwdbcSeVwnLe.png)

## 依赖

### 依赖管理

（1）从仓库下载jar包

<dependiceis>

（2）解决依赖冲突问题

<callout emoji="🥖">
如果直接依赖和间接依赖冲突了，以直接依赖为主
如果直接依赖和直接依赖冲突了，后面的将前面的覆盖
如果间接依赖和间接依赖冲突了，层级一样声明优先
如果间接依赖和间接依赖冲突了，层级不一样深度浅的优先
</callout>

（3）传递依赖 ===>当引入一个jar的时候，这个jar依赖的所有的jar都会依赖传递过来

<callout emoji="🌅">
A ==>CD
B ==>DE
</callout>

(4)排除依赖

<callout emoji="🎨">
如果我们引入一个坐标, 不想要的传递依赖的时候可以通过 **exclusions 标签** 排除
</callout>

### 一键构造

<callout emoji="🏝️">
通过按钮的方式将编译，打包，测试，发布，安装..等功能按钮一键构建
</callout>

![这张图片展示的是Maven项目的界面，项目名称为day03_maven_test，在该项目的Lifecycle目录下，清晰列出了Maven的核心生命周期阶段，分别是clean、validate、compile、test、package、verify、install、site、deploy，这些是Maven编译打包等构建流程的关键步骤；界面右上角的“m”标识，对应文档中提到的“一键构造”功能，该功能可通过按钮完成编译、打包等操作，此界面正是Maven项目一键构造功能对应的操作区域，为该功能提供了可视化的操作入口。](../../../../../Attachments/development/backend/web/spring-yu-springboot/NS5wbq4Tjoizg7xQ9EJcTXNRnNe.png)

### 统一项目目录

<whiteboard token="K1InwfCpehEu50bpehVcqtSnnNf"></whiteboard>

```SQL
Maven项目的目录结构:
maven-project01
        |---  src  (源代码目录和测试代码目录)
               |---  main (源代码目录)
                        |--- java (源代码java文件目录)
                        |--- resources (源代码配置文件目录)
              |---  test (测试代码目录)
                        |--- java (测试代码java目录)
                        |--- resources (测试代码配置文件目录)
        |--- target (编译、打包生成文件存放目录)
```

![这是一个SpringBoot项目的目录结构截图，标注了项目相关的核心目录与文件。最外层是名为“javaweb142”的项目根目录，其下包含多个模块，重点标注了“day04-spring-test”模块。在该模块内，标注了“src”目录下的“main”和“test”子目录：main目录下的“java”目录用于存放Java源码，“resources”目录用于存放配置文件；test目录下的“java”目录则用于存放测试文件。同时还展示了项目内的其他目录与文件，如target目录、pom.xml等。](../../../../../Attachments/development/backend/web/spring-yu-springboot/ODMlblKrToomhLx0kYbctQeYnze.png)

## 三层架构



![图片以饭店为例，展示了软件系统三层架构。上方是饭店，有服务员、厨师、采购员。下方是软件系统，分为表示层、业务逻辑层、数据访问层。表示层与业务逻辑层、数据访问层有双向箭头，业务逻辑层与数据访问层也有双向箭头。该图与文档中三层架构内容相关，通过类比饭店各角色职责，直观呈现了软件系统中表示层、业务逻辑层、数据访问层的交互关系。](../../../../../Attachments/development/backend/web/spring-yu-springboot/DBFZbWrOto7wqCxGgfNcwAY7nx3.png)

即：

![图片展示了三层架构中后台程序的职责划分。控制层（Controller）负责接收前端传入的参数，封装参数，调用service层处理业务得到结果，再将结果响应给前端；业务层（Service）接收Controller层数据，处理数据、校验数据等，调用Dao层持久化操作，将结果返回Controller层；持久层（Dao）接收Service层数据，执行数据库操作，将结果返回Service层。该图与文档中三层架构部分内容对应，直观呈现了各层职责。](../../../../../Attachments/development/backend/web/spring-yu-springboot/HwprbxymboQjf5xV6xKcwlCPnrc.png)



控制层（Controller）

<callout emoji="👍">
1. 接收前端请求参数
2. 封装请求参数
3. 调用 Service 层处理业务
4. 将处理结果响应给前端
</callout>

业务层（Service）

<callout emoji="🎉">
1. 接收 Controller 层传递的数据
2. 进行业务逻辑处理（校验、补全、组织数据）
3. 调用 Dao 层进行数据持久化操作
4. 将处理结果返回给 Controller 层
</callout>

持久层（Dao）

<callout emoji="🥖">
1. 接收 Service 层传递的数据
2. 执行数据库的增删改查操作
3. 将数据库操作结果返回给 Service 层
</callout>

### 代码示例

```Java
package cn.itcast.controller;

import cn.itcast.service.ProductsService;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/products")
public class ProductsController {
    ProductsService ps = new ProductsService();
    @RequestMapping("/findAll")
    public void findAll(){
        System.out.println("1.接受前端传入的参数");
        System.out.println("2.封装参数");
        System.out.println("3.调用service层的功能 处理封装好的数据并得到结果");
        System.out.println("==========================");
        ps.findAll();
        System.out.println("==========================");
        System.out.println("4.将结果相应给前端");
    };
}
```

```Java
package cn.itcast.service;

import cn.itcast.dao.ProductsDao;

public class ProductsService {
    ProductsDao pd = new ProductsDao();
    public void findAll(){
        System.out.println("1.接受Controller层给的数据");
        System.out.println("2.处理数据 补全数据 校验数据 组织数据");
        System.out.println("3.调用Dao层将数据持久化或者更新得到Dao层的结果");
        System.out.println("==========================");
        pd.findAll();
        System.out.println("==========================");
        System.out.println("4.将结果返回controller层");
    }
}
```

```Java
package cn.itcast.dao;

public class ProductsDao {
    public void findAll(){
        System.out.println("1.接受Service层处理好的数据");
        System.out.println("2.持久化操作数据库 更新");
        System.out.println("3.将结果返回Service层");
    }
}
```

## 反射

### 反射的作用

<callout emoji="🎉">
在运行期间动态获取方法区的字节码对象，并且解剖字节码每一部分并操作的一种技术
土话：操作类的另外一种方式
</callout>

### 普通方式操作类

<callout emoji="📌">
（1）创建对象
（2）访问对象的属性
（3）访问对象方法
</callout>

### 反射方式操作类

前提

<callout emoji="🍞">
**须先获取方法区才能反射**
</callout>

#### 获取字节码对象的三种方式

```Java
类名.class;
对象.getClass();
Class.forName("全限定类名");
    全限定类名：
        包名 + "." + 类名
```

代码示例：

```Java
    private static void demo01() throws ClassNotFoundException {
        // 获取字节码对象的方式
        Class<Student> stu1 = Student.class;
        //获取字节码的第二种方式 对象.getClass()
        Student stu = new Student();
        Class<? extends Student> stu2 = stu.getClass();
        // 获取字节码的第三种方式 Class.forName("全类名")
        Class<?> stu3 = Class.forName("cn.itcast.demo01.Student");
//        System.out.println(stu1 == stu2);
        System.out.println(stu1 == stu3);
    }
```

#### *用反射创建对象*

```Java
创建对象的方式
    Constructor getconstructor(classclazz）//只能获取public修饰的单个构造函数
    Constructor getDeclaredconstructor(Class...clazz）//只能获取public修饰的单个构造函数
    Constructor[] getConstructors（）//获取所有public修饰的构造函数
    Constructor[] getDeclaredConstructors() //获取所有构造函数
    Constructor具备功能：
        Object newInstance(Object... args);
暴力反射：
    setAccessible(boolean isAccessible);
```

<callout emoji="🦄">
1. 第一步：获取字节码对象
2. 第二步：获取无参/有参构造

   1. 字节码对象.getConstructor()
3. 第三步：暴力反射

   1. 有参/无参构造对象.setAccessible
4. 第四步：使用newInstance创建对象

   1. 有参/无参构造对象.newInstance(参数列表)
</callout>

```Java
/**
 * 采用反射创建对象
 * Constructor getConstructor(Class ... clazz) 只能获取public修饰的单个构造函数
 * Constructor getDeclaredConstructor(Class ... clazz) 只能获取public修饰的单个构造函数
 * Constructor[] getConstructors() 获取所有public修饰的构造函数
 * Constructor[] getDeclaredConstructors() 获取所有构造函数
 * Constructor具备功能:
 * Object newInstance(Object... args);
 * 暴力反射:
 * setAccessible(boolean isAccessible);
 */
 
 private static void demo02() throws Exception {
    Class<Student> studentClass = Student.class;
    //1. 获取无参构造
    Constructor<Student> noArgsConstructor = studentClass.getDeclaredConstructor();
    //2. 获取有参构造
    Constructor<Student> oneArgsConstructor = studentClass.getDeclaredConstructor(String.class, int.class);
    //3. 暴力反射
    noArgsConstructor.setAccessible(true);
    oneArgsConstructor.setAccessible(true);
    //4. 使用newInstance创建对象
    Student student = noArgsConstructor.newInstance();
    Student student1 = oneArgsConstructor.newInstance("小王", 18);
    System.out.println(student);    //Student(name=null, age=0)
    System.out.println(student1);   //Student(name=小王, age=18)
}
```

#### *采用反射操作成员变量*

```Java
//采用反射操作成员变量
    Field getField(String name); //只能获取public修饰的成员变量
    Field[] getFields(); //获取所有public修饰的成员变量
    Field getDeclaredField(String name); //获取任何修饰符修饰的单个成员
    Field[] getDeclaredFields(); //获取所有修饰符修饰的成员变量
//使用成员变量对象的方法进行操作
    void set(对象，值); //对该对象中的这个成员变量设置值
    Object get(对象);  //获取该对象身上的成员变量的值
    void setAccessible(boolean isAccessible);//暴力反射

```

<callout emoji="🎨">
1. 第一步：获取字节码对象
2. 第二步：获取成员变量对象

   1. 字节码对象.getDeclareField（成员变量名）
3. 第三步：创建实例对象

   1. 字节码.newInstance
4. 第四步：设置变量值/获取变量值

   1. 成员变量对象.set/get
</callout>

```Java
private static void demo03() throws Exception {
    // 1. 获取 Student 类对应的 Class 对象（反射的入口）
    Class<?> studentClass = Student.class;
    // 2. 获取 public 修饰的 name 属性
    // 如果 name 不是 public，会抛出 NoSuchFieldException
    // Field name = studentClass.getField("name");
    // 3. 获取 name 属性（无论是 public / private / protected / 默认）
    Field name = studentClass.getDeclaredField("name");
    // 打印 name 字段的信息（修饰符 + 类型 + 名称）
    System.out.println(name);
    // 4. 获取 age 属性（通常是 private）
    Field age = studentClass.getDeclaredField("age");
    // 5. 获取 Student 类中声明的所有属性（不包含父类）
    Field[] declaredFields = studentClass.getDeclaredFields();
    // 遍历并打印所有属性信息
    for (Field field : declaredFields) {
        System.out.println(field);
    }
    // 6. 暴力反射
    // 取消 Java 的访问权限检查，使 private 属性可以被访问
    name.setAccessible(true);
    age.setAccessible(true);
    // 7. 通过无参构造方法创建 Student 对象
    Student student = new Student();
    // 8. 使用反射为对象的 name 属性赋值
    name.set(student, "张三");
    // 9. 使用反射为对象的 age 属性赋值
    age.set(student, 18);
    // 10. 打印对象，验证属性是否成功赋值
    System.out.println(student);
}

```

#### **利用反射操作成员方法**

<callout emoji="🎁">
1. 第一步：获取字节码对象
2. 第二步：拿到成员方法对象

   1. 字节码对象.getDeclaredMethod("方法名", 方法的参数列表的数据类型对象……)
3. 第三步：通过成员方法对象调用成员方法

   1. 成员方法对象.invoke("实体类对象名", 对方法的参数列表赋值)
</callout>

```Java
/**
 * 采用反射操作成员方法
 *      Method  getMethod();
 *      Method[]  getMethods();
 *      Method  getDeclaredMethod();
 *      Method[]  getDeclaredMethods();
 * 使用Method对象的功能执行该方法
 *      Object  invoke(对象, 对方法的参数进行赋值);
 */
 
 private static void demo04() throws Exception {
    //1.获取字节码对象
    Class<Student> studentClass = Student.class;
    //2. 通过字节码对象的getDelaredMethod拿到成员方法对象
    Method declaredMethod = studentClass.getDeclaredMethod("show", String.class);
    //3.通过反射创建对象
    Student student = studentClass.newInstance();
    //通过成员方法对象调用invoke方法，调用成员方法
    declaredMethod.invoke(student,"小红");
    //4.通过对象给成员变量赋值
    student.setName("小王");
    student.setAge(18);
    //通过字节码对象拿到成员方法对象（无参数的）
    Method declaredMethod1 = studentClass.getDeclaredMethod("show");
    //通过成员方法对象调用成员方法
    declaredMethod1.invoke(student);
}
```

## 分层解耦

### 层与层之间解耦合

#### 业务层面向接口编程的好处

<callout emoji="📚">
通过反射达到解耦合的目的
</callout>

### 问题分析

由于我们现在在程序中，需要什么对象，直接new一个对象 `new EmpDaoImpl()`  。

![这张图片展示了Java代码实现的分层架构相关示例，上方是`EmpServiceImpl`类的代码，其中定义了`EmpDao`类型的成员变量，直接实例化了`EmpDaoImpl`，下方是`EmpDaoImpl`类的代码，实现了`EmpDao`接口的`findAll`方法，该方法负责读取本地的`emp.txt`文件获取数据。图片中红色箭头和标注的内容，直观体现了业务层直接硬编码依赖具体实现类`EmpDaoImpl`的情况，这对应了上下文提到的“需要直接new一个对象”导致后续更换实现类时需修改代码的问题。](../../../../../Attachments/development/backend/web/spring-yu-springboot/PxLybWiEso5Vj3xUkatcx1JhnVf.png)

如果说我们需要更换实现类，比如由于业务的变更，EmpDaoImpl 不能满足现有的业务需求，我们需要切换为 EmpDaoImpl2 这套实现，就需要修改Contorller的代码，需要创建 EmpDaoImpl2 的实现`new EmpDaoImpl2()` 。

![图片展示了EmpService和EmpDaoImpl两个类的代码。EmpServiceImpl类中，empDao属性被赋值为EmpDaoImpl2的实例。EmpDaoImpl类实现了EmpDao接口，包含findAll()方法，该方法通过JDBC操作数据库获取员工数据。图片与上下文紧密相关，通过展示具体代码，直观呈现了上下文提到的直接new对象的问题，即在程序中需要什么对象就new一个对象，如new EmpDaoImpl()，若需更换实现类，需修改相关代码，如创建EmpDaoImpl2的实现new EmpDaoImpl2()，以说明业务层面向接口编程的好处。](../../../../../Attachments/development/backend/web/spring-yu-springboot/CGX3bGjcmopoXuxtbCgccgtAnKg.png)

Controller中调用Service，也是类似的问题。这种呢，我们就称之为层与层之间 **耦合** 了。 那什么是耦合呢 ？

首先需要了解软件开发涉及到的两个概念：内聚和耦合。

- **内聚：**软件中各个功能模块内部的功能联系。
- **耦合：**衡量软件中各个层/模块之间的依赖、关联的程度。



<callout emoji="⛳">
**软件设计原则：高内聚低耦合。**
**高内聚：**指的是一个模块中各个元素之间的联系的紧密程度，如果各个元素(语句、程序段)之间的联系程度越高，则内聚性越高，即 "高内聚"。
**低耦合：**指的是软件中各个层、模块之间的依赖关联程序越低越好。
</callout>

目前层与层之间是存在耦合的，Controller耦合了Service、Service耦合了Dao。而 高内聚、低耦合的目的是使程序模块的可重用性、移植性大大增强。

那最终我们的目标呢，就是做到层与层之间，尽可能的降低耦合，甚至解除耦合。

![图片展示了三层架构中层与层之间解耦合前后的对比。左侧为当前情况，controller依赖service，service依赖dao，用红色圈标注；右侧为理想状态，controller、service、dao之间无直接依赖，用绿色箭头表示。该图与上下文紧密相关，直观呈现了层与层之间耦合与解耦合的现状，为理解业务层面向接口编程的好处及后续解耦操作提供可视化参考。](../../../../../Attachments/development/backend/web/spring-yu-springboot/ASS3b6pGUo1IAUxN2x2cWGBInvf.png)

### 解耦思路

之前我们在编写代码时，需要什么对象，就直接new一个就可以了。 这种做法呢，层与层之间代码就耦合了，当Dao层的实现变了之后， 我们还需要修改Service层的代码。

那应该怎么解耦呢？



**1). 首先不能在EmpServiceImpl中使用new对象。代码如下：**

![图片展示的是EmpServiceImpl类代码，该类实现了EmpService接口。代码中定义了EmpDao类型的empDao私有成员变量，下方有注释“不要再new对象了”。类中有一个findAll()方法，用于获取所有Emp对象。此图片与上文提到的不能new出Dao层对象的问题相关，通过这种方式，Service程序从容器中获取EmpService类型的对象，应用程序用到对象时直接从容器中获取，实现了对象交给容器管理的目标。](../../../../../Attachments/development/backend/web/spring-yu-springboot/HbgbbkRWnoluUcx9PkAcEAKZnLE.png)

此时，就存在另一个问题了，不能new，就意味着没有Dao层对象（程序运行就报错），怎么办呢? 

我们的解决思路是：

- 提供一个容器，容器中存储一些对象(例：EmpDao对象)
- Service程序从容器中获取EmpService类型的对象

**2). 将要用到的对象交给一个容器管理。**

![图片展示了两个EmpDao接口实现类的代码片段。上方是EmpDaoImpl1，下方是EmpDaoImpl2，均实现了EmpDao接口。下方代码中，通过Class.forName()加载数据库驱动，使用DriverManager.getConnection()获取数据库连接，执行SQL查询语句，获取结果集并遍历。图片下方有一个黄色圆角矩形框，内有数字“1”，与上文提到的“那问题来了，我们如何将对象交给容器管理呢？”相呼应，暗示这是容器管理对象的示例代码。](../../../../../Attachments/development/backend/web/spring-yu-springboot/Wd3IbOLJdoy4JExtVf3caiNcnBg.png)

**3). 应用程序中用到这个对象，就直接从容器中获取**

![图片展示了EmpServiceImpl类与EmpServiceImpl2类的代码关系。左侧EmpServiceImpl类中，有empDao属性，其类型为EmpDao。右侧EmpServiceImpl2类中，包含数据库连接、查询数据等代码。下方蓝色区域标注为1，与右侧代码区域有红色箭头指向，表明两者存在关联。该图与上下文关系紧密，直观呈现了上下文中提到的将对象交给容器管理的思路，即应用程序用到对象时，从容器中获取，而对象由容器管理。](../../../../../Attachments/development/backend/web/spring-yu-springboot/SRzwblOpnoxMb6xe19oc6icqnEc.png)

那问题来了，我们如何将对象交给容器管理呢？ 程序运行时，容器如何为程序提供依赖的对象呢？ 

我们想要实现上述解耦操作，就涉及到Spring中的两个核心概念：

<callout emoji="📌">
- **控制反转：** Inversion Of Control，简称**IOC**。对象的创建控制权由程序自身转移到外部（容器），这种思想称为控制反转。

  - 对象的创建权由程序员主动创建转移到容器(由容器创建、管理对象)。这个容器称为：IOC容器或Spring容器。
- **依赖注入：** Dependency Injection，简称**DI**。容器为应用程序提供运行时，所依赖的资源，称之为依赖注入。

  - 程序运行时需要某个资源，此时容器就为其提供这个资源。
  - 例：EmpController程序运行时需要EmpService对象，Spring容器就为其提供并注入EmpService对象。
- **bean对象：**IOC容器中创建、管理的对象，称之为：bean对象。
</callout>

### IOC&DI入门

IOC

<callout emoji="🚅">
控制反转，将对象的创建权交给spring处理，可以将对象放入到ioc容器中，可以结合di依赖注入，达到解耦合的目的

@Component 

@Controller

@Service

@Repository
</callout>

DI

```Java
@Autowired //自动装配，从ioc容器中获取该类型的bean然后注入
```

**1). 将Service及Dao层的实现类，交给IOC容器管理**

在实现类加上 `@Component` 注解，就代表把当前类产生的对象交给IOC容器管理。

接口+IOC+DI方法示例：

**A. EmpDaoImpl**

```Java
package cn.itcast.emp.dao.impl;

import cn.itcast.emp.dao.EmpDao;
import org.springframework.stereotype.Repository;

@Repository
public class EmpDaoImpl implements EmpDao {
    @Override
    public void show(){
        System.out.println("Dao 1.接收Service层的数据");
        System.out.println("Dao 2.持久化操作数据库");
        System.out.println("Dao 3.将结果返回Service层");
    }
}
```

```Java
package cn.itcast.emp.dao;

public interface EmpDao {
    void show();
}
```

**2). 为Controller 及 Service注入运行时所依赖的对象**

**A. EmpServiceImpl**

```Java
package cn.itcast.emp.service.impl;

import cn.itcast.emp.dao.impl.EmpDaoImpl;
import cn.itcast.emp.service.EmpService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class EmpServiceImpl implements EmpService {
    @Autowired
    EmpDaoImpl ed;
    @Override
    public void show(){
        System.out.println("Service: 1.接受Controller的数据");
        System.out.println("Service: 2.处理数据 校验数据 补全 组织数据");
        System.out.println("Service: 3.调用Dao层进行持久化操作");
        ed.show();
        System.out.println("Service: 4.将结果返回给Controller");
    }
}
```

```Java
package cn.itcast.emp.service;

public interface EmpService {
    void show();
}
```

**B. EmpController**

```Java
package cn.itcast.emp.controller;

import cn.itcast.emp.service.impl.EmpServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/emp")
@Controller
public class EmpController{
    @Autowired
    private EmpServiceImpl empService;

    @RequestMapping("/show")
    public void show() {
        System.out.println("Controller 1.接受前端请求");
        System.out.println("Controller 2.封装参数");
        System.out.println("Controller 3.调用service层功能处理参数");
        empService.show();
        System.out.println("Controller 4.将结果返回给前端");
    }
}
```

启动服务，运行测试。 打开浏览器，地址栏直接访问：http://localhost:8080/emp/show 。 依然正常访问，就说明入门程序完成了。 已经完成了层与层之间的解耦。



## 附录：常见状态码

| **状态码** | **英文描述** | **解释** |
|-|-|-|
| 200 | OK | 客户端请求成功，即处理成功，这是我们最想看到的状态码 |
| 302 | Found | 指示所请求的资源已移动到由Location响应头给定的 URL，浏览器会自动重新访问到这个页面 |
| 304 | Not Modified | 告诉客户端，你请求的资源至上次取得后，服务端并未更改，你直接用你本地缓存吧。隐式重定向 |
| 400 | Bad Request | 客户端请求有语法错误，不能被服务器所理解 |
| 403 | Forbidden | 服务器收到请求，但是拒绝提供服务，比如：没有权限访问相关资源 |
| 404 | Not Found | 请求资源不存在，一般是URL输入有误，或者网站资源被删除了 |
| 405 | Method Not Allowed | 请求方式有误，比如应该用GET请求方式的资源，用了POST |
| 428 | Precondition Required | 服务器要求有条件的请求，告诉客户端要想访问该资源，必须携带特定的请求头 |
| 429 | Too Many Requests | 指示用户在给定时间内发送了太多请求（“限速”），配合 Retry-After(多长时间后可以请求)响应头一起使用 |
| 431 | Request Header Fields Too Large | 请求头太大，服务器不愿意处理请求，因为它的头部字段太大。请求可以在减少请求头域的大小后重新提交。 |
| 500 | Internal Server Error | 服务器发生不可预期的错误。服务器出异常了，赶紧看日志去吧 |
| 503 | Service Unavailable | 服务器尚未准备好处理请求，服务器刚刚启动，还未初始化好 |

- 状态码大全：https://cloud.tencent.com/developer/chapter/13553

---
> 来源：飞书技术知识库（脱敏版） · 03 Web 开发与框架 / Spring 与 SpringBoot / 03 SpringBoot基础开发_三层架构_反射分层解耦 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/NV1AwQaHnipdCAk9ozNcAsUWned)
