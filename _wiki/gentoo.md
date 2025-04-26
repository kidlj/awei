---
title: Gentoo
---

### Systemd

	# cat <<EOF > /etc/portage/package.use/systemd
	sys-apps/systemd boot
	sys-kernel/installkernel systemd-boot dracut
	EOF

### Systemd Network

#### DHCP

	# mkdir -p /etc/systemd/network
	# cat > /etc/systemd/network/10-wired.network <<EOF
	[Match]
	Name=en*

	[Network]
	DHCP=yes
	EOF

#### Static IP

	# cat > /etc/systemd/network/10-wired.network <<EOF
	[Match]
	Name=en*

	[Network]
	Address=192.168.1.100/24
	Gateway=192.168.1.1
	DNS=192.168.1.1
	EOF

#### DNS

	# ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf

#### 重启服务

	# systemctl restart systemd-networkd
	# systemctl restart systemd-resolved


### Custom Ebuilds

可以很简单地建立一个本地 overlay，用于存放各种分散的 ebuilds。

举个例子，想要安装 `fcitx-sogoupinyin` 的话，可以添加 `gentoo-zh` 这个 overlay，这是标准做法。可是这个 overlay 太大了，如果仅仅想要安装这一个包该怎么办呢，这时就可以用建立本地 overlay 的方法了。

#### 建立本地 overlay

	$ sudo mkdir /var/lib/layman/custom
	$ sudo mkdir /var/lib/layman/custom/profiles
	$ sudo echo "custom" > /var/lib/layman/custom/profiles/repo_name

然后在 `/var/lib/layman/make.conf` 文件里适当的位置加上一行，
	
	/var/lib/layman/custom

这样这个叫做 `custom` 的 overlay 就建好了。

#### 放置 ebuild

下面的工作就是把第三方或者自己维护的 ebulds 放置在这个 overlay 里，以 `fcitx-sogoupinyin` 为例：

	$ cd /var/lib/layman/custom
	$ sudo mkdir -p app-i18n/fcitx-sogoupinyin/
	$ cd app-i18n/fcitx-sogoupinyin/
	$ cp ~/fcitx-sogoupinyin-0.0.6.ebuld .
	
然后重要的一步，为该 ebuild 生成 Manifest 文件：
	
	$ ebuld fcitx-sogoupinyin-0.0.6.eduild digest

如果成功，该 ebuild 就可以通过 Portage 使用了。

#### 安装 package

	$ sudo emerge -av fcitx-sogoupinyin


### Suspend/Wakeup

	# vim /etc/systemd/logind.conf
	HandleLidSwitch=lock
	HandleLidSwitchExternalPower=lock

	# cat <<EOF > /etc/systemd/system/auto-suspend.timer
	[Unit]
	Description=Automatically suspend on a schedule

	[Timer]
	OnCalendar=*-*-* 02:00:00

	[Install]
	WantedBy=timers.target
	EOF

	# cat <<EOF > /etc/systemd/system/auto-suspend.service
	[Unit]
	Description=Suspend

	[Service]
	Type=oneshot
	ExecStart=/usr/bin/systemctl suspend
	EOF

	# cat <<EOF > /etc/systemd/system/auto-resume.timer
	[Unit]
	Description=Automatically resume on a schedule

	[Timer]
	OnCalendar=*-*-* 08:00:00
	WakeSystem=true

	[Install]
	WantedBy=timers.target
	EOF

	# cat <<EOF > /etc/systemd/system/auto-resume.service
	[Unit]
	Description=Does nothing

	[Service]
	Type=oneshot
	ExecStart=/bin/true
	EOF

	# systemctl enable auto-suspend.timer
	# systemctl start auto-suspend.timer

	# systemctl enable auto-resume.timer
	# systemctl start auto-resume.timer

### Console backlight

	# echo 0 > /sys/class/backlight/intel_backlight/brightness