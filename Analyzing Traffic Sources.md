# Finding Top Traffic Sources
This query identifies the top traffic sources driving sessions to the website by grouping data by source, campaign, and referrer. It helps determine which marketing campaigns and traffic sources were most effective in generating website visits.

utm_source: Source of traffic (e.g., Google, Facebook).
utm_campaign: Campaign name associated with the traffic.
http_referer: The URL from where the session originated.
```MYSQL
SELECT 
	ws.utm_source ,
	ws.utm_campaign ,
	ws.http_referer ,
	count(DISTINCT ws.website_session_id) AS sessions
FROM website_sessions ws
WHERE ws.created_at
GROUP BY 1,
		 2,
		 3
ORDER BY 4 DESC;
```
#### Result:
|utm_source|utm_campaign|http_referer|sessions|
|----------|------------|------------|--------|
|gsearch|nonbrand|https://www.gsearch.com|282706|
|bsearch|nonbrand|https://www.bsearch.com|54909|
||||39917|
|||https://www.gsearch.com|35202|
|gsearch|brand|https://www.gsearch.com|33329|
|||https://www.bsearch.com|8209|
|bsearch|brand|https://www.bsearch.com|7914|
|socialbook|desktop_targeted|https://www.socialbook.com|5590|
|socialbook|pilot|https://www.socialbook.com|5095|


# Finding Traffic Source Conversion
This query evaluates conversion rates by traffic content, determining how effectively different marketing messages or content converted sessions into orders. The session_to_orders_conv_rt measures the effectiveness of traffic in driving sales.

LEFT JOIN with orders ensures all sessions are included, even those without an order.
```MYSQL
SELECT 
	ws.utm_content ,
	count(DISTINCT ws.website_session_id) AS sessions ,
	count(DISTINCT o.order_id) AS orders ,
	count(DISTINCT o.order_id)/count(DISTINCT ws.website_session_id) AS session_to_orders_conv_rt
FROM website_sessions ws 
	LEFT JOIN orders o 
	ON o.website_session_id = ws.website_session_id
WHERE ws.website_session_id
GROUP BY 
	1
ORDER BY sessions DESC;
```
#### Result:
|utm_content|sessions|orders|session_to_orders_conv_rt|
|-----------|--------|------|-------------------------|
|g_ad_1|282706|18822|0.0666|
||83328|6118|0.0734|
|b_ad_1|54909|3818|0.0695|
|g_ad_2|33329|2511|0.0753|
|b_ad_2|7914|701|0.0886|
|social_ad_2|5590|288|0.0515|
|social_ad_1|5095|55|0.0108|

## Traffic Source Trending
```MYSQL
SELECT 
	MIN(DATE(ws.created_at)) AS week_start , 
	count(DISTINCT ws.website_session_id) AS sessions
FROM website_sessions ws 
WHERE WS.created_at < '2012-05-10'
		AND ws.utm_source = 'gsearch'
		AND ws.utm_campaign = 'nonbrand'
GROUP BY 
	YEAR(ws.created_at) ,
	WEEK(ws.created_at);
```
#### Result:
|week_start|sessions|
|----------|--------|
|2012-03-19|896|
|2012-03-25|956|
|2012-04-01|1152|
|2012-04-08|983|
|2012-04-15|621|
|2012-04-22|594|
|2012-04-29|681|
|2012-05-06|399|

## Bid Optimization For Paid Traffic
```MYSQL
SELECT
	ws.device_type ,
	count(DISTINCT ws.website_session_id) AS sessions ,
	count(DISTINCT o.order_id) AS orders,
	count(DISTINCT o.order_id)/count(DISTINCT ws.website_session_id) AS session_to_orders_conv_rt
FROM website_sessions ws 
	LEFT JOIN orders o 
	ON o.website_session_id = ws.website_session_id
WHERE ws.created_at < '2012-04-15'
	AND ws.utm_source = 'gsearch'
		AND ws.utm_campaign = 'nonbrand'
GROUP BY 1
ORDER BY 3 DESC;
```
-- Trending w/ granular segments
```MYSQL
SELECT 
	MIN(DATE(ws.created_at)) AS week_start ,
	COUNT(DISTINCT CASE WHEN WS.device_type = 'desktop' THEN ws.website_session_id ELSE NULL END) AS dtop_sesssions,
	count(DISTINCT CASE WHEN WS.device_type = 'mobile' THEN ws.website_session_id ELSE NULL END) AS mob_sessions
FROM website_sessions ws 
WHERE ws.created_at < '2012-06-09'
	AND ws.created_at > '2012-04-15'
	AND ws.utm_source = 'gsearch'
		AND ws.utm_campaign = 'nonbrand'
GROUP BY 
	YEAR(ws.created_at) ,
	WEEK(ws.created_at);
```
