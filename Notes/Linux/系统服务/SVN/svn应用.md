[TOC]

# svn客户端

## win

TortoiseSVN-1.10.0.28176-x64-svn-1.10.0.msi（svn客户端）

LanguagePack_1.10.0.28176-x64-zh_CN.msi（TortoiseSVN 的汉化包）

下载地址：http://subversion.apache.org/packages.html#windows

客户端安装包和汉化包安装时如无特殊需求一直默认下一步即可。

安装成功以后在鼠标右键会出现快捷方式。

![image-20200319171803593](svn%E5%BA%94%E7%94%A8.assets/image-20200319171803593.png)

打开客户端程序，设置语言

![image-20200319171857307](svn%E5%BA%94%E7%94%A8.assets/image-20200319171857307.png)

![image-20200319171907867](svn%E5%BA%94%E7%94%A8.assets/image-20200319171907867.png)

### 上传项目

1，svn服务器没有项目，需要从本地上传，右击将要上传的项目目录

TortoiseSVN --> Import

![image-20200319172352556](svn%E5%BA%94%E7%94%A8.assets/image-20200319172352556.png)

在弹出的对话框中填上版本库URL，这个URL可以从VisualSVN Server Manager中获取，在你的版本库上点击Copy URL to Clipboard 就可以复制到路径：

![image-20200319172413521](svn%E5%BA%94%E7%94%A8.assets/image-20200319172413521.png)

将复制的url粘贴上，在url后边加上一个自定义路径，一般与项目同名。

![image-20200319172545133](svn%E5%BA%94%E7%94%A8.assets/image-20200319172545133.png)

![image-20200319172624669](svn%E5%BA%94%E7%94%A8.assets/image-20200319172624669.png)

![image-20200319172636868](svn%E5%BA%94%E7%94%A8.assets/image-20200319172636868.png)

![image-20200319172649061](svn%E5%BA%94%E7%94%A8.assets/image-20200319172649061.png)

上传完成，仓库的目录

![image-20200319172703899](svn%E5%BA%94%E7%94%A8.assets/image-20200319172703899.png)

检出项目

![image-20200319172719956](svn%E5%BA%94%E7%94%A8.assets/image-20200319172719956.png)

![image-20200319172735078](svn%E5%BA%94%E7%94%A8.assets/image-20200319172735078.png)

检出完成后，会显示绿色的对勾。

![image-20200319172746091](svn%E5%BA%94%E7%94%A8.assets/image-20200319172746091.png)

### 下载项目

如果SVN服务器上已经有项目存在，需要下载到本地开发。

建立一个 test的工作目录，其实就是用来存放代码的地方。进入test目录，空白处右键选择 "SVN checkout"。

![image-20200319183050933](svn%E5%BA%94%E7%94%A8.assets/image-20200319183050933.png)

选择OK，如果拉取成功，会在本地的test目录下生成一个.svn为目录。



### SVN目录

为了便于创建分支和标签，我们习惯于将Repository版本库的结构布置为:/branches,/tags,/trunk。

Branches代表分支   上线的项目有bug 需要修改而正常开发的项目已经在开发新功能

Tags 标签 存放上线的版本 v1.0, v1.2......  只做保存，不做修改

Trunk 主干 存放正常开发的项目

![image-20200319172924941](svn%E5%BA%94%E7%94%A8.assets/image-20200319172924941.png)



## linux

linux上是通过svn命令来提交检出仓库代码的。

```shell
# 帮助
[root@vcs ~]# svn help

# 将svn服务器上p1项目down到本地当前目录, 指定版本试用: -r 版本号
[root@vcs ~]# svn co svn://172.16.2.10/p1 --username=user01 --password=user01pwd

# 在本地开始编辑源代码
[root@vcs ~]# vim code1
"""
hello,world.
"""

[root@vcs ~]# svn add code1

# 将改动的文件提交到版本库,不指定文件则提交当前所有,支持正则
[root@vcs ~]# svn commit code1 -m "code-v1"

# 加锁/解锁
[root@vcs ~]# svn lock -m "lock test file" test.txt
[root@vcs ~]# svn unlock test.txt

# 将版本库中的code1还原大版本10,如果不指定目录,则更新所有
[root@vcs ~]# svn update -r 10 code1

# 更新,与版本库同步,如果在提交的时候提示过期的话,是因为冲突,需要先update,修改文件,然后清除svn resolved,最后再提交commit,简写：svn up
[root@vcs ~]# svn update code1 

# 删除
[root@vcs ~]# svn delete svn://172.16.2.10/pro1/domain/code1 -m "delete file"
[root@vcs ~]# svn delete code1
[root@vcs ~]# svn ci -m "delete file"

# 查看code所有修改日志
[root@vcs ~]# svn log code1

# 查看详细信息
[root@vcs ~]# svn info code1

# 比较当前版本与原始版本差异
[root@vcs ~]# svn diff code1

# 比较m与n版本的差异
[root@vcs ~]# svn diff -r 200:201 code1

# 将2个版本之间的修改合并到当前,一般会有冲突,需要额外处理
[root@vcs ~]# svn merge -r 200:205 code1
```



# 备份

## 全量备份

```shell
[root@vcs ~]# curr=`svnlook youngest /data/svn/project/` # 此处是查询工程目录的最新版本
[root@vcs ~]# svnadmin dump /data/svn/repos/test --revision 0:$cur --incremental >0-"$curr"svn.bak 
[root@vcs ~]# echo $curr >/tmp/svn_revision
```



## 增量备份

```shell
[root@vcs ~]# old=`cat /tmp/svn_revision`
[root@vcs ~]# new=`svnlook youngest /data/svn/project/`
[root@vcs ~]# svnadmin dump /data/svn/repos/test --revision $old:$new --incremental >$old"-"$new"svn.bak 
```



# 恢复

恢复顺序从低版本逐个恢复到高版本；即，先恢复最近的一次完整备份，然后恢复紧挨着这个文件的增量备份。

```shell
[root@vcs ~]# cd /data/svn/repos/ 
[root@vcs ~]# svnadmin create test2 
[root@vcs ~]# svnadmin load test2 < /data/svnback/20110719/0-1112svn.bak 
[root@vcs ~]# svnadmin load test2 < /data/svnback/20110719/1113-1120svn.bak
```

服务端在svn仓库有变动时自动更新代码脚本

```shell
#!/bin/sh  
REPOS="$1"  
REV="$2"  
export LC_ALL="zh_CN.UTF-8"  
export LANG="en_US.UTF-8"  
  
SVN_PATH=/usr/bin # svn安装路径  
WEB_PATH=/web/ccb # web项目所在  
SVN_USER=admin # svn用户名  
SVN_PASS=admins # svn密码  
LOG_PATH=/tmp/svn.log 
$SVN_PATH/svn update $WEB_PATH --username $SVN_USER --password $SVN_PASS --no-auth-cache >> $LOG_PATH  
exit 0
```

done

