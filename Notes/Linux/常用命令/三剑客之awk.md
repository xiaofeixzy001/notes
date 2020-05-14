[TOC]

# AWK

pattern scanning and processing language

模式扫描和处理语言,文本报告生成器

简单来说awk就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理.

awk有3个不同版本:

awk,nawk和gawk

未作特别说明,linux系统中一般指gawk,gawk是 AWK 的 GNU 版本.

awk其名称得自于它的创始人 Alfred Aho 、Peter Weinberger 和 Brian Kernighan 姓氏的首个字母.实际上 AWK 的确拥有自己的语言:AWK 程序设计语言,三位创建者已将它正式定义为"样式扫描和处理语言".它允许创建简短的程序,这些程序读取输入文件、为数据排序、处理数据、对输入执行计算以及生成报表，还有无数其他的功能.

使用格式

awk [options] -f progfile [--] file ...

awk [options] [--] 'program' file ...

awk [options] 'BEGIN{ action;… } pattern{ action;… } END{ action;… }' file ...

program：相当于编程语言，也就是处理后面文件的一系列操作语句

progfile：带有program或BEGIN等操作语句内容的文件

BEGIN：读取输入流前进行操作的标志 ,读取前操作

END：输入流读取完后进行操作的标志 ,读取后操作

pattern：模式,对输入流进行操作

action：处理动作语言,由多种语句组成,语句间用分号分割

## options

-F : 指明输入分隔符,表示读取文件内容时以什么符号为分隔符进行切片

-v var=value: 自定义变量

-f progfile, --file=progfile: 从文件中来读取awk的program

## program

PATTERN(模式){ACTION STATEMENTS(动作语句);ACTION STATEMENTS;...}

相当于编程语言，也就是处理后面文件的一系列操作语句

## action

处理动作语言,由多种语句组成,语句间用分号分割

表达式Expressions

控制语句Control Statements

组合语句Compound statements

输入语句Input Statements

输出语句Output Statements

## BEGIN&END工作原理

格式:

awk [options] 'BEGIN{ action;… } pattern{ action;… } END{ action;… }' file ...

第一步:执行[option]相关内容，也就是-f，-F，-v选项内容;

第二步:执行BEGIN{action;… } 语句块中的语句;

BEGIN 语句块在awk开始从输入流中读取行之前被执行,这是一个可选的语句块,比如变量初始化、打印输出表格的表头等语句通常可以写在BEGIN语句块中;

第三步:从文件或标准输入(stdin) 读取每一行，然后执行pattern{action;… }语句块,它逐行扫描文件,从第一行到最后一行重复这个过程,直到文件全部被读取完毕.

pattern语句块中的通用命令是最重要的部分,也是可选的.如果没有提供pattern 语句块,则默认执行{print},即打印每一个读取到的行,awk读取的每一行都会执行该语句块;

第四步:当读至输入流末尾时，也就是所有行都被读取完执行完后，再执行END{action;…}语句块.

END语句块在awk从输入流中读取完所有的行之后即被执行,比如打印所有行的分析结果这类信息汇总都是在END语句块中完成,它也是一个可选语句块.

awk语言的最基本功能是在文件或者字符串中基于指定规则浏览和抽取信息,awk抽取信息后,才能进行其他文本操作.

完整的awk脚本通常用来格式化文本文件中的信息.

通常,awk是以文件的一行为处理单位的。awk每接收文件的一行,然后执行相应的命令,来处理文本.

## 打印输出

print函数的参数可以是变量、数值或者字符串。字符串必须用双引号引用，参数用逗号分隔。如果没有逗号，参数就串联在一起而无法区分。这里，逗号的作用与输出文件的分隔符的作用是一样的，只是后者是空格而已。

printf函数，其用法和c语言中printf基本相似,可以格式化字符串,输出复杂时，printf更加好用，代码更易懂.

### print

使用格式:

print item1,item2,...

示例:

 

```
awk -F: '{print "hello!",$1}' /etc/passwd
"""
hello! root
hello! bin
"""

awk -F: '{print}' /etc/passwd
awk -F: '{print "hello"}' /etc/passwd
awk -F: '{print $0}'
awk -F: '{print "hello",$1}' /etc/passwd
awk -F: '{print "hello",$1,"\t",$7}' /etc/passwd
tail -3 /etc/fatab | awk '{print $2,$4}'

# 统计/etc/passwd用户数量
awk '{count++;print $0;} END{print "user count is ", count}' /etc/passwd
"""
root:x:0:0:root:/root:/bin/bash
......
user count is  40
"""
# 优化: awk 'BEGIN {count=0;print "[start]user count is ", count} {count=count+1;print $0;} END{print "[end]user count is ", count}' /etc/passwd
```



注意:

1,逗号为分隔符时,最终显示的是空格;

2,分隔符分隔的字段(域)标记称为域标识,默认空格为分隔符,用$0,$1,$2,...,$n表示,其中$0为所有域,$1就是表示第一个字段(域),以此类推,$nf为最后 ;

3,输出的各item可以字符串(字串必须双引号),也可以是数值,当前记录的字段,变量或awk的表达式等 ;

4,如果省略了item,相当于print $0 ;

### printf

格式化输出

printf "format",item1,item2,...

注意:

必须指定format,即必须指出后面每个itemsN的输出格式;

不会自动换行,需要自行给出换行控制符"\n";

format中需要分别为后面的每个item指定一个格式化符号

#### format格式符

