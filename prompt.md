# Prompt untuk Membuat REST API Manajemen Akun dengan Integrasi Telegram

Salin dan tempel prompt di bawah ini ke AI (seperti ChatGPT atau Claude) untuk menghasilkan kode yang Anda butuhkan.

---

**Prompt:**

Tolong buatkan saya sistem REST API lengkap untuk mengelola script manajemen akun saya yang berbasis Bash. Saya ingin menggunakan **Node.js (Express)** sebagai server API-nya.

Berikut adalah spesifikasi lengkap yang saya butuhkan:

**1. Arsitektur & Cara Kerja:**
*   Server API berjalan di port **5888**.
*   API ini berfungsi sebagai wrapper yang menjalankan perintah bash dari script utama saya (anggap saja nama script utama saya `myscript.sh` yang ada di `/usr/local/bin/myscript.sh`).
*   Setiap request ke API harus menyertakan query parameter `?auth=KEY_RAHASIA`.

**2. Endpoint API:**
Tolong buatkan endpoint berikut yang akan mengeksekusi perintah bash di server:
*   **Buat Akun:** `GET /create/user?user=nama&exp=hari&auth=KEY`
    *   Menjalankan: `myscript.sh create <user> <exp>`
*   **Hapus Akun:** `GET /delete/user?user=nama&auth=KEY`
    *   Menjalankan: `myscript.sh delete <user>`
*   **Perpanjang Akun:** `GET /renew/user?user=nama&exp=hari&auth=KEY`
    *   Menjalankan: `myscript.sh renew <user> <exp>`
*   **Trial Akun:** `GET /trial/user?exp=menit&auth=KEY`
    *   Menjalankan: `myscript.sh trial <exp>`

**3. Fitur Keamanan & Integrasi Telegram (PENTING):**
*   Buatkan script bash terpisah atau fungsi untuk **Setup Telegram**. Script ini harus meminta input **Bot Token** dan **Chat ID** dari user, lalu menyimpannya di file konfigurasi (misalnya `/etc/myscript/telegram.conf`).
*   Buatkan fungsi bash untuk **Generate API Key**. Fungsi ini harus:
    1.  Membuat string acak sebagai Auth Key.
    2.  Menyimpannya ke file `/etc/myscript/api_auth.key`.
    3.  **Mengirim Auth Key tersebut ke Bot Telegram saya** menggunakan `curl` dan konfigurasi yang sudah disimpan tadi.

**4. Installer Otomatis (`install-api.sh`):**
Buatkan script instalasi bash yang melakukan hal berikut secara otomatis:
*   Menginstall **Node.js** dan **NPM** (jika belum ada).
*   Membuat folder proyek, file `package.json`, dan menginstall dependency (`express`).
*   Membuat file `api.js` dengan kode server Node.js sesuai spesifikasi di atas.
*   Membuat **Systemd Service** agar API berjalan otomatis di background saat startup.
*   Mengatur Firewall (iptables) untuk membuka port 5888.
*   Menjalankan wizard setup Telegram di akhir instalasi.

Tolong berikan kode lengkap untuk `api.js`, `install-api.sh`, dan fungsi bash untuk integrasi Telegram-nya.
