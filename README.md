Not a discription.
    passwd snow – поменять пароль
   **iptables settings**
   iptables -t nat -A POSTROUTING -o ens160 -j MASQUERADE / ACCEPT
   iptables -A FORWARD -i ens192/160 -j ACCEPT
   **SSH config** 
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_isp/br/hq
   ssh-copy-id -i ~/.ssh/id_rsa_isp isp
   vim ~/.ssh/config:
Host Zeus
    Hostname 192.168.3.2
    User capybara
    IdentityFile ~/.ssh/id_rsa_Zeus and etc…

ssh ISP, Zeus, Odin, and after that complete cmnds bellow:
На ISP, Zeus, Odin:
**/etc/openssh/sshd_config**
PermitRootLogin no
AllowUsers capybara
PubkeyAuthentication yes
PasswordAuthentication no

**FRR:**
apt-get update -y && apt-get upgrade
apt-get install frr frr-pythontools

**frr ISP:**

9. vtysh
10.	router ospf 
11.	router-id 2.2.2.2
12.	network 10.0.12.0/30 area 0
13.	network 192.168.1.0/24 area 0
    
**frr Zeus:**

15. vtysh
16.	router ospf 
17.	router-id 1.1.1.1
18.	network 10.0.12.0/30 area 0
19.	network 192.168.1.0/24 area 0


**DNS:**

**apt-get install bind**

vim /etc/bind/options.conf
listen-on { any; };
listen-on-v6 { none; };
forwarders { 77.88.8.8; };
allow-query { any; };
vim rfc1912.conf 
""
zone "sa.admin"{
  type master;
  file "sa.admin"
  allow-update { none; };
  }
zone "1.168.192.in-addr.arpa" {
  type master;
  file "192.168.1.in-addr.arpa"
  allow-update { none; };
}
zone "2.168.192.in-addr.arpa" {
  type master;
  file "192.168.2.in-addr.arpa"
  allow-update { none; };
}
zone "3.168.192.in-addr.arpa" {
  type master;
  file "192.168.3.in-addr.arpa"
  allow-update { none; };
}
""

vim /etc/bind/zone/sa.admin

Zeus  IN  A  192.168.1.3
ISP   IN  A  192.168.1.2
Odin  IN  A  192.168.2.3
Prometheus IN  A  192.168.3.3

vim /etc/bind/zone/192.168.1.in-addr.arpa
3  IN  PTR  Zeus.sa.admin
2  IN  PTR  ISP.sa.admin
...

resolv.conf on all devices:
  domain sa.admin
  nameserver 192.168.1.3(Zeus(dns))


**DHCP**
apt-get install dhcpd 
cd /etc/dhpc
cp dhcpd.conf.sample dhcpd.conf - sample (!)
vim dhcpd.conf

subnet 192.168.1.0 netmask 255.255.255.0 {
  option routers            192.168.1.2;
  option netmask            255.255.255.0;
  option nis-domain         "sa.admin";
  option domain-name        "sa.admnin";
  option domain-name-server 192.168.1.3;  

}


subnet 192.168.3.0 netmask 255.255.255.0 {
  option routers            192.168.3.2;
  option netmask            255.255.255.0;
  option nis-domain         "sa.admin";
  option domain-name        "sa.admnin";
  option domain-name-server 192.168.3.2;  
  default lease-time 21600;
  max-lease-time 43200;

  host Prometheus {
  hardware ethernet 00:00:00:00;
  fixed-address 192.168.3.3;
  }
}

after that: 
vim /etc/net/ifaces/ens???/options -> BOOTPROTO=dhcp!
systemctl restart network. and whoolya, you`ll get dynamic global ip addr, see by ip a.
Bug fixing: 
journalctl -xeu <name of service where error exists>
