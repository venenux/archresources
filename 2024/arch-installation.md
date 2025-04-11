# arch install Guide 2024

## Getting Started

Welcome to the VenenuX Arch Linux guide with MATE desktop and/or LXDE/Openbox 
window manager installation guide... a step-by-step walkthrough of installing 
the Arch Linux. It has been carefully created based on the very bad own exp of 
installation on multiple devices over the years.

Unfortunatelly for older devices we must recommend Haiku OS (2000 to 2012) or
then VenenuX (2012 to 2024) becouse Arch linux is only for some recent devices!

#### Support and Feedback

Issues: If you come across any problems or have specific questions, please open 
an issue on the Codeberg repository for this guide. Use Telegram OpenTech group 
if you want direct communication.

Pull Requests: If you have improvements or additions to the guide, feel free to 
submit a pull request. Your contributions can help enhance the clarity of the 
guide for everyone.

## Preinstallation

1. This guide assume a linux OS previously working, if not use live disk of VenenuX

## Step 01: Downloading Arch Linux image

1. Go to Arch Linux downloads page https://archlinux.org/download/

2. Find **HTTP Direct Downloads** section and choose any download mirror.
   Select a mirror that is geographically closer to your location.

3. On the mirror page find archive named like `archlinux-2024.MM.DD-x86_64.iso` 
   or `archlinux-x86_64.iso`  or any other file with `.iso` suffix. Other files 
   (like _.txt_, _.tar.gz_ and even _.iso.sig_) are not needed for installation 
   process.

## Step 02: Preparing installation medium

1. Insert a USB-stick into your PC with at least 2Gb of space available on it. 
   You can also use a blank cd/dvd/bd rom and burn the iso image, or load into 
   the qemu cderom slot!

2. If you use USB stick, download https://github.com/balena-io/etcher/releases 
   and record the ISO downloaded image into the USB stick.

3. If you use CD/DVD rom, put the black disc into the optical device of your 
   computer device, and then open the burn software to record the image on!

### Step 03: Boot into Arch Linux medium

1. Insert the installation medium into the computer on which you wants install 
   Arch Linux. If you use qemu just put the iso file into the cdrom slot.

2. Power on your PC and press _boot menu key_. For most vendor based original, 
   this key is `F12`. For clones computers this could be `ESC` or `F1`.

3. Boot from CD/DVD/USB and wait until boot process is finished.

> Warning: &#128161; IMPORTANT NOTE Many BIOS'es by default come with activated 
**Secure boot** option. You might need to deactivate it in your BIOS.

## Section 01: Installation setup

Arch linux comes with the famous `arch-install` script, we will avoid it 
because we will setup disk partition layout and advanced formating.

### Step 04: networking and sync packages

Unfortunatelly Arch linux will need internet connection, you **can avoid this 
using a offline instalation method**

1. Connect to Network, **if you have wired network, is automatically**, 
   for WiFi just using `iwctl` and check connection is established:

```
iwctl

station wlan0 get-networks

station wlan0 connect

exit

ping 1.1.1.1
```

2. Synchronize pacman packaes by run `pacman -Syy`

### Step 05: Disk partitioning

> Warning: &#128161; IMPORTANT we will use GPT table, otherwise use "o" instead of "g".
> Warning: &#128161; CAUTION we will use `/dev/nvme0n1` as disk targe, could be `/dev/sda`.

1. Partition main storage device using `fdisk` utility. You can find storage 
   device name using `lsblk` command.

<dl><dd>
<pre>
$ <b>fdisk /dev/nvme0n1</b>
                <i>[repeat this command until existing partitions are deleted]</i>
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

### Step 06: Create filesystem and mounting partitions

> Warning: &#128161; only first root partition will be with journaling! for performance

2. Create filesystems on created disk partitions:

```
mkfs.fat -F 32 -S 1024 -n EFI /dev/nvme0n1p1

mkfs.ext4 -b 1024 -L ROOTAR /dev/nvme0n1p2

tune2fs -O "^encrypt" /dev/nvme0n1p2

tune2fs -O "^quota" /dev/nvme0n1p2

mkswap -L SWAPAR /dev/nvme0n1p3

mkfs.ext4 -b 1024 -L DATAAR /dev/nvme0n1p4

tune2fs -O ^has_journal /dev/nvme0n1p4

tune2fs -O "^encrypt" /dev/nvme0n1p4

tune2fs -O "^quota" /dev/nvme0n1p4
```

