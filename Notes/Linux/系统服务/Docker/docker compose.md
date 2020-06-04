[TOC]

# 简介

docker-compose是一个用来把docker自动化的东西。

当在宿主机启动较多的容器时候，如果都是手动操作会觉得比较麻烦而且容器出错，这个时候推荐使用docker 单机编排工具docker compose，Docker Compose 是docker容器的一种编排服务，docker compose是一个管理多个容器的工具，比如可以解决容器之间的依赖关系，就像启动一个web就必须得先把数据库服务先启动一样，docker compose 完全可以替代docker run启动容器。

GITHUB 地址：

https://github.com/docker/compose

# 基础环境准备

安装python环境及pip命令： 

 

```
yum install https://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm -y
yum install python-pip -y
pip install --upgrade pip
```

 

```
yum install https://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm -y
yum install python-pip -y
pip install --upgrade pip
```

安装docker compose：

 

```
pip install docker-compose
docker-compose version
docker-compose --help
```

 

```
pip install docker-compose
docker-compose version
docker-compose --help
```

工程、服务、容器

Docker Compose 将所管理的容器分为三层，分别是工程（project）、服务（service）、容器（container）

Docker Compose 运行目录下的所有.yml文件（docker-compose.yml）组成一个工程,一个工程包含多个服务，每个服务中定义了容器运行的镜像、参数、依赖，一个服务可包括多个容器实例

# **示例：****启动容器**

注意：

目录可以在任意目录，推荐放在有意义的位置。

在此目录下创建一个yml格式的配置文件，用于docker-compose读取执行，配置文件格式注意前后的缩进。

 

```
mkdir docker-compose
cd docker-compose/
cat docker-compose.yml
'''
web1:  # 项目名，自定义，全文件内唯一
  image: 192.168.10.206/nginx/nginx_1.10.3  # 启容器时用到的镜像
  container_name: nginx-web1  # 自定义容器名称，不指定自动给docker-compose_web1_1
  expose:  # 容器开放的端口，无需引号
    - 80
    - 443
  ports:  # 宿主机和容器的端口映射，需要引号
    - "81:80"
    - "82:443"


web2:  # 可继续写，一次性启动多个容器
  image: 192.168.10.206/nginx/nginx_1.10.3
  container_name: nginx-web2
  expose:
    - 80
    - 443
  ports:
    - "83:80"
    - "84:443"
  volumes:  # 挂载目录，宿主机目录:容器目录
    - /data/nginx:/usr/local/nginx/html
'''

docker-compose up # 前台启动
docker-compose up -d  # 后台启动
docker-compose restart  # 重启所有
```

 

```
mkdir docker-compose
cd docker-compose/
cat docker-compose.yml
'''
web1:  # 项目名，自定义，全文件内唯一
  image: 192.168.10.206/nginx/nginx_1.10.3  # 启容器时用到的镜像
  container_name: nginx-web1  # 自定义容器名称，不指定自动给docker-compose_web1_1
  expose:  # 容器开放的端口，无需引号
    - 80
    - 443
  ports:  # 宿主机和容器的端口映射，需要引号
    - "81:80"
    - "82:443"
web2:  # 可继续写，一次性启动多个容器
  image: 192.168.10.206/nginx/nginx_1.10.3
  container_name: nginx-web2
  expose:
    - 80
    - 443
  ports:
    - "83:80"
    - "84:443"
  volumes:  # 挂载目录，宿主机目录:容器目录
    - /data/nginx:/usr/local/nginx/html
'''
docker-compose up # 前台启动
docker-compose up -d  # 后台启动
docker-compose restart  # 重启所有
```

浏览器测试访问

# docker-compose常用命令

停止和启动单个容器

docker-compose stop web1

docker-compose start web1

docker-compose restart web1

重启所有容器

docker-compose stop

docker-compose start

docker-compose restart

docker-compose command

Commands:

build：Build or rebuild services[重构服务]

bundle：Generate a Docker bundle from the Compose file

config ：Validate and view the compose file

create：Create services

down：Stop and remove containers, networks, images, and volumes

events：Receive real time events from containers

exec：Execute a command in a running container

help：Get help on a command

kill：Kill containers

logs：View output from containers

pause：Pause services

port：Print the public port for a port binding

ps：List containers

pull：Pull service images

push：Push service images

restart：Restart services

rm：Remove stopped containers

run：Run a one-off command[运行一个一次性的命令]

scale：Set number of containers for a service

start：Start services[开启服务]

stop：Stop services[停止服务]

top：Display the running processes

unpause：Unpause services

up：Create and start containers[创建并启动容器]

version：Show the Docker-Compose version information

# 单主机实现HA+Nginx+Tomcat

系统版本：centos7.5

IP地址：172.16.2.101

拓补图：

![img](docker%20compose.assets/1ff10304-7615-47d0-9d04-60d0f5c46449.png)

## 1，制作haproxy镜像

操作步骤如下

 

```

```

 

## 2，制作Nginx镜像

操作步骤如下

 

```

```

 

## 3，制作Tomcat镜像

