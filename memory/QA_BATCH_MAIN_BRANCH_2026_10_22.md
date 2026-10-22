# QA Batch — All GATE_5A_IMPLEMENTED Items Without QA (main branch)
**Prepared by:** INVESTIGATION Agent (E1)
**Date:** 2026-10-22
**Branch:** main @ 2dc8994 (2026-10-22)
**Source:** registry.json (925 items) + all session handovers + QA handover files

---

## INVESTIGATION SUMMARY

**Finding:** 111 items carry `GATE_5A_IMPLEMENTED` in registry.json with no `GATE_5B` QA sign-off.
The most recent QA reports on disk are from **2026-10-16** (oct_cr_batch tablet sprint). All items
implemented after that, plus any items from earlier batches never sent to QA, are awaiting Gate 5b.

**Confidence:** HIGH — registry status + session handover cross-check performed.
**Steps used:** 9/10

---

## REGISTRY DRIFT — FIX BEFORE STARTING QA

Two items are flagged GATE_5A in registry but are actually QA-passed per session handovers.
**QA agent must update registry.json for these before executing any batch:**

| ID | Session says | Registry says | Fix |
|----|-------------|---------------|-----|
| **BUG-557** | "QA PASSED 100%" (SESSION_HANDOVER_2026_10_19_MAIN.md) | GATE_5A_IMPLEMENTED | → GATE_5B_QA_PASSED |
| **BUG-595** | "GATE_5B_QA_CONDITIONAL_PASS (drift corrected)" (SESSION_HANDOVER_2026-10-19_INTAKE_BUG597_598.md) | GATE_5A_IMPLEMENTED | → GATE_5B_QA_CONDITIONAL_PASS |

**Note:** BUG-509 is marked `GATE_5A_IMPLEMENTED_WITH_DEFECT` — a known self-reported defect exists.
QA agent must verify the defect is still present and file a BLOCKER finding.

---

## CREDENTIALS (all batches)

| Account | Password | RID | Use for |
|---------|----------|-----|---------|
| owner@cafe103.com | Qplazm@10 | 644 | General POS, coupon, cart, reports |
| owner@palmhouse.com | Qplazm@10 | — | PMS, room orders, folio, tablets, Status View |
| owner@kunafamahal.com | Qplazm@10 | — | Role-based discount (roleBaseDiscount=Yes) |
| cashier@kunafamahal.com | Qplazm@10 | — | Cashier path (no discount perm) |
| owner@thegoankitchen.com | Qplazm@10 | — | SC items, BUG-536 |
| owner@18march.com | Qplazm@10 | — | Room billing |

Login: `POST https://preprod.mygenie.online/api/v1/auth/vendoremployee/login`
App URL: https://core-pos-frontend-13.preview.emergentagent.com

---

## BATCH QA-1 — nov_cr_batch (P0 — ship-critical, newest code)
**4 items | Risk: HIGH | Sprint: nov_cr_batch**

| ID | Title | Priority | QA Handover |
|----|-------|----------|-------------|
| CR-429 | Active table toggle — show only occupied/active tables | P0 | `QA_HANDOVER_CR429_2026_10_17.md` ✅ |
| CR-430 | Served items: decrease card height (remove 44px touch target on static items) | P0 | `QA_HANDOVER_CR430_TOGGLE_UI_2026_10_17.md` ✅ |
| BUG-573 | Order card variation: selected option not shown (shows group name only) | P0 | ❌ NONE — write test cases |
| BUG-599 | Aggregator orders vanish on page refresh — useRefreshAllData replaces stale refs | P0 | ❌ NONE — write test cases |

**CR-429 + CR-430:** Execute test cases from handover files exactly.

**BUG-573 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Variation selected option shown | Open an order with a variation item (e.g. Biryani — Full/Half). Select a specific variation. View order card on dashboard. | Order card shows the SELECTED variation option name (e.g. "Full"), not the group name (e.g. "Size") |
| T2 | Multiple variations | Item with 2 variation groups, different options selected | Each variation's chosen option visible on card |
| T3 | No variation item | Item without variation | No variation row shown — no regression |

