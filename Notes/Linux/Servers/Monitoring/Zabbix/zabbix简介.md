[TOC]

# 监控

数据采集 --> 数据存储 --> 数据展示

报警：采集到的数据达到或超过预设的阈值就会发出报警

# 监控目标 

本地资源，如负载，CPU，磁盘，内存，IO，RAID级别，CPU温度，passwd文件变化，本地所有文件指纹识别监控等；

网络服务，如端口，web，DB，ping丢包率，进程数，IDC网络流量等

其他设备，如路由器，交换机，端口流量，监控光衰，打印机，windows等

业务数据，如用户登录失败次数，登录网站次数，验证码输入失败次数，API接口流量并发，电商网站订单，支付交易数量等；

NMS：网络监控系统。监控端向被监控端发送请求收集各自的信息，被监控端将收集的数据返回给监控端。

# 开源监控工具

## SNMP

Simple Network Management Prococol简单网络管理协议

NMS:162

Agent:161

支持SNMP协议的常用开源监控工具：

zabbix, zennos, opennms, cacti, nagios, ganglia

SNMP的工作模式

\- NMS向agent采集数据

\- agent向NMS报告数据

\- NMS请求agent修改配置

SNMP的工作组件

\- MIB: management infomation base

\- SMI: MIB表示符号

\- SNMP协议

 

SNMP协议版本:

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

![img](zabbix%E7%AE%80%E4%BB%8B.assets/497fdfd8-7c19-405e-8b92-706a251cb21a.jpg)

# zabbix

## 简介

zabbix是一个基于WEB界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案;

zabbix由zabbix server与可选组件zabbix agent两部分组成;

zabbix server可以通过SNMP，zabbix agent，ping，端口监视等方法提供对远程服务器、网络设备等状态的监视;

zabbix agent需要安装在被监视的目标服务器上，它主要完成对硬件信息或与操作系统有关的内存，CPU等信息的收集;

## 主要特点

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

## 主要功能

\- CPU负荷监控

\- 内存使用状况

\- 磁盘使用状况

\- 网络状况

\- 端口监视

\- 日志监视

官方也提供了安装资料：http://www.zabbix.com/wiki/howto/monitor

## 工作方式

数据采集 --> 数据存储 --> 数据展示和分析 --> 报警

## 数据采集协议

SNMP

agent

ICMP/SSH/IPMI

## 数据存储

cacti: rrd

nagios: mysql

zabbix: mysql/pgsql/oracle

## 数据展示（Web）

java

php

移动app

## 报警

mail(smtp)

Chat Message

SMS

## 架构图

![img](zabbix%E7%AE%80%E4%BB%8B.assets/62eb0719-b392-488a-ba6d-178afbb92f82.png)

![img](zabbix%E7%AE%80%E4%BB%8B.assets/b8fee1eb-d55e-4fbe-952c-d27cc5b35897.jpg)

## 工作流程图

![img](zabbix%E7%AE%80%E4%BB%8B.assets/8b0ebd8f-7a2c-4ae1-add3-3ef770129c20.jpg)

其中zabbix-server可以部署到单独一台主机，数据库也可单独一台主机，而lamp也可单独一台，仅需要其能连接访问数据库即可。当然，也可以都部署到一台主机上。

官方文档参考: 

https://www.zabbix.com/documentation/2.4/manual/config/items/itemtypes/zabbix_agent

## zabbix组件

zabbix_server：负责接收agent发送的报告信息，用于配置、统计数据及操作数据；[C]

zabbix_database：用于存储统计数据的数据库[MySQL, PGSQL(postgreSQL), Oracle, DB2, SQLLite]

zabbix_web：zabbix的web界面[PHP]

zabbix_get：主动从被监控端获取数据

zabbix_agent：被监控端，负责收集本地数据

zabbix_send：将收集到的本地数据主动发往监控端或代理监控端

zabbix_proxy：可选，常用于分布式监控环境中，代理server收集被监控端的数据，然后整理统一发往监控端

## zabbix逻辑组件

主机组, 主机

item(监控项), application(应用), 

trigger(触发器), 触发event

event(事件)

action(动作) --> notice, command

media

users(用户)

## 一些常用术语

host（主机）：要监控的网络设备，可由IP或DNS名称指定

host group（主机组）：主机的逻辑容器，可以包含主机和模板，但同一个组内的主机和模板不能互相链接；主机组通常在给用户或用户组指派监控权限时使用（大致了解下就可以了）。

item（监控项）：这个从名字上可以理解，具体要监控哪些指标由它定义,定义收集被监控的数据的项

trigger（触发器）：就是超过了定义的合理范围，这家伙就会报警。

event（事件）：这都是触发器产生的。

action（动作）：对事件如何应对，比如要执行哪些操作。

escalation（报警升级）：如果在定义的5分钟没反应，从warning级别升到high级别，就是要提醒别人要尽快处理。

media（媒介）：发送报警的手段和通道，如Email。

remote command（远程命令）：预定义的命令，可在被监控主机处于某个特定条件下时自动执行。

template（模板）：用于快速定义被监控主机的预设条目集合，通常包含了item、trigger、graph、screen、application以及low-level discovery rule；模板可以直接链接至单个主机。（这个概念不理解不过没关系的，只要具体会怎么操作就可以了）

application（应用）：一组item的集合。

其他:

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

## 监控配置流程

完整的监控配置流程

Host group --> Hosts --> Applications --> Items --> Triggers --> Events --> Actions --> User groups --> Users --> Medias --> 

## 数据组成

产生的数据主要由四部分组成:

1,配置数据;

2,历史数据, 50B

3,历史趋势数据, 128B

4,时间数据, 130B