# Install DNS Server 192.168.8.102 && kelompok5proxy.com
login superuser
```
sudo su
```
allow port 53 pada server
```
apt install bind9
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
      addresses: [192.168.22.254/24]
      gateway4: 192.168.22.1
      nameservers:
        search: [aspal.com]
        addresses: [192.168.22.254, 192.168.22.1]
  version: 2
```

Konfigurasi Resolv.conf
```
nano /etc/resolv.conf
```
```
nameserver 192.168.22.254
nameserver 192.168.22.1
options edns0
search aspal.com
```

konfigurasi host
```
nano /etc/hosts
```
```
127.0.0.1 localhost
127.0.1.1 srv1
192.168.22.254  aspal.com
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
@       IN      A       192.168.22.254
@       IN      MX      10      mail.aspal.com.
ns      IN      A       192.168.22.254
www     IN      CNAME   ns
mail    IN      A       192.168.22.254
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
zone "aspal.com" {
        type master;
        file "/etc/bind/db.aspal";
};

zone "22.168.192.in-addr.arpa" {
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



### Install PHP
install php dll
```
apt install php php-fpm php-mysql libapache2-mod-php
```

jalankan php-fpm
```
systemctl start php7.4-fpm
```

```
a2enmod proxy_fcgi setenvif
```

```
systemctl restart apache2
```

```
a2enconf php7.4-fpm
```

```
systemctl reload apache2
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
touch website1.conf
```

```
nano website1.conf
```

```
<VirtualHost *:80>
        ServerName aspal.com
        ServerALias www.aspal.com
        ServerAdmin webmaster@aspal.com
        DocumentRoot /var/www/html/website1
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

```
a2ensite website1.conf
```

```
systemctl reload apache2
```




# Install Mail Server (squirrelmail)

install beberapa dependencies
```
sudo apt -y install software-properties-common
sudo apt -y install python3-software-properties
sudo apt -y install python-software-properties
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
apt -y install apache2 php php-xmlrpc php-mysql php-gd php-cli
```

```
apt -y install php-curl dovecot-common dovecot-imapd
```

```
apt -y install php5.6 php5.6-mysql php5.6-mbstring php-mbstring
```

```
apt -y install php-xdebug libapache2-mod-php5.6 libapache2-mod-php
```

```
apt -y install postfix
```

```
internet with smarthost
aspal.com
smtp.telkom.net
```

```
systemctl restart postfix
```

```
apt - install dovecot-imapd dovecot-pop3d
```

```
systemctl restart dovecot
```

```
wget https://sourceforge.net/projects/squirrelmail/files/stable/1.4.22/squirrelmail-webmail-1.4.22.zip
apt -y install unzip
unzip squirrelmail-webmail-1.4.22.zip
mv squirrelmail-webmail-1.4.22 /var/www/html/
mv /var/www/html/squirrelmail-webmail-1.4.22/ /var/www/html/squirrelmail
chown -R www-data:www-data /var/www/html/squirrelmail/
chmod 755 -R /var/www/html/squirrelmail/
```

```
sudo perl /var/www/html/squirrelmail/config/conf.pl
```

2 -> 1 -> aspal.com -> s -> q

```
sudo useradd myusername
```





# Install FTP
```
https://devanswers.co/install-ftp-server-vsftpd-ubuntu-20-04/
```