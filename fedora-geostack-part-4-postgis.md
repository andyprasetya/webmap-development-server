### Part 4: Configuring PostgreSQL and PostGIS

Berikut ini adalah langkah-langkah konfigurasi PostgreSQL/PostGIS setelah prosedur instalasinya selesai dilaksanakan pada [Part 1](./README.md).

##### 1. Inisiasi _built-in database_ PostgreSQL:

Setelah proses instalasi PostgreSQL (dan PostGIS, PgRouting, dll.) selesai dilaksanakan, hal pertama yang paling mendasar untuk dilakukan adalah menginisiasi _built-in database_-nya. _Command_-nya adalah sebagai berikut:

  ```
  [rinjani@nusantara ~]$ sudo postgresql-setup --initdb --unit postgresql
  
  [rinjani@nusantara ~]$ sudo systemctl enable postgresql.service
  
  [rinjani@nusantara ~]$ sudo systemctl start postgresql.service
  ```
  
  PostgreSQL sudah aktif dan siap diakses.
  
##### 2. _Create_ users (superuser dan regular user):

  Pada praktiknya, Anda akan membutuhkan 2 (dua) jenis user, yaitu **_superuser_** dan **_regular user_**. User dengan akses level _superuser_ dimanfaatkan untuk _create_, _modify_ dan _drop_ database maupun users, serta mengatur _grant access_ untuk _regular users_. Sedangkan _regular users_ digunakan untuk mengakses database dari aplikasi.

  ```
  [rinjani@nusantara ~]$ sudo su - postgres
  ```
  
  _Shell_ akan berubah menjadi:
  
  ```
  [postgres@nusantara ~]$ 
  ```
  
  Jalankan _commands_:
  
  ```
  [postgres@nusantara ~]$ createuser -sdrP pgdbadmin
  ```
  > _Parameter_ -sdrP digunakan untuk _create_ user yang _username_-nya **pgdbadmin**, dan memiliki _access privilege_ sebagai _superuser_ dan langsung _prompting password_-nya.
  > Anda akan diminta untuk memasukkan _password_ untuk user tersebut dan konfirmasinya.

  ```
  [postgres@nusantara ~]$ createuser -P pgdbuser
  ```
  > _Parameter_ -P digunakan untuk _create_ user yang _username_-nya **pgdbuser**, dan memiliki _access privilege_ sebagai _regular user_ dan langsung _prompting password_-nya.
  > Anda akan diminta untuk memasukkan _password_ untuk user tersebut dan konfirmasinya.
  
  > Untuk mengakses sebuah _database_, _regular users_ butuh _grant access_ yang dijalankan oleh _superuser_.
  
##### 3. _Create_ database _template_ (untuk PostGIS dan PgRouting):

  Masih di _shell_ yang sama, langkah berikutnya adalah membuat _template_ database yang nantinya akan digunakan sebagai _template_ database aplikasi.
  
  ```
  [postgres@nusantara ~]$ createdb postgis_template
  ```
  
  Masuk ke database _postgis_template_ yang baru saja di-_create_ dengan menggunakan _psql_ (PostgreSQL Client):
  
  ```
  [postgres@nusantara ~]$ psql -d postgis_template
  ```
  
  Maka tampilan _shell_-nya akan berubah menjadi:
  
  ```
  postgis_template-#
  ```
  
##### 4. _Create extensions_ pada _template database_:

  ```
  postgis_template-# CREATE EXTENSION postgis;
  
  postgis_template-# CREATE EXTENSION postgis_topology;
  
  postgis_template-# CREATE EXTENSION fuzzystrmatch;
  
  postgis_template-# CREATE EXTENSION address_standardizer;
  
  postgis_template-# CREATE EXTENSION pgcrypto;
  ```
  
  Jika Anda membutuhkan _extension_ PgRouting, maka tinggal jalankan:
  
  ```
  postgis_template-# CREATE EXTENSION pgrouting;
  ```
  
  Hingga tahap ini _template database_ Anda sudah memiliki seluruh _spatial-related extensions_ yang dibutuhkan.
  
  Untuk konfirmasinya, Anda bisa langsung menjalankan _commands_:
  
  ```
  postgis_template-# SELECT postgis_full_version();
  
  postgis_template-# SELECT * FROM pgr_version();
  ```
  
  Untuk keluar dan kembali ke PostgreSQL _Shell_, jalankan command:
  
  ```
  postgis_template-# \q
  ```
  
