# SQL-in-Bigquery_Website-Performance-Analysis
## I. Introduction
In this project, I applied advanced SQL techniques, including sliding window and aggregation queries, using Google BigQuery to analyze e-commerce data. I evaluated product performance, sales trends, discount strategies, customer retention, and inventory management. These insights supported the Marketing and Sales teams in making strategic, data-driven decisions to improve business outcomes.

## II. Dataset Access
The e-commerce dataset is stored in a public Google BigQuery dataset. To access the dataset, follow these steps:

* Log in to your Google Cloud Platform account and create a new project.
* In the navigation panel, search for the "ga_session" dataset and open it in new tab
## III. Key Focus Areas
Product Performance Analysis: Evaluated subcategory performance through sales metrics and year-over-year growth rates.
Geographic Sales Patterns: Identified top-performing territories by order quantity across multiple years.
Discount Strategy Assessment: Analyzed seasonal discount costs across product subcategories.
Customer Retention Analysis: Calculated retention rates for successfully shipped orders using cohort analysis.
Inventory Management: Examined stock level trends, month-over-month changes, and stock-to-sales ratios.
Order Status Monitoring: Quantified pending orders and their value to assess fulfillment efficiency.
## IV. Exploring the Dataset
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


