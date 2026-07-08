# Retail Sales Performance Dashboard

A Power BI dashboard analyzing retail sales, profit, and customer segments to identify growth opportunities and underperforming areas.

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-005A9C?style=flat)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

---

## Business question

Which product categories and regions are driving growth, and where is performance declining, so leadership can reallocate marketing spend and pricing strategy for next quarter?

---

## Dataset

- **Source:** [Sample Superstore dataset](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final) (Kaggle) — replace with your actual source link
- **Size:** ~10,000 orders, 4 years of transaction history
- **Tables used:**
  - `Orders` — order ID, order date, ship date, quantity, sales, profit
  - `Products` — product ID, category, sub-category, cost
  - `Customers` — customer ID, segment, region
  - `Date` — custom date table built in Power Query (not from source data)

> Replace this section with your actual dataset name/link once you've picked one.

---

## Data modeling approach

Rather than using one flat table, I built a star schema with a single fact table (`Orders`) connected to dimension tables (`Products`, `Customers`, `Date`, `Region`). This:

- Improves query performance in large models
- Enables proper time intelligence (YoY, running totals)
- Reflects how real-world enterprise data models are structured

**Key transformations in Power Query:**
- Removed duplicate order lines and nulls in the `Sales` column
- Split `Order Date` into a standalone `Date` dimension table with Year, Month, Quarter, and Day-of-week columns
- Standardized inconsistent region/state naming
- Created a `Product Cost` column to enable margin calculations not present in raw data

---

## DAX measures

```dax
Total Sales = SUM(Orders[Sales])

Total Profit = SUM(Orders[Profit])

Profit Margin % = DIVIDE([Total Profit], [Total Sales], 0)

Sales YoY % = 
VAR CurrentSales = [Total Sales]
VAR PriorYearSales = 
    CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))
RETURN 
    DIVIDE(CurrentSales - PriorYearSales, PriorYearSales, 0)

Running Total Sales = 
CALCULATE(
    [Total Sales],
    FILTER(
        ALLSELECTED('Date'[Date]),
        'Date'[Date] <= MAX('Date'[Date])
    )
)

Top Category Rank = 
RANKX(ALL(Products[Category]), [Total Sales])

Avg Order Value = 
DIVIDE([Total Sales], DISTINCTCOUNT(Orders[Order ID]), 0)
```

---

## Report pages

**1. Executive summary**
- KPI cards: total sales, total profit, profit margin %, YoY growth
- Sales trend line over time
- Sales by region (map or bar chart)

**2. Product performance**
- Sales and profit by category/sub-category
- Top 10 and bottom 10 products by profit margin
- Highlights categories with high revenue but low/negative margin

**3. Customer & segment view**
- Sales by customer segment
- Order frequency and average order value by segment
- Regional breakdown of top customers

---

## Key insight

> Replace with your actual finding once you've explored the data. Example:
>
> The Furniture category generates the second-highest revenue but has a negative profit margin (-2.4%), largely driven by high shipping costs on bulky items. Meanwhile, the Technology category has both strong revenue and the highest margin (17%). **Recommendation:** renegotiate freight terms for Furniture or adjust pricing on large items, and consider reallocating marketing spend toward Technology.

---

## Screenshots

![Executive Summary](screenshots/overview.png)
![Product Performance](screenshots/product-page.png)
![Customer View](screenshots/customer-page.png)

---

## Tools used

- Power BI Desktop (data modeling, DAX, visualization)
- Power Query (data cleaning and transformation)
- Source data: Kaggle / Maven Analytics (replace with actual source)

---

## How to view

1. Download `dashboard.pbix`
2. Open in Power BI Desktop (free download from Microsoft)
3. Or view the published report here: `[link to Power BI Service report]`

---

## What I'd improve with more time

- Add customer churn/retention analysis with cohort tracking
- Incorporate external data (e.g., regional economic indicators) for deeper context
- Add row-level security (RLS) to demonstrate enterprise-readiness
