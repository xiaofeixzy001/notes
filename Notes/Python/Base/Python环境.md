[TOC]

# 系统准备

一般实际工作中,开发在windows环境,运行在linux环境. 

linux最小化系统即可，CentOS 6.x系统默认自带python,但其版本是2.6,我们需要额外安装3.x版本.

## linux

```
# 安装gcc,用于编译python编码
yum install gcc readline-devel zlib* -y
wget https://www.python.org/ftp/python/3.5.3/Python-3.5.3.tar.xz
tar -xf Python-3.5.3.tar.xz
cd Python-3.5.3
./configure --prefix=/usr/local/python35
make all && make install
python -v
ln -sv /usr/local/python36/bin/python3 /usr/bin/python3
vim ~/.bash_profile
'''
PATH=$PATH:$HOME/bin:/usr/local/python35/bin
'''
source ~/.bash_profile
python3 -V
pip3 -V
```



## windows

下载地址:

https://www.python.org/downloads/

如果官网慢,可以到搜获镜像网站下载:

http://mirrors.souhu.com/python/

推荐下载3.X版本，默认安装目录这里为C:\python35

在安装3.x版本的时候,会有自动添加环境变量的选项：

![img](Python%E7%8E%AF%E5%A2%83.assets/6914115c-febf-4a07-9c43-9a3b2a1e5005.jpg)

如果忘记勾选，可以参考如下办法添加：

计算机属性-->高级系统设置-->高级-->环境变量-->找到变量名为path的一行打开-->python安装目录追加到变量值中，用分号；分割

![img](Python%E7%8E%AF%E5%A2%83.assets/4911ea1e-8e1d-43cb-8527-92d524e8f6a0.png)

# 开发工具

在linux系统上安装python，通常我们不会直接安装，而是先安装虚拟环境，然后在安装不同版本的python，这样可以做到版本控制。

## Pyenv

### 简介

官网: https://github.com/pyenv/pyenv

安装文档: https://github.com/pyenv/pyenv-installer 

Python多版本管理工具

\- 管理python解释器

\- 管理python版本

\- 管理python的虚拟环境

### Linux安装

\- 安装python,git

```
yum -y install git gcc make patch gdbm-devel openssl-devel sqlite-devel readline-devel zlib-devel bzip2-devel tk-devel
```

安装pyenv

```
useradd python
su - python
curl -L https://raw.githubusercontent.com/pyenv/pyenv-installer/master/bin/pyenv-installer | bash # 安装脚本

vim ~/.bashrc # 追加下面内容
"""
export PATH="/home/python/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
"""

source ~/.bash_profile
```

说明:

pyenv安装时会去联网,然后下载,解压编译安装,如果网络不好,速度会很慢,可以选择使用cache缓存方式安装

pyenv安装完成后,在家目录会有一个.pyenv目录,然后在里面创建一个cache目录,把已经下载好的python包文件都放进去,在安装时就相当于本地安装.

如果curl出现 *curl:(35) SSL connect error,是nss版本低的问题，更新它yum update nss*

```
cd .pyenv/
mkdir cache
"""
python-3.5.4.tar.gz
python-3.5.4.tar.xz
python-3.5.4.tgz
"""
```



### pyenv命令

pyenv # 查看帮助信息

pyenv help install

pyenv install --list # 列出所有可用版本

pyenv install 3.5.4 -v # 安装指定版本的python,-v显示安装信息

pyenv versions # 查看所有python版本

pyenv version  # 查看当前python版本

pyenv global 3.5.4 # 全局设置,设置非root用户默认python版本,比较危险

pyenv shell 3.5.4 # 会话设置,设置当前会话的python版本,表示仅对当前会话改变版本,重新登录后失效,也不方便

pyenv local 3.5.4 # 本地设置,仅设置当前工作目录及其子目录的python版本,也就是说其他目录还使用系统默认的python版本,仅当前目录使用指定的python版本

## Virtualenv

Virtualenv可用于搭建多版本python 虚拟开发环境,推荐使用虚拟环境.

官网: https://pypi.python.org/pypi/virtualenv

pyenv可以帮助你在一台开发机上建立多个版本的python环境,并提供方便的切换方法: global, shell, local;

virtualenv则提供了一种功能，就是将一个目录建立为一个虚拟的python环境,这样的话,用户可以建立多个虚拟环境,每个环境里面的python版本可以是不同的,也可以是相同的,而且环境之间相互独立.

比如:

首先我们可以用pyenv安装多个python版本,比如安装了2.6, 3.5, 3.6三个版本,用户可以随意切换当前默认的python版本.

但这时候,每个版本的环境仍是唯一的,如果我们想在环境中安装一些库的话,还是会导致这个版本的环境被修改,这个时候，如果我们用virtualenv去建立虚拟环境,就可以完全保证系统路径的干净,无论你在虚拟环境中安装了什么程序,都不会影响已安装版本的系统环境.

### 示例

先部署多版本环境

