[TOC]

# 前言

## 运维工作

--> 系统安装(物理机,虚拟机)

--> 程序包安装,配置,服务启动等

--> 批量操作

--> 程序发布

--> 监控

## 程序发布基本要求

1,不能影响用户体验

2,系统不能停机维护

3,不能造成系统故障导致网站完全不可用

## 预发布验证

新版代码先发布至预发布服务器(跟线上服务器环境和配置相同，但未接入至Director)

## 灰度发布

--> 关闭direcotr上一批服务器

--> 关闭这些服务器上要更新的应用

--> 更新新版本的webapp代码至目标位置

--> 启动服务器上的应用

--> 在director上启动这批服务器

例如:

线上web目录为/web/app/tuangou

测试版本1/webapp/tuangou1.1

测试没问题后,直接做链接至/web/app/目录,并重命名为tuangou

测试版本2/webapp/tuangou1.2

测试没问题后,同样做链接,若有问题可以重新指回版本1,也就是回滚

## 监控系统

不允许没有监控的系统上线

1，监控数据采集

\- 用户行为日志

\- 服务器性能监控

\- 运行数据报告

2，监控管理

\- 异常报警

\- 失效转移

\- 自动优雅降低

## 运维工具

运维工具分类

是否在被管理端安装agent程序

agent(需要): puppet,func

agentless(无需): ansible,fabric,依赖于ssh服务

1,OS Provisioning(OS安装工具)

Cloud: image template

Physical: PXE, Cobbler(repository,distritution,profile)

2,Configuration(OS配置工具)

功能包括安装,配置,服务启动等

ansible(python)

puppet(ruby)

saltstack(python)

func

3,Command and Control(批量操作工具Task Exec)

ansible

fabric

func

4,Deployment(OS部署工具)

fabric

# Ansible简介

官网：www.ansible.com/home

参考中文：http://ansible.com.cn/

由epel源提供，依赖于python相关的程序，可使用yum来安装。

ansible是一个轻量级的运维管理工具,2012年,基于Python研发,可实现对系统，服务，程序等批量管理功能.

仅需在任意客户端安装ansible程序即可实现批量管理被管控主机且被管控的主机无需客户端.

## 核心组件

\- Modules：模块化

\- Core Modules 核心模块

\- Customed Modules 自定义模块

\- Host Iventory 主机库清单，定义要管理的主机

\- Files 可以通过配置文件来实现

\- CMDB 也可以通过外部存储来实现

\- Play Books 剧本(yaml, jinjia2)，定义每个主机即将扮演的角色和执行的任务

\- Hosts 哪些主机

\- Roles 哪些角色

\- Connection Plugins：连接插件，主要连接各管控主机

\- Email

\- Loggin

\- Other

## 特性

1，模块化：调用特定的模块，完成特定的任务(task)； 

2，基于Python语言研发，主要模块由Paramiko, PyYAML和Jinja2三个核心库实现； 

3，部署简单：agent，agentless

4，支持自定义模块，使用任意编程语言； 

5，强大的playbook机制，yaml格式； 

6，具有幂等性(多次执行同一操作，结果相同,比如已经安装过某个程序,再次执行安装任务时,不会再进行安装,会返回一个提示说这个程序已经安装)

7，主从模式：master： ansible， ssh client； slave： ssh server

## 架构

![img](ansible%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/34ae823c-0173-4fbc-89d0-0eea8f006b6a.jpg)

## 执行流程

![img](ansible%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/341750d8-0739-403a-ad03-292e50bc8d71.jpg)

## 安装配置

ansible依赖于Python 2.6 或 更高的版本

### 编译安装方式:

```
# 依赖包
yum -y install python-jinja2 PyYAML python-paramiko python-babel python-crypto

tar xf ansible-1.5.4.tar.gz
cd ansible-1.5.4
python setup.py build
python setup.py install
mkdir /etc/ansible
cp -r examples/* /etc/ansible
```



### YUM安装:

```
# 配置好epel源, 自动解决依赖关系
yum list all | grep ansible
yum info ansible
yum install -y ansible
rpm -ql ansible
```

注意：不同版本的ansible的功能差异可能较大

生成的文件

ansible.cfg: 主配置文件

hosts: 主机分组定义清单(Inventory)

