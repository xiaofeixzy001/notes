[TOC]

# Pymysql

在python3.x中，可以使用pymysql来MySQL数据库的连接，并实现数据库的各种操作。

# 安装

## windows

pip3 install pymysql -i https://pypi.douban.com/simple

## linux

 

```
wget https://pypi.python.org/packages/29/f8/919a28976bf0557b7819fd6935bfd839118aff913407ca58346e14fa6c86/PyMySQL-0.7.11.tar.gz#md5=167f28514f4c20cbc6b1ddf831ade772
python36 setup.py install
```

# 使用

## 测试环境

系统：CentOS release 6.9 (Final)

服务器IP：172.16.1.5

MySQL版本：mysql  Ver 14.14 Distrib 5.5.59, for linux-glibc2.12 (x86_64) using readline 5.1

MySQL管理员和密码：root/123.com 

 

```
# 创建库dbforpymysql
mysql> create database dbforpymysql;

# 创建表userinfo
mysql> use dbforpymysql;
mysql> CREATE TABLE userinfo (
    'id' INT NOT NULL auto_increment PRIMARY KEY,
    'username' VARCHAR (10),
    'passwd' VARCHAR (10)
) ENGINE = INNODB DEFAULT charset = utf8;

# 插入3条数据用于测试
mysql> insert into userinfo (username, passwd) values('frank','123'),('rose','321'),('jeff','666');

# 查看插入的数据
mysql> select * from userinfo;
```

## 连接数据库

pymysql.connect()方法返回的是Connections模块下的Connection类实例。connect方法传参就是给Connection类的__init___提供参数。

Connection.ping()方法，测试数据库服务器是否活着。有一个参数reconnect表示断开与服务器连接是否重连。

 

```
import pymysql
from pymysql.cursors import DictCursor

config = {
    "host":"192.168.100.5",
    "user":"root",
    "password":"123.com",
    "database":"school"
}

sql_cmd1 = "select * from student"
sql_cmd2 = "insert into student (name, age) values ('jerry', 25)"

try:
    # 连接数据库,参数可自行指定，也可以传一个字典(字典KEY固定)

    # db = pymysql.connect("192.168.100.5", "root", "123.com", "school")
    db = pymysql.connect(**config)
    print(db.ping(False))

    # 使用cursor()方法创建游标对象,返回元组
    cursor = db.cursor()
    
    # 使用cursor(DictCursor)方法创建游标对象,返回字典
    cursor = conn.cursor(DictCursor)
    rows = cursor.execute(sql_cmd1)
    print(1, rows)
    print(2, cursor.fetchall())
    """
    [{'id': 23, 'name': 'jerry', 'age': 25}, 
    {'id': 29, 'name': 'jack01', 'age': 2}, 
    {'id': 30, 'name': 'jack02', 'age': 3}]
    """

    # 使用execute()方法传递sql语句
    cursor.execute(sql_cmd2)

    # 使用fetchall()获取所有的返回数据
    data = cursor.fetchall()

    # 使用fetchone()获取单条数据
    # data = cursor.fetchone()

    print(data)

finally:
    if db:
        # 关闭游标和数据库的连接
        cursor.close()
        db.close()
    # print(db.ping(False))

```

说明：cursor其实是调用了cursors模块下的Cursor的类，这个模块主要的作用就是用来和数据库交互的，当你实例化了一个对象的时候，你就可以调用对象下面的各种绑定方法。

select可以获取到结果，但插入语句，没有成功。原因在于，Connection类的__init__方法注释中有说明: autocommit: Autocommit mode. None means use server default. (default: False)，所以需要手动管理事务提交。

## 事务管理

Connection类有三个方法：

begin开始事务,自动

commit提交事务,手动

rollback回滚事务,手动

在数据库里增、删、改的时候，必须要进行提交，否则插入的数据不生效。

 

```
sql_cmd1 = "select * from student"
sql_cmd2 = "insert into student (name, age) values ('jerry', 25)"
sql_cmd3 = "delete from student where name='tom'"

try:
    conn = pymysql.connect(**config)
    cursor = conn.cursor()
    # rows = cursor.execute(sql_cmd3)
    # print(rows)

    # 批量增加
    for i in range(5):
        sql_cmd4 = "insert into student (name, age) values ('jack0{0}', 1+{0})".format(i)
        rows = cursor.execute(sql_cmd4)

    # 批量删除
    # for i in range(5):
    #     sql_cmd4 = "delete from student where name='jack0{}'".format(i)
    #     rows = cursor.execute(sql_cmd4)
    conn.commit()
except:
    conn.rollback()
finally:
    if cursor:
        cursor.close()
    if conn:
        conn.close()
```

