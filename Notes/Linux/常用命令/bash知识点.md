[TOC]



# bash思维导图



![img](bash%E7%9F%A5%E8%AF%86%E7%82%B9.assets/ba44f551-71ac-4d8e-8f37-b71286adf7be.jpg)

 

# bash配置文件

## profile类

为交互式shell提供配置文件

  ~/.bash_profile  个人配置

  /etc/profile，/etc/profile.d/*.sh  全局配置

功用：定义环境变量；运行命令和脚本。

## bashrc类

为非交互式用户提供配置文件

  ~/.bashrc  个人配置

  /etc/bashrc  全局配置

功用：定义本地变量，别名。

## 登录式shell读取配置文件顺序

/etc/profile ---> /etc/profile.d/*.sh --> ~/.bash_profile --> ~/.bashrc --> /etc/bashrc

## 非登录式shell读取顺序

~/.bashrc --> /etc/bashrc --> /etc/profile.d/*.sh



## 让定义的配置文件立即生效

\#source /etc/profile

\#. /etc/bashrc



# bash的工作特点

## 命令执行状态返回值

每个命令执行结束后，会有“执行状态返回值”，有效范围：0-255

0：表示执行成功

1-255：表示执行失败

使用$? 可查看状态返回值

## 命令

格式：

COMMAND  OPTIONS  ARGUMENTS  \# 命令 选项 参数



使用type查看命令类型



内部命令：shell程序自带的命令，由shell内建提供

外部命令：独立的可执行程序文件，名字即为程序名



执行外部命令时查找方式

echo $PATH

/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin



## 选项

短格式选项-char

   -l, -d, -h, 

   -ldh

长格式选项

   --word, --long,  --directory, ...



## 参数

   命令的作用对象



## 快捷键

Ctrl+a：光标跳至命令行首

Ctrl+e：光标跳至命令行尾

Ctrl+u：删除光标之前的所有内容

Ctrl+k：删除光标之后的所有内容



# bash功能

## 文件名通配：globbing

### 特殊字符，元字符

不表示字符的表面意义，而是被匹配符合指定特征的字符串

*：匹配任意长度的字符，包括空

?：匹配任意单个字符，不包括空

[]：匹配指定范围[]内的任意单个字符

[^]取反，匹配指定范围[]之外的任意单个字符

### 字符集合

[:space:]表示所有空白字符

[[:space:]]表示匹配其中任意一个

[:punct:]所有的标点符号

[:lower:]所有小写字母a-z

[:upper:]所有大写字母A-Z

[:digit:]所有的数字 1-9

[:alnum:]所有的数字和字母字符

[:alpha:]所有的字母

命令补全，路径补全



## history

history

  -c清空历史记录

  -w保存缓存中的历史命令到历史文件中

  -a追加保存

  -d n删除第n条历史记录

export HISTCONTROL=ignorespace\\执行history命令时，不显示空格开头的命令

export HISTCONTROL=ignoredups\\忽略重复的命令

export HISTCONTROL=ignoreboth\\空格开头的重复的命令全不显示

位置：通常在用户家目录下.bash_history

!#：执行第几条历史命令

!!：执行上一条命令

!..：执行上一个..开头的命令

!$：调用上一条命令的最后一个参数

按ESC松开再按.号，即会重新调用上一个命令的参数



## 管道和重定向

输出重定向：

COMMAND > POSITION   覆盖输出

COMMAND > POSITION   在原有内容的后面追加输出

错误重定向：

COMMAND 2> POSITION  只有错误信息才会被重定向

COMMAND 2>> POSITION 

合并重定向：

COMMAND &> POSITION  无论正确与否全部重定向

COMMAND > POSITION 2> POSITION  

输入重定向：

COMMAND < POSITION  覆盖输入，是指把POSITION的内容定向给COMMAND，并显示COMMAND的执行结果

COMMAND << POSITION  追加输入，此处文档

常搭配EOF使用：

\#cat > /tmp/123.txt << EOF

\>Hello,world.

\>How are you?

\>EOF

![img](bash%E7%9F%A5%E8%AF%86%E7%82%B9.assets/c89796cb-af01-4460-b8bf-30eb6d7b76e8.png)

## 单双引号区别

  “”：双引号，弱引用，里面的变量会被替换

  ‘’：单引号，强引用，里面的所有字符都是字面量，直接输出

如果一个字串没有变量，单双引号没有区别

如果有变量，想要获取变量的结果，需要双引号，如果想要变量本身，那么使用单引号

如果是命令，需要使用反引号，才能得到该命令执行的结果



# bash变量

内存空间地址+数据，数据是放在内存中，即也叫有名称的内存地址



变量的赋值：向变量的存储空间中存储数据



定义一个变量后，也就确定了：

1，存储机制，如存储的是字符型还是数值型等；

2，存储空间

3，参与的运算方式，如数值型的加减乘除等；



bash对变量的机制：

1，所有都看作字符型

2，不支持浮点数据，需借用外部机制

3，变量无需事先声明，相当于，赋值和声明同时实现



命名规范：

1，不能以数字开头，只能使用字母，数字和下划线

2，变量名要做到见名知义，如userName=admin风格

3，不能使用程序中的关键字（保留字），如if，else，where，for等



查看变量：

查看当前shell进程中所有的本地变量

`set`

查看当前shell进程中的所有环境变量

`export`

`printenv`

`env`



取消赋值：unset VAR_NAME（注意，这里不需要加$）



本地变量：作用范围仅当前shell，对其他shell和当前shell的子进程都无效，重启后失效；

赋值：var_name=VALUE



环境变量：只作用于当前shell进程及其子进程



用户可自定义环境变量：        

bash有许多内置的环境变量，赋值时注意不要重名

export VAR_NAME=VALUE  # 定义一个环境变量

export VAR_NAME   # 导出一个变量为环境变量

readonly VAR_NAME  # 锁定变量为只读变量，在当前shell进程结束前无法更改

var_name=value --> export var_name

declare -x var_name=VALUE  # -x 表示定义为一个环境变量

如：declare -i var_name=VALUE # 将var_name定义为整形



局部变量：作用范围为shell脚本中的某片代码片，通常用于函数本地

local VAR_NAME=VALUE



位置变量

使用$1， $2， ...， $9， ${10}， ...

$0表示脚本文件路径本身；取文件名：basename $0

shift：把引用过的参数'踢'掉，在脚本中永远只使用$1引用下一个

例如：shift 2 ：踢2个，这样就可以在脚本中永远以$1和$2来引用后面的参数



特殊变量：shell内置，有特殊功用

$?  上个命令的状态返回值

$#  传递给脚本参数的个数,就是一共有多少个位置变量

$* 或$@  引用传递给脚本的所有参数



变量引用：

1，${var_name}

2，$var_name

3，`date`    反引号``，有同样效果

一般$var与${var}并没有啥不一样

但是用${}会比较精确的界定变量名称的范围，比如：

$A=B

$echo $AB

原意是想将$A的值替换出来，然后在后面加B，但在命令行上，结果却是只会替换变量为AB的值出来

正确的应该是${A}B



变量撤销

`unset var_name`



bash弱类型：

变量=值 （无需事先声明，可直接使用，默认值为字符型）

变量的值的类型：

数值型：

精确数值：整数

近似数值：浮点型,浮点型又分单精度和双精度浮点型

字符型：char，string

布尔型：true，false



增强型赋值：

a=$[$a+100],一般写成 let a+=100

表示变量a在自己的值上加上一个100，得到的结果在赋值给本身

+=, -=, *=, /=, %=, **=  都需要使用let命令进行描述



变量自加：var=$[$var+1] == let var+=1 == let var++ 

表示一个变量每次只加1或减1，赋值给自己本身，可写成此形式

export PATH=$PATH:

unset PATH //撤销变量



算术运算：bash会对数字执行隐式的类型转换

let VAR_NAME=Integer_Value(语句)

declare -i Var_Name=Integer_Value

VAR_NAME=$Integer_Value(表达式) 



操作符：+, -, *, /, %(取模), **(次方)



双目运算符：需要至少两个操作数



bash的算术运算的方式：

let Var_Name=EXPRESSION(表达式)

$[EXPRESSION]        

$((EXPRESSION))

命令：expr ARG1 OP ARG2

举例：

\#let sum=$num1+$num2

\#echo $sum

\#echo $[ $num1 + $num2 ]

\#echo $(($num1 + $num2))

expr $num1 + $num2

语句不能单独执行，需要let等命令来执行

表达式可以单独执行

![img](bash%E7%9F%A5%E8%AF%86%E7%82%B9.assets/17b81a10-23c3-402c-ba8c-9ca6e2677a0c.png)

给变量以默认值

varName=${varName-：value}

若varName不为空，则其值不变；否则，会使用value作为其值



逻辑运算



布尔运算：真1和假0



与运算：

1 && 1 = 1

1 && 0 = 0

0 && 1 = 0

0 && 0 = 0

只有两个都为真，结果才为真，否则为假。遵守短路法则



或运算：

1 || 0 = 1

1 || 1 = 1

0 || 1 = 1

0 || 0 = 0

只要有一个为真，即为真，类似电路并联



非运算：

非真为假，非假为真



异或运算：

判断两者是否不同，不同者为真；相同者为假

短路法则：

COMMAND1 && COMMAND2

COMMAND1为真，COMMAND2才会执行；COMMAND1为假时，COMMAND2不会在执行。

COMMAND1 ||  COMMAND2

COMMAND1为真，COMMAND2不执行；COMMAND1为假，COMMAND2才会执行



bash条件测试

命令执行成功与否，即为测试条件

命令执行成功与否为测试条件

  test 表达式

  [ 表达式 ]

  [[ 表达式 ]]



比较运算：

<, >, <=, >=, ==, !=,



整型比较：数值间的大小比较

-eq：等于

-gt：大于

-lt：小于

-ge：大于等于

-le：小于等于

-ne：不等于

```shell
a=10
b=20
[ $a -gt $b ]
echo $?
```



字符串比较：

字符串大小比较

1，字符串要加引号引用

2，作比较时使用双中括号[[ VAR ]]

==：等于

\>：大于  需用双中括号

<：小于  [[ ]]

!=：不等于

=~：左侧的字符串是否能被右侧的pattern所匹配，匹配为一部分，不是精确匹配，如：[[ "ab" =~ [ab][Ab][AB] ]]

   例如：写个脚本，可判断输入的用户是否有可登录shell

```shell
#!/bin/bash
#
read -p "Plz input a username: " userName
userInfo=`grep "^$userName\>" /etc/passwd

if [[ "$userName" =~ /bin/.*sh$ ]]; then
     echo "Can login."
else
     echo "Not can login."
fi`
```

-z “STRING”：判断指定的字串是否为空，空则为真，不空则假

-n “string”：判断指定的字串是否不空，空则为假，不空则真

```shell
#[ "$a" -gt "$b" ]
#echo $?
```



判断文件的存在性及属性等

-a或-e FILE：判断一个文件是否存在，存在即为真；

-f FILE ：存在且为普通文件，则为真；

-d FILE：存在且为目录，则为真；

-L或-h FILE ：存在且为软链接文件，则为真；

-b FILE：块设备

-S FILE：套接字文件

-s FILE ：存在且为非空文件

-N FILE：修改时间新于访问时间的文件，如被重定向过的文件

-r FILE：存在且可读

-w：可写

-x：可执行

例如:

FILE1 -nt  FILE2：文件1的修改时间mtime，新于文件2，则为真；

FILE1 -ot FILE2 ：旧于

FILE1 -ef FILE2 ：

例如：

```shell
if [ -x $a ];then
    echo $a
fi
```



权限测试

r，w，x，

组合条件测试：

与：[ condition1 -a condition2 ]

或：[ condition1 -o condition2 ]

非：[ -not condition ]

bash切片

1、基于字符串切片

Usage: ${var:offset: length}

```shell
# 定义一个变量，等会切这个变量
mypath="/etc/sysconfig/network-scripts/"
echo ${mypath:5} #偏移5个字符显示
"""
sysconfig/network-scripts/
"""

echo ${mypath:10} #偏移10个字符显示
"""
nfig/network-scripts/
"""

echo ${mypath:5:5} #偏移5个字符，取5个字符 
"""
sysco    
"""

# 取出字符串的最后几个字符：${var: -length}
# 注意：-length之前有空白字符
echo ${mypath: -10}
"""
k-scripts/
"""
```



2、基于模式取子串

Usage:
${var#*word}: 自左而右，查找var变量中存储的字符串中第一次出现的由word所指明的字符，删除此字符及其左侧的所有内容


${var##*word}: 自左而右，查找var变量中存储的字符串中最后一次出现的由word所指明的字符，删除此字符及其左侧的所有内容


${var%word*}: 自右而左，查找var变量中存储的字符串中第一次出现的由word所指明的字符，删除字符及其右侧的所有内容


${var%%word*}: 自右而左，查找var变量中存储的字符串中最后一次出现的由word所指明的字符，删除此字符及其右侧的所有内容

 

```shell
mypath="/etc/sysconfig/network-scripts"
echo ${mypath#*/}
"""
etc/sysconfig/network-scripts
"""

