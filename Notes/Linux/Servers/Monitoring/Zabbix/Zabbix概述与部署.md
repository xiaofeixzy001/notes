[TOC]

# 一，Zabbix简介

## 前言

教程文档：https://www.zabbix.com/documentation/4.0/start

### 监控

数据采集 --> 数据存储 --> 数据展示

报警：采集到的数据达到或超过预设的阈值就会发出报警

### 监控目标

本地资源，如负载，CPU，磁盘，内存，IO，RAID级别，CPU温度，passwd文件变化，本地所有文件指纹识别监控等；

网络服务，如端口，web，DB，ping丢包率，进程数，IDC网络流量等

其他设备，如路由器，交换机，端口流量，监控光衰，打印机，windows等

业务数据，如用户登录失败次数，登录网站次数，验证码输入失败次数，API接口流量并发，电商网站订单，支付交易数量等；

NMS：网络监控系统。监控端向被监控端发送请求收集各自的信息，被监控端将收集的数据返回给监控端。

### 开源监控工具

#### SNMP

Simple Network Management Prococol简单网络管理协议

NMS:162

Agent:161

支持SNMP协议的常用开源监控工具：

zabbix, zennos, opennms, cacti, nagios, ganglia

SNMP的工作模式

- NMS向agent采集数据
- agent向NMS报告数据
- NMS请求agent修改配置

SNMP的工作组件

- MIB: management infomation base
- SMI: MIB表示符号



#### SNMP协议版本:

\- V1: 无验证功能

\- V2 -- > v2c: 有验证功能,明文

\- V3: 认证,加密,解密, 较少使用

SNMP软件包:

linux：net-snmp

可监控的对象[指标]:

\- 设备/软件: 服务器, 路由器, 交换机, IO系统, OS, 网络, 应用程序, 

\- 偶发性故障: 服务宕机, 服务不可用, 主机不可达, 

\- 严重故障: 磁盘空间已满等, 

\- 主机性能指标

\- 趋势: 时间序列数据

自动化监控

![img](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/497fdfd8-7c19-405e-8b92-706a251cb21a.jpg)



## 1.1 主要特点

\- 安装与配置简单，学习成本低 

\- 支持多语言（包括中文）

\- 免费开源

\- 自动发现服务器与网络设备

\- 分布式监视以及WEB集中管理功能

\- 可以无agent监视

\- 用户安全认证和柔软的授权方式

\- 通过WEB界面设置或查看监视结果

\- email等通知功能

等等

## 1.2 主要功能

\- CPU负荷监控

\- 内存使用状况

\- 磁盘使用状况

\- 网络状况

\- 端口监视

\- 日志监视

官方也提供了安装资料：http://www.zabbix.com/wiki/howto/monitor

## 1.3 工作方式

数据采集 --> 数据存储 --> 数据展示和分析 --> 报警

## 1.4 数据采集协议

SNMP

agent

ICMP/SSH/IPMI

## 1.5 数据存储

cacti: rrd

nagios: mysql

zabbix: mysql/pgsql/oracle

## 1.6 数据展示（Web）

java

php

移动app

## 1.7 报警

mail(smtp)

Chat Message

SMS

## 1.8 架构图



![img](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/b8fee1eb-d55e-4fbe-952c-d27cc5b35897.jpg)

## 1.9 工作流程图

![image-20200113105926731](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200113105926731.png)

zabbix-server可以部署到单独一台主机，数据库也可单独一台主机，而lamp也可单独一台，仅需要其能连接访问数据库即可。当然，也可以都部署到一台主机上。

![image-20200114154626437](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200114154626437.png)

官方文档参考: 

https://www.zabbix.com/documentation/2.4/manual/config/items/itemtypes/zabbix_agent



## 1.9 组件

zabbix 由以下几个组件部分构成：

- Zabbix Server：负责接收 agent 发送的报告信息的核心组件，所有配置，统计数据及操作数据均由其组织进行；

- Database Storage：专用于存储所有配置信息以及由 zabbix 收集的数据。例如MySQL, PGSQL(postgreSQL), Oracle, DB2, SQLLite；

- Web interface：zabbix 的 GUI 接口，通常与 Server 运行在同一台主机上，基于PHP服务；

- Proxy：可选组件，常用于分布监控环境中，代理 Server 收集部分被监控端的监控数据
  并统一发往 Server 端；

