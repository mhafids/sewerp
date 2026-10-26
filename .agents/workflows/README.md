# Direktori Alur Kerja Agen SewERP (Agent Workflows)

Direktori ini memuat alur kerja (*workflows*) operasional resmi untuk ekosistem rekayasa perangkat lunak multi-agen **SewERP**.

Seluruh siklus pengembangan—mulai dari ide mentah klien konveksi hingga rilis produksi—telah dikonsolidasikan ke dalam **1 workflow tunggal terpadu** berbasis *Contract-First Parallelism* dan *Quality Gates*.

---

## 1. Workflow Resmi Aktif

### 🎯 [`agent-orchestration.md`](./agent-orchestration.md)
**Workflow Utama: End-to-End Multi-Agent Delivery Orchestration**

Workflow tunggal yang merumuskan koordinasi 12 peran agen SewERP dalam 8 tahapan terstruktur:
1. **Tahap 1: Penggalian Kebutuhan & Spesifikasi Modular** (`ROLE-REQ-02: Lead Product Analyst` + `ROLE-PM-00`) ➔ Menerbitkan `docs/specs/xxxy-{name}.md` dengan skenario Gherkin 4-pilar.
2. **Tahap 2: Perancangan Arsitektur Kontrak (*Contract-First*)** (`ROLE-ARCH-03: Lead System Architect`) ➔ Menerbitkan Mermaid ERD dan REST DTOs di `docs/memory/shareds/architect-xxxy-*.md`.
3. **Tahap 3: Quality Gate: Definition of Ready (DoR) & Task Graph** (`ROLE-ORCH-01: Lead Delivery Orchestrator`) ➔ Audit DoR dan dekomposisi graf tugas Atlas.
4. **Tahap 4: Eksekusi Paralel (*Contract-First Parallelism*)**:
   - *Terminal 1*: `ROLE-DBA-04 (Database Specialist)` ➔ Goose migrations & kueri sqlc.
   - *Terminal 2*: `ROLE-FE-SPEC-08 (SolidJS Frontend Engineer)` ➔ Views SolidJS, Kobalte, UnoCSS Flat UI, mock DTO, touch target 48px, zero-memory leak.
   - *Terminal 3*: `ROLE-QA-10 (QA Test Engineer)` ➔ Skenario Playwright E2E & mock thermal print 80mm.
5. **Tahap 5: Implementasi Backend & Zero-Alloc Engine** (`ROLE-BE-SPEC-06` & `ROLE-BE-LEAD-05`) ➔ Gin handlers, domain repositories, custom validator enum, dan benchmark `0 B/op` & `0 allocs/op`.
6. **Tahap 6: Sinkronisasi & Dedicated Code Review (*Maker-Checker Gate*)** (`ROLE-CR-09: Code Reviewer`) ➔ Audit Go & TypeScript independen, zero-float, zero detached DOM, PII masking Zerolog.
7. **Tahap 7: Validasi Riil QA & Pengujian Performa** (`ROLE-QA-10: QA Engineer`) ➔ E2E Playwright ke PostgreSQL riil & heap snapshot Chrome DevTools.
8. **Tahap 8: Audit Keamanan, Quality Gate Definition of Done (DoD) & Rilis** (`ROLE-OPS-11`, `ROLE-ORCH-01`, `ROLE-PM-00`) ➔ Distroless build container (<30MB), audit DoD semua subtask hijau, dan rilis resmi.

---

## 2. Format Baku Workflow

Setiap workflow dalam repositori SewERP wajib menyertakan frontmatter YAML:

```yaml
---
name: agent-orchestration
description: Deskripsi fungsi workflow
argument-hint: "<parameter yang diharapkan dari pemanggil>"
uses:
  - agent-orchestration-advisor
  - atlas
  - execute-task
  - spec-kit
outputs:
  - Artefak keluaran 1
  - Artefak keluaran 2
---
```

---

## 3. Protokol Memori 4-Tahap

Setiap agen yang dipanggil via CLI wajib membaca dan memperbarui memori secara berurutan untuk menjamin token efisiensi:

$$\text{Requirement} \longrightarrow \text{Context} \longrightarrow \text{Long Memory} \longrightarrow \text{Shareds}$$

- **Requirement**: `docs/specs/xxxy-{name}.md`
- **Context**: `docs/context/{domain}.md` (*Curated Selection*: hanya muat file yang relevan dengan peran).
- **Long Memory**: `docs/memory/long-memory/{xxxy}-invariants.md`
- **Shareds**: `docs/memory/shareds/{agent_from}-{xxxy}-{title}.md`

---

## 4. Panduan & Referensi Arsitektur Terkait

- 📚 [Panduan Tim Profil Agen & Ekosistem Rekayasa SewERP](../../docs/agent-profile/README.md)
- 🎯 [Profil Lead Delivery Orchestrator (`ROLE-ORCH-01`)](../../docs/agent-profile/01-lead-orchestrator.md)
- 📋 [Standar Spesifikasi Modular Kebutuhan](../../docs/context/requirements_standard.md)
- 🏛️ [Standar Arsitektur Clean Architecture & DDD](../../docs/context/architecture.md)
