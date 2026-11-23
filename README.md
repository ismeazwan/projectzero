================================================================================
🏭 OEM AUTOMATION SUITE — PAKET DISTRIBUSI & DEPLOYMENT MANDIRI
Honda HPPM (GP System) & LG Singlex (HLI Green Power) Procurement Automation
================================================================================

Folder ini berisi seluruh file siap pakai untuk menjalankan sistem otomasi
pengadaan, tender, dan pembuatan quotation harga resmi multi-portal.

--------------------------------------------------------------------------------
📁 DAFTAR ISI FILE DALAM PAKET:
--------------------------------------------------------------------------------
1. OEM_Manager.exe
   -> Aplikasi GUI Desktop (Pusat Kontrol Utama). Digunakan untuk mengatur
      kredensial .env, menyalakan/mematikan Docker, melihat log realtime,
      serta ekspor/impor workflow n8n dengan 1-klik.

2. docker-compose.yml
   -> Konfigurasi multi-container Docker untuk:
      - oem_scraper_worker (FastAPI + Playwright Headless + Telegram Bots)
      - oem_n8n_automation (Mesin otomasi alur kerja workflow)

3. .env
   -> File konfigurasi akun, password portal Honda & LG, server email IMAP,
      token bot Telegram, dan OpenRouter AI API Key.

4. n8n_all_workflows.json
   -> File cadangan (backup template) seluruh alur workflow n8n untuk Honda HPPM
      dan LG Singlex. Siap diimpor kapan saja jika berpindah komputer.

5. portal-scraper-service/
   -> Modul inti scraper Python (Playwright, Bot Telegram Honda, Bot Telegram LG,
      dan generator dokumen resmi Word .docx & PDF).

6. n8n_data_local/
   -> Salinan database SQLite n8n (database.sqlite) dan konfigurasi offline.

7. start_all.bat & start_bridge_silent.vbs
   -> Skrip batch opsional untuk menyalakan semua layanan tanpa membuka GUI.

--------------------------------------------------------------------------------
🚀 PANDUAN CEPAT DEPLOYMENT DI KOMPUTER KLIEN / SERVER BARU:
--------------------------------------------------------------------------------
Syarat Minimal di Komputer Klien:
• Sistem Operasi: Windows 10 (64-bit), Windows 11, atau Windows Server.
• Aplikasi Wajib: Docker Desktop for Windows (pastikan sudah running).
• TIDAK PERLU instalasi Python, pip, Node.js, atau compiler apapun!

Langkah 1: Menyiapkan Folder
• Salin seluruh folder 'OEM_Automation_Package' ini ke drive komputer klien
  (contoh: C:\OEM_Automation).

Langkah 2: Menyesuaikan Kredensial Akun
• Dobel-klik file 'OEM_Manager.exe'.
• Pada Tab '⚙️ Konfigurasi .env & Akun':
  - Sesuaikan akun login Honda GP System dan LG Singlex.
  - Pilih Preset Email:
    * "Hosting Klien (@cbn.net.id)" jika menggunakan server mail CBN (mail.cbn.net.id:993).
    * "Gmail" jika menggunakan testing (wajib Google App Password 16 digit).
  - Masukkan Token Bot Telegram untuk Honda & LG Singlex.
  - Klik tombol "💾 Simpan Perubahan ke File .env".

Langkah 3: Menyalakan Sistem
• Pindah ke Tab '🐳 Kontrol Docker Compose'.
• Klik tombol "▶️ Jalankan Semua Container (Compose Up)".
• Tunggu 10-20 detik sampai status berubah menjadi hijau (🟢 RUNNING).

Langkah 4: Memastikan Workflow n8n Terbawa
• Jika di komputer baru alur kerja n8n belum muncul, cukup klik tombol:
  "📥 Impor / Pulihkan Workflow ke Container" di Tab Kontrol Docker.
• Seluruh alur workflow Honda & LG Singlex akan langsung aktif otomatis!

--------------------------------------------------------------------------------
🌐 AKSES WEB & PENGUJIAN:
--------------------------------------------------------------------------------
• n8n Dashboard       : http://localhost:5678
• Scraper API Swagger : http://localhost:8000/docs
• Bot Telegram Honda  : Ketik nomor RFQ (contoh: 26A-009603) untuk cek penawaran.
• Bot Telegram LG     : Klik menu "📦 PO Masuk LG SINGLEX" atau "🔄 Sinkronkan Cookie".

================================================================================
PT. APSINDO GANANCIA PERKASA • OEM AUTOMATION SUITE
================================================================================
