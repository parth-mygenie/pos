# QA Batch — All IMPLEMENTED, QA Not Done
**Prepared by:** INVESTIGATION Agent (E1)
**Date:** 2026-10-22
**Role:** INVESTIGATION → producing QA batch brief
**Source docs read:** AGENT_PROMPT_ALPHA.md, CR_REGISTRY.md, BUG_TRACKER.md, registry.json, all QA handovers, SESSION_HANDOVER_2026_07_11_FULL_DAY_CLOSE_FINAL.md

---

## INVESTIGATION SUMMARY

**Finding:** 22 items are status=IMPLEMENTED in registry.json with no QA agent sign-off (Gate 5b).
The most recent QA report in `/app/memory/test_reports/` is from 2026-06-18. Every fix implemented after that date has only Gate 5a (self-test) — no QA agent execution.

**Root cause of gap:** Multiple implementation sessions (2026-06-17 through 2026-07-12) produced Gate 5a self-tests and QA handover docs but no QA agent was called to execute them.

**Registry drift detected (1 item):** BUG-135 shows `INTAKE COMPLETE` in registry.json but BUG_TRACKER.md records it as `IMPLEMENTED (2026-07-10 code-verified)`. Per Rule R1 (code is truth), it is IMPLEMENTED and needs QA.

**Excluded from batch (with reason):**
- CR-046, CR-048: internal dev dashboard tools, not user-facing POS app
- BUG-168: owner disputes scope, investigation formally reopened — do NOT QA until scope resolved
- CR-053: registry says "IMPLEMENTATION IN PROGRESS" — not complete

---

## CREDENTIALS (all batches)

| Account | Email | Password | RID | Use for |
|---------|-------|----------|-----|---------|
| cafe103 | owner@cafe103.com | Qplazm@10 | 644 | All expense, sidebar, insights, socket tests |
| 18March | owner@18march.com | Qplazm@10 | — | Room order / paid_room tests |
| palmhouse | owner@palmhouse.com | Qplazm@10 | — | Insights, room features |

Login endpoint: `POST https://preprod.mygenie.online/api/v1/auth/vendoremployee/login`
App URL: https://core-pos-frontend-13.preview.emergentagent.com

---

## BATCH QA-001 — Financial / Order Transform
**Priority: P0 — test FIRST. Money is involved.**
**Risk: CRITICAL (BUG-138) + MEDIUM (BUG-166, BUG-VQTY, BUG-ROOM-PAIDROOM)**
**QA handover for VQTY + PAIDROOM:** `/app/memory/handover/QA_HANDOVER_2026_07_11_BUG163_VQTY_PAIDROOM.md`

### BUG-138 — Discount Payload Wrong (self_discount/order_discount)
**File changed:** `orderTransform.js` (discount builder section)
**Risk:** CRITICAL — money

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Apply restaurant discount | Open order → apply % discount from Collect Bill → inspect Network payload on submit | `order_discount` = total discount applied; `self_discount` = agent's own discount portion only |
| T2 | Apply waiter self-discount | Collect Bill → enter self-discount amount | `self_discount` = correct amount (NOT total incl. coupon/loyalty/wallet) |
| T3 | Apply coupon + order discount | Apply a coupon AND an order discount together | Coupon amount excluded from `self_discount`; `order_discount` correct sum |
| T4 | No discount | Place order with no discount | `self_discount = 0`, `order_discount = 0` |
| T5 | QSR mode discount | QSR Collect Bill with discount | Same payload rules apply as Full mode |

Regression: R1 — order total unchanged after discount fix. R2 — loyalty points calculation unaffected.

---

### BUG-166 — addon_amount Not ×qty in Order Payloads
**File changed:** `orderTransform.js` L704 + L1493
**Risk:** MEDIUM — billing line totals wrong for addon items with qty>1

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T6 | Place order: item with addon qty=1 | Add item with addon (e.g. extra cheese ₹30), qty=1, place order | `addon_amount = 30` in payload |
| T7 | Place order: item with addon qty=3 | Same item, change qty to 3, place order | `addon_amount = 90` in payload (was 30 before fix) |
| T8 | Collect Bill: item with addon qty=3 | Collect Bill for above order | `addon_amount = 90` in BILL_PAYMENT payload |
| T9 | Item with NO addon | Regular item without addon, qty=2 | `addon_amount = 0` |