**BUG-599 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Aggregator orders persist on refresh | Open POS with aggregator orders on dashboard → hard-refresh browser (F5) | Aggregator orders still visible after refresh |
| T2 | useRefreshAllData does not clear aggregator orders | Trigger a data refresh (e.g. place a new order) | Existing aggregator orders remain; only stale data replaced |

---

## BATCH QA-2 — oct_cr_batch: CRM/Payment/Coupon/Loyalty/Tablet
**8 items | Risk: CRITICAL/HIGH | Sprint: oct_cr_batch**
**All have QA handover files — execute as written.**

| ID | Title | Priority | QA Handover File |
|----|-------|----------|-----------------|
| CR-407 | Room Discount (check-in + checkout apply_to + partial_payments_room) | ? | `QA_HANDOVER_CR405_PHASE_AB_2026_10_01.md` (closest — check if covers CR-407) |
| CR-408 | Coupon Auto-Apply Remove — Cashier Must Choose | ? | `QA_HANDOVER_CR408_2026_10_11.md` ✅ |
| CR-409 | Scan Order Coupon Not Pre-Loaded in Collect Payment | ? | `QA_HANDOVER_CR409_2026_10_11.md` ✅ |
| CR-411 | Cart Persists on Type/Table Switch, Clears Only on Dashboard Return | ? | `QA_HANDOVER_CR411_2026_10_12.md` ✅ |
| CR-412 | Loyalty Pre-tick from Scan & Order (loyalty_info.redeemed=true) | ? | `QA_HANDOVER_CR412_2026_10_12.md` ✅ |
| CR-413 | Remove CR-409 initialCoupon Pre-load from Collect Payment Panel | ? | `QA_HANDOVER_CR413_2026_10_12.md` ✅ |
| CR-414 | Dashboard Tablet Responsive Redesign | ? | `QA_HANDOVER_CR414_BUG537_2026_10_13.md` ✅ |
| BUG-537 | Dashboard Layout Broken on Android Tablets | ? | `QA_HANDOVER_CR414_BUG537_2026_10_13.md` ✅ |

**Critical context for CR-409/CR-413 (paired):** CR-409 added initialCoupon pre-load; CR-413 REMOVES it again. Verify final state: CPP opens with NO pre-loaded coupon, cashier must manually enter code.

---

## BATCH QA-3 — oct_cr_batch: Reports / Dashboard / PWA / Sidebar
**10 items | Risk: HIGH/MEDIUM | Sprint: oct_cr_batch**

| ID | Title | Priority | QA Handover |
|----|-------|----------|-------------|
| BUG-353 | OrderReportBetaPage: Date range capped at 1 month | 1 | `QA_HANDOVER_BUG353_2026_10_19.md` ✅ |
| BUG-546 | Order Report: split payment settled orders show "Split Bill" payment method | 2 | `QA_HANDOVER_BUG547_BUG546_2026_10_17.md` ✅ |
| BUG-547 | Channel and Payments doughnut chart displays amount as 0 | 2 | `QA_HANDOVER_BUG547_BUG546_2026_10_17.md` ✅ |
| BUG-556 | Auto Bill: payment mode printed as "Unpaid" on bill even when payment set | ? | `QA_HANDOVER_BUG556_2026_10_16.md` ✅ |
| CR-422 | Last Month filter — replace "Last 30 Days" with "Last Month" across all 13 report pages | 2 | ❌ NONE |
| CR-435 | Sidebar: Remove Redundant Inventory Sub-Links (tab bar is the nav) | 2 | `QA_HANDOVER_CR435_2026_10_18.md` ✅ |
| BUG-560 | Firebase getMessaging() async unsupported-browser rejection escapes try/catch | P0 | ❌ NONE |
| BUG-580 | Discount report — restaurant-specific discount types (e.g. Palmhouse types) | P0 | ❌ NONE |
| BUG-597 | PWA White Screen / App Freeze on Every Production Deploy | P0 | `QA_HANDOVER_BUG597_2026_10_19.md` ✅ |
| BUG-598-A | Reports Date Range Caps Too Restrictive — 8 Pages Still at 60d/365d | P1 | ❌ NONE |
| CR-164 | Send Payment Link from Daily Reports (Per-Row Action) | 1 | ❌ NONE |

