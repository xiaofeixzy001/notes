[TOC]

# 硬件需求

![img](zabbix%E5%AE%89%E8%A3%85.assets/3d44699e-9729-4d8e-bbe4-d96d6d91ba01.jpg)



# 数据库

zabbix产生的数据主要由四部分组成：配置数据；历史数据，50Bytes历史趋势数据，128Bytes事件数据，130Bytes。

![image-20200323110117455](zabbix%E5%AE%89%E8%A3%85.assets/image-20200323110117455.png)



# WEB需求

Apache：1.3.12或者以上；

PHP：5.3.0或者以上；

zabbix早期版本支持5.2，但是2.2版本只支持到5.3；

PHP扩展：gd，bcmath，ctype，libXML2.6.15或以上，xmlreader，xmlwriter，session，sockets，mbstring，gettext，ibm_db2（可选），mysqli（推荐），oci8（可选），pgsql（可选），sqlite3(可选)。

![image-20200323110253205](zabbix%E5%AE%89%E8%A3%85.assets/image-20200323110253205.png)



# 服务端需求

![image-20200323110349407](zabbix%E5%AE%89%E8%A3%85.assets/image-20200323110349407.png)

以下内容都为可选项，如果你需要监控特定项，安装特定支持即可。

OpenIPMI：IPMI硬件监控

libssh2：版本1.0以上，监控ssh服务

fping：icmp监控项

libcurl：监控web项.

libiksemel：支持jabber报警

net-snmp：增加SNMP支持



# JAVA

如果你需要通过Java网关来监控你的Java进程，那么你需要增加如下支持

logback-core-0.9.27.jar ：http://logback.qos.ch/ ，0.9.27, 1.0.13, and 1.1.1已测试

logback-classic-0.9.27.jar ：http://logback.qos.ch/ ， 0.9.27, 1.0.13, and 1.1.1.已测试

slf4j-api-1.6.1.jar ：http://logback.qos.ch/ ，1.6.1, 1.6.6, and 1.7.6.已测试

android-json-4.3_r3.1.jar ：https://android.googlesource.com/platform/libcore/+/master/json ，2.3.3_r1.1 and 4.3_r3.1已测试



# 时间同步

请确保你所有的服务器时间都是正确的，为了确保时间ok，请在crontab里面加上定时时间同步

```shell
# crontab -l
00 00 * * * /usr/sbin/ntpdate -u ntp1.aliyun.com

# crontab -l
00 00 * * * /usr/sbin/ntpdate -u ntp1.aliyun.com
```



# 安装部署

下载地址：http://www.zabbix.com/download.php

安装方式可RPM包安装,可源码安装

![img](zabbix%E5%AE%89%E8%A3%85.assets/b5095ba0-f8e3-49c2-84ee-e76040d8faff.jpg)

![img](zabbix%E5%AE%89%E8%A3%85.assets/be085a8a-d2c2-46eb-919c-4feae685870d.jpg)



## 系统及软件版本

CentOS Linux release 7.7.1908 (Core)

## RPM包安装

1,安装http和mysql和php

```shell
# 安装http和mysql
[root@zabbix ~]# yum groupinstall "Development tools"
[root@zabbix ~]# yum install httpd mysql-community-server.x86_64 -y
[root@zabbix ~]# rpm -ql httpd
[root@zabbix ~]# rpm -ql mysql

# 启动服务
[root@zabbix ~]# service httpd start
[root@zabbix ~]# service mysqld start

# 测试
[root@zabbix ~]# vim /var/www/html/index.html
> Hello, World!
[root@zabbix ~]# curl http://node1.com

# 为zabbix建立数据库和用户
[root@zabbix ~]# mysql
> SHOW DATABASES;
> create database zabbix character set utf8 collate utf8_bin;
> GRANT ALL ON zabbix.* to 'zbxuser'@'172.16.100.%' IDENTIFIED BY 'zbxpass';
> GRANT ALL ON zabbix.* to 'zbxuser'@'node1.com' IDENTIFIED BY 'zbxpass';
> GRANT ALL ON zabbix.* to 'zbxuser'@'localhost' IDENTIFIED BY 'zbxpass';
> FLUSH PRIVILEGES;
> \q

# 测试
[root@zabbix ~]# mysql -uzbxuser -p
```

