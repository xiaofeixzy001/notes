[TOC]







1）安装：

​    yum -y install ruby rubygems ruby-devel  安装ruby 和gem

​    gem install fpm               安装fpm工具

 

2）准备编译安装好的源码包

​    /usr/local/libiconv

 

3）打包：

​    fpm -f -s dir -t rpm -n beyond-libiconv --epoch=0 -v '1.14' -C /usr/local --iteration 1.el6 ./libiconv-1.14

 

参数解释

 

-f 强制输出，如果文件已存在，将会覆盖源文件

-s 指定源文件为目录 dir

-t 指定制作的包类型（rpm，deb solaris etc）

-n 指定制作的包名

-- epoch 指定时间戳

-v 指定软件版本

-C 指定软件安装的目录

--iteration 指定软件的适用平台

./libiconv-1.14 本次打包的文件

 

 

附加参数：

-e 可以在打包之前编译 spec文件

-d 指定依赖的软件包 用法 –d ‘Package’ 或 –d ‘Package > version’

--description 软件包描述

-p 生成的package文件输出位置

--url  说明软件包的url

--post-install ：软件包安装完成之后所要运行的脚本；和”--after-install” 意思一样

--pre-install ：软件包安装完成之前所要运行的脚本；和”--before-install” 意思一样

--post-uninstall ：软件包卸载完成之后所要运行的脚本；和”--after-remove”意思一样

--pre-uninstall：软件包卸载完成之前所要运行的脚本；和”--before-remove”意思一样

 

fpm打包php案例：

fpm  -f -s dir -t rpm -n beyond-php --epoch=0 -v '5.2.14' -C /usr/local/ -p ./ --iteration 1.el6 -d 'beyond-libiconv'  -d 'beyond-libmcrypt' -d 'beyond-mcrypt' -d 'beyond-mhash' -d 'libxml2'  -d 'libxml2-devel' -d 'zlib' -d 'zlib-devel' -d 'libpng' -d  'libpng-devel' -d 'freetype' -d 'freetype-devel' -d 'autoconf' -d 'gd'  -d 'gd-devel' -d 'libjpeg' -d 'libjpeg-devel' -d 'curl' -d 'curl-devel' -d 'mysql-devel' -d 'openssl' -d 'openssl-devel' -d 'openldap-devel' -d 'libtool-ltdl' -d 'libtool-ltdl-devel' --url  http://sa.beyond.com/source/php-5.2.14.tar.gz --license GPL  --post-install ./preinstall.sh  /usr/local/php







# 安装软件方式

1、编译安装软件，优点是可以定制化安装目录、按需开启功能等，缺点是需要查找并实验出适合的编译参数，诸如MySQL之类的软件编译耗时过长。

2、yum安装软件，优点是全自动化安装，不需要为依赖问题发愁了，缺点是自主性太差，软件的功能、存放位置都已经固定好了，不易变更。

3、编译源码，根据自己的需求做成定制RPM包–>搭建内网yum仓库–yum安装。结合前两者的优点，暂未发现什么缺点。可能的缺点就是RPM包的通用性差，只能适用于本公司的环境。另外一般人不会定制RPM包。这是中大型互联网企业运维自动化的必要技能。

# FPM打包工具

FPM的作者是jordansissel

FPM的github：https://github.com/jordansissel/fpm

FPM功能简单说就是将一种类型的包转换成另一种类型。

## 支持的源类型包

dir：将目录打包成所需要的类型，可以用于源码编译安装的软件包
rpm：对rpm进行转换
gem：对rubygem包进行转换
python：将python模块打包成相应的类型

## 支持的目标类型包

rpm：转换为rpm包
deb：转换为deb包
solaris：转换为solaris包
puppet：转换为puppet模块

## FPM安装

fpm是ruby写的，因此系统环境需要ruby，且ruby版本号大于1.8.5。

```shell
# 安装ruby模块
# yum -y install ruby rubygems ruby-devel

# 查看当前使用的rubygems仓库
# gem sources list

# 添加淘宝的Rubygems仓库，外国的源慢，移除原生的Ruby仓库
# gem sources --add https://ruby.taobao.org/ --remove http://rubygems.org/

# 安装fpm，gem从rubygem仓库安装软件类似yum从yum仓库安装软件。首先安装低版本的json，高版本的json需要ruby2.0以上，然后安装低版本的fpm，够用。
# gem install json -v 1.8.3
# gem install fpm -v 1.3.3
```

上面的2步安装仅适合CentOS6系统，CentOS7系统一步搞定，即gem install fpm

## FPM参数

详细使用见：fpm –help

常用参数

-s          指定源类型
-t          指定目标类型，即想要制作为什么包
-n          指定包的名字
-v          指定包的版本号
-C          指定打包的相对路径  Change directory to here before searching forfiles
-d          指定依赖于哪些包
-f           第二次打包时目录下如果有同名安装包存在，则覆盖它
-p          输出的安装包的目录，不想放在当前目录下就需要指定
--post-install      软件包安装完成之后所要运行的脚本；同--after-install
--pre-install       软件包安装完成之前所要运行的脚本；同--before-install
--post-uninstall    软件包卸载完成之后所要运行的脚本；同--after-remove
--pre-uninstall     软件包卸载完成之前所要运行的脚本；同--before-remove

# 应用示例

实战定制nginx的RPM包

## 安装nginx

