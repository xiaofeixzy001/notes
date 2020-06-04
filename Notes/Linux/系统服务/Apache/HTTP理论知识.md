[TOC]

# http简介
Hyper Text Transfer Protocol

httpd是Apache超文本传输协议（HTTP）服务器的主程序。

被设计为一个独立的运行的后台进程，它会建立一个出来请求的子进程或线程的池。通常httpd不被直接调用，而是由apachectl调用。

http是一个应用层的协议，基于C/S架构模式，它可以分为http和https，它们分别使用tcp协议端口的80和443端口。

## http事务
客户端需要访问某资源时会向服务器发送http请求报文，服务器根据客户端请求信息做出http响应报文，所以一次http事务就是http请求然后http会给予请求响应。

## web资源

资源的标识：

URL：用于标识web资源所在的位置。

格式：

协议：//主机地址或者主机名[：端口][/目录资源]

静态资源：不需要服务器做任何操作处理，例如.JGP .PNG格式的文件等

动态资源：服务器需要执行一些程序做出处理后返回给客户端请求所需要的信息，例如.php或.js
一次完整的http请求处理过程：
- 建立或处理连接：接收请求或者是拒绝请求
- 接收请求：接收客户端主机请求报文中对某个资源的一次请求过程
- 处理请求：对请求报文进行解析，获取客户端请求的资源及请求方法相关信息
- 访问资源：获取请求报文中请求的资源
- 构建响应报文
- 发送响应报文
- 记录日志

## 并发访问响应模型

单进程I/O模型：启动一个进程处理用户请求，一次只能处理一个请求对公请求被串行响应。

多进程I/O模型：并行启动多个进程，每个进程响应一个请求。

复用进程I/O模型：一个进程响应多个请求。

多线程模式：一个进程生成多个线程，一个线程处理一个请求。

事件驱动模式：一个进程直接响应多个请求。

复用多进程I/O模型：启动多个进程，每个进程生成多个线程，响应请求的数量就是线程乘以进程。

## http方法
GET：请求获取一个资源，需要服务器发送。

HEAD：跟GET近似，但不需要服务器相应请求的资源，而是返回响应首部。

POST：基于HTML表单向服务器提交数据，服务通常需要存储此数据（位置通常为关系型数据库）。

PUT：与GET相反，向服务器发送资源，服务器通常需要存储此资源（位置通常为文件系统）。

DELETE：删除URL指向的资源。

OPTIONS：探测服务器端对请求的URL所支持使用的请求方法。

TRACE：跟一次请求中间所经过的代理服务器、防火墙或网关等。

## http状态码
1XX：信息性状态码

2XX：成功状态码

200：OK

201：CREATER

3XX：重定向类的状态码

301：永久重定向

302：临时重定向

304：Not Modified

4XX：客户端类错误

403：请求被拒绝Forbidden

404：请求不存在Not Found

405：请求方法被拒绝Method Not Allowed

5XX：服务器类错误

500：服务器内部错误

502：错误的网关

503：服务不可用

# http协议
## 无状态

版本

- 0.9：仅用于传输html文档。

- 1.0：引入MIME机制，从而支持多媒体数据，引入keep-alive（持久连接），缓存。

- 1.1：更多的请求方法，更精细缓存控制，持久连接（persistent）。

## http协议首部

## 通用首部
Connection：定义C/S之间关于请求、响应的有关选项。

Cache-Control：缓存控制。

## 请求首部

Client-IP

host：请求的主机

Referer：指明了请求当前资源的原始资源URL，可防止盗链

User-Agent：用户代理

## Accect首部

Accept：服务端能够发送的媒体的类型

Accept-Charset

Accept-Encoding

Accept-Language

## 条件式请求
跟安全相关请求

Authorization

Cookie

## 响应首部
Age：缓存时长

Server：向客户说明自己的程序名称和版本

## 协商首部
Vary：首部列表，服务器会根据列表中的内容挑一个最适用的版本发送给客户端

## 跟安全相关
WWW-Authentication

Set-Cookie

## 实体首部
Location：资源的新位置

Allow：允许对此资源使用的请求方法

## 内容相关的首部

Content-Encoding

