[TOC]

# 前言

按照其软件包的格式来划分，常见的Linux发行版主要可以分为两类，类ReadHat系列和类Debian系列，这两类系统分别提供了自己的软件包管理系统和相应的工具。 

类RedHat系统中软件包的后缀是rpm，提供了同名的rpm命令来安装、卸载、升级rpm软件包； 类Debian系统中软件包的后缀是deb，同样提供了dpkg命令来对后缀是deb

rpm的全称是Redhat Package  Manager，常见的使用rpm软件包的系统主要有Fedora、CentOS、openSUSE、SUSE企业版、PCLinuxOS以及Mandriva Linux、Mageia等。 使用deb软件包后缀的类Debian系统最常见的有Debian、Ubuntu、Finnix等。

无论是rpm命令还是dpkg命令在安装软件包时都存在软件包依赖关系问题。  

Fedora/CentOS 系统 提供了 yum 来自动解决软件包的安装依赖； openSUSE/SUSE 系统 提供了 zypper 来自动解决软件包的安装依赖； Mandriva Linux/Mageia 系统 提供了 urpmi 来自动解决软件包的安装依赖； Debian/Ubuntu 系统 提供了 apt-* 命令。

这些工具本质上最终还是调用了 rpm (或者dpkg)来进行安装，但是在安装前会自动帮用户解决了软件包的安装依赖。

从软件运行的结构来说，一个软件主要可以分为三个部分：可执行程序、配置文件和动态库。 当然还有可能会有相关文档、手册、供二次开发用的头文件以及一些示例程序等等。其他部分都是可选的，只有可执行文件是必须的。

制作rpm软件包，常用的方法是使用 rpmbuild 这个命令行工具。

# 安装rpmbuild工具

可以执行yum命令安装工具包(能够联网的情况):

```
yum install rpm* rpm-build rpmdev*
```

或者使用rpm包直接接安装(有rpm-build安装包的情况)：

```
rpm -ivh rpm-build-4.8.0-37.el6.x86_64.rpm
```

如果还需要安装rpmdevtools工具，稍微麻烦一些，需要查看依赖自己来安装。

# rpmbuid打包目录

rpm打包目录有一些严格的层次上的要求。

rpm的版本<=4.4.x，rpmbuid工具其默认的工作路径是 /usr/src/redhat因为权限的问题，普通用户不能制作rpm包，制作rpm软件包时必须切换到 root 身份才可以。

rpm从4.5.x版本开始，将rpmbuid的默认工作路径移动到用户家目录下的rpmbuild目录里，即 $HOME/rpmbuild ，并且推荐用户在制作rpm软件包时尽量不要以root身份进行操作。

rpmbuild默认工作路径的确定，通常由在 /usr/lib/rpm/macros 这个文件里的一个叫做 %\_topdir 的宏变量来定义。如果用户想更改这个目录名，rpm官方并不推荐直接更改这个目录，而是在用户家目录下建立一个名为 .rpmmacros 的隐藏文件(Linux下隐藏文件,前面的点不能少)，然后在里面重新定义 %\_topdir，指向一个新的目录名。这样就可以满足某些用户的差异化需求了。通常情况下.rpmmacros文件里一般只有一行内容，比如：

```shell
[root@centos6 ~]# rpmbuild --showrc | grep _topdir
"""
-14: _builddir	%{_topdir}/BUILD
-14: _buildrootdir	%{_topdir}/BUILDROOT
-14: _rpmdir	%{_topdir}/RPMS
-14: _sourcedir	%{_topdir}/SOURCES
-14: _specdir	%{_topdir}/SPECS
-14: _srcrpmdir	%{_topdir}/SRPMS
-14: _topdir	%{getenv:HOME}/rpmbuild
"""

# 修改制作工厂目录
vim ~/.rpmmacros
"""
%_topdir    $HOME/myrpmbuildenv
"""
```

在%\_topdir目录下一般需要建立6个目录 ：

