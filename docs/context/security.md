# Standar Keamanan Sistem & Klasifikasi Data SewERP

Dokumen ini mendefinisikan standar keamanan informasi, klasifikasi perlindungan data (dari data pribadi/PII hingga berkas pola pakaian/desain berstatus rahasia ketat), tata kelola kriptografi kredensial perbankan/payment gateway, isolasi multi-tenancy, serta matriks kepatuhan keamanan untuk seluruh modul sistem **SewERP**.

---

## 1. Metadata & Tautan Referensi Terkait

- **Kode Dokumen**: `STD-SEC-01`
- **Domain Bounded Context**: Seluruh Domain (`crm`, `oms`, `mes`, `wms`, `payroll`, `kms`, `analytics`)
- **Tautan Standar Terkait**:
  - 🏛️ [Arsitektur Sistem & Multi-Tenancy](./architecture.md)
  - ⚙️ [Standar Rekayasa Backend Go](./backend.md)
  - 🖥️ [Standar Rekayasa Frontend SolidJS](./frontend.md)
  - 🗄️ [Standar Database PostgreSQL & Audit Trail](./dbadmin.md)
  - 🚀 [Standar DevOps & S3 Object Storage](./devops.md)
  - 🧪 [Standar QA & Pengujian Keamanan](./qa.md)
  - 📝 [Standar Formulir & Validasi Input](./forms.md)
- **Regulasi & Acuan Hukum**:
  - **UU No. 27 Tahun 2022** tentang Perlindungan Data Pribadi (UU PDP Republik Indonesia).
  - Standar Industri **OWASP Top 10** & **OWASP API Security Top 10**.
- **Rekomendasi Skills untuk Agent (Recommended Agent Skills)**:
  - `infosec`: Audit kerentanan keamanan, kepatuhan OWASP Top 10 & OWASP API Top 10, scanning secret/kredensial, verifikasi enkripsi AES-256-GCM, validasi S3 Presigned URL ($\le$ 15 menit), stempel dinamis watermark pola pakaian CAD/DXF, dan mitigasi ancaman (Threat Modeling).
  - `cookie-debugging`: Inspeksi dan pengujian atribut keamanan cookie peramban (`HttpOnly`, `Secure`, `SameSite=Lax`), token sesi, proteksi XSS/CSRF, dan alur redirect autentikasi.
  - `go-vulncheck`: Deteksi celah keamanan CVE pada dependensi Go pihak ketiga menggunakan tooling gopls/govulncheck.
  - `autonomous-investigation`: Investigasi mendalam terhadap ancaman keamanan rantai pasok (*supply chain dependencies*), kepatuhan privasi data UU PDP, dan review arsitektur zero trust.

---

## 2. Prinsip Dasar Keamanan SewERP

1. **Principle of Least Privilege (PoLP)**:
   Setiap pengguna, service, dan token hanya diberikan izin akses minimum yang mutlak dibutuhkan untuk menjalankan fungsinya. Operator meja jahit tidak boleh memiliki hak akses membaca omset keuangan atau data pelanggan.
2. **Defense-in-Depth (Pertahanan Berlapis)**:
   Keamanan multi-tenancy dan validasi otorisasi tidak hanya ditegakkan di layer aplikasi (Go Gin Handler & Usecase), tetapi juga diproteksi di layer database melalui PostgreSQL Row-Level Security (RLS) serta layer jaringan melalui Nginx reverse proxy.
3. **Zero Plaintext Secrets**:
   Dilarang keras menyimpan kunci API pihak ketiga, token webhook, password, atau credential sensitif dalam bentuk teks polos (*plaintext*) di database, log aplikasi, repositori git, maupun respons REST API.
4. **Zero Trust Boundary**:
   Setiap request yang masuk melalui HTTP, callback webhook, atau pesan antrean internal wajib diautentikasi dan divalidasi ulang integritasnya. Jangan pernah mempercayai data kiriman dari browser klien.

---

## 3. Tingkatan Klasifikasi Perlindungan Data (Data Classification Tier)

