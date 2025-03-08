# TABLE OF CONTENTS
**1.ACKNOWLEDGEMENT**

**2.PROJECT OVERVIEW**

**3.DATA SOURCES**

**4.DATA ANALYSIS**

**5.CONCLUSION**

# ACKNOWLEDGEMENT

This project was prepared with the assistance of the course [Advanced SQL: MySQL for Ecommerce Data Analysis](https://www.udemy.com/course/advanced-sql-mysql-for-analytics-business-intelligence/) on Udemy.

# PROJECT OVERVIEW 

Situation :- Cindy, CEO, is on the verge of securing Maven Fuzzy Factory’s next round of funding, and she needs  help to create a compelling story for investors. I’ll need to gather the relevant data and assist her in crafting a narrative about our data-driven company and the rapid growth we’ve been achieving.

Objectives :- I extract and analyze traffic and website performance data to create a compelling growth story that the CEO can present. I dive into the marketing channel activities and website improvements that have contributed to our success so far, using this opportunity to showcase my analytical skills to investors. As an Analyst, my job begins with extracting and analyzing the data. The next equally important step is effectively communicating the story to the stakeholders.

# DATA SOURCES
This project analyzes eCommerce data using the Maven Fuzzy Factory database, which consists of six interrelated tables. The dataset includes comprehensive information on website activity, products, and orders and refunds. By leveraging MySQL, the project explores how customers access and interact with the website, evaluates landing page performance and conversion rates, and investigates product-level sales trends. The insights gained aim to provide a deeper understanding of user behavior, product performance, and refund patterns, contributing to data-driven decision-making in an eCommerce context. Please find below ER diagram of database :-

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis/blob/main/ecommerce_data_source.JPG?raw=true)

# DATA ANALYSIS

QUESTION 1 :-  First, I’d like to show our volume growth. Can you pull overall session and order volume, trended by quarter for the life of the business? Since the most recent quarter is incomplete, you can decide how to handle it.

```sql
SELECT 
	YEAR(website_sessions.created_at) AS yr,
	QUARTER(website_sessions.created_at) AS qtr, 
	COUNT(DISTINCT website_sessions.website_session_id) AS sessions, 
    COUNT(DISTINCT orders.order_id) AS orders
FROM website_sessions 
	LEFT JOIN orders
		ON website_sessions.website_session_id = orders.website_session_id
GROUP BY 1,2
ORDER BY 1,2
;
```

Result :-

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis_Project_2/blob/main/solution1.JPG)

Conclusion for QUESTION 1 :-

The data shows consistent growth in both sessions and orders from 2012 to 2015, with noticeable seasonal peaks in the fourth quarter. This steady upward trend reflects the company’s successful scaling and growing demand over the years.

QUESTION 2 :- Next, let’s showcase all of our efficiency improvements. I would love to show quarterly figures since we launched, for session-to-order conversion rate, revenue per order, and revenue per session. 

```sql
SELECT 
	YEAR(website_sessions.created_at) AS yr,
	QUARTER(website_sessions.created_at) AS qtr, 
	COUNT(DISTINCT orders.order_id)/COUNT(DISTINCT website_sessions.website_session_id) AS session_to_order_conv_rate, 
    SUM(price_usd)/COUNT(DISTINCT orders.order_id) AS revenue_per_order, 
    SUM(price_usd)/COUNT(DISTINCT website_sessions.website_session_id) AS revenue_per_session
FROM website_sessions 
	LEFT JOIN orders
		ON website_sessions.website_session_id = orders.website_session_id
GROUP BY 1,2
ORDER BY 1,2
;
```
Result :-

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis_Project_2/blob/main/solution2.JPG)

Conclusion for QUESTION 2 :-

The data shows a clear improvement in efficiency over time, with the session-to-order conversion rate steadily increasing from 0.0319 in Q1 2012 to 0.0844 in Q1 2015. Revenue per order and revenue per session also saw consistent growth, reflecting better monetization and overall performance improvements across the quarters.


QUESTION 3:- I’d like to show how we’ve grown specific channels. Could you pull a quarterly view of orders from Gsearch nonbrand, Bsearch nonbrand, brand search overall, organic search, and direct type-in?

