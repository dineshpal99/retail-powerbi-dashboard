# Data Model Design — Star Schema

## Overview

The raw Superstore CSV is one flat table with 19 columns.
In Power BI, we split this into a **fact table** (numbers and keys)
and **dimension tables** (descriptive attributes) using Power Query.

> **Note:** This dataset has no Postal Code or Customer Name columns.
> Geography uses a `GeoKey` (City|State) as the join key instead.

---

## Why a Star Schema?

| Benefit | Explanation |
|---------|-------------|
| Faster DAX | VertiPaq compresses low-cardinality dimension columns very efficiently |
| Cleaner measures | Filter by dimension, aggregate the fact — no ambiguity |
| Correct filtering | Filters flow from dimensions to fact in one direction |
| Smaller model | Descriptive text columns stored once, not repeated 9,994 times |

---

## Final Model Structure

```
                    ┌──────────────┐
                    │  Dim_Date    │
                    │  1,461 rows  │
                    │  [Date]  (1) │
                    └──────┬───────┘
                           │
                           │ 1:M
                           ▼
┌─────────────────┐   ┌────────────────────────────┐   ┌─────────────────┐
│  Dim_Customer   │   │      Fact_Superstore        │   │  Dim_Product    │
│  793 rows       │◄──│      9,994 rows             │──►│  1,862 rows     │
│  [CustomerID](1)│   │  Sales · Profit · Discount  │   │  [ProductID](1) │
└─────────────────┘   │  Quantity · ShipDays        │   └─────────────────┘
                      │  DiscountPct · GeoKey       │
                      └──────────────┬──────────────┘
                                     │
                                     │ M:1
                                     ▼
                      ┌──────────────────────────┐
                      │      Dim_Geography        │
                      │      604 rows             │
                      │      [GeoKey] (1)         │
                      └──────────────────────────┘
```

**All relationships:** Many-to-One (Fact → Dimension) · Single filter direction

---

## Table Specifications

### Fact_Superstore (9,994 rows · 15 columns)

| Column | Type | Role |
|--------|------|------|
| OrderID | Text | — |
| OrderDate | Date | FK → Dim_Date[Date] |
| ShipDate | Date | — |
| ShipMode | Text | — |
| CustomerID | Text | FK → Dim_Customer[CustomerID] |
| ProductID | Text | FK → Dim_Product[ProductID] |
| City | Text | Used to build GeoKey |
| State | Text | Used to build GeoKey |
| GeoKey | Text | FK → Dim_Geography[GeoKey] |
| Sales | Decimal | Measure |
| Quantity | Whole | Measure |
| Discount | Decimal | Measure |
| Profit | Decimal | Measure |
| DiscountPct | Decimal | Derived (Discount × 100) |
| ShipDays | Whole | Derived (ShipDate − OrderDate) |

### Dim_Customer (793 rows · 2 columns)

| Column | Type | Role |
|--------|------|------|
| CustomerID | Text | Primary key (1 side of relationship) |
| Segment | Text | Consumer / Corporate / Home Office |

### Dim_Product (1,862 rows · 4 columns)

| Column | Type | Role |
|--------|------|------|
| ProductID | Text | Primary key |
| ProductName | Text | Full product description |
| Category | Text | Technology / Furniture / Office Supplies |
| SubCategory | Text | 17 sub-categories |

### Dim_Geography (604 rows · 5 columns)

| Column | Type | Role |
|--------|------|------|
| GeoKey | Text | Primary key (City\|State format) |
| City | Text | — |
| State | Text | — |
| Region | Text | East / West / Central / South |
| Country | Text | United States |

### Dim_Date (1,461 rows · 11 columns)

| Column | Type | Notes |
|--------|------|-------|
| Date | Date | Primary key — 2014-01-01 to 2017-12-31 |
| Year | Whole | 2014–2017 |
| Quarter | Text | Q1–Q4 |
| YearQuarter | Text | e.g. 2017-Q4 |
| MonthNumber | Whole | 1–12 |
| MonthName | Text | January–December |
| MonthShort | Text | Jan–Dec |
| YearMonth | Text | e.g. 2017-Nov (used as visual X-axis) |
| YearMonthSort | Whole | e.g. 201711 — used to sort YearMonth column |
| DayName | Text | Monday–Sunday |
| IsWeekend | Logical | TRUE if Saturday or Sunday |

---

## Step-by-Step: Build in Power Query

### Step 1 — Load and rename source