SewERP menetapkan 4 tingkatan klasifikasi data yang menjadi pedoman penanganan teknis di seluruh siklus hidup data:

```
┌────────────────────────────────────────────────────────────────────────┐
│ LEVEL 4: STRICT / SECRET (Kekayaan Intelektual & Kredensial Sistem)    │
│ • Berkas Pola CAD/DXF/CDR/AI/PDF (1:1) • Xendit Secret & Webhook Token │
├────────────────────────────────────────────────────────────────────────┤
│ LEVEL 3: CONFIDENTIAL & PII (Data Pribadi Pelanggan/Karyawan & Finansial│
│ • Nama, No WA, Alamat • Slip Gaji • Kasbon • Omset & Piutang Pelanggan │
├────────────────────────────────────────────────────────────────────────┤
│ LEVEL 2: INTERNAL OPERATIONAL (Operasional Lantai Pabrik & Inventaris)  │
│ • Stok Kain & Aksesoris • Status 6 Stasiun Kanban • Kuantiti Potong Pcs │
├────────────────────────────────────────────────────────────────────────┤
│ LEVEL 1: PUBLIC (Informasi Umum Terbuka)                               │
│ • Katalog Jenis Bahan Kain • Syarat & Ketentuan Umum • Branding Usaha  │
└────────────────────────────────────────────────────────────────────────┘
```

### Rincian Penanganan Teknis per Tingkatan

| Klasifikasi | Cakupan Data | Kontrol Penyimpanan & Enkripsi | Kontrol Akses & Pembagian | Kebijakan Log & Ekspor |
| :--- | :--- | :--- | :--- | :--- |
| **Level 1: Public** | Profil konveksi, panduan ukuran publik, katalog jenis bahan. | Penyimpanan standar PostgreSQL / CDN publik. | Terbuka tanpa autentikasi. | Boleh dicatat di log & cache publik. |
| **Level 2: Internal** | Stok fisik gudang (kg/yard), sebaran stasiun SPK, kuantiti potong. | Database PostgreSQL dengan filter `tenant_id`. | Wajib autentikasi JWT pengguna aktif tenant. | Boleh dicatat di log operasional. |
| **Level 3: Confidential & PII** | Nama pelanggan, nomor WhatsApp, alamat kirim, kontak pekerja, saldo kasbon, rincian slip upah, nominal transaksi kasir. | Enkripsi in-transit (TLS 1.3), enkripsi at-rest (LUKS/PostgreSQL pgcrypto untuk data super-sensitif). | RBAC ketat (hanya owner, finance, atau subjek data terkait). Masking di UI non-otoritatif. | **Wajib masking di log (Zerolog)**: `0812****890`. Ekspor CSV/Excel diaudit & diberi rate-limit. |
| **Level 4: Strict / Secret** | Master file pola baju CAD/DXF/CDR/AI, lembar spesifikasi jahitan (Tech Pack), mockup eksklusif klien, API Secret Key Xendit, token webhook. | Private S3 Bucket (100% no public read), Enkripsi AES-256-GCM di level aplikasi Go sebelum disimpan ke DB. | Presigned URL berdurasi pendek (maks. 15 menit). Dynamic watermarking pada tampilan visual. | **Dilarang keras dicatat di log**. Maskir penuh pada API response (`xnd_production_****`). |

---

## 4. Standar Perlindungan Aset Pola Pakaian & Desain Klien (Strict / Level 4)

Berkas master pola baju dan gambar desain seragam pesanan klien merupakan **kekayaan intelektual berharga tinggi**. Kebocoran berkas dapat mengakibatkan kloning desain oleh konveksi kompetitor dan hilangnya reputasi bisnis.

