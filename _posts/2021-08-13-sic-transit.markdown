---
layout: post
title:  "Sic transit"
date:   2021-08-13 23:07:59 -0500
categories: jekyll update
---

## Introduction

Data changes over time: old data sources are deprecated, new data sources are added, attributes are modified, event semantics and data models are evolved, and so on. How do we avoid the maintenance "tax" associated with keeping our downstream models, dashboards, and ad-hoc queries up-to-date with these changes?

The solution is obvious conceptually, even if it is not obvious how to implement it. We need to separate the form of the data from its function: how the data is represented in a database from how it is accessed. This is much like how an API ("application programming interface") provides methods for interacting with a service that are logically separated from the computations occurring within the service itself. As a programmer, I do not care how the data for the exchange rate between US and Canadian dollars is stored, so long as I can retrieve it on a particular day. This provides the API developer the freedom to modify the data store without impacting the programmer's code. Quite similarly, we would like an interface for data access that a dashboard developer can use without needing to know about changes to data sources, data models, event semantics, etc.

At this point, it is worth being somewhat more concrete, because the statements above are probably too broad and vague to be practically useful. Let's start with some definitions and distinctions.

1. For the purposes of this discussion, a "data source" is a table or view that can be queried in SQL. A "query" accesses one or more data sources to generate another dataset: queries can be standalone (for ad-hoc analysis), or can be used to power dashboards, ML models, derived datasets (which constitute new data sources), etc.
2. A query can "break" either syntactically or semantically. The former ("syntactic") is easier to diagnose: it means that the query is gramatically incorrect, and it won't compile. This can happen if a column is removed from a data source that the query references, or if the data source itself is deprecated. More common is the latter case ("semantic"), however. This means that the query still compiles and runs, but the result is no longer correct. If I write a query that constructs a dataset for daily revenue, and the company introduces a new revenue source not captured by my query, my query remains syntactically correct but becomes broken semantically. These breakages are much more insidious because they are essentially "silent errors".
3. Some changes to the data model necessitate updates to queries; others don't. The difference is whether we are changing the syntax or semantics of our data model. A syntactic change is something that leaves the meaning of the data source intact but changes what various entities are called. Examples include renaming attributes, renaming data values, switching APIs (assuming they have equivalent data that is somewhat differently named), etc. Semantic changes, such as the "new revenue source" example discussed above, will always require associated work to update queries.
4. To phrase it differently: semantic changes to the data model guarantee that queries will break semantically. However, syntactic changes to the data model can cause _either_ semantic or syntactic breakages. As an example, if we drop an attribute of an event in our application code, then our query would be broken syntactically if we also dropped the corresponding column in our data source. But, the query would be broken semantically if we did not drop the column and instead populated it with `NULL`s when the attribute is missing.


### Goals

What are our goals here?

1. *Syntactic data model changes*: We would like to grant our application developers the freedom to make syntactic changes to the data model -- to rename poorly-named event attributes and data values to better ones, etc. -- without having to do much (or any) work to keep our queries up to date. We would also like to ensure that if a data source (API, etc.) that we rely on becomes deprecated, we can seamlessly switch to a new one that is semantically equivalent, again without wasting gobs of analysts' time.
2. *Semantic data model changes*: We would like any changes that affect the semantics of the data model to be properly accounted for in the queries that the analytics team builds. In this case, we concede that we must make changes to our downstream queries, but the goal is to make this work as lightweight as possible.

