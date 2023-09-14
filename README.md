# Explore-Ecommerce-Dataset

## Introduction

In this project, I focused on data exploration and calculation of several metrics in the e-commerce sector. The input data is a publicly available dataset on Google BigQuery. Here are a few characteristics of this dataset:

- Data is stored in tables, each table stores data for one day of the year.
- The entire dataset is a system of tables for each day from January 8, 2016 to August 1, 2017.
- The tables all have the same format as follows:

  ![table](https://github.com/dieppnguyen/Explore-Ecommerce-Dataset/assets/142650906/352f3261-7302-44ac-a963-57f358c45f9c)

- To access the dataset, follow these steps:
  - Log in to your Google Cloud Platform account and create a new project.
  - Navigate to the BigQuery console and select your newly created project.
  - In the navigation panel, select "Add Data" and then "Search a project".
  - Enter the project ID ***"bigquery-public-data.google_analytics_sample.ga_sessions"*** and click "Enter".
  - Click on the ***"ga_sessions_"*** table to open it.
 
## 08 queries in Bigquery based on the Google Analytics dataset I wrote

### Query 1

```WITH sub AS (
  SELECT *, 
    PARSE_DATE('%Y%m%d', date) AS date1
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
  WHERE
  _table_suffix BETWEEN '0101'AND '0331'
)

SELECT 
 FORMAT_DATE('%Y%m', date1)AS month,
  COUNT(totals.visits) AS visits,
  SUM (totals.pageviews) AS pageviews,
  SUM (totals.transactions) AS transactions
FROM
  sub
GROUP BY
  month
ORDER BY
  month;
```

Result table:

![c1](https://github.com/dieppnguyen/Explore-Ecommerce-Dataset/assets/142650906/898668ac-e95a-40ca-ba19-cd2f44a15cbd)

### Query 2

```SELECT 
  trafficSource.source AS source,
  COUNT (totals.bounces) AS total_no_of_bounces,
  COUNT (totals.visits) AS total_visits,
  ROUND((COUNT (totals.bounces) / COUNT (totals.visits) * 100),7) AS bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` 
GROUP BY trafficSource.source
ORDER BY source;
```

Result table:

![c2](https://github.com/dieppnguyen/Explore-Ecommerce-Dataset/assets/142650906/1eea41b5-25cf-4427-93a6-681037e30ac4)

### Query 3

```WITH sub1 AS (
  SELECT *,
  PARSE_DATE('%Y%m%d', date) AS date1
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`
),
sub2 AS (
  SELECT  
    'Month' as time_type, 
    FORMAT_DATE ('%Y%m',date1) AS time,
    trafficSource.source AS source,
    ROUND ((SUM (productRevenue) / 1000000 ),4) AS revenue
  FROM sub1,
  UNNEST (hits) hits,
  UNNEST (hits.product) product
  WHERE productRevenue IS NOT NULL
  GROUP BY trafficSource.source, time
),
sub3 AS (
  SELECT  
    'Month' as time_type, 
    FORMAT_DATE ('%Y%W',date1) AS time,
    trafficSource.source AS source,
    ROUND ((SUM (productRevenue) / 1000000 ),4) AS revenue
  FROM sub1,
  UNNEST (hits) hits,
  UNNEST (hits.product) product
  WHERE productRevenue IS NOT NULL
  GROUP BY trafficSource.source, time
)
SELECT *
FROM sub2
UNION ALL
SELECT *
FROM sub3
ORDER BY time;
```

Result table: 

![c3](https://github.com/dieppnguyen/Explore-Ecommerce-Dataset/assets/142650906/f09a6b04-7a73-46a5-bbb2-dc28888dfb2a)

### Query 4

```SELECT
  FORMAT_DATE ("%Y%m", PARSE_DATE ("%Y%m%d", date)) AS month, 
  ROUND(SUM (CASE WHEN totals.transactions >= 1 AND product.productRevenue IS NOT NULL THEN totals.pageviews END) / COUNT (DISTINCT (CASE
            WHEN totals.transactions >= 1 AND product.productRevenue IS NOT NULL THEN fullVisitorId
        END
          )),7) AS avg_pageviews_purchase,
  ROUND(SUM (CASE WHEN totals.transactions IS NULL AND product.productRevenue IS NULL THEN totals.pageviews END) / COUNT (DISTINCT (CASE
                WHEN totals.transactions IS NULL AND product.productRevenue IS NULL THEN fullVisitorId
            END
              )),7) AS avg_pageviews_non_purchase
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`, UNNEST (hits) hits, UNNEST (hits.product) product
WHERE
  _table_suffix BETWEEN '0601'AND '0731'
GROUP BY
  month
ORDER BY month;
```

Result table: 

![c4](https://github.com/dieppnguyen/Explore-Ecommerce-Dataset/assets/142650906/d5c4ee15-32e2-4fd4-adce-328f420d4716)

### Query 5

```SELECT 
  FORMAT_DATE ("%Y%m", PARSE_DATE ("%Y%m%d", date)) AS month,
  SUM (totals.transactions)/ COUNT (DISTINCT fullVisitorId) AS Avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST (hits) hits,
UNNEST (hits.product) product
WHERE totals.transactions >=1 AND productRevenue IS NOT NULL AND (eCommerceAction.action_type) LIKE '6'
GROUP BY FORMAT_DATE ("%Y%m", PARSE_DATE ("%Y%m%d", date));
```

Result table:

![c5](https://github.com/dieppnguyen/Explore-Ecommerce-Dataset/assets/142650906/c7293c8f-bf29-432d-a848-040e297f2c28)

### Query 6 

```SELECT 

  FORMAT_DATE ("%Y%m", PARSE_DATE ("%Y%m%d", date)) AS month,

  ROUND(SUM (productRevenue) / 1000000 / COUNT(visitId), 2) AS avg_revenue_by_user_per_visit

FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,

UNNEST (hits) hits,

UNNEST (hits.product) product

WHERE totals.transactions IS NOT NULL AND product.productRevenue is not null

GROUP BY FORMAT_DATE ("%Y%m", PARSE_DATE ("%Y%m%d", date));
```

Result table: 

![c6](https://github.com/dieppnguyen/Explore-Ecommerce-Dataset/assets/142650906/120d9933-2625-408c-b826-90c3046f71a8)

### Query 7

```WITH sub AS (
  SELECT DISTINCT fullVisitorId
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST (hits) hits,
UNNEST (hits.product) product
WHERE v2ProductName LIKE "YouTube Men's Vintage Henley" 
AND totals.transactions >=1 
AND productRevenue IS NOT NULL
),
sub2 AS (
SELECT  *
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
UNNEST (hits) hits,
UNNEST (hits.product) product
WHERE totals.transactions >=1 
AND productRevenue IS NOT NULL
)
SELECT 
  v2ProductName AS other_purchased_products,
  SUM(productQuantity) AS quantity
from sub LEFT join sub2 ON sub.fullVisitorId=sub2.fullVisitorId
WHERE totals.transactions >=1 
  AND productRevenue IS NOT NULL
  AND v2ProductName NOT LIKE "YouTube Men's Vintage Henley"
GROUP BY v2ProductName;
```

Result table:

![c7](https://github.com/dieppnguyen/Explore-Ecommerce-Dataset/assets/142650906/7ed102b9-bd34-4cf2-b39b-f5df1ae247bd)

### Query 8

```SELECT

  FORMAT_DATE ("%Y%m", PARSE_DATE ("%Y%m%d", date)) AS month,

  COUNT (CASE WHEN hits.eCommerceAction.action_type LIKE '2' 

    THEN v2ProductName END) AS num_product_view,

  COUNT (CASE WHEN hits.eCommerceAction.action_type LIKE '3' 

    THEN v2ProductName END) AS num_addtocart,

  COUNT (CASE WHEN hits.eCommerceAction.action_type LIKE '6' AND totals.transactions >= 1 AND productRevenue IS NOT NULL 
  
    THEN v2ProductName END) AS num_purchase,

  ROUND ((COUNT (CASE WHEN hits.eCommerceAction.action_type LIKE '3' THEN v2ProductName END)/COUNT (CASE WHEN hits.eCommerceAction.action_type LIKE '2' THEN v2ProductName END) * 100.0),2) AS add_to_cart_rate,

  ROUND ((COUNT (CASE WHEN hits.eCommerceAction.action_type LIKE '6' 
    AND totals.transactions >= 1 AND productRevenue IS NOT NULL
    THEN v2ProductName END)/COUNT (CASE WHEN hits.eCommerceAction.action_type LIKE '2' THEN v2ProductName END)* 100.0), 2) AS purchase_rate

FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,

UNNEST (hits) hits,

UNNEST (hits.product) product

WHERE _table_suffix BETWEEN '0101'AND '0331'

GROUP BY 1 

ORDER BY 1;
```

Result table: 

![c8](https://github.com/dieppnguyen/Explore-Ecommerce-Dataset/assets/142650906/e190b34c-b8db-45dc-8f1b-4bcb81307753)












