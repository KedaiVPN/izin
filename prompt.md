# Prompt Pembuatan Aplikasi Telegram Mini App (Web App) untuk VPN Generator

Anda adalah seorang Software Engineer ahli yang sangat mahir dalam membuat bot Telegram menggunakan Node.js (Telegraf) dan mengembangkan Telegram Mini Apps (Web Apps) menggunakan HTML, CSS, dan JavaScript murni yang ringan dan responsif.

Tugas Anda adalah membuatkan saya *full-stack* source code untuk aplikasi Telegram Mini App (Web App) dengan spesifikasi, fitur, dan ketentuan API sebagai berikut:

## 1. Stack Teknologi
- **Backend Bot:** Node.js dengan framework `telegraf` versi 4.x.
- **Frontend Mini App:** Murni HTML, CSS, dan Vanilla JavaScript (tanpa framework besar seperti React/Vue agar aplikasi sangat ringan).
- **Database (Penyimpanan Data):** **SQLite3** (menggunakan library `sqlite3` pada Node.js untuk menyimpan data pengguna, daftar server, dan pengaturan).
- **Hosting API Backend:** Express.js (untuk melayani halaman statis HTML/CSS/JS dari Mini App ke Telegram Web App dan memproses data dari frontend).

## 2. Fitur Utama & Struktur Aplikasi
Aplikasi ini adalah sebuah "VPN Account Generator" yang langsung terhubung ke API server VPS saya. Aplikasi memiliki dua jenis pengguna: **Admin** dan **User Biasa**.

### A. Fitur User Biasa (Frontend Mini App)
- Saat user mengetik `/start`, bot membalas dengan pesan sambutan dan tombol *Inline Keyboard* bertuliskan "Buka Aplikasi VPN" (Web App Button).
- Ketika tombol ditekan, Telegram Mini App terbuka dan menampilkan *User Interface* (UI) yang berisi:
  - **Pilihan Tipe Akun:** SSH, Vmess, Vless, atau Trojan.
  - **Pilihan Server:** Dropdown daftar server aktif (diambil dari database SQLite3).
  - **Form Input:** 
    - `Username` (teks, wajib diisi)
    - `Password` (khusus SSH, teks, wajib diisi) - *Catatan: Vmess/Vless/Trojan tidak perlu input password, jadi sembunyikan kolom ini jika user memilih selain SSH*.
  - **Verifikasi Keamanan:** Widget **Cloudflare Turnstile** ("Anda bukan robot") yang wajib diselesaikan sebelum tombol bisa ditekan.
  - **Tombol "Create Account"**.
- *Catatan Penting:* User **tidak dapat mengatur masa aktif (duration), kuota (quota), dan limit IP login (ipLimit)**. Parameter ini semuanya **diambil secara otomatis oleh sistem** berdasarkan konfigurasi server yang dipilih user (yang diatur oleh Admin).
- Setelah form di-submit dan Turnstile valid, frontend Mini App akan mengirim data JSON ke Backend route Express.js menggunakan fetch API (mengirim `userId`, `username`, `password`, `protocol`, `duration`, `quota`, `ipLimit`, dan `serverId`). Nilai `duration`, `quota`, dan `ipLimit` ini harus diambil frontend dari data server yang dipilih di dropdown, atau disuntikkan oleh backend.
- Setelah berhasil, bot Telegram akan mengirimkan balasan pesan langsung ke user di chat Telegram berisi detail akun VPN.

### B. Fitur Admin (Halaman Khusus)
- Admin memiliki perintah khusus di bot (contoh: `/admin`) atau memiliki menu khusus (tab tersembunyi) di dalam Mini App jika ID Telegramnya cocok dengan ID Admin.
- **Manajemen Server:** Admin dapat Menambah (Add), Mengedit (Edit), dan Menghapus (Delete) server VPS ke dalam database SQLite3.
  - **Flow Konfigurasi Server:** Saat admin menambahkan/mengedit server, admin wajib memasukkan data berikut:
    1.  `Nama Server`
    2.  `Domain Server` (contoh: vpnsaya.com atau server-upc.com)
    3.  `Auth Server` (parameter rahasia untuk otentikasi API)
    4.  `Masa Aktif (duration)` (dalam hari, diterapkan untuk semua akun di server ini)
    5.  `Limit IP per akun` (diterapkan untuk semua akun di server ini)
    6.  `Kuota per akun` (dalam GB, diterapkan untuk semua akun di server ini)

