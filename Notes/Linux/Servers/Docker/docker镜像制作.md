[TOC]

# 前言

Docker 镜像有没有内核？

从镜像大小上面来说,一个比较小的镜像只有十几MB,而内核文件需要一百多兆,因此镜像里面是没有内核的,镜像在被启动为容器后将直接使

用宿主机的内核,而镜像本身则只提供相应的rootfs,即系统正常运行所必须的用户空间的文件系统,比如/dev/,/proc,/bin,/etc等目

录,所以容器当中基本是没有/boot目录的,而/boot当中保存的就是与内核相关的文件和目录.

为什么没有内核?

由于容器启动和运行过程中是直接使用了宿主机的内核,所以没有直接调用过物理硬件,所以也不会涉及到硬件驱动,因此也用不上内核和驱

动,另外有内核的那是虚拟机.

# 镜像制作

个人理解类似虚拟机的快照

## 手动制作yum版nginx镜像

Docker制作类似于虚拟机的镜像制作，即按照公司的实际业务务求将需要安装的软件、相关配置等基础环境配置完成，然后将其做成镜像，

最后再批量从镜像批量生产实例，这样可以极大的简化相同环境的部署工作，Docker的镜像制作分为手动制作和自动制作(基于DockerFile)，其中手动制作镜像步骤具体如下：

1,下载镜像并初始化系统

基于某个基础镜像之上重新制作，因此需要先有一个基础镜像，本次使用官方提供的centos镜像为基础：

 

```
# 先制作一个基础镜像centos-base，安装一些常用软件，做一些优化。
docker pull centos:latest
docker images
docker run -it centos /bin/bash
[root@9e0c9d668f70 ~]# yum install -y wget
[root@9e0c9d668f70 ~]# cd /etc/yum.repos.d/
[root@9e0c9d668f70 ~]# rm -rf ./*
[root@9e0c9d668f70 ~]# wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
[root@9e0c9d668f70 ~]# wget -O epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
[root@9e0c9d668f70 ~]# yum install -y  tree vim ntpdate man iproute net-tools iotop
[root@9e0c9d668f70 ~]# exit

# 在本地(宿主机)操作,基于容器ID提交为镜像,名字为mynginx:v1
docker ps -a
docker commit -m "mynginx" 9e0c9d668f70 mynginx:v1
```

 

```
# 先制作一个基础镜像centos-base，安装一些常用软件，做一些优化。
docker pull centos:latest
docker images
docker run -it centos /bin/bash
[root@9e0c9d668f70 ~]# yum install -y wget
[root@9e0c9d668f70 ~]# cd /etc/yum.repos.d/
[root@9e0c9d668f70 ~]# rm -rf ./*
[root@9e0c9d668f70 ~]# wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
[root@9e0c9d668f70 ~]# wget -O epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
[root@9e0c9d668f70 ~]# yum install -y  tree vim ntpdate man iproute net-tools iotop
[root@9e0c9d668f70 ~]# exit
# 在本地(宿主机)操作,基于容器ID提交为镜像,名字为mynginx:v1
docker ps -a
docker commit -m "mynginx" 9e0c9d668f70 mynginx:v1
```

2,yum安装并配置Nginx

 

```
[root@9e0c9d668f70 ~]# yum install -y nginx pcre pcre-devel zlib zlib-devel openssl openssl-devel 
[root@9e0c9d668f70 ~]# vim /etc/nginx/nginx.conf
"""
user nginx nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
daemon off; #关闭后台运行
"""

[root@9e0c9d668f70 ~]# cp /usr/share/nginx/html/index.html{,.bak}
[root@9e0c9d668f70 ~]# echo "from from my nginx image!" > /usr/share/nginx/html/index.html
"""
docker yum nginx
"""
```

 

```
[root@9e0c9d668f70 ~]# yum install -y nginx pcre pcre-devel zlib zlib-devel openssl openssl-devel 
[root@9e0c9d668f70 ~]# vim /etc/nginx/nginx.conf
"""
user nginx nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
daemon off; #关闭后台运行
"""
[root@9e0c9d668f70 ~]# cp /usr/share/nginx/html/index.html{,.bak}
[root@9e0c9d668f70 ~]# echo "from from my nginx image!" > /usr/share/nginx/html/index.html
"""
docker yum nginx
"""
```

3,镜像提交与启动

