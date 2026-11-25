# Replacements, Price Upload, Outlet Map, Chart Controls & Offline Mobile (November 2026)

Plain-language summary of the last items from the meeting with Julie
("Replacements", "Testing", "Dashboard & access" and "Mobile App"): **what
changed, how to use it, and what to keep in mind.** Written for statisticians,
supervisors and system administrators.

Nothing changes for a country until someone uses the new features.

---

## 1. Replacements (products and outlets)

**Why.** When a product is no longer sold, or an outlet closes, its price series
must continue with a similar product or outlet **without a jump in the index**
caused by the switch itself.

**The method (Julie's rule, standard CPI practice).**

| Month | What happens |
|---|---|
| **Month M** (replacement month) | The old product is *permanently missing* (code **P**). Its price for M is **imputed** in that month's validation, as any missing price, and that is what the index uses. The collector also notes the **new product's price for M**. That price is only a *link price*: it never enters the index. |
| **Month M+1 onwards** | The new product is priced instead. It now has **two prices** (M and M+1), so its monthly change is *price in M+1 ÷ link price*. |

The new product gets the base price **old base × new price (M) ÷ old price (M)**,
so its long-term index starts exactly where the old product left off. A price
difference between the two products is treated as a **quality difference, not
inflation**.

*Example.* Old base 1,000; old price in M 1,200 (level 120); new price in M
1,500. New base = 1,000 × 1,500 ÷ 1,200 = 1,250. In M+1 the new product costs
1,530: level 1,530 ÷ 1,250 = 122.4, a 2% rise on 120, the same as 1,530 ÷ 1,500.

**Where to start one.**

- *Validation*: a **To replace** button counts permanently missing (P) prices
  with no replacement yet. Each such line has a **Replace** icon that opens a
  replacement already filled in (old product, month, imputed price).
- *Outlet item*: **Replace product**.
- *Outlet*: **Replace outlet**. One outlet is usually replaced by one or two
  others: choose them, click **Load the outlet's items**, and set which new
  outlet takes each item.
- *Outlets → Replacements*: every replacement and its status.

**Statuses.**

- **Draft**: being prepared; nothing has changed yet.
- **Waiting for prices**: confirmed, the new outlet item exists, but one of the
  two month-M prices is not known yet. It links **by itself** when month M's
  validation is promoted. For an outlet replacement with no new price typed in,
  it links in the first month the new outlet is priced. You can also type the
  missing price and click **Link now**.
- **Linked**: the new product has its base price; the old one is archived with
  its whole price history kept.

**What the system takes care of.**

- Promotion never turns a link price into an index price, and never promotes the
  old product after month M; the promotion screen lists any such lines.
- Open questionnaires (draft / price survey) switch to the new product; lines
  that already hold a price are never touched.
- Validation, the range check and the monthly price change all use the link
  price as the "previous price" of the new product in its first month.
- If month M is promoted again with a different imputed price, the base price
  is recalculated and logged on the replacement.
- **Cancel replacement** puts the old product back on the questionnaires. It is
  refused once the new product has index prices (make a new replacement then).

Who can confirm: outlet managers and senior statisticians.

---

## 2. Upload Collected Prices (testing and paper collection)

*Data Collection → Upload Collected Prices* (supervisors and statisticians).

1. Choose the month (and optionally the segments) and click **Download
   template**: one row per active outlet item, with its **last price** to help
   spot typing errors, a *Read Me* sheet and the list of collection codes.
2. Fill in the yellow columns: **Price**; **Quantity / Unit** only if the price
   is not for the standard quantity; **Price (2nd visit)** for twice-collected
   items; a **Collection Code** (e.g. T, P) when there is no price; a remark.
3. Upload the file and click **Check file**. Nothing is saved yet: you see how
   many rows will be imported and every row that is skipped, and why (unknown
   item code, not a number, unknown unit or code, questionnaire already in
   validation).
4. Click **Import**. The prices go into the month's questionnaires exactly as if
   collectors had entered them; with **Send to validation** ticked (default)
   they then go to that month's Data Validation, using each country's own
   validation rules. From there: Validate, inliers, Impute, Finalize, Promote,
   as usual.

Rows left empty stay "not collected" and are imputed. Questionnaires already in
validation are never changed. CSV files work too (columns *Item Code*, *Price*,
and optionally *Quantity*, *Unit*, *Collection Code*, *Remark*).

Tested on Zanzibar: 3,097 rows checked, 3,031 imported in about 25 seconds,
then the whole pipeline produced 3,089 observations (109 imputed).

---

## 3. Dashboard controls, centre map and outlet positions

**Charts (CPI dashboard).**

- Every time chart has the same toolbar: **zoom in, zoom out, drag to zoom,
  pan, reset, download**.
- **1Y / 3Y / 5Y / All** buttons on the national CPI trend and on the division
  trends.
- A **full screen** button on every chart (Esc to close).

**Collection centres map (both dashboards).** Each collection centre
(consumption segment) is a circle on a map, sized by its weight and labelled
with its figure:

- *CPI dashboard*: **Annual inflation** (default), **Monthly change** or
  **Index level**, coloured with the dashboard's rules: red shades = prices
  rose, blue shades = fell, grey = about unchanged or no figure.
- *Operations dashboard*: **Collection progress** (default: green 95%+ of
  questionnaires done, amber 50–94%, red below 50%), plus the two changes.
- Click a centre for its index, monthly and annual change, questionnaires done
  and outlets, and **Show this centre on the dashboard**, which switches the
  dashboard's Area to it.
- Labels that would overlap are hidden until you zoom in; **Fit all** and full
  screen are in the header.
- *Positions*: filled in automatically from the centre's name (its town or
  regional capital) and marked **approximate**; type the exact position on the
  consumption segment (*Outlets → Consumption Segments*). All UG, KE, TZ and ZAR
  centres were matched.
