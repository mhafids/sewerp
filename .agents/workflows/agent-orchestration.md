---
name: agent-orchestration
description: Alur orkestrasi multi-agen SewERP end-to-end dari ide mentah klien hingga rilis produksi, mencakup Discovery, Kontrak Arsitektur, Quality Gate DoR, Eksekusi Paralel (DB, FE, QA), Implementasi BE, Code Review, E2E QA, dan Quality Gate DoD.
argument-hint: "<fitur, modul konveksi, atau ide klien yang akan dikembangkan>"
uses:
  - agent-orchestration-advisor
  - atlas
  - execute-task
  - spec-kit
  - erd-api-contract
  - intent-debugger
  - ui-ux-pro-max
  - infosec
  - chrome-devtools
  - codegraph
outputs:
  - Dokumen Spesifikasi Modular (docs/specs/xxxy-{name}.md)
  - Kontrak Arsitektur Bersama (docs/memory/shareds/architect-xxxy-*.md)
  - Keputusan Gate DoR & DoD (docs/memory/shareds/orchestrator-xxxy-*.md)
  - Artefak Eksekusi Paralel (Database, Backend, Frontend, QA, Review)
---

# Workflow: End-to-End Flexible Agent Delivery Orchestration

Workflow tunggal ini merumuskan orkestrasi multi-agen rekayasa perangkat lunak adaptif untuk ekosistem **SewERP**. Mengubah ide mentah konveksi/garmen menjadi perangkat lunak produksi yang terverifikasi melalui mekanisme **Dynamic Role Selection**, **Contract-First Parallelism**, **Quality Gates Adaptif (DoR & DoD)**, dan **Alur Memori 4-Tahap**.

Dokumen ini merujuk pada hierarki tim resmi di [`docs/agent-profile/README.md`](../../docs/agent-profile/README.md) dan standar spesifikasi modular di [`docs/context/requirements_standard.md`](../../docs/context/requirements_standard.md).

---

## 0. Dynamic Role Selection & Adaptive Pod Matrix

Untuk mencegah **Over-Orchestrating Simple Tasks** (anti-pattern) dan pemborosan token, alur kerja **TIDAK mewajibkan seluruh 8 tahapan monolitik diterapkan pada setiap permintaan**.

Setiap permintaan/tugas pertama kali dievaluasi oleh **`ROLE-ORCH-01: Lead Delivery Orchestrator`** untuk menentukan **Active Agent Pod** yang paling ramping dan efektif.

### A. Diagram Keputusan Seleksi Peran Dinamis
```
[ User Requirement / Task ]
             │
             ▼
[ ROLE-ORCH-01: Lead Delivery Orchestrator ]
(Evaluasi Lingkup & Tentukan Active Pod)
             │
 ┌───────────┼───────────────────────────┬───────────────────────────┐
 ▼                           ▼                           ▼           ▼
[ Pod UI Prototype ]        [ Pod Backend Core ]        [ Pod Full Enterprise ]     [ Custom Dynamic Pod ]
• ORCH-01 (Gatekeeper)      • ORCH-01 (Gatekeeper)      • Seluruh Peran 01 - 11     • Disesuaikan bebas
• FE-SPEC-08 (Frontend)     • ARCH-03 (Data Architect)  • Full 8-Stage DDD Monolith   sesuai requirement
• QA-10 / CR-09 (Review)    • DBA-04 (Goose / sqlc)
                            • BE-SPEC-06 (Go Gin / Engine)
                            • CR-09 (Reviewer)
```

### B. Rubrik Seleksi & Pembatalan Peran (Bypass Rules)