%c: 显示字符的ASCII码

%d,%i：显示为十进制整数

%e,%E：科学计算法显示数值

%f：显示为浮点数

%g,%G：以科学计数法或浮点数格式显示数值

%s：显示字符串

%u：显示无符号整数

%%：显示%自身

#### format修饰符

格式: #[.#],如%3.1f

第一个#表示显示宽度,第二个#表示小数点后的精度

-: 左对齐

+: 显示数值的符号

示例:

 

```
awk -F: '{printf "Username:%s, UID:%d\n",$1,$3}' /etc/passwd
"""
Username:root, UID:0
...
"""

# 将显示的用户名宽带统一设置为15,默认是右对齐
awk -F: '{printf "Username:%15s, UID:%d\n",$1,$3}' /etc/passwd
"""
Username:      root, UID:0
Username:       bin, UID:1
...
"""
```



## 输出重定向

print items > output-file

print items >> output-file

print items | command

特殊文件描述符：

/dev/stdin：标准输入

/dev/stdout：标准输出

/dev/stderr：错误输出

### 变量

#### 内置变量

FS(Filed Seperator)：输入时的字段分隔符

OFS(Output Filed Seperator)：输出时的分隔符,默认为空白字符

RS(Record Seperator)：输入时的行分隔符，默认回车

ORS(Output Record Seperator)：输出时的行分隔符

NF(Numbers of Field)：显示每行的字段总数

NR(Numbers of Record)：显示行数,默认计数所有的文件

FNR(Field Numbers of Record)：行号,各文件分别计数

ARGV: 数组,保存的是命令行所给的各个参数,ARGV[0]指命令本身,ARGV[1]指第一个参数,以此类推

ARGC: 统计命令行参数的个数

FILENAME：awk正在处理的当前文件的名称

示例:

 

```
awk 'BEGIN{FS=":"}{print "hello!",$1}' /etc/passwd
"""
hello! root
...
hello! ntp
"""

# 输入时以冒号分割,输出时以@@分割
awk 'BEGIN{FS=":";OFS="@@"}{print "hello!",$1}' /etc/passwd
"""
# 注意：语句和语句间用分号;分隔
hello!@@root
...
hello!@@ntp
"""

cat /etc/passwd
"""
root:x:0:0:root:/root:/bin/bash
...
"""

# 输入时以冒号为换行符
awk 'BEGIN{RS=":"}{print "hello:",$1}' /etc/passwd
"""
hello: root
hello: x
hello: 0
hello: 0
hello: root
hello: /root
hello: /bin/bash
...
"""

# 输入时以冒号为换行符,输出时以###为换行符
awk -v RS=':' -v ORS='###' '{print }' /etc/passwd
"""
root###x###0###0###root###/root###/bin/bash
"""

# 显示每行字段总数,$NF表示最后一个字段
awk -F: '{print NF}' /etc/passwd
awk -F: '{print $NF}' /etc/passwd

# 显示行号
awk '{print NR}' /etc/fstab
"""
1
2
..
15
"""

# 跟多个文件,行号累加计数
awk '{print NR}' /etc/fstab /etc/passwd
"""
1
2
...
15
16
...
35
"""

# 跟多个文件,行号分开计数
awk '{print FNR}' /etc/passwd /etc/fstab
"""
1
2
...
15
1
...
20
"""

awk 'BEGIN{print ARGV[0],ARGV[1],ARGC}' /etc/passwd /etc/fstab /etc/group
"""
awk /etc/passwd 4
"""

# 统计/etc/passwd:文件名，每行的行号，每行的列数，对应的完整行内容
awk  -F ':'  '{print "filename:" FILENAME ",linenumber:" NR ",columns:" NF ",linecontent:"$0}' /etc/passwd

awk 'BEGIN{print ARGC}' /etc/fstab /etc/passwd
awk 'BEGIN{print ARGV[0]}' /etc/fstab /etc/passwd
awk 'BEGIN{print ARGV[1]}' /etc/fstab /etc/passwd
awk 'BEGIN{print ARGV[2]}' /etc/fstab /etc/passwd
```



#### 自定义变量

定义方法

1,awk -v var1=value1 -v var2=value2 ...

2,在program中直接定义

注:变量名区分字符大小写

例如:

 

```
awk -v a='hello,awk' 'BEGIN{print a}'
awk 'BEGIN{a="hello,awk.";print a}'
"""
hello,awk.
"""

awk -v a='hello,awk' '{print a}' /etc/passwd
awk 'BEGIN{a="hello,awk."}{print a}' /etc/passwd
"""
hello,awk.
...
"""

awk -F: '{sex="male";age=20;print $1,sex,age}' /etc/passwd
"""
root male 20
bin male 20
...
"""

echo "{print script,\$1,\$2}" > awkscript
awk -F: -f awkscript script="hello,awk" /etc/passwd
"""
hello,awk root x
...
"""
```



注意: 在awk中,变量的引用无需加$符号,区别于bash

## 操作符

算术操作符：+, -, *, / ,^, %

赋值操作符：=, +=, -=, *=, /=, %=, ^=，++, --

比较操作符：==, !=, >, >=, <, <=

字符串操作符: 没有符号的操作符,如字符串拼接

模式匹配符：

~ 模式匹配,左边的字符串能够被右边的模式所匹配为真

!~ 不匹配为真

逻辑操作符:

与: &&

或: ||

非: !

条件表达式(三目表达式):

selector?if-true-expression:if-false-expression

