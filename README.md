### Задание 0
- RTR-L
  1. hostnamectl set-hostname rtr-l.au.team; exec bash
  2. vim /etc/net/sysctl.conf
      ```
        изменить:
        net.ipv4.ip_forward = 1
      ```
  3. cp -r /etc/net/ifaces/ens19/ /etc/net/ifaces/ens20/
  4. cp -r /etc/net/ifaces/ens19/ /etc/net/ifaces/ens21/
  5. vim /etc/net/ifaces/ens20/options
       ![image](https://github.com/rtx512/aos/blob/master/images/img1.png)
  7. vim /etc/net/ifaces/ens21/options
       ![image](https://github.com/rtx512/aos/blob/master/images/img1.png)
  8. vim /etc/net/ifaces/ens20/ipv4address
      ```
        10.10.10.1/24
      ```
  9. vim /etc/net/ifaces/ens21/ipv4address
      ```
        20.20.20.1/24
      ```
  10. reboot
- L-SRV
  1. hostnamectl set-hostname l-srv.au.team; exec bash
  2. vim /etc/net/ifaces/ens19/options
      ![image](https://github.com/rtx512/aos/blob/master/images/img1.png)
  4. vim /etc/net/ifaces/ens19/ipv4address
       ```
         10.10.10.100/24
       ```
  6. vim /etc/net/ifaces/ens19/ipv4route
       ```
         default via 10.10.10.1
       ```
  8. reboot
- ADMIN-PC
  1. hostnamectl set-hostname admin-pc.au.team; exec bash
  2. vim /etc/net/ifaces/ens19/options
       ![image](https://github.com/rtx512/aos/blob/master/images/img1.png)
  4. vim /etc/net/ifaces/ens19/ipv4address
       ```
         20.20.20.150/24
       ```
  6. vim /etc/net/ifaces/ens19/ipv4route
       ```
         default via 20.20.20.1
       ```
  8. reboot
- Проверка
  1. с L-SRV: ping 20.20.20.150

### Задание 1
- RTR-L
  1. apt-get update
  2. apt-get install nftables
  4. vim /etc/nftables/nftables.nft
       ```
         table inet nat {
           chain my_masquerade {
             type nat hook postrouting priority srcnat;
             oifname "ens19" masquerade
           }
         }
       ```
  6. systemctl enable --now nftables
  7. systemctl status nftables
- Проверка
  1. на L-SRV: ping 8.8.8.8
  2. на ADMIN-PC: ping 8.8.8.8
### Задание 2
- RTR-L
  1. apt-get update
  2. apt-get install dhcp-server
  3. vim /etc/sysconfig/dhcpd
       ```
         DHCPDARGS="ens20 ens21"
       ```
  5. s
