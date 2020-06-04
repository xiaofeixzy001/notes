[TOC]

# 持久连接
```
# 服务器根目录
ServerRoot “/etc/httpd”

# 是否支持持久连接
KeepAlive {Off|On}

# 允许持久连接的最大个数 
MaxKeepAliveRequests 100

# 持久连接超时时间，单位秒
KeepAliveTimeout 15
```

# MPM模块
```
# 查看httpd核心模块
httpd -l

# 查看event模块
httpd.event -l

# 查看worker模块
httpd.worker -l

# 更改默认模块：
vim /etc/sysconfig/httpd
"""
HTTPD=/usr/sbin/httpd.worker
"""
```

MPM方式:
- prefork:多进程模型，一个主进程生成多个子进程，每个子进程响应一个请求
- worker:多线程模型，由主进程生成多个子进程，由子进程生成线程，每个线程响应一个请求
- event:事件驱动模型，基于事件驱动机制来响应请求，单线程响应多个用户请求
```
<IfModule prefork.c>
StartServers 8  # 服务启动时开启8个空闲进程
MinSpareServers 5  # 保持最少空闲进程
MaxSpareServers 20  # 保持最大空闲进程
ServerLimit 256  # 为MaxClients最多启动多少个进程
MaxClients 256  # 客户端并发请求最大数量
MaxRequestsPerChild 4000  # 每个子进程允许处理的请求上限，达到上限后必须关掉重启 
</IfModule>

<IfModule worker.c>
StartServers 4  # 服务启动时开启空闲进程
MaxClients 300  # 客户端并发请求最大数量
MinSpareThreads 25  # 最小空闲线程
MaxSpareThreads 75  # 最大空闲线程
ThreadsPerChild 25  # 每个进程可以启动多少个线程
MaxRequestsPerChild 0  # 不限制
</IfModule>
```

# 监听的地址和端口
```
Listen 80      ##监听本机的80端口
Listen 8080     ##监听本机的8080端口
Listen 172.10.1.1:80     ##监听特定IP的80端口
Listen 172.10.1.2:8080     ##监听指定IP的特定端口
#httpd -D DUMP_MODULES      ##显示当前装载的模块命令
#LoadModule Module_Name /path/to/Module_File      ##加载某个模块,在配置文件中配置 
DocumentRoot "/var/www/html"      ##指定站点根目录
```

# 站点路径访问控制
## 基于本地文件系统路径:
```
<Directory "/var/www/html"> ###表示默认主页文件存储位置 
    Options -Indexes -FollowSymLinks ###-表示禁用
    AllowOverride None
    Order allow,deny
    Allow from all ###表示允许所有
</Directory>
```
Indexes：当访问的路径下无默认主页，则将所有资源以列表形式呈现给用户，危险，慎用.

FollowSymLinks：跟踪符号链接，如果访问的是个符号链接文件，就会自动找到源文件并显示出来，危险，慎用.

AllowOverride：支持在每个页面目录下创建一个隐藏文件.htaccess用于实现对此目录中资源访问时的访问控制功能。


## 基于URL访问路径做访问控制
```
<Location "path/to/URL"> 
    ...
</location>
```

## 基于IP做访问控制
```
Order Allow, Deny 
Allow from all
Deny form
```
Order：定义检查次序。这里表示先检查allow定义的内容，在检查deny定义的内容.

Allow：定义allow内容，表示允许哪些地址可以访问.

deny：如果不定义内容，默认为拒绝。如果定义内容，则拒绝规定的内容.

注意两者前后次序

如果先allow，后deny，都没定义内容，那么默认都拒绝

先deny，后allow，默认都允许。

所以，如果先A后D，表示定义白名单，仅允许A，

如果先D后A，表示定义黑名单，仅拒绝D。

form：IP，Network Address
网络地址格式较为灵活：
- 172.16
- 172.16.0.0
- 172.16.0.0./16
- 172.16.0.0/255.255.0.0


# 定义默认主页面
```
DirectoryIndex index.html index.html.var
```
优先级自左向右

# 日志配置
默认日志文件路径:/var/log/httpd/
## 错误日志配置
```
ErrorLog logs/error_log
LogLevel warn
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined 
LogFormat "%h %l %u %t \"%r\" %>s %b" common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-agent}i" agent
CustomLog logs/access_log combined
```

## 访问日志配置
```
# 定义访问日志格式和文件名
CustomLog “logs/access_log” LogFormat_Name
       # 错误日志记录位置
       Error_Log “/path/to/error_log”
       # 错误日志级别，定义哪种级别以上才记录日志
       LogLevel {debug，info，notice，warn，error，crit，alert，emerg. }
```

