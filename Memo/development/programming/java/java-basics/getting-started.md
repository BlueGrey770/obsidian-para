---
type: memo
date: 2026-11-09
aliases:
  - "Java快速入门"
---

# Java 快速入门

Hello，各位小伙伴，今天我们开始正式学习Java的入门知识，第一天主要带着大家了解一下Java是什么、Java的常用开发工具以及Java的基本语法。

## Java是什么

### Java语言的历史

- **Java是哪家公司的产品？**

Java是美国**Sun**（Stanford University Network，斯坦福大学网络公司）公司在1995年推出的一门计算机高级编程语言。但是在2009年是Sun公司被Oracle（甲骨文）公司给收购了，所以目前Java语言是Oracle公司所有产品。

- **Java名称的来历？**

早期这门语言的名字其实不叫Java，当时称为Oak（橡树的意思），为什么叫橡树呢？原因是因为Sun公司的门口种了很多橡树，但是后来由于商标注册时，Oak商标已经其他公司注册了，所以后面改名为Java了。那么有人好奇为什么叫Java呢？Java是印度的一个岛屿，上面盛产咖啡，可能是因为他们公司的程序员喜欢喝咖啡，所以就改名为Java了。

- **Java的创始人是谁？**

说完Java名称的来历之后，接下来我们聊聊Java的祖师爷是谁？ Java的联合创始人有很多，但是行业普遍认可的Java的创始人 是**詹姆斯●高斯林**，被称为Java之父。

![M9cFbctSpoNHDYxSZnlc152DnZd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/M9cFbctSpoNHDYxSZnlc152DnZd.png)

### Java的优势

- Java是世界上最流行的编程语言之一，在国内是使用最为广泛的编程语言。

![F65zbon8SoVAC2xByAhcK912n3B](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/F65zbon8SoVAC2xByAhcK912n3B.png)

- Java的开发社区完善，功能丰富。
- Java具有很好的可移植、安全可靠、性能较好。

### Java能做什么

了解了Java语言的历史及优势之后，接下来，大家比较关心的问题可能是Java到底能做什么了？

其实Java能做的事情非常多，它可以做桌面应用的开发、企业互联网应用开发、移动应用开发、服务器系统开发、大数据开发、游戏开发等等。

- 桌面应用开发：能够在电脑桌面运行的软件

    举例：财务管理软件、编写程序用的IDEA开发工具等，可以用Java语言开发

- 企业级应用开发：大型的互联网应用程序

    举例：淘宝、京东、大家每天都用的tlias教学管理系统等

- 移动应用开发：运行的Android手机端的软件

    举例：QQ客户端、抖音APP等

- 服务器系统：应用程序的后台（为客户端程序提供数据）

    举例：服务器系统为用户推荐那你喜爱的视频

- 大数据开发：大数据是一个互联网开发方向

    举例：目前最火的大数据开发平台是Hadoop，就是用Java语言开发的

- 游戏开发：游戏本质上是给用户提供娱乐的软件，有良好的交互感受

    举例：我的世界MineCraft就是用Java语言开发的

<callout emoji="💡">
注意
虽然Java能做的事情非常多，但并不是每一个方向都被市场认可（比如桌面应用使用Java语言开发就不太方便，而使用C#语言是比较推荐的）。**目前Java的主流开发方向是使用Java开发企业级互联网应用程序**（很多公司的OA系统，客户关系管理系统，包括传智播客使用教学实施管理系统都是用Java语言开发的）
</callout>

### Java的技术体系

说完Java语言能做什么之后，接下来我们再给同学们介绍一下Java的技术体系。所谓技术体系，就是Java为了满足不同的应用场景提供了不同的技术版本，主要有三个版本。

![CsjWbfKKLoUe80xkgsMci8X0nKR](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/CsjWbfKKLoUe80xkgsMci8X0nKR.png)

1. Java SE（Java Standard Edition）

Java标准版，它是后面两个版本的基础，也就是学习后面两个版本必须先学习JavaSE。**我们基础班现阶段学习的就是这个版本中的技术**。

1. Java EE（Java Enterprise Edition）:

Java企业版，它是为企业级应用开发提供的一套解决方案。**在后面就业班课程中主要学习这个版本中的技术**。

1. Java ME（Java Micro Edition）

Java小型版，它为开发移动设备的应用提供了一套解决方案。**目前已经不被市场认可（淘汰），取而代之的是基于Android系统的应用开发、鸿蒙、或者IOS系统**。

### 小结

通过上面的介绍，我们对Java有了基本的了解，关于这些内容小伙伴们也不用刻意去记，简单了解即可。下面我们在简要回顾一下：

1.Java是什么？

    答：Java是一门高级编程语言

2.Java是哪家公司的产品？

    答：Java以前是Sun公司的产品，现在Java是属于Oracle公司的产品

3.Java之父是谁？

    答：詹姆斯●高斯林

4.Java主流的开发方向是什么？

    答：企业级互联网应用开发

5.Java技术平台有哪些？

    答：JavaSE（标准版）、JavaEE（企业版）、JavaME（小型版）

## 如何开发一个Java程序

通过上面的学习，我们对Java有了基本了解，接下来，我们来带大家开发一个Java程序，让同学们了解下开发一个Java程序的基本步骤。 通过询问AI我们也可以知道开发一个Java程序总体需要如下一些步骤：

![CGtvb7r61oMVsHx1AgmcPcaNnBg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/CGtvb7r61oMVsHx1AgmcPcaNnBg.png)

![A4A7b6zrUoF2YkxdcU9c3CGvnBe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/A4A7b6zrUoF2YkxdcU9c3CGvnBe.png)
![ReV8bNZifoE6M5xXYnBcV5z4nTh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ReV8bNZifoE6M5xXYnBcV5z4nTh.png)

接下来我们逐个步骤进行学习。

### 安装JDK

#### 概述

JDK（全称是Java Develop Kit），翻译过来意思就是Java开发工具包,必须先安装JDK才可以开发Java程序并运行，就像你先需要安装微信，才能使用微信和朋友聊天是一样的。**我们先要到官网上去下载JDK，然后安装在自己的电脑上，才可以在自己的电脑上使用JDK来开发Java程序，由于**JDK的版本有很多，下图是JDK版本更新的历程图，有LTS标识的是长期支持版本（意思就是Oracle会不定期更新）。

![LScEbVaP9oGWOGxySoic5gdEnMf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/LScEbVaP9oGWOGxySoic5gdEnMf.png)

