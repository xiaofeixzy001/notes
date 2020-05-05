[TOC]

简介

工作原理

脑裂

HeartBeat的作用：

通过HeartBeat，可以将资源（IP以及程序服务等资源）从一台已经故障的计算机快速转移到另一台正常运转的机器上继续提供服务，一般称之为高可用的服务。在实际的生产应用场景中，heartbeat的功能和另一个高可用的开源软件keepalived有很多的相同之处，在我们实际的生产业务中也是有区别的。

HeartBeat的工作原理：

通过修改Heartbeat的软件的配置文件，可以制定那一台Heartbeat服务器作为主服务器，则另一台将自动成为热备服务器。

然后在热备服务器上配置Heartbeat.

守护程序来监听来自主服务器的心跳消息。如果热备服务器在指定时间内为监听到来自主服务器的心跳，就会启动故障转义程

序，并取得主服务器上的相关资源服务的所有权，接替主服务器继续不间断的提供服务，从而达到资源以及服务高可用的目的。

以上的描述heartbeat的主备模式，heartbeat还支持主主模式，即两台服务器互为主备，这是他们之间还会互相发送报文来

告诉对方自己的当前的状态，如果在指定的时间内未收到对方发送的心跳报文，那么，一方就会认为对方失效或者是已经宕机

了，这时每个运行正常的主机就会启动自身的资源接管模块来接管运行在对方主机上的资源或者是服务，继续为用户提供服务。

一般情况下，可以较好的实现一台主机故障后，企业业务能够不间断的持续的提供服务。注意：所谓的业务不间断，在故障转移

期间也是需要切换时间的，heartbeat的切换时间是5-20秒。

切换的常见条件：

1）服务器宕机

2）Heartbeat服务本故障

3）中间的连接线路故障

应用服务故障则不会产生切换，可以通过服务宕机把heartbeat服务停掉。

heartbeat的心跳连接：

讲过上面的描述，要部署heartbeat服务，至少需要两台主机才能完成。那么，要实现高可用服务，这两台主机之间，是如何做

到互相通信互相监控的呢/

下面是两台heartbeat主机之间通信的一些常用的可行的方法：

1）串行电缆，即所谓的串口（首选，缺点是距离不能太远）

2）一根以太网电缆量网口直连（生产环境中常用的方式）

3）以太网电缆，通过交换机等网络设备连接（次选，原因是增加了故障点，不好排查故障，同时，线路不是专用的心跳线，容易

受其他数据传输的影响，导致心跳报文发送问题）

Heartbeat裂脑：

  什么是裂脑？

　　由于两台高可用服务器之间在指定的时间内，无法互相检测到对方心跳而各自启动故障转移功能，取得了资源以及服务的所有权，而此时的两台高可用服务器

对都还活着并作正常运行，这样就会导致同一个IP湖综合服务在两端同时启动而发生冲突的严重问题，最严重的就是两台主机同时占用一个VIP的地址，当用户写入数

据的时候可能会分别写入到两端，这样可能会导致服务器两端的数据不一致或造成数据的丢失，这种情况就本成为裂脑，也有的人称之为分区集群或者大脑垂直分隔

导致裂脑发生的原因：　　

　　一般来说，裂脑的发生，主要是由以下的几个原因导致的：

　　1）高可用服务器对之间心跳线路故障，导致无法正常的通信。原因比如：

　　　　（1）.心跳线本身就坏了（包括断了，老化）

　　　　（2）.网卡以及相关驱动坏了,IP配置及冲突问题

　　　　（3）.心跳线间连接的设备故障（交换机的故障或者是网卡的故障）

　　　　（4）.仲裁的服务器出现问题

　　2）高可用服务器对上开启了防火墙阻挡了心跳消息的传输

　　3）高可用服务器对上的心跳网卡地址等信息配置的不正确，导致发送心跳失败。

　　4）其他服务配置不当等原因，如心跳的方式不同，心跳广播冲突，软件出现了BUG等

防止脑裂发生的方法总结：

　　发生脑裂的时候，对业务的影响是及其严重的，有的时候甚至是致命的。如：两台高可用的服务器对之间发生脑裂，导致互相竞争同一个IP资源，就如同我们局域

网内常见的IP地址冲突一样，两个机器就会有一个或者两个不正常，影响用户正常访问服务器。如果是应用在数据库或者是存储服务这种极重要的高可用上，那就导致

用户发布的数据间断的写在两台服务器上的恶果，最终数据恢复及困难或者是难已恢复

　　实际的生产环境中，我们可以从以下几个方面来防止裂脑的发生：

　　1）同时使用串行电缆和以太网电缆连接，同时用两条心跳线路，这样一条线路坏了，另一个线路还是好的，依然能传送消息（推荐的）

　　2）检测到裂脑的时候强行的关闭一个心跳节点（需要特殊的节点支持，如stonith，fence），相当于程序上备节点发现心跳线故障，发送关机命令到主节点。

　　3）做好对裂脑的监控报警（如邮件以及手机短信等），在问题发生的时候能够人为的介入到仲裁，降低损失。当然，在实施高可用方案的时候，要根据业务的实际需求确定是否能够容忍这样的损失。对于一般的网站业务，这个损失是可控的（公司使用）

　　4）启用磁盘锁。正在服务一方锁住共享磁盘，脑裂发生的时候，让对方完全抢不走共享的磁盘资源。但使用锁磁盘也会有一个不小的问题，如果占用共享盘的乙方不主动解锁，另一方就永远得不到共享磁盘。现实中介入服务节点突然死机或者崩溃，另一方就永远不可能执行解锁命令。后备节点也就截关不了共享的资源和应用服务。于是有人在HA中涉及了“智能”锁，正在服务的一方只在发现心跳线全部断开时才启用磁盘锁，平时就不上锁了

　　5）报警报在服务器接管之前，给人员处理留足够的时间就是1分钟内报警了，但是服务器不接管，而是5分钟之后接管，接管的时间较长。数据不会丢失，但就是会导致用户无法写数据。

　　6）报警后，不直接自动服务器接管，而是由人员接管。

　　7）增加仲裁的机制，确定谁该获得资源，这里面有几个参考的思路：

　　　　1）增加一个仲裁机制。例如设置参考的IP，当心跳完全断开的时候，2个节点各自都ping一下参考的IP，不同则表明断点就出现在本段，这样就主动放弃竞争，让能够ping通参考IP的一端去接管服务。

　　　　2）通过第三方软件仲裁谁该获得资源，这个在阿里有类似的软件应用

HeartBeat的消息类型：

　　heartBeat高可用软件在工作的过程中，一般来说，有三种消息的类型，具体为：

　　1）心跳消息