操作步骤如下

 

```

```

 

## 4，docker compose文件及环境准备

操作步骤如下

1，编辑docker-compose.yml

 

```
cd /opt/dockerfile/web/
vim docker-compose.yml
'''
nginx-web1:  # 项目名，唯一
  image: nginx-app01  # 使用的镜像
  expose:
    - 80
    - 443
  container_name: compose-nginx1
  volumes:
    - /data/nginx/html/:/usr/local/nginx/html  # 映射本地目录/data/nginx/html/到nginx网页根目录
    - /data/nginx/conf/nginx.conf:/usr/local/nginx/conf/nginx.conf   # 映射本地目录/data/nginx/conf/到nginx配置文件目录
  links:
    - compose-tomcat1  # 此处链接容器名称
    - compose-tomcat2

nginx-web2:  # 项目名，唯一
  image: nginx-app02  # 使用的镜像
  volumes:
    - /data/nginx/conf/nginx.conf:/usr/local/nginx/conf/nginx.conf 
  expose:
    - 80
    - 443
  container_name: compose-nginx2
  links:
    - compose-tomcat1  # 此处链接容器名称
    - compose-tomcat2

tomcat-web1:  # 项目名，唯一
  container_name: compose-tomcat1
  image: tomcat8-app01  # 使用的镜像
  user: www
  command: /usr/local/tomcat8/bin/run_tomcat.sh  # 容器启动后默认执行的命令,要保证镜像内对应的目录有此文件
  volumes:
    - /data/tomcat/webapps/SalesManager:/apps/tomcat/webapps/SalesManager
  expose:
    - 8080
    - 8443

tomcat-web2:  # 项目名，唯一
  container_name: compose-tomcat2
  image: tomcat8-app02  # 使用的镜像
  user: www
  command: /usr/local/tomcat8/bin/run_tomcat.sh  # 容器启动后默认执行的命令
  volumes:
    - /data/tomcat/webapps/SalesManager:/apps/tomcat/webapps/SalesManager
  expose:
    - 8080
    - 8443

haproxy:  # 项目名，唯一
  container_name: compose-haproxy
  image: centos_7.2.1511_haproxy_1.7.9  # 使用的镜像
  command: /usr/bin/run_haproxy.sh  # 容器启动后默认执行的命令
  volumes:
    - /data/haproxy/haproxy.cfg:/etc/haproxy/haproxy.cfg
  ports:
    - "9999:9999"
    - "80:80"
  links:
    - compose-nginx1
    - compose-nginx2
'''
```

 

```
cd /opt/dockerfile/web/
vim docker-compose.yml
'''
nginx-web1:  # 项目名，唯一
  image: nginx-app01  # 使用的镜像
  expose:
    - 80
    - 443
  container_name: compose-nginx1
  volumes:
    - /data/nginx/html/:/usr/local/nginx/html  # 映射本地目录/data/nginx/html/到nginx网页根目录
    - /data/nginx/conf/nginx.conf:/usr/local/nginx/conf/nginx.conf   # 映射本地目录/data/nginx/conf/到nginx配置文件目录
  links:
    - compose-tomcat1  # 此处链接容器名称
    - compose-tomcat2
nginx-web2:  # 项目名，唯一
  image: nginx-app02  # 使用的镜像
  volumes:
    - /data/nginx/conf/nginx.conf:/usr/local/nginx/conf/nginx.conf 
  expose:
    - 80
    - 443
  container_name: compose-nginx2
  links:
    - compose-tomcat1  # 此处链接容器名称
    - compose-tomcat2
tomcat-web1:  # 项目名，唯一
  container_name: compose-tomcat1
  image: tomcat8-app01  # 使用的镜像
  user: www
  command: /usr/local/tomcat8/bin/run_tomcat.sh  # 容器启动后默认执行的命令,要保证镜像内对应的目录有此文件
  volumes:
    - /data/tomcat/webapps/SalesManager:/apps/tomcat/webapps/SalesManager
  expose:
    - 8080
    - 8443
tomcat-web2:  # 项目名，唯一
  container_name: compose-tomcat2
  image: tomcat8-app02  # 使用的镜像
  user: www
  command: /usr/local/tomcat8/bin/run_tomcat.sh  # 容器启动后默认执行的命令
  volumes:
    - /data/tomcat/webapps/SalesManager:/apps/tomcat/webapps/SalesManager
  expose:
    - 8080
    - 8443
haproxy:  # 项目名，唯一
  container_name: compose-haproxy
  image: centos_7.2.1511_haproxy_1.7.9  # 使用的镜像
  command: /usr/bin/run_haproxy.sh  # 容器启动后默认执行的命令
  volumes:
    - /data/haproxy/haproxy.cfg:/etc/haproxy/haproxy.cfg
  ports:
    - "9999:9999"
    - "80:80"
  links:
    - compose-nginx1
    - compose-nginx2
'''
```

2，准备nginx静态文件

 

```
cd /data/nginx/conf/nginx.conf
'''

'''
```

 

```
cd /data/nginx/conf/nginx.conf
'''
'''
```