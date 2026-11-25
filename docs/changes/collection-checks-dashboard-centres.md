# Collection Checks, Twice-Collected Items, Fixed Basket, Dashboard Colours & Centres (November 2026)

Plain-language summary of the changes made after the meeting with Julie
("Collection" and "Dashboard & access" items): **what changed, how to use it,
the settings, and what to keep in mind.** Written for statisticians, supervisors
and system administrators.

Nothing changes for a country until it uses the new features: no item is
marked "twice per period" until someone sets it, no user is limited to a centre
until centres are assigned, and the range check only adds flags.

---

## 1. Range check at entry

**What it does.** Every price typed on a questionnaire, on the web or on the
mobile app, is compared with an *expected range*, and is **flagged, never
blocked**, when it falls outside it.

**How the range is worked out** (for the item's standard quantity):

| Situation | Expected range |
|---|---|
| The outlet item has a previous price | last observed price before this month **± tolerance** |
| No previous price yet | the outlet item's base price ± tolerance |
| The item has a *lowest* and/or *highest plausible price* | those replace the matching bound |

- **Tolerance**: 30% by default. Change it for everyone with the system
  parameter `hcpi_data_collection.range_tolerance_pct`, or per item
  (*Item → Collection tab → Range Tolerance (%)*).
- Prices are first converted to the standard quantity (e.g. 1,200 for 2 kg is
  600 for a 1 kg standard), so the check compares like with like.

**Where you see it.**

- *Questionnaire*: a **Range Check** badge per line (green *In range*, orange
  *Above/Below range*, grey *No reference*), orange rows for prices to check, a
  colour key, and an **Out of Range** button listing them.
- *Observation line*: a badge per visit price, which updates **while you type**,
  and the expected range in words, e.g.
  `10,500.00 – 19,500.00 (last price 15,000.00 ± 30%)`.
- *Mobile app* (new version): the expected range under the price field, an
  amber warning when the price is outside it, and a "Save anyway / Check again"
  prompt. It works offline, because the range is downloaded with the
  questionnaire. Older app versions keep working without the hint, and the
  new app keeps working against servers that are not upgraded yet.

**What to keep in mind.**

- The reference is the *latest known* price, so it improves as soon as the
  previous month is promoted to observations.
- The older "unusual compared with history" check is still there as an
  optional column (*Unusual vs history*). It no longer colours the rows, because
  it flagged many normal changes.
- First runs on UG data flagged real problems, for example a box of 144 spokes
  priced against a per-spoke reference, and dried fish at 31 per kg against
  last month's per-kg price. Typically these are a wrong quantity or unit.

## 2. Twice-collected items

**What it does.** Items such as fresh produce can be priced **twice per
period** (e.g. every two weeks).

- Mark the item: *Item → Collection tab → Collection Frequency = Twice per period*.
- Bulk questionnaire creation (core and country wizards) then produces, for each
  outlet, the usual questionnaire **plus a separate "Twice per period"
  questionnaire** holding only those items. Any line created for a twice item
  on the normal questionnaire is moved automatically, and the reverse also
  applies.
- Record **one price per visit** on the line. Each price is converted to the
  standard quantity, then the **average** is the line's price. A line counts
  as complete with 2 prices or a collection code. Progress reflects this.
- One questionnaire per outlet, month and frequency.
- Validation, imputation and promotion are unchanged: they use the line's
  averaged price.

## 3. Fixed basket

The same product specification is priced for the whole basket period (~5 years).

- Once an item has been priced in the basket period, its **description,
  standard quantity, unit and "is standard" flag are locked**. The item form
  shows a *Fixed basket* banner with the first priced date.
- Only **Item managers** can still change them (to correct a typing error).
  They get a warning, and a note is written in the item's history.
- A priced outlet item's **product** can only be switched by **Outlet
  managers**. Normally, add a new outlet item (replacement) and archive the old
  one. Switching re-labels the outlet item's whole price history.
- At rebasing, set the system parameter `hcpi_item.basket_period_start`
  (YYYY-MM-DD) to the new basket start. Specifications then unlock until the
  item is priced in the new period.

## 4. Dashboard colours and the colour key

Each colour has **one meaning per screen**. A **Colour key** button on both
dashboards explains them.

**CPI dashboard (index):**

| Colour | Meaning |
|---|---|
| Red ▲ | index rose / is higher |
| Blue ▼ | index fell / is lower |
| Grey | unchanged |
| Amber | provisional estimate (month still being collected) |
| Heatmap | blue shades below the base period (100), red shades above |
| Line/slice colours in analysis charts | only tell series apart (see the chart legend) |

Also fixed: the *CPI by division* table showed an "up" arrow for any move larger
than 1 point, including falls.

**Operations dashboard:** green = done/on track, amber = needs attention,
red = problem to fix, grey = nothing to report. Validation flags use the same
colours as the validation screens: outlier orange, extreme red, inlier blue,
imputed purple.

**Matrix of events (Uganda):** cells are tinted red (rose), blue (fell) or grey
(within 0.05%). A key is shown on screen and the same colours are used in the
Excel export.

## 5. Centre scoping

A **centre** is a consumption segment (e.g. Arua, Gulu). A city with income
groups, such as Kampala, has several segments. A user can be given several centres.

- Set it on the user: *Settings → Users → HCPI Centres*. **Leave empty for
  national users** (they see everything, as before).
- Users with centres see **only their centres on data screens**: outlets,
  outlet items, observations, questionnaires (lines and prices), validations
  (and lines).
- **Dashboards show everything** and open on the user's own centres. An **Area**
  selector switches between *National*, *My centres* and any single centre:
  - CPI dashboard: the centre's CPI, divisions, trend and growth come from the
    published basket indices. For several centres they are weight-averaged, and
    only months where every selected centre has a value are used. The
    provisional estimate is national only.
  - Operations dashboard: coverage, questionnaires, validation counts, missing
    items and index completeness are filtered to the centres.
- **Index computations** cover the whole country, so they must be started by a
  national user. A centre user gets a clear message.
- **Imputation** still uses donor prices from other centres, even when run by
  a centre user.

Also changed: collectors now download only the lines of **their own**
questionnaires to the mobile app. Before, every questionnaire line in the
database was sent to every collector, which made syncing slow as data grew.

Zanzibar uses one consumption segment with several *collection centres*.
Scoping by ZAR collection centre is a possible follow-up. Segment scoping has
no effect there.

---

## Settings summary

| Setting | Where | Default |
|---|---|---|
| `hcpi_data_collection.range_tolerance_pct` | System parameter | 30 |
| Range tolerance / lowest / highest plausible price | Item → Collection tab | empty (use default) |
| Collection frequency | Item → Collection tab | Once per period |
| `hcpi_item.basket_period_start` | System parameter | empty (all history) |
| HCPI Centres | User form | empty (national) |
