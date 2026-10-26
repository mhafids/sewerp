# Sistem Memori Persisten & Shared Memory SewERP

Dokumen ini mendefinisikan protokol penyimpanan memori jangka panjang (*Long-Term Memory*) dan pertukaran memori kolaboratif antar-agen (*Cross-Agent Shared Memory*) untuk pengembangan sistem **SewERP**.

---

## 1. Prinsip Utama: Alur Kerja 4-Tahap (4-Step Memory Workflow)

Untuk mencegah halusinasi, menjaga konsistensi arsitektur, dan memfasilitasi kerja paralel antar-agen, setiap pemanggilan agen melalui **Antigravity CLI (`agy`)** atau **OpenCode (`opencode`)** wajib mematuhi alur pembacaan dan penulisan 4-tahap:

$$\text{Requirement} \longrightarrow \text{Context} \longrightarrow \text{Long Memory} \longrightarrow \text{Shareds}$$

```
┌────────────────────────────────────────────────────────────────────────┐
│ TAHAP 1: REQUIREMENT (Spesifikasi Sub-Fitur Target)                    │
│ ↳ docs/specs/xxxy-{name requirement}.md                               │
│ Membaca lingkup bisnis, Gherkin 4-pilar, kontrak data, dan kriteria UI.│
├────────────────────────────────────────────────────────────────────────┤
│ TAHAP 2: CONTEXT (Standar Rekayasa Global)                             │
│ ↳ docs/context/{domain}.md                                             │
│ Membaca aturan invariant (Zero-Float BIGINT, UUIDv7, onCleanup, dll).  │
├────────────────────────────────────────────────────────────────────────┤
│ TAHAP 3: LONG MEMORY (Memori Jangka Panjang Internal Agen)            │
│ ↳ docs/memory/long-memory/{xxxy}-{title of memory}.md                  │
│ Membaca/menulis status modul, keputusan domain (ADR), & edge cases.    │
├────────────────────────────────────────────────────────────────────────┤
│ TAHAP 4: SHAREDS (Memori Kolaborasi & Handoff Antar-Agen)             │
│ ↳ docs/memory/shareds/{agent from}-{xxxy}-{title of requirement}.md   │
│ Membaca masukan agen upstream & mempublikasikan artefak ke downstream. │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Struktur Direktori & Konvensi Penamaan

```text
docs/memory/
├── README.md                      # Panduan protokol memori ini
├── long-memory/                   # Memori internal jangka panjang agen per sub-modul
│   └── {xxxy}-{title of memory}.md
└── shareds/                       # Memori pertukaran/handoff antar-agen
    └── {agent from}-{xxxy}-{title of requirement}.md
```

### A. Konvensi Berkas `long-memory/`
- **Format**: `{xxxy}-{title of memory}.md`
  - `xxx`: 3-digit kode modul major (misal: `001` Dashboard, `003` Sales Order, `004` Produksi, `006` Payroll).
  - `y`: Huruf alfabet sub-modul (misal: `a`, `b`, `c`).
  - `{title of memory}`: Deskripsi singkat aspek teknis/domain yang dipertahankan.
- **Contoh Berkas**:
  - `docs/memory/long-memory/003a-sales-order-invariants.md` (Menyimpan batasan invariant DP, validasi toleransi susut kain, dan aturan surcharge ukuran jumbo).
  - `docs/memory/long-memory/004a-cutting-station-rules.md` (Menyimpan aturan anti-overclaim hasil potong dan alokasi kain per rol).
  - `docs/memory/long-memory/006b-piece-rate-tariffs.md` (Menyimpan riwayat pemotongan kasbon mingguan dan tarif borongan per pcs).

### B. Konvensi Berkas `shareds/`
- **Format**: `{agent from}-{xxxy}-{title of requirement}.md`
  - `{agent from}`: Identifier peran agen yang mempublikasikan memori (`pm`, `req-lead`, `architect`, `db`, `backend`, `frontend`, `code-review`, `qa`, `security`, `devops`).
  - `{xxxy}`: Nomor sequence sub-modul target.
  - `{title of requirement}`: Nama fitur sesuai nama spesifikasi.
- **Contoh Berkas**:
  - `architect-003a-sales-order-core.md`: Publikasi ERD, kontrak DTO API, dan UUIDv7 index untuk dikonsumsi oleh `database`, `backend`, `frontend`, dan `qa`.
  - `database-003a-sales-order-core.md`: Publikasi versi migrasi goose yang telah dibuat dan daftar fungsi kueri sqlc untuk dikonsumsi oleh `backend`.
  - `backend-003a-sales-order-core.md`: Publikasi endpoint aktif, custom validation tags, dan payload response untuk dikonsumsi oleh `code-review`, `frontend`, dan `qa`.
  - `code-review-003a-sales-order-core.md`: Publikasi hasil verifikasi benchmark zero-alloc (`0 B/op`) dan audit clean architecture untuk dikonsumsi oleh `lead-backend`, `qa`, dan `pm`.
  - `frontend-003a-sales-order-core.md`: Publikasi view route SolidJS, token UnoCSS, dan status validasi Zod untuk dikonsumsi oleh `qa`.
  - `qa-003a-sales-order-core.md`: Publikasi hasil pengujian unit/integrasi/E2E, performa browser, dan tiket bug untuk dikonsumsi oleh `pm` dan para *engineers*.

---

## 3. Template Standar Penulisan Memori

### A. Template Berkas `long-memory/`
```markdown
# Long Memory: [{xxxy}] {Title of Feature}

