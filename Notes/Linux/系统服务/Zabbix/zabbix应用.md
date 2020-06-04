[TOC]

# **配置文件**

zaabix_agentd.conf 为 agent 的配置文件，至少应该为其指定server的IP地址；

zabbix_proxy.conf 为 proxy 的配置文件，至少应该为其指定proxy的主机名和server的IP，以及数据库等相关的配置信息；

zabbix.conf 为 zabbix-web 的配置文件

zabbix.conf配置文件说明

 

```
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

# **web端配置**

如何确定zabbix的监控对象：

\- 手动添加

\- 自动发现

\- hosts, host group

\- item, application

  \- item: key

\- graph, screen

  \- trigger, event (discovery)

  \- action (notification, operation, condition)

# zbbix模板

模板是一系列配置的集合，它可以方便地快速部署在某监控对象上，并支持重复应用。

模板可包含多种类型的条目：

\- items

\- triggers

\- graphs

\- applications

\- screens (since Zabbix 2.0)

\- low-level discovery rules (since Zabbix 2.0)

将模板应用至某主机上时，其定义的所有条目都会自动添加。因此，模板通常用于为某监控的服务或应用程序整合一组条目并将之分别应用于对应的运行了相应服务的主机上。此外，模板的另一个好处在于，必要时，修改了模板，被应用的主机都会相应的作出修改。

Delta (speed per second)：保存为(value-prev_value)/(time-prev_time的计算结果，即当前值减去前一次获取的数据值，除以当前时间戳减去前一次值获取时的时间戳得到的结果；如果当前值小于前一次的值，其将会被丢弃；

Delta (simple change)：保存为 (value-prev_value)的计算结果；

Groups: 应包含机器用途, 系统版本, 应用程序, 地址位置, 业务单元

Itme: 默认包含的item有多重类型

\- 网卡流量相关item: 

net.if.in[if,<mode>]

net.if.out[if,<mode>]

net.if.total[if,<mode>]

in入站流量;

out出站流量;

total总量;

if网卡接口名称,如eth0;

mode有bytes(默认,字节), packets(监控包的个数),errors(监控错误信息), dropped(监控drop);

例如:

net.if.in[eth0,bytes],监控eth0网卡的字节数

\- 端口相关item

net.tcp.listen[port]

net.tcp.listen[<ip>,port]

net.tcp.service[service,<ip>,<port>]

net.udp.listen[port]

监控服务|ip|端口是否正常运行|打开

\- 进程相关item

kernel.maxfiles: 监控内核打开文件的最大数量

kernel.maxproc: 监控用户打开的最大进程数

\- CPU相关item

system.cpu.intr

system.cpu.load[<cpu>,<mode>]

system.cpu.num[<type>]

system.cpu.switches

system.cpu.util[<cpu>,<type>,<mode>]

\- 磁盘IO或文件系统相关item

vfs.dev.read[<device>,<type>,<mode>]

vfs.dev.write[<device>,<type>,<mode>]

vfs.dev.inde[fs,<mode>]

自定义item

关键是选取一个唯一的key;

命令: 收集数据的命令或脚本;

Trigger

trigger有2中状态: ok|problem

Severity严重级别:

Not classifield: 未知级别,灰色;

Information: 一般信息,亮绿;

Warning: 警告信息,黄色;

Average: 一般故障,橙色;

High: 高级别故障,红色;

Disater: 严重故障,亮红;

Action:

触发条件一般为事件(events)

Trigger events: OK --> PROBLEM

Discovery events: zabbix的netwrok discovery工作时发现主机

Auto registration events: 主动模式的agent注册时产生的事件

Internal events: Item变成不再被支持,或Trigger变成未知状态

# Operations的功能

动作:

\- send message

配置步骤:

1,定义好Media

2,定义好用户

3,配置要发送的信息

\- Remote command

# 用户自定义参数

/etc/zabbix

​	zabbix_agentd.conf, zabbix_agentd.d/*.conf

nginx status 开启方法：

 

```
server {
    ...
    location /status {
        stub_status on;
        access_log off;
        allow 123.123.123.123; # 允许访问的 IP
        allow 127.0.0.1;
        deny all;
    }
}
```

状态页面各项数据的意义：

active connections – 当前 Nginx 正处理的活动连接数。

serveraccepts handled requests — 总共处理了 233851 个连接 , 成功创建 233851 次握手 (证明中间没有失败的 ), 总共处理了 687942 个请求 ( 平均每次握手处理了 2.94 个数据请求 )。

reading — nginx 读取到客户端的 Header 信息数。

writing — nginx 返回给客户端的 Header 信息数。

waiting — 开启 keep-alive 的情况下，这个值等于 active – (reading + writing)， 意思就是 Nginx 已经处理完正在等候下一次请求指令的驻留连接。

/usr/lib/zabbix/externalscripts

/etc/zabbix/externalscripts

# 自动发现

web监控

分布式环境中使用zabbix

 

```
UserParameter=Nginx.active[*], /usr/bin/curl -s "http://$1:$2/status" | awk '/^Active/ {print $NF}'
UserParameter=Nginx.reading[*], /usr/bin/curl -s "http://$1:$2/status" | grep 'Reading' | cut -d" " -f2
UserParameter=Nginx.writing[*], /usr/bin/curl -s "http://$1:$2/status" | grep 'Writing' | cut -d" " -f4
UserParameter=Nginx.waiting[*], /usr/bin/curl -s "http://$1:$2/status" | grep 'Waiting' | cut -d" " -f6
UserParameter=Nginx.accepted[*], /usr/bin/curl -s "http://$1:$2/status" | awk '/^[ \t]+[0-9]+[ \t]+[0-9]+[ \t]+[0-9]+/ {print $$1}'
UserParameter=Nginx.handled[*], /usr/bin/curl -s "http://$1:$2/status" | awk '/^[ \t]+[0-9]+[ \t]+[0-9]+[ \t]+[0-9]+/ {print $$2}'
UserParameter=Nginx.requests[*], /usr/bin/curl -s "http://$1:$2/status" | awk '/^[ \t]+[0-9]+[ \t]+[0-9]+[ \t]+[0-9]+/ {print $$3}'

