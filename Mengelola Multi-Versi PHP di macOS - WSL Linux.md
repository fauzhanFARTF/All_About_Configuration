# Mengelola Multi-Versi PHP di macOS / WSL Linux

Dalam pengembangan web modern, seringkali diperlukan kemampuan untuk menguji aplikasi pada berbagai versi PHP—baik yang masih didukung maupun yang sudah usang namun masih digunakan di lingkungan lama. Panduan ini menjelaskan cara mengatur lingkungan pengembangan lokal di macOS menggunakan Homebrew untuk menginstal dan mengelola beberapa versi PHP secara bersamaan, mulai dari PHP 7.0 hingga PHP 8.4 (termasuk versi pengembangan).

Dengan memanfaatkan tap pihak ketiga seperti shivammathur/php dan skrip pengalih versi seperti sphp, pengembang dapat dengan mudah beralih antar versi PHP baik di terminal maupun di web server. Konfigurasi Apache disesuaikan agar mendukung eksekusi file .php dan mengenali index.php sebagai halaman utama, sementara modul PHP yang sesuai dimuat secara dinamis sesuai versi yang dipilih.

Pendekatan ini memberikan fleksibilitas tinggi tanpa mengorbankan stabilitas sistem, memungkinkan pengujian kompatibilitas, debugging legacy code, atau eksplorasi fitur terbaru PHP—semua dalam satu mesin pengembangan yang terintegrasi dan terkelola dengan rapi.

## 🔧 Instalasi PHP (MacOS / WSL Linux)

Jika Anda memiliki instalasi PHP yang sudah ada melalui Homebrew (Brew), Anda perlu membersihkan konfigurasi tersebut terlebih dahulu dengan mengikuti panduan Upgrading Homebrew kami sebelum melanjutkan bagian ini.

Hingga akhir Maret 2018, semua formula terkait PHP ditangani oleh tab Homebrew/php, namun tab tersebut kini telah dihentikan (deprecated). Sekarang kita menggunakan paket-paket yang tersedia di Homebrew/core. Meskipun paket-paket ini lebih terawat, jumlahnya jauh lebih sedikit dibandingkan sebelumnya.

PHP 7.0, 7.1, 7.2, 7.3, dan 7.4 telah dihentikan dukungannya dan dihapus dari Brew karena masa dukungan resminya telah berakhir. Meskipun tidak disarankan untuk digunakan di lingkungan produksi, ada alasan sah untuk menguji versi-versi lama ini di lingkungan pengembangan. Versi-versi tersebut juga perlu dikompilasi dari source agar kompatibel dengan versi terbaru dari icu4c dan openssl.

Saat ini, hanya PHP 8.3 yang merupakan versi stabil terbaru yang secara resmi didukung oleh Brew, tetapi harus dikompilasi dari source—proses ini cukup lambat. Dalam panduan terbaru ini, kami akan menggunakan tap baru dari @shivammahtur, karena menyediakan banyak versi PHP (termasuk PHP 8.3) dalam bentuk pre-built (sudah dikompilasi). PHP 8.4 juga tersedia, tetapi masih dalam tahap pengembangan.

```bash
brew tap shivammathur/php
```

Kami akan melanjutkan dengan menginstal berbagai versi PHP dan menggunakan skrip sederhana untuk berpindah-pindah antar versi sesuai kebutuhan. Silakan lewati versi mana pun yang tidak ingin Anda instal.

```bash
brew install shivammathur/php/php@7.0
brew install shivammathur/php/php@7.1
brew install shivammathur/php/php@7.2
brew install shivammathur/php/php@7.3
brew install shivammathur/php/php@7.4
brew install shivammathur/php/php@8.0
brew install shivammathur/php/php@8.1
brew install shivammathur/php/php@8.2
brew install shivammathur/php/php@8.3
brew install shivammathur/php/php@8.4
```

PENTING: PHP 8.4 masih dalam tahap pengembangan… PHP 8.3 saat ini adalah versi stabil terbaru.

Anda mungkin juga perlu menyesuaikan pengaturan konfigurasi PHP sesuai kebutuhan. Pengaturan umum yang sering diubah adalah memory_limit atau date.timezone. File php.ini untuk setiap versi PHP berada di direktori berikut:

```console
/opt/homebrew/etc/php/7.0/php.ini
/opt/homebrew/etc/php/7.1/php.ini
/opt/homebrew/etc/php/7.2/php.ini
/opt/homebrew/etc/php/7.3/php.ini
/opt/homebrew/etc/php/7.4/php.ini
/opt/homebrew/etc/php/8.0/php.ini
/opt/homebrew/etc/php/8.1/php.ini
/opt/homebrew/etc/php/8.2/php.ini
/opt/homebrew/etc/php/8.3/php.ini
/opt/homebrew/etc/php/8.4/php.ini
```

Pada titik ini, sangat disarankan untuk menutup SEMUA tab dan jendela terminal Anda. Buka terminal baru untuk melanjutkan langkah berikutnya. Hal ini sangat penting karena masalah aneh terkait PATH bisa muncul pada terminal yang sudah terbuka sebelumnya (percayalah, saya pernah mengalaminya!).

Kami telah menginstal berbagai versi PHP, tetapi belum membuat link-nya. Untuk beralih ke PHP 8.3, misalnya, Anda bisa menjalankan:

```bash
brew unlink php && brew link --overwrite --force php@8.3
```

Lalu uji versi yang aktif:

```bash
php -v
```

Output contoh:

```console
PHP 8.3.11 (cli) (built: Aug 27 2024 19:16:34) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.3.11, Copyright (c) Zend Technologies
    with Zend OPcache v8.3.11, Copyright (c), by Zend Technologies
```

Dan untuk beralih ke PHP 8.2:

```bash
brew unlink php && brew link --overwrite --force php@8.2
```

Lalu periksa lagi:

```bash
php -v
```

Output contoh:

```console
PHP 8.2.23 (cli) (built: Aug 27 2024 15:32:20) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.2.23, Copyright (c) Zend Technologies
    with Zend OPcache v8.2.23, Copyright (c), by Zend Technologiesx
```

## 🔧 Konfigurasi PHP untuk Apache (MacOS / WSL Linux)

Anda telah berhasil menginstal berbagai versi PHP, tetapi Apache belum tahu versi mana yang harus digunakan. Anda perlu mengedit file konfigurasi Apache:
/opt/homebrew/etc/httpd/httpd.conf

Gulir ke bagian bawah daftar LoadModule. Jika Anda mengikuti panduan ini dengan benar, entri terakhir seharusnya adalah modul mod_rewrite:

```bash
LoadModule rewrite_module lib/httpd/modules/mod_rewrite.so
```

Di bawahnya, tambahkan modul-modul libphp berikut (dalam keadaan terkomentari):

```apache
#LoadModule php7_module /opt/homebrew/opt/php@7.0/lib/httpd/modules/libphp7.so
#LoadModule php7_module /opt/homebrew/opt/php@7.1/lib/httpd/modules/libphp7.so
#LoadModule php7_module /opt/homebrew/opt/php@7.2/lib/httpd/modules/libphp7.so
#LoadModule php7_module /opt/homebrew/opt/php@7.3/lib/httpd/modules/libphp7.so
#LoadModule php7_module /opt/homebrew/opt/php@7.4/lib/httpd/modules/libphp7.so
#LoadModule php_module /opt/homebrew/opt/php@8.0/lib/httpd/modules/libphp.so
#LoadModule php_module /opt/homebrew/opt/php@8.1/lib/httpd/modules/libphp.so
LoadModule php_module /opt/homebrew/opt/php@8.2/lib/httpd/modules/libphp.so
#LoadModule php_module /opt/homebrew/opt/php@8.3/lib/httpd/modules/libphp.so
#LoadModule php_module /opt/homebrew/opt/php@8.4/lib/httpd/modules/libphp.so
```

Hanya satu modul PHP yang boleh aktif sekaligus. Saat ini, kami biarkan php@8.2 aktif (tidak dikomentari), sementara yang lain dikomentari. Ini memberi tahu Apache untuk menggunakan PHP 8.2 dalam menangani permintaan PHP. (Kemampuan untuk berpindah versi akan ditambahkan nanti.)

Selain itu, Anda juga harus mengatur Directory Index untuk mendukung file index.php. Cari blok berikut:

```bash
<IfModule dir_module>
DirectoryIndex index.html
</IfModule>
```

Ganti dengan:

```bash
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>

<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>
```

Simpan file tersebut, lalu hentikan dan mulai ulang Apache:

```bash
brew services stop httpd
brew services start httpd
```

