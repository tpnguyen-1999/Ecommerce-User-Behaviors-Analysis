# [SQL] Explore User Behaviors of an E-commerce Company

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

There is a positive trend from February to March in all three metrics. February appears to be the weakest month in terms of site visits, which could be worth investigating for potential
causes (e.g., seasonal trends, marketing activities, etc.,)

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

The result presents a summary of website traffic from different sources and key metrics used to assess user engagement and behaviours. It focuses on four main elements: source,
total_visits, total_no_bounces and bounce_rate. 

Google received the highest website traffic, 38,400 visits in the first row, with 19,798 were single-page visits (bounces), leading to a bounce rate of approximately 51.56%. The 
second place is (direct) with 19,891 visits, of these 8,606 bounces, resulting to a bounce rate of 43.27%. 

This suggests that the majority of users from this source left the website without navigating to other pages. A high bounce rate indicates that users from this source might not find
the content they are looking for or the landing page lacks of engagement to encourage further exploration. Therefore, it is essential to deal with high bounce rate by analyzing the context of the source and user behaviors in order to find effective improvement strategies. 

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
|  Week       | 201724 |  (direct)         |  30908.91   |
|  Week       | 201725 |  (direct)         |  27295.32   |
|  Week       | 201722 |  (direct)         |  6888.9     |
|  Week       | 201723 |  (direct)         |  17325.68   |
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

This table shows revenue data with various attributes, including time type, time period, souce and the amount of revenue. Revenue comes from different sources, including direct website visits, search traffic and referrals from other websites. 

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
  GROUP BY month)

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
  GROUP BY month)

SELECT 
  purchase.month,
  avg_pageviews_purchase,
  avg_pageviews_non_purchase
FROM purchase
FULL JOIN non_purchase
USING(month)
ORDER BY purchase.month;
~~~
- Query results

|  month |  avg_pageviews_purchase |  avg_pageviews_non_purchase  |
|--------|-------------------------|------------------------------|
| 201706 |  94.0205                |  316.8656                    |
| 201707 |  124.2376               |  334.0566                    |

The data reveals a significant difference in average page views between users who made a purchase and those who did not. On average, users who did not purchase tend to view more pages on the website than those who made a purchase. For instance, in June 2017, the average pageviews for non-purchasers (316.87) were sgnificantly higher compared to purchasers (94.02).

While user engagement is high, there might be barriers preventing conversions. It is important to find effective strategies in order to encourage these high-pageview users to make purchase, such as imporving website navigation, enhancing product page, reducing checkout process, etc.  

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

The table presents the average total transactions per user in July, indicating that the typical user made approximately 4.16 transactions during July 2017. This data could help analyze user behavior, track user engagement with the platform, and evaluate the impact of marketing campaigns or promotions during that time.

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

The average revenue per user per visit for July 2017 is 43.86. This suggests that, on average, each user spent approximately 44 during that month. This could be an important metric for businesses to measure user engagement and activity.

**4.7: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.**
- SQL code
~~~sql
WITH buyer_list AS(
    SELECT
        distinct fullVisitorId  
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    , UNNEST(hits) AS hits
    , UNNEST(hits.product) as product
    WHERE product.v2ProductName = "YouTube Men's Vintage Henley"
    AND totals.transactions>=1
    AND product.productRevenue is not null)

SELECT
  product.v2ProductName AS other_purchased_products,
  SUM(product.productQuantity) AS quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
, UNNEST(hits) AS hits
, UNNEST(hits.product) as product
JOIN buyer_list using(fullVisitorId)
WHERE product.v2ProductName != "YouTube Men's Vintage Henley"
 AND product.productRevenue is not null
GROUP BY other_purchased_products
ORDER BY quantity DESC;  
~~~
- Query results

| other_purchased_products  | quantity   |
|---------------------------|------------|
|Google Sunglasses                     | 20|
|Google Women's Vintage Hero Tee Black | 7 |
|SPF-15 Slim & Slender Lip Balm        | 6 |
|Google Women's Short Sleeve Hero Tee Red Heather| 4|
|YouTube Men's Fleece Hoodie Black| 3|
|Google Men's Short Sleeve Badge Tee Charcoal| 3|
|Android Men's Vintage Henley| 2|
|22 oz YouTube Bottle Infuser| 2|
|Google Men's Short Sleeve Hero Tee Charcoal| 2|
|Android Women's Fleece Hoodie| 2|
|Google Doodle Decal| 2|
|YouTube Twill Cap| 2|

Overall, the data offers valuable insights into customer preferences, product popularity, and opportunities for marketing and merchandising strategies. A deeper analysis of historical data, combined with customer demographics, could provide a more comprehensive understanding of these trends.

**4.8: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017.**
- SQL code
~~~sql
with product_data as(
SELECT
    FORMAT_DATE('%Y%m', parse_date('%Y%m%d',date)) AS month,
    COUNT(CASE WHEN eCommerceAction.action_type = '2' THEN product.v2ProductName END) AS num_product_view,
    COUNT(CASE WHEN eCommerceAction.action_type = '3' THEN product.v2ProductName END) AS num_add_to_cart,
    COUNT(CASE WHEN eCommerceAction.action_type = '6' and product.productRevenue is not null THEN product.v2ProductName END) AS num_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
,UNNEST(hits) as hits
,UNNEST (hits.product) as product
WHERE _table_suffix between '20170101' AND '20170331'
 AND eCommerceAction.action_type IN ('2','3','6')
GROUP BY month
ORDER BY month)

SELECT
    *,
    ROUND(num_add_to_cart/num_product_view * 100, 2) AS add_to_cart_rate,
    ROUND(num_purchase/num_product_view * 100, 2) AS purchase_rate
FROM product_data;
~~~
- Query results

| month| num_product_view| num_add_to_cart| num_purchase| add_to_cart_rate| purchase_rate|
|------|-----------------|----------------|-------------|-----------------|--------------|
|201701| 25787| 7342| 2143| 28.47| 8.31|
|201702| 21489| 7360| 2060| 34.25| 9.59|
|201703| 23549| 8782| 2977| 37.29| 12.64|

The table shows five different metrics and rates related to user behaviours from January to March 2017.

In general, the number of product views increases gradually during this period. Similarly, the add-to-cart rate and purchase rate showed an upward trend, indicating enhanced user engagement and conversion.

## 5. Conclusion
- My analysis of the eCommerce dataset using SQL on Google BigQuery, based on the Google Analytics dataset, has uncovered several interesting insights.
- By analyzing the eCommerce dataset, I gained valuable understand about customer behaviors through bounce rate, transactions, revenue, visits and purchase.
- To further explore key insights and trends, the next step is to visualize the data using software such as Power BI, Tableau, or similar tools.
- Overall, this project has illustrated the effectiveness of SQL and big data tools like Google BigQuery in extracting valuable insights from large datasets.

