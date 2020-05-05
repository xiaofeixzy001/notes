[TOC]

# harbor简介

Harbor是一个用于存储和分发Docker镜像的企业级Docker Registry服务器，可以实现 images 的私有存储和日志统计权限控制等功能，并支持创建多项目(Harbor 提出的概念)，基于官方 Registry V2 实现。 下载地址：https://github.com/vmware/harbor/releases

安装文档：https://github.com/vmware/harbor/blob/master/docs/installation_guide.md

## Harbor特性

基于角色控制

用户和仓库都是基于项目进行组织的， 而用户基于项目可以拥有不同的权限。一个用户可以对多个镜像仓库在同一命名空间（project）里有不同的权限。

基于镜像的复制策略

镜像可以在多个Harbor实例之间进行复制。

支持LDAP/AD

Harbor的用户授权可以使用已经存在LDAP用户，Harbor可以集成企业内部已有的AD/LDAP，用于鉴权认证管理。

用户UI

用户可以轻松的浏览、搜索镜像仓库以及对项目进行管理。

审计管理

所有针对镜像仓库的操作都可以被记录追溯，用于审计管理。

国际化

已拥有英文、中文、德文、日文和俄文的本地化版本。更多的语言将会添加进来。

轻松的部署功能

Harbor提供了online、offline安装，除此之外还提供了virtualappliance安装。

RESTful API - RESTful API

提供给管理员对于Harbor更多的操控, 使得与其它管理软件集成变得更容易。

## Harbor认证过程

1、dockerdaemon从docker registry拉取镜像。

2、如果dockerregistry需要进行授权时，registry将会返回401 Unauthorized响应，同时在响应中包含了docker client如何进行认证的信息。

3、dockerclient根据registry返回的信息，向auth server发送请求获取认证token。

4、auth server则根据自己的业务实现去验证提交的用户信息是否存符合业务要求。

5、用户数据仓库返回用户的相关信息。

6、auth server将会根据查询的用户信息，生成token令牌，以及当前用户所具有的相关权限信息。

上述就是完整的授权过程.当用户完成上述过程以后便可以执行相关的pull/push操作。认证信息会每次都带在请求头中。

harbor认证流程图

![img](Harbor%E5%9F%BA%E7%A1%80.assets/1264f389-88cd-4eeb-a1a3-67cdf787830d.jpg)

harbor架构图

![img](Harbor%E5%9F%BA%E7%A1%80.assets/cbd35236-fd85-4b6f-82f2-7e8365401c11.jpg)

A、首先，请求被代理容器监听拦截，并跳转到指定的认证服务器。

B、 如果认证服务器配置了权限认证，则会返回401。通知dockerclient在特定的请求中需要带上一个合法的token。而认证的逻辑地址则指向架构图中的core services。

C、 当docker client接受到错误code。client就会发送认证请求(带有用户名和密码)到coreservices进行basic auth认证。

D、 当C的请求发送给ngnix以后，ngnix会根据配置的认证地址将带有用户名和密码的请求发送到core serivces。

E、 coreservices获取用户名和密码以后对用户信息进行认证(自己的数据库或者介入LDAP都可以)。成功以后，返回认证成功的信息。

从上面的docker registry 和 Harbor的架构图、流程图可以看出Harbor的扩展部分为coreservices。这部分实现了用户的认证，添加了UI模块已经webhook。

## Harbor组件

Harbor的每个组件都是以Docker容器的形式构建的，使用Docker Compose来对它进行部署。用于部署Harbor的Docker Compose模板位于 /Deployer/docker-compose.yml，由5个容器组成：

Proxy：由Nginx 服务器构成的反向代理。

Registry：由Docker官方的开源registry镜像构成的容器实例。

UI：即架构中的core services， 构成此容器的代码是Harbor项目的主体。

MySQL：由官方MySQL镜像构成的数据库容器。

Log：运行着rsyslogd的容器，通过log-driver的形式收集其他容器的日志。

这几个容器通过Docker link的形式连接在一起，在容器之间通过容器名字互相访问。对终端用户而言，只需要暴露proxy （即Nginx）的服务端口。

# 安装和使用

安装环境：

系统：CentOS Linux release 7.5.1804 (Core)

本次使用当前harbor最新的稳定版本1.2.2离线安装包，具体名称为harbor-offline-installer-v1.2.2.tgz

## 1，安装docker

 

