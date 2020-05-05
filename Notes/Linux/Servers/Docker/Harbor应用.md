[TOC]

# 配置存储后端 

默认情况下，Harbor将映像存储在本地文件系统上。在生产环境中，您可以考虑使用其他存储后端而不是本地文件系统，如S3，Openstack Swift，Ceph等。您需要更新的是storage文件中的部分common/templates/registry/config.yml。例如，如果您使用Openstack Swift作为存储后端，则该部分可能如下所示：

 

```
storage:
  swift:
    username: admin
    password: ADMIN_PASS
    authurl: http://keystone_addr:35357/v3/auth
    tenant: admin
    domain: default
    region: regionOne
    container: docker_images
```

 

```
storage:
  swift:
    username: admin
    password: ADMIN_PASS
    authurl: http://keystone_addr:35357/v3/auth
    tenant: admin
    domain: default
    region: regionOne
    container: docker_images
```

注意：有关注册表的存储后端的详细信息，请参阅https://docs.docker.com/registry/configuration/

# 公证人安装

要使用公证服务安装Harbor，请在运行时添加参数install.sh：

 

```
sudo ./install.sh --with-notary
```

 

```
sudo ./install.sh --with-notary
```

注意：对于使用公证人员进行安装，必须将参数ui_url_protocol设置为“https”。

有关Notary和Docker Content Trust的更多信息，请参阅Docker的文档：https://docs.docker.com/engine/security/trust/content_trust/

有关如何使用Harbor 的信息，请参阅Harbor 用户指南：https://github.com/goharbor/harbor/blob/master/docs/user_guide.md

# 持久性数据和日志文件

默认情况下，注册表数据将保留在主机的/data/目录中。即使拆除和/或重建Harbor 的集装箱，这些数据也保持不变。

另外，Harbor使用rsyslog收集每个容器的日志。默认情况下，这些日志文件存储在/var/log/harbor/目标主机上的目录中进行故障排除。

# 配置在自定义端口上监听港口

默认情况下，对于admin portal和docker命令，Harbor将在端口80（HTTP）和443（如果已配置HTTPS）上进行监听，则可以使用自定义方式进行配置。

## 对于HTTP协议

1.修改docker-compose.yml

将第一个“80”替换为自定义的端口，例如8888：80。

 

```
proxy:
    image: library/nginx:1.11.5
    restart: always
    volumes:
      - ./config/nginx:/etc/nginx
    ports:
      - 8888:80
      - 443:443
    depends_on:
      - mysql
      - registry
      - ui
      - log
    logging:
      driver: "syslog"
      options:  
        syslog-address: "tcp://127.0.0.1:1514"
        tag: "proxy"
```

 

```
proxy:
    image: library/nginx:1.11.5
    restart: always
    volumes:
      - ./config/nginx:/etc/nginx
    ports:
      - 8888:80
      - 443:443
    depends_on:
      - mysql
      - registry
      - ui
      - log
    logging:
      driver: "syslog"
      options:  
        syslog-address: "tcp://127.0.0.1:1514"
        tag: "proxy"
```

2.修改ports.cfg，将端口添加到参数“hostname”

 

```
hostname = 192.168.0.2:8888
```

 

```
hostname = 192.168.0.2:8888
```

3.重新部署港口,参考上一节“管理Harbor的生命周期”

## 对于HTTPS协议

1,启用Harbor 中的HTTPS 。

2,修改docker-compose.yml

将第一个“443”替换为自定义的端口，例如8888：443。

修改docker-compose.yml

将第一个“443”替换为自定义的端口，例如8888：443

 

```
proxy:
    image: library/nginx:1.11.5
    restart: always
    volumes:
      - ./config/nginx:/etc/nginx
    ports:
      - 80:80
      - 8888:443
    depends_on:
      - mysql
      - registry
      - ui
      - log
    logging:
      driver: "syslog"
      options:  
        syslog-address: "tcp://127.0.0.1:1514"
        tag: "proxy"
```

 

