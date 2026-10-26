# Spesifikasi Sub-Modul: 004c - Tiket Bundel Kerja & Cetak Thermal Tag (MES)

Dokumen ini memuat spesifikasi pembagian bundel kerja potongan kain (*Cutting Bundle Division*), format cetak tiket bundel fisik (*Thermal Work Order Tag* 80mm / A6 Sticker), dan integrasi Barcode/QR Code untuk penelusuran di lantai kerja.

---

## 1. Metadata & Tautan Referensi Terkait

- **Kode Modul**: `SPEC-004c`
- **Domain Bounded Context**: `production` / `mes`
- **Tautan Standar Acuan Global**:
  - 📋 [Standar Penulisan Requirement](../context/requirements_standard.md)
  - 🔧 [Standar Kebutuhan Teknis & Cetak Thermal](../context/technical_requirements.md)
  - 👆 [Prinsip UX Lapangan Pabrik](../context/ux.md)
  - 🧪 [Standar QA & Benchmark Testing](../context/qa.md)
- **Tautan Spesifikasi Fitur Terkait**:
  - 🧵 [004a - Papan Pelacakan 6 Stasiun Produksi](./004a-production-tracking.md)
  - 🔍 [004d - Inspeksi QC & Potong Tambal](./004d-qc-inspection.md)
  - 👥 [006b - Input Harian Output Jahit Mobile](./006b-piece-rate-logging.md)

---

## 2. Ringkasan Kebutuhan & Masalah Bisnis

### Masalah Lapangan
Saat kain dipotong dalam jumlah ratusan pcs, bundel potongan lengan, badan depan, dan kerah sering tertukar antar-ukuran (misal lengan size M terjahit ke badan size L) karena tidak adanya label fisik yang diikatkan pada tumpukan kain potong.

### Solusi Fitur
1. **Pembagian Bundel Otomatis**: Memecah kuantiti potong ke dalam bundel kerja berisi 20–25 pcs per bundel per ukuran.
2. **Cetak Tiket Bundel Thermal**: Menghasilkan tag tiket ukuran 80mm roll atau stiker A6 berisi informasi nomor SPK, kode bundel, ukuran pakaian, instruksi khusus, checklist paraf stasiun kerja, dan barcode/QR Code.

---

## 3. User Stories

- **US-004c-01 [P1 - Mandor Potong]**: Sebagai *Mandor Potong*, saya ingin mencetak tiket bundel thermal begitu selesai memotong kain, agar setiap tumpukan potongan kain memiliki label identitas yang jelas saat dibawa ke meja sablon atau jahit.
- **US-004c-02 [P1 - Operator Jahit/Mandor]**: Sebagai *Mandor Jahit*, saya ingin memindai barcode pada tiket bundel saat mencatat hasil jahit, agar nomor SPK dan ukuran otomatis terpilih tanpa risiko salah ketik.

---

## 4. Format Fisik Tiket Bundel Thermal (80mm / A6 Sticker)

```text
========================================
           SEWERP PRODUCTION TAG
========================================
NO. SPK   : SPK-202610-0012
DEADLINE  : 10 Nov 2026 (URGENT)
PELANGGAN : PT Sumber Busana
ARTIKEL   : Polo Shirt Seragam Telkom
WARNA     : Merah Maroon
----------------------------------------
KODE BUNDEL : BND-0012-L-01
UKURAN      : L (Large)
ISI BUNDEL  : 24 Pcs
----------------------------------------
INSTRUKSI KHUSUS:
- Sablon Plastisol dada kiri & punggung
- Kancing warna senada kain (Maroon)
- Belahan samping bawah jahit bartack
----------------------------------------
CHECKLIST STASIUN:
[ ] Potong     : ............. (Paraf)
[ ] Makloon/Pr : ............. (Paraf)
[ ] Jahit      : ............. (Paraf)
[ ] QC & Pack  : ............. (Paraf)
========================================
[  BARCODE / QR CODE SPK-202610-0012-B1  ]
========================================
```

---

## 5. Kontrak API RESTful

### Endpoint: `GET /api/v1/production/jobs/{id}/bundles/{bundle_id}/ticket?format=thermal80`
- **Response**: Binary payload stream PDF atau ESC/POS command stream untuk printer thermal 80mm.

---

## 6. Kerangka Pemecahan Tugas (Task Breakdown)
- [ ] **Fase 1**: Goose migration tabel `production_bundles`.
- [ ] **Fase 2**: Algoritma pembagian bundel kerja per 24 pcs per ukuran.
- [ ] **Fase 3**: Layout generator tiket thermal 80mm dengan generator Barcode Code128.
- [ ] **Fase 4**: Tombol "Cetak Semua Tiket Bundel" pada tampilan meja potong.
