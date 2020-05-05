[TOC]

keepalived扩展

1,同步组,需要两块网卡的地址同步转移,如LVS的nat模型,当vip移动后,对应的dip也需要跟随vip一起移动.

 

```
vrrp_sync_group VG_1 {
    group {
    VI_1 
    VI_2
    }
}

vrrp_instance VI_1 {
    eth0
    vip
}

vrrp_instance VI_2 {
    eth1
    dip
}
```



2,自定义日志,默认存储在/var/log/messages中,改变其记录位置:

 

```
# vim /etc/sysconfig/keepalived
KEEPALIVED_OPTIONS="-D -S 3"
# vim /etc/rsyslog.conf
# Save boot messages also to boot.log ##在此处下面添加一行：
local3.*     /var/log/keepalived.log
```



3,自定义状态切换触发邮件通知

先自行写一个script,然后再keepalived配置文件中调用即可.

 

```
vim keepalived.conf
"""
# 添加如下
vrrp_instance {
    ...
    notify_master "/etc/keepalived/notify.sh master"  # 当前主机转换为了master,发送指定信息或执行指定脚本
    notify_backup "/etc/keepalived/notify.sh backup"  # 当前主机转换为了backup,发送指定信息或执行指定脚本
    notify_fault "/etc/keepalived/notify.sh fault"  # 当前主机出现故障,发送指定信息或执行指定脚本
    # notify ""  # 监测本机只要发生变化,就发送通知或执行指定脚本
}
"""

vim /etc/keepalived/notify.sh  # 脚本示例
"""
#!/bin/bash
#author :xiaofei
#Description : an example of notify script
vip=192.168.200.1
contact='root@localhost'
notify() {
    mailsubject="$(hostname) to be $1: $vip floating"
    mailbody="$(date +'%F %H:%M:%S'): vrrp transition,$(hostname) change to be $1"
    echo $mailbody | mail -s "$mailsubject" $contact
}
case "$1" in
    master)
        notify master
        /etc/rc.d/init.d/haproxy start
        #systemctl start nginx.service
        exit 0
    ;;
    backup)
        notify backup
        /etc/rc.d/init.d/haproxy stop
        #systemctl restart nginx.service
        exit 0
    ;;
    fault)
        notify fault
        /etc/rc.d/init.d/haproxy stop
        #systemctl stop nginx.service
        exit 0
    ;;
    *)
        echo "Usage: $(basename $0) {master|backup|fault}"
        exit 1
    ;;
esac
"""
```



常用监测脚本

1,使用vrrp_script实现依赖脚本监测文件的存在性,来降低节点的优先级,以此实现主备模式的切换

 

```
vrrp_script chk_down {     # 此处为自定义名字
   script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0 "     # 判断一个文件是否存在，存在返回1，否则返回0
   interval 1   # 检测间隔
   weight -2    # 如果返回1，权重减2，否则不变

# 监测web的访问返回结果,判断服务的可用性, 来了解你节点的优先级
vrrp_script chk_nginx {
    script "curl -s 192.168.200.1 | grep 192 &> /dev/null"
    interval 1
    weight -2
}

# 监控主机的nginx是否存在,0信号只是做进程探测的
vrrp_script chk_nginx2 {
        script "killall -0 nginx"   
        interval 1
        weight -2
}
 
vrrp_instance VI_1 {
    track_script {
        chk_down
        chk_nginx
        chk_nginx2
    }
}
```



2,使用客户端IE测试测试页面,是否正常显示

 

```
! Configuration File for keepalived
global_defs {
   notification_email {
        root@localhost
   }
   notification_email_from keepalived_admin@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id Centos7.pc2
}
vrrp_script chk_down {
        script "[ -f /etc/keepalived/down ] && exit 1 || exit 0"
        interval 1
        weight -20
}
vrrp_script chk_nginx {
        script "curl -s http://172.16.36.100 | grep 172 &> /dev/null"
        interval 1
        weight -10
}
vrrp_instance VI_1 {
    state BACKUP
    interface eno16777736
    virtual_router_id 100
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 99999999
    }
    virtual_ipaddress {
        172.16.36.100/16 dev eno16777736 label eno16777736:1
    }
    notify_master "/etc/keepalived/keepalived.sh master"
    notify_bakcup "/etc/keepalived/keepalived.sh backup"
    notify_fault "/etc/keepalived/keepalived.sh fault"
    track_script {
        chk_down
        chk_nginx
    }
}
```