说明：

执行事务

事务机制可以确保数据的一致性

1.事务有四个属性：原子，一致，隔离，持久；通常称为ACID

2.Python DB API 2.0的事务提供了两个方法：commit 和 rollback

3.对于支持事务的数据库，在python数据库编程中，当游标建立之时，就自动开始了一个隐形的数据库事务，

这个区别于mysql客户端，commit()方法提交所有的事务，rollback()方法回滚当前游标的所有操作。每个方法都开启了一个新的事务。

一般流程总结

1 建立连接

2 获取游标

3 执行SQL

4 提交事务

5 释放资源

### executemany()方法

用来同时插入多条数据

 

```
import pymysql
config = {
    "host":"172.16.1.5",
    "user":"root",
    "password":"123.com",
    "database":"dbforpymysql"
}
db = pymysql.connect(**config)
cursor = db.cursor()
sql = "INSERT INTO userinfo(username,passwd) VALUES(%s,%s)"
cursor.executemany(sql,[("tom","123"),("alex",'321')])
db.commit()  # 提交数据
cursor.close()
db.close()
```

execute()和executemany()都会返回受影响的行数：

 

```
sql = "delete from userinfo where username=%s"
res = cursor.executemany(sql,("jack",))
print("res=",res)
#运行结果: res= 1
```

当表中有自增的主键的时候，可以使用lastrowid来获取最后一次自增的ID：

 

```
import pymysql

config = {
    "host":"172.16.1.5",
    "user":"root",
    "password":"123.com",
    "database":"dbforpymysql"
}
db = pymysql.connect(**config)
cursor = db.cursor()
sql = "INSERT INTO userinfo(username,passwd) VALUES(%s,%s)"
cursor.execute(sql,("zed","123"))
print("the last rowid is ",cursor.lastrowid)
db.commit()  #提交数据
cursor.close()
db.close()

#运行结果: the last rowid is  10
```

## 查询

fetchone(): 获取结果集的下一行。

fetchonemany(size=None): size指定返回的行数的行，None则返回空元祖。

fetchall(): 获取所有行。

返回多行，如果游标走到末尾，就返回空元祖，否则返回一个元组，其元素就是每一行的记录，每一行的记录也封装在一个元组中。

cursor.rownumber: 返回当前行号。可修改，支持负数。

cursor.rowcount: 返回总行数。

mysql中dbforpymysql.userinfo表中数据

 

```
# mysql
mysql> select * from userinfo;
+----+----------+--------+
| id | username | passwd |
+----+----------+--------+
|  1 | frank    | 123    |
|  2 | rose     | 321    |
|  3 | jeff     | 666    |
|  5 | bob      | 123    |
|  8 | jack     | 123    |
| 10 | zed      | 123    |
+----+----------+--------+
rows in set (0.00 sec)
```

fetchone():获取下一行数据，第一次为首行

 

```
import pymysql
config = {
    "host":"172.16.1.5",
    "user":"root",
    "password":"123.com",
    "database":"dbforpymysql"
}
db = pymysql.connect(**config)
cursor = db.cursor()
sql = "SELECT * FROM userinfo"
cursor.execute(sql)
res = cursor.fetchone() #第一次执行
print(res)
res = cursor.fetchone() #第二次执行
print(res)
res = cursor.fetchone() #第三次执行
print(res)
cursor.close()
db.close()

#运行结果
(1, 'frank', '123')
(2, 'rose', '321')
(3, 'jeff', '666')
```

fetchall():获取所有行数据源

 

```
res = cursor.fetchall() #第一次执行
print(res)
res = cursor.fetchall() #第二次执行
print(res)
res = cursor.fetchall() #第三次执行
print(res)
'''
运行结果：
((1, 'frank', '123'), (2, 'rose', '321'), (3, 'jeff', '666'), (4, 'xiaofei', '123'), (5, 'tom', '123'), (6, 'alex', '321'))
()
()
```

第二三次获取的时候，什么数据都没有获取到，这个类似于文件的读取操作。

## 带列名查询

默认情况下，我们获取到的返回值是元组，只能看到每行的数据，却不知道每一列代表的是什么，这个时候可以使用以下方式来返回字典，每一行的数据都会生成一个字典：

