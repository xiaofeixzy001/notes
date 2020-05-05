[TOC]

# pathlib模块

python3.4版本后，提供Path对象来操作目录和文件

## 目录操作

初始化

 

```
from pathlib import Path

p = Path()  # 默认当前路径
p = Path('/etc')  # 根下etc目录
p = Path('a','b','c/d')  # 当前目录下的a\b\c\d
p.absolute()  # 当前目录绝对路径
```

### 路径拼接

拼接操作符：/

Path对象 / Path对象

Path对象 / 字符串

字符串 / Path对象

joinpath(*other) 连接多个字符串到Path对象中

### 路径分解

parts属性,返回路径中每个部分

### 示例

 

```
p = Path()
p = p / "a"  # a
p1 = 'b' / p  # b
p2 = Path('c')  # c
p3 = p2 / p1  # c\b
print(p3.parts)  # ('c', 'b')
p3.joinpath('etc', 'init.d', Path('httpd'))  # c\b\etc\init.d\httpd
```

### 获取路径

str 获取路径字符串

bytes 获取路径字符串的bytes

 

```
p = Path('/etc')
print(str(p))  # \etc
print(bytes(p))  # b'\\etc'
```

### 父目录

parent 目录的逻辑父目录

parents 父目录序列，索引0是当前位置的父目录

 

```
p = Path('/a/b/c/d')  # 当前目录定位到d
print(p.parent.parent)  # \a\b
print(p.parents[0])  # \a\b\c
for x in p.parents:
    print(x)
'''
\a\b\c
\a\b
\a
\
'''
```

name 目录的最后一部分

suffix 目录中最后一个部分的扩展名

stem 目录最后一个部分，没有后缀

suffixes 返回多个扩展名列表

with_suffix(suffix) 补充扩展名到路径尾部，返回新的路径，扩展名存在则无效

with_name(name) 替换目录最后一个部分并返回一个新的路径

 

```
p = Path('/magedu/mysqlinstall/mysql.tar.gz')
print(p.name)  # mysql.tar.gz
print(p.suffix)  # .gz
print(p.suffixes)  # ['.tar', '.gz']
print(p.stem)  # mysql.tar
print(p.with_name('mysql-5.tgz'))  # /magedu/mysqlinstall/mysql-5.tgz
print(p.with_suffix('.txt'))  # /magedu/mysqlinstall/mysql.tar.txt
```

### 目录判断

cwd() 返回当前工作目录

home() 返回当前家目录

is_dir() 是否是目录,目录存在返回True

is_file() 是否是普通文件,文件存在返回True

is_symlink() 是否是软链接

is_socket() 是否是socket文件

is_block_device() 是否是块设备

is_char_device() 是否是字符设备

is_absolute() 是否是绝对路径

resolve() 返回一个新的路径，这个新路径就是当前Path对象的绝对路径，如果是软链接则直接被解析

absolute() 也可以获取绝对路径，但是推荐resolve()

exists() 目录或文件是否存在

rmdir() 删除空目录，没有提供判断目录为空的方法

touch(mode=0o666, exist_ok=True) 创建一个文件

as_uri() 将路径返回成URI，例如：'[file:///etc/passwd'](http://file:///etc/passwd') 

mkdir(mode=0o777, parents=False, exist_ok=False)

parents,是否创建父目录,True等同于mkdir -p；False时父目录不存在则抛出异常FileNotFoundError

exist_ok，在3.5版本加入，False时路径存在则抛异常FileExistsError，True时则忽略异常

iterdir() 迭代当前目录

 

```
p = Path()
p /= 'a/b/c/d'
print(p.exists())  # False
p.mkdir()  # a/b/c/目录不存在报错
p.mkdir(parents=True, exist_ok=True)  # mkdir -p
p /= 'readme.txt'  # a\b\c\d\readme.txt
p.parent.rmdir() # 报错,找不到指定文件
print(p.parent.exists())  # False
p.mkdir(parents=True)
p.parent.rmdir()  # 报错,提示目录不是空的
```

示例：

遍历一个目录并判断目录内的文件类型,如果是子目录是否可以判断其是否为空

 

```
for x in p.parents[len(p.parents)-1].iterdir():
    print(x, end='\t')
    if x.is_dir():
        flag = False
        for _ in x.iterdir():
            flag = True
            break
        print('dir', 'Not Empty' if flag else 'Empty', sep='\t')
    elif x.is_file():
        print('file')
    else:
        print('other file')
```

### 通配符

glob(pattern) 通配给定的模式

rglob(pattern) 通配给定的模式,递归目录

返回一个生成器

 

```
p = Path()
print(list(p.glob('test*')))  # 返回当前目录对象下的test开头的文件
print(list(p.glob('**/*.py')))  # 递归所有目录,等同rglob
g = p.rglob('*.py')  # 生成器
print(next(g))
print(next(g))
```

### 匹配

match(pattern)

模式匹配，成功返回True

 

```
Path('a/b.py').match('*.py')
Path('a/b/c.py').match('b/*.py')
Path('a/b/c.py').match('a/*.py')
Path('a/b/c.py').match('a/*/*.py')
Path('a/b/c.py').match('a/**/*.py')
Path('a/b/c.py').match('**/*.py')
```

### stat

stat()相当于stat命令

lstat()同stat(),但如果是符号链接，则显示符号链接本身的文件信息

## 文件操作

open(mode='r', buffering=-1, encoding=None, errors=None, newline=None)

使用方法类似内建函数open，返回一个文件对象

3.5增加的新函数

read_bytes() 以'rb'读取路径对应文件，并返回二进制流

read_text(encoding=None, errors=None) 以'rt'方式读取路径对应文件，返回文本

Path.write_bytes(data)  以'wb'方式写入数据到路径对应文件

write_text(encoding=None, errors=None) 以'wt'方式写入字符串到路径对应文件

 

```
p = Path('my_binary_file')
p.write_bytes(b'Binary file contents')
print(p.read_bytes())

p = Path('my_text_file')
p.write_text('Text file contents')
print(p.read_text())

p = Path('text.py')
p.write_text('hello python')
print(p.read_text())

with p.open() as f:
    print(f.read(5))
```