# Q4 Corporate Benefits Renewal Tracker & Executive Dashboard (Google Sheets)

## Overview

This Google Sheets workbook tracks corporate benefits renewals through a standard set of stages and powers an executive dashboard that updates instantly when account managers update a status. It highlights bottlenecks in **Quoting** and **Presentation**, summarizes key KPIs, and flags **At Risk** accounts (renewal in < 30 days and not **Bound**).

## Step 1: Create Sheets

1. Create a new Google Sheets workbook.
2. Import the starter data file:
   - Go to **File → Import → Upload**
   - Upload `q4_benefits_renewal_tracker_starter.csv`
   - Import location: **Replace current sheet**
3. Rename the imported sheet to exactly: `Renewals`
4. Add two more sheets (tabs) with these exact names:
   - `Lists`
   - `Dashboard`

## Step 2: Add Columns

### Sheet: `Renewals`

Confirm these columns exist in row 1 (A1:H1) from the import:

1. `Account ID`
2. `Client Name`
3. `Policy Type`
4. `Renewal Date`
5. `Annual Premium`
6. `Assigned Lead`
7. `Carrier`
8. `Current Status`

Add these 3 new headers to the right (I1:K1):

9. `Days Until Renewal`
10. `At Risk`
11. `Phase`

### Sheet: `Lists`

Create this mapping table starting at A1:

- A1: `Status`
- B1: `Phase`

Fill rows A2:B6 with exactly:

- A2: `Discovery` → B2: `Pre-Quote`
- A3: `Census Received` → B3: `Pre-Quote`
- A4: `Underwriting` → B4: `Quoting`
- A5: `Proposal Presented` → B5: `Presentation`
- A6: `Bound` → B6: `Completed`

### Sheet: `Dashboard`

Set up these labels (you will paste formulas in the next step):

- A1: `Executive Dashboard`
- A3: `Total Premium Under Review`
- A4: `% Renewals Completed`
- A5: `Average Days Until Renewal`
- D3: `Quoting Accounts`
- D4: `Quoting Premium`
- D5: `Presentation Accounts`
- D6: `Presentation Premium`

Add two section headers:

- A7: `At Risk (Next 30 Days, Not Bound)`
- J7: `Carrier Quote Premium (Quoting Phase)`

## Step 3: Paste Formulas

### Sheet: `Renewals`

Paste these formulas exactly:

`Renewals!I2`
```gs
=ARRAYFORMULA(IF(B2:B="","",D2:D-TODAY()))
```

`Renewals!J2`
```gs
=ARRAYFORMULA(IF(B2:B="","",IF((I2:I>=0)*(I2:I<30)*(H2:H<>"Bound"),"At Risk","")))
```

`Renewals!K2`
```gs
=ARRAYFORMULA(IF(B2:B="","",IFERROR(VLOOKUP(H2:H,Lists!A:B,2,FALSE),"")))
```

### Sheet: `Dashboard`

Paste these formulas exactly:

`Dashboard!B3`
```gs
=SUMIFS(Renewals!E2:E,Renewals!B2:B,"<>",Renewals!H2:H,"<>"&"Bound",Renewals!I2:I,">=0")
```

`Dashboard!B4`
```gs
=IFERROR(COUNTIFS(Renewals!B2:B,"<>",Renewals!H2:H,"Bound")/COUNTIFS(Renewals!B2:B,"<>"),0)
```

`Dashboard!B5`
```gs
=IFERROR(AVERAGE(FILTER(Renewals!I2:I,Renewals!B2:B<>"",Renewals!I2:I>=0)),0)
```

`Dashboard!E3`
```gs
=COUNTIFS(Renewals!B2:B,"<>",Renewals!K2:K,"Quoting",Renewals!I2:I,">=0")
```

`Dashboard!E4`
```gs
=SUMIFS(Renewals!E2:E,Renewals!B2:B,"<>",Renewals!K2:K,"Quoting",Renewals!I2:I,">=0")
```

`Dashboard!E5`
```gs
=COUNTIFS(Renewals!B2:B,"<>",Renewals!K2:K,"Presentation",Renewals!I2:I,">=0")
```

`Dashboard!E6`
```gs
=SUMIFS(Renewals!E2:E,Renewals!B2:B,"<>",Renewals!K2:K,"Presentation",Renewals!I2:I,">=0")
```

Create the **At Risk** table headers on row 8:

- A8: `Account ID`
- B8: `Client Name`
- C8: `Renewal Date`
- D8: `Annual Premium`
- E8: `Assigned Lead`
- F8: `Carrier`
- G8: `Current Status`
- H8: `Days Until Renewal`

Then paste this formula in `Dashboard!A9`:

```gs
=SORT(
  FILTER(
    {Renewals!A2:A,Renewals!B2:B,Renewals!D2:D,Renewals!E2:E,Renewals!F2:F,Renewals!G2:G,Renewals!H2:H,Renewals!I2:I},
    Renewals!J2:J="At Risk"
  ),
  8,TRUE
)
```

Create the **Carrier Quote Premium** table headers on row 8:

- J8: `Carrier`
- K8: `Quote Premium`

Then paste this formula in `Dashboard!J9`:

```gs
=QUERY(
  Renewals!A1:K,
  "select G, sum(E) where B is not null and I >= 0 and K = 'Quoting' group by G order by sum(E) desc label G 'Carrier', sum(E) 'Quote Premium'",
  1
)
```

## Step 4: Configure Logic

1. Set **data validation** for consistent statuses:
   - Select `Renewals!H2:H`
   - Go to **Data → Data validation**
   - Criteria: **Dropdown (from a range)** → `Lists!A2:A6`
   - Turn on **Reject input**
2. Format key fields:
   - Format `Renewals!D:D` as **Date**
   - Format `Renewals!E:E` and `Dashboard!B3` as **Currency**
   - Format `Dashboard!B4` as **Percent**
3. Add conditional formatting for risk:
   - Select `Renewals!A2:K`
   - Go to **Format → Conditional formatting**
   - Rule: **Custom formula is**
     ```gs
     =$J2="At Risk"
     ```
   - Set fill color to light red and bold text
4. Protect calculated columns:
   - Protect `Renewals!I:K` so only the owner can edit formulas

## Step 5: Test the Sheet

1. Pick one row in `Renewals` and set the `Renewal Date` to a value within 30 days:
   - Example: set the date cell to:
     ```gs
     =TODAY()+20
     ```
2. Confirm the row shows:
   - `Days Until Renewal` is a positive number
   - `At Risk` shows `At Risk` when `Current Status` is not `Bound`
3. Change `Current Status` to `Bound` using the dropdown:
   - Confirm `At Risk` clears
   - Confirm `Total Premium Under Review` decreases (Dashboard!B3)
   - Confirm `% Renewals Completed` increases (Dashboard!B4)

## Common Mistakes to Avoid

- Typing statuses that are not in `Lists!A2:A6` (fix with dropdown validation).
- Entering `Renewal Date` as text instead of a real date (fix by formatting as Date and re-entering).
- Entering `Annual Premium` with currency symbols as text (fix by entering numbers and formatting as Currency).
- Editing formula columns `Renewals!I:K` instead of only updating `Current Status`.

## Final Notes

- Sharing: use **Share → General access → Anyone with the link → Editor** only when you are ready for collaborators to update statuses.
- Exports: use **File → Download → XLSX (.xlsx)** for the spreadsheet file, and **File → Download → PDF** with **Current sheet = Dashboard** for the weekly executive PDF.
