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




