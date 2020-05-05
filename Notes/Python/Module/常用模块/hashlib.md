[TOC]

# hashlib

hash:一种算法.

在py3里代替了md5模块和sha模块.主要提供 SHA1, SHA224, SHA256, SHA384, SHA512 ，MD5 算法.

三个特点：

1,内容相同则hash运算结果相同,内容稍微改变则hash值则变

2,不可逆推

3,相同算法,无论校验多长的数据,得到的哈希值长度固定.

示例:

 

```
import hashlib

m=hashlib.md5()
m.update('hello'.encode('utf-8'))
print(m.hexdigest())
"""
运行结果：
5d41402abc4b2a76b9719d911017c592

"""
```

update多次,与update一次,得到的结果一样,但update多次为校验大文件提供了可能.

示例2:对文件进行校验

 

```
"""
创建一个文件:a.txt
文件内容:12345
"""

import hashlib
m = hashlib.md5()
with open(r'a.txt, 'rb') as f:
  for line in f:
      m.update(line)
  md5_num = m.hexdigest()
print(md5_num)
"""
运行结果:
827ccb0eea8a706c4c34a16891f84e7b
"""
```

注意:以上加密算法虽然依然非常厉害,但有时候也会存在风险,比如通过撞库可以反解.所以,有必要对加密算法中添加自定义key再来做加密.

例如：

1,sha256加密

2,通过添加一个自定义的key,在进行sha256加密.('加盐'操作.)

 

```
import hashlib

hash = hashlib.sha256() # sha256加密
hash.update('hello'.encode('utf-8'))
print('--->', hash.hexdigest())

hash = hashlib.sha256('hello'.encode('utf-8'))  # 加盐加密
hash.update('123'.encode('utf8'))
print('===>', hash.hexdigest())

"""
运行结果:
---> 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
===> 27cc6994fc1c01ce6659c6bddca9b69c4c6a9418065e612c69d110b3f7b11f8a
```

撞库模拟示例:

库,可以理解为一个密码字典,里面记录了一些常用的密码,用于遍历每一个密码去检测密码是否匹配正确的密码.

 

```
import hashlib

passwds=[
  'alex3714',
  'alex1313',
  'alex94139413',
  'alex123456',
  '123456alex',
  'a123lex',
]

def make_passwd_dic(passwds):
    dic = {}
    for passwd in passwds:
        m = hashlib.md5()
        m.update(passwd.encode('utf-8'))
        dic[passwd] = m.hexdigest()
    return dic

def break_code(cryptograph,passwd_dic):
"""
  cryptograph:为抓取到的密文密码
  :param cryptograph: 
  :param passwd_dic: 密码库
  :return: 
"""
    for k, v in passwd_dic.items():
        if v == cryptograph:
            print('密码是===>: \033[46m%s\033[0m' %k)

cryptograph = 'aee949757a2e698417463d47acac93df' # 此处模拟alex3714经过md5算法产生的密文密码

break_code(cryptograph, make_passwd_dic(passwds))

"""
运行结果：
密码是===>: alex3714
"""
```