# arch install Guide

## Getting Started

**Notes on installing VenenuX Arch Linux on AMD64 UEFI systems**

#### Support and Feedback

**Issues**: If you come across any problems or have specific questions, please open 
an issue on the Codeberg repository. More direct, you can post comments on Telegram 
https://t.me/opentechnologies or https://t.me/latam_programadores or also 
if you want direct communication join to https://t.me/+W94Jx4A4AzdkMmYx

## Phase 0 - Preinstallation medium

* Use a live disk, but if you already has a running linux skip to Phase 01 section

#### Downloading Arch Linux live CD/DVD image

1. Go to Arch Linux downloads page https://archlinux.org/download/ but if 
   **you already has a running linux then skip and go to the Phase 01** below.

2. Find a download type, HTTP direct downloads are prefered for command line!

3. On the mirror page find archive named like `archlinux-2026.MM.DD-x86_64.iso` 
   or `archlinux-x86_64.iso`  or any other file ended with `.iso` suffix.

#### Preparing live linux installation medium

* If you already has a running linux skip to Phase 01 section to install.
   If you use qemu just put/set the iso file into the cdrom slot.

* If you use USB stick, download https://github.com/balena-io/etcher/releases 
   and record the ISO downloaded image into the USB stick or use `dd`.

* If you use CD/DVD rom, put the black disc into the optical device of your 
   computer device, and then open the burn software to record the image on!

#### Boot into Arch Linux live medium

* If you already has a running linux skip to Phase 01 section to install, 
   If you use qemu just put/set the iso file into the cdrom slot.

* Insert the installation medium/usb/disk into the computer on which you wants
   to install Arch Linux (or qemu settings confguration).

* Power on your PC and press _boot menu key_. For most original vendor based, 
   this key is `F12`. For clones computers this could be `ESC` or `F1` or `Del`.

* Boot from CD/DVD/USB and wait until boot process is finished.
   take note **Secure boot** option need to be turned off in your BIOS.

## Phase 01: Installation setup

We will use boostrap procedure

#### Step 01: Preconfiguring the live or running linux

* **1**. In **your running linux or live medium** must have installed those packages:

   * arch-install-script >= 28 https://gitlab.archlinux.org/archlinux/arch-install-scripts/-/releases
   * coreutils >= v8.15 https://www.gnu.org/software/coreutils/#download
   * util-linux >= 2.39 https://www.kernel.org/pub/linux/utils/util-linux/
   * gawk or awk https://ftp.gnu.org/gnu/gawk/
   * bash >= 4.1 http://tiswww.case.edu/php/chet/bash/bashtop.html#Availability
   * dosfstools (EFI mounting partition) https://github.com/dosfstools/dosfstools/releases
   * pacman-package-manager > 5 https://gitlab.archlinux.org/pacman/pacman/-/releases

* **2**. Minimal Pacman configurations for base bootstrap, run as root user:

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
```

* **3** Synchronize pacman packages, run command as root user:

```
pacman -Syu
```

> **Warning**: you must already have internet connection configued!

#### Step 02: Disk partitioning

* **5** Partition the main storage device using `fdisk` utility. You can find 
storage device name using `lsblk -o NAME,MODEL,SIZE` command. Run as root user:

<dl><dd>
<pre>
$ <b>fdisk /dev/nvme0n1</b>
                <i>[this command will erase all partition to clean the disk]</i>
Command (m for help): <b>g</b>
<span />
                <i>[create partition 1: efi]</i>
Command (m for help): <b>n</b>
Partition number (1-128, default 1): <b>Enter &crarr;</b>
First sector (..., default 2048): <b>Enter &crarr;</b>
Last sector ...: <b>+512M</b>
<span />
                <i>[create partition 2: main]</i>
Command (m for help): <b>n</b>
Partition number (2-128, default 2): <b>Enter &crarr;</b>
First sector (..., default ...): <b>Enter &crarr;</b>
Last sector ...: <b>+128G</b> <i>The root directory of OS</i>
<span />
                <i>[create partition 3: swap]</i>
Command (m for help): <b>n</b>
Partition number (3-128, default 3): <b>Enter &crarr;</b>
First sector (..., default ...): <b>Enter &crarr;</b>
Last sector ...: <b>+8G</b> <b>At leas double your ram or minimal 8G swap;</b>
<span />
                <i>[create partition 4: home]</i>
Command (m for help): <b>n</b>
Partition number (4-128, default 4): <b>Enter &crarr;</b>
First sector (..., default ...): <b>Enter &crarr;</b>
Last sector ...: <b>Enter &crarr;</b>
<span />
                <i>[change partition types]</i>
Command (m for help): <b>t</b>
Partition number (1-4, default 1): <b>1</b>
Partion type or alias (type L to list all): <b>1</b>
Command (m for help): <b>t</b>
Partition number (1-4, default 2): <b>2</b>
Partion type or alias (type L to list all): <b>20</b>
Command (m for help): <b>t</b>
Partition number (1-4, default 3): <b>3</b>
Partion type or alias (type L to list all): <b>19</b>
Command (m for help): <b>t</b>
Partition number (1-4, default 4): <b>4</b>
Partion type or alias (type L to list all): <b>20</b>
<span />
                <i>[write partitioning to disk]</i>
Command (m for help): <b>w</b>
</pre>
</dd></dl>

> **Warning**: Partition 4 will use the rest of the disk; it is **recommended** that you do not use all of it but **leave at least 8 megabytes free at the end** of the disk.

### Step 03: Create filesystem on partitions

* **6** Create filesystems on disk partitions, run as root user those commands:

```
mkfs.fat -F 32 -S 2048 -n EFI /dev/nvme0n1p1

