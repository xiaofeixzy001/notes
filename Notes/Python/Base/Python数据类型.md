[TOC]

# 数据类型

## 查看数据类型

### isinstance()

格式： 
 isinstance(object, class_or_tuple)，返回布尔值

参数说明: 
 object表示实例，classinfo可以是直接或间接类名、基本类型或者有它们组成的元组

```
isinstance(2, float)  # False
isinstance('a', (str, unicode))  # True
isinstance((2, 3), (str, list, tuple))  # True
```

### type()

格式： 
 type(obj)，返回类型，而不是字符串

```
a, b, c, d = 20, 5.5, True, 4+3j

type(a)  # <class 'int'>
type(b)  # <class 'float'>
type(c)  # <class 'bool'>
type(d)  # <class 'complex'>

type(1+False)  # <class 'int'>，隐式转换
```

对比： 
 type()不会认为子类是一种父类类型 
 isinstance()会认为子类是一种父类类型

type()不会认为子类是一种父类类型 
 isinstance()会认为子类是一种父类类型 
 id()

## 数值型

### 整型int

python3的int就是长整型 ，且没有大小限制，受限于内存区域的大小。

转换：int(x)

### 浮点型float

由整数部分和小数部分组成。支持十进制和科学计数法表示，只有双精度型。

当传入对象为空时,False,否则为真True. 
 转换: float(x)

### 布尔型bool

int的子类，仅有2个实例True,False，分别对应1和0，可以和整数直接运算。

转换: bool(x)

特殊的False类型： 
 0, '', [], {}, (), False

```
# 示例1
a1 = 0
a2 = 100
a3 = -100
a1_bool = bool(a1)
a2_bool = bool(a2)
a3_bool = bool(a3)
print('a1 %s:%s bool_type:%s' % (a1,a1_bool, type(a1_bool)))
print('a2 %s:%s bool_type:%s' % (a2,a2_bool, type(a1_bool)))
print('a3 %s:%s bool_type:%s' % (a3,a2_bool, type(a1_bool)))
# 运行结果
a1 0:False bool_type:<class 'bool'>
a2 100:True bool_type:<class 'bool'>
a3 -100:True bool_type:<class 'bool'>

# 示例2
b1 = ''
b2 = 'fafa'
b3 = 'AFSD@'
b1_bool = bool(b1)
b2_bool = bool(b2)
b3_bool = bool(b3)
print('b1 %s:%s bool_type:%s'%(b1,b1_bool, type(a1_bool)))
print('b2 %s:%s bool_type:%s'%(b2,b2_bool, type(a1_bool)))
print('b3 %s:%s bool_type:%s'%(b3,b3_bool, type(a1_bool)))
# 运行结果
b1 :False bool_type:<class 'bool'>
b2 fafa:True bool_type:<class 'bool'>
b3 AFSD@:True bool_type:<class 'bool'>
```

可使用and,or,not运算

- and 与运算，只有所有条件为True，and 运算结果才是True;
- or 或运算，只要有一个为True，or运算结果就是True;
- not 非运算，把True变成False， False变成 True;
- 注: 
   在 Python2 中是没有布尔型的,它用数字 0 表示 False,用 1 表示 True. 
   到 Python3 中,把 True 和 False 定义成关键字了,但它们的值还是 1 和 0,它们可以和数字相加.

### 复数complex

由实数和虚数部分组成，实数和虚数部分都是浮点数，例如：3+4.2j

转换：complex(x),comple(x, y)

## 字符串str

创建: a = 'apple|pie' 
 转换: str(a) 
 整个字串被当作一个序列,可通过索引来引用某一个元素

## 列表list

有序的对象集合 
 格式: [1,2,3,a] 
 转换: list()

## 字典dict

无序的对象集合 
 格式: {'a':1, 'b':'2', 'c':'hello'} 
 转换: dict()

## 元组tuple

格式: (1,2,e,a) 
 转换: tuple()

## 集合set

无序不重复元素的序列 
 格式: {1,a,'hello',3.14} 
 转换: set()

## 可变与不可变类型

