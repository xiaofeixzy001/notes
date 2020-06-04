[TOC]

Docker结合负载实现网站高可用

# 整体规划图

下图为一个小型的网络架构图，其中nginx 使用docker 运行

![img](docker%E7%BD%91%E7%BB%9C%E9%AB%98%E5%8F%AF%E7%94%A8.assets/277feadc-ad5c-487c-819b-c60ccf554635.jpg)

# 操作步骤

## 1，安装配置keepalived

master和slave两台主机上分别安装keepalived，用于实现vip漂移

 

```
### master主机 ###

yum install  keepalived –y
vim /etc/keepalived/keepalived.conf
"""
vrrp_instance MAKE_VIP_INT {
    state MASTER
    interface eth0  # 注意这里网卡名称是否与系统一致
    virtual_router_id 1
    priority 100
    advert_int 1
    unicast_src_ip 192.168.10.205
    unicast_peer {
        # 为了减少局域网内广播报文，这里限制仅向slave主机进行广播
        192.168.10.206
    }

    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.10.100/24 dev eth0 label eth0:1  # vip
    }
}
"""

systemctl  restart keepalived
systemctl  enable keepalived
systemctl  status keepalived

### slave主机 ###

yum install  keepalived –y
vim /etc/keepalived/keepalived.conf
"""
vrrp_instance MAKE_VIP_INT {
    state BACKUP
    interface eth0  # 注意这里网卡名称是否与系统一致
    virtual_router_id 1
    priority 50
    advert_int 1
    unicast_src_ip 192.168.10.206
    unicast_peer {
        192.168.10.205
    }

    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.10.100/24 dev eth0 label eth0:1  # vip
    }
}
"""

systemctl  restart keepalived
systemctl  enable keepalived
systemctl  status keepalived
```

 

```
### master主机 ###
yum install  keepalived –y
vim /etc/keepalived/keepalived.conf
"""
vrrp_instance MAKE_VIP_INT {
    state MASTER
    interface eth0  # 注意这里网卡名称是否与系统一致
    virtual_router_id 1
    priority 100
    advert_int 1
    unicast_src_ip 192.168.10.205
    unicast_peer {
        # 为了减少局域网内广播报文，这里限制仅向slave主机进行广播
        192.168.10.206
    }
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.10.100/24 dev eth0 label eth0:1  # vip
    }
}
"""
systemctl  restart keepalived
systemctl  enable keepalived
systemctl  status keepalived
### slave主机 ###
yum install  keepalived –y
vim /etc/keepalived/keepalived.conf
"""
vrrp_instance MAKE_VIP_INT {
    state BACKUP
    interface eth0  # 注意这里网卡名称是否与系统一致
    virtual_router_id 1
    priority 50
    advert_int 1
    unicast_src_ip 192.168.10.206
    unicast_peer {
        192.168.10.205
    }
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.10.100/24 dev eth0 label eth0:1  # vip
    }
}
"""
systemctl  restart keepalived
systemctl  enable keepalived
systemctl  status keepalived
```

## 2，安装配置haproxy

master和slave两台主机上分别安装haproxy，用于实现HA高可用

 

```
# 两个服务器配置好内核参数
vim /etc/sysctl.conf
"""
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptales = 1
net.ipv4.ip_nonlocal_bind = 1
"""
sysctl -p

# 或
sysctl -w net.ipv4.ip_nonlocal_bind=1


### master ###
yum install haproxy -y
vim /etc/haproxy/haproxy.cfg
"""
global
maxconn 100000
uid 99
gid 99
daemon
nbproc 1
log 127.0.0.1 local0 info

defaults
option http-keep-alive
#option  forwardfor
maxconn 100000
mode tcp
timeout connect 500000ms
timeout client  500000ms
timeout server  500000ms

listen stats
    mode http
    bind 0.0.0.0:9999
    stats enable
    log global
    stats uri     /haproxy-status
    stats auth    haadmin:q1w2e3r4ys
#================================================================ 
frontend docker_nginx_web
    bind 192.168.10.100:80 
    mode http
    default_backend docker_nginx_hosts

backend docker_nginx_hosts
    mode http
    #balance source
    balance roundrobin
    server 192.168.10.205   192.168.10.205:81 check inter 2000 fall 3 rise 5
    server 192.168.10.206   192.168.10.206:81 check inter 2000 fall 3 rise 5
"""

systemctl enable haproxy
systemctl restart haproxy
```

 

```
# 两个服务器配置好内核参数
vim /etc/sysctl.conf
"""
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptales = 1
net.ipv4.ip_nonlocal_bind = 1
"""
sysctl -p
# 或
sysctl -w net.ipv4.ip_nonlocal_bind=1
### master ###
yum install haproxy -y
vim /etc/haproxy/haproxy.cfg
"""
global
maxconn 100000
uid 99
gid 99
daemon
nbproc 1
log 127.0.0.1 local0 info
defaults
option http-keep-alive
#option  forwardfor
maxconn 100000
mode tcp
timeout connect 500000ms
timeout client  500000ms
timeout server  500000ms
listen stats
    mode http
    bind 0.0.0.0:9999
    stats enable
    log global
    stats uri     /haproxy-status
    stats auth    haadmin:q1w2e3r4ys
#================================================================ 
frontend docker_nginx_web
    bind 192.168.10.100:80 
    mode http
    default_backend docker_nginx_hosts
backend docker_nginx_hosts
    mode http
    #balance source
    balance roundrobin
    server 192.168.10.205   192.168.10.205:81 check inter 2000 fall 3 rise 5
    server 192.168.10.206   192.168.10.206:81 check inter 2000 fall 3 rise 5
"""
systemctl enable haproxy
systemctl restart haproxy
```

浏览器测试验证

192.168.10.205:9999/haproxy-status

192.168.10.206:9999/haproxy-status

## 3，启动nginx容器并验证

从本地Nginx 镜像启动一个容器，并指定端口，默认协议是tcp方式

 

```
# Server1启动Nginx容器：
docker rm -f `docker ps -a -q` # 先删除之前所有的容器
docker run -it --rm nginx-app01 bash

[root@219e0237198f /]# cat /apps/nginx/html/index.html  # 便于区分负载均衡效果
"""
from server1-nginx-app01 page!!
"""

docker run --name nginx-web1 -d -p 81:80 nginx-app01 nginx
ss -tnlp

# Server2启动Nginx容器：
docker rm -f `docker ps -a -q` # 先删除之前所有的容器
docker run -it --rm nginx-app02 bash

[root@219e0237198f /]# cat /apps/nginx/html/index.html
"""
from server2-nginx-app02 page!!
"""

docker run --name nginx-web2 -d -p 81:80 nginx-app02 nginx
ss -tnlp
```

 

```
# Server1启动Nginx容器：
docker rm -f `docker ps -a -q` # 先删除之前所有的容器
docker run -it --rm nginx-app01 bash
[root@219e0237198f /]# cat /apps/nginx/html/index.html  # 便于区分负载均衡效果
"""
from server1-nginx-app01 page!!
"""
docker run --name nginx-web1 -d -p 81:80 nginx-app01 nginx
ss -tnlp
# Server2启动Nginx容器：
docker rm -f `docker ps -a -q` # 先删除之前所有的容器
docker run -it --rm nginx-app02 bash
[root@219e0237198f /]# cat /apps/nginx/html/index.html
"""
from server2-nginx-app02 page!!
"""
docker run --name nginx-web2 -d -p 81:80 nginx-app02 nginx
ss -tnlp
```

浏览器测试访问VIP：192.168.10.100

查看haproxy状态：192.168.10.100/haproxy-status