| Domain Kebutuhan | Peran yang Diaktifkan | Keahlian Kunci | Kapan Wajib Dilewati (*Skipped*)? |
| :--- | :--- | :--- | :--- |
| **Ide ambigu / Kebutuhan bisnis mentah** | `ROLE-REQ-02` (Product Analyst) | `intent-debugger`, `user-story` | Dilewati jika instruksi/tugas teknis sudah sangat jelas dan spesifik. |
| **Relasi tabel baru / Desain API DDD** | `ROLE-ARCH-03` (Lead Architect) | `spec-kit`, `erd-api-contract` | Dilewati jika tidak ada perubahan arsitektur data atau REST contract baru. |
| **Tabel DB, Goose SQL, kueri sqlc** | `ROLE-DBA-04` (Database Specialist) | PostgreSQL 16, UUIDv7 | Dilewati jika tugas tidak memerlukan penyimpanan persistensi database. |
| **Endpoint Go, Gin, Zero-alloc engine** | `ROLE-BE-SPEC-06` (Backend Eng) | Go Gin, zero-alloc benchmark | Dilewati jika tugas adalah frontend-only, prototipe statis, atau dokumentasi. |
| **Visual UI, HTML/CSS, SolidJS views** | `ROLE-FE-SPEC-08` (Frontend Eng) | `ui-ux-pro-max`, Flat UI | Dilewati jika tugas bersifat backend core, script CLI, atau database murni. |
| **Audit kode, memory leak, security** | `ROLE-CR-09` (Code Reviewer) | Maker-checker, token checks | Wajib untuk kode produksi, dapat disatukan dengan QA untuk tugas ringan. |
| **Validasi browser, A11y, W3C, E2E** | `ROLE-QA-10` (Lead QA Eng) | `chrome-devtools`, A11y | Disesuaikan: E2E Playwright untuk flow kritis, linter/visual test untuk static. |
| **Container Distroless, SSL, UU PDP** | `ROLE-OPS-11` (DevOps & Security) | Container, Nginx, UU PDP | Dilewati jika tidak ada perubahan infrastruktur atau rilis container. |

### C. Dynamic Quality Gates (DoR & DoD Adaptif)
- **Dynamic Definition of Ready (DoR)**:
  - DoR hanya memvalidasi prasyarat peran yang **aktif** di dalam pod.
  - *Contoh*: Jika `ROLE-DBA-04` dan `ROLE-BE-SPEC-06` dilewati, DoR **TIDAK BOLEH** menolak tugas hanya karena tidak ada skema migrasi SQL atau DTO Go.
- **Dynamic Definition of Done (DoD)**:
  - DoD hanya mensyaratkan verifikasi subtask dari peran yang aktif di dalam pod.

---

## 1. Tahapan Eksekusi Agen (Conditional Stage Directives)

Setiap tahap di bawah ini bersifat **kondisional**. Tahap yang tidak melibatkan peran aktif akan otomatis dilewati (*bypassed*) ke tahap berikutnya.

### Tahap 1: Penggalian Kebutuhan & Spesifikasi Modular `[Kondisional: ROLE-REQ-02]`
- **Pelaksana Utama**: `ROLE-REQ-02: Lead Product Analyst` (didukung `ROLE-PM-00`).
- **Kondisi Aktivasi**: Requirement berupa ide bisnis baru, keluhan pabrik konveksi yang belum terstruktur, atau fitur besar baru.
- **Instruksi**:
  1. Analisis ide menggunakan `intent-debugger`: Deteksi asumsi implisit, istilah ambigu, dan aktor konveksi.
  2. Susun PRD dan Gherkin 4-Pilar di `docs/specs/xxxy-{name}.md`.
- *Jika Dilewati*: Langsung lanjut ke Tahap 2 atau Tahap 3.

### Tahap 2: Perancangan Arsitektur Kontrak `[Kondisional: ROLE-ARCH-03]`
- **Pelaksana Utama**: `ROLE-ARCH-03: Lead System Architect`.
- **Kondisi Aktivasi**: Memerlukan pembuatan/perubahan model data relasional atau kontrak REST API baru.
- **Instruksi**:
  1. Buat Mermaid `erDiagram` (UUIDv7, BIGINT zero-float, `rate_bps`, `tenant_id`).
  2. Buat Go DTO struct & Zod schema, lalu terbitkan `docs/memory/shareds/architect-xxxy-{name}.md`.
- *Jika Dilewati*: Langsung lanjut ke Tahap 3.

### Tahap 3: Quality Gate: Definition of Ready (DoR) & Task Graph `[Wajib: ROLE-ORCH-01]`
- **Pelaksana Utama**: `ROLE-ORCH-01: Lead Delivery Orchestrator`.
- **Kondisi Aktivasi**: **WAJIB** untuk semua jenis tugas.
- **Instruksi**:
  1. **Deklarasi Pod Aktif**: Nyatakan peran aktif dan peran yang di-*skip* beserta alasan teknisnya.
  2. **Audit Dynamic DoR**: Verifikasi kriteria kesiapan sesuai pod terpilih.
  3. **Penyusunan Task Graph**: Pecah subtask yang hanya melibatkan peran aktif.
  4. Terbitkan deklarasi di `docs/memory/shareds/orchestrator-xxxy-{name}.md` dengan status `[DOR_APPROVED]`.

