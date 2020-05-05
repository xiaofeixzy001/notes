[TOC]

Column对应的就是数据表中的一个字段，所以它的参数设置是与数据表息息相关的，为了能让ORM创建出符合我们业务要求的数据表就必须把Column的参数给弄清楚。文档来源自源码中的doc注释，不得不说注释写的是真的详细完全可以不用再翻文档了。文档地址：点击跳转。只是对Param文档做一个简单的汉化和介绍，有错误欢迎指正

Param name

字段名参数，用作定义数据表字段名，参数定义需要位于参数列的首位或者通过键值定义。name参数通常可以省略，如果省略的话会自动使用Model的属性名作为name属性。注意name属性不区分大小写，也就是对大小写不敏感，除非是保留字。即使对有着不区分大小写标准的数据库如Oracle，该特性同样是适用的。

Param type

字段的数据类型，对应数据表中字段的数据类型。具体包括那些数据类型可以通过：sqltypes.py查看，和数据库中的数据类型应该是相互对应的，暂时没做过多的考究。

传入的类必须是SQL-Alchemy提供的类型，如果类中有参数的话需要带上参数，没有参数的话可以直接传入相应的类名。该参数定义通常位于参数列的第二位或者通过键值定义。如果type参数被省略或者定义为None时，会首先解析为特殊类型NoneType。当Column通过ForeignKey或者ForeignKeyConstraint类引用其他其他列时，远程引用列的类型将被复制到此列，同时将被解析为远程Column的外键。

Param *args:

其他位置参数包括各种class： SchemaItem的派生类将作为选项应用于Column，这些包括：class：Constraint，：class：ForeignKey，：class：ColumnDefault，和：class：Sequence。在某些情况下可以使用相应的关键字参数代替。例如：server_default、default、unique，这两种方式起到的作用是相同的。如果你还是搞不清楚上面这堆屁话说的什么，我们通过class：ColumnDefault给的文档中的代码就可以很容易的理解。

  \# 原始定义Column声明

  Column('foo', Integer, default=50)

  \# 等效于下面这种声明

  Column('foo', Integer, ColumnDefault(50))

Param autoincrement

字段自动增长参数，为整数主键列设置“自动增量”语义。通俗一点就是让整数型的主键可以自动增长的参数。该设置只作用在部份Column：

  Integer derived (i.e. INT, SMALLINT, BIGINT)，整数型派生出来的数据类型

  Part of the primary key，部份主键

  不会隐射通过ForeignKey定义的其他Column，除非指定该参数为"ignore_fk"

这个参数有很多底层的特性，文档介绍了一大堆，这里只挑了重点的特性阐述，有兴趣的可以查看具体文档。

Param default

default参数之前文章已经详细讨论了，在Python端设置字段的默认值，而不是在服务端设置。default属性会在执行insert参数的时候生成而不是在实例化时就生成。该属性同样可以用类ColumnDefault替代，效果是完全一样的。可传入Python调用，在执行调用后返回值作为Column的默认值。本参数是使用频率很高的，所以务必要了解其特性。

Param doc

可选的文本属性，可以被ORM使用，或者类似与Python端的__doc__属性。该参数只作用于Python端，并不会影响数据库的comment属性。如果想要达到设置comment属性的目的，需要指定comment参数。

Param key

可选的文本参数，如果参数被设置之后就会与该Column建立联系，想要在Python中访问和修改该属性只能通过key的方式进行，原来的name属性只会体现在数据库中。有点难懂啊，放代码来理解一下。

 

```
from sqlalchemy import Column, String, Integer, ColumnDefault
    from flask_sqlalchemy import SQLAlchemy
    db = SQLAlchemy()
    class Test(db.Model):
        # __tablename__ = "test"
        id = Column(Integer, primary_key=True)
        name = Column(String(11), default="佚名", key="my_name")
        age = Column(Integer, default=0)
    test = Test()
    # 之前我们可以直接通过类下的属性访问和修改相应字段，如age字段
    test.age = 123
    print(test.age) 
    >>>123
    # 设置了key之后只能通过key访问相应的字段，不能像上面一样访问
    test.my_name = "weieny"
    print(test.my_name)
    >>>weiney
    PS:虽然直接操作name属性不会报错，这是Python动态语言的特性，但是操作name属性不会对Column值造成影响
    只有操作定义的key属性才能真正操作Column
```

Param index

设置Column为索引参数，当index参数为True时就将Column编入索引这是使用Index索引的快捷方式。要指定具有显式名称的索引或包含多个列的索引，请改用：class：Index构造。对数据库索引不是很清楚的同学可以看看这个文章；深入浅出数据库索引原理。

Param info

可选的数据字典参数，设置了的字典将会填充到SchemaItem.info属性当中。

Param nullable

字段可空参数，当参数设置为False的时候将会在创建数据表的时候给字段加上not null限制。当参数设置为True的时候，不会只生成任何SQL（数据表中的默认值将会时Null）。当Column为主键的时候该参数默认为False，其他情况默认值都是True，并且该属性只会在创建数据表的时候被引用。

Param onupdate

表示要应用于UPDATE语句中的列的默认值，可接受标量，Python调用，或者class:sqlalchemy.sql.expression.ClauseElement。在执行update语句时，如果本列不存在，将会在updete语句更新时自动调用set语句给本列赋值为定义的值。这是使用：class：ColumnDefault作为``for_update = True``的位置参数的快捷方式。

这个参数作用还是很大的，在实际开发过程中，我们的数据表并不是刚开始就定义并且完善的。随着业务逻辑的变更数据表总是不停的增加字段，如何给我们新增的字段设置默认值就是本参数要做的事情。

param primary_key

设置字段是否为主键参数，如果设置值为True时Column将会被设置为主键。作为替代方案，可以通过显式：class：PrimmaryKeyConstraint对象指定数据表的主键。

param unique

设置字段值为唯一，当Column为index时则unique为True。要约束索引中指定多个列或指定显式名称，使用类UniqueConstraint或者Index显示构造。

param comment

数据库字段注解参数。设置此参数会改变创建数据库字段的comment属性，类似于注释。和内部的doc属性做区分，comment是作用于数据库端的，doc是作用于Python端的。

param server_default

设置数据库端的字段默认值。参数接受FetchedValue类的实例，str，Unicode或者函数sqlalchemy.sql.expression.text。具体三种形式的参数使用文档里给了详细的代码介绍，由于篇幅的原因就不做罗列，可以看看文档的实例，其中sqlalchemy.sql.expression.text的使用方法可以详细看看，个人觉得用处很大。

Param server_onupdate

FetchedValue类实例，表示数据库端默认生成函数，例如触发器。 这向SQLAlchemy表明更新后新生成的值将可用。 这个构造实际上并没有在数据库中实现任何类型的生成函数，而是必须单独指定。这个和server_default有很多相似之处。

对于以下两个参数我也不是很理解，翻译直接引自谷歌翻译，等我再消化消化。

Param quote

引用：强制引用或禁用此列的名称，对应于“True”或“False”。 如果保留默认值“None”，则根据名称是否区分大小写（具有至少一个大写字符的标识符被视为区分大小写）或者如果它是保留字来引用列标识符。 只需要强制引用SQLAlchemy方言不知道的保留字，就需要此标志。

param system

如果参数设置为True时表示这是一个由数据库自动提供的列，不应包含在``CREATE TABLE``语句的列列表中。对于更精细的场景，其中列应在不同的后端有条件地呈现，请考虑以下自定义编译规则：class：`.CreateColumn`。