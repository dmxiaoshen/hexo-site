---
title: Linux升级默认python版本2.6到2.7
date: 2016/11/05
toc: true
---

老的linux发行版内置的Python版本还是旧的2.6，但是现在有些新的软件依赖最低支持Python2.7版本，因此需要升级。
<!--more-->

## 查看python版本
`python -V`
注意大写

## 下载Python2.7
`wget http://python.org/ftp/python/2.7.3/Python-2.7.3.tar.bz2`
这时候有可能出现:
 certificate common name “*.python.org” doesn’t match requested host name “python.org”.
不慌，执行改为：
`wget http://python.org/ftp/python/2.7.3/Python-2.7.3.tar.bz2 --no-check-certificate`

### 解压
`tar -jxvf Python-2.7.3.tar.bz2`

### 安装

```
$ ./configure
$ make all
$ make install
$ make clean
$ make distclean
```

### 查看版本信息
`$ /usr/local/bin/python2.7 -V`

### 建立软连接

使系统默认的 python指向 python2.7

```
$ mv /usr/bin/python /usr/bin/python2.6.6
$ ln -s /usr/local/bin/python2.7 /usr/bin/python
```

### 重新检验Python 版本
`#python -V`

**解决系统 Python 软链接指向 Python2.7 版本后，因为yum是不兼容 Python 2.7的，所以yum不能正常工作，我们需要指定 yum 的Python版本**

`$ vi /usr/bin/yum`

将文件头部的
`#!/usr/bin/python`

改成
`#!/usr/bin/python2.6.6`

## 安装pip
```
$ wget https://bootstrap.pypa.io/get-pip.py
$ python get-pip.py
```

可能会出现 zipimport.ZipImportError: can’t decompress data; zlib not available

> 这个时候解决方法如下：
> 1、安装依赖zlib、zlib-devel
> 2、重新编译安装Python
> ./configure
> 编辑Modules/Setup.dist
> 找到下面这句，去掉注释
> `#zlib zlibmodule.c -I(prefix)/include−L(exec_prefix)/lib -lz `
> 重新编译安装
> 4.有可能会出现错误：ImportError: cannot import name HTTPSHandler
> 解决方法如下：需要在安装python前，安装OpenSSl。
> ubuntu下：apt-get install libssi-dev
> 重新编译安装python


如果出现错误 可能需要安装openssl,centos中执行 `yum install openssl-devel`

**重新编译安装python**

选择性安装
```
$ wget https://bootstrap.pypa.io/ez_setup.py
$ python ez_setup.py
```

`pip -V` 查看安装版本.
