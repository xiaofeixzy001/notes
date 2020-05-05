[TOC]

# 知识回顾

在进行安装之前，先回顾几个知识点：

## CGI

common Gateway interface

一种协议，HTTP服务器与其他程序进行“交谈”的一种工具，其程序须运行在网络服务器上，如php等；

cgi方式的服务器有多少连接请求就会有多少cgi子进程，每个子进程都需要启动cgi解释器，加载配置，连接其他服务器等初始化工作，这是cgi性能低下的主要原因，当用户请求数量非常多时，会大量挤占系统的资源时间，造成效能低下。

## FASTCGI

FastCGI是语言无关的、可伸缩架构的CGI开放扩展，其主要行为是将CGI解释器进程保持在内存中并因此获得较高的性能。众所周知，CGI解释器的反复加载是CGI性能低下的主要原因，如果CGI解释器保持在内存中并接受FastCGI进程管理器调度，则可以提供良好的性能、伸缩性、Fail- Over特性等等。

FASTCGI是常驻型的CGI，它可以一直运行，在请求到达时，不会花费时间去fork一个进程来处理。

一般情况下，FastCGI的整个工作流程是这样的：

1、Web Server启动时载入FastCGI进程管理器

​    apache(mod_fastcgi)

​    nginx(ngx_http_fastcgi_module)

​    lighttpd(mod_fastcgi)

2、FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待WebServer的连接。

3、当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。 Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。

4、FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。在CGI模式中，php-cgi在此便退出了。

## FastCGI的不足

因为是多进程，所以比CGI多线程消耗更多的服务器内存，PHP-CGI解释器每进程消耗7至25兆内存，将这个数字乘以50或100就是很大的内存数。

## PHP-CGI

PHP-CGI：php解释器，是PHP自带的FastCGI管理器，php-cgi只是个CGI程序，他自己本身只能解析请求，返回结果，不会进程管理，所以就出现了一些能够调度php-cgi进程的程序，如php-fpm

## PHP-CGI的不足

\1. php-cgi变更php.ini配置后需重启php-cgi才能让新的php-ini生效，不可以平滑重启。

