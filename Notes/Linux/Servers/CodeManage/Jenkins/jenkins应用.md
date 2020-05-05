[TOC]

# **系统环境**

自动部署流程图如下：

![img](jenkins%E5%BA%94%E7%94%A8.assets/4902830f-e6a3-4dec-9b97-42167444649a.jpg)

通常一个网站完整部署的流程如下：

需求分析 --> 原型设计 --> 开发代码 --> 内网部署 --> 提交测试 --> 确认上线 --> 备份数据 --> 外网更新 --> 最终测试 --> 如果发现外网部署的代码有异常，需要及时回滚

# jenkins + gitlab

## **安装gitlab插件**

jenkins需要安装gitlab插件：gitLab Plugin 和 gitlab Hook Plugin，新版本默认已经安装，可在已安装列表查看验证。

![img](jenkins%E5%BA%94%E7%94%A8.assets/36e75da8-c2d1-4d22-8002-1749477c71f1.png)

插件安装界面，会额外安装一些依赖关系的插件，jenkins插件有的基于ruby开发，所有会安装ruby环境

## **配置SSH**

在使用jenkins自动构建并远程登录服务器进行发布应用的时候，需要使用SSH公钥认证来解决登录服务器的问题。

ssh秘钥认证的原理简单来说就是用公钥加密，私钥解密。

配置过程：在两个服务器上分别建立一个普通用户，这里创建为xiaofei。可以将私钥配置在gitlab上，公钥配置在jenkins上，反过来也可以，只要实现jenkins连接gitlab免密即可。

因为仅需jenkins主动去连接git，所以这里以jenkins配置私钥，gitlab配置公钥为例

 

```
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

![img](jenkins%E5%BA%94%E7%94%A8.assets/5741a6ae-38e1-4063-8372-53f0552a54b5.png)

![img](jenkins%E5%BA%94%E7%94%A8.assets/24d2279c-87ff-4533-bfa3-846e3a4c9f20.png)

找到目标项目，点击项目的配置，选择源码管理

![img](jenkins%E5%BA%94%E7%94%A8.assets/adc4c515-f744-4c53-b2a1-62752128612f.png)

![img](jenkins%E5%BA%94%E7%94%A8.assets/7a1cc775-83b3-4ee5-8a3b-92f03a5998e4.png)

确保没有错误提示即可。

### git添加秘钥：

登陆账号，然后在账号头像下拉框中，选择settings --> ssh keys

![img](jenkins%E5%BA%94%E7%94%A8.assets/56dce437-67ed-434e-bc13-5e5d94fc0f02.png)

![img](jenkins%E5%BA%94%E7%94%A8.assets/3b42be9c-4ade-4b37-be93-34ed0f0363bc.png)

测试：

在jenkins上拉取代码(jenkins以普通用户连接gitlab，验证公钥，拉取代码)

项目ssh地址：git@172.22.4.12:web1/myweb.git

未配置公钥前：

![img](jenkins%E5%BA%94%E7%94%A8.assets/a0b3c94f-4340-4318-904e-83966ea042b3.png)

配置公钥后：

![img](jenkins%E5%BA%94%E7%94%A8.assets/bc49ebac-bffd-40df-9ede-f035d31e4f90.png)

测试无误后，到jenkins上配置这个用户的私钥

### **构建项目**

配置好ssh key后，选择一个项目，点击立即构建，查看构建日志

![img](jenkins%E5%BA%94%E7%94%A8.assets/5f2974b1-ab0b-4e4b-bd78-2b5daace42ed.png)

然后验证是否已经pull下来

 

```
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

# **jenkins + svn**

Subversion Plug-in,默认已经安装。

## 配置

在新建的项目中，定位到源码管理：

![img](jenkins%E5%BA%94%E7%94%A8.assets/8e5739b7-e1fa-4fcb-8927-6cff769497b7.png)

其他说明：

Local module directory: jenkins的本地工作目录，拉取svn里面数据存储路径

Repository depth：需要检出的文件夹深度，一般设为infinity（配置文件夹下的所有文件，包括子文件夹）具体说明可见插件帮助

Check-out Strategy：更新svn到本地的方式

\- Use 'svn update' as much as possible

第一次build，将workspace下的所有文件清空，然后从svn上check out一份完整的项目到workspace下，第n次build(除第一次)，update前不会revert

第一次发布的时候，会把工作目录下的所有文件清空，然后check-out一份完整的项目到工作目录下；

以后更新的时候，不会判断已有文件是否在svn里存在。比如工作目录下的文件123在svn里不存在，那么更新的时候不会删除123。

