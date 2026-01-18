# Pengecekan Apache Bawaan macOS

Bagian ini digunakan untuk memastikan **apakah Apache bawaan macOS masih aktif atau tidak**, terutama jika kamu sudah menginstal Apache melalui Homebrew.

---

## 1️⃣ Cek Lewat Browser (Cara Paling Cepat)

Buka browser lalu akses:

```text
http://localhost
```

**Hasil yang mungkin muncul:**

- ✅ Muncul tulisan **"It works!"** atau halaman default Apache
  ➜ **Apache bawaan macOS SEDANG HIDUP**

- ❌ Tidak bisa diakses / _connection refused_
  ➜ **Apache bawaan macOS MATI**

⚠️ **Catatan Penting**
Jika kamu sudah menginstal Apache via **Homebrew**, halaman yang tampil **bisa berasal dari Apache Homebrew**, bukan bawaan macOS. Karena itu, metode ini **kurang akurat** jika sudah ada Apache custom.

---

## 2️⃣ Cek Status Apache Bawaan via Terminal (Paling Akurat)

Jalankan perintah berikut:

```bash
sudo apachectl status
```

**Output umum:**

- ✅ Jika hidup:

```console
Apache Server Status for localhost
Server Version: Apache/2.4.xx (Unix)
```

- ❌ Jika mati:

```console
httpd not running
```

Perintah ini secara langsung memeriksa **Apache bawaan macOS**.

---

## 3️⃣ Cek Apakah Apache Bawaan Sedang Running (Process Check)

Jalankan:

```bash
ps aux | grep httpd
```

Jika muncul banyak proses `httpd` milik **root**:

```console
root   123   0.0   httpd
```

➜ **Kemungkinan besar Apache bawaan macOS sedang aktif**

⚠️ **Catatan**
Apache **Homebrew** biasanya berjalan sebagai **user login**, bukan `root`.

**Contoh Apache Homebrew:**

```console
fauzannurrachman 38866   0.0  0.0 435299824   1312 s003  S+    8:52AM   0:00.01 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn --exclude-dir=.idea --exclude-dir=.tox --exclude-dir=.venv --exclude-dir=venv httpd
```

---

## 4️⃣ Cek Service LaunchDaemon (Cara Paling Valid)

Apache bawaan macOS **dijalankan oleh `launchd`**, bukan Homebrew.

Jalankan:

```bash
sudo launchctl list | grep apache
```

- Jika muncul:

```console
org.apache.httpd
```

➜ ✅ **Apache bawaan macOS HIDUP**

- Jika tidak ada output:

➜ ❌ **Apache bawaan macOS MATI**

---

## 5️⃣ Cek Port Default Apache Bawaan (Port 80)

Apache bawaan macOS **selalu menggunakan port 80**.

```bash
lsof -i :80
```

Jika hasilnya:

```console
httpd   PID   root   TCP *:http (LISTEN)
```

➜ ✅ **Apache bawaan sedang running**

---

## 6️⃣ Cek File Konfigurasi Apache Bawaan

Apache bawaan macOS **selalu menggunakan file konfigurasi berikut**:

```text
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

➜ **Itu adalah Apache bawaan macOS**

---

## 🔎 Perintah Tambahan (Opsional)

### Periksa Proses Apache

Gunakan perintah berikut untuk melihat apakah proses Apache (`httpd`) sedang berjalan:

```bash
ps -aef | grep httpd
```

Jika Apache aktif, kamu akan melihat beberapa proses `httpd` dalam daftar.
Jika hanya muncul proses `grep httpd`, berarti Apache **tidak sedang berjalan**.

**Contoh Output Jika Apache AKTIF:**

```text
root 120 1 0 08:10 ? 00:00:00 /usr/sbin/httpd
root 121 120 0 08:10 ? 00:00:00 /usr/sbin/httpd
root 122 120 0 08:10 ? 00:00:00 /usr/sbin/httpd
```

📌 **Artinya:**

- Apache sedang berjalan
- Proses `httpd` dijalankan oleh `root`
- Biasanya menandakan **Apache bawaan macOS aktif**

**Contoh Output Jika Apache TIDAK AKTIF:**

```text
fauzannurrachman 38901 0.0 0.0 428012 1240 s003 S+ 09:15AM 0:00.00 grep httpd
```

📌 **Artinya:**

- Tidak ada proses `httpd` yang berjalan
- Yang muncul hanya proses `grep`
- Apache **tidak sedang aktif**

---

## 📄 Monitoring Log Error Apache Homebrew (Detail)

Untuk memantau log kesalahan Apache Homebrew secara **real-time**, jalankan perintah berikut di Terminal (disarankan di tab/jendela terpisah):

```bash
tail -f /opt/homebrew/var/log/httpd/error_log
```

**Kegunaan log ini:**

- Mengetahui error konfigurasi (`httpd.conf`, virtual host, dll)
- Mendeteksi masalah izin file (_permission denied_)
- Mengidentifikasi konflik port
- Membantu debugging saat Apache gagal start atau restart

Tekan **Ctrl + C** untuk menghentikan monitoring log.

---

## 🔄 Jika Menggunakan Apache Homebrew

Jika Apache Homebrew bermasalah, coba restart:

```bash
brew services restart httpd
```

Pantau log error secara real-time:

```bash
tail -f /opt/homebrew/var/log/httpd/error_log
```

Log ini sangat membantu untuk mengidentifikasi kesalahan konfigurasi, izin file, atau konflik port.
