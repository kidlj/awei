---
title: Iptables
---

### Fresh new installation

`iptables -F` 并不会清除 chain policy。在用 SSH 远程操作 iptables 的时候，可以按以下流程从新配置 iptables：

	# iptables -P INPUT ACCEPT
	# iptables -F
	# iptables -A INPUT -i lo -j ACCEPT
	# iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
	# iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
	# iptables -P INPUT DROP
	# iptables -P FORWARD DROP
	# iptables -P OUTPUT ACCEPT
	# iptables -L -v

### 删除一个规则

	# iptables -vn -L --line-numbers
	# iptables -D INPUT <rule-line-number>

### 从文件中部署规则

	# vim /etc/sysconfig/iptables
	# /usr/sbin/iptables-restore < /etc/sysconfig/iptables

### 将内核中规则存入文件

	# iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
	# /usr/sbin/iptables-save > /etc/sysconfig/iptables