# Panduan Tim Profil Agen & Ekosistem Rekayasa SewERP

Selamat datang di pusat arsitektur dan panduan operasional **Tim Agen SewERP**. Dokumen ini merumuskan hierarki peran tim (*Project Manager, Team Leads, dan Specialist Engineers*), model eksekusi paralel (*Contract-First Parallelism*), tolak ukur empiris model AI berbasis **LiveBench 2.0** (`https://livebench.ai/`), panduan efisiensi biaya **Pay-As-You-Go (SumoPod / Multi-Provider Proxy)**, integrasi **Model Frontier China (DeepSeek, Qwen, GLM, Kimi)**, serta pemanfaatan **Antigravity CLI (`agy`)** dan **OpenCode (`opencode`)** sebagai *official execution engines*.

---

## 1. Organigram & Hierarki Tim Agen

Sistem SewERP mengadopsi struktur tim rekayasa perangkat lunak modern berskala enterprise yang dibagi menjadi 3 tingkatan:

```
                          [ 00. Project Manager (PM) ]
                                       │
                        [ 01. Lead Delivery Orchestrator ]
                                       │
        ┌──────────────────────────────┼──────────────────────────────┐
        ▼                              ▼                              ▼
[ 02. Lead Product Analyst ]  [ 03. Lead System Architect ]  [ 11. Lead DevOps & Security ]
        │                              │                              │
        │                       [ 04. Database Specialist ]           │
        │                              │                              │
        ├──────────────────────────────┼──────────────────────────────┤
        ▼                              ▼                              ▼
[ 07. Lead Frontend Eng ]     [ 05. Lead Backend Eng ]       [ 10. Lead QA & Test Eng ]
        │                              │                              │
[ 08. Frontend Engineer ]      [ 06. Backend Engineer ]               │
        │                              │                              │
        └──────────────────────────────┼──────────────────────────────┘
                                       ▼
                          [ 09. Dedicated Code Reviewer ]
                            (Maker-Checker Gatekeeper)
```

### Klasifikasi Peran:
1. **Tier 1 - Manajemen & Pengiriman**:
   - `00-project-manager.md`: Pengendali roadmap, milestone sprint, alokasi kapasitas tim, dan pelaporan progres ke stakeholders.
   - `01-lead-orchestrator.md`: Pengendali task graph (Atlas), penjaga quality gate, dan pengelola siklus *Definition of Ready (DoR)* & *Definition of Done (DoD)*.
2. **Tier 2 - Pimpinan Rekayasa & Domain Leads**:
   - `02-lead-product-analyst.md`: Pemimpin analisis kebutuhan, penggalian keluhan konveksi, PRD, dan spesifikasi Gherkin 4-pilar.
   - `03-lead-system-architect.md`: Arsitek utama pola DDD Modular Monolith, isolasi multi-tenancy `tenant_id`, outbox River Queue, dan kontrak DTO REST API.
   - `05-lead-backend-engineer.md`: Pemimpin teknis backend Go, penjaga standar Clean Architecture dan ambang batas tolak ukur *Zero-Alloc* (`0 B/op`).
   - `07-lead-frontend-engineer.md`: Pemimpin arsitektur antarmuka SolidJS, penegak kepatuhan *TypeScript Ultra-Strict*, UnoCSS Flat UI, dan *Zero-Memory Leak*.
   - `10-lead-qa-engineer.md`: Pemimpin strategi piramida tes, otomasi Playwright & Chrome DevTools MCP, pengujian tablet pabrik, dan verifikasi cetak thermal 80mm.
   - `11-lead-devops-security.md`: Pemimpin rilis container Distroless (<30MB), Docker Compose, Nginx SSL, MinIO S3, backup otomatis, serta kepatuhan UU PDP No. 27/2022 & OWASP API.
3. **Tier 3 - Spesialis Eksekusi & Reviewer**:
   - `04-database-specialist.md`: Perancang skema PostgreSQL 16, migrasi SQL goose, kueri sqlc, UUIDv7, dan optimasi partial/compound index.
   - `06-backend-engineer.md`: Pengembang domain models, usecase orchestration, Gin delivery handler, dan custom validator enum.
   - `08-frontend-engineer.md`: Pengembang komponen views SolidJS, headless `@kobalte/core`, integrasi `modular-forms` + `zod`, dan TanStack Query.
   - `09-code-reviewer.md`: Peninjau kode independen (*Maker-Checker*), mendeteksi anti-pattern, kebocoran memori, alokasi runtime, dan pelanggaran standar keamanan.

