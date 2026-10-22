# QA-Ready Batch — 78 Items With Primary QA Handover Docs
**Role:** INVESTIGATION (read-only audit) → QA batch brief
**Date:** 2026-10-22 | **Branch:** main @ 2dc8994
**Prepared by:** Investigation agent — no code, no doc edits

---

## ACCURATE COUNTS

| Category | Count | Note |
|----------|-------|------|
| Total GATE_5A (no QA yet) | 109 | after drift-fix of BUG-557 + BUG-595 |
| **With primary QA handover doc** | **78** | QA agent can execute immediately |
| Regression-mention only | 3 | BUG-462, BUG-464, BUG-499 — covered as regression checks inside other handovers |
| No doc at all | 28 | → see `QA_DEFERRED_NO_DOC_2026_10_22.md` |

---

## CREDENTIALS (all batches)

| Account | Password | Use for |
|---------|----------|---------|
| owner@cafe103.com | Qplazm@10 | Coupon, loyalty, cart, reports, PWA, sidebar |
| owner@palmhouse.com | Qplazm@10 | All PMS/hotel: check-in, folio, in-house, extend stay, checkout |
| owner@kunafamahal.com | Qplazm@10 | Role-based discount tests |
| cashier@kunafamahal.com | Qplazm@10 | Cashier discount path |
| owner@18march.com | Qplazm@10 | Room billing / paid_room |

Login: `POST https://preprod.mygenie.online/api/v1/auth/vendoremployee/login`
App: `https://core-pos-frontend-13.preview.emergentagent.com`

---

## BATCH QA-1 — nov_cr_batch: Active Table Toggle + Served Card Height
**2 items | Priority: P0 (both) | Risk: HIGH | Sprint: nov_cr_batch**

| ID | Title | QA Handover |
|----|-------|-------------|
| CR-429 | Active table toggle — show only occupied/active tables in table view | `QA_HANDOVER_CR429_2026_10_17.md` |
| CR-430 | Served items: decrease card height (remove 44px touch target on static items) | `QA_HANDOVER_CR430_TOGGLE_UI_2026_10_17.md` |

**Execute:** Open each handover file and run all test cases as written.

---

## BATCH QA-2 — oct_cr_batch: PWA + Reports + Auto-Bill
**7 items | Priority: P0–P2 | Risk: HIGH | Sprint: oct_cr_batch**

| ID | Priority | Title | QA Handover |
|----|----------|-------|-------------|
| BUG-597 | P0 | PWA White Screen / App Freeze on Every Production Deploy | `QA_HANDOVER_BUG597_2026_10_19.md` |
| BUG-353 | P1 | OrderReportBetaPage: Date range capped at 1 month (backend) | `QA_HANDOVER_BUG353_2026_10_19.md` |
| BUG-546 | P2 | Order Report: split payment settled orders show "Split Bill" | `QA_HANDOVER_BUG547_BUG546_2026_10_17.md` |
| BUG-547 | P1 | Channel and Payments doughnut chart displays amount as 0 | `QA_HANDOVER_BUG547_BUG546_2026_10_17.md` |
| BUG-556 | ? | Auto Bill: payment mode printed as "Unpaid" even with payment set | `QA_HANDOVER_BUG556_2026_10_16.md` |
| CR-435 | P2 | Sidebar: Remove Redundant Inventory Sub-Links | `QA_HANDOVER_CR435_2026_10_18.md` |
| CR-377 | P2 | Sales Report Complete Redesign — surface all 82 API fields | `QA_HANDOVER_CR377_2026_09_11.md` |

---

## BATCH QA-3 — oct_cr_batch: Dashboard Tablet Responsive
**2 items | Priority: HIGH (money-adjacent) | Risk: HIGH | Sprint: oct_cr_batch**
**Note:** Both covered in the SAME handover — run together.

| ID | Title | QA Handover |
|----|-------|-------------|
| CR-414 | Dashboard Tablet Responsive Redesign | `QA_HANDOVER_CR414_BUG537_2026_10_13.md` |
| BUG-537 | Dashboard Layout Broken on Android Tablets (Fixed Pixel Widths) | `QA_HANDOVER_CR414_BUG537_2026_10_13.md` |

**Needs:** Physical/emulated Android tablet at 600×960 and 768×1024 viewports.
**Note:** BUG-539 (ml-64/ml-16) is also noted in this handover as testing debt — check if it's still GATE_5A.

---

