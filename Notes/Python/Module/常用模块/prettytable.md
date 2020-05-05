[TOC]

PrettyTable

PrettyTable是python中的一个第三方库,可用来生成美观的ASCII格式的表格,十分实用.

默认python库不会自带,需要额外安装.

 

```
pip install prettytable
```

自行创建示例:

 

```
from prettytable import PrettyTable
t = PrettyTable(['A','B','C']) # 这是表头,也就是每列的标题
t.align='l'  # 设置列样式,l左对齐,r右对齐,c中间对齐
t.border=False  # 不显示边框,可省略,默认显示

# 下面是添加数据,也就是每行数据,列表形式,前面设置了几列,这里要给几个列表元素
t.add_row(['111','222','333'])
t.add_rot(['444','555','666'])
t.add_rot(['777','888',''])
print(t)
```

效果:

+-----+-----+-----+

| A  | B  | C  |

+-----+-----+-----+

| 111 | 222 | 333 |

| 444 | 555 | 666 |

| 777 | 888 |   |

+-----+-----+-----+

表属性介绍:

border - 布尔类型参数（必须是True或False）。控制表格边框是否显示。

header - 布尔类型参数（必须是True或False）。控制表格第一行是否作为表头显示。

header-style - 控制表头信息的大小写。允许的参数值：“cap”（每个单词首字母大写），“title”（除了介词助词首字母大写），“lower”（全部小写）或者None（不改变原内容格式）。默认参数为None。

hrules - 设置表格内部水平边线。允许的参数值：FRAME，ALL，NONE。注意这些是在prettytable模块内部定义的变量，在使用之前导入或用类似prettytable.FRAME的方法调用。

vrules - 设置表格内部竖直边线。允许的参数值：FRAME，ALL，NONE。

align - 水平对齐方式（None，“l”（左对齐），“c”（居中），“r”右对齐）

valign - 垂直对齐方式（None，“t”（顶部对齐），“m”（居中），“b”底部对齐）

int_format - 控制整型数据的格式。

float_format - 控制浮点型数据的格式。

padding_width - 列数据左右的空格数量。（当左右padding未设置时生效）

left_padding_width - 列数据左侧的空格数量。

right_padding_width - 列数据右侧的空格数量。

vertical_char - 绘制竖直边线的字符，默认为“|”

horizontal_char - 绘制水平边线的字符，默认为“-”

junction_char - 绘制水平竖直交汇点的字符，默认为“+

用法:

x = PrettyTable()

x.border = False

x.header = False

x.padding_width = 5

x = PrettyTable(["A",'B','C'],border=False, header=False, padding_width=5)

CSV格式的文件直接创建

CSV文件的格式: 以逗号,分割,如上述表格,以csv格式表示:

A,B,C

111,222,333

444,555,666

777,888,''

下面来个栗子:

csv文件内容:

D,E,F

aaa,bbb,ccc

ddd,eee,fff

hhh,iii,''

jjj,'',kkk

 

```
from prettytable import from_csv
with open('csv_test', 'r') as fp:
    pt = from_csv(fp)
    print(pt)
```

+-----+-----+-----+

| D | E | F |

+-----+-----+-----+

| aaa | bbb | ccc |

| ddd | eee | fff |

| hhh | iii |   |

| jjj |   | kkk |

+-----+-----+-----+

HTML格式直接创建

from prettytable import from_html

SQL创建

from prettytable import from_db_cursor