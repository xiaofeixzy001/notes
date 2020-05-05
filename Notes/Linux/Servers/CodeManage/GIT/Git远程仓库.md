[TOC]

远程仓库

Git是分布式版本控制系统,同一个Git仓库,可以分布到不同的机器上.怎么分布呢?最早,肯定只有一台机器有一个原始版本库,此后,别的机器可以“克隆”这个原始版本库,而且每台机器的版本库其实都是一样的,并没有主次之分.

其实一台电脑上也是可以克隆多个版本库的,只要不在同一个目录下.不过,现实生活中是不会有人这么傻的在一台电脑上搞几个远程库玩,因为一台电脑上搞几个远程库完全没有意义,而且硬盘挂了会导致所有库都挂掉,所以我也不告诉你在一台电脑上怎么克隆多个仓库.

实际情况往往是这样,找一台电脑充当服务器的角色,每天24小时开机,其他每个人都从这个"服务器"的仓库克隆一份到自己的电脑上,并且各自把各自的提交推送到服务器仓库里,也从服务器仓库中拉取别人的提交.

搭建自己的git服务器

Git支持本地(local),ssh,git和http(s)这四种协议进行传输,这里将基于ssh协议搭建(此协议不利于开源,适合公司团队使用).

这里以centos为例

 

```
# Git服务端操作

# 新增用户gitadmin用于管理git仓库
useradd gitadmin
passwd gitadmin

# 安装配置git服务器
yum install -y git
mkdir /data/mygit/web.git
chown -R gitadmin:gitadmin /data/mygit/web.git  # 服务器上的Git仓库通常都以.git结尾

# 初始化仓库
cd /data/mygit/web.git
git init
git config --global user.name "gitadmin"
git config --global user.email "gitadmin@172.16.1.50"

# 添加一个初始文件,用于测试
echo "hello, this is my first code." > test1.txt
git add .
git commit -m "v1"
git status

# 出于安全考虑,禁止gitadmin不允许登录shell
vim /etc/passwd
"""
gitadmin:x:500:500::/home/gitadmin:/usr/bin/git-shell  # shell改成这个
"""
```

客户端操作

 

```
# 生成密钥对,公钥默认在~/.ssh/id_rsa.pub
ssh-keygen -t rsa
ssh-copy-id -p 2201 gitadmin@172.16.1.50

#或 ssh gitadmin@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub


# 找一个目录作为本地仓库,然后初始化,通过git-bash工具创建即可
cd mygit
git init

# 克隆远程git仓库到本地当前目录
git clone ssh://gitadmin@172.16.1.50:2201/data/mygit/web.git
ls

# 对克隆下来的文件test1.txt进行修改并提交本地仓库
echo "local v1" >> web.git/test1.txt
cat web.git/test1.txt
git add web.git/test1.txt
git commit -m "local v1"

# 回到git服务器中,因为当前分支是master,别人推送过来的修改,暂时时无法更新的,需要切换到其他分支上,再切回来,就会发现已经更改了.
git branch dev
git checkout dev

# 本地操作,将本地仓库已被更新的操作推送到远程仓库中的master分支上
git push origin master

# git服务器操作
git checkout master
ls
cat test1.txt
```

Git的数据传输及常用命令

![img](Git%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93.assets/bcd1b969-bb62-4ad6-86f0-c788aca419d2.png)

工作区(workspace):初始化时所在的目录,我们新创建的文件,修改的文件如果没有提交都是保存到工作区的 ;



暂存区(index):git add的时候我们会将代码提交到暂存区.只能通过git GUI或git shell 窗口显示,提交代码,解决冲突的中转站.(从上面这张图来说,我们的暂存区也会受到影响);



本地仓库(local repository):git commit的时候会将暂存区的代码提交到本地仓库,只能在git shell 窗口显示,连接本地代码跟远程代码的枢纽,不能联网时本地代码可先提交至该处;



远程仓库(remote repository):git push会将本地仓库代码提交到远程仓库 ;



从上图我们可以看到：git fetch只是将代码从远程拉取到本地仓库而已,我们的工作区以及暂存区代码都是不会受到任何影响的.而git pull和git rebase会对本地仓库,暂存区和工作区都产生影响.

git服务器交互过程中所用到的主要命令有:

git clone

git remote

git fetch

git pull

git push

git clone



此命令是我们和远程仓库交互的第一步,通过此命令,我们可以将远程版本库克隆到本地,



语法：git clone 版本库的网址     本地库名称



本地库名称可以省略,省略后在本地会生成一个和远程版本库名字相同的目录.

git remote



