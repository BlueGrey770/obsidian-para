---
type: memo
date: 2026-11-06
aliases:
  - "vmware-an-zhuang-he-xu-ni-ji-gua-zai"
  - "VMware安装和虚拟机挂载"
---

# VMware安装和虚拟机挂载

#### 安装VMware

![bx3sb0galoetzcxmlytc69ztnmg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/bx3sb0galoetzcxmlytc69ztnmg.png)
> 图注：图片展示的是VMware的安装程序文件，文件名为“VMware-workstation-full-16.1.0-17198959.exe”。该图片位于“安装VMware”步骤的上下文部分，与上下文紧密相关，上下文提到在课程资料中提供了VMware的安装程序，找到安装包（exe文件）后双击下一步安装即可，此图片直观呈现了安装包的文件名称，帮助用户确认找到的安装程序文件是否正确。

在我们的课程资料中提供了VMware的安装程序。找到课程资料中的 Vmware的安装包（exe文件），然后双击下一步下一步的安装即可。

![ladpbm3pxoe5xzxcanyc9hcxnfc](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/ladpbm3pxoe5xzxcanyc9hcxnfc.png)
> 图注：图片展示了电脑文件资源管理器界面，显示了“Linux: Vmware安装Linux问题”文件夹及其中的“Vmware16注册码.txt”和“VMware-workstation-full-16.1.0-17198959.exe”文件。其中，“VMware-workstation-full-16.1.0-17198959.exe”文件被红色框线突出显示。该图片与文档中“安装VMware”部分相关，对应找到课程资料中Vmware安装包（exe文件）的内容，直观呈现了安装包在电脑文件中的位置。

**备注：如果之前安装过，就不用安装了。不要卸载！！！！ 不要卸载！！！！  不要卸载！！！！**



**安装步骤:**

![sm8sb2i8moi1nsxlxy7cbomrnxe](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/sm8sb2i8moi1nsxlxy7cbomrnxe.png)
> 图注：这张图片展示了VMware Workstation Pro 15.5的安装流程界面，共包含三个依次切换的安装步骤窗口。第一个是安装向导的欢迎界面，显示版本为15.5，底部有“下一步(N)”和“取消”按钮；第二个是最终用户许可协议界面，呈现协议内容，底部有“上一步(B)”“下一步(N)”“取消”按钮，且默认勾选了接受协议的选项；第三个是自定义安装界面，高亮标注了指定安装目录为“D:\\develop\\VMware”，同时有“更改”按钮用于调整目录，底部同样有“上一步(B)”“下一步(N)”“取消”按钮。这组图对应文档中VMware安装的操作内容，展示了安装时的关键步骤界面。

![y0gdbduzloxcbsxcrw5c2ovgnsf](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/y0gdbduzloxcbsxcrw5c2ovgnsf.png)
> 图注：这张是VMware Workstation Pro的安装步骤界面图，从左到右依次呈现三个安装阶段。左侧是“用户体验设置”页，有开启客户体验改善计划的选项及相关说明；中间是“快捷方式”页，提供了创建桌面快捷方式等选项；右侧是“已准备好安装VMware Workstation Pro”页，明确显示可点击“安装”按钮完成安装，整体展示了VMware安装过程中设置环节和准备安装环节的界面内容，对应文档中“安装VMware”的步骤说明。

以上就是VMware在安装时的每一步操作，基本上就是点击 "下一步" 一直进行安装。





#### 挂载Linux系统

**1). 打开Vmware虚拟机，打开 `编辑` -> `虚拟网络编辑器(N)...`**

![kldeb35rtospvvxn7xechlcan9s](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/kldeb35rtospvvxn7xechlcan9s.png)
> 图注：这张图片展示了VMware Workstation的操作界面，呈现挂载Linux系统相关操作的起始步骤。界面上方的菜单栏中，「编辑」下拉菜单已被打开，其中「虚拟网络编辑器(N)...」选项被红色箭头明确指向并高亮选中。结合上下文，该图片对应挂载Linux系统的第一步操作，即打开Vmware虚拟机后，需通过该选项进入后续的网络配置界面，以完成虚拟机的网络挂载准备。



选择 `NAT模式`，然后选择右下角的 `更改设置`。

