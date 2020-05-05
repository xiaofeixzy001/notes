[TOC]

# 项目构建

新建项目为pipeline，按需开启虚拟环境。

在pipeline项目下，创建包pipeline；

在pipeline包下，创建model.py和config.py文件，model.py文件用于ORM映射，config.py文件写全局配置信息。



## config.py

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# __Author__: xiaofei

USER_NAME = 'root'
HOST = '192.168.100.10'
PORT = '3306'
PASSWORD = '123.com'
DATABASE = 'pipeline'

DATABASE_DEBUG = True

URL = 'mysql+pymysql://{}:{}@{}:{}/{}'.format(
    USER_NAME,
    PASSWORD,
    HOST,
    PORT,
    DATABASE
)
```



## model.py

安装sqlalchemy和pymysql模块

`pip install sqlalchemy pymysql`

model.py文件如下：

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# __Author__: XiaoFei

from sqlalchemy import Column, Integer, String, Text, ForeignKey, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship, sessionmaker
from .config import URL, DATABASE_DEBUG
from functools import wraps

STATE_WAITING =0
STATE_RUNNING=1
STATE_SUCCEED=2
STATE_FAILED=3
STATE_FINISH=4

Base = declarative_base()


class Graph(Base):

    __tablename__ = 'graph'

    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(48), nullable=False, unique=True)
    desc = Column(String(200), nullable=True)
    checked = Column(Integer, nullable=False, default=0)
    sealed = Column(Integer, nullable=False, default=0)

    def __repr__(self):
        return '<Graph {} {}>'.format(self.id, self.name)

    __str__ = __repr__


class Vertex(Base):

    __tablename__ = 'vertex'

    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(48), nullable=False)
    script = Column(Text, nullable=True)
    input = Column(Text, nullable=True)
    g_id = Column(Integer, ForeignKey('graph.id'), nullable=False)

    def __repr__(self):
        return '<Vertex {} {}>'.format(self.id, self.name)

    __str__ = __repr__


class Edge(Base):

    __tablename__ = 'edge'

    id = Column(Integer, primary_key=True, autoincrement=True)
    tail = Column(Integer, ForeignKey('vertex.id'), nullable=False)
    head = Column(Integer, ForeignKey('vertex.id'), nullable=False)
    g_id = Column(Integer, ForeignKey('graph.id'), nullable=False)

    def __repr__(self):
        return '< Edge {} <{} {}> >'.format(self.id, self.tail, self.head)

    __str__ = __repr__


class Pipeline(Base):

    __tablename__ = 'pipeline'

    id = Column(Integer, primary_key=True, autoincrement=True)
    g_id = Column(Integer, ForeignKey('graph.id'), nullable=False)
    current = Column(Integer, ForeignKey('vertex.id'), nullable=False)
    state = Column(Integer, nullable=False, default=STATE_WAITING)

    def __repr__(self):
        return '<Pipeline {} >'.format(self.id)

    __str__ = __repr__


class Track(Base):

    __tablename__ = 'track'

    id = Column(Integer, primary_key=True, autoincrement=True)
    p_id = Column(Integer, ForeignKey('pipeline.id'), nullable=False)
    v_id = Column(Integer, ForeignKey('vertex.id'), nullable=False)
    input = Column(Text, nullable=True)
    output = Column(Text, nullable=True)
    state = Column(Integer, nullable=False, default=STATE_WAITING)

    def __repr__(self):
        return '<Track {} >'.format(self.id)

    __str__ = __repr__


def singleton(cls):
    instance = None
    @wraps(cls)
    def wrapper(*args, **kwargs):
        nonlocal instance
        if not instance:
            instance = cls(*args, **kwargs)
        return instance
    return wrapper


@singleton
class Database:
    def __init__(self, url, **kwargs):
        self._engine = create_engine(url, **kwargs)
        self._session = sessionmaker(bind=self._engine)()

    @property
    def session(self):
        return self._session

    @property
    def engine(self):
        return self._engine

    def create_all(self):
        Base.metadata.create_all(self._engine)

    def drop_all(self):
        Base.metadata.drop_all(self._engine)

    def __repr__(self):
        return "<{} {} {}>".format(self.__class__.__name__, id(self), self.__dict__)


db = Database(URL, echo=DATABASE_DEBUG)

```

































