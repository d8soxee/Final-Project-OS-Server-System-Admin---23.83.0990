# Final-Project-OS-Server-System-Admin---23.83.0990
Server's Name : MineShraft

Repository ini saya gunakan untuk Dokumentasi Instalasi dan Konfigurasi layanan Server, SSH Server, Web Server, Database Server, File Server, DNS Server, Proxy Server, Dedicated Server Minecraft 

# OPERATING SYSTEM
Ubuntu server 20.04.6 
---

### **Persiapan Awal**
1. **Install Ubuntu Server 20.04**:
   - Unduh dari [Ubuntu Server](https://ubuntu.com/download/server).
   - Saat instalasi, gunakan:
     - **Hostname**: `mineshraft`.
     - **IP Statik**: `192.168.1.36`.

   Contoh konfigurasi IP:
   ```plaintext
   Address: 192.168.1.36
   Netmask: 255.255.255.0
   Gateway: 192.168.1.1
   DNS: 8.8.8.8
   ```

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
   Pastikan parameter berikut:
   ```plaintext
   PermitRootLogin no
   PasswordAuthentication yes
   ```
   Restart layanan:
   ```bash
   sudo systemctl restart ssh
   ```

3. **Uji Koneksi**:
   Dari perangkat lain:
   ```bash
   ssh username@192.168.1.36
   ```

---

### **2. Setup Web Server untuk Pterodactyl**
Panel Pterodactyl membutuhkan **Nginx**, **PHP**, dan **Composer**.

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
