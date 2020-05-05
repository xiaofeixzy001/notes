[TOC]

# Tomcat简介

Tomcat是Apache软件基金会（Apache Software Foundation）的Jakarta 项目中的一个核心项目，由Apache、Sun和其他一些公司及个人共同开发而成。

Tomcat服务器是一个免费的开放源代码的**Web应用服务器**，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP程序的首选。

Tomcat和Nginx、Apache(httpd)、lighttpd等Web服务器一样，具有处理HTML页面的功能，另外它还是一个Servlet和JSP容器，独立的Servlet容器是Tomcat的默认模式。不过，Tomcat处理静态HTML的能力不如Nginx/Apache服务器。

目前Tomcat最新版本为9.0。Java容器还有resin、weblogic等。

# Tomcat安装

## 部署JDK

JDK下载：http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

目前下载需要登录帐号，可以注册一下。

```shell
[root@tomcat ~]# mkdir /application/tools
[root@tomcat ~]# cd /application/tools/
[root@tomcat tools]# tar xf jdk-8u241-linux-x64.tar.gz -C /application/
[root@tomcat tools]# tar xf apache-tomcat-8.5.51.tar.gz -C /application/
[root@tomcat tools]# cd ..
[root@tomcat application]# ln -sv /application/jdk1.8.0_241 /application/jdk
[root@tomcat application]# vim /etc/profile
'''
# jdk env
export JAVA_HOME=/application/jdk
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar
'''

[root@tomcat application]# source  /etc/profile && java -version
```



## 安装tomcat

 Tomcat下载：http://tomcat.apache.org/

```shell
[root@tomcat application]# ln -sv /application/apache-tomcat-8.5.51/ /application/tomcat
```



## tomcat目录

```shell
[root@tomcat ~]# cd /application/tomcat/
[root@tomcat tomcat]# tree -L 1
"""
.
├── bin # 用以启动、关闭Tomcat或者其它功能的脚本（.bat文件和.sh文件）
├── conf # 用以配置Tomcat的XML及DTD文件
├── lib # 存放web应用能访问的JAR包
├── LICENSE
├── logs # Catalina和其它Web应用程序的日志文件
├── NOTICE
├── RELEASE-NOTES
├── RUNNING.txt
├── temp # 临时文件
├── webapps # Web应用程序根目录
└── work # 用以产生有JSP编译出的Servlet的.java和.class文件
7 directories, 4 files
"""

[root@tomcat tomcat]# cd webapps/
[root@tomcat webapps]# ll
"""
total 20
drwxr-xr-x 14 root root 4096 Oct 5 12:09 docs # tomcat帮助文档
drwxr-xr-x 6 root root 4096 Oct 5 12:09 examples # web应用实例
drwxr-xr-x 5 root root 4096 Oct 5 12:09 host-manager # 管理
drwxr-xr-x 5 root root 4096 Oct 5 12:09 manager # 管理
drwxr-xr-x 3 root root 4096 Oct 5 12:09 ROOT # 默认网站根目录
"""
```



## 服务启动

启动程序/application/tomcat/bin/startup.sh
关闭程序/application/tomcat/bin/shutdown.sh

```shell
[root@tomcat ~]# /application/tomcat/bin/startup.sh
[root@tomcat ~]# netstat -tunlp | grep java
"""
tcp 0 0 :::8009 :::* LISTEN 4743/java
tcp 0 0 :::8080 :::* LISTEN 4743/java
"""

[root@tomcat ~]# ps -ef|grep [j]ava
"""
root 4743 1 17 06:10 pts/0 00:00:03 /application/jdk/bin/java -Djava.util.logging.config.file=/application/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djava.endorsed.dirs=/application/tomcat/endorsed -classpath /application/tomcat/bin/bootstrap.jar:/application/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/application/tomcat -Dcatalina.home=/application/tomcat -Djava.io.tmpdir=/application/tomcat/temp org.apache.catalina.startup.Bootstrap start
"""
```

开机自启脚本

```shell
#!/bin/sh
# Tomcat init script for Linux.
#
# chkconfig: 2345 96 14
# description: The Apache Tomcat servlet/JSP container.
# JAVA_OPTS='-Xms64m -Xmx128m'
JAVA_HOME=/usr/local/jdk
CATALINA_HOME=/usr/local/tomcat
export JAVA_HOME CATALINA_HOME

case $1 in
start)
  exec $CATALINA_HOME/bin/catalina.sh start ;;
stop)
  exec $CATALINA_HOME/bin/catalina.sh stop;;
restart)
  $CATALINA_HOME/bin/catalina.sh stop
  sleep 2
  exec $CATALINA_HOME/bin/catalina.sh start ;;
configtest)
  exec $CATALINA_HOME/bin/catalina.sh configtest ;;
*)
  exec $CATALINA_HOME/bin/catalina.sh * ;;
esac
```



## 访问测试

