[TOC]

# 序列化和反序列化

要设计一套协议，按照某种规则，把内存中数据保存到文件中。

文件是一个字节序列，所以必须把数据转换成字节序列，这就是序列化，反之，从文件的字节序列恢复到内存，就是反序列化。

## 定义

序列化(Serialization)是将对象的状态信息转换为可以存储或传输的形式的过程。在序列化期间，对象将其当前状态写入到临时或持久性存储区。以后，可以通过从存储区中读取或反序列化对象的状态，重新创建该对象。

序列化：将内存中对象存储下来，把它变成一个个字节，二进制。

反序列化：将文件的一个个字节恢复成内存中对象，二进制。

序列化保存到文件就是持久化。

可以将数据序列化后再做持久化，或者网络传输，也可以将从文件中或网络接收过来的字节序列做反序列化。

序列化就是把实体对象状态按照一定的格式写入到有序字节流，反序列化就是从有序字节流重建对象，恢复对象状态。

python提供了pickle库

# pickle

python中的序列化、反序列化模块

Pickle的问题和所有其他编程语言特有的序列化问题一样,就是它只能用于Python,并且可能不同版本的Python彼此都不兼容,因此,只能用Pickle保存那些不重要的数据,不能成功地反序列化也没关系.

pickle是将内存中的结构化的数据转换成bytes类型,然后用于存储或传输.

由于pickle转换成bytes格式,所以如果写入文件,需要以wb方式,读取文件以rb方式.

dumps 对象序列化为bytes对象

dump 对象序列化到文件对象，就是存入文件

loads 从bytes对象反序列化

load 对象反序列化，从文件读取数据

示例:

 

```
import pickle

filename = 'ser'
d = {'a': 1, 'b': 'abc', 'c': [1, 2, 3]}
l = list('123')
i = 99

with open(filename, 'wb') as f:
    pickle.dump(d, f)
    pickle.dump(l, f)
    pickle.dump(i, f)

with open(filename, 'rb') as f:
    print(f.read(), f.seek(0))
    for _ in range(3):
        x = pickle.load(f)
        print(type(x), x)

class AA:
    tttt = 'ABC'
    def __init__(self):
        self.dddd = '123'
    def show(self):
        print('abc')

a1 = AA()
sr = pickle.dumps(a1)
print(a1.tttt)
print('sr = {}'.format(sr))

a2 = pickle.loads(sr)
print(a2.tttt)
print(a2.dddd)
a2.show()

with open(filename, 'wb') as f:
    pickle.dump(a1, f)

# 将filename文件放到另一台电脑上
import pickle
with open('ser', 'rb') as f:
    a = pickle.load(f)
```

如果序列化一个类，然后将其放到另一台电脑上，反序列化的时候需要找到AA这个类，否则会抛异常。

如果再新机器上定义了一个同名的类，那么反序列化的时候，就会执行当前的类，因此，序列化和反序列化必须保证使用同一套类的定义，否则会带来不可预料的结果。

可以这样理解，类是模子，二进制序列就是铁水。

一般来说，本地序列化的情况应用较少。大多数场景都应用在网络传输中。

将数据序列化后通过网络传输到远程节点，远程服务器上的服务将接收到的数据反序列化后，就可以使用了。

但是要注意，远程接收端，反序列化时必须保证有对应的数据类型，否则就会报错。尤其是自定义类，必须保证远程端有一致的定义。

现在大多数项目都不是单机的，也不是单服务的，需要通过网络将数据传送到其他节点上去，这就需要大量的序列化反序列化的过程。而pickle仅适用于python平台，如果是跨平台跨语言跨协议就不合适了，就需要公共的协议，比如XML, Json, Protocol Buffer等，不同的协议，效率不同，学习曲线不同，适用于不同场景，具体问题具体分析。

# **json**

JavaScript Object Notation, JS对象标记

https://www.json.org/