## BATCH QA-4 — oct_cr_batch: Coupon / Loyalty / Cart
**5 items | Priority: CRITICAL (money-adjacent) | Risk: CRITICAL | Sprint: oct_cr_batch**
**Important context:** CR-409 adds coupon pre-load; CR-413 REMOVES it. Final expected state: CPP opens with NO pre-loaded coupon. Test CR-409 + CR-413 together to verify final behaviour.

| ID | Title | QA Handover |
|----|-------|-------------|
| CR-408 | Coupon Auto-Apply Remove — Cashier Must Choose | `QA_HANDOVER_CR408_2026_10_11.md` |
| CR-409 | Scan Order Coupon Not Pre-Loaded in Collect Payment | `QA_HANDOVER_CR409_2026_10_11.md` |
| CR-411 | Cart Persists on Type/Table Switch, Clears Only on Dashboard Return | `QA_HANDOVER_CR411_2026_10_12.md` |
| CR-412 | Loyalty Pre-tick from Scan & Order (loyalty_info.redeemed=true) | `QA_HANDOVER_CR412_2026_10_12.md` |
| CR-413 | Remove CR-409 initialCoupon Pre-load from Collect Payment Panel | `QA_HANDOVER_CR413_2026_10_12.md` |

---

## BATCH QA-5 — oct_cr_batch: Room Discount Feature
**1 item | Priority: CRITICAL (money) | Risk: CRITICAL | Sprint: oct_cr_batch**
**Note:** CR-407 coverage is provided via BUG-489's handover (which tests CR-407 Sub-scope A discount on front-desk-v2).

| ID | Title | QA Handover |
|----|-------|-------------|
| CR-407 | Room Discount (check-in + checkout apply_to + partial_payments_room) | `QA_HANDOVER_BUG489_2026_10_05.md` |

---

## BATCH QA-6 — oct_bug_batch: PMS Check-In Wave A — GST & Discount Foundations
**13 items | Priority: CRITICAL (money/GST) | Risk: CRITICAL | Sprint: oct_bug_batch**
**Account:** palmhouse (has hotel/PMS features)
**Run in handover-date order** — earlier fixes are dependencies for later ones.

| ID | Title | QA Handover |
|----|-------|-------------|
| BUG-485 | Insights Dashboard: PDF/Excel download buttons have no onClick | `QA_HANDOVER_BUG485_2026_10_05.md` |
| BUG-487 | orderTransform roomInfo reads wrong key: discount_amount vs room_discount_amount | `QA_HANDOVER_BUG487_2026_10_05.md` |
| BUG-489 | CR-407 Sub-scope A: discount not visible on front-desk-v2 | `QA_HANDOVER_BUG489_2026_10_05.md` |
| BUG-490 | Room discount cap missing — can exceed balance due | `QA_HANDOVER_BUG490_BUG491_2026_10_05.md` |
| BUG-491 | In-House balance + discount display: Sub-A balance formula | `QA_HANDOVER_BUG490_BUG491_2026_10_05.md` |
| BUG-492 | Checkout Bill: Discount not reflected in total + no % cap | `QA_HANDOVER_BUG492_2026_10_06.md` |
| BUG-493 | pmsService: row.charge never set → chargeGst always 0 → BALANCE wrong | `QA_HANDOVER_BUG493_2026_10_06.md` |
| BUG-494 | FolioCheckoutPanel Bill: stale LR charge.* fields when check-in discount applied | `QA_HANDOVER_BUG494_2026_10_06.md` |
| BUG-495 | maxDiscount/maxPct formula ignores GST on advance | `QA_HANDOVER_BUG495_2026_10_06.md` |
| BUG-496 | maxPct formula wrong at check-in (floor formula revised) | `QA_HANDOVER_BUG496_497_500_501_2026_10_07.md` |
| BUG-497 | Collect Now max cap missing (CheckInForm + CheckInPage legacy) | `QA_HANDOVER_BUG496_497_500_501_2026_10_07.md` |
| BUG-500 | CheckInPage GST strip doesn't recalculate on discount | `QA_HANDOVER_BUG496_497_500_501_2026_10_07.md` |
| BUG-501 | Login page copyright year hardcoded '2025' — should be dynamic | `QA_HANDOVER_BUG496_497_500_501_2026_10_07.md` |

---

## BATCH QA-7 — oct_bug_batch: PMS Check-In Wave B — Balance / GST / Alert UX
**8 items | Priority: CRITICAL (money/GST) | Risk: CRITICAL | Sprint: oct_bug_batch**

| ID | Priority | Title | QA Handover |
|----|----------|-------|-------------|
| BUG-504 | P0 | Check-In max discount cap wrong — doesn't preserve GST on advance | `QA_HANDOVER_BUG502_503_504_2026_10_07.md` |
| BUG-503 | P1 | CheckInForm GST slab never recalculates after discount | `QA_HANDOVER_BUG502_503_504_2026_10_07.md` |
| BUG-505 | P1 | CheckInForm bill grid mixed static/dynamic GST | `QA_HANDOVER_BUG505_2026_10_07.md` |
| BUG-506 | P1 | CheckInForm balance due uses booking_charge base (excludes GST) | `QA_HANDOVER_BUG506-REV_BUG508_2026_10_07.md` |
| BUG-508 | P1 | CheckInForm displayGstTotal compound GST error at max discount | `QA_HANDOVER_BUG506-REV_BUG508_2026_10_07.md` |
| BUG-507 | P2 | Flat discount alert missing — Percent-only discountOverMax | `QA_HANDOVER_BUG506_507_2026_10_07.md` |
| BUG-502 | P2 | Check-In discount alert shrinks input — alert inside flex row | `QA_HANDOVER_BUG502_503_504_2026_10_07.md` |
| BUG-498 | ? | Checkout Bill discount base = stale LR booking_charge | `QA_HANDOVER_FU385D_BUG525_2026_10_09.md` |

---

## BATCH QA-8 — oct_bug_batch: PMS Check-In Wave C — Confirm Guard + Group Check-In
**4 items | Priority: CRITICAL | Risk: CRITICAL | Sprint: oct_bug_batch**
**Note:** BUG-513 handover explicitly says "Combined with BUG-511 + BUG-512 check-in discount sweep" — all three are covered in that single handover.

| ID | Title | QA Handover |
|----|-------|-------------|
| BUG-513 | CheckInForm confirm() submits collectNow to backend at max discount | `QA_HANDOVER_BUG513_2026_10_08.md` |
| BUG-511 | CheckInForm GST strip shows wrong values at normal discounts | `QA_HANDOVER_BUG513_2026_10_08.md` |
| BUG-512 | CheckInForm Collect Now allows collecting gstOnAdv at max discount | `QA_HANDOVER_BUG513_2026_10_08.md` |
| BUG-514 | user-group-check-in: gst_tax=0 hardcoded, room_discount wrong | `QA_HANDOVER_BUG514_2026_10_08.md` |

---

## BATCH QA-9 — oct_bug_batch: PMS In-House / VAT Labels / Room Controls
**6 items | Priority: CRITICAL (money) | Risk: CRITICAL | Sprint: oct_bug_batch**

| ID | Title | QA Handover |
|----|-------|-------------|
| BUG-515 | In-House row balance shows rack-rate GST after discount | `QA_HANDOVER_BUG515_2026_10_08.md` |
| BUG-516 | Room orders VAT item shows GST label in FolioCheckoutPanel | `QA_HANDOVER_BUG516_519_2026_10_09.md` |
| BUG-517 | Checkout room discount Percent mode formula wrong | `QA_HANDOVER_BUG516_519_2026_10_09.md` |
| BUG-518 | Both discount display/payload mismatch + per-side cap missing | `QA_HANDOVER_BUG516_519_2026_10_09.md` |
| BUG-519 | FolioCheckoutPanel room discount controls on LEFT panel (UI fix) | `QA_HANDOVER_BUG516_519_2026_10_09.md` |
| BUG-522 | Remove Room/Both/F&B selector — Room Discount is room-only | `QA_HANDOVER_BUG522_2026_10_09.md` |

---

## BATCH QA-10 — oct_bug_batch: PMS CPP Split + Folio Checkout
**8 items | Priority: CRITICAL (money) | Risk: CRITICAL | Sprint: oct_bug_batch**

| ID | Title | QA Handover |
|----|-------|-------------|
| BUG-523 | Split button missing in Folio checkout CPP | `QA_HANDOVER_BUG523_524_2026_10_09.md` |
| BUG-524 | Room discount alert clamped in RoomDiscountControls | `QA_HANDOVER_BUG523_524_2026_10_09.md` |
| FU-385-D | Front Desk Bill — Split payment re-enabled (D88 CSS rule removed) | `QA_HANDOVER_FU385D_BUG525_2026_10_09.md` |
| BUG-525 | Split room payment legs have no amount cap vs room balance | `QA_HANDOVER_FU385D_BUG525_2026_10_09.md` |
| BUG-526 | Folio Checkout button gray when room split + CPP split both active | `QA_HANDOVER_BUG526_2026_10_10.md` |
| BUG-527 | Dashboard CPP: check-in discount missing from Room section + split button | `QA_HANDOVER_BUG527_2026_10_10.md` |
| BUG-530 | CPP split auto-fill uses food+room total → overfills UPI | `QA_HANDOVER_BUG530_2026_10_10.md` |
| BUG-529 | Folio CPP shows food-only (P0 — double-discount regression from BUG-527 E1) | `QA_HANDOVER_BUG529_531_2026_10_10.md` |

---

## BATCH QA-11 — oct_bug_batch: PMS In-House Balance + Extend Stay
**4 items | Priority: CRITICAL (money) | Risk: CRITICAL | Sprint: oct_bug_batch**

| ID | Title | QA Handover |
|----|-------|-------------|
| BUG-531 | InHouse/Departures balance shows rack rate after check-in discount | `QA_HANDOVER_BUG529_531_2026_10_10.md` |
| BUG-533 | Extend Stay: current bill shows rack rate (discount ignored) + Discount Controls | `QA_HANDOVER_BUG533_2026_10_10.md` |
| BUG-534 | ExtendStayForm result panel missing 'Check-in discount' row after extension | `QA_HANDOVER_BUG534_BUG535_2026_10_11.md` |
| BUG-535 | FolioCheckoutPanel: discountedPrice × nights causes wrong GST slab for extended stay | `QA_HANDOVER_BUG534_BUG535_2026_10_11.md` |

---

## BATCH QA-12 — pos_pms_1: PMS Core — New Check-In + Old Modal
**2 items | Priority: P0/P2 | Risk: CRITICAL | Sprint: pos_pms_1**

| ID | Priority | Title | QA Handover |
|----|----------|-------|-------------|
| BUG-411 | P0 | New CheckInPage — no payment method picker for advance payment | `QA_HANDOVER_BUG411_BUG410_2026_09_15.md` |
| BUG-410 | P2 | Old RoomCheckInModal — no accommodation GST for personal (Walk-in/In-House) | `QA_HANDOVER_BUG411_BUG410_2026_09_15.md` |

---

## BATCH QA-13 — pos_pms_1: Folio + In-House Balance + Room Orders GST
**7 items | Priority: P1/CRITICAL | Risk: CRITICAL | Sprint: pos_pms_1**
**Note:** These are cumulative fixes — test in handover sequence. BATCH2 covers BUG-426 through BUG-430 as a single cascade.

| ID | Title | QA Handover |
|----|-------|-------------|
| BUG-424 | Folio Page Missing Room Orders Section | `QA_HANDOVER_BATCHA_BUG422_423_424_425_2026_09_16.md` |
| BUG-425 | PmsCheckoutDrawer ROOM Balance Excludes GST | `QA_HANDOVER_BATCHA_BUG422_423_424_425_2026_09_16.md` |
| BUG-426 | In-House Balance Missing Transferred F&B and Room Orders | `QA_HANDOVER_BATCH2_BUG426-430_2026_09_16.md` |
| BUG-427 | Folio Total Balance Due Excludes Room Orders Post-GST | `QA_HANDOVER_BATCH2_BUG426-430_2026_09_16.md` |
| BUG-428 | Checkout ROOM Breakdown Missing Lodging GST Line | `QA_HANDOVER_BATCH2_BUG426-430_2026_09_16.md` |
| BUG-429 | Room Orders GST Under-Counted in Folio and In-House | `QA_HANDOVER_BATCH2_BUG426-430_2026_09_16.md` |
| BUG-430 | Room Orders GST Missing Add-Ons Calculation | `QA_HANDOVER_BATCH2_BUG426-430_2026_09_16.md` |

---

## BATCH QA-14 — pos_pms_2: Check-In Form + In-House Page + Old Modal
**5 items | Priority: P1/P2 | Risk: HIGH/CRITICAL | Sprint: pos_pms_2**
**Note:** BATCHA covers BUG-422/423 + BUG-424/425 (cross-sprint). BATCH3 covers BUG-419/420/421.

| ID | Priority | Title | QA Handover |
|----|----------|-------|-------------|
| BUG-420 | P1 | Old Check-In Modal: Returning Guest ID Document Not Visible | `QA_HANDOVER_BATCH3_BUG419-421_2026_09_16.md` |
| BUG-421 | P1 | In-House Page: Balance Column Shows Booking Total Not Outstanding | `QA_HANDOVER_BATCH3_BUG419-421_2026_09_16.md` |
| BUG-422 | P1 | Old RoomCheckInModal: balance_payment Sent Without GST | `QA_HANDOVER_BATCHA_BUG422_423_424_425_2026_09_16.md` |
| BUG-423 | P1 | Guest Folio Page: Room Balance Excludes GST | `QA_HANDOVER_BATCHA_BUG422_423_424_425_2026_09_16.md` |
| BUG-419 | P2 | New Check-In Form: Corporate/B2B Billing Field in Wrong Position | `QA_HANDOVER_BATCH3_BUG419-421_2026_09_16.md` |

---

## BATCH QA-15 — sep_bug_closure: Inventory / Stock / Registry Export
**3 items | Priority: HIGH (BUG-459 CRITICAL — stock quantities) | Risk: HIGH | Sprint: sep_bug_closure**

| ID | Priority | Title | QA Handover |
|----|----------|-------|-------------|
| BUG-459 | ? | Stock Audit: base-unit number sent with display-unit label → inflates stock | `QA_HANDOVER_2026-09-25_BUG459_CR387_COMBINED.md` |
| CR-387 | P1 | Smart Purchase: unit-consistent quantity breakdown | `QA_HANDOVER_2026-09-25_BUG459_CR387_COMBINED.md` |
| CR-389 | ? | Registry Excel Export — All CRs + BUGs with Status, Description & Tracking | `QA_HANDOVER_CR389_2026_09_27.md` |

---

## BATCH QA-16 — Misc: CR-388 (TBD sprint)
**1 item | Priority: ? | Risk: MEDIUM**

| ID | Title | QA Handover |
|----|-------|-------------|
| CR-388 | Editable min alert unit field (Inventory) | `QA_HANDOVER_CR388_2026_09_25.md` |

---

## REGRESSION-MENTION ITEMS (3 items — secondary coverage only)
These IDs appear only as **regression checks** inside another item's handover — not as primary test subjects.
QA agent can verify them as part of the listed batch, but dedicated test cases do not exist.

| ID | Title | Covered-as-regression in |
|----|-------|--------------------------|
| BUG-462 | Order Entry: Empty-state not shown on first boot for non-Normal restaurants | `QA_HANDOVER_CR376_FU_B_2026_09_26.md` (R6) |
| BUG-464 | Order Entry chip reads "FOOD MENU Menu" — word Menu duplicated | `QA_HANDOVER_CR376_FU_B_2026_09_26.md` (R7) |
| BUG-499 | Both room+F&B discount sends wrong amounts to backend | `QA_HANDOVER_BUG516_519_2026_10_09.md` (REG-4) |

---

## EXECUTION ORDER FOR QA AGENT

```
Priority order (P0 items first, then feature-area grouping for efficiency):

1.  QA-1   (nov_cr_batch)           — 2 items  — newest code, all P0
2.  QA-2   (oct_cr_batch reports)   — 7 items  — P0 PWA + reports
3.  QA-4   (coupon/loyalty/cart)    — 5 items  — CRITICAL money-adjacent
4.  QA-5   (room discount CR-407)   — 1 item   — CRITICAL money
5.  QA-3   (tablet)                 — 2 items  — HIGH layout
6.  QA-6   (PMS Check-In Wave A)    — 13 items — CRITICAL money/GST
7.  QA-7   (PMS Check-In Wave B)    — 8 items  — CRITICAL money/GST
8.  QA-8   (PMS Check-In Wave C)    — 4 items  — CRITICAL money/GST
9.  QA-9   (PMS In-House/VAT)       — 6 items  — CRITICAL money
10. QA-10  (PMS CPP split)          — 8 items  — CRITICAL money
11. QA-11  (PMS balance/extend)     — 4 items  — CRITICAL money
12. QA-12  (pos_pms_1 check-in)     — 2 items  — CRITICAL money
13. QA-13  (pos_pms_1 folio/GST)    — 7 items  — CRITICAL money
14. QA-14  (pos_pms_2)              — 5 items  — HIGH money
15. QA-15  (sep_bug_closure)        — 3 items  — HIGH stock accuracy
16. QA-16  (CR-388 misc)            — 1 item   — MEDIUM

QA report path: /app/memory/test_reports/QA_REPORT_<BATCH_ID>_<DATE>.md
Format: per AGENT_PROMPT_ALPHA Role 4 — PASS/FAIL per test case, severity, registry spot-check.
```
