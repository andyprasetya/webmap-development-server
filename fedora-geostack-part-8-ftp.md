## Part 8: Configuring Pure-FTPd

Untuk mempermudah editing/uploading files ke HTTP server, lebih baik kita manfaatkan sebuah FTP server, yaitu Pure-FTPd.

+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
# groupadd xftp
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
# useradd xftp -g xftp -s /sbin/nologin -d /home/xftp -c "Default Virtual FTP Users" -m
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
# nano /etc/pure-ftpd/pure-ftpd.conf
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
Edit:

VerboseLog			yes
[uncomment saja:]
PureDB				/etc/pure-ftpd/pureftpd.pdb
PassivePortRange	30000 50000
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
Save + exit
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
# pure-pwconvert >> /etc/pure-ftpd/pureftpd.passwd
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
# chmod 600 /etc/pure-ftpd/pureftpd.passwd
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
# ln -s /etc/pure-ftpd/pureftpd.passwd /etc/pureftpd.passwd
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
# pure-pw mkdb
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
# pure-pw useradd hootie -u xftp -g xftp -d /var/www/html
<password>
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
# pure-pw mkdb
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
# pure-pw useradd sigdesa -u xftp -g xftp -d /var/www/html
<password>
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
# pure-pw mkdb
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
# systemctl enable pure-ftpd
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
# systemctl start pure-ftpd
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
# firewall-cmd --zone=public --add-port=21/tcp --permanent
# firewall-cmd --reload
# firewall-cmd --zone=public --add-port=30000-50000/tcp --permanent
# firewall-cmd --reload