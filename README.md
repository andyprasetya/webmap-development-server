## Membangun GeoStack untuk Webmap Development Berbasis Fedora Linux

Tugas membangun _backend system_ untuk sebuah webmap _project_ adalah sebuah _tedious work_ dan seringkali menjadi _"lempar-lemparan"_ antara _GIS Specialists_ dan staf TI (mulai dari _devops_, _system administrators_, _techleads_, hingga _programmers_). Orang GIS _bilang_: _"...wah, itu yang tahu dan bisa hanya orang TI."_ Sementara orang TI juga bilang: _"...saya nggak tau geografi. Kerjaan itu sudah sangat spesifik. Coba tanya sama orang-orang GIS."_ -- Wah, kalau _gini_ caranya, _bakalan nggak_ selesai-selesai webmap _project_-nya. Dan ini belum sampai ke pertanyaan yang lebih klasik lagi: _"...yang tugas maintain sistemnya siapa nih?"_, saat sistemnya _trouble_. Membingungkan dan _overwhelming_, tapi ini sering sekali terjadi.

Tapi kalau nggak dimulai sekarang juga, ya kapan terwujudnya? _Keburu disamber_ sama "tuan singh" dan _konco-konco_-nya nanti. _Let's make a quickstart_, _berdarah-darah dikit_ dulu _nggak_ apa-apa, asal _ngelmu_ Geomatika kita terus berkembang maju!

> **Asumsi \#1**: Skenario _development ecosystem_ akan kita bangun adalah:

![Ecosystem](./img/dev-ecosystem.png)

