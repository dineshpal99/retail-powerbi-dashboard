# Key Findings & DAX Reference

---

## Key Findings (Verified from Source Data)

### Page 1 — Executive Overview

| KPI | Value |
|-----|-------|
| Total Revenue | $2,297,200.86 |
| Total Profit | $286,397.02 |
| Overall Profit Margin | 12.5% |
| Total Orders | 5,009 |
| Unique Customers | 793 |
| MoM Growth (Dec vs Nov 2017) | -29.2% |

### Page 2 — Sales & Profit Trends

- Revenue grew **51.4%** from 2014 ($484K) to 2017 ($733K)
- Profit grew **88.6%** from 2014 ($49.5K) to 2017 ($93.4K)
- Rolling 12-month chart confirms consistent underlying upward trend
- Strong Q4 seasonal peaks visible every year

### Page 3 — Product Performance

- **3 sub-categories are loss-making:** Tables (-$17,725), Bookcases (-$3,473), Supplies (-$1,189)
- **Best performer:** Copiers at 37.2% margin ($55,618 profit)
- **Most surprising:** Binders has the highest average discount (37.2%) of any sub-category

### Page 4 — Regional Analysis

- **Central region** has the weakest margin at 7.9% vs company average 12.5%
- **10 states** have negative total profit — Texas worst at -$25,729 (-15.1% margin)
- **Central/Furniture** is the only negative cell in the category-region matrix (-1.75%)
- California and New York are the strongest states by profit

### Page 5 — Customer Intelligence

- **155 customers (19.5%)** are loss-making across their full purchase history
- **SM-20320** — top customer by revenue ($25K) but loss-making (-$1,980 profit)
- **Home Office** has the highest margin (14.0%) despite being smallest segment
- **Consumer** generates 50.6% of revenue but has the lowest margin (11.5%)

### Page 6 — Discount & Profitability

- Orders at **0% discount**: 29.5% margin — $321K profit
- Orders at **40%+ discount**: -77.4% margin — -$100K profit loss
- **Capping at 20%** would improve profit by $135.4K (+47.3%)
- **Binders (37.2%)** is the most heavily discounted sub-category, not Tables

---

## DAX Reference — All Measures

### Core Sales Measures

```dax
Total Revenue = SUM(Fact_Retail[Sales])

Total Profit = SUM(Fact_Retail[Profit])

Total Orders = DISTINCTCOUNT(Fact_Retail[OrderID])

Total Quantity = SUM(Fact_Retail[Quantity])

Total Customers = DISTINCTCOUNT(Fact_Retail[CustomerID])

Profit Margin % = DIVIDE([Total Profit], [Total Revenue], 0)

Avg Order Value = DIVIDE([Total Revenue], [Total Orders], 0)

Avg Revenue per Customer =
DIVIDE([Total Revenue], [Total Customers], 0)

Avg Profit per Customer =
DIVIDE([Total Profit], [Total Customers], 0)

Avg Orders per Customer =
DIVIDE([Total Orders], [Total Customers], 0)

Loss Making Customers =
CALCULATE(
    [Total Customers],
    FILTER(
        VALUES(Fact_Retail[CustomerID]),
        CALCULATE([Total Profit]) < 0
    )
)
```

### Time Intelligence Measures

```dax
Revenue YTD =
TOTALYTD([Total Revenue], Dim_Date[Date])

Revenue MTD =
TOTALMTD([Total Revenue], Dim_Date[Date])

Revenue Full Month =
VAR LatestDate = MAX(Dim_Date[Date])
VAR LatestYear = YEAR(LatestDate)
VAR LatestMonthNum = MONTH(LatestDate)
RETURN
    CALCULATE(
        SUM(Fact_Retail[Sales]),
        YEAR(Dim_Date[Date]) = LatestYear,
        MONTH(Dim_Date[Date]) = LatestMonthNum
    )

Revenue LY =
CALCULATE([Total Revenue], SAMEPERIODLASTYEAR(Dim_Date[Date]))

Revenue PM =
CALCULATE([Total Revenue], DATEADD(Dim_Date[Date], -1, MONTH))

Revenue Rolling 12M =
CALCULATE(
    [Total Revenue],
    DATESINPERIOD(
        Dim_Date[Date],
        LASTDATE(Dim_Date[Date]),
        -12,
        MONTH
    )
)
```

