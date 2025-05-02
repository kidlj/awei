---
title: Gentoo
---

### Systemd

    # cat <<EOF > /etc/portage/package.use/systemd
    sys-apps/systemd boot
    sys-kernel/installkernel systemd-boot dracut
    EOF

### Network

#### DHCP

    # mkdir -p /etc/systemd/network
    # cat <<EOF > /etc/systemd/network/10-wired.network <<EOF
    [Match]
    Name=en*

    [Network]
    DHCP=yes
    EOF

#### Static IP

    # cat <<EOF > /etc/systemd/network/10-wired.network <<EOF
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

### Wireless Network

#### wpa_supplicant

    # emerge --ask wpa_supplicant

    # mkdir -p /etc/wpa_supplicant
    # wpa_passphrase "你的SSID" "你的WiFi密码" | sudo tee /etc/wpa_supplicant/wpa_supplicant-wlp44s0.conf

    # systemctl enable wpa_supplicant@wlp44s0.service
    # systemctl start wpa_supplicant@wlp44s0.service

#### DHCP

    # cat <<EOF > /etc/systemd/network/25-wireless.network
    [Match]
    Name=wlp44s0

    [Network]
    DHCP=yes
    EOF

#### Static IP

    # cat <<EOF > /etc/systemd/network/25-wireless.network
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

#### Lid Switch Behavior

    # vim /etc/systemd/logind.conf
    HandleLidSwitch=lock
    HandleLidSwitchExternalPower=lock

#### auto-suspend.timer

    # cat <<EOF > /etc/systemd/system/auto-suspend.timer
    [Unit]
    Description=Automatically suspend on a schedule

    [Timer]
    OnCalendar=*-*-* 02:00:00

    [Install]
    WantedBy=timers.target
    EOF

#### auto-suspend.service

    # cat <<EOF > /etc/systemd/system/auto-suspend.service
    [Unit]
    Description=Suspend

    [Service]
    Type=oneshot
    ExecStart=/usr/bin/systemctl suspend
    EOF
        
#### auto-resume.timer

    # cat <<EOF > /etc/systemd/system/auto-resume.timer
    [Unit]
    Description=Automatically resume on a schedule

    [Timer]
    OnCalendar=*-*-* 08:00:00
    WakeSystem=true

    [Install]
    WantedBy=timers.target
    EOF
                   
#### auto-resume.service

    # cat <<EOF > /etc/systemd/system/auto-resume.service
    [Unit]
    Description=Does nothing

    [Service]
    Type=oneshot
    ExecStart=/bin/true
    EOF

#### Start timers

    # systemctl enable auto-suspend.timer
    # systemctl start auto-suspend.timer

    # systemctl enable auto-resume.timer
    # systemctl start auto-resume.timer

### Console backlight

    # echo 0 > /sys/class/backlight/intel_backlight/brightness