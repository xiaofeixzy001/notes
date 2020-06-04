[TOC]

# 安装redis

```shell
[root@redis-node01 ~]# yum install -y redis
[root@redis-node01 ~]# grep "^[a-Z]" /etc/redis.conf
'''
bind 0.0.0.0
protected-mode yes
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
requirepass 123.com  # 加密
supervised no
pidfile /var/run/redis_6379.pid
loglevel notice
logfile ""
databases 16
save ""
stop-writes-on-bgsave-error yes
rdbcompression no  # 是否压缩
rdbchecksum no  # 是否检验
dbfilename dump.rdb
dir /var/lib/redis
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
'''

[root@redis-node01 ~]# systemctl start redis
[root@redis-node01 ~]# ss -tnlp
[root@redis-node01 ~]# systemctl enable redis
[root@redis-node01 ~]# redis-cli -a 123.com
```

