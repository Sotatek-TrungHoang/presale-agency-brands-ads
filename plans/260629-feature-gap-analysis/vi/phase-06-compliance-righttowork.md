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
compliance engine** — không gì enforce rằng candidate "Compliant" thực sự có đủ required evidence.

## Current State (đã verify — gồm cả black-box behavioral test)
- ✅ **Evidence capture thật TỒN TẠI trong data** *(black-box, candidate id=2)*: tab Identification hiển thị
  **Photograph, Evidence of ID (image), Video verification (video player), ID number**; tab Contracts hiển thị một
  **"Tidal Employment Contract" đã ký + file link**. (evidence: `evidences/blackbox/t2-candidate-identification-tab.png`, `t2-candidate-contracts-tab.png`)
  ⚠️ Điều này nghĩa là một **pipeline onboarding/capture của candidate đã chạy ở đâu đó** (nhiều khả năng là một flow candidate-facing) — cần điều tra; có thể giảm scope ở đây và ở P3.
- ✅ **Required Evidence CRUD hoạt động** — create là một **modal/slide-over** (Title, Time-to-complete, Required), có persist. ("`/create` rỗng" trước đó chỉ vì nó dựa trên modal.) (evidence: `t2-required-evidence-modal.png`, `t2-required-evidence-created.png`)
- ✅ **References workflow tồn tại** — modal "Update references": repeater gồm Name/Telephone/Email/Status (vd "Sent to Referee"). System policy **References required = 2**. (evidence: `t2-update-references-modal.png`)
- 🟡 Compliance status (Compliant / Non-compliant / Incomplete / Pending Approval) được set thủ công qua "Update status" — là một **label, không phải computed gate**.
- 🔴 **Declarations create BỊ LỖI (production bug)** — required Upload field fail ở server-side (Livewire temp-upload error `data.upload_id… failed to upload`); không thể tạo bất kỳ declaration nào. (evidence: `t2-declaration-upload-error.png`)
- 🔴 **Không có enforcement** — không có bằng chứng UI nào cho thấy candidate non-compliant bị chặn khỏi booking; compliance chỉ mang tính khai báo.
- 🔴 Không có rules engine (evidence catalog → checklist → computed status), không track expiry, không có right-to-work integration.

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
| 6.2 | Required Evidence catalog | 🟡 CRUD works (modal), chưa config | Configurable theo role/type | Seed + theo role/type + file types/expiry |
| 6.3 | Document upload | 🟡 capture đã có trong data | Candidate upload từng item | Confirm/mở rộng pipeline hiện có (P3) |
| 6.4 | Verification workflow | 🔴 | Reviewer queue + states | Workflow + admin UI |
| 6.5 | Expiry tracking | 🔴 | Track + re-request | Scheduled checks (P10) |
| 6.6 | Declarations issue/sign | 🔴 create BỊ LỖI (upload bug) | Issue + capture acknowledgement | Fix upload bug + issue/sign flow |
| 6.7 | References collection | 🟡 workflow tồn tại, policy=2 | Referee invite tự động + capture | Tự động hóa invites/capture (base đã có) |
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