- Agent：部署在被监控主机上，负责收集本地数据并发往 Server 端或 Proxy 端；

**注：zabbix node 也是 zabbix server 的一种 。**

## 1.10 进程

默认情况下zabbix包含5个程序： zabbix_agentd、 zabbix_get、 zabbix_proxy、 zabbix_sender、zabbix_server，另外一个 zabbix_java_gateway 是可选，这个需要另外安装。

下面来分别介绍下他们各自的作用：

- zabbix_agentd：客户端守护进程，此进程收集客户端数据，例如 cpu 负载、内存、硬盘使用情况等。

- zabbix_get：zabbix 工具，单独使用的命令，通常在 server 或者proxy端执行获取远程客户端信息的命令。 通常用户排错。 例如在server端获取不到客户端的内存数据， 我们可以使用zabbix_get获取客户端的内容的方式来做故障排查。
- zabbix_sender：zabbix 工具，用于发送数据给 server 或者proxy，通常用于耗时比较长的检查。很多检查非常耗时间，导致 zabbix 超时。于是我们在脚本执行完毕之后，使用 sender 主动提交数据。
- zabbix_server：zabbix 服务端守护进程。zabbix_agentd、zabbix_get、zabbix_sender、zabbix_proxy、zabbix_java_gateway 的数据最终都是提交到 server。当然不是数据都是主动提交给 zabbix_server,也有的是 server 主动去取数据。
- zabbix_proxy：zabbix 代理守护进程。功能类似server，唯一不同的是它只是一个中转站，它需要把收集到的数据提交/被提交到 server 里。
- zabbix_java_gateway：zabbix2.0 之后引入的一个功能。顾名思义：Java 网关，类似 agentd，但是只用于 Java方面。需要特别注意的是，它只能主动去获取数据，而不能被动获取数据。 它的数据最终会给到server或者proxy。

## 1.11 zabbix监控环境中相关术语

- 主机（host） ：要监控的网络设备，可由 IP 或 DNS 名称指定；

- 主机组（host group）：主机的逻辑容器，可以包含主机和模板，但同一个组织内的主机和模板不能互相链接；主机组通常在给用户或用户组指派监控权限时使用；

- 监控项（item） ：一个特定监控指标的相关的数据；这些数据来自于被监控对象；item是 zabbix 进行数据收集的核心，相对某个监控对象，每个 item 都由"key"标识；
- 触发器（trigger） ：一个表达式，用于评估某监控对象的特定 item 内接收到的数据是否在合理范围内，也就是阈值；接收的数据量大于阈值时，触发器状态将从"OK"转变为"Problem"，当数据再次恢复到合理范围，又转变为"OK"；
- 事件（event） ：触发一个值得关注的事情，比如触发器状态转变，新的 agent 或重新上
  线的 agent 的自动注册等；
- 动作（action） ：指对于特定事件事先定义的处理方法，如发送通知，何时执行操作；
- 报警升级（escalation）：如果在定义的5分钟没反应，从warning级别升到high级别，就是要提醒别人要尽快处理。
- 报警媒介类型（media） ：发送通知的手段或者通道，如 Email、Jabber 或者 SMS 等；
- 远程命令（remote command）：预定义的命令，可在被监控主机处于某个特定条件下时自动执行。
- 模板 （template） ：用于快速定义被监控主机的预设条目集合， 通常包含了 item、 trigger、graph、 screen、 application 以及 low-level discovery rule；模板可以直接链接至某个主机；
- 前端（frontend） ：Zabbix 的 web 接口
- 应用（application）：一组item的集合。

## 1.12 其他说明

图形（graph）：通过项目来获得数据，以图形来展示;

自动发现（discovery）:通过定义自动发现条件，配合动作批量添加主机;

自动注册（auto-registraion）：agent想Server发送注册请求，server定义自动注册条件来批量添加主机;

低级自动发现（Low——discovery）：简单定义一个类多个项，如：磁盘容量监控，监控磁盘所有的分区;

维护(maintenance)：定义主机合适的维护状态;

拓扑图（map）：可以主机直接的拓扑;

屏幕（screents）：多种类型显示到一个screents里面;

IT服务（IT Service）：有时一台主机宕掉可能不会影响到服务，IT服务可以定义容忍的限度;

