---
title: Redis集群快速搭建
date: 2019-02-19 13:14:36
tags: 
 - redis
---

> 我们知道，对于大型应用而言，如何保证线上功能稳定是一件很重要的事情；大多数的web应用都会用到缓存来降低数据库的压力，而单机缓存服务带来的问题是，一旦单机机器挂掉，所有的应用将无法使用。因此就需要有多台机器来保证服务的稳定，这里就涉及到了集群的概念。下面以redis5.0为实例进行快速搭建一个简单的集群。

<!-- more -->

##### 环境

- Centos7
- Redis5.0.x

##### Redis安装

1. [下载](https://redis.io/download)，我们这里使用5.0.3的stable版本

```shell
wget http://download.redis.io/releases/redis-5.0.3.tar.gz
```

2. 解压

```shell
tar -zxvf redis-5.0.5.tar.gz
```

3. 安装

```shell
cd redis-5.0.5
make && make install
```

至此，我们的redis已经安装完成。

##### 集群搭建

- 基本配置

在根目录下新建文件夹```redis-cluster```，复制redis-5.0.3 src下的```redis.conf```到```redis-cluster```分别命名为```master.conf```和```slave.conf```。分别修改这两个文件注释掉bind.

```shell
# bind 127.0.0.1
```

修改slave.conf文件，修改端口号，增加配置slaveof.

```shell
# 从节点这里我们使用6380端口
port 7002
# 配置主节点的ip和端口号，这里我们使用本地的，根据情况如果使用其他机器作为主节点，可修改对应配置
slaveof 127.0.0.1 6379
```

使用```redis-server```分别指定对应的配置文件启动，使用```info replication```可查看对应信息。

6379:

```shell
➜ redis-cluster redis-cli -h 127.0.0.1 -p 6379 
127.0.0.1:6379> info replication 
# Replication 
role:master 
connected_slaves:1 
slave0:ip=127.0.0.1,port=6380,state=online,offset=113,lag=0 
master_repl_offset:113 
repl_backlog_active:1 
repl_backlog_size:1048576 
repl_backlog_first_byte_offset:2 
repl_backlog_histlen:112
```

6380:

```shell
➜ redis-cluster redis-cli -h 127.0.0.1 -p 7002 
127.0.0.1:6380> info replication 
# Replication 
role:slave 
master_host:127.0.0.1 
master_port:6379 
master_link_status:up 
master_last_io_seconds_ago:2 
master_sync_in_progress:0 
slave_repl_offset:99 
slave_priority:100 
slave_read_only:1 
connected_slaves:0 
master_repl_offset:0 
repl_backlog_active:0 
repl_backlog_size:1048576 
repl_backlog_first_byte_offset:0 
repl_backlog_histlen:0
```

- 哨兵

至此，一个简单的集群就搭建成功了，但是有个弊端是一旦master挂掉之后，无法生成新的master节点，导致写操作就会失效。因此，我们需要在master挂掉之后会自动启动一个slave节点来代替master节点，使服务仍然可用，这里就用到了哨兵。

哨兵的作用不言而喻，就是用来监控master节点的状态，并生成新的master节点。

复制一份```redis-5.0.3/src```下的```redis-sentinel.conf```到```redis-cluster```下，命名为```redis-sentinel-7002.conf```，并修改如下配置。

```shell
sentinel monitor mymaster 127.0.0.1 7002 2 # 哨兵监听的主节点的ip和端口号，2表示需要的哨兵数 
# Default is 30 seconds. 
sentinel down-after-milliseconds mymaster 30000 # 超时时间，表示3s内没有响应则认为主节点down掉 
# Default is 3 minutes. 
sentinel failover-timeout mymaster 180000 # 表示18s后，master节点仍然没有恢复，则重新生成新的master节点
```

配置完成后使用如下命令来启动sentinel:

```shell
redis-sentinel /etc/redis-cluster/redis-sentinel-7002.conf
```

我们可以模拟让```master```节点挂掉后看是否```7002```端口是否会变成新的```master```节点。手动kill掉```master```，然后使用`info replication`再次查看```7002```的信息。

```shell
➜ redis-cluster redis-cli -h 127.0.0.1 -p 7002 
127.0.0.1:7002> info replication 
# Replication 
role:master 
connected_slaves:0 
master_repl_offset:29 
repl_backlog_active:1 
repl_backlog_size:1048576 
repl_backlog_first_byte_offset:2 
repl_backlog_histlen:28
```

- redis-cluster

哨兵模式的缺点对数据量有限制，受限于机器的内存的最小节点，因此这还可以使用数据分片的方式来实现存储，即```redis-cluster```。

修改配置，注释掉```slave.conf```的```slaveof```.

```shell
# slaveof 127.0.0.1 6379
```

分别修改```6379```和```7002```修改以下配置：

```shell
cluster-enabled yes 
# Every cluster node has a cluster configuration file. This file is not 
# intended to be edited by hand. It is created and updated by Redis nodes. 
# Every Redis Cluster node requires a different cluster configuration file. 
# Make sure that instances running in the same system do not have 
# overlapping cluster configuration file names. 
cluster-config-file nodes-6379.conf # 各自对应 
# Cluster node timeout is the amount of milliseconds a node must be unreachable 
# for it to be considered in failure state. 
# Most other internal time limits are multiple of the node timeout. 
cluster-node-timeout 15000
```

我们复制多个配置文案命名为```redis-7002.conf ~ redis-7005.conf```；使用如下命令创建集群:

```shell
➜ redis-cluster redis-cli --cluster create 127.0.0.1:6379 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 --cluster-replicas 1 
>>> Performing hash slots allocation on 6 nodes... 
Master[0] -> Slots 0 - 5460 
Master[1] -> Slots 5461 - 10922 
Master[2] -> Slots 10923 - 16383 
Adding replica 127.0.0.1:7004 to 127.0.0.1:6379 
Adding replica 127.0.0.1:7005 to 127.0.0.1:7002 
Adding replica 127.0.0.1:7006 to 127.0.0.1:7003 
>>> Trying to optimize slaves allocation for anti-affinity 
[WARNING] Some slaves are in the same host as their master 
M: 21aa1be9fdf18ba229646d31160c771c39298d29 127.0.0.1:6379 
   slots:[0-5460] (5461 slots) master 
M: a2a10c4d6d9e691cb48890f3b9c3d1793613b1ce 127.0.0.1:7002 
   slots:[5461-10922] (5462 slots) master 
M: 3a0b64b91fed7f9f096ffd7c98481d03be964a10 127.0.0.1:7003 
   slots:[10923-16383] (5461 slots) master 
S: aafc91d92f75c5b1572b80ccca49e089bf377dc5 127.0.0.1:7004 
   replicates 3a0b64b91fed7f9f096ffd7c98481d03be964a10 
S: 0a7021591a5498f514005410eff25394d28d5f60 127.0.0.1:7005 
   replicates 21aa1be9fdf18ba229646d31160c771c39298d29 
S: 243d39cc830e701dd314f0d2a9273310c6f04abb 127.0.0.1:7006 
   replicates a2a10c4d6d9e691cb48890f3b9c3d1793613b1ce 
Can I set the above configuration? (type 'yes' to accept):
```

其中—```cluster-replicas 1```表示：一主一从配置，6个节点就是3主3从。

各个节点已经分配好，输入```yes```，打印如下信息说明集群创建成功。

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/20190219161228.png)

如果我们想要删除或者添加节点，可以使用`redis-cli --cluster add-node`和`redis-cli --cluster del-node`命令。

比如我们要新增一个节点```127.0.0.1:7007```，挂载在```6379```节点上。

```shell
redis-cli --cluster add-node 127.0.0.1:7007 127.0.0.1:6379
```

删除该节点，先查出节点的id.

```shell
redis-cli --cluster check 127.0.0.1:7007
```

输出如下信息：

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/20190219164232.png)

找到对应节点id，这里是ef1035f7b8efe2b42cd1ac1c52833aca45edb498，执行以下命令即刻删除。

```shell
redis-cli --cluster del-node 127.0.0.1:7007 ef1035f7b8efe2b42cd1ac1c52833aca45edb498
```