# Final-Project-OS-Server-System-Admin---23.83.0990
Server's Name : MineShraft

Repository ini saya gunakan untuk Dokumentasi Instalasi dan Konfigurasi layanan Server, Web Server, Database Server, File Server, DNS Server, Proxy Server, Minecraft Server

# OPERATING SYSTEM
Ubuntu server 22.04.5 https://releases.ubuntu.com/jammy/ubuntu-22.04.5-desktop-amd64.iso

## 1. Setup SSH Server
SSH memungkinkan akses jarak jauh ke server.

### Install OpenSSH:

bash
Salin kode
sudo apt update
sudo apt install openssh-server -y
Konfigurasi SSH: Edit file konfigurasi:

'''bash
sudo nano /etc/ssh/sshd_config
'''

Pastikan parameter berikut:

plaintext
Salin kode
PermitRootLogin no
PasswordAuthentication yes
Restart layanan:

bash
Salin kode
sudo systemctl restart ssh
Uji Koneksi: Dari perangkat lain:

bash
Salin kode
ssh username@192.168.1.36
2. Setup Web Server untuk Pterodactyl
Panel Pterodactyl membutuhkan Nginx, PHP, dan Composer.

Install Nginx dan PHP:

bash
Salin kode
sudo apt install nginx php-cli php-fpm php-mysql php-zip php-curl php-xml php-bcmath php-mbstring unzip composer -y
Download dan Install Pterodactyl:

bash
Salin kode
cd /var/www
sudo mkdir pterodactyl
sudo chown -R www-data:www-data pterodactyl
sudo curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz
sudo tar -xzvf panel.tar.gz -C pterodactyl
cd pterodactyl
sudo composer install --no-dev --optimize-autoloader
Setup Database untuk Pterodactyl: (Lihat bagian Database Server di bawah).

Konfigurasi Nginx: Buat file konfigurasi:

bash
Salin kode
sudo nano /etc/nginx/sites-available/pterodactyl
Tambahkan:

plaintext
Salin kode
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
Aktifkan konfigurasi:

bash
Salin kode
sudo ln -s /etc/nginx/sites-available/pterodactyl /etc/nginx/sites-enabled/
sudo systemctl restart nginx
Akses Pterodactyl: Akses http://192.168.1.36 untuk menyelesaikan instalasi.

3. Setup Database Server
Install MySQL:

bash
Salin kode
sudo apt install mysql-server -y
Amankan MySQL:

bash
Salin kode
sudo mysql_secure_installation
Buat Database:

bash
Salin kode
sudo mysql
Jalankan perintah berikut:

sql
Salin kode
CREATE DATABASE pterodactyl;
CREATE USER 'ptero_user'@'localhost' IDENTIFIED BY 'strongpassword';
GRANT ALL PRIVILEGES ON pterodactyl.* TO 'ptero_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
4. Setup File Server
Gunakan Samba untuk berbagi file.

Install Samba:

bash
Salin kode
sudo apt install samba -y
Konfigurasi Samba: Edit file konfigurasi:

bash
Salin kode
sudo nano /etc/samba/smb.conf
Tambahkan:

plaintext
Salin kode
[SharedFiles]
path = /srv/shared
browsable = yes
writable = yes
guest ok = no
create mask = 0660
directory mask = 0771
Buat direktori dan atur izin:

bash
Salin kode
sudo mkdir -p /srv/shared
sudo chmod -R 775 /srv/shared
sudo systemctl restart smbd
5. Setup DNS Server
Gunakan Bind9 untuk layanan DNS.

Install Bind9:

bash
Salin kode
sudo apt install bind9 -y
Konfigurasi Zona DNS: Edit file:

bash
Salin kode
sudo nano /etc/bind/named.conf.local
Tambahkan:

plaintext
Salin kode
zone "mineshraft.local" {
    type master;
    file "/etc/bind/db.mineshraft.local";
};
Buat file zona:

bash
Salin kode
sudo cp /etc/bind/db.local /etc/bind/db.mineshraft.local
sudo nano /etc/bind/db.mineshraft.local
Edit:

plaintext
Salin kode
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
Restart Bind9:

bash
Salin kode
sudo systemctl restart bind9
6. Setup Proxy Server
Gunakan Squid untuk proxy server.

Install Squid:

bash
Salin kode
sudo apt install squid -y
Konfigurasi Squid: Edit file konfigurasi:

bash
Salin kode
sudo nano /etc/squid/squid.conf
Tambahkan:

plaintext
Salin kode
acl localnet src 192.168.1.0/24
http_access allow localnet
Restart Squid:

bash
Salin kode
sudo systemctl restart squid
7. Setup Dedicated Minecraft Server
Install Java:

bash
Salin kode
sudo apt install openjdk-17-jre -y
Download Minecraft Server:

bash
Salin kode
mkdir -p ~/minecraft
cd ~/minecraft
wget https://launcher.mojang.com/v1/objects/<latest_version>/server.jar
Jalankan Server:

bash
Salin kode
java -Xmx1024M -Xms1024M -jar server.jar nogui
