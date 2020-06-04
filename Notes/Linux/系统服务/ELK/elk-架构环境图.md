[TOC]

# 需求

## 架构规划

两台nginx服务器，通过keepalive做高可用，对外地址是VIP；

两台elasticsearch服务器，做集群；

一台logstash服务器，用于从redis服务器提取日志；

两台redis服务器，做集群；

两台web服务器，一台nginx，一台tomcat，修改其日志文件格式为json；

在两台web服务器上使用filebeat收集日志，并发送至redis服务器；

在两台elasticsearch服务器上安装kibana，nginx反代至kibana；

## 最终实现

当要访问ELK日志统计平台的时候，首先访问的是两台nginx+keepalived做的负载高可用，访问的地址是keepalived的VIP，当一台nginx代理服务器挂掉之后也不影响访问；

然后nginx将请求转发到kibana，kibana再去elasticsearch获取数据，elasticsearch是两台做的集群，数据会随机保存在任意一台elasticsearch服务器上；

redis服务器做数据的临时保存，避免web服务器日志量过大的时候造成的数据收集与保存不一致导致的日志丢失，可以临时保存到redis，redis可以是集群；

然后再由logstash服务器在非高峰时期从redis持续的取出即可；

web服务器的日志由filebeat收集之后发送给redis；

另外有一台mysql数据库服务器，用于持久化保存特定的数据；



# 架构图





# IP规划

keepalive vip：

nginx-node01：192.168.100.61

nginx-node02：192.168.100.62

elasticsearch-node01：192.168.100.31

elasticsearch-node02：192.168.100.32

logstash：192.168.100.33