## 定义日志格式
使用LogFormat ""

- %h：客户端地址
- %l：远程登录名，通常为-
- %u：认证时输入的用户名，-为空
- %t：服务器端收到请求时的时间
- %r：请求报文的起始行
- %>s：响应状态码
- %b：响应报文的长度(字节)
- %｛HEADER_NAME｝i：记录指定首部对应的值


# URL路径别名
实现URL路径的映射，从而所访问的资源不再依赖于站点根目录
```
Alias /images/ "/www/images/"
Alias /URL/ "/path/to/somewhere/"
```
访问http://www.xiaofei.com/images，原访问的是Decuments定义的/var/www/html/images，通过alias定义，访问的目标改成了/www/images/


# 用户帐号认证
## 用户认证方式
- 基本认证：Basic
- 摘要认证：Digest
- 虚拟用户：仅用于访问某服务或获取某资源的凭证


认证用户的信息可存放在：
- 文本文件：.htpasswd
- SQL数据库
- dbm：数据库引擎
- ldap：轻量级目录服务访问协议

## 开启认证
编辑主配置文件httpd.conf
```
# 定义需要进行认证的目标文件或目录
<Directory "/var/www/html/admin">
   Options Indexes FollowSymLinks
   AllowOverride AuthConfig
   
   # 定义认证的类型：明文(basic),密文(digest)
   AuthType Basic
   
   # 访问该资源时的提示信息
   AuthName "admin user zone"

   # AuthBasicProvider file
   
   # 认证所需要的虚拟帐号存放文件路径
   AuthUserFile "/etc/httpd/conf/.htpasswd"
   
   # 定义.htpasswd文件内的用户都可以登录访问/var/www/html/admin
   Require valid-user

   # 定义.htpasswd文件内只有user1可以登录访问，其他都不可以。
   # Require user user1
   
   # 定义组文件路径
   AuthGroupFile "/etc/httpd/conf/.htgroup"
   
   # 定义groupuser组内的用户可以进行验证。
   Require group groupuser
</Directory>
```
注意：Require user 和 group定义一个,定义完成后，开始创建用于验证的虚拟帐号或组
```
# 创建第1个用户user1，需要用-c 
htpasswd -c -m /etc/httpd/conf/.htpasswd user1 

# 创建第2个用户user2，不需要-c
htpasswd -m /etc/httpd/conf/.htpasswd user2 

# 创建第3个用户user3，不需要-c
htpasswd -m /etc/httpd/conf/.htpasswd user3

# 创建第4个用户user4，不需要-c
htpasswd -m /etc/httpd/conf/.htpasswd user4

# 定义组和组内成员
vim /etc/httpd/conf/.htgroup
"""
groupuser: user2 user4
"""

# 检查配置文件
httpd -t

service httpd reload
```

htpasswd命令

-c：如果此文件不存在，则创建。注意，只能在创建第一个用户时使用，否则创建第二个用户用-c就会覆盖掉第一个用户信息

-m：以md5格式的编码存储用户的密码信息

-D：删除指定用户
测试访问http://172.10.0.1/admin就会提示输入帐号密码


# 设定默认字符集
```
AddDefaultCharset
```

汉语：GB2312, GB18030, GBK


# CGI脚本
让站点支持执行CGI脚本，默认存放CGI脚本路径/var/www/cgi-bin/


# 虚拟主机
虚拟主机可让一个物理主机提供多个站点，每个站点使用不同的访问路径资源.

## 虚拟主机配置方式
- 基于端口
- 基于IP
- 基于主机名


使用虚拟主机前提,取消主服务器,即注释掉物理主机站点根路径指定：

DocumentRoot "/var/www/htdocs/"

## 虚拟主机配置格式
Directory: 定义虚拟主机权限

VirualHost: 定义虚拟主机配置

## 基于端口方式
```
vim /etc/httpd/conf/httpd.conf
"""
Listen 80
Listen 8080  # 注意这里需要添加
#DocumentRoot "/var/www/html"  # 关闭物理主机网站根目录

# 添加:
<VirtualHost 172.10.0.1:80>
    ServerName www.xiaofei.com
    DocumentRoot "/web/hostA"
</VirtualHost>

<VirtualHost 172.10.0.1:8080>
    ServerName www.xiaofei.com
    DocumentRoot "/web/hostB"
</VirtualHost>
"""

mkdir -pv /web/host{A,B}
echo "host A" > /web/hostA/index.html
echo "host B" > /web/hostB/index.html
httpd -t
service httpd restart
```

测试访问
172.10.0.1:80 --> host A

172.10.0.1:8080 --> host B

