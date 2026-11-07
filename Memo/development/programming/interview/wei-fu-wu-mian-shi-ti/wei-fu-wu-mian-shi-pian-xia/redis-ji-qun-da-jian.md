---
type: memo
date: 2026-11-06
aliases:
  - "Redis集群搭建"
---
# Redis集群搭建

## 1.Redis主从

单节点Redis的并发能力是有上限的，要进一步提高Redis的并发能力，这里讲一种方案就是搭建主从集群，实现读写分离。

### 1.1.主从集群结构

下图就是一个简单的Redis主从集群结构：

![图片展示了Redis主从集群结构。左侧为客户端，通过RedisClient与集群交互。集群中有master节点和两个slave节点，slave节点又称为replica。写操作通过RedisClient访问master，master会将数据同步给slave；读操作可访问任意slave，slave间也存在数据同步。该图与上下文紧密相关，直观呈现了主从集群中写操作与读操作的访问路径及数据同步机制。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/NCr5boQ77ogJs4xGDksc5NkYnXc.png)

如图所示，集群中有一个master节点、两个slave节点（现在叫replica）。当我们通过Redis的Java客户端访问主从集群时，应该做好路由：

- 如果是写操作，应该访问master节点，master会自动将数据同步给两个slave节点
- 如果是读操作，建议访问各个slave节点，从而分担并发压力



### 1.2.搭建主从集群

我们会在同一个虚拟机中利用3个Docker容器来搭建主从集群，容器信息如下：

<table><colgroup><col/><col/><col/><col/></colgroup><thead><tr><th><h5><b>容器名</b></h5></th><th><h5><b>角色</b></h5></th><th><h5><b>IP</b></h5></th><th><h5><b>映射端口</b></h5></th></tr></thead><tbody><tr><td>r1</td><td>master</td><td>192.168.101.68</td><td>7001</td></tr><tr><td>r2</td><td>slave</td><td>192.168.101.68</td><td>7002</td></tr><tr><td>r3</td><td>slave</td><td>192.168.101.68</td><td>7003</td></tr></tbody></table>



#### 1.2.1.启动多个Redis实例

我们利用课前资料提供的docker-compose文件来构建主从集群：

![这张图片展示的是Windows系统的文件资源管理器窗口，路径为“新加卷(D:)>课程资料>服务框架>day10-Redis高级>资料”，窗口中列出了两个文件，其中被红色方框突出标注的是名为“docker-compose.yaml”的Yaml源文件，大小为1KB；另一个是名为“sentinel.conf”的CONF文件，大小同样为1KB，该文件是搭建Redis主从集群所需的相关配置文件，对应文档中提及的用于构建Redis主从集群的docker-compose文件。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/TqOvbuuNxoc79cxrXctcz5j0njg.png)

文件内容如下：

```YAML
version: "6.2.7"

services:
  r1:
    image: redis:6.2.7
    container_name: r1
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7001"]
  r2:
    image: redis:6.2.7
    container_name: r2
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7002"]
  r3:
    image: redis:6.2.7
    container_name: r3
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7003"]

```



将其上传至虚拟机的`/root/redis`目录下：

![这张图片展示了虚拟机`/root/redis`目录下的文件列表，目录内仅包含一个名为`docker-compose.yaml`的文件，其大小为1KB，该文件正是用于构建Redis主从集群的docker-compose文件，这与文中提及的需将相关文件上传至虚拟机该目录的操作要求相呼应，界面顶部的路径输入栏也明确显示当前所在路径为`/root/redis/`。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/DyoUbiqlroJylexYmwkcyOVpnGe.png)

执行命令，运行集群：

```Bash
docker-compose up -d
```

结果：

![这张图片展示的是在Linux虚拟机中执行Docker Compose启动命令后的终端输出内容，对应搭建Redis主从集群的操作步骤。执行的命令是`docker-compose up -d`，输出显示共4项任务已全部完成，其中名为redis-cluster的网络已创建，作为主节点的r1、作为从节点的r2和r3这三个容器均已成功启动，各项任务的执行耗时也在输出中依次列出。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/RISabkpdIoZMTuxXE7IcgVG9nYg.png)



由于采用的是host模式，我们看不到端口映射。不过能直接在宿主机通过ps命令查看到Redis进程：