2,安装zabbix

下载地址：https://www.zabbix.com/download?zabbix=4.0&os_distribution=centos&os_version=7&db=MySQL

```shell
[root@zabbix ~]# groupadd zabbix
[root@zabbix ~]# useradd -g zabbix -s /sbin/nologin zabbix
[root@zabbix ~]# rpm -i https://repo.zabbix.com/zabbix/2.2/rhel/6/x86_64/zabbix-release-2.2-1.el6.noarch.rpm  # 安装zabbix的yum源
[root@zabbix ~]# yum clean all
[root@zabbix ~]# yum makecache

# server端安装:zabbix, zabbix-server, zabbix-server-mysql, zabbix-get,
# server端支持web: zabbix-web, zabbix-web-mysql, 
# server端支持监控自己: zabbix-agent, zabbix-sender
[root@zabbix ~]# yum install zabbix zabbix-server zabbix-server-mysql zabbix-web  zabbix-web-mysql zabbix-agent zabbix-get zabbix-sender

[root@zabbix ~]# rpm -ql zabbix
[root@zabbix ~]# ls /etc/zabbix
web  zabbix_agentd.conf  zabbix_agentd.d  zabbix_server.conf

[root@zabbix ~]# service httpd restart
[root@zabbix ~]# ls /etc/httpd/conf.d/
php.conf  README  welcome.conf  zabbix.conf  # 多了一个zabbix.conf

```

3, 配置zabbix-web

```shell
# 导入zabbix数据库
[root@zabbix create]# cd /usr/share/doc/zabbix-server-mysql-2.2.20/create
[root@zabbix create]# ls
data.sql  images.sql  schema.sql

# 注意导入次序
[root@zabbix create]# mysql zabbix < schema.sql
[root@zabbix create]# mysql zabbix < images.sql
[root@zabbix create]# mysql zabbix < data.sql

# 查看是否导入成功
[root@zabbix create]# mysql
> USE zabbix;
> SHOW TABLES;
> \q

# 如果导入出错,可删除zabbix数据库后重新建立导入
[root@zabbix create]# mysql
> DROP DATABASE zabbix;
> CREATE DATABASE zabbix CHARACTER SET utf8;

# 复制web目录到http的根目录下
rpm -ql zabbix-web | more
cp -r /usr/share/zabbix /var/www/html/
chown -R apache.apache /var/www/html/zabbix
ll
```

4, 配置zabbix-server.conf文件

```shell
[root@zabbix ~]# cp zabbix_server.conf{,.bak}
[root@zabbix ~]# vim zabbix-server.conf
"""
LogFile=/tmp/zabbix_server.log      # 日志位置
LogFileSize=0  # 日志文件大小,0无限制,超出指定大小自动滚动
DBHost=172.16.100.7 # 如果是本机通信, 修改为localhost
DBName=zabbix  # 数据库名
DBUser=zbxuser # 连接数据库用户名
DBPassword=zbxpass # 连接数据库密码
DBSocket=/tmp/mysql.sock # sock文件
StartPollers=30 # 开启多线程数，一般不要超过30个
StartTrappers=20 # trapper线程数
StartPingers=10 # fping线程数
StartDiscoverers=120            
MaxHousekeeperDelete=5000  # 最多一次删除多少个过期数据     
CacheSize=1024M # 用来保存监控数据的缓存数，根据监控主机的数量适当调整
StartDBSyncers=8 # 数据库同步时间
HistoryCacheSize=1024M          
TrendCacheSize=128M # 总趋势缓存大小
HistoryTextCacheSize=512M
AlertScriptsPath=/etc/zabbix/alertscripts
LogSlowQueries=1000
"""
```

5,配置php.ini

