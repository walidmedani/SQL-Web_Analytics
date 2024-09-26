-- Analyzing Top Website Pages & entry pages
```MYSQL	
SELECT 
	wp.pageview_url ,
	count(DISTINCT wp.website_pageview_id) AS pvs
FROM
	website_pageviews wp
WHERE
	wp.website_pageview_id < 1000
GROUP BY
	wp.pageview_url
ORDER BY
	pvs DESC ;
```
```MYSQL	
CREATE TEMPORARY TABLE first_pageview
SELECT 
	wp.website_session_id ,
	min(wp.website_pageview_id) AS min_pv_id
FROM
	website_pageviews wp
WHERE
	wp.website_pageview_id < 1000
GROUP BY
	wp.website_session_id ;
```

```MYSQL
SELECT 
	wp.pageview_url AS landing_page,
	count(DISTINCT first_pageview.website_session_id) AS sessions_hitting_this_lander
FROM
	first_pageview
LEFT JOIN website_pageviews wp 
	ON
	first_pageview.min_pv_id = wp.website_pageview_id
GROUP BY
	wp.pageview_url ;
```
-- Analyzing bounce rates & landing page tests
```MYSQL	
SELECT 
	wp.website_session_id ,
	min(wp.website_pageview_id) AS min_pageview_id 
FROM website_pageviews wp 
	INNER JOIN website_sessions ws 
		ON ws.website_session_id = wp.website_session_id 
		AND ws.created_at BETWEEN '2014-01-01' AND '2014-02-01'
GROUP BY 
	wp.website_session_id ;
```
```MYSQL	
CREATE TEMPORARY TABLE first_pageviews_demo -- same query AS above, but storing the dataset AS temp
SELECT 
	wp.website_session_id ,
	min(wp.website_pageview_id) AS min_pageview_id 
FROM website_pageviews wp 
	INNER JOIN website_sessions ws 
		ON ws.website_session_id = wp.website_session_id 
		AND ws.created_at BETWEEN '2014-01-01' AND '2014-02-01'
GROUP BY 
	wp.website_session_id ;

SELECT * FROM first_pageviews_demo ;
```

```MYSQL	
CREATE TEMPORARY TABLE sessions_w_landing_page_demo -- bring in the landing page to each session
SELECT
	first_pageviews_demo.website_session_id,
	wp.pageview_url AS landing_page
FROM
	first_pageviews_demo
LEFT JOIN website_pageviews wp 
		ON
	wp.website_pageview_id = first_pageviews_demo.min_pageview_id;
	
SELECT * FROM sessions_w_landing_page_demo;
```

CREATE TEMPORARY TABLE bounced_sessions_only
SELECT
	sessions_w_landing_page_demo.website_session_id,
	sessions_w_landing_page_demo.landing_page,
	count(wp.website_pageview_id) AS count_of_pages_viewed 
FROM sessions_w_landing_page_demo
LEFT JOIN website_pageviews wp 
	ON wp.website_session_id = sessions_w_landing_page_demo.website_session_id
GROUP BY 
	sessions_w_landing_page_demo.website_session_id,
	sessions_w_landing_page_demo.landing_page
HAVING 
	count(wp.website_pageview_id) = 1;

SELECT * FROM bounced_sessions_only;

SELECT	
	sessions_w_landing_page_demo.landing_page,
	COUNT(DISTINCT sessions_w_landing_page_demo.website_session_id) AS sessions,
	COUNT(DISTINCT bounced_sessions_only.website_session_id) AS bounce_website_session_id,
	COUNT(DISTINCT bounced_sessions_only.website_session_id) / COUNT(DISTINCT sessions_w_landing_page_demo.website_session_id) AS bounce_rate
FROM sessions_w_landing_page_demo
	LEFT JOIN bounced_sessions_only
		ON sessions_w_landing_page_demo.website_session_id = bounced_sessions_only.website_session_id
GROUP BY 
	sessions_w_landing_page_demo.landing_page;


-- Conversion funnels & conversion paths

SELECT 
	website_session_id,
	max(products_page) AS product_made_it,
	max(mrfuzzy_page) AS mrfuzzy_made_it,
	max(cart_page) AS cart_made_it
FROM (
	SELECT 
	ws.website_session_id,
	wp.pageview_url ,
	wp.created_at AS pageview_created_at,
	CASE WHEN pageview_url = '/products' THEN 1 ELSE 0 END AS products_page,
	CASE WHEN pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS mrfuzzy_page,
	CASE WHEN pageview_url = '/cart' THEN 1 ELSE 0 END AS cart_page
FROM website_sessions ws 
	LEFT JOIN website_pageviews wp 
		ON ws.website_session_id = wp.website_session_id 
WHERE 
	ws.created_at BETWEEN '2014-01-01' AND '2014-02-01'
	AND wp.pageview_url IN ('/lander-2', '/products','/the-original-mr-fuzzy','/cart')
ORDER BY 
	ws.website_session_id,
	wp.created_at
) AS pageview_level
GROUP BY 
	website_session_id;


CREATE TEMPORARY TABLE session_level_made_it_flags_demo
SELECT 
	website_session_id,
	max(products_page) AS product_made_it,
	max(mrfuzzy_page) AS mrfuzzy_made_it,
	max(cart_page) AS cart_made_it
FROM (
	SELECT 
	ws.website_session_id,
	wp.pageview_url ,
	wp.created_at AS pageview_created_at,
	CASE WHEN pageview_url = '/products' THEN 1 ELSE 0 END AS products_page,
	CASE WHEN pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS mrfuzzy_page,
	CASE WHEN pageview_url = '/cart' THEN 1 ELSE 0 END AS cart_page
FROM website_sessions ws 
	LEFT JOIN website_pageviews wp 
		ON ws.website_session_id = wp.website_session_id 
WHERE 
	ws.created_at BETWEEN '2014-01-01' AND '2014-02-01'
	AND wp.pageview_url IN ('/lander-2', '/products','/the-original-mr-fuzzy','/cart')
ORDER BY 
	ws.website_session_id,
	wp.created_at
) AS pageview_level
GROUP BY 
	website_session_id;

SELECT 
	count(DISTINCT website_session_id) AS sessions,
	count(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END) AS to_products,
	count(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END) AS to_mrfuzzy,
	count(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END) AS to_cart
FROM session_level_made_it_flags_demo;

SELECT 
	count(DISTINCT website_session_id) AS sessions,
	count(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END)
		/ count(DISTINCT website_session_id) AS click_products,
	count(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END)
		/ count(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END) AS click_mrfuzzy,
	count(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END)
		/ count(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END) AS click_cart
FROM session_level_made_it_flags_demo;
