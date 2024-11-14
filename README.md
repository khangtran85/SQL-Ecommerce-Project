# SQL-Ecommerce-Project
A SQL project analyzing eCommerce data to uncover insights on traffic, customer behavior, and purchasing patterns. Covers key metrics like visits, transactions, and conversion rates, providing data-driven support for optimizing revenue and user experience.
# Introduction
The bigquery-public-data.google_analytics_sample.ga_sessions_ dataset is a sample dataset provided by Google Analytics that contains information about user sessions on websites tracked by Google Analytics. This dataset includes important fields related to user behavior, enabling the analysis and optimization of marketing strategies and user experience.

This dataset primarily describes user actions during website sessions, including metrics such as total visits, pageviews, session duration, and traffic sources. It helps analysts and marketers gain insights into how users interact with websites, which can improve strategies for user acquisition, engagement, and conversion.

By analyzing this dataset, one can track user behavior over time, evaluate traffic sources (such as search, ads, and referrals), and gain insights into user characteristics like devices used and geographical location. Additionally, key metrics like conversion rates, user interactions, and purchase behavior (e.g., adding products to the cart or completing transactions) can help businesses identify areas for improvement in marketing and business strategies.

This dataset can be queried using Google BigQuery to create detailed reports on user behavior, assess the effectiveness of marketing campaigns, and develop strategies for customer engagement and retention.
# Objectives
**Master SQL Querying Techniques:** Develop proficiency in SQL to extract, filter, and aggregate data effectively from large databases.

**Analyze Data Trends and Insights:** Use SQL to identify key trends, patterns, and insights from business data, supporting data-driven decision-making.

**Optimize Data Queries for Efficiency:** Learn to optimize SQL queries for faster execution, ensuring performance efficiency in handling large datasets.

**Integrate and Visualize Data for Reporting:** Extract and transform data using SQL for integration into reporting tools like Power BI, enabling effective visualization and business analysis.
# Query
- **Query 1:** Write a query to calculate the total visits, pageviews, and transactions for January, February, and March 2017, ordered by month.

*Application in trend analysis over time:* This query helps analyze the trends of key metrics such as visits, pageviews, and transactions by month. It can be used in monthly reports, especially when tracking changes and trends during the first quarter of the year.

*Used for forecasting:* Data from these months can serve as a foundation for forecasts in the coming months, such as predicting traffic and transaction performance.
``` sql
SELECT
  FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) AS month,
  SUM(totals.visits) AS visits,
  SUM(totals.pageviews) AS pageviews,
  SUM(totals.transactions) AS transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _TABLE_SUFFIX BETWEEN '0101' AND '0331'
GROUP BY 1
ORDER BY month ASC;
```
- **Query 2:** Write a query to calculate the bounce rate per traffic source for July 2017, where Bounce_rate is defined as num_bounce divided by total_visit. Order the results by total_visit in descending order.

*Application in optimizing marketing strategies:* The bounce rate can help identify ineffective traffic sources, which can then be optimized by adjusting ad campaigns or website content to reduce bounce rates.

*Analyzing traffic source effectiveness:* Managers can use the results to identify traffic sources that require more focus (those with low bounce rates and large traffic).
``` sql
SELECT
  t1.*,
  ROUND(t1.total_no_of_bounces / t1.total_visits * 100, 3) AS bounce_rate
FROM(
  SELECT
    trafficSource.source AS source,
    SUM(totals.visits) AS total_visits,
    SUM(totals.bounces) AS total_no_of_bounces
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
  GROUP BY 1
) AS t1
ORDER BY t1.total_visits DESC;
```
- **Query 3:** Write a query to calculate the revenue by traffic source, broken down by week and by month for June 2017.

*Application in evaluating advertising campaign effectiveness:* If revenue is directly linked to marketing campaigns, this query helps assess the performance of each campaign across different traffic sources.

*Improving sales strategy over time:* Weekly and monthly analysis allows adjustments to the sales strategy for upcoming months, particularly when there are seasonal changes or shifts in campaigns.
``` sql
SELECT
  "Month" AS time_type,
  FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) AS time,
  trafficSource.source AS source,
  SUM(product.productRevenue) / 1000000 AS revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
  UNNEST(hits) AS hits,
  UNNEST(hits.product) AS product
WHERE product.productRevenue IS NOT NULL
GROUP BY 2, 3
UNION ALL
SELECT
  "Week" AS time_type,
  FORMAT_DATE("%Y%W", PARSE_DATE("%Y%m%d", date)) AS time,
  trafficSource.source AS source,
  SUM(product.productRevenue) / 1000000 AS revenue
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
  UNNEST(hits) AS hits,
  UNNEST(hits.product) AS product
WHERE product.productRevenue IS NOT NULL
GROUP BY 2, 3
ORDER BY revenue DESC;
```
- **Query 4:** Write a query to calculate the average number of pageviews by purchaser type (comparing purchasers and non-purchasers) for June and July 2017.

