[TOC]

# 防火墙和SELinux

```shell
#!/bin/bash
################################################
# Author: xf
# Date: 2018-01-19
# Version: 1.0
# Use: forbidden selinux and iptables
################################################

\cp /etc/selinux/config /etc/selinux/config.$(date +%F)
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
setenforce 0
a=$(/usr/sbin/getenforce)
echo -e "SElinux status: \033[1;32m$a!\033[0m"

sleep 2
/etc/init.d/iptables stop
/sbin/chkconfig iptables off
/sbin/iptables -F
/etc/init.d/iptables status
```



# Yum源

```shell
#!/bin/bash
################################################
# Author: xf
# Date: 2018-01-19
# Version: 1.0
# Use: 配置国内yum源和epel源
################################################
cd /etc/yum.repos.d/
echo -e "\033[1;32mTest network connectivity..\033[0m"
/bin/ping -c 3 mirrors.aliyun.com
if [ $? -eq 0 ];then
    echo -e "\033[1;32mWget mirrors!\033[0m"
    /usr/bin/wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
else
    echo -e "\033[1;31mError!Please check the network.\033[0m"
exit $?
fi

sleep 2

echo -e "\033[1;32mDownload EPEL!\033[0m"
/usr/bin/wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm > /dev/null 2>&1
rpm -ivh epel-release-6-8.noarch.rpm

echo -e "\033[1;32mYum cache!\033[0m"
yum clean all
yum makecache
```



# 精简程序自启

```shell
#!/bin/bash
################################################
# Author: xf
# Date: 2018-01-19
# Version: 1.0
# Use: 优化开机启动项,安装常用软件
################################################
echo -e "\033[1;32mInstall Tools...\033[0m"
yum install -y vim tree nmap sysstat ntpdate lrzsz dos2unix wget > /dev/null 2>&1
rpm -qa vim tree nmap sysstat ntpdate lrzsz dos2unix wget
sleep 2
echo -e "\033[1;32mClose the all self-initiated program..\033[0m"
for i in `chkconfig --list | grep 3:on |awk '{print$1}'`; do
    chkconfig --level 3 $i off
done
sleep 2
echo -e "\033[1;32mBoot up program...\033[0m"
for i in crond sysstat rsyslog network sshd; do
         chkconfig --level 3 $i on
         echo -e "The \033[1;32m$i\033[0m boot up!"
done
```



# SSH优化

```shell
#!/bin/bash
################################################
# Author: xf
# Date: 2018-01-19
# Version: 1.0
# Use: ssh seo!
################################################

\cp /etc/ssh/sshd_config /etc/ssh/sshd_config".bak"
echo -e "\033[1;32mSSH port 2201\033[0m"
sed -i 's/#Port 22/Port 2201/g' /etc/ssh/sshd_config

echo -e "\033[1;32mNo root login.\033[0m"
# sed -i 's/#PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config

echo -e "\033[1;32mNo empty passwords.\033[0m"
sed -i 's/#PermitEmptyPasswords no/PermitEmptyPasswords no/g' /etc/ssh/sshd_config

echo -e "\033[1;32mNo UseDNS.\033[0m"
sed -i 's/#UseDNS yes/UseDNS no/g' /etc/ssh/sshd_config

echo -e "\033[1;32mNo GSSAPI authentication.\033[0m"
sed -i 's/^GSSAPIAuthentication yes$/GSSAPIAuthentication no/' /etc/ssh/sshd_config

sleep 2
service sshd restart
```



# 时间同步

```shell
#!/bin/bash
################################################
# Author: xf
# Date: 2018-01-19
# Version: 1.0
# Use: Time async
################################################
timeServer=ntp1.aliyun.com

\cp /var/spool/cron/root /var/spool/cron/root.$(date +%F) 2>/dev/null
NTPDATE=$(grep ntpdate /var/spool/cron/root 2> /dev/null | wc -l)
if [ $NTPDATE -eq 0 ];then
	/usr/sbin/ntpdate $timeServer
        echo "*/10 * * * * /usr/sbin/ntpdate $timeServer > /dev/null 2>&1" >> /var/spool/cron/root
fi
echo -e "The crontab job:\n\033[1;32m$(crontab -l)\033[0m"
echo -e "The latest time: \033[1;32m$(date +"%Y-%m-%d %H:%M:%S")\033[0m"
```