```shell
[root@zabbix ~]# cp /etc/php.ini /etc/php.ini.bak
[root@zabbix ~]# vim /etc/php.ini
date.timezone = Asia/Shanghai  # 时区必须修改
max_execution_time = 300
max_input_time = 300
post_max_size = 32M
memory_limit = 128M
mbstring.func_overload = 0
upload_max_filesize = 2M
always_populate_raw_post_data = -1

[root@zabbix ~]# vim /etc/httpd/conf.d/zabbix.conf
'''
php_value date.timezone Asia/Shanghai
'''
```

6, 配置监控端也监控自己

```shell
[root@zabbix ~]# vim /etc/zabbix/zabbix_agentd.conf
Server=127.0.0.1,172.16.100.7  # 允许谁来我这里获取数据
ServerActive=127.0.0.1,172.16.100.7  # 本机主动向谁发送数据
Hostname=zabbix-server
```

7, 测试访问http://node1.com/zabbix, 默认登录账号密码: admin/zabbix

```shell
[root@zabbix ~]# systemctl restart httpd
[root@zabbix ~]# systemctl start zabbix-server
[root@zabbix ~]# ss -tnl
[root@zabbix ~]# systemctl start zabbix-agent
```

8, 在agent端 node2.com 上安装zabbix-agent

```shell
[root@zabbix ~]# yum install zabbix-2.2.20-1.el6.x86_64.rpm zabbix-agent-2.2.20-1.el6.x86_64.rpm zabbix-sender-2.2.20-1.el6.x86_64.rpm -y
[root@zabbix ~]# vim /etc/zabbix/zabbix_agentd.conf
Server=172.16.100.7
ServerActive=172.16.100.7
Hostname=node2
```

8, 如果开启防火墙

```shell
# 开启zabbix要用的端口
[root@zabbix ~]# vim /etc/sysconfig/iptables  
-A INPUT -m state --state NEW -m udp -p udp --dport 10050 -j ACCEPT  
-A INPUT -m state --state NEW -m tcp -p tcp --dport 10050 -j ACCEPT  
-A INPUT -m state --state NEW -m udp -p udp --dport 10051 -j ACCEPT  
-A INPUT -m state --state NEW -m tcp -p tcp --dport 10051 -j ACCEPT  
[root@zabbix ~]# systemctl restart iptables 
```



## 编译安装

1, 监控端node1,准备部署环境

```shell
# 创建zabbix使用的用户和组
groupadd zabbix
useradd -g zabbix zabbix
# 注意:如果在一台主机上同时安装了server端和agent端,则建议其运行用户不要相同.

# 创建数据库: server端和proxy端的运行都依赖于数据库,agent端则不需要.
mysql

# 创建zabbix存储数据的数据库zabbix
mysql> CREATE DATABASE zabbix CHARACTER SET utf8 COLLATE utf8_bin;

# 授权数据库zabbix的访问权限
mysql> GRANT ALL ON zabbix.* TO zbxuser@'172.16.%.%' IDENTIFIED BY 'zbxpass';
mysql> GRANT ALL ON zabbix.* TO zbxuser@'node1.com' IDENTIFIED BY 'zbxpass';
mysql> GRANT ALL ON zabbix.* TO zbxuser@'localhost' IDENTIFIED BY 'zbxpass';

# 刷新
mysql> FLUSH PRIVILEGES;
mysql> \q

# 连接测试
mysql -uzbxuser -h172.16.100.7 -p
Enter Password: 
```

2, 监控端(zabbix-server), 一般监控端也要监控自己, 所以监控端也要安装agent

```shell
# 安装依赖包
yum install -y gcc make cmake php php-gd php-devel php-mysql php-bcmath php-ctytpe php-xml php-xmlreader php-xlmwriter php-session php-net-socket php-mbstring php-gettext httpd net-snmp curl curl-devel net-snmp net-snmp-devel perl-DBI
```

3, 编译安装zabbix