URL：http://192.168.100.40:8080



## 查看日志

```shell
[root@tomcat tomcat]# tailf logs/catalina.out
```



## 配置文件

```shell
[root@tomcat conf]# pwd
"""
/application/tomcat/conf
"""

[root@tomcat conf]# ll -h
"""
total 216K
drwxr-xr-x 3 root root 4.0K Jan 26 06:10 Catalina
-rw------- 1 root root 13K Sep 28 16:19 catalina.policy
-rw------- 1 root root 7.0K Sep 28 16:19 catalina.properties
-rw------- 1 root root 1.6K Sep 28 16:19 context.xml
-rw------- 1 root root 3.4K Sep 28 16:19 logging.properties
-rw------- 1 root root 6.4K Sep 28 16:19 server.xml # 主配置文件
-rw------- 1 root root 1.8K Sep 28 16:19 tomcat-users.xml # Tomcat管理用户配置文件
-rw------- 1 root root 1.9K Sep 28 16:19 tomcat-users.xsd
-rw------- 1 root root 164K Sep 28 16:19 web.xml
"""
```



## 后台管理

测试功能，生产环境不要用。

Tomcat管理功能用于对Tomcat自身以及部署在Tomcat上的应用进行管理的web应用。在默认情况下是处于禁用状态的。如果需要开启这个功能，就需要配置管理用户，即配置前面说过的tomcat-users.xml。

```shell
[root@tomcat ~]# vim /application/tomcat/conf/tomcat-users.xml
"""
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
	...
	 # new add
	<role rolename="manager-gui"/>
	<role rolename="admin-gui"/>
	<user username="tomcat" password="tomcat" roles="manager-gui,admin-gui"/>
</tomcat-users>
"""

[root@tomcat ~]# catalina.sh configtest  # 测试配置文件是否有误
[root@tomcat ~]# /application/tomcat/bin/shutdown.sh
[root@tomcat ~]# /application/tomcat/bin/startup.sh
```

URL：http://192.168.100.40:8080/status



# server.xml详解

## 组件

顶级组件：位于整个配置的顶层，如server。
容器类组件：可以包含其它组件的组件，如service、engine、host、context。
连接器组件：连接用户请求至tomcat，如connector。
被嵌套类组件：位于一个容器当中，不能包含其他组件，如Valve、logger。

```xml
<server>
	<service>
        <connector />
        <engine>
            <host>
            	<context></context>
            </host>
            <host>
            	<context></context>
            </host>
        </engine>
    </service>
</server>
```

engine：核心容器组件，catalina引擎，负责通过connector接收用户请求，并处理请求，将请求转至对应的虚拟主机host。

host：类似于httpd中的虚拟主机，一般而言支持基于FQDN的虚拟主机。

context：定义一个应用程序，是一个最内层的容器类组件（不能再嵌套）。配置context的主要目的指定对应对的webapp的根目录，类似于httpd的alias，其还能为webapp指定额外的属性，如部署方式等。

connector：接收用户请求，类似于httpd的listen配置监听端口的。

service（服务）：将connector关联至engine，因此一个service内部可以有多个connector，但只能有一个引擎engine。service内部有两个connector，一个engine。因此，一般情况下一个server内部只有一个service，一个service内部只有一个engine，但一个service内部可以有多个connector。

server：表示一个运行于JVM中的tomcat实例。

Valve：阀门，拦截请求并在将其转至对应的webapp前进行某种处理操作，可以用于任何容器中，比如记录日志(access log valve)、基于IP做访问控制(remote address filter valve)。

logger：日志记录器，用于记录组件内部的状态信息，可以用于除context外的任何容器中。

realm：可以用于任意容器类的组件中，关联一个用户认证库，实现认证和授权。可以关联的认证库有两种：UserDatabaseRealm、MemoryRealm和JDBCRealm。

UserDatabaseRealm：使用JNDI自定义的用户认证库。

MemoryRealm：认证信息定义在tomcat-users.xml中。

JDBCRealm：认证信息定义在数据库中，并通过JDBC连接至数据库中查找认证用户。

## 注释

