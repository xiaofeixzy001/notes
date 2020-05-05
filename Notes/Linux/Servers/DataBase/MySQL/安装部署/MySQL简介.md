[TOC]

# 一，概述

## 1.1 MySQL介绍

MySQL属于传统关系型数据库产品，它开放式的架构使得用户选择性很强，同时社区开发与维护人数众多。其功能稳定，性能卓越，且在遵守GPL协议的前提下，可以免费使用与修改，也为MySQL的推广与使用带来了更多的利好。在MySQL成长与发展过程中，支持的功能逐渐增多，性能也不断提高，对平台的支持也越来越多。
MySQL是一种关系型数据库管理系统，关系型数据库的特点是将数据保存在不同的表中，再将这些表放入不同的数据库中，而不是将所有数据统一放在一个大仓库里，这样的设计增加了MySQL的读取速度，而且灵活性和可管理性也得到了很大提高。访问及管理MySQL数据库的最常用标准化语言为SQL结构化查询语言。

MySQL官网下载地址：

http://www.mysql.com/downloads/

https://dev.mysql.com/downloads/mysql/

## 1.2 MySQL特点

Mysql是开源的，所以你不需要支付额外的费用。

Mysql支持大型的数据库。可以处理拥有上千万条记录的大型数据库。

MySQL使用标准的SQL数据语言形式。

Mysql可以允许于多个系统上，并且支持多种语言。这些编程语言包括C、C++、Python、Java、Perl、PHP、Eiffel、Ruby和Tcl等。

Mysql对PHP有很好的支持，PHP是目前最流行的Web开发语言。

MySQL支持大型数据库，支持5000万条记录的数据仓库，32位系统表文件最大可支持4GB，64位系统支持最大的表文件为8TB。

Mysql是可以定制的，采用了GPL协议，你可以修改源码来开发自己的Mysql系统。

## 1.3 MySQL产品线

Alpha版: 内部运行测试版本,不对外公开

Beta版: 有着完整的功能,并且进行了所有的测试的版本,属于内测版本,会邀请或提供给一些用户进行体验测试

RC版: 生产环境发布之前的一个小版本,是Beta版修复bug或缺陷后的版本

GA版: 正式发布的版本,可用于生产环境(常用)

## 1.4 MySQL版本号

如无特别说明，都是以linux系统为基础。

早期：4.1.7和5.0.56等.

在发展到5.1系列版本之后,重新规划为三条产品线：

- 5.0 - 5.1

继续完善与改进原有功能与性能,同时增加新功能,属于mysql早期产品的延续系列

5.1是当前稳定版本,只针对漏洞修复重新发布,没有影响性能的新功能 

5.0是前一个稳定版本,只针对漏洞修复重新发布,没有影响性能的新功能 

4.0和3.23是旧的稳定版本, 该版本不再使用,新的发布只用来修复特别严重的漏洞

- 5.4 - 5.7

整合MySQL AB 公司社区和第三方开发的新存储引擎,吸收新的实现算法等,从而更好的支持SMP架构，5.5为常用稳定版本

- 6.0.xx-7.3.xx

为了更好的推广MySQL CLuster版本而推出的生产线版本,因出来的晚,目前很少有人使用。

## 1.5 版本分类

在这个下载界面会有几个版本的选择:

1, MySQL Community Server 社区版本，开源免费，但不提供官方技术支持;

2, MySQL Enterprise Edition 企业版本，需付费，可以试用30天;

3, MySQL Cluster 集群版，开源免费。可将几个MySQL Server封装成一个Server;

4, MySQL Cluster CGE 高级集群版，需付费;

5, MySQL Workbench（GUI TOOL）一款专为MySQL设计的ER/数据库建模工具;

MySQL Community Server 是开源免费的，这也是我们通常用的MySQL的版本。

访问 https://dev.mysql.com/downloads/mysql/

选择 Generally Available（GA）Release 

## 1.6 版本选择

企业生产场景选择MySQL数据库的建议：

- 稳定版选择开源社区版的稳定版GA版本；

- 产品线,可选择5.1或5.5,互联网公司5.5,其次是5.1和5.6；

- 选择MySQL数据库GA版发布后6个月以上的版本；

- 选择前后几个月没有大的BUG修复的版本,而不是大量修复BUG后的版本；

- 建议选择较长时间没有更新发布的版本；

- 要考虑开发人员开发程序使用的版本是否兼容；

- 作为内部开发测试数据库环境,跑3-6个月的时间；

- 优先企业内非核心业务采用新版本；

