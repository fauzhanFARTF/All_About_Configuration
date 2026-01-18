# Install Custom Apache dan Nginx dalam MacOS

## ✨Pendahuluan

Pada macOS versi terbaru (termasuk macOS Sequoia 15.x), apache sebenarnya sudah tersedia secara bawaan (built-in). Namun, versi bawaan ini memiliki keterbatasan, terutama dalam hal fleksibilitas konfigurasi, manajemen versi PHP, dan integrasi dengan workflow modern seperti multi PHP version, Homebrew, serta kebutuhan pengembangan web masa kini.

Dokumentasi ini dibuat untuk menjelaskan cara menginstall dan mengkonfigurasi Apache dan Nginx secara custom menggunakan Homebrew pada macOS, sehingga:

- Apache dan Nginx dapat berjalan berdampingan (tidak saling bentrok).
- Mendukung multiple PHP versions (PHP 7.x, 8.x, dst) dan mudah berpindah versi.
- Root project dapat diarahkan ke direktori user, misalnya:

```console
  /Users/fauzannurrachman/Developer/index.php
```

- Developer dapat memilih web server (Apache atau Nginx) sesuai kebutuhan proyek.
- Lingkungan development menjadi lebih bersih, terkontrol, dan konsisten.

Panduan ini mengacu pada pendekatan modern menggunakan Homebrew, serta referensi dari artikel berikut:

```apache
macOS Sequoia: Apache with Multiple PHP Versions – getgrav.org
```

Namun dokumentasi ini tidak hanya menyalin, melainkan menyederhanakan dan menyesuaikan dengan kebutuhan developer yang ingin:

Menjalankan Apache dan Nginx secara paralel

- Mengelola PHP secara fleksibel
- Menghindari konflik port
- Menggunakan struktur project lokal yang rapi dan mudah dipindahkan

Dengan mengikuti dokumentasi ini, diharapkan setup web server di macOS menjadi lebih profesional, modular, dan siap untuk berbagai kebutuhan project, baik berbasis Apache maupun Nginx.

## ✨ Tujuan & Arsitektur Setup

### 📦 Tujuan

Setup ini bertujuan untuk menyediakan lingkungan web development fleksibel di macOS dengan karakteristik berikut:

- Apache dan Nginx berjalan berdampingan
- Mendukung multi PHP version (PHP 7.x / 8.x)
- Mudah berpindah web server sesuai kebutuhan project
- Root project berada di direktori user (bukan /Library/WebServer)
- Menghindari konflik port & service bawaan macOS

### 📦 Arsitektur Umum

- Apache → cocok untuk .htaccess, legacy app, CMS
- Nginx → cocok untuk API, high performance, reverse proxy
- PHP-FPM → shared service untuk Apache & Nginx

### 📦 File penting setelah instalasi:

- Konfigurasi utama: /opt/homebrew/etc/nginx/nginx.conf
- Folder root default: /opt/homebrew/var/www
- Log error: /opt/homebrew/var/log/nginx/error.log

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
│ Apache (port 80)   │
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

## Struktur Direktori Folder

```text
/Users/fauzannurrachman/Developer
├── index-apache.html
├── index-nginx.html
├── phpinfo.php
└── logs/
    ├── apache/
    └── nginx/
```