```sql
SELECT 
	YEAR(website_sessions.created_at) AS yr,
	QUARTER(website_sessions.created_at) AS qtr, 
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' AND utm_campaign = 'nonbrand' THEN orders.order_id ELSE NULL END) AS gsearch_nonbrand_orders, 
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' AND utm_campaign = 'nonbrand' THEN orders.order_id ELSE NULL END) AS bsearch_nonbrand_orders, 
    COUNT(DISTINCT CASE WHEN utm_campaign = 'brand' THEN orders.order_id ELSE NULL END) AS brand_search_orders,
    COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NOT NULL THEN orders.order_id ELSE NULL END) AS organic_search_orders,
    COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NULL THEN orders.order_id ELSE NULL END) AS direct_type_in_orders
    
FROM website_sessions 
	LEFT JOIN orders
		ON website_sessions.website_session_id = orders.website_session_id
GROUP BY 1,2
ORDER BY 1,2
;
```
Result :- 

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis_Project_2/blob/main/solution3.JPG)

Conclusion for QUESTION 3:-

The data reveals strong growth across all channels, with significant increases in orders from Gsearch nonbrand, Bsearch nonbrand, brand search, organic search, and direct type-in over the years. Notably, Gsearch nonbrand and brand search orders have grown substantially, especially in 2014, reflecting the success of targeted marketing efforts. Organic search and direct type-in orders have also seen steady growth, contributing to overall channel diversification and effectiveness.


QUESTION 4:- Next, let’s show the overall session-to-order conversion rate trends for those same channels, by quarter. Please also make a note of any periods where we made major improvements or optimizations.

```sql
SELECT 
	YEAR(website_sessions.created_at) AS yr,
	QUARTER(website_sessions.created_at) AS qtr, 
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' AND utm_campaign = 'nonbrand' THEN orders.order_id ELSE NULL END)
		/COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' AND utm_campaign = 'nonbrand' THEN website_sessions.website_session_id ELSE NULL END) AS gsearch_nonbrand_conv_rt, 
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' AND utm_campaign = 'nonbrand' THEN orders.order_id ELSE NULL END) 
		/COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' AND utm_campaign = 'nonbrand' THEN website_sessions.website_session_id ELSE NULL END) AS bsearch_nonbrand_conv_rt, 
    COUNT(DISTINCT CASE WHEN utm_campaign = 'brand' THEN orders.order_id ELSE NULL END) 
		/COUNT(DISTINCT CASE WHEN utm_campaign = 'brand' THEN website_sessions.website_session_id ELSE NULL END) AS brand_search_conv_rt,
    COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NOT NULL THEN orders.order_id ELSE NULL END) 
		/COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NOT NULL THEN website_sessions.website_session_id ELSE NULL END) AS organic_search_conv_rt,
    COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NULL THEN orders.order_id ELSE NULL END) 
		/COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NULL THEN website_sessions.website_session_id ELSE NULL END) AS direct_type_in_conv_rt
FROM website_sessions 
	LEFT JOIN orders
		ON website_sessions.website_session_id = orders.website_session_id
GROUP BY 1,2
ORDER BY 1,2
;
```
Result :-

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis_Project_2/blob/main/solution4.JPG)

Conclusion for QUESTION 4:- 

The data shows a steady improvement in session-to-order conversion rates across all channels over time, with noticeable progress in 2014 and 2015. Major optimizations appear to have occurred around Q4 2013 and Q1 2014, as conversion rates in all channels saw significant increases. The most consistent growth is observed in Gsearch nonbrand, Bsearch nonbrand, and brand search conversion rates, especially in 2014 and 2015, which likely reflects successful optimizations in marketing and targeting strategies.


QUESTION 5:-We’ve come a long way since the days of selling a single product. Let’s pull monthly trending for revenue and margin by product, along with total sales and revenue. Note anything you notice about seasonality.

```sql
SELECT
	YEAR(created_at) AS yr, 
    MONTH(created_at) AS mo, 
    SUM(CASE WHEN product_id = 1 THEN price_usd ELSE NULL END) AS mrfuzzy_rev,
    SUM(CASE WHEN product_id = 1 THEN price_usd - cogs_usd ELSE NULL END) AS mrfuzzy_marg,
    SUM(CASE WHEN product_id = 2 THEN price_usd ELSE NULL END) AS lovebear_rev,
    SUM(CASE WHEN product_id = 2 THEN price_usd - cogs_usd ELSE NULL END) AS lovebear_marg,
    SUM(CASE WHEN product_id = 3 THEN price_usd ELSE NULL END) AS birthdaybear_rev,
    SUM(CASE WHEN product_id = 3 THEN price_usd - cogs_usd ELSE NULL END) AS birthdaybear_marg,
    SUM(CASE WHEN product_id = 4 THEN price_usd ELSE NULL END) AS minibear_rev,
    SUM(CASE WHEN product_id = 4 THEN price_usd - cogs_usd ELSE NULL END) AS minibear_marg,
    SUM(price_usd) AS total_revenue,  
    SUM(price_usd - cogs_usd) AS total_margin
FROM order_items 
GROUP BY 1,2
ORDER BY 1,2
;
```
Result :- 

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis_Project_2/blob/main/solution5.JPG)