**CR-422 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | "Last Month" filter exists | Open any of the 13 report pages with date filter | Filter dropdown/chip shows "Last Month" option instead of "Last 30 Days" |
| T2 | Last Month data correct | Select "Last Month" in January | Shows data for December 1–31 (full calendar month, not rolling 30 days) |
| T3 | Applied across all 13 pages | Check all report pages that had the filter | Consistent label and behavior on all pages |

**BUG-560 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | No unhandled rejection on unsupported browser | Open app in a browser without notification support (or simulate: `delete window.Notification`). Load any page. | No unhandled promise rejection error in console. App does not crash. |
| T2 | Firebase init does not block app | Open app on supported browser | Firebase loads normally, no regression |

**BUG-580 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Discount report shows correct type labels | Login as palmhouse → Insights → Discount Report | Restaurant-specific discount type names shown (e.g. "Palmhouse Staff", not generic "type_1"). No "undefined" values. |

**CR-164 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Payment link button in Daily Reports | Open Daily Reports → find an order row | Row has a "Send Payment Link" action button |
| T2 | Clicking sends WhatsApp/payment link | Click payment link button on an order with customer phone | Payment link sent (toast confirmation). No console error. |

**BUG-598-A — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | 8 report pages allow >60d range | Go to each of the 8 affected report pages → set date range > 60 days | Date picker allows the selection; report loads |
| T2 | No data cap regression | Set range within old cap (e.g. 30 days) on all 8 pages | Still works correctly |

---

## BATCH QA-4 — oct_bug_batch: PMS Check-In Financial (CRITICAL — money)
**28 items | Risk: CRITICAL | Sprint: oct_bug_batch**
**Context: All bugs relate to Front Desk v2 Check-In (CheckInForm/CheckInPage), GST calculation, discount caps, and payment amounts.**
**Test account: palmhouse (has PMS/hotel features)**

Most items have QA handover files. Execute handovers as written, in chronological order (earlier handovers are dependencies for later ones).

