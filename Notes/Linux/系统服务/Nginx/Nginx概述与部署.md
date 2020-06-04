[TOC]

# 前言

## nginx的特点

本节主要对Nginx  Web服务软件进行介绍，涉及Nginx的基础，特性，配置部署，优化，以及企业中的日常运维管理和应用。作为HTTP服务软件的后起之秀，Nginx与它的老大哥Apache相比有很多改进之处，比如，在性能上，Nginx占用的系统资源更少，能支持更多的并发连接（特别是静态小文件场景下），达到更高的访问效率；在功能上，Nginx不但是一个优秀的Web服务软件，还可以作为反向代理负载均衡及缓存服务使用；在安装配置上，Nginx更为方便，简单，灵活，可以说，Nginx是一个极具发展潜力的Web服务软件。

## Nginx是什么？

nginx是一个开源的，支持高性能，高并发的www服务和代理服务软件。

nginx因具有高并发（特别是静态资源），占用系统资源少等特性，且功能丰富而逐渐流行起来。

nginx不但是一个优秀Web服务软件，还具有反向代理负载均衡功能和缓存服务功能，与lvs负载均衡及Haproxy等专业代理软件相比，Nginx部署起来更为简单，方便；在缓存功能方面，它又类似于Squid等专业的缓存服务软件。

## Nginx的重要特性

支持高并发：能支持几万并发连接（特别是静态小文件业务环境）

资源消耗少：在3万并发连接下，开启10哥Nginx线程消耗的内存不到200MB

可以做HTTP反向代理及加速缓存，即负载均衡功能，内置对RS节点服务器健康检查功能，这相当于专业的Haproxy软件或LVS的功能

具备Squid等专业缓存软件等的缓存功能。

支持异步网络I／O事件模型epoll（linux2.6+）。

## 企业功能应用

（1）作为Web服务软件

Nginx是一个支持高性能，高并发的Web服务软件，它具有很多优秀的特性，作为Web服务器，与Apache相比，Nginx能够支持更多的并发连接访问，但占用的资源更少，效率更高，在功能上也强大了很多，几乎不逊色于Apache。

（2）反向代理或负载均衡服务

在反向代理或负载均衡服务方面，Nginx可以作为Web服务，PHP等动态服务及Memcached缓存的代理服务器，它具有类似专业反向代理软件（如Haproxy）的功能，同时也是一个优秀的邮件代理服务软件，但是Nginx的代理功能还是相对简单了些，特别是不支持TCP的代理（Nginx1.9.0版本已经开始支持TCP代理了）

（3）前端业务数据缓存服务

在Web缓存服务方面，Nginx可通过自身的proxy_cache模块实现类Squid等专业缓存软件的功能。

综上：Nginx的这三大功能（Web服务，反向代理或负载均衡服务，前端业务数据缓存服务）是国内使用Nginx的主要场景，特别是前两个。

## Web 服务产品性能对比测试

从下图中可以看出处理静态小文件（小于1MB时），Nginx和Lighttpd比Apache更有优势，Nginx处理小文件的优势明显，Lighttpd综合最强。

