Berikut adalah prompt yang dapat Anda gunakan untuk meminta AI lain atau developer membantu Anda mengimplementasikan konfigurasi Nginx, Xray, dan HAProxy pada script Anda yang sedang dibangun, meniru arsitektur repository ini.

---

**Prompt untuk AI / Developer:**

"Halo, saya sedang mengembangkan script tunneling baru dan ingin mengadopsi arsitektur yang kuat dan terbukti untuk menangani traffic VLESS, VMess, Trojan, dan Shadowsocks (WebSocket & gRPC) serta SSH, semuanya berjalan di port 80 dan 443 secara bersamaan.

Saat ini script saya belum menggunakan HAProxy, namun saya ingin menambahkannya sebagai *load balancer* utama di depan, karena arsitektur referensi yang saya gunakan sangat bergantung padanya untuk memilah traffic.

Tolong bantu saya mengimplementasikan konfigurasi berikut secara lengkap dan terintegrasi:

**1. Instalasi & Konfigurasi HAProxy (Frontend Utama)**
   - **Tujuan:** HAProxy harus menjadi satu-satunya service yang mendengarkan di port publik 80 dan 443.
   - **Logic Routing:**
     - Traffic **HTTP (Port 80)**: Jika header `Upgrade: websocket` ada, teruskan ke backend Nginx WebSocket (127.0.0.1:1010). Jika tidak, teruskan ke backend SSH/Dropbear (127.0.0.1:58080).
     - Traffic **HTTPS (Port 443)**: Lakukan SSL termination di HAProxy.
       - Jika header `Upgrade: websocket` ada, dekripsi dan teruskan ke backend Nginx WebSocket (127.0.0.1:1010).
       - Jika ALPN adalah `h2` (HTTP/2), teruskan ke backend Nginx gRPC (127.0.0.1:1013).
       - Jika tidak keduanya (traffic SSH over SSL/TLS biasa), teruskan ke backend SSH/Dropbear (127.0.0.1:58080).

**2. Konfigurasi Nginx (Reverse Proxy)**
   - Nginx tidak boleh mendengarkan port publik 80/443 lagi, melainkan hanya port lokal yang menerima traffic dari HAProxy.
   - **Server Block 1 (WebSocket - Listen 1010):**
     - Menerima traffic dari HAProxy dengan protokol PROXY (untuk menjaga IP asli user).
     - Menangani path: `/vless`, `/vmess`, `/trojan-ws`, `/ss-ws`.
     - Meneruskan (`proxy_pass`) ke port lokal Xray Core:
       - `/vless` -> 10001
       - `/vmess` -> 10002
       - `/trojan-ws` -> 10003
       - `/ss-ws` -> 10004
       - Path root `/` atau fallback -> Port SSH WebSocket (misal 10015).
   - **Server Block 2 (gRPC - Listen 1013 HTTP/2):**
     - Menerima traffic gRPC dari HAProxy.
     - Menangani path: `/vless-grpc`, `/vmess-grpc`, `/trojan-grpc`, `/ss-grpc`.
     - Meneruskan (`grpc_pass`) ke port lokal Xray Core:
       - `/vless-grpc` -> 10005
       - `/vmess-grpc` -> 10006
       - `/trojan-grpc` -> 10007
       - `/ss-grpc` -> 10008

**3. Konfigurasi Xray Core (Backend)**
   - Pastikan konfigurasi `inbounds` di `config.json` Xray sesuai dengan port mapping di atas:
     - Port 10001-10004 untuk protokol WebSocket.
     - Port 10005-10008 untuk protokol gRPC.
   - Semua inbound harus mendengarkan di `127.0.0.1` saja untuk keamanan.

Mohon buatkan konfigurasi lengkap untuk `haproxy.cfg`, `nginx.conf` (atau file conf.d terpisah), dan snippet `inbounds` untuk `config.json` Xray yang saling kompatibel."
