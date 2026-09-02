# arch desktop devel tune Guide

## Getting Started

Welcome to the VenenuX Arch Linux guide with minimal desktop DEVEL TOOLS guide

Unfortunatelly for older devices we must recommend Haiku OS (2000 to 2012) or
then VenenuX (2012 to 2024) becouse Arch linux is only for some recent devices!
**This guide only works for most recent up to 2026** older ones must check the 
[2024](2024) or [2025](2025) or [2026](2026) subdirectories.

#### Support and Feedback

**Issues**: If you come across any problems or have specific questions, please open 
an issue on the Codeberg repository for this guide. Use Telegram OpenTech group 
if you want direct communication. https://t.me/+W94Jx4A4AzdkMmYx

**Pull Requests**: If you have improvements or additions to the guide, feel 
free to submit a pull request. Your contributions can help enhance the clarity 
of the guide for everyone. - https://codeberg.org/venenux/archresources/fork

### Step 00 optional install rsyslog and log2ram and logrotate

1. we must install those because:
    * rsyslog para archivos planos de log y registro
    * log2ram para no gastar disco SSD o sea lenta la escritura
    * logrotate para que los archivos planos no molesten
2. active the rsyslog and logrotate to generate small files
3. configure log2ram to use more ram so all those files fit in!
4. configure systemd to use less space so log2ram do not waste space

```
yay -Syu rsyslog logrotate log2ram

grep -q "imjournal" /etc/rsyslog.conf || sudo sed -i '/\$ModLoad imklog/a $ModLoad imjournal   # provides systemd traps' /etc/rsyslog.conf

systemctl enable --now rsyslog

systemctl start rsyslog

sed -i 's/\bweekly\b/daily/g' /etc/logrotate.conf

sed -i 's/\bweekly\b/daily/g' /etc/logrotate.conf

sed -i 's/\bsize\s\+.*/size 5M/g' /etc/logrotate.conf

sed -i 's/\brotate\s\+.*/rotate 1/g' /etc/logrotate.conf

systemctl restart logrotate

sed -i 's/^SIZE=.*/SIZE=1512M/' /etc/log2ram.conf

journalctl --vacuum-size=32M

sed -i -E 's/^[# ]*SystemMaxUse=.*/SystemMaxUse=128M/' /etc/systemd/journald.conf

systemctl disable log2ram-daily.timer

systemctl enable log2ram
```

### Step 01 install base devel tools

Those are need to build and clean others package and external projects.

Login as "general" user as we previously show you in previous guides.

```
yay -Syu --noconfirm less man-db perl-libwww ruby-svn2git git mercurial \
 devtools base-devel openssh binutils lld bison make cmake autoconf-archive \
 jdk-openjdk nano nodejs-lts-krypton npm
```

Configure editor

```
export EDITOR=nano
export VISUAL=nano
```

### Step 02 install codium IDE and editors

You must already has yay and desktop installed! Check the others guides.

Login as "general" user as we previously show you in previous guides.

```
yay -Syu --noconfirm --needed code code-features code-marketplace \
 sakura gnome-keyring meld dbeaver
```

### Step 03 install SQL base tools

Those are need to connect and then perform task as cliet on sql services

Login as "general" user as we previously show you in previous guides.

```
yay -Syu mariadb-lts-clients mysql-workbench sqlitebrowser sqlite
```

### Step 05 install docker

Login as "general" user as we previously show you in previous guides. 

```
doas pacman -Syu docker docker-buildx docker-compose qemu-user-static qemu-user-static-binfmt

systemctl disable --now docker.service

systemctl enable docker.socket

systemctl start docker.socket

usermod -a -G docker general
```

> **Warning** this guide will use a different disk/patch for docker layers!
 so your main disks will not be harmed by the piece of shit named docker

We will use /dev/sde3 fpr docker layers so your main SDD-s never will be harmed:

```
mkdir -p /etc/docker
mkdir -p /etc/containerd
mkdir -p /srv/docker
mkdir -p /srv/docker/dockerd
mkdir -p /srv/docker/containerd

cat > /etc/docker/daemon.json << EOF
{
  "data-root": "/srv/docker/dockerd"
}
EOF

cat > /etc/containerd/config.toml << EOF
version = 2
root = "/srv/docker/containerd"
EOF

mkfs.ext3 -b 1024 -L DOCKERS /dev/sde3

mount /dev/sdd3 /srv/docker --mkdir -o rw,commit=3600,relatime,nodiratime,dev,exec,suid,nobarrier,errors=remount-ro

cat >> /etc/fstab << EOF
/dev/sde3           /srv/docker ext3      rw,nodiratime,relatime,nobarrier,errors=remount-ro,commit=3600 0 2
EOF

mkdir -p /srv/docker/dockerd
mkdir -p /srv/docker/containerd

systemctl daemon-reload

systemctl restart docker.socket
```

> **Warning** now we use manager for docker containers

```
doas pacman -Syu ducker lazydocker
```

## NOTES

### About IDE and editors

Unfortunatelly CODE-OSS is now almost a standart, so we only focus on it!

* ONLINE WEB CODE https://github.dev/github/dev
* VSCODE https://archlinux.org/packages/?name=code
* VSCODIUM https://aur.archlinux.org/packages/vscodium/

The first one does not require further isntall or preparation, but, 
the first one it requires serval space in disk apart of take too much RAM, 
becouse it runs inside the web browser overloading the web browser funtions.

The last one is the OSS and the real FOSS ide, you must isntall to support 
all the great effort of your community, unfortunatelly M$ shitty company 
limits and cutoff any possible collaboration, if you dont belive it, just 
check this https://github.com/VSCodium/vscodium/issues/2300 !

