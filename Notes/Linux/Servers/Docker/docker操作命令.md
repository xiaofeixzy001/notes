[TOC]

docker命令是最常使用的命令,其后面可以加不同的参数以实现响应的功能,常用的命令如下:

![image-20200503154801507](docker%E6%93%8D%E4%BD%9C%E5%91%BD%E4%BB%A4.assets/image-20200503154801507.png)



# 搜索镜像

## docker search

Usage:	docker search [OPTIONS] TERM

OPTIONS：

--automated :只列出 automated build类型的镜像；

--no-trunc :显示完整的镜像描述；

-s :列出收藏数不小于指定值的镜像；

 

```
# 镜像名称后面不在带版本号，则默认显示并下载latest版
docker search -s 100 centos:7.2.1511
```

 

```
# 镜像名称后面不在带版本号，则默认显示并下载latest版
docker search -s 100 centos:7.2.1511
```

# 下载镜像

## docker pull

下载镜像，从docker 仓库将镜像下载到本地

Usage:	docker pull [OPTIONS] NAME[:TAG|@DIGEST]

OPTIONS：

-a：拉取所有tagged镜像

--disable-content-trust：忽略镜像的校验，默认是开启的

修改镜像下载位置

 

```
# 方法1
find / -name docker.service
vim /usr/lib/systemd/system/docker.service
"""
ExecStart=/usr/bin/dockerd --graph=/data/docker  # docker新的存储位置
"""

# 方法2
vim /etc/docker/daemon.json
"""
{
    "registry-mirrors": ["https://0b8s8193.mirror.aliyuncs.com"],
    "graph": "/data/docker"
}
"""

systemctl daemon-reload
systemctl restart docker
docker info
"""
Docker Root Dir: /data/docker
"""
```

 

```
# 方法1
find / -name docker.service
vim /usr/lib/systemd/system/docker.service
"""
ExecStart=/usr/bin/dockerd --graph=/data/docker  # docker新的存储位置
"""
# 方法2
vim /etc/docker/daemon.json
"""
{
    "registry-mirrors": ["https://0b8s8193.mirror.aliyuncs.com"],
    "graph": "/data/docker"
}
"""
systemctl daemon-reload
systemctl restart docker
docker info
"""
Docker Root Dir: /data/docker
"""
```

# 查看已下载镜像

## dokcer images 

列出本地镜像

Usage:	docker images [OPTIONS] [REPOSITORY[:TAG]]

OPTIONS：

-a :列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层）；

--digests :显示镜像的摘要信息；

-f :显示满足条件的镜像；

--format :指定返回值的模板文件；

--no-trunc :显示完整的镜像信息；

-q :只显示镜像ID。

 

```
docker images
"""
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c82521676580        6 days ago          109MB
"""

'''
REPOSITORY: 镜像所属的仓库名称
TAG: 镜像版本号(标识符),默认为latest
IMAGE ID: 镜像唯一ID标示
CREATED: 镜像创建时间
VIRTUAL SIZE: 镜像的大小
'''
```

 

```
docker images
"""
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c82521676580        6 days ago          109MB
"""
'''
REPOSITORY: 镜像所属的仓库名称
TAG: 镜像版本号(标识符),默认为latest
IMAGE ID: 镜像唯一ID标示
CREATED: 镜像创建时间
VIRTUAL SIZE: 镜像的大小
'''
```

# 镜像导出导入

## docker save

可以将镜像从本地导出为一个压缩文件，可以复制到其他服务器进行导入使用.

Usage1：docker save -o PATH IMG_NAME[IMG_ID]

Usage2：docker save IMG_NAME[IMG_ID] > PATH

OPTIONS：

-o FILE_NAME：导出到指定文件FILE_NAME。

 

```
docker images
"""
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c82521676580        12 days ago         109MB
<none>              <none>              49f7960eb7e4        2 months ago        200MB
"""

# 方法1
docker save -o /opt/nginx.tar.gz nginx
ll /opt

# 方法2
docker save 49f7960eb7e4 > /opt/centos-7.5.1804.tar.gz
ll /opt
```

 

