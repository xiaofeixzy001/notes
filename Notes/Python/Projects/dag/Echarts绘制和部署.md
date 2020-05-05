[TOC]

# 前言

- 使用Flask微框架
- 使用Jinja2模板技术
- 使用JQuery发起AJAX异步调用
- 使用ECharts图表组件
- 使用uWSGI部署



# ECharts

网页端显示DAG图形

参考: https://www.echartsjs.com/examples/zh/index.html

以简单柱状图为例，修改如下

```shell
option = {
    title: {
        text: 'DAG 简单示例'
    },
    tooltip: {},
    animationDurationUpdate: 1500,
    animationEasingUpdate: 'quinticInOut',
    series : [
        {
            type: 'graph',  // 图的类型
            layout: 'none',
            symbolSize: 50,
            roam: true,
            label: {
                normal: {
                    show: true
                }
            },
            edgeSymbol: ['circle', 'arrow'],
            edgeSymbolSize: [4, 10],
            edgeLabel: {
                normal: {
                    textStyle: {
                        fontSize: 20
                    }
                }
            },
            data: [{
                name: 'A',
                x: 300,
                y: 300
            }, {
                name: 'B',
                x: 800,
                y: 300
            }, {
                name: 'C',
                x: 550,
                y: 100
            }, {
                name: 'D',
                x: 550,
                y: 500
            }],
            // links: [],
            links: [{
                source: 'A',
                target: 'C'
            }, {
                source: 'A',
                target: 'B'
            }, {
                source: 'B',
                target: 'C'
            }, {
                source: 'C',
                target: 'D'
            }],
            lineStyle: {
                normal: {
                    opacity: 0.9,
                    width: 2,
                    curveness: 0
                }
            }
        }
    ]
};
```

参数说明：

title：标题，对象

legend：图例

xAxis：x轴，对象，data属性就是x轴数据

yAxis：y轴，对象，type设定数据类型，'value'是值的类型

series：数据序列，数组，每个元素是一个对象



# Flask

参考：http://www.pythondoc.com/flask/index.html

## 安装

在当前python虚拟环境中安装

```shell
pip install flask
```

## 目录规划

在项目pipeline_10根目录下构建3个目录：

web：包，存放代码，这里代码放到了\_\_init\_\_.py文件中

web包目录下构建：

templates：目录存放模板文件

static：目录存放js和css等静态文件，其下建立js目录，存放jquery、echarts的js文件

service：存放与后台数据交互的代码

## 前端代码

将ECharts所需的js文件拷贝到static目录中

pipeline_10\web\static\js\echarts.min.js

pipeline_10\web\static\js\echarts.simple.min.js

pipeline_10\web\static\js\jquery-2.1.1.min.js

在pipeline_10/web/templates/目录下准备4个模板文件

index.html：首页文件

chart1.html：简单图标

chart2.html：DAG图标

chart3.html：DAG图标

### 首页文件

编辑"pipeline_10/templates/index.html"首页文件

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
        "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <title>流程系统</title>
    <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
</head>
<body>
<h2>流程系统-Flask-JQuery-Ajax-ECharts测试</h2>
<ur>
    <li><a href="1">图表1</a></li>
    <li><a href="2">图表2</a></li>
    <li><a href="3">图表3</a></li>
</ur>
</body>
</html>
```

### 柱状图1

显示柱状图简单示例

编辑"pipeline_10\web\templates\chart1.html"

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>ECharts</title>

    <!-- 引入 jquery -->
    <script src={{url_for('static', filename="js/jquery-2.1.1.min.js" )}}></script>

    <!-- 引入 echarts.js -->
    <script src={{url_for('static', filename="js/echarts.min.js" )}}></script>
</head>
<body>

<!-- 为ECharts准备一个具备大小（宽高）的Dom -->
<div id="main" style="width: 600px;height:400px;"></div>

<script type="text/javascript">

    // 基于准备好的dom，初始化echarts实例
    var myChart = echarts.init(document.getElementById('main'));

    // JQuery Ajax调用
    $.get('/dag/1', function (data) {
        console.log(data);

        // 指定图表的配置项和数据
        var option = {
            title: {
                text: 'ECharts 入门示例'
            },
            tooltip: {},
            legend: { // 图例
                data: ['销量', '产量']
            },
            xAxis: { // x轴
                data: data.xs
            },
            yAxis: {type: 'value'}, // Y轴
            series: [{ // 数据数据
                name: '产量',
                type: 'bar',
                data: data.data
            },
                {
                    name: '销量',
                    type: 'bar',
                    data: data.data.map(x => x + parseInt(Math.random() * 10 - 5))
                }]
        };

        // 使用刚指定的配置项和数据显示图表
        myChart.setOption(option);
    })
</script>
</body>
</html>
```