Regression: R3 — variation_amount still correct (BUG-VQTY fix intact). R4 — grand total unchanged.

---

### BUG-VQTY — variation_amount Not ×qty (QA handover exists)
Refer to: `/app/memory/handover/QA_HANDOVER_2026_07_11_BUG163_VQTY_PAIDROOM.md` — tests T4–T8

### BUG-ROOM-PAIDROOM — paid_room Field Fix (QA handover exists)
Refer to: `/app/memory/handover/QA_HANDOVER_2026_07_11_BUG163_VQTY_PAIDROOM.md` — tests T9–T10

---

## BATCH QA-002 — Expense Module CRUD + DnD
**Priority: P0-P1 — expense data integrity**
**Risk: MEDIUM (DnD, CRUD ops)**
**QA handover for DND:** `/app/memory/handover/QA_HANDOVER_DND_CR059_2026_07_08.md`
**QA handover for BUG-163:** `/app/memory/handover/QA_HANDOVER_2026_07_11_BUG163_VQTY_PAIDROOM.md` — tests T1–T3

Route: `/expense-setup` (cafe103)
Prerequisite: CR-059 QA PASSED — basic expense entry + DnD foundation already verified.

### BUG-158 — addItem to Category Silently Failed
**File:** `ExpenseSetupPanel.jsx` — `addItem()` now calls `createCategoryWithItems(cat.name, [newItemName])`

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Add item to existing category | /expense-setup → select a category → type new item name in input → press Add | Item appears in category list. Toast "Item added". |
| T2 | Persists after page refresh | After T1, refresh page | Item still present in that category |
| T3 | Add item to each category type | Repeat T1 for at least 2 different categories | Both persist correctly |

---

### BUG-159 — addCategory Silently Failed
**File:** `ExpenseSetupPanel.jsx` + `expenseService.js` + `constants.js` — `addCategory()` now calls `createEmptyCategory()`

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T4 | Add new category | /expense-setup → type category name in input → press Add Category | New category pill appears in left panel. Toast "Category added". |
| T5 | Persists after refresh | After T4, refresh page | Category still present |
| T6 | Add item to new category | Add item to the category created in T4 | Item appears; persists on refresh |

---

### BUG-160 — Rename/Delete Category Broken
**File:** `expenseService.js` + `ExpenseSetupPanel.jsx`

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T7 | Rename a category | /expense-setup → click rename (pencil) on a category pill → enter new name → confirm | Category pill updates. Persists after refresh. |
| T8 | Delete a category | Click delete (trash) on a category → confirm | Category removed from left panel. Persists after refresh. Not "Category not found" error. |

---

### BUG-161 — Bulk Save New Items Silently Failed
**File:** `ExpenseSetupPanel.jsx` — `handleBulkSave()` now uses `createCategoryWithItems`

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T9 | Add items via bulk editor | /expense-setup → open bulk editor (if present) → add 2 rows → click Bulk Save | Both items saved. Appear in category list. Persist on refresh. |

---

### BUG-164 — Duplicate Category Shows Success Toast
**File:** `ExpenseSetupPanel.jsx` — `addCategory()` inspects `res.data.errors[0]` on HTTP 201

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T10 | Add duplicate category name | /expense-setup → add a category → try adding same name again | Error toast "Category already exists" (or similar). Category NOT added twice. |

---

### BUG-165 — Duplicate Item Silently Allowed (FE Guard)
**File:** `ExpenseSetupPanel.jsx` — `addItem()` pre-checks `allItems` state (case-insensitive, same categoryId)
**Note:** Backend fix still pending — FE guard only.

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T11 | Add duplicate item in same category | Add item "Milk" to a category → try adding "milk" again to same category | Error toast "Item already exists". Not added again in UI. |
| T12 | Same name, different category | Add "Milk" to Cat A → add "Milk" to Cat B | Both allowed (different category = not a duplicate) |

---