KeepAlived的LVS相关配置

man keepalived.conf查看帮助信息

virtual_server配置段

 

```
virtual_server IP Port | virtual_server fwmark int | virtual_server group string {
    delay_loop <INT>  # 健康检查的时间间隔(次数)
    lb_argo rr|wrr|lc|wlc|lblc|sh|dh  # LVS调度算法
    lb_kind NAT|DR|TUN  # LVS模式
    persistence_timeout 360  # 持久化超时时间,单位是秒,默认360s
    persistence_granularity  # 持久化连接的颗粒度
    protocol TCP|UDP|SCTP  # 4层协议
    ha_suspend  # 如果virtual server的IP地址没有设置,则不进行后端服务器的健康检查
    virtualhost <STRING>  # 为HTTP_GET和SSL_GET执行要检查的虚拟主机,如virtualhost www.felix.com
    sorry_server <IPADDR> <PORT>  # 添加一个备用服务器,当所有的RS都故障时,指向此服务器
    sorry_server_inhibit  # 将inhibit_on_failure指令应用于sorry_server指令

    alpha  # 在keepalived启动时,假设所有的RS都是down,以及健康检查是失败的,有助于防止启动时的误报,默认是禁用的
    omega  # 在keepalived终止时,会执行quorum_down指令所定义的脚本

    quorum <INT>  # 默认值1,所有的存活的服务器的总的最小权重
    quorum_up <STRING>  # 当quorum增长到满足quorum所定义的值时,执行该脚本
    quorum_down <STRING>  # 当quorum减少到不满足quorum所定义的值时,执行该脚本
}
```



real_server配置段

 

```
real_server IP Port {
    weight <INT> # 给服务器指定权重,默认是1
    inhibit_on_failure # 当服务器健康检查失败时,将其weight设置为0,而不是从Virtual Server中移除
    notify_up <STRING> # 当服务器健康检查成功时,执行的脚本
    notify_down <STRING> # 当服务器健康检查失败时,执行的脚本
    uthreshold <INT> # 到这台服务器的最大连接数
    lthreshold <INT> # 到这台服务器的最小连接数
}
```



real_server中的健康检查

 

```
HTTP_GET | SSL_GET {
    url {
        path <STRING> # 指定要检查的URL的路径,如path / or path /mrtg2
        digest <STRING> # 摘要,校验码,计算方式：genhash -s 172.17.100.1 -p 80 -u /index.html
        status_code <INT> # 状态码
    }
    
    nb_get_retry <INT> # get尝试次数
    delay_before_retry <INT> # 在尝试之前延迟多长时间

    connect_ip <IP ADDRESS> # 连接的IP地址,默认是real server的ip地址
    connect_port <PORT> # 连接的端口,默认是real server的端口
    bindto <IP ADDRESS> # 发起连接的接口的地址
    bind_port <PORT> # 发起连接的源端口
    connect_timeout <INT> # 连接超时时间,默认是5s
    fwmark <INTEGER> # 使用fwmark对所有出去的检查数据包进行标记
    warmup <INT> # 指定一个随机延迟,最大为N秒,可防止网络阻塞,如果为0,则关闭该功能
} 

TCP_CHECK {
    connect_ip <IP ADDRESS> # 连接的IP地址,默认是real server的ip地址
    connect_port <PORT> # 连接的端口,默认是real server的端口
    bindto <IP ADDRESS> # 发起连接的接口的地址
    bind_port <PORT> # 发起连接的源端口
    connect_timeout <INT> # 连接超时时间,默认是5s
    fwmark <INTEGER> # 使用fwmark对所有出去的检查数据包进行标记
    warmup <INT> # 指定一个随机延迟,最大为N秒,可防止网络阻塞,如果为0,则关闭该功能
    retry <INIT> # 重试次数,默认是1次
    delay_before_retry <INT> # 默认是1秒,在重试之前延迟多少秒
}

SMTP_CHECK {
    connect_ip <IP ADDRESS> # 连接的IP地址,默认是real server的ip地址
    connect_port <PORT> # 连接的端口,默认是real server的端口,默认是25端口
    bindto <IP ADDRESS> # 发起连接的接口的地址
    bind_port <PORT> # 发起连接的源端口
    connect_timeout <INT> # 连接超时时间,默认是5s
    fwmark <INTEGER> # 使用fwmark对所有出去的检查数据包进行标记
    warmup <INT> # 指定一个随机延迟,最大为N秒,可防止网络阻塞,如果为0,则关闭该功能

    retry <INT> # 重试次数
    delay_before_retry <INT> # 在重试之前延迟多少秒
    helo_name <STRING> # 用于SMTP HELO请求的字符串
}

DNS_CHECK {
    connect_ip <IP ADDRESS> # 连接的IP地址,默认是real server的ip地址
    connect_port <PORT> # 连接的端口,默认是real server的端口,默认是25端口
    bindto <IP ADDRESS> # 发起连接的接口的地址
    bind_port <PORT> # 发起连接的源端口
    connect_timeout <INT> # 连接超时时间,默认是5s
    fwmark <INTEGER> # 使用fwmark对所有出去的检查数据包进行标记
    warmup <INT> # 指定一个随机延迟,最大为N秒,可防止网络阻塞,如果为0,则关闭该功能

    retry <INT> # 重试次数,默认是3次
    type <STRING> # DNS query type,A/NS/CNAME/SOA/MX/TXT/AAAA
    name <STRING> # DNS查询的域名,默认是(.)
}

MISC_CHECK {
    misc_path <STRING> # 外部的脚本或程序路径
    misc_timeout <INT> # 脚本执行超时时间
    user USERNAME [GROUPNAME] # 指定运行该脚本的用户和组,如果没有指定GROUPNAME,则GROUPNAME同USERNAME
    misc_dynamic # 根据退出状态码动态调整权重
    0，健康检查成功,权重不变
    1，健康检查失败
    2-255，健康检查成功,权重设置为退出状态码减去2,如退出状态码是250,则权重调整为248
    warmup <INT> # 指定一个随机延迟,最大为N秒,可防止网络阻塞,如果为0,则关闭该功能
}
```