![图片展示了在虚拟机中执行`ps -ef | grep redis`命令的结果。命令执行后，显示了多个Redis进程的相关信息，包括进程ID、父进程ID、进程状态、运行时间、命令等。其中，进程ID为50200、50230、50240的进程对应命令为`redis-server *:7001`、`redis-server *:7003`、`redis-server *:7002`，与文档中搭建的Redis主从集群中r1、r3、r2容器的映射端口7001、7003、7002相对应，验证了Redis进程的运行情况。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/LTPnb9iVcos2isxYyvBcOMiankf.png)



#### 1.2.2.建立集群

虽然我们启动了3个Redis实例，但是它们并没有形成主从关系。我们需要通过命令来配置主从关系：

```Bash
# Redis5.0以前
slaveof <masterip> <masterport>
# Redis5.0以后
replicaof <masterip> <masterport>
```



有临时和永久两种模式：

- 永久生效：在redis.conf文件中利用`slaveof`命令指定`master`节点
- 临时生效：直接利用redis-cli控制台输入`slaveof`命令，指定`master`节点



我们测试临时模式，首先连接`r2`，让其以`r1`为master

```Bash
# 连接r2
docker exec -it r2 redis-cli -p 7002
# 认r1主，也就是7001
slaveof 192.168.101.68 7001
```



然后连接`r3`，让其以`r1`为master

```Bash
# 连接r3
docker exec -it r3 redis-cli -p 7003
# 认r1主，也就是7001
slaveof 192.168.101.68 7001
```



然后连接`r1`，查看集群状态：

```Bash
# 连接r1
docker exec -it r1 redis-cli -p 7001
# 查看集群状态
info replication
```

结果如下：

```Bash
127.0.0.1:7001> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.101.68,port=7002,state=online,offset=140,lag=1
slave1:ip=192.168.101.68,port=7003,state=online,offset=140,lag=1
master_failover_state:no-failover
master_replid:16d90568498908b322178ca12078114e6c518b86
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:140
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:140
```

可以看到，当前节点`r1:7001`的角色是`master`，有两个slave与其连接：

- `slave0`：`port`是`7002`，也就是`r2`节点
- `slave1`：`port`是`7003`，也就是`r3`节点



#### 1.2.3.测试

依次在`r1`、`r2`、`r3`节点上执行下面命令：

```Bash
set num 123

get num
```



你会发现，只有在`r1`这个节点上可以执行`set`命令（**写操作**）

![图片展示的是在Redis节点`r1:7001`上执行`set`命令的界面。命令为`192.168.101.68-7001:0> set num 123`，执行结果为`"OK"`。该图片与上文测试Redis主从集群中临时模式下节点权限的内容相关，用于说明在`r1`节点上可以执行`set`命令（写操作），体现了Redis主从集群中节点权限的差异，即只有`master`节点可执行写操作。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/SqbKbaQNToQ6SIxNlaOcJyFYnRe.png)

其它两个节点只能执行`get`命令（**读操作**）。也就是说读写操作已经分离了。

![这张图是Redis集群读写测试的命令行输出内容，显示在编号为192.168.101.68的Redis节点（端口7002，为只读从节点）上执行`set num 123`命令后，返回了只读副本无法执行写操作的错误提示，内容为"READONLY You can't write against a read only replica."，该输出对应文档中“只读节点仅能执行读操作、读写分离”的测试结论，验证了临时搭建的Redis主从集群的读写功能特性。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/JlZRb6imPogfldxNCSgcpMIbnNg.png)





## 2.Redis哨兵

主从结构中master节点的作用非常重要，一旦故障就会导致集群不可用。那么有什么办法能保证主从集群的高可用性呢？



### 2.1.哨兵集群介绍

Redis提供了`哨兵`（`Sentinel`）机制来监控主从集群监控状态，确保集群的高可用性。

哨兵集群作用原理图：

![这张图是Redis哨兵集群的作用原理图，展示了各组件的关联关系：上方的三个橙色Sentinel构成哨兵集群，箭头显示它会监控由1个master和2个slave组成的Redis主从集群的状态，还会向Redis Client推送服务状态变更通知。主从节点之间标注了“数据同步”，体现slave会与master进行数据同步，而哨兵则会通过监控、故障处理等维护整个集群的可靠性，这对应了文档中提到的哨兵的状态监控、故障恢复与状态通知作用。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/Pf9IbCjK5ot005xXZRacoXQNnpf.png)

哨兵的作用如下：

- **状态监控**：`Sentinel` 会不断检查您的`master`和`slave`是否按预期工作
- **故障恢复（failover）**：如果`master`故障，`Sentinel`会将一个`slave`提升为`master`。当故障实例恢复后会成为`slave`
- **状态通知**：`Sentinel`充当`Redis`客户端的服务发现来源，当集群发生`failover`时，会将最新集群信息推送给`Redis`的客户端



### 2.2.搭建哨兵集群

首先，我们停掉之前的redis集群：

```Bash
# 老版本DockerCompose
docker-compose down

# 新版本Docker
docker compose down
```



然后，我们找到课前资料提供的sentinel.conf文件：

![这张图片展示了Windows系统的文件资源管理器界面，路径为“新加卷(D:)\\课程资料\\服务框架\\day10-Redis高级\\资料”，界面内有两个文件，分别是大小1KB的Yaml源文件docker-compose.yaml，以及大小1KB的CONF文件sentinel.conf，其中sentinel.conf被蓝色高亮选中。这一图片对应的上下文是Redis哨兵集群搭建的操作步骤，对应了文档中“找到课前资料提供的sentinel.conf文件”这一内容，展示了该配置文件的实际获取路径和文件信息。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/NOk3bva1qodE7xxzTy2culpNn2g.png)



其内容如下：

```Bash
sentinel announce-ip "192.168.101.68"
sentinel monitor hmaster 192.168.101.68 7001 2
sentinel down-after-milliseconds hmaster 5000
sentinel failover-timeout hmaster 60000
```

说明：

- `sentinel announce-ip "192.168.101.68"`：声明当前sentinel的ip
- `sentinel monitor hmaster 192.168.101.68 7001 2`：指定集群的主节点信息 

  - `hmaster`：主节点名称，自定义，任意写
  - `192.168.101.68 7001`：主节点的ip和端口
  - `2`：认定`master`下线时的`quorum`值
- `sentinel down-after-milliseconds hmaster 5000`：声明master节点超时多久后被标记下线
- `sentinel failover-timeout hmaster 60000`：在第一次故障转移失败后多久再次重试



我们在虚拟机的`/root/redis`目录下新建3个文件夹：`s1`、`s2`、`s3`:

![图片展示了在虚拟机`/root/redis`目录下新建的3个文件夹，分别是`s1`、`s2`、`s3`，它们被红色框线突出显示。这与文档中搭建Redis哨兵集群的步骤相关，文档提到在该目录下新建3个文件夹，并将课前资料提供的`sentinel.conf`文件分别拷贝一份到这3个文件夹中，此图直观呈现了文件夹的创建情况，为后续操作提供位置参考。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/O3qabMUvtoSCqFxFXQbcjCJBnth.png)

将课前资料提供的`sentinel.conf`文件分别拷贝一份到3个文件夹中。



接着修改`docker-compose.yaml`文件，内容如下：

```YAML
version: "6.2.7"

services:
  r1:
    image: redis:6.2.7
    container_name: r1
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7001"]
  r2:
    image: redis:6.2.7
    container_name: r2
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7002", "--slaveof", "192.168.101.68", "7001"]
  r3:
    image: redis:6.2.7
    container_name: r3
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7003", "--slaveof", "192.168.101.68", "7001"]
  s1:
    image: redis:6.2.7
    container_name: s1
    volumes:
      - /root/redis/s1:/etc/redis
    network_mode: "host"
    entrypoint: ["redis-sentinel", "/etc/redis/sentinel.conf", "--port", "27001"]
  s2:
    image: redis:6.2.7
    container_name: s2
    volumes:
      - /root/redis/s2:/etc/redis
    network_mode: "host"
    entrypoint: ["redis-sentinel", "/etc/redis/sentinel.conf", "--port", "27002"]
  s3:
    image: redis:6.2.7
    container_name: s3
    volumes:
      - /root/redis/s3:/etc/redis
    network_mode: "host"
    entrypoint: ["redis-sentinel", "/etc/redis/sentinel.conf", "--port", "27003"]
```



直接运行命令，启动集群：

```Shell
docker-compose up -d
```

运行结果：

