# Standar Rekayasa & Penulisan Spesifikasi Kebutuhan (Requirements Standard)

Dokumen ini merupakan standar resmi acuan global di **SewERP** untuk:
1. Protokol penggalian kebutuhan lengkap (*Requirements Elicitation Protocol*) dari user / klien konveksi.
2. Aturan penamaan dan struktur modular berkas spesifikasi `xxxy-{name requirement}.md` guna menghemat token LLM dan context window agen AI.
3. Kewajiban penautan (*linking*) metadata ke dokumen standar konteks (`docs/context/*.md`) dan spesifikasi terkait.
4. Matriks aktivasi keahlian agen AI (*Agent Skills Recommendation Matrix*) di setiap tahapan penyusunan spesifikasi.

---

## 1. Alur & Protokol Penggalian Kebutuhan Lengkap dari User

Mengubah ide mentah, percakapan santai, atau keluhan operasional pabrik konveksi menjadi spesifikasi teknis siap bangun membutuhkan 7 tahapan eksplorasi terstruktur:

```
Ide Mentah / Keluhan Lantai Pabrik
   ↓
[Fase 1: Intent & Problem Discovery]
Skills: intent-debugger, problem-framing-canvas, problem-statement, proto-persona, jobs-to-be-done
   ↓
[Fase 2: Business Invariants & PRD]
Skills: prd-development, prd-taskmaster, derisk-measurement-advisor, spec-kit
   ↓
[Fase 3: User Stories & Kriteria Penerimaan]
Skills: user-story, user-story-mapping, user-story-splitting
   ↓
[Fase 4: Perancangan Data & Kontrak API]
Skills: erd-api-contract, spec-kit, codegraph
   ↓
[Fase 5: Desain Antarmuka & Ergonomi Lantai Pabrik]
Skills: ui-ux-pro-max, generative_ui, a11y-debugging
   ↓
[Fase 6: Non-Functional, Security & Zero-Alloc Benchmarks]
Skills: infosec, cookie-debugging, memory-leak-debugging, debug-optimize-lcp
   ↓
[Fase 7: Dependency-Ordered Task Breakdown]
Skills: atlas, execute-task, spec-kit
```

### 1.1. Fase 1: Intent & Problem Discovery
- **Tujuan**: Memisahkan gejala permukaan (*symptoms*) dari akar masalah (*root cause*), mengenali aktor lapangan, dan mendeteksi asumsi tersembunyi.
- **Keterampilan Direkomendasikan**:
  - `intent-debugger`: Membedah bahasa sehari-hari klien, mengekspos ambiguitas, dan menggali batasan yang belum dinyatakan.
  - `problem-framing-canvas` & `problem-statement`: Merumuskan 4 kuadran masalah: Siapa yang terhambat, Apa yang sedang diupayakan, Mengapa terhambat, dan Bagaimana dampaknya terhadap operasional konveksi.
  - `proto-persona` & `jobs-to-be-done`: Mengidentifikasi peran riil di pabrik (Owner, Admin Sales, Mandor Potong, Penjahit Borongan, Operator QC, Kasir, Kurir).
- **Pertanyaan Wajib untuk User**:
  1. Siapa aktor utama yang menginput atau membaca data ini setiap hari?
  2. Apa yang terjadi jika sistem gagal / offline saat proses ini berjalan di pabrik?
  3. Apa batasan tegas (*Out-of-Scope*) yang sengaja TIDAK dikerjakan pada rilis ini?

### 1.2. Fase 2: Business Invariants & PRD
- **Tujuan**: Menetapkan invariant bisnis konveksi yang tidak boleh dilanggar (*non-negotiable rules*) dan tujuan terukur (KPI).
- **Keterampilan Direkomendasikan**:
  - `prd-development` & `prd-taskmaster`: Mengompilasi latar belakang masalah, target pengguna, dan daftar fitur fungsional.
  - `derisk-measurement-advisor`: Menilai kelayakan teknis, risiko manipulasi kas/gaji, dan de-risking operasional.
  - `spec-kit`: Menjaga keselarasan dengan arsitektur Modular Monolith DDD Go & SolidJS.
