# Spesifikasi Sub-Modul: 001c - Dashboard Radar Deadline Kritis & Peringatan Stok

Dokumen ini memuat spesifikasi sistem peringatan dini visual (*Early Warning System*) di Dashboard untuk mendeteksi pesanan yang mendekati tenggat kirim ($< 3$ hari) serta bahan baku kain/aksesoris di bawah batas persediaan aman (*safety stock*).

---

## 1. Metadata & Tautan Referensi Terkait

- **Kode Modul**: `SPEC-001c`
- **Domain Bounded Context**: `analytics` / `dashboard`
- **Tautan Standar Acuan Global**:
  - 📋 [Standar Penulisan Requirement](../context/requirements_standard.md)
  - 🎨 [Standar Desain Antarmuka Flat UI](../context/ui.md)
  - 👆 [Prinsip UX Lapangan Pabrik](../context/ux.md)
  - 📋 [Kebutuhan Bisnis Konveksi](../context/business_requirements.md)
  - 🧪 [Standar QA & Benchmark Testing](../context/qa.md)
- **Tautan Spesifikasi Fitur Terkait**:
  - 📊 [001a - Quick Counts & Metrik Agregasi](./001a-dashboard-overview-metrics.md)
  - 🧵 [004a - Papan Pelacakan 6 Stasiun Produksi](./004a-production-tracking.md)
  - 📦 [005a - Inventaris Bahan Baku Kain & Lot](./005a-fabric-inventory.md)

---

## 2. Ringkasan Kebutuhan & Masalah Bisnis

### Masalah Lapangan
1. Pesanan sering terlambat dikirim (*overdue*) karena mandor jahit tidak mengetahui bahwa suatu SPK memiliki deadline kirim lusa sementara baju masih berupa tumpukan potongan kain.
2. Proses potong kain terhenti mendadak di tengah hari karena stok benang atau kain warna tertentu habis tak terduga.

### Solusi Fitur
Widget Radar Peringatan yang menyorot:
1. **Radar Deadline Kritis**: SPK dengan sisa hari $\le 3$ hari dari `delivery_deadline` yang belum mencapai stasiun `ready_to_ship`.
2. **Peringatan Stok Bahan Menipis**: Kain atau aksesoris dengan `quantity_available < minimum_threshold`.

---

## 3. User Stories

- **US-001c-01 [P1 - Manajer Produksi]**: Sebagai *Manajer Produksi*, saya ingin melihat daftar SPK berstatus deadline kritis (< 3 hari) lengkap dengan posisi stasiun kerjanya saat ini, agar saya dapat menginstruksikan lembur pada stasiun yang macet.
- **US-001c-02 [P1 - Staf Gudang]**: Sebagai *Staf Gudang*, saya ingin melihat badge peringatan kain yang menipis, agar pengadaan kain ke supplier dapat segera dipesan sebelum lantai potong terhenti.

---

## 4. Kriteria Penerimaan (Acceptance Criteria - Gherkin)

### Skenario 1: Menyorot SPK yang Masuk Radar Kritis
- **Given** SPK `PO-202610-0091` memiliki deadline kirim tersisa 2 hari dan masih berada di stasiun `sewing`.
- **When** Dashboard dimuat oleh Manajer Produksi.
- **Then** SPK tersebut muncul di baris teratas radar dengan badge merah `bg-rose-50 text-rose-700 border-rose-300` beranimasi *pulse* halus.

### Skenario 2: Navigasi Cepat Tombol Aksi
- **Given** Terdapat peringatan stok kain Combed 30s Hitam (tersisa 8.5 kg, minimum 20 kg).
- **When** User mengklik tombol "Pesan Kain" pada kartu peringatan.
- **Then** Sistem membuka drawer kontak supplier spesialis kain tersebut (modul KMS 007b).

---

## 5. Kontrak API RESTful

### Endpoint: `GET /api/v1/dashboard/urgent-alerts`
- **Response Payload (HTTP 200 OK)**:
```json
{
  "status": "success",
  "data": {
    "urgent_orders": [
      {
        "order_number": "PO-202610-0091",
        "customer_name": "SMA Nusantara Bandung",
        "days_remaining": 2,
        "current_stage": "sewing",
        "total_pcs": 120,
        "is_overdue": false
      }
    ],
    "low_stocks": [
      {
        "item_id": "01925b6a-8451-7ef1-953e-51c045b0a111",
        "item_name": "Combed 30s Hitam Reaktif",
        "category": "fabric",
        "remaining_qty": 8.5,
        "minimum_threshold": 20.0,
        "unit": "kg"
      }
    ]
  }
}
```

---

## 6. Desain Antarmuka & Flat UI Tokens
- **Bento Card Kolom 1-7**: Tabel Radar Deadline Kritis dengan bar progress stasiun kerja Flat UI.
- **Bento Card Kolom 8-12**: Daftar item stok rendah dengan badge amber/oranye (`bg-amber-50 text-amber-700`).

---

## 7. Kerangka Pemecahan Tugas (Task Breakdown)
- [ ] **Fase 1**: Kueri `SELECT ... WHERE delivery_deadline <= CURRENT_DATE + INTERVAL '3 days'`.
- [ ] **Fase 2**: Domain DTO `UrgentAlertsResponse`.
- [ ] **Fase 3**: Handler `GET /api/v1/dashboard/urgent-alerts`.
- [ ] **Fase 4**: SolidJS Component `UrgentOrdersRadar` & `LowStockWidget`.
