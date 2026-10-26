# Spesifikasi Sub-Modul: 008d - Laporan Rekapitulasi Upah Borongan & Kasbon (Reporting)

Dokumen ini memuat spesifikasi laporan komprehensif penggajian tenaga kerja: rekapitulasi output pcs per penjahit, total upah kotor, pemotongan pinjaman kasbon, total upah bersih yang dibayarkan, serta ekspor format Excel/CSV resmi untuk pembukuan kas SewERP.

---

## 1. Metadata & Tautan Referensi Terkait

- **Kode Modul**: `SPEC-008d`
- **Domain Bounded Context**: `analytics` / `reporting`
- **Tautan Standar Acuan Global**:
  - 📋 [Standar Penulisan Requirement](../context/requirements_standard.md)
  - 🏛️ [Arsitektur Sistem & Presisi Finansial Zero Float](../context/architecture.md)
  - 🗄️ [Standar Database PostgreSQL](../context/dbadmin.md)
  - 🛡️ [Standar Keamanan Privasi Finansial](../context/security.md)
  - 🧪 [Standar QA & Benchmark Testing](../context/qa.md)
- **Tautan Spesifikasi Fitur Terkait**:
  - 👥 [006a - Roster Tenaga Kerja & Master Tarif](./006a-worker-roster-tariffs.md)
  - 🧵 [006b - Input Harian Output Jahit](./006b-piece-rate-logging.md)
  - 💵 [006c - Pengelolaan Kasbon Penjahit](./006c-cash-advance.md)
  - 🧾 [006d - Rekap Gaji Mingguan & Slip](./006d-payroll-slip.md)

---

## 2. Ringkasan Kebutuhan & Masalah Bisnis

### Masalah Lapangan
Setiap penutupan buku bulanan, staf keuangan membutuhkan rekonsiliasi total pengeluaran biaya tenaga kerja langsung (*Direct Labor Cost*) per minggu, akumulasi pinjaman kasbon yang belum terpotong, serta arsip data slip gaji untuk laporan laba-rugi (*Income Statement*).

### Solusi Fitur
Laporan Rekapitulasi Penggajian & Kasbon:
1. Agregasi total upah kotor, total potongan kasbon, dan upah bersih dibayarkan per periode minggu atau rentang bulan.
2. Filter berdasarkan nama pekerja, kategori divisi (Potong, Sablon, Jahit, QC), dan status pelunasan slip.
3. Ekspor instan satu-klik ke file **CSV** dan **Excel (.xlsx)**.

---

## 3. User Stories

- **US-008d-01 [P1 - Kasir / Finance]**: Sebagai *Kasir/Finance*, saya ingin mengekspor rekapitulasi upah mingguan seluruh pekerja ke file Excel, agar dapat saya arsipkan untuk pembukuan kas bulanan.
- **US-008d-02 [P1 - Owner Konveksi]**: Sebagai *Owner Konveksi*, saya ingin melihat total pengeluaran upah borongan dan sisa kasbon seluruh penjahit dalam bulan berjalan, agar saya mengetahui beban liabilitas tenaga kerja pabrik.

---

## 4. Kriteria Penerimaan (Acceptance Criteria - Gherkin)

### Skenario 1: Ekspor Rekapitulasi Upah ke Berkas CSV
- **Given** Periode penggajian mingguan `2026-W42` telah ditutup.
- **When** Kasir menekan tombol "Ekspor CSV".
- **Then** Sistem menghasilkan berkas CSV dalam tempo $< 500\text{ ms}$ yang memuat kolom: ID Pekerja, Nama Penjahit, Total Pcs Jahit, Upah Kotor, Potongan Kasbon, dan Upah Bersih.

---

## 5. Kontrak API RESTful

### Endpoint: `GET /api/v1/reporting/payroll?start_date=2026-10-01&end_date=2026-10-31&format=json`
- **Response Payload (HTTP 200 OK)**:
```json
{
  "status": "success",
  "data": {
    "summary": {
      "total_workers_paid": 18,
      "total_pieces_produced": 4850,
      "total_gross_wages_idr": 48500000,
      "total_cash_advances_deducted_idr": 6200000,
      "total_net_wages_paid_idr": 42300000
    },
    "workers": [
      {
        "worker_name": "Ahmad Subarjo",
        "role_category": "sewing",
        "total_pcs": 350,
        "gross_wage_idr": 4200000,
        "advance_deduction_idr": 500000,
        "net_wage_idr": 3700000
      }
    ]
  }
}
```

---

## 6. Kerangka Pemecahan Tugas (Task Breakdown)
- [ ] **Fase 1**: Kueri SQL agregasi payroll periods dan cash advances di sqlc.
- [ ] **Fase 2**: Service laporan penggajian dan streaming Excel/CSV.
- [ ] **Fase 3**: Handler route `GET /api/v1/reporting/payroll`.
- [ ] **Fase 4**: Antarmuka tabel laporan penggajian di SolidJS.
