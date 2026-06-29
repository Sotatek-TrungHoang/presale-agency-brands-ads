---
title: "Tidal-Yedi Feature Gap Analysis: Current vs Production-Grade"
description: "Feature-by-feature inventory of the existing Laravel+Filament staffing admin vs a production-grade two-sided staffing marketplace. Documents current state, target, and concrete gap per domain. No cost estimation."
status: pending
priority: P1
branch: "main"
tags: [gap-analysis, presale, staffing-marketplace, filament]
blockedBy: []
blocks: []
created: "2026-06-29T03:31:12.698Z"
createdBy: "ck:plan"
source: skill
---

# Tidal-Yedi Feature Gap Analysis: Current vs Production-Grade

## Tổng quan

Phân tích gap theo từng feature của nền tảng staffing agency **Tidal/Yedi** do client tự
xây. Mục tiêu: thay cho kiểu nói chung chung "build nốt phần còn lại" bằng một inventory cụ
thể — **cái gì đang tồn tại hôm nay (đã verify live), cái gì production-grade yêu cầu, và gap
chính xác là gì.**

- **System đang phân tích:** Laravel + **Filament** admin panel, `admin.tidalagency.co.uk`.
- **Business model:** two-sided recruitment marketplace (Brands ↔ Candidates), agency lấy
  commission cả hai phía (Brand Charge %, Candidate Charge %).
- **Evidence base:** `tidal-teardown-live-findings.md`, `tidal-agency-analysis.md`,
  `evidences/*.png` (11 screenshot), cộng với deep live crawl mọi resource create/edit
  form và trang System settings ngày 2026-06-29.
- **Scope note:** tài liệu này **chỉ là feature/gap map** — không có man-day hay con số chi phí
  (theo yêu cầu). Đây là input cho một pass estimation/roadmap sau này.

## Cách đọc tài liệu này

Mỗi phase = một **feature domain**. Bên trong mỗi phase:
- **Current State (verified)** — cái gì thực sự chạy được trong live app, kèm evidence refs.
- **Production-Grade Target** — một nền tảng staffing thật, bán được phải làm gì.
- **Feature Gap Matrix** — từng dòng `Feature | Current | Target | Gap`.
- **Build Scope (the gap)** — khối lượng công việc cụ thể mà gap đó kéo theo.

**Maturity legend (theo từng feature):**
- ✅ **Done** — đã tồn tại & dùng được như hiện trạng.
- 🟡 **Partial** — scaffold/data-model có nhưng logic, UI, hoặc flow chưa hoàn chỉnh.
- 🔴 **Missing** — chưa có; build greenfield.

## Headline finding

Nền tảng có **data-model + admin-CRUD spine vững chắc** nhưng **các transactional
muscle và toàn bộ self-service surface đều vắng mặt**:

- ✅ **Spine có sẵn:** Advert model phong phú (pricing/charges/shifts/address), Candidate
  model sâu (profile 4-tab, references, compliance status), Brand model, admin RBAC, **và
  System settings được config đầy đủ** (charge %, references-required, invoice bank/terms,
  applicant + advertiser **contract templates**). Record Brand & Candidate đã mang sẵn
  **login user accounts** (email + password).
- 🔴 **Core transaction loop là scaffolding rỗng:** list Applications, Invoices, Payslips đều
  **0 records** — matching → booking → invoice → payslip chưa từng chạy. Đây là các Filament
  resource rỗng, không phải feature hoàn chỉnh.
- 🔴 **Không có self-service:** không có Brand portal, không có Candidate portal — account tồn
  tại nhưng không có chỗ nào để họ log in. Mọi thứ đều do admin vận hành.
- 🔴 **Compliance chưa config:** bảng Required Evidence + Declarations rỗng; Job Roles =
  chỉ có "Any Role". Compliance *engine* (rules, enforcement, right-to-work) chưa được build.
- 🔴 **Không có multi-tenancy:** single Tidal deployment; Yedi không verify được; không có lớp white-label.

## Phases (feature domains)

| Phase | Domain | Current maturity | Headline gap | Priority |
|-------|--------|------------------|--------------|----------|
| 1 | [Identity-Auth-Access](./phase-01-identity-auth-access.md) | 🟡 Admin auth + accounts exist | No portal auth, no reset/verify/2FA, partial RBAC | P1 |
| 2 | [Brand-Portal](./phase-02-brand-portal.md) | 🔴 None | Entire brand self-service surface | P1 |
| 3 | [Candidate-Portal](./phase-03-candidate-portal.md) | 🔴 None | Entire candidate self-service surface | P1 |
| 4 | [Adverts-Job-Lifecycle](./phase-04-adverts-job-lifecycle.md) | 🟡 Rich model, manual status | Approval flow, publish, search, geo | P2 |
| 5 | [Applications-Matching-Booking](./phase-05-applications-matching-booking.md) | 🔴 Empty scaffold | The core marketplace loop | P1 |
| 6 | [Compliance-RightToWork](./phase-06-compliance-righttowork.md) | 🟡 Status fields only | Rules engine, evidence enforcement, RTW | P1 |
| 7 | [Timesheets-Attendance](./phase-07-timesheets-attendance.md) | 🔴 None | Shift confirmation → hours → pay basis | P1 |
| 8 | [Billing-Invoices-Payslips](./phase-08-billing-invoices-payslips.md) | 🟡 Config only, 0 docs | Generation engine + PDF + lifecycle | P1 |
| 9 | [Payments-Settlement](./phase-09-payments-settlement.md) | 🔴 None | Collection + payout + reconciliation | P2 |
| 10 | [Notifications-Comms](./phase-10-notifications-comms.md) | 🔴 None visible | Transactional email/SMS/in-app | P2 |
| 11 | [Documents-Contracts-Esign](./phase-11-documents-contracts-esign.md) | 🟡 Templates exist | Generation + e-signature + storage | P2 |
| 12 | [MultiTenancy-WhiteLabel](./phase-12-multitenancy-whitelabel.md) | 🔴 Single deploy | Tenant isolation + branding/config | P2 |
| 13 | [Reporting-Analytics](./phase-13-reporting-analytics.md) | 🟡 Basic dashboard | Operational + financial reporting | P3 |
| 14 | [NonFunctional-Security-QA](./phase-14-nonfunctional-security-qa.md) | 🔴 Unknown/none | Security, GDPR, tests, CI/CD, perf | P1 |

## Dependencies

Build-order coupling (không phải cross-plan):
- P5 (Matching/Booking) là spine; P7 (Timesheets) → P8 (Billing) → P9 (Payments) nối chuỗi từ nó.
- P1 (Auth) gate P2/P3 (portals).
- P6 (Compliance) gate P5 (chỉ candidate compliant mới được book).
- P12 (Multi-tenancy) mang tính cross-cutting — rẻ nhất nếu quyết trước khi làm UI P2/P3.

## Open questions (chặn việc scope chính xác)

1. **Source code access** — mức độ hoàn chỉnh *backend* (controllers/services) của các module
   rỗng chưa biết nếu không có repo. Một số 🔴 có thể là 🟡.
2. **Yedi** — URL/login chưa được cung cấp; giả định multi-tenant vs separate-deploy chưa verify.
3. **Scope của "full app"** đã xác nhận trước đó: Brand + Candidate **web** portal (không có native mobile),
   giữ & mở rộng Laravel/Filament, single multi-tenant, semi-automated payments.
