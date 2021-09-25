---
layout:     post
title:      "ubuntu rootfs"
subtitle:   " \"制作rootfs\""
author:     "qiuxi"
catalog: true
tags:
    - linux
    - rootfs
---

## 简介

## 流程
### ubuntu基础包
* [ubuntu-base-20.04.3-base-arm64.tar.gz](http://cdimage.ubuntu.com/ubuntu-base/releases/20.04/release/ubuntu-base-20.04.3-base-arm64.tar.gz)

* [ch-mount.sh](https://raw.githubusercontent.com/psachin/bash_scripts/master/ch-mount.sh)

### 关键步骤
* sudo cp -av /usr/bin/qemu-aarch64-static ./rootfs/usr/bin
* sudo cp -av /run/systemd/resolve/stub-resolv.conf ./rootfs/etc/resolv.conf

* change rootfs
```
sudo chroot ./rootfs/

# Change the setting here
USER=qiuxi
HOST=ubuntu

# Create User
useradd -G sudo -m -s /bin/bash $USER
echo $USER:$USER | chpasswd
passwd $USER
# enter user password
passwd root
# enter user password

# Hostname & Network
echo $HOST > /etc/hostname
echo "127.0.0.1    localhost.localdomain localhost" > /etc/hosts
echo "127.0.0.1    $HOST" >> /etc/hosts

# Enable serial console
ln -s /lib/systemd/system/serial-getty\@.service /etc/systemd/system/getty.target.wants/serial-getty@ttyS0.service

# Install packages
apt-get update
apt-get upgrade
apt-get install dialog perl udev
apt-get install locales
locale-gen "en_US.UTF-8"
apt-get install ifupdown net-tools network-manager ethtool wireless-tools iputils-ping resolvconf wget apt-utils wpasupplicant
apt-get install sudo ssh
apt-get install vim git
apt-get install bash-completion htop alsa-utils python3 golang

# auto eth0
echo "auto eth0" > /etc/network/interfaces.d/eth0
echo "iface eth0 inet dhcp" >> /etc/network/interfaces.d/eth0
# echo "nameserver 127.0.1.1" > /etc/resolv.conf

# resolvconf and tzdata
dpkg-reconfigure resolvconf
dpkg-reconfigure tzdata

# enable login at serial console
cat << EOF > /etc/init.d/ttyS0.conf
start on stopped rc or RUNLEVEL=[12345]
stop on runlevel [!12345]
respawn
exec /sbin/getty -L 115200 ttyS0 vt102
EOF
sudo start ttyS0

```
* add fstab
```
echo "/dev/mmcblk0p2 / ext4 defaults,noatime 0 1" >> ./rootfs/etc/fstab
```
* Create a file /etc/dpkg/dpkg.cfg.d/01_nodoc which specifies the desired filters
```
cat << EOF > ./rootfs/etc/dpkg/dpkg.cfg.d/01_nodoc
path-exclude /usr/share/doc/*
# we need to keep copyright files for legal reasons
path-include /usr/share/doc/*/copyright
path-exclude /usr/share/man/*
path-exclude /usr/share/groff/*
path-exclude /usr/share/info/*
# lintian stuff is small, but really unnecessary
path-exclude /usr/share/lintian/*
path-exclude /usr/share/linda/*
EOF
```
* rm useless
```
sudo find rootfs/usr/share/doc -depth -type f ! -name copyright|xargs rm || true
sudo find rootfs/usr/share/doc -empty|xargs rmdir || true
sudo rm -rf rootfs/usr/share/man/* rootfs/usr/share/groff/* rootfs/usr/share/info/*
sudo rm -rf rootfs/usr/share/lintian/* rootfs/usr/share/linda/* rootfs/var/cache/man/* rootfs/var/cache/apt/archives/*
```
* ext4 img
```
dd if=/dev/zero of=rootfs.ext4 bs=1M count=2048
mkdir -p ext4_dir
mount -t ext4 ./rootfs.ext4 ./ext4_dir
cp -rf ./rootfs/* ./ext4_dir/
```

## 参考
* https://a-delacruz.github.io/ubuntu/rpi3-setup-filesystem.html
* https://gnu-linux.org/building-ubuntu-rootfs-for-arm.html
* http://opensource.rock-chips.com/wiki_Distribution
* https://wiki.t-firefly.com/en/Core-3568J/
* https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/185106950/Open+Source+Projects
