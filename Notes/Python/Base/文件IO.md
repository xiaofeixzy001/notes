[TOC]

# 文件IO常用操作

文件处理流程：

打开 --> 获取文件内容 --> 关闭

open  打开

read  读取

write  写入

close  关闭

readline  行读取

readlines 多行读取

seek  文件指针操作

tell  指针位置

## open

打印文件，返回文件对象(流对象)和文件描述符。打开文件失败则返回异常。

语法：

open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)

参数说明：

file：打开或要创建的文件名，如果不指定路径，默认当前路径

mode：文件打开模式，默认只读'r'

buffering：可取值有-1,0,1,n(n>1). -1表示使用默认大小，0代表关闭(只适用于二进制模式)，1代表line buffer（只适用于文本模式），n表示设置的buffer大小；

encoding：表示的是返回的数据采用何种编码，一般采用utf8或者gbk；

errors：errors=[None|strict|ignore]，None和strict表示有编码错误将抛出ValueError异常，ignore表示忽略。

newline：newline=[None,'','\r','\n','\r\n']，文本模式中换行的转换符。

读：

None表示'\r','\n','\r\n'都被转换为'\n'；

''空表示不会自动转换通用换行符;

其他合法字符表示换行符就是指定字符，就会按照指定字符分行。

写：

None表示'\n'都会被替换为系统缺省分隔符(os.linesep)；

'\n'或''空 表示\n不替换;

其他合法字符表示'\n'会被替换为指定的字符

closefd：关闭文件描述符，True表示关闭它。False会在文件关闭后保持这个描述符。查看描述符：fileobj.fileno()

打开文件时，需要指定文件路径和以何等方式打开文件，打开后，即可获取该文件句柄，日后通过此文件对象对该文件操作。

打开文件的模式mode有：

r ，只读打开文件[默认模式，文件必须存在，不存在则抛出异常]

w，写方式打开[不可读；文件不存在则创建；存在则清空内容]

x， 写模式[不可读；文件不存在则创建，存在则报错]

a， 追加模式[可读; 文件存在则以写模式打开，并追加内容，不存在则先创建在以写模式打开，追加内容；]

总结：

r是只读，wxa都是只写，文件不存在报错

wxa都可以产生新文件，w不管文件存在与否，都会生成全新内容的文件

a不管文件是否存在，都能在打开的文件尾部追加

x必须要求文件事先不存在，自己造一个新文件

"+" 为r,w,a,x提供了缺失的读写功能，但是获取文件对象依旧按照各自特有的方式，'+'不能单独使用，可以认为它是为前面的模式做增强功能的。

r+， 读写[可读，可写，文件必须存在，不存在则抛出异常]

w+，写读[可写，可读，文件不存在则创建；存在则清空内容]

x+ ，写读[可读，可写，文件不存在则创建，存在则报错]

a+， 写读[可读，可写，文件存在则以写模式打开，并追加内容，不存在则先创建在以写模式打开，追加内容]

"b"表示以字节的方式操作，需要decode解码，encode编码，二进制可编辑图片，音频等等

rb  或 r+b：以bytes格式读取文件，读取到的内容以bytes格式显示

wb 或 w+b

xb 或 w+b

ab 或 a+b

ps.以b方式打开时，读取到的内容是字节类型，写入时也需要提供字节类型，不能指定编码

示例：

```
f = open('path/to/path') # 可以相对路径或绝对路径
print(f) # 打印获取到的文件句柄信息，而非文件内容
data = f.read() # 根据文件句柄信息，读取内容，将读取到的内容赋值给变量data
print(data) # 打印变量就相当于打印文件内容
f.close() # 关闭


# a.txt内容为‘你好’
with open('a.txt', 'rb') as f:
    print(f.read())
"""
# 运行结果:
b'\xe4\xbd\xa0\xe6\x98\xaf\xe8\xb0\x81'
"""

# 转码
with open('a.txt', 'rb') as f:
    print(f.read().decode('utf-8'))
"""
运行结果:
你好
"""


# 以bytes格式写入内容到文件，写入的内容是二进制bytes格式,如写入内容为二进制格式的‘你好吗’到c.txt中
with open('c.txt', 'wb') as f:
    f.write('你好吗'.encode('utf-8'))
# 查看c.txt文件内容为‘你好吗’
```

encoding：编码，仅文本模式使用。

open打开文件，如果不指定打开的编码格式，则会使用所在平台的编码格式为默认打开格式

None表示使用缺省编码，依赖操作系统。windows、linux下测试如下：

```
f = open('test1.txt', 'w')
f.write('啊')
f.close()
```

windows下默认GBK(0xB0A1), linux下缺省UTF-8(0xE5 95 8A)

## read

read(size=-1)

size表示读取多少个字符或字节。

负数或者None表示读取到EOF

## readline

readline(size=-1)

单行读取,一行行读取文件内容，包括 "\n" 字符，读完后光标不会复位，停在行尾。

