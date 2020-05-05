[TOC]

# 前言

最原始的版本控制方式:

```
毕业论文_初稿.doc
毕业论文_修改1.doc
毕业论文_修改2.doc
毕业论文_修改3.doc
毕业论文_完整版1.doc
毕业论文_完整版2.doc
毕业论文_完整版3.doc
毕业论文_最终版1.doc
毕业论文_最终版2.doc
毕业论文_死也不改版.doc
...
```

这种方式有显著的缺点:

1,多个文件,保留所有版本时,需要为每个版本保存一个文件;

2,协同操作,多人协同操作时,需要将文件打包发来发去;

3,容易丢失,被删除意味着永远失去,可选择网盘,但是会受到网络影响;

那么如果有一个软件,不但能自动帮我记录每次文件的改动,还可以让同事协作编辑,这样就不用自己管理一堆类似的文件了,也不需要把文件传来传去.如果想查看某次改动,只需要在软件里瞄一眼就可以,再有如果改错还可以还原到某个版本,岂不是很方便？

为了解决以上版本控制存在问题,应运而生了一批版本控制工具:VSS、CVS、SVN、Git等,其中Git属于绝对霸主地位.

一般版本控制工具包含两部分:

本地客户端: 本地编写内容以及版本记录;

远程服务端: 将内容和版本记录同时保存在远程服务器上(也可保存在本地服务器)

版本控制主要实现2个功能:

1,版本管理:

在开发中，这是刚需，必须允许可以很容易对产品的版本进行任意回滚，版本控制工具实现这个功能的原理简单来讲，就是你每修改一次代码，它就帮你做一次快照;

2,协作开发:

一个复杂点的软件,往往不是一个开发人员可以搞定的,公司为加快产品开发速度,会招聘一堆跟你一样的开发人员开发这个产品,拿微信来举例,现在假设3个人一起开发微信,A开发联系人功能,B开发发文字、图片、语音通讯功能,C开发视频通话功能,B和C的功能都是要基于通讯录的,你说简单,直接把A开发的代码copy过来,在它的基础上开发就好了,可以,但是你在他的代码基础上开发了2周后,这期间A没闲着,对通讯录代码作了更新,此时怎么办?你和他的代码不一致了,此时我们知道,你肯定要再把A的新代码拿过来替换掉你手上的旧通讯录功能代码,现在人少,3个人之间沟通很简单,但想想,如果团队变成30个人呢?来回这样copy代码,很快就乱了,所以此时需一个工具,能确保一直存储最新的代码库,所有人的代码应该和最新的代码库保持一致.

# 常见版本管理工具

## VSS (Visua Source Safe)

Microsoft提供,使用普遍,可与VS.net进行无缝集成,适用于windows平台上开发的中小型企业,但规模较大后,性能很差,对分支开发和并行开发的支持有限;

## CVS (Concurrent Versions System)

开源工具,与SVN同一个厂家:Collab.Net

CVS源于unix的版本控制工具,对于CVS的安装和使用要求对unix系统有所了解,CVS的服务器端管理需要进行各种命令行操作,目前,cvs的客户端有win-CVS的图形化界面,服务器端也有CVSNT的版本,易用性正在改善提高;

此工具是相当著名,使用得相当广泛的版本控制工具之一,使用成熟的"Copy-Modify-Merge"开发模型,可以大大的提高开发效率,适合于项目比较大,产品发布频繁,分支活动频繁的中大型项目;

## SVN (CollabNet Subversion)

此工具是在CVS的基础上,由CollabNet提供开发的,也是开源工具,应用比较广泛;

他修正cvs的一些局限性,适用范围同cvs,目前有一些基于SVN的第三方工具,如TortoiseSVN,是其客户端程序,使用的也相当广泛;

在权限管理,分支合并等方面做的很出色,他可以与Apache集成在一起进行用户认证;

不过在权限管理方面目前还没有个很好用的界面化工具,SVNManger对于已经使用SVN进行配置的项目来说,基本上是无法应用的,但对于从头开始的项目是可以的,功能比较强大,但是搭建svnManger比较麻烦;

svn是一个跨平台的软件,支持大多数常见的操作系统.作为一个开源的版本控制系统,Subversion管理着随时间改变的数据,这些数据放置在一个中央资料档案库中,这个档案库很像一个普通的文件服务器,不过它会记住每一次文件的变动,这样你就可以把档案恢复到旧的版本,或是浏览文件的变动历史.

