# Lincoln Place Media Budget FY26/27 — Session Feedback & Change Log
**Prepared by:** Claude (Sunny Advertising AI Assistant)  
**Session date:** June 2026  
**Tool URL:** https://lincoln-budget-planning-fy27.vercel.app/

---

## 1. Overview

This document summarises all discussions, bug fixes, data changes, and outstanding items from our planning session on the Lincoln Place Media Budget FY26/27 dashboard tool.

The tool is a **single-file HTML app** (`index.html`) deployed via Vercel, with:
- Google Sheets persistence (JSONP/POST via Apps Script)
- localStorage fallback (timestamp-based merge — newer source wins)
- 15 communities across VIC, NSW, QLD

---

## 2. Bugs Fixed This Session

### Bug 1 — Data Not Saving on Reload (total_media wiped)
**Root cause:** `applyDefaults()` was unconditionally overwriting user-edited `total_media` with COMM_DEF values on every page load.  
**Fix:** Introduced `_DEFHASH` constant (a hash of all COMM_DEF `total_media` sums). `applyDefaults()` now only re-seeds `total_media` when either:
- The COMM_DEF hash has changed (i.e. a code-level brief update), OR
- The field is completely empty

### Bug 2 — Data Not Saving on Reload (timestamp bootstrap)
**Root cause:** On first load after the timestamp system was introduced, both Google Sheets and localStorage had `_ts = 0`, so Sheets always "won" and overwrote local changes.  
**Fix:** Boot sequence now sets `lp_ts = '1'` in localStorage when data exists but no timestamp is present, ensuring local data takes precedence over a stale Sheets blob.

### Bug 3 — Stale Sheets Data Overwriting Local Changes
**Root cause:** `loadData()` blindly applied Sheets data without checking freshness.  
**Fix:** `_ts` timestamp embedded in the Sheets blob on every save; compared against `lp_ts` in localStorage on load. Newer source always wins.

### Bug 4 — Cleared COMMITTED Cells Refilling on Reload
**Root cause:** `applyCommitted()` refilled any empty (`''`) cell regardless of whether the user had deliberately cleared it.  
**Fix:** Introduced `_clr` array per community tracking explicitly cleared cells. `applyCommitted()` now skips cells in `_clr`. The array auto-persists through existing Array.isArray restore logic.

---

## 3. Data Changes Made (COMMITTED Channels)

All values below are **ex-GST** and are stored in the `COMMITTED` constant in `index.html`.

### Griffith (GRIFFITH) — Added TV, Radio, Press
Calculated as a percentage of monthly `total_media` budget (ex-GST):
- **TV at 15%:** `[2045,2045,2864,2864,2864,2455,2455,3273,3682,2864,4091,2864]`
- **Radio at 15%:** `[2045,2045,2864,2864,2864,2455,2455,3273,3682,2864,4091,2864]`
- **Press at 12%:** `[1636,1636,2291,2291,2291,1964,1964,2618,2945,2291,3273,2291]`

Note: CPL is flat at ~$545 because `leads` and `total_media` in COMM_DEF are perfectly proportional across all months (both scale at the same ratio). This is mathematically correct, not a bug.

### Eagle Point / LLEP — Added Meta + Google
Calculated at **30% of monthly total_media budget** (ex-GST):
- **meta_google:** `[8021,8021,9626,7969,4427,5313,2214,4427,5313,6641,6641,6641]`

### REA Listing (all communities with rea_listing > 0)
`REA_PP = [1426,1426,...×12]` — ex-GST value of $1,568.16 inc-GST per month across all 12 months.

---

## 4. Eden Gardens — Requirements Added to Dashboard

**Status:** ✅ Implemented and deployed (commit `7b959eb`)

### What was added to COMM_DEF