```
proxy:
    image: library/nginx:1.11.5
    restart: always
    volumes:
      - ./config/nginx:/etc/nginx
    ports:
      - 80:80
      - 8888:443
    depends_on:
      - mysql
      - registry
      - ui
      - log
    logging:
      driver: "syslog"
      options:  
        syslog-address: "tcp://127.0.0.1:1514"
        tag: "proxy"
```

3.修改ports.cfg，将端口添加到参数“hostname”

 

```
hostname = 192.168.0.2:8888
```

 

```
hostname = 192.168.0.2:8888
```

4.重新部署Harbor.参考上一节“管理Harbor 的生命周期”。

# **故障排除** 

1,当Harbour不能正常工作时，请运行以下命令，查看Harbor的所有容器是否处于UP状态：

 

```
$ sudo docker-compose ps
        Name                     Command               State                    Ports                   
  -----------------------------------------------------------------------------------------------------
  harbor-db           docker-entrypoint.sh mysqld      Up      3306/tcp                                 
  harbor-jobservice   /harbor/harbor_jobservice        Up                                               
  harbor-log          /bin/sh -c crond && rsyslo ...   Up      127.0.0.1:1514->514/tcp                    
  harbor-ui           /harbor/harbor_ui                Up                                               
  nginx               nginx -g daemon off;             Up      0.0.0.0:443->443/tcp, 0.0.0.0:80->80/tcp 
  registry            /entrypoint.sh serve /etc/ ...   Up      5000/tcp
```

 

```
$ sudo docker-compose ps
        Name                     Command               State                    Ports                   
  -----------------------------------------------------------------------------------------------------
  harbor-db           docker-entrypoint.sh mysqld      Up      3306/tcp                                 
  harbor-jobservice   /harbor/harbor_jobservice        Up                                               
  harbor-log          /bin/sh -c crond && rsyslo ...   Up      127.0.0.1:1514->514/tcp                    
  harbor-ui           /harbor/harbor_ui                Up                                               
  nginx               nginx -g daemon off;             Up      0.0.0.0:443->443/tcp, 0.0.0.0:80->80/tcp 
  registry            /entrypoint.sh serve /etc/ ...   Up      5000/tcp
```

如果容器未处于UP状态，请检查目录中该容器的日志文件/var/log/harbor。例如，如果容器harbor-ui没有运行，则应该查看日志文件ui.log。

2,当设置Harbor 后面的nginx的代理或弹性负载均衡，寻找线下，在common/templates/nginx/nginx.http.conf从部分删除它，如果代理已经有类似的设置：location /，location /v2/和location /service/。

 

```
proxy_set_header X-Forwarded-Proto $scheme;
```

 

```
proxy_set_header X-Forwarded-Proto $scheme;
```

并重新部署Harbor 参考上一节“管理Harbor 的生命周期”。

3，拉取镜像时报错如下

 

```
docker push xx.xxx.xx.xx/calico/node
"""
The push refers to a repository [xx.xxx.xx.xx/calico/node]

5a5054a0b567: Preparing

dc759f36d103: Preparing

0ae8598a5313: Preparing

b7fc58bf47e2: Preparing

799d9a47057e: Waiting

503925f2fc18: Waiting

unauthorized: authentication required
"""
```

 

```
docker push xx.xxx.xx.xx/calico/node
"""
The push refers to a repository [xx.xxx.xx.xx/calico/node]
5a5054a0b567: Preparing
dc759f36d103: Preparing
0ae8598a5313: Preparing
b7fc58bf47e2: Preparing
799d9a47057e: Waiting
503925f2fc18: Waiting
unauthorized: authentication required
"""
```

如果权限都没有问题的话，那就是在Harbor里面没有calico项目，Harbor要求xx.xxx.xx.xx/calico/node中第一个/后面的为项目名称，要求Harbor中存在这个项目名称，否则就会报这个错误。

