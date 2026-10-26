# Standar Desain Antarmuka (UI) Modern Flat UI SewERP

Dokumen ini mendefinisikan filosofi desain visual, palet warna, tipografi, komponen desain sistem, dan token UnoCSS untuk SewERP.

---

## Rekomendasi Skills untuk Agent (Recommended Agent Skills)

Ketika agen bertugas mendesain antarmuka, menyusun layout Bento Grid, mengonfigurasi palet warna, atau memperbarui token visual UnoCSS:
- `ui-ux-pro-max`: Penguasaan gaya Modern Flat UI, 1px crisp borders, zero blur shadows, palet Matcha Green (`#5B7E5F`) dan Sakura Pink (`#E8A5B8`), serta tipografi Plus Jakarta Sans.
- `generative_ui`: Menghasilkan artefak mockup visual antarmuka dan prototipe komponen layout interaktif.
- `a11y-debugging`: Memverifikasi rasio kontras warna teks terhadap latar belakang minimal 4.5:1 (WCAG AA) dan kejelasan visual elemen border.

---

## 1. Filosofi Desain: Modern Flat UI

Antarmuka SewERP mengusung estetika **Modern Flat UI**:
- **Zero Blur Shadows**: Menghilangkan drop-shadow yang kabur (*fuzzy blur*). Elemen visual dibedakan melalui kontras latar belakang dan garis batas tegas.
- **1px Crisp Borders**: Semua kartu, modal, tabel, dan input menggunakan garis tepi setebal 1px (`border border-slate-200` pada tema terang, `border-slate-800` pada tema gelap).
- **Densitas Tinggi & Ergonomis**: Ruang layar dimaksimalkan untuk menampilkan data produksi, matriks ukuran pakaian, dan tabel status tanpa perlu scrolling berlebih pada layar desktop admin.

---

## 2. Palet Warna & Visual Token

Warna primer SewERP terinspirasi dari harmoni ketelitian dan kreativitas garmen:

| Peran Warna | Nama Token | Nilai Hex | Penggunaan Utama |
| :--- | :--- | :--- | :--- |
| **Brand Primary** | `matcha-600` | `#5B7E5F` | Tombol utama, header aktif, indikator status selesai/lunas |
| **Primary Hover** | `matcha-700` | `#48654C` | Status hover tombol utama |
| **Primary Surface**| `matcha-50` | `#EBF2EB` | Latar belakang badge status, sel aktif, banner info |
| **Brand Accent** | `sakura-400` | `#E8A5B8` | Tombol penawaran, highlight promo, badge aksen khusus |
| **Accent Hover** | `sakura-500` | `#D88D9F` | Status hover tombol aksen |
| **Accent Surface** | `sakura-50` | `#FDF4F6` | Latar peringatan batas waktu dekat (*deadline alert*) |
| **Neutral Dark** | `slate-900` | `#0F172A` | Teks utama, judul dokumen |
| **Neutral Body** | `slate-700` | `#334155` | Teks konten, label formulir |
| **Neutral Muted**| `slate-400` | `#94A3B8` | Placeholder, teks bantuan, border sekunder |
| **Surface Base** | `slate-50` | `#F8FAFC` | Latar belakang halaman aplikasi |
| **Surface Card** | `white` | `#FFFFFF` | Latar belakang kartu (*card*), modal, dan sel tabel |

---

## 3. Tipografi

- **Font Keluarga Primer**: `Plus Jakarta Sans`, `Inter`, sans-serif.
- **Skala Ukuran Teks**:
  - `text-xs` (12px): Badge status, teks bantuan micro, label footer.
  - `text-sm` (14px): Teks standar isi tabel, input formulir, teks paragraf.
  - `text-base` (16px): Label tombol, judul kartu (*card header*).
  - `text-lg` (18px) - `text-2xl` (24px): Judul modul, angka metrik bento-grid.

---

## 4. Pola Komponen Visual Utama

### A. Bento-Grid Dashboard
Menyajikan metrik penting secara padat dan hierarkis:
```text
┌──────────────────────┬──────────────────────┬──────────────────────┐
│  TOTAL PO AKTIF      │  TENGGAT TERDEKAT    │  NILAI TRANSAKSI     │
│  28 Pesanan          │  3 Hari (PO-0082)    │  Rp 142.500.000      │
├──────────────────────┴──────────────────────┼──────────────────────┤
│  PROGRES 6 STASIUN PRODUKSI                 │  RINGKASAN BORONGAN  │
│  [Potong: 4] [Sablon: 6] [Jahit: 12] [QC: 6]│  Rp 18.200.000 / bln │
└─────────────────────────────────────────────┴──────────────────────┘
```

### B. Badge Status Produksi & Finansial
- `Draft / Penawaran`: `bg-slate-100 text-slate-700 border-slate-300`
- `DP Terbayar / Siap Potong`: `bg-blue-50 text-blue-700 border-blue-200`
- `Sedang Jahit`: `bg-amber-50 text-amber-700 border-amber-200`
- `Lolos QC & Packing`: `bg-emerald-50 text-emerald-700 border-emerald-200`
- `Lunas & Siap Kirim`: `bg-matcha-50 text-matcha-700 border-matcha-200`
- `Deadline Bahaya (<3 Hari)`: `bg-rose-50 text-rose-700 border-rose-200 animate-pulse`

---

## 5. Referensi & Dokumen Terkait

- 👆 Standar Interaksi & UX Lapangan: [ux.md](./ux.md)
- 🖥️ Standar Rekayasa Frontend: [frontend.md](./frontend.md)
- 📝 Standar Formulir & Input: [forms.md](./forms.md)
- 📊 Spesifikasi Dashboard: [../specs/001a-dashboard-overview-metrics.md](../specs/001a-dashboard-overview-metrics.md)
- 🧵 Spesifikasi Modul Produksi: [../specs/004a-production-tracking.md](../specs/004a-production-tracking.md)
- 📋 Standar Dokumentasi Requirement: [requirements_standard.md](./requirements_standard.md)