- **Standar Invariant Konveksi Wajib**:
  - *Multi-Tenancy*: Setiap entitas wajib terikat pada `tenant_id`. Tidak boleh ada kebocoran antar-pabrik.
  - *Zero Floating-Point*: Nominal uang wajib menggunakan `BIGINT` (Rupiah murni tanpa desimal). Persentase menggunakan basis poin (`rate_bps`, misal 500 = 5.00%).
  - *Soft Deletion*: Data transaksi tidak boleh dihapus fisik (`deleted_at IS NULL`).

### 1.3. Fase 3: User Stories & Acceptance Criteria (Gherkin 4-Pilar)
- **Tujuan**: Menyusun daftar cerita pengguna terprioritas dan kriteria penerimaan yang dapat diuji otomatis.
- **Keterampilan Direkomendasikan**:
  - `user-story`: Menulis format Mike Cohn: `Sebagai [Aktor], saya ingin [Aksi/Kapabilitas] agar [Tujuan/Manfaat]`.
  - `user-story-splitting`: Memecah cerita besar menjadi unit kecil independen yang dapat dirilis per sub-modul.
  - `user-story-mapping`: Memetakan urutan alur kerja dari hulu ke hilir.
- **Standar Gherkin 4-Pilar Wajib**:
  Setiap spesifikasi wajib memiliki minimal 4 skenario pengujian:
  1. *Happy Path*: Alur ideal dengan data valid dan sukses.
  2. *Validation Error*: Kegagalan input salah tipe, batas string, atau format nomor WhatsApp/email.
  3. *Business Invariant Violation*: Pelanggaran aturan (contoh: klaim borongan melebihi potong, terbit surat jalan sebelum lunas tanpa izin tempo).
  4. *RBAC & Permission Denied*: Penolakan aksi jika peran pengguna tidak memiliki hak akses yang cukup.

### 1.4. Fase 4: Perancangan Data & Kontrak API RESTful
- **Tujuan**: Merumuskan irisan skema database dan kontrak HTTP JSON yang konsisten.
- **Keterampilan Direkomendasikan**:
  - `erd-api-contract`: Menghasilkan diagram Mermaid `erDiagram`, tipe PostgreSQL, UUIDv7 PK, dan DTO JSON dengan envelope `ApiResponse<T>`.
  - `codegraph`: Memverifikasi struktur Go struct eksisting dan dependensi antarmuka repository.
- **Standar Kontrak API**:
  - Mengikuti pola RFC 7807 Problem Details untuk error.
  - Validasi ketat DTO dengan `go-playground/validator`. Larangan enum `oneof=...` hardcoded; gunakan custom validator `{enum_name}_enum`.

### 1.5. Fase 5: Desain Antarmuka & Ergonomi Lantai Pabrik
- **Tujuan**: Memastikan tampilan visual responsif, kontras tinggi, dan ramah sentuhan.
- **Keterampilan Direkomendasikan**:
  - `ui-ux-pro-max`: Penerapan palet Flat UI: Matcha Green (`#5B7E5F`), Sakura Pink (`#E8A5B8`), Slate Blue (`#3B82F6`), Amber (`#F59E0B`), dan Rose Carmine (`#E11D48`). Border tegas 1px tanpa efek blur berat.
  - `generative_ui`: Pembuatan prototipe layout widget dan Bento-Grid interaktif.
  - `a11y-debugging`: Memastikan touch target berukuran minimal 48x48px untuk operator tablet berkeringat/bersarung tangan di meja potong dan jahit.

### 1.6. Fase 6: Non-Functional, Security & Zero-Alloc SLO
- **Tujuan**: Memastikan sistem cepat, aman, dan mematuhi regulasi privasi data.
- **Keterampilan Direkomendasikan**:
  - `infosec`: Klasifikasi data Level 1–4, kepatuhan UU PDP No. 27/2022, presigned URL S3 untuk pola CAD/DXF, verifikasi HMAC webhook Xendit.
  - `cookie-debugging`: Pengujian session cookie `HttpOnly; Secure; SameSite=Lax` dengan masa aktif 8 jam (1 shift).
  - `memory-leak-debugging` & `debug-optimize-lcp`: Memastikan SolidJS menyertakan `onCleanup()` pada semua chart/event listener dan beban respons API $< 100\text{ ms}$.