```
docker images
"""
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              c82521676580        12 days ago         109MB
<none>              <none>              49f7960eb7e4        2 months ago        200MB
"""
# 方法1
docker save -o /opt/nginx.tar.gz nginx
ll /opt
# 方法2
docker save 49f7960eb7e4 > /opt/centos-7.5.1804.tar.gz
ll /opt
```

## docker load

镜像导入，从tar格式的镜像存档导入到容器中

Usage:	docker load [OPTIONS]

OPTIONS：

-i：从tar存档文件中读取

-q：Suppress the load output

如：

docker load < 镜像压缩包

docker load -i 镜像压缩包

 

```
docker save ubuntu:load>/root/ubuntu.tar
docker load < ubuntu.tar
```

 

```
docker save ubuntu:load>/root/ubuntu.tar
docker load < ubuntu.tar
```

补充：docker容器导入导出还有一种方式：使用export和import，不能混用

 

```
docker export 98ca36> ubuntu.tar
cat ubuntu.tar | sudo docker import - ubuntu:import
```

 

```
docker export 98ca36> ubuntu.tar
cat ubuntu.tar | sudo docker import - ubuntu:import
```

# 镜像删除 

docker rmi

删除本地镜像

Usage:	docker rmi [OPTIONS] IMAGE [IMAGE...]

OPTIONS：

-f: 强制删除,当镜像正在被使用时可以使用此参数

--no-prune：不移除该镜像的过程镜像，默认移除

注:通过镜像启动容器的时候镜像不能被删除，除非将容器全部关闭

# 根据镜像启动容器

## docker run

创建一个新的容器并运行一个命令

获取帮助:docker run --help

官方文档:https://docs.docker.com/engine/reference/commandline/run/#options

Usage：docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

OPTIONS：

-d: 后台运行容器，并返回容器ID；

-i: 以交互模式运行容器，通常与 -t 同时使用；

-p: 端口映射，格式为：主机(宿主)端口:容器端口

-t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用；

--name="nginx-lb": 为容器指定一个名称；

--dns 8.8.8.8: 指定容器使用的DNS服务器，默认和宿主一致；

--dns-search example.com: 指定容器DNS搜索域名，默认和宿主一致；

-h "mars": 指定容器的hostname；

-e username="ritchie": 设置环境变量；

--env-file=[]: 从指定文件读入环境变量；

--cpuset="0-2" or --cpuset="0,1,2": 绑定容器到指定CPU运行；

-m :设置容器使用内存最大值；

--net="bridge": 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；

--link=[]: 添加链接到另一个容器；

--expose=[]: 开放一个端口或一组端口； 

注意：

1，退出容器不注销：ctrl+p+q

2，镜像名称需在所有的选项之后

 

```
# 启动的容器在执行完shel命令就退出了
docker run nginx /bin/echo "hello,world"
"""
hello,world
"""

# 根据镜像以交互模式启动一个容器，在容器内执行/bin/bash命令，注意命令行前缀变化
[root@docker-server1 ~]# docker run -it nginx /bin/bash
root@5c6666c40b36:/# 

# 后台启动一个容器,并将容器命名为mynginx
docker run --name nginx-test1 -d nginx:latest

# 前台启动并随机映射本地端口到容器的80端口,前台启动后,无法在当前窗口进行其他操作,除非退出.-P后面不指定端口号，则随机选择宿主机端口号，随机端口默认从32768开始。
docker run -P -d nginx
ss -tnlp
docker port CONTAINER_ID # 查看某个容器的端口映射信息

# 指定端口：本地端口8001映射到容器的80端口，本地的/data目录映射到容器的/data目录(之前是宿主机文件夹,之后是容器需共享的文件夹),nginx-test-port1为自定义的名字
docker run -d -p 8001:80 -v /data:/data --name nginx-test-port1 nginx

# 一次性映射多个端口协议
docker run -d -p 80:80/tcp -p 443:443/tcp -p 53:53/udp --name nginx-test-port5 nginx

# 运行一个在后台执行的容器,同时,还能用控制台管理,且当停止时自动删除(如果不加--rm,尽管容器停止,但历史记录中还会存在)
docker run -i -t -d --rm nginx

# 后台运行一个容器,指定宿主机的9900端口映射为容器的80端口(不加-p,则选择宿主机的一个随机端口来映射)
docker run -i -t --rm -p 9900:80 nginx  # 此时会卡住
ss -tnlp
curl localhost:9900
```

 

