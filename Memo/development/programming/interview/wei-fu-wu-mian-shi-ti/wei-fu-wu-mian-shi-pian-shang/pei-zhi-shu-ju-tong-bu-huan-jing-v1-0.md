---
type: memo
date: 2026-11-06
aliases:
  - "配置数据同步环境v1.0"
---

<title>配置数据同步环境v1.0</title>

# 1 配置Canal+MQ数据同步环境



## 1.1 配置Mysql主从同步

根据Canal的工作原理，首先需要开启MySQL主从同步。



1.在MySQL中需要创建一个用户，并授权

进入mysql容器：

docker exec -it mysql /bin/bash

-- 使用命令登录：

mysql -u root -p

-- 创建用户 用户名：canal 密码：canal

```Java
create user 'canal'@'%' identified WITH mysql_native_password by 'canal';
```

-- 授权 \*.\*表示所有库

```Java
GRANT SELECT,REPLICATION SLAVE,REPLICATION CLIENT ON *.* TO 'canal'@'%';
FLUSH PRIVILEGES;
```

- **SELECT：** 允许用户查询（读取）数据库中的数据。
- **REPLICATION SLAVE：** 允许用户作为 MySQL 复制从库，用于同步主库的数据。
- **REPLICATION CLIENT：** 允许用户连接到主库并获取关于主库状态的信息。



在MySQL配置文件my.cnf中设置如下信息，开启 Binlog 写入功能，配置 binlog-format 为 ROW 模式

`ROW` 模式表示以行为单位记录每个被修改的行的变更

修改如下：

vi /usr/mysql/conf/my.cnf

```Java
[mysqld]
#打开binlog
log-bin=mysql-bin
#选择ROW(行)模式
binlog-format=ROW
#配置MySQL replaction需要定义，不要和canal的slaveId重复
server_id=1

expire_logs_days=3
max_binlog_size = 100m
max_binlog_cache_size = 512m

```

**说明：在学习阶段为了保证足够的服务器存储空间，binlog日志最大保存100m，mysql会定时清理binlog**



2、重启MySQL，查看配置信息

- 使用命令查看是否打开binlog模式：

`SHOW VARIABLES LIKE 'log_bin';`

![这张图片展示了执行`SHOW VARIABLES LIKE 'log_bin';`命令后的查询结果，表格的两列分别为Variable_name和Value。表格内仅一行数据，Variable_name的值为log_bin，对应的Value为ON。该结果用于验证MySQL的binlog模式是否成功开启，当log_bin的Value为ON时，说明已按要求开启了Binlog写入功能，与文档中配置MySQL主从同步的操作要求相对应。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-shang/WUNbbF5tboUG9xxwESQc2Zp0nlU.png)

ON表示开启binlog模式。



`show variables like 'binlog_format';`

![图片展示的是MySQL服务器中`binlog_format`变量的值为`ROW`的查询结果。表格中包含两列，分别是`Variable_name`和`Value`，其中`Variable_name`列显示为`binlog_format`，`Value`列显示为`ROW`。该图片与文档中配置Mysql主从同步的内容相关，用于说明当`binlog_format`的值为`row`时，表示MySQL服务器当前配置为使用行级别的二进制日志记录，这对于数据库复制和数据同步更为安全。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-shang/ETopbpVkPoVo8SxZDl9cYdNenZe.png)

当 `binlog_format` 的值为 `row` 时，表示 MySQL 服务器当前配置为使用行级别的二进制日志记录，这对于数据库复制和数据同步来说更为安全，因为它记录了对数据行的确切更改。



- 查看binlog日志文件列表：

`SHOW BINARY LOGS;`

![图片展示了MySQL的binlog日志文件列表。表格包含Log_name（日志名称）、File_size（文件大小）和Encrypted（加密）三列。Log_name列显示了多个binlog文件名，如mysql-bin.000001等，File_size列显示了各文件大小，Encrypted列显示所有文件为No。该图片与上文“查看binlog日志文件列表”相关，直观呈现了MySQL当前binlog日志文件的名称、大小及加密状态。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-shang/F1f6b7RwEoOfsFxbRyxc4XcPnFe.png)

- 查看当前正在写入的binlog文件：