- **Modul**: `docs/specs/{xxxy}-{name}.md`
- **Terakhir Diperbarui**: YYYY-MM-DD oleh [{Agent Role}]
- **Domain Context**: `{crm | oms | mes | wms | payroll | kms}`

## 1. Domain Invariants & Business Rules Confirmed
- Invariant 1: [Aturan mutlak bisnis konveksi yang tidak boleh dilanggar]
- Invariant 2: [Presisi moneter BIGINT dan rate_bps]

## 2. Resolved Edge Cases
- Kasus Khusus: [Deskripsi kasus batas dan resolusi teknis yang disepakati]

## 3. Technical Constraints & ADR (Architectural Decision Records)
- Keputusan 1: [Alasan pemilihan tipe data, algoritma kalkulasi, atau indeks]
```

### B. Template Berkas `shareds/`
```markdown
# Shared Memory Handoff: [{Agent From}] ➔ [{Target Consumers}]
## Feature: [{xxxy}] {Title of Requirement}

- **Pengirim (Agent From)**: {pm | req-lead | architect | db | backend | frontend | code-review | qa | security | devops}
- **Penerima Target**: {backend, frontend, code-review, qa, devops, pm}
- **Timestamp / Status**: YYYY-MM-DD | READY_FOR_CONSUMPTION

## 1. Artefak yang Dihasilkan (Deliverables Summary)
- [Ringkasan berkas kode, migrasi skema, atau antarmuka yang selesai dibuat]

## 2. Kontrak Data / Interface / Schema
- [Tautan DTO struct Go, skema Zod, endpoint REST, atau DDL migrasi goose]

## 3. Validasi & Standar yang Telah Diverifikasi
- [x] Zero Floating-Point (BIGINT + rate_bps)
- [x] Multi-Tenancy (tenant_id UUIDv7)
- [x] Zero-Alloc / Zero-Memory Leak Verification

## 4. Instruksi untuk Agen Downstream
- **Untuk Frontend**: [Gunakan endpoint POST /api/v1/... dengan schema berikut]
- **Untuk Code Reviewer**: [Periksa fungsi internal/oms/usecase/pricing_calculator.go]
- **Untuk QA**: [Uji skenario overclaim borongan pada skenario Gherkin #3]
```

---

## 4. Cara Menjalankan Memori via Antigravity CLI (`agy`) & OpenCode (`opencode`)

Semua eksekusi via CLI wajib menggunakan format prompt terstruktur dalam bahasa Inggris untuk menghemat token dan memastikan keputusan formal (*decision block*). Panduan lengkap tersedia di [docs/agent-profile/README.md#6-official-cli-execution-guide-english-prompt-specification](../agent-profile/README.md#6-official-cli-execution-guide-english-prompt-specification).

### Contoh Antigravity CLI (`agy`) - Backend Engineer:
```bash
agy --model "deepseek/deepseek-v3" --prompt "
=== AGENT EXECUTION DIRECTIVE ===
[ROLE]: ROLE-BE-SPEC-06: Go Backend Implementation Specialist
[REQUIRED SKILLS]: erd-api-contract, codegraph, go-vulncheck

=== 4-STEP CONTEXT & MEMORY INJECTION (CURATED ONLY) ===
1. [TARGET REQUIREMENT]: docs/specs/003a-sales-order-core.md
2. [RELEVANT CONTEXT]: docs/context/backend.md, docs/context/forms.md
3. [LONG MEMORY]: docs/memory/long-memory/003a-sales-order-invariants.md
4. [UPSTREAM SHARED MEMORY]: docs/memory/shareds/architect-003a-sales-order-core.md, docs/memory/shareds/database-003a-sales-order-core.md

=== PRE-PROMPT TOKEN SUMMARY & NON-NEGOTIABLE INVARIANTS ===
- Domain: Sales Order Core creation.
- Data Rules: Strictly BIGINT zero-float; percentages in rate_bps; UUIDv7 PK.
- Performance: Hot path calculate_pricing must achieve 0 B/op and 0 allocs/op.

=== TASK & DELIVERABLES ===
Implement sales order usecase and Gin delivery handler.
Publish deliverables and benchmark evidence to docs/memory/shareds/backend-003a-sales-order-core.md with:
- Status: READY_FOR_REVIEW
- Benchmarks: BenchmarkCalculatePricing passing at 0 B/op, 0 allocs/op
- Downstream Handoff: Ready for 09-code-reviewer and 08-frontend-engineer
"
```

### Contoh OpenCode CLI (`opencode`) - QA Test Engineer:
```bash
opencode run --model "deepseek/deepseek-v3" "
=== AGENT EXECUTION DIRECTIVE ===
[ROLE]: ROLE-QA-10: Lead QA & Test Automation Specialist
[REQUIRED SKILLS]: chrome-devtools, a11y-debugging

=== 4-STEP CONTEXT & MEMORY INJECTION ===
1. [TARGET REQUIREMENT]: docs/specs/003a-sales-order-core.md
2. [RELEVANT CONTEXT]: docs/context/qa.md
3. [LONG MEMORY]: docs/memory/long-memory/003a-sales-order-invariants.md
4. [UPSTREAM SHARED MEMORY]: docs/memory/shareds/code-review-003a-sales-order-core.md, docs/memory/shareds/backend-003a-sales-order-core.md

=== TASK & APPROVAL DECISION PROTOCOL ===
Execute Playwright test suite for sales order lifecycle.
Verify thermal print layout (80mm) and minimum 48px touch targets on tablet viewport.
Publish test verdict to docs/memory/shareds/qa-003a-sales-order-core.md with Status: [QA_PASSED | QA_FAILED].
"
```
