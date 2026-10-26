# Spesifikasi Sub-Modul: 008a - Laporan Penjualan & Piutang Pelanggan (Reporting)

Dokumen ini memuat spesifikasi agregasi analitik omset kotor, total diskon, kas masuk, dan sisa piutang tertunggak (*Accounts Receivable*) per pelanggan dengan filter rentang tanggal fleksibel serta ekspor CSV/Excel untuk SewERP.

---

## 1. Metadata & Tautan Referensi Terkait

- **Kode Modul**: `SPEC-008a`
- **Domain Bounded Context**: `analytics` / `reporting`
- **Tautan Standar Acuan Global**:
  - 📋 [Standar Penulisan Requirement](../context/requirements_standard.md)
  - 🏛️ [Arsitektur Sistem & Presisi Finansial Zero Float](../context/architecture.md)
  - ⚙️ [Standar Rekayasa Backend Go & Zero Alloc](../context/backend.md)
  - 🗄️ [Standar Database PostgreSQL & Indexing](../context/dbadmin.md)
  - 🧪 [Standar QA & Benchmark Testing](../context/qa.md)
  - 📋 [Kebutuhan Bisnis Konveksi](../context/business_requirements.md)
- **Tautan Spesifikasi Fitur Terkait**:
  - 📊 [001a - Dashboard Metrik Agregasi](./001a-dashboard-overview-metrics.md)
  - 📄 [003a - Inti Sales & Order (OMS)](./003a-sales-order-core.md)
  - 📑 [003f - Cetak Surat Penagihan Pelunasan](./003f-billing-statement.md)
  - 👥 [002a - Manajemen Master Pelanggan](./002a-customer-management.md)

---

## 2. Ringkasan Kebutuhan & Masalah Bisnis

### Masalah Lapangan
Owner dan staf keuangan kesulitan merekapitulasi total uang muka yang sudah masuk vs sisa piutang yang tertahan di pelanggan pada penutupan buku akhir bulan, sehingga perencanaan arus kas untuk belanja kain dan gaji sering defisit.

### Solusi Fitur
Laporan Keuangan & Piutang Berjalan:
1. Agregasi total nilai transaksi, kas diterima (DP + Pelunasan), dan sisa piutang per pelanggan.
2. Filter rentang tanggal fleksibel (`start_date` s/d `end_date`), filter status order, dan pengelompokan per pelanggan.
3. Ekspor instan satu-klik ke file **CSV** dan **Excel (.xlsx)** dalam waktu $< 500\text{ ms}$.

---

## 3. User Stories

- **US-008a-01 [P1 - Owner Konveksi]**: Sebagai *Owner Konveksi*, saya ingin melihat total omset dan daftar pelanggan yang masih memiliki sisa piutang dalam rentang tanggal tertentu, agar saya dapat menginstruksikan penagihan secara terarah.
- **US-008a-02 [P1 - Kasir / Finance]**: Sebagai *Finance*, saya ingin mengekspor laporan penjualan ke berkas Excel, agar dapat diserahkan ke konsultan akuntansi perusahaan.

---

## 4. Kriteria Penerimaan (Acceptance Criteria - Gherkin)

### Skenario 1: Menghasilkan Laporan Penjualan dengan Filter Tanggal
- **Given** Terdapat 25 transaksi pesanan pada bulan Oktober 2026.
- **When** User memilih filter `2026-10-01` s/d `2026-10-31`.
- **Then** Sistem menghitung total omset kotor, total diskon, total kas masuk, dan sisa piutang menggunakan integer `BIGINT`.
- **And** Waktu respons kueri selesai dalam tempo $< 100\text{ ms}$.

### Skenario 2: Ekspor Laporan ke Excel
- **Given** Hasil laporan penjualan ditampilkan di layar.
- **When** User menekan tombol "Ekspor Excel".
- **Then** Sistem menghasilkan berkas `.xlsx` dengan format mata uang accounting `Rp #,##0` dan formula total baris bawah.

---

## 5. Kontrak API RESTful

### Endpoint: `GET /api/v1/reporting/sales?start_date=2026-10-01&end_date=2026-10-31&format=json`
- **Response Payload (HTTP 200 OK)**:
```json
{
  "status": "success",
  "data": {
    "summary": {
      "total_orders": 25,
      "total_revenue_idr": 185000000,
      "total_paid_idr": 136800000,
      "total_unpaid_receivables_idr": 48200000
    },
    "customer_breakdown": [
      {
        "customer_name": "Komunitas Sepeda Bandung",
        "order_count": 2,
        "total_billed_idr": 18500000,
        "total_paid_idr": 10000000,
        "unpaid_idr": 8500000
      }
    ]
  }
}
```

---

## 6. Kerangka Pemecahan Tugas (Task Breakdown)
- [ ] **Fase 1**: Kueri SQL agregasi penjualan di `backend/sqlc/reporting_sales.sql`.
- [ ] **Fase 2**: Service laporan penjualan dengan zero memory allocation.
- [ ] **Fase 3**: Handler route `GET /api/v1/reporting/sales` (JSON/Excel/CSV).
- [ ] **Fase 4**: Antarmuka tabel laporan penjualan dan filter tanggal di SolidJS.