`SHOW MASTER STATUS;`

![图片展示了MySQL中查看当前正在写入的binlog文件的命令执行结果。结果中显示binlog文件名为“mysql-bin.000008”，位置为156。该图片与上文配置Mysql主从同步时查看当前正在写入的binlog文件的内容相关，通过执行`SHOW MASTER STATUS;`命令，可获取到当前正在写入的binlog文件信息，以确保数据同步环境配置的准确性。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-shang/YnCVbqzGqoTUEzx9ybUcG8aQnic.png)

## 1.2 安装Canal（**使用下发虚拟机无需安装**）

获取canal镜像

`docker pull canal/canal-server:latest`



创建/data/soft/canal目录:

mkdir -p /data/soft/canal

在/data/soft/canal下创建 canal.properties，内容如下，注意修改mq的配置信息：

```Java
#################################################
#########               common argument         #############
#################################################
# tcp bind ip
canal.ip =
# register ip to zookeeper
canal.register.ip =
canal.port = 11111
canal.metrics.pull.port = 11112
# canal instance user/passwd
# canal.user = canal
# canal.passwd = E3619321C1A937C46A0D8BD1DAC39F93B27D4458

# canal admin config
#canal.admin.manager = 127.0.0.1:8089
canal.admin.port = 11110
canal.admin.user = admin
canal.admin.passwd = 4ACFE3202A5FF5CF467898FC58AAB1D615029441
# admin auto register
#canal.admin.register.auto = true
#canal.admin.register.cluster =
#canal.admin.register.name =

canal.zkServers =
# flush data to zk
canal.zookeeper.flush.period = 1000
canal.withoutNetty = false
# tcp, kafka, rocketMQ, rabbitMQ
canal.serverMode = rabbitMQ
# flush meta cursor/parse position to file
canal.file.data.dir = ${canal.conf.dir}
canal.file.flush.period = 1000
## memory store RingBuffer size, should be Math.pow(2,n)
canal.instance.memory.buffer.size = 16384
## memory store RingBuffer used memory unit size , default 1kb
canal.instance.memory.buffer.memunit = 1024 
## meory store gets mode used MEMSIZE or ITEMSIZE
canal.instance.memory.batch.mode = MEMSIZE
canal.instance.memory.rawEntry = true

## detecing config
canal.instance.detecting.enable = false
#canal.instance.detecting.sql = insert into retl.xdual values(1,now()) on duplicate key update x=now()
canal.instance.detecting.sql = select 1
canal.instance.detecting.interval.time = 3
canal.instance.detecting.retry.threshold = 3
canal.instance.detecting.heartbeatHaEnable = false

# support maximum transaction size, more than the size of the transaction will be cut into multiple transactions delivery
canal.instance.transaction.size =  1024
# mysql fallback connected to new master should fallback times
canal.instance.fallbackIntervalInSeconds = 60

# network config
canal.instance.network.receiveBufferSize = 16384
canal.instance.network.sendBufferSize = 16384
canal.instance.network.soTimeout = 30

# binlog filter config
canal.instance.filter.druid.ddl = true
canal.instance.filter.query.dcl = false
# 这个配置一定要修改
canal.instance.filter.query.dml = true
canal.instance.filter.query.ddl = false
canal.instance.filter.table.error = false
canal.instance.filter.rows = false
canal.instance.filter.transaction.entry = false
canal.instance.filter.dml.insert = false
canal.instance.filter.dml.update = false
canal.instance.filter.dml.delete = false

# binlog format/image check
canal.instance.binlog.format = ROW,STATEMENT,MIXED 
canal.instance.binlog.image = FULL,MINIMAL,NOBLOB

# binlog ddl isolation
canal.instance.get.ddl.isolation = false

# parallel parser config
canal.instance.parser.parallel = true
## concurrent thread number, default 60% available processors, suggest not to exceed Runtime.getRuntime().availableProcessors()
#canal.instance.parser.parallelThreadSize = 16
## disruptor ringbuffer size, must be power of 2
canal.instance.parser.parallelBufferSize = 256

# table meta tsdb info
canal.instance.tsdb.enable = true
canal.instance.tsdb.dir = ${canal.file.data.dir:../conf}/${canal.instance.destination:}
canal.instance.tsdb.url = jdbc:h2:${canal.instance.tsdb.dir}/h2;CACHE_SIZE=1000;MODE=MYSQL;
canal.instance.tsdb.dbUsername = canal
canal.instance.tsdb.dbPassword = canal
# dump snapshot interval, default 24 hour
canal.instance.tsdb.snapshot.interval = 24
# purge snapshot expire , default 360 hour(15 days)
canal.instance.tsdb.snapshot.expire = 360

#################################################
#########               destinations            #############
#################################################
canal.destinations = xzb-canal
# conf root dir
canal.conf.dir = ../conf
# auto scan instance dir add/remove and start/stop instance
canal.auto.scan = true
canal.auto.scan.interval = 5
# set this value to 'true' means that when binlog pos not found, skip to latest.
# WARN: pls keep 'false' in production env, or if you know what you want.
canal.auto.reset.latest.pos.mode = false

canal.instance.tsdb.spring.xml = classpath:spring/tsdb/h2-tsdb.xml
#canal.instance.tsdb.spring.xml = classpath:spring/tsdb/mysql-tsdb.xml

canal.instance.global.mode = spring
canal.instance.global.lazy = false
canal.instance.global.manager.address = ${canal.admin.manager}
#canal.instance.global.spring.xml = classpath:spring/memory-instance.xml
canal.instance.global.spring.xml = classpath:spring/file-instance.xml
#canal.instance.global.spring.xml = classpath:spring/default-instance.xml

##################################################
#########             MQ Properties      #############
##################################################
# aliyun ak/sk , support rds/mq
canal.aliyun.accessKey =
canal.aliyun.secretKey =
canal.aliyun.uid=

canal.mq.flatMessage = true
canal.mq.canalBatchSize = 50
canal.mq.canalGetTimeout = 100
# Set this value to "cloud", if you want open message trace feature in aliyun.
canal.mq.accessChannel = local

canal.mq.database.hash = true
canal.mq.send.thread.size = 30
canal.mq.build.thread.size = 8

##################################################
#########                    Kafka                   #############
##################################################
kafka.bootstrap.servers = 127.0.0.1:9092
kafka.acks = all
kafka.compression.type = none
kafka.batch.size = 16384
kafka.linger.ms = 1
kafka.max.request.size = 1048576
kafka.buffer.memory = 33554432
kafka.max.in.flight.requests.per.connection = 1
kafka.retries = 0

kafka.kerberos.enable = false
kafka.kerberos.krb5.file = "../conf/kerberos/krb5.conf"
kafka.kerberos.jaas.file = "../conf/kerberos/jaas.conf"

##################################################
#########                   RocketMQ         #############
##################################################
rocketmq.producer.group = test
rocketmq.enable.message.trace = false
rocketmq.customized.trace.topic =
rocketmq.namespace =
rocketmq.namesrv.addr = 127.0.0.1:9876
rocketmq.retry.times.when.send.failed = 0
rocketmq.vip.channel.enabled = false
rocketmq.tag = 

##################################################
#########                   RabbitMQ         #############
##################################################
rabbitmq.host = 192.168.101.68
rabbitmq.virtual.host = /hmall
rabbitmq.exchange = exchange.canal-hmall
rabbitmq.username = hmall
rabbitmq.password = 123
rabbitmq.deliveryMode = 2

```



