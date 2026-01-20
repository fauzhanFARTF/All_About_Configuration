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

File konfigurasi utama:

```text
/opt/homebrew/etc/httpd/httpd.conf
```

Buka dengan VS Code:

```bash
code /opt/homebrew/etc/httpd/httpd.conf
```

> ⚠️ Jika perintah `code` belum dikenali, ikuti dokumentasi **Install code command di PATH**.

➡️ **Lihat dokumentasi:**  
[Install code command di PATH](https://github.com/fauzhanFARTF/All_About_Configuration/blob/main/Install%20Custom%20Apache%20dan%20Nginx%20dalam%20MacOS.md#:~:text=Mengatasi%20Error%20%60zsh%3A-,command,-not%20found%3A%20code)

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
