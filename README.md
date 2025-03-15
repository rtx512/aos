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
       ![image](https://github.com/rtx512/aos/blob/master/images/img2.png)
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
       ![image](https://github.com/rtx512/aos/blob/master/images/img3.png)
  5. cp /etc/dhcp/dhcpd.conf.sample /etc/dhcp/dhcpd.conf
  6. vim /etc/dhcp/dhcpd.conf
       ![image](https://github.com/rtx512/aos/blob/master/images/img4.png)
       ```
         host l-srv {
           hardware ethernet XX:XX:XX:XX:XX:XX; # Замените на MAC-адрес L-SRV (на L-SRV через команду ip a смотрим mac-адрес для ens19)
           fixed-address 10.10.10.100;
         }
         host admin-pc {
           hardware ethernet YY:YY:YY:YY:YY:YY;  # Замените на MAC-адрес ADMIN-PC (на ADMIN-PC через команду ip a смотрим mac-адрес для ens19)
           option domain-name-servers 10.10.10.100, 94.232.137.105, 94.232.137.105;
         }
       ```
  8. systemctl enable --now dhcpd
  9. systemctl restart dhcpd
  10. systemctl status dhcpd #(если не работает, то команда dhcpd -t покажет ошибки)
- L-SRV
  1. vim /etc/net/ifaces/ens19/options
       ![image](https://github.com/rtx512/aos/blob/master/images/img5.png)
  2. reboot
  3. Проверить, что после перезагрузки ip a 10.10.10.100/24
- ADMIN-PC
  1. vim /etc/net/ifaces/ens19/options
       ![image](https://github.com/rtx512/aos/blob/master/images/img5.png)
  2. reboot
  3. Проверить, что после перезагрузки ip a 20.20.20.150/24
### Задание 3
- L-SRV
  1. apt-get install alterator-fbi
  2. systemctl start alteratord ahttpd
  3. systemctl enable --now alteratord ahttpd
  4. apt-get install alterator-net-domain task-samba-dc krb5-kdc
  5. rm -f /etc/samba/smb.conf
  6. rm -rf /var/lib/samba
  7. rm -rf /var/cache/samba
  8. mkdir -p /var/lib/samba/sysvol
  9. samba-tool domain provision --realm=AU.TEAM --domain=AU --adminpass='Password123P' --dns-backend=SAMBA_INTERNAL --server-role=dc --use-rfc2307 --option="dns forwarder=94.232.137.104"
      ![image](https://github.com/rtx512/aos/blob/master/images/img6.png)
  11. systemctl enable --now samba
  12. systemctl status samba #(Вылезали одни ошибки, перезагрузил машину при помощи)
  13. reboot
  14. systemctl status samba #(всё норм)
  15. Для провероки:
        ```
          samba-tool domain info 127.0.0.1
        ```
        ![image](https://github.com/rtx512/aos/blob/master/images/img7.png)
  16. samba-tool group add left
  17. samba-tool group add admin
  18. vim script.sh
        ```
          #!/bin/bash
          for i in {1..15}; do
                samba-tool user create user${i}.userl "user123U"
                samba-tool group addmembers left user${i}.userl
          done

          for i in {1..5}; do
            samba-tool user create user${i}.admin "user123U"
            samba-tool group addmembers admin user${i}.admin
          done
        ```
        ![image](https://github.com/rtx512/aos/blob/master/images/img8.png)
  20. chmod +x script.sh
  21. ./script.sh
  22. Проверка, что пользователи создались и добавились в группы:
        ```
          pdbedit -L | grep userl
          pdbedit -L | grep admin
          samba-tool group listmembers left
          samba-tool group listmembers admin
        ```
        ![image](https://github.com/rtx512/aos/blob/master/images/img9.png)
        ![image](https://github.com/rtx512/aos/blob/master/images/img10.png)
- ADMIN-PC
  1. apt-get update
  2. apt-get install samba-client krb5-kdc
  3. apt-get install -y task-auth-ad-sssd
  4. Предварительная проверка:
     ```
       host au.team
       smbclient -L l-srv.au.team -U Administrator
       kinit Administrator@AU.TEAM
       klist
     ```
     ![image](https://github.com/rtx512/aos/blob/master/images/img13.jpg)
  6. в