![屏幕快照 2017-07-09 下午10.24.42.png-784.8kB](http://static.zybuluo.com/chensiqi/nuj5bn01ux00e1fmpzzk3edz/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-09%20%E4%B8%8B%E5%8D%8810.24.42.png)

下图是各类Web服务器在动态数据性能上的对比，从图中可以看出，在处理动态数据时，三者的差距不大，Apache更有优势一点。这是因为处理动态数据的能力取决于PHP（java）和后端数据库的服务能力，也就是说瓶颈不在Web服务器上。一般情况下普通PHP引擎支持的并发连接参考值为300～1000，Java引擎和数据库的并发连接参考值为300～1500.业务场景及网站架构不同，并发连接数也会有上下浮动。

![屏幕快照 2017-07-09 下午10.27.54.png-764.2kB](http://static.zybuluo.com/chensiqi/w5ei82vw8njmygdxxszqn1tb/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-09%20%E4%B8%8B%E5%8D%8810.27.54.png)

## Nginx和Apache性能对比

Nginx使用最新的epoll（Linux2.6内核）和kqueue（freebsd）异步网络I／O模型，而Apache使用的是传统的select模型。目前Linux下能够承受高并发访问的Squid，Memcached软件采用的都是epoll模型。

处理大量连接的读写时，Apache所采用的select网络I／O模型比较低效。

![屏幕快照 2017-07-09 下午10.40.24.png-826.6kB](http://static.zybuluo.com/chensiqi/1ysmca33goxznwdj76j90g1t/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-09%20%E4%B8%8B%E5%8D%8810.40.24.png)

![QQ20170709-223902@2x.png-1292.3kB](http://static.zybuluo.com/chensiqi/1kqwqodui81yizbn96yoh7cp/QQ20170709-223902@2x.png)

## 如何正确选择Web服务器

虽然国内很多人都在使用Nginx，但是Apache，Lighttpd这两个Web服务器同样非常强大且实用，尤其时Apache，到目前为止仍是全球使用最广泛的Web服务软件。
 在实际工作，我们需要根据业务需求来选择合适的业务服务软件，有关Web服务，建议如下。

- 静态业务：若是高并发场景，尽量采用Nginx或Lighttpd，二者首选Nginx。
- 动态业务：理论上采用Nginx和Apache均可，建议选择Nginx，为了避免相同业务的服务软件多样化，增加额外维护成本。动态业务可以由Nginx兼做前端代理，再根据页面元素的类型或目录，转发到后端相应的服务器进行处理。
- 既有静态业务又有动态业务：采用Nginx

此外，如果并发不是很大，又对Apache很熟悉，采用Apache也是可以的，Apache2.4版本也很强大，并发连接数也有所增加。总的来说，在满足需求的前提下，首先选择自己最擅长的软件，若发现了更好的软件，可在掌握新软件之后逐步替换。虽然动态和静态业务都倾向于选择Nginx，但是大前提是自己要熟练掌握Nginx。切记，在工作中不要盲目选择软件，这可能最终会导致自己无法控制局面，从而给企业带来灾难性的损失。

# 一，nginx的编译安装

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
[root@linux-study nginx-1.12.2]# chown -R nginx.nginx nginx
[root@linux-study nginx-1.12.2]# service nginx start
```

**特别提示：**
`/usr/local/nginx/sbin/nginx -s reload` nginx平滑重启命令
`/usr/local/nginx/sbin/nginx -s stop` nginx停止服务命令

## 1.1 web排错三部曲下面介绍客户端排查的思路

**第一步，在客户端上ping服务器端IP，命令如下：**

`ping 10.0.0.8`排除物理线路问题影响

**第二步，在客户端上telnet服务器端IP，端口，命令如下：**

`telnet 10.0.0.8 80`排除防火墙等得影响

**第三步，在客户端使用wget命令检测，如下：**

`wget 10.0.0.8(curl -I 10.0.0.8)`模拟用户访问，排除http服务自身问题，根据输出在排错

提示：
以上三步是客户端访问网站异常排查的重要三部曲。

## 1.2，Nginx主配置文件nginx.conf

Nginx主配置文件nginx.conf是一个纯文本类型的文件（其他配置文件大多也是如此），它位于Nginx安装目录下的conf目录，整个配置文件是以区块的形式组织的。一般，每个区块以一个大括号“{}”来表示，区块可以分为几个层次，整个配置文件中Main区位于最上层，在Main区下面可以有Events区，HTTP区等层级，在HTTP区中又包含有一个或多个Server区，每个Server区中又可有一个或多个location区，整个Nginx配置文件nginx.conf的主体框架为：

```shell
[root@chensiqi conf]# egrep -v "#|^$" nginx.conf # 去掉包含#号和空行的内容
worker_processes  1; # worker进程的数量
error_log  logs/error.log;  # 错误日志（默认没开）
pid        logs/nginx.pid;  # 进程号（默认没开）
events {    # 事件区块开始
    worker_connections  1024;   #每个worker进程支持的最大连接数
}           # 事件区块结束
http {      # http区块开始
    include       mime.types;   #Nginx支持的媒体类型库文件包含
    default_type  application/octet-stream; #默认的媒体类型
    sendfile        on;     # 开启高效传输模式
    keepalive_timeout  65;  # 连接超时。
    server {      # 网站配置区域（第一个server第一个虚拟主机站点）
        listen       80;    # 提供服务的端口，默认80
        server_name  www.chensiqi.org; # 提供服务的域名主机名
        location / {    # 第一个Location区块开始
            root   html;  # 站点的根目录（相对于nginx安装路径）
            index  index.html index.htm; # 默认的首页文件，多个用空格分开
        }
        error_page 500 502 503 504  /50x.html;  # 出现对应的http状态码时，使用50x.html回应客户
        location = /50x.html {  # Location区块开始，访问50x.html
            root   html;     # 指定对应的站点目录为html
        }
    }
    server {      # 网站配置区域（第二个server第二个虚拟主机站点）
        listen       80;    # 提供服务的端口，默认80
        server_name  bbs.chensiqi.org; # 提供服务的域名主机名
        location / {    # 服务区块
            root   html;  # 相对路径（nginx安装路径）
            index  index.html index.htm;
        }
        location = /50x.html { # 发生错误访问的页面
            root   html;
        }
    }
}
```

整个nginx配置文件的核心框架如下：

```
worker_processes 1;
events {
    
    worker_connections 1024;

}
http {
    include mime.types;
    server {
        listen  80;
        server_name localhost;
        location / {
            root  html;
            index  index.html index.htm;
        }
    }
}
```

## 1.3，Nginx其他配置文件

如果是配合动态服务（例如PHP服务），Nginx软件还会用到扩展的fastcgi相关配置文件，这个配置是通过在Nginx.conf主配置文件中嵌入include命令来实现的，不过默认情况是注释状态，不会生效。

fastcgi.conf配置文件的初始内容如下：

```
[root@localhost conf]# cat fastcgi.conf

fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
```

fastcgi_params 默认配置文件的内容如下：

```
[root@localhost conf]# cat fastcgi_params

fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
```

上述未做注释的目录或文件是比较少用的，有关动态扩展配置后文讲到PHP服务时再来讲解。

## 1.4，Nginx的功能模块说明

![屏幕快照 2017-07-10 上午7.22.21.png-1799.1kB](http://static.zybuluo.com/chensiqi/pmpdqcf487ys6z56qksmli9w/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-10%20%E4%B8%8A%E5%8D%887.22.21.png)

# 二，Nginx虚拟主机配置

## 2.1 虚拟主机概念和类型介绍

虚拟主机概念

所谓虚拟主机，在Web服务里就是一个独立的网站站点（www.baidu.org），这个站点对应独立的域名（也可能是IP或端口），具有独立的程序及资源目录，可以独立地对外提供服务供用户访问。
这个独立的站点在配置里是由一定格式的标签段标记，对于Apache软件来说，一个虚拟主机的标签段通常被包含在内，而Nginx软件则使用一个server{}标签来标示一个虚拟主机，一个Web服务里可以有多个虚拟主机标签对，即同时可以支持多个虚拟主机站点。

虚拟主机类型

常见的虚拟主机类型有如下几种。

- 基于域名的虚拟主机

  所谓基于域名的虚拟主机，意思就是通过不同的域名区分不同的虚拟主机，基于域名的虚拟主机是企业应用最广的虚拟主机类型，几乎所有对外提供服务的网站都是使用基于域名的虚拟主机，例如：www.etiantian.org

- 基于端口的虚拟主机

  同理，所谓基于端口的虚拟主机，意思就是通过不同的端口来区分不同的虚拟主机，此类虚拟主机对应的企业应用主要为公司内部的网站，例如：一些不希望直接对外提供用户访问的网站后台等，访问基于端口的虚拟主机地址里要带有端口，例如：http://www.baidu.com:80

- 基于IP的虚拟主机

  同理，所谓基于IP的虚拟主机，意思就是通过不同的IP区分不同的虚拟主机，此类虚拟主机对应的企业应用非常少见，一般不同业务需要使用多IP的场景都会在负载均衡器上进行VIP绑定，而不是在Web上通过绑定IP区分不同的虚拟机。
  三种虚拟主机类型均可独立使用，也可以互相混合一起使用，同学们应把**基于域名**的虚拟主机类型**当作重点**来学习掌握，其他的两个类型了解即可。

## 2.2 基于域名的虚拟主机配置实战

**说明：本节内容再生产场景中是最常用到的，因此，同学们要优先并且熟练掌握。**

配置基于域名的nginx.conf内容

**这里使用grep过滤命令来生成基础的Nginx主配置文件nginx.conf，然后根据生成的初始配置进行修改，使其成为所需的形式，具体步骤为：**

```
[root@localhost conf]# pwd
/application/nginx/conf
[root@localhost conf]# egrep -v "#|^$" nginx.conf.default >nginx.conf
```

**或者干脆直接新创建配置文件nginx.conf,然后编辑，输入如下内容：**

```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html/www;
            index  index.html index.htm;
        }
    }
}
```

**编辑完配置文件后，我们需要检查语法**

```
[root@localhost conf]# /application/nginx/sbin/nginx -t
nginx: the configuration file /application/nginx-1.10.2//conf/nginx.conf syntax is ok
nginx: configuration file /application/nginx-1.10.2//conf/nginx.conf test is successful
```

**然后由于web的存放路径是相对路径，因此我们需要创建个目录，**

![QQ20170621-200705@2x.png-135.5kB](Nginx%E5%9F%BA%E7%A1%80.assets/QQ20170621-200705@2x.png)

```
[root@localhost html]# cd /application/nginx/html/
[root@localhost html]# ls
50x.html  index.html            #原本的网页文件
[root@localhost html]# mkdir www   #创建一个目录叫做www
[root@localhost html]# echo "I am www" > www/index.html  #写入网页文件
[root@localhost html]# cat www/index.html   #查看一下
I am www
[root@localhost html]# curl 192.168.0.100  #测试链接
I am www
```

**接下来，我们再创建一个域名的网站,配置文件如下**

![QQ20170621-201305@2x.png-149.2kB](Nginx%E5%9F%BA%E7%A1%80.assets/QQ20170621-201305@2x.png)
 注意：修改配置文件需要重启动nginx

**给第二个网站添加网页文件**

```
[root@localhost html]# ls
50x.html  index.html  www
[root@localhost html]# mkdir bbs
[root@localhost html]# echo "I am bbs" > bbs/index.html
```

**通过测试，我们发现，永远都只能看到第一个网站**

```
[root@localhost html]# curl 192.168.0.100
I am www
[root@localhost html]# curl 192.168.0.100
I am www
[root@localhost html]# curl 192.168.0.100
I am www
[root@localhost html]# curl 192.168.0.100
I am www
```

> 这是因为通过IP地址来访问的话，nginx并不知道你想要访问哪个站点，因此，他默认你是要访问他配置文件里的第一个站点，也就是www.chensiqi.com
>  通过修改hosts映射我们可以访问不同的站点。

**修改hosts映射文件**

```
[root@localhost html]# echo "192.168.0.100 www.chensiqi.com bbs.chensiqi.com" >> /etc/hosts
[root@localhost html]# tail -1 /etc/hosts
192.168.0.100 www.chensiqi.com bbs.chensiqi.com
```

**现在我们再进行访问测试**

```
[root@localhost html]# curl www.chensiqi.com
I am www
[root@localhost html]# curl www.chensiqi.com
I am www
[root@localhost html]# curl bbs.chensiqi.com
I am bbs
[root@localhost html]# curl bbs.chensiqi.com
I am bbs
```

> **如上所示**：基于域名的虚拟主机配置完毕。我们在工作中遇到的基本都是这种类型的网站。配置过程需要重点练习。其他类型，了解即可。

# 三，Nginx常用功能配置实战

## 3.1 规范化Nginx配置文件

下面是优化后的Nginx配置的实战方案

```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    include extra/www.conf;   #虚拟网站配置信息统一放在了当前的extra目录下
    include extra/mail.conf;
    include extra/status.conf;

}
[root@localhost nginx]# tree conf/extra/
conf/extra/
├── mail.conf
├── status.conf
└── www.conf

