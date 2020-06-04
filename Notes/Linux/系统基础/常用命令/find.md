[TOC]

# find

实时查找工具,通过遍历指定路径下的文件系统完成文件查找

## 工作特点

查找速度略慢

精确查找

实时查找

用来在指定目录下查找文件,任何位于参数之前的字符串都将被视为欲查找的目录名,如果使用该命令时,不设置任何参数,则find命令将在当前目录下查找子目录与文件,并且将查找到的子目录和文件全部进行显示.

## 语法

find [-H] [-L] [-P] [-D debugopts] [-Olevel] [path...] [expression]

find [选项] [查找路径] [查找条件] [处理动作]

## options

-amin ：查找在指定时间曾被存取过的文件或目录，单位以分钟计算；

-anewer ：查找其存取时间较指定文件或目录的存取时间更接近现在的文件或目录

-atime ：查找在指定时间曾被存取过的文件或目录，单位以24小时计算

-cmin ：查找在指定时间之时被更改过的文件或目录

-cnewer :查找其更改时间较指定文件或目录的更改时间更接近现在的文件或目录

-ctime ：查找在指定时间之时被更改的文件或目录，单位以24小时计算

-daystart：从本日开始计算时间

-depth：从指定目录下最深层的子目录开始查找

-empty：长度为零的文件 

-expty：寻找文件大小为0 Byte的文件，或目录下没有任何子目录或文件的空目录

-exec ：假设find指令的回传值为True，就执行该指令

-false：将find指令的回传值皆设为False

-fls ：此参数的效果和指定“-ls”参数类似，但会把结果保存为指定的列表文件

-follow：排除符号连接

-fprint ：此参数的效果和指定“-print”参数类似，但会把结果保存成指定的列表文件

-fprint0 ：此参数的效果和指定“-print0”参数类似，但会把结果保存成指定的列表文件

-fprintf ：此参数的效果和指定“-printf”参数类似，但会把结果保存成指定的列表文件

-fstype ：只寻找该文件系统类型下的文件或目录

-gid ：查找符合指定之群组识别码的文件或目录

-group ：查找符合指定之群组名称的文件或目录

-help或——help ：在线帮助

-ilname ：此参数的效果和指定“-lname”参数类似，但忽略字符大小写的差别

-iname ：此参数的效果和指定“-name”参数类似，但忽略字符大小写的差别

-inum ：查找符合指定的inode编号的文件或目录

-ipath ：此参数的效果和指定“-path”参数类似，但忽略字符大小写的差别

-iregex ：此参数的效果和指定“-regexe”参数类似，但忽略字符大小写的差别

-links ：查找符合指定的硬连接数目的文件或目录

-iname ：指定字符串作为寻找符号连接的范本样式

-ls：假设find指令的回传值为Ture，就将文件或目录名称列出到标准输出

-maxdepth ：设置查找最大目录层级,默认3级

-maxdepth n: 基于目录深度搜索,n为数字：设置最小目录层级

-mmin ：查找在指定时间曾被更改过的文件或目录，单位以分钟计算

-mount：此参数的效果和指定“-xdev”相同=

-mtime ：查找在指定时间曾被更改过的文件或目录，单位以24小时计算

-name ：指定字符串作为寻找文件或目录的范本样式

-newer ：查找其更改时间较指定文件或目录的更改时间更接近现在的文件或目录

-nogroup ：找出不属于本地主机群组识别码的文件或目录

-noleaf ：不去考虑目录至少需拥有两个硬连接存在

-nouser ：找出不属于本地主机用户识别码的文件或目录

-ok ：此参数的效果和指定“-exec”类似，但在执行指令之前会先询问用户，若回答“y”或“Y”，则放弃执行命令

-path ：指定字符串作为寻找目录的范本样式

-perm ：查找符合指定的权限数值的文件或目录

-print ：假设find指令的回传值为Ture，就将文件或目录名称列出到标准输出。格式为每列一个名称，每个名称前皆有“./”字符串

-print0：假设find指令的回传值为Ture，就将文件或目录名称列出到标准输出。格式为全部的名称皆在同一行

-printf ：假设find指令的回传值为Ture，就将文件或目录名称列出到标准输出。格式可以自行指定 

-prune：不寻找字符串作为寻找文件或目录的范本样式

-regex ：指定字符串作为寻找文件或目录的范本样式

-size ：查找符合指定的文件大小的文件

-true：将find指令的回传值皆设为True=

-typ ：只寻找符合指定的文件类型的文件=

-uid ：查找符合指定的用户识别码的文件或目录=

-used ：查找文件或目录被更改之后在指定时间曾被存取过的文件或目录，单位以日计算=

-user ：查找符和指定的拥有者名称的文件或目录=

-version或——version：显示版本信息=

-xdev：将范围局限在先行的文件系统中=

