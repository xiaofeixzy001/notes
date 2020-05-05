[TOC]

# 简介

SVN是Subversion的简称，是一个开放源代码的版本控制系统，相较于RCS、CVS，它采用了分支管理系统，它的设计目标就是取代CVS。互联网上很多版本控制服务已从CVS迁移到Subversion。说得简单一点SVN就是用于多个人共同开发同一个项目，共用资源的目的。

# 安装配置

SVN服务器有2种运行方式：独立服务器和借助apache运行(Web/DAV)

## 独立服务器

安装SVN服务

 

```
yum install -y subversion
rpm -ql subversion
```

创建SVN资源库目录 

 

```
# 在/data目录下创建仓库svn
mkdir -pv /data/svn

# 在仓库中创建一个项目pro1
svnadmin create /data/svn/pro1
ls /data/svn/pro1

cd /data/svn/pro1
ls
"""
conf  db  format  hooks  locks  README.txt
"""
```

说明:

conf

 |-authz是权限控制文件  

 |-passwd是帐号密码文件  

 |-svnserve.conf是SVN服务配置文件 

修改conf/svnserve.conf配置文件

 

```
[general]
# 非鉴权用户没有权限
anon-access = none

# 鉴权用户有写权限
auth-access = write

# 指定用户名口令文件名
password-db = passwd

# 指定权限配置文件名
authz-db = authz

# 认证空间名，版本库所在目录
realm = /web/mysvn 
```

修改conf/passwd，开通用户账号

 

```
[users]
admin=admin # 配置了一个用户,用户名为admin密码为admin
```

修改conf/authz文件，为用户授权

 

```
[groups]
admins = admin1,admin2

[/]
@admins = rw # 为admins组配置读写权限
admin1 = r # 为admin1用户配置读权限
```

服务启动

修改配置文件需重启服务,修改用户和权限文件无需重启

 

```
# 启动服务
svnserve -d -r /data/svn

# 重启需要kill掉pid
ss -tnlp
kill svn-pid
svnserve -d -r /data/svn
```

## 结合apache

### 安装apache

下载地址：http://mirrors.cnnic.cn/apache/httpd/

 

```
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

### **安装SVN**

SVN需要需要SQLite数据库支持，我们先安装SQLite

SQLite下载：http://www.sqlite.org/download.html 

SVN下载地址：http://subversion.apache.org/download

 

```
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

 

```
vim /usr/local/apache/bin/apxs
   #!/replace/with/path/to/perl/interpreter –w  

# 将第一行修改为#!/usr/bin/perl –w 即可
```

### **apache与svn整合**

创建组、用户并加载svn库

 

```
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

 

```
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

IP/svn/

# 企业项目

System：Centos 7.5

IP：172.16.2.10

Hostname：svn-server

Project: /data/svn/

安装

 

```
yum install -y subversion
rpm -ql subversion
```

创建版本库所在目录和版本库

 

```
# 创建版本库所在目录svn
mkdir -pv /data/svn

# 创建版本库名称pro1
svnadmin create /data/svn/pro1
ls /data/svn/pro1
```

pro1项目配置

 

```
cd /data/svn/pro1
ls
"""
conf  db  format  hooks  locks  README.txt
"""

# 项目专用配置
vim conf/svnserve.conf
"""
anon-access = none
auth-access = write
password-db = passwd
authz-db = authz
realm = /data/svn/pro1
"""

# 项目专用用户配置
vim conf/passwd
"""
[users]
user1 = user1pw
user2 = user2pw
"""

# 项目专用权限配置
vim conf/authz
"""
[groups]
g_r = user1
g_w = user2

[/]
@g_r = r
@g_w = rw
"""
```

SVN服务启动

 

```
svnserve -d -r /data/svn

# 如果修改了配置文件,则需要kill掉服务,重新开启,而用户与权限被修改则无需重启
ss -tunlp
kill svn-pid
svnserve -d -r /data/svn

```

# 备份

## 全量备份

 

