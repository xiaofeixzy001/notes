[TOC]

# 简介

SQLAlchemy是一个ORM框架，大量使用了元编程。学习之前建议复习一下元编程。

## 官方文档

https://docs.sqlalchemy.org/en/13/

## 需要安装

pip install sqlalchemy

## 查看版本

import sqlalchemy

print(sqlalchemy.\_\_version\_\_)

# 使用

sqlalchemy内部使用了连接池

## 连接数据库

参见：https://docs.sqlalchemy.org/en/13/core/engines.html#database-urls 

mysqldb的连接

mysql+mysqldb://<user>:<pwd>@<host>:<port>/<dbname>

pymsqyl的连接

mysql+pymysql://<user>:<pwd>@<host>:<port>/<dbname>?<options>

示例：

```python
engine = sqlalchemy.create_engine("mysql+pymysql://root:123.com@192.168.100.5:3306/students", echo=True)
```

echo=True: 表示引擎是否打印执行语句,用于调试。

Column

## 创建映射

Declare a Mapping

students表

CREATE TABLE students (id INTEGER NO NULL AUTO_INCREMENT, name VARCHAR(64) NOT NULL, age INTEGER, PRIMARY KEY (id))

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, Date, Enum

engine_name = 'mysql+pymysql'
host = '192.168.100.5'
user = 'root'
pwd = '123.com'
port = 3306
database = 'school'

conn_str = '{}://{}:{}@{}:{}/{}'.format(engine_name, user, pwd, host, port, database)

# 创建引擎,用于连接数据库
engine = create_engine(conn_str, echo=True)

# 创建基类,便于实体类继承
Base = declarative_base()

class Students(Base):
    """
    model类,实体类
    """
    __tablename__ = 'students'  # 指定表名

    # 定义属性对应表的字段,第一参数是指定属性名,对应表字段名,如果不一致必须要指定
    id = Column('id', Integer, primary_key=True, autoincrement=True)
    name = Column('name', String(64), nullable=False)
    age = Column(Integer)

    def __repr__(self):
        return "< {}: id={} name={} age={} >".format(self.__class__.__name__, self.id, self.name, self.age)

# 实例化
s1 = Students()
s1.age = 20
s1.name = 'tom'

s2 = Students(id=5, name='jerry')
s2.age = 18

print(s1)
print(s2)

# Base.metadata.create_all(bind=engine, )  # 创建继承自Base的所有表,写入数据库
Base.metadata.drop_all(bind=engine, )  # 删除所有表
```

\_\_tablename\_\_: 指定表名。

Column类指定对应的字段,必须指定。

生产环境中很少这样创建表,都是系统上线时由脚本生成,而删除,宁可废弃不用也不要轻易删除。

## 会话session

在一个会话中操作数据库,会话建立在连接上,连接被引擎管理。

创建session

```python
from sqlalchemy.orm import sessionmaker
Session = sessionmaker(bing=engine)  # return class
session = Session()  # 实例化
```

session对象线程不安全，所以不同的线程使用不同的session对象。

Session类和engine都是线程安全的，有一个就行了。

## CRUD操作

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, Date, Enum
from sqlalchemy.orm import sessionmaker

engine_name = 'mysql+pymysql'
host = '192.168.100.5'
user = 'root'
pwd = '123.com'
port = 3306
database = 'school'

conn_str = '{}://{}:{}@{}:{}/{}'.format(engine_name, user, pwd, host, port, database)

# 创建引擎,用于连接数据库
engine = create_engine(conn_str, echo=True)

# 创建基类,便于实体类继承
Base = declarative_base()

class Students(Base):
    """
    model类,实体类
    """
    __tablename__ = 'students'  # 指定表名

    # 定义属性对应表的字段,第一参数是指定属性名,对应表字段名,如果不一致必须要指定
    id = Column('id', Integer, primary_key=True, autoincrement=True)
    name = Column('name', String(64), nullable=False)
    age = Column(Integer)

    def __repr__(self):
        return "< {}: id={} name={} age={} >".format(self.__class__.__name__, self.id, self.name, self.age)
```

