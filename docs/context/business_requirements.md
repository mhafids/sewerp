# Standar Kebutuhan Bisnis Konveksi Garmen SewERP

Dokumen ini memuat model bisnis, alur kerja operasional harian, aturan validasi bisnis, dan logika perhitungan khusus untuk industri manufaktur konveksi pakaian jadi.

---

## Rekomendasi Skills untuk Agent (Recommended Agent Skills)

Ketika agen bertugas menganalisis kebutuhan bisnis garmen, aturan validasi SPK, margin susut bahan, formula upah borongan, atau kebijakan uang muka:
- `intent-debugger`: Menganalisis ide mentah klien, mengidentifikasi ambiguitas operasional garmen, mendeteksi edge cases (seperti split order atau revisi makloon), dan mengonfirmasi batas cakupan.
- `jobs-to-be-done`: Merumuskan kebutuhan mendasar pelanggan, sales representatif, dan pengelola pabrik dalam format Jobs, Pains, dan Gains.
- `customer-journey-map`: Menyelaraskan alur 8 fase bisnis konveksi dari prospek hingga pengiriman barang.
- `prd-taskmaster` / `prd-development`: Mentransformasikan kebutuhan bisnis menjadi dokumen PRD yang terstruktur dan siap dieksekusi oleh tim rekayasa.
- `user-story`: Memformulasikan cerita pengguna berformat Mike Cohn (`As a ... I want ... so that ...`) lengkap dengan kriteria penerimaan Gherkin.

---

## 1. Siklus Hidup Inti Bisnis Konveksi (End-to-End Workflow)

Operasional bisnis konveksi baju bergerak dalam 8 tahapan terstruktur:

```text
1. Prospek & Permintaan ──> 2. Penawaran Harga (Quotation) ──> 3. DP (Uang Muka 50%)
                                                                      │
                                                                      ▼
6. Pencatatan Borongan   <── 5. Jalur 6 Stasiun Produksi   <── 4. Terbit SPK Resmi &
   (Upah per Pcs)               (Potong s/d Packing)              Alokasi Kain (+4% Margin)
         │
         ▼
7. Pelunasan Pembayaran ──> 8. Surat Jalan & Pengiriman
```

1. **Pencatatan Prospek Masuk (Leads)**: Merekam calon pembeli dari saluran WhatsApp, Instagram, atau kunjungan langsung.
2. **Penerbitan Penawaran Harga (Quotation)**: Menghitung harga jual berdasarkan master *Price List* dengan tiering kuantitas (misal: 24-49 pcs, 50-99 pcs, 100-499 pcs, 500+ pcs), jenis kain, serta add-on sablon/bordir.
3. **Penerimaan DP & Penerbitan SPK**: Pesanan resmi (*Sales Order* / SPK) hanya terbit setelah pembayaran Uang Muka (DP minimal 50%) diverifikasi.
4. **Alokasi Bahan Baku & Toleransi Penyusutan**: Sistem menghitung estimasi kebutuhan kain (yard/kg/roll) ditambah **Toleransi Penyusutan (Wastage Margin 3% - 5%)** untuk mengantisipasi cacat pinggir atau susut saat proses press/sablon.
5. **Eksekusi 6 Stasiun Produksi**:
   - Stasiun 1: Bahan Datang (Pengecekan fisik kain & aksesoris)
   - Stasiun 2: Meja Potong / Cutting (Gelar kain, potong pola, ikat bundel)
   - Stasiun 3: Sablon / Bordir (Cetak grafis sebelum atau sesudah potong)
   - Stasiun 4: Meja Jahit / Sewing (Perakitan garmen oleh penjahit borongan)
   - Stasiun 5: QC & Packing (Pembersihan benang, setrika uap, hangtag, polybag)
   - Stasiun 6: Siap Kirim (Penyimpanan di rak transit barang jadi)
6. **Kalkulasi Upah Borongan**: Menghitung hak upah per penjahit berdasarkan kuantitas baju yang diselesaikan dan lolos QC.
7. **Penagihan Pelunasan (Final Invoice)**: Mengirimkan tagihan sisa pembayaran sebelum barang diberangkatkan.
8. **Penerbitan Surat Jalan & Pengiriman**: Mencetak dokumen fisik Surat Jalan dan Packing List sebagai bukti serah terima kurir/ekspedisi.

---

## 2. Aturan Bisnis Kunci (Strict Business Rules)

### BR-01: Otorisasi Mulai Produksi
Pekerjaan cutting kain pada modul produksi **dilarang dimulai** sebelum status pembayaran SPK minimal bernilai `dp_paid` (DP lunas), kecuali terdapat otorisasi *Bypass* darurat dengan persetujuan Direktur/Owner konveksi yang tercatat pada audit log.

### BR-02: Batas Toleransi Penyusutan Kain (Wastage Margin)
- Setiap kalkulasi kebutuhan bahan baku untuk SPK wajib menambahkan toleransi penyusutan baku:
  $$\text{Kebutuhan Bahan Final} = \text{Konsumsi Standar} \times \left(1 + \frac{\text{Wastage Margin BPS}}{10.000}\right)$$