提交为镜像(保存)

 

```
# 在本地(宿主机)操作,基于容器ID提交为镜像,名字为mynginx:v1
docker images
docker commit -m "mynginx" 9e0c9d668f70 mynginx:v1
```

 

```
# 在本地(宿主机)操作,基于容器ID提交为镜像,名字为mynginx:v1
docker images
docker commit -m "mynginx" 9e0c9d668f70 mynginx:v1
```

说明:

带tag的镜像提交,提交的时候标记tag号.

标记tag号，生产当中比较长用，后期可以根据tag标记启动不同版本启动image启动,

4,以自定义的镜像启动一个容器

 

```
docker run -d -p 80:80 --name mynginx:v1 mynginx:v1 /usr/sbin/nginx
ss -tnlp
curl http://172.16.2.1/
```

 

```
docker run -d -p 80:80 --name mynginx:v1 mynginx:v1 /usr/sbin/nginx
ss -tnlp
curl http://172.16.2.1/
```

## 手动制作编译版本的nginx镜像 

过程为在centos 基础镜像之上手动编译安装nginx，然后再提交为镜像

 

```
[root@docker-server1 opt]# docker run -it centos /bin/bash
[root@fc9179085ce9 /]# yum install wget lrzsz -y
[root@fc9179085ce9 /]# rm -rf /etc/yum.repo.d/*
[root@fc9179085ce9 /]# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
[root@fc9179085ce9 /]# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

[root@fc9179085ce9 /]# yum install -y vim wget tree  lrzsz gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel openssl openssl-devel iproute net-tools iotop

[root@fc9179085ce9 /]# cd /usr/local/src
[root@fc9179085ce9 src]# wet http://nginx.org/download/nginx-1.10.3.tar.gz
[root@fc9179085ce9 src]# tar xvf nginx-1.10.3.tar.gz
[root@fc9179085ce9 src]# cd nginx-1.10.3
[root@fc9179085ce9 nginx-1.10.3]# ./configure  --prefix=/apps/nginx --with-http_sub_module
[root@fc9179085ce9 nginx-1.10.3]# make && make install
[root@fc9179085ce9 nginx-1.10.3]# cd /apps/nginx
[root@fc9179085ce9 nginx]# vim conf/nginx.conf
"""
user nginx nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
daemon off; #关闭后台运行
"""
[root@fc9179085ce9 nginx]# ln -sv /apps/nginx/sbin/nginx  /usr/sbin/nginx
[root@fc9179085ce9 nginx]# useradd  nginx -s /sbin/nologin
[root@fc9179085ce9 nginx]# chown  nginx.nginx /usr/local/nginx/ -R
[root@fc9179085ce9 nginx]# echo "from my nginx test page!" > /apps/nginx/html/index.html
[root@fc9179085ce9 nginx]# exit

[root@docker-server1 ~]# docker images
[root@docker-server1 ~]# docker commit -m "my-nginx" 86a48908bb97  my-nginx:v1
[root@docker-server1 ~]# docker run -d -p 80:80  --name my-nginx my-nginx:v1  /usr/sbin/nginx
[root@docker-server1 ~]# tail -f /apps/nginx/logs/access.log
[root@docker-server1 opt]# docker run -it centos /bin/bash
[root@fc9179085ce9 /]# yum install wget lrzsz -y
[root@fc9179085ce9 /]# rm -rf /etc/yum.repo.d/*
[root@fc9179085ce9 /]# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
[root@fc9179085ce9 /]# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
[root@fc9179085ce9 /]# yum install -y vim wget tree  lrzsz gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel openssl openssl-devel iproute net-tools iotop
[root@fc9179085ce9 /]# cd /usr/local/src
[root@fc9179085ce9 src]# wet http://nginx.org/download/nginx-1.10.3.tar.gz
[root@fc9179085ce9 src]# tar xvf nginx-1.10.3.tar.gz
[root@fc9179085ce9 src]# cd nginx-1.10.3
[root@fc9179085ce9 nginx-1.10.3]# ./configure  --prefix=/apps/nginx --with-http_sub_module
[root@fc9179085ce9 nginx-1.10.3]# make && make install
[root@fc9179085ce9 nginx-1.10.3]# cd /apps/nginx
[root@fc9179085ce9 nginx]# vim conf/nginx.conf
"""
user nginx nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
daemon off; #关闭后台运行
"""
[root@fc9179085ce9 nginx]# ln -sv /apps/nginx/sbin/nginx  /usr/sbin/nginx
[root@fc9179085ce9 nginx]# useradd  nginx -s /sbin/nologin
[root@fc9179085ce9 nginx]# chown  nginx.nginx /usr/local/nginx/ -R
[root@fc9179085ce9 nginx]# echo "from my nginx test page!" > /apps/nginx/html/index.html
[root@fc9179085ce9 nginx]# exit
[root@docker-server1 ~]# docker images
[root@docker-server1 ~]# docker commit -m "my-nginx" 86a48908bb97  my-nginx:v1
[root@docker-server1 ~]# docker run -d -p 80:80  --name my-nginx my-nginx:v1  /usr/sbin/nginx
[root@docker-server1 ~]# tail -f /apps/nginx/logs/access.log
```

