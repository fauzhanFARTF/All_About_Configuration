# Mengatasi Error `zsh: command not found: code` di macOS

Dokumen ini menjelaskan cara **mengaktifkan perintah `code` di Terminal macOS** agar kamu bisa membuka Visual Studio Code langsung dari command line.

Masalah ini biasanya muncul ketika **VS Code sudah terinstal**, tetapi **perintah `code` belum terdaftar di PATH shell**.

---

## ❌ Masalah yang Ditemui

Saat menjalankan perintah berikut di Terminal:

```console
code .
```

Muncul error:

```console
zsh: command not found: code
```

Artinya: **Terminal tidak mengenali perintah `code`**, meskipun Visual Studio Code sudah terinstal.

---

## ✅ Solusi: Menambahkan Perintah `code` ke PATH

Ikuti langkah-langkah di bawah ini satu per satu.

---

## ✅ Langkah 1: Pastikan Visual Studio Code Terinstal

Pastikan aplikasi **Visual Studio Code** sudah terpasang dan bisa dibuka secara normal melalui Finder atau Spotlight.

---

## ✅ Langkah 2: Buka Command Palette di VS Code

1. Buka **Visual Studio Code**.

2. Tekan kombinasi tombol:

   ```text
   Cmd + Shift + P
   ```

   untuk membuka **Command Palette**.

3. Ketik perintah berikut:

   ```text
   Shell Command: Install 'code' command in PATH
   ```

4. Pilih opsi tersebut lalu tekan **Enter**.

---

## 🔧 Apa yang Terjadi di Balik Layar?

VS Code akan otomatis:

- Membuat **symlink (shortcut)** perintah `code`
- Menaruhnya ke direktori PATH, biasanya:

```text
/usr/local/bin/code
```

Dengan begitu, Terminal (ZSH) bisa mengenali perintah `code` secara global.

---

## ✅ Langkah 3: Restart Terminal atau Reload Shell

Agar perubahan PATH terbaca, lakukan salah satu dari opsi berikut:

### Opsi 1: Tutup & Buka Kembali Terminal

Atau

### Opsi 2: Reload konfigurasi ZSH

Karena macOS modern menggunakan **ZSH** secara default, jalankan:

```bash
source ~/.zshrc
```

> ℹ️ Catatan:
>
> - File konfigurasi ZSH biasanya berada di `~/.zshrc`
> - Jika file tersebut belum ada, VS Code biasanya tetap menambahkan PATH ke profil shell yang aktif

---

## ✅ Langkah 4: Coba Jalankan Kembali

Di Terminal, masuk ke folder proyek kamu lalu jalankan:

```bash
code .
```

🎉 Jika berhasil, folder saat ini akan langsung terbuka di **Visual Studio Code**.

---

## ✅ Kesimpulan

- Error `command not found: code` **bukan berarti VS Code rusak**
- Masalahnya hanya karena **PATH belum dikonfigurasi**
- VS Code sudah menyediakan solusi resmi melalui Command Palette

Dengan langkah di atas, kamu bisa:

- Membuka project dengan cepat dari Terminal
- Meningkatkan workflow development (Docker, Python, Node.js, GIS, dll)

---

📌 **Tips Tambahan**

- Gunakan `code nama-folder` untuk membuka folder tertentu
- Gunakan `code file.py` untuk membuka file langsung
- Gunakan `code .` untuk membuka direktori aktif

Semoga membantu 🚀
