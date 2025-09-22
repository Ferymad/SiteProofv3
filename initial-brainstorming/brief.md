# Project Brief: SiteProof

## Introduction

SiteProof transforms WhatsApp group chatter from construction projects (text, voice notes, photos/videos) into structured daywork entries and a monthly evidence pack. The MVP targets EU/Ireland deployments with human-in-the-loop verification for accuracy and auditability.

- Primary users: Project Manager (PM) and Contract Manager (CM).
- Approach: Start with WhatsApp export .zip ingestion → parse → extract Labour/Material/Plant/Delay with confidence scores → queue for human review → generate monthly PDF/CSV pack with traceability.
- Region & data: EU-first hosting and processing; optional name redaction (default: keep names, toggle available). Data retention policies are out of scope for MVP.
- Rate cards: EUR only. Support upload from spreadsheets; MVP path is guided CSV mapping; backlog adds AI-assisted Excel parsing (e.g., via an LLM indexer) once schema examples are gathered.
- Scope boundaries (MVP): No e-signature, no in-chat bot ingestion (research-only), no formal retention policy.
- Large imports: Baseline 100 MB (MVP) with resumable/streamed uploads and background processing; 600 MB is a post‑MVP stretch target.

Inputs considered: `docs/brainstorming-session-results.md` (2025-09-21) and sample pack reference `whatsapp-data/dayworks-sheets/Evo Civils Eden Dayworks April rev1.pdf` for format inspiration.

---

(Interactive drafting in progress; sections will be filled as we proceed.)

### Risks Snapshot (MVP)

- Large zip ingestion (100–600 MB): Use resumable multipart uploads, streaming unzip/parse, background jobs with progress. Guard with per-file limits and backpressure.
- Media volume and cost: Deduplicate via SHA‑256 file hashing (perceptual hashing later); defer STT/OCR until needed; skip oversized/long videos; add quotas and cost visibility.
- EU data residency/compliance: Select EU-region providers (STT/OCR/storage); encrypt data; maintain processor list; provide redaction toggle.
- Rate card import from Excel: Start with guided CSV mapping + schema validation; use AI parsing only as an optional assist; require HITL confirmation.
- Extraction accuracy (jargon/accents/low-quality images): Domain dictionary, phrase patterns, pre-processing, and confidence thresholds that route to review queue.
- Long-running compute/timeouts: Offload ingestion/STT/OCR to worker queue; avoid long-running serverless requests; make APIs async with polling (no webhooks in MVP).

## Executive Summary

- What: SiteProof converts exported WhatsApp chats (text, voice, photos/videos) from construction site groups into structured daywork entries and a client-ready monthly evidence pack (PDF/CSV) with traceable links back to each source.
- Who: Contract and Project Managers at small–mid construction contractors in EU/Ireland.
- How: Reliable .zip ingestion, human-in-the-loop review with confidence cues, and automatic rate card application (EUR; CSV import initially).
- Value: Target >50% reduction in monthly pack preparation time while improving auditability and consistency; EU-first processing with optional name redaction.

## Problem Statement

- Current state and pain: Dayworks evidence is reconstructed from sprawling WhatsApp group chats (text, voice notes, photos/videos). PMs and CMs manually skim threads, transcribe audio, and retype figures into spreadsheets. Media quality and domain jargon reduce STT/OCR reliability, and the lack of a consistent schema (Labour/Material/Plant/Delay) forces ad‑hoc interpretation.
- Impact: Month-end preparation consumes significant PM/CM time and risks missed items or miscalculations. Limited traceability back to the original messages and media undermines confidence during client review. We will validate baselines, targeting a measurable reduction in prep time and fewer rework cycles.
- Why existing solutions fall short: Generic chat exports are archival and unstructured. Spreadsheet workflows are fragile, lack confidence scoring, and rarely preserve message/media links required for defensible dayworks.
- Urgency: Imminent billing cycles require a dependable process that can handle 100–600 MB exports in the EU, prioritizing reliability and auditability over real-time ingestion.

## Proposed Solution