- *Zanzibar* collects by smaller collection centres: in Collection progress
  view the map shows its 10 active collection centres (positions on the
  collection centre form) instead of the single Zanzibar segment.

**Outlet map (operations dashboard).** One dot per outlet coloured by this
month's collection status (green done, amber in progress, red not started, grey
no questionnaire), with the legend as a filter. **No country has usable outlet
positions yet**: Kenya's are all 1, 1; Zanzibar's share one latitude and have
no longitude; Uganda and Tanzania have none. Such placeholders are ignored, and
the map shows a short note until real positions arrive.

**Outlet positions from the field.** The server now accepts an outlet's
position captured by a collector at the outlet (from the questionnaire, for the
questionnaire's collector, supervisors and outlet managers). It rejects empty,
whole-number (1, 1) and shared placeholder positions and GPS fixes rougher than
±300 m, and records who saved it, when and how accurate it was (outlet form,
Location tab). Each questionnaire also says whether its outlet's position is
known, so the app can stop blocking collectors on distance when it is not
(the likely cause of NBS's "GPS not working"). The app side is described in the
mobile repo (`HCPI_SERVER_UPDATES_NOV_2026.md`).

---

## 4. Mobile app

**Download for offline work.** A download button on the Questionnaires screen
(and pull-to-refresh) fetches, in a few requests: the questionnaires assigned
to the collector, **all their items**, the prices already saved, the units and
the collection codes. Before, a questionnaire's items were only stored once it
had been opened while online, so questionnaires not opened beforehand were
empty in the field. Prices entered on the phone and not uploaded yet are
**never overwritten**.

A line under the search box says whether the phone is ready:
*"Ready offline · 12 questionnaires · 340 items · today 08:15"*, or an orange
reminder when nothing was downloaded or the last download is more than a day
old.

**Price form.**

- Order: **price first**, then quantity and unit, then the collection code.
- Mistakes show **while typing** (required, a number, above zero).
- Only **T** (temporarily missing) and **P** (permanently missing) mean "no
  price": choosing them clears the price and says it will be imputed. Other
  codes (season, rise or drop confirmed) **still need a price**. Before, choosing
  any code skipped the price check.
- A live line shows the price for the standard quantity (e.g. "= 1,250 for the
  standard 1 kg"), or notes that the unit differs from the standard one.
- When a price is outside the expected range, a remark is suggested.

The mobile changes are on the local branch `feature/offline-download-form`
(built on `feature/range-check-twice-collection`) and need a new APK build.
