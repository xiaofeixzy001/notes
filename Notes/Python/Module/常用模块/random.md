[TOC]

# **random**

randint(a, b) 返回[a, b]之间的整数

random.choice(seq) 从非空序列的元素中随机挑选一个元素

randrange ([start,] stop [,step]) 从指定范围内，按指定基数递增的集合中获取一个随机数，基数缺省值为1

random.shuffle(list) -> None 就地打乱列表元素

random.sample(population, k) 从样本空间或总体（序列或者集合类型）中随机取出k个不同的元素，返回一个新的列表

示例：

 

```
import random
print(random.random())  # 随机显示大于0且小于1之间的某个小数  运行结果:0.8411519662343422

print(random.randint(1, 3))  # 随机显示一个大于等于1且小于等于3之间的整数 运行结果:2

print(random.randrange(1, 3))  # 随机显示一个大于等于1且小于3之间的整数  运行结果:1

print(random.choice([1, '23', [4, 5]]))  # 随机选择列表中的元素,1或者23或者[4,5]  运行结果:1

print(random.sample([1, '23', [4, 5]], 2))  # 随机选择列表元素任意2个组合,拼成一个新的列表  运行结果:[[4, 5], '23']

print(random.uniform(1, 3))  # 随机选择大于1小于3的小数,如1.927109612082716  运行结果:1.1835899789185504

item = [1, 3, 5, 7, 9]
random.shuffle(item)  # 打乱item的顺序,相当于"洗牌"
print(item)  # 运行结果:[9, 7, 1, 3, 5]
```

应用场景:随机验证码

 

```
import random
def v_code(n=5):
    res=''
    for i in range(n):
        num=random.randint(0,9) # 随机选择数字
        s=chr(random.randint(65,90)) # 随机选择字母,chr()转换数字为ASCII码对应的字母
        t = chr(random.randint(97,122))
        add=random.choice([num,s,t])
        res+=str(add)  # res = res + str(add) 字符串拼接
    return res
print(v_code())
"""
运行结果:
91W2
"""
```

常用ASCII码

0-9:48-57

A-Z：65-90

a-z：97-122