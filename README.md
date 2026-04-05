# temp-mail-multi
Serverless Temporary Email VIP System on Cloudflare

# Rx-TempMail Engine: Serverless Multi-Domain Inbox (Open API)

Rx-TempMail ini solusi Temporary Email serverless pakai ekosistem Cloudflare. Versi ini dibikin pakai UI ala (Cyber-Minimalist) dan mode Open API biar gampang diintegrasiin buat SaaS tanpa ribet mikirin login.

Sistem ini support Multi-Domain dari satu worker aja, dan ada dashboard CCTV admin tersembunyi buat mantau trafik email.

## Fitur Utama
* Full Serverless: Pake Cloudflare Workers, D1 (SQLite), dan R2 (Object Storage).
* Multi-Domain Ready: Bisa nambahin domain berapapun, tinggal colokin Catch-all ke satu Worker ini.
* Smart Polling: UI-nya otomatis berhenti ngecek email kalo tab browser disembunyiin. Ini ngirit kuota Worker banget.
* Support Attachment: Bisa nangkep dan nyimpen lampiran file langsung ke Cloudflare R2.
* Open API: Siap dipake publik tanpa perlu sistem password buat baca email.
* Omniscient Dashboard: Panel rahasia buat mantau semua email masuk dan keluar dari semua user.

---

## Syarat Awal (Prerequisites)
Pastiin lu atau user lu udah punya infrastruktur dasar ini di Cloudflare:
1. Cloudflare Worker yang udah jadi.
2. Cloudflare D1 Database yang udah terbuat (pastiin udah ada tabel `emails` dan `outbox`).
3. Cloudflare R2 Bucket (opsional, kalo mau support file lampiran).
4. Minimal satu domain aktif di Cloudflare.

---

## Cara Setup

### Langkah 1: Update Kode Worker (Backend)
1. Buka dashboard Cloudflare, terus masuk ke menu Workers & Pages.
2. Klik Worker TempMail lu yang udah ada.
3. Klik tombol Edit Code.
4. Hapus semua kode yang lama, terus copas kode Worker (Level 4 - Open API) dari repo ini ke situ.
5. Klik Deploy di pojok kanan atas.

### Langkah 2: Setting Bindings & Variables
Biar kodenya jalan, pastiin Worker-nya udah nyambung ke Database dan Storage.
1. Di halaman Worker, masuk ke tab Settings > Bindings.
2. Pastiin Binding D1 udah ada:
   * Variable name: `DB`
   * Arahin ke database D1 lu.
3. Pastiin Binding R2 udah ada (kalo pake lampiran file):
   * Variable name: `BUCKET`
   * Arahin ke R2 Bucket lu.
4. Masuk ke tab Variables and Secrets di panel yang sama.
5. Tambahin variabel rahasia ini:
   * Variable name: `ADMIN_KEY` | Value: `PasswordRahasiaLu` (Ini dipake buat ngakses Omniscient Dashboard nanti).

### Langkah 3: Colokin Domain ke Worker (Email Routing)
Biar email yang masuk ke domain lu otomatis masuk ke Worker:
1. Balik ke dashboard utama Cloudflare, klik domain lu.
2. Masuk ke menu Email > Email Routing. Pastiin fiturnya udah aktif.
3. Pindah ke tab Routing rules.
4. Cari bagian Catch-all address, terus klik Edit.
5. Ubah settingannya jadi:
   * Action: Send to a Worker
   * Destination: [Nama-Worker-Lu]
6. Klik Save. 
*(Ulangin langkah ini buat domain-domain lain kalo lu mau main multi-domain).*

### Langkah 4: Deploy Frontend (Hacker UI)
1. Download file `index.html` dari repo ini.
2. Host file HTML ini di mana aja bebas (bisa GitHub Pages, Cloudflare Pages, atau server biasa).
3. Buka webnya. Pas pertama kali dibuka, bakal muncul layar setup SYS.INIT.
4. Masukin config-nya:
   * Endpoint.URL: Isi pake link Worker lu (contoh: https://worker-lu.username.workers.dev)
   * Valid.Domains: Daftar domain lu, pisahin pake koma (contoh: domain-a.com, domain-b.net)
5. Klik EXECUTE. Udah beres, web TempMail lu udah online dan siap dipake publik!

---

## Cara Akses Omniscient Dashboard (Admin)
Buat mantau semua trafik email dari semua user, lu tinggal buka browser terus masukin link ini:
`https://[LINK_WORKER_LU]/api/admin?key=[ADMIN_KEY_LU]`

Contoh: `https://worker-lu.username.workers.dev/api/admin?key=PasswordRahasiaLu`

---

## Catatan Keamanan & Pembersihan Otomatis
* Auto-Delete: Sistem ini udah dipasang script sapu jagat. Email dan file lampiran yang umurnya lebih dari 24 jam bakal otomatis kehapus dari database tiap kali ada email baru yang masuk.
* Privacy: Karena ini jalan di mode Open API, pastiin user lu bikin nama alias email yang lumayan unik biar ga gampang diintip sama user lain yang nebak-nebak nama.
