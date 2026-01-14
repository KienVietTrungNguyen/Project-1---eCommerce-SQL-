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
# Query 01: Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
- SQL code
<img width="770" height="181" alt="Image" src="https://github.com/user-attachments/assets/73020b99-d796-4e93-9c4e-09cf4751c8bc" />

- Query results
<img width="792" height="135" alt="Image" src="https://github.com/user-attachments/assets/597d7389-d8a2-4372-be8c-8c3507f27a74" />

March 2017 shows a significant improvement in all metrics ( visits, pageviews, and transactions) compared to January and February
## Query 02: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)
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