UserParameter=varnish.stat[*], /usr/lib/zabbix/externalscripts/varnishstatus varnish_stat $1
UserParameter=varnish.count[*], /usr/lib/zabbix/externalscripts/varnishstatus varnish_count $1
UserParameter=varnish.hitrate, /usr/lib/zabbix/externalscripts/varnishstatus varnish_hitrate

# This is a config file for Zabbix Agent (Unix)
# To get more information about Zabbix, visit http://www.zabbix.com

############ GENERAL PARAMETERS #################

### Option: PidFile
# Name of PID file.
#
# Mandatory: no
# Default:
PidFile=/var/log/zabbix/zabbix_agentd.pid

### Option: LogFile
# Name of log file.
# If not set, syslog is used.
#
# Mandatory: no
# Default:
# LogFile=

LogFile=/var/log/zabbix/zabbix_agentd.log

### Option: LogFileSize
# Maximum size of log file in MB.
# 0 - disable automatic log rotation.
#
# Mandatory: no
# Range: 0-1024
# Default:
# LogFileSize=1
LogFileSize=20

### Option: DebugLevel
# Specifies debug level
# 0 - no debug
# 1 - critical information
# 2 - error information
# 3 - warnings
# 4 - for debugging (produces lots of information)
#
# Mandatory: no
# Range: 0-4
# Default:
# DebugLevel=3


### Option: SourceIP
# Source IP address for outgoing connections.
#
# Mandatory: no
# Default:
# SourceIP=

### Option: EnableRemoteCommands
# Whether remote commands from Zabbix server are allowed.
# 0 - not allowed
# 1 - allowed
#
# Mandatory: no
# Default:
# EnableRemoteCommands=0

### Option: LogRemoteCommands
# Enable logging of executed shell commands as warnings.
# 0 - disabled
# 1 - enabled
#
# Mandatory: no
# Default:
# LogRemoteCommands=0

##### Passive checks related

### Option: Server
# List of comma delimited IP addresses (or hostnames) of Zabbix servers.
# Incoming connections will be accepted only from the hosts listed here.
# No spaces allowed.
# If IPv6 support is enabled then '127.0.0.1', '::127.0.0.1', '::ffff:127.0.0.1' are treated equally.
#
# Mandatory: no
# Default:
# Server=

Server=172.16.10.15,127.0.0.1

### Option: ListenPort
# Agent will listen on this port for connections from the server.
#
# Mandatory: no
# Range: 1024-32767
# Default:
# ListenPort=10050

### Option: ListenIP
# List of comma delimited IP addresses that the agent should listen on.
# First IP address is sent to Zabbix server if connecting to it to retrieve list of active checks.
#
# Mandatory: no
# Default:
# ListenIP=0.0.0.0

### Option: StartAgents
# Number of pre-forked instances of zabbix_agentd that process passive checks.
# If set to 0, disables passive checks and the agent will not listen on any TCP port.
#
# Mandatory: no
# Range: 0-100
# Default:
# StartAgents=3
StartAgents=2

##### Active checks related

