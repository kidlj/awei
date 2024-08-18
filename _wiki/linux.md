---
title: Linux
---


### 使用 ifconfig/route 配置网络

`ifconfig`和`route`命令均来自`sys-apps/net-tools`包。

	# ifconfig enp2s0 up
	# ifconfig enp2s0 192.168.1.113 broadcast 192.168.1.255 \
	netmask 255.255.255.0
	# route add default gw 192.168.1.1 enp2s0
	# echo "nameserver 202.106.0.20" > /etc/resolve.conf

### 使用 ip 配置网络

ip(sys-apps/iproute2)工具应看作是`ifconfig`的替代，因此更倾向于用此工具配置网络。

	# ip link (查看可用设备）
	# ip addr (查看地址配置）
	
	# ip link set enp2s0 up
	# ip addr add 192.168.1.114/24 dev enp2s0
	# ip route add default via 192.168.1.1

	# ip route show (查看路由表）



### 账户管理

添加新用户:

	# useradd -m -G wheel,audio,users,portage mellon

给已存在用户追加组:

	# usermod -a -G dialout,plugdev,uucp mellon

or,

	# gpasswd -a mellon dialout
	# gpasswd -a mellon uucp

将某用户从某个组除去:

	# gpasswd -d mellon dialout
	# gpasswd -d mellon uucp

创建和删除组:

	# groupadd dialout
	# groupadd uucp

	# groupdel dialout
	# groupdel uucp


### 目录的执行权限(x):

目录的执行权限表示允许用户在目录中查找，并能用`cd`命令将工作目录改到该目录。

### 特殊文件权限

1. Setuid 

	Setuid 的作用是让*执行*该命令的用户以该命令拥有者的权限去执行。考察如	   下两个文件：

		$ ls -l /bin/passwd
		-rws--x--x 1 root root 45260 Oct  5  2013 /bin/passwd

		$ ls -l /etc/passwd
		-rw-r--r-- 1 root root 1780 Oct 18  2013 /etc/passwd

	当普通用户执行`/bin/passwd`的时候，因为其上的 setuid 位，那么实际上是以    root 的身份在执行它，这样`/bin/passwd`就能够对`/etc/passwd`进行写入操作    了。

		$ passwd
		...

		$ ps -ef | grep passw[d]
		root      1758  1138  0 20:54 pts/1    00:00:00 passwd

	会注意到`passwd`命令的实际执行者为`root`。

	而 setgid 的意思是和它一样的，即让执行文件的用户以该文件所属组的权限去     执行。


2. Setgid

	暂时没见过 Setgid 安置在可执行文件上是什么作用；但下面是一个 Setgid 安     置在目录上的作用举例。

	AWS 上的文档目录`/var/www/`目录的权限设置如下：

		$ ls -ld /var/www
		drwxrwxr-x 8 root www 4096 Oct  8 07:43 /var/www

	我日常使用的`ubuntu`用户是从属于`www`组的，因此可以在这个目录下建立文件    和目录：

		$ mkdir /var/www/test
		$ ls -ld /var/www/test
		drwxrwxr-x 2 ubuntu ubuntu 4096 Oct  7 11:12 test

	新目录所属组为`ubuntu`；而如果此时给`/var/www/`目录加上 Setgid 位的话，

		$ sudo chmod g+s /var/www
		$ ls -ld /var/www
		drwxrwsr-x 8 root www 4096 Oct  8 07:43 /var/www
		$ mkdir /var/www/test_setgid
		$ ls -ld /var/www/test_setgid
		drwxrwsr-x 2 ubuntu   www    4096 Oct  8 07:42 test_setgid

	可见这个时候新建的目录所属组变成了`www`。


3. 粘滞位

	粘滞位是针对目录来说的，比如 `/tmp` 目录设置了粘滞位，虽然任何人都可以在该目录下创建和修改文件，但除了 root 用户以外，任何人不能修改别人的文件，这就是粘滞位的作用。如果先用 `mellon` 账户创建了 `/tmp/passwd.bak` 文件，那么除了 root 以外别的用户将不能再创建 `/tmp/passwd.bak`。

		$ ls -ld /tmp
		drwxrwxrwt 9 root root 240 Oct  6 21:22 /tmp

4. 特殊权限的设定 

		$ chmod u+s filename
		$ chmod 4775 filename

		$ chmod g+s filename
		$ chmod 2775 filename

		$ chmod o+t dirname
		$ chmod 1775 dirname


### System load

    cat /proc/loadavg

man proc:

> The first three fields in this file are load average figures giving the number of jobs in the run queue (state R) or waiting for disk I/O (state D) averaged over 1, 5, and 15 minutes. They are the same as the
> load average numbers given by uptime(1) and other programs. The fourth field consists of two numbers separated by a slash (/). The first of these is the number of currently runnable kernel scheduling entities
> (processes, threads). The value after the slash is the number of kernel scheduling entities that currently exist on the system. The fifth field is the PID of the process that was most recently created on the
> system.

Linux, unlike most if not all other Unix like OSes, is not only counting processes using a CPU or waiting for a CPU in the run queue as a reference for its load calculation, but also add the number of processes (threads actually) being in uninterruptible state, i.e. waiting for for a disk or network I/O to complete. The latter are actually idle, i.e. not using the CPU.[1]

There is then probably nothing to worry about your (not so) high load. The processes your are looking for are likely the single threaded redis plus transcient kernel threads.



[1]: https://unix.stackexchange.com/a/301744




