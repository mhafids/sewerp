# Spesifikasi Sub-Modul: 003f - Cetak Surat Penagihan Pelunasan (Billing Statement)

Dokumen ini memuat spesifikasi fungsional, struktur data, tata letak cetak PDF, dan skema ekspor Excel untuk **Surat Penagihan Pembayaran / Proforma Invoice (Billing Statement)** sebagai dokumen resmi penagihan pelunasan atau termin kredit pesanan konveksi.

---

## 1. Metadata & Tautan Referensi Terkait

- **Kode Modul**: `SPEC-003f`
- **Domain Bounded Context**: `oms`
- **Tautan Standar Acuan Global**:
  - 📋 [Standar Penulisan Requirement](../context/requirements_standard.md)
  - 🏛️ [Arsitektur Sistem & Presisi Finansial Zero Float](../context/architecture.md)
  - 🔧 [Standar Kebutuhan Teknis & Cetak Fisik](../context/technical_requirements.md)
  - 🗄️ [Standar Database PostgreSQL](../context/dbadmin.md)
  - 🧪 [Standar QA & Benchmark Testing](../context/qa.md)
- **Tautan Spesifikasi Fitur Terkait**:
  - 📄 [003a - Inti Sales & Order (OMS)](./003a-sales-order-core.md)
  - 🚚 [003b - Cetak Surat Jalan](./003b-delivery-note.md)
  - 🧾 [003c - Cetak Nota Penjualan Kasir](./003c-sales-receipt.md)
  - 💳 [003d - Cetak Log Pembayaran Transaksi](./003d-payment-log.md)
  - 💰 [003g - Master & Order Multi-Level Surcharges](./003g-sales-order-surcharges.md)

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
- **Footer**: Tanda Tangan Manajer Keuangan & Stempel Resmi Perusahaan.

---

## 4. Spesifikasi Ekspor Excel (Excel Layout)

- **Library Engine**: Go `github.com/xuri/excelize/v2` via `excelize.NewStreamWriter()`.
- **Nama Sheet**: `Tagihan_{order_number}`.
- Format Excel resmi B2B dengan breakdown detail pekerjaan, subtotal, DP terdahulu, dan sisa tagihan via cell formula.

---

## 5. Kontrak REST API

### Endpoint: `GET /api/v1/oms/orders/{id}/documents/billing-statement?format={format}`
- **Parameter URL**:
  - `id` (path, required): UUIDv7 Sales Order.
  - `format` (query, required): `pdf` atau `excel`.
- **Response**: Binary stream file PDF (`application/pdf`) atau Excel (`application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`).

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

---

## 7. Kerangka Pemecahan Tugas (Task Breakdown)
- [ ] **Fase 1**: Generator PDF formal invoice A4 dengan kop resmi dan QR code instruksi bayar.
- [ ] **Fase 2**: Generator Excel proforma invoice B2B.
- [ ] **Fase 3**: Handler route `GET /api/v1/oms/orders/{id}/documents/billing-statement`.
