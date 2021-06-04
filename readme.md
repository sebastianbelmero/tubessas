IP : 191.168.8.102
reverse : 8.168.192
link : aspal.com

email : sebastian
password : sebastian123

mysql : server
password : server123

konfigurasi apache
utama : website1.conf
email : website2.conf

ftp
username : ftpuser

IP client : 192.168.8.101

# Install DNS Server
login superuser
```
sudo su
```

```
apt install bind9
```

allow port 53 pada server
```
ufw allow 53
```

konfigurasi IP secara static
```
nano /etc/netplan/00-installer-config.yaml
```
konfigurasi interface
```
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      dhcp4: false
      addresses: [192.168.8.102/24]
      gateway4: 192.168.8.1
      nameservers:
        search: [aspal.com]
        addresses: [192.168.8.102, 192.168.8.1]
  version: 2
```

Konfigurasi Resolv.conf

```
nano /etc/resolv.conf
```

```
nameserver 192.168.8.102
nameserver 192.168.8.1
options edns0
search aspal.com
```

konfigurasi host

```
nano /etc/hosts
```

```
192.168.8.102  aspal.com
```

tambahkan zone pada primary server

```
nano /etc/bind/named.conf.local
```

```
zone "aspal.com" {
        type master;
        file "/etc/bind/db.aspal";
};
```

buat data file dengan cara copy template yang sudah ada

```
cp /etc/bind/db.local /etc/bind/db.aspal
```

```
nano /etc/bind/db.aspal
```

```
;
; BIND data file for PT.Aspal
;
$TTL    604800
@       IN      SOA     ns.aspal.com. root.aspal.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.aspal.com.
@       IN      A       192.168.8.102
@       IN      MX      10      mail.aspal.com.
ns      IN      A       192.168.8.102
www     IN      CNAME   ns
mail    IN      A       192.168.8.102
```

restart bind9

```
systemctl restart bind9
```

buat reverse zone

```
nano /etc/bind/named.conf.local
```

```
zone "8.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/db.192";
};
```

buat data file 192 dari template 127

```
cp /etc/bind/db.127 /etc/bind/db.192
```

```
nano /etc/bind/db.192
```

```
;
; BIND reverse data file for PT.Aspal
;
$TTL    604800
@       IN      SOA     ns.aspal.com. root.aspal.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.aspal.com.
1       IN      PTR     ns.aspal.com.
1       IN      PTR     www.aspal.com
1       IN      PTR     mail.aspal.com
```

```
systemctl restart bind9
```

DNS Caching berfungsi jika Client menggunakan DNS Local dan ingin terhubung dengan Internet. jadi PC Client masih bisa terhubung ke Internet meskipun Client menggunakan DNS Local.

```
nano /etc/bind/named.conf.options
```

```
forwarders {
        8.8.8.8;
        8.8.4.4;
};
```

```
systemctl restart bind9
```

tes

```
nslookup www.aspal.com
```

```
ping aspal.com
```


# Install WEBSERVER LEMP (Linux NginX Mysql PHP)
### Install Apache    
allow apache
```
sudo ufw allow in "Apache"
sudo ufw allow in "Apache Full"
```

### Install MYSQL
```
sudo apt install mysql-server
```
```
systemctl start mysql
```
```
sudo mysql_secure_installation
```
```
y -> 0 -> y -> y -> y -> y -> y
```

### Setting MYSQL
```
mysql
```

```
create user 'server'@'localhost' identified by 'server123';
```

```
grant all privileges on *.* to 'server'@'localhost';
```

```
flush privileges;
```

```
exit
```

```
mysql -u server -p
```
password : server123


coba buat database, tablenya dan isinya
```
create database contoh;
use contoh;
create table contohnya(
id int primary key auto_increment,
isi varchar(100)
);
insert into contohnya (isi) values ('satu'),('dua'),('tiga'),('empat'),('lima');
exit;
```

tes apakah php sudah bisa mengakses mysql
```
cd /var/www/html
```

```
touch tes.php
```

```
nano tes.php
```

```
<?php
$koneksi = mysqli_connect("localhost", "server", "server123", "contoh");
$ambil = $koneksi -> query("select * from contohnya");
while($pecah = $ambil -> fetch_assoc()){
        print_r($pecah);
}
```

### Virtual host pada Apache
```
cd /var/www/html/
```

