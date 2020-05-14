[TOC]

# crontab

## 服务程序

依赖于crond服务程序,确保开机启动.

```shell
chkconfig --list | grep crond

/etc/init.d/crond status
```



## 日志文件

/var/log/cron



## 帮助信息

```shell
man crontab
```

说明：

| 参数      | 含义                            |
| --------- | ------------------------------- |
| -l(字母） | 查看crontab文件内容             |
| -e        | 编辑crontab文件内容             |
| -r        | 删除crontab文件内容（用的很少） |
| -u user   | 指定使用的用户执行任务          |

**特别强调：-r参数在生产中很少用，没什么特殊需求必须要用-e进入编辑即可**

**补充：**

crontab { -l | -e } 实际上就是在操作/var/spool/cron/当前用户这样的文件
使用crontab命令的优点：
1，crontab可以检查语法
2，输入方便

## 时间格式

| 段     | 含义           |
| ------ | -------------- |
| 第一段 | 代表分钟       |
| 第二段 | 代表小时       |
| 第三段 | 代表日，天     |
| 第四段 | 代表月份       |
| 第五段 | 代表星期，周几 |

**提示：**时间记忆口诀：分时日月周。取值范围记忆：正常日期时间范围

示例

```shell
01 * * * * cmd # 每小时的01分钟执行
01 9 * * * cmd # 每天9点的01分钟执行
01 09 * * 00 cmd # 每周日的9点01分执行
00 09 01 * * cmd # 每月1日的9点00分执行
```



## 特殊符号

| 特殊符号 | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| *        | *号，表示任意时间都，实际就是“每”的意思                      |
| -        | 减号表示分隔符，表示一个时间范围，区间段，如17-19点，例如：每天的17，18，19点的00分执行任务。00 17-19 * * * cmd |
| ，       | 逗号，表示分隔时段的意思，例如：每天的5点10点00分执行任务，00 5,10 * * * cmd |
| /n       | n代表数字，即“每隔n单位时间”，例如：每10分钟执行一次任务可以写成\*/10 *  * * * cmd，*/10,\*的范围是0-59，因此也可以写成0-59/10 |



# 生产示例

```shell
# 每5分钟同步一次时间
*/5 * * * * /sbin/ntpdate ntp1.aliyun.com >/dev/null 2>&1

# 每周六,日上午9点和下午14点执行指定脚本
00 09,14 * * 6,0 /bin/sh /root/test.sh >> /root/crontab.log

# 每12小时备份一次/etc目录至/backups目录中，保存格式为etc-yyy-mm-dd-hh.tar.xz”
01 */12 * * * root tar Jcf /backups/etc-$(date +"\%F-\%H-\%S").tar.xz /etc/

# 每周2、4、7备份/var/log/secure文件至/logs目录中，文件名格式为 secure-yyyymmdd
01 09 * * 2,4,7 root tar Jcf /logs/secure-$(date +"\%Y\%m\%d") /var/log/secure

# 每两个小时取当前系统/proc/meminfo文件中以S或M开头的行信息追加至/tmp/meminfo.txt文件中
01 */2 * * * root grep '^[M\|S]' /proc/meminfo > /tmp/meminfo.txt
```

*注意*：

如果定时任务规则结尾不加>/dev/null 2>&1，很容易导致硬盘inode空间被占满，从而系统服务不正常。当一个定时任务执行的时候,就会给系统发一封邮件,如果没有开启邮件服务器,则会产生临时文件.

| 目录名                       | 解释                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| /var/spool/clientmqueue      | `centos5.x`sendmail临时邮件文件目录，有很多原因会导致这个目录碎文件很多，比如crontab定时任务命令不加>/dev/null等，并且sendmail服务没开。工作中偶尔会因为该目录文件太多，导致/var所在的分区inode数量被消耗尽，无法写入文件的情况 |
| /var/spool/postfix/maildrop/ | `centos6.x`  postfix临时队列目录/var/spool/postfix/maildrop/默认定时任务执行时会给root发邮件，如果邮件服务不开，就会把邮件推到上述目录。当定时任务结尾不加>/dev/null 2>&1的时候，定时任务就会在上述目录存大量小文件 |

*解决办法*

```shell
# 1,定期删除
ls /var/spool/clientmqueue/ | xargs rm -f
ls /var/spool/postfix/maildrop/ | xargs rm -f

# 2,编辑crontab配置文件,将MAILTO=root替换成MAILTO="",然后service crond restart即可,如果还不行,crontab -e 第一行增加MAILTO=""
cat /etc/crontab
"""
SHELL=/bin/bash #shell解释器
PATH=/sbin:/bin:/usr/sbin:/usr/bin #PATH环境变量
MAILTO=root #定义如果任务有输出，发给哪个用户，默认发给root用户
HOME=/ #定时任务执行命令从根目录开始
"""
```



# 日志轮询

周期性切割日志
系统定时任务 + logrotate

```shell
[root@chensiqi1 ~]# cat /etc/cron.daily/logrotate 
"""
#!/bin/sh
/usr/sbin/logrotate /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0
"""

[root@chensiqi1 ~]ll /var/log/messages*
-rw-------. 1 root root   58049 Feb 10 23:18 /var/log/messages
-rw-------. 1 root root 1492005 Jan  2 06:51 /var/log/messages-20170102
-rw-------. 1 root root  633737 Jan  8 08:02 /var/log/messages-20170108
-rw-------. 1 root root 1594144 Feb  4 04:25 /var/log/messages-20170204
-rw-------. 1 root root   21512 Feb  6 03:41 /var/log/messages-20170206
[root@chensiqi1 ~]# ll /var/log/secure*
-rw-------. 1 root root  4810 Feb 10 22:39 /var/log/secure
-rw-------. 1 root root 64822 Jan  2 06:27 /var/log/secure-20170102
-rw-------. 1 root root 14187 Jan  8 07:22 /var/log/secure-20170108
-rw-------. 1 root root 13540 Jan 12 00:17 /var/log/secure-20170204
-rw-------. 1 root root  5723 Feb  6 02:50 /var/log/secure-20170206
```





# 使用者权限及定时任务文件

| 文件                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| /etc/cron.deny(拒绝)  | 该文件中所列用户不允许使用crontab                            |
| /etc/cron.allow(允许) | 该文件优先级高于cron.deny(默认不存在，一般不用)              |
| /var/spool/cron/      | **所有用户crontab配置文件默认都存在此目录，文件名以用户名命名 |