仪表盘（dashboard）：监视整体状态的显示;

总览（overview）：显示所有机器的数据和触发器的状态;

web(web scennario)：定义场景监控的web服务器，用于检测web站点可用性的一个或多个HTTP请求；

前端(frontend )：zabbix的web接口；

最新数据（last data）：可查看主机项目获得的最新数据;



## 1.13 监控配置流程

完整的监控配置流程

Host group --> Hosts --> Applications --> Items --> Triggers --> Events --> Actions --> User groups --> Users --> Medias --> 

## 1.14 数据组成

产生的数据主要由四部分组成:

1,配置数据;

2,历史数据, 50B

3,历史趋势数据, 128B

4,时间数据, 130B



# 二，ZabbixServer的安装与配置

## 2.1 硬件需求

![image-20200109175321425](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200109175321425.png)

如上图所示，P2的CPU、256MB内存已经可以监控20个主机。AMD 3200+/2G内存可以监控500个主机（05年大学的时候，中低端主流cpu，这都快10年了，尤其可见zabbix对服务器的硬件配置要求有多低）,现在的服务器一般都比上面最高配还来得高，所以我武断的认为，大家手头的服务器都有能力监控1w+以上的服务器，我再武断的认为手头上有1w+服务器的公司能有多少.

## 2.2 软件需求

### 2.2.1 数据库

![image-20200109175404602](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200109175404602.png)

### 2.2.2 WEB

![image-20200109175444351](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200109175444351.png)

### 2.2.3 依赖环境

![image-20200109175519607](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200109175519607.png)

以下内容都为可选项，如果你需要监控特定项，安装特定支持即可。

OpenIPMI：IPMI硬件监控

libssh2：版本1.0以上，监控ssh服务

fping：icmp监控项

libcurl：监控web项.

libiksemel：支持jabber报警

net-snmp：增加SNMP支持

### 2.2.4 java网关

如果需要通过Java网关来监控你的Java进程，那么你需要增加如下支持

logback-core-0.9.27.jar ：http://logback.qos.ch/ ，0.9.27, 1.0.13, and 1.1.1已测试

logback-classic-0.9.27.jar ：http://logback.qos.ch/ ， 0.9.27, 1.0.13, and 1.1.1.已测试

slf4j-api-1.6.1.jar ：http://logback.qos.ch/ ，1.6.1, 1.6.6, and 1.7.6.已测试

android-json-4.3_r3.1.jar ：https://android.googlesource.com/platform/libcore/+/master/json ，2.3.3_r1.1 and 4.3_r3.1已测试

### 2.2.5 时间同步

```shell
[root@zabbix-server ~]# crontab -l
00 00 * * * /usr/sbin/ntpdate -u ntp1.aliyun.com

[root@zabbix-server ~]# crontab -l
00 00 * * * /usr/sbin/ntpdate -u ntp1.aliyun.com
```



## 2.3 安装配置

环境：

CentOS 6.9 x86_64

nginx-1.14.2

mysql-5.6.41-linux-glibc2.12-x86_64

php-5.6.22.tar.gz

需求：

部署在同一台主机

开启虚拟主机

php以php-fpm方式启动

zabbix架构安装和配置步骤：

![image-20200114154815920](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200114154815920.png)

在 Zabbix 监控服务器上，安装 Zabbix Server 和 Zabbix UI（Web）。

Zabbix Server 用来接受和发送监控详细，Zabbix UI（Web）用来对 Zabbix Server 的各项功能进行配置。

同时在被监控设备上，安装 Zabbix Agent。在其安装完毕以后，需要通过 zabbix_agenttd.conf 文件，对 Server 和 ServerActive 两个参数进行配置。

由于 Zabbix Agent 有被动模式和主动模式。被动模式，是 Zabbix Server 从 Zabbix Agent 上获取数据。

而主动模式，是 Zabbix Agent 主动将信息上传到 Zabbix Server。因此，这两个参数的内容都指的是 Zabbix Server 的 IP 地址。

Server 配置的是被动模式 Zabbix Server 的 IP，ServerActive 配置的是主动模式 Zabbix Server 的 IP。

当然，除此之外还需要更新防火墙配置，并打开 Zabbix 的访问端口（10050 和 10051）。

最后，给这个被监控的设备（Host），起一个 Hostname，这个名字会在 Zabbix Server 端配置的时候用到。



