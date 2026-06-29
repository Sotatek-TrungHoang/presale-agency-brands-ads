# Tidal / Yedi — Ước tính effort (map từ feature-gap analysis)

> Ước tính bottom-up suy ra từ `plans/260629-feature-gap-analysis/` (14 feature domain, ~110 feature).
> Mỗi phase được cost từ gap matrix của chính nó, không phải đoán top-down.
> Ngày: 2026-06-29 · Đơn giá blended giả định: **$240–300/man-day** (chờ rate card Sotatek).

## TL;DR (tiếng Việt)

- Ước tính **bottom-up theo từng feature** (không phải áng chừng). Tổng **production-grade đầy đủ** (cả 14 domain) ≈ **~870 man-days**.
- Bản **MVP để Tidal go-live** (cắt phần tự động hoá thanh toán, e-sign, reporting sâu, Yedi đầy đủ) ≈ **~660 man-days**.
- **Con số này CAO HƠN ~2× báo giá thô trước ($80–99k)** — vì báo giá thô đã *thiếu*: lõi giao dịch (P5), engine compliance (P6), engine billing (P8), và gần như bỏ qua toàn bộ non-functional production (P14: security/GDPR/test/CI-CD) + multi-tenancy (P12). Đây chính là điều anh đã chỉ ra. Bản bottom-up này mới là con số đáng tin.

## Method & assumptions

- **Unit:** man-day (md) của một engineer Laravel/Filament + frontend đủ năng lực.
- **Costing rule:** feature 🟡 partial size theo *finish-the-gap*; 🔴 missing theo *full build*; ✅ loại trừ.
- **Cross-phase overlap** (vd timesheet UI trong portal vs engine trong P7) hấp thụ vào contingency, không double-count trong tổng.
- **Loại trừ** chi phí run/hosting thường xuyên, phí license bên thứ ba (Twilio/GoCardless/DocuSign/Yoti), và support sau launch.
- **Rate** là placeholder — xác nhận rate card presale Sotatek trước khi báo giá client.

## Effort theo từng phase

| Phase | Domain | Total md | MVP-launch md | Notes / MVP defer gì |
|-------|--------|---------:|--------------:|--------------------------|
| 1 | Identity, Auth & Access | 35 | 26 | Defer 2FA, impersonation, full brand multi-user |
| 2 | Brand Portal | 60 | 44 | Defer messaging, multi-seat, calendar polish |
| 3 | Candidate Portal | 70 | 54 | Defer availability calendar, UX polish |
| 4 | Adverts & Job Lifecycle | 40 | 26 | Defer geo-search, multi-slot (nếu single-hire OK), templates |
| 5 | **Applications-Matching-Booking (core)** | 58 | 58 | Không — spine không thể thương lượng |
| 6 | **Compliance & Right-to-Work** | 52 | 52 | Không — cổng pháp lý |
| 7 | Timesheets & Attendance | 35 | 35 | Không — billing basis |
| 8 | Billing — Invoices & Payslips | 55 | 44 | Defer accounting export, credit notes, statements |
| 9 | Payments & Settlement | 32 | 8 | MVP = chỉ offline tracked; automate sau |
| 10 | Notifications & Comms | 30 | 18 | MVP = transactional email + key SMS; defer in-app centre/preferences |
| 11 | Documents, Contracts & E-sign | 27 | 15 | MVP = contract gen + storage; defer e-sign provider |
| 12 | Multi-Tenancy & White-Label | 39 | 25 | MVP = tenant-ready foundation + Tidal; full Yedi enablement sau |
| 13 | Reporting & Analytics | 26 | 8 | MVP = wire dashboard + basic ops KPIs |
| 14 | **Non-Functional (security/GDPR/QA/CI-CD)** | 66 | 48 | MVP = security + core tests + CI/CD + GDPR basics; defer adv. perf |
| | **Dev subtotal** | **625** | **461** | |

## Cross-cutting overlays (cộng thêm lên trên)

| Item | md (cả hai scope) |
|------|-----------------:|
| Discovery + source-code audit | 10 |
| UI/UX design + design system (2 portal) | 25 |
| Foundation (API layer, project/env scaffolding, CI baseline) | 15 |
| **Fixed overlay subtotal** | **50** |