0 directories, 3 files
[root@localhost nginx]# cat conf/extra/www.conf 
    server {
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
            root   /var/www/html/wwwcom;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /var/www/html;
        }
    }
```

## 3.2 Nginx状态信息功能实战

### 3.2.1 确认编译时是否设定了此模块

```
root@localhost nginx]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.10.2
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-4) (GCC) 
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/usr/local/nginx-1.10.2/ --with-http_stub_status_module --with-http_ssl_module

##说明
--with-http_stub_status_module模块就是状态信息模块
```

### 3.2.2 设定信息模块配置

```
[root@localhost nginx]# cat conf/extra/status.conf 
##status


server{

    listen  80;
    server_name  status.yunjisuan.com;
    location / {
       stub_status  on;   #开启状态信息功能
           access_log   off;  #不记录访问日志

         }
}  

##说明
状态信息模块配置方式和搭建虚拟网站类似需要占用一个域名来访问
```

**注意：需要重启Nginx服务**

### 3.2.3 nginx-status

```
"""
server {
		location = /nginx-status {
                stub_status on;
                access_log off;
                allow 127.0.0.1;
                allow 172.22.0.0/16;
                deny all;
        }
}
"""

[root@localhost nginx]# curl status.yunjisuan.com
Active connections: 2 #表示Nginx正在处理的活动连接2个
server accepts handled requests
 39 39 41 
