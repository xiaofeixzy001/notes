[TOC]

# CD持续部署流程图

自动部署流程图如下：

![img](Jenkins%E5%BA%94%E7%94%A8.assets/4902830f-e6a3-4dec-9b97-42167444649a.jpg)

通常一个网站完整部署的流程如下：

需求分析 --> 原型设计 --> 开发代码 --> 内网部署 --> 提交测试 --> 确认上线 --> 备份数据 --> 外网更新 --> 最终测试 --> 如果发现外网部署的代码有异常，需要及时回滚

# Jenkins+SVN+Maven

## Jenkins所需插件

安装自动部署插件和maven集成插件

Jenkins的自动部署插件：Deploy to container

maven集成插件：Maven Integration

SVN插件：Subversion Plug-in，默认已经安装

## SVN安装

```shell
[root@vcs ~]# yum install -y subversion
[root@vcs ~]# rpm -ql subversion
[root@vcs ~]# mkdir -pv /data/svn
[root@vcs ~]# svnadmin create /data/svn/pro1
[root@vcs ~]# ls /data/svn/pro1
[root@vcs ~]# vim /data/svn/p1/conf/svnserve.conf
"""
[general]
# 匿名用户没有任何权限
anon-access = none
auth-access = write

# 指定用户配置文件
password-db = passwd

# 指定权限配置文件
authz-db = authz
"""

[root@vcs ~]# vim /data/svn/p1/conf/passwd
"""
[users]
subuser = 123123
"""

[root@vcs ~]# vim /data/svn/p1/conf/authz
"""
[/]
subuser = rw
* =
"""

[root@vcs ~]# svnserve -d -r /data/svn/pro1
[root@vcs ~]# ps -ef | grep svnserve
```

连接测试：svn://192.168.100.20/pro01

## Maven安装

Maven是基于项目对象模型(POM project object model)，可以通过一小段描述信息（配置）来管理项目的构建，报告和文档的软件项目管理工具[百度百科]

确认已经安装jdk，已经环境变量中配置JAVA_HOME，已经修改Path

下载地址：http://maven.apache.org/download.cgi

```shell
[root@tomcat ~]# cd /usr/local/
[root@tomcat local]# java -version
[root@tomcat local]# tar xf src/apache-maven-3.6.3-bin.tar.gz -C .
[root@tomcat local]# ln -sv /usr/local/apache-maven-3.6.3 /usr/local/maven
[root@tomcat local]# vim /etc/profile.d/maven.sh
"""
# set maven env
export MAVEN_HOME=/usr/local/maven
export PATH=$MAVEN_HOME/bin:$PATH
"""

[root@tomcat local]# source /etc/profile
[root@tomcat local]# mvn -version
```

默认仓库位置：maven/conf/settings.xml

```xml
<!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
```



## Jenkins配置SVN

### 新建任务

==新建任务Pro1，自由风格。==

![image-20200321130110981](Jenkins%E5%BA%94%E7%94%A8.assets/image-20200321130110981.png)



### 配置svn仓库

在新建的项目中，定位到源码管理：

==配置SVN：==

![image-20200322150221645](Jenkins%E5%BA%94%E7%94%A8.assets/image-20200322150221645.png)



==添加svn账号：==

![image-20200321132543878](Jenkins%E5%BA%94%E7%94%A8.assets/image-20200321132543878.png)

其他说明：

==Local module directory==：jenkins的本地工作目录，拉取svn里面数据存储路径；

==Repository depth==：需要检出的文件夹深度，一般设为infinity（配置文件夹下的所有文件，包括子文件夹）具体说明可见插件帮助；

==Check-out Strategy==：更新svn到本地的方式：

- **Use 'svn update' as much as possible**

  第一次build，将本地workspace下的所有文件清空，然后从svn上check out一份完整的项目到workspace下，第n次build(除第一次)，update前不会revert；

  第一次发布的时候，会把工作目录下的所有文件清空，然后check-out一份完整的项目到工作目录下；

  以后更新的时候，不会判断已有文件是否在svn里存在。比如工作目录下的文件123在svn里不存在，那么更新的时候不会删除123；

  不会判断工作目录下的文件是否被改动，只会判断svn是否有新版本需要更新。比如工作目录下的文件zzz.txt内容为zzz，svn上的zzz.txt内容为空，如果svn上zzz.txt没有新版本，则在更新的时候不会更新zzz.txt，也就是说如果手动修改了工作目录下的文件，如果此文件在svn上没有出现新版本，就不会更新。一旦svn上的zzz.txt有新版本后就会更新工作目录的zzz.txt，这时工作目录下会生成如下几个文件：zzz.txt、zzz.txt.mine、zzz.txt.r223、zzz.txt.r224，其中zzz.txt.r223为svn上老版本、zzz.txt.r224为svn上新版本、zzz.txt.mine为工作目录上的zzz.txt的副本、zzz.txt记录了文件变化。

  svn上删除了文件，更新的时候，工作目录里的此文件也会被删除。但是如上例中的zzz.txt手动修改过，已经和svn上的不一样了，这时将不会被删除。