3. Correctly mount all filesystems to the `/mnt`:

```
mkdir -p /mnt/boot/efi /mnt/home

mount -t ext4 -o rw,relatime,exec,dev,suid,barrier=0,errors=remount-ro,commit=1800 /dev/nvme0n1p2 /mnt

mount -t vfat -o rw,relatime,umask=0077,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro /dev/nvme0n1p1 /mnt/boot/efi

mount -t ext4 -o rw,noatime,exec,nodev,nosuid,barrier=0,errors=remount-ro,commit=1800 /dev/nvme0n1p4 /mnt/home

swapon /dev/nvme0n1p3
```

### Step 07: Installing the base system

4. Install essential packages into new filesystem and generate fstab:

```
pacstrap -i /mnt base linux linux-firmware doas nano base-devel git wget less networkmanager man-db iptables mkinitcpio gawk perl psmisc

genfstab -U -p /mnt > /mnt/etc/fstab
```

### Step 08: Basic configuration and boot of new system

1. Chroot into freshly created filesystem by run `arch-chroot /mnt`

2. Setup system locale and timezone, sync hardware clock with system clock:

```
sed -i s|#en_US.UTF-8|en_US.UTF-8|g /etc/locale.gen

locale-gen

echo "LANG=en_US.UTF-8" > /etc/locale.conf

ln -sf /usr/share/zoneinfo/America/Panama /etc/localtime

hwclock --systohc
```

3. Setup system hostname:

```
echo archisshit > /etc/hostname

cat > /etc/hosts << EOF
127.0.0.1 localhost
::1       localhost
127.0.1.1 archisshit
EOF
```

4. Add new users and setup passwords:

```
pacman -S bash

useradd -m -G wheel,storage,power,audio,video,disk,games,users,network,input,lp,optical -s /bin/bash general

passwd root

passwd general
```

5. Add wheel group to doas file to allow users to run doas-sudo-shim command from AUR repos in future:

```
cat > /etc/doas.conf << EOF
permit nopass root
permit nopass :wheel
EOF
```


6. Install administrative packages and enable post network minimal environment

```
pacman -S mc aspell cabextract gawk p7zip unace unarj unrar unzip zip doas bash coreutils nano

pacman -S dhcpcd networkmanager systemd-resolvconf openssh libfido2

systemctl enable dhcpcd

systemctl enable NetworkManager

systemctl enable systemd-resolved

systemctl enable sshd
```

7. Install and configure GRUB to next boot of the system:

```
pacman -S grub efibootmgr os-prober

grub-install /dev/nvme0n1

cp /etc/default/grub  /etc/default/grub.orig

cat > /etc/default/grub << EOF
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="Archshit"
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 efi=runtime elevator=noop vsyscall=emulated iommu=on acpi_enforce_resources=lax noaslr iomem=relaxed"
GRUB_CMDLINE_LINUX=""
GRUB_PRELOAD_MODULES="part_gpt part_msdos"
GRUB_DISABLE_OS_PROBER=false
GRUB_TIMEOUT_STYLE=menu
GRUB_TERMINAL=console
GRUB_GFXMODE=800x600
GRUB_DISABLE_LINUX_UUID=false
GRUB_DISABLE_OS_PROBER=false
EOF

grub-mkconfig -o /boot/grub/grub.cfg

efibootmgr --create --disk /dev/nvme0n1 --part 1 --loader '\EFI\arch\grubx64.efi' --label 'Arshit Boot Linux' --unicode -e 3
```

9. Exit chroot, unmount all disks and reboot:

```
exit

umount /mnt/boot/efi

umount /mnt/home

umount /mnt

reboot
```

## Section 02: Configuring userspace after initial system setup &#127919;

This is depending of your needs, in this case we setup a MATE desktop with 
alternate Openbox/mate minimal desktop. We choose MATE becouse is the only 
mayor desktop with equilibrated performance and features, XFCE4 is so ugly!

### Prerequisites

1. Already pass and performs all of the section 1 above!
2. Reboot and start in your new disk with Arch Linux installation.
3. We assumed the networking setup from previous sectioin!
4. Install the `opendoas` package with `pacman -S opendoas doas`
5. login with the user `general` we configured previously

