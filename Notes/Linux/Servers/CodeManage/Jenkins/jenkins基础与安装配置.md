[TOC]

# CI/CD

## 持续集成(Continuous Integration)

持续集成是指多名开发者在开发不同功能代码的过程当中，可以频繁的将代码进行合并到一起切不相互影响工作。

## 持续部署(Continuous Deployment)

是基于某种工具或平台实现代码自动化的构建、测试和部署到线上环境以实现交付高质量的产品,持续部署在某种程度上代表了一个开发团队的更新迭代速率。

# 版本控制系统

在公司的服务器安装某种程序，该程序用于按照特定格式和方式记录和保存公司多名开发人员不定期提交的源代码，且后期可以按照某种标记及方式对用户提交的数据进行还原。

## CVS(Concurrent Version System)：

早期的集中式版本控制系统，现已基本淘汰

会出现数据提交后不完整的情况

## SVN(Subversion)--集中式版本控制系统

2000年开始开发，目标就是替代CVS

集中式管理，依赖于网络，一台服务器集中管理

目前依然有部分公司在使用

## Gitlib—分布式版本控制系统

2002年由linux内核作者Linus使用C语言开发

分布式版本控制系统，不依赖于服务器，离线依然可以工作

目前广泛适用于互联网公司

功能强大，使用灵活

以后会替换SVN

## 集中式版本控制系统

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/06894a88-e576-4411-89fc-d9e5bc45998c.jpg)

任何的提交和回滚都依赖于连接服务器

SVN服务器是单点

## 分布式版本控制系统

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/b56758e7-e0dd-46c4-a961-016f1ddec3b0.jpg)

Git在每个用户都有一个完整的服务器，然后在有一个中央服务器，用户可以先将代码提交到本地，没有网络也可以先提交到本地，然后在有网络的时候再提交到中央服务器，这样就大大方便了开发者，而相比CVS和SVN都是集中式的版本控制系统，工作的时候需要先从中央服务器获取最新的代码，改完之后需要提交，如果是一个比较大的文件则需要足够快的网络才能快速提交完成，而使用分布式的版本控制系统，每个用户都是一个完整的版本库，即使没有中央服务器也可以提交代码或者回滚，最终再把改好的代码提交至中央服务器进行合并即可。

# Jenkins

1、齿轮

如果将 java / maven / ant / git / tomcat / jenkins 等等软件比喻为齿轮：如下图

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/0.5697769407227777.png)

两个软件在一起可以驱动另外一个软件：如下图

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/0.4385932260377061.png)

如果把这些软件要集成在一起工作，那么这个软件就可以存在其他软件的

中间来驱动各个软件工作：如下图：

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/0.18583339409671606.png)

jenkins就是类似于中间那个齿轮，来驱动其他软件的集成一起工作,如下图

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/0.3480943326559667.png)

## 安装jdk1.8

jenkins基于java环境，需要先安装jdk

 

```
wget http://download.oracle.com/otn-pub/java/jdk/8u181-b13/96a7b8442fe848ef90c96a2fad6ed6d1/jdk-8u181-linux-x64.tar.gz
tar xf jdk-8u181-linux-x64.tar.gz  -C /usr/local/jdk

vim /etc/profile
'''
export JAVA_HOME=/usr/local/jdk
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar
'''

source  /etc/profile && java –version
```

## 1，安装Jenkins

Jenkins下载：https://jenkins.io/download/

jdk下载：https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html 

最低配置：不少于256M内存，不低于1G磁盘，jdk版本>=8

### 1.1，两种安装方式

#### 方法1：WAR包

步骤简介：

wget在官方下载jenkins.war的包到tomcat下

修改tomcat的server.xml配置，重启tomcat

调整防火墙规则，允许端口访问

浏览器访问：http://x.x.x.x:port

优点：

只有一个war包，轻量级部署

配置过程简单，无需额外配置

对于已经部署好tomcat+jdk环境的Server，可以在10分钟内就搭建好Jenkins平台，适用于快速部署和使用；

适合新手，或者对Linux不太熟的人员

缺点：

因为是官方直接打包好的.war包，修改配置容易出现报错

运行不稳定，增加插件、修改权限等，很容易崩溃

重启jenkins服务不太方便（java -jar /xx/xx/jenkins.war --httpPort=8080）

#### 方法2：YUM安装

步骤简介：

添加官方的rpm包源，进行yum安装

编辑jenkins的/etc/init.d/jenkins程序文件，添加java路径

编辑jenkins的/etc/sysconfig/jenkins配置文件，修改端口、系统运行账户

编辑/etc/profiles文件添加jenkins的环境变量

启动jenkins服务service jenkins start

浏览器访问：

http://x.x.x.x:port

优点：

对于熟悉Linux服务配置的人员来说，轻车熟路的配置流程

可以根据Server环境，定制化的修改jenkins配置文件

可以很方便的查看服务运行状态（state）、日志、排错、重启服务

适用于Linux老司机

缺点：

配置的过程稍复杂，要修改的文件和参数

