[TOC]

# grep

global search regular expression(RE) and print out the line (全面搜索正则表达式并把行打印出来)

文本搜索工具,根据用户指定的"模式"对目标文本逐行进行匹配检查,并打印匹配到的行;

"模式":是由正则表达式字符及文本字符所编写的过滤条件

grep家族包括grep、egrep和fgrep.

grep: 支持基本正则表达式

egrep: 支持扩展正则表达式，等同于 grep -E

fgrep: fixed grep或fast grep,快速过滤,不再支持正则，等同于 grep -F

## 使用格式

grep [OPTIONS] PATTERN [FILE...]

grep  [OPTIONS]  [-e  PATTERN  |  -f   FILE] [FILE...]

## OPTIONS

-A [行数n] (after)显示匹配字符所在行的后n行

-B [行数n] (before)显示前n行

-C [行数n] (centext)显示前后各n行

-E 相当于egrep,支持扩展正则

-F 相当于fgrep,不支持正则

-G 相当于grep,仅支持基本正则

-c 统计匹配结果

-d<进行动作> 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep命令将回报信息并停止动作

-h 在显示符合范本样式的那一列之前，不标示该列所属的文件名称

-H 在显示符合范本样式的那一列之前，标示该列的文件名称

-i 忽略字符大小写

-n 显示行号

-q 静默输出,不会输出任何信息,成功匹配返回0,否则返回非0,一般用于条件测试

-s 不显示错误信息

-v 反向查找,即显示没有匹配到的内容

-o 仅输出匹配到的字串

--color=auto 可以将找到的关键词部分加上颜色的显示

注:在关键字的显示方面,可以在 ~/.bashrc文件内加上这行：alias grep='grep --color=auto'，再以source ~/.bashrc 来立即生效即可

 

```
vi .bashrc
"""
alias rm='rm -i'
alias cp='cp -vi'
alias mv='mv -vi'
alias mkdir='mkdir -pv'
alias cdnet='cd /etc/sysconfig/network-scripts/'
alias grep='grep --color'
"""

# 使修改生效
source .bashrc
```



简单示例:

 

```
# 将/etc/passwd，将没有出现 root 的行取出来
grep -v root /etc/passwd

# 将/etc/passwd，将没有出现 root 和nologin的行取出来
grep -v root /etc/passwd | grep -v nologin

# 用 dmesg 列出核心信息，再以 grep 找出内含 eth 那行,在关键字所在行的前两行与后三行也一起捉出来显示，-n显示行号
dmesg | grep -n -A3 -B2 --color=auto 'eth'
```



## PATTERN

正则表达式是一类字符所书写的模式，其中许多字符不表示其字面意义，而是表达控制或通配功能

grep 'PATTERN' FILE_NAME

基本正则表达式元字符

字符匹配

匹配次数

位置锚定

扩展正则表达式元字符与基本正则区别不大,只是一些原本需要转义的符号,在扩展正则中则不再需要

### 字符匹配

. 匹配任意单个字符

\* 任意长度字符

.* 匹配任意长度的任意字符

[] 匹配指定范围内的任意字符：

[^] 取反

[0-9] <=> [[:digit:]] 所有数字

[a-z] <=> [[:lower:]] 所有小写字母

[A-Z] <=> [[:upper:]] 所有大写字母

[[:space:]] 空格

[[:punct:]] 所有标点符号

[[:alpha:]] 所有大小写字母

[[:alnum:]] 所有字母和数字符号

### 匹配次数

用于指定其前面的字符所能够出现的次数,通过分组的方式可对不止一个字符做次数匹配

\*  匹配任意次数(0次,1次,多次)

例：grep "x*y"

y的前面出现任意次数x,都会被显示

\?  表示匹配前面的字符0次或1次,即前面的字符是可有可无的,花括号为特殊符号,需要做转义

例：grep "x\?y"

y的前面出现0次或1次x都会被显示

\{m\}  匹配前面的字符m次,花括号为特殊符号,需要做转义

例：grep "x\{3\}y"

y的前面出现3次x，都会被显示，即便是xxxxxxy也会被显示

\{m,n\}  至少m次，至多n次

\{m,\}  匹配前面的字符至少出现m次

\{0,n\}  匹配前面的字符至多出现n次

例：grep "x\{2,5\}y"

y前面至少有2个x，至多5个x都会被匹配

### 位置锚定

^：行首锚定,写在搜索条件最左侧

例：grep "^xy"

$：行尾锚定,写在搜索条件最右侧

例：grep "xy$"

^PATTERN$  全匹配

^$  空白行

不包含特殊字符的连续字符组成的串叫单词

\<  词首锚定,出现于单词的左侧

\>  词尾锚定,出现于单词的右侧

\b  出现在单词首部,表示词首锚定,出现在单词尾部,表示词尾锚定

例: 

grep "\<xy"  单词首部是xy的匹配成功

grep "xy\>"  单词尾部是xy的匹配成功

grep "\bxy\b"  整个单词为xy 匹配

### 分组

\(\)：匹配多个字符,小括号为特殊符号,需要做转义

如,匹配ab，则表示成\(ab\)*

示例:

在文件passwd中找出至少匹配ro字串一次

grep "\(ro\)\{1,\}" /etc/passwd

