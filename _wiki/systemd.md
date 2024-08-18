---
title: Systemd
---

### 几个好用的命令

查看启动的Unit及其之间的依赖关系:

	$ systemctl list-dependencies

查看系统安装的所有Units及其状态:
	
	$ systemctl list-unit-files

设定默认启动target:

	# systemctl set-default graphical.target


### Systemd服务诊断三部曲

以`systemd-modules-load.service`启动失败举例：

	$ systemctl --state=failed (发现`systemd-modules-load`启动失败)
	$ systemctl status systemd-modules-load (发现其PID为15630)
	$ journalctl -b _PID=15630 (发现错误日志)


### 查看一个Unit依赖的其他Units

比如查看`multi-user.target`的依赖关系：

	$ systemctl show -p "Wants" multi-user.target

此处`-p`为指定属性，可以用`systemctl show multi-user.target`查看其所有属性。


### 为某个特定设备开启服务

假设WIFI设备名为`wlp0s29f7u1`, 为其开启`wpa_supplicant`服务：

	# ln -s /usr/lib/systemd/system/wpa_supplicant@.service /etc/systemd/system/multi-user.target.wants/wpa_supplicant@wlan.service

注意`wpa_supplicant.service`和`wpa_supplicant@.service`是不同的服务。


### 不是所有Unit都可以enable或disable

不要固执地追求一个“干净的”，“最少启动项”系统，这在systemd里行不通。比如`printer.target`会自动启动，但这并不是显式地指定的。也许`systemctl disable printer.target`会可行，但当你用`systemctl enable printer.target`时会出错，因为它并没有`[Install]`部分。有些单元会在被需要时通过各种途径激活，比如通过socket, path, timer, D-Bus, udev, scripted systemctl call等。


### Journalctl

首先，将用户追到`systemd-journal`组，

	# gpasswd -a mellon systemd-journal

比较常用的选项，

	$ journalctl -e
	$ journalctl -f
	$ journalctl -b

设定最多使用50M的日志大小，`/etc/systemd/journald.conf`

	SystemMaxUse=50M


### 用Systemd设定静态网络

`network.service`:

	[Unit]
	Description=Network Connectivity

	[Service]
	Type=oneshot
	RemainAfterExit=yes

	EnvironmentFile=/etc/conf.d/network_systemd
	ExecStart=/bin/ip link set dev ${interface} up
	ExecStart=/bin/ip addr add ${address}/${netmask} broadcast ${broadcast}
	ExecStart=/bin/ip route add default via ${gateway}

	ExecStop=/bin/ip addr flush dev ${interface}
	ExecStop=/bin/ip link set dev ${interface} down

	[Install]
	WantedBy=network.target

`/etc/conf.d/network_systemd`:

	PATH=/sbin:/usr/sbin
	interface=enp2s0
	address=192.168.1.115
	netmask=255.255.255.0
	broadcast=192.168.1.255
	gateway=192.168.1.1


### RedHat系统服务启动管理

惊讶地发现，RedHat 从6.0开始采用了Canonical(Ubuntu)开发的Upstart替代sysvinit。 没错。可是奇怪的是，`initctl`命令却不好使。

RedHat用`service`命令启动或关闭服务：

	$ service nginx start|stop|status

Gentoo用`rc-update`很优雅地管理各个runlevel成员，`chkconfig`(man)可以同样做到：

	$ chkconfig nginx on (默认2, 3, 4, 5有效)
	$ chkconfig nginx off --level 1|2|3 (指定对某个runlevel生效)

*注意*，Upstart中其实并没有runlevel的概念，原本在sysvinit中用于指定各个runlevel执行的`/etc/inittab`只剩下用来指定启动缺省runlevel。



### Kernel Modules Loading

Systemd 内置 `systemd-modules-load.service` 来加载内核模块。大部分内核模块都可以根据情况自动加载，不需要做任何设置。如果想要静态设定要加载的模块，配置文件在 `/etc/modules-load.d/*.conf`。

参见 `man 5 modules-load.d`。
