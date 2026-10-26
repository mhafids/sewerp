# Spesifikasi Sub-Modul: 008c - Laporan Pemakaian Bahan Baku & Efisiensi Kain (Yield Report)

Dokumen ini memuat spesifikasi analitik efisiensi pemakaian bahan baku kain: perbandingan kebutuhan kain teoritis vs pemakaian riil, sisa rol yang dikembalikan (*roll return*), dan persentase limbah perca (*fabric yield & waste*) untuk SewERP.

---

## 1. Metadata & Tautan Referensi Terkait

- **Kode Modul**: `SPEC-008c`
- **Domain Bounded Context**: `analytics` / `reporting`
- **Tautan Standar Acuan Global**:
  - 📋 [Standar Penulisan Requirement](../context/requirements_standard.md)
  - 📋 [Kebutuhan Bisnis Konveksi & Efisiensi Bahan](../context/business_requirements.md)
  - 🗄️ [Standar Database PostgreSQL](../context/dbadmin.md)
  - 🧪 [Standar QA & Benchmark Testing](../context/qa.md)
- **Tautan Spesifikasi Fitur Terkait**:
  - 📦 [005a - Inventaris Bahan Baku Kain & Lot](./005a-fabric-inventory.md)
  - 📦 [005b - Alokasi Bahan Baku & Toleransi Susut](./005b-stock-allocation-wastage.md)
  - 📦 [005c - Pengembalian Sisa Rol Kain](./005c-roll-return.md)
  - 🧵 [004a - Papan Pelacakan 6 Stasiun Produksi](./004a-production-tracking.md)

---

## 2. Ringkasan Kebutuhan & Masalah Bisnis

### Masalah Lapangan
Kain merupakan komponen biaya terbesar dalam produksi konveksi (mencapai 60%–70% dari HPP pakaian). Owner sering mengalami kebutaan biaya kain: tidak tahu apakah margin susut 3%–5% yang ditetapkan sudah realistis atau justru terjadi pemborosan kain akibat tata letak pola potong yang tidak efisien (*marker layout waste*).

### Solusi Fitur
Laporan Efisiensi Bahan Baku per SPK:
1. Menghitung rasio yield pemakaian kain:
   $$\text{Fabric Yield (\%)} = \frac{\text{Berat Bersih Pakaian Jadi}}{\text{Berat Kain Riil Terpakai}} \times 100\%$$
2. Menghitung persentase limbah perca (*waste percentage*) aktual per SPK dan per tukang potong.

---

## 3. User Stories

- **US-008c-01 [P1 - Owner Konveksi]**: Sebagai *Owner Konveksi*, saya ingin melihat laporan efisiensi pemakaian kain per SPK, agar saya mengetahui apakah penggunaan kain melebihi anggaran atau ada pemborosan di meja potong.
- **US-008c-02 [P1 - Mandor Potong]**: Sebagai *Mandor Potong*, saya ingin melihat statistik yield hasil potongan saya, agar saya dapat mengevaluasi penataan pola kain yang lebih hemat.

---

## 4. Kriteria Penerimaan (Acceptance Criteria - Gherkin)

### Skenario 1: Evaluasi Efisiensi Kain SPK
- **Given** SPK `SPK-0012` dialokasikan 26.0 kg kain.
- **And** Realisasi pemotongan menggunakan 22.0 kg kain dan mengembalikan 3.5 kg sisa rol ke gudang.
- **When** Laporan efisiensi bahan dimuat untuk SPK tersebut.
- **Then** Sistem menampilkan: Konsumsi Riil = 22.0 kg, Retur Sisa Rol = 3.5 kg, Limbah/Perca Aktual = 0.5 kg (2.22% waste aktual dari kain potong).

---

## 5. Kontrak API RESTful

### Endpoint: `GET /api/v1/reporting/fabric-yield?start_date=2026-10-01&end_date=2026-10-31`
- **Response Payload (HTTP 200 OK)**:
```json
{
  "status": "success",
  "data": {
    "total_fabric_allocated_kg": 1250.0,
    "total_fabric_consumed_kg": 1180.5,
    "total_roll_returned_kg": 69.5,
    "average_yield_percentage_bps": 9420,
    "average_waste_percentage_bps": 580
  }
}
```

---

## 6. Kerangka Pemecahan Tugas (Task Breakdown)
- [ ] **Fase 1**: Kueri kalkulasi selisih alokasi, pemakaian, dan retur rol di sqlc.
- [ ] **Fase 2**: Service laporan yield kain dengan perbandingan toleransi susut.
- [ ] **Fase 3**: Handler route `GET /api/v1/reporting/fabric-yield`.
- [ ] **Fase 4**: Antarmuka tabel efisiensi kain dan ekspor CSV/Excel di SolidJS.
