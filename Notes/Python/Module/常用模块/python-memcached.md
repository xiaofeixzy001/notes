[TOC]

# 安装mamcache

Memcached 是一个高性能的分布式内存对象缓存系统，用于动态Web应用以减轻数据库负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高动态、数据库驱动网站的速度。Memcached基于一个存储键/值对的hashmap。其守护进程（daemon ）是用C写的，但是客户端可以用任何语言来编写，并通过memcached协议与守护进程通信。

 

```
# 安装依赖包
yum install gcc gcc++

wget https://github.com/libevent/libevent/releases/download/release-2.1.8-stable/libevent-2.1.8-stable.tar.gz
tar zxvf libevent-2.1.8-stable.tar.gz
cd /applications/libevent
./configure --prefix=/applications/libevent
make && make install

# 安装memcached
wget http://memcached.org/latest
tar -zxvf memcached-1.x.x.tar.gz
cd memcached-1.x.x
./configure --prefix=/applications/memcached --with-libevent=/applications/libevent
make && make install

# 启动
/applications/memcached/bin/memcached -d -m 10 -u root -l 127.0.0.1 -p 12000 -c 256 -P /applications/memcached/lock/memcached.pid

ps aux | grep memcached
ss -tunlp | grep 12000

# 连接
telnet 127.0.0.1 12000
```

memcached参数说明：

-p  指定端口号（默认11211）  

-m  指定最大使用内存大小（默认64MB）  

-t  线程数（默认4）  

-l  连接的IP地址, 默认是本机  

-d  以后台守护进程的方式启动

-c  最大同时连接数，默认是1024

-P  制定memecache的pid文件

-h  打印帮助信息

# python-memcached模块

python连接memcache可以使用python-memcached模块和pylibmc模块

下载地址：https://pypi.python.org/pypi/python-memcached

## 连接memcache

 

```
import memcache

mem_host = ['172.16.1.40:11211']

mc = memcache.Client(mem_host, debug=True)
mc.set('k1','v1')
ret = mc.get('k1')
print(ret)
```

注：debug = True 表示开启调试功能，当程序运行出现错误时，现实错误信息，上线时可移除该参数。

## 集群操作

python-memcached模块原生支持集群操作，其原理是在内存维护一个主机列表，且集群中主机的权重值和主机在列表中重复出现的次数成正比

 

```
  主机   权重
1.1.1.1   1
1.1.1.2   2
1.1.1.3   1
```

那么在内存中主机列表为：

host_list = ["1.1.1.1", "1.1.1.2", "1.1.1.2", "1.1.1.3", ]

如果用户根据如果要在内存中创建一个键值对（如：k1 = "v1"），那么要执行一下步骤：

1,根据算法将 k1 转换成一个数字

2,将数字和主机列表长度求余数，得到一个值 N（ 0 <= N < 列表长度 ）

3,在主机列表中根据 第2步得到的值为索引获取主机，例如：host_list[N]

4,连接 将第3步中获取的主机，将 k1 = "v1" 放置在该服务器的内存中

代码实现如下：

 

```
mc = memcache.Client([('1.1.1.1:12000', 1), ('1.1.1.2:12000', 2), ('1.1.1.3:12000', 1)], debug=True)
mc.set('k1', 'v1')
```

## 命令操作

### add & replace & set & set_multi

add - 添加键值对，如果已存在key，则报异常

replace - 修改某个key的值，如果key不存在则报异常

set - 设置键值对，如果key不存在则创建，key存在则修改

set_multi - 设置多个键值对，如果key不存在则创建，key存在则修改

 

```
import memcache
mem_host = ['172.16.1.40:11211']

mc = memcache.Client(mem_host, debug=True)

mc.add('k2','v2')
mc.add('k2','v3') # 报错

mc.replace('k2','v3')
mc.replace('k3','v3')  # 报错

mc.set('k2','v2')
mc.set('k3','v3')

# 传递参数为字典
mc.set_multi({'k3':'v4', 'k4':'v4'})

ret1 = mc.get('k1')
ret2 = mc.get('k2')
ret3 = mc.get('k3')
ret4 = mc.get('k4')
print(ret1)
print(ret2)
print(ret3)
print(ret4)
```

### get & get_multi

get - 获取一个键值对

get_multi - 获取多个键值对

 

```
# 传递参数为列表
ret = mc.get_multi(['k1','k2','k3','k4'])

# 运行结果为字典：
{'k1': 'v1', 'k2': 'v2', 'k3': 'v4', 'k4': 'v4'}
```

**delete & delete_multi**

delete - 删除指定的一个键值对

delete_multi - 删除指定的多个键值对

 

```
mc.delete('k4')

# 传递参数为字典
mc.delete_multi(['k1','k3'])
```

### append & prepend

append - 修改指定的key对应的value，在该value后追加内容

prepend - 修改指定的key对应的value，在该value前追加内容

 

```
mc.append('k2','-after')  # {'k2': 'v2-after'}

mc.prepend('k2','before-')  # {'k2': 'before-v2'}

mc.append('k5','555555')
mc.prepend('k5','55')
print(mc.get_multi(['k5']))  # {}
```

注：如果指定的key不存在，不会报错，只返回空字典

### decr & incr

incr - 自增，将某一值增加N

decr - 自减，将某一值减少N

(N默认为1)

 

```
import memcache
mem_host = ['172.16.1.40:11211']
mc = memcache.Client(mem_host, debug=True)
mc.set_multi({'k5':100,'k6':200})

mc.incr('k5')
mc.incr('k6',50)
print(mc.get_multi(['k5','k6'])) # {'k5': 101, 'k6': 250}

mc.decr('k5')
mc.decr('k6',10)
print(mc.get_multi(['k5','k6'])) # {'k5': 99, 'k6': 190}
```

### gets & cas

获取和修改值

主要用来解决数据读写冲突问题

如果不采用CAS，则有如下的情景： 

第一步，A取出数据对象X； 

第二步，B取出数据对象X； 

第三步，B修改数据对象X，并将其放入缓存； 

第四步，A修改数据对象X，并将其放入缓存。 

我们可以发现，第四步中会产生数据写入冲突

如果采用CAS协议，则是如下的情景。 

第一步，A取出数据对象X，并获取到CAS-ID1； 

第二步，B取出数据对象X，并获取到CAS-ID2； 

第三步，B修改数据对象X，在写入缓存前，检查CAS-ID与缓存空间中该数据的CAS-ID是否一致。结果是“一致”，就将修改后的带有CAS-ID2的X写入到缓存。 

第四步，A修改数据对象Y，在写入缓存前，检查CAS-ID与缓存空间中该数据的CAS-ID是否一致。结果是“不一致”，则拒绝写入，返回存储失败。 

我们可以通过重试，或者其他业务逻辑解决第四步设置失败的问题