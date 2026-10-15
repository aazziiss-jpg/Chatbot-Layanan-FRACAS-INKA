# Chatbot Layanan Informasi FRACAS - PT. INKA

Repositori ini berisi workflow n8n dan panduan konfigurasi untuk **Chatbot Layanan Informasi FRACAS PT. INKA**. Chatbot ini dibangun menggunakan n8n, WAHA (WhatsApp HTTP API), Google Gemini AI, dan Supabase untuk memberikan informasi, mengelola laporan kerusakan, serta menyajikan dokumen internal perusahaan secara otomatis melalui WhatsApp.

## Struktur Repositori

```text
├── Documentation/
│   ├── 1. Register.jpg
│   ├── 2. Status Validasi (waiting).png
│   ├── 3. Status Validasi (not approve).png
│   ├── 4. Information Mode Route.png
│   ├── 5. Req Data Mode v1 prompt.jpg
│   ├── 6. Req Data Mode v2 prompt.jpg
│   ├── 7. Req Data Mode v3 prompt.jpg
│   ├── 8. Not recommended prompting.png
│   └── 9. Document Mode Route.png
├── Project-Files/
│   └── Chatbot-Layanan-Informasi-FRACAS-n8n-PT-INKA.json
└── README.md
```

## Prasyarat & Setup Lingkungan

Untuk menjalankan sistem chatbot ini, Anda membutuhkan alat hosting (disarankan menggunakan **Docker**) dan beberapa layanan pihak ketiga.

### 1. Deployment dengan Docker (n8n & WAHA)
Sistem ini dirancang untuk di-deploy menggunakan Docker container secara lokal atau pada Virtual Machine (VM). Anda perlu menjalankan dua container utama:
*   **n8n**: Sebagai mesin orkestrasi (workflow automation).
*   **WAHA (WhatsApp HTTP API)**: Sebagai jembatan komunikasi antara n8n dan WhatsApp.

