# arch emulation tune Guide

**Notes on installing QEMU Arch Linux on AMD64 UEFI systems**

## Support and Feedback

**Issues**: https://codeberg.org/venenux/archresources/issues/new or also 
https://t.me/opentechnologies or https://t.me/latam_programadores or also 
if you want direct communication join to https://t.me/+W94Jx4A4AzdkMmYx

## Part 1 - Install qemu software

#### Requeriments

* Those notes assumes X86_64v2 as minimun machine https://bbs.archlinux.org/viewtopic.php?id=281639
* This guide will assume QEMU > 8.0.1
* This guide assumes SSD devices for qemu images on virtual machines
* This guide assumes you have a Fully supported GPU on ber metal machine

#### install qemu basic emulation and tools

```
pacman -Syu edk2-shell qemu-hw-uefi-vars qemu-ui-curses \
 qemu-audio-dbus qemu-ui-dbus qemu-user qemu-user-binfmt \
 qemu-system-ppc qemu-block-dmg dmg2img \
 qemu-system-s390x qemu-system-s390x-firmware qemu-hw-s390x-virtio-gpu-ccw \
 qemu-system-riscv edk2-riscv64 qemu-system-riscv-firmware \
 qemu-system-amr edk2-aarch64 qemu-system-arm-firmware qemu-system-aarch64 \
 qemu-system-x86 edk2-ovmf qemu-hw-display-virtio-vga seabios \
 qemu-img qemu-tools qemu-block-curl cdrkit make
```

#### optional qemu avanced sound with graphics and headless integration

```
pacman -Syu qemu-ui-egl-headless virtiofsd qemu-chardev-baum \
 qemu-ui-gtk qemu-ui-opengl qemu-ui-sdl qemu-hw-display-virtio-vga-gl \
 qemu-audio-alsa qemu-audio-pa qemu-audio-pipewire qemu-audio-sdl \
 qemu-hw-display-virtio-gpu qemu-hw-display-virtio-gpu-pci \
 qemu-hw-display-virtio-gpu-gl qemu-hw-display-virtio-gpu-pci-gl \
 qemu-hw-usb-redirect qemu-vhost-user-gpu
```

## Part 2 - Configuring qemu network and hardware

### Configuring network bridge with network manager

```
nmcli -t -f UUID connection show | while read -r uuid; do sudo nmcli connection delete "$uuid"; do

nmcli con add ifname br0 type bridge con-name br0

nmcli con add type bridge-slave ifname eno1 master br0 con-name eno1

nmcli con mod eno1 802-3-ethernet.cloned-mac-address random

nmcli con mod eno1 802-3-ethernet.generate-mac-address-mask "FF:FF:FF:00:00:00 24:B6:FD:00:00:00"

nmcli con mod eno1 connection.autoconnect yes

nmcli con mod eno1 connection.autoconnect-priority 1000

nmcli con mod eno1 connection.permissions ""

nmcli con mod br0 bridge.stp no

nmcli con mod br0 bridge.priority 0

nmcli con mod br0 connection.autoconnect yes

nmcli con mod br0 connection.autoconnect-priority 1000

nmcli con mod br0 bridge.priority 1000

nmcli con mod br0 connection.permissions ""

nmcli con mod br0 bridge.mac-address 24:B6:FD:AA:BB:CC

nmcli con mod br0 802-3-ethernet.mac-address 24:B6:FD:AA:BB:CC

nmcli con mod br0 802-3-ethernet.cloned-mac-address 24:B6:FD:AA:BB:CC

nmcli -t -f UUID connection show | while read -r uuid; do nmcli connection down "$uuid"; do

nmcli connection up br0

nmcli connection up eno1
```

### setup qemu network support

```
mkdir -p /etc/qemu/ && touch /etc/qemu/bridge.conf

cat > /etc/qemu/bridge.conf << EOF
allow br0
allow br1
allow virbr0
allow virbr1
allow bridge0
EOF

sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" > /etc/sysctl.d/ip-forward.conf

sysctl -w kernel.split_lock_mitigate=0
echo "kernel.split_lock_mitigate = 0" > /etc/sysctl.d/split-lock.conf

chown -R root:qemu /etc/qemu && chmod 640 /etc/qemu/bridge.conf

chmod u+s /usr/lib/qemu/qemu-bridge-helper

iptables -t nat -A POSTROUTING -o br0 -j MASQUERADE
iptables -P FORWARD ACCEPT

pacman -S  shadow

usermod -a -G kvm,input,disk,render,qemu general
```

### setup qemu hardware support

```
cat > /etc/modprobe.d/kvm.conf << EOF
options kvm_intel nested=1
options kvm_intel emulate_invalid_guest_state=0
options kvm_amd nested=1
options kvm ignore_msrs=1 report_ignored_msrs=0
EOF

rmmod kvm && modprove kvm
grep kvm /etc/modules|| echo kvm >> /etc/modules

rmmod tun && rmmod vhost_net && rmmod vhost && modprobe vhost_net && modprobe tun

grep tun /etc/modules|| echo tun >> /etc/modules
grep vhost_net /etc/modules|| echo vhost_net >> /etc/modules
```

