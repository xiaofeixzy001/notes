[TOC]

# 定义

key-value键值对的数据集合

字典使用{}创建，是可变的、无序的、key不重复

存储的元素是键值对，即{'key':'value'}格式，每个键值对之间使用逗号分割

## 定义方式

1，d = dict() 或者d = {}

2，dict(**kwargs)，使用name=value对初始化一个字典

3，dict(iterable, **kwarg)，使用可迭代对象和name=value对构造字典，不过可迭代对象的元素必须是一个二元结构

4，dict(mapping, **kwarg)，使用一个字典构建另一个字典

5，类方法dict.fromkeys(iterable, value)

字典的创建示例：

```
d = {"a": 1, "b": 2}
d1 = dict(a=1, b=2)

d2 = dict(((1,'a'),(2,'b')))
d3 = dict(([1,'a'],[2,'b']))

d4 = {'a':10, 'b':20, 'c':None, 'd':[1,2,3]}

d5 = dict.fromkeys(range(5))  # {0: None, 1: None, 2: None, 3: None, 4: None}
d6 = dict.fromkeys(range(5),[0])  # {0: [0], 1: [0], 2: [0], 3: [0], 4: [0]}
```



## fromkeys()

根据指定的key名，来生成一个拥有统一值的字典

```
dic = dict.fromkeys(['k1', 'k2', 'k3'], 1)
print(dic)

dic['k1'] = 'aaaa'
print(dic)

# 运行结果:
{'k2': 1, 'k1': 1, 'k3': 1}
{'k2': 1, 'k1': 'aaaa', 'k3': 1}
```

注意：如果dict.fromkeys('key', val)中的val是一个可变的值，比如是个列表或数字等，实质上是key对应到了val的内存空间地址上，如果修改val的值，那么所有的key对应的值都会被更改

```
dic = dict.fromkeys(['k1', 'k2', 'k3'], [1])
print(dic)

dic['k1'].append(222)
print(dic)

print(id(dic['k1']))
print(id(dic['k2']))
print(id(dic['k3']))

"""
{'k1': [1], 'k2': [1], 'k3': [1]}
{'k1': [1, 222], 'k2': [1, 222], 'k3': [1, 222]}
1828616729544
1828616729544
1828616729544
"""
```



## 字典的key

key的要求和set的元素要求一致

set的元素可以就是看做key，set可以看做dict的简化版

hashable 可哈希才可以作为key，可以使用hash()测试

```
d = {1: 0, 2.0: 3, "abc": None, ('hello', 'world', 'python'): "string", b'abc': '135'}
```



## defaultdict

用于构造一个较复杂的字典结构

collections.defaultdict([default_factory[, ...]])

第一个参数是default_factory，缺省是None，它提供一个初始化函数。当key不存在的时候，会调用这个工厂函数来生成key对应的value

```
import random
d1 = {}
for k in 'abcdef':
    for i in range(random.randint(1,5)):
        if k not in d1.keys():
            d1[k] = []
        d1[k].append(i)
print(d1)
# {'a': [0], 'b': [0, 1], 'c': [0], 'd': [0, 1, 2, 3, 4], 'e': [0, 1, 2, 3, 4], 'f': [0, 1, 2, 3]}

from collections import defaultdict
import random
d1 = defaultdict(list)
for k in 'abcdef':
    for i in range(random.randint(1,5)):
        d1[k].append(i)
print(d1)
# defaultdict(<class 'list'>, {'a': [0], 'b': [0], 'c': [0, 1, 2, 3, 4], 'd': [0, 1], 'e': [0, 1], 'f': [0, 1, 2, 3]})
```

## OrderedDict有序字典

collections.OrderedDict([items])

key并不是按照加入的顺序排列，可以使用OrderedDict记录顺序

```
from collections import OrderedDict
import random
d = {'banana': 3, 'apple': 4, 'pear': 1, 'orange': 2}
print(d)
keys = list(d.keys())
random.shuffle(keys)
print(keys)
od = OrderedDict()
for key in keys:
    od[key] = d[key]
print(od)
print(od.keys())
```

有序字典可以记录元素插入的顺序，打印的时候也是按照这个顺序输出打印

3.6版本的Python的字典就是记录key插入的顺序（IPython不一定有效果）

应用场景：

假如使用字典记录了N个产品，这些产品使用ID由小到大加入到字典中

除了使用字典检索的遍历，有时候需要取出ID，但是希望是按照输入的顺序，因为输入顺序是有序的

否则还需要重新把遍历到的值排序

# 字典方法

## 字典查询

d[key]

返回key对应的值value

key不存在抛出KeyError异常

get(key[, default])

根据'key'获取对应'value'

key不存在返回缺省值，如果没有设置缺省值就返回None

setdefault(key[, default])

增加一个键值对，如果字典内存在，则不做操作，如果不存在，则新增

返回key对应的值value

key不存在，添加kv对，value为default，并返回default，如果default没有设置，缺省为None

示例：