说明：

-name是指定容器的名称，-d是后台运行，-p是端口映射，my-nginx:v1是xx仓库下的xx镜像的xx版本，可以不加版本，不加版本默认是使用latest，最后面的nginx是运行的命令，即镜像里面要运行一个nginx命令，所以才有了前面将/apps/nginx/sbin/nginx软连接到/usr/sbin/nginx，目的就是为了让系统可以执行此命令。

## DockerFile制作镜像

DockerFile可以说是一种可以被Docker程序解释的脚本，DockerFile是由一条条的命令组成的，每条命令对应linux下面的一条命令，Docker程序将这些DockerFile指令再翻译成真正的linux命令，其有自己的书写方式和支持的命令，Docker程序读取DockerFile并根据指令生成Docker镜像，相比手动制作镜像的方式，DockerFile更能直观的展示镜像是怎么产生的，有了DockerFile，当后期有额外的需求时，只要在之前的DockerFile添加或者修改响应的命令即可重新生成新的Docke镜像，避免了重复手动制作镜像的麻烦。

应用场景：假设我们需要一个最小化的系统centos，然后安装常用软件工具，配置好常用环境，此为第二版的系统centos-base，然后在此基础上继续操作，如下图所示：

![img](docker%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C.assets/ab549d41-cc3a-4e09-8462-0b9f1c690523.png)

开始之前先创建好目录结构，按照业务类型或系统类型等方式划分，方便后期镜像比较多的时候进行分类。

dockerfile/

├── system

│   ├── centos

│   ├── redhat

│   └── ubuntu

└── web

  ├── apache

  ├── jdk

  ├── nginx

  └── tomcat

 

```
docker pull centos
docker images

# 
mkdir -pv /opt/dockerfile/{web/{nginx,tomcat,jdk,apache},system/{centos,ubuntu,redhat}}
tree -L 2 dockerfile/
```

 

```
docker pull centos
docker images
# 
mkdir -pv /opt/dockerfile/{web/{nginx,tomcat,jdk,apache},system/{centos,ubuntu,redhat}}
tree -L 2 dockerfile/
```

### Dockerfile制作Nginx镜像 

cat Dockerfile

 

```
# Nginx Image

FROM centos-base:v1

MAINTAINER xiaofei 1023668666@qq.com

RUN yum groupinstall "Development Tools" "Server Platform Development" -y

RUN yum install gcc openssl-devel pcre-devel zlib-devel gcc-c++ -y

RUN groupadd -r nginx

RUN useradd -r -g nginx -s /bin/false -M nginx

ADD nginx-1.12.2.tar.gz /usr/local/src/

RUN mkdir -p /usr/local/nginx/{conf,log,run,lock,client,proxy,fcgi,uwsgi,scgi}

WORKDIR /usr/local/src/nginx-1.12.2

RUN ./configure --prefix=/usr/local/nginx --conf-path=/usr/local/nginx/conf/nginx.conf --user=nginx --group=nginx --error-log-path=/usr/local/nginx/log/error.log --http-log-path=/usr/local/nginx/log/access.log --pid-path=/usr/local/nginx/run/nginx.pid  --lock-path=/usr/local/nginx/lock/nginx.lock --with-http_ssl_module --with-http_flv_module --with-http_stub_status_module --with-http_gzip_static_module --http-client-body-temp-path=/usr/local/nginx/client/ --http-proxy-temp-path=/usr/local/nginx/proxy/ --http-fastcgi-temp-path=/usr/local/nginx/fcgi/ --http-uwsgi-temp-path=/usr/local/nginx/uwsgi --http-scgi-temp-path=/usr/local/nginx/scgi --with-pcre  && make && make install

RUN chown -R nginx.nginx /usr/local/nginx

EXPOSE 80

ADD run_nginx.sh /usr/local/nginx/sbin/run_nginx.sh

CMD ["/usr/local/nginx/sbin/run_nginx.sh"]
```

 

