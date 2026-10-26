# Spesifikasi Sub-Modul: 001d - Dashboard Bento-Grid Layout & RBAC Persona Views

Dokumen ini memuat spesifikasi tata letak responsif berbasis **Bento-Grid 12-Kolom Modern Flat UI** serta matriks personalisasi tampilan widget dashboard berdasarkan peran pengguna (*Role-Based Views*).

---

## 1. Metadata & Tautan Referensi Terkait

- **Kode Modul**: `SPEC-001d`
- **Domain Bounded Context**: `analytics` / `dashboard`
- **Tautan Standar Acuan Global**:
  - 📋 [Standar Penulisan Requirement](../context/requirements_standard.md)
  - 🖥️ [Standar Frontend SolidJS Ultra-Strict](../context/frontend.md)
  - 🎨 [Standar Desain Antarmuka Flat UI](../context/ui.md)
  - 👆 [Prinsip UX Lapangan Pabrik](../context/ux.md)
  - 🛡️ [Standar Keamanan, PII & RLS](../context/security.md)
- **Tautan Spesifikasi Fitur Terkait**:
  - 📊 [001a - Quick Counts & Metrik Agregasi](./001a-dashboard-overview-metrics.md)
  - 📊 [001b - Visualisasi Analitik Penjualan & Pipeline](./001b-dashboard-sales-analytics.md)
  - ⚠️ [001c - Radar Deadline Kritis & Peringatan Stok](./001c-dashboard-radar-alerts.md)
  - 🔐 [010c - Matriks Hak Akses Granular RBAC](./010c-granular-rbac.md)

---

## 2. Ringkasan Kebutuhan & Masalah Bisnis

### Masalah Lapangan
Jika semua widget ditampilkan sekaligus tanpa memandang peran pengguna:
1. Informasi rahasia omset dan laba pabrik terlihat oleh operator jahit atau staf gudang.
2. Layar menjadi terlalu padat (*visual clutter*), menyulitkan staf operasional menemukan data yang relevan dengan tugas harian mereka.

### Solusi Fitur
Tata letak Bento-Grid yang secara otomatis me-*mount* dan me-*unmount* widget sesuai role yang tersemat pada token otorisasi pengguna (`Owner`, `Production_Manager`, `Sales_Admin`, `Cashier`, `Workshop_Foreman`).

---

## 3. Matriks Hak Akses & Personalisasi Widget

| Komponen / Widget Bento | Owner | Manajer Produksi | Admin Sales | Kasir / Keuangan | Mandor Bengkel / Gudang |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Kartu Omset & Piutang** | ✅ Tampil | ❌ Sembunyi | ❌ Sembunyi | ✅ Tampil | ❌ Sembunyi |
| **Kartu Kas Masuk & Pending**| ✅ Tampil | ❌ Sembunyi | ❌ Sembunyi | ✅ Tampil | ❌ Sembunyi |
| **Grafik Tren 6 Bulan** | ✅ Penuh | ⚠️ Volume Saja | ⚠️ Volume Saja | ❌ Sembunyi | ❌ Sembunyi |
| **Grafik Pipeline Status** | ✅ Lengkap | ⚠️ Status Saja | ✅ Lengkap | ✅ Nilai DP | ⚠️ Status WIP Saja |
| **Radar Deadline Kritis** | ✅ Tampil | ✅ Aksi Cepat | ✅ Baca Saja | ❌ Sembunyi | ✅ Aksi Mandor |
| **Peringatan Stok Rendah** | ✅ Tampil | ✅ Tampil | ❌ Sembunyi | ❌ Sembunyi | ✅ Aksi Gudang |
| **Liabilitas Upah & Kasbon** | ✅ Tampil | ❌ Sembunyi | ❌ Sembunyi | ✅ Tampil | ❌ Sembunyi |

---

## 4. User Stories

- **US-001d-01 [P1 - Semua Role]**: Sebagai *Pengguna Terautentikasi*, saya ingin layout dashboard otomatis menyajikan widget yang relevan dengan tanggung jawab tugas saya, agar antarmuka bersih dan cepat dimuat.
- **US-001d-02 [P1 - Owner]**: Sebagai *Owner Konveksi*, saya ingin melihat kokpit lengkap mencakup keuangan, grafik tren, dan radar operasional sekaligus dalam satu bentang layar 12-kolom desktop.

---

## 5. Kriteria Penerimaan (Acceptance Criteria - Gherkin)

### Skenario 1: Layout Khusus Manajer Produksi
- **Given** Pengguna login dengan role `Production_Manager`.
- **When** Rute `/dashboard` dimuat di peramban.
- **Then** Komponen kartu keuangan tidak di-render ke dalam Virtual DOM / real DOM.
- **And** Widget Radar Deadline Kritis dan Beban Stasiun Kerja mengisi lebar 12 kolom secara penuh (*auto-expanded grid*).

---

## 6. Struktur Tata Letak Bento-Grid (UnoCSS Flat Tokens)
```text
+---------------------------------------------------------------------------------------------------+
|  [Header] Selamat Datang, Pak Budi (Owner)            |  Filter: [6 Bulan v]  | [Tombol Refresh]  |
+---------------------------------------------------------------------------------------------------+
| [STAT 1: Omset]             | [STAT 2: Piutang]            | [STAT 3: Kas Hari Ini]  | [STAT 4: PO Aktif] |
+-----------------------------+------------------------------+-------------------------+--------------------+
|                                                            |                                              |
|  GRAFIK UTAMA (Kolom 1 - 7):                               |  GRAFIK PIPELINE (Kolom 8 - 12):             |
|  Dual-Axis Tren Penjualan 6 Bulan                          |  Pipeline Siklus Pesanan                     |
+------------------------------------------------------------+----------------------------------------------+
|                                                            |                                              |
|  RADAR DEADLINE KRITIS (< 3 Hari) (Kolom 1 - 7):           |  PERINGATAN STOK & OPERASIONAL (Kolom 8 - 12)|
+------------------------------------------------------------+----------------------------------------------+
```

---

## 7. Kerangka Pemecahan Tugas (Task Breakdown)
- [ ] **Fase 1**: Buat komponen deklaratif SolidJS `<Can role="...">` untuk seleksi widget.
- [ ] **Fase 2**: Implementasikan layout Bento-Grid di `frontend/src/routes/dashboard.tsx`.
- [ ] **Fase 3**: Uji responsivitas pada breakpoint Desktop (1200px+), Tablet (768px), dan HP (375px).
