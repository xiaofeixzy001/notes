[TOC]

# curl（文件传输工具）

## 常用参数如下：

-c，--cookie-jar：将cookie写入到文件

-b，--cookie：从文件中读取cookie

-C，--continue-at：断点续传

-d，--data：http post方式传送数据

-D，--dump-header：把header信息写入到文件

-F，--from：模拟http表达提交数据

-s，--slient：减少输出信息

-o，--output：将信息输出到文件

-O，--remote-name：按照服务器上的文件名，存在本地

--l，--head：仅返回头部信息

-u，--user[user:pass]：设置http认证用户和密码

-T，--upload-file：上传文件

-e，--referer：指定引用地址

-x，--proxy：指定代理服务器地址和端口

-w，--write-out：输出指定格式内容

--retry：重试次数

--connect-timeout：指定尝试连接的最大时间/s

## 使用示例：

例1：抓取页面到指定文件，如果有乱码可以使用iconv转码

\# curl -o baidu.html www.baidu.com

\# curl –s –o baidu.html www.baidu.com |iconv -f utf-8  #减少输出信息

例2：模拟浏览器头（user-agent）

\# curl -A "Mozilla/4.0 (compatible;MSIE 6.0; Windows NT 5.0)" www.baidu.com

例3：处理重定向页面

\# curl –L http://192.168.1.100/301.php  #默认curl是不处理重定向

例4：模拟用户登陆，保存cookie信息到cookies.txt文件，再使用cookie登陆

\# curl -c ./cookies.txt -F NAME=user -F PWD=***URL       #NAME和PWD是表单属性不同，每个网站基本都不同

\# curl -b ./cookies.txt –o URL

例5：获取HTTP响应头headers

\# curl -I http://www.baidu.com

\# curl -D ./header.txt http://www.baidu.com  #将headers保存到文件中

例6：访问HTTP认证页面

\# curl –u user:pass URL

例7：通过ftp上传和下载文件

\# curl -T filename ftp://user:pass@ip/docs  #上传

\# curl -O ftp://user:pass@ip/filename  #下载

参考博客：http://lizhenliang.blog.51cto.com

# wget（文件下载工具）

常用参数如下：

2.1 启动参数

-V，--version：显示版本号

-h，--help：查看帮助

-b，--background：启动后转入后台执行

2.2 日志记录和输入文件参数

-o，--output-file=file：把记录写到file文件中

-a，--append-output=file：把记录追加到file文件中

-i，--input-file=file：从file读取url来下载

2.3 下载参数

-bind-address=address：指定本地使用地址

-t，-tries=number：设置最大尝试连接次数

-c，-continue：接着下载没有下载完的文件

-O，-output-document=file：将下载内容写入到file文件中

-spider：不下载文件

-T，-timeout=sec：设置响应超时时间

-w，-wait=sec：两次尝试之间间隔时间

--limit-rate=rate：限制下载速率

-progress=type：设置进度条

2.4 目录参数

-P，-directory-prefix=prefix：将文件保存到指定目录

2.5 HTTP参数

-http-user=user：设置http用户名

-http-passwd=pass：设置http密码

-U，--user-agent=agent：伪装代理

-no-http-keep-alive：关闭http活动链接，变成永久链接

-cookies=off：不使用cookies

-load-cookies=file：在开始会话前从file文件加载cookies

-save-cookies=file：在会话结束将cookies保存到file文件

2.6 FTP参数

-passive-ftp：默认值，使用被动模式

-active-ftp：使用主动模式

2.7 递归下载排除参数

-A，--accept=list：分号分割被下载扩展名的列表

-R，--reject=list：分号分割不被下载扩展名的列表

-D，--domains=list：分号分割被下载域的列表

--exclude-domains=list：分号分割不被下载域的列表

使用示例：

例1：下载单个文件到当前目录下，也可以-P指定下载目录

\# wgethttp://nginx.org/download/nginx-1.8.0.tar.gz

例2：对于网络不稳定的用户可以使用-c和--tries参数，保证下载完成

\# wget --tries=20 -c http://nginx.org/download/nginx-1.8.0.tar.gz

例3：下载大的文件时，我们可以放到后台去下载，这时会生成wget-log文件来保存下载进度

\# wget -b http://nginx.org/download/nginx-1.8.0.tar.gz

例4：可以利用—spider参数判断网址是否有效

\# wget --spider http://nginx.org/download/nginx-1.8.0.tar.gz

例5：自动从多个链接下载文件

\# cat url_list.txt  #先创建一个URL文件

http://nginx.org/download/nginx-1.8.0.tar.gz

http://nginx.org/download/nginx-1.6.3.tar.gz

\# wget -i url_list.txt

例6：限制下载速度

\# wget --limit-rate=1m http://nginx.org/download/nginx-1.8.0.tar.gz

例7：登陆ftp下载文件

\# wget --ftp-user=user --ftp-password=pass ftp://ip/filename