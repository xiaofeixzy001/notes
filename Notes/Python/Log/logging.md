[TOC]

# logging

## 简介

logging模块是Python内置的标准模块,主要用于输出运行日志,可以设置输出日志的等级、日志保存路径、日志文件回滚等;相比print,具备如下优点:

1,可以通过设置不同的日志等级,在release版本中只输出重要信息,而不必显示大量的调试信息;

2,print将所有信息都输出到标准输出中,严重影响开发者从标准输出中查看其它数据,logging则可以由开发者决定将信息输出到什么地方,以及怎么输出.

## 日志级别

日志级别指的是产生日志的事件的严重程度。

level严重级别由低到高: NOTSET(0)，DEBUG(10)，INFO(20)，WARNING(30)，ERROR(40)，CRITICAL(50)

默认级别是WARNING

logging模块只会输出指定level以上的信息，低于设置值的日志消息将被忽略。

这样的好处, 就是在项目开发时debug用的log，在产品release阶段不用一一注释，只需要调整logger的级别就可以了，很方便。

## 格式字符串

%(message)s: 日志消息内容,调用Formatter.format()时设置。

%(asctime)s: 字符串形式的当前时间.默认格式是"2003-07-08 16:49:45,896",逗号后面的是毫秒.

%(funcName)s: 产生日志的函数名

%(levelname)s: 日志级别名称,如'NOTSET, DEBUG, INFO, WARN, ERROR, CRITICAL'。

%(levelno)s: 数字形式的日志级别,如0,10,20,30,40,50。

%(lineno): 日志调用所在的源码行号。

%(module)s: 日志调用的模块名，filename的名字部分。

%(filename)s: 当前执行程序名

%(process)d: 进程ID

%(processName)d: 进程名

%(thread)d: 线程ID

%(threadName)s: 线程名

%(name)s: 执行者

%(pathname)s: 当前执行该程序的路径,其实就是sys.argv[0]

%(created)f: 当前时间,用UNIX标准的表示时间的浮点数表示

%(relativeCreated)d: 输出日志信息时的,自Logger创建以来的毫秒数

默认的日志格式为: "日志级别:用户(默认root):用户输出消息;"

## basicConfig()

logging.basicConfig()用于配置日志格式显示等信息

### 参数

filename=FILE_NAME: 指定一个文件,将日志写入该文件中

filemode='w/a': 文件打开方式(a追加,w写),默认为"a",追加日志文件方式

format: 指定日志输出的格式和内容

datefmt: 指定日期时间格式

level: 设置日志级别,默认WARNING

stream: 指定日志的输出流,可输出到sys.stderr,sys.stdout或者文件,默认sys.stderr,当stream和filename同时指定时,stream将被忽略。

### 示例

修改级别和自定义显示格式

 

```
import logging
FORMAT = "%(asctime) - 15s\tThread info: %(thread)d %(threadName)s %(message)s %(school)s"
logging.basicConfig(format=FORMAT, datefmt=DMT, level=logging.INFO)  # 设置级别,默认WARNING
d = {'school': 'magedu.com'}
logging.info("I am %s %s", 20, 'years old.', extra=d)
logging.warning("I am %s %s", 20, 'year old.', extra=d)
```

修改日期格式

 

```
import logging
logging.basicConfig(format='%(asctime)s %(message)s', datefmt='%Y/%m/%d %I:%M:%S')
logging.warning('this event was logged.')
```

输出到文件

 

```
import logging
logging.basicConfig(format="%(asctime)s %(message)s", filename='test.log')
for _ in range(5):
    logging.warning("this event was logged.")
```

记录异常信息

当你使用logging模块记录异常信息时，不需要传入该异常对象，只要你直接调用logger.error() 或者 logger.exception()就可以将当前异常记录下来。

 

```
try:
    1 / 0
except:
    # 等同于error级别，但是会额外记录当前抛出的异常堆栈信息
    logger.exception('this is an exception message')
"""
2016-10-08 21:59:19,493 ERROR : this is an exception message
Traceback (most recent call last):
File "D:/Git/py_labs/demo/use_logging.py", line 45, in 
1 / 0
ZeroDivisionError: integer division or modulo by zero
"""
```

Python使用logging模块记录日志涉及四个主要类，使用官方文档中的概括最为合适：

1,logger提供了应用程序可以直接使用的接口；

