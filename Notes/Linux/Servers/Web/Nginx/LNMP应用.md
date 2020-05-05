[TOC]

## 一，LNMP应用环境

### 1.1 LNMP介绍

> 大约在2010年以前，互联网公司最常用的经典Web服务环境组合就是LAMP（即Linux，Apache，MySQL，PHP），近几年随着Nginx  Web服务的逐渐流行，又出现了新的Web服务环境组合--LNMP或LEMP，其中LNMP为Linux，Nginx，MySQL，PHP等首字母的缩写，而LEMP中的E则表示Nginx，它取自Nginx名字的发音（engine x）。现在，LNMP已经逐渐成为国内大中型互联网公司网站的主流组合环境，因此，我们必须熟练掌握LNMP环境的搭建，优化及维护方法。

### 1.2 LNMP组合工作流程

> 在深入学习LNMP组合之前，有必要先来了解以下LNMP环境组合的基本原理，也就是它们之间到底是怎样互相调度的？
>  在LNMP组合工作时，首先是用户通过浏览器输入域名请求Nginx  Web服务，如果请求是静态资源，则由Nginx解析返回给用户；如果是动态请求（.php结尾），那么Nginx就会把它通过FastCGI接口（生产常用方法）发送给PHP引擎服务（FastCGI进程php-fpm）进行解析，如果这个动态请求要读取数据库数据，那么PHP就会继续向后请求MySQL数据库，以读取需要的数据，并最终通过Nginx服务把获取的数据返回给用户，这就是LNMP环境的基本请求顺序流程。这个请求流程是企业使用LNMP环境的常用流程。

