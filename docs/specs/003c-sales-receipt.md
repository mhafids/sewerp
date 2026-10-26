# Spesifikasi Sub-Modul: 003c - Cetak Nota Penjualan Kasir (Sales Receipt)

Dokumen ini memuat spesifikasi fungsional, struktur data, tata letak cetak PDF (A5 Landscape & Thermal 80mm), dan skema ekspor Excel untuk **Nota Penjualan / Bukti Transaksi Kasir (Sales Receipt / Cash Bill)** sebagai tanda bukti pembayaran sah bagi pelanggan.

---

## 1. Metadata & Tautan Referensi Terkait

- **Kode Modul**: `SPEC-003c`
- **Domain Bounded Context**: `oms`
- **Tautan Standar Acuan Global**:
  - 📋 [Standar Penulisan Requirement](../context/requirements_standard.md)
  - 🔧 [Standar Kebutuhan Teknis & Cetak Fisik](../context/technical_requirements.md)
  - 🏛️ [Arsitektur Sistem & Presisi Finansial Zero Float](../context/architecture.md)
  - 🗄️ [Standar Database PostgreSQL](../context/dbadmin.md)
  - 🧪 [Standar QA & Benchmark Testing](../context/qa.md)
- **Tautan Spesifikasi Fitur Terkait**:
  - 📄 [003a - Inti Sales & Order (OMS)](./003a-sales-order-core.md)
  - 🚚 [003b - Cetak Surat Jalan](./003b-delivery-note.md)
  - 💳 [003d - Cetak Log Pembayaran Transaksi](./003d-payment-log.md)
  - 💰 [003g - Master & Order Multi-Level Surcharges](./003g-sales-order-surcharges.md)

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
- Susunan baris dan kolom terformat dengan formatting mata uang integer `Rp #,##0`.

---

## 5. Kontrak REST API

### Endpoint: `GET /api/v1/oms/orders/{id}/documents/nota?format={format}&paper={paper_size}`
- **Parameter URL**:
  - `id` (path, required): UUIDv7 Sales Order.
  - `format` (query, required): `pdf` atau `excel`.
  - `paper` (query, optional): `a5` (default) atau `thermal80` (hanya berlaku jika `format=pdf`).
- **Response Headers**:
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

---

## 7. Kerangka Pemecahan Tugas (Task Breakdown)
- [ ] **Fase 1**: Renderer PDF A5 Landscape dan Thermal 80mm roll.
- [ ] **Fase 2**: Renderer Excel ringkasan transaksi kasir.
- [ ] **Fase 3**: Handler route `GET /api/v1/oms/orders/{id}/documents/nota`.
- [ ] **Fase 4**: Dialog modal pratinjau cetak di SolidJS kasir.