Reading: 0 Writing: 1 Waiting: 1 
```

状态页面各项数据的意义：

active connections – 当前 Nginx 正处理的活动连接数。

serveraccepts handled requests — 总共处理了 233851 个连接 , 成功创建 233851 次握手 (证明中间没有失败的 ), 总共处理了 687942 个请求 ( 平均每次握手处理了 2.94 个数据请求 )。

reading — nginx 读取到客户端的 Header 信息数。

writing — nginx 返回给客户端的 Header 信息数。

waiting — 开启 keep-alive 的情况下，这个值等于 active – (reading + writing)， 意思就是 Nginx 已经处理完正在等候下一次请求指令的驻留连接。

/usr/lib/zabbix/externalscripts

/etc/zabbix/externalscripts



**特别提示：**

出于安全起见，这个状态信息要防止外部用户查看。

### 3.2.4 增加错误日志

```
范例：error_log  file  level;
```

> 常见的日志级别【debug|info|notice|warn|error|crit|alert|emerg】
>  生产场景一般是warn|error|crit这三个级别之一，注意不要配置info等较低级别，会带来巨大磁盘I／O消耗。
>  error_log的默认值为：
>  \# default：error_log logs/error.log error;

```
worker_processes  1;
error_log  logs/error.log;    #非常简单，一般增加此行即可
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    include extra/www.conf;
    include extra/mail.conf;
    include extra/status.conf;

}
```

### 3.2.5 Nginx访问日志轮询切割

> 默认情况下Nginx会把所有的访问日志生成到一个指定的访问日志文件access.log里，但这样一来，时间长了就会导致日志个头很大，不利于日志的分析和处理，因此，有必要对Nginx日志，按天或按小时进行切割，使其分成不同的文件保存。

```
[root@localhost nginx]# cat /server/scripts/cut_nginx_log.sh 
#!/bin/bash
#日志切割脚本可挂定时任务，每天00点整执行