| 目录名    | 说明                                                     | macros中的宏名 |
| --------- | -------------------------------------------------------- | -------------- |
| BUILD     | 制作车间，源码包解压后存放的位置，并在此目录进行编译安装 | %_builddir     |
| BUILDROOT | 临时安装目录，以此目录为根，制作完成后会自动清理         | %_buildrootdir |
| RPMS      | 存放最终制作好的rpm包                                    | %_rpmdir       |
| SOURCES   | 存放源码包，补丁包，配置文件或脚本的目录                 | %_sourcedir    |
| SPECS     | 存放SPEC文件的目录                                       | %_specdir      |
| SRPMS     | 存放src格式的rpm包的目录                                 | %_srcrpmdir    |

==注意：全部大写==

如果有安装rpmdevtools，可以使用 rpmdev-setuptree命令在当前用户home/rpmbuild目录里自动建立上述目录。

# SPEC文件详解

注意：liunx环境可以使用vi来编辑spec文件，windows环境可以可以使用sublime编辑spec文件。 需要注意的是，绝对不能使用记事本来编辑，这是因为windows的默认换行符是 `\r\n`,而 liunx 的换行符是`\n`。如果spec文件中包含多余的`\r`会导致rpm包创建失败。

当目录建立好之后，将所有用于生成rpm包的源代码、shell脚本、配置文件都拷贝到SOURCES目录里，注意通常情况下源码的压缩格式都为*.tar.gz格式。然后，将最最最重要的SPEC文件，命名格式一般是“软件名-版本.spec”的形式，将其拷贝到SPECS目录下，切换到该目录下执行： 

```
rpmbuild -bb 软件名-版本.spec 
```

如果系统有rpmdevtools工具，可以用rpmdev-newspec -o Name-version.spec命令来生成SPEC文件的模板，然后进行修改：

```
[root@localhost ~]# rpmdev-newspec -o myapp-0.1.0.spec
Skeleton specfile (minimal) has been created to "myapp-0.1.0.spec".

[root@localhost ~]# cat myapp-0.1.0.spec 
```

如果没有安装rpmdevtools，也可以自己手动创建一个spec文件。

## spec文件说明

Centos6.10中，使用vim创建.spec结尾的文件时，会自动生成一个spec模板文件，默认显示的是必须的字段：

```
Name:		
Version:	
Release:	1%{?dist}  
Summary:	

Group:		
License:	
URL:		
Source0:	

BuildRequires:	
Requires:	

%description 


%prep
%setup -q


%build
%configure
make %{?_smp_mflags}


%install
%make_install


%files
%doc



%changelog
```

PEC文件的核心是它定义了一些“阶段”(%prep、%build、%install和%clean)，当rpmbuild执行时它首先会去解析SPEC文件，然后依次执行每个“阶段”里的指令。

使用#号表述注释，但切记注释中不能带%号，如果非要带，使用2个%表示。

%define：定义宏的关键字，K V格式。

查看spec宏定义：

rpmbuild --showrc

一个下划线：定义环境的使用情况，

二个下划线：通常定义的是命令，

为什么要定义宏，因为不同的系统，命令的存放位置可能不同，所以通过宏的定义找到命令的真正存放位置。

详细说明：