示例:

 

```
awk '$0~"^root"' /etc/passwd
awk -F: '$3 == 0' /etc/passwd
awk -F: '$3 >= 0 && $3 <= 10 {print $1,$3}' /etc/passwd
awk -F: '$3 == 0 || $3 >= 10 {print $1,$3}' /etc/passwd
awk -F: '!($3==0) {print $1,$3}' /etc/passwd
awk -F: '!($3>=500) {print $1,$3}' /etc/passwd

# 判断用户uid是否大于500,大于500的则显示为普通用户,小于等于500的则显示为系统用户或管理员
awk -F: '{$3>=500?utype="common user.":utype="admin or system user.";printf "%15s:%-s\n",$1,utype}' /etc/passwd
```



函数调用：

function_name（argu1，argu2，...）

## 模式pattern

1,未指定,表示空模式,全文处理,匹配每一行;

2,/regular expression/: 仅处理被匹配到的行,支持RE正则,需要//括起来;

3,比较表达式(Expression):关系表达式,其结果为非0或非空字符串时为真, 被匹配;

4,范围表达式(Ranges):行范围,此前称为地址界定,startline,endline, /pat1/,/pat2/

5,BEGIN/END:仅在开始处理文件中的文本之前|之后执行一次,

示例:

 

```
# 找到匹配'root'的行并打印
awk -F: '/root/{print}' /etc/passwd

# 找到开头匹配root的行并打印
awk -F: '/^root/{print}' /etc/passwd

# 找到uid大于等于100的行,然后打印第1,3字段
awk -F: '$3>=100{print $1,$3}' /etc/passwd

# 打印显示第一次匹配h开头的行,到第一次匹配u开头的行
awk -F: '/^h/,/^u/{print $1}' /etc/passwd

# 显示行号在2-5间的行并显示这些行的行号和第一个字段
awk -F: '(NR>=2 && NR<=5){print NR,$1}' /etc/passwd

# 找到/etc/passwd中bash为/sbin/nologin的行,打印其第name和bash
awk -F: '$NF=="/sbin/nologin"{print $1,$NF}' /etc/passwd

awk -F: 'BEGIN{print "      USER      USERID"}{printf "|%8s| %10d|\n",$1,$3}END{print "END FILE"}' /etc/passwd
"""
      USER      USERID
|    root|          0|
|     bin|          1|
...
"""
```



## 控制语句

### if

语法:

如果满足条件,就行statement

if(condition){statement;...}

如果满足条件1就执行statement1,否则执行statement2

if(condition){statement1;...}else {statement2}

满足条件1,执行statement1,满足条件2,执行statement2,否则执行statement3

if(condition1){statement1}else if(condition2){statement2}else{statement3}

应用场景: 对awk取得的整行或某个字段做条件判断

 

```
# 如果uid大于等于1000,就显示这个用户
awk -F: '{if($3>=10) {print $1,$3}}' /etc/passwd

# 如果uid大于等于500,就显示这个用户是普通用户,否则就显示这个用户是系统用户
awk -F: '{if($3>=500) {printf "common user: %s\n",$1} else {printf "system user: %s\n",$1}}' /etc/passwd

# 打印shell为/bin/bash的用户
awk -F: '{if($NF=="/bin/bash") {print $1}}' /etc/passwd

# 显示字段大于10的行
awk '{if(NF>10) {print $0}}' /etc/fstab

# 打印使用率超过80%的分区
df -h | awk -F% '/^\/dev/{print $1}'|awk '$NF>=80{print $1,$5}'

# 判断test值,如果大于90,显示very good
awk 'BEGIN{test=100;if(test>90){print "very good"}else if(test>60){print "good"}else{print "no pass"}}'
```



### while

语法:

条件为'真'时,进入循环,否则退出循环

while(condition){statement;...}

无论'真假',先至少执行一次循环,完了当条件为'假'时,继续循环,当条件为'真'时,退出循环

do {statement;...}while(conding)

应用场景: 对一行内的多个字段逐一类似处理时使用

 

```
# 计算1-100的和
awk 'BEGIN{total=0;i=0;do{total+=i;i++}while(i<=100);print total}'

# 打印以任意空白linux16开头的行,以空格分割,统计每个字段的字符个数
awk '/^[[:space:]]*linux16/{i=1;while(i<=NF){print $i,length($i); i++}}' /etc/grub2.cfg

# 遍历以任意空白linux16开头的行,以空格分割,统计每个字段的字符个数,打印显示字符个数大于等于7的行
awk '/^[[:space:]]*linux16/{i=1;while(i<=NF) {if(length($i)>=7) {print $i,length($i)}; i++}}' /etc/grub2.cfg

# 取出/etc/inittab文件中，每一行中，字符串的长度大于等于6的字串
awk '{i=1; while (i<=NF){if (length($i)>=6) {print $i}; i++ }}' /etc/inittab


```



注:

length()：内置函数，取字符串的长度

### for循环

语法:

for(var assignment; condition; iteration process){for-body}

for (变量赋值;条件判断;迭代语句){循环体动作}

for循环还可用来遍历数组元素,遍历的是数组中的元素的下标

语法:

for (var in array) {for body}

变量var遍历数组array,每次遍历执行for-body

 

