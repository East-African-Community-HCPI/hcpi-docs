# System Issues
Dashboard issues For all:
1. A drop-down option to select the month and year to show the figures (both index and inflation), Filter by month (Latest)
2. Show National CPI Value at the top, historical trend comes higher, 
3. See how to show all Divisions on the Screen 
4. Fix Issues of No national figures shown on dashboards (uganda-specific bug)

5. See How to setup Dashboard for public access

 
KNBS;
1. Systems shows 100% during data collection but later reduces to 67%
2. Standard and non-standard of weights*
3. Provision of in-built calculator - Explanation of how inbuilt calculations are being done 
4. [data collection] System slow and more so when more users are added and data increases. 
5. Dashboard Issue: Figures from every sector are showing 0
 
UBOS:
1. Dashboard Issue: Figures from every sector are showing 0
 
NBS:
1. GPS not working during data collection. 
2. Clarify if server is offline whether the tablet offline works [Test to see]

OCGS:
Unable to update EA Indices - Resolved

EAC:
1. Review Syncing of Data to Secretariat Dashboard

# Computations:
1. Review of Computations Process 

# Suggestions:
1. Data Collection Analytics: Show counts by state of collection qtnires by outlet, centre, national, etc.
2. Calculation of Provisional Index / with details of items. methodology: copy over prev. month values.
3. Review outliers to ensure we have 3 levels, extreme outlier=24, outlier=12, mild Outlier =6 months
4. Review Inliers to ensure we have it checking 12 previous consecutive months and also if change in the price is the same for the same period.
5. Report of outliers and inliers
6. Monitor and flag changes in Centrally Administrative Prices
7. Document XML/RPC endpoints for system.
8. Transfer Mobile App from Kola to EAC-Controlled Google Play Account

Annual % change
core and non-core to indicator cards


- Item codes should be standardized
- Deleting update history and jobs does not mark as failed. a lock remains preventing restarting an update on the same items
- Collected Data needs to be able to go through all validation steps and make it to observations so that it is used in calculations.
- Add Period and division filters to index update functions in HCPI_Index. Iwe should be able to run the update on a subset of the data.
- Large log tables leading to slow queries and large database size. review what is being logged and see howto balance what is needed and what isn't.
- After 3 months of temporarily missing, the system should indicate how long it has been
- A report on items that have been temporarily missing for 3 months or more.
- Code on replacement of items not sufficient.

Now, some updates:
✅ swap blue and red, and add blue to key for matrix on dashboard. and have the key explain them
operational dashboard should show which consumption segments are so far behind in terms of low coverage of outlet-items or something like that which is relevant. 

We need to review the computations that happen, and compare current process to how it should be. 
1. there should be no zero prices. if price has not been captured for an item, it should have been imputed, or should be imputed. Those imputations need to follow one of the proper imputation methods
2. there should be no 0s or missing values are w ecompute the heigher levels. any item that has a weight should have a value, ensuring that these figures cascade up to the final view
3. If a figure is missing, it needs to be highlighted on operational dashboard for each of the levels.
4. For the provisional CPI, carry-forward method can be used, with operational dashboard highlighting that
There should be no zeros at all i nthe computation at all the levels
all active items should be included in ccalculations, impute where none
all EA indexes should be included, no gaps. if item has weight, it needs to have a figure. highlight any on operational dashboard, calc
impute prices, not later fields 
carry-forward can be used at start of calculations, but is temporary. Proper method should be proper imputation using one of imputation methods. 
Add a link to report for validations on the provisional
a repor tshowing when collections are happening vs days
Link the matrix of events from 

# Data Collection App
- When Questionnaires synced to app, they shouldcome with the questionnaire content, not coming afterwards. so that once sync happens they can work offline.
- reorder questionnaire so that collection code is selected after price is entered, and we clear the price if temporarily missing or permanently missing is selected as the code. 
- Provide a way to filter by current period, provide a way for device to only show questionnaires

- Pulish a version of the APK on the servers so that it can be used to ensure data collectors are always on the right version
- Publish to play store to replace the existing entry 



## 
- Consider serving some server information for admins to be able to see how resources are doing, what is causing load, etc

- Promote to observations view should include item code column

On Outliers/inliers review, Add option to Reject or approve the items, either singular or in bulk. rejected not included in calculation, but appear in a report

In A reports section, maybe on operations dashboard, to have a repot of history of processing/logs/ how cpi wa calculated per segment per period and per user.

