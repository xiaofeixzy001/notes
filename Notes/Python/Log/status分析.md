[TOC]

# **数据源**

上面的示例中，数据源都是自定义的格式，实际应用中,数据源一般都来自日志文件的一行行数据,所以将获取数据的方法封装成函数，注意考虑目录问题。

 

```
# 加载文件
def openfile(path: str, encoding='utf-8'):
    with open(path, encoding=encoding) as f:
        for line in f:
            fields = extract(line)
            if fields:
                yield fields
            else:
                continue


def load(*paths):
    for item in paths:
        p = Path(item)
        if not p.exists():
            continue
        if p.is_dir():
            for file in p.iterdir():
                if file.is_file():
                    yield from openfile(str(file))
        elif p.is_file():
            yield from openfile(str(p))
```

离线日志分析项目

可以指定文件或目录，对日志进行数据分析

分析函数可以动态注册

数据可以分发给不同的分析处理程序处理

# status分析

添加状态分析处理功能

 

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# __Author__: xiaofei
"""
功能:
对日志格式的字符串做处理;
对处理后的时间字符串做处理;

扩展:
根据时间实现滑动窗口显示数据,每隔m秒显示n秒范围内的数据;
添加队列处理机制;
添加支持处理日志文件功能;
添加状态码分析;
"""
from queue import Queue
from pathlib import Path
import datetime
import re
import threading


# 提取方法
PATTERN = '''(?P<remote>[\d\.]{7,})\
\s-\s-\s\[(?P<datetime>[^\[\]]+)\]\
\s"(?P<method>\w+)\
\s(?P<url>\/[\w\/\?=\.]*)\
\s(?P<protocol>\w+\/\d\.\d)\
"\s(?P<status>\d+)\
\s(?P<size>\d+)\
\s"[^"]+"\s"(?P<useragent>[^"]+)"'''

regex = re.compile(PATTERN)

# 处理格式
ops = {
    'datetime': lambda datestr: datetime.datetime.strptime(datestr, '%d/%b/%Y:%H:%M:%S %z'),
    'status': int,
    'size': int
}


def extract(line: str) -> dict:
    """
    提取数据
    :param line:
    :return:
    """
    matcher = regex.match(line)
    if matcher:
        return {k: ops.get(k, lambda x: x)(v) for k, v in matcher.groupdict().items()}


def openfile(path: str, encoding='utf-8'):
    """
    打开文件并返回文件行
    :param path:
    :param encoding:
    :return:
    """
    with open(path, encoding=encoding) as f:
        for line in f:
            fields = extract(line)
            if fields:
                yield fields
            else:
                continue


def load(*paths):
    """
    读取数据来源,判断其是文件还是目录
    :param paths:
    :return:
    """
    for item in paths:
        p = Path(item)
        if not p.exists():
            continue
        if p.is_dir():
            for file in p.iterdir():
                if file.is_file():
                    yield from openfile(str(file))
        elif p.is_file():
            yield from openfile(str(p))


def donothing_handler(iterable):
    """
    处理函数: 什么都不做
    :param iterable:
    :return:
    """

    return iterable


def status_handler(iterable):
    """
    处理数据: 状态码占比,时间窗口内的一批数据
    :param iterable:
    :return:
    """

    status = {}
    for item in iterable:
        key = item['status']
        status[key] = status.get(key, 0) + 1
    total = len(iterable)
    # ret = {k: status[k] / total for k, v in status.items()}
    # print(ret)
    return {k: status[k] / total for k, v in status.items()}


def window(src: Queue, handler, width: int, interval: int):
    """
    时间窗口展示函数
    :param src: 数据源,缓存队列,用来拿数据
    :param handler: 处理函数
    :param width: 数据展示范围
    :param interval: 数据展示间隔
    :return:
    """
    start = datetime.datetime.strptime('20170101 000000 +0800', '%Y%m%d %H%M%S %z')
    current = datetime.datetime.strptime('20170101 010000 +0800', '%Y%m%d %H%M%S %z')
    delta = datetime.timedelta(seconds=(width - interval))

    # 待处理数据
    buffer = []

    while True:
        # 获取数据
        data = src.get()
        if data:
            buffer.append(data)
            current = data['datetime']

        # 每隔interval时间,计算buffer中数据一次
        if (current - start).total_seconds() >= interval:
            ret = handler(buffer)
            print('{}'.format(ret))
            start = current

            # 清除超出width部分数据
            buffer = [x for x in buffer if x['datetime'] > current - delta]


def dispatch(src):
    """
    分发器
    :param src:
    :return:
    """
    # 记录handler,同时保存各自队列
    handlers = []
    queues = []

    def reg(handler, width: int, interval: int):
        q = Queue()
        queues.append(q)

        h = threading.Thread(target=window, args=(q, handler, width, interval))
        handlers.append(h)

    def run():
        for h in handlers:
            h.start()

        for item in src:
            for q in queues:
                q.put(item)

    return reg, run


# 执行
if __name__ == '__main__':
    path = 'test.log'
    reg, run = dispatch(load(path))
    reg(status_handler, 5, 5)
    run()
```