### 增

```python
Session = sessionmaker(bind=engine)
session = Session()
s1 = Students()
s1.age = 20
s1.name = 'tom'
s2 = Students(id=5, name='jerry')
s2.age = 18
session.add(s1)
session.add(s2)
session.commit()
print('~~~~~~~')
try:
    session.commit()
except Exception as e:
    print(e)
    session.rollback()
```

s实例化时一般不会指定id,因为id是自增的。

### 查

使用query()方法,返回一个Query对象

```python
Session = sessionmaker(bind=engine)  # Session类,会话Session类
session = Session()  # session实例
query = session.query(Students)  # select * from students
print(type(query))

# 拿所有
for student in query:
    print(student)
    print(type(student))
print('~~~~~~')

# 通过主键查询
student = session.query(Student).get(10)
student.age = 30
session.add(student)
try:
    session.commit()
except Exception as e:
    print(e)
    session.rollback()
print(student)
```

查不到数据返回None

query方法将实体类传入，返回类的对象可迭代对象，这时候并不查询。迭代它就执行SQL来查询数据库，封装数据到指定类的实例。

get方法使用主键查询，返回一条传入类的实例。

先查后改,会使用update,如果不查直接改则会使用insert。

### 改

先查,在修改,最后提交修改。

```python
student = session.query(Students).get(2)
print(student)
student.name = "sam"
student.age = 20
print(student)
session.add(student)
session.commit()
```

改之前先查一下是否有数据是否是持久化的，改的时候就会变成update语句,否则会是insert语句。

### 删

删除数据前提，确保数据是持久的，也就是存在的。

删除之前会先到数据库中查找数据，如果不存在，那么删时会报错。

```python
Session = sessionmaker(bind=engine)
session = Session()
student = Student(id=2, name='sam', age=30)
session.delete(student)
try:
    session.commit()
except Exception as e:
    session.rollback()
    print("-" * 30)
    print(e)

"""
------------------------------
Instance '<Student at 0x25f7a14a780>' is not persisted
"""
```

所以需要先确保删除的数据存在

```python
Session = sessionmaker(bind=engine)
session = Session()
student = session.query(Student).get(2)
print(student)
session.delete(student)
try:
    session.commit()
except Exception as e:
    session.rollback()
    print("-" * 30)
    print(e)
```



## 状态

每一个实体，都有一个状态：属性_sa_instance_state，其类型是sqlalchemy.orm.state.instanceState,可以使用sqlalchemy.inspect(entity)函数查看状态。

常见状态有transient, pending, persistent, deleted, detached.

transient: 实体类尚未加入到session中，同时并没有保存到数据库中。

pending: transient的实体被add()到session中，状态切换到pending，但它还没有flush到数据库中。

persistent: session中的实体对象对应着数据库中的真实记录。pending状态在提交成功后可以变成persistent状态，或者查询成功返回的实体也是persistent状态。

deleted: 实体被删除且已经flush但未commit完成。事务提交成功了，实体变成detached，事务失败，返回persistent状态。

detached: 删除成功的实体进入这个状态。

新建一个实体，状态是transient临时的；

一旦add()后从transient变成pending状态；

成功commit()后从pending变成persistent状态；

成功查询返回的实体对象，也是persistent状态；

persistent状态的实体，修改依然是persistent状态；

persistent状态的实体，删除后，flush但未commit，变成deleted状态，commit成功，变为detached状态，commit失败还原到persisttent状态。flush方法，主动把改变应用到数据库中去。

删除、修改操作，需要对应一个真实的记录，所以要求实体对象是persistent状态。

示例：