size设置一次能读取行内几个字符或字节

## readlines

readlines(hint=-1)

多行读取,读取所有行并返回列表，指定hint则返回指定的行数。

```
f = open('test1.txt')
for line in f:
    print(line)
f.close()
```

## write

write(str)

把字符串写入到文件中并返回字符的个数。

## writelines

writelines(lines)

将字符串列表写入文件

```
f = open('test1.txt', 'w+')
lines = ['abc', '123\n', 'magedu']
f.writelines(lines)
f.seek(0)
print(f.read())
f.close()
```

## seek

文件指针操作。文件指针，指向当前字节位置，mode=r,指针起始在0，mode=a，指针起始在EOF。

格式：

seek(offset[, whence])

指定光标移动到第n个字节位置(以开头为标准，后移n个字节)，whence从哪里开始。

将文件打操作标记移到offset的位置。这个offset一般是相对于文件的开头来计算的，一般为正数。但如果提供了whence参数就不一定了，whence可以为0表示从头开始计算，1表示以当前位置为原点计算。2表示以文件末尾为原点进行计算。需要注意，如果文件以a或a+的模式打开，每次进行写操作时，文件操作标记会自动返回到文件末尾。

文本模式下：

whence 0 缺省值，表示从头开始，offset只能正整数

whence 1 表示从当前位置，offset只接受0

whence 2 表示从末尾EOF开始，offset只接受0

```
f = open('test1.txt', 'w+')
f.write("hello,world.")
print(f.tell())
print(f.read())
print(f.tell())
print('*****')
print(f.tell())
print(f.read())
print('#####')
f.seek(0)
print(f.tell())
print(f.read())
f.close()
```

二进制模式下：

whence 0 缺省值，表示从头开始，offset只能正整数

whence 1 表示从当前位置，offset可正可负

whence 2 表示从末尾EOF开始，offset可正可负

二进制模式支持任意起点的偏移，头尾中间等位置开始。

向后seek可以超界，但是向前seek的时候，不能超界，否则抛异常。

```
f = open('test1.txt', 'rb+')
print(f.tell())  # 起始位置
print(f.read())
print(f.tell())  # 末尾EOF

f.write(b'abc')
f.seek(0)  # 起始

f.seek(2, 1) # 从当前指针开始向后2个字节
print(f.read())

f.seek(-2, 1)  # 从当前指针开始向前2个字节
print(f.read())

f.seek(2, 2)  # 从EOF开始向后2, 超界无异常
print(f.read())

f.seek(0)  # 回到起始
f.seek(-20, 2)  # 从最后向前20,超界报错
f.close()
```

二进制模式支持任意起点的偏移，从头从尾从中间都可以。向后seek偏移可以超界，但向前seek的时候，不能超界，否则报错。

## tell

显示指针当前位置

file.tell()

返回文件file中当前指针(光标)位置，以文件的开头为原点。

## close

flush并关闭文件对象。

关闭后文件不能再进行读写操作。python会在一个文件不用后自动关闭文件，不过这一功能没有保证，最好还是养成自己关闭的习惯。如果一个文件在关闭后还对其进行操作会产生ValueError。

## 其他

seekable()是否可seek

readable()是否可读

writable()是否可写

closed()是否已关闭

# 上下文管理

一种特殊的语法，交给解释器去是释放文件对象。

## with..as..

自动打开文件并自动关闭

上下文管理的语句块并不会开启新的作用域

with语句块执行完的时候，会自动关闭文件对象

```
# 方式1
with open('a.txt', 'r', encoding='utf-8') as f1, open('b.txt', 'r', encoding='utf-8') as f2:
    print(f1.read())
    print(f2.read())

# 方式2
f3 = open('c.txt')
with f3:
    f3.write('abc')
```

对于类似于文件对象的IO对象，一般来说都需要在不使用的时候关闭、注销，以释放资源。IO被打开的时候，会获得一个文件描述。计算机资源是有限的，所以操作系统都会做限制，就是为了保护计算机的资源不要被完全耗尽，计算机资源是共享的，不是独占的，一般情况下，除非特别明确知道资源情况，否则不要提高资源的限制值来解决问题。

## 应用示例

模仿linux下tail实时监控一个文件内容更新并显示：

要求：监控一个文件access.log的内容，如果有新内容，则提示出来：

```
import time
with open('access.log', 'r', encoding='utf-8') as f:
    f.seek(0, 2)
    while True:
        line = f.readline().strip()
        if line:
            print('新增：', line)
        time.sleep(1)
```

# 缓冲区

当将文件内容写入到硬件设备时，使用系统调用，这类IO操作时间长，为了减小IO通常使用缓冲区，文件缓冲行为分为：全缓冲，行缓冲，无缓冲。

关键参数：buffering = N

-1表示使用缺省大小的buffer，如果是二进制模式，使用io.DEFAULT_BUFFER_SIZE值，默认是4096或8129.如果是文本模式，如果是终端设备，是行缓存方式，如果不是，则使用二进制模式的策略。