-xtype ：此参数的效果和指定“-type”参数类似，差别在于它针对符号连接检查=

## 查找路径

指定具体目标路径,默认为当前目录

## 查找条件

指定的查找标准,可以文件名,大小,类型,权限等标准进行,默认查找目标路径下的所有文件

### 根据文件名查找

-name "文件名称" : 支持golb(*,?,[],[^])

-iname "文件名称" : 忽略大小写

-regex "PATTERN" : 使用正则表达式,以PATTERN匹配整个文件路径字符串,而不仅仅是文件名称

示例:

 

```
# 列出当前目录及子目录下所有文件和文件夹
find .

# 在/home目录下查找以.txt结尾的文件名
find /home -name "*.txt"

# 同上，但忽略大小写
find /home -iname "*.txt"

# 当前目录及子目录下查找所有以.txt和.pdf结尾的文件
find . \( -name "*.txt" -o -name "*.pdf" \)
# 或
find . -name "*.txt" -o -name "*.pdf" 

# 匹配文件路径或者文件
find /usr/ -path "*local*"

# 基于正则表达式匹配文件路径
find . -regex ".*\(\.txt\|\.pdf\)$"

# 同上，但忽略大小写
find . -iregex ".*\(\.txt\|\.pdf\)$"
```



### 根据属主属组查找

-user USERNAME: 查找属主为USERNAME的文件

-group GRPNAME: 查找属组为GRPNAME的文件

-uid UID: 查找属主为UID的文件

-gid GID: 查找属组为GID的文件

-nouser: 查找没有属主的文件

-nogroup: 查找没有属组的文件

示例:

 

```
find /home -user root -ls
find /etc -group root -ls
```



### 根据文件类型查找

find 路径 -type TYPE

TYPE的类型有:

f: 普通文件

d: 目录文件

l: 符号链接文件

s: 套接字文件

b: 块设备文件

c: 字符设备文件

p: 管道文件

示例:

 

```
# 搜索出深度距离当前目录至少2个子目录的所有文件
find . -mindepth 2 -type f

# 仅查找当前目录一级下的所有文件
find . maxdepth 1 -type f
```



### 根据文件大小来查找

-size [+,-]#UNIT

符号#为数字

文件大小单位(UNIT):

b: 块(512字节)

c: 字节

w: 字(2字节)

k: 千字节

M: 兆字节

G: 吉字节

-size #: (#-1, #]

-size -#: [0, #-1]

-size +#: (#, +oo)

示例:

 

```
# 搜索大于10KB的文件
find . -type f -size +10k

# 搜索小于10KB的文件
find . -type f -size -10k

# 搜索等于10KB的文件
find . -type f -size 10k
```



### 根据时间戳查找

每个文件都有三种时间戳

访问时间(-atime/天, -amin/分钟): 用户最近一次访问时间。

修改时间(-mtime/天, -mmin/分钟): 文件最后一次修改时间。

变化时间(-ctime/天, -cmin/分钟): 文件数据元（例如权限等）最后一次修改时间。

#### 以'天'为单位

-atime [+,-]n

-mtime [+,-]n

-ctime [+,-]n

#### 以'分钟'为单位

-amin [+,-]n

-mmin [+,-]n

-cmin [+,-]n

n: [n,n+1], n为数字,意义为在n天之前的一天以内被更改过的文件

+n: [n+1,+oo], 列出n天之前（不含n天本身）被更改过的文件名

-n: [0,n)列出n天之内（含有n天本身）被更改过的文件名

比如现在是30号的中午12点整

n=1表示过去一天,即29号中午12点到30号的中午12点之前这段时间(过去第一个24小时)

n=3则表示过去第三天,即26号中午12点-27号中午12点之间的24小时内的时间

n=+3则表示过去三天以前的时间,即27号中午12点之前的时间

n=-3则表示过去三天以内的时间,即27号中午12点到现在的时间

示例:

 

```
# 搜索最近七天内被访问过的所有文件
find . -type f -atime -7

# 搜索恰好在七天前被访问过的所有文件
find . -type f -atime 7

# 搜索超过七天内被访问过的所有文件
find . -type f -atime +7

# 搜索访问时间超过10分钟的所有文件
find . -type f -amin +10

# 找出比file.log修改时间更长的所有文件
find . -type f -newer file.log
```



### 根据权限查找

-perm [+,-]mode

mode为 rwx

mode: 精确权限匹配

+mode: 任何一类(u,g,o)对象的权限中只要有一位匹配即可

-mode: 每一类对象都必须同时拥有为其指定的权限标准

## 处理动作

对符合条件的文件做什么操作,默认输出至屏幕

-print: 默认的处理动作,显示至屏幕

-ls: 类似于对查找到的文件执行'ls -l'命令

-delete: 删除查找到的文件(慎重)