cursor = db.cursor(cursor=pymysql.cursors.DictCursor)  

意思是在实例化的时候，将属性cursor设置为pymysql.cursors.DictCursor

所以修改后代码如下：

 

```
import pymysql
config = {
    "host":"172.16.1.5",
    "user":"root",
    "password":"123.com",
    "database":"dbforpymysql"
}
db = pymysql.connect(**config)
cursor = db.cursor(cursor=pymysql.cursors.DictCursor) # 游标设置为字典类型
sql = "SELECT * FROM userinfo"
cursor.execute(sql)
res = cursor.fetchall()
print(res)
cursor.close()
db.close()
'''
运行结果
[{'id': 1, 'username': 'frank', 'passwd': '123'}, {'id': 2, 'username': 'rose', 'passwd': '321'}, {'id': 3, 'username': 'jeff', 'passwd': '666'}, {'id': 5, 'username': 'bob', 'passwd': '123'}, {'id': 8, 'username': 'jack', 'passwd': '123'}, {'id': 10, 'username': 'zed', 'passwd': '123'}]
这样获取到的内容就能够容易被理解和使用了！
'''
```

在获取行数据的时候，可以理解开始的时候，有一个行指针指着第一行的上方，获取一行，它就向下移动一行，所以当行指针到最后一行的时候，就不能再获取到行的内容，所以我们可以使用如下方法来移动行指针：

cursor.scroll(1,mode='relative')  # 相对当前位置移动

cursor.scroll(2,mode='absolute')  # 相对绝对位置移动

第一个值为移动的行数，整数为向下移动，负数为向上移动，mode指定了是相对当前位置移动，还是相对于首行移动。

例如：

 

```
sql = "SELECT * FROM userinfo"
cursor.execute(sql)
res = cursor.fetchall()
print(res)
cursor.scroll(0,mode='absolute') # 相对首行移动了0，就是把行指针移动到了首行
res = cursor.fetchall()  # 第二次获取到的内容
print(res)

'''
运行结果
[{'id': 1, 'username': 'frank', 'passwd': '123'}, {'id': 2, 'username': 'rose', 'passwd': '321'}, {'id': 3, 'username': 'jeff', 'passwd': '666'}, {'id': 5, 'username': 'bob', 'passwd': '123'}, {'id': 8, 'username': 'jack', 'passwd': '123'}, {'id': 10, 'username': 'zed', 'passwd': '123'}]
[{'id': 1, 'username': 'frank', 'passwd': '123'}, {'id': 2, 'username': 'rose', 'passwd': '321'}, {'id': 3, 'username': 'jeff', 'passwd': '666'}, {'id': 5, 'username': 'bob', 'passwd': '123'}, {'id': 8, 'username': 'jack', 'passwd': '123'}, {'id': 10, 'username': 'zed', 'passwd': '123'}]
'''
```

fetchmany(2):获取下2行数据

### 

 

```
res = cursor.fetchmany(2)

'''
运行结果：
[{'id': 1, 'username': 'frank', 'passwd': '123'}, {'id': 2, 'username': 'rose', 'passwd': '321'}]
'''
```

注意：fetch操作的是结果集，结果集是保存在客户端的，也就是说fetch的时候，查询已经结束了。

## 

## 上下文管理

在python的文件操作中支持上下文管理器，在操作数据库的时候也可以使用。

查看连接类和游标类的源码：

 

```
# 连接类
class Connection(object):
    def __enter__(self):
        """Context manager that returns a Cursor"""
        warnings.warn(
            "Context manager API of Connection object is deprecated; Use conn.begin()",
            DeprecationWarning)
        return self.cursor()

    def __exit__(self, exc, value, traceback):
        """On successful exit, commit. On exception, rollback"""
        if exc:
            self.rollback()
        else:
            self.commit()

# 游标类
class Cursor(object):
    def __enter__(self):
        return self

    def __exit__(self, *exc_info):
        del exc_info
        self.close()
```

连接类进入上下文的时候会返回一个游标对象，退出时如果没有异常会提交更改。

游标类也使用上下文，在退出时关闭游标对象。

 

```
config = {
    "host": "192.168.100.5",
    "user": "root",
    "password": "123.com",
    "database": "school"
}
conn = pymysql.connect(**config)
with conn as cursor:
    with cursor:
        sql_cmd1 = "select * from student"
        cursor.execute(sql_cmd1)
        print(cursor.fetchall())
conn.close()
```

