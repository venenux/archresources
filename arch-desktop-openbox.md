# arch desktop tune Guide

## Getting Started

Welcome to the VenenuX Arch Linux guide with minimal desktop and/or LXDE/Openbox 
window manager installation guide...

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

## Section 01 desktop minimal Openbox


### Step 01 desktop base packages

Those packages will bring base applicartions to use any minimal desktop

```
yay -Syu gtk3 galculator tint2 gmrun pavucontrol arandr xinit-xsession mesa \
 archlinux-xdg-menu xdg-utils upower gdk-pixbuf2 gdk-pixbuf-xlib xorg-xinit \
 gtk-update-icon-cache gtkmm3 gtkmm-4.0 xdg-user-dirs-gtk shared-mime-info \
 libxrandr libxinerama libxcursor libxtst libxss lightdm lightdm-gtk-greeter \
 accountsservice xf86-input-libinput xf86-input-evdev libinput xorg-xinput \
 xdotool xautomation xorg-xkill desktop-file-utils polkit-gnome polkit \
 ttf-dejavu ttf-bitstream-vera xorg-fonts-75dpi xorg-fonts-100dpi xorg-xdpyinfo \
 pcmanfm libfm-extra menu-cache gvfs simple-scan gvfs-gphoto2 gvfs-mtp \
 gvfs-afc ifuse gvfs-goa gvfs-onedrive gvfs-google gvfs-wsdd gvfs-smb \
 openbox lxappearance lxappearance-obconf jgmenu tint2 xvkbd viewnior python-pyxdg

yay -Syu gtk-nocsd-git

cat > /etc/X11/xinit/xinitrc.d/51gtk-nocsd.sh << EOF
#!/bin/sh
          if [ -z "\$GTK_CSD" ] ; then
              GTK_CSD=0
          fi
          export GTK_CSD
          if [ x"\$GTK_CSD"x = x"0"x ] ; then
              export LD_PRELOAD="/usr/lib/libgtk-nocsd.so.0\${LD_PRELOAD:+:\$LD_PRELOAD}"
          fi
EOF
```

### Step 02 preparation of X11

```
cat > /etc/X11/Xwrapper.config < EOF
needs_root_rights = yes
EOF

cat > /etc/X11/xorg.conf.d/20-intel.conf << EOF
      Section "Device"
        Identifier  "Intel Graphics HD-GMA"
        Driver      "intel"
        Option      "DRI" "2"
        Option      "AccelMethod"  "sna"
        Option      "TearFree"        "false"
        Option      "TripleBuffer"    "false"
        Option      "SwapbuffersWait" "false"
        Option      "VSync" "false"
        Option      "RelaxedFencing" "false"
      EndSection
EOF

systemctl enable lightdm.service

systemctl start lightdm.service
```

### Step 03 networking and remote management

* networking packages

```
yay -Syu gtk4 nm-connection-editor network-manager-applet iso-codes \
 mobile-broadband-provider-info modemmanager usb_modeswitch ppp wpa_supplicant \
 dhcpcd net-tools nmap networkmanager-pptp networkmanager-openconnect \
 networkmanager-vpnc networkmanager-openvpn bluez gnome-nettool

systemctl enable ModemManager

systemctl start ModemManager

systemctl enable NetworkManager

systemctl start NetworkManager
```

* remote management preparation

```
yay -Syu nomachine anydesk-bin dkms nawk elfutils lsb-release

systemctl enable nxserver.service anydesk.service

systemctl start nxserver

systemctl start anydesk
```

### Step 04 general user

```
useradd -m -s /bin/bash general

usermod -a -G audio,video,disk,games,input,optical,power,storage general

usermod -a  -G adm,log,users,lp,kvm,network general
```

### Step 05 multimedia pacakges

```
yay -Syu --needed gstreamer gst-plugins-base-libs gst-plugins-bad gst-plugin-gif \
 gst-plugin-gtk gst-plugin-gtk4 gst-plugin-mp4 gst-plugins-good gst-plugins-ugly \
 ffmpeg gst-libav mpv sox alsa-plugins libavtp libsamplerate libcamera-tools \
 pulseaudio pavucontrol pulseaudio-equalizer pulseaudio-bluetooth pulseaudio-alsa \
 zathura zathura-pdf-poppler zathura-djvu zathura-cb jack2
```

## Anex


### Step 06 using only gtk2 packages

```
yay -Syu gtk2 gtk-engines gtkglext gtkmm gtkrc-reload gtkspell gksu \
 ttf-dejavu ttf-bitstream-vera light-locker-settings yad-gtk2 \
 leafpad mount-gtk2 xvkbd gxmessage terminus-font-td1
```

### Step 07 extended multimedia base

```
yay -Syu --needed gst-plugin-rsclosedcaption tesseract tesseract-data-eng \
 qt5gtk2 qt6-base qt6gtk2 qt6-multimedia-gstreamer qt6-multimedia-ffmpeg \
 gnome-mplayer gnome-disk-utility strawberry hardinfo2
```

### Step 08 development tools for desktops

```
yay -Syu heidisql mysql-workbench vscodium meld
```


## Menu

Using xdg_open as a pipe menu gives you the added benefit of having a menu that automatically updates when you install new applications.

Add the following line below the `<menu id="root-menu">` element in `~/.config/openbox/menu.xml`:

`<menu id="applications" label="Applications" execute="xdg_menu --format openbox3-pipe" />`

A very basic example:

```
<?xml version="1.0" encoding="UTF-8"?>

<openbox_menu xmlns="http://openbox.org/3.4/menu">

<menu id="root-menu" label="Openbox 3">
  <menu id="applications" label="Applications" execute="xdg_menu --format openbox3-pipe" />
  <separator />
  <item label="Log Out">
    <action name="Exit">
      <prompt>yes</prompt>
    </action>
  </item>
</menu>

</openbox_menu>
```

With update-menus
Uncomment openbox in `/etc/update-menus.conf`.
Run update-menus as root.
Add the following line below the `<menu id="root-menu">` element in `~/.config/openbox/menu.xml`:

```
<menu id="applications" label="Applications" execute="cat /var/cache/xdg-menu/openbox/menu.xml"/>
```