主备模式(lvs)的高可用(ha)WEB配置实例

实验环境

Directory:

VIP:172.16.1.100

DIP:

node1:172.16.1.31

node2:172.16.1.32

RS:

node3:172.16.1.33

node4:172.16.1.34

拓扑图如下:

![img](Keepalived%E5%BA%94%E7%94%A8.assets/6b0817e8-6581-4ade-8f28-482523e598df.png)

1,先配置好RS,这里以httpd服务为例.

 

```
vim setip.sh
"""
#!/bin/bash
case $1 in
start)
    echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
    echo 1 > /proc/sys/net/ipv4/conf/lo/arp_ignore
    echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
    echo 2 > /proc/sys/net/ipv4/conf/lo/arp_announce
    ;;
stop)
    echo 0 > /proc/sys/net/ipv4/conf/all/arp_ignore
    echo 0 > /proc/sys/net/ipv4/conf/lo/arp_ignore
    echo 0 > /proc/sys/net/ipv4/conf/all/arp_announce
    echo 0 > /proc/sys/net/ipv4/conf/lo/arp_announce
    ;;
esac
"""

# node3
yum install -y httpd
vim /var/www/html/index.html
"""
from node3.
"""
service httpd start
curl http://172.16.1.33
bash setip.sh start

ifconfig lo:0 172.16.1.100 netmask 255.255.255.255 broadcast 172.16.1.100 up
route add -host 172.16.1.100 dev lo:0
ip addr list
route -n

# node4
yum install -y httpd
vim /var/www/html/index.html
"""
from node4.
"""
service httpd start
curl http://172.16.1.34
bash setip.sh start

ifconfig lo:0 172.16.1.100 netmask 255.255.255.255 broadcast 172.16.1.100 up
route add -host 172.16.1.100 dev lo:0
ip addr list
route -n
```



2,在Director上安装ipvsadm服务,并配置一个vip,为了启用sorry server,还需安装http服务

 

