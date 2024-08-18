---
title: OpenSSH
---

### 指定 ssh-key

当你有多个对应不同服务器的私钥时，可以依据服务器地址指定不同的 ssh-key。

在`~/.ssh/config`文件中，填入如下内容：

	Host github.com
		IdentityFile ~/.ssh/github.pem

### 添加公钥

有一个方便的命令用于向远程机器上按照公钥，它自动将公钥内容追加在远程机器上的 `authorized_keys` 文件里，如果该文件不存在还可以自动创建。

	$ sudo -u zabbix ssh-copy-id [-i <public_key>] root@192.168.56.4

这样本地的 zabbix 用户就可以登录远程机器的 root 账户了。

### 更改 SSHD 端口流程

1. 先修改 /etc/ssh/sshd_config 文件，然后重启 sshd，已有连接不会断掉。

2. 再修改防火墙端口，更改后已建立的连接仍然不会断掉。