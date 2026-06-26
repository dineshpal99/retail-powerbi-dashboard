# Power Query Transformation Steps

## Retail Power BI Dashboard

All transformations are done inside Power BI Desktop's Power Query Editor.
Load the original CSV **once** — then duplicate it to create each dimension table.

---

## How to Open Power Query

1. Power BI Desktop → **Get Data** → **Text/CSV** → select `Retail_clean.csv`
2. Preview appears → click **Transform Data** (NOT Load)
3. Power Query Editor opens with one query named after your file
4. Rename it: right-click → **Rename** → type `Source_Retail`

---

## Step 1 — Set correct data types on Source_Retail

Set types explicitly — never trust auto-detect:

| Column | Set Type To |
|--------|------------|
| Order_Date | Date |
| Ship_Date | Date |
| Sales | Decimal Number |
| Profit | Decimal Number |
| Discount | Decimal Number |
| Quantity | Whole Number |
| Row_ID | Whole Number |
| All text columns | Text (usually auto-detected correctly) |

**How to set:** click the type icon on the left of each column header → select the correct type.

---

## Step 2 — Disable load on Source_Retail

Right-click **Source_Retail** in the left Queries panel → untick **Enable Load**

The query name turns grey/italic. This means it acts as a base reference for all other queries but does NOT load into your Power BI model as a visible table.

---

## Step 3 — Create Dim_Customer

1. Right-click **Source_Retail** → **Duplicate**
2. Rename the new query to **Dim_Customer**
3. Hold **Ctrl** → click **Customer_ID** and **Segment** column headers
4. Right-click → **Remove Other Columns** (only these 2 remain)
5. Home ribbon → **Remove Rows** → **Remove Duplicates**
6. Double-click **Customer_ID** header → rename to **CustomerID** (no space)

**Expected result:** 793 rows · 2 columns (CustomerID, Segment)

---

## Step 4 — Create Dim_Product

1. Right-click **Source_Retail** → **Duplicate** → rename to **Dim_Product**
2. Hold Ctrl → click: **Product_ID**, **Product_Name**, **Category**, **Sub_Category**
3. Right-click → **Remove Other Columns**
4. Click **Product_ID** column header only (single column, not Ctrl)
5. Home → **Remove Rows** → **Remove Duplicates**

   > ⚠️ Important: select **only Product_ID** before removing duplicates.
   > If you select all 4 columns, duplicate product names with the same ID
   > will both survive and cause a relationship error later.

6. Rename headers: **Product_ID** → **ProductID** · **Product_Name** → **ProductName** · **Sub_Category** → **SubCategory**

**Expected result:** 1,862 rows · 4 columns (ProductID, ProductName, Category, SubCategory)

---

## Step 5 — Create Dim_Geography

1. Right-click **Source_Retail** → **Duplicate** → rename to **Dim_Geography**
2. Hold Ctrl → click: **City**, **State**, **Region**, **Country**
3. Right-click → **Remove Other Columns**
4. Home → **Remove Rows** → **Remove Duplicates**
5. Add Column ribbon → **Custom Column**:
   - Name: **GeoKey**
   - Formula: `[City] & "|" & [State]`
   - Click OK

**Expected result:** 604 rows · 5 columns (City, State, Region, Country, GeoKey)

> The GeoKey column combines City and State as a unique location identifier
> since this dataset has no Postal Code column.

---

## Step 6 — Create Dim_Date (from blank query — no CSV needed)

1. Home ribbon → **New Source** → **Blank Query**
2. Rename to **Dim_Date**
3. Home ribbon → **Advanced Editor**
4. Delete all existing code and paste this complete M code:

```m
let
    StartDate = #date(2014, 1, 1),
    EndDate   = #date(2017, 12, 31),
    NumDays   = Duration.Days(EndDate - StartDate) + 1,

    DateList  = List.Dates(StartDate, NumDays, #duration(1, 0, 0, 0)),
    DateTable = Table.FromList(DateList, Splitter.SplitByNothing(), {"Date"}),

    TypedDate = Table.TransformColumnTypes(DateTable, {{"Date", type date}}),

    AddYear = Table.AddColumn(TypedDate, "Year",
        each Date.Year([Date]), Int64.Type),

    AddQuarter = Table.AddColumn(AddYear, "Quarter",
        each "Q" & Text.From(Date.QuarterOfYear([Date])), type text),

    AddYearQtr = Table.AddColumn(AddQuarter, "YearQuarter",
        each Text.From(Date.Year([Date])) & "-Q"
           & Text.From(Date.QuarterOfYear([Date])), type text),

    AddMonthNum = Table.AddColumn(AddYearQtr, "MonthNumber",
        each Date.Month([Date]), Int64.Type),

    AddMonthName = Table.AddColumn(AddMonthNum, "MonthName",
        each Date.MonthName([Date]), type text),

    AddMonthShort = Table.AddColumn(AddMonthName, "MonthShort",
        each Text.Start(Date.MonthName([Date]), 3), type text),

    AddYearMonth = Table.AddColumn(AddMonthShort, "YearMonth",
        each Text.From(Date.Year([Date])) & "-"
           & Text.Start(Date.MonthName([Date]), 3), type text),

    AddYearMonthSort = Table.AddColumn(AddYearMonth, "YearMonthSort",
        each Date.Year([Date]) * 100 + Date.Month([Date]), Int64.Type),

    AddDayName = Table.AddColumn(AddYearMonthSort, "DayName",
        each Date.DayOfWeekName([Date]), type text),

    AddIsWeekend = Table.AddColumn(AddDayName, "IsWeekend",
        each Date.DayOfWeek([Date], Day.Monday) >= 5, type logical)

in
    AddIsWeekend
```

5. Click **Done**

**Expected result:** 1,461 rows · 11 columns

> The `YearMonthSort` column (e.g., 201401 for Jan 2014) is used to sort the
> `YearMonth` text column chronologically in visuals. Without this, YearMonth
> sorts alphabetically which breaks your trend charts.

---

## Step 7 — Create Fact_Retail

1. Right-click **Source_Retail** → **Duplicate** → rename to **Fact_Retail**
2. Hold Ctrl and select these columns to **REMOVE**:
   - Segment (lives in Dim_Customer)
   - Product_Name, Category, Sub_Category (live in Dim_Product)
   - Country, Region (live in Dim_Geography)
   - Row_ID (no analytical value)
3. Right-click any selected column → **Remove Columns**
4. Add **DiscountPct** column: Add Column → Custom Column → Name: `DiscountPct`, Formula: `[Discount] * 100`
5. Add **ShipDays** column: Add Column → Custom Column → Name: `ShipDays`, Formula: `Duration.Days([Ship_Date] - [Order_Date])`
6. Add **GeoKey** column: Add Column → Custom Column → Name: `GeoKey`, Formula: `[City] & "|" & [State]`

   > ⚠️ The GeoKey formula must be **identical** to the one in Dim_Geography.
   > Any difference in spacing or character will break the relationship.

7. Rename columns: **Order_ID** → **OrderID** · **Order_Date** → **OrderDate** · **Ship_Date** → **ShipDate** · **Ship_Mode** → **ShipMode** · **Customer_ID** → **CustomerID** · **Product_ID** → **ProductID**

**Expected result:** 9,994 rows · 15 columns

---

## Step 8 — Apply Sort by Column to YearMonth

After Close & Apply (Step 9), do this in the Report/Data view:

1. Go to **Data view** (table icon on left sidebar)
2. Select **Dim_Date** table
3. Click the **YearMonth** column header
4. **Column tools** ribbon → **Sort by column** → select **YearMonthSort**

This ensures your monthly trend charts display January → December in correct order, not alphabetical order.

---

## Step 9 — Verify and Close & Apply

Before applying, check row counts by clicking each query in the left panel:

| Query | Expected Rows |
|-------|--------------|
| Fact_Retail | 9,994 |
| Dim_Customer | 793 |
| Dim_Product | 1,862 |
| Dim_Geography | 604 |
| Dim_Date | 1,461 |
| Source_Retail | Greyed out (disabled) |

Click **Close & Apply** (top-left of Home ribbon) to load all 5 tables into the model.

---

## Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| Duplicate ProductID relationship error | Remove Duplicates run on all columns instead of ProductID only | Go back to Dim_Product → select only ProductID column → Remove Duplicates again |
| YearMonth chart sorting alphabetically | Sort by Column not set | Data view → Dim_Date → click YearMonth → Column tools → Sort by column → YearMonthSort |
| GeoKey relationship has blanks | GeoKey formula differs between fact and dim | Check both use exactly `[City] & "\|" & [State]` with no extra spaces |
| Source_Retail still visible in Fields | Enable Load not disabled | Right-click Source_Retail in Power Query → untick Enable Load |

---

*Part of the Retail Power BI Dashboard build guide*
