[TOC]

# 修改主机名

```shell
[root@centos7 ~]# hostnamectl set-hostname mariadb
[root@centos7 ~]# vim /etc/hostname
```



# 关闭防火墙

```shell
[root@centos7 ~]# iptables -F
[root@centos7 ~]# iptabls -L
[root@centos7 ~]# systemctl stop firewalld
[root@centos7 ~]# systemctl disable firewalld
```



# yum源

```shell
# 本地源
[root@centos7 ~]# cd /etc/yum.repo.d/
[root@centos7 yum.repos.d]# vim media.repo
"""
[base]
name=media
baseurl=file:///media/cdrom
enabled=1
gpgcheck=0
"""

[root@centos7 yum.repos.d]# mkdir /media/cdrom
[root@centos7 yum.repos.d]# mount /dev/cdrom /media/cdrom

# 国内yum源
[root@centos7 yum.repos.d]# wget http://mirrors.aliyun.com/repo/Centos-7.repo

# epel源
[root@centos7 yum.repos.d]# wget -O /etc/yum.repos.d/epel-7.repo http://mirrors.aliyun.com/repo/epel-7.repo

[root@centos7 yum.repos.d]# yum clean all
[root@centos7 yum.repos.d]# yum makecache
[root@centos7 yum.repos.d]# yum repolist enabled
[root@centos7 yum.repos.d]# yum repolist all
```



# 配置网络

```shell
[root@centos7 ~]# vim /etc/sysconfig/network-scripts/ifcfg-eno16777728
"""
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=eno16777728
UUID=f6e84975-d398-4925-bbbc-883aa2735a87
DEVICE=eno16777728
ONBOOT=yes
IPADDR=172.16.0.1
GATEWAY=172.16.0.1
NETMASK=255.255.0.0
DNS1=172.16.0.1
"""

[root@centos7 ~]# systemctl restart network.service
[root@centos7 ~]# service network restart
[root@centos7 ~]# ip addr
```



# 关闭SELINUX

```shell
[root@centos7 ~]# vim /etc/selinux/config
"""
#SELINUX=enforcing
#SELINUXTYPE=targeted 
SELINUX=disabled
"""
```



# 时间同步

```shell
[root@centos7 ~]# echo "*/60 * * * * /usr/sbin/ntpdate ntp1.aliyun.com" >> /var/spool/cron/root
[root@centos7 ~]# crontab -l
```

时间同步服务器: 

国官网：http://ntp.org.cn/index.html

NTP 服务器：cn.ntp.org.cn

来自阿里云的 NTP 服务器：

ntp1.aliyun.com

ntp2.aliyun.com

ntp3.aliyun.com

ntp4.aliyun.com

ntp5.aliyun.com

ntp6.aliyun.com

ntp7.aliyun.com

# 修改系统语言

```shell
[root@centos7 ~]# locale    # 查看系统语言
[root@centos7 ~]# vim /etc/locale.conf
"""
LANG="zh_CN.GB18030" # 英文en_US.UTF-8
LANGUAGE="zh_CN.GB18030:zh_CN.GB2312:zh_CN" 
SUPPORTED="zh_CN.UTF8:zh_CN:zh:en_US.UTF-8:en_US:en" 
SYSFONT="lat0-sun16"
"""

[root@centos7 ~]# yum -y groupinstall chinese-support
```



# alias

```shell
[root@centos7 ~]# vim .bashrc
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
alias cdnet='cd /etc/sysconfig/network-scripts'
```



# 常用工具

```shell
[root@centos7 ~]# yum install -y vim tree nmap sysstat ntpdate lrzsz dos2unix wget net-tools
```



# 'Backspace'键显示'^#'

```shell
[root@centos7 ~]# echo "stty erase ^H" >>/root/.bash_profile
[root@centos7 ~]# echo "stty erase ^?" >>/root/.bash_profile
[root@centos7 ~]# source /root/.bash_profile
```



# SSH优化

```shell
[root@centos7 ~]# cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
[root@centos7 ~]# sed -i 's/#Port 22/Port 2201/g' /etc/ssh/sshd_config
[root@centos7 ~]# sed -i 's/#PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config
[root@centos7 ~]# sed -i 's/#PermitEmptyPasswords no/PermitEmptyPasswords no/g' /etc/ssh/sshd_config
[root@centos7 ~]# sed -i 's/#UseDNS yes/UseDNS no/g' /etc/ssh/sshd_config
[root@centos7 ~]# sed -i 's/^GSSAPIAuthentication yes$/GSSAPIAuthentication no/' /etc/ssh/sshd_config
[root@centos7 ~]# systemctl restart sshd
```

GSSAPI ( Generic Security Services Application Programming Interface)  是一套类似Kerberos 5  的通用网络安全系统接口。该接口是对各种不同的客户端服务器安全机制的封装，以消除安全接口的不同，降低编程难度。但该接口在目标机器无域名解析时会有问题





