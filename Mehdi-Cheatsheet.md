# DHCP

sudo dnf install dhcp-server  
sudo nano /etc/dhcp/dhcpd.conf  
dhcpd -cf /etc/dhcp/dhcpd.conf (check of je config goed staat)  
sudo systemctl start dhcpd  
sudo systemctl enable dhcpd  
systemctl status dhcpd  
cat /var/lib/dhcpd/dhcpd.leases  

---

# DNS

sudo dnf install bind bind-utils  
nano /var/named/bmc.example.zone  

    $TTL 300
    @   IN  SOA rocky1.bmc.example. hostmaster.bmc.example. (
            2026033101
            3M
            1M
            30D
            1M )

    @       IN  NS      rocky1.bmc.example.

    rocky1  IN  A       192.168.15.125

• SOA → info over zone (serial, timers)  
• NS → nameserver  
• A → hostname → IP  

sudo systemctl reload named  

## named.conf

/etc/named.conf  

    options {
        listen-on port 53 { any; };
        listen-on-v6 port 53 { ::1; };
        directory "/var/named";
        allow-query { localhost; 192.168.15.0/24; };
        recursion yes;
    };

    zone "bmc.example" IN {
        type master;
        file "bmc.example.db";
    };

    zone "15.168.192.in-addr.arpa" IN {
        type master;
        file "bmc.example.rev";
    };

• listen-on → op welke IP’s DNS luistert  
• allow-query → wie DNS mag gebruiken  
• zone → definieert een domein  
• forward zone → naam → IP  
• reverse zone → IP → naam  

---

## Reverse zone

/var/named/bmc.example.rev  

    $TTL 300
    @   IN  SOA rocky1.bmc.example. hostmaster.bmc.example. (
            2026033101
            3M
            1M
            30D
            1M )

    @       IN  NS      rocky1.bmc.example.

    125     IN  PTR     rocky1.bmc.example.

sudo chown root:named /var/named/bmc.example.db  
sudo chmod 640 /var/named/bmc.example.db  

sudo chown root:named /var/named/bmc.example.rev  
sudo chmod 640 /var/named/bmc.example.rev  

sudo -u named named-checkconf -z = check je file  

sudo systemctl start named  
sudo systemctl restart named  
sudo systemctl enable named  
systemctl status named  

---

# Storage

lsblk = overzicht disk en partities  
fdisk -l = disk details  
sudo fdisk /dev/sdb = partitie aanmaken  

## Variabelen partities aanmaken:
N  
T  
W  

## Filesysteem aanmaken:

/dev/sdb1 uitleg:  
/dev = device  
/a = disk 1 /b disk 2  
1 = eerste partitie  

Formateert partities:  

mkfs.ext2 /dev/sdb1  
mkfs.ext3 /dev/sdb2  
mkfs.xfs /dev/sdb3  

mount /dev/sdb1 /data/part1  
umount /data/part1  

df -h = ruimte bekijken  
nano /etc/fstab = automatische mounten bij booten  

pvcreate /dev/sdb1  
vgcreate vg_data /dev/sdb1  

---

# Webserver

curl http://rocky1.bmc.example/test.html  

---

# RAID

sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1  

👉 Betekent:  
• /dev/md0 = jouw RAID naam (standaard!)  
• /dev/sdb1 /dev/sdc1 = echte disks  

cat /proc/mdstat  

---

# Security

ls -l = rechten bekijken  
chown user:file = eigenaar wijzigen  
chgrp group file = group wijzigen  
chmod 755 file = rechten toewijzen  
chmod u+s file = draait als eigenaar  
chmod +t /tmp alleen eigenaar mag file  

---

# SELinux

ls -Z  

sudo mkdir -p /plf/www/html /plf/www/cgi-bin  
sudo chmod -R 755 /plf  

sudo systemctl status httpd  
sudo firewall-cmd --list-all  

sudo nano /plf/www/html/test.html  

## HTML file

```html
<html>
<head>
<title>Testpagina</title>
</head>
<body>
<h1>Testpagina</h1>
Pas als deze pagina te lezen is, dan staan de bestandsrechten en SELinux goed ingesteld
</body>
</html>

man curl; curl http://rocky1.bmc.example/test.html
 als het via browser gedaan
curl http://rocky1.bmc.example/test.html

Als we in de logfile met foutmeldingen van Apache kijken (sudo tail -n 25 /var/log/httpd/error_log), dan zien we een melding dat er een permissieprobleem op het bestand is, zelfs als de ‘normale’ bestandsrechten de hele wereld leesrechten geeft. Dit duidt erop dat SELinux roet in het eten gooit

De tool audit2why is onder RH meestal al geïnstalleerd. Met deze tool ga je de foutmelding analyseren.

sudo grep http /var/log/audit/audit.log | audit2why

De process security context (scontext) is niet compatibel met de target context (tcontext).

Kijk met ls -Z /var/www naar de rechten op de default Apache directory en op de /plf/www directory

curl http://rocky1.bmc.example/test.html

sudo tail -n 25 /var/log/httpd/error_log = webserver log
sudo grep http /var/log/audit/audit.log | audit2why

sudo restorecon -R /plf/www = tijdelijke fix

Conclusie

• SELinux kan toegang blokkeren
• /var/www werkt standaard
• /plf/www niet

Oplossing

chcon -R -t httpd_sys_content_t /plf/www