```spec
Name: nginx  # 软件包的名字,可使用%{name}的方式引用

Version: 1.16.1  # 软件包的版本,注意不要到-符号,可使用%{version}引用

Release: 1%{?dist} # 发布序号,1%表示第几次打包,问号?表示判断,可使用%{release}引用

Summary: my first rpm # 软件包的摘要信息 

Group: Applications/Internet # 软件包的安装分类，参见/usr/share/doc/rpm-4.x.x/GROUPS这个文件定义

License: GPLv2 # 软件的授权方式 

URL: http://localhost/download/   # 源码包的下载URI

Packager:XiaoFei  # 制作者

Vendor: Blizzardwind.com  # 打包组织或者人员

Source: %{name}-%{version}.tar.gz  # 源码包的名称(默认时rpmbuid会到SOURCES目录中去找)，这里的name和version就是前两行定义的值。可以带多个用Source1、Source2等源，后面也可以用%{source1}、%{source2}引用.如果只有一个可以省去0

BuildRoot: %{_topdir}/BUILDROOT/%{name}-%{version}-%{release}-root # 临时安装的虚拟根目录，安装或编译时使用的临时目录，即模拟安装完以后生成的文件目录，可使用$RPM_BUILD_ROOT或%{buildroot}引用。

BuildRequires: gcc,binutils  # 编译时需要的依赖包,以逗号分隔,要求编译nginx时,gcc的版本至少为4.4.2,则可以写成gcc >=4.2.2,还有其他依赖的话则以逗号分别继续写道后面.

Requires: # 安装时需要依赖包,以逗号分隔,可以用>=或<=表示大于或小于某一特定版本,例如:libxxx-devel >= 1.1.1 openssl-devel 。 注意：“>=”号两边需用空格隔开，而不同软件名称也用空格分开

%description # 软件包的详细说明信息

Patch:  # 补丁源码，可使用Patch1、Patch2等标识多个补丁，使用%patch0或%{patch0}引用

Prefix: %{_prefix}  # 这个主要是为了解决今后安装rpm包时，并不一定把软件安装到rpm中打包的目录的情况。这样，必须在这里定义该标识，并在编写%install脚本的时候引用，才能实现rpm安装时重新指定位置的功能

Prefix: %{_sysconfdir}  # 这个原因和上面的一样，但由于%{_prefix}指/usr，而对于其他的文件，例如/etc下的配置文件，则需要用%{_sysconfdir}标识

%define: 预定义的变量,例如定义日志路径: _logpath /var/log/weblog

%prep: 预备参数,通常为%setup -q,先cd后解压

%build: 编译参数,无需编译则留空
./configure --user=nginx --group=nginx --prefix=/usr/local/nginx/……

%install: 安装步骤,此时需要指定安装路径,创建编译时自动生成目录,复制配置文件至所对应的目录中（这一步比较重要！）

%pre: 安装前需要做的任务,如:创建用户

%post: 安装后需要做的任务,如:自动启动的任务

%preun: 卸载前需要做的任务,如:停止任务

%postun: 卸载后需要做的任务,如:删除用户，删除/备份业务数据

%clean: 清除上次编译生成的临时文件,就是上文提到的虚拟目录

%files: 设置文件属性,包含编译文件需要生成的目录、文件以及分配所对应的权限

%changelog: 修改历史
```

制作rpm包有 %prep，%build，%install，%files，%clean 几个关键阶段。

## %prep 段

这个阶段里通常情况，主要完成对源代码包的解压和打补丁，而解压时最常见到的就是一句指令：

```
%prep
%setup -q
```

将 %sourcedir 目录下的源代码解压到 %builddir 目录下。

## %build 阶段

这个阶段会在 %_builddir 目录下执行源码包的编译。一般是执行执行常见的configure和make操作，如果有些软件需要最先执行bootstrap之类的，可以放在configure之前来做。

这个阶段我们最常见只有两条指令：

```
%build
%{configure} \
    --etcdir="%{_sysconfdir}" \
    --mandir="%{_mandir}" \
    --i18n="0" \
    --script="0"

%{__make} %{?_smp_mflags} 
```

它就自动将软件安装时的路径自动设置成如下约定：

1. 可执行程序/usr/bin
2. 依赖的动态库/usr/lib或者/usr/lib64视操作系统版本而定。
3. 二次开发的头文件/usr/include
4. 文档及手册/usr/share/man

这里的 %configure 是个宏常量，会自动将 prefix 设置成 /usr 。另外，这个宏还可以接受额外的参数，如果某些软件有某些高级特性需要开启，可以通过给%configure宏传参数来开启。如果不用 %configure这个宏的话，就需要完全手动指定configure时的配置参数了。同样地，我们也可以给make传递额外的参数，例如：

```
make %{?_smp_mflags} CFLAGS="" … 
```

问号?表示判断，这个宏是否存在。

## %install 阶段

这个阶段就是执行make install操作，这个阶段会在%buildrootdir目录里建好目录结构，然后将需要打包到rpm软件包里的文件从%builddir里拷贝到%_buildrootdir里对应的目录里。