目前公司中用得最多的版本是JDK11版本，在目前这套课程中我们为了讲一些新特性会使用JDK21版本。

注意：版本问题不是很重要的问题，以后需要切换版本也是很容易的事情，我们就先用JDK 21。

接下来我们开发下载JDK

#### JDK的下载

这是JDK下载的官方网址 [https://www.oracle.com/java/technologies/downloads/](https://www.oracle.com/java/technologies/downloads/)，你需要把该网址复制到浏览器的地址栏，敲回车

![L4cybwmbyoRIanxv3UGc4mvNnIe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/L4cybwmbyoRIanxv3UGc4mvNnIe.png)

进入网址后，选择JDK21版本，找到Windows标签，选择x64 Installer版本。如下图所示

![RXCLbybxmovpTyxQgB2cqtxsndc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/RXCLbybxmovpTyxQgB2cqtxsndc.png)

下载完成之后，在你下载的目录下会出现一个JDK的安装包，如下图所示

![NRLnboGIkoqd57x3fepcRtj3nAf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/NRLnboGIkoqd57x3fepcRtj3nAf.png)

到这JDK的下载就完成了，接下来就需要按照下面的步骤完成JDK安装.

#### JDK的安装

双击安装包，按照下图引导，点击下一步即可安装。**需要注意的是安装JDK后不像你安装QQ一样会在桌面上显示一个图标，JDK安装后桌面上没有图标！！！**

![BfhhbOkS6oRATWxPa3PcJqP9nAg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/BfhhbOkS6oRATWxPa3PcJqP9nAg.png)

![Ayz0bMSRnoiqLqxscNvcqcB3nbe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Ayz0bMSRnoiqLqxscNvcqcB3nbe.png)
![TxyMbJS9toJFb0x6si6cMmp7nPh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/TxyMbJS9toJFb0x6si6cMmp7nPh.png)

![EqQVb5ymPoInD9x0FcLc8h0wnQh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/EqQVb5ymPoInD9x0FcLc8h0wnQh.png)
![IorDbMlBPoXYnSxT0locYKA8nhh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/IorDbMlBPoXYnSxT0locYKA8nhh.png)

![UhphbTYKwolVBLxBMKzcOKR4nKf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/UhphbTYKwolVBLxBMKzcOKR4nKf.png)
![KBSLbm4KFo4Bo2xu5W5culaXnOe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/KBSLbm4KFo4Bo2xu5W5culaXnOe.png)

#### JDK的验证

JDK安装后，还要验证其是否可以使用，主要是看你安装的JDK，安装目录下的bin包里面的两个命令程序是否可以使用，依次是javac、java命令程序，如下图所示：

![SIOIbSLYeom8cJx6lW3cRkghngg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/SIOIbSLYeom8cJx6lW3cRkghngg.png)

我们先了解一下这两个程序是干啥的？然后再看如何验证其是否可以使用

- **javac 称之为编译工具**，负责对我们将来写的java程序进行编译的（翻译）
- **Java 称之为执行工具**，负责对编译后的java程序进行执行的。

**为什么要有这两个命令**

  因为**`我们写好的``Java程序``都是``高级语言``，``计算机底层是硬件不能识别这些语言``，``必须先通过javac编译工具进行翻译``，然后再通过``java执行工具执行``才可以驱动机器干活``。`**

**怎么验证这两个程序是否可以直接使用？**

如果大家双击启动Javac和Java是没有反应的，他们不是图形界面化程序，不支持双击启动，必须在电脑的命令行窗口下使用，如何启动命令行窗口呢？

**1、直接同时按住两个键：win + R   如下图所示**

![N5flbqMkuoJXuhxElk0cHxsGnwf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/N5flbqMkuoJXuhxElk0cHxsGnwf.png)

**2、在左下角启动窗口中输入 cmd 回车，就打开了命令行窗口了**

![SMRRbyDX8oFl8rxt86IcXfqEnzg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/SMRRbyDX8oFl8rxt86IcXfqEnzg.png)
![VseBbqHoRohxcLxh3lmcqAcHnQd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/VseBbqHoRohxcLxh3lmcqAcHnQd.png)

**3、在命令行窗口中输入 “java -version”，以及“javac -version”，看提示的版本号是否与安装的JDK 21版本一致**

![HGLabsoOQoub1OxIhPscRgOhn5g](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/HGLabsoOQoub1OxIhPscRgOhn5g.png)

**如果提示的版本号也是 21.0.1 ，恭喜你，你的JDK已经安装成功了！**

#### JDK的组成

下面我们把JDK的组成，

![ZGq1bP9JkoU9krxGD4xcWjgEnm2](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ZGq1bP9JkoU9krxGD4xcWjgEnm2.png)

下面分别介绍一下JDK中每一个部分是用来干什么的：

- JVM

JDK最核心的组成部分是JVM（Java Virtual Machine），它是Java虚拟机，真正运行Java程序的地方

- 核心类库

它是Java本身写好的一些程序，给程序员调用的。 Java程序员并不是凭空开始写代码，是要基于核心类库提供的一些基础代码，进行编程。

- JRE

JRE（Java Runtime Enviroment），意思是Java的运行环境；它是由JVM和核心类库组成的；如果你不是开发人员，只需要在电脑上安装JRE就可以运行Java程序。

- 开发工具

Java程序员写好源代码之后，需要编译成字节码，这里会提供一个编译工具叫做javac.exe，编写好源代码之后，想要把class文件加载到内存中运行，这里需要用到运行工具java.exe。
除了编译工具和运行工具，还有一些其他的工具，如：反编译工具、文档工具等...

### 配置JDK的环境变量

#### 什么是Path环境变量

- JDK安装完成了，按照AI开发入门程序的步骤，我们还需要Path和JAVA_HOME**配置环境变量。**

![DA0YbT66toZxN3xVoLic5cl6nPl](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/DA0YbT66toZxN3xVoLic5cl6nPl.png)
![H7rSb480koz3g6x6Eprci9Nbnob](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/H7rSb480koz3g6x6Eprci9Nbnob.png)

首先来学习Path环境变量，我们先来看**什么是Path环境变量？再看为什么要配置环境变量?**

- **Path环境变量是用来记住程序的路径，以方便程序员在命令行窗口的任意目录下启动程序**；
- 比如实现在命令行窗口的任意目录下输入QQ.exe程序的名称就直接启动QQ

![BpGPb6SHRoiOATxKX6LcHv0PnR7](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/BpGPb6SHRoiOATxKX6LcHv0PnR7.png)

#### 如何配置环境变量呢？

比如把QQ的启动程序，配置到Path环境变量下就可以在任意目录下启动QQ，按照一下步骤操作。

**第一步：**先找到QQ启动程序所在的目录，我的QQ位置在：D:\soft\Tencent\QQ\Bin

![PcbMbKxFPoh0qxxuUCicolhEnuf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/PcbMbKxFPoh0qxxuUCicolhEnuf.png)

复制这个路径： D:\soft\Tencent\QQ\Bin

**第二步：**按照下面的步骤，找到Path环境变量。

首先找到此电脑，右键点击属性，可以按照下面的界面；点击【高级系统设置】，再点击【环境变量】

双击Path后，点击新建，把QQ启动目录粘贴进来，不要忘记点确定哦^\_^

![XEQmbaD2WogbaqxFggTcUZoinae](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/XEQmbaD2WogbaqxFggTcUZoinae.png)
![VLUYbOrCyolGsTxG2rHcFVkPnQe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/VLUYbOrCyolGsTxG2rHcFVkPnQe.png)

![NfUqb7zh4otbjzxLJiBc79bfnaf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/NfUqb7zh4otbjzxLJiBc79bfnaf.png)
![CoPFbTDi1oQoYTxdIGDcCHORnug](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/CoPFbTDi1oQoYTxdIGDcCHORnug.png)

![OFyjbJGvIoRXs0x4WqbcZDYNnLm](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/OFyjbJGvIoRXs0x4WqbcZDYNnLm.png)

**第三步：**配置好之后，重启命令行窗口，检查是否配置成功

1.Win+R 输入cmd回车，打开命令行窗口，输入QQ，可以看到QQ启动了，代表配置QQ程序的Path环境变量成功了。

![FjV4bNjn3ouBEnxpIGEcKWv1nDb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/FjV4bNjn3ouBEnxpIGEcKWv1nDb.png)

#### 配置JDK的Path环境变量

 **将JDK的javac和java程序路径，配置到Path环境变量下**

为了方便在命令行窗口的任意目录下直接是要javac和java程序，以便可以在任意目录下能直接用Javac编译程序，直接用java执行程序。意味着就必须也把**JDK的javac和java程序路径，配置到Path环境变量下，以方便在命令行窗口的任意目录下直接使用这两个程序。**

**注意：**

<callout emoji="💡">
较新版本的JDK实际上你在安装该JDK时，JDK会自动把自己的javac和java程序的路径配置到Path环境变量下，因此大家会发现，你现在在命令行窗口的任意目录下就可以直接用javac和java程序。
那为什么还要配置Path环境变量呢？那是因为如果在公司里遇到了老版本的JDK,安装JDK时是不会自动把自己的javac和java程序的路径配置到Path环境变量下的，这就意味着程序员还是要自己配置JDK的Path环境变量。
</callout>

Path环境变量的配置步骤，跟配置QQ程序的Path环境变量类似，步骤如下：

**第一步：**找到你的JDK的bin目录 D:\develop\Java\jdk-21\bin 复制一下

**第二步：**将JDK的bin目录粘贴在Path环境变量后面

**第三步：检测否配置成功**

1.按住Win+R输入cmd 回车，打开命令行创建
2.输入javac -version 看提示信息是否显示你安装JDK的版本号
  输入java -version 看提示信息是否显示你安装JDK的版本号
【如果显示版本号都是JDK21就表示配置安装成功】

你如果按照前面的操作到这里，就说明JDK环境变量已经配置好了，后面使用JDK命令可以在任意目录下运行。

#### 配置JAVA_HOME环境变量

**JAVA_HOME：是用于**告诉操作系统JDK安装在了哪个位置（将来其他技术要通过这个环境变量找JDK）。

![Pjetb1sOqownC9xzTDMcn68tnec](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Pjetb1sOqownC9xzTDMcn68tnec.png)

配置JAVA_HOME环境变量：点击新建，添加JAVA_HOME环境变量

![Sib1bdHGLoyc86x4tI2cds1inub](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Sib1bdHGLoyc86x4tI2cds1inub.png)

![DZs2bAIlXoTvf1x7J6OcXwtBnQe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/DZs2bAIlXoTvf1x7J6OcXwtBnQe.png)

一旦配置了JAVA_HOME 了 ，就不建议JDK的PATH配置成如下完整路径的形式了

![KxDkbwUzVof3FgxBNvac8dQnnkb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/KxDkbwUzVof3FgxBNvac8dQnnkb.png)

而建议配置成如下形式，这样的好处是JDK升级时，只需要改一下JAVA_HOME即可，Path下面的路径会自动去找JAVA_HOME环境变量配置的对应JDK路径下bin包里的javac和java程序，这样以后即便升级了JDK也只需要改JAVA_HOME，而不需要维护JDK的Path环境变量了，非常的方便。

![OByabhdXZokQrmxRWrfcqyq6nGR](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/OByabhdXZokQrmxRWrfcqyq6nGR.png)

最终的PATH环境变量配置效果如下：

![Gyz9bqcuhoIUWdxhClSck5xJnCc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Gyz9bqcuhoIUWdxhClSck5xJnCc.png)

### 开发Java入门程序

#### 开发第一个java程序

接下来，我们就正式开始开发第一个入门Java程序。按照国际惯例，学习任何一门编程语言第一个案例都叫做 **Hello World**，意思是向世界问好，从此开用程序和世界沟通的大门。

编写一个Java程序需要经过3个步骤：**编写代码，编译代码，运行代码**

-  编写代码：第一次写代码。就选择用记事本写代码
-  编译代码：将人能看懂的源代码（.java文件）转换为Java虚拟机能够执行的字节码文件（.class文件）
-  运行代码：将字节码文件交给Java虚拟机执行

![KLFmbXGwMoIlEqxevlAcbZHKnmg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/KLFmbXGwMoIlEqxevlAcbZHKnmg.png)

#### 编写、编译、执行代码

按照下面提供的步骤，一步一步的完成第一个Java入门程序的编写、编译和执行。

**第一步**：新建一个后缀为.java的文本文件`HelloWorld.java`，用记事本编写代码如下。

```Java
public class HelloWorld {
   public static void main(String[] args) {
     System.out.println(" HelloWorld ");
    }
}
```

**第二步**：进入`HelloWorld.java`文件所在目录，在地址栏输入cmd回车，即可在此处打开命令行窗口。

编译：在命令行窗口输入编译命令`javac HelloWorld`.java 完成编译，编译后会生成一个`HelloWorld.class`文件。

![TjxsbWsPwom8MIxbQwgcRWcVnwh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/TjxsbWsPwom8MIxbQwgcRWcVnwh.png)

**第三步**：再接着输入`java HelloWorld`就可以运行了，运行结果如下。

![XmXJbpQYwohx2BxqP4FcI3xFnPd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/XmXJbpQYwohx2BxqP4FcI3xFnPd.png)

#### Java程序的跨平台

Java程序可以实现一次编译，处处可用，这就是Java的跨平台，这样的好处是，只需要招一批程序员开发一套Java程序就可以部署到很多平台执行，这样可以提升开发效率，降低开发成本。

**为啥可以跨平台？**

因为Java程序运行在JVM虚拟机中，而每个平台都可以安装一个属于自己的JVM虚拟机，我们的Java程序编译后都是运行在JVM虚拟机里的，因此JVM虚拟机是Java程序跨平台的根本原因，如下图所示：

![OXEnbryAHoZaOsx8RBycisMznUe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/OXEnbryAHoZaOsx8RBycisMznUe.png)

#### cmd常见命令

前面测试JDK是否安装成功，需要在黑窗口中输入\`javac -version\`和\`java -version\` 这其实就是JDK查看编译工具和运行工具版本号的命令。

这种输入命令的和电脑交互的方式，称之为命令行交互。也就是说，可以使用命令指挥电脑做事情。接下来我们了解几种Windows系统常见的命令，后面可能会用到。

下面是Windows系统常见的命令以及作用，小伙伴们可以自己试一试。需要注意的是，每敲完一条命令之后，马上敲回车就表示执行这条命名。

```Java
E:  //切换到E盘
cd [目录]        //进入指定的目录
cd ..         //退回到上一级目录
cd /         //退回到根目录
dir             //显示当前目录下所有的内容
cls             //清空屏幕
```

#### Java程序中常见的问题

刚才小伙伴们在编写第一个HelloWorld程序的时候，是不是很容易报错啊？ 我观察过第一次写代码，90%的同学都会有些小问题的，比如单词写错了！ 括号少写一个！等等！  我想给大家说的是，写错代码都是很正常的，**一个什么错都犯过的程序员，才是真正的程序员**。

以下是入门时常见的问题，需要大家注意：

-  Windows的文件扩展名没有勾选
-  代码写了，但是忘记保存了
-  文件名和类名不一致。
-  英文大小写错误，单词拼写错误，存在中文符号，找不到main方法。
-  括号不配对。
-  编译或执行工具使用不当。

以上问题的详细说明如下：

- 文件扩展名没有打开

下图中文件扩展名的勾勾没有勾选，就会导致你创建的文件是普通的文本文件（.txt）文件，而不是java文件。

![ZpOYbyOjNoHIrCxVYCDcwmGMnqf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ZpOYbyOjNoHIrCxVYCDcwmGMnqf.png)

正确做法是把文件扩展名的勾选上。

- 文件名和类名不一致

你看下图中，文件名是`HelloWorld`，但是类名是`Helloworld`看出区别了吗？一个是大写的W，一个是小写的w。 不仔细看还真看不出来。

**正确写法是文件名叫`HelloWorld`，类名也叫\`HelloWorld**\`

![MwvMbtKkNoHBbjxj6vIcDPU5nTc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/MwvMbtKkNoHBbjxj6vIcDPU5nTc.png)

- 单词大小写错误

下图中不是string和system这两个单词都写错了， 这里是严格区分大小写的

**正确写法是String和System**

![CJqybYXeoo1QemxgwFacn4mInhd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/CJqybYXeoo1QemxgwFacn4mInhd.png)

- 主方法写错了

下图所示，主方法的名称写成了`mian`，这是错误的。

主方法正确写法：必须是`public static void main(String[] args){}`，一个字母都不能错。

![GQS4bGd6moqxwGxzlRLcKSLhnbc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/GQS4bGd6moqxwGxzlRLcKSLhnbc.png)

- 标点符号写错了

下图中打印语句最后的分号，写成了中文分号`；`

**正确写法应该是英文分号** `;`  不仔细看还真看不出区别，要小心

![PkZobHbreoFW7lx60XkcOqAgn1d](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/PkZobHbreoFW7lx60XkcOqAgn1d.png)

## Java编程工具的使用

### IntelliJ IDEA的下载、安装

#### IDE集成开发环境

工欲善其事必先利其器，作为一名零基础的Java初学者，了解企业中常用的Java编程工具至关重要。

企业里不会使用记事本来写代码，记事本没有语法提示、错误提示、没有项目结构，开发起来很不方便。实际开发时都会使用**IDE（Integrated Development Environment）集成开发环境。**IDE是集成了JDK、代码编写窗口、编译、执行等多种功能的开发工具。

常见的IDE工具如下：

![AhsXbBCBvoNi0lxQpq3c8N4gncd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/AhsXbBCBvoNi0lxQpq3c8N4gncd.png)

目前Java开发，企业里用的最多的开发工具是: **IntelliJ IDEA**

- 简称IDEA，在代码提示、智能补全等多方面表现的都非常优秀，是进行Java开发时首选的企业级开发工具。

#### IDEA的下载和安装

要安装idea，可以直接去IDEA官网下载**：** [**https://www.jetbrains.com/zh-cn/idea/download**](https://www.jetbrains.com/zh-cn/idea/download)

> **注意：下载对应操作系统的安装包，傻瓜式安装(建议修改安装路径，不带中文)**

![I8j9bIKdrobMhMxT1i5cXBzjnPf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/I8j9bIKdrobMhMxT1i5cXBzjnPf.png)

**安装步骤如下：**

![JXPRbOQetoBSVsxjetUckKI0n6e](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/JXPRbOQetoBSVsxjetUckKI0n6e.png)

![GTQ5bmvK6op7I2xixP5c835en7d](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/GTQ5bmvK6op7I2xixP5c835en7d.png)
![XzVrbmR8NotHZfxxqwxc2WvAnKg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/XzVrbmR8NotHZfxxqwxc2WvAnKg.png)

![XzKnblWptoMxRqxKm6YclGaDnQg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/XzKnblWptoMxRqxKm6YclGaDnQg.png)
![Qs1vbJ2qnoy7Jrx8BWYcGQ6xnLc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Qs1vbJ2qnoy7Jrx8BWYcGQ6xnLc.png)

![SUZ6bZ8JqowPbYxYJcWcv4wTncg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/SUZ6bZ8JqowPbYxYJcWcv4wTncg.png)
![YgdKbgB4MoWXyXx8ZAFcqYQBn2b](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/YgdKbgB4MoWXyXx8ZAFcqYQBn2b.png)

![MNr2bvc6LoFPdNxClCRcde7wnee](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/MNr2bvc6LoFPdNxClCRcde7wnee.png)
![VpONb4sCJoZR9AxQDt2c1PP1nqh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/VpONb4sCJoZR9AxQDt2c1PP1nqh.png)

### 使用IDEA开发第一个Java程序

#### 认识IDEA工程结构

想要在IDEA正确的写一个Java程序，必须先认识一下IDEA的管理Java程序的工程结构。

- 第一步：首先得在IDEA中创建一个Project（工程、也叫项目），后面统称为工程，其实是用一个文件夹代表
- 第二步：需要在Project中创建Module（模块），一个工程中可以包含多个模块，其实是用一个文件夹代表
- 第三步：需要在Module中新建Package（包），一个模块中可以有多个包，其实是用一个文件夹代表
- 第四步：需要在Package中新建Class（类），一个包中可以包含多个类，其实是一个.java结尾的文件

软件工程其实类似于建筑工程，我们对比建筑工程来理解。

- Project（工程）：你可以理解成小区的院子
- Module（模块）：你可以理解成小区院子里面的每一栋楼
- Package（包）：你可以理解成每一栋楼的一层
- Class（类）：你可以理解成每一层的住户

这样划分的目的是为了更好的管理我们的系统程序。

![ZPakb2aSkoIEpNx1L5XcUmMqn9e](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ZPakb2aSkoIEpNx1L5XcUmMqn9e.png)

在实际开发中，比如淘宝网站这样的工程，但是由于功能很多，所以就把淘宝网站分为不同的模块，首页是一个模块、购物车是一个模块、订单也是一个模块；

![CI4UbrTX1oDfs1x3oBPcy15TnIg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/CI4UbrTX1oDfs1x3oBPcy15TnIg.png)

我们可以认为整个基础阶段是一个工程，每天课程是一个模块。接下来我们来建工程和模块演示一下如何开发一个Java程序并执行。

#### 新建工程new Project

首次启动，新建工程的界面如下（注意：工程本质是一个文件夹路径）

![TbVpbM6wToacqvx9JQrcnVPZnLb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/TbVpbM6wToacqvx9JQrcnVPZnLb.png)
![O5GSblHWNokJIxxVGrPcIJ7tnIf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/O5GSblHWNokJIxxVGrPcIJ7tnIf.png)

![LE2ObSnxporrm3xyTQAcNfWPnec](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/LE2ObSnxporrm3xyTQAcNfWPnec.png)

#### 新建模块new Module

选择当前工程，为今天内容新建一个模块（注意：模块本质也是一个文件夹路径），具体如下

![Kza8bPqbuoYP3Jx1nuecq8dYn7c](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Kza8bPqbuoYP3Jx1nuecq8dYn7c.png)

![IRlhbxADyoNUggxpjEkcepednKf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/IRlhbxADyoNUggxpjEkcepednKf.png)

![YXRJbnLYxotiGJxA5GCcKG2Jnzg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/YXRJbnLYxotiGJxA5GCcKG2Jnzg.png)

模块创建出来后，模块下的src为源代码路径，也就是写代码的地方，建议大家删除下面的Main程序，一会自己建，不用默认的。

![I6O5bUYQAobJ14xXkLbcGiajnze](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/I6O5bUYQAobJ14xXkLbcGiajnze.png)

#### 新建包new Package

一般还会在模块下创建很多包（注意：包本质也是一个文件夹路径），以便分门别类的管理各种不同的程序代码，建包操作如下：

![FzSVbWIxnoMib0xe3OpctuJQnvc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/FzSVbWIxnoMib0xe3OpctuJQnvc.png)

![MjrTb3QNMoJUUdxFIiOciRJanPf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/MjrTb3QNMoJUUdxFIiOciRJanPf.png)

输入完包名后，按回车键

![V050buEm4ohXoqxR6ZZcka8Gnhf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/V050buEm4ohXoqxR6ZZcka8Gnhf.png)

#### 新建类new Class

创建包后，接下来是新建类文件，也就是.java结尾的程序文件，这个文件是我们写java代码的地方。通常需要将类文件创建在包中。具体创建步骤如下：

![R7NTbJKeDoDmqex4vNhcC1qqnOh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/R7NTbJKeDoDmqex4vNhcC1qqnOh.png)

![VKygbwp5NoDkdqxxIYncTdiinZd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/VKygbwp5NoDkdqxxIYncTdiinZd.png)

![Z9yKbdX4joVb7txk0ZwcMig2nGk](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Z9yKbdX4joVb7txk0ZwcMig2nGk.png)

#### 编写、启动程序

直接在HelloWorld类文件中，输入main就可以提示出main方法，直接输入sout就可以提示出输出语句，执行程序直接右键，就可以选择执行，具体如下:

![Tc8PbNrcfol8Szxx4g9cDkcinwd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Tc8PbNrcfol8Szxx4g9cDkcinwd.png)

![Z0Ribd0WGo1CfGxtsvYcHyson5g](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Z0Ribd0WGo1CfGxtsvYcHyson5g.png)

有一个需要注意的细节：idea也会把程序自动编译成class文件，编译后的class文件在模块下的一个out目录文件下，大家去查看一下

![CufPbs9SIo6mjOx8NCRcQRWqneb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/CufPbs9SIo6mjOx8NCRcQRWqneb.png)

### IDEA安装AI编程插件[重点]

#### 概述和安装

目前在IDEA中写程序，程序的编码工作，还是需要程序员思考，并逐行编写。现在已经是AI时代了，AI出现后，IDEA可以集成很多辅助编程的AI编程插件，可以帮助我们自动生成代码，从而提高我们的生产效率**。**

常见的AI插件如下：

![K2BWbhZ34o1JTCx2knMcX3BTnXg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/K2BWbhZ34o1JTCx2knMcX3BTnXg.png)

我们教学环境选择集成阿里巴巴的通义灵码AI插件，是很好用，很强大的。阿里巴巴的通义灵码学习了很多阿里的Java代码，市面上很多程序员都在使用。

在idea中使用通义灵码，需要安装插件，具体步骤如下：

1. 必须先联网，否则安装失败，以后用AI插件时也都必须联网。
2. 搜索并安装插件

![XzDgbdhByoTkhFxUdf2cBpdonLh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/XzDgbdhByoTkhFxUdf2cBpdonLh.png)

![IJ8fbd0Y5ow2GYxXJbucU9AxnKd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/IJ8fbd0Y5ow2GYxXJbucU9AxnKd.png)

![OSXFbuAslooLUSxaoYUcATTanAp](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/OSXFbuAslooLUSxaoYUcATTanAp.png)

至此，你的IDEA已经很强大了，已经集成了AI插件：通义灵码了。

#### 通义灵码的使用

通义灵码插件使用其实很简单，主要有以下两种方式：

1. 写代码时，它会提示代码怎么写，你按一下Tab键，代码就会自动写出来。
2. 写注释后，按回车键后，通义灵码也会自动写好代码，然后你按一下Tab键，代码就会自动写出来。

![GvERblR98o1IpAxwj1JcNFZNnRg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/GvERblR98o1IpAxwj1JcNFZNnRg.png)

AI的功能，还很强大，接下来的课程中，每天都会演示其用法，大家记得坚持学习，不要放弃哦！

### IDEA的设置：字体、背景色、快捷键

#### 字体、主题、背景色

刚才有同学在使用IDEA编写程序时，可能会觉得字体比较小，背景色是黑色的，注释是灰色的，看不清，长时间可能对眼睛不好。我们可以通过IDEA相关的设置，把字体调大一点，背景色调为护眼色，注释也调为绿色。

-  **主题配置**

![Sh9xbXNmdo4qKfxopThcvhVpnKd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Sh9xbXNmdo4qKfxopThcvhVpnKd.png)

- **字体配置**

![FIVsb7bC9o5N13xxEwcc79wanCc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/FIVsb7bC9o5N13xxEwcc79wanCc.png)

- **背景色配置**

可以把背景色的颜色值，调为204、238、200；就是豆沙绿的护眼色了

![DsJFbdYtNoq2bXxpwEkcT1zznge](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/DsJFbdYtNoq2bXxpwEkcT1zznge.png)

#### 常用快捷键

所谓快捷键就是通过键盘上的按键组合起来，可以快速实现我们想要的功能，使用快捷键可以大大提高我们的开发效率。

下面是几种常见的快捷键，以及他们的作用，大家可以自己试试

![EguxbobavoKo8hx7gy9ckx9Jnnf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/EguxbobavoKo8hx7gy9ckx9Jnnf.png)

快捷键其实有很多，这里列举的是现阶段我们用得比较多的，现在记住不也不要紧，以后经常用，用着用着就记住了。在后面的课程中讲到一些新的知识点时，还有会继续给大家讲一些新的快捷键。

### IDEA的其他常用操作

idea中除了新建工程、模块之外，还有一些较常用的操作也需要我们掌握：

- 打开、关闭工程
- 删除类文件
- 修改类名称
- 修改模块
- 导入模块
- 删除模块

接下来，给大家分别演示一下以下操作步骤。

- 打开工程

![OxWZbFZHrokWcIx9XpUcjyuWnoe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/OxWZbFZHrokWcIx9XpUcjyuWnoe.png)

![UMs0bDjn5oaap6x4h5bcEOwHneb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/UMs0bDjn5oaap6x4h5bcEOwHneb.png)

- 关闭工程

![TGX1bY3mHo6kKNxhKYhc4YhWnOc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/TGX1bY3mHo6kKNxhKYhc4YhWnOc.png)

- 删除类文件

方法一：右击要删除的类，然后选择“Delete..."即可删除。

方法二：选中要删除的类，然后按键盘的 delete键。

![S8mqbEJWtoKALFxZ9qocpsdknzi](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/S8mqbEJWtoKALFxZ9qocpsdknzi.png)

- 修改类名称

![LA06b9kxCoYlqexV7vNcqixJnCb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/LA06b9kxCoYlqexV7vNcqixJnCb.png)

- 修改模块

![MmyGb9hWdoK2UjxvLQRcAEvonnc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/MmyGb9hWdoK2UjxvLQRcAEvonnc.png)

![QnWgbrftYo2yf2xcPImcVnV5nzh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/QnWgbrftYo2yf2xcPImcVnV5nzh.png)

修改模块时，会弹出以上3个选择，分别代表如下含义：

- Rename directory

这个选项只会重命名文件系统中的目录（文件夹）。它不会影响 IDEA 中的模块名称。

- Rename module

选择这个选项会重命名 IDEA 中的模块名称，但不会更改文件系统中的目录名称

- Rename module and directory

这个选项同时重命名模块和其对应的目录，同时改变两者，保持一致性，**提倡用这个选项**。

- 导入模块

![A6OzbrEdsoFZbYxgW8ncwwOpnud](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/A6OzbrEdsoFZbYxgW8ncwwOpnud.png)

- 删除模块

右击模块名，然后选择”Remove Module“，或者直接按键盘的 Delete 键进行删除。

![B7ltbOcOXoaExLx6qpDcZ32gnLd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/B7ltbOcOXoaExLx6qpDcZ32gnLd.png)

## Java程序的基本语法

想要编写Java代码，就必须学习Java的语法，学习语法重点需要做到下面两点：

- 记住语法格式
- 明确这种语法格式能达到什么效果

语法格式是Java语言的设计者规定好的，我们不用关心它为什么这么写，只须记住书写规则即可。

### 注释

<callout emoji="🌷">
注释是解释说明程序的问题，方便自己和别人阅读代码，对程序的功能没有任何影响。
</callout>

Java中的注释有三种格式：

1. 单行注释
2. 多行注释
3. 文档注释

三种注释书写格式示例如下：

```Java
package com.heima;

public class Demo01 {
    public static void main(String[] args) {
        //这是一个单行注释: 下面代码的功能:在屏幕上输出helloworld
        System.out.println("HelloWorld");

       /*
        这是一个多行注释
        这是一个多行注释
        这是一个多行注释
        */
        System.out.println("HelloWorld");

        /**
         * 这是一个文档注释
         * 这是一个文档注释
         * 这是一个文档注释
         * 这是一个文档注释
         * 这是一个文档注释
         */
        System.out.println("HelloWorld");
    }
}
```

<callout emoji="💡">
多学一招
如果想快速将代码变为注释，可以使用以下快捷键：
- Ctrl + /    单行注释（对当前行进行注释）
- Ctrl + Shift + /    对选中的代码进行多行注释。
</callout>

### 字面量

学习完注释之后，我们来学习一个全新的知识点：字面量。

<callout emoji="🌷">
计算机是来处理数据的，**字面量其实就是告诉程序员数据在程序中的书写格式**。
</callout>

常见的字面量有: **整数,小数,字符,字符串, 布尔值**, **空值**

<sheet sheet-id="pRysBV" token="YWDmsAJt7hPQh2tya1TcwYoWnNf"></sheet>

下面是常用的数据在程序中的书写格式：

```Java
/*
目标：需要同学们掌握常见数据在程序中的书写格式
*/
public class LiteralDemo{
    public static void main(String[] args){
        //1.整数
        System.out.println(666);

        //2.小数
        System.out.println(3.66);

        //3.字符: 字符必须用单引号引起来
        System.out.println('a');
        System.out.println('0');
        System.out.println('中');
        System.out.println(' '); //空格也算字符

        //特殊字符：\t表示制表符 \n表示换行
        System.out.println('\t'); //这相当于一个tab键，专业叫做制表符
        System.out.println('\n'); //这是换行的意思

        //4.字符串：字符串是双引号引起来的
        System.out.println("我爱你中国abc");

        //5.布尔值：只有两个值true和false
        System.out.println(true);
        System.out.println(false);
    }
}
```

### 变量

#### 3.3.1 认识变量

<callout emoji="🌷">
**变量是用来记录程序中的数据的。其本质上是内存中的一块区域**
**定义格式:**
**数据类型 变量名 =  数据;**
</callout>

你可以把这块区域理解成一个小盒子。我们通过先通过一段代码演示一下，并解释变量的含义：

```Java
int age = 18;
System.out.println(a);
```

当执行`int age = 18;` 这句代码时，JVM会在内存中申请一块区域，在这个区域中存储了一个整数18，给这个区域取的名字叫age； 相当于在盒子中存了一个数据18，这个盒子的名字是age，当我们打印age时，就是从盒子中把盒子中的数据取出来再打印。

![W4hdbcvLMov8eExidgmcFZZ2n0I](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/W4hdbcvLMov8eExidgmcFZZ2n0I.png)

**为什么要用变量呢？**

使用变量来记录数据，对于数据的管理更为灵活。

比如：有一个程序在多个地方用到整数10; 现在我想把10改为20，这时如果不使用变量，就得一条语句一条语句的修改：

```Java
System.out.println(10);
System.out.println(10);
System.out.println(10);
System.out.println(10);
```

为了减少工作量，可以先用一个变量记录这个整数10，然后在需要用到整数10的地方，用x代替就行：

```Java
// 将10存储到名称为x的变量中
int x = 10;

//如果我们把x值改了，那么后面用到x的值的地方就都一起改变了
x=20
System.out.println(x);
System.out.println(x);
System.out.println(x);
System.out.println(x);
```

除了以上通过变量使用数据方便之外，变量中的数据还是可以替换的（可变的），如果在一些场景中数据需要频繁变化，就必须使用变量。

- **变量有应用场景**

变量的应用场景无处不在，只要是程序中能发生变化的数据，都可以用变量存储。比如：你微信钱包中的余额，你微信的昵称，你微信的个性签名； 余额随着你花钱是不是在变少，昵称和个性签名也可以经常修改。

示例1：微信余额

```Java
//比如：你微信余额目前是188.8元，一会要发100元，再收20元。
double money = 188.8;
//发100元
money = money - 100;
//再收20元
money = money + 20;
//再打印money记录的值是多少
System.out.println(money);
```

示例2：公交车人数记录

```Java
//公交车：大人2元，小孩1元，公交车经过2站
//第一站：3个大人1个小孩上车
//第二站：1个大人1个小孩上车，请问一共上了多少人，一共收入多少钱？

//刚开始公交车箱子里没有钱，用money2表示箱子里的钱
int money2 = 0;

//刚开始公交车上也没有人，用count变量表示上车的人数
int count = 0;

//经过两站的人数：第一站3人，第二站1人，总的人数是3+1
count = 3 + 1 + 1 + 1;

//经过两站的钱数：
money2 = money2 + 3*2 + 1*1; //经过第一站后
money2 = money2 + 1*2 + 1*1; //经过第二站后

//打印人数和钱数
System.out.println(count);
System.out.println(money);
```

#### 数据类型

Java 的数据类型用于定义变量可以存储的数据种类，主要分为两大类：**基本数据类型（Primitive Types）和引用数据类型（Reference Types）**。

1. **基本数据类型**

基本数据类型是 Java 中最简单的数据类型，直接存储值。Java 提供了 8 种基本数据类型：

![GlA9b8Owto5PuBxYnOJcnmu0n6f](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/GlA9b8Owto5PuBxYnOJcnmu0n6f.png)

<callout emoji="💡">
整数、小数字面量，他们默认什么类型？
23 ，默认是int类型 , 加上L/l就是long类型的数据了。
23.8，默认是double类型，加上F/f就是float类型了。
</callout>

1. **引用数据类型**

这部分内容引用数据类型用于存储对象的引用，包括但不限于以下类型：

- **类（Class）**：用户定义的类实例。
- **接口（Interface）**：接口引用类型。
- **数组（Array）**：一种特殊类型的对象，用于存储相同类型的元素集合。

引用数据类型在后续章节中讲解（讲解面向对象知识点之后），以上简单了解即可。

数据类型示例：

```Java
//byte 字节类型
byte x1 = 100;

//short短整数
short x2 = 30000;

//int 整数
int x3 = 1000;

//long 长整数 , 末尾要加 L 或 l
long x4 = 200000000088880000L;

//float 单精度浮点数, 末尾要加 F 或 f
float x5 = 100.55F;

//double 双精度浮点数
double x6 = 100.55;

//char 字符类型 鰹
char x7 = 'A';
char x8 = 65;
char x9 = 29993;
System.out.println(x7);
System.out.println(x8);
System.out.println(x9);

//boolean 布尔类型
boolean x10 = true;
```

#### 变量的注意事项

在上节课已经给大家讲了变量的基本使用，变量在实际开发中用得特别多，关于变量使用注意事项需要给大家交代一下。

讲这些注意事项目的是让大家知道，有哪些写法是允许，有哪些写法是不允许的，能分辨对错，并知道为什么错。

1.变量定义在哪个{}范围内，就只在哪个大括号内有效。变量的有效范围称之为变量的作用域

```Java
    {
        int a = 10;
        System.out.println(a); //这是是对的
    }

    System.out.println(a); //这里会出错
```

2.在同一个作用域内，不能有两个同名的变量

```Java
    {
        int a = 10;
        int a = 20; //这里会出错
    }
```

3.变量没有初始化只，不能直接使用

```Java
    int a; //仅仅定义了变量，但是没有初始值
    System.out.println(a); //这里会出错
```

4.变量可以定义在同一行

```Java
  int a=10, b=20; //a和b都是int类型
```

变量小结

- 变量是用来记录程序中的数据的，可以把变量理解成内存中的小盒子，盒子里放的东西就是变量记录的数据
- 变量的定义格式： `数据类型 变量名 = 初始值;`
- 变量记录的数据程序运行过程中是可以发生改变的：`变量名 = 值;`

#### 变量里数据的存储原理

计算机数据存储的原理主要涉及如何以不同的进制表示和存储信息。进制是数字系统中用于表示数值的基数。不同的进制系统使用不同数量的符号来表示数字。

##### 二进制

二进制 (Base 2)使用 0 和 1 两个符号，按照逢2进1的方式表示数据。计算机内部使用二进制，因为它与电路的开关状态（开/关）相对应。

计算机中的数据（如文本、图像、音频等）都是以二进制形式存储的：

- 文本：美国信息交换标准编码（ASCII编码表），规定了英语、数字字符、和其他西欧字符对应的数字编号。

![SD69bMKYVolrCPxqWvHcXikWnWg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/SD69bMKYVolrCPxqWvHcXikWnWg.png)

- 图片：就是无数个像素点组成的，每个像素点的数据：用0 \~ 255\*255\*255表示其颜色

![KuQsbQQDho9bzCxeA8ZcDudJnVd](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/KuQsbQQDho9bzCxeA8ZcDudJnVd.png)

![SCRWbkzksocsq0xtlAoc9ru7nsl](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/SCRWbkzksocsq0xtlAoc9ru7nsl.png)

- 声音：也可以被编码成二进制存储

![B5LSbOVVHoiGhzxNQuLcvmLPnGf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/B5LSbOVVHoiGhzxNQuLcvmLPnGf.png)

![AmYZbk1vmo3LzDxG86wc3WM2nUf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/AmYZbk1vmo3LzDxG86wc3WM2nUf.png)

<callout emoji="💡">
多学一招
Java程序中支持书写二进制数据，需要以0B或者0b开头。
</callout>

##### 八进制、十六进制

- **八进制 (Base 8)**：使用 0 到 7，共八个符号。常用于某些计算机系统中，便于转换为二进制。
- **十六进制 (Base 16)**：使用 0 到 9 和 A 到 F（其中 A 表示 10，B 表示 11，依此类推）共十六个符号。通常用于表示计算机中的字节和内存地址。

<callout emoji="💡">
多学一招
Java程序中支持书写八进制数据，需要以0开头。
Java程序中支持书写十六进制的数据，需要以0X（或者0x）开头。
</callout>

##### 进制转换

（1）**二进制到十进制**

将每个位的数字乘以 2 的相应幂次，然后求和

![IitpblxNxo3J2IxTdZzcubyGnMc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/IitpblxNxo3J2IxTdZzcubyGnMc.png)

（2）**十进制到二进制**

除2取余法：不断除以 2，记录余数，最后将余数反向排列

![Pwlmb5go1oDaEFxF6zmcbwzLnUh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/Pwlmb5go1oDaEFxF6zmcbwzLnUh.png)

（3）**二进制到八进制**

每3位二进制作为一个单元，最小数是0，最大数是7，0-7有8个数字。

![GE2Pblqt1oNHzcxkv96cxhNenod](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/GE2Pblqt1oNHzcxkv96cxhNenod.png)

（4）**二进制到十六进制**

每4位二进制作为一个单元，最小数是0，最大数是15；

0-15有16个数字，依次用： 0\~9 A B C D E F  代表。

![H2SLbnaETo9eM6xe5a0cPmuCnp7](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/H2SLbnaETo9eM6xe5a0cPmuCnp7.png)

### 关键字、标识符

学完变量之后，我们再来认识一下Java的关键字和标识符。

#### 关键字

关键字是java语言中有特殊含义的单词。比如用int表示整数，用double表示小数，等等！

Java中一共有50多个关键字（如下图所示），这些关键字我们不需要强制记住，以后随着大家积累的代码量越来越多，自然就能记住这些关键字。

![RqR7bQjDKoB5zKx0J7YcGTHinve](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/RqR7bQjDKoB5zKx0J7YcGTHinve.png)

- **如何识别那些单词是关键字？**

现在我们不用知道这些关键字是什么意思，主要能够根据特点识别那些是关键字就可以了

关键字的特点：

    1.关键字都是小写的

    2.关键字在idea中有特殊颜色标记（在idea中默认是蓝色的）

#### 3.4.2 标识符

所谓标志符其实就是我们自己给数据取的名字。像前面我们取的类名，变量名其实都是标志符。

我们在给数据命名时，是要遵循一定规则的，如果不遵守就会报错。标识符的规则如下：

- 标识符可以包含字母（大小写均可）、数字、下划线（`_`）和美元符号（`$`）
- 标识符不能以数字开头
- Java 是大小写敏感的语言，因此 `myVariable` 和 `myvariable` 被视为两个不同的标识符
- 不能使用 Java 的关键字（如 `int`、`class`、`if` 等）作为标识符。
- 标识符的长度是无限制的，但为了可读性和维护性，一般不宜过长

标签符的命名约定：在java开发领域，标识符有约定俗成的规定，按照这些规定命名会显得更加专业。

- 类名：每个单词的首字母大写（驼峰命名法），例如 MyClass。
- 方法名和变量名：第一个单词小写，其余单词首字母大写（驼峰命名法），示例 ：

```Java
double money = 6.88;
double applePrice = 7.5;
```

- 所有的名字要见名知意，便于自己和别人阅读

```Java
 class Student{} //一看这个类就知道表示一个学生

int age =10;    //一看这个变量就知道表示年龄
```

## 相关笔记

- [[Memo/development/programming/java/java-syntax-and-control-flow/01-getting-started.md|01 入门·笔记]]
- [[Memo/development/programming/java/java-syntax-and-control-flow/05-variables-data-types-and-conversion.md|变量、数据类型与类型转换精讲]]

---

> 来源：飞书《讲义》知识库 · day01-入门 · [原文](https://baxsu7o100e.feishu.cn/wiki/IJ6pwAYFQixdlKk2i8OcC2Nxnng)
