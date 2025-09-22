Brainstorming Session Results

**Session Date:** 2025-09-21
**Facilitator:** Business Analyst Mary
**Participant:** You

## Executive Summary

- **Topic:** SaaS web app to turn WhatsApp voice/text/media from construction site groups into structured daywork data and produce a monthly evidence pack for payment.
- **Session Goals:** Explore broadly, then narrow to an MVP.
- **Techniques Used:** How Might We (in progress)
- **Total Ideas Generated:** TBD

### Decisions & Constraints (MVP)
- Frontend-first build with mock data, then connect backend.
- UI stack: Next.js + TypeScript, TailwindCSS, ShadCN UI.
- Backend: Prisma + Supabase (EU region/Ireland). Managed STT/OCR in EU.
- Ingestion: Primary = WhatsApp export (.zip); research “bot-in-group” later; optional browser helper.
- Redaction: Keep names by default, provide toggle (choice 4‑B).
- Constraint: WhatsApp Web does NOT export zip; use WhatsApp Desktop (Windows/macOS) or phone export for MVP. Ubuntu Desktop currently blocked for you.

### Key Themes Identified
- WhatsApp groups exist per project; daily voice/text/video/picture posts.
- Target users: one contract manager and two project managers (EU/Ireland).
- Pain: Manually combing messages to build dayworks sheets is slow and error‑prone.
- MVP form factor: Web app; STT (speech‑to‑text) required; EU-friendly data handling.
- Outputs: Monthly dayworks evidence pack (PDF/CSV) with traceable links to messages/media.

## Technique Sessions

### How Might We (ongoing)
**Description:** Facilitate user‑driven idea generation around three chosen HMW prompts.

#### Prompts Selected
1. Turn ad‑hoc WhatsApp/text/voice into structured daywork entries automatically.
2. Extract Labour/Material/Plant with quantities, units, and rates from sloppy text.
3. Produce a monthly evidence pack (PDF + CSV) ready for submission in one click.

#### Ideas Generated

HMW 1 — Turn WhatsApp/text/voice into structured entries (chosen: 8,1,4,6)
1) Corrections queue after ingest: inline edit for fields; approve → locks item; reject → sends back to triage
2) Monthly chat export (.zip) per project; drag‑drop to import; auto‑parse `txt` and media folder
3) Bot-in-group (research feasibility): if unsupported, fall back to export or browser helper
4) Web dropzone for ad‑hoc voice notes/photos (outside chat); project/date auto-linking

HMW 3 — Extract L/M/P with qty/units/rates (chosen: 1,2,4,7; HITL required)
1) Domain dictionary for trades/units/materials/plant (e.g., “MEWP”, “6‑tonne digger”, “topsoil”)
2) Pattern library for common phrases (e.g., “2 men 4h”, “15 loads A30 dumper”)
3) Confidence scoring on each field; low confidence → flagged to human review queue
4) Few‑shot examples from real messages (text + OCR’d images + voice transcripts) to improve extraction
5) Human‑in‑the‑loop (HITL): every item shows source snippet + media; edits tracked with user + timestamp

HMW 9 — One‑click monthly evidence pack (chosen: 1,4,5,3)
1) PDF pack per project with line items + thumbnails; sectioned by date
2) Exceptions report listing all low‑confidence/unapproved items to fix before finalization
3) PM sign‑off flow: approve → generate sealed PDF; versioned; hash saved for audit
4) Traceability appendix: original message id, author (redacted option), timestamp, and media links

#### Insights Discovered
- WhatsApp groups already exist per project; MVP should embrace export/import first, research bot‑in‑group in parallel.
- Mandatory HITL validation step before items enter the pack; confidence‑driven workflow.
- EU hosting/Ireland preferable; data minimization and optional name redaction.

#### Notable Connections
- Voice notes → STT → same extraction pipeline as text; OCR for notebook photos → same pipeline.
- Pattern library + few‑shots + rate card mapping → quicker approval in HITL queue.