# 系统参数优化

```shell
#!/bin/bash
################################################
# Author: xf
# Date: 2018-01-19
# Version: 1.0
# Use: later
################################################

echo -e "\033[1;32m加大文件描述符\033[0m"
LIMIT=`grep nofile /etc/security/limits.conf | grep -v "^#"| wc -l`
if [ $LIMIT -eq 0 ];then
        \cp /etc/security/limits.conf /etc/security/limits.conf.$(date +%F)
        echo '*                  -        nofile         65535' >> /etc/security/limits.conf
fi
ulimit -HSn 65535
echo -e "\033[1;32m当前文件描述符大小:$(ulimit -n)\033[0m"

sleep 2

echo -e "\033[1;32m去除登录屏幕显示系统信息\033[0m"
user_id=$(id | cut -d '(' -f1 | cut -d '=' -f2)
if [ $user_id -ne 0 ];then
        echo -e "This script must use the \033[1;32mroot\033[0m user." 
        sleep 2
        exit 0
fi
\cp /etc/redhat-release /etc/redhat-release.$(date +%F)
\cp /etc/issue /etc/issue.$(date +%F)
>/etc/redhat-release
>/etc/issue

sleep 2

echo -e "\033[1;32m禁用ctrl+alt+delete功能键\033[0m"
sed -i 's#exec /sbin/shutdown -r now#\#exec /sbin/shutdown -r now#' /etc/init/control-alt-delete.conf

sleep 2

echo -e "\033[1;32m优化内核参数\033[0m"
SYSCTL=`grep "net.ipv4.tcp" /etc/sysctl.conf |wc -l`
if [ $SYSCTL -lt 10 ];then
                \cp /etc/sysctl.conf /etc/sysctl.conf.$(date +%F)
                cat >> /etc/sysctl.conf << EOF
net.ipv4.tcp_fin_timeout = 2
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_keepalive_time = 600
net.ipv4.ip_local_port_range = 4000 65000
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.route.gc_timeout = 100
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_synack_retries = 1
net.core.somaxconn = 16384
net.core.netdev_max_backlog = 16384
net.ipv4.tcp_max_orphans = 16384
net.netfilter.nf_conntrack_max = 25000000
net.netfilter.nf_conntrack_tcp_timeout_established = 180
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 120
net.netfilter.nf_conntrack_tcp_timeout_close_wait = 60
net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 120
EOF
fi

\cp /etc/rc.local /etc/rc.local.$(date +%F)
/sbin/modprobe nf_conntrack
echo "modprobe nf_conntrack" >> /etc/rc.local
modprobe bridge
echo "modprobe bridge" >> /etc/rc.local
echo -e "\033[1;32m$(sysctl -p)\033[0m"

sleep 2

echo -e "\033[1;32m设置默认历史记录数和连接超时时间\033[0m"
echo "TMOUT=3600" >> /etc/profile
echo "HISTSIZE=100" >> /etc/profile
echo "HISTFILESIZE=50" >> /etc/profile
echo "HISTCONTROL=ignorespace" >> /etc/profile
source /etc/profile
echo -e "\033[1;32m$(tail -5 /etc/profile)\033[0m"

echo -e "\033[1;32m设置系统默认语言\033[0m"
# \cp /etc/sysconfig/i18n /etc/sysconfig/i18n.bak
# sed -i 's#LANG="en_US.UTF-8"#LANG="zh_CN.UTF-8#' /etc/sysconfig/i18n
# source /etc/sysconfig/i18n
# echo $LANG

sleep 2
```

以下参数是对iptables防火墙的优化，防火墙不开会提示，可以忽略不理。

net.nf_conntrack_max = 25000000

net.netfilter.nf_conntrack_max = 25000000

net.netfilter.nf_conntrack_tcp_timeout_established = 180

net.netfilter.nf_conntrack_tcp_timeout_time_wait = 120

net.netfilter.nf_conntrack_tcp_timeout_close_wait = 60

net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 120

如果优化完内核有错误1

error: "net.bridge.bridge-nf-call-ip6tables" is an unknown key

error: "net.bridge.bridge-nf-call-iptables" is an unknown key

error: "net.bridge.bridge-nf-call-arptables" is an unknown key

解决办法：

\#modprobe nf_conntrack

\#echo "modprobe nf_conntrack" >> /etc/rc.local

错误2

error: "net.bridge.bridge-nf-call-ip6tables" is an unknown key

error: "net.bridge.bridge-nf-call-iptables" is an unknown key

error: "net.bridge.bridge-nf-call-arptables" is an unknown key

解决办法：

\#modprobe bridge

\#echo "modprobe bridge" >> /etc/rc.local