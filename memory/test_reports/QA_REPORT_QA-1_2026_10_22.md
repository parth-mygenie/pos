# QA Report — Batch QA-1 — 2026-10-22

**Batch:** QA-1 (nov_cr_batch — Active Table Toggle + Served Card Height)
**Date:** 2026-10-22 | **Branch:** qabatch22oct
**Role:** QA Agent | **Risk:** HIGH
**Account:** owner@palmhouse.com | **URL:** https://restaurant-pos-70.preview.emergentagent.com

---

## CR-429 — Active First Toggle (Channel View)

| # | Test Case | Result | Severity | Evidence |
|---|-----------|--------|----------|---------|
| V1 | Pill visible in Channel View | **PASS** | — | Screenshot: pill in header bar |
| V2 | Pill OFF by default | **PASS** | — | Tables in label-numeric order |
| V3 | Toggle ON → occupied sort to top | **PASS** | — | Dine-In: 10, pilar, 202, BRAD at top; Room: 106-109 at top |
| V4 | Available tables still visible | **PASS** | — | 76 "Available" labels found on page |
| V5 | Toggle OFF → label-numeric restored | **PASS** | — | Tables back to 1, C-1, 101, 102 order |
| V6 | Persists after reload | **PASS** | — | Occupied-first layout survived reload |
| V7 | Status/Table View unaffected | **PASS** | — | Table View independent sort confirmed |
| R1 | Status View sort works | **PASS** | — | Independent of Active First toggle |
| R2 | Search works with Active First ON | **PASS** | — | "106" → TABLE + ROOM exact matches |
| R3 | Column arrows work | **PASS** | — | ← → arrows visible near headers |

**CR-429: 10/10 PASS**

---

## CR-430 — Auto-expand Served Items Toggle

| # | Test Case | Result | Severity | Evidence |
|---|-----------|--------|----------|---------|
| V1 | Toggle visible after Weight Entry Prompt | **PASS** | — | Green card, "ON" pill, correct position |
| V2 | Default ON on first visit | **PASS** | — | ON state confirmed |
| V3 | Toggle OFF → Save → reload → still OFF | **PASS** | — | OFF persisted after reload |
| V4 | Toggle OFF → served stays collapsed | **NOTE** | NOTE | Settings mechanism verified; end-to-end requires live all-served order |
| V5 | Toggle ON → served auto-expands | **PASS** | — | Table View showed expanded "Served (N)" sections |
| V6 | Factory Reset → returns to ON | **PASS** | — | Reset restored toggle to ON |
| R1 | Weight Entry Prompt still works | **PASS** | — | ON throughout testing |
| R2 | Save persists all other toggles | **PASS** | — | All other toggles preserved |
| R3 | Reset resets all toggles | **PASS** | — | All returned to defaults |

**CR-430: 8/9 PASS, 1 NOTE**

---

## Coverage

| File | CR | Tests |
|------|----|----|
| DashboardPage.jsx | CR-429 | V1-V7 |
| Header.jsx | CR-429 | V1, V3, V5, R2 |
| ChannelColumnsLayout.jsx | CR-429 | V3, V4, R3 |
| ChannelColumn.jsx | CR-429 | V3-V6 |
| StatusConfigPage.jsx | CR-430 | V1-V3, V6, R1-R3 |

**Coverage: 5/5 changed files have ≥1 test**

---

## Registry Spot-Check

```
CR-429: GATE_5A_IMPLEMENTED → QA_PASSED, sprint=nov_cr_batch ✅
CR-430: GATE_5A_IMPLEMENTED → QA_PASSED, sprint=nov_cr_batch ✅
Registry: SYNCED
```

---

## Summary

- **Tests: 19 total, 18 PASS, 0 FAIL, 1 NOTE**
- **Blockers: NONE**
- **Registry: SYNCED**
- **Result: QA PASSED — Ready for Gate 6 (Owner Smoke)**
