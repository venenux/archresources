# arch install Guide

**Notes on installing VenenuX Arch Linux on AMD64 UEFI systems**

## Support and Feedback

**Issues**: https://codeberg.org/venenux/archresources/issues/new or also 
https://t.me/opentechnologies or https://t.me/latam_programadores or also 
if you want direct communication join to https://t.me/+W94Jx4A4AzdkMmYx

## Step 1 - Booting Preinstallation medium

1. **Download Arch Linux** from downloads page https://archlinux.org/download/ 
   but **if you already has a running linux then skip and goto Step 02** below.

2. **Then find http downloadas and the archive** named 
   like `archlinux-2026.MM.DD-x86_64.iso` or `archlinux-x86_64.iso`  
   or any other file ended with `.iso` suffix.

3. **Use USB or CD/DVD image** medium as USB stick or as CD/DVD rom
   and then **Boot live medium** or into **Arch Linux live medium**

## Step 2 - Preconfiguring installer

4. **Software Requeriments** in the preinstalation running linux/live medium:

   * arch-install-script >= 28 https://gitlab.archlinux.org/archlinux/arch-install-scripts/-/releases
   * coreutils >= v8.15 https://www.gnu.org/software/coreutils/#download
   * util-linux >= 2.39 https://www.kernel.org/pub/linux/utils/util-linux/
   * gawk or awk https://ftp.gnu.org/gnu/gawk/
   * bash >= 4.1 http://tiswww.case.edu/php/chet/bash/bashtop.html#Availability
   * dosfstools (EFI mounting partition) https://github.com/dosfstools/dosfstools/releases
   * pacman-package-manager > 5 https://gitlab.archlinux.org/pacman/pacman/-/releases
   * internet connection working and running

5. **Pacman setup bootstrap**, run as root (only if you boot non Arch live medium):

```
pacman-key --init

pacman-key --populate archlinux

cat > /etc/pacman.conf << EOF
[options]
HoldPkg = pacman glibc
Architecture = x86_64
SigLevel = Never
LocalFileSigLevel = Never

[core]
Include = /etc/pacman.d/mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist
EOF

mkdir -p /etc/pacman.d

cat > /etc/pacman.d/mirrorlist << EOF
Server = http://mirror.csclub.uwaterloo.ca/archlinux/\$repo/os/\$arch
Server = http://mirrors.sonic.net/archlinux/\$repo/os/\$arch
EOF

pacman -Syu
```

## Step 3 - Disk partitioning

6. **Partitioning** the main storage device using `fdisk` utility. 
   You can find storage device name using `lsblk -o NAME,MODEL,SIZE` command.
   Here the `/dev/nvme0n1` disk will be used and there are the process:

```
$ fdisk /dev/nvme0n1
                [this command will erase all partition to clean the disk]
Command (m for help): g

                [create partition 1: efi]
Command (m for help): n
Partition number (1-128, default 1): Enter ↵
First sector (..., default 2048): Enter ↵
Last sector ...: +512M

                [create partition 2: main]
Command (m for help): n
Partition number (2-128, default 2): Enter ↵
First sector (..., default ...): Enter ↵
Last sector ...: +128G The root directory of OS

                [create partition 3: swap]
Command (m for help): n
Partition number (3-128, default 3): Enter ↵
First sector (..., default ...): Enter ↵
Last sector ...: +8G At leas double your ram or minimal 8G swap;

                [create partition 4: home]
Command (m for help): n
Partition number (4-128, default 4): Enter ↵
First sector (..., default ...): Enter ↵
Last sector ...: Enter ↵

                [change partition types]
Command (m for help): t
Partition number (1-4, default 1): 1
Partion type or alias (type L to list all): 1
Command (m for help): t
Partition number (1-4, default 2): 2
Partion type or alias (type L to list all): 20
Command (m for help): t
Partition number (1-4, default 3): 3
Partion type or alias (type L to list all): 19
Command (m for help): t
Partition number (1-4, default 4): 4
Partion type or alias (type L to list all): 20

                [write partitioning to disk]
Command (m for help): w
```