ansible_plugins/: 插件目录

ansible-playbook: 定义剧本的文件

ansible-vault: 用于加密解密playbook

ansible-doc: 获取帮助的程序

### 部署：

ansible通过ssh协议实现配置管理、应用部署、任务执行等功能，因此，需要事先配置ansible服务端能免密登录各被管理节点.

免密方式：

a.基于秘钥认证方式,将管理端的公钥上传至被管理端的/root/.ssh/authorized_keys;

b.在inventory文件中指定被管理端的账号和密码;

```
# 方式一(推荐):
ssh-keygen -t rsa
# 一路回车
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.100.8
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.100.9
ssh root@172.16.100.8 'date'
ssh root@172.16.100.9 'date'
# 如果ssh端口非默认21
ssh-copy-id -i /root/.ssh/id_rsa.pub '-p 2201 root@172.16.100.10'
# 验证
ssh -p 2201 root@172.16.100.8 'date'

# 方式二(不安全):
vim /etc/ansible/ansible.cfg
'''
remote_user = root
remote_port = 21
private_key_file = /root/.ssh/id_rsa.pub
'''
```

### 工具

获取可支持的所有模块帮助信息

ansible-doc -l

查看某个模块的帮助信息

ansible-doc -s MODULES_NAME

读取执行写好的playbook文件[.yml格式]，即一些任务的集合

ansible-playbook playbook.yml

上传下载Roles：https://galaxy.ansible.com

ansible-galaxy --help

拉取，默认push模式，用于数量巨大的机器配置，以及没有网络的主机上运行ansible

ansible-pull --help

加密配置文件

ansible-vault

进入类似shell的交互模式

ansible-console

**ansible命令**

语法格式: 

ansible <host-pattern> [-f forks] [-m module_name] [-a args]

host-pattern:对哪些主机进行管理,支持正则

-k : 表示使用时手动输入远程主机的密码来连接

-f forks: 启动的并发线程数,一次管理多少主机

-m module_name: 要使用的模块

-a args: 模块特有的参数

### Ad-Hoc命令：

所谓Ad-Hoc,简而言之是"临时命令",英文中作为形容词有"特别的,临时"的含义。

Ad-Hoc只是官方对Ansible命令的一种称谓。

从功能上讲,Ad-Hoc是相对于Ansible-playbook而言的,Ansible提供两种完成任务方式:

一种是Ad-Hoc命令集,即ansible

另一种就是Ansible-playbook了,即Ansible-playbook命令

前者更注重于解决一些简单的或者平时工作中临时遇到的任务,相当于Linux系统命令行下的Shell命令,后者更适合与解决复杂或需固化下来的任务,相当于Linux系统的Shell Scripts。

需要使用Ad-Hoc的场景

情景1:

节假日将至,我们需要关闭所有不必要的服务器,并对所有服务器进行节前健康检查。

情景2:

临时更新Apache&Nginx的配置文件,且需同时将其分发至所有需更新该配置的Web服务器。

需要使用ansible-playbook的场景

情景1:

新购置的服务器安装完系统后需做一系列固化的初始化工作,诸如:定制防火墙策略、时间同步配置、添加EPEL源等。

情景2:

每周定期对生产环境发布更新程序代码

Ad-Hoc使用格式如下，命令集通过/usr/bin/ansible命令实现:

ansible <host-pattern> [options]

options:

-v,--verbose  输出执行过程信息

verbose mode (-vvv for more, -vvvv to enable connection debugging)

-i PATH,--inventory-file=PATH  指定inventory信息,默认/etc/ansible/hosts)

-f NUM,--forks=NUM  并发线程数,默认5

--private-key=PRIVATE_KEY_FILE,--key-file=PRIVATE_KEY_FILE  指定秘钥文件

-m MODUEL_NAME,--module-name=MODULE_NAME  指定模块名(默认command)

-M DIRECTORY,--module-path=DIRECTORY  指定模块存放路径,默认/usr/share/ansible ,可通过ANSIBLE_LIBRARY设定默认路径

-a 'ARGUMENTS',--args='ARGUMENTS'  模块参数

-k,--ask-pass SSH_PWD  认证ssh密码

-K,--ask-sudo-pass SUDO_PWD  认证sudo密码