当用户最终用 `rpm -ivh name-version.rpm` 安装软件包时，这些文件会安装到用户系统中相应的目录里。

这个阶段最常见的两条指令是：

```
%install
rm -rf $RPM_BUILD_ROOT 
make install DESTDIR=$RPM_BUILD_ROOT 
```

其中$RPMBUILDROOT也可以换成我们前面定义的BuildRoot变量，不过要写成%{buildroot}才可以，必须全部用小写，不然要报错。

如果软件有配置文件或者额外的启动脚本之类，就要手动用copy命令或者install命令你给将它也拷贝到%{buildroot}相应的目录里。用copy命令时如果目录不存在要手动建立，不然也会报错，所以推荐用install命令。 

## %files 阶段

脚本段，包含httpd, %pre, %post, %preun, %postun。

这个阶段主要用来说明会将%{buildroot}目录下的哪些文件和目录最终打包到rpm包里。

```
%files -f %{name}.lang
%defattr(-,root,root,0755) 
%doc API CHANGES COPYING CREDITS README axelrc.example
%doc %{_mandir}/man1/axel.1*
%doc %{_mandir}/*/man1/axel.1*
%config %{_sysconfdir}/axelrc
/usr/local/bin/axel
```

在%files阶段的第一条命令的语法是：  

```
%defattr(文件权限,用户名,组名,目录权限) 
```

如果不牵扯到文件、目录权限的改变，则一般用%defattr(-,root,root,-)这条指令来为其设置缺省权限。所有需要打包到rpm包的文件和目录都在这个地方列出，例如：

```
%files 
%{_bindir}/* 
%{_libdir}/* 
%config(noreplace) %{_sysconfdir}/*.conf 
```

在安装rpm时，会将可执行的二进制文件放在/usr/bin目录下，动态库放在/usr/lib或者/usr/lib64目录下，配置文件放在/etc目录下，并且多次安装时新的配置文件不会覆盖以前已经存在的同名配置文件。  这里在写要打包的文件列表时，既可以以宏常量开头，也可以为“/”开头，没任何本质的区别，都表示从%{buildroot}中拷贝文件到最终的rpm包里；如果是相对路径，则表示要拷贝的文件位于%{_builddir}目录，这主要适用于那些在%install阶段没有被拷贝到%{buildroot}目录里的文件，最常见的就是诸如README、LICENSE之类的文件。如果不想将%{buildroot}里的某些文件或目录打包到rpm里，则用： `%exclude dic_name` 或者 `file_name`

关于%files阶段有两个特性：

1，%{buildroot}里的所有文件都要明确被指定是否要被打包到rpm里。什么意思呢？假如，%{buildroot}目录下有4个目录a、b、c和d，在%files里仅指定a和b要打包到rpm里，如果不把c和d用exclude声明是要报错的；

2，如果声明了%{buildroot}里不存在的文件或者目录也会报错。

关于%doc宏，所有跟在这个宏后面的文件都来自%{_builddir}目录，当用户安装rpm时，由这个宏所指定的文件都会安装到/usr/share/doc/name-version/目录里。

## %clean阶段

```shell
%clean
%{__rm} -rf %{buildroot}
```

编译完成后一些清理工作，主要包括对%{buildroot}目录的清空(这不是必须的)，通常执行诸如make clean之类的命令。

## %changelog 阶段

这是最后一个阶段，主要记录的每次打包时的修改变更日志。标准格式是：

```
* date +"%a %b %d %Y" 修改人 邮箱 本次版本x.y.z-p 

- 本次变更修改了那些内容 
```



##  打包流程总结

安装 rpmbuild

构建rpm的编译目录结构：

 rpmbuild/ 

├ BUILD 

├ RPMS 

├ SOURCES 

├ SPECS 

└ SRPMS 

将源码放到到 rpmbuild/SOURCES 

在rpmbuild/SPECS目录创建spec文件 

在rpmbuild/SPECS目录下执行打包编译： `rpmbuild -bb xxxxx.spec`

最终生成的rpm包

# 制作Nginx的RPM包

1，安装rpmbuild

