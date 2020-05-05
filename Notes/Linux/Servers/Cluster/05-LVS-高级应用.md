[TOC]

# 防火墙标记

用于将不同的服务定义至后端RS

通过FWM定义集群的方式:在DS上netfilter的mangle表的PREROUTING定义用于'打标'的规则;

格式:

 

```
iptables -t mangle -A PREROUTING -d $vip -p $protocol --dports $port -j MARK --set-mark N
```

 

```
iptables -t mangle -A PREROUTING -d $vip -p $protocol --dports $port -j MARK --set-mark N
```

说明:

$vip: VIP地址

$protocol: 协议

$port: 协议端口

N: 标记号

基于标记FWM定义集群服务;

 

```
ipvsadm -A -f N -s scheduler
ipvsadm -a -f N -r $rip -g|-w
```

 

```
ipvsadm -A -f N -s scheduler
ipvsadm -a -f N -r $rip -g|-w
```

功用:将共享一组RS的集群服务统一进行定义

比如我们来定义一条防火墙规则: 目标是VIP,协议是tcp,请求端口是80和22,都打一个标记,比如这里为10,

然后我们在定义RS集群时,可将只要是此标记的端口服务,都转到后端的RS

 

```
iptables -t mangle -A PREROUTING -d vip -p tcp -dport 80 -j MAKE --set-mark 10
iptables -t mangle -A PREROUTING -d vip -p tcp -dport 22 -j MAKE --set-mark 10
iptables -t mangle -L -n
ipvsadm -A -f 10 -s rr
ipvsadm -a -f 10 -r rip1 -g
ipvsadm -a -f 10 -r rip2 -g
ipvsadm -L -n
```

 

```
iptables -t mangle -A PREROUTING -d vip -p tcp -dport 80 -j MAKE --set-mark 10
iptables -t mangle -A PREROUTING -d vip -p tcp -dport 22 -j MAKE --set-mark 10
iptables -t mangle -L -n
ipvsadm -A -f 10 -s rr
ipvsadm -a -f 10 -r rip1 -g
ipvsadm -a -f 10 -r rip2 -g
ipvsadm -L -n
```

# LVS的持久连接

lvs persistence

功能:无论ipvs使用何种调度方法,其都能实现将来自于同一个Client的请求,在一定时间内,始终定向至第一次调度时选择的RS,而无关调度算法.

设置持久连接命令:

 

```
ipvsadm -A|-E -t $VIP -s SCHEDULER -p TIMEOUT
# TIMEOUT表示超时时长,默认300s
```

 

```
ipvsadm -A|-E -t $VIP -s SCHEDULER -p TIMEOUT
# TIMEOUT表示超时时长,默认300s
```

Session保持的方法:

1,session绑定：始终将来自同一个源IP的请求定向至同一个RS(lvs的sh算法);

缺点：没有容错能力，有损均衡效果.

2,session复制：在RS间同步session，每个RS拥有集群中的所有session;

缺点：不适用于大规模集群.

3,session服务器：利用单独部署的服务器来统一管理集群中的session;

缺点：session服务器为单点故障所在，一旦损坏，所有信息全部丢失.

持久连接的实现方式

持久端口连接:PPC,单服务持久调度,将来自于同一个客户端对同一个集群服务的请求,在有效时间内,始终定向至此前选定的RS;

 

```
ipvsadm -A -t 172.16.200.1:80 -s rr -p 600
ipvsadm -a -t 172.16.200.1:80 -r 172.16.100.7 -g 
ipvsadm -a -t 172.16.200.1:80 -r 172.16.100.8 -g
```

 

```
ipvsadm -A -t 172.16.200.1:80 -s rr -p 600
ipvsadm -a -t 172.16.200.1:80 -r 172.16.100.7 -g 
ipvsadm -a -t 172.16.200.1:80 -r 172.16.100.8 -g
```