不会判断工作目录下的文件是否被改动，只会判断svn是否有新版本需要更新。比如工作目录下的文件zzz.txt内容为zzz，svn上的zzz.txt内容为空，如果svn上zzz.txt没有新版本，则在更新的时候不会更新zzz.txt，也就是说如果手动修改了工作目录下的文件，如果此文件在svn上没有出现新版本，就不会更新。一旦svn上的zzz.txt有新版本后就会更新工作目录的zzz.txt，这时工作目录下会生成如下几个文件：zzz.txt、zzz.txt.mine、zzz.txt.r223、zzz.txt.r224，其中zzz.txt.r223为svn上老版本、zzz.txt.r224为svn上新版本、zzz.txt.mine为工作目录上的zzz.txt的副本、zzz.txt记录了文件变化。

svn上删除了文件，更新的时候，工作目录里的此文件也会被删除。但是如上例中的zzz.txt手动修改过，已经和svn上的不一样了，这时将不会被删除。

\- Always check out a fresh copy

第一次build，将workspace下的所有文件清空，然后从svn上check out一份完整的项目到workspace下，第n次build(除第一次)，删除workspace下的所有文件，然后重新check out一份完整的项目到workspace下。

第一次发布的时候，会把工作目录下的所有文件清空，然后check-out一份完整的项目到工作目录下；

每一次更新的时候，都会先清除工作目录下的所有文件，然后重新check-out一份完整的项目到工作目录下。

\- Do not touch working copy, it is updated by other script

脚本方式

脚本方式下，只需了解svn所支持的命令列表，即可在shell中自行配置。其自由度相比插件更高，可以方便地对特殊需求进行处理。

常用命令包括：

检出  svn checkout

更新  svn update

取消本地修改 svn revert

清理本地项目 svn cleanup

向代码仓库新增文件 svn add

提交到代码仓库  svn commit

脚本方式下取得svn 版本号可以通过shell的sed命令： 

 

```
build_svn_version=`svnversion ${clientPath} |sed 's/^.*://' |sed 's/[A-Z]*$//'`
# 上述语句中${clientPath}为本地svn项目的根目录（即包含了.svn隐藏文件夹的目录），build_svn_version存放了取出的svn版本号
```

\- Emulate clean checkout by first deleting unversioned/ignored files, then 'svn update'

第一次build，将workspace下的所有文件清空，然后从svn上check out一份完整的项目到workspace下，第n次build(除第一次)，update前先删除unversioned/ignored文件。

第一次发布的时候，会把工作目录下的所有文件清空，然后check-out一份完整的项目到工作目录下；

以后更新的时候会判断工作目录下的文件是否在svn里存在，如果不存在则删除，如果存在且有新版本则更新。

会判断工作目录下的文件是否被改动，不管有没有新版本，都会还原为svn上的最新版本。

svn上删除了文件，更新的时候，工作目录里的此文件也会被删除。

\- Use 'svn update' as much as possible, with 'svn revert' before update

第一次build，将workspace下的所有文件清空，然后从svn上check out一份完整的项目到workspace下，第n次build(除第一次)，update前先revert。

第一次发布的时候，会把工作目录下的所有文件清空，然后check-out一份完整的项目到工作目录下；

以后更新的时候不会判断工作目录下的文件是否在svn里存在。

会判断工作目录下的文件是否被改动，不管有没有新版本，都会还原为svn上的最新版本。

svn上删除了文件，更新的时候，工作目录里的此文件也会被删除。

配置好已经点击"立即构建"，查看workspace工作目录下项目

# Maven

Maven是基于项目对象模型(POM project object model)，可以通过一小段描述信息（配置）来管理项目的构建，报告和文档的软件项目管理工具[百度百科]

## 安装

确认已经安装jdk，已经环境变量中配置JAVA_HOME，已经修改Path

下载地址：http://maven.apache.org/download.cgi

 

```
java -version
tar xf apache-maven-3.6.0-bin.tar.gz -C /usr/local/
mv /usr/local/apache-maven-3.6.0-bin /usr/local/maven
vim /etc/profile.d/maven.sh
"""
# set maven env
export MAVEN_HOME=/usr/local/maven
export PATH=$MAVEN_HOME/bin:$PATH
"""

source /etc/profile
mvn -version
```

仓库位置：conf/settings.xml

 

```
<!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->

```

## jenkins的maven插件

manven插件：Maven Integration

![img](jenkins%E5%BA%94%E7%94%A8.assets/88d10f7f-5e2c-482f-bbed-535fd76bd17a.jpg)

安装自动部署 Deploy to container 插件

这个是支持将代码部署到tomcat容器的

## jenkins配置

系统管理 -> 全局配置 -> Maven Configuration

![img](jenkins%E5%BA%94%E7%94%A8.assets/eb1ab7bc-45f3-4759-a396-ecf369918c62.png)