Subversion是一个通用的系统,可用来管理任何类型的文件,其中包括了程序源码.

## GIT

因为最初是从Linux起家的,非常依赖文件系统的一些特性,这些在Linux下表现的很好,而Windows下特别糟糕;

Git是一个开源的分布式版本控制系统,用以有效、高速的处理从很小到非常大的项目版本管理;

Git是Linus Torvalds为了帮助管理Linux内核开发而开发的一个开放源码的版本控制软件;

Torvalds开始着手开发Git是为了作为一种过渡方案来替代BitKeeper,后者之前一直是Linux内核开发人员在全球使用的主要源代码工具.

开放源码社区中的有些人觉得BitKeeper的许可证并不适合开放源码社区的工作,因此Torvalds决定着手研究许可证更为灵活的版本控制系统.尽管最初Git的开发是为了辅助Linux内核开发的过程,但是我们已经发现在很多其他自由软件项目中也使用了Git.

## BitKeeper

是由BitMover公司提供的,BitKeeper自称是"分布式"可扩缩SCM系统;

不是采用C/S结构,而是采用P2P结构来实现的,同样支持变更任务,所有变更集的操作都是原子的,与svn,cvs一致.

除了免费的外,还有收费的集中式版本控制系统,比如IBM的ClearCase(以前是Rational公司的,被IBM收购了),特点是安装比Windows还大,运行比蜗牛还慢,能用ClearCase的一般是世界500强,他们有个共同的特点是财大气粗,或者人傻钱多;

微软自己也有一个集中式版本控制系统叫VSS,集成在Visual Studio中,由于其反人类的设计,连微软自己都不好意思用了;

分布式版本控制系统除了Git以及促使Git诞生的BitKeeper外,还有类似Git的Mercurial和Bazaar等.这些分布式版本控制系统各有特点,但最快,最简单也最流行的依然是Git！

## 扩展

### 集中式和分布式对比

CVS和SVN都是集中式的版本控制系统,而Git是分布式版本控制系统

集中式版本控制系统,版本库是集中存放在中心服务器上,客户端需先从服务器下载最新版本的代码,在本地修改更新完成后再推送给中心服务器.

![img](git%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/302db82e-2d45-40fb-9922-84da3f935565.jpg)

最大的缺点就是必须联网才能工作,本地局域网还好,如果是互联网,则会受到多方因素影响.

分布式版本控制系统,没有'中心服务器',每个客户端都是一个完整版本库,所以有没有联网都无所谓了.多人协作时,比如我改动了文件A,同事在他电脑上也改动了A,这时仅需将各自修改的部分推送给对方,就可以相互查看并选择性处理了.

与集中式相比,分布式的安全性要高很多,集中式的中心服务器如果down掉,所有人都没办法干活了.

实际工作中使用分布式版本控制系统的时候,通常不会两方相互推送,而是搭建一台'中心Git服务器'作为一个推送平台,其作用仅仅是'方便交换'大家的修改,就算down掉了也不影响原始版本.

![img](git%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/261c5127-f4c1-491f-b30a-d09cf3d63903.jpg)

# Git简介

git,是一个开源的分布式版本控制软件,用以有效高速的处理从很小到非常大的项目版本管理;

git,最初是由Linus Torvalds设计开发的,用于管理Linux内核开发;

git,是根据GNU通用公共许可证版本2的条款分发的自由/免费软件;

安装参见：http://git-scm.com/