```shell
tar xf zabbix-2.2.20.tar.gz -C /usr/local/
cd /usr/local/zabbix-2.2.20

./configure --prefix=/usr/local/zabbix --with-mysql --with-net-snmp --with-libcurl --enable-server --enable-agent --enable-proxy

make && make install
```

\# 同时安装server和agent，并支持将数据放入mysql数据中，可使用类似如下配置命令：

./configure --enable-server --enable-agent --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-ssh2

\# 如果仅安装server，并支持将数据放入mysql数据中，可使用类似如下配置命令：

./configure --enable-server --with-mysql --with-net-snmp --with-libcurl

\# 如果仅安装proxy，并支持将数据放入mysql数据中，可使用类似如下配置命令：

./configure --prefix=/usr --enable-proxy --with-net-snmp --with-mysql --with-ssh2

\# 如果仅安装agent，可使用类似如下配置命令：

./configure --enable-agent

4，添加服务执行文件到/etc/init.d目录

```shell
cd /usr/local/zabbix-2.2.20
cp misc/init.d/fedora/core/zabbix_agentd /etc/init.d/
cp misc/init.d/fedora/core/zabbix_server /etc/init.d/
chmod +x /etc/init.d/zabbix*

vim /etc/init.d/zabbix_server
> BASEDIR=/usr/local/zabbix   # 这里修改为zabbix的安装目录
chkconfig --add zabbix_server
```

5, 复制zabbix-web页面文件到http服务的根目录下

```shell
mkdir /data
vim /data/index.html
> <h3>Hello,node1.</h3>

cp -r frontends/php /data/zabbix
chown -R apache.apache /data/zabbix
```

6, 配置php，支持zabbix

```shell
# 编辑php.ini,修改如下参数
cp /etc/php.ini /etc/php.ini.bak
vim /etc/php.ini
> date.timezone = Asia/Shanghai  # 时区必须修改
> max_execution_time = 300
> max_input_time = 300
> post_max_size = 16M
> memory_limit = 128M
> mbstring.func_overload = 0
> upload_max_filesize = 2M
> always_populate_raw_post_data = -1
```

6, 配置apache，这里使用虚拟主机

```shell
cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bak
vim /etc/httpd/conf/httpd.conf
> # DocumentRoot  "/var/www/html"  注释掉此行
vim /etc/httpd/conf.d/vhosts.conf
<VirtualHost 172.16.100.7:80>
     ServerName node1.com
     DocumentRoot "/data"
</VirtualHost>
```

7,编辑zabbix-server.conf配置文件，定义连接数据库信息

```shell
cd /usr/local/zabbix
cp etc/zabbix_server.conf /etc/zabbix_server.conf.bak
vim etc/zabbix_server.conf
"""
LogFile=/tmp/zabbix_server.log      # 日志位置
PidFile=/tmp/zabbix_server.pid     # PID所在位置
DBHost=172.16.100.7 # 如果是本机通信, 修改为localhost
DBName=zabbix  # 数据库名
DBUser=zbxuser # 连接数据库用户名
DBPassword=zbxpass # 连接数据库密码
DBSocket=/tmp/mysql.sock # sock文件
StartPollers=30 # 开启多线程数，一般不要超过30个
StartTrappers=20 # trapper线程数
StartPingers=10 # fping线程数
StartDiscoverers=120            
MaxHousekeeperDelete=5000       
CacheSize=1024M # 用来保存监控数据的缓存数，根据监控主机的数量适当调整
StartDBSyncers=8 # 数据库同步时间
HistoryCacheSize=1024M          
TrendCacheSize=128M # 总趋势缓存大小
HistoryTextCacheSize=512M
AlertScriptsPath=/etc/zabbix/alertscripts
LogSlowQueries=1000
"""
```

8，配置 zabbix-web

web页面启动前需要先导入zabbix自己的数据库到 mysql, 导入的数据库文件位置在zabbix源码包的dataabase/mysql目录下, 需倒序依次导入 schema.sql, images.sql, data.sql