### index-apache.html

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>It works! - Apache HTTP Server</title>
    <style>
      body {
        font-family:
          -apple-system, BlinkMacSystemFont, 'Segoe UI', Helvetica, Arial,
          sans-serif;
        background-color: #f6f6f6;
        margin: 0;
        padding: 40px;
      }
      .container {
        background: #ffffff;
        padding: 30px;
        max-width: 720px;
        margin: auto;
        border-radius: 8px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
      }
      h1 {
        color: #b24926;
      }
      code {
        background: #f0f0f0;
        padding: 4px 6px;
        border-radius: 4px;
      }
      footer {
        margin-top: 30px;
        font-size: 14px;
        color: #777;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <h1>It works!</h1>

      <p>Apache HTTP Server is installed and working properly.</p>

      <p>This page is served from:</p>

      <p>
        <code>/Users/fauzannurrachman/Developer/</code>
      </p>

      <p>
        If you can see this page, Apache is successfully configured to serve web
        content.
      </p>

      <footer>
        <hr />
        <p>
          Apache HTTP Server<br />
          Homebrew on macOS
        </p>
      </footer>
    </div>
  </body>
</html>
```

### index-nginx

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Welcome to nginx!</title>
    <style>
      body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
      }
    </style>
  </head>
  <body>
    <h1>Welcome to nginx!</h1>
    <p>
      If you see this page, the nginx web server is successfully installed and
      working. Further configuration is required.
    </p>

    <p>
      For online documentation and support please refer to
      <a href="http://nginx.org/">nginx.org</a>.<br />
      Commercial support is available at
      <a href="http://nginx.com/">nginx.com</a>.
    </p>

    <p><em>Thank you for using nginx.</em></p>
  </body>
</html>
```

### info.php

```php
<?php phpinfo();
```

## 🛠 Cara Menginstal Apache & Nginx di macOS (via Homebrew)

- ### Langkah 1: Install Homebrew

  ```bash
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```

- ### Langkah 2: Pengecekan Apache Bawaan MacOS

  #### 1️⃣ Cek lewat Browser (cara paling cepat)

  Buka browser lalu akses:

  ```apache
  http://localhost
  ```

  Hasilnya:

  ✅ Muncul “It works!” atau halaman Apache default
  - Apache bawaan SEDANG HIDUP

  ❌ Tidak bisa diakses / connection refused
  - Apache bawaan MATI

  ⚠️ Catatan

  Kalau kamu sudah install Apache via Homebrew, yang tampil bisa Apache Homebrew, bukan bawaan. Jadi cara ini kurang akurat jika sudah ada Apache custom.

  #### 2️⃣ Cek Status Apache Bawaan via Terminal (Paling Akurat)

  Jalankan:

  ```bash
  sudo apachectl status
  ```

  Output umum:

  ✅ Jika hidup:

  ```console
  Apache Server Status for localhost
  Server Version: Apache/2.4.xx (Unix)
  ```

  ❌ Jika mati:

  ```console
  httpd not running
  ```

  3️⃣ Cek Apakah Apache Bawaan Sedang Running (Process Check)

  ```bash
  ps aux | grep httpd
  ```

  Jika muncul banyak proses httpd milik root:

  ```console
  root 123 0.0 httpd
  ```

  → kemungkinan Apache bawaan macOS aktif

  ⚠️ Catatan

  Apache Homebrew biasanya berjalan sebagai user kamu, bukan root.

  contoh

  ```console
  fauzannurrachman 38866   0.0  0.0 435299824   1312 s003  S+    8:52AM   0:00.01 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn --exclude-dir=.idea --exclude-dir=.tox --exclude-dir=.venv --exclude-dir=venv httpd
  ```

  4️⃣ Cek Service LaunchDaemon (CARA PALING VALID)

  Apache bawaan macOS dijalankan oleh launchd, bukan Homebrew.

  ```bash
  sudo launchctl list | grep apache
  ```

  Jika muncul:

  ```console
  org.apache.httpd
  ```

  → ✅ Apache bawaan HIDUP

  Jika tidak ada output:

  → ❌ Apache bawaan MATI

  5️⃣ Cek Port Default Apache Bawaan (Port 80)

  Apache bawaan selalu pakai port 80.

  ```bash
  lsof -i :80
  ```

  Jika hasilnya:

  ```console
  httpd PID root TCP \*:http (LISTEN)
  ```

  → ✅ Apache bawaan sedang running

  6️⃣ Cek File Konfigurasi Apache Bawaan

  Apache bawaan macOS SELALU pakai config ini:

  ```console
  /etc/apache2/httpd.conf
  ```

  Cek dengan:

```bash
  apachectl -V
```

Jika output mengandung:

```console
SERVER_CONFIG_FILE="/etc/apache2/httpd.conf"
```

→ itu Apache bawaan macOS

### Langkah 3: 🔴 Mematikan Apache Bawaan

Disarankan DIMATIKAN jika kamu pakai Apache Homebrew.

```bash
sudo apachectl stop
```

lalu Nonaktifkan permanen:

```bash
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```

atau

```bash
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
```

- sudo → jalankan sebagai root
- launchctl unload → hentikan service
- -w → menandai agar tidak otomatis jalan lagi saat boot
- org.apache.httpd.plist → Apache bawaan macOS
- 2 → stderr (error output)
- \> /dev/null → buang error ke 'tempat sampah' Error TIDAK ditampilkan di terminal

🔍 Jadi bedanya ?

| Aspek              | Tanpa `2>/dev/null` | Dengan `2>/dev/null` |
| ------------------ | ------------------- | -------------------- |
| Fungsi utama       | Sama                | Sama                 |
| Apache dimatikan   | ✅                  | ✅                   |
| Permanen disable   | ✅                  | ✅                   |
| Error ditampilkan  | ✅                  | ❌                   |
| Cocok untuk script | ❌                  | ✅                   |
| Terminal bersih    | ❌                  | ✅                   |

### Langkah 4: Install dan Config Apache

```bash
brew install httpd
```

Tanpa opsi tambahan, httpd akan diinstal langsung dari binary yang telah dikompilasi, sehingga prosesnya cepat. Setelah selesai, Anda akan melihat pesan seperti:

```console
🍺  /opt/homebrew/Cellar/httpd/2.4.62: 1,664 files, 32.2MB
```

```bash
brew services start httpd
```

Kini Anda telah berhasil menginstal Apache versi Homebrew dan mengonfigurasinya untuk otomatis berjalan dengan hak akses yang sesuai. Layanan Apache tersebut seharusnya sudah aktif. Untuk memverifikasi, buka browser dan kunjungi:

```apache
http://localhost:8080
```

Anda akan melihat halaman sederhana dengan tulisan "It works!" — tanda bahwa server Apache berjalan dengan baik.

Tips Pemecahan Masalah

Jika browser menampilkan pesan bahwa tidak dapat terhubung ke server, langkah pertama adalah memastikan bahwa server Apache benar-benar sedang berjalan.

Periksa proses Apache dengan perintah berikut:

### Langkah 1: Pastikan Homebrew Sudah Terinstal

📦 File penting setelah instalasi:

- Konfigurasi utama: /opt/homebrew/etc/nginx/nginx.conf
- Folder root default: /opt/homebrew/var/www
- Log error: /opt/homebrew/var/log/nginx/error.log

### Langkah 3: Jalankan Nginx

```bash
# Mulai sebagai layanan latar belakang (auto-start saat login)

brew services start nginx

# Atau jalankan manual (akan berhenti saat terminal ditutup)
nginx

```

### Langkah 4: Uji Instalasi

Buka browser dan kunjungi:

```nginx
http://localhost:8080
```

Anda akan melihat halaman selamat datang Nginx: "Welcome to nginx!"

⚠️ Port default Nginx di Homebrew adalah 8080, bukan 80 — agar tidak bentrok dengan layanan lain.

### Langkah 5 : 🔧 Ubah Port ke 80 (Opsional, untuk akses via http://localhost)

Jika ingin akses tanpa :8080, edit file konfigurasi:

```bash
code /opt/homebrew/etc/nginx/nginx.
```

Cari baris:

```nginx
listen     8080;
```

Ubah menjadi:

```nginx
listen       80;
```

Lalu restart Nginx:

```bash
brew services restart nginx
```

⚠️ Pastikan Apache tidak sedang berjalan!
Jalankan brew services stop httpd jika sebelumnya pakai Apache.

### Langkah 6: 📁 Ubah Document Root (Folder Proyek)

Secara default, Nginx menampilkan file dari

```apache
/opt/homebrew/var/www.
```

Untuk mengarahkannya ke folder Anda (misal ~/Sites):

- Buat folder Sites (jika belum ada):

  ```bash
  mkdir -p ~/Sites
  echo "<h1>My Nginx Site</h1>" > ~/Sites/index.html
  ```

- Edit nginx.conf:

  ```bash
  server {
  listen 80;
  server_name localhost;
  root /Users/fauzannurrachman/Sites; # ← ganti dengan username Anda
  index index.html index.htm;

      location / {
          try_files $uri $uri/ =404;
      }

  }
  ```

Restart Nginx:

```bash
brew services restart nginx
```

## 🧪 Perintah Berguna untuk Mengelola Nginx

| **Perintah**                                    | **Fungsi**                                                  |
| ----------------------------------------------- | ----------------------------------------------------------- |
| `brew services start nginx`                     | Menjalankan Nginx di latar belakang (auto start saat boot). |
| `brew services stop nginx`                      | Menghentikan layanan Nginx.                                 |
| `brew services restart nginx`                   | Memulai ulang Nginx.                                        |
| `nginx -t`                                      | Menguji validitas konfigurasi Nginx.                        |
| `tail -f /opt/homebrew/var/log/nginx/error.log` | Memantau log error Nginx secara real-time.                  |

## Permasalahan (⚠️ Penting: Jangan Jalankan Apache & Nginx Bersamaan di Port 80)

Menginstal Nginx tidak secara otomatis merusak atau membuat Apache error, namun keduanya bisa bentrok jika dijalankan bersamaan pada port yang sama — terutama port 80 (HTTP) atau 443 (HTTPS).

Keduanya tidak bisa berjalan bersamaan di port yang sama.
Jika Anda beralih antara keduanya, selalu hentikan yang tidak dipakai:

### Berikut penjelasan lengkapnya:

#### ✅ 1. Instalasi Saja Tidak Masalah

- Anda boleh menginstal Nginx dan Apache sekaligus di macOS (atau sistem lain).
- Proses instalasi via Homebrew (brew install nginx dan brew install httpd) tidak saling mengganggu.
- Keduanya akan tinggal di direktori terpisah dan tidak mengubah konfigurasi satu sama lain.

  🔧 Contoh lokasi

  ```bash
  Apache: /opt/homebrew/etc/httpd/
  Nginx: /opt/homebrew/etc/nginx/
  ```

#### ⚠️ 2. Masalah Terjadi Saat Keduanya DIJALANKAN di Port yang Sama

Port 80 (HTTP) hanya bisa dipakai oleh satu layanan dalam satu waktu.

Jika Anda:

Menjalankan Apache di port 80 → lalu
Menjalankan Nginx di port 80 juga,
Maka salah satu akan gagal start dengan error seperti:

```text
Address already in use
```

atau

```text
bind() to 0.0.0.0:80 failed (48: Address already in use)
```

#### ✅ Solusi: Jangan Jalankan Keduanya Secara Bersamaan

Anda punya beberapa pilihan:

- Opsi A: Gunakan salah satu saja (disarankan untuk pengembangan lokal)
  - Jika Anda sedang pakai Apache → hentikan Nginx:

  - Jika beralih ke Nginx → hentikan Apache:

    ```bash
    brew services stop nginx
    ```

- Jika beralih ke Nginx → hentikan Apache:

  ```bash
  brew services stop httpd
  ```

- Opsi B: Jalankan di port berbeda

  Misalnya:
  - Apache tetap di port 80
  - Nginx di port 8080

    ```apache
    Ubah konfigurasi Nginx (/opt/homebrew/etc/nginx/nginx.conf):
    ```

    Lalu akses via http://localhost:8080.

- Opsi C: Gunakan reverse proxy (lanjutan)

  Nginx di depan sebagai reverse proxy, meneruskan permintaan ke Apache di port berbeda (misal 8080).

  Ini jarang diperlukan untuk pengembangan lokal, tapi umum di produksi.

### 🔍 Cara Cek Layanan yang Sedang Pakai Port 80

```bash
sudo lsof -i :80
```

Output akan menunjukkan proses (Apache/Nginx) yang sedang mendengarkan di port 80.

## 💡 Kapan Harus Pakai Nginx?

Gunakan Nginx jika Anda:

- Mengembangkan aplikasi React, Vue, Angular (frontend modern)
- Membuat REST API dengan PHP/Laravel, Node.js, Python
- Ingin performa tinggi dan konsumsi memori rendah
- Tidak bergantung pada .htaccess
- Gunakan Apache jika Anda:
  - Masih pakai WordPress, Laravel lama, atau sistem yang butuh .htaccess
  - Lebih nyaman dengan modul seperti mod_php

## ✅ Kesimpulan

- Nginx = cepat, ringan, modern.
- Di macOS, instal via Homebrew (brew install nginx).
- Default port = 8080 → ubah ke 80 jika perlu.
- Jangan jalankan bersamaan dengan Apache di port yang sama.
- Sangat cocok untuk pengembangan web modern!
  Jika Anda ingin contoh konfigurasi untuk Laravel, WordPress, atau React, beri tahu — saya siap bantu!
- ❌ Tidak, menginstal Nginx tidak membuat Apache error.
- ✅ Tapi, menjalankan keduanya secara bersamaan di port yang sama akan menyebabkan konflik port, sehingga salah satu gagal jalan.
  Dengan begitu, lingkungan Anda tetap stabil dan bebas error.

```

```