```shell
[xiaofei@centos6 ~]$ sudo yum install rpm* rpm-build rpmdev*
[xiaofei@centos6 ~]$ rpmbuild --showrc
[xiaofei@centos6 ~]$ rpmbuild --showrc | grep _topdir
[xiaofei@centos6 ~]$ vim .rpmmacros
"""
%_topdir        /home/xiaofei/rpmbuild
"""

[xiaofei@centos6 ~]$ mkdir -pv rpmbuild/{BUILD,RPMS,SOURCES,SPEC,SRPMS}
[xiaofei@centos6 ~]$ rpmbuild --showrc | grep _topdir

[xiaofei@centos6 ~]$ cd rpmbuild/SOURCES
[xiaofei@centos6 SOURCES]$ wget http://nginx.org/download/nginx-1.16.1.tar.gz
[xiaofei@centos6 SOURCES]$ 
```



2，创建spec文件

在/home/xiaofei/rpmbuild/SPEC目录下创建一个nginx.spec文件

```shell
Name:       nginx
Version:    1.16.1
Release:    3%{?dist}
Summary:    nginx, small and high performance http and reverse proxy server
Group:      System Environment/Daemons
Vendor:     https://blizzardwind.com
Packager:   XiaoFei(1023668666@qq.com)
License:    GPL
URL:        http://nginx.org
BuildRoot:  %{_tmppath}/%{name}-%{version}-%{release}-root

BuildRequires:    openssl-devel,pcre-devel,zlib-devel
BuildRequires:    libxslt-devel,gd-devel
Requires:         pcre,openssl,gd

# for /usr/sbin/useradd
Requires(pre):    shadow-utils
Requires(post):   chkconfig

# for /sbin/service
Requires(preun):  chkconfig,initscripts
Requires(postun): initscripts
Provides:         webserver

Source0:    htt://sysoev.ru/nginx/nginx-%{version}.tar.gz
Source1:    nginx.sysinit
Source2:    nginx.params

%description
Nginx [engine x] is an HTTP(S) server, HTTP(S) reverse proxy and IMAP/POP3
proxy server written by Igor Sysoev.

%prep
%setup

%build
export DESTDIR=%{buildroot}
./configure \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--user=nginx \
--group=nginx \
--with-http_ssl_module \
--with-http_flv_module \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/tmp/nginx/client/ \
--http-proxy-temp-path=/var/tmp/nginx/proxy/ \
--http-fastcgi-temp-path=/var/tmp/nginx/fcgi/ \
--with-pcre
make %{?_smp_mflags}

%install
rm -rf %{buildroot}
make install DESTDIR=%{buildroot}
%{__install} -p -d -m 0755 %{buildroot}/var/run/nginx
%{__install} -p -d -m 0755 %{buildroot}/var/log/nginx
%{__install} -p -D -m 0755 %{SOURCE1} %{buildroot}/etc/rc.d/init.d/nginx
mv %{buildroot}/etc/nginx/fastcgi_params %{buildroot}/etc/nginx/fastcgi_params.orig
%{__install} -p -D -m 0644 %{SOURCE2} %{buildroot}/etc/nginx/fastcgi_params

%clean
rm -rf %{buildroot}

%pre
if [ $1 == 1 ]; then
    /usr/sbin/useradd -s /bin/false -r nginx > /dev/null 2>&1
fi

%post
if [ $1 == 1 ]; then
    /sbin/chkconfig --add %{name}
fi

%preun
if [ $1 == 0 ]; then
    /sbin/service %{name} stop > /dev/null 2>&1
	/sbin/chkconfig --del %{name}
fi

%files
%defattr(-,root,root,1)
%doc LICENSE CHANGES README
%{_sbindir}/%{name}
%dir /var/run/nginx
%dir /var/log/nginx
%dir /etc/nginx
%config(noreplace) /etc/nginx/win-utf
%config(noreplace) /etc/nginx/mime.types
%config(noreplace) /etc/nginx/mime.types.default
%config(noreplace) /etc/nginx/fastcgi.conf
%config(noreplace) /etc/nginx/fastcgi.conf.default
%config(noreplace) /etc/nginx/fastcgi_params
%config(noreplace) /etc/nginx/fastcgi_params.orig
%config(noreplace) /etc/nginx/fastcgi_params.default
%config(noreplace) /etc/nginx/%{name}.conf
%config(noreplace) /etc/nginx/%{name}.conf.default
%config(noreplace) /etc/nginx/scgi_params
%config(noreplace) /etc/nginx/scgi_params.default
#%config(noreplace) /etc/nginx/uwcgi_params
#%config(noreplace) /etc/nginx/uwcgi_params.default
%config(noreplace) /etc/nginx/koi-win
%config(noreplace) /etc/nginx/koi-utf

/usr/local/nginx/html/50x.html
/usr/local/nginx/html/index.html

%attr(0755,root,root) /etc/rc.d/init.d/nginx

%changelog
* Mon Apr 29 2020 XiaoFei <1023668666@qq.com> - 1.16.1-3
- Add fastcgi_params file.

* Mon Apr 29 2020 XiaoFei <1023668666@qq.com> - 1.16.1-2
- Add sysv script /etc/rc.d/init.d/nginx

* Wed Apr 11 2020 XiaoFei <1023668666@qq.com> - 1.16.1-1
- Initial version

# End of nginx.spec
```