### BUG-150 — DnD Placeholder Display Fix
**File:** `ExpenseSetupPanel.jsx` — placeholder `display:none → height:0`; hover shows solid border + "Drop here" label

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T13 | Drag item, hover over empty category | Drag any item → hover over a category that has no items | Category highlight visible (solid border + "Drop here" text). No layout jump. |

---

### BUG-DND-CR059 — Cross-Category Move (QA handover exists)
Refer to: `/app/memory/handover/QA_HANDOVER_DND_CR059_2026_07_08.md` — tests T1–T6 + R1–R4

---

### BUG-163 — Export Body Missing {type:'all'} (QA handover exists)
Refer to: `/app/memory/handover/QA_HANDOVER_2026_07_11_BUG163_VQTY_PAIDROOM.md` — tests T1–T3

---

**Batch QA-002 Regression:**
- R1: `/expenses` daily entry page unaffected by all above changes
- R2: CR-059 base flows still work (edit, delete expense entries)
- R3: No errors on `/expense-setup` page load

---

## BATCH QA-003 — Sidebar & Navigation
**Priority: P1 — affects all routes**
**Risk: MEDIUM**
**QA handover for BUG-136:** `/app/memory/handover/QA_HANDOVER_2026_06_17_BUG_136.md`

### CR-041 — Full Sidebar Restructure
**Files:** `Sidebar.jsx`, `App.js`, + 4 new route files (Settings sub-pages, panel→route migrations)
**Changes:** 4 panel→route migrations, 4 renames, Settings parent with 8 children, Insights moved to last

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Settings parent with sub-items | Sidebar → click "Settings" | Expands to show 8 children (Restaurant, Channels, Menu Config, etc.) |
| T2 | Panel→route migrations | Click each previously-panel item (e.g. Settlement, Menu Management) | Opens as a dedicated route/page, NOT a slide-over panel |
| T3 | Renamed items | Check sidebar for renamed labels | Labels match new names per CR-041 scope (4 renames) |
| T4 | Insights at bottom | Expand sidebar fully | Insights group is the LAST section, not mid-list |
| T5 | All routes resolve | Click each sidebar item | No 404, no blank page, no console error |
| T6 | Active route highlights correctly | Navigate to a Settings child page | Correct sidebar item highlighted |

Regression: R1 — logout/login flow unaffected. R2 — Dashboard still default landing.

---

### BUG-136 — Sidebar Scroll Persistence (QA handover exists)
Refer to: `/app/memory/handover/QA_HANDOVER_2026_06_17_BUG_136.md` — tests T1–T8 + R1–R4

---

### CR-052 — Sidebar Hover Flyout
**File:** `Sidebar.jsx` (collapsed state flyout menu)

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T7 | Collapsed sidebar hover shows flyout | Collapse sidebar → hover over any nav icon | Flyout menu appears showing children / label |
| T8 | Flyout navigates correctly | In collapsed flyout, click a child item | Navigates to correct route |
| T9 | Flyout closes on mouse-out | Move mouse off flyout | Flyout disappears |
| T10 | No flyout in expanded mode | Keep sidebar expanded → hover items | No flyout (full labels visible) |

---

## BATCH QA-004 — POS Order Flow: Settings & Permissions
**Priority: P1**
**Risk: HIGH (CR-051 validation gates, BUG-130 channel control)**

### CR-051 — Customer Field Mandatoriness (6 Toggles)
**Files:** `StatusConfigPage.jsx`, `OrderEntry.jsx`
**Toggles:** Walk-in Name, Walk-in Phone, Dine-in Name, Dine-in Phone, TakeAway Phone, TakeAway Name (default ON)

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Toggle OFF Walk-in Phone → Place Order | Settings → Customer Fields → turn off "Walk-in Phone" → create walk-in order without phone → Place Order | Order placed successfully without phone field |
| T2 | Toggle ON Walk-in Phone → enforce | Turn "Walk-in Phone" back ON → try same | Place Order blocked until phone entered |
| T3 | Dine-in Name toggle | Turn off "Dine-in Name" → create dine-in order without name | Allowed |
| T4 | TakeAway Phone toggle | Turn off "TakeAway Phone" → takeaway without phone | Allowed |
| T5 | TakeAway Name default ON | Fresh order with default settings → TakeAway | Name field required by default |
| T6 | Collect Bill validation | Have mandatoriness ON for a field → try Collect Bill without it | Collect Bill also blocked |
| T7 | Settings persist | Toggle settings → reload page | Toggles still in saved state (localStorage) |