- **Always check out a fresh copy**

  第一次build，将本地workspace下的所有文件清空，然后从svn上check out一份完整的项目到workspace下，第n次build(除第一次)，删除workspace下的所有文件，然后重新check out一份完整的项目到workspace下；

  第一次发布的时候，会把工作目录下的所有文件清空，然后check-out一份完整的项目到工作目录下；

  每一次更新的时候，都会先清除工作目录下的所有文件，然后重新check-out一份完整的项目到工作目录下。

- **Do not touch working copy, it is updated by other script**

  脚本方式。脚本方式下，只需了解svn所支持的命令列表，即可在shell中自行配置。其自由度相比插件更高，可以方便地对特殊需求进行处理。

  常用命令包括：

  检出  svn checkout

  更新  svn update

  取消本地修改 svn revert

  清理本地项目 svn cleanup

  向代码仓库新增文件 svn add

  提交到代码仓库  svn commit

  脚本方式下取得svn 版本号可以通过shell的sed命令： 

```shell
build_svn_version=`svnversion ${clientPath} |sed 's/^.*://' |sed 's/[A-Z]*$//'`
# 上述语句中${clientPath}为本地svn项目的根目录（即包含了.svn隐藏文件夹的目录），build_svn_version存放了取出的svn版本号
```



- **Emulate clean checkout by first deleting unversioned/ignored files, then 'svn update'**

  第一次build，将workspace下的所有文件清空，然后从svn上check out一份完整的项目到workspace下，第n次build(除第一次)，update前先删除unversioned/ignored文件；

  第一次发布的时候，会把工作目录下的所有文件清空，然后check-out一份完整的项目到工作目录下；

  以后更新的时候会判断工作目录下的文件是否在svn里存在，如果不存在则删除，如果存在且有新版本则更新；

  会判断工作目录下的文件是否被改动，不管有没有新版本，都会还原为svn上的最新版本；

  svn上删除了文件，更新的时候，工作目录里的此文件也会被删除；

- **Use 'svn update' as much as possible, with 'svn revert' before update**

  第一次build，将workspace下的所有文件清空，然后从svn上check out一份完整的项目到workspace下，第n次build(除第一次)，update前先revert；

  第一次发布的时候，会把工作目录下的所有文件清空，然后check-out一份完整的项目到工作目录下；

  以后更新的时候不会判断工作目录下的文件是否在svn里存在；

  会判断工作目录下的文件是否被改动，不管有没有新版本，都会还原为svn上的最新版本；

  svn上删除了文件，更新的时候，工作目录里的此文件也会被删除；

### SVN测试

立即构建

![image-20200322122053394](Jenkins%E5%BA%94%E7%94%A8.assets/image-20200322122053394.png)

查看/root/.jenkins/workspace/Pro1目录验证

## Jenkins配置maven

### 全局配置

系统管理 -> 全局配置 -> Maven配置和JDK

![image-20200321124134756](Jenkins%E5%BA%94%E7%94%A8.assets/image-20200321124134756.png)

系统管理 -> 全局配置 -> Maven安装  -> 新增Maven

![image-20200321124241911](Jenkins%E5%BA%94%E7%94%A8.assets/image-20200321124241911.png)



### 项目配置

上传一个maven项目到svn服务器，然后再次点立即构建，载jenkins上查看上传的项目是否同步成功。

然后定位到构建选项，添加maven配置项：

![image-20200321135359033](Jenkins%E5%BA%94%E7%94%A8.assets/image-20200321135359033.png)

clean：清除

install：安装

注意：

Repository URL这一项要填写项目的绝对路径，也就是pom.xml文件所在的绝对路径，否则编译构建会出错。

保存后点击立即构建

![image-20200322125715612](Jenkins%E5%BA%94%E7%94%A8.assets/image-20200322125715612.png)

