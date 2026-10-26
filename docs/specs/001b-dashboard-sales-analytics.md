# Spesifikasi Sub-Modul: 001b - Dashboard Visualisasi Penjualan & Pipeline

Dokumen ini memuat spesifikasi visualisasi analitik penjualan: Grafik Tren 6–12 Bulan (Dual-Axis Omset Rupiah vs Volume Pcs) dan Grafik Pipeline Siklus Pesanan (*Order Lifecycle Stepped Funnel Bar*).

---

## 1. Metadata & Tautan Referensi Terkait

- **Kode Modul**: `SPEC-001b`
- **Domain Bounded Context**: `analytics` / `dashboard`
- **Tautan Standar Acuan Global**:
  - 📋 [Standar Penulisan Requirement](../context/requirements_standard.md)
  - 🏛️ [Arsitektur Sistem & Presisi Finansial](../context/architecture.md)
  - 🖥️ [Standar Frontend SolidJS Ultra-Strict](../context/frontend.md)
  - 🎨 [Standar Desain Antarmuka Flat UI](../context/ui.md)
  - 👆 [Prinsip UX Lapangan Pabrik](../context/ux.md)
  - 🧪 [Standar QA & Benchmark Testing](../context/qa.md)
- **Tautan Spesifikasi Fitur Terkait**:
  - 📊 [001a - Quick Counts & Metrik Agregasi](./001a-dashboard-overview-metrics.md)
  - 🎛️ [001d - Tata Letak Bento-Grid & RBAC Views](./001d-dashboard-bento-rbac.md)
  - 📄 [003a - Inti Sales & Order (OMS)](./003a-sales-order-core.md)

---

## 2. Ringkasan Kebutuhan & Masalah Bisnis

### Masalah Lapangan
Owner kesulitan membaca tren musiman (apakah omset naik karena volume produksi meningkat atau harga artikel pakaian yang naik). Selain itu, mandor tidak mengetahui titik kemacetan pipeline (apakah pesanan tersendat di antrean DP atau di stasiun jahit).

### Solusi Fitur
1. **Dual-Axis Chart**: Memadukan Bar Chart (Volume Pcs pakaian) dan Line Chart (Omset Rupiah) selama 6–12 bulan terakhir.
2. **Stepped Funnel Pipeline**: Visualisasi pesanan aktif di setiap fase (`Quotation` $\to$ `Menunggu DP` $\to$ `Antrean Potong` $\to$ `Lantai Jahit/WIP` $\to$ `Siap Kirim` $\to$ `Selesai`).

---

## 3. User Stories

- **US-001b-01 [P1 - Owner]**: Sebagai *Owner Konveksi*, saya ingin melihat grafik tren omset bulanan vs volume pcs baju, agar saya mengetahui produktivitas bengkel dan tren artikel pakaian.
- **US-001b-02 [P1 - Admin Sales]**: Sebagai *Admin Sales*, saya ingin melihat batang pipeline siklus pesanan, agar saya segera mem-follow-up pesanan yang menumpuk di status "Menunggu DP".

---

## 4. Kriteria Penerimaan (Acceptance Criteria - Gherkin)

### Skenario 1: Menampilkan Grafik Tren Penjualan 6 Bulan
- **Given** Terdapat data riwayat SPK disetujui 6 bulan terakhir.
- **When** Halaman Dashboard dimuat.
- **Then** Grafik menampilkan 6 titik sumbu X dengan batang volume pcs dan garis hijau Matcha (`#5B7E5F`) untuk omset IDR.
- **And** Hover tooltip menampilkan: Bulan, Omset IDR, Volume Pcs, dan Average Selling Price (ASP).

### Skenario 2: Siklus Pembersihan Memori Komponen Chart
- **Given** Komponen chart di-mount pada SolidJS.
- **When** Pengguna berpindah ke rute halaman lain (`/crm/customers`).
- **Then** Lifecycle hook `onCleanup()` mengeksekusi penghancuran instance chart tanpa meninggalkan detached DOM nodes.

---

## 5. Kontrak API RESTful

### Endpoint: `GET /api/v1/dashboard/sales-trend?months=6`
- **Response Payload (HTTP 200 OK)**:
```json
{
  "status": "success",
  "data": {
    "periods": [
      {
        "month_label": "Mei 2026",
        "total_revenue_idr": 112000000,
        "total_pcs_produced": 2100,
        "average_selling_price_idr": 53333
      },
      {
        "month_label": "Okt 2026",
        "total_revenue_idr": 142500000,
        "total_pcs_produced": 2450,
        "average_selling_price_idr": 58163
      }
    ]
  }
}
```

---

## 6. Desain Antarmuka & Ergonomi UI
- **Library**: Unovis / Canvas native lightweight tanpa Virtual DOM overhead.
- **Clean Flat Style**: Tanpa gradient bayangan pekat, garis grid abu-abu tipis `1px solid #F3F4F6`.

---

## 7. Kerangka Pemecahan Tugas (Task Breakdown)
- [ ] **Fase 1**: Kueri `DATE_TRUNC('month', order_date)` di sqlc.
- [ ] **Fase 2**: Service kalkulasi agregasi bulanan dan ASP.
- [ ] **Fase 3**: HTTP Endpoint `GET /api/v1/dashboard/sales-trend`.
- [ ] **Fase 4**: Komponen SolidJS `SalesTrendChart` dengan `onCleanup()`.
- [ ] **Fase 5**: Uji kebocoran memori browser di DevTools MCP.
