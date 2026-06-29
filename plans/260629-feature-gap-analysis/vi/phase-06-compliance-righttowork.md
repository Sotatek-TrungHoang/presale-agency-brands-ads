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
Staffing tại UK theo luật bắt buộc phải có right-to-work + kiểm tra evidence trước khi
candidate được làm việc. Platform có **status fields và config tables rỗng** nhưng **không có
compliance engine** — không gì enforce rằng candidate "Compliant" thực sự có đủ evidence yêu cầu.

## Current State (đã verify)
- 🟡 Candidate có Compliance status (Compliant / Non-compliant / Incomplete / Pending Approval) — chỉ là **label, không phải gate được tính toán**. (evidence: `05-candidates-list.png`)
- 🟡 Brand cũng có compliance status.
- 🟡 **Declarations** model schema có sẵn: Title, Description, Time-to-complete, Required switch — nhưng **0 records**. (evidence: `declarations/create`, `11-declarations-empty.png`)
- 🔴 **Required Evidence: 0 records** — danh sách documents mà candidate phải cung cấp **chưa được config**. (evidence: `10-required-evidence-empty.png`)
- ✅ System setting: **References required = 2** (policy số học có tồn tại).
- 🔴 Không có document upload, không có verification workflow, không track expiry, không có right-to-work integration,
  không có enforcement liên kết evidence → compliance status → booking eligibility.

## Production-Grade Target
- **Required Evidence catalog** theo candidate type/role (passport, visa/share-code, DBS, proof of address, NI, v.v.),
  mỗi cái có: mandatory flag, expiry, accepted file types.
- **Document upload + verification** workflow (submitted → under review → approved/rejected/expired) với reviewer queue.
- **Declarations** phát hành cho candidates, ký/acknowledge, được track.
- Thu thập **References** (target = 2): mời referee → capture phản hồi → status.
- **Right-to-work check** (manual reviewer workflow hiện tại; tùy chọn Yoti/identity provider sau).
- **Compliance status auto-compute** từ độ đầy đủ evidence + tính hợp lệ + references + declarations.
- **Enforcement gate**: chỉ candidate Compliant mới được offer/book (gắn với P5).
- Giám sát expiry + reminder re-request.

## Feature Gap Matrix
| # | Feature | Current | Target | Gap |
|---|---------|---------|--------|-----|
| 6.1 | Compliance status field | 🟡 label thủ công | Gate auto-compute | Rules engine |
| 6.2 | Required Evidence catalog | 🔴 rỗng | Configurable theo role/type | Seed + schema (file types/expiry) |
| 6.3 | Document upload | 🔴 | Candidate upload từng item | Upload + storage (P3) |
| 6.4 | Verification workflow | 🔴 | Reviewer queue + states | Workflow + admin UI |
| 6.5 | Expiry tracking | 🔴 | Track + re-request | Scheduled checks (P10) |
| 6.6 | Declarations issue/sign | 🟡 chỉ schema | Issue + capture acknowledgement | Records + signing flow |
| 6.7 | References collection | 🟡 admin action + policy=2 | Referee invite tự động + capture | Reference workflow + emails |
| 6.8 | Right-to-work check | 🔴 | Manual workflow (+optional KYC) | Workflow hiện tại; integration sau |
| 6.9 | Eligibility enforcement | 🔴 | Block booking non-compliant | Gate ở P5 |
| 6.10 | Audit trail | 🔴 | Ai verify cái gì, khi nào | Compliance audit log |

## Build Scope (gap)
- Build **compliance rules engine**: evidence catalog → checklist theo candidate → tính toán
  completeness → compliance status → booking eligibility.
- Reviewer queue + document verification states.
- References + declarations workflows.
- Quy trình right-to-work thủ công hiện tại; giữ integration seam cho Yoti/TrueLayer sau (P9-adjacent).

## Risk Assessment
- **Rủi ro legal/regulatory** nếu gating sai — một candidate non-compliant được book là liability
  thật sự. Enforcement phải cứng, không phải advisory.
- "Compliant" hiện tại là label set tay trên test data → rules thật chưa định nghĩa; cần
  evidence requirements thực tế của client làm input.