---

## 2. Model Eksekusi Paralel (*Contract-First Parallelism*)

Agen-agen SewERP dirancang untuk dapat bekerja secara simultan (*parallel*) tanpa saling memblokir berkat mekanisme **Contract-First** yang dipertukarkan melalui berkas shared memory:

```
Fase 1: Kebutuhan Disetujui
[ Lead Product Analyst ] ➔ Terbitkan docs/specs/xxxy-{name}.md
                                    │
                                    ▼
Fase 2: Perancangan Kontrak
[ Lead System Architect ] ➔ Terbitkan docs/memory/shareds/architect-xxxy-{name}.md
                            (Memuat: Mermaid ERD, Struct Go DTO, Schema Zod, Endpoints)
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        ▼ (Paralel Eksekusi 1)      ▼ (Paralel Eksekusi 2)      ▼ (Paralel Eksekusi 3)
[ Database Specialist ]     [ Frontend Engineer ]       [ QA Test Engineer ]
• Tulis Goose Migration     • Susun SolidJS Views       • Tulis skenario Playwright
• Generate sqlc Go Queries  • Rakit Formulir Zod        • Rancang unit test boundary
• Verifikasi UUIDv7 & BPS   • Gunakan mock data DTO     • Rancang uji thermal 80mm
        │                           │                           │
        ▼                           │                           │
[ Backend Engineer ]                │                           │
• Implementasi Usecase              │                           │
• Hubungkan Repo sqlc               │                           │
• Tulis Gin HTTP Controller         │                           │
• Publikasi backend-xxxy-*.md       │                           │
        │                           │                           │
        ├───────────────────────────┴───────────────────────────┤
        ▼ (Sinkronisasi Paralel Tahap 4)                        │
[ Frontend Engineer ] ◄── Beralih dari Mock DTO ke API Asli     │
        │                                                       │
        ▼                                                       │
[ Dedicated Code Reviewer ] ◄── Audit Go & TypeScript           │
• Jalankan benchmark: 0 B/op & 0 allocs/op                      │
• Audit: Bebas tipe 'any', bebas floating-point                 │
• Publikasi code-review-xxxy-*.md                               │
        │                                                       │
        ▼                                                       │
[ Lead QA & Test Engineer ] ◄── Eksekusi E2E & Browser Tests ───┘
• Jalankan tes integrasi riil ke database PostgreSQL
• Inspeksi Heap Snapshot DevTools (Zero Detached DOM)
• Publikasi qa-xxxy-*.md
        │
        ▼
[ Lead DevOps & Security ] ➔ [ Lead Orchestrator ] ➔ [ Project Manager (Release) ]
```

---

## 3. Tolak Ukur Empiris Model AI: LiveBench 2.0 (`https://livebench.ai/`) & Biaya Pay-As-You-Go

LiveBench 2.0 adalah tolak ukur evaluasi LLM bebas kontaminasi (*contamination-free*) dengan pengujian berbasis jawaban matematis dan kode yang dapat diverifikasi secara objektif (*ground-truth verifiable*).

### Skor LiveBench 2.0 & Analisis Biaya Token (Pay-As-You-Go):

| Model AI | Provider | Biaya Input (1M Token) | Biaya Output (1M Token) | Skor Total | Coding Score | Reasoning Score | Instruction Following | Rekomendasi di SewERP |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **DeepSeek-V3** | DeepSeek | **$0.14** | **$0.28** | 82.5 | **94.7%** | 89.1% | 91.6% | **Primary Execution**: Koding Go, SolidJS, Unit Test, PM |
| **DeepSeek-R1** | DeepSeek | **$0.55** | **$2.19** | 85.4 | **96.0%** | **92.6%** | 90.8% | **Primary Reasoning**: Architect, DBA, Code Review, Security |
| **Qwen 2.5 Coder 32B** | Alibaba | **$0.15** | **$0.20** | 81.9 | **93.1%** | 88.7% | 90.5% | **Primary DBA / Alternative**: DDL SQL, sqlc, UnoCSS |
| **Gemini 2.0 Flash** | Google | **$0.10** | **$0.40** | 84.1 | 89.7% | **95.1%** | 89.4% | **Alternative Mass Context**: Membaca puluhan spek (2M window) |
| **OpenAI o3-mini (High)** | OpenAI | $1.10 | $4.40 | 86.5 | 96.3% | 92.2% | 94.2% | **Alternative Math & Code**: Formula tarif & margin susut |
| **Claude 3.7 Sonnet** | Anthropic | $3.00 | $15.00 | **88.0** | **97.7%** | 91.7% | **98.5%** | **Premium Fallback**: Inspeksi aturan negatif super ketat |
| **OpenAI o1** | OpenAI | $15.00 | $60.00 | 86.1 | 93.7% | 92.7% | 92.0% | **Premium Fallback**: Threat modeling enterprise |