```
# Nginx Image
FROM centos-base:v1
MAINTAINER xiaofei 1023668666@qq.com
RUN yum groupinstall "Development Tools" "Server Platform Development" -y
RUN yum install gcc openssl-devel pcre-devel zlib-devel gcc-c++ -y
RUN groupadd -r nginx
RUN useradd -r -g nginx -s /bin/false -M nginx
ADD nginx-1.12.2.tar.gz /usr/local/src/
RUN mkdir -p /usr/local/nginx/{conf,log,run,lock,client,proxy,fcgi,uwsgi,scgi}
WORKDIR /usr/local/src/nginx-1.12.2
RUN ./configure --prefix=/usr/local/nginx --conf-path=/usr/local/nginx/conf/nginx.conf --user=nginx --group=nginx --error-log-path=/usr/local/nginx/log/error.log --http-log-path=/usr/local/nginx/log/access.log --pid-path=/usr/local/nginx/run/nginx.pid  --lock-path=/usr/local/nginx/lock/nginx.lock --with-http_ssl_module --with-http_flv_module --with-http_stub_status_module --with-http_gzip_static_module --http-client-body-temp-path=/usr/local/nginx/client/ --http-proxy-temp-path=/usr/local/nginx/proxy/ --http-fastcgi-temp-path=/usr/local/nginx/fcgi/ --http-uwsgi-temp-path=/usr/local/nginx/uwsgi --http-scgi-temp-path=/usr/local/nginx/scgi --with-pcre  && make && make install
RUN chown -R nginx.nginx /usr/local/nginx
EXPOSE 80
ADD run_nginx.sh /usr/local/nginx/sbin/run_nginx.sh
CMD ["/usr/local/nginx/sbin/run_nginx.sh"]
```

cat run_nginx.sh

 

```
#!/bin/bash
#
/usr/local/nginx/sbin/nginx

tail -f /usr/local/nginx/log/access.log
```

 

```
#!/bin/bash
#
/usr/local/nginx/sbin/nginx
tail -f /usr/local/nginx/log/access.log
```

cat build-command.sh 

 

```
#!/bin/bash
#
docker build -t centos7/nginx-base:v1 .
```

 

```
#!/bin/bash
#
docker build -t centos7/nginx-base:v1 .
```

启动测试

 

```
docker run -it -p 8001:80 centos7/nginx-base:v1
```

 

```
docker run -it -p 8001:80 centos7/nginx-base:v1
```

http://19.87.100.101:8001

### Dockerfile制作mysql镜像

构建的原理：

1，利用Dockerfile进行安装MySQL服务（yum安装或者以rpm包安装（由于网络问题可将需要安装的包下载到本地进行安装））

2，编写shell脚本，将安装好的mariadb进行重新初始化，并启动mariadb，执行需要的sql脚本，关闭mariadb，最后通过前台开启服务

3，由于MySQL5.6和MySQL5.7的初始化方式不一样，本文介绍的适用于MySQL5.6（后面会有5.7的案例）

### Dockerfile制作tomcat镜像 

基于上面创建的centos-base镜像构建JDK和tomcat镜像，先构建JDK镜像，然后再基于JDK镜像构建tomcat镜像，再构建2个tomcat业务实例镜像app01和app02

1,构建JDK镜像

 

```
cd /opt/dockerfile/system/centos
mkdir jdk-base
cd jdk-base/

vim Dockerfile
"""
# JDK Base Image

FROM centos-base:latest

MAINTAINER xiaofei 1023668666@qq.com

ADD jdk-8u162-linux-x64.tar.gz /usr/local/src/

RUN ln -sv /usr/local/src/jdk1.8.0_162 /usr/local/jdk

ADD profile /etc/profile

# 虽然下面的环境变量声明已经写在了profile中，但是docker容器在启动时不会自动去读取这个文件，可以进入到容器中手动加载，也可以在构建镜像时手动加载。
ENV JAVA_HOME /usr/local/jdk
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/:$JRE_HOME/lib/
ENV PATH $PATH:$JAVA_HOME/bin

RUN rm -rf /etc/localtime && ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone
"""
```

 

