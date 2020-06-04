[TOC]

# 网络名称空间

```shell
rpm -q iproute

# 添加网络名称空间
ip netns help
ip netns add r1
ip netns add r2

# 查看
ip netns list

# 在网络名称空间中执行命令
ip netns exec r1 ifconfig -a
ip netns exec r2 ifconfig -a

# 创建虚拟网卡对
ip link help
ip link add veth1.1 type veth peer name veth1.2
ip link show

# 将veth1.2添加到网络名称空间r2中
ip link set dev veth1.2 netns r2
ip link show
ip netns exec r2 ifconfig -a

# 改名
ip netns exec r2 ip link set dev veth1.2 name eth0

# 激活
ifconfig veth1.1 10.1.1.1/24 up
ip netns exec r2 ifconfig eth0 10.1.1.2/24 up
ip netns exec r2 ifconfig

# 测试物理机与虚拟机通信
ping 10.1.1.2

# 将veth1.1添加到网络名称空间r1中
ip link set dev veth1.1 netns r1
ip link show
ip netns exec r1 ifconfig -a
ip netns exec r1 ip link set dev veth1.1 name eth0
ip netns exec r1 ifconfig eth0 10.1.1.1/24 up
ip netns exec r1 ifconfig

# 测试两个虚拟机之间通信
ip netns exec r1 ping 10.1.1.2
```



# 通过容器名称互联

即在同一个宿主机上的容器之间可以通过自定义的容器名称相互访问，比如一个业务前端静态页面是使用nginx，动态页面使用的是tomcat，由于容器在启动的时候其内部IP地址是DHCP 随机分配的，所以如果通过内部访问的话，自定义名称是相对比较固定的，因此比较适用于此场景。

此方式最少需要两个容器之间操作，这里使用到了--link选项

先创建第一个容器，后续会使用到这个容器的名称：

 

```
docker run --name nginx-1 -d -p 8801:80  nginx-web01 nginx
'''
c045c82e85bd620eb9444275b135634a9248760e2061505a1c8b4167e9b24b3d
'''

docker run -it nginx-1 bash
[root@c045c82e85bd /]# cat /etc/hosts
'''
127.0.0.1   localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2  c045c82e85bd
'''

docker run -d --name nginx-2 --link nginx-1  -p 8802:80 nginx-web02 nginx
'''
23813883ed977d4cc1e50355adaf37564832fc90b4b8b307866c1c99c8256c57
'''

docker run -it nginx-1 bash
[root@23813883ed977 /]# cat /etc/hosts
'''
127.0.0.1   localhost
127.0.0.1   localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2 nginx-1 c045c82e85bd  # 第一个容器的名称和ID,只会添加到本地不会添加到对方
172.17.0.3  23813883ed97
'''
[root@23813883ed977 /]# ping nginx-1
```

 

```
docker run --name nginx-1 -d -p 8801:80  nginx-web01 nginx
'''
c045c82e85bd620eb9444275b135634a9248760e2061505a1c8b4167e9b24b3d
'''
docker run -it nginx-1 bash
[root@c045c82e85bd /]# cat /etc/hosts
'''
127.0.0.1   localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2  c045c82e85bd
'''
docker run -d --name nginx-2 --link nginx-1  -p 8802:80 nginx-web02 nginx
'''
23813883ed977d4cc1e50355adaf37564832fc90b4b8b307866c1c99c8256c57
'''
docker run -it nginx-1 bash
[root@23813883ed977 /]# cat /etc/hosts
'''
127.0.0.1   localhost
127.0.0.1   localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2 nginx-1 c045c82e85bd  # 第一个容器的名称和ID,只会添加到本地不会添加到对方
172.17.0.3  23813883ed97
'''
[root@23813883ed977 /]# ping nginx-1
```

# 通过自定义容器别名互联

上一步骤中，自定义的容器名称可能后期会发生变化，那么一旦名称发生变化，程序之间也要随之发生变化，比如程序通过容器名称进行服务调用，但是容器名称发生变化之后再使用之前的名称肯定是无法成功调用，每次都进行更改的话又比较麻烦，因此可以使用自定义别名的方式解决，即容器名称可以随意更，只要不更改别名即可，具体如下：

