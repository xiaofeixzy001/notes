[TOC]

# XML

XML 指可扩展标记语言(EXtensible Markup Language);

XML 是一种标记语言,很类似 HTML;

XML 的设计宗旨是传输数据,而非显示数据;

XML 标签没有被预定义,您需要自行定义标签;

XML 被设计为具有自我描述性;

XML 是 W3C 的推荐标准

xml是实现不同语言或程序之间进行数据交换的协议,跟json差不多,但json使用起来更简单.

movies.xml文件示例模板

 

```
<collection shelf="New Arrivals">
<movie title="Enemy Behind" name="1">
   <type>War, Thriller</type>
   <format>DVD</format>
   <year>2003</year>
   <rating>PG</rating>
   <stars>10</stars>
   <description name="****War, Thriller****">Talk about a US-Japan war</description>
</movie>
<movie title="Transformers" name="2">
   <type>Anime, Science Fiction</type>
   <format>DVD</format>
   <year>1989</year>
   <rating>R</rating>
   <stars>8</stars>
   <description name="****Transformers****">A schientific fiction</description>
</movie>
   <movie title="Trigun" name="3">
   <type>Anime, Action</type>
   <format>DVD</format>
   <episodes>4</episodes>
   <rating>PG</rating>
   <stars>10</stars>
   <description name="****Trigun****">Vash the Stampede!</description>
</movie>
<movie title="Ishtar" name="4">
   <type>Comedy</type>
   <format>VHS</format>
   <rating>PG</rating>
   <stars>2</stars>
   <description name="****Ishtar****">Viewable boredom</description>
</movie>
</collection>
```

常见的XML编程接口有DOM和SAX,这两种接口处理XML文件的方式不同,当然使用场合也不同.

# xml处理

python有三种方法解析XML:

### 1,SAX

simple API for XML

python标准库包含SAX解析器,SAX用事件驱动模型,通过在解析XML的过程中触发一个个的事件并调用用户定义的回调函数来处理xml文件.

示例:

获取movies.xml中每部电影的信息

 

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# __author__ = "xiaofei"

import xml.sax

class MovieHandler(xml.sax.ContentHandler):
    def __init__(self):
        self.CurrentData = ""
        self.type = ""
        self.format = ""
        self.year = ""
        self.rating = ""
        self.stars = ""
        self.description = ""

    # 元素开始调用
    def startElement(self, tag, attributes):
        self.CurrentData = tag
        if tag == "movie":
            print("*****Movie*****")
            title = attributes["title"]
            print("Title:", title)

    # 元素结束调用
    def endElement(self, tag):
        if self.CurrentData == "type":
            print("Type:", self.type)
        elif self.CurrentData == "format":
            print("Format:", self.format)
        elif self.CurrentData == "year":
            print("Year:", self.year)
        elif self.CurrentData == "rating":
            print("Rating:", self.rating)
        elif self.CurrentData == "stars":
            print("Stars:", self.stars)
        elif self.CurrentData == "description":
            print("Description:", self.description)
        self.CurrentData = ""

    # 读取字符时调用
    def characters(self, content):
        if self.CurrentData == "type":
            self.type = content
        elif self.CurrentData == "format":
            self.format = content
        elif self.CurrentData == "year":
            self.year = content
        elif self.CurrentData == "rating":
            self.rating = content
        elif self.CurrentData == "stars":
            self.stars = content
        elif self.CurrentData == "description":
            self.description = content


if (__name__ == "__main__"):
    # 创建一个 XMLReader
    parser = xml.sax.make_parser()
    # 关闭命名空间
    parser.setFeature(xml.sax.handler.feature_namespaces, 0)

    # 重写 ContextHandler
    Handler = MovieHandler()
    parser.setContentHandler(Handler)

    parser.parse("movies.xml")
```

### 2,DOM

Document Object Model

将xml数据在内存中解析成一个树,通过对树的操作来操作xml.

 

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# __author__ = "xiaofei"
from xml.dom.minidom import parse
import xml.dom.minidom

# 使用minidom解析器打开 XML 文档
DOMTree = xml.dom.minidom.parse("movies.xml")
collection = DOMTree.documentElement
if collection.hasAttribute("shelf"):
    print ("Root element : %s" % collection.getAttribute("shelf"))

# 在集合中获取所有电影
movies = collection.getElementsByTagName("movie")

# 打印每部电影的详细信息
for movie in movies:
    print ("*****Movie*****")
    if movie.hasAttribute("title"):
        print ("Title: %s" % movie.getAttribute("title"))

    type = movie.getElementsByTagName('type')[0]
    print ("Type: %s" % type.childNodes[0].data)
    format = movie.getElementsByTagName('format')[0]
    print ("Format: %s" % format.childNodes[0].data)
    rating = movie.getElementsByTagName('rating')[0]
    print ("Rating: %s" % rating.childNodes[0].data)
    description = movie.getElementsByTagName('description')[0]
    print ("Description: %s" % description.childNodes[0].data)
```

### 3,ElementTree

解析xml方法:

1,调用parse()方法,获取解析树

 