```python
import sqlalchemy
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, Date, Enum
from sqlalchemy.orm import sessionmaker
from sqlalchemy.orm.state import InstanceState

conns_tr = '{}://{}:{}@{}:{}/{}'.format(
    'mysql+pymysql',
    'root', '123.com',
    '192.168.100.5', 3306,
    'school'
)

# 创建引擎
engine = create_engine(conns_tr, echo=True)  # 连接

# 创建基类
Base = declarative_base()  # 基类,元类

class Students(Base):
    """
    model类,实体类
    """
    __tablename__ = 'student'  # 指定表名

    # 指定字段
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(64), nullable=False)
    age = Column(Integer)

    def __repr__(self):
        return "< student {} {} {} >".format(self.id, self.name, self.age)

Session = sessionmaker(bind=engine)  # Session类,会话Session类
session = Session()  # session实例

def getstate(entify, i):
    """
    查看状态
    :param entify:
    :param i:
    :return:
    """
    insp = sqlalchemy.inspect(entify)
    state = "sessionid={}, attached={}, transient={}, persistent={}, pending={}, deleted={}, detached={}".format(
        insp.session_id,
        insp._attached,
        insp.transient,
        insp.persistent,
        insp.pending,
        insp.deleted,
        insp.detached
    )
    print(i, state)
    print("insp.key: {}".format(insp.key))
    print("-" * 30)

print("-" * 30)
student = session.query(Students).get(1)
getstate(student, 1)  # persistent
```

查一个id,确保它存在。

```python
# 创建
try:
    student = Students(id=2, name='caesar', age=30)
    getstate(student, 2)  # transit
    student = Students(name='jack', age=20)
    getstate(student, 3)  # transit
    session.add(student)
    getstate(student, 4)  # pending 准备创建
    # session.delete(student)  # 删除的前提是persistent,否则抛异常
    # getstate(student, 5)
    session.commit()
    getstate(student, 6)  # persistent  已创建
except Exception as e:
    session.rollback()
    print('~~~~~~~~')
    print(e)

# 删除
try:
    session.delete(student)  # 删除的前提是persistent,否则抛异常
    getstate(student, 11)  # persistent
    session.flush()
    getstate(student, 12)  # deleted  准备删除
    session.commit()
    getstate(student, 13)  # detached  已删除
except Exception as e:
    session.rollback()
    print('~~~~~~~~~')
    print(e)
```



## 复杂查询

使用3张新表为例，顺带整体回顾数据库操作流程

1 连接数据库

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

conns_tr = '{}://{}:{}@{}:{}/{}'.format(
    'mysql+pymysql',
    'root', '123.com',
    '192.168.100.5', 3306,
    # 'school'  # 空库,用于测试
    'test'
)

engine = create_engine(conn_str, echo=True)  # 打印执行语句
Base = declarative_base()
Session = sessionmaker(bind=engine)
session = Session()
```

2 创建映射关系

```python
from sqlalchemy import Column, Integer, String, Date, Enum, ForeignKey

import enum
class MyEnum(enum.Enum):
    M = 'M'
    F = 'F'

class Employee(Base):
    # 指定表名,对应数据库真实的表
    __tablename__ = "employees"
    
    # 定义属性对应的字段
    emp_no = Column(Integer, primary_key=True)
    birth_date = Column(Date, nullable=False)
    first_name = Column(String(14), nullable=False)
    last_name = Column(String(16), nullable=False)
    gender = Column(Enum(MyEnum), nullable=False)
    hire_date = Column(Date, nullable=False)
    
    dept_emps = relationship('Dept_emp')  # 用于存放部门信息,解决join连表查询结果只有一个问题
    
    def __str__(self):
        return "<{} no={} first_name={} last_name={} gender={} no={}>".format(
            self.__class__.__name__, self.emp_no, self.first_name, self.last_name, self.gender.value, self.dept_emps
        )

class Department(Base):
    __tablename__ = "departments"
    dept_no = Column(String(4), primary_key=True)
    dept_name = Column(String(40), nullable=False, unique=True)

    def __repr__(self):
        return "<{} no={} name={}>".format(
            type(self.__name__, self.dept_no, self.dept_name)
        )