```
curr=`svnlook youngest /data/svn/project/` #此处是查询工程目录的最新版本
svnadmin dump /data/svn/repos/test --revision 0:$cur --incremental >0-"$curr"svn.bak 
echo $curr >/tmp/svn_revision

```

## 增量备份

 

```
old=`cat /tmp/svn_revision`
new=`svnlook youngest /data/svn/project/`
svnadmin dump /data/svn/repos/test --revision $old:$new --incremental >$old"-"$new"svn.bak 
```

# 恢复

恢复顺序从低版本逐个恢复到高版本；即，先恢复最近的一次完整备份，然后恢复紧挨着这个文件的增量备份。

 

```
cd /data/svn/repos/ 
svnadmin create test2 
svnadmin load test2 < /data/svnback/20110719/0-1112svn.bak 
svnadmin load test2 < /data/svnback/20110719/1113-1120svn.bak
```

服务端在svn仓库有变动时自动更新代码脚本

 

```
#!/bin/sh  
REPOS="$1"  
REV="$2"  
export LC_ALL="zh_CN.UTF-8"  
export LANG="en_US.UTF-8"  
  
SVN_PATH=/usr/bin # svn安装路径  
WEB_PATH=/web/ccb # web项目所在  
SVN_USER=admin # svn用户名  
SVN_PASS=admins # svn密码  
LOG_PATH=/tmp/svn.log 
$SVN_PATH/svn update $WEB_PATH --username $SVN_USER --password $SVN_PASS --no-auth-cache >> $LOG_PATH  
exit 0
```

# 客户端

## linux

 

```
# 帮助
svn help

# 将svn服务器上pro1项目down到本地当前目录, 指定版本试用: -r 版本号
svn co svn://172.16.2.10/pro1 --username=user1 --password=user1pw

# 在本地开始编辑源代码
vim code1
"""
hello,world.
"""

svn add code1

# 将改动的文件提交到版本库,不指定文件则提交当前所有,支持正则
svn commit code1 -m "code-v1"

# 加锁/解锁
svn lock -m "lock test file" test.txt
svn unlock test.txt

# 将版本库中的code1还原大版本10,如果不指定目录,则更新所有
svn update -r 10 code1

# # 更新,于版本库同步,如果在提交的时候提示过期的话,是因为冲突,需要先update,修改文件,然后清除svn resolved,最后再提交commit,简写：svn up
svn update code1 

# 删除
svn delete svn://172.16.2.10/pro1/domain/code1 -m "delete file"
svn delete code
svn ci -m "delete file"

# 查看code所有修改日志
svn log code

# 查看详细信息
svn info code

# 比较当前版本与原始版本差异
svn diff code

# 比较m与n版本的差异
svn diff -r 200:201 code

# 将2个版本之间的修改合并到当前,一般会有冲突,需要额外处理
svn merge -r 200:205 code

```

## win

下载地址：https://tortoisesvn.net/downloads.html

包含语言包下载

程序安装

![img](svn%E5%9F%BA%E7%A1%80%E4%B8%8E%E9%85%8D%E7%BD%AE.assets/install01.gif)

语言包安装

![img](svn%E5%9F%BA%E7%A1%80%E4%B8%8E%E9%85%8D%E7%BD%AE.assets/install02.gif)

默认英文，设置为中文

![img](svn%E5%9F%BA%E7%A1%80%E4%B8%8E%E9%85%8D%E7%BD%AE.assets/changeLANG.gif)

## 客户端使用

建立一个 test的工作目录，其实就是用来存放代码的地方。

### 拉取到本地

进入test目录，空白处右键选择 "SVN checkout"

![img](svn%E5%9F%BA%E7%A1%80%E4%B8%8E%E9%85%8D%E7%BD%AE.assets/792c2b8e-ddc5-471e-b4e6-c8c5c80c2276.png)

选择OK，如果拉取成功，会在本地的test目录下生成一个.svn为目录

### 上传到服务器

在test目录下新建一个文档，然后空白处右键选择"SVN Commit..."

建议在编辑前都先进行更新的动作，选择 SVN Update