\2. 直接杀死php-cgi进程，php就不能运行了。(PHP-FPM和Spawn-FCGI就没有这个问题，守护进程会平滑从新生成新的子进程。）



PHP-FPM:(php fastcgi process Manager)

PHP-FPM是一个PHP FastCGI管理器，是只用于PHP的，可以在http://php-fpm.org/download下载得到。

PHP-FPM其实是PHP源代码的一个补丁，旨在将FastCGI进程管理整合进PHP包中。必须将它patch到你的PHP源代码中，在编译安装PHP后才可以使用。PHP5.3.3已经集成php-fpm了，不再是第三方的包了。PHP-FPM提供了更好的PHP进程管理方式，可以有效控制内存和进程、可以平滑重载PHP配置，比spawn-fcgi具有更多有点，所以被PHP官方收录了。在./configure的时候带 –enable-fpm参数即可开启PHP-FPM。

## Spawn-FCGI

Spawn-FCGI是一个通用的FastCGI管理服务器，它是lighttpd中的一部份，很多人都用Lighttpd的Spawn-FCGI进行FastCGI模式下的管理工作，不过有不少缺点。而PHP-FPM的出现多少缓解了一些问题，但PHP-FPM有个缺点就是要重新编译，这对于一些已经运行的环境可能有不小的风险(refer)，在php 5.3.3中可以直接使用PHP-FPM了。

Spawn-FCGI目前已经独成为一个项目，更加稳定一些，也给很多Web 站点的配置带来便利。已经有不少站点将它与nginx搭配来解决动态网页。

最新的lighttpd也没有包含这一块了(http://www.lighttpd.net/search?q=Spawn-FCGI)，但可以在以前版本中找到它。在lighttpd-1.4.15版本中就包含了(http://www.lighttpd.net/download/lighttpd-1.4.15.tar.gz)，目前Spawn-FCGI的下载地址是http://redmine.lighttpd.net/projects/spawn-fcgi，最新版本是http://www.lighttpd.net/download/spawn-fcgi-1.6.3.tar.gz。

注：最新的Spawn-FCGI可以到lighttpd.net网站搜索“Spawn-FCGI”找到它的最新版本发布地址。



举个例子：

你(PHP)去和爱斯基摩人(web服务器，如Apache、Nginx)谈生意你说中文(PHP代码)，他说爱斯基摩语(C代码)，互相听不懂，怎么办？那就都把各自说的话转换成英语(FastCGI协议)吧。怎么转换呢？你就要使用一个翻译机(PHP-FPM)(当然对方也有一个翻译机，那个是他自带的)我们这个翻译机是最新型的，老式的那个（PHP-CGI）被淘汰了。不过它(PHP-FPM)只有年轻人（Linux系统）会用，老头子们（Windows系统）不会摆弄它，只好继续用老式的那个。

## PHP-FPM与spawn-CGI对比

PHP-FPM的使用非常方便，配置都是在PHP-FPM.ini的文件内，而启动、重启都可以从php/sbin/PHP-FPM中进行。更方便的是修改php.ini后可以直接使用PHP-FPM reload进行加载，无需杀掉进程就可以完成php.ini的修改加载

结果显示使用PHP-FPM可以使php有不小的性能提升。PHP-FPM控制的进程cpu回收的速度比较慢,内存分配的很均匀。

Spawn-FCGI控制的进程CPU下降的很快，而内存分配的比较不均匀。有很多进程似乎未分配到，而另外一些却占用很高。可能是由于进程任务分配的不均匀导致的。而这也导致了总体响应速度的下降。而PHP-FPM合理的分配，导致总体响应的提到以及任务的平均。

## 总结

fastCGI是nginx和php之间的一个通信接口，该接口实际处理过程通过启动php-fpm进程来解析php脚本，即php-fpm相当于一个动态应用服务器，从而实现nginx动态解析php。因此，如果nginx服务器需要支持php解析，需要在nginx.conf中增加php的配置：将php脚本转发到fastCGI进程监听的IP地址和端口（php-fpm.conf中指定）。同时，php安装的时候，需要开启支持fastCGI选项，并且编译安装php-fpm补丁/扩展，同时，需要启动php-fpm进程，才可以解析nginx通过fastCGI转发过来的php脚本。

## Nginx工作原理

传统上基于进程或线程模型架构的web服务通过每进程或每线程处理并发连接请求，这势必会在网络和I/O操作时产生阻塞，其另一个必然结果则是对内存或CPU的利用率低下。生成一个新的进程/线程需要事先备好其运行时环境，这包括为其分配堆内存和栈内存，以及为其创建新的执行上下文等。这些操作都需要占用CPU，而且过多的进程/线程还会带来线程抖动或频繁的上下文切换，系统性能也会由此进一步下降。在设计的最初阶段，nginx的主要着眼点就是其高性能以及对物理计算资源的高密度利用，因此其采用了不同的架构模型。受启发于多种操作系统设计中基于“事件”的高级处理机制，nginx采用了模块化、事件驱动、异步、单线程及非阻塞的架构，并大量采用了多路复用及事件通知机制。在nginx中，连接请求由为数不多的几个仅包含一个线程的进程worker以高效的回环(run-loop)机制进行处理，而每个worker可以并行处理数千个的并发连接及请求。如果负载以CPU密集型应用为主，如SSL或压缩应用，则worker数应与CPU数相同；如果负载以IO密集型为主，如响应大量内容给客户端，则worker数应该为CPU个数的1.5或2倍.



Nginx会按需同时运行多个进程：一个主进程(master)和几个工作进程(worker)，配置了缓存时还会有缓存加载器进程(cache loader)和缓存管理器进程(cache manager)等。所有进程均是仅含有一个线程，并主要通过“共享内存”的机制实现进程间通信。主进程以root用户身份运行，而worker、cache loader和cache manager均应以非特权用户身份运行.



主进程主要完成如下工作：

   \1. 读取并验正配置信息；

   \2. 创建、绑定及关闭套接字；

   \3. 启动、终止及维护worker进程的个数；

   \4. 无须中止服务而重新配置工作特性；

   \5. 控制非中断式程序升级，启用新的二进制程序并在需要时回滚至老版本；

   \6. 重新打开日志文件，实现日志滚动；

   \7. 编译嵌入式perl脚本；

worker进程主要完成的任务包括：

​     \1. 接收、传入并处理来自客户端的连接；

​     \2. 提供反向代理及过滤功能；

​     \3. nginx任何能完成的其它任务；

cache loader进程主要完成的任务包括：

​     \1. 检查缓存存储中的缓存对象；

​     \2. 使用缓存元数据建立内存数据库；

cache manager进程的主要任务：

​     \1. 缓存的失效及过期检验;



Nginx的配置有着几个不同的上下文：main、http、server、upstream和location(还有实现邮件服务反向代理的mail),配置语法的格式和定义方式遵循所谓的C风格，因此支持嵌套，还有着逻辑清晰并易于创建、阅读和维护等优势;



Nginx的代码是由一个核心和一系列的模块组成, 核心主要用于提供Web Server的基本功能，以及Web和Mail反向代理的功能；还用于启用网络协议，创建必要的运行时环境以及确保不同的模块之间平滑地进行交互。不过，大多跟协议相关的功能和某应用特有的功能都是由nginx的模块实现的。这些功能模块大致可以分为事件模块、阶段性处理器、输出过滤器、变量处理器、协议、upstream和负载均衡几个类别，这些共同组成了nginx的http功能;

事件模块主要用于提供OS独立的(不同操作系统的事件机制有所不同)事件通知机制如kqueue或epoll等。协议模块则负责实现nginx通过http、tls/ssl、smtp、pop3以及imap与对应的客户端建立会话。在nginx内部，进程间的通信是通过模块的pipeline或chain实现的；换句话说，每一个功能或操作都由一个模块来实现。例如，压缩、通过FastCGI或uwsgi协议与upstream服务器通信，以及与memcached建立会话等。

# 部署配置

## 环境

CentOS 6.9 x86_64

nginx-1.14.2

mysql-5.6.41-linux-glibc2.12-x86_64

php-5.6.22.tar.gz

## 需求

1,部署在同一台主机

2,开启虚拟主机

3,php以php-fpm方式启动

## 步骤

### nginx

#### 安装

编译安装目录/apps/nginx

配置文件/apps/nginx/conf/nginx.conf

日志/apps/nginx/logs/

pid文件/apps/nginx/run/nginx.pid

锁文件/apps/nginx/run/nginx.lock

```shell
# 安装依赖包
[root@linux-study ~]# yum -y groupinstall "Server Platform Development" "Development Tools"
[root@linux-study ~]# yum -y install pcre pcre-devel openssl openssl-devel zlib-devel gcc
[root@linux-study ~]# rpm -q pcre pcre-devel openssl openssl-devel
[root@linux-study ~]# groupadd -r nginx
[root@linux-study ~]# useradd -r -g nginx -s /bin/false -M nginx
[root@linux-study ~]# mkdir -pv /apps/nginx/{logs,run,client,proxy,fcgi,uwsgi,scgi}

# 编译安装Nginx
[root@linux-study ~]# tar xf nginx-1.12.2.tar.gz
[root@linux-study ~]# cd nginx-1.12.2/
[root@linux-study ~]# ./configure \
--prefix=/apps/nginx \
--conf-path=/apps/nginx/conf/nginx.conf \
--sbin-path=/apps/nginx/sbin/nginx \
--error-log-path=/apps/nginx/logs/error.log \
--http-log-path=/apps/nginx/logs/access.log \
--pid-path=/apps/nginx/run/nginx.pid  \
--lock-path=/apps/nginx/run/nginx.lock \
--user=nginx \
--group=nginx \
--with-http_ssl_module \
--with-http_flv_module \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--http-client-body-temp-path=/apps/nginx/client/ \
--http-proxy-temp-path=/apps/nginx/proxy/ \
--http-fastcgi-temp-path=/apps/nginx/fcgi/ \
--http-uwsgi-temp-path=/apps/nginx/uwsgi \
--http-scgi-temp-path=/apps/nginx/scgi \
--with-pcre

[root@linux-study nginx-1.12.2]# make && make install

# 添加脚本
[root@linux-study nginx-1.12.2]# vim /etc/rc.d/init.d/nginx
"""
脚本附在最后.注意启动脚本的配置文件路径
"""
[root@linux-study nginx-1.12.2]# chmod +x /etc/rc.d/init.d/nginx 
[root@linux-study nginx-1.12.2]# chkconfig --add nginx
[root@linux-study nginx-1.12.2]# chkconfig --level 3 nginx on

[root@linux-study nginx-1.12.2]# service nginx start
```

浏览器测试访问

#### 配置

开启虚拟主机

```shell
vim /apps/nginx/conf/nginx.conf
# 编辑配置文件，server段去掉，在http字段内添加include
"""
worker_processes  1;

error_log   /apps/nginx/logs/error.log  notice;

events {
    worker_connections  10240;
    multi_accept        on;
    use                 epoll;
}
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile      on;

    gzip          on;
    gzip_min_length 1k;
    gzip_comp_level 3;
    gzip_types text/plain application/javascript application/x-javascript text/javascript text/xml text/css;
    gzip_disable "MSIE[1-6]";
    gzip_vary on;

    keepalive_timeout  65;
    include extra/*.conf;  # 添加
    server_tokens off;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    #access_log  /applications/nginx/logs/access.log  main;
}

# 如果extra不存在则创建一个即可
mkdir /apps/nginx/conf/extra
cd /apps/nginx/conf/extra/
vim vhosts.conf  # 基于tomcat的配置
"""
server{
    listen 80;
    server_name 90live.cn;
    rewrite ^(.*)$ https://${server_name}$1 permanent;
}

server {
    listen 443;
    ssl on;
    server_name  90live.cn;
    client_max_body_size 100M;
    root /apps/tomcat/webapps;
    index index.jsp index.html;

    ssl_certificate   /apps/nginx/cert/crm/1718889_90live.cn.pem;
    ssl_certificate_key  /apps/nginx/cert/crm/1718889_90live.cn.key;

    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host:$server_port;
        proxy_set_header X-Forwarded-Proto https;
        proxy_redirect off;
        proxy_connect_timeout      240;
        proxy_send_timeout         240;
        proxy_read_timeout         240;
        proxy_pass http://127.0.0.1:8080/;
    }
}
"""

/apps/nginx/sbin/nginx -t 
/apps/nginx/sbin/nginx -s reload
```

浏览器测试

### mysql

这里使用二进制安装

 

```
groupadd -r mysql
useradd -g mysql -r -s /sbin/nologin -M -d /data/mysql/ mysql
mkdir /data/mysql
chown -R mysql:mysql /data/mysql

tar xf mysql-5.5.59-linux-glibc2.12-x86_64.tar.gz -C /usr/local/
cd /usr/local/
ln -sv mysql-5.5.59-linux-glibc2.12-x86_64 mysql
cd mysql
cp support-files/my-small.cnf /etc/my.cnf

vim /etc/my.cnf
"""
thread_concurrency = 2  # CPU个数x2
datadir = /data/mysql   # 指定数据文件存放位置
"""

# 安装MySQL依赖函数库，否则下面的初始化有可能会失败
[root@node1 mysql]# yum install -y libaio  

# 输出mysql的man手册至man命令的查找路径
[root@node1 mysql]# vim /etc/man.config
"""
MANPATH  /usr/local/mysql/man
"""

# 输出mysql的头文件至系统头文件路径/usr/include
[root@node1 mysql]# ln -sv /usr/local/mysql/include  /usr/include/mysql

# 输出mysql的库文件给系统库查找路径
[root@node1 mysql]# echo '/usr/local/mysql/lib' > /etc/ld.so.conf.d/mysql.conf
[root@node1 mysql]# ldconfig

# 添加mysql环境变量
[root@node1 mysql]# echo 'export PATH=/usr/local/mysql/bin:$PATH' >>/etc/profile   \\添加mysql的环境变量
[root@node1 mysql]# source /etc/profile
[root@node1 mysql]# echo $PATH
/usr/mysql/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin

# 初始化
[root@node1 mysql]# scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/data/mysql/ --user=mysql

# 开机启动服务脚本,注意检查路径是否匹配
[root@node1 mysql]# cp support-files/mysql.server /etc/init.d/mysqld
[root@node1 mysql]# chmod +x /etc/init.d/mysqld
[root@node1 mysql]# /etc/init.d/mysqld start
[root@node1 mysql]# netstat -lntup | grep mysql
[root@node1 mysql]# chkconfig --add mysqld
[root@node1 mysql]# chkconfig mysqld on

# 登录mysql,优化root安全, 然后添加2个可被远程使用的用户,为php连接做准备.
[root@node1 mysql]# mysql
mysql> SELECT user,host,password FROM mysql.user;
mysql> DELETE FROM mysql.user WHERE user='';
mysql> DELETE FROM mysql.user WHERE host='::1';

mysql> USE mysql;
mysql> GRANT ALL PRIVILEGES ON *.* TO root@"localhost" IDENTIFIED BY "123.com";
mysql> GRANT ALL PRIVILEGES ON *.* TO root@"172.16.100.%" IDENTIFIED BY "123.com";
mysql> GRANT ALL PRIVILEGES ON *.* TO root@"127.0.0.1" IDENTIFIED BY "123.com";
mysql> FLUSH PRIVILEGES;

mysql> CREATE DATABASE wpdb;
mysql> CREATE DATABASE phpdb;
mysql> GRANT ALL ON wpdb.* TO 'wpuser'@'172.16.100.%' IDENTIFIED BY '123.com';
mysql> GRANT ALL ON phpdb.* TO 'phpuser'@'172.16.100.%' IDENTIFIED BY '123.com';
mysql> FLUSH PRIVILEGES;
```

如果误删root或无法本地登录解决办法：

```
[root@node1 ~]# mysql_safe --skip-grant-tables
mysql> INSERT INTO mysql.user (host, user, password) VALUES ('localhost', 'root', PASSWORD('密码'));
mysql> FLUSH PRIVILEGES;
mysql> GRANT ALL ON *.* TO 'root'@'localhost';
```

登录测试

### 安装PHP

略

#### 整合nginx和php

在nginx服务器上操作

因为我们开启了2个虚拟机,v1作为WordPress, v2作为phpmyadmin,所以我们需要编辑虚拟主机文件,将php的请求转到php服务器上去.

```
[root@study-linux ~]# cd /usr/local/nginx/conf/extra
[root@study-linux extra]# vim vhosts.conf
"""
server {
    listen 80;
    server_name v1.com;
    
    # 默认所有请求,转到/data/vhosts/v1去查找
    location / {
        root /data/vhosts/v1;
        index index.php index.html index.htm;
    }
    
    # 以.php结尾的所有请求,通过fastcgi协议,转到172.16.100.40:9000中的/data/wp去查找
    location ~ \.php$ {
        root         /data/wp;
        fastcgi_pass 172.16.100.40:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /scripts$fastcgi_script_name;
        include       fastcgi_params;
    }
}

server {
    listen 80;
    server_name v2.com;
    location / {
        root /data/vhosts/v2;
        index index.php index.html index.htm;
    }
    location ~ \.php$ {
        root         /data/php;
        fastcgi_pass 172.16.100.40:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /scripts$fastcgi_script_name;
        include       fastcgi_params;
    }
}
"""

 [root@study-linux extra]# vim /etc/nginx/fastcgi_params
 """ 
 # 增加如下行
 fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
 """
```

4-2,在PHP服务器上操作

```
[root@node1 ~]# groupadd -r nginx
[root@node1 ~]# useradd -g nginx -r -s /sbin/nologin -M nginx
[root@node1 ~]# mkdir -pv /data/{wp,php}
[root@node1 ~]# chown -R nginx.nginx /data/*
[root@node1 ~]# ls -l /data/

# 修改配置文件
[root@node1 ~]# vim /usr/local/php/etc/php-fpm.conf
"""
user = nginx
group = nginx
listen = 172.16.100.40:9000
"""

# 添加php测试页
[root@node1 data]# vim wp/index.php
"""
<h3>from wp!!</h3>

<?php
$link_id=mysql_connect('172.16.100.30', 'wpuser', '123.com');
    if($link_id){
        echo "mysql succesful!";
    }
    else{
        echo "mysql_error()";
    }
    phpinfo();
?>
"""

[root@node1 data]# vim php/index.php
"""
<h3>from php!!</h3>

<?php
$link_id=mysql_connect('172.16.100.30', 'phpuser', '123.com');
    if($link_id){
        echo "mysql succesful!";
    }
    else{
        echo "mysql_error()";
    }
    phpinfo();
?>
"""
```

测试访问:

http://v1.com/index.php

http://v2.com/index.php

5,部署WordPress和phpMyAdmin

5-1,配置wordpress

 

```
[root@node1 packages]# wget https://cn.wordpress.org/wordpress-4.9.4-zh_CN.tar.gz
[root@node1 packages]# tar xf wordpress-4.9.4-zh_CN.tar.gz -C /data/wp/
[root@node1 packages]# cd /data/wp
[root@node1 wp]# cp -r wordpress/* .
[root@node1 wp]# chown -R nginx.nginx ./*
[root@node1 wp]# ll
```

5-2,配置phpMyAdmin

 

```
[root@node1 packages]# wget https://files.phpmyadmin.net/phpMyAdmin/4.7.7/phpMyAdmin-4.7.7-all-languages.tar.gz
[root@node1 packages]# tar xf phpMyAdmin-4.7.7-all-languages.tar.gz -C /data/php/
[root@node1 packages]# cd /data/php
[root@node1 php]# cp -r phpMyAdmin-4.7.7-all-languages/* .
[root@node1 php]# chown -R nginx.nginx *
[root@node1 wp]# ll
```

最后访问:

http://v1.com

http://v1.com

根据提示进行安装配置即可.

附：Nginx服务脚本

 

```shell
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /applications/nginx/conf/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /applications/nginx/run/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/apps/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/apps/nginx/conf/nginx.conf"

[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx

lockfile=/apps/nginx/run/nginx.lock

make_dirs() {
   # make required directories
   user=`nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   options=`$nginx -V 2>&1 | grep 'configure arguments:'`
   for opt in $options; do
       if [ `echo $opt | grep '.*-temp-path'` ]; then
           value=`echo $opt | cut -d "=" -f 2`
           if [ ! -d "$value" ]; then
               # echo "creating" $value
               mkdir -p $value && chown -R $user $value
           fi
       fi
   done
}

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```