```
# 打印以任意空白linux16开头的行,以空格分割,统计每个字段的字符个数
awk '/^[[:space:]]*linux16/{for(i=1;i<=NF;i++){print $i,length($i)}}' /etc/grub2.cfg

# 显示文件/etc/inittab中奇数行
awk '{for (i=1;i<=NF;i+=2){printf "%s ",$i };print ""}' /etc/inittab

# 取出/etc/inittab文件中，每一行中，字符串的长度大于等于6的字串
awk '{for (i=1;i<=NF;i++){if (length($i)>=6) print $i}}' /etc/inittab
```



### switch语句

相当于bash中case语句

指定一个表达式expression，如果此表达式被VALUE所匹配或者被正则RGEEXP所匹配，那么就执行statement1,...,如果都不满足,则执行default对应的statement

格式：switch (expression) {case VALUE1 or /RGEEXP1/: statement1; case VALUE2 or /REGEXP2/: statement2; ...; default: stemertN}

## 循环控制

break, continue, next, exit

break和continue,用于条件判断循环语句, next用于awk自身循环的语句

break[n]: 当第n次循环时,结束整个循环,n=0表示本次循环

continue[n]: 满足条件后,直接进行第n次循环,本次循环不在进行,n=0也就是提前结束本次循环而直接进入下一轮轮换

next：如果满足条件则,提前结束对本行的处理,然后进入下一行的处理

示例

 

```
# 取出/etc/passwd文件中id号为奇数的用户名
awk -F: '{if ($3%2==0) next; print $1,$3}' /etc/passwd

# 取出/etc/passwd中奇数行的用户名
awk -F: '{if (NR%2==0) next; print NR,$1}' /etc/passwd

# 1到100内的偶数和
awk 'BEGIN{sum=0; for(i=1;i<=100;i++){if(i%2==0)continue;sum+=i} print sum}'

# 打印出uid为偶数的用户
awk -F: '{if($3%2!=0) next; print $1,$3}' /etc/passwd
```



## 数组

awk中数组的下标可以是数字和字母，数组的下标通常被称为关键字(key)，可以用于计算总和、统计单词以及跟踪模板被匹配的次数等等。

数组中的元素在内存是连续排列的，有时我们我处理里面的数据并不是从头取到尾，有时从中间取，所以这里就需要一个下标，标记是在哪个位置，这个下标就就是索引，比如中药铺里面的药柜，要写明药名是不，这个名就是索引．而数组下标是从0开始的

### 关联数组

array[index-expression]

其中index-expression可以使用任意字串,如果某数组元素事先不存在,那么在引用时,awk会自动创建此元素并将其初始化为空串;

因此,要判断某数组是否存在某元素,必须使用“index in array”这种格式。

要遍历数组中的每一个元素，需要使用如下特殊结构：

for (var in array) {for body}

其var会遍历array的索引（即元素的下标）

示例

 

```
# 数组的定义与显示
awk 'BEGIN{weekdays["mon"]="Monday";weekdays["tue"]="Tuesday"; print weekdays["mon"]}'
awk 'BEGIN{weekdays["mon"]="Monday";weekdays["tue"]="Tuesday"; print weekdays["tue"]}'

# 遍历数组
awk 'BEGIN{weekdays["mod"]="Monday";weekdays["tue"]="Tuesday"; for(i in weekdays) {print weekdays[i]}}'

# 查看命令netstat -tan的结果，统计state每一种连接状态（LISTEN、ESTABLISHED等）的个数,$NF是最后一个元素
netstat -tan | awk '/^tcp\>/ {state[$NF]=state[$NF]+1} END{for(i in state){print i,state[i]}}'
netstat -tan | awk '/^tcp/ {state[$NF]++} END {for (i in state){print i,state[i]}}'

# 统计/var/log/httpd/access_log中，每个IP请求的资源的次数
awk '{ip[$1]++} END{for(i in ip) {print i,ip[i]}}' /var/log/httpd/access_log

# 统计/etc/fstab文件中每个文件系统类型出现的次数
awk '/^UUID/{fs[$3]++} EDN{for(i in fs) {print i,fs[i]}}' /etc/fstab

# 统计指定文件中每个单词出现的次数
awk '{for(i=1;i<=NF;i++){count[$i]++}}END{for(i in count) {print i,count[i]}}' /etc/fstab
```



删除数组元素：

delete array[index]

delete array

## 函数

函数分内置函数和自定义函数

### 内置函数

rand(): 返回0和1之间的一个随机数

srand(): 生成随机数种子

int(): 取整数

length([s]): 返回指定字符串的长度

sub(r,s,[t]): 以r表示的模式来查找t所表示的字符中的匹配的内容,并将第一次出现替换为s所表示的内容

gsub(r,s,[t]): 对t字符串进行搜索,r表示的模式匹配的内容,并全部替换为s所表示的内容

split(s,array,[r]): 元素切片,以r为分隔符,切割字符串s,并将切割后的结果保存至array所表示的数组中,该数组下标从1开始

substr(s,i,[n]): 从s所表示的字符串中取子串,取法:从i表示的位置开始,取n个字符

systime(): 取当前系统时间,结果形式为时间戳

system(): 调用shell中的命令,空格是awk中的字符串连接符,

如果system 中需要使用awk中的变量可以使用空格分隔，或者说除了awk的变量外其他一律用"" 引用 起来

示例:

 

