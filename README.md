## VALERIAN KALEB SETIAWIRAWAN---FINAL-PROJECT-OS-SERVER-&-SYSTEM-ADMIN---23.83.0965
# Judul : Instalasi layanan server untuk hosting web delivery makanan

Repository ini berisi Dokumentasi Instalasi dan Konfigurasi berbagai layanan Server, seperti SSH, DNS, FTP, WEB, Database, dll. Saya menggunakan Ubuntu Server versi 22.04 sebagai base operasi sistem saya.
Beberapa Service yang dijelaskan dalam Repository ini masih dalam proses pengembangan, artinya masih ada beberapa service yang progressnya masih 50% jadi, kedepannya akan dikembangkan lagi.

Update: Progress telah mencapai 99%, dan ada kemungkinan masih akan dikembangkan lagi kedepannya

Progress:
- 19 November 2024: Penentuan tema yang di Install
- 29 November 2024: Penyelesaian tahap pertama
- 3 December 2024: Pergantian Tema
- 7 December 2024: Penyelesaian tahap kedua
- 14 December 2024: Penyelesaian tahap ketiga

# Versi layanan yang digunakan
1. Mysql: mysql  Ver 8.0.40-0ubuntu0.22.04.1 for Linux on x86_64
2. PHP: PHP 8.1.2-1ubuntu2.20 (cli)
3. Apache: Apache/2.4.52 (Ubuntu)
4. Operation System: Ubuntu Server 22.04 LTS
5. vsftpd: version 3.0.5
6. Bind9: BIND 9.18.28-0ubuntu0.22.04.1-Ubuntu (Extended Support Version)
7. OpenSSH: OpenSSH_8.9p1 Ubuntu-3ubuntu0.10