| ID | Title | Priority | QA Handover |
|----|-------|----------|-------------|
| BUG-485 | Insights Dashboard: PDF/Excel download buttons have no onClick | ? | `QA_HANDOVER_BUG485_2026_10_05.md` ✅ |
| BUG-487 | orderTransform roomInfo reads wrong key: discount_amount (not room_discount_amount) | ? | `QA_HANDOVER_BUG487_2026_10_05.md` ✅ |
| BUG-488 | 'To Room' transfer always sends source_order_id:'' | ? | ❌ NONE |
| BUG-489 | CR-407 Sub-scope A discount not visible on front-desk-v2 | ? | `QA_HANDOVER_BUG489_2026_10_05.md` ✅ |
| BUG-490 | Room discount cap missing — can exceed balance due | ? | `QA_HANDOVER_BUG490_BUG491_2026_10_05.md` ✅ |
| BUG-491 | In-House balance + discount display batch | ? | `QA_HANDOVER_BUG490_BUG491_2026_10_05.md` ✅ |
| BUG-492 | Checkout Bill panel: Discount not reflected in Checkout total + no % cap | ? | `QA_HANDOVER_BUG492_2026_10_06.md` ✅ |
| BUG-493 | pmsService: row.charge never set → chargeGst always 0 → BALANCE column wrong | ? | `QA_HANDOVER_BUG493_2026_10_06.md` ✅ |
| BUG-494 | FolioCheckoutPanel Bill: stale LR charge.* fields when check-in discount applied | ? | `QA_HANDOVER_BUG494_2026_10_06.md` ✅ |
| BUG-495 | maxDiscount/maxPct formula ignores GST on advance | ? | `QA_HANDOVER_BUG495_2026_10_06.md` ✅ |
| BUG-496 | maxPct formula wrong at check-in | ? | `QA_HANDOVER_BUG496_497_500_501_2026_10_07.md` ✅ |
| BUG-497 | Collect Now max cap missing | ? | `QA_HANDOVER_BUG496_497_500_501_2026_10_07.md` ✅ |
| BUG-498 | Checkout Bill discount base = stale LR booking_charge | ? | ❌ NONE |
| BUG-499 | Both room+F&B discount sends wrong amounts to backend | ? | ❌ NONE |
| BUG-500 | CheckInPage GST strip doesn't recalculate on discount | ? | `QA_HANDOVER_BUG496_497_500_501_2026_10_07.md` ✅ |
| BUG-501 | Login page copyright year hardcoded '2025' | ? | `QA_HANDOVER_BUG496_497_500_501_2026_10_07.md` ✅ |
| BUG-502 | Check-In discount alert shrinks input — alert inside flex row | P2 | `QA_HANDOVER_BUG502_503_504_2026_10_07.md` ✅ |
| BUG-503 | CheckInForm GST slab never recalculates after discount | P1 | `QA_HANDOVER_BUG502_503_504_2026_10_07.md` ✅ |
| BUG-504 | Check-In max discount cap wrong — doesn't preserve GST on advance | P0 | `QA_HANDOVER_BUG502_503_504_2026_10_07.md` ✅ |
| BUG-505 | CheckInForm bill grid mixed static/dynamic GST | P1 | `QA_HANDOVER_BUG505_2026_10_07.md` ✅ |
| BUG-506 | CheckInForm balance due uses booking_charge base (excludes GST) | P1 | `QA_HANDOVER_BUG506-REV_BUG508_2026_10_07.md` ✅ |
| BUG-507 | Flat discount alert missing — Percent-only discountOverMax | P2 | `QA_HANDOVER_BUG506_507_2026_10_07.md` ✅ |
| BUG-508 | CheckInForm displayGstTotal compound GST error at max discount | P1 | `QA_HANDOVER_BUG506-REV_BUG508_2026_10_07.md` ✅ |
| BUG-509 | CheckInForm computeBase universal subtraction — **KNOWN DEFECT** | ? | ❌ NONE — verify defect + file BLOCKER |
| BUG-510 | CheckInForm/CheckInPage maxPct toFixed(2) rounds DOWN | ? | ❌ NONE |
| BUG-511 | CheckInForm GST strip shows wrong values at normal discounts | ? | ❌ NONE |
| BUG-512 | CheckInForm Collect Now allows collecting gstOnAdv at max discount | ? | ❌ NONE |
| BUG-513 | CheckInForm confirm() submits collectNow to backend at max discount | ? | `QA_HANDOVER_BUG513_2026_10_08.md` ✅ |
| BUG-514 | user-group-check-in: gst_tax=0 hardcoded, room_discount wrong value | ? | `QA_HANDOVER_BUG514_2026_10_08.md` ✅ |

**BUG-488 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | 'To Room' transfer sends correct source_order_id | From an active food order on dashboard, click "To Room" transfer. Inspect Network payload. | `source_order_id` = actual order ID (not empty string `''`) |

**BUG-498 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Checkout discount base uses post-discount balance | Front Desk → Checkout a guest who has an existing check-in discount. Open checkout bill. | Discount calculation base = current discounted balance, not original `booking_charge` |

**BUG-499 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Both room+F&B discount payload correct | Apply "Both" discount on a room order with food items. Inspect Network payload on submit. | `room_discount` = room-portion amount; food discount = food-portion amount. Neither is the total combined discount. |

**BUG-509 KNOWN DEFECT:**
Registry says "GATE_5A_IMPLEMENTED_WITH_DEFECT". QA agent must:
1. Check the implementation agent's self-test note for what the defect is
2. Reproduce it
3. File as BLOCKER finding

**BUG-510/511/512 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 (510) | maxPct rounds up correctly | Enter max discount % = 14.5% → type 14.5 in percent input | Input accepts 14.5, not rounded down to 14 |
| T2 (511) | GST strip shows correct values at normal discount | CheckInForm → apply non-maximum discount → check GST strip | GST strip reflects discounted room price, not rack rate |
| T3 (512) | Collect Now blocked at max discount for gstOnAdv | CheckInForm → apply maximum allowed discount → check "Collect Now" field | Collect Now amount = ₹0 or disabled (cannot collect GST on advance when fully discounted) |