Dateformat=`date +%Y%m%d`
Basedir="/usr/local/nginx"
Nginxlogdir="$Basedir/logs"
Logname="access"

[ -d $Nginxlogdir ] && cd $Nginxlogdir || exit 1
[ -f ${Logname}.log ] || exit 1
/bin/mv ${Logname}.log ${Dateformat}_${Logname}.log
$Basedir/sbin/nginx -s reload

[root@localhost nginx]# cat >>/var/spool/cron/root << KOF
#cut nginx access log by Mr.chen
00 00 * * * /bin/bash /server/scripts/cut_nginx_log.sh >/dev/null 2>&1
```

## 3.3 Nginx location

### 3.3.1 location使用的语法为：

```
location [ = | ~ | ~* | ^~ ] uri {

  ...

}
```

> 上图是对location语法的说明。上述语法中的URI部分是关键，这个URI可以是普通的字符串地址路径，或者是正则表达式，匹配成功则执行后面大括号里的相关命令。正则表达式的前面还可以有“～”或“～*”等特殊字符。

![屏幕快照 2017-07-10 下午9.40.57.png-201.6kB](http://static.zybuluo.com/chensiqi/4t6bpgih9slir4wjr80qsiua/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-10%20%E4%B8%8B%E5%8D%889.40.57.png)

匹配这两种特殊字符“～”或“～*”的区别为：“～”用于区分大小写（大小写敏感）的匹配；“～*”用于不区分大小写的匹配。还可以用逻辑操作符“！”对上面的匹配取反，即“！～”和“！～*”。此外，“^~”的作用是先进行字符串的前缀匹配（必须以后边的字符串开头），如果能匹配到，就不再进行其他location的正则匹配了。

### 3.3.2 location匹配示例

```
[root@localhost nginx]# cat /usr/local/nginx/conf/extra/www.conf 
    server {
        listen       80;
        server_name  www.yunjisuan.com;
    root    /var/www/html/wwwcom;
        location / {
        return 401;     
        }
    location = / {
        return 402;
    }
    location = /images/ {
        return 501;
    }
    location /documents/ {
        return 403;
    }
    location ^~ /images/ {
        return 404;
    }
    location ~* \.(gif|jpg|jpeg)$ {
        return 500;
    }
    
    }