```
[ Klien Browser / Tablet ]
           │
           │  1. Request Akses Pola (GET /patterns/:id/download)
           ▼
[ Backend Go (Auth & RBAC Check) ]
           │
           ├─► Validasi Role: Apakah user 'owner', 'designer', atau 'cutting_foreman'?
           ├─► Validasi Tenant: Apakah pattern_file milik tenant pemanggil?
           ├─► Audit Trail: INSERT INTO pattern_download_logs (user_id, file_id, ip, time)
           │
           │  2. Generate Presigned URL (TTL: 15 Menit) + Watermark Parameter
           ▼
[ MinIO / AWS S3 (Private Bucket) ] ────► Tautan Unduh Terenkripsi (Expiring Signature)
```

### Aturan Rekayasa Wajib untuk Modul Pola (`kms` / SPEC-007):
1. **Konfigurasi Bucket MinIO / S3**:
   - Seluruh bucket pola garmen wajib berstatus **Private**. Matikan semua *anonymous read/write policies*.
   - Jalur penyimpanan (*storage key*) wajib diacak menggunakan UUIDv7:
     ```text
     s3://sewerp-patterns/{tenant_id}/{pattern_id}/{uuidv7}.dxf
     ```
   - Dilarang keras menggunakan nama file asli kiriman klien sebagai S3 key guna mencegah *path traversal* atau kebocoran nama klien.
2. **Tautan Akses Bertenggat (Presigned URL)**:
   - Unduhan berkas hanya boleh diakses melalui *Presigned URL* yang dibuat on-demand oleh backend Go dengan durasi kedaluwarsa **maksimal 15 menit (900 detik)**.
3. **Pemberian Stempel Air Dinamis (Dynamic Visual Watermarking)**:
   - Saat file pola berformat PDF, gambar mockup, atau lembar *Tech Pack* dibuka di pratinjau browser, backend/worker River wajib menyematkan stempel semi-transparan diagonal:
     ```text
     "PROPERTY OF [TENANT_NAME] — ACCESSED BY [USER_NAME] ([USER_ID]) ON [TIMESTAMP]"
     ```
   - Hal ini bertujuan mencegah tangkapan layar (*screenshot*) atau foto kamera ponsel bocor secara anonim.
4. **Sanitasi Unggahan Berkas (File Upload Hardening)**:
   - **Validasi Magic Bytes**: Backend Go wajib memeriksa integritas tipe berkas menggunakan deteksi *header byte* (bukan hanya ekstensi file `.dxf`, `.pdf`, `.cdr`, `.ai`).
   - **Batas Ukuran**: Maksimal 50 MB per berkas pola.
   - **Penyimpanan Non-Executable**: Partisi storage tidak boleh memiliki permission eksekusi (`noexec`).
5. **Pencatatan Audit Unduhan Pola (Immutable Audit Log)**:
   - Setiap kali presigned URL dibuat, sistem wajib mencatat transaksi ke tabel `pattern_access_logs`:
     ```sql
     CREATE TABLE pattern_access_logs (
         id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
         tenant_id UUID NOT NULL,
         garment_pattern_id UUID NOT NULL,
         pattern_file_id UUID NOT NULL,
         accessed_by UUID NOT NULL,
         ip_address INET NOT NULL,
         user_agent TEXT NOT NULL,
         action_type VARCHAR(20) NOT NULL, -- 'preview' atau 'download'
         created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP
     );
     ```

---

## 5. Standar Perlindungan Data Pribadi (PII - Confidential / Level 3 - UU PDP)

Mengacu pada **Undang-Undang No. 27 Tahun 2022 tentang Perlindungan Data Pribadi (UU PDP)**:

### 1. Data Kontak Pelanggan & Alamat Pengiriman
- **Penyembunyian Nomor Telepon (Data Masking)**:
  - Pada tabel antarmuka umum (seperti daftar antrean pesanan, kanban stasiun, atau dashboard kasir), nomor WhatsApp pelanggan wajib dimaskir:
    - Contoh: `081234567890` ditampilkan sebagai `0812****7890`.
  - Hanya admin sales yang sedang membuka detail penawaran atau kurir yang mencetak Surat Jalan yang dapat melihat nomor utuh.