在/home/xiaofei/rpmbuild/SOURCES/目录下，添加nginx源码包和nginx服务启动文件和fastcgi_params文件，注意服务启动文件的路径要与编译安装时的路径配置一致。

3， 安装所需依赖环境

```shell
yum install -y openssl-devel pcre-devel zlib-devel libxslt-devel gd-devel
```

4，创建RPM包

```shell
[xiaofei@centos6 SOURCES]$ rpmbuild -ba nginx.spec
[xiaofei@centos6 SOURCES]$ ll ../RPMS/x86_64/
```

5，安装或升级RPM包

```shell
[root@centos6 ~]# rpm -Uvh /home/xiaofei/rpmbuild/RPMS/x86_64/nginx-1.16.1-1.el6.x86_64.rpm

# or

[root@centos6 ~]# rpm -Uvh /home/xiaofei/rpmbuild/RPMS/x86_64/nginx-1.16.1-2.el6.x86_64.rpm
```

6，启动服务

```shell
[root@centos6 ~]# service nginx start
```



## nginx.spec参考

SOURCES目录下需准备的文件：引用的源码文件，引用配置文件，引用System-V风格的Service服务脚本，引用日志轮转的配置文件

```shell
%define _prefix /usr/local/nginx    # 预定义的prefix目录
%define _logpath /var/log/weblog    # 预定义日志目录
Name: nginx 
Version: 1.12.1
Release: 1%{?dist}
Summary: The Nginx HTTP and reverse proxy server
Group: Applications/System
License: GPLv2
URL: https://nginx.org
Packager: XXX <XXX@XXX.com>
Vendor: XXX-XXX
Source0: %{name}-%{version}.tar.gz  # 引用的源码文件
Source1: nginx.conf                 # 引用配置文件
Source2: nginx                      # 引用System-V风格的Service服务
Source3: nginx.logrotate            # 引用日志切割的配置文件
BuildRoot: %_topdir/BUILDROOT       # 虚拟根目录
Requires: libxslt-devel,openssl-devel,pcre-devel    # 所依赖的软件包

%description
NGINX is the heart of the modern web, powering half of the world’s busiest sites and applications. The company's comprehensive application delivery platform combines load balancing, content caching, web serving, security controls, and monitoring in one easy-to-use software package.

%prep                               # 编译前准备工作，这里指定的就是Setup,有条件也可以指定编译器
%setup -q

%build                              # 编译参数，这个看到这里的人基本都懂，没啥讲的，最后一个参数可以使用并行编译: make -j 6
./configure \
  --user=nginx \
  --group=nginx \
  --prefix=%{_prefix} \
  --http-log-path=%{_logpath}/access.log \
  --error-log-path=%{_logpath}/error.log \
  --pid-path=/var/run/nginx.pid \
  --with-http_dav_module \
  --with-http_flv_module \
  --with-http_realip_module \
  --with-http_addition_module \
  --with-http_xslt_module \
  --with-http_sub_module \
  --with-http_random_index_module \
  --with-http_degradation_module \
  --with-http_secure_link_module \
  --with-http_gzip_static_module \
  --with-http_ssl_module \
  --with-http_stub_status_module \
  --with-pcre \
  --with-threads \
  --with-stream \
  --with-ld-opt=-Wl,-E
make %{?_smp_mflags}

%install                            # 安装步骤
rm -rf %{buildroot}                 # 保证虚拟根的干净
make install DESTDIR=%{buildroot}   # install 到虚拟根
%{__install} -p -d -m 0755 %{buildroot}%{_logpath}  # 定义一个日志目录并赋予其权限，这个文件会在编译时自动生成，因此要声明
%{__install} -p -D -m 0644 %{SOURCE1} %{buildroot}%{_prefix}/conf/nginx.conf # 复制SOURCE1中的文件到虚拟根中
%{__install} -p -D -m 0755 %{SOURCE2} %{buildroot}/etc/rc.d/init.d/nginx # 复制SOURCE2中的文件到虚拟根中
%{__install} -p -D -m 0644 %{SOURCE3} %{buildroot}%{_prefix}/conf/nginx.logrotate # 复制SOURCE3中的文件到虚拟根中

%pre                                # 安装前准备操作
if [ $1 == 1 ]; then                # 这里的1为安装,0为卸载,2为更新升级
	/usr/sbin/useradd -r nginx -s /sbin/nologin 2> /dev/null
fi

%post                               # 安装后准备操作
if [ $1 == 1 ]; then
    echo "export PATH=/usr/local/nginx/sbin:$PATH" >> /etc/profile
    source /etc/profile
    cp %{_prefix}/conf/nginx.logrotate /etc/logrotate.d/nginx
fi

%preun                              # 卸载前准备操作
if [ $1 == 0 ]; then
    /etc/init.d/nginx stop 2>&1 /dev/null
    /usr/sbin/userdel -r nginx 2> /dev/null
fi

%postun
if [ $1 == 0 ]; then                # 卸载后准备操作
    rm -f /etc/logrotate.d/nginx
fi

%clean
rm -rf %{buildroot}

%files                              # 定义rpm包安装时创建的相关目录及文件。在该选项中%defattr (-,root,root)一定要注意。它是指定安装文件的属性，分别是(mode,owner,group)，-表示默认值，对文本文件是0644，可执行文件是0755。
%defattr(-,root,root,0755)
%{_prefix}
%dir /var/log/weblog  # 将指定目录当作空目录
%attr(644,root,root) %{_prefix}/conf/nginx.conf
%attr(755,root,root) /etc/rc.d/init.d/nginx

%changelog
* Fri Feb 22 2019 <XXX@XXX> - 1.12.1-3
- Initial Version
- Update Installtion
- Add Logrotate Feature
- Fix Uninstall Bug With logrotate

# End Of nginx.spec
```