### 🔧 Memverifikasi Instalasi PHP (MacOS / WSL Linux)

Cara terbaik untuk menguji apakah PHP berjalan dengan benar adalah dengan menggunakan fungsi phpinfo(). Fungsi ini tidak boleh dibiarkan di server produksi, tetapi sangat berguna di lingkungan pengembangan.

Buat file bernama info.php di folder Developer/ Anda:

```bash
echo "<?php phpinfo();" > ~/Developer/info.php
```

Anda seharusnya melihat halaman informasi PHP yang lengkap.

Jika tampilan mirip dengan yang diharapkan, selamat! Apache dan PHP Anda sudah berjalan dengan baik. Anda bisa menguji versi PHP lain dengan mengomentari baris LoadModule untuk php@8.2 dan mengaktifkan salah satu versi lain, lalu restart Apache dan muat ulang halaman tersebut.

## 🔧 Konfigurasi PHP untuk Nginx (MacOS / WSL Linux)

Jika Anda menggunakan Nginx (bukan Apache) di macOS atau WSL Linux, konfigurasi untuk menjalankan PHP tidak melibatkan LoadModule atau libphp.so, karena Nginx tidak menyertakan dukungan PHP secara langsung seperti Apache. Sebagai gantinya, Nginx berkomunikasi dengan PHP melalui PHP-FPM (FastCGI Process Manager).

Berikut adalah panduan penyesuaian untuk Nginx + PHP multi-versi:

### 🔧 Konfigurasi PHP untuk Nginx – Bagian 1 (macOS / WSL Linux)

Anda telah menginstal berbagai versi PHP melalui shivammathur/php. Untuk menggunakannya dengan Nginx, Anda perlu:

- Menjalankan PHP-FPM untuk versi PHP yang diinginkan
- Mengonfigurasi Nginx agar meneruskan permintaan .php ke socket atau port PHP-FPM

#### ✅ Langkah 1: Jalankan PHP-FPM untuk Versi Tertentu

Setiap versi PHP dari tap shivammathur/php menyertakan layanan php-fpm. Misalnya, untuk menjalankan PHP 8.2:

```bash
brew services start shivammathur/php/php@8.2
```

⚠️ Pastikan hanya satu versi PHP-FPM yang aktif sekaligus, karena mereka biasanya menggunakan socket yang sama (/opt/homebrew/var/run/php-fpm.sock) atau port yang bentrok.

Jika Anda ingin menghindari konflik, Anda bisa:

- Menghentikan versi lama terlebih dahulu

```bash
brew services stop php@8.1
brew services start php@8.2
```

- Atau ubah konfigurasi php-fpm.conf tiap versi agar menggunakan socket berbeda (opsional, lebih lanjut).

#### ✅ Langkah 2: Konfigurasi Nginx

Edit file server block Anda (misalnya di /opt/homebrew/etc/nginx/servers/developer.conf):