![图片展示了在终端执行`docker-compose up -d`命令后，Redis哨兵集群的运行结果。命令执行后，显示“Running 6/6”，表明6个容器已启动，分别是s1、s2、r1、r2、s3、r3，状态均为Started。这与上文搭建哨兵集群时，将`sentinel.conf`文件拷贝到3个文件夹并修改`docker-compose.yaml`文件，最后运行命令启动集群的内容相呼应，直观呈现了集群启动情况。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/YnDab58kqojVt2x8S8Kcyq12nLg.png)

我们以s1节点为例，查看其运行日志：

```Bash
# Sentinel ID is 8e91bd24ea8e5eb2aee38f1cf796dcb26bb88acf
# +monitor master hmaster 192.168.101.68 7001 quorum 2
* +slave slave 192.168.101.68:7003 192.168.101.68 7003 @ hmaster 192.168.101.68 7001
* +sentinel sentinel 5bafeb97fc16a82b431c339f67b015a51dad5e4f 192.168.101.68 27002 @ hmaster 192.168.101.68 7001
* +sentinel sentinel 56546568a2f7977da36abd3d2d7324c6c3f06b8d 192.168.101.68 27003 @ hmaster 192.168.101.68 7001
* +slave slave 192.168.101.68:7002 192.168.101.68 7002 @ hmaster 192.168.101.68 7001
```

可以看到`sentinel`已经联系到了`7001`这个节点，并且与其它几个哨兵也建立了链接。哨兵信息如下：

- `27001`：`Sentinel ID`是`8e91bd24ea8e5eb2aee38f1cf796dcb26bb88acf`
- `27002`：`Sentinel ID`是`5bafeb97fc16a82b431c339f67b015a51dad5e4f`
- `27003`：`Sentinel ID`是`56546568a2f7977da36abd3d2d7324c6c3f06b8d`



### 2.3.演示failover

接下来，我们演示一下当主节点故障时，哨兵是如何完成集群故障恢复（failover）的。

停止r1

```Bash

docker stop r1
```

登录r2，查看集群状态：

![图片展示的是在r2容器中使用redis-cli命令执行`info replication`操作后的输出结果。其中关键信息是“role:slave”，表明r2当前角色为slave。这与上下文内容相关，上下文提到登录r2查看集群状态后，稍后会发现sentinel节点触发了`failover`，而此图片正是在r2上查看集群状态时的部分输出，用于说明r2当前处于slave状态，为后续主节点故障时的集群故障恢复（failover）演示做准备。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/C5EMbkbduoudjVxX7fYc6TWdnIh.png)



稍微等待一段时间后，会发现sentinel节点触发了`failover`：

继续查看集群状态发现主节点为r2

![这张图片展示了执行`info replication`命令后的输出内容，是在哨兵完成故障切换后查看集群状态的结果。图片中红框突出显示了关键信息“role:master”，表明当前节点（对应端口7002的Redis节点）已成为新的主节点，符合文档中哨兵完成故障恢复后将r2设为主节点的演示内容。输出还显示该主节点已连接1个从节点，从节点的IP、端口、运行状态、偏移量和同步滞后等信息，可验证Redis集群故障切换后的主从复制状态。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/IUIwbj7u3otdRkxLWpjc3JOqnWf.png)

## 3.Redis分片集群

主从模式可以解决高可用、高并发读的问题。但依然有两个问题没有解决：

- 海量数据存储
- 高并发写



要解决这两个问题就需要用到分片集群了。分片的意思，就是把数据拆分存储到不同节点，这样整个集群的存储数据量就更大了。

Redis分片集群的结构如图：

![图片展示了Redis分片集群的结构。图中有多个master节点，每个master节点下有slave节点，master节点间通过绿色箭头表示的心跳进行健康状态监测，slave节点间通过绿色箭头表示的数据同步。每个master节点保存不同分片数据，master节点间及master与slave节点间均存在数据同步关系。该图与上下文紧密相关，直观呈现了分片集群中master、slave节点的分布及数据同步、心跳监测等关键信息。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/JC2jbh8lFoLyTcxYawvckbsWnnh.jpg)

分片集群特征：

-  集群中有多个master，每个master保存不同分片数据 ，解决海量数据存储问题
-  每个master都可以有多个slave节点 ，确保高可用
-  master之间通过ping监测彼此健康状态 ，类似哨兵作用
-  客户端请求可以访问集群任意节点，最终都会被转发到数据所在节点 





### 3.1.搭建分片集群