```
cd /opt/dockerfile/system/centos
mkdir jdk-base
cd jdk-base/
vim Dockerfile
"""
# JDK Base Image
FROM centos-base:latest
MAINTAINER xiaofei 1023668666@qq.com
ADD jdk-8u162-linux-x64.tar.gz /usr/local/src/
RUN ln -sv /usr/local/src/jdk1.8.0_162 /usr/local/jdk
ADD profile /etc/profile
# 虽然下面的环境变量声明已经写在了profile中，但是docker容器在启动时不会自动去读取这个文件，可以进入到容器中手动加载，也可以在构建镜像时手动加载。
ENV JAVA_HOME /usr/local/jdk
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/:$JRE_HOME/lib/
ENV PATH $PATH:$JAVA_HOME/bin
RUN rm -rf /etc/localtime && ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone
"""
```

2,上传JDK压缩包和profile文件，将JDK 压缩包上传到Dockerfile当前目录

profile文件最后添加：

 

```
export JAVA_HOME=/usr/local/jdk
export TOMCAT_HOME=/apps/tomcat
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$TOMCAT_HOME/bin:$PATH
export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar
```

 

```
export JAVA_HOME=/usr/local/jdk
export TOMCAT_HOME=/apps/tomcat
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$TOMCAT_HOME/bin:$PATH
export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar
```

jdk包下载地址：

https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

然后执行构建：

 

```
ls
"""
Dockerfile  jdk-8u162-linux-x64.tar.gz  profile
"""

vim build-command.sh
"""
docker build -t jdk-base .
"""

chmod +x build-command.sh

bash build-command.sh

docker run -it --rm jdk-base /bin/bash

java -version
```

 

```
ls
"""
Dockerfile  jdk-8u162-linux-x64.tar.gz  profile
"""
vim build-command.sh
"""
docker build -t jdk-base .
"""
chmod +x build-command.sh
bash build-command.sh
docker run -it --rm jdk-base /bin/bash
java -version
```

3, 基于上面构建的jdk-base镜像，制作tomcat-base镜像

下载地址：https://tomcat.apache.org/download-80.cgi

 

```
mkdir tomcat8-base
ls
"""
apache-tomcat-8.0.49.tar.gz  build-command.sh  Dockerfile
"""

cat Dockerfile
"""
# Tomcat8 Base Image

FROM jdk-base:latest

MAINTAINER xiaofei 1023668666@qq.com

RUN useradd www -u 2000

#env settinf
ENV TZ "Asia/Shanghai"
ENV LANG en_US.UTF-8
ENV TERM xterm
ENV TOMCAT_MAJOR_VERSION 8
ENV TOMCAT_MINOR_VERSION 8.0.49
ENV CATALINA_HOME /apps/tomcat
ENV APP_DIR ${CATALINA_HOME}/webapps

RUN mkdir /apps 
ADD apache-tomcat-8.0.49.tar.gz /apps  
RUN ln -sv /apps/apache-tomcat-8.0.49 /apps/tomcat
"""

cat build-command.sh
"""
#!/bin/bash
docker build -t tomcat8-base .
"""

bash build-command.sh
docker images
```

 

```
mkdir tomcat8-base
ls
"""
apache-tomcat-8.0.49.tar.gz  build-command.sh  Dockerfile
"""
cat Dockerfile
"""
# Tomcat8 Base Image
FROM jdk-base:latest
MAINTAINER xiaofei 1023668666@qq.com
RUN useradd www -u 2000
#env settinf
ENV TZ "Asia/Shanghai"
ENV LANG en_US.UTF-8
ENV TERM xterm
ENV TOMCAT_MAJOR_VERSION 8
ENV TOMCAT_MINOR_VERSION 8.0.49
ENV CATALINA_HOME /apps/tomcat
ENV APP_DIR ${CATALINA_HOME}/webapps
RUN mkdir /apps 
ADD apache-tomcat-8.0.49.tar.gz /apps  
RUN ln -sv /apps/apache-tomcat-8.0.49 /apps/tomcat
"""
cat build-command.sh
"""
#!/bin/bash
docker build -t tomcat8-base .
"""
bash build-command.sh
docker images
```

