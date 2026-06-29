---
phase: 5
title: "Applications-Matching-Booking"
status: pending
priority: P1
effort: "58d"
dependencies: [4, 6]
---

# Phase 5: Applications, Matching & Booking (VÒNG LẶP CỐT LÕI)

## Overview
**Trái tim của marketplace** — candidate apply → agency/brand match → booking được
xác nhận → ca làm diễn ra. Hiện tại đây chỉ là **scaffolding rỗng** và là gap lớn nhất:
không có nó thì không có thu nhập, không invoice, không payslip.

## Current State (đã verify)
- 🔴 **Applications: 0 records** dù có 9 candidates + 1 advert. Vòng lặp chưa từng chạy. (evidence: `07-applications-empty.png`)
- 🟡 Form tạo application tối giản: 3 listbox (advert / candidate / status) + "Actioned at". (evidence: `applications/create`)
- 🟡 Trang advert detail có cột đếm "Applications" và field "Accepted application" → relationship đã được model.
- 🔴 Không có matching, không có handshake offer/accept, không thấy booking entity, không có allocation, không có clash detection.

## Production-Grade Target
- **Application**: candidate→advert với status lifecycle (applied → shortlisted → offered →
  accepted/declined → withdrawn → rejected), timestamps, audit.
- **Matching**: lọc candidates đủ điều kiện (compliant, đúng role/type, available, trong phạm vi);
  ranking/suggestions tùy chọn; bulk invite.
- **Handshake offer/accept** giữa brand/agency và candidate (có expiry).
- **Booking/Assignment** entity = candidate↔advert(/slot)↔date(s) đã xác nhận: object mà timesheets,
  invoices, payslips đều bám vào.
- **Availability & clash detection** (không double-book một candidate trên các ca trùng giờ).
- Xử lý cancellation/no-show với policy (ảnh hưởng charges).
- Allocation theo headcount/slots của advert (P4).

## Feature Gap Matrix
| # | Feature | Current | Target | Gap |
|---|---------|---------|--------|-----|
| 5.1 | Application entity | 🟡 scaffold trơ | Full status lifecycle + audit | Logic, transitions, events |
| 5.2 | Candidate apply | 🔴 | Từ candidate portal | Phụ thuộc P3 |
| 5.3 | Matching/eligibility | 🔴 | Filter compliance+role+availability | Eligibility engine |
| 5.4 | Suggestions/ranking | 🔴 | Gợi ý candidate phù hợp nhất | Scoring (tùy chọn) |
| 5.5 | Handshake offer/accept | 🔴 | Two-sided confirm + expiry | Offer model + flow |
| 5.6 | Booking/Assignment entity | 🔴 | Object assignment đã xác nhận | **Core entity mới** |
| 5.7 | Multi-slot allocation | 🔴 | Fill N của N | Phụ thuộc P4.5 |
| 5.8 | Availability model | 🔴 | Candidate availability | Model mới |
| 5.9 | Clash detection | 🔴 | Ngăn double-booking | Overlap checks |
| 5.10 | Cancellation/no-show | 🔴 | Policy + tác động charge | Rules + hooks tới P8 |

## Build Scope (gap)
- Implement Application status lifecycle + events (chủ yếu là logic greenfield trên scaffold).
- **Giới thiệu Booking/Assignment entity** — keystone còn thiếu mà P7/P8/P9 phụ thuộc.
- Eligibility/matching engine (tiêu thụ compliance P6 + availability).
- Handshake offer/accept xuyên các surface admin/brand/candidate (P2/P3).
- Availability + clash detection.

## Risk Assessment
- **Gap rủi ro cao nhất.** Mọi thứ downstream (timesheets, invoices, payslips, payments, dashboards)
  đều vô nghĩa cho tới khi cái này tồn tại và đúng.
- Booking entity có thể có hoặc chưa có trong code (không thấy trong admin nav) — **confirm qua source**;
  nếu chưa có thì đây là model mới nền tảng đụng tới toàn bộ schema.
- Độ chính xác về tiền (charges, cancellations) nằm ở đây — phải kín kẽ + tested (P14).