Redis分片集群最少也需要3个master节点，由于我们的机器性能有限，我们只给每个master配置1个slave，形成最小的分片集群：

![图片展示了Redis分片集群中节点间的数据同步和心跳机制。图中有三个master节点和两个slave节点，master节点间通过绿色箭头表示心跳，slave节点间通过绿色箭头表示数据同步，master与slave间通过黑色箭头表示数据同步。该图与文档中3.1.1.集群配置部分内容相关，直观呈现了分片集群中Redis节点间的数据同步和心跳交互情况，帮助理解集群中节点间通信机制。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/FQIQbQtJ4oQFgExGxZvcc9YDn4R.jpg)



计划部署的节点信息如下：

<sheet sheet-id="q37cQG" token="AJkbs1A2zh1UkutBTkucdb9InKb"></sheet>



#### 3.1.1.集群配置

分片集群中的Redis节点必须开启集群模式，一般在配置文件中添加下面参数：

```Bash
port 7000
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
appendonly yes
```

其中有3个我们没见过的参数：

- `cluster-enabled`：是否开启集群模式
- `cluster-config-file`：集群模式的配置文件名称，无需手动创建，由集群自动维护
- `cluster-node-timeout`：集群中节点之间心跳超时时间



一般搭建部署集群肯定是给每个节点都配置上述参数，不过考虑到我们计划用`docker-compose`部署，因此可以直接在启动命令中指定参数，偷个懒。



在虚拟机的`/root`目录下新建一个`redis-cluster`目录，然后在其中新建一个`docker-compose.yaml`文件，内容如下：

```YAML
version: "3.2"

services:
  r1:
    image: redis
    container_name: r1
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7001", "--cluster-enabled", "yes", "--cluster-config-file", "node.conf"]
  r2:
    image: redis
    container_name: r2
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7002", "--cluster-enabled", "yes", "--cluster-config-file", "node.conf"]
  r3:
    image: redis
    container_name: r3
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7003", "--cluster-enabled", "yes", "--cluster-config-file", "node.conf"]
  r4:
    image: redis
    container_name: r4
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7004", "--cluster-enabled", "yes", "--cluster-config-file", "node.conf"]
  r5:
    image: redis
    container_name: r5
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7005", "--cluster-enabled", "yes", "--cluster-config-file", "node.conf"]
  r6:
    image: redis
    container_name: r6
    network_mode: "host"
    entrypoint: ["redis-server", "--port", "7006", "--cluster-enabled", "yes", "--cluster-config-file", "node.conf"]
```



**注意**：使用Docker部署Redis集群，network模式必须采用host



#### 3.1.2.启动集群

进入`/root/redis-cluster`目录，使用命令启动redis：

```Bash
docker-compose up -d
```

启动成功，可以通过命令查看启动进程：

```Bash
ps -ef | grep redis
# 结果：
root       4822   4743  0 14:29 ?        00:00:02 redis-server *:7002 [cluster]
root       4827   4745  0 14:29 ?        00:00:01 redis-server *:7005 [cluster]
root       4897   4778  0 14:29 ?        00:00:01 redis-server *:7004 [cluster]
root       4903   4759  0 14:29 ?        00:00:01 redis-server *:7006 [cluster]
root       4905   4775  0 14:29 ?        00:00:02 redis-server *:7001 [cluster]
root       4912   4732  0 14:29 ?        00:00:01 redis-server *:7003 [cluster]
```

可以发现每个redis节点都以cluster模式运行。不过节点与节点之间并未建立连接。



接下来，我们使用命令创建集群：

```Bash
# 进入任意节点容器
docker exec -it r1 bash
# 然后，执行命令
redis-cli --cluster create --cluster-replicas 1 \
192.168.101.68:7001 192.168.101.68:7002 192.168.101.68:7003 \
192.168.101.68:7004 192.168.101.68:7005 192.168.101.68:7006
```

命令说明：

- `redis-cli --cluster`：代表集群操作命令
- `create`：代表是创建集群
- `--cluster-replicas 1` ：指定集群中每个`master`的副本个数为1

  - 此时`节点总数 ÷ (replicas + 1)` 得到的就是`master`的数量`n`。因此节点列表中的前`n`个节点就是`master`，其它节点都是`slave`节点，随机分配到不同`master`



输入命令后控制台会弹出下面的信息：

