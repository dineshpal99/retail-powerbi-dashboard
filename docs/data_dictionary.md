# Data Dictionary — Superstore Power BI Dashboard

## Source: Sample Superstore (Kaggle)

**Period:** January 2014 – December 2017  
**Rows:** 9,994  
**Columns:** 21  
**Geography:** United States only  
**Granularity:** One row per product line within an order

---

## Fact_Superstore (9,994 rows)

| Column | Type | Description |
|--------|------|-------------|
| RowID | INT | Sequential row number |
| OrderID | VARCHAR | Unique order identifier |
| OrderDate | DATE | Date order was placed |
| ShipDate | DATE | Date order was shipped |
| ShipMode | VARCHAR | First Class / Second Class / Same Day / Standard Class |
| CustomerID | VARCHAR | Foreign key → Dim_Customer |
| ProductID | VARCHAR | Foreign key → Dim_Product |
| City | VARCHAR | Customer city |
| State | VARCHAR | Customer state |
| GeoKey | VARCHAR | City\|State — Foreign key → Dim_Geography |
| Sales | DECIMAL | Revenue in USD (line level) |
| Quantity | INT | Units ordered |
| Discount | DECIMAL | Discount fraction (0 = no discount, 0.2 = 20%) |
| Profit | DECIMAL | Profit in USD — can be negative |
| DiscountPct | DECIMAL | Discount × 100 (for readability) |
| ShipDays | INT | Days between OrderDate and ShipDate |

---

## Dim_Customer (793 rows)

| Column | Type | Description |
|--------|------|-------------|
| CustomerID | VARCHAR | Primary key |
| Segment | VARCHAR | Consumer / Corporate / Home Office |

---

## Dim_Product (1,862 rows)

| Column | Type | Description |
|--------|------|-------------|
| ProductID | VARCHAR | Primary key |
| ProductName | VARCHAR | Full product description |
| Category | VARCHAR | Technology / Furniture / Office Supplies |
| SubCategory | VARCHAR | 17 sub-categories |

---

## Dim_Geography (604 rows)

| Column | Type | Description |
|--------|------|-------------|
| GeoKey | VARCHAR | Primary key (City\|State) |
| City | VARCHAR | City name |
| State | VARCHAR | State name |
| Region | VARCHAR | West / East / Central / South |
| Country | VARCHAR | United States |

---

## Dim_Date (1,461 rows)

| Column | Type | Description |
|--------|------|-------------|
| Date | DATE | Primary key — one row per day (2014-01-01 to 2017-12-31) |
| Year | INT | Calendar year |
| Quarter | VARCHAR | Q1 / Q2 / Q3 / Q4 |
| YearQuarter | VARCHAR | e.g., 2017-Q4 |
| MonthNumber | INT | 1–12 |
| MonthName | VARCHAR | January – December |
| MonthShort | VARCHAR | Jan – Dec |
| YearMonth | VARCHAR | e.g., 2017-Nov (used as X-axis on trend charts) |
| DayName | VARCHAR | Monday – Sunday |
| IsWeekend | BOOLEAN | TRUE if Saturday or Sunday |

---

## Business Rules

| Rule | Definition |
|------|------------|
| Valid sale | quantity > 0 AND discount <= 1 |
| Loss-making order | profit < 0 |
| Loss-making customer | SUM(profit) < 0 across all orders |
| Profit Margin % | profit / sales × 100 |
| GeoKey | City & "\|" & State — must match exactly in both Fact and Dim_Geography |
| Revenue Full Month | Full calendar month total — used for MoM growth (not MTD) |

---

## Calculated Columns in Fact_Superstore

| Column | Formula (DAX) | Purpose |
|--------|---------------|---------|
| DiscountBand | SWITCH(TRUE(), Discount=0, "0%", ...) | Groups orders into 6 discount buckets |
| DiscountBandSort | SWITCH(TRUE(), Discount=0, 1, ...) | Controls sort order of DiscountBand |


