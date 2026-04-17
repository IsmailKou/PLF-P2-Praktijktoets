# =========================================
# PLF04: DNS en DHCP
# =========================================

## Tabel 1 IP-instellingen

| VM | NIC | Netwerk | IP adres | Geïnstalleerde services tijdens labs |
|----|-----|--------|----------|--------------------------------------|
| **BMC-DC1** (Windows Server met GUI) | ethernet0 | LAN Segment "Windows" of VMnet2 Host-only | 192.168.14.1/24 | SMB Server, IIS Management |
| **BMC-Core1** (Windows Server Core) | ethernet0 | LAN Segment "Windows" of VMnet2 Host-only | 192.168.14.4/24 | DHCP server x.y.2.0, HTTP + SMB server |
| **BMC-PC1** (Windows 10) | ethernet0 | LAN Segment "Windows" of VMnet2 Host-only | DHCP | Tests Windows services |
|  | ethernet1 | LAN Segment "Linux" of VMnet3 Host-only | DHCP | Tests Linux services |
| **rocky1.bmc.example** (Rocky 8) | ethernet0 | LAN Segment "Linux" of VMnet3 Host-only | 192.168.15.1/24 | DHCP server x.y.3.0, Apache, Squid, Samba |
|  | ethernet1 | VMnet8 NAT | DHCP | Optioneel |
| **VyOS** | ethernet0 | VMnet8 NAT | DHCP | Optioneel |
|  | ethernet1 | LAN Segment "Linux" of VMnet3 Host-only | 192.168.15.254/24 | |

---

## DHCP Linux

```bash
sudo dnf install dhcp-server        # installeer DHCP server
sudo nano /etc/dhcp/dhcpd.conf     # config bestand

systemctl start dhcpd              # start service
systemctl enable dhcpd             # start bij boot
systemctl status dhcpd             # check status

ipconfig /renew                    # client nieuw IP
