[TOC]

## 一，PHP缓存加速器介绍与环境准备

### 1.1 PHP缓存加速器介绍

#### 1.1.1 操作码介绍及缓存原理

> 当客户端请求一个PHP程序时，服务器的PHP引擎会解析该PHP程序，并将其编译为特定的操作码（Operate  Code，简称opcode）文件，该文件是执行PHP代码后的一种二进制表示形式。默认情况下，这个编译好的操作码文件由PHP引擎执行后丢弃。而操作码缓存（Opcode  Cache）的原理就是将编译后的操作码保存下来，并放到共享内存里，以便在下一次调用该PHP页面时重用它，避免了相同代码的重复编译，节省了PHP引擎重复编译的时间，降低了服务器负载，同时减少了CPU和内存开销。

#### 1.1.2 PHP缓存加速软件介绍

> 为了提高PHP引擎的高并发访问及执行速度，产生了一系列PHP缓存加速软件。这些软件设计的目的就是缓存前文提到的PHP引擎解析过的1操作码文件，以便在指定时间内有相同的PHP程序请求访问时，不再需要重复解析编译，而是直接调用缓存中的PHP操作码文件，这样就提高了动态Web服务的处理速度，从而提升了用户访问企业网站的整体体验。

### 1.2 LAMP环境PHP缓存加速器的原理

**下面简单介绍Apache环境的PHP缓存加速器原理**

> 在LAMP环境中，Apache服务是使用libphp5.so响应处理PHP程序请求的，整个流程大概如下：

1）Apache接收客户的PHP程序请求，并根据规则过滤之。
 2）Apache将PHP程序请求传递给PHP处理模块libphp5.so
 3）PHP引擎定位磁盘上的PHP文件，并将其加载到内存中解析。
 4）PHP处理模块libphp5.so将PHP源代码编译成为opcode。
 5）PHP处理模块libphp5.so执行opcode，然后把opcode缓存起来。
 6）Apache接收客户端新的PHP程序请求，PHP引擎直接读取缓存执行opcode文件，并将结果返回。在这一次任务中，就无第4步的编译解析了，从而提升了PHP编译解析效率。

> PHP缓存加速器解决的是上述第5步的问题，默认情况下PHP会将opcode内容执行后丢弃，这里却通过PHP缓存加速软件，将opcode内容缓存了下来，目的是当有重复请求时，不需要再重复编译解析PHP程序代码，因为在高并发高访问量的网站上，大量的重复编译会消耗很多的系统资源和时间，而这也就会成为瓶颈，既影响了处理速度，又加重了服务器的负载，为了解决此问题，PHP缓存加速器就这样诞生了。

**下图是LAMP环境下PHP请求及操作码缓存过程的原理示意图：**

