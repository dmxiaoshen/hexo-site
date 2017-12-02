---
title: Redis的安装与启动
date: 2017/02/22
toc: true
---

redis的安装还是比较简单的，只不过有一些配置什么的需要注意，还有如果能设置为服务启动最好。
<!--more-->
## 解压文件
`tar -zxvf redis-3.2.5.tar.gz`

## 编译
```
cd redis-3.2.5
make
```
编译完成后，在./Src目录下，有三个可执行文件redis-server、redis-benchmark、redis-cli和./redis.conf然后拷贝到一个目录下。

## 修改配置
`vi redis.conf`
注意几个点:
1. bind 127.0.0.1 注释掉该句
2. 顺便在下面一行添加 requirepass 123456
3. daemonize no 改为 daemonize yes

## 启动
进入src目录
`redis-server ../redis.conf`

测试是否启动成功
```sh
redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```

## 设置为服务
```sh
vi /etc/rc.d/init.d/redisd
chmod a+x /etc/rc.d/init.d/redisd
cp /etc/rc.d/init.d/redisd /etc/init.d/redisd
chmod a+x /etc/init.d/redisd
chkconfig --add redisd
chkconfig redisd on
chkconfig --list redisd
```

脚本内容如下:
```sh
#!/bin/sh
#chkconfig: 345 86 14
#description: Startup and shutdown script for Redis

REDISPORT=6379
PROGDIR=/usr/local/redis-3.2.5/src
PROGNAME=redis-server
DAEMON=$PROGDIR/$PROGNAME
CONFIG=/usr/local/redis-3.2.5/redis.conf
PIDFILE=/var/run/redis_${REDISPORT}.pid
DESC="redis daemon"
SCRIPTNAME=/etc/init.d/redisd

start()
{
         if test -x $DAEMON
         then
        echo -e "Starting $DESC: $PROGNAME"
                   if $DAEMON $CONFIG
                   then
                            echo -e "OK"
                   else
                            echo -e "failed"
                   fi
         else
                   echo -e "Couldn't find Redis Server ($DAEMON)"
         fi
}

stop()
{
         if test -e $PIDFILE
         then
                   echo -e "Stopping $DESC: $PROGNAME"
                   if kill `cat $PIDFILE`
                   then
                            echo -e "OK"
                   else
                            echo -e "failed"
                   fi
         else
                   echo -e "No Redis Server ($DAEMON) running"
         fi
}

restart()
{
    echo -e "Restarting $DESC: $PROGNAME"
    stop
         start
}

list()
{
         ps aux | grep $PROGNAME
}

case $1 in
         start)
                   start
        ;;
         stop)
        stop
        ;;
         restart)
        restart
        ;;
         list)
        list
        ;;

         *)
        echo "Usage: $SCRIPTNAME {start|stop|restart|list}" >&2
        exit 1
        ;;
esac
exit 0
```

```sh
service redisd start
service redisd stop
service redisd restart
service redisd list
```

如果遇到No such file or directory问题。

```
解决方法
分析原因，可能因为我平台迁移碰到权限问题我们来进行权限转换


1）在Windows下转换：

利用一些编辑器如UltraEdit或EditPlus等工具先将脚本编码转换，再放到Linux中执行。转换方式如下（UltraEdit）：File-->Conversions-->DOS->UNIX即可。


2)方法

用vim打开该sh文件，输入：
[plain]
:set ff
回车，显示fileformat=dos，重新设置下文件格式：
[plain]
:set ff=unix
保存退出:
[plain]
:wq
再执行，竟然可以了

3）在linux中的权限转换


也可在Linux中转换：

首先要确保文件有可执行权限

#chmod u+x filename

然后修改文件格式

#vi filename

三种方法都可以方便快速的解决关于Linux执行.sh文件，提示No such file or directory这个问题了。
```

如果要清理缓存
```sh
redis-cli -p 6379
auth 123456
flushall
```