### Option: ServerActive
# List of comma delimited IP:port (or hostname:port) pairs of Zabbix servers for active checks.
# If port is not specified, default port is used.
# IPv6 addresses must be enclosed in square brackets if port for that host is specified.
# If port is not specified, square brackets for IPv6 addresses are optional.
# If this parameter is not specified, active checks are disabled.
# Example: ServerActive=127.0.0.1:20051,zabbix.domain,[::1]:30051,::1,[12fc::1]
#
# Mandatory: no
# Default:
# ServerActive=

ServerActive=172.16.10.15

### Option: Hostname
# Unique, case sensitive hostname.
# Required for active checks and must match hostname as configured on the server.
# Value is acquired from HostnameItem if undefined.
#
# Mandatory: no
# Default:
# Hostname=

# Hostname=Zabbix server

### Option: HostnameItem
# Item used for generating Hostname if it is undefined.
# Ignored if Hostname is defined.
#
# Mandatory: no
# Default:
HostnameItem=system.hostname

### Option: RefreshActiveChecks
# How often list of active checks is refreshed, in seconds.
#
# Mandatory: no
# Range: 60-3600
# Default:
# RefreshActiveChecks=120
RefreshActiveChecks=60

### Option: BufferSend
# Do not keep data longer than N seconds in buffer.
#
# Mandatory: no
# Range: 1-3600
# Default:
# BufferSend=5
BufferSend=10

### Option: BufferSize
# Maximum number of values in a memory buffer. The agent will send
# all collected data to Zabbix Server or Proxy if the buffer is full.
#
# Mandatory: no
# Range: 2-65535
# Default:
# BufferSize=100
BufferSize=1000

### Option: MaxLinesPerSecond
# Maximum number of new lines the agent will send per second to Zabbix Server
# or Proxy processing 'log' and 'logrt' active checks.
# The provided value will be overridden by the parameter 'maxlines',
# provided in 'log' or 'logrt' item keys.
#
# Mandatory: no
# Range: 1-1000
# Default:
# MaxLinesPerSecond=100
MaxLinesPerSecond=200

### Option: AllowRoot
# Allow the agent to run as 'root'. If disabled and the agent is started by 'root', the agent
#       will try to switch to user 'zabbix' instead. Has no effect if started under a regular user.
# 0 - do not allow
# 1 - allow
#
# Mandatory: no
# Default:
# AllowRoot=0

############ ADVANCED PARAMETERS #################

### Option: Alias
# Sets an alias for parameter. It can be useful to substitute long and complex parameter name with a smaller and simpler one.
#
# Mandatory: no
# Range:
# Default:

### Option: Timeout
# Spend no more than Timeout seconds on processing
#
# Mandatory: no
# Range: 1-30
# Default:
Timeout=20

### Option: Include
# You may include individual files or all files in a directory in the configuration file.
# Installing Zabbix will create include directory in /usr/local/etc, unless modified during the compile time.
#
# Mandatory: no
# Default:
# Include=

# Include=/usr/local/etc/zabbix_agentd.userparams.conf
Include=/usr/local/zabbix/etc/zabbix_agentd.conf.d/

####### USER-DEFINED MONITORED PARAMETERS #######

### Option: UnsafeUserParameters
# Allow all characters to be passed in arguments to user-defined parameters.
# 0 - do not allow
# 1 - allow
#
# Mandatory: no
# Range: 0-1
# Default:
# UnsafeUserParameters=0

### Option: UserParameter
# User-defined parameter to monitor. There can be several user-defined parameters.
# Format: UserParameter=<key>,<shell command>
# See 'zabbix_agentd' directory for examples.
#
# Mandatory: no
# Default:
# UserParameter=
```

对于有高度写入操作的系统

dirty_background_ratio: 主要调整参数。如果需要把缓存持续的而不是一下子大量的写入硬盘，降低这个值。

dirty_ratio：第二调整参数。

Swapping参数

/proc/sys/vm/swappiness

默认，linux倾向于从物理内存映射到硬盘缓存，保持硬盘缓存尽可能大。未用的页缓存会被放进swap区。

数值为0，将会避免使用swapping

100，将会尽量使用swapping

少用swapping会增加程序的响应速度；多用swapping将会提高系统的可用性。

如果有大量的写操作，为避免I/O的长时间等待，可以设置： 多次测试最好的配置

$ echo 5 > /proc/sys/vm/dirty_background_ratio

$ echo 10 > /proc/sys/vm/dirty_ratio

注意内存和cpu

将语言修改为中文

修改这个下面文件

vim /usr/share/zabbix/include/locales.inc.php 

找到55行，将false改为true

或在web端修改

![img](zabbix%E5%BA%94%E7%94%A8.assets/9fd1b592-fbbc-406e-bbbc-dae2b1100cbc.png)