---

## BATCH QA-5 — oct_bug_batch: PMS Checkout / Folio / CPP Split (CRITICAL — money)
**14 items | Risk: CRITICAL | Sprint: oct_bug_batch**
**Most have QA handover files — execute as written.**

| ID | Title | Priority | QA Handover |
|----|-------|----------|-------------|
| BUG-515 | In-House row balance + expanded row show rack-rate GST after discount | ? | `QA_HANDOVER_BUG515_2026_10_08.md` ✅ + `QA_HANDOVER_BUG515_SUBB_2026_10_08.md` ✅ |
| BUG-516 | Room orders VAT item shows GST label in FolioCheckoutPanel | ? | `QA_HANDOVER_BUG516_519_2026_10_09.md` ✅ |
| BUG-517 | Checkout room discount Percent mode formula applies % to baseBalance not room price | ? | ❌ NONE |
| BUG-518 | Both discount display/payload mismatch + per-side cap missing | ? | ❌ NONE |
| BUG-519 | FolioCheckoutPanel room discount controls on LEFT panel | ? | `QA_HANDOVER_BUG516_519_2026_10_09.md` ✅ |
| BUG-522 | Remove Room/Both/F&B selector — Room Discount is room-only | ? | `QA_HANDOVER_BUG522_2026_10_09.md` ✅ |
| BUG-523 | Split button missing in Folio checkout CPP | ? | `QA_HANDOVER_BUG523_524_2026_10_09.md` ✅ |
| BUG-524 | Room discount alert clamped in RoomDiscountControls | ? | `QA_HANDOVER_BUG523_524_2026_10_09.md` ✅ |
| BUG-525 | Split room payment legs have no amount cap vs room balance | ? | `QA_HANDOVER_FU385D_BUG525_2026_10_09.md` ✅ |
| FU-385-D | Front Desk Bill — Split payment re-enabled (D88 CSS rule removed) | ? | `QA_HANDOVER_FU385D_BUG525_2026_10_09.md` ✅ |
| BUG-526 | Folio Checkout button gray when room split + CPP split both active | ? | `QA_HANDOVER_BUG526_2026_10_10.md` ✅ |
| BUG-527 | Dashboard CPP: check-in discount missing from Room section + split button | ? | `QA_HANDOVER_BUG527_2026_10_10.md` ✅ + `QA_HANDOVER_BUG527_F1F4_2026_10_10.md` ✅ |
| BUG-529 | Folio CPP shows food-only (double-discount regression from BUG-527 E1) | P0 | `QA_HANDOVER_BUG529_531_2026_10_10.md` ✅ |
| BUG-530 | CPP split auto-fill uses food+room total → overfills UPI | ? | `QA_HANDOVER_BUG530_2026_10_10.md` ✅ |
| BUG-531 | InHouse/Departures balance shows rack rate after discount | ? | `QA_HANDOVER_BUG529_531_2026_10_10.md` ✅ |
| BUG-533 | Extend Stay: current bill shows rack rate (discount ignored) | ? | `QA_HANDOVER_BUG533_2026_10_10.md` ✅ |
| BUG-534 | ExtendStayForm result panel missing 'Check-in discount' row | ? | `QA_HANDOVER_BUG534_BUG535_2026_10_11.md` ✅ |
| BUG-535 | FolioCheckoutPanel: discountedPrice * nights causes wrong GST slab | ? | `QA_HANDOVER_BUG534_BUG535_2026_10_11.md` ✅ |

**BUG-517 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Percent mode uses room price as base | FolioCheckout → apply Percent discount 10% on a room of ₹2,000 | Discount = ₹200 (10% of ₹2,000 room price), not 10% of net balance |

**BUG-518 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Display matches payload | Apply Both discount → compare shown amount to Network payload | `room_discount` in payload matches what is shown on screen. Per-side cap enforced. |

---

## BATCH QA-6 — pos_pms_1/2: PMS Core (HIGH — financial)
**17 items | Risk: HIGH/CRITICAL | Sprint: pos_pms_1, pos_pms_2**
**Most have QA handover files.**

| ID | Title | Priority | QA Handover |
|----|-------|----------|-------------|
| BUG-411 | New CheckInPage — no payment method picker for advance payment | P0 | `QA_HANDOVER_BUG411_BUG410_2026_09_15.md` ✅ |
| BUG-410 | Old RoomCheckInModal — no accommodation GST for personal | P2 | `QA_HANDOVER_BUG411_BUG410_2026_09_15.md` ✅ |
| BUG-419 | New Check-In Form: Corporate/B2B Billing Field in Wrong Position | P2 | `QA_HANDOVER_BATCH3_BUG419-421_2026_09_16.md` ✅ |
| BUG-420 | Old Check-In Modal: Returning Guest ID Document Not Visible | P1 | `QA_HANDOVER_BATCH3_BUG419-421_2026_09_16.md` ✅ |
| BUG-421 | In-House Page: Balance Column Shows Booking Total Not Outstanding | P1 | `QA_HANDOVER_BATCH3_BUG419-421_2026_09_16.md` ✅ |
| BUG-422 | Old RoomCheckInModal: balance_payment Sent Without GST | P1 | `QA_HANDOVER_BATCHA_BUG422_423_424_425_2026_09_16.md` ✅ |
| BUG-423 | Guest Folio Page: Room Balance Excludes GST | P1 | `QA_HANDOVER_BATCHA_BUG422_423_424_425_2026_09_16.md` ✅ |
| BUG-424 | Folio Page Missing Room Orders Section | ? | `QA_HANDOVER_BATCHA_BUG422_423_424_425_2026_09_16.md` ✅ |
| BUG-425 | PmsCheckoutDrawer ROOM Balance Excludes GST | ? | `QA_HANDOVER_BATCHA_BUG422_423_424_425_2026_09_16.md` ✅ |
| BUG-426 | In-House Balance Missing Transferred F&B and Room Orders | ? | `QA_HANDOVER_BUG426_2026_09_16.md` ✅ |
| BUG-427 | Folio Total Balance Due Excludes Room Orders Post-GST | ? | `QA_HANDOVER_BUG427_2026_09_16.md` ✅ |
| BUG-428 | Checkout ROOM Breakdown Missing Lodging GST Line | ? | `QA_HANDOVER_BUG428_2026_09_16.md` ✅ |
| BUG-429 | Room Orders GST Under-Counted in Folio and In-House | ? | `QA_HANDOVER_BUG429_2026_09_16.md` ✅ |
| BUG-430 | Room Orders GST Missing Add-Ons Calculation | P1 | `QA_HANDOVER_BUG430_2026_09_16.md` ✅ |
| BUG-402 | Extend Stay — extra nights rate shows ₹0 | P1 | `QA_HANDOVER_BATCH_A_2026_09_10.md` ✅ (check if covers BUG-402) |
| BUG-415 | Room Orders Report — Y-axis ticks show ₹0k or collide | P3 | `QA_HANDOVER_BATCH_A_2026_09_10.md` ✅ (check if covers BUG-415) |
| BUG-416 | Night Audit + Revenue Dashboard — no Sidebar navigation and no Back button | P2 | `QA_HANDOVER_BATCH_A_2026_09_10.md` ✅ (check if covers BUG-416) |

---

## BATCH QA-7 — sep_bug_closure: Inventory / Stock / Registry
**6 items | Risk: HIGH (BUG-459 P0/CRITICAL — stock quantities) | Sprint: sep_bug_closure**

