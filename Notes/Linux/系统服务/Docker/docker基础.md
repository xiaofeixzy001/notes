[TOC]

# **前言** 

容器技术是虚拟化、云计算、大数据之后的一门新兴的并且是炙手可热的新技术，容器技术提高了硬件资源利用率、方面了业务

快速横向扩容、实现了业务宕机自愈功能，因此未来数年会是一个容器愈发流行的时代，这是一个对于IT行业来说非常有影响和价值的技术，而对于IT行业的从业者来说，熟练掌握容器技术无疑是一个很有前景的的行业工作机会。

# **docker简介**

## 1,什么是Docker

首先Docker是一个在2013年开源的应用程序并且是一个基于go语言编写是一个开源的pass服务(Platform as a 

Service，平台即服务的缩写)，go语言是由google开发，docker公司最早叫dotCloud后由于Docker开源后大受欢迎就将公

司改名为 Docker Inc，总部位于美国加州的旧金山，Docker是基于linux 内核实现，Docker最早采用LXC技术(Linux 

Container的简写，LXC是Linux 原生支持的容器，可以提供轻量级的虚拟化，可以说 docker 就是基于 LXC 发展起来的，

提供 LXC 的高级封装，发展标准的配置方法)，而虚拟化技术KVM(Kernel-based Virtual Machine) 基于模块实现，后

改为自己研发的runc运行容器。

Docker 相比虚拟机的交付速度更快，资源消耗更低，Docker 采用客户端/服务端架构，使用远程API来管理和创建Docker容

器，其可以轻松的创建一个轻量级的、可移植的、自给自足的容器，docker 的三大理念是build(构建)、ship(运输)、 

run(运行)，Docker遵从aoache 2.0协议，并通过（namespace及cgroup等）来提供容器的资源隔离与安全保障等，所以

Docke容器在运行时不需要类似虚拟机（空运行的虚拟机占用物理机6-8%性能）的额外资源开销，因此可以大幅提高资源利用

率,总而言之Docker是一种用了新颖方式实现的轻量级虚拟机.类似于VM但是在原理和应用上和VM的差别还是很大的，并且

docker的专业叫法是应用容器(Application Container)。

IDC/IAAS/PAAS/SAAS对比图

![img](docker%E5%9F%BA%E7%A1%80.assets/23087b0c-ce56-4d0a-a5af-ffcefa3daab7-1587885150862.jpg)

## **2,docker的组成**

client(客户端): 客户端使用的dokcer命令或其他可调用docker API的工具

server(服务端): docker守护进程,运行docker容器

images(镜像): 镜像可以理解为创建实例使用的模板,类似win下ghost的镜像

container(容器): 容器是从镜像生成对外提供提供服务的一个或一组服务,类似虚机实例

registry(仓库): 保存镜像的仓库,类似于git或svn这样的版本控制系统,官方仓库:https://hub.docker.com/

host(docker主机): 一个物理机或虚拟机,用于运行docker服务进程和容器

![img](docker%E5%9F%BA%E7%A1%80.assets/262777d9-6ae2-4b91-84ca-5024fcfae717-1587885150862.jpg)

## **3,docker对比虚拟机** 

资源利用率更高,一台物理机可以运行数百个容器,但是一般只能运行数十个虚拟机;

开销更小,不需要启动单独的虚拟机占用硬件资源;

启动速度更快,可以在数秒内完成启动;

二者架构对比

![img](docker%E5%9F%BA%E7%A1%80.assets/5e64efbb-b69c-41fe-b359-c81290a37cf7-1587885150862.jpg)

## 4,docker优势

\- 快速部署,短时间内可以部署成百上千个应用,更快速的交付到线上;

\- 高效虚拟化,不需要额外的hypervisor支持,直接基于linux实现应用虚拟化;

\- 相比虚拟机大幅度提高性能和效率;

\- 节省开支,提高服务器利用率,降低IT支出;

\- 简化配置,将运行环境打包保存至容器,使用时直接启动即可;

\- 快速迁移和扩展,可跨平台运行在物理机|虚拟机|公有云等环境,良好的兼容性可以很方便的将应用从A平台迁移到B;

## 5,docker缺点

\- 隔离性,各应用之间的隔离不如虚拟机;

