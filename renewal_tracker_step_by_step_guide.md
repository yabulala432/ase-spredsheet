# Q4 Benefits Renewal Tracker — Step‑by‑Step Build Guide (Excel + Google Sheets)

This guide shows how to turn `q4_benefits_renewal_tracker_starter.csv` into a “hands‑off” renewal tracker where updating `Current Status` on a tracking tab automatically updates an executive dashboard.

## What you’ll build (tab structure)

- `Tracking` (data + formulas + conditional formatting)
- `Lists` (status list + stage order mapping used by validation/sorting)
- `Dashboard` (KPIs + charts + carrier breakdown, printable to PDF)

## 0) Prep (recommended standards)

Use these exact renewal stages (spelling/case matters):

1. `Discovery`
2. `Census Received`
3. `Underwriting`
4. `Proposal Presented`
5. `Bound`

## 1) Import the starter CSV into Excel (to create the `.xlsx`)

1. Open Excel → **Data** → **From Text/CSV**.
2. Select `q4_benefits_renewal_tracker_starter.csv`.
3. Confirm:
   - **File origin/encoding**: UTF‑8 (prevents odd characters in headers)
   - **Delimiter**: Comma
4. Click **Load**.
5. Rename the sheet to `Tracking`.
6. Fix the first header if needed:
   - If you see `﻿Account ID` (looks like “Account ID” but acts weird), rename it to exactly `Account ID`.

## 2) Convert the data range into an Excel Table

1. Click any cell in the dataset.
2. **Insert** → **Table** → check “My table has headers” → OK.
3. With the table selected: **Table Design** → set **Table Name** to `Renewals`.

## 3) Create the `Lists` tab (validation + stage ordering)

1. Add a new sheet named `Lists`.
2. In `Lists!A1:B1`, enter headers: `Status` | `Stage Order`.
3. In `Lists!A2:B6`, enter:
   - `Discovery` | `1`
   - `Census Received` | `2`
   - `Underwriting` | `3`
   - `Proposal Presented` | `4`
   - `Bound` | `5`
4. Convert this into a table (same steps as above) and name it `StageMap`.

## 4) Add calculated columns on `Tracking` (inside the `Renewals` table)

On the `Tracking` tab, add these new columns to the right of `Current Status` (type the header names exactly; Excel will auto-fill formulas down the table).

### A) `Stage Order`

In the first data row of `Stage Order`, enter:

```excel
=XLOOKUP([@[Current Status]],StageMap[Status],StageMap[Stage Order],"")
```

If your Excel doesn’t have `XLOOKUP`, use:

```excel
=IFERROR(VLOOKUP([@[Current Status]],StageMap[[Status]:[Stage Order]],2,FALSE),"")
```

### B) `Days Until Expiry`

```excel
=[@[Renewal Date]]-TODAY()
```

Format this column as Number (0 decimals).

### C) `At Risk`

Flags accounts where the renewal date is within the next 30 days **and** not Bound. This avoids falsely flagging already-expired items.

```excel
=IF(AND([@[Days Until Expiry]]>=0,[@[Days Until Expiry]]<30,[@[Current Status]]<>"Bound"),"At Risk","")
```

Note: If your starter file’s renewal dates are in the past relative to `TODAY()`, `Days Until Expiry` will be negative and `At Risk` will (correctly) stay blank.

### D) (Optional but useful) `Premium Under Review`

```excel
=IF([@[Current Status]]<>"Bound",[@[Annual Premium]],0)
```

## 5) Data validation for `Current Status` (consistency across managers)

1. On `Tracking`, select the entire `Renewals[Current Status]` column.
2. **Data** → **Data Validation** → Allow: **List**.
3. Source:
   - Click into Source → go to `Lists` → select `StageMap[Status]` cells.
4. Turn on:
   - “In-cell dropdown”
   - Optional: an Input Message like “Select a standard renewal stage.”

## 6) Conditional formatting (makes stand-ups fast)

### A) Highlight `At Risk`

Option 1 (fast + reliable): highlight only the `At Risk` cells.

1. Select the `Renewals[At Risk]` column.
2. **Home** → **Conditional Formatting** → **Highlight Cells Rules** → **Equal To…**
3. Type `At Risk` → choose a red fill style → OK.

Option 2 (optional): highlight the entire row when `At Risk`.

1. Select the full `Renewals` table range.
2. **Home** → **Conditional Formatting** → **New Rule** → “Use a formula…”
3. In the formula box, click the first data-row cell in the `At Risk` column (same row as the first record), then complete:

```excel
=<that_cell>="At Risk"
```

Notes:
- The key is that the reference points to the `At Risk` cell for the current row (Excel will apply it row-by-row).
- Use a bold fill (e.g., light red fill, dark red text) for quick scanning in stand-ups.

### B) Visual “days to expiry” heat