class Dept_emp(Base):
    __tablename__ = "dept_emp"
    emp_no = Column(Integer, ForeignKey('employees.emp_no', ondelete='CASCADE'), primary_key=True)
    dept_no = Column(String(4), ForeignKey('departments.dept_no', ondelete='CASCADE'), primary_key=True)
    from_date = Column(Date, nullable=False)
    to_date = Column(Date, nullable=False)

    def __repr__(self):
        return "< {} empno={} deptno={} >".format(type(self).__name__, self.emp_no, self.dept_no)

Base.metadata.create_all(bind=engine)
```

3，复查查询

```python
def show(emps):
    # 打印函数
    for x in emps:
        print(x)
    print('~~~~~~~~~', end='\n\n')

# where对应filter
query = session.query(Employee).filter(Employee.emp_no > 10016)
show(query)

# and与
from sqlalchemy import and_, or_, not_
query1 = session.query(Employee).filter(Employee.emp_no > 10016).filter(Employee.emp_no < 10019)
query2 = session.query(Employee).filter((Employee.emp_no > 10016) & (Employee.emp_no < 10019))  # 注意&符号两边加括号
query3 = session.query(Employee).filter(and_(Employee.emp_no > 10016, Employee.emp_no < 10019))
show(query1)
show(query2)
show(query3)

# or或
query1 = session.query(Employee).filter((Employee.emp_no < 10003) | (Employee.emp_no > 10018))  # 注意&符号两边加括号
query2 = session.query(Employee).filter(or_(Employee.emp_no < 10003, Employee.emp_no > 10018))
show(query1)
show(query2)

# not非
query1 = session.query(Employee).filter(~(Employee.emp_no < 10018))
query2 = session.query(Employee).filter(not_(Employee.emp_no < 10018))
show(query1)
show(query2)

# in,not in, 后跟可迭代对象,字段上的方法
emplist = [10010, 10005, 10018]
query1 = session.query(Employee).filter(Employee.emp_no.in_(emplist))
query2 = session.query(Employee).filter(~(Employee.emp_no.in_(emplist)))
show(query1)
show(query2)

# like, ilike(i:ignore,忽略大小写)
query1 = session.query(Employee).filter(Employee.last_name.like('P%'))
query2 = session.query(Employee).filter(Employee.last_name.ilike('P%'))
show(query1)
show(query2)

# 排序asc()升序,desc()降序,默认升序
query1 = session.query(Employee).filter(Employee.emp_no > 10010).order_by(Employee.emp_no.desc())
query2 = session.query(Employee).filter(Employee.last_name.like('P%')).order_by(Employee.emp_no.asc())
query3 = session.query(Employee).filter(Employee.emp_no > 10010).order_by(Employee.last_name).order_by(Employee.emp_no.desc())
show(query1)
show(query2)
show(query3)

# 分页
query1 = session.query(Employee).limit(4)
query2 = session.query(Employee).filter(Employee.last_name.like('P%')).order_by(Employee.emp_no.asc()).limit(4).offset(3)
show(query1)
show(query2)
```

4, 消费者方法

方法调用后,Query对象(可迭代)就转换成了一个容器。

```python
query = session.query(Employee).filter(Employee.last_name.like('T%')).order_by(Employee.emp_no.desc())
print(query)
print("-" * 30)
print(query.all())  # 列表,所有结果
print("-" * 30)
print(query.first())  # 取结果第一个,没有返回None
print("-" * 30)
print(query.one())  # 结果必须有且只有一个才取回来,否则报错
print("-" * 30)
print(query.count())  # 子查询,聚合函数count(*)的查询,结果只有一个
print("-" * 30)
print(len(query.all()))  # 获取所有结果,转换list,不建议

# 删除
session.query(Employee).filter(Employee.emp_no > 10018).delete()
session.commit()
```

5, 聚合和分组查询

```python
from sqlalchemy import func