命令格式：

 

```
docker run -d --name 新容器名称 --link 目标容器名称1:自定义的名称1 --link 目标容器名称2:自定义的名称2 --link ...   -p 本地端口:容器端口 镜像名称 shell命令
```

 

```
docker run -d --name 新容器名称 --link 目标容器名称1:自定义的名称1 --link 目标容器名称2:自定义的名称2 --link ...   -p 本地端口:容器端口 镜像名称 shell命令
```

启动第三个容器

 

```
docker run -d --name nginx-3 --link nginx-1:web1 -p 8803:80 nginx-app01 nginx
'''
9c5d5b70205e17de262cf7abe5393988d41581d5440f576ca3b9b8f8f71dd485
'''

bash docker-in.sh nginx-3

[root@9c5d5b70205e /]# cat /etc/hosts
'''
127.0.0.1   localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2  web1 d1877cad5f0a nginx-1
172.17.0.4  9c5d5b70205e
'''

[root@9c5d5b70205e /]# ping web1
'''
PING web1 (172.17.0.2) 56(84) bytes of data.
64 bytes from web1 (172.17.0.2): icmp_seq=1 ttl=64 time=0.523 ms
64 bytes from web1 (172.17.0.2): icmp_seq=2 ttl=64 time=0.131 ms
'''

docker network list
'''
NETWORK ID          NAME                DRIVER              SCOPE
36a6c6bf1301        bridge              bridge              local
35ea9bed62dc        host                host                local
aea7309e73ef        none                null                local
'''
```

 

```
docker run -d --name nginx-3 --link nginx-1:web1 -p 8803:80 nginx-app01 nginx
'''
9c5d5b70205e17de262cf7abe5393988d41581d5440f576ca3b9b8f8f71dd485
'''
bash docker-in.sh nginx-3
[root@9c5d5b70205e /]# cat /etc/hosts
'''
127.0.0.1   localhost
::1 localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2  web1 d1877cad5f0a nginx-1
172.17.0.4  9c5d5b70205e
'''
[root@9c5d5b70205e /]# ping web1
'''
PING web1 (172.17.0.2) 56(84) bytes of data.
64 bytes from web1 (172.17.0.2): icmp_seq=1 ttl=64 time=0.523 ms
64 bytes from web1 (172.17.0.2): icmp_seq=2 ttl=64 time=0.131 ms
'''
docker network list
'''
NETWORK ID          NAME                DRIVER              SCOPE
36a6c6bf1301        bridge              bridge              local
35ea9bed62dc        host                host                local
aea7309e73ef        none                null                local
'''
```

启动容器时，可以指定使用哪种网络，默认bridge

--net=bridge|host|net|none

# 通过网络跨宿主机互联

同一个宿主机之间的各容器之间是可以直接通信的，但是如果访问到另外一台宿主机的容器呢？

server1：172.16.2.101

server2:172.16.2.102

## docker网络类型

Docker 的网络有四种类型，下面将介绍每一种类型的具体工作方式：

Bridge模式，使用参数 –net=bridge 指定，不指定默认就是bridge模式。

### Host 模式

Host 模式，使用参数 –net=host 指定。

启动的容器如果指定了使用host模式，那么新创建的容器不会创建自己的虚拟网卡，而是直接使用宿主机的网卡和IP地址，因此在容器里面查看到的IP信息就是宿主机的信息，访问容器的时候直接使用宿主机IP+容器端口即可，不过容器的其他资源们比如文件系统、系统进程等还是和宿主机保持隔离。

此模式的网络性能最高，但是各容器之间端口不能相同，适用于运行容器端口比较固定的业务。

为避免端口冲突，先删除所有的容器：

先确认宿主机端口没有占用80端口

 

```
ss -tnlp
ifconfig
docker run -d --name net_host_nginx --net=host nginx-app01 nginx
'''
abdc7c9c1e984278bc9344393edf493c3cb09929afb83e34bcd21179d226b61a
'''

bash docker_in.sh net_host_nginx
[root@abdc7c9c1e /]# hostname
[root@abdc7c9c1e /]# ifconfig
```

 

