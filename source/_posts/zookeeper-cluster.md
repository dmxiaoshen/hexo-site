---
title: Zookeeper集群配置
date: 2017/03/19
toc: true
---

zk集群是分布式系统比较基础的，而且其实也不难，主要是细心，这里详细记录一下。
<!--more-->

## 修改zoo.cfg文件

以3台zookeeper集群为例

以下操作针对3个zk,每个zk都需要如此操作

假定zk路径都为: /usr/local/zookeeper-3.4.10

首先新建一个data目录

```
$ cd /usr/local/zookeeper-3.4.10
$ mkdir data
```

修改zoo.cfg文件，其中

> dataDir=/tmp/zookeeper

修改为

> dataDir=/usr/local/zookeeper-3.4.10/data

注意: 如果是单台机器部署多个(伪集群)，则clientPort需修改为不同端口

末尾添加如下配置

```
server.1=192.168.0.1:2888:3888
server.2=192.168.0.2:2888:3888
server.3=192.168.0.3:2888:3888
```

注意: 这里ip地址根据实际情况配置，也可以配置为host映射。如果是单台机器部署多个(伪集群)，则其中端口不能重复(2888,3888)

## 创建myid文件

以下操作针对3个zk,每个zk都需要如此操作

在刚才配置的

```dataDir=/usr/local/zookeeper-3.4.10/data```

目录下创建myid文件，内容为各自ip对应的server.X,也就是上述配置的1，2，3


| zk地址      | 文件 | 内容 |
| ----------- | ---- | ---- |
| 192.168.0.1 | myid | 1    |
| 192.168.0.2 | myid | 2    |
| 192.168.0.3 | myid | 3    |

## 查看集群启动状态

依次启动每一个zk

`$ ./zkServer.sh start`


执行如下命令查看状态

`$ ./zkServer.sh status`


输出如下:

> ZooKeeper JMX enabled by default
> Using config: /usr/local/zookeeper-3.4.10/bin/../conf/zoo.cfg
> Mode: leader

或者:

> ZooKeeper JMX enabled by default
> Using config: /usr/local/zookeeper-3.4.10/bin/../conf/zoo.cfg
> Mode: follower