| ID | Title | Priority | QA Handover |
|----|-------|----------|-------------|
| BUG-459 | Stock Audit physical count: base-unit number sent with display-unit label | ? | `QA_HANDOVER_2026-09-25_BUG459_CR387_COMBINED.md` ✅ |
| CR-387 | Smart Purchase: unit-consistent quantity breakdown | P1 | `QA_HANDOVER_2026-09-25_BUG459_CR387_COMBINED.md` ✅ |
| CR-389 | Registry Excel Export — All CRs + BUGs with Status, Description & Tracking | ? | `QA_HANDOVER_CR389_2026_09_27.md` ✅ |
| BUG-461 | Sub-recipes appear in Bulk Edit Ingredients | ? | ❌ NONE |
| BUG-462 | Order Entry: Empty-state not shown on first boot for non-Normal restaurants | ? | ❌ NONE |
| BUG-464 | Order Entry chip reads "FOOD MENU Menu" — word Menu duplicated | ? | ❌ NONE |

**BUG-461 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Sub-recipes not in Bulk Edit Ingredients | Menu Management → Ingredients → Bulk Edit | Sub-recipe items do NOT appear in the ingredient list |

**BUG-462 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Empty-state on first boot for non-Normal restaurants | Login as a restaurant with menu type "Aggregator" or "Premium" → boot → open Order Entry | Empty-state shown correctly (no items flash or show wrong state) |

**BUG-464 — inline test cases:**
| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Chip label not duplicated | Open Order Entry → check menu type chip | Chip shows "FOOD MENU" or "Menu" — not "FOOD MENU Menu" |

---

## BATCH QA-8 — tooling_oct1: Internal Dev Tools (LOW — internal, non-user-facing)
**6 items | Risk: LOW | Sprint: tooling_oct1**
**No QA handover files. Inline test cases below.**
**Note: These are dev tooling changes (Google Sheets sync, AGENT_PROMPT doc, area module migration). QA can be deferred if owner prioritises user-facing batches.**

| ID | Title | Priority |
|----|-------|----------|
| CR-406 | Google Sheets Two-Way Sync — Registry Export/Import via OAuth2 | P2 |
| CR-416 | AGENT_PROMPT_ALPHA: Add SHEET PUSH + SHEET DRIFT Roles | ? |
| CR-419 | sheets_sync.py: Contract v1.5 Compliance Rewrite | ? |
| CR-420 | sheets_sync.py: Post-Push QA Gap Fixes (8 gaps from Abhishek review) | ? |
| BUG-541 | SHEET DRIFT only diffs 3 columns — misses Priority and other owner fields | ? |
| CR-428 | Area module migration — replace 18-value list with 13 modules across app | ? |

**QA approach for tooling batch:** Code review + grep verification rather than E2E browser test. QA agent should verify:
- CR-416/419/420/BUG-541: Run `python3 scripts/sheets_sync.py --drift` (if available). Verify SHEET PUSH and SHEET DRIFT roles are documented in AGENT_PROMPT_ALPHA.md.
- CR-406: Verify Google Sheets OAuth credentials configured in env. Run push and verify registry data appears in sheet.
- CR-428: Grep for the 18-value area list → confirm replaced with 13-module structure.

---

## BATCH QA-9 — Backlog / Older Sprints
**10 items | Risk: MEDIUM/LOW | Various sprints**
**These are older items that were never sent to QA. Owner may decide to defer some.**

| ID | Title | Sprint | Priority | QA Handover |
|----|-------|--------|----------|-------------|
| BUG-463 | Local Settings Active Menu: Two visually identical Promotional Menu pics | oct_cr_batch | 2 | ❌ NONE — partial impl, backend-blocked |
| BUG-183 | Daily Report — Phone number and customer name missing in Credit tab | pos_5_0 | P1 | ❌ NONE |
| BUG-191 | Customer Intelligence — Phone number missing in Insights section | pos_5_0 | P2 | ❌ NONE |
| CR-119 | Aggregator Food Mapping — POS Menu to Platform Item Mapping | pos_5_1 | P1 | ❌ NONE |
| CR-377 | Sales Report Complete Redesign — surface all 82 API fields | pos_7_0 | P2 | ❌ NONE |
| CR-370 | Stale Doc Corrections — 5 Records from Audit §3 | pos_audit_1 | P2 | ❌ NONE (doc-only) |
| CR-388 | Editable min alert unit field | TBD | ? | ❌ NONE |
| BUG-312 | fromAPI.ingredients() missing isSubRecipe/subrecipeId fields — SUBSUMED by CR-139 | pos_5_0 | P1 | SUBSUMED |
| BUG-313 | Sub-recipe appears in Stock Update auto-plan — SUBSUMED by CR-139 | pos_5_0 | P1 | SUBSUMED |
| PROD-003 | PayLater table clear | ? | ? | ❌ NONE |