上边配置文件中主要修改RabbitMQ 区域中MQ的相关配置。

```Java
##################################################
#########                   RabbitMQ         #############
##################################################
rabbitmq.host = 192.168.101.68
rabbitmq.virtual.host = /hmall
rabbitmq.exchange = exchange.canal-hmall
rabbitmq.username = hmall
rabbitmq.password = 123
rabbitmq.deliveryMode = 2
```



创建instance.properties，内容如下：

`canal.instance.master.journal.name` 用于指定主库正在写入的 binlog 文件的名称。

如果不配置 `canal.instance.master.journal.name`，Canal 会尝试自动检测 MySQL 主库的 binlog 文件，并从最新位置开始进行复制。

```Java
#################################################
## mysql serverId , v1.0.26+ will autoGen
canal.instance.mysql.slaveId=1000

# enable gtid use true/false
canal.instance.gtidon=false

# position info
canal.instance.master.address=192.168.101.68:3306
canal.instance.master.journal.name=mysql-bin.000001
canal.instance.master.position=0
canal.instance.master.timestamp=
canal.instance.master.gtid=

# rds oss binlog
canal.instance.rds.accesskey=
canal.instance.rds.secretkey=
canal.instance.rds.instanceId=

# table meta tsdb info
canal.instance.tsdb.enable=true
#canal.instance.tsdb.url=jdbc:mysql://127.0.0.1:3306/canal_tsdb
#canal.instance.tsdb.dbUsername=canal
#canal.instance.tsdb.dbPassword=canal

#canal.instance.standby.address =
#canal.instance.standby.journal.name =
#canal.instance.standby.position =
#canal.instance.standby.timestamp =
#canal.instance.standby.gtid=

# username/password
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
canal.instance.connectionCharset = UTF-8
# enable druid Decrypt database password
canal.instance.enableDruid=false
#canal.instance.pwdPublicKey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALK4BUxdDltRRE5/zXpVEVPUgunvscYFtEip3pmLlhrWpacX7y7GCMo2/JM6LeHmiiNdH1FWgGCpUfircSwlWKUCAwEAAQ==

# table regex
# canal.instance.filter.regex=test01\\..*,test02\\..*
#canal.instance.filter.regex=test01\\..*,test02\\.t1
#canal.instance.filter.regex=jzo2o-foundations\\.serve_sync,jzo2o-orders-0\\.orders_seize,jzo2o-orders-0\\.orders_dispatch,jzo2o-orders-0\\.serve_provider_sync,jzo2o-customer\\.serve_provider_sync
#canal.instance.filter.regex=jzo2o-orders-1\\.orders_dispatch,jzo2o-orders-1\\.orders_seize,jzo2o-foundations\\.serve_sync,jzo2o-customer\\.serve_provider_sync,jzo2o-orders-1\\.serve_provider_sync,jzo2o-orders-1\\.history_orders_sync,jzo2o-orders-1\\.history_orders_serve_sync,jzo2o-market\\.activity
canal.instance.filter.regex=hm-item\\.item_sync
# table black regex
canal.instance.filter.black.regex=mysql\\.slave_.*
# table field filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
#canal.instance.filter.field=test1.t_product:id/subject/keywords,test2.t_company:id/name/contact/ch
# table field black filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
#canal.instance.filter.black.field=test1.t_product:subject/product_image,test2.t_company:id/name/contact/ch

# mq config
#canal.mq.topic=topic_test01
# dynamic topic route by schema or table regex
#canal.mq.dynamicTopic=mytest1.user,mytest2\\..*,.*\\..*
#canal.mq.dynamicTopic=topic_test01:test01\\..*,topic_test02:test02\\..*
#canal.mq.dynamicTopic=canal-mq-jzo2o-orders-dispatch:jzo2o-orders-0\\.orders_dispatch,canal-mq-jzo2o-orders-seize:jzo2o-orders-0\\.orders_seize,canal-mq-jzo2o-foundations:jzo2o-foundations\\.serve_sync,canal-mq-jzo2o-customer-provider:jzo2o-customer\\.serve_provider_sync,canal-mq-jzo2o-orders-provider:jzo2o-orders-0\\.serve_provider_sync
#canal.mq.dynamicTopic=canal-mq-jzo2o-orders-dispatch:jzo2o-orders-1\\.orders_dispatch,canal-mq-jzo2o-orders-seize:jzo2o-orders-1\\.orders_seize,canal-mq-jzo2o-foundations:jzo2o-foundations\\.serve_sync,canal-mq-jzo2o-customer-provider:jzo2o-customer\\.serve_provider_sync,canal-mq-jzo2o-orders-provider:jzo2o-orders-1\\.serve_provider_sync,canal-mq-jzo2o-orders-serve-history:jzo2o-orders-1\\.history_orders_serve_sync,canal-mq-jzo2o-orders-history:jzo2o-orders-1\\.history_orders_sync,canal-mq-jzo2o-market-resource:jzo2o-market\\.activity
canal.mq.dynamicTopic=canal-mq-hmall-item:hm-item\\.item_sync
canal.mq.partition=0
# hash partition config
#canal.mq.partitionsNum=3
#canal.mq.partitionHash=test.table:id^name,.*\\..*
#canal.mq.dynamicTopicPartitionNum=test.*:4,mycanal:6
#################################################

```