系统管理 -> 全局配置 -> maven

![img](jenkins%E5%BA%94%E7%94%A8.assets/8fc1d05c-080b-483c-b01c-109c1418beae.png)

使用maven编译

（maven编译  与 测试 test 和打包 package 和 部署 install 类似，不再赘述 ）

在项目的配置页面中有个maven配置：里面只有一个clean   就是清除以前的构建信息：

之前我使用了clean   package来编译打包：结果如下图：

![img](data:image/png;base64,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)

当执行完只有clean的时候，之前构建的信息就被删除了，也就是说，clean和package是2个选项，表示删除之前构建信息并构建包

这里的Goals  就是用maven 编译要用的命令如下图中所示：

jenkins使用maven编译以invoke top-level Maven targets中的Goals为准：

![img](data:image/png;base64,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)

![img](data:image/png;base64,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)

# 附：mvn命令管理项目

## 创建项目

第一次编译因为要下载一些jar包，会比较慢。

创建java项目

 

```
mvn archetype:generate -DgroupId=com.xiaofei.maven.quickstart -DartifactId=simple -DarchetypeArtifactId=maven-archetype-quickstart
```

参数说明：

archetype:generate：创建项目

-DgroupId=com.wuhao.maven.quickstart ：创建该maven项目时的groupId是什么，一般使用包名的写法。因为包名是用公司的域名的反写，独一无二

-DartifactId=simple：创建该maven项目时的artifactId是什么，就是项目名称

-DarchetypeArtifactId=maven-archetype-quickstart：表示创建的是[maven]java项目

创建完成后项目结构

 

```
tree simple/
"""
simple/
├── pom.xml   # 核心配置
└── src
    ├── main  # java源码
    │   └── java  
    │       └── com
    │           └── xiaofei
    │               └── maven
    │                   └── quickstart
    │                       └── App.java
    └── test  # 测试源码
        └── java
            └── com
                └── xiaofei
                    └── maven
                        └── quickstart
                            └── AppTest.java
"""
```

创建web项目

 

```
mvn archetype:generate  -DgroupId=com.xiaofei.maven.quickstart -DartifactId=myWebApp -DarchetypeArtifactId=maven-archetype-webapp -Dversion=0.0.1-snapshot

tree myWebApp/
"""
myWebApp/
├── pom.xml
└── src
    └── main
        ├── resources
        └── webapp
            ├── index.jsp
            └── WEB-INF
                └── web.xml
"""
```

命令操作maven java或web项目 

编译：mvn compile　　# src/main/java目录java源码编译生成class （target目录下）

测试：mvn test　　　 # src/test/java 目录编译

清理：mvn clean　　　# 删除target目录，也就是将class文件等删除

打包：mvn package　　# 生成压缩文件：java项目#jar包；web项目#war包，也是放在target目录下

安装：mvn install　　# 将压缩文件(jar或者war)上传到本地仓库

部署|发布：mvn deploy  # 将压缩文件上传私服

maven java或web项目转换Eclipse工程

mvn eclipse:eclipse

mvn eclipse:clean  # 清除eclipse设置信息，又从eclipse工程转换为maven原生项目了　　　　

转换IDEA工程

mvn idea:idea

mvn idea:clean　　同上　

示例：将maven java项目打包上传到本地仓库供别人调用

注：使用命令时，必须在maven java项目的根目录下，即可以看到pom.xml

 

```
pwd
"""
/root/temp/tmp/myWebApp
"""
mvn install

ll
"""
total 4
-rw-r--r-- 1 root root 729 Dec  7 10:03 pom.xml
drwxr-xr-x 3 root root  18 Dec  7 10:03 src
drwxr-xr-x 5 root root  79 Dec  7 10:08 target  # 会多出此目录，打包后的目录
"""

ls target/
"""
classes  maven-archiver  myWebApp  myWebApp.war
"""
```

参考博客：https://www.cnblogs.com/whgk/p/7112560.html

## Tomcat验证

各web服务器准备tomcat运行环境

 

```
useradd  www  -u 2000
mkdir /apps && cd /apps

tar xvf apache-tomcat-7.0.59.tar.gz
ln -sv /apps/apache-tomcat-7.0.59 /apps/tomcat

# 准备tomcat启动脚本：
cp /root/tomcatd /etc/init.d/
```

部署app

确认各web服务器访问正常

 

```
mkdir  /data/tomcat_appdir # 保存web压缩包
mkdir /data/tomcat_webdir # 保存解压后的web目录
cd tomcat_webdir
mkdir myapp
echo tomcatX > /apps/tomcat_webdir/myapp/index.html
```

测试访问：182.168.10.134:8080/myapp

