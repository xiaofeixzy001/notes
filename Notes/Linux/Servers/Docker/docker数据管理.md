[TOC]

# 数据类型

Docker的镜像是分层设计的，底层是只读的，通过镜像启动的容器添加了一层可读写的文件系统，用户写入的数据都保存在这一层当中，如果要将写入的数据永久生效，需要将其提交为一个镜像然后通过这个镜像在启动实例，然后就会给这个启动的实例添加一层可读写的文件系统，目前Docker的数据类型分为两种，一是数据卷，二是数据容器，数据卷类似于挂载的一块磁盘，数据容器是将数据保存在一个容器上

# 数据卷(data volume)

数据卷实际上就是宿主机上的目录或者是文件，可以被直接mount到容器当中使用。

 

```
# 在宿主机上操作(本地)
mkdir testapp
echo "Test App Page From localhost" > /test/app/index.html
cat /testapp/index.html

# 启动容器,使用-v参数，将宿主机目录testapp/映射到容器内部
# 要求对数据目录web1读写，web2只读。
# ro表示在容器内对该目录只读，默认是可读写的
docker run -d --name web1 -v /root/testapp/:/apps/tomcat/webapps/testapp -p8811:8080 tomcat-web:app1

docker run -d --name web2 -v /root/testapp/:/apps/tomcat/webapps/testapp:ro -p8812:8080 tomcat-web:app2

docker ps

docker run -it --rm web1 bash
echo "web1" > /apps/tomcat/webapps/testapp/index.html  # 可写

docker run -it --rm web2 bash
echo "web2" > /apps/tomcat/webapps/testapp/index.html  # 不可写

```

 

```
# 在宿主机上操作(本地)
mkdir testapp
echo "Test App Page From localhost" > /test/app/index.html
cat /testapp/index.html
# 启动容器,使用-v参数，将宿主机目录testapp/映射到容器内部
# 要求对数据目录web1读写，web2只读。
# ro表示在容器内对该目录只读，默认是可读写的
docker run -d --name web1 -v /root/testapp/:/apps/tomcat/webapps/testapp -p8811:8080 tomcat-web:app1
docker run -d --name web2 -v /root/testapp/:/apps/tomcat/webapps/testapp:ro -p8812:8080 tomcat-web:app2
docker ps
docker run -it --rm web1 bash
echo "web1" > /apps/tomcat/webapps/testapp/index.html  # 可写
docker run -it --rm web2 bash
echo "web2" > /apps/tomcat/webapps/testapp/index.html  # 不可写
```

浏览器验证：

172.16.2.101:8811/testapp/

172.16.2.101:8812/testapp/

在宿主机修改数据

 

```
echo "hello,world." >> /root/testapp/index.html
cat /root/testapp/index.html
```

 

```
echo "hello,world." >> /root/testapp/index.html
cat /root/testapp/index.html
```

浏览器验证：

172.16.2.101:8811/testapp/

172.16.2.101:8812/testapp/

## 删除容器

创建容器的时候指定参数-v，可以删除/var/lib/docker/containers/的容器数据目录，默认不删除，但是不能删除数据卷的内容，如下：

![img](docker%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86.assets/575d9e1f-ce30-4424-981c-d883af07147a.jpg)

## 数据卷的特点及使用

1、数据卷是目录或者文件，并且可以在多个容器之间共同使用；

2、对数据卷更改数据容器里面会立即更新；

3、数据卷的数据可以持久保存，即使删除，使用该容器卷的容器也不影响；

4、在容器里面的写入数据不会影响到镜像本身；

## 文件挂载

文件挂载用于很少更改文件内容的场景，比如nginx 的配置文件、tomcat的配置文件等

 

```
# 创建容器并挂载文件：
ll testapp/catalina.sh
-rwxr-xr-x 1 root root 23705 Jul  3 14:06 testapp/catalina.sh

cat testapp/catalina.sh
"""
#自定义JAVA 选项：
JAVA_OPTS="-server -Xms4g -Xmx4g -Xss512k -Xmn1g -XX:CMSInitiatingOccupancyFraction=65  -XX:+UseFastAccessorMethods -XX:+AggressiveOpts -XX:+UseBiasedLocking -XX:+DisableExplicitGC -XX:MaxTenuringThreshold=10 -XX:NewSize=2048M -XX:MaxNewSize=2048M -XX:NewRatio=2 -XX:PermSize=128m -XX:MaxPermSize=512m -XX:CMSFullGCsBeforeCompaction=5 -XX:+ExplicitGCInvokesConcurrent -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods"
"""

docker run -d --name web1 -v /root/bin/catalina.sh:/apps/tomcat/bin/catalina.sh:ro -p 8811:8080  tomcat-web:app1

# 验证参数是否生效
docker ps
ps -ef | grep java
```

 

```
# 创建容器并挂载文件：
ll testapp/catalina.sh
-rwxr-xr-x 1 root root 23705 Jul  3 14:06 testapp/catalina.sh
cat testapp/catalina.sh
"""
#自定义JAVA 选项：
JAVA_OPTS="-server -Xms4g -Xmx4g -Xss512k -Xmn1g -XX:CMSInitiatingOccupancyFraction=65  -XX:+UseFastAccessorMethods -XX:+AggressiveOpts -XX:+UseBiasedLocking -XX:+DisableExplicitGC -XX:MaxTenuringThreshold=10 -XX:NewSize=2048M -XX:MaxNewSize=2048M -XX:NewRatio=2 -XX:PermSize=128m -XX:MaxPermSize=512m -XX:CMSFullGCsBeforeCompaction=5 -XX:+ExplicitGCInvokesConcurrent -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods"
"""
docker run -d --name web1 -v /root/bin/catalina.sh:/apps/tomcat/bin/catalina.sh:ro -p 8811:8080  tomcat-web:app1
# 验证参数是否生效
docker ps
ps -ef | grep java
```

