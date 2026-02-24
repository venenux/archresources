# arch install Guide

## Getting Started

Welcome to the **VenenuX Arch Linux tutorials for AMD64 computers** because 
ArchLinux only are for AMD64 machines!!

It has been carefully created based on the very bad own exp of installation 
on multiple devices over the years. **It uses GRUB! not shitstemd-boot.**

Unfortunatelly for older devices we must recommend Haiku OS (2000 to 2012) or
then VenenuX (2010 to 2021) because Arch linux is only for some recent devices!
**This guide only works for most recent up to 2026** older ones must check the 
[2024](2024) or [2025](2025) or [2026](2026) or [archive](archive) subdirectories.

#### Support and Feedback

**Issues**: If you come across any problems or have specific questions, please open 
an issue on the Codeberg repository for this guide. Use Telegram OpenTech group 
if you want direct communication. https://t.me/+W94Jx4A4AzdkMmYx

**Pull Requests**: If you have improvements or additions to the guide, feel 
free to submit a pull request. Your contributions can help enhance the clarity 
of the guide for everyone. - https://codeberg.org/venenux/archresources/fork

## Phase 0 - Preinstallation medium

* This guide assume a linux OS running, if not use any linux live disk or VenenuX
* All those commmands are running under root administrator user only.
* If you already has a running linux skip to Phase 01 section to install

#### Downloading Arch Linux live CD/DVD image

1. Go to Arch Linux downloads page https://archlinux.org/download/ but if 
   you already has a running linux skip step 01 and go to the section 1 below.

2. Find **HTTP Direct Downloads** section and choose any download mirror.
   Select a mirror that is geographically closer to your location.

3. On the mirror page find archive named like `archlinux-2025.MM.DD-x86_64.iso` 
   or `archlinux-x86_64.iso`  or any other file with `.iso` suffix. Other files 
   (like _.txt_, _.tar.gz_ and even _.iso.sig_) are not needed for installation 
   process.

#### Preparing live linux installation medium

*. Insert a USB-stick into your PC with at least 2Gb of space available on it. 
    You can also use a blank cd/dvd/bd rom and burn the iso image, or load into 
    the qemu cderom slot!

* If you use USB stick, download https://github.com/balena-io/etcher/releases 
   and record the ISO downloaded image into the USB stick.

* If you use CD/DVD rom, put the black disc into the optical device of your 
   computer device, and then open the burn software to record the image on!

#### Boot into Arch Linux live medium

* Insert the installation medium into the computer on which you wants install 
   Arch Linux. If you use qemu just put the iso file into the cdrom slot.

* Power on your PC and press _boot menu key_. For most vendor based original, 
   this key is `F12`. For clones computers this could be `ESC` or `F1`.

* Boot from CD/DVD/USB and wait until boot process is finished.
   take note **Secure boot** option need to be turned off it in your BIOS.

## Phase 01: Installation setup

Arch linux comes with **the famous `arch-install` script, we will avoid it** 
because we will setup disk partition layout and advanced formating.

#### Step 01: Use the live medium or use the running linux

This also works for instalation inside another linux. How? easy, just using 
the another linux start from the disk partitioning step and that is all.

* **1**. In **your linux or live medium must have installed following packages**:

* arch-install-script >= 28 https://gitlab.archlinux.org/archlinux/arch-install-scripts/-/releases
* pacman-package-manager > 5 https://gitlab.archlinux.org/pacman/pacman/-/releases
* makepkg + archlinux-keyring https://gitlab.archlinux.org/pacman/pacman/-/releases
* coreutils >= v8.15 https://www.gnu.org/software/coreutils/#download
* util-linux >= 2.39 https://www.kernel.org/pub/linux/utils/util-linux/
* gawk or awk https://ftp.gnu.org/gnu/gawk/
* bash >= 4.1 http://tiswww.case.edu/php/chet/bash/bashtop.html#Availability
* dosfstools (EFI mounting partition) https://github.com/dosfstools/dosfstools/releases

* **2**. Then if you have this packages in any linux do the necesary configs 
(those are not necesary if you are running ArchLinux from live disc), so then 
run as root user following commands in the current running/live linux:

