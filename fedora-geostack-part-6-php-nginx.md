## Part 6: Configuring PHP, PHP-FPM and Nginx

Salah-satu komponen yang dapat menjembatani webmap frontend (tentu saja ini: HTML) dengan database yang paling populer adalah PHP. Pada bagian ini kita akan membahas tentang bagaimana konfigurasinya dan bagaimana menghubungkannya dengan HTTP server yang akan kita gunakan, yaitu Nginx.

> Sedikit OOT, banyak developer yang memandang-rendah terhadap PHP. Mereka bilang PHP itu bahasa pemrogramannya _newbies_, _nggak_ mutu, _sucks_ dan ketinggalan jaman. _Issue_ yang kebanyakan mereka angkat adalah performa (terutama pada saat sistemnya diakses oleh ratusan, ribuan, bahkan jutaan users). Dan masih banyak lagi _issues_ lain yang dimanfaatkan oleh para _PHP-haters_ itu untuk _bad-mouthing about_ PHP di Internet.

> Tapi bagi saya (saya berharap Anda juga) tidak. Saya bukan _PHP-hater_. Saya jadi _PHP-user_ saja. (Webmap) _development_ pada prinsipnya adalah proses penyelesaian masalah, yang mana hasil akhirnya adalah sebuah solusi. PHP (dan sama juga dengan bahasa-bahasa pemrograman lainnya) itu pada akhirnya hanya menjadi sebuah alat untuk mencapai tujuan akhir. Pada prosesnya, yang penting juga bukan PHP-nya juga _koq_, tapi arsitektur dan algoritma yang diterapkan dalam setiap _business logic_-nya, bahkan hingga komponen terkecil dan paling sederhana sekalipun. Saya nggak malu pakai PHP, karena saya pikir reputasi saya akan lebih memalukan lagi jika webmap saya nggak jalan.

Stop OOT-nya, langsung mulai kerja saja. Asumsikan saja Anda sudah login ke server sebagai rinjani melalui SSH dari workstation Anda:

### 1. _Editing_ php.ini file

  Langkah pertama adalah membuat direktori untuk penampungan **PHP_SESSION** _files_, dan diikuti dengan mengedit file ```php.ini``` yang _by default_ berada di direktori ```/etc```. _Commands_-nya:
  
  ```
  [rinjani@nusantara ~]$ sudo mkdir /tmp/phpsessions
  
  [rinjani@nusantara ~]$ sudo chmod -R 777 /tmp/phpsessions
  
  [rinjani@nusantara ~]$ sudo nano /etc/php.ini
  ```
  
  > Sebagai catatan, pada file ```php.ini```, **comments** selalu diawali dengan karakter **;** (titik-koma / _semicolon_).
  
  Tambahkan, atau ganti _values_ pada _entries_ berikut ini, yang akan ditampilkan dalam sebuah _table_:
  
  Entry/Setting | Default/Old Value | New Value | Comments
  ------------- | ----------------- | --------- | --------
  **expose_php** | ```On``` | ```Off``` | -
  **error_reporting** | ```E_ALL & ~E_DEPRECATED & ~E_STRICT``` | ```E_ALL``` | -
  **post_max_size** | ```2M``` | ```200M``` | -
  **upload_max_filesize** | ```2M``` | ```200M``` | -
  **date.timezone** | ```disabled, empty``` | ```"Asia/Jakarta"``` | ```uncomment```
  **pdo_mysql.default_socket** | ```empty``` | ```/var/lib/mysql/mysql.sock``` | -
  **mysqli.default_socket** | ```empty``` | ```/var/lib/mysql/mysql.sock``` | -
  **mysqli.default_host** | ```empty``` | ```localhost``` | -
  **session.save_path** | ```disabled, empty``` | ```"/tmp/phpsessions"``` | ```uncomment```
  
  Misal:
  
  ```
  ...
  ;date.timezone = 
  ...
  ```
  
  menjadi:
  
  ```
  ...
  date.timezone = "Asia/Jakarta"
  ...
  ```
  
  Dan misal:
  
  ```
  ...
  mysql.default_socket = 
  ...
  ```
  
  menjadi:
  
  ```
  ...
  mysql.default_socket = /var/lib/mysql/mysql.sock
  ...
  ```
  
  > Jadi, ada _values_ yang pakai **"..."** (petik-ganda / _double-quotes_), dan ada yang tidak. Cermati format _values_-nya.
  
  _Save_ perubahannya dengan menekan **Ctrl+O** lalu **\<Enter\>** untuk mengkonfirmasi **Yes**, dan _exit_ dari **_nano editor_** dengan menekan **Ctrl-X**.
  