- Nilai default sistem adalah **4% (400 bps)**, dapat dikonfigurasi antara **3% hingga 5%** tergantung jenis bahan (kain fleece dan combed tebal memiliki susut lebih tinggi daripada katun drill).

### BR-03: Integritas Upah Borongan (Anti-Overclaim)
- Total klaim kuantitas jahit dari seluruh penjahit untuk satu nomor SPK tidak boleh melebihi total pcs hasil potong yang dinyatakan lolos dari stasiun cutting.
- Pcs baju yang berstatus *Reject Jahit* akibat kelalaian penjahit tidak mendapatkan upah dan dicatat dalam rekap penalti.

### BR-04: Pelunasan Sebelum Keberangkatan Ekspedisi
Surat Jalan pengiriman pihak ketiga (JNE Trucking, Dakota Cargo, Indah Kargo) tidak dapat diterbitkan jika status pembayaran belum `fully_paid`, kecuali pelanggan bertipe institusi dengan perjanjian tempo (*Terms of Payment*).

---

## 3. Autentikasi Google SSO & Matriks Hak Akses Granular (RBAC)

SewERP menerapkan pengamanan tingkat tinggi untuk operasional konveksi:
1. **Autentikasi 100% Google Single Sign-On (SSO)**: Mengeliminasi kerentanan password lemah di lantai pabrik. Sistem hanya mengizinkan akun Google yang sudah terdaftar (*Whitelist Invitation Only*) oleh Owner/Admin.
2. **Sesi Backend 1 Shift Kerja (8 Jam)**: Sesi disimpan di PostgreSQL dan browser hanya menyimpan cookie `HttpOnly; Secure; SameSite=Lax`.
3. **Pemisahan Hak Akses Menu & Aksi Granular**: Akses menu mengatur visibilitas halaman, sementara 6 aksi inti (`view`, `create`, `update`, `delete`, `export`, `import`) mengatur izin eksekusi data yang ditegakkan secara mutlak di middleware backend dan route guard frontend.

| Peran Pengguna | Akses Menu Utama | Hak Aksi CRM & OMS | Hak Aksi Produksi (MES) | Hak Aksi Gudang (WMS) | Hak Aksi Upah (Payroll) |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **Admin Penjualan** | Dashboard, CRM, OMS, Laporan | `V, C, U, E, I` | `V` | `V (Stok Kain)` | Ditolak |
| **Manajer Produksi** | Dashboard, OMS, MES, WMS, Laporan | `V` | `V, C, U, E, I, Approve, Print` | `V, U (Alokasi Bahan)` | `V, U (Verifikasi Pcs)` |
| **Mandor / Operator** | MES (Produksi Saja) | Ditolak | `V, U (Update Stasiun)` | Ditolak | `V, C (Input Output)` |
| **Staf Gudang (WMS)**| Gudang & Stok Saja | Ditolak | `V (Kebutuhan SPK)` | `V, C, U, E, I` | Ditolak |
| **Kasir & HRD** | OMS (Bayar), HRIS & Payroll | `V, U (Bayar), Print` | `V (Progres Jahit)` | Ditolak | `V, C, U, E, I, Approve` |
| **Super Admin / Owner**| Seluruh Menu & Pengaturan | Penuh (`ALL`) | Penuh (`ALL`) | Penuh (`ALL`) | Penuh (`ALL`) |

> *Keterangan Aksi: `V` = View, `C` = Create, `U` = Update, `D` = Delete, `E` = Export Excel/PDF, `I` = Import Excel/CSV.*

Detail teknis, skema database, dan kontrak API dirinci pada: [Spesifikasi Modul 010a - Autentikasi Google SSO & RBAC Granular](../specs/010a-google-sso.md).

---

## 4. Referensi & Dokumen Terkait

- 🏛️ Standar Arsitektur: [architecture.md](./architecture.md)
- 🔧 Standar Kebutuhan Teknis: [technical_requirements.md](./technical_requirements.md)
- 🧪 Standar Pengujian & Validasi Bisnis: [qa.md](./qa.md)
- 🔐 Spesifikasi Autentikasi & RBAC (IAM): [../specs/010a-google-sso.md](../specs/010a-google-sso.md)
- 📄 Spesifikasi Sales & Order (OMS): [../specs/003a-sales-order-core.md](../specs/003a-sales-order-core.md)
- 🧵 Spesifikasi Modul Produksi: [../specs/004a-production-tracking.md](../specs/004a-production-tracking.md)
- 📦 Spesifikasi Manajemen Inventaris: [../specs/005a-fabric-inventory.md](../specs/005a-fabric-inventory.md)
- 👥 Spesifikasi HRIS & Upah Borongan: [../specs/006a-worker-roster-tariffs.md](../specs/006a-worker-roster-tariffs.md)
- 📋 Standar Dokumentasi Requirement: [requirements_standard.md](./requirements_standard.md)