4,Harbor只支持Registry V2 API，因此你需要使用Docker1.6以及以上的客户端。

5, 访问仓库时提示如下：

 

```
curl https://192.168.1.200/v2/
"""
curl: (60) Peer's Certificate issuer is notrecognized.
More details here:http://curl.haxx.se/docs/sslcerts.html
curl performs SSL certificate verificationby default, using a "bundle"
 ofCertificate Authority (CA) public keys (CA certs). If the default
 bundlefile isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificatesigned by a CA represented in
 thebundle, the certificate verification probably failed due to a
 problem with the certificate (it might beexpired, or the name might
 notmatch the domain name in the URL).
If you'd like to turn off curl'sverification of the certificate, use
 the-k (or --insecure) option.
```

 

```
curl https://192.168.1.200/v2/
"""
curl: (60) Peer's Certificate issuer is notrecognized.
More details here:http://curl.haxx.se/docs/sslcerts.html
curl performs SSL certificate verificationby default, using a "bundle"
 ofCertificate Authority (CA) public keys (CA certs). If the default
 bundlefile isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificatesigned by a CA represented in
 thebundle, the certificate verification probably failed due to a
 problem with the certificate (it might beexpired, or the name might
 notmatch the domain name in the URL).
If you'd like to turn off curl'sverification of the certificate, use
 the-k (or --insecure) option.
```

此种情况多发生在自签名的证书，报错含义是签发证书机构未经认证，无法识别。

解决办法是将签发该证书的私有CA公钥ca.crt文件内容，追加到/etc/pki/tls/certs/ca-bundle.crt。

 

```
cat/etc/docker/certs.d/192.168.1.200/ca.crt >>/etc/pki/tls/certs/ca-bundle.crt
curl https://192.168.1.200/v2/
{"errors":[{"code":"UNAUTHORIZED","message":"authenticationrequired","detail":null}]}
```

 

```
cat/etc/docker/certs.d/192.168.1.200/ca.crt >>/etc/pki/tls/certs/ca-bundle.crt
curl https://192.168.1.200/v2/
{"errors":[{"code":"UNAUTHORIZED","message":"authenticationrequired","detail":null}]}
```

6, 报错如下

 

```
docker pull xx.xxx.xx.xx:5000/vmware/harbor-db:0.4.5
"""
Error response from daemon: Get https://xx.xxx.xx.xx:5000/v1/_ping:http: server gave HTTP response to HTTPS client
"""

# 解决方法：
# 修改/usr/lib/systemd/system/docker.service文件，在ExecStart中添加--insecure-registry内容：
ExecStart=/usr/bin/dockerd--insecure-registry=xx.xxx.xx.xx:5000

# 然后重启Docker服务：
systemctl restart docker

# 然后再登录执行pull就可以了：
docker login xx.xxx.xx.xx:5000
"""
Username: admin
Password:
Login Succeeded
"""

# 再次访问
docker pull xx.xxx.xx.xx:5000/calico/node
```

 

```
docker pull xx.xxx.xx.xx:5000/vmware/harbor-db:0.4.5
"""
Error response from daemon: Get https://xx.xxx.xx.xx:5000/v1/_ping:http: server gave HTTP response to HTTPS client
"""
# 解决方法：
# 修改/usr/lib/systemd/system/docker.service文件，在ExecStart中添加--insecure-registry内容：
ExecStart=/usr/bin/dockerd--insecure-registry=xx.xxx.xx.xx:5000
# 然后重启Docker服务：
systemctl restart docker
# 然后再登录执行pull就可以了：
docker login xx.xxx.xx.xx:5000
"""
Username: admin
Password:
Login Succeeded
"""
# 再次访问
docker pull xx.xxx.xx.xx:5000/calico/node
```

# 基于https访问

1，修改配置文件harbor.cfg

 

```
hostname = k8s-harbor1.example.com
ui_url_protocol = https

ssl_cert = /usr/local/src/harbor/cert/server.crt
ssl_cert_key = /usr/local/src/harbor/cert/server.key
harbor_admin_password = 123456
```

 