### Tahap 4: Eksekusi Paralel `[Kondisional: Sesuai Peran Aktif]`
Agen-agen yang aktif di pod bekerja secara simultan tanpa saling memblokir:
- **Jalur Database (`ROLE-DBA-04`)**: Jika aktif, tulis Goose migrations dan jalankan `sqlc generate` pgx/v5.
- **Jalur Frontend (`ROLE-FE-SPEC-08`)**: Jika aktif, susun tampilan (SolidJS/HTML statis), styling Flat UI (Matcha `#5B7E5F` & Sakura `#E8A598`), dan validasi input.
- **Jalur QA (`ROLE-QA-10`)**: Jika aktif, siapkan skenario pengujian Playwright atau rubrik audit kesesuaian.

### Tahap 5: Implementasi Backend & Zero-Alloc Engine `[Kondisional: ROLE-BE-SPEC-06]`
- **Pelaksana Utama**: `ROLE-BE-SPEC-06: Go Backend Engineer` (diawasi `ROLE-BE-LEAD-05`).
- **Kondisi Aktivasi**: Tugas memerlukan business logic, kalkulasi harga/susut borongan, atau Gin HTTP handlers.
- **Instruksi**:
  1. Implementasikan domain usecase dan repo sqlc.
  2. Buktikan ambang batas **`0 B/op` & `0 allocs/op`** pada kalkulasi krusial.
  3. Terbitkan log di `docs/memory/shareds/backend-xxxy-{name}.md`.

### Tahap 6: Sinkronisasi & Dedicated Code Review `[Kondisional: ROLE-CR-09]`
- **Pelaksana Utama**: `ROLE-CR-09: Dedicated Code Reviewer`.
- **Kondisi Aktivasi**: Kode ditujukan untuk produksi atau modul inti.
- **Instruksi**:
  1. Audit kesesuaian tipe, zero memory leak (`onCleanup`), integritas zero-float, dan masking PII pada Zerolog.
  2. Terbitkan keputusan di `docs/memory/shareds/code-review-xxxy-{name}.md` (`[APPROVED | CHANGES_REQUESTED]`).

### Tahap 7: Validasi Riil QA & Pengujian Performa `[Kondisional: ROLE-QA-10]`
- **Pelaksana Utama**: `ROLE-QA-10: Lead QA & Test Engineer`.
- **Kondisi Aktivasi**: Ada artefak antarmuka pengguna, API aktif, atau alur transaksi.
- **Instruksi**:
  1. Uji halaman/komponen: responsivitas tampilan, standar W3C/A11y, dan DevTools (zero detached DOM nodes).
  2. Terbitkan laporan pengujian di `docs/memory/shareds/qa-xxxy-{name}.md`.

### Tahap 8: Audit Keamanan, Quality Gate DoD & Rilis `[Wajib: ROLE-ORCH-01 & PM-00]`
- **Pelaksana Utama**: `ROLE-ORCH-01: Lead Orchestrator` & `ROLE-PM-00: Project Manager` (didukung `ROLE-OPS-11` jika aktif).
- **Instruksi**:
  1. Verifikasi bahwa semua deliverable peran aktif telah tuntas dan valid.
  2. Terbitkan keputusan DoD di `docs/memory/shareds/orchestrator-xxxy-{name}.md` dengan status `[DOD_VERIFIED]`.
  3. `ROLE-PM-00` memberikan final sign-off penyelesaian.

---

## 2. Protokol Memori 4-Tahap (*Token-Efficient Lifecycle*)

Setiap agen yang terlibat dalam workflow ini wajib mematuhi alur memori 4-tahap untuk menjamin konsistensi konteks antar-terminal tanpa pemborosan token (*anti-context stuffing*):

$$\text{Requirement} \longrightarrow \text{Context} \longrightarrow \text{Long Memory} \longrightarrow \text{Shareds}$$

1. **Step 1 - Requirement (`docs/specs/xxxy-{name}.md`)**:
   Membaca spesifikasi modular target untuk memahami batasan fungsional dan kriteria penerimaan.
2. **Step 2 - Context Curated Selection (`docs/context/{domain}.md`)**:
   **Hanya muat dokumen konteks yang relevan** dengan peran agen (misal: Backend hanya membaca `backend.md` dan `forms.md`; Frontend hanya membaca `frontend.md` dan `ui.md`). Dilarang memuat seluruh isi `docs/context/`.
3. **Step 3 - Long Memory (`docs/memory/long-memory/{xxxy}-invariants.md`)**:
   Membaca keputusan arsitektur permanen, konvensi penamaan tabel, dan aturan bisnis domain yang telah disepakati sebelumnya.
