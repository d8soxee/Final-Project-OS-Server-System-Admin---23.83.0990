# Final-Project-OS-Server-System-Admin---23.83.0990
Server's Name : MineShraft

Repository ini saya gunakan untuk Dokumentasi Instalasi dan Konfigurasi layanan Server, SSH Server, Web Server, Database Server, File Server, DNS Server, Proxy Server, Dedicated Server Minecraft 

# OPERATING SYSTEM
Ubuntu server 20.04
---
**10 Desember 2024**
### **Persiapan Awal**
1. **Install Ubuntu Server 20.04**:
   - Unduh dari [Ubuntu Server](https://ubuntu.com/download/server).
   - Saat instalasi, gunakan:
     - **Hostname**: `mineshraft`.
     - **IP Statik**: `192.168.1.36`.

2. **Login**:
   Masuk dengan kredensial yang dibuat saat instalasi.

---

### **1. Setup SSH Server**
SSH memungkinkan akses jarak jauh ke server.

1. **Install OpenSSH**:
   ```bash
   sudo apt update
   sudo apt install openssh-server -y
   ```

2. **Konfigurasi SSH**:
   Edit file konfigurasi:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   perintah ini digunakan untuk membuka file konfigurasi SSH daemon dengan hak akses superuser agar pengguna dapat melakukan perubahan pada pengaturan SSH.
   
   Pastikan parameter berikut:
   ```plaintext
   PermitRootLogin no
   PasswordAuthentication yes
   ```
   
   Restart layanan:
   ```bash
   sudo systemctl restart ssh
   ```

4. **Uji Koneksi**:
   Dari perangkat lain:
   ```bash
   ssh username@192.168.1.36
   ```

---
**16 Desember 2024**

### **2. Setup Database Server untuk Pterodactyl**
Panel Pterodactyl membutuhkan **Nginx**, **PHP**, dan **Composer**.

1. **Instalasi Depedency**:
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
2. **Menginstall Composer**
Composer adalah pengelola dependensi untuk PHP yang memungkinkan kita mengirimkan semua kode yang Anda perlukan untuk mengoperasikan Panel.
```bash
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```
3. **Mengunduh File**
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
4. **Instalasi**
Sekarang semua file sudah didownload, kita perlu mengkonfigurasi beberapa aspek inti panel
Konfigurasi DataBase

Anda memerlukan pengaturan database dan pengguna dengan izin yang tepat yang dibuat untuk database tersebut.
```bash
# If using MySQL
mysql_secure_installation

mysql -u root -p
   
# Remember to change 'yourPassword' below to be a unique password
CREATE USER 'pterodactyl'@'127.0.0.1' IDENTIFIED BY 'yourPassword';
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
5. **Konfigurasi Environment**
```bash
php artisan p:environment:setup
php artisan p:environment:database

# To use PHP's internal mail sending (not recommended), select "mail". To use a
# custom SMTP server, select "smtp".
php artisan p:environment:mail
```
6. **Database Setup**
Sekarang kita perlu menyiapkan semua data dasar untuk Panel dalam database yang dibuat sebelumnya. Perintah di bawah ini mungkin memerlukan waktu untuk dijalankan. Harap JANGAN keluar dari proses hingga selesai! Perintah ini akan menyiapkan tabel database dan kemudian menambahkan semua Nests & Eggs untuk Pterodactyl.
```bash
php artisan migrate --seed --force
```
7. **Menambahkan user**
Selanjutnya, Kita perlu membuat administratif user agar dapat masuk ke panel. Untuk melakukannya, jalankan perintah di bawah ini. Saat ini, kata sandi harus memenuhi persyaratan berikut: 8 karakter, huruf besar dan kecil, minimal satu angka.
```bash
php artisan p:user:make
```
8. **Set Permission**
```bash
# If using NGINX, Apache or Caddy (not on RHEL / Rocky Linux / AlmaLinux)
chown -R www-data:www-data /var/www/pterodactyl/*

# If using NGINX on RHEL / Rocky Linux / AlmaLinux
chown -R nginx:nginx /var/www/pterodactyl/*

# If using Apache on RHEL / Rocky Linux / AlmaLinux
chown -R apache:apache /var/www/pterodactyl/*
```
9. **Konfigurasi Crontab**
Hal pertama yang perlu kita lakukan adalah membuat cronjob baru yang berjalan setiap menit untuk memproses tugas-tugas Pterodactyl tertentu, seperti pembersihan sesi dan pengiriman tugas-tugas terjadwal ke daemon. Kita perlu membuka crontab Anda menggunakan sudo crontab -edan kemudian menempelkan baris di bawah ini
```bash
* * * * * php /var/www/pterodactyl/artisan schedule:run >> /dev/null 2>&1
```
10. **Creat Queue Worker**
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
### **3. Konfigurasi Web Server**
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
### **Menginstall Wings**
Wings adalah server control plane generasi berikutnya dari Pterodacty

1. **Menginstall Docker**
Untuk instalasi cepat Docker CE, Anda dapat menjalankan perintah di bawah ini:
```bash
curl -sSL https://get.docker.com/ | CHANNEL=stable bash
```
2. **Start Docker saat Booting**
Jika Anda menggunakan sistem operasi dengan systemd (Ubuntu 16+, Debian 8+, CentOS 7+) jalankan perintah di bawah ini agar Docker dimulai saat Anda mem-boot komputer Anda.
```bash
sudo systemctl enable --now docker
```
2. **Mengaktifkan Swap**
```bash
GRUB_CMDLINE_LINUX_DEFAULT="swapaccount=1"
```
```bash
Konfigurasi GRUB

Beberapa distro Linux mungkin mengabaikan GRUB_CMDLINE_LINUX_DEFAULT. Oleh karena itu, Anda mungkin harus menggunakan . GRUB_CMDLINE_LINUXsebagai gantinya jika . default tidak berfungsi untuk Anda.
```

3. **Menginstall Wings**
Langkah pertama untuk menginstal Wings adalah memastikan kita memiliki pengaturan struktur direktori yang diperlukan. Untuk melakukannya, jalankan perintah di bawah ini, yang akan membuat direktori dasar dan mengunduh file yang dapat dieksekusi Wings.
```bash
sudo mkdir -p /etc/pterodactyl
curl -L -o /usr/local/bin/wings "https://github.com/pterodactyl/wings/releases/latest/download/wings_linux_$([[ "$(uname -m)" == "x86_64" ]] && echo "amd64" || echo "arm64")"
sudo chmod u+x /usr/local/bin/wings
```
4. **Konfigurasi**
Setelah Anda memasang Wings dan komponen yang dibutuhkan, langkah selanjutnya adalah membuat node pada Panel yang telah Anda pasangBuka tampilan administratif Panel, pilih Nodes dari bilah sisi, dan di sisi kanan klik tombol Create New.

Setelah Anda membuat node, klik node tersebut dan akan muncul tab yang disebut Konfigurasi. Salin konten blok kode, tempel ke dalam file baru bernama config.ymlin /etc/pterodactyldan simpan.


5. **jalankan Wings**
Untuk memulai Wings, jalankan saja perintah di bawah ini, yang akan memulainya dalam mode debug. Setelah Anda memastikan bahwa Wings berjalan tanpa kesalahan, gunakan perintah CTRL+Cuntuk mengakhiri proses dan melakukan daemonisasi dengan mengikuti petunjuk di bawah ini. Bergantung pada koneksi internet server Anda.
```bash
sudo wings --debug
```

6. **Daemonizing(Menggunakan systemd)
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
7. **Alokasi Node**
Alokasi adalah kombinasi IP dan Port yang dapat Anda tetapkan ke server. Setiap server yang dibuat harus memiliki setidaknya satu alokasi. Alokasi akan menjadi alamat IP antarmuka jaringan Anda. Dalam beberapa kasus, seperti saat berada di belakang NAT, alamat IP internal akan menjadi alamat IP. Untuk membuat alokasi baru, buka Node > node Anda > Alokasi.
![PteroNodes2](https://github.com/user-attachments/assets/9ffc6d6f-36f1-4568-b7e5-a76f079a68c7)
![PteroConsole](https://github.com/user-attachments/assets/3affd587-3bc8-426c-83ae-d24710884720)
Jangan gunakan 127.0.0.1 untuk alokasi.


### **4. File Server (Samba)**
1. **Install Samba**
```bash
sudo apt update
sudo apt install samba -y
```
2. **Konfigurasi Direktori Berbagi**
Buat direktori untuk berbagi file:
```bash
sudo mkdir -p /srv/samba/share
sudo chmod 777 /srv/samba/share
```
Tambahkan beberapa file sebagai contoh:
```bash
echo "Selamat datang di File Server" | sudo tee /srv/samba/share/welcome.txt
```
3. **Edit Konfigurasi Samba**
Buka file konfigurasi Samba:
```bash
sudo nano /etc/samba/smb.conf
```
Tambahkan konfigurasi berikut di bagian bawah file:
ini
```bash
[Share]
path = /srv/samba/share
browseable = yes
writable = yes
guest ok = yes
read only = no
force user = nobody
```
Simpan file dan keluar.
4. **Restart Samba**
```bash
sudo systemctl restart smbd
```
5. **Verifikasi dan Akses**
Verifikasi Samba berjalan:
```bash
sudo systemctl status smbd
```
Akses dari komputer lain menggunakan alamat \\192.168.1.36\Share di File Explorer (Windows) atau menggunakan pengelola file di Linux.

### **5. DNS Server (Bind9)**
1. **Install Bind9**
```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc -y
```
2. **Konfigurasi File Zona**
Buat file zona untuk domain Anda:

```bash
sudo nano /etc/bind/db.mineshraft
Tambahkan isi berikut (sesuaikan IP dengan server Anda):

bind
Salin kode
$TTL    604800
@       IN      SOA     mineshraft.com. admin.mineshraft.com. (
                        2023121601 ; Serial
                        604800     ; Refresh
                        86400      ; Retry
                        2419200    ; Expire
                        604800 )   ; Negative Cache TTL
;
@       IN      NS      ns.mineshraft.com.
ns      IN      A       192.168.1.36
panel   IN      A       192.168.1.36
files   IN      A       192.168.1.36
```
Tambahkan file zona ke konfigurasi Bind:

```bash
sudo nano /etc/bind/named.conf.local
```
Tambahkan konfigurasi berikut:
```bash
bind
zone "mineshraft.com" {
    type master;
    file "/etc/bind/db.mineshraft";
};
```
3. **Restart Bind9**
```bash
sudo systemctl restart bind9
```

4. **Verifikasi Konfigurasi**
Uji konfigurasi Bind:
```bash
sudo named-checkconf
```

Verifikasi file zona:

```bash
sudo named-checkzone mineshraft.com /etc/bind/db.mineshraft
```
### **6. VPN Server (WireGuard)**
1. **Install WireGuard
```bash
sudo apt update
sudo apt install wireguard -y
```
2. **Konfigurasi WireGuard**
Buat direktori konfigurasi dan file kunci:

```bash
sudo mkdir -p /etc/wireguard
sudo chmod 700 /etc/wireguard
wg genkey | sudo tee /etc/wireguard/server_private.key | wg pubkey | sudo tee /etc/wireguard/server_public.key
```
Catat kunci pribadi dan publik.

Buat file konfigurasi:

```bash
sudo nano /etc/wireguard/wg0.conf
```
Tambahkan isi berikut:
```bash
ini
[Interface]
Address = 10.0.0.1/24
SaveConfig = true
ListenPort = 51820
PrivateKey = <ServerPrivateKey>
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <ClientPublicKey>
AllowedIPs = 10.0.0.2/32
```
Ganti <ServerPrivateKey> dan <ClientPublicKey> dengan kunci yang sesuai.

3. **Aktifkan IP Forwarding**
Buka file sysctl:
```bash
sudo nano /etc/sysctl.conf
```
Cari dan hapus komentar pada baris berikut:
```bash
net.ipv4.ip_forward=1
```
Terapkan perubahan:
```bash
sudo sysctl -p
```
4. **Jalankan WireGuard**
```bash
sudo systemctl enable --now wg-quick@wg0
```
5. **Konfigurasi Klien**
Instal aplikasi WireGuard di perangkat Anda.
Tambahkan konfigurasi klien:
ini
```bash
[Interface]
PrivateKey = <ClientPrivateKey>
Address = 10.0.0.2/24

[Peer]
PublicKey = <ServerPublicKey>
Endpoint = 192.168.1.36:51820
AllowedIPs = 0.0.0.0/0, ::/0
```

Ganti kunci dan IP sesuai konfigurasi server Anda.











### **4. Setup File Server**
Gunakan **Samba** untuk berbagi file.
1. **Install Samba**:
   ```bash
   sudo apt install samba -y
   ```

2. **Konfigurasi Samba**:
   Edit file konfigurasi:
   ```bash
   sudo nano /etc/samba/smb.conf
   ```
   Tambahkan:
   ```plaintext
   [SharedFiles]
   path = /srv/shared
   browsable = yes
   writable = yes
   guest ok = no
   create mask = 0660
   directory mask = 0771
   ```
   Buat direktori dan atur izin:
   ```bash
   sudo mkdir -p /srv/shared
   sudo chmod -R 775 /srv/shared
   sudo systemctl restart smbd
   ```
   
---

### **5. Setup DNS Server**
Gunakan **Bind9** untuk layanan DNS.

1. **Install Bind9**:
   ```bash
   sudo apt install bind9 -y
   ```

2. **Konfigurasi Zona DNS**:
   Edit file:
   ```bash
   sudo nano /etc/bind/named.conf.local
   ```
   Tambahkan:
   ```plaintext
   zone "mineshraft.local" {
       type master;
       file "/etc/bind/db.mineshraft.local";
   };
   ```
   Buat file zona:
   ```bash
   sudo cp /etc/bind/db.local /etc/bind/db.mineshraft.local
   sudo nano /etc/bind/db.mineshraft.local
   ```
   Edit:
   ```plaintext
   $TTL    604800
   @       IN      SOA     mineshraft.local. root.mineshraft.local. (
                               2         ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
   ;
   @       IN      NS      ns.mineshraft.local.
   ns      IN      A       192.168.1.36
   ```
   Restart Bind9:
   ```bash
   sudo systemctl restart bind9
   ```

---

### **6. Setup Proxy Server**
Gunakan **Squid** untuk proxy server.

1. **Install Squid**:
   ```bash
   sudo apt install squid -y
   ```

2. **Konfigurasi Squid**:
   Edit file konfigurasi:
   ```bash
   sudo nano /etc/squid/squid.conf
   ```
   Tambahkan:
   ```plaintext
   acl localnet src 192.168.1.0/24
   http_access allow localnet
   ```
   Restart Squid:
   ```bash
   sudo systemctl restart squid
   ```

---

### **7. Setup Dedicated Minecraft Server**
1. **Install Java**:
   ```bash
   sudo apt install openjdk-17-jre -y
   ```

2. **Download Minecraft Server**:
   ```bash
   mkdir -p ~/minecraft
   cd ~/minecraft
   wget https://launcher.mojang.com/v1/objects/<latest_version>/server.jar
   ```

3. **Jalankan Server**:
   ```bash
   java -Xmx1024M -Xms1024M -jar server.jar nogui
   ```

---

Jika Anda memerlukan detail tambahan atau langkah lanjutan, beri tahu saya!
