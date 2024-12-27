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
### Giải thích:
- **Khối mã (code block)**: Câu lệnh SQL được bao trong một khối mã, giúp mã dễ đọc và rõ ràng.
- **Văn bản giải thích**: Sau khối mã, bạn có thể viết văn bản giải thích về kết quả của câu lệnh SQL và mục đích sử dụng dữ liệu, tất cả dưới dạng văn bản bình thường.


