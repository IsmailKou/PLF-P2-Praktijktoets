# BMC EXAM CHEAT SHEET (FINAL + COMMENTS)

## LINUX BASIS

    whoami                # toon huidige gebruiker
    id                    # toon user + groepen
    sudo -i               # volledige root shell
    sudo cmd              # voer 1 command uit als root

    pwd                   # huidige directory
    ls -l                 # lijst bestanden + rechten
    ls -la                # inclusief hidden files
    cd dir                # ga naar directory
    cd ..                 # 1 map omhoog

## FILES / DIRECTORIES

    touch file            # maak leeg bestand
    mkdir dir             # maak directory
    mkdir -p a/b/c        # maak geneste directories

    cp file1 file2        # kopie file
    cp -r dir1 dir2       # kopie directory (BELANGRIJK)

    mv old new            # verplaats of hernoem

    rm file               # verwijder file
    rm -r dir             # verwijder directory
    rm -rf dir            # force delete (gevaarlijk)

## BESTAND BEKIJKEN / ZOEKEN

    cat file              # toon volledige inhoud
    less file             # scroll door bestand
    head file             # eerste regels
    tail file             # laatste regels
    tail -f log           # live log volgen

    grep "text" file      # zoek tekst in file
    grep -r "text" /dir   # zoek recursief

    find / -name file     # zoek bestand op naam
    find / -perm -4000    # zoek SUID files

## PERMISSIONS (HEEL BELANGRIJK)

    ls -l                 # check rechten

    chmod 755 file        # rwx r-x r-x (owner alles)
    chmod 700 file        # alleen owner toegang

    chown user:group file # verander eigenaar

    chmod u+s file        # SUID (draait als eigenaar)
    chmod g+s dir         # SGID (group overerving)
    chmod +t /tmp         # sticky bit (alleen eigenaar delete)

👉 meeste fouten = verkeerde permissions

## NETWORK (DEBUG)

    ip a                  # toon IP adressen
    ip route              # toon routing tabel

    ping 8.8.8.8          # test internet
    ping host             # test DNS + netwerk

    nslookup host         # DNS lookup
    dig host              # uitgebreide DNS test

    curl http://site      # test webserver

    ss -tuln              # open poorten bekijken

👉 ping IP werkt maar hostname niet = DNS fout

## SERVICES (SYSTEMCTL)

    systemctl start svc   # start service
    systemctl stop svc    # stop service
    systemctl restart svc # herstart service
    systemctl status svc  # check status
    systemctl enable svc  # start bij boot

    journalctl -xe        # bekijk fouten/logs (ZEER BELANGRIJK)
    journalctl -u svc     # logs van specifieke service

## DISK / STORAGE

    lsblk                 # overzicht disks/partities
    fdisk -l              # gedetailleerde info

    mkfs.ext4 /dev/sdX    # maak filesystem
    mkfs.xfs /dev/sdX     # alternatief filesystem

    mount /dev/sdX /mnt   # mount disk
    umount /mnt           # unmount disk

    df -h                 # bekijk disk gebruik
    blkid                 # toon filesystem types

## FSTAB (AUTO MOUNT)

    nano /etc/fstab       # configureer mounts bij boot
    mount -a              # test fstab (BELANGRIJK)

👉 fout in fstab = boot failure

## LVM

    pvcreate /dev/sdX     # maak physical volume
    vgcreate vg1 /dev/sdX # maak volume group
    lvcreate -L 1G -n lv1 vg1   # maak logical volume

    lvdisplay             # toon LV info
    vgdisplay             # toon VG info

    lvextend -L+1G /dev/vg1/lv1  # vergroot volume

👉 flexibel storage systeem

## RAID

    mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
    # maak RAID1 (mirror)

    cat /proc/mdstat      # status RAID bekijken
    mdadm --detail /dev/md0  # detail info

👉 RAID ≠ backup

## DHCP (LINUX)

    dnf install dhcp-server     # installeer DHCP

    nano /etc/dhcp/dhcpd.conf   # configuratie

    dhcpd -t -cf /etc/dhcp/dhcpd.conf   # config check

    systemctl restart dhcpd     # herstart service
    systemctl enable dhcpd      # auto start

    cat /var/lib/dhcpd/dhcpd.leases   # uitgegeven IP's

👉 werkt niet → config + logs checken

## DNS (BIND)

    dnf install bind bind-utils   # install DNS

    named-checkconf -z            # check config
    systemctl restart named       # herstart DNS

    dig host @127.0.0.1           # test DNS lokaal
    nslookup host                 # simpele test

👉 DNS moet werken voor alles (AD!)

## WEB / APACHE

    dnf install httpd             # install webserver
    systemctl enable --now httpd  # start + enable

    curl http://IP                # test website

    tail -f /var/log/httpd/error_log   # error logs

👉 403 error = permissions / SELinux

## SELINUX (VALKUIL)

    getenforce            # check status
    setenforce 0          # permissive (tijdelijk)
    setenforce 1          # enforcing

    ls -Z                 # check context

    chcon -R -t httpd_sys_content_t /dir   # tijdelijke fix

    semanage fcontext -a -t httpd_sys_content_t "/dir(/.*)?"
    restorecon -R /dir    # permanente fix

👉 SELinux blokkeert vaak zonder duidelijke error

## SSH

    ssh user@host         # login

    nano /etc/ssh/sshd_config   # config file

    PermitRootLogin no          # root login uit
    PasswordAuthentication no   # wachtwoord uit

    systemctl restart sshd      # herstart SSH

    ssh-keygen -t ecdsa         # maak key
    ssh-copy-id user@host       # upload key

## WINDOWS CORE / POWERSHELL

    sconfig                     # basis config menu

    rename-computer core1       # naam wijzigen
    restart-computer            # reboot

    Add-Computer -DomainName bmc.test   # domain join

    ipconfig /all               # netwerk info
    ipconfig /flushdns          # reset DNS cache

    Test-Connection host        # ping in PowerShell

    New-SmbShare                # share maken
    Get-SmbShare                # shares bekijken

## TROUBLESHOOTING (MEEST BELANGRIJK)

    werkt niets →
    1. ip a
    2. ping
    3. nslookup / dig
    4. systemctl status
    5. journalctl -xe

    web werkt niet →
    - permissions goed?
    - SELinux?
    - firewall open?

    DNS werkt niet →
    - named actief?
    - zone correct?
    - dig test?

## KERN PRINCIPES

    DNS nodig voor AD
    DHCP conflict vermijden
    RAID ≠ backup
    SELinux blokkeert vaak
    permissions = grootste foutbron
    logs lezen = oplossing