[中文版权教程](http://git.oschina.net/progit/)

## git发展史历史

linus在1991年创建了开源的linux,linux系统不断发展,已成为最大的服务器系统软件,其代码库已经十分庞大,linus不得不放弃手工管理源码方式,改为商业的版本控制系统BitKeeper.

起初,BitKeeper出于人道主义对linux社区授权免费使用,但后来在2005年,linux社区的牛人,Samba的开发Andrew试图破解BitKeeper,被BitKeeper公司发现,于是取消了免费授权,linus花了2周的时间自己用C写了一个分布式版本控制系统,这就是Git,一个月之内,linux系统的源码已经由Git管理了.

Git迅速成为最流行的分布式版本控制系统,尤其是2008年,GitHub网站上线了(github是一个基于git的代码托管平台,付费用户可以建私人仓库,我们一般的免费用户只能使用公共仓库,也就是代码要公开.),它为开源项目免费提供Git存储,无数开源项目开始迁移至GitHub,包括jQuery,PHP,Ruby等等.

今天,GitHub已是:

1,一个拥有143万开发者的社区,其中不乏Linux发明者Torvalds这样的顶级黑客,以及Rails创始人DHH这样的年轻极客;

2,这个星球上最流行的开源托管服务,目前已托管431万git项目,不仅越来越多知名开源项目迁入GitHub,比如Ruby on Rails、jQuery、Ruby、Erlang/OTP.近三年流行的开源库往往在GitHub首发,例如：BootStrap、Node.js、CoffeScript等;

3,alexa全球排名414的网站;

## github

GitHub是一个基于Git的远程文件托管平台（同GitCafe、BitBucket和GitLab等）;

Git本身完全可以做到版本控制，但其所有内容以及版本记录只能保存在本机，如果想要将文件内容以及版本记录同时保存在远程，则需要结合GitHub来使用;

从名字就可以看出，这个网站就是提供Git仓库托管服务的，所以，只要注册一个GitHub账号，就可以免费获得Git远程仓库

使用场景：

无GitHub：在本地 .git 文件夹内维护历时文件

有GitHub：在本地 .git 文件夹内维护历时文件，同时也将历时文件托管在远程仓库

其他---->

集中式：远程服务器保存所有版本，用户客户端有某个版本

分布式：远程服务器保存所有版本，用户客户端有所有版本

# Git安装配置

最早Git是在Linux上开发的,很长一段时间内,Git也只能在Linux和Unix系统上跑,不过,慢慢地有人把它移植到了Windows上.现在,Git可以在Linux、Unix、Mac和Windows这几大平台上正常运行了.

不同系统安装方式如下：

```shell
# for linux
yum install git -y

# for Debian & Ubuntu
sudo apt-get install git

# for windows
# 下载地址:https://git-scm.com/download/win

# for mac
# 方法1:安装homebrew,然后通过homebrew安装Git,具体方法请参考homebrew的文档:http://brew.sh/
# 方法2:从AppStore安装Xcode,Xcode集成了Git,不过默认没有安装,你需要运行Xcode,选择菜单"Xcode"->"Preferences",在弹出窗口中找到"Downloads",选择"Command Line Tools",点"Install"就可以完成安装了
```

安装完成后,可在开始菜单或鼠标右键里面找到Git Bash工具.

最后一步设置,设置全局个人信息,因为Git是分布式版本控制系统,所以,每个机器都必须自报家门:你的名字和Email地址.

你也许会担心,如果有人故意冒充别人怎么办?这个不必担心,首先我们相信大家都是善良无知的群众,其次,真的有冒充的也是有办法可查的.

```
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
```

注:

--global参数,表示你这台机器上所有的Git仓库都会使用这个配置,当然也可以对某个仓库指定不同的用户名和Email地址.

# 创建版本库

版本库又名仓库,英文名repository,你可以简单理解成一个目录,这个目录里面的所有文件都可以被Git管理起来,每个文件的修改或删除等操作,Git都能跟踪,以便任何时刻都可以追踪历史,或者在将来某个时刻可以"还原".

```
mkdir /data/git-test  # 创建git仓库存储目录
cd /data/git-test
git init  # 将当前目录变成git仓库
ls -a
ls -al .git/  # 生成的这个.git目录,就是git用来跟踪管理版本库的
"""
branches  config  description  HEAD  hooks  info  objects  refs
"""
```

.git/目录是Git来跟踪管理版本库的,没事千万不要手动修改这个目录里面的文件,不然改乱了,就把Git仓库给破坏了.

首先这里再明确一下,所有的版本控制系统,其实只能跟踪文本文件的改动,比如TXT文件,网页,所有的程序代码等等,Git也不例外.版本控制系统可以告诉你每次的改动,比如在第5行加了一个单词“Linux”,在第8行删了一个单词“Windows”,而图片、视频这些二进制文件,虽然也能由版本控制系统管理,但没法跟踪文件的变化,只能把二进制文件每次改动串起来,也就是只知道图片从100KB改成了120KB,但到底改了啥,版本控制系统不知道,也没法知道.

不幸的是,Microsoft的Word格式是二进制格式,因此,版本控制系统是没法跟踪Word文件的改动的,前面我们举的例子只是为了演示,如果要真正使用版本控制系统,就要以纯文本方式编写文件.

因为文本是有编码的,比如中文有常用的GBK编码,日文有Shift_JIS编码,如果没有历史遗留问题,强烈建议使用标准的UTF-8编码,所有语言使用同一种编码,既没有冲突,又被所有平台所支持.

示例:创建一个readme.txt文件,内容如下:

hello,git is free software.

```
cd /data/git-test  # 在仓库创建文件
vim readme.txt
"""
hello,git is free software.
"""

# 查看git已检测到仓库的所有文件
git status

# 使用git add添加文件到git的stage区域,可添加多个文件,空格分割;注意,此时git只是跟踪,不会进行管控.
git add readme.txt

# 使用git commit告诉git,把stage区域的所有文件提交到git管理仓库区域,真正的对目标文件进行版本控制. 
# -m 后面输入的是本次提交的说明,可以输入任意内容,当然最好是有意义的,这样你就能从历史记录里方便地找到改动记录.
git commit -m "init file"
```

# 提交流程

流程如下图:

![img](git%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/384def75-680b-4b34-8e4e-c80e77cca377.jpg)

![img](git%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/4d19fe56-9d09-48b7-ac31-fb2315149866.jpg)

# 回滚操作

查看并恢复某个时间点的内容

```
# 第一次修改code1.txt
vim readme.txt
"""
hello,git is free software.
+++++++++++++++first edit.  # 增加了此处内容
"""

# 查看发生变化的所有文件(此时还未提交到暂存区)
git status

# 查看具体的修改内容(此时还未提交到暂存区)
git diff readme.txt
"""
diff --git a/readme.txt b/readme.txt
index bb4527d..8264ec9 100644
--- a/readme.txt  # －号表示去掉或被修改的内容;
+++ b/readme.txt  # ＋号表示修改或新增的内容;
@@ -1 +1,2 @@
 hello,git is free software.
+++++++++++++++first edit.  # 此处为不同之处
"""

# 将修改提交到暂存区,版本库,让git管理起来
git add readme.txt
git commit -m "first edit"

# 再次查看git库内是否有发生变化的文件
git status
"""
# On branch master
nothing to commit (working directory clean)
"""

# 第二次修改,提交
vim readme.txt
"""
hello,git is free software.
++++++++++++++first edit.
*******hello*******  # 增加
>>>>>>>>>>>>>>second edit...  # 增加
"""

git add readme.txt
git commit -m "second edit"


# 第三次修改,提交
vim code1.txt
"""
hello,git is free software.
++++++++++++++first edit.
$$$$$$$$$$$$$$$$  # 删除并新增
>>>>>>>>>>>>>>second edit...
"""

git add readme.txt
git commit -m "third edit"
# 每次提交,就相当于创建了一次快照,可用于恢复
```

如此修改3次,像这样,不断对文件进行修改,然后不断提交修改到版本库里,就好比玩RPG游戏时,每通过一关就会自动把游戏状态存盘,如果某一关没过去,你还可以选择读取前一关的状态.有些时候,在打Boss之前,你会手动存盘,以便万一打Boss失败了,可以从最近的地方重新开始.

Git也是一样,每当你觉得文件修改到一定程度的时候,就可以"保存一个快照",这个快照在Git中被称为commit.一旦你把文件改乱了,或者误删了文件,还可以从最近的一个commit恢复,然后继续工作,而不是把几个月的工作成果全部丢失.

在实际工作中,我们脑子里怎么可能记得一个几千行的文件每次都改了什么内容,不然要版本控制系统干什么,

版本控制系统肯定有某个命令可以告诉我们历史记录,在Git中,我们用git log命令查看:

```
# 显示从最近到最远的提交日志
git log
"""
commit 939f4af7943195917930352cae061b11dd047f88
Author: xiaofei <1023668666@qq.com>
Date:   Tue May 22 15:55:51 2018 +0800

    thired edit

commit cc612b6bc77970923ba871de9c09d98ca1caa16d
Author: xiaofei <1023668666@qq.com>
Date:   Tue May 22 15:46:14 2018 +0800

    second edit

commit b40f82418075b80520f1980b76598072cfa1e606
Author: xiaofei <1023668666@qq.com>
Date:   Tue May 22 15:42:35 2018 +0800

    first edit

commit dcc013871b173709b6c458654fe41404badb99fd
Author: xiaofei <1023668666@qq.com>
Date:   Tue May 22 15:36:09 2018 +0800

    init file
"""

# 简化显示
git log --pretty=oneline
"""
939f4af7943195917930352cae061b11dd047f88 thired edit  # 这里的ID串不一样的
cc612b6bc77970923ba871de9c09d98ca1caa16d second edit
b40f82418075b80520f1980b76598072cfa1e606 first edit
dcc013871b173709b6c458654fe41404badb99fd init file
"""
```

# 时光穿梭

Git必须知道当前版本是哪个版本,在Git中,用HEAD表示当前版本,也就是最新的提交,上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100

```
# 回滚到上一个版本
git reset --hard HEAD^
"""
HEAD is now at cc612b6 second edit
"""

# 查看文件内容是否与第二次的修改内容一致
cat readme.txt
"""
hello,git is free software.
++++++++++++++first edit.
*******hello*******
>>>>>>>>>>>>>>second edit...
"""

# 查看文件版本库的状态
git log --pretty=oneline
"""
cc612b6bc77970923ba871de9c09d98ca1caa16d second edit
b40f82418075b80520f1980b76598072cfa1e606 first edit
dcc013871b173709b6c458654fe41404badb99fd init file
"""
```

发现刚才的最新版已经没有了,那么如果想要恢复回来最新的版本,如何做呢?

办法其实还是有的,只要上面的命令行窗口还没有被关掉,你就可以顺着往上找啊找啊,找到那个最新版本的版本号字串是'445965781d1fd0d91e76d120450dd18fd06c7489',于是就可以指定回到未来的某个版本:

```
# 版本号没必要写全,前几位就可以了,Git会自动去找.当然也不能只写前一两位,因为Git可能会找到多个版本号,就无法确定是哪一个了
git reset --hard 939f4af7943
"""
HEAD is now at 939f4af thired edit
"""
```

Git的版本回退速度非常快,因为Git在内部有个指向当前版本的HEAD指针,当你回退版本的时候,Git仅仅是把HEAD从指向那个最新版本.

现在,你回退到了某个版本,关掉了电脑,第二天早上就后悔了,想恢复到新版本怎么办?找不到新版本的版本号怎么办?

在Git中,总是有后悔药可以吃的.当你用回退到过去某版本后,再想恢复到最新的版本,就必须找到版本号,git提供了查询记录命令的命令reflog

```
git reset --hard HEAD^^
git reflog
"""
b40f824 HEAD@{0}: HEAD^^: updating HEAD
939f4af HEAD@{1}: 939f4af7943: updating HEAD
cc612b6 HEAD@{2}: HEAD^: updating HEAD
939f4af HEAD@{3}: commit: thired edit
cc612b6 HEAD@{4}: commit: second edit
b40f824 HEAD@{5}: commit: first edit
dcc0138 HEAD@{6}: commit (initial): init file
"""

git reset --hard 939f4af
cat readme.txt
```

# 工作区和缓存区

Git和其他版本控制系统如SVN的一个不同之处就是有暂存区的概念.

## 工作区（Working Directory）

就是你在电脑里能看到的目录,比如我们创建的/data/git-test/目录就是git的工作区

## 版本库（Repository）

工作区有一个隐藏目录.git,这个不算工作区,而是Git的版本库.

Git的版本库里存了很多东西,其中最重要的就是称为stage(或者叫index)的暂存区,还有Git为我们自动创建的第一个分支master,以及指向master的一个指针叫HEAD.

![img](git%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.assets/959e9bca-ea94-4a37-afbd-fe5c354d0fb9.jpg)

前面讲了我们把文件往Git版本库里添加的时候,是分两步执行的：

第一步是用git add把文件添加进去,实际上就是把文件修改添加到暂存区;

第二步是用git commit提交更改,实际上就是把暂存区的所有内容提交到master分支;

因为我们创建Git版本库时,Git自动为我们创建了唯一的一个master分支.所以,现在git commit就是往master分支上提交更改.

可以简单理解为,需要提交的文件修改,通通放到暂存区,然后一次性将暂存区的所有修改提交到master.

示例:

```
# 对工作区的readme文件进行继续第四次修改
vim readme.txt
"""
hello,git is free software.
++++++++++++++first edit.
$$$$$$$$$$$$$$$$
>>>>>>>>>>>>>>second edit...
!!!!!forth edit....  # 新增
"""

# 在工作区新增一个license.md文件,内容随便写
echo "hello,world." > license.md
cat license.md

# 查看git状态
git status
```

Git非常清楚地告诉我们,readme.txt被修改了,而license.md还从来没有被添加过,所以它的状态是Untracked.

```
# git add,再git status
git add .
git status
"""
# On branch master
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#   new file:   license.md
#   modified:   readme.txt
"""
```

git add命令实际上就是把要提交的所有修改放到暂存区(Stage),然后,执行git commit就可以一次性把暂存区的所有修改提交到master分支.

一旦提交后,如果你又没有对工作区做任何修改,那么工作区就是"干净"的,暂存区是Git非常重要的概念,弄明白了暂存区,就弄明白了Git的很多操作到底干了什么.

另外还有一个临时区(stash)

git stash 保存工作区到临时区

git stash apply 恢复stash区的内容

git stash drop 删除stash区的内容

git stash pop 恢复并删除

# 撤销修改

撤销修改的文件,这里有两种情况:

1,文件还未add到暂存区的时候,被修改了;

2.文件已add到了暂存区,被修改了;

```
# 撤销文件在工作区时的修改
git checkout -- readme.txt

# 撤销文件在暂存区时的修改,重新放回工作区,显示已被修改
git reset HEAD readme.txt
git status
git checkout -- readme.txt
```

注意: 

1,命令"git checkout -- code1"中的"--"很重要,没有"--",就变成了“切换到另一个分支”的命令,我们在后面的分支管理中会再次遇到git checkout命令.

2,"git reset HEAD code1"命令既可以回退版本,也可以把暂存区的修改回退到工作区,当我们用HEAD时,表示最新的版本.

当然,谁还没有犯二的时候,假如你修改了文件,也提交到了暂存区,更是提交到了版本库!但庆幸的是还没将本地版本库推送到远程服务器上,可以参考'版本回退'操作

# 删除操作

在Git中,删除也是一个修改操作.

你通常直接在文件管理器中把没用的文件删了,或者用rm命令删了,也就是删除了工作区中的文件,这个时候,Git知道你删除了文件,因此,工作区和版本库就不一致了,git status命令会立刻告诉你哪些文件被删除了.

现在你有两个选择:

一是确定要从版本库中删除该文件,那就用命令"git rm"删掉,并且"git commit";

二是误删,因为版本库里还有呢,所以可以很轻松地把误删的文件恢复到最新版本;

"git checkout"其实是用版本库里的文件替换工作区的文件,无论工作区是修改还是删除,都可以"一键还原"

```
# 创建一个新文件test1.txt,并提交到git
vim test1.txt
git add test1.txt
git commit -m 'add test1.txt'

# 删除工作区中的test1.txt文件,然后查看git状态
rm test1.txt
ls
git status
>     deleted:    test1.txt

# 从版本库中彻底删除
git rm test1.txt
git commit -m "remove test1.txt file"

# 误删恢复
git checkout -- test1.txt
```

注:

只能恢复文件到最新版本，你会丢失最近一次提交后你修改的内容

# 忽略特殊文件.gitignore

有些时候,你必须把某些文件放到Git工作目录中,但又不能提交它们,比如保存了数据库密码的配置文件啦,等等,每次git status都会显示Untracked files ...,有强迫症的童鞋心里肯定不爽.好在Git考虑到了大家的感受,这个问题解决起来也很简单,在Git工作区的根目录下创建一个特殊的.gitignore文件,然后把要忽略的文件名填进去,Git就会自动忽略这些文件.

不需要从头写.gitignore文件,GitHub已经为我们准备了各种配置文件,只需要组合一下就可以使用了.所有配置文件可以直接在线浏览:https://github.com/github/gitignore

忽略文件的原则是:

1,忽略操作系统自动生成的文件,比如缩略图等;

2,忽略编译生成的中间文件,可执行文件等,也就是如果一个文件是通过另一个文件自动生成的,那自动生成的文件就没必要放进版本库,比如Java编译产生的.class文件;

3,忽略你自己的带有敏感信息的配置文件,比如存放口令的配置文件.

举个例子：

假设你在Windows下进行Python开发,Windows会自动在有图片的目录下生成隐藏的缩略图文件,如果有自定义目录,目录下就会有Desktop.ini文件,因此你需要忽略Windows自动生成的垃圾文件:

```
# Windows:
Thumbs.db
ehthumbs.db
Desktop.ini
```

然后,继续忽略Python编译产生的.pyc, .pyo, dist等文件或目录:

```
# Python:
*.py[cod]
*.so
*.egg
*.egg-info
dist
build
```

加上你自己定义的文件,最终得到一个完整的.gitignore文件,内容如下：

```
# Windows:
Thumbs.db
ehthumbs.db
Desktop.ini

# Python:
*.py[cod]
*.so
*.egg
*.egg-info
dist
build

# My configurations:
db.ini
deploy_key_rsa
```

最后一步就是把.gitignore也提交到Git,就完成了.

使用Windows的童鞋注意了,如果你在资源管理器里新建一个.gitignore文件,它会非常弱智地提示你必须输入文件名,但是在文本编辑器里"保存"或者"另存为"就可以把文件保存为.gitignore了.

有些时候,你想添加一个文件到Git,但发现添加不了,原因是这个文件被.gitignore忽略了:

```
git add App.class
"""
The following paths are ignored by one of your .gitignore files:
App.class
Use -f if you really want to add them.
"""

# 如果你确实想添加该文件,可以用-f强制添加到Git:
git add -f App.class
```

或者你发现,可能是.gitignore写得有问题,需要找出来到底哪个规则写错了,可以用git check-ignore命令检查：

```
git check-ignore -v App.class
```

Git会告诉我们, .gitignore的第3行规则忽略了该文件,于是我们就可以知道应该修订哪个规则.

# 总结

忽略某些文件时，需要编写.gitignore；

.gitignore文件本身要放到版本库里,并且可以对.gitignore做版本管理.

git的GUI工具:

source tree

Git的基本操作 

初始化操作

$ it config -global user.name <name> #设置提交者名字

$ git config -global user.email <email> #设置提交者邮箱

$ git config -global core.editor <editor> #设置默认文本编辑器

$ git config -global merge.tool <tool> #设置解决合并冲突时差异分析工具

$ git config -list #检查已有的配置信息

创建新版本库

$ git clone <url> #克隆远程版本库

$ git init #初始化本地版本库

修改和提交

$ git add . #添加所有改动过的文件

$ git add <file> #添加指定的文件

$ git mv <old> <new> #文件重命名

$ git rm <file> #删除文件

$ git rm -cached <file> #停止跟踪文件但不删除

$ git commit -m <file> #提交指定文件

$ git commit -m “commit message” #提交所有更新过的文件

$ git commit -amend #修改最后一次提交

$ git commit -C HEAD -a -amend #增补提交（不会产生新的提交历史纪录）

查看提交历史

$ git log #查看提交历史

$ git log -p <file> #查看指定文件的提交历史

$ git blame <file> #以列表方式查看指定文件的提交历史

$ gitk #查看当前分支历史纪录

$ gitk <branch> #查看某分支历史纪录

$ gitk --all #查看所有分支历史纪录

$ git branch -v #每个分支最后的提交

$ git status #查看当前状态

$ git diff #查看变更内容

撤消操作

$ git reset -hard HEAD #撤消工作目录中所有未提交文件的修改内容

$ git checkout HEAD <file1> <file2> #撤消指定的未提交文件的修改内容

$ git checkout HEAD. #撤消所有文件

$ git revert <commit> #撤消指定的提交

分支与标签

$ git branch #显示所有本地分支

$ git checkout <branch/tagname> #切换到指定分支或标签

$ git branch <new-branch> #创建新分支

$ git branch -d <branch> #删除本地分支

$ git tag #列出所有本地标签

$ git tag <tagname> #基于最新提交创建标签

$ git tag -d <tagname> #删除标签

合并与衍合

$ git merge <branch> #合并指定分支到当前分支

$ git rebase <branch> #衍合指定分支到当前分支

远程操作

$ git remote -v #查看远程版本库信息

$ git remote show <remote> #查看指定远程版本库信息

$ git remote add <remote> <url> #添加远程版本库

$ git fetch <remote> #从远程库获取代码

$ git pull <remote> <branch> #下载代码及快速合并

$ git push <remote> <branch> #上传代码及快速合并

$ git push <remote> : <branch>/<tagname> #删除远程分支或标签

$ git push -tags #上传所有标签