- **Pembersihan Barcode Pengiriman (`SPEC-003d`)**:
  - Barcode stiker pengiriman (A6) **hanya boleh mengkodekan `SPK-Number` dan `Koli-Index`**.
  - **Dilarang keras menyematkan nomor telepon atau nama pelanggan langsung ke dalam payload teks barcode/QR Code** guna menghindari pemindaian massal pihak luar.

### 2. Privasi Upah & Kasbon Pekerja (`SPEC-006`)
- **Isolasi Akun Pekerja**:
  - Penjahit atau operator hanya boleh melihat riwayat output jahit dan slip upah mingguan miliknya sendiri.
  - Mandor potong/jahit hanya memiliki izin menginput jumlah pcs output, **tidak boleh melihat saldo pinjaman kasbon atau akumulasi upah penjahit**.
- **Kerahasiaan Pinjaman Kasbon**:
  - Data pinjaman kasbon penjahit diklasifikasikan sebagai data rahasia pribadi yang hanya boleh diakses oleh Pekerja bersangkutan, Kasir/HRD, dan Owner.

### 3. Kebijakan Retensi & Anonymization
- Saat pelanggan meminta penghapusan akun atau data sesuai hak UU PDP (*Right to Erasure*):
  - Record data pada kolom `name`, `phone_whatsapp`, `email`, dan `shipping_address` di tabel `customers` diganti menjadi string acak ter-anonymize: `ANON-[HASH]`.
  - Data historis transaksi SPK dan invoice tetap dipertahankan untuk kepatuhan pembukuan keuangan dan audit pajak, namun identitas personal pelanggan terputus.

---

## 6. Standar Keamanan Kredensial & Payment Gateway (Strict / Level 4)

Mengatur integrasi payment gateway Xendit, rekening transfer manual, dan pencatatan transaksi kas (`SPEC-009` & `SPEC-003c`):

### 1. Enkripsi Kredensial Database (Application-Level AES-256-GCM)
Dilarang menyimpan `api_secret_key` dan `webhook_token` Xendit dalam bentuk plaintext di database.
- Backend Go wajib mengenkripsi nilai secret menggunakan **AES-256-GCM** sebelum kueri `INSERT`/`UPDATE` dieksekusi:
  ```go
  // Contoh standar logika enkripsi kredensial
  ciphertext, nonce, err := encryptAES256GCM(plainSecretKey, appMasterKey)
  ```
- Kunci master (`APP_MASTER_ENCRYPTION_KEY`) disimpan di environment variable / secret manager, terpisah dari database.
- Kolom tabel `payment_gateway_configs` menyimpan `secret_key_ciphertext` bertipe `BYTEA` dan `nonce` bertipe `BYTEA`.

### 2. Masking Kredensial pada Antarmuka (UI Settings)
- Endpoint HTTP `GET /api/v1/settings/payments` dilarang mengembalikan secret key mentah.
- Response payload hanya menampilkan awalan 8 karakter:
  ```json
  {
    "xendit_api_key_masked": "xnd_production_••••••••••••••••",
    "xendit_webhook_token_masked": "wht_••••••••••••••••"
  }
  ```

### 3. Validasi Integritas Webhook Xendit
Setiap request callback webhook yang masuk ke endpoint `/api/v1/payments/webhook/xendit` wajib melalui 3 lapisan pengamanan:
1. **Verifikasi Token / Signature Header**:
   - Memeriksa kesesuaian header `x-callback-token` dengan token webhook resmi tenant yang tersimpan.
2. **Pencegahan Serangan Replay (Idempotensi)**:
   - Setiap payload webhook memiliki event ID unik (`event_id` atau `payment_id`).
   - Sistem wajib melakukan pengecekan ke tabel `payment_gateway_logs`. Jika ID transaksi sudah berstatus `PROCESSED`, sistem langsung merespons `200 OK` tanpa memproses ulang pesanan.
3. **Pembatasan IP (Optional IP Whitelisting)**:
   - Nginx proxy menyaring rentang IP resmi server Xendit sebelum meneruskan request ke backend Go.