> dimana:
> _Development workstation_ IP _address_: **192.168.1.2/24**, OS: **Windows 10**. Pada _workstation_ ini akan terpasang beberapa _software_ yang umum digunakan untuk webmap _development_, seperti [**Quantum GIS**](https://qgis.org/en/site/forusers/download.html), [**PostGIS Shapefile and DBF Loader/Exporter**](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads), [**Microsoft Visual Studio Code**](https://code.visualstudio.com/download) (atau [**Notepad++**](https://notepad-plus-plus.org/)), [**Postman**](https://www.getpostman.com/downloads/), [**PgAdmin**](https://www.pgadmin.org/download/pgadmin-4-windows/), [**MySQL Workbench**](https://dev.mysql.com/downloads/workbench/), [**SQLite DB Browser**](https://sqlitebrowser.org/), [**MongoDB Compass Community Edition**](https://www.mongodb.com/download-center/compass), [**PuTTy**](https://www.putty.org/) dan [**WinSCP**](https://winscp.net/eng/download.php).

> Webmap Development Server IP _address_: **192.168.1.23/24**, hostname: **nusantara**, default user: **rinjani**.

> **Asumsi \#2**: Instalasi OS (Fedora Server versi 30) pada Webmap Development Server sudah dilaksanakan, dengan tidak lupa untuk mengganti hostname dan setting IP address secara manual, sehingga bisa diakses dari Development Workstation dengan menggunakan PuTTy atau WinSCP. Adapun mode instalasi OS yang disarankan adalah **Minimal Installation**. Untuk langkah-langkah instalasinya, Anda bisa mengacu pada [dokumentasi](https://docs.fedoraproject.org/en-US/fedora/f30/install-guide/)-nya di situs [Fedora](https://getfedora.org/).

> Sebagai catatan tambahan, Anda bisa/boleh menggunakan [**Oracle VirtualBox**](https://www.virtualbox.org/) atau [**VMware Workstation Player**](https://www.vmware.com/id/products/workstation-player.html) untuk _hosting_ Fedora Linux-nya di workstation Anda.

> Untuk menjalankan langkah-langkah _post-installation_, _ecosystem_ ini harus terhubung dengan Internet!

## Part 1: Post-Installation / OS Configuration

#### 1. Login sebagai _administrator_ user

  Aktifkan **PuTTy** pada workstation Anda, dan buka akses ke **Webmap Development Server** (host: **192.168.1.23**, port: **22**, user: **rinjani**), dan setelah Anda masukkan _password_-nya muncul _shell_:
  
  ```
  [rinjani@nusantara ~]$ 
  ```
  
  masuk ke mode _superuser_ dan _update_ dulu sistemnya:

  ```
  [rinjani@nusantara ~]$ sudo su
  
  [root@nusantara rinjani]# dnf update
  
  [root@nusantara rinjani]# reboot
  ```
  
#### 2. Disabling SELinux

Setelah server di-_reboot_, _login_ lagi, dan langkah pertama yang dilakukan adalah men-_disable_ fitur SELinux. Langkah ini ditempuh supaya handling _filesystem_ tidak _ribet_. Walaupun SELinux bersifat _mandatory_ untuk production server, tapi untuk sementara dapat diabaikan dulu.

  ```
  [rinjani@nusantara ~]$ sudo nano /etc/sysconfig/selinux
  ```
  
  Ubah:
  
  ```
  SELINUX=enforcing
  ```
  
  Menjadi:
  
  ```
  SELINUX=disabled
  ```
  
  \[save + exit\]
  
  ```
  [rinjani@nusantara ~]$ sudo reboot
  ```
  
  Karena SELinux nya di-_disable_, maka sekarang pengamanan server akan diserahkan pada service _firewalld_. Untuk memeriksa apakah _firewall_ sudah terinstall dan aktif, jalankan shell command:
  
  ```
  [rinjani@nusantara ~]$ sudo firewall-cmd --list-all-zones
  ```
  
  Jika service _firewalld_ belum terinstall karena suatu hal, maka langkah-langkah instalasinya adalah sebagai berikut:
  
  ```
  [rinjani@nusantara ~]$ sudo dnf install firewalld
  
  [rinjani@nusantara ~]$ sudo systemctl unmask firewalld
  
  [rinjani@nusantara ~]$ sudo systemctl enable firewalld.service
  
  [rinjani@nusantara ~]$ sudo systemctl start firewalld.service
  ```
  
#### 3. Component Installation:

  ##### 3.1. Install base components:
  
  ```
  [rinjani@nusantara ~]$ sudo dnf install git binutils gcc kernel-headers kernel-devel virtualenv
  ```
  
  ##### 3.2. Install OpenJDK:
  
  ```
  [rinjani@nusantara ~]$ sudo dnf install java-1.8.0-openjdk-*
  ```
  
  Test instalasi OpenJDK dengan shell command:
  
  ```
  [rinjani@nusantara ~]$ java -version
  ```
  
  ##### 3.3. Install SELinux-related components (kalau SELinux-nya nanti mau diaktifkan)
  
  ```
  [rinjani@nusantara ~]$ sudo dnf install python3-policycoreutils policycoreutils-python-utils policycoreutils-devel policycoreutils-newrole policycoreutils-sandbox
  ```
  
  ##### 3.4. Install GDAL
  
  ```
  [rinjani@nusantara ~]$ sudo dnf install gdal gdal-libs gdal-devel gdal-doc gdal-java gdal-perl gdal-python3 python3-networkx-geo geos geos-devel proj proj-devel hdf5 hdf5-devel
  ```
  
  Untuk test GDAL, jalankan shell command:
  
  ```
  [rinjani@nusantara ~]$ gdalinfo --version
  ```
  
  ##### 3.5. Install GMT (Generic Mapping Tools):
  
  ```
  [rinjani@nusantara ~]$ sudo dnf install GMT GMT-common GMT-doc GMT-devel gshhg-gmt-nc4 gshhg-gmt-nc4-full gshhg-gmt-nc4-high dcw-gmt
  ```
  
  Untuk test GMT, jalankan shell command:
  
  ```
  [rinjani@nusantara ~]$ gmt
  ```
  
  atau langsung saja cek versi GMT:
  
  ```
  [rinjani@nusantara ~]$ gmt --version
  ```
  
  ##### 3.6. Install PostgreSQL base:
  
  ```
  [rinjani@nusantara ~]$ sudo dnf install postgresql postgresql-server postgresql-pgpool-II postgresql-contrib postgresql-devel postgresql-docs postgresql-pgpool-II-extensions postgresql-pgpool-II-devel postgresql-odbc postgresql-jdbc python3-postgresql postgresql-test
  ```
  
  ##### 3.7. Install PostgreSQL tools:
  
  ```
  [rinjani@nusantara ~]$ sudo dnf install pgaudit pg_top pg_view
  ```
  
  ##### 3.8. Install PostGIS, PgRouting dan OSM-related tools:
  
  ```
  [rinjani@nusantara ~]$ sudo dnf install postgis pgRouting readosm osmpbf osmpbf-java osmpbf-devel osmctools osmium-tool osm2pgsql
  ```
  
  ##### 3.9. Install MySQL Community:
  
  > Sebaiknya, cek terlebih dahulu versi termutakhir di situsnya MySQL.
  
  ```
  [rinjani@nusantara ~]$ sudo rpm -Uvh https://dev.mysql.com/get/mysql80-community-release-fc30-1.noarch.rpm
  
  [rinjani@nusantara ~]$ sudo dnf update
  
  [rinjani@nusantara ~]$ sudo dnf install mysql-community-server mysql-community-client mysql-community-common mysql-community-libs mysql-community-test mysql-community-devel
  ```
  
  ##### 3.10. Install MongoDB 4 (Community Edition):
  
  Biar nggak jadi dinosaurus, _nge-hype_ sedikit pasang **NoSQL** juga: **MongoDB**.
  
  ```
  [rinjani@nusantara ~]$ sudo nano /etc/yum.repos.d/mongodb.repo
  ```
  dan masukkan entry:
  
  ```
  [Mongodb]
  name=MongoDB Repository
  baseurl=https://repo.mongodb.org/yum/amazon/2013.03/mongodb-org/4.0/x86_64/
  gpgcheck=1
  enabled=1
  gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
  ```
  
  Save + exit, lalu update dan install MongoDB nya:
  
  ```
  [rinjani@nusantara ~]$ sudo dnf update
  
  [rinjani@nusantara ~]$ sudo dnf install mongodb-org mongodb-org-server mongodb-org-mongos mongodb-org-shell mongodb-org-tools
  ```
  
  ##### 3.11. Install Apache Tomcat:
  
  ```
  [rinjani@nusantara ~]$ sudo dnf install tomcat tomcat-webapps tomcat-admin-webapps
  ```
  
  ##### 3.12. Install PHP
  
  ```
  [rinjani@nusantara ~]$ sudo dnf install php php-fpm php-devel php-bcmath php-dba php-dbg php-exif php-gd php-gmp php-interbase php-mbstring php-pecl-mcrypt php-mysqlnd php-odbc php-opcache php-pdo php-pdo-dblib php-pear php-pecl-selinux php-pecl-redis php-pgsql php-process php-soap php-xml php-xmlrpc
  ```
  
  ##### 3.13. Install other components/software:
  
  ```
  [rinjani@nusantara ~]$ sudo dnf install composer samba* nginx pure-ftpd nodejs golang
  
  [rinjani@nusantara ~]$ sudo npm install -g nodemon
  
  [rinjani@nusantara ~]$ sudo curl --silent --location https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
  
  [rinjani@nusantara ~]$ sudo dnf install yarn
  ```

:grin: Sampai pada tahap ini, _geostack_ Anda sudah siap untuk dikonfigurasi seluruh komponen terkait nya.

> Instalasi dan konfigurasi sebuah _tech-stack_ adalah sebuah _craftmanship_ -- semakin mendalam Anda memahami bagaimana sebuah komponen bekerja dalam ekosistemnya, _in-and-out_, maka semakin paham pula Anda terhadap seluruh ketidaksempurnaan yang pernah Anda jumpai dalam hidup.

### Related parts:
  * Part 1: Post-Installation / OS Configuration (this file)
  * [Part 2](./fedora-geostack-part-2-apache-tomcat.md): Configuring Apache Tomcat for GeoServer
  * [Part 3](./fedora-geostack-part-3-geoserver.md): GeoServer Installation / Configuration
  * [Part 4](./fedora-geostack-part-4-postgis.md): Configuring PostgreSQL and PostGIS
  * [Part 5](./fedora-geostack-part-5-mysql.md): Configuring MySQL Database
  * [Part 6](./fedora-geostack-part-6-php-nginx.md): Configuring PHP, PHP-FPM and Nginx
  * [Part 7](./fedora-geostack-part-7-reverse-proxy.md): Configuring Nginx as a Reverse-Proxy
  * [Part 8](./fedora-geostack-part-8-ftp.md): Configuring Pure-FTPd
