[TOC]

# PHP

FastCGI: php-fpm

## 依赖环境

a,扩展支持

如果想让编译的php支持mcrypt、mhash扩展和libevent，需要安装以下包

libmcrypt , libmcrypt-devel, mhash, mhash-devel,

最好使用升级的方式安装上面的rpm包，命令格式如下:

\# rpm -Uvh

mcrypt 扩展库可以实现加密解密功能，就是既能将明文加密，也可以密文还原。

mhash 是基于离散数学原理的不可逆向的php加密方式扩展库，其在默认情况下不开启。

mhash 可以用于创建校验数值，消息摘要，消息认证码，以及无需原文的关键信息保存（如密码）等.

libevent 是一个异步事件通知库文件，其API提供了在某文件描述上发生某事件时或其超时时执行回调函数的机制

它主要用来替换事件驱动的网络服务器上的event loop机制,目前来说, libevent支持/dev/poll、kqueue、select、poll、epoll及Solaris的event ports.

b,也可以根据需要安装libevent,系统一般会自带libevent,但版本有些低,因此可以升级安装之,它包含如下两个rpm包:

libevent, libevent-devel

说明: libevent是一个异步事件通知库文件,其API提供了在某文件描述上发生某事件时或其超时时执行回调函数的机制，它主要用来替换事件驱动的网络服务器上的event loop机制.目前来说,libevent支持/dev/poll、kqueue、select、poll、epoll及Solaris的event ports

c,支持xml的相关包

bzip2, libcurl

bzip2 是一个基于Burrows-Wheeler 变换的无损压缩软件能够高效的完成文件数据的压缩

libcurl主要功能就是用不同的协议连接和沟通不同的服务器，也就是相当封装了的sockPHP 

libcurl允许你用不同的协议连接和沟通不同的服务器

d,图形相关的rpm包

通常对应的错误提示: JIS-mapped Japanese font support in GD

yum install libjpeg-devel libpng-devel freetype-devel

最后附编译配置项的详细描述

## 编译安装php5.6

```shell
[root@localhost ~]# yum install -y gcc gcc-c++ zlib-devel libxml2-devel libjpeg-devel libjpeg-turbo-devel libiconv-devel freetype-devel libpng-devel gd-devel libcurl-devel libxslt-devel libmcrypt-devel mhash mcrypt openssl-devel bzip2 bzip2-devel gcc make gd-devel libjpeg-devel libpng-devel libxml2-devel bzip2-devel libcurl-devel

[root@localhost ~]# tar zxf php-5.6.22.tar.gz
[root@localhost php-5.6.22]# cd php-5.6.22
./configure --prefix=/apps/php \
--with-config-file-path=/apps/php/etc \
--with-config-file-scan-dir=/apps/php/etc/php.d \
--with-mysql=mysqlnd \
--with-mysqli=mysqlnd \
--with-iconv-dir \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir \
--with-zlib-dir \
--with-gd \
--with-libxml-dir \
--with-curl \
--with-mcrypt \
--with-openssl \
--with-mhash \
--with-bz2 \
--enable-xml \
--enable-bcmath \
--enable-shmop \
--enable-sysvsem \
--enable-sysvshm \
--enable-sysvmsg \
--enable-inline-optimization \
--enable-mbregex \
--enable-fpm \
--enable-mbstring \
--enable-gd-native-ttf \
--enable-pcntl \
--enable-sockets \
--enable-zip \
--enable-soap \
--without-pear \
--disable-rpath \
--with-gettext

[root@localhost php-5.6.22]# make && make install

# 为php提供配置文件
[root@localhost php-5.6.22]# cp php.ini-production /apps/php/etc/php.ini

# 为php-fpm提供Sysv init脚本，并将其添加至服务列表
[root@localhost php-5.6.22]# cp sapi/fpm/init.d.php-fpm /etc/rc.d/init.d/php-fpm
[root@localhost php-5.6.22]# chmod +x /etc/rc.d/init.d/php-fpm
[root@localhost php-5.6.22]# chkconfig --add php-fpm
[root@localhost php-5.6.22]# chkconfig php-fpm on

# 为php-fpm提供配置文件
[root@localhost php-5.6.22]# cd /apps/php/etc
[root@zabbix-server etc]# cp php-fpm.conf.default php-fpm.conf
[root@zabbix-server etc]# vim php-fpm.conf
"""
user = nginx
group = nginx
pid = /apps/php/var/run/php-fpm.pid
pm.max_children = 150
pm.start_servers = 8
pm.min_spare_servers = 5
pm.max_spare_servers = 10
"""

[root@python-study php-5.6.22]# service php-fpm start
[root@python-study php-5.6.22]# ps aux | grep php-fpm
```