# 回退显示^H或^?

```shell
[root@mysql scripts]# cat backspace.sh 
#!/bin/bash
################################################
# Author: xf
# Date: 2018-01-19
# Version: 1.0
# Use: stty erase ^H^?
################################################
\cp /root/.bash_profile  /root/.bash_profile_$(date +%F)
erase=`grep -wx "stty erase ^H" /root/.bash_profile |wc -l`
if [ $erase -lt 1 ];then
    echo "stty erase ^H" >>/root/.bash_profile
    echo "stty erase ^?" >>/root/.bash_profile
    source /root/.bash_profile
fi
```



# 内核优化

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
source /etc/profile
echo -e "\033[1;32m$(tail -5 /etc/profile)\033[0m"

echo -e "\033[1;32m设置系统默认语言\033[0m"
# \cp /etc/sysconfig/i18n /etc/sysconfig/i18n.bak
# sed -i 's#LANG="en_US.UTF-8"#LANG="zh_CN.UTF-8#' /etc/sysconfig/i18n
# source /etc/sysconfig/i18n
# echo $LANG
```



# 网络设置脚本

```shell
#!/bin/sh 
#auto Change ip netmask gateway scripts 

cat << EOF 
++++++++自动修改ip和主机名等相关信息+++++++++ 
ETHCONF=/etc/sysconfig/network-scripts/ifcfg-eth0
HOSTS=/etc/hosts
NETWORK=/etc/sysconfig/network
#DIR=/data/backup/`date +%Y%m%d` 
NETMASK=255.255.255.0 
+++++++++-------------------------+++++++++++ 
EOF 
#Define Path 定义变量，可以根据实际情况修改 

ETHCONF=/etc/sysconfig/network-scripts/ifcfg-eth0
ETHBATH=/etc/sysconfig/network-scripts/
HOSTS=/etc/hosts
NETWORK=/etc/sysconfig/network
DIR=/data/backup/`date +%Y%m%d`
NETMASK=255.255.255.0 

echo "================================================"

#定义change_ip函数 
function Change_ip () 
{ 
    echo -e "\033[1;32m 备份网卡文件ifcfg-eth0为ifcfg-eth0.bak \033[0m"
    cp $ETHCONF $ETHCONF".bak"

    # 上一次执行命令成功为0,失败为1
    grep "dhcp" $ETHCONF
    if [ $? -eq 0 ];then
        read -p "PLZ enter IP: " IPADDR 
        sed -i 's/dhcp/static/g' $ETHCONF
        
        # awk -F. 意思是以.号为分隔域，打印前三列$1,$2,$3
        echo -e "IPADDR=$IPADDR\nNETMASK=$NETMASK\nGATEWAY=`echo $IPADDR|awk -F. '{print $1"."$2"."$3}'`.254" >> $ETHCONF 
        echo -e "IP address change \033[1;32msuccess\033[0m !"
    else
        echo -e "BOOTPROTO was \033[1;32mstatic\033[0m mode, Do you want to set ip? [y or n] ": 
        read i 
    fi

    if [ "$i" == "y" ];then
        read -p "PLZ enter IP: " IPADDR
        
        # 将输入的ip地址,以.为分隔符,打印出每一列
        count=(`echo $IPADDR | awk -F. '{print $1,$2,$3,$4}'`)
        
        #定义数组,${#count[@]} 代表获取变量值总个数# 
        A=${#count[@]}
        
        # while条件语句判断，个数是否正确，不正确循环提示输入，也可以用[0-9]来判断ip
        while [ "$A" -ne "4" ]; do
            read -p "Please reEnter ip Address[例如:192.168.0.1]": IPADDR 
            count=(`echo $IPADDR | awk -F. '{print $1,$2,$3,$4}'`) 
            A=${#count[@]} 
        done
        
        # sed -e 可以连续修改多个参数 
        sed -i -e 's/^IPADDR/#IPADDR/g' -e 's/^NETMASK/#NETMASK/g' -e 's/^GATEWAY/#GATEWAY/g' $ETHCONF 
        
        # echo -e 为连续追加内容,\n自动换行
        echo -e "IPADDR=$IPADDR\nNETMASK=$NETMASK\nGATEWAY=`echo $IPADDR|awk -F. '{print $1"."$2"."$3}'`.254" >> $ETHCONF 
        
        echo "This IP address Change \033[1;32msuccess\033[0m !"
        
    else
        echo "This \033[1;32mstatic\033[0m$ETHCONF exist, please exit."
        exit $? 
    fi
} 

# 定义hosts函数 
function Change_hosts () 
{
    if
        [ ! -d $DIR ];then
        mkdir -p $DIR 
    fi

    cp $HOSTS $HOSTS".BAK"
    read -p "Please insert ip address": IPADDR 

    # 将输入的ip地址转换为192-168-1-1形式
    host=`echo $IPADDR | sed 's#\.#-#g'`
    
    # 判断/etc/hosts文件中是否存在输入的ip
    cat $HOSTS | grep 127.0.0.1 | grep "$host"

    # 如果存在则返回0,不存在则返回非0
    if [ $? -ne 0 ]; then
        sed -i "s/127.0.0.1/127.0.0.1    $host/g" $HOSTS 
        echo "This hosts change \033[1;32msuccess\033[0m "
    else
        echo "This $host is \033[1;32mExist\033[0m .........."
    fi
}

#定义network函数 
# function Change_network ()
# {
    # cp $NETWORK $NETWORK".BAK"
    
    # read -p "Please insert ip address": IPADDR 
    # host=`echo $IPADDR|sed 's/\./-/g'` 
    # grep "$host" $NETWORK 
    
    # if [ $? -ne 0 ]; then
        # sed -i "s/^HOSTNAME/#HOSTNAME/g" $NETWORK 
        # echo "NETWORK=$host" >> $NETWORK
    # else
        # echo "This $host IS Exist .........."
    # fi

# }

#PS3一般为菜单提示信息
PS3="Please Select ip or hosts Menu":

#select为菜单选择命令，格式为select $var in ..command.. do .... done 
select i in "Change_ip" "Change_hosts" "Change_network"; do
    
    #case 方式，一般用于多种条件下的判断 
    case $i in
        Change_ip ) 
        Change_ip 
        ;;
        
        Change_hosts ) 
        Change_hosts 
        ;;
        
        Change_network ) 
        Change_network 
        ;;
        
        *) 
        echo
        echo "Please Insert $0: Change_ip(1)|Change_hosts(2)|Change_network(3)"
        echo
        ;; 
    esac 