##### 5. Mengganti postgres/root _password_:

  _By default_, _password_ untuk _built-in superuser_ PostgreSQL (postgres) adalah **KOSONG** atau tidak ber-_password_. Hal ini tentu saja berbahaya bagi sistem dari segi keamanan. Oleh sebab itu, langkah-langkahnya adalah sebagai berikut:
  
  > **Catatan:** _shell_ adalah postgres, atau langsung melanjutkan dari _command_ pada bagian sebelumnya.
  
  ```
  [postgres@nusantara ~]$ psql
  ```
  
  _Shell_ yang muncul adalah:
  
  ```
  postgres-# 
  ```
  
  _Command_ nya:
  
  ```
  postgres-# \password
  ```
  
  > Anda akan diminta untuk memasukkan _password_ untuk _superuser_ **postgres** ini dan konfirmasinya.
  
  Setelah mengganti _password_ selesai, keluar ke PostgreSQL _shell_ lagi:
  
  ```
  postgres-# \q
  ```
  
  Sehingga tampilan _shell command_-nya menjadi:
  
  ```
  [postgres@nusantara ~]$ 
  ```
  
  Keluar dari PostgreSQL Client _shell_:
  
  ```
  [postgres@nusantara ~]$ exit
  ```
  
  dan _shell_-nya akan kembali menjadi:
  
  ```
  [rinjani@nusantara ~]$ 
  ```
  
##### 6. Meng-edit file konfigurasi PostgreSQL

  Ada 2 (dua) file konfigurasi utama di PostgreSQL, yaitu **_postgresql.conf_** dan **_pg_hba.conf_**. Untuk meng-edit-nya, jalankan _command_:
  
  ```
  [rinjani@nusantara ~]$ sudo systemctl stop postgresql.service
  
  [rinjani@nusantara ~]$ sudo nano /var/lib/pgsql/data/postgresql.conf
  ```
  
  Ganti:
  
  ```listen_address = 'localhost'``` menjadi: ```listen_address = '*'``` yang berarti server akan listen dari seluruh IP address yang terdaftar di _local-machine_-nya, dan
  
  ```#port = 5432``` menjadi ```port = 5432``` sehingga PostgreSQL bisa diakses melalui port 5432.
  
  Save + exit, dan edit file pg_hba.conf dengan menjalankan _command_:
  
  ```
  [rinjani@nusantara ~]$ sudo nano /var/lib/pgsql/data/pg_hba.conf
  ```
  
  Ganti _entries_ berikut ini untuk mengatur _mode_ akses dari dalam:
  
  ```
  local       all       all                     peer
  
  host        all       all     127.0.0.1/32    ident
  
  host        all       all     ::1/128         ident
  ```
  
  menjadi:
  
  ```
  local       all       all                     trust
  
  host        all       all     127.0.0.1/32    md5
  
  host        all       all     ::1/128         md5
  ```
  
  Dan tambahkan _entry_ yang disesuaikan dengan _setting_ jaringan server Anda, untuk mengatur akses dari luar, pada akhir file:
  
  ```
  host        all       all     192.168.1.0/0   md5
  ```
  
  > _Entry_ ini mengasumsikan server Anda memiliki IP address 192.168.1.xxx/24.
  
  Save + exit, dan aktifkan kembali PostgreSQL:
  
  ```
  [rinjani@nusantara ~]$ sudo systemctl start postgresql.service
  ```

##### 7. _Create database_ dan pengaturan akses untuk _user(s)_:

  Pada dasarnya, _creating database_ adalah _create database_ berdasar _template_ yang sudah _spatially-enabled_ (yaitu **postgis_template**), dan mengatur akses untuk _user(s)_-nya . Langkah-langkahnya:
  
  ```
  [rinjani@nusantara ~]$ sudo su - postgres
  
  [postgres@nusantara ~]$ createdb webmap_db -T postgis_template -O pgdbadmin
  ```
  
  Penjelasan _command_ di atas adalah _create database_ dengan nama **webmap_db**, berdasar _template_ (**-T**) **postgis_template**, yang _owner_ (_superuser_)-nya (**-O**) **pgdbadmin**.
  
  Berikutnya, masuk ke _database_ **webmap_db** untuk mengatur akses _user_ (dalam hal ini: **pgdbuser**):
  
  ```
  [postgres@nusantara ~]$ psql -d webmap_db
  ```
  
  Dan jalankan _commands_:
  
  ```
  webmap_db=# GRANT CONNECT ON DATABASE webmap_db TO pgdbuser;
  
  webmap_db=# GRANT ALL PRIVILEGES ON DATABASE webmap_db TO pgdbuser;
  
  webmap_db=# \q
  
  [postgres@nusantara ~]$ exit
  ```
  
  Sehingga _shell command_-nya kembali menjadi:
  
  ```
  [rinjani@nusantara ~]$ 
  ```


> Instalasi dan konfigurasi sebuah _tech-stack_ adalah sebuah _craftmanship_ -- semakin mendalam Anda menggali bagaimana sebuah komponen bekerja dalam ekosistemnya, _in-and-out_, maka semakin paham pula Anda terhadap seluruh ketidaksempurnaan yang pernah Anda jumpai dalam hidup.

### Related parts:
  * [Part 1](./README.md): Post-Installation / OS Configuration
  * [Part 2](./fedora-geostack-part-2-apache-tomcat.md): Configuring Apache Tomcat for GeoServer
  * [Part 3](./fedora-geostack-part-3-geoserver.md): GeoServer Installation / Configuration
  * Part 4: Configuring PostgreSQL and PostGIS (this file)
