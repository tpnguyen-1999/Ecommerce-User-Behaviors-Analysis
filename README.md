# [SQL] Ecommerce Dataset Exploring

## 1. Introduction
Using BigQuery to explore and analyze datasets from an E-commerce company.

## 2. Dataset Access
The E-commerce dataset is stored in the public Google Bigquery dataset. To access this dataset, follow these steps:
- Log in to your Google Cloud Platform account and create a new project.
- Navigate to the BigQuery console and select your newly created project.
- Select "Add Data" in the navigation panel and then "Search a project".
- Enter the project ID **"bigquery-public-data.google_analytics_sample.ga_sessions"** and click "Enter".
- Click on the **"ga_sessions_"** table to open it.

## 3. Explain dataset
https://support.google.com/analytics/answer/3437719?hl=en

|  Field Name                   | Data Type | Description |
| ------------------------------|-----------|-------------|
fullVisitorId                   |	STRING    |	  The unique visitor ID.|
date	                          | STRING    |   The date of the session in YYYYMMDD format.|
totals	                        | RECORD    |	  This section contains aggregate values across the session.|
totals.bounces	                | INTEGER   |	  Total bounces (for convenience). For a bounced session, the value is 1, otherwise it is null.|
totals.hits	                    | INTEGER   | 	Total number of hits within the session.|
totals.pageviews	              | INTEGER   |	  Total number of pageviews within the session.|
totals.visits	                  | INTEGER   |	  The number of sessions (for convenience). This value is 1 for sessions with interaction events. The value is null if there are no interaction events in the session.
totals.transactions	            | INTEGER	  |   Total number of ecommerce transactions within the session.|
trafficSource.source	          | STRING	  |   The source of the traffic source. Could be the name of the search engine, the referring hostname, or a value of the utm_source URL parameter.|
hits	                          | RECORD	  |   This row and nested fields are populated for any and all types of hits.|
hits.eCommerceAction	          | RECORD	  |   This section contains all of the ecommerce hits that occurred during the session. This is a repeated field and has an entry for each hit that was collected.|
hits.product	                  | RECORD	  |   This row and nested fields will be populated for each hit that contains Enhanced Ecommerce PRODUCT data.|
hits.product.productQuantity	  | INTEGER   |	  The quantity of the product purchased.|
hits.product.productRevenue	    | INTEGER   |	  The revenue of the product, expressed as the value passed to Analytics multiplied by 10^6 (e.g., 2.40 would be given as 2400000).|
hits.product.productSKU	        | STRING    |	  Product SKU.|
hits.product.v2ProductName	    | STRING    |	  Product Name.|

## 4. Data Processing & Exploring
**4.1: Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)**
- SQL code
~~~sql
SELECT 
  FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) month,
  SUM(totals.visits) visits,
  SUM(totals.pageviews) pageviews,
  SUM(totals.transactions) transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*` 
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY month
ORDER BY month;
~~~
- Query results
  
|  month  |  visits |  pageviews  |  transactions  |
|---------|---------|-------------|----------------|
|  201701 |  64694  |  257708     |  713           |
|  201702 |  62192  |  233373     |  733           |
|  201703 |  69931  |  259522     |  993           |

**4.2: Bounce rate per traffic source in July 2017**
- SQL code
~~~sql
SELECT 
  trafficSource.source source,
  SUM(totals.visits) total_visits,
  SUM(totals.bounces) total_no_of_bounces,
  ROUND(100.0*SUM(totals.bounces)/SUM(totals.visits),3) bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` 
GROUP BY source
ORDER BY total_visits DESC;
~~~
- Query results
  
|  source                |  total_visits  |  total_no_of_bounces  |  bounce_rate  |
|------------------------|----------------|-----------------------|---------------|
|  google                |  38400         |  19798                |  51.56        |
|  (direct)              |  19891         |  8606                 |  43.27        |
|  youtube.com           |  6351          |  4238                 |  66.73        |  
|  analytics.google.com  |  1972          |  1064                 |  53.96        |
|  Partners              |  1788          |  936                  |  52.35        |
|  m.facebook.com        |  669           |  430                  |  64.28        |
|  google.com            |  368           |  183                  |  49.73        |
|  dfa                   |  302           |  124                  |  41.06        |
|  sites.google.com      |  230           |  97                   |  42.17        |
|  facebook.com          |  191           |  102                  |  53.40        |
|  reddit.com            |  189           |  54                   |  28.57        |
|  qiita.com             |  146           |  72                   |  49.32        |
|  baidu                 |  140           |  84                   |  60           |
|  quora.com             |  140           |  70                   |  50           |

