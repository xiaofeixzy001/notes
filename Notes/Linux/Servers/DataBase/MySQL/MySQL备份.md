[TOC]

mysqldump

lvm

XtraBackup：https://www.percona.com/software/mysql-database/percona-xtrabackup

```shell
# 全备
innobackupex --default-file=/usr/local/mysql/my.cnf --user=root --password=redhat /bcakup/2015/38/1

# 全备恢复
innobackupex --apply-log /bcakup/2015/38/1
innobackupex --defaults-file=/usr/local/mysql/my.cnf --copy-back /bcakup/2015/38/1/xxx

# 增量备份
# 先做一次全备
innobackupex --defaults-file=/usr/local/mysql/my.cnf --user=root --password=redhat /bcakup/2015/38/1

# 基于全备之后的第一次增量备份
innobackupex --defaults-file=/usr/local/mysql/my.cnf --user=root --password=redhat --incremental --incremental-basedir=/bcakup/2015/38/1/xxx /backup/2015/38/2

# 基于第一次增量备份后的再一次增量备份
innobackupex --defaults-file=/usr/local/mysql/my.cnf --user=root --password=redhat --incremental --incremental-basedir=/bcakup/2015/38/2/xxx /backup/2015/38/3

# 增量备份的恢复
# 准备完全备份
innobackupex --apply-log --redo-only /bcakup/2015/38/1/xxx

# 准备第一次增量备份
innobackupex --apply-log --redo-only /bcakup/2015/38/1/xxx --incremental-dir=/backup/2015/38/2/xxx

# 准备第二次增量备份
innobackupex --apply-log --redo-only /bcakup/2015/38/1/xxx --incremental-dir=/backup/2015/38/3/xxx

# 恢复数据
innobackupex --defaults-file=/usr/local/mysql/my.cnf --copy-back /backup/2015/38/1/xxx

# 修改data目录属性
chown -R mysql.mysql /data/mysql
```