![y8dqbiuthokzkgxlk9vce8kpn0d](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/y8dqbiuthokzkgxlk9vce8kpn0d.png)
> 图注：图片展示了VMware虚拟网络编辑器界面。其中，VMnet8的类型为NAT模式，外部连接已连接，主机连接已启用，DHCP已启用，子网地址为192.168.200.0，子网掩码为255.255.255.0。界面右下角有“更改设置”按钮，箭头指向该按钮。该图片与上下文关系紧密，是挂载Linux系统步骤中，选择NAT模式并点击“更改设置”以设置子网IP为192.168.100.0的示例图。



设置子网IP为 **`192.168.100.0`**，然后选择 `应用` -> `确定`。

![qr8sbdni6ork3nxkxoycbe12n7l](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/qr8sbdni6ork3nxkxoycbe12n7l.png)
> 图注：这张图是VMware的虚拟网络编辑器界面，界面中红色虚线框标注了“VMnet8”的NAT模式条目，其子网地址为192.168.100.0，且该条目显示已连接、已启用。界面下方的子网IP输入框被红色箭头和红框标注，内容为192.168.100.0，子网掩码为255.255.255.0；界面右下角的“应用”“确定”按钮也被红色箭头标注。该界面内容对应文档中挂载Linux系统步骤里的网络设置操作，用于配置虚拟机的NAT模式网络参数。



**2). 解压 `资料/Linux镜像/CentOS7-1.zip` 到一个比较大的磁盘中 (没有中文的目录)。**

![g1klbiytmo1h6sxdiu6cpwygnkg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/g1klbiytmo1h6sxdiu6cpwygnkg.png)
> 图注：这张图片展示了Windows系统的文件资源管理器界面，当前路径为“此电脑>文档(E:)>02.课程研发>04.Web开发(All版本)>19.项目部署(Linux)>资料>02.Linux镜像”，该路径与文档中挂载Linux系统步骤2里提到的“资料/Linux镜像”路径完全对应。界面中清晰显示了名为“CentOS7-1.zip”的ZIP文件，修改日期为2024年7月26日15点27分，大小约为2351KB，该文件正是文档中要求解压的Linux镜像文件。



**3). 打开解压目录，双击 `.vmx` 文件，选择以 `Vmware Workstation` 打开这个文件。**

![tyjabzva7odrfsxv01fcniawnsg](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/tyjabzva7odrfsxv01fcniawnsg.png)
> 图注：图片展示了在VMware中挂载Linux镜像的操作界面。左侧是解压后的Linux镜像文件夹，其中“CentOS 7 64 位.vmx”文件被红色框突出显示。右侧弹出窗口询问如何打开该文件，有“继续使用此应用”“其他选项”“在Microsoft Store中查找应用”等选项，其中“继续使用此应用”被蓝色框突出显示。该图片与文档中“打开解压目录，双击.vmx文件，选择以Vmware Workstation打开这个文件”的操作步骤相关，直观呈现了操作步骤中的关键文件及选择应用界面。



**4). 挂载完毕之后，启动Linux服务器。**

![twdqboc7uomxxvxwarycpjmenoh](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/twdqboc7uomxxvxwarycpjmenoh.png)
> 图注：图片展示了在VMware Workstation中启动Linux服务器的界面。左侧为虚拟机列表，选中“CentOS7”；右侧是“CentOS7”虚拟机的设置界面，显示其状态为“已关闭”，并有“启动此虚拟机”选项。界面右下角显示了虚拟机的详细信息，如状态、配置文件、硬件版本等。该图片与文档中“4).挂载完毕之后，启动Linux服务器”步骤相关，直观呈现了启动前的界面及操作选项。



启动过程中，如果出现如下界面，选择 **`我已移动该虚拟机`**。

![twbnbvdeioiyf5xrr0vczw9jncb](https://notes-images-para.oss-cn-hongkong.aliyuncs.com/img/twbnbvdeioiyf5xrr0vczw9jncb.png)
> 图注：这张图是VMware Workstation虚拟机操作时的提示界面，对应文档中启动Linux服务器的步骤。弹窗标题为“CentOS7-1 - VMware Workstation”，提示内容为“此虚拟机可能已被移动或复制”，需要判断虚拟机的状态。界面提供了三个选项，其中红色箭头和红框重点标注的是“我已移动该虚拟机(M)”选项，另外还有“我已复制该虚拟机(P)”和“取消”选项，对应上下文要求出现该界面时选择标注的选项。

---
> 来源：飞书技术知识库（脱敏版） · 05 工程化与部署 / 开发环境与虚拟化 / VMware安装和虚拟机挂载 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/XXggwsPDSi69KWkenycc6mGjn2c)
