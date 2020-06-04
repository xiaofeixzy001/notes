[TOC]

**Linux时钟**

系统时钟(System Clock)

系统时钟是指当前Linux Kernel中的时钟

硬件时钟（Real Time Clock，简称RTC）

硬件时钟则是主板上由电池供电的时钟，这个硬件时钟可以在BIOS中进行设置

当Linux启动时，硬件时钟会去读取系统时钟的设置，然后系统时钟就会独立于硬件运作.

**关于时钟相关的命令**

cal

显示日历

-1:显示一个月的月历

-3:显示3个月的月历,当月,上一个月和下一个月

-y:显示年历

-j:显示一年中的天数,从当年1月1日算起.

date

显示系统时间

/etc/sysconfig/clock

时钟配置文件,可修改时区

显示系统时钟

date --help

date [options] [+FORMAT]

options

> -s:
>
> -R：显示系统时间和时区

FORMAT

> +%F，+%D：仅显示日期，即显示年月日
>
> +%T：仅显示时间，即显示时分秒
>
> +%Y：仅显示年份
>
> +%m：仅显示月份
>
> +%d：仅显示日期
>
> +%H：仅显示小时
>
> +%M：仅显示分钟
>
> +%S：仅显示秒
>
> +%z:显示时区
>
> +%s：时间戳计时法，从Unix元年（1970年1月1号0点0分0秒）到现在经过了多少秒

修改系统时钟

 

```
#设置当前时间为2016年11月21日17点20分30秒
date 112117202016.30  

#修改系统时钟为15年5月25日
date -s 05/25/15

#修改时间为12点00分00秒，日期不变
date -s 12:00:00

#修改全部的时间
date -s "2015-05-25 12:00:00"

#查看n天前的时间：(如要查n天后，改为-n即可)
date -d "n days ago" +%Y-%m-%d
```



修改时区：

 

```
#编辑配置文件,修改ZONE行
vim /etc/sysconfig/clock
 ZONE="America/New_York"

#或直接覆盖
cp /usr/share/zoneinfo/Hongkong /etc/localtime
```



**clock|hwclock**

显示硬件时钟

  -s：以硬件时钟为准，系统时间同步为硬件时间

  -w：以系统时钟为准，硬件时间同步为系统时间

 

```
#设置硬件时钟为15年5月25日12点12分
hwclock --set --date="05/25/15 12:12"
```



\#rpm -ql tadata

/usr/share/zoneinfo        ###为各时区的时间格式文件

相关的配置文件还有/etc/sysconfig/clock，/etc/localtime

\#vim /etc/sysconfig/clock      ###设置时区是否使用UTC时钟的配置文件。每次开机后系统会自动读取此文件来设置默认时间。

  ZONE="Asia/Shanghai"       ###

\#vim /etc/localtime         ###本地端的时间配置文件，里面是内容同

例如：

如果要去美国出差，笔记本时间当前为中国时区，与美国时区不一致，需要如何校正？

\#vim /etc/sysconfig/clock

 ZONE="America/New_York"

\#cp /usr/share/zoneinfo/America/New_York /etc/localtime

\#data

ntpdate

ntpdate Server-IP    \\后面跟服务器地址，同步网络服务器时间