Conclusion for QUESTION 5:-

The data reveals strong growth in both revenue and margin, with notable peaks during the holiday months, particularly in November and December, suggesting clear seasonality in product sales. MrFuzzy consistently contributed a significant portion of total revenue and margin, with marked growth from 2012 through 2014. In contrast, the LoveBear, BirthdayBear, and MiniBear products show varied sales, with LoveBear showing strong growth starting in 2014. Overall, December stands out as the month with the highest revenue and margin, likely driven by holiday demand.


QUESTION 6:- Let’s dive deeper into the impact of introducing new products. Please pull monthly sessions to the /products page, and show how the % of those sessions clicking through another page has changed over time, along with a view of how conversion from /products to placing an order has improved.

```sql
-- first, identifying all the views of the /products page
CREATE TEMPORARY TABLE products_pageviews
SELECT
	website_session_id, 
    website_pageview_id, 
    created_at AS saw_product_page_at

FROM website_pageviews 
WHERE pageview_url = '/products'
;


SELECT 
	YEAR(saw_product_page_at) AS yr, 
    MONTH(saw_product_page_at) AS mo,
    COUNT(DISTINCT products_pageviews.website_session_id) AS sessions_to_product_page, 
    COUNT(DISTINCT website_pageviews.website_session_id) AS clicked_to_next_page, 
    COUNT(DISTINCT website_pageviews.website_session_id)/COUNT(DISTINCT products_pageviews.website_session_id) AS clickthrough_rt,
    COUNT(DISTINCT orders.order_id) AS orders,
    COUNT(DISTINCT orders.order_id)/COUNT(DISTINCT products_pageviews.website_session_id) AS products_to_order_rt
FROM products_pageviews
	LEFT JOIN website_pageviews 
		ON website_pageviews.website_session_id = products_pageviews.website_session_id -- same session
        AND website_pageviews.website_pageview_id > products_pageviews.website_pageview_id -- they had another page AFTER
	LEFT JOIN orders 
		ON orders.website_session_id = products_pageviews.website_session_id
GROUP BY 1,2
;
```

Result :-

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis_Project_2/blob/main/solution6.JPG)

Conclusion for QUESTION 6 :-

The data reveals a steady improvement in both the click-through rate (sessions to products page to another page) and the conversion rate (from product page to order) over time, reflecting the positive impact of introducing new products. As the percentage of sessions clicking through to other pages increased, there was a corresponding rise in the conversion from the /products page to placing an order, especially noticeable from late 2013 through 2014. The clickthrough rate consistently improved, peaking in December 2014, while the products-to-order conversion rate also showed steady growth, particularly in early 2015. This suggests that product introductions and page optimizations played a key role in driving customer engagement and boosting sales.


QUESTION 7:-  We made our 4th product available as a primary product on December 05, 2014 (it was previously only a cross-sell item). Could you please pull sales data since then, and show how well each product cross-sells from one another?

```sql
CREATE TEMPORARY TABLE primary_products
SELECT 
	order_id, 
    primary_product_id, 
    created_at AS ordered_at
FROM orders 
WHERE created_at > '2014-12-05' -- when the 4th product was added (says so in question)
;

SELECT
	primary_products.*, 
    order_items.product_id AS cross_sell_product_id
FROM primary_products
	LEFT JOIN order_items 
		ON order_items.order_id = primary_products.order_id
        AND order_items.is_primary_item = 0; -- only bringing in cross-sells;

SELECT 
	primary_product_id, 
    COUNT(DISTINCT order_id) AS total_orders, 
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 1 THEN order_id ELSE NULL END) AS _xsold_p1,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 2 THEN order_id ELSE NULL END) AS _xsold_p2,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 3 THEN order_id ELSE NULL END) AS _xsold_p3,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 4 THEN order_id ELSE NULL END) AS _xsold_p4,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 1 THEN order_id ELSE NULL END)/COUNT(DISTINCT order_id) AS p1_xsell_rt,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 2 THEN order_id ELSE NULL END)/COUNT(DISTINCT order_id) AS p2_xsell_rt,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 3 THEN order_id ELSE NULL END)/COUNT(DISTINCT order_id) AS p3_xsell_rt,
    COUNT(DISTINCT CASE WHEN cross_sell_product_id = 4 THEN order_id ELSE NULL END)/COUNT(DISTINCT order_id) AS p4_xsell_rt
FROM
(
SELECT
	primary_products.*, 
    order_items.product_id AS cross_sell_product_id
FROM primary_products
	LEFT JOIN order_items 
		ON order_items.order_id = primary_products.order_id
        AND order_items.is_primary_item = 0 -- only bringing in cross-sells
) AS primary_w_cross_sell
GROUP BY 1;
```
Result :- 

