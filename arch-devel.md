# arch desktop devel tune Guide

**Notes on installing QEMU Arch Linux on AMD64 UEFI systems**

## Support and Feedback

**Issues**: https://codeberg.org/venenux/archresources/issues/new or also 
https://t.me/opentechnologies or https://t.me/latam_programadores or also 
if you want direct communication join to https://t.me/+W94Jx4A4AzdkMmYx

## Advertising

1. move to ram LOG behaviour to save disk filetime:
    * rsyslog para archivos planos de log y registro
    * log2ram para no gastar disco SSD o sea lenta la escritura
    * logrotate para que los archivos planos no molesten
2. active the rsyslog and logrotate to generate small files
3. configure log2ram to use more ram so all those files fit in!
4. configure systemd to use less space so log2ram do not waste space
5. the amount of RAM must be at least 10Gb!
6. we install a default jdk and the kryto npm+nodejs and manage 
  others versions with
7. we use the code package and not the direct MScode, neither codium, 
  to improve compatibility: vscodium package is less compatible!
    * install package that patch product.json every time 
      code package is updated
    * Syncs APIs with oficial ones so extensions can work in code
    * Disable telemetry.
    * Use ~/.vscode and ~/.Code instead of ~/.vscode-oss and ~/.Code OSS
      necesary to some non-open source plugins to work!
8. docker will be use as demand by socket activation, no service running
    * move to extra disk the docker containers directory 
      to save disk lifetime and improve performance I/O access
    * the docker partiton must be 1024 small block size, becouse 
      the docker container are many small files of layers
9. Optionally setup specific environment like Java or Android
    * Apache deploy with or witout PHP latest and extensions
    * Packages for SQL browsing
    * Sqlite instalation and sqlite browser
    * Mysql Percona with tmpfs so do not harm SSD or/and 
      then improve I/O performance
    * Mariadb with tmpfs so do not harm SSD or/and 
      then improve I/O performance
    * Postgresql proper initialization and console editor usage

## Base development

### Step 01 install rsyslog and log2ram

```
mkdir -p /home/general/Devel && cd /home/general/Devel

git clone https://github.com/hartmark/libfastjson-aur

cd /home/general/Devel/libfastjson-aur && makepkg -i --skippgpcheck

doas -u general yay -Syu --needed rsyslog logrotate log2ram

grep -q "imjournal" /etc/rsyslog.conf || sudo sed -i '/\$ModLoad imklog/a $ModLoad imjournal   # provides systemd traps' /etc/rsyslog.conf

systemctl enable --now rsyslog

systemctl start rsyslog

sed -i 's/\bweekly\b/daily/g' /etc/logrotate.conf

sed -i 's/\bweekly\b/daily/g' /etc/logrotate.conf

sed -i 's/\bsize\s\+.*/size 5M/g' /etc/logrotate.conf

sed -i 's/\brotate\s\+.*/rotate 1/g' /etc/logrotate.conf

systemctl restart logrotate

sed -i 's/^SIZE=.*/SIZE=1512M/' /etc/log2ram.conf

journalctl --vacuum-size=24M

sed -i -E 's/^[# ]*SystemMaxUse=.*/SystemMaxUse=96M/' /etc/systemd/journald.conf

systemctl disable log2ram-daily.timer

systemctl enable log2ram
```

### Step 02 cache packages to tmpfs

> **Warning** will use a different disk/patch for package downloads 
 so your main disks will not be harmed by constants shity updates 
 We will use /dev/sde3 or tmpfs for cache directory

```
pacman -Scc

mkdir -pv /var/cache/pacman/pkg
mount -t tmpfs -o rw,nodev,nosuid,size=4G,noatime tmpfs /var/cache/pacman/pkg

genfstab -P -U -f /var/cache / >> /etc/fstab
```

### Step 03 install base devel tools

```
doas -u general yay -Syu --noconfirm \
 ruby-svn2git git mercurial less man-db perl-libwww \
 devtools base-devel openssh binutils lld bison make cmake autoconf-archive \
 jdk-openjdk java-openjfx gtk2-ng-git nano nodejs-lts-krypton npm

export EDITOR=nano
export VISUAL=code
```

### Step 04 install codium IDE and editors

```
doas -u general yay -Syu --noconfirm --needed \
 code code-features code-marketplace \
 sakura gnome-keyring meld dbeaver gitg
```

