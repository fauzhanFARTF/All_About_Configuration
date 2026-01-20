# Install Custom Apache dan Nginx di macOS

## ✨ Pendahuluan

Pada macOS versi terbaru (termasuk **macOS Sequoia 15.x**), Apache sebenarnya sudah tersedia secara bawaan (_built-in_). Namun, Apache bawaan ini memiliki keterbatasan, terutama dalam hal:

- Fleksibilitas konfigurasi
- Manajemen **multiple PHP version**
- Integrasi dengan workflow modern (Homebrew, Docker, multi-project)

Dokumentasi ini dibuat untuk menjelaskan cara **menginstal dan mengonfigurasi Apache dan Nginx secara custom menggunakan Homebrew**, sehingga:

- Apache dan Nginx dapat berjalan **berdampingan tanpa konflik**
- Mendukung **multiple PHP version** (PHP 7.x, 8.x, dst)
- Root project dapat diarahkan ke direktori user, misalnya:

```text
/Users/fauzannurrachman/Developer/index.php
```

- Developer dapat memilih web server (**Apache atau Nginx**) sesuai kebutuhan proyek
- Lingkungan development lebih bersih, terkontrol, dan konsisten

Panduan ini terinspirasi dari artikel:

```text
macOS Sequoia: Apache with Multiple PHP Versions – getgrav.org
```

Namun dokumentasi ini telah **disederhanakan dan disesuaikan** untuk kebutuhan developer yang ingin:

- Menjalankan Apache & Nginx secara paralel
- Mengelola PHP secara fleksibel
- Menghindari konflik port
- Menggunakan struktur project lokal yang rapi

---

## ✨ Tujuan & Arsitektur Setup

### 📦 Tujuan

Setup ini bertujuan menyediakan lingkungan web development fleksibel di macOS dengan karakteristik:

- Apache dan Nginx berjalan berdampingan
- Mendukung multi PHP version
- Mudah berpindah web server per proyek
- Root project berada di direktori user
- Tidak bentrok dengan service bawaan macOS

### 📦 Arsitektur Umum

- **Apache** → cocok untuk `.htaccess`, legacy app, CMS
- **Nginx** → cocok untuk API, high performance, reverse proxy
- **PHP-FPM** → shared service untuk Apache & Nginx

### 📦 Lokasi File Penting (Nginx)

- Config utama: `/opt/homebrew/etc/nginx/nginx.conf`
- Root default: `/opt/homebrew/var/www`
- Log error: `/opt/homebrew/var/log/nginx/error.log`

---

## 🧩 Diagram Port Apache vs Nginx

Untuk menghindari konflik, masing-masing web server berjalan di port berbeda.

```text
APACHE
┌──────────────┐
│   Browser    │
└──────┬───────┘
       │
       │ http://localhost:80
       │
┌──────▼───────────────┐
│ Apache (port 80)     │
│ DocumentRoot:        │
│ /Users/.../Developer │
└────────┬─────────────┘
         │ SPHP
         ▼
     PHP 8.x / 7.x

NGINX
┌──────────────┐
│   Browser    │
└──────┬───────┘
       │
       │ http://localhost:8080
       │
┌──────▼───────────────┐
│ Apache (port 8080)   │
│ DocumentRoot:        │
│ /Users/.../Developer │
└────────┬─────────────┘
         │ PHP-FPM
         ▼
     PHP 8.x / 7.x
```

---

## 📂 Struktur Direktori Project

```text
/Users/fauzannurrachman/Developer
├── index-apache.html
├── index-nginx.html
├── phpinfo.php
└── logs/
    ├── apache/
    └── nginx/
```

---

## 🛠 Cara Menginstal Apache & Nginx di macOS (Homebrew)

### Langkah 1: Install Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

---

### Langkah 2: Pengecekan Apache Bawaan macOS dan Homebrew

Lakukan pengecekan Apache bawaan macOS terlebih dahulu untuk memastikan tidak terjadi konflik.

