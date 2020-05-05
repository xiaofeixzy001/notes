[TOC]

# 简介

Samba是一个能让Linux系统应用Microsoft网络通讯协议的软件，而SMB是Server Message Block的缩写，即为服务器消息块 ，SMB主要是作为Microsoft的网络通讯协议，后来Samba将SMB通信协议应用到了Linux系统上，就形成了现在的Samba软件。

Samba最大的功能就是可以用于Linux与windows系统直接的文件共享和打印共享.Samba既可以用于windows与Linux之间的文件共享,也可以用于Linux与Linux之间的资源共享。

由于NFS(网络文件系统）可以很好的完成Linux与Linux之间的数据共享，因而 Samba较多的用在了Linux与windows之间的数据共享上面。

SMB是基于C/S的协议，因而一台Samba服务器既可以充当文件共享服务器，也可以充当一个Samba的客户端Samba在windows下使用的是NetBIOS协议，如果你要使用Linux下共享出来的文件，请确认你的windows系统下是否安装了NetBIOS协议。

组成Samba运行的有两个服务：smb和nmb==。

- SMB是Samba 的核心启动服务，主要负责建立 Linux Samba服务器与Samba客户机之间的对话， 验证用户身份并提供对文件和打印系统的访问，只有SMB服务启动，才能实现文件的共享，监听139 TCP端口。

- NMB服务是负责解析用的，类似与DNS实现的功能，NMB可以把Linux系统共享的工作组名称与其IP对应起来，如果NMB服务没有启动，就只能通过IP来访问共享文件，监听137和138 UDP端口。

## 功能

- WINS和DNS服务

- 网络浏览服务

- Linux和Windows域之间的认证和授权

- UNICODE字符集和域名映射

- 满足CIFS协议的UNIX共享等



## 服务程序

nmbd：netbios

smbd：cifs

winbindd：实现AD域中活动目录

## 监听端口

udp/137  udp/138

tcp/139  tcp/445

## 用户管理

帐号: 都是系统用户，存在于/etc/passwd

密码: samba服务自由密码文件

将系统用户添加为samba的命令：

smbpasswd

  -a：添加系统用户为samba用户

  -d：禁用

  -e：启用

  -x：删除

## 配置文件

配置文件：/etc/samba/smb.conf

Global Settings 全局设定

\# 该设置都是与Samba服务整体运行环境有关的选项，它的设置项目是针对所有共享资源的



workgroup = WORKGROUP

\# 设定 Samba Server 所要加入的工作组或者域



server string = Samba Server Version %v 

\# 设定 Samba Server 的注释，可以是任何字符串，也可以不填。宏%v表示显示Samba的版本号



netbios name = smbserver

\# 设置Samba Server的NetBIOS名称



interfaces = lo eth0 192.168.12.2/24 192.168.13.2/24

\# 设置Samba Server监听哪些网卡，可以写网卡名，也可以写该网卡的IP地址



hosts allow = 127. 192.168.1. 192.168.10.1

\# 表示允许连接到Samba Server的客户端，多个参数以空格隔开。可以用一个IP表示，也可以用一个网段表示, hosts deny 与hosts allow 刚好相反



max connections = 0

\# 指定连接Samba Server的最大连接数目。如果超出连接数目，则新的连接请求将被拒绝。0表示不限制。



deadtime = 0

\# 用来设置断掉一个没有打开任何文件的连接的时间,单位是分钟,0代表Samba Server不自动切断任何连接。



time server = yes/no

\# 用来设置让nmdb成为windows客户端的时间服务器。



log file = /var/log/samba/log.%m

\# 设置Samba Server日志文件的存储位置以及日志文件名称。在文件名后加个宏 %m（主机名），表示对每台访问Samba Server的机器都单独记录一个日志文件



max log size = 50

\# 设置Samba Server日志文件的最大容量,单位为kB，0代表不限制



security = user

\# 设置用户访问Samba Server的验证方式，一共有四种验证方式:

1,share：用户访问Samba Server不需要提供用户名和口令, 安全性能较低;

2,user：Samba Server共享目录只能被授权的用户访问,由Samba Server负责检查账号和密码的正确性。账号和密码要在本Samba Server中建立;

3,server：依靠其他Windows NT/2000或Samba Server来验证用户的账号和密码,是一种代理验证。此种安全模式下,系统管理员可以把所有的Windows用户和口令集中到一个NT系统上,使用 Windows NT进行Samba认证, 远程服务器可以自动认证全部用户和口令,如果认证失败,Samba将使用用户级安全模式作为替代的方式;

4,domain：域安全级别,使用主域控制器(PDC)来完成认证;



passdb backend = tdbsam

\# passdb backend就是用户后台的意思,目前有三种后台：smbpasswd、tdbsam和ldapsam

sam应该是security account manager（安全账户管理）的简写;

smbpasswd:该方式是使用smb自己的工具smbpasswd来给系统用户（真实用户或者虚拟用户）设置一个Samba密码，客户端就用这个密码来访问Samba的资源,smbpasswd文件默认在/etc/samba目录下,不过有时候要手工建立该文件;

tdbsam: 该方式则是使用一个数据库文件来建立用户数据库。数据库文件叫 passdb.tdb，默认在/etc/samba目录下。passdb.tdb用户数据库 可以使用smbpasswd –a来建立Samba用户，不过要建立的Samba用户必须先是系统用户;

我们也可以使用pdbedit命令来建立Samba账户。pdbedit命令的参数很多，我们列出几个主要的：

pdbedit

pdbedit –a username：新建Samba账户。

pdbedit –x username：删除Samba账户。

pdbedit –L：列出Samba用户列表，读取passdb.tdb数据库文件。

pdbedit –Lv：列出Samba用户列表的详细信息。

pdbedit –c “[D]” –u username：暂停该Samba用户的账号。

pdbedit –c “[]” –u username：恢复该Samba用户的账号。

ldapsam: 该方式则是基于LDAP的账户管理方式来验证用户,首先要建立LDAP服务，然后设置“passdb backend = ldapsam:ldap://LDAP Server”



encrypt passwords = yes/no

\# 是否将认证密码加密



smb passwd file = /etc/samba/smbpasswd

\# 用来定义samba用户的密码文件,smbpasswd文件如果没有那就要手工新建



username map = /etc/samba/smbusers

\# 用来定义用户名映射，比如可以将root换成administrator、admin等。不过要事先在smbusers文件中定义好, 比如：root = administrator admin，这样就可以用administrator或admin这两个用户来代替root登陆Samba Server，更贴近windows用户的习惯



guest account = nobody

\# 用来设置guest用户名



socket options = TCP_NODELAY SO_RCVBUF=8192 SO_SNDBUF = 8192

\# 用来设置服务器和客户端之间会话的Socket选项,可以优化传输速度



Share Definitions 特定共享的设定

该设置针对的是共享目录个别的设置，只对当前的共享资源起作用。

config file可以定义一个单独的配置文件，用来为某台主机或用户单独使用。

比如客户端pc1访问samba服务器，可以在服务器/etc/samba/host/目录下创建pc1的配置文件smb.conf.pc1，然后在samba配置文件中添加一行: 

config file = /usr/loacl/samba/lib/smb.conf.%m

当pc1连接至samba服务器时，smb.conf.%m会自动替换为smb.conf.pc1，客户端pc1使用的samba服务器即为smb.conf.pc1文件定义的规则，其他客户端访问则会按照smb.conf规则

comment = 任意字符串

\# comment 是对该共享的描述，可以是任意字符串

path = 共享目录路径

\# 指定共享目录的路径

browseable = yes/no

\# 用来指定该共享是否可以浏览

writable = yes/no

\# 用来指定该共享路径是否可写

available = yes/no

\# available用来指定该共享资源是否可用

admin users = 该共享的管理者

\# admin users用来指定该共享的管理员(对该共享具有完全控制权限)

valid users = 允许访问该共享的用户

\# 多个用户或者组中间用逗号隔开，如果要加入一个组就用“@组名”表示

invalid users = 禁止访问该共享的用户

write list = 允许写入该共享的用户

public = yes/no

\# public用来指定该共享是否允许guest账户访问

特殊共享：

[homes]

comment = Home Directories

browseable = no

writable = yes

valid users = %S

; valid users = MYDOMAIN\%S

 

[printers]

comment = All Printers

path = /var/spool/samba

browseable = no

guest ok = no

writable = no

printable = yes

 

[netlogon]

comment = Network Logon Service

path = /var/lib/samba/netlogon

guest ok = yes

writable = no

share modes = no

 

[Profiles]

path = /var/lib/samba/profiles

browseable = no

guest ok = yes

testparm  测试配置文件是否有误

 

```
[root@linux-study ~]# vim /etc/samba/smb.conf
"""
# 修改为与windows同组名
workgroup = WORKGROUP  
config file = /usr/local/samba/lib/smb.conf.%m
"""
```

如果有开启iptables和selinux，需要修改如下：

配置conf文件 /etc/samba/smb.conf设置security 为 share, 并且注释其邻近的下一行 tbsam 

 

```
[transwarp_shared]
comment = Public Stuff
path = /mnt/Samba_Shared
public = yes
writable = yes

directory mask = 0777
force directory mode = 0777
directory security mask = 0777
force directory security mode = 0777

create mask = 0777
force create mode = 0777
security mask = 0777
force security mode = 0777
```

这是因为我们的sharefile的目录的selinux值，不和samba默认的selinux值匹配，我们又打开了selinux,所以会被selinux拒绝，导致访问不了，那怎么办呢？好办，更改掉sharefile这个目录的selinux值
[CentOS7] 需要机器重启才能设置samba_share_t 类型

 

```
[root@linux-study ~]# ls -Zd /home/sharefile
"""
drwxrwxr-x root student root:object_r:file_t    /home/sharefile/
"""
```

标红色的地方时整个文件的selinux的类型，而samba得selinux类型为:samba_share_t所以我们需要用命令 chcon -R -t samba_share_t /home/sharefile这样我们就把sharefile得类型更改成samba的类型了重启samba 服务
3,配置iptables 139 和 445 端口添加到iptables例外中netstat -anpl|grep smb 命令
\4. 挂载网络smb到启动://172.16.0.178/transwarp_shared/     /share/ cifs   defaults     0 1因为我是在自己的机器上搭建一个SMB服务器，所以重启遇到了一些问题：smb服务还没启动却先进行smb网盘挂载！解决方案就是，去掉fstab中的那行smb服务挂载，然后到/etc/rc.local里面写上手动挂载的语句：mount //172.16.0.178/transwarp_shared/ /share/ -o password=    (密码为空)

Samba Web管理工具 SWATSWAT(Samba WEB Administration Tool) 是通过浏览器对 Samba 进行管理的工具之一。通过 SWAT，可以在 Samba 允许访问范围内的客户端，用浏览器对服务端的 Samba 进行控制。在线文档的阅览、smb.conf 的确认和编辑，以及密码的变更、服务的重启等等都可以通过 SWAT 来完成，它的直观让 Samba 变得温和化，对那些不喜欢文本界面管理服务器的朋友来说，是一个强大的工具。swat工具嵌套在xinetd超级守护进程中，要通过启用xinetd进程来启用swat。因此要先安装xinetd工具包，然后安装swat工具包。上面已经安装过 samba-swat-3.5.10-125.el6.x86_64，这里不再赘述。

安装配置swat：

因为swat是xinetd超级守护进程的一个子进程，所以swat工具配置文件在xinetd目录中。我们要设置swat配置文件，开启此子进程，以便在启用xinetd进程是来启用swat。swat配置文件在/etc/xinetd.d目录中

 

```
[root@linux-study ~]# vim /etc/xinetd.d/swat
"""
default: off # description: SWAT is the Samba Web Admin Tool. 
Use swat \ # to configure your Samba server. To use SWAT, \ # connect to port 901 with your favorite web browser. service swat 
{ port = 901 # swat默认使用tcp的901端口, 可以修改 
socket_type = stream # 通过web来配置samba, 默认使用root账号进入, 可以修改成其他的系统用户 
wait = no only_from = 127.0.0.1  
only_from = 10.0.0.0 # 添加此行, 将“only_from=127.0.0.1”改成“only_from=10.0.0.0”, 只允许内网范围对SWAT进行访问
user = root server = /usr/sbin/swat # swat的执行程序默认在/usr/sbin目录下 log_on_failure += USERID 
disable = yes # 将“disable=yes”改成“disable=no”, 这样swat子进程就可以随xinetd超级守护进程一起启动了 
}
"""
```

启动 swat

因为swat是xinetd的子进程，所以只要启用了xinetd，那么swat也就会伴随xinetd启动。

在服务端启动 swat后，我们就可以通过 swat允许范围内的客户机的浏览器中，通过 http://服务器的内网IP:901 来访问服务端的 swat了，输入 root用户的用户名及密码进入 swat的管理首页，如下所示：swat管理中心的首页
通过 swat管理 Samba 与直接修改 smb.conf 的方式，在本质上并无差异，但通过浏览器访问的方式，可以使 Samba 的管理更加温和化，更加适用于不擅长使用文本界面、直接修改配置文件的朋友。
通过swat配置samba在swat页面我们可以看到有8个选项，每个选项可以配置samba的不同功能:HOME：Samba相关程序及文件说明。GLOBALS：设置Samba的全局参数。即smb.conf文件的[global]。SHARES：设置Samba的共享参数。PRINTERS：设置Samba的打印参数。WIZARD：Samba配置向导。STATUS：查看和设置Samba的服务状况。VIEW：查看Samba的文本配置文件，即smb.conf。PASSWORD：设置Samba用户，可以修改密码，新建删除用户。
至此，Samba服务器的所有配置完成。



# 需求

1 - 有3个部门：dep01，dep02，dep03，分别对应各自目录：dep01,dep02,dep03;

2 - 各部门成员对部门目录有读写权限，对其他部门只有访问权限；

3 - 有一个公共共享目录，所有人可读可写，但仅能删除自己所属目录或文件；

4 - 有一个总管理员，对所有目录有读写权限

# 账号和目录规划

公共目录：/data/shared/pub

dep01：/data/shared/dep01

dep02：/data/shared/dep02

dep03：/data/shared/dep03

# 环境

系统：CentOS 6.9

IP：172.22.5.107

软件：Samba Version 3.6.23-53.el6_10

# 配置

## 1 - 挂载新硬盘作为数据存储

```shell
# fdisk /dev/sdb
'''
m
n
p
1
[默认]
[默认]
p
w
'''

# mkfs.ext4 /dev/sdb1
# fdisk -l
# mkdir /data
# useradd -d /data/share -s /sbin/nologin samba  # 创建用户samba,家目录为/data/share
# gpasswd -A samba samba  # 指定组samba管理员为samba
# mount /dev/sdb1 /data/share
# vim /etc/fstab
'''
/dev/sdb1               /data/share             ext4    defaults        0 0
'''
```



## 2 - 安装samba服务

```shell
# yum install samba -y
# rpm -ql samba
# cp /etc/samba/smb.conf{,.bak}
# rpm -ql samba | less
"""
/etc/samba/smb.conf  # 配置文件
/etc/rc.d/init.d/nmb  # 用于启动netbios协议
/etc/rc.d/init.d/smb  # 用于启用cifs协议
"""
```



## 3 - 创建目录和用户

```shell
# 创建组
groupadd dep01
groupadd dep02
groupadd dep03

# 创建系统用户
useradd -s /sbin/nologin shareadmin
useradd -g dep01 -s /sbin/nologin user01
useradd -g dep02 -s /sbin/nologin user02
useradd -g dep03 -s /sbin/nologin user03
groups shareadmin
groups user01
groups user02
groups user03

# 创建samba用户,默认密码均为123456
smbpasswd -a shareadmin
smbpasswd -a user01
smbpasswd -a user02
smbpasswd -a user03
pdbedit -L

# 创建目录
mkdir -pv {pub,dep01,dep02,dep03}

# 设置属主和属组
chown -R shareadmin.dep01 dep01
chown -R shareadmin.dep02 dep02
chown -R shareadmin.dep03 dep03

```



## 4 - 共享配置

修改samba配置文件,设置各共享目录权限。

```shell
# grep -v ^# /etc/samba/smb.conf
[global]
	workgroup = WORKGROUP
	server string = PHCF Samba Server Version %v
	netbios name = SHARE
	socket options = TCP_NODELAY SO_RCVBUF=8192 SO_SNDBUF = 8192
;	interfaces = lo eth0 192.168.12.2/24 192.168.13.2/24 
;	hosts allow = 127. 192.168.12. 192.168.13.
	# logs split per machine
	log file = /var/log/samba/log.%m.%U
	# max 50KB per log file, then rotate
	max log size = 50
	log level = 3
	
	security = user
	passdb backend = tdbsam
;	smb passwd file = /etc/samba/smbpasswd
;	security = domain
;	passdb backend = tdbsam
;	realm = MY_REALM
;	password server = <NT-Server-Name>
;	security = user
;	passdb backend = tdbsam
;	domain master = yes 
;	domain logons = yes
	# the login script name depends on the machine name
;	logon script = %m.bat
	# the login script name depends on the unix user used
;	logon script = %u.bat
;	logon path = \\%L\Profiles\%u
	# disables profiles support by specifing an empty path
;	logon path =          
;	add user script = /usr/sbin/useradd "%u" -n -g users
;	add group script = /usr/sbin/groupadd "%g"
;	add machine script = /usr/sbin/useradd -n -c "Workstation (%u)" -M -d /nohome -s /bin/false "%u"
;	delete user script = /usr/sbin/userdel "%u"
;	delete user from group script = /usr/sbin/userdel "%u" "%g"
;	delete group script = /usr/sbin/groupdel "%g"
;	local master = no
;	os level = 33
;	preferred master = yes
;	wins support = yes
;	wins server = w.x.y.z
;	wins proxy = yes
;	dns proxy = yes
	load printers = yes
	cups options = raw
;	printcap name = /etc/printcap
	#obtain list of printers automatically on SystemV
;	printcap name = lpstat
;	printing = cups
;	map archive = no
;	map hidden = no
;	map read only = no
;	map system = no
;	store dos attributes = yes

;[homes]
;	comment = Home Directories
;	browseable = no
;	writable = yes
;	valid users = %S
;	valid users = MYDOMAIN\%S
	
;[printers]
;	comment = All Printers
;	path = /var/spool/samba
;	browseable = no
;	guest ok = no
;	writable = no
;	printable = yes
	
;	[netlogon]
;	comment = Network Logon Service
;	path = /var/lib/samba/netlogon
;	guest ok = yes
;	writable = no
;	share modes = no
	
;	[Profiles]
;	path = /var/lib/samba/profiles
;	browseable = no
;	guest ok = yes
	
;	[public]
;	comment = Public Stuff
;	path = /home/samba
;	public = yes
;	writable = yes
;	printable = no
;	write list = +staff

[Public]
	comment = This is public directory.
	path = /data/shared/pub
	public = yes
	writable = yes
	admin users = @shareadmin,@dep01,@dep02,@dep03
	valid users = @shareadmin,@dep01,@dep02,@dep03
	create mask = 0777
	directory mask = 0777

[dep01]
	comment = Welcome!
	path = /data/shared/dep01
	writable = yes
	admin users = @shareadmin,@dep01
	valid users = @shareadmin,@dep01,@dep02,@dep03
	create mask = 0774
	directory mask = 0775

[dep02]
        comment = Welcome!
        path = /data/shared/dep02
        writable = yes
        admin users = @shareadmin,@dep02
        valid users = @shareadmin,@dep01,@dep02,@dep03
        create mask = 0774
        directory mask = 0775

[dep03]
        comment = Welcome!
        path = /data/shared/dep03
        writable = yes
        admin users = @shareadmin,@dep03
        valid users = @shareadmin,@dep01,@dep02,@dep03
        create mask = 0774
        directory mask = 0775
```



## 5 - 测试

在windows客户端上：

```powershell
# 开始 - 运行
\\172.22.5.107
# 输入对应帐号和密码即可
```

在linux客户端上：

```shell
# 查看
# smbclient -L //172.22.5.107 -U shareadmin

# 挂载
# mount -t cifs -l ///172.22.5.107 /mnt/samba/
```



## 6 - windows清除登录记录

在windows下通过“\\ip地址”的方式访问其它文件资源时，一般第一次需要输入密码，以后就无需输入密码直接登陆了，那么如果我们要切换到其它Samba用户怎么办呢？可以在windows下执行如下指令实现：

首先通过开始-->运行-->cmd 输入：“net use”命令查看现有的连接，然后执行“net use \\Samba服务器IP地址或者netbios名称\ipc$ /del”，删除Samba服务器已经建立的连接。或者执行“net use * /del”将现在所有的连接全部删除。最后，再次执行“\\ip地址”时，就可以切换用户了。

## 7 - samba定义的变量：

%S = 当前服务名（如果有的话）

%P = 当前服务的根目录（如果有的话）

%u = 当前服务的用户名（如果有的话）

%g = 当前用户说在的主工作组

%U = 当前对话的用户名

%G = 当前对话的用户的主工作组

%H = 当前服务的用户的Home目录

%v = Samba服务的版本号。

%h = 运行Samba服务机器的主机名

%m = 客户机的NETBIOS名称

%L = 服务器的NETBIOS名称

%M = 客户机的主机名

%N = NIS服务器名

%p = NIS服务的Home目录

%R = 说采用的协议等级(值可以是CORE, COREPLUS, LANMAN1, LANMAN2，NT1)

%d = 当前服务进程的ID

%a = 客户机的结构（只能识别几项：Samba，WfWg，WinNT，Win95）

%I = 客户机的IP

%T = 当前日期和时间

## 8 - 注意事项

配置文件中，关于admin users选项，经试验要么都是用户，要么都是组，如果是用户和组混合，不生效。

启用日志，注意配置日志级别。