### Graph图2

使用JQuery Ajax方式提交数据

编辑"pipeline_10\web\templates\chart2.html"

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>ECharts</title>
    <!-- 引入 jquery -->
    <script src={{url_for('static', filename="js/jquery-2.1.1.min.js" )}}></script>
    <!-- 引入 echarts.js -->
    <script src={{url_for('static', filename="js/echarts.min.js" )}}></script>
</head>
<body>
<!-- 为ECharts准备一个具备大小（宽高）的Dom -->
<div id="main" style="width: 600px;height:400px;"></div>
<script type="text/javascript">
    // 基于准备好的dom，初始化echarts实例
    var myChart = echarts.init(document.getElementById('main'));
    $.get('/dag/2', function (data) {
        console.log(data);
        // 指定图表的配置项和数据
        option = {
            title: {
                text: 'DAG 简单示例 Echarts'
            },
            tooltip: {},
            animationDurationUpdate: 1500,
            animationEasingUpdate: 'quinticInOut',
            series: [
                {
                    type: 'graph',
                    layout: 'none',
                    symbolSize: 50,
                    roam: true,
                    label: {
                        normal: {
                            show: true
                        }
                    },
                    edgeSymbol: ['circle', 'arrow'],
                    edgeSymbolSize: [4, 10],
                    edgeLabel: {
                        normal: {
                            textStyle: {
                                fontSize: 20
                            }
                        }
                    },
                    data: data.data,
                    // links: [],
                    links: data.links,
                    lineStyle: {
                        normal: {
                            opacity: 0.9,
                            width: 2,
                            curveness: 0
                        }
                    },
                    tooltip: { // 提示框，鼠标放在节点或边上试一试
                        formatter: "{b}<br />{c}", // 占位符，{b}表示类目，{c}表示数值
                        backgroundColor: "#000000" //背景色
                    }
                }
            ]
        };
        // 使用刚指定的配置项和数据显示图表
        myChart.setOption(option);
    })
</script>
</body>
</html>
```

### Graph图3

编辑"pipeline_10\web\templates\chart3.html"

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>ECharts</title>
    <!-- 引入 jquery -->
    <script src={{url_for('static', filename="js/jquery-2.1.1.min.js" )}}></script>
    <!-- 引入 echarts.js -->
    <script src={{url_for('static', filename="js/echarts.min.js" )}}></script>
</head>
<body>
<!-- 为ECharts准备一个具备大小（宽高）的Dom -->
<div id="main" style="width: 600px;height:400px;"></div>
<script type="text/javascript">
    // 基于准备好的dom，初始化echarts实例
    var myChart = echarts.init(document.getElementById('main'));
    $.get('/dag/3', function (data) {
        console.log(data);
        myChart.hideLoading();
        // 指定图表的配置项和数据
        option = {
            title: {
                text: data.title
            },
            tooltip: {trigger: 'item'},
            animationDurationUpdate: 1500,
            animationEasingUpdate: 'quinticInOut',
            series: [
                {
                    type: 'graph',
                    layout: 'none',
                    symbolSize: 50,
                    roam: true,
                    label: {
                        normal: {
                            show: true
                        }
                    },
                    edgeSymbol: ['circle', 'arrow'],
                    edgeSymbolSize: [4, 10],
                    edgeLabel: {
                        normal: {
                            textStyle: {
                                fontSize: 20
                            }
                        }
                    },
                    data: data.data,
                    // links: [],
                    links: data.links,
                    lineStyle: {
                        show: false,
                        normal: {
                            opacity: 0.9,
                            width: 2,
                            curveness: 0
                        }
                    },
                    tooltip: { // 提示框，鼠标放在节点或边上试一试
                        // 使用函数重新定义显示文字的格式，回调送入3个参数
                        formatter: function (params, ticket, callback) {
                            if (params.dataType === 'edge') // 连线没有值返回空串
                                return '';
                            if (params.value)
                                return params.name + '<br />' + params.value
                            return params.name
                        }
                        //, backgroundColor: "#000000"
                    }
                }
            ]
        };
        // 使用刚指定的配置项和数据显示图表
        myChart.setOption(option);
        // 遍历数据
        echarts.util.map(data.data, function (item, dataIndex) {
            console.log(item);
            console.log(dataIndex);
        });
        // 鼠标事件，click点击
        myChart.on('mouseover', function (item) {
            console.log(item);
            if (item.value) {
                console.log(item.value)
            }
        });
    });
</script>
</body>
</html>
```