```
wget https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm

yum install docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm

systemctl start docker   # 启动docker服务 
systemctl enable docker  # docker开机启动

systemctl status docker  # 查看docker运行状态
```

 

```
wget https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm
yum install docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm
systemctl start docker   # 启动docker服务 
systemctl enable docker  # docker开机启动
systemctl status docker  # 查看docker运行状态
```

## 2，安装配置harbor 

 

```
cd /usr/local/src
wget https://github.com/vmware/harbor/releases/download/v1.2.2/harbor-offline-installer-v1.2.2.tgz
tar xf harbor-offline-installer-v1.2.2.tgz
ln -sv /usr/local/src/harbor /applications/harbor
cd /applications/harbor
yum install -y python-pip
pip install docker-compose

vim harbor.cfg
"""
hostname = 172.16.2.103  # hostname设置访问地址，可以使用ip、域名，不可以设置为127.0.0.1或localhost
harbor_admin_password = 123456  # mysql数据库root用户默认密码root123，实际使用时修改下
ui_url_protocol = http
db_password = root123
max_job_workers = 3 
customize_crt = on
ssl_cert = /data/cert/server.crt
ssl_cert_key = /data/cert/server.key
secretkey_path = /data
admiral_url = NA
clair_db_password = password

# 邮件设置，发送重置密码邮件时使用
email_identity = harbor
email_server = smtp.163.com
email_server_port = 25
email_username = rooroot@163.com
email_password = zhang@123
email_from = admin <rooroot@163.com>
email_ssl = false

harbor_admin_password = zhang@123  # 启动Harbor后，管理员UI登录的密码，默认是Harbor12345
auth_mode = db_auth  # 认证方式，这里支持多种认证方式，如LADP、本次存储、数据库认证。默认是db_auth，mysql数据库认证

# LDAP认证时配置项
ldap_url = ldaps://ldap.mydomain.com
ldap_basedn = ou=people,dc=mydomain,dc=com
ldap_uid = uid 
ldap_scope = 3 
ldap_timeout = 5

self_registration = on  # 是否开启自注册
token_expiration = 30  # Token有效时间，默认30分钟
project_creation_restriction = everyone  # 用户创建项目权限控制，默认是everyone（所有人），也可以设置为adminonly（只能管理员）

verify_remote_cert = on
"""

# 官方构建harbor和启动方式，推荐此方法，会下载官方的docker镜像，在1.1.0版本之后，Harbor已经与Notary进行了集成，但默认情况下安装不包括公证(https)服务。
./install.sh

docker-compose start
"""
Starting log         ... done
Starting redis       ... done
Starting adminserver ... done
Starting registry    ... done
Starting ui          ... done
Starting mysql       ... done
Starting jobservice  ... done
Starting proxy       ... done
"""

# 查看本地镜像
docker images

# 查看端口
ss -tnlp
```

 

```
cd /usr/local/src
wget https://github.com/vmware/harbor/releases/download/v1.2.2/harbor-offline-installer-v1.2.2.tgz
tar xf harbor-offline-installer-v1.2.2.tgz
ln -sv /usr/local/src/harbor /applications/harbor
cd /applications/harbor
yum install -y python-pip
pip install docker-compose
vim harbor.cfg
"""
hostname = 172.16.2.103  # hostname设置访问地址，可以使用ip、域名，不可以设置为127.0.0.1或localhost
harbor_admin_password = 123456  # mysql数据库root用户默认密码root123，实际使用时修改下
ui_url_protocol = http
db_password = root123
max_job_workers = 3 
customize_crt = on
ssl_cert = /data/cert/server.crt
ssl_cert_key = /data/cert/server.key
secretkey_path = /data
admiral_url = NA
clair_db_password = password
# 邮件设置，发送重置密码邮件时使用
email_identity = harbor
email_server = smtp.163.com
email_server_port = 25
email_username = rooroot@163.com
email_password = zhang@123
email_from = admin <rooroot@163.com>
email_ssl = false
harbor_admin_password = zhang@123  # 启动Harbor后，管理员UI登录的密码，默认是Harbor12345
auth_mode = db_auth  # 认证方式，这里支持多种认证方式，如LADP、本次存储、数据库认证。默认是db_auth，mysql数据库认证
# LDAP认证时配置项
ldap_url = ldaps://ldap.mydomain.com
ldap_basedn = ou=people,dc=mydomain,dc=com
ldap_uid = uid 
ldap_scope = 3 
ldap_timeout = 5
self_registration = on  # 是否开启自注册
token_expiration = 30  # Token有效时间，默认30分钟
project_creation_restriction = everyone  # 用户创建项目权限控制，默认是everyone（所有人），也可以设置为adminonly（只能管理员）
verify_remote_cert = on
"""
# 官方构建harbor和启动方式，推荐此方法，会下载官方的docker镜像，在1.1.0版本之后，Harbor已经与Notary进行了集成，但默认情况下安装不包括公证(https)服务。
./install.sh
docker-compose start
"""
Starting log         ... done
Starting redis       ... done
Starting adminserver ... done
Starting registry    ... done
Starting ui          ... done
Starting mysql       ... done
Starting jobservice  ... done
Starting proxy       ... done
"""
# 查看本地镜像
docker images
# 查看端口
ss -tnlp
```

