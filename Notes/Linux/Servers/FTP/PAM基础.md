[TOC]

Linux-PAM(linux可插入认证模块)是一套共享库,使本地系统管理员可以随意选择程序的认证方式. 换句话说,不用(重新编写)重新编译一个包含PAM功能的应用程序,就可以改变它使用的认证机制. 这种方式下,就算升级本地认证机制,也不用修改程序. PAM使用配置/etc/pam.d/下的文件,来管理对程序的认证方式.应用程序 调用相应的配置文件,从而调用本地的认证模块.模块放置在/lib/security下,以加载动态库的形式进，像我们使用su命令时,系统会提示你输入root用户的密码.这就是su命令通过调用PAM模块实现的.

pam简介

一种是写在/etc/pam.conf文件中，但centos6之后的系统中，这个文件就没有了。另一种写法是,将PAM配置文件放到/etc/pam.d/目录下,其规则内容都是不包含 service 部分的，即不包含服务名称，而/etc/pam.d 目录下文件的名字就是服务名称。如: vsftpd,login等.,只是少了最左边的服务名列.如/etc/pam.d/sshdpnnn如:/etc/pam.d/sshd

PAM配置文件有两种写法:

PAM的配置文件介绍

 

```
#%PAM-1.0
auth       required     pam_sepermit.so
auth       include      password-auth
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    include      password-auth
```



由上图可以将配置文件分为四列，

第一列代表模块类型

第二列代表控制标记

第三列代表模块路径

第四列代表模块参数

1.PAM的模块类型

Linux-PAM有四种模块类型,分别代表四种不同的任务

它们是:认证管理(auth),账号管理(account),会话管理(session)和密码(password)管理,一个类型可能有多行,它们按顺序依次由PAM模块调用.

auth:用来对用户的身份进行识别.如:提示用户输入密码,或判断用户是否为root等.account:对帐号的各项属性进行检查.如:是否允许登录,是否达到最大用户数,或是root用户是否允许在这个终端登录等.session:这个模块用来定义用户登录前的,及用户退出后所要进行的操作.如:登录连接信息,用户数据的打开与关闭,挂载文件系统等.password:使用用户信息来更新.如:修改用户密码.

2,PAM的控制标记PAM使用控制标记来处理和判断各个模块的返回值.（在此只说明简单的认证标记） required:表示即使某个模块对用户的验证失败，也要等所有的模块都执行完毕后,PAM 才返回错误信息。这样做是为了不让用户知道被哪个模块拒绝。如果对用户验证成功，所有的模块都会返回成功信息。
requisite:与required相似,但是如果这个模块返回失败,则立刻向应用程序返回失败,表示此类型失败.不再进行同类型后面的操作.
sufficient:表示如果一个用户通过这个模块的验证，PAM结构就立刻返回验证成功信息（即使前面有模块fail了，也会把 fail结果忽略掉），把控制权交回应用程序。后面的层叠模块即使使用requisite或者required 控制标志，也不再执行。如果验证失败，sufficient 的作用和 optional 相同optional:表示即使本行指定的模块验证失败，也允许用户接受应用程序提供的服务，一般返回PAM_IGNORE(忽略).
3.模块路径模块路径.即要调用模块的位置. 如果是64位系统，一般保存在/lib64/security,如: pam_unix.so,同一个模块,可以出现在不同的类型中.它在不同的类型中所执行的操作都不相同.这是由于每个模块针对不同的模块类型,编制了不同的执行函数.
4.模块参数模块参数,即传递给模块的参数.参数可以有多个,之间用空格分隔开,如:password  required  pam_unix.so nullok obscure min=4 max=8 md5
三、常用的PAM模块介绍
pam_unix.so  auth:提示用户输入密码,并与/etc/shadow文件相比对.匹配返回0  account:检查用户的账号信息(包括是否过期等).帐号可用时,返回0.
  password:修改用户的密码. 将用户输入的密码,作为用户的新密码更新shadow文件
pam_shells.so  \\如果用户想登录系统，那么它的shell必须是在/etc/shells文件中之一的shell  auth  account
pam_deny.so  \\该模块可用于拒绝访问  auth
  account
  password
  session


pam_permit.so  \\该模块任何时候都返回成功  auth  account  password  session


pam_securetty.so   \\如果用户要以root登录时,则登录的tty必须在/etc/securetty之中.  authpam_listfile.so   \\访问应用程的控制开关  auth  account  password session
pam_cracklib.so   \\这个模块可以插入到一个程序的密码栈中,用于检查密码的强度.  password
pam_limits.so   \\定义使用系统资源的上限，root用户也会受此限制，可以通过/etc/security/limits.conf或/etc/security/limits.d/*.conf来设定  session

温馨提示：如果发生错误，Linux-PAM 可能会改变系统的安全性。这取决于你自己的选择，你可以选择不安全(开放系统)和绝对安全(拒绝任何访问)。通常，Linux-PAM 在发生错误时，倾向于后者。任何的配置错误都可能导致系统整个或者部分无法访问。配置 Linux-PAM 时，可能遇到最大的问题可能就是 Linux-PAM 的配置文件/etc/pam.d/*被删除了。如果发生这种事情，你的系统就会被锁住。有办法可以进行恢复，最好的方法就是用一个备份的镜像来恢复系统，或者登录进单用户模式然后进行正确的配置。