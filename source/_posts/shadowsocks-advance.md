---
title: Shadowsocks优化篇
date: 2017/07/17
toc: true
---

个人感觉，优化和不优化差别还是很大的，简直就是两个软件，当然本文是参考网络上的教程，非原创，仅供参考。
<!--more-->

## 首先要确认vps主机属于何种架构

> 我们知道VPS的虚拟技术有许多种，如Openvz、Xen、VMware vSphere、Hyper-V、KVM及Xen的HVM与PV等。在Xen中pv是半虚拟化，hvm是全虚拟化，pv只能用于linux内核的系统，hvm可以虚拟所有常见操作系统(Linux+windows)，理论效率比pv略低，另外hvm需要cpu虚拟化指令支持，pv无此要求。而Openvz是一个类似于Linux-VServer的操作系级全虚拟化解决方案，目前基于Xen和Openvz的VPS服务商比较多。
那么如何判断买到的是哪种虚拟技术的VPS呢？价格上，Openvz一般比Xen便宜得多，但稳定性和用途范围就不及Xen和Kvm了。
1、通过系统目录判断
执行命令：ls -al /proc
一般Openvz的话，则会有vz目录，Xen的话则会有xen目录。
2、通过网卡信息判断
执行命令：ifconfig
一般Openvz的话，则会有venet0或venet0:x网卡标识，Xen的话一般则是eth0。
3、通过VPS控制面板判断
流行的VPS面板包括SolusVM、vePortal等，会显示具体的虚拟技术。
4、通过virt-what命令判断
CentOS或RedHat系统的话，执行命令：yum install -y virt-what
ubuntu（debian系）：sudo apt-get install virt-what
virt-what是一个判断当前环境所使用的虚拟技术的脚本，常见的虚拟技术基本上都能正常识别出来。
安装好virt-what后，执行命令：sudo virt-what
根据返回的信息，即可判断出当前VPS所使用的虚拟技术。
腾讯云、UCLOUD云、青云都是基于KVM的，阿里后面的也转成KVM架构了，Linode也转成KVM了（注明：非原创，转载于互联网，有删减）

### virt也可以编译安装

> 可以执行如下命令安装(需要安装好gcc、make)：
wget http://people.redhat.com/~rjones/virt-what/files/virt-what-1.12.tar.gz
tar zxvf virt-what-1.12.tar.gz
cd virt-what-1.12/
./configure
make && make install
再运行 virt-what ，脚本就会判断出当前环境所使用的虚拟技术

## 优化

### 基于kvm的系统优化

基于kvm架构vps的优化
这方面SS给出了非常详尽的优化指南，主要有：优化内核参数，开启TCP Fast Open
#### 优化内核参数
编辑vi /etc/sysctl.conf，复制进去

```sh
# max open files
fs.file-max = 1024000
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096

# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1

# for high-latency network
net.ipv4.tcp_congestion_control = hybla
# forward ipv4
net.ipv4.ip_forward = 1
```

保存生效
sysctl -p
其中最后的hybla是为高延迟网络（如美国，欧洲）准备的算法，需要内核支持，测试内核是否支持，在终端输入：
sysctl net.ipv4.tcp_available_congestion_control
如果结果中有hybla，则证明你的内核已开启hybla，如果没有hybla，可以用命令modprobe tcp_hybla开启。

对于低延迟的网络（如日本，香港等），可以使用htcp，可以非常显著的提高速度，首先使用modprobe tcp_htcp开启，再将net.ipv4.tcp_congestion_control = hybla改为net.ipv4.tcp_congestion_control = htcp，建议EC2日本用户使用这个算法。

#### TCP优化

1.修改文件句柄数限制
如果是ubuntu/centos均可修改/etc/sysctl.conf
找到fs.file-max这一行，修改其值为1024000，并保存退出。然后执行sysctl -p使其生效
修改`vi /etc/security/limits.conf`文件，加入

```
*               soft    nofile           512000
*               hard    nofile          1024000
```

针对centos,还需要修改vi /etc/pam.d/common-session文件，加入
`session required pam_limits.so`

