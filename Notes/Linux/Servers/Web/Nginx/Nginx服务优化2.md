[TOC]

## 四，Nginx站点目录及文件URL访问控制

### 4.1 根据扩展名限制程序和文件访问

> - Web2.0时代，绝大多数网站都是以用户为中心多的，例如：bbs，blog，sns产品，这几个产品都有一个共同特点，就是不但允许用户发布内容到服务器，还允许用户发图片甚至上传附件到服务器上，由于为用户开了上传功能，因此给服务器带来了很大的安全风险。虽然很多程序在上传前会着一定的控制，例如：文件大小，类型等，但是，一不小心就会被黑客钻了控制，上传了木马程序。
> - 下面将利用Nginx配置禁止访问上传资源目录下的PHP，Shell，Perl，Python程序文件，这样用户即使上传了木马文件也没法执行，从而加强了网站的安全。

**范例1：**配置Nginx，禁止解析指定目录下的指定程序。

```
location ~ ^/images/.*\.(php|php5|sh|pl|py)$
{
    deny all;
}

location ~ ^/static/.*\.(php|php5|sh|pl|py)$
{
    deny all;
}
location ~* ^/data/(attachment|avatar)/.*\.(php|php5)$
{
    deny all;
}
#对上述目录的限制必须写在Nginx处理PHP服务配置的前面，如下：
location ~ .*\.(php|php5)$
{
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    include fcgi.conf;
}
```

**范例2：**Nginx下配置禁止访问*.txt和*.doc文件。

**实际配置信息如下：**

```
location ~* \.(txt|doc)$
{
    if (-f $request_filename)
    {
        root /data/www/www;
        #rewrite ...可以重定向到某个URL
        break；
    }
}
location ~* \.(txt|doc)$
{
    root /data/www/www;
    deny all;
}
```

### 4.2 禁止访问指定目录下的所有文件和目录

**范例1：**配置禁止访问指定的单个或多个目录

```
#禁止访问单个目录的命令如下：
location ~ ^/static
{
    deny all;
}

#禁止访问多个目录的命令如下：
location ~ ^/(static|js)
{
    deny all;
}
```

**范例2：**禁止访问目录并返回指定的HTTP状态码，命令如下：

```
server 
{
    listen 80;
    server_name www.yunjisuan.com yunjisuan.com;
    root /data/www/www;
    index index.html index.htm;
    access_log logs/www_access.log commonlog;
    location /admin/
    {
        return 404;
    }
    location /tmplates/
    {
        return 403;
    }
}
```

> **作用：**
>  禁止访问目录下的指定文件1，或者禁止访问指定目录下的所有内容。
>  **最佳应用场景：**
>  对于集群的共享存储，一般是存放静态资源文件，所以，可禁止执行指定扩展名的程序，例：.php,.sh,.pl,.py

### 4.3 限制网站来源IP访问

**下面介绍如何使用ngx_http_access_module限制网站来源IP访问**

**案例环境：phpmyadmin数据库的Web客户端，内部开发人员用的。**

**范例1：**禁止某目录让外界访问，但允许某IP访问该目录，且支持PHP解析，命令如下：

```
location ~ ^/yunjisuan/ 
{
    allow 202.111.12.211;
    deny all;
}

location ~ .*\.(php|php5)$
{
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index   index.php;
    include         fastcgi.conf;
}
```

**范例2：**限制指定IP或IP段访问，命令如下：

```
location / 
{
    deny    192.168.1.1;
    allow 192.168.1.0/24;
    allow 10.1.1.0/16;
    deny all;
}
```

**参考资料：http://nginx.org/en/docs/http/ngx_http_access_module.html**

> **企业问题案例：** Nginx做反向代理的时候可以限制客户端IP吗？

答：可以，具体方法如下：

**方法一：**使用if来控制，命令如下：

```
if ($remote_addr = 10.0.0.7)
{
    return 403;
}

if ($remote_addr = 218.247.17.130)
{
    set $allow_access_root 'ture';      #我也不知道什么意思
}
```

**方法二：**利用deny和allow只允许IP访问，命令如下：

```
location / {
    root html/blog;
    index index.php index.html index.htm;
    allow 10.0.0.7;
    deny all;
}
```

**方法三：**只拒绝某些IP访问，命令如下：

```
location / {
    root html/blog;
    index index.php index.html index.htm;
    deny 10.0.0.7;
    allow all;
}
```

**注意事项：**

- deny一定要加一个IP，否则会直接跳转到403，不再往下执行了，如果403默认页是在同一域名下，会造成死循环访问。
- 对于allow的IP段，从允许访问的段位从小到大排列，如127.0.0.0/24的下面才能是10.10.0.0/16，其中：
- 24表示子网掩码：255.255.255.0
- 16表示子网掩码：255.255.0.0
- 8表示子网掩码：255.0.0.0
- 以deny all:结尾，表示除了上面允许的，其他的都禁止。如：

```
deny 192.168.1.1;
allow 127.0.0.0/24;
allow 192.168.0.0/16;
allow 10.10.0.0/16;
deny all;
```

### 4.4 配置Nginx，禁止非法域名解析访问企业网站

> 问题：Nginx如何防止用户IP访问网站（恶意域名解析，也相当于是直接IP访问企业网站）

**方法一：**让使用IP访问网站的用户，或者恶意解析域名的用过户，收到501错误，命令如下：

```
server {
    listen 80 default_server;
    server_name _;
    return 501;
}

#说明：直接报501错误，从用户体验上不是很好
```

**方法二：**通过301跳转到主页，命令如下：

```
server {
    listen 80 default_server;
    server_name _;
    rewrite ^(.*) http://www.yunjisuan.com/$1 permanent;
}
```

**方法三：**发现某域名恶意解析到公司的服务器IP，在server标签里添加以下代码即可，若有多个server则要多处添加。

```
if ($host ! ~ ^www\.yunjisuan\.com$)
{
    rewrite ^(.*) http://www.yunjisuan.com/$1 permanent;
}

#说明：代码含义为如果header信息的host主机名字非www.yunjisuan.com，就301跳转到www.yunjisuan.com
```