![QQ截图20170820213118.png-55.8kB](http://static.zybuluo.com/chensiqi/8gd7jgy6quibjl3nu8xlet7a/QQ%E6%88%AA%E5%9B%BE20170820213118.png)

### 1.3 LNMP环境PHP缓存加速器的原理详解

> 在LNMP环境中，PHP引擎不再使用libphp5.so模块了，而是启动了独立的FCGI即php-fpm进程，由它监听来自Nginx的PHP程序请求，并交给PHP引擎解析处理，整个执行流程大概如下：

1）Nginx接收客户端的PHP程序访问请求
 2）Nginx根据扩展名等过滤规则将PHP程序请求传递给解析PHP的FCGI（php-fpm）进程
 3）PHP FPM进程调用PHP解析器读取站点磁盘上的PHP文件，并加载到内存中。
 4）PHP解析器将PHP程序编译成为opcode文件，然后把opcode缓存起来。
 5）PHP FPM引擎执行opcode树后，返回数据给Nginx，进而返回客户端。
 6）Nginx接收客户新的PHP程序请求，PHP FPM引擎就会直接读取缓存中的opcode并执行，将结果返回。该过程中无需第4步操作，从而提升了PHP编译解析效率。

**下图为LNMP环境下PHP请求及操作码缓存过程的原理示意图**

![QQ截图20170820213237.png-44.9kB](http://static.zybuluo.com/chensiqi/qv47xgvpru22tnkrh9m99mep/QQ%E6%88%AA%E5%9B%BE20170820213237.png)

### 1.4 PHP缓存加速器软件种类及选择建议

> - PHP缓存加速器软件常见的种类有XCache，eAccelerator，APC（Alternative PHP Cache），ZendOpcache等，那么，在企业环境我们要如何选择PHP缓存加速器软件呢？
> - 事实上，任选其一即可，没必要都安装上1，都安装也可能会发生冲突。总的建议就是根据企业的业务需求及选择前的压力测试结果，或者根据个人的经验偏好选择。不过，我建议同学们首选XCache，其次是eAccelerator，如果想尝新，可以选择ZendOpcache。

- [x] :首选XCache的原因如下：
  - 经过测试，XCache效率更高，速度更快
  - XCache软件开发社区更活跃。
  - 支持更高版本的PHP，例如PHP5.5，PHP5.6
- [x] :次选eAccelerator的原因如下：
  - 安装及配置参数更简单，加速效果也不错。
  - 文档资料较多，但官方对软件的更新很慢，社区不活跃
  - 仅适合PHP版本5.4以下的程序
- [x] :选择ZendOpcache的原因如下：
  - 是PHP官方研发的新一代缓存加速软件，以后的发展潜力可能会很好，PHP5.5以前的版本可以通过ZendOpcache软件以插件扩展的方式安装，从PHP5.5版本开始已经整合到PHP软件里了，编译时只需要指定一个参数即可，例如：--enable-opcache。
  - ZendOpcache可能是未来的缓存加速首选，现在的稳定性还有待检验，小规模环境下PHP5以前的版本可以通过插件式安装使用，PHP5以上的版本可以直接指定参数编译使用。若可以忍受ZendOpcache的各种未知问题的话，也可以尝试使用。

### 1.5 PHP缓存加速器安装环境准备

#### 1.5.1 LNMP基础Web环境准备

> 在安装PHP的扩展及缓存加速软件之前，需要先安装好LNMP的完整环境，例如：能配置出现phpinfo信息的界面，表示PHP服务正常安装，同时最好可以编写一个调用数据库的简单PHP程序，例如test_mysql.php,进而确认MySQL数据库是否正常。在之前的课程中已经详细讲解了LNMP环境的安装，配置及部署方法，此处不再多提。

**当前LNMP环境软件的各个版本信息如下表：**

| 软件  | 版本            |
| ----- | --------------- |
| Linux | CentOS6.5 64bit |
| Nginx | 1.6.2           |
| MySQL | 5.5.32          |
| PHP   | 5.3.28          |

> 如果上述软件的版本对不上，在安装PHP的扩展软件时可能会遇到一些小问题。因此，建议在学习中使用的版本尽量和教案保持一致，否则可能会出现额外的问题，影响学习进度，等按照书上的操作完成了部署后，再去变换版本操作。这样的学习方法是最好的。

#### 1.5.2 检查LNMP的软件版本

1）查看Linux内核及版本相关信息，命令如下：

```
[root@LNMP ~]# cat /etc/redhat-release 
CentOS release 6.5 (Final)
[root@LNMP ~]# uname -r
2.6.32-431.el6.x86_64
[root@LNMP ~]# uname -m
x86_64
```

2）查看Nginx Web版本相关信息，命令如下：

```
[root@LNMP ~]# /usr/local/nginx/sbin/nginx -v
nginx version: nginx/1.6.2
```

3）查看PHP服务版本相关信息，命令如下：

```
[root@LNMP ~]# /usr/local/php/bin/php -v
PHP 5.3.28 (cli) (built: Aug 21 2017 19:03:26) 
Copyright (c) 1997-2013 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2013 Zend Technologies
```

4）查看MySQL服务版本相关信息，命令如下：

```
[root@LNMP ~]# mysqladmin -uroot -p123123 version
mysqladmin  Ver 8.42 Distrib 5.5.32, for linux2.6 on x86_64
Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Server version      5.5.32
Protocol version    10
Connection      Localhost via UNIX socket
UNIX socket     /tmp/mysql.sock
Uptime:         5 min 21 sec

Threads: 1  Questions: 168  Slow queries: 0  Opens: 113  Flush tables: 1  Open tables: 4  Queries per second avg: 0.523
```

### 1.6 有关LNMP环境扩展插件的部署说明

#### 1.6.1 LNMP缓存加速特别提示

> 不管是Apache还是Nginx，最后都是通过PHP提供动态程序解析的，因此，不管是Apache的libphp5.so模块方式，还是Nginx的FCGI的PHP服务方式，最终在PHP引擎上的优化是一致的，即都是基于PHP服务（php.ini）的，因此，如无特殊说明，本节以后的环境安装和优化均适用于LNMP和LAMP。

#### 1.6.2 解决部分加速软件的Perl编译问题

> 在下面各类软件的安装编译过程中，如果不解决Perl的一些环境问题可能会带来意想不到的安装错误或警告。为了避免出现这些问题导致前功尽弃，下面把一些拦路虎提前告诉大家，并解决之。

（1）配置环境变量LC_All

```
#配置环境变量LC_ALL的过程如下：
[root@LNMP ~]# echo 'export LC_ALL=C' >> /etc/profile
#设置环境变量，解决后面Perl程序插件的编译问题。符号“>>”表示向文件最佳内容
[root@LNMP ~]# tail -1 /etc/profile
#查看是否正确追加了export LC_ALL=C环境配置
export LC_ALL=C
[root@LNMP ~]# source /etc/profile
#使增加的环境变量配置生效
[root@LNMP ~]# echo $LC_ALL
C            #查看生效结果，如果不设置该变量，在安装某些加速软件时，可能会遇到如下警告（安装eAccelerator时遇到警告）
```

（2）安装Perl相关软件依赖

> 需要提前安装Perl相关软件依赖软件包，执行yum -y install perl-CPAN或yum -y install perl-devel，任意一个即可，大约依赖17个包，提前解决后面安装软件时可能遇到的报错问题。
>  如果不安装上述软件包，在后面安装ImageMagick时可能会报错。后文安装ImageMagick时有相应的报错说明。

## 二，安装PHP缓存加速器扩展

### 2.1 安装PHP eAccelerator缓存加速模块

#### 2.1.1 eAccelerator缓存加速插件说明

> - eAccelerator是一个免费的，开放源代码的PHP加速，优化及缓存的扩展插件软件，它可以缓存PHP程序编译后的中间代码文件（opcode），session数据等，降低PHP程序在编译解析时对服务器的性能开销。eAccelerator还可以加快PHP程序的执行速度，降低服务器负载压力，使PHP程序代码执行效率提高1~10倍。
> - eAccelerator会把编译好的PHP程序存放在共享内存里，然后每次从内存里调用执行，可以设定把一些不适合放在内存里缓存的编译结果存储到磁盘上，默认情况下，磁盘和内存缓存都会被eAccelerator使用。
> - eAcclelerator诞生于2004年，前身是Turck MMCache，因为开发者进入了Zend公司工作，所以开发eAccelerator的人继承了Turck MMCache的一些特性，从而设计出了eAccelerator加速器。
> - eAccelerator算是一个“老牌”的缓存加速软件，曾经在结合PHP引擎解析时被广泛使用，成熟稳定，目前代码更新不活跃，因此，使用的企业逐渐减少，但eAccelerator仍是一款值得信赖的缓存加速软件。XCache的官方也称赞eAccelerator是不错的opcode缓存器。
> - eAccelerator0.9.6.1版下载地址：https://github.com/eaccelerator/eaccelerator/downloads

#### 2.1.2 eAccelerator插件安装过程

```
[root@LNMP ~]# ls -l eaccelerator-0.9.6.1.tar.bz2 
-rw-r--r-- 1 root root 106049 Aug 22 17:05 eaccelerator-0.9.6.1.tar.bz2
[root@LNMP ~]# tar xf eaccelerator-0.9.6.1.tar.bz2 -C /usr/src/
[root@LNMP ~]# cd /usr/src/eaccelerator-0.9.6.1/
[root@LNMP eaccelerator-0.9.6.1]# /usr/local/php/bin/phpize 
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
[root@LNMP eaccelerator-0.9.6.1]# ./configure --enable-eaccelerator=shared --with-php-config=/usr/local/php/bin/php-config 
[root@LNMP eaccelerator-0.9.6.1]# make
..以上省略若干..
Build complete.
Don't forget to run 'make test'.
[root@LNMP eaccelerator-0.9.6.1]# make install
Installing shared extensions:     /usr/local/php5.3.28/lib/php/extensions/no-debug-non-zts-20090626/
[root@LNMP eaccelerator-0.9.6.1]# ls /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/
eaccelerator.so             #最后生成了eaccelerator.so模块就表示eaccelerator成功安装
```

### 2.2 安装PHP XCache缓存加速模块

#### 2.2.1 XCache缓存加速插件说明

> - XCache是一个开源的，又快又稳定的PHP  opcode缓存器/优化器，其项目leader曾经是Lighttpd（和Nginx类似的高速Web服务软件）的开发成员之一。XCache把PHP程序编译后的数据（opcode）缓存到共享内存里，避免相同的1程序重复编译。用户请求相同的PHP程序时，可以直接使用缓存中已编译好的数据，从而提高PHP的访问速度，通常可以提升2~5倍，并大幅降低服务器负载开销。
> - 很多公司使用XCache，它已经能在大流量/高负载的生产环境中稳定运行，与同类型的opcode缓存器相比在各个方面都更胜一筹，例如：社区活跃，快速开发，能够快速跟进PHP的版本更新等。
> - 当前稳定版本为3.1.x（全面支持PHP5.1~5.5）和3.2.x（2014年底发布，全面支持PHP5.1~5.6）
>    XCache软件详情请参考：
>    http://xcache.lighttpd.net或http://xcache.lighttpd.net/wiki/Introduction

#### 2.2.2 XCache插件的安装过程

```
[root@LNMP ~]# tar xf xcache-3.2.0.tar.bz2 -C /usr/src/
[root@LNMP ~]# cd /usr/src/xcache-3.2.0/
[root@LNMP xcache-3.2.0]# /usr/local/php/bin/phpize 
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
[root@LNMP xcache-3.2.0]# ./configure --enable-xcache --with-php-config=/usr/local/php/bin/php-config
[root@LNMP xcache-3.2.0]# make
..以上省略若干..
Build complete.
Don't forget to run 'make test'.
[root@LNMP xcache-3.2.0]# make install
Installing shared extensions:     /usr/local/php5.3.28/lib/php/extensions/no-debug-non-zts-20090626/
[root@LNMP xcache-3.2.0]# ls -l /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/
total 1052
-rwxr-xr-x 1 root root 416869 Aug 22 17:10 eaccelerator.so
-rwxr-xr-x 1 root root 657516 Aug 22 17:33 xcache.so           #最后生成xcache.so模块表示成功
```

### 2.3 PHP官方加速插件ZendOpcache

#### 2.3.1 ZendOpcache插件说明

> - 上面讲解了目前常见的PHP缓存加速插件：APC，eAccelerator，XCache，从PHP5.5开始，官方已经集成了新一代的缓存加速插件，其名字为ZendOpcache，功能和前三者相似但又有少许不同，据官方说，ZendOpcache缓存速度更快。
> - 这几个PHP加速插件的主要原理基本相同，就是把PHP执行后的数据缓存到内存中从而避免重复的编译过程，使其能够直接使用缓存中已编译的代码，从而提高速度，降低服务器负载。他们的效率是显而易见的，一些大型的CMS，每次打开一个页面要调用数十个PHP文件，执行数万代码，效率可想而知，安装上述加速器后，打开页面的速度明显加快。
> - PHP5.5以上版本，支持ZendOpcache也有独立的软件，并且也支持低版本的PHP 5.2.* ,PHP 5.3.*,PHP 5.4.* 。下面就以PHP 5.3版本为例讲解ZendOpcache软件，以PHP扩展插件的方式介绍安装步骤。
>    官方下载地址为：http://pecl.php.net/package/ZendOpcache

#### 2.3.2 ZendOpcache插件安装过程

> 这里以当前的最新稳定版本：zendopcache-7.0.5.tgz为例进行介绍，由于我们使用的PHP版本能为5.3，因此需要以PHP扩展的插件的方式安装，不能使用PHP编译直接加参数（--enable-opcache）的方式（PHP 5.5以上才可以），操作步骤及过程如下。

**具体的操作过程如下：**

```
[root@LNMP ~]# tar xf zendopcache-7.0.5.tgz -C /usr/src/
[root@LNMP ~]# cd /usr/src/zendopcache-7.0.5/
[root@LNMP zendopcache-7.0.5]# /usr/local/php/bin/phpize 
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
[root@LNMP zendopcache-7.0.5]# ./configure --enable-opcache --with-php-config=/usr/local/php/bin/php-config
..以上省略若干..
[root@LNMP zendopcache-7.0.5]# make
..以上省略若干..
Build complete.
Don't forget to run 'make test.
[root@LNMP zendopcache-7.0.5]# make install
Installing shared extensions:     /usr/local/php5.3.28/lib/php/extensions/no-debug-non-zts-20090626/
[root@LNMP zendopcache-7.0.5]# ls -l /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/
total 1532
-rwxr-xr-x 1 root root 416869 Aug 22 17:10 eaccelerator.so
-rwxr-xr-x 1 root root 491518 Aug 22 17:53 opcache.so              #生成opcache.so模块表示成功
-rwxr-xr-x 1 root root 657516 Aug 22 17:33 xcache.so
```

## 三，安装数据库缓存及其他PHP扩展插件

### 3.1 安装PHP Memcached扩展插件

#### 3.1.1 Memcached缓存软件说明

> - Memcached是一个开源的，支持高性能，高并发及分布式的内存缓存服务软件，从名称上看，前3个字符的单词Mem就是内存的意思，而后面5个字符的单词Cache就是缓存的意思，最后字符d，是daemon的意思，表示服务器端进程模式服务。
> - Memcached服务分为服务器端和客户端两部分，其中，服务器端软件的名字形如Memcached-1.4.13.tar.gz，客户端软件的名字形如Memcached-2.27.tar.gz。
> - Memcached诞生于2003年，最初由LiveJournal的Brad Fitzpatrick开发完成。Memcache是整个项目的名称，而Memcached是服务器端的主程序名，因其协议简单，且支持高并发而被广泛使用。
> - 在传统场景下，多数Web应用都将数据保存到RDBMS中，www服务器从中读取数据并在浏览器中显示。但随着数据量的增大，访问的集中，就会出现RDBMS的负担加重，数据库响应缓慢，网站打开延迟等问题。
> - 这时就需要Memcached了。Memcached是高性能的分布式内存缓存服务。使用Memcached的主要目的是，通过在自身内存中缓存数据库的查询结果，减少数据库访问次数，以提高动态Web应用的速度，提高网站架构的并发能力和可扩展性。
> - Memcached服务通过在事先规划好的系统内存空间中临时缓存数据库中的各类数据，以达到减少前端业务对数据库的直接高并发访问，从而提升大规模网站集群中动态服务的并发访问能力。
> - 生产场景的Memcached服务一般被用来保存网站中经常被读取的对象或数据，就像我们的客户端浏览器把经常访问的网页缓存起来一样，通过内存缓存来存取对象或数据要比磁盘存取快很多，因为磁盘是机械的介质，因此，在当今的IT企业中，Memcached的应用范围很广。

![QQ截图20170823184124.png-79.8kB](http://static.zybuluo.com/chensiqi/c977lxz5eshnh5azzi0vukqx/QQ%E6%88%AA%E5%9B%BE20170823184124.png)

**Memcached服务的工作步骤如下：**

**第一步：**程序首先检查客户端请求的数据在Memcached服务的缓存中是否存在，如果存在，直接把请求的数据返回，不再请求后端数据库。
 **第二步：**如果请求的数据在Memcached缓存中不存在，则程序会去Memcached后端的数据库服务。
 **第三步：**把从数据库中取到的数据返回给客户端。
 **第四步：**同时把新取到的数据库的数据缓存一份到Memcached服务缓存中，下次同样的请求就直接从Memcached服务缓存返回数据1，从而减轻数据库的访问压力。

#### 3.1.2 Memcached缓存PHP扩展插件安装

- 前文已经提过，Memcached分为服务器端软件和客户端插件两部分，本文是Memcached客户端PHP的扩展插件（memcache-2.2.7.tgz）在PHP环境中的安装，用于访问Memcached服务器端数据。
- PHP的Memcached扩展插件下载地址为：http://pecl.php.net/package/memcache.
- PHP的Memcached客户端扩展插件安装命令操作如下：

```
[root@LNMP memcache-2.2.7]# ./configure --enable-memcache --with-php-config=/usr/local/php/bin/php-config
[root@LNMP memcache-2.2.7]# make
..以上忽略若干..
Build complete.
Don't forget to run 'make test'.
[root@LNMP memcache-2.2.7]# make install
Installing shared extensions:     /usr/local/php5.3.28/lib/php/extensions/no-debug-non-zts-20090626/
[root@LNMP memcache-2.2.7]# make install
Installing shared extensions:     /usr/local/php5.3.28/lib/php/extensions/no-debug-non-zts-20090626/
[root@LNMP memcache-2.2.7]# ls -l /usr/local/php5.3.28/lib/php/extensions/no-debug-non-zts-20090626/
total 1776
-rwxr-xr-x 1 root root 416869 Aug 22 17:10 eaccelerator.so
-rwxr-xr-x 1 root root 246576 Aug 22 19:23 memcache.so                 #最后生成了memcache.so模块，表示memcache扩展插件成功安装
-rwxr-xr-x 1 root root 491518 Aug 22 17:53 opcache.so
-rwxr-xr-x 1 root root 657516 Aug 22 17:33 xcache.so
```

### 3.2 安装PDO_MYSQL扩展模块

#### 3.2.1 PDO_MYSQL扩展插件说明

> PDO扩展为PHP访问数据库定义了一个轻量级一致性的接口，它提供了一个数据访问抽象层，这样，无论使用的是什么数据库，都可以通过一致的函数执行查询并获取数据。
>  PDO_MYSQL扩展插件下载地址为：http://pecl.php.net/get/PDO_MYSQL-1.0.2.tgz

#### 3.2.2 PDO_MYSQL扩展插件的安装过程

> PDO_MYSQL的安装有两种方法，一种是插件方式安装，另一种是编译PHP时加入PDO_MYSQL支持,直接指定PHP的对应PDO_MYSQL编译参数安装，例如：--with-pdo-mysql=mysqlnd,同时PHP的环境也可以不装MySQL软件，直接指定如下参数--with-mysql=mysqlnd，即可让PHP支持连接MySQL数据库。当然了，建议同学们跟着本书的演示进行，先把路走通了，再去测试老男孩说明方案中提到的方法。此处采用工作中常用的PHP扩展插件方式安装。

**完整安装PDO_MYSQL的操作过程如下：**

```
[root@LNMP ~]# tar xf PDO_MYSQL-1.0.2.tgz -C /usr/src/
[root@LNMP ~]# cd /usr/src/PDO_MYSQL-1.0.2/
[root@LNMP PDO_MYSQL-1.0.2]# /usr/local/php/bin/phpize 
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
config.m4:104: warning: AC_CACHE_VAL(pdo_inc_path, ...): suspicious cache-id, must contain _cv_ to be cached
../../lib/autoconf/general.m4:1974: AC_CACHE_VAL is expanded from...
../../lib/autoconf/general.m4:1994: AC_CACHE_CHECK is expanded from...
aclocal.m4:2754: PHP_CHECK_PDO_INCLUDES is expanded from...
config.m4:104: the top level
config.m4:104: warning: AC_CACHE_VAL(pdo_inc_path, ...): suspicious cache-id, must contain _cv_ to be cached
../../lib/autoconf/general.m4:1974: AC_CACHE_VAL is expanded from...
../../lib/autoconf/general.m4:1994: AC_CACHE_CHECK is expanded from...
aclocal.m4:2754: PHP_CHECK_PDO_INCLUDES is expanded from...
config.m4:104: the top level
[root@LNMP PDO_MYSQL-1.0.2]# ./configure --with-php-config=/usr/local/php/bin/php-config --with-pdo-mysql=/usr/local/mysql
[root@LNMP PDO_MYSQL-1.0.2]# make && make install
[root@LNMP PDO_MYSQL-1.0.2]# ls -l /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/
total 1932
-rwxr-xr-x 1 root root 416869 Aug 22 17:10 eaccelerator.so
-rwxr-xr-x 1 root root 246576 Aug 22 19:23 memcache.so
-rwxr-xr-x 1 root root 491518 Aug 22 17:53 opcache.so
-rwxr-xr-x 1 root root 155916 Aug 22 20:02 pdo_mysql.so                #出现pdo_mysql.so模块，表示pdo_mysql成功安装
-rwxr-xr-x 1 root root 657516 Aug 22 17:33 xcache.so
```

## 四，安装其他的PHP扩展插件模块

### 4.1 安装图像处理程序及imagick扩展模块

#### 4.1.1 安装ImageMagick图像软件

> - ImageMagick是一套功能强大，稳定而且免费的工具集和开发包，可以用来读，写和处理超过89种基本格式的图片文件，包括流行的tiff，jpeg，gif，png，pdf，以及PhotoCD等。利用ImageMagick，可以根据Web应用程序的需要动态生成图片，还可以对一个（或一组）图片进行改变大小，旋转，锐化，减色或增加特效等操作，并将操作的结果以相同格式或其他格式保存。对图片的操作，即可以通过命令行进行，也可以用C/C++，Perl，Java，PHP，Python或Ruby编程来完成。同时ImageMagick提供了一个高质量的2D工具包，部分支持SVG。现在，ImageMagic的主要精力集中在加强性能，减少bug，以及提供稳定的API和ABI上。

**ImageMagick的常见功能如下：**

- 将图片从一个格式转换成另一个格式，包括直接转换成图标。
- 可以改变图片尺寸，旋转，锐化，减色，设置图片特效。
- 对图片设置各种尺寸缩略图。
- 将图片设置为可以适应于Web背景的透明图片。
- 将一组图片做成gif动画，直接convert
- 将几张图片做成一张组合图片。
- 在一个图片上写字或画图形，带文字阴影和边框渲染。
- 给图片加边框或框架
- 取得一些图片的特性信息

> 它几乎包括了gimp可以实现的所有常规插件功能，甚至包括各种曲线参数的渲染功能。ImageMagick的下载地址为：http://download.chinaunix.net/download/0001000/95.shtml,请提前下载好放到指定服务器的目录下。

**安装ImageMagick的操作过程如下：**

```
[root@LNMP ~]# tar xf ImageMagick-6.7.9-9.tar.xz -C /usr/src/
[root@LNMP ~]# cd /usr/src/ImageMagick-6.7.9-9/
[root@LNMP ImageMagick-6.7.9-9]# ./configure
[root@LNMP ImageMagick-6.7.9-9]# make && make install
#提示：此步不是安装PHP的扩展。因此，没有生成.so的文件
```

#### 4.1.2 安装imagick PHP扩展插件

> - Imagick插件工作需要ImageMagick软件的支持，所以，必须要先安装ImageMagick，否则会报错。
> - Imagick插件是一个可以供PHP调用ImageMagick功能的扩展模块。使用这个扩展可以使PHP具备和ImageMagick相同的功能。
> - 安装了ImageMagick图像程序后，再安装PHP的扩展Imagick插件，才能使用ImageMagick提供的api进行图片的创建与修改，压缩等操作，因为它们都集成在Imagick这个PHP扩展中。

**其安装命令操作过程如下：**

```
[root@LNMP ~]# tar xf imagick-2.3.0.tgz -C /usr/src/ 
[root@LNMP ~]# cd /usr/src/imagick-2.3.0/
[root@LNMP imagick-2.3.0]# /usr/local/php/bin/phpize
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
[root@LNMP imagick-2.3.0]# ./configure --with-php-config=/usr/local/php/bin/php-config 
#configure 的参数路径要正确配置
[root@LNMP imagick-2.3.0]# make && make install
[root@LNMP imagick-2.3.0]# ls -l /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/
total 2980
-rwxr-xr-x 1 root root  416869 Aug 22 17:10 eaccelerator.so
-rwxr-xr-x 1 root root 1072857 Aug 22 21:07 imagick.so          #出现imagick.so模块就对了
-rwxr-xr-x 1 root root  246576 Aug 22 19:23 memcache.so
-rwxr-xr-x 1 root root  491518 Aug 22 17:53 opcache.so
-rwxr-xr-x 1 root root  155916 Aug 22 20:02 pdo_mysql.so
-rwxr-xr-x 1 root root  657516 Aug 22 17:33 xcache.so
```

### 4.2 检查所有PHP扩展插件模块安装的成果

> 到此为止，常见的PHP扩展插件安装得差不多了，下面看看安装的成果把。

```
[root@LNMP imagick-2.3.0]# ls -l /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/
total 2980
-rwxr-xr-x 1 root root  416869 Aug 22 17:10 eaccelerator.so
-rwxr-xr-x 1 root root 1072857 Aug 22 21:07 imagick.so
-rwxr-xr-x 1 root root  246576 Aug 22 19:23 memcache.so
-rwxr-xr-x 1 root root  491518 Aug 22 17:53 opcache.so
-rwxr-xr-x 1 root root  155916 Aug 22 20:02 pdo_mysql.so
-rwxr-xr-x 1 root root  657516 Aug 22 17:33 xcache.so
```

> 当前一共有6个常用扩展模块，其他的若有需要以后可以后续安装。其中，eaccelerator.so，opcache.so，xcache.so属于同类软件，生产环境中安装其中一种即可，否则，可能会引起同时使用冲突，这里全都介绍全了，目的是让同学们了解方法。另外，pdo_mysql.so,imagick.so属于功能软件，可选安装，memcache.so是数据库缓存软件，可选安装。

## 五， 配置PHP加速与缓存相关的扩展插件模块

### 5.1 配置Memcache/PDO_MYSQL/imagick模块生效。

#### 5.1.1 修改PHP的配置文件php.ini

**修改php.ini的配置文件过程如下：**

```
[root@LNMP imagick-2.3.0]# cd /usr/local/php/lib/
[root@LNMP lib]# ls -l php.ini 
-rw-r--r--. 1 root root 69627 Aug 21 19:05 php.ini
[root@LNMP lib]# awk '/extension_dir/{print NR,$0}' php.ini 
819 ; extension_dir = "./"
821 ; extension_dir = "ext"
957 ; Be sure to appropriately set the extension_dir directive.
1046 ;sqlite3.extension_dir =
[root@LNMP lib]# sed -i '819 s#./#/usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/#' php.ini
[root@LNMP lib]# sed -n '819p' php.ini 
; extension_dir = "/usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/"
[root@LNMP lib]# sed -i '819 s#;##' php.ini
[root@LNMP lib]# sed -n '819p' php.ini 
 extension_dir = "/usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/"
```

**然后在php.ini的配置文件结尾追加如下几行内容**

```
[root@LNMP lib]# echo "extension = memcache.so" >> php.ini 
[root@LNMP lib]# echo "extension = pdo_mysql.so" >> php.ini 
[root@LNMP lib]# echo "extension = imagick.so" >> php.ini 
[root@LNMP lib]# tail -3 php.ini 
extension = memcache.so
extension = pdo_mysql.so
extension = imagick.so
```

#### 5.1.2 检查配置的相关模块生效情况

（1）重启PHP服务，编写测试程序phpinfo

**首先，重启PHP服务，并检查模块生效情况，命令如下：**

```
[root@LNMP lib]# pkill php-fpm
[root@LNMP lib]# ps -ef | grep php-fpm | grep -v grep
[root@LNMP lib]# /usr/local/php/sbin/php-fpm 
[root@LNMP lib]# ps -ef | grep php-fpm | grep -v grep | wc -l
3
```

**然后在前文提到的blog程序的站点目录下面增加如下phpinfo.php代码文件：**

```
[root@LNMP html]# cd /usr/local/nginx/html/
[root@LNMP html]# vim test.php
[root@LNMP html]# cat test.php 
<?php
phpinfo();
?>
```

（2）检查Memcached扩展插件

> 配好客户端的host解析，然后在浏览器中输入http://192.168.0.220/test.php页面的地址，出现的内容如下图所示：

![QQ截图20170823224111.png-57.9kB](http://static.zybuluo.com/chensiqi/1ls28m5silvaq4gny7avjcta/QQ%E6%88%AA%E5%9B%BE20170823224111.png)

通过快捷键Ctrl+F进行页面搜索，如果查找到如下图所示的内容，表示Memcached插件已生效。

![QQ截图20170823223740.png-76kB](http://static.zybuluo.com/chensiqi/onm2s3bsrwucqonz80k3xtdk/QQ%E6%88%AA%E5%9B%BE20170823223740.png)

（3）检查PDO_MYSQL扩展插件

**同理，搜索检查PDO_MYSQL扩展插件，如下图所示：**

![QQ截图20170823224501.png-33kB](http://static.zybuluo.com/chensiqi/mhw9qeumqpmd5ddye0px1dx0/QQ%E6%88%AA%E5%9B%BE20170823224501.png)

（4）检查IMAGICK扩展插件

同理，搜索检查IMAGICK扩展插件，如下图所示：

![QQ截图20170823224915.png-89.4kB](http://static.zybuluo.com/chensiqi/u5c7y6yqor204bpk1czda3kd/QQ%E6%88%AA%E5%9B%BE20170823224915.png)

> 到此为止，pdo_mysql.so,imagick.so,memcache.so这三个PHP的扩展插件就全部安装及配置完毕，后面将配置其余的缓存插件。

### 5.2 配置eAccelerator插件生效并优化参数

#### 5.2.1 配置eAccelerator缓存目录

**配置命令1：**配置eAccelerator缓存目录，操作如下：

```
[root@LNMP html]# cd /usr/local/php/lib/
[root@LNMP lib]# mkdir -p /tmp/eaccelerator
[root@LNMP lib]# chown -R nginx.nginx /tmp/eaccelerator
[root@LNMP lib]# ls -ld /tmp/eaccelerator/
drwxr-xr-x 2 nginx nginx 4096 Aug 22 22:07 /tmp/eaccelerator/
```

**配置命令2：**配置eAccelerator参数，命令如下：

```
[root@LNMP lib]# cat >> /usr/local/php/lib/php.ini <<EOF
> [eaccelerator]
> extension=eaccelerator.so
> eaccelerator.shm_size="64"
> eaccelerator.cache_dir="/tmp/eaccelerator"
> eaccelerator.enable="1"
> eaccelerator.optimizer="1"
> eaccelerator.check_mtime="1"
> eaccelerator.debug="0"
> eaccelerator.filter=""
> eaccelerator.shm_max="0"
> eaccelerator.shm_ttl="3600"
> eaccelerator.shm_prune_period="3600"
> eaccelerator.shm_only="0"
> eaccelerator.compress="1"
> eaccelerator.compress_level="9"
> EOF
```

**下表为参数详细说明：**

![QQ截图20170823230802.png-261.5kB](http://static.zybuluo.com/chensiqi/evsw606aq4dyab4z8atx0f3h/QQ%E6%88%AA%E5%9B%BE20170823230802.png)

> 更多信息请参考http://github.com/eaccelerator/eaccelerator/wiki/Settings

#### 5.2.2 检查eAccelerator加速配置情况

提示：如果仅检查加速情况配置，可以不重启Apache。

执行如下PHP命令，测试缓存的配置情况：

```
[root@LNMP lib]# /usr/local/php/bin/php -v
PHP 5.3.28 (cli) (built: Aug 21 2017 19:03:26) 
Copyright (c) 1997-2013 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2013 Zend Technologies
    with eAccelerator v0.9.6.1, Copyright (c) 2004-2010 eAccelerator, by eAccelerator
```

**重启PHP服务的命令如下：**

```
[root@LNMP lib]# pkill php-fpm
[root@LNMP lib]# ps -ef | grep php-fpm | grep -v grep
[root@LNMP lib]# /usr/local/php/sbin/php-fpm 
[root@LNMP lib]# ps -ef | grep php-fpm | grep -v grep | wc -l
3
```

**现在通过phpinfo检查eAccelerator插件结果，如下图所示：此时看看缓存目录/tmp/eaccelerator,结果如下：**

```
[root@LNMP lib]# ls -l /tmp/eaccelerator/
total 64
drwxrwxrwx 18 root root 4096 Aug 22 22:20 0
drwxrwxrwx 18 root root 4096 Aug 22 22:20 1
drwxrwxrwx 18 root root 4096 Aug 22 22:20 2
drwxrwxrwx 18 root root 4096 Aug 22 22:20 3
drwxrwxrwx 18 root root 4096 Aug 22 22:20 4
drwxrwxrwx 18 root root 4096 Aug 22 22:20 5
drwxrwxrwx 18 root root 4096 Aug 22 22:20 6
drwxrwxrwx 18 root root 4096 Aug 22 22:20 7
drwxrwxrwx 18 root root 4096 Aug 22 22:20 8
drwxrwxrwx 18 root root 4096 Aug 22 22:20 9
drwxrwxrwx 18 root root 4096 Aug 22 22:20 a
drwxrwxrwx 18 root root 4096 Aug 22 22:20 b
drwxrwxrwx 18 root root 4096 Aug 22 22:20 c
drwxrwxrwx 18 root root 4096 Aug 22 22:20 d
drwxrwxrwx 18 root root 4096 Aug 22 22:20 e
drwxrwxrwx 18 root root 4096 Aug 22 22:20 f
```

**我们可以看到/tmp/eaccelerator/缓存目录下也有内容了。以上两个检查可以确认配置是否生效。**

![QQ截图20170823231715.png-62.7kB](http://static.zybuluo.com/chensiqi/9t9jkrksoxl70r8g46sur5sk/QQ%E6%88%AA%E5%9B%BE20170823231715.png)

#### 5.2.3 访问PHP页面测试检查eAccelerator加速情况

> 重启PHP服务后，在浏览器里访问PHP页面，如出现访问phpinfo页面，就会有下面的缓存文件（其实上面已经访问过了）。

```
[root@LNMP lib]# find /tmp/eaccelerator/ -type f | xargs file
/tmp/eaccelerator/9/e/eaccelerator-86746.197162: data
/tmp/eaccelerator/b/2/eaccelerator-86746.592229: data
/tmp/eaccelerator/e/2/eaccelerator-86746.643229: data
/tmp/eaccelerator/e/1/eaccelerator-86746.926262: data
/tmp/eaccelerator/c/2/eaccelerator-86746.558262: data
/tmp/eaccelerator/6/0/eaccelerator-86746.442262: data
/tmp/eaccelerator/6/0/eaccelerator-86746.352262: data
/tmp/eaccelerator/d/1/eaccelerator-86746.226262: data
/tmp/eaccelerator/d/4/eaccelerator-86746.283362: data
/tmp/eaccelerator/d/4/eaccelerator-86746.583362: data
/tmp/eaccelerator/8/e/eaccelerator-86746.167162: data

提示：eaccelerator-xxxxxx等就是Cache的内容，而且是phpinfo的页面缓存内容，类型为data。
```

#### 5.2.4 使用tmpfs优化eAccelerator缓存目录

> tmpfs是一种基于内存的文件系统，通常使用tmpfs作为数据临时存储，比本地磁盘存储快很多，此方法适用于临时使用的各类缓存场景。例如：上传图片时很多软件默认在/tmp下临时缓存切图，到tmpfs文件系统上，让访问缓存的数据更快。具体操作方法如下：

```
[root@LNMP ~]# mount -t tmpfs -o size=16m tmpfs /tmp/eaccelerator 
#创建16M大小的tmpfs类型文件系统挂载到/tmp/eaccelerator
[root@LNMP ~]# df -h
Filesystem                    Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root   18G  2.6G   14G  16% /
tmpfs                         491M     0  491M   0% /dev/shm
/dev/sda1                     485M   33M  427M   8% /boot
/dev/sr0                      4.2G  4.2G     0 100% /media/cdrom
tmpfs                          16M     0   16M   0% /tmp/eaccelerator
[root@LNMP ~]# grep eacc /proc/mounts       #检查挂载情况
tmpfs /tmp/eaccelerator tmpfs rw,relatime,size=16384k 0 0
[root@LNMP ~]# echo "mount -t tmpfs -o size=16m tmpfs /tmp/eaccelerator" >> /etc/rc.local 
[root@LNMP ~]# tail -1 /etc/rc.local 
mount -t tmpfs -o size=16m tmpfs /tmp/eaccelerator  #配置永久挂载，生产场景size可以根据实际情况调整。
```

### 5.3 配置XCache插件加速

> 提示：
>  XCache和eAccelerator功能相近，安装一个即可。考虑到知识的完整性，本节将其作为知识点来讲解，配置之前应删除eAccelerator的所有配置。

#### 5.3.1 xcache.ini参数说明

> xcache软件的解压目录/usr/src/xcache-3.2.0/下存在一个名字为xcache.ini的配置文件，取XCache的配置文件。下面的表给出了xcache配置文件参数的说明。

![QQ截图20170823235301.png-324kB](http://static.zybuluo.com/chensiqi/mqtvfhtsu0ep5kxm29tw1nr8/QQ%E6%88%AA%E5%9B%BE20170823235301.png)

#### 5.3.2 修改php.ini配置XCache

1）先在配置XCache参数前加个配置分界符，配置命令如下：

```
[root@LNMP ~]# cd /usr/local/php/lib/        
[root@LNMP lib]# echo >> php.ini 
[root@LNMP lib]# echo '; xcache config by Mr.chen 2017-8-24' >> php.ini 
[root@LNMP lib]# tail -2 php.ini

; xcache config by Mr.chen 2017-8-24
```

2）编辑xcache.ini,修改XCache的配置参数，调整的关键参数见下表：

![QQ截图20170824000559.png-47.8kB](http://static.zybuluo.com/chensiqi/fe7sdzhinh0qeat9xh2yiadq/QQ%E6%88%AA%E5%9B%BE20170824000559.png)

**以上参数需要根据生产硬件的大小，以及业务数据的访问量来调整**

```
[root@LNMP lib]# vim /usr/src/xcache-3.2.0/xcache.ini
修改相应配置参数后保存退出
```

3）将修改后的xcache.ini合并到php.ini结尾。命令如下：

```
[root@LNMP lib]# cat /usr/src/xcache-3.2.0/xcache.ini >> php.ini
修改后，整个xcache.ini的内容如下：
[root@LNMP lib]# tail -85 php.ini | egrep -v "^;|^$"
[xcache-common]
extension = xcache.so
[xcache.admin]
xcache.admin.enable_auth = On
xcache.admin.user = "mOo"
xcache.admin.pass = "md5 encrypted password"
[xcache]
xcache.shm_scheme =        "mmap"
xcache.size  =               256M
xcache.count =                 2
xcache.slots =                8K
xcache.ttl   =                86400
xcache.gc_interval =           3600
xcache.var_size  =            64M
xcache.var_count =             1
xcache.var_slots =            8K
xcache.var_ttl   =             0
xcache.var_maxttl   =          0
xcache.var_gc_interval =     300
xcache.var_namespace_mode =    0
xcache.var_namespace =        ""
xcache.readonly_protection = Off
xcache.mmap_path =    "/dev/zero"
xcache.coredump_directory =   ""
xcache.coredump_type =         0
xcache.disable_on_crash =    Off
xcache.experimental =        Off
xcache.cacher =               On
xcache.stat   =               On
xcache.optimizer =           Off
[xcache.coverager]
xcache.coverager =           Off
xcache.coverager_autostart =  On
xcache.coveragedump_directory = ""
```

#### 5.3.3 检查XCache加速情况配置

**再次执行PHP的命令，查看缓存的生效情况，命令如下：**

```
[root@LNMP lib]# /usr/local/php/bin/php -v
PHP Warning:  Cannot load module 'XCache' because conflicting module 'eAccelerator' is already loaded in Unknown on line 0
PHP 5.3.28 (cli) (built: Aug 21 2017 19:03:26) 
Copyright (c) 1997-2013 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2013 Zend Technologies
    with eAccelerator v0.9.6.1, Copyright (c) 2004-2010 eAccelerator, by eAccelerator
```

看到了把，上面提示说XCache和eAccelerator冲突。在实际工作中，这两个软件可以任选一个。此时可以把eAccelerator的配置参数删除，保留XCache参数，命令如下：

```
[root@LNMP lib]# sed -n '1922,1936p' php.ini
[eaccelerator]
extension=eaccelerator.so
eaccelerator.shm_size="64"
eaccelerator.cache_dir="/tmp/eaccelerator"
eaccelerator.enable="1"
eaccelerator.optimizer="1"
eaccelerator.check_mtime="1"
eaccelerator.debug="0"
eaccelerator.filter=""
eaccelerator.shm_max="0"
eaccelerator.shm_ttl="3600"
eaccelerator.shm_prune_period="3600"
eaccelerator.shm_only="0"
eaccelerator.compress="1"
eaccelerator.compress_level="9"
[root@LNMP lib]# sed -i '1922,1936 s#^#;#' php.ini
[root@LNMP lib]# sed -n '1922,1936p' php.ini
;[eaccelerator]
;extension=eaccelerator.so
;eaccelerator.shm_size="64"
;eaccelerator.cache_dir="/tmp/eaccelerator"
;eaccelerator.enable="1"
;eaccelerator.optimizer="1"
;eaccelerator.check_mtime="1"
;eaccelerator.debug="0"
;eaccelerator.filter=""
;eaccelerator.shm_max="0"
;eaccelerator.shm_ttl="3600"
;eaccelerator.shm_prune_period="3600"
;eaccelerator.shm_only="0"
;eaccelerator.compress="1"
;eaccelerator.compress_level="9"

#提示：将eaccelerator部分的配置信息注释掉

[root@LNMP lib]# /usr/local/php/bin/php -v
PHP 5.3.28 (cli) (built: Aug 21 2017 19:03:26) 
Copyright (c) 1997-2013 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2013 Zend Technologies
    with XCache v3.2.0, Copyright (c) 2005-2014, by mOo
    with XCache Cacher v3.2.0, Copyright (c) 2005-2014, by mOo
```

> XCache和eAccelerator均使用系统的共享内存作为存储空间，因此，有必要调整系统的共享内存大小参数。下面介绍对应的XCache和eAccelerator内核优化方法，命令如下：

```
[root@LNMP lib]# tail /etc/sysctl.conf 
kernel.msgmnb = 65536

# Controls the maximum size of a message, in bytes
kernel.msgmax = 65536

# Controls the maximum shared segment size, in bytes
kernel.shmmax = 68719476736

# Controls the maximum number of shared memory segments, in pages
kernel.shmall = 4294967296
```

**现在重启PHP服务，然后通过phpinfo界面检查XCache插件结果，如下图所示：**

```
[root@LNMP lib]# pkill php-fpm
[root@LNMP lib]# /usr/local/php/sbin/php-fpm   
```

![QQ截图20170824003426.png-48.2kB](http://static.zybuluo.com/chensiqi/dhhxmo9hv258gjceyfwvfje3/QQ%E6%88%AA%E5%9B%BE20170824003426.png)

#### 5.3.4 配置Web界面查看XCache缓存加速信息

> 使用Linux命令行md5sum命令，或者打开浏览器输入地址:http://xcache.lighttpd.net/demo/cacher/mkpassword.php，通过输入字符串生成Xcache管理员的密码。这里使用如下md5sum命令生成密文密码。

```
[root@LNMP xcache-3.2.0]# cd /usr/local/php/lib/ 
[root@LNMP lib]# sed -n '1950p' php.ini
xcache.admin.user = "mOo"
[root@LNMP lib]# sed -i '1950 s#mOo#yunjisuan#' php.ini
[root@LNMP lib]# sed -n '1950p' php.ini
xcache.admin.user = "yunjisuan"                     #改用户名 
[root@LNMP lib]# sed -n '1951p' php.ini
xcache.admin.pass = "md5 encrypted password"        #改成密文密码
[root@LNMP lib]# echo "123456" | md5sum
e10adc3949ba59abbe56e057f20f883e  -
[root@LNMP lib]# vim php.ini +1951
[root@LNMP lib]# sed -n '1951p' php.ini 
xcache.admin.pass = "e10adc3949ba59abbe56e057f20f883e"
```

**然后复制XCache软件下面的缓存加速管理PHP程序到站点目录下，命令如下：**

```
[root@LNMP lib]# cd /usr/src/xcache-3.2.0/
[root@LNMP xcache-3.2.0]# cp -a htdocs/ /usr/local/nginx/html/xadmin
[root@LNMP xcache-3.2.0]# chown -R nginx.nginx /usr/local/nginx/html/xadmin
[root@LNMP xcache-3.2.0]# pkill php-fpm
[root@LNMP xcache-3.2.0]# /usr/local/php/sbin/php-fpm 
```

**访问http://192.168.0.220/xadmin/index.php，弹出验证框，需要输入密码，如下图所示：**

![QQ截图20170824005350.png-45.9kB](http://static.zybuluo.com/chensiqi/pr7j0p4yj76a4ykk8ozkno1q/QQ%E6%88%AA%E5%9B%BE20170824005350.png)

**登陆后的相关信息如下图所示：**

![QQ截图20170824010102.png-91.6kB](http://static.zybuluo.com/chensiqi/ggwebjv00msdu86mkhr6n2b1/QQ%E6%88%AA%E5%9B%BE20170824010102.png)

> 有关缓存XCache的状态，命中等相关信息都可以通过这个XCache管理界面查看。

### 5.4 配置ZendOpcache插件加速

#### 5.4.1 配置ZendOpcache参数

**在php.ini的最后面加入下面几行：**

```
[root@LNMP ~]# tail -10 /usr/local/php/lib/php.ini
[opcache]
zend_extension=/usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/opcache.so;extension=opcache.so
opcache.memory_consumption=32
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=1000
opcache.revalidate_freq=60
opcache.fast_shutdown=1
opcache.enable_cli=1
```

#### 5.4.2 检查ZendOpcache生效情况

```
#下面使用PHP命令检查生效情况：
[root@LNMP ~]# /usr/local/php/bin/php -v
PHP 5.3.28 (cli) (built: Aug 21 2017 19:03:26) 
Copyright (c) 1997-2013 The PHP Group
Zend Engine v2.3.0, Copyright (c) 1998-2013 Zend Technologies
    with XCache v3.2.0, Copyright (c) 2005-2014, by mOo
    with Zend OPcache v7.0.5, Copyright (c) 1999-2015, by Zend Technologies
    with XCache Cacher v3.2.0, Copyright (c) 2005-2014, by mOo
```

> 可以看到ZendOpcache已经生效，并且貌似和XCache相处的比较融洽，不过工作中是否多选，还要慎重选择。
>  现在重启PHP服务，并通过phpinfo界面检查XCAche插件结果，如下图所示：

![QQ截图20170824185113.png-18.3kB](http://static.zybuluo.com/chensiqi/rqvvyf7phh7xauwytcwzkcp1/QQ%E6%88%AA%E5%9B%BE20170824185113.png)

![QQ截图20170824185447.png-44.3kB](http://static.zybuluo.com/chensiqi/db92cx959dfrou53hz6gzlgn/QQ%E6%88%AA%E5%9B%BE20170824185447.png)

#### 5.4.3 ZendOpcache配置参数说明

如下图中针对OPcache的部分重要参数进行了说明。

![QQ截图20170824185534.png-112.8kB](http://static.zybuluo.com/chensiqi/j2ib7g8fqjvzla7ixqjf7iib/QQ%E6%88%AA%E5%9B%BE20170824185534.png)

> **说明：**

> - ZendOpcache是PHP官方的新一代的缓存加速软件，PHP 5.5以前通过ZendOpcache软件以插件扩展的方式安装，从PHP 5.5版本开始已经整合到PHP软件里，编译时只需要指定一个参数即可，
>    例如：---enable-opcache
> - ZendOpcache可能是未来的首选项，现在的稳定性还有待检查。在小规模环境下，php5以上的版本可以使用。如果可以忍受其未知的问题也可以使用。

## 六，生产环境PHP扩展插件的安装建议

### 6.1 PHP的安装插件表格列表

**常见的PHP扩展插件及其说明见下表所示：**

![QQ截图20170824222816.png-127.2kB](http://static.zybuluo.com/chensiqi/oxozhxkun8bbz5vr1448avja/QQ%E6%88%AA%E5%9B%BE20170824222816.png)

#### 6.2 生产环境插件的安装建议

1）对于功能性插件，如果业务产品不需要使用，可以暂时不考虑安装，例如PDO_MYSQl\memcache\imagick等。如果不清楚是否需要，最好还是装上，有备无患。

2）对于性能优化插件，eAccelerator，XCache，ZendOpcache，APC可以安装任意一种，具体情况看实际业务需求，在选择时最好能搭建相关环境进行压力测试，然后根据实际测试结果来选择，用数据说话很重要。

#### 6.3 PHP加速插件的测试对比

下表为相关PHP加速插件的测试结果对比参考
 下面是针对PHP加速器比较结果进行的总结

- 通过测试得出，eAccelerator在请求时间和内存占用综合方面是最好的。
- 通过测试得出，是用加速器比无加速器在请求时间快了3倍左右。
- 通过各个官方观察，XCache的更新是最快的，这也说明它是最有发展前景的。

![QQ截图20170824223825.png-46.7kB](http://static.zybuluo.com/chensiqi/aisy8k4wb10j4554hmhw5s5l/QQ%E6%88%AA%E5%9B%BE20170824223825.png)

> 以上是总结结果，也许有人会疑惑到底用哪个加速器好呢？我只能告诉你，首先，用一定比不用好，其次每个加速器还有一些可以调优的参数，所以要根据系统环境而定。此外，XCache和ZendOpcache这两款加速器的潜力还是很大的，可以多关注一下。

## 七，补充知识

### 7.1 phpize是什么？

> - 安装PHP扩展插件的时候，常常有这样一条命令：/usr/local/php/bin/phpize，可能有同学会问phpize有什么用？
> - 事实上，phpize是用来扩展PHP扩展模块的，通过phpize可以建立PHP的外挂模块。比如想在原来编译好的PHP中加入Memcached等扩展模块，可以使用phpize工具。
>    PHP的官方说明地址：http://php.net/manual/en/install.pecl.phpize.php
> - 那么，要如何使用phpize呢？
> - 编译PHP后，其bin目录下会有phpize这个脚本文件。在编译要添加的扩展模块之前，执行以下phpize就可以了。比如，现在现在PHP中加入memcached扩展模块，那么要做的只是执行如下的Memcached客户端软件安装命令：

```
#例如：如下操作，这里仅作操作示例
wget -q http://pecl.php.net/get/memcache-2.2.7.tgz
tar zxf memcache-2.2.7.tgz
cd memcache-2.2.7
/usr/local/php/bin/phpize       #在执行configure前必须先执行这个命令
./configure --enable-memcache --with-php-config=/usr/local/php/bin/php-config
make
make install
```

**这样就编译完成了，还需要做的就是在php.ini里加入如下**
 extension_dir = "/usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/"
 extension = memcache.so

> **提示：**
>  上述两行配置的作用是加载使得Memcached客户端配置生效
>  extension_dir的路径就是memcache.so模块文件所在的路径。

## 八，PHP缓存加速压力测试练习（同学们开始实验）

分别安装ZendOpcache，eAccelerator，XCache缓存加速插件，通过压测软件对比三者的缓存效率。

**测试方法：**

1）不装任何加速插件和分别安装某一个缓存加速软件。
 2）可用压力测试软件webbench，loadruner
 3）用压力测试方法，通过数据看看到底哪个加速器好。

```

```

## 九，本节学习内容参考资料

- eAccelerator官方地址
   http://eaccelerator.net
   https://github.com/eaccelerator/eaccelerator/wiki
   https://github.com/eaccelerator/eaccelerator/downloads
- XCache官方资料
   http://xcache.lighttpd.net
   http://xcache.lighttpd.net/wiki/Introduction
- ZendOpcache官方资料
   http://pecl.php.net/package/ZendOpcache

## 附录：webbench的安装及使用

1)webbench的安装

```
[root@LNMP webbench-1.5]# wget http://www.ha97.com/code/webbench-1.5.tar.gz
[root@LNMP ~]# tar xf webbench-1.5.tar.gz
[root@LNMP ~]# cd webbench-1.5
[root@LNMP webbench-1.5]# mkdir /usr/local/man
[root@LNMP webbench-1.5]# make install clean
[root@LNMP webbench-1.5]# which webbench    
/usr/local/bin/webbench
```

2）webbench的使用

```shell
 [root@localhost ~]# echo "192.168.0.220 www.yunjisuan.com" >> /etc/hosts              #将要测试的域名添加映射
[root@localhost ~]# webbench -c 600 -t 60 http://www.yunjisuan.com/test.php 
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.

Benchmarking: GET http://www.yunjisuan.com/test.php
600 clients, running 60 sec.

Speed=322357 pages/min, 1984728 bytes/sec.
Requests: 322357 susceed, 0 failed.



#说明：
-c：并发数
-t：持续时间
Speed：每秒处理的请求数
bytes/sec:每秒传输的数据量
Requests：322357成功，0失败

[root@localhost ~]# webbench -c 2000 -t 60 http://www.yunjisuan.com/test.php
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.

Benchmarking: GET http://www.yunjisuan.com/test.php
2000 clients, running 60 sec.

Speed=303303 pages/min, 5304600 bytes/sec.
Requests: 303176 susceed, 127 failed.

#这个测试就到达负荷了。出现了127个失败
```