- Core approach: Upload exported WhatsApp `.zip` per project to EU object storage → background worker/queue (e.g., BullMQ/Redis) stream‑unzips and parses messages/media → unify STT (voice) and OCR (images) → extract L/M/P/Delay fields with confidence scores → human‑in‑the‑loop review queue → apply EUR rate cards (CSV‑mapped; CSV mapping is the source of truth, AI Excel parsing is optional and requires HITL) → generate monthly evidence pack (PDF/CSV) with traceable links to original sources; job status exposed via polling (webhooks post‑MVP).
- Key differentiators: Dayworks‑specific schema and confidence gating; full traceability (message IDs, timestamps, media refs); EU‑first storage/compute; large‑zip‑first ingestion; keyboard‑first review UI; domain dictionary + phrase patterns with few‑shot examples.
- MVP features: Keyboard‑first review experience; ingestion/processing telemetry; cost visibility (STT/OCR/storage), progress and exceptions views.
- Non‑functional targets (MVP): Import a 100 MB export in < 10 minutes end‑to‑end; no API request > 30 seconds; EU‑only storage/compute; idempotent, restartable jobs; stable memory during unzip; duplicate media deduplication.
- Why this will succeed: Mirrors how teams already work (exports, not bots), prioritizes auditability and defensibility, controls costs via dedupe and on‑demand STT/OCR, and focuses on reliability over real‑time features.
- Vision beyond MVP: Optional browser helper; real‑time ingestion (if compliant); sign‑off flow and sealed packs; configurable templates/branding; analytics; collaborator/QS sharing; improved rate import (AI‑assisted Excel parsing with HITL).

## Target Users

### Primary User Segment: Project Manager (PM)

Demographic/Firmographic
- Small–mid construction contractors operating in EU/Ireland; WhatsApp group per project; heavy Excel usage; Windows/macOS desktop + phone.

Behaviors & Workflows
- Posts daily updates (text, voice notes, photos/videos) to project WhatsApp groups; end‑of‑month compilation into dayworks spreadsheets.
- Uses WhatsApp Desktop/phone exports; organizes evidence manually across folders/spreadsheets.

Needs & Pain Points
- Reduce time spent scanning chats and transcribing audio/photos; avoid missed items/miscalcs.
- Preserve traceability to original messages/media for client reviews.
- Apply correct EUR rates consistently without fragile spreadsheet formulas.

Goals
- Produce a defensible monthly pack quickly; minimize rework cycles; maintain EU‑resident processing.

### Secondary User Segment: Contract Manager (CM)

Demographic/Firmographic
- Oversees multiple projects; ensures contractual compliance and rate application; collaborates with PMs.

Behaviors & Workflows
- Reviews compiled dayworks; spot‑checks traceability; coordinates sign‑off with clients.

Needs & Pain Points
- Confidence in data fidelity and rate application; visibility into low‑confidence/exception items.

Goals
- Increase acceptance rate of submitted dayworks; reduce back‑and‑forth with clients; maintain audit trail.

#### Client-Facing Summary

- PM: Produce monthly dayworks packs faster with clear links back to WhatsApp messages and media; apply correct EUR rates; EU‑resident processing.
- CM: Higher acceptance rate with audit‑ready traceability; resolve exceptions before submission; less back‑and‑forth with clients.

#### Internal Build Notes

- Top tasks: Import .zip per project → Review flagged items (keyboard‑first) → Apply EUR rates (CSV mapping) → Generate PDF/CSV pack with traceability → Submit/sign‑off.
- Edge cases: Multi‑project exports, missing/ambiguous rates, duplicate media, very long videos, timezone normalization, corrupt files.
- Success metrics (MVP): Time reduction per project/month; pack acceptance rate; % items auto‑approved vs reviewed; % exceptions resolved pre‑pack; review throughput.
- Accessibility/DX: Keyboard‑first review; large‑table navigation; prominent confidence cues; dark mode later.

## Goals & Success Metrics

### Business Objectives

- Reduce monthly dayworks preparation time per project by ≥50% within two months of adoption.
- Achieve ≥80% client acceptance on first submission of monthly packs.
- Enforce EU‑only storage/compute; 0 critical compliance issues in MVP.
- Keep ingestion/processing cost per 100 MB export under €X (set after first measurements).

### User Success Metrics

- Median time to produce a monthly pack ≤ N hours (target to be validated with baseline).
- ≥40% of items auto‑approved without edits at configured confidence thresholds.
- ≥95% of exceptions resolved before pack generation.
- Reviewer throughput ≥60 items/hour using keyboard‑first flow on test dataset.

### Key Performance Indicators (KPIs)

