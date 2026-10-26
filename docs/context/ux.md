# Standar Pengalaman Pengguna (UX) Lapangan Konveksi SewERP

Dokumen ini mendefinisikan prinsip ergonomi antarmuka, alur interaksi pengguna, dan optimasi khusus untuk lingkungan kerja pabrik garmen/konveksi baju.

---

## Rekomendasi Skills untuk Agent (Recommended Agent Skills)

Ketika agen bertugas merancang alur interaksi operator pabrik, pemindaian QR tiket bundel, form mobile penjahit, atau toleransi offline:
- `ui-ux-pro-max`: Menerapkan prinsip ergonomi antarmuka kerja, target sentuh minimal 48x48px (*fat-finger prevention*), feedback haptic/audio, dan keypad angka cepat.
- `customer-journey-map` / `customer-journey-mapping-workshop`: Memetakan alur kerja pengguna lintas peran (Admin Sales -> Manajer Potong -> Operator Sablon -> Penjahit Borongan -> QC & Packing -> Kasir Pengiriman).
- `jobs-to-be-done`: Menganalisis kebutuhan pekerjaan inti, pain points debu/oli, dan efisiensi waktu operasional pekerja konveksi.
- `a11y-debugging`: Memvalidasi ukuran target sentuh di layar perangkat seluler dan kejelasan feedback visual/auditori saat pemindaian berhasil.

---

## 1. Realitas Lapangan Pabrik Konveksi

Desain UX SewERP mempertimbangkan kondisi fisik nyata area workshop konveksi:
1. **Debu Serat Kain & Tangan Operator**: Area meja potong dan mesin jahit dipenuhi debu benang. Tangan operator sering kali berkeringat atau kotor terkena oli mesin jahit, sehingga membutuhkan target sentuh yang besar dan toleran terhadap sentuhan tidak presisi.
2. **Koneksi Wi-Fi Fluktuatif**: Area belakang workshop sering mengalami sinyal internet yang lemah (*spotty network*). Form pencatatan harian harus memiliki toleransi offline sementara (*local draft*).
3. **Pencahayaan Kontras**: Area cutting terang benderang karena lampu meja potong, sementara sudut gudang kain sering redup. Antarmuka harus mempertahankan kontras rasio minimal 4.5:1 (WCAG AA).

---

## 2. Prinsip Desain UX Kunci

### A. Target Sentuh Minimal 48x48 Pixel
Untuk perangkat tablet dan smartphone yang digunakan oleh mandor potong, penjahit, dan staf QC:
- Seluruh tombol aksi vital (tombol "Mulai", "Selesai", "Lolos QC", "Tambah Output") memiliki tinggi minimal 48px dengan padding luas.
- Jarak antar tombol minimal 12px untuk mencegah salah pencet (*fat-finger prevention*).

### B. Fast Numeric Input & Number Pad Khusus
- Saat memasukkan kuantitas ukuran baju (S, M, L, XL) atau jumlah pcs hasil jahit borongan, keyboard otomatis membuka mode angka (`inputmode="numeric"`).
- Menyediakan tombol cepat pintasan (misal: `+12`, `+24`, `+60`, atau "Isi Sesuai SPK") untuk mempercepat input dalam hitungan detik.

### C. Pemindaian Cepat Tiket Bundel (QR/Barcode Scanning)
- Pada stasiun jahit dan QC, operator tidak perlu mengetikkan nomor SPK manual yang panjang.
- Sistem menyediakan tombol pemindai kamera bawaan browser (*Camera QR Scanner*) untuk memindai tiket bundel fisik yang ditempel pada tumpukan kain potong.

### D. Umpan Balik Visual & Audio (Haptic/Audio Feedback)
- Setiap kali pemindaian berhasil atau form borongan tersimpan, sistem membunyikan nada "beep" pendek dan menampilkan toast hijau tebal.
- Jika terjadi kelebihan klaim (*overclaim*) atau kuantiti melebihi SPK, layar memunculkan peringatan merah menyala dengan getaran singkat (haptic) di perangkat seluler.

---

## 3. Matriks Alur Pengguna (User Journey Mapping)

```text
[ Admin Sales / Kasir ]
Quotation Masuk ──> Hitung Otomatis Tiering ──> Input DP ──> SPK Terbit
                                                               │
                                                               ▼
[ Manajer Produksi / Cutting ]
Alokasi Kain (+4% Wastage) ──> Cetak Tiket Bundel (Thermal) ──> Meja Potong
                                                               │
                                                               ▼
[ Operator Sablon & Penjahit ]
Scan QR Tiket ──> Eksekusi Jahit ──> Input Output Harian via HP/Tablet
                                                               │
                                                               ▼
[ Tim QC & Packing ]
Pemeriksaan Cacat ──> Verifikasi Lolos QC ──> Masuk Polybag ──> Siap Kirim
                                                               │
                                                               ▼
[ Kasir & Admin Pengiriman ]
Pelunasan Sisa Tagihan ──> Cetak Surat Jalan / Packing List ──> Ekspedisi
```

---

## 4. Referensi & Dokumen Terkait

- 🎨 Standar Visual Desain UI: [ui.md](./ui.md)
- 📝 Standar Formulir & Validasi: [forms.md](./forms.md)
- 📋 Kebutuhan Bisnis Konveksi: [business_requirements.md](./business_requirements.md)
- 🧵 Spesifikasi Modul Produksi: [../specs/004a-production-tracking.md](../specs/004a-production-tracking.md)
- 👥 Spesifikasi HRIS & Payroll Borongan: [../specs/006a-worker-roster-tariffs.md](../specs/006a-worker-roster-tariffs.md)
- 📋 Standar Dokumentasi Requirement: [requirements_standard.md](./requirements_standard.md)
