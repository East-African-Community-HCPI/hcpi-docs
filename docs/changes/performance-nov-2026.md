# Speed and efficiency (November 2026)

What was slow, why, what was changed, and what is planned next. Measured on the
restored country databases (UG, KE, TZ, ZAR) on the development machine; production
servers will differ in absolute numbers but not in the ratios.

## 1. What was found

| Area | Finding |
|---|---|
| **Log tables** | Every computed index row (EA, micro class … national) and every price kept chatter: a "created" message, follower rows and tracking values. **UG: 11 million messages, 5.4 GB of 8.8 GB**; TZ 4.3 million; KE 0.4 million. Nobody reads them, and each index computation was writing 3–4 extra rows per index row. |
| **Missing indexes** | Questionnaire lines had no index on their questionnaire or outlet item (TZ); validation lines none on their validation; prices none on centre; basket index tables none on (centre, month). Many screens and computations read whole tables. |
| **Month filters** | Dashboard queries compared `date_trunc('month', date) = …`, which no index can serve, so they scanned millions of rows; the month list read every price to find which months exist. |
| **Imputation** | Works one missing price at a time, with several searches per related group (EA, micro class, …). See section 3. |

## 2. What was changed (live on all four countries)

| Change | Effect (measured) |
|---|---|
| Computed index rows no longer write chatter | saving index rows **2.1× faster** (5,000 rows: 3.45 s → 1.63 s) and **0 log rows** instead of one per row |
| Prices and questionnaire lines: no "created" message or automatic follower (edits are still tracked) | stops the 3.6 million empty messages UG had from growing |
| **Clean up logs** tool (*Indices → Computations → Clean up logs*, administrators): shows what it would remove, then removes it in batches | see below |
| Indexes added: questionnaire lines (questionnaire, outlet item), validation lines (validation, outlet item), observation lines (line), prices (centre + date), every basket index table (centre + month) | created on module update; UG 12 s, KE 25 s, TZ 18 s, ZAR 11 s |
| Month filters rewritten as date ranges; month list probes the date index | month list 0.63 s → 0.01 s (TZ) |
| **Operations dashboard (TZ)** | **15.8 s → 1.7 s** |
| **CPI dashboard (KE)** | **2.1 s → 0.34 s** |
| Validation check now uses a sanity bound (see the walkthrough) | — correctness, not speed |

### Log cleanup on the UG copy

| | Before | After cleanup | After `VACUUM FULL` |
|---|---|---|---|
| Log messages | 11,046,268 | 3,836,536 | 3,836,536 |
| Follower rows | 6,559,872 | 126,761 | 126,761 |
| `mail_message` on disk | 5.4 GB | 5.4 GB | 2.0 GB |
| Whole database | **8.8 GB** | 8.8 GB | **3.8 GB** |
| Time | | 11 min (batches of 50,000) | 42 s |

On production, take a backup first and run it in a quiet period. The space is
reused straight away; `VACUUM FULL mail_message;` (an administrator, during a
maintenance window) shrinks the files on disk.

## 3. Planned next

1. **Imputation** (the slowest step a statistician waits for): on ZAR October 2025 (3,097 lines) the checks take 1.6 s and the inliers 1.0 s, but **imputation takes 133 s** for 1,525 missing prices (about 0.09 s each).
   Plan: compute each group's price change **once per validation** (item, EA, micro
   class, … in one pass with a few SQL queries) and then fill every missing price
   from that table, instead of searching again for every line. Expected: seconds
   instead of minutes. Also fix: at the class, group and division levels the
   "previous" index is taken as the latest EA index on file, which after a
   recomputation can be the current month itself; it should be the month before.
2. **Index computations**: measure each of the four steps end to end on UG now that
   the chatter is gone, then batch the per-row work the same way.
3. **Questionnaire → validation**: lines are created one by one; create them in one
   batch (UG and ZAR; KE and TZ have their own versions).
4. **Production maintenance**: run the log cleanup and `VACUUM FULL` on each server,
   and review PostgreSQL memory settings (`shared_buffers`, `work_mem`) and Odoo
   workers against each server's RAM.
5. **Operations dashboard**: load the page in two stages (headline figures first,
   gap lists after) so it appears at once even on the largest country.
