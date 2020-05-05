[TOC]

# DAG定义

service层需求：

1，定义流程DAG，即Schema定义。

2，执行某一个DAG的流程。

## 代码实现

新建service.py文件。

```python
from .model import Graph, Vertex, Edge, Pipeline, Track, db


# 创建DAG
def create_graph(name, desc=None):
    g = Graph()
    g.name = name
    g.desc = desc
    db.session.add(g)
    try:
        db.session.commit()
        return g
    except:
        db.session.rollback()


# 添加顶点
def add_vertex(graph: Graph, name: str, input=None, script=None):
    v = Vertex()
    v.g_id = graph.id
    v.name = name
    v.input = input
    v.script = script
    db.session.add(v)
    try:
        db.session.commit()
        return v
    except:
        db.session.rollback()


# 添加边
def add_edge(graph: Graph, tail: Vertex, head: Vertex):
    e = Edge()
    e.g_id = graph.id
    e.tail = tail.id
    e.head = head.id
    db.session.add(e)
    try:
        db.session.commit()
        return e
    except:
        db.session.rollback()

# 删除顶点
def del_vertex(id):
    query = db.session.query(Vertex).filter(Vertex.id == id)
    v = query.first()
    if v:
        try:
            # 找到顶点所关联的边
            db.session.query(Edge).filter((Edge.tail == v.id) | (Edge.head == v.id)).delete()
            query.delete()
            db.session.commit()
        except:
            db.session.rollback()
    return v


# 删除边
def del_edge(id):
    query = db.session.query(Edge).filter(Edge.id == id)
    e = query.first()
    if e:
        try:
            query.delete()
            db.session.commit()
        except:
            db.session.rollback()
    return e
```

这里是真删除，实际上一般都是假删除。

通过上面代码，可发现事务处理代码差不多，提出来使用装饰器。

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# __Author__: xiaofei

from .model import Graph, Vertex, Edge, Pipeline, Track, db
from functools import wraps

def transactional(func):
    @wraps
    def wrapper(*args, **kwargs):
        ret = func(*args, **kwargs)
        try:
            db.session.commit()
            return ret
        except Exception as e:
            print(e)
            db.session.rollback()
    return wrapper


# 创建DAG
@transactional
def create_graph(name, desc=None):
    g = Graph()
    g.name = name
    g.desc = desc
    db.session.add(g)
    return g


# 添加顶点
@transactional
def add_vertex(graph: Graph, name: str, input=None, script=None):
    v = Vertex()
    v.g_id = graph.id
    v.name = name
    v.input = input
    v.script = script
    db.session.add(v)
    return v


# 添加边
@transactional
def add_edge(graph: Graph, tail: Vertex, head: Vertex):
    e = Edge()
    e.g_id = graph.id
    e.tail = tail.id
    e.head = head.id
    db.session.add(e)
    return e

# 删除顶点
@transactional
def del_vertex(id):
    query = db.session.query(Vertex).filter(Vertex.id == id)
    v = query.first()
    if v:
        # 找到顶点所关联的边
        db.session.query(Edge).filter((Edge.tail == v.id) | (Edge.head == v.id)).delete()
        query.delete()
    return v


# 删除边
@transactional
def del_edge(id):
    query = db.session.query(Edge).filter(Edge.id == id)
    e = query.first()
    if e:
        query.delete()
    return e
```



## 测试数据

app.py

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# __Author__: xiaofei
import json
from pipeline.model import Database, db
from pipeline.config import URL, DATABASE_DEBUG
from pipeline.service import create_graph, add_edge, add_vertex, del_edge, del_vertex


def test_create_dag():
    try:
        # 创建DAG
        g = create_graph('test1')
        input = {
            'ip': {
                'type': 'str',
                'required': True,
                'default': '192.168.100.10'
            }
        }

        script = {
            'script': 'echo "test1.A"\nping {ip}',
            'next': 'B'
        }

        # 增加顶点
        a = add_vertex(g, 'A', json.dumps(input), json.dumps(script))
        b = add_vertex(g, 'B', None, 'echo B')
        c= add_vertex(g, 'C', None, 'echo C')
        d= add_vertex(g, 'D', None, 'echo D')

        # 增加边
        ab = add_edge(g, a, b)
        ac = add_edge(g, a, c)
        cb = add_edge(g, c, b)
        bd = add_edge(g, b, d)

        # 创建DAG,环路
        g = create_graph('test2')

        # 增加顶点
        a = add_vertex(g, 'A', None, 'echo A')
        b = add_vertex(g, 'B', None, 'echo B')
        c = add_vertex(g, 'C', None, 'echo C')
        d = add_vertex(g, 'D', None, 'echo D')

        # 增加边
        ba = add_edge(g, b, a)
        ac = add_edge(g, a, c)
        cb = add_edge(g, c, b)
        bd = add_edge(g, b, d)

        # 创建DAG,多终点
        g = create_graph('test3')

        # 增加顶点
        a = add_vertex(g, 'A', None, 'echo A')
        b = add_vertex(g, 'B', None, 'echo B')
        c = add_vertex(g, 'C', None, 'echo C')
        d = add_vertex(g, 'D', None, 'echo D')

        # 增加边
        ba = add_edge(g, b, a)
        ac = add_edge(g, a, c)
        bc = add_edge(g, b, c)
        bd = add_edge(g, b, d)

        # 创建DAG,多入口
        g = create_graph('test4')

        # 增加顶点
        a = add_vertex(g, 'A', None, 'echo A')
        b = add_vertex(g, 'B', None, 'echo B')
        c = add_vertex(g, 'C', None, 'echo C')
        d = add_vertex(g, 'D', None, 'echo D')

        # 增加边
        ab = add_edge(g, a, b)
        ac = add_edge(g, a, c)
        cb = add_edge(g, c, b)
        db = add_edge(g, d, b)

    except Exception as e:
        print(e)


# db.drop_all()
# db.create_all()
test_create_dag()

```



