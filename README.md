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
![Topologi](https://user-images.githubusercontent.com/61305028/139605398-b66d9c34-b269-4948-a6d2-371047524e9d.PNG)
  
Kemudian dilakukan konfigurasi pada setiap node yang ada:       
       
**Foosha sebagai Router**         
![Foosha](https://user-images.githubusercontent.com/61305028/139605408-e6454259-ac20-4397-b7a8-67365d1feb5f.PNG)
       
**Loguetown sebagai Client**  
```
apt-get update         
apt-get install dnsutils  
```
![Loguetown](https://user-images.githubusercontent.com/61305028/139605416-20c03cc4-afe7-492f-903e-1291690c2c14.PNG)

**Alabasta sebagai Client**  
![Alabasta](https://user-images.githubusercontent.com/61305028/139605426-7cb1f839-a909-4a6a-af3b-38d8e45dee7b.PNG)

**Enieslobby Sebagai DNS Master**  
```
apt-get update  
apt-get install bind9 -y  
```
![Enieslobby](https://user-images.githubusercontent.com/61305028/139605463-dcebf7f7-021a-46e0-b5d2-cdcf63f9ef10.PNG)
  
**Water7 sebagai DNS Slave**  
```
apt-get update  
apt-get install bind9 -y       
echo "nameserver 192.168.122.1" > /etc/resolv.conf       
```        
![Water7](https://user-images.githubusercontent.com/61305028/139605486-d8953a0f-f859-4e5d-bf54-1d49f942249c.PNG)
            
**Skypie Sebagai Webserver**           
![Skypie](https://user-images.githubusercontent.com/61305028/139605492-c655e641-a0c8-40a0-a988-bdbc7d98f5fe.PNG)
          
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

### 10.Setelah itu, pada subdomain `www.super.franky.yyy.com`, Luffy membutuhkan penyimpanan aset yang memiliki DocumentRoot pada `/var/www/super.franky.yyy.com`

**Server Skypie**

konfigurasi file `/etc/apache2/sites-available/super.franky.b13.com.conf` dengan
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.b13.com
        ServerName super.franky.b13.com
        ServerAlias www.super.franky.b13.com

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/franky.b13.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>

```
kemudian aktifkan virtualhost dengan a2ensite, membuat direktori untuk documentroot di `/var/www/super.franky.b13.com` dan untuk melakukan copy content ke documentroot dengan cara
```
a2ensite super.franky.b13.com
mkdir /var/www/super.franky.13.com
cp -r /root/Praktikum-Modul-2-Jarkom/super.franky/. /var/www/super.franky.b13.com
service apache2 restart

```
konfigurasi file `/var/www/super.franky.t07.com/index.php` dengan `echo "<?php echo 'yok lanjut' ?>"`

### 11.Akan tetapi, pada folder `/public`, Luffy ingin hanya dapat melakukan directory listing saja

**Server Skypie**

konfigurasi file `/etc/apache2/sites-available/super.franky.b13.com.conf` menamahkan Options +Indexes ke direktori yang ingin di directory list dengan
```

<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.b13.com
        ServerName super.franky.b13.com
        ServerAlias www.super.franky.b13.com

        <Directory /var/www/super.franky.b13.com/public>
                Options +Indexes
        </Directory>

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/franky.b13.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>

```
Melakukan restart service apache2 dengan `service apache2 restart`

### 12.Tidak hanya itu, Luffy juga menyiapkan error file `404.html` pada folder `/error` untuk mengganti error kode pada apache

**Server Skypie**

konfigurasi file `/etc/apache2/sites-available/super.franky.b13.com.conf` menambahkan konfigurasi ErrorDocumentuntuk setiap error yang ada yang diarahkan ke file `/error/404.html` dengan
```

<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.b13.com
        ServerName super.franky.b13.com
        ServerAlias www.super.franky.b13.com

        ErrorDocument 404 /error/404.html
        ErrorDocument 500 /error/404.html
        ErrorDocument 502 /error/404.html
        ErrorDocument 503 /error/404.html
        ErrorDocument 504 /error/404.html

        <Directory /var/www/super.franky.b13.com/public>
                Options +Indexes
        </Directory>

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/franky.b13.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>

```
Melakukan restart service apache2 dengan `service apache2 restart`

### 13.Luffy juga meminta Nami untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset `www.super.franky.yyy.com/public/js` menjadi `www.super.franky.yyy.com/js`

**Server Skypie**

konfigurasi file `/etc/apache2/sites-available/super.franky.b13.com.conf` menambahkan konfigurasi Alias dengan
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.b13.com
        ServerName super.franky.b13.com
        ServerAlias www.super.franky.b13.com

        ErrorDocument 404 /error/404.html
        ErrorDocument 500 /error/404.html
        ErrorDocument 502 /error/404.html
        ErrorDocument 503 /error/404.html
        ErrorDocument 504 /error/404.html

        <Directory /var/www/super.franky.b13.com/public>
                Options +Indexes
        </Directory>

        Alias \"/js\" \"/var/www/super.franky.b13.com/public/js\"


        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/franky.b13.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>

```
Melakukan restart service apache2 dengan `service apache2 restart`

### 14.Dan Luffy meminta untuk web `www.general.mecha.franky.yyy.com` hanya bisa diakses dengan port 15000 dan port 15500

**Server Skypie**

konfigurasi `file /etc/apache2/sites-available/general.mecha.franky.b13.com.conf` disini menambahkan CirtualHost baru yang berada pada port 15000 dan 15500 dengan
```
<VirtualHost *:15000>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/general.mecha.franky.b13.com
        ServerName general.mecha.franky.b13.com
        ServerAlias www.general.mecha.franky.b13.com


        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
<VirtualHost *:15500>        
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/general.mecha.franky.b13.com
        ServerName general.mecha.franky.b13.com
        ServerAlias www.general.mecha.franky.b13.com
        

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
Lalu lakukan
```
a2ensite general.mecha.franky.b13.com
service apache2 restart
mkdir /var/www/general.mecha.franky.b13.com
cp -r /root/Praktikum-Modul-2-Jarkom/general.mecha.franky/. /var/www/general.mecha.franky.b13.com/

```
konfigurasi file `/var/www/general.mecha.franky.b13.com/index.php` dengan
```
<?php
    echo 'okee 14';
?>

```
konfigurasi file `/etc/apache2/ports.conf` menambahkan Listen 15000 dan 15500 dengan
```
# If you just change the port or add more ports here, you will likely also
# have to change the VirtualHost statement in
# /etc/apache2/sites-enabled/000-default.conf

Listen 80
Listen 15000
Listen 15500
<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>

```
Melakukan restart service apache2 dengan `service apache2 restart`

### 15. dengan authentikasi username luffy dan password onepiece dan file di `/var/www/general.mecha.franky.yyy`

**Server Skypie**

Jalankan Command `htpasswd -c -b /etc/apache2/.htpasswd luffy onepiece`
konfigurasi file `/etc/apache2/sites-available/general.mecha.franky.b13.com.conf` dengan
```
<VirtualHost *:15000>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/general.mecha.franky.b13.com
        ServerName general.mecha.franky.b13.com
        ServerAlias www.general.mecha.franky.b13.com

        <Directory \"/var/www/general.mecha.franky.b13.com\">
                AuthType Basic
                AuthName \"Restricted Content\"
                AuthUserFile /etc/apache2/.htpasswd
                Require valid-user
        </Directory>

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
<VirtualHost *:15500>        
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/general.mecha.franky.b13.com
        ServerName general.mecha.franky.b13.com
        ServerAlias www.general.mecha.franky.b13.com
        
        <Directory \"/var/www/general.mecha.franky.b13.com\">
                AuthType Basic
                AuthName \"Restricted Content\"
                AuthUserFile /etc/apache2/.htpasswd
                Require valid-user
        </Directory>
        
        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
Melakukan restart service apache2 dengan `service apache2 restart`

### 16.Dan setiap kali mengakses IP Skypie akan diahlikan secara otomatis ke `www.franky.yyy.com` 

**Server Skypie**

konfigurasi file `/etc/apache2/sites-available/000-default.conf` dengan
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        RewriteEngine On
        RewriteCond %{HTTP_HOST} !^franky.b13.com$
        RewriteRule /.* http://franky.b13.com/ [R]

        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

```
Melakukan restart service apache2 dengan `service apache2 restart`

### 17.Dikarenakan Franky juga ingin mengajak temannya untuk dapat menghubunginya melalui website `www.super.franky.yyy.com`, dan dikarenakan pengunjung web server pasti akan bingung dengan randomnya images yang ada, maka Franky juga meminta untuk mengganti request gambar yang memiliki substring `“franky”` akan diarahkan menuju `franky.png`. Maka bantulah Luffy untuk membuat konfigurasi dns dan web server ini!

**Server Skypie**

konfigurasi file `/var/www/super.franky.b13.com/.htaccess` dengan
```
echo "
RewriteEngine On
RewriteCond %{REQUEST_URI} ^/public/images/(.*)franky(.*)
RewriteCond %{REQUEST_URI} !/public/images/franky.png
RewriteRule /.* http://super.franky.b13.com/public/images/franky.png [L]
"

```
konfigurasi file `/etc/apache2/sites-available/super.franky.b13.com.conf` dengan
```
echo "
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.b13.com
        ServerName super.franky.b13.com
        ServerAlias www.super.franky.b13.com

        ErrorDocument 404 /error/404.html
        ErrorDocument 500 /error/404.html
        ErrorDocument 502 /error/404.html
        ErrorDocument 503 /error/404.html
        ErrorDocument 504 /error/404.html

        <Directory /var/www/super.franky.b13.com/public>
                Options +Indexes
        </Directory>

        Alias \"/js\" \"/var/www/super.franky.b13.com/public/js\"

        <Directory /var/www/super.franky.b13.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
        ErrorLog \${APACHE_LOG_DIR}/error.log
        CustomLog \${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/franky.b13.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>
</VirtualHost>
"

```
Melakukan restart service apache2 dengan `service apache2 restart`