上面的示例可以知，连接应该不需要反复被创建销毁，应该是多个cursor共享一个conn。

上下文管理器可以使代码的可读性更强。

# SQL注入攻击

猜测后台数据库的查询语句，使用拼接字符串的方式，从而经过设计为服务端传参，令其拼出特殊字符串，返回用户想要的结果。

永远不要相信客户端传来的数据是规范和安全的。

看一下实例，说名SQL注入攻击的危险程度。

用户名和密码都存在表userinfo中，内容如下：

 

```
+----+----------+--------+
| id | username | passwd |
+----+----------+--------+
|  1 | frank    | 123    |
|  2 | rose     | 321    |
|  3 | jeff     | 666    |
|  4 | xiaofei  | 123    |
+----+----------+--------+
```

python程序如下：

 

```
import pymysql

# 连接数据库信息
config = {
    "host":"172.16.1.5",
    "user":"root",
    "password":"123.com",
    "database":"dbforpymysql"
}

# 开始连接数据库
db = pymysql.connect(**config)
cursor = db.cursor()

user = input("username: ").strip()
pwd = input("password: ").strip()

result = cursor.execute("SELECT * FROM userinfo WHERE username=%s and passwd=%s", (user,pwd,))
cursor.close()
db.close()
if result:
    print('登录成功')
else:
    print('登录失败')
```

说明：

在上面的代码中：

 

```
# 如果这里写成这样形式，则会有sql注入风险。
sql = "select * from userinfo where username='%s' and passwd='%s'" %(user,pwd)
result=cursor.execute(sql)

# 执行代码然后输入：
username: xiaofei or 1=1 --
password: 1
登录成功
```

sql支持'--'写法，表示注释的意思。这里--后面的会被注释，所以where一定会成功，这里等于查看了所有行的内容，返回值也不等于0，所以就登录成功了。

解决方法就是将变量或者实参直接写到execute中即可，也就是参数化查询。

 

```
sql = "select * from userinfo where username=%s and passwd=%s"
result=cursor.execute(sql, (user,pwd))
```

参数化查询，可以有效防止注入攻击，并提高查询的效率。

Cursor.execue(query, args=None)

args,必须是元组、列表或字典。如果查询字符串使用%(name)s,就必须使用字典。

提高查询效率原因

SQL语句缓存，数据库一般会对SQL语句编译和缓存，编译只对SQL语句部分，需要词法分析、语法分析、生成AST、优化、生成执行计划等过程，比较耗费资源。

开发时应该使用参数化查询。

注意：这里说的查询字符串的缓存，不是查询结果的缓存。

# 示例

## 用户登录重写

 

```
#!/usr/bin/python
# -*- coding:utf-8 -*-
import pymysql
 
user = input('请输入用户名:')
pwd = input('请输入密码:')
 
# 获取数据
conn = pymysql.Connect(host='192.168.12.89',port=3306,user='root',password="123",database="s17day11db",charset='utf8')
cursor = conn.cursor()
sql = 'select * from userinfo where username="%s" and password="%s" ' %(user,pwd,)
# user = alex" --
# pwd= asdf
'select * from userinfo where username="alex" -- " and password="sdfsdf"'
# user = asdfasdf" or 1=1  --
# pwd= asdf
'select * from userinfo where username="asdfasdf" or 1=1  -- " and password="asdfasdf"'
v = cursor.execute(sql)
result = cursor.fetchone()
cursor.close()
conn.close()
 
print(result)
```

## 练习

 

```
import pymysql
 
user = input('请输入用户名:')
pwd = input('请输入密码:')
 
# 获取数据
conn = pymysql.Connect(host='192.168.12.89',port=3306,user='root',password="123",database="s17day11db",charset='utf8')
cursor = conn.cursor()
 
v = cursor.execute('select * from userinfo where username=%s and password=%s',[user,pwd])
result = cursor.fetchone()
cursor.close()
conn.close()
 
print(result)
```

## 新建数据

 

```
import pymysql
 
# 获取数据
conn = pymysql.Connect(host='192.168.12.89',port=3306,user='root',password="123",database="s17day11db",charset='utf8')
cursor = conn.cursor()
 
cursor.execute('insert into class(caption) values(%s)',['新班级'])
conn.commit()
new_class_id = cursor.lastrowid # 获取新增数据自增ID
 
cursor.execute('insert into student(sname,gender,class_id) values(%s,%s,%s)',['李杰','男',new_class_id])
conn.commit()
 
cursor.close()
conn.close()
```