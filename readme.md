#Install WEBSERVER LEMP (Linux NginX Mysql PHP)

## Install Nginx    

update repository
```
sudo apt update
```

update repository
```
sudo apt install nginx
```

jalankan server nginx
```
sudo systemctl start nginx
```

buat firewall
```
sudo ufw enable
```

allow nginx
```
sudo ufw allow 'Nginx HTTP'
```

## Install MYSQL
```
sudo apt install mysql-server
```
```
sudo mysql_secure_installation
```

## Install PHP
install php
```
sudo apt install php
```

install php-fpm dan php-mysql

```
sudo apt install php-fpm php-mysql
```

jalankan php-fpm
```
sudo systemctl start php7.4-fpm
```

## Setting MYSQL
```
sudo mysql
```

```
create user 'server'@'localhost' identified by 'server123'
```

```
grant all privileges on *.* to 'server'@'localhost';
```

```
flush privileges
```


## Install DNS Server
```
sudo su
```

```
apt install bind9
```

##### IP : 192.168.197.168 & my-site.com

```
nano /etc/bind/named.conf.options
```

```
// forwarders {
//      192.168.197.168;
// };
```


```
nano /etc/bind/named.conf.local
```

```
zone "my-site.com" IN {
        type master;
        file "/etc/bind/db.my-site.com"
};

zone "197.168.192.in-addr.arpa" IN { 
        type master;
        file "/etc/bind/db.192";
};
```

```
cp /etc/bind/db.local /etc/bind/db.my-site.com
```

```
nano /etc/bind/db/my-site.com
```

```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     my-site.com. root.my-site.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.my-site.com.
@       IN      A       192.168.197.168
@       IN      AAAA    ::1
ns      IN      A       192.168.197.168
```

```
cp /etc/bind/db.127 /etc/bind/db.192
```

```
nano /etc/bind/db.192
```

```
; BIND reverse data file for local loopback interface
;
$TTL    604800
@       IN      SOA     ns.examplecom. root.my-site.com. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.
197     IN      PTR     ns.my-site.com.
```

```
nano /etc/resolv.conf
```

```
nameserver 127.0.0.53
options edns0 trust-ad
```

```
nameserver 192.168.197.168
options edns0         
search Home
```

nslookup 192.168.197.168




### 192.168.197.168