```
pyenv install -l # 查看可安装python版本

# 假设这里安装2个版本
pyenv install 3.5.4 -v
pyenv install 2.7.15 -v

pyenv versions # 查看当前所有的python版本
"""
* system (set by /home/tony/.pyenv/version) # 代表系统当前版本, *号代表当前使用的版本
  3.5.4
  2.7.15
"""

ls .pyenv/versions  # 安装的各个版本所在路径
"""
3.5.4  2.7.15
"""

ls 3.5.4
"""
bin  include  lib  share  # 虚拟环境将来会出现在对应的版本目录里面
"""

ls 2.7.15
"""
bin  include  lib  share
"""

# 卸载某个版本
pyenv uninstall x.x.x
```

使用virtual创建虚拟python环境

virtualenv本是一个独立的工具,如果你是安装我们前面的方式安装pyenv的,那它已经帮我们以plugin的形式安装好了virtualenv, 我们只要使用就好了.

```
# 家目录下创建一个projects目录,以此目录为公共环境,默认版本为system
mkdir projects
cd projects
pyenv version

# 在projects目录下分别创建2个子目录,比如cmdb,web,因为继承特性,子目录的环境也是system
mkdir cmdb
mkdir web

# 进入到cmdb中,因为默认版本是系统的默认版本,但是我想用3.5.4版本来开发
cd cmdb
pyenv virtualenv 3.5.4 app354
# 上面的命令的意思是:在当前目录创建一个名为app354且python版本是3.5.4

pyenv versions
"""
* system (set by /home/python/projects/cmdb/.python-version)
  3.5.4
  3.5.4/envs/app354  # 这里是虚拟环境的真是目录: ~/.pyenv/versions/3.5.4/env/app354
  2.7.15
  app354
"""

ls ~/.pyenv/versions/  # 查看安装的各版本路径,发现多了一个链接文件
"""
3.5.4  2.7.15  app354 # app354 -> /home/python/.pyenv/versions/3.5.4/envs/app354
"""
```

虚拟环境的使用

```
# 改变当前目录的环境为虚拟环境app354
pyenv local app354
"""
(app354) [python@study-python cmdb]$  # 然后提示符前面多了(app354)
"""

pyenv version
"""
app354 (set by /home/python/projects/cmdb/.python-version)
"""
# 也就是说,现在是的版本环境使用的就是虚拟环境的版本了

cd ..
"""
[python@study-python cmdb]$  # 切换目录后,也就退出了虚拟环境
"""

cd cmdb/
"""
(app354) [python@study-python cmdb]$  # 在切进去,则又重新进入虚拟环境了
"""

# 同样,在web/目录下也创建一个虚拟环境,版本选择3.6.1
cd web/
pyenv version
pyenv virtualenv 2.7.15 app361
pyenv local app361
pyenv version
```

至此,不同的目录分别使用了不同版本的虚拟环境,可以很好的适用于多人开发.



## IPython

### 简介

增强的python shell, 自动补全,自动缩进,支持shell,增加了很多函数.

```
su python
mkdir projects/ipython
pyenv local 3.5.4
pip install --upgrade pip
pip install ipython
ipython3
```

说明: 默认pip程序都安装在了.pyenv/versions/3.5.3/envs/app353/lib/python3.5/site-packages/目录下

### ipython使用

```
ipython
```



## jupyter

安装启动

```
pip install jupyter
jupyter notebook --help
jupyter notebook password  # 设置密码
jupyter notebook  # 启动,默认会启动一个窗口化工具
ss -tnlp  # 默认监听在本地127.0.0.1:8888

jupyter notebook --ip=0.0.0.0 --port=8888 --no-browser

# 脚本
cat jupyter_start.sh
"""
#!/bin/bash
#
# Set jupyter boot start

jupyter notebook --ip=0.0.0.0 --port=8888 --allow-root --no-browser >> jupyter.logs 2>&1 &
"""

# 将脚本加入开机执行文件中
cat /etc/rc.local
'''
/root/jupyter_start.sh
'''
```

web访问:http://0.0.0.0:8888,显示如下页面

![img](Python%E7%8E%AF%E5%A2%83.assets/1245a54d-9b42-4a9f-9583-23820bee894c.png)

查看启动日志：

![img](Python%E7%8E%AF%E5%A2%83.assets/36f44bd4-6c29-446e-9ff3-c71c0d5aa9fb.png)

这里有一个 token值，如果不是使用默认浏览器打开的需要在 password or token 中输入 token 值，点击登录即可，或启用密码或设置新密码。

```
# 生成配置文件
jupyter-notebook --generate -config

# 修改默认工作目录
vim /home/python/.jupyter/jupyter_notebook_config.py
"""
c.NotebookApp.ip = '*'  # 允许访问ip
c.NotebookApp.open_browser = False  # 是否打开新窗口
c.NotebookApp.port = 8888  # 默认监听端口
c.NotebookApp.notebook_dir = 'jobs/'  # 默认工作目录
"""

# 设置web访问密码
jupyter-notebook password
```



### 离线安装

提前从github上克隆项目