```
ss -tnlp
ifconfig
docker run -d --name net_host_nginx --net=host nginx-app01 nginx
'''
abdc7c9c1e984278bc9344393edf493c3cb09929afb83e34bcd21179d226b61a
'''
bash docker_in.sh net_host_nginx
[root@abdc7c9c1e /]# hostname
[root@abdc7c9c1e /]# ifconfig
```

### None模式

None模式，使用参数 –net=none 指定

在使用none 模式后，Docker 容器不会进行任何网络配置，其没有网卡、没有IP也没有路由，因此默认无法与外界通信，需要手动添加网卡配置IP等，所以极少使用，

命令使用方式：

 

```
docker run -d --name net_none --net=none nginx-app01 nginx
'''
143ce15733c1961bf6e23989cbd7d76fc9f9297dc7f11e610ae30c418c21297c
'''

bash docker_in.sh net_host_nginx
[root@143ce15733 /]# ifconfig
```

 

```
docker run -d --name net_none --net=none nginx-app01 nginx
'''
143ce15733c1961bf6e23989cbd7d76fc9f9297dc7f11e610ae30c418c21297c
'''
bash docker_in.sh net_host_nginx
[root@143ce15733 /]# ifconfig
```

### Container模式

Container模式，使用参数 –net=container:名称或ID 指定。

使用此模式创建的容器需指定和一个已经存在的容器共享一个网络，而不是和宿主机共享网，新创建的容器不会创建自己的网卡也不会配置自己的IP，而是和一个已经存在的被指定的容器共享IP和端口范围，因此这个容器的端口不能和被指定的端口冲突，除了网络之外的文件系统、进程信息等仍然保持 相互隔离，两个容器的进程可以通过lo网卡通信。

 

```
# 启动一个容器
docker run -d --name nginx-web1 nginx-app01 nginx
'''
95d22a5d36c18544af47373cc227a1679f239e790f86907d310d13ef4eb85d5e
'''

# 再启动一个容器，绑定到之前的容器网卡上
docker run -it --name net_container --net=container:nginx-web1 nginx-app01 bash
```

 

```
# 启动一个容器
docker run -d --name nginx-web1 nginx-app01 nginx
'''
95d22a5d36c18544af47373cc227a1679f239e790f86907d310d13ef4eb85d5e
'''
# 再启动一个容器，绑定到之前的容器网卡上
docker run -it --name net_container --net=container:nginx-web1 nginx-app01 bash
```

 直接使用对方的网络，较少使用

### Bridge模式

docker的默认模式，即不指定任何模式就是bridge模式，也是使用比较多的模式，此模式创建的容器会为每一个容器分配自己的网络 IP等信息，并将容器连接到一个虚拟网桥与外界通信。

 

```
docker run -d  --name  net_bridge nginx-app01 /usr/sbin/nginx
'''
58e251cdb17fc309ee364c332549b434f4d51019b793702e81b5edb1ff701a7c
'''

bash docker-in.sh net_bridge

[root@58e251cdb17 /]# ifconfig
```

 

```
docker run -d  --name  net_bridge nginx-app01 /usr/sbin/nginx
'''
58e251cdb17fc309ee364c332549b434f4d51019b793702e81b5edb1ff701a7c
'''
bash docker-in.sh net_bridge
[root@58e251cdb17 /]# ifconfig
```

## 跨主机互联简单实现

夸主机互联是说A宿主机的容器可以访问B主机上的容器，但是前提是保证各宿主机之间的网络是可以相互通信的，然后各容器才可以通过宿主机访问到对方的容器，实现原理是在宿主机做一个网络路由就可以实现A宿主机的容器访问B主机的容器的目的，复杂的网络或者大型的网络可以使用google开源的k8s进行互联。

### 1，修改各宿主机网段

Docker的默认网段是172.17.0.x/24,而且每个宿主机都是一样的，因此要做路由的前提就是各个主机的网络不能一致，具体如下：

 