- Import duration (100 MB export): P50 < 10 min, P95 < 15 min end‑to‑end.
- API request limits: no request > 30 s; worker job success rate ≥99% over rolling 24 h.
- Traceability coverage: 100% of items retain source message/media link and timestamp.
- Deduplication effectiveness: ≥Y% duplicate media detected and re‑used (to be baselined).
- STT/OCR yield: ≥90% of voice notes transcribed; ≥85% of images OCR’d; average extraction confidence ≥0.80.
- Cost per active project per month ≤ €Z (to be set after initial runs).

## MVP Scope

### Core Features (Must Have)

- WhatsApp export ingestion
  - Upload exported `.zip` per project to EU object storage with resumable multipart uploads.
  - Background worker/queue stream‑unzips and parses `*_chat.txt`; indexes message metadata and links media by reference (no full zip in memory).
  - Large file baseline: support 100 MB exports end‑to‑end within targets; 600 MB treated as stretch test post‑MVP.
  - Import wizard confirms project and month; supports iOS/Android/desktop export variants; allows override of inferred mappings.

- Traceability and audit
  - Persist message id, timestamp, author, chat title, media filename; compute stable import batch id; per‑project mapping.
  - Signed URLs for media with expiry; optional name‑redaction toggle.

- Extraction + Review (HITL)
  - Hybrid extraction (rules + few‑shot) for Labour/Material/Plant/Delay with field‑level confidence.
  - STT for voice notes and OCR for images on‑demand via EU providers; low‑confidence items routed to review queue.
  - Keyboard‑first review UI showing source snippet + media thumb; approve/reject with audit trail.
  - Default thresholds: auto‑approve item when all fields ≥0.90 confidence; otherwise queue for review (configurable per project).

- Rates and totals
  - Import EUR rate cards via CSV mapping UI (CSV is source of truth); manual overrides with audit; warnings for missing rates.
  - Minimal CSV schema: `category(L|M|P)`, `item_name_or_code`, `unit`, `rate_eur`; unit normalization dictionary (e.g., hr/h/Hour → hour).

- Evidence pack generation
  - Monthly PDF + CSV with sections by date, image thumbnails only (videos show icon/link), line items, page totals by day, and a traceability appendix (message/media refs) with signature placeholders (printed name/date, no e‑signature).
  - Exceptions report listing unresolved items blocking pack finalization.

- Platform and operations
  - EU‑resident storage/compute (e.g., Supabase EU + EU STT/OCR); Prisma + Postgres with RLS; signed URLs; encryption in transit.
  - RBAC baseline: Organization Admin and Project Editor roles; RLS policy examples enforced per organization/project.
  - Job status via polling endpoints only (no webhooks in MVP); basic telemetry (import duration, STT/OCR counts, storage used), and cost visibility.
  - Delete/purge: Manual "delete import batch" removes DB rows and associated storage objects for that batch.
  - NFRs: 100 MB ingestion P50 < 10 min for parse/index stage (STT/OCR run async with progress); no API > 30 s; idempotent, restartable jobs; duplicate media dedup via SHA‑256 file hashing (perceptual hashing is post‑MVP); worker retries (e.g., 3) with exponential backoff and DLQ.

### Out of Scope (MVP)

- Live ingestion/bot‑in‑group, WhatsApp Business API, or browser helper extension.
- E‑signature, sealed/hashed packs, or cryptographic notarization.
- Mobile apps (iOS/Android); offline desktop packaging.
- Advanced retention policies, DSAR tooling, or legal DPA generator.
- SSO/SCIM and granular role matrices beyond simple admin/editor.
- Multi‑currency; complex tax/VAT handling; non‑EU hosting/processing.
- Advanced analytics dashboards, alerting, or client portals/QS collaboration spaces.
- Deep accounting integrations (Sage/QuickBooks/ERP).
- Full custom theming/branding beyond logo/header and simple styles.
- High‑fidelity handwriting OCR; long‑video transcription; media editing.
 - Perceptual hashing for near‑duplicate media; webhooks for job status.

### MVP Success Criteria

 - Using a 100 MB iOS/Android WhatsApp export for one active project, user can:
  - Upload and complete ingestion within performance targets.
  - Review and approve items via keyboard‑first UI, with 100% traceability preserved.
  - Import a EUR rate card (CSV) and generate a monthly PDF + CSV pack with an exceptions report.
  - Achieve ≥80% first‑submission acceptance with the client on the generated pack.
  - Demonstrate EU‑only storage/compute and signed‑URL media access.
  - Purge an import batch and verify associated data and storage objects are removed.

