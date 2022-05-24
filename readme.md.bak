IP : 192.168.81.133
reverse : 81.168.192
link : sebastian.com
gateway : 192.168.81.1

email : server
password : server123

mysql : server
password : server123

konfigurasi apache
utama : website.conf
email : squirrelmail.conf

ftp
username : ftpuser

IP client : 192.168.81.67

ANGGI YOHANES PARDEDE 191402143
DANIEL SITUMEANG 191402140
GEYLFEDRA MATTHEW PANGGABEAN 191402065
MEILY BENEDICTA HAREFA 191402053
SEBASTIAN BELMERO SITORUS 191402113

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
      addresses: [192.168.81.133/24]
      gateway4: 192.168.81.1
      nameservers:
        search: [sebastian.com]
        addresses: [192.168.81.133, 192.168.81.1]
  version: 2
```

Konfigurasi Resolv.conf

```
nano /etc/resolv.conf
```

```
nameserver 192.168.81.133
nameserver 192.168.81.1
options edns0
search sebastian.com
```

konfigurasi host

```
nano /etc/hosts
```

```
192.168.81.133  sebastian.com
```

tambahkan zone pada primary server

```
nano /etc/bind/named.conf.local
```

```
zone "sebastian.com" {
        type master;
        file "/etc/bind/db.sebastian";
};
```

buat data file dengan cara copy template yang sudah ada

```
cp /etc/bind/db.local /etc/bind/db.sebastian
```

```
nano /etc/bind/db.sebastian
```

```
;
; BIND data file for PT.sebastian
;
$TTL    604800
@       IN      SOA     ns.sebastian.com. root.sebastian.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.sebastian.com.
@       IN      A       192.168.81.133
@       IN      MX      10      mail.sebastian.com.
ns      IN      A       192.168.81.133
www     IN      CNAME   ns
mail    IN      A       192.168.81.133
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
zone "81.168.192.in-addr.arpa" {
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
; BIND reverse data file for PT.sebastian
;
$TTL    604800
@       IN      SOA     ns.sebastian.com. root.sebastian.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.sebastian.com.
1       IN      PTR     ns.sebastian.com.
1       IN      PTR     www.sebastian.com
1       IN      PTR     mail.sebastian.com
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
nslookup www.sebastian.com
```

```
ping sebastian.com
```

# Install Mail Server (squirrelmail)

install beberapa dependencies
```
apt -y install ca-certificates software-properties-common python3-software-properties
```

tambahkan repo PHP
```
add-apt-repository ppa:ondrej/php
```

update repo
```
apt-get update
```

```
locale-gen id_ID.UTF-8
```

```
apt -y install apache2 php php-xmlrpc php-mysql php-gd php-cli \
php-curl dovecot-common dovecot-imapd \
dovecot-pop3d postfix php-mbstring \
php-xdebug libapache2-mod-php unzip
```

```
internet with smarthost
sebastian.com
smtp.telkom.net
```

```
cd /usr/local/src
git clone https://github.com/RealityRipple/squirrelmail
mv squirrelmail /var/www/html/squirrelmail/
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

2 -> 1 -> sebastian.com -> s -> q

```
service dovecot restart
/etc/init.d/apache2 restart
/etc/init.d/postfix restart
```

menambahkan email

```
useradd anggi
useradd daniel
useradd geyl
useradd meily
useradd sebastian
```

```
passwd anggi
```

```
passwd daniel
```

```
passwd geyl
```

```
passwd meily
```

```
passwd sebastian
```

```
mkdir -p /var/www/html/anggi
mkdir -p /var/www/html/daniel
mkdir -p /var/www/html/geyl
mkdir -p /var/www/html/meily
mkdir -p /var/www/html/sebastian
usermod -m -d /var/www/html/anggi anggi
usermod -m -d /var/www/html/daniel daniel
usermod -m -d /var/www/html/geyl geyl
usermod -m -d /var/www/html/meily meily
usermod -m -d /var/www/html/sebastian sebastian
```

```
chown -R anggi:anggi /var/www/html/anggi
chown -R daniel:daniel /var/www/html/daniel
chown -R geyl:geyl /var/www/html/geyl
chown -R meily:meily /var/www/html/meily
chown -R sebastian:sebastian /var/www/html/sebastian
```

emailnya
anggi@sebastian.com;
daniel@sebastian.com;
geyl@sebastian.com;
meily@sebastian.com;
sebastian@sebastian.com;


# Install LAMP (Linux Apache Mysql PHP)
### Install Apache    

allow apache
```
ufw allow in "Apache"
ufw allow in "Apache Full"
```

### Install MYSQL
```
apt install mysql-server
```
```
systemctl start mysql
```
```
mysql_secure_installation
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
create database db_mahasiswa;
use db_mahasiswa;
create table mahasiswa(
id int primary key auto_increment,
nama varchar(100),
nim varchar(9),
email varchar(100)
);
insert into mahasiswa (nama, nim, email) values
('Anggi Yohanes Pardede', '191402143', 'anggiyohanespdd@gmail.com'),
('Daniel Situmeang', '191402140', 'dsitumeang47@gmail.com'),
('Geylfedra Matthew Panggabean', '191402065', 'geylrillas@gmail.com'),
('Meily Benedicta', '191402053', 'meilybenedicta2001@gmail.com'),
('Sebastian Belmero Sitorus', '191402113', 'sebastian.belmero.1@gmail.com');
exit;
```

### Virtual host pada Apache

```
cd /etc/apache2/sites-available/
```
```
touch website.conf squirrelmail.conf
```

```
nano website.conf
```

```
<VirtualHost *:80>
        ServerName sebastian.com
        ServerALias www.sebastian.com
        ServerAdmin webmaster@sebastian.com
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

```
nano squirrelmail.conf
```

```
<VirtualHost *:80>
        ServerName sebastian.com
        ServerALias mail.sebastian.com
        ServerAdmin webmaster@sebastian.com
        DocumentRoot /var/www/html/squirrelmail
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

```
a2ensite website.conf
a2ensite squirrelmail.conf
```

```
systemctl reload apache2
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

### Upload ke Folder Home
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
nano /etc/squid/squid.conf
```

cari http_access deny all

```
acl mypc src 192.168.81.67
acl badurl dstdomain "/etc/squid/social.network"
acl pagi time MTWHF 08:30-12:00
http_access deny badurl pagi
http_access allow mypc
```

```
systemctl restart squid
```