```
d = dict.fromkeys(range(5), [100])

a = d.setdefault(100)
print(d)
print(a)
"""
{0: [100], 1: [100], 2: [100], 3: [100], 4: [100], 100: None}
None
"""

b = d.setdefault(100, 10000)
print(d)
print(b)
"""
{0: [100], 1: [100], 2: [100], 3: [100], 4: [100], 100: None}
None
"""

c = d.setdefault(101, 10000)
print(d)
print(c)
"""
{0: [100], 1: [100], 2: [100], 3: [100], 4: [100], 100: None, 101: 10000}
10000
"""
```



## 字典修改

d[key] = value

将key对应的值修改为value

key不存在添加新的kv对

update([other]) -> None

使用另一个字典的kv对更新本字典，实现批量增加更新

key不存在，就添加

key存在，覆盖已经存在的key对应的值

就地修改

示例：

```
d.update(red=1)
d.update((('red',2),))
d.update({'red':3})
```



## 字典删除

pop(key[, default])

key存在，移除它，并返回它的value，并允许赋值给一个变量

key不存在，返回给定的default

default未设置，key不存在则抛出KeyError异常

popitem()

移除并返回一个任意的键值对,并允许将被删除的键值对赋值给2个变量

如果字典为empty，抛出KeyError异常

clear()

清空字典

del

删除键值对

看着像删除了一个对象，本质上减少了一次对象的引用计数，del 实际上删除的是名称，而不是对象

示例：

```
a = True
b = [6]
d = {'a': 1, 'b': b, 'c': [1,3,5]}  # 'b': b 注意这里有个变量引用

del a  # a被删除无法访问
del d['c'] # 删除了[1,3,5]的门牌号码,也就是引用计数-1
del b[0]  # 删除了列表内的元素

c = b  # c=[]
del c  # 删除了空列表[]的一个引用计数(门牌号码)
del b  # 删除了空列表[]的一个门牌号码
b = d['b']  # b=[]
```

## **字典遍历**

默认遍历的是字典的key

for .. in d.keys() 遍历字典的key

for .. in d.values() 遍历字典的value

for k,v in d.items() 遍历字典的key和value

示例:

```
dic = {'user':'fly', 'age':'25', 'gender':'man'}

for k in dic:
    print(k)
# 等同于
for k in dic.keys():
    print(k)

for v in dic.values():
    print(v)
# 等同于
for k in dic:
    print(dic[k])
# 等同于
for k in dic:
    print(dic.get(k))

for k, v in dic.items():
    print(k, v)
```

总结

Python3中，keys、values、items方法返回一个类似一个生成器的可迭代对象，不会把函数的返回结果复制到内存中

Dictionary view对象

字典的entry的动态的视图，字典变化，视图将反映出这些变化

Python2中，上面的方法会返回一个新的列表，占据新的内存空间。所以Python2建议使用iterkeys、itervalues、iteritems版本，返回一个迭代器，而不是一个copy

## 字典遍历并移除

```
d = dict(a=1, b=2, c='abc')

keys = []
for k,v in d.items():
    if isinstance(v, str):
        keys.append(k)

for k in keys:
    d.pop(k)
print(d)
```



## 字典嵌套

'k1':'v1'

'k2':[1,2,3]

'k3':[1,2,3,{'kk1':'vv1','kk2':'vv2'}]

...

例如：

```
user_dic =[
    {'name': 'jack', 'pwd': '123', 'times': 1},
    {'name': 'tom', 'pwd': '123', 'times': 1},
    {'name': 'rain', 'pwd': '123', 'times': 1},
]
for item in user_dic:
    print(item['name'])
    print(item['pwd'])
    print(item['times'])

# 输出结果：
jack
123
1
tom
123
1
rain
123
1
```

上面嵌套的都是values，还有一种是作为keys来嵌套，前提条件是不可变类型

'k1':'v1'

'k2':[1,2,3]

'k3':[1,2,3,{'kk1':'vv1','kk2':'vv2'}]

'(k4,k5)':[1,2,3,{'kk1':'vv1','kk2':'vv2'}]

不可变类型包括：元组、True、数字

# 字典练习

1,用户输入一个数字，打印每一位数字及其重复的次数

```
"""
假设输入了111223333
最终目标{'1':3, '2':2, '3':4}
"""
num = input('>>> ')
d = {}

for i in num:
    if i not in d.keys():
        d[i] = 0
    d[i] += 1

for c in num:
    d[c] = d.get(c, 0) + 1

print(d)
```

2,数字重复统计

随机产生100个整数

数字的范围[-1000, 1000]

升序输出所有不同的数字及其重复的次数

```
import random
numlist = []
for i in range(100):
    numlist.append(random.randint(-100, 100))
print(numlist)

numdic = {}
for i in numlist:
    if i not in numdic.keys():
        numdic[i] = 0
    numdic[i] += 1
print(numdic)

sort_numdit = sorted(numdic)
print(sort_numdic)
```

3,字符串重复统计

字符表'abcdefghijklmnopqrstuvwxyz'

随机挑选2个字母组成字符串，共挑选100个

按重复次数降序输出所有不同的字符串及重复的次数

```python
import random

words = []
alphabet = 'abcdefghijklmnopqrstuvwxyz'
for _ in range(100):
    words.append(''.join(random.choice(alphabet) for _ in range(2)))
print(words)

d = {}
for x in words:
    d[x] = d.get(x, 0) + 1
print(d)

d1 = sorted(d.items(), key=lambda item: item[1], reverse=True)
print(d1)
```