### Step 09: Basic configuration of userspace

1. Activate time syncronization using NTP:

```
doas timedatectl set-ntp true
```

2. Install common desktop utilities, console utilities, bluetooth and sound support

```
doas pacman -S \
 aspell cabextract gawk p7zip unace unarj unrar unzip zip doas bash coreutils nano \
 dialog intel-ucode fuse2 lshw powertop acpi htop tree socat lsof unhide psmisc \
 wget rsync aria2 base-devel git less groff bash-completion yazi ueberzugpp \
 dbus iw wpa_supplicant tcpdump mtr net-tools ethtool openbsd-netcat procps-ng \
 alsa-oss alsa-plugins alsa-ucm-conf pulseaudio pavucontrol sof-firmware alsa-firmware \
 pango lxappearance uget dunst fehfl flameshot gsimplecal

doas pacman -S bluez bluez-utils blueman

doas systemctl enable bluetooth
```

### Step 10 enable the AUR repository management

Pacman only handles updates for pre-built packages in its repositories. 
AUR packages are redistributed in form of PKGBUILDs and need an AUR helper 
to automate the re-build process. However, keep in mind that a rebuild of a 
package may be required when its shared library dependencies are updated, 
not only when the package itself is updated.

```
doas pacman -S sudo base-devel git

mkdir -p /home/general/Devel && /home/general/Devel

git clone https://aur.archlinux.org/yay.git

cd yay && makepkg -is
```

### Step 11: tuneup the hardware support

1. **Enable hibernation support** We wil use the first partition only, if you 
   want another one, must be configured manually, those steps only setup 
   automatically the first one! We will detect SWAP partition and then..
   Open GRUB configuration file and add resume UUID to the line boot, 
   all of those commands must be run as root and not using `sudo` or `doas` 
   becouse the variable expansion will not be parsed to `GRUB_CMDLINE_LINUX_DEFAULT`
   so then you can hibernate your system using: `systemctl hibernate`

```
grep swap /etc/fstab | sed 's/   */:/g' | cut -d: -f1 | cut -f1 | cut -d' ' -f1 | grep -v "#" | head -n1

PSWAP=$(grep swap /etc/fstab | sed 's/   */:/g' | cut -d: -f1 | cut -f1 | cut -d' ' -f1 | grep -v "#" | head -n1)

sed -i s/"GRUB_CMDLINE_LINUX_DEFAULT=\""/"GRUB_CMDLINE_LINUX_DEFAULT=\"resume=$PSWAP "/ /etc/default/grub

grub-mkconfig -o /boot/grub/grub.cfg

sed -i s/HOOKS.*/"HOOKS=(base udev resume autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)"/ /etc/mkinitcpio.conf

mkinitcpio -p linux
```

> Warning: &#128161; Those hooks dont assume encripted filesystems, neither raid setups!

> Warning: &#128161; all of those commands cannot be run using `sudo` or `doas`

2. **Enable printing support on your PC**: also optionally install hp plugins 
   for firmware updates b using `doas hpsetup -i`

```
doas pacman -S cups cups-filters cups-pdf system-config-printer

doas systemctl enable cups.service

doas pacman -S \
 foomatic-db-engine foomatic-db-nonfree-ppds foomatic-db-nonfree \
 hplip gutenprint foomatic-db-gutenprint-ppds

doas cups-genppdupdate
```

3. **[Optional] Improve laptop battery usage with TLP** - 
   this utility that basically does kernel settings tweaking but over desktops 
   machines will decrease performance.. **only use if you are on laptop** 

```
doas pacman -S tlp tlp-rdw

doas systemctl enable tlp

doas systemctl enable NetworkManager-dispatcher.service

doas systemctl mask systemd-rfkill.service

doas systemctl mask systemd-rfkill.socket
```

4. **[Optional] use avahi services to auto discover devices**, you can browse 
   manually by run `avahi-browse --all` aftyer avahi setup and install:

```
doas pacman -S avahi nss-mdns

doas sed -i s/.*disallow-other-stacks.*/disallow-other-stacks=yes/ /etc/avahi/avahi-daemon.conf

doas systemctl enable avahi-daemon.service

doas systemctl start avahi-daemon.service
```

## Section 03: Userspace desktop setup

Se next tutorial: [arch-postinstallation.md](arch-postinstallation.md)


