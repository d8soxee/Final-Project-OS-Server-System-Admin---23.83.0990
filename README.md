# Final-Project-OS-Server-System-Admin---23.83.0990
Server's Name : MineShraft

Repository ini saya gunakan untuk Dokumentasi Instalasi dan Konfigurasi layanan Server, Web Server, Database Server, File Server, DNS Server, Proxy Server, Minecraft Server

# OPERATING SYSTEM
Ubuntu server 22.04.5 https://releases.ubuntu.com/jammy/ubuntu-22.04.5-desktop-amd64.iso

# 1. Instalasi dan Konfigurasi SSH Server

### Instalasi
```bash
sudo apt install openssh-server
```
### Konfigurasi 
```bash
sudo nano /etc/ssh/sshd_config
```

Ubah konfigurasi berikut:
- Nonaktifkan login root
- Batasi autentikasi password
- Gunakan kunci SSH

```bash
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

Restart layanan SSH:
```bash
sudo systemctl restart ssh
sudo systemctl enable ssh
```
# 2. Instalasi dan Konfigurasi DNS Server (Bind9)

### Instalasi Bind9
```bash
sudo apt update
sudo apt install bind9 bind9-utils
```

### Konfigurasi Zone
```bash
sudo nano /etc/bind/named.conf.local
```

Contoh konfigurasi zone:
```
zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
};
```

### Buat File Zone
```bash
sudo nano /etc/bind/db.example.com
```

### Restart DNS
```bash
sudo systemctl restart bind9
sudo systemctl enable bind9
```
