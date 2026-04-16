# DNS (BIND)

    sudo dnf install bind bind-utils

    → Installeert DNS server + tools

    sudo nano /etc/named.conf

    → Hoofdconfiguratie aanpassen

    listen-on port 53 { any; };

    → DNS luistert op alle interfaces

    allow-query { localhost; 192.168.15.0/24; };

    → Welke clients DNS mogen gebruiken

    zone "bmc.example"

    → Forward zone (naam → IP)

    zone "15.168.192.in-addr.arpa"

    → Reverse zone (IP → naam)

    sudo -u named named-checkconf -z

    → Controleert config en zones

    sudo systemctl restart named

    → Herstart DNS

    systemctl status named

    → Check status

    ss -tulnp | grep :53

    → Check of DNS luistert

    dig @127.0.0.1 host

    → Test lokaal

    dig -x IP

    → Reverse DNS test

# DHCP

    sudo dnf install dhcp-server

    → Installeert DHCP

    nano /etc/dhcp/dhcpd.conf

    → Config bestand

    dhcpd -t -cf file

    → Check config

    systemctl start dhcpd

    → Start DHCP

    systemctl status dhcpd

    → Check status

    cat /var/lib/dhcpd/dhcpd.leases

    → Bekijk leases

# Storage

    lsblk

    → Bekijk disks en partities

    fdisk -l

    → Toon schijven

    fdisk /dev/sdb

    → Partities maken

    mkfs.ext4 /dev/sdb1

    → Filesystem maken

    mount /dev/sdb1 /mnt

    → Mount schijf

    umount /mnt

    → Unmount

    df -h

    → Bekijk opslaggebruik

    nano /etc/fstab

    → Automatisch mounten

# RAID

    mdadm --create /dev/md0 ...

    → Maak RAID array

    cat /proc/mdstat

    → Bekijk RAID status

# LVM

    pvcreate /dev/sdb1

    → Maak physical volume

    vgcreate vg_data /dev/sdb1

    → Maak volume group

    lvcreate -n lv1 -L 1G vg_data

    → Maak logical volume

    mkfs.ext4 /dev/vg_data/lv1

    → Filesystem maken

    mount /dev/vg_data/lv1 /mnt

    → Mount LVM

# Web + SELinux

    systemctl start httpd

    → Start webserver

    systemctl status httpd

    → Check status

    firewall-cmd --add-service=http

    → Open HTTP in firewall

    curl http://server

    → Test website

    ls -Z

    → Bekijk SELinux context

    restorecon -R /var/www

    → Reset SELinux context

    chcon -t httpd_sys_content_t file

    → Pas SELinux context aan

# Permissions

    ls -l

    → Bekijk rechten

    chmod 755 file

    → Rechten aanpassen

    chown user file

    → Eigenaar aanpassen

    chgrp group file

    → Groep aanpassen

    chmod u+s file

    → SetUID bit

    chmod +t /tmp

    → Sticky bit
