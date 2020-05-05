[TOC]

# time

在Python中,通常有这几种方式来表示时间：

1,时间戳(timestamp):通常来说,时间戳表示的是从计算机元年1970年1月1日00:00:00开始按秒计算的偏移量,我们运行'type(time.time())',返回的是float类型;

2,格式化的时间字符串(Format String)

3,结构化的时间(struct_time):struct_time元组共有9个元素共九个元素:(年，月，日，时，分，秒，一年中第几周，一年中第几天，夏令时)

示例

 

```
import time   # 我们先以当前时间为准，快速认识三种形式的时间
print(time.time())    # 时间戳:1487130156.419527,计算机用的时间
print(time.strftime("%Y-%m-%d %X")) # 格式化的时间字符串:'2017-02-15 11:40:53'
print(time.localtime()) # 本地时区的struct_time,结构化的时间
print(time.gmtime())    # UTC时区的struct_time,标准时间

# 运行结果:
"""
1496747605.4952035
2017-06-06 19:13:25
time.struct_time(tm_year=2017, tm_mon=6, tm_mday=6, tm_hour=19, tm_min=13, tm_sec=25, tm_wday=1, tm_yday=157, tm_isdst=0)
time.struct_time(tm_year=2017, tm_mon=6, tm_mday=6, tm_hour=11, tm_min=13, tm_sec=25, tm_wday=1, tm_yday=157, tm_isdst=0)
"""
```

其中计算机认识的时间只能是'时间戳'格式，而程序员可处理的或者说人类能看懂的时间有: '格式化的时间字符串'，'结构化的时间' ，于是有了下图的转换关系：

![img](time%20&%20datetime.assets/51042b26-4a1f-4b6a-8fbf-9fcc97b084e0.jpg)

![img](time%20&%20datetime.assets/4f46cd08-0be8-40fb-ae4e-fed230de4f8c.jpg)

转换示例:

 

```
print(time.localtime(time.time())) # 将当前时间戳转换成结构化的时间
# 运行结果:time.struct_time(tm_year=2017, tm_mon=6, tm_mday=6, tm_hour=21, tm_min=0, tm_sec=25, tm_wday=1, tm_yday=157, tm_isdst=0)

print(time.mktime(time.localtime())) # 将当前的结构化时间转换为时间戳格式的时间
# 运行结果:1496754025.0

print(time.strftime('%Y-%m-%d %X', time.localtime())) # 将当前的结构化的时间转换为格式化的字符串时间
# 运行结果:2017-06-06 21:00:25

print(time.strptime('2017-06-06 11:59:59', '%Y-%m-%d %X'))  # 将字符串时间转换为结构化的时间
# 运行结果:time.struct_time(tm_year=2017, tm_mon=6, tm_mday=6, tm_hour=11, tm_min=59, tm_sec=59, tm_wday=1, tm_yday=157, tm_isdst=-1)

print(time.ctime(time.time()))  # 将当前时间戳转换成linux形式的时间格式
# 运行结果:Tue Jun  6 21:07:21 2017

print(time.asctime(time.localtime()))  # 将当前结构化的时间转换为linux形式的时间格式
# 运行结果:Tue Jun  6 21:07:21 2017

print(time.asctime())  # linux格式的时间
# 运行结果:Tue Jun  6 21:04:54 2017

print(time.strftime('%a %b %d %H:%M:%S %Y', time.localtime()))
# 运行结果:Tue Jun 06 21:04:54 2017
```

time.sleep(t): 将调用线程挂起指定的秒数，就是延迟几秒在运行

 

```
import time
def foo():
    start_time = time.time()
    print('reday~~')
    time.sleep(2)
    print('go!!')
    stop_time = time.time()
    print(stop_time - start_time)

foo()
"""
运行结果:
reday~~
此处等待2s..
go!!
2.0006494522094727
"""
```

# datetime

对日期、时间、时间戳的处理

## datetime类

### 类方法

today() 返回本地时区当前时间的datetime对象

now(tz=None) 返回当前时间的datetime对象，时间到微秒，如果tz为None，返回

和today()一样

utcnow() 没有时区的当前时间

fromtimestamp(timestamp, tz=None) 从一个时间戳返回一个datetime对象

### datetime对象

timestamp() 返回一个到微秒的时间戳。

时间戳：格林威治时间1970年1月1日0点到现在的秒数

构造方法datetime.datetime(2016, 12, 6, 16, 29, 43, 79043)

year、month、day、hour、minute、second、microsecond，取datetime对象的年月日时

分秒及微秒

weekday() 返回星期的天，周一0，周日6

isoweekday() 返回星期的天，周一1，周日7

date() 返回日期date对象

time() 返回时间time对象

replace() 修改并返回新的时间

isocalendar() 返回一个三元组(年，周数，周的天)

## 日期格式化*

类方法strptime(date_string, format) ，返回datetime对象

对象方法strftime(format) ，返回字符串

字符串format函数格式化

 

```
import datetime
dt = datetime.datetime.strptime("21/11/06 16:30", "%d/%m/%y %H:%M")
print(dt)
print(dt.strftime("%Y-%m-%d %H:%M:%S"))  # 2006-11-21 16:30:00
print("{0:%Y}/{0:%m}/{0:%d} {0:%H}::{0:%M}::{0:%S}".format(dt))  # 2006/11/21 16::30::00

```