After imputation, outliers should still be higlhihted. their state should not go away since it feeds alot.

No un-imputed. if item cannot be imputed from relative, use same item in another segment. if not possible, then use items in another EA. follow standard practice. then the default rate to refuse to impute from object should be linent, maybe as long as like 20 or 30% of objects have prices ocmputed in that group?  

Index running should have filters to run for a given collection period or all periods( the way it does right now)

On dashboard, Why provisional column in CPI by division? we can remove it.

The progress field on Data Validations never changes, we need to update it as status changes.

Add bulk select and group actions to screen of promotion to observations / when uploading items in importation

Review permissions for Validatio nand promotion to Observation to ensure that only allowed people work on them.



ssh ocgs_ubuntu@hcpi.ocgs.go.tz -p 7575



# Meeting with Julie

## Weights & classification
- ✅ **Weight totals:** Check that weights sum to 1000 at every level (EA → sub-class → … → division). Flag any level that doesn't.
- ✅ **Special index sets:** Group special indices into sets (e.g. Core / Non-core). An item can belong to only one index per set. Flag sets that don't total 1000.
- ✅ **`is_standard` toggle:** Warn the user when it's switched on, because it changes how the item is collected and standardised.


## Elementary Aggregate page
- ✅ Rename **Code** to **EA Code** and hide the current EA code column. It shouldn't be summed when grouping.
- ✅ Add the **With / Without weights** filter to the higher-level pages too.

## Collection
- ✅ **Range check at entry:** Flag out-of-range prices as they're entered, in both the web app and the mobile app.
- ✅ **Twice-collected items:** Some non-standard items are priced twice (e.g. 2 weeks). Standardise each price by weight, then use the average. Add a flag for these items and split them into a separate questionnaire (once vs. twice per period).
- ✅ **Fixed basket:** The same product specs are priced for the whole basket period (~5 years).

## Replacements
- ✅ Review the replacement flow for items and outlets. One outlet can be replaced by 1 or 2 outlets. work on a replacement wizard.
- ✅ **Replacement computation:** In month 1, compute the relative with the old product's imputed price and capture the new product's price. In month 2, switch to the new product, which now has 2 prices.

## Validation
- ✅ **Inlier re-flagging:** Once an inlier is marked OK, don't flag it again for 6 months.
- ✅ **Imputed prices report:** Add a report of imputed prices. _Confirm what "resend" meant._
- ✅ validation is also currently vague. It should be cleareer. able to add a note for each or something, revie wand think of the best way to handle eahc of the different kinds of validatio nbesides just relying on seeing the oclours highlgihted and proceeding!! can we see details for any of them? can they make somewhere to mark each by each as okay? can ther ebe a key for the colours somewhere? how can the different types of validatio nbe done more clearly, and in a much better way?  Think of other ways to imporve things. 

## Dashboard & access
- ✅ **Colours:** Don't mix colour meanings. Add a key or legend button that explains each colour.
- ✅ **Centre scoping:** Users see only their centre on data screens. On the dashboard they see everything, filtered to their area by default.
- ✅ How can we improve controls on dashboard? zoom in zoom out? can we add a map? claude memories on here know where maps are

## Exports
- ✅ **Export Excel Workbook** (Indices) already includes the price sheet, so no separate Observation Lines export is needed. Add a **raw data** sheet that has collected data
- ✅ Flag imputed prices in the exported price sheet.

## Testing
- ✅ Add an option to upload raw outlet prices so imputation, validation and the rest of the pipeline can be tested.

## Mobile App
- ✅ sync button that can sync down all assigned quesionnaries s othat ocllection can happen offline
- ✅ Better form inputs and validation on the go.


--- new weights

when selecting items, any computation, it should always show those with wieghts. 
- to simplify addition to special index categories, when adding fresh items, let us have a filter to only eas that have not yet been selected to be part of this 
- Fixed basket for the period. each of the observstions rexplected/ outlet items should never change for the given items, i.e. if tomatoes being collected from 3 outlets, should always calcluate fro mthe 3 outlets.
- When validatin prices, offer option for people to enter the data, so that calucations for standardisation can be done, ensure it is recorded right so that they can work well.  Also, a price changed there should be tracked in audit log and proper so that we know that we can tell a price changed at that level from a price that was captured earlier
-  

- Lines that could not be imputed (no price) will be excluded from the index. Finalize this validation?