# SQL_E-Commerce_Dataset
# DATASET
https://support.google.com/analytics/answer/3437719?hl=en
<img width="924" height="712" alt="Image" src="https://github.com/user-attachments/assets/e962f770-00d9-4764-b1d0-55a2832677dd" />
## Query 01: Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
SELECT 
    FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date)) AS month
    ,SUM(totals.visits) AS visits
    ,SUM(totals.pageviews) AS pageviews
    ,SUM(totals.transactions) AS transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY month
ORDER BY month;
<img width="343" height="118" alt="Image" src="https://github.com/user-attachments/assets/cdf2b282-41c7-4c15-882a-2b9404cfc146" />
March 2017 shows a significant improvement in all metrics ( visits, pageviews, and transactions) compared to January and February
## Query 02: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)