不可变数据（四个）：Number（数字）、String（字符串）、Tuple（元组）、Sets（集合） 
 可变数据（两个）：List（列表）、Dictionary（字典）

## 空值

空值是Python里一个特殊的值，用None表示。None不能理解为0，因为0是有意义的，而None是一个特殊的空值。

## 转义字符\

可以转义很多字符，比如\n表示换行，\t表示制表符，字符\本身也要转义，所以\表示的字符就是\;如果字符串里面有很多字符都需要转义，就需要加很多\，为了简化，Python还允许用r''表示''内部的字符串默认不转义.

# 常见数据类型转换

int(x): 转换为整型 

float(x): 转换为浮点型

complex(real [,imag]): 创建复数

str(x): 转换为字符型

repr(x): 转换为表达式字符串

eval("exp"): 转换表达式,返回对象

list(s): 转换为列表

tuple(s): 转换为元组

set(s): 转换为可变集合 

dict(d): 创建一个字典,d为k-v格式的元组 

frozenset(s): 转换为不可变集合 

unichr(x): int转换为Unicode

chr(x): int转换为str 

ord(x): 字符转换为它的整数值

hex(x): int转换为十六进制字符串

oct(x): int转换为八进制字符串

# 变量

### 命名规范

由字母、数字、下划线组成； 
 不能以数字开头； 
 区分大小写； 
 不能使用python内置的关键字

如:

```
# 不需要特意去记,以后用的多了自然会记住
'and', 'as', 'assert', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'exec', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'not', 'or', 'pass', 'print', 'raise', 'return', 'try', 'while', 'with', 'yield'
```

### 命名习惯

1. 以单一下划线开头的变量名(_x)不会被诸如from module import,面向对象和面向过程,面向对象和面向过程,面向对象和面向过程语句导入；
2. 前后有下划线的变量名(**x**)是系统变量名，对解释器有特殊意义；
3. 以两个下划线开头、但结尾没有下划线的变量名(__x)是类的本地变量；
4. 交互式模式下，只有单个下划线的变量名(_)用于保存最后表达式的结果；

注意: 变量没有类型,对象才有

### 变量的赋值

使用等号'='

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# FileName:hello.py
name1 = "user1"
name2 = "user2"
```

### 变量的赋值原理

实际上就是在内存空间里面开辟出一块用来存放赋予的数据user1，user2，这个变量name1和name2就是代表数据在内存空间中的位置的一个路标

![img](Python%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.assets/0450f9cc-168f-4e42-96bd-1be78ae33ebb.jpg)

# 运算符

### 赋值操作符：

=： 变量赋值，a = b + c 
 +=： a += b 等同于 a = a + b 
 -=： a -= b 等同于 a = a - b 
*=： a* = b 等同于 a = a * b 
 %=： a %= b 等同于 a = a % b 
**=： a** = b 等同于 a = a ** b

### 转义字符：

: 字符串太长，换一行接着输入 
 \' 或 \": 单引号和双引号 
 \r: 光标 
 \t: 横向制表符（tab键） 
 \v: 纵向制表符 
 \n: 换行符,打印到下一行

### 运算操作符：

+，-，*，%(取模)，**(指数)

1. 算术运算 

round()，四舍六入五取偶
floor()向下取整、ceil()向上取整
int() 取整数部分
// 整除且向下取整


![img](Python%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.assets/5945ab96-7e15-489b-a6fd-fa75d0028b5f.jpg)

1. 比较运算 
   ![img](Python%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.assets/e948d4bf-7c16-4239-92a2-a603a7574b1c.jpg)
2. 赋值运算 
   ![img](Python%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.assets/a61f48f8-6a63-48e5-a2dc-e202d074ce95.jpg)
3. 逻辑运算 
   ![img](Python%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.assets/df5067c6-58a8-4ef5-b564-07f8606a9fc1.jpg)
4. 成员运算 
   ![img](Python%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.assets/09a8ba9c-a490-43dd-a1d1-c550d3d27fdb.jpg)

### 运算符优先级

算数运算符 -> 位运算符 -> 身份运算符 -> 成员运算符 -> 逻辑运算符 
 记不住,用括号 
 长表达式,多用括号,易懂易读

### 位运算符

计算机中的数字都是用二进制形式表示的. 
 在python里面,数字或字符串前有 
0b表示是二进制; 
 0x表示十六进制; 
 0o代表八进制.

示例:

```
# 二进制 -> 十进制
0b1 -> 1
0b10 -> 2
0b1111 -> 15