canal.instance.filter.regex和canal.mq.dynamicTopic的配置稍后解释。



创建日志目录：

mkdir -p /data/soft/canal/logs   /data/soft/canal/conf

启动容器：

```Java
docker run --name canal -p 11111:11111 -d -v /data/soft/canal/instance.properties:/home/admin/canal-server/conf/xzb-canal/instance.properties  -v /data/soft/canal/canal.properties:/home/admin/canal-server/conf/canal.properties -v /data/soft/canal/logs:/home/admin/canal-server/logs/xzb-canal -v /data/soft/canal/conf:/home/admin/canal-server/conf/xzb-canal canal/canal-server:latest
```



## 1.3 安装RabbitMQ（**使用下发虚拟机无需安装**）

安装RabbitMQ参考 微服务课程内容。

配置虚拟主机：/hmall

账号:hmall

密码：123



## 1.4 配置Canal+RabbitMQ

下边通过配置Canal与RabbitMQ，保证Canal收到binlog消息将数据发送至MQ。

最终我们要实现的是：

**修改hm-item数据库下的item_sync表的数据后通过canal将修改信息发送到MQ。**



**1、在Canal中配置RabbitMQ的连接信息**

**修改/data/soft/canal/canal.properties**

```Java
# tcp, kafka, rocketMQ, rabbitMQ
canal.serverMode = rabbitMQ

##################################################
#########                     RabbitMQ             #############
##################################################
rabbitmq.host = 192.168.101.68
rabbitmq.virtual.host = /hmall
rabbitmq.exchange = exchange.canal-hmall
rabbitmq.username = hmall
rabbitmq.password = 123
rabbitmq.deliveryMode = 2
```