**4.3: Revenue by traffic source by week, by month in June 2017**
- SQL code
~~~sql
SELECT 
  'Month' as time_type,
  FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) time,
  trafficSource.source source,
  ROUND(SUM(productRevenue)/1000000,4) revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
UNNEST(hits) hits,
UNNEST(hits.product) product
WHERE product.productRevenue IS NOT NULL
GROUP BY time, source
UNION ALL
SELECT 
  'Week' as time_type,
  FORMAT_DATE('%Y%W', PARSE_DATE('%Y%m%d', date)) time,
  trafficSource.source source,
  ROUND(SUM(productRevenue)/1000000,4) revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
UNNEST(hits) hits,
UNNEST(hits.product) product
WHERE product.productRevenue IS NOT NULL
GROUP BY time, source
ORDER BY source, time_type;
~~~
- Query results

|  time_type  |  time  |  source           |  revenue    |
|-------------|--------|-------------------|-------------|
|  Month      | 201706 |  (direct)         |  97333.62   |
|  Week       | 201726 |  (direct)         |  14914.81   |
|  Week       | 201725 |  (direct)         |  27295.32   |
|  Week       | 201722 |  (direct)         |  6888.9     |
|  Week       | 201723 |  (direct)         |  17325.68   |
|  Week       | 201724 |  (direct)         |  30908.91   |
|  Month      | 201706 |  bing             |  13.98      |
|  Week       | 201724 |  bing             |  13.98      |
|  Month      | 201706 |  chat.google.com  |  74.03      |
|  Week       | 201723 |  chat.google.com  |  74.03      |
|  Month      | 201706 |  dealspotr.com    |  72.95      |
|  Week       | 201724 |  dealspotr.com    |  72.95      |
|  Month      | 201706 |  dfa              |  8862.23    |
|  Week       | 201724 |  dfa              |  2341.56    |
|  Week       | 201726 |  dfa              |  3704.74    |    
|  Week       | 201722 |  dfa              |  1670.65    |
|  Month      | 201706 |  google           |  18757.18   |
|  Week       | 201722 |  google           |  2119.39    |

**4.4: Average number of pageviews by purchaser type in June, July 2017**
- SQL code
~~~sql
WITH purchase AS(
  SELECT 
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) month,
    ROUND (SUM(totals.pageviews)/COUNT(DISTINCT fullVisitorId),4) avg_pageviews_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST(hits) hits,
  UNNEST(hits.product) product
  WHERE _table_suffix BETWEEN '0601' AND '0731'
    AND product.productRevenue is not null
    AND totals.transactions >=1
  GROUP BY month
  ORDER BY month)

, non_purchase AS(
  SELECT 
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) month,
    ROUND (SUM(totals.pageviews)/COUNT(DISTINCT fullVisitorId),4) avg_pageviews_non_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
  UNNEST(hits) hits,
  UNNEST(hits.product) product
  WHERE _table_suffix BETWEEN '0601' AND '0731'
    AND product.productRevenue is null
    AND totals.transactions is null
  GROUP BY month
  ORDER BY month)

SELECT 
  purchase.month,
  avg_pageviews_purchase,
  avg_pageviews_non_purchase
FROM purchase
LEFT JOIN non_purchase
USING(month)
ORDER BY purchase.month;
~~~
- Query results

|  month |  avg_pageviews_purchase |  avg_pageviews_non_purchase  |
|--------|-------------------------|------------------------------|
| 201706 |  94.0205                |  316.8656                    |
| 201707 |  124.2376               |  334.0566                    |

**4.5: Average number of transactions per user that made a purchase in July 2017**
- SQL code
~~~sql
SELECT 
  FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) month,
  ROUND (SUM(totals.transactions)/COUNT(DISTINCT fullVisitorId),4) Avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST(hits) hits,
UNNEST(hits.product) product
WHERE product.productRevenue is not null
  AND totals.transactions >=1
GROUP BY month;
~~~
- Query results

| month  | Avg_total_transactions_per_user |
|--------|---------------------------------|
| 201707 | 4.1639                          |

**4.6: Average amount of money spent per session. Only include purchaser data in July 2017**
- SQL code
~~~sql
SELECT 
  FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) month,
  ROUND((SUM(product.productRevenue)/ 1000000)/SUM(totals.visits),2) avg_revenue_by_user_per_visit
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST(hits) hits,
UNNEST(hits.product) product
WHERE product.productRevenue is not null
  AND totals.transactions is not null
GROUP BY month;
~~~
- Query results

| month  | avg_revenue_by_user_per_visit   |
|--------|---------------------------------|
| 201707 | 43.86                           |