done
```



# 一键系统优化脚本

```shell
!/bin/sh
################################################
Author:xf
Date: 2017-10-10
version:1.0
#实现功能：系统优化脚本，适用于Centos6.x
################################################

#Source function library.
. /etc/init.d/functions

#date
DATE=`date +"%y-%m-%d %H:%M:%S"`

#ip
IPADDR=`grep "IPADDR" /etc/sysconfig/network-scripts/ifcfg-eth0 | cut -d= -f 2 `

#hostname
HOSTNAME=`hostname -s`

#user
USER=`whoami`

#disk_check
DISK_SDA=`df -h |grep -w "/" |awk '{print $5}'`

#cpu_average_check
cpu_uptime=`cat /proc/loadavg|awk '{print $1,$2,$3}'`

#set LANG
export LANG=zh_CN.UTF-8

# network path
ETHCONF=/etc/sysconfig/network-scripts/ifcfg-eth0
NETMASK=255.255.255.0

# time server
timeServer=ntp1.aliyun.com

#Require root to run this script.
uid=`id | cut -d '(' -f1 | cut -d '=' -f2`
if [ $uid -ne 0 ];then
  action "Please run this script as root." /bin/false
  exit 1
fi

#stty erase ^H
\cp /root/.bash_profile  /root/.bash_profile_$(date +%F)
erase=`grep -wx "stty erase ^H" /root/.bash_profile |wc -l`
erase=`grep -wx "stty erase ^?" /root/.bash_profile |wc -l`
if [ $erase -lt 1 ];then
    echo "stty erase ^H" >>/root/.bash_profile
    source /root/.bash_profile
fi

#set_hostname
set_Hostname(){
        echo -e "\033[1;32m=========请输入主机名===========\033[0m"
        read -p "Please enter hostname: " new_hostname
        old_hostname=$(grep "HOSTNAME" /etc/sysconfig/network)
        sed -i "s/$old_hostname/HOSTNAME=$new_hostname/g" /etc/sysconfig/network
        echo -e "The hostname is changed \033[1;32m$new_hostname\033[0m."
        echo -e "\033[1;32m================================\033[0m"
        sleep 2
}