4. **Step 4 - Upstream Shared Memory (`docs/memory/shareds/{upstream-agent}-xxxy-*.md`)**:
   Membaca artefak masukan dari agen sebelumnya (contoh: Backend membaca memori dari Architect dan DBA) dan menuliskan artefak keluarannya ke `docs/memory/shareds/{my-role}-xxxy-*.md`.

---

## 3. Template Struktur Blueprint Spesifikasi (Standar Tahap 1-3)

Setiap spesifikasi baru yang dihasilkan dari alur ini wajib ditulis ke `docs/specs/xxxy-{name requirement}.md` dengan menggunakan struktur blueprint 9-bagian berikut:

```markdown
# Specification & Implementation Blueprint: [Nama Fitur/Modul]

## 1. Ide Client & Context Summary
[Ringkasan ide mentah klien, latar belakang operasional konveksi, dan masalah inti]

## 2. Requirement Discovery & Intent Clarification
- **Confirmed Assumptions**: [Asumsi yang telah dikonfirmasi]
- **Identified Actors & Personas**: [Aktor konveksi yang terlibat: Owner, Kasir, Mandor, dll.]
- **Unresolved Edge Cases / Consequential Nuances**: [Kasus batas dan nuansa aturan bisnis]
- **Boundary Conditions & Exclusions**: [Batasan apa yang TIDAK dibuat]

## 3. Business Requirements
- **Business Objectives & Value**: [Tujuan bisnis & efisiensi yang dicapai]
- **Key Performance Indicators (KPIs)**: [Metrik keberhasilan terukur]
- **Business Rules & Compliance**: [Aturan konveksi, zero-float, UU PDP]
- **Scope (In-Scope vs. Out-of-Scope)**: [Cakupan fitur]

## 4. Product Requirements Document (PRD)
- **Problem Statement**: [Pernyataan masalah pengguna]
- **Target Users & Workflows**: [Alur kerja di lantai pabrik/toko]
- **Core Functional Requirements**:
  - FR-1: [...]
  - FR-2: [...]
- **Non-Functional Requirements**:
  - Multi-tenancy isolation (`tenant_id`)
  - Concurrency & data integrity
  - Security, zero-alloc benchmark, telemetry & audit logging

## 5. User Stories (Prioritized P1, P2, P3)
- **US-01 [P1]**: As a [persona], I want [capability] so that [benefit].
- **US-02 [P2]**: As a [persona], I want [capability] so that [benefit].

## 6. Acceptance Criteria (Gherkin 4-Pilar Format)
### Scenario 1: [Happy Path]
- **Given** [kondisi awal]
- **When** [aksi pengguna]
- **Then** [hasil yang diharapkan]

### Scenario 2: [Validation / Edge Case]
- **Given** [kondisi awal]
- **When** [input tidak valid]
- **Then** [pesan error RFC 7807 sesuai domain]

## 7. Entity Relationship Diagram (ERD)
```mermaid
erDiagram
    ...
```
- **Table Schema Details & Indexes**: [Rincian kolom, tipe PostgreSQL, UUIDv7, indeks compound]

## 8. API Contract Specifications
### Endpoint: `[METHOD] /api/v1/{domain}/...` (contoh: `POST /api/v1/oms/orders`)
- **Permissions**: `rbac:domain:action`
- **Request Headers / Query / Body (dengan validasi struct & custom enum)**:
```json
...
```
- **Response Payloads (Success & RFC 7807 Error)**:
```json
...
```

## 9. Dependency-Ordered Task Breakdown
- [ ] **Phase 1: Database & Migrations**
  - [ ] Task 1.1: Goose PostgreSQL schema migration (`backend/migrations/`) & UUIDv7
  - [ ] Task 1.2: sqlc queries definition & code generation (`sqlc generate` pgx/v5)
- [ ] **Phase 2: Domain Entities & Repositories**
  - [ ] Task 2.1: Domain models with zero-float monetary standards (`BIGINT` + `rate_bps`)
  - [ ] Task 2.2: Repository implementation wrapping sqlc queries & pgxpool
- [ ] **Phase 3: Usecases & Business Logic**
  - [ ] Task 3.1: Service logic, validation & calculation engine (0 B/op & 0 allocs/op)
  - [ ] Task 3.2: Asynchronous River Queue background worker (jika diperlukan)
- [ ] **Phase 4: API Deliveries & Routes**
  - [ ] Task 4.1: HTTP Handlers, Gin route group registration, dan custom validator enum DTO
  - [ ] Task 4.2: Auth & tenant middleware integration
- [ ] **Phase 5: Frontend Integration & UI**
  - [ ] Task 5.1: SolidJS views, `@solidjs/router`, UnoCSS Flat UI (`@kobalte/core`)
  - [ ] Task 5.2: `@tanstack/solid-query` hooks & `modular-forms` + `zod` strict validation
- [ ] **Phase 6: Quality Assurance & Verification**
  - [ ] Task 6.1: Playwright integration test & Chrome DevTools zero detached DOM audit
```

