# arch emulation tune Guide

## Getting Started

Welcome to the VenenuX Arch Linux emulation guide guide with minimal qemu and 
gamin emulation installation guide...

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


## Install qemu software

```
yay -Syu seabios ovmf qemu-system-x86 qemu-system-common qemu-system-gui qemu-efi
```

## Configuring qemu settings

```
mkdir -p /etc/qemu/ && touch /etc/qemu/bridge.conf

cat > /etc/qemu/bridge.conf << EOF
allow br0
allow br1
allow virbr0
EOF
sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" > /etc/sysctl.d/ip-forward.conf

chown -R root:qemu /etc/qemu && chmod 640 /etc/qemu/bridge.conf

chmod u+s /usr/lib/qemu/qemu-bridge-helper

iptables -t nat -A POSTROUTING -o br0 -j MASQUERADE
iptables -t nat -A POSTROUTING -o br1 -j MASQUERADE
iptables -P FORWARD ACCEPT
```

## setup qemu to start

```
rmmod tun && rmmod vhost_net && rmmod vhost && modprobe vhost_net && modprobe tun
grep tun /etc/modules|| echo tun >> /etc/modules
grep vhost_net /etc/modules|| echo vhost_net >> /etc/modules
```