---

## 4. Matriks Pemilihan Model Pay-As-You-Go (Murah & Berkinerja Tinggi)

Berdasarkan rasio performa LiveBench terhadap biaya token, berikut adalah penetapan resmi model di SewERP:

| Kode Peran | Nama Peran | Model Primary (Hemat & Juara LiveBench) | Biaya Token Est. (per 1M) | Alternatif 1 (Ultra Budget / Free) | Alternatif 2 (Premium Fallback) |
| :--- | :--- | :--- | :---: | :--- | :--- |
| **00** | **Project Manager** | **DeepSeek-V3** | $0.14 In / $0.28 Out | **Gemini 2.0 Flash** ($0.10 / $0.40) | GPT-4o / Claude 3.7 Sonnet |
| **01** | **Lead Orchestrator** | **DeepSeek-R1** | $0.55 In / $2.19 Out | **DeepSeek-V3** ($0.14 / $0.28) | OpenAI o3-mini ($1.10 / $4.40) |
| **02** | **Lead Product Analyst** | **DeepSeek-V3** / **GLM-4-Plus** | $0.14 In / $0.28 Out | **Gemini 2.0 Flash** ($0.10 / $0.40) | Claude 3.7 Sonnet |
| **03** | **Lead System Architect** | **DeepSeek-R1** | $0.55 In / $2.19 Out | **Qwen 2.5 Coder 32B** (~$0.18 / 1M) | Claude 3.7 Sonnet (Thinking) |
| **04** | **Database Specialist (DBA)** | **Qwen 2.5 Coder 32B** | ~$0.18 / 1M | **DeepSeek-V3** ($0.14 / $0.28) | OpenAI o3-mini ($1.10 / $4.40) |
| **05** | **Lead Backend Engineer** | **DeepSeek-R1** | $0.55 In / $2.19 Out | **Qwen 2.5 Coder 32B** (~$0.18 / 1M) | OpenAI o3-mini ($1.10 / $4.40) |
| **06** | **Go Backend Engineer** | **DeepSeek-V3** | $0.14 In / $0.28 Out | **Qwen 2.5 Coder 32B** (~$0.18 / 1M) | Claude 3.7 Sonnet |
| **07** | **Lead Frontend Engineer** | **DeepSeek-V3** | $0.14 In / $0.28 Out | **Qwen 2.5 Coder 32B** (~$0.18 / 1M) | Claude 3.7 Sonnet |
| **08** | **SolidJS Frontend Engineer** | **DeepSeek-V3** | $0.14 In / $0.28 Out | **Qwen 2.5 Coder 32B** (~$0.18 / 1M) | GPT-4o / Claude 3.7 Sonnet |
| **09** | **Dedicated Code Reviewer** | **DeepSeek-R1** | $0.55 In / $2.19 Out | **Qwen 2.5 Coder 32B** (~$0.18 / 1M) | Claude 3.7 Sonnet (Thinking) |
| **10** | **Lead QA & Test Engineer** | **DeepSeek-V3** | $0.14 In / $0.28 Out | **Gemini 2.0 Flash** ($0.10 / $0.40) | Claude 3.7 Sonnet |
| **11** | **Lead DevOps & Security** | **DeepSeek-R1** | $0.55 In / $2.19 Out | **Qwen 2.5 72B** (~$0.35 / 1M) | OpenAI o1 / Claude 3.7 Sonnet |

---

## 5. Panduan Finansial SumoPod Pay-As-You-Go

Bagi developer yang jarang berlangganan banyak model ($80+/bln), gunakan platform **SumoPod** atau agregator serupa:
1. **Deposit Kecil Fleksibel**: Cukup isi deposit $5 - $10.
2. **Estimasi Biaya per Sub-Fitur**:
   - Fase Desain (Architect R1): ~50k token input / 8k token output = **~$0.04**
   - Fase Database (Qwen 32B): ~30k token input / 5k token output = **~$0.01**
   - Fase Backend (DeepSeek V3): ~80k token input / 15k token output = **~$0.02**
   - Fase Frontend (DeepSeek V3): ~80k token input / 15k token output = **~$0.02**
   - Fase Review & QA (DeepSeek R1 & V3): ~70k token input / 10k token output = **~$0.04**
   - **Total Biaya 1 Fitur Penuh**: **~$0.13** (Jauh lebih murah dari $20/bulan).

---

## 6. Official CLI Execution Guide (English Prompt Specification)

To ensure token efficiency, zero context stuffing, and verifiable sign-off decisions, all agent executions via **Antigravity CLI (`agy`)** and **OpenCode CLI (`opencode`)** MUST follow the structured English prompt specification below.

### A. Pre-Prompt Token Optimization Protocol
Before dispatching a task to the LLM:
1. **Select ONLY Relevant Docs**: Never inject all 13 context documents. Only supply the 1-2 context documents relevant to that specific role.
2. **Summarize Invariants**: Provide an explicit summary block of critical invariants (e.g., zero-float, UUIDv7, onCleanup) to prime the model's working memory before execution.
3. **Approval Decision Framework**: Require the agent to conclude with a structured **Approval Decision Block** (`Status: APPROVED | CHANGES_REQUESTED | REJECTED`).

### B. Standardized Universal CLI Prompt Template

```bash
# Template for Antigravity CLI (agy)
agy --model "[PRIMARY_MODEL]" --prompt "
=== AGENT EXECUTION DIRECTIVE ===
[ROLE]: [e.g. ROLE-BE-SPEC-06: Go Backend Implementation Specialist]
[REQUIRED SKILLS]: [e.g. erd-api-contract, codegraph, go-vulncheck]

=== 4-STEP CONTEXT & MEMORY INJECTION (CURATED ONLY) ===
1. [TARGET REQUIREMENT]: docs/specs/[xxxy-name].md
2. [RELEVANT CONTEXT]: docs/context/[relevant_domain].md (DO NOT load irrelevant context files)
3. [LONG MEMORY]: docs/memory/long-memory/[xxxy-title].md
4. [UPSTREAM SHARED MEMORY]: docs/memory/shareds/[upstream_agent]-[xxxy-title].md

=== PRE-PROMPT TOKEN SUMMARY & NON-NEGOTIABLE INVARIANTS ===
- Currency: Wajib BIGINT integer (zero floating-point). Percentages: rate_bps (5% = 500 bps).
- Identity: All primary keys must be time-ordered UUIDv7.
- Tenancy: All tables and queries MUST enforce tenant_id isolation.
- Validation: Strict DTO validation without loose types (no 'any', no 'interface{}', no hardcoded 'oneof').
- Memory/Performance: Hot calculation paths must verify 0 B/op and 0 allocs/op. Frontend must unbind in onCleanup().

=== TASK INSTRUCTIONS ===
[Specific actionable task description for this role]

=== DELIVERABLES & APPROVAL DECISION OUTPUT ===
1. Publish artifact to: docs/memory/shareds/[agent_from]-[xxxy-title].md
2. Output a formal Sign-Off Block:
   - Status: [APPROVED | CHANGES_REQUESTED | REJECTED]
   - Invariants Verified: [List of verified technical constraints]
   - Downstream Handoff: [Specific instructions for next agent in the parallel pipeline]
"
```

### C. Concrete CLI Examples for Multi-Terminal Parallel Work

#### 1. Backend Engineer (Terminal 1) via OpenCode:
```bash
export OPENAI_BASE_URL="https://api.sumopod.com/v1"
export OPENAI_API_KEY="sk-sumopod-..."

opencode run --model "deepseek/deepseek-v3" "
=== AGENT EXECUTION DIRECTIVE ===
[ROLE]: ROLE-BE-SPEC-06: Go Backend Implementation Specialist
[REQUIRED SKILLS]: erd-api-contract, codegraph, go-vulncheck

=== 4-STEP CONTEXT & MEMORY INJECTION ===
1. [TARGET REQUIREMENT]: docs/specs/003a-sales-order-core.md
2. [RELEVANT CONTEXT]: docs/context/backend.md, docs/context/forms.md
3. [LONG MEMORY]: docs/memory/long-memory/003a-sales-order-invariants.md
4. [UPSTREAM SHARED MEMORY]: docs/memory/shareds/architect-003a-sales-order-core.md, docs/memory/shareds/database-003a-sales-order-core.md

=== PRE-PROMPT SUMMARY ===
Target: Implement Sales Order creation usecase and Gin handler. Enforce zero floating-point (BIGINT), rate_bps, UUIDv7, and custom validator 'order_status_enum'. Ensure 0 allocs/op in pricing arithmetic.

=== DELIVERABLES & APPROVAL DECISION OUTPUT ===
Write deliverables to backend/internal/oms/ and log results to docs/memory/shareds/backend-003a-sales-order-core.md with:
- Status: READY_FOR_REVIEW
- Benchmarks: BenchmarkCalculatePricingWithSurcharges passing at 0 B/op, 0 allocs/op
- Downstream Handoff: Ready for 09-code-reviewer and 08-frontend-engineer
"
```

