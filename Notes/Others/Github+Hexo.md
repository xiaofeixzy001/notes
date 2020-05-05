[TOC]

# Github

注册github帐号

https://github.com

如果打开访问慢，可通过修改本地hosts文件。

- http://tool.chinaz.com/dns或其他DNS解析网站
- 解析github官网https://github.com
- 多检测几遍，将TTL值最小的ip，写入hosts文件

注册Github的时候一定要选择一个合适的名字，因为后来博客网站的域名也会用到这个名字。

当然，如果有自己的域名，可以任意，后期也可以更改。

# Nodejs

下载地址：http://nodejs.cn/download

测试安装：命令行使用node -v 、npm -v，查看显示版本号即成功。

Hexo基于Node.js环境，Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境，可以在非浏览器环境下，解释运行 JS 代码。

# Git

下载地址：https://git-scm.com/downloads

Windows系统需下载，Mac系统因为自带Git无需操作。

测试安装：git  - -version，查看显示版本号即成功。

# 创建github仓库

在github上创建一个仓库，用于存储博客相关代码.

注意仓库的名字，一般是以"UserName" + “github.io”的形式命名。

这样之后我们最后的博客网站的链接就会是：https://UserName.github.io

Enforce HTTPS选项如果不能勾选，稍等一会在刷新试试，因为证书的申请需要一点点时间。

## 添加ssh key

windows下，默认在用户家目录下，创建.ssh文件夹。

打开Git bash窗口，执行如下命令：

```shell
ssh-genkey -t rsa -C "1023668666@qq.com"
cat id_rsa.pub  # 将此文件内容复制下来
ssh -T git@github.com  # 测试

git config --global user.email "1023668666@qq.com"
git config --global user.name "xiaofei"
```

-C 指定github注册时邮箱。

将id_rsa.pub的内容复制到github中。

头像处 -- settings -- SSH and GPG keys -- New SSH key

起个任意名，下面粘贴id_rsa.pub内容

# Hexo部署

Hexo是高效的静态站点生成框架，基于Node.js。

通过Hexo可以轻松地使用 Markdown 编写文章，除了 Markdown 本身的语法之外，还可以使用 Hexo 提供的标签插件来快速的插入特定形式的内容，而且相对于其他框架，Hexo在速度上也有很大优势。

在硬盘的某个位置，新建一个目录，鼠标右键，点击Git Base Here，执行如下命令：

```shell
pwd
npm install -g hexo-cli  # 安装hexo框架
hexo init  # 初始化
npm install
hexo g  # 生成博客
hexo s  # 启动服务,默认4000端口,指定端口运行hexo server -p 端口号
```

测试访问：localhost:4000

如果无法访问，查看一下theme目录下有没有主题目录。

## 关联github博客

```shell
vim Hexo/_config.xml
"""
deploy:
  type: git
  repository: git@github.com:xiaofeixzy001/xiaofeixzy001.github.io.git
  branch: master
"""

npm install hexo-deployer-git —save # 安装部署插件
hexo d  # 部署到github
```



## 使用自己的域名

进入新建的仓库，Setting进入设置页面。

找到GitHub Pages标签，Custom domain填写自己的域名，如果没有忽略此步骤。

然后登录自己的域名所在管理平台，比如阿里云。

添加解析记录，github的ip地址可以通过 `ping UserName.github.io `获取

在Hexo\source\目录下创建一个CNAME文件，里面写上自己希望对外访问的域名

```shell
hexo clean
hexo generate
hexo deploy
```

检查并测试访问。

# 安装主题

next主题

`git clone https://github.com/theme-next/hexo-theme-next themes/next`

bubuzou主题

`git clone https://github.com/Bulandent/hexo-theme-bubuzou themes/bubuzou`

bubuzou主题需要的依赖：

```shell
npm install --save hexo-renderer-jade hexo-generator-feed hexo-generator-sitemap hexo-browsersync hexo-generator-archive

npm install --save hexo-deployer-git hexo-generator-json-content hexo-generator-search
```

编辑_config.yml

```shell
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: 听风起雨落
subtitle: ''
description: ''
keywords:
author: 暴风
language: CN
timezone: 'Aisa/Shanghai'

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
pretty_urls:
  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
  trailing_html: true # Set to false to remove trailing '.html' from permalinks

# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link:
  enable: true # Open external links in new tab
  field: site # Apply to the whole site
  exclude: ''
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace: ''
  wrap: true
  hljs: false

# Home page setting
# path: Root path for your blogs index page. (default = '')
# per_page: Posts displayed per page. (0 = disable pagination)
# order_by: Posts order. (Order by date descending by default)
index_generator:
  path: ''
  per_page: 10
  order_by: -date

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Metadata elements
## https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta
meta_generator: true

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss
## Use post's date for updated date unless set in front-matter
use_date_for_updated: false

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Include / Exclude file(s)
## include:/exclude: options only apply to the 'source/' folder
include:
exclude:
ignore:

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
#theme: landscape
theme: bubuzou  # 此处改为themes目录下的主题目录名

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: git@github.com:xiaofeixzy001/xiaofeiblog.github.io.git
  branch: master
```

然后重新执行

hexo clean

hexo g

hexo d

测试



## hexo常用命令

hexo new "postName" #新建文章

hexo new page "pageName" #新建页面

hexo generate #生成静态页面至public目录

hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）

hexo deploy #部署到GitHub

hexo help  # 查看帮助

hexo version  #查看Hexo的版本

缩写：

hexo n == hexo new

hexo g == hexo generate

hexo s == hexo server

hexo d == hexo deploy

组合命令：

hexo s -g #生成并本地预览

hexo d -g #生成并上传