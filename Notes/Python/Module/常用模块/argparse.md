[TOC]

# 前言

参数分类

位置参数：就是程序根据该参数出现的位置来确定的.

例如: 

*ls -l /etc*

/etc就是对应一个位置参数

选项参数：是程序已经提前定义好的参数,不是随意指定的.

例如: 

*ls -l /etc 或 ls --list /etc*

-l或*--list* 就是ls命令的一个选项参数

# argparse模块

3.2开始，python提供了参数分析的模块argparse

argparse是python标准库里面用来处理命令行参数的库,用于命令行选项、参数和子命令的分析器.

argparse模块使编写用户友好的命令行界面变得容易。程序定义了它需要的参数，argparse将找出如何从sys.argv中解析这些参数。argparse模块还自动生成帮助和用法消息，并在用户向程序提供无效参数时发出错误。

## ArgumentParser

定义一个命令程序

ArgumentParser(prog=None, usage=None, description=None, epilog=None, parents=[], formatter_class=argparse.HelpFormatter, prefix_chars='-', fromfile_prefix_chars=None, argument_default=None, conflict_handler='error', add_help=True, allow_abbrev=True)

创建新的ArgumentParser对象。所有参数都应作为关键字参数传递。每个参数在下面都有自己更详细的描述:

Keyword Arguments:

prog - 软件的程序的名称（默认：sys.argv [ 0 ]）

usage - 描述程序用法的字符串（默认值：由添加到分析器的参数生成）

description - 要在参数help之前显示的文本(默认值:none)

epilog - 参数help(默认:none)之后显示的文本

parents - ArgumentParser对象的列表，其中也应该包含参数

formatter_class - 用于自定义帮助输出的类

prefix_chars - 前缀可选参数的字符集(默认值:' - ')

fromfile_prefix_chars - 应该从前缀文件中读取附加参数的一组字符(默认值:None)

argument_default - 参数的全局默认值(默认值:None)

conflict_handler - 解决冲突选项的策略(通常是不必要的)

add_help - 向解析器添加-h/ -help选项(默认值:True)

allow_abbrev - 允许长选项被缩写，如果缩写是明确的。(默认为True)

## add_argument

定义如何解析单个命令行参数

ArgumentParser.add_argument(name or flags...[, action][, nargs][, const][, default][, type][, choices][, required][, help][, metavar][, dest]) 

name or flags - 一个名称或一个选项字符串列表，例如foo或-f，——foo。

action - 在命令行中遇到此参数时要采取的基本操作类型。

nargs - 应该使用的命令行参数的数量。

const - 某些操作和nargs选择所需的常量。 

default - 如果参数不在命令行中，则生成的值。

type - 应将命令行参数转换为的类型。

choices - 参数的允许值的容器。

required - 是否可以省略命令行选项(仅限选项)。

help - 简单描述一下这个参数的作用。

metavar - 在提示信息中显示参数的名字

dest - 要添加到parse_args()返回的对象中的属性的名称。

## parse_args

parse_args(args=None, namespace=None)

args参数列表，一个可迭代对象。内部会把可迭代对象转换成list。如果为None则使用命令行传入参数，非None则使用args参数的可迭代对象。

 

```
# 分析参数，同时传入可迭代的参数
parser = argparse.ArgumentParser(prog='ls', add_help=True, description='list directory contents')
parser.add_argument('path')  # 定义位置参数,默认必须传入
args = parser.parse_args(('/etc',))  # 分析参数,同时传入可迭代的参数
print(args)  # Namespace(path=['/etc'])
```

Namespace(path='/etc')里面的path参数存储在了一个Namespace对象内的属性上，可以通过Namespace对象属性来访问，例如args.path

定义可选位置参数，可有可无，有缺省值，显示帮助信息

 

```
parser.add_argument('path', nargs='?', default='.', help='path help desc')
```

没有提供path就使用默认值'.'点号表示当前路径。

help：表示帮助文档中这个参数的描述

nargs：表示这个参数接收参数结果，?表示可有可无,+表示至少一个,*可以任意个,数字表示必须指定指定数目个

default：表示如果不提供该参数，就使用这个值。一般和(?,*)配合，因为它们都可以不提供位置参数，不提供就是用缺省值。

# 模拟ls功能

实现ls命令功能，实现-l、-a和--all、-h选项

实现显示路径下的文件列表

-a和-all 显示包含.开头的文件

-l 详细列表显示

-h 和-l配合，人性化显示文件大小，例如1K、1G、1T等，可以认为1G=1000M

c 字符；d 目录；- 普通文件；l 软链接；b 块设备；s socket文件；p pipe文件，即FIFO

-rw-rw-r-- 1 python python 5 Oct 25 00:07 test4

mode 硬链接属主属组字节时间文件名

