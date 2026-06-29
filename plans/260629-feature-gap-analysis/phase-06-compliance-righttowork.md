---
phase: 6
title: "Compliance-RightToWork"
status: pending
priority: P1
effort: "52d"
dependencies: []
---

# Phase 6: Compliance & Right-to-Work

## Overview
UK staffing legally requires right-to-work + evidence checks before a candidate can work.
The platform has **status fields and empty config tables** but **no compliance engine** —
nothing enforces that a "Compliant" candidate actually has the required evidence.

## Current State (verified)
- 🟡 Candidate has Compliance status (Compliant / Non-compliant / Incomplete / Pending Approval) — a **label, not a computed gate**. (evidence: `05-candidates-list.png`)
- 🟡 Brand also has compliance status.
- 🟡 **Declarations** model schema exists: Title, Description, Time-to-complete, Required switch — but **0 records**. (evidence: `declarations/create`, `11-declarations-empty.png`)
- 🔴 **Required Evidence: 0 records** — the list of documents a candidate must provide is **not configured**. (evidence: `10-required-evidence-empty.png`)
- ✅ System setting: **References required = 2** (numeric policy exists).
- 🔴 No document upload, no verification workflow, no expiry tracking, no right-to-work integration,
  no enforcement linking evidence → compliance status → booking eligibility.

## Production-Grade Target
- **Required Evidence catalog** per candidate type/role (passport, visa/share-code, DBS, proof of address, NI, etc.),
  each with: mandatory flag, expiry, accepted file types.
- **Document upload + verification** workflow (submitted → under review → approved/rejected/expired) with reviewer queue.
- **Declarations** issued to candidates, signed/acknowledged, tracked.
- **References** collection (target = 2): referee invite → response capture → status.
- **Right-to-work check** (manual reviewer workflow now; optional Yoti/identity provider later).
- **Compliance status auto-computed** from evidence completeness + validity + references + declarations.
- **Enforcement gate**: only Compliant candidates can be offered/booked (ties to P5).
- Expiry monitoring + re-request reminders.

## Feature Gap Matrix
| # | Feature | Current | Target | Gap |
|---|---------|---------|--------|-----|
| 6.1 | Compliance status field | 🟡 manual label | Auto-computed gate | Rules engine |
| 6.2 | Required Evidence catalog | 🔴 empty | Configurable per role/type | Seed + schema (file types/expiry) |
| 6.3 | Document upload | 🔴 | Candidate uploads per item | Upload + storage (P3) |
| 6.4 | Verification workflow | 🔴 | Reviewer queue + states | Workflow + admin UI |
| 6.5 | Expiry tracking | 🔴 | Track + re-request | Scheduled checks (P10) |
| 6.6 | Declarations issue/sign | 🟡 schema only | Issue + capture acknowledgement | Records + signing flow |
| 6.7 | References collection | 🟡 admin action + policy=2 | Automated referee invites + capture | Reference workflow + emails |
| 6.8 | Right-to-work check | 🔴 | Manual workflow (+optional KYC) | Workflow now; integration later |
| 6.9 | Eligibility enforcement | 🔴 | Block non-compliant booking | Gate in P5 |
| 6.10 | Audit trail | 🔴 | Who verified what, when | Compliance audit log |

## Build Scope (the gap)
- Build the **compliance rules engine**: evidence catalog → per-candidate checklist → completeness
  computation → compliance status → booking eligibility.
- Reviewer queue + document verification states.
- References + declarations workflows.
- Manual right-to-work process now; keep an integration seam for Yoti/TrueLayer later (P9-adjacent).

## Risk Assessment
- **Legal/regulatory risk** if gating is wrong — a non-compliant candidate getting booked is a real
  liability. Enforcement must be hard, not advisory.
- "Compliant" today is a hand-set label on test data → the real rules are undefined; needs the client's
  actual evidence requirements as input.
