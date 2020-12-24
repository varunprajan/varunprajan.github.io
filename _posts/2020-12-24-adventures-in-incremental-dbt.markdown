---
layout: post
title:  "Adventures in incremental dbt"
date:   2020-12-24 23:07:59 -0500
categories: jekyll update
---
(For those of you who don't know what `dbt` is or how it works, parts of this post might be confusing. I'd recommend starting with the documentation and working through an example to get your feet wet. That being said, if you know SQL and a little bit of Jinja, most of what I write below should make sense.)

## Introduction

I love `dbt`. It is a tool for "ELT", as opposed to "ETL": in other words, transforming the "source" data in your data lake/data warehouse after it has been loaded, as opposed to before.

(The reasons you might want to adopt the ELT paradigm are too involved to explain in this post, but one main one is that transforming the source data *before* it arrives in your data warehouse is often irreversible and therefore inflexible. By contrast, transforming the raw data *after* is reversible: if you decide that a different data model is appropriate, you can simply wipe away the previously generated "transformed data" and start anew from the source data.)

A [few different trends](https://blog.getdbt.com/future-of-the-modern-data-stack/) have driven the rapid adoption of `dbt`. First, the relative cheapness of storage means that storing source data in a raw, unaggregated form is economical. Second, the rise of data warehouses such as Redshift and Snowflake means that this SQL "transformation layer" is both easy to write and performant on large datasets. Redshift and Snowflake support distributed/parallel SQL in a way that traditional "OLTP" databases, such as Postgres, do not.

Typically, `dbt` is run as a nightly cron job: each night, it will execute a set of SQL queries against the raw source data and produce the "derived" or "transformed" data, which are the tables that support your analysts, dashboards, and possibly even ML applications. This process might strike you as inefficient, particularly for source data that comprises (stateless) events. If only the source data loaded in the last 24 hours has changed between the prior `dbt` run and the current one, why not simply process that data instead of reprocessing the entire table?

For this purpose, `dbt` supplies the concept of "incremental" materialization: instead of reprocessing the entire source table, we can process the *increment* of new data. In this post, I will use a case study to demonstrate how this works and some problems I've run into.

## A case study

Suppose we have a table called `user_transactions`. It captures the revenue for each transaction (say, buying an item on our e-commerce store) associated with a user. It might look like this:

| user_id | birth_date | date | revenue |
| :-----: | :--------: | :--: | :-----: |
| 1 | 2020-12-01 | 2020-12-01 | 2.00 |
| 1 | 2020-12-01 | 2020-12-01 | 2.00 |
| 1 | 2020-12-01 | 2020-12-03 | 6.00 |
| 1 | 2020-12-01 | 2020-12-07 | 14.00 |
| 1 | 2020-12-01 | 2020-12-08 | 5.00 |
| 2 | 2020-12-02 | 2020-12-05 | 6.00 |

Here, the `user_id` is the unique identifier for the user; the `date` is the date of activity (the date the transaction occurred); the `birth_date` is the date the user joined our service (was "born"), and the `revenue` is the dollar amount collected. We can, of course, have multiple transactions for a user on a particular date.

(One subtle point: Note that this representation is denormalized, since the `user_id` + `birth_date` relationship is fixed, and could/should be a separate table. In other words, we could have a separate `users` table, of which the `birth_date` is a dimension. The `users` table would be a "dimension" table, [in the language](https://stackoverflow.com/questions/20036905/difference-between-fact-table-and-dimension-table) of data modeling, and the original `user_transactions` table would become a "fact" table. In what follows, however, I've chosen to keep this denormalized representation because it makes writing the subsequent SQL easier.)

Our goal in this case study is to compute the cumulative revenue per user for various time windows: specifically, 3, 7, 60, and 365 days after the user joined our service. We refer to the cumulative revenue up to day N as the "DN revenue". For the example data above, user 1's D1 revenue is $20.00; D2 revenue is $30.00, and D4 revenue is $81.00. In contrast, user 3's D1 and D2 revenue is $0.00 (they did not spend in the their first 2 days), but their D4 revenue is $30.00.

(Another subtle point: some numbering conventions use 0-based indexing, so that our "D1 revenue" would be equivalent to their "D0 revenue". Be attentive to this "off-by-one" issue when writing your own business logic.)

### Attempt 1: full table refresh

The simplest approach is to create a derived table (say, `user_cumulative_revenue`), and to refresh the entire table each day. The SQL would look like this:

{% raw %}
```sql
SELECT
    user_id
    ,birth_date
    ,SUM(
        CASE
        WHEN DATEDIFF(DAY, birth_date, date) < 3
        THEN
            revenue
        ELSE
            0
        END
    ) AS d3_revenue
    ,SUM(
        CASE
        WHEN DATEDIFF(DAY, birth_date, date) < 7
        THEN
            revenue
        ELSE
            0
        END
     ) AS d7_revenue
     ...
FROM {{ ref('user_transactions') }}
GROUP BY 1, 2
```
{% endraw %}

(where `ref` is `dbt`'s way of referring to other tables/data sources. Note that [source](https://docs.getdbt.com/docs/building-a-dbt-project/using-sources/) might be even better than `ref` here.)

Using `dbt`'s jinja functionality, we can clean up the repetitive logic like so:

{% raw %}
```sql
-- attempt_1.sql
{% set day_windows = [3, 7, 60, 365] %}

SELECT
    user_id
    ,birth_date
    {% for day_window in day_windows -%}
    ,SUM(
        CASE
        WHEN DATEDIFF(DAY, birth_date, date) < {{ day_window }}
        THEN
            revenue
        ELSE
            0
        END
     ) AS d{{ day_window }}_revenue
     {% endfor -%}
FROM {{ ref('user_transactions') }}
GROUP BY 1, 2
```
{% endraw %}


There's still one error in our logic, however. For day windows that are "incomplete" — for instance, if a user has been with us for only 2 days but we want to find their D3 revenue — we should make the result `NULL`. We want to calculate the D3 revenue only when the 3 day window for a particular user is complete. We can incorporate that logic like so:

{% raw %}
```sql
-- attempt_1_fixed.sql
{% set day_windows = [3, 7, 60, 365] %}

SELECT
    user_id
    ,birth_date
    {% for day_window in day_windows -%}
    ,(
        CASE
        WHEN DATEDIFF(DAY, birth_date, CURRENT_DATE) >= {{ day_window }}
        THEN
            SUM(
                CASE
                WHEN DATEDIFF(DAY, birth_date, date) < {{ day_window }}
                THEN
                    revenue
                ELSE
                    0
                END
            )
        ELSE
            NULL
        END
    ) AS d{{ day_window }}_revenue
    {% endfor -%}
FROM {{ ref('user_transactions') }}
GROUP BY 1, 2
```
{% endraw %}

(Lots of nesting! This could be simplified if your database supports the `IFF` and/or `NULLIF` keywords.)

In other words, we set the DN revenue to be `NULL` if insufficient time has elapsed between the user's `birth_date` and the current date.

(One more subtle note: I'm assuming that the last *complete* day of data is `CURRENT_DATE - 1` . This is true if the `dbt` job runs somewhat after midnight UTC and our data is loaded in a timely way. In practice, this assumption might be dangerous, however, and data ["freshness"](https://docs.getdbt.com/reference/commands/source#dbt-source-snapshot-freshness) checks are probably needed.)


### Attempt 2: incremental, naive

The approach developed above "works", but you might notice that it's rather inefficient. First of all, even though our maximum LTV window is 365 days, our query scans over the  entire `user_transactions` table, even for users who joined more than 365 days ago, and for transactions that occurred more than 365 days ago. If we were to filter out these users and transactions, however, then the data for them would not appear in the final table, which is bad. What we need is a way to update the data for the last 365 days without destroying the data from dates before that. `dbt`'s default approach is to destroy the previous table and create a new one, which isn't appropriate in this case.

Enter the `dbt` incremental materialization. Here, we have to define a key or set of keys for "upserting" new data. If the keys for the new rows match those for existing rows in the table, the existing rows are overwritten. If the keys do not match, the new rows are inserted. This might seem confusing, but the basic idea, as applied to this case study, is that our transformed data has a unique id: we have one row per user. We only want to update rows for which the user's `birth_date` is in the last 365 days; for older users, their data need not be recomputed, since it is fixed and should not change.

The SQL looks something like this:

{% raw %}
```sql
-- attempt_2.sql
{{
    config(
        materialized='incremental',
        unique_key='user_id'
    )
}}

{% set day_windows = [3, 7, 60, 365] %}

SELECT
    user_id
    ,birth_date
    {% for day_window in day_windows -%}
    ,(
        CASE
        WHEN DATEDIFF(DAY, birth_date, CURRENT_DATE) >= {{ day_window }}
        THEN
            SUM(
                CASE
                WHEN DATEDIFF(DAY, birth_date, date) < {{ day_window }}
                THEN
                    revenue
                ELSE
                    0
                END
            )
        ELSE
            NULL
        END
    ) AS d{{ day_window }}_revenue
    {% endfor -%}
FROM {{ ref('user_transactions') }}
{% if is_incremental() %}
WHERE
    DATEDIFF(DAY, birth_date, CURRENT_DATE) < {{ day_windows|max }}
    -- superfluous, but might help query optimizer
    AND DATEDIFF(DAY, date, CURRENT_DATE) < {{ day_windows|max }}
{% endif %}
GROUP BY 1, 2
```
{% endraw %}

Pretty simple, right? The only things that have changed are the configuration of the `dbt` model and the date filters (in the `WHERE` clause). We can run this in `full-refresh` mode initially, which will create the entire table from scratch by ignoring the `is_incremental` WHERE clause; then, on all subsequent runs, we can run this model in incremental mode, which will respect the `is_incremental` `WHERE` clause and avoid reprocessing data for old users. For more details on how this works, here is the [dbt documentation](https://docs.getdbt.com/docs/building-a-dbt-project/building-models/configuring-incremental-models/) for incremental models.

(As an aside, this incremental query will be more efficient only if the source data is sorted/indexed appropriately. In Redshift, we would need to have a [sort/dist key](https://www.flydata.com/blog/amazon-redshift-distkey-and-sortkey/) that involves either the `date` and/or the `birth_date`. In Snowflake, these keys/indexes are managed for you, and it is likely that this optimization would take place. If we had separated the source data into dimension and fact tables, the choice of keys would be somewhat different.)

### Attempt 3: incremental, more sophisticated

It turns out that even this incremental model is rather inefficient, in the sense of scanning over rows that don't matter. How inefficient? Well, each day we have only 4 `birth_date` "cohorts" that mature. As an example, let's suppose we are processing data on Jan 1, 2020, and the last date of data available is for Dec 31, 2019. There are four important milestones that have been reached:

1. The cohort with `birth_date` Dec 29, 2019 has completed 3 days of activity, so its D3 revenue can be computed.
2. The cohort with `birth_date` Dec 25, 2019 has completed 7 days of activity, so its D7 revenue can be computed.
3. The cohort with `birth_date` Nov 2, 2019 has completed 60 days of activity, so its D60 revenue can be computed.
4. The cohort with `birth_date` Jan 1, 2019 has completed 365 days of activity, so its D365 revenue can be computed.

Importantly, no other cohorts have matured or hit an important milestone! In other words, if we reprocess the last 365 days of `birth_date`s, we will be scanning over 365/4 = 91 times as many rows as is necessary. (Whether this means the query will take 91 times as long is a separate question; it is unlikely that the performance degradation would be that severe.)

You might argue that modern distributed SQL is fast enough that this doesn't matter. This is probably true for many applications. But, depending on our business model and number of users, the `transactions` table might have millions or tens of millions of rows per day, and reprocessing 365 days of it might be too inefficient.

Regardless, even merely as an intellectual exercise I think it is interesting to see whether we can make this logic more performant, working within the strictures of `dbt`.

Let's try to directly implement this idea related to cohort maturation. On a given date, we will reprocess only the four `birth_date` cohorts that have matured. My attempt looks like this:

{% raw %}
```sql
-- attempt_3.sql
{{
    config(
        materialized='incremental',
        unique_key='user_id'
    )
}}

{% set day_windows = [3, 7, 60, 365] %}

SELECT
    user_id
    ,birth_date
    {% for day_window in day_windows -%}
    ,(
        CASE
        WHEN DATEDIFF(DAY, birth_date, CURRENT_DATE) >= {{ day_window }}
        THEN
            SUM(
                CASE
                WHEN DATEDIFF(DAY, birth_date, date) < {{ day_window }}
                THEN
                    revenue
                ELSE
                    0
                END
            )
        ELSE
            NULL
        END
    ) AS d{{ day_window }}_revenue
    {% endfor -%}
FROM {{ ref('user_transactions') }}
{% if is_incremental() %}
WHERE
    DATEDIFF(DAY, date, CURRENT_DATE) < {{ day_windows|max }}
    AND (
        {% for day_window in day_windows -%}
        DATEDIFF(DAY, birth_date, CURRENT_DATE) = {{ day_window }}
        {% if not loop.last -%}
        OR
        {% endif %}
        {% endfor -%}
    )
{% endif %}
GROUP BY 1, 2
```
{% endraw %}

which is starting to get more complicated! We have to modify the `WHERE` clause to filter using equality statements, chained with OR, for the `birth_date`.

## Unpivoting

One source of inflexibility with our current approach, which you might have noticed, is that adding new "day windows" is inefficient: we have to run a full table refresh for any new columns. In other words, if we want to compute a new, D1 cumulative revenue, the incremental materialization doesn't help us.

One way to surmount this difficulty is to ["unpivot"](https://github.com/fishtown-analytics/dbt-utils#unpivot-source) the table: for each user, we will have N rows, one for each day window of interest. Then, when we need to add a new day window, all we need to do is insert a new row for each user, and this operation does not require a full table refresh. This might be a bit tough to visualize, so here is what the old table looked like:


| user_id | birth_date | d3_revenue | d7_revenue | d60_revenue | d365_revenue |
| :-----: | :--------: | :--------: | :--------: | :---------: | :----------: |
| 1 | 2020-12-01 | 10.00 | 24.00 | NULL | NULL |
| 2 | 2020-12-02 | 0.00 | 6.00 | NULL | NULL |

and this is what the new, pivoted table looks like:

| user_id | birth_date | day_window | revenue |
| :-----: | :--------: | :--------: | :-----: |
| 1 | 2020-12-01 | 3 | 10.00 |
| 1 | 2020-12-01 | 7 | 24.00 |
| 2 | 2020-12-02 | 3 | 0.00 |
| 2 | 2020-12-02 | 7 | 6.00 |


The unique key for upsertion, in this case, is the combination of the `user_id` and the `day_window`. How might this model be represented in `dbt`? It's not too different from our previous model:

{% raw %}
```sql
-- attempt_pivot_incremental.sql
{{
    config(
        materialization='incremental',
        unique_key='user_day_window_id'
    )
}}

{% set day_windows = [1, 3, 7, 60, 365] %}

WITH tmp AS (
SELECT
    user_id
    ,birth_date
    ,DATEDIFF(DAY, birth_date, CURRENT_DATE) - 1 AS day_window,
    ,SUM(revenue) AS cumulative_revenue
FROM {{ ref('user_transactions') }}
WHERE
    -- select birth_dates that correspond to cohort maturation on CURRENT_DATE - 1
    (
        {% for day_window in day_windows -%}
        DATEDIFF(DAY, birth_date, CURRENT_DATE) = day_window
        {% if not loop.last -%}
        OR
        {% endif %}
        {% endfor -%}
    )
    -- superfluous, but might help query optimizer
    AND date >= birth_date
    AND DATEDIFF(DAY, date, CURRENT_DATE) < {% max(day_windows) %}
GROUP BY 1, 2
)

SELECT
    {{ dbt_utils.surrogate_key('user_id', 'day_window') }} AS user_day_window_id,
    *
FROM tmp
```
{% endraw %}

When we turn to the "full table" model, things get complicated, though. It is not obvious how to implement this model (without using the `unpivot` macro), which means that we cannot easily have the incremental and full models in the same model file, which `dbt` requires. (If you have ideas, let me know!)

(My best guess is that the solution involves some complicated SQL/Jinja to generate all possible combinations of (`day_window`, `birth_date`) tuples, and then joining that to the `user_transactions` table. In addition to being complicated, this might also be inefficient.)

## Taking a step back

Even if there is a solution to this issue, I think the difficulties we had constructing it point to some deficiencies with dbt:

1. Writing incremental models involves reasoning carefully about the differences between the full and incremental models, and trying to include both sets of logic in a single model. This often requires SQL/Jinja gymnastics, if it is indeed possible at all.
2. We often don't even need the full table model. We could, instead, compose the table of interest by iterating over individual incremental runs. `dbt` does not easily support this functionality, though. "For" loops must be written within model files; they cannot be used to iterate over `dbt` runs.

Let me try to expand upon point 2. One relatively straightforward approach would be to run the dbt model for a single tuple of (`day_window`, `date_of_interest`). How might this look? (I'm using the [vars syntax](https://docs.getdbt.com/docs/building-a-dbt-project/building-models/using-variables/) to pass these variables into the model.)

{% raw %}
```sql
-- attempt_single_tuple_incremental.sql
{{
    config(
        materialization='incremental',
        unique_key='user_day_window_id'
    )
}}

WITH tmp AS (
SELECT
    user_id
    ,birth_date
    ,{{ var("day_window") }} AS day_window
    ,SUM(revenue) AS cumulative_revenue
FROM {{ ref('user_transactions') }}
WHERE
    DATEDIFF(DAY, birth_date, '{{ var("date_of_interest") }}' ) = {{ var("day_window") }}
    AND date <= '{{ var("date_of_interest") }}'
GROUP BY 1, 2
)

SELECT
    {{ dbt_utils.surrogate_key('user_id', 'day_window') }} AS user_day_window_id,
    *
FROM tmp
```
{% endraw %}

There's actually hardly any Jinja in this solution at all! On a particular date of interest (which we had taken to be synonymous with the current date, but could really be any date at all), we can get the desired `birth_date` cohort by subtracting the `day_window` from that date. For instance, for a date of interest of Jan 1, 2020, and a `day_window` of 365, the `birth_date` is Jan 1, 2019. Then, we simply grab all the transactions that occurred before the date of interest, for users with that birth date. Each `dbt` run would be very fast to execute, particularly for short day windows, but there would be a *lot* of runs: N_dates * N_day_windows.

Note further that, in theory, all of these runs could be executed concurrently. Because each of them generates a separate set of rows (since each deals with a distinct `user_day_window_id`), none of the runs should, again in theory, interfere with one another. This could make the "backfill" (the initial run to build the table from scratch from historical data) rather fast to execute.

This approach is very similar to the "functional" approach to transformations, discussed [here](https://medium.com/@maximebeauchemin/functional-data-engineering-a-modern-paradigm-for-batch-data-processing-2327ec32c42a). Each dbt run would process a chunk of data and generate a disjoint "partition", indexed by the `user_day_window_id`. The final table can be thought of as the UNION ALL of each of these different `user_day_window` partitions.

Another benefit of this approach is supporting "late arriving" events: in this case, transactions for which the data is received one or more days late, or cases where a transaction is refunded, so we have an additional, negative transaction that arrives one or more days later. We can incorporate these late arriving events into our logic by also running the dbt job for a previous date of interest (say, 7 days ago, by which time we can assume all of the late arriving events have "settled"). This would ensure that we have approximate results immediately, and correct results with a 7 day lag. Again, the normal job and the late arriving job could conceivably run concurrently.

In practice, on the other hand, implementing this solution is not at all easy:

- Multiple `dbt` jobs can be run concurrently, but it is not safe to do so. In particular, the names of intermediate VIEWs might clash, which would lead to race conditions and other bad outcomes.
- Databases like Redshift struggle with concurrent writes because of serializability guarantees. Telling the compiler that the partitions being generated are indeed disjoint is not easy.
- `dbt` does not natively support looping over runs/tasks and supplying "logical dates" in the way that a workflow management tool like [Airflow](https://airflow.apache.org/) does. dbt has to be combined with a tool like Airflow to get this functionality. We need the logical date to support things like backfills, retries, table rebuilds using incremental logic alone, and late arriving events. (We can use Airflow's `ds` as dbt's `var("date_of_interest")`.)

(Others have encountered related issues: [Shopify mentioned](https://shopify.engineering/build-production-grade-workflow-sql-modelling) that "dbt’s current incremental support doesn’t provide safe and consistent methods to handle late arriving data, key resolution, and rebuilds. For this reason, a handful of models (Type 2 dimensions or models in the 1.5B+ event territory) that required incremental semantics weren’t doable—for now.")

It would be nice to see `dbt` try to support this more functional approach to writing transformations. I imagine it will be difficult given the issues I've mentioned, particularly on the database side, but I'm curious to see what their team of very smart engineers can cook up!

(The Github code for this project is available [here](https://github.com/varunprajan/data-eng-blog/tree/main/incremental-dbt), if you're interested. The `user_transactions` table can be generated using `dbt seed`.)
