---
title: Tomcat的安装与配置
date: 2016/08/19
toc: true
---

tomcat安装本身不难，主要是一些配置的修改。
<!--more-->
## 解压
`tar -zxvf apache-tomcat-8.0.32.tar.gz`

## 启动

`cd apache-tomcat-8.0.32/bin/`
`./startup.sh`

## 配置

配置内存大小:
linux在catalina.sh中加入
`JAVA_OPTS="$JAVA_OPTS -Xms512m -Xmx1024m -XX:PermSize=512M -XX:MaxPermSize=512m"`
位置在**cygwin=false**这一行之前。

windows在catalina.bat中加入
`set JAVA_OPTS=%JAVA_OPTS% -server -XX:PermSize=128m -XX:MaxPermSize=512m`
位置在**rem ----- Execute The Requested Command ---------------------------------------** 之后

配置用户：
在tomcat-users.xml中加入 ：

```xml
<role rolename="manager-script"/>
<role rolename="manager-gui"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>
<user username="admin" password="Fengdai2016" roles="manager-gui,manager-status"/>
```

具体详细介绍如下 ：

```xml
在配置好Tomcat7/8后，我们往往需要访问Tomcat7/8的Manager以及Host Manager。就需要在tomcat-users.xml中配置用户角色来实现。
在地址栏输入：localhost:8080访问 Tomcat，在打开的界面中，在右上角有这样三个按钮：
　　1. Server Status
　　2. Manager App
　　3. Host Manager
　　
可是在我们配置好tomcat-users.xml后，这三个按钮往往不能都访问，要么是只能访问其中一个，或者就是两个。
出现这种问题很有可能是你在配置中，角色没有添加全，尤其是在第三个按钮的配置上，
第三个按钮的配置Tomcat7和Tomcat8的配置是不相同的。
为了实现配置让三个按钮都能访问到，我们先看下tomcat-users.xml里面的一段用户配置：

<tomcat-users>
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="tomcat" password="tomcat" roles="manager-gui"/>
<user username="admin" password="admin" roles="manager-script"/>
</tomcat-users>

<role rolename="角色名">这个是用来定义角色的，很明显rolename的属性值并不是我们随意写的。实际上，
  Tomcat已经为我们定义了4种不同的角色，也就是4个rolename，
  我们只需要使用Tomcat为我们定义的这几种角色就足够满足我们的工作需要了。

manager-gui     #允许访问html接口(即URL路径为/manager/html/*)
manager-script   #允许访问纯文本接口(即URL路径为/manager/text/*)
manager-jmx   #允许访问JMX代理接口(即URL路径为/manager/jmxproxy/*)
manager-status   #允许访问Tomcat只读状态页面(即URL路径为/manager/status/*)

　　特别需要说明的是：manager-gui、manager-script、manager-jmx均具备manager-status的权限，
也就是说，manager-gui、manager-script、manager-jmx三种角色权限无需再额外添加manager-status权限，
即可直接访问路径”/manager/status/*”。
　　
<user username="用户名" password="密码" roles="角色（可多个）"/>这个很简单，就是用来表示用户的，
其中roles对应的就是上面定义的角色，可以有多个角色，多个角色用“，”隔开即可。也可以配置多个用户。

需要访问前两个按钮，只需配置manager-*（4个按需配置）即可。
需要访问第三个按钮，需要配置 admin-gui（HTML UI接口）或admin-script（纯文本接口）。
如果都要访问，配置在一起即可。
下面根据Tomcat7和Tomcat8分别贴出用户配置：
1、Tomcat7访问Server Status、Manager App、Host Manager的配置。

声明：此配置不注重安全性，只是测试。具体根据需求可删减

<role rolename="admin"/>
<role rolename="admin-gui"/>
<role rolename="admin-script"/>
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>
<user username="admin" password="admin" roles="admin,admin-gui,admin-script,manager-gui,manager-script,manager-jmx,manager-status"/>

2、Tomcat8访问Server Status、Manager App、Host Manager的配置。

声明：此配置不注重安全性，只是测试。具体根据需求可删减
Tomcat8如果在上面配置的基础上，访问时会报403错误，所以需要修改，如果没有的话新建conf/Catalina/localhost/manager.xml 文件。
配置内容如下：

<Context privileged="true" antiResourceLocking="false"
         docBase="${catalina.home}/webapps/manager">
    <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />
</Context>

以上经过亲测，如果配置好后还提示404或403错误，欢迎留言！
```

上述参考地址:[地址](http://blog.csdn.net/weixian52034/article/details/53218584)

## 修改deploy文件大小限制

`cd webapps/manager/WEB-INF/`
`vi web.xml`

大概在50行出修改：

```xml
<multipart-config>
    <!-- 50MB max -->
    <max-file-size>52428800</max-file-size>
    <max-request-size>52428800</max-request-size>
    <file-size-threshold>0</file-size-threshold>
</multipart-config>
```