```
# node1
yum install ipvsadm httpd -y
echo "node1,sorry server!" >> /var/www/html/index.html

ip addr add 172.16.1.100/32 dev eth0

ipvsadm -A -t 172.16.1.100:80 -s rr
ipvsadm -a -t 172.16.1.100:80 -r 172.16.1.33 -g -w 1
ipvsadm -a -t 172.16.1.100:80 -r 172.16.1.34 -g -w 2
ipvsadm -L -n

# 测试无误后,清空lvs规则并删除vip
curl http://172.16.1.100
ipvsadm -C
ip addr del 172.16.1.100/32 dev eth0


# node2
yum install ipvsadm httpd -y
echo "node1,sorry server!" >> /var/www/html/index.html

ifconfig eth0:0 172.16.1.100/32 up

ipvsadm -A -t 172.16.1.100:80 -s rr
ipvsadm -a -t 172.16.1.100:80 -r 172.16.1.33 -g -w 1
ipvsadm -a -t 172.16.1.100:80 -r 172.16.1.34 -g -w 2
ipvsadm -L -n

# 测试无误后,清空lvs规则并删除vip
curl http://172.16.1.100
ipvsadm -C
ifconfig eth0:0 down
```



3,安装配置keepalived

notify.sh脚本

 

```
#!/bin/bash
#author :xiaofei
#Description : an example of notify script
vip=172.16.1.100
contact='root@localhost'
notify() {
    mailsubject="$(hostname) to be $1: $vip floating"
    mailbody="$(date +'%F %H:%M:%S'): vrrp transition,$(hostname) change to be $1"
    echo $mailbody | mail -s "$mailsubject" $contact
}
case "$1" in
    master)
        notify master
        exit 0
    ;;
    backup)
        notify backup
        exit 0
    ;;
    fault)
        notify fault
        exit 0
    ;;
    *)
        echo "Usage: $(basename $0) {master|backup|fault}"
        exit 1
    ;;
esac
```



keepalived.conf配置文件

 

```
yum install -y keepalived
cd /etc/keepalived
cp keepalived.conf{,.bak}
vim keepalived.conf
"""
! Configuration File for keepalived

global_defs {
   notification_email {
        root@localhost
   }
   notification_email_from kaadmin@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id node1.com  # node2节点上id为node2.com
   vrrp_mcast_group4 224.18.0.10
}

vrrp_script chk_down {
        script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"
        interval 1
        weight -2
}

vrrp_instance VI_1 {
    state MASTER  # node2节点上为BACKUP
    interface eth0
    virtual_router_id 51
    priority 100  # node2节点上权重低于100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 64e06f86
    }
    virtual_ipaddress {
        172.16.1.100/32
    }
    track_script {
        chk_down
    }
    
    notify_master "/etc/keepalived/notify.sh master"
    notify_backup "/etc/keepalived/notify.sh backup"
    notify_fault "/etc/keepalived/notify.sh fault"
}

virtual_server 172.16.1.100 80 {  # vip
    delay_loop 6
    lb_algo wrr
    lb_kind DR
    nat_mask 255.255.0.0
    protocol TCP
    sorry_server 127.0.0.1 80
    
    real_server 172.16.1.33 80 {  # RIP
        weight 1
        HTTP_GET {
            url {
              path /
              status_code 200
            } 
            #url {  # 检测多个url
            #  path /mrtg/
            #  digest 9b3a0c85a887a256d6939da88aabd8cd
            #} 
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }   
    }
    real_server 172.16.1.34 80 {  # RIP
        weight 2
        HTTP_GET {
            url {
              path /
              status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }   
    }
}
"""
```



4,测试

 

```
# node1
tcpdump -i eht0 -nn host 172.16.1.32
ip addr list
ipvsadm -L -n

touch down
rm -rf down

# node2
tcpdump -i eht0 -nn host 172.16.1.31
ip addr list
ipvsadm -L -n
```



基于Nginx做反向代理,并对Nginx做高可用

拓扑架构如下:

![img](Keepalived%E5%BA%94%E7%94%A8.assets/632a2b79-9da5-42cf-aeb1-e5e219f52884.png)

1,先确保Nginx可反代至后端的2个RS,关键配置如下

 

```
http {
    ...
    upstream websrvs {
        server 172.16.1.33:80 weight=1;
        server 172.16.1.34:80 weight=2;
    }
    server {
        ...
        proxy_pass http://websrvs/;
        ...
    }
    ...
}
```



2,keepalived配置文件加入检测脚本

 

```
vrrp_script chk_nginx {
    script "killall -0 nginx &> /dev/null"  # nginx进程存在,返回0,不存在返回1
    interval 1  # 检测周期
    weight -10  # 存在则权重-10
}

track_script {
    chk_nginx
}
```