　　　　心跳消息为约150字节的数据包，可能为单播，广播或者多播的方式，控制心跳频率以及出现故障要等待多久进行故障转换

　　2）集群转换消息

　　　　当主服务器恢复在线状态时，通过ip-request消息是要求备机释放主服务器失败时被服务器取得的的资源，然后被服务器关闭是仿主服务器失败时取得的资源以及服务。

　　　　备服务器释放主服务器失败时取得的资源以及服务后，就会通过ip-request-resp消息通知主服务器它不在拥有该资源以及服务，主服务器收到来自备节点的ip-request-resp消息通知后，启动失败时释放的资源以及服务，并开始提供正常的访问服务。

　　3）重传消息请求

　　　　rexmit-request控制重传心跳请求。此消息不太重要，细节就不多介绍了

提示：以上的心跳控制消息都使用的是UDP协议发送到/etc/ha.d/ha.cf文件指定到任意的端口，或者指定到多播地址。

Heartbeat ip地址接管和故障转移：

　　Heartbeat是通过IP地址接管和ARP广播进行故障转移的。

　　ARP广播：在主服务器故障的时候，备用节点接管资源后，会强制更新所有的客户端本地的ARP表（即清除客户端本地缓存的失败服务器的VIP地址和mac地址的

　　解析记录）。确保客户端和新的主服务器进行对话。

（这提到的客户端机器是和Heartbeat高可用服务器对在同一个网络中的客户机，并不是最终的互联网用户，这里的客户端及其是相对Heartbeat高可用服务器对说的，这点，请注意下）

VIP/IP 别名/辅助IP：

真实IP

又被称为管理IP，一般是配置在物理网卡上的实际IP，这可以看做是你本人的真实姓名，如：张三。在负载均衡以及高可用环境中，管理IP是不会对外提供用户的访问服务的，而是仅作管理服务器使用，如ssh可以通过这个管理IP连接服务器　　　　

VIP是虚拟的IP

只是个概念而已，可能会误导，实际上就是Heartbeat临时绑在物理网卡上的别名IP，如eth0：x，x为0-255的任意数字，可以在一块网卡上绑定多个别名，这样做的好处是当提供服务的服务器宕机之后，在接管的服务器上会直接会自动配置上同样的VIP提供服务。如果使用管理IP的话，来回迁移就难以做到，而且，管理IP迁移过去了我们就不能够登录到这台机器上，这就需要到机房登陆了。VIP的实质就是确保两台服务器有一个管理IP不懂，就是随时可以连上机器，然后，增加绑定其他的VIP，这样就算VIP转移走了，也不至于服务器本身连不上，因为还有管理的IP呢

手工配置VIP的方法：

　　　　ifconfig eth0:1 124.42.61.109 netmask  255.255.255.224 up（ip alias） --》heartbeat2软件默认是使用这个命令来添加VIP的

　　　　ip addr add 10.0.15.1/24 broadcast  10.0.15.255 dev eth1（辅助Ip）--》keepalived以及heartbeat3采用的方案添加VIP的

注意：使用ip addr能够查看到包括别名和辅助IP，用ifconfig无法查到辅助IP的配置情况

手工删除VIP的方法：

　　　　ip addr del 10.0.15.1/24 broadcast 10.0.15.255 dev eth1（辅助IP）

　　　　ifconfig eth0:1 124.42.61.109 netmask 255.255.255.244 down(ip alias)

HeartBeat配置文件：

　　　  heartbeat的默认配置文件的目录为/etc/ha.d heartbeat的常用配置文件有三个，分别为ha.cf,authkey,haresource.

　　　　ha.cf   heartbeat参数配置文件    在这里配置一些基本的参数

　　　　authkey  heartbeat认证文件     高可用服务器对之间根据对端的authkey，对对端的进行认证

　　　　haresource   heartbeat的资源文件   如配置资源以及一些脚本程序

重要资源目录：/etc/ha.d/resource.d/,如果以后自己开发程序，就放到这个地方即可，然后在haresource文件里直接调用

ha.cf配置文件部分参数详解：

autojoin none     #集群中的节点不会自动加入

logfile /var/log/ha-log  #指名heartbaet的日志存放位置

keepalive 2     #指定心跳使用间隔时间为2秒（即每两秒钟在eth1上发送一次广播）

deadtime 30     #指定备用节点在30秒内没有收到主节点的心跳信号后，则立即接管主节点的服务资源

warntime 10     #指定心跳延迟的时间为十秒。当10秒钟内备份节点不能接收到主节点的心跳信号时，就会往日志中写入一个警告日志，但此时不会切换服务

initdead 120    #在某些系统上，系统启动或重启之后需要经过一段时间网络才能正常工作，该选项用于解决这种情况产生的时间间隔。取值至少为deadtime的两倍。

​			   

udpport 694     #设置广播通信使用的端口，694为默认使用的端口号。

baud   19200     #设置串行通信的波特率 

   

bcast  eth0     # Linux  指明心跳使用以太网广播方式，并且是在eth0接口上进行广播。

\#mcast eth0 225.0.0.1 694 1 0     

\#采用网卡eth0的Udp多播来组织心跳，一般在备用节点不止一台时使用。Bcast、ucast和mcast分别代表广播、单播和多播，是组织心跳的三种方式，任选其一即可。

\#ucast eth0 192.168.1.2   #采用网卡eth0的udp单播来组织心跳，后面跟的IP地址应为双机对方的IP地址

auto_failback on     

\#用来定义当主节点恢复后，是否将服务自动切回，heartbeat的两台主机分别为主节点和备份节点。主节点在正常情况下占用资源并运行所有的服务，遇到故障时把资源交给备份节点并由备份节点运行服务。在该选项设为on的情况下，一旦主节点恢复运行，则自动获取资源并取代备份节点，如果该选项设置为off，那么当主节点恢复后，将变为备份节点，而原来的备份节点成为主节点

\#stonith baytech /etc/ha.d/conf/stonith.baytech     

\# stonith的主要作用是使出现问题的节点从集群环境中脱离，进而释放集群资源，避免两个节点争用一个资源的情形发生。保证共享数据的安全性和完整性。

\#watchdog /dev/watchdog    

\#该选项是可选配置，是通过Heartbeat来监控系统的运行状态。使用该特性，需要在内核中载入"softdog"内核模块，用来生成实际的设备文件，如果系统中没有这个内核模块，就需要指定此模块，重新编译内核。编译完成输入"insmod softdog"加载该模块。然后输入"grep misc /proc/devices"(应为10)，输入"cat /proc/misc |grep watchdog"(应为130)。最后，生成设备文件："mknod /dev/watchdog c 10 130" 。即可使用此功能

node node1.magedu.com      #主节点主机名，可以通过命令“uname –n”查看。