### 1.7. Fase 7: Dependency-Ordered Task Breakdown
- **Tujuan**: Memecah implementasi menjadi urutan tugas teknis yang dapat dieksekusi secara berurutan (*chronological execution graph*).
- **Keterampilan Direkomendasikan**:
  - `atlas` & `execute-task`: Memandu transisi fase kerja:
    - *Fase 1*: Database & Migrasi Goose PostgreSQL + kueri sqlc.
    - *Fase 2*: Domain Entities & Repository Interfaces.
    - *Fase 3*: Usecase Logic & Validasi Bisnis.
    - *Fase 4*: HTTP Handlers & Gin Route Registration.
    - *Fase 5*: SolidJS UI Views, Components & UnoCSS.
    - *Fase 6*: QA Verification, Unit & Browser Testing.

---

## 2. Arsitektur Dokumen Spesifikasi: Master (`xxx-*.md`) & Sub-Modul (`xxxy-*.md`)

Untuk mengoptimalkan context window agen AI, mencegah kebingungan referensi, dan menjaga kerapian cetak biru, dokumentasi spesifikasi SewERP mengadopsi **Arsitektur Spesifikasi Dua Tingkat (Two-Tier Specification System)**:

### 2.1. Tier 1: Master Specification Hub (`docs/specs/xxx-{name requirement}.md`)
1. **Peran Dokumen**:
   - Menjadi **dokumen induk / payung arsitektur** untuk setiap modul major `xxx` (misal: `001-dashboard.md`, `003-sales-order.md`).
   - Merupakan **satu-satunya tingkatan spesifikasi yang dicantumkan pada `docs/README.md`** agar katalog utama tetap ringkas, bersih, dan mudah dipahami.
2. **Kandungan Wajib Dokumen Master**:
   - Gambaran umum domain dan *bounded context*.
   - Alur proses bisnis global & diagram state machine (jika ada).
   - Aturan bisnis payung yang mengikat seluruh sub-modul.
   - **Tabel Indeks Navigasi Sub-Modul**: Daftar lengkap seluruh sub-spesifikasi `xxxy-{name}.md` yang dinaungi beserta tautan langsungnya.

### 2.2. Tier 2: Sub-Spesifikasi Fungsional Detail (`docs/specs/xxxy-{name requirement}.md`)
1. **Format Penamaan**:
   ```
   docs/specs/xxxy-{name requirement}.md
   ```
   - `xxx` (3 digit angka): Modul Major (`001` s/d `011`).
   - `y` (1 huruf kecil alfabet: `a`, `b`, `c`, `d`, ...): Sub-modul atau unit requirement spesifik.
   - `{name requirement}`: Deskripsi singkat fitur dalam format lowercase kebab-case.
2. **Batasan Ukuran Berkas**:
   - Berkas sub-spesifikasi dirancang berukuran **3 KB hingga 12 KB** (rata-rata 100–300 baris markdown).
   - Ukuran ringkas ini memastikan agen AI hanya mengonsumsi sedikit token inferensi saat mengimplementasikan fitur spesifik tanpa perlu memuat seluruh dokumen induk.
   - Jika suatu sub-spesifikasi membengkak melebihi 15 KB, sub-spesifikasi tersebut wajib dipecah menjadi huruf sub-modul berikutnya (`y+1`).
3. **Indeks & Tautan**:
   - Seluruh sub-spesifikasi diindeks secara eksklusif di dalam dokumen Master masing-masing (tidak diekspos di `docs/README.md`).
   - Setiap berkas sub-spesifikasi wajib mencantumkan tautan balik ke modul Master induknya.

---

## 3. Struktur Standar Dokumen Sub-Spesifikasi

Setiap berkas spesifikasi di `docs/specs/xxxy-{name}.md` **WAJIB** mengikuti susunan bab berikut:

```markdown
# Spesifikasi Sub-Modul: xxxy - [Nama Sub-Modul Deskriptif]

[Ringkasan 1-2 paragraf mengenai peran sub-modul ini dalam operasional pabrik]

---

## 1. Metadata & Tautan Referensi Terkait

- **Kode Modul**: `SPEC-xxxy`
- **Domain Bounded Context**: `[crm | oms | mes | wms | payroll | kms | analytics | iam]`
- **Tautan Standar Acuan Global**:
  - 📋 [Standar Penulisan Requirement](../context/requirements_standard.md)
  - 🏛️ [Arsitektur Sistem & Presisi Finansial](../context/architecture.md)
  - ⚙️ [Standar Rekayasa Backend Go & Zero Alloc](../context/backend.md)
  - 🖥️ [Standar Frontend SolidJS Ultra-Strict](../context/frontend.md)
  - 🗄️ [Standar Database PostgreSQL & Indexing](../context/dbadmin.md)
  - 🎨 [Standar Desain Antarmuka Flat UI](../context/ui.md)
  - 👆 [Prinsip UX Lapangan Pabrik](../context/ux.md)
  - 🛡️ [Standar Keamanan, PII & RLS](../context/security.md)
  - 📋 [Kebutuhan Bisnis Konveksi](../context/business_requirements.md)
  - 🧪 [Standar QA & Benchmark Testing](../context/qa.md)
  - 🔧 [Standar Kebutuhan Teknis & Cetak Thermal](../context/technical_requirements.md)
- **Tautan Spesifikasi Fitur Terkait**:
  - [Kode - Nama Fitur Relatif](./xxxy-nama-fitur.md)

---

## 2. Ringkasan Kebutuhan & Masalah Bisnis
- **Masalah Lapangan**: [Kendala operasional konkret di workshop]
- **Solusi Fitur**: [Solusi teknis yang disediakan]
- **Aturan Bisnis & Invariants**: [Batasan mutlak yang harus ditaati]

---

## 3. User Stories (Prioritas P1 / P2 / P3)
- **US-xxx-01 [P1]**: Sebagai [Peran], saya ingin [Aksi] agar [Manfaat].

---

## 4. Kriteria Penerimaan (Acceptance Criteria - Gherkin)
### Skenario 1: [Nama Skenario Happy Path]
- **Given** [Kondisi awal valid]
- **When** [Aksi dipicu]
- **Then** [Hasil sukses yang diharapkan]

### Skenario 2: [Nama Skenario Validasi / Invariant Violation]
- **Given** [Kondisi dengan anomali]
- **When** [Aksi dipicu]
- **Then** [Sistem menolak dengan pesan error RFC 7807 yang sesuai]

---

## 5. Skema Relasi Data Granular (Mermaid ERD)
[Irisan tabel yang relevan dengan tipe data PostgreSQL dan multi-tenant tenant_id]

---

## 6. Kontrak API RESTful
[Endpoint path DDD, headers, request body JSON, response payload ApiResponse<T>]

---

## 7. Desain Antarmuka Pengguna & Format Cetak (Bila Ada)
[Layout Flat UI, warna Matcha/Sakura, touch targets, format thermal/A4]

---

## 8. Persyaratan Keamanan & Non-Fungsional
[Hak akses RBAC, Level PII, target response time, Zero Alloc Go/SolidJS]

---

## 9. Kerangka Pemecahan Tugas (Dependency-Ordered Task Breakdown)
- [ ] **Fase 1: Lapisan Database & Migrasi**
- [ ] **Fase 2: Domain Entities & Repository**
- [ ] **Fase 3: Layanan Bisnis & Usecases**
- [ ] **Fase 4: Delivery HTTP & Endpoint API**
- [ ] **Fase 5: Frontend SolidJS & Komponen UI**
- [ ] **Fase 6: Pengujian QA, Benchmark & Aksesibilitas**
```

---

## 4. Matriks Rekomendasi Skills untuk Penulisan Requirement

| Tahapan Spesifikasi | Keterampilan Utama (*Primary Skills*) | Keterampilan Pendukung (*Secondary Skills*) |
| :--- | :--- | :--- |
| **Penggalian Kebutuhan & Intent** | `intent-debugger`, `problem-statement` | `proto-persona`, `jobs-to-be-done`, `discovery-process` |
| **PRD & Batasan Bisnis** | `prd-development`, `prd-taskmaster` | `spec-kit`, `derisk-measurement-advisor` |
| **User Story & Acceptance** | `user-story`, `user-story-splitting` | `user-story-mapping` |
| **Model Data & REST API** | `erd-api-contract` | `codegraph`, `spec-kit` |
| **Antarmuka & Ergonomi** | `ui-ux-pro-max`, `a11y-debugging` | `generative_ui`, `customer-journey-map` |
| **Keamanan & Privasi Data** | `infosec`, `cookie-debugging` | `go-vulncheck`, `autonomous-investigation` |
| **Kinerja & Non-Fungsional** | `memory-leak-debugging`, `debug-optimize-lcp` | `troubleshooting`, `chrome-devtools` |
| **Pemecahan & Eksekusi Tugas**| `execute-task`, `atlas` | `spec-kit` |