### 2. PHP-FPM (PHP FastCGI Process Manager)

  Setelah file ```php.ini``` diedit, maka **PHP-FPM**-nya harus diaktifkan lewat **systemd**:
  
  ```
  [rinjani@nusantara ~]$ sudo systemctl enable php-fpm.service
  
  [rinjani@nusantara ~]$ sudo systemctl start php-fpm.service
  ```
  
  _Test_ PHP-nya dengan _commands_ berikut ini:
  
  ```
  [rinjani@nusantara ~]$ php --ri pgsql
  
  [rinjani@nusantara ~]$ php --ri mysqli
  
  [rinjani@nusantara ~]$ php --ri gd
  
  [rinjani@nusantara ~]$ php --ri gmp
  
  [rinjani@nusantara ~]$ php --ri json
  
  [rinjani@nusantara ~]$ php --ri session
  
  [rinjani@nusantara ~]$ php --ri sqlite3
  
  [rinjani@nusantara ~]$ php --ri exif
  ```
  
  Kalau _tests_ di atas sudah menampilkan informasi bahwa modul-modul tersebut _support_-nya _enabled_, maka PHP Anda sudah siap.
  
### 3. _Editing_ nginx.conf file

  Setting ```php.ini``` memang _njlimet_ (rumit). Yang berikut ini bakalan lebih _njlimet_ lagi, yaitu setting Nginx pada file ```nginx.conf```:
  
  ```
  [rinjani@nusantara ~]$ sudo nano /etc/nginx/nginx.conf
  ```
  
  Yang diedit/ditambah adalah:
  
  #### 3.1. types_hash_max_size
  
  Yang tadinya:
  
  ```
  ...
  types_hash_max_size 2048;
  ...
  ```
  
  menjadi:
  
  ```
  ...
  types_hash_max_size 4096;
  ...
  ```
  
  #### 3.2. client_max_body_size
  
  Tambahkan _entry_ ```client_max_body_size``` di bawah ```types_hash_max_size``` supaya _user_ bisa meng-_upload file_ hingga **200 MB**:
  
  ```
  ...
  types_hash_max_size 4096;
  client_max_body_size 200m;
  ...
  ```
  
  #### 3.3. index
  
  Masuk ke blok ```server { ... }```, ubah **index** yang tadinya:
  
  ```
  ...
  root /usr/share/nginx/html;
  index index.html index.htm;
  ...
  ```
  
  menjadi:
  
  ```
  ...
  root /usr/share/nginx/html;
  index index.php index.html index.htm;
  ...
  ```
  
  #### 3.4. PHP FastCGI
  
  Sambil mulai membahas strategi **_reverse-proxy_** pada Nginx, masih di blok ```server { ... }```, tambahkan:
  
  ```
  ...
  proxy_buffering     off;
  proxy_buffer_size   128k;
  proxy_buffers       128k;
  ...
  ```
  
  sebagai _proxy buffer directives_ yang berlaku secara global.
  
  Kemudian tambahkan _directives_ untuk **PHP FastCGI**-nya:
  
  ```
  ...
  location / {
  }
  ...
  ```
  
  menjadi:
  
  ```
  ...
  location / {
  }

  location ~ [^/]\.php(/|$) {
    try_files $uri =404;
    fastcgi_split_path_info ^(.+?\.php)(/.*)$;
    if (!-f $document_root$fastcgi_script_name) {
      return 404;
    }
    fastcgi_param HTTP_PROXY "";
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param  SCRIPT_FILENAME   /usr/share/nginx/html$fastcgi_script_name;
  }
  ...
  ```
  
  Dan akhirnya tambahkan _directives_ berikut ini di akhir blok ```server { ... }```:
  
  ```
  ...
  location ~ /\.ht {
    deny all;
  }
  ...
  ```
  
  _Save_ perubahannya dengan menekan **Ctrl+O** lalu **\<Enter\>** untuk mengkonfirmasi **Yes**, dan _exit_ dari **_nano editor_** dengan menekan **Ctrl-X**.
  
  > Untuk lebih lengkapnya, Anda bisa melihat contoh [**nginx.conf**](./files/nginx-part-6.conf) ini.
  
  #### 3.5. systemd dan firewalld untuk Nginx
  
  Untuk mengaktifkan Nginx sebagai **service**, jalankan _commands_:
  
  ```
  [rinjani@nusantara ~]$ sudo chmod -R 777 /usr/share/nginx/html
  
  [rinjani@nusantara ~]$ sudo systemctl enable nginx.service
  
  [rinjani@nusantara ~]$ sudo firewall-cmd --permanent --zone=FedoraServer --add-port=80/tcp
  
  [rinjani@nusantara ~]$ sudo firewall-cmd --reload
  
  [rinjani@nusantara ~]$ sudo firewall-cmd --permanent --zone=FedoraServer --add-port=443/tcp
  
  [rinjani@nusantara ~]$ sudo firewall-cmd --reload
  
  [rinjani@nusantara ~]$ sudo systemctl restart php-fpm.service
  
  [rinjani@nusantara ~]$ sudo systemctl start nginx.service
  ```
  
  Buka URL **```http://192.168.1.23```** di _browser_ Anda, maka Nginx _test page_ akan muncul.
  
  ![Nginx PHP](./img/nginx-01-test-page.jpg)