## 基于IP方式
```
ifconfig eth0:0 172.10.0.2/16 
"""
<VirtualHost 172.10.0.1:80>
    ServerName www.xiaofei.com
    DocumentRoot "/web/hostA"
</VirtualHost>
<VirtualHost 172.10.0.2:80>
    ServerName www.xiaofei.com
    DocumentRoot "/web/hostB"
</VirtualHost>
"""

service httpd reload
```
测试访问172.10.0.1和172.10.0.2

## 基于主机名方式
```
vim /etc/httpd/conf/httpd.conf
"""
NameVirtualHost *:80     # 启用此选项
<VirtualHost *:80>
    ServerName www.xiaofei.com
    DocumentRoot "/web/hostA"
</VirtualHost>
<VirtualHost *:80>
    ServerName mail.xiaofei.com
    DocumentRoot "/web/hostB"
</VirtualHost>
<VirtualHost *:8080>
    ServerName web.xiaofei.com
    DocumentRoot "/web/hostC"
</VirtualHost>
```
因域名不同，所以需要修改hosts文件解析域名,把三个主机名全部指定到本机,推荐三种方式混用.


# 可能出现的问题

如果虚拟主机无法访问或访问被拒
检查主配置文件中[/etc/httpd/httpd.conf]
```
<Directory />
    AllowOverride none
    Require all denied  # 修改为granted
</Directory>
```
或者单独在httpd-vhost.conf中定义每个虚拟主机自己的权限
```
vim /etc/httpd/extra/httpd-vhosts.conf
"""
<Directory "/data/vhost/v1">
        Options Indexes MultiViews FollowSymLinks
        AllowOverride None
        Order allow,deny
        Allow from all
        # 注意,这里如果不定义,则会受到httpd配置文件中Require控制,它默认是deny所有的.
        Require all granted
</Directory>
<VirtualHost *:80>
        ServerName v1.com
        DocumentRoot "/data/vhosts/v1"
        ErrorLog "logs/v1_errors_log"
        CustomLog "logs/v1_access_log" combind
</VirtualHost>

<Directory "/data/vhosts/v2">
        Options Indexes MultiViews FollowSymLinks
        AllowOverride None
        Order allow,deny
        Allow from all
        Require all granted
</Directory>
<VirtualHost *:80>
        ServerName v2.com
        DocumentRoot "/data/vhosts/v2"
        ErrorLog "logs/v2_errors_log"
        CustomLog "logs/v2_access_log" combind
</VirtualHost>
"""
```

# 压缩传输
使用mod_deflate模块压缩页面，再传输，优化传输速度.
```
SetOutputFilter DEFLATE
# mod_deflate configuration
                # Restrict compression to these MIME types
AddOutputFilterByType DEFLATE text/plain
AddOutputFilterByType DEFLATE text/html
AddOutputFilterByType DEFLATE application/xhtml+xml
AddOutputFilterByType DEFLATE text/xml
AddOutputFilterByType DEFLATE application/xml
AddOutputFilterByType DEFLATE application/x-javascript
AddOutputFilterByType DEFLATE text/javascript
AddOutputFilterByType DEFLATE text/css
 
# Level of compression (Highest 9 - Lowest 1)
DeflateCompressionLevel 9
 
# Netscape 4.x has some problems.
BrowserMatch ^Mozilla/4 gzip-only-text/html
 
# Netscape 4.06-4.08 have some more problems
BrowserMatch ^Mozilla/4\.0[678] no-gzip
 
# MSIE masquerades as Netscape, but it is fine
BrowserMatch \bMSI[E] !no-gzip !gzip-only-text/html
```

# 启用服务器状态功能

==mod_status==模块可以让管理员查看服务器的执行状态，它通过一个HTML页面展示了当前服务器的统计数据。

这些数据通常包括但不限于：
- 处于工作状态的worker进程数；
- 空闲状态的worker进程数；
- 每个worker的状态，包括此worker已经响应的请求数，及由此worker发送的内容的字节数；
- 当前服务器总共发送的字节数；
- 服务器自上次启动或重启以来至当前的时长；
- 平均每秒钟响应的请求数、平均每秒钟发送的字节数、平均每个请求所请求内容的字节数；

启用状态页面的方法很简单，只需要在主配置文件中添加如下内容即可：
```
<Location /server-status>
   SetHandler server-status
   Require all granted
</Location></span>
```
需要提醒的是，这里的状态信息不应该被所有人随意访问，因此，应该限制仅允许某些特定地址的客户端查看。

比如使用Require ip 172.16.0.0/16来限制仅允许指定网段的主机查看此页面.

# End