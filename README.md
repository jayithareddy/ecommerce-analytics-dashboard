# E-Commerce Sales & Customer Analytics Dashboard

## Problem Statement

Analyzing 2 years of order data from a Brazilian e-commerce marketplace to answer core business questions around revenue growth, customer retention, category/regional performance, and the relationship between delivery experience and customer satisfaction. The goal is to identify where the business should focus to grow sustainably, rather than just reporting historical numbers.

## Dataset & Tools

- **Dataset:** [Olist Brazilian E-Commerce Public Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (Kaggle, ~100k orders, Sept 2016–Aug 2018)
- **Database:** SQLite (queried via DB Browser for SQLite)
- **Analysis:** Python / Pandas (cohort retention analysis)
- **Visualization:** Tableau Public

## SQL Queries Used

**Monthly revenue trend**
```sql
SELECT 
    strftime('%Y-%m', o.order_purchase_timestamp) AS month,
    SUM(oi.price) AS total_revenue,
    COUNT(DISTINCT o.order_id) AS num_orders
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_status = 'delivered'
GROUP BY month
ORDER BY month;
```

**Repeat purchase rate** (joined through `customer_unique_id`, since Olist assigns a new `customer_id` per order)
```sql
SELECT 
    order_count,
    COUNT(*) AS num_customers
FROM (
    SELECT c.customer_unique_id, COUNT(DISTINCT o.order_id) AS order_count
    FROM orders o
    JOIN customers c ON o.customer_id = c.customer_id
    GROUP BY c.customer_unique_id
)
GROUP BY order_count
ORDER BY order_count;
```

**Delivery delay vs. review score**
```sql
SELECT 
    r.review_score,
    AVG(julianday(o.order_delivered_customer_date) - julianday(o.order_estimated_delivery_date)) AS avg_delay_days,
    COUNT(*) AS num_orders
FROM orders o
JOIN reviews r ON o.order_id = r.order_id
WHERE o.order_delivered_customer_date IS NOT NULL
GROUP BY r.review_score
ORDER BY r.review_score;
```

Full query set (including top categories and top states) is in `queries.sql`. Cohort retention logic (built in Pandas, not SQL) is in `cohort_analysis.ipynb`.

## Key Visuals

Dashboard published on Tableau Public: **[View live dashboard](https://public.tableau.com/views/OlistE-CommerceAnalyticsRevenueRetentionDeliveryInsights/Dashboard1?:language=en-GB&publish=yes&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)**

1. **Revenue Trend** — monthly revenue, Sept 2016–Aug 2018
2. **Repeat Purchases** — distribution of customers by number of orders placed
3. **Top Categories** — top 15 product categories by revenue
4. **Top States** — revenue by customer state
5. **Delivery vs. Reviews** — average delivery delay by review score
6. **Retention Heatmap** — % of each monthly cohort still purchasing in subsequent months

## Business Recommendations

- **Retention is the bigger opportunity than acquisition.** The large majority of customers place only one order, and the retention heatmap confirms this drops off almost to zero from month 1 onward for nearly every cohort. Acquisition spend alone won't fix this — a first-order-to-second-order incentive (e.g. a time-limited coupon after first delivery) is worth testing.
- **It's not lateness, it's lost delight.** Review scores don't track whether an order arrived late — almost every order arrived before the estimated date. What tracks with review score is *how much earlier* than estimated the order arrived (12.7 days early on average for 5-star reviews vs. 3.4 days for 1-star). This suggests tightening delivery estimates, or building in more buffer, could lift satisfaction without changing actual logistics speed.
- **Revenue is heavily concentrated in São Paulo.** SP dominates state-level revenue by a wide margin. Worth investigating whether this reflects an underserved opportunity in other states or simply mirrors Brazil's population/economic concentration.
- **A small number of categories drive most revenue.** Marketing and inventory investment should weight toward the top 15 categories identified in the dashboard rather than spreading evenly.

## How to Reproduce

1. Download the dataset from [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and unzip into a `data/` (or `archive/`) folder.
2. Run `code.ipynb` to load the CSVs into `olist.db`, run the SQL queries, and generate the cohort retention analysis — all steps are in this single notebook.
3. Query outputs are exported to the `results/` folder as CSVs.
4. Open Tableau Public, connect to each CSV in `results/`, and rebuild the 6 sheets described above, or open the saved `.twbx` workbook directly.

## Folder Structure

```
ecommerce-project/
├── data/ (or archive/)     raw Kaggle CSVs
├── results/                exported query outputs (revenue, repeat purchases, categories, states, delivery/reviews, retention)
├── olist.db                SQLite database
├── code.ipynb               loads data, runs SQL queries, and performs cohort/retention analysis
├── queries.sql              all 5 SQL queries (for reference / running outside the notebook)
├── dashboard.twbx           Tableau workbook
└── README.md
```
