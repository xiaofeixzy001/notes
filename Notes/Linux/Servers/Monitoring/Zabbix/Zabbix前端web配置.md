[TOC]

#  一，添加被监控主机



## 1.1 Host Groups/Hosts 配置

每个被监控的设备，都是一个 Host，那么将多个 Host 按照某种方式分组，这个分组就是 Host Groups。这里的分组方式有地理位置，业务单位，机器用途，系统版本等等。

建立一个 Host Groups，然后在其中建立一个 Host，这个 Host 就是刚才安装 Zabbix Agent 的设备在 Zabbix Server 上的概念设备。

在配置 Host 的时候，需要注意这里的 HostName 和被监控设备中定义的 HostName 保持一致，方便辨识。配置的 IP 就是被监控设备的 IP，端口号是 10050。

如图：

![image-20200114155600525](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/image-20200114155600525.png)

总结：

添加群组：配置 --> 主机群组 --> 创建主机群组 --> 填写组名 --> 添加

添加主机：配置 --> 主机 --> 创建主机 --> 填写主机信息 --> 添加

## 1.2 Items配置

配置完 Host 之后，就需要告诉 Zabbix 监控 Host 中的什么数据。这个要监控的数据就是监控项，也叫 Items。

Items 配置包括监控数据的方式，取值的数据类型，获取数值的间隔，历史数据保存时间，趋势数据保存时间，监控 Key 的分组等信息。

首选，需要选择 Type，它是要监听的 Zabbix 客户端的类型。一般来说，在安装 Zabbix Agent 以后，这个类型就是 Zabbix Agent。也可以选择 SNMP，IMPI 或者其他类型。

如图：

![image-20200114160247920](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/image-20200114160247920.png)

其次，需要注意 Key 的选择，Key 是来确定具体监控项的，对于同一个 Host 来说它是唯一的。

Zabbix 默认就带有一些 Key 可供选择，例如：vm.memory.size[total]，就是获取内存大小的 Key。

由于是针对 Host 进行配置的，所以也会指定对应的 Host IP 和 Port。另外，还有一些其他的数据需要配置。

例如：数据更新间隔，更新周期，历史数据保存天数，趋势数据保存天数等等。

如图：![image-20200114161528766](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/image-20200114161528766.png)

上图中还有一个 Applications 的选项。它实际上是对 Items 的一个集合，例如：要监控 MySQL，可以定义一个 MySQL 的 Application。

把相关的 Items 包括 availability of MySQL，disk space，processor load，transactions per second，number of slow queries 全部放到这个 Application 中方便选择和管理。

## 1.3 Trigger 配置

Items 是用来配置监控什么数据的，而不判断数据是否正常。那么，Trigger 的作用就是对采集的数据进行判断。


通常会设置判断规则或者阀值，一旦满足某种规则或者超过对应的阀值就会产生一个事件。

同时，Action 会对满足条件的 Trigger 执行操作。这些规则通过正则表达式来定义。

如图：

![image-20200114161628400](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/image-20200114161628400.png)

信息经过表达式判断，会产生两类 Trigger 状态，OK（正常）和 PROBLEM（异常）。

每个 Trigger 会对应一个 Items，每个 Items 会对应多个 Trigger。同时，Trigger 又可以设置不同的事件级别，可以根据这些级别设置多重告警。

如图：

![image-20200114161729480](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/image-20200114161729480.png)

在配置 Trigger 的时候主要是添加正则表达式。Zabbix 会根据对应 Item 的 Function 生成对应的正则表达式。

Trigger 会根据监控的内容（Item）来配置，例如：Item 是检测 Linux 的登陆人数。选择 Item 为“Template OS linux：Number of logged in users”。

对应的 Function 是 Last(most recent) T value is = N。意思是获取最近登陆的人数 T，当 T 等于 N 的时候触发 Trigger。

这个 N 就是需要我们配置的值，比如填写 2。也就是登陆人数等于 2 的时候触发 Trigger。

当你配置完毕以后就会生成类似如下图正则表达式了，{Template OS Linux:system.users.num.last()}>2。

整个过程不需要你输入表达式，只要通过选择和配置就可以完成。

如图：

![image-20200114161803367](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/image-20200114161803367.png)



上图有一个 Tab 项叫做 Dependencies，这个是 Trigger 的告警依赖，在实际场景中非常有用。

