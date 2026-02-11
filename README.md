# SQL_E-Commerce_Dataset
# I. Introduction
This project contains an eCommerce dataset that I will explore using SQL on Google BigQuery. The dataset is based on the Google Analytics public dataset and contains data from an eCommerce website.
# II. Requirement
    - Google Cloud Platform account
    - Project on Google Cloud Platform
    - Google Bigquery API enabled
    - SQL query editor or IDE
# III. Dataset Access
The eCommerce dataset is available in a public Google BigQuery dataset. To access it, complete the following steps:
   - Sign in to your Google Cloud Platform account and create a new project.
   - Open the BigQuery console and choose the project you just created.
   - From the navigation menu, click Add Data, then select Search a project.
   - Enter the project ID bigquery-public-data.google_analytics_sample.ga_sessions and press Enter.
   - Click on the ga_sessions_* table to view the data.

https://support.google.com/analytics/answer/3437719?hl=en
<img width="924" height="712" alt="Image" src="https://github.com/user-attachments/assets/e962f770-00d9-4764-b1d0-55a2832677dd" />
# IV. Explore the Dataset
In this project, I will write 08 query in Bigquery base on Google Analytics dataset
## Query 01: Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
- SQL code
```sql
SELECT 
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month,
    SUM(totals.visits) AS visits,
    SUM(totals.pageviews) AS pageviews,
    SUM(totals.transactions) AS transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY month
ORDER BY month;
```
- Query results
<img width="792" height="135" alt="Image" src="https://github.com/user-attachments/assets/597d7389-d8a2-4372-be8c-8c3507f27a74" />

## Query 02: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)
- SQL code
```sql
SELECT
    trafficSource.source as source,
    SUM(totals.visits) as total_visits,
    SUM(totals.Bounces) as total_no_of_bounces,
    (SUM(totals.Bounces)/SUM(totals.visits))* 100.00 as bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
GROUP BY source
ORDER BY total_visits DESC;
```
- Query results
<img width="788" height="536" alt="Image" src="https://github.com/user-attachments/assets/1ac4919d-ea51-44f8-a998-0690dac8be25" />

## Query 3: Revenue by traffic source by week, by month in June 2017
- SQL code
```sql
WITH month_revenue AS 
(
  SELECT 
      'Month' AS time_type 
      ,FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d', date)) AS time
      ,trafficSource.source AS source
      ,SUM(product.productRevenue) / 1000000 AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`
      ,UNNEST (hits) hits
      ,UNNEST (hits.product) product
  WHERE product.productRevenue IS NOT NULL
  GROUP BY time,source
),
week_revenue AS
(
  SELECT 
      'Week' AS time_type 
      ,FORMAT_DATE('%Y%W',PARSE_DATE('%Y%m%d', date)) AS time
      ,trafficSource.source AS source
      ,SUM(product.productRevenue) / 1000000 AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`
      ,UNNEST (hits) hits
      ,UNNEST (hits.product) product
  WHERE product.productRevenue IS NOT NULL
  GROUP BY time,source   
)
    SELECT * 
    FROM month_revenue
    UNION ALL
    SELECT *
    FROM week_revenue
    ORDER BY time_type,revenue DESC;
```
- Query results
<img width="983" height="707" alt="Image" src="https://github.com/user-attachments/assets/fadc1699-ff44-4390-bbf0-6ec48e9dc778" />

## Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017

- SQL code
```sql
WITH
purchaser_data AS(
  SELECT
      FORMAT_DATE("%Y%m",PARSE_DATE("%Y%m%d",date)) as month,
      SUM(totals.pageviews)/COUNT(DISTINCT fullvisitorid) as avg_pageviews_purchase,
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
    ,UNNEST(hits) hits
    ,UNNEST(product) product
  WHERE _table_suffix BETWEEN '0601' AND '0731'
    AND totals.transactions>=1
    AND product.productRevenue IS NOT NULL
  GROUP BY month
),

non_purchaser_data AS(
  SELECT
      FORMAT_DATE("%Y%m",PARSE_DATE("%Y%m%d",date)) as month,
      SUM(totals.pageviews)/COUNT(DISTINCT fullvisitorid) as avg_pageviews_non_purchase,
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
      ,UNNEST(hits) hits
      ,UNNEST(product) product
  WHERE _table_suffix BETWEEN '0601' AND '0731'
    AND totals.transactions IS NULL
    AND product.productRevenue IS NULL
  GROUP BY month
)

SELECT
    pd.*,
    avg_pageviews_non_purchase
FROM purchaser_data pd
FULL JOIN non_purchaser_data USING(month)
ORDER BY pd.month;
```
- Query results
<img width="778" height="102" alt="image" src="https://github.com/user-attachments/assets/e5189696-3632-4b3c-9158-50a2fcccfcfc" />

## Query 05: Average number of transactions per user that made a purchase in July 2017
- SQL code
```sql
SELECT
    FORMAT_DATE("%Y%m",PARSE_DATE("%Y%m%d",date)) as month,
    SUM(totals.transactions)/COUNT(DISTINCT fullvisitorid) as Avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    ,UNNEST (hits) hits,
    UNNEST(product) product
WHERE  totals.transactions>=1
    AND product.productRevenue IS NOT NULL
GROUP BY month;
```
- Query results
<img width="581" height="65" alt="image" src="https://github.com/user-attachments/assets/5fea76a8-de5b-486d-a564-adfc93fa0abd" />

## Query 06: Average amount of money spent per session. Only include purchaser data in July 2017

- SQL code
```sql
SELECT
    FORMAT_DATE("%Y%m",PARSE_DATE("%Y%m%d",date)) as month,
    ((SUM(product.productRevenue)/SUM(totals.visits))/POWER(10,6)) as avg_revenue_by_user_per_visit
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
  ,UNNEST(hits) hits
  ,UNNEST(product) product
WHERE product.productRevenue IS NOT NULL
  AND totals.transactions>=1
GROUP BY month;
```
- Query results
<img width="591" height="70" alt="image" src="https://github.com/user-attachments/assets/27c56178-ee9e-43e2-823a-db0b4bfcd14a" />

## Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.

- SQL code
```sql 
WITH customer AS 
(
  SELECT 
      DISTINCT(fullVisitorId)
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` 
      ,UNNEST (hits) hits
      ,UNNEST (hits.product) product
  WHERE product.v2ProductName = "YouTube Men's Vintage Henley"
      AND hits.eCommerceAction.action_type = '6'
      AND totals.transactions >= 1
      AND product.productRevenue IS NOT NULL 
)
    SELECT 
        product.v2ProductName AS other_purchased_products
        ,SUM(product.productQuantity) AS quantity
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` sessions
        ,UNNEST (sessions.hits) hits
        ,UNNEST (hits.product) product
    INNER JOIN customer
        ON sessions.fullVisitorId = customer.fullVisitorId
    WHERE product.v2ProductName != "YouTube Men's Vintage Henley"
        AND hits.eCommerceAction.action_type = '6'
        AND sessions.totals.transactions >= 1
        AND product.productRevenue IS NOT NULL
    GROUP BY other_purchased_products
    ORDER BY quantity DESC;
    ```
