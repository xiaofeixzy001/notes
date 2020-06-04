[TOC]

# 需求

收集nginx服务器的访问日志以及错误日志进行实时统计，在kibana页面进行搜索展现。

nginx服务器要安装logstash或filebeat负责收集日志，然后将日志转发给elasticsearch进行分析，在通过kibana在前端展现。

# 安装nginx

系统：CentOS Linux release 7.7.1908 (Core)

hostname：nginx-node01

IP：192.168.100.61

nginx：nginx-1.16.1.tar.gz

path : /data/apps/nginx

下载地址：http://nginx.org/en/download.html

```shell
[root@nginx-node01 ~]# yum install gcc gcc-c++ automake pcre pcre-devel zlip zlib-devel openssl openssl-devel -y
[root@nginx-node01 ~]# groupadd -r nginx
[root@nginx-node01 ~]# useradd -r -g nginx -s /bin/false -M nginx
[root@nginx-node01 ~]# mkdir -pv /data/apps/nginx/{logs,run,client,proxy,fcgi,uwsgi,scgi}
[root@nginx-node01 ~]# wget http://nginx.org/download/nginx-1.16.1.tar.gz
[root@nginx-node01 ~]# tar xvf nginx-1.10.3.tar.gz
[root@nginx-node01 nginx-1.16.1]# cd nginx-1.10.3
[root@nginx-node01 nginx-1.16.1]# ./configure \
--prefix=/data/apps/nginx \
--conf-path=/data/apps/nginx/conf/nginx.conf \
--sbin-path=/data/apps/nginx/sbin/nginx \
--error-log-path=/data/apps/nginx/logs/error.log \
--http-log-path=/data/apps/nginx/logs/access.log \
--pid-path=/data/apps/nginx/run/nginx.pid  \
--lock-path=/data/apps/nginx/run/nginx.lock \
--user=nginx \
--group=nginx \
--with-http_ssl_module \
--with-http_flv_module \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--http-client-body-temp-path=/data/apps/nginx/client/ \
--http-proxy-temp-path=/data/apps/nginx/proxy/ \
--http-fastcgi-temp-path=/data/apps/nginx/fcgi/ \
--http-uwsgi-temp-path=/data/apps/nginx/uwsgi \
--http-scgi-temp-path=/data/apps/nginx/scgi \
--with-pcre

[root@nginx-node01 nginx-1.16.1]# make && make install
[root@nginx-node01 nginx-1.16.1]# vim /etc/profile.d/nginx.sh
'''
export PATH=$PATH:/usr/local/nginx/sbin
'''

# 添加脚本
[root@nginx-node01 nginx-1.16.1]# cat /usr/lib/systemd/system/nginx.service
"""
[Unit]
Description=nginx
After=network.target remote-fs.target nss-lookup.target
 
[Service]
Type=forking
PIDFile=/data/apps/nginx/run/nginx.pid
ExecStartPre=/usr/bin/rm -f /data/apps/nginx/run/nginx.pid
ExecStartPre=/data/apps/nginx/sbin/nginx -t
ExecStart=/data/apps/nginx/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
 
[Install]
WantedBy=multi-user.target
"""

[root@nginx-node01 nginx-1.16.1]# systemctl daemon-reload
[root@nginx-node01 nginx-1.16.1]# systemctl start nginx.service
[root@nginx-node01 nginx-1.16.1]# systemctl enable nginx.service
[root@nginx-node01 nginx-1.16.1]# systemctl status nginx.service
[root@nginx-node01 nginx-1.16.1]# ps aux | grep nginx
```

测试访问web页面

# nginx日志转json

```shell
[root@nginx-node01 ~]# cd /data/apps/nginx/
[root@nginx-node01 nginx]# vim conf/nginx.conf
'''
#user  nobody;
worker_processes  auto;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;
    
    log_format main '{"@timestamp":"$time_iso8601",'
        '"host":"$server_addr",'
        '"clientip":"$remote_addr",'
        '"size":$body_bytes_sent,'
        '"responsetime":$request_time,'
        '"upstreamtime":"$upstream_response_time",'
        '"upstreamhost":"$upstream_addr",'
        '"http_host":"$host",'
        '"url":"$uri",'
        '"domain":"$host",'
        '"xff":"$http_x_forwarded_for",'
        '"referer":"$http_referer",'
        '"status":"$status"}';

    access_log /data/apps/nginx/logs/access.log access_json;

    sendfile        on;
    keepalive_timeout  65;
	gzip  on;

    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }

        error_page  404              /404.html;
        error_page   500 502 503 504  /50x.html;
        
        location = /50x.html {
            root   html;
        }
    }
}
'''

[root@nginx-node01 nginx]# nginx -t
[root@nginx-node01 nginx]# nginx -s reload
```

确认日志格式为json：

```shell
[root@nginx-node01 nginx]# tail -100 /var/log/nginx/access.log 
'''
{"@timestamp":"2017-04-21T17:03:09+08:00","host":"192.168.100.61","clientip":"192.168.100.1","size":0,"responsetime":0.000,"upstreamtime":"-","upstreamhost":"-","http_host":"192.168.100.61","url":"/web/index.html","domain":"192.168.100.61","xff":"-","referer":"-","status":"304"}
'''
```



# 收集nginx日志

## 使用logstash收集

在nginx服务器安装logstash用于收集nginx的访问日志和错误日志.

配置logstash收集nginx访问日志：

```shell
[root@nginx-node01 ~]# yum install logstash-5.3.0.rpm  -y
[root@nginx-node01 ~]# cat /etc/logstash/conf.d/nginx.conf 
'''
input {
    file {
        path => "/data/apps/nginx/logs/access.log"
        start_position => "end"
        type => "nginx-access-log"
        codec => json
    }
}

filter{
}

output {
    if [type] == "nginx-access-log" {
        elasticsearch {
            hosts => ["192.168.100.31","192.168.100.32"]
            index => "nginx-access-log-%{+YYYY.MM.dd}"
        }
    }
}
'''


```

kibana界面添加索引[nginx-access-log-]YY-MM-DD

kibana界面验证数据

## 使用filebeat收集