## Roll-up

| | Dev | +Overlay | +PM (~12%) | +Contingency (~15%) | **Total** |
|---|---:|---:|---:|---:|---:|
| **MVP launch (Tidal)** | 461 | 511 | ~572 | ~658 | **~660 md** |
| **Full production-grade (cả 14)** | 625 | 675 | ~756 | ~870 | **~870 md** |

> Contingency đặt ở **15%** (trên mức 12% thông thường) vì **source code vẫn chưa được thấy** — một số 🔴 có thể co lại thành 🟡 (tiết kiệm), nhưng các module rỗng cũng có thể giấu backend thiếu (tốn thêm). Audit (WS0) thu hẹp dải này.

## Timeline & team

**Team ~6:** 1 Tech Lead/BE, 2 Laravel/Filament BE, 2 FE, 1 QA + part-time PM & UI/UX.
Giả định ~5 md hiệu quả mỗi working-day sau khi trừ overhead phối hợp.

| Scope | Effort | Throughput | Realistic timeline |
|-------|-------:|-----------|--------------------|
| **MVP launch** | ~660 md | ~5 md/day | **~6.5–7.5 tháng** |
| **Full production-grade** | ~870 md | ~5 md/day | **~8.5–10 tháng** |

## Cost (blended $240–300/md)

| Scope | Low ($240) | High ($300) |
|-------|-----------:|------------:|
| **MVP launch (Tidal)** | **~$158k** | **~$198k** |
| **Full production-grade** | **~$209k** | **~$261k** |
| Enhancement tranche (Full − MVP) | ~$50k | ~$63k |

(MVP ≈ £125k–£156k; Full ≈ £165k–£206k ở tỷ giá ~0.79.)

## Recommended delivery shape

1. **Tranche 1 — MVP launch (Tidal):** ~660 md, ~$160k–$200k, ~7 tháng.
   Nền tảng two-sided thực sự giao dịch được: auth + cả hai portal + core loop (P5) + compliance (P6) +
   timesheets (P7) + billing generation (P8) + tenant-ready foundation (P12) + security/QA baseline (P14).
   Payments offline-tracked, comms email-first.
2. **Tranche 2 — Automation + Yedi + polish:** ~+210 md, ~$50k–$63k, ~+3 tháng.
   Payment automation (P9), e-sign (P11), in-app notifications (P10), accounting export + credit notes (P8),
   geo/multi-slot (P4), reporting depth (P13), full Yedi tenant enablement (P12), advanced security/perf (P14).

## Reconciliation với báo giá thô trước đó

| | Báo giá thô trước | Bottom-up này (MVP) |
|---|---|---|
| Method | top-down, ~9 workstream | per-feature trên 14 domain |
| Effort | ~330 md | ~660 md |
| Cost | $80k–$99k | $158k–$198k |
| Vì sao khác | under-scope P5/P6/P8; **bỏ sót P14** (security/GDPR/test/CI-CD) và coi P12 là tầm thường | mọi feature được đếm kèm gap của nó |

Con số trước **lạc quan và under-scoped** — lộ ra chính xác khi làm pass feature-level mà anh yêu cầu.

## Confidence & cái gì làm nó thay đổi

- **Dải ±15%** cho đến khi source audit (WS0). Audit + Yedi access siết lại thành firm fixed-price.
- Các yếu tố swing lớn nhất: (a) module rỗng có partial backend hay không (có thể cắt P5/P6/P8), (b) candidate
  employment model PAYE/umbrella/self-employed (chi phối độ phức tạp tax của P8/P9), (c) single-hire vs multi-slot
  advert (schema P4/P5), (d) Yedi code-divergence (P12).

## Open questions

1. Xác nhận **rate card** blended của Sotatek để thay placeholder $240–300.
2. **Source code + Yedi access** để thu hẹp dải ±15% và xác nhận giả định P5/P12.
3. Xác nhận **MVP scope** có phải là launch line đúng không, hoặc điều chỉnh deferral nào chuyển vào Tranche 1.