### Idea Burst (divergent)
Generate many concrete solution pieces to later shortlist.

- Ingestion modes: (A) WhatsApp export .zip, (B) browser helper for WhatsApp Web, (C) official Business API for 1:1 forward, (D) manual paste/upload
- Media handling: compute perceptual hashes for duplicate images/videos; store EU‑region only
- Project detection: infer from group export filename; allow override at import
- Time resolver: normalize “yesterday morning” using message timestamp and timezone
- L/M/P extractor: regex + ML hybrid; unit normalization (tonne, hr, load, m)
- Rate application: per‑project rate card; warn if missing rate; allow override
- Exceptions: missing qty/unit/rate → auto‑suggest from historical items
- Approvals UI: keyboard‑first; left pane sources, right pane structured fields with confidence bars
- Evidence pack builder: choose month range; show completion % and blockers (unapproved items)
- Audit trail: every edit stored with user/time; exportable change log

### Convergent (impact × effort snapshot)
- High impact / Low effort: zip import pipeline; HITL review UI; PDF pack without e‑signature
- High impact / Medium effort: OCR notebook photos; voice STT; confidence scoring; rate card mapping
- Medium impact / Medium effort: traceability appendix; exceptions report; CSV export
- Research‑dependent: bot‑in‑group ingestion (official vs third‑party)

### Proposed MVP (4–6 weeks)
- EU‑hosted web app (Ireland region)
- Import: WhatsApp export .zip per project; manual paste/upload for ad‑hoc
- Processing: STT for voice; OCR for images; hybrid L/M/P extraction with confidence
- HITL Review: corrections queue; approve/reject; track edits
- Outputs: monthly PDF pack + CSV + exceptions report; traceability appendix
- Admin: project setup, rate card, user roles (PM, Contract Manager)

### Frontend‑First Plan (with mock JSON)
- Screens: Import, Review Queue, Packs, Settings.
- Components (ShadCN): Table, Form, Dialog/Sheet, Badge, Progress, Tabs.
- Fixture endpoints (static JSON during MVP UI dev):
  - `/fixtures/projects.json`
  - `/fixtures/import_batches.json`
  - `/fixtures/review_queue.json`
  - `/fixtures/packs.json`
- Later: swap fixtures for Supabase API routes and storage.

### Fixture JSON Contracts (draft)
Projects
```json
[
  { "id": "p1", "name": "Prospect Building", "timezone": "Europe/Dublin" }
]
```

Import Batches
```json
[
  { "id": "ib1", "projectId": "p1", "status": "READY", "createdAt": "2025-09-15T10:00:00Z", "messages": 1245, "media": 312 }
]
```

Review Queue Items (excerpt)
```json
[
  {
    "id": "it_001",
    "projectId": "p1",
    "date": "2025-09-12",
    "category": "LABOUR",
    "activity": "Knock and load away wall",
    "fields": {
      "peopleCount": { "value": 3, "confidence": 0.86 },
      "hours": { "value": 1.5, "confidence": 0.92 },
      "unit": { "value": "hour", "confidence": 0.99 }
    },
    "sources": [
      { "type": "message", "ref": "msg1", "snippet": "7am-8:30am knock and load away wall" },
      { "type": "image", "ref": "IMG-20250912-WA0001.jpg", "thumbUrl": "/media/IMG-...thumb.jpg" }
    ],
    "reviewStatus": "PENDING"
  }
]
```

Packs
```json
[
  { "id": "pack_sep_2025_p1", "projectId": "p1", "month": "2025-09", "completion": 0.78, "pdfUrl": null, "csvUrl": null, "exceptions": 12 }
]
```

### Backend Outline (Prisma + Supabase)
- Entities: organization, user (Supabase Auth), project, import_batch, source_message, source_media,
  transcript, ocr_text, extracted_item, extracted_item_source, rate_card, rate, review_task,
  evidence_pack, pack_item, audit_event.