### 引用

分组括号中的模式,匹配到的内容会被正则记录于内部变量中,这些变量的命名方式为\1,\2,\3,...

\1是指从左侧开始,第一个左括号以及与之匹配的右括号之间的模式所匹配到字符

例:

\(ab\+\(xy\)*\)

\1表示的是"ab\+\(xy\)*"

\2表示的是"xy"

\#:  引用前面的分组括号中的模式所匹配字符(而非模式本身)

例如\(ab\?c\).*\1  意思是，第一个括号内的结果匹配到\后面，括号内的结果是abc，那么\后面也是abc，即只要是abc.*abc即可被搜索到

## 总结

字符匹配：

  . 任意单个字符

  [] 范围内的字符

  [^] 范围外的任意字符

次数匹配：  

  *：任意次

  ?：0次或1次

  +：至少1次

  {m}：精确匹配m次

  {m,n}：至少m次，至多n次

  {m,}：至少m次  

  {0，n}：至多n次

位置锚定：

  ^ ：锚定行首

  $ ：锚定行尾

  \<,\b：锚定词首

  \>,\b：锚定词尾

  ^$ ：匹配空行

  ^[[:space:]]*$：匹配任意空白行

分组：

  \(\)

引用分组(后向引用)：

  \1,\2,\3...

或者：

  |  整个左侧或整个右侧

  例如: 

​    a|b：a或b

​    C|cat: C或cat

​    (C|c)at: Cat或cat

# 练习

 

```
# 显示/proc/meminfo文件中以大小s开头的行
grep -i '^s' /proc/meminfo
grep '^[sS]' /proc/meminfo

# 显示/etc/passwd文件中不以/bin/bash结尾的行
grep -v '/bin/bash$' /etc/passwd

# 找出/etc/passwd中的两位或三位数
grep '\b[[:digit:]]\{2,3\}\b' /etc/passwd
grep -E '\b[[:digit:]]{2,3}\b' /etc/passwd

# 显示/etc/rc.d/rc.sysinit文件中,至少以一个空白字符开头的且后面存在非空白字符的行
grep '^[[:space:]]\+[^[:space:]]' /etc/rc.d/rc.sysinit

# 找出"netstat -tan"命令的结果中以"LISTEN"后跟0,1或多个空白字符结尾的行
netstat -tan | grep 'LISTEN[[:space:]]*$'

# 显示/etc/passwd文件中id号最大的用户的用户名
sort -t: -k3 -n /etc/passwd | tail -1 | cut -d: -f1

# 如果用户root存在,显示其默认的shell程序
id root &> /dev/null && grep '^root\b' /etc/passwd | cut -d: -f 7

# 添加用户bash,testbash,basher以及nologin(其shell为/sbin/nologin),而后找出/etc/passwd文件中用户名同shell名的行
useradd bash
useradd testbash
useradd basher
useradd -s /sbin/nologin nologin
grep '^\([[:alnum:]]\+\b\).*\1$' /etc/passwd  # \1引用了前面括号中的匹配结果


# 显示当前系统root,centos或user1用户的默认shell和UID
grep -E '^(root|centos|user1)\b' /etc/passwd | cut -d: -f1,3,7

# 找出/etc/rc.d/init.d/functions文件中某单词后面跟一组小括号的行,如 hello()
grep -E -o '^[_[:alpha:]]+\(\)' /etc/rc.d/init.d/functions

# 使用echo输出绝对路径,使用egrep取出其基名
echo '/mnt/sdc' | grep -E -o '[^/]+/?$' | cut -d'/' -f1

# 使用egrep取出路径的目录名,类似dirname命令
echo "/etc/rc.d/init.d/sshd" | grep -E -o '/.*/'

# 找出ifconfig命令结果中1-255之间的数值
ifconfig | grep -E '\b([1-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\b'

# 找出ifconfig命令结果中的ip地址
ifconfig | grep "Bcast" | cut -d":" -f2 | cut -d" " -f1\

# 查出分区空间使用率的最大百分比值
df | grep "/dev/[sh]d" | tr -s ' ' '%'| cut -d'%' -f5 | sort -nr | head -1

# 列出/tmp的权限,以数字方式显示
stat /tmp | grep "(" | cut -d'(' -f2 | cut -d'/' -f1

# 统计当前连接本机的每个远程主机IP的连接数,并按从大到小排序
netstat -tn | grep 'tcp' | tr -s ' ' | cut -d' ' -f5 | cut -d: -f1 | sort -t '.' -k4 | uniq -c | sort -nr

# 统计以root身份登录的每个远程主机ip地址的登录次数
last | egrep "^root.*(([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5]).*" | tr -s " " | cut -d" " -f3 | uniq -c

# 获取分区利用率
df | grep "/dev/[sh]d" | tr -s ' ' '%' | cut -d% -f5 | sort -nr

# 匹配合理的ip地址
ifconfig | grep -E -o "(\<([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\>.){3}\<([0-9]|[1-9][0-9]|1[0-9][0-9]|2[0-4][0-9]|25[0-5])\>"

# 匹配出所有的邮件地址
grep -E '^([a-zA-Z0-9_-\.+]+@[a-zA-Z0-9_.]+\.[a-zA-Z]{2,5})$'
```