### Step 05 Container docker management

> **Warning** will use a different disk/patch for docker layers!
 so your main disks will not be harmed by the piece of shit named docker
 We will use /dev/sde3 fpr docker layers so your main SDD-s never will be harmed

```
pacman -Syu docker docker-buildx docker-compose composefs qemu-user-static qemu-user-static-binfmt

systemctl disable --now docker.service

systemctl enable docker.socket

usermod -a -G docker general

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
pacman -Syu ducker lazydocker
```

## SQL development

### SQL browsers and managers

* SQL browsers:
    * SQLITE: `pacman -Syu sqlitebrowser`
    * MySQL: `pacman -Syu mysql-workbench`
    * PostgreSQL: `pacman -Syu rainfrog`
* SQL managers:
    * MySQL: `pacman -Syu mysql-workbench`
    * Mult Database: `pacman -Syu dbeaver`

### Sqlite development

```
doas -u general yay -Syu sqlitebrowser sqlite
```

### Mariadb development

```
pacman -Rns percona-server-clients percona-server mariadb-clients mariadb mysql mysql-clients

doas -u general yay -Syu mariadb-lts-clients mariadb-lts mysql-workbench

mkdir -p /var/lib/mysql
chown mysql:mysql /var/lib/mysql
chmod 750 /var/lib/mysql

mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql

mkdir -pv /var/lib/mysqltmp
chown mysql:mysql /var/lib/mysqltmp
mount -t tmpfs -o rw,gid=mysql,uid=mysql,size=512M,mode=0750,noatime tmpfs /var/lib/mysqltmp

genfstab -P -U -f /var/lib / >> /etc/fstab

cat > /etc/my.cnf.d/tmpfs.cnf << EOF
tmpdir      = /var/lib/mysqltmp
EOF

systemctl enable --now mariadb

mariadb-tzinfo-to-sql /usr/share/zoneinfo | mariadb -u root
```

### Mysql percona development

```
pacman -Rns mariadb-lts-clients mariadb-lts mariadb-clients mariadb mysql mysql-clients

doas -u general yay -Syu percona-server-clients percona-server mysql-workbench percona-toolkit

mkdir -p /var/lib/mysql /var/log/mysql
chown mysql:mysql /var/lib/mysql
chmod 750 /var/lib/mysql

touch /var/log/mysqld.log
chown mysql:mysql /var/log/mysql
chmod 770 /var/log/mysql

doas -u mysql mysqld --initialize --datadir=/var/lib/mysql --log-error=/var/log/mysql/error.log

grep 'temporary password' /var/log/mysql/error.log

mkdir -pv /var/lib/mysqltmp
chown mysql:mysql /var/lib/mysqltmp
mount -t tmpfs -o rw,gid=mysql,uid=mysql,size=512M,mode=0750,noatime tmpfs /var/lib/mysqltmp

genfstab -P -U -f /var/lib / >> /etc/fstab

cat > /etc/my.cnf.d/tmpfs.cnf << EOF
tmpdir      = /var/lib/mysqltmp
EOF

systemctl enable --now mysqld

mysql-tzinfo-to-sql /usr/share/zoneinfo | mariadb -u root
```

### PostgreSQL development

```
pacman -Syu postgresql rainfrog

doas -u postgres initdb --locale=C.UTF-8 --encoding=UTF8 -D /var/lib/postgres/data

doas -u postgres pg_checksums --pgdata /var/lib/postgres/data --enable

systemctl enable --now postgresql
```

### ODBC + SQLserver development

```
pacman -Syu unixodbc freetds

cat > /usr/share/doc/freetds/odbcinst.ini << EOF
[FreeTDS]
Description     = ODBC Driver for Sybase and MsSQL
Driver          = /usr/lib/libtdsodbc.so
EOF

odbcinst -i -d -f /usr/share/doc/freetds/odbcinst.ini

cat >> /home/general/.odbc.ini << EOF
[MSSQLserverConnec]
Description  = A shit of database from Mocosoft
Driver   = FreeTDS
Trace   = Yes
TraceFile  = /tmp/errc.log
Database  = mydbname
Server   = 192.168.1.100
UserName  = dba
Password  = sql
Port   = 1723
ReadOnly  = Yes
TDS_Version  = 8.0
EOF

chown general:general /home/general/.odbc.ini

doas -u general isql MSSQLserverConnec dba sql
```

