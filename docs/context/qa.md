# Standar QA & Pengujian Sistem SewERP

Dokumen ini mendefinisikan strategi pengujian, piramida tes, skenario uji kritis, protokol pengujian berbasis browser (*QA Browser Access*), pengujian tolak ukur performa (*Benchmark Testing: Zero Overhead, Zero Allocations & Zero Footprint*), serta kriteria penerimaan otomatisasi untuk menjamin keandalan sistem ERP konveksi.

---

## Rekomendasi Skills untuk Agent (Recommended Agent Skills)

Ketika agen bertugas merancang skenario uji, menjalankan audit browser, atau memvalidasi tolak ukur performa pada sistem SewERP, aktifkan dan kombinasikan skills berikut:
- `chrome-devtools`: Menjalankan otomasi pengujian browser, emulasi perangkat bergerak/tablet pabrik, inspeksi konsol JavaScript, dan simulasi pembatasan jaringan (*network throttling*).
- `a11y-debugging`: Memverifikasi kepatuhan aksesibilitas WCAG AA, keabsahan atribut ARIA pada komponen headless Kobalte, kontras warna minimal 4.5:1, dan ukuran target sentuh 48px.
- `debug-optimize-lcp`: Menganalisis metrik Core Web Vitals (Largest Contentful Paint < 1.2s), waktu muat aset, dan efisiensi render antarmuka SolidJS.
- `memory-leak-debugging`: Memeriksa heap snapshot JavaScript di Chrome DevTools, memvalidasi pelepasan referensi memori pada `onCleanup()`, dan memastikan 0 detached DOM nodes.
- `troubleshooting`: Mendiagnosis kendala konektivitas target browser, headless runner, atau kegagalan inisialisasi lingkungan pengujian.

---

## 1. Strategi & Piramida Pengujian

Pengujian perangkat lunak di SewERP dibagi menjadi tiga tingkatan berjenjang:

```text
       ▲
      / \        15% E2E Browser Tests (Playwright & Chrome DevTools MCP)
     /───\
    /     \      25% Integration Tests (Go Test + Postgres Testcontainers)
   /───────\
  /         \    60% Unit Tests (Domain Math, Borongan, Wastage, Validasi)
 ─────────────
```

1. **Unit Tests (60%)**: Menguji fungsi murni tanpa I/O database. Fokus pada formula perhitungan harga bertingkat, penambahan wastage margin kain, kalkulasi upah borongan per penjahit, pemotongan kasbon, surcharge ukuran jumbo, dan validasi form DTO.
2. **Integration Tests (25%)**: Menguji interaksi repository sqlc dengan instance PostgreSQL nyata, memastikan query SQL, constraint foreign key, partial index, dan isolation level `SELECT FOR UPDATE` berfungsi tepat pada mutasi stok dan alokasi SPK.
3. **End-to-End Browser Tests (15%)**: Menjalankan skenario browser otomatis untuk alur lengkap pesanan: dari input Quotation -> SPK -> Pembagian Bundel Jahit -> Input Borongan -> Pelunasan -> Surat Jalan.

---

## 2. Standar Akses Browser untuk QA (QA Browser Access & Tooling)

Untuk memastikan keandalan antarmuka di lapangan pabrik, QA dilengkapi dengan akses otomasi browser modern menggunakan **Playwright** dan integrasi **Chrome DevTools MCP**:

### A. Emulasi Perangkat Seluler & Tablet Pabrik
QA wajib menguji tampilan responsif pada 3 resolusi perangkat kunci:
- **Mobile Smartphone Operator (375x667 & 390x844)**: Memastikan form input output harian penjahit dapat diisi nyaman dengan satu tangan dan target sentuh minimal 48px.
- **Tablet Mandor Meja Potong & QC (768x1024 & 800x1280)**: Memastikan papan Kanban 6 stasiun kerja dan tabel matriks ukuran tampil leluasa tanpa pergeseran elemen (*layout shifts*).
- **Desktop Admin / Kasir (1440x900 & 1920x1080)**: Memastikan dashboard bento-grid dan form pembuatan penawaran harga tampil padat dan ergonomis.