### 4. phpinfo()

  Kalau pada bagian sebelumnya kita test Nginx-nya untuk menampilkan **_default_ HTML file** (index.html) sebagai _test page_, maka langkah selanjutnya adalah _test_ PHP-nya. Ini mudah saja, _create file_ ber-_extension_ **.php** dalam _root directory_ Nginx, yaitu di ```/usr/share/nginx/html```.
  
  ```
  [rinjani@nusantara ~]$ sudo nano /usr/share/nginx/html/phptest.php
  ```
  
  Dalam file yang masih kosong ini tuliskan saja:
  
  ```
  <?php phpinfo(); ?>
  ```
  
  _Save_ perubahannya dengan menekan **Ctrl+O** lalu **\<Enter\>** untuk mengkonfirmasi **Yes**, dan _exit_ dari **_nano editor_** dengan menekan **Ctrl-X**. Lalu ubah _permission_-nya menjadi **777** saja, dengan _command_:
  
  ```
  [rinjani@nusantara ~]$ sudo chmod 777 /usr/share/nginx/html/phptest.php
  ```
  
  Buka URL **```http://192.168.1.23/phptest.php```** di _browser_ Anda, maka PHP _info page_ akan muncul.
  
  ![Nginx PHP](./img/nginx-02-php-test.jpg)
  
  Sampai pada tahap ini, **PHP** dan **PHP-FPM** sudah terintegrasi dengan **Nginx HTTP server**, sehingga dapat mengakses _database_ dan layanan data lainnya, seperti **WMS/WMTS/WFS services_**.
  