# 十六进制 -> 十进制
0x10 -> 16
0xff -> 255

# 八进制 -> 十进制
0o10 -> 8
0o17 -> 15
```

二进制数字有自己的特殊运算，是对每一位数字分别进行的操作，所以叫做位操作，Python共有以下几种位操作符：

- x >> y: 返回 x 向右移 y 位得到的结果
- x << y: 返回 x 向左移 y 位得到的结果
- x & y: 且操作，返回结果的每一位是 x 和 y 中对应位做 and 运算的结果，只有 1 and 1 = 1，其他情况位0
- x | y : 或操作，返回结果的每一位是 x 和 y 中对应位做 or 运算的结果，只有 0 or 0 = 0，其他情况位1
- ~x: 反转操作，对 x 求的每一位求补，只需记住结果是 -x - 1
- x ^ y: 或非运算，如果 y 对应位是0，那么结果位取 x 的对应位，如果 y 对应位是1，取 x 对应位的补

示例:

```
# 1,左右位移操作
"""
向右移1位可以看成除以2
向左移一位可以看成乘以2
移动n位可以看成乘以或者除以2的n次方
"""
8 >> 2 <=> 8 / 2 / 2 <=> 0b1000 >> 2 = 0b10 = 2
8 << 2 <=> 8 * 2 * 2 <=> 0b1000 << 2 = 0b100000 = 32

# 2,且操作
"""
两个数字的且操作就是对每一位进行且操作取结果
"""
0b1 & 0b0 = 0
0b1111 & 0b1010 = 0b1010 = 10
0b1010 & 0b1100 = 0b1000 = 8

# 3,或操作
"""
两个数字的或操作就是对每一位进行或操作取结果
"""
0b1 | 0b0 = 0b1 =1
0b1000 | 0b0111 = 0b1111 = 15
0b1010 | 0b1100 = 0b1110 = 14

# 4,取反操作
'''
python的反转操作只接受一个参数n，n必须是整数，效果是对n的内部表示的每一位求补，运算结果: -n-1
如: ~8 = -9
一些同学可能会疑惑，~8不应该是 ~0b1000 = 0b0001 = 1 才对吗.
事情是这样的，计算机在内部表示负整数的时候用的是正数的补，比如 0b0001 是1，它的补是 0b1110，这个时候0b1110 在计算机内部不是7，而是-1。 这样一来，可以推导出来~n的结果是 -n-1 。不过你自己写的0b1111在这个语境下并不是一个负数，所以结果仍是15.
'''

# 5,或非操作
'''
对于 x ^ y，如果y的位是0，那么取x的原始值，如果y的位是1，那么取x此位的补，例如
'''
0b1111 ^ 0b0101 = 0b1010
0b1111 ^ 0b1 = 0b1110 # 自动填0
```

### 补充

- 二进制: 
   1对应 00000001 
   3对应 00000011 
   最多支持2的8次幂 
   表示符号: b 
   十进制转二进制转换方式:

```
>>> bin(10)
'0b1010'
```

- 八进制: 
   逢8进1位 1 2 3 4 5 6 7 10 
   表示符号: o 
   十进制转八进制转换方式:

```
>>> oct(123)
'0o173'
```

- 十进制: 
   0 1 2 3 4 5 6 7 8 9 10 
   表示符号: d 
   十进制转十六进制转换方式:

```
>>> hex(10)
'0xa'
```

- 十六进制: 
   逢16进1位 
   1 2 3 4 5 6 7 8 9 A B C D E F 
   表示符号: x 
   十六进制转二进制转换方式:

```
>>> bin(int('fc',16))
'0b11111100'
```

- 利用函数直接转

```
>>> bin(0xa)
'0b1010'
>>> oct(0xa)
'012'
>>> hex(10)
'0xa'
```

持续更新...