0：只在二进制模式使用，表示管buffer

1：只在文本模式使用，表示使用行缓冲。意思就是见到换行符就flush

大于1用于指定buffer的大小

buffer缓冲区

缓冲区,一个内存空间，一般来说是一个FIFO队列，当缓冲区满了或者达到阈值，数据才会flush到磁盘。也就是说写入的内容如果小于buffer值，则会先存在内存空间中，如果超过buffer值，则会将缓冲区内的数据写入到磁盘上。

flush()将缓冲区数据写入磁盘

close()关闭前会调用flush()

```
import io

f = open('test1.txt', 'w+b')
print(io.DEFAULT_BUFFER_SIZE)
f.write("mageud.com".encode())


# 设置定长缓冲区
with open('test.text', 'w+', encoding='utf-8', buffering=20) as f:
    f.write('hello word!')
    f.write('定个小目标，挣它一个亿')
    f.write('are you ok')

# 设置行缓冲
with open('test_1.text', 'w+', encoding='utf-8', buffering=1) as f:
    f.write('hello word!\n')
    f.write('定个小目标，挣它一个亿\n')
    f.write('are you ok\n')

#设置无缓冲
# 注意，text文件类型必须要写缓冲区
with open('test_2.text', 'wb+', buffering=0) as f:
    f.write(b'hello word!\n')
    f.write(b'are you ok')
```

总结：

文本模式：一般都用默认缓冲区大小；

二进制模式：是一个个字节的操作，可以指定buffer的大小；

一般来说，默认缓冲区大小是个比较好的选择，除非明确知道，否则不调整它；

一般编程中，明确知道需要写磁盘了，都会手动调用一次flush，而不是等到自动flush或者close的时候。

# 类文件对象

## StringIO

io模块中的类

from io import StringIO

内存中，开辟的一个文本模式的buffer，可以像文件对象一样操作它

当close方法被调用的时候，这个buffer会被释放

方法：

getvalue() 获取全部内容。跟文件指针没有关系

```
from io import StringIO
# 内存中构建
sio = StringIO() # 像文件对象一样操作
print(sio.readable(), sio.writable(), sio.seekable())
sio.write("magedu\nPython")
sio.seek(0)
print(sio.readline())
print(sio.getvalue()) # 无视指针，输出全部内容
sio.close()
```

好处

一般来说，磁盘的操作比内存的操作要慢得多，内存足够的情况下，一般的优化思路是少落地，减少

磁盘IO的过程，可以大大提高程序的运行效率

## BytesIO

io模块中的类

from io import BytesIO

内存中，开辟的一个二进制模式的buffer，可以像文件对象一样操作它

当close方法被调用的时候，这个buffer会被释放

```
from io import BytesIO  # 内存中构建
bio = BytesIO()
print(bio.readable(), bio.writable(), bio.seekable())
bio.write(b"magedu\nPython")
bio.seek(0)
print(bio.readline())
print(bio.getvalue())  # 无视指针，输出全部内容
bio.close()
```

## file-like

类文件对象，可以像文件对象一样操作

socket对象、输入输出对象（stdin、stdout）都是类文件对象

```
from sys import stdout
f = stdout
print(type(f))
f.write('magedu.com')
```

# 练习

有一个英文文件，对其进行单词统计，不区分大小写，并显示单词重复最多的10个单词

```
chars = set(r"""!'"#./\()[],*-`""")
d = {}

# 思路1
def makekey(k: str):
    key = k.lower()
    ret = []
    for i, c in enumerate(key):
        if c in chars:
            ret.append(' ')
        else:
            ret.append(c)
    return ''.join(ret).split()  # []


# 思路2
def makekey1(k: str):
    key = k.lower()
    ret = []
    start = 0
    length = len(key)

    for i, c in enumerate(key):
        if c in chars:
            if start == i:  # 如果2个特殊字符相临,start一定等于i
                start = i + 1
                continue
            ret.append(key[start:i])
            start = i + 1  # 因为i是特殊字符c的索引,不需要,所以跳过
    else:
        if start < len(key):  # 小于说明还有有效的字符,且一直到末尾
            ret.append(key[start:])

    return ret  # []


# print(makekey1('os.path.exists(path)'))
# print(makekey1('os.path.-exists(path)'))
# print(makekey1('path.os...'))
# print(makekey1('path'))
# print(makekey1('path-p'))
# print(makekey1('***...'))
# print(makekey1(''))


def search_word(file, encode):

    with open(file, encoding=encode) as f:
        for line in f:
            words = line.split()
            for wordlist in map(makekey1, words):  # TODO 复习map函数
                for word in wordlist:
                    d[word] = d.get(word, 0) + 1

    for k, v in sorted(d.items(), key=lambda item: item[1], reverse=True):
        print(k, v)

        
search_word('sample.txt', 'utf-8')
```