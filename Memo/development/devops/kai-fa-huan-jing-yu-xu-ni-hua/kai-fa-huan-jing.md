---
type: memo
date: 2026-11-06
aliases:
  - "开发环境"
---
# 开发环境

工欲善其事必先利其器。

1. 个人电脑内存至少16G以上（为了更好的学习最好升级到32G）不符合条件的同学趁休息日升级电脑。
2. 有条件的购买便携显示器进行分屏开发，开发效率杠杠滴，这是专业程序员的标配。

<callout emoji="⚽">
**先下载资料！！！先下载资料！！！先下载资料！！！**
链接: https://pan.baidu.com/s/14hBDPAoYFDFZ5qXoSHesHA 提取码: 8tgq 
--来自百度网盘超级会员v5的分享
</callout>

## 1 导入虚拟机

黑马商成项目中用到的一些服务端软件如：Nacos配置中心、Redis、RabbitMQ消息队列等通常会安装在企业局域网的服务器中，开发人员去远程连接它们。在教学中我们在自己的电脑上安装虚拟机，虚拟机代表了企业局域网中的服务器。

服务器操作系统使用Centos7，导入下发的镜像文件。

### 1.1 **导入虚拟机**

首先安装VMware-workstation 16.x 或 VMware-workstation 17.x 虚拟机软件（**已安装vmware软件的同学不需要重新安装**），可从课程资料获取安装程序（VMwareworkstation16.rar）。

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/O6PBbNgdBoRybaxuyHNcOxFTntb.png)



**1）设置网络**

点击 “编辑--》虚拟网络编辑器”配置网络地址，地址须与下图一致。

设置子网IP：192.168.101.0，子网掩码：255.255.255.0。

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/RjC3bst1YoMSJ9xU4qfcKN5jnVc.png)

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/P1EMbT2WzoXoEUxjPVCceegFn1e.png)

确认自己的虚拟网卡VMnet8的IP地址和网关如下图所示

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/X66abCnUHoOaQOxKiHwcHKROn7f.png)

> **自己电脑的防火墙必须处于如下关闭状态！！！**
> 
> **自己电脑的防火墙必须处于如下关闭状态！！！**
> 
> **自己电脑的防火墙必须处于如下关闭状态！！！**

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/MpdsblIJ3oQJPzxqkYscfS55n2c.png)

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/HPKvbNJ7ioBHduxKfdccoP9ynFf.png)

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/Msy6b2zs8o9EPEx5a5ycw4iMn0m.png)

**2）导入虚拟机**

解压老师提供的虚拟机文件，进入解压后的文件架，**双击**"CentOS 7 64 位.vmx" 文件，选择复制虚拟机。

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/TcEmbnO4GoQRCyxTye8cqoe9nvg.png)

对此虚拟机的设置建议至少8G内存、8核CPU，硬盘存储至少40G。

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/DaYNbaqUpoonnRxs4KqcbgxgnCs.png)

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/C4oubQ7BwobB0nxaIIAcleMknyg.png)

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/X9FzbSz7ToGaj7xeoawcEodPnfc.png)

设置完成，启动虚拟机。

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/V6F5blivhom9QfxqsbmcIczan4b.png)

注意：虚拟机的IP地址为192.168.101.68，不用修改IP地址。

虚拟机启动成功如下界面：

账号：root

密码: centos

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/CSBobw2l4oecH2xwHHbcOHPfnNg.png)



#### 1.1.1 **可能遇到的问题**

win11打开.vmx文件报错：“未能启动虚拟机”

尝试解决方法：

1. 打开虚拟机的存放路径
2. 找到.vmx的配置文件
3. 修改 virtualHW.version改为virtualHW.version = "18"



### 1.2 苹果电脑用户（windows电脑同学无需操作）

使用苹果电脑的学生需要自行安装docker并安装MySQL8，已安装MySQL8的不必重新安装。

#### 1.2.1 安装MySQL

拉取镜像：

docker pull mysql:8.0.26

 创建以下目录：

sudo mkdir -p /usr/mysql/conf /usr/mysql/data /var/log/mysql

