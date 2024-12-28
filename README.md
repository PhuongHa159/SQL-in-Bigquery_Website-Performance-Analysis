# SQL-in-Bigquery_Website-Performance-Analysis
## I. Introduction
In this project, I applied advanced SQL techniques, including sliding window and aggregation queries, using Google BigQuery to analyze e-commerce data. I evaluated product performance, sales trends, discount strategies, customer retention, and inventory management. These insights supported the Marketing and Sales teams in making strategic, data-driven decisions to improve business outcomes.

## II. Dataset Access
The e-commerce dataset is stored in a public Google BigQuery dataset. To access the dataset, follow these steps:

* Log in to your Google Cloud Platform account and create a new project.
* In the navigation panel, search for the "ga_session" dataset and open it in new tab
## III. Exploring the Dataset
### Query 01: Calculate total visit, pageview, transaction for Jan, Feb and March 2017 (order by month)
```sql
SELECT DISTINCT(FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date))) AS Month,
  SUM(totals.visits) AS Visits,
  SUM(totals.pageviews) AS Pageviews ,
  SUM(totals.transactions) AS Transactions 
 FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
 WHERE _table_suffix BETWEEN '0101' AND '0331'
 GROUP BY month
  ORDER BY month;
```
| Month  | Visits | Pageviews | Transactions |
|--------|--------|-----------|--------------|
| 201701 | 64694  | 257708    | 713          |
| 201702 | 62192  | 233373    | 733          |
| 201703 | 69931  | 259522    | 993          |

In Q1 2017, March saw a significant spike across all aspects such as visits, pageviews, and transactions, indicating an improvement in conversion rates or that conversion rates were influenced by seasonal effects.

### Query 02: Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit)
```sql
SELECT trafficSource.source,
  SUM(totals.visits) AS total_visits,
  SUM(totals.bounces) AS total_no_of_bounces,
  FORMAT('%.3f',100*(SUM(totals.bounces)/SUM(totals.visits))) AS bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
GROUP BY trafficSource.source
ORDER BY total_visits DESC;
```
| Row | source                | total_visits | total_no_of_bounces | bounce_rate |
|-----|-----------------------|--------------|---------------------|-------------|
| 1   | google                | 38400        | 19798               | 51.557      |
| 2   | (direct)              | 19891        | 8606                | 43.266      |
| 3   | youtube.com           | 6351         | 4238                | 66.730      |
| 4   | analytics.google.com  | 1972         | 1064                | 53.955      |
| 5   | Partners              | 1788         | 936                 | 52.349      |
| 6   | m.facebook.com        | 669          | 430                 | 64.275      |
| 7   | google.com            | 368          | 183                 | 49.728      |
| 8   | dfa                   | 302          | 124                 | 41.060      |
| 9   | sites.google.com      | 230          | 97                  | 42.174      |
| 10  | facebook.com          | 191          | 102                 | 53.403      |

Google has the highest traffic, but its bounce rate is also high, possibly because users perform quick queries and leave immediately afterward. YouTube has the highest bounce rate and low traffic, which may be due to users accessing other platforms and only watching one video on YouTube without further exploration—leading to a higher bounce rate. The direct channel has the best engagement rate, as it achieves high traffic and a low bounce rate.

**Query 3: Revenue by traffic source by week, by month in June 2017**