至此手动构建已配置完成，接下来配置触发器实现自动构建。

### 构建后操作

==添加tomcat用户==

tomcat/conf/tomcat-user.xml中配置

tomcat_user/123456

![image-20200322141234686](Jenkins%E5%BA%94%E7%94%A8.assets/image-20200322141234686.png)

tomcat URL配置，当项目构建后，会自动部署项目到指定的tomcat项目目录下。

![image-20200322141528668](Jenkins%E5%BA%94%E7%94%A8.assets/image-20200322141528668.png)



构建测试

![image-20200322142755389](Jenkins%E5%BA%94%E7%94%A8.assets/image-20200322142755389.png)

查看tomcat的webapps目录，是否已经有项目webtest存在。

http://192.168.100.20:8080/webtest/

### 构建触发器

触发器方式有很多，这里选择如下：![image-20200322143355019](Jenkins%E5%BA%94%E7%94%A8.assets/image-20200322143355019.png)

意思是说：当访问地址 http://192.168.100.20:8080/jenkins/job/Pro1/build?token=WEBTEST_TOKEN 后，自动触发构建操作。

### 钩子程序配置

利用SVN自带的钩子文件

```shell
[root@tomcat ~]# cd /data/svn/pro1/hooks
[root@tomcat hooks]# ll
"""
post-commit.tmpl          post-unlock.tmpl  pre-revprop-change.tmpl
post-lock.tmpl            pre-commit.tmpl   pre-unlock.tmpl
post-revprop-change.tmpl  pre-lock.tmpl     start-commit.tmpl
"""
```

post-commit.tmpl：commit提交后操作

```shell
[root@tomcat hooks]# cp post-commit.tmpl post-commit
[root@tomcat hooks]# chmod 755 post-commit
[root@tomcat hooks]# vim post-commit
"""
/usr/bin/curl -X post -v -u xiaofei:xiaofei http://192.168.100.20:8080/jenkins/job/Pro1/build?token=WEBTEST_TOKEN
"""
```

注意：所有以.tmpl为扩展名的SVN默认都不会识别，只有不带扩展名的才可以被识别并执行。

测试



# Jenkins+Git

## **安装gitlab插件**

jenkins需要安装gitlab插件：gitLab Plugin 和 gitlab Hook Plugin，新版本默认已经安装，可在已安装列表查看验证。

![img](Jenkins%E5%BA%94%E7%94%A8.assets/36e75da8-c2d1-4d22-8002-1749477c71f1.png)

插件安装界面，会额外安装一些依赖关系的插件，jenkins插件有的基于ruby开发，所有会安装ruby环境

## **配置SSH**

在使用jenkins自动构建并远程登录服务器进行发布应用的时候，需要使用SSH公钥认证来解决登录服务器的问题。

ssh秘钥认证的原理简单来说就是用公钥加密，私钥解密。

配置过程：在两个服务器上分别建立一个普通用户，这里创建为xiaofei。可以将私钥配置在gitlab上，公钥配置在jenkins上，反过来也可以，只要实现jenkins连接gitlab免密即可。

因为仅需jenkins主动去连接git，所以这里以jenkins配置私钥，gitlab配置公钥为例

```shell
# jenkins服务器
useradd xiaofei
su xiaofei
ssh-keygen -t rsa
# 一路回车, 默认路径和文件名, 不要密码

cat .ssh/id_rsa
'''
# 复制私钥添加到jenkins上
'''

cat .ssh/id_rsa.pub
'''
# 复制公钥添加到gitlab上
'''
```

### jenkins添加秘钥：

jenkins的ssh插件：Publish over SSH

左侧找到 凭据 -> 系统 -> 全局凭据 -> 添加凭据

![img](Jenkins%E5%BA%94%E7%94%A8.assets/5741a6ae-38e1-4063-8372-53f0552a54b5.png)

![img](Jenkins%E5%BA%94%E7%94%A8.assets/24d2279c-87ff-4533-bfa3-846e3a4c9f20.png)

找到目标项目，点击项目的配置，选择源码管理

![img](Jenkins%E5%BA%94%E7%94%A8.assets/adc4c515-f744-4c53-b2a1-62752128612f.png)

![img](Jenkins%E5%BA%94%E7%94%A8.assets/7a1cc775-83b3-4ee5-8a3b-92f03a5998e4.png)

确保没有错误提示即可。

### git添加秘钥：

登陆账号，然后在账号头像下拉框中，选择settings --> ssh keys