此命令用于管理远程主机名,此命令在没有参数的情况下可以列出所有主机名.



显示origin是在使用clone命令,克隆远程版本库时Git自动为远程主机命名.



通过命令git remote –v,可查看版本库的网址.

git fetch



此命令可以将远程版本库的更新,更新到本地库



语法：git fetch 主机名字



在默认情况下,git fetch origin将会更新远程主机origin上的所有分支,如果只想更新某个分支,则在主机名origin后面加分支名.



语法：git fetch origin master

git push



此命令用于将本地分支的更新推送到远程主机



语法：git push 远程主机名 本地分支名：远程分支名



如果省略远程分支名,则表示将本地分支推送与存在最终关系的远程分支,如果远程分支不存在,则会被新建.



如：git push origin master,表示将本地master分支推送到origin主机的master分支上



如果省略本地分子名,则表示要删除远程主机中分支,如git push origin:master,则表示删除origin主机中master分支

git pull



此命令用于获取远程分支中更新.



语法:git pull 远程主机 远程分支

本地分支如：git pull origin master:master,表示将远程主机origin中的master分支跟新到本地分支master



GitHub的使用

GitHub是一个神奇的网站,从名字就可以看出,这个网站就是提供Git仓库托管服务的,所以,只要注册一个GitHub账号,就可以免费获得Git远程仓库.

网址:https://github.com

1,注册自己的账号

2,创建ssh key.因为本地git仓库和github仓库间的传输是通过ssh加密的.

windows下打开"git bash"程序创建;

linux下,直接shell命令行下创建.

ssh-keygen -t ras -C "your_email@example.com"

邮件地址换成自己的邮件地址,然后一路回车,默认即可,密码可又有无.

结束以后,在用户家目录中隐藏目录.ssh/,里面的id_rsa和id_rsa.pub就是创建的私钥和公钥.

3,登录github,打开"Account settings" --> "SSH Keys" --> "Add SSH key" --> title任意填写,key文本框粘贴自己的id_rsa.pub的内容.

GitHub需要识别出你推送的提交确实是你推送的,而不是别人冒充的,而Git支持SSH协议,所以,GitHub只要知道了你的公钥,就可以确认只有你自己才能推送.

当然，GitHub允许你添加多个Key,假定你有若干电脑,你一会儿在公司提交,一会儿在家里提交,只要把每台电脑的Key都添加到GitHub,就可以在每台电脑上往GitHub推送了.

最后友情提示,在GitHub上免费托管的Git仓库,任何人都可以看到(但只有你自己才能改).所以,不要把敏感信息放进去.

如果你不想让别人看到Git库,有两个办法,一个是交点保护费,让GitHub把公开的仓库变成私有的,这样别人就看不见了(不可读更不可写),另一个办法是自己动手,搭一个Git服务器,因为是你自己的Git服务器,所以别人也是看不见的.这个方法我们后面会讲到的.相当简单,公司内部开发必备.

创建远程仓库

背景:你已经在本地创建了一个Git仓库后,又想在GitHub创建一个Git仓库,并且让这两个仓库进行远程同步,这样,GitHub上的仓库既可以作为备份,又可以让其他人通过该仓库来协作,真是一举多得.

1,登录账号,右上角找到"New repository"

![img](Git%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93.assets/cf4373d9-5646-4fea-928a-af893cd9ea4c.jpg)

2,填写仓库信息

![img](Git%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93.assets/233dab34-2fa4-4266-ad2b-47ec30f74478.png)

3,创建好的仓库

![img](Git%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93.assets/6f5431b3-b2cf-4d4c-a22e-4152181f87c6.png)

目前,在GitHub上的这个oldboy_website仓库还是空的,GitHub告诉我们,可以从这个仓库克隆出新的仓库到本地,也可以把一个已有的本地仓库与之关联,然后,把本地仓库的内容推送到GitHub仓库.

1,把本地库的内容推送到远程

用git push命令,实际上是把当前分支master推送到远程.

 

```
# 将本地git仓库添加到远程仓库github
$ git remote add origin git@github.com:xiaofeixzy001/oldboy_website.git 
$ git push -u origin master

# 添加后,远程库的名字就是origin[原始的],这是Git默认的叫法,也可以改成别的,但是origin这个名字一看就知道是远程库;
# master是主分支上, 关于分支概念,后面说.

```

请千万注意,把上面的xiaofeixzy001替换成你自己的GitHub账户名,否则,你在本地关联的就是我的远程库,关联没有问题,但是你以后推送是推不上去的,因为你的SSH Key公钥不在我的账户列表中.

此时刷新远程仓库页面,就看到了你刚从本地推上来的代码了.