### 配置文件说明

配置参数位于文件harbor.cfg中。

在ports.cfg中有两类参数，必需参数和可选参数。

必需参数：需要在配置文件中设置这些参数。如果用户更新它们harbor.cfg并运行install.sh脚本以重新安装Harbor，它们将生效。

可选参数：这些参数是可选的。如果他们 配置到harbor.cfg，他们只能在首次启动Harbor 生效。这些参数的后续更新harbor.cfg将被忽略。海港启动后，用户可以将其留空，并在Web UI上进行更新。注意：如果您选择通过用户界面设置这些参数，请务必在Harbour启动后立即进行。特别地，您必须在注册或创建任何新的用户之前设置所需的auth_mode。当系统中有用户（默认管理员用户除外）时， 无法更改auth_mode。

参数如下所述 ：请注意，至少需要更改hostname属性。

### 必需参数

hostname：目标主机的主机名，用于访问UI和注册表服务。它应该是目标机器的IP地址或完全限定域名（FQDN），例如192.168.1.10或reg.yourdomain.com。不要使用localhost或127.0.0.1为主机名 – 注册表服务需要外部客户端访问！

ui_url_protocol：（http或https。默认为http）用于访问UI和令牌/通知服务的协议。如果启用公证，则此参数必须为https。默认情况下，这是http。要设置https协议，请参阅使用HTTPS访问harbor：https://github.com/goharbor/harbor/blob/master/docs/configure_https.md

db_password：用于db_auth的MySQL数据库的根密码。更改此密码以供任何生产用途！

max_job_workers：（默认值为3）作业服务中的最大复制工作数。对于每个映像复制作业，工作程序将存储库的所有标签同步到远程目标。增加此数字允许系统中更多的并发复制作业。但是，由于每个工作人员都会消耗一定数量的网络/ CPU / IO资源，请根据主机硬件资源选择该属性的值。

customize_crt：（打开或关闭，默认为打开）当此属性打开时，准备脚本将为注册表令牌的生成/验证创建私钥和根证书。当密钥和根证书由外部源提供时，将此属性设置为off。有关详细信息，请参阅自定义密钥和harbor令牌服务证书：https://github.com/goharbor/harbor/blob/master/docs/customize_token_service.md

ssl_cert：SSL证书的路径，仅当协议设置为https时才应用

ssl_cert_key：SSL密钥的路径，仅当协议设置为https时才应用

secretkey_path：用于在复制策略中加密或解密远程注册表的密码的密钥路径。

### 可选参数

电子邮件设置：Harbor需要这些参数才能向用户发送“密码重设”电子邮件，只有在需要该功能时才需要这些参数。另外，请注意，在默认情况下SSL连接时没有启用-如果你的SMTP服务器需要SSL，但不支持STARTTLS，那么你应该通过设置启用SSL email_ssl = TRUE。

email_server = smtp.mydomain.com

email_server_port = 25

email_username = sample_admin@mydomain.com

email_password = abc

email_from = admin sample_admin@mydomain.com

email_ssl = false

harbor_admin_password：管理员的初始密码。该密码仅在Harbor 第一次启动时生效。之后，此设置将被忽略，并且应在UI中设置管理员的密码。请注意，默认用户名/密码为admin / Harbor12345。

auth_mode：使用的身份验证类型。默认情况下，它是db_auth，即凭据存储在数据库中。对于LDAP身份验证，请将其设置为ldap_auth。重要提示：从现有的Harbor 实例升级时，必须确保auth_modeharbor.cfg在启动新版本的Harbor之前是一样的。否则，升级后用户可能无法登录。

ldap_url：LDAP端点URL（例如ldaps://ldap.mydomain.com）。 仅当auth_mode设置为ldap_auth时才使用。