```nginx
server {
    listen 80;
    server_name localhost;
    root /Users/fauzannurrachman/Developer;  # sesuaikan path
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        fastcgi_pass   unix:/opt/homebrew/var/run/php-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

💡 Pastikan jalur root sesuai dengan direktori proyek Anda (misalnya ~/Developer).

#### ✅ Langkah 3: Restart Nginx

```bash
brew services restart nginx
```

#### ✅ Langkah 4: Uji Instalasi PHP

Buat file uji di folder Developer:

```bash
echo "<?php phpinfo();" > ~/Developer/info.php
```

Lalu buka browser ke:

👉 http://localhost/info.php

Jika muncul halaman phpinfo(), berarti Nginx berhasil berkomunikasi dengan PHP-FPM!

### 🔄 Beralih Versi PHP di Nginx

Karena Nginx tidak "terikat" ke versi PHP tertentu (ia hanya mengarahkan ke socket), Anda cukup:

Hentikan PHP-FPM versi lama
Mulai PHP-FPM versi baru
(Opsional) Restart Nginx jika ada perubahan socket
Contoh beralih ke PHP 8.3:

```bash
brew services stop php@8.2
brew services start php@8.3
```

## Tidak perlu restart Nginx jika socket tetap sama

🔍 Catatan: Semua versi PHP dari shivammathur/php menggunakan socket yang sama secara default:

```bash
/opt/homebrew/var/run/php-fpm.sock
```

Jadi selama hanya satu php-fpm yang jalan, Nginx akan otomatis menggunakan versi tersebut.

### 🔧 Skrip Pengganti Versi PHP (PHP Switcher Script) (MacOS / WSL Linux)

Saat ini, Apache dikonfigurasi secara manual untuk menggunakan PHP 8.2. Namun, kita ingin bisa beralih versi dengan mudah. Untungnya, ada skrip siap pakai yang sangat membantu.

Instal skrip sphp ke direktori standar Brew:

```bash
curl -L https://raw.githubusercontent.com/rhukster/sphp.sh/main/sphp.sh > /opt/homebrew/bin/sphp
chmod +x /opt/homebrew/bin/sphp
```

### Menguji Penggantian Versi PHP (MacOS / WSL Linux)

Setelah langkah-langkah di atas selesai, Anda bisa mengganti versi PHP dengan perintah:

```bash
sphp 8.3
```

Anda mungkin diminta memasukkan password administrator. Output yang muncul kira-kira seperti ini:

```console
Switching to php@8.3
Switching your shell
Unlinking /opt/homebrew/Cellar/php@8.2/8.2.23... 25 symlinks removed.
Unlinking /opt/homebrew/Cellar/php/8.3.11... 0 symlinks removed.
Unlinking /opt/homebrew/Cellar/php@8.4/8.4.0_1... 0 symlinks removed.
Linking /opt/homebrew/Cellar/php/8.3.11... 24 symlinks created.
Switching your apache conf
Restarting apache
Stopping `httpd`... (might take a while)
==> Successfully stopped `httpd` (label: homebrew.mxcl.httpd)
==> Successfully started `httpd` (label: homebrew.mxcl.httpd)