## Post‑MVP Vision

### Phase 2 Features

- Browser helper for WhatsApp Web (feasibility and ToS permitting).
- Sign‑off flow and sealed PDF versions; configurable templates/branding options.
- Exceptions dashboard with drill‑downs; richer pack builder options.
- Improved rate import: AI‑assisted Excel parsing with HITL; unit dictionary expansion.

### Long‑term Vision

- Near‑real‑time ingestion (if compliant) with notifications; QS/client portal for collaboration and approvals.
- Advanced search across messages/media/items; analytics on productivity and costs.
- Perceptual hashing for near‑duplicate media; smarter dedup and media management.
- E‑signature integrations; stronger compliance features and retention policies.

### Expansion Opportunities

- Multi‑currency and regional roll‑outs beyond EU/Ireland.
- Accounting integrations (Sage/QuickBooks/ERP) for downstream posting.
- Mobile capture companion app for ad‑hoc uploads outside WhatsApp.

## Technical Considerations

### Client-Facing Summary

- EU-first: All storage and compute reside in EU regions; media access via short‑lived signed links.
- Performance: 100 MB export ingests in about 10 minutes on average (parse/index); longer tasks like transcription run in the background with progress.
- Reliability: Background workers handle big files safely; failed jobs retry automatically.
- Security & Privacy: Per‑project access controls; audit trail for edits; optional name redaction in outputs.
- Compatibility: Modern desktop browsers (Chrome, Edge, Safari) on Windows/macOS; mobile read‑only post‑MVP.

### Platform Requirements

- Target Platforms: Modern web browsers on desktop (Chrome, Edge, Safari) for PM/CM; mobile web read‑only acceptable post‑MVP.
- Browser/OS Support: Windows 10+, macOS 12+; Safari 16+; Chrome/Edge last 2 versions.
- Performance: 100 MB export ingestion P50 < 10 min (parse/index only); review UI responsive with 5k+ items; pack generation < 60 s for 1‑month dataset (excluding STT/OCR jobs running async).

### Technology Preferences

- Frontend: Next.js (App Router) + TypeScript + TailwindCSS + shadcn/ui; keyboard‑first components; server actions for small tasks.
- Backend: Node/TypeScript single repo; Prisma ORM; Supabase Postgres (EU region) with RLS; REST endpoints for job submission + polling.
- Workers/Queue: BullMQ + Redis (EU) for ingestion/STT/OCR/extraction jobs; idempotent processors; DLQ.
- Storage: Supabase Storage or S3‑compatible EU bucket; SHA‑256 dedup; signed URLs with short TTL.
- STT/OCR: EU‑resident managed services or self‑hosted Whisper/Tesseract on EU compute; run on‑demand behind queue.

### Architecture Considerations

- Repository Structure: Monorepo (Next.js app + worker package); shared `@siteproof/types` for schemas; `@siteproof/extractors` for rules + few‑shots.
- Service Architecture: Web app (upload/monitor/review/pack) + worker (ingest, generate image thumbnails, STT/OCR, extraction) + Postgres + Redis.
- Integration Requirements: None for MVP beyond email login; optional SMTP for notifications later.
- Security/Compliance: RLS by organization/project; signed URLs; encryption in transit; audit events; PII redaction toggle; EU‑only regions configured.

### Internal Technical Notes

#### Decision Log (ADRs)

- ADR‑001: EU‑only storage/compute (Accepted)
- ADR‑002: Node/TypeScript monorepo, shared types (Accepted)
- ADR‑003: BullMQ/Redis workers; status via polling (Accepted)
- ADR‑004: Rates via CSV mapping (source of truth); AI Excel optional with HITL (Accepted)
- ADR‑005: SHA‑256 media dedup (perceptual hashing post‑MVP) (Accepted)

#### Quality Attribute Scenarios (QAS)

- Performance: When a user uploads a 100 MB export, messages are parsed and indexed within 10 min P50 (15 min P95) with visible progress; STT/OCR runs async per‑file.
- Reliability: When a worker job fails, it retries up to 3 times with exponential backoff; on exhaustion it moves to DLQ and UI shows recovery actions.
- Security: When media is viewed, access uses signed URLs with TTL ≤ 10 min; access is project‑scoped and logged.
- Compliance: When any job runs, processing occurs only in EU regions; cross‑region writes are disabled by config.
- Operability: When ingestion completes, telemetry records duration, message/media counts, STT/OCR counts, dedup rate, and estimated cost.