是一种轻量级的数据交换格式。基于ECMAScript(w3c制定的JS规范)的一个子集，采用完全独立于编程语言的文本格式来存储和表示数据。JSON表示出来就是一个字符串,可以被所有语言读取,也可以方便地存储到磁盘或者通过网络传输.JSON不仅是标准格式,并且比XML更快,而且可以直接在Web页面中读取,非常方便.

## 数据类型

值

JSON的值必须是双引号引起来的字符串、数值、true、false、null、对象、数组等。

字符串

由双引号包围起来的任意字符的组合，可以由转义字符。

数值

有正负，有整数，有浮点数

对象

无序的键值对的集合

格式：{key1:val1,...,keyn:valn}

key必须是一个双引号的字符串，val可以是任意合法的值。

数组

有序的值的集合

格式：[val1, ..., valn]

对应python数据类型

python支持少量内建数据类型到Json类型的转换。

| Python类型 | Json类型 |
| ---------- | -------- |
| True       | true     |
| False      | false    |
| None       | null     |
| str        | string   |
| int        | integer  |
| float      | float    |
| list       | array    |
| dict       | object   |

常用方法

dumps: json编码

dump: json编码并存入文件

loads: json解码

load: json解码，从文件读取数据

dumps(参数)支持的参数

sort_keys: 对dict对象进行排序

separators: 压缩,移除多余空白

indent n: 缩进n个空格

skipkeys: 默认False,dumps方法存储dict对象时,key必须是str类型,如果出现了其他类型的话,那么会产生TypeError异常,如果开启该参数,设为True的话,则会比较优雅的过度.

示例：

 

```
import json

d = dict(zip('abcde', [None, True, False, [1, 'abc'], {'a': 1, 'b': 2}]))

s1 = json.dumps(d)
print(s1, type(s1))

s2 = json.loads(s1)
print(s2, type(s2))

with open('a.json', 'w') as f:
    json.dump(d, f)

with open('a.json', 'r') as f:
    s3 = json.load(f)
    print(s3)
```

注意:dumps对应loads,dump对应load

一般json编码的数据很少落地，数据都是通过网络传输。传输的时候，要考虑压缩它。

本质上上来说它就是个文本，就是个字符串。

json很简单，几乎语言编程都支持json，所以应用范围十分广泛。

json不能序列化函数,pickle可以

# MessagePack

MessagePack是一个基于二进制高效的对象序列化类库，可用于跨语言通信。

它可以像json那样，在许多种语言之间交换解构对象，但它比json更快速也更轻巧。

支持Python，Ruby，Java，C/C++等众多语言，宣称比Google Protocol Buffers还要快4倍，兼容json和pickle。

![img](json%20&%20pickle%20&%20messagepack.assets/86a6955b-4080-4635-a84e-73c90bc9caa5.png)

![img](json%20&%20pickle%20&%20messagepack.assets/00de986e-17f2-4cdf-a905-899b4faf6454.png)

## 安装

pip install msgpack-python

## 常用方法

packb：序列化对象，提供了dumps来兼容pickle和json

unpackb：反序列化对象，提供了loads来兼容。

pack序列化对象保存到文件对象，提供了dump来兼容。

unpack反序列化对象保存到文件对象，提供了load来兼容。

示例

 

```
import msgpack
import json

d = dict(zip('abcde', [None, True, False, [1, 'abc'], {'a': 1, 'b': 2}]))

s1 = json.dumps(d)
print(s1)
print(len(s1))  # 74
print(type(s1))

b1 = msgpack.dumps(d)
print(b1)
print(len(b1))  # 27
print(type(b1))

b2 = msgpack.loads(b1)
print(b2)
print(type(b2))
# {b'a': None, b'b': True, b'c': False, b'd': [1, b'abc'], b'e': {b'a': 1, b'b': 2}}
```

小技巧

 

```
#from pickle import dumps, loads
#from json import dumps, loads
from msgpack import dumps, loads

# 还可以定义一个函数,根据传参的不同调用不同的序列化模块
```