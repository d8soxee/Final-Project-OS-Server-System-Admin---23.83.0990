# Final-Project-OS-Server-System-Admin---23.83.0990

Repository ini saya gunakan untuk Dokumentasi Instalasi dan Konfigurasi Dedicated Server Minecraft, SSH Server, Web Server, Database Server, File Server, Mail Server. 

# OPERATING SYSTEM
Ubuntu server 20.04
---
**10 Desember 2024**
### **Persiapan Awal**
**Install Ubuntu Server 20.04**:
- Unduh dari [Ubuntu Server](https://ubuntu.com/download/server).
- Saat instalasi, gunakan:
- **Hostname**: `mineshraft`.
- **IP**: `192.168.40.210`.
---
Server Service
```bash
**Remote Access SSH Server (Secure Shell)**
**Reverse Proxy Server (NGINX)**  
**Web Server (PHP)**  
**Dependency Manager (Composer)**  
**Database Server (MySQL))**  
**Container Management (Docker)**  
**Caching and Data Storage (Redis)**
**Game Server Control Panel Wings (Pterodactyl Server Control Panel)**
**File Sharing Server (Samba)**
```
---

**21 Desember 2024**

**Mengaktifkan SSH di Ubuntu**

Secara default, saat Ubuntu pertama kali diinstal, akses jarak jauh melalui SSH tidak diizinkan. Mengaktifkan SSH di Ubuntu cukup mudah.
Lakukan langkah-langkah berikut sebagai root atau pengguna dengan menggunakan sudo untuk menginstal dan mengaktifkan SSH pada Ubuntu:
```bash
sudo apt update
sudo apt install openssh-server
```
Saat diminta, masukkan kata sandi Anda dan tekan Enter untuk melanjutkan instalasi.

Setelah instalasi selesai, layanan SSH akan mulai secara otomatis. Anda dapat memverifikasi bahwa SSH berjalan dengan mengetik:
```bash
sudo systemctl status ssh
```
Output akan memberi tahu Anda bahwa layanan sedang berjalan dan diaktifkan untuk memulai saat sistem melakukan boot:
```bash
● ssh.service - OpenBSD Secure Shell server
    Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
    Active: active (running) since Mon 2024-12-21 15:13:16 CUTC; 27s ago
```
Tekan q untuk kembali ke prompt baris perintah.

Ubuntu dilengkapi dengan alat konfigurasi firewall yang disebut UFW. Jika firewall diaktifkan pada sistem Anda, pastikan untuk membuka port SSH:
```bash
sudo ufw allow ssh
```
Selesai! Kini Anda dapat terhubung ke sistem Ubuntu melalui SSH dari komputer jarak jauh mana pun. Sistem Linux dan macOS telah memasang klien SSH secara default. Untuk terhubung dari komputer Windows, gunakan klien SSH seperti PuTTY .

**Menghubungkan ke Server SSH**

Untuk terhubung ke mesin Ubuntu Anda melalui LAN, jalankan perintah ssh diikuti dengan nama pengguna dan alamat IP dalam format berikut:
```bash
ssh username@ip_address
```
Pastikan Anda mengganti usernamedengan nama pengguna sebenarnya dan ip_addressdengan Alamat IP mesin Ubuntu tempat Anda menginstal SSH.
SSH memungkinkan akses jarak jauh ke server.

---
**16 Desember 2024**

**Instalasi Depedency**:
```bash
# Add "add-apt-repository" command
apt -y install software-properties-common curl apt-transport-https ca-certificates gnupg

# Add additional repositories for PHP (Ubuntu 20.04 and Ubuntu 22.04)
LC_ALL=C.UTF-8 add-apt-repository -y ppa:ondrej/php

# Add Redis official APT repository
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

# MariaDB repo setup script (Ubuntu 20.04)
curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash

# Update repositories list
apt update

# Install Dependencies
apt -y install php8.3 php8.3-{common,cli,gd,mysql,mbstring,bcmath,xml,fpm,curl,zip} mariadb-server nginx tar unzip git redis-server
```

**Menginstall Composer**

Composer adalah pengelola dependensi untuk PHP yang memungkinkan kita mengirimkan semua kode yang Anda perlukan untuk mengoperasikan Panel.
```bash
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```
![Composer](https://github.com/user-attachments/assets/d887f095-2f3e-4eae-beaf-7d1ba2f460d2)

**Mengunduh File**

proses ini adalah membuat folder tempat panel akan berada, lalu memindahkannya ke folder yang baru dibuat tersebut.
```bash
mkdir -p /var/www/pterodactyl
cd /var/www/pterodactyl
```
Setelah membuat direktori baru untuk Panel dan memindahkannya, Kita perlu mengunduh file Panel. Ini semudah mengunduh curlkonten yang telah dikemas sebelumnya. Setelah diunduh, perlu membongkar arsip dan kemudian menetapkan izin yang benar pada direktori storage/dan    bootstrap/cache/. Direktori ini memungkinkan kita untuk menyimpan file serta menyediakan cache yang cepat untuk mengurangi waktu load.
```bash
curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz
tar -xzvf panel.tar.gz
chmod -R 755 storage/* bootstrap/cache/
```
**Instalasi**

Setelah semua file sudah didownload, kita perlu mengkonfigurasi beberapa aspek inti panel
Konfigurasi DataBase

Anda memerlukan pengaturan database dan pengguna dengan izin yang tepat yang dibuat untuk database tersebut.
```bash
# If using MySQL
mysql_secure_installation

mysql -u root -p
   
# Remember to change 'yourPassword' below to be a unique password
CREATE USER 'pterodactyl'@'127.0.0.1' IDENTIFIED BY 'ptero123';
CREATE DATABASE panel;
GRANT ALL PRIVILEGES ON panel.* TO 'pterodactyl'@'127.0.0.1' WITH GRANT OPTION;
exit
```
   
Pertama-tama kita akan menyalin default environment settings file kita, menginstal dependensi inti, dan kemudian membuat kunci enkripsi aplikasi baru.
 ```bash
cp .env.example .env
COMPOSER_ALLOW_SUPERUSER=1 composer install --no-dev --optimize-autoloader

# Only run the command below if you are installing this Panel for
# the first time and do not have any Pterodactyl Panel data in the database.
php artisan key:generate --force
```
**Konfigurasi Environment**

```bash
php artisan p:environment:setup
php artisan p:environment:database

# To use PHP's internal mail sending (not recommended), select "mail". To use a
# custom SMTP server, select "smtp".
php artisan p:environment:mail
```
**Database Setup**

Sekarang kita perlu menyiapkan semua data dasar untuk Panel dalam database yang dibuat sebelumnya. Perintah di bawah ini mungkin memerlukan waktu untuk dijalankan. Harap JANGAN keluar dari proses hingga selesai! Perintah ini akan menyiapkan tabel database dan kemudian menambahkan semua Nests & Eggs untuk Pterodactyl.
```bash
php artisan migrate --seed --force
```
**Menambahkan user**

Selanjutnya, Kita perlu membuat administratif user agar dapat masuk ke panel. Untuk melakukannya, jalankan perintah di bawah ini. Saat ini, kata sandi harus memenuhi persyaratan berikut: 8 karakter, huruf besar dan kecil, minimal satu angka.
```bash
php artisan p:user:make
```
**Set Permission**
```bash
# If using NGINX, Apache or Caddy (not on RHEL / Rocky Linux / AlmaLinux)
chown -R www-data:www-data /var/www/pterodactyl/*
```
**Konfigurasi Crontab**

Hal pertama yang perlu kita lakukan adalah membuat cronjob baru yang berjalan setiap menit untuk memproses tugas-tugas Pterodactyl tertentu, seperti pembersihan sesi dan pengiriman tugas-tugas terjadwal ke daemon. Kita perlu membuka crontab Anda menggunakan sudo crontab -edan kemudian menempelkan baris di bawah ini
```bash
* * * * * php /var/www/pterodactyl/artisan schedule:run >> /dev/null 2>&1
```
**Creat Queue Worker**

Selanjutnya Anda perlu membuat systemd worker baru untuk menjaga proses antrian tetap berjalan di latar belakang. Antrean ini bertanggung jawab untuk mengirim email dan menangani banyak tugas latar belakang lainnya untuk Pterodactyl.
Buat sebuah berkas dengan nama ```bash pteroq.service ``` pada ```bash /etc/systemd/system:```

```bash
# Pterodactyl Queue Worker File
# ----------------------------------

[Unit]
Description=Pterodactyl Queue Worker
After=redis-server.service

[Service]
# On some systems the user and group might be different.
# Some systems use `apache` or `nginx` as the user and group.
User=www-data
Group=www-data
Restart=always
ExecStart=/usr/bin/php /var/www/pterodactyl/artisan queue:work --queue=high,standard,low --sleep=3 --tries=3
StartLimitInterval=180
StartLimitBurst=30
RestartSec=5s

[Install]
WantedBy=multi-user.target
```
Jika menggunakan redis untuk sistem Anda, Anda perlu memastikan untuk mengaktifkannya agar dapat dimulai saat boot. Anda dapat melakukannya dengan menjalankan perintah berikut:
```bash
sudo systemctl enable --now redis-server
```
Terakhir, aktifkan layanan dan atur agar boot saat mesin dinyalakan.
```bash
sudo systemctl enable --now pteroq.service
```

**Konfigurasi Web Server**

Masuk ke direktori nginx
```bash
cd /etc/nginx/sites-enabled/default
```

lalu ubah konfigurasi pada file default
```bash
nano default
```

```bash
server {
    # Replace the example <domain> with your domain name or IP address
    listen 80;
    server_name <domain>;

    root /var/www/pterodactyl/public;
    index index.html index.htm index.php;
    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/pterodactyl.app-error.log error;

    # allow larger file uploads and longer script runtimes
    client_max_body_size 100m;
    client_body_timeout 120s;

    sendfile off;

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param PHP_VALUE "upload_max_filesize = 100M \n post_max_size=100M";
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTP_PROXY "";
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

**Enabling Configuration**
```bash
# You do not need to symlink this file if you are using RHEL, Rocky Linux, or AlmaLinux.
sudo ln -s /etc/nginx/sites-available/pterodactyl.conf /etc/nginx/sites-enabled/pterodactyl.conf

# You need to restart nginx regardless of OS.
sudo systemctl restart nginx
```

**Menginstall Wings**

Wings adalah server control plane generasi berikutnya dari Pterodacty. Sebelum menginsatll wings kita haru menginstall Docker.

**Menginstall Docker**

Untuk instalasi cepat Docker CE, Anda dapat menjalankan perintah di bawah ini:
```bash
curl -sSL https://get.docker.com/ | CHANNEL=stable bash
```
**Start Docker saat Booting**

Jika Anda menggunakan sistem operasi dengan systemd (Ubuntu 16+, Debian 8+, CentOS 7+) jalankan perintah di bawah ini agar Docker dimulai saat Anda mem-boot komputer Anda.
```bash
sudo systemctl enable --now docker
```
**Mengaktifkan Swap**

```bash
GRUB_CMDLINE_LINUX_DEFAULT="swapaccount=1"
```
```bash
Konfigurasi GRUB

Beberapa distro Linux mungkin mengabaikan GRUB_CMDLINE_LINUX_DEFAULT. Oleh karena itu, Anda mungkin harus menggunakan . GRUB_CMDLINE_LINUXsebagai gantinya jika . default tidak berfungsi untuk Anda.
```

**Menginstall Wings**

Langkah pertama untuk menginstal Wings adalah memastikan kita memiliki pengaturan struktur direktori yang diperlukan. Untuk melakukannya, jalankan perintah di bawah ini, yang akan membuat direktori dasar dan mengunduh file yang dapat dieksekusi Wings.
```bash
sudo mkdir -p /etc/pterodactyl
curl -L -o /usr/local/bin/wings "https://github.com/pterodactyl/wings/releases/latest/download/wings_linux_$([[ "$(uname -m)" == "x86_64" ]] && echo "amd64" || echo "arm64")"
sudo chmod u+x /usr/local/bin/wings
```
**Konfigurasi**

Setelah Anda memasang Wings dan komponen yang dibutuhkan, langkah selanjutnya adalah membuat node pada Panel yang telah Anda pasangBuka tampilan administratif Panel, pilih Nodes dari bilah sisi, dan di sisi kanan klik tombol Create New.

Setelah Anda membuat node, klik node tersebut dan akan muncul tab yang disebut Konfigurasi. Salin konten blok kode, tempel ke dalam file baru bernama config.ymlin /etc/pterodactyldan simpan.


**jalankan Wings**

Untuk memulai Wings, jalankan saja perintah di bawah ini, yang akan memulainya dalam mode debug. Setelah Anda memastikan bahwa Wings berjalan tanpa kesalahan, gunakan perintah CTRL+Cuntuk mengakhiri proses dan melakukan daemonisasi dengan mengikuti petunjuk di bawah ini. Bergantung pada koneksi internet server Anda.
```bash
sudo wings --debug
```

**Daemonizing(Menggunakan systemd)

Menjalankan Wings di backgroun merupakan tugas yang mudah, pastikan saja ia berjalan tanpa kesalahan sebelum melakukannya. Letakkan konfigurasi di bawah ini dalam sebuah file bernama wings.serviced pada /etc/systemd/system direktori.
```bash
[Unit]
Description=Pterodactyl Wings Daemon
After=docker.service
Requires=docker.service
PartOf=docker.service

[Service]
User=root
WorkingDirectory=/etc/pterodactyl
LimitNOFILE=4096
PIDFile=/var/run/wings/daemon.pid
ExecStart=/usr/local/bin/wings
Restart=on-failure
StartLimitInterval=180
StartLimitBurst=30
RestartSec=5s

[Install]
WantedBy=multi-user.target
```
Kemudian, jalankan perintah di bawah ini untuk memuat ulang systemd dan menjalankan Wings.
```bash
sudo systemctl enable --now wings
```
**22 Desember 2024**

**Alokasi Node**

Alokasi adalah kombinasi IP dan Port yang dapat Anda tetapkan ke server. Setiap server yang dibuat harus memiliki setidaknya satu alokasi. Alokasi akan menjadi alamat IP antarmuka jaringan Anda. Berikut Contoh membuat dan setting Alokasi Nodes > Konfiguration > Mmebuat Server > Server Panel
![PteroLogin](https://github.com/user-attachments/assets/f68303e1-cfef-438d-ae53-65b267ffde31)
![PteroNodes](https://github.com/user-attachments/assets/f8ae9593-5598-4826-b81a-7cf651816f83)
![PteroAlokasi](https://github.com/user-attachments/assets/52a0f7f3-4c64-4000-be35-a09bbba29a7b)
![PteroConfigGenToken](https://github.com/user-attachments/assets/98d2bcdf-ddf1-4c0a-8a75-7ac88be2d980)
![PteroDone](https://github.com/user-attachments/assets/402f9ba1-5e4b-4c8d-828a-373856bfb938)

Jangan gunakan 127.0.0.1 untuk alokasi.
Setelah itu Kita harus membuat server minecraft nya, dengan cara klik menu server pada sidebar.
![image](https://github.com/user-attachments/assets/1839cce3-5019-4b78-8e15-8abcf5a96056)
![PteroServerMIne](https://github.com/user-attachments/assets/330559f0-561b-4b98-81f2-21e6088d5be4)
![image](https://github.com/user-attachments/assets/8c531e0f-66dc-4ba9-afd4-ddc4266893b7)

**22 Desember 2024**

**File Sharing Server (Samba)**

```bash
sudo apt update
sudo apt install samba -y
```
Sebelum melakukan Konfigurasi kita harus membuat username dan password Samba di Linux agar bisa diakses dari Windows.

**Buat Username dan Password Samba**

Pastikan user sudah ada di sistem Linux. Jika belum ada, buat user baru dengan perintah berikut (ganti sambauser dengan nama pengguna yang diinginkan):
```bash
sudo adduser praditya
```
Anda akan diminta membuat password untuk user tersebut.
Tambahkan user tersebut ke Samba. Gunakan perintah berikut untuk menambahkan user Samba dan mengatur passwordnya:

```bash
sudo smbpasswd -a praditya
```

Anda akan diminta memasukkan password Samba untuk user tersebut.
Aktifkan user Samba. Pastikan user Samba diaktifkan dengan perintah:

```bash
sudo smbpasswd -e praditya
```
**Konfigurasi Direktori Berbagi**

Buat direktori untuk berbagi file:
```bash
sudo mkdir -p /srv/samba/share
sudo chmod 777 /srv/samba/share
```
Tambahkan beberapa file sebagai contoh:
```bash
echo "Selamat datang di File Server" | sudo tee /srv/samba/share/welcome.txt
```
**Edit Konfigurasi Samba**

Buka file konfigurasi Samba:
```bash
sudo nano /etc/samba/smb.conf
```
Tambahkan konfigurasi berikut di bagian bawah file:
ini
```bash
[Share]
path = /srv/samba/share
valid users = sambauser
read only = no
browsable = yes
create mask = 0660
directory mask = 0770
```
Simpan file dan keluar. lalu restart Samba
```bash
sudo systemctl restart smbd
```
**Verifikasi dan Akses**

Verifikasi Samba berjalan:
```bash
sudo systemctl status smbd
```
**Akses Share dari Windows**

Buka File Explorer di Windows.
Masukkan alamat jaringan Samba di address bar:
mathematica
```bash
\\IP_ADDRESS_LINUX\Share
Contoh: \\192.168.40.210\Share
```

Saat diminta Enter network credentials, masukkan:
```bash
Username: sambauser
Password: Password Samba yang Anda atur sebelumnya.
Centang "Remember my credentials" jika ingin menyimpan kredensial.
```
![image](https://github.com/user-attachments/assets/5db71d59-03e1-4c99-913a-790a31831284)
