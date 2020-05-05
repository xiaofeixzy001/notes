[TOC]

# filebeat

目前logstash是部署到数据源同一台主机上，由于logstash非常的重量级，所以建议将logstash部署到单独一台主机上，然后在数据源的主机上安装beat，将收集到的数据发送至logstash服务器上，beat的output为logstash，而logstash的input为beat。

如果数据源太多，可以在中间增加redis服务器，这样就需要beat的output为redis，而logstash的input为redis。

具体要根据实际场景来回切换。

## 安装

```shell
[root@node01 ~]# rpm -ivh filebeat-5.6.16-x86_64.rpm
```

配置样例，可以参考/etc/filebeat/filebeat.full.yml

## 示例1:filebeat发送数据到logstash

```shell
[root@node01 ~]# cd /etc/filebeat/
[root@node01 filebeat]# cp filebeat.yml{,.bak}
[root@node01 filebeat]# vim filebeat.yml
"""
#=========================== Filebeat prospectors =============================
- input_type: log
  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /var/log/httpd/access_log
#----------------------------- Logstash output --------------------------------
output.logstash:
  # The Logstash hosts
  hosts: ["192.168.100.41:5044"]
"""
```

配置logstash从filebeat读数据

编辑/etc/logstash/conf.d/file-beats.conf

```shell
input {
    beats {
        port => 5044
    }
}

filter {
    grok {
        match => { "message" => "%{HTTPD_COMBINEDLOG}"}
        remove_field => "message"
    }
    date {
        match => ["timestamp", "dd/MMM/YYYY:H:m:s Z"]
        remove_field => "timestamp"
    }
    geoip {
        source => "clientip"
        target => "geoip"
        database => "/etc/logstash/maxmind/GeoLite2-City.mmdb"
    }
}

output {
    elasticsearch {
        hosts => ["http://192.168.100.41:9200/"]
        user => ""
        password => ""
        index => "logstash-%{+YYYY.MM.dd}"
        document_type => "apache_logs"
    }
}
```

启动logstash服务，查看监听端口，确定端口5044，启动filebeat服务

```shell
[root@node01 conf.d]# logstash -f file-beats.conf
[root@node01 conf.d]# ss -tnlp

[root@node01 ~]# systemctl start filebeat
[root@node01 ~]# systemctl status filebeat
[root@node01 ~]# systemctl enable filebeat

[root@node01 ~]# while true; do curl -H "X-Forwarded-For:$[$RANDOM%223+1].$[$RANDOM%255].1.1" http://192.168.100.41/test$[$RANDOM%21+1].html; sleep 1; done

[root@node01 ~]# curl http://192.168.100.41:9200/logstash-*/_search?q=response:404 | jq .
```



## 示例2:filebeat发送数据到redis

```shell
[root@node01 ~]# yum install redis -y
[root@node01 ~]# vim /etc/redis.conf
"""
bind 0.0.0.0
port 6379
requirepass 123.com  # 加密验证
"""
[root@node01 ~]# systemctl start redis
[root@node01 ~]# systemctl enable redis
[root@node01 ~]# systemctl status redis

[root@node01 filebeat]# vim filebeat.yml
"""
# 注意注释掉上面的logstash
#----------------------------- Redis output --------------------------------
output.redis:
  enabled: true
  hosts: ["192.168.100.41:6379"]
  port: 6379
  key: filebeat
  password:123.com
  db: 0
  datatype: list
"""

[root@node01 ~]# systemctl restart filebeat
```

查看redis

```shell
[root@node01 ~]# redis-cli -a 123.com
127.0.0.1:6379> KEYS *
1) "filebeat"
127.0.0.1:6379> LINDEX filebeat 0
"{\"@timestamp\":\"2020-04-15T12:32:45.337Z\",\"beat\":{\"hostname\":\"node01\",\"name\":\"node01\",\"version\":\"5.6.16\"},\"input_type\":\"log\",\"message\":\"81.65.1.1 - - [15/Apr/2020:20:24:56 +0800] \\\"GET /test18.html HTTP/1.1\\\" 200 8 \\\"-\\\" \\\"curl/7.29.0\\\"\",\"offset\":988967,\"source\":\"/var/log/httpd/access_log\",\"type\":\"log\"}"
127.0.0.1:6379> 
```

配置logstash从redis读数据

```shell
input {
    redis {
        host => "192.168.100.41"
        port => 6379
        password => "123.com"
        db => 0
        key => "filebeat"
        data_type => "list"
    }
}
```

logstash读一次redis数据，redis数据则减少

```shell
127.0.0.1:6379> LLEN filebeat
127.0.0.1:6379> LLEN filebeat
(integer) 964
127.0.0.1:6379> LLEN filebeat
(integer) 969
127.0.0.1:6379> LLEN filebeat
(integer) 844
127.0.0.1:6379> LLEN filebeat
(integer) 0
127.0.0.1:6379> LLEN filebeat
(integer) 0
127.0.0.1:6379> 
```

至此，一个比较完整的pipeline架构已搭建完毕，后期可以根据数据量，来做扩展，比如目前的redis是单点，可以做成主备模式或集群模式，还有logstash，可以做高可用，需要注意logstash收集过来的数据发送走以后记得要删除，否则对端会接收到很多重复数据。

## 日志分类传输

需求：

读取httpd的日志，如果是访问日志，给他加个字段access发送到redis，如果是错误日志，加个字段error发送到redis

```shell
[root@node01 ~]# vim /etc/filebeat/filebeat.yml
'''
- input_type: log
  paths:
    - /var/log/httpd/access_log
  fields:
    logtype: access

- paths:
    - /var/log/httpd/error_log
  fields:
    logtype: error
'''

[root@node01 ~]# systemctl restart filebeat
[root@node01 ~]# cd /etc/logstash/conf.d/
[root@node01 conf.d]# vim redis-condition-els.conf
'''
input {
    redis {
        host => "192.168.100.41"
        port => 6379
        password => "123.com"
        db => 0
        key => "filebeat"
        data_type => "list"
    }
}

filter {
    if [fields][logtype] == "access" {
        grok {
            match => { "message" => "%{HTTPD_COMBINEDLOG}"}
            remove_field => "message"
        }
        date {
            match => ["timestamp", "dd/MMM/YYYY:H:m:s Z"]
            remove_field => "timestamp"
        }
        geoip {
            source => "clientip"
            target => "geoip"
            database => "/etc/logstash/maxmind/GeoLite2-City.mmdb"
        }
    }
}

output {
    if [fields][logtype] == "access" {
        elasticsearch {
            hosts => ["http://192.168.100.41:9200/"]
            index => "logstash-%{+YYYY.MM.dd}"
            document_type => "httpd_access_logs"
        }
    } else {
        elasticsearch {
            hosts => ["http://192.168.100.41:9200/"]
            index => "logstash-%{+YYYY.MM.dd}"
            document_type => "httpd_error_logs"
        }
    }
}
'''

[root@node01 conf.d]# logstash -f redis-condition-els.conf -t
[root@node01 conf.d]# systemctl start logstash
[root@node01 conf.d]# systemctl status logstash
[root@node01 conf.d]# ps aux | grep logstash
```

conf.d目录下所有conf结尾的文件都会当作输入输出插件配置的一部分被使用。

