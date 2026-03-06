# Prompt Pembuatan Aplikasi Telegram Mini App (Web App) untuk VPN Generator

Anda adalah seorang Software Engineer ahli yang sangat mahir dalam membuat bot Telegram menggunakan Node.js (Telegraf) dan mengembangkan Telegram Mini Apps (Web Apps) menggunakan HTML, CSS, dan JavaScript murni yang ringan dan responsif.

Tugas Anda adalah membuatkan saya *full-stack* source code untuk aplikasi Telegram Mini App (Web App) dengan spesifikasi, fitur, dan ketentuan API sebagai berikut:

## 1. Stack Teknologi
- **Backend Bot:** Node.js dengan framework `telegraf` versi 4.x.
- **Frontend Mini App:** Murni HTML, CSS, dan Vanilla JavaScript (tanpa framework besar seperti React/Vue agar aplikasi sangat ringan).
- **Database (Penyimpanan Data):** SQLite3 atau JSON file (pilih yang paling ringan dan mudah untuk menyimpan data pengguna, daftar server, dan riwayat).
- **Hosting API Backend:** Express.js (opsional, jika diperlukan untuk melayani halaman statis HTML/CSS/JS dari Mini App ke Telegram Web App).

## 2. Fitur Utama & Struktur Aplikasi
Aplikasi ini adalah sebuah "VPN Account Generator" yang langsung terhubung ke API server VPS saya. Aplikasi memiliki dua jenis pengguna: **Admin** dan **User Biasa**.

### A. Fitur User Biasa
- Saat user mengetik `/start`, bot membalas dengan pesan sambutan dan tombol *Inline Keyboard* bertuliskan "Buka Aplikasi VPN" (Web App Button).
- Ketika tombol ditekan, Telegram Mini App terbuka dan menampilkan *User Interface* (UI) yang berisi:
  - **Pilihan Tipe Akun:** SSH, Vmess, Vless, atau Trojan.
  - **Pilihan Server:** Dropdown daftar server aktif (diambil dari database).
  - **Form Input:** 
    - `Username` (teks)
    - `Password` (khusus SSH, teks) - *Catatan: Vmess/Vless/Trojan tidak perlu input password dari user*.
    - `Expired` (angka, dalam hari).
  - **Tombol "Create Account"**.
- Setelah form di-submit, frontend Mini App akan mengirim data ke Backend (Bot Telegram) menggunakan `Telegram.WebApp.sendData()` atau via fetch API.
- Backend bot akan memproses data tersebut, melakukan hit/request ke URL API Server VPS yang dipilih (menggunakan `axios` atau `node-fetch`), dan mengirimkan balasan langsung ke user di chat Telegram berupa detail akun VPN yang berhasil dibuat.
- **Batasan (Limit):** Jika kuota pembuatan akun harian pada server yang dipilih sudah habis (diatur oleh admin), user tidak bisa membuat akun di server tersebut.

### B. Fitur Admin (Halaman Khusus)
- Admin memiliki perintah khusus di bot (contoh: `/admin`) atau memiliki menu khusus (tab tersembunyi) di dalam Mini App jika ID Telegramnya cocok dengan ID Admin.
- **Manajemen Server:** Admin dapat Menambah (Add) dan Menghapus (Delete) server VPS ke dalam database.
  - Data server yang disimpan meliputi: `Nama Server`, `URL API Base`, `Auth Key` (parameter rahasia), dan `Limit Harian Akun`.
- **Manajemen Limit:** Admin bisa mengatur batas maksimal jumlah akun yang dapat dibuat pada masing-masing server setiap harinya.

## 3. Spesifikasi API Server VPS
Aplikasi backend (Node.js) harus mengirim *HTTP GET Request* ke API Server VPS saya. Server saya memiliki 4 endpoint untuk pembuatan akun.

*Catatan penting:*
Setiap request **wajib** menyertakan parameter query `auth=` yang berisi `Auth Key` dari server tersebut.

**A. Endpoint Create SSH**
- **Path:** `/createssh`
- **Parameter Query yang dibutuhkan:**
  - `auth`: (string) Auth Key server
  - `user`: (string) Username akun
  - `password`: (string) Password akun
  - `exp`: (number) Masa aktif dalam hari
  - `quota`: (number) Limit kuota GB (default: 0)
  - `iplimit`: (number) Limit IP login (default: 0)
- **Contoh Request:** `GET http://<URL_API>:5888/createssh?auth=RAHASIA&user=test&password=123&exp=30`

**B. Endpoint Create Vmess**
- **Path:** `/createvmess`
- **Parameter Query yang dibutuhkan:**
  - `auth`: (string) Auth Key server
  - `user`: (string) Username akun
  - `exp`: (number) Masa aktif dalam hari
  - `quota`: (number) Limit kuota GB (default: 0)
  - `iplimit`: (number) Limit IP login (default: 0)
- **Contoh Request:** `GET http://<URL_API>:5888/createvmess?auth=RAHASIA&user=test&exp=30`

**C. Endpoint Create Vless**
- **Path:** `/createvless`
- **Parameter Query yang dibutuhkan:** Sama persis dengan Vmess.

**D. Endpoint Create Trojan**
- **Path:** `/createtrojan`
- **Parameter Query yang dibutuhkan:** Sama persis dengan Vmess.

## 4. Respons API (JSON)
Jika berhasil, API Server VPS saya akan merespons dengan JSON yang memiliki struktur seperti ini:
```json
{
  "status": "success",
  "message": "Account created successfully",
  "data": {
    "username": "test",
    "domain": "vpnsaya.com",
    "expired": "2024-12-31",
    "ip_limit": "0",
    "quota": "0",
    ... (dan link akun VPN spesifik sesuai protokol)
  }
}
```
Bot Telegram Anda harus mem-parsing balasan JSON ini dan mengirimkannya kembali ke chat pengguna dalam format pesan HTML yang rapi.

## 5. Yang Harus Anda (AI) Buatkan Untuk Saya:
Tolong berikan saya kode lengkap yang terstruktur:
1.  **Struktur Folder & File:** Berikan struktur proyek (contoh: `bot.js`, `public/index.html`, `public/style.css`, `public/app.js`, `database.json`).
2.  **`package.json`**: Berisi dependencies yang diperlukan (seperti `telegraf`, `express`, `axios`, `cors`).
3.  **Kode Bot Backend (`bot.js`)**: Logika Telegraf, routing Express untuk hosting file Mini App (public folder), dan logika HTTP request ke API VPS saya.
4.  **Kode Frontend Mini App (`public/index.html`, `style.css`, `app.js`)**: Tampilan form yang terintegrasi dengan fungsi `window.Telegram.WebApp`. UI harus mendukung tema gelap (Dark Theme) mengikuti tema bawaan Telegram, terlihat elegan dan modern.

Tolong berikan semua *source code* tersebut dengan penjelasan cara menginstalnya dan menjalankannya di server Node.js saya!
