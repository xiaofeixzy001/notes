[TOC]

# yum管理工具
yum主要用于自动安装、升级rpm软件包，它能自动查找并解决rpm包之间的依赖关系;

要成功的使用YUM工具安装更新软件或系统，就需要有一个包含各种rpm软件包的repository（软件仓库），这个软件仓库我们习惯称为yum源;

互联网上有大量的yum源，但由于受到网络环境的限制，导致软件安装耗时过长甚至失败,特别是当有大量服务器大量软件包需要安装时，缓慢的进度条令人难以忍受,因此我们在优化系统时，一般都会更换国内的源;

目前常见得国内yum源:阿里,搜狐,网易都可以.

还可以搭建本地yum源,相比较而言,本地YUM源服务器最大优点是局域网的快速网络连接和稳定性,有了局域网中的YUM源服务器，即便在Internet连接中断的情况下，也不会影响其他YUM客户端的软件安装和升级.



# yum与rpm关系

1,rpm本身是一个包管理器，所以它具备一个管理器的打包、安装查询、升级、卸载、校验、数据库管理这些基本功能;



2,使用rpm命令安装最让人头疼的问题就是软件包之间的依赖关系，使用yum工具会自动处理我们安装过程中包之间依赖关系,它只是一个前端的工具，并不能替代rpm包管理器;

3,yum本身并不会知道包之间的依赖关系，而包之间的依赖关系等元数据，会存放在repodata这个文件中;

4,以光盘为例，当我们使用挂在光盘到本地时,如 mount /dev/cdrom /media/cdrom ,就会在/media/cdrom/下看到光盘内的目录和文件,repodata文件中的repomd.xml就存放着各个rpm包之间的关联信息，而TRANS.TBL存放着rpm包的分组信息;

5,客户端在使用yum命令时，会先下载yum的配置文件，从中找到yum仓的路径，再下载repodate里的元数据，而后安装rpm包;yum客户端安装过程：1,获取仓库元数据目录repodata/2,分析要安装的程序包3,获取程序包.


# yum配置方式

1,直接配置/etc/yum.conf文件;

2,在/etc/yum.repos.d/目录下创建以.repo结尾的文件