mypath="/etc/sysconfig/network-scripts"
echo ${mypath##*/}
"""
network-scripts
"""

mypath="/etc/sysconfig/network-scripts"
echo ${mypath%c*}
"""
/etc/sysconfig/network-s
"""

echo ${mypath%%c*}
"""
/et
"""
```



3、基于字串查找替换

Usage:
${var/pattern/replacement} :查找var变量存储的字符中第一次由pattern匹配到的内容，并替换为replacement


${var//pattern/replacement} :查找var变量存储的字符中所有能够由pattern匹配到的内容，并替换为replacement


${var/#pattern/replacement} :查找var变量存储的字符中最开始处能够由pattern匹配到的内容，并替换为replacement


${var/%pattern/replacement} : 查找var变量存储的字符中最后位置能够由pattern匹配到的内容，并替换为replacement


示例：

 

```shell
url="http://www.baidu.com:80"
echo ${url/www/WWW}
"""
http://WWW.baidu.com:80
"""

echo ${url/w/W}
"""
http://Www.baidu.com:80
"""

echo ${url//w/W}
"""
http://WWW.baidu.com:80
"""

userinfo="root:x:0:0:rootuser:/root:/bin/bash"
echo ${userinfo/#root/ROOT}
"""
ROOT:x:0:0:root user:/root:/bin/bash
"""

userinfo="root:x:0:0:rootuser:/root:/bin/root"
echo ${userinfo/%root/ROOT}
"""
root:x:0:0:root user:/root:/bin/ROOT
"""
```



4、基于字串查找删除
Usage:
${var/pattern}：删除变量第一次pattern匹配到的内容
${var//pattern}：删除变量所有pattern匹配到的内容
${var/#pattern}：删除变量头部匹配pattern的内容
${var/%pattern}：删除变量尾部能够匹配pattern的内容
示例：

 

```shell
userinfo="root:x:0:0:rootuser:/root:/bin/root"
echo ${userinfo/root}
"""
:x:0:0:root user:/root:/bin/root
"""

echo ${userinfo//root}
"""
:x:0:0: user:/:/bin/
"""

echo ${userinfo/#root}
"""
:x:0:0:root user:/root:/bin/root
"""

echo ${userinfo/%root}
"""
root:x:0:0:root user:/root:/bin/
"""

```



5、基于字符串大小写转换
Usage:
${var^^}：把var变量中的所有小写字母，统统替换为大写；
${var,,}：把var变量中的所有大写字母，统统替换为小写；
示例

```shell
echo $userinfo
"""
root:x:0:0:root user:/root:/bin/root
"""

myinfo=${userinfo^^}
echo $myinfo
"""
ROOT:X:0:0:ROOT USER:/ROOT:/BIN/ROOT
"""

echo ${myinfo,,}
"""
root:x:0:0:root user:/root:/bin/root
"""
```



6、空变量判断赋值
Usage:
${var:-word}：如果变量var为空或未声明，则返回word所表示的字符串；否则，则返回var变量的值, 临时赋值

```shell
echo $name # 这行的值为空
echo ${name:-tom}
tom

name=hello 
echo ${name:-tom}            hello
```



${var:=word}：如果变量var为空或未声明，则返回word所表示的字符串，并且把word赋值为var变量；否则，则返回var变量的值，直接等值

```shell
echo "User's name is${name:?wrong}"
-bash: name: wrong

name=tom
echo "User's name is${name:?wrong}"
User's name is tom
```



${var:+word}：如果变量var为空或未声明则忽略；否则，则返回word；

```shell
unset name
echo "User's name is ${name:+wrong}"
"""
User's name is
"""

name=tom
echo "User's name is${name:+wrong}"
"""
User's name is wrong 
"""
```



# 正则表达式字符匹配

.：匹配任意单个字符

.*：匹配任意字符，0个到无穷多个

[]：匹配指定范围内的任意字符：

[0-9],[[:digit:]]

[a-z],[[:lower:]]

[A-Z],[[:upper:]]

空白  [[:space:]]

所有标点符号 [[:punct:]]

所有大小写字母 [[:alpha:]]

所有大小写字母和数字符号 [[:alnum:]]

[^] 取反

# bash环境中特殊符号

\#：批注符号，视为说明

\：转义符号，将特殊字符或通配符还原成一般字符

|：管道。左边命令结果，输送给右边命令执行

;：连续命令执行分隔符，前面命令执行完，执行后面一个，然后同时输出屏幕

&&：如果前面的命令执行正确，则开始执行后面的命令，如果前面命令出错，后面命令也不会执行

||：如果前面的命令执行成功，则后面的命令不执行，反之，如果前面的命令执行失败，则开始执行后面的命令

$：使用变量前导符

&：作业控制，将命令送到后台执行

!：逻辑运算中‘非’的意思

‘’：单引号，不具有变量置换功能，原样输出

“”：双引号，具有变量置换的功能

··：反单引号，优先执行的命令，等同于$()

# RAID

:0：条带技术  性能+，读写+
  冗余 -
1：镜像 （每块硬盘都存储同一种数据）  性能+，读+，写-
  冗余+
2：3：4：校验码技术5：RAID 5 （至少3块）  性能+，读写+
  冗余+1+0： （至少4块）  性能+，读写+
  冗余+
0+1：（至少4块）  性能+，读写+  冗余+5+0： （至少6块）  性能+，读写+
  冗余+JBOD技术：适用于存储单个大文件，多个硬盘累加当成一块硬盘用  性能0，读写0
  冗余0（至少2块）(ps:比如一块硬盘存储一个文件，这个文件的大小累计增长的，当文件大小达到硬盘容量后，这时可以在添加一块硬盘，然后使用JBOD技术，把新加的硬盘与原硬盘合并成一整块硬盘。)

![img](bash%E7%9F%A5%E8%AF%86%E7%82%B9.assets/400a682f-eeeb-4784-acd7-ab06ca109287.png)![img](file:///D:/My Knowledge/temp/6ea688a4-c878-4750-b831-eda777da87d0/128/index_files/9b88cf85-3efc-465c-a7f3-390a86c4e9b9.png)
![img](file:///D:/My Knowledge/temp/6ea688a4-c878-4750-b831-eda777da87d0/128/index_files/4dd05eca-0141-4c9f-b8de-d67d044d3045.jpg)



# man手册的段落

NAME：命令的名称
        DESCRIPTION：命令功能的详细描述
        OPTIONS：所有选项
        SYNOPSIS：使用格式
        EXAMPLES：使用示例
        FILES：与当前命令相关的配置文件
        SEE ALSO：可参考的其他手册
帮助中的格式字串        []：可省略
        <>:不可省略
        | : 二选一或多选一
        ... : 同类内容可以出现多个





# Linux将命令行执行的命令记录到日志文件中便于审计使用

配置方法：

1,编辑/etc/bashrc文件

vim /etc/bashrc

\# 在此文件的最后一行加入如下内容

export PROMPT_COMMAND='{ msg=$(history 1 | { read x y; echo $y; });logger "[hostname- $(hostname)]": "[euid=$(whoami)]":$(who am i):[`pwd`]:"$msg"; }'

\# 保存退出

2,重新加载下bashrc

source /etc/bashrc

3,查看配置结果

\# 在执行如下之令之前,可以随意执行几个命令,以便显示效果

tail -f /var/log/messages



# end