[TOC]

一个'队列'，一个排列整齐的队伍

列表内的个体称作元素，由若干元素组成列表

元素可以是任意对象（数字、字符串、对象、列表等）

列表内元素有顺序，可以使用索引

线性的数据结构

使用[ ] 表示

列表是可变的

注意列表list、链表、queue(先进先出)、stack(栈,后进先出)的差异

# 定义

list() -> new empty list

list(iterable) -> new list initialized from iterable's items

列表不能一开始就定义大小

示例:

```
lst = list()
lst = []
lst = [2, 6, 9, 'ab']
lst = list(range(5))
```



# 方法

## 索引访问

索引，也叫下标

正索引：从左至右，从0开始，为列表中每一个元素编号

负索引：从右至左，从-1开始

正负索引不可以超界，否则引发异常IndexError

为了理解方便，可以认为列表是从左至右排列的，左边是头部，

右边是尾部，左边是下界，右边是上界

列表通过索引访问

list[index]：index就是索引，使用中括号访问

```
print(list01[0])
print(list01[-2])
print(list01[0:3])
```



## 列表查询

### index()

index(value,[start,[stop]])

通过值value，从指定区间查找列表内的元素是否匹配

匹配第一个就立即返回索引

匹配不到，抛出异常ValueError

### count()

count(value)

返回列表中匹配value的次数

### 时间复杂度

index和count方法都是O(n)

随着列表数据规模的增大，而效率下降

### len()

获取列表长度(统计的是列表元素的个数)

## 列表修改

### 索引访问修改

list[index] = value

```
list01[0] = 'fly'
```

注意：索引不要超界

### append()

append(object) -> None

列表尾部追加元素，返回None

返回None就意味着没有新的列表产生，就地修改

时间复杂度是O(1)

### insert()

insert(index, object) -> None

在指定的索引index处插入元素object

返回None就意味着没有新的列表产生，就地修改

时间复杂度是O(n)

超越上界，尾部追加

超越下界，头部追加

### extend()

extend(iteratable) --> None

将可迭代对象的元素追加进来，返回None

就地修改

```
lst = ['abc', 0, 1, 200, 10000]
lst.extend(range(11, 16))
print(lst)  # ['abc', 0, 1, 200, 10000, 11, 12, 13, 14, 15]
```

### 加号+

\+ -> list

连接操作，将两个列表连接起来

产生新的列表，原列表不变

本质上调用的是__add__()方法

```
print(list01 + list02)
```

### 星号*

\* -> list

重复操作，将本列表元素重复n次，返回新的列表

浅拷贝

```
# 简单元素类型
x = [1, 2, 3]
print(x * 3)  # [1, 2, 3, 1, 2, 3, 1, 2, 3]

y = [[4, 5, 6]]
print(y * 3)  # [[4, 5, 6], [4, 5, 6], [4, 5, 6]]

# 复杂元素类型
z = [[4, [5], 6]]
a = z * 3
print(a)  # [[4, [5], 6], [4, [5], 6], [4, [5], 6]]
print(z)  # [[4, [5], 6]]

a[0][1] = 99
print(a)  # [[4, 99, 6], [4, 99, 6], [4, 99, 6]]
print(z)  # [[4, 99, 6]]

z[0][1] = 100
print(a)  # [[4, 100, 6], [4, 100, 6], [4, 100, 6]]
print(z)  # [[4, 100, 6]]
```

理解python变量定义的原理，z指向的是一段内存地址, a指向z*3计算结果之后的内存地址,z发生改变,a也随之变化，反之a变化，对应的z也变化

## 删除元素

### remove()

remove(value) -> None

从左至右查找第一个匹配value的值，移除该元素，返回None

就地修改

时间复杂度O(n)

### pop()

pop([index]) -> item

移除列表中的一个元素(默认最后一个元素),并且返回该元素的值,这个值还可以赋值给变量

不指定索引index，就从列表尾部弹出一个元素,时间复杂度O(1)

指定索引index，就从索引处弹出一个元素，索引超界抛出IndexError错误，时间复杂度O(n)

### clear()

clear() -> None

清除列表所有元素，剩下一个空列表

## 其他操作

### reverse()

reverse() -> None

将列表元素反转，返回None

就地修改

反向排序列表中元素(默认按照ASCII码列表排序,从大到小)

### sort()

sort(key=None, reverse=False) -> None

对列表元素进行排序，就地修改，默认升序

reverse为True，反转，降序

key一个函数，指定key表示如何排序

lst.sort(key=function_name)

```
lst = [1, 2, 10, 11, 21, '1.5']
lst.sort()  # 报错
lst.sort(key=str)  # [1, '1.5', 10, 11, 2, 21]
lst.sort(key=int)  # 报错,str不能转换为int
lst.sort(key=float)  # [1, '1.5', 2, 10, 11, 21]

```



### sorted()

内建函数，对可迭代对象进行排序，返回一个新的可迭代对象,不影响原来的值

### in

判断一个或多个元素是否在列表中,返回True/False

```
print([3, 4] in [1, 2, [3, 4]])  # True
print([3] in [1, 2, [3,4]])  # False
```

## 列表复制

先来个示例：

```
lst0 = list(range(4))  # 新开内存地址
lst2 = list(range(4))  # 新开内存地址
print(lst0 == lst2)  # True, ==比较的是内容,内容相同, 内存地址不同

lst1 = lst0  # 引用计数，0和1指向了同一个内存地址
lst1[2] = 10
print(lst0)  # [0, 1, 10, 3]
print(lst1)  # [0, 1, 10, 3]
print(lst2)  # [0, 1, 2, 3]
```

lst0==lst2相等吗？为什么？lst0里面存的是什么？  相等, 内容相同, 内存地址不同, lst0=[0,1,2,3]

请问lst0的索引为2的元素的值是什么？  2

请问lst1 = lst0这个过程中有没有复制过程？  没有元素复制，仅有地址复制

### copy()

shadow copy，影子拷贝，也叫浅拷贝，遇到引用类型，只是复制了一个引用而已

返回一个新的列表

```
# 示例1
lst0 = list(range(4))
lst5 = lst0.copy()  # 返回一个新的列表,新的内存空间地址
print(lst5 == lst0)  # 比较的是内容
lst5[2] = 10
print(lst5 == lst0)  # 内容已经不一样

# 示例2
lst0 = [1, [2, 3, 4], 5]  # new
lst5 = lst0.copy()  # new
lst5 == lst0  # True
lst5[2] = 10
lst5 == lst0  # False
lst5[2] = 5
lst5[1][1] = 20  # 第二层列表指向的又是新的一块内存地址,lst0和lst5都有指向,copy仅复制了第一层
lst5 == lst0  # True
```

如图：

![img](Python%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B-%E5%88%97%E8%A1%A8list.assets/b428ec50-7a33-449c-bfbc-9c89463abf1b.jpg)

可以使用id()来查看内存地址编码

### 深拷贝

copy模块提供了deepcopy

```
import copy
lst0 = [1, [2, 3, 4], 5]
lst5 = copy.deepcopy(lst0)
lst5[1][1] = 20
lst5 == lst0  # False
```