## Daftar Isi
1. [Instalasi dan Konfigurasi SSH](#1-instalasi-dan-konfigurasi-ssh-server)
2. [Instalasi dan Konfigurasi FTP server vsftpd](#2-Instalasi-dan-Konfigurasi-FTP-server-vsftpd)
3. [Instalasi dan Konfigurasi Database Server](#3-instalasi-dan-konfigurasi-database-server)
4. [Instalasi dan Konfigurasi Web Server](#4-instalasi-dan-konfigurasi-web-server)
5. [Instalasi dan Konfigurasi DNS Server](#5-instalasi-dan-konfigurasi-dns-server)
6. [Pengaplikasian website](#6-Pengaplikasian-website)

## 1. Instalasi dan Konfigurasi SSH Server
Kita akan menggunakan SSH agar mempermudah kita dalam mengontrol dan memodifikasi server secara remote
### 1.1 Instalasi SSH
**Langkah 1: Lakukan Update paket Ubuntu**
```
apt update
```
![SSH1](SS/SSH/1.png)

**Langkah 2: Lakukan Instalasi Paket SSH Server**
```
apt-get install openssh-server
```
### 1.2 Konfigurasi SSH
**Langkah 1: Buka Direktori konfigurasi ssh dengan text editor(disini saya menggunakan nano)**
```
nano /etc/ssh/sshd_config
```
![SSH2](SS/SSH/2.png)

**Langkah 2: Edit Konfigurasi seperti dibawah ini**

![SSH3](SS/SSH/3.png)

Saya mengubah Port dari 22 ke 4646 untuk mengamankanya agar tidak menggunakan default port.

**Langkah 3: Restart layanan SSH Server**
```
systemctl restart sshd
```
![SSH4](SS/SSH/4.png)

### 1.3 Menguji Konfigurasi
```
ssh root@IPADDR -p 4646
```
![SSH5](SS/SSH/5.png)

Bisa disesuaikan dengan IP dan Port yang anda konfigurasi

### 1.4 Konfigurasi Firewall
**Langkah 1: Mengizinkan lalu lintas TCP yang masuk pada port 4646**
```
ufw allow 4646
```
![SSH6](SS/SSH/6.png)

### 1.5 cek status Firewall
```
ufw status
```
![SSH6](SS/SSH/7.png)

Jika firewall belum aktif, masukkan perintah
```
ufw enable
```

## 2. Instalasi dan Konfigurasi FTP server vsftpd
Kita akan menginstall FTP server untuk mentransfer file antar Ubuntu dengan windows melalui jaringan lokal. Sementara itu, vsftpd (Very Secure FTP Daemon) adalah server FTP yang dapat meningkatkan keamanan, kinerja, dan stabilitas.

### 2.1 Instalasi vsftpd

**Langkah 1: Instalasi Paket vsftpd**
```
apt update
apt install vsftpd
```
![FTP1](SS/FTP/1.png)

![FTP2](SS/FTP/2.png)


### 2.2 Konfigurasi vsftpd
**Langkah 1: Buka Direktori Konfigurasi vsftpd**
```
nano /etc/vsftpd.conf
```

![FTP3](SS/FTP/3.png)

**Langkah 2: Edit Konfigurasi file Seperti dibawah ini**
```
listen=YES
listen_ipv6=NO
anonymous_enable=NO

#HILANGKAN TANDA PAGAR PADA LINE KONFIGURASI DIBAWAH INI#
local_enable=YES
write_enable=YES
local_umask=022
ascii_upload_enable=YES
ascii_download_enable=YES
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list
ls_recurse_enable=YES
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES

rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=YES

#TAMBAHKAN DI BAGIAN PALING AKHIR
pasv_enable=YES
pasv_enable=Yes
pasv_max_port=10090
pasv_min_port=10100
```
Setelah itu keluar dari tampilan konfigurasi lalu restart vsftpd
```
systemctl restart vsftpd
```
![FTP4](SS/FTP/4.png)

**Langkah 3: cek status vsftpd**
```
nano service vsftpd status
```
![FTP5](SS/FTP/5.png)


**Langkah 4: Konfigurasi UFW Firewall**
Jika Anda telah mengaktifkan firewall UFW, maka Anda perlu membuka port di atas di firewall dengan perintah berikut.
```
sudo ufw allow from any to any port 20,21,10090:10100 proto tcp
```
![FTP6](SS/FTP/6.png)

**Langkah 5: Menyiapkan akun pengguna**
```
useradd rian
passwd rian
sudo mkdir -p /home/ftpnic/ftp
sudo chown nobody:nogroup /home/ftpnic/ftp
sudo chmod a-w /home/ftpnic/ftp
```
![FTP7](SS/FTP/7.png)
![FTP8](SS/FTP/8.png)
![FTP9](SS/FTP/9.png)

**Langkah 6: Menambahkan user baru ke list**
```
nano /etc/vsftpd.chroot_list
#Tambahkan user baru di direktori ini
rian
```
![FTP10](SS/FTP/10.png)

**Langkah 7: Restart Layanan Vsftpd**
```
systemctl restart vsftpd
```
![FTP11](SS/FTP/11.png)

### 2.3 Menguji Konfigurasi

FTP Server ini adalah Tampilan dari FTP client(Menggunakan Filezila)
![FTP12](SS/FTP/12.png)

Jika FTP tidak dapat terhubung, coba matikan ufw firewall dengan cara
```
ufw disable
```

## 3. Instalasi dan Konfigurasi Database Server
Dalam proyek ini, saya melakukan instalasi database server menggunakan MySQL, sebuah sistem manajemen basis data open-source yang kuat. MySQL digunakan untuk menyimpan dan mengelola data yang diperlukan oleh aplikasi dan situs web. Saya juga menginstal phpMyAdmin, antarmuka web yang memudahkan administrasi dan manajemen database MySQL, memungkinkan pengguna untuk dengan mudah membuat, mengedit, dan mengelola basis data melalui antarmuka berbasis web yang intuitif.

### 3.1 Instalasi MySQL

**Langkah 1: Instalasi Paket MySQL**
```
apt update
apt install mysql-server
```
![Database1](SS/Database/1.png)
![Database2](SS/Database/2.png)

**Langkah 2: Jalankan MySQL**
```
systemctl start mysql.service
systemctl status mysql.service
```
![Database3](SS/Database/3.png)

### 3.2 Konfigurasi MySQL
**Langkah 1: Ikuti Konfigurasi dibawah ini**

Peringatan: Mulai Juli 2022, sebuah kesalahan akan terjadi ketika Anda menjalankan skrip `mysql_secure_installation` tanpa konfigurasi lebih lanjut. Alasannya adalah skrip ini akan mencoba mengatur kata sandi untuk akun MySQL root instalasi, tetapi, secara default pada instalasi Ubuntu, akun ini tidak dikonfigurasikan untuk terhubung menggunakan kata sandi.

Hal ini akan membawa skrip ke dalam perulangan rekursif yang hanya dapat Anda hentikan dengan menutup jendela terminal Anda.

Karena skrip `mysql_secure_installation` melakukan sejumlah tindakan lain yang berguna untuk menjaga keamanan instalasi MySQL Anda, Anda tetap disarankan untuk menjalankannya sebelum mulai menggunakan MySQL untuk mengelola data Anda. Namun, untuk menghindari perulangan ini, Anda perlu menyesuaikan terlebih dahulu cara autentikasi pengguna root MySQL Anda.

Pertama, buka prompt MySQL:
```
sudo mysql
```
![Database4](SS/Database/4.png)

Kemudian jalankan perintah `ALTER USER` berikut ini untuk mengubah metode autentikasi pengguna root menjadi metode yang menggunakan kata sandi. Contoh berikut ini mengubah metode autentikasi menjadi `mysql_native_password`:
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
![Database5](SS/Database/5.png)

Setelah melakukan perubahan ini, keluarlah dari prompt MySQL:
```
mysql> exit
```
Jalankan scriptnya lagi dengan:
```
mysql_secure_installation
```
![Database6](SS/Database/6.png)

Pertanyaan pertama akan menanyakan apakah Anda ingin menyiapkan Plugin Validasi Kata Sandi, yang dapat digunakan untuk menguji kekuatan kata sandi pengguna MySQL baru sebelum menganggapnya valid.

Jika Anda memilih untuk menyiapkan Plugin Validasi Kata Sandi, setiap pengguna MySQL yang Anda buat yang mengautentikasi dengan kata sandi akan diminta untuk memiliki kata sandi yang sesuai dengan kebijakan yang Anda pilih:
![Database7](SS/Database/7.png)
![Database8](SS/Database/8.png)
![Database9](SS/Database/9.png)

Dari sini, Anda dapat menekan Y dan kemudian ENTER untuk menerima default untuk semua pertanyaan berikutnya. Ini akan menghapus beberapa pengguna anonim dan basis data uji, menonaktifkan login root jarak jauh, dan memuat aturan baru ini sehingga MySQL segera menghormati perubahan yang Anda buat.
![Database10](SS/Database/10.png)

Setelah skrip keamanan selesai, Anda dapat membuka kembali MySQL dan mengubah metode autentikasi pengguna root kembali ke metode default,` auth_socket`. Untuk mengautentikasi sebagai pengguna root MySQL menggunakan kata sandi, jalankan perintah ini:
```
mysql -u root -p
```
![Database11](SS/Database/11.png)

Kemudian kembali menggunakan metode autentikasi default dengan menggunakan perintah ini:
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;
```
![Database12](SS/Database/12.png)

Namun saya memilih untuk membiarkan metode autentikasi pengguna root tetap `mysql_native_password`
### 3.3 Instalasi dan Konfigurasi Phpmyadmin

**Langkah 1: Lakukan instalasi paket**
```
apt-get install phpmyadmin
```
![Database13](SS/Database/13.png)

**Langkah 2: Konfigurasi Phpmyadmin**
Peringatan: Saat prompt muncul, “apache2” disorot, tetapi tidak dipilih. Jika Anda tidak menekan SPASI untuk memilih Apache, installer tidak akan memindahkan file yang diperlukan selama instalasi. Tekan SPASI, TAB, lalu ENTER untuk memilih Apache.
![Database14](SS/Database/14.png)

Jika anda tetap membiarkan metorde autentikasi pengguna root tetap `mysql_native_password`, maka akan memicu kesalahan saat Anda mencoba mengatur kata sandi untuk pengguna phpmyadmin:
![Database15](SS/Database/15.png)

Untuk mengatasinya, pilih opsi batalkan untuk menghentikan proses instalasi. Kemudian, buka prompt MySQL Anda:
```
mysql -u root -p
```
![Database16](SS/Database/16.png)

Dari prompt, jalankan perintah berikut untuk menonaktifkan komponen Validasi Kata Sandi. Perhatikan bahwa ini tidak akan benar-benar menghapus instalasinya, tetapi hanya menghentikan komponen agar tidak dimuat pada server MySQL Anda:
```
UNINSTALL COMPONENT "file://component_validate_password";
```
![Database17](SS/Database/17.png)

Kemudian keluar dari MySQL client dan menginstal paket phpmyadmin lagi
```
apt-get install phpmyadmin
```
![Database18](SS/Database/18.png)

Setelah phpMyAdmin terinstal, Anda dapat membuka prompt MySQL sekali lagi dengan sudo mysql atau mysql -u root -p lalu jalankan perintah berikut ini untuk mengaktifkan kembali komponen Validasi Kata Sandi:
```
INSTALL COMPONENT "file://component_validate_password";
```
![Database19](SS/Database/19.png)

 ### 3.4 Menguji Konfigurasi 

**Langkah 1: aktifkan file konfigurasi Apache untuk phpMyAdmin**
```
a2enconf phpmyadmin.conf
```
![Database20](SS/Database/20.png)

**Langkah 2: Mengonfigurasi Akses Kata Sandi untuk Pengguna MySQL Khusus **
Masuk kedalam Shell MySQL lagi lalu buat user
```
mysql -u root -p
CREATE USER 'username'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

Kemudian, berikan hak istimewa yang sesuai kepada pengguna baru Anda. Sebagai contoh, Anda dapat memberikan hak istimewa kepada pengguna ke semua tabel dalam database, serta kekuasaan untuk menambah, mengubah, dan menghapus hak istimewa pengguna, dengan perintah ini:
```
GRANT ALL PRIVILEGES ON *.* TO 'username'@'localhost' WITH GRANT OPTION;
```
Lalu keluar dari shell
```
mysql> exit
```
**Langkah 3: Akses web interface**
Buka browser lalu isi ip atau domain yang telah di konfigurasi ke kolom pencarian
`https://your_domain_or_IP/phpmyadmin`
![Database21](SS/Database/21.png)

login menggunakan akun yang tadi telah dibuat
![Database22](SS/Database/22.png)

## 4. Instalasi dan Konfigurasi Web Server
Saya memilih untuk menggunakan Apache2 sebagai server web utama untuk proyek saya daripada Nginx. tidak ada alasan khusus saya memilih apache2,anda bisa memilih Nginx. saya juga akan menginstall CMS wordpress untuk Webserver ini. Apache2 memiliki dukungan yang baik untuk PHP, yang merupakan salah satu persyaratan utama untuk menjalankan WordPress dengan lancar.

### 4.1 Instalasi Apache2

**Langkah 1: Instalasi Paket Apache2**
```
apt update && apt upgrade
apt-get install apache2
```
![Web1](SS/Web/1.png)
![Web2](SS/Web/2.png)

### 4.2 Konfigurasi Apache2
**Langkah 1: Buka File Konfigurasi Apache2**
```
nano /etc/apache2/sites-available/000-default.conf
```
![Web3](SS/Web/3.png)

**Langkah 2: Sesuaikan Konfigurasi ini dengan domain yang anda gunakan**
![Web4](SS/Web/4.png)

**Lengkah 3: Restart Layanan Apache2**
```
systemctl restart apache2
```
![Web5](SS/Web/5.png)

**Langkah 4: Cek Apache2**
```
systemctl status apache2
```
![Web6](SS/Web/6.png)

Konfigurasi ufw
```
ufw allow in "Apache"
ufw status
```
![Web7](SS/Web/7.png)

Jika Konfigurasi Berhasil seharusnya muncul layanan web default seperti gambar dibawah ini:
![Web8](SS/Web/8.png)

### 4.3 Konfigurasi CMS Wordpress pada Apache2

**Langkah 1: Melakukan Instalasi PHP**
```
sudo apt install -y php libapache2-mod-php php-{common,mysql,xml,xmlrpc,curl,gd,imagick,cli,dev,imap,mbstring,opcache,soap,zip,intl}
```
![Web9](SS/Web/9.png)

cek versi php
```
php -v
```
![Web10](SS/Web/10.png)

**Langkah 2: Melakukan Instalasi Database Server**

Untuk ini sudah dibuat bab lain jadi anda bisa cek ke [Instalasi dan Konfigurasi Database Server](#3-instalasi-dan-konfigurasi-database-server)

**Langkah 3: Buat Database untuk wordpress**

login ke Database Terlebih dahulu:
```
mysql -u root -p
```
![Web11](SS/Web/11.png)

buat database untuk Wordpress dan berikan password sesuai keinginan anda
```
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
![Web12](SS/Web/12.png)
![Web13](SS/Web/13.png)
![Web14](SS/Web/14.png)

**Langkah 4: Download dan Extract Paket Wordpress**
```
cd /tmp && wget https://wordpress.org/latest.tar.gz
tar -xvf latest.tar.gz
```
![Web15](SS/Web/15.png)
![Web16](SS/Web/16.png)

**Langkah 5: Salin direktori wordpress ke dalam direktori /var/www/html.**
```
cp -R wordpress /var/www/html/
```
![Web17](SS/Web/17.png)

**Langkah 6: Mengubah kepemilikan direktori /var/www/html/wordpress. dan Memodifikasi izin file.**
```
chown -R www-data:www-data /var/www/html/wordpress/
chmod -R 755 /var/www/html/wordpress/
```
![Web18](SS/Web/18.png)

**Langkah 8: Salin wp-config-sample.php dan ubah kepemilikan berkas wp-config.php.**
```
cd /var/www/html/wordpress
cp wp-config-sample.php wp-config.php
chown www-data:www-data wp-config.php
```
![Web19](SS/Web/19.png)

**Langkah 9: Edit file wp-config.php dan tambahkan baris di bawah ini pada file wp-config.php.**
```
nano wp-config.php
```
![Web20](SS/Web/20.png)
![Web21](SS/Web/21.png)

**Langkah 10: Tambahkan secure values dari wordpress secret key generator**
```
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```
Salin nilai ini dan tambahkan di file wp-config.php
![Web22](SS/Web/22.png)
![Web23](SS/Web/23.png)

**Langkah 11: Konfigurasikan apache untuk memuat situs wordpress.**
```
cd /etc/apache2/sites-available
cp 000-default.conf wordpress.conf
```
![Web24](SS/Web/24.png)

Masuk kedalam file wordpress.conf
```
nano wordpress.conf
```
![Web25](SS/Web/25.png)

Edit file wordpress.conf dan tambahkan baris di bawah ini
![Web26](SS/Web/26.png)

Aktifkan WordPress.conf dan muat ulang layanan apache.
```
a2ensite wordpress.conf
systemctl reload apache2
```
![Web27](SS/Web/27.png)

**Langkah 12: Instalasi Admin Wordpress**

1. Buka web wordpress pada browser dengan mengetikkan
```
http://your_domain_or_IP/wordpress
```
pada kolom pencarian

2. Pilih Bahasa yang diinginkan 
![Web28](SS/Web/28.png)

3. Isi Nama situs,Password,dan Email
![Web29](SS/Web/29.png)

4. Login dengan Akun yang sudah dibuat
![Web30](SS/Web/30.png)

5. CMS sudah Berhasil diinstall
![Web31](SS/Web/31.png)


### 4.4 Pengujian Konfigurasi Apache2

Uji coba tampilan pada webserver

![Web32](SS/Web/32.png)

## 5. Instalasi dan Konfigurasi DNS Server

### 5.1 Instalasi BIND9

**Langkah 1: Instalasi Paket bind9**
```
apt update
apt-get install bind9
```
![DNS1](SS/DNS/1.png)
![DNS2](SS/DNS/2.png)

### 5.2 Konfigurasi BIND9

**Langkah 1: copy file untuk Konfigurasi "Forward" dan "Reverse"**
```
cd /etc/bind
cp db.local db.forward
cp db.127 db.reverse
```
![DNS3](SS/DNS/3.png)

**Langkah 2: Konfigurasi file db.forward**
```
nano db.forward
```
![DNS4](SS/DNS/4.png)

Ubahlah Konfigurasi seperti dibawah ini:
![DNS5](SS/DNS/5.png)

disesuaikan dengan domain anda

**Langkah 3: Konfigurasi file db.reverse**
```
nano db.reverse
```
![DNS6](SS/DNS/6.png)

ubahlah Konfigurasi seperti dibawah ini:
![DNS7](SS/DNS/7.png)

**Langkah 4: Buka Konfigurasi named.conf.local untuk konfigurasi DNS Zones**
```
nano named.conf.local
```
![DNS8](SS/DNS/8.png)

Ubahlah isi file konfigurasi seperti dibawah ini:
![DNS9](SS/DNS/9.png)

**Langkah 5: Konfigurasi Forwarders**
```
nano named.conf.options
```
![DNS10](SS/DNS/10.png)

tambahkan DNS forwarders
![DNS11](SS/DNS/11.png)

**Langkah 6: Konfigurasi DNS diperangkat Server**
```
nano /etc/resolv.conf
```
![DNS12](SS/DNS/12.png)

ubahlah jadi seperti ini:

![DNS13](SS/DNS/13.png)

**Langkah 7: Restart Layanan Bind9**
```
systemctl restart bind9
```
![DNS14](SS/DNS/14.png)

### 5.3 Pengujian Konfigurasi DNS

**Langkah 1: Instalasi paket dns resolver**
```
apt-get install dnsutils
```
**Langkah 2: Lakukan pengujian dengan nslookup**
![DNS15](SS/DNS/15.png)
![DNS16](SS/DNS/16.png)

ini menunjukkan bahwa Konfigurasi DNS untuk mengubah alamat domain ke IP dan sebaliknya(foward&reverse) sudah berjalan dengan baik

## 6. Pengaplikasian website
dikarenakan saya akan menggunakan tampilan website sendiri, saya akan menampilkan website menggunakan apache2 langsung.

### 6.1 Konfigurasi file kedalam ubuntu

**Langkah 1: pemindahan file**
Kita dapat memindahkan file html yang telah dibuat dengan menggunakan Filezilla FTP, atau dengan git clone website yang telah dibuat (disini saya menggunakan git clone saja).
```
cd /tmp && git clone "nama repository"
```

**Langkah 2: salin direktori html kedalam direktori /var/www/html.**
```
cp -R "nama direktori web" /var/www/html/
```

**Langkah 3: Mengubah kepemilikan direktori web. dan Memodifikasi izin file.**
```
chown -R www-data:www-data /var/www/html/"nama direktori web"/
chmod -R 755 /var/www/html/"nama direktori web"/
```

**Langkah 4: Konfigurasikan apache untuk memuat web.**
```
cd /etc/apache2/sites-available
cp 000-default.conf web.conf
```
Masuk kedalam file web.conf dan ganti DocumentRoot menjadi halaman web mu
```
nano web.conf
DocumentRoot /var/www/html/"nama direktori web"
```
Aktifkan web.conf dan muat ulang layanan apache.
```
a2ensite web.conf
systemctl reload apache2
```

### 6.2 Tampilan web
#### Tampilan Home/Awal
![image](SS/Tampilan/H1.png)
![image](SS/Tampilan/H2.png)

#### Tampilan Detail Makanan
![image](SS/Tampilan/D1.png)
![image](SS/Tampilan/D2.png)

#### Tampilan Keranjang
![image](SS/Tampilan/C1.png)
![image](SS/Tampilan/C2.png)

#### Tampilan About
![image](SS/Tampilan/A1.png)
![image](SS/Tampilan/A2.png)
![image](SS/Tampilan/A3.png)

#### Tampilan Login
![image](SS/Tampilan/Login.png)

#### Tampilan Registrasi
![image](SS/Tampilan/Registration.png)