*Comparison between buyers and non-buyers:* Segmenting customers into buyers and non-buyers helps identify which customer group engages more with the products, optimizing the user experience.

*Application in improving user experience:* Non-buyers could be targeted for marketing campaigns to bring them back to the website for transactions.
``` sql
WITH t1 AS (
  SELECT
    FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) AS month,
    SUM(
      CASE
        WHEN totals.transactions >= 1 AND productRevenue IS NOT NULL THEN totals.pageviews
        ELSE NULL
      END
    ) AS total_pageview_purchase,
    COUNT(DISTINCT
      CASE
        WHEN totals.transactions >= 1 AND productRevenue IS NOT NULL THEN fullVisitorId
        ELSE NULL
      END
    ) AS num_visitor_purchase,
    SUM(
      CASE
        WHEN totals.transactions IS NULL AND productRevenue IS NULL THEN totals.pageviews
        ELSE NULL
      END
    ) AS total_pageview_non_purchase,
    COUNT(DISTINCT
      CASE
        WHEN totals.transactions IS NULL AND productRevenue IS NULL THEN fullVisitorId
        ELSE NULL
      END
    ) AS num_visitor_non_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) AS hits,
    UNNEST(hits.product) AS product
  WHERE _TABLE_SUFFIX BETWEEN "0601" AND "0731"
  GROUP BY 1
)
SELECT
  t1.month,
  t1.total_pageview_purchase / t1.num_visitor_purchase AS avg_pageviews_purchase,
  t1.total_pageview_non_purchase / t1.num_visitor_non_purchase AS avg_pageviews_non_purchase
FROM t1;
```
or
``` sql
WITH avg_pageviews_purchase AS (
  SELECT
    FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) AS month,
    SUM(totals.pageviews) / COUNT(DISTINCT fullVisitorId) AS avg_pageviews_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) AS hits,
    UNNEST(hits.product) AS product
  WHERE
    _TABLE_SUFFIX BETWEEN "0601" AND "0731"
    AND totals.transactions >= 1
    AND productRevenue IS NOT NULL
  GROUP BY 1
),
avg_pageviews_non_purchase AS (
  SELECT
    FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) AS month,
    SUM(totals.pageviews) / COUNT(DISTINCT fullVisitorId) AS avg_pageviews_non_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) AS hits,
    UNNEST(hits.product) AS product
  WHERE
    _TABLE_SUFFIX BETWEEN "0601" AND "0731"
    AND totals.transactions IS NULL
    AND productRevenue IS NULL
  GROUP BY 1
)
SELECT
  t1.month,
  t1.avg_pageviews_purchase,
  t2.avg_pageviews_non_purchase
FROM avg_pageviews_purchase AS t1
INNER JOIN avg_pageviews_non_purchase AS t2
  ON t1.month = t2.month
ORDER BY t1.month ASC;
```
- **Query 5:** Write a query to calculate the average number of transactions per user who made a purchase in July 2017

*Application in shopping behavior analysis:* Helps measure customer loyalty and shopping patterns. Users who make multiple transactions can be targeted for promotional offers or discounts.

*Identifying valuable customers:* Analyzing transactions per buyer helps identify high-value customers, allowing for the development of targeted sales strategies.
``` sql
SELECT
  LEFT(date, 6) AS month,
  SUM(totals.transactions) / COUNT(DISTINCT fullVisitorId) AS Avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST(hits) AS hits,
  UNNEST(hits.product) AS product
WHERE
  totals.transactions >= 1
  AND productRevenue IS NOT NULL
GROUP BY 1;
```
- **Query 6:** Write a query to calculate the average amount of money spent per session, including only data from purchasers in July 2017.

*Application in calculating average order value:* Helps understand the average spending of customers per transaction, informing pricing strategies and promotions.

*Optimizing marketing costs:* If advertising costs are high, analyzing spend per session helps assess whether marketing campaigns are yielding a good return on investment (ROI).
``` sql
SELECT
  FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) AS month,
  SUM(productRevenue) / SUM(totals.visits) / power(10, 6) AS avg_revenue_by_user_per_visit
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST(hits) AS hits,
  UNNEST(hits.product) AS product
WHERE
  totals.transactions IS NOT NULL
  AND productRevenue IS NOT NULL
GROUP BY 1;
```
- **Query 7:** Write a query to find other products purchased by customers who bought the product 'YouTube Men's Vintage Henley' in July 2017. The output should display the product name and the quantity ordered.

