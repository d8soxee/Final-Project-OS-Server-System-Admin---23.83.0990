# Final-Project-OS-Server-System-Admin---23.83.0990

Repository ini saya gunakan untuk Dokumentasi Instalasi dan Konfigurasi layanan Server, seperti SSH, DNS, WEB, NTP, Mail Server dll. Beberapa Service yang dijelaskan dalam Repository ini masih dalam progres

#OPERATING SYSTEM
Ubuntu server 22.04.5 https://releases.ubuntu.com/jammy/ubuntu-22.04.5-desktop-amd64.iso

## 1. Instalasi dan Konfigurasi SSH Server

### Langkah Instalasi
sudo apt update
sudo apt install openssh-server

bash 
sudo apt update
sudo apt install openssh-server

### Konfigurasi Keamanan
sudo nano /etc/ssh/sshd_config

Ubah konfigurasi berikut:
- Nonaktifkan login root
- Batasi autentikasi password
- Gunakan kunci SSH

PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes

Restart layanan SSH:
sudo systemctl restart ssh
sudo systemctl enable ssh