持久客户端连接: PCC, 单客户端持久调度,将来自于同一个客户端对directory上所有端口的请求,始终定向至此前选定的RS,PCC是把所有端口统统定义为集群服务,一律向RS转发;

 

```
ipvsadm -A -t 172.16.200.1:0 -s rr -p 600
ipvsadm -a -t 172.16.200.1:0 -r 172.16.100.7 -g 
ipvsadm -a -t 172.16.200.1:0 -r 172.16.100.8 -g

# 0端口表示所有端口,1-65535
```

 

```
ipvsadm -A -t 172.16.200.1:0 -s rr -p 600
ipvsadm -a -t 172.16.200.1:0 -r 172.16.100.7 -g 
ipvsadm -a -t 172.16.200.1:0 -r 172.16.100.8 -g
# 0端口表示所有端口,1-65535
```

持久防火墙标记连接: PNMPP,单FWM持久调度,将来自于同一个客户端对同一个防火墙标记的集群服务的请求,始终定向至此前选定的RS

 

```
iptables -t mangle -A PREROUTING -d 172.16.200.1 -i eth0 -p tcp --dport 80 -j MARK --set-mark 10
iptables -t mangle -A PREROUTING -d 172.16.200.1 -i eth0 -p tcp --dport 443 -j MARK --set-mark 10
ipvsadm -A -f 10 -s rr -p 600
ipvsadm -a -f 10 -r 172.16.100.7 -g
ipvsadm -a -f 10 -r 172.16.100.8 -g
```

 

```
iptables -t mangle -A PREROUTING -d 172.16.200.1 -i eth0 -p tcp --dport 80 -j MARK --set-mark 10
iptables -t mangle -A PREROUTING -d 172.16.200.1 -i eth0 -p tcp --dport 443 -j MARK --set-mark 10
ipvsadm -A -f 10 -s rr -p 600
ipvsadm -a -f 10 -r 172.16.100.7 -g
ipvsadm -a -f 10 -r 172.16.100.8 -g
```

持久连接的模板

lvs的持久连接功能,同iptables一样,也有一个自己的持久连接模板(一个内存缓冲区),这个模板独立于调度算法,保存着每一个客户端及分配给它的RS的映射关系,当模板内有某个client的记录并且还未失效时,则优先调度至记录的RS中,无视调度算法

查看这个模板的命令:

 

```
~]# ipvsadm -L -c
```

 

```
~]# ipvsadm -L -c
```

# HA[High avaliability]高可用

健康状态检测

directory: 做高可用集群

RealServer: 让directory对其做健康状态检测,并能根据检测的结果自动完成添加或移除等管理功能

检测方式

1,基于协议层检查

如基于ip(icmp);基于传输层(检测端口的开放状态);基于应用层(请求获取关键性的资源).

2,检查频度

估算检查的频率

3,状态判断

下线: ok -> failure -> failure -> failure -> down 多次检查确认再下线

上线: failure -> ok -> up 检测一次即让其上线

## HA示例

以CentOS6.8,2节点为例,构建HA集群:corosync + cluster-glue + pacemaker

节点:

node1.com  192.168.100.7  vip:

node2.com 192.168.100.8  vip:

控制端：

192.168.100.9

前提条件:

- 时间同步
- 基于主机名互相通信
- ssh密钥认证互信

1,利用ansible来统一安装部署

 