```shell
cd zabbix-2.0.9/database/mysql

ls
"""
data.sql  images.sql  schema.sql
# 按照倒序依次导入
"""
mysql -uzbxuser -pzbxpass zabbix < schema.sql
mysql -uzbxuser -pzbxpass zabbix < images.sql
mysql -uzbxuser -pzbxpass zabbix < data.sql

mysql
mysql> USE zabbix;
mysql> SHOW TABLES;
mysql> \q
```

开启zabbix服务

```shell
service zabbix_server start
service zabbix_agent start
ss -tnlp
service httpd restart
```



## zabbix-web配置

浏览器访问：node1.com/zabbix  -> http://node1.com/zabbix/setup.php

![img](zabbix%E5%AE%89%E8%A3%85.assets/50b6b4f2-6902-4e0c-80a0-a86e3444c4f5.png)

点击Next step后，会检查相关的条件是否满足，对于不满足的要进行修改。主要修改下面内容： 根据提示修改 php.ini的配置，有可能还要安装php的扩展包等，这块挺磨人的，但是网上文章比较多，可以参考。修改完成后，都需要 重启httpd服务，和zabbix server，然后重新打开浏览器查看。如果都正确了，会出现下面的界面：

![img](zabbix%E5%AE%89%E8%A3%85.assets/65a235ad-8b21-4308-9ed4-35fc769131f4.png)
点击next step后，进入数据库配置界面，填写zabbix数据库的用户名、密码、数据库地址等信息，OK后点击Next

![img](zabbix%E5%AE%89%E8%A3%85.assets/11987136-5182-47b0-9b4b-504216f4bbc8.png)

填写zabbix服务器的信息,host是主机名或IP，port是监听的的端口，Name 是可选项，可以不填，默认即可，点击Next

![img](zabbix%E5%AE%89%E8%A3%85.assets/870c4cb6-895e-4c4f-9e10-61850418223c.png)

如果一切正常，就会提示你安装成功，在安装成功界面点击OK，会直接跳转到登录界面

![img](zabbix%E5%AE%89%E8%A3%85.assets/e27126dee9ca836c2445d35e24851718.png)

默认的登录名和密码是Admin/zabbix

登陆后页面如下：

![img](zabbix%E5%AE%89%E8%A3%85.assets/6134b528-fa88-48ed-9b3b-2be45dda9e9b.png)

9，配置agent，以便让自己监控自己

```shell
cd /usr/local/zabbix/etc/
vim zabbix_agentd.conf
"""
Server=172.16.100.7  # 允许哪个主机来采集本机信息
ListenPort=10050
StartAgents=3
ServerActive=172.16.100.7  # 定义本机是主动监控还是被动监控
"""
```

10, 在另一台主机 node2 上安装 zabbix-agent ,作为被监控端

```shell
tar xf zabbix-2.2.20.tar.gz -C /usr/local/
cd /usr/local/zabbix-2.2.20
./configure --prefix=/usr/local/zabbix --enable-agent
make && make install

cd /usr/local/zabbix/etc/
vim zabbix_agentd.conf
"""
Server=172.16.100.7
ListenPort=10050
StartAgents=3
ServerActive=172.16.100.7
"""
```

注意： 

如果启动zabbix失败，需要从下面方向解决：

1，php 安装是否正确，含版本（3.2版本的zabbix 需要php 5.X以上）

2，其次，可以查看zabbix的日志，一般在/tmp目录下，如果是连接mysql太多导致无法启动，修改相关的zabbix连接mysql参数；

小技巧：

如果是通过yum方式安装的，一般是5.3.3，但是打开 setup.php 无法打开。可以先检查http服务是否正确。http://ip 如果可以看到正确的apache页面，说明http服务正确。否则检查http服务是否启动，还有是否关闭Selinux和关闭的防火墙。

如果还无法打开setup.php，可以用命令行，登录到 /var/www/html/zabbix目录，执行php setup.php 如果报告不支持“【”，那么是php版本还不够高 请安装5.6.27 这个版本，我是验证可以的。