- 虚心请教技术圈内的DBA师兄, 推荐用过的好用的GA版本；
- 以上筛选工作后,若没有重要BUG和性能瓶颈,则可以开始考虑生产环境部署；

## 1.7 win包说明

mysql-5.5.19-win32.msi：windows完整安装包，包含安装程序和配置向导，有MySQL文档;

mysql-5.5.19.zip：这个是Mysql源码压缩包，需要编译;

mysql-5.5.19-win32.zip：是非安装的zip压缩包，没有自动安装程序和配置向导，需手动安装配置，有MySQL文档;

mysql-essential-5.1.60-win32.msi：是精简版，如果只需要mysql服务，就选择此版本;

"essentials"是指精简版，不包含 embedded server and benchmark suite，有自动安装程序和配置向导，没有MySQL文档;

"noinstall"是指非安装的压缩包的, 包含 embedded server and benchmark suite，没有自动安装程序和配置向导，需手动安装配置，有MySQL文档;

一般做后台开发，我们就下载 mysql-5.5.19-win32.msi 安装即可.

# 二，分支与变种

MySQL的第一个版本3.23是由瑞典的MySQL AB公司发行的，后来被SUN公司收购，然后又被Oracle公司收购。这中间的故事太多可讲。

Oracle官方对于MySQL有企业版和社区版，但是两个版本之间并没有什么改变。只是企业版提供MySQL监控程序和技术支持但收费也是挺贵，Oracle公司两年使MySQL企业版的费用翻了4倍，MySQL一直是oracle数据库的强大对手之一，这其中Oracle到底想什么猫腻自行想象。

MySQL在两次转让的过程中，出现了好几个MySQL变种。主要有Percona server，MariaDB和Drizzle。它们都有活跃的用户社区和某种程度上的商业支持，均由独立的服务供应商支持。

## 2.1 Percona server

Percona server是由一家商业公司运作，当然也是开源的但跟MySQL商业版一样提供服务，技术支持，咨询等。Percona server是在源代码的基础上加以修改，其主要目标是对MySQL的性能和操作灵活性加以提升。

Percona server是个与MySQL向后兼容的替代品，它尽可能不改变SQL语法、客户端/服务器协议和磁盘上的文件格式。任何运行在MySQL上的都可以运行在percona server上而不需要修改。切换到percona只需要关闭MySQL和启动percona即可，不需要导出和重新导入数据。

Percona server包括percona XtraDB存储引擎，即改进版的InnoDB。这同样是个向后兼容的替代品。例如，如果创建一个使用innodb存储引擎的表，percona server能自动识别并用percona XtraDB替代之。Percona XtraDB同样包括在MariaDB内。其次就是percona server的一些改进已经包括在MySQL的oracle版本中，只是percona server在5.5版本中的许多改进可能要在MySQL 5.6中才会实现。

## 2.2 MariaDB

在Sun收购MySQL后，Monty Widenius这位MySQL的创建者，因不认同MySQL开发流程而离开Sun。他成立了Monty程序公司并创立了MariaDB数据库，没办法牛人就牛人，说开发就开发一个数据库。更屌的是Monty大神一共开发了三个数据库且都是以自己孩子的名字命名的，他女儿叫My用在了MySQL上、儿子叫Max名字用在了跟SAP公司合作开发的数据MaxDB上、现在的MariaDB同样是以小孙女名字Maria命名的。Monty公司的理念是以培养一个开放的开发环境以鼓励外部的参与，并且在Red Hat的7.0版本发行中已经使用MariaDB替换MySQL了。但是MySQL和MariaDB内部机制没有变化，所以学习MySQL也就是在学习MariaDB，只是MariaDB的开发者都是以前的MySQL开发者，所以更有优势。据说现在国外有很多公司已经开始转MariaDB了。

MariaDB有什么不同呢？与percona server相比，它包括了更多对服务器的扩展（percona的大部分改变是在于percona XtraDB的存储引擎，而不是服务器层）。例如，有许多是对查询优化和复制的改变。它使用Aria存储引擎取代了MyISAM来存储内部临时表。Aria最初叫Maria。除了Percona XtraDB和Aria外，MariaDB还包括许多社区的存储引擎，例如SphinxSE和PBXT。

MariaDB还是使用的InnoDB存储引擎（10.2换回InnoDB），随着MySQL与MariaDB版本跨度越来越大，其区别也就越来越多了。具体还是要看官方信息。

官方地址为：https://mariadb.org/ 

## 2.3 国内第三方发行版

AliSQL：来自阿里巴巴阿里云RDS团队。

TXSQL：来自腾讯。

InnoSQL：来自网易。