## 3. Spesifikasi Route API Backend (Express.js)
Untuk fungsi pembuatan akun, Anda **WAJIB** menggunakan kode *Express Router* persis seperti di bawah ini tanpa mengubah logika dasarnya. Kode ini menangani koneksi ke API server VPS saya berdasarkan protokol (ssh, vmess, vless, trojan).

```javascript
const express = require("express");
const axios = require("axios");
const sqlite3 = require("sqlite3").verbose();
const path = require("path");
const router = express.Router();

const dbPath = path.join(__dirname, "../db/miniapp.db");
const db = new sqlite3.Database(dbPath);

router.post("/", (req, res) => {
  const { userId, username, password, protocol, duration, quota, ipLimit, serverId } = req.body;

  if (!username || !protocol || !duration || !ipLimit || !serverId) {
    return res.status(400).json({ success: false, message: "Parameter tidak lengkap" });
  }

  db.get("SELECT * FROM Server WHERE id = ?", [serverId], async (err, server) => {
    if (err || !server) {
      return res.status(404).json({ success: false, message: "Server tidak ditemukan" });
    }

    // 🔑 Tentukan port berdasarkan pola domain
    const port = server.domain.includes("-upc.") ? 8443 : 5888;

    // 🔑 Susun endpoint API
    const endpoint = \`http://\${server.domain}:\${port}/create\${protocol}?user=\${username}\` +
      (protocol === "ssh" ? \`&password=\${password || "123"}\` : "") +
      \`&exp=\${duration}&quota=\${quota || 0}&iplimit=\${ipLimit}&auth=\${server.auth}\`;

    try {
      const response = await axios.get(endpoint);
      const data = response.data;

      if (data.status === "success") {
        // ✅ Simpan ke tabel User
        const stmt = db.prepare(\`
          INSERT INTO User (username, password, protocol, server_id, duration, quota, ip_limit)
          VALUES (?, ?, ?, ?, ?, ?, ?)
        \`);
        stmt.run(
          username,
          password || "123",
          protocol,
          serverId,
          duration,
          quota || 0,
          ipLimit
        );

        // ✅ Update jumlah akun dibuat
        db.run(\`
          UPDATE Server 
          SET total_create_akun = total_create_akun + 1 
          WHERE id = ?
        \`, [serverId]);

        return res.json({
          success: true,
          message: data.message,
          data: data.data
        });
      } else {
        return res.status(400).json({ success: false, message: data.message });
      }
    } catch (e) {
      console.error("API error:", e.message);
      return res.status(500).json({ success: false, message: "Gagal menghubungi API server" });
    }
  });
});

module.exports = router;
```

*Catatan untuk Anda (AI):* 
- Pastikan frontend mengirim JSON body dengan field `userId`, `username`, `password`, `protocol`, `duration`, `quota`, `ipLimit`, dan `serverId` ke route di atas.
- Struktur tabel database SQLite3 (`Server` dan `User`) harus Anda buat agar cocok dengan *query* di atas.

## 4. Respons ke Telegram
Jika respons dari route di atas sukses, bot Telegram harus mengirimkan pesan format HTML ke user Telegram berdasarkan `data.data` yang dikembalikan dari API server (yang berisi username, domain, expired, link vpn, dll).

## 5. Yang Harus Anda (AI) Buatkan Untuk Saya:
Tolong berikan saya kode lengkap yang terstruktur:
1.  **Struktur Folder & File:** Berikan struktur proyek (contoh: `bot.js`, `routes/create.js`, `db/database.js`, `public/index.html`, `public/style.css`, `public/app.js`).
2.  **Kode Database SQLite3 (`db/database.js`)**: Kode untuk membuat tabel `Server` dan tabel `User` sesuai dengan atribut di atas.
3.  **Kode Route (`routes/create.js`)**: Gunakan *kode murni* yang saya sediakan pada poin 3 di atas.
4.  **Kode Bot Backend (`bot.js`)**: Logika Telegraf, routing Express untuk hosting frontend, validasi Cloudflare Turnstile, dan interaksi webhook/pesan.
5.  **Kode Frontend Mini App (`public/index.html`, `style.css`, `app.js`)**: Tampilan form dengan integrasi widget **Cloudflare Turnstile** dan fungsi `window.Telegram.WebApp`. UI harus mendukung tema gelap (Dark Theme) mengikuti tema bawaan Telegram, elegan, rapi, responsif.
6.  **Instruksi Instalasi:** Berikan perintah npm install beserta environment variables yang harus disetel (seperti `BOT_TOKEN`, `TURNSTILE_SITE_KEY`, `TURNSTILE_SECRET_KEY`).