ldap_searchdn：具有搜索LDAP / AD服务器权限的用户的DN（例如uid=admin,ou=people,dc=mydomain,dc=com）。

ldap_search_pwd：由ldap_searchdn指定的用户的密码。

LDAP_BASEDN：基本DN查找用户，如ou=people,dc=mydomain,dc=com。 仅当auth_mode设置为ldap_auth时才使用。

LDAP_FILTER：用于查找用户，例如，搜索过滤器(objectClass=person)。

ldap_uid：用于在LDAP搜索期间匹配用户的属性，它可以是uid，cn，电子邮件或其他属性。

ldap_scope：搜索用户的范围，1-LDAP_SCOPE_BASE，2-LDAP_SCOPE_ONELEVEL，3-LDAP_SCOPE_SUBTREE。默认值为3。

self_registration：（开或关，默认为开）启用/禁用用户注册自己的能力。禁用时，只能由管理员用户创建新用户，只有管理员用户才能在海港创建新用户。 注意：当auth_mode设置为ldap_auth时，自注册功能始终被禁用，并且该标志被忽略。

token_expiration：令牌服务创建的令牌的到期时间（以分钟为单位），默认值为30分钟。

project_creation_restriction：用于控制用户有权创建项目的标志。默认情况下，每个人都可以创建一个项目，设置为“adminonly”，以便只有admin才能创建项目。

verify_remote_cert：（上或关闭，默认为上）该标志，判断是否验证SSL / TLS证书时码头与远程注册表实例通信。将此属性设置为off可绕过SSL / TLS验证，SSL / TLS验证通常在远程实例具有自签名或不受信任的证书时使用。

## 3，首次部署harbor更新

 

```
pwd
'''
/usr/local/harbor  #在harbor当前目录执行
'''

# 更新配置，执行完毕后会在当前目录生成一个docker-compose.yml文件，用于配置数据目录等配置信息
./prepare

# 后期如果要更改配置，步骤操作如下：

# 停止服务
docker-compose stop

# 编辑harbor.cfg进行相关配置
vim harbor.cfg

# 更新配置,实际会清理之前的配置并重新生成新的配置
./prepare

# 启动harbor服务
docker-compose start
```

 

```
pwd
'''
/usr/local/harbor  #在harbor当前目录执行
'''
# 更新配置，执行完毕后会在当前目录生成一个docker-compose.yml文件，用于配置数据目录等配置信息
./prepare
# 后期如果要更改配置，步骤操作如下：
# 停止服务
docker-compose stop
# 编辑harbor.cfg进行相关配置
vim harbor.cfg
# 更新配置,实际会清理之前的配置并重新生成新的配置
./prepare
# 启动harbor服务
docker-compose start
```

## 4，docker-compose

使用docker-compose来管理Harbor的生命周期。一些有用的命令如下所列（必须与docker -compose.yml在同一目录中运行）。

 

```
# 停止Harbor
docker-compose stop

# 停止后重新启动
docker-compose start

# 要更改Harbor的配置，请首先停止现有的Harbor实例并进行更新harbor.cfg。然后运行prepare脚本来填充配置。最后重新创建并启动Harbor的实例：
docker-compose down -v
vim harbor.cfg
./prepare
docker-compose up -d

# 删除Harbor 的容器，同时保留图像数据和Harbor的数据库文件在文件系统上：
docker-compose down -v

# 删除Harbor 的数据库和图像数据（为了干净的重新安装）：
rm -rf /data/database
rm -rf /data/registry
```

 

```
# 停止Harbor
docker-compose stop
# 停止后重新启动
docker-compose start
# 要更改Harbor的配置，请首先停止现有的Harbor实例并进行更新harbor.cfg。然后运行prepare脚本来填充配置。最后重新创建并启动Harbor的实例：
docker-compose down -v
vim harbor.cfg
./prepare
docker-compose up -d
# 删除Harbor 的容器，同时保留图像数据和Harbor的数据库文件在文件系统上：
docker-compose down -v
# 删除Harbor 的数据库和图像数据（为了干净的重新安装）：
rm -rf /data/database
rm -rf /data/registry
```

当Harbor 与Notary安装时，docker docker-compose.notary.yml-compose命令需要一个额外的模板文件。用于管理Harbor 生命周期的码头组合命令是：

docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml [ up|down|ps|stop|start ]

