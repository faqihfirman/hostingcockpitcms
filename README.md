![cockpit-banner](https://github.com/user-attachments/assets/c8d4daf1-86cc-45c9-be24-5c6a6a2ca8ca)

# Pendahuluan: Apa itu Cockpit?

Cockpit adalah sebuah **headless CMS** yang dirancang untuk memberikan fleksibilitas penuh dalam membangun aplikasi berbasis konten sesuai dengan cara kerja Anda. Disebut "headless" karena Cockpit fokus pada penyediaan infrastruktur konten di *backend* dan menyerahkan tampilan atau *frontend* kepada Anda untuk dikembangkan secara bebas.
Baik Anda sedang membuat situs web, aplikasi seluler, ataupun aplikasi untuk IoT (*Internet of Things*), Cockpit menyediakan fondasi konten yang solid dan mudah dikelola melalui API.

# Instalasi

### 📋 Prasyarat Sistem (System Requirements)

<p align="justify">
Sebelum memulai proses instalasi, pastikan lingkungan server Anda memenuhi persyaratan minimum berikut:
</p>

- **PHP**: Versi `8.3` atau lebih baru, dengan ekstensi `PDO` dan `GD`.
- **Database**: `SQLite` (default, sudah termasuk dalam panduan ini) atau `MongoDB`.
- **Web Server**: `Apache` dengan modul `mod_rewrite` aktif atau `Nginx`.
- **Hak Akses**: Direktori `/storage` di dalam folder Cockpit harus bisa ditulis oleh server (`writable`).

### ⚙️ Tata Cara Instalasi (Step-by-Step)
<p align="justify">
Berikut adalah panduan instalasi lengkap, mulai dari pembuatan server hingga aplikasi siap digunakan.
</p>

#### 1. Pembuatan Virtual Machine (Google Cloud)
<p align="justify">
Langkah pertama adalah menyiapkan server. Dalam panduan ini, kita menggunakan Google Compute Engine.
</p>

- **Project**: `kdjk-p1-k3` (Sesuaikan dengan project Anda)
- **Region**: `asia-southeast1` (Contoh: Jakarta)
- **Zone**: `asia-southeast1-a`
- **Machine type**: `e2-small`
- **Boot disk**:
    - **Operating System**: `Ubuntu`
    - **Version**: `24.04 LTS Minimal`
    - **Size**: `10 GB`
- **Firewall**: Centang ✅ **Allow HTTP traffic** dan ✅ **Allow HTTPS traffic**.

<p align="justify">
Setelah VM dibuat, catat <strong>IP Publik</strong> yang diberikan. Kemudian, hubungkan ke VM menggunakan SSH.
</p>

#### 2. Konfigurasi Environment Server
<p align="justify">
Setelah masuk ke server melalui SSH, jalankan perintah berikut untuk menginstal semua perangkat lunak yang dibutuhkan.
</p>

```bash
# 1. Update package list
sudo apt update && \
\
# 2. Install Apache, PHP, extensions, and other tools in one command
sudo apt install -y \
  apache2 libapache2-mod-php php8.3 php8.3-cli php8.3-common \
  php8.3-gd php8.3-curl php8.3-zip php8.3-mbstring php8.3-xml \
  php8.3-bcmath php8.3-opcache php8.3-sqlite3 php8.3-redis \
  php-mongodb unzip zip sqlite3 wget nano composer && \
\
# 3. Enable required Apache modules
sudo a2enmod rewrite expires && \
\
# 4. Restart Apache to apply all changes
sudo systemctl restart apache2
\
# Beri pesan bahwa proses selesai
echo "=========================================="
echo "Instalasi environment server telah selesai!"
echo "=========================================="
```

#### 3. Unduh Cockpit dan atur hak akses
<p align="justify">
Pindah ke direktori web, unduh source code Cockpit dari GitHub, lalu atur kepemilikan dan hak akses folder agar bisa diakses oleh Apache.
</p>

```bash
# Pindah ke direktori root web server
cd /var/www

# Clone repository Cockpit ke dalam folder bernama 'cockpit'
sudo git clone [https://github.com/Cockpit-HQ/Cockpit.git](https://github.com/Cockpit-HQ/Cockpit.git) cockpit

# Ubah kepemilikan folder ke user 'www-data' (user default Apache di Ubuntu)
sudo chown -R www-data:www-data /var/www/cockpit

# Berikan hak akses tulis untuk direktori 'storage'
sudo chmod -R 755 /var/www/cockpit/storage
```

#### 4. Arahkan Domain ke Server (DNS Setup)
<p align="justify">
Masuk ke panel manajemen domain Anda (misalnya di Rumahweb) dan tambahkan <strong>A Record</strong> baru. Arahkan domain Anda (contoh: cockokin.my.id) 
ke alamat <strong>IP Publik</strong> VM yang sudah Anda catat sebelumnya.
</p>

#### 5. Konfigurasi Vistual Host Apache
<p align="justify">
Buat file konfigurasi Virtual Host untuk memberitahu Apache cara menyajikan situs Cockpit Anda. Ganti domain-anda.com dengan nama domain yang sebenarnya.
</p>

```bash
# Buat file konfigurasi baru untuk situs Cockpit
sudo tee /etc/apache2/sites-available/cockpit.conf >/dev/null <<'EOF'
<VirtualHost *:80>
    ServerName domain-anda.com
    ServerAlias [www.domain-anda.com](https://www.domain-anda.com)
    DocumentRoot /var/www/cockpit

    <Directory /var/www/cockpit>
        AllowOverride All
        Require all granted
        Options -Indexes
    </Directory>

    <DirectoryMatch "^.*/\.git/">
        Require all denied
    </DirectoryMatch>

    ErrorLog ${APACHE_LOG_DIR}/cockpit_error.log
    CustomLog ${APACHE_LOG_DIR}/cockpit_access.log combined
</VirtualHost>
EOF
```
<p align="justify">
Setelah file dibuat, aktifkan situs baru ini dan nonaktifkan situs default Apache.
</p>

```bash
# Aktifkan situs cockpit
sudo a2ensite cockpit

# Nonaktifkan situs default
sudo a2dissite 000-default

# Reload Apache untuk menerapkan konfigurasi
sudo systemctl reload apache2
```
#### 6. Amankan Situs dengan HTTPS
<p align="justify">
Instal Certbot untuk mendapatkan sertifikat SSL/TLS gratis dari Let's Encrypt dan mengaktifkan HTTPS.
</p>

```bash
# Instal Certbot dan plugin Apache-nya
sudo apt install -y certbot python3-certbot-apache

# Jalankan Certbot untuk mendapatkan sertifikat dan mengkonfigurasi Apache secara otomatis
# Ganti 'domain-anda.com' dengan domain Anda
sudo certbot --apache -d domain-anda.com -d [www.domain-anda.com](https://www.domain-anda.com)
```
<p align="justify">
Ikuti instruksi interaktif di layar untuk menyelesaikan proses ini.
</p>

#### 7. Selesaikan instalasi cockpit
<p align="justify">
Instalasi di sisi server sudah selesai! Sekarang, buka browser Anda dan kunjungi domain Anda (yang sudah HTTPS) diikuti dengan /install.
</p>

Contoh: https://domain-anda.com/install

<p align="justify">
Anda akan melihat halaman installer Cockpit. Ikuti petunjuk untuk membuat akun admin pertama Anda. Selamat, instalasi Cockpit Anda telah berhasil! 🎉
</p>