\- 安全性,目前docker server无法判断容器是由哪个用户启动的,删除的时候不会做权限审核,即A用户可删除B用户启动的容器;

\- 稳定性和功能性,目前docker版本还在快速迭代更新,很多新功能依赖于较高版本的linux内核,因此会存在版本兼容性问题;

## 6,docker(容器)的核心技术

\- 容器规范

除了docker之外的docker技术，还有coreOS的rkt，还有阿里的Pouch，为了保证容器生态的标志性和健康可持续发展，包括Google、Docker等公司共同成立了一个叫open container（OCI）的组织，其目的就是制定开放的标准的容器规范，目前OCI一共发布了两个规范，分别是runtime spec和image format spec，有了这两个规范，不通的容器公司开发的容器只要兼容这两个规范，就可以保证容器的可移植性和相互可操作性。

\- 容器runtime

runtime是真正运行容器的地方,因此为了运行不同的容器runtime,需要和操作系统内核紧密合作相互支持,以便为容器提供相应的运行环境.

目前主流的三种runtime:

\- Lxc, linux上早期的runtime,docker早期就是采用lxc作为runtime

\- runc, 似乎docker自己开发的容器runtime,也是目前默认的runtime,runc遵守OCI规范,因此可兼容lxc

\- rkt, CoreOS开发的容器runtime,也符合OCI规范,所以使用rkt的runtime也可以运行docker容器

\- 容器管理工具

管理工具连接runtime和用户,对用户提供图形或命令方式操作,然后管理工具将用户操作传递给runtime执行.好比汽车(容器)的方向盘和油门.

lxc的管理工具是Lxd;

runc的管理工具是docker engine,其包含后台deamon和cli两部分,大家经常提到的docker就是指的docker engine;

rkt的管理工具是rkt cli;

\- 容器定义工具

容器定义工具允许用户定义容器的属性和内容,以方便容器能够被保存,共享和重建.

docker image: 是容器的模板,runtime依据docker image创建容器;

dockerfile: 包含N个命令的文本文件,通过dockerfile创建docker image;

ACI(App Container Image): 与docker image类似,是CoreOS开发的rkt容器的镜像格式;

\- Registry仓库

统一保存共享镜像的地方,叫做镜像仓库.

Image registry: docker官方提供的私有仓库部署工具;

Docker hub: docker官方的公共仓库,已经保存了大量的常用镜像,可以方便大家直接使用;

Harbor: vmware提供的自带web的镜像仓库,目前有很多公司使用;

\- 编排工具

当多个容器在多个主机运行的时候，单独管理每个容器是相当负载而且很容易出错，而且也无法实现某一台主机宕机后容器自动

迁移到其他主机从而实现高可用的目的，也无法实现动态伸缩的功能，因此需要有一种工具可以实现统一管理、动态伸缩、故障

自愈、批量执行等功能，这就是容器编排引擎。

容器编排通常包括容器管理、调度、集群定义和服务发现等功能。

Docker swarm: docker开发的容器编排引擎;

Kubernetes: google领导开发的容器编排引擎,且同时支持Docker和CoreOS;

Mesos: 通用的集群组员调度平台,mesos与marathon一起提供容器编排引擎功能;

## 7,docker容器的依赖技术

\- 容器网络

docker自带的网络docker network仅支持管理单机上的容器网络，当多主机运行的时候需要使用第三方开源网络，例如calico、flannel等。

\- 服务发现

容器的动态扩容特性决定了容器IP也会随之变化，因此需要有一种机制开源自动识别并将用户请求动态转发到新创建的容器上，kubernetes自带服务发现功能，需要结合kube-dns服务解析内部域名。

\- 容器监控

可以通过原生命令docker ps/top/stats 查看容器运行状态，另外也可以使heapster/Prometheus等第三方监控工具监控容器的运行状态。

\- 数据管理

容器的动态迁移会导致其在不同的Host之间迁移，因此如何保证与容器相关的数据也能随之迁移或随时访问，可以使用逻辑卷/存储挂载等方式解决。

\- 日志收集

docker 原生的日志查看工具docker logs，但是容器内部的日志需要通过ELK等专门的日志收集分析和展示工具进行处理。