4, 构建业务镜像tomcat8-app01和tomcat8-app02

创建tomcat-app1和tomcat-app2两个目录，代表不同的两个基于tomcat的业务

在生产环境中，tomcat的web页面路径一般会修改，通过conf/server.xml

 

```
mkdir tomcat8-app{01,02} -pv
cd tomcat8-app01/
cat Dockerifle
"""
# Tomcat8 APP01 Image

FROM tomcat8-base:latest

MAINTAINER xiaofei 1023668666@qq.com

ENV JAVA_HOME /usr/local/jdk
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/:$JRE_HOME/lib/
ENV PATH $PATH:$JAVA_HOME/bin

RUN rm -rf /etc/localtime && ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone

RUN useradd www -u 2000

ADD run_tomcat.sh /apps/tomcat8/bin/
ADD myapp/* /apps/tomcat8/webapps/myapp/

RUN chown -R www.www /apps/tomcat8/*

CMD ["/apps/tomcat8/bin/run_tomcat.sh"]
EXPOSE 8080 8009
"""

cat run_tomcat.sh
"""
# 此脚本用于启动容器时执行，可理解为开机启动
#!/bin/bash
su - www -c "/apps/tomcat8/bin/catalina.sh start"
su - www -c "tail -f /etc/hosts"
"""

cat build-command.sh
"""
#!/bin/bash
docker build -t tomcat8-app01 .
"""

mkdir myapp
cat myapp/index.html
"""
from tomcat-app01!
"""

bash build-command.sh

# tomcat8-app02构建步骤同上，此处略
# 根据2个镜像启动容器
docker run -it -p 8888:8080 tomcat8-app01
docker run -it -p 8889:8080 tomcat8-app02
```

 

```
mkdir tomcat8-app{01,02} -pv
cd tomcat8-app01/
cat Dockerifle
"""
# Tomcat8 APP01 Image
FROM tomcat8-base:latest
MAINTAINER xiaofei 1023668666@qq.com
ENV JAVA_HOME /usr/local/jdk
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/:$JRE_HOME/lib/
ENV PATH $PATH:$JAVA_HOME/bin
RUN rm -rf /etc/localtime && ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone
RUN useradd www -u 2000
ADD run_tomcat.sh /apps/tomcat8/bin/
ADD myapp/* /apps/tomcat8/webapps/myapp/
RUN chown -R www.www /apps/tomcat8/*
CMD ["/apps/tomcat8/bin/run_tomcat.sh"]
EXPOSE 8080 8009
"""
cat run_tomcat.sh
"""
# 此脚本用于启动容器时执行，可理解为开机启动
#!/bin/bash
su - www -c "/apps/tomcat8/bin/catalina.sh start"
su - www -c "tail -f /etc/hosts"
"""
cat build-command.sh
"""
#!/bin/bash
docker build -t tomcat8-app01 .
"""
mkdir myapp
cat myapp/index.html
"""
from tomcat-app01!
"""
bash build-command.sh
# tomcat8-app02构建步骤同上，此处略
# 根据2个镜像启动容器
docker run -it -p 8888:8080 tomcat8-app01
docker run -it -p 8889:8080 tomcat8-app02
```

测试访问：

172.16.2.101:8888/myapp/

172.16.2.101:8889/myapp/

### Dockerfile制作HAProxy镜像

实验要求：

haproxy负载代理至上面两个tomcat业务实例中去。

 

