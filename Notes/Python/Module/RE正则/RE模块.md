[TOC]

# re

python使用re模块提供了正则表达式处理的能力

 

```
s = """bottle\nbag\nbig\napple"""

for i, c in enumerate(s, 1):
    print((i-1, c), end='\n' if i % 8 == 0 else ' ')
print()
'''
(0, 'b') (1, 'o') (2, 't') (3, 't') (4, 'l') (5, 'e') (6, '\n') (7, 'b') (8, 'a') (9, 'g') (10, '\n') (11, 'b') (12, 'i') (13, 'g') (14, '\n') (15, 'a') (16, 'p') (17, 'p') (18, 'l') (19, 'e')
'''
```

## 常量

re.M & re.MULTILINE  多行模式

re.S & re.DOTALL  单行模式

re.l & re.IGNORECASE  忽略大小写

re.X & re.VERBOSE  忽略表达式中的空白字符

## 方法

### compile

re.compile(pattern, flags=0)

设定flags，编译模式，返回正则表达式对象regex

pattern就是正则表达式字符串

flags是选项

正则表达式需要被编译，为了提高效率，这些编译后的结果被保存，下次使用的同样的pattern的时候，就不需要再次编译

re的其他方法为了提高效率都调用了编译方法，就是为了提速。

### 单词匹配

#### match

re.match(pattern, string, flags=0)

从字符串开头进行匹配查找,只找一次，没有则返回None，同search(^)

regex.match(string[, pos[, endpos]])

regex对象的match方法可以指定查找的开始位置和结束位置，返回match对象

 

```
s = """bottle\nbag\nbig\napple"""

# 方式1
print(re.match('b', s))  # <_sre.SRE_Match object; span=(0, 1), match='b'>
print(re.match('o', s))  # None
print(re.match('t', s))  # None
print(re.match('^t', s, re.M))  # None,从头开始找,多行模式没用
print(re.match('^t', s, re.S))  # None,从头开始找,单行模式没用

# 方式2
# 先将pattern编译,然后在使用编译后的对象调用match方法
regex = re.compile('t')
print(regex.match(s))  # None，从头开始
print(regex.match(s, 2))  # 从索引为2开始查找

```

re.match只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回None

#### search

re.search(pattern, string, flags=0)

只找到第一次匹配到的条件，返回一个包含匹配信息的对象,该对象可通过调用group()方法来得到匹配的字符串,没有匹配则返回None

regex.search(string[, pos[, endpos]])

从头搜索，直到第一个匹配，regex对象的search方法可以重设开始位置和结束位置，返回match对象

 

```
s = """bottle\nbag\nbig\napple"""

# 方式1
print(re.search('b', s))
print(re.search('o', s))
print(re.search('t', s))
'''
<_sre.SRE_Match object; span=(0, 1), match='b'>
<_sre.SRE_Match object; span=(1, 2), match='o'>
<_sre.SRE_Match object; span=(2, 3), match='t'>
'''

# 方式2
regex = re.compile('b')
print(regex.search(s, 1))  # <_sre.SRE_Match object; span=(7, 8), match='b'>

regex = re.compile('^b', re.M)
print(regex.search(s))  # <_sre.SRE_Match object; span=(0, 1), match='b'>
print(regex.search(s, 8))  # <_sre.SRE_Match object; span=(11, 12), match='b'>
# 不管是否多行查找,找到就返回,不在继续
```

re.search匹配整个字符串，直到找到第一个匹配项,后面就不在找了

#### fullmatch

re.fullmatch(pattern, string, flags=0)

完全匹配

regex.fullmatch(string[, pos[, endpos]])

整个字符串和正则表达式进行匹配

 

```
s = """bottle\nbag\nbig\napple"""

print(re.fullmatch('bag', s))  # None

regex = re.compile('bag')
print(regex.fullmatch(s))  # None,从头开始查找匹配
print(regex.fullmatch(s, 7))  # None,从索引为7位置开始查找匹配
print(regex.fullmatch(s, 7, 10))  # <_sre.SRE_Match object; span=(7, 10), match='bag'>
```

fullmatch要完全匹配，多了少了都不行，[7, 10)

默认多行模式

### 全文搜索