本项目用于数据同步的MQ交换机：exchange.canal-hmall

虚拟主机地址：/hmall

账号和密码：hmall/ 123

rabbitmq.deliveryMode = 2   设置消息持久化



**2、设置需要读取binlog的mysql库和表**

**修改/data/soft/canal/instance.properties**

- `canal.instance.filter.regex` 需要监听的mysql库和表

  - 全库： `.*\\..*`
  - 指定库下的所有表： `canal\\..*`
  - 指定库下的指定表： `canal\\.canal,test\\.test`
  
    - `库名\\.表名`：转义需要用\\\，使用逗号分隔多个库

这里配置监听 hm-item数据库下item_sync表，如下：

```Java
canal.instance.filter.regex=hm-item\\.item_sync
```

**3、在Canal配置MQ的topic**

这里使用动态topic，格式为：topic:[schema](https://so.csdn.net/so/search?q=schema&spm=1001.2101.3001.7020).table,topic:schema.table,topic:schema.table

配置如下：

```Java
canal.mq.dynamicTopic=canal-mq-hmall-item:hm-item\\.item_sync
```

上边的配置表示：对hm-item数据库的item_sync表的修改消息发到exchange.canal-hmall交换机，topic为canal-mq-hmall-item关联的队列中。



**4、进入rabbitMQ配置交换机和队列**

创建exchange.canal-hmall交换机：

![该图片是RabbitMQ的交换机配置界面，用于创建新的交换机。界面中Virtual host设为/hmall，交换机名称为exchange.canal-hmall，类型选择topic，Durability设为Durable，Auto delete和Internal均设为No。界面下方还提供了Arguments配置栏，以及Alternate exchange配置选项，左下角有Add exchange的操作按钮，这一界面对应了文档中配置Canal+RabbitMQ时创建exchange.canal-hmall交换机的步骤。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-shang/QTiUbn4oMoCCyKxNVqQcB1Gdnph.png)

创建队列：canal-mq-hmall-item

![图片展示的是rabbitMQ配置队列的界面。在“Add a new queue”区域，Virtual host下拉框选中“/hmall”，Type下拉框选中“Classic”，Name输入框填写“canal-mq-hmall-item”，Durability下拉框选中“Durable”，Auto delete下拉框选中“No”，Arguments区域有多个选项，如Message TTL、Auto expire等，底部有“Add queue”按钮。该图片与上文配置Canal+RabbitMQ中创建队列的内容相关，直观呈现了创建队列时的参数设置界面。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-shang/JdSzbenapoASIex07rOcgj14nIc.png)

绑定交换机：

![该图片展示了RabbitMQ交换机的绑定配置界面，对应的交换机是virtual host为/hmall的exchange.canal-hmall。在该界面的绑定配置区域，用户需完成将该交换机与指定队列绑定的操作，其中To queue下拉框已选定目标队列名称为canal-mq-hmall-item，Routing key输入框中填写了相同的canal-mq-hmall-item值，图中这两个输入项被红框标注，界面底部还有用于确认绑定的Bind按钮，此界面对应文档中进入rabbitMQ配置交换机和队列后进行绑定的操作步骤。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-shang/XaAjbVZgCo6jecxabX5cMl7KnpH.png)

绑定成功：

![这张图片展示的是RabbitMQ中交换机与队列的绑定配置界面，核心为exchange.canal-hmall交换机的绑定详情，该交换机处于虚拟主机/hmall下。界面的Bindings区域清晰显示，该交换机绑定了名称为canal-mq-hmall-item的队列，绑定使用的路由键为canal-mq-hmall-item，绑定操作的相关按钮也在界面中呈现，此配置与上下文内容对应，是为实现hm-item数据库item_sync表数据的同步，完成交换机与对应队列绑定的配置界面呈现。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-shang/D360bY0cvoUdlIxF6BVcHgB2nCj.png)



## 1.5 测试数据同步

复制item表到item_sync表。

重启canal

修改hmall-item数据库的item_sync表的数据，稍等片刻查看canal-mq-hmall-item队列，如果队列中有的消息说明同步成功，如下图：

![这张是RabbitMQ的队列管理页面，展示了名为canal-mq-hmall-item的队列信息。该队列所属的虚拟主机为/hmall，类型为classic，状态为idle，当前Ready消息数为1，Unacked消息数为0，消息总数为1，消息流入速率为0.00/s，符合测试数据同步成功后的预期状态，对应文档中测试步骤里同步成功后队列存在消息的情况。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-shang/I2hWbgGSBoJykaxQhkgcQPoknXe.png)

如果没有同步到 MQ参考常见问题中“数据不同步”进行解决。



我们可以查询队列中的消息内容发现它一条type为"UPDATE"的消息，如下所示：

```JSON
{
   "data" : [
      {
         "brand" : "RIMOWA",
         "category" : "拉杆箱",
         "comment_count" : "0",
         "create_time" : "2019-05-01 00:00:00",
         "creater" : null,
         "id" : "317578",
         "image" : "https://m.360buyimg.com/mobilecms/s720x720_jfs/t6934/364/1195375010/84676/e9f2c55f/597ece38N0ddcbc77.jpg!q70.jpg.webp",
         "isAD" : "0",
         "name" : "RIMOWA 21寸托运箱拉杆箱 SALSA AIR系列果绿色 820.70.36.41",
         "price" : "28900",
         "sold" : "0",
         "spec" : "{\"颜色\": \"红色\", \"尺码\": \"21寸\"}",
         "status" : "1",
         "stock" : "10000",
         "update_time" : "2023-05-06 11:06:17",
         "updater" : "100"
      }
   ],
   "database" : "hm-item",
   "es" : 1724319800000.0,
   "id" : 1,
   "isDdl" : false,
   "mysqlType" : {
      "brand" : "varchar(100)",
      "category" : "varchar(200)",
      "comment_count" : "int",
      "create_time" : "datetime",
      "creater" : "bigint",
      "id" : "bigint",
      "image" : "varchar(200)",
      "isAD" : "tinyint",
      "name" : "varchar(200)",
      "price" : "int",
      "sold" : "int",
      "spec" : "varchar(200)",
      "status" : "int",
      "stock" : "int",
      "update_time" : "datetime",
      "updater" : "bigint"
   },
   "old" : [
      {
         "name" : "RIMOWA 21寸托运箱拉杆箱 SALSA AIR系列果绿色 820.70.36.4"
      }
   ],
   "pkNames" : null,
   "sql" : "",
   "sqlType" : {
      "brand" : 12,
      "category" : 12,
      "comment_count" : 4,
      "create_time" : 93,
      "creater" : -5,
      "id" : -5,
      "image" : 12,
      "isAD" : -6,
      "name" : 12,
      "price" : 4,
      "sold" : 4,
      "spec" : 12,
      "status" : 4,
      "stock" : 4,
      "update_time" : 93,
      "updater" : -5
   },
   "table" : "item_sync",
   "ts" : 1724319801224.0,
   "type" : "UPDATE"
}

```



# 常见问题

## 数据不同步

当发现修改了数据库后修改的数据并没有发送到MQ，通过查看Canal的日志发现下边的错误。

进入Canal目录，查看日志：

```Java
tail -f /data/soft/canal/logs/xzb-canal.log 
```

Canal报错如下：

```Java
2023-09-22 08:34:40.802 [destination = xzb-canal , address = /192.168.101.68:3306 , EventParser] WARN  c.a.o.c.p.inbound.mysql.rds.RdsBinlogEventParserProxy - ---> find start position successfully, EntryPosition[included=false,journalName=mysql-bin.000055,position=486221,serverId=1,gtid=,timestamp=1695341830000] cost : 13ms , the next step is binlog dump
2023-09-22 08:34:40.811 [destination = xzb-canal , address = /192.168.101.68:3306 , EventParser] ERROR c.a.o.canal.parse.inbound.mysql.dbsync.DirectLogFetcher - I/O error while reading from client socket
java.io.IOException: Received error packet: errno = 1236, sqlstate = HY000 errmsg = Could not find first log file name in binary log index file
        at com.alibaba.otter.canal.parse.inbound.mysql.dbsync.DirectLogFetcher.fetch(DirectLogFetcher.java:102) ~[canal.parse-1.1.5.jar:na]
        at com.alibaba.otter.canal.parse.inbound.mysql.MysqlConnection.dump(MysqlConnection.java:238) [canal.parse-1.1.5.jar:na]
        at com.alibaba.otter.canal.parse.inbound.AbstractEventParser$1.run(AbstractEventParser.java:262) [canal.parse-1.1.5.jar:na]
        at java.lang.Thread.run(Thread.java:748) [na:1.8.0_181]
```

找到关键的位置：**Could not find first log file name in binary log index file**

根据日志分析是Canal找不到mysql-bin.000055 的486221位置，原因是mysql-bin.000055文件不存在，这是由于为了节省磁盘空间将binlog日志清理了。



解决方法：

把canal复位从最开始开始同步的位置。

1）首先重置mysql的bin log：

连接mysql执行：reset master

执行后所有的binlog删除，从000001号开始

通过show master status;查看 ，结果显示 mysql-bin.000001

2）先停止canal

docker stop canal

3）删除meta.dat

 rm -rf /data/soft/canal/conf/meta.dat 

4) 再启动canal

docker start canal



## MQ同步消息无法消费

这里以Es和MySQL之间的同步举例：

当出现ES和MySQL数据不同步时可能会出现MQ的同步消息无法被消费，比如：从MySQL删除一条记录通过同步程序将ES中对应的记录进行删除，此时由于ES中没有该记录导致删除ES中的记录失败。出现此问题的原因是因为测试数据混乱导致，可以手动将MQ中的消息删除。

进入MQ的管理控制台，进入要清理消息的队列，通过purge功能清理消息：

![这张图片展示了MQ管理控制台中的一个操作按钮区域，区域上方标注有“Purge”字样，下方是一个醒目的橙色按钮，按钮上标注有“Purge Messages”的文字，该按钮是用于清理MQ队列消息的功能入口，对应文档中提及的清理MQ同步消息的操作步骤。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-shang/F2cNb5GLToTxXtxGM43csjhlndf.png)

---
> 来源：飞书技术知识库（脱敏版） · 08 面试知识库 / 微服务面试题 / 微服务面试篇（上） / 配置数据同步环境v1.0 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/QgsHwSqFDitNVqkZ9a4cxQVznhh)