### 5. PHP and Modern API-Style

  Sekarang adalah eranya **API (Application Programming Interface)**. Salah satu implementasi dari API adalah **Web Service**, yaitu layanan data (_data service_) yang "dibungkus" atau "ditumpangkan" dalam layanan **HTTP** (atau **HTTPS**). Dalam terminologi Web Service, kita sering mendengar istilah **"endpoint"**.
  
  _There's nothing new under the sun_. Sesuatu yang terlihat kompleks dan "canggih" itu ternyata (masih saja) gabungan dari hal-hal yang sederhana dan sangat mendasar. Yang dimaksud dengan _endpoint_ pada _modern API_ adalah **URL** yang dituju aplikasi untuk mendapatkan data tertentu. Misal: ```http://192.168.1.23/webservice/points``` adalah endpoint yang outputnya GeoJSON point. Sekarang masalahnya sederhana saja: _endpoint_ itu kalau bisa ya berformat sederhana dan tidak "berpihak" pada salah satu jenis bahasa pemrograman saja. Kondisi Webmap Development Server kita sekarang, kalau mau membuat _endpoint_, ya di belakangnya masih harus menuliskan **.php** -- misal: ```http://192.168.1.23/webservice/points.php```.
  
  Nah, webmap moderen itu ya juga menggunakan **Web Service**, misalnya untuk mendapatkan **data GeoJSON**. Pada pemrograman _frontend_-nya, penulisan _endpoint_ sudah cenderung nggak akan mempedulikan _engine_ apa yang bekerja dalam _endpoint_-nya. Misal, kalau _frontend_ butuh data **points**, ya mengarahnya ke ```http://192.168.1.23/webservice/points```, bukan ke ```http://192.168.1.23/webservice/points.php```. Kita lihat segi positifnya saja, dengan menerapkan API-style seperti ini, tentu saja akan lebih membebaskan backend untuk memilih dan mengembangkan teknologi under the hood-nya. Misal, kalau webmap-nya sudah sangat kompleks dan traffic-nya tinggi, dan backend (yang pakai PHP) sudah mulai kepayahan, ya backend bisa switching teknologinya (misal pakai Node atau Golang) tanpa mengubah endpoints-nya sama-sekali.
  
  Lalu apa langkah awal yang harus kita lakukan supaya Webmap Development Server bisa fleksibel dengan kebutuhan modern API-style? Ya mode akses PHP di Nginx-nya dibuat extensionless. Jadi nanti kalau kita mengakses -- misalnya -- ```http://192.168.1.23/webservice/points``` ya berarti sama saja dengan mengakses ```http://192.168.1.23/webservice/points.php```.
  
  Langkah-langkahnya adalah sebagai berikut:
  
  ```
  [rinjani@nusantara ~]$ sudo systemctl stop php-fpm.service
  
  [rinjani@nusantara ~]$ sudo systemctl stop nginx.service
  
  [rinjani@nusantara ~]$ sudo nano /etc/nginx/nginx.conf
  ```
  
  dan ubah blok:
  
  ```
  ...
  location / {
  }
  
  location ~ [^/]\.php(/|$) {
    try_files $uri =404;
    fastcgi_split_path_info ^(.+?\.php)(/.*)$;
    if (!-f $document_root$fastcgi_script_name) {
      return 404;
    }
    fastcgi_param HTTP_PROXY "";
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param  SCRIPT_FILENAME   /usr/share/nginx/html$fastcgi_script_name;
  }
  ...
  ```
  
  menjadi:
  
  ```
  ...
  location / {
    try_files $uri $uri.html $uri/ @extensionless-php;
  }
  
  location ~ [^/]\.php(/|$) {
    try_files $uri =404;
    fastcgi_split_path_info ^(.+?\.php)(/.*)$;
    if (!-f $document_root$fastcgi_script_name) {
      return 404;
    }
    fastcgi_param HTTP_PROXY "";
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    include fastcgi_params;
    fastcgi_param  SCRIPT_FILENAME   /usr/share/nginx/html$fastcgi_script_name;
  }
  
  location @extensionless-php {
    rewrite ^(.*)$ $1.php last;
  }
  ...
  ```
  
  _Save_ perubahannya dengan menekan **Ctrl+O** lalu **\<Enter\>** untuk mengkonfirmasi **Yes**, dan _exit_ dari **_nano editor_** dengan menekan **Ctrl-X**. Lalu aktifkan lagi PHP-FPM dan Nginx-nya:
  
  ```
  [rinjani@nusantara ~]$ sudo systemctl start php-fpm.service
  
  [rinjani@nusantara ~]$ sudo systemctl start nginx.service
  ```
  
  > Untuk lebih lengkapnya, Anda bisa melihat contoh [**nginx.conf**](./files/nginx-extensionless-php.conf) ini.
  
  Kalau pada bagian sebelumnya Anda sudah mencoba membuat PHP test page di URL **```http://192.168.1.23/phptest.php```**, maka coba sekarang Anda buka URL **```http://192.168.1.23/phptest```** (perhatikan: tanpa **.php**) di _browser_ Anda, maka PHP _info page_ akan tetap muncul, tanpa harus menggunakan **.php** pada URL-nya (lihat _box_ merah).
  
  ![Nginx PHP](./img/nginx-03-extensionless-php.jpg)
  
  Dan akhirnya, kalau sudah beres semua, delete saja file ```phptest.php``` dari root-nya Nginx, supaya sistemnya tetap bersih.
  
  ```
  [rinjani@nusantara ~]$ sudo rm -rf /usr/share/nginx/html/phptest.php
  ```
  
Hingga tahap ini, **Webmap Development Server** Anda sudah dapat berfungsi sebagai **database server** dan **HTTP server** yang (juga) mendukung _modern API-style_. Pada tahap berikutnya, kita akan mengulas _reverse-proxy_ di Nginx. Terima kasih sudah bersabar untuk mengikuti terus tutorial ini!

> Instalasi dan konfigurasi sebuah _tech-stack_ adalah sebuah _craftmanship_ -- semakin mendalam Anda menggali bagaimana sebuah komponen bekerja dalam ekosistemnya, _in-and-out_, maka semakin paham pula Anda terhadap seluruh ketidaksempurnaan yang pernah Anda jumpai dalam hidup.

### Related parts:
  * [Part 1](./README.md): Post-Installation / OS Configuration
  * [Part 2](./fedora-geostack-part-2-apache-tomcat.md): Configuring Apache Tomcat for GeoServer
  * [Part 3](./fedora-geostack-part-3-geoserver.md): GeoServer Installation / Configuration
  * [Part 4](./fedora-geostack-part-4-postgis.md): Configuring PostgreSQL and PostGIS
  * [Part 5](./fedora-geostack-part-5-mysql.md): Configuring MySQL Database
  * Part 6: Configuring PHP, PHP-FPM and Nginx (this file)
  * [Part 7](./fedora-geostack-part-7-reverse-proxy.md): Configuring Nginx as a Reverse-Proxy
  * [Part 8](./fedora-geostack-part-8-ftp.md): Configuring Pure-FTPd
