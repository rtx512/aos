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
          ![image](https://github.com/rtx512/aos/blob/master/images/image.png)
      6. vim /etc/net/ifaces/ens21/options
          ![image](https://github.com/rtx512/aos/blob/master/images/image.png)
      7. vim /etc/net/ifaces/ens20/ipv4address
          ```
            10.10.10.1/24
          ```
      8. vim /etc/net/ifaces/ens21/ipv4address
          ```
            20.20.20.1/24
          ```
      9. reboot