![img](Git%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93.assets/8fc41a55-efd4-495b-9895-ecaefa0a2d50.jpg)

示例如下:只要本地做了提交,就可以进行同步上传到远程仓库中了.

 

```
# vim index.html
# git add .
# git commint -m "add home page"
# git push origin master
```

刷新远程仓库页面,就可看到新创建的文件了.

从远程库克隆到本地

现在,假设我们从零开发,那么最好的方式是先创建远程库,然后,从远程库克隆.

首先,登陆GitHub,创建一个新的仓库,名字叫gitskills：

![img](Git%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93.assets/701b3ab6-929f-4386-ae9c-cb74661ae155.png)

我们勾选Initialize this repository with a README,这样GitHub会自动为我们创建一个README.md文件,创建完毕后,可以看到README.md文件

![img](Git%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93.assets/1b26e059-49b8-47f7-a6f8-b21798d9001b.png)

现在,远程库已经准备好了,下一步是用命令git clone克隆一个本地库：

![img](Git%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93.assets/184947f9-4bcd-4bad-99ae-e6648d538476.jpg)

在本地找一个你想存放这个远程仓库的目录,然后在这个目录下执行命令git clone来克隆这个远程库:

 

```
# 从远程仓库克隆一份到本地仓库
$ git clone git@github.com:xiaofeixzy001/gitskills.git
$ cd gitskills/  #进入刚clone下来的目录
$ ls
README.md
# 将修改后的文件提交到远程仓库
$ git add .
$ git commit -m "from linux edit."
$ git push origin master
```

如果有多个人协作开发,那么每个人各自从远程克隆一份就可以了.

但是需要注意的是:

远程仓库中的某个文件,同时被多个人修改,但提交的时间有早有晚,这时,后面提交的人就会提交不上去,出现错误提示:无法推送,有冲突...

默认,只要不涉及同一行的修改,都会自动合并.

你也许还注意到,GitHub给出的地址不止一个,还可以用https://github.com/triaquae/gitskills.git这样的地址.

实际上,Git支持多种协议,默认的git://使用ssh,但也可以使用https等其他协议.

使用https除了速度慢以外,还有个最大的麻烦是每次推送都必须输入口令,但是在某些只开放http端口的公司内部就无法使用ssh协议而只能用https.

GitHub使用

我们一直用GitHub作为免费的远程仓库,如果是个人的开源项目,放到GitHub上是完全没有问题的.其实GitHub还是一个开源协作社区,通过GitHub,既可以让别人参与你的开源项目,也可以参与别人的开源项目.

在GitHub出现以前,开源项目开源容易,但让广大人民群众参与进来比较困难,因为要参与,就要提交代码,而给每个想提交代码的群众都开一个账号那是不现实的,因此,群众也仅限于报个bug,即使能改掉bug,也只能把diff文件用邮件发过去,很不方便.

但是在GitHub上,利用Git极其强大的克隆和分支功能,广大人民群众真正可以第一次自由参与各种开源项目了.

如何参与一个开源项目呢?比如人气极高的bootstrap项目,这是一个非常强大的CSS框架,你可以访问它的项目主页https://github.com/twbs/bootstrap,点“Fork”就在自己的账号下克隆了一个bootstrap仓库,然后,从自己的账号下clone:

 

```
$ git clone git@github.com:michaelliao/bootstrap.git
```

一定要从自己的账号下clone仓库,这样你才能推送修改.如果从bootstrap的作者的仓库地址git@github.com:twbs/bootstrap.git克隆,因为没有权限,你将不能推送修改.

Bootstrap的官方仓库twbs/bootstrap,你在GitHub上克隆的仓库my/bootstrap,以及你自己克隆到本地电脑的仓库,他们的关系就像下图显示的那样:

![img](Git%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93.assets/02ce09e5-73f6-4b6a-ba24-a4cf89176622.png)

如果你想修复bootstrap的一个bug,或者新增一个功能,立刻就可以开始干活,干完后,往自己的仓库推送.

如果你希望bootstrap的官方库能接受你的修改,你就可以在GitHub上发起一个pull request,当然,对方是否接受你的pull request就不一定了.

如果你没能力修改bootstrap,但又想要试一把pull request,那就Fork一下我的仓库:https://github.com/triaquae/gitskills,创建一个your-github-id.txt的文本文件,写点自己学习Git的心得,然后推送一个pull request给我,我会视心情而定是否接受.

小结

在GitHub上,可以任意Fork开源仓库；

自己拥有Fork后的仓库的读写权限；

可以推送pull request给官方仓库来贡献代码.