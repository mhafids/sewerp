# Spesifikasi Sub-Modul: 003b - Nota Penjualan (Sales Receipt)

Dokumen ini memuat spesifikasi fungsional, struktur data, tata letak cetak PDF, dan skema ekspor Excel untuk **Nota Penjualan / Bukti Transaksi Kasir (Sales Receipt / Cash Bill)** sebagai tanda bukti pembelian langsung bagi pelanggan.

---

## 1. Metadata & Identitas Dokumen

- **Kode Modul**: `SPEC-003b`
- **Modul Induk**: [003 - Sales & Order Management System (OMS)](./003-sales-order.md)
- **Domain Bounded Context**: `oms`
- **Aktor Utama**: *Kasir*, *Admin Penjualan*, *Pelanggan*
- **Format Output**:
  - **PDF**: A5 Landscape (210mm x 148mm) atau Kertas Struk Thermal 80mm roll
  - **Excel**: Tabel rekapitulasi nota penjualan untuk arsip harian kasir

---

## 2. Struktur Data & Sumber Bidang (Data Source DTO)

Data ditarik dari entitas `sales_orders`, `order_items`, `order_item_sizes`, `order_surcharges`, dan `order_payments`:

| Bidang Data | Tipe Data | Keterangan |
| :--- | :--- | :--- |
| `receipt_number` | `varchar(32)` | Nomor nota (Format: `NOTA-YYYYMM-XXXX`) |
| `order_number` | `varchar(32)` | Nomor SPK rujukan |
| `order_date` | `date` | Tanggal transaksi |
| `customer_name` | `varchar(100)` | Nama pelanggan / instansi |
| `customer_phone` | `varchar(20)` | Nomor kontak telepon / WhatsApp |
| `cashier_name` | `varchar(100)` | Nama staf kasir yang melayani |
| `subtotal_amount` | `bigint` | Total harga kotor sebelum diskon & surcharge |
| `surcharge_total_amount`| `bigint` | Akumulasi seluruh biaya tambahan terkonfigurasi (Order, Item, Size) |
| `surcharges_breakdown` | `array` | Rincian nama snapshot, scope, dan total biaya tambahan |
| `discount_amount` | `bigint` | Potongan diskon promosi/khusus |
| `grand_total` | `bigint` | Total akhir tagihan |
| `total_paid` | `bigint` | Akumulasi uang yang telah dibayar (DP / Pelunasan) |
| `unpaid_balance` | `bigint` | Sisa kekurangan pembayaran |
| `payment_status` | `varchar(20)` | `UNPAID`, `DP_PAID`, `LUNAS` |

---

## 3. Spesifikasi Tata Letak Cetak (PDF Layout)

### Opsi A: Format Standar Kantor (A5 Landscape)
- **Ukuran**: 210mm x 148mm (Margin: 10mm sekeliling).
- **Header**: Logo Konveksi, Alamat Usaha, Judul **NOTA PENJUALAN**, Nomor Nota, Tanggal & Jam.
- **Data Pelanggan**: Nama Pemesan & No. Telepon.
- **Tabel Transaksi**:
  - Kolom: `No` | `Deskripsi Artikel & Warna` | `Qty (Pcs)` | `Harga Satuan` | `Surcharge (Item/Size)` | `Subtotal`.
- **Kotak Total (Kanan Bawah)**: Subtotal Produk, Biaya Tambahan Order (Rush Order/dll.), Diskon, Grand Total, Jumlah Bayar Masuk, Sisa Tagihan.
- **Cap Status**: Watermark stempel dinamis **"LUNAS"** (Warna Hijau) atau **"DP DITERIMA"** (Warna Biru).
- **Footer**: Catatan konveksi ("Barang yang sudah diproduksi sesuai SPK tidak dapat diretur"), Tanda tangan Kasir & Pelanggan.

### Opsi B: Format Thermal Roll (80mm)
- **Lebar Cetak**: 72mm (Kertas 80mm roll).
- Font monospaced, rata tengah untuk header toko, item berbaris vertikal, garis pemisah putus-putus (`--------------------------------`).

---

## 4. Spesifikasi Ekspor Excel (Excel Layout)

- **Library Engine**: Go `github.com/xuri/excelize/v2` via `excelize.NewStreamWriter()`.
- **Nama Sheet**: `Nota_{order_number}`.
- **Susunan**:
  - `A1:E2`: Header Nota & Metadata Toko.
  - `A4:B6`: Data Pemesan & Status Transaksi.
  - `Row 8`: Baris Judul Kolom (`No`, `Nama Artikel`, `Qty`, `Harga Satuan (Rp)`, `Subtotal (Rp)`).
  - `Row 9...N`: Rincian item dengan formatting mata uang integer `Rp #,##0`.
  - `Row N+1 s/d N+4`: Grand Total, Uang Muka Masuk, dan Sisa Tagihan dengan formula aritmatika cell.

---

## 5. Kontrak REST API

### Endpoint: Unduh / Cetak Nota Penjualan
```http
GET /api/v1/oms/orders/{id}/documents/nota?format={format}&paper={paper_size}
```

#### Parameter URL
- `id` (path, required): UUIDv7 Sales Order.
- `format` (query, required): `pdf` atau `excel`.
- `paper` (query, optional): `a5` (default) atau `thermal80` (hanya berlaku jika `format=pdf`).

#### Response Headers
- `format=pdf`: `Content-Type: application/pdf; filename="NOTA-{order_number}.pdf"`
- `format=excel`: `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet; filename="NOTA-{order_number}.xlsx"`

---

## 6. Kriteria Penerimaan (Acceptance Criteria - Gherkin)

### Skenario 1: Cetak Nota Berstatus LUNAS
- **Given** Pesanan `SPK-202610-0012` memiliki sisa piutang Rp 0 (`unpaid_balance = 0`).
- **When** Kasir mencetak nota via endpoint dokumen nota PDF.
- **Then** Tampilan dokumen memuat teks/stempel "LUNAS" secara tegas dan mencantumkan histori pelunasan.

### Skenario 2: Cetak Nota DP Masuk
- **Given** Pesanan baru menerima DP sebesar Rp 5.000.000 dari total Rp 12.000.000.
- **When** Nota dicetak.
- **Then** Dokumen secara akurat menampilkan Sisa Tagihan Rp 7.000.000 dengan status "DP DITERIMA".