#set_network 
set_Network (){ 
    echo -e "\033[1;32m备份网卡文件ifcfg-eth0为ifcfg-eth0.bak\033[0m"
    cp $ETHCONF $ETHCONF".bak"

    # 上一次执行命令成功为0,失败为1
    grep "dhcp" $ETHCONF
    if [ $? -eq 0 ];then
        read -p "PLZ enter IP: " IPADDR 
        sed -i 's/dhcp/static/g' $ETHCONF
        
        # awk -F. 意思是以.号为分隔域，打印前三列$1,$2,$3
        echo -e "IPADDR=$IPADDR\nNETMASK=$NETMASK\nGATEWAY=`echo $IPADDR|awk -F. '{print $1"."$2"."$3}'`.254" >> $ETHCONF 
        echo -e "IP address change \033[1;32msuccess\033[0m !"
    else
        echo -e "BOOTPROTO was \033[1;32mstatic\033[0m mode, Do you want to set ip? [y or n] ": 
        read i 
    fi

    if [ "$i" == "y" ];then
        read -p "PLZ enter IP: " IPADDR
        
        # 将输入的ip地址,以.为分隔符,打印出每一列
        count=(`echo $IPADDR | awk -F. '{print $1,$2,$3,$4}'`)
        
        #定义数组,${#count[@]} 代表获取变量值总个数# 
        A=${#count[@]}
        
        # while条件语句判断，个数是否正确，不正确循环提示输入，也可以用[0-9]来判断ip
        while [ "$A" -ne "4" ]; do
            read -p "Please reEnter ip Address[例如:192.168.0.1]": IPADDR 
            count=(`echo $IPADDR | awk -F. '{print $1,$2,$3,$4}'`) 
            A=${#count[@]} 
        done
        
        # sed -e 可以连续修改多个参数 
        sed -i -e 's/^IPADDR/#IPADDR/g' -e 's/^NETMASK/#NETMASK/g' -e 's/^GATEWAY/#GATEWAY/g' $ETHCONF 
        
        # echo -e 为连续追加内容,\n自动换行
        echo -e "IPADDR=$IPADDR\nNETMASK=$NETMASK\nGATEWAY=`echo $IPADDR|awk -F. '{print $1"."$2"."$3}'`.254" >> $ETHCONF 
        
        echo -e "This IP address Change \033[1;32msuccess\033[0m !"
        
    else
        echo -e "This \033[1;32mstatic\033[0m$ETHCONF exist, please exit."
        exit $? 
    fi
}

#Config Yum CentOS-Bases.repo and save Yum file
configYum(){
        echo -e "\033[1;32m=========配置本地YUM源==========\033[0m"
        if [ -f /etc/yum.repos.d/bak ]; then
            mv CentOS-* bak
        else
            mkdir bak && mv CentOS-* bak
        fi
        touch Media.repo
        cat >> /etc/yum.repos.d/media.repo << EOF
[cdrom]
name=local_cdrom
baseurl=file:///media/cdrom
enabled=1
gpgcheck=0
EOF
        echo "/dev/cdrom    /media/cdrom    ext4    defaults    0 0" >> /etc/fstab
        echo "\033[1;32m===================================\033[0m"
        echo ""
        sleep 2
}

#install tools
initTools(){
        echo -e "\033[1;32m========安装系统常用工具(选择最小化安装minimal)=========\033[0m"
        # ping -c 2 mirrors.aliyun.com
        sleep 2
        yum install vim tree nmap sysstat ntpdate lrzsz dos2unix wget -y
        sleep 2
        rpm -qa vim tree nmap sysstat ntpdate lrzsz dos2unix wget
        sleep 2
        action "安装系统常用工具(适用于最小化安装minimal)" /bin/true
        echo -e "\033[1;32m===================================\033[0m"
        echo ""
        sleep 2
}

#Config Yum CentOS-Bases.repo and save Yum file
# configYum(){
        # echo -e "\033[1;32m================更新为国内YUM源==================\033[0m"
        # cd /etc/yum.repos.d/
        # \cp CentOS-Base.repo CentOS-Base.repo.$(date +%F)
        # ping -c 1 mirrors.aliyun.com >/dev/null
        # if [ $? -eq 0 ];then
                # wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
        # else
                # echo -e "\033[1;32m无法连接网络\033[0m"
                # exit $?
        # fi

        # echo "==============设置保存下载的YUM源文件================="
        # sed -i 's#keepcache=0#keepcache=1#g' /etc/yum.conf
        # grep keepcache /etc/yum.conf
        # sleep 5

        # action "配置国内YUM完成"  /bin/true
        # echo "================================================="
        # echo ""
        # sleep 2
# }

#Charset zh_CN.UTF-8
initI18n(){
        echo -e "\033[1;32m================更改为中文字符集=================\033[0m"
        \cp /etc/sysconfig/i18n /etc/sysconfig/i18n.$(date +%F)
        > /etc/sysconfig/i18n
        cat >> /etc/sysconfig/i18n <<EOF
        LANG="zh_CN.UTF-8"
        #LANG="en_US.UTF-8"
        SYSFONT="latarcyrheb-sun16"
EOF
        source /etc/sysconfig/i18n
        echo '#cat /etc/sysconfig/i18n'
        grep LANG /etc/sysconfig/i18n
        action "更改字符集zh_CN.UTF-8完成" /bin/true
        echo "================================================="
        echo ""
        sleep 2
}

#Close Selinux and Iptables
initFirewall(){
        echo -e "\033[1;32m============禁用SELINUX及关闭防火墙==============\033[0m"
        \cp /etc/selinux/config /etc/selinux/config.$(date +%F)
        sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
        setenforce 0

        /etc/init.d/iptables stop
        /sbin/chkconfig iptables off
        /sbin/iptables -F
        /etc/init.d/iptables status
        echo '#grep SELINUX=disabled /etc/selinux/config'
        grep SELINUX=disabled /etc/selinux/config
        echo '#getenforce'
        getenforce
        action "禁用selinux及关闭防火墙完成" /bin/true
        echo "================================================="
        echo ""
        sleep 2
}

#Init Auto Startup Service
initService(){
        echo -e "\033[1;32m===============精简开机自启动====================\033[0m"
        export LANG="en_US.UTF-8"

        for A in `chkconfig --list |grep 3:on |awk '{print $1}' `;do
                chkconfig $A off;
        done

        for B in rsyslog network sshd crond;do
                chkconfig $B on;
        done

        echo '+--------which services on---------+'
        chkconfig --list |grep 3:on
        echo '+----------------------------------+'
        export LANG="zh_CN.UTF-8"
        action "精简开机自启动完成" /bin/true
        echo "================================================="
        echo ""
        sleep 2
}

#Removal system and kernel version login before the screen display
initRemoval(){
        echo -e "\033[1;32m======去除系统及内核版本登录前的屏幕显示=======\033[0m"

        #must use root user run scripts
        if [ $UID -ne 0 ];then
                echo -e "This script must use the \033[1;32mroot\033[0m user !!!" 
                sleep 2
                exit 0
        fi
        >/etc/redhat-release
        >/etc/issue
        action "去除系统及内核版本登录前的屏幕显示" /bin/true
        echo "================================================="
        echo ""
        sleep 2
}

#Change sshd default port and prohibit user root remote login.
initSsh(){
        \cp /etc/ssh/sshd_config /etc/ssh/sshd_config".bak"
        echo -e "\033[1;32m========修改ssh默认端口为22==========\033[0m"
        sed -i 's/#Port 22/Port 22/g' /etc/ssh/sshd_config

        #echo "========禁用root远程登录==============="
        #sed -i 's/#PermitEmptyPasswords no/PermitEmptyPasswords no/g' /etc/ssh/sshd_config
        #sed -i 's/#PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config
        
        echo -e "\033[1;32m========禁用DNS反向解析===============\033[0m"
        sed -i 's/#UseDNS yes/UseDNS no/g' /etc/ssh/sshd_config
        
        echo -e "========禁用GSSAPI认证=========="
        sed -i 's/^GSSAPIAuthentication yes$/GSSAPIAuthentication no/' /etc/ssh/sshd_config
        service sshd restart
        
        echo -e '\033[1;32m+-------modify the sshd_config-------+\033[0m'
        echo -e '\033[1;32mPort 22\033[0m'
        #echo 'PermitEmptyPasswords no'
        #echo 'PermitRootLogin no'
        echo -e '\033[1;32mUseDNS no\033[0m'
        echo -e '\033[1;32mGSSAPIAuthentication no\033[0m'
        echo '+------------------------------------+'
        /etc/init.d/sshd reload && action "修改ssh默认参数完成" /bin/true || action "修改ssh参数失败" /bin/false
        echo "================================================="
        echo ""
        sleep 2
}

#time sync
syncSysTime(){
        echo -e "\033[1;32m================配置时间同步=====================\033[0m"
        \cp /var/spool/cron/root /var/spool/cron/root.$(date +%F) 2>/dev/null
        NTPDATE=`grep ntpdate /var/spool/cron/root 2> /dev/null | wc -l`
        if [ $NTPDATE -eq 0 ];then
                echo "#times sync by lee at $(date +%F)" >> /var/spool/cron/root
                echo "*/5 * * * * /usr/sbin/ntpdate $timeServer > /dev/null 2>&1" >> /var/spool/cron/root
        fi
        echo '#crontab -l'  
        crontab -l
        action "配置时间同步完成" /bin/true
        echo "================================================="
        echo ""
        sleep 2
}

#add user and give sudoers
addUser(){
        echo -e "\033[1;32m===================新建用户======================\033[0m"
        #add user
        while true;do
                read -p "请输入新用户名:" name
                NAME=`awk -F':' '{print $1}' /etc/passwd | grep -wx $name 2> /dev/null | wc -l`
                if [ ${#name} -eq 0 ];then
                        echo "用户名不能为空，请重新输入。"
                        continue
                elif [ $NAME -eq 1 ];then
                        echo "用户名已存在，请重新输入。"
                        continue
                fi
                useradd $name
                break
        done
        #create password
        while true;do
                read -p "为 $name 创建一个密码:" pass1
                if [ ${#pass1} -eq 0 ];then
                        echo "密码不能为空，请重新输入。"
                        continue
                fi
                read -p "请再次输入密码:" pass2
                if [ "$pass1" != "$pass2" ];then
                        echo "两次密码输入不相同，请重新输入。"
                        continue
                fi
                echo "$pass2" | passwd --stdin $name
                break
        done
        sleep 1

        #add visudo
        echo -e "\033[1;32m##### add visudo #####\033[0m"
        \cp /etc/sudoers /etc/sudoers.$(date +%F)
        SUDO=`grep -w "$name" /etc/sudoers |wc -l`
        if [ $SUDO -eq 0 ];then
                echo "$name  ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers
                echo '#tail -1 /etc/sudoers'
                grep -w "$name" /etc/sudoers
                sleep 1
        fi
        action "创建用户$name并将其加入visudo完成"  /bin/true
        echo "================================================="
        echo ""
        sleep 2
}

#Adjust the file descriptor(limits.conf)
initLimits(){
        echo -e "\033[1;32m===============加大文件描述符=================\033[0m"
        LIMIT=`grep nofile /etc/security/limits.conf | grep -v "^#"| wc -l`
        if [ $LIMIT -eq 0 ];then
                \cp /etc/security/limits.conf /etc/security/limits.conf.$(date +%F)
                echo '*                  -        nofile         65535' >> /etc/security/limits.conf
        fi
        echo '#tail -1 /etc/security/limits.conf'
        tail -1 /etc/security/limits.conf
        ulimit -HSn 65535
        echo '#ulimit -n'
        ulimit -n
        action "配置文件描述符为65535" /bin/true
        echo "================================================="
        echo ""
        sleep 2
}

#set the control-alt-delete to guard against the miSUSE
initRestart(){
        echo -e "\033[1;32m===============禁用ctrl+alt+delete功能键=================\033[0m"
        sed -i 's#exec /sbin/shutdown -r now#\#exec /sbin/shutdown -r now#' /etc/init/control-alt-delete.conf
        action "将ctrl + alt + delete键进行屏蔽，防止误操作的时候服务器重启" /bin/true
        echo "================================================="
        echo ""
        sleep 2
}

#Optimizing the system kernel
initSysctl(){
        echo "\033[1;32m==============优化内核参数===================\033[0m"
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
        modprobe nf_conntrack
        echo "modprobe nf_conntrack" >> /etc/rc.local
        modprobe bridge
        echo "modprobe bridge" >> /etc/rc.local
        sysctl -p
        action "内核调优完成" /bin/true
        echo "================================================="
        echo ""
        sleep 2
}

#setting history and login timeout
initHistory(){
        echo -e "\033[1;32m======设置默认历史记录数和连接超时时间======\033[0m"
        echo "TMOUT=600" >> /etc/profile
        echo "HISTSIZE=10" >> /etc/profile
        echo "HISTFILESIZE=5" >> /etc/profile
        tail -3 /etc/profile
        source /etc/profile
        action "设置默认历史记录数和连接超时时间" /bin/true
        echo "================================================="
        echo ""
        sleep 2
}

#chattr file system
# initChattr(){
    # echo -e "\033[1;32m======锁定关键文件系统======\033[0m"
    # chattr +i /etc/passwd
    # chattr +i /etc/inittab
    # chattr +i /etc/group
    # chattr +i /etc/shadow
    # chattr +i /etc/gshadow
    # /bin/mv /usr/bin/chattr /usr/bin/lock
    # action "锁定关键文件系统" /bin/true
    # echo "================================================="
    # echo ""
    # sleep 2
# }

#menu2
menu2(){
        while true;do
                clear
                cat << EOF
----------------------------------------
|****Please Enter Your Choice:[0-15]****|
----------------------------------------
(1)  设置主机名和网络
(2)  配置本地yum源
(3)  安装常用工具
(4)  更改系统字符集为中文
(5)  禁用SELINUX及关闭防火墙,
(6)  精简开机自启动
(7)  去除系统及内核版本登录前的屏幕显示
(8)  SSH连接优化
(9)  设置时间同步
(10) 建立用户并授权
(11) 加大文件描述符
(12) 屏蔽ctrl+alt+delete键,防止误操作重启
(13) 系统内核调优
(14) 设置默认历史记录数和连接超时时间
(15) 锁定关键文件系统
(0)  返回上一级菜单
EOF
read -p "Please enter your Choice[0-15]: " input2
case "$input2" in
        0)
                clear
                break
                ;;
        1)
                set_Hostname
                set_Network
                ;;
        2)
                configYum
                ;;
        3)
                initTools
                ;;
        4)
                initI18n
                ;;
        5)
                initFirewall
                ;;
        6)
                initService
                ;;
        7)
                initRemoval
                ;;
        8)
                initSsh
                ;;
        9)
                syncSysTime
                ;;
        10)
                addUser
                ;;
        11)
                initLimits
                ;;
        12)
                initRestart
                ;;
        13)
                initSysctl
                ;;
        14)
                initHistory
                ;;
        15)
                initChattr
                ;;
        *)
                echo "----------------------------------"
                echo "|          Warning!!!            |"
                echo "|   Please Enter Right Choice!   |"
                echo "----------------------------------"
                for i in `seq -w 3 -1 1`;do
                        echo -ne "\b\b$i";
                        sleep 1;
                done
                clear