--ask-su-pass  认证su密码

-o,--one-line  标准输出至一行

-s,--sudo  相当于linux系统下的sudo

-t DIRECTORY,--tree=DIRECTORY  输出信息至目录下，以远程主机名命名

-T SECONDS,--timeout=SECONDS  连接远程主机的最大超时时间,单位秒

-B NUM,--background=NUM  后台执行命令，超过NUM秒后中止正在执行的任务

-P NUM,--poll=NUM  定期返回后台任务进度

-u USERNAME,--user=USERNAME  指定远程运行命令的用户

-U SUDO_USERNAME,--sudo-user=SUDO_USERNAME  指定sudo的用户

-c CONNECTION,--connection=CONNECTION  指定连接方式,可用选项paramiko(SSH),ssh,local(常用于crontab和kickstarts)

-l SUBSET,--limit=SUBSET  指定运行主机

-l ~REGEX,--limit=~REGEX  指定运行主机(正则)

--list-hosts  列出符合条件的主机列表而不执行命令

### 简单示例

```
# 定义被管理主机,因为ssh默认端口我修改过了,所以特别指明了一下
cat /etc/ansible/hosts
"""
[websrvs]
172.16.100.10
172.16.100.11

[dbsrvs]
172.16.100.12

[websrvs:vars]
ansible_ssh_port=2201

[dbsrvs:vars]
ansible_ssh_port=2201
"""

# 检查websrvs主机组在线状态
ansible websrvs -f 5 -m ping

# 查看websrvs组所有主机各自的主机名
ansible websrvs -m command -a "hostname"

# 查看websrvs主机组的所有主机
ansible websrvs --list

# 在websrvs主机组上执行shell命令
ansible websrvs -m shell -a "df -h"
ansible websrvs -m shell -a "free -m"

# 仅对websrvs主机组下的一台主机执行ping
ansible websrvs -m ping --limit "192.168.2.5"
```

## **常用模块**

### *文件管理*

**copy** 复制文件

格式：ansible-doc -s copy

```
# src源, dest目地(绝对路径)，owner属主, mode权限, force=yes是否覆盖文件
ansible test -m copy -a "src=/etc/fstab dest=/tmp/fstab.ansible owner=root mode=600 force=yes"

# content内容为文件的内容,会代替src(不能同时使用),表示直接用此处指定的信息生成为目标文件内容
ansible test -m copy -a 'content="Hello,Ansible\nHi girl!\n" dest=/tmp/test.ansible'
```

**fetch** 节点复制到管理机

格式：ansible-doc -s fetch

```
ansible test -m copy -a "dest=/root src=/tmp/test.txt force=yes"
```

**file** 设定文件属性

格式：ansible-doc -s file

```
# 创建链接文件
ansible test -m file -a "path=/tmp/test.txt src=/tmp/fstab.ansible state=link"
```

path=文件路径

state=默认值file创建文件

directory创建文件夹

link创建软连接

hard创建硬链接

absent删除文件

src=文件链接路径

mode=文件权限

owner=属主

group=属组

recurse=yes递归设置属性

**stat** 获取远程文件状态信息

格式：ansible-doc -s stat

```
ansible test -m stat -a "path=/etc/sysctl.conf"
```

**setup** 获取远程主机的facts

每个被管理节点在接收并运行管理命令之前,会将自己主机相关信息(如系统版本,ip地址等)报告给远程的ansible主机

格式：ansible-doc -s setup

```
ansible test -m setup
```

### *命令执行模块*

**command**

格式：ansible-doc -s command

支持的参数有:

chdir=执行前先进入的某个目录

creates=文件,假如文件存在则不会执行命令

removes=文件,假如文件不存在则不会执行命令

注：ansible的默认模块，该模块获取不到环境变量，管道和重定向都不能使用

**shell**

格式：ansible-doc -s shell

用法与command类似，区别在于shell支持管道,变量或重定向等

```
# shell与command对比
# 无法设置密码
ansible all -m command -a "echo 123.com | passwd --stdin user1"

# 设置密码成功
ansible all -m shell -a "echo 123.com | passwd --stdin user1"
```

**script**

格式：ansible-doc -s script

执行脚本模块，可将本地脚本下发到节点主机中执行