### 4. Immutability Buku Log Pembayaran (`SPEC-003c`)
- Tabel `order_payments` dan `payment_gateway_logs` bersifat **Append-Only**.
- Hak akses database untuk role aplikasi reguler tidak memiliki wewenang `UPDATE` atau `DELETE` pada riwayat transaksi kas yang sudah berstatus `PAID`.
- Koreksi pembayaran hanya dapat dilakukan melalui transaksi jurnal pembalik (*adjustment/refund log*) baru dengan otorisasi role `Owner`.

---

## 7. Autentikasi, Otorisasi (RBAC) & Isolasi Multi-Tenancy

### 1. Matriks Hak Akses Peran (Role-Based Access Control)

| Peran (Role) | Kode Peran | Modul yang Dapat Diakses | Akses Data Finansial / PII | Akses File Pola Master |
| :--- | :--- | :--- | :--- | :--- |
| **Owner Konveksi** | `owner` | Seluruh Modul (Akses Penuh) | Penuh (Omset, Piutang, Gaji, Kasbon, PII) | Penuh (Unduh & Kelola) |
| **Manajer Produksi** | `prod_manager` | MES (004), WMS (005), KMS (007), Dashboard (001) | Terbatas (Tanpa nominal omset/uang) | Penuh (Unduh & Kelola) |
| **Mandor Potong** | `cutting_foreman`| MES (Stasiun Cutting), WMS (Ambil Kain & Retur Rol), KMS (007) | Tanpa Akses Finansial & PII | Unduh Pola Terkait SPK Aktif Saja |
| **Operator Jahit** | `sewing_operator`| Form Input Harian (006), Rincian Slip Mandiri | Hanya slip gaji & kasbon miliknya sendiri | Tanpa Akses |
| **Staf QC & Packing** | `qc_packer` | MES (QC Inspeksi), OMS (Cetak Surat Jalan & Stiker A6) | Terbatas (Nama & Alamat Pengiriman) | Tanpa Akses |
| **Admin Gudang** | `warehouse_admin`| WMS (Inventaris Kain & Aksesoris), OMS (Cetak Surat Jalan) | Terbatas (Stok Fisik & Alamat Pengiriman) | Tanpa Akses |
| **Kasir & Finance** | `cashier_finance`| OMS (003), Payment (009), Payroll (006), Reporting (008) | Penuh (Uang Masuk, Bank, Kasbon, Slip Gaji) | Tanpa Akses |
| **Admin Sales** | `sales_admin` | CRM (002), OMS (003), Dashboard (001 - Ringkas) | PII Pelanggan & Status Pesanan Saja | Hanya Gambar Mockup (Tanpa Pola CAD) |

### 2. Standar Token Sesi & JWT
- **Access Token**: JWT berdurasi pendek (**15 menit**), ditandatangani menggunakan algoritma asymmetric **Ed25519** atau symmetric **HMAC-SHA256**.
  - Payload JWT wajib memuat: `user_id`, `tenant_id`, `role`, `session_id`, `exp`.
- **Refresh Token**: Disimpan di dalam cookie browser dengan konfigurasi proteksi maksimal:
  ```http
  Set-Cookie: refresh_token=...; HttpOnly; Secure; SameSite=Strict; Path=/api/v1/auth; Max-Age=604800
  ```
- **Pencabutan Sesi (Instant Revocation)**: Logout mencatat `session_id` ke dalam memori blacklist (Redis / local fast-cache) untuk membatalkan akses secara instan sebelum masa berlaku token berakhir.

### 3. Jaring Pengaman Multi-Tenancy (PostgreSQL RLS)
Selain kewajiban menyertakan `WHERE tenant_id = $1` pada setiap kueri repository sqlc di backend Go, PostgreSQL dikonfigurasi dengan Row-Level Security:

```sql
-- Mengaktifkan RLS pada tabel sensitif
ALTER TABLE sales_orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE garment_patterns ENABLE ROW LEVEL SECURITY;
ALTER TABLE worker_cash_advances ENABLE ROW LEVEL SECURITY;

-- Policy isolasi berdasarkan session setting
CREATE POLICY tenant_isolation_policy ON sales_orders
    FOR ALL
    USING (tenant_id = NULLIF(current_setting('app.current_tenant_id', true), '')::uuid);
```