![图片展示了在Linux系统下使用redis-cli命令启动Redis分片集群时的控制台输出信息。命令为`redis-cli --cluster create --cluster-replicas 1 192.168.150.101:7001 192.168.150.101:7002 192.168.150.101:7003 192.168.150.101:7004 192.168.150.101:7005 192.168.150.101:7006`。输出中显示了集群中各节点的槽位分配情况，如`7001`是`master`，`7002`是`master`等，并询问是否同意上述配置，红框突出显示了“yes”选项。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/TUaXbBi4Zoq9l0xSa7zcRuJ4njh.png)

这里展示了集群中`master`与`slave`节点分配情况，并询问你是否同意。节点信息如下：

- `7001`是`master`，节点`id`后6位是`da134f`
- `7002`是`master`，节点`id`后6位是`862fa0`
- `7003`是`master`，节点`id`后6位是`ad5083`
- `7004`是`slave`，节点`id`后6位是`391f8b`，认`ad5083`（7003）为`master`
- `7005`是`slave`，节点`id`后6位是`e152cd`，认`da134f`（7001）为`master`
- `7006`是`slave`，节点`id`后6位是`4a018a`，认`862fa0`（7002）为`master`



输入`yes`然后回车。会发现集群开始创建，并输出下列信息：

![图片展示的是Redis分片集群启动后，通过命令查看集群状态时的输出信息。内容显示节点配置更新，各节点被分配不同配置版本，发送CLUSTER MEET消息加入集群，等待集群加入。接着进行集群检查，各节点间进行槽位分配和复制，最终确认所有槽位配置一致，无开放槽位，槽位覆盖完整。该图片与上文通过命令查看集群状态的内容对应，直观呈现了集群状态信息。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/JdUJbpa08oJxoUx39GKcoqyznhh.png)



接着，我们可以通过命令查看集群状态：

```Bash
redis-cli -p 7001 cluster nodes
```

结果：

![图片展示了在Redis集群搭建中，通过`redis-cli -p 7001 cluster nodes`命令查看集群状态的结果。命令执行后，输出了各节点信息，包括节点ID、IP地址、端口号、角色（master或slave）、主节点ID、连接状态等。其中，`7001`节点为`master`，`7002`和`7003`节点为`master`，`7004`和`7005`节点为`slave`，各节点间状态均为`connected`。该图片与上文通过命令查看集群状态的内容对应，直观呈现了集群节点信息。](../../../../../../Attachments/development/programming/interview/wei-fu-wu-mian-shi-ti/wei-fu-wu-mian-shi-pian-xia/CtdEbd7vcom0FjxaECCcmWX7ngf.png)

### 3.2.Java客户端连接分片集群

RedisTemplate底层同样基于lettuce实现了分片集群的支持，而使用的步骤与哨兵模式基本一致，参考`2.5节`：

1）引入redis的starter依赖

引入SpringDataRedis的依赖：

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2）配置分片集群地址

与传统单点模式不同，需要配置每个redis实例的地址，如下：

```YAML
spring:
  redis:
    cluster:
      nodes:
        - 192.168.101.68:7001
        - 192.168.101.68:7002
        - 192.168.101.68:7003
        - 192.168.101.68:8001
        - 192.168.101.68:8002
        - 192.168.101.68:8003
```

3）配置读写分离

还要配置读写分离，让java客户端将写请求发送到master节点，读请求发送到slave节点。定义一个bean即可：

```Java
@Bean
public LettuceClientConfigurationBuilderCustomizer clientConfigurationBuilderCustomizer(){
    return clientConfigurationBuilder -> clientConfigurationBuilder.readFrom(ReadFrom.REPLICA_PREFERRED);
}
```

这个bean中配置的就是读写策略，包括四种：

- `MASTER`：从主节点读取
- `MASTER_PREFERRED`：优先从`master`节点读取，`master`不可用才读取`slave`
- `REPLICA`：从`slave`节点读取
- `REPLICA_PREFERRED`：优先从`slave`节点读取，所有的`slave`都不可用才读取`master`

---
> 来源：飞书技术知识库（脱敏版） · 08 面试知识库 / 微服务面试题 / 微服务面试篇（下） / Redis集群搭建 · [原文](https://ccn8v2zngwsm.feishu.cn/wiki/MQT4wExxbimVQ1k90wOcYU5Gnke)
