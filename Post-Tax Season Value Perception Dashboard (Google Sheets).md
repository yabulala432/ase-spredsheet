# Post-Tax Season Value Perception Dashboard for ClearPath CFO Services (Google Sheets)

## Overview

This Google Sheets workbook imports the post-tax season survey CSV, cleans and standardizes key fields, calculates a **Value-to-Cost Index** to flag churn-risk outliers, and produces a polished dashboard with report-usefulness comparisons, an industry satisfaction heatmap, and firm-wide scorecards.

## Step 1: Create Sheets

1. Create a new Google Sheets workbook.
2. Import the starter data file:
   - Go to **File → Import → Upload**
   - Upload `clearpath_cfo_survey_data_starter.csv`
   - Import location: **Replace current sheet**
3. Rename the imported sheet tab to exactly: `Survey_Data`
4. If the first header shows odd characters, click A1 and rename it to exactly: `Client Name`
5. Create these additional sheets (tabs) with these exact names:
   - `Control`
   - `Summary`
   - `Dashboard`

## Step 2: Add Columns

### Sheet: `Survey_Data`

Confirm these columns exist in row 1 (A1:G1) from the import:

1. `Client Name`
2. `Industry`
3. `Service Tier`
4. `Monthly Retainer Amount`
5. `Satisfaction Score`
6. `Most Useful Report`
7. `Open-Ended Feedback`

Add these 5 new headers to the right (H1:L1):

8. `Value-to-Cost Index`
9. `Outlier Flag`
10. `Sentiment Category`
11. `Industry Valid`
12. `Service Tier Valid`

### Sheet: `Control`

Create this table starting at A1:

- A1: `Industry`
- B1: `Service Tier`

Fill the Industry list (A2:A11) with exactly:

- `Artisan Glass`
- `Automotive Parts`
- `Ceramics Studio`
- `Craft Brewery`
- `Custom Woodworking`
- `Distillery`
- `Furniture Mfg`
- `Precision Metal Fab`
- `Specialty Chemicals`
- `Textile Mill`

Fill the Service Tier list (B2:B4) with exactly:

- `Core`
- `Growth`
- `Enterprise`

### Sheet: `Dashboard`

Set up these labels (you will paste formulas in the next step):

- A1: `ClearPath CFO Services — Post-Tax Season Value Perception`
- A3: `Average Satisfaction Score`
- A4: `Total Monthly Recurring Revenue (MRR)`
- A5: `Outlier Accounts (High Spend, Low Satisfaction)`
- A8: `Report Usefulness by Service Tier`
- A20: `Satisfaction Heatmap (Avg Score) — Industry x Service Tier`

## Step 3: Paste Formulas

### Sheet: `Survey_Data`

Paste these formulas exactly:

`Survey_Data!H2`
```gs
=ARRAYFORMULA(IF(A2:A="","",IFERROR(ROUND(E2:E/(D2:D/1000),2),"")))
```

`Survey_Data!I2`
```gs
=ARRAYFORMULA(IF(A2:A="","",IF((D2:D>=Summary!$B$2)*(E2:E<=Summary!$B$3),"Outlier","")))
```

`Survey_Data!J2`
```gs
=ARRAYFORMULA(IF(G2:G="","",IF(REGEXMATCH(LOWER(G2:G),"slow"),"Slow",IF(REGEXMATCH(LOWER(G2:G),"complicated"),"Complicated",IF(REGEXMATCH(LOWER(G2:G),"helpful"),"Helpful",IF(REGEXMATCH(LOWER(G2:G),"value"),"Value","Other"))))))
```

`Survey_Data!K2`
```gs
=ARRAYFORMULA(IF(A2:A="","",IF(COUNTIF(Control!A$2:A,B2:B)>0,"OK","Invalid")))
```

`Survey_Data!L2`
```gs
=ARRAYFORMULA(IF(A2:A="","",IF(COUNTIF(Control!B$2:B,C2:C)>0,"OK","Invalid")))
```

### Sheet: `Summary`

Create the outlier thresholds (these power the Outlier logic and conditional formatting):

`Summary!A1` = `Outlier Thresholds`

`Summary!A2` = `High Spend (75th percentile)`

`Summary!B2`
```gs
=PERCENTILE(FILTER(Survey_Data!D2:D,Survey_Data!A2:A<>""),0.75)
```

`Summary!A3` = `Low Satisfaction (25th percentile)`

`Summary!B3`
```gs
=PERCENTILE(FILTER(Survey_Data!E2:E,Survey_Data!A2:A<>""),0.25)
```

Build the requested summaries:

`Summary!A6`
```gs
=QUERY(Survey_Data!A1:L,"select B, avg(E), sum(D), count(A) where A is not null group by B label avg(E) 'Avg Satisfaction', sum(D) 'MRR', count(A) 'Responses'",1)
```

`Summary!F6`
```gs
=QUERY(Survey_Data!C1:F,"select Col4, count(Col4) where Col4 is not null group by Col4 pivot Col1 label count(Col4) ''",1)
```

`Summary!A20`
```gs
=QUERY(Survey_Data!B1:E,"select Col1, avg(Col4) where Col1 is not null group by Col1 pivot Col2 label avg(Col4) ''",1)
```

Create the “most requested report type by Service Tier” table:

`Summary!H1` = `Requests by Tier`

`Summary!H2`
```gs
=QUERY(Survey_Data!C2:F,"select Col1, Col4, count(Col4) where Col1 is not null and Col4 is not null group by Col1, Col4 label count(Col4) 'Requests'",0)
```

