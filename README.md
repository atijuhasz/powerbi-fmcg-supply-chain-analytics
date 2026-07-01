# NorthStar FMCG Distribution — End-to-End BI Portfolio Project

> A Power BI portfolio project simulating the role of a BI Analyst at a multinational FMCG distributor — covering Sales Performance, Supply Chain Reliability, and Finance / Profitability in a single connected data model.

---

## Business Problem

NorthStar FMCG Distribution operates across 23 global regions and 10 product categories. Leadership lacks a single source of truth to answer three core questions:

- **Sales:** Where is revenue growing, which products drive the most volume, and how does performance vary by region and customer segment?
- **Supply Chain:** How reliable is our delivery performance, and which shipping modes and regions are causing the most late deliveries?
- **Finance:** How does our actual profitability compare to the annual budget plan, and where are our margins healthiest?

This report provides an interactive, drill-capable answer to all three — with actuals measured against a structured budget plan and KPIs tracked over a Jan 2015 – Jan 2018 window.

---

## Report Pages

| Page | Focus | Key Visuals |
|---|---|---|
| Executive Summary | One-page leadership view | KPI cards, Total Sales trend, Year & Market slicers |
| Sales Performance | Revenue deep-dive | Sales by Category / Region / Segment, Top 5 Products, Pareto callout |
| Supply Chain / Logistics | Delivery reliability | Late Delivery Risk % by Shipping Mode & Region, trend line |
| Finance / Profitability | Budget vs. Actual | Clustered bar chart, Variance % table with conditional formatting |
| Cost & Margin Analysis | Cost structure & margins | Cost breakdown (donut), Margin % by Category & trend |

---

## Key Business Insights

**1. Top 5 products generate 60.6% of total revenue**
A Pareto concentration pattern confirmed and quantified via DAX: five products out of 118 account for over three-fifths of the company's $33M revenue base. Significant SKU rationalisation opportunity in the long tail.

**2. First Class shipping has a 95% late delivery rate**
Counterintuitively, the premium shipping tier is the least reliable — nearly 2.5x worse than Standard Class (38% late rate). Root cause: overly aggressive 1-day delivery promises that the underlying logistics chain cannot consistently meet. Recommendation: review First Class SLA commitments before the next contract cycle.

**3. Budget variance is driven by category price mix, not volume**
The annual budget was built on order-volume weights with a uniform average order value. High-ticket categories (Dairy & Chilled, mapped from high-value electronics) structurally beat their budget (+248%), while low-ticket categories miss theirs (Baby Care: −85%). A revenue-mix-aware budgeting approach would produce tighter variance — a real S&OP planning consideration.

**4. Standard Class is the most reliable shipping tier**
Average shipping delay ≈ 0 days, on-time rate ~62% — the only tier consistently meeting its promised window. A strong candidate for volume migration away from underperforming First and Second Class tiers.

---

## Data Model

The model is a **snowflake schema** — a star schema extended with two conformed dimensions (`Dim_Category`, `Dim_Region`) to bridge a grain mismatch between SKU-level actuals and category/region-level budget data.

```
                    Dim_Date
                       │
          ┌────────────┼────────────┐
          │            │            │
     Fact_Orders   Fact_Budget      │
          │            │            │
    ┌─────┼──────┐     │            │
    │     │      │     │            │
Dim_    Dim_   Dim_  Dim_      Dim_
Product Customer Geo  Category  Region
    │    Mode    │      │          │
    └────────────┘      │          │
         │              │          │
    Dim_Category ───────┘    Dim_Region
    (conformed)              (conformed)
```

**Fact tables**
- `Fact_Orders` — 180,519 line-item rows; order date, shipping date, sales, profit, delivery status
- `Fact_Budget` — 8,510 rows; Category × Region × Month budget plan (Jan 2015 – Jan 2018)

**Dimension tables**
- `Dim_Date` — DAX `CALENDAR()` table spanning both order and shipping date ranges; marked as official Date Table
- `Dim_Product` — 118 SKUs with FMCG-relabelled Category, Sub-Category, and Product Name
- `Dim_Geography` — Order Country / Region / State / Market; surrogate key built in Power Query
- `Dim_Customer` — Customer Id and Segment only (address fields excluded — they describe store location, not customer geography)
- `Dim_ShippingMode` — Shipping Mode only; Delivery Status stays on `Fact_Orders` as a per-order outcome
- `Dim_Category` / `Dim_Region` — conformed dimensions (`DISTINCT()` DAX tables) enabling cross-fact-table filtering for Budget vs. Actual measures

**Two active date relationships:** `order date` (active) and `shipping date` (inactive, invoked with `USERELATIONSHIP()` for shipping-date analysis).

---

## DAX Measures

