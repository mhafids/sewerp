# Spesifikasi Sub-Modul: 003e - Surat Penagihan (Billing Statement)

Dokumen ini memuat spesifikasi fungsional, struktur data, tata letak cetak PDF, dan skema ekspor Excel untuk **Surat Penagihan Pembayaran / Proforma Invoice (Billing Statement)** sebagai dokumen resmi penagihan pelunasan atau termin kredit pesanan konveksi.

---

## 1. Metadata & Identitas Dokumen

- **Kode Modul**: `SPEC-003e`
- **Modul Induk**: [003 - Sales & Order Management System (OMS)](./003-sales-order.md)
- **Domain Bounded Context**: `oms`
- **Aktor Utama**: *Bagian Keuangan (Finance)*, *Admin Sales*, *Bagian Procurement/Finance Pelanggan*
- **Format Output**:
  - **PDF**: A4 Standar Formal Invoice / Surat Penagihan Resmi
  - **Excel**: Rincian Tagihan B2B beserta perhitungan pajak & termin

---

## 2. Struktur Data & Sumber Bidang (Data Source DTO)

Data ditarik dari entitas `sales_orders`, `order_items`, `order_surcharges`, `customers`, dan `order_payments`:

| Bidang Data | Tipe Data | Keterangan |
| :--- | :--- | :--- |
| `statement_number` | `varchar(32)` | Nomor surat tagihan (Format: `INV-YYYYMM-XXXX`) |
| `order_number` | `varchar(32)` | Nomor SPK rujukan |
| `issue_date` | `date` | Tanggal penerbitan surat tagihan |
| `due_date` | `date` | Batas akhir pembayaran (Jatuh Tempo) |
| `customer_company` | `varchar(100)` | Nama PT/CV/Instansi pemesan |
| `customer_pic` | `varchar(100)` | Nama kontak PIC penanggung jawab |
| `customer_address` | `text` | Alamat kantor penagihan |
| `subtotal_amount` | `bigint` | Nilai pokok pesanan sebelum biaya tambahan |
| `surcharge_total_amount`| `bigint` | Akumulasi biaya tambahan terkonfigurasi (Order/Item/Size) |
| `surcharges_breakdown` | `array` | Rincian nama snapshot, scope, dan total biaya tambahan |
| `contract_amount` | `bigint` | Total nilai kesepakatan SPK (Grand Total) |
| `total_received` | `bigint` | Total dana yang sudah diterima sebelumnya (DP / termin) |
| `outstanding_amount`| `bigint` | Total sisa piutang yang wajib dibayarkan sekarang |
| `payment_instructions`| `array` | Daftar rekening bank resmi konveksi & link QRIS |

---

## 3. Spesifikasi Tata Letak Cetak (PDF Layout)

### Standar Fisik & Margin
- **Ukuran Kertas**: A4 Portrait (210mm x 297mm), margin 15mm.
- **Header Resmi**: Kop surat resmi konveksi (Logo, Nama Legal Badan Usaha, NPWP, Alamat, No. Telp, Email).
- **Metadata Penagihan**:
  - Kotak Tagihan Kepada: Nama Instansi, Alamat, Up: Nama PIC.
  - Kotak Info Faktur: No. Tagihan, No. SPK Rujukan, Tanggal Terbit, **Tanggal Jatuh Tempo (Bold Merah)**.
- **Tabel Ringkasan Nilai & Pembayaran**:
  - Baris 1: Subtotal Pokok Barang (`subtotal_amount`).
  - Baris 2: Rincian Biaya Tambahan / Surcharges (Rush Order, Treatment, Ukuran Jumbo).
  - Baris 3: Total Nilai Kontrak SPK (`contract_amount`).
  - Baris 4: Akumulasi Pembayaran Masuk (`total_received`).
  - Baris 5 (Highlight Latar Abu-abu / Kotak Tebal): **JUMLAH TAGIHAN YANG HARUS DIBAYAR (`outstanding_amount`)**.
- **Instruksi Kanal Pembayaran**:
  - Rekening Bank Resmi (BCA / Mandiri / BNI dengan Nomor Rekening dan Atas Nama Perusahaan Konveksi).
  - Tautan Pembayaran / QR Code dinamis Xendit.
- **Klausul Kebijakan**:
  - Peringatan serah terima: *"Barang pesanan yang telah selesai hanya dapat dikirim setelah pelunasan diverifikasi atau disetujui perjanjian kredit resmi"*.
- **Footer**: Tanda Tangan Manajer Keuangan & Stempel Resmi Perusahaan.

---

## 4. Spesifikasi Ekspor Excel (Excel Layout)

- **Library Engine**: Go `github.com/xuri/excelize/v2` via `excelize.NewStreamWriter()`.
- **Nama Sheet**: `Tagihan_{order_number}`.
- **Struktur Baris & Kolom**:
  - `A1:F2`: Kop Surat Konveksi & Info Faktur Tagihan.
  - `A4:B7`: Identitas Pelanggan & Jatuh Tempo.
  - `Row 9`: Header Rincian Pekerjaan (`No`, `Nama Artikel`, `Bahan/Warna`, `Qty (Pcs)`, `Tarif Satuan`, `Total`).
  - `Row 10...N`: Baris data artikel pesanan.
  - `Row N+2 s/d N+4`: Subtotal, Pembayaran Terdahulu (DP), dan Sisa Tagihan Akhir via formula `=C(N+2)-C(N+3)`.
  - Format Angka: Currency standard `Rp #,##0`.

---

## 5. Kontrak REST API

### Endpoint: Unduh / Cetak Surat Penagihan
```http
GET /api/v1/oms/orders/{id}/documents/billing-statement?format={format}
```

#### Parameter URL
- `id` (path, required): UUIDv7 Sales Order.
- `format` (query, required): `pdf` atau `excel`.

#### Response
- Binary stream file PDF (`application/pdf`) atau Excel (`application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`).

---

## 6. Kriteria Penerimaan (Acceptance Criteria - Gherkin)

### Skenario 1: Penerbitan Tagihan Pelunasan Sebelum Pengiriman
- **Given** SPK `SPK-202610-0012` berstatus `ready_to_ship` dan sisa piutang Rp 7.350.000.
- **When** Finance menerbitkan Surat Penagihan dalam format PDF.
- **Then** Dokumen menampilkan nominal jatuh tempo persis Rp 7.350.000 lengkap dengan instruksi nomor rekening bank resmi.

### Skenario 2: Pencegahan Penagihan Pada SPK Lunas
- **Given** SPK telah lunas 100% (`outstanding_amount = 0`).
- **When** Pengguna mencoba mengakses endpoint tagihan.
- **Then** Sistem mengembalikan respons informatif bahwa pesanan sudah lunas dan mengarahkan ke dokumen Nota Lunas / Kuitansi.
