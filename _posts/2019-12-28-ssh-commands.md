---
title: Useful ssh commands
categories: ssh
tags: [ssh, Tools, Tips]
---
# ssh

```shell
#远程连接
$ ssh -p 22 root@host

#从本地传输文件到远端host
$ tar czv file | ssh root@host 'tar xz'

#将远端host文件取回本地
$ ssh root@host 'tar cz file' | tar xzv

#动态端口转发
$ ssh -D 9001 root@host

#本地端口转发
$ ssh -L 9001:host1:host1_port root@host

#远程端口转发
$ ssh -R 9001:pi_host:22 root@host
```




