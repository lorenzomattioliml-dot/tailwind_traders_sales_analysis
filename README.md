
# Tailwind Traders — Multi-Country Sales & Currency Analysis

An end-to-end Power BI project covering data preparation, star-schema data modeling, DAX time-intelligence measures, multi-page interactive reporting, and Power BI Service deployment (executive dashboard, mobile layout, data alerts, and scheduled subscriptions) for a fictional multinational retail company.

<img width="1385" height="789" alt="Screenshot 2026-09-08 140959" src="https://github.com/user-attachments/assets/71af3d92-17fe-4480-a724-280927131e50" />

---

## Overview

Tailwind Traders sells across multiple countries and currencies. This project builds the full analytics layer needed to evaluate performance consistently across markets: cleaning and typing raw transactional data, modeling it into a relational structure, layering in currency conversion and time intelligence, and packaging the results into a report and dashboard suitable for business stakeholders.

---

## Business Problem

The analysis was scoped around four questions:

- Which sales representatives, product categories, and countries drive net revenue?
- How does order quantity relate to actual profitability — is a higher unit count always higher value?
- What share of orders results in returns, and does warranty length vary meaningfully by supplier?
- What does performance look like once all figures are normalized to a single reporting currency (USD)?

---

## Data Sources

| Source | Description | Rows |
|---|---|---|
| Sales | Order-level data: products, pricing, tax, quantity, sales representative, customer, country | 54 orders |
| Purchases | Return status, warranty length, return policy, supplier, purchase and last-visit dates | 54 records |
| Countries | Country reference table, linked to exchange rate | 5 countries |
| Exchange Data | Currency exchange rates (USD, GBP, EUR, AED, AUD), generated via a Python script inside Power BI | 5 currencies |
<img width="669" height="390" alt="Screenshot 2026-09-07 123419" src="https://github.com/user-attachments/assets/ed4156e2-53bb-4305-a99d-fd4c690d630a" />

All source files were provided as Excel workbooks. The exchange rate table was generated programmatically using a Python data source (pandas) rather than imported as a static file.

---

## Data Model

<img width="1304" height="795" alt="Screenshot 2026-09-07 123655" src="https://github.com/user-attachments/assets/f6a684f1-d808-4e5d-a8fe-59404f3df98c" />


Star-schema-oriented model with six tables:

- Sales to Countries — many-to-one, on Country ID
- Countries to Exchange Data — one-to-one, on Exchange ID
- Purchases to Sales — one-to-one, on OrderID
- CalendarTable (custom DAX date table, 2020-2023) to Purchases — many-to-one, on Date / Purchase Date
- Sales in USD (calculated table) to Sales — one-to-one, on OrderID

CalendarTable is marked as a date table. Relationships between one-to-one pairs are automatically bidirectional in Power BI, which allows time-intelligence measures on the Sales in USD table to respond correctly to CalendarTable filters through the full relationship chain (CalendarTable to Purchases to Sales to Sales in USD).

---

## Technical Approach

**Data preparation (Power Query)**
- Explicit data typing enforced on every column (integers, fixed decimals, text, dates) rather than relying on auto-detection
- Verified 100 percent column quality (no errors or blanks) on key fields before modeling
- Used column distribution and distinct/unique value profiling to identify outliers before they reached the model

**Calculated columns (Sales table)**
- Gross Revenue = Gross Product Price times Quantity Purchased
- Total Tax = Tax Per Product times Quantity Purchased
- Net Revenue = Gross Revenue minus Total Tax

**Data modeling and DAX**
- Custom CalendarTable built with CALENDAR() and ADDCOLUMNS() for year, month, quarter, and weekday-level time intelligence
- Sales in USD calculated table: converts Gross Revenue, Net Revenue, and Total Tax into a single reporting currency using RELATED() against the exchange rate table, enabling direct cross-country comparison
- Annual Profit Margin — DIVIDE(SUM(Net Revenue USD), SUM(Gross Revenue USD)), formatted as a percentage
- Quarterly Profit — CALCULATE(SUM(Net Revenue USD), DATESQTD(CalendarTable[Date]))
- Annual Profit (YTD) — TOTALYTD(SUM(Net Revenue USD), CalendarTable[Date])
- Median Sales — MEDIAN(Gross Revenue USD)
- Report performance validated with Performance Analyzer; DAX query time for all four measures confirmed under 200 milliseconds
<img width="296" height="80" alt="Screenshot 2026-09-07 151450" src="https://github.com/user-attachments/assets/cd038992-7851-461b-9273-0756649ee93c" />
<img width="310" height="78" alt="Screenshot 2026-09-07 151319" src="https://github.com/user-attachments/assets/92b101fc-1f5b-4d2f-812a-1fb5ee5f36a0" />

---

## Report Design

The report contains four pages:

1. **Cover** — project title, business context, and headline metrics
2. **Sales Overview** — loyalty points by country, quantity sold by product, median sales distribution by country, median sales over time, supporting KPI cards, and a country slicer
3. **Profit Overview** — net revenue by product, annual profit margin by country, annual profit margin over time, YTD profit and net revenue cards, a gross revenue KPI with trend axis, and a date slicer
4. **Sales Rep and Returns Detail** — net revenue by sales representative, return status distribution, and average warranty by supplier, using fields not surfaced elsewhere in the report
<img width="1385" height="789" alt="Screenshot 2026-09-08 140959" src="https://github.com/user-attachments/assets/723bba60-b56e-40ff-940a-f2767107745f" /><img width="1381" height="786" alt="Screenshot 2026-09-08 141025" src="https://github.com/user-attachments/assets/6bb5f0a1-c023-473c-bd0d-09ab50f9baac" /><img width="1385" height="790" alt="Screenshot 2026-09-08 141042" src="https://github.com/user-attachments/assets/24fc94f4-d9b9-4f13-b7e2-5af221fcb137" />



Design elements applied consistently across all pages:
- A custom color theme (five to seven colors, no default Power BI styling)
- In-page text callouts stating the key finding directly on the relevant chart, rather than leaving the reader to infer it
- A page navigation bar with an active-page indicator, present on every page

---

## Mobile-Responsive Design

<img width="424" height="729" alt="Screenshot 2026-09-08 141611" src="https://github.com/user-attachments/assets/25ca43b1-d436-4ab7-b2d5-1df9753c2b19" />


Cards, KPIs, and core charts were rearranged using Power BI Service's dedicated mobile layout editor — a purpose-built layout rather than a scaled-down version of the desktop report — so the executive dashboard remains legible on a phone screen.

---

## Power BI Service Deployment

**Executive Dashboard**
All primary visuals from the Sales Overview and Profit Overview pages were pinned to a single Tailwind Traders Executive Dashboard, giving stakeholders a one-screen summary without navigating the full report.
<img width="1382" height="806" alt="Screenshot 2026-09-08 154455" src="https://github.com/user-attachments/assets/f274c123-12a9-447a-b00f-d5bc83526db8" />

**Data alerts**
A threshold alert on the Gross Revenue USD KPI tile notifies if the value drops below 400 dollars, checked at a maximum frequency of 24 hours.

<img width="371" height="556" alt="Screenshot 2026-09-08 153733" src="https://github.com/user-attachments/assets/0ea1a925-d32c-4db1-bad6-cf43c05d0b6a" />

**Scheduled subscriptions**
- Weekly Sales Summary — Sales Overview page, delivered every Monday at 5:00 AM
- Weekly Profit Summary — Profit Overview page, delivered every Monday, Wednesday, and Friday at 6:00 AM

---

## Key Insights

- Top product: Modular Sofa Set generates the highest net revenue of any product in the catalog.
- Top market by loyalty points: the United Kingdom leads all countries on total loyalty points.
- Top market by median sales: [fill in once confirmed from the median sales pie chart].
- Top performer: Bob generates the highest net revenue among sales representatives, confirmed on the full 54-order dataset.
- Returns: 31 percent of all orders are returned, a meaningful operational signal rather than noise.
- Warranty by supplier: ComfortZone Forniture offers the longest average warranty among all suppliers (overall range 6 to 48 months, average approximately 19 months).
- Quantity is not a reliable proxy for value: several of the highest-revenue orders came from single-unit purchases of high-price items.
- Currency exposure: converting to USD shows that revenue figures in local currency understate comparative value for orders in stronger currencies.

---

## Recommendations

1. Track net revenue per unit, not total quantity alone, when evaluating product or sales representative performance.
2. Investigate the return-rate driver by supplier and warranty length. A 31 percent return rate is high enough to warrant a root-cause review, starting with whether shorter-warranty suppliers show disproportionately more returns than ComfortZone Forniture.
3. Standardize all cross-country reporting on the USD-converted table to avoid comparing figures across incompatible currencies.
4. Review what Bob and the UK market are doing differently from the rest of the sales force and customer base, as both are strong enough signals to justify a deeper segment-level analysis.

---

## Tech Stack

Power BI Desktop, Power Query (M), DAX, Python (pandas), Excel, Power BI Service

---

## Explore the Project

- Live interactive report: [add your Power BI Service Publish to Web link]
- Executive Dashboard screenshot: see assets/dashboard-mobile.png or the Power BI Service workspace
- One-page insights summary (PDF): [add your executive summary link]

---

## Repository Structure

```
├── data/                          Source Excel files (Sales, Purchases, Countries)
├── Tailwind Traders Report.pbix
├── assets/                        Screenshots used in this README
│   ├── dashboard-overview.png
│   ├── model-view.png
│   └── mobile-layout.png
└── README.md
```