## 五，Nginx图片及目录防盗链解决方案

### 5.1 什么是资源盗链

> 简单的说，就是某些不法网站未经允许，通过在其自身网站程序里非法调用其他网站的资源，然后在自己的网站上显示这些调用的资源，达到填充自身网站的效果。这一举动不仅浪费了调用资源网站的网络流量，还造成其他网站的带宽及服务压力吃紧，甚至宕机。

**下面通过示意图阐述资源被盗链原理，如下图所示：**

![QQ截图20170831214823.png-89.1kB](http://static.zybuluo.com/chensiqi/vto31ijmqcoocghv8ahlf99d/QQ%E6%88%AA%E5%9B%BE20170831214823.png)

### 5.2 网站资源被盗链带来的问题

> 若网站图片及相关资源被盗链，最直接的影响就是网络带宽占用加大了，带宽费用多了，网络流量也可能忽高忽低，Nagios/Zabbix等报警服务频繁报警，类似下图所示：

![QQ截图20170831215104.png-187.4kB](http://static.zybuluo.com/chensiqi/rhfh85k7e5qr6d3blobiskll/QQ%E6%88%AA%E5%9B%BE20170831215104.png)

> 最严重的情况就是网站的资源被非法使用，使网站带宽成本加大和服务器压力加大，这有可能导致数万元的损失，且网站的正常用户访问也会受到影响。

### 5.3 企业真实案例：网站资源被盗链，出现严重问题

> 某日，接到从事运维工作的朋友的紧急求助，其公司的CDN源站，源站的流量没有变动，但CDN加速那边的流量无故超了好几个GB，不知道怎么处理。
>  该故障的影响：由于是购买的CDN网站加速服务，因此虽然流量多了几个GB，但是业务未受影响。只是，这么大的异常流量，持续下去可直接导致公司无故损失数万元。解决这个问题可体现运维的价值。

**那么，这样的问题如何及时发现，又如何处理呢？**

**第一：**对IDC及CDN带宽做监控报警
 **第二：**作为高级运维或运维经理，每天上班的重要任务，就是经常查看网站流量图，关注流量变化，关注异常流量。
 **第三：**对访问日志做分析，迅速定位异常流量，并且和公司市场推广等保持较好的沟通，以便调度带宽和服务器资源，确保网站正常的访问体验。

### 5.4 常见防盗链解决方案的基本原理

（1）根据HTTP referer实现防盗链

> - 在HTTP协议中，有一个表头字段叫referer，使用URL格式来表示是哪里的链接用了当前网页的资源。通过referer可以检测访问的来源网页，如果是资源文件，可以跟踪到显示它的网页地址，一旦检测出来源不是本站，马上进行阻止或返回指定的页面。
> - HTTP  referer是header的一部分，当浏览器向Web服务器发送请求时，一般会带上referer，告诉服务器我是从哪个页面链接过来的，服务器借此获得一些信息用于处理。Apache，Nginx，Lighttpd三者都支持根据HTTP referer实现防盗链，referer是目前网站图片，附件，html等最常用的防盗链手段。下图是referer防盗链的基本原理图。

![QQ截图20170901200511.png-157.1kB](http://static.zybuluo.com/chensiqi/21981dvekizczo6k9mz959ba/QQ%E6%88%AA%E5%9B%BE20170901200511.png)

（2）根据cookie防盗链

> - 对于一些特殊的业务数据，例如流媒体应用通过ActiveX显示的内容（例如，Flash，Windows  Media视频，流媒体的RTSP协议等），因为他们不向服务器提供referer  header，所以若采用上述的referer的防盗链手段，就达不到想要的效果。
> - 对于Flash，Windows Media视频这种占用流量较大的业务数据，防盗链是比较困难的，此时可以采用Cookie技术，解决Flash，Windows Media视频等的防盗链问题。

例如：ActiveX插件不传递referer，但会传递Cookie，可以在显示ActiveX的页面的``标签内嵌入一段JavaScript代码，设置“Cookie：Cache=av”如下：

```
<script>document.cookie="Cache=av;domain=domain.com;path=/";</script>
```

然后就可以通过各种手段来判断这个Cookie的存在，以及验证其值的操作了。
 根据Cookie来防盗链的技术比较复杂，我们不在这里涉及，同学们如果感兴趣，或者企业确实需要，可通过网络来了解相关方法。

（3）通过加密变换访问路径实现防盗链

> 此种方法比较适合视频及下载类业务数据的网站。例如：Lighttpd有类似的插件mod_secdownload来实现此功能。先在服务器端配置此模块，设置一个固定用于加密的字符串，比如yunjisuan，然后设置一个url前缀，比如/mp4/，再设置一个过期时间，比如1小时，然后写一段PHP代码，利用加密字符串和系统时间等通过md5算法生成一个加密字符串。最终获取到的文件的URL链接中会带有一个时间戳和一个加密字符的MD5数值，在访问时系统会对这两个数据进行验证。如果时间不在预期的时间段内（如1小时内）则失效；如果时间戳符合条件，但是加密的字符串不符合条件也会失效，从而达到防盗链的效果。

### 5.5 Nginx Web服务实现防盗链实战

> 在默认情况下，只需要进行简单的配置，即可实现防盗链处理。请看下面的实例。

（1）利用referer，并且针对扩展名rewrite重定向

```
#下面的代码为利用referer且针对扩展名rewrite重定向，即实现防盗链的Nginx配置。
location ~* \.(jpg|gif|png|swf|flv|wma|wmv|asf|mp3|mmf|zip|rar)$
{
    valid_referers none blocked *.yunjisuan.com yunjisuan.com;  #
    if ($invalid_referer)
    {
        rewrite ^/ http://www.yunjisuan.com/img/nolink.jpg;
    }
}
#提示：要根据自己公司的实际业务（是否有外链的合作）进行域名设置。
```

（2）利用referer，并且针对站点目录过滤返回错误码

**针对目录的方法如下：**

```
location /images {
    root /data/www/www;
    valid_referers none blocked *.yunjisuan.com yunjisuan.com;
    if ($invalid_referer) {
        return403;
    }
}
```

> 在上面这段防盗链设置中，分别针对不同文件类型和不同的目录进行了设置，同学们可以根据自己需求进行类似设定。下面是上述代码的说明：

### 5.6 NginxHttpAccessKeyModule实现防盗链介绍

> - 如果不怕麻烦，有条件实现的话，推荐使用NginxHttpAccessKeyModule。
> - 其运行方式是：如download目录下有一个file.zip文件。对应的URI是http：//www.abc.com/download/file.zip,使用ngx_http_accesskey_module  模块后就成了http://www.bac.com/download/file.zipkey=09093abeac094,只有正确地给定了key值，才能下载download目录下的file.zip，而且key值是与用户的IP相关的，这样就可以避免被盗链了。据说，现在NginxHttpAccessKeyModule连迅雷都可以防了，同学们执行尝试一下。

### 5.7 在产品设计上解决盗链方案

> 产品设计时，处理盗链问题可将计就计，为网站上传的图片增加水印。例如：下图就是为网站上传的图片增加的水印。

![QQ截图20170901211858.png-23kB](http://static.zybuluo.com/chensiqi/u1gfhbf1wuhz35tdsqspvuup/QQ%E6%88%AA%E5%9B%BE20170901211858.png)

> 为图片添加版权水印是很有效的方法。网站直接转载图片一般是为了快捷，但是对于有水印的图片，很多站长是不愿意转载的。

## 六，Nginx错误页面的优雅显示

### 6.1 生产环境常见的HTTP状态码列表

![QQ截图20170901213041.png-357.1kB](http://static.zybuluo.com/chensiqi/xcjhyrxctsndfcvvbmk4t4iw/QQ%E6%88%AA%E5%9B%BE20170901213041.png)

### 6.2 为什么要配置错误页面优雅显示

> 在网站的运行过程中，可能因为页面不存在或系统过载等原因，导致网站无法正常响应用户的请求，此时Web服务会返回系统默认的错误码，或者很不友好的页面，如下图所示：

![QQ截图20170901213712.png-32.9kB](http://static.zybuluo.com/chensiqi/lvvdzks3iuuqfw705ai01nxt/QQ%E6%88%AA%E5%9B%BE20170901213712.png)

> 我们可以将404，403等的错误信息页面重定向到网站首页或其他事先指定的页面，提升网站的用户访问体验。

**范例1：**对错误代码403实行本地页面跳转，命令如下：

```
server {
    listen 80;
    server_name     www.yunjisuan.com;
    location / {
        root html/www;
        index   index.html  index.htm;
    }
    error_page  403 /403.html;      #当出现403错误时，会跳转到403.html页面
}

#上面的/403.html是相对于站点根目录html/www的。
```

**范例2：**对错误代码404实行本地页面优雅显示，命令如下：

```
server {
    listen  80;
    server_name www.yunjisuan.com;
    location / {
        root    html/www;
        index   index.html  index.htm;
        error_page  404 /404.html;
        #当出现404错误时，会跳转到404.html页面
    }
}
#代码中的/404.html是相对于站点根目录html/www的
```

**范例3：** 50x页面放到本地单独目录下，进行优雅显示

```
error_page  500 502 503 504 /50x.html;
location = /50x.html {
    root    /data/www/html;
}
#这里指定单独的站点目录存放到50x.html文件中。
```

**范例4：** 错误状态码URL重定向，命令如下：

```
server {
    listen 80;
    server_name www.yunjisuan.com;
    location / {
        root    html/www;
        index   index.html  index.htm;
        error_page  404 http://bbs.yunjisuan.com;
        #当出现404错误时，会跳转到指定的URL http://bbs.yunjisuan.com页面显示给用户，这个URL一般是企业另外的可用地址。
        access_log  /usr/local/nginx/logs/bbs_access.log    commonlog;
    }
}
#代码中的/404.html是相对于站点根目录html/www的。
```

**范例5：** 将错误状态码重定向到一个location，命令如下：

```
location / {
    error_page  404 = @fallback;
}
location @fallback {
    proxy_pass  http://backend;
}
```

### 6.3 阿里门户网站天猫的Nginx优雅显示配置案例如下：

```
error_page  500 501 502 503 504 http://err.tmall.com/error2.html;
error_page  400 403 404 405 408 410 411 412 413 414 415 http://err.tmall.com/error1.html;
```

## 七，Nginx站点目录文件及目录权限优化

### 7.1 单机LNMP环境目录权限严格控制措施

> 为了保证网站不遭受木马入侵，所有站点目录的用户和组都应该为root，所有的目录权限是755；所有的文件权限是644.设置如下：

```
[root@LNMP nginx]# ls -l /usr/local/nginx/html/ | tail -5
-rw-r--r--.  1 root root  8048 Jan 11  2017 wp-mail.php
-rw-r--r--.  1 root root 16255 Apr  6 14:23 wp-settings.php
-rw-r--r--.  1 root root 29896 Oct 19  2016 wp-signup.php
-rw-r--r--.  1 root root  4513 Oct 14  2016 wp-trackback.php
-rw-r--r--.  1 root root  3065 Aug 31  2016 xmlrpc.php
```

> - 以上的权限设置可以防止黑客上传木马，以及修改站点文件，但是，合理的网站用户上传的内容也会被拒之门外。那么如何让合法的用户可以上传文件，而又不至于被黑客利用攻击呢？
> - 如果是单机的LNMP环境，站点目录和文件属性设置如下。
> - 先把所有的目录权限设置为755，所有的文件权限设置为644，所用的目录，以及文件用户和组都是root；然后把用户上传资源的目录权限设置为755，将用户和组设置为Nginx服务的用户；最后针对上传资源的目录做资源访问限制。
> - 部分公司所采用的授权方式不是很安全，常见的有如下两种：

```
chmod -R 777 /directory
chmod -R nginx.nginx /directory
```

> - 上述两种授权方法虽然不能说错误，但是没有做到授权最小化，会给网站带来非常大的安全隐患，特别是木马入侵的时候。
> - 在比较好的网站业务架构中，应把资源文件，包括用户上传的图片，附件等服务和程序服务分离，最好把上传程序服务也分离出来，这样就可以从容地按照之前所述进行安全授权了。

### 7.2 Nginx企业网站集群超级安全设置

> 结合Linux权限体系及Nginx大型集群架构进行配置，严格控制针对Nginx目录的访问才能降低网站被入侵的风险。比如，可根据下图中的企业集群架构逻辑图和不同角色提供的不同服务来严格控制不同服务器的Nginx目录权限。

![QQ截图20170901234255.png-157.2kB](http://static.zybuluo.com/chensiqi/50v3o7id276oolt8wf7clp54/QQ%E6%88%AA%E5%9B%BE20170901234255.png)

**下图为集群架构中不同于前面Web业务的权限管理细化。**

| 服务器角色     | 权限处理                                                     | 安全系数                                                     |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 动态Web集群    | 目录权限755，文件权限644，所用目录，以及文件用户和组都是root。环境为Nginx+PHP | 文件不能被改，目录不能被写入，安全系数10                     |
| static图片集群 | 目录权限755，文件权限644，所用的目录，以及文件用户和组都是root。环境为Nginx | 文件不能被改，目录不能被写入，安全系数10                     |
| 上传upload集群 | 目录权限755，文件权限644，所用的目录，以及文件用户和组都是root。特别：用户上传的目录设置为755，用户和组使用Nginx服务配置的用户 | 文件不能被改，目录不能被写入，但是用户上传的目录允许写入文件且需要通过Nginx的其他功能来禁止读文件，安全系数8 |

> 做到上述的设置后，网站服务在系统层面被入侵的风险就大大降低了。

## 八，Nginx防爬虫优化

### 8.1 robots.txt机器人协议常识

> - Robots协议（也称为爬虫协议，机器人协议等）的全称是“网络爬虫排除标准”(Robots Exclusion Protocol),网站通过Robots协议告诉搜索引擎哪些页面可以抓取，哪些页面不能抓取。
> - 我理解的是robots.txt是通过代码控制搜索引擎蜘蛛索引的一个手段，以便减轻网站服务器的带宽使用率，从而让网站的空间更稳定，同时也可以提高网站其他页面的索引效率，提高网站收录。
> - 我们只需要创建一个robots.txt文本文件，然后在文档内设置好代码，告诉搜索引擎我网站的哪些文件你不能访问。然后上传到网站根目录下面，因为当搜索引擎蜘蛛在索引一个网站时，会先爬行查看网站根目录下是否有robots.txt文件。

### 8.2 机器人协议起源

> - 2011年10月25日，京东商城正式将一淘网的搜索爬虫屏蔽，以防止一淘网对其内容进行抓取。

**京东的robots.txt设置如下：**
 https://www.jd.com/robots.txt

![QQ截图20170902000613.png-27.6kB](http://static.zybuluo.com/chensiqi/h148ufzpl8xpzk90djr1cer4/QQ%E6%88%AA%E5%9B%BE20170902000613.png)

> - 2008年9月8日，淘宝网宣布封杀百度爬虫，百度忍痛遵守爬虫协议。因为一旦破坏协议，用户的隐私和利益就无法得到保障，搜索网站就谈不上人性关怀。

**淘宝的robots.txt设置如下：**
 https://www.taobao.com/robots.txt

![QQ截图20170902000647.png-41.3kB](http://static.zybuluo.com/chensiqi/zkjow05wsbb1ulazd6hbhwyy/QQ%E6%88%AA%E5%9B%BE20170902000647.png)

> - 2012年8月，360综合搜索被指违反robots协议。

**360的robots.txt设置如下：**
 http://www.360.cn/robots.txt

![QQ截图20170902000824.png-27.4kB](http://static.zybuluo.com/chensiqi/2xr87oumgjpvxiw9e1zi01e3/QQ%E6%88%AA%E5%9B%BE20170902000824.png)

### 8.3 Nginx防爬虫优化配置

> 我们可以根据客户端的user-agents信息，轻松地阻止指定的爬虫爬取我们的网站。下面来看几个案例。

**范例1：**阻止下载协议代理，命令如下：

```
##Block download agents##
if ($http_user_agent ~* LWP:Simple | BBBike | wget) 
{
    return 403;
}

#说明：如果用户匹配了if后面的客户端（例如wget），就返回403.
```

> 这里根据$http_user_agent获取客户端agent，然后判断是否允许或返回指定错误码。

**范例2：**添加内容防止N多爬虫代理访问网站，命令如下：

```
#这些爬虫代理使用“|”分隔，具体要处理的爬虫可以根据需求增加或减少，添加的内容如下：
if ($http_user_agent ~* "qihoobot|Baiduspider|Googlebot-Modile|Googlebot-Image|Mediapartners-Google|Adsbot-Google|Yahoo! SSlurp  China|YoudaoBot|Sosospider|Sogou spider|Sogou web spider|MSNBot")
{
    return 403;
}
```

**范例3：**测试禁止不同的浏览器软件访问

**示例代码如下：**

```
if ($http_user_agent ~* "Firefox|MSIE")
{
    rewrite ^(.*) http://www.yunjisuan.com/$1 permanent;
}

#如果浏览器为Firefox或IE，就会跳转到http://www.yunjisuan.com
```

## 九，利用Nginx限制HTTP的请求方法

> 在之前的HTTP协议原理一节中，讲解了很多HTTP方法，其中最常用的HTTP方法为GET，POST，我们可以通过Nginx限制HTTP请求的方法来达到提升服务器安全的目的，例如，让HTTP只能使用GET，HEAD和POST方法的配置如下：

```
#Only allow these request methods
if ($request_method ! ~ ^(GET|HEAD|POST)$)
{
    return 501;
}
```

> 当上传服务器上传数据到存储服务器时，用户上传写入的目录就不得不给Nginx对应的用户相关权限，这样一旦程序有漏洞，木马就有可能被上传到服务器挂载的对应存储服务器的目录里，虽然我们也做了禁止PHP，SH，PL，PY等扩展名的解析限制，但还是会遗漏一些想不到的可执行文件。对于这样情况，该怎么办呢？事实上，还可以通过限制上传服务器的Web服务（可以具体到文件）使用GET方法，防止用户通过上传服务器访问存储内容，让访问存储渠道只能从静态或图片服务器入口进入。例如，在上传服务器上限制HTTP的GET方法的配置如下：

```
#Only deny GET request methods ##
if ($request_method ~* ^(GET)$)
{
    return 501;
}
#提示：还可以加一层location，更具体地限制文件名
```

**实际效果如下图所示：**

![QQ截图20170902210551.png-30kB](http://static.zybuluo.com/chensiqi/j1ogmdjrf3n3dt9zepuugj6i/QQ%E6%88%AA%E5%9B%BE20170902210551.png)

## 十，使用CDN做网站内容加速

### 10.1 什么是CDN

> - CDN的全称是Content Delivery  Network，中文意思是内容分发网络。简单地讲，通过在现有的Internet中增加一层新的网络架构，将网站的内容发布到最接近用户的Cache服务器内，通过智能DNS负载均衡技术，判断用户的来源，让用户就近使用与服务器相同线路的带宽访问Cache服务器，取得所需的内容。例如：天津网通用户访问天津网通Cache服务器上的内容，北京电信访问北京电信Cache服务器上的内容。这样可以有效减少数据在网络上传输的时间，提高访问速度。
> - CDN是一套全国或全球的1分布式缓存集群，其实质是通过智能DNS判断用户的来源地域及上网线路，为用户选择一个最接近用户地域，以及和用户上网线路相同的服务器节点，因为地域近，且线路相同，所以，可以大幅提升用户浏览网站的体验。
> - CDN产生背景之一：BGP机房虽然可以提升用户体验，但是价格昂贵，对于用户来说，CDN的诞生可以提供比BGP机房更好的体验（让同一地区，同一线路的用户访问和当地同一线路的网站），BGP机房和普通机房有将近5~10倍的价格差。CDN多使用单线的机房，根据用户的线路及位置，为用户选择靠近用户的位置，以及相同的运营商线路，不但提升了用户体验，价格也降下来了。

- [x] CDN的价值：
  - 为架设网站的企业省钱
  - 提升企业网站的用户访问体验（相同线路，相同地域，内存访问）
  - 可以阻挡大部分流量攻击，例如：DDOS攻击

### 10.2 CDN的特点

- [x] CDN就是一个具备根据用户区域和线路智能调度的分布式内存缓存集群。其特点如下：
  - 通过服务器内存缓存网站数据，提高了企业站点（尤其含有大量图片，视频等的站点）的访问速度，并大大提高企业站点的稳定性（省钱且提升用户体验）。
  - 用户根据智能DNS技术自动选择最适合的Cache服务器，降低了不同运营商之间互联瓶颈造成的影响，实现了跨运营商的网络加速，保证不同网络中的用户都能得到良好的访问质量。
  - 加快了访问速度，减少了原站点的带宽
  - 用户访问时从服务器的内存中读取数据，分担了网络流量，同时减轻了原站点负载压力等。
  - 使用CDN可以分担源站的网络流量，同时可以减轻原站点的负载压力，并降低黑客入侵及各种DDOS攻击对网站的影响，保证网站有较好的服务质量。

**下面是通过curl命令访问163网站的header信息，可以看到163网站的首页就使用了CDN进行加速。**

```
[root@LNMP logs]# curl -I www.163.com
HTTP/1.1 200 OK
Expires: Sat, 02 Sep 2017 13:53:55 GMT
Date: Sat, 02 Sep 2017 13:52:35 GMT
Server: nginx
Content-Type: text/html; charset=GBK
Transfer-Encoding: chunked
Vary: Accept-Encoding,User-Agent,Accept
Cache-Control: max-age=80
Age: 34
X-Via: 1.1 fzhwtxz25:6 (Cdn Cache Server V2.0), 1.1 wangtong42:0 (Cdn Cache Server V2.0)
Connection: keep-alive
```

> 上面的“（CdnCacheServerV2.0）”表示163首页使用了CDN加速。

### 10.3 企业使用CDN的基本要求

> 首先要说的是，不是所有的网站都可以一上来就能用CDN的。要加速的业务数据应该存在独立的域名，例如：img1-4.yunjisuan.com/video1-4.yunjisuan.com，业务内容图片，附件，JS，CSS等静态元素，这样的静态网站域名才可以使用CDN。

**下面来看一个DNS解析范例。DNS服务器加速前的A记录如下：**

```
;A records
img.yunjisuanl.com IN A 124.106.0.21 (企业服务器的IP)

#删除上面的记录，命令如下：
img.yunjisuanl.com IN A 124.106.0.21 (服务器的IP)

#然后，做下面的别名解析：
；CNAME records
img.yunjisuan.com IN CNAME bbs
img.yunjisuan.com 3M IN CNAME img.yunjisuan.com.cachecn.com.
```

> **提示：**
>  这个img.yunjisuan.com.cachecn.com.地址必须是事先由CDN公司配置好的CDN公司的域名。国内较大的CDN提供商为网宿，蓝讯，快网。

## 十一，Nginx程序架构优化

> 解耦是开发人员中流行的一个名词，简单地说就是把一堆程序代码按照业务用途分开，然后提供服务，例如：注册登录，上传，下载，浏览列表，商品内容页面，订单支付等都应该是独立的程序服务，只不过在客户端看来是一个整体而已。如果中小公司做不到上述细致的解耦，起码也要让下面的几个程序模块独立。

- 网页页面服务。（静态，动态页面）
- 图片附件及下载服务。（upload）
- 上传图片服务。（static）

> - 上述三者的功能尽量分离。分离的最佳方式是分别使用独立的服务器（需要改动程序），如果程序实在不易更改，次选方案是在前端负载均衡器Haproxy/Nginx上，根据URI（例如目录或扩展名）过滤请求，然后抛给后面对应的服务器。
> - 例如：根据扩展名分发，请求http://www.yunjisuan.com/a/b.jpg就应抛给图片服务器（独立的静态服务器最适合使用CDN）；根据URL路径分发，请求http://www.yunjisuan.com/upload/index.php就应抛给上传服务器。不符合上面两个要求的，默认抛给Web服务器。

**说明：**可以部署3台服务器，人为分布请求服务器。当然了，这适合并发比较高，服务器较多的情况。程序架构分离了，效率，安全性都会提高很多。

## 十二，使用普通用户启动Nginx（监牢模式）

### 12.1 为什么要让Nginx服务使用普通用户

> 默认情况下，Nginx的Master进程使用的是root用户，worker进程使用的是Nginx指定的普通用过户，使用root用户跑Nginx的Master进程有两个最大的问题：

- 管理权限必须是root，这就使得最小化分配权限原则遇到难题。
- 使用root跑Nginx服务，一旦网站出现漏洞，用户就可以很容易地获得服务器的root权限。

> 因此，如果能有一种不用给开发人员，甚至普通运维人员管理员权限，就可以很好地管理Nginx服务的方法，具体内容将在下节为大家讲解。

### 12.2 给Nginx服务降权的解决方案

**解决方案如下：**

- 给Nginx服务降权，用inca用户跑Nginx服务，给开发及运维设置普通账号，只要与inca同组即可管理Nginx，该方案解决了Nginx管理问题，防止root分配权限过大。
- 开发人员使用普通账户即可管理Nginx服务及站点下的程序和日志。
- 采取项目负责制度，即谁负责项目维护，出了问题就是谁负责。

> 很多公司开发和运维为root权限争得不可开交，甚至大打出手。
>  参考资料：到底要不要给开发人员管理服务器的权限？（http://down.51cto.com/data/844517）.

### 12.3 给Nginx服务降权实战

> 本优化属架构优化（同样适合其他软件），通过Nginx启动命令的-c参数指定不同的Nginx配置文件，可以同时启动多个实例，并使用普通的用户运行服务。
>  Nginx安装后的启动命令路径为“/usr/local/nginx/sbin/nginx”,可以通过加-h参数查看相关参数的用法，命令及结果如下：

```
[root@LNMP logs]# /usr/local/nginx/sbin/nginx -h
nginx version: nginx/1.6.2
Usage: nginx [-?hvVtq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /usr/local/nginx/)
  -c filename   : set configuration file (default: conf/nginx.conf)     #使用指定的配置文件而不是conf目录下的nginx.conf这里我们就是通过nginx -c路径的功能来实现跑多实例nginx
  -g directives : set global directives out of configuration file
```

> 较常用的方法是使服务跑在指定用户的家目录下面，这样相对比较安全，同时有利于批量业务部署和上线。

**配置普通用户启动Nginx的过程如下：**

1）添加用户并创建相关目录和文件，操作如下：

```
[root@LNMP ~]# useradd yunjisuan
[root@LNMP ~]# su - yunjisuan
[yunjisuan@LNMP ~]$ pwd
/home/yunjisuan
[yunjisuan@LNMP ~]$ mkdir conf logs www         #在普通用户家目录下创建nginx配置文件目录
[yunjisuan@LNMP ~]$ ls
conf  logs  www
[yunjisuan@LNMP ~]$ cp /usr/local/nginx/conf/mime.types ~/conf/     #复制媒体类型配置文件
[yunjisuan@LNMP ~]$ echo "yunjisuan" > www/index.html           #创建网页首页
```

2）配置Nginx配置文件。配置后的查看命令如下：

```
[yunjisuan@LNMP ~]$ cat conf/nginx.conf 
worker_processes    4;
worker_cpu_affinity 0001 0010 0100 1000;
worker_rlimit_nofile 65535;
error_log   /home/yunjisuan/logs/error.log;
user yunjisuan yunjisuan;
pid /home/yunjisuan/logs/nginx.pid;
events {
    use epoll;
    worker_connections  10240;
}
http {
    include     mime.types;
    default_type    application/octet-stream;
    sendfile    on;
    keepalive_timeout   65;
    log_format main '$remote_addr-$remote_user[$time_local]"$request"'
    '$status $body_bytes_sent "$http_referer"'
    '"$http_user_agent""$http_x_forwarded_for"';
    
    server {
        listen 8080;
        server_name www.yunjisuan.com;
        root    /home/yunjisuan/www;
        location / {
            index   index.php index.html index.htm;
        }
        access_log  /home/yunjisuan/logs/web_blog_access.log main;
    }
}
[yunjisuan@LNMP ~]$ tree
.
├── conf
│   ├── mime.types
│   └── nginx.conf
├── logs
└── www
    └── index.html

3 directories, 3 files
```

> **说明如下：**
>
> - 所有参数的值，带路径的都要改成/home/yunjisuan.
> - 特权用户root使用的80端口，改为普通用过户使用的端口，在1024以上，这里为8080.

3）启动Nginx，命令如下：

```
[yunjisuan@LNMP ~]$ ps -ef | grep nginx | grep -v grep
[yunjisuan@LNMP ~]$ /usr/local/nginx/sbin/nginx -c /home/yunjisuan/conf/nginx.conf &>/dev/null &
[1] 2478
[yunjisuan@LNMP ~]$ ps -ef | grep nginx | grep -v grep
503        2479      1  0 14:34 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx -c /home/yunjisuan/conf/nginx.conf
503        2480   2479  0 14:34 ?        00:00:00 nginx: worker process                                         
503        2481   2479  0 14:34 ?        00:00:00 nginx: worker process                                         
503        2482   2479  0 14:34 ?        00:00:00 nginx: worker process                                         
503        2483   2479  0 14:34 ?        00:00:00 nginx: worker process                                         
[1]+  Done                    /usr/local/nginx/sbin/nginx -c /home/yunjisuan/conf/nginx.conf &>/dev/null

#提示503是用户yunjisuan的UID号
[yunjisuan@LNMP ~]$ id yunjisuan
uid=503(yunjisuan) gid=503(yunjisuan) groups=503(yunjisuan)
```

> **特别提示：**
>  此处启动nginx，如果不定向到空会显示一些提示，不是错误，可以通过&>/dev/null 定向到空，从而忽略不见。

```
[yunjisuan@LNMP ~]$ killall nginx
[yunjisuan@LNMP ~]$ ps -ef | grep nginx | grep -v grep
[yunjisuan@LNMP ~]$ /usr/local/nginx/sbin/nginx -c /home/yunjisuan/conf/nginx.conf &
[1] 2495
[yunjisuan@LNMP ~]$ nginx: [alert] could not open error log file: open() "/usr/local/nginx/logs/error.log" failed (13: Permission denied)
2017/08/30 14:38:20 [warn] 2495#0: the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /home/yunjisuan/conf/nginx.conf:5

#此处错误可以忽略，定向输出到&>/dev/null
```

4）配置好解析后，浏览器带端口访问结果如下图所示：

![QQ截图20170903013350.png-11.8kB](http://static.zybuluo.com/chensiqi/mvm4btftnafn9nzscukduu3i/QQ%E6%88%AA%E5%9B%BE20170903013350.png)

5）解决普通端口非80提供服务的问题

> 用负载均衡器解决Web服务非80端口的转换问题，负载均衡器可为Haproxy，Nginx，F5等。

**本解决方案的优点如下：**

- 给Nginx服务降权，让网站更安全。
- 按用户设置站点权限，使站点更独立（无需虚拟化隔离）
- 开发不需要用root即可完整管理服务及站点。
- 可实现责任划分，即网络问题属于运维的责任，网站打不开就是开发的责任，或者两者共同承担。

## 十三，控制Nginx并发连接数量

> - ngx_http_limit_conn_module这个模块用于限制每个定义的key值的连接数(Nginx默认已经被编译)，特别是单IP的连接数。
> - 不是所有的连接数都会被计数。一个符合计数要求的连接是整个请求头已经被读取的连接。

**控制Nginx并发连接数量参数的说明如下：**

1）limit_conn_zone参数：

```
语法：limit_conn_zone key zone=name:size;
上下文：http
#用于设置共享内存区域，key可以是字符串，Nginx自带变量或前两个组合，如$binary_remote_addr,$server_name.name为内存区域的名称，size为内存区域的大小。
```

2）limit_conn参数：

```
语法：limit_conn zone number;
上下文：http,server,location
#用于指定key设置最大连接数。当超过最大连接数时，服务器会返回503（Service Temporarily Unavailable）错误
```

### 13.1 限制单IP并发连接数

**Nginx的配置文件如下：**

```
[root@web03 nginx]# cat conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    server {
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
            root   html;
            index  index.html index.htm;
            limit_conn addr 1;      #限制同IP的并发为1；
        }
    }
}
```

**测试1：**利用Webbench进行并发连接测试（模拟一个客户端）

```
[root@localhost ~]# webbench -c 1 -t 5 http://192.168.0.225/
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.
Benchmarking: GET http://192.168.0.225/
1 client, running 5 sec.
Speed=341472 pages/min, 4808941 bytes/sec.
Requests: 28456 susceed, 0 failed.

#提示：192.168.0.225为web服务器，
#-c：指定客户端数量
#-t：指定持续时间
```

**我们查看Web服务器的access.log日志如下：**

```
[root@web03 nginx]# tail logs/access.log 
192.168.0.240 - - [30/Aug/2017:16:29:20 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:29:20 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:29:20 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:29:20 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:29:20 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:29:20 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:29:20 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:29:20 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:29:20 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:29:20 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
```

> 从上边的日志我们可以看到，当并发数为1时（就一个客户端），nginx正常记录用户的访问情况，http返回状态码为200

**测试2：**利用Webbench进行多客户端并发测试(模拟2个客户端)

```
[root@localhost ~]# webbench -c 2 -t 5 http://192.168.0.225/
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.
Benchmarking: GET http://192.168.0.225/
2 clients, running 5 sec.
Speed=565608 pages/min, 5808154 bytes/sec.
Requests: 47134 susceed, 0 failed.
```

**我们查看Web服务器的access.log日志如下：**

```
[root@web03 nginx]# tail logs/access.log 
192.168.0.240 - - [30/Aug/2017:16:31:08 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:31:08 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:31:08 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:31:08 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:31:08 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:31:08 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:31:08 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:31:08 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:31:08 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:31:08 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
```

> 从上边的日志我们可以看出，当我们用单IP地址模拟两个并发客户端进行连接时，日志以1：1比例交替出现了503错误。即Nginx已经做了并发连接限制，对超过限制的请求返回503.

**测试3：**利用Webbench进行多客户端并发测试(模拟3个客户端)

```
[root@localhost ~]# webbench -c 3 -t 5 http://192.168.0.225/
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.
Benchmarking: GET http://192.168.0.225/
3 clients, running 5 sec.
Speed=653388 pages/min, 6162385 bytes/sec.
Requests: 54449 susceed, 0 failed.
```

**我们查看Web服务器的access.log日志如下：**

```
[root@web03 nginx]# tail logs/access.log 
192.168.0.240 - - [30/Aug/2017:16:39:20 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:39:20 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:39:20 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:39:20 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:39:20 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:39:20 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:39:20 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:39:20 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:39:20 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:16:39:20 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
```

> 经过测试，可以看得出，200与503基本出现次数为1:2，即Nginx已经做了并发连接限制，对超过限制的请求返回503.

**以上功能的应用场景之一是用于服务器下载，命令如下：**

```
location /download/ {
    limit_conn addr 11;
}
```

> 上面的命令限制访问download下载目录的连接数，该连接数1.

### 13.2 限制虚拟主机总连接数

> 不仅可以限制单IP的并发连接数，还可以限制虚拟主机总连接数，甚至可以对两者同时限制。

**Nginx的配置文件如下:**

```
[root@web03 nginx]# cat conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    limit_conn_zone $server_name zone=perserver:10m;
    server {
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
            root   html;
            index  index.html index.htm;
        #limit_conn addr 1;
        limit_conn perserver 2;     #设置虚拟主机连接数为2
        }
    }
}
```

**测试：**

```
#重启Nginx服务后进行测试
root@localhost ~]# webbench -c 5 -t 5 http://192.168.0.225/
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.
Benchmarking: GET http://192.168.0.225/
5 clients, running 5 sec.
Speed=682128 pages/min, 7674820 bytes/sec.
Requests: 56844 susceed, 0 failed.
```

**我们查看Web服务器的access.log日志如下：**

```
[root@web03 nginx]# tail -20 logs/access.log 
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 503 213 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
192.168.0.240 - - [30/Aug/2017:17:11:14 -0400] "GET / HTTP/1.0" 200 612 "-" "WebBench 1.5"
[root@web03 nginx]# grep -c 200 logs/access.log 
35850
[root@web03 nginx]# grep -c 503 logs/access.log 
20998
[root@web03 nginx]# cat logs/access.log | wc -l
56848
```

> **结论：200和503出现的次数近似2：1**
>  至此，Nginx限制连接数的应用实践就讲解完毕了。
>  更多资料可参考：http：//nginx.org/en/docs/http/ngx_http_limit_conn_module.html

## 十四，控制客户端请求Nginx的速率

> ngx_http_limit_req_module模块用于限制每个IP访问每个定义key的请求速率。

```
语法：limit\_req\_zone key zone=name: size rate=rate;
上下文1：http
#用于设置共享内存区域，key可以是1字符串，Nginx自带变量或前两个组合，如$binary_remote_addr。name为内存区域的名称，size为内存区域的大小，rate为速率，单位为r/s，每秒一个请求。
limit_req参数说明如下：
语法：limit_req zone=name [burst=number][nodelay];
上下文：http,server,location
```

> - 这里运用了令牌桶原理，burst=num,一共有num块令牌，令牌发完后，多出来的那些请求就会返回503。
> - 换句话说，一个银行，只有一个营业员，银行很小，等候室只有5个人的位置。因此，营业员一个时刻只能为一个人提供服务，剩下的不超过5个人可以在银行内等待，超出的人不提供服务，直接返回503。
> - nodelay默认在不超过burst值的前提下会排队等待处理，如果使用此参数，就会处理完num + 1次请求，剩余的请求都视为超时，返回503。

**用于测试的Nginx配置文件如下：**

```
[root@web03 nginx-1.10.2]# cat /usr/local/nginx/conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;      #以请求的客户端IP作为key值，内存区域命令为one，分配10m内存空间，访问速率限制为1秒1次请求（request）
#    limit_conn_zone $binary_remote_addr zone=addr:10m;
#    limit_conn_zone $server_name zone=perserver:10m;
    server {
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
            root   html;
            index  index.html index.htm;
            limit_req zone=one burst=5;     #使用前面定义的名为one的内存空间，队列值为5，即可以有5个请求排队等待
#           limit_conn addr 1;
#           limit_conn addr 1;
        }
    }
}
[root@web03 nginx-1.10.2]# /usr/local/nginx/sbin/nginx -s reload          #重启服务
```

**利用Webbench进行测试**

```
[root@localhost ~]# webbench -c 100 -t 10 http://192.168.0.225/
Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.
Benchmarking: GET http://192.168.0.225/
100 clients, running 10 sec.
Speed=749706 pages/min, 4811119 bytes/sec.
Requests: 124951 susceed, 0 failed.
```

**我们查看Web服务器的access.log日志如下:**

```
[root@web03 nginx]# > logs/access.log 
[root@web03 nginx]# grep -c 200 logs/access.log 
11
[root@web03 nginx]# grep -c 503 logs/access.log 
124978
[root@web03 nginx]# cat logs/access.log | wc -l
124994

#说明：本次测试只成功响应了11个请求，近视1秒处理1个请求。多余的请求一律用503进行响应。
```

![QQ截图20170903121844.png-153.6kB](http://static.zybuluo.com/chensiqi/r02m7kze6ppm171wzm6dg3uw/QQ%E6%88%AA%E5%9B%BE20170903121844.png)

> - 通过过滤排重日志，可以看到第一个请求返回200，为正常处理，剩余2582次+9858次超过了限制的在1秒内执行完成，但是都返回了503
> - 具体过程原理为：Nginx在第1秒先处理第一个请求，同时接下来的5个请求等待排队，剩下的（2582+9858）请求返回503。接着第2秒到第6秒处理等待的5个请求。
>    更多内容可参考：http://nginx.org/en/docs/http/ngx_http_limit_req_module.html。

## 十五，本章重点回顾

1. 安全优化：隐藏Nginx软件名及版本号。
2. 性能加安全优化：连接超时参数及FastCGI相关参数调优
3. 性能优化：expires缓存功能及调试查看方法。
4. 性能优化：gzip压缩功能及调试查看方法。
5. 安全优化：集群中各角色服务站点目录权限控制策略。
6. 安全优化：站点目录下所有的文件和目录访问控制。
7. 性能加安全优化：静态资源防盗链解决方案
8. 性能加安全优化：robots.txt协议及防爬虫优化解决方案
9. 用户体验优化：错误页面优雅显示方法
10. 安全优化：限制http请求方法
11. 性能加安全优化：CDN加速知识
12. 安全优化：监牢模式运行Nginx方案策略
13. 性能加安全优化：Nginx并发连接数及请求速率控制