# Spesifikasi Sub-Modul: 003c - Log Pembayaran (Payment Log)

Dokumen ini memuat spesifikasi fungsional, struktur data, tata letak cetak PDF, dan skema ekspor Excel untuk **Buku Log Riwayat Pembayaran (Payment Log)** per pesanan SPK sebagai dokumen pertanggungjawaban audit keuangan konveksi.

---

## 1. Metadata & Identitas Dokumen

- **Kode Modul**: `SPEC-003c`
- **Modul Induk**: [003 - Sales & Order Management System (OMS)](./003-sales-order.md)
- **Domain Bounded Context**: `oms`
- **Aktor Utama**: *Bagian Keuangan (Finance)*, *Kasir*, *Auditor Internal*, *Owner*
- **Format Output**:
  - **PDF**: A4 Standar Audit Sheet (Laporan rekapitulasi pembayaran resmi)
  - **Excel**: Data grid mutasi rekening lengkap untuk rekonsiliasi perbankan

---

## 2. Struktur Data & Sumber Bidang (Data Source DTO)

Data ditarik dari relasi `sales_orders`, `order_payments`, dan `payment_gateway_logs`:

| Bidang Data | Tipe Data | Keterangan |
| :--- | :--- | :--- |
| `order_number` | `varchar(32)` | Nomor SPK rujukan |
| `customer_name` | `varchar(100)` | Nama pemesan |
| `grand_total` | `bigint` | Total kontrak nilai SPK |
| `total_paid` | `bigint` | Akumulasi uang yang telah berhasil diverifikasi |
| `unpaid_amount` | `bigint` | Sisa piutang tertunggak |
| `payments[].id` | `uuid` | ID unik transaksi pembayaran |
| `payments[].payment_date` | `date` | Tanggal transaksi dibayarkan |
| `payments[].payment_type` | `varchar(20)` | `down_payment`, `installment`, `settlement` |
| `payments[].method_category`| `varchar(20)` | `xendit`, `manual_transfer`, `cash` |
| `payments[].bank_code` | `varchar(20)` | Kode bank (BCA, Mandiri, BRI, BSI, QRIS) |
| `payments[].reference_number`| `varchar(64)` | No. Referensi Mutasi Bank / ID Virtual Account |
| `payments[].amount` | `bigint` | Nominal dana masuk |
| `payments[].status` | `varchar(20)` | `PAID`, `PENDING`, `REJECTED` |
| `payments[].verified_by` | `varchar(100)` | Petugas finance yang memvalidasi transfer manual |

---

## 3. Spesifikasi Tata Letak Cetak (PDF Layout)

### Standar Fisik & Margin
- **Ukuran Kertas**: A4 Portrait (210mm x 297mm), margin 15mm.
- **Header**: Kop Usaha Konveksi, Judul Dokumen **BUKU RIWAYAT PEMBAYARAN & AUDIT LOG**, Nomor SPK, Tanggal Cetak Laporan.
- **Ringkasan Finansial (Summary Cards)**:
  - Kotak 1: Total Nilai SPK (`grand_total`).
  - Kotak 2: Total Uang Masuk Diverifikasi (`total_paid`).
  - Kotak 3: Sisa Piutang (`unpaid_amount`).
  - Kotak 4: Status Akhir (`LUNAS` / `SEBAGIAN`).
- **Tabel Kronologi Transaksi**:
  - Kolom: `No` | `Tanggal & Jam` | `Tipe Pembayaran` | `Metode & Bank` | `No. Referensi / VA` | `Petugas Verifikasi` | `Jumlah Masuk (Rp)`.
  - Baris Rekapitulasi: Total Akumulasi Dana Masuk.
- **Footer**: Kolom tanda tangan Bagian Keuangan / Kasir & Manajer Operasional.

---

## 4. Spesifikasi Ekspor Excel (Excel Layout)

- **Library Engine**: Go `github.com/xuri/excelize/v2` via `excelize.NewStreamWriter()`.
- **Nama Sheet**: `Payment_Log_{order_number}`.
- **Struktur Baris & Kolom**:
  - `A1:G2`: Judul Laporan Rekonsiliasi Pembayaran SPK.
  - `A4:C6`: Metadata Pelanggan & Ringkasan Nilai Pesanan.
  - `Row 8`: Baris Header Kolom (`No`, `Tanggal Bayar`, `Tipe`, `Metode/Bank`, `No. Referensi`, `Status`, `Nominal (Rp)`).
  - `Row 9...N`: Baris data transaksi pembayaran.
  - `Row N+1`: Total Bayar dengan formula `=SUM(G9:GN)`.
  - Format Angka: Accounting currency format `_([$Rp-id-ID]* #,##0_);_([$Rp-id-ID]* (#,##0);_([$Rp-id-ID]* "-"_);_(@_)`.

---

## 5. Kontrak REST API

### Endpoint: Unduh / Cetak Log Pembayaran
```http
GET /api/v1/oms/orders/{id}/documents/payment-log?format={format}
```

#### Parameter URL
- `id` (path, required): UUIDv7 Sales Order.
- `format` (query, required): `pdf` atau `excel`.

#### Response
- Binary stream file PDF (`application/pdf`) atau Excel (`application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`).

---

## 6. Kriteria Penerimaan (Acceptance Criteria - Gherkin)

### Skenario 1: Rekap Pembayaran Bertahap
- **Given** SPK memiliki 3 tahapan pembayaran (DP 50%, Pembayaran Bahan 30%, Pelunasan 20%).
- **When** Dokumen payment log diekspor dalam format PDF atau Excel.
- **Then** Ketiga baris transaksi muncul berurutan secara kronologis beserta nomor referensi bank masing-masing.

### Skenario 2: Pencegahan Transaksi Gantung / Belum Diverifikasi
- **Given** Terdapat pembayaran manual yang masih berstatus `PENDING`.
- **When** Dokumen payment log dicetak.
- **Then** Nominal pembayaran `PENDING` tidak dihitung ke dalam baris `total_paid` dan diberi tanda status jelas "Menunggu Verifikasi".
