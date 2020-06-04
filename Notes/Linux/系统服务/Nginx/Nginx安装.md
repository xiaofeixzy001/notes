[TOC]

# 环境



# 需求

操作系统：CentOS release 6.10 (Final)

IP地址：192.168.100.60

Nginx版本：nginx-1.16.1

安装方式：源码编译安装

主程序安装目录：/apps/nginx

配置文件目录：/apps/nginx/conf/nginx.conf

日志文件目录：/apps/nginx/logs/

PID文件目录：/apps/nginx/run/nginx.pid

LOCK文件目录：/apps/nginx/run/nginx.lock



# 安装

```shell
# 安装依赖包
[root@LNMP ~]# yum -y install pcre pcre-devel openssl openssl-devel zlib-devel gcc
[root@LNMP ~]# rpm -q pcre pcre-devel openssl openssl-devel
[root@LNMP ~]# groupadd -r nginx
[root@LNMP ~]# useradd -r -g nginx -s /bin/false -M nginx
[root@LNMP ~]# mkdir -pv /apps/nginx/{logs,run,client,proxy,fcgi,uwsgi,scgi}

# 编译安装Nginx
[root@LNMP ~]# tar xf nginx-1.16.1.tar.gz
[root@LNMP ~]# cd nginx-1.16.1/
[root@LNMP nginx-1.16.1]# ./configure \
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

[root@LNMP nginx-1.16.1]# make && make install

# 添加脚本
[root@LNMP nginx-1.16.1]# vim /etc/rc.d/init.d/nginx
"""
脚本附在最后.注意启动脚本的配置文件路径
"""
[root@LNMP nginx-1.16.1]# chmod +x /etc/rc.d/init.d/nginx 
[root@LNMP nginx-1.16.1]# chkconfig --add nginx
[root@LNMP nginx-1.16.1]# chkconfig --level 3 nginx on
[root@LNMP nginx-1.16.1]# service nginx start
```

测试访问：http://192.168.100.60/

# 配置

开启虚拟机

```shell
[root@LNMP ~]# cd /apps/nginx/conf/
[root@LNMP conf]# vim nginx.conf
"""
user  nginx;
worker_processes  1;

error_log  /apps/nginx/logs/error.log  notice;

pid        /apps/nginx/run/nginx.pid;

events {
    worker_connections  10240;
    multi_accept        on;
    use                 epoll;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /apps/nginx/logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  65;
    include extra/*.conf;
    server_tokens off;
    gzip  on;
    gzip_min_length 1k;
    gzip_comp_level 3;
    gzip_types text/plain application/javascript application/x-javascript text/javascript text/xml text/css;
    gzip_disable "MSIE[1-6]";
    gzip_vary on;
}
"""

[root@LNMP conf]# ../sbin/nginx -t
[root@LNMP conf]# mkdir extra
[root@LNMP conf]# vim extra/tomcatweb.conf
"""
# tomcat
server{
    listen 80;
    server_name 90live.cn;
    rewrite ^(.*)$ https://${server_name}$1 permanent;
}

server {
    listen 443;
    ssl on;
    server_name  blizzardwind.com;
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

[root@LNMP conf]# vim extra/phpweb.conf
"""
server {
    listen 80;
    server_name blizzardwind.com;
    
    location / {
        root /data/webroot/;
        index index.php index.html index.htm;
    }

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /scripts$fastcgi_script_name;
        include       fastcgi_params;
    }
}
"""

[root@LNMP conf]# vim fastcgi_params
 """ 
 # 增加如下行
 fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
 """
```

去掉server段，新增include extra/*.conf;配置项。



# 启动脚本

nginx服务启动脚本，注意目录

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