### qemu EFI secure boot support

```
doas -u general yay -S edk2-ovmf-fedora
```

## Part 3 - Running USER qemu machines

```
pacman -Syu binfmt-support shadow qemu-user qemu-user-binfmt

systemctl restart systemd-binfmt
```

### qemu to emulate a instalation of other linux debian

```
pacman -S coreutils debootstrap

mkdir /home/general/Devel/chroots/debianSarm

cd /home/general/Devel/chroots/

/usr/sbin/debootstrap --foreign --arch=arm64 --no-check-gpg stable debianSarm http://deb.debian.org/debian

qemu-arm -L /home/general/Devel/chroots/debianSarm /home/general/Devel/chroots/debianSarm/bin/bash
```

### qemu to emulate arm instaled disk into your kernel

```
mount /dev/sdx2 /mnt/rootalpine

qemu-arm -L /mnt/rootalpine/ /mnt/rootalpine/usr/bin/ls
```

## PArt 4 - Running FULL qemu machines

```
pacman -S  shadow qemu-system-ppc qemu-system-s390x qemu-system-riscv \
 qemu-system-amr edk2-aarch64 qemu-system-arm-firmware qemu-system-aarch64 \
 qemu-system-x86
```

### qemu to emulate msdos 6 in computer

```
/usr/bin/qemu-img create -f raw -o preallocation=falloc vm2-dos-vd1.raw 512M

/usr/bin/qemu-system-i386 \
 -nodefaults -monitor stdio \
 -m 64M -overcommit mem-lock=off -parallel none \
 -name "vm2-dos-vd-x86" \
 -rtc base=localtime,clock=host,driftfix=slew \
 -machine pc,accel=kvm:tcg,sata=off,hpet=off,dump-guest-core=off,usb=off \
 -cpu pentium \
 -device ide-hd,bus=ide.0,drive=hd0,bootindex=1 \
 -device ide-cd,bus=ide.1,drive=cd0,bootindex=2 \
 -device pcnet,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::2022-:22 \
 -device sb16,iobase=0x220,irq=5,dma=1,audiodev=au0 \
 -device adlib,audiodev=au0 \
 -device pcspk,audiodev=au0 \
 -device cirrus-vga,vgamem_mb=16 \
 -audiodev pa,id=au0 \
 -display sdl \
 -drive index=0,file=vm2-dos-vd1.raw,format=raw,if=none,id=hd0,media=disk \
 -drive index=1,file=MSDOS622UP.iso,format=raw,if=none,id=cd0,media=cdrom \
 -drive index=0,file=MSDOS6D1.img,if=floppy,format=raw,id=fd0 \
 -drive index=1,file=MSDOS6SD.img,if=floppy,format=raw,id=fd1 \
 -boot order=cda,once=a,menu=on
```

### qemu to emulate MAC OS 9 sonata

```
/usr/bin/qemu-img create -f raw -o preallocation=off vm3-macos-09-vd1-ide-ppc.raw 2G

/usr/bin/qemu-system-ppc \
 -m 756 \
 -name "vm3-macos-092-hd-ide-ppc-32bit" \
 -rtc base=localtime,clock=host,driftfix=slew \
 -machine mac99,accel=kvm:tcg,via=pmu \
 -cpu g4 \
 -device sungem,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::3022-:22 \
 -device pci-ohci,id=usb0 -device usb-mouse,bus=usb0.0 -device usb-kbd,bus=usb0.0 \
 -device ati-vga,vgamem_mb=32,model=rage128p \
 -display sdl \
 -drive index=0,id=hd0,file=vm3-macos-092-hd1-ide-g4ppc.raw,format=raw,media=disk \
 -drive index=1,id=cd0,file=macos_921_iso_ppc.iso,format=raw,cache=none,media=cdrom \
 -boot once=c,order=cd,menu=on
```


### qemu to emulate windos xp

* Donwload files from @MEDIATECA205 Telegram group https://t.me/c/2676150917/80/83