```

**正则匹配结果如下：**

```
[root@localhost nginx]# curl -s -o /dev/null -w "%{http_code}\n" www.yunjisuan.com
402        #匹配了=的情况
[root@localhost nginx]# curl -s -o /dev/null -w "%{http_code}\n" www.yunjisuan.com/
402         #匹配了=的情况
[root@localhost nginx]# curl -s -o /dev/null -w "%{http_code}\n" www.yunjisuan.com/xxxx
401         #匹配不到默认匹配 ／的情况
[root@localhost nginx]# curl -s -o /dev/null -w "%{http_code}\n" www.yunjisuan.com/documents/
403         #匹配字符串
[root@localhost nginx]# curl -s -o /dev/null -w "%{http_code}\n" www.yunjisuan.com/images/
501         #优先匹配=的情况
[root@localhost nginx]# curl -s -o /dev/null -w "%{http_code}\n" www.yunjisuan.com/images/1.jpg
404         #匹配
[root@localhost nginx]# curl -s -o /dev/null -w "%{http_code}\n" www.yunjisuan.com/documents/images/1.jpg
500         #匹配～*的情况
```

**从多个location的配置匹配可以看出匹配的优先顺序**

| 顺序 | 匹配标识的location                       | 匹配说明                                             |
| ---- | ---------------------------------------- | ---------------------------------------------------- |
| 1    | " location = / { "                       | 精确匹配                                             |
| 2    | " location ^~ /images/ { "               | 先进行字符串的前缀匹配，如果匹配到就不做正则匹配检查 |
| 3    | " loction ~* \.(gif \| jpg \| jpeg)$ { " | 正则匹配，*为不区分大小写                            |
| 4    | " location /documents/ { "               | 匹配常规字符串，模糊匹配，如果有正则检查，正则优先   |
| 5    | " location / { "                         | 所有location都不能匹配后的默认匹配原则               |

## 3.4 Nginx rewrite

### 3.4.1 什么是Nginx rewrite？

和Apache等Web服务软件一样，Nginx  rewrite的主要功能也是实现URL地址重写。Nginx的rewrite规则需要PCRE软件的支持，即通过Perl兼容正则表达式语法进行规则匹配。默认参数编译时，Nginx就会安装支持rewrite的模块，但是，也必须要有PCRE软件的支持。

### 3.4.2 Nginx rewrite 语法

（1）rewrite指令语法

指令语法：rewrite regex replacement 【flag】；
 默认值：none
 应用位置：server，location，if
 rewrite是实现URL重写的关键指令，根据regex（正则表达式）部分的内容，重定向到replacement部分，结尾是flag标记。下面是一个简单的URL rewrite跳转例子：

```
rewrite ^/(.*) http://www.baidu.com/$1  permanent;
```

在上述指令中，rewrite为固定关键字，表示开启一条rewrite匹配规则，regex部分是^(.*),这是一个正则表达式，表示匹配所有，匹配成功后跳转到http://www.baidu.com/$1  。这里的$1是取前面regex部分括号里的内容，结尾的permanent；是永久301重定向标记，即跳转到后面的http://www.baidu.com/$1 地址上。

（2）regex常用正则表达式说明

![屏幕快照 2017-07-12 上午9.12.09.png-1130.9kB](http://static.zybuluo.com/chensiqi/0crya4u7ubyja2yy9e6knvmh/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-12%20%E4%B8%8A%E5%8D%889.12.09.png)

（3）rewrite指令的最后一项参数flag标记的说明

![屏幕快照 2017-07-12 上午9.14.31.png-300.7kB](http://static.zybuluo.com/chensiqi/ub07sy81wrw8d9j56fwqm7ag/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-12%20%E4%B8%8A%E5%8D%889.14.31.png)

在以上的flag标记中，last和break用来实现URL重写，浏览器地址栏的URL地址不变，但在服务器端访问的程序及路径发生了变化。redirect和permanent用来实现URL跳转，浏览器地址栏会显示跳转后的URL地址。

last和break标记的实现功能类似，但二者之间有细微的差别，使用alias指令时必须用last标记，使用proxy_pass指令时要使用break标记。last标记在本条rewrite规则执行完毕后，会对其所在的server{...}标签重新发起请求，而break标记则会在本条规则匹配完成后，终止匹配，不再匹配后面的规则。

### 3.4.3 Nginx rewrite 的企业应用场景

**Nginx的rewrite功能在企业里应用非常广泛：**

- 可以调整用户浏览的URL，使其看起来更规范，合乎开发及产品人员的需求。
- 为了让搜索引擎收录网站内容，并让用户体验更好,企业会将动态URL地址伪装成静态地址提供服务
- 网站换新域名后，让旧域名的访问跳转到新的域名上，例如：让京东的360buy换成了jd.com
- 根据特殊变量，目录，客户端的信息进行URL跳转等。

### 3.4.4 Nginx rewrite 301 跳转

以往我们是通过别名方式实现yunjisuan.com和www.yunjisuan.com访问同一个地址的，事实上，除了这个方式外，还可以使用nginx rewrite 301 跳转的方式来实现。实现的配置如下：

```
[root@localhost nginx]# cat conf/extra/www.conf 
#www virtualhost by Mr.chen   
    server {
        listen       80;
        server_name  www.yunjisuan.com;
    root    /var/www/html/wwwcom;
        location / {
        index index.html index.htm;
        }
#   location = / {
#       return 402;
#   }
    location = /images/ {
        return 501;
    }
    location /documents/ {
        return 403;
    }
    location ^~ /images/ {
        return 404;
    }
    location ~* \.(gif|jpg|jpeg)$ {
        return 500;
    }
    
    }

    server{
        listen  80;
        server_name yunjisuan.com;
        rewrite ^/(.*)  http://www.yunjisuan.com/$1 permanent;
        #当用户访问yunjisuan.com及下面的任意内容时，都会通过这条rewrite跳转到www.yunjisuan.com对应的地址
    }
