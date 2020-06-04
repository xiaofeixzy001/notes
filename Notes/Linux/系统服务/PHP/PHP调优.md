[TOC]

# 常用的加速器有
Alternative PHP Cache (APC)

遵循PHP License的开源框架，PHP opcode 缓存加速器，目前的版本不适用于PHP 5.4;

项目地址: http://pecl.php.net/package/APC

eAccelerator

源于Turck MMCache，早期的版本包含了一个 PHP encoder 和 PHP loader，目前 encoder已经不在支持

项目地址: http://eaccelerator.net/

XCache

快速而且稳定的 PHP opcode 缓存,经过严格测试且被大量用于生产环境,应用广泛

项目地址: http://xcache.lighttpd.net/

Zend Optimizer和Zend Guard Loader

Zend Optimizer并非一个opcode加速器，它是由Zend Technologies为PHP5.2及以前的版本提供的一个免费、闭源的PHP扩展，其能够运行由Zend Guard生成的加密的PHP代码或模糊代码;

Zend Guard Loader则是专为PHP5.3提供的类似于Zend Optimizer功能的扩展.

项目地址:http://www.zend.com/en/products/guard/runtime-decoders

NuSphere PhpExpress

NuSphere的一款开源PHP加速器，它支持装载通过NuSphere PHP Encoder编码的PHP程序文件，并能够实现对常规PHP文件的执行加速。

项目地址:http://www.nusphere.com/products/phpexpress.htm

Opcache

新一代PHP加速器，由Zend公司研发，其实现原理与Xcache类似，都是把PHP执行后的数据缓冲到内存中从而避免重复的编译过程，能够直接使用缓冲区已编译的代码从而提高速度，降低服务器负载，但性能却比Xcache更加优越

# XCache和Opcache
下面分别使用XCache和Opcache来对php加速

## xcache安装配置
```
tar xf xcache-3.1.0.tar.gz
cd xcache-3.1.0
/usr/local/php/bin/phpize  # php工具，生成configure脚本
./configure --enable-xcache --with-php-config=/usr/local/php/bin/php-config 
make && make install

cd xcache-3.1.0
cp xcache.ini /etc/php.d/
vim /etc/php.d/xcache.ini
"""
extension = /usr/local/php/lib/php/extensions/no-debug-non-zts-20100525/xcache.so
# 如果php.ini文件中有多条extension指令行，要确保此新增的行排在第一位
"""

/usr/local/php/bin/php -m |grep -i xcache  # 测试xcache是否安装成功
service php-fpm restart
```

安装结束时，会出现类似如下行：

Installing shared extensions: /usr/local/php/lib/php/extensions/no-debug-zts-20100525/

xcache安装路径: /usr/local/php/lib/php/extensions/no-debug-zts-20100525/

## Opcache安装配置
```
wget http://pecl.php.net/get/zendopcache-7.0.2.tgz
tar xzf zendopcache-7.0.2.tgz
cd zendopcache-7.0.2
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config
make
make install
```
配置文件设置，可直接在php.ini的最后添加如下内容，也可在PHP配置文件的扫描目录php.d下配置新文件opcache.ini，易于管理，php-config-scan-dir是在编译安装PHP时定义的。
```
vi /etc/php.d/opcache.ini
"""
[opcache]
zend_extension=/usr/local/php/lib/php/extensions/no-debug-non-zts-20100525/opcache.so
opcache.memory_consumption=128 # 分配的内存大小，单位MB，即能够存储多少预编译的PHP代码
opcache.interned_strings_buffer=8 # interned字符串占内存大小，单位MB
opcache.max_accelerated_files=4000 # 允许缓存的文件最大数量
opcache.revalidate_freq=60 # 多长时间检查文件时间戳，以改变共享内存分配，单位为s
opcache.fast_shutdown=1 # 是否开启快速关闭队列功能，1为开启
opcache.enable_cli=1 # 允许缓存CLI下的PHP程序
"""
#检查模块安装成功：
/usr/local/php/bin/php -m |grep -i opcache
```

# 压力测试
ab压测工具,可通过httpd-tools安装获取
## ab命令
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

注意:如果测试的是虚拟主机,记得要在本地hosts文件中添加解析记录.

## 测试PHP
```
ab -c 200 -n 1000 http://v1.com/
"""
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
"""
```

对压力测试的结果重点关注==吞吐率（Requests per second）、用户平均请求等待时间（Time per request）指标==：
### 吞吐率[Requests per second]:
服务器并发处理能力的量化描述，单位是reqs/s，指的是在某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐率。
记住：吞吐率是基于并发用户数的。这句话代表了两个含义：
- a,吞吐率和并发用户数相关
- b,不同的并发用户数下，吞吐率一般是不同的
- 计算公式：总请求数/处理完成这些请求数所花费的时间，即
Request per second=Complete requests/Time taken for tests，必须要说明的是，这个数值表示当前机器的整体性能，值越大越好.

### 用户平均请求等待时间[Time per request]：
计算公式：处理完成所有请求数所花费的时间/（总请求数/并发用户数），即：
Time per request=Time taken for tests/（Complete requests/Concurrency Level）

### 服务器平均请求等待时间[Time per request:across all concurrent requests]：
计算公式：处理完成所有请求数所花费的时间/总请求数，即：
Time taken for/testsComplete requests
可以看到，它是吞吐率的倒数。
同时，它也等于用户平均请求等待时间/并发用户数，即 
Time per request/Concurrency Level。


### 使用ab测试时,可能遇到的问题:
```
apr_socket_recv: Connection timed out (110)
```

在kernel2.6之前的添加项:
```
[root@linux-study ~]# uname -r
[root@linux-study ~]# vi /etc/sysctl.conf
"""
net.ipv4.netfilter.ip_conntrack_max = 655360
net.ipv4.netfilter.ip_conntrack_tcp_timeout_established = 180
"""
```
kernel2.6之后的添加项:
```
uname -r
vi /etc/sysctl.conf
"""
net.nf_conntrack_max = 655360
net.netfilter.nf_conntrack_tcp_timeout_established = 1200
"""

sysctl -p /etc/sysctl.conf  # 重载
```
如果报错：error: "net.nf_conntrack_max" is an unknown key 则需要使用modprobe载入ip_conntrack模块，lsmod查看模块已载入
```
modprobe ip_conntrack
lsmod
```

# End