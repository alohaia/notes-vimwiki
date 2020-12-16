# 通过 Linux 系统来安装 Debian

> [废弃]直接通过图形界面安装即可

```bash
axel https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/debian-10.7.0-amd64-DVD-1.iso
# 安装 debootstrap
yay -S debootstrap
# 或者
axel http://ftp.debian.org/debian/pool/main/d/debootstrap/debootstrap_1.0.123_all.deb
ar -x debootstrap_1.0.123_all.deb
cd /
sudo zcat /home/aloha/Downloads/work/data.tar.gz | sudo tar xv
```
