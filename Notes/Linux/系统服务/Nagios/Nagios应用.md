[TOC]

## 七，服务器端Nagios图形监控显示和管理

> 前面搭建的Nagios服务虽然能显示信息，能报警。但是在企业工作中还会需要一个历史趋势图，跟踪每一个业务的长期趋势，并且能以图形的方式展示，例如：根据磁盘的剩余趋势，确定是否需要提前购买磁盘。

### 7.1 服务器端安装PNP生成图形监控曲线

> PNP是一款配合Nagios出图的软件，官方站点为：http://www.pnp4nagios.org

#### 7.1.1 PNP出图基础依赖软件安装

```
[root@Nagios 6]# yum -y install cairo pango zlib zlib-devel freetype freetype-devel gd gd-devel
[root@Nagios 6]# rpm -qa cairo pango zlib zlib-devel freetype freetype-devel gd gd-devel
freetype-2.3.11-17.el6.x86_64
zlib-1.2.3-29.el6.x86_64
zlib-devel-1.2.3-29.el6.x86_64
freetype-devel-2.3.11-17.el6.x86_64
gd-2.0.35-11.el6.x86_64
gd-devel-2.0.35-11.el6.x86_64
cairo-1.8.8-6.el6_6.x86_64
pango-1.28.1-11.el6.x86_64

#然后安装rrdtool依赖的libart_lgpl相关软件包，这个软件包要优先于rrdtool安装
[root@Nagios 6]# yum -y install libart_lgpl libart_lgpl-devel
[root@Nagios 6]# rpm -qa libart_lgpl libart_lgpl-devel
libart_lgpl-2.3.20-5.1.el6.x86_64
libart_lgpl-devel-2.3.20-5.1.el6.x86_64

#PNP工具最终是通过rrdtool实现的画图，因此需要提前安装rrdtool
[root@Nagios 6]# yum -y install rrdtool rrdtool-devel
[root@Nagios 6]# rpm -qa rrdtool rrdtool-devel
rrdtool-1.3.8-10.el6.x86_64
rrdtool-devel-1.3.8-10.el6.x86_64
[root@Nagios 6]# which rrdtool
/usr/bin/rrdtool
```

#### 7.1.2 安装出图web界面展示软件PNP

> 此处选择0.4.14的PNP版本，如果选择高版本在出图方面可能会有坑，正常情况下，选04版本已经足够了，因此，如果没有特殊需求，建议最好完全按照书本测试步骤，在弄清楚之后再变通版本。

**PNP软件无法yum安装，可通过编译的方式进行安装，操作过程如下：**

```
[root@Nagios ~]# yum -y install perl-Time-HiRes
[root@Nagios ~]# cd nagios/
[root@Nagios nagios]# ll pnp-0.4.14.tar.gz 
-rw-r--r--. 1 root root 455593 Aug 12 12:22 pnp-0.4.14.tar.gz
[root@Nagios nagios]# tar xf pnp-0.4.14.tar.gz -C /usr/src/
[root@Nagios nagios]# cd /usr/src/pnp-0.4.14/
[root@Nagios pnp-0.4.14]# ./configure \ 

> --with-rrdtool \
> --with-perfdata-dir=/usr/local/nagios/share/perfdata/
[root@Nagios pnp-0.4.14]# make all
[root@Nagios pnp-0.4.14]# make install
[root@Nagios pnp-0.4.14]# make install-config
[root@Nagios pnp-0.4.14]# make install-init
[root@Nagios pnp-0.4.14]# ll /usr/local/nagios/libexec/ | grep process
-rwxr-xr-x  1 nagios nagios  31813 Aug 19 23:04 process_perfdata.pl
```

> 如果configure后出现如下警告信息，请忽略：

```
#################

# WARNING：The RRDs Perl Modules are not found on your System

# Using RRDs will speedup things in larg

##################
```

**PNP提供了一个获取数据出图的Perl脚本，可以用如下命令查到：**

```
[root@Nagios pnp-0.4.14]# ll /usr/local/nagios/libexec/ | grep process
-rwxr-xr-x  1 nagios nagios  31813 Aug 19 23:04 process_perfdata.pl
```

#### 7.1.3 Nagios出图相关配置

1）执行编辑命令vi，需要改nagios.cfg主配置文件833行，将如下参数对应的值从0改为1，表示记录数据。

```
[root@Nagios nagios]# sed -n '833p' /usr/local/nagios/etc/nagios.cfg 
process_performance_data=0    #默认0，改为1

#然后继续向下大概在845，846行的位置，找到如下两项，取消参数开头的注释。
[root@Nagios nagios]# sed -n '845,846p' /usr/local/nagios/etc/nagios.cfg 
#host_perfdata_command=process-host-perfdata    #取消注释
#service_perfdata_command=process-service-perfdata  #取消注释
```

2）执行编辑命令vi，需要修改commands.cfg配置文件，定义出图获取数据的命令。

```
[root@Nagios nagios]# sed -n '227,238p' /usr/local/nagios/etc/objects/commands.cfg 
# 'process-host-perfdata' command definition
define command{
    command_name process-host-perfdata
    command_line  /usr/bin/printf "％b"             "$LASTHOSTCHECK$\t$HOSTNAME$\t$HOSTSTATE$\t$HOSTATTEMPT$\t$HOSTSTATETYPE$\t$HOSTEXECUTIONTIME$\t$HOSTOUTPUT$\t$HOSTPERFDATA$\n" >>/usr/local/nagios/var/host-perfdata.out
}
# 'process-service-perfdata' command definition
define command{
    command_name process-service-perfdata
    command_line /usr/bin/printf "％b" "$LASTSERVICECHECK$\t$HOSTNAME$\t$SERVICEDESC$\t$SERVICESTATE$\t$SERVICEATTEMPT$\t$SERVICESTATETYPE$\t$SERVICEEXECUTIONTIME$\t$SERVICELATENCY$\t$SERVICEOUTPUT$\t$SERVICEPERFDATA$\n" >>/usr/local/nagios/var/service-perfdata.out
}
```

**现在删除上述的默认配置，然后将其修改为如下的配置内容：**

