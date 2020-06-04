[TOC]

概念

数据类型

mongo shell

mongodb基本概念

包括文档,集合,数据库等.

| sql         | mongodb     | 说明                                |
| ----------- | ----------- | ----------------------------------- |
| database    | database    | 数据库                              |
| table       | collection  | 数据库表/集合                       |
| row         | document    | 行/文档                             |
| column      | field       | 字段/域                             |
| index       | index       | 索引                                |
| table joins |             | 连表,mongodb不支持                  |
| primary key | primary key | 主键,mongodb自动将_id字段设置为主键 |

下面使用2张图对比一下关系型数据库和MongoDB数据库的数据存储结构:

关系型数据库数据结构:

![img](Mongodb%E8%BF%9B%E9%98%B6.assets/70de84ac-d582-473f-b3cf-3d50e536950b.jpg)

MongoDB数据结构:

![img](Mongodb%E8%BF%9B%E9%98%B6.assets/5c330de3-bda8-4695-bd0b-65700584a4f1.jpg)

数据类型

MongoDB数据类型

1、String : 这是最常用的数据类型来存储数据。在MongoDB中的字符串必须是有效的UTF-8。

2、Integer : 这种类型是用来存储一个数值。整数可以是32位或64位，这取决于您的服务器。

3、Boolean : 此类型用于存储一个布尔值 (true/ false) 。

4、Double : 这种类型是用来存储浮点值。

5、Min/ Max keys : 这种类型被用来对BSON元素的最低和最高值比较。

6、Arrays : 使用此类型的数组或列表或多个值存储到一个键。

7、Timestamp : 时间戳。这可以方便记录时的文件已被修改或添加。

8、Object : 此数据类型用于嵌入式的文件。

9、Null : 这种类型是用来存储一个Null值。

10、Symbol : 此数据类型用于字符串相同，但它通常是保留给特定符号类型的语言使用。

11、Date : 此数据类型用于存储当前日期或时间的UNIX时间格式。可以指定自己的日期和时间，日期和年，月，日到创建对象。

12、Object ID : 此数据类型用于存储文档的ID。

13、Binary data : 此数据类型用于存储二进制数据。

14、Code : 此数据类型用于存储到文档中的JavaScript代码。

15、Regular expression : 此数据类型用于存储正则表达式。

基本命令

1,服务端与客户端命令

mongod options # 服务端命令

mongo  # 客户端命令

usage: mongo [options] [db address] [file names (ending in .js)]

2,CRUD操作
Create,Read,Update,Delete

help

列出mongodb支持的操作

 

```
    db.help()                    help on db methods  # db对象支持的操作方法
    db.mycoll.help()             help on collection methods  # 集合支持的操作方法
    sh.help()                    sharding helpers  # 
    rs.help()                    replica set helpers
    help admin                   administrative help
    help connect                 connecting to a db help
    help keys                    key shortcuts
    help misc                    misc things to know
    help mr                      mapreduce

    show dbs                     show database names
    show collections             show collections in current database
    show users                   show users in current database
    show profile                 show most recent system.profile entries with time >= 1ms
    show logs                    show the accessible logger names
    show log [name]              prints out the last segment of log in memory, 'global' is default
    use <db_name>                set current database
    db.foo.find()                list objects in collection foo
    db.foo.find( { a : 1 } )     list objects in foo where a == 1
    it                           result of the last line evaluated; use to further iterate
    DBQuery.shellBatchSize = x   set default number of items to display on shell
    exit                         quit the mongo shell
```

 

```
    db.help()                    help on db methods  # db对象支持的操作方法
    db.mycoll.help()             help on collection methods  # 集合支持的操作方法
    sh.help()                    sharding helpers  # 
    rs.help()                    replica set helpers
    help admin                   administrative help
    help connect                 connecting to a db help
    help keys                    key shortcuts
    help misc                    misc things to know
    help mr                      mapreduce
    show dbs                     show database names
    show collections             show collections in current database
    show users                   show users in current database
    show profile                 show most recent system.profile entries with time >= 1ms
    show logs                    show the accessible logger names
    show log [name]              prints out the last segment of log in memory, 'global' is default
    use <db_name>                set current database
    db.foo.find()                list objects in collection foo
    db.foo.find( { a : 1 } )     list objects in foo where a == 1
    it                           result of the last line evaluated; use to further iterate
    DBQuery.shellBatchSize = x   set default number of items to display on shell
    exit                         quit the mongo shell
```

db.listCommands(): 列举数据库命令

db.foo.update: 查看update函数方法的实现或定义,不带括号

用户管理

MongoDB用户角色：

read: 允许用户读取指定数据库

readWrite: 允许用户读写指定数据库

dbAdmin: 允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile

userAdmin: 允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户

clusterAdmin: 只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限

readAnyDatabase: 只在admin数据库中可用，赋予用户所有数据库的读权限

readWriteAnyDatabase: 只在admin数据库中可用，赋予用户所有数据库的读写权限

userAdminAnyDatabase: 只在admin数据库中可用，赋予用户所有数据库的userAdmin权限

dbAdminAnyDatabase: 只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限

root: 只在admin数据库中可用。超级账号，超级权限

 

```
# 创建用户
use admin # 如果此库不存在,则默认创建
db.createUser({user:"admin",customData:{description:"superuser"},pwd:"123.com",roles:[{role:"root",db:"admin"}]})

# 列出所有用户
db.system.users.find()

# 查看当前库下所有用户
show users

# 删除用户
db.dropUser("admin")
show users

# 开启验证,需编辑启动脚本文件
vim /usr/lib/systemd/system/mongod.service
"""
Environment="OPTIONS=--auth -f /etc/mongod.conf"
"""

# 重启服务并使用admin登录
systemctl daemon-reload
systemctl restart mongod
mongo -u"admin" -p"123.com" --authenticationDatabase "admin" --host 172.16.2.15 --port 27017
```

 

