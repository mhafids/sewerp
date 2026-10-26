# Spesifikasi Sub-Modul: 008b - Laporan Kinerja Stasiun & Defect Rate (Reporting)

Dokumen ini memuat spesifikasi analitik operasional pabrik: durasi pengerjaan pesanan (*Production Lead Time*), beban throughput 6 stasiun kerja, dan tingkat cacat (*Defect Rate*) per kategori kerusakan untuk SewERP.

---

## 1. Metadata & Tautan Referensi Terkait

- **Kode Modul**: `SPEC-008b`
- **Domain Bounded Context**: `analytics` / `reporting`
- **Tautan Standar Acuan Global**:
  - 📋 [Standar Penulisan Requirement](../context/requirements_standard.md)
  - 📋 [Kebutuhan Bisnis Konveksi](../context/business_requirements.md)
  - 🗄️ [Standar Database PostgreSQL](../context/dbadmin.md)
  - 🧪 [Standar QA & Benchmark Testing](../context/qa.md)
- **Tautan Spesifikasi Fitur Terkait**:
  - 🧵 [004a - Papan Pelacakan 6 Stasiun Produksi](./004a-production-tracking.md)
  - 🧵 [004b - Subkontrak Makloon Luar](./004b-makloon-subcontract.md)
  - 🔍 [004d - Inspeksi QC & Potong Tambal](./004d-qc-inspection.md)
  - 📊 [001c - Radar Deadline Kritis](./001c-dashboard-radar-alerts.md)

---

## 2. Ringkasan Kebutuhan & Masalah Bisnis

### Masalah Lapangan
Manajer konveksi tidak memiliki data statistik mengenai rata-rata waktu pengerjaan pesanan (*lead time*) dan di stasiun kerja mana pesanan paling sering tersendat (*bottleneck*). Selain itu, tingkat kerusakan baju tidak terukur, sehingga pabrik tidak tahu apakah kerugian kain disebabkan oleh meja potong, makloon sablon, atau kesalahan penjahit.

### Solusi Fitur
1. **Analisis Waktu Siklus (Lead Time Analytics)**: Menghitung rata-rata hari pengerjaan per model artikel dari status `approved` hingga `ready_to_ship`.
2. **Matriks Tingkat Cacat (Defect Rate Matrix)**: Menghitung persentase reject terhadap total kuantiti potong per kategori cacat (Jahit, Sablon, Kain, Pola).

---

## 3. User Stories

- **US-008b-01 [P1 - Manajer Produksi]**: Sebagai *Manajer Produksi*, saya ingin melihat laporan persentase cacat per kategori kerusakan, agar saya dapat memberikan evaluasi kerja kepada staf jahit atau vendor sablon.
- **US-008b-02 [P1 - Owner Konveksi]**: Sebagai *Owner Konveksi*, saya ingin melihat rata-rata lead time pengerjaan pesanan per bulan, agar janji waktu kirim ke pelanggan di penawaran berikutnya lebih akurat.

---

## 4. Kriteria Penerimaan (Acceptance Criteria - Gherkin)

### Skenario 1: Kalkulasi Defect Rate Bulanan
- **Given** Selama bulan Oktober 2026 dipotong total 5.000 pcs pakaian.
- **And** Tercatat 120 pcs reject di stasiun QC (60 cacat jahit, 40 cacat sablon, 20 cacat kain).
- **When** Laporan kinerja produksi dimuat.
- **Then** Sistem menghitung total defect rate sebesar $\frac{120}{5000} \times 100\% = 2.40\%$ (240 bps) dan menyajikan diagram sebaran cacat.

---

## 5. Kontrak API RESTful

### Endpoint: `GET /api/v1/reporting/production-defects?start_date=2026-10-01&end_date=2026-10-31`
- **Response Payload (HTTP 200 OK)**:
```json
{
  "status": "success",
  "data": {
    "total_cut_pieces": 5000,
    "total_rejected_pieces": 120,
    "overall_defect_rate_bps": 240,
    "average_lead_time_days": 6.8,
    "defect_categories": [
      { "category": "sewing_defect", "pieces": 60, "percentage_bps": 120 },
      { "category": "printing_defect", "pieces": 40, "percentage_bps": 80 },
      { "category": "fabric_defect", "pieces": 20, "percentage_bps": 40 }
    ]
  }
}
```

---

## 6. Kerangka Pemecahan Tugas (Task Breakdown)
- [ ] **Fase 1**: Kueri agregasi waktu transisi stasiun dan kalkulasi reject di sqlc.
- [ ] **Fase 2**: Service laporan kinerja produksi dan persentase basis poin.
- [ ] **Fase 3**: Handler route `GET /api/v1/reporting/production-defects`.
- [ ] **Fase 4**: SolidJS visualisasi diagram batang cacat dengan UnoCSS Flat UI.