## 安装PHP7.2改动

20181019更新：安装PHP7.2改动

--with-mysql 改为 --with-pdo-mysql

--with-mcrypt, --enable-gd-native-ttf不在支持

```
cp php-fpm.d/www.conf.default php-fpm.d/www.conf 
vim www.conf
"""
user = nginx
group = nginx
"""
```

 

## 安装mysql扩展

安装mysql_pdo.so模块

```
cd php-7.2.11/ext/pdo_mysql/
/applications/php7/bin/phpize
./configure --help
./configure --with-php-config=/applications/php7/bin/php-config --with-pdo-mysql
make
make install

vim /applications/php7/etc/php.ini
"""
增加
extension_dir=/applications/php7/lib/php/extensions/no-debug-non-zts-20170718/pdo_mysql.so
"""

# 重启服务查看PHPinfo()
```

## 安装openssl扩展

可以重新再次编译PHP，加上--enable-openssl参数即可。

但是如果只为了安装这一个扩展就去重新编译，未免有点麻烦，其实可以简单一点，只要安装openssl.so扩展就可以了。

找到之前编译安装PHP的安装包，或者从php的官网下载php7，解压并进入文件夹。

```
cd php-7.2.11/ext/openssl
/applications/php7/bin/phpize
./configure --with-openssl --with-php-config=/applications/php7/bin/php-config
make && make install
cp openssl.so /applications/php7/include/php/ext
vim /applications/php7/etc/php.ini
"""
extension=openssl.so
"""
```

如果出现如下错误：Cannot find config.m4.

Make sure that you run '/usr/local/php/bin/phpize' in the top level source directory of the module

解决办法：cp ./config0.m4 ./config.m4 即可解决

# php加速器

对2个网站分别进行压测,然后安装xcache加速器,比较前后速度

在php服务器上安装xcache

```
[root@linux-study ~]# tar xf xcache-3.1.0.tar.gz
[root@linux-study ~]# cd xcache-3.1.0
[root@linux-study ~]# /usr/local/php/bin/phpize  # php工具，生成configure脚本
[root@linux-study ~]# ./configure --enable-xcache --with-php-config=/usr/local/php/bin/php-config 
[root@linux-study ~]# make && make install

[root@linux-study ~]# cd xcache-3.1.0
[root@linux-study ~]# mkdir /etc/php.d
[root@linux-study ~]# cp xcache.ini /etc/php.d/
[root@linux-study ~]# vim /etc/php.d/xcache.ini
"""
extension = /usr/local/php/lib/php/extensions/no-debug-non-zts-20100525/xcache.so
# 如果php.ini文件中有多条extension指令行，要确保此新增的行排在第一位
"""

[root@linux-study ~]# /usr/local/php/bin/php -m |grep -i xcache  # 检查xcache是否安装成功
[root@linux-study ~]# service php-fpm restart
```

# php加载模块失败

出现类似警告提示:

PHP Warning: Module 'curl' already loaded in Unknown on line 0

PHP Warning: PHP Startup: Unable to load dynamic library '/usr/local/php/lib/php/extensions/no-debug-non-zts-20100525/fileinfo.so' - /usr/local/php/lib/php/extensions/no-debug-non-zts-20100525/fileinfo.so: cannot open shared object file: No such file or directory in Unknown on line 0

PHP Warning: PHP Startup: Unable to load dynamic library '/usr/local/php/lib/php/extensions/no-debug-non-zts-20100525/json.so' - /usr/local/php/lib/php/extensions/no-debug-non-zts-20100525/json.so: cannot open shared object file: No such file or directory in Unknown on line 0