1. **Get Data** → Text/CSV → select `Superstore_clean.csv`
2. Click **Transform Data** (NOT Load)
3. Right-click the query in the left panel → **Rename** → type `Source_Superstore`
4. Set correct data types (Order_Date and Ship_Date as **Date**, Sales/Profit/Discount as **Decimal Number**)
5. Right-click `Source_Superstore` → untick **Enable Load** (turns grey — it's a reference only)

### Step 2 — Create Dim_Customer

1. Right-click `Source_Superstore` → **Duplicate** → rename to `Dim_Customer`
2. Hold Ctrl → select **Customer_ID** and **Segment** only
3. Right-click → **Remove Other Columns**
4. Home → **Remove Rows** → **Remove Duplicates**
5. Rename **Customer_ID** → **CustomerID**

✅ Expected: **793 rows · 2 columns**

### Step 3 — Create Dim_Product

1. Right-click `Source_Superstore` → **Duplicate** → rename to `Dim_Product`
2. Hold Ctrl → select **Product_ID**, **Product_Name**, **Category**, **Sub_Category**
3. Right-click → **Remove Other Columns**
4. Click **Product_ID column only** (single column) → Remove Duplicates

   > ⚠️ Select Product_ID column alone before removing duplicates.
   > Selecting all 4 columns means duplicates are only removed when ALL
   > 4 values match — products with slightly different names but the same
   > Product_ID will survive and break the relationship.

5. Rename: **Product_ID** → **ProductID** · **Product_Name** → **ProductName** · **Sub_Category** → **SubCategory**

✅ Expected: **1,862 rows · 4 columns**

### Step 4 — Create Dim_Geography

1. Right-click `Source_Superstore` → **Duplicate** → rename to `Dim_Geography`
2. Hold Ctrl → select **City**, **State**, **Region**, **Country**
3. Right-click → **Remove Other Columns**
4. Home → **Remove Rows** → **Remove Duplicates**
5. Add Column → **Custom Column** → Name: `GeoKey` → Formula:
   ```
   [City] & "|" & [State]
   ```

✅ Expected: **604 rows · 5 columns**

### Step 5 — Create Dim_Date (blank query — M code)

1. Home → **New Source** → **Blank Query** → rename to `Dim_Date`
2. Home → **Advanced Editor** → delete everything → paste:

```m
let
    StartDate = #date(2014, 1, 1),
    EndDate   = #date(2017, 12, 31),
    NumDays   = Duration.Days(EndDate - StartDate) + 1,
    DateList  = List.Dates(StartDate, NumDays, #duration(1, 0, 0, 0)),
    DateTable = Table.FromList(DateList, Splitter.SplitByNothing(), {"Date"}),
    TypedDate = Table.TransformColumnTypes(DateTable, {{"Date", type date}}),
    AddYear        = Table.AddColumn(TypedDate,   "Year",          each Date.Year([Date]),                                      Int64.Type),
    AddQuarter     = Table.AddColumn(AddYear,     "Quarter",       each "Q" & Text.From(Date.QuarterOfYear([Date])),            type text),
    AddYearQtr     = Table.AddColumn(AddQuarter,  "YearQuarter",   each Text.From(Date.Year([Date])) & "-Q" & Text.From(Date.QuarterOfYear([Date])), type text),
    AddMonthNum    = Table.AddColumn(AddYearQtr,  "MonthNumber",   each Date.Month([Date]),                                     Int64.Type),
    AddMonthName   = Table.AddColumn(AddMonthNum, "MonthName",     each Date.MonthName([Date]),                                 type text),
    AddMonthShort  = Table.AddColumn(AddMonthName,"MonthShort",    each Text.Start(Date.MonthName([Date]), 3),                  type text),
    AddYearMonth   = Table.AddColumn(AddMonthShort,"YearMonth",    each Text.From(Date.Year([Date])) & "-" & Text.Start(Date.MonthName([Date]), 3), type text),
    AddYMSort      = Table.AddColumn(AddYearMonth,"YearMonthSort", each Date.Year([Date]) * 100 + Date.Month([Date]),           Int64.Type),
    AddDayName     = Table.AddColumn(AddYMSort,   "DayName",       each Date.DayOfWeekName([Date]),                             type text),
    AddIsWeekend   = Table.AddColumn(AddDayName,  "IsWeekend",     each Date.DayOfWeek([Date], Day.Monday) >= 5,               type logical)
in
    AddIsWeekend
```

3. Click **Done**

✅ Expected: **1,461 rows · 11 columns**

### Step 6 — Create Fact_Superstore

1. Right-click `Source_Superstore` → **Duplicate** → rename to `Fact_Superstore`
2. Hold Ctrl and select these to **REMOVE** → right-click → Remove Columns:
   - Segment · Product_Name · Category · Sub_Category · Country · Region · Row_ID
3. Add **DiscountPct**: Add Column → Custom Column → `[Discount] * 100`
4. Add **ShipDays**: Add Column → Custom Column → `Duration.Days([Ship_Date] - [Order_Date])`
5. Add **GeoKey**: Add Column → Custom Column → `[City] & "|" & [State]`
6. Rename columns to remove spaces and underscores:
   - Order_ID → OrderID · Order_Date → OrderDate · Ship_Date → ShipDate
   - Ship_Mode → ShipMode · Customer_ID → CustomerID · Product_ID → ProductID

✅ Expected: **9,994 rows · 15 columns**

### Step 7 — Close & Apply

Verify row counts in the bottom bar for each query, then click **Close & Apply**.

---

## Step-by-Step: Create Relationships in Model View

Click the **Model view** icon (3 connected boxes) on the left sidebar.

Drag to create each relationship — always drag FROM the dimension (1 side) TO the fact (many side):

| Relationship | From (1 side) | To (Many side) |
|---|---|---|
| Date | Dim_Date[Date] | Fact_Superstore[OrderDate] |
| Customer | Dim_Customer[CustomerID] | Fact_Superstore[CustomerID] |
| Product | Dim_Product[ProductID] | Fact_Superstore[ProductID] |
| Geography | Dim_Geography[GeoKey] | Fact_Superstore[GeoKey] |

**For each relationship, double-click to verify:**
- Cardinality: **One to Many (1:*)**
- Cross filter direction: **Single**
- Active: **Yes**

---

## Step-by-Step: Mark Date Table + Sort YearMonth

### Mark as Date Table (required for time intelligence)

1. In Model view, right-click **Dim_Date** → **Mark as date table**
2. Select **Date** column → click OK
3. This enables TOTALYTD, SAMEPERIODLASTYEAR, DATEADD etc.

### Sort YearMonth by YearMonthSort (required for correct chart ordering)

1. Go to **Data view** (table icon, left sidebar)
2. Click **Dim_Date** table
3. Click the **YearMonth** column header
4. **Column tools** ribbon → **Sort by column** → select **YearMonthSort**

Without this, your monthly trend charts sort alphabetically
(2014-Apr before 2014-Jan) instead of chronologically.

---

## Hide Foreign Key Columns from Report View

Hiding keys keeps the Fields pane clean for report building:

1. In Data view, right-click **CustomerID** in Dim_Customer → **Hide in report view**
2. Right-click **ProductID** in Dim_Product → **Hide in report view**
3. Right-click **GeoKey** in Dim_Geography → **Hide in report view**
4. Right-click **CustomerID, ProductID, GeoKey** in Fact_Superstore → **Hide in report view**

---

## Final Verification Checklist

- [ ] 5 tables loaded (Fact_Superstore + 4 Dims)
- [ ] Source_Superstore shows as grey/italic (disabled)
- [ ] All 4 relationships created — showing 1:M in Model view
- [ ] No relationship warnings (yellow triangles)
- [ ] Dim_Date marked as Date Table
- [ ] All filter directions are Single (not Both)
- [ ] YearMonth sorted by YearMonthSort in Dim_Date
- [ ] Foreign key columns hidden in all tables

---

## Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| "Duplicate values" on ProductID relationship | Remove Duplicates ran on all columns, not just ProductID | In Dim_Product Power Query: select ProductID column only → Remove Duplicates |
| Time intelligence returns blank/wrong values | Dim_Date not marked as Date Table | Right-click Dim_Date → Mark as date table → select Date column |
| Map visual shows no data | State names not matching Power BI geography | Check Dim_Geography[State] — must be full state names e.g. "California" not "CA" |
| Monthly chart sorts alphabetically | YearMonth not sorted by YearMonthSort | Data view → Dim_Date → click YearMonth → Column tools → Sort by column |
| GeoKey relationship has missing values | GeoKey formula differs between tables | Both must use exactly `[City] & "\|" & [State]` |

---

*Part of the Superstore Power BI Dashboard build guide*
