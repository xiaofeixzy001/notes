[TOC]

# 单向复制 

Harbor支持基于策略的Docker镜像复制功能，这类似于MySQL的主从同步，其可以实现不同的数据中心、不同的运行环境之间同步镜像，并提供友好的管理界面，大大简化了实际运维中的镜像管理工作，已经有用很多互联网公司使用harbor搭建内网docker仓库的案例，并且还有实现了双向复制的案列，本文将实现单向复制的部署

新部署一台harbor服务器172.16.2.104，同步主harbor服务器172.16.2.103

 

```
cd /usr/local/src/
tar xf harbor-offline-installer-v1.2.2.tgz
ln -sv /usr/local/src/harbor /usr/local/
cd /usr/local/harbor/
grep "^[a-Z]" harbor.cfg
'''
hostname = 192.168.10.206 
ui_url_protocol = http
db_password = root123
max_job_workers = 3 
customize_crt = on
ssl_cert = /data/cert/server.crt
ssl_cert_key = /data/cert/server.key
secretkey_path = /data
admiral_url = NA
clair_db_password = password
email_identity = harbor-1.2.2
email_server = smtp.163.com
email_server_port = 25
email_username = rooroot@163.com
email_password = zhang@123
email_from = admin <rooroot@163.com>
email_ssl = false
harbor_admin_password = zhang@123
auth_mode = db_auth
ldap_url = ldaps://ldap.mydomain.com
ldap_basedn = ou=people,dc=mydomain,dc=com
ldap_uid = uid 
ldap_scope = 3 
ldap_timeout = 5
self_registration = on
token_expiration = 30
project_creation_restriction = everyone
verify_remote_cert = on
'''

yum install python-pip -y
pip install --upgrade pip
pip install  docker-compose
./install.sh
```

 

```
cd /usr/local/src/
tar xf harbor-offline-installer-v1.2.2.tgz
ln -sv /usr/local/src/harbor /usr/local/
cd /usr/local/harbor/
grep "^[a-Z]" harbor.cfg
'''
hostname = 192.168.10.206 
ui_url_protocol = http
db_password = root123
max_job_workers = 3 
customize_crt = on
ssl_cert = /data/cert/server.crt
ssl_cert_key = /data/cert/server.key
secretkey_path = /data
admiral_url = NA
clair_db_password = password
email_identity = harbor-1.2.2
email_server = smtp.163.com
email_server_port = 25
email_username = rooroot@163.com
email_password = zhang@123
email_from = admin <rooroot@163.com>
email_ssl = false
harbor_admin_password = zhang@123
auth_mode = db_auth
ldap_url = ldaps://ldap.mydomain.com
ldap_basedn = ou=people,dc=mydomain,dc=com
ldap_uid = uid 
ldap_scope = 3 
ldap_timeout = 5
self_registration = on
token_expiration = 30
project_creation_restriction = everyone
verify_remote_cert = on
'''
yum install python-pip -y
pip install --upgrade pip
pip install  docker-compose
./install.sh
```

验证从harborWEB端登录，新建项目nginx(公有)，添加复制规则

![img](Harbor%E5%A4%8D%E5%88%B6.assets/9067c77a-9fe2-43d1-9014-c175107e32fc.jpg)

![img](Harbor%E5%A4%8D%E5%88%B6.assets/576fd8b1-8f8e-4426-baf4-0ac55475e675.png)

目标这里如果没有，可以先创建，注意测试连接。

![img](Harbor%E5%A4%8D%E5%88%B6.assets/bb05a59d-91f5-499d-b818-f2a273c3b908.png)

![img](Harbor%E5%A4%8D%E5%88%B6.assets/fef63401-6ed5-4a37-8a13-0e7e0d12a588.png)

注意用户名和密码，写对方的IP+用户名密码，然后点测试连接，确认可以测试连接通过。

![img](Harbor%E5%A4%8D%E5%88%B6.assets/a384d802-26fa-4037-80e0-75de32561979.png)

在新的harbor服务器上查看同步状态。

# 双向复制 

harbor-1：172.16.2.103，项目名为webimage

harbor-2：172.16.2.104，项目名为webimage

在docker客户端上传镜像nginx-base:v1到harbor-1中的webimage，上传镜像nginx-base:v2到harbor-2中的webimage

 

```
docker tag nginx-base 172.16.2.103/webimage/nginx-base:v1
docker tag nginx-base 172.16.2.104/webimage/nginx-base:v2
docker images

docker login 172.16.2.103
'''
Username: admin
Password: 
Login Succeeded
'''

docker push 172.16.2.103/nginx/nginx-base:v1

docker login 172.16.2.104
'''
Username: admin
Password: 
Login Succeeded
'''
docker push 172.16.2.104/nginx/nginx-base:v2
```

 

```
docker tag nginx-base 172.16.2.103/webimage/nginx-base:v1
docker tag nginx-base 172.16.2.104/webimage/nginx-base:v2
docker images
docker login 172.16.2.103
'''
Username: admin
Password: 
Login Succeeded
'''
docker push 172.16.2.103/nginx/nginx-base:v1
docker login 172.16.2.104
'''
Username: admin
Password: 
Login Succeeded
'''
docker push 172.16.2.104/nginx/nginx-base:v2
```

在两个harbor服务器上查看同步状态

# 本地镜像上传至官方docker仓库

将自制的镜像上传至docker仓库；https://hub.docker.com/

1，准备账户

登录到docker hub创建官网创建账户，登录后点击settings完善账户信息：

![img](Harbor%E5%A4%8D%E5%88%B6.assets/03b00c23-ef48-4d84-83aa-3d226fe19878.jpg)

填写账户基本信息

![img](Harbor%E5%A4%8D%E5%88%B6.assets/948a0812-d2d5-446b-ba5d-1cec91cfae69.jpg)

在虚拟机使用自己的账号登录：

![img](Harbor%E5%A4%8D%E5%88%B6.assets/8cb72f1b-cc36-41ba-91ea-5b3f347946fd.jpg)

查看认证信息：登录成功之后会在当前目录生成一个隐藏文件用于保存登录认证信息

![img](Harbor%E5%A4%8D%E5%88%B6.assets/be584876-8d61-4427-bb9b-a3e20fa9d6b0.jpg)

给镜像做tag并开始上传：

 

```
docker images
docker tag 678e2f074b0d docker.io/zhangshijie/centos-nginx
docker login  # 不指定登录地址，默认登录到docker官方网站
docker push docker.io/zhangshijie/centos-nginx
```

 

```
docker images
docker tag 678e2f074b0d docker.io/zhangshijie/centos-nginx
docker login  # 不指定登录地址，默认登录到docker官方网站
docker push docker.io/zhangshijie/centos-nginx
```

上传完成以后，到docker官网验证

![img](Harbor%E5%A4%8D%E5%88%B6.assets/1d11e815-be52-4557-8c60-2e527afd5bb6.png)

 

```
docker login https://hub.docker.com
docker pull zhangshijie/alpine-test
docker images
docker run -it docker.io/zhangshijie/alpine-test bash
```

 

```
docker login https://hub.docker.com
docker pull zhangshijie/alpine-test
docker images
docker run -it docker.io/zhangshijie/alpine-test bash
```