#### Provider Evaluation Matrix (to be completed)

- Columns: Service, EU Regions, Pricing, Accuracy (domain vocab), Latency, DPA/Terms, Residency Guarantees, Integration notes.

#### Budgets & Quotas (initial)

- 1 project (MVP), up to 2 imports/month, 100 MB baseline file size, image thumbnails only, max concurrent STT/OCR jobs per batch = K (TBD by testing).

#### Threat Model (concise)

- Areas: Upload tampering, URL leakage, cross‑tenant access, PII in logs, third‑party LLM data exposure.
- Mitigations: Checksums on upload parts; short TTL signed URLs; strict RLS per org/project; structured logs with PII filters; no third‑party LLM over raw media without consent.

#### Developer Experience

- Monorepo scaffolding; seed data and synthetic WhatsApp export generator; minimal E2E covering QAS; “cost sandbox” to stub STT/OCR locally; keyboard‑first UI components library.

## Constraints & Assumptions

### Constraints

- Region & compliance: All storage/compute and managed services run in EU regions; media served via short‑lived signed URLs.
- File sizes: Baseline support for 100 MB exports in MVP; 600 MB considered a post‑MVP stretch scenario.
- Resources: Solo builder (“vibe coder”) with AI‑assisted coding; prefer TypeScript‑only stack and managed services to minimize ops.
- Timeframe: Vertical slices targeted in 2–6 weeks total (import → review → pack), per brainstorming plan.
- Tech constraints: Next.js + TS + Prisma + Supabase Postgres; Redis/BullMQ workers; polling status (no webhooks in MVP).
- EU STT/OCR: Use EU‑resident providers or self‑host in EU; background/async only.
- Desktop export availability: WhatsApp Desktop export on Ubuntu not available for you currently; rely on WhatsApp Desktop on Windows/macOS or phone export flows.
- Security: RLS enforced by organization/project; audit events required for all reviewer edits; optional name‑redaction toggle.
- Budget: Cost visibility required; hard caps/alerts to avoid runaway STT/OCR/storage spend; placeholders to be set after baseline.

### Key Assumptions

- Each project maintains a distinct WhatsApp group; exported `.zip` includes a single `*_chat.txt` and media folder.
- PM/CM are the only MVP users; QS/finance reviewers may read packs but won’t author in MVP.
- Clients accept PDF + CSV outputs modeled after existing dayworks sheets; reference file: `whatsapp-data/dayworks-sheets/Evo Civils Eden Dayworks April rev1.pdf`.
- Rate cards are available in Excel; MVP uses a guided CSV mapping as the source of truth; AI Excel parsing is optional with HITL.
- Confidence thresholds can gate auto‑approval without harming acceptance (initial default: all fields ≥0.90).
- Data retention policy is out of scope for MVP; simple manual purge by import batch is acceptable.
- Mobile usage is view‑only post‑MVP; desktop browsers are primary for ingestion/review.

## Risks & Open Questions

### Key Risks

- Ingestion performance & memory: Large zips (100–600 MB) can spike memory or stall; enforce streaming unzip, backpressure, and strict concurrency.
- STT/OCR accuracy: Noisy site audio, accents, and low‑quality images reduce accuracy; rely on domain dictionary, patterns, and HITL gating.
- Rate card mapping errors: Ambiguous columns/units in Excel; mitigate with CSV schema, unit normalization, previews, and HITL.
- EU residency/compliance: Vendor services may process outside EU; limit to EU‑region services and verify DPAs/residency guarantees.
- Cost unpredictability: STT/OCR/storage costs can balloon; add quotas, per‑batch cost telemetry, and a “cost sandbox” mode in dev.
- Evidence acceptance: Client expectations on layout may vary; keep export templated and configurable enough to mirror current sheets.
- Project mapping mistakes: Wrong group→project linkage on import; require import wizard confirmation and allow overrides.
- Time normalization: Phrases like “yesterday morning” vs timestamps; prefer chat timestamps and show derived dates explicitly.
- Media handling: Long videos and corrupt files; skip long‑video STT, image‑only thumbnails, robust error handling and retries.
- Security exposures: URL leakage or cross‑tenant access; use short‑TTL signed URLs, strict RLS, and audit events.