node node2.magedu.com      #备用节点主机名

ping 192.168.100.7     

\#选择ping的节点，ping节点选择的越好，HA集群就越强壮，可以选择固定的路由器作为ping节点，但是最好不要选择集群中的成员作为ping节点，ping节点仅仅用来测试网络连接

\#ping_group group1 192.168.12.120 192.168.12.237     #类似于ping  ping一组ip地址

apiauth pingd  gid=haclient uid=hacluster

respawn hacluster /usr/local/ha/lib/heartbeat/pingd -m 100 -d 5s     

\#该选项是可选配置，列出与heartbeat一起启动和关闭的进程，该进程一般是和heartbeat集成的插件，这些进程遇到故障可以自动重新启动。最常用的进程是pingd，此进程用于检测和监控网卡状态，需要配合ping语句指定的ping node来检测网络的连通性。其中hacluster表示启动pingd进程的身份。

​			

\#下面的配置是关键，也就是激活crm管理，开始使用v2 style格式

\# crm respawn

\#注意，还可以使用crm yes的写法，但这样写的话，如果后面的cib.xml配置有问题

\#会导致heartbeat直接重启该服务器，所以，测试时建议使用respawn的写法

\#下面是对传输的数据进行压缩，是可选项

compression   bz2

compression_threshold 2(K)

注意，v2 style不支持ipfail功能，须使用pingd代替

资源文件(/etc/ha.d/haresources)

node1  IPaddr::192.168.1.100/24/eth0  httpd

认证文件(/etc/ha.d/authkeys)

auth 1

1 crc

实验案例：

heartbeat + httpd实现HA热备(以两节点为例)

通过heartbeat验证HA的工作特性，以2节点为例，各节点分别提供httpd服务，通过配置heartbeat来为httpd服务提供高可用

node1:192.168.100.7,node1.xiaofei.com

node2:192.168.100.8,node2.xiaofei.com

1,node1节点配置

 

```
//修改主机名
# vim /etc/sysconfig/network
hostname=node1.xiaofei.com
# vim /etc/hosts
192.168.100.7 node1.xiaofei.com node1
192.168.100.8 node2.xiaofei.com node2
# uname -n
//配置时间
# ntpdate 192.168.100.254
# crontab -e
*/5 * * * * /usr/sbin/ntpdate 192.168.100.254 &> /dev/null
# date
//安装http服务
# yum install httpd -y
# echo "<h1>node1</h1>" > /var/www/html/index.html
# service httpd start
# curl node1.xiaofei.com
# curl node2.xiaofei.com
//配置基于ssh密钥认证免密登录
# ssh-keygen -t rsa -P ''        ##这里为了方便密钥为空
# ssh-copy-id -i ~/.ssh/id_rsa.pub root@node2
node2 password:


```

 

```
//修改主机名
# vim /etc/sysconfig/network
hostname=node1.xiaofei.com
# vim /etc/hosts
192.168.100.7 node1.xiaofei.com node1
192.168.100.8 node2.xiaofei.com node2
# uname -n
//配置时间
# ntpdate 192.168.100.254
# crontab -e
*/5 * * * * /usr/sbin/ntpdate 192.168.100.254 &> /dev/null
# date
//安装http服务
# yum install httpd -y
# echo "<h1>node1</h1>" > /var/www/html/index.html
# service httpd start
# curl node1.xiaofei.com
# curl node2.xiaofei.com
//配置基于ssh密钥认证免密登录
# ssh-keygen -t rsa -P ''        ##这里为了方便密钥为空
# ssh-copy-id -i ~/.ssh/id_rsa.pub root@node2
node2 password:
```

2,node2节点配置

 

```
//修改主机名
# vim /etc/sysconfig/network
hostname=node1.xiaofei.com
# vim /etc/hosts
192.168.100.7 node1.xiaofei.com node1
192.168.100.8 node2.xiaofei.com node2
# uname -n
//修改时间
# ntpdate 192.168.100.254
# crontab -e
*/5 * * * * /usr/sbin/ntpdate 192.168.100.254 &> /dev/null
# date
//安装http服务
# yum install httpd -y
# echo "<h1>node2</h1>" > /var/www/html/index.html
# service httpd start
# curl node1.xiaofei.com
# curl node2.xiaofei.com
//配置基于ssh密钥认证免密登录
# ssh-keygen -t rsa -P ''       //这里为了方便密钥为空
# ssh-copy-id -i ~/.ssh/id_rsa.pub root@node1
node1 password:
# scp ssh_keygen root@node2:/root/.ssh/
在node2上操作
# cat .ssh/ssh_keygen >> .ssh/authorized_key

```

 

```
//修改主机名
# vim /etc/sysconfig/network
hostname=node1.xiaofei.com
# vim /etc/hosts
192.168.100.7 node1.xiaofei.com node1
192.168.100.8 node2.xiaofei.com node2
# uname -n
//修改时间
# ntpdate 192.168.100.254
# crontab -e
*/5 * * * * /usr/sbin/ntpdate 192.168.100.254 &> /dev/null
# date
//安装http服务
# yum install httpd -y
# echo "<h1>node2</h1>" > /var/www/html/index.html
# service httpd start
# curl node1.xiaofei.com
# curl node2.xiaofei.com
//配置基于ssh密钥认证免密登录
# ssh-keygen -t rsa -P ''       //这里为了方便密钥为空
# ssh-copy-id -i ~/.ssh/id_rsa.pub root@node1
node1 password:
# scp ssh_keygen root@node2:/root/.ssh/
在node2上操作
# cat .ssh/ssh_keygen >> .ssh/authorized_key
```

3，测试验证

 

```
# ping node1.xiaofei.com
# ping node2.xiaofei.com
# curl node1.xiaofei.com
# curl node2.xiaofei.com
# ssh node2 'date';date
# ssh node1 'date';date
// 测试完成后，关闭httpd服务和开机启动
# service httpd stop
# chkconfig httpd off
```

 

```
# ping node1.xiaofei.com
# ping node2.xiaofei.com
# curl node1.xiaofei.com
# curl node2.xiaofei.com
# ssh node2 'date';date
# ssh node1 'date';date
// 测试完成后，关闭httpd服务和开机启动
# service httpd stop
# chkconfig httpd off
```

4，安装heartbeat

2节点分别安装依赖的包和heartbeat包

 

```
# yum install perl-TimeDate PyXML gettext gnutls libtool-ltdl -y
# rpm -ivh libnet-1.1.6-7.el6.x86_64.rpm
# yum install net-snmp-libs -y
# rpm -ivh heartbeat-pils-2.1.4-12.el6.x86_64.rpm
# rpm -ivh heartbeat-stonith-2.1.4-12.el6.x86_64.rpm
# rpm -ivh heartbeat-2.1.4-12.el6.x86_64.rpm
```

 

