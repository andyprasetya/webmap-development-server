## Part 8: Configuring Pure-FTPd

Untuk mempermudah _editing_/_uploading_ files ke server, lebih baik kita manfaatkan sebuah **FTP server**, yaitu **Pure-FTPd**. Kalau di [**Part 1**](./README.md) kita sudah menyelesaikan instalasinya, berikut ini adalah langkah-langkah konfigurasinya.

> _Users_ yang akan kita _create_ ada 2 jenis, yaitu _user_ untuk mengakses file HTML/PHP/JavaScript/CSS/dll. di direktori **```/usr/share/nginx/html```** dan _user_ untuk mengakses **Node.JS _projects_** di direktori **```/home/rinjani/nodeapps```**.

  ### 1. _Create group_ dan _user_

  ```
  [rinjani@nusantara ~]$ mkdir nodeapps
  
  [rinjani@nusantara ~]$ sudo chmod -R 777 nodeapps
  
  [rinjani@nusantara ~]$ sudo groupadd xftp
  
  [rinjani@nusantara ~]$ sudo useradd xftp -g xftp -s /sbin/nologin -d /home/xftp -c "Default Virtual FTP Users" -m
  ```

  ### 2. Edit pure-ftpd.conf

  ```
  [rinjani@nusantara ~]$ sudo nano /etc/pure-ftpd/pure-ftpd.conf
  ```
  
  Edit (ditampilkan dalam _table_ saja):

  Entry/Setting | Default/Old Value | New Value | Comments
  ------------- | ----------------- | --------- | --------
  **VerboseLog** | ```no``` | ```yes``` | -
  **PureDB** | ```disabled```, ```@sysconfigdir/pureftpd.pdb``` | ```/etc/pure-ftpd/pureftpd.pdb``` | ```uncomment```
  **PassivePortRange** | ```disabled``` | ```30000 50000``` | ```uncomment```

  > Catatan: _comment_ di file ```pure-ftpd.conf``` adalah karakter **#**.
  
  > Untuk lebih lengkapnya, Anda bisa melihat contoh [**pure-ftpd.conf**](./files/pure-ftpd-sample.conf) ini.
  
  _Save_ perubahannya dengan menekan **Ctrl+O** lalu **\<Enter\>** untuk mengkonfirmasi **Yes**, dan _exit_ dari **_nano editor_** dengan menekan **Ctrl-X**.

  ### 3. Create virtual user
  
  ```
  [rinjani@nusantara ~]$ sudo pure-pwconvert >> /etc/pure-ftpd/pureftpd.passwd
  
  [rinjani@nusantara ~]$ sudo chmod 600 /etc/pure-ftpd/pureftpd.passwd
  
  [rinjani@nusantara ~]$ sudo ln -s /etc/pure-ftpd/pureftpd.passwd /etc/pureftpd.passwd
  
  [rinjani@nusantara ~]$ sudo pure-pw mkdb
  
  [rinjani@nusantara ~]$ sudo pure-pw useradd webmapdev -u xftp -g xftp -d /usr/share/nginx/html
  
  ...create + confirm password...
  
  [rinjani@nusantara ~]$ sudo pure-pw mkdb
  
  [rinjani@nusantara ~]$ sudo pure-pw useradd nodedev -u rinjani -g rinjani -d /home/rinjani/nodeapps
  
  ...create + confirm password...
  
  [rinjani@nusantara ~]$ sudo pure-pw mkdb
  ```
  
  ### 4. Enable service di systemd dan buka port di firewalld

  ```
  [rinjani@nusantara ~]$ sudo systemctl enable pure-ftpd
  
  [rinjani@nusantara ~]$ sudo firewall-cmd --permanent --zone=FedoraServer --add-port=21/tcp
  
  [rinjani@nusantara ~]$ sudo firewall-cmd --reload
  
  [rinjani@nusantara ~]$ sudo firewall-cmd --permanent --zone=FedoraServer --add-port=30000-50000/tcp
  
  [rinjani@nusantara ~]$ sudo firewall-cmd --reload
  
  [rinjani@nusantara ~]$ sudo systemctl start pure-ftpd
  ```

Akhirnya, sampai dengan tahap ini, **Webmap Development Server** Anda sudah (juga) memiliki **FTP server** dan siap sepenuhnya untuk "diajak kemanapun".

Anda bisa mengakses _root_ direktorinya Nginx dan Node.JS _projects_ Anda dengan menggunakan IDE atau editor seperti **Notepad++** atau **Visual Studio Code**, dengan menambahkan _plugin_ FTP.

Terima kasih atas perhatian dan kesabaran Anda, semoga rangkaian tutorial ini bisa bermanfaat bagi Anda, dan mari bersama kita majukan _ngelmu_ Geomatika untuk bangsa Indonesia!

> Instalasi dan konfigurasi sebuah _tech-stack_ adalah sebuah _craftmanship_ -- semakin mendalam Anda menggali bagaimana sebuah komponen bekerja dalam ekosistemnya, _in-and-out_, maka semakin paham pula Anda terhadap seluruh ketidaksempurnaan yang pernah Anda jumpai dalam hidup.

### Related parts:
  * [Part 1](./README.md): Post-Installation / OS Configuration
  * [Part 2](./fedora-geostack-part-2-apache-tomcat.md): Configuring Apache Tomcat for GeoServer
  * [Part 3](./fedora-geostack-part-3-geoserver.md): GeoServer Installation / Configuration
  * [Part 4](./fedora-geostack-part-4-postgis.md): Configuring PostgreSQL and PostGIS
  * [Part 5](./fedora-geostack-part-5-mysql.md): Configuring MySQL Database
  * [Part 6](./fedora-geostack-part-6-php-nginx.md): Configuring PHP, PHP-FPM and Nginx
  * [Part 7](./fedora-geostack-part-7-reverse-proxy.md): Configuring Nginx as a Reverse-Proxy
  * Part 8: Configuring Pure-FTPd (this file)