### ODBC + Sybase development

```
pacman -Syu unixodbc freetds

cat > /usr/share/doc/freetds/odbcinst.ini << EOF
[FreeTDS]
Description     = ODBC Driver for Sybase and MsSQL
Driver          = /usr/lib/libtdsodbc.so
EOF

odbcinst -i -d -f /usr/share/doc/freetds/odbcinst.ini

cat >> /home/general/.odbc.ini << EOF
[SybaseConnec]
Description  = The prdecesor of MSSQLserver
Driver   = FreeTDS
Trace   = Yes
TraceFile  = /tmp/errc.log
Database  = sybasedemo
Server   = 192.168.1.100
UserName  = dba
Password  = sql
Port   = 2638
TDS_Version  = 5.0
EOF

chown general:general /home/general/.odbc.ini

doas -u general isql SybaseConnec dba sql
```

### ODBC + MySQL development

```
doas -u general yay -Syu unixodbc mariadb-connector-odbc

cat > /usr/share/doc/mariadb-connector-odbc/odbcinst.ini << EOF
[MySQL]
Description     = ODBC Driver for MySQL
Driver          = /usr/lib/libmaodbc.so
EOF

odbcinst -i -d -f /usr/share/doc/mariadb-connector-odbc/odbcinst.ini

cat >> /home/general/.odbc.ini << EOF
[AnyMysqlMariadbPercona]
Description  = Mysql any compatible
Driver   = MySQL
Database  = mysql
Server   = 192.168.1.100
Port   = 3306
Socket   = /var/run/mysqld/mysqld.sock
EOF

chown general:general /home/general/.odbc.ini

doas -u general isql AnyMysqlMariadbPercona mysql password
```

### ODBC + PostgreSQL development

```
doas -u general yay -Syu unixodbc psqlodbc

cat > /usr/share/doc/psqlodbc/odbcinst.ini << EOF
[PostgreSQL]
Description=General ODBC for PostgreSQL 
Driver=/usr/lib/psqlodbca.so 
Setup=/usr/lib/psqlodbcw.so
EOF

odbcinst -i -d -f /usr/share/doc/psqlodbc/odbcinst.ini

cat >> /home/general/.odbc.ini << EOF
[postgresdb]
Driver         = PostgreSQL
Description    = PostgreSQL odbc access to postgres db
Database       = postgres
ServerName     = 127.0.0.1
Port           = 5432
User           = root
Password       = toor
ReadOnly        = Yes
EOF

chown general:general /home/general/.odbc.ini

doas -u general isql postgresdb root
```

## PHP development and HTTP deploys

### apache2 deploy with home user

```
pacman -Syu apache

sed -i 's/public_html/Devel/' /etc/httpd/conf/extra/httpd-userdir.conf

sed -i 's/.*ProtectHome=.*/ProtectHome=no/' /etc/systemd/system/httpd.service.d/hardening.conf

mkdir /home/general/Devel

setfacl -m u:http:x /home/general

setfacl -R -m u:http:rx /home/general/Devel

systemctl daemon-reload

sysyemctl enable --now httpd
```

### Apache deploy with PHP development