```
# yum install perl-TimeDate PyXML gettext gnutls libtool-ltdl -y
# rpm -ivh libnet-1.1.6-7.el6.x86_64.rpm
# yum install net-snmp-libs -y
# rpm -ivh heartbeat-pils-2.1.4-12.el6.x86_64.rpm
# rpm -ivh heartbeat-stonith-2.1.4-12.el6.x86_64.rpm
# rpm -ivh heartbeat-2.1.4-12.el6.x86_64.rpm
```

5，heartbeat配置

 

```
//配置文件主要有3个：
//ha.cf(主配置文件)
//authkeys(认证密钥文件，权限必须为600)
//haresources(资源配置)
///usr/share/doc/heartbeat-2.1.4/目录下有模版
# cd /etc/ha.d
# cp /usr/share/doc/heartbeat-2.1.4/{authkeys,haresources,ha.cf} /etc/ha.d/
# cd /etc/ha.d/ 
# chmod 600 authkeys
# ll
//生成一串随机码当作认证的密钥
# openssl rand -hex 8
0576e27920f46d92
//配置密钥认证文件
# vim authkeys
auth 2
2 sha1 0576e27920f46d92
//配置ha.cf配置文件
# grep -v '#' ha.cf 
logfile /var/log/ha-log
mcast eth0 225.100.0.1 694 1 0
node    node1.xiaofei.com
node    node2.xiaofei.com
ping 192.168.100.254
//配置haresources资源管理文件
# vim haresources
node1.xiaofei.com 192.168.100.100/24/eth0/192.168.100.255 httpd
//复制到node2节点上去
# scp -p ha.cf haresources authkeys node2:/etc/ha.d/
//启动heartbeat服务，查看694端口是否被监听，VIP地址192.168.100.1是否启用，80端口是否被开启
# service heartbeat start
# ss -unl
# ifconfig 
# ss -tnl
```

 

```
//配置文件主要有3个：
//ha.cf(主配置文件)
//authkeys(认证密钥文件，权限必须为600)
//haresources(资源配置)
///usr/share/doc/heartbeat-2.1.4/目录下有模版
# cd /etc/ha.d
# cp /usr/share/doc/heartbeat-2.1.4/{authkeys,haresources,ha.cf} /etc/ha.d/
# cd /etc/ha.d/ 
# chmod 600 authkeys
# ll
//生成一串随机码当作认证的密钥
# openssl rand -hex 8
0576e27920f46d92
//配置密钥认证文件
# vim authkeys
auth 2
2 sha1 0576e27920f46d92
//配置ha.cf配置文件
# grep -v '#' ha.cf 
logfile /var/log/ha-log
mcast eth0 225.100.0.1 694 1 0
node    node1.xiaofei.com
node    node2.xiaofei.com
ping 192.168.100.254
//配置haresources资源管理文件
# vim haresources
node1.xiaofei.com 192.168.100.100/24/eth0/192.168.100.255 httpd
//复制到node2节点上去
# scp -p ha.cf haresources authkeys node2:/etc/ha.d/
//启动heartbeat服务，查看694端口是否被监听，VIP地址192.168.100.1是否启用，80端口是否被开启
# service heartbeat start
# ss -unl
# ifconfig 
# ss -tnl
```

至此，配置已经完成，可以测试访问：http:192.168.100.1，显示node1；然后让node1下线，再次测试，显示node2，再次让node1上线，则又会显示回了node1

缺点：无法检测后端RS的健康状态

**heartbeat配置基于NFS文件系统的HA双机热备方案：**

目的：

由于已安装的heartbeat方案，访问的httpd资源都存储到了本地，如果node1下线后，存储到node1上的资源将不可访问，所以需要让存储到2个节点上的数据实时保持一致，NFS文件系统仅为了验证原理，不适用于生产环境。

要求：

使用NFS共享文件存储方案，分别挂载到2个节点上来实现。

NFS共享文件：/www/htdocs

NFS server：192.168.100.9

NFS Name：www.node3.com

1，在nfs节点上配置nfs及共享目录

 

```
# vim /etc/sysconfig/network
hostname=nfs.xiaofei.com
# vim /etc/hosts
192.168.100.9 nfs.xiaofei.com nfs
# mkdir -pv /data/html
# vim /data/html/index.html
<h1>Shared Page</h1>
//共享此目录
# vim /etc/exports
/data/html 192.168.100.0/24(rw,async，no_root_squash)
# exportfs -arv
# service nfs start
# chkconfig nfs on
```

 

```
# vim /etc/sysconfig/network
hostname=nfs.xiaofei.com
# vim /etc/hosts
192.168.100.9 nfs.xiaofei.com nfs
# mkdir -pv /data/html
# vim /data/html/index.html
<h1>Shared Page</h1>
//共享此目录
# vim /etc/exports
/data/html 192.168.100.0/24(rw,async，no_root_squash)
# exportfs -arv
# service nfs start
# chkconfig nfs on
```

2,各节点挂载nfs，需要注意的是各节点要已安装nfs服务

node1：

 

```
# mount -t nfs 192.168.100.9:/data/html /var/www/html
# mount 
# cat /var/www/html/index.html
<h1>Shared Page</h1>
# umount /var/www/html
```

 

```
# mount -t nfs 192.168.100.9:/data/html /var/www/html
# mount 
# cat /var/www/html/index.html
<h1>Shared Page</h1>
# umount /var/www/html
```

node2：

 

```
# mount -t nfs 192.168.100.9:/data/html /var/www/html
# mount 
# cat /var/www/html/index.html
<h1>Shared Page</h1>
# umount /var/www/html
```

 

```
# mount -t nfs 192.168.100.9:/data/html /var/www/html
# mount 
# cat /var/www/html/index.html
<h1>Shared Page</h1>
# umount /var/www/html
```

3，配置heartbeat资源配置文件haresources

 

```
//资源代理的配置类型可在/etc/ha.d/resource.d/目录下查看
# vim /etc/ha.d/haresources
node1.xiaofei.com 192.168.100.1/24/eth0 Filesystem::192.168.100.9:/data/html::/var/www/html::nfs httpd
# scp /etc/ha.d/haresources node2:/etc/ha.d/
# service heartbeat start
# ssh node2 'service heartbeat start'
```

 

```
//资源代理的配置类型可在/etc/ha.d/resource.d/目录下查看
# vim /etc/ha.d/haresources
node1.xiaofei.com 192.168.100.1/24/eth0 Filesystem::192.168.100.9:/data/html::/var/www/html::nfs httpd
# scp /etc/ha.d/haresources node2:/etc/ha.d/
# service heartbeat start
# ssh node2 'service heartbeat start'
```