反复查看日志中的ERROR，根据模糊的错误信息，调整环境和配置

对于Linux新手来说，配置起来有点难，Troubleshouting有点懵



jenkins数据存储可能会比较大，建议新添加一块硬盘，并挂载至/data/目录，安装jenkins，确保8080端口未被占用。

 

```
mkfs.xfs  /dev/sdb
mount /dev/sdb  /data/
chown -R jenkins.jenkins /data/


# yum安装
# 安装yum源以获取最新版本
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
yum clean all && yum makecache
yum install -y jenkins

# 下载LTS版本
wget https://pkg.jenkins.io/redhat-stable/jenkins-2.150.1-1.1.noarch.rpm

rpm -ql jenkins
'''
/var/lib/jenkins  # 安装目录
/etc/sysconfig/jenkins  # 配置文件
/var/log/jenkins  # 日志目录
'''

# 修改jenkins数据存储目录为/data
vim /etc/sysconfig/Jenkins
'''
JENKINS_HOME="/data/jenkins"  # 数据目录，使用高IO大容量磁盘
JENKINS_USER="jenkins" # 启动用户
JENKINS_PORT="8080"  # 默认启动端口，确保端口未被占用
'''

vim /etc/init.d/jenkins
"""
candidates="
/usr/local/jdk/bin/java  # 新增
"""

systemctl start jenkins
systemctl status jenkins
systemctl enable jenkins

# 查询admin初始密码
tail -f /var/log/jenkins/jenkins.log
```

### 访问[http://19.87.2.11:8080](http://19.87.2.11:8080进行安装) 

### 1.2，可能遇到的错误

jenkins无法启动

Job for jenkins.service failed because the control process exited with error code. See "systemctl status jenkins.service" and "journalctl -xe" for details.

解决办法：修改jenkins启动脚本，添加java路径

 

```
vim /etc/rc.d/init.d/jenkins
'''
candidates="
/usr/local/jdk8/bin/java  # 新增
/usr/local/jdk8/jre/bin/java  # 新增
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-1.7.0/bin/java
/usr/lib/jvm/jre-1.7.0/bin/java
/usr/bin/java
'''

```

## 2，忘记管理员密码

/root/.jenkins/secrets/initialAdminPassword 

或

cat /var/jenkins_home/secrets

 

```
# 此为admin用户的目录，也可以对应自己创建的用户
cd /var/lib/jenkins/users/admin
vim config.xml
'''
# 定位到<passwordHash>那一行，删除并改为
<passwordHash>#jbcrypt:$2a$10$pDQks0ytOkCfmpdgpLygrOC3uY7i/XnZHBKRQDhrBPwKoN2f5Kz8C</passwordHash>
'''

# 重启jenkins,新密码为admin
systemctl restart jenkins
```

## 3，Jenkins配置

如果虚拟机配置较低，则启动页面可能需要很长的时间，因此推荐4G或更多内存，4C或更多CPU。

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/36f6f80c-0526-4cb9-9126-9455a8eaef1e.png)

重启完成的日志：

 

```
tail /var/log/jenkins/jenkins.log -f
```

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/a39d7935-cff7-420f-bdea-3da6aa9cc422.png)

### 3.1，验证安装密钥

在安装时会有提示，也可到/data/secrets/initialAdminPassword查看

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/3bc343cd-8537-441f-ad44-236b6065301d.png)

### 3.2，安装插件

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/e686695d-1aa5-4193-9807-631482e1f2e0.png)

插件安装过程图，如果有安装失败的可以忽略，后期还可安装

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/baab6970-48fa-46f0-b074-c2a2093f5b75.png)

安装失败的插件

未成功安装的插件最好重新安装一下

插件下载地址：http://updates.jenkins-ci.org/download/plugins/

jenkins插件目录：

默认/var/lib/jenkins/plugins，我们之前已经修改为/data/jenkins/目录下

可以将插件解压到plugins目录下，并修改属主属组

也可以通过web端，在 系统管理--插件管理--高级 中上传插件

 

```
cd /var/lib/jenkins/plugins
tar xvf jenkins_plugin.tar.gz
chown jenkins.jenkins ./* -R
/etc/init.d/jenkins start
```

## 4，创建用户

管理员为admin，密码admin

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/23896b26-698b-449a-82dd-a1219b5548b6.png)

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/5448f342-2945-4e44-b132-648039130905.png)

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/48de40c7-edfb-4542-aa46-0c7c9480e9e2.png)

## 5，邮箱设置

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/4c53740a-b47e-4ab5-9b72-8aca8cf776f2.png) --> ![img](file:///F:/My Knowledge/temp/6da646fd-52fe-4da1-bc0c-fce0d198febf/128/index_files/0ceae8cc-068a-44c5-ba6d-2b11b09af71f.png) --> 

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/c63c73b7-104f-473d-ad64-89e58abbcf11.png)

### 5.1，发送邮件设置

![img](jenkins%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/7c926948-37d5-473e-a6cb-2524608ea613.png)

填写完毕后，记得测试成功后在保存。