OneSQL：来自平民软件



# 三，扩展资料

## 3.1 MySQL企业版

MySQL Enterprise，MySQL企业版是一个已被证明和值得信赖的平台，这个平台包含了MySQL企业级数据库软件,、监控与咨询服务，以及确保您的业务达到最高水平的可靠性、安全性和实时性的技术支持。

MySQL企业版包括：

MySQL企业级服务器，这是全球最流行的开源数据库最可靠、最安全的最新版本。

MySQL企业级系统监控工具，它可以提供监控和自动顾问服务，以此来帮助您消除安全上的隐患、改进复制、优化性能等。

MySQL技术支持，可以使您最棘手的技术问题得到快速解答。

MySQL咨询支持，只有购买了MySQL企业级银质或金质服务的客户才能得到此项支持。 MySQL技术支持团队将为您的系统提供针对性的建议，告诉您如何恰当地设计和调整您的MySQL服务器、计划、查询和复制设定，以获得更好的性能。 

## 3.2 MySQL社区版

MySQL Community Server，MySQL公司一直专注于向开源社区发布全球最流行的开源数据库——MySQL Community Server。

在开源GPL许可证之下可以自由的使用。

## 3.3 企业版和社区版区别

2006年底，MySQL开始发行MySQL Enterprise，这个产品包含了一系列更健全的提高MySQL server可靠性、安全性和性能的服务。

为了更好的了解MySQL企业版和社区版之间的区别，可以在下面的表格中得到信息：  

如果您的业务符合以下任何一个需求特征，那么推荐您采用MySQL企业版解决方案.

![img](MySQL%E7%AE%80%E4%BB%8B.assets/57cdda5f-5b11-4941-805f-8258fabeadd1-1578556455894.png)

其他回答：

MySQL社区版是开源的GPL许可，可以免费获取。

MySQL网络版是通过MySQL认证的许可，需要花钱购买。

MySQL网络版在网络和企业部署功能、排错功能、技术和产品支持、升级更新、享有MySQL知识库、直接得到MySQL开发人员指导等方面，都是MySQL社区版所没有的。

第一个 MySQL Community Server，这个不要钱！

第二个 MySQL Enterprise 这个要掏钱，不过可以打电话咨询问题，也就是电话技术支持。

第三个 MySQL Cluster，这个单独是没法用的，要在1或2的基础上用。当然用来平衡多台数据库的。

第四个 MySQL Workbench，这是个好东西，用来设计数据库的。erwin知道吗？他就是这个作用。

MySQL Community Server 社区版本 应该不提供官方技术支持

MySQL Enterprise Server MySQL企业版服务器

软件是最可靠、最安全、更新版本的MySQL企业级服务器数据库，它能够高性价比地提供电子商务、联机事务处理(OLTP)、千兆规模的数据仓库应用等。它支持ACID事务处理，能提供完整的提交、回滚、崩溃恢复和行级锁定功能。MySQL数据库因其易用性、可扩展性和高性能等特点，成为全球最流行的开源数据库。

MySQL Cluster 2台以上 mysql集群服务器

MySQL Workbench有两个版本：

MySQL Workbench Community Edition（又叫MySQL Workbench OSS，社区版）和MySQL Workbench Standard Edition（又叫MySQL Workbench SE，商业版）:

MySQL Workbench OSS是在GPL证书下发布的开源社区版本;

MySQL Workbench SE则是按年收费的商业版本。其功能方面有差异

（不得不说，数据库/模型同步这一重要的功能竟然只在收费的MySQL Workbench SE中可用，而在DBDesigner4中这却是基本功能，这种“继任”方式实在让人恶心.）

# 四，客户端工具

## 4.1 图形客户端

phpMysqlAdmin

Workbench

Mysql Front

Navicat for Mysql

Toad

## 4.2 MySQL GUI Tools

MySQL GUI Tools一个可视化界面的MySQL数据库管理控制台，提供了四个非常好用的图形化应用程序，方便数据库管理和数据查询。这些图形化管理工具可以大大提高数据库管理、备份、迁移和查询效率，即使没有丰富的SQL语言基础的用户也可以应用自如。它们分别是：

MySQL Migration Toolkit：数据库迁移

MySQL Administrator：MySQL管理器

MySQL Query Browser：用于数据查询的图形化客户端

MySQL Workbench：DB Design工具

### 4.2.1 特性

支持插件式存储引擎,插件式存储引擎也称之为'表类型'

1,更多的存储的存储引擎MyISAM和InnoDB

2,诸多扩展和新特性

3,提供了较多测试组件

4,开源
