[TOC]

# 单机Docker Registry

Docker Registry作为Docker的核心组件之一负责镜像内容的存储与分发，客户端的docker pull以及push命令都将直接与registry进行交互.

最初版本的registry 由Python实现,由于设计初期在安全性，性能以及API的设计上有着诸多的缺陷，该版本在0.9之后停止了开发，由新的项目distribution（新的docker register被称为Distribution）来重新设计并开发下一代registry.

新的项目由go语言开发，所有的API，底层存储方式，系统架构都进行了全面的重新设计已解决上一代registry中存在的问题.

2016年4月份rgistry 2.0正式发布，docker 1.6版本开始支持registry 2.0，而八月份随着docker 1.8 发布，docker hub正式启用2.1版本registry全面替代之前版本 registry，新版registry对镜像存储格式进行了重新设计并和旧版不兼容，docker 1.5和之前的版本无法读取2.0的镜像，另外，Registry 2.4版本之后支持了回收站机制，也就是可以删除镜像了，在2.4版本之前是无法支持删除镜像的，所以如果你要使用最好是大于Registry 2.4版本的。

## 搭建单机仓库

本部分将介绍通过官方提供的docker registry镜像来简单搭建一套本地私有仓库环境。

```
# 下载docker registry镜像
docker pull  registry

# 创建授权使用目录
mkdir -pv /docker/auth

# 创建授权用户和密码，生成文件
cd /docker
docker run --entrypoint htpasswd registry -Bbn xiaofei 123456 > auth/htpasswd

cat auth/htpasswd

# 启动
docker run -d -p 5000:5000 --restart=always --name registry1 -v /docker/auth:/auth -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
'''
registryce659e85018bea3342045f839c43b66de1237ce5413c0b6b72c0887bece5325a
'''

# 验证
docker ps
ss -tnlp
docker login 172.16.2.101:5000
Username:jack
Password:


# 上传镜像,在新的客户机上
docker images
docker tag nginx-base:v1 172.16.2.101:5000/xiaofei/nginx-base:v1
docker push 172.16.2.101:5000/xiaofei/nginx-base:v1


# 下载镜像
docker images
docker login 172.16.2.101:5000
docker pull 172.16.2.101:5000/xiaofei/nginx-base:v1
```

 

```
# 下载docker registry镜像
docker pull  registry
# 创建授权使用目录
mkdir -pv /docker/auth
# 创建授权用户和密码，生成文件
cd /docker
docker run --entrypoint htpasswd registry -Bbn xiaofei 123456 > auth/htpasswd
cat auth/htpasswd
# 启动
docker run -d -p 5000:5000 --restart=always --name registry1 -v /docker/auth:/auth -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
'''
registryce659e85018bea3342045f839c43b66de1237ce5413c0b6b72c0887bece5325a
'''
# 验证
docker ps
ss -tnlp
docker login 172.16.2.101:5000
Username:jack
Password:
# 上传镜像,在新的客户机上
docker images
docker tag nginx-base:v1 172.16.2.101:5000/xiaofei/nginx-base:v1
docker push 172.16.2.101:5000/xiaofei/nginx-base:v1
# 下载镜像
docker images
docker login 172.16.2.101:5000
docker pull 172.16.2.101:5000/xiaofei/nginx-base:v1
```

如果报错：

![img](docker%E4%BB%93%E5%BA%93.assets/85dff37e-e492-44b9-9743-7c17bc7b6ca7.jpg)

解决办法：

编辑各docker 服务器/etc/sysconfig/docker 配置文件如下：

 

```
# server2
vim /etc/sysconfig/docker
'''
OPTIONS='--selinux-enabled --log-driver=journald'
ADD_REGISTRY='--add-registry 192.168.10.205:5000'
INSECURE_REGISTRY='--insecure-registry 192.168.10.205:5000'
'''

systemctl restart docker


# server2
vim /etc/sysconfig/docker
'''
OPTIONS='--selinux-enabled --log-driver=journald'
if [ -z "${DOCKER_CERT_PATH}" ]; then
    DOCKER_CERT_PATH=/etc/docker
fi
ADD_REGISTRY='--add-registry 192.168.10.205:5000'
INSECURE_REGISTRY='--insecure-registry 192.168.10.205:5000'
'''

systemctl restart docker
```