- Query results
<img width="485" height="708" alt="image" src="https://github.com/user-attachments/assets/3e6e2304-0a8f-4afa-a10d-8b0e6580ef4b" />

## Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. For example, 100% product view then 40% add_to_cart and 10% purchase.Add_to_cart_rate = number product  add to cart/number product view. Purchase_rate = number product purchase/number product view. The output should be calculated in product level.
- SQL code
Cách 1: 
```sql 
WITH
product_view as(
  SELECT
    FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) as month,
    COUNT(product.productSKU) as num_product_view
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '2'
  GROUP BY 1
),

add_to_cart as(
  SELECT
    FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) as month,
    COUNT(product.productSKU) as num_addtocart
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '3'
  GROUP BY 1
),

purchase as(
  SELECT
    FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) as month,
    COUNT(product.productSKU) as num_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
  , UNNEST(hits) AS hits
  , UNNEST(hits.product) as product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
  AND hits.eCommerceAction.action_type = '6'
  AND product.productRevenue IS NOT NULL
  GROUP BY 1
)

SELECT
    pv.*,
    num_addtocart,
    num_purchase,
    ROUND(num_addtocart*100/num_product_view,2) as add_to_cart_rate,
    ROUND(num_purchase*100/num_product_view,2) as purchase_rate
FROM product_view pv
LEFT JOIN add_to_cart a ON pv.month = a.month
LEFT JOIN purchase p ON pv.month = p.month
ORDER BY pv.month;
```
Cách 2: Dùng Case When
```sql
WITH product_data AS(
SELECT
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d',date)) as month,
    COUNT(CASE WHEN eCommerceAction.action_type = '2' THEN product.v2ProductName END) as num_product_view,
    COUNT(CASE WHEN eCommerceAction.action_type = '3' THEN product.v2ProductName END) as num_add_to_cart,
    COUNT(CASE WHEN eCommerceAction.action_type = '6' AND product.productRevenue IS NOT NULL THEN product.v2ProductName END) as num_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
,UNNEST(hits) as hits
,UNNEST (hits.product) as product
WHERE _table_suffix BETWEEN '20170101' AND '20170331'
    AND eCommerceAction.action_type in ('2','3','6')
GROUP BY month
ORDER BY month
)

SELECT
    *,
    ROUND(num_add_to_cart/num_product_view * 100, 2) as add_to_cart_rate,
    ROUND(num_purchase/num_product_view * 100, 2) as purchase_rate
FROM product_data;
```
- Query results
<img width="1101" height="135" alt="image" src="https://github.com/user-attachments/assets/9b0c303b-67f9-42c5-8fdd-e78292d2c9ef" />







    


