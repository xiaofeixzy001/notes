[TOC]

# YAML介绍

YAML是一个可读性高的用来表达资料序列的格式。YAML参考了其他多种语言，包括：XML、C语言、Python、Perl以及电子邮件格式RFC2822等。Clark Evans在2001年在首次发表了这种语言，另外Ingy döt Net与Oren Ben-Kiki也是这语言的共同设计者。

YAML Ain't Markup Language，即YAML不是XML。不过，在开发的这种语言时，YAML的意思其实是："Yet Another Markup Language"（仍是一种标记语言）

## 特性

YAML的可读性好

YAML和脚本语言的交互性好

YAML使用实现语言的数据类型

YAML有一个一致的信息模型

YAML易于实现

YAML可以基于流来处理

YAML表达能力强，扩展性好

更多的内容及规范参见http://www.yaml.org。

# YAML语法

YAML的语法和其他高阶语言类似，并且可以简单表达清单、散列表、标量等数据结构。其结构（Structure）通过空格来展示，序列（Sequence）里的项用"-"来代表，Map里的键值对用":"分隔。下面是一个示例。

```
name: John Smith
age: 41
gender: Male
spouse:
    name: Jane Smith
    age: 37
    gender: Female
children:
    -  name: Jimmy Smith
        age: 17
        gender: Male
    -  name: Jenny Smith
        age 13
        gender: Female
```

YAML文件扩展名通常为.yaml，如example.yaml

## list

列表的所有元素均使用“-”打头，例如：

```
# A list of tasty fruits
- Apple
- Orange
- Strawberry
- Mango
```

## dictionary

字典通过key与value进行标识，例如：

```
# An employee record
name: Example Developer
job: Developer
skill: Elite
```

也可以将key:value放置于{}中进行表示，例如：

```
# An employee record
{name: Example Developer, job: Developer, skill: Elite}
```

Ansible中使用的YAML基础元素

包括:正则,变量,Inventory,条件测试,迭代

## 正则参数

(1)全量匹配 all与*功能相同，但*需引起来。

```
ansible all -m ping
ansible "*" -m ping
```

(2)逻辑或(or)匹配

多台主机或多个组同时执行

```
ansible "web1:web2" -m ping
```

(3)逻辑非(!)匹配

所有在web1组，但不在web2组的主机

```
web1:!web2
```

(4)逻辑与(&)匹配

web1和web2中同时存在的主机

```
web1:&web2
```

(5)模糊匹配

```
# 检查192.168.1.0/24网段所有主机存活状态。
ansible 192.168.1.* -m ping
# test开头的所有组
ansible "test*" -m ping
```

(6)域切割，同python字符串域切割

例：

```
[webservers]
web1.example.com
web2.example.com
web3.example.com

webservers[0]    #==web1.example.com
webservers[-1]    #==web3.example.com
webservers[0:2]    #第一位到第三位==web1.example.com、web2.example.com、web3.example.com
webservers[1:]    #第二位到最后==web2.example.com、web3.example.com
```

(7)正则匹配，"~"开始表示正则匹配

```
ansible "~(web|data|test)\.example\.(com|org)" -m ping
```

## 变量

### 变量命名

变量名仅能由字母、数字和下划线组成，且只能以字母开头

#### facts

facts是由正在通信的远程目标主机发回的信息，这些信息被保存在ansible变量中。要获取指定的远程主机所支持的所有facts，可使用如下命令进行：

```
ansible hostname -m setup
```

所有的facts信息都可以当做变量来调用

#### register

把任务的输出定义为变量，然后用于其他任务，示例如下:

```
  tasks:
    - shell: /usr/bin/foo
      register: foo_result
      ignore_errors: True
```

通过命令行传递变量

在运行playbook的时候也可以传递一些变量供playbook使用，示例如下：

ansible-playbook test.yml --extra-vars "hosts=www user=mageedu"

通过roles传递变量

当给一个主机应用角色的时候可以传递变量，然后在角色内使用这些变量，示例如下：

```
    - hosts: webservers
      roles:
        - common
        - { role: foo_app_instance, dir: '/web/htdocs/a.com',  port: 8080 }
```

变量读取的四个位置

Inventory配置

Playbook中vars定义的区域

Roles中vars目录下的文件

Roles同级目录group_vars和hosts_vars目录下的文件

\#设置变量时尽量沿用同一种方式。

Inventory

ansible的主要功用在于批量主机操作，为了便捷地使用其中的部分主机，可以在inventory file中将其分组命名。默认的inventory file为/etc/ansible/hosts

ansible通过Inventory来定义主机和组，使用时可通过 -i HOSTS_NAME 来指定读取，默认/etc/ansible/hosts，可存在多个Inventory，支持动态生成.

inventory文件格式

inventory文件遵循INI文件风格，中括号中的字符为组名.

可以将同一个主机同时归并到多个不同的组中；此外，当如若目标主机使用了非默认的SSH端口，还可以在主机名称之后使用冒号加端口号来标明.

示例:定义被管控的节点主机

vim /etc/ansible/hosts