> **Warning**: The block show you the process .. commands are just the single letters, output is show to you as better oriented process.. this is a step that cannot be automated, unless you use `parted`

> **Warning**: Partition 4 will use the rest of the disk; it is **recommended** that you do not use all of it but **leave at least 8 megabytes free at the end** of the disk.

## Step 4 - Create filesystems and mount target

7. **Format partitions**, run as root user those commands:

```
mkfs.fat -F 32 -S 2048 -n EFI /dev/nvme0n1p1

mkfs.ext4 -L ROOTAR /dev/nvme0n1p2

mkfs.ext4 -L HOMEAR /dev/nvme0n1p4

mkswap -L SWAPAR /dev/nvme0n1p3
```

8. **Mount filesystems** to the `/mnt/lx` in running linux as root user:

```
mkdir -p /mnt/lx

mount -t ext4 -o rw,relatime,exec,dev,suid,barrier=0,errors=remount-ro,commit=900,nofail /dev/nvme0n1p2 /mnt/lx

mkdir -p /mnt/lx/boot/efi /mnt/lx/home

mount -t vfat -o rw,umask=0077,iocharset=ascii,shortname=mixed,nofail /dev/nvme0n1p1 /mnt/lx/boot/efi

mount -t ext4 -o rw,relatime,exec,nodev,nosuid,barrier=0,errors=remount-ro,commit=1800,nofail /dev/nvme0n1p4 /mnt/lx/home

swapon -o pri=1,nofail /dev/nvme0n1p3
```

## Step 5 - Installing the base system with pacstrap

9. **pacstrap and chroot system** on mounted filesystem, tke care becouse 
   **untill Step 10 all the commands will be made into the chroot**, but 
   the command will put in prompt **the word `CHROOT`, if you dont see it** 
   that word, **must again do the `chroot` command**, so run as root then:

```
pacstrap -i /mnt/lx base nano iptables

genfstab -U -p /mnt/lx > /mnt/lx/etc/fstab

arch-chroot /mnt/lx /bin/bash --login -c "export PS1='[CHROOT] \w\$'; /bin/bash"
```

1.   **Setup base system** on the new future system, setup locales, 
   setup clock, setup timezone, configure hostname, run inside chroot:

```
pacman -Syu --noconfirm linux-lts linux-lts-headers linux-firmware mkinitcpio \
 wget less man-db iptables gawk perl psmisc arch-install-scripts

pacman -Syu --noconfirm sed

sed -i 's|#en_US.UTF-8|en_US.UTF-8|g' /etc/locale.gen

locale-gen

echo "LANG=en_US.UTF-8" > /etc/locale.conf

ln -sf /usr/share/zoneinfo/America/Panama /etc/localtime

hwclock --systohc

echo venenux > /etc/hostname

cat > /etc/hosts << EOF
127.0.0.1 localhost
::1       localhost
127.0.1.1 venenux
EOF

hostnamectl set-hostname venenux
```

11. **Userspace and privilegie escalation** so add new users 
   and setup privilegies by run inside chroot:

```
pacman -Syu --noconfirm bash

useradd -m -G audio,video,disk,games,users,network,input,optical -s /bin/bash general

echo "root:newpass" | chpasswd

echo "general:newpass" | chpasswd

pacman -Syu --noconfirm doas

cat > /etc/doas.conf << EOF
permit nopass root
permit :wheel
permit general
EOF
```

## Step 6 - Base system software userspace and networking

12. **Networking, coreutils, archivers, editors** to install inside chroot:

```
pacman -Syu --noconfirm dhcpcd networkmanager systemd-resolvconf openssh libfido2

systemctl enable NetworkManager systemd-resolved sshd

timedatectl set-ntp true

pacman -Syu --noconfirm mc nano cabextract gawk coreutils elfutils \
 arj binutils bzip2 cpio gzip lhasa lrzip lz4 7zip tar zip \
 xz zstd unarj unzip unrar libunrar rarcrack atool file \
 exfatprogs dosfstools f2fs-tools e2fsprogs ntfs-3g \
 erofs-utils squashfs-tools udftools xfsprogs
```