-fls /path/to/somefile: 将查找到的所有文件的长格式信息保存至指定文件中

-ok COMMAND {} \; :对查找到的每个文件执行COMMAND命令,在执行COMMAND命令前,会有提示确认

-exec COMMAND {} \; :对查找到的每个文件执行COMMAND命令,在执行COMMAND命令前,不会有提示确认

{}: 用于引用查找到的文件名称本身

注意:

find传递命令给查找到的文件时,会一次性传递,而有些命令不能接受过多参数,从而会导致命令执行失败,可使用 find | xargs COMMAND 来规避此问题

示例:

 

```
# 找出当前目录下所有root的文件，并把所有权更改为用户tom
find . -type f -user root -exec chown tom {} \;

# 找出自己家目录下所有的.txt文件并删除,为了安全,这里使用ok方式,提示确认删除(y,n)
find $HOME/. -name "*.txt" -ok rm {} \;

# 查找当前目录下所有.txt文件并把他们拼接起来写入到all.txt文件中
find . -tpye f -name "*.txt" -exec cat {} \; > all.txt

# 将/var/log/目录中30天前的.log文件移动到/var/log/old目录中
find /var/log/ -type f -name "*.log" -exec cp -rv {} /var/log/old \;

# 找到/tmp目录下过去5分钟内发生变化的文件,并将其文件名修改为原文件名加上.bak
find /tmp -cmin -5 -exec mv {} {}.bak \;
```



## 否定参数

找出/home下不是以.txt结尾的文件

 

```
find /home ! -name "*.txt"
```



## 组和条件

-a: 与

-o: 或

-not,!: 非

!A -a !B = !(A -o B)

!A -o !B = !(A -a B)

例如:查找当前系统中没有属主或没有属组的文件或目录

 

```
# 由于-ls仅会显示一种条件的内容,所以这里加上括号分组,表示ls显示所有结果
find / \(-nouser -o -nogroup\) -ls

# 找出/tmp目录下,属主不是root且文件名不是fstab的文件
find /tmp \( -not -user root -a -not -name 'fstab' \) -ls
find /tmp -not \( -user root -o -name 'fstab' \) -ls
```



## 练习

 

```
# 找出/tmp目录下，属主不是root，且文件名不以f开头的文件 
find /tmp/ ! \( -user root -o -name "f*" \)

# 查找/etc/下，除/etc/sane.d目录的其它所有.conf后缀的文件
find /etc/ -path /etc/sane.d -a -prune -o -name "*.conf"

# 查找/etc/下,除/etc/sane.d目录和/etc/fonts目录的其他所有.conf后缀的文件
find /etc/ \( -path /etc/sane.d -o -path /etc/fonts \) -a -prune -o -name "*.conf"

# 查找/var目录下属主为root，且属组为mail的所有文件 
find /var -user root -a -group mail

# 查找/var目录下不属于root、lp、gdm的所有文件 
find /var ! \( -user root -o -user lp -o -user gdm \)

# 查找/var目录下最近一周内其内容修改过，同时属主不为root，也不是postfix的文件 
解:find /var -mtime -8 ! \( -user root -o -name "postfix" \) -ls

# 查找当前系统上没有属主或属组，且最近一个周内曾被访问过的文件 
find / -nouser -nogroup -atime -8

# 查找/etc目录下大于1M且类型为普通文件的所有文件 
find /etc -size +1M -type f

# 查找/etc目录下所有用户都没有写权限的文件 
find /etc -perm 444

# 查找/etc目录下至少有一类用户没有执行权限的文件 
find /etc ! -perm /222

# 查找/etc/init.d目录下，所有用户都有执行权限，且其它用户有写权限的文件
find /etc/init.d -perm -111 -perm -002

```



# locate 

查找文件或目录

依赖于实现构建的索引,索引的构建是在系统较为空间时自动进行(周期性任务),也可以手动构建更新

索引构建过程需要便利整个根文件系统,极消耗资源

locate命令其实是 find -name 的另一种写法,但是locate要比后者快得多,原因在于它不搜索具体目录,而是搜索一个数据库/var/lib/locatedb,这个数据库中含有本地所有文件信息

## 工作特点

查找速度快

模糊查找

非实时查找

## 使用格式

locate [OPTION]... PATTERN...

OPTION：

-d<目录>或--database=<目录>：指定数据库所在的目录

-u：更新slocate数据库

--help：显示帮助

--version：显示版本信息

### 安装

安装包名: mlocate

安装完成后需要执行一条命令(updatedb)为你的文件系统建立索引,否则你就得等到程序哪天自动执行了

 

```
yum list all | grep mlocate
yum install -y mlocate
updatedb &
```



这条命令会后台更新你的mlocate数据库，这数据库包含了你的文件系统的索引（This updates the mlocate database that indexes your files and forks it to the background (the ‘&’ forks it to the background）