![屏幕快照 2017-07-12 下午10.13.49.png-344.1kB](http://static.zybuluo.com/chensiqi/mtc14cjofs08uc55tlrlj3mf/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-12%20%E4%B8%8B%E5%8D%8810.13.49.png)

## 二，LNMP之MySQL数据库

### 2.1 MySQL数据库介绍

> MySQL是互联网领域里非常重要的，深受广大用户欢迎的一款开源关系型数据库软件，由瑞典MySQL  AB公司开发与维护。2006年，MySQL  AB公司被SUN公司收购，2008年，SUN公司又被传统数据数据库领域大佬甲骨文（Oracle）公司收购。因此，MySQL数据库软件目前属于Oracle公司，但扔是开源的，Oracle公司收购MySQL的战略意图显而易见，其自身的Oracle数据库继续服务于传统大中型企业，而利用收购的MySQL抢占互联网领域数据库份额，完成其战略布局。
>  MySQL是一种关系型数据库管理软件，关系型数据库的特点是将数据保存在不同的二维表中，并且将这些表放入不同的数据库中，而不是把所有数据统一放在一个大仓库里，这样的设计增加了MySQL的读取速度，灵活性和可管理性也得到了很大提高。访问及管理MySQL数据库的最常用标准化语言为SQL结构化查询语言。

### 2.2 为什么选择MySQL数据库

> 目前，绝大多数使用Linux操作系统的互联网企业都使用MySQL作为后端的数据库，从大型的BAT门户，到电商门户平台，分类门户平台等无一例外。那么，MySQL数据库到底有哪些优势和特点，让大家毫不犹豫的选择它呢？

**原因可能有以下几点**

1. 性能卓越，服务稳定，很少出现异常宕机。
2. 开放源代码且无版权制约，自主性强，使用成本低。
3. 历史悠久，社区及用户非常活跃，遇到问题，可以很快获取到帮助。
4. 软件体积小，安装使用简单，并且易于维护，安装及维护成本低。
5. 支持多种操作系统，提供多种API接口，支持多种开发语言，特别是对流行的PHP语言无缝支持。
6. 品牌口碑效应，使得企业无需考虑就直接用之。

### 2.3 安装MySQL数据库

安装配置略

## 三，FastCGI介绍

### 3.1 什么是CGI

> - CGI的全称为“通用网关接口”（Common Gateway Interface），为HTTP服务器与其他机器上的程序服务通信交流的一种工具，CGI程序须运行在网络服务器上。
> - 传统CGI接口方式的主要缺点是性能较差，因为每次HTTP服务器遇到动态程序时都需要重新启动解析器来执行解析，之后结果才会被返回给HTTP服务器。这在处理高并发访问时几乎是不可用的，因此就诞生了FastCGI。另外，传统的CGI接口方式安全性也很差，故而现在已经很少被使用了。

### 3.2 什么是FastCGI

> FastCGI是一个可伸缩的，高速地在HTTP服务器和动态脚本语言间通信的接口（在Linux下，FastCGI接口即为socket，这个socket可以是文件socket，也可以是IP  socket），主要优点是把动态语言和HTTP服务器分离出来。多数流行的HTTP服务器都支持FastCGI，包括Apache，Nginx和Lighttpd等。
>  同时，FastCGI也被许多脚本语言所支持，例如当前比较流程的脚本语言PHP。FastCGI接口采用的是C/S架构，它可以将HTTP服务器和脚本解析服务器分开，同时还能在脚本解析服务器上启动一个或多个脚本来解析守护进程。当HTTP服务器遇到动态程序时，可以将其直接交付给FastCGI进程来执行，然后将得到的结果返回给浏览器。这种方式可以让HTTP服务器专一地处理静态请求，或者将动态脚本服务器的结果返回给客户端，这在很大程度上提高了整个应用系统的性能。

**FastCGI的重要特点如下：**

- HTTP服务器和动态脚本语言间通信的接口或工具。
- 可把动态语言解析和HTTP服务器分离开。
- Nginx，Apache，Lighttpd，以及多数动态语言都支持FastCGI。
- FastCGI接口方式采用C/S结构，分为客户端（HTTP服务器）和服务器端（动态语言解析服务器）
- PHP动态语言服务器端可以启动多个FastCGI的守护进程（例如php-fpm（fcgi process mangement））
- HTTP服务器通过（例如Nginx fastcgi_pass）FastCGI客户端和动态语言FastCGI服务器端通信（例如php-fpm）

### 3.3 Nginx FastCGI的运行原理

> Nginx不支持对外部动态程序的直接调用或者解析，所有的外部程序（包括PHP）必须通过FastCGI接口来调用。FastCGI接口在Linux下是socket，为了调用CGI程序，还需要一个FastCGI的wrapper（可以理解为用于启动另一个程序的程序），这个wrappper绑定在某个固定的socket上，如端口或文件socket。当Nginx将CGI请求发送给这个socket的时候，通过FastCGI接口，wrapper接收到请求，然后派生出一个新的线程，这个线程调用解释器或外部程序处理脚本来读取返回的数据；接着，wrapper再将返回的数据通过FastCGI接口，沿着固定的socket传递给Nginx；最后，Nginx将返回的数据发送给客户端，这就是Nginx+FastCGI的整个运作过程。

![屏幕快照 2017-07-13 下午9.09.02.png-535.6kB](http://static.zybuluo.com/chensiqi/r5rc1xp1ls7ha8f57ly1idm0/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-13%20%E4%B8%8B%E5%8D%889.09.02.png)

FastCGI的主要优点是把动态语言和HTTP服务器分离开来，使Nginx专门处理静态请求及向后转发的动态请求，而PHP/PHP-FPM服务器则专门解析PHP动态请求。

### 3.4 LNMP之PHP（FastCGI方式）服务的安装和准备

PHP安装配置略

### 3.6 配置Nginx支持PHP程序请求访问

#### 3.6.1 修改Nginx配置文件

（1）查看nginx当前的配置，命令如下：

```
[root@localhost etc]# cd /usr/local/nginx/conf/
[root@localhost conf]# cp nginx.conf nginx.conf.02
[root@localhost conf]# cat nginx.conf
worker_processes  1;
error_log  logs/error.log;
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
    include extra/blog.conf;
    
}
```

（2）PHP解析，这里以blog为例讲解，内容如下：

```
[root@localhost conf]# cat extra/blog.conf 
    server {
        listen       80;
        server_name  blog.yunjisuan.com;
        location / {
            root   /var/www/html/blogcom;
            index  index.html index.htm;
        }
    }
```

**最终blog虚拟机的完整配置如下：**

```
[root@localhost conf]# cat extra/blog.conf 
    server {
        listen       80;
        server_name  blog.yunjisuan.com;
        location / {
            root   /var/www/html/blogcom;
            index  index.html index.htm;
        }
        location ~ .*\.(php|php5)?$ {
            root    /var/www/html/blogcom;
            fastcgi_pass    127.0.0.1:9000;
            fastcgi_index   index.php;
            include     fastcgi.conf;
        }
    }
```

#### 3.6.2 检查并启动Nginx

可通过如下命令检查Nginx配置文件的语法：

```
[root@localhost conf]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx-1.10.2//conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx-1.10.2//conf/nginx.conf test is successful
[root@localhost conf]# /usr/local/nginx/sbin/nginx -s reload
```

> **此步在生产环境很关键，如不提前检查语法，重启后发现语法错误会导致Nginx无法提供服务，，给用户访问体验带来不好的影响。**

#### 3.6.3 测试LNMP环境生效情况

**（1）测试PHP解析请求是否OK**

1）进入指定的默认站点目录后，编辑index.php,添加如下内容：

```
[root@localhost conf]# cd /var/www/html/blogcom/
[root@localhost blogcom]# echo "<?php phpinfo(); ?>" >test_info.php
[root@localhost blogcom]# cat test_info.php 
<?php phpinfo(); ?>
```

**以上代码为显示PHP配置信息的简单PHP文件代码**

> **注意：**
>  **对于初学者来说，以上内容最好手工录入而不要拷贝，否则可能会导致意外结果。**

2）调整Windows下的host解析（192.168.0.121为当前的机器IP），命令如下：

```
192.168.0.121 www.yunjisuan.com mail.yunjisuan.com yunjisuan.com blog.yunjisuan.com
```

3）打开浏览器，输入http://blog.yunjisuan.com/test_info.php 即可打开如下图所示界面：

![屏幕快照 2017-07-15 下午7.28.50.png-209.8kB](http://static.zybuluo.com/chensiqi/go5ygdwzfoox8ekdnwo9mxre/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-15%20%E4%B8%8B%E5%8D%887.28.50.png)

出现上述界面，表示Nginx配合PHP解析已经正常。

**（2）针对Nginx请求访问PHP，然后对PHP连接MySQL的情况进行测试**

**编辑test_mysql.php,加入如下内容：**

```
[root@localhost blogcom]# cat test_mysql.php 
<?php
    //$link_id=mysql_connect('主机名','用户','密码');
    $link_id=mysql_connect('localhost','root','123123');
    if($link_id){
        echo "mysql successful by Mr.chen !";
    }else{
        echo mysql_error();
    }
?>
```

**测试结果如下：**

![屏幕快照 2017-07-15 下午7.49.14.png-109.6kB](http://static.zybuluo.com/chensiqi/0325nrbmwvb74ny5xdqgqm3z/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-15%20%E4%B8%8B%E5%8D%887.49.14.png)

**至此，LNMP的组合已基本搭建完毕。当然，我们还没有做相关优化，因此，我们需要将虚拟机保存好。留待以后之用**

## 四， 部署一个blog程序服务

### 4.1 开源博客程序WordPress介绍

> WordPress  是一套利用PHP语言和MySQL数据库开发的开源免费的blog（博客，网站）程序，用户可以在支持PHP环境和MySQL数据库的服务器上建立blog站点。它的功能非常强大，拥有众多插件，易于扩充功能。其安装和使用也都非常方便。目前WordPress已经成为搭建blog平台的主流，很多发布平台都是根据WordPress二次开发的，如果你也想像他们一样拥有自己的blog，可购买网上的域名及空间，然后搭建LNMP环境，部署WordPress程序后就可以轻松成就自己的梦想了。

**注意：**

> **WordPress是单用户个人博客，与blog.51cto.com的多用户博客是有区别的。**

### 4.2 WordPress 博客程序的搭建准备

**（1）MySQL数据库配置准备**

1）登陆MySQL数据库，操作如下：

```
[root@localhost ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 5.5.32 MySQL Community Server (GPL)

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

2）创建一个专用的数据库WordPress，用于存放blog数据，操作如下：

```
mysql> create database wordpress;   #创建一个数据库，名字为wordpress
Query OK, 1 row affected (0.00 sec)

mysql> show databases like 'wordpress';  #查看
+----------------------+
| Database (wordpress) |
+----------------------+
| wordpress            |
+----------------------+
1 row in set (0.00 sec)

mysql> 
```

3）创建一个专用的WordPress blog管理用户，命令如下：

```
mysql> grant all on wordpress.* to wordpress@'localhost' identified by '123123';                    #localhost为客户端地址
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;            #刷新权限，使得创建用户生效
Query OK, 0 rows affected (0.00 sec)

mysql> show grants for wordpress@'localhost';   #查看用户对应权限
+------------------------------------------------------------------------------------------------------------------+
| Grants for wordpress@localhost                                                                                   |
+------------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'wordpress'@'localhost' IDENTIFIED BY PASSWORD '*E56A114692FE0DE073F9A1DD68A00EEB9703F3F1' |
| GRANT ALL PRIVILEGES ON `wordpress`.* TO 'wordpress'@'localhost'                                                 |
+------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> select user,host from mysql.user;        #查看数据库里创建的wordpress用户
+-----------+-----------+
| user      | host      |
+-----------+-----------+
| root      | 127.0.0.1 |
| root      | localhost |
| wordpress | localhost |   #只允许本机通过wordpress用户访问数据库
+-----------+-----------+
3 rows in set (0.00 sec)

mysql> quit
Bye
```

**（2）Nginx及PHP环境配置准备**

1）选择之前配置好的支持LNMP的blog域名对应的虚拟主机，命令如下：

```
[root@localhost extra]# cat blog.conf 
    server {
        listen       80;
        server_name  blog.yunjisuan.com;
        location / {
            root   /var/www/html/blogcom;
            index  index.php index.html index.htm;  #补充一个首页文件index.php
        }
        location ~ .*\.(php|php5)?$ {
            root    /var/www/html/blogcom;
            fastcgi_pass    127.0.0.1:9000;
            fastcgi_index   index.php;
            include     fastcgi.conf;
        }
    }

[root@localhost extra]# /usr/local/nginx/sbin/nginx -s reload
```

2）获取WordPress博客程序，并放置到blog域名对应虚拟主机的站点目录下，即/var/www/html/blogcom,操作命令如下：

```
[root@localhost blogcom]# ls   #浏览www.wordpress.org下载博客程序
index.html  test_info.php  test_mysql.php  wordpress-4.7.4-zh_CN.tar.gz
[root@localhost blogcom]# tar xf wordpress-4.7.4-zh_CN.tar.gz #解压
[root@localhost blogcom]# ls
index.html  test_info.php  test_mysql.php  wordpress  wordpress-4.7.4-zh_CN.tar.gz
[root@localhost blogcom]# rm -f index.html test_info.php  test_mysql.php #删除无用文件
[root@localhost blogcom]# ls
wordpress  wordpress-4.7.4-zh_CN.tar.gz
[root@localhost blogcom]# mv wordpress/* .  #把目录里的内容移动到blogcom根目录下
[root@localhost blogcom]# /bin/mv wordpress-4.7.4-zh_CN.tar.gz /root/ #移走源程序
[root@localhost blogcom]# ls -l  #完整的blog程序内容
total 192
-rw-r--r--.  1 nobody 65534   418 Sep 24  2013 index.php
-rw-r--r--.  1 nobody 65534 19935 Jan  2  2017 license.txt
-rw-r--r--.  1 nobody 65534  6956 Apr 23 09:24 readme.html
drwxr-xr-x.  2 nobody 65534  4096 Jul 14 16:04 wordpress
-rw-r--r--.  1 nobody 65534  5447 Sep 27  2016 wp-activate.php
drwxr-xr-x.  9 nobody 65534  4096 Apr 23 09:24 wp-admin
-rw-r--r--.  1 nobody 65534   364 Dec 19  2015 wp-blog-header.php
-rw-r--r--.  1 nobody 65534  1627 Aug 29  2016 wp-comments-post.php
-rw-r--r--.  1 nobody 65534  2930 Apr 23 09:24 wp-config-sample.php
drwxr-xr-x.  5 nobody 65534  4096 Apr 23 09:24 wp-content
-rw-r--r--.  1 nobody 65534  3286 May 24  2015 wp-cron.php
drwxr-xr-x. 18 nobody 65534 12288 Apr 23 09:24 wp-includes
-rw-r--r--.  1 nobody 65534  2422 Nov 20  2016 wp-links-opml.php
-rw-r--r--.  1 nobody 65534  3301 Oct 24  2016 wp-load.php
-rw-r--r--.  1 nobody 65534 33939 Nov 20  2016 wp-login.php
-rw-r--r--.  1 nobody 65534  8048 Jan 11  2017 wp-mail.php
-rw-r--r--.  1 nobody 65534 16255 Apr  6 14:23 wp-settings.php
-rw-r--r--.  1 nobody 65534 29896 Oct 19  2016 wp-signup.php
-rw-r--r--.  1 nobody 65534  4513 Oct 14  2016 wp-trackback.php
-rw-r--r--.  1 nobody 65534  3065 Aug 31  2016 xmlrpc.php
root@localhost blogcom]# chown -R www.www ../blogcom/ #授权用户访问
[root@localhost blogcom]# ls -l  #最终博客目录和权限
total 192
-rw-r--r--.  1 www www   418 Sep 24  2013 index.php
-rw-r--r--.  1 www www 19935 Jan  2  2017 license.txt
-rw-r--r--.  1 www www  6956 Apr 23 09:24 readme.html
drwxr-xr-x.  2 www www  4096 Jul 14 16:04 wordpress
-rw-r--r--.  1 www www  5447 Sep 27  2016 wp-activate.php
drwxr-xr-x.  9 www www  4096 Apr 23 09:24 wp-admin
-rw-r--r--.  1 www www   364 Dec 19  2015 wp-blog-header.php
-rw-r--r--.  1 www www  1627 Aug 29  2016 wp-comments-post.php
-rw-r--r--.  1 www www  2930 Apr 23 09:24 wp-config-sample.php
drwxr-xr-x.  5 www www  4096 Apr 23 09:24 wp-content
-rw-r--r--.  1 www www  3286 May 24  2015 wp-cron.php
drwxr-xr-x. 18 www www 12288 Apr 23 09:24 wp-includes
-rw-r--r--.  1 www www  2422 Nov 20  2016 wp-links-opml.php
-rw-r--r--.  1 www www  3301 Oct 24  2016 wp-load.php
-rw-r--r--.  1 www www 33939 Nov 20  2016 wp-login.php
-rw-r--r--.  1 www www  8048 Jan 11  2017 wp-mail.php
-rw-r--r--.  1 www www 16255 Apr  6 14:23 wp-settings.php
-rw-r--r--.  1 www www 29896 Oct 19  2016 wp-signup.php
-rw-r--r--.  1 www www  4513 Oct 14  2016 wp-trackback.php
-rw-r--r--.  1 www www  3065 Aug 31  2016 xmlrpc.php
```

### 4.3 开始安装blog博客程序

很多开源程序都支持浏览器傻瓜式的界面安装，此处也用这种方法。

1）打开浏览器输入blog.yunjisuan.com（提前做好hosts或DNS解析），回车后，出现下图：

![屏幕快照 2017-07-15 下午9.20.22.png-107.5kB](http://static.zybuluo.com/chensiqi/nhqd0ubkfkkvqk9scl3r2oj7/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-15%20%E4%B8%8B%E5%8D%889.20.22.png)

2）仔细阅读页面的文字信息后，单击“现在就开始”按钮继续，然后在出现的页面表单上填写相应的内容，如下图所示：

![QQ20170715-212718@2x.png-134kB](LNMP%E5%BA%94%E7%94%A8.assets/QQ20170715-212718@2x.png)

3）在页面表单里填好内容后，单击结尾的“提交”按钮继续，得到下图：

![屏幕快照 2017-07-15 下午9.28.36.png-46.2kB](http://static.zybuluo.com/chensiqi/3dqtx1x7xkxy2e0g0l3efebw/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-15%20%E4%B8%8B%E5%8D%889.28.36.png)

4）出现上图就表示可以安装了，单击“进行安装”按钮继续，进入下图：

![QQ20170715-213332@2x.png-166.9kB](LNMP%E5%BA%94%E7%94%A8.assets/QQ20170715-213332@2x.png)

5）根据界面提示设置blog站点的信息后，单击“安装WordPress”按钮继续。
 出现下图所示的信息就表明已经成功安装了WordPress博客。

![屏幕快照 2017-07-15 下午9.35.29.png-49.9kB](http://static.zybuluo.com/chensiqi/y2yravkri0w3gdgnuijezw0p/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-15%20%E4%B8%8B%E5%8D%889.35.29.png)

### 4.4 博客的简单使用

（1）后台登录，如下图：

![屏幕快照 2017-07-15 下午10.08.08.png-53.9kB](http://static.zybuluo.com/chensiqi/rxsh0sczf2a39ru53d7tbd4s/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-15%20%E4%B8%8B%E5%8D%8810.08.08.png)

![屏幕快照 2017-07-15 下午10.42.40.png-497.1kB](http://static.zybuluo.com/chensiqi/8ncvnrl9i8ihzsvpvpn6u99w/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-15%20%E4%B8%8B%E5%8D%8810.42.40.png)

> 其他功能同学们自己玩

### 4.5 实现WordPress博客程序URL静态化

> 实现此功能时，首先要在WordPress后台依次单击设置--->固定链接--->自定义结构，然后输入下面的代码，并保存更改。

```repl
/archives/%post_id%.html

#说明：%post_id%是数据库对应博文内容的唯一ID，例如423
```

![QQ20170715-225054@2x.png-409.5kB](LNMP%E5%BA%94%E7%94%A8.assets/QQ20170715-225054@2x.png)

**接着，在Nginx配置文件的server容器中添加下面的代码：**

```
[root@localhost extra]# cat blog.conf 
    server {
        listen       80;
        server_name  blog.yunjisuan.com;
    root    /var/www/html/blogcom;
        location / {
                index  index.php index.html index.htm;
        if (-f $request_filename/index.html){
            rewrite (.*) $1/index.html break;
        }
        if (-f $request_filename/index.php){
            rewrite (.*) $1/index.php;
        }
        if (!-f $request_filename){
            rewrite (.*) /index.php;
        }
        }
    location ~ .*\.(php|php5)?$ {
        fastcgi_pass    127.0.0.1:9000;
        fastcgi_index   index.php;
        include     fastcgi.conf;
    }
    }
```

**最后检查语法并重新加载Nginx服务，操作如下：**

```
[root@localhost extra]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx-1.10.2//conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx-1.10.2//conf/nginx.conf test is successful
[root@localhost extra]# /usr/local/nginx/sbin/nginx -s reload
```

**现在可以通过浏览器访问了，如下图所示：**

![QQ20170715-230225@2x.png-869.5kB](LNMP%E5%BA%94%E7%94%A8.assets/QQ20170715-230225@2x.png)

## 五， 本章重点回顾

1. LNMP的组合中各组件工作调度逻辑关系。
2. Nginx与PHP通过FastCGI模式通信的原理。
3. LNMP环境的企业级搭建。
4. WordPress博客程序的安装搭建与URL静态化