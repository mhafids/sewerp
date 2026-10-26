# Spesifikasi Sub-Modul: 003d - Cetak Log Pembayaran (Payment Log)

Dokumen ini memuat spesifikasi fungsional, struktur data, tata letak cetak PDF, dan skema ekspor Excel untuk **Buku Log Riwayat Pembayaran (Payment Log)** per pesanan SPK sebagai dokumen pertanggungjawaban audit keuangan konveksi.

---

## 1. Metadata & Tautan Referensi Terkait

- **Kode Modul**: `SPEC-003d`
- **Domain Bounded Context**: `oms`
- **Tautan Standar Acuan Global**:
  - 📋 [Standar Penulisan Requirement](../context/requirements_standard.md)
  - 🏛️ [Arsitektur Sistem & Presisi Finansial Zero Float](../context/architecture.md)
  - 🛡️ [Standar Keamanan & Audit Trail](../context/security.md)
  - 🗄️ [Standar Database PostgreSQL](../context/dbadmin.md)
  - 🧪 [Standar QA & Benchmark Testing](../context/qa.md)
- **Tautan Spesifikasi Fitur Terkait**:
  - 📄 [003a - Inti Sales & Order (OMS)](./003a-sales-order-core.md)
  - 🧾 [003c - Cetak Nota Penjualan Kasir](./003c-sales-receipt.md)
  - 📑 [003f - Cetak Surat Penagihan Pelunasan](./003f-billing-statement.md)
  - 💳 [009b - Gateway Pembayaran Xendit](./009b-xendit-gateway.md)
  - 🏦 [009d - Pembayaran Manual & Kasir](./009d-manual-cash-payment.md)

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
- Data grid mutasi rekening dengan formula auto-sum dan pemformatan accounting currency tanpa pembulatan desimal.

---

## 5. Kontrak REST API

### Endpoint: `GET /api/v1/oms/orders/{id}/documents/payment-log?format={format}`
- **Parameter URL**:
  - `id` (path, required): UUIDv7 Sales Order.
  - `format` (query, required): `pdf` atau `excel`.
- **Response**: Binary stream file PDF (`application/pdf`) atau Excel (`application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`).

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

---

## 7. Kerangka Pemecahan Tugas (Task Breakdown)
- [ ] **Fase 1**: Kueri audit trail pembayaran di `backend/sqlc/payments.sql`.
- [ ] **Fase 2**: Service penyusun DTO `PaymentLogExport`.
- [ ] **Fase 3**: Renderer PDF & Excel generator.
- [ ] **Fase 4**: Handler route `GET /api/v1/oms/orders/{id}/documents/payment-log`.