Middleware backend Go menyetel session variable saat meminjam koneksi dari pool:
```sql
SET LOCAL app.current_tenant_id = 'tenant-uuid-here';
```

---

## 8. Keamanan API, Jaringan & Observabilitas

1. **Pembatasan Beban API (Rate Limiting)**:
   - Mencegah serangan *Brute Force* dan *Scraping Data*:
     - Endpoint Autentikasi (`/auth/login`, `/auth/refresh`): Maksimal **5 request / menit / IP**.
     - Endpoint Ekspor Dokumen (`/reports/*/export`, `/orders/*/print`): Maksimal **10 request / menit / tenant**.
     - Endpoint Operasional Umum: Maksimal **100 request / menit / user**.
2. **HTTP Security Headers (Nginx Configuration)**:
   ```nginx
   add_header X-Frame-Options "DENY" always;
   add_header X-Content-Type-Options "nosniff" always;
   add_header X-XSS-Protection "1; mode=block" always;
   add_header Referrer-Policy "strict-origin-when-cross-origin" always;
   add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self' https://api.xendit.co;" always;
   add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
   ```
3. **Penyaringan Log Terstruktur (Zerolog PII Redaction)**:
   - Konfigurasi logger JSON Zerolog di backend Go wajib menggunakan interceptor penapis (*filter hook*) untuk menyamarkan kata kunci rahasia:
     ```text
     "password" -> "[REDACTED]"
     "secret_key" -> "[REDACTED]"
     "webhook_token" -> "[REDACTED]"
     "phone_whatsapp" -> "0812****7890"
     ```

---

## 9. Matriks Audit Keamanan Komprehensif Seluruh Modul Spesifikasi

Berikut adalah rekapitulasi audit keamanan terhadap seluruh berkas spesifikasi SewERP:

| No | Spesifikasi Fitur | Bidang Data Kritis | Klasifikasi | Kontrol Keamanan Wajib |
| :---: | :--- | :--- | :---: | :--- |
| 1 | **[001a-dashboard-overview-metrics.md](../specs/001a-dashboard-overview-metrics.md)** | Agregasi omset, piutang, dan estimasi upah borongan. | **Level 3** | RBAC khusus Owner & Finance. Query wajib dibatasi `tenant_id`. |
| 2 | **[002a-customer-management.md](../specs/002a-customer-management.md)** | Kontak WhatsApp, email, dan alamat pengiriman pelanggan. | **Level 3 (PII)** | Masking nomor telepon pada UI list. Audit trail pada ekspor kontak. Validasi anti-duplikasi nomor. |
| 3 | **[003a-sales-order-core.md](../specs/003a-sales-order-core.md)** | Nominal kontrak SPK, rincian diskon, dan persetujuan tempo. | **Level 3** | Kalkulasi harga murni sisi server (*anti-tampering*). Kunci state machine SPK pasca-DP. |
| 4 | **[003b-delivery-note.md](../specs/003b-delivery-note.md)** | Alamat fisik penerima dan nomor kontak telepon. | **Level 3 (PII)** | Tautan cetak PDF berotentikasi. Token unduh sementara. |
| 5 | **[003c-sales-receipt.md](../specs/003c-sales-receipt.md)** | Nilai pembayaran kasir dan identitas petugas. | **Level 3** | QR Code verifikasi integritas struk. Riwayat cetak immutable. |
| 6 | **[003d-payment-log.md](../specs/003d-payment-log.md)** | Mutasi perbankan, nomor referensi bank, identitas verifikator. | **Level 3 & 4** | Tabel append-only (larangan UPDATE/DELETE). Audit log per transaksi masuk. |
| 7 | **[003e-shipping-label.md](../specs/003e-shipping-label.md)** | Stiker karung/koli pakaian (nama pemesan, alamat, no telepon). | **Level 3 (PII)** | Barcode hanya memuat `SPK-Number` + `Koli-Index`. Dilarang menaruh nomor HP di barcode. |
| 8 | **[003f-billing-statement.md](../specs/003f-billing-statement.md)**| Surat tagihan pelunasan, nomor rekening transfer konveksi. | **Level 3** | Rekening tujuan pembayaran divalidasi silang dengan konfigurasi resmi tenant (*anti-spoofing*). |
| 9 | **[004a-production-tracking.md](../specs/004a-production-tracking.md)** | Tiket bundel kerja fisik, data rekanan makloon, data reject stasiun. | **Level 2 & 3** | Barcode tiket bundel divalidasi ke UUID entitas aktif. Verifikasi serah terima pcs makloon. |
| 10 | **[005a-fabric-inventory.md](../specs/005a-fabric-inventory.md)** | HPP kain (`cost_per_unit`), mutasi timbangan retur sisa rol. | **Level 2 & 3** | Field-level security: HPP disembunyikan dari staf lantai pabrik. Log retur rol anti-manipulasi. |
| 11 | **[006a-worker-roster-tariffs.md](../specs/006a-worker-roster-tariffs.md)** | Data penjahit, catatan pinjaman kasbon, slip gaji mingguan. | **Level 3 (PII)** | Isolasi slip upah antar-penjahit. Transaksi lock (`FOR UPDATE`) untuk pencegahan *overclaim race condition*. |
| 12 | **[007a-pattern-techpack.md](../specs/007a-pattern-techpack.md)** | **Pola Baju Master CAD/DXF/PDF, Tech Pack, mockup logo klien**. | **Level 4 (STRICT)** | **Private S3 Bucket, Presigned URL (15 menit), Dynamic Watermark, Audit Log unduhan, sanitasi MIME type**. |
| 13 | **[008a-sales-receivables-report.md](../specs/008a-sales-receivables-report.md)** | Laporan laba/rugi, rekapitulasi penggajian pabrik, ekspor Excel. | **Level 3** | Rate limiting ekspor CSV/Excel. Audit trail aktor pengunduh laporan. RBAC khusus Owner & Finance. |
| 14 | **[009a-payment-settings.md](../specs/009a-payment-settings.md)** | **API Secret Key Xendit, Webhook Token, Signature Callback**. | **Level 4 (STRICT)** | **Enkripsi AES-256-GCM di DB, masking di UI, verifikasi signature webhook HMAC SHA-256, Idempotensi ID**. |
| 15 | **[010a-google-sso.md](../specs/010a-google-sso.md)** | **Google OAuth2 Client Secret, HttpOnly Session Cookie**. | **Level 4 (STRICT)** | **Proteksi cookie 8 jam, CSRF state verification, whitelist invitation check**. |
| 16 | **[011a-matrix-price-list.md](../specs/011a-matrix-price-list.md)** | Struktur tarif acuan, diskon tiering, dan audit override. | **Level 2** | Jejak audit `is_price_overridden` dan isolasi multi-tenant. |

---

## 10. Checklist Kepatuhan Pengembang (Developer Security Checklist)

Sebelum kode di-merge ke branch utama atau dideploy ke lingkungan staging/produksi:

- [ ] Tidak ada password, secret key, atau API token yang di-commit ke repositori git (lulus scan `git-secrets` / `trufflehog`).
- [ ] Seluruh kueri database menyertakan klausa tenant `tenant_id = $1` dan tabel sensitif memiliki policy RLS.
- [ ] Nilai nominal uang menggunakan `BIGINT` integer tanpa floating-point.
- [ ] Endpoint unduh berkas pola pakaian menggunakan Presigned URL berdurasi $\le 15$ menit.
- [ ] Masking nomor telepon dan identitas sensitif aktif pada tampilan antarmuka reguler dan logger JSON.
- [ ] Endpoint callback webhook memverifikasi signature kriptografis dan memeriksa idempotensi event ID.
- [ ] Endpoint ekspor CSV/Excel dilindungi rate limiter dan mencatat jejak audit pengunduhan.