sudo chmod -R 777 /usr/mysql/ /var/log/mysql

 

创建配置文件

vim /usr/mysql/conf/my.cnf

添加以下内容到上述创建的配置文件中

```Java
 
[client]
#socket = /usr/mysql/mysqld.sock
default-character-set = utf8mb4
 
[mysqld]
#pid-file        = /var/run/mysqld/mysqld.pid
#socket          = /var/run/mysqld/mysqld.sock
#datadir         = /var/lib/mysql
#socket = /usr/mysql/mysqld.sock
#pid-file = /usr/mysql/mysqld.pid
 
datadir  = /usr/mysql/data
 
general_log = 1
general_log_file = /var/log/mysql/access.log
log-error = /var/log/mysql/error.log
 
character_set_server = utf8mb4
collation_server = utf8mb4_bin
secure-file-priv= NULL

Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

Custom config should go here
!includedir /etc/mysql/conf.d/
```

  

创建启动容器

```Java
docker run -itd --name=mysql -v /usr/mysql/conf/my.cnf:/etc/mysql/my.cnf -v /usr/mysql/data:/usr/mysql/data -v /var/log/mysql:/var/log/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=mysql mysql:8.0.26
```

说明： -v : 挂载宿主机目录和 docker容器中的目录，前面是宿主机目录，后面是容器内部目录 -d : 后台运行容器 -p 映射容器端口号和宿主机端口号 -e 环境参数，MYSQL_ROOT_PASSWORD设置root用户的密码

 

下边修改mysql的访问权限为所有客户端可以访问：

进入容器内部

sudo docker exec -it mysql /bin/bash

连接mysql

mysql -uroot -pmysql

使用mysql库

use mysql;

修改访问主机以及密码等，设置为所有主机可访问

```Java
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'mysql';
flush privileges;
```

```Plain Text
docker ps 

```

windows下mysql是否启动

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/By2EbZFQnoobq3xje3Fcmg7Fngg.png)

### 1.3 **远程连接虚拟机**

虚拟机启动成功，下边使用ssh客户端工具（FinalShell）远程 连接 虚拟机。

打开FinalShell软件，没有安装的可以从课程资料“软件工具”目录获取安装程序（finalshell_install.exe）。

通过下图所示建立新连接：

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/EUB5bA1CQotvPSxrwkpckeZtnYf.png)

填写连接信息：

IP地址：192.168.101.68

账号与密码为：root/centos

如下图：

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/RDYPbQL5koALlsxKBzuctX6DnId.png)

点击“确定”，创建连接成功，如下图：

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/JNNjboOnKo7rZYxgvLhcknf3nPm.png)

双击连接信息，连接成功如下图：



执行 **systemctl start docker 启动docker。**

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/V4f1b0bAxogX6Rxt59acsTarnKb.png)

~~启动mysql：docker start mysql~~

以上软件如果已经启动不用重复启动**。**

查询docker容器：docker ps

如下图：

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/XPKebL4c8on8phxTMHYc5hX6nPe.png)



### 1.4 修改IP

```Plain Text
ip addr | grep 192.168.101
```

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/FJj4b7Exzoym6cxKIUOceQuHn0c.png)

如果虚拟机IP不是192.168.101.68需要修改虚拟机IP地址为192.168.101.68（已经是这个IP的无需操作）。

修改方法如下：

确保在root用户下进行操作，进入网络配置文件network-scripts目录下,找到我们需要修改的配置文件:

```Java
cd /etc/sysconfig/network-scripts/
vi ifcfg-ens33
```

使用ls命令，列出该目录下的文件。其中“ifcfg-ens33”的文件，为我们需要修改的网络配置文件。

 修改配置文件

 使用vim命令（vi命令也可以）,对文件进行配置：

1.  我们需要修改BOOTPROTO="static"也就是将dhcp改为static
2. 修改ONBOOT=“yes” 意思是将网卡设置 为开机启用.
3. 修改IPADDR=192.168.101.68
4. 修改GATEWAY=192.168.101.2