```

**客户端访问测试结果**

![QQ20170712-105655@2x.png-43.5kB](Nginx%E5%9F%BA%E7%A1%80.assets/QQ20170712-105655@2x.png)

![QQ20170712-105719@2x.png-83kB](Nginx%E5%9F%BA%E7%A1%80.assets/QQ20170712-105719@2x.png)

### 3.4.5 实现不同域名的URL跳转

示例：实现访问http://mail.yunjisuan.com时跳转到http://www.yunjisuan.com/mail/yunjisuan.html

外部跳转时，使用这种方法可以让浏览器地址变为跳转后的地址，另外，要事先设置http://www.yunjisuan.com/mail/yunjisuan.html有结果输出，不然会出现401等权限错误。

（1）配置Nginx rewrite规则

```
[root@localhost nginx]# cat conf/extra/mail.conf 
    server {
        listen       80;
        server_name  mail.yunjisuan.com;
        location / {
            root   /var/www/html/mailcom;
            index  index.html index.htm;
        }
    if ( $http_host ~* "^(.*)\.yunjisuan\.com$") {
        set $domain $1;
        rewrite ^(.*) http://www.yunjisuan.com/$domain/yunjisuan.html break;

    }
}
```

**客户端访问测试**

![QQ20170712-114726@2x.png-67.3kB](Nginx%E5%9F%BA%E7%A1%80.assets/QQ20170712-114726@2x.png)

![QQ20170712-114744@2x.png-52.6kB](Nginx%E5%9F%BA%E7%A1%80.assets/QQ20170712-114744@2x.png)

### 3.4.6 rewrite跳转标记flag使用总结

1，在根location（即location ／ {...}）中或server{...} 标签中编写rewrite规则，建议使用last标记
2，在普通的location（例 location/yunjisuan/{...}或if{}中编写rewrite规则，则建议使用break标记）

## 3.5 Nginx访问认证

有时，在实际工作中企业要求我们为网站设置访问账号和密码权限，这样操作后，只有拥有账号密码的用户才可以访问网站内容。
这种使用账号密码才可以访问网站的功能主要应用在企业内部人员访问的地址上，例如：企业网站后台，MySQL客户端phpmyadmin，企业内部的CRM，WIKI网站平台。

### 3.5.1 创建密码文件

**我们可以借用apache的htpasswd软件，来创建加密的账号和密码**

```
[root@localhost ~]# which htpasswd
[root@localhost ~]# htpasswd -bc /usr/local/nginx/conf/htpasswd yunjisuan 123123
Adding password for user yunjisuan   
[root@localhost ~]# cat /usr/local/nginx/conf/htpasswd 
yunjisuan:FC1/eEc/iK0Mo   #账号密码是加密的（htpasswd是文件的名字）
```

### 3.5.2 在虚拟主机配置文件里加入两条配置信息

```
[root@localhost html]# cat /usr/local/nginx/conf/extra/blog.conf 
    server {
        listen       80;
        server_name  blog.yunjisuan.com;
        location / {
            root   /var/www/html/blogcom;
            index  index.html index.htm;
            auth_basic      "yunjisuan training";   #加入这条配置
            auth_basic_user_file    /usr/local/nginx/conf/htpasswd; #加入这条配置
        }
    }