| Phase | Measures | Functions |
|---|---|---|
| 1 — Basics | Total Sales, Total Profit, Total Orders, AOV, Profit Margin % | `SUM`, `DISTINCTCOUNT`, `DIVIDE`, `COUNTROWS` |
| 2 — Time intelligence | Total Sales YTD, Sales PY, Sales YoY Growth % | `TOTALYTD`, `SAMEPERIODLASTYEAR`, `CALCULATE` |
| 3 — Filter context | On-Time Delivery %, Late Delivery Risk %, Avg Shipping Delay | `CALCULATE`, `AVERAGEX` |
| 4 — Variance | Sales Budget Variance %, Cost Budget Variance, Actual Cost | Cross-fact-table via conformed dims |
| 5 — Context manipulation | % of Total Sales, % of Region Sales | `ALL`, `ALLEXCEPT` |
| 6 — Ranking | Product Sales Rank, Top 5 Products Sales, Top 5 % of Total | `RANKX`, `TOPN`, `SUMX` |

---

## Data Sources

### Primary: DataCo Smart Supply Chain for Big Data Analysis
- Source: [Kaggle](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis)
- ~180,000 order rows, 50+ columns
- Covers: order/shipping dates, delivery status, late delivery risk, geography, product, sales, profit
- **Note on relabelling:** the source data is a sporting-goods retailer. A custom 118-row SKU mapping table (`product_fmcg_mapping.csv`) was built to re-label all products, categories, and sub-categories to realistic FMCG equivalents — merged into the model in Power Query on `Product Card Id`. Sales, profit, and order dynamics are completely unchanged; only the labels are remapped.

### Custom: Fact_Budget
- Built programmatically (Python) — not hand-typed
- Category and Region weights derived from real order-count distributions in the source data
- Dollar scale anchored to real `SUM(Order Item Total)` = $33,054,402
- Two explicit planning assumptions: 5%/year growth target (smooth forward plan, not a copy of actuals) and 75% cost ratio (~25% gross margin)
- Result: 8,510 rows across 10 categories × 23 regions × 37 months

---

## Tech Stack

| Tool | Purpose |
|---|---|
| Power BI Desktop (free) | Data model, DAX measures, report pages |
| Power Query (M) | Data shaping — encoding fix, column pruning, surrogate key, product mapping merge |
| DAX | All KPI measures across 6 phases |
| Python | Programmatic generation of `Fact_Budget` and `product_fmcg_mapping.csv` |
| Power BI Service | Publishing and shareable Publish to Web link |
| GitHub | Version control and portfolio hosting |

---

## How to View

**Option A — Power BI Service (recommended)**
👉 [View the live report](#) *(add your Publish to Web link here)*

**Option B — Power BI Desktop**
1. Download `NorthStar_FMCG_BI.pbix` from this repo
2. Open in [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free)
3. All data is embedded — no source files needed

---

## Project Structure

```
/
├── NorthStar_FMCG_BI.pbix          # Main Power BI file
├── data/
│   ├── product_fmcg_mapping.csv    # 118-row SKU → FMCG label mapping table
│   └── Fact_Budget.csv             # 8,510-row programmatic budget plan
├── README.md
└── screenshots/
    ├── 01_executive_summary.png
    ├── 02_sales_performance.png
    ├── 03_supply_chain.png
    ├── 04_finance_profitability.png
    └── 05_cost_margin_analysis.png
```

> **Screenshots:** take one screenshot per report page and drop them into a `/screenshots` folder in the repo — GitHub will render them inline if you add image links to this README, which makes a much stronger first impression than a blank folder.

---

## Modeling Notes & Lessons Learned

A few decisions worth documenting for transparency:

- **Grain mismatch between actuals and budget** — `Fact_Orders` is at SKU/line-item grain; `Fact_Budget` is at Category/Region/Month grain. Resolved with two conformed dimensions (`Dim_Category`, `Dim_Region`) sitting above the existing dimensions — a deliberate snowflake rather than forcing everything into a pure star.
- **Date/Time vs. Date type mismatch** — source date columns parsed as `Date/Time` (correct for locale-aware day/month parsing) then truncated to `Date` in a second Power Query step. Skipping this truncation step causes relationships to `Dim_Date` to silently fail — every row falls into an unmatched bucket and all time-intelligence measures return the same grand total.
- **Power Query Reference queries** — dimension tables are built as References off a shared `Orders_Staging` query (load disabled). Column trimming is applied only on the `Fact_Orders` branch. Trimming the source query directly would retroactively break all downstream References.
- **Budget price-mix limitation** — the budget was generated with one global average order value applied to all categories. High-ticket categories (Dairy & Chilled) structurally over-perform against this volume-based budget; low-ticket categories under-perform. Documented as a deliberate modeling tradeoff rather than hidden or corrected.

---

*Built as an end-to-end BI portfolio project demonstrating data modeling, Power Query, DAX, and dashboard design skills for an FMCG analyst role.*
