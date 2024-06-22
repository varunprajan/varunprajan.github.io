---
layout: post
title:  "Why no one writes unit tests for data"
date:   2023-10-31 23:07:59 -0500
categories: jekyll update
---

## Introduction

Back in grad school, I used to write objectively terrible code: hundred-plus line functions, spaghettified logic, no separation of concerns, and whatever the opposite of DRY is ([WET?](https://betterprogramming.pub/when-dry-doesnt-work-go-wet-6befda0444bf)). Worst of all, I never wrote tests. Without tests, it is difficult to iterate on your codebase, let alone quickly. Testing ensures that, when you refactor existing code, it continues to perform as it had before. It also makes sure that, when you add new features, they behave as intended and, moreover, don’t break existing functionality.

Tests for software are often classified into [four categories](https://dev.to/lucaspaganini/static-unit-integration-and-end-to-end-tests-explained-5g62): static, unit, integration, and end-to-end. Static tests, such as type checking, run without needing to execute the code at all. Unit tests verify the functionality of individual functions: the units or building blocks of a codebase. And integration and end-to-end tests ensure that larger groups of code, or the codebase as a whole, work as intended.

When I entered the world of data pipelines (both SQL and otherwise), I found the lack of a similar taxonomy confusing. Part of the confusion stems from mistaking testing tools --- and the vendors selling them --- with a testing philosophy. dbt, probably the most popular tool for writing SQL-based data pipelines, includes support for data tests, but testing in dbt is not identical to testing data. One is a particular approach; the other is a generic philosophy or set of principles. Similarly, we should not confuse a tooling stack, like “dbt + great expectations + Monte Carlo”, with a generic framework for thinking about data testing. The purpose of this article is to describe, in a tool-agnostic way, this framework. (But, since examples are helpful, I also show how this framework can be implemented using particular tools.) Along the way, I also try to answer the (clickbaity) question posed by this article’s title — why is it that no one seems to write unit tests for data?

## A taxonomy of data tests

I argue that there are two “dimensions” of data testing. The first is related to the “type” of data being tested; the other has to do with how we assess its “correctness”.

Let’s begin with the data “type”. Recall that data pipelines are inherently “functional” in nature: they take some set of inputs, and produce some set of outputs. In data testing, we can either test the logic of that function, or we can test the correctness of the inputs themselves. This is one crucial way in which data testing differs from software testing! In software testing, we usually just assume that the inputs are valid to begin with, and simply test the logic, which is equivalent to testing the correctness of the outputs. In data testing, we can’t afford that assumption; we have to test both inputs and outputs.

If we decide to test the functional logic, we have a further bifurcation. We can either test this logic with “real” inputs (at the scale of thousands or even millions of rows), or “mocked” or “fake” ones (at the scale of just a few rows). The latter is akin to unit testing in traditional software engineering. The former has no real analogue.

The second dimension, as I mentioned, is data “correctness”. Again, there is one obvious fork: do we know the “right” answer, or not? In some cases, we obviously do: in particular, if we are testing the logic of a data pipeline, and we supply “fake” (mocked) inputs, we should be able to write out our “fake” expected output (although it might be quite tedious to do so). I will call this a “unit test” for data. There is one other important case here: where we have some existing logic and want to refactor it. Here, we know that the outputs before the refactor must match that after, even if that output is too large to write down in a proper unit test. So, this is a different type of test: one that I will call a “data diff” test. (The idea is that the difference of these datasets should be empty.)

More common, though, is the case where we do not know the “right” answer. (This is yet another way in which data testing differs from software testing!) Instead, we can only make weaker statements about correctness. I argue that there are two main types of these “weaker” tests: “constraint-based” and “statistic-based”. Constraint-based testing ensures that all rows in a dataset, whether in an input table _or_ an output table, satisfy certain conditions (”constraints”). Examples include testing “non-NULLness” of a single column, ensuring that an “enum”-based column conforms to a set of expected values, or even tests across multiple tables, such as that all values of a foreign key in one table match a primary key in another table. Statistic-based testing is an even weaker guarantee. It simply ensures that certain statistics of the dataset are what we expect. For example, the row count of a date-partitioned table should not have a large discontinuity day-over-day. Or, the distribution of values for an enum-based column should not differ too much from our expected distribution. Typically the way this is checked is probabilistic: by constructing a time-series forecast, and detecting anomalies (values outside of the range of the prediction interval), or by doing some sort of statistical test. Unsurprisingly, statistic-based data tests are also much more difficult to implement, and they tend to be “noisier” --- more false positives --- as well.

I have spent a lot of words describing this taxonomy, and a table might be easier to comprehend.

|  | Unit test | Data diff test | Constraint-based test | Statistic-based test |
| --- | --- | --- | --- | --- |
| Testing logic, fake inputs | Unit testing | — | — | — |
| Testing logic, real inputs | — | Data diff test | Constraint-based (outputs) | Statistic-based (outputs) |
| Testing inputs | — | — | Constraint-based (inputs) | Statistic-based (inputs) |

Each row represents a possible “type” of data being tested, and each column represents a possible way in which “correctness” is assessed. Some combinations thereof don’t make sense, and are marked with —. For example, suppose we are testing our pipeline logic using fake inputs. In this case, we can write out our expected output, and there is no point doing further constraint-based, statistic-based, or data diff testing. As another example, unit tests don’t really work on “real” inputs, as opposed to “fake”/mocked inputs, since writing the expected output is impossible.

We’re left with 4 types of tests: unit, data diff, constraint-based, and statistic-based, which, in an odd coincidence, matches the number of test types for software testing. But these types of tests are very different from those in software.

Most tools in the data/analytics space fail to implement all types of data tests. Dbt provides support only for constraint-based testing, although unit testing [is now being considered](https://github.com/dbt-labs/dbt-core/discussions/8275). DataFold maintains the nice [data-diff package](https://github.com/datafold/data-diff), but that is only for data diff tests. [SQLMesh](https://sqlmesh.com/), a somewhat new product, makes a much better effort, and [implements three](https://tobikodata.com/we-need-even-greater-expectations.html) out of four test types, all except statistic-based testing. (It refers to constraint-based tests as "audits".) At my company, we have separate solutions for anomaly detection on top of datasets (essentially, statistic-based tests), for unit tests, constraint-based tests, and data-diff tests. Each solution is built in-house, usually by different teams! The space is clearly very fragmented. In the future, I expect things to consolidate somewhat, and platforms like [Monte Carlo](https://www.montecarlodata.com/) to provide a unified view of these tests and of data quality as a whole (what they term [“data observability”](https://www.montecarlodata.com/blog-testing-data-pipelines/))

## A case study

### Background 

I have been quite abstract so far, and now I want to talk through some examples, wherein we apply this testing framework to actual data. Along the way, we will learn about why unit testing is not nearly enough for data (despite the protestations of software engineers), the strengths and weaknesses of each of these testing techniques, and why we really need several of them to feel at least somewhat confident about the quality of our data.

Let’s assume we’re working at a mobile game studio that makes just one very successful game. (I used to work at one of these companies!) For this game, we have just two tables. One is an events table of “game sessions”: an event fires each time a user opens the game on their phone and starts playing. It looks like this:

| user_id | country | country_region | session_start_time | app_version |
| --- | --- | --- | --- | --- |
| 123 | US | North America | 2023-10-31 02:03:04 | 8.12.0 |
| 234 | SE | Europe | 2023-11-01 14:00:01 | 7.1.1 |
| 234 | SE | Europe | 2023-11-01 17:00:02 | 7.1.1 |

 We also have a user metadata table. It records, for each user, when they first started playing our game:

| user_id | start_date |
| --- | --- |
| 123 | 2022-11-01 |
| 234 | 2023-10-02 |

The goal is to produce a dataset of aggregates, broken out by cohort + region + date. The “cohort” is defined as the month in which the user started (inferred from `start_date`), and the region is the region in which they played our game. We want to report on metrics like “active users” per day and the average number of sessions played per user per day.

Our (very simple) SQL pipeline might look like this:

```sql
SELECT
    game_sessions.country_region,
    TO_CHAR("%Y-%m", user_metadata.start_date) AS cohort_start_month,
    TO_CHAR("%Y-%m-%d", game_sessions.session_start_time) AS date,
    COUNT(DISTINCT game_sessions.user_id) AS active_users,
    COUNT(*)/COUNT(DISTINCT game_sessions.user_id) AS avg_sessions_per_user
FROM game_sessions
INNER JOIN user_metadata
    ON user_metadata.user_id = game_sessions.user_id
GROUP BY 1, 2, 3
```

This is pretty compact and clear! We could make it more efficient by using [“incremental” logic](https://docs.getdbt.com/docs/build/incremental-models) on the `date`, but this is good enough for now.

Let’s write a unit test for the logic of this data pipeline. It is pretty simple: with the input data given (which we can take as our “fake”/mocked data), the output should be:

| country_region | cohort_start_month | date | active_users | avg_sessions_per_user |
| --- | --- | --- | --- | --- |
| North America | 2022-11 | 2023-10-31 | 1 | 1 |
| Europe | 2023-10 | 2023-11-01 | 1 | 2 |

We could perhaps add some complexity to the “fake” input data (multiple users from the same region, multiple users on the same day, multiple users from the same cohort month, etc.) to cover some additional cases. But there are, of course, diminishing returns to such complexity.

Consider now the following scenarios:

### Scenario 1

Our client team realizes it can slim down the event by removing the `country_region` field. After all, there is a static mapping between `country` and `country_region`, and the data team can maintain the mapping on its side. Our refactored pipeline might look like:

```sql
SELECT
    country_map.country_region,
    TO_CHAR("%Y-%m", user_metadata.start_date) AS cohort_start_month,
    TO_CHAR("%Y-%m-%d", game_sessions.session_start_time) AS date,
    COUNT(DISTINCT game_sessions.user_id) AS daily_active_users,
    COUNT(*)/COUNT(DISTINCT game_sessions.user_id) AS avg_daily_sessions_per_user
FROM game_sessions
INNER JOIN user_metadata
    ON user_metadata.user_id = game_sessions.user_id
LEFT JOIN country_map
    ON country_map.country = game_sessions.country
GROUP BY 1, 2, 3
```

Our unit test passes (you can try it yourself!), and we call it a day. Everything seems fine...but is it?

Suppose that the mapping used to convert between `country` and `country_region` on the client was different than the mapping in the data team’s `country_map` table. For example, the one the data team pulled might be out of date, missing rows for newer countries like South Sudan. Or it might have different mappings for countries in multiple regions (like Turkey, which straddles Europe and Asia). Our unit test would catch these problems only if we had users in our input tables corresponding to South Sudan or Turkey. Taken to the extreme, we’d need hundreds of rows in our mock input data, each one corresponding to a different country, in addition to all the other edge cases we have to consider. Is there a better way?

As it turns out, yes! We have two complementary possibilities. We can implement a data diff test, since, in theory, we expect the same output before and after this refactor. We can also implement a constraint test on the inputs, to ensure that each value for `country` in `game_sessions` is also present in our `country_map` mapping table, or a constraint test on the outputs, to ensure that `country_region` is never NULL (since we're using a `LEFT JOIN`). The constraint-based test would catch the case of missing countries, like South Sudan, but not the case of ambiguous regions, like for Turkey. The data diff test would catch both, although, with this test, it would be unclear if the problem were in the old logic or in the new logic.

This scenario illustrates some of the fundamental weaknesses of unit testing for data. Constructing a fake dataset that captures the complexity of real data is incredibly tedious. The example being considered here is relatively simple, and involves just a few columns in the input table. But real datasets often have dozens or even hundreds of columns, many of which are higher cardinality than `country`. The advantage of doing data testing using real data is that (most of) these cases are covered automatically, although it’s worth noting that fake data can help cover gaps that exist in real data (suppose our game has launched only in North America, and we want to future-proof ourselves for worldwide expansion).

With unit tests for data, it is typically unclear what to mock and what to leave alone. `game_sessions` should be mocked, since it is potentially an enormous table. But should `country_map`? If it is mocked out, and replaced with fake data, we would be even less likely to catch edge cases corresponding to new or ambiguous countries.

It is often said that unit tests should [only test public methods](https://softwareengineering.stackexchange.com/questions/380287/why-is-unit-testing-private-methods-considered-as-bad-practice) — the “contract” of our software with the outside world. For data pipelines, our contract is the output tables that are produced: the ones to be consumed by analysts and dashboards. In this sense, the intermediate "models" in dbt, or the intermediate processing functions in Scala/Spark, are private methods, and therefore *shouldn’t* be tested. There is a fundamental problem. If we test only output datasets (”public methods”), we might end up writing a very complicated unit test, one that has to mock over many inputs, and test many pieces of logic compounded together. On the other hand, if we do test intermediate logic (”private methods”), our tests will be much simpler, but any refactor of the overall logic will end up invalidating most of them. If our unit tests for data are tightly coupled to implementation details, investing in them might not be justified.

I hope this discussion provides an answer to the question, “why does no one write unit tests for data?”. It is not that they are a bad idea, necessarily. It’s just that their “recall” is poor (they don’t catch a lot of problems), and the level of investment is high, even for the bugs they do catch.

This scenario also reveals the limitations of relying on constraint-based testing alone, as `dbt` does. There are many data quality issues, such as the “ambiguous region” issue, that are very difficult to catch using constraint-based tests. They are much easier to catch using data diff tests, as discussed previously, or even using statistic-based tests (for example, we might detect a discontinuity in DAU by `country_region` if some countries were switched from one region to another, although this is not guaranteed).

### Scenario 2

Suppose, over time, we have rogue users, ones who hack the game and manage to emit tens of thousands of game session events per day. Our CEO sees the (gradual) uptick in `avg_daily_sessions_per_user` in our KPI dashboard and enthuses about how investing in AI has really paid off. How might we have caught this problem _before_ it got to the CEO?

This scenario is much trickier — and, I argue, much more representative of real problems with data quality. Here, the problem is not in our logic but in our input data. As such, it cannot be caught with unit tests or with data diff tests, as we can see by referencing the testing taxonomy table above. For unit tests in particular: if we mock over the data pipeline inputs, we cannot detect issues with these same inputs!

Unfortunately, even constraint-based testing is insufficient here. These rogue users aren’t violating any constraints: they are real users, with entries in the `user_metadata` table, so the primary-foreign key association is satisfied. And each game session event they emit satisfies the constraint-based tests we might write for `game_sessions`: that the `user_id` is not NULL, that the `country` is not NULL, that timestamp is valid, etc.

Our only recourse is statistic-based testing. And this is something that most companies (at least the ones I’ve worked at) don’t really do. It’s also easy to see why. I myself struggle to formulate a statistic-based test in this scenario, let alone implement it. The row count of the `game_sessions` dataset won’t necessarily increase discontinuously, provided that we have enough other users generating game sessions to partly drown out these rogue users. Our best bet is to look at the distribution of sessions by user, and see that something is amiss, but making this into an automated test requires some arbitrary thresholds or human judgment. Is 10 sessions in a day too high? Probably not. But 100 or 1000 might be. Setting the threshold too high reduces our recall, but setting it too low tanks our precision, and might lead to a bunch of noisy alerts. In fact, in my experience, the “test” most likely to catch this bug is when a data analyst does exploratory data analysis and notices something awry. But this is not automated data testing, and it doesn’t scale particularly well.

(Statistic-based testing can also run into other problems. Suppose we have an errant release, which contaminates our data temporarily, but then things go back to normal. How would we, in an automated way, exclude these anomalies that we know about from our prediction, in order to catch anomalies we don’t know about? Statistic-based testing is also often not sensitive enough to catch tiny bugs, like users in South Sudan being omitted from our DAU numbers.)

## Summary and conclusions

There are many more scenarios to consider, but I hope I’ve made my point by now. To recap:

1. We have four main types of data tests. Unit tests, data diff tests, constraint-based tests, and statistic-based tests. We arrived at this taxonomy by considering comprehensively what “type” of data we were testing, and how we were assessing its “correctness”.
2. Each type of test has very significant weaknesses! In particular, unit tests do not catch problems with the input data, since this is effectively mocked over, and they may introduce a high maintenance burden if they end up testing “private” methods. Even if they are limited to "public" methods (the output datasets), they may be very tedious to construct.
3. Most tooling in the data space covers only one or two types of tests. dbt, in particular, only supports constraint-based testing, which is a limitation that most practitioners don’t even realize. (Or, if they do, they might not realize that constraint-based testing doesn’t catch many data quality issues.)
4. We need easier ways to write statistics-based tests. These can be quite powerful, and can catch subtle bugs missed by the other approaches, but they are not at all easy to implement in an automated way, at least right now (and using open-source tools). I think this is an interesting domain in which statistics/data science and analytics/data engineering intersect! Can we apply the powerful statistical tools that have primarily been developed for other purposes to the problem of data quality itself?
