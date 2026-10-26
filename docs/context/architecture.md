# Standar Arsitektur Sistem SewERP

Dokumen ini mendefinisikan prinsip-prinsip arsitektur global, pemisahan bounded context, isolasi multi-tenancy, dan standar integritas data untuk SewERP.

---

## Rekomendasi Skills untuk Agent (Recommended Agent Skills)

Ketika agen bertugas merancang arsitektur sistem, memetakan batas bounded context, atau memvalidasi integritas multi-tenancy dan presisi moneter:
- `erd-api-contract`: Memodelkan entitas relasional multi-tenant, primary key UUIDv7, dan antarmuka API RESTful per domain DDD.
- `spec-kit`: Mengelola spesifikasi konstitusi arsitektur, kepatuhan Clean Architecture, dan checklist konsistensi non-fungsional.
- `codegraph`: Memetakan dependensi lintas bounded context, memeriksa pelanggaran boundary antar-domain, dan melacak alur event in-process.
- `autonomous-investigation`: Menganalisis trade-off arsitektur modular monolith vs microservices, skalabilitas konkurensi, dan investigasi pola sistem.

---

## 1. Prinsip Utama & Pola Arsitektur

SewERP mengadopsi pola **Domain-Driven Design (DDD) Modular Monolith** dengan Clean Architecture. Pendekatan ini dipilih untuk menjaga kesederhanaan operasional (single deployable binary) namun tetap menjamin batas domain (*bounded context*) yang terisolasi ketat, sehingga memudahkan pemisahan menjadi microservices jika skala pabrik konveksi berkembang pesat di masa depan.

### Batasan Bounded Context
Sistem dibagi menjadi 6 modul domain inti:
1. **`crm` (Customer Relationship Management)**: Mengelola master pelanggan, riwayat interaksi, dan katalog daftar harga bertingkat (*tiered price lists*).
2. **`oms` (Order Management System)**: Mengelola siklus hidup penawaran harga (*quotation*), pesanan resmi (*sales order* / SPK), matriks ukuran (*size breakdown*), verifikasi pembayaran DP/pelunasan, dan surat jalan.
3. **`mes` (Manufacturing Execution System / Produksi)**: Melacak progres kerja 6 stasiun konveksi (Bahan Datang -> Cutting -> Sablon/Bordir -> Sewing -> QC & Packing -> Siap Kirim) dan cetak tiket bundel kerja (*work order*).
4. **`wms` (Warehouse & Inventory Management)**: Mencatat stok kain (roll/yard/kg), aksesoris garmen, dan alokasi bahan per SPK dengan toleransi penyusutan (*wastage margin*).
5. **`payroll` (HRIS & Upah Borongan)**: Mengelola daftar pekerja (penjahit, tukang potong, QC) dan pencatatan output harian untuk kalkulasi upah borongan per pcs.
6. **`kms` (Knowledge Management System)**: Manajemen berkas pola baju (CAD/DXF/PDF), *tech pack*, dan direktori kontak supplier bahan baku.

---

## 2. Multi-Tenancy & Isolasi Data

Setiap data operasional konveksi bersifat terisolasi menggunakan arsitektur **Shared Database, Shared Schema** dengan penegakan baris data (*Row-Level Multi-Tenancy*).

- **Tenant Identifier**: Setiap tabel transaksional dan master wajib memiliki kolom:
  ```sql
  tenant_id UUID NOT NULL
  ```
- **Context Injection**: Middleware HTTP mengekstrak `tenant_id` dari token sesi/JWT pengguna dan menyuntikkannya ke dalam konteks request (`context.Context`).
- **Enforcement At Repositories**: Seluruh kueri pembacaan (`SELECT`) dan manipulasi data (`INSERT`, `UPDATE`, `DELETE`) wajib menyertakan filter `WHERE tenant_id = $1`. Larangan keras melakukan query tanpa klausa tenant.

---

## 3. Presisi Finansial: Zero Floating-Point Rule

Dalam industri konveksi, kesalahan pembulatan pada penawaran kuantitas besar (ribuan pcs) atau kalkulasi upah borongan receh per pcs dapat menimbulkan selisih neraca kas yang signifikan.

- **Mata Uang**: Dilarang menggunakan tipe data `FLOAT`, `DOUBLE`, atau `REAL`. Seluruh nilai nominal mata uang wajib disimpan dalam bentuk bilangan bulat integer terkecil (`BIGINT`) dalam format Rupiah tanpa desimal (atau satuan minor terkecil):
  - Contoh: Nilai Rp 150.000 disimpan sebagai `150000` (`BIGINT`).
- **Persentase & Diskon**: Disimpan dalam bentuk *Basis Points* (`rate_bps` dengan tipe `INT` atau `BIGINT`), di mana 100% = 10.000 bps.
  - Diskon 5% = 500 bps.
  - PPN 11% = 1.100 bps.
  - Toleransi penyusutan kain 3.5% = 350 bps.

---

## 4. Standar Identitas Entitas (UUIDv7)

Seluruh entitas menggunakan **UUIDv7** (Time-Ordered Unique Identifier) sebagai *Primary Key*:
- Menjamin keunikan global tanpa bergantung pada sequence database.
- Terurut berdasarkan waktu (*time-sortable*), menghasilkan performa indeks B-Tree yang sangat efisien dan mencegah fragmentasi halaman memori pada PostgreSQL.
- Kompatibel dengan migrasi dan sinkronisasi data antar cabang tanpa konflik kunci primer.

---

## 5. Komunikasi Antar-Domain & Event-Driven

1. **In-Process Domain Events**: Komunikasi sinkron antar-domain dilakukan melalui interface layanan yang didefinisikan secara eksplisit (*Contract First*).
2. **Asynchronous Background Jobs**: Tugas berat seperti *rendering* PDF SPK/Invoice, pengiriman pesan WhatsApp notifikasi, dan rekapitulasi penggajian bulanan dieksekusi secara asinkron menggunakan antrean **River** berbasis PostgreSQL transactional outbox.

---

## 6. Referensi & Dokumen Terkait

- ⚙️ Standar Rekayasa Backend: [backend.md](./backend.md)
- 🗄️ Standar Database & Administrasi: [dbadmin.md](./dbadmin.md)
- 🖥️ Standar Rekayasa Frontend: [frontend.md](./frontend.md)
- 📋 Kebutuhan Bisnis Konveksi: [business_requirements.md](./business_requirements.md)
- 🔧 Kebutuhan Teknis Non-Fungsional: [technical_requirements.md](./technical_requirements.md)
- 📦 Spesifikasi Modul OMS & SPK: [../specs/003a-sales-order-core.md](../specs/003a-sales-order-core.md)
- 🧵 Spesifikasi Modul Produksi: [../specs/004a-production-tracking.md](../specs/004a-production-tracking.md)
- 📋 Standar Dokumentasi Requirement: [requirements_standard.md](./requirements_standard.md)
