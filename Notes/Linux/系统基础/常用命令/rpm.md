[TOC]


# rpm管理工具
rpm [option] /path/to/package_file

# 选项

-vv|-vvv 显示详细安装信息

-h 以#号显示程序包管理执行进度,每个#表示2%的进
-e package_name :卸载
-q package_name1 package_name2 :查询某软件是否安装，可一次查询多个，空格隔开
-ivh 安装软件包，显示安装信息
-qa 查询已经安装的软件 
-U 升级,无论是否有老版本，都安装最新版本程序

-F 升级,必须有老版本才会升级，否则不会,--force,强制升级
-qf 查看文件隶属于哪个软件包
-qi 查看已安装软件包的信息

-qip查看未安装软件包的信息
-ql 查看已经安装的软件包在系统中安装了哪些文件
-qlp查看未安装的软件将要安装哪些软件
-qd 查看软件包帮助文档
-qc 查看软件包的配置文件

--nodeps 忽略依赖关系

--excludedocs 不安装软件包中的文档文件
--prefix (PATH路径) 指定软件包安装路径
--test 测试安装,不会真正执行安装过程:dry run模式
--replacepkgs 重新安装
--replacefiles 忽略错误信息

--nosignature 不检查来源合法性

--nodigest 不检查包完整性

-q --scripts 软件名 查看程序包的相关脚本

--noscripts 不执行程序包脚本片段
脚本有四类：
%pre:安装前脚本,--nopre

%post:安装后脚本,--nopost

%preun:卸载前脚本,--nopreun

%postun:卸载后脚本,--nopostun

如果对某个程序或服务不知道是干啥的，可以运用rpm来查看帮助信息,比如vncserver服务
rpm -qf vncserver :查看该服务所属软件包
rpm -qi vnc-server :查看该软件包的信息
rpm -V 软件名  :校验,当前与默认做比较有了哪些改变
rpm2cpio :解压所有文件到当前目录
大多数安装包的示例文档名称都会带有example

注意:

1,在升级软件包时，有时不同版本间的配置文件可能会不太兼容。RPM使用 .rpmnew 和 .rpmsave 来处理这种情况。有时，新版本的配置文件会保存为 .rpmnew ；有时，会把老版本的配置文件保存为 .rpmsave
处理 .rpmnew 和 .rpmsave 文件的方法为：

a,比较新旧配置文件的不同

b,参照新的配置文件手动修改

c,删除旧的配置文件
2,不要对内核进行升级操作，而是安装，因为系统允许多内核并存，新老内核均可使用。  

3,包来源合法性及完整性检验,
安装时自动验证来源合法性

前提：在当前系统导入包的制作者的公钥
导入：
  rpm --import /path/to/key_file (系统光盘的公钥在/mnt/RPM-GPG-KEY-CentOS-6)
  rpm -qa gpg-pubkey* 查看当前系统已导入的所有以gpg开头的公钥
  rpm -qi gpg-pubkey-NAME 查看某个密钥NAME的详细信息

手动检查：
rpm -K /path/to/package_file
rpm --checksig /path/to/package_file
不检查包完整性
  rpm -K --nodigest
不检查来源合法性：
  rpm -K --nosignature



# 数据库重建
数据库目录：CentOS默认路径为 /var/lib/rpm
重建方法：
  rpm --initdb：初始化,如果没有库，则会新建一个，如果有则不重建
  rpm --rebuilddb：重建,直接重建数据库，覆盖原有