---
title: Zookeeper日志指定输出
date: 2017/03/15
toc: true
---

zk的安装启动是比较容易的，但是网上很少有说明zk的日志该如何保存以及设置（有也是很零碎或者不简洁),这里结合网上的教程及自身实践，给出完整方案。
<!--more-->

## 重命名zoo_sample.cfg为zoo.cfg

假设zk路径为: /usr/local/zookeeper-3.4.10

```
$ cd /usr/local/zookeeper-3.4.10/conf
$ mv zoo_sample.cfg zoo.cfg
```

## 修改zk的log4j.properties文件

`$ cd /usr/local/zookeeper-3.4.10/conf`

修改log4j.properties首行的

> zookeeper.root.logger=INFO, CONSOLE

修改为

> zookeeper.root.logger=INFO, ROLLINGFILE

这样就可以了，如果想要日志按天保存文件，则

```
log4j.appender.ROLLINGFILE=org.apache.log4j.RollingFileAppender
修改为
log4j.appender.ROLLINGFILE=org.apache.log4j.DailyRollingFileAppender
```

## 修改zkEnv.sh文件

```
$ cd /usr/local/zookeeper-3.4.10/
$ mkdir logs
$ cd bin
```

首先建立logs目录，用于存放日志
修改zkEnv.sh文件

> ZOO_LOG_DIR="/usr/local/zookeeper-3.4.10/logs"
> ZOO_LOG4J_PROP="INFO,ROLLINGFILE"

这里的ZOO_LOG4J_PROP必须和第2步的zookeeper.root.logger一致。
