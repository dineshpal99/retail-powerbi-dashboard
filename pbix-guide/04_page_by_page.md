# Dashboard Build Guide — Page by Page

---

## PRE-BUILD CHECKLIST
- [ ] Star schema model built (see 01_data_model.md)
- [ ] Dim_Date created and marked as Date Table
- [ ] All DAX measures added from dax/all_measures.dax
- [ ] A professional theme applied (View → Themes → browse themes)

---

## PAGE 1: Executive Overview

**Purpose:** First screen every stakeholder sees. Headline KPIs + quick orientation.

### Layout (top to bottom)
```
[Logo/Title area]
[KPI Cards Row: Revenue | Profit | Margin% | Orders | Customers]
[Left: Revenue by Category bar] [Right: Profit by Sub-category bar]
[Bottom: Monthly revenue trend line chart]
[Slicers: Year | Region | Category | Segment]
```

### Visual Specifications

| Visual | Type | Fields | Format |
|--------|------|--------|--------|
| Total Revenue | Card | [Total Revenue] | Currency, 2dp |
| Total Profit | Card | [Total Profit] | Currency, conditional red if <0 |
| Profit Margin | KPI | [Profit Margin %], Target=15% | % |
| Total Orders | Card | [Total Orders] | Number |
| MoM Growth | Card | [MoM Label] | Text measure |
| Revenue by Category | Clustered bar | Category, [Total Revenue] | Sorted desc |
| Profit by Sub-Category | Bar chart | Sub-Category, [Total Profit] | Conditional format |
| Monthly Trend | Line chart | Date[Month], [Total Revenue], [Total Profit] | Dual axis |

### Slicers
- Year (buttons style — 2014, 2015, 2016, 2017)
- Region (dropdown)
- Segment (dropdown)

### Conditional Formatting on Profit Bar
- Rules: Profit < 0 → Red, Profit 0–5K → Amber, > 5K → Green

---

## PAGE 2: Sales & Profit Trends

**Purpose:** Time-series analysis — growth patterns, seasonality, YoY.

### Visuals

| Visual | Type | Fields | Notes |
|--------|------|--------|-------|
| Revenue MoM waterfall | Waterfall chart | Month, [MoM Growth %] | Shows gains/losses |
| YoY comparison | Clustered column | Month Name, [Total Revenue], [Revenue LY] | Side by side |
| Rolling 12M trend | Area chart | Date, [Revenue Rolling 12M] | Smooth trend line |
| YoY growth % by category | Matrix | Category vs Year, [YoY Growth %] | Heatmap colours |
| Revenue YTD vs Target | KPI visual | [Revenue YTD] vs target | — |

### Key DAX measures used
- [Revenue LY], [YoY Growth %], [Revenue Rolling 12M], [MoM Growth %]

### Design tip
Add a text box above the YoY chart:
> "Year-over-Year revenue comparison — bars show current year, 
>  line shows prior year for easy comparison"

---

## PAGE 3: Product Performance

**Purpose:** Which products, categories, and sub-categories are making or losing money?

### Visuals

| Visual | Type | Fields | Notes |
|--------|------|--------|-------|
| Revenue vs Profit bubble | Scatter | X=[Total Revenue], Y=[Total Profit], Size=[Total Quantity], Legend=Category | Key insight visual |
| Sub-category profitability | Horizontal bar | Sub-Category, [Total Profit] | Sorted, RED for negative |
| Top 10 products table | Table | Product Name, [Total Revenue], [Total Profit], [Profit Margin %] | Conditional format margins |
| Category donut | Donut | Category, [Total Revenue] | 3 segments |
| Discount vs Margin scatter | Scatter | X=AVG(Discount), Y=[Profit Margin %] | By sub-category |

### Drillthrough Setup
1. Create a new page "Product Detail"
2. Add Sub-Category to Drillthrough well
3. Add: revenue trend, top customers for that sub-category, profit history
4. Add back button

### Conditional Formatting on Sub-Category Bar
- Data bars + background colour by [Profit Margin %]
- Red: < 0%, Amber: 0-10%, Green: > 10%

---

## PAGE 4: Regional Analysis

**Purpose:** Where is the business performing and where are the problem markets?

### Visuals