```
awk 'BEGIN{print rand()}'

awk 'BEGIN{srand(); for (i=1;i<=10;i++)print int(rand()*100) }'

echo "2008:08:08 08:08:08" | awk 'sub(/:/,“-",$1)'

echo "2008:08:08 08:08:08" | awk ‘gsub(/:/,“-",$0)'

# 统计每个ip发起了多少条连接
netstat -tan | awk '/^tcp\>/ {split($5,ip,":"); count[ip[1]]++} END{for(i in count) {print i,count[i]}}'

awk BEGIN'{system("hostname") }'

awk 'BEGIN{score=100; system("echo your score is " score) }'
```



### 自定义函数

格式

function fname ( arg1,arg2 , ... ) {

  statements

  return expr

}

fname为函数名

arg1...为函数的参数

statements是动作语言

return expr为由statements的结果从而决定最终函数所显示的内容

示例:

 

```
vim fun.awk
"""
function max(v1,v2) {
    v1>v2? var=v1:var=v2
    return var
}
BEGIN{a=3;b=2; print max(a,b)}
"""

awk –f fun.awk
```



## awk的脚本

awk的脚本就是将awk程序写成脚本形式，来直接调用或直接执行

例如上面写自定义函数的样子也算是脚本

 

```
# 格式1：
BEGIN{} pattern{} END{}

# 格式2：
\#!/bin/awk -f
\#add 'x' right 
BEGIN{} pattern{} END{}

```



格式1假设为f1.awk文件，格式2假设为f2.awk文件，那么用法是： 

 

```
# 格式1用法:就是把处理阶段放到一个文件而已,真正展开后,也就是普通的awk语句
awk [-v var=value] f1.awk [file]

# 格式2用法:[-v var=value]是在BEGIN之前设置的变量值,[var1=value1]是在BEGIN过程之后进行的,也就是说直到首行输入完成后,这个变量才可用
f2.awk [-v var=value] [var1=value1] [file]
```



示例

 

```
# 示例1
cat f1.awk
"""
{if($3>=10)print $1,$3}
"""

awk -F: -f f1.awk /etc/passwd

# 示例2
cat f2.awk
"""
#!/bin/awk –f
# this is a awk script
{if($3>=1000)print $1,$3}
"""

chmod +x f2.awk
./f2.awk –F: /etc/passwd

# 示例3
cat f3.awk
"""
#!/bin/awk –f
{if($3 >=min && $3<=max)print $1,$3}
"""

chmod +x f3.awk
./f3.awk -F: min=100 max=200 /etc/passwd
```



# 案例

知识点：

1）数组

数组是用来存储一系列值的变量，可通过索引来访问数组的值。

Awk中数组称为关联数组，因为它的下标（索引）可以是数字也可以是字符串。

下标通常称为键，数组元素的键和值存储在Awk程序内部的一个表中，该表采用散列算法，因此数组元素是随机排序。

数组格式：array[index]=value

## Nginx日志分析

  日志格式：'$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$http_x_forwarded_for"'

  日志记录：27.189.231.39 - - [09/Apr/2016:17:21:23 +0800] "GET /Public/index/images/icon_pre.png HTTP/1.1" 200 44668 "http://www.test.com/Public/index/css/global.css" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.157 Safari/537.36" "-"

  1）统计日志中访问最多的10个IP

​    思路：对第一列进行去重，并输出出现的次数

​    方法1：$ awk '{a[$1]++}END{for(i in a)print a[i],i|"sort -k1 -nr|head -n10"}' access.log

​    方法2：$ awk '{print $1}' access.log |sort |uniq -c |sort -k1 -nr |head -n10

​    说明：a[$1]++ 创建数组a，以第一列作为下标，使用运算符++作为数组元素，元素初始值为0。处理一个IP时，下标是IP，元素加1，处理第二个IP时，下标是IP，元素加1，如果这个IP已经存在，则元素再加1，也就是这个IP出现了两次，元素结果是2，以此类推。因此可以实现去重，统计出现次数。

  2）统计日志中访问大于100次的IP

​    方法1：$ awk '{a[$1]++}END{for(i in a){if(a[i]>100)print i,a[i]}}' access.log

​    方法2：$ awk '{a[$1]++;if(a[$1]>100){b[$1]++}}END{for(i in b){print i,a[i]}}' access.log

​    说明：方法1是将结果保存a数组后，输出时判断符合要求的IP。方法2是将结果保存a数组时，并判断符合要求的IP放到b数组，最后打印b数组的IP。

  3）统计2016年4月9日一天内访问最多的10个IP

​    思路：先过滤出这个时间段的日志，然后去重，统计出现次数

​    方法1：$ awk '$4>="[9/Apr/2016:00:00:01" && $4<="[9/Apr/2016:23:59:59" {a[$1]++}END{for(i in a)print a[i],i|"sort -k1 -nr|head -n10"}' access.log

​    方法2：$ sed -n '/\[9\/Apr\/2016:00:00:01/,/\[9\/Apr\/2016:23:59:59/p' access.log |sort |uniq -c |sort -k1 -nr |head -n10  #前提开始时间与结束时间日志中必须存在

  4）统计当前时间前一分钟的访问数

​    思路：先获取当前时间前一分钟对应日志格式的时间，再匹配统计

​    $ date=$(date -d '-1 minute' +%d/%b/%Y:%H:%M);awk -vdate=$date '$0~date{c++}END{print c}' access.log

​    $ date=$(date -d '-1 minute' +%d/%b/%Y:%H:%M);awk -vdate=$date '$4>="["date":00" && $4<="["date":59"{c++}END{print c}' access.log

​    $ grep -c $(date -d '-1 minute' +%d/%b/%Y:%H:%M) access.log