2.修改vi /etc/profile文件，加入
`ulimit -SHn 1024000`
然后重启服务器执行`ulimit -n`，查询返回1024000即可。

> sysctl.conf报错解决方法
修复modprobe的：
rm -f /sbin/modprobe
ln -s /bin/true /sbin/modprobe
修复sysctl的：
rm -f /sbin/sysctl
ln -s /bin/true /sbin/sysctl

#### 开启TCP Fast Open

这个需要服务器和客户端都是Linux 3.7+的内核，一般Linux的服务器发行版只有debian jessie有3.7+的，客户端用Linux更是珍稀动物，所以这个不多说，如果你的服务器端和客户端都是Linux 3.7+的内核，那就在服务端和客户端的vi /etc/sysctl.conf文件中再加上一行。
```sh
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
```
然后把vi /etc/shadowsocks.json配置文件中"fast_open": false改为"fast_open": true。这样速度也将会有非常显著的提升。

### 加密层面

#### 安装M2Crypto
这个可以提高SS的加密速度，安装办法：
Debian/Ubuntu
`apt-get install python-m2crypto`
安装之后重启SS，速度将会有一定的提升

CentOS
先安装依赖包：

```sh
yum install -y openssl-devel gcc swig python-devel autoconf libtool
yum install -y python-pip
```

再通过pip安装M2Crypto：
`pip install M2Crypto` 或者`pip install M2Crypto --upgrade`

#### 安装 gevent

安装 gevent可以提高 Shadowsocks 的性能。
Debian/Ubuntu
```sh
apt-get install python-dev libevent-dev python-setuptools python-gevent
easy_install greenlet gevent
```

CentOS

```sh
yum install -y libevent
pip install greenlet
pip install gevent
```

#### 使用CHACHA20加密算法

首先，安装libsodium，让系统支持chacha20算法。
Debian/Ubuntu

```sh
apt-get install build-essential
wget https://download.libsodium.org/libsodium/releases/LATEST.tar.gz
tar xf LATEST.tar.gz && cd libsodium*
./configure && make && make install
ldconfig
```

CentOS

```sh
yum groupinstall "Development Tools"
wget https://download.libsodium.org/libsodium/releases/LATEST.tar.gz
tar zxf LATEST.tar.gz
cd libsodium*
./configure
make
make install
```

`vi /etc/ld.so.conf`
添加一行：
`/usr/local/lib`
保存退出后，运行命令：
`ldconfig`

然后修改ss加密方式：
vi /etc/shadowsocks.json "method":"aes-256-cfb"改成"method":"chacha20",
重启SS即可`/etc/init.d/shadowsocks restart`或者`ssserver -c /etc/shadowsocks.json -d restart`

### 网络层面

此外，选择合适的端口也能优化梯子的速度，广大SS用户的实践经验表明，检查站（GFW）存在一种机制来降低自身的运算压力，即常用的协议端口（如http，smtp，ssh，https，ftp等）的检查较少，所以建议SS绑定这些常用的端口（如：21，22，25，80，443），速度也会有显著提升。
如果你还要给小伙伴爬，那我建议开启多个端口而不是共用，这样网络会更加顺畅。

#### 防火墙设置（如有）

自动调整MTU
`iptables -I FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu`

开启 NAT （记得把 eth0 改成自己的网卡名，openvz 的基本是 venet0 ）
`iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`

开启 IPv4 的转发
`sysctl -w net.ipv4.ip_forward=1`

打开 443 端口

```sh
iptables -I INPUT -p tcp --dport 443 -j ACCEPT
iptables -I INPUT -p udp --dport 443 -j ACCEPT
```

重启防火墙iptables：
`service iptables restart`

## 使用感受
首先肯定要挑选一个网络跳转少的vps主机，不一定日本，新加坡就好，美西也不一定比美东好，可以用[besttrace](https://www.ipip.net/download.html)软件测试下网络路由，主要看是否直连或者跳了几层。然后才是优化，优化以后效果还是很明显的。
但是最重要的，或许是开启**bbr**加速，这将在下一篇文章介绍。

[参考链接](https://github.com/iMeiji/shadowsocks_install/wiki/shadowsocks-optimize)
