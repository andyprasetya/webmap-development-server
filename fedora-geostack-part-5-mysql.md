### Part 5: Configuring MySQL Database

Berikut ini adalah langkah-langkah konfigurasi **MySQL Database** setelah prosedur instalasinya selesai dilaksanakan pada [Part 1](./README.md). Tapi sebelumnya, ada baiknya juga Anda download dulu contoh datanya, dengan _command_:

  ```
  [rinjani@nusantara ~]# wget https://gitlab.com/andyprasetya/missing-files/raw/master/files/sampledata.sql
  ```
  
  Alternatif:
  ```
  [rinjani@nusantara ~]# wget https://raw.githubusercontent.com/andyprasetya/webmap-development-server/master/files/sampledata.sql
  ```

#### 1. MySQL Community Server

  ##### 1.1. _Enable_ dan _start_ MySQL sebagai _daemon_ (systemd)
  
  Langkah awal yang harus kita lakukan adalah enable service-nya, dan langsung di-start:
  
  ```
  [rinjani@nusantara ~]$ sudo systemctl enable mysqld.service
  
  [rinjani@nusantara ~]$ sudo systemctl start mysqld.service
  ```
  
  ##### 1.2. Mengganti _root password_
  
  Saat pertama kali dijalankan, **MySQL Community Server** akan membuat _temporary password_ untuk **root** (administrator) _user_-nya. Kita harus mengganti password ini, dengan login sebagai root dengan temporary password yang bisa kita lihat di log-nya dengan menjalankan shell command:
  
  ```
  [rinjani@nusantara ~]$ sudo grep 'A temporary password is generated for root@localhost' /var/log/mysqld.log | tail -1
  ```
  
  Temporary password-nya akan terlihat, dan selanjutnya login sebagai root:
  
  ```
  [rinjani@nusantara ~]$ mysql -u root -p
  ```
  
  Masukkan temporary password tersebut, dan Anda akan langsung masuk ke shell MySQL Client. Jalankan queries berikut ini untuk mengganti password root-nya:
  
  ```
  > ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '__password__';
  ```
  
  > _Password_ sebaiknya minimal 8 karakter dan mengandung kombinasi dari _upper_ [A-Z] dan _lower_ [a-z] _case characters_, numerik [0-9] dan alphanumerik [!@#$%^&*()-_=+]. Misal: **```EPeEsGe!4326```**.
  
  Sebagai catatan, query di atas masih menggunakan mysql_native_password untuk mendukung kompatibilitas dengan MySQL driver versi sebelum 8.0. Jika Anda yakin benar bahwa yang akan mengakses pasti menggunakan driver versi 8.0 atau yang lebih baru, query-nya:
  
  ```
  > ALTER USER 'root'@'localhost' IDENTIFIED BY '__password__';
  ```
  
  ##### 1.3. _Create user_
  
  Melanjutkan penggantian password untuk root, langsung saja Anda create user untuk mengakses MySQL dengan query:
  
  ```
  > CREATE USER 'webmap'@'localhost' IDENTIFIED WITH mysql_native_password BY '__password__';
  
  > CREATE USER 'webmap'@'%' IDENTIFIED WITH mysql_native_password BY '__password__';
  ```
  
  atau:
  
  ```
  > CREATE USER 'webmap'@'localhost' IDENTIFIED BY '__password__';
  
  > CREATE USER 'webmap'@'%' IDENTIFIED BY '__password__';
  ```
  
  Mengapa pada contoh di atas _create user_-nya 2X? **```'webmap'@'localhost'```** untuk akses dari _localhost_/mesin yang sama, dan **```'webmap'@'%'```** untuk akses dari _host_/mesin lain.
  
  ##### 1.4. _Create database_
  
  
  
  ##### 1.5. Pengaturan akses untuk user
  
  
  
  ##### 1.5. Membuka port di firewalld
  
  ```
  [rinjani@nusantara ~]$ sudo firewall-cmd --permanent --zone=FedoraServer --add-port=3306/tcp
  
  [rinjani@nusantara ~]$ sudo firewall-cmd --reload
  ```
  
#### 2. MySQL Workbench

### Related parts:
  * [Part 1](./README.md): Post-Installation / OS Configuration
  * [Part 2](./fedora-geostack-part-2-apache-tomcat.md): Configuring Apache Tomcat for GeoServer
  * [Part 3](./fedora-geostack-part-3-geoserver.md): GeoServer Installation / Configuration
  * [Part 4](./fedora-geostack-part-4-postgis.md): Configuring PostgreSQL and PostGIS
  * Part 5: Configuring MySQL Database (this file)