​    说明：date +%d/%b/%Y:%H:%M --> 09/Apr/2016:01:55

  5）统计访问最多的前10个页面（$request）

​    $ awk '{a[$7]++}END{for(i in a)print a[i],i|"sort -k1 -nr|head -n10"}' access.log

  6）统计每个URL访问内容的总大小（$body_bytes_sent）

​    $ awk '{a[$7]++;size[$7]+=$10}END{for(i in a)print a[i],size[i],i}' access.log

  7）统计每个IP访问状态码数量（$status）

​    $ awk '{a[$1" "$9]++}END{for(i in a)print i,a[i]}' access.log

  8）统计访问状态码为404的IP及出现次数

​    $ awk '{if($9~/404/)a[$1" "$9]++}END{for(i in a)print i,a[i]}' access.log

## 两个文件对比

  文件内容如下：

  $ cat a

  1

  2

  3

  4

  5

  6

  $ cat b

  3

  4

  5

  6

  7

  8

  1）找出相同记录

​    方法1：$ awk 'FNR==NR{a[$0];next}($0 in a)' a b

​    3

​    4

​    5

​    6

​    解释前，先看下FNR和NR区别：

​    $ awk '{print NR,$0}' a b

​    1 1

​    2 2

​    3 3

​    4 4

​    5 5

​    6 6

​    7 3

​    8 4

​    9 5

​    10 6

​    11 7

​    12 8

​    $ awk '{print FNR,$0}' a b

​    1 1

​    2 2

​    3 3

​    4 4

​    5 5

​    6 6

​    1 3

​    2 4

​    3 5

​    4 6

​    5 7

​    6 8

​    可以看出NR是处理一行记录，编号就会加1，同时也可以看到awk将两个文件当成一个合并后的文件处理。

​    而FNR则是处理一行记录，编号也会加1，但是，处理到第二个文件时，编号重新计数。

​    说明：FNR和NR是内置变量。FNR==NR常用于对两个文件处理，这个例子可以理解为awk将两个文件当成一个文件处理。

​    处理a文件时，FNR是等于NR的，条件为真，执行a[$0],next表达式，意思是将每条记录存放到a数组作为下标（无元素），next是跳出，类似于continue，不执行后面表达式。

​    执行过程以此类推，直到处理b件时，FNR不等于NR（FNR重新计数是1，NR继续加1是7），条件为假，不执行后面a[$0],next表达式，直接执行($0 in a)表达式，这句意思是处理b文件第一条继续判断是否在a数组中，如果在则打印这条记录，以此类推。

​    这样可能更好理解些：

​    $ awk 'FNR==NR{a[$0]}NR>FNR{if($0 in a)print $0}' a b

​    方法2：

​    $ awk 'FNR==NR{a[$0]=1;next}(a[$0])' a b  #小括号可以不加

​    $ awk 'FNR==NR{a[$0]=1;next}(a[$0]==1)' a b

​    $ awk 'FNR==NR{a[$0]=1;next}{if(a[$0]==1)print}' a b

​    $ awk 'FNR==NR{a[$0]=1}FNR!=NR&&a[$0]==1' a b

​    说明：先要知道后面的a[$0]不是一个数组，而是通过下标（b文件每条记录）来访问a数组元素。如果a[b的一行记录]获取的a数组元素是1，则为真，也就是等于1，打印这条记录，否则获取不到元素，则为假。

​    方法3：

​    $ awk 'ARGIND==1{a[$0]=1}ARGIND==2&&a[$0]==1' a b

​    $ awk 'FILENAME=="a"{a[$0]=1}FILENAME=="b"&&a[$0]==1' a b

​    说明：ARGIND内置变量，处理文件标识符，第一个文件为1，第二个文件为2。FILENAME也是内置变量，表示输入文件的名字

​    方法4：$ sort a b |uniq -d

​    方法5：$ grep -f a b

  2）找不同记录（同上，取反）

​    $ awk 'FNR==NR{a[$0];next}!($0 in a)' a b

​    $ awk 'FNR==NR{a[$0]=1;next}!a[$0]' a b

​    $ awk 'ARGIND==1{a[$0]=1}ARGIND==2&&a[$0]!=1' a b

​    $ awk 'FILENAME=="a"{a[$0]=1}FILENAME=="b"&&a[$0]!=1' a b

​    7

​    8

​    方法2：$ sort a b |uniq -d

​    方法3：$ grep -vf a b

 3、合并两个文件

  1）将d文件性别合并到c文件

​    $ cat c

​    zhangsan 100

​    lisi 200

​    wangwu 300

​    $ cat d

​    zhangsan man

​    lisi woman

​    方法1：$ awk  'FNR==NR{a[$1]=$0;next}{print a[$1],$2}' c d

​    zhangsan 100  man

​    lisi 200 woman

​    wangwu 300 man

​    方法2：$ awk  'FNR==NR{a[$1]=$0}NR>FNR{print a[$1],$2}' c d

​    说明：NR==FNR匹配第一个文件，NR>FNR匹配第二个文件，将$1为数组下标