2,handler将(logger创建的)日志记录发送到合适的目的输出；

3,filter提供了细度设备来决定输出哪条日志记录；

4,formatter决定日志记录的最终输出格式。

# Logger

logging模块加载时，会创建一个root logger的实例，即根Logger，根logger对象的默认级别是WARNING。

调用logging.basicConfig来调整级别，实际就是对这个根Logger的级别进行修改。

## 层次结构

logging.getLogger([name=None])

指定name,返回一个名称为name的实例，不指定就返回根Logger的实例。

 

```
import logging
root = logging.getLogger()  # 根logger
log1 = logging.getLogger(__name__)  # 子logger
log2 = logging.getLogger(__name__ + 'child')  # 没有点,root的孩子
log3 = logging.getLogger(__name__ + '.child')  # 有点,__name__的孩子,也是root的孙子
print(log2.name, id(log2.parent), id(log1), id(root))
print(log3.name, id(log3.parent), id(log1), id(log1.parent), id(root))
```

Logger是有层次结构的，使用点号.分割，如a, a.b, a.b.c, a是a.b的parent, a.b是a.b.c的parent。

## level设置

 

```
import logging

FORMAT = '%(asctime)s  %(message)s'
logging.basicConfig(format=FORMAT, level=logging.WARNING)  # 根logger级别

logger = logging.getLogger(__name__)  # 子logger,未设置级别
print(logger.level)  # 查看当前level: 0
print(logger.getEffectiveLevel())  # 查看真实level: 父类级别30

logger.info('info msg')  # 不打印
logger.warning('warning msg')  # 打印

logger.setLevel(20)  # 修改子logger级别
print(logger.level)  # 20
print(logger.getEffectiveLevel())  # 20

logger.info('info msg')  # 打印
logger.warning('warning msg')  # 打印

root = logging.getLogger()
root.info('root info msg')  # 不打印
```

根据上例可以得出结论：

通过工厂方法getLogger()得到根Logger的实例,不指定name则为根Logger实例root，指定name则为根Logger的子类的实例logger。

子类没有设置日志级别，以父类级别为准，父没有继续向上找，到root为止;

子类设置了自己的级别，则以自己的为准，无论是否比父类级别小。

## **Handler**

Handler控制日志信息的输出目的地，可以是控制台、文件。

handler对象负责发送相关的信息到指定目的地。Python的日志系统有多种Handler可以使用。有些Handler可以把信息输出到控制台，有些Logger可以把信息输出到文件，还有些 Handler可以把信息发送到网络上。如果觉得不够用，还可以编写自己的Handler。可以通过addHandler()方法添加多个多handler。

可单独设置level；

可单独设置格式；

可设置过滤器；

日志的输出其实是Handler做的，也就是真正干活的是Handler。

查看logging.basicConfig部分源码：

 

```
if handlers is None:
    filename = kwargs.pop("filename", None)
    mode = kwargs.pop("filemode", 'a')
    if filename:
        h = FileHandler(filename, mode)
    else:
        stream = kwargs.pop("stream", None)
        h = StreamHandler(stream)
    handlers = [h]
```

如果设置文件名，就是为root logger加了一个输出到文件的handler；

如果没有设置文件名，就是为root logger加了一个StreamHandler，默认输出到sys.stderr；

也就是说，根logger一定会至少有一个handler的。

### Handler类继承

Handler

|----- StreamHandler  # 不指定使用sys.stderr

​    |----- FileHandler  # 文件

​    |----- _StderrHandler  # 标准输出

|----- NullHandler  # 什么都不做

### 示例

 

```
import logging
import sys

FORMAT = '%(asctime)s ** %(message)s'
logging.basicConfig(format=FORMAT, level=10)  # 根logger级别和格式

FMT = logging.Formatter('%(levelname)s %(asctime)s %(name)s %(message)s')
log1 = logging.getLogger(__name__)

file_handler = logging.FileHandler('log1.log', 'w')  # 创建handler
file_handler.setFormatter(FMT)  # 设置handler处理格式
file_handler.setLevel(30)  # 设置handler级别,默认为0

# file_handler.addFilter(my_filter)  # 添加filter
# file_handler.removeFilter(my_filter)  # 移除filter

log1.addHandler(file_handler)  # 给log1绑定一个handler
# log1.removeHandler(file_handler)  # 移除指定handler

log1.setLevel(20)  # 设置自己的日志级别
print(1, log1.level)
print(2, log1.getEffectiveLevel())
print(3, file_handler.level)

log1.debug('debug msg...')
log1.info('info msg...')
log1.warning('warning msg...')
log1.error('error msg...')
log1.critical('critical msg...')
```