### 2.3.1 安装nginx

nginx版本：

编译安装目录/apps/nginx

配置文件/apps/nginx/conf/nginx.conf

日志/apps/nginx/logs/

pid文件/apps/nginx/run/nginx.pid

锁文件/apps/nginx/run/nginx.lock

开启虚拟主机conf/extra/*.conf

web根目录为/apps/web/zabbix

**安装略**

### 2.3.2 安装PHP

PHP版本：php-5.6

安装路径：/apps/php

编辑php.ini，修改如下参数：

```shell
[root@zabbix-server ~]# cp /etc/php.ini /etc/php.ini.bak
[root@zabbix-server ~]# vim /etc/php.ini
"""
date.timezone = Asia/Shanghai  # 时区必须修改
max_execution_time = 300
max_input_time = 300
post_max_size = 16M
memory_limit = 128M
mbstring.func_overload = 0
upload_max_filesize = 2M
always_populate_raw_post_data = -1
"""
```

**安装略**

### 2.3.3 配置Nginx代理至PHP

```shell
[root@zabbix-server conf]# vim nginx.conf
[root@zabbix-server conf]# cat nginx.conf
"""
worker_processes  1;
error_log   /apps/nginx/logs/error.log  notice;
events {
    worker_connections  10240;
    multi_accept        on;
    use                 epoll;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile      on;
    gzip          on;
    gzip_min_length 1k;
    gzip_comp_level 3;
    gzip_types text/plain application/javascript application/x-javascript text/javascript text/xml text/css;
    gzip_disable "MSIE[1-6]";
    gzip_vary on;
    keepalive_timeout  65;
    include extra/*.conf;  # 开启虚拟主机
    server_tokens off;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    #access_log  /applications/nginx/logs/access.log  main;
}
"""

[root@zabbix-server conf]# mkdir extra
[root@zabbix-server conf]# vim extra/zabbix.conf
"""
server {
	listen 80;
	server_name 192.168.100.11;
	location / {
		root            /data/web/zabbix;
		index           index.php index.html index.htm;
	}
	location ~ .*\.(php|php5)?$ {
		root	        /data/web/zabbix;
		fastcgi_pass	127.0.0.1:9000;
		fastcgi_index	index.php;
		include         fastcgi.conf;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	}
}
"""

[root@zabbix-server conf]# nginx -t
[root@zabbix-server conf]# service nginx reload
```

测试

### 2.3.4 安装mysql

安装略

为zabbix建立数据库和用户

```shell
mysql> SHOW DATABASES;
mysql> CREATE DATABASE zabbix CHARACTER SET utf8;
mysql> GRANT ALL ON zabbix.* to 'zbxuser'@'%' IDENTIFIED BY 'zbxpass';
mysql> GRANT ALL ON zabbix.* to 'zbxuser'@'localhost' IDENTIFIED BY 'zbxpass';
mysql> FLUSH PRIVILEGES;
mysql> \q
```



### 2.3.5 安装zabbix

zabbix版本：4.0 LTS

下载地址：https://repo.zabbix.com/zabbix/4.0/rhel/6/x86_64/

参考文档：https://www.zabbix.com/documentation/4.0/manual/installation/install_from_packages/rhel_centos

server端安装：zabbix，zabbix-server，zabbix-server-mysql，zabbix-get

server端支持web：zabbix-web，zabbix-web-mysql

server端支持监控自己：zabbix-agent，zabbix-sender

```shell
[root@zabbix-server ~]# groupadd zabbix
[root@zabbix-server ~]# useradd -g zabbix -s /sbin/nologin zabbix
[root@zabbix-server ~]# yum install -y net-snmp net-snmp-devel libevent libevent-devel
[root@zabbix-server src]# tar xf zabbix-4.0.16.tar.gz
[root@zabbix-server src]# cd zabbix-4.0.16
[root@zabbix-server zabbix-4.0.16]# ./configure --prefix=/apps/zabbix --enable-server --enable-agent --enable-proxy --enable-java  --with-mysql --with-net-snmp --with-libcurl --with-openssl
[root@zabbix-server zabbix-4.0.16]# make && make install

# 修改zabbix_server.conf配置文件
[root@zabbix-server zabbix-4.0.16]# cd /apps/zabbix/
[root@zabbix-server zabbix]# vim etc/zabbix_server.conf
"""
ListenPort=10051  # zabbix server监听端口
LogFile=/tmp/zabbix_server.log  # zabbix server日志路径
DBName=zabbix
DBUser=zbxuser
DBPassword=zbxpass
DBSocket=/tmp/mysql.sock  # MySQL的实例文件位置
StartPollers=5  # 用于设置zabbix server服务启动时启动Pollers（主动收集数据进程）的数量，数量越多，则服务端吞吐能力越强，同时对系统资源消耗越大
StartTrappers=10  # 用于设置zabbix server服务启动时启动Trappers（负责处理Agentd推送过来的数据的进程）的数量。Agentd为主动模式时，zabbix server需要设置这个值大一些。
StartDiscoverers=10  # 用于设置zabbix server服务启动时启动Discoverers进程的数量，如果zabbix监控报Discoverers进程忙时，需要提高该值。
ListenIP=0.0.0.0  # zabbix server启动的监听端口对哪些ip开放，Agentd为主动模式时，这个值建议设置为0.0.0.0
AlertScriptsPath=${datadir}/zabbix/alertscripts  # zabbix server运行脚本存放目录，一些供zabbix server使用的脚本，都可以放在这里。
"""

# 修改zabbix_agentd.conf配置文件
[root@zabbix-server zabbix]# vim etc/zabbix_agentd.conf
"""
LogFile=/tmp/zabbix_agentd.log  # zabbix agent日志存放路径
Server=192.168.100.11  # 指定zabbix server端IP地址
ListenPort=10050  # 指定agentd的监听端口
StartAgents=3  # 指定启动agentd进程数量。设置0表示关闭
ServerActive=127.0.0.1,192.168.100.11  # 启用agnetd主动模式，启动主动模式后，agentd将主动将收集到的数据发送到zabbix server端，Server Active后面指定的IP就是zabbix server端IP
Hostname=zabbix-server # 需要监控服务器的主机名或者IP地址，此选项的设置一定要和zabbix web端主机配置中对应的主机名一致。
Include=etc/zabbix/zabbix_agentd.d/  # 相关配置都可以放到此目录下，自动生效
UnsafeUserParameters=1  # 启用agent端自定义item功能，设置此参数为1后，就可以使用UserParameter指令了,UserParameter用于自定义item
"""

# 配置服务启动脚本文件
[root@zabbix-server zabbix-4.0.16]# cp misc/init.d/fedora/core/zabbix_server /etc/rc.d/init.d/
[root@zabbix-server zabbix-4.0.16]# cp misc/init.d/fedora/core/zabbix_agentd /etc/rc.d/init.d/

[root@zabbix-server zabbix-4.0.16]# vim /etc/rc.d/init.d/zabbix_server
"""
# Zabbix-Directory
BASEDIR=/apps/zabbix  # 这里修改为zabbix的安装目录
"""

[root@zabbix-server zabbix-4.0.16]# vim /etc/rc.d/init.d/zabbix_agentd
"""
# Zabbix-Directory
BASEDIR=/apps/zabbix  # 这里修改为zabbix的安装目录
"""

[root@zabbix-server zabbix-4.0.16]# chkconfig --add zabbix_server
[root@zabbix-server zabbix-4.0.16]# chkconfig --add zabbix_agentd
[root@zabbix-server zabbix-4.0.16]# chkconfig zabbix_server on
[root@zabbix-server zabbix-4.0.16]# chkconfig zabbix_agentd on
```



### 2.3.6 导入zabbix数据库

导入zabbix数据库文件，注意导入顺序。

```shell
[root@zabbix-server zabbix-4.0.16]# mysql -uzbxuser -p zabbix < database/mysql/schema.sql
[root@zabbix-server zabbix-4.0.16]# mysql -uzbxuser -p zabbix < database/mysql/images.sql
[root@zabbix-server zabbix-4.0.16]# mysql -uzbxuser -p zabbix < database/mysql/data.sql

[root@zabbix-server zabbix-4.0.16]# service zabbix_server start
[root@zabbix-server zabbix-4.0.16]# service zabbix_agentd start
[root@zabbix-server zabbix-4.0.16]# ss -tnlp
```

### 2.3.7 配置Zabbix web GUI

zabbix的WEB目录：/apps/web/zabbix

zabbix的WEB默认用户名和密码为**Admin/zabbix**，注意大小写

```
[root@zabbix-server zabbix-4.0.16]# mkdir /data/web
[root@zabbix-server zabbix-4.0.16]# cp -r frontends/php /data/web/zabbix
[root@zabbix-server zabbix-4.0.16]# chown -R nginx.nginx /data/web/zabbix
[root@zabbix-server zabbix-4.0.16]# service nginx reload
```

### 2.3.8 生成zabbix.conf.php配置文件

起初并没有zabbix.conf.php配置文件，通过访问zabbix的WEB页面，自动生成配置文件。

WEB访问：http://192.168.100.11/

配置zabbix数据库环境，如下图所示：

![image-20200110134820020](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200110134820020.png)

自动检测环境

![image-20200110135130650](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200110135130650.png)

设置数据库信息

![image-20200110135244014](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200110135244014.png)

zabbix服务器信息

![image-20200110135320902](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200110135320902.png)

最终如下：

![image-20200110135507948](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200110135507948.png)

![image-20200110135532600](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200110135532600.png)

**查看生成的zabbix.conf.php配置文件**

```shell
[root@zabbix-server ~]# cd /data/web/zabbix/conf
[root@zabbix-server conf]# ls
"""
maintenance.inc.php  zabbix.conf.php  zabbix.conf.php.example
"""

# 里面记录的信息就是我们刚刚的操作
[root@zabbix-server conf]# cat zabbix.conf.php
"""
<?php
// Zabbix GUI configuration file.
global $DB;

$DB['TYPE']     = 'MYSQL';
$DB['SERVER']   = 'localhost';
$DB['PORT']     = '0';
$DB['DATABASE'] = 'zabbix';
$DB['USER']     = 'zbxuser';
$DB['PASSWORD'] = 'zbxpass';

// Schema name. Used for IBM DB2 and PostgreSQL.
$DB['SCHEMA'] = '';

$ZBX_SERVER      = 'localhost';
$ZBX_SERVER_PORT = '10051';
$ZBX_SERVER_NAME = 'zabbix-server';

$IMAGE_FORMAT_DEFAULT = IMAGE_FORMAT_PNG;
"""
```

**提示：**
除了通过web界面的方式生成zabbix.conf.php文件外，我们也可以利用zabbix.conf.php.example的模版文件直接修改成我们需要的配置文件。

###  2.3.9 设置zabbix中文模式

![2.png-30.1kB](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/2-1577243163108.png)

或者修改这个下面文件

```shell
# vim /usr/share/zabbix/include/locales.inc.php 
```

找到55行，将false改为true.



# 三，zabbixAgent的安装与配置

## 3.1 本地安装agent

配置agent，以便让自己监控自己

```shell
[root@zabbix-server ~]# vim /apps/zabbix/etc/zabbix_agentd.conf
"""
Server=127.0.0.1,192.168.100.11  # 允许哪个主机来采集本机信息,逗号分隔
ListenPort=10050
StartAgents=3
ServerActive=127.0.0.1,192.168.100.11  # 定义本机是主动监控还是被动监控
"""
```

测试：在zabbix-server上执行如下命令

```shell
/apps/zabbix/bin/zabbix_get -s 192.168.0.221 -p 10050 -k "system.uptime"
"""
12614
"""
```

-s 是指定zabbix agent端的IP地址
-p 是指定zabbix agent端的监听端口
-k 是监控项，即item
如果有输出结果，表面zabbix server可以从zabbix agent获取数据，配置成功。

## 3.2 被监控端安装

### 3.2.1 Linux

官方下载地址：https://www.zabbix.com/download_agents

![image-20200110173621997](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200110173621997.png)

选择对应的系统版本和软件版本，因为zabbix-server是4.0，这里也选择4.0版本。

![image-20200110173646779](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200110173646779.png)

选择系统内核是2.6版本。

https://www.zabbix.com/downloads/4.0.16/zabbix_agents-4.0.16-linux2.6-amd64-static.tar.gz

```shell
[root@zabbix-server ~]# mkdir zabbix-agent
[root@zabbix-server ~]# cd zabbix-agent
[root@zabbix-server zabbix-agent]# wget https://www.zabbix.com/downloads/4.0.16/zabbix_agents-4.0.16-linux2.6-amd64-static.tar.gz
[root@zabbix-server zabbix-agent]# tar xf zabbix_agents-4.0.16-linux2.6-amd64-static.tar.gz
[root@zabbix-server zabbix-agent]# ll
"""
bin                conf                 sbin
"""

[root@zabbix-server zabbix-agent]# ln -sv bin/zabbix_sender bin/zabbix_get /usr/bin
[root@zabbix-server zabbix-agent]# ln -sv sbin/zabbix_agentd /usr/sbin
[root@zabbix-server zabbix-agent]# cp conf/zabbix_agentd.conf /usr/local/etc/
[root@zabbix-server zabbix-agent]# vim /usr/local/etc/zabbix_agentd.conf
"""
Server=192.168.100.11
ListenPort=10050
ServerActive=192.168.11
"""

[root@zabbix-server zabbix-agent]# vim /etc/services
"""
zabbix_agent 10050/tcp
zabbix_agent 10050/udp
"""

[root@zabbix-server zabbix-agent]# cp sbin/zabbix_agentd /etc/rc.d/init.d/
[root@zabbix-server zabbix-agent]# chmod +x /etc/rc.d/init.d/zabbix_agentd
[root@zabbix-server zabbix-agent]# /etc/rc.d/init.d/zabbix_agentd
[root@zabbix-server zabbix-agent]# ps -ef | grep zabbix
```

以上有点麻烦，为了快速安装部署，建议yum或rpm安装

参考网址：https://www.zabbix.com/download?zabbix=4.0&os_distribution=centos&os_version=6&db=&ws=

![image-20200110180011741](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200110180011741.png)

```shell
[root@mysql-server ~]# rpm -Uvh https://repo.zabbix.com/zabbix/4.0/rhel/6/x86_64/zabbix-release-4.0-2.el6.noarch.rpm
[root@mysql-server ~]# ll /etc/yum.repos.d/
[root@mysql-server ~]# yum clean all
[root@mysql-server ~]# yum -y install zabbix-agent 
[root@mysql-server ~]# chkconfig --add zabbix-agent
[root@mysql-server ~]# chkconfig zabbix-agent on
[root@mysql-server ~]# rpm -ql zabbix-agent
[root@mysql-server ~]# vim /etc/zabbix/zabbix_agentd.conf
"""
PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=192.168.100.11
ServerActive=192.168.100.11
Hostname=mysql-server
Include=/etc/zabbix/zabbix_agentd.d/*.conf
"""
[root@mysql-server ~]# service zabbix-agent start
```



### 3.2.2 Windows

下载地址：https://www.zabbix.com/download_agents

![image-20200113100419063](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200113100419063.png)

打开CMD，执行命令：
cd "C:\ProgramFiles\ZABBIX_AGENT3\bin"
zabbix_agentd.exe -i -c "C:\ProgramFiles\ZABBIX_AGENT3\conf\zabbix_agentd.conf"



# 五，server与agent通信加密

## 5.1 编译Zabbix支持加密

Zabbix使用TransportLayerSecurity (TLS) protocol v1.2进行加密，为了让Zabbix支持加密功能，在源码编译安装时必须要链接到下面三个加密库中的其中一个。

mbed TLS：早期也叫PolarSSL，目前仅支持1.3.x版本。注意不支持2.x版本。

GnuTLS：支持v3.1.18及更高的版本。

OpenSSL：支持v1.0.1及更高的版本。

根据你的选择，configure脚本可以使用下面的某个选项：

--with-mbedtls[=DIR]

--with-gnutls[=DIR]

--with-openssl[=DIR]

例如：

```shell
./configure --enable-server --enable-agent --with-mysql --enable-ipv6 --with-net-snmp--with-libcurl --with-libxml2 --with-openssl
```

在编译安装Zabbix的不同组件时可以使用不同的加密库，例如server使用OpenSSL，agent使用GnuTLS。建议使用OpenSSL，在实际测试中OpenSSL是最快的，接下来是GnuTLS。

如果你使用安装包安装Zabbix组件时，默认已经支持加密功能。你可以通过查看日志文件确定Zabbix安装的功能特性。例如下面是Zabbixserver启动时显示的特性列表。

## 5.2 生成并配置PSK

```shell
openssl rand -hex 32 > /apps/zabbix/etc/zabbix_agentd.conf.d/zabbix_agentd.psk

cat /apps/zabbix/etc/zabbix_agentd.conf.d/zabbix_agentd.psk
'''
dc0b5fd09660e7fd6fcba9e7a76daffb01687f85cab361c9e6b44fdb43972097
'''

vim /apps/zabbix/etc/zabbix_agentd.conf
'''
TLSConnect=psk
TLSAccept=psk
TLSPSKIdentity=agent
TLSPSKFile=/apps/zabbix/etc/zabbix_agentd.conf.d/zabbix_agentd.psk
'''

/apps/zabbix/bin/zabbix_get -s 127.0.0.1 -k "system.cpu.load[all,avg1]" --tls-connect=psk --tls-psk-identify="agent" --tls-psk-file=/apps/zabbix/etc/zabbix_agentd.conf.d/zabbix_agentd.psk
```

测试没问题后，就可以在添加主机时，选择加密。

![image-20200113142033206](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200113142033206.png)



### 5.3 windowns

下载agent安装包，注意后面对应的加密方式。例如：

![image-20200113134528584](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200113134528584.png)

![image-20200113134755029](Zabbix%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/image-20200113134755029.png)

修改其配置文件：

C:\ProgramFiles\Zabbix Agent\zabbix_agent.conf 配置文件，添加以下几行：

```shell
TLSConnect=psk
TLSAccept=psk
TLSPSKIdentity=agent
TLSPSKFile=C:\ProgramFiles\Zabbix Agent\zabbix_agentd.psk.txt
```

新建C:\ProgramFiles\Zabbix Agent\zabbix_agentd.psk.txt文档，将上面生成的字符串放进去，重启agent服务。

# 配置文件

zaabix_agentd.conf 为 agent 的配置文件，至少应该为其指定server的IP地址；

zabbix_proxy.conf 为 proxy 的配置文件，至少应该为其指定proxy的主机名和server的IP，以及数据库等相关的配置信息；

zabbix.conf 为 zabbix-web 的配置文件

## zabbix.conf 配置文件说明

```shell
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=1024    
DebugLevel=3                # 日志级别
PidFile=/var/run/zabbix/zabbix_server.pid
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix
DBSocket=/var/lib/mysql/mysql.sock
StartPollers=100              # poller进程  100                    
StartPollersUnreachable=30    # 无法访问的主机轮询进程30
StartPingers=30              # ping轮询数量
StartDiscoverers=30
StartTimers=10
SenderFrequency=30            # 发送报警超时
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
CacheSize=4096M              # 存储主机，项目和触发器数据的共享内存          
CacheUpdateFrequency=120      # 执行配置缓存的更新频率
StartDBSyncers=24            # 数据库同步进程
HistoryCacheSize=2048M
HistoryIndexCacheSize=2048M
TrendCacheSize=2048M        # 趋势数据最大2G
ValueCacheSize=2048M        # 缓存项历史数据请求，历史值缓存
Timeout=30
UnreachablePeriod=120        # 几秒钟的不可达性将主机视为不可用。  不可用
UnavailableDelay=60          # 主机在不可用期间内检查可用性的频率(秒)
UnreachableDelay=5          # 不可达检测频率，解决wait for 3 seconds
AlertScriptsPath=/usr/lib/zabbix/alertscripts
ExternalScripts=/usr/lib/zabbix/externalscripts
LogSlowQueries=2000          # 记录慢查询 
HousekeepingFrequency=1      # 从历史记录，警报和警报表中删除不必要的信息  不超过4个小时  每隔1小时启动一次，删除过期数据
MaxHousekeeperDelete=1000000 # 清除过期数据，超过这个阀值的行都会被清理
```



## 防火墙配置

```shell
[root@mysql-server ~]# vim /etc/sysconfig/iptables  
[root@mysql-server ~]# -A INPUT -m state --state NEW -m udp -p udp --dport 10050 -j ACCEPT  
[root@mysql-server ~]# -A INPUT -m state --state NEW -m tcp -p tcp --dport 10050 -j ACCEPT  
[root@mysql-server ~]# -A INPUT -m state --state NEW -m udp -p udp --dport 10051 -j ACCEPT  
[root@mysql-server ~]# -A INPUT -m state --state NEW -m tcp -p tcp --dport 10051 -j ACCEPT  
#service iptables restart
```



# 附



