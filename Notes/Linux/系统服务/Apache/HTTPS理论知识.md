[TOC]

# https协议

ssl(安全的套接字层)

tls(传输层安全)

http协议是文本编码

https协议是基于ssl二进制编码


验证，使用telnet发请求
```
[root@xiaofei ~]# telnet 172.10.0.1 80
Trying 172.10.0.1...
Connected to 172.10.0.1.
Escape character is '^]'.
GET /index.html HTTP/1.0

####GET请求的资源为/index.html，协议版本为HTTP1.0/1.1
HOST: www.xiaofei.com ####请求的首部，主机
HTTP/1.1 200 OK
Date: Sun, 07 Aug 2016 14:46:12 GMT
Server: Apache/2.2.15 (CentOS)
Last-Modified: Sat, 06 Aug 2016 17:13:03 GMT
ETag: "20068-10-5396a4bbd5979"
Accept-Ranges: bytes
Content-Length: 16
Connection: close
Content-Type: text/html; charset=UTF-8
<h1>HOST A</h1> ####请求结果
Connection closed by foreign host.
```

# 数字证书格式

证书格式版本号V3

证书序列号

证书签名算法

证书颁发者CA

有效期

对象名称

对象的公开密钥

其他扩展信息


# 数字签名

ssl会话基于IP地址创建，每个IP仅能创建一个SSL会话.

SSL握手要完成的工作：
- 交换协议版本号
- 选择双方都支持的加密方式
- 客户端对服务器端实现身份验证
- 密钥交换


https协议是基于ssl二进制编码，端口443/TCP


客户端验证服务器端证书：

有效性检测：证书是否仍在有效期内

CA的可信度检测

证书的完整性检测

持有者的身份检测

# HTTPS

https，是基于ssl加密实现，ssl是一个模块，需要单独安装并加载启用。

```
# 查看所有单独成包的模块
[root@linux-study ~]# yum list all mod*
```

# 示例:配置https

## 安装mod_ssl模块
```
[root@linux-study ~]# yum install mod_ssl
[root@linux-study ~]# rpm -ql mod_ssl
/etc/httpd/conf.d/ssl.conf # 主配置文件
/usr/lib64/httpd/modules/mod_ssl.so # 模块文件 
/var/cache/mod_ssl # ssl缓存文件
/var/cache/mod_ssl/scache.dir
/var/cache/mod_ssl/scache.pag
/var/cache/mod_ssl/scache.sem
```

## 为服务器生成私钥，并为其提供证书
首先需要建立CA,不一定要在web服务器上,在CA服务器上操作.
```
[root@linux-study ~]# cd /etc/pki/CA/
[root@linux-study ~]# (umask 077; openssl genrsa -out private/cakey.pem 2048) # 生成私钥，文件为cakey.pem
[root@linux-study ~]# openssl req -new -x509 -key private/cakey.pem -out cacert.pem -days 3650 # 创建自签证书
"""
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:BJ
Locality Name (eg, city) [Default City]:CY
Organization Name (eg, company) [Default Company Ltd]:xiaofei
Organizational Unit Name (eg, section) []:Ops
Common Name (eg, your name or your server's hostname) []:caserver.xiaofei.com
Email Address []:
"""
[root@linux-study ~]# touch index.txt serial # index.txt证书索引文件，serial已签署的证书编号文件
[root@linux-study ~]# echo 01 > serial # 定义编号初始编号

# 如果忘记定初始编号操作，会出现如下错误提示：
"""
Using configuration from /etc/pki/tls/openssl.cnf
/etc/pki/CA/index.txt: No such file or directory
unable to open '/etc/pki/CA/index.txt'
139985951024968:error:02001002:system library:fopen:No such file or directory:bss_file.c:398:fopen('/etc/pki/CA/index.txt','r')
139985951024968:error:20074002:BIO routines:FILE_CTRL:system lib:bss_file.c:400:
"""
```

openssl命令格式

openssl req: 生成证书签署请求

-news:新的请求

-key cakey.pem所在路径:指定私钥

-x509:生成自签署证书

-days n:有效天数


配置完成后，查看CA目录下，生成一个cacert.pem文件，该文件即为自签证书文件.

接下来在web服务器上创建web的私钥，注意，此web私钥和CA的私钥两码事.
```
[root@linux-study ~]# mkdir /etc/httpd/ssl
[root@linux-study ~]# cd /etc/httpd/ssl

# 生成私钥，文件为httpd.key
[root@linux-study ~]# (umask 077; openssl genrsa -out httpd.key 1024)

# 创建自签证书httpd.csr
[root@linux-study ~]# openssl req -new -key httpd.key -out httpd.csr
"""
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:BJ
Locality Name (eg, city) [Default City]:CY
Organization Name (eg, company) [Default Company Ltd]:xiaofei
Organizational Unit Name (eg, section) []:Ops
Common Name (eg, your name or your server's hostname) []:www.xiaofei.com
Email Address []:
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
"""
[root@xiaofei ssl]# ls

# 为客户端的请求进行签署证书httpd.crt
[root@linux-study ~]# openssl ca -in httpd.csr -out httpd.crt -days 3650 
```

## 配置使用https的虚拟主机

编辑ssl.conf配置文件
```
[root@linux-study ~]# vim /etc/httpd/conf.d/ssl.conf
"""
<VirtualHost 172.10.0.1:443>
    DocumentRoot /web/hostA
    ServerName www.xiaofei.com
</VirtualHost>

# 指定自签证书位置
SSLCertificateFile /etc/httpd/ssl/httpd.crt

# ssl的私钥文件位置
SSLCertificateKeyFile /etc/httpd/ssl/httpd.key
"""
[root@linux-study ~]# httpd -t
[root@linux-study ~]# service httpd restart
[root@linux-study ~]# ss -tnl
```

## 测试
使用opssl s_client工具
```shell
# 从服务器上下载证书
[root@linux-study ~]# scp 172.10.0.1:/etc/pki/CA/cacert.pem ./ 

# 测试连接
[root@linux-study ~]# openssl s_client -connect 172.10.0.1:443 -CAfile /root/cacert.pem 
"""
GET /index.html HTTP/1.0
Host: www.xiaofei.com
"""
```



# End