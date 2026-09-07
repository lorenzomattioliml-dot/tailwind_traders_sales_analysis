# Tailwind Traders — Multi-Country Sales & Currency Analysis

End-to-end Power BI project analyzing sales performance, product returns, and multi-currency revenue for a fictional retail company operating across multiple countries.

![Dashboard Overview](assets/dashboard-overview.png)

---

##  Business Problem

Tailwind Traders sells across multiple countries and currencies, and leadership needs a single, reliable view to answer:

- Which sales representatives, product categories, and countries drive net revenue?
- How does order quantity relate to actual profitability — is "more units sold" always "more value"?
- What share of orders results in returns, and does warranty length correlate with return behavior?
- What does performance look like once everything is normalized to a single reporting currency (USD)?

This project builds the data model and reporting layer needed to answer those questions consistently.

---

##  Data Sources

| Source | Description | Rows |
|---|---|---|
| `Sales` | Order-level sales data: products, pricing, tax, quantity, sales rep, customer, country | 54 orders |
| `Purchases` | Returns, warranty length, return policy, supplier, purchase/last-visit dates | 54 records |
| `Countries` | Country reference table, linked to exchange rate | 5 countries |
| `Exchange Data` | Currency exchange rates (USD, GBP, EUR, AED, AUD), loaded via a Python script (`pandas`) | 5 currencies |

<img width="669" height="390" alt="Screenshot 2026-09-07 123419" src="https://github.com/user-attachments/assets/0c83e7c1-bda0-40fa-99b1-22224304998c" />

All raw files were provided as Excel workbooks; the exchange rate table was generated programmatically inside Power BI using a Python data source.

---

##  Data Model

<img width="1304" height="795" alt="Screenshot 2026-09-07 123655" src="https://github.com/user-attachments/assets/1c6b0681-a381-463d-b568-a30b0c969a70" />


Star-schema-oriented model with 6 tables:

- **Sales** *(fact)* → **Countries** — many-to-one on `Country ID`
- **Countries** ↔ **Exchange Data** — one-to-one on `Exchange ID`
- **Purchases** ↔ **Sales** — one-to-one on `OrderID`
- **CalendarTable** *(DAX date table, 2020–2023)* → **Purchases** — many-to-one on `Date` / `Purchase Date`
- **Sales in USD** *(calculated table)* ↔ **Sales** — one-to-one on `OrderID`

All relationships use bidirectional cross-filtering where needed to allow filtering from either the sales or the reference/lookup side.

---

## ⚙️ Technical Approach

**Data preparation (Power Query)**
- Enforced explicit data types on every column (integers, fixed decimals, text, dates) rather than relying on auto-detection
- Verified 100% column quality (no errors/blanks) on key fields before modeling
- Filtered and profiled data (column distribution, distinct/unique value counts) to catch outliers before they hit the model

**Data modeling (DAX)**
- Built a custom `CalendarTable` using `CALENDAR()` + `ADDCOLUMNS()` for year/month/quarter-level time intelligence
- Built a calculated `Sales in USD` table that converts Gross Revenue, Net Revenue, and Total Tax into a single reporting currency using `RELATED()` against the exchange rate table — enabling true cross-country comparison

**Calculated columns (Excel/Power Query)**
- `Ricavo lordo` (Gross Revenue) = Gross Product Price × Quantity Purchased
- `Imposta totale` (Total Tax) = Tax Per Product × Quantity Purchased
- `Ricavo netto` (Net Revenue) = Gross Revenue − Total Tax

---

## 💡 Key Insights

- **Quantity ≠ value**: several of the highest-revenue orders came from *single-unit* purchases of high-price items — order volume alone is a misleading performance signal for this business.
- **Sales rep concentration**: in the initial order batch, one rep (Alice) appears across four different product categories with consistently mid-to-high net revenue, suggesting a cross-category top performer worth investigating at full-dataset scale.
- **Returns**: filtering `ReturnStatus` shows 39 of 54 orders (72%) as "Not Returned" — a 28% return rate is a meaningful baseline to track against warranty length (6–48 months, average ~19 months) and supplier.
- **Currency exposure**: converting to USD reveals that revenue figures in local currency understate true comparative value for orders in stronger currencies (e.g., an AED-denominated order's USD-equivalent value is materially higher once converted).

*(Replace with final numbers once the full dashboard is built — these reflect the initial data exploration phase.)*

---

##  Recommendations

1. Track **net revenue per unit**, not just total quantity, when evaluating product or rep performance.
2. Investigate the return-rate driver by supplier and warranty length — a 28% return rate on 54 orders is high enough to warrant a root-cause view.
3. Standardize all cross-country reporting on the USD-converted table to avoid comparing apples to oranges across currencies.

---

##  Tech Stack

`Power BI Desktop` · `Power Query (M)` · `DAX` · `Python (pandas)` · `Excel`

---

## 🔗 Explore the Report

- 📊 **[Live interactive report](#)** *(replace with your Power BI Service published link)*
- 📄 **[One-page insights summary (PDF)](#)** *(replace with your executive summary link)*

---

## 📁 Repository Structure

```
├── data/                  # Source Excel files (Sales, Purchases, Countries)
├── Tailwind Traders Report.pbix
├── assets/                # Screenshots and GIFs used in this README
└── README.md
```
