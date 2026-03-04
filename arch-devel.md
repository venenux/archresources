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

### Step 01 install base devel tools

Those are need to build and clean others package and external projects.

Login as "general" user as we previously show you in previous guides.

```
yay -Syu --noconfirm less man-db perl-libwww ruby-svn2git git mercurial \
 devtools base-devel openssh binutils bison make cmake autoconf-archive \
```

### Step 02 install codium IDE and editors

You must already has yay and desktop installed! Check the others guides.

Login as "general" user as we previously show you in previous guides.

```
yay -Syu --noconfirm --needed code code-features code-marketplace \
 sakura gnome-keyring meld dbeaver orbada
```

### Step 03 install SQL base tools

Those are need to connect and then perform task as cliet on sql services

Login as "general" user as we previously show you in previous guides.

```
yay -Syu mariadb-lts-clients mysql-workbench sqlitebrowser sqlite
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

