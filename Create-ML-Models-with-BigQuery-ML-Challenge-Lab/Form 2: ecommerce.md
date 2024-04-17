If you find my repository helpful, please star⭐ it 🌟.


## Task-1: Create a new dataset with dataset id 'bq_dataset'
1. In the Explorer pane of the BigQuery Console, click on the three dots next to your Project ID and then select Create dataset.
2. In the Create dataset dialog, set the Dataset ID to `bq_dataset` and leave the other options at their default values.
3. Click __Create dataset__.

OR
      
1. Open Cloud Shell
2. Click Authorize
3. Copy and paste the command. Press enter
```
bq mk bq_dataset
```
4. Dataset will be created

## Task-2: Evaluate classification model performance
 
For this task you have precreated dataset `ecommerce` which contains a pretrained BigQuery ML model `customer_classification_model`.
You have to evaluate the performance of the model against new unseen evaluation data.
 
1. Open the Explorer pane of the BigQuery Console,
2. Click "+" (Compose New Query) icon and then run the following query:
```
SELECT

roc_auc,

CASE

  WHEN roc_auc > .9 THEN 'good'

  WHEN roc_auc > .8 THEN 'fair'

  WHEN roc_auc > .7 THEN 'decent'

  WHEN roc_auc > .6 THEN 'not great'

ELSE 'poor' END AS model_quality

FROM

ML.EVALUATE(MODEL ecommerce.customer_classification_model,  (

 

SELECT

* EXCEPT(fullVisitorId)

FROM

 

# features

(SELECT

  fullVisitorId,

  IFNULL(totals.bounces, 0) AS bounces,

  IFNULL(totals.timeOnSite, 0) AS time_on_site

FROM

  `data-to-insights.ecommerce.web_analytics`

WHERE

  totals.newVisits = 1

  AND date BETWEEN '20170501' AND '20170630') # eval on 2 months

JOIN

(SELECT

  fullvisitorid,

  IF(COUNTIF(totals.transactions > 0 AND totals.newVisits IS NULL) > 0, 1, 0) AS will_buy_on_return_visit

FROM

    `data-to-insights.ecommerce.web_analytics`

GROUP BY fullvisitorid)

USING (fullVisitorId)

 

));
```
## Task-3: Improve model performance with Feature Engineering and Evaluate the model to see if there is better predictive power
 
For this task you have precreated dataset `ecommerce` which contains a pretrained BigQuery ML model `customer_classification_model`, but there are many more features in the dataset that may help the model better understand the relationship between a visitor's first session and the likelihood that they will purchase on a subsequent visit.
 
You have to add some new features and create a second machine learning model called `improved_customer_classification_model`.
Also, evaluate the newly created model.
 
1. Open the Explorer pane of the BigQuery Console,
2. Click "+" (Compose New Query) icon and then run the following query:
 
Improve model performance with Feature Engineering.

```
CREATE OR REPLACE MODEL `ecommerce.improved_customer_classification_model`

OPTIONS

(model_type='logistic_reg', labels = ['will_buy_on_return_visit']) AS

 

WITH all_visitor_stats AS (

SELECT

fullvisitorid,

IF(COUNTIF(totals.transactions > 0 AND totals.newVisits IS NULL) > 0, 1, 0) AS will_buy_on_return_visit

FROM `data-to-insights.ecommerce.web_analytics`

GROUP BY fullvisitorid

)

 

# add in new features

SELECT * EXCEPT(unique_session_id) FROM (

 

SELECT

    CONCAT(fullvisitorid, CAST(visitId AS STRING)) AS unique_session_id,

 

    # labels

    will_buy_on_return_visit,

 

    MAX(CAST(h.eCommerceAction.action_type AS INT64)) AS latest_ecommerce_progress,

 

    # behavior on the site

    IFNULL(totals.bounces, 0) AS bounces,

    IFNULL(totals.timeOnSite, 0) AS time_on_site,

    IFNULL(totals.pageviews, 0) AS pageviews,

 

    # where the visitor came from

    trafficSource.source,

    trafficSource.medium,

    channelGrouping,

 

    # mobile or desktop

    device.deviceCategory,

 

    # geographic

    IFNULL(geoNetwork.country, "") AS country

 

FROM `data-to-insights.ecommerce.web_analytics`,

   UNNEST(hits) AS h

 

  JOIN all_visitor_stats USING(fullvisitorid)

 

WHERE 1=1

  # only predict for new visits

  AND totals.newVisits = 1

  AND date BETWEEN '20160801' AND '20170430' # train 9 months

 

GROUP BY

unique_session_id,

will_buy_on_return_visit,

bounces,

time_on_site,

totals.pageviews,

trafficSource.source,

trafficSource.medium,

channelGrouping,

device.deviceCategory,

country

);
```