| Field | Values |
|-------|--------|
| **Leads** | `[30, 30, 45, 45, 40, 30, 45, 45, 30, 40, 40, 30]` — **450 total FY27** |
| **Total media (inc-GST)** | `[19800, 19800, 29700, 29700, 26400, 19800, 29700, 29700, 19800, 26400, 26400, 19800]` |
| **Total media (ex-GST display)** | `[18000, 18000, 27000, 27000, 24000, 18000, 27000, 27000, 18000, 24000, 24000, 18000]` |

### Eden FY27 Budget Breakdown (ex-GST)
- Low months (Jul, Aug, Dec, Mar, Jun): **$18,000/mo**
- Peak months (Sep, Oct, Jan, Feb): **$27,000/mo**
- Mid months (Nov, Apr, May): **$24,000/mo**
- REA Listing: $1,426/mo (ex-GST)
- **Annual total (ex-GST inc REA): ~$287,112**

### Eden Events Calendar (FY26/27)

| Month | Event |
|-------|-------|
| Jul | Completed Stock Brand pillars Display Home Open + OFI |
| Aug | Completed Stock Brand pillars Display Home Open + Online video series + OFI |
| Sep | Campaign Spring + OFI |
| Oct | Campaign Spring + Oktoberfest Event + OFI |
| Nov | Campaign Family Influencer Xmas Convo |
| Dec | Campaign Family Influencer Xmas Convo |
| Jan | Campaign Family Influencer Xmas Convo + Family Open Day + OFI |
| Feb | Campaign Settle & Save + OFI |
| Mar | Info session / Live webinar + OFI |
| Apr | *(not confirmed)* |
| May | Webinar: Downsizing + OFI |
| Jun | Webinar: Downsizing (replay) |

> **Note:** April events were not visible in the source screenshot — confirm with the Eden team and update in the tool if needed.

---

## 5. Technical Architecture Reference

### Boot Sequence
1. `initS()` — seeds default state from COMM_DEF
2. Load localStorage → merge data → bootstrap `lp_ts` if absent
3. `applyDefaults()` — re-seeds rea_listing, leads, events always; re-seeds total_media only if COMM_DEF hash changed or field is all-empty
4. `applyFY26()` — applies prior-year actuals
5. `applyCommitted()` — fills empty (non-cleared) cells from COMMITTED constant
6. `renderAll()` — draws all inputs
7. `loadData()` — async Sheets load; applies only if `_ts` ≥ `lp_ts`

### Key Constants
| Constant | Purpose |
|----------|---------|
| `COMM_DEF` | Source-of-truth community briefs (inc-GST values) |
| `COMMITTED` | Hard-coded channel allocations (ex-GST); fills empty cells on load |
| `REA_PP` | REA listing fee array (1426 ex-GST × 12 months) |
| `_DEFHASH` | Hash of COMM_DEF total_media sums; triggers re-seed on code-level changes |
| `S[id]._clr` | Per-community array of explicitly cleared cell keys; prevents refill on reload |

### Month Index Reference
`Jul=0, Aug=1, Sep=2, Oct=3, Nov=4, Dec=5, Jan=6, Feb=7, Mar=8, Apr=9, May=10, Jun=11`

---

## 6. Outstanding Items / To Do

- [ ] **Eden April events** — confirm with Eden team; April event field is currently blank
- [ ] **Northern Beaches events** — all 12 months are currently blank in COMM_DEF; confirm campaign calendar
- [ ] **Mackay / Yeppoon / Coral Cove events** — all blank; confirm with QLD teams
- [ ] **Wangaratta July/August** — no events confirmed for these months; holding at base digital activity
- [ ] **Griffith CPL review** — flat ~$545 CPL is mathematically correct but worth confirming lead targets are proportional to budget month-by-month with the Griffith team

---

## 7. Commits This Session

| Commit | Description |
|--------|-------------|
| *(prior session)* | rea_listing update — REA_PP applied to all communities |
| `7b959eb` | Eagle Point meta_google + Eden Gardens leads, budgets & events |

---

*Generated by Claude · Sunny Advertising · Lincoln Place Media Budget FY26/27*