#配置解释：

auth_basic :验证的基本信息选项（后边跟着的双引号里就是验证窗口的名字）
auth_basic_user_file ：验证的用户文件（后边根账号密码文件的绝对路径）
```

### 3.5.3 网页登陆验证

![QQ20170712-203741@2x.png-46.7kB](Nginx%E5%9F%BA%E7%A1%80.assets/QQ20170712-203741@2x.png)

![屏幕快照 2017-07-12 下午8.36.57.png-57.4kB](http://static.zybuluo.com/chensiqi/9kba0klmsjdw59jeqht7ugn8/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-12%20%E4%B8%8B%E5%8D%888.36.57.png)

![QQ20170712-203807@2x.png-45.3kB](Nginx%E5%9F%BA%E7%A1%80.assets/QQ20170712-203807@2x.png)

# 四，Nginx相关问题解答

## 4.1 Tengine和Nginx是什么关系？

Tengine是淘宝开源Nginx的分支，官方站点为http://tengine.taobao.org/

## 4.2 访问Nginx时出现状态码“403 forbidden”的原因

（1）Nginx配置文件里没有配置默认首页参数，或者首页文件在站点目录下没有如下内容：

```
index  index.php   index.html  index.htm;
```

（2）站点目录或内部的程序文件没有Nginx用户访问权限

（3）Nginx配置文件中设置了allow，deny等权限控制，导致客户端没有访问权限。



# 附

## Nginx服务启动脚本

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