# DAG检测算法

回顾一下图论的知识。

## DFS算法

Depth First Search，深度优先遍历，递归算法。

需要改进算法以适用于有向图，不能直接检测有向图是否有环。

## 拓扑排序算法

拓扑排序算法就是把有向图中的顶点以线性方式排序，如果有弧<u, v>，则最后线性排序的结果，顶点u总是在顶点v的前面。

一个有向图能被拓扑排序的充要条件式：它必须是DAG。

## kahn算法

1，选择一个入度为0的顶点并输出它；

2，删除以此顶点为弧尾的弧；

重复上面2步，知道输出全部顶点为止，或者图中不存在入度为0的顶点为止。

如果输出了全部顶点，就是DAG。

 ## DAG验证

当增加或修改一个DAG定义后，就需要对DAG进行验证，判断是否是一个DAG图。

通过在graph表中增加字段checked，为1表示检测通过，为0表示未通过检测。

另外如果有一个pipeline流程使用了某个DAG，就不再允许呗修改和删除，通过在graph表中增加字段sealed来判断，1表示已被使用，0表示未使用。

1，根据kahn算法，先找出入度为0的顶点

```sql
# 找出graph_id为1的所有顶点和边
select * from vertex inner join edge on vertex.g_id = edge.g_id and vertex.g_id = 1;

# 找出graph_id为1的顶点和边,且弧尾是顶点的,因为左联,有head为null
select * from vertex left join edge on vertex.g_id = edge.g_id and edge.head = vertex.id where vertex.g_id = 1;

# 增加一个条件edge的head为null
select vertex.* from vertex left join edge on vertex.g_id = edge.g_id and edge.head = vertex.id where vertex.g_id = 1 and edge.head is null;
```

采用左联找edge里面找null的方式来找出入度为0 的顶点。

第一批入度为0的顶点被找到后，还需要再次查询找第二批顶点，不适合验证DAG。

换个思路，把所有的顶点和边都先查一遍，然后在客户端数据库结构中想办法处理，而不是多次查数据库。

### 算法1

算法思路：

一次把一个DAG的所有顶点和边都查回来,存到列表中；

遍历顶点列表，拿出一个顶点，去边的列表中找是否作为弧头，如果找到了，说明这个顶点的入度不为0，立即判断下一个顶点，如果没找到，则说明该顶点入度为0，可以移除以它作为弧尾的边和它本身了；

注意，因为移除会导致列表的索引发生变化，所以采用了先记录索引，倒序删除索引的方式；

如果入度为0的顶点和以它作为弧尾的有向边都被移除后，剩下一个空图，就说明检测通过，是DAG，空图的判断使用非负整数的相加为0一定都是0为依据。

如果遍历结束后，没有找到入度为0的顶点，说明它不是DAG。