### Growth Measures

```dax
YoY Growth % =
VAR CY = [Total Revenue]
VAR LY = [Revenue LY]
RETURN
    DIVIDE(CY - LY, LY, BLANK()) * 100

MoM Growth % =
VAR CM = [Revenue Full Month]
VAR PM = CALCULATE([Revenue Full Month], DATEADD(Dim_Date[Date], -1, MONTH))
RETURN
    DIVIDE(CM - PM, PM, BLANK()) * 100

MoM Growth Label =
VAR LatestDate = CALCULATE(MAX(Fact_Retail[OrderDate]), ALL(Fact_Retail))
VAR LatestYear = YEAR(LatestDate)
VAR LatestMonthNum = MONTH(LatestDate)
VAR CurrentRev =
    CALCULATE(
        SUM(Fact_Retail[Sales]),
        YEAR(Dim_Date[Date]) = LatestYear,
        MONTH(Dim_Date[Date]) = LatestMonthNum
    )
VAR PriorDate = EDATE(LatestDate, -1)
VAR PriorRev =
    CALCULATE(
        SUM(Fact_Retail[Sales]),
        YEAR(Dim_Date[Date]) = YEAR(PriorDate),
        MONTH(Dim_Date[Date]) = MONTH(PriorDate)
    )
VAR GrowthPct = DIVIDE(CurrentRev - PriorRev, PriorRev, 0)
VAR ArrowSymbol = IF(GrowthPct >= 0, "▲", "▼")
RETURN
    ArrowSymbol & " " & FORMAT(ABS(GrowthPct), "0.0%")
```

### Ranking Measures

```dax
Product Revenue Rank =
RANKX(
    ALL(Dim_Product[ProductName]),
    [Total Revenue], , DESC, Dense
)

Sub-Category Profit Rank =
RANKX(
    ALL(Dim_Product[SubCategory]),
    [Total Profit], , DESC, Dense
)

Region Profit Rank =
RANKX(
    ALL(Dim_Geography[Region]),
    [Total Profit], , DESC, Dense
)
```

### Dynamic Measures

```dax
Margin Band =
SWITCH(
    TRUE(),
    [Profit Margin %] < 0,   "Loss",
    [Profit Margin %] < 0.05, "Low",
    [Profit Margin %] < 0.15, "Medium",
    "High"
)

Selected Category Title =
"Performance: " &
SELECTEDVALUE(Dim_Product[Category], "All Categories")

Margin vs Company Avg =
VAR ThisMargin = [Profit Margin %]
VAR CompanyMargin = CALCULATE([Profit Margin %], ALL(Dim_Product))
RETURN ThisMargin - CompanyMargin
```

### What-If Measures

```dax
-- Requires What-If parameter named "Discount Cap"
-- (Whole numbers 0-80, increment 5, default 20)

Profit Above Cap =
CALCULATE(
    SUM(Fact_Retail[Profit]),
    Fact_Retail[Discount] <=
        SELECTEDVALUE('Discount Cap'[Discount Cap Value]) / 100
)

Projected Profit Improvement =
[Profit Above Cap] - [Total Profit]
```

---

## Common Issues & Fixes

| Issue | Root Cause | Fix |
|-------|-----------|-----|
| MoM shows blank | Used Revenue_MTD (partial month) instead of full month | Use Revenue Full Month measure |
| TOTALYTD wrong numbers | Date table not marked as Date Table | Right-click Dim_Date → Mark as date table |
| Relationship error on ProductID | Duplicate ProductIDs in Dim_Product | Remove Duplicates on ProductID column only |
| Discount cap slider not responding | Decimal increments render unreliably | Recreate parameter with whole numbers (0-80), divide by 100 in DAX |
| Conditional formatting wrong colors | "Percent" type in rules dialog | Switch to "Number" with decimal thresholds (0.15, 0.25) |
| YearMonth sorting alphabetically | Text column sorted alphabetically | Create YearMonthSort helper column → Sort by Column |

---

*Document maintained by: [Your Name] | Last updated: 2024*
