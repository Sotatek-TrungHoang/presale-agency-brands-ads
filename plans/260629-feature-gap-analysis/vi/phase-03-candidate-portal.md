---
phase: 3
title: "Candidate-Portal"
status: pending
priority: P1
effort: "70d (MVP 54d)"
dependencies: [1]
---

# Phase 3: Candidate-Facing Portal (responsive web)

## Overview
Toàn bộ web surface phía candidate. **Chưa tồn tại** — candidates đang được admin quản lý.
Đây là nơi đa số candidates dành thời gian (tìm shifts, chứng minh right-to-work, được trả lương),
nên là surface có traffic cao nhất và bắt buộc phải mobile-responsive.

## Current State (verified)
- 🟡 Candidate entity là model giàu nhất: 4 tabs (Personal / Identification / Work / Contracts),
  status + compliance, references action, login account. (evidence: `03-candidate-detail.png`, `applicants/create`)
- 🟡 Candidate statuses tồn tại (Incomplete / Pending Approval / Active) — ngụ ý một onboarding funnel
  hiện đang được hoàn tất *bởi admin*, không phải candidate.
- 🔴 Không có app phía candidate: no profile self-edit, no document upload, no job search, no apply,
  no booking accept, no timesheet, no payslip view.
- ⚠️ **NHƯNG — tín hiệu mạnh cho thấy một candidate capture flow đã tồn tại sẵn** *(black-box)*: bản ghi candidate chứa
  **video-verification + ID image + signed contract thật** (`evidences/blackbox/t2-candidate-identification-tab.png`,
  `t2-candidate-contracts-tab.png`). Dữ liệu đó không thể nhập bằng tay — nhiều khả năng có một pipeline onboarding/capture
  hướng candidate chạy bên ngoài admin. **Điều tra trước khi sizing phase này** — nếu candidate front-end
  đã tồn tại, phần lớn 3.2/3.4/3.6 co từ 🔴 xuống 🟡. Đây là ẩn số có giá trị cao nhất.

## Production-Grade Target
- **Onboarding wizard**: personal details, address, upload right-to-work evidence, qualifications,
  declarations, references → chuyển Incomplete → Pending → Compliant/Active.
- **Document upload** cho từng Required Evidence item (passport, visa, DBS, v.v.) kèm status feedback.
- **Browse/search adverts**: filter theo role, type, location, date; xem pay rate (net of candidate charge).
- **Apply** vào adverts; track application status.
- **Accept/decline bookings**; set availability/calendar.
- **Shift management**: xem upcoming shifts, **clock-in/out hoặc submit timesheet** (P7).
- **Payslips**: list + download PDF (P8).
- Notifications (offers, compliance reminders, shift reminders) (P10).
- Profile completeness meter + compliance status.

## Feature Gap Matrix
| # | Feature | Current | Target | Gap |
|---|---------|---------|--------|-----|
| 3.1 | Candidate dashboard | 🔴 | Status, next shift, action items | New responsive surface |
| 3.2 | Onboarding wizard | 🟡 admin-driven | Candidate self-serve funnel | Multi-step UI + state machine |
| 3.3 | Profile self-edit (4 sections) | 🟡 admin-only | Candidate edit Personal/Work | Scoped forms |
| 3.4 | Right-to-work doc upload | 🔴 (config rỗng) | Upload theo Required Evidence | Phụ thuộc P6 |
| 3.5 | Declarations | 🔴 (config rỗng) | Sign required declarations | Phụ thuộc P6 |
| 3.6 | References | 🟡 admin action | Candidate submit referees, hệ thống thu thập | Reference workflow + emails |
| 3.7 | Job search/browse | 🔴 | Filterable advert list | Search + public advert view |
| 3.8 | Apply to advert | 🔴 | One-tap apply + track | Phụ thuộc P5 |
| 3.9 | Accept/decline booking | 🔴 | Respond to offers | Phụ thuộc P5 |
| 3.10 | Availability calendar | 🔴 | Set available dates | Availability model |
| 3.11 | Timesheet / clock | 🔴 | Submit hours mỗi shift | Phụ thuộc P7 |
| 3.12 | Payslips | 🔴 | List + PDF | Phụ thuộc P8 |
| 3.13 | Notifications | 🔴 | Offers/reminders | Phụ thuộc P10 |

## Build Scope (the gap)
- App web candidate responsive mới (mobile-first).
- Onboarding state machine điều khiển các status Incomplete→Pending→Active sẵn có theo hướng self-service.
- Secure document upload gắn với compliance rules (P6).
- Job search + apply + offer-response gắn với P5.
- Availability model (mới).

## Risk Assessment
- Chất lượng mobile-responsive quan trọng nhất ở đây (candidates dùng điện thoại) — tập trung design effort.
- Right-to-work UX nhạy cảm về pháp lý; phải khớp với P6 compliance rules trước khi ship.
