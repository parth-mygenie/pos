# QA Deferred — 31 Items Without Primary QA Handover Doc
**Role:** INVESTIGATION (read-only, no edits)
**Date:** 2026-10-22 | **Branch:** main @ 2dc8994

These items are GATE_5A_IMPLEMENTED but cannot be QA'd yet — no primary QA handover
doc exists. A QA handover doc must be written by an IMPLEMENTATION agent (or the original
implementer) before QA can run. Owner decides priority.

---

## GROUP A — User-Facing, P0/P1 (write docs first)
**11 items — highest urgency**

| ID | Sprint | Priority | Title | Notes |
|----|--------|----------|-------|-------|
| BUG-573 | nov_cr_batch | P0 | Order card variation: selected option not shown (shows group name only) | Most recent, in active sprint |
| BUG-599 | nov_cr_batch | P0 | Aggregator orders vanish on page refresh — useRefreshAllData replaces refs | Most recent, in active sprint |
| BUG-560 | oct_cr_batch | P0 | Firebase getMessaging() async unsupported-browser rejection escapes try/catch | No crash guard |
| BUG-580 | oct_cr_batch | P0 | Discount report — restaurant-specific discount types wrong (e.g. Palmhouse) | Report data incorrect |
| BUG-598-A | oct_cr_batch | P1 | Reports Date Range Caps Too Restrictive — 8 Pages Still at 60d/365d | Date range UX |
| CR-164 | oct_cr_batch | P1 | Send Payment Link from Daily Reports (Per-Row Action) | New feature |
| BUG-183 | pos_5_0 | P1 | Daily Report — Phone number and customer name missing in Credit tab | Data missing |
| BUG-402 | pos_pms_1 | P1 | Extend Stay — extra nights rate shows ₹0 (wrong dateRateMap path + nights calc) | Money |
| CR-119 | pos_5_1 | P1 | Aggregator Food Mapping — POS Menu to Platform Item Mapping | Feature |
| BUG-488 | oct_bug_batch | ? | 'To Room' transfer always sends source_order_id:'' — tableTransform.from() reads undefined | Payload bug |
| BUG-509 | oct_bug_batch | ? | CheckInForm computeBase universal subtraction — **GATE_5A_IMPLEMENTED_WITH_DEFECT** ⚠️ | Known defect — needs investigation before QA doc |

---

## GROUP B — User-Facing, P2/P3
**8 items**

| ID | Sprint | Priority | Title |
|----|--------|----------|-------|
| CR-422 | oct_cr_batch | P2 | Last Month filter — replace "Last 30 Days" across 13 report pages |
| BUG-191 | pos_5_0 | P2 | Customer Intelligence — Phone number missing in Insights section |
| BUG-416 | pos_pms_1 | P2 | Night Audit + Revenue Dashboard — no Sidebar navigation and no Back button |
| BUG-461 | sep_bug_closure | ? | Sub-recipes appear in Bulk Edit Ingredients |
| BUG-510 | oct_bug_batch | ? | CheckInForm/CheckInPage maxPct toFixed(2) rounds DOWN — entering state wrong |
| BUG-415 | pos_pms_1 | P3 | Room Orders Report — Y-axis ticks show ₹0k or collide at low values |
| BUG-462 | sep_bug_closure | ? | Order Entry: Empty-state not shown on first boot for non-Normal restaurants **(regression-only mention exists)** |
| BUG-464 | sep_bug_closure | ? | Order Entry chip reads "FOOD MENU Menu" — word Menu duplicated **(regression-only mention exists)** |

---

## GROUP C — Blocked / Architecture Hold (do NOT write QA doc yet)
**2 items**

| ID | Sprint | Title | Why blocked |
|----|--------|-------|-------------|
| BUG-463 | oct_cr_batch | Local Settings Active Menu: Two visually identical Promotional Menu picks | Architecture on hold — waiting on backend confirmation of `food_for` filter. Session handover 2026-10-18 explicitly says: "Do NOT implement LoadingPage change until backend confirms Q1/Q2." |
| BUG-499 | oct_bug_batch | Both room+F&B discount sends wrong amounts **(regression-only mention exists)** | Regression mention in BUG-516_519 handover — needs dedicated test cases written |

---

## GROUP D — SUBSUMED (no QA needed — verify closure only)
**2 items**

| ID | Sprint | Title | Action |
|----|--------|-------|--------|
| BUG-312 | pos_5_0 | fromAPI.ingredients() missing isSubRecipe/subrecipeId — SUBSUMED by CR-139 | Verify CR-139 is CLOSED/OWNER VERIFIED, then close BUG-312 as SUBSUMED |
| BUG-313 | pos_5_0 | Sub-recipe in Stock Update auto-plan — SUBSUMED by CR-139 | Same as above |

---

## GROUP E — Internal Tooling (no browser QA needed, code/grep only)
**6 items**

| ID | Sprint | Title |
|----|--------|-------|
| CR-406 | tooling_oct1 | Google Sheets Two-Way Sync — Registry Export/Import via OAuth2 |
| CR-416 | tooling_oct1 | AGENT_PROMPT_ALPHA: Add SHEET PUSH + SHEET DRIFT Roles |
| CR-419 | tooling_oct1 | sheets_sync.py: Contract v1.5 Compliance Rewrite |
| CR-420 | tooling_oct1 | sheets_sync.py: Post-Push QA Gap Fixes |
| BUG-541 | tooling_oct1 | SHEET DRIFT only diffs 3 columns — misses Priority |
| CR-428 | tooling_oct1 | Area module migration — 18-value list → 13 modules |

**QA approach when owner wants these done:** `grep` + `python3 scripts/sheets_sync.py` verification. No browser needed.

---

## GROUP F — Old Backlog (low urgency)
**2 items**

| ID | Sprint | Priority | Title |
|----|--------|----------|-------|
| CR-370 | pos_audit_1 | P2 | Stale Doc Corrections — 5 Records from Audit §3 *(doc-only change — verify doc content, no browser test needed)* |
| PROD-003 | ? | ? | PayLater table clear *(stale hotfix from sprint unknown)* |

---

## SUMMARY

| Group | Count | Action |
|-------|-------|--------|
| A — User-facing P0/P1 | 11 | Write QA handover docs — then QA can run |
| B — User-facing P2/P3 | 8 | Write QA handover docs — lower urgency |
| C — Blocked/Architecture | 2 | Wait for owner/backend decision first |
| D — Subsumed | 2 | Verify CR-139 closure, then close |
| E — Internal tooling | 6 | Grep/code verification only (no handover needed) |
| F — Old backlog | 2 | Low urgency, owner decides |
| **Total** | **31** | |