#### 2. Frontend Engineer (Terminal 2) via OpenCode:
```bash
opencode run --model "deepseek/deepseek-v3" "
=== AGENT EXECUTION DIRECTIVE ===
[ROLE]: ROLE-FE-SPEC-08: SolidJS Frontend Implementation Specialist
[REQUIRED SKILLS]: ui-ux-pro-max, generative_ui, a11y-debugging

=== 4-STEP CONTEXT & MEMORY INJECTION ===
1. [TARGET REQUIREMENT]: docs/specs/003a-sales-order-core.md
2. [RELEVANT CONTEXT]: docs/context/frontend.md, docs/context/ui.md
3. [LONG MEMORY]: docs/memory/long-memory/003a-sales-order-invariants.md
4. [UPSTREAM SHARED MEMORY]: docs/memory/shareds/architect-003a-sales-order-core.md

=== PRE-PROMPT SUMMARY ===
Target: Build Sales Order view using SolidJS v1.9+, UnoCSS Flat UI (Matcha Green #5B7E5F), Kobalte primitives, and Zod strict validation. Enforce zero props destructuring and mandatory onCleanup().

=== DELIVERABLES & APPROVAL DECISION OUTPUT ===
Publish view to frontend/src/views/oms/CreateOrder.tsx and log to docs/memory/shareds/frontend-003a-sales-order-core.md with:
- Status: READY_FOR_REVIEW
- Touch Target: Validated 48px on mobile & tablet
- Downstream Handoff: Ready for 10-lead-qa-engineer E2E browser tests
"
```

#### 3. Dedicated Code Reviewer (Terminal 3) via Antigravity CLI:
```bash
agy --model "deepseek/deepseek-r1" --prompt "
=== AGENT EXECUTION DIRECTIVE ===
[ROLE]: ROLE-CR-09: Dedicated Code Reviewer (Maker-Checker Gatekeeper)
[REQUIRED SKILLS]: spec-kit, codegraph, memory-leak-debugging, go-vulncheck, infosec

=== 4-STEP CONTEXT & MEMORY INJECTION ===
1. [TARGET REQUIREMENT]: docs/specs/003a-sales-order-core.md
2. [RELEVANT CONTEXT]: docs/context/backend.md, docs/context/frontend.md, docs/context/security.md
3. [LONG MEMORY]: docs/memory/long-memory/003a-sales-order-invariants.md
4. [UPSTREAM SHARED MEMORY]: docs/memory/shareds/backend-003a-sales-order-core.md, docs/memory/shareds/frontend-003a-sales-order-core.md

=== TASK & APPROVAL DECISION PROTOCOL ===
Audit the code submitted by Backend and Frontend engineers.
Verify:
1. Go: Zero float (BIGINT + rate_bps), zero alloc benchmarks, zero 'any'/'interface{}', zero binding 'oneof'.
2. SolidJS: Zero destructuring props, onCleanup() implemented on all listeners/scanners, zero 'any'.
3. Security: PII phone number masked in Zerolog, no plaintext secrets.

Produce a formal Approval Decision in docs/memory/shareds/code-review-003a-sales-order-core.md:
- Status: [APPROVED | CHANGES_REQUESTED]
- Line-by-line inspection evidence
- Go/No-Go release verdict for 01-lead-orchestrator and 00-project-manager
"
```

---

## 7. Protokol Memori 4-Tahap (`Requirement ➔ Context ➔ Long Memory ➔ Shareds`)