4,访问http://192.168.100.1测试，显示Shared Page

node1下线，依旧可正常访问，node1上线，实现自动切换

需要注意的：

在实际生产过程中，如需测试，一般不用stop来停止服务，而是在/etc/lib64/heartbeat/目录下，有很多程序脚本，如hb_standby,hb_takeover

\# ./hb_standby：让本机切换成备节点，释放所有已挂载的资源

\# ./hb_takeover：切换成主节点，取回资源

\# ./ha_propagate：通告，自动把自己的配置文件ha.cf和authkeys传送到其他节点上去，除了haresources

\# ./sed_arp：通告前端路由器，主节点的mac是否发生改变

**基于heartbeat v1配置mysql的基于nfs共享数据的高可用集群**

在nfs节点上，安装mysql，数据库存储位置为/data/mysql

 

```
//安装依赖包
# yum groupinstall "Development tools" "Server Platform Development" -y  
# yum install pcre* -y
# yum install gcc gcc-c++ -y
# yum install ncurses-devel -y
# yum install libxml2-devel -y
//安装cmake
# tar xf cmake-2.8.8.tar.gz 
# cd cmake-2.8.8    
# ./bootstrap 
# make
# make install
//创建mysql数据目录
# groupadd -r mysql 
# useradd -g mysql -r mysql 
# mkdir -pv /data/mysql 
# chown -R mysql:mysql /data/mysql
//编译安装MariaDB
# tar zxf mariadb-10.0.12.tar.gz
# cd mariadb-10.0.12
# cmake -LH   ##查看cmake默认的编译特性
# cmake . -DMYSQL_DATADIR=/data/mysql/ -DWITH_SPHINX_STORAGE_ENGINE=1 -DWITH_SSL=system
# make && make install
# cp support-files/mysql.server /etc/rc.d/init.d/mysqld
# chmod +x /etc/rc.d/init.d/mysqld
# cp support-files/my-large.cnf /etc/my.cnf
# vim /etc/my.cnf    ##添加一项：
 datadir = /data/mysql
# cd /usr/local/mysql
# scripts/mysql_install_db --user=mysql --datadir=/data/mysql/
# service mysqld start
# ss -tnl
# vim /etc/profile.d/mysql.sh
export PATH=/usr/local/mysql/bin/:$PATH
# . /etc/profile.d/mysql.sh
# mysql
> SHOW ENGINES;
> quit;
//创建mysql远程连接的用户并授予权限
# mysql -uroot -p
password:
> CREATE USER 'tom'@'%' IDENTIFIED BY '123.com';
> GRANT ALL ON mydb.* TO 'tom'@'%';
> flush privileges;
//通过NFS共享mysql数据目录
# vim /etc/exports
/data/mysql 192.168.100.0/24(rw,async,no_root_squash)
# exportfs -arv
# service nfs restart
```

 

```
//安装依赖包
# yum groupinstall "Development tools" "Server Platform Development" -y  
# yum install pcre* -y
# yum install gcc gcc-c++ -y
# yum install ncurses-devel -y
# yum install libxml2-devel -y
//安装cmake
# tar xf cmake-2.8.8.tar.gz 
# cd cmake-2.8.8    
# ./bootstrap 
# make
# make install
//创建mysql数据目录
# groupadd -r mysql 
# useradd -g mysql -r mysql 
# mkdir -pv /data/mysql 
# chown -R mysql:mysql /data/mysql
//编译安装MariaDB
# tar zxf mariadb-10.0.12.tar.gz
# cd mariadb-10.0.12
# cmake -LH   ##查看cmake默认的编译特性
# cmake . -DMYSQL_DATADIR=/data/mysql/ -DWITH_SPHINX_STORAGE_ENGINE=1 -DWITH_SSL=system
# make && make install
# cp support-files/mysql.server /etc/rc.d/init.d/mysqld
# chmod +x /etc/rc.d/init.d/mysqld
# cp support-files/my-large.cnf /etc/my.cnf
# vim /etc/my.cnf    ##添加一项：
 datadir = /data/mysql
# cd /usr/local/mysql
# scripts/mysql_install_db --user=mysql --datadir=/data/mysql/
# service mysqld start
# ss -tnl
# vim /etc/profile.d/mysql.sh
export PATH=/usr/local/mysql/bin/:$PATH
# . /etc/profile.d/mysql.sh
# mysql
> SHOW ENGINES;
> quit;
//创建mysql远程连接的用户并授予权限
# mysql -uroot -p
password:
> CREATE USER 'tom'@'%' IDENTIFIED BY '123.com';
> GRANT ALL ON mydb.* TO 'tom'@'%';
> flush privileges;
//通过NFS共享mysql数据目录
# vim /etc/exports
/data/mysql 192.168.100.0/24(rw,async,no_root_squash)
# exportfs -arv
# service nfs restart
```

配置heartbeat资源配置文件haresources

 

```
# vim /etc/ha.d/haresources
node1.xiaofei.com 192.168.100.1/24/eth0 Filesystem::192.168.100.9:/data/mysql::/mydata/mysql::nfs mysql
# scp /etc/ha.d/haresources node2:/etc/ha.d/
# service heartbeat start
# ssh node2 'service heartbeat start'
```

 

```
# vim /etc/ha.d/haresources
node1.xiaofei.com 192.168.100.1/24/eth0 Filesystem::192.168.100.9:/data/mysql::/mydata/mysql::nfs mysql
# scp /etc/ha.d/haresources node2:/etc/ha.d/
# service heartbeat start
# ssh node2 'service heartbeat start'
```

测试：在node1上连接mysql服务器，在mydb数据库的db1表上，添加一些数据，然后在node2上查看是否同步显示

 

```
# mysql -utom -h192.168.100.9 -p
> USE mydb
> INSERT INTO db1 (Name) VALUES (111);
> INSERT INTO db1 (Name) VALUES (222);
> INSERT INTO db1 (Name) VALUES (333);
> SELECT Name FROM db1;
```

 

```
# mysql -utom -h192.168.100.9 -p
> USE mydb
> INSERT INTO db1 (Name) VALUES (111);
> INSERT INTO db1 (Name) VALUES (222);
> INSERT INTO db1 (Name) VALUES (333);
> SELECT Name FROM db1;
```

V1版本想要监测后端RS健康状态，需要安装ldirectord插件：

由于V1版本无法检测后端RealServer的健康状态，因此，需要安装Heartbeat自带的一个包：

Heartbeat-ldirectord-2.1.4-12.el6.x86_64.rpm，它能帮LVS监测后端的RS的健康状态.

安装ldirectord

两节点分别安装此包，安装前停止Heartbeat服务,依赖包perl-MailTools

 

