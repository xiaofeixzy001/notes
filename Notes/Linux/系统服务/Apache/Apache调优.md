[TOC]

# 核心思路
减少请求层，尽可能让前端层返回用户请求的数据，减少后端服务器访问频率，最重要是数据库层。

# 优化
## mod_deflate压缩模块

```
# 查看是否加载：
apachectl –M |grep deflate

# 如果没有安装使用apxs编译进去：
/usr/local/apache/bin/apxs –c –I –A apache源码目录/modules/mod_deflate.c

# deflate配置参数：
'''
<IfModulemod_deflate.c>
    DeflateCompressionLevel6      #压缩等级（1-9），数值越大效率越高，消耗CPU也就越高
    SetOutputFilterDEFLATE      #启用压缩
    AddOutputFilterByTypeDEFLATE text/html text/plain text/xml #压缩类型
    AddOutputFilterByTypeDEFLATE css js html htm xml php  
</IfModule>
'''
```

## mod_expires缓存模块
```
# 查看是否加载：
apachectl –M |grep expires

# 如果没有安装使用apxs编译进去：
/usr/local/apache/bin/apxs –c –I –A apache源码目录/modules/mod_expires.c

# 再在httpd.conf启用模块：LoadModule expires_module modules/mod_expires.so 
```

缓存机制有三种用法：全局、目录和虚拟主机
```
# 全局配置，在配置文件末尾添加：
<IfModulemod_expires.c>
    ExpiresActiveon #启用有效期控制，会自动清除已过期的缓存，然后从服务器获取新的
    ExpiresDefault "accessplus 1 days" #默认任意格式的文档都是1天后过期
    ExpiresByTypetext/html "access plus 12 months"  
    ExpiresByTypeimage/jpg "access plus 12 months" #jpg格式图片缓存12月
</IfModule>
```

## 工作模式选择及优化

apache有两种常见工作模式，worker和prefork，默认是worker，是混合型的MPM（多路处理模块），支持多进程和多线程，由线程来处理请求，所以可以处理更多请求，提高并发能力，系统资源开销也小于基于进程的MPM.

由于线程使用进程内存空间，进程崩溃会导致其下线程崩溃。而prefork是非线程型MPM，进程占用系统资源也比worker多，由于进程处理连接，在工作效率上也比worker更稳定。可通过```apachectl -l```查看当前工作模式，在编译时使用```--with-mpm```参数指定工作模式。根据自己业务需求选择不同工作模式，再适当增加工作模式相关参数，可提高处理能力。

配置参数说明

```
<IfModuleprefork.c>
    StartServers 8 #默认启动8个httpd进程
    MinSpareServers 5 #最小的空闲进程数
    MaxSpareServers 20 #最大的空闲进程数，如果大于这个值，apache会自动kill一些进程
    ServerLimit 256 #服务器允许进程数的上限
    MaxClients 256 #同时最多发起多少个访问，超过则进入队列等待
    MaxRequestsPerChild 4000 #每个进程启动的最大线程
</IfModule>
```



# End