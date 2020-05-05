[TOC]

# 定义

一个个字符组成的有序的序列，是字符的集合

使用单引号、双引号、三引号引住的字符序列

字符串是不可变对象

Python3起，字符串就是Unicode类型

```
s1 = 'string'
s2 = "string2"
s3 = '''this's a "String" '''
s4 = 'hello \n magedu.com'  # 解释\n
s5 = r"hello \n magedu.com"  # 不解释\n
s6 = 'c:\windows\nt'
s7 = R"c:\windows\nt"
s8 = 'c:\windows\\nt'
sql = """select * from user where name='tom' """
```



## 转义符

右斜杠可以转义很多字符:

\n 表示换行

\t 表示制表符

\\ 表示右斜杠本身

如果字符串里面有很多字符都需要转义,就需要加很多\,为了简化,Python还允许用r' ',表示' '内部的所有字符串默认不转义.

例如：

```
a = "I\'m \'OK\'!"
b = r"I'm 'OK'!"
print(a)
print(b)
'''
运行结果：
I'm "OK"!
I'm 'OK'!
'''
```



## __add__

str类的隐藏参数,详细的知识在学习到面向对象在详细分析.

目前用途为 v = v1 + v2 本质上就是调用了str()中的__add__方法

# 方法

## 元素访问

字符串支持使用索引访问

```
sql = "select * from user where name='tom'"
sql[4] # 字符串'c'
sql[4] = 'o'
```

有序的字符集合，字符序列

```
for c in sql:
    print(c)
print(type(c))  # <class 'str'>
```

可迭代

```
lst = list(sql)
print(lst)
```



## 切片

通过索引区间访问线性结构的一段数据;

sequence[start:stop] 表示返回[start, stop)区间的子序列;

支持负索引;

start为0，可以省略;

stop为末尾，可以省略;

超过上界（右边界），就取到末尾；超过下界（左边界），取到开头;

start一定要在stop的左边;

[:] 表示从头至尾，全部元素被取出，等效于copy()方法;

示例：

```
a = 'www.magedu.com'
b = b'www.magedu.com'
print(a[4:10])
print(a[:10])
print(a[4:])
print(a[:])
print(a[:-1])
print(a[4:-4])
print(a[4:50])
print(b[-40:10])

res1 = bytearray(b'www.magedu.com')[-4:10]
print(res1)
res2 = list('www.magedu.com')[-10:-4]
print(res2)
res3 = tuple('www.magedu.com')[-10,10]  # 报错,元组不支持切片
print(res3)
```

步长切片

[start:stop:step]

step为步长，可以正、负整数，默认是1;

step要和start:stop同向，否则返回空序列;

示例：

```
'www.magedu.com'[4:10:2]
list('www.magedu.com')[4:10:-2]
tuple('www.magedu.com')[-10:-4:2]
b'www.magedu.com'[-4:-10:2]
bytearray(b'www.magedu.com')[-4:-10:-2]
```



### 总结

#### 线性结构

可迭代for ... in

len()可以获取长度

通过下标可以访问

可以切片

学过的线性结构

列表、元组、字符串、bytes、bytearray

#### 切片

通过索引区间访问线性结构的一段数据

sequence[start:stop] 表示返回[start, stop)区间的子序列

支持负索引

start为0，可以省略

stop为末尾，可以省略

超过上界（右边界），就取到末尾；超过下界（左边界），取到开头

start一定要在stop的左边

[:] 表示从头至尾，全部元素被取出，等效于copy()方法

#### 步长切片

[start:stop:step]

step为步长，可以正、负整数，默认是1

step要和start:stop同向，否则返回空序列

## 拼接

### join

"string".join(iterable) -> str

将可迭代对象连接起来，使用string作为分隔符

可迭代对象本身元素都是字符串

返回一个新字符串

```
lst = ['1','2','3']
print("\"".join(lst))  # 分隔符是双引号
print(" ".join(lst))
print("\n".join(lst))
```



### +

\+ -> str

将2个字符串连接在一起

返回一个新字符串

## 分割

### split系

将字符串按照分隔符分割成若干字符串，并返回列表

#### split

str.split()：不保留分隔符,结果为列表

split(sep=None, maxsplit=-1) -> list of strings

从左至右

sep 指定分割字符串，缺省的情况下空白字符串作为分隔符

maxsplit 指定分割的次数，-1 表示遍历整个字符串

#### rsplit

rsplit(sep=None, maxsplit=-1) -> list of strings

从右向左

sep 指定分割字符串，缺省的情况下空白字符串作为分隔符;

maxsplit 指定分割的次数，-1 表示遍历整个字符串;

#### splitlines

splitlines([keepends]) -> list of strings

按照行来切分字符串;

keepends 指的是是否保留行分隔符,行分隔符包括\n、\r\n、\r等;

### partition系

将字符串按照分隔符分割成2段，返回这2段和分隔符的元组

#### partition