Setiap agen yang dipanggil via CLI wajib membaca dan memperbarui memori secara berurutan:
1. **Requirement**: `docs/specs/xxxy-{name}.md`
2. **Context**: `docs/context/{domain}.md` (hanya domain relevan untuk menghemat token)
3. **Long Memory**: `docs/memory/long-memory/{xxxy}-{title of memory}.md`
4. **Shareds**: `docs/memory/shareds/{agent from}-{xxxy}-{title of requirement}.md`

Rincian lengkap protokol memori dan skema template tersedia di [docs/memory/README.md](../memory/README.md).

---

## 8. Matriks Lengkap Keterampilan Agen (Skills Matrix)

Keahlian terdaftar di repositori (`.agents/skills/*`) yang wajib diaktifkan per peran:

| Kode Peran | Nama Peran | Skills Wajib yang Harus Diaktifkan |
| :--- | :--- | :--- |
| **00** | **Project Manager** | `atlas`, `roadmap-planning`, `prioritization-advisor`, `business-health-diagnostic`, `stakeholder-mapping` |
| **01** | **Lead Orchestrator** | `atlas`, `execute-task`, `spec-kit`, `troubleshooting` |
| **02** | **Lead Product Analyst** | `intent-debugger`, `problem-framing-canvas`, `problem-statement`, `proto-persona`, `jobs-to-be-done`, `prd-development`, `user-story`, `user-story-mapping`, `user-story-splitting`, `derisk-measurement-advisor` |
| **03** | **Lead System Architect** | `spec-kit`, `codegraph`, `erd-api-contract`, `autonomous-investigation` |
| **04** | **Database Specialist** | `erd-api-contract`, `spec-kit`, `codegraph` |
| **05** | **Lead Backend Engineer** | `spec-kit`, `codegraph`, `go-vulncheck`, `troubleshooting` |
| **06** | **Backend Engineer** | `erd-api-contract`, `codegraph`, `go-vulncheck` |
| **07** | **Lead Frontend Engineer** | `ui-ux-pro-max`, `memory-leak-debugging`, `debug-optimize-lcp`, `spec-kit` |
| **08** | **Frontend Engineer** | `ui-ux-pro-max`, `generative_ui`, `a11y-debugging` |
| **09** | **Dedicated Code Reviewer** | `spec-kit`, `codegraph`, `memory-leak-debugging`, `go-vulncheck`, `infosec` |
| **10** | **Lead QA & Test Engineer** | `chrome-devtools`, `a11y-debugging`, `debug-optimize-lcp`, `memory-leak-debugging`, `troubleshooting` |
| **11** | **Lead DevOps & Security** | `infosec`, `cookie-debugging`, `troubleshooting`, `autonomous-investigation` |

---

## 9. Indeks Navigasi Berkas Profil Peran

Silakan buka masing-masing berkas profil untuk mempelajari deskripsi tugas mendalam, aturan interaksi paralel, dan template *system prompt* siap pakai:

1. 📊 [00-project-manager.md](./00-project-manager.md): Project Manager & Roadmap Coordinator
2. 🎯 [01-lead-orchestrator.md](./01-lead-orchestrator.md): Delivery Orchestrator & Task Gatekeeper
3. 📋 [02-lead-product-analyst.md](./02-lead-product-analyst.md): Lead Requirements & Business Discovery
4. 🏛️ [03-lead-system-architect.md](./03-lead-system-architect.md): Chief Architect & DDD Modular Monolith
5. 🗄️ [04-database-specialist.md](./04-database-specialist.md): Database Administrator & PostgreSQL Specialist
6. ⚙️ [05-lead-backend-engineer.md](./05-lead-backend-engineer.md): Lead Backend Engineer & Zero-Alloc Gatekeeper
7. 🔧 [06-backend-engineer.md](./06-backend-engineer.md): Go Backend Implementation Specialist
8. 🎨 [07-lead-frontend-engineer.md](./07-lead-frontend-engineer.md): Lead Frontend Engineer & Architecture Specialist
9. 🖥️ [08-frontend-engineer.md](./08-frontend-engineer.md): SolidJS Views & UI Implementation Specialist
10. 🛡️ [09-code-reviewer.md](./09-code-reviewer.md): Dedicated Code Reviewer (Maker-Checker Gatekeeper)
11. 🧪 [10-lead-qa-engineer.md](./10-lead-qa-engineer.md): Lead QA, Browser Testing & Benchmark Specialist
12. 🚀 [11-lead-devops-security.md](./11-lead-devops-security.md): Lead DevOps, Infrastructure & Information Security
