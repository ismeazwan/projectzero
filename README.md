# projectzero
automation vendor


# PANDUAN DEPLOYMENT & MIGRASI SISTEM OTOMASI TENDER HONDA HPPM
**Dokumen Panduan Teknis & Serah Terima Client**  
*Versi: 2.0 (Fase 1: Deteksi RFQ & Notifikasi + Fase 2: Otomasi Quotation via Telegram & AI)*  
*Tanggal: 11 November 2026*  

---

## 1. RINGKASAN SISTEM & ARSITEKTUR

Sistem ini adalah solusi otomasi menyeluruh (*End-to-End Automation*) untuk siklus tender pengadaan suku cadang **PT Honda Precision Parts Manufacturing (HPPM)** melalui portal resmi **Honda Global Partner System (`https://gpsystem.id`)**.

```
┌────────────────────────────────────────────────────────────────────────┐
│               FASE 1: MONITORING & NOTIFIKASI TENDER                   │
├──────────────────┐     ┌─────────────────────┐     ┌───────────────────┤
│ Email Tender     │ ──> │ n8n Workflow Engine │ ──> │ Dokumen PDF Asli  │
│ (Gmail IMAP 993) │     │ (Schedule 1 Menit)  │     │ + Caption Lengkap │
└──────────────────┘     └──────────┬──────────┘     │ dikirim Telegram  │
                                    │                └───────────────────┘
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│             BACKEND WORKER & CACHE (Docker Python FastAPI)             │
│   - SQLite Cache: auth_tokens, processed_emails, rfq_cache             │
│   - JWT Session Caching (365 Hari, Zero Redundant Login)               │
│   - REST API Integrator ke https://gpsystem.id                         │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│             FASE 2: PEMBUATAN & REVISI QUOTATION (INTERAKTIF)          │
├──────────────────┐     ┌─────────────────────┐     ┌───────────────────┤
│ User Chat Bot    │ <─> │ Ekstraksi AI & Teks │ ──> │ Submit Final      │
│ (@exportir2_bot) │     │ (Nemotron/Vision)   │     │ Status "COMPLETE" │
│ Menu UI/UX Nav   │     │ + Lampiran PDF Fisik│     │ ke Portal Honda   │
└──────────────────┘     └─────────────────────┘     └───────────────────┘
```

---

## 2. PRASYARAT LINGKUNGAN (PREREQUISITES)

### A. Spesifikasi Server / Virtual Machine (Client)
* **Sistem Operasi**: Linux Ubuntu 22.04 / 24.04 LTS (Direkomendasikan) atau Windows Server 2022.
* **CPU / RAM**: Minimal 2 vCPU / 4 GB RAM.
* **Storage**: Minimal 20 GB SSD.
* **Software Wajib**:
  * Docker Engine (v24.0+) & Docker Compose (v2.20+).
  * Port Lokal Terbuka: `5678` (n8n Web UI) dan `8000` (Python Backend Worker).

### B. Akses Jaringan (Firewall / Outbound Rule)
Server client harus memiliki akses internet keluar (*outbound*) ke:
* `imap.gmail.com:993` (Protokol IMAP SSL untuk email).
* `api.telegram.org:443` (API Telegram Bot).
* `gpsystem.id:443` (Portal Honda GP System).
* `openrouter.ai:443` (API AI Nemotron / Vision untuk ekstraksi PDF).

---

## 3. STRUKTUR DIREKTORI PROYEK

Folder proyek yang wajib disalin ke server client:

```text
/opt/honda-oem-automation/
├── .env                                       # Kredensial & Variabel Lingkungan
├── docker-compose.yml                         # Konfigurasi Container Docker
├── n8n_workflow_honda_rfq_lifecycle.json      # Template Workflow n8n Fase 1
├── portal-scraper-service/                    # Layanan Microservice Python Worker
│   ├── Dockerfile                             # Build Image Python Container
│   ├── requirements.txt                       # Dependensi Python
│   ├── main.py                                # API Backend & Sinkronisasi
│   ├── telegram_phase2_bot.py                 # Bot Telegram Interaktif Fase 2
│   └── rfq_quotation_cache.db                 # Database SQLite Lokal (Auto-create)
└── README_STRUKTUR_FILE.md                    # Dokumentasi Struktur File
```