​    方法3：$ awk 'ARGIND==1{a[$1]=$0}ARGIND==2{print a[$1],$2}' c d

  2）将a.txt文件中服务名称合并到一个IP中

  $ cat a.txt

  192.168.2.100 : httpd

  192.168.2.100 : tomcat

  192.168.2.101 : httpd

  192.168.2.101 : postfix

  192.168.2.102 : mysqld

  192.168.2.102 : httpd

  $ awk -F: -vOFS=":" '{a[$1]=a[$1] $2}END{for(i in a)print i,a[i]}' a.txt

  $ awk -F: -vOFS=":" '{a[$1]=$2 a[$1]}END{for(i in a)print i,a[i]}' a.txt

  192.168.2.100 : httpd  tomcat

  192.168.2.101 : httpd  postfix

  192.168.2.102 : mysqld  httpd

  说明：a[$1]=$2 第一列为下标，第二个列是元素，后面跟的a[$1]是通过第一列取a数组元素（服务名），结果是$1=$2 $2，并作为a数组元素。

  3）将第一行附加给下面每行开头

  $ cat a.txt

  xiaoli

  a 100

  b 110

  c 120

  $ awk 'NF==1{a=$0;next}{print a,$0}' a.txt

  $ awk 'NF==1{a=$0}NF!=1{print a,$0}' a.txt

  xiaoli  a 100

  xiaoli  b 110

  xiaoli  c 120

4、倒叙列打印文本

  $ cat a.txt

  xiaoli  a 100

  xiaoli  b 110

  xiaoli  c 120

  $ awk '{for(i=NF;i>=1;i--){printf "%s ",$i}print s}' a.txt

  100 a xiaoli

  110 b xiaoli

  120 c xiaoli

  $ awk '{for(i=NF;i>=1;i--)if(i==1)printf $i"\n";else printf $i" "}' a.txt

  说明：利用NF降序输出，把最后一个域作为第一个输出，然后自减，print s或print ""打印一个换行符

5、从第二列打印到最后

  方法1：$ awk '{for(i=2;i<=NF;i++)if(i==NF)printf $i"\n";else printf $i" "}' a.txt

  方法2：$ awk '{$1=""}{print $0}' a.txt

  a 100

  b 110

  c 120

6、将c文件中第一列放到到d文件中的第三列

  $ cat c

  a

  b

  c

  $ cat d

  1 one

  2 two

  3 three

  方法1：$ awk 'FNR==NR{a[NR]=$0;next}{$3=a[FNR]}1' c d

  说明：以NR编号为下标，元素是每行，当处理d文件时第三列等于获取a数据FNR（重新计数1-3）编号作为下标。

  方法2：$ awk '{getline f<"c";print $0,f}' d

  1 one a

  2 two b

  3 three c

  1）替换第二列

  $ awk '{getline f<"c";gsub($2,f,$2)}1' d

  1 a

  2 b

  3 c

  2）替换第二列的two

  $ awk '{getline f<"c";gsub("two",f,$2)}1' d

  1 one

  2 b

  3 three

7、数字求和

  方法1：$ seq 1 100 |awk '{sum+=$0}END{print sum}'

  方法2：$ awk 'BEGIN{sum=0;i=1;while(i<=100){sum+=i;i++}print sum}'

  方法3：$ awk 'BEGIN{for(i=1;i<=100;i++)sum+=i}END{print sum}' /dev/null

  方法4：$ seq -s + 1 100 |bc

8、每隔三行添加一个换行符或内容

  方法1：$ awk '$0;NR%3==0{printf "\n"}' a

  方法2：$ awk '{print NR%3?$0:$0"\n"}' a

  方法3：$ sed '4~3s/^/\n/' a

9、字符串拆分

  方法1：

  $ echo "hello" |awk -F '' '{for(i=1;i<=NF;i++)print $i}'

  $ echo "hello" |awk -F '' '{i=1;while(i<=NF){print $i;i++}}'

  h

  e

  l

  l

  o

  方法2：

  $ echo "hello" |awk '{split($0,a,"''");for(i in a)print a[i]}'  #无序

  l

  o

  h

  e

  l

10、统计字符串中每个字母出现的次数

  $ echo a,b.c.a,b.a |tr "[,. ]" "\n" |awk -F '' '{for(i=1;i<=NF;i++)a[$i]++}END{for(i in a)print i,a[i]|"sort -k2 -rn"}'

  a 3

  b 2

  c 1

11、第一列排序

  $ awk '{a[NR]=$1}END{s=asort(a,b);for(i=1;i<=s;i++){print i,b[i]}}' a.txt

  说明：以每行编号作为下标值为$1，并将a数组值放到数组b，a下标丢弃，并将asort默认返回值（原a数组长度）赋值给s，使用for循环小于s的行号，从1开始到数组长度打印排序好的数组。

12、删除重复行，顺序不变

  $ awk '!a[$0]++' file

13、删除指定行

  删除第一行：

  $ awk 'NR==1{next}{print $0}' file #$0可省略

  $ awk 'NR!=1{print}' file

  $ sed '1d' file

  $ sed -n '1!p' file

14、在指定行前后加一行

  在第二行前一行加txt：

  $ awk 'NR==2{sub('/.*/',"txt\n&")}{print}' a.txt

  $ sed'2s/.*/txt\n&/' a.txt

  在第二行后一行加txt：

  $ awk 'NR==2{sub('/.*/',"&\ntxt")}{print}' a.txt

  $ sed'2s/.*/&\ntxt/' a.txt

15、通过IP获取网卡名

  $ ifconfig |awk -F'[: ]' '/^eth/{nic=$1}/192.168.18.15/{print nic}'

16、浮点数运算（数字46保留小数点）

  $ awk 'BEGIN{print 46/100}'

  $ awk 'BEGIN{printf "%.2f\n",46/100}'

  $ echo 46|awk '{print $0/100}'

  $ echo 'scale=2;46/100' |bc|sed 's/^/0/'

  $ printf "%.2f\n" $(echo "scale=2;46/100" |bc)

  结果：0.46