### Open Questions

- Redaction default: Keep names by default (current plan) — confirm per‑project default and whether to generate a sanitized pack variant.
- Confidence gates: Are item auto‑approve thresholds of ≥0.90 per field acceptable initially? Any category‑specific thresholds?
- Rate CSV schema: Do we need item codes in addition to names? Any per‑project overrides or seasonal rate changes?
- Pack layout: Any mandatory client elements (logos, cover page wording, footer text)? Confirm “signature placeholders only” is acceptable.
- STT/OCR concurrency caps: What’s an acceptable default to balance speed vs. cost for your typical month?
- Multi‑project exports: Should we support splitting a single export across projects in MVP, or warn and block?
- Auth: Is email‑link (Supabase) sufficient for MVP, or do you need password or OAuth?
- Baselines: Can you provide ballpark current time spent per month per project to validate the “>50% reduction” target?

### Areas Needing Further Research

- EU STT/OCR providers vs. self‑hosting: Compare EU‑region managed services with self‑hosted Whisper/Tesseract on EU compute (accuracy, cost, ops load).
- WhatsApp export variants: Catalog differences across iOS, Android, and Desktop (date formats, attachments) and adjust parser.
- PDF generation approach: HTML‑to‑PDF (headless browser) vs. PDFKit/react‑pdf (control vs. simplicity) for layout fidelity.
- Perceptual hashing: Evaluate pHash/aHash for near‑duplicate detection as a post‑MVP enhancement.
- Browser helper feasibility: Explore risk/ToS and technical path for future near‑real‑time ingestion.

## Appendices

### A. Research Summary

- Source: docs/brainstorming-session-results.md (2025-09-21).
- Decisions & Constraints (MVP): Next.js + TS + Tailwind + shadcn/ui; Prisma + Supabase (EU); export‑first ingestion; HITL review; optional name redaction; Ubuntu desktop export unavailable.
- Key Themes: Per‑project WhatsApp groups; STT/OCR feed a unified extractor with confidence; monthly pack with traceability; EU‑friendly handling.
- Priorities: (1) Zip import + normalization, (2) Extraction + HITL review, (3) Evidence pack with exceptions.
- Backend Outline: Entities for project/import/message/media/transcript/ocr/extracted_item/rate_card/pack/audit; RLS by org/project; signed URLs.
- Open Questions: Bot‑in‑group feasibility; accuracy thresholds; redaction defaults.

### B. Stakeholder Input

- Users: PM and CM for MVP.
- Rates: EUR; rate cards live in Excel — CSV mapping as truth; AI parsing optional with HITL.
- Pack Layout: Mirror current dayworks where possible; reference file available.
- Retention: Out of scope for MVP; manual purge by batch acceptable.
- Dev Style: AI‑assisted, TypeScript‑first; stack should be DX‑friendly.
- Large Files: 100–600 MB exports expected; design for streaming and background processing.

### C. References

- docs/brainstorming-session-results.md
- whatsapp-data/dayworks-sheets/Evo Civils Eden Dayworks April rev1.pdf
- whatsapp-data/zip/ (sample exports for development/testing)

## Next Steps

### Immediate Actions

1. Scaffold monorepo (Next.js app + worker) with shared types and basic auth.
2. Implement resumable upload to EU storage and streaming unzip/parse on worker; persist parsed messages/media refs.
3. Define Prisma schema and RLS; seed scripts + synthetic export generator.
4. Build import wizard (project/month confirm; iOS/Android/desktop variants) with polling job status.
5. Draft review UI (keyboard‑first) showing source snippet/media thumb, confidence bars, approve/reject with audit trail.
6. Implement CSV rate import (minimal schema + unit normalization) and application in totals.
7. Generate basic PDF/CSV pack (date‑sectioned, page totals, traceability appendix, signature placeholders).
8. Add telemetry (ingestion duration, counts, dedup) and “cost sandbox” flags; quotas/caps.
9. Set NFR checks: 100 MB ingest P50 < 10 min (parse/index); no API > 30 s; retries + DLQ.

### PM Handoff

This Project Brief provides the full context for SiteProof. Please start in 'PRD Generation Mode', review the brief thoroughly to work with the user to create the PRD section by section as the template indicates, asking for any necessary clarification or suggesting improvements.
