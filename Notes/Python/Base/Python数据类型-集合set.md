[TOC]

# set

set 翻译为集合

collection 翻译为集合类型，是一个大概念

set是可变的、无序的、不重复的元素的集合

## set定义

set() -> new empty set object

set(iterable) -> new set object

例如:

```
s1 = set()
s2 = set(range(5))

print(s1,s2)
print(type(s1),type(s2))

"""
set() {0, 1, 2, 3, 4}
<class 'set'> <class 'set'>
"""
```



## set特点

\- 自动去重

\- 不支持索引取值

\- 支持迭代

\- set的元素要求必须可以hash

\- 可嵌套元组

## set方法

### add(elem)

增加一个元素到set中

如果元素存在，什么都不做

### update(*others)

批量更新，合并其他元素到set集合中来

参数others必须是可迭代对象

就地修改

```
s1 = {'a', 'b', 'c', 'd'}
s1.update({'111', '222', '333'})
print(s1)
```

|=  等同update

### remove(elem)

从set中移除一个元素

元素不存在，抛出KeyError异常

### discard(elem)

从set中移除一个元素

元素不存在，什么都不做

### pop()

移除并返回任意的元素，移除一个值并获取到移除的值赋值给一个变量

空集返回KeyError异常

### clear()

清空所有元素

### in 和 not in

先来看一下列表和集合的效率

```
lst1 = list(range(100))
lst2 = list(range(1000000))
%timeit (-1 in lst1)
%timeit (-1 in lst2)
'''
1.18 µs ± 8.79 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)
11.9 ms ± 237 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
'''

set1 = set(range(100))
set2 = set(range(1000000))
%timeit (-1 in set1)
%timeit (-1 in set2)
'''
36.2 ns ± 0.172 ns per loop (mean ± std. dev. of 7 runs, 10000000 loops each)
40 ns ± 0.329 ns per loop (mean ± std. dev. of 7 runs, 10000000 loops each)
'''

```



### set和线性结构

线性结构的查询时间复杂度是O(n)，即随着数据规模的增大而增加耗时；

set、dict等结构，内部使用hash值作为key，时间复杂度可以做到O(1)，查询时间和数据规模无关

可hash的数据类型：

数值型int、float、complex

布尔型True、False

字符串string、bytes

tuple

None

以上都是不可变类型，成为可哈希类型，hashable

set的元素必须是可hash的

## 集合

### 全集

所有元素的集合。例如实数集，所有实数组成的集合就是全集；

### 子集subset和超集superset

一个集合A所有元素都在另一个集合B内，A是B的子集，B是A的超集

### 真子集和真超集

A是B的子集，且A不等于B，A就是B的真子集，B是A的真超集

### 并集

多个集合合并的结果

将两个集合A和B的所有的元素合并到一起，组成的集合称作集合A与集合B的并集

![img](Python%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B-%E9%9B%86%E5%90%88set.assets/db2057ff-ca0b-4ccb-9273-31c0c6affa8f.jpg)

### 交集

多个集合的公共部分

集合A和B，由所有属于A且属于B的元素组成的集合

![img](Python%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B-%E9%9B%86%E5%90%88set.assets/a124eba5-899a-45a9-ba58-2056728e7f97.jpg)

### 差集

集合中除去和其他集合公共部分

集合A和B，由所有属于A且不属于B的元素组成的集合

![img](Python%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B-%E9%9B%86%E5%90%88set.assets/e34096e7-c2d1-48dc-ba87-1980f2a84b6c.jpg)

### 对称差集

集合A和B，由所有不属于A和B的交集元素组成的集合，记作（A-B）∪（B-A）

![img](Python%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B-%E9%9B%86%E5%90%88set.assets/613c0241-3dfb-478c-99f0-2f8d7adf9cd4.jpg)

## 集合方法

### isdisjoint()

是否有交集

### issubset()

是否是被比较者的子集

### issuperset()

是否是被比较者的父集

### difference()

比较不同之处,注意比较的双方位置

\-   等同difference

### difference_update()

-=  等同difference_update

### symmetric_difference()

比较不同之处，并将所有不同的元素同时赋值给一个变量

```
s1 = {'a', 'b', 'c', '111'}
s2 = {'a', 'b', 'c', '222'}
v = s1.symmetric_difference(s2)
print(v)
```

^   等同symmetric_differece

### symmetric_difference_update()

将比较结果重新更新赋值给比较者

^=  等同symmetric_differece_update

### intersection()

取比较对象的交集

```
s1 = {'a', 'b', 'c', '111'}
s2 = {'a', 'b', 'c', '222'}
v = s1.intersection(s2)
print(v)
```

&   等同intersection

### intersection_update()

将比较结果重新更新赋值给比较者

&=  等同intersection_update

### union()

取比较对象的并集

```
s1 = {'a', 'b', 'c', '111'}
s2 = {'a', 'b', 'c', '222'}
v = s1.union(s2)
print(v)
```

|   运算符重载，等同union

### union_update()

将比较结果重新更新赋值给比较者

# 集合应用

## 共同好友

你的好友A、B、C，他的好友C、B、D，求共同好友

例如：

```
set1 = {A、B、C}
set2 = {C、B、D}
if {'A', 'B', 'C'}.intersection({'B', 'C', 'D'}) == set():
    print('no')
else:
    print('yes')
```

intersection或& 求交集

## 微信群提醒

XXX与群里其他人都不是微信朋友关系

例如：

XXX与群里其他人都不是微信朋友关系

并集：userid in (A | B | C | ...) == False，A、B、C等是微信好友的并集，用户ID不在这个并集中，说明他和任何人都不是朋友

## 权限判断

1，有一个API，要求权限同时具备A、B、C才能访问，用户权限是B、C、D，判断用户是否能够访问该API

例如：

API集合A，权限集合P

A - P = {} ，A-P为空集，说明P包含A

A.issubset(P) 也行，A是P的子集也行

A & P = A 也行

2，有一个API，要求权限具备A、B、C任意一项就可访问，用户权限是B、C、D，判断用户是否能,够访问该API

例如：

API集合A，权限集合P

A & P != {} 就可以

A.isdisjoint(P) == False 表示有交集

## 任务完成度

一个总任务列表，存储所有任务。一个完成的任务列表。找出为未完成的任务

例如：

业务中，任务ID一般不可以重复

所有任务ID放到一个set中，假设为ALL

所有已完成的任务ID放到一个set中，假设为COMPLETED，它是ALL的子集

ALL - COMPLETED = UNCOMPLETED

## 练习

随机产生2组各10个数字的列表，如下要求：

每个数字取值范围[10,20]

统计20个数字中，一共有多少个不同的数字？

2组中，不重复的数字有几个？分别是什么？

2组中，重复的数字有几个？分别是什么？

```
a = [1, 9, 7, 5, 6, 7, 8, 8, 2, 6]
b = [1, 9, 0, 5, 6, 4, 8, 3, 2, 3]
s1 = set(a)
s2 = set(b)
print(s1)
print(s2)
print(s1.union(s2))
print(s1.symmetric_difference(s2))
print(s1.intersection(s2))
```