```shell
# yum -y install pcre-devel openssl-devel
# useradd nginx -M -s /sbin/nologin
# tar xf nginx-1.6.2.tar.gz
# cd nginx-1.6.2
# ./configure --prefix=/application/nginx-1.6.2 --user=nginx --group=nginx --with-http_ssl_module --with-http_stub_status_module
# make && make install
# ln -s /application/nginx-1.6.2/ /application/nginx
```



## 编写脚本

```SHELL
# cd /server/scripts/
# vim nginx_rpm.sh  # 这是安装完rpm包要执行的脚本
"""
#!/bin/bash
useradd nginx -M -s /sbin/nologin
ln -s /application/nginx-1.6.2/ /application/nginx
"""
```



## 打包

```shell
# fpm -s dir -t rpm -n nginx -v 1.6.2 -d 'pcre-devel,openssl-devel' --post-install /server/scripts/nginx_rpm.sh -f /application/nginx-1.6.2/  
"""
no value for epoch is set, defaulting to nil {:level=>:warn}
no value for epoch is set, defaulting to nil {:level=>:warn}
Created package {:path=>"nginx-1.6.2-1.x86_64.rpm"}
"""

[root@oldboy ~]# ll -h nginx-1.6.2-1.x86_64.rpm 
"""
-rw-r--r-- 1 root root 6.7M Nov  1 10:02 nginx-1.6.2-1.x86_64.rpm
"""
```



## 安装rpm包

安装rpm包的三种方法：

### rpm命令安装

```shell
[root@LB-nginx-01 ~]# rpm -ivh nginx-1.6.2-1.x86_64.rpm
"""
error: Failed dependencies:
pcre-devel is needed by nginx-1.6.2-1.x86_64
openssl-devel is needed by nginx-1.6.2-1.x86_64
"""

```

会报如上依赖错误，需要先yum安装依赖才能安装rpm包。

### yum命令安装

```shell
yum -y localinstall nginx-1.6.2-1.x86_64.rpm
```

这个命令会自动先安装rpm包的依赖，然后再安装rpm包。



## 注意事项

### 相对路径问题

```shell
# 相对路径
[root@oldboy nginx]# fpm -s dir -t rpm -n nginx -v 1.6.2 .
no value for epoch is set, defaulting to nil {:level=>:warn}
no value for epoch is set, defaulting to nil {:level=>:warn}
Created package {:path=>"nginx-1.6.2-1.x86_64.rpm"}

[root@oldboy nginx]# rpm -qpl nginx-1.6.2-1.x86_64.rpm    
/client_body_temp
/conf/extra/dynamic_pools
/conf/extra/static_pools
…………

# 绝对路径
[root@oldboy ~]# fpm -s dir -t rpm -n nginx -v 1.6.2 /application/nginx-1.6.2/
no value for epoch is set, defaulting to nil {:level=>:warn}
no value for epoch is set, defaulting to nil {:level=>:warn}
Created package {:path=>"nginx-1.6.2-1.x86_64.rpm"}

[root@oldboy ~]# rpm -qpl nginx-1.6.2-1.x86_64.rpm 
/application/nginx-1.6.2/client_body_temp
/application/nginx-1.6.2/conf/extra/dynamic_pools
/application/nginx-1.6.2/conf/extra/static_pools
/application/nginx-1.6.2/conf/fastcgi.conf
/application/nginx-1.6.2/conf/fastcgi.conf.default
…………
```

使用rpm -qpl 命令可以查看rpm包的内容。
注：fpm类似tar打包一样，只是fpm打的包能够被yum命令识别而已。

### 软链接问题

```shell
[root@oldboy ~]# fpm -s dir -t rpm -n nginx -v 1.6.2 /application/nginx
no value for epoch is set, defaulting to nil {:level=>:warn}
File already exists, refusing to continue: nginx-1.6.2-1.x86_64.rpm {:level=>:fatal}

# 报错是因为当前目录存在同名的rpm包，可以使用-f参数强制覆盖。

[root@oldboy ~]# fpm -s dir -t rpm -n nginx -v 1.6.2 -f /application/nginx
no value for epoch is set, defaulting to nil {:level=>:warn}
Force flag given. Overwriting package at nginx-1.6.2-1.x86_64.rpm {:level=>:warn}
no value for epoch is set, defaulting to nil {:level=>:warn}
Created package {:path=>"nginx-1.6.2-1.x86_64.rpm"}

# 打包看似成功，但查看包的内容，只是这一个软链接文件。
[root@oldboy ~]# rpm -qpl nginx-1.6.2-1.x86_64.rpm 
/application/nginx
# 原因：目录结尾的/问题，类似rm删除软链接目录
```



## 定制LNMP的RPM包思路

编译安装好nginx，mysql，php，此处有个问题，就是php的大部分依赖环境是通过yum安装的，但有一个libiconv-1.14.tar.gz包需要编译安装,安装时已经指定了安装目录，只需一同打包即可。

还有一个问题，就是mysql这个目录比较大，用fpm打包耗时长。平时我们有可能需要对nginx或php做优化，这样又得重新打包。因此我们可以将mysql分离出来，分别打包。只需在制作nginx+php的rpm包时添加mysql的依赖即可。

```shell
# 参考命令
[root@web2 ~]# fpm -s dir -t rpm -n web2 -v 1.1 \
--description 'lnmp.cms,bbs.blog' \
-d ‘libxslt-devel,nfs-utils,rpcbind,mysql,libmcrypt-devel,mhash,mhash-devel,mcrypt' \
--post-install /server/scripts/lnmp-init.sh  \
/application /usr/local/libiconv/ /app/logs/ /data0/  /server/
```

