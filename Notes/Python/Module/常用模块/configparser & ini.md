[TOC]

# ini文件

作为配置文件，ini文件格式很流行

ini格式如下：

 

```
[DEFAULT]
a = test

[mysql]
default-character-set=utf8

[mysqld]
datadir=/data/mysql
port=33060
character-set-server=utf8
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
```

中括号部门为section，节、区、段;

每一个section内,都是key=value形成的键值对,key称为option选项，默认支持等号'='与分号':'两种分隔符号.

注意这里的DEFAULT是缺省section的名字,必须大写。

# configparser

configparser模块中ConfigParser类是python标准库中用来解析配置文件的模块,并且内置方法和字典非常接近.

可以将ini配置文件中section当作key，section存储着键值对组成的嵌套字典,默认使用的是有序字典。

使用方式:

使用configparser首先要初始化示例,并读取配置文件.

假设配置文件config.ini内如如下:

 

```
[DEFAULT]
key1=1
keya=a
foo=100000

[section1]
key1 = value1
key2 = value2
#key3 = value3

[section2]
keya = valueA
keyb = valueB
keyc = valueC

[section3]
foo = 999
bar = True
baz = 3.1415
```

## 读取数据

read(filenames, encoding=None)

读取ini文件，可以使单个文件，也可以是文件列表，可指定文件编码。

sections()

返回section列表,缺省的section[DEFAULT]不包括在内。

has_section(section_name)

判断section是否存在

options(section)

获取section下所有的option(key-value对),会追加缺省section的option

has_option(section, option)

判断section是否存在这个option

get(section, option, *, raw=False, vars=None[, fallback])

从指定的段section获取option,如果找到则返回，找不到则去DEFAULT段找

getint(section, option, *, raw=False, vars=None[, fallback])

getfloat(section, option, *, raw=False, vars=None[, fallback])

getboolean(section, option, *, raw=False, vars=None[, fallback])

返回指定类型的数据

items(raw=False, vars=None)

items(section, raw=False, vars=None)

没有指定section，则返回所有section名字及其对象，如果指定section，则返回指定section的键值对组成的二元组。

示例：

 

```
import configparser

# 实例化
config = configparser.ConfigParser()

# 读取指定ini文件内容
config.read('config.ini', encoding='utf-8')
config.read_dict({'a':1})

# 获取所有的段名,返回列表
res=config.sections()
print(res)  # ['section1', 'section2', 'section3']

# 获取指定段名section1下key=value中所有的key,通常称为options
options=config.options('section1')
print(options) # ['key1', 'key2', 'key3']

# 获取标题section1下所有key=value的(key,value)格式
item_list=config.items('section1')
print(item_list) # [('key1', 'value1'), ('key2', 'value2'), ('key3', 'value3')]

# 获取标题section1下key3的值,字符串格式,如果不存在则报错
val = config.get('section1','key3')
print(val) # value3

# 获取标题section3下foo的值,数值格式
val2 = config.getint('section3','foo')
print(val2) # 999

# 获取标题section3下bar的值,布尔值格式
val3 = config.getboolean('section3','bar')
print(val3) # True

# 获取标题section3下baz的值=>浮点型格式
val4=config.getfloat('section3','baz')
print(val4) # 3.1415
```

## 修改数据

add_section(section_name)

增加一个section

set(section, option, value)

section存在的情况下，写入option=value, 要求option和value必须是字符串

remove_section(section)

移除section及其所有option

remove_option(section, option)

移除指定section下的option

write(fileobject, space_around_delimiters=True)

将当前config的所有的内容写入fileobj中，一般open函数使用w模式

示例：

 

```
# 删除section2标题
config.remove_section('section2')
print(config.sections())  # ['section1', 'section3']

# 删除标题section1下的k1和k2
config.remove_option('section1','key1')
config.remove_option('section1','key2')
print(config.options('section1'))  # ['key3']

# 判断是否存在某个标题
print(config.has_section('section1'))  # True
print(config.has_section('section100'))  # False

# 判断标题section1下是否有key3,key5
print(config.has_option('section1','key3'))  # True
print(config.has_option('section1','key5'))  # False

# 添加一个标题
config.add_section('egon')
print(config.sections())  # ['section1', 'section2', 'section3', 'egon']

# 在标题egon下添加name:egon,age:18数据
config.add_section('egon')
config.set('egon','name','egon')
config.set('egon','age','18')  # 这里必须全部是str格式
print(config.items('egon'))  # [('name', 'egon'), ('age', '18')]

# 最后将修改的内容写入文件,完成最终的修改
with open(newfilename, 'w', encoding='utf-8') as f:
    config.write(f)
```

# 应用:ini模板

 

```
import configparser
  
config = configparser.ConfigParser()

# 设置DEFAULT标题及其内容
config['DEFAULT'] = {
    'ServerAliveInterval': '45',
    'Compression':'yes',
    'CompressionLevel':'9',
}
config['DEFAULT']['ForwardX11'] = 'yes'


# 定义标题为bitbucket.org的内容
config['bitbucket.org'] = {}
config['bitbucket.org']['User'] = 'hg'

# 定义标题为topsecret.server.com的内容
config['topsecret.server.com'] = {}
topsecret = config['topsecret.server.com']
topsecret['Host Port'] = '50022'
topsecret['ForwardX11'] = 'no'

# 写入文件
with open('example.ini', 'w') as configfile:
    config.write(configfile)
```

生成的example.ini文件内容如下:

 

```
[DEFAULT]
serveraliveinterval = 45
compression = yes
compressionlevel = 9
forwardx11 = yes

[bitbucket.org]
user = hg

[topsecret.server.com]
host port = 50022
forwardx11 = no
```