```
#rpm -qpi heartbeat-ldirectord-2.1.4-12.el6.x86_64.rpm   查看一下此包的信息，
#yum install heartbeat-ldirectord-2.1.4-12.el6.x86_64.rpm 
#rpm -ql heartbeat-ldirectord
#cp /usr/share/doc/heartbeat-ldirectord-2.1.4/ldirectord.cf /etc/ha.d/
#vim /etc/ha.d/ldirectord.cf
  # Global Directives      ##全局配置，对所有虚拟服务生效
   checktimeout=3          ##探测超时时长
   checkinterval=1         ##探测时间间隔
   #fallback=127.0.0.1:80  ##备用返回地址，当RS都不可探测时，返回此地址的数据
   autoreload=yes          ##重新载入，当配置文件修改后立即生效
   #logfile="/var/log/ldirectord.log"
   #logfile="local0"
   #emailalert="admin@x.y.z"
   #emailalertfreq=3600
   #emailalertstatus=all
   quiescent=yes            ###是否工作在静默模式下
  virtual=192.168.100.200:80             ##VIP
        real=192.168.6.2:80 gate         ##RIP，gate指
        real=192.168.6.3:80 gate
        real=192.168.6.6:80 gate
        fallback=127.0.0.1:80 gate       ##备用资源
        service=http                     ##检查realserver健康状态时使用的协议
        request="index.html"             ##向RS请求的页面
        receive="Test Page"              ##请求的页面希望包含的数据
        virtualhost=some.domain.com.au   ##探测RS的虚拟主机
        scheduler=rr                     ##调度方法
        #persistent=600                  ###持久时长
        #netmask=255.255.255.255
        protocol=tcp                     ##使用tcp协议进行探测
        checktype=negotiate
        checkport=80                     ##通过80端口进行探测
        request="index.html"
        receive="Test Page"
        virtualhost=www.x.y.z
```

 

```
#rpm -qpi heartbeat-ldirectord-2.1.4-12.el6.x86_64.rpm   查看一下此包的信息，
#yum install heartbeat-ldirectord-2.1.4-12.el6.x86_64.rpm 
#rpm -ql heartbeat-ldirectord
#cp /usr/share/doc/heartbeat-ldirectord-2.1.4/ldirectord.cf /etc/ha.d/
#vim /etc/ha.d/ldirectord.cf
  # Global Directives      ##全局配置，对所有虚拟服务生效
   checktimeout=3          ##探测超时时长
   checkinterval=1         ##探测时间间隔
   #fallback=127.0.0.1:80  ##备用返回地址，当RS都不可探测时，返回此地址的数据
   autoreload=yes          ##重新载入，当配置文件修改后立即生效
   #logfile="/var/log/ldirectord.log"
   #logfile="local0"
   #emailalert="admin@x.y.z"
   #emailalertfreq=3600
   #emailalertstatus=all
   quiescent=yes            ###是否工作在静默模式下
  virtual=192.168.100.200:80             ##VIP
        real=192.168.6.2:80 gate         ##RIP，gate指
        real=192.168.6.3:80 gate
        real=192.168.6.6:80 gate
        fallback=127.0.0.1:80 gate       ##备用资源
        service=http                     ##检查realserver健康状态时使用的协议
        request="index.html"             ##向RS请求的页面
        receive="Test Page"              ##请求的页面希望包含的数据
        virtualhost=some.domain.com.au   ##探测RS的虚拟主机
        scheduler=rr                     ##调度方法
        #persistent=600                  ###持久时长
        #netmask=255.255.255.255
        protocol=tcp                     ##使用tcp协议进行探测
        checktype=negotiate
        checkport=80                     ##通过80端口进行探测
        request="index.html"
        receive="Test Page"
        virtualhost=www.x.y.z
```

**基于heartbeat v2版crm，实现HA + mysql + wordpress**

环境：

node1:

host:www.node1.com

vip:192.168.100.71

eth0:192.168.100.7

node2:

host:www.node2.com

vip:192.168.100.81

eth0:192.168.100.8

1，安装heartbeat组件：

heartbeat-2.1.4-12.el6.x86_64.rpm

heartbeat-pils-2.1.4-12.el6.x86_64.rpm

heartbeat-stonith-2.1.4-12.el6.x86_64.rpm

heartbeat-gui-2.1.4-12.el6.x86_64.rpm，此包依赖pygtk2-libglade

依赖包：perl-TimeDate PyXML gettext gnutls libtool-ltdl pygtk2-libglade libnet net-snmp-libs

 

```
# yum install perl-TimeDate PyXML gettext gnutls libtool-ltdl -y
# rpm -ivh libnet-1.1.6-7.el6.x86_64.rpm
# yum install net-snmp-libs -y
# rpm -ivh heartbeat-pils-2.1.4-12.el6.x86_64.rpm
# rpm -ivh heartbeat-stonith-2.1.4-12.el6.x86_64.rpm
# rpm -ivh heartbeat-2.1.4-12.el6.x86_64.rpm
# yum install pygtk2-libglade -y
# rpm -ivh heartbeat-gui-2.1.4-12.el6.x86_64.rpm
```

 

```
# yum install perl-TimeDate PyXML gettext gnutls libtool-ltdl -y
# rpm -ivh libnet-1.1.6-7.el6.x86_64.rpm
# yum install net-snmp-libs -y
# rpm -ivh heartbeat-pils-2.1.4-12.el6.x86_64.rpm
# rpm -ivh heartbeat-stonith-2.1.4-12.el6.x86_64.rpm
# rpm -ivh heartbeat-2.1.4-12.el6.x86_64.rpm
# yum install pygtk2-libglade -y
# rpm -ivh heartbeat-gui-2.1.4-12.el6.x86_64.rpm
```

2，配置文件

 

```
# cp /usr/share/doc/heartbeat-2.1.4/ha.cf /etc/ha.d/
# cp /usr/share/doc/heartbeat-2.1.4/authkeys /etc/ha.d/
# chmod 600 /etc/ha.d/authkeys
# vim /etc/ha.d/ha.cf
 logfile /var/log/ha-log
 keepalive 2
 deadtime 30
 warntime 10
 initdead 120
 udpport 694
 baud    19200
 mcast eth0 225.0.100.1 694 1 0
 auto_failback on
 node www.node1.com
 node www.node2.com
 ping 192.168.100.1
 crm on
# vim /etc/ha.d/authkeys
auth 2
2 sha1 9fe252b0cf5ba0d3       ##随机数可使用命令: #openssl rand -hex 8
# /usr/lib64/heartbeat/ha_propagate      ##heartbeat自带的同步工具
# netstat -tnlp
crm通过mgmtd进程监听在5560/tcp端口
```

 