➡️ **Lihat dokumentasi:**  
[Langkah 2 – Pengecekan Apache Bawaan macOS dan Homebrew](https://github.com/fauzhanFARTF/All_About_Configuration/blob/main/Install%20Custom%20Apache%20dan%20Nginx%20dalam%20MacOS.md#:~:text=Pengecekan-,Apache,-Bawaan%20macOS)

---

### Langkah 3: 🔴 Mematikan Apache Bawaan macOS (Disarankan)

```bash
sudo apachectl stop
```

Nonaktifkan permanen:

```bash
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```

Atau versi bersih (tanpa error output):

```bash
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
```

**Penjelasan singkat:**

- `sudo` → jalankan sebagai root
- `launchctl unload` → hentikan service
- `-w` → nonaktif permanen
- `2>/dev/null` → sembunyikan pesan error

---

### Langkah 4: Install Apache (Homebrew)

```bash
brew install httpd
```

Aktifkan sebagai service:

```bash
brew services start httpd
```

Cek di browser:

```text
http://localhost:8080
```

Jika muncul **"It works!"**, Apache Homebrew sudah berjalan.

---

### 🔧 Troubleshooting Apache

Cek proses:

```bash
ps -aef | grep httpd
```

Restart service:

```bash
brew services restart httpd
```

Pantau log error:

```bash
tail -f /opt/homebrew/var/log/httpd/error_log
```

---

### Langkah 5: Konfigurasi Apache

Sekarang setelah web server kita berjalan dengan baik, hal berikutnya yang ingin kita lakukan adalah melakukan beberapa perubahan konfigurasi agar Apache bekerja lebih optimal sebagai local development server.

Pada versi terbaru Homebrew, kamu harus mengatur port listen secara manual dari default 8080 menjadi 80, sehingga kita perlu mengedit file konfigurasi Apache berikut:

File konfigurasi utama:

```text
/opt/homebrew/etc/httpd/httpd.conf
```

Jika kamu mengikuti instruksi sebelumnya, kamu seharusnya bisa menggunakan Visual Studio Code untuk mengedit file menggunakan perintah code di Terminal. Namun, jika ingin menggunakan aplikasi TextEditor bawaan macOS, kamu bisa memakai perintah open -e diikuti dengan path file.

1. Buka dengan VS Code:

   ```bash
   code /opt/homebrew/etc/httpd/httpd.conf
   ```

   > ⚠️ Jika perintah `code` belum dikenali, ikuti dokumentasi.

   ➡️ **Lihat dokumentasi:**
   [Install code command di PATH](https://github.com/fauzhanFARTF/All_About_Configuration/blob/main/Install%20Custom%20Apache%20dan%20Nginx%20dalam%20MacOS.md#:~:text=Mengatasi%20Error%20%60zsh%3A-,command,-not%20found%3A%20code)

2. Cari baris berikut:

   ```yaml
   Listen 8080
   ```

   Lalu ganti dengan

   ```yaml
   Listen 80
   ```

### Langkah 6 : Mengubah Document Root Apache

Selanjutnya, kita akan mengonfigurasi Document Root Apache. Ini adalah folder tempat Apache mengambil file yang akan disajikan.

Secara default, document root diatur ke:

```bash
/opt/homebrew/var/www

```

Karena ini adalah mesin pengembangan, kita akan mengubah document root agar menunjuk ke folder di dalam home directory kita.

Cari kata DocumentRoot, lalu kamu akan menemukan baris berikut:

```apache
DocumentRoot "/opt/homebrew/var/www"
```

Ubah menjadi (ganti your_user dengan username kamu):

```bash
DocumentRoot "/Users/fauzannurrachman/Developer"
```

Kamu juga perlu mengubah referensi tag \<Directory> tepat di bawah baris DocumentRoot agar sesuai dengan document root baru:

```apache
<Directory "/Users/fauzannurrachman/Sites">
```

Di dalam blok \<Directory> yang sama, kamu akan menemukan pengaturan AllowOverride. Ubah menjadi seperti berikut:

```graphql
#
# AllowOverride mengontrol directive apa saja yang boleh
# diletakkan di file .htaccess.
# Bisa berupa "All", "None", atau kombinasi keyword:
#   AllowOverride FileInfo AuthConfig Limit
#
AllowOverride All
```

### Langkah 7 : Mengaktifkan mod_rewrite

Selanjutnya, kita perlu mengaktifkan mod_rewrite yang secara default dalam keadaan dikomentari.

Cari mod_rewrite.so, lalu hapus tanda # di depannya (di VS Code bisa dengan menekan ⌘ + /):

```bash
LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so
```

### Langkah 8 : User & Group

Sekarang Apache sudah menunjuk ke folder Sites di home directory kita. Namun, masih ada satu masalah.

Secara default, Apache berjalan dengan user

```bash
\_www dan group \_www.
```

Ini bisa menyebabkan masalah izin (permission) saat mengakses file di home directory.

Sekitar sepertiga bagian atas file httpd.conf, cari pengaturan User dan Group, lalu ubah menjadi (ganti your_user dengan username kita):

```bash
User your_user
Group staff
```

contoh

```bash
User fauzannurrachman
Group staff
```

### Langkah 9 : ServerName

Apache membutuhkan ServerName dalam konfigurasi, namun secara default dinonaktifkan.

Cari baris berikut:

```grahql
#ServerName www.example.com:8080
```

Lalu ganti menjadi:

```bash
ServerName localhost
```

### Folder Developer

Sekarang, buat folder Sites di root home directory kamu. Ini bisa dilakukan lewat Terminal atau Finder.

Di dalam folder Developer, buat file index-apache.html sederhana dengan isi dummy seperti berikut:

➡️ **Lihat dokumentasi:**  
[File index-apache.html]()

Perintah Terminal:

mkdir ~/Sites
echo "<h1>My User Web Root</h1>" > ~/Sites/index.html

Restart Apache

Restart Apache agar perubahan konfigurasi diterapkan:

brew services stop httpd
brew services start httpd

Jika kamu mendapatkan error saat restart Apache, coba hapus tanda kutip (") pada pengaturan DocumentRoot dan <Directory> yang sebelumnya kita ubah.

Pengujian

Buka browser dan akses:

http://localhost

Jika berhasil, kamu akan melihat pesan baru yang tadi dibuat.

Pastikan kamu menghapus port :8080 yang sebelumnya digunakan. Selain itu, mungkin kamu perlu melakukan Shift + Reload untuk membersihkan cache browser agar file terbaru dimuat.

Jika semuanya sudah berjalan, kita bisa lanjut ke tahap berikutnya 🚀

---

## ✅ Penutup

Dengan setup ini:

- Apache & Nginx tidak saling bentrok
- PHP bisa dikelola multi versi
- Root project lebih rapi
- Setup siap untuk project skala kecil hingga kompleks

Dokumentasi selanjutnya akan membahas:

- Install Nginx
- Konfigurasi PHP-FPM
- Switch PHP version
- Virtual Host Apache & Server Block Nginx

```

```

```

```

```

```