**Note on BUG-463:** Registry shows GATE_5A but Session Handover 2026-10-18 (BUG-463 Architecture Hold) says current implementation is PARTIAL and architecture revision is pending backend confirmation. Do NOT QA until backend confirms `food_for` filter support. Mark as BLOCKED.

**Note on BUG-312/313:** Both are SUBSUMED by CR-139. Verify CR-139 is CLOSED/OWNER VERIFIED, then close these as SUBSUMED.

---

## BATCH SUMMARY TABLE

| Batch | Items | Sprint | Risk | QA Handovers | Action |
|-------|-------|--------|------|--------------|--------|
| QA-1 | CR-429, CR-430, BUG-573, BUG-599 | nov_cr_batch | HIGH | 2/4 exist | Execute handovers + write 2 inline |
| QA-2 | CR-407–413, CR-414, BUG-537 | oct_cr_batch | CRITICAL | 7/8 exist | Execute handovers |
| QA-3 | BUG-353, BUG-546-547, BUG-556, CR-422, CR-435, BUG-560, BUG-580, BUG-597, BUG-598-A, CR-164 | oct_cr_batch | HIGH | 6/10 exist | Execute handovers + 4 inline |
| QA-4 | BUG-485, BUG-487-514 (28 bugs) | oct_bug_batch | CRITICAL | 20/28 exist | Execute handovers + 8 inline |
| QA-5 | BUG-515-535, FU-385-D (17 bugs) | oct_bug_batch | CRITICAL | 15/17 exist | Execute handovers + 2 inline |
| QA-6 | BUG-402, BUG-410-430 (17 bugs) | pos_pms_1/2 | HIGH | 14/17 exist | Execute handovers |
| QA-7 | BUG-459, CR-387, CR-389, BUG-461-464 | sep_bug_closure | HIGH/MEDIUM | 3/6 exist | Execute handovers + 3 inline |
| QA-8 | CR-406, CR-416-420, BUG-541, CR-428 | tooling_oct1 | LOW | 0/6 | Grep/code review only |
| QA-9 | BUG-463 (blocked), BUG-183, BUG-191, CR-119, CR-377, CR-388, PROD-003 | various | MEDIUM/LOW | 0/7 | Defer or write inline |
| **TOTAL** | **111 items** | | | **~65/111 have handovers** | |

---

## QA AGENT EXECUTION ORDER

**Before starting any batch:**
1. Fix registry drift: BUG-557 → GATE_5B_QA_PASSED; BUG-595 → GATE_5B_QA_CONDITIONAL_PASS
2. Flag BUG-463 as BLOCKED (backend confirmation pending)
3. Verify BUG-312/313 are SUBSUMED (check CR-139 is CLOSED)

**Execution priority:**
1. **QA-1** (nov_cr_batch — newest, P0 all)
2. **QA-2** (CRM/coupon/loyalty — user-facing financial)
3. **QA-3** (reports/PWA/dashboard)
4. **QA-4** (PMS CheckIn financial — CRITICAL money, 28 bugs)
5. **QA-5** (PMS Checkout/Folio — CRITICAL money)
6. **QA-6** (PMS Core — HIGH)
7. **QA-7** (Inventory — HIGH)
8. **QA-8** (Tooling — defer unless owner requests)
9. **QA-9** (Backlog — defer unless owner requests)

**QA report path:** `/app/memory/test_reports/QA_REPORT_<BATCH>_<DATE>.md`
Per AGENT_PROMPT_ALPHA Role 4 format: per-item PASS/FAIL/BLOCKER/MAJOR/MINOR/NOTE + evidence + registry spot-check.
