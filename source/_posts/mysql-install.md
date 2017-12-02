---
title: MySQL安装与启动Linux篇
date: 2016/10/13
toc: true
---

一般我们在linux上安装mysql有很多方法，这里介绍下如何通过下载源码包安装。
<!--more-->

## 解压
`tar -zxvf mysql-5.7.16-linux-glibc2.5-x86_64.tar.gz`

## 改名
`mv mysql-5.7.16-linux-glibc2.5-x86_64 mysql`
`cp mysql /usr/local/mysql`

可能需要修改 /etc/my.cnf文件如下
```sh
[mysqld]
datadir=/usr/local/mysql/data  #注意这里
socket=/var/lib/mysql/mysql.sock
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd

[mysqld_safe]
log-error=/var/log/mysqld.log #注意这里
pid-file=/var/run/mysqld/mysqld.pid #注意这里

[client] #注意这里
socket=/var/lib/mysql/mysql.sock #注意这里
#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
```

## 执行命令
```sh
groupadd mysql

useradd mysql -g mysql

cd /usr/local/mysql/bin

yum install libaio

./mysqld --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --initialize-insecure
```

然后注意，这里的datadir要和etc/my.conf的一致，
`vi /etc/my.conf`  修改 datadir=**/usr/local/mysql/data**

## 启动mysql
`cd /usr/local/mysql/support-files`
`./mysql.server start`

## 登录尝试
```
对于Mysql 5.7.6以后的5.7系列版本，Mysql使用mysqld --initialize或mysqld --initialize-insecure
命令来初始化数据库，后者可以不生成随机密码。但是安装Mysql时默认使用的是前一个命令，
这个命令也会生成一个随机密码。改密码保存在了Mysql的日志文件中。
```

如果用的是 --initialize 打开/var/log/mysqld.log文件，搜索字符串A temporary password is generated for root@localhost:，可以找到这个随机密码，通常这一行日志在log文件的最初几行，比较容易看到。
如果用的是 --initialize-insecure 则默认首次登录不用密码

```
cd /usr/local/mysql/bin
./mysql -uroot
```

如果出现错误Can't connect to local MySQL server through socket '/tmp/mysql.sock'  如果
则是因为client的默认socket地址为 /temp/mysql.sock
这里需要编辑 vi /etc/my.cnf  添加一行
```
[client]
socket=/var/lib/mysql/mysql.sock
```
<font color=#FF5511>这里需要注意，etc/my.conf文件里面配置的路径所有人应该为mysql组及mysql用户，即 chown -R mysql:mysql /var/lib/mysql</font>
重启mysql即可。

## 修改root密码，同时设置root可以远程连接
```
set password =password('新密码');

GRANT ALL PRIVILEGES ON *.* TO root@"%" IDENTIFIED BY "1新密码";

flush privileges;
```

## 添加mysql为服务

```
cp support-files/mysql.server /etc/init.d/mysqld
chkconfig --add mysqld
chkconfig mysqld on
```

配置文件参考（包含修改默认端口及默认字符集）

```sh
[mysqld]
port=5506
datadir=/usr/local/mysql/data
socket=/var/lib/mysql/mysql.sock
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
character-set-server=utf8
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

[client]
port=5506
socket=/var/lib/mysql/mysql.sock
default-character-set=utf8

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
```
