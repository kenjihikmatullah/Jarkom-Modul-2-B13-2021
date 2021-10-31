# Jarkom-Modul-2-B13-2021      

**Members**
- Abdulatif Fajar Sidiq (05111840007002)
- Kenji Hikmatullah (05111840000074)   
      
#### Pembukaan         
Luffy adalah seorang yang akan jadi Raja Bajak Laut. Demi membuat Luffy menjadi Raja Bajak Laut, Nami ingin membuat sebuah peta, bantu Nami untuk membuat peta berikut:    
![Contoh](/screenshots/topologi-contoh.png)          

EniesLobby akan dijadikan sebagai DNS Master, Water7 akan dijadikan DNS Slave, dan Skypie akan digunakan sebagai Web Server. Terdapat 2 Client yaitu Loguetown, dan Alabasta. Semua node terhubung pada router Foosha, sehingga dapat mengakses internet      
         
#### 1. EniesLobby akan dijadikan sebagai DNS Master, Water7 akan dijadikan DNS Slave, dan Skypie akan digunakan sebagai Web Server. Terdapat 2 Client yaitu Loguetown, dan Alabasta. Semua node terhubung pada router Foosha, sehingga dapat mengakses internet       
    
Dibuat topologi terlebih dahulu sebagai berikut:       
![Topologi](/screenshots/topologi.png)           
Kemudian dilakukan konfigurasi pada setiap node yang ada:       
       
**Foosha sebagai Router**         
![Foosha](/screenshots/network-config/foosha.png)         
       
**Loguetown sebagai Client**  
```
apt-get update         
apt-get install dnsutils  
```
![Loguetown](/screenshots/network-config/loguetown.png)  

**Alabasta sebagai Client**  
![Alabasta](/screenshots/network-config/alabasta.png)  

**Enieslobby Sebagai DNS Master**  
```
apt-get update  
apt-get install bind9 -y  
```
![Enieslobby](/screenshots/network-config/enies-lobby.png)  
  
**Water7 sebagai DNS Slave**  
```
apt-get update  
apt-get install bind9 -y       
echo "nameserver 192.168.122.1" > /etc/resolv.conf       
```        
![Water7](/screenshots/network-config/water-7.png)           
            
**Skypie Sebagai Webserver**           
![Skypie](/screenshots/network-config/skypie.png)        
          
Kemudian setiap node diaktifkan dan command `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.183.0.0/16` dijalankan pada router Foosha agar dapat terhubung ke internet.         

#### 2. Luffy ingin menghubungi Franky yang berada di EniesLobby dengan denden mushi. Kalian diminta Luffy untuk membuat website utama dengan mengakses `franky.yyy.com` dengan alias `www.franky.yyy.com` pada folder kaizoku  

**Server EniesLobby**  
Dilakukan konfigurasi terhadap file  `/etc/bind/named.conf.local` dengan menambahkan  
```
zone "franky.b13.com" {  
        type master;  
        file "/etc/bind/kaizoku/franky.b13.com";
};
```  
Dibuat sebuah direktori baru yaitu `/etc/bind/kaizoku`  
dan dilanjutkan dengan konfigurasi pada `/etc/bind/kaizoku/franky.b13.com`  
```
$TTL    604800  
@       IN      SOA     franky.b13.com. root.franky.b13.com. (
                        2021100401      ; Serial
                        604800          ; Refresh
                        86400           ; Retry
                        2419200         ; Expire
                        604800 )        ; Negative Cache TTL
;
@               IN      NS      franky.b13.com.
@               IN      A       192.183.2.2 ; IP EniesLobby
www             IN      CNAME   franky.b13.com.

```
Dilakukan restart service bind9 dengan `service bind9 restart`  

**Server Loguetown**  
```
apt-get update  
apt-get install dnsutils -y  
echo "nameserver 192.183.2.2" > /etc/resolv.conf  
```  

