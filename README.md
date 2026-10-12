# Isolir Tanpa WebProxy Menggunakan L2TP
**TOPOLOGI**

Ip PPPoE 10.10.10.2-10.10.10.254

Ip Isolir 20.30.40.2-20.30.40.254

Ip lokal L2TP server Isolir 192.168.42.1

Ip Public Isolir 103.xxx.xxx.xxx




**MIKROTIK**
```
/ip ipsec proposal
set [ find default=yes ] auth-algorithms=sha256,sha1 enc-algorithms=\
    aes-256-cbc,aes-128-cbc pfs-group=none	
/ip pool
add name=pool_pppoe ranges=10.10.10.2-10.10.10.254
add name=pool_isolir ranges=20.30.40.2-20.30.40.254
/ppp profile
add local-address=10.10.10.1 name=20Mbps_Bandung rate-limit=20M/20M \
    remote-address=pool_pppoe
add local-address=20.30.40.1 name=ISOLIR rate-limit=1M/1M remote-address=\
    pool_isolir	
/interface pppoe-server server
/ip dns
set servers=8.8.8.8,8.8.4.4
/ip firewall address-list
add address=20.30.40.2-20.30.40.254 comment="IP LOKAL ISOLIR" list=ISOLIR
/ip firewall filter
add action=accept chain=forward comment="ISOLIR - ALLOW PORTAL HTTPS" \
    dst-address=103.xxx.xxx.xxx dst-port=443 protocol=tcp src-address-list=\
    ISOLIR
add action=drop chain=forward comment="ISOLIR - BLOCK INTERNET" \
    out-interface=ether1 src-address-list=ISOLIR
/ip firewall nat
add action=dst-nat chain=dstnat comment=ISOLIR-DNS-UDP dst-port=53 protocol=\
    udp src-address-list=ISOLIR to-addresses=192.168.42.1 to-ports=53
add action=dst-nat chain=dstnat comment=ISOLIR-DNS-TCP dst-port=53 protocol=\
    tcp src-address-list=ISOLIR to-addresses=192.168.42.1 to-ports=53
add action=dst-nat chain=dstnat comment=ISOLIR-HTTP dst-port=80 protocol=tcp \
    src-address-list=ISOLIR to-addresses=192.168.42.1 to-ports=80
add action=masquerade chain=srcnat out-interface=ether1
add action=masquerade chain=srcnat comment="ISOLIR RETURN VIA L2TP" \
    dst-address=192.168.42.1 out-interface=L2TP_ISOLIR src-address-list=\
    ISOLIR
add action=masquerade chain=srcnat disabled=yes out-interface=ether3

/ppp secret
add name=soreang001 password=12345678 profile=20Mbps_Soreang service=pppoe
/system clock
set time-zone-name=Asia/Jakarta
/system ntp client
set enabled=yes primary-ntp=202.162.32.12 secondary-ntp=185.125.190.58
```

**Rubah ip public di**

```
nano /etc/dnsmasq.conf
```
> listen-address=103.xxx.xxx.xxx
> 
> address=/#/103.xxx.xxx.xxx

**buat subdomain arahkan ke ip public misal: isolir.domainkamu.com**


**Setting nginx**
```
mkdir /var/www/html/isolir.domainkamu.com
sudo chown -R $USER:$USER /var/www/isolir.domainkamu.com/html
sudo chmod -R 755 /var/www/isolir.jkbtel.com
nano /etc/nginx/sites-available/isolir.domainkamu.com
```

```
server {

        root /var/www/isolir.domainkamu.com/html;
        index index.html index.htm index.nginx-debian.html;

        server_name isolir.domainkamu.com www.isolir.domainkamu.com;

        location / {
                try_files $uri $uri/ =404;
        }
}
```
UBAH return subdomain kamu
```
nano /etc//etc/nginx/sites-available/default
```
> return 302 https://isolir.subdomainkamu.com/;

```
sudo ln -s /etc/nginx/sites-available/isolir.domainkamu.com /etc/nginx/sites-enabled/
sudo systemctl restart nginx
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d isolir.domainkamu.com
```


**UBAH IPsec**
```
nano /etc/ipsec.secrets
```
ganti 12345678 dengan IPsec yang anda inginkan


**Menambahkan uservpn baru**
```
cd /root
bash add_vpn_user.sh 'username' 'password'
```

**login ssh**
```
root
12345678
```