Apply color scale to `Days Until Expiry` (red near 0, green far out):
**Home** → **Conditional Formatting** → **Color Scales**.

## 7) Build the `Dashboard` tab (KPIs + pivots + charts)

1. Add a new sheet named `Dashboard`.
2. Turn off gridlines (View → uncheck Gridlines).
3. Create three KPI blocks (top row). Use these formulas:

### KPI 1 — Total Premium Volume Under Review

If you added `Premium Under Review`:

```excel
=SUM(Renewals[Premium Under Review])
```

Otherwise:

```excel
=SUMIFS(Renewals[Annual Premium],Renewals[Current Status],"<>Bound")
```

Format as Currency.

### KPI 2 — Percentage of Renewals Completed

```excel
=COUNTIFS(Renewals[Current Status],"Bound")/COUNTA(Renewals[Account ID])
```

Format as Percent (0–1 decimals).

### KPI 3 — Average Days Until Expiry

Recommended (open renewals only; ignores already-expired rows):

```excel
=AVERAGEIFS(Renewals[Days Until Expiry],Renewals[Current Status],"<>Bound",Renewals[Days Until Expiry],">=0")
```

Format as Number (0 decimals).

## 8) Carrier-specific breakdown (who is handling the most quote volume)

### A) Create a PivotTable

1. Click anywhere in `Renewals`.
2. **Insert** → **PivotTable** → place it on `Dashboard` (or a `Pivots` sheet if you prefer).
3. Build the pivot:
   - **Rows**: `Carrier`
   - **Values**:
     - Sum of `Premium Under Review` (or Sum of `Annual Premium` with a Status filter)
     - Count of `Account ID`
   - **Filters** (optional): `Policy Type`, `Assigned Lead`
4. Sort the Sum (descending) so top carriers appear first.

### B) Add a PivotChart

1. Click the pivot → **PivotTable Analyze** → **PivotChart**.
2. Choose a bar chart for “Premium Under Review by Carrier”.

## 9) Renewal-stage pipeline (progress across stages)

Create another pivot:

- **Rows**: `Current Status`
- **Values**:
  - Count of `Account ID`
  - Sum of `Annual Premium` (or `Premium Under Review`)

Add a clustered column or stacked bar chart. Use `Stage Order` to sort stages in the correct sequence:

1. Add `Stage Order` to the pivot (as a second row field or as a sort helper).
2. Sort ascending by `Stage Order`.

## 10) Make it “hands-off” for weekly meetings

### A) Make pivots refresh automatically (Excel)

For each PivotTable:
- **PivotTable Analyze** → **Options** → **Data** tab → check **Refresh data when opening the file**.

### B) Suggested weekly workflow (Project Manager)

1. Open the workbook.
2. Update `Tracking` → `Current Status` only (dropdown).
3. Scan `At Risk` and `Days Until Expiry`.
4. Go to `Dashboard` (KPIs + charts update; pivots refresh on open if enabled).
5. Export Dashboard to PDF for the meeting.

## 11) Export deliverables

### A) Export `.xlsx`

- Save the workbook as `Q4_Benefits_Renewal_Tracker.xlsx`.

### B) Export Dashboard to PDF (clean executive format)

1. On `Dashboard`, set a print area:
   - Select the dashboard layout range → **Page Layout** → **Print Area** → **Set Print Area**
2. **File** → **Export** → **Create PDF/XPS**
3. In options:
   - Fit sheet on one page (or 1 page wide, 1 page tall)
   - Landscape orientation (usually best for dashboards)

### C) Publish to Google Sheets (shareable editor link)

1. Upload the `.xlsx` to Google Drive.
2. Right click → **Open with** → **Google Sheets** (converts it).
3. In Google Sheets:
   - Check that `Current Status` validation still exists (recreate if needed)
   - Verify formulas in calculated columns (Sheets supports `XLOOKUP` but sometimes conversion changes references)
4. Click **Share** → set access to **Editor** → copy link.

## 12) Google Sheets formula equivalents (if you build directly in Sheets)

Assuming headers are in row 1 and data starts row 2:

- `Days Until Expiry` (in e.g. `I2`):

```gs
=D2-TODAY()
```

- `At Risk` (in e.g. `J2`):

```gs
=IF(AND(I2>=0,I2<30,H2<>"Bound"),"At Risk","")
```

- Total Premium Under Review:

```gs
=SUMIF(H:H,"<>Bound",E:E)
```

- % Completed:

```gs
=COUNTIF(H:H,"Bound")/(COUNTA(A:A)-1)
```

- Avg Days Until Expiry (open only, non-expired):

```gs
=AVERAGE(FILTER(I:I,(I:I>=0)*(H:H<>"Bound")))
```

---

If you want, I can also generate a ready-to-use `Q4_Benefits_Renewal_Tracker.xlsx` in this folder with the extra calculated columns, validation list, and a prebuilt Dashboard layout (you’d only need to upload/share it).
