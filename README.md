# E-commerce Sales Analysis

End-to-end analysis of 50,000 orders from an online electronics retailer (January 2024 – June 2026): data cleaning and validation, revenue performance, product mix, revenue leakage, customer value and discount effectiveness, with business recommendations.

**Tools:** Python (pandas, NumPy, SciPy, Matplotlib) in Google Colab

---

## Contents

1. [Executive summary](#executive-summary)
2. [Business questions](#business-questions)
3. [Dataset](#dataset)
4. [Data cleaning and validation](#data-cleaning-and-validation)
5. [Key definitions](#key-definitions)
6. [Methodology](#methodology)
7. [Findings](#findings)
   - [1. Revenue performance](#1-revenue-performance)
   - [2. Product mix](#2-product-mix)
   - [3. Revenue leakage](#3-revenue-leakage)
   - [4. Customer value](#4-customer-value)
   - [5. Discount effectiveness](#5-discount-effectiveness)
8. [Recommendations](#recommendations)
9. [Limitations](#limitations)
10. [Repository structure](#repository-structure)

---

## Executive summary

| Area | Headline |
|---|---|
| **Revenue** | Flat at \~105K per month through 2024–2025, then a sudden **\~32% drop from February 2026** that hit every segment equally. |
| **Products** | **Electronics brings in 51% of revenue**; 10 of 20 products bring in 71%. Cheap accessories drive orders, not revenue. |
| **Leakage** | **14.4% of potential revenue (504K) is lost**, mostly through order and payment records that don't match, not through lost sales. |
| **Customers** | Segment labels, age and city **do not predict customer value**; buying behaviour does. 22% of buyers bring in 32% of revenue. |
| **Discounts** | 65% of orders are discounted, costing **246K**, but discounts **do not make customers buy more**. |

**Top priorities:** verify the February 2026 drop with the data team, fix order–payment reconciliation, win back high-value customers who have gone quiet, and reduce blanket discounting.

---

## Business questions

1. **Revenue performance:** How has revenue changed month by month, and is there a seasonal pattern?
2. **Product mix:** Which categories and products bring in the most revenue, and which bring in the most orders?
3. **Revenue leakage:** How much potential revenue is lost through cancellations, returns and payment problems, and where does it come from?
4. **Customer value:** Who are our buyers, which customers have potential, and who should the business focus on?
5. **Discount effectiveness:** Do discounts make customers buy more?

---

## Dataset

Four related tables, joined on `OrderID`, `CustomerID` and `ProductID`:

| Table | Rows | Contents |
|---|---|---|
| `orders.csv` | 50,120 | Order date, customer, product, quantity, discount, payment method, order status |
| `payments.csv` | 50,000 | Payment date and payment status (Paid / Failed / Refunded) for each order |
| `customers.csv` | 10,000 | Age, city, signup date, customer segment (New / Regular / VIP) |
| `products.csv` | 20 | Product name, category (6 categories), unit price |

---

## Data cleaning and validation

### Issues found and how they were handled

| Table | Issue | Rows | Action |
|---|---|---|---|
| Customers | Missing age | 180 | Grouped as "Unknown" age group |
| Customers | Missing city | 119 | Labelled "Unknown" |
| Customers | City spelling variants (`tehran`, `Mashad`) | 110 | Standardised to Tehran and Mashhad |
| Orders | Exact duplicate rows | 120 | Removed (each appeared twice) |
| Orders | Missing discount | 221 | Set to 0% (no discount) |
| Orders | Missing payment method | 452 | Labelled "Unknown" |
| Orders | Missing quantity | 80 | Kept but flagged; excluded from revenue, since revenue cannot be calculated |
| Orders + Payments | Missing order and payment date (same rows) | 35 | Kept in totals; excluded from time-based analysis |
| Orders | Zero or negative quantity | 25 | Removed (0.05% of orders). Tested whether negative quantities were returns: only 2 of 19 had a matching earlier purchase, so they were treated as invalid. |
| Orders | Customer ID 999999 not in customers table | 30 | Treated as guest orders: kept in revenue totals, excluded from customer-level analysis |

All 50,000 orders matched exactly one payment record (no orphan orders or payments).

### Data quality findings

- **Order status and payment status disagree on 13.9% of orders (6,945).** For example, over 2,300 cancelled orders are marked *Paid* and 1,752 completed orders are marked *Failed*. Payment status is distributed almost identically across all order statuses (\~93% Paid), which suggests the two systems are not reconciled. Revenue is therefore only counted when an order is **Completed and Paid**.
- **28% of orders (13,987) are dated before the customer's signup date.** This is too common to be a typing error, so `SignupDate` probably records something other than account creation. Signup-based measures (such as time from signup to first order) were not used.
- **Payment date always equals order date,** so the time between ordering and paying carries no information.
- **Monitor's unit price (21) looks wrong:** it is cheaper than a desk lamp, and its quantity never exceeds 3 while every other product reaches 5.

---

## Key definitions

| Term | Meaning |
|---|---|
| **Realized revenue** | Revenue from orders that are **Completed and Paid**, after discount |
| **Potential revenue** | Revenue if every order had been successful |
| **Revenue leakage** | Potential revenue minus realized revenue |
| **Buyer** | A registered customer with at least one realized order |
| **Behaviour group** | Customer group based on how recently and how often they buy (RFM analysis): Champions, Loyal, At risk, Promising, Needs attention, Hibernating |
| **Adjusted p-value** | p-value corrected for running many tests at once (Benjamini–Hochberg), so chance results aren't mistaken for real differences |

---

## Methodology

- **Comparisons between groups** used non-parametric tests (Kruskal–Wallis, Mann–Whitney), since revenue and order counts are skewed, and **chi-square tests** for counts and proportions.
- **Relationships** were measured with Spearman correlation and linear trend tests.
- **Multiple testing:** when many groups or products were tested at once, p-values were adjusted (Benjamini–Hochberg) to avoid false positives.
- **Revenue leakage over time** was monitored with a statistical control chart (±3 standard deviations).
- **Repeat-offender check:** the number of unsuccessful orders per customer was compared with a binomial "pure chance" model.
- **Customer segmentation** used RFM scoring: each buyer was scored 1–5 on recency, frequency and monetary value, then grouped by recency and frequency.
- **Price sensitivity** was measured within each product (price elasticity), using discounts as the only source of price variation.
- **Trend and seasonality** were measured on 2024–2025 only, so the February 2026 drop would not distort them.

Significance level: p < 0.05.

---

## Findings

### 1. Revenue performance

![Monthly realized revenue](images/01_monthly_revenue.png)

- **Revenue was flat through 2024–2025** at \~105K per month. 2025 revenue was within 0.4% of 2024, with no significant trend (p = 0.79).
- **From 1 February 2026, revenue fell \~32% overnight and stayed there.**
  - The drop came from fewer orders (−33%) and fewer active customers (−31%); average order value did not change.
  - Daily orders fell from \~48 to \~33 between January and February 2026 (p < 0.001).
  - No category, city, segment or payment method was hit harder than another (no share changed by more than 1 percentage point).
  - A sudden, even drop like this usually points to an external or technical cause, such as a sales channel or data feed that stopped being recorded. **It needs to be confirmed with the data owner before being treated as a real fall in sales.**
- **Registered customers nearly tripled** (3,653 → 10,000) over 2024–2025, but **monthly active customers stayed flat** at \~1,400 (p = 0.41), so new signups did not turn into more buyers. This is indicative only, given the unreliable signup dates.

![Revenue by calendar month, each year](images/02_revenue_by_calendar_month.png)

- **No monthly seasonal pattern:** differences between calendar months are random (p = 0.64), and 2024 and 2025 do not rise and fall in the same months (p = 0.38). There is no peak season to plan stock or campaigns around.

![Average daily revenue by weekday](images/03_revenue_by_weekday.png)

- **Clear weekly pattern:** revenue is 10–13% above average on Saturday and Sunday and lowest mid-week (p < 0.001).

### 2. Product mix

![Share of orders vs share of revenue by category](images/04_category_orders_vs_revenue.png)

| Category | Share of orders | Share of revenue | Revenue per order |
|---|---|---|---|
| Electronics | 36% | **51%** | 98 |
| Accessories | 37% | 16% | 31 |
| Home Office | 6% | 13% | 143 |
| Wearables | 6% | 13% | 141 |
| Gaming | 3% | 6% | 119 |
| Stationery | 11% | 2% | 12 |

- **Electronics brings in the most revenue.** Accessories has a similar share of orders but only a third as much revenue.
- **Home Office and Wearables** bring in about **twice their share of orders** in revenue. They are high-value niches with room to grow.

![Orders vs revenue by product](images/05_product_orders_vs_revenue.png)

*Products above the dashed line bring in more than their share of orders in revenue.*

- **Revenue is concentrated:** the top 10 products bring in **71% of revenue** from 36% of orders. Headphones, Office Chair and Tablet alone make up 28%; Tablet is in the top 3 with only 1.3% of orders.
- **Cheap items bring in orders, not revenue:** the bottom 5 products (cables, notebook, mouse, monitor) make up 37% of orders but only 10% of revenue.
- **Price affects how often a product is ordered** (cheaper products get far more orders, r = −0.76), **but not how many units are bought per order** (\~1.9 for every product).
- **The mix is stable:** category shares did not change between 2024 and 2025 (p = 0.53).

### 3. Revenue leakage

**14.4% of potential revenue is lost:** 504K of 3.50M potential revenue was not realized.

| Leakage type | Orders | Lost revenue | Share of leakage |
|---|---|---|---|
| **Refund owed to customer** (cancelled or returned, but still marked as paid) | 3,711 | 268K | **53%** |
| **Payment not collected** (delivered, but payment failed) | 1,752 | 118K | 23% |
| **Refunded after delivery** | 1,394 | 101K | 20% |
| Normal cancellation or return (handled correctly) | 284 | 17K | 3% |

- **Most of the leakage is a records problem, not lost sales.** Over half is money the business may *owe* customers (cancelled or returned orders still marked as paid), which is a customer-trust and possibly legal risk.
- **The leakage rate is the same everywhere:** 13–15% across payment methods, categories, customer segments, cities and discount levels (no significant differences).

![Monthly revenue leakage rate](images/06_leakage_control_chart.png)

- **It is stable over time:** no month falls outside the control limits.
- **Customers are not put off:** customers buy again within 180 days at the same rate after an unsuccessful order as after a successful one (\~64%, p = 0.92).
- **No repeat offenders:** unsuccessful orders per customer match what random chance predicts (p = 0.66). There is no sign of refund abuse by specific customers.

**Conclusion:** because the leakage is identical across every group, the cause is a **process problem**: the order system and the payment system are not kept in sync.

### 4. Customer value

**Who our buyers are:** 9,858 of 10,000 registered customers (98.6%) have bought at least once. They are mostly Regular-labelled (56%), aged 36–65 (61%), and based in Tehran (28%) and Mashhad (13%). A typical buyer placed \~4 orders worth \~300 in total over 2.5 years.

![Average revenue per buyer by profile group](images/07_revenue_per_buyer_by_profile.png)

**Profile does not predict value.** Across segment labels, age groups and cities, there is no significant difference in:
- buyer rate (share of customers who bought)
- revenue per customer
- orders per customer
- revenue per order
- days between orders
- which categories they buy

All 15 tests are non-significant after adjustment (adjusted p > 0.1). **The VIP label does not identify valuable customers:** VIPs spend the same as everyone else (296 vs 298–300) and are no more likely to be top customers.

**Behaviour does predict value.** Grouping buyers by how recently and how often they buy (RFM) separates them clearly:

![Share of buyers vs share of revenue by behaviour group](images/08_behaviour_groups.png)

| Behaviour group | Share of buyers | Share of revenue | Profile | Recommended action |
|---|---|---|---|---|
| **Champions** | 22% | **32%** | \~6.5 orders, \~455 each, bought \~2 months ago | Retain: loyalty rewards, early access |
| **Loyal** | 22% | 25% | \~5 orders, \~351 each | Grow: cross-sell other categories |
| **At risk** | 17% | **20%** | \~5 orders, \~357 each, **no order for \~11 months** | Win back: personalised re-engagement |
| **Promising** | 10% | 6% | \~3 orders, bought recently | Nurture: second-purchase programme |
| **Needs attention** | 7% | 4% | \~3 orders, middling recency | Re-activate: reminders, recommendations |
| **Hibernating** | 23% | 12% | \~2 orders, over a year ago | Low priority: automated emails only |

- Revenue per order is \~70 in every group, so **value comes from how often customers buy, not how much they spend per order.**
- Behaviour groups do not depend on segment label, age or city (adjusted p ≥ 0.06), which confirms that **profile fields cannot be used to find the best customers.**

### 5. Discount effectiveness

Each product has one fixed price, so discounts (0–30%) are the only place where the price of the same product changes. They were used to measure how sensitive customers are to price.

| Discount | Orders | Units per order | Revenue per order | vs no discount |
|---|---|---|---|---|
| 0% | 15,061 | 1.87 | 74.6 | — |
| 5% | 9,331 | 1.90 | 71.6 | −4% |
| 10% | 8,463 | 1.89 | 68.9 | −8% |
| 15% | 5,128 | 1.88 | 65.0 | −13% |
| 20% | 3,062 | 1.87 | 61.4 | −18% |
| 30% | 1,676 | 1.88 | 56.1 | −25% |

- **Discounts do not make customers buy more:** units per order stay at \~1.9 at every discount level (p = 0.56), and price elasticity is effectively zero (−0.007, 95% CI −0.06 to +0.05).
- **Discounts lower revenue per order without adding volume:** a 30%-discount order brings in 25% less revenue than a full-price order with the same number of units.
- **Discounted customers do not choose more expensive products** (average product price, order value and share of premium items show no difference, all p > 0.6).
- **The cost is large:** 65% of orders are discounted, giving away **246K (7.6% of full-price value)**.

![Units per order with and without discount, by product](images/09_discount_units_by_product.png)

- **No category or product responds to discounts.** Tablet (p = 0.006) and Desk Lamp (p = 0.041) looked significant on their own, but neither is after adjusting for testing 20 products (adjusted p = 0.11 and 0.41). Tablet is worth a controlled test, but its 30%-discount group has only 29 orders, and its revenue per order is no higher than at full price.

---

## Recommendations

In priority order:

1. **Investigate the February 2026 drop** with the data and engineering team. A sudden, even 32% fall across every group points to a technical or channel issue. Confirm it before making decisions based on 2026 figures.
2. **Fix order–payment reconciliation.**
   - *Finance:* check the 3,711 orders where a refund may be owed; issue refunds or correct the records.
   - *Operations:* confirm payment before dispatch, to stop delivering unpaid orders (118K).
   - *Customer service:* record a reason for every refund after delivery.
   - *IT/Data:* run a daily status-mismatch report with an owner; target a mismatch rate below 1% (currently 14%).
3. **Win back At-risk customers.** 1,666 proven buyers (\~5 orders, \~357 each) have gone quiet. Run a personalised re-engagement campaign, and check whether their lapse is linked to the February 2026 drop.
4. **Reduce blanket discounting**, starting with the 20–30% levels. Validate with an A/B test before rolling out; if one targeted discount is kept, test it on Tablet first.
5. **Retain Champions without discounts**, using loyalty rewards and early access for the 22% of buyers who bring in a third of revenue.
6. **Grow order value through the product range.** Protect stock and marketing for the top 10 products, bundle low-priced items (cables, cases) with higher-priced ones, and cross-sell Home Office and Wearables to Loyal customers, who currently buy from about 3 of 6 categories.
7. **Replace the New / Regular / VIP labels with behaviour groups** for customer targeting.
8. **Time campaigns for the weekend:** schedule promotions and emails for Friday–Sunday, when customers are most active.

---

## Limitations

- **Signup dates are unreliable** (28% of orders predate signup), so signup-based measures were not used, and the customer-growth comparison is indicative only.
- **Each product has one fixed price and there is no cost data,** so profitability and full price optimisation could not be analysed. Adding cost of goods sold would allow profit-based analysis.
- **Monitor's unit price looks incorrect** and should be verified; its revenue is likely understated.
- **The dataset may be synthetic.** Several patterns suggest this, such as identical behaviour across all customer groups, and a Saturday–Sunday peak in an Iranian market where the weekend is Thursday–Friday. Findings are framed as what they would mean for a real business.

---

## Repository structure

```
Sales-Analysis/
├── README.md                            # this report
├── E-commerce Sales Analysis.ipynb      # full analysis notebook (cleaning, analysis, tests)
├── customers.csv                        # raw data
├── orders.csv
├── payments.csv
├── products.csv
├── clean/                               # cleaned tables (star schema) for SQL / Power BI
│   ├── fact_orders.csv
│   ├── dim_customers.csv
│   └── dim_products.csv
└── images/                              # charts used in this report
```

To reproduce the analysis, open the notebook in Google Colab and run all cells; it loads the raw data directly from this repository.
