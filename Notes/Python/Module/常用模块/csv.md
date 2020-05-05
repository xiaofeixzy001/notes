[TOC]

# 简介

参考RFC4108

https://www.ietf.org/rfc/rfc4180.txt

逗号分隔值Comma-Separated Values

逗号分隔值（Comma-Separated Values，CSV，有时也称为字符分隔值，因为分隔字符也可以不是逗号），其文件以纯文本形式存储表格数据（数字和文本）。纯文本意味着该文件是一个字符序列，不含必须像二进制数字那样被解读的数据。CSV文件由任意数目的记录组成，记录间以某种换行符分隔；每条记录由字段组成，字段间的分隔符是其它字符或字符串，最常见的是逗号或制表符。通常，所有记录都有完全相同的字段序列。通常都是纯文本文件。建议使用WORDPAD或是记事本（NOTE）来开启，再则先另存新档后用EXCEL开启，也是方法之一

# 特征

csv是一个被行分隔符、列分隔符划分成行和列的文本文件

csv不指定字符编码

行分隔符为\r\n，最后一行可以没有换行符。

列分隔符常为逗号或制表符

每一行成为一条记录record

字段可以使用双引号括起来，也可以不使用。

如果字段中出现了双引号、逗号、换行符必须使用双引号括起来。

如果字段的值是双引号，使用两个双引号表示一个转义。

表头可选，和字段列对齐就行了。

例如：手动生成一个csv文件

 

```
from pathlib import Path
csv_body = '''\
id,name,age,comment
1,zs,18,"I'm 18"
2,ls,20,"this is a ""test"" string."
4,ww,23,"你好

计算机
"
'''
p = Path('E:/Projects/mage/study/文件IO/test.csv')

p.parent.mkdir(parents=True, exist_ok=True)
p.write_text(csv_body)
```

# csv模块

import csv

reader(scvfile, dialect='excel', **fmtparams)

返回DictReader对象，是一个行迭代器

读取数据

 

```
import csv
csvname = 'E:/Projects/mage/study/文件IO/test.csv'
with open(csvname) as f:
    reader = csv.reader(f)
    print(list(reader))
```

写入数据

 

```
import csv
rows = [
    ['id', 'name', 'age', 'comment'],
    ['1', 'zs', '18', "I'm 18"],
    ['2', 'ls', '20', 'this is a "test" string.'],
    ['4', 'ww', '23', '你好\r\n\r\n计算机\r\n'],
    ((1, 2, 3,), [2, 3, 4])
]

csvname = 'E:/Projects/mage/study/文件IO/test.csv'

row = rows[0]
p = Path(csvname)
with open(str(p), 'a', newline='', encoding='utf-8') as f:  # 如果不指定newline,默认是换行符,写入时会隔一个空行
    writer = csv.writer(f)
    writer.writerow(row)  # 单行写入
    writer.writerows(rows)  # 多行写入
```

DictReader和DictWriter对象

使用DictReader可以像操作字典那样获取数据，把表的第一行(一般是标头)作为key。可访问每一行中那个某个key对应的数据。

读数据

 

```
with open(csvname, encoding='utf-8') as f:
    reader = csv.DictReader(f)
    for row in reader:
        # print(row)  # 有序字典
        print(row['name'])  # 通过列的字段访问一列数据
```

写数据

 

```
headers = ['name', 'age']
datas = [
    {'name':'Bob', 'age':23},
    {'name':'Jerry', 'age':44},
    {'name':'Tom', 'age':15}
]
with open('test1.csv', 'w', newline='', encoding='utf-8') as f:
    # 写入表头
    writer = csv.DictWriter(f, headers)
    writer.writeheader()
    # 写入数据,注意数据的key要与表头一致
    writer.writerows(datas)  # 多行写
    for row in datas:  # 单行写
        writer.writerow(row)
```