```
pacman-key --init

pacman-key --populate archlinux

cat > /etc/pacman.conf << EOF
[options]
HoldPkg = pacman glibc
Architecture = auto
CheckSpace
ParallelDownloads = 5
SigLevel = Never

[core]
Include = /etc/pacman.d/mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist
EOF

mkdir /etc/pacman.d/mirrorlist

cat > /etc/pacman.d/mirrorlist << EOF
Server = http://mirror.csclub.uwaterloo.ca/archlinux/\$repo/os/\$arch
EOF
```

#### Step 02: networking and sync packages

* **3**  Arch linux need internet connection, to sync package originis, 
Also skip to next if your linux must be configured in another way! so 
Connect to Network, **if you have wired network, is automatically**, 
for WiFi just using `iwctl` and check connection is established as root user:

```
iwctl

station wlan0 get-networks

station wlan0 connect

exit

ping 1.1.1.1
```

* **4** Synchronize pacman packaes by runing the following command as root user:

```
pacman -Syu
```


#### Step 03: Disk partitioning

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
Last sector ...: <b>+64G</b> <i>AT least double your RAM size</i>
<span />
                <i>[create partition 3: swap]</i>
Command (m for help): <b>n</b>
Partition number (3-128, default 3): <b>Enter &crarr;</b>
First sector (..., default ...): <b>Enter &crarr;</b>
Last sector ...: <b>+8G</b> <b>Enter &crarr;</b>
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

### Step 04: Create filesystem and mounting partitions

* **6** Create filesystems on disk partitions, run as root user those commands:

```
mkfs.fat -F 32 -S 1024 -n EFI /dev/nvme0n1p1

mkfs.ext4 -b 1024 -L ROOTAR /dev/nvme0n1p2

tune2fs -O "^encrypt" /dev/nvme0n1p2

tune2fs -O "^quota" /dev/nvme0n1p2

mkswap -L SWAPAR /dev/nvme0n1p3

mkfs.ext4 -b 1024 -L HOMEAR /dev/nvme0n1p4

tune2fs -O "^encrypt" /dev/nvme0n1p4

tune2fs -O "^quota" /dev/nvme0n1p4
```

* **7** Mount all filesystems to the `/mnt` in running linux as root user:

```
mkdir -p /mnt

mount -t ext4 -o rw,relatime,exec,dev,suid,barrier=0,errors=remount-ro,commit=1800 /dev/nvme0n1p2 /mnt

mkdir -p /mnt/boot/efi /mnt/home

mount -t vfat -o rw,umask=0077,iocharset=ascii,shortname=mixed /dev/nvme0n1p1 /mnt/boot/efi

mount -t ext4 -o rw,noatime,exec,nodev,nosuid,barrier=0,errors=remount-ro,commit=1800 /dev/nvme0n1p4 /mnt/home

swapon /dev/nvme0n1p3
```

### Step 05: Installing the base system

* **8** Create base system into new filesystem and generate fstab, run as root:

```
pacstrap -i /mnt base nano iptables

genfstab -U -p /mnt > /mnt/etc/fstab
```

### Step 06: Basic configuration base system

* **9** Chroot into freshly created filesystem by run as root user this command:

```
arch-chroot /mnt
```

* **10** The console terminal will look similar or equal BUT IS NOT THE SAME, 
cos now you are inside the new installed system linux, the process allows to 
use the new filesystem of the new linux but limited, so then run those commands 
inside the "chrooted" environment:

```
pacman -Syu --noconfirm linux-lts linux-lts-headers linux-firmware mkinitcpio \
 base-devel git wget less networkmanager man-db iptables gawk perl psmisc \
 doas arch-install-scripts
```

* **11** Still inside "chrooted" setup locale, clock and timezone, run those commands:

```
pacman -Syu --noconfirm sed

sed -i 's|#en_US.UTF-8|en_US.UTF-8|g' /etc/locale.gen

locale-gen

echo "LANG=en_US.UTF-8" > /etc/locale.conf

ln -sf /usr/share/zoneinfo/America/Panama /etc/localtime

hwclock --systohc
```