```
# Ex 1: Ungrouped hosts, specify before any group headers.
# 定义未分组的主机,如下：
192.168.100.8    # 可直接为ip
www.node3.com    # 可直接为域名
www.node4.com:22 # 可直接接ssh端口

# Ex 2: A collection of hosts belonging to the 'webservers' group
# 把几个主机分组，用中括号定义组名，后跟组成员
[hbhosts] # 组名
# 可定义连接密码
192.168.100.8 ansible_ssh_pass=123.com

# 可定义连接的类型，连接的用户和密码
192.168.100.9 ansible_connection=ssh ansible_ssh_user=fedora ansible_ssh_pass=123.com   

# If you have multiple hosts following a pattern you can specify
# 对主机进行通配，[1：99]表示到一个范围
www.node[1:10].com       # 表示1-10所有的数字
www.node[a:z].com        # 表示a-z所有的字母

# Ex 3：定义主机变量：
# 定义的变量和在playbook中使用，在playbook中设定的[同名变量]会优先于此处的变量设定
other1.example.com    ansible_connection=ssh    ansible_ssh_user=mpdehaan    # 选择连接类型和连接用户
other2.example.com    http_port=8800            maxRequestsPerChild =999     # 定义http_port端口号8800, 每个子进程可接收的请求量

# Ex 4：定义组变量:
# 组变量是指赋予给指定组内所有主机上的在playbook中可用的变量
[test]
web1.example.com
web2.example.com

[test:vars]    # 组名:变量,固定格式,下面定义的变量适用于test组内的所有主机调用
ntp_server=ntp.example.com
proxy=proxy.example.com

# Ex 5：组嵌套,把一个组作为另一个组的子成员
# inventory中，组还可以包含其它的组，并且也可以向组中的主机指定变量.不过，这些变量只能在ansible-playbook中使用，而ansible不支持
[apache]
web1.example.com

[nginx]
web2.example.com
[webserver]
other1.example.com

[webserver:children] # 定义子组
apache  # 组名
nginx   # 组名
#上例中webserver包括web1.example.com、web2.example.com、other1.example.com

[webserver:vars] # 定义组变量,可让子组内成员调用
ntp_server=ntp.magedu.com
```

ansible基于ssh连接inventory中指定的远程主机时,还可以通过参数指定其交互方式:

ansible_ssh_host

\# 将要连接的远程主机名.与你想要设定的主机的别名不同的话,可通过此变量设置.

ansible_ssh_port

\# ssh端口号.如果不是默认的端口号,通过此变量设置.

ansible_ssh_user

\# 默认的 ssh 用户名

ansible_ssh_pass

\# ssh 密码(这种方式并不安全,我们强烈建议使用 --ask-pass 或 SSH 密钥)

ansible_sudo_pass

\# sudo 密码(这种方式并不安全,我们强烈建议使用 --ask-sudo-pass)

ansible_sudo_exe (new in version 1.8)

\# sudo 命令路径(适用于1.8及以上版本)

ansible_connection

\# 与主机的连接类型.比如:local, ssh 或者 paramiko. Ansible 1.2 以前默认使用 paramiko.1.2 以后默认使用 'smart','smart' 方式会根据是否支持 ControlPersist, 来判断'ssh' 方式是否可行.

ansible_ssh_private_key_file

\# ssh 使用的私钥文件.适用于有多个密钥,而你不想使用 SSH 代理的情况.

ansible_shell_type

\# 目标系统的shell类型.默认情况下,命令的执行使用 'sh' 语法,可设置为 'csh' 或 'fish'.

ansible_python_interpreter

\# 目标主机的 python 路径.

适用于的情况: 

--系统中有多个 Python, 或者命令路径不是"/usr/bin/python",比如  \*BSD, 或者 /usr/bin/python 

--不是 2.X 版本的 Python.我们不使用 "/usr/bin/env" 机制,因为这要求远程用户的路径设置正确,且要求 "python" 可执行程序名不可为 python以外的名字(实际有可能名为python26).

条件测试

如果需要根据变量、facts或此前任务的执行结果来做为某task执行与否的前提时要用到条件测试.

when语句

在task后添加when子句即可使用条件测试；when语句支持Jinja2表达式语法.

```
tasks:
  - name: "shutdown Debian flavored systems"
    command: /sbin/shutdown -h now
    when: ansible_os_family == "Debian"
```

when语句中还可以使用Jinja2的大多“filter”，例如要忽略此前某语句的错误并基于其结果（failed或者sucess）运行后面指定的语句，可使用类似如下形式:

```
tasks:
  - command: /bin/false
    register: result
    ignore_errors: True
  - command: /bin/something
    when: result|failed
  - command: /bin/something_else
    when: result|success
  - command: /bin/still/something_else
    when: result|skipped
```

此外，when语句中还可以使用facts或playbook中定义的变量

迭代

当有需要重复性执行的任务时，可以使用迭代机制。其使用格式为将需要迭代的内容定义为item变量引用，并通过with_items语句来指明迭代的元素列表即可

```
- name: add several users
  user: name={{ item }} state=present groups=wheel
  with_items:
     - testuser1
     - testuser2
```

上面语句的功能等同于下面的语句：

```
- name: add user testuser1
  user: name=testuser1 state=present groups=wheel
- name: add user testuser2
  user: name=testuser2 state=present groups=wheel
```

事实上，with_items中可以使用元素还可为hashes

with_items中的列表值也可以是字典,但是引用时要使用item.key方式

```
- name: add several users
  user: name={{ item.name }} state=present groups={{ item.groups }}
  with_items:
    - { name: 'testuser1', groups: 'wheel' }
    - { name: 'testuser2', groups: 'root' }
```

