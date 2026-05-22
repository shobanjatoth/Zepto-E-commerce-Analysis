<img width="3000" height="2000" alt="elf-moondance-online-shop-7089820" src="https://github.com/user-attachments/assets/28badc6e-5c27-469b-9ae9-3142acf22e9b" />

## E-Commerce Inventory & Revenue Analytics (Zepto Business Case)
📌 Project Overview
This project delivers a SQL-driven analytics framework designed to optimize retail operations for a high-volume e-commerce platform managing thousands of SKUs across diverse product categories.

The primary focus is transforming fragmented transactional and inventory logs into high-impact operational insights. These insights empower business stakeholders to make data-backed decisions regarding dynamic pricing strategies, discount allocations, supply chain replenishment, and macro-level category management.

🎯 Business Challenges Addressed
Pricing Inconsistencies: Identifying base price deviations and unexpected system price fluctuations.

Discount Effectiveness: Evaluating if promotion tiers yield healthy sales volume growth or unnecessarily compress profit margins.

Inventory Volatility: Proactively monitoring live stock levels to flag critical supply chain gaps and prevent out-of-stock scenarios.

Category Revenue Shares: Assessing individual product category performance to isolate macro drivers of company revenue.

🏗️ Relational Database Architecture
The data pipeline relies on a structured schema composed of three core relational tables:

    [ PRODUCTS ]               [ INVENTORY ]
  - product_id (PK)          - product_id (FK)
  - product_name             - stock_on_hand
  - category_id              - reorder_level
  - base_price               - warehouse_id
        │                           │
        └─────────────┬─────────────┘
                      ▼
            [ SALES_TRANSACTIONS ]
          - transaction_id (PK)
          - product_id (FK)
          - quantity_sold
          - selling_price
          - discount_amount
          - transaction_date
products (Dimension): Master catalogue capturing static metadata including names, assigned verticals, and standardized baseline retail pricing.

inventory (Operational Fact): Granular snapshot tracking dynamic physical warehouse stock-keeping levels against defined reorder safety nets.

sales_transactions (Transactional Fact): High-velocity ledger recording real-time consumer orders, precise promotional metrics, and actualized sales revenue points.

📊 Analytical SQL Framework
1. Revenue Vulnerability & Price Variance
Detects structural anomalies or unauthorized systemic discounting by isolating items trading at multi-tier price variants below baseline projections.

SQL
SELECT 
    p.product_id,
    p.product_name,
    p.base_price,
    MIN(s.selling_price) AS min_selling_price,
    MAX(s.selling_price) AS max_selling_price,
    AVG(s.selling_price) AS avg_selling_price,
    COUNT(DISTINCT s.selling_price) AS distinct_price_points
FROM sales_transactions s
JOIN products p ON s.product_id = p.product_id
GROUP BY p.product_id, p.product_name, p.base_price
HAVING COUNT(DISTINCT s.selling_price) > 1;
2. Promo Tier Elasticity Analysis
Aggregates performance cross-referenced by custom discount tiers to evaluate volume velocity against net revenue margins.

SQL
SELECT 
    p.category_id,
    CASE 
        WHEN s.discount_amount = 0 THEN '0% No Discount'
        WHEN s.discount_amount / NULLIF(p.base_price, 0) <= 0.1 THEN '1-10% Low Discount'
        WHEN s.discount_amount / NULLIF(p.base_price, 0) <= 0.25 THEN '11-25% Mid Discount'
        ELSE '25%+ High Discount'
    END AS discount_tier,
    COUNT(s.transaction_id) AS total_orders,
    SUM(s.quantity_sold) AS total_units_sold,
    SUM((s.selling_price - s.discount_amount) * s.quantity_sold) AS net_revenue
FROM sales_transactions s
JOIN products p ON s.product_id = p.product_id
GROUP BY p.category_id, discount_tier
ORDER BY p.category_id, net_revenue DESC;
3. Supply Chain Stock-Gap Monitoring
Generates real-time, high-priority operational visibility into depleted SKU inventory pipelines requiring instant replenishment.

SQL
SELECT 
    product_id,
    product_name,
    stock_on_hand,
    reorder_level,
    CASE 
        WHEN stock_on_hand = 0 THEN 'OUT OF STOCK'
        WHEN stock_on_hand <= reorder_level THEN 'CRITICAL STOCK GAP'
        ELSE 'Healthy'
    END AS stock_status
FROM inventory
WHERE stock_on_hand <= reorder_level
ORDER BY stock_on_hand ASC;
4. Share of Wallet (Category Contribution)
Leverages CTEs windowed across total turnover figures to rank corporate revenue generation concentration.

SQL
WITH total_sales AS (
    SELECT SUM(selling_price * quantity_sold) AS overall_revenue 
    FROM sales_transactions
)
SELECT 
    p.category_id,
    SUM(s.selling_price * s.quantity_sold) AS category_revenue,
    ROUND((SUM(s.selling_price * s.quantity_sold) / (SELECT overall_revenue FROM total_sales)) * 100, 2) AS revenue_contribution_percent
FROM sales_transactions s
JOIN products p ON s.product_id = p.product_id
GROUP BY p.category_id
ORDER BY category_revenue DESC;
🚀 Key Business Takeaways & Impact
Margin Protection: Flagging anomalous low-price sales points preserves target profit thresholds.

Optimized Promotional Spend: Data proves whether deep discounts actively boost cross-selling metrics or unhealthily degrade gross product margins.

Reduced Customer Churn: Isolating inventory bottlenecks before they become an issue lowers order-cancellation rates, preserving critical flash-delivery SLA times.

🛠️ Tech Stack & Environment
Dialect: MySQL Compatible

Design Paradigm: Relational Database Management System (RDBMS) OLAP