```
WITH month_data AS( -- tính revenue theo month
SELECT 'Month' as time_type,
  FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS time,
  trafficSource.source,
  FORMAT('%.4f',SUM(product.productRevenue)/1000000) AS revenue,
FROM 
  `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
  UNNEST(hits) AS hits,
  UNNEST(hits.product) AS product
  WHERE product.productRevenue IS NOT NULL
  GROUP BY time,trafficSource.source),

 week_data AS( -- tính revenue theo week
  SELECT 'Week' as time_type,
  FORMAT_DATE('%Y%W', PARSE_DATE('%Y%m%d', date)) AS time,
  trafficSource.source,
  FORMAT('%.4f',SUM(product.productRevenue)/1000000) AS revenue,
FROM 
  `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
  UNNEST(hits) AS hits,
  UNNEST(hits.product) AS product
  WHERE product.productRevenue IS NOT NULL
  GROUP BY time,trafficSource.source)

  SELECT *
    FROM month_data
  UNION ALL 
  SELECT *
    FROM week_data
  ORDER BY source,time_type,time;
```

| time_type | time   | source   | revenue    |
| --------- | ------ | -------- | ---------  |
| Month     | 201706 | (direct) | 97333.6197 |
| Week      | 201722 | (direct) | 6888.9000  |
| Week      | 201723 | (direct) | 17325.6799 |
| Week      | 201724 | (direct) | 30908.9099 |
| Week      | 201725 | (direct) | 27295.3199 |
| Week      | 201726 | (direct) | 14914.8100 |
| Month     | 201706 | bing     | 13.9800    |
| Week      | 201724 | bing     | 13.9800    |
| Month     | 201706 | chat.google.com | 74.0300 |
| Week      | 201723 | chat.google.com | 74.0300 |

The **Direct source** had the highest revenue in June 2017, possibly because the business had built a loyal customer base. These customers were already familiar with the brand and accessed the website directly without relying on intermediary channels. This often leads to a **higher conversion rate**, significantly contributing to increased revenue from the Direct source.

**Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.**

WITH avg_pageviews_purchase AS(-- tính tỷ lệ purchase
SELECT DISTINCT(FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date))) AS month
,ROUND(SUM(totals.pageviews)/COUNT(DISTINCT fullVisitorId),8) AS avg_pageviews_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
UNNEST(hits)hits,
UNNEST(hits.product)product
WHERE _table_suffix BETWEEN '0601' AND '0731'
AND totals.transactions >=1
AND product.productRevenue IS NOT NULL
GROUP BY month
ORDER BY month),

avg_pageviews_no_purchase AS( -- tính tỷ lệ no purchase
SELECT DISTINCT(FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date))) AS month
,ROUND(SUM(totals.pageviews)/COUNT(DISTINCT fullVisitorId),7) AS avg_pageviews_no_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
UNNEST(hits)hits,
UNNEST(hits.product)product
WHERE _table_suffix BETWEEN '0601' AND '0731'
AND totals.transactions IS NULL
AND product.productRevenue IS NULL
GROUP BY month
ORDER BY month)

SELECT a.month
 ,a.avg_pageviews_purchase
 ,a_n.avg_pageviews_no_purchase
FROM avg_pageviews_purchase AS a
FULL JOIN avg_pageviews_no_purchase AS a_n
ON a.month = a_n.month;

| month | avg_pageviews_purchase | avg_pageviews_no_purchase |
| --- | --- | --- |
| 201706 | 94.02050114 | 316.86558850 |
| 201707 | 124.23755187 | 334.05655980 |

Non-purchasers have an average pageview count about 3 times higher than purchasers. Both groups showed an increase in pageviews, with purchasers experiencing a more significant increase (around 31%), indicating the effectiveness of efforts to improve and enhance the quality of the page.

**Query 05: Average number of transactions per user that made a purchase in July 2017**

SELECT FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date)) AS month
,ROUND(SUM(totals.transactions)/COUNT (DISTINCT fullVisitorId),9) AS Avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST(hits)hits,
UNNEST(hits.product)product
WHERE totals.transactions >=1
AND productRevenue IS NOT NULL
GROUP BY month;

|Row| month | Avg_total_transactions_per_user |  
| --- | --- | --- |
| 1 | 201707 | 4.163900415 |

A user who made purchases in July 2017 completed approximately 4.16 transactions on average. This indicates a moderate level of repeat purchasing behavior, which needs improvement in the future.

**Query 06: Average amount of money spent per session. Only include purchaser data in July 2017**

```
 SELECT FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date)) AS month
,ROUND((SUM(productRevenue)/COUNT(CASE WHEN totals.visits =1 THEN totals.visits END))/1000000,2) AS avg_revenue_by_user_per_visit
 FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST(hits)hits,
  UNNEST(hits.product)product
  WHERE totals.transactions IS NOT NULL
  AND productRevenue IS NOT NULL
 GROUP BY month;
```