# count
query = session.query(func.count(Employee.emp_no))
print(query.all())
print(query.first())
print(query.one())  # 只能有一行结果
print(query.scalar())  # 取one()返回元组的第一个元素

# max/min/avg
query1 = session.query(func.max(Employee.emp_no)).scalar()
query2 = session.query(func.min(Employee.emp_no)).scalar()
query3 = session.query(func.avg(Employee.emp_no)).scalar()
print(query1)
print(query2)
print(query3)

# 分组,按性别分组查询,统计每个性别的人数
query = session.query(Employee.gender, func.count(Employee.emp_no)).group_by(Employee.gender).all()
print(query)
```

6, 关联查询

ForeignKey("employee.emp_no", ondelete="CASCADE")  定义外键约束,ondelete="CASCADE"是否同步删除

查询10010员工所属的部门编号

```python
query1 = session.query(Employee, Dept_emp).filter(Employee.emp_no == 10010).filter(Employee.emp_no == Dept_emp.emp_no)
query2 = session.query(Employee).join(Dept_emp).filter(Employee.emp_no == 10010)
query3 = session.query(Employee).join(Dept_emp, Employee.emp_no == Dept_emp.emp_no).filter(Employee.emp_no == 10010)

print(1, query1.all())
# [(<__main__.Employee object at 0x0000024F7AA003C8>, < Dept_emp d004 10010 >), (<__main__.Employee object at 0x0000024F7AA003C8>, < Dept_emp d006 10010 >)]

print(2, query2.all())
print(3, query3.all())
# [<__main__.Employee object at 0x000001CC4B3F1400>]
```

使用join查询的两种方式，返回结果只有一行数据,原因在于query(Employee)这个只能返回一个实体对象中去，为了解决这个问题，需要修改实体类Employee,增加属性用来存放部门信息。

sqlalchemy.orm.relationship(实体类名字的字符串)

```python
class Employee(Base):
    # 指定表名,对应数据库真实的表
    __tablename__ = "employees"
    
    # 定义属性对应的字段
    emp_no = Column(Integer, primary_key=True)
    birth_date = Column(Date, nullable=False)
    first_name = Column(String(14), nullable=False)
    last_name = Column(String(16), nullable=False)
    gender = Column(Enum(MyEnum), nullable=False)
    hire_date = Column(Date, nullable=False)
    
    dept_emps = relationship('Dept_emp')  # 用于存放部门信息,解决join连表查询结果只有一个问题
    
    def __str__(self):
        return "<{} no={} first_name={} last_name={} gender={} no={}>".format(
            self.__class__.__name__, self.emp_no, self.first_name, self.last_name, self.gender.value, self.dept_emps
        )

results1 = session.query(Employee).join(Dept_emp).filter(Employee.emp_no == Dept_emp.emp_no).filter(Employee.emp_no == 10010)
results2 = session.query(Employee).join(Dept_emp, Employee.emp_no == Dept_emp.emp_no).filter(Employee.emp_no == 10010)
results3 = session.query(Employee).join(Dept_emp, (Employee.emp_no == Dept_emp.emp_no) & (Employee.emp_no == 10010))
show(results1.all())
show(results2.all())
show(results3.all())
# 观察三种方式对应的SQL语句
```

上面三种方式，results1中join(Dept_emp)没有等值条件，会自动生成一个等值条件，如果后面又filter，哪怕是filter(Employee.emp_no == Dept_emp.emp_no)，这个条件会在where中出现。第一种这种自动增加join的等值条件方式不好，不推荐。

results2在join中增加等值条件，组织了自动生成，推荐使用。

results3同results2。

# 总结

在开发中一般都会采用ORM框架，这样就可以使用对象操作表了。

定义表映射的类，使用Column的描述器定义类属性，使用ForeignKey定义外键约束。

如果再一个对象中，想查看其他表对应的对象的内容，就要使用relationship来定义关系。

至于是否使用外键，根据实际情况来定。