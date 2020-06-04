[TOC]

# 需求

两台nginx服务器，基于keepalive做高可用。

VIP：172

# 架构图



# 主机信息表

| *主机名* | *资源配置*   | *操作系统*                    | *角色*     | *IP*           |
| -------- | ------------ | ----------------------------- | ---------- | -------------- |
| nginx    | 2/cpu+2G/mem | CentOS Linux release 7.5.1804 | Web-Master | 192.168.100.11 |
| nginx02  | 2/cpu+2G/mem | CentOS Linux release 7.5.1804 | Web-Slave  | 192.168.100.12 |



# 软件版本

| **名称** | **版本**            |
| -------- | ------------------- |
| Nginx    | nginx-1.16.1.tar.gz |
| PHP      |                     |



# 1,时间同步

主备服务器同步进行

```shell
yum install chrony -y
vim /etc/chrony.conf
"""
server time1.aliyun.com iburst
server time2.aliyun.com iburst
server time3.aliyun.com iburst
server time4.aliyun.com iburst
server time5.aliyun.com iburst
server time6.aliyun.com iburst
server time7.aliyun.com iburst
"""

systemctl start chronyd
systemctl status chronyd
systemctl enable chronyd
chronyc sources  # 同步时间,可以加入任务计划
```



# 2,SSH免密访问

## 2.1，master节点

```shell
cat /etc/hosts
"""
192.168.100.11    nginx
192.168.100.12    nginx02
"""

ssh-keygen
cat ~/.ssh/id_rsa.pub
ssh-copy-id 192.168.100.12  # 拷贝到目标主机,使从本机访问目标主机免密
ssh nginx02 date; date
```



## 2.2，slave节点

```shell
cat /etc/hosts
"""
192.168.100.11    nginx
192.168.100.12    nginx02
"""

ssh-keygen
cat ~/.ssh/id_rsa.pub
ssh-copy-id nginx
ssh nginx date; date
```



## 2.3，多主机相互免密认证

如果服务器很多，要实现互相免密认证，可以这样做：

```shell
ssh-copy-id 192.168.100.11  # 直接拷贝给自己
ls ~/.ssh/
"""
authorized_keys
"""

cat ~/.ssh/id_rsa.pub && cat ~/.ssh/authorized_keys  # 内容一样
scp -rp ~/.ssh nginx02:/root/
```

在不同机器上测试。由于所有机器用的都是同一组公钥私钥，相对来说安全性不好。

# 3,Keepalive安装配置

主备nginx同时安装keepalived

## 3.1，yum安装keepalive

```shell
yum intall -y keepalived
rpm -ql keepalived
cp /etc/keepalived/keepalived.conf{,.bak}
systemctl enable keepalived
systemctl start keepalived
systemctl status keepalived
```



## 3.2，keepalive配置

master配置

```shell
# 停止服务
systemctl stop keepalived

# 生成一个随机密钥,用于主备节点间探测消息加密
openssl rand -base64 8
"""
YQsusA==
"""

vim /etc/keepalived/keepalived.conf
"""
! Configuration File for keepalived

global_defs {
   notification_email {
      1023668666@qq.com  # 收件人
   }
   notification_email_from keepalived-nginx@qq.com  # 发件人
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id ka1
   vrrp_mcast_group4 224.100.100.100  # 多播地址,224-239
}

vrrp_instance VI_1 {
    state MASTER  # 节点身份
    interface ens33  # 绑定vip的网卡
    virtual_router_id 51  # 虚拟路由实例id,0-255
    priority 100  # 优先级,越大越高,0-255
    advert_int 1  # 通告时间间隔
    authentication {  # 认证加密
        auth_type PASS
        auth_pass YQsusA==
    }
    virtual_ipaddress {
        192.168.100.10/24 dev eth1  # vip
    }
}
"""

scp keepalived.conf nginx02:/etc/keepalived/
```

slave配置

```shell
vim /etc/keepalived/keepalived.conf
"""
! Configuration File for keepalived

global_defs {
   notification_email {
      1023668666@qq.com
   }
   notification_email_from keepalived-nginx@qq.com
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id ka2  # 不能一样
   vrrp_mcast_group4 224.100.100.100
}

vrrp_instance VI_1 {
    state BACKUP  # 不能一样
    interface ens33
    virtual_router_id 51
    priority 99  # 不能一样
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass YQsusA==  # 此处需要一致
    }
    virtual_ipaddress {
        192.168.100.10  # 此处需要一致
    }
}
"""
```

抓包

```shell
tcpdump -i ens33 -nn dst host 224.100.100.100
```

如果网卡名为eth0可不指定网卡名称

测试访问：192.168.100.10

# 监控nginx进程
keepalived是通过检测keepalived进程是否存在判断服务器是否宕机，如果keepalived进程存在但是nginx进程不在了那么keepalived是不会做主备切换，所以我们需要写个脚本来监控nginx进程是否存在，如果nginx不存在就将keepalived进程杀掉。

需求: