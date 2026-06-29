# Báo giá ước tính — Tidal / Yedi Agency Platform

> Tài liệu presale Sotatek. Ước tính chi phí & thời gian để phát triển nền tảng từ admin panel hiện có thành "full app".
> Ngày lập: 2026-06-29 · Trạng thái: **đã cập nhật sau bóc tách trực tiếp**
> Bằng chứng & teardown: `tidal-teardown-live-findings.md` + thư mục `evidences/`

---

## 1. Bối cảnh

Khách hàng tự xây một **admin panel** (Laravel + **Filament**) cho một **agency tuyển dụng & cung ứng nhân sự ngành bán lẻ cao cấp (luxury retail staffing)** tại UK.

- Bản chất là **marketplace hai chiều**: Brands (Creed, Kering) ↔ Candidates. Tidal ăn hoa hồng hai đầu (Brand Charge 15%, Candidate Charge 5%).
- **Tidal (thị trường UK)** và **Yedi (thị trường khác)** = "cùng một nền tảng, khác thị trường".
- Sản phẩm hiện tại **chỉ là admin nội bộ** — chưa có portal cho Brand, chưa có app cho Candidate.
- **Bóc tách trực tiếp (2026-06-29):** data model + admin CRUD (Advert/Candidate/Brand + dashboard tài chính) **đã có thật**; nhưng lõi giao dịch (Applications/Invoices/Payslips) và compliance (Required Evidence/Declarations) **mới là scaffold rỗng, chưa cài logic** → phần lõi là build mới. Chi tiết: `tidal-teardown-live-findings.md`.

Mục tiêu: biến nền tảng thành **full app** với portal tự phục vụ cho cả hai phía.

---

## 2. Phạm vi đã chốt (Phase 1)

| Hạng mục | Quyết định |
|---|---|
| Bề mặt (surfaces) | **Brand web portal** + **Candidate web portal** (responsive, chưa làm native mobile) + mở rộng admin hiện có |
| Backend | **Giữ lại & mở rộng** code Laravel + Filament của khách |
| Đa thị trường | **Multi-tenant một codebase**, white-label (Tidal + Yedi), Tidal go-live trước |
| Thanh toán & compliance | **Bán tự động** (auto sinh PDF invoice/payslip, settlement thủ công, compliance review có quy trình nhưng review thủ công) |

---

## 3. Hiện trạng thanh toán & compliance + khuyến nghị

**Hiện trạng (từ phân tích admin):**
- **Có sẵn data model tính tiền, chưa có settlement.** Mỗi advert đã lưu `Brand Pay Rate`, `Brand Charge %` (15), `Candidate Charge %` (5) → đầu vào để tính tiền đã có. Nhưng **module Invoices và Payslips đang trống** → chưa có engine sinh hóa đơn, chưa có cổng thanh toán, đang xử lý tay offline.
- **Compliance dạng checklist & thủ công.** Module `Required Evidence` + `Declarations` = upload giấy tờ + người duyệt. **Chưa có tích hợp xác minh right-to-work / ID tự động.**

**Khuyến nghị: chọn "Bán tự động" cho Phase 1** (không làm full automation ngay):
- **Auto sinh PDF invoice/payslip** từ data rate/charge đã có → xử lý đúng nỗi đau thủ công lớn nhất với chi phí thấp. ROI cao.
- **Giữ settlement offline** (BACS/chuyển khoản), hoặc thêm GoCardless ở Phase 2. Volume ban đầu thấp → cổng thanh toán là chi phí + gánh nặng vận hành tài chính/PCI chưa cần thiết.
- **Giữ compliance review thủ công** nhưng có quy trình. Luật UK cho phép kiểm tra right-to-work thủ công hợp lệ; vendor KYC trả phí (Yoti/TrueLayer) phát sinh phí theo lượt + procurement + rủi ro tích hợp. Để dành Phase 2 khi volume đủ lớn.
- Tóm lại: đạt ~80% giá trị (hết phải tính tay invoice/payslip, có hàng đợi compliance) với ~40% chi phí của "full automation".

---

## 4. Bóc tách khối lượng (man-days)

| Workstream | Days |
|---|---:|
| **0. Discovery + audit code** của khách (chất lượng, độ hoàn thiện, khả năng tái sử dụng) | 10 |
| **1. Foundation** — multi-tenancy, cấu hình white-label, lớp API (Sanctum), RBAC cho 3 loại user | 33 |
| **2. Brand portal** — onboarding, đăng/quản lý advert, duyệt ứng viên, bookings, invoices | 40 |
| **3. Candidate portal** — onboarding, upload evidence/right-to-work, tìm & ứng tuyển, nhận booking, timesheet, payslip | 42 |
| **4. Hoàn thiện lõi giao dịch (Filament)** — applications/matching, booking lifecycle, status workflow, quản lý tenant *(build mới — module đang là scaffold rỗng)* | 35 |
| **4b. Compliance engine** — Required Evidence rules + checklist enforcement, Declarations, right-to-work manual workflow *(chưa cấu hình/chưa có logic)* | 18 |
| **5. Billing engine** — auto sinh PDF invoice + payslip từ model rate/charge | 20 |
| **6. UI/UX design** — 2 portal, design system white-label | 20 |
| **7. QA / testing** | 32 |
| **8. DevOps** — multi-env, CI/CD, deployment | 12 |
| **9. PM / điều phối** (~13% overlay) | 32 |
| **Tạm tính** | **294** |
| **Dự phòng (~12%)** | ~36 |
| **Tổng** | **~330 man-days** |

> **Thay đổi sau bóc tách:** WS4 tăng 25→35, thêm WS4b (18) vì Applications/Invoices/Payslips/Required Evidence/Declarations là **scaffold rỗng chưa cài logic** (xác minh tại `evidences/07–11`), không phải "extend cái có sẵn".

---

## 5. Thời gian & nhân sự

**Team (~5–6 người):** 1 Tech Lead/BE, 1–2 Laravel/Filament BE, 2 FE, 1 QA, PM bán thời gian + UI/UX designer bán thời gian.

**Thời gian: ~5.5–6 tháng** (≈ 22–26 tuần), gồm ~2 tuần discovery/audit đầu kỳ:
- Tháng 1: discovery + audit + foundation (multi-tenancy, API, design system)
- Tháng 2–3: build lõi giao dịch (applications/booking/billing) + compliance engine + Brand portal
- Tháng 3–4.5: Candidate portal + quy trình compliance
- Tháng 5–6: integration, QA hardening, Tidal go-live; Yedi bật lên như một config của cùng codebase

---

## 6. Chi phí (Sotatek offshore, blended)

Với đơn giá blended **~$240–300/man-day**:

> ### Phase 1: **~US$80,000 – $99,000 · ~5.5–6 tháng**
> (~£63k–£78k)
>
> *Cao hơn ước tính ban đầu ($70k–$87k) vì lõi giao dịch + compliance là build mới, không phải extend — xác nhận qua bóc tách trực tiếp.*

**Phase 2 tùy chọn** (khi volume đủ lớn) — **+~$30k–$50k, +2–3 tháng:**
- Cổng thanh toán / payout tự động (GoCardless / Stripe Connect)
- Xác minh right-to-work / ID tự động (Yoti / TrueLayer)
- Native mobile app cho candidate (iOS/Android)
- Check-in/out ca làm trong app với geolocation, push notification

---

## 7. Rủi ro lớn nhất: audit code

Đã **xác thực độ hoàn thiện UI/flow** qua bóc tách trực tiếp (chắc chắn hơn nhiều so với chỉ đọc mô tả). Tuy nhiên **vẫn chưa thấy source code** → chất lượng code (sạch/rối) còn là biến số **±15–20%**. Audit ở Workstream 0:
- Code Laravel sạch, cấu trúc tốt → có thể về cận dưới.
- Code kiểu prototype, rối → một phần biến thành "rebuild", đẩy lên **>$100k** và ~6.5 tháng.
- Nếu code module trống đã có sẵn controller/service phần nào (chưa rõ tới khi xem source) → có thể tiết kiệm một phần WS4/4b.

**Khuyến nghị cách báo giá cho khách:** chốt **giá cố định cho 2 tuần audit** trước, sau đó mới ra **fixed-price chắc chắn** cho phần còn lại — bảo vệ cả hai bên.

---

## 8. Câu hỏi chưa giải quyết

1. **Đơn giá/ngày** — đang dùng $240–300/md. Có cần siết lại theo rate card presale thực tế hiện tại của Sotatek không?
2. **Xin source code (quyền Git)** để audit chất lượng code — biến số lớn nhất còn lại (±15–20%).
3. **URL + login Yedi** — khách mới gửi Tidal; chưa verify được deployment Yedi (đã thử đoán domain, không resolve).
4. Module trống trong code đã có controller/logic phần nào chưa, hay mới chỉ là Filament resource rỗng? (chỉ trả lời được khi có source).