partition(sep) -> (head, sep, tail)

从左至右，遇到分隔符就把字符串分割成两部分，返回头、分隔符、尾三部分的三元组；如果没有找到分隔符，就返回头、2个空元素的三元组;

sep 分割字符串，必须指定

str.partition()：保留分隔符,结果为元组

#### rpartition

rpartition(sep) -> (head, sep, tail)

从右至左，遇到分隔符就把字符串分割成两部分，返回头、分隔符、尾三部分的三元组；如果没有找到分隔符，就返回2个空元素和尾的三元组

## 大小写

### upper()

全大写

### lower()

全小写，仅支持英文转换

### casefold()

全小写，支持转换英文和德文

### swapcase()

交互大小写

首字母大写

### capitalize()

str.capitalize()：忽略大小写,全部转换成首字母大写其余小写的格式

### 判断大小写字母

str.islower()：判断小写，返回True和False

str.isupper()：判断大写，返回True和False

## 排版

以对象为起点.ljust就是让对象在左边,向后填充x长度的内容y

str.title()：标题的每个单词都大写

str.ljust(宽度, '填充符')

str.rjust(宽度, '填充符')

str.center(总宽度, 填充符)

str.expandtabs(宽度) ：以制表符\t为分隔符,填充替换为指定长度

str.zfill(width)：width 打印宽度，居右，左边用0填充

str.istitle()：判断是否为标题格式，即首字母大写,全字母格式

## 修改

### replace()

replace(old, new[, count]) -> str

字符串中找到匹配替换为新子串，返回新字符串

count表示替换几次，不指定就是全部替换

str.replace('old', 'new', 替换次数)：替换,将旧的值替换为新的值,n表示从左往右替换几次

```
'www.magedu.com'.replace('w','p')
'www.magedu.com'.replace('w','p',2)
'www.magedu.com'.replace('w','p',3)
'www.magedu.com'.replace('ww','p',2)
'www.magedu.com'.replace('www','python',2)
```



### strip()

strip([chars]) -> str

str.strip()

移除空白,包括\n,\t，还有自定义移除

从字符串两端去除指定的字符集chars中的所有字符

如果chars没有指定，去除两端的空白字符

s.strip(): 移除所有

s.lstrip(): 移除左边

s.rstrip(): 移除右边

```
s = "\r \n \t Hello Python \n \t"
s.strip()
s = " I am very very very sorry "
s.strip('Iy')
s.strip('Iy ')
```



## 查找

### find()

find(sub[, start[, end]]) -> int

在指定的区间[start,end)，从左至右，查找子串sub。找到返回索引，没找到返回-1;

rfind(sub[, start[, end]]) -> int

在指定的区间[start,end)，从右至左，查找子串sub。找到返回索引，没找到返回-1;

str.find('str')

查找关键字的索引位置,不存在则返回-1而不会报错，和index()不同,index如果查找的关键字不存在,则会报错终止程序。

```
s = "I am very very very sorry"
print(s.find('very'))  # 5
print(s.find('very', 5))  # 5
print(s.find('very', 6, 13))  # -1
print(s.rfind('very', 10))  # 15
print(s.rfind('very', 10, 15))  # 10
print(s.rfind('very',-10,-1))  # 15
```



### index

index(sub[, start[, end]]) -> int

在指定的区间[start,end)，从左至右，查找子串sub。找到返回索引，没找到抛出异常ValueError

rindex(sub[, start[, end]]) -> int

在指定的区间[start,end)，从左至右，查找子串sub。找到返回索引，没找到抛出异常ValueError

```
s = "I am very very very sorry"
s.index('very')  # 5
s.index('very', 5)  # 5
s.index('very', 6, 13) # 报错
s.rindex('very', 10)  # 15
s.rindex('very', 10, 15)  # 10
s.rindex('very',-10,-1)  # 15
```



## 统计

### count()

str.count(sub[, start[, end]])

sub：要统计的目标字串

start：起始位置(索引),从哪开始统计

end：结束位置(索引),从哪结束统计

在指定的区间[start, end)，从左至右，统计子串sub出现的次数

### len()

srt.len()

返回字符串的长度，即字符的个数

### 时间复杂度

index和count方法都是O(n)

随着列表数据规模的增大，而效率下降

## 判断

endswith(suffix[, start[, end]]) -> bool

在指定的区间[start, end)，字符串是否是suffix结尾

startswith(prefix[, start[, end]]) -> bool

在指定的区间[start, end)，字符串是否是prefix开头

isalnum() -> bool 判断是否是数字、字母、汉字组成,含有有特殊符号则False

isalpha() 判断是否不含数字,有数字则False,无数字则True

isdecimal() 是否只包含十进制数字，仅识别阿拉伯数字,如1,2,3

isdigit() 是否全部数字(0~9)，除了阿拉伯数字,也可识别特殊的数字格式,如 1,2,①,②

str.isnumeric()：可识别大多数数字表示类型,如1,2,①,②,一,二