---

## 4. Panduan Eksekusi CLI Multi-Terminal Baku & Dynamic Model Routing

Sesuai [`.agents/rules/dynamic-model-dispatch.md`](../rules/dynamic-model-dispatch.md), Lead Agent (`ROLE-ORCH-01`) **DILARANG** menulis kode spesialis di percakapan utama dan **WAJIB** mengeksekusi peran melalui subprocess CLI dengan model yang dipilih secara dinamis.

### A. Matriks Pemilihan Model & Engine Router Otomatis

| Tingkat Kognitif | Peran Target | CLI Engine | Perintah Eksekusi Baku (Model Primary & Fallback) |
| :--- | :--- | :---: | :--- |
| **Tier 1: Reasoning & Audit** | `ROLE-ARCH-03`<br/>`ROLE-CR-09`<br/>`ROLE-OPS-11` | **`opencode`**<br/>(Fallback: `agy`) | `opencode run --model "deepseek/deepseek-r1" --auto "..."`<br/>*Fallback*: `agy --dangerously-skip-permissions --model "claude-sonnet-4-6" --print "..."` |
| **Tier 2: Code Generation** | `ROLE-DBA-04`<br/>`ROLE-BE-SPEC-06`<br/>`ROLE-FE-SPEC-08`<br/>`ROLE-QA-10` | **`opencode`**<br/>(Fallback: `agy`) | `opencode run --model "deepseek/deepseek-v3" --auto "..."`<br/>*DBA Alternative*: `opencode run --model "qwen/qwen-2.5-coder-32b" --auto "..."`<br/>*Fallback*: `agy --dangerously-skip-permissions --model "gemini-3.8-flash-high" --print "..."` |
| **Tier 3: Mass Context & PRD** | `ROLE-REQ-02`<br/>`ROLE-PM-00` | **`agy`**<br/>(Fallback: `opencode`) | `agy --dangerously-skip-permissions --model "gemini-3.8-flash-high" --print "..."`<br/>*Fallback*: `opencode run --model "deepseek/deepseek-v3" --auto "..."` |

### B. Format Baku Template Direktif Prompt Agen CLI

```text
=== AGENT EXECUTION DIRECTIVE ===
[ROLE]: [Kode Peran & Nama Peran, misal: ROLE-FE-SPEC-08: Frontend Specialist]
[REQUIRED SKILLS]: [Daftar keahlian wajib dari matriks rekomendasi]

=== 4-STEP CONTEXT & MEMORY INJECTION (CURATED ONLY) ===
1. [TARGET REQUIREMENT]: docs/specs/[xxxy-name].md
2. [RELEVANT CONTEXT]: docs/context/[relevant_domain].md (HANYA muat yang relevan)
3. [LONG MEMORY]: docs/memory/long-memory/[xxxy-title].md
4. [UPSTREAM SHARED MEMORY]: docs/memory/shareds/[upstream_agent]-[xxxy-title].md

=== PRE-PROMPT TOKEN SUMMARY & NON-NEGOTIABLE INVARIANTS ===
- Currency: Wajib BIGINT integer (zero-float). Pajak/Diskon: rate_bps.
- Identity: Primary Key wajib time-ordered UUIDv7.
- Tenancy: Seluruh tabel dan kueri wajib menyertakan tenant_id.
- Validation: Strict DTO validation (tanpa loose type, tanpa 'any', tanpa hardcoded 'oneof').
- Performance: Hot path perhitungan wajib 0 B/op dan 0 allocs/op. Frontend wajib onCleanup().

=== TASK INSTRUCTIONS ===
[Uraian instruksi spesifik dan tindakan yang harus dilakukan agen]

=== DELIVERABLES & APPROVAL DECISION OUTPUT ===
1. Publikasikan artefak luaran ke: docs/memory/shareds/[my_role]-[xxxy-title].md
2. Wajib tutup respons dengan blok keputusan formal:
   - Status: [APPROVED | CHANGES_REQUESTED | REJECTED]
   - Invariants Verified: [Daftar invarian yang telah diuji valid]
   - Downstream Handoff: [Instruksi penyerahan tugas untuk agen berikutnya di pipa paralel]
```