ifcfg-ens33文件内容如下：

```Java
[root@localhost network-scripts]# cat ifcfg-ens33
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=28711c7c-1216-454e-a454-b7c9a08e71e6
DEVICE=ens33
ONBOOT=yes
IPV6_PRIVACY=no
IPADDR=192.168.101.68
PREFIX=24
GATEWAY=192.168.101.2
DNS1=8.8.8.8
```



还需要修改vmware的网络配置

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/COIubzAszoMM6cxCPI4cVijsnAg.png)

修改完毕，重启虚拟机即可。



### 1.5 特别注意

使用虚拟机要特别小心，如果没有正常点击虚拟机关闭或者挂起虚拟机，可能会导致网卡文件损坏，因此要每次关闭电脑前，要确保虚拟机正常关闭后，再对电脑关机。咱们每次结束前就尽量选择挂起客户机就行，如下图：

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/UoRfbhR1Io2jVyxgXWBcIvROnmg.png)

下次运行时再点击继续运行此虚拟机即可

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/WtL9bOkAMofrg7xXsD3c20lNnEf.png)



万一出现网卡损坏，连不上虚拟机，那么可以通过之前拍的快照进行恢复；但是如果忘记拍快照，那么就没拍照可恢复就只能重新解压一份虚拟机镜像文件再打开进行拍照了。



拍摄快照和恢复快照方式-确保虚拟机文件正常可用的情况下，找到拍摄入口

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/TQUdb04Gzo0iezx44BEcFYCPnid.png)

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/EbY5bZxdsoVsvoxDrlLcqJvnn7c.png)

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/KYZ6b7RIZoUhvXxVwNYcIR8cnIe.png)

## 2 **IDEA环境配置**

安装IDEA，根据下边的步骤进行配置。如果已经配置过则忽略。

### **2.1 编码配置**

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/UZECbVFP4o36kpxFu1qcKs29nyc.png)

### 2.2 **自动导包设置**

IDEA可以自动优化导入包，**但是有多个同名的类调用不同的包，必须自己手动Alt+Enter设置**，  
下面可以通过设置来进行导包优化。

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/Tag6baLsZoXes7x8A1DccVmLnhh.png)

### 2.3 **提示忽略大小写**

IDEA代码提示默认是区分大小写的，设置为提示忽略大小写，编译我们后期的开发

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/QDNGbIoqFoT45jxn2UXcUO7YnIg.png)

### 2.4 **设置 Java 编译级别**

工程创建成功，点击Project Structure:

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/B0fMb8h4WoEu2mxWuolcqHienzd.png)

点击Project，设置SDK为11及Project language level，如下图：

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/EsIzbMZW0o5OpvxU4Ehc8TQNnQb.png)

## 3 Maven环境

### 3.1 安装Maven

下载maven3.8.6版本，下载链接如下：

https://dlcdn.apache.org/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.zip

解压apache-maven-3.8.6-bin.zip到没有中文的目录下。

**已安装maven的不用重复安装。**

### 3.2 **配置仓库**

配置中央仓库位置：

在setting.xml文件中配置阿里云中央仓库地址。

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/FdzsbMoh2ozotRx1p7TcYA8unrb.png)

### **3.3 IDEA中配置maven**

在IDEA中配置maven：进入 File --> Settings --> Build --> Build Tools --> Maven

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/VdwYb1ZMSoJVEDxpv06c8uZYn9d.png)

配置maven安装目录、setting.xml及本地仓库的位置。

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/KrmDb5MLeoMaQhxa4j0c254jnhg.png)

## 4 配置Git环境



在个人电脑安装Git，使用常用软件工具目录中的Git-2.37.3-64-bit.exe。

也可以自行下载，地址：https://git-scm.com/  （windows版本：https://git-scm.com/download/win）

安装成功，在右键菜单出现Git菜单，如下图

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/EiKYb2w4WoWwUjxPPAKcR97dnuc.png)

配置git邮箱：

```Java
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```



安装成功在IDEA中配置git的安装目录

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/R4r3bDXCJokn8oxvEx9cJCjpncc.png)



