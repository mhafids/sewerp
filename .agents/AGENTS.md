# Workspace Agent Rules & Guidelines

## Core Principles
1. **Always Plan First**:
   - Every task, code modification, feature addition, or workflow execution MUST go through Planning Mode first.
   - Always formulate an Implementation Plan artifact (`RequestFeedback: true`, `UserFacing: true`) detailing the goal, changes, and verification plan.
   - Wait for explicit user review and approval before touching workspace files or executing modifying commands.

2. **Custom Skills & Workflow**:
   - For end-to-end multi-agent delivery orchestration (from client ideas to production deployment), follow `.agents/workflows/agent-orchestration.md`.
   - Practice **Dynamic Role Selection & Pod Sizing**: `ROLE-ORCH-01` selects only the minimal required roles for the task, bypassing unused pipelines (e.g., bypassing DB/Go backend for lightweight UI/static prototypes).
   - Adhere to the project's Go DDD backend and SolidJS frontend architectural standards when applicable.
   - All requirement specifications MUST follow [`docs/context/requirements_standard.md`](../docs/context/requirements_standard.md) and be modularized using the `xxxy-{name requirement}.md` format.

3. **Dynamic Model Selection & Mandatory CLI Subagent Dispatch**:
   - Enforce [`rules/dynamic-model-dispatch.md`](./rules/dynamic-model-dispatch.md).
   - `ROLE-ORCH-01` MUST NOT write implementation code or QA scripts directly in the lead session; tasks MUST be dispatched to independent subagents via `run_command` using `opencode` or `agy`.
   - Models MUST be dynamically selected based on Cognitive Tiers: Tier 1 Reasoning (`deepseek-r1`, `claude-sonnet`), Tier 2 Coding (`deepseek-v3`, `qwen-2.5-coder-32b`), Tier 3 Context (`gemini-3.8-flash`), with self-healing fallback support.

4. **Agent Skills Recommendation Matrix (Multi-Skill Activation by Domain & Context)**:
   When an agent is assigned a task or working on a specific domain, document, or role, the agent should activate and combine the following recommended skills:

   - 📋 **Requirements Engineering & User Elicitation** (`docs/context/requirements_standard.md`, `docs/context/business_requirements.md`, `docs/specs/*`):
     - `intent-debugger`: Resolving ambiguous client ideas, unearthing edge cases, detecting implicit constraints, and clarifying intent.
     - `problem-framing-canvas` & `problem-statement`: Framing root problems (who is blocked, why it matters, operational impact on the workshop).
     - `proto-persona` & `jobs-to-be-done`: Profiling factory roles (Owner, Sales Admin, Cutting/Sewing Foremen, QC, Cashier) and core job motivations.
     - `discovery-process`: End-to-end problem framing, field stakeholder interviews, synthesis, and experiment formulation.
     - `prd-taskmaster` / `prd-development`: Translating discovery notes into structured PRDs with functional & non-functional boundaries.
     - `user-story` / `user-story-mapping` / `user-story-splitting`: Formulating prioritized stories in Mike Cohn format (P1-P3) and testable Gherkin 4-pillar scenarios.
     - `derisk-measurement-advisor`: De-risking assumptions across DUFV (desirability, usability, feasibility, viability) and PESTEL.
     - `erd-api-contract`: PostgreSQL DDL schemas, UUIDv7 PKs, BIGINT zero-float monetary storage, and RESTful API contracts with RFC 7807 problem details.
     - `spec-kit`: Spec-Driven Development, constitution rules, and architectural consistency checklists.
     - `execute-task` / `atlas`: Decomposing and executing implementation tasks with CDD verification.

   - 🛡️ **Security & Data Privacy** (`docs/context/security.md`, `docs/specs/009*.md`, `docs/specs/010*.md`):
     - `infosec`: Security audit, OWASP Top 10 / API Top 10, UU PDP compliance, data classification (Levels 1-4), secret protection, S3 presigned URL/watermark validation, and webhook signature HMAC verification.
     - `cookie-debugging`: Inspection of secure HttpOnly session cookies, SameSite policies, auth redirects, and session revocation.
     - `go-vulncheck`: Automated Go dependency vulnerability auditing (CVE checks).
     - `autonomous-investigation`: Researching threat models, supply chain vulnerabilities, and regulatory compliance.

   - 🧪 **QA & Testing** (`docs/context/qa.md`):
     - `chrome-devtools`: Playwright and Chrome DevTools MCP browser testing, console error audit, network throttling.
     - `a11y-debugging`: Accessibility auditing, WCAG AA compliance, ARIA attributes, tap targets.
     - `debug-optimize-lcp`: Core Web Vitals, page speed, LCP diagnosis.
     - `memory-leak-debugging`: Heap snapshots, detached DOM node detection, memory leak prevention.
     - `troubleshooting`: Target connection and test environment diagnosis.

   - 🖥️ **Frontend Engineering & Form Validation** (`docs/context/frontend.md`, `docs/context/forms.md`):
     - `ui-ux-pro-max`: Design systems, Flat UI tokens, component hierarchy, UnoCSS utilities.
     - `memory-leak-debugging`: Verifying SolidJS `onCleanup()` lifecycle hooks and eliminating detached DOM nodes.
     - `debug-optimize-lcp`: SPA load optimization and Vite asset bundling.
     - `generative_ui`: Interactive layout prototyping and inline preview widgets.
     - `a11y-debugging`: Ensuring keyboard navigation (Tab/Enter) on dynamic size matrices.

   - ⚙️ **Backend Engineering & Clean Architecture** (`docs/context/backend.md`, `docs/context/architecture.md`):
     - `erd-api-contract`: PostgreSQL DDL migrations, sqlc type-safe queries, UUIDv7 PKs, RESTful API contracts with RFC 7807 problem details.
     - `spec-kit`: Spec-Driven Development, architectural consistency checklists, boundary analysis.
     - `codegraph`: Verbatim source exploration, symbol references, and call graph analysis across Go layers.
     - `go-vulncheck`: Third-party Go module vulnerability auditing.

   - 🗄️ **Database Administration** (`docs/context/dbadmin.md`):
     - `erd-api-contract`: PostgreSQL schemas, migrations, UUIDv7, compound indexing, and zero-float monetary storage (`BIGINT` + `rate_bps`).
     - `spec-kit`: Schema rules and consistency validation.
     - `codegraph`: Tracing sqlc queries and database pool usage.

   - 🚀 **DevOps & Infrastructure** (`docs/context/devops.md`):
     - `troubleshooting`: Container configuration, Docker Compose healthchecks, reverse proxy debugging.
     - `autonomous-investigation`: Infrastructure security, image minimization (Distroless), and backup policies.
     - `infosec`: Container security, SSL/TLS configuration, and secret isolation.

   - 🎨 **UI & UX Design** (`docs/context/ui.md`, `docs/context/ux.md`):
     - `ui-ux-pro-max`: Modern Flat UI styling, Matcha Green & Sakura Pink palette, 1px crisp borders.
     - `customer-journey-map` / `customer-journey-mapping-workshop`: Garment factory floor ergonomics, workstation handoffs, operator touchpoints.
     - `jobs-to-be-done`: Identifying core user motivations for factory operators and sales admins.
     - `generative_ui`: Standalone mockups and visual UI components.
     - `a11y-debugging`: Minimum 48x48px touch target validation and contrast ratio auditing.
