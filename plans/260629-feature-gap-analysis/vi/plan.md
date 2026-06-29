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

Phân tích gap theo từng feature của nền tảng staffing agency **Tidal/Yedi** do client tự xây.
Mục tiêu: thay thế kiểu nói mơ hồ "build nốt phần còn lại" bằng một inventory cụ thể —
**cái gì đã có hôm nay (đã verify live), production-grade yêu cầu gì, và gap chính xác là gì.**

- **Hệ thống phân tích:** admin panel Laravel + **Filament**, `admin.tidalagency.co.uk`.
- **Business model:** two-sided recruitment marketplace (Brands ↔ Candidates), agency ăn
  commission cả hai phía (Brand Charge %, Candidate Charge %).
- **Evidence base:** `tidal-teardown-live-findings.md`, `tidal-agency-analysis.md`,
  `evidences/*.png` (11 screenshot), deep live crawl mọi resource create/edit
  form và trang System settings, **cộng thêm một black-box behavioral test 4-agent**
  (`tidal-blackbox-test-findings.md`, `evidences/blackbox/*.png`, 47 screenshot) — tất cả 2026-06-29.
- **Các hiệu chỉnh từ black-box** (về behavioral, không chỉ UI): Application CRUD + aggregation
  logic *hoạt động* (không phải scaffold trống); invoice/payslip generation *xác nhận không có* (không có trigger ở đâu cả);
  candidate evidence capture (ID/video/contract) *là thật* → một candidate onboarding flow nhiều khả năng đã tồn tại;
  Declarations create đang *hỏng* (server upload bug); **admin KHÔNG có RBAC** (cái "role selector" trước đó là
  đọc nhầm trường Title — đã hiệu chỉnh).
- **Lưu ý scope:** tài liệu này chỉ là một **feature/gap map** — không có con số man-day hay cost
  (theo yêu cầu). Nó là input cho một bước estimation/roadmap sau này.

## Cách đọc tài liệu này

Mỗi phase = một **feature domain**. Trong mỗi phase:
- **Current State (verified)** — cái gì thực sự hoạt động trong app live, kèm evidence refs.
- **Production-Grade Target** — một staffing platform thật, bán được phải làm được gì.
- **Feature Gap Matrix** — từng dòng `Feature | Current | Target | Gap`.
- **Build Scope (the gap)** — công việc cụ thể mà gap đó hàm ý.

**Maturity legend (theo từng feature):**
- ✅ **Done** — đã có & dùng được luôn.
- 🟡 **Partial** — scaffold/data-model có nhưng logic, UI, hoặc flow chưa hoàn chỉnh.
- 🔴 **Missing** — chưa có; build greenfield.

## Headline finding

Nền tảng có một **xương sống data-model + admin-CRUD vững** nhưng **cơ bắp giao dịch
(transactional) và mọi bề mặt self-service đều vắng mặt**:

- ✅ **Xương sống có sẵn:** Advert model phong phú (pricing/charges/shifts/address), Candidate
  model sâu (profile 4-tab, references, **ID/video/contract evidence thật**, compliance status),
  Brand model, **và một System settings được cấu hình đầy đủ** (charge %, references-required,
  invoice bank/terms, applicant + advertiser **contract templates**). Brand & Candidate records
  mang theo **login user accounts**. Application CRUD + aggregation (advert.accepted_application,
  candidate counts) **hoạt động** — đã verify bằng black-box.
- 🔴 **Core transaction loop dừng sau Application:** Application persist + cập nhật relations,
  nhưng **không có Booking entity**, advert status **không** auto-transition, và **không có
  trigger invoice/payslip generation ở bất kỳ đâu** (đã verify — không chỉ là list trống).
- ⚠️ **Tín hiệu MỚI:** candidate records giữ video-verification thật + ID image + signed contract
  → một **candidate onboarding/capture flow nhiều khả năng đã tồn tại** bên ngoài admin. Cần điều tra —
  nó có thể giảm đáng kể scope P3/P6.
- 🔴 **Không có admin RBAC** (đã hiệu chỉnh): user form không có trường role/permission; identity nhiều khả năng
  polymorphic (các bảng applicant/advertiser/admin riêng).