handler的初始的level是0，并且只认自己的level。当handler的level为0时，按照log1的级别记录，如果log1为0，按照根级别记录。

### 常用的Handler

每个Logger可以附加多个Handler。接下来我们就来介绍一些常用的Handler：

1) logging.StreamHandler

使用这个Handler可以向类似与sys.stdout或者sys.stderr的任何文件对象(file object)输出信息。它的构造函数是：

StreamHandler([strm])

其中strm参数是一个文件对象。默认是sys.stderr

2) logging.FileHandler

和StreamHandler类似，用于向一个文件输出日志信息。不过FileHandler会帮你打开这个文件。它的构造函数是：

FileHandler(filename[,mode])

filename是文件名，必须指定一个文件名。

mode是文件的打开方式。参见Python内置函数open()的用法。默认是’a'，即添加到文件末尾。

3) logging.handlers.RotatingFileHandler

这个Handler类似于上面的FileHandler，但是它可以管理文件大小。当文件达到一定大小之后，它会自动将当前日志文件改名，然后创建 一个新的同名日志文件继续输出。比如日志文件是chat.log。当chat.log达到指定的大小之后，RotatingFileHandler自动把 文件改名为chat.log.1。不过，如果chat.log.1已经存在，会先把chat.log.1重命名为chat.log.2。。。最后重新创建 chat.log，继续输出日志信息。它的构造函数是：

RotatingFileHandler( filename[, mode[, maxBytes[, backupCount]]])

其中filename和mode两个参数和FileHandler一样。

maxBytes用于指定日志文件的最大文件大小。如果maxBytes为0，意味着日志文件可以无限大，这时上面描述的重命名过程就不会发生。

backupCount用于指定保留的备份文件的个数。比如，如果指定为2，当上面描述的重命名过程发生时，原有的chat.log.2并不会被更名，而是被删除。

4) logging.handlers.TimedRotatingFileHandler

这个Handler和RotatingFileHandler类似，不过，它没有通过判断文件大小来决定何时重新创建日志文件，而是间隔一定时间就 自动创建新的日志文件。重命名的过程与RotatingFileHandler类似，不过新的文件不是附加数字，而是当前时间。它的构造函数是：

TimedRotatingFileHandler( filename [,when [,interval [,backupCount]]])

其中filename参数和backupCount参数和RotatingFileHandler具有相同的意义。

interval是时间间隔。

when参数是一个字符串。表示时间间隔的单位，不区分大小写。它有以下取值：

S 秒

M 分

H 小时

D 天

W 每星期（interval==0时代表星期一）

midnight 每天凌晨

# 通过文件配置logging

如果你希望通过配置文件来管理logging，可以参考这个官方文档。在log4net或者log4j中这是很常见的方式。

 

```
# logging.conf
[loggers]
keys=root
 
[logger_root]
level=DEBUG
handlers=consoleHandler
#,timedRotateFileHandler,errorTimedRotateFileHandler
 
#################################################
[handlers]
keys=consoleHandler,timedRotateFileHandler,errorTimedRotateFileHandler
 
[handler_consoleHandler]
class=StreamHandler
level=DEBUG
formatter=simpleFormatter
args=(sys.stdout,)
 
[handler_timedRotateFileHandler]
class=handlers.TimedRotatingFileHandler
level=DEBUG
formatter=simpleFormatter
args=('debug.log', 'H')
 
[handler_errorTimedRotateFileHandler]
class=handlers.TimedRotatingFileHandler
level=WARN
formatter=simpleFormatter
args=('error.log', 'H')
 
#################################################
[formatters]
keys=simpleFormatter, multiLineFormatter
 
[formatter_simpleFormatter]
format= %(levelname)s %(threadName)s %(asctime)s:   %(message)s
datefmt=%H:%M:%S

[formatter_multiLineFormatter]
format= ------------------------- %(levelname)s -------------------------
 Time:      %(asctime)s
 Thread:    %(threadName)s
 File:      %(filename)s(line %(lineno)d)
 Message:
 %(message)s
 
datefmt=%Y-%m-%d %H:%M:%S
```