### A case study
Suppose we have a chess mobile app (such as [Really Bad Chess](http://reallybadchess.com/), which I highly recommend). Suppose the app makes money when a player purchases lessons (on how to play chess really badly, presumably). We might imagine a transactions table that looks like this:

| player_id | event_at | lesson_id | revenue_usd |
| :---------: | :------: | :-------: | :----: |
| 1 | 2021-08-01 02:00:01 | Bad Chess | 1.99 | 
| 1 | 2021-08-05 02:00:01 | Worse Chess | 2.99 |
| 2 | 2021-08-10 02:00:01 | The Worst Chess | 4.99 |


 We want to be able to compute how much revenue each player generates, how much each lesson makes per day, etc. These queries are straightforward to write; the latter might look like:

```sql
SELECT
    event_at::DATE AS purchase_date
    ,lesson_id
    ,SUM(revenue_usd) AS revenue_usd
FROM transactions
GROUP BY 1, 2
```

#### Enter: a problem

Our mobile app developer realizes that she had implemented the transaction event incorrectly in the first version of the app (1.0.0): some of these transactions are fraudulent, and we don't actually receive money for them. She decides to implement a new event, `ValidTransaction`, that should track only non-fraudulent transactions. So that we can get an estimate of what percentage of transactions are fraudulent, she decides to keep the existing event in place. Later, when our sample size is large enough, we will deprecate the existing event. She excitedly announces that these changes will go out in app version 1.0.1, and the existing event will eventually be deprecated in yet another app version (not yet decided on). How do we keep the query written above from breaking, both syntactically and semantically?

This is surprisingly tricky. There are three periods: one in which only the existing event is firing (1.0.0 and before); one in which both events are firing (1.0.1 to some unknown version); and one in which only the new event is firing (the unknown version and after). Furthermore, these periods don't necessarily correspond to calendar dates: different users will upgrade their chess mobile app on different dates. Finally, if we add attributes for the app version or event name, both of which are currently missing, these will not be backfilled in the source table itself. See example dataset below:

| player_id | event_at | lesson_id | revenue_usd | app_version | event_name |
| :---------: | :------: | :-------: | :----: | :----------: | :----------: |
| 1 | 2021-08-01 02:00:01 | Bad Chess | 1.99 | NULL | NULL |
| 1 | 2021-08-05 02:00:01 | Worse Chess | 2.99 | NULL | NULL |
| 2 | 2021-08-10 02:00:01 | The Worst Chess | 4.99 | NULL | NULL |
| 1 | 2021-08-15 02:00:01 | The Worst Chess | 4.99 | 1.0.1 | OldTransaction |
| 1 | 2021-08-15 02:00:01 | The Worst Chess | 4.99 | 1.0.1 | ValidTransaction |


### Solutions

There are a few interrelated solutions to this problem.

#### View layer
Use a "view layer" on top of the raw table. _All_ queries, whether they are in dashboards, Jupyter notebooks, ML data preparation jobs, etc, should reference the view instead of the table itself. To repeat: no one should reference the raw data in the table. The view can encapsulate the logic for assigning defaults, renaming columns, dealing with backwards compatibility, etc. So, we might have something like (in dbt style):

{% raw %}
```sql
-- transactions_cleaned.sql
{{
    config(materialized='view')
}}

SELECT
    player_id
    ,event_at
    ,lesson_id
    ,revenue_usd
    ,app_version
    ,COALESCE(event_name, 'OldPurchase') AS event_name
FROM {{ source('transactions') }}
```
{% endraw %}

By creating a single point of contact with the raw, granular data, we can avoid making changes to every single downstream query when simple, syntactic changes to the data model happen, such as renaming of attributes or data values (like enums). (Of course, if we make _semantic_ changes to the data model, it is likely that many downstream queries will still need to be updated.)

The view layer has an additional benefit: if we need to replace the underlying data source with a new one that is semantically equivalent, that becomes as simple as replacing `data_table_old` with `data_table_new` in the view layer, and none of the downstream queries need to know the difference.

#### Analytics versioning
Unfortunately, we are still not able to compute the revenue by lesson easily. We need to be able to apply different business logic depending on which of the three periods described above the event pertains to. So, something like:

```sql
SELECT
    event_at::DATE AS purchase_date
    ,lesson_id
    ,SUM(
        CASE
            WHEN [period_1] AND name = 'OldPurchase'
                -- note: this is only approximate, because of the error
                -- we might try applying a discount factor to the revenue,
                -- based on the comparison of OldPurchase and ValidPurchase in period 2
                -- in order to fix this
                THEN revenue_usd
            WHEN [period_2] AND name = 'ValidPurchase'
                THEN revenue_usd
            WHEN [period_3] AND name = 'ValidPurchase'
                THEN revenue_usd
            ELSE
                0
        END
    ) AS revenue_usd
FROM transactions_cleaned
GROUP BY 1, 2
```

The periods are not defined by `event_at::DATE`, for the reasons described above. Instead, they correspond to the version of the app that was released, the `app_version`. Unfortunately, semantic versions are difficult to compare in SQL. More generically, it is useful to create a separate attribute, which I will call the `analytics_version`, that tracks changes to the data model and can be used for cases like these in which having consistent logic for historical and current events is important. The `analytics_version` can be attached to every event, and [calendar versioning](https://calver.org/) can be used to make the SQL easier to write. So, we might have something like:

| player_id | event_at | lesson_id | revenue_usd | app_version | event_name | analytics_version |
| :---------: | :------: | :-------: | :----: | :----------: | :----------: | :-------: |
| 1 | 2021-08-01 02:00:01 | Bad Chess | 1.99 | NULL | NULL | NULL |
| 1 | 2021-08-05 02:00:01 | Worse Chess | 2.99 | NULL | NULL | NULL |
| 2 | 2021-08-10 02:00:01 | The Worst Chess | 4.99 | NULL | NULL | NULL |
| 1 | 2021-08-15 02:00:01 | The Worst Chess | 4.99 | 1.0.1 | OldPurchase | 2021.08.10 |
| 1 | 2021-08-15 02:00:01 | The Worst Chess | 4.99 | 1.0.1 | ValidPurchase | 2021.08.10 |

where, each time the analytics in the application code undergoes a semantic change, the `analytics_version` is bumped to the date of the release. With this, we can write our view layer as:

{% raw %}
```sql
-- transactions_cleaned.sql
{{
    config(materialized='view')
}}

SELECT
    player_id
    ,event_at
    ,lesson_id
    ,revenue_usd
    ,app_version
    ,COALESCE(event_name, 'OldPurchase') AS event_name
    -- some arbitrarily old date
    ,COALESCE(analytics_version, '1970.01.01') AS analytics_version
FROM {{ source('transactions') }}
```
{% endraw %}

and our desired query as:

```sql
SELECT
    event_at::DATE AS purchase_date
    ,lesson_id
    ,SUM(
        CASE
            WHEN analytics_version < '2021.08.10' AND event_name = 'OldTransaction'
                -- again, this is only approximate
                THEN revenue_usd
            WHEN analytics_version >= '2021.08.10' AND event_name = 'ValidTransaction'
                THEN revenue_usd
            ELSE
                0
        END
    ) AS revenue_usd
FROM transactions_cleaned
GROUP BY 1, 2
```

#### Metrics layer
There is still a problem: any time that we want to write a new query that involves the revenue, we need to copy this complicated "CASE/WHEN" logic into that query. Each different type of aggregation -- whether by player, lesson, date, or combinations thereof -- will require this logic. Thus, although we've solved the problem of computing the revenue, we've set ourselves up for a potential maintenance headache. Our code is the opposite of [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

There are a few ways to address this issue, although none of them seems particularly good to me. 

(This problem was also discussed by Benn Stancil and his solution is a ["metrics layer"](https://benn.substack.com/p/metrics-layer), which seems theoretically correct to me although I have yet to see a practical implementation. Others refer to this problem as ["Headless BI"](https://basecase.vc/blog/headless-bi))

1. We can rewrite the `revenue_usd` column in the view, like so:

{% raw %}
```sql
{{
    config(materialized='view')
}}

SELECT
    player_id
    ,event_at
    ,lesson_id
    ,(CASE
        WHEN COALESCE(analytics_version, '1970.01.01') < '2021.08.10' AND COALESCE(event_name, 'OldTransaction') = 'OldTransaction'
            -- note: this is only approximate, because of the error
            -- we might try applying a discount factor to the revenue,
            -- based on the comparison of OldPurchase and ValidPurchase in period 2
            -- in order to fix this
            THEN revenue_usd
        WHEN analytics_version >= '2021.08.10' AND event_name = 'ValidTransaction'
            THEN revenue_usd
        ELSE
            0
    END) AS revenue_usd
    ,app_version
    ,COALESCE(event_name, 'OldTransaction') AS event_name
    -- some arbitrarily old date
    ,COALESCE(analytics_version, '1970.01.01') AS analytics_version
FROM {{ source('transactions') }}
```
{% endraw %}

This strikes me as being wrong aesthetically, although I'm not able to fully articulate why. Moreover, it does not allow us to run the analysis comparing the revenue from the old and new events during period 2: the revenue from the former will be zeroed out.

2. We can encode this definition into LookML. This works as far as Looker goes, but it is not accessible outside of the Looker environment.

3. We can have the logic exist in `dbt`. This works only if queries need only access the aggregated/transformed data created by `dbt`. In my experience, this is not always the case.

4. We can accept that the logic will not be centralized, and manually update our queries when a change to the revenue definition is made. This becomes easier if the queries are source controlled and easily deployable. This is not the case for most BI tools (Tableau, etc.), and, as the list of places that queries live becomes more expansive -- Jupyter notebooks/[knowledge repositories](https://airbnb.io/projects/knowledge-repo/), BI tools, data apps, ML models, dbt, etc. -- this solution becomes increasingly untenable. To take one example, suppose we conduct an analysis in a Jupyter notebook and deploy it to a knowledge repository to make it available to the entire company. What happens if the data model changes and the queries residing within the notebook break semantically? How do we ensure that our findings remain reproducible?

### Summary and conclusions

To conclude:

1. Keeping queries synchronized with changes to the data model can be an onerous task. Analysts tasked with stuff they don't enjoy doing (query maintenance) instead of stuff they do (deriving data insights) are also likelier to leave.
2. The goal of an analytics team should be to make syntactic changes to the data model trivial to implement, and semantic changes relatively easy. Unless this is the case, old code, outdated attribute names and events, and undesirable ways of instrumenting the application will linger, creating "analytics debt" that will only build up with time.
3. We do not yet have the tools to solve this problem completely, or even well. The fundamental problem is that for semantic changes to the data model, like our new transaction event, there is no single place (a "metrics layer") to encode the logic for a metric/aggregation that is easily accessible by all downstream queries.
4. That being said, using a view layer to encapsulate 
the raw source data is one good approach for making updates related to syntactic changes much less work to implement.
5. As our case study revealed, dealing with event data can be tricky because of backwards compatibility issues. Incorporating an `analytics_version` can make it possible to compute historical metrics that otherwise would not be.
6. Making analytics conform more to practices from software engineering (source control, one-button deployment) can ameliorate the issues described above partly, but not wholly. Many of the leading BI tools, such as Tableau, do a poor job of this. This partly reflects the historical divisions between engineering and analytics: divisions that I feel are somewhat artificial.