mkfs.ext4 -L ROOTAR /dev/nvme0n1p2

tune2fs -O "^encrypt" /dev/nvme0n1p2

tune2fs -O "^quota" /dev/nvme0n1p2

mkswap -L SWAPAR /dev/nvme0n1p3

mkfs.ext4 -L HOMEAR /dev/nvme0n1p4

tune2fs -O "^encrypt" /dev/nvme0n1p4

tune2fs -O "^quota" /dev/nvme0n1p4
```

### Step 04: Mounting partitions to target installation

* **7** Mount all filesystems to the `/mnt` in running linux as root user:

```
mkdir -p /mnt

mount -t ext4 -o rw,relatime,exec,dev,suid,barrier=0,errors=remount-ro,commit=900,nofail /dev/nvme0n1p2 /mnt

mkdir -p /mnt/boot/efi /mnt/home

mount -t vfat -o rw,umask=0077,iocharset=ascii,shortname=mixed,nofail /dev/nvme0n1p1 /mnt/boot/efi

mount -t ext4 -o rw,relatime,exec,nodev,nosuid,barrier=0,errors=remount-ro,commit=1800,nofail /dev/nvme0n1p4 /mnt/home

swapon -o pri=1,nofail /dev/nvme0n1p3
```

### Step 05: Installing the base system

* **8** Create base system into new filesystem and generate fstab, run as root:

```
pacstrap -i /mnt base nano iptables bash

genfstab -U -p /mnt > /mnt/etc/fstab
```

### Step 06: Basic configuration base system

* **9** Chroot into freshly created filesystem by run as root user this command:

```
arch-chroot /mnt
```

* **10** now you are **inside the new installed system linux, then run** those 
commands inside the "chrooted" environment:

```
pacman -Syu --noconfirm linux-lts linux-lts-headers linux-firmware mkinitcpio \
 base-devel git wget less networkmanager man-db iptables gawk perl psmisc \
 doas arch-install-scripts
```

* **11** Still inside "chrooted" setup locale, clock and timezone, run:

```
pacman -Syu --noconfirm sed

sed -i 's|#en_US.UTF-8|en_US.UTF-8|g' /etc/locale.gen

locale-gen

echo "LANG=en_US.UTF-8" > /etc/locale.conf

ln -sf /usr/share/zoneinfo/America/Panama /etc/localtime

hwclock --systohc
```

* **12** Still inside "chrooted" setup system hostname by run:

```
echo venenux > /etc/hostname

cat > /etc/hosts << EOF
127.0.0.1 localhost
::1       localhost
127.0.1.1 venenux
EOF

hostnamectl set-hostname venenux
```

### Step 07: Basic configuration elevation privilegies and users

* **13** Still inside "chrooted" add new users and setup passwords by run:

```
pacman -S --noconfirm bash

useradd -m -G audio,video,disk,games,users,network,input,optical -s /bin/bash general

echo "root:newpass" | chpasswd

echo "general:newpass" | chpasswd
```

* **14** Still inside "chrooted" setup elevation "doas" privilegies by run:

```
pacman -Syu --noconfirm doas

cat > /etc/doas.conf << EOF
permit nopass root
permit :wheel
permit general
EOF
```

## Phase 02: Booting configuration setup

We will only use grub and UEFI, for BIOS just dont use the EFI parameters!

### Step 08: Configuring boot of the new system

* **15** Still inside "chrooted" install network environment for next boot:

```
pacman -S --noconfirm dhcpcd networkmanager systemd-resolvconf openssh libfido2