```
try:
    import xml.etree.cElementTree as ET
except ImportError:
    import xml.etree.ElementTree as ET

tree = ET.parse("movies.xml")  # <class 'xml.etree.ElementTree.ElementTree'>
root = tree.getroot()           # 获取根节点 <Element 'data' at 0x02BF6A80>
```

2,调用fromstring(),获取解析树

 

```
import xml.etree.ElementTree as ET
data = open("movies.xml").read()
root = ET.fromstring(data)   # <Element 'data' at 0x036168A0>
```

3,调用ElementTree类,获取解析树

 

```
import xml.etree.ElementTree as ET
tree = ET.ElementTree(file="movies.xml")  # <xml.etree.ElementTree.ElementTree object at 0x03031390>
root = tree.getroot()  # <Element 'data' at 0x030EA600>
```

遍历xml

 

```
import xml.etree.ElementTree as ET

tree = ET.parse("movies.xml")
root = tree.getroot()  # 获取所有根节点
print(root.tag, ":", root.attrib)  # 打印根元素的tag和属性

# 遍历xml文档的第二层
for child in root:
    # 第二层节点的标签名称和属性
    print(child.tag,":", child.attrib) 
    # 遍历xml文档的第三层
    for children in child:
        # 第三层节点的标签名称和属性
        print(children.tag, ":", children.attrib)

# 通过下标的方式直接访问节点
year1 = root[0][0].text
year2 = root[0][1].text
year3 = root[1][0].text
year4 = root[1][1].text
print(year1)  # War, Thriller
print(year2)  # DVD
print(year3)  # Anime, Science Fiction
print(year4)  # DVD
```

ElementTree提供的方法

find(match) # 查找第一个匹配的子元素， match可以时tag或是xpaht路径

findall(match) # 返回所有匹配的子元素列表

findtext(match, default=None) # 

iter(tag=None) # 以当前元素为根节点 创建树迭代器,如果tag不为None,则以tag进行过滤

iterfind(match) # 

 

```
# 遍历出所有的description标签及其属性
for i in root.iter('description'):
    print(i.tag, ":", i.attrib)
"""
运行结果:
description : {'name': '****War, Thriller****'}
description : {'name': '****Transformers****'}
description : {'name': '****Trigun****'}
description : {'name': '****Ishtar****'}
"""

# 遍历所有的movie标签
for m in root.findall('movie'):
    # 查找movie标签下的第一个tpye标签
    type = m.find('type').text
    
    # 获取movie标签的name属性
    name = m.get('name')
    print(type)
    print(name)
"""
运行结果:
War, Thriller
1
Anime, Science Fiction
2
Anime, Action
3
Comedy
4
"""
```

修改xml

1,修改属性

 

```
# 将所有的episodes的值加1,并添加属性updated为yes
for e in root.iter('episodes'):
    e1 = int(e.text) + 1
    e.text = str(e1)  # 必须将int转换为str格式
    e.set('updated','yes')
    del e.attrib['updated'] # 删除

ET.dump(root)  # 显示内存中修改后的xml内容
tree.write('movies.xml')  # 将内存中的修改更新到文件
```

小结:

attrib: 为包含元素属性的字典

keys(): 返回元素属性名称列表

items(): 返回(name,value)列表

get(key, default=None): 获取属性

set(key, value): 跟新/添加属性

del xxx.attrib[key]: 删除对应的属性

2,修改节点

 

```
# 删除stars小于8的电影
for m in root.iter('movie'):
    s = int(m.find('stars').text)
    if s < 8:
        root.remove(m)  # 删除子元素
ET.dump(root)

# 添加子元素
movie = root[0]
last_ele = movie[len(list(movie)) - 1]
last_ele.tail = '\n\t'
elem1 = ET.Element('test_append1')
elem1.text = '这是elem1添加的内容'
movie.append(elem1)

elem2 = ET.SubElement(movie,'test_append2')
elem2.text = '这是elem2添加的内容'

elem3 = ET.Element('test_append3')
elem3.text = '这是elem3添加的内容'
elem4 = ET.Element('test_append4')
elem4.text = '这是elem4添加的内容'
movie.extend([elem3,elem4])

elem5 = ET.Element('test_append5')
elem5.text = '这是elem5添加的内容'
movie.insert(0,elem5)  # 0是子元素的下标,表示插在第几个子元素的前面

ET.dump(root)
```

添加子元素方法:

append(subelement)

extend([sub1,sub2,..])

insert(index,element)

创建xml模板

 

```
import xml.etree.ElementTree as ET
from xml.dom import minidom

def subElement(root, tag, text):
    ele = ET.SubElement(root, tag)
    ele.text = text

def saveXML(root, filename, indent="\t", newl="\n", encoding="utf-8"):
    rawText = ET.tostring(root)
    dom = minidom.parseString(rawText)
    with open(filename, 'w') as f:
        dom.writexml(f, "", indent, newl, encoding)

root = ET.Element("note")

to = root.makeelement("to", {})
to.text = "peter"
root.append(to)

subElement(root, "from", "marry")
subElement(root, "heading", "Reminder")
subElement(root, "body", "Don't forget the meeting!")

# 保存xml文件
saveXML(root, "temp.xml")
```