按照文件名排序输出，可以和ls的顺序不一样，但要求文件名排序

要求详细列表显示时，时间可以按照“年-月-日时:分:秒” 格式显示

解决命令、传参、选项问题

ls [-l[--list]] [path [path [..]]]

 

```
import argparse

parser = argparse.ArgumentParser(prog='ls', add_help=True, description='list directory contents')
parser.add_argument('path', nargs='?', default='.', help='Directory')
parser.add_argument('-l', '--list', action='store_true', help='Use a long listing format')
parser.add_argument('-a', '--all', action='store_true', help="Show all files, do not ignore entries starting with .")

parser.print_help()
print('~~~~~~')
args = parser.parse_args('-l -a'.split())
args1 = parser.parse_args('/etc -la'.split())
print(args, args1)
```

解决业务问题

上面解决了参数的定义和传参的问题，下面考虑解决业务问题：

列出所有指定路径的文件，默认是不递归的

-a显示所有文件，包括隐藏文件

-l详细列表模式显示

 

```
def listdir(path, all=False):
    """列出本目录文件"""
    p = Path(path)  # 获取path路径的对象
    for i in p.iterdir(): # 迭代path路径下子目录
        if not all and i.name.startswith('.'):  # 不显示隐藏文件
            continue
        yield i.name  # 返回目录和文件名
```

最终代码

 

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# __Author__: xiaofei

"""
实现ls命令功能，实现-l、-a和--all、-h选项
实现显示路径下的文件列表
-a和-all 显示包含.开头的文件
-l 详细列表显示
-h 和-l配合，人性化显示文件大小，例如1K、1G、1T等，可以认为1G=1000M
c 字符；d 目录；- 普通文件；l 软链接；b 块设备；s socket文件；p pipe文件，即FIFO
-rw-rw-r-- 1 python python 5 Oct 25 00:07 test4
mode 硬链接属主属组字节时间文件名
按照文件名排序输出，可以和ls的顺序不一样，但要求文件名排序
要求详细列表显示时，时间可以按照“年-月-日时:分:秒” 格式显示
"""

import argparse
from pathlib import Path
from datetime import datetime
import stat

parser = argparse.ArgumentParser(prog='ls', add_help=False, description='List directory contents')
parser.add_argument('path', nargs='?', default='.', help='Directory')
parser.add_argument('-l', action='store_true', help='use a long listing format')
parser.add_argument('-a', '--all', action='store_true', help="Show all files, do not ignore entries starting with .")
parser.add_argument('-h', '--human-readable', action='store_true', help="with -l, print sizes in human readable format.")


def listdir(path, all=False, detail=False, human=False):

    def _gethuman(size:int):
        units = ' KMGTP'
        depth = 0
        while size >= 1000:
            size = size // 1000
            depth += 1
        return '{}{}'.format(size, units[depth])

    # 定义文件类型
    def _getfiletype(f: Path):
        if f.is_dir():
            return 'd'
        elif f.is_block_device():
            return 'b'
        elif f.is_char_device():
            return 'c'
        elif f.is_socket():
            return 's'
        elif f.is_symlink():
            return 'l'
        elif f.is_fifo():
            return 'p'
        else:
            return '-'

    # 将权限对应为rwx格式
    modelist = dict(zip(range(9), ['r', 'w', 'x', 'r', 'w', 'x', 'r', 'w', 'x']))

    def _getmodestr(mode: int):
        m = mode & 0o777  # 0oXXX表示八进制,十转八:oct(), &:位运算
        mstr = ''
        for i in range(8, -1, -1):
            if m >> i & 1:
                mstr += modelist[8 - i]
            else:
                mstr += '-'
        return mstr

    def _listdir(path, all=False, detail=False, human=False):
        """详细列出"""
        p = Path(path)
        for i in p.iterdir():
            if not all and i.name.startswith('.'):  # 不显示隐藏文件
                continue
            if not detail:
                yield (i.name)
            else:
                # 详细信息: mode 硬链接 属主 属组 字节 时间 name
                # -rw-r--r--. 1 root root 894 Feb 21 20:43 /etc/passwd
                st = i.stat()
                mode = _getfiletype(i) + _getmodestr(st.st_mode)
                atime = datetime.fromtimestamp(st.st_atime).strftime('%Y %m %d %H:%M:%S')

                yield (mode, st.st_nlink, st.st_uid, st.st_gid, st.st_size, atime, i.name)

    yield from sorted(_listdir(path,all, detail), key=lambda x: x[len(x) -1])


if __name__ == '__main__':
    args = parser.parse_args()
    print(args)
    parser.print_help()
    files = listdir(args.path, args.all, args.l, args.human_readable)
    print(list(files))
```

在linux上测试

python xxx.py -lah

python xxx.py /var/log/ -lah