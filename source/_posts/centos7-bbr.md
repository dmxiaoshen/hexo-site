---
title: Centos7开启bbr加速
date: 2017/07/18
toc: true
author: 故里
---

此文完全是搬运而来，因为原文写的太棒了，根本不需要任何整理修改，如果喜欢，也可以直接访问[原文](https://ofcss.com/2016/12/12/bbr-congestion-control-algorithm-for-centos-7.html)
<!--more-->

## bbr算法由来

> Google 在 2016年9月份开源了他们的优化网络拥堵算法BBR，最新版本的 Linux内核（4.9-rc8）中已经集成了该算法。
>
> 对于TCP单边加速，并非所有人都很熟悉，不过有另外一个大名鼎鼎的商业软件“锐速”，相信很多人都清楚。特别是对于使用国外服务器或者VPS的人来说，效果更佳。
>
> 网上有很多在 Debian 和 Ubuntu 系统下启用 BBR 的教程，我就不粘贴了，我自己一直用的是 CentOS，本文介绍一下在 64位 CentOS 7 系统下开启BBR的方法。


## 升级内核

第一步首先是升级内核到支持BBR的版本：

```sh
#导入公钥
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

CentOS 7
```sh
#安装elrepo源
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
```
CentOS 6
```sh
#安装elrepo源
rpm -Uvh http://www.elrepo.org/elrepo-release-6-6.el6.elrepo.noarch.rpm
```
CentOS 6 & 7
```
#安装4.9.0以上的内核
yum --enablerepo=elrepo-kernel install kernel-ml -y
```

##  确认是否成功安装新版内核
`rpm -qa | grep kernel`
如果安装成功，你应该会看到 kernel-ml-4.*.*-*.el7.elrepo.x86_64 这样的条目：

```
kernel-tools-3.10.0-514.el7.x86_64
kernel-ml-4.12.4-1.el7.elrepo.x86_64
kernel-tools-libs-3.10.0-514.el7.x86_64
kernel-3.10.0-514.el7.x86_64
```
调整GRUB启动顺序

在安装好新版本内核以后，要先用新安装的内核引导系统看看能否正常启动，下面是直接调整 GRUB2 启动顺序的命令：

```sh
# 查看可用的启动项
egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
```
执行完这条命令以后，能看到多个可以引导的系统，比如我的是：

```
CentOS Linux (4.12.4-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (3.10.0-514.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-d4d0adfea8e944e5b8019ed1aa3c9e16) 7 (Core)
```
不管有多少个，从上往下，记住要引导的项的序号（从0开始计数）即可，比如上面的例子，我要使用第一项 CentOS Linux (4.12.4-1.el7.elrepo.x86_64) 7 (Core) 来引导，序号是 0。

## 设置默认引导项
`grub2-set-default 0`

## 重启系统
`reboot`

修改sysctl 开启 BBR

重启系统之后，通过 uname -r 或者其它命令可以看到我们的内核已经是新版内核了，接下来开启 BBR

```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```

```sh
# 加载 /etc/sysctl.conf 文件中的参数并显示，主要看看有没有报错的设置（显示的结果与你的配置文件内容有关）
sysctl -p

# 验证 bbr 是否开启，如果成功，应该会看到 net.ipv4.tcp_congestion_control = bbr
sysctl net.ipv4.tcp_available_congestion_control

# 依然是验证，如果成功，应该会看到类似 tcp_bbr    16384    3 这样的文字
lsmod | grep bbr
```
以上每一步最好都根据注释中的说明仔细检查一下是否顺利，然后再进行下一步，如果都成功的话，到这里已经成功开启BBR算法。可以在你的服务器上放一个大文件，然后用浏览器下载一下看看速度是否有提升。

> 我的洛杉矶VPS加速以后，用Chrome下载的速度从500K/s左右提升到了3.3M/s左右。