#### findall

findall(pattern, string, flags=0)

regex.findall(string[, pos[, endpos]])

对整个字符串从左至右匹配，返回所有匹配项的列表.

 

```
s = """bottle\nbag\nbig\napple"""

print(re.findall('b', s))  # ['b', 'b', 'b']

regex = re.compile('^b')
print(regex.findall(s))  # ['b']  bottle中的b

regex = re.compile('^b', re.M)
print(regex.findall(s, 7))  # ['b', 'b']  bag,big中的b
print(regex.findall(s, 7, 10))  # ['b']  bag中的b

regex = re.compile('^b', re.S)
print(regex.findall(s))  # ['b']  bottle中的b
```

#### finditer

finditer(pattern, string, flags=0)

regex.finditer(string[, pos[, endpos]])

对整个字符串从左至右匹配，返回所有匹配项，返回迭代器.

注意每次迭代返回的是match对象

 

```
s = """bottle\nbag\nbig\napple"""

regex = re.compile('^b', re.M)
result = regex.finditer(s)
print(result)
print(type(result))
print(next(result))
print(next(result))
'''
<callable_iterator object at 0x0000025F256396D8>
<class 'callable_iterator'>
<_sre.SRE_Match object; span=(0, 1), match='b'>
<_sre.SRE_Match object; span=(7, 8), match='b'>
'''
```

### 匹配替换

#### sub

re.sub(pattern, replacement, string, count=0, flags=0)

regex.sub(repl, string, count=0)

使用pattern对字符串string进行匹配，对匹配项使用repl替换，replacement可以使string，bytes，function

 

```
s = """bottle\nbag\nbig\napple"""

regex = re.compile('b\wg')
print(regex.sub('magedu', s))  # 被替换后的字符串
print('-------')
print(regex.sub('magedu', s, 1))  # 替换1次
'''
bottle
magedu
magedu
apple
-------
bottle
magedu
big
apple
'''
```

regex.sub(repl, str)

repl也可以是一个函数名，类似回调函数，可接收一个参数，表示对str做模式匹配，然后将匹配到的数据交给函数repl处理。

 

```
import re
TYPE_PATTERNS = {
    'str': r'[^/]+',
    'word': r'\w+',
    'int': r'[+-]?\d+',
    'float': r'[+-]?\d+\.\d+',
    'any': r'.+'
}
def repl(matcher):
    """
    :param matcher: re.compile(pattern).search(src)的结果
    :return: string
    """
    a = matcher.group(0)
    n = matcher.group(1)
    t = matcher.group(2)
    
    return '/(?P<{}>{})'.format(n, TYPE_PATTERNS.get(t, TYPE_PATTERNS['str']))

print(2, regex.sub(repl, src))
```

sub做的事：每一次search(src)匹配到的结果都交给repl函数处理，然后返回一个被替换后的字符串。常用来做replace替换，扩展性差。

#### subn

re.subn(pattern, replacement, string, count=0, flags=0)

regex.subn(replacement, string, count=0)

同sub，返回一个元组(new_string, number_of_subs_made)

 

```
s = """bottle\nbag\nbig\napple"""

regex = re.compile('\s+')
print(regex.subn('\t', s))
'''
('bottle\tbag\tbig\tapple', 3)  #  被替换后的字符串及替换次数的元组
'''
```

### 分割字符串

字符串的分割函数，太难用，不能指定多个字符进行分割

re.split(pattern, string, maxsplit=0, flags=0)

 

```
s1 = 'a    \t* (b'
s2 = '''01 bottle
02 bag
03         big1
100     able'''

print(re.split('[\s\*\(\)]+', s1))
print(re.split('\s+\d+\s+', ' ' + s2))
```

### 分组

使用小括号的pattern捕获的数据被放到了组group中

match、search函数可以返回match对象

findall返回字符串列表

finditer返回一个个match对象

如果pattern中使用了分组，如果有匹配结果，会在match对象中

\- 使用group(N)方式返回对应分组，1-N是对应的分组，0返回整个匹配的字符串

\- 如果使用了命名分组，可以使用group('name')的方式取分组

\- 也可以使用group()返回所有组

\- 使用groupdict()返回所有命名的分组

 