yum的配置文件分为主配置段[/etc/yum.conf]和仓库配置段[/etc/yum.repos.d/*.repo];

这么设计是因为yum仓库可有多个,如果都写在/etc/yum.conf文件中，不便于查看，所以有了仓库配置段;

yum工具会将所有在/etc/yum.repos.d/目录内以.repo结尾的文件来作为配置文件;

比如我们可以在/etc/yum.repos.d/下为每一个yum仓库定义一个.repo文件，或者在一个.repo文件中分段表示多个yum仓,



# yum.conf文件

yum配置文件分为两部分:main和repository

main部分定义了全局配置选项,整个yum 配置文件应该只有一个main,常位于/etc/yum.conf中;

repository部分定义了每个源/服务器的具体配置,可以有一到多个,常位于/etc/yum.repo.d/目录下的各文件中;

## yum.conf详解

```
[main]
#yum缓存的目录.yum在此存储下载的rpm包和数据库,默认设置为/var/cache/yum
cachedir=/var/cache/yum

#安装完成后是否保留软件包,0为不保留（默认为0）,1为保留
keepcache=0

#Debug信息输出等级,范围为0-10，缺省为2
debuglevel=2

#yum日志文件位置,用户可以到/var/log/yum.log文件去查询过去所做的更新。
logfile=/var/log/yum.log

#包的策略,一共有两个选项:newest和last.这个作用是如果你设置了多个repository，而同一软件在不同的repository中同时存在，yum 应该安装哪一个，如果是newest，则yum 会安装最新的那个版本;如果是last，则yum 会将服务器id 以字母表排序，并选择最后的那个服务器上的软件安装,一般都是选newest
pkgpolicy=newest

#指定一个软件包，yum 会根据这个包判断你的发行版本，默认是redhat-release，也可以是安装的任何针对自己发行版的rpm包
distroverpkg=redhat-release

#有1和0两个选项，表示yum 是否容忍命令行发生与软件包有关的错误，比如你要安装1,2,3三个包，而其中3此前已经安装了，如果你设为1,则yum 不会出现错误信息。默认是0
tolerant=1

#有1和0两个选项，设置为1，则yum 只会安装和系统架构匹配的软件包，例如，yum 不会将i686的软件包安装在适合i386的系统中,默认为1
exactarch=1

#网络连接发生错误后的重试次数，如果设为0，则会无限重试。默认值为6.
retries=6

#这是一个update 的参数，具体请参阅yum(8)，简单的说就是相当于upgrade，允许更新陈旧的RPM包
obsoletes=1

#是否启用插件，默认1为允许，0表示不允许。我们一般会用yum-fastestmirror这个插件
plugins=1

除了上述默认之外,还有一些可以添加的选项:
exclude=selinux*
#排除某些软件在升级名单之外,可以用通配符，列表中各个项目要用空格隔开，这个对于安装了诸如美化包，中文补丁的朋友特别有用

gpgcheck=1
#有1和0两个选择,分别代表是否进行gpg(GNU Private Guard)校验,以确定rpm 包的来源是有效和安全的。这个选项如果设置在[main]部分，则对每个repository 都有效,默认值为0.
```





## .repo文件格式

一个repo文件定义了一个或多个软件仓库，指定yum从哪里下载或更新的软件包;

repodata目录所在的父目录就是一个可用仓库

base.repo文件格式

```
#包来源的名称，可自定义，不能重名
[base]

#yum仓库的名称，可自定义,一般见名知意
name=base $releasever - $basearch

#表示从列出的baseurl中顺序选择还是随机选择
failovermethod=priority|roundrobin

#排除软件程序不允许安装更新，空格分隔，支持通配符
exclude=software1* software2* software3* ...

#指定yum源,file指向本地文件,http指向互联网网站,可指多个,顺序或随机使用仓库,注意，不能顶格写
baseurl=
        file:///mnt/cdrom
        http://mirrors.163.com/centos/$releasever/os/$basearch/    
        http://mirrors.sohu.com/centos/$releasever/os/$basearch/

#是否启动此仓库，1为启动，0为禁用
enabled=1

#是否对下载的rpm进行gpg的校验，以确定rpm包的来源是有效和安全的，1为启动，0为禁用
gpgcheck=1

#定义用于校验的gpg密钥
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

#定义开销,默认为1000,数字越小,越优先使用
cost=1000
```



配置文件中常用的四个宏

$releasever:程序的版本,如CentOS6.5替换为6

$arch：系统架构,如i386,x86 64等,注意，=号左右不能出现空格

$basearch：系统基本架构，如i686，i586等的基本架构为i386

$YUM0-9:在系统中定义的环境变量，可以在yum中使用



# 基于本地光盘配置yum源示例:

```
mount /dev/cdrom/ /media/cdrom/
vim /etc/yum.repo.d/cdrom.repo
[cdrom]
name=local cdrom
baseurl=file:///media/cdrom
enabled=1
gpgcheck=0
```



# 基于公网镜像网站配置yum源示例:

```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
yum clean all
yum makecache
```





# yum命令

选项:

  -c：指定配置文件
  -v：详细模式
  -q：静默模式
  -C：完全从缓存中运行，而不去下载或者更新任何头文件

yum -y：自动回答yes



参数:

```shell
#自动搜索最快的镜像插件
yum install yum-fastestmirror

#安装yum图形窗口插件
yum install yumex

#查看可批量安装的列表
yum grouplist

#根据关键字package_name查找安装包
yum search package_name

#列出所有可用repo  
yum repolist {enable|disabled|all}

#列出rpm包,支持通配符,如php*
yum list {all|installed|available}

#查看包的描述信息
yum info package_name

#列出所有的包组的信息
yum groupinfo group_name

#清理yum缓存
yum clean ｛all|package|metadata|expire-cache|rpmdb|plugins｝

# 清除缓存目录下的headers
yum clean headers

#清楚缓存目录下旧的headers
yum clean oldheaders

#安装程序包
yum install package_name

#重装
yum reinstall 

#升级,不跟包名,升级所有可升级的包
yum update 
yum update package_name

#卸载
yum remove|erase pack_name

#查询某个文件是由哪个安装包生产的
yum whatprovides|provides|path/to/somefile

#安装包组
yum groupinstall "group_name"

#卸载包组
yum groupremove "group_name"

#不安装仅下载包到本地,需要安装插件yum-plugin-downloadonly
yum install --downloadonly --downloaddir=/local_rpm/*.rpm
```





跟开发相关的包组：
Development Tools 开发工具
Server Platform Development 服务开发平台
Desktop Platform Development  图形开发平台



错误1:

安装包时提示failure: Packages/package_name from base: [Errno 256] No more mirrors to try.

解决办法:

\#yum clean all

\#yum makecache

\#yum -y update



# yum源和yum仓库

基于本地光盘配置yum源示例:

 

```
mount /dev/cdrom/ /media/cdrom/
vim /etc/yum.repo.d/cdrom.repo
[cdrom]
name=local cdrom
baseurl=file:///media/cdrom
enabled=1
gpgcheck=0
```



基于公网镜像网站配置yum源示例:

 

```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
yum clean all
yum makecache
```



使用第三方YUM源(EPEL,RPMForge,RPMFusion)

CentOS自带的yum源中rpm包数量有限,很多时候找不到我们需的软件包,所以我们需要安装包含丰富的第三方YUM源来满足我们的需求.

 

```
# 安装yum-priorities插件,yum-priorities插件的作用主要是设置调用源时的优先级的,一般将官方的优先级设置为最高,在每个库最后加上priority=[#]字段来设置每个镜像的优先级,#=1-99,1为最高,99为最低
yum install -y yum-priorities

#添加字段:priority=#(N为1到99的正整数,数值越小越优先);一般在[base],[addons],[updates],[extras]字段下设置高优先级;[CentOSplus],[contrib]字段下设置低优先级,其他第三的软件源为最低优先级;
vim /etc/yum.repo.d/CentOS-Base.repo
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
priority=1

#安装epel
#32位：
rpm -ivh http://mirrors.ustc.edu.cn/fedora/epel/6/i386/epel-release-6-8.noarch.rpm
#64位：
rpm -ivh http://mirrors.ustc.edu.cn/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
vi /etc/yum.repos.d/epel.repo  #在每个库后面添加优先级2
[epel]
...
priority=2

#安装rpmforge
#32位：
rpm -ivh http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.i686.rpm
#64位：
rpm -ivh http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm
# 设置优先级
vi /etc/yum.repos.d/rpmforge.repo
[rpmforge]
...
priority=3

#安装rpmfusion
#32位：
rpm -ivh http://download1.rpmfusion.org/free/el/updates/6/i386/rpmfusion-free-release-6-1.noarch.rpm
#64位：
rpm -ivh http://download1.rpmfusion.org/free/el/updates/6/x86_64/rpmfusion-free-release-6-1.noarch.rpm
# 设置优先级
vi /etc/yum.repos.d/rpmfusion-free-updates.repo
[rpmfusion-free-updates]
...
prioryty=4

# 设置优先级
vi /etc/yum.repos.d/rpmfusion-free-updates-testing.repo
[rpmfusion-free-updates-testing]
...
priority=4

# 加载所有yum源
yum check-update
# 列出所有第三方yum源
yum list
```



# 搭建本地yum服务器
使用场景:很多企业都会自建yum仓来便于企业内部的rpm包安装,这样在安装软件时就可以利用局域网内的高速带宽进行下载和安装rpm包，缩短了通过公网下载的长时间等待;

方法示例:

 

```
#创建yum仓库目录
mkdir -p /application/yum/centos6.6/x86_64/    
cd /application/yum/centos6.6/x86_64/
rz
#上传rpm包到此目录，此目录下面还可以包括文件夹

#安装createrepo软件
yum -y install createrepo 

#初始化repodata索引文件
createrepo -pdo /application/yum/centos6.6/x86_64/ /application/yum/centos6.6/x86_64/

#提供yum服务,可以用Apache或nginx提供web服务,但用Python的http模块更简单,适用于内网环境
python -m SimpleHTTPServer 80 &
python -m SimpleHTTPServer 80 &>/dev/null &  

#添加新的rpm包,这里只下载软件不安装
yumdownloader zlib-devel
cp zlib-devel-1.2.3-29.el6.x86_64.rpm /application/yum/centos6.6/x86_64/
createrepo --update /application/yum/centos6.6/x86_64/  #每加入一个rpm包就要更新一下。

#客户端配置
vi /etc/yum.repos.d/etiantian.repo 
[etiantian]
name=Server
baseurl=http://yum.etiantian.org
enabled=1
gpgcheck=0
#在CentOS-Base.repo的几个可用源加上enabled=0使其不可用，唯一能用的是etiantian.repo，分发修改过的CentOS-Base.repo文件。要使用外部源，则使用yum --enablerepo=base -y install
```



注:yumdownloader,由yum-utils包提供