- Enums: ItemCategory[LABOUR|MATERIAL|PLANT|DELAY], ReviewStatus[PENDING|NEEDS_INFO|APPROVED|REJECTED].
- RLS: scope all tables by organization_id and project_id. Storage via signed URLs only.
- API routes (later): POST /imports, GET /review, PATCH /items/:id, POST /packs, GET /packs/:id.

### Ingestion Notes
- Primary: upload exported `.zip` with messages + media.
- Alternatives: manual paste of `.txt` with a `media/` folder; future browser helper.
- Bot-in-group: research feasibility and ToS risk; not critical for MVP.

### Open Questions / Research
- WhatsApp “bot‑in‑group” feasibility for Cloud API (likely unsupported) vs third‑party workarounds; risk/ToS
- Minimum accuracy thresholds to auto‑approve without human touch?
- Name redaction default for evidence packs?

### Sample Extraction Examples (from provided notes; names redacted)
Date context: September (year unspecified)

1) 12 Sep, 07:00–08:30 — Knock and load away wall, Plant: 225 excavator + A30 dumper
   - Labour: 3 workers × 1.5 h = 4.5 h
   - Plant: 225 digger 1.5 h; A30 dumper 1.5 h
   - Material moved: 3 loads stone (qty=3, unit=load)
   - Notes: Source shows team names; redacted; parsed from photo 1

2) 12 Sep, 08:30–19:00 — Dig out haul road prospect→site; load out stone
   - Labour: 3 workers × 10.5 h = 31.5 h (needs confirm)
   - Plant: 225 digger 10.5 h; A30 dumper 10.5 h
   - Material moved: 7 loads stone
   - Flags: long shift; confirm breaks; confirm 7 loads

3) 12 Sep, 07:00–09:30 — Waiting on permit (delay)
   - Labour: 2 workers × 2.5 h = 5 h (billable if dayworks allow waiting on instruction)
   - Category: Delay/Instruction; attach permit reference if available

4) 12 Sep, 10:00–13:30 — Dozer operation
   - Plant: Dozer 3.5 h
   - Labour: 1 operator × 3.5 h

5) 11 Sep, 15:00–17:00 — Move stockpile of clay up to berm; 15 loads via A30 dumper; load with 225
   - Labour: 3 workers × 2 h = 6 h
   - Plant: 225 digger 2 h; A30 dumper 2 h
   - Material moved: 15 loads clay

6) 03 Sep, 11:30–12:30 — Waiting on instruction (delay)
   - Labour: 3 workers × 1 h = 3 h
   - Category: Delay/Instruction; add client sign‑off if available

Schema used for examples (draft):
- project_id, date, start_time, end_time, activity, category(L/M/P/Delay),
  item_name, quantity, unit, rate_id?, people_count, equipment, description,
  source_ref (message_id/file path), confidence (0–1), review_status

## Action Planning

Top 3 Priority Ideas

#1 Priority: Zip Import + Normalization
- Rationale: Lowest friction to unblock data flow; high impact
- Next steps: Build parser for WhatsApp export `.txt` + media; project mapping; timestamp normalization
- Resources needed: Backend dev, sample exports, test harness
- Timeline: Week 1–2

#2 Priority: Extraction + HITL Review
- Rationale: Converts raw messages to daywork lines with auditability
- Next steps: Implement OCR/STT; L/M/P extractor; confidence + review queue; rate card mapping
- Resources needed: NLP/ML dev, domain dictionary, rate cards
- Timeline: Week 2–4

#3 Priority: Evidence Pack (PDF/CSV + Exceptions)
- Rationale: Delivers end user value monthly; closes loop
- Next steps: Pack builder UI; exceptions list; sign‑off flow; traceability appendix
- Resources needed: Backend + UI dev, template design
- Timeline: Week 4–6


---

*Session facilitated using the BMAD-METHOD brainstorming framework*
