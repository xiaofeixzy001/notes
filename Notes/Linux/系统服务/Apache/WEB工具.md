[TOC]

# 压测工具

## ab
apache benchmark，httpd的基准性能测试工具
```
Usage: ab [options] [http[s]://]hostname[:port]/path
```
-c：模拟的并发数，就是同时有多少用户请求资源。

-n：总的请求数，就是说这些用户一共发起多少次请求，一般n=>c的值

比如：模拟100个用户，对web服务器一共发起1000次请求，也就是说平均每个用户发10次请求。
```
ab -c 100 -n 1000 http://www.xiaofei.com/test1.html

"""
Server Software:        Apache/2.2.15  # web服务器的版本号
Server Hostname:        www.xiaofei.com  # 主机名
Server Port:            80
Document Path:          /test1.html
Document Length:        289 bytes  # 访问的页面大小
Concurrency Level:      100  # 并发数
Time taken for tests:   0.120 seconds  # 整个测试所经过的时长
Complete requests:      1000  # 总共访问次数
Failed requests:        0
Write errors:           0
Non-2xx responses:      1005
Total transferred:      494460 bytes
HTML transferred:       290445 bytes
Requests per second:    8342.93 [#/sec] (mean)  # 每秒完成的请求的个数
Time per request:       11.986 [ms] (mean)  # 并发100个请求所消耗的时长
Time per request:       0.120 [ms] (mean, across all concurrent requests)  # 单个请求所消耗时长
Transfer rate:          4028.56 [Kbytes/sec] received  # 传输速率
Connection Times (ms)
             min  mean[+/-sd] median   max
Connect:        0    0   0.3      0       3
Processing:     6   11   3.9     10      25
Waiting:        6   11   3.9     10      25
Total:          6   11   4.1     10      26
Percentage of the requests served within a certain time (ms)
 50%     10
 66%     11
 75%     12
 80%     12
 90%     20
 95%     23
 98%     25
 99%     25
100%     26 (longest request)
"""
```

# 网站访问工具

## curl

curl是基于URL语法在命令行方式下工作的文件传输工具;

它支持FTP,FTPS,HTTP,HTTPS,GOPHER,TELNET,DICT,FILE及LDAP等协议;

curl支持HTTPS认证，并且支持HTTP的POST、PUT、等方法，FTP上传，kerberos认证，HTTP上传，代理服务器，cookies，用户名/密码认证，下载文件断点续传，上载文件断点续传，http代理服务器管道proxy tunneling,甚至它还支持IPv6，socks5代理服务器，通过http代理服务器上传文件到FTP服务器等，功能十分强大。

常用选项

-A "BrowerName"：设置用户访问某web服务器时显示的浏览器类型

-e “URL”：设置访问源地址

--cacert 证书 URL ：通过某个证书访问某个网址

--compressed：要求返回是压缩的格式

-H ：自定义头部信息

-I：只显示响应报文首部信息

--limit-rate：设置传输速度

-u：设置服务器的用户名和密码

-0：使用HTTP 1.0

## elinks

elinks文本浏览器,需要额外安装
```
yum install elinks -y
elinks www.xiaofei.com   # 就会打开此页面，q退出
elinks -dump   # 获取到页面数据后直接退出进程
```


# 其他工具
## ulimit
资源限定
- 软限定：允许临时超过软限定数量
- 硬限定：绝对不允许超过硬限定数量
```
# 查看当前用户允许打开的进程数量，由内核限定。
ulimit -n

# 临时修改同时打开的文件数，只有管理员可修改各种资源的软限制
ulimit -n 2000

# 修改能同时启动的进程数
ulimit -u 2000

# 永久修改，需要修改配置文件
# /etc/security/limits.conf
# /etc/security/limits.d/*.conf

vim /etc/security/limits.conf
'''
<domain>      <type>  <item>         <value>    # item表示哪个资源
*               soft    core            0
*               hard    rss             10000
@student        hard    nproc           20
@faculty        soft    nproc           20
@faculty        hard    nproc           50
ftp             hard    nproc           0        # 0表示无限制
@student        -       maxlogins       4        # -表示软硬都包括
'''
```

# htcacheclean
磁盘缓存清理工具

# htdigest
为digest认证创建和更新用户认证文件

# httxt2dbm
为rewrite map创建dbm格式的文件

# rotatelogs
不关闭httpd而切换其使用日志文件的工具，日志滚动机制，实现日志切割。

# suexec
当httpd进程需要以另外的用户身份去访问某些资源时，可以以suexec作临时切换

# apxs
让httpd得以扩展使用第三方模块的工具



# End