```
s = """bottle\nbag\nbig\napple"""
regex = re.compile('(b\w+)')
result = regex.match(s)
print(result.group())  # bottle
result = regex.search(s)
print(result.group())  # bottle

regex = re.compile('(b\w+)\n(?P<name2>b\w+)\n(?P<name3>b\w+)')
result = regex.match(s)
print('match', result)  # match <_sre.SRE_Match object; span=(0, 14), match='bottle\nbag\nbig'>
print(result.group())  # bottle big bag
print(result.group(1))  #  bottle
print(result.group(3).encode())  # big
print(result.group('name2'))  # bag
print(result.groupdict())  # {'name2': 'bag', 'name3': 'big'}

result = regex.findall(s)
for x in result:
    print(type(x), x)  # <class 'tuple'> ('bottle', 'bag', 'big')

regex = re.compile('(?P<head>b\w+)')
result = regex.finditer(s)
for x in result:
    print(type(x), x, x.group(), x.group('head'))
```

示例

 

```
print(re.findall("<(?P<tag_name>\w+)>\w+</(?P=tag_name)>","<h1>hello</h1>")) #['h1']
print(re.search("<(?P<tag_name>\w+)>\w+</(?P=tag_name)>","<h1>hello</h1>").group()) #<h1>hello</h1>
print(re.search("<(?P<tag_name>\w+)>\w+</(?P=tag_name)>","<h1>hello</h1>").groupdict()) #<h1>hello</h1>
print(re.search(r"<(\w+)>\w+</(\w+)>","<h1>hello</h1>").group())
print(re.search(r"<(\w+)>\w+</\1>","<h1>hello</h1>").group())
'''
运行结果：
['h1']
<h1>hello</h1>
{'tag_name': 'h1'}
<h1>hello</h1>
<h1>hello</h1>
'''

# 找出所有数字['1', '-12', '60', '-40.35', '5', '-4', '3']
print(re.findall(r'-?\d+\.?\d*',"1-12*(60+(-40.35/5)-(-4*3))")) 
# 使用|，先匹配的先生效，|左边是匹配小数，而findall最终结果是查看分组，所有即使匹配成功小数也不会存入结果
# 而不是小数时，就去匹配(-?\d+)，匹配到的自然就是，非小数的数，在此处即整数

# 找出所有整数['1', '-2', '60', '', '5', '-4', '3']
print(re.findall(r"-?\d+\.\d*|(-?\d+)","1-2*(60+(-40.35/5)-(-4*3))"))
'''
运行结果：
['1', '-12', '60', '-40.35', '5', '-4', '3']
['1', '-2', '60', '', '5', '-4', '3']
'''
```

总结:

尽量精简,详细的如下

尽量使用泛匹配模式.*

尽量使用非贪婪模式:.*?

使用括号得到匹配目标:用group(n)去取得结果

有换行符就用re.S:修改模式

# 练习

1,匹配邮箱地址

test@hot-mail.com

v-ip@magedu.com

web.manager@magedu.com.cn

super.user@google.com

a@w-a-com

 

```
([\w\.]+)@([\w-\.]+\.[\w-]+)
```

2,匹配html标记内的内容

<a href='http://www.magedu.com/index.html' target='_blank'>马哥教育</a>

 

```
<[^<>]+>(.*)<[^<>]+>
```

```
<(\w+)\s+[^<>]+>(.*)(</\1>)
```

3,匹配URL

http://www.magedu.com/index.html

https://login.magedu.com

file:///ect/sysconfig/network

 

```
(\w+)://([^\s]+)
```

4,匹配二代中国身份证id

321105700101003

321105197001010030

11210020170101054X

17位数字+1位校验码组成

前6位地址码，8位出生年月，3位数字，1位校验位（0-9或X）

 

```
\d{17}[0-9xX]|\d{15}
```

5,匹配密码强弱

要求密码必须由10-15位指定字符组成

十进制数字

大写字母

小写字母

下划线

要求四种类型的字符都要出现才算合法的强密码

例如Aatb32_67mnq，其中包含大写字符、小写字母、数字和下划线，是合格的强密码

6,单词统计

对一个英文文档进行单词统计，要求使用正则表达式