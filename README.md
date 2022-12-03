# DEMO2022

![image](https://user-images.githubusercontent.com/79700810/149956179-026c9bba-e6fc-495a-81df-4c6ddb0ec1d6.png)

### RTR-L Полная настройка
```
enable
conf t
hostname RTR-L
```
```
interface gi 1
ip address 4.4.4.100
ip nat outside
no sh

interface gi 2
ip address 192.168.100.254
ip nat inside
no sh

access-list 1 permit 192.168.100.0 0.0.0.255
ip nat inside source list 1 interface gi 1 overload
```
```
line vty 0 15
transport input shh
```
```
ip route 0.0.0.0 0.0.0.0 4.4.4.1

interface Tunnel1
ip address 172.16.1.1 255.255.255.0
tunnel mode gre ip
tunnel source 4.4.4.100
tunnel destanation 5.5.5.100

router eigrp 6500
network 192.168.100.0 0.0.0.255
network 172.16.1.0 0.0.0.255
```
```
crypto isakmp policy 1
encr aes
auth pre-share
hash sha256
group 14

crypto isakmp key TheSecretMustBeAtLeast13bytes address 5.5.5.100
crypto isakmp nat keepalive 5

crypto ipsec transform-set TSET esp-aes 256 esp-sha256-hmac
mode tunnel

crypto ipsec profile VTI
set transform-set TSET

interface Tunnel1
tunnel mode ipsec ipv4
tunnel protection ipsec profile VTI
```
```
ip access-list extended Lnew
permit tcp any any established
permit tcp any host 4.4.4.100 eq 53
permit udp any host 4.4.4.100 eq 53
permit udp any host 4.4.4.100 eq 123
permit udp host 4.4.4.1 eq 53 host 4.4.4.100
permit tcp any host 4.4.4.100 eq 80
permit tcp any host 4.4.4.100 eq 443
permiy tcp any host 4.4.4.100 eq 2222
permit udp host 5.5.5.100 host 4.4.4.100 eq 500
permit esp any any
permit icmp any any

interface gi 1
ip access-group Lnew in
```
```
ip domain name srv.int.demo.wsr
ip name-server 192.168.100.200
ntp server ntp.int.demo.wsr
```
```
no ip http secure-server
no ip http server
do wr
reload
```
```
ip nat inside source static tcp 192.168.100.200 53 4.4.4.100 53
ip nat inside source static udp 192.168.100.200 53 4.4.4.100 53
ip nat inside source static tcp 192.168.100.100 80 4.4.4.100 80
ip nat inside source static tcp 192.168.100.100 443 4.4.4.100 443
ip nat inside source static tcp 192.168.100.100 22 4.4.4.100 2222
```
### RTR-R Полная настройка
```
enable
conf t
hostname RTR-R
```
```
interface gi 1
ip address 5.5.5.100
ip nat outside
no sh

interface gi 2
ip address 172.16.100.254
ip nat inside
no sh

access-list 1 permit 172.16.100.0 0.0.0.255
ip nat inside source list 1 interface gi 1 overload
```
```
line vty 0 15
transport input ssh
```
```
ip route 0.0.0.0 0.0.0.0 5.5.5.1

interface Tunnel1
ip address 172.16.1.2 255.255.255.0
tunnel mode gre ip
tunnel source 5.5.5.100
tunnel destanation 4.4.4.100

router eigrp 6500
network 172.16.100.0 0.0.0.255
network 172.16.1.0 0.0.0.255
```
```
crypto isakmp policy 1
encr aes
auth pre-share
hash sha256
group 14

crypto isakmp key TheSecretMustBeAtLeast13bytes address 4.4.4.100
crypto isakmp nat keepalive 5

crypto ipsec transform-set TSET esp-aes 256 esp-sha256-hmac
mode tunnel

crypto ipsec profile VTI
set transform-set TSET

interface Tunnel1
tunnel mode ipsec ipv4
tunnel protection ipsec profile VTI
```
```
ip access-list extended Rnew
permit tcp any any established
permit tcp any host 5.5.5.100 eq 80
permit tcp any host 5.5.5.100 eq 443
permit tcp any host 5.5.5.100 eq 2244
permit udp host 4.4.4.100 host 5.5.5.100 eq 500
permit esp any any
permit icmp any any

interface gi 1
ip access-group Rnew in
```
```
ip domain name srv.int.demo.wsr
ip name-server 192.168.100.200
ntp server ntp.int.demo.wsr
```
```
no ip http secure-server
no ip http server
do wr
reload
```
```
ip nat inside source static tcp 172.16.100.100 80 5.5.5.100 80
ip nat inside source static tcp 172.16.100.100 443 5.5.5.100 443
ip nat inside source static tcp 172.16.100.100 22 5.5.5.100 2244
```

### ISP
```
nano /etc/hostname
ISP
```
```
apt-cdrom add
apt-get update
apt-get upgrade
apt install chrony bind9(либо dnsmasq) network-manager
```
```
nano /etc/sysctl.conf

no.ipv4.ip_forward=1

sysctl -p
```
```
nmtui 
Настраиваем в соответствии с таблицей маршрутизации
```
```
nano /etc/chrony/chrony.conf

local stratum 4
allow 4.4.4.0/24
allow 3.3.3.0/24
```
### Настройка DNS с помощью Bind9
```
cp /etc/bind/db.local > /etc/bind/demo.db
cp /etc/bind/db.127 > /etc/bind/db.1.3
```
```
nano /etc/bind/named.conf.options

	forwarders {
		4.4.4.100
	};

	dnssec-validation no;
	allow-query { any; };
	listen-on-v6 { any; };
};
```
```
nano /etc/bind/named.conf.default-zones

zone "demo.wsr" {
	type master;
	allow-transfer { any; };
	file "/etc/bind/demo.db";
}

zone "1.3.in-addr.arpa" {
	type master;
	allow-transfer { any; };
	file "/etc/bind/db.1.3"
}
```
```
@ IN SOA demo.wsr. root.demo.wsr.
```
```
@ IN NS isp.demo.wsr.
isp	IN	A 3.3.3.1
www	IN	A	4.4.4.100
www	IN	A	5.5.5.100
internet	CNAME	isp.demo.wsr.
int	IN	NS	rtr-l.demo.wsr.
rtr-l	IN	A	4.4.4.100
```
```
systemctl restart bind9
```
### Настройка DNS с помощью DNSMASq

```
apt install dnsmasq

Вносим днс-записи в файл /etc/hosts:
3.3.3.1 isp isp.demo.wsr
4.4.4.100 www www.demo.wsr
5.5.5.100 www www.demo.wsr

Добавляем две строки в файл настроек /etc/dnsmasq.conf:
cname=internet.demo.wsr,isp.demo.wsr
server=/int.demo.wsr/4.4.4.100

Рестартуем сервис dnsmasq:
systemctl restart dnsmasq
```
### WEB-L,R Полная настройка
```
nano /etc/hostname
WEB-L
```
```
apt-cdrom add
apt update
apt upgrade
apt install openssh-server ssh cifs-utils chrony network-manager nginx
```
```
nmtui
Настройка интерфейсов в соответствии с таблицей Маршрутизации
```
```
systemctl start sshd
systemctl enable ssh

nano /etc/ssh/sshd_config
PermitRootLogin yes

systemctl restart sshd
systemctl restart ssh
```
```
nano /etc/chrony.conf

pool ntp.int.demo.wsr iburst

allow 192.168.100.0/24
```
```
nano /root/.smbclient
username=Administrator
password=P@ssw0rd

mkdir /opt/share

nano /etc/fstab
//srv.int.demo.wsr/smb /opt/share cifs user,rw,_netdev,credentials=/root/.smbclient	0	0
mount -a
```
```
Это делается после того, как мы экспортируем сертификат с SRV на /opt/share
openssl pkcs12 -nodes -nocerts -in www.pfx -out www.key
openssl pkcs12 -nodes -nokeys -in www.pfx -out www.cer

cp www.key /etc/nginx/www.key
cp www.cer /etc/nginx/www.cer

nano /etc/nginx/snippets/snakeoil.conf
/etc/nginx/www.cer
/etc/nginx/www.key
```
```
nano /etc/nginx/sites-avaliable/default

upstream backend {
	server 192.168.100.100:8080 fail_timeout=25;
	server 172.16.100.100:8080 fail_timeout=25;
}

server {
	listen 443 ssl default_server;
	include snippets/snakeoil.conf;
	
	server_name www.demo.wsr;
	
	location / {
		proxy_pass http://backend;
	}
}

server {	
	listen 80 default_server;
	server_name www.demo.wsr;
	return 301 https://www.demo.wsr;
}
```
```
systemctl restart nginx.service
```
```
Вставляем диск с докером
mkdir /mnt/app
mkdir /mnt/docker

mount /dev/sr0 /mnt/app
cp /mnt/app/* /mnt/docker/

cd /mnt/docker/

dpkg -i *.deb

docker load < /mnt/docker/appdocker0.tar

docker images
docker run --name app -p 8080:5000 -d appdocker0
docker ps
```
