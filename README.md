# Electronics Retailer Analytics — Star Schema, Hypothesis Testing & Power BI Dashboard

End-to-end analytics project covering data modeling, statistical testing, and dashboard
design for a simulated multi-region electronics retailer (7 years of sales data, 100K
transactions).

## Business Problem

Retail leadership wants to know whether channel (Online vs. Retail), region, and discount
strategy are meaningfully driving differences in sales performance — and whether current
customer segmentation (loyalty tiers) reflects real behavioral differences worth acting on.

## Data

Source: [Complete DAX Practice Dataset](https://www.kaggle.com/datasets/thesnak/complete-dax-practice-dataset)
(Kaggle) — 6 related tables covering 100,000 sales transactions, 1,500 customers, 200
products, 50 employees, and 40 geographies across 2018–2024.

## Data Model

Designed and implemented a star schema in MySQL: one fact table (`FactSales`) surrounded by
5 dimension tables, with `DimDate` used as a **role-playing dimension** (referenced twice by
the fact table — once for order date, once for ship date).

![ERD](SQL/ER%20diagram.png)

**Data quality note:** 137 orders shipped into January 2025, beyond the original date
dimension's range — caught during schema validation and fixed by extending `DimDate` rather
than dropping the rows.

## Tools & Techniques

| Layer | Tools | What it demonstrates |
|---|---|---|
| Data modeling | MySQL, SQL DDL | Star schema design, primary/foreign keys, referential integrity |
| Analysis | SQL (window functions, views) | Multi-table joins, running totals, ranking |
| Statistics | Python (Pandas, SciPy) | Welch's t-test, chi-square test of independence, one-way ANOVA |
| Visualization | Power BI, DAX | Data modeling in Power BI, time-intelligence measures, role-playing dimension handling via `USERELATIONSHIP` |

## Key Findings

Three hypotheses were formally tested (full detail in `Notebook/EDA & Hypothesis testing.ipynb`):

1. **Sales channel does not significantly affect average order value** (Welch's t-test,
   p = 0.57) — channel prioritization should be based on cost-to-serve or margin, not order value.
2. **Product category is statistically independent of region** (chi-square, p = 0.54) — no
   evidence to justify region-specific merchandising strategies.
3. **Discount levels don't differ significantly by channel** (one-way ANOVA, p = 0.15) —
   discounting is applied consistently, not concentrated in any one channel.

All three results were non-significant, which is the expected and correct outcome for this
dataset — the value of this analysis is in the *methodology* (correct test selection,
clearly stated hypotheses, honest interpretation), not manufactured findings.

## SQL Analysis Highlights

From the window-function and aggregation queries in `SQL/Analysis code.sql`:

- **Revenue & profit by region** — revenue ranges from $14.02M (East) to $14.29M (Central),
  a spread of under 2%, and profit margin holds at roughly 35% in every region. No single
  region is disproportionately more profitable than another.
- **Running total of monthly sales** — cumulative revenue climbs steadily to $70.64M by
  December 2024 with no visible seasonal spike or slump, confirming consistent month-to-month
  contribution rather than a few outlier months driving totals.
- **Year-over-year growth (LAG)** — growth oscillates in a tight band between -0.97% and
  +1.75% with no sustained multi-year trend in either direction, reinforcing the flat revenue
  pattern seen in the Executive Summary dashboard.
- **Shipping delay by channel** — average ship delay is ~3.5 days across all four channels
  (3.51–3.53 days), meaning fulfillment speed doesn't depend on how the order came in.
- **Top customers by region (RANK)** — top regional spenders total roughly $20K–$25K over
  the 7-year window each; no single customer dominates a region's revenue.
- **Revenue by category and channel** — category totals stay close across all four channels
  (e.g. Clothing ranges $3.56M–$3.64M depending on channel), reinforcing the ANOVA finding
  that revenue behavior doesn't cluster around any one channel.

## Exploratory Data Analysis

From `Notebook/EDA & Hypothesis testing.ipynb`:

**Revenue is flat year over year** — no compounding growth across the 7-year window, each
year landing within $10.0M–$10.2M.

![Revenue by Year](Notebook/Total%20Revenue%20by%20Year%20output.png)

**Order value distributions are nearly identical across channels** — medians and
interquartile ranges line up closely for Phone, Retail, Online, and Partner, visually
confirming the t-test's non-significant result. Every channel shows the same long tail of
high-value outliers up to ~$5,200.

![Order Value by Channel](Notebook/Order%20Value%20Distribution%20by%20Channel%20output.png)

**Order volume is even across channels, but heavily skewed across loyalty tiers** — each
channel handles roughly 25K orders, while Bronze accounts for ~60K orders vs. Platinum's
~5K. Tier reflects customer *count*, not spending behavior — the Customer Behavior dashboard
page shows average order value stays roughly flat across tiers, so loyalty tier is a volume
signal here, not a value signal.

![Volume by Channel and Loyalty Tier](Notebook/Order%20Volume%20by%20Channel%20&%20Loyality%20Tier%20output.png)

**Category drives revenue more than region does** — Clothing leads in every region (~$2.8M–
$3.0M) and Furniture trails in every region (~$1.86M–$2.03M), a consistent category-level
pattern rather than a regional one — this is the visual counterpart to the chi-square test
finding no category-region association.

![Revenue by Category and Region](Notebook/Revenue-%20Category%20x%20Region%20output.png)

## Dashboard

Three-page Power BI dashboard:

**Executive Summary** — KPIs, YTD/YoY trend

![Executive Summary](Dashboard/Executive%20Summary%20Page.png)

**Regional & Channel Deep-Dive** — category × region matrix, channel comparison

![Regional Deep-Dive](Dashboard/Regional%20&%20Channel%20Deep-Dive%20Page.png)

**Customer Behavior** — loyalty tier breakdown, top customers, acquisition trend

![Customer Behavior](Dashboard/Customer%20Behavior%20Page.png)

## How to Run This Yourself

1. Run `SQL/Data modeling code.sql` in MySQL to build the schema
2. Load the 6 CSVs with `Raw Data/Load data code.py` (use the Extended `DimDate.csv`)
3. Run `SQL/Analysis code.sql` for the reusable view + analysis queries
4. Open `Notebook/EDA & Hypothesis testing.ipynb`, update the DB password, run all cells
5. Open `Dashboard/Electronics Retail Sales & Profitability Analysis Dashboard.pbix` in Power BI Desktop, refresh the data source
