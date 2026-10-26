# Spesifikasi Sub-Modul: 003e - Cetak Stiker Alamat Pengiriman Koli (Shipping Label)

Dokumen ini memuat spesifikasi fungsional, struktur data, tata letak cetak PDF, dan skema ekspor Excel untuk **Stiker Alamat Pengiriman (Shipping Label)** yang ditempel pada karung/koli/karton pakaian konveksi sebelum diserahkan ke kurir atau ekspedisi cargo.

---

## 1. Metadata & Tautan Referensi Terkait

- **Kode Modul**: `SPEC-003e`
- **Domain Bounded Context**: `oms`
- **Tautan Standar Acuan Global**:
  - 📋 [Standar Penulisan Requirement](../context/requirements_standard.md)
  - 🔧 [Standar Kebutuhan Teknis & Cetak Thermal](../context/technical_requirements.md)
  - 👆 [Prinsip UX Lapangan Pabrik](../context/ux.md)
  - 🧪 [Standar QA & Benchmark Testing](../context/qa.md)
- **Tautan Spesifikasi Fitur Terkait**:
  - 📄 [003a - Inti Sales & Order (OMS)](./003a-sales-order-core.md)
  - 🚚 [003b - Cetak Surat Jalan](./003b-delivery-note.md)
  - 👥 [002a - Manajemen Master Pelanggan](./002a-customer-management.md)

---

## 2. Struktur Data & Sumber Bidang (Data Source DTO)

Data ditarik dari entitas `sales_orders`, `customers`, dan `delivery_notes`:

| Bidang Data | Tipe Data | Keterangan |
| :--- | :--- | :--- |
| `order_number` | `varchar(32)` | Nomor SPK rujukan |
| `barcode_data` | `varchar(64)` | Kode Barcode 128 / QR Code nomor SPK + Koli |
| `koli_index` | `int` | Nomor urut koli (misal: 1) |
| `koli_total` | `int` | Jumlah total koli dalam pengiriman (misal: 4) |
| `sender_name` | `varchar(100)` | Nama usaha konveksi tenant |
| `sender_phone` | `varchar(20)` | Kontak WhatsApp konveksi |
| `sender_city` | `varchar(50)` | Kota asal pengiriman |
| `recipient_name` | `varchar(100)` | Nama penerima / instansi pemesan |
| `recipient_phone`| `varchar(20)` | Nomor kontak telepon / WhatsApp penerima |
| `destination_address`| `text` | Alamat lengkap tujuan pengiriman |
| `destination_city` | `varchar(50)` | Kota/Kabupaten & Kode Pos tujuan |
| `courier_name` | `varchar(50)` | Nama ekspedisi (JNE Trucking, Dakota, Indah Cargo, dll.) |
| `service_type` | `varchar(30)` | Jenis layanan (Trucking, Cargo Darat, Reguler) |
| `package_weight` | `decimal(6,2)` | Berat estimasi koli (kg) |
| `handling_instructions`| `text` | Peringatan: "Pakaian Jadi - Hindarkan dari Basah/Benda Tajam" |

---

## 3. Spesifikasi Tata Letak Cetak (PDF Layout)

### Standar Fisik & Margin
- **Ukuran Label**: Standar Thermal A6 (100mm x 150mm).
- **Margin**: 4mm sekeliling label agar isi tidak terpotong tepi pemanas print head.
- **Warna**: Hitam putih murni (100% Black `#000000`) dengan border tegas 2px untuk keterbacaan barcode scanner.

### Blok Susunan Stiker (Vertikal)
1. **Bagian Atas (Header Ekspedisi & Barcode)**:
   - Nama Kurir / Ekspedisi & Jenis Layanan dalam font bold besar.
   - Barcode Code128 / QR Code dengan teks nomor SPK di bawahnya.
   - Tanda Indikator Koli: Tulisan besar mencolok **KOLI: [ 1 / 4 ]**.
2. **Bagian Tengah (Penerima vs Pengirim)**:
   - Kotak Utama (Penerima): Judul "KEPADA", Nama Penerima (Bold 14pt), No. Telp, Alamat Lengkap, Kota Tujuan & Kode Pos.
   - Kotak Sekunder (Pengirim): Judul "DARI", Nama Konveksi, Kota Asal, No. Kontak.
3. **Bagian Bawah (Rincian Isi & Penanganan)**:
   - Deskripsi Paket: Jenis garmen, estimasi berat (misal: 25.5 kg).
   - Label Peringatan Khusus: Ikon / teks **"AWAS BASAH / JANGAN DIBANTING"**.

---

## 4. Spesifikasi Ekspor Excel (Excel Layout)

- **Library Engine**: Go `github.com/xuri/excelize/v2` via `excelize.NewStreamWriter()`.
- **Nama Sheet**: `Shipping_Manifest_{order_number}`.
- Berkas daftar koli untuk batch booking ekspedisi eksternal (No. Koli, Ref SPK, Nama, HP, Alamat, Kota, Kode Pos, Ekspedisi, Berat kg).

---

## 5. Kontrak REST API

### Endpoint: `GET /api/v1/oms/orders/{id}/documents/shipping-label?format={format}&koli={koli_number}`
- **Parameter URL**:
  - `id` (path, required): UUIDv7 Sales Order.
  - `format` (query, required): `pdf` atau `excel`.
  - `koli` (query, optional): Nomor koli spesifik (`1`, `2`, dst.) atau `all` (default mencetak seluruh stiker koli sekaligus).
- **Response**: File PDF multi-halaman jika total koli > 1 (1 halaman = 1 stiker A6) atau spreadsheet manifest.

---

## 6. Kriteria Penerimaan (Acceptance Criteria - Gherkin)

### Skenario 1: Cetak Seluruh Stiker Koli Sekaligus
- **Given** Pesanan `SPK-202610-0012` dikemas menjadi 3 karung koli.
- **When** Staf packing memanggil endpoint cetak stiker dengan parameter `koli=all&format=pdf`.
- **Then** Sistem menghasilkan 1 berkas PDF berisi tepat 3 halaman ukuran A6 berturut-turut: `KOLI: 1 / 3`, `KOLI: 2 / 3`, dan `KOLI: 3 / 3`.

### Skenario 2: Keterbacaan Barcode Scanner
- **Given** Stiker pengiriman dicetak pada printer thermal 100x150mm.
- **When** Barcode discan menggunakan scanner optik gudang.
- **Then** Scanner membaca nomor referensi SPK dan nomor koli secara instan tanpa error.

---

## 7. Kerangka Pemecahan Tugas (Task Breakdown)
- [ ] **Fase 1**: Generator Barcode Code128 / QR Code di Go backend.
- [ ] **Fase 2**: PDF layout builder A6 (100x150mm) thermal page generator.
- [ ] **Fase 3**: Handler route `GET /api/v1/oms/orders/{id}/documents/shipping-label`.
