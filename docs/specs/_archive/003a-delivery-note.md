# Spesifikasi Sub-Modul: 003a - Surat Jalan (Delivery Note)

Dokumen ini memuat spesifikasi fungsional, struktur data, tata letak cetak PDF, dan skema ekspor Excel untuk **Surat Jalan (Delivery Note)** sebagai bukti serah terima fisik barang pesanan dari konveksi ke pihak ekspedisi atau pelanggan.

---

## 1. Metadata & Identitas Dokumen

- **Kode Modul**: `SPEC-003a`
- **Modul Induk**: [003 - Sales & Order Management System (OMS)](./003-sales-order.md)
- **Domain Bounded Context**: `oms`
- **Aktor Utama**: *Admin Gudang*, *Staf Ekspedisi / Kurir*, *Pelanggan*
- **Format Output**:
  - **PDF**: A4 / Folio (3 Rangkap: Putih untuk Pelanggan, Merah untuk Gudang, Kuning untuk Ekspedisi/Sopir)
  - **Excel**: Matriks *Packing List* rincian koli/karung per artikel & sebaran ukuran

---

## 2. Struktur Data & Sumber Bidang (Data Source DTO)

Data ditarik dari entitas `sales_orders`, `order_items`, `order_item_sizes`, dan `delivery_notes`:

| Bidang Data | Tipe Data | Keterangan |
| :--- | :--- | :--- |
| `delivery_number` | `varchar(32)` | Nomor unik Surat Jalan (Format: `SJ-YYYYMM-XXXX`) |
| `order_number` | `varchar(32)` | Nomor SPK / Sales Order rujukan |
| `delivery_date` | `date` | Tanggal pengiriman fisik barang |
| `recipient_name` | `varchar(100)` | Nama penerima / PIC di lokasi tujuan |
| `shipping_address` | `text` | Alamat lengkap pengiriman & kontak telepon |
| `courier_type` | `varchar(50)` | Kurir internal / ekspedisi eksternal (JNE Trucking, Indah Cargo, dll.) |
| `tracking_number` | `varchar(64)` | Nomor resi ekspedisi (opsional jika kurir internal) |
| `total_packages` | `int` | Jumlah total koli/karung/karton |
| `items` | `array` | Rincian artikel, warna kain, total pcs, dan rincian koli |

---

## 3. Spesifikasi Tata Letak Cetak (PDF Layout)

### Standar Fisik & Margin
- **Ukuran Kertas**: A4 (210mm x 297mm) atau Folio / F4 (215mm x 330mm).
- **Margin**: Top 15mm, Bottom 15mm, Left 15mm, Right 15mm.
- **Warna & Kontras**: Monokrom / Grayscale tajam (`#1e293b`), siap cetak dot-matrix / inkjet / laserjet.

### Blok Susunan Dokumen
1. **Header**:
   - Kiri: Logo Konveksi, Nama Usaha, Alamat Workshop, No. Telepon / WhatsApp.
   - Kanan: Judul besar **SURAT JALAN (DELIVERY NOTE)**, Nomor Dokumen, Tanggal Kirim, Nomor SPK Rujukan.
2. **Identitas Pengiriman**:
   - Kolom Kiri: Tujuan Pengiriman (Nama Pelanggan, Alamat Tujuan, No. Telepon Penerima).
   - Kolom Kanan: Ekspedisi Pengiriman, Nomor Resi / Plat Nomor Kendaraan, Total Koli.
3. **Tabel Rincian Barang**:
   - Kolom: `No` | `Nama Artikel / Seragam` | `Warna Kain` | `Rincian Ukuran (S/M/L/XL/XXL)` | `Total Pcs` | `No. Koli/Karung`.
   - Baris Total: Total keseluruhan pcs pakaian dan total koli.
4. **Catatan Tambahan**:
   - Instruksi pengecekan fisik barang, batas komplain maksimal (misal: 2x24 jam sejak diterima).
5. **Footer Tanda Tangan (3 Kolom Sejajar)**:
   - Kolom 1: Penerima / Pelanggan (Nama Terang, Tanda Tangan, Tanggal Terima).
   - Kolom 2: Sopir / Pihak Ekspedisi (Nama Terang & Tanda Tangan).
   - Kolom 3: Hormat Kami / Bagian Gudang Konveksi (Nama Terang & Stempel).

---

## 4. Spesifikasi Ekspor Excel (Excel Layout)

- **Library Engine**: Go `github.com/xuri/excelize/v2` via `excelize.NewStreamWriter()`.
- **Nama Sheet**: `Surat_Jalan_{order_number}`.
- **Struktur Baris & Kolom**:
  - `A1:F2`: Kop Surat Konveksi & Metadata Pengiriman.
  - `A4:B7`: Informasi Penerima & Ekspedisi.
  - `Row 9`: Baris Header Kolom (`No`, `Nama Artikel`, `Warna`, `Size Breakdown`, `Qty (Pcs)`, `Koli Ke-`).
  - `Row 10...N`: Data baris pengiriman dengan auto-fit column width.
  - `Row N+1`: Baris Total menggunakan formula `=SUM(E10:EN)`.
  - Format Kolom Angka: Number format integer `#,##0` (tanpa desimal).

---

## 5. Kontrak REST API

### Endpoint: Unduh / Cetak Surat Jalan
```http
GET /api/v1/oms/orders/{id}/documents/delivery-note?format={format}
```

#### Parameter URL
- `id` (path, required): UUIDv7 Sales Order.
- `format` (query, required): `pdf` atau `excel`.

#### Response
- `format=pdf`:
  - `Content-Type`: `application/pdf`
  - `Content-Disposition`: `inline; filename="SJ-{order_number}.pdf"`
- `format=excel`:
  - `Content-Type`: `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`
  - `Content-Disposition`: `attachment; filename="SJ-{order_number}.xlsx"`

---

## 6. Kriteria Penerimaan (Acceptance Criteria - Gherkin)

### Skenario 1: Ekspor Surat Jalan ke PDF Berhasil
- **Given** Pesanan `SPK-202610-0012` telah selesai produksi dan memiliki data koli pengiriman.
- **When** Pengguna melakukan request `GET /api/v1/oms/orders/{id}/documents/delivery-note?format=pdf`.
- **Then** Sistem mengembalikan status HTTP 200 dengan payload binary PDF valid.
- **And** Ukuran file PDF kurang dari 300 KB serta memuat nomor SPK, alamat tujuan, dan kolom tanda tangan 3 pihak.

### Skenario 2: Validasi Format Ekspor Excel
- **Given** Pengguna meminta ekspor Surat Jalan dengan `format=excel`.
- **When** Sistem menghasilkan berkas `.xlsx`.
- **Then** Berkas dapat dibuka di Microsoft Excel / Google Sheets tanpa corrupt warning dan memuat formula sum total pcs.
