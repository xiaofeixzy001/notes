[TOC]

## 一，Nginx基本安全优化

### 1.1 调整参数隐藏Nginx软件版本号信息

> - 一般来说，软件的漏洞都和版本有关，这个很像汽车的缺陷，同一批次的要有问题就都有问题，别的批次可能就都是好的。因此，我们应尽量隐藏或消除Web服务对访问用户显示各类敏感信息（例如Web软件名称及版本号等信息），这样恶意的用户就很难猜到他攻击的服务器所用的是否有特定漏洞的软件，或者是否有对应漏洞的某一特定版本，从而加强Web服务的安全性。这在武侠小说里，就相当于隐身术，你隐身了，对手就很难打着你了。
> - 想要隐身，首先要了解所使用软件的版本号，对于Linux客户端，可通过命令行查看Nginx版本号，最简单的方法就是在Linux客户端系统命令行执行如下curl命令：

```
[root@LNMP html]# curl -I 192.168.0.220
HTTP/1.1 200 OK
Server: nginx/1.6.2             #这里清晰的暴露了Web版本号（1.6.2）及软件名称（nginx）
Date: Wed, 23 Aug 2017 10:45:47 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Powered-By: PHP/5.3.28
Link: <http://192.168.0.220/wp-json/>; rel="https://api.w.org/"
```

**在Windows客户端上，通过浏览器访问Web服务时，若找不到页面，默认报错的信息如下图所示：**