```
mkdir haproxy
cd haproxy/
wget http://xxx/haproxy-1.8.12.tar.gz

vim Dockerfile
"""
#Haproxy Base Image

FROM centos-base:latest

MAINTAINER xiaofei "1023668666@qq.com"

RUN  yum install gcc gcc-c++ glibc glibc-devel pcre pcre-devel openssl  openssl-devel systemd-devel net-tools vim iotop bc  zip unzip zlib-devel lrzsz tree screen lsof tcpdump wget ntpdate -y

ADD haproxy-1.8.12.tar.gz /usr/local/src/

RUN cd /usr/local/src/haproxy-1.8.12 &&  make  ARCH=x86_64 TARGET=linux2628 USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1 USE_SYSTEMD=1  USE_CPU_AFFINITY=1  PREFIX=/usr/local/haproxy && make install PREFIX=/usr/local/haproxy && cp haproxy  /usr/sbin/ && mkdir /usr/local/haproxy/run

ADD haproxy.cfg /etc/haproxy/

ADD run_haproxy.sh /usr/bin

EXPOSE 80 9999

CMD ["/usr/bin/run_haproxy.sh"]
"""

vim haproxy.cfg
"""
global
chroot /usr/local/haproxy
#stats socket /var/lib/haproxy/haproxy.sock mode 600 level admin
uid 99
gid 99
daemon
nbproc 1
pidfile /usr/local/haproxy/run/haproxy.pid
log 127.0.0.1 local3 info

defaults
option http-keep-alive
option  forwardfor
mode http
timeout connect 300000ms
timeout client  300000ms
timeout server  300000ms

listen stats
 mode http
 bind 0.0.0.0:9999
 stats enable
 log global
 stats uri     /haproxy-status
 stats auth    haadmin:q1w2e3r4ys

listen  web_port
 bind 0.0.0.0:80
 mode http
 log global
 balance roundrobin
 server web1  172.16.2.101:8888  check inter 3000 fall 2 rise 5
 server web2  172.16.2.101:8889  check inter 3000 fall 2 rise 5
"""

vim run_haproxy.sh
"""
#!/bin/bash
haproxy -f /etc/haproxy/haproxy.cfg

tail -f /etc/hosts
"""

vim build-command.sh
"""
#!/bin/bash
docker build -t centos-haproxy-base:7.5-1.8.12 .
"""

bash build-command.sh
docker images
docker run -it -d -p80:80 -p9999:9999 centos-haproxy-base:7.5-1.8.12
```

 

```
mkdir haproxy
cd haproxy/
wget http://xxx/haproxy-1.8.12.tar.gz
vim Dockerfile
"""
#Haproxy Base Image
FROM centos-base:latest
MAINTAINER xiaofei "1023668666@qq.com"
RUN  yum install gcc gcc-c++ glibc glibc-devel pcre pcre-devel openssl  openssl-devel systemd-devel net-tools vim iotop bc  zip unzip zlib-devel lrzsz tree screen lsof tcpdump wget ntpdate -y
ADD haproxy-1.8.12.tar.gz /usr/local/src/
RUN cd /usr/local/src/haproxy-1.8.12 &&  make  ARCH=x86_64 TARGET=linux2628 USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1 USE_SYSTEMD=1  USE_CPU_AFFINITY=1  PREFIX=/usr/local/haproxy && make install PREFIX=/usr/local/haproxy && cp haproxy  /usr/sbin/ && mkdir /usr/local/haproxy/run
ADD haproxy.cfg /etc/haproxy/
ADD run_haproxy.sh /usr/bin
EXPOSE 80 9999
CMD ["/usr/bin/run_haproxy.sh"]
"""
vim haproxy.cfg
"""
global
chroot /usr/local/haproxy
#stats socket /var/lib/haproxy/haproxy.sock mode 600 level admin
uid 99
gid 99
daemon
nbproc 1
pidfile /usr/local/haproxy/run/haproxy.pid
log 127.0.0.1 local3 info
defaults
option http-keep-alive
option  forwardfor
mode http
timeout connect 300000ms
timeout client  300000ms
timeout server  300000ms
listen stats
 mode http
 bind 0.0.0.0:9999
 stats enable
 log global
 stats uri     /haproxy-status
 stats auth    haadmin:q1w2e3r4ys
listen  web_port
 bind 0.0.0.0:80
 mode http
 log global
 balance roundrobin
 server web1  172.16.2.101:8888  check inter 3000 fall 2 rise 5
 server web2  172.16.2.101:8889  check inter 3000 fall 2 rise 5
"""
vim run_haproxy.sh
"""
#!/bin/bash
haproxy -f /etc/haproxy/haproxy.cfg
tail -f /etc/hosts
"""
vim build-command.sh
"""
#!/bin/bash
docker build -t centos-haproxy-base:7.5-1.8.12 .
"""
bash build-command.sh
docker images
docker run -it -d -p80:80 -p9999:9999 centos-haproxy-base:7.5-1.8.12
```

测试访问：

172.16.2.101/myapp/

172.16.2.101:9999/haproxy-status/

账号密码在haproxy配置文件中