17、替换换行符为逗号

  $ cat a.txt

  1

  2

  3

  替换后：1,2,3

  方法1：

  $ awk '{s=(s?s","$0:$0)}END{print s}' a.txt

  说明：三目运算符(a?b:c)，第一个s是变量，s?s","$0:$0,第一次处理1时，s变量没有赋值初值是0，0为假，结果打印1，第二次处理2时，s值是1，为真，结果1,2。以此类推，小括号可以不写。

  方法2：

  $ tr '\n' ',' < a.txt

  方法3：

  $ sed ':a;N;s/\n/,/;$!b a' a.txt

  说明：第一个标签a，先读取第一行记录1追加到模式空间，此时模式空间内容是1$，执行$!b（$!最后一行不跳转，b是控制流跳转命令）跳转到a标签，继续读取第二行记录2追加到模式空间，因为使用N命令，每个记录以换行符（\n）分割，此时模式空间内容是1\n2$，执行将换行符替换逗号命令，继续跳转到a标签...

  方法4：

  $ sed ':a;$!N;s/\n/,/;t a' a.txt

  说明：与上面类似，其中t是测试命令，当上一个命令（替换）执行成功才跳转。

  方法5：

  $ awk '{if($0!=3)printf "%s,",$0;else print $0}' a.txt

  说明：3是文本最后一个数

  方法6：

  while read line; do

​    a+=($line)

  done < a.txt

  echo ${a[*]} |sed 's/ /,/g'

  说明：将每行放到数组，然后替换

18、把奇数换行符去掉

  $ cat b.txt

  string

  number

  a

  1

  b

  2

  $ awk 'ORS=NR%2?"\t":"\n"' b.txt  #把奇数行换行符去掉

  $ xargs -n2 < a.txt  #将两个字段作为一行

  string number

  a 1

  b 2

19、费用统计

  $ cat a.txt

  姓名       费用  数量

  zhangsan     8000   1

  zhangsan     5000   1

  lisi       1000   1

  lisi       2000   1

  wangwu      1500   1

  zhaoliu     6000   1

  zhaoliu     2000   1

  zhaoliu     3000   1

  统计每人总费用、总数量：

  $ awk '{name[$1]++;number[$1]+=$3;money[$1]+=$2}END{for(i in name)print i,number[i],money[i]}' a.txt

  zhaoliu 3 11000

  zhangsan 2 13000

  wangwu 1 1500

  lisi 2 3000

20、打印乘法口诀

  方法1：

  $ awk 'BEGIN{for(n=0;n++<9;){for(i=0;i++<n;)printf i"x"n"="i*n" ";print ""}}'

  1x1=1

  1x2=2 2x2=4

  1x3=3 2x3=6 3x3=9

  1x4=4 2x4=8 3x4=12 4x4=16

  1x5=5 2x5=10 3x5=15 4x5=20 5x5=25

  1x6=6 2x6=12 3x6=18 4x6=24 5x6=30 6x6=36

  1x7=7 2x7=14 3x7=21 4x7=28 5x7=35 6x7=42 7x7=49

  1x8=8 2x8=16 3x8=24 4x8=32 5x8=40 6x8=48 7x8=56 8x8=64

  1x9=9 2x9=18 3x9=27 4x9=36 5x9=45 6x9=54 7x9=63 8x9=72 9x9=81

  方法2：

  \#!/bin/bash

  for ((i=1;i<=9;i++)); do

​    for ((j=1;j<=i;j++)); do

​     result=$(($i*$j))

​     \#let "result=i*j"

​     echo -n "$i*$j=$result "

​    done

​    echo

  done

21、只打印奇数或偶数行

  打印奇数行：

  方法1：

  $ seq 1 5 |awk 'i=!i'

  说明：先知道对于数值运算，未定义变量初值为0，对于字符运算，未定义变量初值为空字符串。

  读取第一行记录，然后进行模式匹配，i是未定义变量，也就是i=!0，!取反意思。感叹号右边是个布尔值，0或空字符串为假，非0或非空字符串为真，!0就是真，因此i=1，条件为真打印第一条记录。

  没有print为什么会打印呢？因为模式后面没有动作，默认会打印整条记录。

  读取第二行记录，进行模式匹配，因为上次i的值由0变成了1，此时就是i=!1，条件为假不打印。

  读取第三行记录，因为上次条件为假，i恢复初值为0，继续打印。以此类推...

  可以看出，运算时并没有判断记录，而是利用布尔值真假判断。

  方法2：

  $ seq 1 5 |awk 'NR%2!=0'

  方法3：

  $ seq 1 5 |sed -n '1~2p'

  说明：步长，每隔一行打印一次

  方法4：

  $ seq 1 5 |sed -n 'p;n'

  说明：先打印第一行，执行n命令读取当前行的下一行2，放到模式空间，后面再没有打印模式空间行操作，所以只保存不打印，同等方式继续打印第三行。

  1

  3

  5

  打印偶数行：

  $ seq 1 5 |awk '!(i=!i)'

  $ seq 1 5 |awk 'NR%2==0'

  $ seq 1 5 |sed -n '0~2p'

  $ seq 1 5 |sed -n 'n;p'

  说明：读取当前行的下一行2，放到模式空间，使用p命令打印模式空间的行，输出2。