![QQ截图20170825205242.png-31.5kB](http://static.zybuluo.com/chensiqi/0dqg51jlequm269frbaflira/QQ%E6%88%AA%E5%9B%BE20170825205242.png)

> 以上虽然是不同的客户端，但是都获得了Nginx软件名称，而且查到了Nginx的版本号，这就使得Nginx Web服务的安全存在一定的风险，因此，应隐藏掉这些敏感信息或用一个其他的名字将其替代。例如，下面是百度搜索引擎网站Web软件的更名做法：

```
[root@LNMP html]# curl -I baidu.com
HTTP/1.1 200 OK
Date: Fri, 25 Aug 2017 12:22:29 GMT
Server: Apache          #将Web服务软件更名为了Apache，并且版本号也去掉了
[root@LNMP html]# curl -I -s www.baidu.com       
HTTP/1.1 200 OK
Server: bfe/1.0.8.18        #将Web服务软件更名为了bfe，并且版本号改为1.0.8.18(闭源软件名称和版本就无所谓了)
```

> 门户网站尚且如此，我们也学着隐藏或改掉应用服务软件名和版本号把！事实上，还可以通过配置文件加参数来隐藏Nginx版本号。编辑nginx.conf配置文件增加参数，实现隐藏Nginx版本号的方式如下：

```
#在Nginx配置文件nginx.conf中的http标签段内加入“server_tokens off”

http
{
...............
server_tokens off;
...............
}
```

> 此参数放置在http标签内，作用是控制http response header内的Web服务版本信息的显示，以及错误信息中Web服务版本信息的显示。

```
server_tokens参数的官方说明如下：
syntax:     server_tokens on|off;   #此行为参数语法，on为开启状态，off为关闭状态
default:    server_tokens on;       #此行意思是不配置该参数，软件默认情况的结果
context:    http,server,location    #此行为server_tokens参数可以放置的位置参数作用：激活或禁止Nginx的版本信息显示在报错信息和Server的响应首部位置中。
```

官方资料地址：http://nginx.org/en/docs/http/ngx_http_core_module.html

**配置完毕后保存，重新加载配置文件，再次通过curl查看，结果如下：**

```
[root@LNMP nginx]# curl -I 192.168.0.220
HTTP/1.1 200 OK
Server: nginx                   #版本号已经消失
Date: Wed, 23 Aug 2017 11:22:15 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Powered-By: PHP/5.3.28
Link: <http://192.168.0.220/wp-json/>; rel="https://api.w.org/"
```

> 此时，浏览器的报错提示中没有了版本号，如下图所示，修改成功。

![QQ截图20170825205203.png-34.3kB](http://static.zybuluo.com/chensiqi/qiuyx0uycden5bq0qkayids7/QQ%E6%88%AA%E5%9B%BE20170825205203.png)

### 1.2 更改源码隐藏Nginx软件名及版本号

> 隐藏了Nginx版本号后，更进一步，可以通过一些手段把Web服务软件的名称也隐藏起来，或者更改为其他Web服务软件名以迷惑黑客。但软件名字的隐藏修改，一般情况下不会有配置参数和入口，Nginx也不例外，这可能是由于商业及品牌展示等原因，软件提供商不希望使用者把软件名字隐藏起来。因此，此处需要更改Nginx源代码，具体的解决方法如下：

#### 1.2.1 第一步：依次修改3个Nginx源码文件。

修改的第一个文件为nginx-1.6.3/src/core/nginx.h,如下：

```
[root@LNMP ~]# cd /usr/src/nginx-1.6.2/src/core/
[root@LNMP core]# ls -l nginx.h
-rw-r--r--. 1 1001 1001 351 Sep 16  2014 nginx.h
[root@LNMP core]# sed -n '13,17p' nginx.h
#define NGINX_VERSION      "1.6.2"      #修改为想要显示的版本号
#define NGINX_VER          "nginx/" NGINX_VERSION
#将nginx修改为想要修改的软件名称。
#define NGINX_VAR          "NGINX"      #将nginx修改为想要修改的软件名称
#define NGX_OLDPID_EXT     ".oldbin"
```

**修改后的结果如下：**

```
[root@LNMP core]# sed -n '13,17p' nginx.h
#define NGINX_VERSION      "0.0.0.0"
#define NGINX_VER          "yunjisuan/" NGINX_VERSION

#define NGINX_VAR          "YUNJISUAN"
#define NGX_OLDPID_EXT     ".oldbin"
```

**修改的第二个文件是nginx-1.6.3/src/http/ngx_http_header_filter_module.c的第49行，需要修改的字符串内容如下：**

```
ls -l /usr/src/nginx-1.6.2/src/http/ngx_http_header_filter_module.c 
-rw-r--r--. 1 1001 1001 19321 Sep 16  2014 /usr/src/nginx-1.6.2/src/http/ngx_http_header_filter_module.c
[root@LNMP http]# grep -n 'Server: nginx' ngx_http_header_filter_module.c 
49:static char ngx_http_server_string[] = "Server: nginx" CRLF;           #修改本行结尾的nginx
```

**通过sed替换修改，后如下：**

```
[root@LNMP http]# grep -n 'Server: nginx' ngx_http_header_filter_module.c 
49:static char ngx_http_server_string[] = "Server: nginx" CRLF;
[root@LNMP http]# sed -i 's#Server: nginx#Server: yunjisuan#g' ngx_http_header_filter_module.c 
[root@LNMP http]# grep -n 'Server: yunjisuan' ngx_http_header_filter_module.c 
49:static char ngx_http_server_string[] = "Server: yunjisuan" CRLF;
```

**修改的第三个文件是nginx-1.6.3/src/http/nginx_http_special_response.c,对面页面报错时，它会控制是否展开敏感信息。这里输出修改前的信息ngx_http_special_response.c中的第21~30行，如下：**

```
[root@LNMP http]# sed -n '21,30p' ngx_http_special_response.c 
static u_char ngx_http_error_full_tail[] =
"<hr><center>" NGINX_VER "</center>" CRLF       #此行需要修改
"</body>" CRLF
"</html>" CRLF
;

static u_char ngx_http_error_tail[] =
"<hr><center>nginx</center>" CRLF               #此行需要修改
"</body>" CRLF
```

**修改后的结果如下：**

```
[root@LNMP nginx-1.6.2]# sed -n '21,32p' src/http/ngx_http_special_response.c 
static u_char ngx_http_error_full_tail[] =
"<hr><center>" NGINX_VER "  (Mr.chen 2018-08-26)</center>"  CRLF    #此行是定义对外展示的内容
"</body>" CRLF
"</html>" CRLF
;


static u_char ngx_http_error_tail[] =
"<hr><center>yunjisuan</center>" CRLF       #此行将对外展示的Nginx名字更改为yunjisuan
"</body>" CRLF
"</html>" CRLF
;
```

#### 1.2.2 第二步是修改后编辑软件，使其生效

> 修改后再编译安装软件，如果是已经安装好的服务，需要重新编译Nginx，配好配置，启动服务。
>  再次使浏览器出现404错误，然后看访问结果，如下图所示：

![QQ截图20170826205843.png-33.2kB](http://static.zybuluo.com/chensiqi/pwllei3z2nwsysxv5185wpvc/QQ%E6%88%AA%E5%9B%BE20170826205843.png)

> 如上面所示：Nginx的软件和版本名都被改掉了，并且加上了本人的大名。
>  再看看Linux curl命令响应头部信息，如下：

```
[root@LNMP conf]# curl -I localhost/xxx/
HTTP/1.1 404 Not Found
Server: yunjisuan/0.0.0.0           #也更改了
Date: Wed, 23 Aug 2017 15:33:54 GMT
Content-Type: text/html
Content-Length: 196
Connection: keep-alive
```

### 1.3 更改Nginx服务的默认用户

> - 为了让Web服务更安全，要尽可能地改掉软件默认的所有配置，包括端口，用户等。
> - 下面就来更改Nginx服务的默认用户
> - 首先，查看Nginx服务对应的默认用户。一般情况下，Nginx服务启动后，默认使用的用户是nobody，查看默认的配置文件，如下：

```
[root@LNMP conf]# cd /usr/local/nginx/conf/
[root@LNMP conf]# grep "#user" nginx.conf.default
#user  nobody;
```

> 为了防止黑客猜到这个Web服务的用户，我们需要更改成特殊的用户名，例如nginx或特殊点的inca，但是这个用户必须是系统里事先存在的，下面以nginx用户为例进行说明。

**（1）为Nginx服务建立新用户**

```
useradd nginx -s /sbin/nologin -M
#不需要有系统登录权限，应当禁止登陆。
```

**（2）配置Nginx服务，让其使用刚建立的nginx用户**

> 更改Nginx服务默认使用用户，方法有二：

**第一种**：直接更改配置文件参数，将默认的#user nobody；改为如下内容：

```
user nginx nginx;
```

> 如果注释或不设置上述参数，默认为nobody用户，不推荐使用nobody用户名，最好采用一个普通用户，此处用大家习惯的，前面建立好的nginx用户。

**第二种：**直接在编译nginx软件时指定编译的用户和组，命令如下：

```
./configure --user=nginx --group=nginx --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module

#提示：
#前文在编译Nginx服务时，就是这样带着参数的，因此无论配置文件中是否加参数，默认都是nginx用户。
```

**（3）检查更改用户的效果**

> 重新加载配置后，检查Nginx服务进程的对应用户，如下：

```
[root@LNMP conf]# ps -ef | grep nginx | grep -v grep
root      52023      1  0 11:30 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx     52024  52023  0 11:30 ?        00:00:00 nginx: worker process      
```

> 通过查看上述更改后的Nginx进程，可以看到worker processes进程对应的用户都变成了nginx。所以，我们有理由得出结论，上述的两种方法都可设置Nginx的worker进程运行的用户。当然，Nginx的主进程还是以root身份运行的。

## 二，根据参数优化Nginx服务性能

### 2.1 优化Nginx服务的worker进程个数

> - 在高并发，高访问量的Web服务场景，需要事先启动好更多的Nginx进程，以保证快速响应并处理大量并发用户的请求。
> - 这类似于开饭店，在营业前，需要事先招聘一定数量的服务员准备接待顾客，但这里就有一个问题，如果饭店对客流没有正确预估，就会导致一些问题发生，例如：服务员人数招聘多了，但是客流很少，那么服务员就可能很闲，没事干，饭店的成本也高了；如果客流很大，而服务员人数少了，可能就接待不过来顾客，导致顾客吃饭体验差。因此，饭店要根据客户的流量及并发量来调整接待的服务人员数量，然后根据检测顾客量变化及时调整到最佳配置。
> - Nginx服务就相当于饭店，网站用户就相当于顾客，Nginx的进程就相当于服务员，下面就来优化Nginx进程的个数。

#### 2.1.1 优化Nginx进程对应的配置

```
#优化Nginx进程对应Nginx服务的配置参数如下：
worker_processes 1;     #指定了Nginx要开启的进程数，结尾数字就是进程个数
```

> 上述参数调整的是Nginx服务的worker进程数，Nginx有Master进程和worker进程之分，Master为管理进程，真正接待“顾客”的是worker进程。

#### 2.1.2 优化Nginx进程个数的策略

> - 前面已经讲解过，worker_processes参数大小的设置最好和网站的用户数量相关联，可如果是新配置，不知道网站的用户数量该怎么办呢？
> - 搭建服务器时，worker进程数最开始的设置可以等于CPU的核数，且worker进程数要多一些，这样起始提供服务时就不会出现因为访问量快速增加而临时启动新进程提供服务的问题，缩短了系统的瞬时开销和提供服务的时间，提升了服务用户的速度。高流量高并发场合也可以考虑将进程数提高至CPU核数*2，具体情况要根据实际的业务来选择，因为这个参数除了要和CPU核数匹配外，也和硬盘存储的数据及系统的负载有关，设置为CPU的核数是一个好的起始配置，这也是官方的建议。

#### 2.1.3 查看Web服务器CPU硬件资源信息

下面介绍查看Linux服务器CPU总核数的方法：

（1）通过/proc/cpuinfo可查看CPU个数及总核数。查看CPU总核数的示例如下：

```
[root@LNMP ~]# grep processor /proc/cpuinfo 
processor   : 0
processor   : 1
processor   : 2
processor   : 3
[root@LNMP ~]# grep processor /proc/cpuinfo | wc -l
4               #表示为1颗CPU四核
[root@LNMP ~]# grep -c processor /proc/cpuinfo
4               #表示为1颗CPU四核

#查看CPU总颗数示例如下：
[root@LNMP ~]# grep "physical id" /proc/cpuinfo 
physical id : 0     #物理ID一致，同一颗CPU
physical id : 0     #物理ID一致，同一颗CPU
physical id : 0     #物理ID一致，同一颗CPU
physical id : 0     #物理ID一致，同一颗CPU
[root@LNMP ~]# grep "physical id" /proc/cpuinfo | sort | uniq | wc -l
1               #去重复，表示1颗CPU
```

（2）通过执行top命令，然后按数字1，即可显示所有的CPU核数，如下：

![QQ截图20170827152136.png-52.9kB](http://static.zybuluo.com/chensiqi/0trr07e4wglx8virg7sp6f1n/QQ%E6%88%AA%E5%9B%BE20170827152136.png)

#### 2.1.4 实践修改Nginx配置

> 假设服务器的CPU颗数为1颗，核数为4核，则初始的配置可通过查看默认的nginx.conf里的worker_processes数来了解，命令如下:

```
[root@LNMP ~]# grep worker_processes /usr/local/nginx/conf/nginx.conf
worker_processes  1;
[root@LNMP ~]# sed -i 's#worker_processes  1#worker_processes  4#' /usr/local/nginx/conf/nginx.conf
[root@LNMP ~]# grep worker_processes /usr/local/nginx/conf/nginx.conf
worker_processes  4;        #提示可以通过vi修改

#优雅重启Nginx，使修改生效，如下：
[root@LNMP ~]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@LNMP ~]# /usr/local/nginx/sbin/nginx -s reload

#现在检查修改后的worker进程数量，如下：
[root@LNMP ~]# ps -ef | grep "nginx" | grep -v grep
root       1110      1  0 11:12 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx      1429   1110  0 11:33 ?        00:00:00 nginx: worker process      
nginx      1430   1110  0 11:33 ?        00:00:00 nginx: worker process      
nginx      1431   1110  0 11:33 ?        00:00:00 nginx: worker process      
nginx      1432   1110  0 11:33 ?        00:00:00 nginx: worker process      
```

> 从“worker_processes 4”可知，worker进程数为4个。Nginx Master主进程不包含在这个参数内，Nginx Master的主进程为管理进程，负责调度和管理worker进程。

**有关worker_processes参数的官方说明如下：**

```
syntax:         worker_processes number;    #此行为参数语法，number为数量
default:        worker_processes 1;     #此行意思是不匹配该参数，软件默认情况数量为1
context:        main;   #此行为worker_processes参数可以放置的位置
```

> worker_processes为定义worker进程数的数量，建议设置为CPU的核数或CPU核数*2，具体情况要根据实际的业务来选择，因为这个参数，除了要和CPU核数匹配外，和硬盘存储的数据以系统的负载也有关，设置为CPU的个数或核数是一个好的起始配置。From：http://nginx.org/en/docs/ngx_core_module.html

### 2.2 优化绑定不同的Nginx进程到不同的CPU上

> - 默认情况下，Nginx的多个进程有可能跑在某一个CPU或CPU的某一核上，导致Nginx进程使用硬件的资源不均，本节的优化是尽可能地分配不同的Nginx进程给不同的CPU处理，达到充分有效利用硬件的多CPU多核资源的目的。
> - 在优化不同的Nginx进程对应不同的CPU配置时，四核CPU服务器的参数配置参考如下：

```
worker_processes 4;
worker_cpu_affinity 0001 0010 0100 1000;

#worker_cpu_affinity就是配置Nginx进程与CPU亲和力的参数，即把不同的进程分给不同的CPU处理。这里0001 0010 0100 1000是掩码，分别代表第1，2，3，4核CPU，由于worker_processes进程数为4，因此，上述配置会把每个进程分配一核CPU处理，默认情况下进程不会绑定任何CPU，参数位置为main段。
```

**四核和八核CPU服务器的参数配置参考如下：**

```
#八核掩码
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000；

#四核掩码
worker_cpu_affinity 0001 0010 0100 1000；
```

> worker_cpu_affinity 的作用是绑定不同的worker进程数到一组CPU上。通过设置bitmask控制进程允许使用的CPU，默认worker进程不会绑定到任何CPU（自动平均分配。）

#### 2.2.1 实验环境准备

| 主机名  | IP地址        | 备注             |
| ------- | ------------- | ---------------- |
| Nginx   | 192.168.0.220 | nginxWeb         |
| 测试机1 | 192.168.0.240 | Webbench压力测试 |
| 测试机2 | 192.168.0.245 | Webbench压力测试 |

```
#安装webbench
tar xf webbench-1.5.tar.gz 
cd webbench-1.5
mkdir /usr/local/man
make install clean
which webbench.
```

**虚拟机开启4核心**

![QQ截图20170827184049.png-7.5kB](http://static.zybuluo.com/chensiqi/kfx8nxnj5v7t40n3s1fhsop0/QQ%E6%88%AA%E5%9B%BE20170827184049.png)

#### 2.2.2 第一步：不绑定worker进程进行压力测试

```
#配置文件如下：（未绑定worker进程）
[root@LNMP nginx]# cat conf/nginx.conf
worker_processes  4;
#worker_cpu_affinity 0001 0010 0100 1000;
events {
    worker_connections  10240;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  bbs.yunjisuan.com;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}

#在NginxWeb上执行如下命令：
[root@LNMP nginx]# top -u nginx
Tasks: 120 total,   1 running, 119 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.0%us,  0.2%sy,  0.0%ni, 99.8%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   1004412k total,   911632k used,    92780k free,     6952k buffers
Swap:  2031608k total,        0k used,  2031608k free,   749976k cached

   PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                                                                 
  1454 nginx     20   0 49240 5640  468 S  0.0  0.6   0:00.00 nginx                                                                                    
  1455 nginx     20   0 49240 5672  500 S  0.0  0.6   0:00.00 nginx                                                                                    
  1456 nginx     20   0 49240 5672  500 S  0.0  0.6   0:00.00 nginx                                                                                    
  1457 nginx     20   0 49240 5672  500 S  0.0  0.6   0:00.00 nginx  
  

#在以上界面时按键盘的数值1键，出现如下界面：
top - 14:44:46 up 36 min,  1 user,  load average: 0.00, 0.00, 0.00
Tasks: 120 total,   1 running, 119 sleeping,   0 stopped,   0 zombie
Cpu0  :  0.0%us,  0.0%sy,  0.0%ni,100.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Cpu1  :  0.0%us,  0.0%sy,  0.0%ni,100.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Cpu2  :  0.0%us,  0.0%sy,  0.0%ni,100.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Cpu3  :  0.0%us,  0.0%sy,  0.0%ni,100.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   1004412k total,   911384k used,    93028k free,     6960k buffers
Swap:  2031608k total,        0k used,  2031608k free,   749976k cached

   PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                                                                 
  1454 nginx     20   0 49240 5640  468 S  0.0  0.6   0:00.00 nginx                                                                                    
  1455 nginx     20   0 49240 5672  500 S  0.0  0.6   0:00.00 nginx                                                                                    
  1456 nginx     20   0 49240 5672  500 S  0.0  0.6   0:00.00 nginx                                                                                    
  1457 nginx     20   0 49240 5672  500 S  0.0  0.6   0:00.00 nginx      
```

**在另外的两台测试机器上同时进行压力测试，命令如下：**
 `webbench -c 2000 -t 60 http://192.168.0.220/`

**结果如下：**

![QQ截图20170827185601.png-36.8kB](http://static.zybuluo.com/chensiqi/6hepz5i8txvaw3tde5qo6c4k/QQ%E6%88%AA%E5%9B%BE20170827185601.png)

#### 2.2.3 第二步：绑定worker进程进行压力测试

```
#配置文件如下（绑定worker进程）
[root@LNMP nginx]# cat conf/nginx.conf
worker_processes  4;
worker_cpu_affinity 0001 0010 0100 1000;    #修改本行
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  bbs.yunjisuan.com;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

**在另外的两台测试机器上同时进行压力测试，命令如下：**
 `webbench -c 2000 -t 60 http://192.168.0.220/`

**结果如下：**

![QQ截图20170827190217.png-32.6kB](http://static.zybuluo.com/chensiqi/eti4pr6n9oys0cz04t2l4vxn/QQ%E6%88%AA%E5%9B%BE20170827190217.png)

> 根据图示，我们基本可以看出，平均绑定worker进程和不绑定的实验效果基本是一致的（CPU0是默认会被使用的）。原因在nginx在经过不断的优化后，会自动对worker进程进行动态的平均分配。

#### 2.2.4 第三步：修改nginx配置，将所有worker进程绑定到CPU3上

```
#配置文件如下所示：
[root@LNMP nginx]# cat conf/nginx.conf
worker_processes  4;
worker_cpu_affinity 1000 1000 1000 1000;    #修改本行
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  bbs.yunjisuan.com;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

**在另外的两台测试机器上同时进行压力测试，命令如下：**
 `webbench -c 2000 -t 60 http://192.168.0.220/`

**结果如下：**

![QQ截图20170827190657.png-20.5kB](http://static.zybuluo.com/chensiqi/nnkpcqphpvh8t9ggzeljnpvn/QQ%E6%88%AA%E5%9B%BE20170827190657.png)

> 从上图我们可以得知，worker进程的压力被集中分配到了CPU3上。（CPU0是默认被使用的）

### 2.3 Nginx事件处理模型优化

> - Nginx的连接处理机制在不同的操作系统会采用不同的额I/O模型，在Linux下，Nginx使用epoll的I/O多路复用模型，在Freebsd中使用kqueue的I/O多路复用模型，在Solaris中使用/dev/poll方式的I/O多路复用模型，在Windows中使用的是icop，等等。
> - 要根据系统类型选择不同的事件处理模型，可供使用的选择有“use [kqueue|rtsig|epoll|/dev/poll|select|poll];”。因为教学使用的是CentOS 6.5 Linux，因此将Nginx的事件处理模型调整为epoll模型。

```
#具体的配置参数如下：
events      #events指令是设定Nginx的工作模式及连接数上限
{
    use epoll；     #use是一个事件模块指令，用来指定Nginx的工作模式。Nginx支持的工作模式有select，poll，kqueue，epoll，rtsig和/dev/poll。其中select和poll都是标准的工作模式，kqueue和epoll是高效的工作模式，不同的是epoll用在Linux平台上，而kqueue用在BSD系统中。对于Linux系统Linux2.6+内核，推荐选择epoll工作模式，这是高性能高并发的设置
}
```

> 根据Nginx官方文档建议，也可以不指定事件处理模型，Nginx会自动选择最佳的事件处理模型服务。
>  对于使用连接进程的方法，通常不需要进行任何设置，Nginx会自动选择最有效办法。

### 2.4 调整Nginx单个进程允许的客户端最大连接数

> 接下来，调整Nginx单个进程允许的客户端最大连接数，这个控制连接数的参数为work_connections。
>  worker_connections的值要根据具体服务器性能和程序的内存使用量来指定（一个进程启动使用的内存根据程序确定），如下：

```
events  #events指令是设定Nginx的工作模式和连接数上线
{
    worker_connections 20480;
    #worker_connections也是个事件模块指令，用于定义Nginx每个进程的最大连接数，默认是1024.最大客户端连接数由worker_processes和worker_connections决定，即Max_client=  worker_processes*worker_connections。进程的最大连接数受Linux系统进程的最大打开文件数限制，在执行操作系统命令“ulimit   -HSn    65535”或配置相应文件后，worker_connections的设置才能生效。
}
```

**下面是worker_connections的官方说明**

参数语法：worker_connections number
 默认配置：worker_connections 512
 放置位置：events

**说明：**

> worker_connections用来设置一个worker  process支持的最大并发连接数，这个连接数包括了所有链接，例如：代理服务器的连接，客户端的连接等，实际的并发连接数除了受worker_connections参数控制外，还和最大打开文件数worker_rlimit_nofile有关(见下文)，Nginx总并发连接=worker数量*worker_connections。
>  参考资料：http://nginx.org/en/docs/ngx_core_module.html

### 2.5 配置Nginx worker进程最大打开文件数

> 接下来，调整配置Nginx worker进程的最大打开文件数，这个控制连接数的参数为worker_rlimit_nofile。该参数的实际配置如下：

```
worker_rlimit_nofile 65535;
#最大打开文件数，可设置为系统优化后的ulimit     -HSn的结果
```

**下面是worker_rlimit_nofile number的官方说明：**

参数语法：worker_rlimit_nofile number
 默认配置：无
 放置位置：主标签段

> **说明**：此参数的作用是改变worker processes能打开的最大文件数
>  参考资料：http://nginx.org/en/docs/ngx_core_module.html
>
> **备注：**
>  Linux系统文件最大打开数设置：ulimit -n 65535

#### 2.5.1 实验环境准备

| 主机名  | IP地址        | 备注             |
| ------- | ------------- | ---------------- |
| Nginx   | 192.168.0.220 | nginxWeb         |
| 测试机1 | 192.168.0.240 | Webbench压力测试 |
| 测试机2 | 192.168.0.245 | Webbench压力测试 |

#### 2.5.2 修改nginx.conf配置文件

```
[root@LNMP nginx]# cat conf/nginx.conf
worker_processes  1;
#worker_cpu_affinity 0000 0010 0100 1000;
#worker_rlimit_nofile 65535;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  bbs.yunjisuan.com;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

> 先定1核1024连接数。同学们注意多开几个虚拟机进行压力测试。不然的话，web服务器还没出问题，你测试服务器先down掉了。

#### 2.5.2 测试nginx服务连接数的极值

```
#使用这个命令可以抓取nginx的连接数
[root@LNMP nginx]# netstat -antp | grep nginx | wc -l
554
[root@LNMP nginx]# netstat -antp | grep nginx | wc -l
471
```

> 逐渐提高压力，抓连接数，看看nginx啥时候down

### 2.6 开启高效文件传输模式

（1）设置参数：sendfile on;

> sendfile参数用于开启文件的高效传输模式。同时将tcp_nopush和tcp_nodelay两个指令设置为on，可防止网络及磁盘I/O阻塞，提升Nginx工作效率。

```
#sendfile参数的官方说明如下：
syntax:     sendfile on | off;   #参数语法
default:    sendfile off;        #参数默认大小
context:    http,server,location,if in location #可以放置的标签段
```

> **参数作用：**
>  激活或禁用sendfile()功能功能。sendfile()是作用于两个文件描述符之间的数据拷贝函数，这个拷贝操作是在内核之中的，被称为“零拷贝”，sendfile()比read和write函数要高效很多，因为，read和write函数要把数据拷贝到应用层再进行操作。相关控制参数还有sendfile_max_chunk,同学们可以执行查询。细节见http://nginx.org/en/docs/http/ngx_http_core_module.html#sendfile

（2）设置参数：tcp_nopush on;

```
#tcp_nopush参数的官方说明如下：
syntax:     tcp_nopush on | off;    #参数语法
default:    tcp_nopush off;         #参数默认大小
context:    http,server,location    #可以放置的标签段
```

> **参数作用：**
>  激活或禁用Linux上的TCP_CORK  socket选项，此选项仅仅当开启sendfile时才生效，激活这个tcp_nopush参数可以允许把http response  header和文件的开始部分放在一个文件里发布，其积极的作用是减少网络报文段的数量。细节见http://nginx.org/en/docs/http/ngx_http_core_module.html。

### 2.7 优化Nginx连接参数，调整连接超时时间

#### 2.7.1 什么是连接超时

> - 先来个比喻吧，某饭店请了服务员招待顾客，但是现在饭店不景气，此时，为多余的服务员发工资使得成本被提高，想减少饭店开支成本就得解雇服务员。
> - 这里的服务员就相当于Nginx服务建立的连接，当服务器建立的连接没有接收处理请求时，可在指定的时间内就让它超时自动退出。还有当Nginx和FastCGI服务建立连接请求PHP时，如果因为一些原因（负载高，停止响应），FastCGI服务无法给Nginx返回数据，此时可以通过配置Nginx服务参数使其不会死等，因为前面用过户还等着它返回数据呢，例如，可设置为如果请求FastCGI 10秒内不能返回数据，那么Nginx就中断本次请求，向用户汇报取不到数据的错误。

#### 2.7.2 连接超时的作用

- 将无用的连接设置为尽快超时，可以保护服务器的系统资源（CPU，内存，磁盘）。
- 当连接很多时，及时断掉那些已经建立好的但又长时间不做事的连接，以减少其占用的服务器资源，因为服务器维护连接也是消耗资源的。
- 有时黑客或恶意用户攻击网站，就会不断地和服务器建立多个连接，消耗连接数，但是啥也不干，大量消耗服务器的资源，此时就应该及时断掉这些恶意占用资源的连接。
- LNMP环境中，如果用户请求了动态服务，则Nginx就会建立连接，请求FastCGI服务以及后端MySQL服务，此时这个Nginx连接就要设定一个超时时间，在用户容忍的时间内返回数据，或者再多等一会儿后端服务返回数据，具体的策略要根据具体业务进行具体分析。当然了，后端的FastCGI服务及MySQL服务也有对连接的超时控制。

> 简单的说，连接超时是服务的一种自我管理，自我保护的重要机制。

#### 2.7.3 连接超时带来的问题，以及不同程序连接设定知识

> - 服务器建立新连接也是要消耗资源的，因此，超时设置得太短而并发很大，就会导致服务器瞬间无法响应用户的请求，导致用户体验下降。
> - 企业生产有些PHP程序站点会希望设置成短连接，因为PHP程序建立连接消耗的资源和时间相对要少些。而对于Java程序站点来说，一般建议设置长连接，因为Java程序建立连接消耗的资源和时间更多，这是语言运行机制决定的。

#### 2.7.4 Nginx连接超时的参数设置

（1）设置参数：keepalive_timeout 60;

> 用于设置客户端连接保持会话的超时时间为60秒。超过这个时间，服务器会关闭该连接，此数值为参考值。

```
keepalive_timeout参数的官方说明如下：
syntax: keepalive_timeout  timeout [header_timeout] #参数语法
default: keepalive_timeout 75s;     #参数默认大小
context: http,serverr,location      #可以放置的标签段
```

> **参数作用：**

> - keep-alive可以使客户端到服务器端已经建立的连接一直工作不退出，当服务器有持续请求时，keep-alive会使用已经建立的连接提供服务，从而避免服务器重新建立新连接处理请求。
> - 此参数设置一个keep-alive(客户端连接在服务器端保持多久后退出)，其单位是秒，和HTTP响应header域的“Keep-Alive:timeout=time”参数有关，这些header信息也会被客户端浏览器识别并处理，不过有些客户端并不能按照服务器端的设置来处理，例如：MSIE大约60秒后会关闭keep-alive连接。细节见：http://nginx.org/en/docs/http/ngx_http_core_module.html

（2）设置参数：tcp_nodelay on;

> 用于激活tcp_ondelay功能，提高I/O性能。

```
#tcp_nodelay参数的官方说明如下：
syntax:     tcp_nodelay on | off    #参数语法
default:    tcp_nodelay on;         #参数默认大小
context:    http,server,location    #可以放置的标签段
```

> **参数作用：**
>  默认情况下当数据发送时，内核并不会马上发送，可能会等待更多的字节组成一个数据包，这样可以提高I/O性能。但是，在每次只发送很少字节的业务场景中，使用tcp_nodelay功能，等待时间会比较长。
>  **参数生效条件：**
>  激活或禁用TCP_NODELAY选项，当一个连接进入keep-alive状态时生效。细节见http://nginx.org/en/docs/http/ngx_http_core_module.html。

（3）设置参数：client_header_timeout 15;

> 用于设置读取客户端请求头数据的超时时间。此处的数值15，其单位是秒，为经验参考值。

```
#client_header_timeout参数的官方说明如下：
syntax:     client_header_timeout time; #参数语法
default:    client_header_timeout 60s;  #参数默认大小
context:    http,server         #可以放置的标签段
```

> **参数作用：**
>  设置读取客户端请求头数据的超时时间。如果超过这个时间，客户端还没有发送完整的header数据，服务器端将返回“Request time out  (408)”错误，可指定一个超时时间，防止客户端利用http协议进行攻击。细节见：http://nginx.org/en/docs/http/ngx_http_core_module.html。

（4）设置参数：client_body_timeout 15;

> 用于设置读取客户端请求主体的超时时间，默认值60

```
#client_body_timeout参数的官方说明如下：
syntax:     client_body_timeout time;   #参数语法
default:    client_body_timeout 60s;    #默认60
context:    http,server,location    #可以放置的标签段
```

> **参数作用：**
>  设置读取客户端请求主体的超时时间。这个超时仅仅为两次成功的读取操作之间的一个超时，非请求整个主体数据的超时时间，如果在这个超时时间内，客户端没有发送任何数据，Nginx将返回“Request time  out(408)”错误，默认值60，细节见:http://nginx.org/en/docs/http/ngx_http_core_module.html

（5）设置参数：send_timeout 25;

> 用于指定响应客户端的超时时间。这个超时仅限于两个连接活动之间的时间，如果超过这个时间，客户端没有任何活动，Nginx将会关闭连接，默认值为60秒，可以改为参考值25秒。

```
#send_timeout参数的官方说明如下：
syntax:     send_timeout    time;   #参数语法
default:    send_timeout    60s;    #默认值60
context:    http,server,location    #可以放置的标签段
```

> **参数作用：**
>  设置服务器端传送HTTP响应信息到客户端的超时时间，这个超时仅仅为两次成功握手后的一个超时，非请求整个响应数据的超时时间，如在这个超时时间内，客户端没有接收任何数据，连接将被关闭。细节见http://nginx.org/en/docs/http/ngx_http_module.html。

![QQ截图20170827232141.png-614kB](http://static.zybuluo.com/chensiqi/sifcj85hqg28f9zb2bxncy3p/QQ%E6%88%AA%E5%9B%BE20170827232141.png)

```
备注：结合http原理画图讲解超时参数
```

### 2.8 上传文件大小的限制（动态应用）

> 下面我们学习如何调整上传文件的大小（http Request body size）限制。

```
#首先，我们可以在Nginx主配置文件里加入如下参数：
client_max_body_size 8m;

#具体大小根据公司的业务做调整，如果不清楚就先设置为8m把，有关客户端请求主体的解释在HTTP原理一节已经解释过了，一般情况下，HTTP的post方法在提交数据时才会携带请求主体信息。
#client_max_body_size参数的官方说明如下：
syntax:     client_max_body_size size;  #参数语法
default:    client_max_body_size 1m;    #默认值1m
context：   http,server,location        #可以放置的1标签段
```

> **参数作用：**
>  设置最大的允许的客户端请求主体大小，在请求头域有“Content-Length”，如果超过了此配置值，客户端会受到413错误，意思是请求的条目过大，有可能浏览器不能正确显示。设置为0表示禁止检查客户端请求主体大小。此参数对提高服务器端的安全性有一定作用。细节见http://nginx.org/en/docs/http/ngx_http_core_module.html

### 2.9 FastCGI相关参数调优（配合PHP引擎动态服务）

> FastCGI参数是配合Nginx向后请求PHP动态引擎服务的相关参数。Nginx FastCGI工作的逻辑图如下图所示：

![QQ截图20170828004930.png-77.1kB](http://static.zybuluo.com/chensiqi/4r56hbdaj9i6tylu07thu3ju/QQ%E6%88%AA%E5%9B%BE20170828004930.png)

> 此处讲解的参数均为Nginx FastCGI客户端向后请求PHP动态引擎服务（php-fpm（FastCGI服务器端））的相关参数，属于Nginx的配置参数。下表是Nginx FastCGi常见参数的说明。

![QQ截图20170828005612.png-445.9kB](http://static.zybuluo.com/chensiqi/hectq0iqy25np6w1ai3ihk0c/QQ%E6%88%AA%E5%9B%BE20170828005612.png)

![QQ截图20170828005628.png-208.8kB](http://static.zybuluo.com/chensiqi/p908uurvhb8pw65vmlif25ce/QQ%E6%88%AA%E5%9B%BE20170828005628.png)

FastCGI Cache资料见http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache。

**FastCGI常见参数的Nginx配置示例如下：**

```
[root@LNMP nginx]# cat /usr/local/nginx/conf/nginx.conf
worker_processes  4;
worker_cpu_affinity 0001 0010 0100 1000;
worker_rlimit_nofile 65535;
user nginx;
events {
    use epoll;
    worker_connections  10240;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    tcp_nopush      on;
    keepalive_timeout  65;
    tcp_nodelay     on;
    client_header_timeout 15;
    client_body_timeout   15;
    send_timeout          15;
    log_format main '$remote_addr - $remote_user [$time_local] "$request"'
    '$status $body_bytes_sent "$http_referer"'
    '"$http_user_agent""$http_x_forwarded_for"';
    server_tokens off;
    fastcgi_connect_timeout 240;        #Nginx允许fcgi连接超时时间
    fastcgi_send_timeout 240;           #Nginx允许fcgi返回数据的超时时间
    fastcgi_read_timeout 240;           #Nginx读取fcgi响应信息的超时时间
    fastcgi_buffer_size 64k;            #Nginx读取响应信息的缓冲区大小
    fastcgi_buffers 4 64k;              #指定Nginx缓冲区的数量和大小
    fastcgi_busy_buffers_size 128k;     #当系统繁忙时buffer的大小
    fastcgi_temp_file_write_size 128k;  #Nginx临时文件的大小
#   fastcgi_temp_path   /data/ngx_fcgi_tmp; #指定Nginx临时文件放置路径
    fastcgi_cache_path  /data/ngx_fcgi_cache    levels=2:2  keys_zone=ngx_fcgi_cache:512m   inactive=1d;    #指定Nginx缓存放置路径
    
# web

    server {
        listen       80;
        server_name  www.yunjisuan.com;
    location / {
        root html;
        index index.php index.html index.htm;
        if (-f $request_filename/index.html){
            rewrite (.*) $1/index.html break;
        }
        if (-f $request_filename/index.php){
            rewrite (.*) $1/index.php;
        }
        if (!-f $request_filename){
            rewrite (.*) /index.php;
        }
    }
    access_log  logs/web_www_access.log     main;
        location ~ .*\.(php|php5)?$ {
                root   html;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        include fastcgi.conf;
        fastcgi_cache ngx_fcgi_cache;       #开启fcgi缓存并起名叫ngx_fcgi_cache,很重要，有效降低CPU负载，并且防止502错误发生。
        fastcgi_cache_valid 200 302 1h; #指定应答代码的缓存时间，1h=1小时
        fastcgi_cache_valid 301 1d;     #1d=1天
        fastcgi_cache_valid any 1m;     #and 1m：将其他应答缓存1分钟
        fastcgi_cache_min_uses 1;       #待缓存内容至少要被用户请求过1次
        fastcgi_cache_use_stale error timeout invalid_header http_500;  #当遇到error，timeout，或者返回码500时，启用过期缓存返回用户（返回过期也比返回错误强）
#       fastcgi_cache_key   http://$host$request_uri;   
        
        }
    }
    upstream    www_yunjisuan {

    server 192.168.0.225:8000 weight=1;

    }
    server {
        
    listen 8000;
    server_name www.yunjisuan.com;
    location / {
#       root    html;
        proxy_pass      http://www_yunjisuan;
        proxy_set_header    host    $host;
        proxy_set_header    x-forwarded-for $remote_addr;

    }
    access_log  logs/proxy_www_access.log   main;
    }
}
```

![QQ截图20170829203733.png-161.9kB](http://static.zybuluo.com/chensiqi/1ysz3wxapjcf1qmar8byup02/QQ%E6%88%AA%E5%9B%BE20170829203733.png)

> Nginx的FastCGI的相关参数和反向代理proxy的相关参数非常接近，同学们可以拿来比对，一起理解。

### 2.10 配置Nginx gzip压缩实现性能优化

#### 2.10.1 Nginx gzip压缩功能介绍

> Nginx gzip压缩模块提供了压缩文件内容的功能，用户请求的内容在发送到用户客户端之前，Nginx服务器会根据一些具体的策略实施压缩，以节约网站出口带宽，同时加快数据传输效率，来提升用户访问体验。

#### 2.10.2 Nginx gzip压缩的优点

- 提升网站用户体验：发送给用户的内容小了，用户访问单位大小的页面就加快了，用户体验提升了，网站口碑就好了。
- 节约网站带宽成本：数据是压缩传输的，因此节省了网站的带宽流量成本，不过压缩时会稍微消耗一些CPU资源，这个一般可以忽略。

> 此功能既能提升用户体验，又能使公司少花钱，一举多得。对于几乎所有的Web服务来说，这是一个非常重要的功能，Apache服务也有此功能。

#### 2.10.3 需要和不需要压缩的对象

- 纯文本内容压缩比很高，因此，纯文本的内容最好进行压缩，例如：html，js，css，xml，shtml等格式的文件。
- 被压缩的纯文本文件必须要大于1KB，由于压缩算法的特殊原因，极小的文件压缩后可能反而变大。
- 图片，视频（流媒体）等文件尽量不要压缩，因为这些文件大多都是经过压缩的，如果再压缩很可能不会减少或减少很少，或者有可能增大，同时压缩时还会消耗大量的CPU，内存资源。

#### 2.10.4 参数介绍及配置说明

> 此压缩功能与早期Apache服务的mod_deflate压缩功能很相似，Nginx的gzip压缩功能依赖于ngx_http_gzip_module模块，默认已安装。

**对应的压缩参数说明如下：**

```
#######压缩的配置介绍######
gzip on；
#开启gzip压缩功能
gzip_min_length 1k;
#设置允许压缩的页面最小字节数，页面字节数从header头的Content-Length中获取。默认值0，表示不管页面多大都进行压缩。建议设置成大于1K，如果小于1K可能会越压越大。
gzip_buffers 4 16K;
#压缩缓冲区大小。表示申请4个单位为16K的内存作为压缩结果流缓存，默认值是申请与原始数据大小相同的内存空间来存储gzip压缩结果。
gzip_http_version 1.1;
#压缩版本（默认1.1，前端为squid2.5时使用1.0），用于设置识别HTTP协议版本，默认是1.1，目前大部分浏览器已经支持GZIP解压，使用默认即可。
gzip_comp_level 2;
#压缩比率。用来指定gzip压缩比，1压缩比最小，处理速度最快；9压缩比最大，传输速度快，但处理最慢，也比较消耗CPU资源。
gzip_types text/plain application/x-javascript text/css application/xml;
#用来指定压缩的类型，“text/html”类型总是会被压缩，这个就是HTTP原理部分讲的媒体类型。
gzip_vary on;
#vary header支持。该选项可以让前端的缓存服务器缓存经过gzip压缩的页面，例如用Squid缓存经过Nginx压缩的数据。
```

> 不同的Nginx版本中，gzip_types的配置可能会有不同，上述配置示例适合Nginx 1.6.3。对应的文件类型，请查看安装目录下的mime.types文件。

**更多官方资源请看http://nginx.org/en/docs/http/ngx_http_gzip_module.html**

#### 2.10.5 Nginx压缩配置效果检查

> 可通过火狐浏览器+firebug插件+yslow插件查看gzip压缩及expires缓存结果。提前安装好yslow插件，开启监控，然后打开LNMP时安装的博客地址，就可以看到如下图所示的压缩结果：

![QQ截图20170830002722.png-456kB](http://static.zybuluo.com/chensiqi/g2af4cc5ghq7ogka88ri9uyo/QQ%E6%88%AA%E5%9B%BE20170830002722.png)

#### 2.10.6 重要的前端网站调试工具介绍

**常见的前端网站调试工具有如下几种：**

- Google浏览器（Chrome）：通过该浏览器直接按F12键即可查看压缩及缓存结果，另外，谷歌浏览器（Chrome）上也可以直接安装yslow插件
- 火狐浏览器：在该浏览器上安装firebug，yslow，即可进行调试（火狐要用老版本比如V28）
- IE浏览器：在该浏览器上安装httpwatch即可进行调试（省略）

### 2.11 配置Nginx expires缓存实现性能优化

#### 2.11.1 Nginx expires功能介绍

> - 简单说，Nginx  expires的功能就是为用户访问的网站内容设定一个过期时间，当用户第一次访问这些内容时，会把这些内容存储在用户浏览器本地，这样用户第二次及以后继续访问该网站时，浏览器会检查加载已经缓存在用户浏览器本地的内容，就不会去服务器下载了，直到缓存的内容过期或被清除位置。
> - 更深入的理解：expires的功能就是允许通过Nginx配置文件控制HTTP的“Expires”和“Cache-Control”响应头部内容，告诉客户端浏览器是否缓存和缓存多久以内访问的内容。这个expires模块控制Nginx服务器应答时的expires头内容和Cache-Control头的max-age指令。缓存的有效期可以设置为相对于源文件的最后修改时刻或客户端的访问时刻。
> - 这些HTTP头向客户端表明了额内容的有效性和持久性。如果客户端本地有内容缓存，则内容就可以从缓存而不是从服务器中读取，然后客户端会检查缓存中的副本，看其是否过期或失效，以决定是否重新从服务器获得内容更新。

#### 2.11.2 Nginx expires作用介绍

> 在网站的开发和运营中，视频，图片，CSS，JS等网站元素的更改机会较少，特别是图片，这时可以将图片设置在客户浏览器本地缓存365天或3650天，而将CSS，JS，html等代码缓存10~30天。这样用户第一次打开页面后，会在本地的浏览器按照过期日期缓存相应的内容，下次用户再打开类似的页面时，重复的元素就无需下载了，从而加快用户访问速度。用户的访问请求和数据减少了，也可节省大量的服务器端带宽。此功能同Apache的expires功能类似。

#### 2.11.3 Nginx expires功能优点

- expires可以降低网站的带宽，节约成本。
- 加快用户访问网站的速度，提升用户访问体验。
- 服务器访问量降低了，服务器压力就减轻了，服务器成本也会降低，甚至可以节约人力成本。
- 对于几乎所有的Web服务来说，这是非常重要的功能之一，Apache服务也有此功能。

#### 2.11.4 Nginx expires配置详解

> 前面已经介绍了expires的功能原理，接下来就来配置Nginx  expires的功能。这里以location标签为例进行讲解，通过location  URI规则将需要缓存的扩展名列出来，然后指定缓存时间。如果针对所有内容设置缓存，也可以不用location。Nginx默认安装了expires功能。

**（1）根据文件扩展名进行判断，添加expires功能范例**

**范例1：**

```
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
{
    expires 3650d;
}
```

> 该范例的意思是当用户访问网站URL结尾的文件扩展名为上述指定类型的额图片时，设置缓存3650天，即10年。

**范例2：**

```
location ~ .*\.(js|css)$
 {
    expires 30d;
 }
```

> 该范例的意思是当用户访问网站URL结尾的文件扩展名为js，css类型的元素时，设置缓存30天，即1个月。

**（2）根据URL中的路径（目录）进行判断，添加expires功能范例**

**范例3：**

```
location ~ ^/(images|javascript|js|css|flash|media|static)/
{
    expires 360d;
}
```

> 该范例的意思是当用过户访问网站URL中包含上述路径（例：images，js，css，这些在服务器端是程序目录）时，把访问的内容设置缓存360天，即1年。

#### 2.11.5 Nginx expires配置效果检查

> 检查Nginx expires的方法和检查Nginx gzip的方法相同。
>  通过火狐浏览器加yslow插件查看gzip压缩及expires缓存结果时，要提前安装好火狐浏览器，并且要安装好yslow插件，开启监控，然后打开LNMP时安装的博客地址（带有图片，JS，CSS），就可以看到如下图所示的缓存结果了。

![QQ截图20170830191330.png-11.7kB](http://static.zybuluo.com/chensiqi/yc54zbs21vja3g5ucu9hf146/QQ%E6%88%AA%E5%9B%BE20170830191330.png)

**在Linux客户端可通过如下curl命令查看图片URL的缓存header信息：**

```
[root@localhost ~]# curl -I 192.168.0.220:8000
HTTP/1.1 200 OK
Server: nginx
Date: Wed, 30 Aug 2017 02:15:54 GMT
Content-Type: text/html
Content-Length: 18
Connection: keep-alive
Last-Modified: Tue, 29 Aug 2017 19:37:44 GMT
ETag: "59a5c288-12"
Expires: Sat, 09 Sep 2017 02:15:54 GMT      #缓存的过期时间
Cache-Control: max-age=864000               #缓存的总时间，单位秒
Accept-Ranges: bytes
```

#### 2.11.6 Nginx expires功能缺点及解决方法

> - 几乎所有的事物都是有两面性，没有十全十美的人和事。Nginx expires功能也不例外，虽然这个功能很好，但是也会给企业带来一些困惑。
> - 当网站被缓存的页面或数据更新了，此时用户端看到的可能还是旧的已经缓存的内容，这样就会影响用户体验，那么如何解决这个问题呢？解决方法如下。
> - 第一，对于经常需要变动的图片等文件，可以缩短对象缓存时间，例如：谷歌和百度的首页图片经常根据不同的日期换成一些节日的图，所以这里可以将这个图片设置为缓存期1天。
> - 第二，当网站改版或更新时，可以在服务器将缓存的对象改名（网站代码程序）。
> - 对于网站的图片，附件，一般不会被用户直接修改，用户层面上的修改图片，实际上是重新传到服务器，虽然内容一样但是是一个新的图片名了。
> - 网站改版升级会修改JS，CSS元素，若改版时对这些元素改了名，会使得前端的CDN及用户端需要重新缓存内容。

#### 2.11.7 企业网站缓存日期曾经的案例参考

> 若企业的业务和网站访问量不同，那么网站的缓存期时间设置也是不同的，比如，如下企业所用的缓存日期就是不一样的。

- 51CTO：1周
- 新浪：15天
- 京东：25年
- 淘宝：10年

#### 2.11.8 企业网站有可能不希望被缓存的内容

- 广告图片，用于广告服务，都缓存了就不好控制展示了。
- 网站流量统计工具（JS代码），都缓存了流量统计就不准了。
- 更新很频繁的文件（google的logo），这个如果按天，缓存效果还是显著的。

## 三，Nginx日志相关优化与安全

### 3.1 编写脚本实现Nginx access日志轮询

> 当用户请求一个软件时，绝大多数软件都会记录用户的访问情况，Nginx服务也不例外。Nginx软件目前还没有类似Apache的通过cronolog或rotatelog对日志分割处理的功能，但是，运维人员可以利用脚本开发，Nginx的信号控制功能或reload重新加载，来实现日志的自动切割，轮询。

**日志切割脚本如下：**

```
[root@localhost nginx]# cat /server/scripts/cut_nginx_log.sh 
#!/bin/bash
#日志切割脚本可挂定时任务，每天00点整执行

Dateformat=`date +%Y%m%d`
Basedir="/usr/local/nginx"
Nginxlogdir="$Basedir/logs"
Logname="access"

[ -d $Nginxlogdir ] && cd $Nginxlogdir || exit 1
[ -f ${Logname}.log ] || exit 1
/bin/mv ${Logname}.log ${Dateformat}_${Logname}.log
$Basedir/sbin/nginx -s reload

[root@localhost nginx]# cat >>/var/spool/cron/root << KOF
#cut nginx access log by Mr.chen
00 00 * * * /bin/bash /server/scripts/cut_nginx_log.sh >/dev/null 2>&1
```

### 3.2 不记录不需要的访问日志

> 在实际工作中，对于负载均衡器健康节点检查或某些特定文件（比如图片，JS，CSS）的日志，一般不需要记录下来，因为在统计PV时是按照页面计算的，而且日志写入太频繁会消耗大量磁盘I/O，降低服务的性能。

**具体配置方法如下：**

```
location ~ .*\.(js|jpg|JPG|jpeg|JPEG|css|bmp|gif|GIF)$
{
    access_log  off;
}
#这里用location标签匹配不记录日志的元素扩展名，然后关闭日志
```

### 3.3 访问日志的权限设置

**假如日志目录为/app/logs,则授权方法如下：**

```
chown -R root.root /app/logs
chmod -R 700 /app/logs
```

> 不需要在日志目录上给Nginx用户读或写许可，但很多网友都没注意这个问题，他们把该权限直接给了Nginx或Apache用户，这就成为了安全隐患。