# 附录

## 查看src.rpm包的spec制作流程

```shell
[xiaofei@centos6 tmp]$ rpm2cpio nginx-1.16.1-3.el6.src.rpm | cpio -id
[xiaofei@centos6 tmp]$ ls
"""
nginx-1.16.1-3.el6.src.rpm  nginx.params  nginx.sysinit
nginx-1.16.1.tar.gz         nginx.spec    yum.log
"""
```



## rpmbuild相关命令

基本格式：rpmbuild [options] [spec文档|tarball包|源码包] 

查看相关宏定义：

```shell
[root@centos6 ~]# rpmbuild --showrc
```



从spec文档建立有以下选项： 

制作过程的几个状态

rpmbuild -bp   执行到%prep

rpmbuild -bc    执行到%build中的config

rpmbuild -bi     执行至%build中的install

rpmbuild -ba    制作二进制和源码格式的包

rpmbuild -bs     仅制作源码格式的rpm包

rpmbuild -bb    仅制作二进制格式rpm包

rpmbuild -bl      检测BUILDROOT目录下的文件和%files指定的文件是否一致



## spec文档中常用的几个宏(变量)

1. RPM_BUILD_DIR:  /usr/src/redhat/BUILD 
2. RPM_BUILD_ROOT:  /usr/src/redhat/BUILDROOT 
3. %{_sysconfdir}:  /etc 
4. %{_sbindir}：   /usr/sbin 
5. %{_bindir}:    /usr/bin 
6. %{_datadir}:   /usr/share 
7. %{_mandir}:    /usr/share/man 
8. %{_libdir}:    /usr/lib64 
9. %{_prefix}:    /usr 
10. %{_localstatedir}:  /usr/var 