它会针对特殊场景使用，例如：整个 IDC 机房的路由出现故障，那么机房所有的机器的网络状态都会出现异常，此时 Zabbix Server 会收到大量异常报警。运维人员会被报警信息淹没，不知道故障的真正原因。

此时，就可以在 Dependencies 中选择对应规则，并且勾选 Multiple PROBLEM events generation 选项。

之后，就会收到一条报警信息，“某某 IDC 机房路由器 X 发生故障”，对其他的报警信息做了聚合操作。

触发器与表达式的编写方法

（1）avg

参数：秒或#num

支持类型：float,int

作用：返回一段时间的平均值

举例： 

avg(5)：最后5秒的平均值

avg(#5)：表示最近5次得到值的平均值 

avg（3600,86400）：表示一天前的一个小时的平均值，如果仅有一个参数，表示指定时间的平均值，从现在开始算起，如果有第二个参数，表示漂移，从第二个参数前开始算时间，#n表示最近n次的值

（2）last

参数：秒或#num

支持值类型：float，int，str，text，log

作用：最近的值，如果为秒，则忽略，#num表示最近第N个值，请注意当前的#num和其他一些函数的#num的意思是不同的。

last(0) 等价于last(#1) 

last(#3) 表示最近第3个值(并不是最近的三个值)

本函数也支持第二个参数time_shift，例如last(0, 86400)，返回一天前的最近的值。 

如果在history中同一秒中有多个值存在，Zabbix不保证值的精确顺序

#num，从Zabbix1.6.2起开始支持，timeshift，从1.8.2起开始支持，可以查询avg()函数获取它的使用方法。

（3）max

参数：秒或#num

支持值类型：float，int

描述：返回指定时间间隔的最大值。

时间间隔作为第一个参数可以是秒或收集值的数目（前缀为#）。从Zabbix1.8.2开始，函数支持第二个可选参数time_shift，可以查看avg()，函数获取它的使用方法。

例如：max(#3)=0 返回3次值如果都是0则触发告警。

（4）min

参数：秒或#num

支持值类型：float，int

描述：返回指定时间间隔的最小值。时间间隔作为第一个参数可以是秒或收集值的数目（前缀为#）。从Zabbix1.8.2开始，函数支持第二个可选参数time_shift，可以查看avg()函数获取它的使用方法。

（5）nodata

参数：秒

支持值类型：any

描述：当返回值为1表示指定的间隔（间隔不应小于30秒）没有接收到数据，0表示获取到了。

例：nodata(5m)=1 ===>5分钟之内获取不到数据就告警

（6）prev

参数：忽略

支持值类型：float，int，str，text，log

描述：返回之前的值，类似于last（#2）

（7）sum

参数：秒或#num

支持值类型：float，int

描述：返回指定时间间隔中收集到的值的总和，时间间隔作为第一个参数支持秒或收集值的数目（以#开始）.从Zabbix1.8.2开始，本函数支持time_shift作为第二个参数。可以查看avg()函数获取它的用法。

（8）change

参数：忽略

支持类型：float,int,str,text,log

作用：返回最近获得值与之前获得值的差值，对于字符串0表示相等，1表示不同change（0）>n:忽略参数一般输入0，表示最近得到的值与上一个值的差值大于n

（9）diff

参数：忽略

支持值类型：float，init，str，text，log

作用：返回值为1,表示最近的值与之前的值不同，0为相同。例如：diff（0）>0 ===>表示现在获取的值如果和之前的不同就告警

**例如：**

![QQ截图20180118093853.png-90.8kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.36509071336154064.png)

![QQ截图20180118094336.png-32kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.6786492656237724.png)



## 1.4 Action 配置

如果说 Trigger 定义触发事件的规则，那么 Action 就是事件触发后的动作。即当 Trigger 条件被满足以后，Action 会执行一些操作。

比如：发送事件通知（短信，钉钉，邮件），远程执行命令（重启服务）。

![image-20200114161921989](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/image-20200114161921989.png)

Zabbix 中有多种事件类型，Trigger 只是其中一种，例如：自动发现监控设备，自动注册监控设备等等。因此，先要选择事件的来源，当然这里我们选择 Trigger 作为来源。

![image-20200114161947323](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/image-20200114161947323.png)

然后，填写 Action 的基本信息。例如：名字，主题，默认发送的信息内容，异常恢复主题，以及对应的信息内容。这里的填写可以是字符串，但是更多的会使用宏。其实就是替换字符，比如{TRIGGER.STATUS} 表示触发器的状态，{ITEM.NAME} 表示监控项的名字。通过这些宏和字符串的拼接形成最终的信息。

如图：

![image-20200114162028863](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/image-20200114162028863.png)

接下来，就是对条件的配置（Condition）。由于 Action 可以面对一个或者多个 Trigger，每日每个 Trigger 又有一个或者多个条件。

为了保证其灵活性，可以针对 AND，OR，AND/OR，对条件进行组合。

例如：可以用“AND”条件，将 Maintenance status not in maintenance （机器不在维护状态）和 Trigger value = PROBLEM（触发器异常）两个条件（Condition）组合起来，意思是当两个条件同时满足时触发后续操作。

![image-20200114162242599](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/image-20200114162242599.png)

最后，就是操作（Operation）的配置。包括执行操作的时间间隔，执行次数，每次执行的间隔，操作类型（发送消息，执行命令），发送给哪些用户/用户组等等。

![image-20200114162333900](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/image-20200114162333900.png)



## 1.5 Template配置

如果有很多监控设备需要配置，是有工作量的。于是，Zabbix 会将有相同 Item，Trigger，Application 等规则项放到一起，于是就有了模版（Template）。

当对同类型的设备需要配置监控项时，就可以选择现成的模版。从而，减少运维工程师的工作量。在创建模版的时候，需要输入模版名字，以及对应的分组。

![image-20200114165423826](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/image-20200114165423826.png)



如果需要继承模版，可以在 Linked template 中进行配置。模版继承可以理解为模板嵌套。

例如，事先定义了一个基础模板，Item 配置好了 CPU、内存、硬盘、网卡等信息。

如果需要在这个基础模版上扩展其他模版，比如：MySQL 监控模版或者 Web 监控模板。那么在配置模板的时候，就可以继承于基础模板，而不需要重新定义模版。

模版创建完毕后，就可以添加 Item，Trigger，Application 等信息，具体的方式类似 Item，Trigger 配置。

## 总结

上面大段的文字讲述了 Zabbix 构建监控系统的过程。为了方便记忆，这里做个总结。

Zabbix 构建监控系统，先安装 Zabbix Agent 到 Host 收集信息，Zabbix Server 用来获取信息，Zabbix UI（Web）用来展示和配置信息。

Zabbix Agent 在 Host 中配置好监控服务器的 IP 和 Port 之后，回到 Zabbix Server 上，通过 Zabbix UI（Web）对要监控的 Host（被监控设备）进行配置。

依次配置：Item（监控什么数据），Trigger（故障触发条件），Action（故障触发动作）。



# 二，Zabbix常用模版与触发器功能详解

（1）`{Template App Zabbix Agent:agent.version.diff(0)}>0`

解释：如果当前获取的agent客户端的版本号大于前一次的不同，那么触发告警

（2）`{Template App Zabbix Agent:agent.ping.nodata(5m)}=1`

解释：如果ping客户端在5分钟内都没有数据，那么触发告警

（3）`{Template OS AIX:vm.memory.size[available].last(0)}<20M`

解释：如果最后一次获取的空闲内存大小得值小于20M，那么触发告警

（4）`{Template App SSH Service:net.tcp.service[ssh].max(#3)}=0`

解释：如果ssh远程连接连续获取的3次值的最大值都是0，那么触发告警

（5）`{Template ICMP Ping:icmppingloss.min(5m)}>20`

解释：如果连续5分钟里获取的最小值都大于20，那么触发告警

（6）`{Template ICMP Ping:icmppingsec.avg(5m)}>0.15`

解释：如果连续5分钟内的平均值大于0.15，那么触发告警

# 三，Zabbix报警媒介类型设置和告警动作、频率设置

## 3.1 QQ邮件告警平台

### 3.1.1 安装sendmail

```
 [root@localhost ~]# wget http://caspian.dotconf.net/menu/Software/SendEmail/sendEmail-v1.56.tar.gz [root@localhost ~]# yum -y install perl-Net-SSLeay perl-IO-Socket-SSL [root@localhost ~]# tar xf sendEmail-v1.56.tar.gz -C /usr/local/ [root@localhost ~]# cd /usr/local/sendEmail-v1.56/ [root@localhost sendEmail-v1.56]# /bin/cp -ra sendEmail /usr/local/bin/ [root@localhost sendEmail-v1.56]# chmod +x /usr/local/bin/sendEmail  [root@localhost sendEmail-v1.56]# which sendmail /usr/sbin/sendmail
```

### 3.1.2 sendmail命令使用说明

| 命令/参数                | 内容                                | 解释说明                             |
| ------------------------ | ----------------------------------- | ------------------------------------ |
| /usr/local/bin/sendEmail | 无                                  | 命令主程序                           |
| -f                       | [from@163.com](mailto:from@163.com) | 发件人邮箱                           |
| -t                       | [to@163.com](mailto:to@163.com)     | 收件人邮箱                           |
| -s                       | smtp.163.com                        | 发件人邮箱的smtp服务器               |
| -u                       | "我是邮件主题"                      | 邮件的标题                           |
| -o                       | message-content-type=html           | 邮件内容的格式，html表示它是html格式 |
| -o                       | message-charset=utf8                | 邮件内容编码                         |
| -xu                      | [from@163.com](mailto:from@163.com) | 发件人邮箱的用户名                   |
| -xp                      | 123456                              | 发件人邮箱密码（授权码）             |
| -m                       | "我是邮件内容"                      | 邮件的具体内容                       |

### 3.1.3 调整QQ邮箱设置

![QQ截图20180119225714.png-57kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.9382737801733039.png)

![QQ截图20180119230151.png-29.7kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.11016251584146453.png)

![QQ截图20180119225952.png-26.7kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.24569725578289825.png)

**测试邮件发送**

```
 [root@localhost sendEmail-v1.56]# sendEmail -f 215379068@qq.com -t 215379068@qq.com -u "zabbix_server" -s smtp.qq.com -o message-content-type=html -o message-charset=utf8 -xu 215379068@qq.com -xp rtqnwthiqajdbihd -m "邮件发送成功" Jan 19 18:09:10 localhost sendEmail[2403]: Email was sent successfully!
```

![QQ截图20180119231007.png-53.4kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.44742983751019993.png)

### 3.1.4 编写QQ邮件平台报警脚本

```
 [root@localhost alertscripts]# pwd /usr/local/zabbix/share/zabbix/alertscripts [root@localhost alertscripts]# cat sendmail.sh  #!/bin/bash # author:Mr.chen   to=$1  subject=$2  body=$3  from=215379068@qq.com  smtp=smtp.qq.com  passwd=rtqnwthiqajdbihd  /usr/local/bin/sendEmail -f "$from" -t "$to" -s "$smtp" -u "$subject" -o message-content-type=html -o message-charset=utf8 -xu "$from" -xp "$passwd" -m "$body" [root@localhost alertscripts]# chmod +x sendmail.sh  [root@localhost alertscripts]# chown zabbix.zabbix sendmail.sh 
```

### 3.1.5 脚本测试

```
 [root@localhost alertscripts]# sh sendmail.sh 215379068@qq.com "hello world" "新的一天" Jan 19 18:20:32 localhost sendEmail[2478]: Email was sent successfully!
```

![QQ截图20180119232055.png-56.9kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.2332598855043353.png)

### 3.1.6 修改zabbix_server.conf配置文件

```
[root@localhost alertscripts]# cat -n /usr/local/zabbix/etc/zabbix_server.conf | grep "447"
   447  # AlertScriptsPath=${datadir}/zabbix/alertscripts

#将上述内容修改成如下所示
[root@localhost alertscripts]# cat -n /usr/local/zabbix/etc/zabbix_server.conf | grep "447"
   447  AlertScriptsPath=/usr/local/zabbix/share/zabbix/alertscripts

#重启zabbix_server服务
[root@localhost zabbix]# /etc/init.d/zabbix_server restart
Shutting down zabbix_server:                               [  OK  ]
Starting zabbix_server:                                    [  OK  ]
```

### 3.1.7 创建报警媒介

![QQ截图20180119234327.png-34.9kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.2952056777588843.png)

![QQ截图20180119234558.png-24.5kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.8127712272069645.png)

![QQ截图20180119234640.png-32.4kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.8070793211524516.png)

![QQ截图20180119234839.png-25.3kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.7974419487047237.png)

![QQ截图20180120223425.png-31.3kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.911816064724265.png)

### 3.1.8 创建报警动作

![QQ截图20180119235155.png-26.2kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.03757635046018826.png)

![QQ截图20180120110138.png-26.4kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.6596750669826028.png)

![QQ截图20180120110308.png-31kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.9048969143818832.png)

![QQ截图20180120110353.png-50.6kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.7941732187786483.png)

步骤1-3也就是从1开始到3结束。一旦发生故障，就是执行sendmail.sh脚本发生报警邮件给zabbix用户。假如故障持续了1个小时，它也只发送3次，第1-3次（即前3次）邮箱发送给zabbix用户，时间间隔为0秒。如果改成1-0,0是表示不限制，无限发送。

![QQ截图20180120110812.png-34.9kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.3707460750086524.png)

![QQ截图20180120110850.png-14.6kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.4807719057132074.png)

### 3.1.9 QQ邮件报警测试

给自定义监控项nginx.avtive创建一个触发器，如下

![QQ截图20180120230607.png-15.5kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.6248192748289443.png)

> 利用Web进行访问，增加活动连接数，触发报警

![QQ截图20180120224433.png-77.9kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.31311760683838363.png)

## 3.2 微信报警平台

### 3.2.1 注册微信报警平台并绑定微信号

**企业号注册连接**：https://qy.weixin.qq.com/cgi-bin/loginpage

![QQ截图20180121001001.png-45.7kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.7225759562648051.png)

![QQ截图20180121001247.png-36kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.268547534560305.png)

![QQ截图20180121001751.png-19.2kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.10292079762135331.png)

### 3.2.2 编写微信平台报警脚本

编写脚本前，我们需要先记住3个关键的参数

（1）企业的CorpID

![QQ截图20180121005237.png-58.6kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.8306655054348717.png)

（2）企业的Secret

![QQ截图20180121005310.png-38kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.4642850427574996.png)

![QQ截图20180121005334.png-9kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.28885907066542216.png)

（3）部门ID号

![QQ截图20180121005557.png-47.2kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.018429817342111132.png)

然后我们就可以编写微信告警脚本了，如下：

```
[root@Zabbix_Server alertscripts]# cat weixin.sh 
#!/bin/bash
# author:Mr.chen

CropID="########"           #这里填写我们的应用的CropID

Secret="#######"            #这里是应用的Secret

#下面的GURL和PURL地址无需改变，不用做任何变动
GURL="https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=$CropID&corpsecret=$Secret"

Gtoken=`/usr/bin/curl -s -G $GURL | awk -F\" '{print $10}'`

PURL="https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=$Gtoken"

function body() {
        local int AppID=1000002 #这里是创建的应用ID
        local UserID=$1         #接收消息用户，系统传参
        local PartyID=1         #接收消息的部门ID
        local Msg=`echo "$@" | cut -d" " -f3-`
        printf '{\n'
        printf '\t"touser": "'"$UserID"\"",\n"
        printf '\t"toparty": "'"$PartyID"\"",\n"
        printf '\t"msgtype": "text",\n'
        printf '\t"agentid": "'" $AppID "\"",\n"
        printf '\t"text": {\n'
        printf '\t\t"content": "'"$Msg"\""\n"
        printf '\t},\n'
        printf '\t"safe":"0"\n'
        printf '}\n'
}

/usr/bin/curl --data-ascii "$(body $1 $2 $3)" $PURL
```

### 3.2.3 脚本测试

```
[root@Zabbix_Server alertscripts]# chmod +x weixin.sh 
[root@Zabbix_Server alertscripts]# chown zabbix.zabbix weixin.sh 
[root@Zabbix_Server alertscripts]# sh weixin.sh yinsendemogui "题目" "报警内容"
{"errcode":0,"errmsg":"ok","invaliduser":"#######"}
```

![IMG_3575.PNG-62.5kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.4392009073702783.png)

![IMG_3576.PNG-53.3kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.3437569689648663.png)

### 3.2.4 创建微信报警媒介类型

![QQ截图20180121011219.png-25.6kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.2320152475636812.png)

![QQ截图20180121011244.png-35.4kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.715345318196039.png)

![QQ截图20180121011307.png-21.8kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.45446703780806974.png)

![QQ截图20180121011355.png-14.1kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.253557701339594.png)

![QQ截图20180121011413.png-25.1kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.9639959835369447.png)

### 3.2.5 设定报警动作

![QQ截图20180121011641.png-17.9kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.8816010167949995.png)

![QQ截图20180121011900.png-25kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.49680917048062656.png)

### 3.2.6 微信平台报警测试

![QQ截图20180121012248.png-65.4kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.28550456324027595.png)

![IMG_3577.PNG-118.4kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.14639627963804824.png)

### 3.3 自定义自动报警的内容

（1）自定义内容样例

如果不修改报警的内容格式，看起来太乱了。我们可以按照如下方式修改

```
#告警通知格式样例
#题目
A故障：{TRIGGER.STATUS}，服务器：{HOSTNAME1}发生：{TRIGGER.NAME}故障！
#内容
告警主机：&nbsp；{HOSTNAME1}<br/>
告警时间：&nbsp；{EVENT.DATE} {EVENT.TIME}<br/>
告警等级：&nbsp；{TRIGGER.SEVERITY}<br/>
告警信息：&nbsp；{TRIGGER.NAME}<br/>
告警项目：&nbsp；{TRIGGER.KEY1}<br/>
问题详情：&nbsp；{ITEM.NAME}&nbsp{ITEM.VALUE}<br/>
当前状态：&nbsp；{TRIGGER.STATUS}&nbsp{ITEM.VALUE1}<br/>
事件ID：&nbsp；{EVENT.ID}
```

![QQ截图20180121195205.png-40.7kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.4232287884116399.png)

（2）样例测试

![QQ截图20180121195126.png-67.1kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.10735234066922095.png)

# 四，用户参数User parameters

## 4.1 概述

有时候当我们监控的项目在Zabbix预定义的key中没有定义时，这时候我们可以通过编写Zabbix的用户参数的方法来监控我们要求的项目item。形象一点说Zabbix代理端配置文件中的User parameters就相当于通过脚本获取要监控的值，然后把相关的脚本或者命令写入到配置文件中的User parameter中，然后Zabbix server读取配置文件中的返回值通过处理前端的方式返回给用户。

**用户参数的语法**

```
UserParameter=,
```

其中，Userparameter为关键字，key为用户自定义key名字可以随便起，为我们要运行的命令或者脚本。

**一个简单的例子：**

```
UserParameter=ping，echo 1
```

代理程序将会永远的返回1，当我们在服务器端添加item的key为ping的时候。

**稍微复杂的例子：**

```
UserParameter=mysql.ping,/usr/local/mysql/bin/mysqladmin ping | grep -c alive
```

当我们执行mysqladmin -uroot ping命令的时候如果mysql存活要返回mysqld is alive，我们通过grep -c来计算mysqld is alive的个数，如果mysql存活着，则个数为1，如果不存活很明显mysqld is alive的个数为0，通过这种方法我们可以来判断mysql的存活状态。

当我们在服务器端添加item的key为mysql.ping时候，对于Zabbix代理程序，如果mysql存活，则状态将返回1，否则，状态将返回0。

## 4.2 让key接受参数

让key也接受参数的方法使item添加时更具备了灵活性，例如系统预定义key：vm.memory.size[],其中的mode模式就是用户要接受的参数，当我们填写为free时则返回的为内存的剩余大小，如果我们填入的为userd时这返回的是内存已经使用的大小。

**相关语法：**

```
UserParameter=key[*],command
#描述：

key：key的值在主机系统中必须是唯一的，其中*代表命令中接受的参数

command：客户端系统中可执行的命令

#举例：

UserParameter=ping[*],echo $1
ping[0]:此时0就是*，也就是传入参数是0，$1也就是0，因此表达式将一直返回‘0’
ping[aaa]:此时aaa就是*，也就是传入参数是aaa，$1也就是aaa,因此表达式将一直返回‘aaa’
```

## 4.3 让我们自定义一个可以传递参数的监控项

我们做一个可以根据条件获取内存数值大小的监控项mem_check当我们键值为mem_check[free]时，获取剩余可用内存大小当我们键值为mem_check[used]时，获取实际占用内存大小当我们键值为mem_check时，获取总内存大小

### 4.3.1 我们先制作一个获取数据的脚本

```
[root@Zabbix_Server ~]# mkdir -p /server/scripts
[root@Zabbix_Server ~]# cd /server/scripts/
[root@Zabbix_Server scripts]# cat mem_check 
#!/bin/bash
# author:Mr.chen

case $1 in
    free)
        echo "`free | awk 'NR==3{print $4}'`"
        ;;
    used)
        echo "`free | awk 'NR==3{print $3}'`"
        ;;
    *)
        echo "`free | awk 'NR==2{print $2}'`"
        ;;
esac
```

#### 4.3.2 测试脚本

```
[root@Zabbix_Server scripts]# chmod +x mem_check 
[root@Zabbix_Server scripts]# chown zabbix.zabbix mem_check
[root@Zabbix_Server scripts]# sh mem_check 
1004412
[root@Zabbix_Server scripts]# sh mem_check free
782492
[root@Zabbix_Server scripts]# sh mem_check used
221912
```

#### 4.3.3 后台自定义一个监控项的键值

```
[root@Zabbix_Server ~]# cd /etc/zabbix/zabbix_agentd.d/
[root@Zabbix_Server zabbix_agentd.d]# cat mem_check.conf 
UserParameter=mem.check[*],/server/scripts/mem_check $1
```

#### 4.3.4 测试自定义的键值

```
#重启zabbix-agent客户端
[root@Zabbix_Server zabbix_agentd.d]# /etc/init.d/zabbix-agent restart
Shutting down Zabbix agent:                                [  OK  ]
Starting Zabbix agent:                                     [  OK  ]
[root@Zabbix_Server zabbix_agentd.d]# zabbix_get -s 192.168.0.220 -p 10050 -k "mem.check"
1004412
[root@Zabbix_Server zabbix_agentd.d]# zabbix_get -s 192.168.0.220 -p 10050 -k "mem.check[free]"
782676
[root@Zabbix_Server zabbix_agentd.d]# zabbix_get -s 192.168.0.220 -p 10050 -k "mem.check[used]"
221744
```

#### 4.3.5 前台自定义一个监控项及触发器

过程略，此时我相信同学们已经会了。请同学们自己尝试创建完整。

# 五，Agentd主动模式与被动模式

默认情况下，zabbix server会直接去每个agent上抓取数据，这对于agent来说，是被动模式，也是默认的一种获取数据的方式，但是，当zabbix server监控主机数量过多的时候，由server端去抓取agent上的数据，zabbix server会出现严重的性能问题，主要表现如下：

-  :Web操作很卡，容易出现502
-  :图层断裂
-  :开启的进程（Pollar）太多，即使减少item数量，以后加入一定量的机器也会有问题

**所以，下面主要往两个优化方向考虑：**

-  :用Proxy或者Node模式做分布式监控
-  :调整Agentd为主动模式

### 5.1 Agentd的配置调整

修改zabbix_agentd.conf配置文件，注意是打开如下参数：

```
ServerActive=192.168.0.220
Hostname=192.168.0.220
StartAgents=1
```

ServerActive是指定Agentd收集的数据往哪里发送，Hostname是必须要和zabbix web端添加主机时的主机名对应起来，这样zabbix server端接收到数据才能找到对应关系，这里为了兼容被动模式，没有把StartAgents设为0，如果一开始就是使用主动模式的话建议把StartAgents设为0，关闭被动模式。

### 5.2 zabbix Server端配置调整

如果开启了agent端的主动发送数据模式，需要在zabbix Server端修改如下两个参数，保证性能。

```
StartPollers=10     #把这个zabbix Server主动收集数据进程减少一些。
StartTrappers=200   #把这个负责处理Agentd推送过来的数据的进程开大一些。
```

### 5.3 调整模版

因此收集数据的模式发生了变化，因此需要把所有的监控项的类型由原来的“zabbix客户端”改成“zabbix客户端（主动式）”。

这样，只需要简单的几步，就完成了主动模式的切换，调整之后服务器不卡了，图层不裂了，进程也少了。

![QQ截图20180121221430.png-46.7kB](Zabbix%E5%89%8D%E7%AB%AFweb%E9%85%8D%E7%BD%AE.assets/0.3028670862865861.png)