- 🔴 **Không có self-service:** không có Brand portal, không có Candidate portal — accounts tồn tại nhưng không có
  chỗ nào để họ log in. Mọi thứ đều do admin vận hành.
- 🔴 **Compliance chưa cấu hình:** bảng Required Evidence + Declarations trống; Job Roles =
  chỉ có "Any Role". *Engine* compliance (rules, enforcement, right-to-work) chưa được build.
- 🔴 **Không có multi-tenancy:** single Tidal deployment; Yedi không verify được; không có lớp white-label.

## Phases (feature domains)

| Phase | Domain | Current maturity | Headline gap | Priority |
|-------|--------|------------------|--------------|----------|
| 1 | [Identity-Auth-Access](./phase-01-identity-auth-access.md) | 🟡 Admin auth + accounts có | Không có portal auth, không reset/verify/2FA, RBAC một phần | P1 |
| 2 | [Brand-Portal](./phase-02-brand-portal.md) | 🔴 Không có | Toàn bộ bề mặt brand self-service | P1 |
| 3 | [Candidate-Portal](./phase-03-candidate-portal.md) | 🔴 Không có | Toàn bộ bề mặt candidate self-service | P1 |
| 4 | [Adverts-Job-Lifecycle](./phase-04-adverts-job-lifecycle.md) | 🟡 Model phong phú, status thủ công | Approval flow, publish, search, geo | P2 |
| 5 | [Applications-Matching-Booking](./phase-05-applications-matching-booking.md) | 🔴 Scaffold trống | Core marketplace loop | P1 |
| 6 | [Compliance-RightToWork](./phase-06-compliance-righttowork.md) | 🟡 Chỉ có status fields | Rules engine, evidence enforcement, RTW | P1 |
| 7 | [Timesheets-Attendance](./phase-07-timesheets-attendance.md) | 🔴 Không có | Shift confirmation → hours → pay basis | P1 |
| 8 | [Billing-Invoices-Payslips](./phase-08-billing-invoices-payslips.md) | 🟡 Chỉ config, 0 docs | Generation engine + PDF + lifecycle | P1 |
| 9 | [Payments-Settlement](./phase-09-payments-settlement.md) | 🔴 Không có | Collection + payout + reconciliation | P2 |
| 10 | [Notifications-Comms](./phase-10-notifications-comms.md) | 🔴 Không thấy | Transactional email/SMS/in-app | P2 |
| 11 | [Documents-Contracts-Esign](./phase-11-documents-contracts-esign.md) | 🟡 Templates có | Generation + e-signature + storage | P2 |
| 12 | [MultiTenancy-WhiteLabel](./phase-12-multitenancy-whitelabel.md) | 🔴 Single deploy | Tenant isolation + branding/config | P2 |
| 13 | [Reporting-Analytics](./phase-13-reporting-analytics.md) | 🟡 Dashboard cơ bản | Operational + financial reporting | P3 |
| 14 | [NonFunctional-Security-QA](./phase-14-nonfunctional-security-qa.md) | 🔴 Unknown/không có | Security, GDPR, tests, CI/CD, perf | P1 |

## Dependencies

Build-order coupling (không phải cross-plan):
- P5 (Matching/Booking) là xương sống; P7 (Timesheets) → P8 (Billing) → P9 (Payments) nối tiếp từ nó.
- P1 (Auth) gate P2/P3 (portals).
- P6 (Compliance) gate P5 (chỉ candidate compliant mới được booked).
- P12 (Multi-tenancy) là cross-cutting — rẻ nhất nếu quyết trước khi làm UI P2/P3.

## Open questions (chặn việc scope chính xác)

1. **Source code access** — độ hoàn chỉnh của *backend* các module trống (controllers/services)
   chưa biết nếu không có repo. Một số 🔴 có thể là 🟡.
2. **Yedi** — URL/login chưa được cung cấp; giả định multi-tenant vs separate-deploy chưa verify.
3. **Scope của "full app"** đã xác nhận trước đó: Brand + Candidate **web** portals (không native mobile),
   giữ & mở rộng Laravel/Filament, single multi-tenant, semi-automated payments.
