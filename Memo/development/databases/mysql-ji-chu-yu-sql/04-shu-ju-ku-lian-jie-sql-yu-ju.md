---
type: memo
date: 2026-11-06
aliases:
  - "04 数据库连接 sql语句"
---

<title>04 数据库连接 sql语句</title>



# idea连接数据库

打开ide在右侧找到下图中的MySQL图标 依次点击操作

![这张图是IDEA软件的Database界面，核心展示了连接数据库的操作步骤，左上角有带红色标记的加号按钮，点击该按钮后展开的下拉菜单中，“Data Source”选项被红色框标注，点击该选项后弹出的数据库列表里，“MySQL”选项被红色框特别突出，还带有红色标记，整体对应了文档中打开IDE后，点击MySQL相关图标连接数据库的操作指引，是idea连接数据库流程的操作演示。](../../../../Attachments/development/databases/mysql-ji-chu-yu-sql/X2rCbgmgyoh23QxcIZ8ch3EHnUc.png)

登录并连接数据库

![这是IDEA中配置MySQL数据库连接的“Data Sources and Drivers”设置界面，用于完成MySQL数据库的连接配置。界面中连接类型默认设为default，驱动为MySQL的5.2及以上版本，需填写的信息有Host、Port、用户名和密码，其中用户名和密码的填写位置被红色框标注提示。界面下方的连接测试区域显示“Test Connection”按钮旁标注“点击步骤②显示测试连接成功”，区域内提示填写自己设置的用户名和密码，测试结果显示连接成功，同时界面底部还有“Test Connection”“OK”“Apply”三个操作按钮，分别标注了步骤序号，整体是idea连接MySQL数据库的关键配置操作界面。](../../../../Attachments/development/databases/mysql-ji-chu-yu-sql/HmcmbFP5boPKUSxWqC8cdQranoh.png)

创建数据库文件 并设置默认数据库方言

![图片展示了在IntelliJ IDEA中创建数据库文件并设置默认数据库方言的操作界面。左侧为项目结构，选中“test01.sql”文件。右侧弹出“SQL Dialect is not configured”窗口，显示“Global SQL Dialect”和“Project SQL Dialect”均为“Generic SQL”。窗口右上角有“Change dialect to...”选项，下方有“OK”和“Cancel”按钮。该图片与文档中“创建数据库文件并设置默认数据库方言”内容对应，直观呈现了操作步骤。](../../../../Attachments/development/databases/mysql-ji-chu-yu-sql/Ok7Wba0Hfoipxfxz6hscnrPbnHL.png)

创建数据库 并显示

![这张图展示了数据库相关的操作界面，左侧是test01.sql文件的代码区域，其中用红框标注了两条创建数据库的SQL命令，分别为“create database db01;”和“create database db02;”，旁侧标注有“通过命令创建数据库”的提示；右侧是数据库连接界面，标注为“Database”区域，界面上方显示有“2 of 6”的内容，右侧的数据库列表区域用红框勾选了db01和db02选项，同时标注有“选择创建的数据库 显示”的提示，整体对应文档中创建数据库并连接查看的操作内容。](../../../../Attachments/development/databases/mysql-ji-chu-yu-sql/RxiWbBZMMoClbqxtqo4crbHenBd.png)

# Mysql_DDL语句

## 操作库的crud

```SQL
-- 创建一个库 名字itheima142001
create database itheima142001;
-- 创建库 如果不存在则创建 itheima142002
create database if not exists itheima142002;
-- 修改库的编码为utf8
alter database itheima142001 character set utf8;
-- 删除库
drop database itheima142001;
-- 查看所有库
show databases;
-- 切换库
use itheima142002;
-- 查看当前库的创建语句
show create database itheima142002;
```

## 操作表的crud

1. **Java与sql数据类型比对**

<table><colgroup><col/><col/><col/></colgroup><thead><tr><th>Java</th><th colspan="2">sql</th></tr></thead><tbody><tr><td>int</td><td>int</td><td></td></tr><tr><td>short</td><td>smallint</td><td></td></tr><tr><td>byte</td><td>tinyint</td><td></td></tr><tr><td>long</td><td>bigint</td><td></td></tr><tr><td>float</td><td>float</td><td></td></tr><tr><td>double</td><td>double</td><td></td></tr><tr><td>LocalDate</td><td>date</td><td></td></tr><tr><td>LocalTime</td><td>time</td><td></td></tr><tr><td>LocalDateTime</td><td>datetime</td><td></td></tr><tr><td>String</td><td>varchar(变长)</td><td>char（定长）</td></tr><tr><td>BigDecimal</td><td>DECIMAL(M,D)</td><td></td></tr></tbody></table>

```SQL
-- 创建表
create table user(
    id int,
    name varchar(20),
    age int,
    gender char(1)
);
```

---
> 来源：飞书技术知识库（脱敏版） · 02 数据库与持久层 / MySQL 基础与 SQL / 04 数据库连接 sql语句 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/UIFBw9lcmiXpWuk1L8yc9pBwnWg)