```xml
<?xml version='1.0' encoding='utf-8'?>
<!--
<Server>元素代表整个容器,是Tomcat实例的顶层元素.由org.apache.catalina.Server接口来定义.它包含一个<Service>元素.并且它不能做为任何元素的子元素.
port指定Tomcat监听shutdown命令端口.终止服务器运行时,必须在Tomcat服务器所在的机器上发出shutdown命令.该属性是必须的.
shutdown指定终止Tomcat服务器运行时,发给Tomcat服务器的shutdown监听端口的字符串.该属性必须设置
-->
<Server port="8005" shutdown="SHUTDOWN">
    <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
    <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
    <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
    <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
    <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />
    <GlobalNamingResources>
        <Resource name="UserDatabase" auth="Container"
        type="org.apache.catalina.UserDatabase"
        description="User database that can be updated and saved"
        factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
        pathname="conf/tomcat-users.xml" />
    </GlobalNamingResources>
    <!--service服务组件-->
    <Service name="Catalina">
        <!--
        connector：接收用户请求，类似于httpd的listen配置监听端口.
        port指定服务器端要创建的端口号，并在这个端口监听来自客户端的请求。
        address：指定连接器监听的地址，默认为所有地址（即0.0.0.0）
        protocol连接器使用的协议，支持HTTP和AJP。AJP（Apache Jserv Protocol）专用于tomcat与apache建立通信的， 在httpd反向代理用户请求至tomcat时使用（可见Nginx反向代理时不可用AJP协议）。
        minProcessors服务器启动时创建的处理请求的线程数
        maxProcessors最大可以创建的处理请求的线程数
        enableLookups如果为true，则可以通过调用request.getRemoteHost()进行DNS查询来得到远程客户端的实际主机名，若为false则不进行DNS查询，而是返回其ip地址
        redirectPort指定服务器正在处理http请求时收到了一个SSL传输请求后重定向的端口号
        acceptCount指定当所有可以使用的处理请求的线程数都被使用时，可以放到处理队列中的请求数，超过这个数的请求将不予处理
        connectionTimeout指定超时的时间数(以毫秒为单位)
        -->
        <Connector port="8080" protocol="HTTP/1.1"
        connectionTimeout="20000"
        redirectPort="8443" />
        <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
        <!--engine,核心容器组件,catalina引擎,负责通过connector接收用户请求,并处理请求,将请求转至对应的虚拟主机host
        defaultHost指定缺省的处理请求的主机名，它至少与其中的一个host元素的name属性值是一样的
        -->
        <Engine name="Catalina" defaultHost="localhost">
            <!--Realm表示存放用户名，密码及role的数据库-->
            <Realm className="org.apache.catalina.realm.LockOutRealm">
            <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
            resourceName="UserDatabase"/>
            </Realm>
            <!--
            host表示一个虚拟主机
            name指定主机名
            appBase应用程序基本目录，即存放应用程序的目录.一般为appBase="webapps" ，相对于CATALINA_HOME而言的，也可以写绝对路径。
            unpackWARs如果为true，则tomcat会自动将WAR文件解压，否则不解压，直接从WAR文件中运行应用程序
            autoDeploy：在tomcat启动时，是否自动部署。
            xmlValidation：是否启动xml的校验功能，一般xmlValidation="false"。
            xmlNamespaceAware：检测名称空间，一般xmlNamespaceAware="false"。
            -->
            <Host name="localhost" appBase="webapps"
            unpackWARs="true" autoDeploy="true">
                <!--
                Context表示一个web应用程序，通常为WAR文件
                docBase应用程序的路径或者是WAR文件存放的路径,也可以使用相对路径，起始路径为此Context所属Host中appBase定义的路径。
                path表示此web应用程序的url的前缀，这样请求的url为http://localhost:8080/path/****
                reloadable这个属性非常重要，如果为true，则tomcat会自动检测应用程序的/WEB-INF/lib 和/WEB-INF/classes目录的变化，自动装载新的应用程序，可以在不重启tomcat的情况下改变应用程序
                -->
                <Context path="" docBase="" debug=""/>
                <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                prefix="localhost_access_log" suffix=".txt"
                pattern="%h %l %u %t &quot;%r&quot; %s %b" />
            </Host>
            </Engine>
    </Service>
</Server>
```



# WEB站点部署

上线的代码有两种方式，第一种方式是直接将程序目录放在webapps目录下面，这种方式大家已经明白了，就不多说了。第二种方式是使用开发工具将程序打包成war包，然后上传到webapps目录下面。下面让我们见识一下这种方式。

## war包

```shell
[root@tomcat webapps]# pwd
/application/tomcat/webapps
[root@tomcat webapps]# rz # 上传memtest.war，此文件也在上面的百度网盘里
[root@tomcat webapps]# ls
"""
# 自动解压war包同名目录
docs examples host-manager manager memtest memtest.war ROOT
"""
```

URL：http://192.168.100.40:8080/memtest/index.jsp

## 自定义

修改访问URLhttp://192.168.100.40:8080/memtest/index.jsp为http://192.168.100.40:8080/index.jsp

方法一

将index.jsp或其他程序放在tomcat/webapps/ROOT目录下即可。因为默认网站根目录为tomcat/webapps/ROOT

方法二

```shell
[root@tomcat ~]# vim /application/tomcat/conf/server.xml
"""
<Host name="localhost" appBase="webapps"
unpackWARs="true" autoDeploy="true">
<Context path="" docBase="/application/tomcat/webapps/memtest" debug="0" reloadable="false" crossContext="true"/>
"""
[root@tomcat ~]# /application/tomcat/bin/shutdown.sh
[root@tomcat ~]# /application/tomcat/bin/startup.sh
```