假设以上的配置文件放在和模块相同的目录，代码中的调用如下：

 

```
import os
filepath = os.path.join(os.path.dirname(__file__), 'logging.conf')
logging.config.fileConfig(filepath)
return logging.getLogger()
```

### 日志重复输出问题

因为有一个日志传递

解决办法：

log1.propagate = False

你有可能会看到你打的日志会重复显示多次，可能的原因有很多，但总结下来无非就一个，日志中使用了重复的handler。

第一坑

 

```
import logging
logging.basicConfig(level=logging.DEBUG)
fmt = '%(levelname)s:%(message)s'
console_handler = logging.StreamHandler()
console_handler.setFormatter(logging.Formatter(fmt))
logging.getLogger().addHandler(console_handler)
logging.info('hello!')
# INFO:root:hello!
# INFO:hello!
```

上面这个例子出现了重复日志，因为在第3行调用basicConfig()方法时系统会默认创建一个handler，如果你再添加一个控制台handler时就会出现重复日志。

第二坑

 

```
import logging

def get_logger():
    fmt = '%(levelname)s:%(message)s'
    console_handler = logging.StreamHandler()
    console_handler.setFormatter(logging.Formatter(fmt))
    logger = logging.getLogger('App')
    logger.setLevel(logging.INFO)
    logger.addHandler(console_handler)
    return logger

def call_me():
    logger = get_logger()
    logger.info('hi')

call_me()
call_me()

# INFO:hi
# INFO:hi
# INFO:hi
```

在这个例子里hi居然打印了三次，如果再调用一次call_me()呢？我告诉你会打印6次。why? 因为你每次调用get_logger()方法时都会给它加一个新的handler，你是自作自受。正常的做法应该是全局只配置logger一次。

第三坑

 

```
import logging

def get_logger():
    fmt = '%(levelname)s: %(message)s'
    console_handler = logging.StreamHandler()
    console_handler.setFormatter(logging.Formatter(fmt))
    logger = logging.getLogger('App')
    logger.setLevel(logging.INFO)
    logger.addHandler(console_handler)
    return logger

def foo():
    logging.basicConfig(format='[%(name)s]: %(message)s')
    logging.warn('some module use root logger')

def main():
    logger = get_logger()
    logger.info('App start.')
    foo()
    logger.info('App shutdown.')

main()

# INFO: App start.
# [root]: some module use root logger
# INFO: App shutdown.
# [App]: App shutdown.
```

为嘛最后的App shutdown打印了两次？所以在Stackoverflow上很多人都问，我应该怎么样把root logger关掉，root logger太坑爹坑妈了。只要你在程序中使用过root logger，那么默认你打印的所有日志都算它一份。上面的例子没有什么很好的办法，我建议你找到那个没有经过大脑就使用root logger的人，乱棍打死他或者开除他。

如果你真的想禁用root logger，有两个不是办法的办法：

logging.getLogger().handlers = []  # 删除所有的handler

logging.getLogger().setLevel(logging.CRITICAL)  # 将它的级别设置到最高

logger.addFilter(filt) , Logger.removeFilter(filt)  # 添加或删除指定的filter

# 日志流(重要)

## level的继承

 

```
FORMAT = '%(asctime)s ** %(message)s'
logging.basicConfig(format=FORMAT, level=10)  # 根logger级别

FMT = logging.Formatter('%(levelname)s %(asctime)s %(name)s %(message)s')
root = logging.getLogger()

log1 = logging.getLogger('s')
log1.setLevel(20)  # 分别修改20,30,40试试

log2 = logging.getLogger('s.s1')
log2.info('log2 info.')
log2.warning('log2 warning.')
```

logger实例，如果设置了level，就用自己的level和信息的级别比较，否则，就会继承最近的祖先的level。

## 继承关系

每一个Logger实例的level如同入口，让水流进来，如果第一道门槛太高，信息就进不来。例如log3.warning('log3'),如果log3定义的级别高，就不会有信息通过log3.

如果level没有设置，就用父logger的，如果父也没设置，继续找父的父，最终找到root上，root默认值是WARNING。

## 信息传递

见下图

![img](logging.assets/ec8a4bc7-a1ab-4d28-957a-6134824ab494.png)

![img](logging.assets/7a603536-59e9-4a63-b747-d6913df7e06e.png)