Regression: R1 — Delivery fields unaffected (they have their own mandatory rules). R2 — QSR mode parity.

---

### BUG-130 — Channel Visibility Not Reflected on Dashboard
**Files:** `StatusConfigPage.jsx`, `DashboardPage.jsx`, `restaurantSettingsTransform.js`
**Context:** Two-layer model — Restaurant-level API + per-user localStorage override

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T8 | Disable a channel in Restaurant Settings | Restaurant Settings → Channels → disable "Delivery" → save | "Delivery" channel no longer appears on POS Dashboard order grid |
| T9 | Re-enable channel | Re-enable "Delivery" in settings | Channel reappears on dashboard |
| T10 | Per-user override (StatusConfig) | Channel enabled in restaurant settings but toggled off in StatusConfig | User-level override takes precedence; channel hidden for this device |

Regression: R3 — Dashboard still loads all active channels correctly.

---

### BUG-092 — Phone Normalization + CRM Lookup on Room Check-in
**Files:** `orderTransform.js` or `orderService.js` (room check-in path)

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T11 | Room check-in with phone | Create a room check-in order with customer phone number | Phone sent in normalized format to CRM. No 4xx errors in network. |
| T12 | CRM customer lookup on check-in | Room check-in with a registered CRM customer's phone | Customer data auto-populated |

Regression: R4 — Non-room order phone field unaffected.

---

### BUG-167 — Menu Socket Lost on Non-Dashboard Routes
**File:** `AppSocketManager.jsx` (new, mounted at app level in `App.js`)

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T13 | Menu update on /menu route | Open two browser tabs: Tab A = dashboard, Tab B = /menu (Item Ledger or Menu Management). Update a menu item on preprod via another session. | Tab B ALSO receives the socket update and reflects it without needing to navigate back to dashboard first. |
| T14 | Menu update on /expense-setup | Same test but Tab B = /expense-setup | Socket still active; console shows `food_update_644` received |
| T15 | No duplicate socket events on dashboard | Navigate dashboard → check console for food_update events | No duplicate logs (AppSocketManager fires once; DashboardPage.jsx no longer calls useSocketEvents redundantly) |

Regression: R5 — Dashboard continues to receive menu updates as before.

---

## BATCH QA-005 — Insights Data Integrity
**Priority: P1 — data display correctness post-migration**
**Risk: HIGH (data shape changes may break report screens)**

### CR-045 — Stripper Deletion + getDashboardAggregated Deletion
**Changes:** `orderPayloadStripper.js` DELETED, `getDashboardAggregated` (474 lines) DELETED, `ItemSalesMockup.jsx` DELETED. ~600 lines removed. 8 files modified to call backend aggregation instead.

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T1 | Insights Dashboard loads | Login → Insights → Dashboard | Dashboard renders all KPI cards with data. No blank/error state. |
| T2 | Item Ledger loads | Insights → Item Ledger | Items table populated. Same data as before CR-045 (backend now strips, not FE). |
| T3 | Order Ledger loads | Insights → Order Ledger | Full order table. No missing columns. No console errors. |
| T4 | No JS import errors | Check browser console after navigating all Insights pages | No "Cannot find module" or "is not a function" errors from deleted files |

Regression: R1 — Phase C (audit lazy-load migration) is deferred — Audit tab should still work if present. R2 — FoodCourt + PrepServe screens load cleanly.

---

### CR-049 — Backend Aggregation Migration (4 Phases)
**Context:** FE migrated from raw 450MB payload to 4 backend aggregation endpoints (~135 KB). 8 files changed.

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T5 | Insights Dashboard KPIs | Date range: last 30 days → load Dashboard | KPI totals match same period in Insights Reports (cross-check Net Sales tile vs Order Ledger total) |
| T6 | Aggregation endpoints respond | Network tab → Insights Dashboard load | Calls to `/order-logs-aggregated` (or equivalent backend endpoint) returning 200 |
| T7 | Date range changes refresh data | Change date range picker → re-submit | Dashboard reloads with correct data for new range |
| T8 | No 450MB payload calls | Network tab while loading Insights | No calls to raw `order-logs-report` with 450MB response on initial Dashboard load |