PHP Warning: PHP Startup: Unable to load dynamic library '/usr/local/php/lib/php/extensions/no-debug-non-zts-20100525/phar.so' - /usr/local/php/lib/php/extensions/no-debug-non-zts-20100525/phar.so: cannot open shared object file: No such file or directory in Unknown on line 0

PHP Warning: PHP Startup: Unable to load dynamic library '/usr/local/php/lib/php/extensions/no-debug-non-zts-20100525/zip.so' - /usr/local/php/lib/php/extensions/no-debug-non-zts-20100525/zip.so: cannot open shared object file: No such file or directory in Unknown on line 0



报错，就是动态库加载不进来，后来查看官方手册

http://php.net/manual/zh/mongo.installation.php

发现extension_dir 路径不一致

官方手册解释如下：

确保 extension_dir 变量指向了 mongo.so 的位置,编译时会显示安装 PHP 驱动的位置,比如输出：

Installing '/usr/lib/php/extensions/no-debug-non-zts-20060613/mongo.so'

确保和运行的 PHP 是同一个扩展目录：

$ php -i | grep extension_dir

extension_dir => /usr/lib/php/extensions/no-debug-non-zts-20060613 =>

​        /usr/lib/php/extensions/no-debug-non-zts-20060613

如果不一致，则需要修改 php.ini 里的 extension_dir，或者把 mongo.so 移过去。

修改好以后，重新启动php-fpm,然后重启一下Apache就可以了，使用 phpinfo():就可以查看到Mongo了

# 压力测试

这里使用ab压测工具,可通过httpd-tools安装获取

注意:如果测试的是虚拟主机,记得要在本地hosts文件中添加解析记录.

 

```
[root@study-linux ~]# ab -c 200 -n 1000 http://v1.com/
Server Software:        nginx/1.6.1
Server Hostname:        v1.com
Server Port:            80

Document Path:          /
Document Length:        53003 bytes

Concurrency Level:      200
Time taken for tests:   152.877 seconds
Complete requests:      1000
Failed requests:        20
   (Connect: 0, Receive: 0, Length: 20, Exceptions: 0)
Write errors:           0
Non-2xx responses:      20
Total transferred:      52179020 bytes
HTML transferred:       51946580 bytes
Requests per second:    7.72 [#/sec] (mean)  # 吞吐率,单位时间内处理的请求数
Time per request:       25915.556 [ms] (mean)  # 用户平均请求等待时间
Time per request:       129.578 [ms] (mean, across all concurrent requests)  # 服务器平均请求等待时间

```

附:ab命令

-A：指定连接服务器的基本的认证凭据；

-c：指定一次向服务器发出请求数；

-C：添加cookie；

-g：将测试结果输出为“gnuolot”文件；

-h：显示帮助信息；

-H：为请求追加一个额外的头；

-i：使用“head”请求方式；

-k：激活HTTP中的“keepAlive”特性；

-n：指定测试会话使用的请求数；

-p：指定包含数据的文件；

-q：不显示进度百分比；

-T：使用POST数据时，设置内容类型头；

-v：设置详细模式等级；

-w：以HTML表格方式打印结果；

-x：以表格方式输出时，设置表格的属性；

-X：使用指定的代理服务器发送请求；

-y：以表格方式输出时，设置表格属性。

对压力测试的结果重点关注吞吐率（Requests per second）、用户平均请求等待时间（Time per request）指标：

1,吞吐率[Requests per second]:

服务器并发处理能力的量化描述，单位是reqs/s，指的是在某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐率。

记住：吞吐率是基于并发用户数的。这句话代表了两个含义：

a,吞吐率和并发用户数相关

b,不同的并发用户数下，吞吐率一般是不同的

计算公式：总请求数/处理完成这些请求数所花费的时间，即

Request per second=Complete requests/Time taken for tests

必须要说明的是，这个数值表示当前机器的整体性能，值越大越好.

2、用户平均请求等待时间[Time per request]：