```
# 启动的容器在执行完shel命令就退出了
docker run nginx /bin/echo "hello,world"
"""
hello,world
"""
# 根据镜像以交互模式启动一个容器，在容器内执行/bin/bash命令，注意命令行前缀变化
[root@docker-server1 ~]# docker run -it nginx /bin/bash
root@5c6666c40b36:/# 
# 后台启动一个容器,并将容器命名为mynginx
docker run --name nginx-test1 -d nginx:latest
# 前台启动并随机映射本地端口到容器的80端口,前台启动后,无法在当前窗口进行其他操作,除非退出.-P后面不指定端口号，则随机选择宿主机端口号，随机端口默认从32768开始。
docker run -P -d nginx
ss -tnlp
docker port CONTAINER_ID # 查看某个容器的端口映射信息
# 指定端口：本地端口8001映射到容器的80端口，本地的/data目录映射到容器的/data目录(之前是宿主机文件夹,之后是容器需共享的文件夹),nginx-test-port1为自定义的名字
docker run -d -p 8001:80 -v /data:/data --name nginx-test-port1 nginx
# 一次性映射多个端口协议
docker run -d -p 80:80/tcp -p 443:443/tcp -p 53:53/udp --name nginx-test-port5 nginx
# 运行一个在后台执行的容器,同时,还能用控制台管理,且当停止时自动删除(如果不加--rm,尽管容器停止,但历史记录中还会存在)
docker run -i -t -d --rm nginx
# 后台运行一个容器,指定宿主机的9900端口映射为容器的80端口(不加-p,则选择宿主机的一个随机端口来映射)
docker run -i -t --rm -p 9900:80 nginx  # 此时会卡住
ss -tnlp
curl localhost:9900
```

注:如果启动容器时报错,添加参数--privileged=true即可

docker: Error response from daemon: Container command could not be invoked..

# 查看容器

## docker ps

显示正在运行的容器

Usage:	docker ps [OPTIONS]

OPTIONS：

-a :显示所有的容器，包括未运行的。

-f :根据条件过滤显示的内容。

--format :指定返回值的模板文件。

-l :显示最近创建的容器。

-n :列出最近创建的n个容器。

--no-trunc :不截断输出。

-q :静默模式，只显示容器编号。

-s :显示总的文件大小

 

```
# 列出所有在运行的容器信息
docker ps

# 显示所有状态的容器,状态:created|restarting|running|removing|paused|exited|dead
docker ps -a

# 列出最近创建的5个容器信息(不限状态)
docker ps -n 5

# 列出最后被创建的容器
docker ps -l
docker ps -n 1

# 仅列出容器ID
docker ps -q

# 显示容器文件大小,可以获得2个数值,一个是容器真实增加的大小,一个是整个容器的虚拟大小(容器虚拟大小 = 容器真实增加大小 + 容器镜像大小)
docker ps -s
```

 

```
# 列出所有在运行的容器信息
docker ps
# 显示所有状态的容器,状态:created|restarting|running|removing|paused|exited|dead
docker ps -a
# 列出最近创建的5个容器信息(不限状态)
docker ps -n 5
# 列出最后被创建的容器
docker ps -l
docker ps -n 1
# 仅列出容器ID
docker ps -q
# 显示容器文件大小,可以获得2个数值,一个是容器真实增加的大小,一个是整个容器的虚拟大小(容器虚拟大小 = 容器真实增加大小 + 容器镜像大小)
docker ps -s
```