13. **wordlist and dictionary** support on system

```
pacman -Syu --noconfirm aspell hunspell ispell enchant gspell \
 aspell-en aspell-de aspell-it aspell-es aspell-ru \
 hunspell-en_us hunspell-de hunspell-es_any hunspell-ru \
 hyphen-en hyphen-de hyphen-es mythes-en mythes-de mythes-es
```

## Step 7 - Extents software package : AUR and makepkg

14. **Configure AUR and makepkg.conf** Pacman handles pre-built packages
AUR packages **needs an AUR helper (yay)** and must be build on install!

```
su general

doas pacman -Syu base-devel git go

mkdir -p /home/general/Devel && cd /home/general/Devel

cd ~/Devel && git clone https://aur.archlinux.org/yay.git

cd yay && makepkg && rm yay-debug*zst

doas pacman -U --noconfirm ~/Devel/yay/yay*.zst

yay -Syu doas-sudo-shim

yay -R sudo

doas mkdir -p /usr/src/makepkg

doas pacman -Syu base-devel git go

doas chown general:wheel /usr/src/makepkg

doas chmod 775 /usr/src/makepkg

GARCH=$(gcc -march=native -Q --help=target | grep -- '-march=' | cut -f3 | head -n1)

GTUNE=$(gcc -mtune=native -Q --help=target | grep -- '-mtune=' | cut -f3 | head -n1)

doas sed -i "s|-march=[^[:space:]]*|-march=$GARCH|" /etc/makepkg.conf

doas sed -i "s|-mtune=[^[:space:]]*|-mtune=$GTUNE|" /etc/makepkg.conf

doas sed -i s|.*PACMAN_AUTH=.*|PACMAN_AUTH=(doas)| /etc/makepkg.conf
```

## Step 8 - Extents hardware support : RAM usage and more devices

15. **Extend wireless support** bluetooyh support and improve wireless:

```
su general

yay -Syu --noconfirm rkhunter dkms ncurses procps-ng \
 wpa_supplicant net-tools netdiscover nmap vulscan netdiscover \
 lshw acpi htop tree socat lsof unhide psmisc usb_modeswitch alsa-utils \
 libgudev udisks2 bluez bluez-utils blueman arandr upower sane

doas systemctl enable bluetooth

exit
```

16. **Log2ram to save life of SSD** or improve disk speed using ram to 
   write logs files and not into disk!

```
su general

yay -Syu --noconfirm log2ram 

doas sed -i 's/^SIZE=.*/SIZE=1024M/' /etc/log2ram.conf

doas journalctl --vacuum-size=16M

doas sed -i -E 's/^[# ]*SystemMaxUse=.*/SystemMaxUse=64M/' /etc/systemd/journald.conf

doas systemctl disable log2ram-daily.timer

doas systemctl enable log2ram
```

17. **Improve CPU performance** by scaling frecuency processor but 
    of course will drain more battery, to change a more conservative 
    just check `cpupower frequency-info --governors` and change it:

```
su general

yay -Syu --noconfirm cpupower

doas echo passive > /sys/devices/system/cpu/intel_pstate/status

doas sed -i s/.*GOVERNOR=.*/GOVERNOR=performance/ /etc/default/cpupower-service.conf

doas sed -i s/.*PERF_BIAS=.*/PERF_BIAS=4/ /etc/default/cpupower-service.conf

doas sed -i s/.*EPP=.*/EPP=balance_performance/ /etc/default/cpupower-service.conf

doas systemctl enable cpupower

doas systemctl start cpupower

doas cpupower -c all set -b 4 -e performance -t 1

doas systemctl stop power-profiles-daemon.service

doas systemctl mask power-profiles-daemon.service
```

18. **Enable printing support**: also optionally install hp plugins 
   for firmware updates by using `doas hpsetup -i`

```
doas pacman -S --noconfirm cups cups-filters cups-pdf system-config-printer

doas systemctl enable cups.service

doas pacman -S --noconfirm foomatic-db-gutenprint-ppds \
 foomatic-db-engine foomatic-db-nonfree-ppds foomatic-db-nonfree \
 hplip gutenprint

doas cups-genppdupdate
```

