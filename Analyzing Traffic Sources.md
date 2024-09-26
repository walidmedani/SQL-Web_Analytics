# Finding Top Traffic Sources
This query identifies the top traffic sources driving sessions to the website by grouping data by source, campaign, and referrer. It helps determine which marketing campaigns and traffic sources were most effective in generating website visits.

- utm_source: Source of traffic (e.g., Google, Facebook).
- utm_campaign: Campaign name associated with the traffic.
- http_referer: The URL from where the session originated.
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

&nbsp;
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

&nbsp;
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

&nbsp;
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
WHERE ws.utm_source = 'gsearch'
		AND ws.utm_campaign = 'nonbrand'
GROUP BY 1
ORDER BY 3 DESC;
```
#### Result:
|device_type|sessions|orders|session_to_orders_conv_rt|
|-----------|--------|------|-------------------------|
|desktop|195155|16037|0.0822|
|mobile|87551|2785|0.0318|

&nbsp;
## Trending w/ Granular Segments
```MYSQL
SELECT 
	MIN(DATE(ws.created_at)) AS week_start ,
	COUNT(DISTINCT CASE WHEN WS.device_type = 'desktop' THEN ws.website_session_id ELSE NULL END) AS dtop_sesssions,
	count(DISTINCT CASE WHEN WS.device_type = 'mobile' THEN ws.website_session_id ELSE NULL END) AS mob_sessions
FROM website_sessions ws 
WHERE ws.utm_source = 'gsearch'
		AND ws.utm_campaign = 'nonbrand'
GROUP BY 
	YEAR(ws.created_at) ,
	WEEK(ws.created_at);
```
#### Result:
|week_start|dtop_sesssions|mob_sessions|
|----------|--------------|------------|
|2012-03-19|543|353|
|2012-03-25|585|371|
|2012-04-01|693|459|
|2012-04-08|605|378|
|2012-04-15|383|238|
|2012-04-22|360|234|
|2012-04-29|425|256|
|2012-05-06|430|282|
|2012-05-13|403|214|
|2012-05-20|661|190|
|2012-05-27|585|183|
|2012-06-03|625|167|
|2012-06-10|689|186|
|2012-06-17|647|195|
|2012-06-24|582|173|
|2012-07-01|599|181|
|2012-07-08|596|205|
|2012-07-15|647|203|
|2012-07-22|624|172|
|2012-07-29|748|280|
|2012-08-05|835|252|
|2012-08-12|730|268|
|2012-08-19|771|241|
|2012-08-26|793|263|
|2012-09-02|683|242|
|2012-09-09|720|231|
|2012-09-16|877|274|
|2012-09-23|784|266|
|2012-09-30|761|238|
|2012-10-07|752|250|
|2012-10-14|934|323|
|2012-10-21|997|305|
|2012-10-28|923|288|
|2012-11-04|1027|323|
|2012-11-11|956|290|
|2012-11-18|2655|853|
|2012-11-25|2058|692|
|2012-12-02|1326|396|
|2012-12-09|1277|424|
|2012-12-16|1370|412|
|2012-12-23|765|245|
|2012-12-30|140|44|
|2013-01-01|464|157|
|2013-01-06|582|172|
|2013-01-13|619|199|
|2013-01-20|656|216|
|2013-01-27|627|184|
|2013-02-03|611|186|
|2013-02-10|1671|536|
|2013-02-17|653|197|
|2013-02-24|667|235|
|2013-03-03|683|242|
|2013-03-10|652|231|
|2013-03-17|692|197|
|2013-03-24|841|260|
|2013-03-31|883|276|
|2013-04-07|947|306|
|2013-04-14|974|282|
|2013-04-21|883|315|
|2013-04-28|986|305|
|2013-05-05|1014|262|
|2013-05-12|922|289|
|2013-05-19|907|284|
|2013-05-26|930|276|
|2013-06-02|1001|300|
|2013-06-09|901|317|
|2013-06-16|1029|324|
|2013-06-23|997|322|
|2013-06-30|932|313|
|2013-07-07|972|282|
|2013-07-14|955|280|
|2013-07-21|1003|293|
|2013-07-28|955|280|
|2013-08-04|914|287|
|2013-08-11|939|318|
|2013-08-18|960|490|
|2013-08-25|958|501|
|2013-09-01|914|482|
|2013-09-08|909|492|
|2013-09-15|974|523|
|2013-09-22|944|558|
|2013-09-29|1008|511|
|2013-10-06|1012|555|
|2013-10-13|941|510|
|2013-10-20|982|477|
|2013-10-27|1037|545|
|2013-11-03|992|519|
|2013-11-10|943|554|
|2013-11-17|989|567|
|2013-11-24|2952|1600|
|2013-12-01|2158|1159|
|2013-12-08|1317|684|
|2013-12-15|1537|799|
|2013-12-22|1134|584|
|2013-12-29|406|244|
|2014-01-01|625|325|
|2014-01-05|1089|563|
|2014-01-12|1079|534|
|2014-01-19|1035|557|
|2014-01-26|1210|615|
|2014-02-02|1200|603|
|2014-02-09|1695|862|
|2014-02-16|1266|693|
|2014-02-23|1251|680|
|2014-03-02|1244|621|
|2014-03-09|1250|672|
|2014-03-16|1262|665|
|2014-03-23|1221|670|
|2014-03-30|1451|781|
|2014-04-06|1610|845|
|2014-04-13|1550|841|
|2014-04-20|1587|834|
|2014-04-27|1497|802|
|2014-05-04|1546|907|
|2014-05-11|1624|928|
|2014-05-18|1526|892|
|2014-05-25|1524|821|
|2014-06-01|1544|819|
|2014-06-08|1548|930|
|2014-06-15|1621|832|
|2014-06-22|1638|873|
|2014-06-29|1597|881|
|2014-07-06|1615|886|
|2014-07-13|1614|808|
|2014-07-20|1584|883|
|2014-07-27|1626|852|
|2014-08-03|1576|819|
|2014-08-10|1538|833|
|2014-08-17|1599|819|
|2014-08-24|1597|870|
|2014-08-31|1553|845|
|2014-09-07|1620|877|
|2014-09-14|1540|903|
|2014-09-21|1591|826|
|2014-09-28|1611|890|
|2014-10-05|1621|900|
|2014-10-12|1670|899|
|2014-10-19|1691|887|
|2014-10-26|1657|836|
|2014-11-02|1585|880|
|2014-11-09|2087|828|
|2014-11-16|1999|877|
|2014-11-23|3767|1591|
|2014-11-30|3082|1329|
|2014-12-07|2373|1054|
|2014-12-14|2362|997|
|2014-12-21|2325|984|
|2014-12-28|1237|525|
|2015-01-01|874|372|
|2015-01-04|2193|905|
|2015-01-11|2181|933|
|2015-01-18|2123|949|
|2015-01-25|2225|971|
|2015-02-01|2105|897|
|2015-02-08|2682|1173|
|2015-02-15|2180|921|
|2015-02-22|2163|913|
|2015-03-01|2250|955|
|2015-03-08|2241|955|
|2015-03-15|1435|546|