### B. Pengujian Cetak Fisik Thermal di Browser (`@media print`)
QA wajib memverifikasi pratinjau cetak (*print preview*) langsung melalui browser engine:
- Memastikan tiket bundel thermal 80mm dan label stiker A6 tidak terpotong tepi halamannya (*zero overflow*).
- Memastikan barcode dan QR code ter-render tajam dalam mode monokrom hitam-putih murni.

### C. Inspeksi Jaringan & Penanganan Offline (Chrome DevTools MCP)
- Menguji ketahanan formulir saat terjadi *network throttling* (simulasi koneksi 3G/Slow Wi-Fi pabrik).
- Memverifikasi penyimpanan draft sementara di browser saat koneksi internet terputus, memastikan data ketikan mandor tidak hilang.
- Memeriksa konsol browser bebas dari JavaScript error, unhandled promise rejections, dan memory leak.

---

## 3. Standar Uji Tolak Ukur (Benchmark Testing): Zero Overhead, Zero Alloc, Zero Footprint

QA dan Lead Engineer wajib menjalankan pengujian tolak ukur (*benchmarks*) secara berkala untuk memvalidasi efisiensi memori dan CPU:

### A. Backend Go Benchmarks
- **Perintah Eksekusi**:
  ```bash
  go test -bench=. -benchmem -run=^$ ./internal/...
  ```
- **Kriteria Ambang Kelulusan (Gate Threshold)**:
  - Fungsi aritmatika mata uang & diskon: **`0 B/op`** dan **`0 allocs/op`**.
  - Kalkulasi margin susut kain (wastage margin): **`0 B/op`** dan **`0 allocs/op`**.
  - Validasi penjumlahan matriks ukuran garmen: **`0 B/op`** dan **`0 allocs/op`**.
  - Verifikasi anti-overclaim upah borongan: **`0 B/op`** dan **`0 allocs/op`**.
- **Deteksi Kebocoran Memori (Zero Footprint)**:
  - Menjalankan uji beban 1.000 simulasi request paralel menggunakan `pprof`.
  - Memastikan jumlah goroutine kembali ke baseline awal (zero goroutine leaks).

### B. Frontend Browser Memory & Rendering Benchmarks
- **Deteksi Detached DOM Nodes (Zero Memory Leaks)**:
  - Mengambil Chrome DevTools Heap Snapshot sebelum dan sesudah membuka dialog modal matriks ukuran 50 kali.
  - Memastikan tidak ada *detached HTML elements* yang tertahan di memori.
- **Efisiensi Render Tabular**:
  - Render tabel matriks ukuran garmen dan daftar 500 baris inventaris kain wajib mempertahankan kecepatan render 60 FPS tanpa jank.
  - Memastikan seluruh event listener kamera dan timer dibersihkan sempurna melalui `onCleanup()`.

---

## 4. Skenario Uji Kritis Operasional Konveksi

### A. Validasi Kalkulasi Upah Borongan & Potongan Kasbon
- **Kondisi Uji**: Penjahit "Ahmad" menyelesaikan 50 pcs jahit kemeja (tarif Rp 12.000/pcs = total upah kotor Rp 600.000). Ahmad memiliki kasbon tercatat sebesar Rp 150.000 di minggu tersebut.
- **Hasil yang Diharapkan**: Sistem otomatis menghitung upah bersih pada slip gaji mingguan:
  $$\text{Upah Bersih} = \text{Rp } 600.000 - \text{Rp } 150.000 = \text{Rp } 450.000$$
  Status kasbon berubah menjadi `deducted` (lunas terpotong).

### B. Validasi Anti-Overclaim Borongan
- **Kondisi Uji**: SPK `PO-001` memiliki hasil cutting kemeja sebanyak 100 pcs.
- **Kasus Uji Negatif**: Penjahit A mengklaim 60 pcs, Penjahit B mengklaim 50 pcs (total 110 pcs).
- **Hasil yang Diharapkan**: Sistem menolak klaim Penjahit B dengan kode error `422 Unprocessable Entity` dan pesan: *"Total klaim operasi jahit (110 pcs) melebihi batas hasil potong yang tersedia (100 pcs)"*.