Content-Language

Content-Length

Content-Location

Content-Type

## 缓存相关首部
ETag

Expires

Lsat-Modified

## 扩展首部
非标准，由程序员自行创建（如：X-Forward，X-Via）

# httpd的特性
## 高度模块化

DSO：Dynamic Shared Object

```
LoadModule module_alias “/path/to/module_file” 
```

即时生效

MPM[Multipath Processing Module]:

统称，事实上有多个实现：

prefork：每个进程响应一个用户请求，预先生成多个空闲进程；

select（）：1024

worker：启动多个进程，每个进程生成多个线程，每个线程响应一个用户请求

event-driver：事件驱动模式，一个进程响应多个用户请求

## 功能特性

丰富的用户认证：基本认证和摘要认证

CGI：原生支持perl

虚拟主机：基于端口、IP、主机名

反向代理：负载均衡

用户站点：

路径别名：

支持第三方模块

# CGI, FASTCGI, PHP-FPM

## Cgi

讲Fastcgi之前需要先讲CGI，CGI是为了保证web server传递过来的数据是标准格式的，它是一个协议，方便CGI程序的编写者。

Fastcgi是CGI的更高级的一种方式，是用来提高CGI程序性能的。

web server（如nginx）只是内容的分发者。比如，如果请求/index.html，那么web server会去文件系统中找到这个文件，发送给浏览器，这里分发的是静态资源。

如果现在请求的是/index.php，根据配置文件，nginx知道这个不是静态文件，需要去找PHP解析器来处理，那么他会把这个请求简单处理后交给PHP解析器。此时CGI便是规定了要传什么数据／以什么格式传输给php解析器的协议。

当web server收到/index.php这个请求后，会启动对应的CGI程序，这里就是PHP的解析器。接下来PHP解析器会解析php.ini文件，初始化执行环境，然后处理请求，再以CGI规定的格式返回处理后的结果，退出进程。web server再把结果返回给浏览器。

那么CGI相较于Fastcgi而言其性能瓶颈在哪呢？CGI针对每个http请求都是fork一个新进程来进行处理，处理过程包括解析php.ini文件，初始化执行环境等，然后这个进程会把处理完的数据返回给web服务器，最后web服务器把内容发送给用户，刚才fork的进程也随之退出。 如果下次用户还请求动态资源，那么web服务器又再次fork一个新进程，周而复始的进行。

## Fastcgi
Fastcgi则会先fork一个master，解析配置文件，初始化执行环境，然后再fork多个worker。当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当worker不够用时，master可以根据配置预先启动几个worker等着；当然空闲worker太多时，也会停掉一些，这样就提高了性能，也节约了资源。这就是Fastcgi的对进程的管理。大多数Fastcgi实现都会维护一个进程池。注：swoole作为httpserver，实际上也是类似这样的工作方式。

## PHP-FPM
PHP-FPM,它是一个实现了Fastcgi协议的程序,用来管理Fastcgi起的进程的,即能够调度php-cgi进程的程序。现已在PHP内核中就集成了PHP-FPM，使用 --enalbe-fpm 这个编译参数即可。另外，修改了php.ini配置文件后，没办法平滑重启，需要重启php-fpm才可.此时新fork的worker会用新的配置，已经存在的worker继续处理完手上的活.

通俗的可以把服务器看作餐厅，用户请求看作来用餐的顾客，服务器处理请求看作解决顾客的就餐问题（响应输出一份饭）:
服务器上静态资源看作已做好的饭:快餐，只要放到餐盒里就可以返回给顾客，
动态资源需要厨房大厨现成做份再放到餐盒里返回给顾客:
php_mod这个大厨有个特点，看见有顾客进门就点火，不管顾客要不要现做的，有点浪费资源
php_fpm这个大厨有好多小弟一直点着火（多个处理进程），等有顾客说要现做，大厨就安排小弟做份返回给客户
cgi也是个大厨，不过他等到顾客要现做，他才点火，做饭，然后熄火。等待下一个要现做的到来
fastcgi呢就是个大厨雇了一帮小弟，专门做需要现场做的饭，大厨只管分派任务，小弟真正操锅做饭