计算公式：处理完成所有请求数所花费的时间/（总请求数/并发用户数），即：

Time per request=Time taken for tests/（Complete requests/Concurrency Level）

3、服务器平均请求等待时间[Time per request:across all concurrent requests]：

计算公式：处理完成所有请求数所花费的时间/总请求数，即：

Time taken for/testsComplete requests

可以看到，它是吞吐率的倒数。

同时，它也等于用户平均请求等待时间/并发用户数，即 

Time per request/Concurrency Level。

# 编译配置说明

PHP编译安装配置项说明:

```SHELL
# 大部分与apache、nginx等web服务有关
--with-aolserver=DIR    AOLserver的安装路径
--with-apxs=FILE        编译出apache1.x版本的共享模块所存放的路径
--with-apache=DIR       编译出apache1.x版本的模块,这里设定为apache软件根目录
--enable-mod-charset    启用apache的mod_charset(俄文apache用的)
--with-apxs2filter=FILE 编译apache2.0的共享过滤模块,这里设定为apache apxs工具的路径
--with-apxs2=FILE       编译共享apache2.0处理程序的模块,这里设定为apache apxs工具的路径
--with-apache-hooks=FILE共享的apache1.0的钩子模块,这里设定为apache apxs工具的路径
--with-apache-hooks-static=DIR 这里设定为apache apxs工具的路径
--disable-cli           禁用命令行模式(php-cli)
--with-continuity=DIR   编译php为连续服务模块。参数为安装Continuity Server的根目录
--enable-embed=TYPE     建立内嵌的SAPI库。参数为shared、static
--enable-fpm            开启fpm模式(nginx等服务用的)
--with-fpm-user=USER    fpm运行的用户,默认为nobody
--with-fpm-group=GRP    fpm运行的组,默认为nobody
--with-fpm-systemd      激活系统集成功能,开启后fpm可以上报给系统一些信息
--with-fpm-acl          使用POSIX 访问控制列表,5.6.5版本起有效
--with-isapi=DIR        为Zeus web服务器建立ISAPI模块
--with-litespeed        编译PHP为litespeed模块
--with-milter=DIR       编译PHP为Milter应用程序
--with-nsapi=DIR        为Netscape/iPlanet/Sun Web服务器编译PHP为NSAPI模块
--enable-phpdbg         编译开启phpdbg调试器
--enable-phpdbg-debug   编译phpdbg调试器为debug模式
--with-phttpd=DIR       编译PHP为phttpd模块
--with-pi3web=DIR       编译PHP为pi3web模块
--with-roxen=DIR        编译PHP为roxen模块
--enable-roxen-zts      编译PHP为roxen模块,线程安全
--with-thttpd=SRCDIR    编译PHP为thttpd模块
--with-tux=MODULEDIR    编译PHP为tux模块
--with-webjames=SRCDIR  编译PHP为webjames模块
--disable-cgi           禁用cgi

# General settings(综合设置):
--enable-gcov           开启gcov支持(测试代码覆盖率功能,)
--enable-debug          Compile with debugging symbols
--with-layout=TYPE      Set how installed files will be laid out.  Type can be either PHP or GNU [PHP]
--with-config-file-path=PATH php.ini文件位置[PREFIX/lib]
--with-config-file-scan-dir=PATH 扫描配置文件的路径
--enable-sigchild       使用PHP自带的SIGCHLD处理器
--enable-libgcc         启用libgcc的精确链接
--disable-short-tags    默认禁用短形式的<?作为php代码的开始标记
--enable-dmalloc        启用dmalloc（dmalloc是Linux C编程侦测记忆体溢出工具）
--disable-ipv6          关闭ipv6支持
--enable-dtrace         开启DTrace(动态跟踪)支持
--enable-fd-setsize     设置描述集的大小

# Extensions(扩展):
Extensions:
--with-EXTENSION=shared[,PATH]
# 并非所有扩展都能编译成共享方式
NOTE: Not all extensions can be build as 'shared'.

# 给个例子,如何把扩展编译成共享模式:
Example: --with-foobar=shared,/usr/local/foobar/

o Builds the foobar extension as shared extension.
o foobar package install prefix is /usr/local/foobar/


--disable-all           关闭默认为启用的所有扩展功能
--with-regex=TYPE       正则表达式库类型。选项:system|php(默认) 警告:如果你不知道这是干嘛的就别动这个选项了!
--disable-libxml        禁用LIBXML支持
--with-libxml-dir=DIR   LIBXML安装目录
--with-openssl=DIR      启用openssl支持 (OpenSSL版本号必须大于等于 0.9.6)
--with-kerberos=DIR     OPENSSL: 包含kerberos支持
--with-system-ciphers   OPENSSL: 用系统自带的密码清单(cipher list)去替代硬编码(hard coded)
--with-pcre-regex=DIR   引用pear兼容的正则表达式库
--without-sqlite3=DIR   不开启sqlite3支持
--with-zlib=DIR         开启ZLIB支持 (ZLIB版本号必须大于等于 1.0.9)
--with-zlib-dir=<DIR>   ZLIB的安装路径
--enable-bcmath         启用bcmatch（公元前风格精度数学）
--with-bz2=DIR          开启BZip2
--enable-calendar       启用日历转换支持
--disable-ctype         禁用ctype功能
--with-curl=DIR         启用cURL支持
--enable-dba            构架捆绑模块的DBA。要建立扩展的共享模块使用--enable-dba=shared参数。
--with-qdbm=DIR         DBA: QDBM support
--with-gdbm=DIR         DBA: GDBM support
--with-ndbm=DIR         DBA: NDBM support
--with-db4=DIR          DBA: Oracle Berkeley DB 4.x or 5.x support
--with-db3=DIR          DBA: Oracle Berkeley DB 3.x support
--with-db2=DIR          DBA: Oracle Berkeley DB 2.x support
--with-db1=DIR          DBA: Oracle Berkeley DB 1.x support/emulation
--with-dbm=DIR          DBA: DBM support
--with-tcadb=DIR        DBA: Tokyo Cabinet abstract DB support
--without-cdb=DIR       DBA: CDB support (bundled)（捆绑方式）
--disable-inifile       DBA: INI support (bundled)（捆绑方式）
--disable-flatfile      DBA: FlatFile support (bundled)（捆绑方式）
--disable-dom           禁用DOM支持
--with-libxml-dir=DIR   DOM: 启用libxml2并指定其安装目录
--with-enchant=DIR      启用 enchant 支持.GNU Aspell 版本号必须高于 1.1.3
--enable-exif           启用EXIF支持（从图片中获取元数据）
--disable-fileinfo      关闭fileinfo支持
--disable-filter        关闭 input filter 支持
--with-pcre-dir         FILTER: pcre install prefix
--enable-ftp            开启ftp支持
--with-openssl-dir=DIR  FTP: openssl install prefix
--with-gd=DIR           开启GD图像处理库
--with-vpx-dir=DIR      GD: 指定libvpx的安装目录
--with-jpeg-dir=DIR     GD: 指定libjpeg的安装目录
--with-png-dir=DIR      GD: 指定libpng的安装目录
--with-zlib-dir=DIR     GD: 指定libz的安装目录
--with-xpm-dir=DIR      GD: 指定libXpm的安装目录
--with-freetype-dir=DIR GD: 指定FreeType2的安装目录
--with-t1lib=DIR        GD: 指定T1lib支持
--enable-gd-native-ttf  GD: 启用TureType字符功能
--enable-gd-jis-conv    GD: 启用JIS-mapped日语字体支持
--with-gettext=DIR      包含GNU gettext支持
--with-gmp=DIR          启用GNU MP支持
--with-mhash=DIR        指定mhash的目录
--disable-hash          禁用hash支持
--without-iconv=DIR     禁用iconv支持
--with-imap=DIR         包含IMAP支持。指定c-client安装目录
--with-kerberos=DIR     IMAP: 启用kerberos支持并指定其目录
--with-imap-ssl=DIR     IMAP: 启用ssl支持并指定openssl目录
--with-interbase=DIR    启用interbase支持并指定其目录
--enable-intl           开启国际化支持(internationalization)
--with-icu-dir=DIR      Specify where ICU libraries and headers can be found
--disable-json          关闭json支持
--with-ldap=DIR         开启 LDAP 支持
--with-ldap-sasl=DIR    LDAP: 开启 Cyrus SASL 支持
--enable-mbstring       启用多字节字符串的支持
--disable-mbregex       MBSTRING: 禁用多字节正则表达式的支持
--disable-mbregex-backtrack MBSTRING: 禁用多字节正则表达式回溯检查
--with-libmbfl=DIR      MBSTRING: 使用外部的libmbfl并制定其目录
--with-onig=DIR         MBSTRING: 使用外部的onig并制定其目录
--with-mcrypt=DIR       包含mcrypt支持
--with-mssql=DIR        包含MSSQL-DB支持，并指定FreeTDS软件目录
--with-mysql-sock=SOCKPATH 定位mysql的unix 套接字指针。如果未指定，则按默认位置搜索。
--with-zlib-dir=DIR     MySQL: 设置zlib的安装目录
--with-mysqli=FILE      包含MySQLi支持。参数为mysql_config的位置
--enable-embedded-mysqli MYSQLi: 启用embedded支持。
--with-oci8=DIR         包含Oracle支持。如果使用Oracle客户端安装则使用--with-oci8=instantclient,/path/to/oic/lib
--with-odbcver=HEX      Force support for the passed ODBC version. A hex number is expected, default 0x0300.
         Use the special value of 0 to prevent an explicit ODBCVER to be defined.
--with-adabas=DIR       Include Adabas D support /usr/local
--with-sapdb=DIR        Include SAP DB support /usr/local
--with-solid=DIR        Include Solid support /usr/local/solid
--with-ibm-db2=DIR      Include IBM DB2 support /home/db2inst1/sqllib
--with-ODBCRouter=DIR   Include ODBCRouter.com support /usr
--with-empress=DIR      Include Empress support \$EMPRESSPATH
                        (Empress Version >= 8.60 required)
--with-empress-bcs=DIR
      Include Empress Local Access support \$EMPRESSPATH
      (Empress Version >= 8.60 required)
--with-birdstep=DIR     Include Birdstep support /usr/local/birdstep
--with-custom-odbc=DIR  Include user defined ODBC support. DIR is ODBC install base
      directory /usr/local. Make sure to define CUSTOM_ODBC_LIBS and
      have some odbc.h in your include dirs. f.e. you should define
      following for Sybase SQL Anywhere 5.5.00 on QNX, prior to
      running this configure script:
        CPPFLAGS=\"-DODBC_QNX -DSQLANY_BUG\"
        LDFLAGS=-lunix
        CUSTOM_ODBC_LIBS=\"-ldblib -lodbc\"
--with-iodbc=DIR        Include iODBC support /usr/local
--with-esoob=DIR        Include Easysoft OOB support /usr/local/easysoft/oob/client
--with-unixODBC=DIR     Include unixODBC support /usr/local
--with-dbmaker=DIR      Include DBMaker support
--enable-opcache        Enable Zend OPcache support
--enable-pcntl          Enable pcntl support (CLI/CGI only)
--disable-pdo           Disable PHP Data Objects support
--with-pdo-dblib=DIR    PDO: DBLIB-DB support.  DIR is the FreeTDS home directory
--with-pdo-firebird=DIR PDO: Firebird support.  DIR is the Firebird base
      install directory /opt/firebird
--with-pdo-mysql=DIR    PDO: MySQL support. DIR is the MySQL base directory
      If no value or mysqlnd is passed as DIR, the
      MySQL native driver will be used
--with-zlib-dir=DIR     PDO_MySQL: Set the path to libz install prefix
--with-pdo-oci=DIR      PDO: Oracle OCI support. DIR defaults to \$ORACLE_HOME.
      Use --with-pdo-oci=instantclient,prefix,version
      for an Oracle Instant Client SDK.
      For example on Linux with 11.2 RPMs use:
        --with-pdo-oci=instantclient,/usr,11.2
      With 10.2 RPMs use:
        --with-pdo-oci=instantclient,/usr,10.2.0.4
--with-pdo-odbc=flavour,dir
      PDO: Support for 'flavour' ODBC driver.
include and lib dirs are looked for under 'dir'.

'flavour' can be one of:  ibm-db2, iODBC, unixODBC, generic
If ',dir' part is omitted, default for the flavour
you have selected will be used. e.g.:

--with-pdo-odbc=unixODBC

will check for unixODBC under /usr/local. You may attempt
to use an otherwise unsupported driver using the \"generic\"
flavour.  The syntax for generic ODBC support is:

--with-pdo-odbc=generic,dir,libname,ldflags,cflags

When built as 'shared' the extension filename is always pdo_odbc.so
--with-pdo-pgsql=DIR    PDO: PostgreSQL support.  DIR is the PostgreSQL base
      install directory or the path to pg_config
--without-pdo-sqlite=DIR
      PDO: sqlite 3 support.  DIR is the sqlite base
      install directory BUNDLED
--with-pgsql=DIR        Include PostgreSQL support.  DIR is the PostgreSQL
      base install directory or the path to pg_config
--disable-phar          Disable phar support
--disable-posix         Disable POSIX-like functions
--with-pspell=DIR       Include PSPELL support.
      GNU Aspell version 0.50.0 or higher required
--with-libedit=DIR      Include libedit readline replacement (CLI/CGI only)
--with-readline=DIR     Include readline support (CLI/CGI only)
--with-recode=DIR       Include recode support
--disable-session       Disable session support
--with-mm=DIR           SESSION: Include mm support for session storage
--enable-shmop          Enable shmop support
--disable-simplexml     Disable SimpleXML support
--with-libxml-dir=DIR   SimpleXML: libxml2 install prefix
--with-snmp=DIR         Include SNMP support
--with-openssl-dir=DIR  SNMP: openssl install prefix
--enable-soap           Enable SOAP support
--with-libxml-dir=DIR   SOAP: libxml2 install prefix
--enable-sockets        Enable sockets support
--with-sybase-ct=DIR    Include Sybase-CT support.  DIR is the Sybase home
      directory /home/sybase
--enable-sysvmsg        Enable sysvmsg support
--enable-sysvsem        Enable System V semaphore support
--enable-sysvshm        Enable the System V shared memory support
--with-tidy=DIR         Include TIDY support
--disable-tokenizer     Disable tokenizer support
--enable-wddx           Enable WDDX support
--with-libxml-dir=DIR   WDDX: libxml2 install prefix
--with-libexpat-dir=DIR WDDX: libexpat dir for XMLRPC-EPI (deprecated)
--disable-xml           Disable XML support
--with-libxml-dir=DIR   XML: libxml2 install prefix
--with-libexpat-dir=DIR XML: libexpat install prefix (deprecated)
--disable-xmlreader     Disable XMLReader support
--with-libxml-dir=DIR   XMLReader: libxml2 install prefix
--with-xmlrpc=DIR       Include XMLRPC-EPI support
--with-libxml-dir=DIR   XMLRPC-EPI: libxml2 install prefix
--with-libexpat-dir=DIR XMLRPC-EPI: libexpat dir for XMLRPC-EPI (deprecated)
--with-iconv-dir=DIR    XMLRPC-EPI: iconv dir for XMLRPC-EPI
--disable-xmlwriter     Disable XMLWriter support
--with-libxml-dir=DIR   XMLWriter: libxml2 install prefix
--with-xsl=DIR          Include XSL support.  DIR is the libxslt base
      install directory (libxslt >= 1.1.0 required)
--enable-zip            Include Zip read/write support
--with-zlib-dir=DIR     ZIP: Set the path to libz install prefix
--with-pcre-dir         ZIP: pcre install prefix
--with-libzip=DIR       ZIP: use libzip
--enable-mysqlnd        Enable mysqlnd explicitly, will be done implicitly
      when required by other extensions
--disable-mysqlnd-compression-support
      Disable support for the MySQL compressed protocol in mysqlnd
--with-zlib-dir=DIR     mysqlnd: Set the path to libz install prefix

```