```
$ git clone https://github.com/pyenv/pyenv.git ~/.pyenv
$ git clone https://github.com/pyenv/pyenv-virtualenv.git ~/.pyenv/plugins
/pyenv-virtualenv
$ git clone https://github.com/pyenv/pyenv-update.git ~/.pyenv/plugins/pyenvupdate
$ git clone https://github.com/pyenv/pyenv-which-ext.git ~/.pyenv/plugins
/pyenv-which-ext
```

可以把克隆的目录打包，方便以后离线使用

```
$ vim ~/.bash_profile
'''
export PATH="/home/python/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
'''

$ source ~/.bash_profile
```

jupyter启动脚本

```
#!/bin/bash
# jupyter start
jupyter notebook --config ~/.jupyter/jupyter_notebook_config.py --allow-root >> jupyter.logs 2>&1 &
```

jupyter停止脚本

```
#!/bin/bash
# jupyter stop
kill -9 `ps -ef | grep 'jupyter-notebook' |grep -v grep | awk -F' ' '{print $2}'`
```



## Pip

### 简介

pip是python包管理工具,类似linux系统的yum,用法也类似.

pip默认使用的是python自己的pip,python版本不同,pip也会不同.

官方文档：https://pip.pypa.io/en/latest/

```
# 查看pip版本
pip -V
```



### 修改镜像源

pip安装模块或软件包时,默认会去官网去search, 受网络影响,速度会很慢,可修改search源,指向一个速度快的快的url中.

国内常用pypi镜像

阿里：https://mirrors.aliyun.com/pypi/simple

中国科学技术大学：http://pypi.mirrors.ustc.edu.cn/simple/

示例: 修改pip源为阿里云镜像网站.

```
# 指定单次安装源
pip install <包名> -i https://mirrors.aliyun.com/pypi/simple

# 指定全局安装源
# 在unix和macos，配置文件为：~/.pip/pip.conf
# 在windows上，配置文件为：~\pip\pip.ini

mkdir ~/.pip
vim ~/.pip/pip.conf
'''
[global]
timeout = 6000
index-url = https://mirrors.aliyun.com/pypi/simple/
trusted-host=mirrors.aliyun.com
'''
```



### 升级降级

```
# 升级到最新版本
python -m pip install --upgrade pip

# 降级到指定版本[==,>=,<=,>,<]
python -m pip install pip==9.0.3
```



### pip使用

格式:

pip <command> [options]

```
# 列出已安装的包
pip freeze
pip list

# 查询可升级包
pip list -o

# 升级包
pip install -U <包名>
pip install <包名> --upgrade

# 导出已安装包的列表至requirements.txt文件中
pip freeze > /PATH/requirements.txt

# 安装包
pip install <包名>
pip install -r requirements.txt
pip install /PATH/<文件名>
pip install --use-wheel --no-index --find-links=wheelhouse/ <包名>
pip install --no-index -f=/PATH/ <包名>

# 下载而不安装
pip install <包名> -d <目录>
pip install -d <目录> -r requirements.txt

# 显示包所在目录
pip show -f <包名>

# 搜索包
pip search <搜索关键字>

# 打包
pip wheel <包名>
```

说明：

requirements.txt内容格式为:

\```

APScheduler==2.1.2

Django==1.5.4

MySQL-Connector-Python==2.0.1

MySQL-python==1.2.3

PIL==1.1.7

South==1.0.2

django-grappelli==2.6.3

django-pagination==1.0.7

\```

## Pycharm

### 简介

PyCharm是一种PythonIDE，带有一整套可以帮助用户在使用Python语言开发时提高其效率的工具，比如调试、语法高亮、Project管理、代码跳转、智能提示、自动完成、单元测试、版本控制。此外，该IDE提供了一些高级功能，以用于支持Django框架下的专业Web开发。

官网http://www.jetbrains.com/pycharm

### 下载地址

Windows: http://www.jetbrains.com/pycharm/download/#section=windows

Linux: http://www.jetbrains.com/pycharm/download/#section=linux

Mac: http://www.jetbrains.com/pycharm/download/#

### 快捷键

help目录下可以找到ReferenceCard.pdf

Ctrl+t SVN更新

Ctrl+k SVN提交

Ctrl + Alt + I 自动缩进行

Ctrl + Shift + J 合并行

Ctrl + Shift + F 全局查找

Ctrl + Shift + R 全局替换

CTRL q: 在参数列表位置，显示可以输入的所有参数。

CTRL q: 查看选中方法的文档字符串

ctrl+w 选中单词

ctrl+y  删除当前行

Ctrl + Delete 删除到字符结束

Ctrl + Backspace 删除到字符开始

shift+o 自动建议代码补全

Ctrl+Enter 补全

F3 下一个

Shift + F3 前一个

Ctrl + R 替换

### 修改模板

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# __Author__: xiaofei
```

![img](Python%E7%8E%AF%E5%A2%83.assets/16bc1153-3daf-49c8-9381-24c2376b29f2.jpg)