3. Evaluate the model to see if there is better predictive power.
```
#standardSQL

SELECT

roc_auc,

CASE

  WHEN roc_auc > .9 THEN 'good'

  WHEN roc_auc > .8 THEN 'fair'

  WHEN roc_auc > .7 THEN 'decent'

  WHEN roc_auc > .6 THEN 'not great'

ELSE 'poor' END AS model_quality

FROM

ML.EVALUATE(MODEL ecommerce.improved_customer_classification_model,  (

 

WITH all_visitor_stats AS (

SELECT

fullvisitorid,

IF(COUNTIF(totals.transactions > 0 AND totals.newVisits IS NULL) > 0, 1, 0) AS will_buy_on_return_visit

FROM `data-to-insights.ecommerce.web_analytics`

GROUP BY fullvisitorid

)

 

# add in new features

SELECT * EXCEPT(unique_session_id) FROM (

 

SELECT

    CONCAT(fullvisitorid, CAST(visitId AS STRING)) AS unique_session_id,

 

    # labels

    will_buy_on_return_visit,

 

    MAX(CAST(h.eCommerceAction.action_type AS INT64)) AS latest_ecommerce_progress,

 

    # behavior on the site

    IFNULL(totals.bounces, 0) AS bounces,

    IFNULL(totals.timeOnSite, 0) AS time_on_site,

    totals.pageviews,

 

    # where the visitor came from

    trafficSource.source,

    trafficSource.medium,

    channelGrouping,

 

    # mobile or desktop

    device.deviceCategory,

 

    # geographic

    IFNULL(geoNetwork.country, "") AS country

 

FROM `data-to-insights.ecommerce.web_analytics`,

   UNNEST(hits) AS h

 

  JOIN all_visitor_stats USING(fullvisitorid)

 

WHERE 1=1

  # only predict for new visits

  AND totals.newVisits = 1

  AND date BETWEEN '20170501' AND '20170630' # eval 2 months

 

GROUP BY

unique_session_id,

will_buy_on_return_visit,

bounces,

time_on_site,

totals.pageviews,

trafficSource.source,

trafficSource.medium,

channelGrouping,

device.deviceCategory,

country

)

));

```

## Task-4: Predict which new visitors will come back and purchase
 
You have precreated dataset `ecommerce` which contains a pretrained BigQuery ML model `finalized_classification_model`.
Write a query to predict which new visitors will come back and make a purchase.
 
1. Open the Explorer pane of the BigQuery Console,
2. Click "+" (Compose New Query) icon and then run the following query:
```
SELECT

*

FROM

ml.PREDICT(MODEL `ecommerce.finalized_classification_model`,

 (

 

WITH all_visitor_stats AS (

SELECT

fullvisitorid,

IF(COUNTIF(totals.transactions > 0 AND totals.newVisits IS NULL) > 0, 1, 0) AS will_buy_on_return_visit

FROM `data-to-insights.ecommerce.web_analytics`

GROUP BY fullvisitorid

)

 

SELECT

    CONCAT(fullvisitorid, '-',CAST(visitId AS STRING)) AS unique_session_id,

 

    # labels

    will_buy_on_return_visit,

 

    MAX(CAST(h.eCommerceAction.action_type AS INT64)) AS latest_ecommerce_progress,

 

    # behavior on the site

    IFNULL(totals.bounces, 0) AS bounces,

    IFNULL(totals.timeOnSite, 0) AS time_on_site,

    totals.pageviews,

 

    # where the visitor came from

    trafficSource.source,

    trafficSource.medium,

    channelGrouping,

 

    # mobile or desktop

    device.deviceCategory,

 

    # geographic

    IFNULL(geoNetwork.country, "") AS country

 

FROM `data-to-insights.ecommerce.web_analytics`,

   UNNEST(hits) AS h

 

  JOIN all_visitor_stats USING(fullvisitorid)

 

WHERE

  # only predict for new visits

  totals.newVisits = 1

  AND date BETWEEN '20170701' AND '20170801' # test 1 month

 

GROUP BY

unique_session_id,

will_buy_on_return_visit,

bounces,

time_on_site,

totals.pageviews,

trafficSource.source,

trafficSource.medium,

channelGrouping,

device.deviceCategory,

country

)

 

)

 

ORDER BY

predicted_will_buy_on_return_visit DESC;
```
