[TOC]

openssl



ISO（国际标准组织）定义了x.800安全框架，框架基本结构如下：

安全攻击：

\- 被动攻击：窃听数据

\- 主动攻击：伪装、修改消息、删除消息、重播消息



安全服务机制：

\- 认证：验证消息的发送方是不是发送方自己声称的那个人

\- 访问控制：只允许用户访问授权给用户资源

\- 数据保密性：

 连接保密：数据流保密（tcp连接）

 无连接保密：数据包加密（udp）

 选择域保密：对数据流或者数据包中的部分数据进行加密

 流量保密：对向互联网上发送的真实数据的流量大小进行加密                        

 数据完整性：保证数据从消息发送方到达消息接收方时没有经过非授权更改

 不可否认性：一旦通讯发生，通讯双方都不能否认



由于加密算法大多数都是公开的，纯粹用算法对数据进行加密是不安全的。因此现在的加密算法在加密的数据的时候会同时输入一段密钥，以保证数据的安全性.

数据加密流程：

[![wKiom1WaTM-QNxB3AABcSNXRXQ4737.jpg](ssl%E5%9F%BA%E7%A1%80.assets/0.5318847752641886.png)](http://s3.51cto.com/wyfs02/M01/6F/60/wKiom1WaTM-QNxB3AABcSNXRXQ4737.jpg)

数据解密流程：   

[![wKiom1WaTQ6CR4SfAABawxhzaLU078.jpg](ssl%E5%9F%BA%E7%A1%80.assets/0.8976108238566667.png)](http://s3.51cto.com/wyfs02/M02/6F/60/wKiom1WaTQ6CR4SfAABawxhzaLU078.jpg)

实现上述安全服务机制是需要算法参与的，算法的类型主要有以下几种：

对称加密

特性：加密解密使用同一个密钥。

优势：加密速度快

劣势：当一个用户需要跟众多用户进行安全通讯时需要维护众多密钥。

公钥加密（非对称加密）

特性： 加密解密使用一对密钥（公钥、私钥），公钥是从私钥中提取出来的

用公钥加密的数据只能用与之配对的私钥解密

用私钥加密的加密的数据只能用与之配对的公钥解密

公钥是公开的，任何人都可以得到，并使用

私钥是不公开的，只有私钥持有者可以使用

常见算法：**rsa**、**dsa**

优势：只需要维护一对密钥就可以跟众多用户进行安全通讯

劣势：加密速度慢.

单向加密

特性：只加密,不解密.提取数据特征码.

定长输出：不管输入的数据有多大，输出的特征码的长度都输固定的

蝴蝶效应：输入数据的微小改变会引起输出的特征码的巨大变化

md5（128bit）、sha1（160bit）、sha192、sha256、sha384、sha512   

现在互联网上常用的安全通讯模型：

[![wKiom1WbKdGiDNlrAACz-_IZLNA696.jpg](ssl%E5%9F%BA%E7%A1%80.assets/0.8430642897728831.png)](http://s3.51cto.com/wyfs02/M02/6F/64/wKiom1WbKdGiDNlrAACz-_IZLNA696.jpg)



发送方发送数据时执行的步骤如下（分别对应于上图发送方的1、2、3）：

1、发送方将需要在互联网上进行安全传输的数据采用单向加密算法提取数据的特征码，然后用自己的私钥加密这段特征码放在数据的尾部；

2、发送方生成一个一次性的对称加密算法的秘钥，然后使用对称加密算法和生成的秘钥将数据和加密后的特征码加密后生成密文；

3、将上一步对称加密用到的秘钥使用接收方的公钥加密后放在密文的后面；随后就可以将数据放到互联网上进行传输。



接收方接收到数据后执行的步骤如下（分别对应于上图接收方的1、2、3）：

1、接收方接收到数据后，用自己的私钥解密加密后的对称加密算法的秘钥。（如果能解密则能验证数据的机密性）

2、接收方用解密后得到的对称加密算法的秘钥后使用与发送方同样的对称加密算法解密加密的数据和加密的数据的特征码。

3、接收方使用与发送方相同的单向加密算法提取解密后的数据的特征码，而后使用发送方的公钥对上一步解密得到的加密的数据的特征码（如果能解密，则可以验证接收方的身份），并比较这两个特征码进行比较是否一致。（如果一致，则可以验证数据的完整）。



上面讲到了单向加密、对称加密、非对称加密等算法。那这几种类型的算法该如何实现呢？有没有什么可靠的工具呢？在linux上有一款非常好用的，并且能够实现上述的所发的开源软件openssl，下面我就介绍openssl：

OpenSSL是一套强大的具有加密功能的组件，它包含libcrypto（公共加密库）、libssl（SSL协议的实现）和openssl（多功能命令工具），因其开源思想，现已广泛应用于数据通信加密领域。OpenSSL还可在局域网内构建私有CA，实现局域网内的证书认证和授权，保证数据传输的安全性。如何构建私有CA呢？本文将详细讲述基于OpenSSL实现私有CA构建。

\#yum -y install openssl  \\安装opensll

\#rpm -ql openssl  \\查看openssl生成的所有文件

  openssl：多用途命令行工具

  libcrypto：公共加密库

  libssl：ssl协议的实现

主配置文件：/etc/pki/tls/openssl.cnf

openssl命令行：

\#openssl version  \\查看版本信息

 

**对称加密**

```
用法：openssl enc -cipher_name [-in filename] [-out filename]     ###cipher_name:aes,des,3des,rc6,idea...   
```

 

```
#openssl enc -des3 -in /etc/passwd -out /tmp/passwd.enc    ###对文件passwd加密，方式为des3，加密后的文件为passwd.enc
#file /etc/passwd
/etc/passwd: ASCII text   # 明文
#file /tmp/passwd.enc 
/tmp/passwd.enc: data     # 密文 
#openssl enc -des3 -in /tmp/passwd.enc -out /tmp/passwd.2 -d     ###对加密文件passwd.enc进行解密，解密后的文件为passwd.2
enter des-ede3-cbc decryption password:                          ###输入加密时使用的密码对文件进行解密
#file /tmp/passwd.2
/tmp/passwd.2: ASCII text # 明文
```



单项加密

用途：保证数据完整性，只能加密，不能解密，提取数据特征码。

特征：定长输出，雪崩效应（一个字节改变，所有的特征码均发生改变）

算法：md5:128bit

​      sha1:160bit

​      sha256

​      sha384

​      sha512

工具：md5sum,sha1sum,openssl dgst,chsum

```
用法：openssl dgst [-md5|-md4|-md2|-sha1|-sha] [file...]
```

 

```
#openssl dgst -md5 /etc/passwd    
MD5(/etc/passwd)= cde0b986a93a765834fe7183e53dc16d

#md5sum /etc/inittab               \\计算inittab的特征码，即md5码
#openssl dgst -md5 /etc/inittab     \\提取inittab的特征码

###dgst：指定用openssl工具实现提取文件特征码，
###-md5：指定使用MD5算法提取文件特征码
```



MAC：消息摘要码。单向加密的一种延伸类应用，用于实现在网络通信中保证所传输的数据的完整性。

单向加密应用实例：

用户密码：

openssl passwd -1 -salt 123.com

![img](ssl%E5%9F%BA%E7%A1%80.assets/7cd6048a-a50c-4844-8af6-db0212dc69d3.png)

 -1：表示使用md5加密算法加密

$1$:表示加密算法

$123.com$：表示salt

后面为加密后生成的password

生成随机数：

\#openssl rand -hex|base64 4  \\hex,base64:随机数的格式，不可同时使用，后跟数字表示生成的随机数多少位

\#openssl passwd -1 -salt `openssl rand -hex 4`

Password:

![img](ssl%E5%9F%BA%E7%A1%80.assets/7bc36b29-9344-454c-8b3c-6e02a40642f4.png)

 

公钥加密：

算法：RSA,EIGamal

工具：gpg，openssl rsautl

数字签名：

用途：主要用于让接收方确认发送方的身份

算法：RSA,EIGamal，

DSA(仅可用于签名，不能加密)数字签名

DSS：数字签名标准

公钥加密IKE：

算法：

  公钥加密：使用对方的公钥加密一个对称密钥发送给对方的方式。该方式可暴力破解

  DH（Diffie-Hellman）该方式不易被暴力破解

生成指定位数的密钥文件：

\#openssl genrsa -out /path/to/keyfile  num

使用rsa算法生成一个密钥，num位，保存到文件keyfile中，

例如：

\#（umask 077; openssl genrsa -out /tmp/key 2048）

生成一个权限为600的key密钥文件。在（）中执行，只对当前子shell有效

从生成的私钥文件提取公钥

\#openssl rsa -in /path/from/private_key_file -public

```
用法：openssl genrsa [-out filename] [numbits]
示例：
#openssl genrsa -out /tmp/key.pem 2048       Generating RSA private key, 2048 bit long modulus       .......................+++       ...+++       e is 65537 (0x10001)                         从私钥中提取与之对应的公钥用法：openssl rsa [-in filename] [-pubout] [-out filename]
示例：
#openssl rsa -in /tmp/key.pem -pubout -out /tmp/key.pub       writing RSA key
```

双方都加密时都会用到对方的公钥。由于公钥是公开的，任何人都可以得到并使用。私钥是保密的，只有私钥的拥有者才能使用。那么在互联网上的安全通讯过程中，通讯双方如何可靠的得到对方的公钥呢？这就需要一种手段来实现。CA其实就是实现让通讯双方可靠的得到对方公钥的一种手段。CA的实现方式是通过给通讯者颁发证书，目前通用的证书格式为x509，证书基本格式如下：

![img](ssl%E5%9F%BA%E7%A1%80.assets/dd31482e-24f1-4059-a65a-0dc284a5cfd2.jpg)

CA其实就是一个大家都信任的公信机构，因此如果信任该CA就信任此CA给通讯者颁发的证书，那么就可以放心的使用证书中的公钥跟证书的拥有者通讯。CA要想给客户端发证书，首先就要给自己签署一个自签证书。而且此证书也需要分发给各个信任他的互联网主机。

证书主要有以下几个组件组成（统称为PKI）：

​     证书存取库：证书申请者或拥有者跟RA、CA、CRL打交道的接口

​     RA（证书注册机构）：证书申请者申请证书的机构

​     CA（证书颁发机构）：给证书申请者签署证书的机构

​     CRL（证书吊销列表）：  证书拥有者的私钥丢失所要用到的列表    

​     PS： PKI全称为公钥基础设施

​     

​      早期的互联网中，大多数的协议都是明文的，使得在互联网上传输的数据非常的不安全。Netscape公司为了实现数据在互联网上安全传输，就开发了一种实现数据安全传输的名叫SSL的协议，常见的SSL协议共有三个版本SSLv1、SSLv2、SSLv3，目前常用的是SSLv3。后来在开源界也有SSL协议的开源实现TLS，有TLSv1的版本，此版本相当于SSLv3。

​     TLS（transfer layer security)、SSL（secure socket layer）：他们都是一种位于应用层和传输层之间的协议，可以基于TCP和UDP，上层应用可以使用该协议对数据进行加密，从而保证传输数据的安装也可以不使用此协议。因此，大多数人都称ssl是一种位于应用层和传输层之间的半层协议。



​     如果想使用证书在企业或组织内部进行安全通讯，那么就需要构建私有CA，颁发证书时也有一个流程 ，具体流程和步骤如下：

```
[root@localhost CA]# rpm -ql openssl        /etc/pki/CA    /etc/pki/CA/certs    /etc/pki/CA/crl    /etc/pki/CA/newcerts           # 新生成的证书的副本存放位置    /etc/pki/CA/private    /etc/pki/tls    /etc/pki/tls/certs    /etc/pki/tls/certs/Makefile    /etc/pki/tls/certs/make-dummy-cert    /etc/pki/tls/certs/renew-dummy-cert    /etc/pki/tls/misc    /etc/pki/tls/misc/CA    /etc/pki/tls/misc/c_hash    /etc/pki/tls/misc/c_info    /etc/pki/tls/misc/c_issuer    /etc/pki/tls/misc/c_name    /etc/pki/tls/openssl.cnf       # openssl配置文件信息    /etc/pki/tls/private           # CA私钥存放位置
```

建立私有CA服务器：

CA server端：

1,创建自己的私钥：

 

```
#（umask 077; openssl genrsa -out /etc/pki/CA/private/cakey.pem 2048）
\\在/etc/pki/CA/private/目录下生成私钥cakey.pem，private是用来存放私钥文件的，cakey.pem是配置文件指定的名称，如果要自定义，需要同步更新openssl配置文件内的名称
```



2，提取私钥对应的公钥，并给创建一份自签署证书

 

```
#openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 3650    
\\在/etc/pki/CA/目录下生成公钥cacert.pem，cacert.pem是配置文件指定的名称，如果要自定义，需要同步更新openssl配置文件内的名称
```



说明：

req:生成证书签署请求

-news:新的请求

-key cakey.pem所在路径:指定私钥

-x509:生成自签署证书

-days 3650:有效天数

配置完成后，查看CA目录下，生成一个cacert.pem文件，该文件即为自签证书文件

3，初始化CA的工作环境

 

```
# touch /etc/pki/CA/{index.txt,serial}    \\index证书索引文件，serial已签署的证书编号文件
# echo 00 > /etc/pki/CA/serial            \\定义初始编号
# cat /etc/pki/CA/serial
```



在客户端上申请证书：

1，在客户端生成密钥对

```
#(umask 077;openssl genrsa -out /etc/httpd/ssl/httpd.key 2048)
```

2，生成证书签署请求

```
#openssl req -new -key /etc/httpd/ssl/httpd.key -out /etc/httpd/ssl/httpd.csr
```

3，把证书请求发给CA

```
#scp /etc/httpd/ssl/httpd.csr root@172.10.1.1    ###172.10.1.1为CA服务器地址
```

然后回到CA服务器上

4，为客户端的请求进行签署证书 

```
#openssl ca -in /etc/httpd/ssl/httpd.csr -out /etc/httpd/ssl/httpd.crt -days 300###已签署的证书为httpd.crt
```

5,把签署的证书发送给客户端

```
#scp /etc/httpd/ssl/httpd.crt root@172.10.1.100    ###172.10.1.100为客户端服务器地址
```

有了CA颁发的证书之后，互联网上的安全通讯过程如下（以https为例，这个过程也称之为SSL、TLS会话的建立过程）：

[![wKiom1Wd8T2D63SSAADOgEl9ws0974.jpg](ssl%E5%9F%BA%E7%A1%80.assets/0.8364731054753065.png)](http://s3.51cto.com/wyfs02/M02/6F/7D/wKiom1Wd8T2D63SSAADOgEl9ws0974.jpg)

​      由于https是基于tcp/443号端口，因此在SSL会话开始之前有TCP的三次握手过程，而后才是SSL：

​      1、客户端向服务器请求服务器的证书；

​      2、服务器准备证书，客户端从服务器处下载证书

​      3、客户端用事先下载在本地的CA证书中的公钥解密从服务器下载下来的证书的数字签名来获得证书主体部分的特征码（如果能解密，则就能验证此证书确实是客户端信任的CA颁发的证书），而后用与同样的算法提取证书主体部分的特征码，并比较两者是否一致。（如果一致，就能验证证书是完整的，没有被人篡改过，后续的通讯过程客户端就用从服务器上下载下来的证书中的公钥来进行加密通讯。）

吊销证书

1，在客户端上，获取自己的证书编号和信息

\#openssl x509 -in /etc/httpd/ssl/httpd.crt -noout -serial -subject

![img](ssl%E5%9F%BA%E7%A1%80.assets/cc66e2fd-d1cf-48c2-b0c4-d1b75c0defb7.png)

2， 在服务端上，查看/etc/pki/CA/index.txt文件中编号为01的行，是否于客户端提供的一致。

![img](ssl%E5%9F%BA%E7%A1%80.assets/98a408c9-4c27-4fff-a719-33d11cf8a6d1.png)

 3，在CA服务器上，吊销对应编号的证书

\#openssl ca -revoke /etc/pki/CA/newcerts/01.pem

![img](ssl%E5%9F%BA%E7%A1%80.assets/f3db2b05-48bb-45b3-bcc9-ab2c20ced1dc.png)

 

4,生成吊销证书的编号

\#touch /etc/pki/CA/crlnumber

\#echo 00 > /etc/pki/CA/crlnumber

5,更新吊销列表，同步配置文件

\#openssl ca -gencrl -out /etc/pki/CA/crl/this.crl

![img](ssl%E5%9F%BA%E7%A1%80.assets/4a3dacad-6fb1-448e-9aa7-c43deb4e1552.png)

 6，查看吊销列表crl文件内容

\#openssl crl -in /etc/pki/CA/this.crl -noout -text

![img](ssl%E5%9F%BA%E7%A1%80.assets/e7755920-70ed-4923-a61b-8556b45640db.png)

 