注意观察2和3的边，鼠标放上去的效果。

## 服务端

### 路由

编辑"pipeline_10/web/\_\_init\_\_.py"

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# __Author__: XF
# 视图层
from flask import Flask, make_response, request, render_template, jsonify
from .service import get_pipeline


app = Flask(__name__)


@app.route('/', methods=['GET'])
def index():
    return render_template('index.html')


@app.route('/<int:graph_id>')
def show_dag(graph_id):
    return render_template('chart{}.html'.format(graph_id))


@app.route('/dag/<int:pipeline_id>')
def dags(pipeline_id):
    if pipeline_id == 1:
        return simple_graph()
    elif pipeline_id == 2:
        return jsonify(get_pipeline(1))
    elif pipeline_id == 3:
        return jsonify(get_pipeline(1))


def simple_graph():
    xs = ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
    data = [5, 20, 36, 10, 10, 20]

    return jsonify({'xs': xs, 'data': data})
```

### 数据处理

从pipeline中复制config.py和model.py到web目录下，当然也可以import导入，不过后期修改不方便。

编辑"pipeline_10\web\service.py"文件

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# __Author__: XF

from .model import db, Pipeline, Track, Vertex, Edge
import random


def randomxy():
    # 随机模仿x,y坐标
    return random.randint(300, 500)


def get_pipeline(p_id):
    # 根据pipeline的id返回数据
    query = db.session.query(
        Pipeline.id, Pipeline.name, Pipeline.state, Pipeline.graph_id,
        Vertex.id, Vertex.name,
        Track.state, Track.script, Track.input
    ).join(
        Track, Track.pipeline_id == Pipeline.id
    ).join(
        Vertex, Vertex.id == Track.vertex_id
    ).filter(
        Pipeline.id == p_id
    )

    # 顶点数据
    data = []  # 顶点数据
    vertexes = {}  # 让edge查询少join
    for pl_id, p_name, p_state, g_id, v_id, v_name, t_state, t_script, t_input in query:
        data.append({
            'name': v_name,
            'x': randomxy(),
            'y': randomxy(),
            'value': t_script
        })
        vertexes[v_id] = v_name

    # 边数据
    links = []
    q = db.session.query(Edge.tail, Edge.head).filter(Edge.graph_id == g_id).all()
    for tail, head in q:
        links.append({
            'source': vertexes[tail],
            'target': vertexes[head]
        })

    title = p_name

    return {'data': data, 'links': links, 'title': title}
```

### 启动

在项目根目录下创建测试文件

编辑"pipeline_10\test.py"

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# __Author__: xiaofei

from web import app

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

模板一旦被调用，返回的HTML页面会立即发起AJAX调用，请求DAG数据，视图函数会向Service层请求数据。

## 部署

采用uWSGI+Flask

uwsgi安装在Linux服务器上，假设服务器上已经安装了python3和pyenv

```shell
# su python
$ cd
$ mkdir projects/flask
$ cd projects/flask
$ pyenv versions
$ pyenv virtualenv 3.5.4 f354
$ pyenv local f354
$ pyenv version
$ cd ..
$ cd flask
$ pip list
$ pip install uwsgi flask pymysql sqlalchemy
$ pip list
```

将pipeline_10\web整个目录上传到linux服务器上，上传位置为/home/python/projects/flask

```shell
$ uwsgi --http :5000 -w web:app
```

或创建一个配置文件

```shell
$ vim flask.ini
"""
[uwsgi]
http = 0.0.0.0:5000
module = web:app
"""
$ uwsgi flask.ini
$ ss -tnlp
```

访问http://192.168.100.3:5000/