```
hostname = k8s-harbor1.example.com
ui_url_protocol = https
ssl_cert = /usr/local/src/harbor/cert/server.crt
ssl_cert_key = /usr/local/src/harbor/cert/server.key
harbor_admin_password = 123456
```

2，创建用于存储证书的目录，并生成证书

 

```
mkdir  /usr/local/src/harbor/cert

# 生成私有key
openssl genrsa -out /usr/local/src/harbor/cert/server.key 2048

# 创建有效期时间的自签名证书
openssl req -x509 -new -nodes -key /usr/local/src/harbor/cert/server.key  -subj "/CN=k8s-harbor1.example.com" -days 7120 -out /usr/local/src/harbor/cert/server.crt

openssl req -x509 -new -nodes -key /usr/local/src/harbor/cert/server.key -subj "/CN=k8s-harbor2.example.com" -days 7120 -out /usr/local/src/harbor/cert/server.crt
```

 

```
mkdir  /usr/local/src/harbor/cert
# 生成私有key
openssl genrsa -out /usr/local/src/harbor/cert/server.key 2048
# 创建有效期时间的自签名证书
openssl req -x509 -new -nodes -key /usr/local/src/harbor/cert/server.key  -subj "/CN=k8s-harbor1.example.com" -days 7120 -out /usr/local/src/harbor/cert/server.crt
openssl req -x509 -new -nodes -key /usr/local/src/harbor/cert/server.key -subj "/CN=k8s-harbor2.example.com" -days 7120 -out /usr/local/src/harbor/cert/server.crt
```

3，安装docker，python-pip，docker-compose

 

```
yum install python-pip -y
pip install docker-compose

# 验证连接，在客户端连接harbor目前是连不上的，需要在harbor上签发证书给客户端
docker login k8s-harbor1.example.com
```

 

```
yum install python-pip -y
pip install docker-compose
# 验证连接，在客户端连接harbor目前是连不上的，需要在harbor上签发证书给客户端
docker login k8s-harbor1.example.com
```

4，配置客户端使用harbor：下发证书给客户端并验证

 

```
# 1，创建用于docker存储证书的目录
mkdir /etc/docker/certs.d/k8s-harbor1.example.com -pv
mkdir /etc/docker/certs.d/k8s-harbor2.example.com -pv

# 2，在harbor服务器上将证书下发
[root@k8s-harbor1 harbor]# scp cert/server.crt  192.168.100.101:/etc/docker/certs.d/k8s-harbor1.example.com/
[root@k8s-harbor2 harbor]# scp cert/server.crt  192.168.100.101:/etc/docker/certs.d/k8s-harbor2.example.com/

# 3，测试登录
[root@k8s-master1 ~]# docker login k8s-harbor1.example.com
Username (admin):  
Password: 
Login Succeeded
[root@k8s-master1 ~]# docker login k8s-harbor2.example.com
Username (admin): 
Password: 
Login Succeeded
```

 

```
# 1，创建用于docker存储证书的目录
mkdir /etc/docker/certs.d/k8s-harbor1.example.com -pv
mkdir /etc/docker/certs.d/k8s-harbor2.example.com -pv
# 2，在harbor服务器上将证书下发
[root@k8s-harbor1 harbor]# scp cert/server.crt  192.168.100.101:/etc/docker/certs.d/k8s-harbor1.example.com/
[root@k8s-harbor2 harbor]# scp cert/server.crt  192.168.100.101:/etc/docker/certs.d/k8s-harbor2.example.com/
# 3，测试登录
[root@k8s-master1 ~]# docker login k8s-harbor1.example.com
Username (admin):  
Password: 
Login Succeeded
[root@k8s-master1 ~]# docker login k8s-harbor2.example.com
Username (admin): 
Password: 
Login Succeeded
```

参考博客：https://blog.csdn.net/jiangshouzhuang/article/details/53267094