```
# cp /usr/share/doc/heartbeat-2.1.4/ha.cf /etc/ha.d/
# cp /usr/share/doc/heartbeat-2.1.4/authkeys /etc/ha.d/
# chmod 600 /etc/ha.d/authkeys
# vim /etc/ha.d/ha.cf
 logfile /var/log/ha-log
 keepalive 2
 deadtime 30
 warntime 10
 initdead 120
 udpport 694
 baud    19200
 mcast eth0 225.0.100.1 694 1 0
 auto_failback on
 node www.node1.com
 node www.node2.com
 ping 192.168.100.1
 crm on
# vim /etc/ha.d/authkeys
auth 2
2 sha1 9fe252b0cf5ba0d3       ##随机数可使用命令: #openssl rand -hex 8
# /usr/lib64/heartbeat/ha_propagate      ##heartbeat自带的同步工具
# netstat -tnlp
crm通过mgmtd进程监听在5560/tcp端口
```

3，测试

 

```
# crm_mon    ##实时监测命令
```

 

```
# crm_mon    ##实时监测命令
```

![img](Heartbeat%E5%9F%BA%E7%A1%80.assets/da24dabc-d8f2-4865-9a48-b30305c5275c.png)

4，在nfs服务器上：

 

```
# groupadd -g 27 mysql
# useradd -u 27 -g mysql -s /sbin/nologin -M mysql
# id mysql
uid=27(mysql) gid=27(mysql) groups=27(mysql)
# groupadd apache -g 48 
# useradd -g 48 -u 48 -s /sbin/nologin -M apache
# id apache
uid=48(apache) gid=48(apache) groups=48(apache)
# mkdir /data/mydata -p
# chown -R mysql.mysql /data/mydata
# ls -ld /data/mydata
# mkdir /data/html -p
# chown -R apache.apache /data/html
# ls -ld /data/html
# vim /etc/exports
/data/mydata    192.168.100.0/24(rw,no_root_squash)
/data/html      192.168.100.0/24(rw,no_root_squash)
# exportfs -rav    \\重新导出
# service nfs restart

```

 

```
# groupadd -g 27 mysql
# useradd -u 27 -g mysql -s /sbin/nologin -M mysql
# id mysql
uid=27(mysql) gid=27(mysql) groups=27(mysql)
# groupadd apache -g 48 
# useradd -g 48 -u 48 -s /sbin/nologin -M apache
# id apache
uid=48(apache) gid=48(apache) groups=48(apache)
# mkdir /data/mydata -p
# chown -R mysql.mysql /data/mydata
# ls -ld /data/mydata
# mkdir /data/html -p
# chown -R apache.apache /data/html
# ls -ld /data/html
# vim /etc/exports
/data/mydata    192.168.100.0/24(rw,no_root_squash)
/data/html      192.168.100.0/24(rw,no_root_squash)
# exportfs -rav    \\重新导出
# service nfs restart
```

注意：nfs服务器上创建的mysql,apache用户gid和uid都要与node1,node2的mysql,apache用户保持一致

下载wordpress，并解压至/data/html目录中

 

```
# wget https://cn.wordpress.org/wordpress-4.3.1-zh_CN.tar.gz
```

 

```
# wget https://cn.wordpress.org/wordpress-4.3.1-zh_CN.tar.gz
```

5,在node1和node2上安装mysql

 

```
#groupadd mysql
#useradd -r -g mysql mysql
# mkdir -p /data/{mydata,binlogs}
# chown -R mysql.mysql /data/*
# tar xf mysql-5.5.29-linux_x86_64.tar.gz -C /usr/local/
# ln -sv mysql-5.5.29 mysql
# cd /usr/local/mysql
# chown -R root.mysql /mysql/*
# scripts/mysql_install_db --user=mysql --datadir=/data/mydata
# mkdir /etc/mysql
# cp support-files/my-large.cnf /etc/mysql/my.cnf
# vim my.cnf
[mysqld]标签下，添加：
 datadir = /data/mydata
 innodb_file_per_table = ON
 log-bin=/data/mydata/binlogs/master-bin                                        
# cp support-files/mysql.server /etc/rc.d/init.d/mysqld
# chkconfig --add mysqld
# chkconfig mysqld off
# service mysqld start
# /usr/local/mysql/bin/mysql
```

 

```
#groupadd mysql
#useradd -r -g mysql mysql
# mkdir -p /data/{mydata,binlogs}
# chown -R mysql.mysql /data/*
# tar xf mysql-5.5.29-linux_x86_64.tar.gz -C /usr/local/
# ln -sv mysql-5.5.29 mysql
# cd /usr/local/mysql
# chown -R root.mysql /mysql/*
# scripts/mysql_install_db --user=mysql --datadir=/data/mydata
# mkdir /etc/mysql
# cp support-files/my-large.cnf /etc/mysql/my.cnf
# vim my.cnf
[mysqld]标签下，添加：
 datadir = /data/mydata
 innodb_file_per_table = ON
 log-bin=/data/mydata/binlogs/master-bin                                        
# cp support-files/mysql.server /etc/rc.d/init.d/mysqld
# chkconfig --add mysqld
# chkconfig mysqld off
# service mysqld start
# /usr/local/mysql/bin/mysql
```

在node1上挂载nfs

 

```
# mount -t nfs 192.168.100.9:/data/html /var/www/html
# mount
# ls /var/www/html
#
# mkdir /data/data -p
# chown -R mysql.mysql /data/data
# mount -t nfs 192.168.100.9:/data/mydata /data/mydata
# mount
```

 

```
# mount -t nfs 192.168.100.9:/data/html /var/www/html
# mount
# ls /var/www/html
#
# mkdir /data/data -p
# chown -R mysql.mysql /data/data
# mount -t nfs 192.168.100.9:/data/mydata /data/mydata
# mount
```

在node1上初始化mysql,测试成功后关闭服务并卸载nfs，node2上同样测试

 

```
# vim /etc/my.cnf
[mysqld]
datadir=/data/data
# /usr/bin/mysql_install_db --user=mysql --datadir=/mydata/data/
# service mysqld start
# 测试启动成功后，关闭服务，卸载挂载
# service mysqld stop
# umount /data/data
# umount /var/www/html
```

 

```
# vim /etc/my.cnf
[mysqld]
datadir=/data/data
# /usr/bin/mysql_install_db --user=mysql --datadir=/mydata/data/
# service mysqld start
# 测试启动成功后，关闭服务，卸载挂载
# service mysqld stop
# umount /data/data
# umount /var/www/html
```

6，使用ha_gui图形管理启动ha

安装ha_gui程序后，会自动创建一个系统用户hacluster，此用户是作为ha_gui管理程序的登录验证，默认不允许空密码登录，so需要手动设置一个密码

 