* **12** Still inside "chrooted" setup system hostname by run those commands:

```
echo archisshit > /etc/hostname

cat > /etc/hosts << EOF
127.0.0.1 localhost
::1       localhost
127.0.1.1 archisshit
EOF

hostnamectl set-hostname archisshit
```

### Step 07: Basic configuration elevation privilegies and users

* **13** Still inside "chrooted" add new users and setup passwords by run:

```
pacman -S --noconfirm bash

useradd -m -G audio,video,disk,games,users,network,input,optical -s /bin/bash general

passwd root

passwd general
```

* **14** Still inside "chrooted" setup elevation "doas" privilegies by run:

```
pacman -Syu --noconfirm doas

cat > /etc/doas.conf << EOF
permit nopass root
permit nopass :wheel
permit general
EOF
```

## Phase 02: Booting configuration setup

We will only use grub and UEFI, for BIOS just don use the EFI parameters!

### Step 08: Configuring boot of the new system

* **15** Still inside "chrooted" install network minimal environment for next boot:

```
pacman -S --noconfirm dhcpcd networkmanager systemd-resolvconf openssh libfido2

systemctl enable NetworkManager systemd-resolved sshd
```

* **16** Still inside "chrooted" install and configure EFI using GRUB manager:

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

reboot
```

## Phase 03: Configuring userspace initial system startup

The base system and boot are set, but now we alredy need/have this:

* Already pass and performs all of the section 1 above!
* Reboot and start in your new disk with Arch Linux installation.
* We assumed the networking setup from previous sectioin!
* Install the `opendoas` package with `pacman -S opendoas doas`
* login with the user `general` we configured previously

### Step 09: Basic configuration of userspace

* **18** Activate time syncronization using NTP:

```
timedatectl set-ntp true
```

* **19** Install minimal administrative packages

```
pacman -S --noconfirm mc cabextract gawk 7zip zip doas coreutils nano nawk elfutils
```

* **20** Install common desktop utilities, console utilities

```
pacman -Syu --noconfirm --needed doas lesspipe perl xz ncurses nano \
 cabextract gawk tar 7zip unace unarj unrar unzip zip doas bash coreutils \
 dialog intel-ucode fuse3 lshw powertop acpi htop tree socat lsof unhide \
 wget rsync aria2 base-devel git less groff bash-completion unhide rkhunter \
 dbus psmisc tcpdump mtr net-tools ethtool openbsd-netcat procps-ng \
 alsa-oss alsa-plugins alsa-ucm-conf sof-firmware alsa-firmware \
 pulseaudio pulsemixer pulseaudio-alsa gxmessage libcanberra \
 libfm-extra libfm-gtk3 gtk3 gvfs pango cairo jgmenu perl-net-dbus xdg-utils \
 pcmanfm sakura lxappearance uget notification-daemon scrot gsimplecal xarchiver \
 parcellite perl-x11-protocol xorg-xprop xdotool xorg-xwininfo accountsservice
```

* **21** Compresion and archiving utilities

```
pacman -Syu --noconfirm arj binutils bzip2 cpio gzip lhasa lrzip lz4 7zip tar \
 xz zstd unarj unzip unrar libunrar rarcrack \
 atool file
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

cd ~/Devel && git clone https://aur.archlinux.org/doas-sudo-shim-k.git

cd doas-sudo-shim-k && makepkg && rm -f doas-debug*zst

doas pacman -R --noconfirm sudo base-devel

doas pacman -U --noconfirm ~/Devel/doas-sudo-shim-k/doas-sudo-shim-k*.zst

doas mkdir -p /usr/src/makepkg

doas pacman -Syu base-devel git go

doas chown general:wheel /usr/src/makepkg