上面"{0:%Y}/{0:%m}/{0:%d} {0:%H}::{0:%M}::{0:%S}".format(dt)中的0不可省略，如果不写则需要写成format(dt,dt,dt,dt,dt,dt)

## timedelta对象

时间差

datetime2 = datetime1 + timedelta

datetime2 = datetime1 - timedelta

timedelta = datetime1 - datetime2

构造方法

 

```
datetime.timedelta(days=0, seconds=0, microseconds=0, milliseconds=0,
minutes=0, hours=0, weeks=0)
year = datetime.timedelta(days=365)
print(year.total_seconds())  # 返回时间差的总秒数
```

# 示例

 

```
#1、返回昨天日期
def getYesterday():
    today=datetime.date.today()
    oneday=datetime.timedelta(days=1)
    yesterday=today-oneday
    return yesterday

# 2、返回今天日期
def getToday():
    return datetime.date.today()

# 3、获取给定参数的前几天的日期，返回一个list
def getDaysByNum(num):
    today=datetime.date.today()
    oneday=datetime.timedelta(days=1)
    li=[]       
    for i in range(0,num):
        # 今天减一天，一天一天减
        today=today-oneday
        # 把日期转换成字符串
        # result=datetostr(today)  
        li.append(datetostr(today))
    return li

# 4、将字符串转换成datetime类型
def strtodatetime(datestr,fmt):
    return datetime.datetime.strptime(datestr,fmt)

# 5、时间转换成字符串,格式为2008-08-02
def datetostr(date):
    return str(date)[0:10]

# 6、两个日期相隔多少天，例：2008-10-03和2008-10-01是相隔两天  
def datediff(beginDate,endDate):
    format="%Y-%m-%d"
    bd=strtodatetime(beginDate,format)
    ed=strtodatetime(endDate,format)
    oneday=datetime.timedelta(days=1)
    count=0
    while bd!=ed:
        ed=ed-oneday
        count+=1
    return count

# 7、获取两个时间段的所有时间,返回list
def getDays(beginDate,endDate):
    format="%Y-%m-%d";
    bd=strtodatetime(beginDate,format)
    ed=strtodatetime(endDate,format)
    oneday=datetime.timedelta(days=1)
    num=datediff(beginDate,endDate)+1
    li=[]
    for i in range(0,num):
        li.append(datetostr(ed))
        ed=ed-oneday
    return li

# 8、获取当前年份 是一个字符串
def getYear():
    return str(datetime.date.today())[0:4]

# 9、获取当前月份 是一个字符串
def getMonth():
    return str(datetime.date.today())[5:7]

# 10、获取当前天 是一个字符串
def getDay():
    return str(datetime.date.today())[8:10]
def getNow():
    return datetime.datetime.now()

print(getToday())
print(getYesterday())
print(getDaysByNum(3))
print(getDays('2008-10-01','2008-10-05'))
print('2008-10-04 00:00:00'[0:10])

print(str(getYear())+getMonth()+getDay())
print(getNow())

# 11、将字符串格式化成时间  
import datetime
>>> s="2006-1-2"
print(datetime.datetime.strptime(s,"%Y-%m-%d"))
"2006-01-02 00:00:00"
import time
>>> s="2006-1-2"
>>> time.strptime(s,"%Y-%m-%d")
>>> from time import *
>>> strftime("%Y-%m-%d %H:%M:%S", localtime())
'2011-10-12 03:00:58'

# 12、将格式字符串转换为时间戳
>>> a = "Sat Mar 28 22:24:24 2009"
>>> b = mktime(strptime(a,"%a %b %d %H:%M:%S %Y"))
>>> print(b)
"1238250264.0"


－－－－－－－－－－－－－－－－－
# 13、 DateTime示例
# 演示计算两个日期相差天数的计算
>>> import datetime
>>> d1 = datetime.datetime(2005, 2, 16)
>>> d2 = datetime.datetime(2004, 12, 31)
>>> (d1 - d2).days
"47"

# 演示计算运行时间的例子，以秒进行显示
import datetime
starttime = datetime.datetime.now()
# long running
endtime = datetime.datetime.now()
print((endtime - starttime).seconds)

# 14、python取前几天的日期
>>> from datetime import timedelta, date
>>> print(date.today() + timedelta(days = -2))  # 是不是有点类似 date -d 呢
"""
2011-10-09
"""

# 演示计算当前时间向后10小时的时间。
>>> d1 = datetime.datetime.now()
>>> d3 = d1 + datetime.timedelta(hours=10)
>>> d3.ctime()
# 其本上常用的类有：datetime和timedelta两个。它们之间可以相互加减。每个类都有一些方法和属性可以查看具体的值。

# 15、根据一个起始天数，返回相对今天的日期列表。如 MyDate(0).getDaysByNum(1, 7)将得到从昨天开始一周内的日期列表。
class MyDate：
    def __init__ (self, i):
        self.i = i
    def getDaysByNum(self, st, en):
        today = datetime.date.today() + datetime.timedelta(-self.i)
        oneday = datetime.timedelta(days=1)
        global yesterday
        yesterday = today - oneday
        li = []
        for i in range(0, en):
            today = today - oneday
            li.append(str(today).replace("-",""))
        return li[st-1:en]   

# 16、glob：可以使用简单的方法匹配某个目录下的所有子目录或文件，用法也很简单。
3.1 glob.glob(regression)  # 返回一个列表
3.2 glob.iglob(regression)  # 返回一个遍历器
```