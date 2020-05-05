[TOC]

# 简介

SVN是Subversion的简称，是一个开放源代码的版本控制系统，相较于RCS、CVS，它采用了分支管理系统，它的设计目标就是取代CVS。互联网上很多版本控制服务已从CVS迁移到Subversion。说得简单一点SVN就是用于多个人共同开发同一个项目，共用资源的目的。

3690为svn默认端口

# 安装配置

SVN服务器有2种运行方式：独立服务器和借助apache运行(Web/DAV)

## Linux服务端

### 安装SVN服务

```shell
[root@vcs ~]# yum install -y subversion
[root@vcs ~]# rpm -ql subversion
```



### 创建SVN资源库目录

```shell
# 在/data目录下创建仓库svn
[root@vcs ~]# mkdir -pv /data/svn

# 在仓库中创建一个项目pro1
[root@vcs ~]# svnadmin create /data/svn/pro1
[root@vcs ~]# ls /data/svn/pro1
"""
conf  db  format  hooks  locks  README.txt
"""
```

说明:

conf

 |__authz：是权限控制文件  

 |__passwd：是帐号密码文件  

 |__svnserve.conf：是SVN服务配置文件 

### 配置文件

```shell
[root@vcs ~]# vim /data/svn/p1/conf/svnserve.conf
"""
[general]
# 匿名用户没有任何权限
anon-access = none

# 授权用户可写
auth-access = write

# 指定用户配置文件
password-db = passwd

# 指定权限配置文件
authz-db = authz

# 认证空间名，版本库所在目录
realm = /data/svn/p1
"""
```

最后一行的realm记得改为项目根目录。

### 创建用户

修改conf/passwd，创建3个用户

```shell
[root@vcs ~]# vim /data/svn/p1/conf/passwd
"""
[users]
user01 = user01pwd
user02 = user02pwd
user03 = user03pwd
"""
```



### 分配权限

user01和user02对该项目有读写权限，user03只读权限。

创建组admins，将user01和user02加入组，配置读写权限。

```shell
[root@vcs ~]# vim /data/svn/p1/conf/authz
"""
[groups]
admins = user01,user02

[/]
@admins = rw
user03 = r
* =
"""
```

最后一行*=很重要不能少，表示其它用户均无任何权限，

### 服务启动

```shell
# 启动服务
[root@vcs ~]# svnserve -d -r /data/svn/p1
[root@vcs ~]# ps -ef | grep svnserve

# 重启需要kill掉pid
[root@vcs ~]# ps -ef | grep svnserve
[root@vcs ~]# kill -9 1715
[root@vcs ~]# svnserve -d -r /data/svn/p1
```

-d：表示守护进程

 -r：表示在后台执行

修改配置文件需重启服务,修改用户和权限文件无需重启。

## windows服务端

VisualSVN-Server-3.8.1-x64（svn服务端）

下载地址：https://www.visualsvn.com/downloads/

打开服务端程序

![image-20200319171950630](svn%E9%83%A8%E7%BD%B2.assets/image-20200319171950630.png)

可以在窗口的右边看到版本库的一些信息,比如状态,日志,用户认证,版本库等.要建立版本库,需要右键单击左边窗口的Repositores,如图:

![image-20200319172025261](svn%E9%83%A8%E7%BD%B2.assets/image-20200319172025261.png)



### 创建项目仓库

![image-20200319172039095](svn%E9%83%A8%E7%BD%B2.assets/image-20200319172039095.png)

仓库名

![image-20200319172100304](svn%E9%83%A8%E7%BD%B2.assets/image-20200319172100304.png)

![image-20200319172110062](svn%E9%83%A8%E7%BD%B2.assets/image-20200319172110062.png)

![image-20200319172118141](svn%E9%83%A8%E7%BD%B2.assets/image-20200319172118141.png)

![image-20200319172129035](svn%E9%83%A8%E7%BD%B2.assets/image-20200319172129035.png)

### 创建用户和组

![image-20200319172143636](svn%E9%83%A8%E7%BD%B2.assets/image-20200319172143636.png)

![image-20200319172151957](svn%E9%83%A8%E7%BD%B2.assets/image-20200319172151957.png)

