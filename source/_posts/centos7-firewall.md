---
title: Centos7防火墙
date: 2017/06/18
toc: true
---

centos从6到7，防火墙由原先的iptables变为了firewalld，不熟悉的话操作就很不习惯，这里记录下firewalld的一些常用操作。
<!--more-->

## 系统命令

```sh
# 启动
$ systemctl start firewalld.service
# 关闭
$ systemctl stop firewalld.service
# 设置开机启动
$ systemctl enable firewalld.service
# 关闭开机启动
$ systemctl disable firewalld.service
# 查看状态
$ systemctl status firewalld
```

## firewall-cmd 命令


```sh
# 查看状态
$ firewall-cmd --state //running 表示运行
# 重新加载防火墙(一般用于修改配置后)
$ firewall-cmd --reload
# 开启某个端口
$ firewall-cmd --permanent --zone=public --add-port=8080-8081/tcp //永久
$ firewall-cmd --zone=public --add-port=8080-8081/tcp //临时
# 设置某个ip 访问某个服务
$ firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.0.4/24" service name="http" accept"
# 查看开启的服务和端口
$ firewall-cmd --permanent --zone=public --list-services //服务空格隔开 例如 dhcpv6-client https ss
$ firewall-cmd --permanent --zone=public --list-ports //端口空格隔开 例如 8080-8081/tcp 8388/tcp 80/tcp
# 查看防火墙当前配置
$ firewall-cmd --list-all
```

设置某个ip 访问某个服务：执行命令的效果相当于在配置文件(/etc/firewalld/zones/public.xml)中加入：

```xml
<rule family="ipv4">
 <source address="192.168.0.4/24"/>
 <service name="http"/>
 <accept/>
</rule>
```

设置某个ip 访问某个服务：执行命令的效果相当于在配置文件(/etc/firewalld/zones/public.xml)中加入：

```xml
<port protocol="tcp" port="8080"/>
```

## 关闭firewall防火墙，使用iptables防火墙

```sh
1、直接关闭防火墙

systemctl stop firewalld.service #停止firewall

systemctl disable firewalld.service #禁止firewall开机启动

2、设置 iptables service

yum -y install iptables-services

如果要修改防火墙配置，如增加防火墙端口3306

vi /etc/sysconfig/iptables

增加规则

-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT

保存退出后

systemctl restart iptables.service #重启防火墙使配置生效

systemctl enable iptables.service #设置防火墙开机启动
```
最后重启系统使设置生效即可