systemctl enable NetworkManager systemd-resolved sshd
```

* **16** Still inside "chrooted" install and configure EFI and GRUB manager:

```
pacman -S --noconfirm efibootmgr grub os-prober

grub-install --target=x86_64-efi --efi-directory=/boot/efi/ --bootloader-id=Archit --removable

grub-mkconfig -o /boot/grub/grub.cfg
```

* **17** Still inside "chrooted", now you can exit chroot, unmount all and reboot:

```
exit

umount /mnt/boot/efi

umount /mnt/home

umount /mnt
```

## Phase 03: Configuring userspace initial system startup

The base system and boot are set, but now we alredy need/have minimal desktop

* **18** Chroot again into created filesystem by remount and run:

```
mkdir -p /mnt

mount -t ext4 -o rw,relatime,exec,dev,suid,barrier=0,errors=remount-ro,commit=900 /dev/nvme0n1p2 /mnt

mkdir -p /mnt/boot/efi /mnt/home

mount -t vfat -o rw,umask=0077,iocharset=ascii,shortname=mixed /dev/nvme0n1p1 /mnt/boot/efi

mount -t ext4 -o rw,relatime,exec,nodev,nosuid,barrier=0,errors=remount-ro,commit=1800 /dev/nvme0n1p4 /mnt/home

arch-chroot /mnt
```

### Step 09: Basic configuration utilities and userspace

* **19** Inside chroot or after first boot enable time syncronization by NTP:

```
timedatectl set-ntp true
```

* **20** Inside chroot or after first boot install base administrative utilities

```
pacman -S --noconfirm mc cabextract gawk 7zip zip coreutils nano nawk elfutils
```

* **21** Compresion and archiving utilities

```
pacman -Syu --noconfirm arj binutils bzip2 cpio gzip lhasa lrzip lz4 7zip tar \
 xz zstd unarj unzip unrar libunrar rarcrack atool file
```

* **22** Filesystems support for external storage devices

```
pacman -Syu --noconfirm exfatprogs dosfstools f2fs-tools e2fsprogs ntfs-3g \
 erofs-utils squashfs-tools udftools xfsprogs
```

* **23** If you are inside chroot, then umount and restart

```
exit

umount /mnt/boot/efi

umount /mnt/home

umount /mnt

reboot
```

### Step 10 enable the AUR repository management and makepkg

Pacman only handles updates for pre-built packages in its repositories. 
AUR packages are redistributed in form of PKGBUILDs and need an AUR helper 
to automate the re-build process. However, keep in mind that a rebuild of a 
package may be required when its shared library dependencies are updated, 
not only when the package itself is updated.

```
su general

doas pacman -S base-devel git go

mkdir -p /home/general/Devel && cd /home/general/Devel

cd ~/Devel && git clone https://aur.archlinux.org/yay.git

cd yay && makepkg && rm yay-debug*zst

doas pacman -U --noconfirm ~/Devel/yay/yay*.zst

yay -Syu doas-sudo-shim
```

Now lest configure the most important file of AUR packages: `/etc/makepkg.conf` 
lest create specific compilation directory (you can use separate disk then):

```

doas mkdir -p /usr/src/makepkg

doas pacman -Syu base-devel git go

doas chown general:wheel /usr/src/makepkg

doas chmod 775 /usr/src/makepkg
````

Becouse cannot be done using a command, so you must open the file for 
editing with `doas nano /etc/makepkg.conf` and **then sustitute contents**:

``` bash
#!/hint/bash
# shellcheck disable=2034

DLAGENTS=('file::/usr/bin/curl -qgC - -o %o %u'
          'ftp::/usr/bin/curl -qgfC - --ftp-pasv --retry 3 --retry-delay 3 -o %o %u'
          'http::/usr/bin/curl -qgb "" -fLC - --retry 3 --retry-delay 3 -o %o %u'
          'https::/usr/bin/curl -qgb "" -fLC - --retry 3 --retry-delay 3 -o %o %u'
          'rsync::/usr/bin/rsync --no-motd -z %u %o'
          'scp::/usr/bin/scp -C %u %o')
VCSCLIENTS=('bzr::breezy'
            'fossil::fossil'
            'git::git'
            'hg::mercurial'
            'svn::subversion')
CARCH="x86_64"
CHOST="x86_64-pc-linux-gnu"
CFLAGS="-march=generic -mtune=generic -O3 -pipe -fno-plt -fexceptions \
        -Wp,-D_FORTIFY_SOURCE=1 -Wno-error=format-security \
        -fstack-clash-protection -fcf-protection -Wno-incompatible-pointer-types \
        -fno-omit-frame-pointer -mno-omit-leaf-frame-pointer -g0 -fno-lto \
        -Wno-unused-but-set-variable -Wno-error=deprecated-declarations -Wall -Wextra"
CPPFLAGS="$CFLAGS"
CXXFLAGS="$CFLAGS -Wp,-D_GLIBCXX_ASSERTIONS"
LDFLAGS="-Wl,-O1 -Wl,--sort-common -Wl,--as-needed -Wl,-z,relro -Wl,-z,now -Wl,-z,pack-relative-relocs"
LTOFLAGS="-flto=auto"
MAKEFLAGS="-j4"
DEBUG_CFLAGS="-g0 -fno-lto"
DEBUG_CXXFLAGS="$DEBUG_CFLAGS"
BUILDENV=(!distcc color !ccache check !sign)
BUILDDIR=/usr/src/makepkg
OPTIONS=(strip docs !libtool !staticlibs emptydirs zipman purge !debug !lto)
INTEGRITY_CHECK=(sha256)
STRIP_BINARIES="--strip-all"
STRIP_SHARED="--strip-unneeded"
STRIP_STATIC="--strip-debug"
MAN_DIRS=({usr{,/local}{,/share},opt/*}/{man,info})
DOC_DIRS=(usr/{,local/}{,share/}{doc,gtk-doc} opt/*/{doc,gtk-doc})
PURGE_TARGETS=(usr/{,share}/info/dir .packlist *.pod)
DBGSRCDIR="/usr/src/debug"
LIB_DIRS=('lib:usr/lib' 'lib32:usr/lib32')
COMPRESSGZ=(gzip -c -f -n)
COMPRESSBZ2=(bzip2 -c -f)
COMPRESSXZ=(xz -c -z -)
COMPRESSZST=(zstd -c -T0 -)
COMPRESSLRZ=(lrzip -q)
COMPRESSLZO=(lzop -q)
COMPRESSZ=(compress -c -f)
COMPRESSLZ4=(lz4 -q)
COMPRESSLZ=(lzip -c -f)
PKGEXT='.pkg.tar.zst'
SRCEXT='.src.tar.gz'
PACMAN_AUTH=(doas)
```

> **Warning** save it using `CRTL+X` and then `Y`

The tuned part here you can change is `march` and `tune` change `generic` with 
the arch spcific cpu of your computer.

* **HOW TO DETERMINE `-march` BEST VALUE:

```
gcc -march=native -Q --help=target | grep -- '-march=' | cut -f3 | head -n1
```

* **HOW TO DETERMINE `-mtune` BEST VALUE:

```
gcc -mtune=native -Q --help=target | grep -- '-mtune=' | cut -f3 | head -n1
```

Take the values of "march" and "mtune"  and use it on the makepkg.conf file, 
by change the word "generic" with those values.

### Step 11: tuneup the hardware support

1. **Extend hardware support** by savin life of SSD with usage of log2ram, 
   management of network devices and linux underliying utilites

```
su general

yay -Syu --noconfirm wpa_supplicant os-prober grub ntfs-3g dkms \
 ncurses procps-ng net-tools netdiscover rkhunter nmap vulscan netdiscover \
 lshw acpi htop tree socat lsof unhide psmisc usb_modeswitch alsa-utils \
 libgudev udisks2 log2ram bluez bluez-utils blueman arandr upower sane

doas systemctl enable bluetooth

doas systemctl enable log2ram.service log2ram-daily.timer
```

1. **Improve performance** by scaling frecuency processor

```
yay -Syu cpupower

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

3. **Refine boot initrd and hibernation support** using the first partition 
   only, detect first SWAP partition and then.. all of this is outside 
   of secure boot, so disable it! No encripted fs neither raid setups!

```
doas su

grep swap /etc/fstab | sed 's/   */:/g' | cut -d: -f1 | cut -f1 | cut -d' ' -f1 | grep -v "#" | head -n1

sed -i s/HOOKS.*/"HOOKS=(base udev systemd resume keyboard autodetect microcode modconf kms block filesystems)"/ /etc/mkinitcpio.conf

mkinitcpio -P

cp -f /boot/*.img /boot/vmlinuz* /boot/efi/

PSWAP=$(grep swap /etc/fstab | sed 's/   */:/g' | cut -d: -f1 | cut -f1 | cut -d' ' -f1 | grep -v "#" | head -n1)