esac
done
}

#initTools
#menu
while true;do
        clear
        echo "========================================"
        echo '          Linux Optimization            '   
        echo "========================================"
        
        # init_sys_info
        # set_Hostname
        # set_Network
        
        cat << EOF
 |-----------System Infomation-----------
 | DATE       :$DATE
 | HOSTNAME   :$HOSTNAME
 | USER       :$USER
 | IP         :$IPADDR
 | DISK_USED  :$DISK_SDA
 | CPU_AVERAGE:$cpu_uptime
 ----------------------------------------
 |****Please Enter Your Choice:[1-3]****|
 ----------------------------------------
 (1) 一键优化
 (2) 自定义优化
 (3) 退出
EOF

#choice
read -p "Please enter your choice[0-3]: " input1

case "$input1" in
        1)
                set_Hostname
                set_Network
                configYum
                initTools
                initI18n
                initFirewall
                initService
                initRemoval
                initSsh
                syncSysTime
                addUser
                initLimits
                initRestart
                initSysctl
                initHistory
                initChattr
                ;;

        2)
                menu2
                ;;
        3)
                clear
                break
                ;;
        *)
                 echo "----------------------------------"
                 echo "|          Warning!!!            |"
                 echo "|   Please Enter Right Choice!   |"
                 echo "----------------------------------"
                 for i in `seq -w 3 -1 1`;do
                        echo -ne "\b\b$i";
                        sleep 1;
                 done
                 clear
esac
done
```