19. **Use avahi services to auto discover new devices**, you 
   can browse manually by run `avahi-browse --all`:

```
doas pacman -S --noconfirm avahi nss-mdns

doas sed -i s/.*disallow-other-stacks.*/disallow-other-stacks=yes/ /etc/avahi/avahi-daemon.conf

doas systemctl enable avahi-daemon.service

doas systemctl start avahi-daemon.service
```

20. **[Optional] Improve laptop battery usage with TLP** - 
   only use this if you are on laptop, or will decrease performance..

```
doas pacman -S --noconfirm tlp tlp-rdw

doas systemctl enable tlp

doas systemctl enable NetworkManager-dispatcher.service

doas systemctl mask systemd-rfkill.service

doas systemctl mask systemd-rfkill.socket
```

## Step 9 - Finish installation and enable booting

21. **Refine boot initrd and hibernation support** using the first partition 
   only, detect first SWAP partition and then.. all of this is outside 
   of secure boot, so disable it! No encripted fs neither raid setups!

```
doas su

pacman -Syu --noconfirm efibootmgr grub os-prober

SWAPID=$(grep swap /etc/fstab | sed 's/   */:/g' | cut -d: -f1 | cut -f1 | cut -d' ' -f1 | grep -v "#" | head -n1)

ROOTID=$(grep -E '/ +|\t/ +' /etc/fstab | sed 's/   */:/g' | cut -d: -f1 | cut -f1 | cut -d' ' -f1 | grep -v "#" | head -n1)

CMDLIN="loglevel=3 efi=runtime elevator=noop vsyscall=emulated iommu=on acpi_enforce_resources=lax iomem=relaxed resume=$SWAPID"

sed -i s/^HOOKS.*/"HOOKS=(base udev systemd keyboard autodetect microcode resume modconf kms block filesystems)"/ /etc/mkinitcpio.conf

mkinitcpio -P

sed -i s/GRUB_CMDLINE_LINUX_DEFAULT.*/"GRUB_CMDLINE_LINUX_DEFAULT=\"$CMDLIN\""/ /etc/default/grub

sed -i s/GRUB_SAVEDEFAULT.*/"GRUB_SAVEDEFAULT=false"/ /etc/default/grub

sed -i s/GRUB_GFXPAYLOAD_LINUX.*/"GRUB_GFXPAYLOAD_LINUX=keep"/ /etc/default/grub

sed -i s/.*GRUB_DISABLE_LINUX_UUID.*/"GRUB_DISABLE_LINUX_UUID=false"/ /etc/default/grub

sed -i s/.*GRUB_DISABLE_OS_PROBER.*/"GRUB_DISABLE_OS_PROBER=false"/ /etc/default/grub

grub-install --target=x86_64-efi --efi-directory=/boot/efi/ --bootloader-id=ArchLinux --removable

grub-mkconfig -o /boot/grub/grub.cfg
```

22. **Exit the chroot installation  and restart**, take note that 
   all the commands were made into the chroot and be carefully becouse
   all the time **the word CHROOT must appears always, if you dont see it** 
   then, **must again do the `chroot` command and repeat all**, so 
   exits and reboot as root run:


```
exit

umount /mnt/lx/boot/efi

umount /mnt/lx/home

umount /mnt/lx

reboot
```

# Follows

* Install some king of desktop:
   * Wayland + Labwc Windowmanager as desktop : [arch-desktop-labwc.md](arch-desktop-labwc.md)
   * Xorg + Openbox Windowmanager as desktop : [arch-desktop-openbox.md](arch-desktop-openbox.md)
   * Wayland + Omarchi ShellDesktop as desktop : [arch-desktop-omarchi.md](arch-desktop-omarchi.md)
* Install Qemu and manage virtuals : [arch-emulation.md](arch-emulation.md)
* Install IDE and development base environtment : [arch-devel.md](arch-devel.md)

# LICENSE

CC-BY-SA-NC