```
# 问避免影响，先在各服务器删除之前穿件的所有容器
docker rm -f `docker ps -a -q`

# server1容器默认ip改为192.168.11.1/24
vim /usr/lib/systemd/system/docker.service
'''
ExecStart=/usr/bin/dockerd-current --bip=192.168.11.1/24
'''

systemctl daemon-reload
systemctl restart docker

ifconfig


# server2容器默认ip改为192.168.12.1/24
vim /usr/lib/systemd/system/docker.service
'''
ExecStart=/usr/bin/dockerd-current --bip=192.168.12.1/24
'''

systemctl daemon-reload
systemctl restart docker

ifconfig
```

 

```
# 问避免影响，先在各服务器删除之前穿件的所有容器
docker rm -f `docker ps -a -q`
# server1容器默认ip改为192.168.11.1/24
vim /usr/lib/systemd/system/docker.service
'''
ExecStart=/usr/bin/dockerd-current --bip=192.168.11.1/24
'''
systemctl daemon-reload
systemctl restart docker
ifconfig
# server2容器默认ip改为192.168.12.1/24
vim /usr/lib/systemd/system/docker.service
'''
ExecStart=/usr/bin/dockerd-current --bip=192.168.12.1/24
'''
systemctl daemon-reload
systemctl restart docker
ifconfig
```

### 2，在server1和server2上分别启动一个实例

 

```
# server1
docker run -d --name test-net-vm nginx-app01 nginx
'''
5d625a868718656737d452d6e57cd23a8a9a1cc9d2e8f76057f1977da2b52a60
'''

bash docker-in.sh test-net-vm

[root@5d625a8687 /]# ifconfig


# server2
docker run -d --name test-net-vm nginx-app01 nginx
'''
ce8fbe4c1d6b5515136ac280f87e6c03884daa35996e996f124ac7056f609e0c
'''

bash docker-in.sh test-net-vm

[root@ce8fbe4c1 /]# ifconfig
```

 

```
# server1
docker run -d --name test-net-vm nginx-app01 nginx
'''
5d625a868718656737d452d6e57cd23a8a9a1cc9d2e8f76057f1977da2b52a60
'''
bash docker-in.sh test-net-vm
[root@5d625a8687 /]# ifconfig
# server2
docker run -d --name test-net-vm nginx-app01 nginx
'''
ce8fbe4c1d6b5515136ac280f87e6c03884daa35996e996f124ac7056f609e0c
'''
bash docker-in.sh test-net-vm
[root@ce8fbe4c1 /]# ifconfig
```

### 3，在FORWARD链上加一条规则，并添加一条静态路由

 

```
# server1添加规则：允许转发来自172.16.2.0网络的包
iptables -A FORWARD -s 172.16.2.0/16 -j ACCEPT

# 添加静态路由，将目标网络为192.168.12.0/24的请求，下一跳交给server2的地址172.16.2.102
route add -net 192.168.12.0/24 gw 172.16.2.102


# server2添加规则：允许转发来自172.16.2.0网络的包
iptables -A FORWARD -s 172.16.2.0/16 -j ACCEPT

# 添加静态路由，将目标网络为192.168.11.0/24的请求，下一跳交给server1的地址172.16.2.101
route add -net 192.168.11.0/24 gw 172.16.2.101
```

 

```
# server1添加规则：允许转发来自172.16.2.0网络的包
iptables -A FORWARD -s 172.16.2.0/16 -j ACCEPT
# 添加静态路由，将目标网络为192.168.12.0/24的请求，下一跳交给server2的地址172.16.2.102
route add -net 192.168.12.0/24 gw 172.16.2.102
# server2添加规则：允许转发来自172.16.2.0网络的包
iptables -A FORWARD -s 172.16.2.0/16 -j ACCEPT
# 添加静态路由，将目标网络为192.168.11.0/24的请求，下一跳交给server1的地址172.16.2.101
route add -net 192.168.11.0/24 gw 172.16.2.101
```

ping测试连通性

抓包分析：tcpdump -i ens33 -vnn icmp

两台主机的容器测试