```
mkdir corosync
mkdir -pv roles/{common,ha}/{files,tasks,handlers,templates,vars,meta,default}
cp /etc/hosts corosync/roles/common/files/
cd corosync/roles/

vim common/files/hosts
"""
 192.168.100.7  www.node1.com  node1
 192.168.100.8  www.node2.com  node2
"""

vim common/tasks/main.yml
"""
- name: hosts file
  copy: src=hosts  dest=/etc/hosts
- name: sync time
  cron: name="sync time" minute="*/10" job="/usr/sbin/ntpdate 192.168.100.1 &> /dev/null"
"""

vim ha/tasks/main.yml
"""
- name: installed corosync and pacemaker
  yum: name={{ item }} state=present
  with_items:
     - corosync
     - pacemaker
  tags: inst
- name: auth key file
  copy: src=authkey dest=/etc/corosync/authkey owner=root group=root mode=0400
  tags: authkey
- name: configration file
  copy: src=corosync.conf dest=/etc/corosync/corosync.conf
  tags: conf
  notify:
        - restart corosync
- name: start corosync
  service: name=corosync state=started enabled=no
  tags: start
"""

vim ha/handlers/main.yml
"""
- name: restart corosync
  service: name=corosync state=restarted
"""

cd /root/corosync/ 
touch site.yml

vim ha.yml
"""
- name: install and config corosync
  remote_user: root
  hosts: hbhosts
  roles:
  - common
  - ha
"""

vim /etc/ansible/hosts
"""
[hbhosts]
192.168.100.7
192.168.100.8
"""
cp /etc/corosync/corosync.conf.example /etc/corosync/corosync.conf
vim /etc/corosync/corosync.conf
"""
# 添加如下内容
service {
  ver:  0
  name: pacemaker
  # use_mgmtd: yes
}

aisexec {
  user: root
  group: root
}
# 然后修改
totem {
    interface {
        bindnetaddr: 192.168.100.0
        mcastaddr:226.194.21.191
    }
}
 
logging {
    to_logfile: yes
    to_syslog: no 
}
"""

corosync-keygen  # 生成authkey文件 
cp -p /etc/corosync/authkey ha/files/  # 注意权限
cp -p /etc/corosync/corosync.conf ha/files/
```

 

```
mkdir corosync
mkdir -pv roles/{common,ha}/{files,tasks,handlers,templates,vars,meta,default}
cp /etc/hosts corosync/roles/common/files/
cd corosync/roles/
vim common/files/hosts
"""
 192.168.100.7  www.node1.com  node1
 192.168.100.8  www.node2.com  node2
"""
vim common/tasks/main.yml
"""
- name: hosts file
  copy: src=hosts  dest=/etc/hosts
- name: sync time
  cron: name="sync time" minute="*/10" job="/usr/sbin/ntpdate 192.168.100.1 &> /dev/null"
"""
vim ha/tasks/main.yml
"""
- name: installed corosync and pacemaker
  yum: name={{ item }} state=present
  with_items:
     - corosync
     - pacemaker
  tags: inst
- name: auth key file
  copy: src=authkey dest=/etc/corosync/authkey owner=root group=root mode=0400
  tags: authkey
- name: configration file
  copy: src=corosync.conf dest=/etc/corosync/corosync.conf
  tags: conf
  notify:
        - restart corosync
- name: start corosync
  service: name=corosync state=started enabled=no
  tags: start
"""
vim ha/handlers/main.yml
"""
- name: restart corosync
  service: name=corosync state=restarted
"""
cd /root/corosync/ 
touch site.yml
vim ha.yml
"""
- name: install and config corosync
  remote_user: root
  hosts: hbhosts
  roles:
  - common
  - ha
"""
vim /etc/ansible/hosts
"""
[hbhosts]
192.168.100.7
192.168.100.8
"""
cp /etc/corosync/corosync.conf.example /etc/corosync/corosync.conf
vim /etc/corosync/corosync.conf
"""
# 添加如下内容
service {
  ver:  0
  name: pacemaker
  # use_mgmtd: yes
}
aisexec {
  user: root
  group: root
}
# 然后修改
totem {
    interface {
        bindnetaddr: 192.168.100.0
        mcastaddr:226.194.21.191
    }
}
 
logging {
    to_logfile: yes
    to_syslog: no 
}
"""
corosync-keygen  # 生成authkey文件 
cp -p /etc/corosync/authkey ha/files/  # 注意权限
cp -p /etc/corosync/corosync.conf ha/files/
```