## docker ps的高级用法:

-f, --filter: 过滤显示,如果容器数量过多,或想排除干扰容器时使用此选项.

-f, --format: 格式化显示,如果想自定义显示容器字段时使用此选项

### --filter

过滤条件万变不离其宗,只要再记住以下3条准:

1,选项后跟的都是键值对key=value(可不带引号）,如果有多个过滤条件,就多次使用filter选项.

 

```
docker ps --filter id=c82521676580 --filter name=mynginx
```

 

```
docker ps --filter id=c82521676580 --filter name=mynginx
```

2,相同条件之间的关系是或,不同条件之间的关系是与.

 

```
# 找出name包含mynginx或centos_server,并且status为running的容器.
docker ps --filter name=mynginx --filter name=centos_server --filter status=running
```

 

```
# 找出name包含mynginx或centos_server,并且status为running的容器.
docker ps --filter name=mynginx --filter name=centos_server --filter status=running
```

3,id和name,支持正则表达式,使用起来非常灵活.

 

```
# 精确匹配name为mynginx的容器.注意,容器实际名称,开头是有一个正斜线/,可用docker inspect查看
docker ps --filter name=^/mynginx$

# 匹配name包含centos_server的容器,和--filter name=centos_server一个效果
docker ps --filter name=.*centos_server.*

# 清理名称包含mynginx,且状态为exited或dead的容器
docker rm $(docker ps -q --filter name=.*mynginx.* --filter status=exited --filter status=dead 2> /dev/null)
```

 

```
# 精确匹配name为mynginx的容器.注意,容器实际名称,开头是有一个正斜线/,可用docker inspect查看
docker ps --filter name=^/mynginx$
# 匹配name包含centos_server的容器,和--filter name=centos_server一个效果
docker ps --filter name=.*centos_server.*
# 清理名称包含mynginx,且状态为exited或dead的容器
docker rm $(docker ps -q --filter name=.*mynginx.* --filter status=exited --filter status=dead 2> /dev/null)
```

### --format

格式化显示,支持的占位符如下:

.ID 容器ID

.Image 镜像ID

.Command Quoted command

.CreatedAt 创建容器的时间点.

.RunningFor 从容器创建到现在过去的时间.

.Ports 暴露的端口.

.Status 容器状态.

.Size 容器占用硬盘大小.

.Names 容器名称.

.Labels 容器所有的标签.

.Label 指定label的值 例如'{{.Label “com.docker.swarm.cpu”}}’

.Mounts 挂载到这个容器的数据卷名称

 

```
docker ps --format "{{.ID}}: {{.Command}}"
"""
94c2ce92b6e9: "nginx -g 'daemon of…"
"""

docker ps --format "table {{.ID}}\t{{.Labels}}"
"""
CONTAINER ID        LABELS
94c2ce92b6e9        maintainer=NGINX Docker Maintainers <docker-maint@nginx.com>
"""
```

 

```
docker ps --format "{{.ID}}: {{.Command}}"
"""
94c2ce92b6e9: "nginx -g 'daemon of…"
"""
docker ps --format "table {{.ID}}\t{{.Labels}}"
"""
CONTAINER ID        LABELS
94c2ce92b6e9        maintainer=NGINX Docker Maintainers <docker-maint@nginx.com>
"""
```

# **删除容器** 

## docker rm

删除容器

Usage:	docker rm [OPTIONS] CONTAINER [CONTAINER...]

OPTIONS说明：

-f :通过SIGKILL信号强制删除一个运行中的容器

-l :移除容器间的网络连接，而非容器本身

-v :-v 删除与容器关联的卷

 

```
# 强制删除容器db01，db02
docker rm -f db01 db02

# 移除容器nginx01对容器01的连接，连接名为db
docker rm -l db

# 删除容器nginx01，并删除容器挂载的数据卷
docker rm -v nginx01
```

 

```
# 强制删除容器db01，db02
docker rm -f db01 db02
# 移除容器nginx01对容器01的连接，连接名为db
docker rm -l db
# 删除容器nginx01，并删除容器挂载的数据卷
docker rm -v nginx01
```

# **查看操作历史** 

## docker history

查看指定镜像的创建历史

Usage:	docker history [OPTIONS] IMAGE

OPTIONS

-H :以可读的格式打印镜像大小和日期，默认为true；

--no-trunc :显示完整的提交记录；

-q :仅列出提交记录ID。

# 查看日志

docker logs nginx-test-port3  # 一次性显示

docker logs -f nginx-test-port3  # 持续显示

# 进入运行中的容器

## 方法1：使用attach命令

命令格式：docker attach container_id

attach 类似于vnc，当多个窗口同时attach到同一个容器时,操作会在各个容器界面显示，所有使用此方式进入容器的操作都是同步显示的,假如其中的一个窗口发生阻塞时，其它的窗口也会阻塞,且exit后容器将被关闭，

且使用exit退出后容器关闭，不推荐使用.

## 方法2：使用exec命令

命令格式：docker exec -ti container_id /bin/bash

执行单次命令与进入容器，不是很推荐此方式，虽然exit退出容器还在运行

参数:

-t:如果只使用-t参数，则可以看到一个console窗口，可以在宿主机上执行容器里的命令并查看到命令结果，但是这种方式登陆到容器内执行的命令是没有结果信息输出的.

-i:如果只用-i时,由于没有分配伪终端,看起来像pipe执行一样.但是执行结果、命令返回值都可以正确获取.

这种方式可以理解为:在运行的容器上执行新进程!即在宿主机上执行容器里的命令并查看到命令结果!这很方便的,但是仅仅使

用-i参数无法直接登陆到容器内!

使用-it时,则和我们平常操作console界面类似,而且不会像attach方式因为退出而导致整个容器退出,这种方式可以替

代ssh或者nsenter方式,在容器内进行操作.

## 方法3：使用nsenter命令

推荐使用此方式，nsenter命令需要通过PID进入到容器内部，使用docker inspect获取到容器的PID

nsenter工具在util-linux包2.23版本后包含,如果系统中util-linux包没有该命令,可以按照下面的方法从源码安装.

\# 安装

 

```
cd /usr/local/src/
curl https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz | tar -zxf-; cd util-linux-2.24;
./configure --without-ncurses
make nsenter && cp nsenter /usr/local/bin
```

 

```
cd /usr/local/src/
curl https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz | tar -zxf-; cd util-linux-2.24;
./configure --without-ncurses
make nsenter && cp nsenter /usr/local/bin
```

在使用nsenter命令之前需要获取到docker容器的进程id,然后再使用nsenter工具进去到docker容器中,具体的使用方法如下:

docker inspect -f {{.State.Pid}} 容器名或者容器id

每一个容器都有'.State.Pid'，所以这个命令除了容器的id需要我们根据'docker ps -a'去查找，其他的全部为固定的格式:

nsenter --target 上面查到的进程id --mount --uts --ipc --net --pid

输入该命令便进入到容器中.

获取帮助信息

nsenter --help

man nsenter

target:

--mount参数是进去到mount namespace中

--uts参数是进入到uts namespace中

--ipc参数是进入到System V IPC namaspace中

--net参数是进入到network namespace中

--pid参数是进入到pid namespace中

--user参数是进入到user namespace中

 

```
# 获取CONTAINER ID
docker ps -a

# 根据容器id获取ip
docker inspect -f "{{.NetworkSettings.IPAddress}}" 7ddbc2885f3f

# 根据容器名获取pid
docker inspect -f "{{.State.Pid}}" nginx-test-port1

# 脚本方式,将nsenter写入到脚本中进行调用
cat docker-in.sh
"""
#!/bin/bash
docker_in(){
    NAME_ID=$1
    PID=$(docker inspect -f "{{.State.Pid}}" ${NAME_ID})
    nsenter -t ${PID} -m -u -i -n -p
}
docker_in $1
"""
chomd +x docker-in.sh
bash docker-in.sh 7ddbc2885f3f
pwd
exit
"""

# 查看容器内部的hosts文件
docker run -i -t --name nginx-test1 docker.io/nginx /bin/bash
root@dfe515b02712:/# cat /etc/hosts
"""
127.0.0.1 localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2 dfe515b02712 # 默认会将实例的ID 添加到自己的hosts文件
"""
```

 

```
# 获取CONTAINER ID
docker ps -a
# 根据容器id获取ip
docker inspect -f "{{.NetworkSettings.IPAddress}}" 7ddbc2885f3f
# 根据容器名获取pid
docker inspect -f "{{.State.Pid}}" nginx-test-port1
# 脚本方式,将nsenter写入到脚本中进行调用
cat docker-in.sh
"""
#!/bin/bash
docker_in(){
    NAME_ID=$1
    PID=$(docker inspect -f "{{.State.Pid}}" ${NAME_ID})
    nsenter -t ${PID} -m -u -i -n -p
}
docker_in $1
"""
chomd +x docker-in.sh
bash docker-in.sh 7ddbc2885f3f
pwd
exit
"""
# 查看容器内部的hosts文件
docker run -i -t --name nginx-test1 docker.io/nginx /bin/bash
root@dfe515b02712:/# cat /etc/hosts
"""
127.0.0.1 localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2 dfe515b02712 # 默认会将实例的ID 添加到自己的hosts文件
"""
```

# 指定容器DNS 

容器的dns默认采用宿主机的dns地址,修改容器dns有一下2种方式:

一是将dns地址配置在宿主机;

二是将参数配置在docker启动脚本里面: -dns=8.8.8.8

 

```
# 方式1
docker run -it --rm --dns 8.8.8.8 centos bash
[root@c177474075e8 /]# cat /etc/resolv.conf
"""
nameserver 8.8.8.8
"""

# 方式2
vim /usr/lib/systemd/system/docker.service
"""
ExecStart=/usr/bin/dockerd -dns=8.8.8.8
```

 

```
# 方式1
docker run -it --rm --dns 8.8.8.8 centos bash
[root@c177474075e8 /]# cat /etc/resolv.conf
"""
nameserver 8.8.8.8
"""
# 方式2
vim /usr/lib/systemd/system/docker.service
"""
ExecStart=/usr/bin/dockerd -dns=8.8.8.8
```

# 总结

企业使用镜像及常用操作,搜索,下载,导出,导入,删除.

docker load -i centos-latest.tar.xz  # 导入本地镜像

docker save -o  /opt/centos.tar  #centos # 导出镜像

docker rmi 镜像ID/镜像名称   # 删除指定ID的镜像，通过镜像启动容器的时候镜像不能被删除，除非将容器全部关闭

docker  rm 容器ID/容器名称  # 删除容器 

docker   rm 容器ID/容器名 -f  # 强制删除正在运行的容器

容器的启动和关闭,删除

docker start 7ddbc2885f3f  # 启动指定容器

docker stop 7ddbc2885f3f  # 关闭指定容器

docker stop $(docker ps -a -q)  # 关闭所有运行中的容器

docker kill $(docker ps -a -q)  # 强制关闭

docker rm -f `docker ps -aq -f status=exited`  # 批量删除已退出容器

docker rm -f $(docker ps -a -q)  # 批量删除所有容器

docker启动脚本:

/usr/lib/systemd/system/docker.service

ExecStart=/usr/bin/dockerd +参数

docker镜像下载位置

/var/lib/docker/image/overlay2/layerdb/sha256/

/var/lib/docker/image/overlay2/imagedb/content/sha256/