![img](Jenkins%E5%BA%94%E7%94%A8.assets/56dce437-67ed-434e-bc13-5e5d94fc0f02.png)

![img](Jenkins%E5%BA%94%E7%94%A8.assets/3b42be9c-4ade-4b37-be93-34ed0f0363bc.png)

测试：

在jenkins上拉取代码(jenkins以普通用户连接gitlab，验证公钥，拉取代码)

项目ssh地址：git@172.22.4.12:web1/myweb.git

未配置公钥前：

![img](Jenkins%E5%BA%94%E7%94%A8.assets/a0b3c94f-4340-4318-904e-83966ea042b3.png)

配置公钥后：

![img](Jenkins%E5%BA%94%E7%94%A8.assets/bc49ebac-bffd-40df-9ede-f035d31e4f90.png)

测试无误后，到jenkins上配置这个用户的私钥

### **构建项目**

配置好ssh key后，选择一个项目，点击立即构建，查看构建日志

![img](Jenkins%E5%BA%94%E7%94%A8.assets/5f2974b1-ab0b-4e4b-bd78-2b5daace42ed.png)

然后验证是否已经pull下来

```shell
[xiaofei@jenkins ~]$ cd /var/lib/jenkins/workspace/test-demo
[xiaofei@jenkins test-demo]$ ll
total 4
-rw-r--r-- 1 jenkins jenkins 40 Dec  3 16:28 index.html
[xiaofei@jenkins test-demo]$ cat index.html
<h1>
    1111
</h1>
<h1>
    2222
</h1>
```







# 附：mvn命令管理项目

## mvn命令

mvn compile

编译，src/main/java目录java源码编译生成class(target目录下)

mvn test

测试，src/test/java 目录编译

mvn clean

清理，删除target目录，也就是将class文件等删除

mvn package

打包，生成压缩文件：java项目#jar包；web项目#war包，也是放在target目录下

mvn install

安装，将压缩文件(jar或者war)上传到本地仓库

mvn deploy

部署|发布，将压缩文件上传私服



==JAVA项目转换Eclipse工程==

mvn eclipse:eclipse

mvn eclipse:clean

清除eclipse设置信息，又从eclipse工程转换为maven原生项目了　　　　

====JAVA项目转换IDEA工程==

mvn idea:idea

mvn idea:clean



## 创建项目

==创建web项目webapp==

第一次编译因为要下载一些jar包，会比较慢。

```shell
[root@tomcat maven_pro]# mvn archetype:generate  -DgroupId=com.xiaofei.maven.quickstart -DartifactId=webapp -DarchetypeArtifactId=maven-archetype-webapp -Dversion=0.0.1-snapshot -DarchetypeCatalog=internal

[root@tomcat maven_pro]# tree webapp/
"""
webapp/
├── pom.xml
└── src
    └── main
        ├── resources
        └── webapp
            ├── index.jsp
            └── WEB-INF
                └── web.xml
"""

# vim webapp/src/main/webapp/index.jsp
"""

"""

[root@tomcat maven_pro]# cd webapp/
[root@tomcat webapp]# ls  # pom.xml很重要
[root@tomcat webapp]# mvn clean
[root@tomcat webapp]# mvn compile
[root@tomcat webapp]# mvn package
```

参数说明 ：

archetype:generate：创建项目

-DgroupId=com.xiaofei.maven.quickstart ：创建该maven项目时的groupId是什么，一般使用包名的写法。因为包名是用公司的域名的反写，独一无二

-DartifactId=simple：创建该maven项目时的artifactId是什么，就是项目名称

-DarchetypeArtifactId=maven-archetype-quickstart：表示创建的是[maven]java项目.

-DarchetypeCatalog=internal：禁止从远程服务器上获取catalog，解决卡住问题。

## Tomcat验证

各web服务器准备tomcat运行环境

```shell
useradd  www  -u 2000
mkdir /apps && cd /apps

tar xvf apache-tomcat-7.0.59.tar.gz
ln -sv /apps/apache-tomcat-7.0.59 /apps/tomcat

# 准备tomcat启动脚本：
cp /root/tomcatd /etc/init.d/
```

部署app

确认各web服务器访问正常

```shell
mkdir  /data/tomcat_appdir # 保存web压缩包
mkdir /data/tomcat_webdir # 保存解压后的web目录
cd tomcat_webdir
mkdir myapp
echo tomcatX > /apps/tomcat_webdir/myapp/index.html
```

测试访问：182.168.10.134:8080/myapp