Silakan pelajari dokumentasi resmi instalasi via Docker untuk [n8n](https://docs.n8n.io/hosting/installation/docker/) dan [WAHA](https://waha.devlike.pro/docs/how-to/install/).

### 2. Konfigurasi WAHA & WhatsApp
Untuk menghubungkan nomor WhatsApp yang akan digunakan sebagai Chatbot:
1. Buka dashboard WAHA lokal Anda (biasanya di `http://localhost:3000` atau sesuai port yang Anda atur).
2. Lakukan proses *Scan QR Code* untuk menghubungkan nomor WhatsApp.
3. Dapatkan **API Key WAHA** melalui dashboard konfigurasi WAHA (pelajari lebih lanjut di [waha.devlike.pro](https://waha.devlike.pro/)).
4. Konfigurasikan **Webhook** di dalam n8n:
   * Buka node **WAHA Trigger** di n8n.
   * Masukkan URL Webhook n8n Anda ke dalam konfigurasi WAHA agar setiap pesan masuk diteruskan ke n8n.
5. Masukkan WAHA API Key Anda pada kredensial n8n.

### 3. Konfigurasi Gemini API
Chatbot ini menggunakan kecerdasan buatan dari Google Gemini untuk memproses dan mengklasifikasikan pesan.
1. Kunjungi [Google AI Studio](https://aistudio.google.com/app/api-keys) atau [Google Cloud Console](https://cloud.google.com).
2. Buat project baru (jika belum ada) dan *Generate API Key* untuk Gemini.
3. Masukkan API Key tersebut ke dalam Node **Google Gemini** di workflow n8n Anda (pada bagian *Credentials*).

### 4. Konfigurasi Database Supabase (Vector Store & Chat Memory)
Supabase digunakan untuk menyimpan *Chat History* (memori obrolan) dan *Document History/Knowledge Base* menggunakan vector database. Anda perlu menyiapkan kredensial berikut di dashboard Supabase Anda:
*   **Host / Database URL**
*   **Database Name**
*   **Table Name** (Buat tabel khusus untuk *chat memory* dan dokumen vektor)
*   **User & Password**
*   **API Key (Anon/Service Role)**

Masukkan data koneksi (Postgres) ke dalam node **Postgres Chat Memory** dan **Supabase Vector Store** di n8n.

### 5. Persiapan Google Sheets
Sistem ini membaca data tabular untuk validasi dan informasi dokumen. Anda harus membuat Google Spreadsheet dengan format yang sesuai dengan query di workflow:
1.  **Data Karyawan:** Berisi daftar nomor telepon, nama, divisi, dan status approval untuk memvalidasi apakah pengguna diizinkan menggunakan chatbot.
2.  **Data Dokumen PM:** Berisi metadata dokumen (Nama Dokumen, Kode, Penanggung Jawab, Pendukung, dan Link URL PDF dari Google Drive).

*Catatan: Pastikan Anda menghubungkan kredensial Google Drive dan Google Sheets API Anda di n8n.*

---

## Cara Import Workflow

1. Buka dashboard n8n Anda.
2. Buat Workflow baru.
3. Klik menu titik tiga di pojok kanan atas, pilih **Import from File**.
4. Pilih file `Project-Files/Chatbot-Layanan-Informasi-FRACAS-n8n-PT-INKA.json` dari repositori ini.
5. Sesuaikan semua kredensial (WAHA, Gemini, Google Sheets, Google Drive, Supabase) yang memiliki tanda peringatan/error.
6. Aktifkan (Activate) workflow.

---

## Dokumentasi & Panduan Penggunaan (Fitur Chatbot)

Berikut adalah panduan penggunaan chatbot berdasarkan alur yang telah dikonfigurasi, beserta tangkapan layarnya.

### 1. Tahap Registrasi
Jika pengguna baru mengirim pesan dan nomornya belum terdaftar di database (Google Sheets), sistem akan otomatis mengirimkan form pendaftaran.
<br>
![Register](Documentation/1.%20Register.png)

### 2. Status Validasi (Waiting)
Setelah mengisi form, status pengguna di database menjadi *WAITING*. Jika pengguna mencoba mengakses chatbot, sistem akan memberitahu bahwa data sedang dalam tahap validasi oleh admin.
<br>
![Status Waiting](Documentation/2.%20Status%20Validasi%20(waiting).png)

### 3. Status Validasi (Not Approve)
Jika pendaftaran ditolak atau admin menonaktifkan akses pengguna, status akan berubah menjadi *NOT APPROVE*. Sistem akan memblokir akses dan mengarahkan pengguna untuk menghubungi pihak MDO.
<br>
![Status Not Approve](Documentation/3.%20Status%20Validasi%20(not%20approve).png)

### 4. Information Mode Route
Ini adalah mode asisten percakapan umum. Chatbot dapat merespons sapaan, menjawab identitasnya, atau menjelaskan definisi dan prosedur umum terkait sistem FRACAS. AI dikonfigurasi agar tidak menampilkan data mentah pada mode ini.
<br>
![Information Mode](Documentation/4.%20Information%20Mode%20Route.png)

### 5. Req Data Mode (Permintaan Data Laporan)
Sistem ini menggunakan Google Visualization API (tq) yang di-generate oleh AI untuk melakukan *query* langsung ke database laporan. Pengguna dapat memanggil data dengan berbagai variasi *prompt*:

**Variasi 1: Prompt Terstruktur**
Menyebutkan Nomor KA dan Tanggal secara spesifik.
<br>
![Req Data v1](Documentation/5.%20Req%20Data%20Mode%20v1%20prompt.png)

**Variasi 2: Prompt Kontekstual/Santai**
Menggunakan bahasa sehari-hari seperti *"Saya mau kamu berikan data di bulan Juni tanggal 1 2026, tapi KA K125104 ya.."*. AI cukup cerdas untuk mengekstrak parameter tersebut.
<br>
![Req Data v2](Documentation/6.%20Req%20Data%20Mode%20v2%20prompt.png)

**Variasi 3: Prompt Singkat (Keyword)**
Cukup mengetikkan kata kunci pencarian yang dipisahkan koma, seperti *"Data K1125104, 30 Juli 2026, OPEN, Aris"*.
<br>
![Req Data v3](Documentation/7.%20Req%20Data%20Mode%20v3%20prompt.png)

### 6. Prompt yang Tidak Direkomendasikan
Hindari memberikan *prompt* yang meminta AI untuk menghitung seluruh data, melakukan estimasi kompleks yang tidak didukung data, atau meminta rincian dari pencarian yang terlalu luas (misal hanya menyebutkan "Bogie"). Hal ini dapat menyebabkan beban *runtime* yang sangat lama (Time Out) atau server sibuk karena ukuran data balasan yang terlalu besar.
<br>
![Not Recommended](Documentation/8.%20Not%20recommended%20prompting.png)

### 7. Document Mode Route
Jika pengguna menanyakan isi, prosedur, pendukung, atau link fisik dokumen, sistem akan mencari informasi tersebut melalui *Knowledge Base* (Supabase Vector) dan memadukannya dengan metadata link dari Google Sheets.
<br>
![Document Mode](Documentation/9.%20Document%20Mode%20Route.png)