## 5 导入商城项目

### 5.1 拉取项目

首先，我们需要熟悉黑马商城项目的基本结构：

本课程提供的git地址为：git clone https://gitee.com/fjlcyl/hm-service.git

大家需要登录自己gitee账号，**Fork该仓库到自己的空间**，Fork是将其它仓库的内容复制到自己仓库。

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/RBusbe35DoIH16xDBbGcLwEpnyb.png)

查看已复制到自己的仓库中的项目，点击“克隆/下载”，复制仓库地址

然后从此Git仓库克隆hmall到代码目录。

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/JfQsbakzmoM5nlxys9ecnjFFnHb.png)

输入远程仓库地址，并选择本地代码目录：

在弹出窗口选择信任该项目，接下来打开克隆成功的工程如下图：

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/AoGbbJ2Noopc4gxpTgAcuHPWnof.png)



### 5.2 连接数据库

首先创建数据库连接

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/DGETb7Zm2oxCraxB75bc7iAFn1c.png)

虚拟机中的MySQL数据库账号为root，密码为：mysql

连接成功数据库和对应表如下：

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/HQl3b5Y42otqcYxqbxVciis8n0b.png)





接下来修改数据库连接参数，数据库的ip地址和数据库密码：在application-dev.yaml中：

```YAML
hm:
  db:
    host: 192.168.101.68 # 修改为你自己的虚拟机IP地址
    pw: mysql # 修改为docker中的MySQL密码
```

### 5.3 启动项目

启动项目：

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/XdpXb53v7oXefYxR82icDS6ynmb.png)

启动项目成功：

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/RLw8bCFAwordkex9fd6crDHVnhd.png)

### 5.4 运行前端

接下来运行前端，前端工程通过nginx运行，找到课程资料下“hmall-nginx”目录将其拷贝到代码目录下（将其复制到一个非中文、不包含特殊字符的目录下）：

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/WEXbbbiEZo62tix2h9Rc17i4nMg.png)

进入hmall-nginx目录后，利用cmd启动，运行start nginx.exe启动nginx：

```PowerShell
# 启动nginx
start nginx.exe
# 停止
nginx.exe -s stop
# 重新加载配置
nginx.exe -s reload
# 重启
nginx.exe -s restart
```



<callout emoji="❗">
特别注意：
nginx.exe 不要双击启动，而是打开cmd窗口，通过命令行启动。停止的时候也一样要是用命令停止。如果启动失败不要重复启动，而是查看logs目录中的error.log日志，查看是否是端口冲突。如果是端口冲突则自行修改端口解决。
</callout>



通过任务管理器可以查看到nginx进程说明nginx启动成功

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/PfOTbCE6ooxU1XxT5ggckykUnsc.png)

接下来可以访问门户：http://localhost:18080/

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/KapzbOUSoodpMfxv30nchhIUnVe.png)

点击“登录”进入登录页面

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/WPrPbnctRobUggxgnuWcPMunnch.png)

输入密码：123

点击“登录”按钮，登录成功

![](../../../../Attachments/development/devops/kai-fa-huan-jing-yu-xu-ni-hua/CcP7bmOQyo0REsxhenVcEGIpnMc.png)

如果登录出现异常信息（mybatis的报错），说明IDEA环境配置中2.4步没设置JDK版本为11，务必按要求做



## 6.问题汇总

<callout emoji="🏆">
如果问题不在下面，那就从头、一步一步的再搭建下环境！！！
如果问题不在下面，那就从头、一步一步的再搭建下环境！！！
如果问题不在下面，那就从头、一步一步的再搭建下环境！！！
</callout>

### 1.docker启动失败

重新解压下镜像文件

### 2.数据库连接失败

mysql是否正常启动

### 3.FinalShell等连接虚拟机失败

检查下网络配置

---
> 来源：飞书技术知识库（脱敏版） · 05 工程化与部署 / 开发环境与虚拟化 / 开发环境 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/LvZNwcFB0iBwUBkTkehcp2Knnwh)