ROOTUID=$(grep -E '/ +|\t/ +' /etc/fstab | sed 's/   */:/g' | cut -d: -f1 | cut -f1 | cut -d' ' -f1 | grep -v "#" | head -n1)

CMDLINE="loglevel=3 efi=runtime elevator=noop vsyscall=emulated iommu=on acpi_enforce_resources=lax iomem=relaxed"

sed -i s/GRUB_CMDLINE_LINUX_DEFAULT.*/"GRUB_CMDLINE_LINUX_DEFAULT=\"$CMDLINE\""/ /etc/default/grub

sed -i s/GRUB_SAVEDEFAULT.*/"GRUB_SAVEDEFAULT=false"/ /etc/default/grub

sed -i s/GRUB_GFXPAYLOAD_LINUX.*/"GRUB_GFXPAYLOAD_LINUX=keep"/ /etc/default/grub

sed -i s/.*GRUB_DISABLE_LINUX_UUID.*/"GRUB_DISABLE_LINUX_UUID=false"/ /etc/default/grub

sed -i s/.*GRUB_DISABLE_OS_PROBER.*/"GRUB_DISABLE_OS_PROBER=false"/ /etc/default/grub

grub-install --target=x86_64-efi --efi-directory=/boot/efi/ --bootloader-id=ArchLinux --removable

grub-mkconfig -o /boot/grub/grub.cfg
```

2. **Enable printing support on your PC**: also optionally install hp plugins 
   for firmware updates by using `doas hpsetup -i`

```
doas pacman -S --noconfirm cups cups-filters cups-pdf system-config-printer

doas systemctl enable cups.service

doas pacman -S --noconfirm \
 foomatic-db-engine foomatic-db-nonfree-ppds foomatic-db-nonfree \
 hplip gutenprint foomatic-db-gutenprint-ppds

doas cups-genppdupdate
```

3. **[Optional] Improve laptop battery usage with TLP** - 
   only use this if you are on laptop, or will decrease performance..

```
doas pacman -S --noconfirm tlp tlp-rdw

doas systemctl enable tlp

doas systemctl enable NetworkManager-dispatcher.service

doas systemctl mask systemd-rfkill.service

doas systemctl mask systemd-rfkill.socket
```

4. **[Optional] use avahi services to auto discover devices**, you 
   can browse manually by run `avahi-browse --all`:

```
doas pacman -S --noconfirm avahi nss-mdns

doas sed -i s/.*disallow-other-stacks.*/disallow-other-stacks=yes/ /etc/avahi/avahi-daemon.conf

doas systemctl enable avahi-daemon.service

doas systemctl start avahi-daemon.service
```

## Section 03: Userspace basic desktop

1. Install common desktop utilities, console utilities

```
pacman -Syu --noconfirm --needed doas lesspipe perl xz ncurses nano \
 cabextract gawk tar 7zip unace unarj unrar unzip zip doas bash coreutils \
 dialog intel-ucode fuse3 lshw powertop acpi htop tree socat lsof unhide \
 wget rsync aria2 base-devel git less groff bash-completion unhide rkhunter \
 psmisc tcpdump mtr net-tools ethtool openbsd-netcat procps-ng gnome-nettool \
 dbus alsa-oss alsa-plugins alsa-ucm-conf sof-firmware alsa-firmware \
 pulseaudio pulsemixer pulseaudio-alsa gxmessage libcanberra \
 otf-atkinson-hyperlegible ttf-atkinson-hyperlegible otf-droid-nerd \
 libfm-extra libfm-gtk3 gtk3 gvfs pango cairo jgmenu perl-net-dbus xdg-utils \
 pcmanfm sakura lxappearance uget notification-daemon scrot gsimplecal xarchiver \
 parcellite perl-x11-protocol xorg-xprop xdotool xorg-xwininfo gprename \
 accountsservice hicolor-icon-theme openbox
```

2. Enable wordlist and dictionary support on system

```
pacman -S --noconfirm aspell hunspell nano \
 aspell-en aspell-de aspell-it aspell-es hunspell-es_any aspell-ru \
 hunspell-en_us hunspell-en_gb hunspell-es_ve hunspell-es_ar hunspell-ru \
 hyphen-en hyphen-de hyphen-fr hyphen-it hyphen-nl hunspell-es_mx hunspell-es_pa \
 mythes-en mythes-de mythes-fr mythes-it mythes-nl mythes-pl
```

For better desktop focused setup use: [arch-desktop-openbox.md](arch-desktop-openbox.md)