例如，如果要在配有Notary的情况下更改配置harbor.cfg并重新部署Harbor，则应使用以下命令：

 

```
docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml down -v

vim harbor.cfg

docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml up -d
```

 

```
docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml down -v
vim harbor.cfg
docker-compose -f ./docker-compose.yml -f ./docker-compose.notary.yml up -d
```

有关docker-compose的更多信息，请查看Docker Compose命令行参考:

https://docs.docker.com/compose/reference/

## 5，web访问harbor管理页面

默认管理员用户名/密码为admin / Harbor12345。

![img](Harbor%E5%9F%BA%E7%A1%80.assets/f59bd5eb-dda9-4a32-a5e9-a0776fa3d802.jpg)

## 6，配置docker使用harbor仓库上传下载镜像

编辑docker配置文件,让其启动时自动执行一些参数

 

```
vim /etc/sysconfig/docker
'''
# 其中192.168.10.205是我们部署Harbor的地址，即hostname配置项值。配置完后需要重启docker服务。
4 OPTIONS='--selinux-enabled --log-driver=journald --insecure-registry 172.16.2.103'
'''

# 如果新版本在上述位置没有文件，则加在docker的启动脚本中
vim /usr/lib/systemd/system/docker.service
'''
ExecStart=/usr/bin/dockerd --selinux-enabled --log-driver=journald --insecure-registry 172.16.2.103 --insecure-registry 172.16.2.104
'''

systemctl  stop  docker
systemctl daemon-reload
systemctl  start  docker
ps -ef | grep docker

# 验证docker是否能登录harbor
docker login 172.16.2.103
docker login 172.16.2.104
Username: admin
Password: 123456
Login Succeeded
```

 

```
vim /etc/sysconfig/docker
'''
# 其中192.168.10.205是我们部署Harbor的地址，即hostname配置项值。配置完后需要重启docker服务。
4 OPTIONS='--selinux-enabled --log-driver=journald --insecure-registry 172.16.2.103'
'''
# 如果新版本在上述位置没有文件，则加在docker的启动脚本中
vim /usr/lib/systemd/system/docker.service
'''
ExecStart=/usr/bin/dockerd --selinux-enabled --log-driver=journald --insecure-registry 172.16.2.103 --insecure-registry 172.16.2.104
'''
systemctl  stop  docker
systemctl daemon-reload
systemctl  start  docker
ps -ef | grep docker
# 验证docker是否能登录harbor
docker login 172.16.2.103
docker login 172.16.2.104
Username: admin
Password: 123456
Login Succeeded
```

将之前单机仓库构构建的Nginx镜像上传到harbor服务器用于测试

在harbor管理页面创建一个新的项目，例如myproject。然后使用docker命令登录和推送图像（默认情况下，服务器侦听端口80）

格式：

docker login reg.yourdomain.com

docker push reg.yourdomain.com/myproject/myrepo:mytag

 

```
# 准备要上传到harbor的镜像，设置好tag
docker images
'''
centos-base           latest              781e93eca458        2 days ago          433MB
'''

# 设置tag
docker tag centos-base 172.16.2.103/webimage/centos-base:v1

# push上传到harbor服务器
docker push 172.16.2.103/webimage/centos-base:v1
```

 

```
# 准备要上传到harbor的镜像，设置好tag
docker images
'''
centos-base           latest              781e93eca458        2 days ago          433MB
'''
# 设置tag
docker tag centos-base 172.16.2.103/webimage/centos-base:v1
# push上传到harbor服务器
docker push 172.16.2.103/webimage/centos-base:v1
```

在harbor页面验证

![img](Harbor%E5%9F%BA%E7%A1%80.assets/0c76a403-5ad2-48ef-9436-c81d696c7e43.png)

![img](Harbor%E5%9F%BA%E7%A1%80.assets/afbd2d8a-453f-45f9-a72e-acfb13cdf97a.png)

从harbor下载镜像并启动容器

![img](Harbor%E5%9F%BA%E7%A1%80.assets/c46ffb4a-d395-4455-b5e8-ff887b2ef9c4.png)

 

```
docker pull 172.16.2.103/webimage/centos-base:v1
docker images
docker run -it -d -p80:80 -p443:443 172.16.2.103/webimage/centos-base:v1 bash
```

 

```
docker pull 172.16.2.103/webimage/centos-base:v1
docker images
docker run -it -d -p80:80 -p443:443 172.16.2.103/webimage/centos-base:v1 bash
```