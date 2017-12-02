---
title: VPS创建需要做的一些事
date: 2017/07/04
toc: true
---

新建一个vps主机，一般而言需要做一些基本的事情，比如修改root密码，更改ssh默认端口之类，本文只是列举一些，仅供参考，暂时就这些，后续可能会补充。
<!--more-->

## 修改root密码

`$ passwd root`

## 安装vim及net-tools等

```sh
$ yum install vim
$ yum install net-tools
$ yum install zip unzip
$ yum install lrzsz
$ yum install gcc gcc-c++
```

## 更改ssh默认22端口

`$ vim /etc/ssh/sshd_config`

在 #Port 22 这一行下面 加入 Port 2102 然后别忘了防火墙开放2102端口

`$ vim /etc/firewalld/zones/public.xml`

然后在`<service name="ssh">`下面添加一行`<port protocol="tcp" port="2102"/>`保存退出

重启防火墙`$ systemctl restart firewalld.service`
重启sshd服务`$ service sshd restart`

## 安装besttrace

> 一款由 IPIP.NET 开发的路由追踪检测工具。目前已经涵盖了 Windows、Android、iOS、Mac平台。Best Trace 可对服务器所经过的路由进行可视化检测，并可以准确标注路由所在位置以及 AS 号。支持查询本机 IP、路由监控、批量 Ping 等。一款由 IPIP.NET 开发的路由追踪检测工具。目前已经涵盖了 Windows、Android、iOS、Mac平台。Best Trace 可对服务器所经过的路由进行可视化检测，并可以准确标注路由所在位置以及 AS 号。支持查询本机 IP、路由监控、批量 Ping 等。

官网地址 [https://www.ipip.net/download.html](https://www.ipip.net/download.html)

```sh
$ wget http://cdn.ipip.net/17mon/besttrace4linux.zip
$ unzip besttrace4linux.zip
$ chmod a+x besttrace
$ ./besttrace -q 1 目标ip
```

说明: besttrace是64位下执行，32位则执行besttrace32

## 安装virt

> VPS的虚拟化技术有很多种，常见的VPS虚拟化架构有多种：OpenVZ、Xen、Hyper-V、KVM、VMWare，各种虚拟化基础虚拟的VPS特点性能也不仅相同，为了不被JS忽悠，需要检测一下当前VPS使用的虚拟化技术是什么。
windows的VPS方便一点，在添加和删除程序那里看就行了，一般会安装虚拟化需要用到的工具，再加上Windows大家用的也多，操作起来可能也得心应手一点。
Linux的VPS可以使用一个叫做virt-what的小工具来检测一下当前的VPS使用的是哪种虚拟化技术。

virt-what官方网站:[http://people.redhat.com/~rjones/virt-what/files/](http://people.redhat.com/~rjones/virt-what/files/)

```sh
$ wget http://people.redhat.com/~rjones/virt-what/files/virt-what-1.12.tar.gz
$ tar zxvf virt-what-1.12.tar.gz
$ cd virt-what-1.12/
$ ./configure
$ make && make install
$ virt-whatvirt-what
```
