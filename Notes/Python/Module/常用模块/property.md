[TOC]

# **property**

属性方法,内置函数,装饰器

将类的一个方法变成一个静态属性(变量)，可让实例来调用，不需加括号即可运行

一旦在函数属性前加上装饰器property，这个函数就会变成一个数据属性

基本语法：

 

```
class Dog(object):
    def __init__(self, name):
        self.name = name
    @property
    def eat(self):
        print('%s is eating..' % self.name)
d = Dog('wangwang')
d.eat  # 不加@property,实例调用方法时,需要加括号d.eat(),但如果加上,则无须加括号,否则会报错
```

应用场景：

比如想要知道一个航班当前的状态，是到达了,延迟了,取消了还是飞走了,首先要连接航空API查询,分析,然后返回结果给用户,这个结果是一些列动作后才得到的.所以每次调用时,无需关系它经过了怎样的处理有过什么动作,我们只需要知道它的结果,并可以调用这个属性即可.

示例1:

计算成人BMI值

过轻：低于18.5

正常：18.5-23.9

过重：24-27

肥胖：28-32

非常肥胖：高于32

体质指数(BMI) = 体重(kg) / (身高(m) ^ 2)

 

```
class People:
    def __init__(self, name, weight, height):
        self.name = name
        self.weight = weight
        self.height = height
    @property
    def bmi(self):

        return self.weight / (self.height ** 2)

p = People('rain', 75, 1.80)
# print(p.bmi())

print(p.bmi)


运行结果：
23.148148148148145
```

示例2:

计算圆的周长和面积

 

```
import math
class Circle:
    def __init__(self, radius):  # 圆的半径radius
        self.radius = radius

    @property
    def area(self):  # 计算圆的面积
        return math.pi * self.radius ** 2

    @property
    def perimeter(self):  # 计算圆的周长
        return 2 * math.pi * self.radius

c = Circle(10)
print(c.radius)
print(c.area)
print(c.perimeter)

运行结果：
10
314.1592653589793
62.83185307179586
```

## 方法

property有2个动作方法：

既然property可以将一个方法变成一个属性,那么是否可以类似变量那样,赋值和删除呢?

property有2个方法可以实现:

property.setter:修改

property.deleter:删除

实例1:

 

```
class People:
    def __init__(self, name, permission=False):
        self.__name = name  #可转换为:self._People__name = name
        self.permission = permission  #用于判断,是否允许删除

    @property  #可转换为:name = property(name)
    def name(self):
        return self.__name  #可转换为:self._People__name

    @name.setter  #name = name.setter(name)修改
    def name(self, value):
        if not isinstance(value, str):  #判断value是否是字符串
            raise TypeError('输入的内容必须是字符串.')
        self.__name = value  #self._People__name = value

    @name.deleter  #name = name.deleter(name)删除
    def name(self):
        if not self.permission:
            raise TypeError('禁止删除.')
        del self.__name
        print('已删除名字')


p = People('rain')
print(p.name())
print(p.name)
p.name = 'rain656'
print(p.name)
p.name = 123123
print(p.name)
p.permission = True
del p.name
print(p.name)
```

[注意：加了装饰器property,arear和perimeter不能被赋值]