Regression: R3 — Drill-down (side-sheet per order) still works by fetching single order on click. R4 — Export still works (Excel/PDF).

---

## BATCH QA-006 — CR-011 FE Fixes (F-1, F-2, F-5, F-6, F-10)
**Priority: P1**
**Risk: MEDIUM**
**QA Handover EXISTS:** `/app/memory/handover/QA_HANDOVER_2026_06_17_CR011_FE_FIXES.md`

Execute all 5 test cases (TC-1 through TC-5) + 6 regression tests (R-1 through R-6) from the handover document exactly as written. Credentials: cafe103 + palmhouse.

---

## REGISTRY DRIFT — ACTION REQUIRED BEFORE QA

### BUG-135 Status Conflict
- **registry.json says:** `INTAKE COMPLETE`
- **BUG_TRACKER.md says:** `IMPLEMENTED (2026-07-10 code-verified). A: toggleFoodStatus. B+C: err.readableMessage surfaced.`
- **Rule R1:** Code is truth — BUG-135 is IMPLEMENTED.
- **Action for QA agent:** Update `registry.json` BUG-135 status to `IMPLEMENTED` before testing, then add to QA-002 batch (Bulk Editor). Test:

| # | Test | Steps | Expected |
|---|------|-------|----------|
| T_BUG135_A | Status OFF persists in Bulk Editor | Bulk Editor → toggle item status to OFF → Save | Status saves correctly. Not "Partial Save 0/6". |
| T_BUG135_B | Import shows backend error message | Import invalid Excel file | Toast shows backend's error message (e.g. "Duplicate row: Fish Amritsari") NOT generic "Import failed" |
| T_BUG135_C | Duplicate item 422 shows backend message | Attempt to save duplicate item | Toast shows readable backend message, not raw Axios error |

---

## BATCH SUMMARY TABLE

| Batch | Items | Risk | QA Handover | Est. Tests |
|-------|-------|------|-------------|------------|
| QA-001 | BUG-138, BUG-166, BUG-VQTY, BUG-ROOM-PAIDROOM | CRITICAL/MEDIUM | Partial (VQTY+PAIDROOM) | 10 |
| QA-002 | BUG-DND-CR059, BUG-150, BUG-158-161, 163-165 | MEDIUM | Partial (DND+163) | 13 + regression |
| QA-003 | CR-041, BUG-136, CR-052 | MEDIUM | Partial (BUG-136) | 10 |
| QA-004 | CR-051, BUG-130, BUG-092, BUG-167 | HIGH | None | 15 |
| QA-005 | CR-045, CR-049 | HIGH | None | 8 |
| QA-006 | CR-011 F-1/F-2/F-5/F-6/F-10 | MEDIUM | EXISTS ✅ | 5 TC + 6 R |
| DRIFT | BUG-135 (registry fix first) | MEDIUM | None | 3 |
| **TOTAL** | **22 items** | | | **~64 tests** |

---

## QA AGENT EXECUTION ORDER

1. **Fix registry drift first:** Update BUG-135 status in registry.json → IMPLEMENTED
2. **Run QA-001** (financial — P0 CRITICAL first)
3. **Run QA-002** (expense CRUD — data integrity)
4. **Run QA-003** (sidebar — navigation foundation for all other tests)
5. **Run QA-004** (POS order flow)
6. **Run QA-005** (insights data)
7. **Run QA-006** (use existing handover doc)
8. **Write QA report** per batch at `/app/memory/test_reports/QA_REPORT_<DATE>_POS5_BATCH.md`

---

## WHAT QA AGENT MUST NOT DO

- Do NOT fix code. Report failures only.
- Do NOT run QA on BUG-168 (scope disputed, investigation open).
- Do NOT run QA on CR-053 (implementation in progress).
- Do NOT run QA on CR-046, CR-048 (internal dev tools — outside QA scope).
- For BUG-165: backend fix is pending. FE guard only — test FE guard works, note backend gap.
