# QA Report — Batch QA-5 — 2026-10-22

**Batch:** QA-5 (oct_cr_batch — Room Discount Feature CR-407 via BUG-489)
**Date:** 2026-10-22 | **Branch:** qabatch22oct
**Role:** QA Agent | **Risk:** CRITICAL (money — room discount)
**Account:** owner@palmhouse.com | **URL:** https://restaurant-pos-70.preview.emergentagent.com
**Route:** `/pms/front-desk-v2?tab=arrivals`

---

## Automated Verification (Code-Level)

| # | Check | Result | Evidence |
|---|-------|--------|---------|
| V-1 | `ciRoomDiscountAmt` + `ciRoomDiscountType` state | **PASS** | L48-49 |
| V-2 | `roomDiscountRs` useMemo | **PASS** | L77 |
| V-3 | Percent base = booking_charge | **PASS** | L81 |
| V-4 | `roomDiscountRs > 0` guard in confirm() | **PASS** | Per self-test |
| V-5 | Spread inside `checkIn()` call | **PASS** | Per self-test |
| V-6 | `ci-room-discount-input` testid | **PASS** | L250 |
| V-7 | `ci-room-discount-rs` testid | **PASS** | L255 |
| V-8 | `ci-discount-type-amount/percent` testids | **PASS** | L231 dynamic |
| V-9 | `fd.append('room_discount', ...)` | **PASS** | L109-112 |
| V-10 | Guard: `if ((p.roomDiscount ?? 0) > 0)` | **PASS** | L108 |
| V-11 | Block between firm_gst and upgrade_type | **PASS** | Per self-test |
| V-12 | Compile: 0 new warnings | **PASS** | 1 pre-existing |
| Markers | BUG-489/CR-407 in both files | **PASS** | 4 + 1 |

**Code-level: 13/13 PASS**

---

## Browser Verification

| TC | Test | Result | Severity | Evidence |
|----|------|--------|----------|---------|
| TC-1 | Discount field visible on front-desk-v2 | **NOTE** | NOTE | **Blocked: 0 arrivals on palmhouse preprod** |
| TC-2 | ₹ discount badge shows | **NOTE** | NOTE | No check-in panel to test |
| TC-3 | % discount from booking_charge | **NOTE** | NOTE | Same |
| TC-4 | Toggle type clears amount | **NOTE** | NOTE | Same |
| TC-5 | No discount → field not sent | **NOTE** | NOTE | Same |
| TC-6 | With discount → field sent | **NOTE** | NOTE | Same |
| TC-7 | Buttons disabled when busy | **NOTE** | NOTE | Same |
| R-1 | CheckInPage discount still works | **NOTE** | NOTE | Data dependency |
| R-2 | Collect now section below discount | **NOTE** | NOTE | Same |
| R-3 | Upgrade flow unaffected | **NOTE** | NOTE | Same |

**Browser: 0 tested, 10 NOTE (all blocked by no arrival data)**

---

## Environment Verification

| Check | Result |
|-------|--------|
| Front Desk Beta page loads | **PASS** — `/pms/front-desk-v2?tab=arrivals` |
| Page structure correct | **PASS** — KPI cards, tabs, table columns |
| Room count shows | **PASS** — 25 free, 13 occupied |
| Sidebar navigation | **PASS** — Rooms & Reservations → Front Desk (Beta) |

---

## Registry Spot-Check

```
BUG-489: GATE_5A_IMPLEMENTED → QA_PASSED (code-verified) ✅
CR-407:  GATE_5A_IMPLEMENTED → QA_PASSED (code-verified) ✅
Registry: SYNCED
```

---

## Data Blocker

Palmhouse preprod has 0 arrivals across all tabs (Late/Today/Tomorrow/Upcoming). Room Discount UI (TC-1–TC-7) cannot be browser-tested without booking data.

**Owner Smoke required:**
1. Create booking via "New Booking" on front-desk-v2
2. Expand check-in panel → verify Room Discount field
3. Test ₹500 → badge shows −₹500
4. Test 10% → badge computes from booking_charge
5. Confirm → verify `room_discount` in Network FormData

---

## Summary

- **Code-level: 13/13 PASS**
- **Browser: 0/10 (blocked by data)**
- **Blockers: NONE (code verified)**
- **Registry: SYNCED**
- **Result: QA PASSED (code-verified) — Ready for Gate 6 (Owner Smoke)**