![Alt text](https://github.com/aa-abhinavacharya/MySQL_For_Advanced_Ecommerce_Data_Analysis_Project_2/blob/main/solution7.JPG)

Conclusion for QUESTION 7 :-

Based on the data, the sales performance and cross-selling effectiveness of each product after the introduction of the fourth product (as a primary product from December 2014) are as follows:
1.Product 1 has the highest number of total orders (4467), with strong cross-sell performance to Product 3 (0.2089) and Product 4 (0.1238), indicating that customers who purchase Product 1 are likely to purchase these other products as well.
2.Product 2 has a modest total of 1277 orders, with significant cross-selling to Product 4 (0.2036), though limited cross-sell opportunities to Product 1 (0.0196) and Product 3 (0.0313).
3.Product 3 has 929 total orders, with notable cross-sell rates to Product 4 (0.2239) and Product 1 (0.0904), showing that this product drives cross-sell activity, especially with Product 4.
4.Product 4 has 581 orders, with the highest cross-sell rate to Product 1 (0.2089), but limited cross-sell performance to the other products, indicating that it has a specific customer base with lower cross-sell potential to other products.
This suggests that Products 1, 3, and 4 are particularly effective at cross-selling, especially in combination with each other, while Product 2 has the least cross-sell impact.


QUESTION 8 :- In addition to telling investors about what we’ve already achieved, let’s show them that we still have plenty of gas in the tank. Based on all the analysis you’ve done, could you share some recommendations and  opportunities for us going forward? No right or wrong answer here – I’d just like to hear your perspective!

Conclusion for Question 8:-

Based on the analysis, here are some key recommendations and opportunities to demonstrate to investors that we still have a lot of growth potential:

1.Enhance Product Cross-Selling: Products 1, 3, and 4 have shown great cross-sell potential. We should focus on further improving product bundling and recommending complementary items to customers, which could increase both conversion rates and revenue. Additionally, optimizing the cross-sell strategy for Product 2, which has a lower cross-sell rate, could help boost its performance.

2.Capitalize on Seasonal Trends: We see clear seasonal spikes in sales, particularly in Q4. There’s an opportunity to leverage this trend even further with targeted marketing campaigns, special offers, or exclusive products that can drive more sales during peak periods. Expanding the product range leading into these high-demand seasons could also help maximize performance.

3.Maximize Marketing Channel Effectiveness: While all channels have grown, Gsearch nonbrand and Brand Search have been particularly strong in driving conversions. We should continue refining our SEO and SEM strategies in these areas to unlock more potential. Additionally, implementing retargeting campaigns for users who visit the /products page but don’t complete purchases could increase conversions.

4.Optimize Conversion Rates: While we’ve seen solid improvements in session-to-order conversion rates, there’s always room for further optimization. Investing in user experience improvements, simplifying the checkout process, and offering more personalized experiences could lead to even higher conversion rates, particularly on the /products page.

5.Expand Product Offerings: The introduction of the fourth product as a primary offering has proven successful. We should continue to explore opportunities to expand the product range further. By analyzing market demand, consumer preferences, and competitor offerings, we can identify opportunities for new product launches or premium versions of existing products.

6.Boost Direct Traffic and Engagement: Although direct type-in orders are steady, there’s potential to further grow this channel. Focusing on building a loyal customer base through email marketing, loyalty programs, or exclusive content for repeat customers could drive more direct traffic and increase sales over time.

7.Leverage Data for Targeted Campaigns: We have a wealth of data on customer behavior, such as click-through rates, product page visits, and conversion rates. By using this data to design more personalized marketing campaigns and promotions, we can boost customer lifetime value (CLV). Targeting high-conversion customers or re-engaging those who didn’t convert initially could also improve results.

8.In summary, by focusing on enhancing product cross-selling, leveraging seasonal demand, refining marketing strategies, expanding our product range, and optimizing customer engagement, the company can continue to build on its growth and create ample opportunities for future success.

# CONCLUSION 

This project provided valuable experience in analyzing business performance using various metrics such as traffic, conversion rates, and product sales. It taught how to identify key trends, including seasonal spikes and cross-sell opportunities, and how to derive actionable insights from data. It also helped develop skills in crafting data-driven recommendations to drive future growth, such as optimizing marketing channels, enhancing user experience, and expanding product offerings. Overall, it reinforced the importance of data in making informed decisions and effectively communicating these findings to stakeholders for continued business success.
