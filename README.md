## Membangun GeoStack untuk Webmap Development Berbasis Fedora Linux

Berikut ini adalah langkah-langkah instalasi sebuah _geostack_ berbasis Fedora Linux 29.

> Asumsi: Instalasi dari DVD/USB Flashdisk/ISO file (jika dijalankan di atas VirtualBox/VM Ware) sudah dilaksanakan, dengan tidak lupa untuk mengganti hostname dan setting IP address secara manual. Untuk menjalankan langkah-langkah post-install, sistem harus terhubung dengan Internet!

### Part 1: Post-Installation / OS Configuration

#### 1. Login sebagai _administrator_ user

  ```
  [user@hostname ~]$ sudo su
  
  [root@hostname user]# dnf update
  
  [root@hostname user]# reboot
  ```
  
#### 2. Disabling SELinux

Langkah ini ditempuh supaya handling _filesystem_ tidak _ribet_. Walaupun SELinux bersifat _mandatory_ untuk production server, tapi untuk sementara dapat diabaikan dulu.

  ```
  [user@hostname ~]$ sudo nano /etc/sysconfig/selinux
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
  [user@hostname ~]$ sudo reboot
  ```
  
  Karena SELinux nya di-_disable_, maka sekarang pengamanan server akan diserahkan pada service _firewalld_. Untuk memeriksa apakah _firewall_ sudah terinstall dan aktif, jalankan shell command:
  
  ```
  [user@hostname ~]$ sudo firewall-cmd --list-all-zones
  ```
  
  Jika service _firewalld_ belum terinstall karena suatu hal, maka langkah-langkah instalasinya adalah sebagai berikut:
  
  ```
  [user@hostname ~]$ sudo dnf install firewalld
  
  [user@hostname ~]$ sudo systemctl unmask firewalld
  
  [user@hostname ~]$ sudo systemctl enable firewalld.service
  
  [user@hostname ~]$ sudo systemctl start firewalld.service
  ```
  
#### 3. Component Installation:

  ##### 3.1. Install base components:
  
  ```
  [user@hostname ~]$ sudo dnf install wget curl git glibc binutils gcc libaio kernel-headers kernel-devel virtualenv
  ```
  
  ##### 3.2. Install OpenJDK:
  
  ```
  [user@hostname ~]$ sudo dnf install java-1.8.0-openjdk-*
  ```
  
  Test instalasi OpenJDK dengan shell command:
  
  ```
  [user@hostname ~]$ java -version
  ```
  
  ##### 3.3. Install SELinux-related components (Kalau SELinux nya diaktifkan)
  
  ```
  [user@hostname ~]$ sudo dnf install python3-policycoreutils policycoreutils-python-utils policycoreutils-devel policycoreutils-newrole policycoreutils-sandbox
  ```
  
  ##### 3.4. Install GDAL
  
  ```
  [user@hostname ~]$ sudo dnf install gdal gdal-libs gdal-devel gdal-doc gdal-java gdal-perl gdal-python3 python3-networkx-geo geos geos-devel proj proj-devel hdf5 hdf5-devel
  ```
  
  Untuk test GDAL, jalankan shell command:
  
  ```
  [user@hostname ~]$ gdalinfo --version
  ```
  
  ##### 3.5. Install GMT (Generic Mapping Tools):
  
  ```
  [user@hostname ~]$ sudo dnf install GMT GMT-common GMT-doc GMT-devel gshhg-gmt-nc4 gshhg-gmt-nc4-full gshhg-gmt-nc4-high dcw-gmt
  ```
  
  Untuk test GMT, jalankan shell command:
  
  ```
  [user@hostname ~]$ gmt
  ```
  
  atau langsung saja cek versi GMT:
  
  ```
  [user@hostname ~]$ gmt --version
  ```
  
  ##### 3.6. Install PostgreSQL base:
  
  ```
  [user@hostname ~]$ sudo dnf install postgresql postgresql-server postgresql-pgpool-II postgresql-contrib postgresql-devel postgresql-docs postgresql-pgpool-II-extensions postgresql-pgpool-II-devel postgresql-odbc postgresql-jdbc python3-postgresql postgresql-test
  ```
  
  ##### 3.7. Install PostgreSQL tools:
  
  ```
  [user@hostname ~]$ sudo dnf install pgtune pgaudit pg_top pg_view
  ```
  
  ##### 3.8. Install PostGIS, PgRouting dan OSM-related tools:
  
  ```
  [user@hostname ~]$ sudo dnf install postgis pgRouting readosm osmpbf osmpbf-java osmpbf-devel osmctools osmium-tool osm2pgsql
  ```
  
  ##### 3.9. Install MySQL Community:
  > Sebaiknya, cek terlebih dahulu versi termutakhir di situsnya MySQL.
  
  ```
  [user@hostname ~]$ sudo rpm -Uvh https://dev.mysql.com/get/mysql80-community-release-fc29-2.noarch.rpm
  
  [user@hostname ~]$ sudo dnf update
  
  [user@hostname ~]$ sudo dnf install mysql-community-server mysql-community-client mysql-community-common mysql-community-libs mysql-community-test mysql-community-devel
  ```
  
  > **Catatan**: Jika diinstall di Fedora Workstation (yang memiliki GUI Gnome 2), maka dapat juga langsung menginstall **MySQL Workbench** dan **SQLite DB Browser**:
  
  ```
  [user@hostname ~]$ sudo dnf install mysql-community-server mysql-community-client mysql-community-common mysql-community-libs mysql-community-test mysql-community-devel mysql-workbench-community sqlitebrowser
  ```
  
  ##### 3.10. Install Apache Tomcat:
  
  ```
  [user@hostname ~]$ sudo dnf install tomcat tomcat-webapps tomcat-admin-webapps
  ```
  
  ##### 3.11. Install PHP
  
  ```
  [user@hostname ~]$ sudo dnf install php php-fpm php-devel php-bcmath php-dba php-dbg php-exif php-gd php-gmp php-interbase php-mbstring php-pecl-mcrypt php-mysqlnd php-odbc php-opcache php-pdo php-pdo-dblib php-pear php-pecl-selinux php-pecl-redis php-pgsql php-process php-soap php-xml php-xmlrpc
  ```
  
  ##### 3.12. Install other components/software:
  
  ```
  [user@hostname ~]$ sudo dnf install composer samba* nginx pure-ftpd nodejs golang
  ```

:grin: Sampai pada tahap ini, _geostack_ Anda sudah siap untuk dikonfigurasi seluruh komponen terkait nya.

> Instalasi dan konfigurasi sebuah _tech-stack_ adalah sebuah _craftmanship_ -- semakin mendalam Anda memahami bagaimana sebuah komponen bekerja dalam ekosistemnya, _in-and-out_, maka semakin paham pula Anda terhadap seluruh ketidaksempurnaan yang pernah Anda jumpai dalam hidup.

### Related parts:
  * Part 1: Post-Installation / OS Configuration (this file)
  * [Part 2](./fedora-geostack-part-2-apache-tomcat.md): Configuring Apache Tomcat for GeoServer
  * [Part 3](./fedora-geostack-part-3-geoserver.md): GeoServer Installation / Configuration
  * [Part 4](./fedora-geostack-part-4-postgis.md): Configuring PostgreSQL and PostGIS
