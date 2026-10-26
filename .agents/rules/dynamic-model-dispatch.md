---
description: Enforce dynamic model selection and mandatory CLI subagent dispatch via opencode and agy.
---

# Dynamic Agent Dispatch & Dynamic Model Selection Rule

Aturan ini mengikat secara hukum bagi seluruh agen dalam ekosistem **SewERP** untuk mencegah pola *Monolithic In-Session Execution* (anti-pattern) dan menjamin efisiensi biaya komputasi berbasis tolak ukur LiveBench 2.0.

---

## 1. Mandat Lead Agent sebagai Dispatcher (Anti-Monolith Mandate)

1. **Lead Agent (`ROLE-ORCH-01`) DILARANG MENULIS KODE SPESIALIS**:
   - Ketika bertindak sebagai Lead Delivery Orchestrator, agen dilarang menulis, mengedit, atau menghasilkan kode produksi, skema migrasi database, tampilan antarmuka, atau skrip pengujian QA secara langsung di dalam sesi percakapan utama.
2. **Kewajiban Subprocess Dispatch**:
   - Lead Agent **WAJIB** mendelegasikan tugas-tugas peran aktif (Database Specialist, Backend Engineer, Frontend Engineer, Code Reviewer, QA Engineer) kepada child agents independen melalui terminal `run_command` menggunakan engine CLI resmi: **`opencode`** atau **`agy`**.

---

## 2. Dynamic Model Selection Protocol (3 Tingkatan Kognitif)

Pemilihan model AI **DILARANG DI-HARDCODE** ke satu string statis. Model wajib dipilih secara adaptif berdasarkan **Beban Kognitif Peran** (*Role Cognitive Load*) dan ketersediaan provider:

### A. Matriks Tingkatan Kognitif & Rekomendasi Model

| Tingkatan Kognitif | Peran Terkait | Model Primary (Hemat & Juara LiveBench) | Model Alternatif (Budget / Fast) | Fallback Premium / Multi-Platform |
| :--- | :--- | :--- | :--- | :--- |
| **Tier 1: Deep Reasoning, Architecture & Audit** | `ROLE-ARCH-03` (Architect)<br/>`ROLE-CR-09` (Reviewer)<br/>`ROLE-OPS-11` (Security)<br/>`ROLE-ORCH-01` (Orchestrator) | **`deepseek/deepseek-r1`** | **`deepseek/deepseek-v3`** | **`claude-sonnet-4-6`** (via agy)<br/>`openai/o3-mini` |
| **Tier 2: Code Generation & Strict Syntax** | `ROLE-DBA-04` (DBA)<br/>`ROLE-BE-SPEC-06` (Backend)<br/>`ROLE-FE-SPEC-08` (Frontend)<br/>`ROLE-QA-10` (Lead QA) | **`deepseek/deepseek-v3`**<br/>**`qwen/qwen-2.5-coder-32b`** | **`flint-40b-free`** | **`gemini-3.8-flash-high`** (via agy)<br/>`claude-sonnet-4-6` |
| **Tier 3: Mass Context & Spec Discovery** | `ROLE-REQ-02` (Product Analyst)<br/>`ROLE-PM-00` (Project Manager) | **`gemini-3.8-flash-high`** (via agy) | **`deepseek/deepseek-v3`** | **`claude-sonnet-4-6`** |

### B. Protokol Self-Healing Fallback
Jika model primary mengalami kendala (HTTP 429 Rate Limit, 503 Provider Down, Timeout, atau kegagalan kuota):
1. **Otomatis Switch ke Alternatif**: Beralih ke model pada kolom *Alternatif*.
2. **Eskalasi ke Fallback**: Jika alternatif gagal, eskalasikan ke model pada kolom *Fallback Premium*.
3. **Catat Insiden**: Catat kegagalan dan fallback pada shared memory peran tersebut.

---

## 3. CLI Engine Auto-Routing (`opencode` vs `agy`)

Lead Agent wajib memilih CLI engine yang tepat berdasarkan lokasi hosting model:

### Jalur A: OpenCode CLI (`opencode`)
Gunakan `opencode` untuk model-model frontier hemat biaya via SumoPod / Multi-Provider:
- Model: `deepseek/*`, `qwen/*`, `glm/*`, `flint-*`
- Format Eksekusi Non-Interaktif:
  ```bash
  opencode run --model "<provider/model>" --auto "=== AGENT EXECUTION DIRECTIVE ===
  [ROLE]: [Nama Peran]
  ...
  "
  ```

### Jalur B: Antigravity CLI (`agy`)
Gunakan `agy` untuk model-model native Google atau Anthropic:
- Model: `gemini-3.8-flash-high`, `claude-sonnet-4-6`, `claude-opus-4-6-thinking`
- Format Eksekusi Non-Interaktif:
  ```bash
  agy --dangerously-skip-permissions --model "<model>" --print "=== AGENT EXECUTION DIRECTIVE ===
  [ROLE]: [Nama Peran]
  ...
  "
  ```

---

## 4. Protokol Memori & Verifikasi Selesai (DoD Handshake)

1. **Child Agent Menulis Shared Memory**:
   Setiap subagent yang dieksekusi via `opencode` atau `agy` wajib mempublikasikan laporannya ke berkas:
   `docs/memory/shareds/{role}-{task_identifier}.md`
2. **Lead Agent Membaca & Memverifikasi**:
   Lead Agent membaca berkas shared memory tersebut, memvalidasi blok keputusan formal (`[APPROVED]`), lalu memperbarui status Definition of Done (DoD).