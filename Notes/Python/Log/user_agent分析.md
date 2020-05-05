[TOC]

# 概述

User Agent中文名为用户代理，是Http协议中的一部分，属于头域的组成部分，User Agent也简称UA。它是一个特殊字符串头，是一种向访问网站提供你所使用的浏览器类型及版本、操作系统及版本、浏览器内核、等信息的标识。通过这个标 识，用户所访问的网站可以显示不同的排版从而为用户提供更好的体验或者进行信息统计；例如用手机访问谷歌和电脑访问是不一样的，这些是谷歌根据访问者的 UA来判断的。UA可以进行伪装。

浏览器的UA字串的标准格式：浏览器标识 (操作系统标识; 加密等级标识; 浏览器语言) 渲染引擎标识版本信息。但各个浏览器有所不同。

## 操作系统标识符

PC端

![img](user_agent%E5%88%86%E6%9E%90.assets/43bb9060-a607-42af-b162-0abc78739852.jpg)

移动端

![img](user_agent%E5%88%86%E6%9E%90.assets/feecd402-49dd-481c-9f2c-912210afc84c.jpg)

常见user-agent：

 

```
# Chrome
Win7:
Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.163 Safari/535.1
 
# Firefox
Win7:
Mozilla/5.0 (Windows NT 6.1; WOW64; rv:6.0) Gecko/20100101 Firefox/6.0
 
# Safari
Win7:
Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50
 
# Opera
Win7:
Opera/9.80 (Windows NT 6.1; U; zh-cn) Presto/2.9.168 Version/11.50
 
# IE
Win7+ie9：
Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 2.0.50727; SLCC2; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; InfoPath.3; .NET4.0C; Tablet PC 2.0; .NET4.0E)
 
Win7+ie8：
Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.1; WOW64; Trident/4.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; .NET4.0C; InfoPath.3)
 
WinXP+ie8：
Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; GTB7.0)
 
WinXP+ie7：
Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)
 
WinXP+ie6：
Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)
 
# 傲游
傲游3.1.7在Win7+ie9,高速模式:
Mozilla/5.0 (Windows; U; Windows NT 6.1; ) AppleWebKit/534.12 (KHTML, like Gecko) Maxthon/3.0 Safari/534.12
 
傲游3.1.7在Win7+ie9,IE内核兼容模式:
Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; InfoPath.3; .NET4.0C; .NET4.0E)
 
# 搜狗
搜狗3.0在Win7+ie9,IE内核兼容模式:
Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; InfoPath.3; .NET4.0C; .NET4.0E; SE 2.X MetaSr 1.0)
 
搜狗3.0在Win7+ie9,高速模式:
Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US) AppleWebKit/534.3 (KHTML, like Gecko) Chrome/6.0.472.33 Safari/534.3 SE 2.X MetaSr 1.0
 
# 360
360浏览器3.0在Win7+ie9:
Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; InfoPath.3; .NET4.0C; .NET4.0E)
 
# QQ浏览器
QQ浏览器6.9(11079)在Win7+ie9,极速模式:
Mozilla/5.0 (Windows NT 6.1) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/13.0.782.41 Safari/535.1 QQBrowser/6.9.11079.201
 
QQ浏览器6.9(11079)在Win7+ie9,IE内核兼容模式:
Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 6.1; WOW64; Trident/5.0; SLCC2; .NET CLR 2.0.50727; .NET CLR 3.5.30729; .NET CLR 3.0.30729; Media Center PC 6.0; InfoPath.3; .NET4.0C; .NET4.0E) QQBrowser/6.9.11079.201
 
# 阿云浏览器
阿云浏览器1.3.0.1724 Beta(编译日期2011-12-05)在Win7+ie9:
Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; WOW64; Trident/5.0)
```

## user-angent分析工具

安装

pip install pyyaml ua-parser user-agents

使用

browser.family返回浏览器名称

browser.version_string返回版本号

 

```
from user_agents import parse

useragents = [
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) \
    Chrome/14.0.835.163 Safari/535.1",
    "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:6.0) Gecko/20100101 Firefox/6.0",
    "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 \
    Safari/534.50",
    "Opera/9.80 (Windows NT 6.1; U; zh-cn) Presto/2.9.168 Version/11.50",
    "Mozilla/5.0 (compatible; MSIE 9.0; \
    Windows NT 6.1; Win64; x64; Trident/5.0; .NET CLR 2.0.50727; \
    SLCC2; .NET CLR 3.5.30729; .NET CLR 3.0.30729; \
    Media Center PC 6.0; InfoPath.3; .NET4.0C; Tablet PC 2.0; .NET4.0E)",
    "Mozilla/5.0 (Windows; U; Windows NT 6.1; ) AppleWebKit/534.12 (KHTML, like Gecko) \
    Maxthon/3.0 Safari/534.12",
]

for uastring in useragents:
    ua = parse(uastring)
    print(ua.browser,ua.browser.family,ua.browser.version,ua.browser.version_string)
'''
Browser(family='Chrome', version=(14, 0, 835), version_string='14.0.835') Chrome (14, 0, 835) 14.0.835

'''
```

# user_agent分析

为日志分析添加user_agent提取功能

 

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
添加状态码分析
添加浏览器分析,统计次数,并按次数排序;
"""
from queue import Queue
from pathlib import Path
from user_agents import parse
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
    'size': int,
    'request': lambda request: dict(zip(('method', 'url', 'protocol'), request.split())),
    'useragent': lambda ua: parse(ua)
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


allbrowsers = {}
def browser_handler(iterable):
    """
    浏览器类型和版本提取处理
    :param iterable:
    :return:
    """
    browsers = {}
    for item in iterable:
        ua = item['useragent']
        key = (ua.browser.family, ua.browser.version_string)
        browsers[key] = browsers.get(key, 0) + 1
        allbrowsers[key] = allbrowsers.get(key, 0) + 1
    print(sorted(allbrowsers.items(), key=lambda x: x[1], reverse=True)[:10])
    return browsers


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
    reg(browser_handler, 5, 5)
    run()

```