| Visual | Type | Fields | Notes |
|--------|------|--------|-------|
| US State Map | Filled Map | State, [Total Profit] | Diverging colours: red=loss, green=profit |
| Region performance | Clustered bar | Region, [Total Revenue], [Total Profit] | Dual measure |
| State table | Table | State, Region, [Total Revenue], [Total Profit], [Profit Margin %] | Top 15 |
| Loss-making states | Slicer/Filter | Filter to Profit < 0 | Highlight problems |
| Region × Category matrix | Matrix | Region (rows), Category (cols), [Profit Margin %] | Heatmap |

### Map Setup
- Visual: Filled Map
- Location: Dim_Geography[State]
- Color saturation: [Total Profit]
- In Format → Data colours → Diverging → Min=Red (negative), Mid=White (zero), Max=Green

### Key Insight to Label
Add annotation or text box:
> "States with negative total profit — investigate discount levels and shipping costs"

---

## PAGE 5: Customer Intelligence

**Purpose:** Who are our customers? Who is most valuable? Who is at risk?

### Visuals

| Visual | Type | Fields | Notes |
|--------|------|--------|-------|
| RFM segment donut | Donut | [RFM Segment], [Total Customers] | 6 segments |
| Revenue by segment | Bar chart | [RFM Segment], [Total Revenue] | Sorted |
| Top 20 customers | Table | Customer Name, [Total Revenue], [Total Profit], [Total Orders] | Ranked |
| Customer segment performance | Matrix | Segment (Consumer/Corp/HO) vs Category | [Profit Margin %] heatmap |
| Revenue per customer trend | Line | Date, [Avg Customer Revenue] | By year |

### RFM Summary Cards
Add 4 small cards:
- Champions: CALCULATE([Total Customers], [RFM Segment]="Champion")
- At Risk: CALCULATE([Total Customers], [RFM Segment]="At Risk")
- Lapsed: CALCULATE([Total Customers], [RFM Segment]="Lapsed")
- New Customers: CALCULATE([Total Customers], [RFM Segment]="New Customer")

---

## PAGE 6: Discount & Profitability Analysis

**Purpose:** Prove the link between discounting and profit destruction.

### Visuals

| Visual | Type | Fields | Notes |
|--------|------|--------|-------|
| Discount vs Profit scatter | Scatter | X=Discount%, Y=Profit, Size=Revenue | Per order line |
| Margin by discount band | Column chart | Discount Band, [Profit Margin %] | Key insight |
| Loss orders by sub-category | Bar chart | Sub-Category, [Loss Orders] | Sorted |
| What-If simulator | Slicer + Cards | Discount Threshold slider, [Profit Above Threshold], [Projected Improvement] | Interactive |
| Discount by category heatmap | Matrix | Category × Discount Band, [Total Orders] | — |

### What-If Parameter Setup
1. Modeling → New Parameter → Numeric Range
2. Name: "Discount Threshold", Type: Decimal Number
3. Min: 0, Max: 0.8, Increment: 0.05, Default: 0.2
4. Add slicer to page
5. Add cards: [Profit Above Discount Threshold], [Projected Profit Improvement]

### The Key Story for This Page
"Orders with discounts above 20% consistently generate negative margins.
 If we cap discounts at 20%, projected profit improvement = $X"

---

## NAVIGATION SETUP

Create navigation buttons on each page:
1. Insert → Buttons → Blank
2. Add icon (using format pane → Icon)
3. Action → Page Navigation → select target page
4. Group all navigation buttons on each page

Recommended icons (use emoji or download icon set):
- 🏠 Home → Page 1
- 📈 Trends → Page 2
- 📦 Products → Page 3
- 🗺️ Regions → Page 4
- 👥 Customers → Page 5
- 💰 Discounts → Page 6

---

## PUBLISHING CHECKLIST
- [ ] All 6 pages complete and reviewed
- [ ] Navigation buttons working on all pages
- [ ] Slicers have default selections
- [ ] Conditional formatting applied on key visuals
- [ ] Drillthrough page working for Product Detail
- [ ] Phone layout done for Page 1
- [ ] File → Publish → Power BI Service
- [ ] In Service: schedule daily refresh
- [ ] Get shareable link → add to resume and LinkedIn
