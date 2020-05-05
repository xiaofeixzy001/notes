[TOC]





# 常用命令

## firewalld服务

```shell
# systemctl start firewalld
# systemctl stop firewalld
# systemctl status firewalld
# systemctl disable firewalld
# systemctl enable firewalld
```

systemctl是CentOS7的服务管理工具中主要的工具，它融合之前service和chkconfig的功能于一体。

启动一个服务：systemctl start firewalld.service
关闭一个服务：systemctl stop firewalld.service
重启一个服务：systemctl restart firewalld.service
显示一个服务的状态：systemctl status firewalld.service
在开机时启用一个服务：systemctl enable firewalld.service
在开机时禁用一个服务：systemctl disable firewalld.service
查看服务是否开机启动：systemctl is-enabled firewalld.service
查看已启动的服务列表：systemctl list-unit-files|grep enabled
查看启动失败的服务列表：systemctl —failed

## firewalld-cmd

```shell
firewall-cmd --version //查看版本
firewall-cmd --help // 查看帮助
firewall-cmd --state // 显示状态
firewall-cmd --get-services //获取支持服务列表（firewalld内置服务支持）
firewall-cmd --zone=public --list-ports //查看所有打开的端口
firewall-cmd --list-forward-ports //查看转发的端口
fierewall-cmd –reload //重新加载防火墙策略
firewall-cmd --get-active-zones //查看区域信息
firewall-cmd --list-all-zones //列出全部启用的区域的特性
firewall-cmd --list-services //显示防火墙当前服务
firewall-cmd --get-zone-of-interface=eth0 //查看指定接口所属区域
firewall-cmd --panic-on //拒绝所有包
firewall-cmd --panic-offfir //取消拒绝状态
firewall-cmd --query-panic //查看是否拒绝
# 《实例二》 运行时区域策略设置示例（注：以下示例不加zone的为默认区域public）
firewall-cmd --add-service=ssh //允许 ssh 服务通过
firewall-cmd --remove-service=ssh //禁止 ssh 服务通过
firewall-cmd --add-service=samba --timeout=600 //临时允许 samba 服务通过 600 秒
firewall-cmd --add-service=http --zone=work //允许http服务通过work区域
firewall-cmd --zone=work --add-service=http //从work区域打开http服务
firewall-cmd --zone=internal --add-port=443/tcp //打开 443/tcp 端口在内部区域（internal）
firewall-cmd --zone=internal --remove-port=443/tcp //关闭443/tcp 端口在内部区域（internal）
firewall-cmd --add-interface=eth0 //打开网卡eth0
firewall-cmd --remove-interface=eth0 //关闭网卡eth0
#《 实例三》 永久区域策略设置示例（注：以下示例不加zone的为默认区域public；永久设置均需重新加载防火墙策略或重启系统）
firewall-cmd --permanent --add-service=ftp //永久允许 ftp 服务通过
firewall-cmd --permanent --remove-service=ftp //永久禁止 ftp 服务通过
firewall-cmd --permanent --add-service=http --zone=external //永久允许http服务通过external区域
firewall-cmd --permanent --add-service=http --zone=external //永久允许http服务通过external区域
firewall-cmd --permanent --zone=internal --add-port=111/tcp //打开 111/tcp 端口在内部区域（internal）
firewall-cmd --permanent --zone=internal --remove-port=111/tcp //关闭 111/tcp 端口在内部区域（internal）
firewall-cmd --permanent --add-interface=eth0 //永久打开网卡eth0
firewall-cmd --permanent --remove-interface=eth0 //永久关闭网卡eth0
firewall-cmd --permanent --zone=public --add-port=8080-8083/tcp //添加多个端口
firewall-cmd --permanent --zone=public --remove-port=81/tcp //删除某个端口
firewall-cmd --permanent --remove-rich-rule="rule family="ipv4" source address="192.168.1.51" accept" //删除某个IP
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.0.0/16" accept" //
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.1.0/24" port protocol="tcp" port="9200" accept" //针对某个ip段访问
firewall-cmd --reload //添加操作后别忘了执行重载才会生效
```

