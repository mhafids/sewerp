# Standar Kebutuhan Teknis Non-Fungsional SewERP

Dokumen ini mendefinisikan kriteria kinerja teknis, Service Level Objectives (SLO), standar tolak ukur performa (*Zero Overhead, Zero Allocations & Zero Footprint*), keamanan, integrasi pencetakan fisik, dan protokol komunikasi eksternal untuk SewERP.

---

## Rekomendasi Skills untuk Agent (Recommended Agent Skills)

Ketika agen bertugas memvalidasi pemenuhan target SLO latensi P95, optimasi alokasi memori nol (`0 B/op`), pengujian cetak thermal, atau integrasi WhatsApp:
- `debug-optimize-lcp`: Mengaudit performa muat halaman frontend, waktu render first-paint, dan pemenuhan ambang batas LCP < 1.2 detik.
- `memory-leak-debugging`: Memverifikasi ketiadaan memory leak pada browser, integritas snapshot memori DOM, dan pembersihan objek.
- `chrome-devtools`: Menjalankan verifikasi cetak fisik `@media print` thermal 80mm dan label stiker A6 secara presisi tanpa margin overflow.
- `spec-kit`: Mengaudit kriteria penerimaan non-fungsional dan checklist konsistensi performa pada setiap fase delivery.

---

## 1. Kinerja & Service Level Objectives (SLO)

| Metrik Kinerja | Target Batas | Strategi Pemenuhan |
| :--- | :--- | :--- |
| **Latensi API Pembacaan (Read)** | P95 < 100ms | Indeks compound PostgreSQL terindeks `(tenant_id, created_at DESC)` |
| **Latensi API Penulisan (Write)** | P95 < 250ms | Transaksi Go minimalis dengan sqlc, tanpa overhead ORM |
| **Waktu Muat Halaman (LCP)** | < 1.2 detik | SolidJS SPA tanpa Virtual DOM, CSS atomik UnoCSS instan |
| **Alokasi Memori Backend (Hot Paths)**| **`0 B/op`, `0 allocs/op`** | Value receiver murni, zero escape to heap pada kalkulasi finansial & matriks |
| **Footprint Memori Frontend** | **Zero Memory Leaks** | Pembersihan listener di `onCleanup()`, 0 detached DOM nodes pada Heap Snapshot |
| **Kepatuhan Tipe TypeScript** | **100% Strict** | `strict: true`, `noImplicitAny: true`, 0 penggunaan tipe `any` |
| **Ketersediaan Sistem (Uptime)** | 99.5% | Docker restart policy `unless-stopped`, healthcheck monitoring |
| **Ukuran Dokumen PDF / Cetak** | < 300 KB | Layout native `@media print` dan optimasi font embedding |

---

## 2. Standar Uji Tolak Ukur (Zero Overhead, Zero Memory Allocations & Zero Footprint)

### A. Backend Go Engine
- **Zero Allocations Rule**: Seluruh fungsi domain di bawah `internal/*/domain` yang melakukan kalkulasi nominal mata uang (`types.Money`), penambahan margin susut bahan (*wastage margin*), rekapitulasi matriks ukuran garmen, dan validasi anti-overclaim borongan wajib diuji dengan `testing.B` dan lulus dengan ambang batas **`0 B/op` dan `0 allocs/op`**.
- **Escape Analysis**: Verifikasi `go build -gcflags="-m"` wajib menunjukkan bahwa variabel-variabel operasional tidak berpindah dari stack ke heap.
- **Zero Footprint**: Profiling memori via `pprof` harus menunjukkan pelepasan alokasi sempurna dan tidak ada kebocoran goroutine setelah pengujian beban 1.000 request bersamaan.

### B. Frontend SolidJS Engine
- **Zero Virtual DOM Overhead**: Memanfaatkan arsitektur kompilasi reaktif SolidJS yang memperbarui DOM secara langsung pada node target tanpa proses reconciler pohon Virtual DOM.
- **Strict Typing Compliance**: Seluruh state, DTO response, dan form input wajib diikat oleh interface TypeScript konkret tanpa pengecualian tipe `any` atau `as unknown as Type`.
- **Zero Footprint DOM**: Pengujian Chrome DevTools Heap Snapshot wajib membuktikan bahwa komponen modal dan pemindai QR kamera melepaskan seluruh referensi memori DOM saat ditutup (*zero detached DOM trees*).

---

## 3. Kebutuhan Cetak Fisik & Thermal Printing

Workshop konveksi membutuhkan integrasi pencetakan cepat langsung dari browser tanpa driver khusus:

### A. Format Tiket Bundel Produksi (Work Order Ticket)
- **Ukuran Kertas**: Printer Thermal 80mm roll atau stiker thermal ukuran A6 (100mm x 150mm).
- **Isi Dokumen**:
  - Barcode / QR Code nomor SPK & ID Bundel.
  - Nama artikel & varian warna kain.
  - Rincian ukuran dalam bundel (misal: Size L - 24 pcs).
  - Instruksi khusus (letak sablon dada kiri, warna benang, jenis kancing).
  - Kotak paraf checklist 6 stasiun kerja (Cutting -> Makloon/Print -> Sewing -> QC -> Packing).
- **Styling CSS**: Menggunakan aturan `@media print` dengan margin 0mm dan kontras hitam putih murni tanpa warna abu-abu tipis agar tajam pada pemanas thermal head.

### B. Dokumen Standar Kantor (A4 / Folio)
- Digunakan untuk: Penawaran Harga (*Quotation*), SPK Arsip Pabrik, Faktur Tagihan (*Invoice*), dan Surat Jalan Resmi (*Delivery Note*).
- Dihasilkan dengan opsi simpan PDF instan atau cetak langsung menggunakan printer ink/laserjet.

---

## 4. Integrasi Pengiriman Pesan WhatsApp

Sistem menyediakan dua tingkat integrasi pengiriman dokumen ke nomor WhatsApp pelanggan:

### A. Level 1: Deep Link Click-to-Chat (Zero Cost)
- Menghasilkan tautan langsung `https://wa.me/62812xxxx?text=...` dengan teks template otomatis yang rapi dan menyertakan URL publik download dokumen/penawaran.
- Dapat langsung diklik oleh Admin Penjualan dari antarmuka web.

### B. Level 2: Webhook WA Gateway Otomatis
- Mengintegrasikan antrean background job **River** dengan REST API gateway WhatsApp lokal (seperti Baileys/WPPConnect/Fonnte).
- Mengirimkan notifikasi instan saat penawaran terbit, DP diterima, pesanan selesai, dan Surat Jalan diterbitkan.

---

## 5. Keamanan, Autentikasi & Isolasi Data

1. **Autentikasi Eksklusif Google SSO**: Login hanya melalui Google OAuth2/OIDC. Tidak ada mekanisme login kata sandi lokal untuk mencegah eksploitasi credential stuffing dan kelemahan password di lantai pabrik.
2. **Backend Session Storage (PostgreSQL & HttpOnly Cookie)**:
   - Token sesi disimpan di basis data PostgreSQL (tabel `sessions`) dan diverifikasi menggunakan hash SHA-256.
   - Dikirimkan ke peramban pengguna dalam format cookie `HttpOnly; Secure; SameSite=Lax; Max-Age=28800` (8 jam / 1 shift kerja).
   - **Zero XSS Exposure**: Kredensial tidak dapat diakses atau dibaca oleh skrip JavaScript di sisi klien (*anti-token theft*).
3. **Pengecekan Izin Zero-Allocation (Gin Middleware)**:
   - Validasi hak akses (`RequirePermission`) pada hot path HTTP routing dieksekusi dengan *O(1) in-memory bitmask* / *hash set lookup* berkinerja tinggi, memenuhi ambang batas **`0 B/op` dan `0 allocs/op`** tanpa alokasi memori heap berulang per request.
4. **Isolasi Multi-Tenant Mutlak**: Setiap query database terisolasi melalui filter `tenant_id`. Kegagalan menyertakan filter tenant akan dideteksi oleh static analyzer query.
5. **Pencatatan Audit Trail**: Setiap perubahan status pesanan, penyesuaian stok manual di gudang, dan approval SPK disimpan permanen pada tabel `audit_logs` bersama ID pengguna, IP address, dan timestamp.

---

## 6. Referensi & Dokumen Terkait

- 🏛️ Standar Arsitektur Sistem: [architecture.md](./architecture.md)
- ⚙️ Standar Rekayasa Backend: [backend.md](./backend.md)
- 🖥️ Standar Rekayasa Frontend: [frontend.md](./frontend.md)
- 🧪 Standar QA & Benchmark: [qa.md](./qa.md)
- 📝 Standar Formulir & Validasi: [forms.md](./forms.md)
- 🔐 Spesifikasi Autentikasi & RBAC (IAM): [../specs/010a-google-sso.md](../specs/010a-google-sso.md)
- 📄 Spesifikasi Sales & Order (OMS): [../specs/003a-sales-order-core.md](../specs/003a-sales-order-core.md)
- 🧵 Spesifikasi Modul Produksi: [../specs/004a-production-tracking.md](../specs/004a-production-tracking.md)
- 📋 Standar Dokumentasi Requirement: [requirements_standard.md](./requirements_standard.md)