## rpm常用命令

- 手动安装 rpm 包 `rpm-ivh xxxxx.rpm`

- 查看 rpm 包信息 `rpm-qpi xxxxx.rpm`

- 查看 rpm 包依赖 `rpm -qpR xxxxx.rpm`

- 查看 rpm 包中包含那些文件 `rpm -qlp xxxxx.rpm`  可以加grep搜索 `rpm -qlp xxxxx.rpm|grep spec`

- 使用工具rpm2cpio提取文件： `rpm2cpio xxxxx.rpm |cpio -ivd xxx.jpg`

- 用rpm2cpio将rpm文件转换成cpio文件 `rpm2cpio xxxxxx.rpm >xxxxx.cpio`

- 用cpio解压cpio文件 `cpio -i --make-directories`

- 提取所有文件： `rpm2cpio xxx.rpm | cpio -vi` `rpm2cpio xxx.rpm | cpio -idmv` `rpm2cpio xxx.rpm | cpio --extract --make-directories` 

- cpio 参数说明:  i 和 extract 表示提取文件 v 表示指示执行进程 d 和 make-directory 表示根据包中文件原来的路径建立目录   m 表示保持文件的更新时间

- 查看rpm包里的pre和post install脚本： `rpm -qp --scripts xxxxx.rpm` 

- 查看安装的过程中，代码的执行过程： `rpm -ih -vv xxxxx.rpm`

- RPM包添加签名验证：

  ```shell
  gpg --gen-key
  gpg --list-keys
  gpg --export -a "Mage Edu" > RPM-GPG-KEY-magedu
  rpm --addsign nginx-1.16.1.rpm
  rpm --import RPM-GPG-KEY-magedu
  rpm --checksig nginx-1.16.1.rpm
  ```

  

## nginx服务启动文件

```shell
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /applications/nginx/conf/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /applications/nginx/run/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/etc/nginx/conf/nginx.conf"

[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx

lockfile=/var/lock/subsys/nginx

make_dirs() {
   # make required directories
   user=`nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   options=`$nginx -V 2>&1 | grep 'configure arguments:'`
   for opt in $options; do
       if [ `echo $opt | grep '.*-temp-path'` ]; then
           value=`echo $opt | cut -d "=" -f 2`
           if [ ! -d "$value" ]; then
               # echo "creating" $value
               mkdir -p $value && chown -R $user $value
           fi
       fi
   done
}

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```



## nginx.logrotate

```shell
/var/log/nginx/*.log {
	daily
	missingok
	rotate 14
	compress
	delaycompress
	notifempty
	create 0640 www-data adm
	sharedscripts
	prerotate
		if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
			run-parts /etc/logrotate.d/httpd-prerotate; \
		fi \
	endscript
	postrotate
		invoke-rc.d nginx rotate >/dev/null 2>&1
	endscript
}
```



## zabbix-logrotate

```shell
/var/log/zabbix/zabbix_agentd.log {
    weekly
    rotate 12
    compress
    delaycompress
    missingok
    notifempty
    create 0640 zabbix zabbix
}
```



## php-logrotate

```shell
/var/log/php7.0-fpm.log {
	rotate 12
	weekly
	missingok
	notifempty
	compress
	delaycompress
	postrotate
		/usr/lib/php/php7.0-fpm-reopenlogs
	endscript
}

/var/log/php/*.log {
        rotate 12
        daily
        missingok
        notifempty
        compress
        delaycompress 
}
```



## fastcgi_params内容

```shell
fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx;
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;
```