```
wget https://github.com/BlackrockDigital/startbootstrap-landing-page/archive/gh-pages.zip
```

```
unzip gh-pages.zip
```

```
mv startbootstrap-landing-page-gh-pages/ website1
```

```
cd /etc/apache2/sites-available/
```
```
touch website1.conf website2.conf
```

```
nano website1.conf
```

```
<VirtualHost *:80>
        ServerName aspal.com
        ServerALias www.aspal.com
        ServerAdmin webmaster@aspal.com
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

```
nano website2.conf
```

```
<VirtualHost *:80>
        ServerName aspal.com
        ServerALias mail.aspal.com
        ServerAdmin webmaster@aspal.com
        DocumentRoot /var/www/html/squirrelmail
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

```
a2ensite website1.conf
a2ensite website2.conf
```

```
systemctl reload apache2
```




# Install Mail Server (squirrelmail)

install beberapa dependencies
```
sudo apt -y install software-properties-common
sudo apt -y install python3-software-properties
```

tambahkan repo PHP
```
sudo add-apt-repository ppa:ondrej/php
```

update repo
```
sudo apt-get update
```

```
sudo locale-gen id_ID.UTF-8
```

```
apt -y install apache2 php php-xmlrpc php-mysql php-gd php-cli \
php-curl dovecot-common dovecot-imapd \
dovecot-pop3d postfix php-mbstring \
php-xdebug libapache2-mod-php unzip
```

```
internet with smarthost
aspal.com
smtp.telkom.net
```

```
cd /usr/local/src
git clone https://github.com/RealityRipple/squirrelmail
unzip squirrelmail-webmail-1.4.22.zip
mv squirrelmail-webmail-1.4.22 /var/www/html/squirrelmail/
mkdir -p /var/local/squirrelmail/data
chown -Rf www-data: /var/local/squirrelmail/
chmod -Rf 777 /var/local/squirrelmail/
```

```
cd /var/www/html/squirrelmail/
```

```
./configure
```

2 -> 1 -> aspal.com -> s -> q

```
service dovecot restart
/etc/init.d/apache2 restart
/etc/init.d/postfix restart
```

menambahkan email

```
useradd myusername
```

```
passwd myusername
```

```
mkdir -p /var/www/html/myusername
usermod -m -d /var/www/html/myusername myusername
```

```
chown -R myusername:myusername /var/www/html/myusername
```



# Install FTP
```
https://devanswers.co/install-ftp-server-vsftpd-ubuntu-20-04/
```

```
apt install vsftpd
```

```
systemctl start vsftpd
```

```
ufw allow 20/tcp
ufw allow 21/tcp
ufw allow 40000:50000/tcp
ufw allow 990/tcp
```

```
adduser ftpuser
```

```
nano /etc/ssh/sshd_config
```

```
DenyUsers ftpuser
```

```
systemctl restart sshd
```

### Upload ke Web Server

```
usermod -d /var/www ftpuser
chown ftpuser:ftpuser /var/www/html
```

### Uplaod ke Folder Home
```
mkdir /home/ftpuser/ftp
chown nobody:nogroup /home/ftpuser/ftp
chmod a-w /home/ftpuser/ftp
mkdir /home/ftpuser/ftp/files
chown ftpuser:ftpuser /home/ftpuser/ftp/files
```

### Konfigurasi vsftpd
```
mv /etc/vsftpd.conf /etc/vsftpd.conf.bak
```

```
nano /etc/vsftpd.conf
```

```
listen=NO
listen_ipv6=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
chroot_local_user=YES
secure_chroot_dir=/var/run/vsftpd/empty
pam_service_name=vsftpd
force_dot_files=YES
pasv_min_port=40000
pasv_max_port=50000
```

```
user_sub_token=$USER
local_root=/home/$USER/ftp
```

```
systemctl restart vsftpd
```


# Proxy Server

install squid

```
apt -y install squid
```

```
touch /etc/squid/social.network
nano /etc/squid/social.network
```

masukkan link yang akan di blokir
.usu.ac.id

```
/etc/squid/squid.conf
```

cari http_access deny all

```
acl mypc src 192.168.8.101
acl badurl dstdomain "/etc/squid/social.network"
acl pagi time MTWHF 08:30-12:00
http_access deny badurl pagi
http_access allow mypc
```

```
systemctl restart squid
```
