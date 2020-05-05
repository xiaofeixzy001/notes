[TOC]

# 元组tuple

一个有序的元素组成的集合

使用小括号()表示

元组是不可变对象

# 定义

tuple() -> empty tuple

tuple(iterable) -> tuple initialized from iterable's items

约定:创建一个元组的时候，要在最后一个元素的后面再加上一个逗号','；

如:

```
t = tuple()  # 工厂方法
t = ()
t = tuple(range(1, 7, 2))  # iteratable
t = (2, 4, 6, 3, 4, 2)
t = (1,)  # 一个元素元组的定义，注意有个逗号
t = (1,) * 5
t = (1, 2, 3) * 6
```



# 方法

支持索引（下标）

正索引：从左至右，从0开始，为列表中每一个元素编号

负索引：从右至左，从-1开始

正负索引不可以超界，否则引发异常IndexError

元组是只读的，所以增、改、删方法都没有

## 通过索引访问

tuple[index]

index就是索引，使用中括号访问

```
list02 = ('user', '123', 'a1', '',)

# 索引
list02[0]

# 切片
list02[0:2]

# 循环
for i in list02:
    print(i)

# 长度
v = len(list02)

# 包含
if '123' in list02:
    print('ok')
else:
    print('no')

```

需要注意的是：

元组的子元素是不可修改的,但是子元素如果是列表或字典等集合类型,那么该子元素的子元素是可以被修改的;

## 元组查询

### index

index(value,[start,[stop]])

通过值value，从指定区间查找列表内的元素是否匹配

匹配第一个就立即返回索引

匹配不到，抛出异常ValueError

### count

count(value)

返回列表中匹配value的次数

时间复杂度

index和count方法都是O(n)

随着列表数据规模的增大，而效率下降

### len

len(tuple)

返回元素的个数

## 命名元组namedtuple

帮助文档中，查阅namedtuple，有使用例程

使用方法

namedtuple(typename, field_names, verbose=False, rename=False)

命名元组，返回一个元组的子类，并定义了字段;

field_names可以是空白符或逗号分割的字段的字符串，可以是字段的列表

示例：

```
from collections import namedtuple
Point = namedtuple('_Point',['x','y'])  # Point为返回的类
p = Point(11, 22)

Student = namedtuple('Student', 'name age')
tom = Student('tom', 20)
jerry = Student('jerry', 18)
tom.name
```