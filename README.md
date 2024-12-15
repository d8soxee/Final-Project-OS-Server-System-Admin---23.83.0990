# Final-Project-OS-Server-System-Admin---23.83.0990
Server's Name : MineShraft

Repository ini saya gunakan untuk Dokumentasi Instalasi dan Konfigurasi layanan Server, SSH Server, Web Server, Database Server, File Server, DNS Server, Proxy Server, Dedicated Server Minecraft 

**10 Desember 2024**
# OPERATING SYSTEM
Ubuntu server 20.04
---

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
```bash
16-December-2024
```
### **2. Setup Web Server untuk Pterodactyl**
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

1. **Install Nginx dan PHP**:
   ```bash
   sudo apt install nginx php-cli php-fpm php-mysql php-zip php-curl php-xml php-bcmath php-mbstring unzip composer -y
   ```

2. **Download dan Install Pterodactyl**:
   ```bash
   cd /var/www
   sudo mkdir pterodactyl
   sudo chown -R www-data:www-data pterodactyl
   sudo curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz
   sudo tar -xzvf panel.tar.gz -C pterodactyl
   cd pterodactyl
   sudo composer install --no-dev --optimize-autoloader
   ```

3. **Setup Database untuk Pterodactyl**:
   (Lihat bagian **Database Server** di bawah).

4. **Konfigurasi Nginx**:
   Buat file konfigurasi:
   ```bash
   sudo nano /etc/nginx/sites-available/pterodactyl
   ```
   Tambahkan:
   ```plaintext
   server {
       listen 80;
       server_name mineshraft.local;
       root /var/www/pterodactyl/public;
       index index.php;

       location / {
           try_files $uri $uri/ /index.php;
       }

       location ~ \.php$ {
           include snippets/fastcgi-php.conf;
           fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
           fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
           include fastcgi_params;
       }
   }
   ```
   Aktifkan konfigurasi:
   ```bash
   sudo ln -s /etc/nginx/sites-available/pterodactyl /etc/nginx/sites-enabled/
   sudo systemctl restart nginx
   ```

5. **Akses Pterodactyl**:
   Akses `http://192.168.1.36` untuk menyelesaikan instalasi.

---

### **3. Setup Database Server**
1. **Install MySQL**:
   ```bash
   sudo apt install mysql-server -y
   ```

2. **Amankan MySQL**:
   ```bash
   sudo mysql_secure_installation
   ```

3. **Buat Database**:
   ```bash
   sudo mysql
   ```
   Jalankan perintah berikut:
   ```sql
   CREATE DATABASE pterodactyl;
   CREATE USER 'ptero_user'@'localhost' IDENTIFIED BY 'strongpassword';
   GRANT ALL PRIVILEGES ON pterodactyl.* TO 'ptero_user'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```

---

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