### C. Validasi Surcharge Ukuran Jumbo
- **Kondisi Uji**: Kaos Combed 30s harga dasar Rp 50.000. Matriks surcharge: XXL (+Rp 5.000), 3XL (+Rp 10.000).
- **Kasus Uji**: Pelanggan memesan 10 pcs L, 5 pcs XXL, dan 2 pcs 3XL.
- **Hasil yang Diharapkan**: Subtotal terhitung otomatis secara presisi:
  $$(10 \times 50.000) + (5 \times 55.000) + (2 \times 60.000) = 500.000 + 275.000 + 120.000 = \text{Rp } 895.000$$

### D. Validasi Selisih Pcs Pengembalian Makloon Luar
- **Kondisi Uji**: 200 pcs kain potong dikirim ke vendor makloon sablon "Berkah Screen".
- **Kasus Uji**: Vendor mengembalikan 196 pcs lolos sablon dan 4 pcs cacat sablon (rusak tinta meluber).
- **Hasil yang Diharapkan**: Sistem mencatat penerimaan 196 pcs ke stasiun jahit, mencatat 4 pcs sebagai reject makloon, dan menerbitkan peringatan "Potong Tambal 4 Pcs" untuk bagian cutting.

### E. Validasi Toleransi Penyusutan Bahan (Wastage Margin 3% - 5%)
- **Kondisi Uji**: Kebutuhan dasar 350 kg kain, margin susut 4%.
- **Hasil yang Diharapkan**: Sistem mengalokasikan:
  $$350 \times (1 + 0.04) = 364\text{ kg}$$
  Jika stok bebas < 364 kg, sistem mencegah proses cutting dengan status `Pending Restock`.

---

## 5. Matriks Pengujian Regresi & Otomatisasi

| Modul | Lingkup Uji | Alat Pengujian | Kriteria Lolos |
| :--- | :--- | :--- | :--- |
| **CRM & Price List** | Lookup harga tiering, add-on, & size surcharge | Go Unit Test | 100% presisi nominal rupiah, zero rounding error |
| **Sales & OMS** | Transisi status SPK, nominal DP fleksibel, Surat Jalan | Go Integration Test | State-machine tidak melompati fase, DP valid |
| **Production (MES)**| 6 stasiun kerja, Surat Jalan Makloon, cetak tiket | Playwright E2E Browser | Tiket thermal presisi 80mm, selisih makloon terdeteksi |
| **Inventory (WMS)** | Alokasi kain (+4% margin), retur kain sisa rol potong | Go Transaction Test | Tidak ada stok minus, sisa rol tercatat kembali |
| **Payroll Borongan**| Rekap harian output jahit, potongan kasbon, slip gaji | Go Unit Test & Browser | Anti-overclaim aktif, pemotongan kasbon akurat |
| **Frontend UI/UX** | Input matriks ukuran, target sentuh 48px, offline draft | Playwright + Chrome DevTools | Responsif tablet/HP, form tidak hilang saat offline |
| **Performance Benchmark** | Hot paths math & memory profiling | Go `testing.B` & Heap Snapshot | `0 B/op`, `0 allocs/op`, 0 detached DOM nodes |

---

## 6. Referensi & Dokumen Terkait

- 🏛️ Standar Arsitektur Sistem: [architecture.md](./architecture.md)
- ⚙️ Standar Rekayasa Backend: [backend.md](./backend.md)
- 🖥️ Standar Rekayasa Frontend: [frontend.md](./frontend.md)
- 📋 Aturan Bisnis & Logika Konveksi: [business_requirements.md](./business_requirements.md)
- 🔧 Kebutuhan Teknis Non-Fungsional: [technical_requirements.md](./technical_requirements.md)
- 🧵 Spesifikasi Modul Produksi: [../specs/004a-production-tracking.md](../specs/004a-production-tracking.md)
- 👥 Spesifikasi HRIS & Payroll: [../specs/006a-worker-roster-tariffs.md](../specs/006a-worker-roster-tariffs.md)
- 📋 Standar Dokumentasi Requirement: [requirements_standard.md](./requirements_standard.md)