![](https://github.com/BryanYehuda/Jarkom-Modul-2-T7-2021/blob/main/image/testing-1-2-testing.png?raw=true)  

#### 3. Setelah itu buat subdomain `super.franky.yyy.com` dengan alias `www.super.franky.yyy.com` yang diatur DNS nya di EniesLobby dan mengarah ke Skypie(3).  
  
**Server EniesLobby**  
Dilakukan pembaruan pada file `/etc/bind/kaizoku/franky.b13.com` menjadi seperti berikut:  
```  
$TTL    604800  
@       IN      SOA     franky.b13.com. root.franky.b13.com. (  
                        2021100401      ; Serial
                        604800          ; Refresh
                        86400           ; Retry
                        2419200         ; Expire
                        604800 )        ; Negative Cache TTL
;
@               IN      NS      franky.b13.com.
@               IN      A       192.183.2.2 ; IP EniesLobby
www             IN      CNAME   franky.b13.com.
super           IN      A       192.183.2.4 ; IP skype
www.super       IN      CNAME   super.franky.b13.com.
```
bind9 direstart dengan `service bind9 restart`  

#### 4. Buat juga reverse domain untuk domain utama  
  
**Server EsniesLobby**  
File `/etc/bind/named.conf.local` diperbarui menjadi:  
```  
zone "franky.b13.com" {  
        type master;  
        file "/etc/bind/kaizoku/franky.b13.com";  
};

zone "2.183.192.in-addr.arpa" {
        type master;
        file "/etc/bind/kaizoku/2.183.192.in-addr.arpa";
};
```  
dan dilakukan konfigurasi pada file `/etc/bind/kaizoku/2.183.192.in-addr.arpa` seperti berikut:  
```  
$TTL    604800  
@       IN      SOA     franky.b13.com. root.franky.b13.com. (
                        2021100401      ; Serial
                        604800          ; Refresh
                        86400         ; Retry
                        2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
2.183.192.in-addr.arpa.   IN      NS      franky.b13.com.
2                       IN      PTR     franky.b13.com.
```  
#### 5. Supaya tetap bisa menghubungi Franky jika server EniesLobby rusak, maka buat Water7 sebagai DNS Slave untuk domain utama  
  
**Service Water7**  
Dilakukan konfigurasi pada file `/etc/bind/named.conf.local` menjadikan node Water7 sebagai DNS Slave:  
```  
zone "franky.b13.com" {  
        type master;
        notify yes;
        also-notify {192.183.2.3;};  //Masukan IP Water7 tanpa tanda petik
        allow-transfer {192.183.2.3;}; // Masukan IP Water7 tanpa tanda petik
        file "/etc/bind/kaizoku/franky.b13.com";
};
  
zone "2.183.192.in-addr.arpa" {
        type master;
        file "/etc/bind/kaizoku/2.183.192.in-addr.arpa";
};
```  
bind9 direstart dengan `service bind9 restart`  
  
**Server Water7**  
perintah `apt-get update` dan `apt-get install bind9 -y` dipanggil pada note Water7 untuk menginstall package yang diperlukan sebagai DNS Slave.   
  
Dilakukan konfigurasi pada file `/etc/bind/named.conf.local `  

```
zone "franky.b13.com" {
        type slave;
        masters { 192.183.2.2; }; // Masukan IP EniesLobby tanpa tanda petik
        file "/var/lib/bind/franky.b13.com";
};
```  
bind9 direstart dengan `service bind9 restart`  

#### 7. Setelah itu terdapat subdomain `mecha.franky.yyy.com` dengan alias `www.mecha.franky.yyy.com` yang didelegasikan dari EniesLobby ke Water7 dengan IP menuju ke Skypie dalam folder sunnygo  
  
**Server Enieslobby**  
Dilakukan konfigurasi `/etc/bind/kaizoku/franky.b13.com`  
```
$TTL    604800
@       IN      SOA     franky.b13.com. root.franky.b13.com. (
                        2021100401      ; Serial
                        604800          ; Refresh
                        86400           ; Retry
                        2419200         ; Expire
                        604800 )        ; Negative Cache TTL
;
@               IN      NS      franky.b13.com.
@               IN      A       192.183.2.4 ; IP skypea
www             IN      CNAME   franky.b13.com.
super           IN      A       192.183.2.4 ; IP skypea
www.super       IN      CNAME   super.franky.b13.com.
ns1             IN      A       192.183.2.3; IP Water7
mecha           IN      NS      ns1
```  
Kemudian file `/etc/bind/named.conf.options` diperbarui dengan meng-comment `dnssec-validation auto;` dan menambahkan baris berikut pada `/etc/bind/named.conf.options`  
```  
allow-query{any;};  
```  
Kemudian file `/etc/bind/named.conf.local` diperbarui menjadi seperti  
```  
zone "franky.b13.com" {
        type master;
        //notify yes;
        //also-notify {192.183.2.3;};  Masukan IP Water7 tanpa tanda petik
        allow-transfer {192.183.2.3;}; // Masukan IP Water7 tanpa tanda petik
        file "/etc/bind/kaizoku/franky.b13.com";
};

zone "2.183.192.in-addr.arpa" {
        type master;
        file "/etc/bind/kaizoku/2.183.192.in-addr.arpa";
};
```
bind9 direstart dengan `service bind9 restart`  
  
**Server Water7**  
file `/etc/bind/named.conf.options` diperbarui dengan meng-comment `dnssec-validation auto;` dan menambahkan baris berikut pada `/etc/bind/named.conf.options` 
```  
allow-query{any;};  
```  
kemudian file `/etc/bind/named.conf.local` diedit untuk delegasi `mecha.franky.b13.com`  
```  
zone "franky.b13.com" {
    type slave;
    masters { 192.183.2.2; }; // Masukan IP EniesLobby tanpa tanda petik
    file "/var/lib/bind/franky.b13.com";
};  
  
zone "mecha.franky.b13.com"{  
        type master;
        file "/etc/bind/sunnygo/mecha.franky.b13.com";
};  
```  
Lalu dibuat direktori `mkdir /etc/bind/sunnygo` dan dilakukan konfigurasi pada file `/etc/bind/sunnygo/mecha.franky.b13.com`
```  
$TTL    604800
@       IN      SOA     mecha.franky.b13.com. root.mecha.franky.b13.com. (
                        2021100401      ; Serial
                        604800         ; Refresh
                        86400         ; Retry
                        2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
@               IN      NS      mecha.franky.b13.com.
@               IN      A       192.183.2.4       ;ip skypie
www             IN      CNAME   mecha.franky.b13.com.
```  
bind9 direstart dengan `service bind9 restart`

#### 7. Untuk memperlancar komunikasi Luffy dan rekannya, dibuatkan subdomain melalui Water7 dengan nama `general.mecha.franky.yyy.com` dengan alias `www.general.mecha.franky.yyy.com` yang mengarah ke Skypie  
  
**Server Water7**  
file konfigurasi `/etc/bind/sunnygo/mecha.franky.b13.com` dibuat sebagai berikut  
```
$TTL    604800
@       IN      SOA     mecha.franky.b13.com. root.mecha.franky.b13.com. (
                        2021100401      ; Serial
                        604800         ; Refresh
                        86400         ; Retry
                        2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
@               IN      NS      mecha.franky.b13.com.
@               IN      A       192.183.2.4       ;ip skypie
www             IN      CNAME   mecha.franky.b13.com.
general         IN      A       192.183.2.4       ;IP skypie
www.general     IN      CNAME   mecha.franky.b13.com.
```
bind9 direstart dengan `service bind9 restart`

#### 8. Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver `www.franky.yyy.com.` Pertama, luffy membutuhkan webserver dengan DocumentRoot pada `/var/www/franky.yyy.com.`  
    
**Client Loguetown**   
Perinah berikut dipanggi untuk menginstall browser berbasis command-line    
```
apt-get update
apt-get install lynx -y
```
  
**Server Skypie**      
Dilakukan instalasi Apache, php, dan openssl untuk melakukan download ke website https dengan cara
```
apt-get install apache2 -y
service apache2 start
apt-get install php -y
apt-get install libapache2-mod-php7.0 -y
service apache2 
apt-get install ca-certificates openssl -y
```
dilakukan konfigurasi menggunakan file `/etc/apache2/sites-available/franky.b13.com.conf`. DcumentRoot diletakkan  di /var/www/franky.b13.com. Ditambahkan juga ServerName dan ServerAlias  
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/franky.b13.com
        ServerName franky.b13.com
        ServerAlias www.franky.b13.com

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Lalu dibuat sebuah direktori baru bernama franky.b13.com pada direktory /var/www
```
mkdir /var/www/franky.b13.com
```

setelah mengisi direktori tersebut dengan konten, webserver direstart dengan `service apache2 restart`
  
#### 9. Setelah itu, Luffy juga membutuhkan agar url `www.franky.yyy.com/index.php/home` dapat menjadi menjadi `www.franky.yyy.com/home.`

**Server Skypie**     
konfigurasi file `/var/www/franky.b13.com/.htaccess` dengan    
```
a2enmod rewrite
service apache2 restart
echo "
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule (.*) /index.php/\$1 [L]
```
Dengan file tersebut, kita bisa mengakses suatu halaman pada website tersebut tanpa harus menambahkan suffix .php

file konfigurasi `/etc/apache2/sites-available/franky.b13.com.conf` diperbarui menjadi  
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/franky.b13.com
        ServerName franky.b13.com
        ServerAlias www.franky.b13.com

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/franky.b13.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>
```
service apache2 direstart       dengan `service apache2 restart`