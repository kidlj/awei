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