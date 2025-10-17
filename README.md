![cockpit-banner](https://github.com/user-attachments/assets/c8d4daf1-86cc-45c9-be24-5c6a6a2ca8ca)
<a name="readme-top"></a>



<div align="center">
    
| [Pendahuluan](#pendahuluan-apa-itu-cockpit) | [Instalasi](#instalasi) | [Cara Penggunaan](#cara-penggunaan) | [Perbandingan](#perbandingan-headless-cms-cockpit-vs-payload-vs-strapi) | [Referensi](#referensi) |
|---|---|---|---|---|

</div>

---


# Pendahuluan: Apa itu Cockpit?
<p align="right"><a href="#readme-top">⬆️ kembali ke atas</a></p>

Cockpit adalah sebuah **headless CMS** yang dirancang untuk memberikan fleksibilitas penuh dalam membangun aplikasi berbasis konten sesuai dengan cara kerja Anda. Disebut "headless" karena Cockpit fokus pada penyediaan infrastruktur konten di *backend* dan menyerahkan tampilan atau *frontend* kepada Anda untuk dikembangkan secara bebas.
Baik Anda sedang membuat situs web, aplikasi seluler, ataupun aplikasi untuk IoT (*Internet of Things*), Cockpit menyediakan fondasi konten yang solid dan mudah dikelola melalui API.

# Instalasi
<p align="right"><a href="#readme-top">⬆️ kembali ke atas</a></p>

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

# Cara Penggunaan
<p align="right"><a href="#readme-top">⬆️ kembali ke atas</a></p>

#### 1. Login ke Dashboard
<p align="justify">
Langkah pertama adalah masuk ke dasbor admin Cockpit menggunakan username dan password yang telah Anda buat saat proses instalasi.
</p>

![Login ke Dasbor Cockpit](https://github.com/faqihfirman/hostingcockpitcms/blob/main/images/login-cockpit.png)


#### 2. Membuat Model Konten (Collection)
<p align="justify">
Setelah login, hal pertama yang harus dilakukan adalah mendefinisikan "wadah" atau struktur untuk konten Anda. Kita akan membuat sebuah <strong>Collection</strong> yang bisa menampung banyak item sejenis, contohnya untuk postingan blog.
</p>

- **Type**: Pilih `Collection`.
- **Name**: Beri nama internal untuk API (contoh: `postinganBlog`).
- **Display name**: Beri nama yang akan tampil di menu (contoh: `Postingan Blog`).

![Membuat Model Collection](https://github.com/faqihfirman/hostingcockpitcms/blob/main/images/create-model.png)


#### 3. Mendefinisikan Fields (Struktur Data)
<p align="justify">
Setelah model dibuat, kita perlu menentukan "laci" atau kolom data apa saja yang akan ada di dalamnya. Ini disebut <strong>Fields</strong>. Sebagai contoh, kita akan membuat sebuah field untuk menampung gambar.
</p>

- **Type**: Pilih `Asset` untuk menautkan file media.
- **Name**: Beri nama internal (contoh: `foto-blog`).
- **Display name**: Beri label yang akan tampil di form (contoh: `Foto Blog`).

![Membuat Fields untuk Model](https://github.com/faqihfirman/hostingcockpitcms/blob/main/images/create-fields.png)


#### 4. Mengunggah Aset (Gambar/Media)
<p align="justify">
Sebelum mengisi konten, ada baiknya kita mengunggah semua file media yang dibutuhkan ke dalam perpustakaan media Cockpit yang disebut <strong>Assets</strong>.
</p>

![Proses Unggah Aset Selesai](https://github.com/faqihfirman/hostingcockpitcms/blob/main/images/upload-item.png)


#### 5. Membuat Item Konten Baru
<p align="justify">
Sekarang saatnya mengisi konten. Masuk ke Collection yang sudah Anda buat (contoh: `Postingan Blog`), lalu buat item baru. Anda akan melihat form kosong dengan field yang sudah Anda definisikan sebelumnya.
</p>

![Membuat Item Baru di dalam Collection](https://github.com/faqihfirman/hostingcockpitcms/blob/main/images/select-asset.png)


#### 6. Menautkan Aset ke Item
<p align="justify">
Pada field gambar, klik `Link Asset`. Akan muncul jendela pop-up yang menampilkan semua media yang ada di perpustakaan Assets Anda. Pilih gambar yang Anda inginkan.
</p>

![Memilih Aset dari Perpustakaan Media](https://github.com/faqihfirman/hostingcockpitcms/blob/main/images/create-item-in-model.png)

<p align="justify">
Setelah dipilih, gambar akan tertaut ke item Anda. Jangan lupa untuk mengubah statusnya dari `UNPUBLISHED` menjadi `PUBLISHED` jika ingin data ini bisa diakses melalui API. Terakhir, klik `UPDATE ITEM`.
</p>

![Aset Berhasil Ditautkan ke Item](https://github.com/faqihfirman/hostingcockpitcms/blob/main/images/upload-asset.png)


#### 7. Melihat Item yang Telah Dibuat
<p align="justify">
Selamat! Anda telah berhasil membuat item konten pertama Anda. Item ini sekarang akan muncul dalam daftar di dalam Collection Anda.
</p>

![Daftar Item di dalam Collection](https://github.com/faqihfirman/hostingcockpitcms/blob/main/images/asset-uploaded.png)

# Perbandingan Headless CMS: Cockpit vs Payload vs Strapi
<p align="right"><a href="#readme-top">⬆️ kembali ke atas</a></p>

<p align="justify">
Pemilihan <em>headless Content Management System (CMS)</em> yang tepat sangat bergantung pada kebutuhan proyek, keahlian tim, dan filosofi pengembangan yang dianut. Berikut adalah perbandingan mendalam antara tiga headless CMS populer: Cockpit, Payload, dan Strapi, berdasarkan beberapa kriteria teknis dan fungsional.
</p>

| Pembeda | Cockpit | Payload | Strapi |
| :---: | :---: | :---: | :---: |
| **Teknologi Dasar** | PHP | NodeJS & TypeScript | NodeJS & JavaScript/TypeScript |
| **Filosofi Inti** | Minimalis, ringan, fleksibel, dan agnostik terhadap backend PHP/SQLite/MongoDB. | Code-First (Berbasis Kode) & Next.js-Native. Bertindak sebagai Full-stack Framework yang menyatu dengan React/Next.js. | GUI-First (Berbasis GUI) & Plugin-Oriented. Menawarkan pengalaman yang ramah pengguna dan ekosistem plugin yang luas. |
| **UI Admin Panel** | Fungsional, cukup sederhana. | Dibangun dengan React, sangat dapat disesuaikan (customizable) dengan komponen React Anda sendiri. | Dibangun dengan React, sangat intuitif untuk pengguna non-teknis dengan Content-Type Builder visual. |
| **Content Modelling** | Menggunakan antarmuka Admin untuk membuat Collections, Singletons, Trees. | Didefinisikan dalam kode TypeScript (Code-First), memastikan keamanan tipe data (type safety). | Dibuat melalui GUI (Content-Type Builder) di admin panel, namun dapat juga dimodifikasi via kode. |
| **Database Support** | SQLite (default) atau MongoDB. | MongoDB (default), tetapi juga mendukung PostgreSQL, MySQL, SQLite. | PostgreSQL, MySQL, SQLite, MongoDB. |
| **Akses API** | REST dan GraphQL | REST dan GraphQL (Sangat Dioptimalkan) | REST dan GraphQL (Diaktifkan melalui plugin) |
<br>

#### Kesimpulan Singkat

- **Gunakan Cockpit jika:** Anda membutuhkan solusi yang sangat ringan, cepat, dan sederhana dengan backend PHP, serta tidak memerlukan kustomisasi admin panel yang mendalam.
- **Gunakan Payload jika:** Anda adalah developer React/Next.js yang menyukai pendekatan *code-first* dan ingin kontrol penuh atas semua aspek CMS langsung dari kode TypeScript.
- **Gunakan Strapi jika:** Anda bekerja dengan tim yang memiliki anggota non-teknis, membutuhkan antarmuka admin yang sangat intuitif, dan menyukai ekosistem berbasis plugin.

# Referensi 
<p align="right"><a href="#readme-top">⬆️ kembali ke atas</a></p>
- Source Code : https://github.com/Cockpit-HQ/Cockpit