```
[root@Nagios nagios]# sed -n '227,238p' /usr/local/nagios/etc/objects/commands.cfg 
# 'process-host-perfdata' command definition
define command{
    command_name    process-host-perfdata
    command_line    /usr/local/nagios/libexec/process_perfdata.pl
    }
# 'process-service-perfdata' command definition
define command{
    command_name    process-service-perfdata
    command_line    /usr/local/nagios/libexec/process_perfdata.pl
    }
```

3）执行检查语法命令

```
[root@Nagios nagios]# /etc/init.d/nagios checkconfig
#..以上省略若干...
Total Warnings: 0
Total Errors:   0
Things look okay - No serious problems were detected during the pre-flight check
 OK
```

4）执行命令使Nagios配置文件生效。

```
[root@Nagios nagios]# /etc/init.d/nagios reload     
Running configuration check...done.
Reloading nagios configuration...done
```

5）此时打开浏览器访问“http：//192.168.0.200/nagios/pnp/”,应该会出现如下图所示的图形界面，但是没有业务数据显示。

![QQ截图20170820114101.png-103.1kB](http://static.zybuluo.com/chensiqi/na5hm80i9epwetqhht11cx3x/QQ%E6%88%AA%E5%9B%BE20170820114101.png)

**如果同学们打开出现如下错误：**

![QQ截图20170820111137.png-58.8kB](http://static.zybuluo.com/chensiqi/zbwy19s7viv3l1fjxjlsme5e/QQ%E6%88%AA%E5%9B%BE20170820111137.png)

> 如果出现上图中的错误，先别着急，可能过一会儿重新访问上述地址就会恢复正常。
>  如果过了很长时间重新访问上述地址还不正常，可以执行如下命令看看，然后再访问试试：
>  yum -y install php-gd gd gd-devel

### 7.2 配置主机及服务获取状态数据出图

> 7.1结尾的图形是没有具体的业务数据图形趋势的，因为那时还没有为Nagios的各个主机和具体要监控的服务配置获取数据信息，下面是让各个主机或服务获取数据的配置。

#### 7.2.1 设置让被监控的主机记录数据

> 如果要让所有的主机获取数据并出趋势图，则需编辑Nagios的主机hosts.cfg文件，不过，只要在每一个被监控主机的配置下面增加同一个参数项“process_perf_data 1”即可。操作步骤如下：

```
[root@Nagios nagios]# cd /usr/local/nagios/etc/objects/
[root@Nagios objects]# cat hosts.cfg

# Define a host for the local machine
define host{
        use                     linux-server            
        host_name               web01
        alias                   web01
        address                 192.168.0.223
    process_perf_data 1    #为web01增加1此行，表示记录web01主机状态数据
        }
define host{
        use                     linux-server            
        host_name               web02
        alias                   web02
        address                 192.168.0.224
    process_perf_data 1     #为Web02增加此行，表示记录web02主机状态数据
        }

define hostgroup{
        hostgroup_name  linux-servers 
        alias           Linux Servers 
        members         web01,web02
```

#### 7.2.2 设置让被监控主机对应的服务记录数据

> 如果需要所有的主机对应的服务获取数据并出趋势图，则要编辑Nagios的服务配置文件services.cfg，当然，也只需要在每一个对应服务下面增加同一个参数项即可，即“process_perf_data 1”,配置步骤如下：

```
[root@Nagios objects]# cat /usr/local/nagios/etc/objects/services.cfg 

define service {
    use generic-service
    host_name web01,web02
    service_description Disk Partition
    check_command check_nrpe!check_disk
    process_perf_data 1             #为每个service添加此行
}
define service {
    use generic-service
    host_name web01,web02
    service_description Swap Useage
    check_command check_nrpe!check_swap
    process_perf_data 1             #为每个service添加此行
}
define service {
    use generic-service
    host_name web01,web02
    service_description MEM Useage
    check_command check_nrpe!check_mem
    process_perf_data 1             #为每个service添加此行
}
define service {
    use generic-service
    host_name web01,web02
    service_description Current Load
    check_command check_nrpe!check_load
    process_perf_data 1                 #为每个service添加此行
}
define service {
    use generic-service
    host_name web01,web02
    service_description Disk lostat
    check_command check_nrpe!check_iostat!5!11
    process_perf_data 1                 #为每个service添加此行
}
define service {
    use generic-service
    host_name web01,web02
    service_description PING
    check_command check_ping!100.0,20%!500.0,60%
    process_perf_data 1                 #为每个service添加此行
}

#url examples http://www.yunjisuan.com

define service {

    use generic-service
    host_name web01
    service_description www_url
    check_command check_weburl! -H www.yunjisuan.com
    process_perf_data 1                 #为每个service添加此行
    
}

define service {

    use generic-service
    host_name web01
    service_description www_url
    check_command check_http
    process_perf_data 1                 #为每个service添加此行

}

define service {

    use generic-service
    host_name web01
    ervice_description www_static_url
    check_command check_weburl! -H www.yunjisuan.com -u /static/test.html
    process_perf_data 1                 #为每个service添加此行

}

define service {

    use generic-service
    host_name web01
    service_description www_yunjisuan_url
    check_command check_weburl! -H www.yunjisuan.com -u "/article/index.phpm=article&a=list&id=670"
    process_perf_data 1                 #为每个service添加此行

}

#tcp examples
define service {

    use generic-service
    host_name web01
    service_description ssh_22
    check_command check_tcp! 22
    process_perf_data 1                     #为每个service添加此行

}

define service {

    use generic-service
    host_name web01
    service_description http_80
    check_command check_tcp! 80
    process_perf_data 1                     #为每个service添加此行
    
}
```

> 由于每个主机对应的服务内容太多了，因此可以采取在所有服务对应的统一模板里添加配置参数的方式，这样可使所有的服务都可以生效。这里每个服务使用的模板就是由服务里的“use generic-service”这个选项确定的，查看与模板文件里服务模板generic-service名对应的服务参数：

```
[root@Nagios objects]# sed -n '154,177p' /usr/local/nagios/etc/objects/templates.cfg | awk -F ";" '{print $1}'
        name                            generic-service     
        active_checks_enabled           1               
        passive_checks_enabled          1               
        parallelize_check               1               
        obsess_over_service             1               
        check_freshness                 0               
        notifications_enabled           1               
        event_handler_enabled           1               
        flap_detection_enabled          1               
        failure_prediction_enabled      1               
        process_perf_data               1               
        retain_status_information       1               
        retain_nonstatus_information    1               
        is_volatile                     0               
        check_period                    24x7            
        max_check_attempts              3           
        normal_check_interval           10          
        retry_check_interval            2           
        contact_groups                  admins          
    notification_options        w,u,c,r         
        notification_interval           60          
        notification_period             24x7            
         register                        0              
        }
```

> **提示：**
>  为了看的清晰，这里去掉了所有注释，服务的模板里默认已经配置了“process_perf_data  1”,即凡是使用templates.cfg模板文件里名字为generic-service的模板，均作为服务的模板，这样就相当于所有服务都执行generic-service模板里的配置了。

**配置完毕重启Nagios服务：**

```
[root@Nagios objects]# /etc/init.d/nagios reload     
Running configuration check...done.
Reloading nagios configuration...done
```

> 到此，如果等一段时间，然后查看PNP URL就可以发现生成了图形数据，有些数据需要压力测试或者真实环境才能看到，例如主机负载等。趋势图如下图所示：

![QQ截图20170820122224.png-98.7kB](http://static.zybuluo.com/chensiqi/yh74m0cfb0a8q9o9szeud0sp/QQ%E6%88%AA%E5%9B%BE20170820122224.png)

### 7.3 整合PNP URL超链接到Nagios Web界面

> 在整合PNP URL超链接到Nagios Web界面后，会在所有的主机或主机对应服务的前面，出现一个闪电样的超链接1图标，单击超链接，就可以查看到对应的主机或服务实际的监控状态趋势图。

#### 7.3.1 给被监控的所有主机添加超链接图标

> 默认情况PNP的URL为http：//192.168.0.200/nagios/pnp/index.php和Nagios不在一个界面里，所以查看主机或服务对应的趋势图很费劲。那么如何完善呢？
>  我们可以直接在host.cfg里在希望出图的主机里配置如下一行参数：

```
action_url  /nagios/pnp/index.php?host=$HOSTNAME$    #实际上就是给URL传个主机参数
```

**然后编辑host.cfg，增加上述配置。配置结果如下：**

```
[root@Nagios objects]# cat /usr/local/nagios/etc/objects/hosts.cfg

# Define a host for the local machine
define host{
        use                     linux-server            
        host_name               web01
        alias                   web01
        address                 192.168.0.223
    process_perf_data 1
    action_url  /nagios/pnp/index.php?host=$HOSTNAME$    #添加超链接图标
        }
define host{
        use                     linux-server            
        host_name               web02
        alias                   web02
        address                 192.168.0.224
    process_perf_data 1
    action_url  /nagios/pnp/index.php?host=$HOSTNAME$    #添加超链接图标
        }

define hostgroup{
        hostgroup_name  linux-servers 
        alias           Linux Servers 
        members         web01,web02
        }
```

**接着，检查语法重新加载Nagios**

```
[root@Nagios objects]# /etc/init.d/nagios reload
Running configuration check...done.
Reloading nagios configuration...done
```

> 如果配置过程都正确，打开浏览器访问Nagios界面，最终可以看到如下图所示的图形。图中，右边方框里标记的白色方格里，中间带波浪线的就是超链接图标。单击进去即可看到一个主机所有的服务图。

![QQ截图20170820134626.png-42.9kB](http://static.zybuluo.com/chensiqi/aao7wcxw2z5ix6ehtuny2ik7/QQ%E6%88%AA%E5%9B%BE20170820134626.png)

#### 7.3.2 给被监控主机指定的服务添加超链接图标

> 和上述主机添加超链接图标的配置几乎一样，执行“vi /usr/local/nagios/etc/objects/services.cfg”,添加如下内容：

```
action_url /nagios/pnp/index.php?host=$HOSTNAME$&srv=$SERVICEDESC$

#实际上就是给URL传了一个主机的参数和一个主机对应服务的参数
```

> 给具体服务增加超链接配置方法是，直接在define service {}大括号中增加参数即可，具体配置的内容如下“action_url参数部分”：

```
[root@Nagios objects]# cat /usr/local/nagios/etc/objects/services.cfg

define service {
    use generic-service
    host_name web01,web02
    service_description Disk Partition
    check_command check_nrpe!check_disk
    process_perf_data 1
    action_url /nagios/pnp/index.php?host=$HOSTNAME$&srv=$SERVICEDESC$
    #给具体服务增加超链接配置
}
define service {
    use generic-service
    host_name web01,web02
    service_description Swap Useage
    check_command check_nrpe!check_swap
    process_perf_data 1
    action_url /nagios/pnp/index.php?host=$HOSTNAME$&srv=$SERVICEDESC$
    #给具体服务增加超链接配置
}
define service {
    use generic-service
    host_name web01,web02
    service_description MEM Useage
    check_command check_nrpe!check_mem
    process_perf_data 1
    action_url /nagios/pnp/index.php?host=$HOSTNAME$&srv=$SERVICEDESC$
    #给具体服务增加超链接配置
}
define service {
    use generic-service
    host_name web01,web02
    service_description Current Load
    check_command check_nrpe!check_load
    process_perf_data 1
}
define service {
    use generic-service
    host_name web01,web02
    service_description Disk lostat
    check_command check_nrpe!check_iostat!5!11
    process_perf_data 1
}
define service {
    use generic-service
    host_name web01,web02
    service_description PING
    check_command check_ping!100.0,20%!500.0,60%
    process_perf_data 1
}

#url examples http://www.yunjisuan.com

define service {

        use generic-service
        host_name web01
        service_description www_url
    check_command check_weburl! -H www.yunjisuan.com
    process_perf_data 1
    
}

define service {

    use generic-service
    host_name web01
    service_description www_url
    check_command check_http
    process_perf_data 1

}

define service {

    use generic-service
        host_name web01
        service_description www_static_url
    check_command check_weburl! -H www.yunjisuan.com -u /static/test.html
    process_perf_data 1

}

define service {

        use generic-service
        host_name web01
        service_description www_yunjisuan_url
    check_command check_weburl! -H www.yunjisuan.com -u "/article/index.phpm=article&a=list&id=670"
    process_perf_data 1

}

#tcp examples
define service {

    use generic-service
    host_name web01
    service_description ssh_22
    check_command check_tcp! 22
    process_perf_data 1

}

define service {

        use generic-service
        host_name web01
        service_description http_80
    check_command check_tcp! 80
    process_perf_data 1
    
}
```

**配置完成后的效果图如下：**

![QQ截图20170820135700.png-78.6kB](http://static.zybuluo.com/chensiqi/uq8do9w5f4djzopzv5gqukrv/QQ%E6%88%AA%E5%9B%BE20170820135700.png)

> 也可以快速设置让全部的服务出图，找到templates.cfg模板文件，找到默认的服务名generic-service，在这个服务名大括号的内部结尾增加“action_url /nagios/pnp/index.php?host=$HOSTNAME$&srv=$SERVICEDESC$ ”一行即可。

```
[root@Nagios objects]# sed -n '153,178p' /usr/local/nagios/etc/objects/templates.cfg | awk -F ";" '{print $1}'
define service{
        name                            generic-service     
        active_checks_enabled           1               
        passive_checks_enabled          1               
        parallelize_check               1               
        obsess_over_service             1               
        check_freshness                 0               
        notifications_enabled           1               
        event_handler_enabled           1               
        flap_detection_enabled          1               
        failure_prediction_enabled      1               
        process_perf_data               1               
        retain_status_information       1               
        retain_nonstatus_information    1               
        is_volatile                     0               
        check_period                    24x7            
        max_check_attempts              3           
        normal_check_interval           10          
        retry_check_interval            2           
        contact_groups                  admins          
    notification_options        w,u,c,r         
        notification_interval           60          
        notification_period             24x7            
         register                        0              
    action_url /nagios/pnp/index.php?host=$HOSTNAME$&srv=$SERVICEDESC$
       #在最后添加此行
        }
```

> 这样所有主机的所有服务都将增加出图的超链接图标了。
>  现在，人要检查语法并重新加载Nagios

```
[root@Nagios objects]# /etc/init.d/nagios reload
Running configuration check...done.
Reloading nagios configuration...done
```

**全部主机和服务的监控图最终结果如下图所示：**

![QQ截图20170820140636.png-105.5kB](http://static.zybuluo.com/chensiqi/jqzlrzo3aog8s387sfhz0bk4/QQ%E6%88%AA%E5%9B%BE20170820140636.png)

> 此时，单击任意一个超链接图标，就可以查看对应的主机或服务的业务趋势图了，到此，Nagios的主机和服务出图的配置就完成了，是不是很简单？

#### 7.3.3 Nagios出图获取的数据存放路径

> 想真正绘制出业务的趋势图全靠下面命令生成的数据。这些历史数据要备份好。

```
[root@Nagios objects]# ll /usr/local/nagios/share/perfdata/
total 8
drwxr-xr-x 2 nagios nagios 4096 Aug 20 02:10 web01
drwxr-xr-x 2 nagios nagios 4096 Aug 20 02:03 web02
[root@Nagios objects]# tree /usr/local/nagios/share/perfdata/
/usr/local/nagios/share/perfdata/
|-- web01
|   |-- Current_Load.rrd
|   |-- Current_Load.xml
|   |-- Disk_Partition.rrd
|   |-- Disk_Partition.xml
|   |-- Disk_lostat.rrd
|   |-- Disk_lostat.xml
|   |-- MEM_Useage.rrd
|   |-- MEM_Useage.xml
|   |-- PING.rrd
|   |-- PING.xml
|   |-- Swap_Useage.rrd
|   |-- Swap_Useage.xml
|   |-- http_80.rrd
|   |-- http_80.xml
|   |-- ssh_22.rrd
|   |-- ssh_22.xml
|   |-- www_static_url.rrd
|   |-- www_static_url.xml
|   |-- www_url.rrd
|   |-- www_url.xml
|   |-- www_yunjisuan_url.rrd
|   `-- www_yunjisuan_url.xml
`-- web02
    |-- PING.rrd
    `-- PING.xml

2 directories, 24 files
```

## 八，实现将Nagios故障报警给管理员

**要将Nagios故障报警给管理员时，常用的方式包括邮件报警和手机报警，下面分别介绍**

### 8.1 邮件报警

> - 普通邮件报警就是在故障发生或恢复时，将报警信息发到系统管理员或相关维护人员的信箱中，一般来说最好使用公司内部信箱作为报警信箱。同学们回家学习测试时如果用QQ，126等信箱可能会有收不到邮件的情况或者被当作垃圾邮件了。
> - 一般白天上班时，邮件报警还算比较及时，但是如果人不在计算机旁，邮件报警就不行了，因此，邮件报警只适合不是特别重要的业务，或者作为发送大量报警信息中的一个辅助方式，如硬盘，内存，及日志相关等不需要及时解决的服务报警。故而，在生产环境中，邮件报警一般会结合其他报警方式一起使用。
> - 那么，下面就来看一下邮件报警的基本配置方法。

**首先，添加监控报警的接收Email地址**

```
[root@Nagios objects]# sed -n '35p' /usr/local/nagios/etc/objects/contacts.cfg | awk -F ";" '{print $1}'
        email                           215379068@qq.com    #将本行内容改成你的QQ邮箱
```

**打开postfix服务**

```
[root@Nagios objects]# /etc/init.d/postfix start
Starting postfix:                                          [  OK  ]
[root@Nagios objects]# echo "/etc/init.d/postfix start" >> /etc/rc.local 
[root@Nagios objects]# tail -3 /etc/rc.local
touch /var/lock/subsys/local
/etc/init.d/nagios start
/etc/init.d/postfix start
```

**用命令测试发邮件：**

```
[root@Nagios objects]# echo "this is test email" | mail -s "yunjisuan" 215379068@qq.com

#将邮件从QQ拦截名单取出，然后添加白名单
```

![QQ截图20170820160305.png-40kB](http://static.zybuluo.com/chensiqi/5xfnnvfyei3llswuoaxmj1z0/QQ%E6%88%AA%E5%9B%BE20170820160305.png)

> **特别警示！**
>  同学们在家玩Nagios一定要用自己的QQ玩，谁给我发，我和谁急-_-!

### 8.2 关于邮件相关的配置文件的大概解释（喜欢的同学自己百度含义）

**templates.cfg系统定义模板**

```
#模板：generic-service
[root@Nagios objects]# sed -n '153,178p' /usr/local/nagios/etc/objects/templates.cfg | awk -F ";" '{print $1}'
define service{
        name                            generic-service     
        active_checks_enabled           1               
        passive_checks_enabled          1               
        parallelize_check               1               
        obsess_over_service             1               
        check_freshness                 0               
        notifications_enabled           1               
        event_handler_enabled           1               
        flap_detection_enabled          1               
        failure_prediction_enabled      1               
        process_perf_data               1               
        retain_status_information       1               
        retain_nonstatus_information    1               
        is_volatile                     0               
        check_period                    24x7        #告诉Nagios检查服务的时间段   
        max_check_attempts              3           #对Nagios服务的最大检查次数
        normal_check_interval           10          #两次检查的时间间隔   
        retry_check_interval            2           #重新检查时间间隔
        contact_groups                  admins      #指定联系人主 
    notification_options        w,u,c,r             #定义何种异常可以被通知（email），w即warn表示警告状态，r即recover，表示恢复状态
        notification_interval           60          #服务出现异常，故障一直没解决，Nagios再次对联系人发出通知的时间间隔       
        notification_period             24x7        #指定email的时间段    
         register                        0              
    action_url /nagios/pnp/index.php?host=$HOSTNAME$&srv=$SERVICEDESC$
        }

#模板：generic-contact
[root@Nagios objects]# sed -n '28,37p' /usr/local/nagios/etc/objects/templates.cfg | awk -F ";" '{print $1}'
define contact{
        name                            generic-contact     #联系人名称
        service_notification_period     24x7                #服务异常，发送通知时间段
        host_notification_period        24x7                #主机异常，发送通知时间段   
        service_notification_options    w,u,c,r,f,s     #何种异常进行通知
        host_notification_options       d,u,r,f,s       #何种异常进行通知
        service_notification_commands   notify-service-by-email #定义服务异常发送邮件命令，commands.cfg文件里定义
        host_notification_commands      notify-host-by-email #定义主机异常发送邮件命令，commands.cfg文件里定义    
        register                        0               
        }
```

**commands.cfg命令定义模板**

```
#定义发送邮件命令
[root@Nagios objects]# sed -n '27,37p' commands.cfg 
# 'notify-host-by-email' command definition
define command{
    command_name    notify-host-by-email        #主机异常发送邮件命令的定义
    command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n" | /bin/mail -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$
    }

# 'notify-service-by-email' command definition
define command{
    command_name    notify-service-by-email     #服务异常发送邮件命令的定义
    command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n" | /bin/mail -s "** $NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$ **" $CONTACTEMAIL$
    }
```

**contacts.cfg联系人定义模板**

```
[root@Nagios objects]# cat contacts.cfg | egrep -v "#|^$" | awk -F ";" '{print $1}'
define contact{
        contact_name                    nagiosadmin     #定义成员
    use             generic-contact     
        alias                           Nagios Admin    #成员别名   
        email                           215379068@qq.com    #成员邮箱
        }
define contactgroup{
        contactgroup_name       admins                  #联系人组名
        alias                   Nagios Administrators   #别名
        members                 nagiosadmin             #组员名单定义
        }
```

### 8.3 Nagios各个配置文件的相互间关系图

![QQ截图20170820155456.png-22.5kB](http://static.zybuluo.com/chensiqi/0672gufhm947e0apxniyvhu2/QQ%E6%88%AA%E5%9B%BE20170820155456.png)

## 九，Nagios插件开发

### 9.1 概述

#### 9.1.1 什么是Nagios插件

> 前文在部署Nagios服务时已经安装了nagios-plugins-1.4.16.tar.gz,这个软件包就是Nagios的插件安装包，安装后，执行ls -l /usr/local/nagios/libexec可以看到如下插件内容：

```
[root@Nagios objects]# ls -l /usr/local/nagios/libexec/        
total 5288
lrwxrwxrwx  1 root   root       27 Aug 18 08:29 check_111 -> /service/scripts/check_test
-rwxr-xr-x. 1 nagios nagios 376524 Aug 14 10:11 check_apt
-rwxr-xr-x. 1 nagios nagios   2245 Aug 14 10:11 check_breeze
-rwxr-xr-x. 1 nagios nagios 128296 Aug 14 10:11 check_by_ssh
lrwxrwxrwx. 1 root   root        9 Aug 14 10:11 check_clamd -> check_tcp
-rwxr-xr-x. 1 nagios nagios  85694 Aug 14 10:11 check_cluster
-r-sr-xr-x. 1 root   nagios 123603 Aug 14 10:11 check_dhcp
-rwxr-xr-x. 1 nagios nagios 417895 Aug 14 10:11 check_disk
-rwxr-xr-x. 1 nagios nagios   9148 Aug 14 10:11 check_disk_smb
-rwxr-xr-x. 1 nagios nagios  80689 Aug 14 10:11 check_dummy
-rwxr-xr-x. 1 nagios nagios   3056 Aug 14 10:11 check_file_age
-rwxr-xr-x. 1 nagios nagios   6318 Aug 14 10:11 check_flexlm
lrwxrwxrwx. 1 root   root        9 Aug 14 10:11 check_ftp -> check_tcp
-rwxr-xr-x. 1 nagios nagios 520614 Aug 14 10:11 check_http
-r-sr-xr-x. 1 root   nagios 133689 Aug 14 10:11 check_icmp
-rwxr-xr-x. 1 nagios nagios  93416 Aug 14 10:11 check_ide_smart
-rwxr-xr-x. 1 nagios nagios  15137 Aug 14 10:11 check_ifoperstatus
-rwxr-xr-x. 1 nagios nagios  12601 Aug 14 10:11 check_ifstatus
lrwxrwxrwx. 1 root   root        9 Aug 14 10:11 check_imap -> check_tcp
-rwxr-xr-x. 1 nagios nagios   6890 Aug 14 10:11 check_ircd
lrwxrwxrwx. 1 root   root        9 Aug 14 10:11 check_jabber -> check_tcp
-rwxr-xr-x. 1 nagios nagios 106573 Aug 14 10:11 check_load
-rwxr-xr-x. 1 nagios nagios   6020 Aug 14 10:11 check_log
-rwxr-xr-x. 1 nagios nagios  20287 Aug 14 10:11 check_mailq
-rwxr-xr-x. 1 nagios nagios  93142 Aug 14 10:11 check_mrtg
-rwxr-xr-x. 1 nagios nagios  92487 Aug 14 10:11 check_mrtgtraf
-rwxr-xr-x. 1 nagios nagios 105606 Aug 14 10:11 check_nagios
lrwxrwxrwx. 1 root   root        9 Aug 14 10:11 check_nntp -> check_tcp
lrwxrwxrwx. 1 root   root        9 Aug 14 10:11 check_nntps -> check_tcp
-rwxrwxr-x. 1 nagios nagios  76744 Aug 14 10:32 check_nrpe
-rwxr-xr-x. 1 nagios nagios 127679 Aug 14 10:11 check_nt
-rwxr-xr-x. 1 nagios nagios 130078 Aug 14 10:11 check_ntp
-rwxr-xr-x. 1 nagios nagios 119167 Aug 14 10:11 check_ntp_peer
-rwxr-xr-x. 1 nagios nagios 117728 Aug 14 10:11 check_ntp_time
-rwxr-xr-x. 1 nagios nagios 159372 Aug 14 10:11 check_nwstat
-rwxr-xr-x. 1 nagios nagios   8324 Aug 14 10:11 check_oracle
-rwxr-xr-x. 1 nagios nagios 108934 Aug 14 10:11 check_overcr
-rwxr-xr-x. 1 nagios nagios 132691 Aug 14 10:11 check_ping
-rwxr-xr-x  1 nagios nagios   6184 Aug 19 23:04 check_pnp_rrds.pl
lrwxrwxrwx. 1 root   root        9 Aug 14 10:11 check_pop -> check_tcp
-rwxr-xr-x. 1 nagios nagios 396833 Aug 14 10:11 check_procs
-rwxr-xr-x. 1 nagios nagios 106492 Aug 14 10:11 check_real
-rwxr-xr-x. 1 nagios nagios   9584 Aug 14 10:11 check_rpc
-rwxr-xr-x. 1 nagios nagios   1412 Aug 14 10:11 check_sensors
lrwxrwxrwx. 1 root   root        9 Aug 14 10:11 check_simap -> check_tcp
-rwxr-xr-x. 1 nagios nagios 446511 Aug 14 10:11 check_smtp
lrwxrwxrwx. 1 root   root        9 Aug 14 10:11 check_spop -> check_tcp
-rwxr-xr-x. 1 nagios nagios 103000 Aug 14 10:11 check_ssh
lrwxrwxrwx. 1 root   root        9 Aug 14 10:11 check_ssmtp -> check_tcp
-rwxr-xr-x. 1 nagios nagios 108233 Aug 14 10:11 check_swap
-rwxr-xr-x. 1 nagios nagios 160386 Aug 14 10:11 check_tcp
-rwxr-xr-x. 1 nagios nagios 105022 Aug 14 10:11 check_time
lrwxrwxrwx. 1 root   root        9 Aug 14 10:11 check_udp -> check_tcp
-rwxr-xr-x. 1 nagios nagios 117534 Aug 14 10:11 check_ups
-rwxr-xr-x. 1 nagios nagios  83434 Aug 14 10:11 check_users
-rwxr-xr-x. 1 nagios nagios   2939 Aug 14 10:11 check_wave
-rwxr-xr-x. 1 nagios nagios 109723 Aug 14 10:11 negate
-rwxr-xr-x  1 nagios nagios  31813 Aug 19 23:04 process_perfdata.pl
-rwxr-xr-x. 1 nagios nagios 103242 Aug 14 10:11 urlize
-rwxr-xr-x. 1 nagios nagios   1904 Aug 14 10:11 utils.pm
-rwxr-xr-x. 1 nagios nagios   2728 Aug 14 10:11 utils.sh
```

> **提示：**
>  默认安装后大概有60个左右的插件，数量比较多，这里只介绍几个常见的。
>  以上结果内容都是Nagios插件，现在大家应该对Nagios插件有一个基本的了解了。其实，Nagios软件本身仅仅是一个监控的平台，如果要监控具体的主机及服务的状态和数据信息，还必须配置或调用插件或程序文件才能完成任务，因此，如果没有Nagios插件，Nagios就是一个空壳，啥都做不了。

#### 9.1.2 为什么要开发Nagios插件

> - 既然已经安装了Nagios的插件软件包，为什么还要开发Nagios插件呢？
> - 首先想说明的是，在生产场景中常用的大部分服务都是不需要编写插件就可以完成监控的，check_http,check_tcp,check_nrpe等这些自带的插件已经很强大了。但是，仍然有部分我们想要监控的服务，是Nagios未自带插件的，如：监控LVS  RS的lo网卡的VIP，监控NFS状态，又或是监控iostat，mem，sar系统指标及相关APP应用（MQ队列）等。这个时候我们有两个选择，一个是去网上搜索，看看有没有别人写过的脚本，拿来使用或修改后使用；另外就是自己开发编写脚本。这里建议大家学会手工编写插件，如果开始不会写，可以把网上别人分享的插件拿来改，改着改着就会写了。
> - 如果要开发插件，最好掌握一门开发语言，例如：Shell，Python。

### 9.2 编写Nagios插件的规则

#### 9.2.1 编写Nagios插件说明

> - Nagios插件是Nagios提供的一种通过可扩展的方式部署的程序组件，该插件可通过Shell，Java，C/C++，PHP等多种语言开发，运维或者系统架构人员只要通过修改Nagios配置文件和相应参数，就能很方便的将该插件集成到Nagios中，实现对目标系统的监控。
> - Nagios服务为1插件程序提供了两个返回值接口和插件交互：一个是插件执行后的退出状态码，另一个是插件执行过程中在控制台打印的1第一行数据。退出状态码可以被Nagios主程序作为判断被监控系统服务状态的依据，控制台打印的第一行数据可以被Nagios主程序作为被监控系统服务状态的补充说明，会显示在Web管理页面，如下图所示：

![QQ截图20170820165857.png-4.5kB](http://static.zybuluo.com/chensiqi/il1jlkx03j5ccp0xhxhs3s2o/QQ%E6%88%AA%E5%9B%BE20170820165857.png)

为了管理Nagios插件，Nagios每查询一个服务的状态时，就会产生一个子进程，并使用来自该命令的输出和退出代码来确定其具体的状态。Nagios主程序可识别的插件的退出状态码和说明如下：

- OK：退出代码，0表示服务工作正常。
- WARNING：退出代码，1表示服务处于警告状态
- CRITICAL：退出代码，2表示服务处于紧急，严重状态。
- UNKNOWN：退出代码,3表示服务处于未知状态。

> **注意:**
>  此处数字代码的含义曾经有公司面试时考过。
>
> 最后一种状态通常表示该插件无法确定服务的状态。例如，可能出现了网络或内部错误。相关状态可以从如下文件中看到：

```
[root@Nagios objects]# head -7 /usr/local/nagios/libexec/utils.sh 
#! /bin/sh

STATE_OK=0
STATE_WARNING=1
STATE_CRITICAL=2
STATE_UNKNOWN=3
STATE_DEPENDENT=4

#提示：结尾处比列举的还多个状态，但不常用
```

#### 9.2.2 Nagios插件开发原理

> Nagios插件程序中需要调用监控服务规定的操作序列，并根据预先定义的规则，对返回结果进行分析，判断服务的当前状态，然后以指定的状态码退出程序，同时将对该状态的说明**不换行**输出到控制台。

#### 9.2.3 Nagios插件开发语言

> Nagios的插件开发不限制任何开发语言，只要该插件能被Nagios调用，并获取到相应业务数据就OK，如能在命令行执行输出结果也可以，常用的插件语言有Shell，Perl，Python，PHP， C/C++。

### 9.3 使用Shell开发Nagios插件

#### 9.3.1 编写检查WebURL地址的插件

**以下脚本只是针对访问客户端192.168.0.223的IP的**

```
[root@Nagios libexec]# cat check_url.sh 
#!/bin/bash
# anthor:Mr.chen by 2017-8-20

wget -T 10 --spider 192.168.0.223 >/dev/null 2>&1   #用wget检查192.168.0.223是不是可以访问，-T超时时间 --spider不下载网页

if [ $? -eq 0 ];then                    #判断上述wget命令返回值，0成功非0失败
    echo "URL 192.168.0.223 OK"
    exit 0
else
    echo "URL 192.168.0.223 CRITICAL"
    exit 2
fi
```

**下面利用传参把脚本改进为通用的WebURL插件**

```
[root@Nagios libexec]# cat check_url.sh 
#!/bin/bash
# anthor:Mr.chen by 2017-8-20

PROGNAME=`basename $0`      #取脚本名
PROGPATH=`dirname $0`       #取脚本路径

usage(){            #打印帮助

    echo "Usage: /bin/sh ${PROGPATH}/${PROGNAME} url" 
    exit 1

}

[ $# -ne 1 ] && usage           #参数个数不是1，打印帮助

wget -T 10 --spider $1 >/dev/null 2>&1       #URL地址改成传参   

if [ $? -eq 0 ];then
    echo "URL $1 OK"
    exit 0
else
    echo "URL $1 CRITICAL"
    exit 2
fi
```

**以下是监控WebURL的插件脚本专业型写法**

```
[root@Nagios libexec]# cat check_url.sh 
#!/bin/bash
# anthor:Mr.chen by 2017-8-20

PROGNAME=`basename $0`
PROGPATH=`dirname $0`

usage(){

    echo "Usage: /bin/sh ${PROGPATH}/${PROGNAME} url" 
    exit 1

}

[ $# -ne 1 ] && usage

. $PROGPATH/utils.sh

if wget -T 20 --spider $1 >/dev/null 2>&1;then
    echo "URL $1 OK"
    exit $STATE_OK
else
    echo "URL $1 NO"
    exit $STATE_CRITICAL
fi
```

**最后手工测试以下改进的WebURL插件脚本**

```
[root@Nagios libexec]# sh /usr/local/nagios/libexec/check_url.sh www.yunjisuan.com
URL www.yunjisuan.com OK
[root@Nagios libexec]# echo $?
0
[root@Nagios libexec]# sh /usr/local/nagios/libexec/check_url.sh bbs.yunjisuan.com
URL bbs.yunjisuan.com OK
[root@Nagios libexec]# echo $?
0
[root@Nagios libexec]# sh /usr/local/nagios/libexec/check_url.sh blog.yunjisuan.com
URL blog.yunjisuan.com NO
[root@Nagios libexec]# echo $?
2
```

#### 9.3.2 WebURL插件脚本的部署过程（主动监控方式）

> Nagios主动模式监控和Nagios客户端的nrpe进程没有关系。
>  主动模式的所有操作完全在Nagios主服务器上进行。部署步骤如下：

**（1）开发check_url.sh,放到/usr/local/nagios/libexec中，授权为可执行**

```
root@Nagios libexec]# cd /usr/local/nagios/libexec/
[root@Nagios libexec]# chmod +x check_url.sh 
[root@Nagios libexec]# ll check_url.sh
-rwxr-xr-x 1 root root 337 Aug 20 06:38 check_url.sh
[root@Nagios libexec]# cat check_url.sh 
#!/bin/bash
# anthor:Mr.chen by 2017-8-20

PROGNAME=`basename $0`
PROGPATH=`dirname $0`

usage(){

    echo "Usage: /bin/sh ${PROGPATH}/${PROGNAME} url" 
    exit 1

}

[ $# -ne 1 ] && usage

. $PROGPATH/utils.sh

if wget -T 20 --spider $1 >/dev/null 2>&1;then
    echo "URL $1 OK"
    exit $STATE_OK
else
    echo "URL $1 NO"
    exit $STATE_CRITICAL
fi
```

**（2）在commands.cfg中建立check_url命令：**

```
[root@Nagios objects]# cd /usr/local/nagios/etc/objects/
[root@Nagios objects]# tail -7 commands.cfg 
# 'check_url' command definition by Mr.chen
define command {

    command_name check_url
    command_line $USER1$/check_url.sh 192.168.0.223    #加载脚本并传参数

}
#提示：$USER1$是Nagios默认变量，为/usr/local/nagios/libexec
```

**（3）在services.cfg里添加监控上述URL地址的服务**

> 可以将服务直接添加进services里也可以，写一个子服务的配置文件，写在/usr/local/nagios/etc/objects/services目录里

```
#创建需要监控的子服务配置文件
[root@Nagios objects]# pwd
/usr/local/nagios/etc/objects
[root@Nagios objects]# cd services
[root@Nagios services]# pwd
/usr/local/nagios/etc/objects/services
[root@Nagios services]# vim check_url.cfg
[root@Nagios services]# cat check_url.cfg 
define service {

    use generic-service
    host_name web01
    service_description     http_zhudong_url
    check_command       check_url

}
```

> 由于/usr/local/nagios/etc/objects/services/*已经被nagios.cfg主配置文件引用，因此无需在include进service.cfg配置文件。

```
[root@Nagios etc]# cat nagios.cfg | grep "/usr/local/nagios/etc/objects/services"
cfg_file=/usr/local/nagios/etc/objects/services.cfg
cfg_dir=/usr/local/nagios/etc/objects/services
```

**各个配置文件与Nagios.cfg主配置文件的关系如下图所示：**

![QQ截图20170820190632.png-30.3kB](http://static.zybuluo.com/chensiqi/hyf6h17wfd7c40mpyd7n9jt0/QQ%E6%88%AA%E5%9B%BE20170820190632.png)

**（4）重新加载Nagios，查看结果**

```
[root@Nagios etc]# /etc/init.d/nagios reload     
Running configuration check...done.
Reloading nagios configuration...done
```

**（5）查看Nagios服务页面监控结果，如下图所示：**

![QQ截图20170820190940.png-109.9kB](http://static.zybuluo.com/chensiqi/0l0jq3s41xv4hcg7hhzag12g/QQ%E6%88%AA%E5%9B%BE20170820190940.png)

> **备注：**
>  Web01服务器需要能够提供http协议的web访问。
>  等待刷新....

![QQ截图20170820192403.png-96.9kB](http://static.zybuluo.com/chensiqi/yjmsw7ryror75ysi69wp19y1/QQ%E6%88%AA%E5%9B%BE20170820192403.png)

### 9.3.3 利用被动模式的nrpe方式监控/etc/passwd文件是否变化

> Nagios被动模式下的所有插件都需要部署在被监控的Nagios客户端。部署步骤如下。

1）在Nagios客户端web01上取/etc/passwd的文件指纹，即md5值。

```
[root@web01 ~]# md5sum /etc/passwd >/opt/ps.md5
[root@web01 ~]# cat /opt/ps.md5
3660c548ce618df6c066f0db6bedd2af  /etc/passwd       #记住这个校验码
```

2）在Nagios客户端web01上开发插件脚本，并测试

```
#请注意这是在web01客户端的操作
[root@web01 ~]# cd /usr/local/nagios/libexec/
[root@web01 libexec]# vim check_passwd
[root@web01 libexec]# vim check_passwd
[root@web01 libexec]# cat check_passwd 
#!/bin/bash
# author:Mr.chen by 2017-8-20

OriMd5="3660c548ce618df6c066f0db6bedd2af"       #之前记录的校验码
CurrMd5=`md5sum /etc/passwd | cut -c 1-32`      #每次都重新生成校验码
if [ "$OriMd5" == "$CurrMd5" ];then
    echo "/etc/passwd:OK"
    exit 0
else
    echo "/etc/passwd:FAILED"
    exit 2
fi
[root@web01 libexec]# sh check_passwd
/etc/passwd:OK
[root@web01 libexec]# chmod +x check_passwd 

#提示：还可以用md5sum -c /opt/ps.md5的方法比较
```

3）在Nagios客户端web01上编辑nrpe.cfg，插入如下的内容后保存

```
root@web01 libexec]# cd /usr/local/nagios/etc/
[root@web01 etc]# vim nrpe.cfg 
[root@web01 etc]# tail -1 nrpe.cfg      #在文件末尾加入如下内容
command[check_passwd]=/usr/local/nagios/libexec/check_passwd
```

4）在Nagios客户端web01上重启nrpe，并检查是否重启成功（check_nrpe检验）

```
[root@web01 etc]# ps -ef | grep nrpe | grep -v grep
nagios     1027      1  0 Aug18 ?        00:00:05 /usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
[root@web01 etc]# pkill nrpe
[root@web01 etc]# ps -ef | grep nrpe | grep -v grep
[root@web01 etc]# /usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
[root@web01 etc]# ps -ef | grep nrpe | grep -v grep
nagios     4362      1  0 06:33 ?        00:00:00 /usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
```

5）在Nagios服务器端nagios-server上进入service目录，创建配置文件check_passwd_web01.cfg

```
#请注意这里是Nagios服务器端的操作
[root@Nagios ~]# cd /usr/local/nagios/etc/objects/services
[root@Nagios services]# vim check_passwd_web01.cfg
[root@Nagios services]# cat check_passwd_web01.cfg
define service {

    use generic-service
    service_description check_passwd
    check_command check_nrpe!check_passwd
#这里的check_passwd就是Nagios客户端nrpe.cfg里command[check_passwd]=/usr/local/nagios/libexec/check_passwd配置的中括号命令名check_passwd
}
```

6）在Nagios服务器端检查语法

```
[root@Nagios services]# /etc/init.d/nagios checkconfig
#以上省略若干....
Total Warnings: 0
Total Errors:   0

Things look okay - No serious problems were detected during the pre-flight check
OK.
```

7）在Nagios服务器端加载Nagios配置，然后打开Nagios页面查看

```
[root@Nagios services]# /etc/init.d/nagios reload
Running configuration check...done.
Reloading nagios configuration...done
```

![QQ截图20170820195452.png-102.1kB](http://static.zybuluo.com/chensiqi/qzq5crvwcae4cx1jkg7kkmpv/QQ%E6%88%AA%E5%9B%BE20170820195452.png)

**等待刷新....**

![QQ截图20170820195654.png-101.3kB](http://static.zybuluo.com/chensiqi/eese15txxguzydfx0x25h7vp/QQ%E6%88%AA%E5%9B%BE20170820195654.png)

## 十，本节重点回顾

1. Nagios监控系统家族成员功能介绍
2. Nagios监控系统完整框架图解说明
3. Nagios服务器端核心配置文件之间的关系原理
4. Nagios服务器端及客户端安装，配置细节。
5. Nagios利用check_nrpe插件进行监控的原理
6. Nagios图形监控显示和数据管理
7. Nagios报警方式选择及报警实施细节
8. Nagios自定义插件开发原理及开发实践
9. Nagios插件主动和被动方式工作原理及实施部署细节。