doas chmod 775 /usr/src/makepkg
```

Now lest configure themost important file of AUR packages, the `/etc/makepkg.conf`
This cannot be done using a command, so you must open the file for editing:
`doas nano /etc/makepkg.conf`  and then sustitute all the contents with this:

``` bash
#!/hint/bash
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
```

The tuned part here you cna change is `march` and `tune` change `generic` with 
the arch spcific cpu of your computer.

* **HOW TO DETERMINE `-march` BEST VALUE:

```
gcc -march=native -Q --help=target | grep -- '-march=' | cut -f3 | head -n1
```

* **HOW TO DETERMINE `-mtune` BEST VALUE:

```
gcc -mtune=native -Q --help=target | grep -- '-mtune=' | cut -f3 | head -n1
```

### Step 11: tuneup the hardware support

> **WARNING**: this section just improve the hardware support for better desktop experience

1. **Extend hardware support**

```
su general

yay -Syu --noconfirm wpa_supplicant os-prober grub ntfs-3g dkms \
 psmisc ncurses procps-ng net-tools rkhunter unhide alsa-utils \
 lshw acpi htop tree socat lsof unhide psmisc usb_modeswitch \
 libgudev udisks2 log2ram bluez bluez-utils blueman arandr upower

doas systemctl enable bluetooth

doas systemctl enable log2ram.service log2ram-daily.timer
```

1. **Refine boot initrd and hibernation support** We wil use the first partition 
   only, if another one, must be configured manually, those steps only setup 
   automatically the first one! We will detect SWAP partition and then..
   Open GRUB configuration file and add resume UUID to the line boot, 
   all of those commands must be run as root and not using `sudo` or `doas` 
   becouse the variable expansion will not be parsed to `GRUB_CMDLINE_LINUX_DEFAULT`
   so then you can hibernate your system using: `systemctl hibernate`.
   Be carefully, all of this is outside of secure boot, so disable it!

> Warning: &#128161; Those hooks dont assume encripted filesystems, neither raid setups!

> Warning: &#128161; all of those commands cannot be run using `sudo` or `doas`

```
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
   for firmware updates b using `doas hpsetup -i`

```
doas pacman -S --noconfirm cups cups-filters cups-pdf system-config-printer

doas systemctl enable cups.service

doas pacman -S --noconfirm \
 foomatic-db-engine foomatic-db-nonfree-ppds foomatic-db-nonfree \
 hplip gutenprint foomatic-db-gutenprint-ppds

doas cups-genppdupdate
```

3. **[Optional] Improve laptop battery usage with TLP** - 
   this utility that basically does kernel settings tweaking but over desktops 
   machines will decrease performance.. **only use if you are on laptop** 

```
doas pacman -S --noconfirm tlp tlp-rdw

doas systemctl enable tlp

doas systemctl enable NetworkManager-dispatcher.service

doas systemctl mask systemd-rfkill.service

doas systemctl mask systemd-rfkill.socket
```

4. **[Optional] use avahi services to auto discover devices**, you can browse 
   manually by run `avahi-browse --all` aftyer avahi setup and install:

```
doas pacman -S --noconfirm avahi nss-mdns

doas sed -i s/.*disallow-other-stacks.*/disallow-other-stacks=yes/ /etc/avahi/avahi-daemon.conf

doas systemctl enable avahi-daemon.service

doas systemctl start avahi-daemon.service
```

## Section 03: Userspace desktop setup

1. Enable wordlist and dictionary support on system

```
pacman -S --noconfirm aspell hunspell nano \
 aspell-en aspell-de aspell-it aspell-es hunspell-es_any aspell-ru \
 hunspell-en_us hunspell-en_gb hunspell-es_ve hunspell-es_ar hunspell-ru \
 hyphen-en hyphen-de hyphen-fr hyphen-it hyphen-nl hunspell-es_mx hunspell-es_pa \
 mythes-en mythes-de mythes-fr mythes-it mythes-nl mythes-pl
 
```


Se next tutorial: [arch-desktop-openbox.md](arch-desktop-openbox.md)

## Warning and advertices

> **Warning**: we will use GPT table, otherwise use "o" instead of "g".

> **Warning**: we will use `/dev/nvme0n1` as diskdrive, could be `/dev/sda`.

> **Warning**: format with 1k sector sizes its for performance of small files

> **Warning**: we use `wlan0` for example, but check with `ip add` command.

> Warning: &#128161; this will use the running linux as fake new linux install, 
it means that mounted filesystems in the running linux, will be used indirectly 
as a new running linux. The ArchLinux will be installed in the partitions.



