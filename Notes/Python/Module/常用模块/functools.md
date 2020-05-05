[TOC]

# functools 

functools用于高阶函数，即那些作用于函数或者返回其他函数的函数，通常只要是可以被当做函数调用的对象就是这个模块的目标。

类似copy_properties函数(详情请参阅[装饰器](wiz://open_document?guid=ef0a184c-8a0b-4b14-9f2d-1400b995ef0c&kbguid=&private_kbguid=c598329a-66c4-4d09-8758-8a5f3be4fe9b))

functools 是python2.5被引入的,一些工具函数放在此包里。

python2.7： dir(functools)

python3: import functools --> dir(functools)

## 常用工具函数

### update_wrapper

functools.update_wrapper(wrapper,wrapped, assigned=WRAPPER_ASSIGNMENTS, updated=WRAPPER_UPDATES)

wrapper：包装函数、被更新者

wrapped：被包装函数、数据源，就是被装饰函数

assigned：赋值,元祖WRAPPER_ASSIGNMENTS中是要被覆盖的属性,如模块名'__module__', 函数名称'__name__', 限定名'__qualname__', 文档'__doc__', 参数注解'__annotations__'

updated：更新，元祖WRAPPER_UPDATES中是要被更新的属性，__dict__属性字典；

增加一个__wrapped__属性，保留着wrapped函数

 

```
import datetime, time, functools

def logger(duration, func=lambda name, duration: print('{} took {}s'.format(name, duration))):
    def _logger(fn):
        def wrapper(*args,**kwargs):
            start = datetime.datetime.now()
            ret = fn(*args,**kwargs)
            delta = (datetime.datetime.now() - start).total_seconds()
            if delta > duration:
                func(fn.__name__, duration)
            return ret
        return functools.update_wrapper(wrapper, fn)  # 在这里使用
    return _logger

@logger(5) # add = logger(5)(add)
def add(x,y):
    time.sleep(1)
    return x + y

print(add(5, 6), add.__name__, add.__wrapped__, add.__dict__, sep='\n')
```

### update_wraps

使用方法：

@functools.wraps(wrapped, assigned=WRAPPER_ASSIGNMENTS,updated=WRAPPER_UPDATES)

类似copy_properties功能

wrapped：被包装函数

两个默认参数：assigned和updated

元组WRAPPER_ASSIGNMENTS中是要被覆盖的属性'__module__'(模块名), '__name__'(名称), '__qualname__'(限定名), '__doc__'(文档), '__annotations__'(参数注解)

元组WRAPPER_UPDATES中是要被更新的属性，__dict__(属性字典)

增加一个__wrapped__属性，保留着wrapped函数.

 

```
import datetime, time, functools

def logger(duration, func=lambda name, duration: print('{} took {}s'.format(name, duration))):
    def _logger(fn):
        @functools.wraps(fn)  # 在这里使用
        def wrapper(*args,**kwargs):
            start = datetime.datetime.now()
            ret = fn(*args,**kwargs)
            delta = (datetime.datetime.now() - start).total_seconds()
            if delta > duration:
                func(fn.__name__, duration)
            return ret
        return wrapper
    return _logger

@logger(5) # add = logger(5)(add)
def add(x,y):
    time.sleep(1)
    return x + y

print(add(5, 6), add.__name__, add.__wrapped__, add.__dict__, sep='\n')
```

### partial

方法。偏函数，把函数部分的参数固定下来，相当于为部分的参数添加了一个固定的默认，返回一个新的函数；

从partial生成的新函数，是对原函数的封装。

 

```
import functools
import inspect

# 示例1
def add(x,y):
    return x+y

newadd1 = functools.partial(add, y=5)
newadd2 = functools.partial(add, 3, 9)
print(newadd1(6))  # 11
print(newadd2())  # 12
print(newadd1(y=10, x=6))  # 16

# 查看新函数的签名
print(inspect.signature(newadd1))  # (x, *, y=5)


# 示例2
def add1(x, y, *args) -> int:
    print(args)
    return x + y

newadd3 = functools.partial(add1, 1, 3, 6, 5)
print(inspect.signature(newadd3))  # (*args) -> int
print(newadd(7))
print(newadd(7, 10))
print(newadd())
```

### partial本质

其源代码

 

```
def partial(func, *args, **keywords):
    def newfunc(*fargs, **fkeywords): # 包装函数
        newkeywords = keywords.copy()
        newkeywords.update(fkeywords)  # 字典合并
        return func(*(args + fargs), **newkeywords)
    newfunc.func = func # 保留原函数
    newfunc.args = args # 保留原函数的位置参数
    newfunc.keywords = keywords # 保留原函数的关键字参数参数
    return newfunc

def add(x,y):
    return x+y

foo = partial(add,4)
foo(5)
```

### lru_cache

Least-recently-used装饰器,lru，最近最少使用，cache缓存。

格式：

@functools.lru_cache(maxsize=128, typed=False)

如果maxsize设置为None，则禁用LRU功能，并且缓存可以无限制增长；

当maxsize是二的幂时，LRU功能执行得最好；

如果typed设置为True，则不同类型的函数参数将单独缓存，例如，f(3)和f(3.0)将被视为具有不同结果的不同调用。

 

```
import functools
import time

@functools.lru_cache()  # 有参装饰器且有默认值
def add(x, y, z=3):
    time.sleep(z)
    return x + y

# 第一次执行
print(add(4, 5))  # 3s后出结果
print(add(4.0, 5))  # 立刻出结果
print(add(4, 6))  # 继续等3s后出结果

# 第二次执行
add(4, 6, 3)  # 3s后出结果
add(6, 4)  # 等3s后出结果
add(4, y=6)  # 等3s后出结果
add(x=4, y=6)  # 等3s后出结果
add(y=6, x=4)  # 立即出结果,同add(x=4, y=6)
```

### 缓存机制

lru_cache装饰器，通过一个字典缓存被装饰函数的调用和返回值

key是什么？分析代码看看

 

```
functools._make_key((4,6),{'z':3},False)
# [4, 6, <object at 0x7f53bf74b0b0>, 'z', 3]

functools._make_key((4,6,3),{},False)
# [4, 6, 3]

functools._make_key(tuple(),{'z':3,'x':4,'y':6},False)
# [<object at 0x7f53bf74b0b0>, 'x', 4, 'y', 6, 'z', 3] 这里是排序过的

functools._make_key(tuple(),{'z':3,'x':4,'y':6}, True)
# [<object at 0x7f53bf74b0b0>, 'x', 4, 'y', 6, 'z', 3, int, int, int]
```

### **缓存应用**

裴波那契数列递归方法的改造

 

```
# 原方法
def fib(n):
    if n < 3:
        return n
    return fib(n-1) + fib(n-2)

print([fib(x) for x in range(35)])

# 改造方法
import functools

@functools.lru_cache()
def fib(n):
    if n < 3:
        return n
    return fib(n-1) + fib(n-2)

print([fib(x) for x in range(35)])
```

### 总结

使用前提：

同样的函数参数一定得到同样的结果

函数执行时间很长，且要多次执行

本质是函数调用的参数=>返回值

缺点：

不支持缓存过期，key无法过期、失效

不支持清除操作

不支持分布式，是一个单机的缓存

适用场景：

单机上需要空间换时间的地方，可以用缓存来将计算变成快速的查询