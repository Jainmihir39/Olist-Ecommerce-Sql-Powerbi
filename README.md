# Olist E-Commerce Sales Analysis (SQL + Power BI)

Analysis of the Brazilian Olist e-commerce dataset (about 99K orders, 2016-2018) to find revenue trends, top categories and the effect of late delivery on customer reviews.

**Tools:** MySQL, Power BI, Excel



![Dashboard](dashboard.png.jpg)



## Key findings
- Monthly revenue grew about 8x from Jan 2017 (about R$112K) to Jan 2018 (about R$925K), with a spike in Nov 2017.
- Top categories by revenue: health_beauty (R$1.26M), watches_gifts (R$1.21M), bed_bath_table (R$1.04M).
- About 8% of delivered orders arrived late. Late orders averaged a review score of 2.57 vs 4.29 for on-time orders.

## SQL used
```sql
-- Monthly revenue
SELECT LEFT(o.order_purchase_timestamp, 7) AS month,
       ROUND(SUM(i.price), 2) AS revenue,
       COUNT(DISTINCT o.order_id) AS orders
FROM olist_orders_dataset o
JOIN olist_order_items_dataset i ON o.order_id = i.order_id
WHERE o.order_status = 'delivered'
GROUP BY month ORDER BY month;

-- Top 10 categories by revenue
SELECT t.product_category_name_english AS category,
       ROUND(SUM(i.price), 2) AS revenue
FROM olist_order_items_dataset i
JOIN olist_products_dataset p ON i.product_id = p.product_id
JOIN product_category_name_translation t ON p.product_category_name = t.product_category_name
GROUP BY category ORDER BY revenue DESC LIMIT 10;

-- Late vs on-time delivery and review score
SELECT CASE WHEN o.order_delivered_customer_date > o.order_estimated_delivery_date
            THEN 'Late' ELSE 'On time' END AS delivery,
       COUNT(*) AS orders, ROUND(AVG(r.review_score), 2) AS avg_review
FROM olist_orders_dataset o
JOIN olist_reviews_clean r ON o.order_id = r.order_id
WHERE o.order_status = 'delivered' AND o.order_delivered_customer_date <> ''
GROUP BY delivery;
```

## Notes
- Data source: Olist Brazilian E-Commerce Public Dataset (Kaggle). Currency is Brazilian Real (R$).
- Review comments were removed before import, because only review scores were needed.
- About 2% of product rows did not import, so category totals may be slightly understated.