|Row| month | avg_revenue_by_user_per_visit |  
| --- | --- | --- |
| 1 | 201707 | 43.86 |

In July 2017, users spent an average of $43.86 per transaction session. This metric helps businesses gain insights into customer spending levels to develop suitable pricing strategies that attract customers.

**Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. Output should show product name and the quantity was ordered.**

WITH raw_data AS( -- tìm ds người mua YouTube Men's Vintage Henley
SELECT DISTINCT fullVisitorId AS user
,v2ProductName AS product_name
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST(hits)hits,
UNNEST(hits.product)product
WHERE v2ProductName="YouTube Men's Vintage Henley"
AND productRevenue IS NOT NULL)

SELECT DISTINCT v2ProductName AS other_purchased_products
,SUM(productQuantity) AS quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST(hits)hits,
UNNEST(hits.product)product
INNER JOIN raw_data AS b
ON fullVisitorId =b.user
WHERE productRevenue IS NOT NULL
AND v2ProductName <>"YouTube Men's Vintage Henley"
GROUP BY v2ProductName
ORDER BY quantity DESC;

| other_purchased_products | quantity |
| --- | --- |
| Google Sunglasses | 20 |
| Google Women's Vintage Hero Tee Black | 7 |
| SPF-15 Slim & Slender Lip Balm | 6 |
| Google Women's Short Sleeve Hero Tee Red
  Heather | 4 |
| YouTube Men's Fleece Hoodie Black | 3 |
| Google Men's Short Sleeve Badge Tee
  Charcoal | 3 |
| YouTube Twill Cap | 2 |
| Recycled Mouse Pad | 2 |
| Red Shine 15 oz Mug | 2 |
| Google Doodle Decal | 2 |

Customers who purchased the YouTube Men's Vintage Henley also preferred Google-branded items, especially sunglasses. The business could bundle these two products together to boost revenue growth.

**Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017**

WITH raw_data AS( -- tìm num_product_view,num_addtocart,num_purchase
SELECT DISTINCT FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date)) AS month
,COUNT(CASE WHEN eCommerceAction.action_type ='2' THEN 1 END) AS num_product_view
,COUNT(CASE WHEN eCommerceAction.action_type ='3' THEN 1 END) AS num_addtocart
,COUNT(CASE WHEN eCommerceAction.action_type ='6' AND productRevenue IS NOT NULL THEN 1 END) AS num_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
,UNNEST(hits)hits
,UNNEST(hits.product)product
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY month
ORDER BY month)

SELECT month
,num_product_view
,num_addtocart
,num_purchase
,ROUND(100*(num_addtocart/num_product_view),2) AS add_to_cart_rate
,ROUND(100*(num_purchase/num_product_view),2) AS purchase_rate
FROM raw_data

|Row| **month** | **num_product_view** | **num_addtocart** | **num_purchase** | **add_to_cart_rate** | **purchase_rate** | 
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 201701 | 25787 | 7342 | 2143 | 28.47 | 8.31 |
| 2 | 201702 | 21489 | 7360 | 2060 | 34.25 | 9.59 |
| 3 | 201703 | 23549 | 8782 | 2977 | 37.29 | 12.64 |

The conversion rate from product views to purchases improved consistently from January to March 2017. In March, the add-to-cart rate was 37.29%, an increase of 8.82%, and the purchase rate was 12.64%, up by 4.33% compared to January. This trend indicates effectiveness in converting viewers into buyers, possibly due to improved marketing strategies or optimized customer experience.

## IV. Conclusion

Overall, this project utilized Google BigQuery to analyze user interaction data on the business's website over a specific period. From this, valuable insights into business performance, customer shopping behavior, and sales trends were extracted. The analysis showed improvements in conversion rates, an increase in cross-selling opportunities among products, and positive changes in sales trends over time. By applying big data analytics, the project provided the business with a comprehensive view of its performance, enabling the development of strategies to improve products, adjust pricing strategies, and enhance customer retention, ultimately driving performance and growth in the competitive e-commerce environment.