*Application in cross-selling strategies:* Products can be recommended to customers who purchased the "YouTube Men's Vintage Henley" to increase revenue from related items.

*Building product bundles:* Use this data to create attractive product bundles, such as buy one, get one free offers for products frequently bought together.
``` sql
WITH list_visitorid_buy_YoutubeMensVintageHenley AS (
  SELECT DISTINCT
    fullVisitorId
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
    UNNEST(hits) AS hits,
    UNNEST(hits.product) AS product
  WHERE
    productRevenue IS NOT NULL
    AND v2ProductName LIKE "YouTube Men's Vintage Henley"
)
SELECT
  v2ProductName AS other_purchased_products,
  SUM(productQuantity) AS quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` AS t2,
  UNNEST(hits) AS hits,
  UNNEST(hits.product) AS product
INNER JOIN list_visitorid_buy_YoutubeMensVintageHenley AS t1
  ON t1.fullVisitorId = t2.fullVisitorId
WHERE
  productRevenue IS NOT NULL
  AND v2ProductName NOT LIKE "YouTube Men's Vintage Henley"
GROUP BY 1
ORDER BY quantity DESC;
```
- **Query 8:** Write a query to calculate the cohort map from product view to add-to-cart to purchase for January, February, and March 2017. For example, 100% product view, then 40% add_to_cart, and 10% purchase. The add_to_cart_rate should be calculated as the number of products added to cart divided by the number of product views. The purchase_rate should be calculated as the number of products purchased divided by the number of product views. The output should be calculated at the product level.

*Application in analyzing consumer behavior:* Data from this conversion rate can provide insights into consumer behavior on the website, helping improve the user experience to boost conversion rates from viewing to purchasing.

*Optimizing the sales process:* Reducing the rate of users who view products but don’t add them to the cart or make a purchase can help optimize the sales process and increase conversion rates.
``` sql
WITH num_visitor_in_236_actiontype AS (
  SELECT
    FORMAT_DATE("%Y%m", PARSE_DATE("%Y%m%d", date)) AS month,
    COUNT(
      CASE
        WHEN hits.eCommerceAction.action_type = '2' THEN fullVisitorId
        ELSE NULL
      END
    ) AS num_product_view,
    COUNT(
      CASE
        WHEN hits.eCommerceAction.action_type = '3' THEN fullVisitorId
        ELSE NULL
      END
    ) AS num_addtocart,
    COUNT(
      CASE
        WHEN hits.eCommerceAction.action_type = '6' AND productRevenue IS NOT NULL THEN fullVisitorId
        ELSE NULL
      END
    ) AS num_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) AS hits,
    UNNEST(hits.product) AS product
  WHERE
    _TABLE_SUFFIX BETWEEN '0101' AND '0331'
  GROUP BY 1
)
SELECT
  t1.*,
  ROUND(t1.num_addtocart / t1.num_product_view * 100, 2) AS add_to_cart_rate,
  ROUND(t1.num_purchase / t1.num_product_view * 100, 2) AS purchase_rate
FROM num_visitor_in_236_actiontype AS t1
ORDER BY month ASC;
```
# Recommendation
These queries help track key metrics like visits, revenue, conversion rates, and customer behavior. By analyzing this data, businesses can identify trends and factors influencing performance. For example, bounce rates and conversion rates can help optimize the user experience, boosting marketing campaigns and improving resource allocation.

The queries are valuable for assessing the effectiveness of marketing campaigns. By analyzing traffic sources, bounce rates, and purchase behaviors, marketers can refine their strategies, optimize ad spending, and drive revenue growth. Additionally, identifying popular products can help design targeted cross-selling and up-selling strategies.

Data from these queries also support long-term strategic decisions. Understanding customer behavior, such as purchase frequency or average order value, helps businesses develop loyalty programs and optimize products and services to meet market demands. This data can also forecast future trends and guide decisions on expansion or product adjustments.

To enhance decision-making, it's recommended to extract and visualize the results of these queries in Power BI. By creating dynamic dashboards and reports, businesses can gain real-time insights into key metrics like visits, transactions, and revenue, enabling quick analysis and actionable insights. Visualizations like trend graphs, heatmaps, and pie charts can help stakeholders better understand the data and make informed decisions faster, improving both marketing and operational strategies.
