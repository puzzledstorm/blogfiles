---
title: "从服务器上查找文件然后复制文件到本地机器"
date: 2022-11-29T16:48:57+08:00
description: 从服务器上查找文件然后复制文件到本地机器
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: puzzled
authorEmoji: 😅
tocFolding: false
tocPosition: inner
tocLevels: ["h2", "h3", "h4"]
tags:
- python
- scp
series:
-
categories:
-
image: images/feature1/2.jpg
---



## 从远程机器上查找文件

```bash
find / -name "target file" 
```

## 远程scp

```bash
scp -r root@host:/mnt/xx.txt /mnt/
```

## 用python实现上述功能

```python
import paramiko
from scp import SCPClient
from pathlib import Path
import os
import yaml

root = Path(__file__).parent
config_path = f"{root}/deploy.config.yml"
with open(config_path, "r", encoding="utf-8") as f:
    config = yaml.safe_load(f)
    # print(config)


class Model(object):
    def __init__(self, host="your host", port="22", username="root", password="your password"):
        self.host = host
        self.port = port
        self.username = username
        self.password = password

    def find(self, target):
        # 创建SSHClient 实例对象
        ssh = paramiko.SSHClient()
        # 调用方法，表示没有存储远程机器的公钥，允许访问
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        # 连接远程机器，地址，端口，用户名密码
        ssh.connect(self.host, self.port, self.username, self.password, timeout=10)
        # 输入linux命令
        cmd = f"find / -name {target}"
        # ls = "ls"
        stdin, stdout, stderr = ssh.exec_command(cmd)
        # 输出命令执行结果
        result = stdout.read()
        r = result.decode("utf-8")
        print(result)
        try:
            t = r.split("\n")[0]
        except Exception as e:
            print(f"error: {e}")
            t = None
        # 关闭连接
        ssh.close()
        return t

    def scp(self, remote_file, local_path=None):
        ssh_client = paramiko.SSHClient()
        ssh_client.set_missing_host_key_policy(paramiko.AutoAddPolicy)
        ssh_client.connect(self.host, self.port, self.username, self.password)
        scp_client = SCPClient(ssh_client.get_transport(), socket_timeout=15.0)
        file_path_lo = local_path or os.path.dirname(os.path.realpath(__file__))
        file_path_re = remote_file

        print(file_path_lo)
        print(file_path_re)
        try:
            scp_client.get(file_path_re, file_path_lo)  # 从服务器中获取文件
        except FileNotFoundError as e:
            print(e)
            print("system could not find the specified file" + local_path)
            result = "system could not find the specified file" + local_path
        else:
            print("File downloaded successfully")
            result = "File downloaded successfully"
        ssh_client.close()
        return result

    def download(self, remote_target, local_path):
        remote_tar = self.find(remote_target)
        if remote_tar is not None:
            self.scp(remote_tar, local_path)
        else:
            print(f"{remote_target} not found")


if __name__ == '__main__':
    m = Model()
    remote_target = "xxx.json"
    m.download(remote_target, os.getcwd())

    self = __file__
    os.remove(self)
```

