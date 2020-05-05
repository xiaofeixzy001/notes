[TOC]

# Paramiko模块

模块基于SSH用于连接远程服务器并执行相关操作

## SSHClient

用于连接远程服务器并执行基本命令

### 基于用户名密码连接：

 

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# __author__ = "xiaofei"
"""
实现远程服务器操作,如执行命令
"""

import paramiko

HOST = '172.16.2.1'
PORT = 22
USERNAME = 'root'
PWD = 'xiaofei.com'

def ssh_func(cmd):
    # 创建SSH对象
    ssh = paramiko.SSHClient()

    # 允许连接不在know_host文件中的主机
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

    # 连接服务器
    ssh.connect(hostname=HOST, port=PORT, username=USERNAME, password=PWD)

    # 执行命令
    stdin, stdout, stderr = ssh.exec_command(cmd)

    # 获取命令执行结果
    result = stdout.read().decode('utf-8')

    # 关闭连接
    ssh.close()

    return result

while True:
    cmd = input('>> ').strip()
    if not cmd:  # 如果输入为空
        continue
    elif cmd == 'q':  # 输入q退出
        break
    else:
        res = ssh_func(cmd)
        print(res)  # 打印命令执行结果
```

### 基于公钥密钥连接 

 

```
import paramiko

HOST = '172.16.2.1'
PORT = 22
USERNAME = 'root'
PWD = 'xiaofei.com'
key_path = '/home/root/.ssh/id_rsa'  # 这里的路径是基于linux下root家目录，如果在win上执行，需要改路径

CMD = 'ls'

private_key = paramiko.RSAKey.from_private_key_file(key_path)

# 创建SSH对象
ssh = paramiko.SSHClient()

# 允许连接不在know_hosts文件中的主机
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

# 连接服务器
ssh.connect(hostname=HOST, port=PORT, username=USERNAME, key=private_key)

# 执行命令
stdin, stdout, stderr = ssh.exec_command(CMD)

# 获取命令执行结果
result = stdout.read()

ssh.close()
```

### 基于私钥字符串进行连接

 

```
import paramiko
from io import StringIO

key_str = """-----BEGIN RSA PRIVATE KEY-----
...
-----END RSA PRIVATE KEY-----"""

"""
1,f=StringIO(key_str)
2,f.read()
"""
private_key = paramiko.RSAKey(file_obj=StringIO(key_str))
transport = paramiko.Transport(('10.0.1.40', 22))
transport.connect(username='wupeiqi', pkey=private_key)

ssh = paramiko.SSHClient()
ssh._transport = transport

stdin, stdout, stderr = ssh.exec_command('df')
result = stdout.read()

transport.close()

print(result)
```

### Transport()方法 

paramiko模块下有一个Transport()方法，可用于封装

#### 用户名和密码方式

 

```
import paramiko

HOST = '172.16.2.1'
PORT = 22
USERNAME = 'root'
PWD = 'xiaofei.com'

# 这两步实现了连接服务器
transport = paramiko.Transport((HOST, PORT))
transport.connect(username=USERNAME, password=PWD)

# 
ssh = paramiko.SSHClient()
ssh._transport = transport

stdin, stdout, stderr = ssh.exec_command('df')
print(stdout.read())

transport.close()
```

#### 公钥密钥方式

 

```
import paramiko

private_key = paramiko.RSAKey.from_private_key_file('/home/auto/.ssh/id_rsa')

transport = paramiko.Transport(('hostname', 22))
transport.connect(username='wupeiqi', pkey=private_key)

ssh = paramiko.SSHClient()
ssh._transport = transport

stdin, stdout, stderr = ssh.exec_command('df')

transport.close()

SSHClient 封装 Transport
```

## SFTPClient

用于连接远程服务器并执行上传下载

### 基于用户名密码上传下载

 

```
import paramiko
 
transport = paramiko.Transport(('hostname',22))
transport.connect(username='wupeiqi',password='123')
 
sftp = paramiko.SFTPClient.from_transport(transport)

# 将location.py 上传至服务器 /tmp/test.py
sftp.put('/tmp/location.py', '/tmp/test.py')

# 将remove_path 下载到本地 local_path
sftp.get('remove_path', 'local_path')
 
transport.close()
```

### 基于公钥密钥上传下载

 

```
import paramiko
 
private_key = paramiko.RSAKey.from_private_key_file('/home/auto/.ssh/id_rsa')
 
transport = paramiko.Transport(('hostname', 22))
transport.connect(username='wupeiqi', pkey=private_key )
 
sftp = paramiko.SFTPClient.from_transport(transport)

# 将location.py 上传至服务器 /tmp/test.py
sftp.put('/tmp/location.py', '/tmp/test.py')

# 将remove_path 下载到本地 local_path
sftp.get('remove_path', 'local_path')
 
transport.close()
```

### 应用实例

 

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import paramiko
import uuid

class Haproxy(object):

    def __init__(self):
        self.host = '172.16.103.191'
        self.port = 22
        self.username = 'wupeiqi'
        self.pwd = '123'
        self.__k = None

    def create_file(self):
        file_name = str(uuid.uuid4())
        with open(file_name,'w') as f:
            f.write('sb')
        return file_name

    def run(self):
        self.connect()
        self.upload()
        self.rename()
        self.close()

    def connect(self):
        transport = paramiko.Transport((self.host,self.port))
        transport.connect(username=self.username,password=self.pwd)
        self.__transport = transport

    def close(self):

        self.__transport.close()

    def upload(self):
        # 连接，上传
        file_name = self.create_file()

        sftp = paramiko.SFTPClient.from_transport(self.__transport)
        # 将location.py 上传至服务器 /tmp/test.py
        sftp.put(file_name, '/home/wupeiqi/tttttttttttt.py')

    def rename(self):

        ssh = paramiko.SSHClient()
        ssh._transport = self.__transport
        # 执行命令
        stdin, stdout, stderr = ssh.exec_command('mv /home/wupeiqi/tttttttttttt.py /home/wupeiqi/ooooooooo.py')
        # 获取命令结果
        result = stdout.read()


ha = Haproxy()
ha.run()
```