```
# 要使用相对路径指定脚本
vim test.sh
"""
echo "hello,world."
"""
ansible test -m script -a "test.sh"
```

**raw**

格式：ansible-doc -s raw

直接ssh的方式，不通过python，节点没有python也可以使用

**cron**

格式：ansible-doc -s cron

定义计划任务

常用参数:

state=present  创建,安装

state=absent 删除,移除

```
# 在websrvs主机组中创建任务计划,每10分钟执行/bin/echo hello,world.,该任务计划的描述为"test cron job"
ansible websrvs -m cron -a 'minute="*/10" job="/bin/echo hello,world." name="test cron job"'

# 验证
ansible websrvs -a 'crontab -l'

# 移除计划任务
ansible websrvs -m cron -a 'minute="*/10" job="/bin/echo hello,world." name="test cron job" state=absent'

ansible websrvs -a 'crontab -l'
```

### *网络相关*

**ping**  测试节点是否连通

格式：ansible all -m ping 

all表示所有主机

**get_url**  用于下载网络上的文件

格式：ansible test -m get_url -a 'url=http://10.1.1.116/favicon.ico dest=/tmp'   #把http://10.1.1.116/favicon.ico下载到/tmp

**uri**  用于发送HTTP协议，让节点主机向指定的网址发送Get、Post这样的HTTP请求，并且返回状态码

### *包管理模块*

**yum**

格式：使用yum模块时，管理机设置yum源就好

例：

```
ansible test -m yum -a "name=httpd state=present"    #安装apache
ansible test -m yum -a "name=httpd state=absent"    #删除apache
ansible test -m yum -a "name=httpd state=latest"    #更新apache
ansible test -m yum -a "name=httpd enablerepo=testing state=absent"    #用testing这个repo安装apache
ansible test -m yum -a "name=* state=latest"    #upgrade all packages
ansible test -m yum -a "name=http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm state=present"    #install the nginx rpm from a remote repo
ansible test -m yum -a "name=/usr/local/src/nginx-release-centos-6-0.el6.ngx.noarch.rpm state=present"    #install nginx rpm from a local file
ansible test -m yum -a "name="@Development tools" state=present"    #install the 'Development tools' package group
```

**pip和easy_install模块**

**APT**

### *系统管理*

**service** 管理服务

格式：ansible test -m service -a "enabled=true name=httpd state=started"

arguments: 可向服务传递的参数

enabled=true|false: 是否开机自动启动

name: 表示控制哪个服务

pattern: 

runlevel: 表示在哪些级别下自启

sleep:

state=started|stopped|restarted|reloaded : 控制服务

**group** 组管理

格式：ansible-doc -s group

```
# name=test添加test组,gid=组的GID,system=yes/no是否是系统组,state=present/absent删除还是创建
ansible test -m group -a "name=test state=present"    
```

**user** 用户管理

格式：ansible-doc -s user

```
# 新建test用户,group指定属组，groups指定附加组，'groups='删除所有附加组，append=yes增量添加属组，state默认present，表示新建
ansible test -m user -a "name=test shell=/bin/bash groups=admin,dba append=yes home=/home/test state=present"

# test用户附加组改为dba，append=no全量变更属组信息。
ansible test -m user -a "name=test groups=dba append=no"

# generate_ssh_key=yes生成SSH key，ssh_key_bits指定加密位数，默认2048，ssh_key_file指定key文件位置，默认.ssh/id_rsa，相对路径表示家目录下
ansible test -m user -a "name=test1 generate_ssh_key=yes ssh_key_bits=2048 ssh_key_file=.ssh/id_rsa"    

# 删除test用户，state=absent表示删除用户，remove=yes结合state=absent相当于userdel --remove
ansible test -m user -a "name=test state=absent remove=yes"

# 变更用户密码，使用加密密码
openssl passwd -1 -salt `openssl rand -hex 8`
password:  # 输入一个密码,来进行加密
ansible test -m user -a "name=test password=加密后的密码串"
```

注:加密方式还可以使用python的passlib、getpass库

**setup** 收集远程主机的facts

每个被管理远程节点在接收并运行管理命令之前,会将自己的相关信息,比如操作系统版本,ip地址等报告给ansible主机

```
# 查看系统变量
ansible all -m setup
```