```python
def check_graph(graph: Graph):
    """
    验证是否是一个合法的DAG
    既然要遍历所有的顶点和边,不如一次性把所有的顶点和边都查回来
    通过代码在内存中遍历查询
    :param graph:
    :return:
    """
    # 顶点列表[1,2,3,4]
    query = db.session.query(Vertex).filter(Vertex.g_id == graph.id)
    vertexes = [vertex.id for vertex in query]

    # 边列表[(1,2),(1,3),(3,2),(2,4)]
    query = db.session.query(Edge).filter(Edge.g_id == graph.id)
    edges = [(edge.tail, edge.head) for edge in query]

    # 遍历顶点
    while True:
        vertex_index = []
        for i, vertex in enumerate(vertexes):
            for _, head in edges:
                if head == vertex:
                    # 当前顶点是弧头,也就是有入度
                    break
                else:
                    # 不是弧头,说明入度为0
                    edge_index = []
                    for j, (tail, _) in enumerate(edges):
                        if tail == vertex:
                            # 找到以此顶点作为弧尾的边,也就是出度的边
                            # 将这条边的索引记录下来
                            edge_index.append(j)

                    # 将入度为0的这个顶点的索引记录下来
                    vertex_index.append(i)

                    # 删除边,反转一下索引,从后向前删,不会影响索引变化
                    for j in reversed(edge_index):
                        edges.pop(j)
                    # 删除入度为0的顶点,循环结束,重新遍历
                    break
            else:
                # 遍历一遍剩余顶点,如果没有被break,说明没有入度为0的顶点
                return False

            # 删除顶点
            for i in vertex_index:
                vertexes.pop(i)

            print(vertexes, edges)

            if len(vertexes) + len(edges) == 0:
                # 检测通过
                try:
                    graph = db.session.query(Graph).filter(Graph.id == graph.id).first()
                    if graph:
                        graph.checked = 1
                    db.session.add(graph)
                    db.session.commit()
                    return True
                except Exception as e:
                    db.session.rollback()
                    raise e
```

缺点：迭代次数太多了。

### 算法2

算法思路：

还是一次性把所有顶点和边都查回来，减少和数据库的交互。

顶点id不可能重复，所以采用set存储。

边从库中拿出的时候，就把弧尾作为字典的key，便于删除入度为0的顶点的边。

注意，只要边的字典有值，就说明一定有入度不为0的顶点。

如果用当前的顶点集减去所有入度不为0的顶点集，结果有2种可能：

1，不为空集，说明这是入度为0的顶点集；

2，空集，说明有环。

判断依据：

如果边的字典为空，退出循环，说明已经没有边了，但是顶点集可能还有顶点。

如果顶点集还有顶点，都是入度为0的顶点，都可以移除。

说明就是DAG。

如果入度为0的顶点没有找到，退出了循环，如果边的字典不为空，说明有环。

```python
from collections import defaultdict


def check_graph(graph: Graph) -> bool:
    # 查询某图的所有顶点
    query = db.session.query(Vertex).filter(Vertex.g_id == graph.id)
    vertexes = {vertex.id for vertex in query}
    print(vertexes)

    # 查询某图的所有的边
    query = db.session.query(Edge).filter(Edge.g_id == graph.id)
    edges = defaultdict(list)
    ids = set()  # 用于存放有入度的顶点
    for edge in query:
        print(edge)
        # defaultdict(<class 'list'>,{1:[(1,2),(1,3)], 2:[(2,4)],3:[(3,2)]})
        # 以边的弧尾顶点作为key,对应以此顶点为弧尾的边
        edges[edge.tail].append((edge.tail, edge.head))
        ids.add(edge.head)  # 存放的是边的弧头,也就是这个顶点有入度
    print("*" * 20)
    print(vertexes, edges)

    # 测试数据
    # {1,2,3,4}
    # defaultdict(<class list>,{1:[(1,2),(1,3)], 2:[(2,4)],3:[(3,2)]})
    # vertexex = {1,2,3,4}
    # edges = {1:[(1,2),(1,3)], 2:[(2,4)],3:[(3,2)]}
    # ids = set()

    # 一条边都没有,业务用不上,直接return
    if len(edges) == 0:
        return False

    """
    如果edges不为空,说明一定有ids,也就是一定有有入度的顶点.
    因为边依赖于两个顶点,有边就一定会有弧头和弧尾.
    然后利用集合的运算特性,如a={1,2,3},b={1,3},a-b={2}.
    也就是用所有顶点的集合,减去有入度的顶点,得出来入度为0的顶点.
    """
    # 入度为0的顶点的集合
    zds = vertexes - ids

    # 如果没有入度为0的顶点,则说明不是DAG,直接return
    # 如果一条边的弧尾入度为0,删除这个边
    if len(zds):
        for zd in zds:  # set
            if zd in edges:
                del edges[zd]

        while edges:
            """
            边集为空集则算法终止
            将顶点集改为当前入度顶点集ids
            因为已经删除了没有入度的顶点和其所关联的边
            要从剩下的顶点集中继续找入度为0的顶点
            """

            vertexes = ids
            ids = set()  # 重新寻找有入度的顶点

            for edge_lst in edges.values():
                for edge in edge_lst:
                    # edge元组, edge[1]弧头
                    ids.add(edge[1])
            zds = vertexes - ids  # 入度为0
            print(vertexes, ids, zds)
            if len(zds) == 0:
                break
            for zd in zds:
                if zd in edges:  # 顶点可能没有出度
                    del edges[zd]
            print(edges)
    # 边集为空,剩下的所有顶点都是入度为0,都可以多次迭代删除掉
    if len(edges) == 0:
        # 检测通过,修改checked字段为1
        try:
            graph = db.session.query(Graph).filter(Graph.id == graph.id).first()
            if graph:
                graph.checked = 1
            db.session.add(graph)
            db.session.commit()
            return True
        except Exception as e:
            db.session.rollback()
            raise e

    return False

```



