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
ip address 4.4.4.100 255.255.255.0
ip nat outside
no sh

interface gi 2
ip address 192.168.100.254 255.255.255.0
ip nat inside
no sh

access-list 1 permit 192.168.100.0 0.0.0.255
ip nat inside source list 1 interface gi 1 overload
```
```
line vty 0 15
transport input ssh
login local
exit
crypro key generate rsa
1024
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
ip domain name int.demo.wsr
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
ip address 5.5.5.100 255.255.255.0
ip nat outside
no sh

interface gi 2
ip address 172.16.100.254 255.255.255.0
ip nat inside
no sh

access-list 1 permit 172.16.100.0 0.0.0.255
ip nat inside source list 1 interface gi 1 overload
```
```
line vty 0 15
transport input ssh
login local
exit
crypro key generate rsa
1024
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
ip domain name int.demo.wsr
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

После того как скопировали файлы в /mnt/docker/ встявляем диск BD1 и после продолжаем установку
cd /mnt/docker/

dpkg -i *.deb

docker load < /mnt/docker/appdocker0.tar

docker images
docker run --name app -p 8080:5000 -d appdocker0
docker ps
```
### SRV Полная настройка
```
Первым делом, после запуска SRV заходим в Server Manager, там переходим в вкладку Local Server и устанавливаем Hostname и IP-адрес в соотвествии с заданием. После этого перезагружаем SRV, чтобы изменения применились.
```
![image](https://i.imgur.com/nlVZMVx.png)
![image](https://i.imgur.com/lhQRTrT.png)
```
После перезагрузки устанавдиваем роли, которые нам будут нужны
```
![image](https://i.imgur.com/q6vUpeK.png)
```
Для этого, на главной страницы наижимаем на кнопку Manage > Add Roles and Features. Выбираем всё тоже самое, что указано на ниже представленной картинке
```
![image](https://i.imgur.com/IhSni8e.png)

```
Далее настройка DNS на SRV. В Server Meneger переходим на вкладку DNS, 
нажимем правой кнопокой мыши на строку с нашим IP-адресом и в всплывших вариантах выбираем DNS Manger
```
![image](https://i.imgur.com/JJOEQmy.png)
```
Далее всё делаем по следующим представленным картинкам
```
![image](https://user-images.githubusercontent.com/60313293/205475108-d742bc75-ef32-4376-a88b-5e075cdd8931.png)
![image](https://user-images.githubusercontent.com/60313293/205475178-bf42dfc8-0afa-48ca-b1c3-9c7f7ae334ca.png)
![image](https://user-images.githubusercontent.com/60313293/205475194-e63c2153-a115-4b89-a6ab-d7f56ffa1d7f.png)
![image](https://user-images.githubusercontent.com/60313293/205475201-15f5fe6c-1616-4bdc-9b89-1f96bd19707f.png)
![image](https://user-images.githubusercontent.com/60313293/205475228-77d1df65-dee9-412f-90a7-f36c640de211.png)
![image](https://user-images.githubusercontent.com/60313293/205475233-fa06f11d-cc00-48ab-a194-5b7f1df366b0.png)
![image](https://user-images.githubusercontent.com/60313293/205475239-f9842f6a-1134-4a3c-bb97-de0c9d3dffa8.png)
![image](https://user-images.githubusercontent.com/60313293/205475248-76ce8124-0d54-4e66-a771-ca7540924e98.png)
![image](https://user-images.githubusercontent.com/60313293/205475252-8d327603-613e-4e39-9c1c-16a0f1fec797.png)
```
После создания зон вводим в них адреса, которые представленны нам в задании
```
### Настройка RAID для smb
![image](https://i.imgur.com/pYIRsoi.png)
###
![image](https://i.imgur.com/qRDsGB4.png)
###
![image](https://i.imgur.com/UZHvOrz.png)
###
![image](https://i.imgur.com/4Uahgfy.png)
###
![image](https://i.imgur.com/Tqlf9l7.png)
###
![image](https://i.imgur.com/EceFEbp.png)
###
![image](https://i.imgur.com/7BUyhdD.png)
###
![image](https://i.imgur.com/OlOycCC.png)
###
![image](https://i.imgur.com/rxkPxAc.png)
###
![image](https://i.imgur.com/eK2KoJM.png)
###
![image](https://i.imgur.com/wL96RiT.png)
###
![image](https://i.imgur.com/jwKkJ36.png)

### Настройка сертификации
```
После установки роли начинаем её настройку, в неё можно перейти по значку флага в верхней части Server Meneger, центр сертификации попросит настроить сервис, поэтому там должен появиться знак треугольника с восклицательным знаком.
```
![iamge](https://i.imgur.com/XFcmssX.png)
```
После этого нажимаем Next до момента настройки CA NAME, там в первой строчке надо будет написать следующие - Demo.wsr, после чего нажимать Next до конца.
```
```
Дальше переходим к настройки IIS
```
![image](https://i.imgur.com/FYXFxkS.png)
![image](https://i.imgur.com/UCRro7s.png)
![iamge](https://i.imgur.com/y9EyO1b.png)
![image](https://i.imgur.com/B4TUtHW.png)

### Продолжаем настройку сертификации
```
Заходим в браузер и вводим данную ссылку - https://localhost/certsrv/
```
![image](https://i.imgur.com/F2bKovB.png)
![image](https://i.imgur.com/eqlnmSx.png)
![image](https://i.imgur.com/s7YAvDi.png)
![image](https://user-images.githubusercontent.com/60313293/205651246-49b7cc7c-3d2e-43df-b977-7075c0ae743a.png)
![image](https://i.imgur.com/mzTpFR0.png)
![image](https://i.imgur.com/uMCS8dB.png)
![iamge](https://i.imgur.com/80XIi6h.png)
```
Там будет устанвка сертификата,просто нажимаем на install certificate
```
### Экспорт сертификата на smb папку
![image](https://i.imgur.com/OT9mUVN.png)
![image](https://i.imgur.com/BSUm9dW.png)
![image](https://i.imgur.com/jOrOkg2.png)
![image](https://i.imgur.com/pZFt6UE.png)
![image](https://i.imgur.com/YZ3JIN1.png)

### Настройка NTP
![image](https://i.imgur.com/MjpL55h.png)
![iamge](https://i.imgur.com/zQcIkrq.png)
![image](https://i.imgur.com/WhTRItR.png)
![image](https://i.imgur.com/sMubFKu.png)
![image](https://i.imgur.com/ZvGVTPg.png)
![image](https://user-images.githubusercontent.com/60313293/205655011-730bf3b2-2820-4f9a-a1e2-c9e262ec0d1a.png)
![image](https://user-images.githubusercontent.com/60313293/205655135-c3bf267e-1646-4a06-a757-cefb1fc4d120.png)

### CLI Полная настройка

### Настройка сети
![image](https://user-images.githubusercontent.com/60313293/205930700-875df70e-3180-4c37-8cf7-ae3349a1d25a.png)
![image](https://user-images.githubusercontent.com/60313293/205930789-73d999d2-ced6-45a1-8ef8-830a90fb018b.png)
![image](https://user-images.githubusercontent.com/60313293/205930875-c52f7564-97bb-449f-8ba3-26c1e8c92184.png)
![image](https://user-images.githubusercontent.com/60313293/205931034-acf5dadf-c963-477d-8d4c-a3aca34cbfc4.png)

### Настройка NTP
![image](https://user-images.githubusercontent.com/60313293/205929728-35451477-15df-4532-a0aa-9344dd8f50e5.png)
![image](https://user-images.githubusercontent.com/60313293/205929823-ae7f2f3c-9c8e-4348-9eca-2b680274c903.png)
![image](https://user-images.githubusercontent.com/60313293/205930030-8f27314f-3dfd-48a9-8bad-10f354e6cd7b.png)

### Получение и установка сертификата
![image](https://user-images.githubusercontent.com/60313293/205928192-d22997c7-cb72-42dd-84b9-34bba6ff0b3d.png)
```
```
![image](https://user-images.githubusercontent.com/60313293/205928988-e21f5870-c91d-40fa-a9eb-183cc3e2eb41.png)
```
```
![image](https://user-images.githubusercontent.com/60313293/205929271-87d624df-31fe-426c-a680-fc377a031fa5.png)
![image](https://user-images.githubusercontent.com/60313293/205929335-d181854b-4d37-4561-8985-08564b73985b.png)
![image](https://user-images.githubusercontent.com/60313293/205929387-8ce9e237-689f-4298-9cb5-11598c6474af.png)
![image](https://user-images.githubusercontent.com/60313293/205929447-69abd63b-322a-473f-8cfd-072178409531.png)
![image](https://user-images.githubusercontent.com/60313293/205929518-8d8d2b72-09fb-4f01-b5c6-af32c67b5bdc.png)

### Итог
```
Переходим в браузер и прописываем следующий адрес - https://www.demo.wsr
```
![image](https://user-images.githubusercontent.com/60313293/205930224-c1364ddb-15bd-4cbf-9e2d-3b8c6c16cfb0.png)