![image-20200319172205037](svn%E9%83%A8%E7%BD%B2.assets/image-20200319172205037.png)

![image-20200319172214974](svn%E9%83%A8%E7%BD%B2.assets/image-20200319172214974.png)

![image-20200319172224221](svn%E9%83%A8%E7%BD%B2.assets/image-20200319172224221.png)

### 给仓库设置访问用户

![image-20200319172306261](svn%E9%83%A8%E7%BD%B2.assets/image-20200319172306261.png)

![image-20200319172316165](svn%E9%83%A8%E7%BD%B2.assets/image-20200319172316165.png)

![image-20200319172327695](svn%E9%83%A8%E7%BD%B2.assets/image-20200319172327695.png)





# 附：通过HTTP协议访问SVN

### 安装apache

下载地址：http://mirrors.cnnic.cn/apache/httpd/

```shell
# 安装依赖包
yum install gcc gcc-++ make pcre-devel zlib-devel -y
tar zxvf apr-1.4.6.tar.gz
cd apr-1.4.6
./configure --prefix=/usr/local/apr
make && make install
tar zxvf apr-util-1.4.1.tar.gz
cd apr-util-1.4.1
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr

make && make install

# 安装httpd
tar zxvf httpd-2.4.7.tar.gz
cd httpd-2.4.7

./configure --prefix=/usr/local/apache --enable-dav --enable-so--enable-rewrite --enable-maintainer-mode --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util/

cp /usr/local/apache/bin/apachectl /etc/init.d/httpd

sed -i 's/#ServerName.*/ServerName localhost/' /usr/local/apache/conf/httpd.conf
```



### 安装SVN

SVN需要需要SQLite数据库支持，我们先安装SQLite

SQLite下载：http://www.sqlite.org/download.html 

SVN下载地址：http://subversion.apache.org/download

```shell
# 安装SQLite
tar zxvf sqlite-autoconf-3080200.tar.gz
cd sqlite-autoconf-3080200
./configure
make && make install


# 安装svn
tar zxvf subversion-1.8.5.tar.gz
cd subversion-1.8.5
./configure --prefix=/usr/local/subversion--with-apxs=/usr/local/apache/bin/apxs --with-apr=/usr/local/apr--with-apr-util=/usr/local/apr-util/

make && make install

echo "PATH=$PATH/:/usr/local/subversion/bin" >> /etc/profile
source /etc/profile
svnserve -version  # 显示版本信息表示正常
```

可能出现的错误

./confiure报错：

apache/bin/apxs:/usr/local/perl: bad interpreter: No such file or directory

configure: error: no - APXSrefers to an old version of Apache

这是因为apsx文件没有指定perl执行程序位置

解决方法：

```shell
vim /usr/local/apache/bin/apxs
   #!/replace/with/path/to/perl/interpreter –w  

# 将第一行修改为#!/usr/bin/perl –w 即可
```

### **apache与svn整合**

创建组、用户并加载svn库

```shell
groupadd svn
useradd -g svn -s /sbin/nologin svn
cp subversion/mod_dav_svn/.libs/mod_dav_svn.so/usr/local/apache/modules/

cp subversion/mod_authz_svn/.libs/mod_authz_svn.so /usr/local/apache/modules/

vi /usr/local/apache/conf/httpd.conf
"""
User svn
Group svn
LoadModule dav_svn_module modules/mod_dav_svn.so
LoadModule authz_svn_module modules/mod_authz_svn.so
"""
```

创建svn仓库

```shell
mkdir /svn
svnadmin create /svn/test
vi /usr/local/apache/conf/httpd.conf
"""
<Location /svn>         #url访问路径 
DAV svn                 #声明
SVNParentPath /svn      #svn仓库根目录
AuthType Basic          #基本认证
AuthName "PleaseLogin"  #登陆时提示信息
AuthUserFile/usr/local/apache/.passwd  #用户密码文件
Require valid-user      #允许所有用户访问
</Location>
"""

# 生成passwd文件
/usr/local/apache/bin/htpasswd -c -m /usr/local/apache/.passwd user1

chown :svn /usr/local/apache/.passwd
service httpd restart
```

通过浏览器访问SVN地址,并输入用户名和密码

http://IP/svn/