### 效率测试

test.py

```python
def check_graph1(graph=None):
    # 测试数据
    vertexes = [1,2,3,4]
    edges = [(1,2),(1,3),(3,2),(2,4)]
    # 遍历顶点
    while True:
        vertex_index = []  # 存放顶点索引
        for i, vertex in enumerate(vertexes):
            for _, head in edges:
                if head == vertex:
                    # 当前顶点是弧头,也就是有入度
                    break
            else:
                # 没有被break,执行else,说名不是弧头,说明入度为0
                edge_index = []
                for j, (tail, _) in enumerate(edges):
                    if tail == vertex:
                        # 找到以此顶点作为弧尾的边,也就是出度的边
                        # 将这条边的索引记录下来
                        edge_index.append(j)

                # 将入度为0的这个顶点的索引记录下来
                vertex_index.append(i)

                # 删除边,反转一下索引,从后向前删,不会影响索引变化
                for j in reversed(edge_index):
                    edges.pop(j)
                # 删除入度为0的顶点,循环结束,重新遍历
                break
        else:
            # 遍历一遍剩余顶点,如果没有被break,说明没有入度为0的顶点
            return False

        # 删除顶点
        for i in vertex_index:
            vertexes.pop(i)
            # print(vertexes, edges, '!!!!!!!!!')
        if len(vertexes) + len(edges) == 0:
            # 检测通过
            return True
        return False


def check_graph2(graph=None) -> bool:
    # 测试数据
    # {1,2,3,4}
    # defaultdict(<class list>,{1:[(1,2),(1,3)], 2:[(2,4)],3:[(3,2)]})
    vertexes = {1,2,3,4}
    edges = {1:[(1,2),(1,3)], 2:[(2,4)],3:[(3,2)]}
    ids = set()

    # 一条边都没有,业务用不上,直接return
    if len(edges) == 0:
        return False

    """
    如果edges不为空,说明一定有ids,也就是一定有有入度的顶点.
    因为边依赖于两个顶点,有边就一定会有弧头和弧尾.
    然后利用集合的运算特性,如a={1,2,3},b={1,3},a-b={2}.
    也就是用所有顶点的集合,减去有入度的顶点,得出来入度为0的顶点.
    """
    # 入度为0的顶点的集合
    zds = vertexes - ids

    # 如果没有入度为0的顶点,则说明不是DAG,算法终止,直接return
    # 如果一条边的弧尾入度为0,删除这个边
    if len(zds):
        for zd in zds:  # set
            if zd in edges:
                del edges[zd]

        while edges:
            """
            边集为空集则算法终止
            将顶点集改为当前入度顶点集ids
            因为已经删除了没有入度的顶点和其所关联的边
            要从剩下的顶点集中继续找入度为0的顶点
            """

            vertexes = ids
            ids = set()  # 重新寻找有入度的顶点

            for edge_lst in edges.values():
                for edge in edge_lst:
                    # edge元组, edge[1]弧头
                    ids.add(edge[1])
            zds = vertexes - ids  # 入度为0
            # print(vertexes, ids, zds)
            if len(zds) == 0:
                break
            for zd in zds:
                if zd in edges:  # 顶点可能没有出度
                    del edges[zd]
            # print(edges)
    # 边集为空,剩下的所有顶点都是入度为0,都可以多次迭代删除掉
    if len(edges) == 0:
        return True
    return False


import datetime
start = datetime.datetime.now()
for _ in range(100000):
    check_graph1()
print((datetime.datetime.now() - start).total_seconds())

start = datetime.datetime.now()
for _ in range(100000):
    check_graph2()
print((datetime.datetime.now() - start).total_seconds())
```

效率提升约一倍。

使用算法2检测DAG

# 最终代码

app.py

```python
import json
from pipeline.model import Graph, Vertex, Edge, Pipeline, Track, db
from pipeline.service import create_graph, add_edge, add_vertex, del_edge, del_vertex
from pipeline.service import check_graph


def test_create_dag():
	...


def test_check_all_graph():
    query = db.session.query(Graph).filter(Graph.checked == 0).all()
    for g in query:
        if check_graph(g):
            g.checked = 1
            db.session.add(g)
    try:
        db.session.commit()
        print('done')
    except Exception as e:
        db.session.rollback()
        print(e)


db.drop_all()
db.create_all()
test_create_dag()
test_check_all_graph()

```

验证成功，修改所有图定义的checked字段为1

业务上应该在创建一个新的DAG的时候立即验证，或在修改一个DAG后立即验证。