## 一次挂载多个目录

要求：多个目录要位于不同的目录下

 

```
docker run -d --name web1 -v /root/bin/catalina.sh:/apps/tomcat/bin/catalina.sh:ro -v /root/testapp:/apps/tomcat/webapps/testapp -p 8811:8080 tomcat-web:app1

docker run -d --name web2 -v /root/bin/catalina.sh:/apps/tomcat/bin/catalina.sh:ro -v /root/testapp:/apps/tomcat/webapps/testapp -p 8812:8080 tomcat-web:app2

docker ps
```

 

```
docker run -d --name web1 -v /root/bin/catalina.sh:/apps/tomcat/bin/catalina.sh:ro -v /root/testapp:/apps/tomcat/webapps/testapp -p 8811:8080 tomcat-web:app1
docker run -d --name web2 -v /root/bin/catalina.sh:/apps/tomcat/bin/catalina.sh:ro -v /root/testapp:/apps/tomcat/webapps/testapp -p 8812:8080 tomcat-web:app2
docker ps
```

## 数据卷使用场景

1，日志输出

2，静态web页面

3，应用配置文件

4，多容器间目录或文件共享

# 数据卷容器

数据卷容器最大的功能是可以让数据在多个docker容器之间共享，即可以让B容器访问A容器的内容，而容器C也可以访问A容器的内容，即先要创建一个后台运行的容器作为Server，用于卷提供，这个卷可以为其他容器提供数据存储服务，其他使用此卷的容器作为client端。

1，启动一个卷容器Server

先启动一个容器，并挂载宿主机的数据目录：

 

```
[root@docker-server1 ~]# docker run -d --name volume-docker -v /root/bin/catalina.sh:/apps/tomcat/bin/catalina.sh:ro -v /root/testapp:/apps/tomcat/webapps/testapp tomcat-web:app2
```

 

```
[root@docker-server1 ~]# docker run -d --name volume-docker -v /root/bin/catalina.sh:/apps/tomcat/bin/catalina.sh:ro -v /root/testapp:/apps/tomcat/webapps/testapp tomcat-web:app2
```

2，启动端容器Client：

 

```
[root@docker-server1 ~]# docker run -d --name web1 -p 8801:8080 --volumes-from volume-docker tomcat-web:app1
[root@docker-server1 ~]# docker run -d --name web2 -p 8802:8080 --volumes-from volume-docker tomcat-web:app2
```

 

```
[root@docker-server1 ~]# docker run -d --name web1 -p 8801:8080 --volumes-from volume-docker tomcat-web:app1
[root@docker-server1 ~]# docker run -d --name web2 -p 8802:8080 --volumes-from volume-docker tomcat-web:app2
```

3，进入容器测试读写：

读写权限依赖于源数据卷Server容器

![img](docker%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86.assets/506d22fd-6f80-44bf-9b96-ebcf38ca6a05.jpg)

4，关闭卷容器Server测试能否启动新容器web3：

 

```
[root@docker-server1 ~]# docker stop volume-docker
[root@docker-server1 ~]# docker run -d --name web3 -p 8803:8080 --volumes-from volume-docker tomcat-web:app2
```

 

```
[root@docker-server1 ~]# docker stop volume-docker
[root@docker-server1 ~]# docker run -d --name web3 -p 8803:8080 --volumes-from volume-docker tomcat-web:app2
```

![img](docker%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86.assets/1e79aafe-93f4-4e3d-9a2c-68e50cc53c7f.jpg)

5，测试删除源卷容器Server创建容器web4：

 

```
[root@docker-server1 ~]# docker rm -fv volume-docker
[root@docker-server1 ~]# docker run -d --name web4 -p 8804:8080 --volumes-from volume-docker tomcat-web:app2
```

 

```
[root@docker-server1 ~]# docker rm -fv volume-docker
[root@docker-server1 ~]# docker run -d --name web4 -p 8804:8080 --volumes-from volume-docker tomcat-web:app2
```

![img](docker%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86.assets/f2789adb-ea30-4913-a08b-b0796109f537.jpg)

6，测试之前的容器是否正常

已经运行的容器不受任何影响

7，重新创建容器卷Server：

 

```
[root@docker-server1 ~]# docker run -d --name volume-docker -v /root/bin/catalina.sh:/apps/tomcat/bin/catalina.sh:ro -v /root/testapp:/apps/tomcat/webapps/testapp tomcat-web:app2

[root@docker-server1 ~]# docker run -d --name web4 -p 8804:8080 --volumes-from volume-docker tomcat-web:app2
```



在当前环境下，即使把提供卷的容器Server删除，已经运行的容器Client依然可以使用挂载的卷，因为容器是通过挂载访问数据的，但是无法创建新的卷容器客户端，但是再把卷容器Server创建后即可正常创建卷容器Client，此方式可以用于线上数据库、共享数据目录等环境，因为即使数据卷容器被删除了，其他已经运行的容器依然可以挂载使用。

## 数据卷容器应用场景

数据卷容器可以作为共享的方式为其他容器提供文件共享，类似于NFS共享，可以在生产中启动一个实例挂载本地的目录，然后其他的容器分别挂载此容器的目录，即可保证各容器之间的数据一致性。