`Summary!K1` = `Requests Sorted`

`Summary!K2`
```gs
=SORT(H2:J,1,TRUE,3,FALSE)
```

`Summary!N1` = `Top Report by Tier`

`Summary!N2`
```gs
=UNIQUE(FILTER(Survey_Data!C2:C,Survey_Data!C2:C<>""))
```

`Summary!O2`
```gs
=ARRAYFORMULA(IF(N2:N="","",XLOOKUP(N2:N,K2:K,L2:L,"")))
```

`Summary!P2`
```gs
=ARRAYFORMULA(IF(N2:N="","",XLOOKUP(N2:N,K2:K,M2:M,"")))
```

### Sheet: `Dashboard`

Paste these formulas exactly:

`Dashboard!B3`
```gs
=ROUND(AVERAGE(FILTER(Survey_Data!E2:E,Survey_Data!A2:A<>"")),2)
```

`Dashboard!B4`
```gs
=SUM(FILTER(Survey_Data!D2:D,Survey_Data!A2:A<>""))
```

`Dashboard!B5`
```gs
=COUNTIF(Survey_Data!I2:I,"Outlier")
```

## Step 4: Configure Logic

### 4.1 Data Validation (Standardize Categories)

1. Select `Survey_Data!B2:B` (Industry).
2. Go to **Data → Data validation**.
3. Criteria: **Dropdown (from a range)**.
4. Range: `Control!A2:A11`
5. Turn on **Reject input**.
6. Click **Done**.
7. Select `Survey_Data!C2:C` (Service Tier).
8. Repeat the steps above with range: `Control!B2:B4`

### 4.2 Protect the `Control` Sheet

1. Right-click the `Control` sheet tab → **Protect sheet**.
2. Set permissions so only editors you trust can modify it.

### 4.3 Conditional Formatting (Outliers + Data Quality)

**Outliers (high spend, low satisfaction)**

1. Select `Survey_Data!A2:L`.
2. Go to **Format → Conditional formatting**.
3. Format rules: **Custom formula is**.
4. Use this formula:
```gs
=($I2="Outlier")
```
5. Set fill color `#B00020`, text color `#FFFFFF`, and bold text.
6. Click **Done**.

**Invalid Industry / Tier**

1. Select `Survey_Data!K2:K` and add a rule with:
```gs
=($K2="Invalid")
```
2. Select `Survey_Data!L2:L` and add a rule with:
```gs
=($L2="Invalid")
```
3. Set fill color `#FFF3CD` and text color `#7A4B00`.

### 4.4 Dashboard Visualizations

**Bar chart: Report Usefulness by Service Tier**

1. Go to the `Summary` sheet and select the pivoted table starting at `Summary!F6` (including headers).
2. Go to **Insert → Chart**.
3. Chart type: **Bar chart**.
4. Use the series for each Service Tier so the chart compares tiers side-by-side per report.
5. Set series colors:
   - `Core`: `#1F3A5F`
   - `Growth`: `#2A9D8F`
   - `Enterprise`: `#E9C46A`
6. Move the chart to the `Dashboard` sheet under the `Report Usefulness by Service Tier` section.

**Heatmap: Satisfaction by Industry**

1. In `Dashboard!A22`, paste this formula:
```gs
=QUERY(Survey_Data!B1:E,"select Col1, avg(Col4) where Col1 is not null group by Col1 pivot Col2 label avg(Col4) ''",1)
```
2. Select the numeric grid (exclude the header row and the Industry label column).
3. Go to **Format → Conditional formatting → Color scale**.
4. Set:
   - Min (low): `#D73027`
   - Mid: `#FFFFBF`
   - Max (high): `#1A9850`

**Scorecards (Typography + Consistent Style)**

1. On `Dashboard`, format `B3:B5` as large bold numbers.
2. Format `B4` as currency.
3. Use font `Arial` across the workbook and keep chart colors consistent.

## Step 5: Test the Sheet

1. Confirm calculations populate down the sheet:
   - `Survey_Data!H:H` should show higher values when satisfaction is high and/or retainer is low.
2. Validate outlier detection:
   - Identify a few rows where `Monthly Retainer Amount` is high and `Satisfaction Score` is low and confirm `Outlier Flag` shows `Outlier`.
3. Validate dropdown enforcement:
   - Try typing an Industry that is not in `Control!A2:A11` and confirm input is rejected.
4. Validate sentiment tagging:
   - Confirm feedback containing “slow”, “complicated”, “helpful”, or “value” is categorized correctly in `Sentiment Category`.
5. Confirm dashboard updates:
   - Update one Satisfaction Score in `Survey_Data` and confirm `Dashboard!B3` and the heatmap values change accordingly.

## Common Mistakes to Avoid

- Typing new Industry or Service Tier values that do not match the `Control` lists (these will be rejected and/or flagged as `Invalid`).
- Leaving `Monthly Retainer Amount` as text with hidden symbols (it must be numeric for percentiles and the Value-to-Cost Index).
- Applying the outlier conditional format to the header row (start at row 2).
- Building charts off filtered ranges that exclude categories (always use the full summary tables on `Summary`).

## Final Notes

- Export `.xlsx`: **File → Download → .xlsx**.
- Export multi-page PDF (Dashboard + Summary): **File → Download → PDF**, choose **Selected sheets**, select `Dashboard` and `Summary`, then export.
- Share the live file: click **Share**, set access to **Editor** for the intended recipients, and copy the link.