```
# passwd hacluster
123.com
# hb_gui &
```

 

```
# passwd hacluster
123.com
# hb_gui &
```

注意：高可用Web集群中有四个资源，分别是VIP、httpd服务、mysql服务、filesystem（NFS,用来存放Web文件和mysql数据库文件）

![img](Heartbeat%E5%9F%BA%E7%A1%80.assets/052146b2-4def-48e5-913f-23e8b20e6c5f.png)

 ![img](Heartbeat%E5%9F%BA%E7%A1%80.assets/16f5426b-f997-4444-8548-03bb6d9513b1.png)

 

![img](Heartbeat%E5%9F%BA%E7%A1%80.assets/07c1cefd-3f23-4174-9e92-d642fdc1fab4.png)

![img](Heartbeat%E5%9F%BA%E7%A1%80.assets/6bbebb41-f332-4d7e-b193-a9e7c625bedc.png)

![img](Heartbeat%E5%9F%BA%E7%A1%80.assets/ffbfe385-81bf-49af-a799-ba3c681f786b.png)

配置完成后进行测试：

http://192.168.100.71/wordpress

注意：需要httpd支持php

 

如果启动不成功，可尝试：

1，退出xshell重新进入

2，安装桌面系统

3，在物理机上安装XManager软件

 

```
# yum install xorg-x11-xauth -y
# yum groupinstall Desktop
```

 

```
# yum install xorg-x11-xauth -y
# yum groupinstall Desktop
```

**lvs高可用：**

环境：

node1：192.168.100.7，lvs

node2：192.168.100.8，lvs

node3：192.168.100.9，rs

node4：192.168.100.10，rs

vip：192.168.100.36

注意ipvs不要开机启动

RS上操作：

 

```
# echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
# echo 1 > /proc/sys/net/ipv4/conf/eth0/arp_ignore 
# echo 2 > /proc/sys/net/ipv4/conf/eth0/arp_announce
# echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
# ifconfig lo:0 192.168.100.36 netmask 255.255.255.255 broadcast 192.168.100.36 up
# route add -host 192.168.100.36 dev lo:0
# ifconfig
# route -n
# 启动httpd服务，创建一个index主页文件
```

 

```
# echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore
# echo 1 > /proc/sys/net/ipv4/conf/eth0/arp_ignore 
# echo 2 > /proc/sys/net/ipv4/conf/eth0/arp_announce
# echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce
# ifconfig lo:0 192.168.100.36 netmask 255.255.255.255 broadcast 192.168.100.36 up
# route add -host 192.168.100.36 dev lo:0
# ifconfig
# route -n
# 启动httpd服务，创建一个index主页文件
```

配置完成后，进行测试，正常来说，可以ping通9和10，但不能ping通36

node1，node2上配置并测试：

 

```
# ifconfig eth0:0 192.168.100.36/24 up
# route add -host 192.168.100.36 dev eth0:0
# ipvsadm -A -t 192.168.100.36:80 -s rr
# ipvsadm -a -t 192.168.100.36:80 -r 192.168.100.9 -g
# ipvsadm -a -t 192.168.100.36:80 -r 192.168.100.10 -g
# service ipvsadm save
# chkconfig ipvsadm off
# curl http://192.168.100.36
# service ipvsadm stop
# ifconfig eth0：0 down
# service heartbeat start
# hb_gui
```

 

```
# ifconfig eth0:0 192.168.100.36/24 up
# route add -host 192.168.100.36 dev eth0:0
# ipvsadm -A -t 192.168.100.36:80 -s rr
# ipvsadm -a -t 192.168.100.36:80 -r 192.168.100.9 -g
# ipvsadm -a -t 192.168.100.36:80 -r 192.168.100.10 -g
# service ipvsadm save
# chkconfig ipvsadm off
# curl http://192.168.100.36
# service ipvsadm stop
# ifconfig eth0：0 down
# service heartbeat start
# hb_gui
```

创建一个组director资源，创建vip，创建ipvsadm，运行测试

![img](Heartbeat%E5%9F%BA%E7%A1%80.assets/87565a41-e450-4d0c-9222-bb9d2ac5c335.png)

 node1,node2分别安装heartbeat-ldirectord

 

```
# rpm -ivh heartbeat-ldirectord-2.1.4-12.el6.x86_64.rpm
# rpm -ql heartbeat-ldirectord
# chkconfig ldirectord off
# cp /usr/share/doc/heartbeat-ldirectord-2.1.4/ldirectord.cf /etc/ha.d/
# vim /etc/ha.d/ldirectord.cf
# Sample for an http virtual service
virtual=192.168.100.36:80              \\定义vip
        real=192.168.100.9:80 gate      \\定义rs，gate（DR类型）masq（NAT类型）
        real=192.168.100.10:80 gate
        fallback=127.0.0.1:80 gate      \\当rs不可用，备用错误页面
        service=http                     \\基于http方式进行健康状态检测
        request=".health.html"              \\对监测的资源请求那个数据
        receive="OK"               \\数据包含啥
        #virtualhost=some.domain.com.au    \\表示对那个虚拟主机进行监测
        scheduler=rr                     \\调度方法
        #persistent=600
        #netmask=255.255.255.255
        #protocol=tcp                      \\协议
        #checktype=negotiate
        #checkport=80
        #request="index.html"
        #receive="Test Page"
        #virtualhost=www.x.y.z
# vim /var/www/html/.health.html
OK
# service heartbeat start
# hb_gui
```

 

```
# rpm -ivh heartbeat-ldirectord-2.1.4-12.el6.x86_64.rpm
# rpm -ql heartbeat-ldirectord
# chkconfig ldirectord off
# cp /usr/share/doc/heartbeat-ldirectord-2.1.4/ldirectord.cf /etc/ha.d/
# vim /etc/ha.d/ldirectord.cf
# Sample for an http virtual service
virtual=192.168.100.36:80              \\定义vip
        real=192.168.100.9:80 gate      \\定义rs，gate（DR类型）masq（NAT类型）
        real=192.168.100.10:80 gate
        fallback=127.0.0.1:80 gate      \\当rs不可用，备用错误页面
        service=http                     \\基于http方式进行健康状态检测
        request=".health.html"              \\对监测的资源请求那个数据
        receive="OK"               \\数据包含啥
        #virtualhost=some.domain.com.au    \\表示对那个虚拟主机进行监测
        scheduler=rr                     \\调度方法
        #persistent=600
        #netmask=255.255.255.255
        #protocol=tcp                      \\协议
        #checktype=negotiate
        #checkport=80
        #request="index.html"
        #receive="Test Page"
        #virtualhost=www.x.y.z
# vim /var/www/html/.health.html
OK
# service heartbeat start
# hb_gui
```