ansible的循环机制还有更多的高级功能，具体请参见官方文档[http://docs.ansible.com/playbooks_loops.html](http://docs.ansible.com/playbooks_loops.html）)

playbook

playbook是由一个或多个”play”组成的列表

play的主要功能在于将事先归为一组的主机，装扮成事先通过ansible中的task定义好的角色。从根本上来将，所谓的task无非是调用ansible的一个module。将多个paly组织在一个playbook中，即可以让他们联通起来按事先编排的机制同唱一台大戏.

一个完整的剧本可包含:

Tasks：任务，由模块定义的操作的列表,即调用模块完成的某操作

Variables：变量

Templates：模板，即使用了模板语法的文本文件

Handlers：处理器, 由特定条件触发的Tasks

Roles：角色；自包含，有完整独立实体

Hosts：指定要执行任务的主机组，可多个,冒号分割

remote_user：指定在远程主机上以哪个用户身份执行任务

sudo_user：非管理员需要拥有sudo权限

tasks：给出任务列表，执行完一个，执行第二个

剧本执行命令

ansible-playbook: 用来执行剧本的命令

  -i   指定inventory文件

  -v   详细输出，-vvv更详细，-vvvv更更详细

  -f   并发数

  -e   定义Playbook中使用的变量，格式"key=value,key=value"

  --remote-user   # 远程用户

  --ask-pass   # 远程用户密码

  --sudo   # 使用sudo,sudo: yes

  --sudo-user   # sudo的用户，默认root

  --ask-sudo-pass   # sudo密码

下面是一个简单示例:

以root的身份,安装htpd服务,忽略错误信息,安装完后启动

vim first.yaml

```
-hosts: webnodes
 remote_user: root
 vars:  # 定义2个变量
 - service_name: httpd
 - http_port: 80
 - max_clients: 256
 tasks:
 - task-name: ensure apache is at the lastest version.
   yum: name={{ service_name }} state=latest  # 变量引用使用花括号{{ vars }}
 - task-name: ensure apache is running
   service: name={{ service_name }} state=started
   ignore_errors: True   # 忽略错误信息
 handlers:   
 - handler-name: restart apache
   service: name={{ service_name }} state=restarted
```

vim second.yaml

```
- name: test        # 定义test组内的节点主机
  hosts: test       
  gather_facts: no    # 跳过收集facts信息，默认为yes
  vars:        # 定义变量，引用变量使用{{vars}}
    http: 8080
    https: 1443
  vars_files:    # 使用某个文件定义的变量，文件中也是用k/v形式定义
    - vars/server_vars.yml
  remote_user: root    # 定义以root身份控制节点
  sudo: yes        # 使用sudo
  tasks:        # 指定一系列将要运行的任务
    - name: test1        # test1任务
      copy: src=/opt/apps/web/* dest=/opt/apps/web/
      notify：
        - restart apache
      # notify与下面handlers对应，上一步copy之后文件发生改变，则通过name匹配handlers中的任务，这里匹配'restart appche'的service任务
      
    - name: test2    # test2任务
      yum: name={{ item }} state=present
      with_items:        # 迭代,定义列表变量，直接用{{ item }}引用,该关键字也可以用来定义数组
        - httpd
        - httpd-devel
        
    - name: test3
      shell: "echo $TEST"
      environment:
      - TEST: abc
      
  handlers:
    - name: restart apache
      service: name=httpd state=restarted
```

语法特性如下：

(1)"---"首行顶格开始

(2)#号注释

(3)缩进统一，不同的缩进代表不同的级别，缩进要对齐，空格和tab不能混用

(4)区别大小写，键值对k/v的值也大小写敏感

(5)k/v的值同行使用":"分开，换行写需以"-"分隔

Hosts和Users

playbook中的每一个play的目的都是为了让某个或某些主机以某个指定的用户身份执行任务

hosts用于指定要执行指定任务的主机，其可以是一个或多个由冒号分隔主机组

remote_user则用于指定远程主机上的执行任务的用户。如上面示例中的

-hosts: webnodes

 remote_user: root

不过，remote_user也可用于各task中。也可以通过指定其通过sudo的方式在远程主机上执行任务，其可用于play全局或某任务；此外，甚至可以在sudo时使用sudo_user指定sudo时切换的用户.

```
        - hosts: webnodes
          remote_user: mageedu
          tasks:
            - name: test connection
              ping:
              remote_user: mageedu
              sudo: yes
```

task任务列表和action

--play的主体部分是task list。task list中的各任务按次序逐个在hosts中指定的所有主机上执行，即在所有主机上完成第一个任务后再开始第二个。在运行自下而下某playbook时，如果中途发生错误，所有已执行任务都将回滚，因此，在更正playbook后重新执行一次即可。

--task的目的是使用指定的参数执行模块，而在模块参数中可以使用变量。模块执行是幂等的，这意味着多次执行是安全的，因为其结果均一致。

--每个task都应该有其name，用于playbook的执行结果输出，建议其内容尽可能清晰地描述任务执行步骤。如果未提供name，则action的结果将用于输出。

--定义task的可以使用“action: module options”或“module: options”的格式，推荐使用后者以实现向后兼容。如果action一行的内容过多，也中使用在行首使用几个空白字符进行换行。

```
    tasks:
      - name: make sure apache is running
        service: name=httpd state=running
```

在众多模块中，只有command和shell模块仅需要给定一个列表而无需使用“key=value”格式，例如：

```
    tasks:
      - name: disable selinux
        command: /sbin/setenforce 0
```

如果命令或脚本的退出码不为零，可以使用如下方式替代：

```
    tasks:
      - name: run this command and ignore the result
        shell: /usr/bin/somecommand || /bin/true 
```

或者使用ignore_errors来忽略错误信息：

```
    tasks:
      - name: run this command and ignore the result
        shell: /usr/bin/somecommand
        ignore_errors: True
```

handlers

用于当关注的资源发生变化时采取一定的操作.

“notify”这个action可用于在每个play的最后被触发，这样可以避免多次有改变发生时每次都执行指定的操作，取而代之，仅在所有的变化发生完成后一次性地执行指定操作。在notify中列出的操作称为handler，也即notify中调用handler中定义的操作。

```
    - name: template configuration file
      template: src=template.j2 dest=/etc/foo.conf
      notify:
        - restart memcached
        - restart apache 
```

handler是task列表，这些task与前述的task并没有本质上的不同。

```
    handlers:
        - name: restart memcached
          service:  name=memcached state=restarted
        - name: restart apache
          service: name=apache state=restarted
```

示例1:利用palybook让2台host安装httpd服务并启动

```
vim httpd.yaml
- hosts: hbhosts
  remote_user: root
  tasks:
  - name: install httpd package
    yum: name=httpd state=latest
    tags: installpkg# 指明跑某个标签 -t installpkg
  - name: install conf file
    copy: src=/root/httpd.conf dest=/etc/httpd/conf/httpd.conf
    tags: installconf # 指明跑某个标签 -t installconf
    notify: restart httpd service  # 通知给《===httpd
  - name: start httpd service
    service: name=httpd state=started
  handlers: # 处理器，也是任务，但是在一定条件下触发
  - name: restart httpd service //handlers名字
    service: name=httpd state=restarted
# 语法检查，测试运行
annsible-playbook --syntax-check  -t installconf  --list-tags httpd.yaml
annsible-playbook --syntax-check httpd.yaml
annsible-playbook --list-hosts --list-tasks httpd.yaml
```

注意： 

handlers：是指在一定条件下触发，指明在一个需要让别人重启服务才生效的任务上使用notify，通知一定是handlers名字 

tags：只执行playbook中指定的tags标签，满足部分需要，多个任务可指明一个同tags 

也可以一次调用两个标签，如下 

annsible-playbook -t installconf，installpkg httpd.yaml

示例2:利用playbook指挥2组主机创建hb组

vim group.yaml 

```
- hosts: hbhosts                    # 任务运行在hbhosts组的主机
  remote_user: root                 # 在远程主机以哪个用户执行
  tasks:                            # 任务列表
  - name: install a group           # 任务名1
    group: name=hb system=true      # 调用group模块创建组hb
  - name: install a user            # 任务名2
    user: name=user1 group=hb system=true  # 创建user1 组hb
    
- hosts: hbhosts                  # 任务运行在hbhosts组内的主机
  remote_user: root # 在远程主机以哪个用户执行
  tasks: # 任务列表
   - name: install httpd package         # 任务名1
     yum: name=httpd                     # 调用yum模块安装httpd
   - name: start httpd service           # 任务名2
     service: name=httpd state=started   # 调用service模块启动服务

# 检查语法，测试运行
```

案例：

vim heartbeat.yaml

```
- hosts: hbhosts
  remote_user: root
  tasks:
    - name: ensure heartbeat latest version
      yum: name=heartbeat state=present
    - name: authkeys configure file
      copy: src=/root/hb_conf/authkeys dest=/etc/ha.d/authkeys
    - name: authkeys mode 600
      file: path=/etc/ha.d/authkeys mode=600
      notify:
        - restart heartbeat
    - name: ha.cf configure file
      copy: src=/root/hb_conf/ha.cf dest=/etc/ha.d/ha.cf
      notify: 
        - restart heartbeat
  handlers:
    - name: restart heartbeat
      service: name=heartbeat state=restarted
```

roles

ansilbe自1.2版本引入的新特性，用于层次性、结构化地组织playbook

roles能够根据层次型结构自动装载变量文件、tasks以及handlers等。要使用roles只需要在playbook中使用include指令即可。简单来讲，roles就是通过分别将变量、文件、任务、模板及处理器放置于单独的目录中，并可以便捷地include它们的一种机制。角色一般用于基于主机构建服务的场景中，但也可以是用于构建守护进程等场景中。

一个roles的案例如下所示：

目录结构:

|-site.yml

|-webservers.yml

|-dbservers.yml

|-roles/

  |-common/

​    |-files/

​    |-templates/

​    |-tasks/

​    |-handlers/

​    |-vars/

​    |-meta/

  |-webservers/

​    |-files/

​    |-templates/

​    |-tasks/

​    |-handlers/

​    |-vars/

​    |-meta/

创建role的步骤:

(1) 创建以roles命名的目录；

(2) 在roles目录中分别创建以各角色名称命名的目录，如webservers等；

(3) 在每个角色命名的目录中分别创建files、handlers、meta、tasks、templates和vars目录；用不到的目录可以创建为空目录，也可以不创建；

(4) 在playbook文件中，调用各角色；

role内各目录中可用的文件

roles是以特定的层级目录结构进行组织的tasks、variables、handlers、templates… 

tasks目录：至少应该包含一个名为main.yml的文件，其定义了此角色的任务列表；此文件可以使用include包含其它的位于此目录中的task文件,其它的文件需要由main.yml进行“包含”调用；

files目录：存放由copy或script等模块调用的静态文件；

templates目录：template模块会自动在此目录中寻找Jinja2模板文件；

handlers目录：此目录中应当包含一个main.yml文件，用于定义此角色用到的各handler；在handler中使用include包含的其它的handler文件也应该位于此目录中；

vars目录：应当包含一个main.yml文件，用于定义此角色用到的变量,定义各variable；其它的文件需要由main.yml进行“包含”调用,注意字典格式;

meta目录：应当包含一个main.yml文件，用于定义此角色的特殊设定及其依赖关系；ansible 1.3及其以后的版本才支持；

default目录：为当前角色设定默认变量时使用此目录；应当包含一个main.yml文件；

site.yml：roles目录同级目录下,site.yml为调用各角色,额外也可以有其他.yml格式的文件

示例：

在webserver组内机器分别各自安装nginx，memcached，msql-server通过roles调用tasks 

创建目录:

```
mkdir -pv ./{nginx,memcached,httpd,mysql}/{files,templates,vars,handlers,meta,tasks,default}
```

而在playbook中，可以这样使用roles：

```
    - hosts: webservers
      roles:
        - common
        - webservers
```

也可以向roles传递参数，例如：

```
    - hosts: webservers
      roles:
        - common
        - { role: foo_app_instance, dir: '/opt/a',  port: 5000 }
        - { role: foo_app_instance, dir: '/opt/b',  port: 5001 }
```

甚至也可以条件式地使用roles，例如：

```
    - hosts: webservers
      roles:
        - { role: some_role, when: "ansible_os_family == 'RedHat'" }
```

示例2,有3台主机,a主机是websrvs,b主机是phpsrvs,c主机是dbsrvs,

```
# 定义roles目录结构
mkdir -pv ansible_playbooks//roles/{websrvs,dbsrvs}/{tasks,files,templates,meta,handlers,vars}
tree ansible_playbooks/
"""
ansible_playbooks/
└── roles
    ├── dbsrvs
    │   ├── files
    │   ├── handlers
    │   ├── meta
    │   ├── tasks
    │   ├── templates
    │   └── vars
    └── websrvs
        ├── files
        ├── handlers
        ├── meta
        ├── tasks
        ├── templates
        └── vars
"""

# 定义websrvs组的任务
cd ansible_playbooks/roles/websrvs
cp /etc/httpd/conf/httpd.conf files/

vim tasks/main.yml
"""
# 仅定义所有任务
- name: install httpd package
  yum: name=httpd
- name: install configuration file
  copy: src=httpd.conf dest=/etc/httpd/conf/httpd.conf
  tags:
  - httpconf
  notify:
  - restart httpd  # 会调用handlers/main.yml文件中定义对应名字的handlers
- name: start httpd
  service: name=httpd state=started
"""

vim handlers/main.yml
"""
# 定义所有的handlers
- name: restart httpd
  service: name=httpd state=restarted
"""

# 定义dbsrvs主机组的任务
cd ansible_playbooks/roles/dbsrvs
cp /etc/my.cnf files/

vim tasks/main.yml
"""
# 仅定义所有任务
- name: install mysql-server package
  yum: name=mysql-server state=latest
- name: install configuration file
  copy: src=my.cnf dest=/etc/my.cnf
  tags:
  - myconf
  notify:
  - restart mysqld  # 会调用handlers/main.yml文件中定义对应名字的handlers
- name: start mysqld service
  service: name=mysqld enabled=true state=started
"""

vim handlers/main.yml
"""
# 定义所有的handlers
- name: restart mysqld
  service: name=mysqld state=restarted
"""

# 调用角色
cd ansible_playbooks/
vim site.yml
"""
- hosts: 172.16.100.10  # 应用在哪些主机组上
  remote_user: root
  roles:
  - websrvs_roles  # 为目标主机组调用角色,角色名为websrvs_roles
  
- hosts: 172.16.100.11  # 应用在哪些主机组上
  remote_user: root
  roles:
  - dbsrvs_roles  # 为目标主机组调用角色,角色名为websrvs_roles
  
- hosts: 172.16.100.12  # 应用在哪些主机组上
  remote_user: root
  roles:
  - websrvs_roles  # 为目标主机组调用角色,角色名为websrvs_roles
  - dbsrvs_roles
"""

ansible-playbook ansible_playbooks/site.yml
```

Tags

tags用于让用户选择运行playbook中的部分代码。ansible具有幂等性，因此会自动跳过没有变化的部分，即便如此，有些代码为测试其确实没有发生变化的时间依然会非常地长。此时，如果确信其没有变化，就可以通过tags跳过此些代码片断。

扩展：

例：

```
# 把shell命令委派给192.168.2.30节点执行，其他hosts中指定的节点不执行。
- name: 123
        shell: "echo $PATH>/test/1"
        delegate_to: 192.168.2.30
```

register：注册变量

将操作结果，包括stdout和stderr，保存到变量中，再根据变量的内容决定下一步，这个保存操作结果的变量就是注册变量。

例：

```
- shell: ***
        register: result
# 使用result.stdout和result.stderr读取执行结果的标准输出和标准错误。
```

(5)vars_prompt: 交互式提示

例：

```
---
- hosts: all
  vars_prompt:
  - name: share_user
    prompt: "username?"
  - name: share_pass
    prompt: "password"
    private: yes
```

常用选项：

private: yes   #用户输入不可见

default   #设置默认值

confirm: yes   #要求输入两次

自定义变量Variables

可自定义安装任意程序包

```
cat name.yaml 
- hosts: webserver
  remote_user: root
  vars: 
  - pkgname: memcached //playbook中使用的变量Variables
  tasks:
  - name: install a package
    yum: name={{ pakgname }} state=present //自定义变量
```

语法测试

ansible-playbook --syntax-check name.yaml 

ansible-playbook -e pkgname=httpd/vsftpd/ name.yaml 

注意：在执行时命令行中给出的值要优先于playbook内置变量

如：ansible-playbook -e pkgname=httpd name.yaml

检查：rpm -ql httpd memcached

传递给主机单独变量，实现某些主机安装不同程序包

编辑主机清单在每个主机后面指定调用变量并赋值

```
vim /etc/ansible/hosts
[webserver]
172.16.5.102 pkgname=nginx
172.16.5.103 pkgname=httpd
[webserver:vars] //为一个组内不同主机定义相同变量，效果同上
webserver组内有一组变量vars，其中变量名是pkgname值是memcached
pkgname=memcached
vim name.yaml
    - hosts: webserver
      remote_user: root
      tasks:
      - name: install a package
        yum: name={{ pkgname }} state=present
```

语法测试

ansible-playbook --syntax-check name.yaml 

Inventory还可以使用参数： 

用于定义ansible远程连接目标主机使用的属性，而非传递给playbook的变量 

如： [webserver] 

172.16.5.102 ansible_ssh_user= ansible_ssh_pass ansible_sudo_pass 

不再是传递给playbook变量，而是主机本身。

模版template

模版本身是文本文件，内部嵌套有模板语言脚本（使用模板语言编写jinjia2）

简单示例:为不同的主机配置不同内容的配置文件

```
# 创建模板文件存放目录
mkdir /root/templates
cp /etc/httpd/conf/httpd.conf /root/templates/httpd.conf.j2

# vim /etc/ansible/hosts
[websrvs]
172.16.100.10 http_port=80 maxClients=110
172.16.100.11 http_port=8080 maxClients=210

[dbsrvs]
172.16.100.12

[websrvs:vars]
ansible_ssh_port=2201

[dbsrvs:vars]
ansible_ssh_port=2201

# vim /root/templates/httpd.conf.j2
Listen {{ http_port }}
MaxClients {{ maxClients }}
ServerName {{ ansible_fqdn }}  # facts信息的key都可作为变量来调用

# ansible 172.16.100.10 -m setup | grep "ansible_fqdn" 可查看ansible_fqdn变量值

# vim apache.yml
- hosts: websrvs
  remote_user: root
  vars:
  - package_name: httpd
  - service_name: httpd
  tasks:
  - name: install httpd package
    yum: name={{ package_name }} state=latest
  - name: install configuration file for httpd
    template: src=/root/templates/httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf
    notify:
    - restart httpd
  - name: start httpd service
    service: enabled=true name={{ service_name }} state=started
  handlers:
  - name: restart httpd
    service: name={{ service_name }} state=restarted
```

Jinja2 是仿照 Django 模板的 Python 模板语言。 它速度快，被广泛使用，并且提供了可选的沙箱模板执行环境保证安全。只介绍简单使用，不过多阐述

jinjia2是用来写模版文件的，接下来的模版文件都是以j2为后缀

语法格式：

​    支持字面量：

​    字符串：使用单引号或双引号； 

​    数字：整数、浮点数；

​    列表：[item1, item2, ...]

​    元组：(item1, item2, ...)

​    字典：{key1:value1, key2:value2, ...}

​    布尔型：true/false        

​    算术运算：

​      +, -, *, /, //, %, **

​    比较操作：

​      ==, !=, >, <, >=, <=   

​    逻辑运算：and, or, not 

基于模版把nginx配置文件对应的虚拟cpu值改成不是auto

Jinja2相关

字面量

表达式最简单的形式就是字面量。字面量表示诸如字符串和数值的 Python 对象。下面 的字面量是可用的:

“Hello World”:

双引号或单引号中间的一切都是字符串。无论何时你需要在模板中使用一个字 符串（比如函数调用、过滤器或只是包含或继承一个模板的参数），它们都是 有用的。

42 / 42.23:

直接写下数值就可以创建整数和浮点数。如果有小数点，则为浮点数，否则为 整数。记住在 Python 里， 42 和 42.0 是不一样的。

[‘list’, ‘of’, ‘objects’]:

一对中括号括起来的东西是一个列表。列表用于存储和迭代序列化的数据。例如 你可以容易地在 for 循环中用列表和元组创建一个链接的列表:

```
<ul>
{% for href, caption in [('index.html', 'Index'), ('about.html', 'About'),
                        ('downloads.html', 'Downloads')] %}
    <li><a href="{{ href }}">{{ caption }}</a></li>
{% endfor %}
</ul>
```

(‘tuple’, ‘of’, ‘values’):

元组与列表类似，只是你不能修改元组。如果元组中只有一个项，你需要以逗号 结尾它。元组通常用于表示两个或更多元素的项。更多细节见上面的例子。

{‘dict’: ‘of’, ‘key’: ‘and’, ‘value’: ‘pairs’}:

Python 中的字典是一种关联键和值的结构。键必须是唯一的，并且键必须只有一个 值。字典在模板中很少使用，罕用于诸如 xmlattr() 过滤器之类。

true / false:

true 永远是 true ，而 false 始终是 false.

算术运算

Jinja 允许你用计算值。这在模板中很少用到，但是为了完整性允许其存在。支持下面的 运算符:

\+ :把两个对象加到一起。通常对象是素质，但是如果两者是字符串或列表，你可以用这 种方式来衔接它们。无论如何这不是首选的连接字符串的方式！连接字符串见 ~ 运算符。 {{ 1 + 1 }} 等于 2 

\- :用第一个数减去第二个数。 {{ 3 - 2 }} 等于 1 

/ :对两个数做除法。返回值会是一个浮点数。 {{ 1 / 2 }} 等于 {{ 0.5 }} 

// :对两个数做除法，返回整数商。 {{ 20 // 7 }} 等于 2 

% :计算整数除法的余数。 {{ 11 % 7 }} 等于 4 

\* :用右边的数乘左边的操作数。 {{ 2 * 2 }} 会返回 4 .也可以用于重 复一个字符串多次。 {{ ‘=’ * 80 }} 会打印 80 个等号的横条

** :取左操作数的右操作数次幂。 {{ 2**3 }} 会返回 8 

比较操作符

== :比较两个对象是否相等

!= :比较两个对象是否不等

\> :如果左边大于右边，返回 true 

\>= :如果左边大于等于右边，返回 true 

< :如果左边小于右边，返回 true 

<= :如果左边小于等于右边，返回 true 

逻辑运算符

对于 if 语句，在 for 过滤或 if 表达式中，它可以用于联合多个表达式:

and :如果左操作数和右操作数同为真，返回 true

or :如果左操作数和右操作数有一个为真，返回 true

not :对一个表达式取反（见下）

(expr)

  表达式组。

实例演示:

安装nginx程序包，我们要使用nginx的配置文件当模版 

```
yum install nginx 
cp /etc/nginx/nginx.conf . 
vim nginx.conf 
worker_processes {{ ansible_processor_vcpus }}; //由auto改成这样 
vim nginx.yaml 
– hosts: webserver 
remote_user: root 
tasks: 
– name: copy a nginx file 
template: src=/root/nginx.conf dest=/tmp/nginx.conf

# 语法测试 
ansible-playbook –syntax-check nginx.yaml
```

完整安装nginx程序，并调用nginx.conf.j2模版

```
vim nginx.yaml 
– hosts: nginxserver # 指定组 
remote_user: root 
tasks: 
– name: install nginx package # 安装nginx程序包 
yum: name=nginx state=latest # 调用的yum模块 
– name: install conf file 
template: src=/root/nginx.conf.j2 dest=/etc/nginx.conf # 调用的配置文件模版 
tags: ngxconf # 标签 
notfiy: reload nginx service # 通知让标签干什么 
– name: start nginx service 
service: name=nginx state=started enabled=true # 启动服务，并开机启动 
handlers: # 触发器，当某个条件满足时将触发，也是任务 
– name: reload nginx service 
shell: /usr/sbin/nginx -s reload 
# 语法测试 
ansible-playbook –syntax-check nginx.yaml
```

使用when条件判断语句，判断基于不同发行版使用不同命令来重启nginx服务

```
vim nginx2.yaml
- hosts: all
  remote_user: root
  tasks:
- name: install nginx package
  yum: name=nginx state=latest
- name: start nginx service on CentOS6
  shell: service nginx start
  when: ansible_distribution == "CentOS" and ansible_distribution_major_version == "6"
- name: start nginx service
  shell: systemctl start nginx.service
  when: ansible_distribution == "CentOS" and ansible_distribution_major_version == "7"
# 语法测试
ansible-playbook --syntax-check nginx2.yaml 
```

基于字符串列表给出元素循环：迭代，需要重复执行的任务； 

对迭代项的引用，固定变量名为”item”，使用with_item属性给定要迭代的元素； 

元素列表有两种：字符串，字典

```
vim web.yaml
- hosts: webserver
  remote_user: root
  tasks:
  - name: install package
    yum: name={{ item }} state=latest
    with_items:
    - httpd
    - php
    - php-mysql
    - php-mbstring
    - php-gd
# 语法测试
ansible-playbook --syntax-check web.yaml
```

基于字典列表给出元素循环：迭代，需要重复执行的任务； 

之前我们讲过我们要批量在组内的机器上创建用户是可以的。无非调用user模块并指明组就可以，但是在不同组内的机器上创建的，可能用户名与组名都是不相同的，那我们想要它都同样该怎么办呢？ 

这个时候我们就可以通过固定变量item来实现 增加两条name，一个指明用户，一个指明组名

```
vim creat.yaml
- hosts: webserver
  remote_user: root
  tasks:
  - name: creat groups # 第一个任务
  yum: name={{ item }} state=latest
  with_items:
  - groupx1
  - groupx2
  - groupx3
  - name: creat users # 第二个任务
user: name={{ item.name }} group={{ item.group }} state=present
  with_items: # 元素列表
  - ｛name: 'userx1', group: 'groupx1'｝ # name键值队
  - ｛name: 'userx1', group: 'groupx1'｝ # name键值队
  - ｛name: 'userx1', group: 'groupx1'｝ # name键值队
# 语法测试：
ansible-playbook --check creat.yaml
```

注释：

item.name表示调用第一个键name里的值userx1

item.group表示调用第一个键group里的值groupx1

角色(自包含，按照目录结构来组织)

指定由哪些主机去对应的完成哪个已定义好的角色

安装nginx

1,在nginx目录tasks目录下创建main.yml文件

```
vim main.yml 
    - name: copy nginx package to remote host //先拷贝安装程序
      copy: src=nginx-1.10.1-1.el7.x86_64.rpm 
            dest=/tmp/nginx-nginx-1.10.1-1.el7.x86_64.rpm
      tags: cppkg
    - name: install nginx package //再安装程序包
      yum: name=/tmp/nginx-1.10.0-1.el7.nginx.x86_64.rpm  state=present
    - name: install conf file nginx.conf //然后提供配置文件
      template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
      tags: nginxconf //加了一个标签，按需执行
      notify: reload nginx service //通知标签要干什么，我们需要在handlers目录下创建触发器
    - name: install conf file default.conf //再提供nginx默认文件
      template: src=default.conf.j2 dest=/etc/nginx/conf.d/default.conf
      tags: nginxconf
      notify: reload nginx service 
    - name: start nginix service //启动服务
      service : name=nginx enabled=true state=started
```

2,在nginx目录handlers目录下创建main.yml文件

```
cat main.yml 
        - name: reload nginx service 
          service: name=nginx state=restarted
```

3,拷贝nginx.conf文件到templates目录下做模版

```
cp /etc/nginx/nginx.conf nginx/templates/nginx.conf.j2
        # 更改 
        worker_processes {{ ansible_processor_vcps }}; 
```

4,拷贝default.conf文件到templates目录下做模版

cp /etc/nginx/conf.d/default.conf nginx/default/default.conf.j2

更改   listen    {{ nginxport }};  //变量定义

5,nginx/vars目录下创建main.yml 调用变量

​     写入 nginxport: "8090"

6,在ansible目录下创建nginx.yml文件来调用以上各种组织好的nginx角色

```
vim nginx.yml
                - hosts: webserver
                  remote_user: root
                  roles: # 定义角色
                  - ｛ role: nginx, ngxport: 8080 # 即能调角色，又能向角色传变量
                  - nginx # 角色调用
```

7,编辑ansible.cfg文件启用roles_path让roles去找所定义的文件

vim ansible.cfg 

启用把前面注释去掉 roles_path   = /etc/ansible/roles

8,要注意，避免错误，最好定义好文件后各自做个语法测试

9,然后就可以让剧本跑一遍了

10,确保你的系统selinux是disabled，不然服务起不来

ansible-playbook --syntax-check nginx.yml

安装memcached

```
ls ../memcached/*
../memcached/handlers:
main.yml
- name: reload memcached
  service: name=memcached state=restarted
../memcached/tasks:
main.yml
- name: install memcached
  yum: name=memcached state=latest
- name: install conf file
  template: src=memcached.j2 dest=/etc/sysconfig/memcached
  tags: mcconf
  notify: reload memcached
- name: start memcached service 
  service: name=memcached state=started enabled=true
../memcached/templates:
memcached.j2
    PORT="11211"
    USER="memcached"
    MAXCONN="1024"
    CACHESIZE="{{ ansible_memtotal_mb //4 }}"
    OPTIONS=""
```

安装mysql

```
vim mysql/tasks/main.yml 
mysql/tasks:
main.yml
- name: install mysql-server
  yum: name=mysql-server state=latest
  when: ansible_distribution == "Centos" and ansible_distribution_major_version == "6"
- name: install mariadb-server
  yum: name=mariadb-server state=latest
  when: ansible_distribution == "Centos" and ansible_distribution_major_version == "7"
- name: start mysql service
  service: name=mysqld state=started
  when: ansible_distribution == "Centos" and ansible_distribution_major_version == "6"
- name: start mariadb service
  service: name=mariadb state=started
  when: ansible_distribution == "Centos" and ansible_distribution_major_version == "7"
[root@Centos6 ansible]# vim mysql.yml 
    - hosts: webserver
      remote_user: root
      roles:
      - mysql
```

语法测试：

ansible-playbook --syntax-check  mysql.yml

done！！！

流程控制

(1)when条件判断

例1：第一步执行的返回值中有'ready'时，再执行下一步操作。

  \- command: my-app --status

   register: myapp_result

  \- command: do-something

   when: "'ready' in myapp_result"

例2：stat模块查看文件信息，如果文件没创建，则执行command

  \- stat: path=/etc/sysctl.conf

   register: stat_result

  \- command: touch /etc/sysctl.conf

   when: stat_result.stat.exists == false

(2)changed_when和failed_when

(3)wait_for模块，任务暂停

6、循环

原文：http://xdays.me/ansible状态管理.html

(1)标准遍历

用with_items可以遍历一个列表，注意这里只会遍历一层。示例如下：

\- name: add several users

 user: name={{ item }} state=present groups=wheel

 with_items:

   \- testuser1

   \- testuser2

(2)嵌套遍历

用with_nested可以遍历一个列表，注意这里会遍历多层，直到最内层。示例如下：

\- name: give users access to multiple databases

 mysql_user: name={{ item[0] }} priv={{ item[1] }}.*:ALL append_privs=yes password=foo

 with_nested:

  \- [ 'alice', 'bob', 'eve' ]

  \- [ 'clientdb', 'employeedb', 'providerdb' ]

(3)遍历字典

用with_dict可以遍历一个字典，用key和value来表示。示例如下：

\#变量文件

\---

users:

 alice:

  name: Alice Appleworth

  telephone: 123-456-7890

 bob:

  name: Bob Bananarama

  telephone: 987-654-3210

\#playbook文件

tasks:

 \- name: Print phone records

  debug: msg="User {{ item.key }} is {{ item.value.name }} ({{ item.value.telephone }})"

  with_dict: users

(4)文件通配符循环

用with_fileglob可以获取本地文件列表。示例如下：

 \# copy each file over that matches the given pattern

 \- copy: src={{ item }} dest=/etc/fooapp/ owner=root mode=600

  with_fileglob:

   \- /playbooks/files/fooapp/*

(5)对齐的列表

用with_together可以达到类似Python里的zip函数的功能。示例如下：

\#变量文件：

\---

alpha: [ 'a', 'b', 'c', 'd' ]

numbers:  [ 1, 2, 3, 4 ]

\#playbook文件

tasks:

  \- debug: msg="{{ item.0 }} and {{ item.1 }}"

   with_together:

​    \- alpha

​    \- numbers

(6)循环一个task的输出

with_lines指令后跟一个命令，ansible会遍历命令的输出。示例如下：

\- name: Example of looping over a command result

 shell: /usr/bin/frobnicate {{ item }}

 with_lines: /usr/bin/frobnications_per_host --param {{ inventory_hostname }}

7,Tags标签

标签功能可以给Roles、文件、单独的任务甚至整个Playbook打上标签，然后利用这些标签来指定要运行或不运行Playbook中的个别任务.tags标签写在tasks任务的后面

例：

```
vim test.yml
---
- hosts: test
  tags: deploy    #给整个Playbook打一个标签
  tasks:
    - name: test_tags
      shell: ***
      tags:    #为任务打标签
        - test1
        - test2
    - include: foo.yml
      tags: foo
```

ansible-playbook test.yml --tags "test1"   #只执行test_tags任务

ansible-playbook test.yml --skip-tags "test2"   #跳过test_tags任务

写法格式：

tags: ['one','two']

tags:

 \- one

 \- two

特殊tags: always

8、Block块

9、Include

```shell
报错：
[DEPRECATION WARNING]: docker is kept for backwards compatibility but usage is discouraged. The module documentation details page may explain more about this rationale..
This feature will be removed in
a future release. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
[ERROR]: unable to parse /usr/lib/python2.6/site-packages/ansible/modules/extras/cloud/misc/rhevm.py
ERROR! module rhevm has a documentation error formatting or is missing documentation

解决方法：
sed -i 's/^#deprecation_warnings = True/deprecation_warnings = False/' /etc/ansible/ansible.cfg
rm -f /usr/lib/python2.6/site-packages/ansible/modules/extras/cloud/misc/rhevm.py

报错:
The authenticity of host '[192.168.99.43]:22051 ([192.168.99.43]:22)' can't be established.
RSA key fingerprint is 5e:9d:5c:4c:e8:cd:6e:78:70:a2:04:1c:5f:6f:3a:1e.
Are you sure you want to continue connecting (yes/no)? The authenticity of host '[192.168.99.248]:22051 ([192.168.99.248]:22)' can't be established.
RSA key fingerprint is 64:d9:ef:67:6a:d5:37:ff:70:2f:06:d2:35:d1:6b:a2.
Are you sure you want to continue connecting (yes/no)? The authenticity of host '[192.168.99.247]:22051 ([192.168.99.247]:22)' can't be established.

解决方法：
sed -i 's/^#host_key_checking = False/host_key_checking = False/' /etc/ansible/ansible.cfg
```