```
pacman -Rs $(pacman -Qsq php-legacy)

pacman -Syu composer php php-apache php-apcu php-fpm \
 php-dblib php-enchant php-snmp php-memcached php-mongodb

sed -s -i -r 's|.*cgi.fix_pathinfo=.*|cgi.fix_pathinfo=1|g' /etc/php*/php.ini
sed -s -i -r 's#display_errors =.*#display_errors = On#g' /etc/php*/php.ini
sed -s -i -r 's#memory_limit =.*#memory_limit = 512M#g' /etc/php*/php.ini
sed -s -i -r 's#upload_max_filesize =.*#upload_max_filesize = 128M#g' /etc/php*/php.ini
sed -s -i -r 's#post_max_size =.*#post_max_size = 256M#g' /etc/php*/php.ini
sed -s -i -r 's#^file_uploads =.*#file_uploads = On#g' /etc/php*/php.ini
sed -s -i -r 's#^max_file_uploads =.*#max_file_uploads = 12#g' /etc/php*/php.ini
sed -s -i -r 's#^allow_url_fopen = .*#allow_url_fopen = On#g' /etc/php*/php.ini
sed -s -i -r 's#^.default_charset =.*#default_charset = "UTF-8"#g' /etc/php*/php.ini
sed -s -i -r 's#^.max_execution_time =.*#max_execution_time = 150#g' /etc/php*/php.ini
sed -s -i -r 's#^max_input_time =.*#max_input_time = 90#g' /etc/php*/php.ini
sed -s -i -r 's#^.date.timezone =.*#date.timezone = "America/Panama"#g' /etc/php*/php.ini

cat > /etc/php/conf.d/deafultsext.ini << EOF
extension=bcmath
extension=bz2
extension=calendar
extension=curl
extension=dba
extension=exif
extension=ffi
extension=ftp
extension=gettext
extension=gmp
extension=iconv
extension=intl
extension=ldap
extension=mysqli
extension=pdo_mysql
extension=shmop
extension=soap
extension=sockets
extension=sysvmsg
extension=sysvsem
extension=sysvshm
extension=zip
EOF

echo "extension=apcu" > /etc/php/conf.d/apcu.ini
pacman -Syu php-apcu

echo "extension=xsl" > /etc/php/conf.d/xsl.ini
pacman -Syu php-xsl

echo "extension=gd" > /etc/php/conf.d/gd.ini
pacman -Syu php-gd

echo -e "extension=pdo_pgsql\nextension=pgsql" > /etc/php/conf.d/pgsql.ini
pacman -Syu php-pgsql

echo -e "extension=pdo_sqlite\nextension=sqlite3" > /etc/php/conf.d/sqlite.ini
pacman -Syu php-sqlite

echo -e "extension=pdo_odbc\nextension=odbc" > /etc/php/conf.d/odbc.ini
pacman -Syu php-odbc

echo "extension=imagick" > /etc/php/conf.d/imagick.ini
pacman -Syu php-imagick librsvg

systemctl enable --now php-fpm

sed -s -i -r 's|LoadModule mpm_event_module|#LoadModule mpm_event_module|g' /etc/httpd/conf/httpd.conf
sed -s -i -r 's|.*LoadModule mpm_prefork_module|LoadModule mpm_prefork_module|g' /etc/httpd/conf/httpd.conf

cat > /etc/httpd/conf/conf.d/phpmodule.conf << EOF
LoadModule php_module modules/libphp.so
AddHandler php-script .php
Include conf/extra/php_module.conf
EOF

sysyemctl enable --now httpd

cat > /srv/http/index.php << EOF
<?php
phpinfo();
?>
EOF
```

### Web management of Databases with adminer

```
doas -u general yay -Syu adminer

cat > /etc/httpd/conf/conf.d/adminer.conf << EOF
Include conf/extra/httpd-adminer.conf
EOF

sysyemctl restart httpd
```

## Java development

### Manage JDK and JRE

```
doas -u general yay -Syu --noconfirm java-runtime-common \
 jdk-openjdk gtk2-ng-git java-openjfx \
 jdk8-openjdk jdk17-openjdk jdk25-openjdk \
 java8-openjfx java17-openjfx ffmpeg3.4

archlinux-java set java-25-openjdk
```

### Java decompilers

```
doas -u general yay -Syu cfr jadx

```

## Android Development

MODES:

* Full upstream will require to install "dummy" package names as example `android-sdk-platform-tools-dummy`
* Full distro packages require to install all package from Arch as example `android-sdk-platform-tools`
* Mixed upstream is best choice, you can use `android-tools` but all the rest must be removed to use the upstream provided.

Check [arch-android-development-notes.md](arch-android-development-notes.md)

### Android ADB management and SCRCPY

```
pacman -S android-tools android-udev libmtp scrcpy

usermod -a -G adbusers,input,storage,power,wheel general
```

Check https://arch-deb.blogspot.com/2025/07/conectar-el-telefono-o-smarttv-y.html



# Follows

* Install some king of desktop:
   * Wayland + Labwc Windowmanager as desktop : [arch-desktop-labwc.md](arch-desktop-labwc.md)
   * Xorg + Openbox Windowmanager as desktop : [arch-desktop-openbox.md](arch-desktop-openbox.md)
   * Wayland + Omarchi ShellDesktop as desktop : [arch-desktop-omarchi.md](arch-desktop-omarchi.md)
* Install Qemu and manage virtuals : [arch-emulation.md](arch-emulation.md)

# LICENSE

CC-BY-SA-NC