```
/usr/bin/qemu-img create -f raw -o preallocation=falloc vm4-WinXP-ide-32bit.raw 4G

/usr/bin/qemu-system-i386 -nodefaults \
 -m 4096 -overcommit mem-lock=off \
 -name "vm4-WinXP-ide-32bit" \
 -rtc base=localtime,clock=host,driftfix=slew \
 -machine pc,accel=kvm:tcg,sata=off,hpet=off,dump-guest-core=off,usb=off,pcspk-audiodev=au0 -serial none -parallel none \
 -cpu pentium3,kvm=off,vendor=GenuineIntel,hv-relaxed,hv-vapic,hv-spinlocks=0x1fff,hv-time \
 -device ide-hd,bus=ide.0,drive=hd0,bootindex=1 \
 -device ide-cd,bus=ide.1,drive=cd0,bootindex=2 \
 -device pcnet,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::4022-:22 \
 -device ac97,audiodev=au0 \
 -device qemu-xhci,id=usb0 -device usb-tablet,bus=usb0.0 \
 -device ati-vga,vgamem_mb=32,model=rage128p \
 -audiodev pa,id=au0 \
 -display sdl \
 -drive index=0,file=vm4-WinXP-ide-32bit.raw,format=raw,if=none,id=hd0,media=disk \
 -drive index=1,file=windows_XP_32_pro.iso,format=raw,if=none,id=cd0,media=cdrom \
 -boot order=cda,once=a,menu=on,strict=on
```

### qemu linux olders with most common compatible settings

```
/usr/bin/qemu-img create -f raw -o preallocation=falloc vm5-linux-sata-32bit.raw 4G

/usr/bin/qemu-system-i386 -nodefaults \
 -m 2048 -overcommit mem-lock=off \
 -name "vm5-linux-sata-32bit" \
 -rtc base=localtime,clock=host,driftfix=slew \
 -machine pc,accel=kvm:tcg,sata=off,hpet=off,dump-guest-core=off,usb=off,pcspk-audiodev=au0 \
 -cpu pentium3,kvm=off,vendor=GenuineIntel,vmware-cpuid-freq=on \
 -device ide-hd,bus=ide.0,drive=hd0,bootindex=1 \
 -device ide-cd,bus=ide.1,drive=cd0,bootindex=2 \
 -device pcnet,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::5022-:22 \
 -device ac97,audiodev=au0 \
 -device qemu-xhci,id=usb0 -device usb-tablet,bus=usb0.0 \
 -device ati-vga,vgamem_mb=32,model=rage128p \
 -audiodev pa,id=au0 \
 -display sdl \
 -drive index=0,file=vm5-linux-sata-32bit.raw,format=raw,if=none,id=hd0,media=disk \
 -drive index=1,file=alpine-ext-3.8.5-x86.iso,format=raw,if=none,id=cd0,media=cdrom \
 -boot order=cda,once=a,menu=on,strict=on
```

### qemu linux newers runing on linux full opengl virtio

> **Warning** : both guest and host must have linux 6.14+, mesa 25+, qemu 11+

> **Warning** : 64bit+KVM+Vulkan mandatory, AMD 2018+ , INTEL 2016+ , XE 2022+

```
/lvm6-linux-virt-64bit.raw 4G

/usr/bin/qemu-system-x86_64 -nodefaults \
 -m 4096 -overcommit mem-lock=off \
 -name "vm6-linux-virt-64bit" \
 -rtc base=localtime,clock=host,driftfix=slew -accel kvm,honor-guest-pat=on \
 -machine pc-q35-9.2,kernel-irqchip=on,sata=off,hpet=off,dump-guest-core=off,usb=off,pcspk-audiodev=au0 \
 -bios /usr/share/edk2/x64/OVMF.4m.fd \
 -cpu host,kvm=off,vmware-cpuid-freq=on,+hv-time,+hv-relaxed,+hv-vapic,+hv-spinlocks=0x1fff -smp sockets=2,cpus=4 \
 -device virtio-scsi-pci,id=scsi0 \
 -device scsi-hd,bus=scsi0.0,channel=0,scsi-id=0,lun=0,drive=hd0,bootindex=1 \
 -device scsi-cd,bus=scsi0.0,channel=0,scsi-id=1,lun=0,drive=cd0,bootindex=2 \
 -device virtio-net-pci,netdev=nd1 -netdev user,id=nd1,restrict=off,hostfwd=tcp::6022-:22 \
 -device virtio-sound-pci,audiodev=au0,streams=2 \
 -device virtio-tablet-pci -device virtio-keyboard-pci \
 -device virtio-gpu-gl,hostmem=1G,blob=on \
 -audiodev pa,id=au0 \
 -display sdl,gl=es \
 -drive index=0,file=vm6-linux-virt-64bit.raw,format=raw,if=none,id=hd0,media=disk,aio=io_uring,cache=none \
 -drive index=1,file=alpine-ext-3.22.0-x64.iso,format=raw,if=none,id=cd0,media=cdrom \
 -boot order=cda,once=a,menu=on,strict=on
```

# Follows

* Install some king of desktop:
   * Wayland + Labwc Windowmanager as desktop : [arch-desktop-labwc.md](arch-desktop-labwc.md)
   * Xorg + Openbox Windowmanager as desktop : [arch-desktop-openbox.md](arch-desktop-openbox.md)
   * Wayland + Omarchi ShellDesktop as desktop : [arch-desktop-omarchi.md](arch-desktop-omarchi.md)
* Install IDE and development base environtment : [arch-devel.md](arch-devel.md)

# LICENSE

CC-BY-SA-NC