---

## 4. KONFIGURASI VARIABEL LINGKUNGAN (`.env`)

Buat atau edit file `.env` di root folder proyek:

```ini
# ==========================================================
# 1. KREDENSIAL PORTAL HONDA GP SYSTEM (https://gpsystem.id)
# ==========================================================
HONDA_PORTAL_USER=apsindo@cbn.net.id
HONDA_PORTAL_PASS=nuclearportalhppm
HONDA_PORTAL_BASE=https://gpsystem.id

# ==========================================================
# 2. KREDENSIAL TELEGRAM BOT (FASE 1 & FASE 2)
# ==========================================================
TELEGRAM_BOT_TOKEN=9195418527:AAE86SeYoCnNHcCLD916RemhovjFMZGksYY
TELEGRAM_CHAT_ID=1859219519

# ==========================================================
# 3. KREDENSIAL EMAIL MONITORING (GMAIL IMAP)
# ==========================================================
EMAIL_USERNAME=jaganetworks25@gmail.com
EMAIL_APP_PASSWORD=dvluwqhrlcnttucd

# ==========================================================
# 4. KREDENSIAL AI ENGINE (OPENROUTER / GEMINI)
# ==========================================================
OPENROUTER_API_KEY=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

# ==========================================================
# 5. KEAMANAN INTERNAL SERVICE
# ==========================================================
SCRAPER_SECRET_KEY=manufacturing-secure-token-2026
N8N_PORT=5678
GENERIC_TIMEZONE=Asia/Jakarta
```

> [!IMPORTANT]
> Password Gmail harus menggunakan **Google App Password** (16 karakter), bukan password login akun biasa.

---

## 5. PANDUAN INSTALASI & DEPLOYMENT STEP-BY-STEP

### Langkah 1: Salin File Proyek ke Server Client
```bash
# Buat folder proyek di server
mkdir -p /opt/honda-oem-automation
cd /opt/honda-oem-automation

# Salin seluruh file proyek ke folder ini
```

### Langkah 2: Build & Jalankan Container Docker
Jalankan perintah berikut untuk mengompilasi dan mengaktifkan service:
```bash
docker compose up -d --build
```

Pastikan kedua container berjalan normal:
```bash
docker ps
```
*Output yang diharapkan:*
* `oem_scraper_worker` (Port `8000:8000`, Status `Up`)
* `oem_n8n_automation` (Port `5678:5678`, Status `Up`)

---

### Langkah 3: Import & Aktivasi Workflow n8n (Fase 1)

