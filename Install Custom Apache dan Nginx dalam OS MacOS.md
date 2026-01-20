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

  Dapat dilihat di link ini

- ### Langkah 3: 🔴 Mematikan Apache Bawaan

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

#### Install Apache

```bash
brew install httpd
```

Tanpa opsi tambahan, httpd akan diinstal langsung dari binary yang telah dikompilasi, sehingga prosesnya cepat. Setelah selesai, Anda akan melihat pesan seperti:

```console
🍺  /opt/homebrew/Cellar/httpd/2.4.62: 1,664 files, 32.2MB
```

Langkah terakhir adalah mengatur agar Apache dari Homebrew otomatis berjalan saat sistem dinyalakan:

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

```bash
ps -aef | grep httpd
```

Jika Apache aktif, Anda akan melihat beberapa proses httpd dalam daftar.

Jika tidak muncul atau Anda curiga ada masalah, coba mulai ulang Apache:

```bash
brew services restart httpd
```

Untuk memantau log kesalahan secara real-time selama proses restart (atau saat mengakses server), buka tab atau jendela Terminal baru dan jalankan:

```bash
tail -f /opt/homebrew/var/log/httpd/error_log
```

Log ini sangat membantu untuk mengidentifikasi konfigurasi yang salah, izin file, atau masalah lainnya.

Karena Apache dikelola melalui brew services, berikut beberapa perintah berguna:

```bash
brew services stop httpd      # Menghentikan Apache
brew services start httpd     # Menjalankan Apache
brew services restart httpd   # Memulai ulang Apache
```

#### Konfigurasi Apache

Sekarang Anda memiliki server web yang berjalan, langkah berikutnya adalah menyesuaikan konfigurasinya agar lebih optimal untuk lingkungan pengembangan lokal.

Di versi terbaru Homebrew, Apache secara default berjalan di port 8080, bukan port standar HTTP (80). Untuk mengubahnya, Anda perlu mengedit file konfigurasi utama Apache:

```bash
/opt/homebrew/etc/httpd/httpd.conf
```

Jika Anda telah mengikuti langkah instalasi VS Code di atas, Anda bisa langsung membuka file tersebut dari Terminal dengan:

```bash
code /opt/homebrew/etc/httpd/httpd.conf
```

jika code belum teridentifikasi ikut langkah berikut di link ini