PHP 8.3.11 (cli) (built: Aug 27 2024 19:16:34) (NTS)
...
All done!
```

Uji kembali dengan membuka http://localhost/info.php di browser. Jika berhasil, Anda akan melihat informasi PHP 8.3.

### Pemecahan Masalah (Troubleshooting)

- Jika muncul pesan:
  Unknown version of PHP. PHP Switcher can only handle arguments of: 5.6,7.0,...,8.4
  - → Perbarui skrip sphp dengan mengunduh ulang versi terbaru.
- Jika muncul konflik versi PHP:
  - → Kemungkinan besar Anda masih memiliki tap lama. Hapus tap lama, lalu pasang tap baru:

```bash
brew untap exolnet/deprecated
brew tap shivammathur/php
brew install shivammathur/php/php@8.0
```

### Memperbarui PHP dan Paket Brew Lainnya

Memperbarui sangat mudah:

```bash
brew update      # perbarui daftar paket
brew upgrade     # perbarui semua paket
```

Namun, karena hanya satu versi PHP yang aktif sekaligus, Anda perlu beralih ke setiap versi PHP yang terinstal dan jalankan brew upgrade untuk masing-masing agar semuanya tetap mutakhir.

Untuk melihat versi PHP yang sedang aktif:

```bash
php -v
```

Untuk melihat detail versi tertentu:

```bash
brew info php@8.3
```

## Permasalahan

Jika Anda sudah menjalankan perintah brew unlink dan brew link --overwrite --force php@X.Y, tetapi php -v masih menunjukkan versi lama, kemungkinan besar disebabkan oleh salah satu dari hal berikut:

### 🔍 1. Terminal Belum Memuat Ulang PATH yang Baru

Setelah mengganti symlink PHP via brew link, shell (terminal) Anda mungkin masih menggunakan cache path lama.

#### ✅ Solusi:

- Tutup semua tab dan jendela terminal, lalu buka terminal baru.
  Atau, muat ulang environment dengan:
  bash
  123
  source ~/.zshrc # jika pakai ZSH (default di macOS Catalina+)

# atau

source ~/.bashrc # jika pakai Bash
💡 Catatan: Di macOS modern, shell default adalah ZSH, bukan Bash.

🔍 2. Masih Ada PHP Lain di PATH (Misalnya dari XAMPP, MAMP, atau instalasi manual)
Jalankan perintah berikut untuk melihat lokasi sebenarnya dari binary php yang digunakan:

bash
1
which php
Jika hasilnya bukan /opt/homebrew/bin/php, misalnya:

/usr/bin/php → ini adalah PHP bawaan macOS (biasanya sangat tua)
/Applications/MAMP/bin/php/... → dari MAMP
/usr/local/bin/php → mungkin dari instalasi Homebrew lama (Intel Mac)
✅ Solusi:
Pastikan /opt/homebrew/bin ada di awal PATH Anda.

Cek PATH:

bash
1
echo $PATH
Harus ada bagian seperti:

1
/opt/homebrew/bin:/opt/homebrew/sbin:...
Jika tidak, tambahkan ke file shell Anda (~/.zprofile atau ~/.zshrc):

bash
12
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
source ~/.zprofile
Lalu coba lagi:

bash
12
which php # harus: /opt/homebrew/bin/php
php -v # harus sesuai versi yang dilink
🔍 3. PHP Tidak Benar-Benar Dilink dengan Benar
Cek apakah symlink benar-benar mengarah ke versi yang diinginkan:

bash
1
ls -l /opt/homebrew/bin/php
Output contoh yang benar:

1
lrwxr-xr-x 1 fauzannurrachman admin 36 Jan 16 10:00 /opt/homebrew/bin/php -> ../Cellar/php@8.3/8.3.11/bin/php
Jika masih mengarah ke versi lama (misalnya php@8.2), maka brew link belum berhasil.

✅ Solusi:
Ulangi proses linking secara eksplisit:

bash
12
brew unlink php@8.2 # ganti dengan versi lama yang aktif
brew link --overwrite --force php@8.3
⚠️ Jika muncul error "Permission denied", pastikan Anda tidak perlu sudo — Homebrew di macOS modern diinstal di user space.

🔍 4. Anda Menggunakan sphp tapi CLI Tidak Terupdate
Skrip sphp memang mengatur:

Symlink CLI (/opt/homebrew/bin/php)
Konfigurasi Apache
Restart layanan
Tapi kadang shell tidak langsung mengenali perubahan symlink karena caching internal.

✅ Solusi:
Setelah sphp 8.3, jalankan:

bash
1
Perintah ini membersihkan cache path executable di shell Anda.

Lalu cek:

bash
1
php -v
✅ Ringkasan Langkah Cepat
bash
123456789101112131415

# 1. Pastikan PATH benar

eval "$(/opt/homebrew/bin/brew shellenv)"

# 2. Bersihkan cache shell

hash -r

# 3. Cek lokasi php

which php # harus /opt/homebrew/bin/php

# 4. Jika perlu, force link ulang

Jika semua langkah di atas sudah dilakukan dan php -v tetap salah, kemungkinan besar ada konflik instalasi PHP dari sumber lain. Dalam kasus seperti itu, pertimbangkan untuk:

Hapus MAMP/XAMPP sementara
Atau gunakan sphp yang memang dirancang untuk menangani multi-versi dengan aman
Semoga membantu!

## Penutup

Dengan skrip sphp, beralih antar versi PHP kini menjadi semudah mengetik satu perintah—tanpa perlu mengedit konfigurasi manual berulang kali atau khawatir tentang inkonsistensi antara CLI dan Apache. Ini tidak hanya menghemat waktu, tetapi juga meminimalkan risiko kesalahan konfigurasi saat bekerja pada proyek yang membutuhkan versi PHP berbeda.

Lingkungan pengembangan Anda sekarang fleksibel, terkelola, dan siap menangani berbagai skenario: dari pemeliharaan aplikasi lama yang masih bergantung pada PHP 7.x, hingga eksplorasi fitur mutakhir di PHP 8.4 (development). Semua ini berjalan lancar di atas fondasi Apache dan Homebrew yang stabil, memberi Anda kontrol penuh tanpa mengorbankan kenyamanan.

Langkah selanjutnya? Pertimbangkan untuk melengkapi stack pengembangan ini dengan database (seperti MariaDB atau PostgreSQL), virtual host untuk proyek-proyek terpisah, serta debugger seperti Xdebug—sehingga Anda benar-benar memiliki workstation pengembangan lokal yang profesional, modular, dan siap produksi.

## Referensi

- [shivammathur/homebrew-php](https://github.com/shivammathur/homebrew-php) – Tap Homebrew untuk multi-versi PHP
- [macOS Sequoia: Apache + Multiple PHP Versions](https://getgrav.org/blog/macos-sequoia-apache-multiple-php-versions) – Grav Official Blog
- [PHP.net](https://www.php.net/) – Dokumentasi & rilis resmi PHP
- [Homebrew Docs](https://docs.brew.sh/) – Panduan penggunaan Homebrew