```
# 创建用户
use admin # 如果此库不存在,则默认创建
db.createUser({user:"admin",customData:{description:"superuser"},pwd:"123.com",roles:[{role:"root",db:"admin"}]})
# 列出所有用户
db.system.users.find()
# 查看当前库下所有用户
show users
# 删除用户
db.dropUser("admin")
show users
# 开启验证,需编辑启动脚本文件
vim /usr/lib/systemd/system/mongod.service
"""
Environment="OPTIONS=--auth -f /etc/mongod.conf"
"""
# 重启服务并使用admin登录
systemctl daemon-reload
systemctl restart mongod
mongo -u"admin" -p"123.com" --authenticationDatabase "admin" --host 172.16.2.15 --port 27017
```

CRUD

1,创建或修改数据

db.createCollection("user1",{"name":"xiaofei","age":18}})  # 当前库创建集合数据

db.mydb.insert({key1:val1,key2:val2,key3:{key4:val4,key5:val5}})  # 向mydb库中插入数据

db.storeCollection.save({"key1:"key2."key2":"val2"})  # 当前库中插入集合数据,存在则覆盖

db.mycoll.update(criteria,objNew,upsert,multi)

说明:

criteria:update的查询条件，类似sql update查询内where后面的

objNew:update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的

upsert:这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。

multi:mongodb默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。

示例:

 

```
db.test0.update( { "count" : { $gt : 1 } } , { $set : { "test2" : "OK"} } ); 只更新了第一条记录
db.test0.update( { "count" : { $gt : 3 } } , { $set : { "test2" : "OK"} },false,true ); 全更新了
db.test0.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false ); 只加进去了第一条
db.test0.update( { "count" : { $gt : 5 } } , { $set : { "test5" : "OK"} },true,true ); 全加进去了
db.test0.update( { "count" : { $gt : 15 } } , { $inc : { "count" : 1} },false,true );全更新了
db.test0.update( { "count" : { $gt : 10 } } , { $inc : { "count" : 1} },false,false );只更新了第一条
```

 

```
db.test0.update( { "count" : { $gt : 1 } } , { $set : { "test2" : "OK"} } ); 只更新了第一条记录
db.test0.update( { "count" : { $gt : 3 } } , { $set : { "test2" : "OK"} },false,true ); 全更新了
db.test0.update( { "count" : { $gt : 4 } } , { $set : { "test5" : "OK"} },true,false ); 只加进去了第一条
db.test0.update( { "count" : { $gt : 5 } } , { $set : { "test5" : "OK"} },true,true ); 全加进去了
db.test0.update( { "count" : { $gt : 15 } } , { $inc : { "count" : 1} },false,true );全更新了
db.test0.update( { "count" : { $gt : 10 } } , { $inc : { "count" : 1} },false,false );只更新了第一条
```

2,查找数据

1,show dbs  # 查看当前所有数据库

use mydb  # 切换到指定数据库,没有则自动创建

showcollections  # 查看当前库中的所有集合

db.serverStatus()  # 查看服务器状态

db.stats()  # 查看当前数据库信息

db.getCollectionNames()  # 查看当前库中所有的集合名字

find

db.mydb.find(condition).actions  # 查找显示所有符合的数据,类似select * from mydb;

db.mydb.findone(condition).actions  # 查找显示第一条符合的数据

查找条件condition:

1,指定条件

db.mycoll.find({'name':'xiaofie'},{'user':1,'age':1})  # 类似select user,age from mycoll where name='xiaofei';

第一个{}放置where条件,第二个{}指定是否显示列字段,0不显示,1显示.

2,比较条件:$gt,$gte,$lt,$lte,$ne,$in,$nin

格式: find({field:{$gt:value}})

示例:查找大于等于18小于等于28的所有数据

db.mycoll.find({'age':{'$gte':18,'$lte':28}})

\# 类似于select age from mycoll where age>=18 and age<=28;

3,逻辑条件:$or,$and,$not

格式:find({$or:[{条件1},{条件2}]})

示例:

db.mycoll.find({$or:[{'name':'xiaofei'},{'age':18}]})

\# 类似于select user,age,id from mycoll where name='xiaofei' or age=18;

4,存在性判断: $exists

5,类型判断: $type

find查找结果的处理动作actions

.limit()

.count()  # 统计结果数

.skip()  # 跳过

示例:

db.storeCollection.find({"name":"xiaofei"}).count()

db.storeCollection.findone({"name":"xiaofie"})

3,删除数据

db.dropDatabase()  # 删除数据库

db.mydb.drop()  # 删除mydb库内的所有集合

db.storeCollection.remove(condition)  # 根据条件删除集合内数据

db.mydb.remove({"name":"xiaofei"})

索引操作

索引类型:

B+ Tree, hash

db.mycoll.ensureIndex({"title":1,"url":-1})  # 当前库中的mycoll集合内创建索引,ensureIndex参数中,1为升序,-1为降序

db.mycoll.getIndexes()  # 查询当前库中的mycoll集合内的索引

db.system.indexes.find()  # 查看所有的索引,包括其他的库

db.mycoll.dropIndex(name)  # 删除指定索引

db.mycoll.dropIndexes()  # 删除所有索引

db.mycoll.reIndex()  # 索引重建