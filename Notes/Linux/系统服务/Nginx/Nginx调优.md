[TOC]

# 优化

## gzip压缩模块
```
http {
    ……
    gzip on;
    gzip_min_length 1k; #允许压缩的页面最小字节数，默认是0，多大都压缩，小于1k的可能适得其反
    gzip_buffers 4 16k; #gzip申请内存的大小，按数据大小的4倍去申请内存
    gzip_http_version 1.0; #识别http协议版本
    gzip_comp_level 2; #压缩级别，1压缩比最小，处理速度最快，9压缩比最大，处理速度最慢
    gzip_types text/plainapplication/x-javascripttext/css application/xml image/jpg; #压缩数据类型
    gzip_vary on; #根据客户端的http头来判断，是否需要压缩
}
```

## expires缓存模块
```
server {
    location ~ .*\.(gif|jpg|png|bmp|swf)$ #缓存数据后缀类型
    {
      expires 30d; #使用expires缓存模块，缓存到客户端30天
    }
    location ~ .*\.( jsp|js|css)?$
    {
      expires 1d;
    }
}
```

## fastcgi优化
nginx不支持直接调用或者解析动态程序（php），必须通过fastcgi（通用网关接口）来启动php-fpm进程来解析php脚本。也就是说用户请求先到nginx，nginx再将动态解析交给fastcgi，fastcgi启动php-fpm解析php脚本。所以我们有必要对fastcgi和php-fpm进行适当的参数优化。

```
http {

    ……
    
    fastcgi_cache_path/usr/local/nginx/fastcgi_cache levels=1:2 keys_zone=TEST:10m inactive=5m;  
    # FastCGI缓存指定一个文件路径、目录结构等级、关键字区域存储时间和非活动删除时间
    
    fastcgi_connect_timeout 300; #指定连接到后端FastCGI的超时时间
    
    fastcgi_send_timeout 300; #指定向FastCGI传送请求的超时时间
    
    fastcgi_read_timeout 300; #指定接收FastCGI应答的超时时间
    
    fastcgi_buffer_size 64k; #指定读取FastCGI应答第一部分需要多大的缓冲区
    
    fastcgi_buffers 4 64k; #指定本地需要用多少盒多大的缓冲区来缓冲FastCGI的应答请求
    
    fastcgi_busy_buffers_size 128k;   
    fastcgi_temp_file_write_size 128k; #表示在写入缓存文件时使用多大的数据块，默认值是fastcgi_buffers的两倍
    
    fastcgi_cache TEST; #开启fastcgi_cache缓存并指定一个TEST名称
    
    fastcgi_cache_valid 200 302 1h; #指定200、302应答代码的缓存1小时
    
    fastcgi_cache_valid 301 1d; #将301应答代码缓存1天
    
    fastcgi_cache_valid any 1m; #将其他应答均缓存1分钟
}
```

php-fpm.conf配置参数
```
pm =dynamic #两种控制子进程方式（static和dynamic）

pm.max_children= 5 #同一时间存活的最大子进程数

pm.start_servers= 2 #启动时创建的进程数

pm.min_spare_servers= 1 #最小php-fpm进程数

pm.max_spare_servers= 3 #最大php-fpm进程数
```

## proxy_cache本地缓存模块

```
http {
    ……
    proxy_temp_path /usr/local/nginx/proxy_cache/temp; # 缓存临时目录
    
    proxy_cache_path /usr/local/nginx/proxy_cache/cache levels=1:2 keys_zone=one:10m inactive=1d max_size=1g;
    # 缓存文件实际目录，levels定义层级目录，1:2说明1是一级目录，2是二级目录，keys_zone存储元数据，并分配10M内存空间。inctive表示1天没有被访问的缓存就删除，默认10分钟。max_size是最大分配磁盘空间
    
    server {
        listen 80;
        server_name 192.168.1.10;
        location / {
        proxy_cache one; #调用缓存区
        #proxy_cache_valid 200 304 12h; #可根据HTTP状态码设置不同的缓存时间
        proxy_cache_valid any 10m; #缓存有效期为10分钟
    }
      #清除URL缓存，允许来自哪个网段的IP可以清除缓存(需要安装第三方模块"ngx_cache_purge"),清除URL缓存方法：访问http://192.168.1.10/purge/文件名
    location ~ /purge(/.*){
        allow 127.0.0.1;
        allow 192.168.1.0/24;
        deny all;
        proxy_cache_purge cache_one$host$1$is_args$args;
    }
}
```

# 总结
deflate配置参数：

启用压缩模块可以节省一部分带宽，会增加WEB端CPU处理，但在上图网站架构中，WEB端启用压缩模块并没有起到作用，因为传输到上层走的是局域网。对于直接面向用户的架构还是要启用的。WEB也不用启用expires模块，因为有了反向代理服务器和CDN，所以到不了用户浏览器，开启起不到作用。

如果反向代理使用nginx做代理，可开启expires模块，将静态文件缓存到用户浏览器，浏览器发起请求时，先判断本地缓存是否有请求的数据，如果有再判断是否过期，如果不过期就直接浏览缓存数据，哪怕服务器资源已经改变，所以要根据业务情况合理设置过期时间。


利用PHP缓存器提高代码执行效率

php程序在没有使用缓存器情况下，每次请求php页面，php都会对此页面进行代码编译，这就意味着重复的编译工作会增加服务器负载。有了缓存器就会把每次编译后的数据缓存到共享内存中，下次访问直接使用缓冲区已编译好的代码，从而避免重复的编译过程，以加快其执行效率。因此PHP网站使用缓存器是完全有必要的！主流的PHP缓存器有：eAccelerator、XCache。