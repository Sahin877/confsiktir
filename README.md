Добавляем 4 диск
apt-cdrom add 
nano /etc/ssh/sshd_config 
nano /etc/sysctl.conf
sysctl –p  
apt install network-manager и идем в nmtui

ISP
apt install mc bind9 openssh-server ntp ntpdate

RTR-L  
apt install mc firewalld openssh-server libreswan
touch /etc/ipsec.d/gre.conf
touch /etc/ipsec.d/gre.secrets 
systemctl enable ipsec
systemctl start ipsec 
nano /etc/systemd/timesyncd.conf

nmcli connection modify ISP connection.zone external
nmcli connection modify LEFT connection.zone trusted 
firewall-cmd --permanent --zone=external --add-service=gre 
firewall-cmd --permanent --zone=external --add-forward-port=port=53:proto=tcp:toaddr=192.168.100.200:toport=53
firewall-cmd --permanent --zone=external --add-forward-port=port=53:proto=udp:toaddr=192.168.100.200:toport=53 
firewall-cmd --permanent --zone=external --add-forward-port=port=80:proto=tcp:toaddr=192.168.100.100:toport=80
firewall-cmd --permanent --zone=external --add-forward-port=port=443:proto=tcp:toaddr=192.168.100.100:toport=443
firewall-cmd --permanent --zone=external --add-forward-port=port=2222:proto=tcp:toaddr=192.168.100.100:toport=22
firewall-cmd --permanent --zone=external --add-forward-port=port=2244:proto=tcp:toaddr=172.16.100.100:toport=22
firewall-cmd –reload

RTR-R 
apt install mc firewalld openssh-server libreswan 
touch /etc/ipsec.d/gre.conf
touch /etc/ipsec.d/gre.secrets 
systemctl enable ipsec 
systemctl start ipsec  
nano /etc/systemd/timesyncd.conf

nmcli connection modify ISP connection.zone external
nmcli connection modify RIGHT connection.zone trusted
firewall-cmd --permanent --zone=external --add-service=gre
firewall-cmd --permanent --zone=external --add-forward-port=port=80:proto=tcp:toaddr=172.16.100.100:toport=80
firewall-cmd --permanent --zone=external --add-forward-port=port=443:proto=tcp:toaddr=172.16.100.100:toport=443
firewall-cmd –reload

WEB-L  и  WEB-R
apt install mc nginx openssh-server lynx cifs-utils
nano /etc/systemd/timesyncd.conf

SRV и CLI
ip 192.168.100.200
subnet 255.255.255.0
default 192.168.100.254
preferred DNS server 192.168.100.254
ip 3.3.3.10
subnet 255.255.255.0
default 3.3.3.1
preferred DNS server 3.3.3.1
сменяем NAME у обоих
netsh adv set all state off
slmgr /rearm

SRV
УСТАНОВКА РОЛЕЙ ADD и NFS
Disk managment
настраиваем DNS (не забываем про Conditional.. "demo.wsr" и 3.3.3.1)
GPO
Edit group на CLI
gpupdate /force на SRV и CLI
CИНХРОНИЗИРУЕМ ВРЕМЯ
Выпускаем сертификацию Web

mmc
WEB-L и WEB-R
systemctl restart sshd, times...
Идем в cmd
cd \
scp web.pfx root@web-l:~ P@ssw0rd, (toor)
scp ca.cer root@web-l:~ P@ssw0rd, (toor)

WEB-L
openssl pkcs12 -in web.pfx -nokeys -out cert.pem (P@ssw0rd)
openssl pkcs12 -in web.pfx -nocerts -out key.pem (P@ssw0rd)
openssl rsa -in key.pem -out key.pem (P@ssw0rd) 
cp *. pem /etc/nginx/
scp*. pem root@web-r:/etc/nginx (toor)

WEB-L; WEB-R
nano /etc/nginx/sites-available/default 
server {
listen 80;
server_name www.demo.wsr;
return 301 https://$server_name$request_uri;
}
upstream backend {
server 192.168.100.100:5000 fail_timeout=43s;
server 172.16.100.100:5000 fail_timeout=43s;
}
server {
listen 443 ssl;
ssl_certificate /etc/nginx/cert.pem;
ssl_certificate_key /etc/nginx/key.pem;
server_name www.demo.wsr;
location / {
	proxy_pass http://backend;
}
}
systemctl restart nginx.service (WEB-L; WEB-R)

WEB-L; WEB-R
docker image load -i /opt/appdocker0.zip
docker run -d --restart always –p 5000:5000 appdocker0:latest 
