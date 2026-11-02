# 🏫 DapodikSQ — Kemendikdasmen Data Bridge & REST API Engine

[![PHP Version](https://img.shields.io/badge/PHP-8.3-777BB4?style=flat-square&logo=php&logoColor=white)](https://php.net)
[![Database](https://img.shields.io/badge/MySQL-InnoDB-4479A1?style=flat-square&logo=mysql&logoColor=white)](https://mysql.com)
[![Frontend](https://img.shields.io/badge/Bootstrap-5.3-7952B3?style=flat-square&logo=bootstrap&logoColor=white)](https://getbootstrap.com)
[![API Status](https://img.shields.io/badge/REST_API-Active-success?style=flat-square)](#-dokumentasi-rest-api-apiphp)
[![Ecosystem](https://img.shields.io/badge/Ecosystem-SmarQuest_App-blueviolet?style=flat-square)](#)
[![License](https://img.shields.io/badge/License-Open--Source-orange?style=flat-square)](#-etika-dan-tanggung-jawab-penggunaan)

**DapodikSQ** adalah *data ingestion engine* dan penyedia layanan *RESTful API* open-source untuk data sekolah nasional (Kemendikdasmen RI). Proyek ini dibangun sebagai subsistem dari ekosistem **SmarQuest App** guna mempermudah aplikasi frontend (Web, Mobile, Flutter, React, Vue) dalam mengonsumsi data sekolah Indonesia tanpa terbebani kerumitan *payload formatting*, *firewall timeout*, maupun parsing berulang.

---

## 🌟 Fitur Utama

- 🔄 **Multi-Mode Ingestion (Anti-Timeout / Anti-Block)**:
  - **Mode Server cURL**: Mengirimkan HTTP POST berformat JSON persis dengan *signature* browser & dukungan Cookie WAF.
  - **Mode JSON Bridge**: Penyelamat saat server hosting (seperti shared/free hosting) memblokir koneksi keluar (*outbound port 443*). Data respons dapat di-paste langsung untuk disimpan ke MySQL.
- ⚡ **Auto-Table Migration**: Otomatis membuat tabel `sekolah_dapodik` lengkap dengan indeks dan struktur kolom saat pertama kali dijalankan.
- 🛡️ **Pencegahan Duplikasi (Upsert Mechanism)**: Menggunakan `ON DUPLICATE KEY UPDATE` berbasis NPSN unik agar sinkronisasi berulang tidak menghasilkan data ganda.
- 📊 **Dashboard Interaktif (`tampil_data.php`)**:
  - Filter dinamis berdasarkan Jenjang (SD, SMP, SMA, SMK, dll.) dan Status (Negeri/Swasta).
  - Pencarian multi-kolom aman (*Prepared Statements*) meliputi Nama, NPSN, Alamat, dan Kota.
  - Pagination dinamis dan Modal Detail untuk inspeksi JSON mentah.
- 🚀 **REST API Siap Pakai (`api.php`)**:
  - Mendukung **CORS** (`Access-Control-Allow-Origin: *`).
  - Filter lengkap (Jenjang, Status, Kabupaten, Provinsi, NPSN, Kata Kunci).
  - Sistem pengaman beban server (*Safety Max Limit*) untuk mencegah lag atau kehabisan memori.
  - Fitur simulasi jeda waktu (`delay`) untuk pengujian UI *skeleton/loading state*.

---

## 📁 Struktur Berkas

```text
dapodiksq/
├── db.php           # Konfigurasi PDO MySQL & pembuatan skema tabel otomatis
├── ambil_data.php   # Antarmuka penarikan data dari API Kemendikdasmen (POST cURL & JSON Bridge)
├── tampil_data.php  # Dashboard visual web untuk pencarian, filter, dan monitoring data
├── api.php          # REST API Endpoint mandiri (Output JSON) untuk SmarQuest App
└── README.md        # Dokumentasi proyek
```

---

## ⚙️ Prasyarat Sistem

- **PHP**: Versi **8.3** atau yang lebih baru (ekstensi: `pdo_mysql`, `curl`, `json`, `mbstring`).
- **Database**: MySQL 5.7+ / MariaDB 10.3+ (Engine: `InnoDB`, Collation: `utf8mb4_unicode_ci`).
- **Web Server**: Apache / Nginx / LiteSpeed.

---

## 🚀 Panduan Instalasi

### 1. Salin Berkas
Unggah seluruh berkas ke direktori web server Anda (misalnya `public_html/` atau `htdocs/`).

### 2. Konfigurasi Basis Data
Buka berkas **`db.php`** dan sesuaikan kredensial database Anda:

```php
define('DB_HOST', 'localhost'); // atau host MySQL cluster Anda
define('DB_NAME', 'nama_database');
define('DB_USER', 'username_db');
define('DB_PASS', 'password_db');
```

> **Catatan:** Anda tidak perlu mengimpor file `.sql` secara manual. Saat Anda pertama kali mengakses `ambil_data.php` atau `tampil_data.php`, sistem akan membuatkan tabel `sekolah_dapodik` secara otomatis.

---

## 📖 Dokumentasi REST API (`api.php`)

Endpoint ini menghasilkan output JSON standar yang siap digunakan oleh modul apa pun pada **SmarQuest App**.

- **URL Dasar**: `https://hostgmkns.byethost32.com/dapodiksq/api.php`
- **Metode**: `GET`
- **Format Respons**: `application/json`

### Parameter Query URL

| Parameter | Tipe | Contoh Nilai | Deskripsi |
| :--- | :--- | :--- | :--- |
| `q` / `keyword` | `string` | `Negeri 1` | Mencari kata kunci pada Nama, NPSN, atau Alamat. |
| `bentuk` / `jenjang` | `string` | `SMK`, `SMA`, `SMP`, `SD` | Memfilter berdasarkan jenjang pendidikan. |
| `status` | `string` | `Negeri`, `Swasta` | Memfilter berdasarkan status kepemilikan sekolah. |
| `kab` / `kabupaten` | `string` | `Bandung` | Memfilter nama kabupaten atau kota. |
| `prov` / `provinsi` | `string` | `Jawa Barat` | Memfilter nama provinsi. |
| `npsn` | `string` | `20219141` | Mengambil data sekolah secara spesifik berdasarkan NPSN. |
| `page` | `integer` | `1` (Default: 1) | Nomor halaman data. |
| `limit` | `integer` | `25` (Maks: 200, atau `all` untuk maks 500) | Jumlah data per respons. |
| `delay` | `integer` | `500` (ms) atau `2` (detik) | Menambahkan jeda waktu buatan (maks. 3000 ms) untuk tes UI. |
| `raw` | `boolean` | `true` / `1` | Menyertakan data mentah asli dari Kemendikdasmen. |

### Contoh Pemanggilan

#### 1. Mengambil Daftar SMK Negeri di Kabupaten Bandung (Page 1)
```http
GET /api.php?bentuk=SMK&status=Negeri&kab=Bandung&limit=15&page=1
```

#### 2. Contoh Respons JSON:
```json
{
  "status": "success",
  "code": 200,
  "message": "Data sekolah berhasil diambil.",
  "pagination": {
    "page": 1,
    "limit": 15,
    "total_rows": 1,
    "total_pages": 1,
    "has_next": false,
    "has_prev": false
  },
  "filters_applied": {
    "keyword": null,
    "npsn": null,
    "bentuk": "SMK",
    "status": "Negeri",
    "kabupaten": "Bandung",
    "provinsi": null,
    "kecamatan": null,
    "delay_ms": 0
  },
  "data": [
    {
      "id": 42,
      "sekolah_id": "9B0A3D2F-XXXX-XXXX-XXXX",
      "npsn": "20219141",
      "nama_sekolah": "SMKN 1 KATAPANG",
      "bentuk_pendidikan": "SMK",
      "status_sekolah": "Negeri",
      "alamat": "Jl. Ceuri BK No. 1, Katapang",
      "kecamatan": "Kec. Katapang",
      "kabupaten_kota": "Kab. Bandung",
      "provinsi": "Prov. Jawa Barat",
      "updated_at": "2026-03-01 10:20:00"
    }
  ]
}
```

---

## 🛠️ Panduan Penggunaan Panel Sinkronisasi (`ambil_data.php`)

1. Buka halaman `ambil_data.php` melalui browser Anda.
2. Terdapat dua tab metode:
   - **Metode 1 (POST cURL Server)**:
     - Masukkan kata kunci (misal: `SMK negeri`), ukuran data (`size`: 50 atau 100).
     - Kolom *Cookie WAF* bersifat opsional jika server Kemendikdasmen meminta verifikasi cookie sesi (`TSc78de468027`).
     - Klik **Kirim Permintaan POST cURL & Simpan ke MySQL**.
   - **Metode 2 (Paste JSON Langsung)**:
     - Gunakan metode ini bila hosting Anda mengalami *timeout* akibat firewall outbound.
     - Ambil JSON menggunakan cURL lokal, DevTools, atau Postman.
     - Tempel teks respons JSON ke textarea yang tersedia, lalu klik **Simpan JSON Tersebut ke Database**.

---

## ⚖️ Etika dan Tanggung Jawab Penggunaan

Aplikasi ini dipublikasikan sebagai perangkat lunak sumber terbuka (*open-source software*) di bawah naungan ekosistem **SmarQuest App**. 

Setiap pengembang, peneliti, maupun institusi bebas menggunakan, memodifikasi, dan mendistribusikan kode ini dengan ketentuan etis:

1. **Penuh Kesadaran & Tanggung Jawab**: Jangan melakukan *bombardir request* (DDoS/spamming) ke server Kemendikdasmen. Berikan jeda (*rate-limiting*) yang wajar antar-penarikan data.
2. **Kemanfaatan Pendidikan**: Data yang diperoleh ditujukan untuk kemudahan integrasi sistem, analisis data pendidikan, transparansi informasi publik, atau pengembangan aplikasi penunjang sekolah.
3. **Privasi & Keamanan**: Pastikan informasi kredensial basis data Anda disimpan secara aman dan tidak diunggah ke repositori publik (*public repository*) tanpa penyensoran.

---

## 🤝 Kontribusi & Ekosistem

Bagian dari ekosistem **SmarQuest App**. Kontribusi perbaikan bug, penambahan fitur filter, dan optimalisasi query selalu terbuka melalui mekanisme *Pull Request* atau pembuatan *Issue*.

*Dibuat untuk memajukan integrasi ekosistem pendidikan yang lebih adaptif, efisien, dan terbuka.*
