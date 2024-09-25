# Finding top traffic sources
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
WHERE ws.created_at < '2012-04-12'
GROUP BY 1,
		 2,
		 3
ORDER BY 4 DESC;
```

# Finding traffic source conversion
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
WHERE ws.website_session_id BETWEEN 1000 AND 2000
GROUP BY 
	1
ORDER BY sessions DESC;
```
```MYSQL
SELECT 
	count(DISTINCT ws.website_session_id) AS sessions ,
	count(DISTINCT o.order_id) AS orders,
	count(DISTINCT o.order_id)/count(DISTINCT ws.website_session_id) AS session_to_orders_conv_rt
FROM website_sessions ws 
	LEFT JOIN orders o 
	ON o.website_session_id = ws.website_session_id
WHERE ws.created_at < '2012-04-14'
	  AND ws.utm_source = 'gsearch'
	  AND ws.utm_campaign = 'nonbrand'
ORDER BY 3 DESC;
```
-- Traffic source trending
SELECT
	YEAR(ws.created_at) ,
	WEEK(ws.created_at) ,
	MIN(DATE(ws.created_at)) AS week_start , 
	count(DISTINCT ws.website_session_id) 
FROM website_sessions ws 
WHERE ws.website_session_id BETWEEN 100000 AND 115000
GROUP BY 1,2;


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

-- Bid optimization for paid traffic
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

-- Trending w/ granular segments
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

SELECT * FROM website_sessions ws 