1. Buka browser dan akses n8n di: `http://<IP_SERVER_CLIENT>:5678`
2. Buat akun admin pertama jika instalasi baru.
3. Di panel kiri, klik **Workflows** $\rightarrow$ **Import from File**.
4. Pilih file: [`n8n_workflow_honda_rfq_lifecycle.json`](file:///c:/Users/muham/Documents/co129/n8n_workflow_honda_rfq_lifecycle.json).
5. Pada sudut kanan atas workflow, ubah toggle menjadi **Active** (warna hijau).
6. Workflow Fase 1 sekarang akan memeriksa email tender secara otomatis setiap 1 menit.

---

### Langkah 4: Verifikasi Layanan Fase 2 (Telegram Bot UI/UX)

1. Periksa log container worker untuk memastikan bot Telegram aktif:
   ```bash
   docker logs oem_scraper_worker --tail 20
   ```
   *Output yang diharapkan:*
   ```text
   Background thread Telegram Bot Phase 2 dengan Rich UI/UX berhasil dijalankan.
   Memulai Telegram Phase 2 Polling Listener dengan Rich UI/UX...
   ```
2. Buka aplikasi Telegram, cari bot Anda (contoh: `@exportir2_bot`), dan ketik `/start`.
3. Menu keyboard di bawah layar akan langsung muncul:
   * `[ 📝 Buat / Edit Quotation ]`
   * `[ 📋 Daftar RFQ Terbaru ]`
   * `[ 📊 Cek Status Portal ]`
   * `[ 💡 Panduan Penggunaan ]`

---

## 6. SOP PENGUJIAN & VALIDASI SISTEM (CHECKLIST)

### Pengujian Fase 1 (Email $\rightarrow$ Telegram)
1. Kirim email uji coba ke `jaganetworks25@gmail.com` dengan subjek:  
   `RECEIPT OF DOCUMENTS PO No 26A-009594`
2. Tunggu maksimal 1 menit.
3. **Hasil yang Diharapkan di Telegram**:
   * Masuk pesan dokumen PDF resmi dari Honda (nama file: `RFQ_26A-009594.pdf`).
   * Terdapat caption ringkasan: Nomor RFQ, Buyer, Department, Batas Waktu, dan Daftar Barang.
   * Email di Gmail otomatis ditandai sebagai Terbaca (*Seen*).
   * Email tidak akan dikirim ulang dua kali jika scheduler berjalan kembali.

### Pengujian Fase 2 (Pengisian Quotation)
1. Di Telegram, klik tombol **`[ 📋 Daftar RFQ Terbaru ]`**.
2. Bot akan menampilkan 5 tender RFQ terkini yang terurut berdasarkan tanggal terbaru (November 2026).
3. Klik tombol salah satu RFQ (misal `👉 Pilih 26A-009594`).
4. Bot akan mendeteksi status RFQ:
   * **Jika Belum Ada Quotation**: Menampilkan status baru dan meminta pengisian.
   * **Jika Sudah Pernah Ada**: Menampilkan nomor quotation dan total harga yang tersimpan di portal Honda.
5. Pilih metode:
   * **Jalur Teks**: Salin template, ubah harga, dan kirim balik.
   * **Jalur File PDF**: Upload file penawaran resmi PDF, AI akan membaca seluruh field.
6. Periksa kartu konfirmasi pratinjau harga.
7. Klik tombol **`[ 🚀 Ya, Kirim ke Honda ]`**.
8. Bot akan mengirimkan pesan sukses bahwa data telah di-assign dan quotation berstatus **`COMPLETE`** di portal Honda.

---

## 7. PEMELIHARAAN (MAINTENANCE) & TROUBLESHOOTING

### 1. Masalah Token Sesi Honda
* **Gejala**: API Honda merespons HTTP 401 Unauthorized.
* **Penyebab**: Seseorang melakukan klik "Logout" manual pada akun di browser web.
* **Solusi Otomatis**: Sistem microservice Python dirancang mendeteksi status 401 dan secara otomatis melakukan login ulang menggunakan kredensial di `.env`, lalu memperbarui token baru ke database SQLite.

### 2. Sertifikat SSL Honda (`gpsystem.id`)
* Portal Honda menggunakan intermediate certificate yang tidak lengkap pada beberapa sistem operasi.
* **Penanganan**: Semua request ke `gpsystem.id` sudah dikonfigurasi dengan flag `verify=False` pada Python dan `allowUnauthorizedCerts: true` pada n8n.

### 3. Backup & Reset Database SQLite
Database lokal tersimpan di `portal-scraper-service/rfq_quotation_cache.db`.
* **Untuk Backup**:
  ```bash
  cp portal-scraper-service/rfq_quotation_cache.db /opt/backup_rfq_cache_$(date +%Y%m%d).db
  ```
* **Untuk Reset Email Cache (jika ingin memproses ulang email lama)**:
  ```bash
  docker exec -it oem_scraper_worker python -c "import sqlite3; conn=sqlite3.connect('rfq_quotation_cache.db'); conn.execute('DELETE FROM processed_emails'); conn.commit()"
  ```

### 4. Restarting Service
Jika ada perubahan konfigurasi `.env`, cukup jalankan:
```bash
docker compose restart
```

---

## 8. KONTAK DUKUNGAN TEKNIS
Jika terdapat kendala teknis lebih lanjut pada saat migrasi atau perubahan endpoint API Honda, hubungi tim pengembang otomatisasi sistem.

