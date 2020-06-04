[TOC]

# 需求

web服务器定期对项目目录做备份；
单独一台备份服务器上，定期拉取web服务器上的备份数据到本地

# 主机信息


| 主机名 | ip             | 角色                     |
| ------ | -------------- | ------------------------ |
| node01 | 192.168.100.11 | 备份服务器(rsync-server) |
| node02 | 192.168.100.12 | WEB服务器(rsync-client)  |


# 配置

备份服务器配置
```
yum install -y rsync
cp /etc/rsyncd.conf{,.bak}
vim /etc/rsyncd.conf
"""
uid = xiaofei  # 用户身份,系统用户
gid = xiaofei
use chroot = no
max connections = 10
pid file = /var/run/rsyncd.pid
log file = /var/log/rsyncd.log
lock file = /var/run/rsyncd.lock
transfer logging = yes
strict modes = yes  # 是否检查密码文件权限
timeout = 900
ignore nonreadable = yes
dont compress   = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2

[backup]  # 模块名
	path = /backup/  # 模块对应的具体路径
	comment = backup export area
	read only = false
	#write only = true
	auth users = backup_user  # 虚拟用户
	secrets file = /etc/rsyncd.password  # 虚拟用户授权文件
	ignore errors = yes
	hosts allow = 192.168.100.0/24
	hosts deny = *
	list = false
"""

mkdir /backup
useradd -s /sbin/nologin -M xiaofei
chown -R xiaofei.xiaofei /backup
echo "backup_user:123456" > /etc/rsyncd.password
chmod 600 /etc/rsyncd.password
systemctl start rsyncd
ss -tnl
```

web服务器配置

```
yum install -y rsync
useradd xiaofei
mkdir /backup
chown -R xiaofei:xiaofei /backup
```

备份服务端测试
```
# upload
rsync -az /tmp/test.txt backup_user@192.168.100.11::backup

# download
rsync -az backup_user@192.168.100.11::backup /backup/
```

结合crontab自动任务计划

```
echo "123456" > /etc/rsyncd.password
chmod 600 /etc/rsyncd.password

crontab -e
"""
* * * * 1,3,5 /usr/bin/rsync --delete --password-file=/etc/rsyncd.password -az backup_user@192.168.100.11::backup /backup/
"""

```
注意：ip::后面的test是模块名，而不是目录名，一般我们都定义同样的名字，以便见名知意
# Rsync + Inotify

inotify，实时监控指定目录下文件元数据，发生变化则会触发执行指定动作。

注意需要安装在 ==rsync-client== 端

最终实现：
实时监控目标文件夹，一旦发生变化，则执行上传操作到备份服务器上。

```
yum install -y inotify-tools
cat inotify_script.sh
"""
#!/bin/bash
#
bakip=192.168.100.11
src=/backup/
dst=backup
user=backup_user
pwdfile=/etc/rsyncd.password

/usr/bin/inotifywait -mrq \
     --timefmt '%d-%m-%y %H:%M' \
     --format '%T%w%f%e' \
     -e create,delete,modify,attrib,move,close_write $src \
     | while read files;do
	/usr/bin/rsync -az --delete --progress --password-file=$pwdfile $src ${user}@${bakip}::${dst}
	echo "${files} is rsynced." >> /tmp/rsync.log 2>&1
     done
"""

nohup sh /scripts/inotify_script.sh &

touch /backup/{1..9}.html
```
注意：备份数据的用户权限


# NFS + Inodify