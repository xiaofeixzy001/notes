[TOC]

# 1,Docker安装 

## - Docker官网

https://www.docker.com/

## - 系统版本选择

Docker 目前已经支持多种操作系统的安装运行，比如Ubuntu、CentOS、Redhat、Debian、Fedora，甚至是还支持了Mac和Windows，在linux系统上需要内核版本在3.10或以上，docker版本号之前一直是0.X版本或1.X版本，但是从2017年3月1号开始改为每个季度发布一次稳版，其版本号规则也统一变更为YY.MM，例如17.09表示是2017年9月份发布的，本次演示的操作系统使用Centos 7.5.1804为例。

## - Docker版本选择

Docker之前没有区分版本，但是2017年推出(将docker更名为)新的项目Moby(github地址：

https://github.com/moby/moby)，Moby项目属于Docker项目的全新上游，Docker将是一个隶属于的Moby的子产品，而

且之后的版本之后开始区分为CE版本（社区版本）和EE（企业收费版），CE社区版本和EE企业版本都是每个季度发布一个新版

本，但是EE版本提供后期安全维护1年，而CE版本是4个月，本次演示的Docker版本为18.03，以下为官方原文：

 

```
# https://blog.docker.com/2017/03/docker-enterprise-edition/
"""
Docker CE and EE are released quarterly, and CE also has a monthly “Edge” option. Each Docker 
EE release is supported and maintained for one year and receives security and critical bugfixes during that period. We are also improving Docker CE maintainability by maintaining each quarterly CE release for 4 months. That gets Docker CE users a new 1-month window to update from one version to the next.
"""
```

 

```
# https://blog.docker.com/2017/03/docker-enterprise-edition/
"""
Docker CE and EE are released quarterly, and CE also has a monthly “Edge” option. Each Docker 
EE release is supported and maintained for one year and receives security and critical bugfixes during that period. We are also improving Docker CE maintainability by maintaining each quarterly CE release for 4 months. That gets Docker CE users a new 1-month window to update from one version to the next.
"""
```

## - 安装

系统: Centos 7.5.1804

docker包: docker-18.03.rpm

官方rpm包下载地址:

https://download.docker.com/linux/centos/7/x86_64/stable/Packages/

阿里镜像下载地址:

https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/

 

```
# rpm安装
wget https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm
yum install docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm

# 修改yum源直接yum安装
rm -rf /etc/yum.repos.d/*
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
wget -O /etc/yum.repos.d/docker-ce.repo  https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install docker-ce
```

 

```
# rpm安装
wget https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm
yum install docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm
# 修改yum源直接yum安装
rm -rf /etc/yum.repos.d/*
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
wget -O /etc/yum.repos.d/docker-ce.repo  https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install docker-ce
```

## - 启动并验证docker服务

 

```
systemctl start docker   # 启动docker服务 
systemctl enable docker  # docker开机启动

systemctl status docker  # 查看docker运行状态
"""
...
Active: active (running) since Tue 2018-07-31 14:54:18 CST; 46s ago
...
"""

docker version  # 查看docker版本
"""
Client:
 Version:      18.03.1-ce
 API version:  1.37
 Go version:   go1.9.5
 Git commit:   9ee9f40
 Built:        Thu Apr 26 07:20:16 2018
 OS/Arch:      linux/amd64
 Experimental: false
 Orchestrator: swarm

Server:
 Engine:
  Version:      18.03.1-ce
  API version:  1.37 (minimum version 1.12)
  Go version:   go1.9.5
  Git commit:   9ee9f40
  Built:        Thu Apr 26 07:23:58 2018
  OS/Arch:      linux/amd64
  Experimental: false
"""

ifconfig  # 查看docker0网卡

docker info  # 查看docker信息
```

 

```
systemctl start docker   # 启动docker服务 
systemctl enable docker  # docker开机启动
systemctl status docker  # 查看docker运行状态
"""
...
Active: active (running) since Tue 2018-07-31 14:54:18 CST; 46s ago
...
"""
docker version  # 查看docker版本
"""
Client:
 Version:      18.03.1-ce
 API version:  1.37
 Go version:   go1.9.5
 Git commit:   9ee9f40
 Built:        Thu Apr 26 07:20:16 2018
 OS/Arch:      linux/amd64
 Experimental: false
 Orchestrator: swarm
Server:
 Engine:
  Version:      18.03.1-ce
  API version:  1.37 (minimum version 1.12)
  Go version:   go1.9.5
  Git commit:   9ee9f40
  Built:        Thu Apr 26 07:23:58 2018
  OS/Arch:      linux/amd64
  Experimental: false
"""
ifconfig  # 查看docker0网卡
docker info  # 查看docker信息
```

注:在docker安装并启动之后,默认会生成一个名称为docker0的网卡并且默认IP地址为172.17.0.1

## - docker存储引擎

目前docker的默认存储引擎为overlay2，需要磁盘分区支持d-type文件分层功能，因此需要系统磁盘的额外支持。

 

```
xfs_info /
"""
...
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1  # 这里为1代表支持,0表示不支持
...
"""
```

 

```
xfs_info /
"""
...
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1  # 这里为1代表支持,0表示不支持
...
"""
```

官方文档关于存储引擎的选择文档：

https://docs.docker.com/storage/storagedriver/select-storage-driver/

Docker官方推荐首选存储引擎为overlay2其次为devicemapper，但是devicemapper存在使用空间方面的一些限制，虽然

可以通过后期配置解决，但是官方依然推荐使用overlay2，以下是网上查到的部分资料：

https://www.cnblogs.com/youruncloud/p/5736718.html

如果docker数据目录是一块单独的磁盘分区而且是xfs格式的，那么需要在格式化的时候加上参数-n ftype=1，否则后期在启

动容器的时候会报错不支持d-type。

报错提示:

![img](docker%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/1e5abf53-7eba-4740-822a-610c6588841e.jpg)

## - docker 镜像加速配置

国内下载国外的镜像有时候会很慢，因此可以更改docker配置文件添加一个加速器，可以通过加速器达到加速下载镜像的目的。

方法:

浏览器打开http://cr.console.aliyun.com，注册或登录阿里云账号;

登陆以后,打开产品列表,在"弹性计算"分类下找到"容器镜像服务",然后再左侧选择"镜像加速",将会得到一个专属的加速

地址，而且下面有各系统的使用配置说明.

https://cr.console.aliyun.com/cn-beijing/mirrors

![img](docker%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/7c4e034f-4c41-4c1c-b16c-7d2da61fee6a.jpg)

 

```
cat /etc/docker/daemon.json
"""
{
  "registry-mirrors": ["https://0b8s8193.mirror.aliyuncs.com"]
}
"""

# 重启docke服务：
systemctl daemon-reload
systemctl restart docker
docker info
```

 

```
cat /etc/docker/daemon.json
"""
{
  "registry-mirrors": ["https://0b8s8193.mirror.aliyuncs.com"]
}
"""
# 重启docke服务：
systemctl daemon-reload
systemctl restart docker
docker info
```

注:如果docker info出现如下警告信息

![img](docker%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/ebb12a84-82e4-4bef-9e70-f47582a8950e.jpg)

解决办法:

 

```
sysctl -a | grep bridge-nf-call
vim /etc/sysctl.conf
"""
# 添加如下:
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
"""

sysctl -p
```

 

```
sysctl -a | grep bridge-nf-call
vim /etc/sysctl.conf
"""
# 添加如下:
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
"""
sysctl -p
```

附:

完整的daemon.json的启动参数如下：

 

```
{
    "api-cors-header": "",
    "authorization-plugins": [],
    "bip": "",
    "bridge": "",
    "cgroup-parent": "",
    "cluster-store": "",
    "cluster-store-opts": {},
    "cluster-advertise": "",
    "debug": true,
    "default-gateway": "",
    "default-gateway-v6": "",
    "default-runtime": "runc",
    "default-ulimits": {},
    "disable-legacy-registry": false,
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "exec-root": "",
    "fixed-cidr": "",
    "fixed-cidr-v6": "",
    "graph": "",
    "group": "",
    "hosts": [],
    "icc": false,
    "insecure-registries": [],
    "ip": "0.0.0.0",
    "iptables": false,
    "ipv6": false,
    "ip-forward": false,
    "ip-masq": false,
    "labels": [],
    "live-restore": true,
    "log-driver": "",
    "log-level": "",
    "log-opts": {},
    "max-concurrent-downloads": 3,
    "max-concurrent-uploads": 5,
    "mtu": 0,
    "oom-score-adjust": -500,
    "pidfile": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "runtimes": {
        "runc": {
            "path": "runc"
        },
        "custom": {
            "path": "/usr/local/bin/my-runc-replacement",
            "runtimeArgs": [
                "--debug"
            ]
        }
    },
    "selinux-enabled": false,
    "storage-driver": "",
    "storage-opts": [],
    "swarm-default-advertise-addr": "",
    "tls": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "tlsverify": true,
    "userland-proxy": false,
    "userns-remap": ""
}
```

 

```
{
    "api-cors-header": "",
    "authorization-plugins": [],
    "bip": "",
    "bridge": "",
    "cgroup-parent": "",
    "cluster-store": "",
    "cluster-store-opts": {},
    "cluster-advertise": "",
    "debug": true,
    "default-gateway": "",
    "default-gateway-v6": "",
    "default-runtime": "runc",
    "default-ulimits": {},
    "disable-legacy-registry": false,
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "exec-root": "",
    "fixed-cidr": "",
    "fixed-cidr-v6": "",
    "graph": "",
    "group": "",
    "hosts": [],
    "icc": false,
    "insecure-registries": [],
    "ip": "0.0.0.0",
    "iptables": false,
    "ipv6": false,
    "ip-forward": false,
    "ip-masq": false,
    "labels": [],
    "live-restore": true,
    "log-driver": "",
    "log-level": "",
    "log-opts": {},
    "max-concurrent-downloads": 3,
    "max-concurrent-uploads": 5,
    "mtu": 0,
    "oom-score-adjust": -500,
    "pidfile": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "runtimes": {
        "runc": {
            "path": "runc"
        },
        "custom": {
            "path": "/usr/local/bin/my-runc-replacement",
            "runtimeArgs": [
                "--debug"
            ]
        }
    },
    "selinux-enabled": false,
    "storage-driver": "",
    "storage-opts": [],
    "swarm-default-advertise-addr": "",
    "tls": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "tlsverify": true,
    "userland-proxy": false,
    "userns-remap": ""
}
```

1