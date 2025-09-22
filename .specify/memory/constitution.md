<!--
Sync Impact Report
- Version change: 1.1.0 → 1.2.0
- Modified principles: None (clarifications only)
- Added sections:
  - Evidence Pack Layout Baseline (from sample sheet)
  - Development Workflow: Docs Sourcing & MCP (Context7)
- Removed sections: None
- Templates requiring updates:
  - ✅ .specify/templates/plan-template.md (update to v1.2.0)
  - ✅ .specify/templates/spec-template.md (aligned; no changes required)
  - ✅ .specify/templates/tasks-template.md (aligned; no changes required)
  - ⚠ .specify/templates/commands/* (no directory present; none to update)
- Follow-up TODOs:
  - TODO(PACK_LAYOUT_CONF): Confirm mandatory layout elements from provided dayworks PDF.
-->

# SiteProof Constitution

## Core Principles

### I. Frontend‑First, Fixtures‑Driven MVP
The MVP MUST be built UI‑first using mock JSON fixtures to validate flows quickly.
Initial screens (Import, Review Queue, Packs, Settings) SHALL consume static
endpoints: `/fixtures/projects.json`, `/fixtures/import_batches.json`,
`/fixtures/review_queue.json`, `/fixtures/packs.json`. Backend integration occurs
only after UI/UX validation with stakeholders.

Rationale: De‑risks backend complexity, accelerates iteration, and enables
feedback on real workflows before wiring data pipelines.

### II. Export‑First Ingestion (WhatsApp .zip) — EU‑Resident Only
Primary ingestion for MVP MUST be exported WhatsApp `.zip` per project. No
"bot‑in‑group" or real‑time ingestion in MVP. All storage and compute MUST be EU
region (preferably Ireland). Media access MUST use short‑TTL signed URLs.

Rationale: Mirrors current user behavior, minimizes ToS risk, and ensures EU
compliance from day one.

### III. Async Jobs With Polling — No Long‑Running APIs
Ingestion, STT, OCR, and extraction MUST run in background workers with job
status exposed via polling. No API request may exceed 30 seconds. Jobs MUST be
idempotent, restartable, and retried with exponential backoff and DLQ.

Rationale: Reliable handling of large files (100–600 MB) and variable compute
tasks without overloading request lifecycles.

### IV. HITL Review With Confidence Gates and Audit Trail
Extraction produces field‑level confidence. Auto‑approve EXISTS but STARTS STRICT:
by default, items require human review unless all fields ≥0.98 and there are no
blocking flags (e.g., missing rate, low‑quality OCR/STT). As accuracy improves
and acceptance targets are met over a full month, thresholds MAY be lowered
(e.g., towards ≥0.90) per project with explicit sign‑off. Otherwise items MUST
enter a keyboard‑first review queue. All edits MUST be audited (who, when,
before/after) and traceability to source message/media MUST be preserved.

Rationale: Ensures accuracy and defensibility while keeping reviewers efficient.

### V. Traceability, Rates via CSV as Source of Truth, and Cost Discipline
Every item MUST retain links to original message/media and timestamps. Rate
application MUST come from a CSV mapping (EUR) with unit normalization; manual
overrides are audited. Media MUST be deduplicated via SHA‑256; STT/OCR SHOULD
run on‑demand to control cost. Keep solutions simple; avoid premature features.

Rationale: Defensible packs, consistent totals, and predictable operating costs.

## MVP Constraints & Standards

- Technology stack: TBD (to be finalized via ADRs). Preferences exist in the
  brief, but stack selection MUST be explicitly decided with links to official
  documentation. The agent MUST request official docs for chosen technologies
  and escalate when blocked for debugging. Until finalized, UI proceeds with
  fixtures and does not block backend decisions.
- Performance targets:
  - 100 MB export parse/index P50 < 10 min, P95 < 15 min (ingestion only).
  - Pack generation < 60 s (excluding background STT/OCR still running).
  - No API > 30 s.
- Scope boundaries (MVP): No e‑signature, no bot‑in‑group ingestion, mobile
  view‑only later, CSV rate import is source of truth, optional name redaction
  (default: keep names), manual purge by import batch.
- Security & compliance:
  - EU‑resident storage/compute only; signed URLs (TTL ≤ 10 min).
  - RLS by organization/project; audit events for reviewer edits.
  - PII in logs is prohibited; structured logs with PII filtering.
- Media handling: SHA‑256 dedup; thumbnails only for images; skip very long
  videos; robust error handling and retries.

## Development Workflow & Quality Gates

1) UI First
- Build screens against fixtures. Do not block on backend. Fixtures MUST reflect
  the contracts listed above. Any schema change requires updating fixtures and
  noting the change in `CHANGELOG` or equivalent.

2) Contract‑First Backend
- Define API contracts and Prisma schema before implementation. For backend,
  write contract tests first. For UI, story/interaction tests MAY follow after
  fixture validation.

3) Async by Design
- Long tasks run in workers (BullMQ). Expose job IDs and polling endpoints.
  Enforce timeouts on all HTTP requests.

4) Quality Gates
- Constitution Check in plans/specs MUST pass or include a justified Complexity
  deviation. No deployment if EU residency, traceability, or audit gates fail.
- Performance gates MUST be tracked in telemetry; deviations require rationale
  and a remediation task.

5) Accessibility & UX
- Keyboard‑first review flows; confidence cues visible; responsive tables for
  large queues. Dark mode is optional post‑MVP.

6) Code Reviews & Automation
- All commits and PRs MUST be reviewed by the CodeRabbit GitHub App. Configure
  required status checks so merges are blocked until CodeRabbit reviews pass.
  Add a repository config file if required by CodeRabbit (e.g., `.coderabbit.yml`)
  once official documentation is provided. CI SHOULD run lint/tests on PRs.

7) Docs Sourcing & MCP (Context7)
- For any setup/config, library API usage, or debugging, the agent MUST request
  and embed official documentation, preferably via the Context7 MCP tools
  (`resolve-library-id`, `get-library-docs`) when available. Prompts SHOULD
  include `use context7` to enrich answers with up‑to‑date docs. Decisions MUST
  reference doc versions/links in ADRs and specs.

## Evidence Pack Layout Baseline

Derived from the provided timesheet sample (EVO Civils):
- Header: Contractor name/logo, sheet title (e.g., “Plant & Labour Weekly Timesheet”),
  sheet number, location, week ending date, contract/project identifiers.
- Description of Work: Free‑text multi‑line area summarizing activities.
- Line Items Table (per day columns):
  - Columns: Name/Item, Trade/Description, Mon..Sun daily hours/qty, Total, Comments.
  - Rows can include Labour (names/operators), Plant (equipment), and Materials.
- Totals/Footers: Separate sections for Tools, Transport, Plant; Client Signature area.
- Signature: Client sign‑off (printed name/date optional for MVP).

This baseline informs the PDF template and CSV export structure; exact wording and
styling may be adjusted. TODO(PACK_LAYOUT_CONF): confirm any mandatory client fields.

## Governance

- Authority: This constitution supersedes other practices for MVP work.
- Amendments: Propose via PR including rationale, migration/impact notes, and
  updates to dependent templates. On approval, bump version per SemVer and set
  Last Amended date.
- Versioning (SemVer for governance):
  - MAJOR: Backward‑incompatible governance changes or principle removals.
  - MINOR: New principles/sections or material expansions.
  - PATCH: Clarifications and non‑semantic refinements.
- Compliance Reviews: Plans (/plan) and specs (/spec) MUST evaluate Constitution
  Gates. If violations exist and cannot be justified, simplify before proceeding.
- Review Cadence: Re‑evaluate principles after first end‑to‑end pack is
  generated on a real dataset.

- Tech Stack Determination & Agent Documentation Policy:
  - Tech stack is not locked until ADRs are approved. The agent MUST request and
    reference official documentation for any proposed stack and for debugging
    when blocked. Decisions and changes MUST be recorded as ADRs.

**Version**: 1.2.0 | **Ratified**: 2025-09-22 | **Last Amended**: 2025-09-22

# [PROJECT_NAME] Constitution
<!-- Example: Spec Constitution, TaskFlow Constitution, etc. -->

## Core Principles

### [PRINCIPLE_1_NAME]
<!-- Example: I. Library-First -->
[PRINCIPLE_1_DESCRIPTION]
<!-- Example: Every feature starts as a standalone library; Libraries must be self-contained, independently testable, documented; Clear purpose required - no organizational-only libraries -->

### [PRINCIPLE_2_NAME]
<!-- Example: II. CLI Interface -->
[PRINCIPLE_2_DESCRIPTION]
<!-- Example: Every library exposes functionality via CLI; Text in/out protocol: stdin/args → stdout, errors → stderr; Support JSON + human-readable formats -->

### [PRINCIPLE_3_NAME]
<!-- Example: III. Test-First (NON-NEGOTIABLE) -->
[PRINCIPLE_3_DESCRIPTION]
<!-- Example: TDD mandatory: Tests written → User approved → Tests fail → Then implement; Red-Green-Refactor cycle strictly enforced -->

### [PRINCIPLE_4_NAME]
<!-- Example: IV. Integration Testing -->
[PRINCIPLE_4_DESCRIPTION]
<!-- Example: Focus areas requiring integration tests: New library contract tests, Contract changes, Inter-service communication, Shared schemas -->

### [PRINCIPLE_5_NAME]
<!-- Example: V. Observability, VI. Versioning & Breaking Changes, VII. Simplicity -->
[PRINCIPLE_5_DESCRIPTION]
<!-- Example: Text I/O ensures debuggability; Structured logging required; Or: MAJOR.MINOR.BUILD format; Or: Start simple, YAGNI principles -->

## [SECTION_2_NAME]
<!-- Example: Additional Constraints, Security Requirements, Performance Standards, etc. -->

[SECTION_2_CONTENT]
<!-- Example: Technology stack requirements, compliance standards, deployment policies, etc. -->

## [SECTION_3_NAME]
<!-- Example: Development Workflow, Review Process, Quality Gates, etc. -->

[SECTION_3_CONTENT]
<!-- Example: Code review requirements, testing gates, deployment approval process, etc. -->

## Governance
<!-- Example: Constitution supersedes all other practices; Amendments require documentation, approval, migration plan -->

[GOVERNANCE_RULES]
<!-- Example: All PRs/reviews must verify compliance; Complexity must be justified; Use [GUIDANCE_FILE] for runtime development guidance -->

**Version**: [CONSTITUTION_VERSION] | **Ratified**: [RATIFICATION_DATE] | **Last Amended**: [LAST_AMENDED_DATE]
<!-- Example: Version: 2.1.1 | Ratified: 2025-06-13 | Last Amended: 2025-07-16 -->