str.isidentifier() 是不是字母和下划线开头，其他都是字母、数字、下划线，

判断变量的值是否允许作为标识符,即是否允许作为变量名.

注意,不能识别关键字类型

str.islower() 是否都是小写

str.isupper() 是否全部大写

str.isspace() 是否只包含空白字符

## 格式化

字符串的格式化是一种拼接字符串输出样式的手段，更灵活方便

join拼接只能使用分隔符，且要求被拼接的是可迭代对象

\+ 拼接字符串还算方便，但是非字符串需要先转换为字符串才能拼接

在2.5版本之前，只能使用printf style风格的print输出

printf-style formatting，来自于C语言的printf函数

格式要求：

占位符：使用%和格式字符组成，例如%s、%d等

s调用str()，r会调用repr()。所有对象都可以被这两个转换。

占位符中还可以插入修饰字符，例如%03d表示打印3个位置，不够前面补零

format % values，格式字符串和被格式的值之间使用%分隔

values只能是一个对象，或是一个和格式字符串占位符数目相等的元组，或一个字典

#### %占位符

%c : ASCII

%s : 字符串

%d : 整数

%u : 无符号整型

%o : 无符号八进制数

%x : 无符号十六进制数

%X : 无符号十六进制数(大写)

%f : 浮点数字

%e : 科学计数法格式化浮点数

%g : %f 和 %e 的简写

%p : 用十六进制数格式化变量的地址

### format()

format函数格式字符串语法，Python鼓励使用

"{} {xxx}".format(*args, **kwargs) -> str

args是位置参数，是一个元组

kwargs是关键字参数，是一个字典

花括号表示占位符

{}表示按照顺序匹配位置参数，{n}表示取位置参数索引为n的值

{xxx}表示在关键字参数中搜索名称一致的

{{}} 表示打印花括号

#### 位置参数

"{}:{}".format('192.168.1.100',8888)，这就是按照位置顺序用位置参数替换前面的格式字符串的占位符中

#### 关键字参数或命名参数

```
"{server} {1}:{0}".format(8888, '192.168.1.100', server='Web Server Info : ') 
```

位置参数按照序号匹配，关键字参数按照名词匹配

#### 访问元素

```
"{0[0]}.{0[1]}".format(('magedu','com'))
```



#### 对象属性访问

```
from collections import namedtuple
Point = namedtuple('Point','x y')
p = Point(4,5)
"{{{0.x},{0.y}}}".format(p)
```



#### 对齐

```
'{0}*{1}={2:<2}'.format(3,2,2*3)
'{0}*{1}={2:<02}'.format(3,2,2*3)
'{0}*{1}={2:>02}'.format(3,2,2*3)
'{:^30}'.format('centered')
'{:*^30}'.format('centered')
```



#### 进制

```
"int: {0:d}; hex: {0:x}; oct: {0:o}; bin: {0:b}".format(42)
"int: {0:d}; hex: {0:#x}; oct: {0:#o}; bin: {0:#b}".format(42)
octets = [192, 168, 0, 1]
'{:02X}{:02X}{:02X}{:02X}'.format(*octets)
```



### str.format_map({字典})

```
a = "我是:{name};年龄:{age};性别:{gender}"
v = a.format_map({'name':"李杰",'age':19,'gender':'中'})
print(v)
"""
我是:李杰;年龄:19;性别:中
"""
```



## 映射

.maketrans(参数1,参数2):用于创建字符映射的转换表

注意:两个字符串的长度必须相同,为一一对应的关系

.maketrans(intab, outtab)

参数说明:

intab -- 字符串中要替换的字符串

outtab -- 转换的目标

.translate()

结合上面maketrans()使用

str.translate(tab, del)

bytes.translate(tab, del)

bytearray.translate(tab, del)

tab -- 翻译表,可理解为定制的转换规则,是通过maketrans()方法转换而来,例如:(b'abcd', b'1234')

del -- 要过滤掉的字符列表

```
intab = "aeiou"
outtab = "12345"
str_trans = str.maketrans(intab, outtab)  # 定义替换规则
str_test = "This is string example,..wow!!"  # 要处理的对象
v = str_test.translate(str_trans)
print(v)
"""
Th3s 3s str3ng 2x1mpl2,..w4w!!
"""

str1 = '123456789'  # 处理目标
str2 = v.maketrans('13579', 'acegi')  # 指定规则
k = str1.translate(str2)  # 应用规则
# k结果:'a2c4e6g8i'
```



## 编码转换

str.encode(encoding='编码')：以指定编码转换成字节形式，不指定字符类型默认utf-8

```
name = "张益达"
n1 = name.encode(encoding='utf-8')
n2 = name.encode(encoding='gbk')
print(n1)
print(n2)
"""
b'\xe5\xbc\xa0\xe7\x9b\x8a\xe8\xbe\xbe'
b'\xd5\xc5\xd2\xe6\xb4\xef'
"""
```