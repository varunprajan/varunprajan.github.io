## Introduction

Simple English is a simplified version of English, typically limited to a vocabulary of ~1500 words or fewer. It is intended for students and others new to English. English, infamously, has many related (but subtly different) words for the same concept, and Simple English intends to cut through this redundancy. At the cost of economy of expression (and some beauty), we get a language that is much easier to read and to learn than English proper. Compare, for example, the [Wikipedia page](https://en.wikipedia.org/wiki/Calculus) on calculus with the [Simple Wikipedia page](https://simple.wikipedia.org/wiki/Calculus).

The purpose of this post is to do for pandas what Simple English did for English. Pandas, like English, is notorious for having many ways to do roughly the same thing. It violates the [Zen of Python](https://peps.python.org/pep-0020/) precept that “There should be one — and preferably only one — obvious way to do it.” To take one example, there are [several methods](https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html) for accessing data within a pandas DataFrame: `.loc`, `.iloc`, `.at`, `.iat`, `.ix`, and attribute (`.column_name`) and indexing (`[column_name]`) operators.

Below is the set of methods and functions I feel you should learn as a pandas beginner. These focus on the “T” part of an ETL (extract, transform, and load) data pipeline: transforming one or many raw data sources into usable data. 

(As an aside, the details of the “extract” and “load” steps depend too heavily on individual use cases for me to be too prescriptive; as an example, you can extract data from an API, a CSV file, a local database (e.g., DuckDB), a cloud database (e.g., Bigquery) or a Parquet file in cloud storage. You can also load data to many of these same locations. Which one is appropriate depends on what resources you have access to and, to some extent, the size of the data.)

I provide “cheat sheet” that helps compare these methods to their SQL counterparts, which might be helpful for readers with a SQL background. There is also a list of methods *not* to learn, or to use only in exceptional cases. To return to the Simple English analogy, these would be like using the word “utilize” when “use” would suffice. I’m sure many will disagree with exactly which methods to include and exclude, but, as with Simple English, the cost of simplicity is (some) expressiveness. That being said, 90% of my work as a professional data scientist is based on the methods I suggest learning, and it is rare that I have to stray too far from them. It’s also worth mentioning that I almost never use the methods I suggest not to learn.

I also provide a set of coding “idioms” and best practices that you should follow. It is certainly possible to write correct and effective pandas without following these guidelines, but they are designed to make your code simpler, and therefore easier to read, write, and rewrite.

This is not the first attempt to write an opinionated guide on how to use pandas, but it is shorter (and hopefully simpler!) than other attempts. I highly recommend Matt Harrison’s “[Effective Pandas](https://hairysun.com/announcing-effective-pandas.html)” if you want the book-length treatment, however.

Finally, I provide some a code example, adapted from my professional work, showing how this fairly limited “vocabulary” can be chained together to construct effective (if not beautiful) code “sentences”. There is a working Jupyter notebook [here](https://github.com/varunprajan/data-eng-blog/blob/main/simple-pandas/simple_pandas.ipynb) with all the code in this post.

## The Cheat Sheet

There are two types of operations, those at the dataframe level, and those at the series level. The latter are typically passed into an `.assign` call and are not used on their own; more on this later. If I’m referring to a method on a dataframe, I will use the `.method_name` syntax; if I’m referring to a pandas or numpy function that applies to one or more dataframes, I’ll use the `pd.function_name` (or `np.function_name`) syntax.

### DataFrame level operations

| SQL Keyword | Pandas method(s) to learn | Pandas method(s) not to learn |
| --- | --- | --- |
| `SELECT` | Indexing operator (`[]`) <br /> `.assign` <br /> `.get` <br /> `.drop(columns=)` | Attribute operator (`.`) <br /> `.loc`, `iloc` <br /> `.at`, `.iat` <br /> `.ix` (deprecated) <br /> `.take` |
| `AS` | `.rename(columns=)` <br /> `pd.NamedAgg` |  |
| `WHERE/HAVING` | `.query` | `.eval` <br /> `.filter` <br /> `.dropna` <br /> `.isin` |
| `GROUP BY` (and certain window functions) | `.groupby.agg` <br /> `.groupby.transform` <br /> `.reset_index` |  |
| `ORDER BY` | `.sort_values` |  |
| `JOIN` | `.merge` | `.join` |
| `UNION ALL` | `pd.concat` |  |
| `DISTINCT` | `.drop_duplicates` |  |
| — (`dbt_utils.pivot`) | `.pivot_table` | `.pivot` <br /> `.crosstab` <br /> `.unstack` |
| — (`dbt_utils.unpivot`) | `.melt` | `.stack` |

(An aside: There are no proper analogues of `pivot_table` and `melt` in standard SQL, but in [`dbt`](https://www.getdbt.com/) world, there are the `pivot` and `unpivot` macros provided by [`dbt_utils`](https://github.com/dbt-labs/dbt-utils))

### Series level operations (passed into `assign`)

| SQL Keyword | Purpose | Methods to learn | Methods not to learn |
| --- | --- | --- | --- |
| `CASE WHEN`/`IF` | Conditionally applying transformations | `np.select` | `.where` <br /> `.mask` |
| `COALESCE` | Special case of `CASE WHEN`/`IF` | `np.select` | `.fillna` <br /> `.pad` <br /> `.combine` <br /> `.combine_first` |
| `CAST` | Type coercion | `.astype` | `.convert_dtypes` |
| `IS NULL` | Testing NULLness | `.isnull` <br /> | `.isna` <br /> `.notnull` <br /> `.notna` |
| — | Bucketing data | `.cut` <br /> `.qcut` |  |
| — | Element-wise mapping/replacement | `.replace` | `.map` <br /> `.apply` |
| — (many, such as `CONCAT` or `LIKE`) | Manipulating strings | `.str` |  |
| — (many, such as `AND` OR `NOT`) | Logical operators on boolean series | `&`, `~`, `\|`, etc.

## The Workhorse Methods

In total, there are 15 methods that operate at the level of the dataframe, of which 4 are what I term the “workhorse” methods: `assign`, `query`, `merge`, and `groupby`.

### Assign

`assign` takes a dataframe and returns a dataframe. It should be used for *assigning* the results of simple transformations (bucketing data, concatenating strings, ensuring correct types, etc.) to new columns. Some examples:

```python
# Taking a column and producing a new column
# creates a new column called "zipcode_str" padded with zeros on the left
new_frame = frame.assign(
    zipcode_str=lambda x: x["zipcode"].astype(str).str.zfill(5)
)

# Taking several columns and producing a new column
# creates a new column called "full_name"
new_frame = frame.assign(full_name=lambda x: x["first_name"] + " " + x["last_name"])

# Taking the results of a previous assignment and using it for a further manipulation
# creates new columns called "full_name" and "full_name_capitalized"
new_frame = frame.assign(
    full_name=lambda x: x["first_name"] + " " + x["last_name"],
    full_name_title=lambda x: x["full_name"].str.title(),
)

# Dynamically generating many new columns
# creates new columns with correct type for each column ending with "_at"
new_frame = frame.assign(
    **{
        f"{col}_timestamp": lambda x: pd.to_datetime(x[col])
        for col in frame.columns
        if col.endswith("_at")
    }
)
```

### Query 

`query` also takes a dataframe and returns a dataframe. It is used to filter down data; to keep rows that are relevant and discard those that aren’t. Some examples:

```python
# filter out rows that have a particular column that is NULL
new_frame = frame.query("not user_id.isnull()")

# keep rows that match one of many choices
new_frame = frame.query("user_id in ('123', '789')")

# use variables defined elsewhere, using the @ syntax
acceptable_ids = ["123", "789"]
new_frame = frame.query("user_id in @acceptable_ids")

# filter on a dynamically defined column
col = max(frame.columns)
new_frame = frame.query(f"not {col}.isnull()")

# filter on a column with a space in its name
# (note: creating these types of columns is not advisable)
new_frame = (
    frame
    .assign(
        **{"full name": lambda x: x["first_name"] + " " + x["last_name"]}
    )
    .query("`full name` == 'Varun box'")
)
```

### Groupby

`groupby.agg` and `groupby.transform` can be used to group a dataframe by one or more columns. 

`groupby.agg` aggregates the result and typically produces a dataframe with fewer rows, indexed by the grouped columns. It is useful to call `.reset_index` afterwards to destroy the index that was created. Also, always use `pd.NamedAgg` so that the aggregation columns that are created have meaningful names.

Conversely, `groupby.transform` aggregates the result but leaves the number of rows unchanged. It can then be assigned back to the original dataframe using `assign`.  Some examples should make this clearer:

```python
# get the counts of a column, assign it to "first_name_counts"
# like a COUNT(*) in SQL
agg_frame = (
    frame
    .groupby(by=["first_name"])
    .agg(first_name_counts=pd.NamedAgg("first_name", "size"))
    .reset_index()
)

# get the counts of a column, assign it to the original frame
# like a COUNT(*) OVER (PARTITION BY ...) in SQL
# the syntax here is quite clunky, unfortunately
agg_frame = (
    frame
    .assign(
        first_name_counts=lambda x: x.groupby(
            by=["first_name"]
        )["first_name"].transform(lambda x: x.size)
    )
)

# groupby several columns, generate several metrics from different columns
agg_frame = (
    frame
    .groupby(by=["first_name", "user_id"])
    .agg(
        first_name_counts=pd.NamedAgg("first_name", "size"),
        max_last_name=pd.NamedAgg("last_name", "max")
    )
    .reset_index()
)
```

### Merge

`merge` does the equivalent of a SQL join on two dataframes. There are some limitations compared to SQL (for instance, *in*equality joins are not allowed), but the core functionality is the same. It is useful to explicitly specify the join type using the `how` keyword arg (the default is “inner”). Some examples:

```python
# merge two dataframes together on a common column
joined_frame = frame.merge(frame_2, how="inner", on="user_id")

# merge two dataframes together on two different columns
joined_frame = frame.merge(
    frame_2, how="left", left_on="user_id", right_on="user_id_other",
)

# avoid name clashes between columns by explicitly specifying suffixes
joined_frame = frame.merge(
    frame_2, how="left", on="user_id", suffixes=("", "_2"),
)

# join on multiple columns
joined_frame = frame.merge(
    frame_2,
    how="inner",
    left_on=["user_id", "first_name"],
    right_on=["user_id_other", "first_name"],
    suffixes=("", "_2"),
)

# do a cross join (this requires some hackery to create a new join column)
joined_frame = (
    frame
    .assign(dummy_col=0)
    .merge(
        frame_2.assign(dummy_col=0),
        on="dummy_col",
        how="inner",
        suffixes=("", "_2"),
    )
    .drop(columns=["dummy_col"])
)
```

There are a number of other methods that operate at the level of the series. As stated above, they are used in conjunction with `assign`, typically, in order to create new columns and leave the granularity of the table unchanged. I have listed the ones I use regularly in the table above, but this is by no means a comprehensive list.

All-in-all, there are some ~20-30 methods to know, and ~4 to know very well, which is not too bad as far as a “simple” vocabulary goes!

## Best Practices and Idioms

* Build up your transformed data as a sequence of simple transformations. This will look like a long chain of `.` methods—or “method chaining”, if you want to sound fancy—applied to the dataframe, like:

```python
duplicate_names = (
    frame
    .assign(
        full_name=lambda x: x["first_name"] + " " + x["last_name"]
    )
    .groupby(by=["full_name"])
    .agg(name_count=pd.NamedAgg("full_name", "size"))
    .reset_index()
    .query("name_count > 1")
)
```

(If you’re curious, this code takes a dataframe of first and last names and generates a list of full names that appear twice. The same could be done with `.drop_duplicates`, the equivalent of SQL `DISTINCT`, of course.)

Setting the specifics aside, the key point is that we start with a dataframe and arrive at our final answer using a sequence of small “pure” transformations.

* Use a code formatter like `black` to make your code more readable. `black` also integrates with Jupyter notebooks via a Jupyter extension; see [here](https://github.com/drillan/jupyter-black).
* Never use the `inplace=True` version of operations; always use the operation to create a new, modified dataframe. In other words, prefer code like `frame = frame.reset_index()` to `frame.reset_index(inplace=True)`
* Defining your own functions to DRY repetitive transformations is recommended; make sure these take a dataframe (as the first argument) and return a dataframe, like the operations discussed above do. The `.pipe` operator can be used to retain “method chaining”. As an example:

```python
def my_value_counts(frame, group_cols):
    return (
        frame
        .groupby(by=group_cols)
        .agg(name_count=pd.NamedAgg(group_cols[0], "size"))
        .reset_index()
    )

frame_with_value_counts = (
    frame
    .assign(
        full_name=lambda x: x["first_name"] + " " + x["last_name"]
    )
    .pipe(my_value_counts, group_cols=["last_name"])
)
```

* Ignore the index of the dataframe, for the most part. The only exception is that `groupby.agg` and `pivot_table` (which does a `groupby` under the hood) unfortunately create a new index, which should be destroyed with `reset_index`. Clever use of the index sometimes makes data transformations more performant (e.g., fast lookups), but for 95% of work I find it unnecessary.
* Always pay attention to the granularity or “grain” of the dataframe. This is essentially the combination of columns that provides a unique id for each row. Most operations keep the grain intact. `groupby.agg` and `pivot_table`  are important exceptions: they allow you to “coarsen” the grain of the dataframe.

## A Worked Example

We run a lot of experiments at my company, and sometimes I want to analyze the results outside of our experimentation platform for the purpose of generating hypotheses (i.e., not to show something is statistically significant)

Suppose I have a dataframe of raw user metrics (`raw_metrics_frame`), like:

| user_id | week_number | metric |
| --- | --- | --- |
| 1 | 1 | 2 |
| 2 | 1 | 1.5 |
| 3 | 1 | 0.2 |
| 2 | 2 | 0.5 |
| 3 | 2 | 1 |

And a dataframe of experiment assignments (`assignments_frame`), like:

| user_id | treatment_name |
| --- | --- |
| 1 | control |
| 2 | control |
| 3 | variant |

I have a hypothesis that the variant treatment leads to better long-term results, with respect to the control, even if the short-term results, again with respect to the control, are worse. I want to see if this is the case by generating a dataframe like this:

| week_number | diff_between_variant_and_control_metric_per_user |
| --- | --- |
| 1 | -1.65 |
| 2 | 0.25 |

The first step is to join our two raw dataframes together, and fill in zero values for the metric where the join “missed” (i.e., the user was assigned but had no activity that week):

```python
joined_frame = (
    assignments_frame
    # cross join to get all user_id + week_num combinations
    # use "dummy_col" hack
    .assign(dummy_col=0)
    .merge(
        raw_metrics_frame[["week_num"]].drop_duplicates().assign(dummy_col=0),
        on="dummy_col"
    )
    .drop(columns=["dummy_col"])
    .merge(
        raw_metrics_frame, on=["user_id", "week_num"], how="left"
    )
    .assign(
        metric_filled=lambda x: np.select(
            condlist=[~x["metric"].isnull()],
            choicelist=[x["metric"]],
            default=0,
        )
    )
)
```

Then we can aggregate and produce a new column that defines the per-user metric for each treatment name + week combination

```python
aggregated_frame = (
    joined_frame
    .groupby(by=["treatment_name", "week_num"])
    .agg(
        user_count=pd.NamedAgg("treatment_name", "size"),
        metric_sum=pd.NamedAgg("metric_filled", "sum"),
    )
    .reset_index()
    .assign(per_user_metric=lambda x: x["metric_sum"]/x["user_count"])
)
```

Finally we can reshape the data to get the dataframe at the granularity of the week number, instead of the treatment name + week number combination, and then compute the “diff” metric we’re ultimately seeking.

```python
final_frame = (
    aggregated_frame
    .pivot_table(
        values="per_user_metric",
        index="week_num",
        columns="treatment_name",
    )
    .reset_index()
    .assign(
        diff_between_variant_and_control_metric_per_user=lambda x: x["variant"] - x["control"],
    )
)
```

Putting it all together, we have:

```python
joined_frame = (
    assignments_frame
    # cross join to get all user_id + week_num combinations
    # use "dummy_col" hack
    .assign(dummy_col=0)
    .merge(
        raw_metrics_frame[["week_num"]].drop_duplicates().assign(dummy_col=0),
        on="dummy_col"
    )
    .drop(columns=["dummy_col"])
    .groupby(
        by=["treatment_name", "week_num"]
    )
    .agg(
        user_count=pd.NamedAgg("treatment_name", "size"),
        metric_sum=pd.NamedAgg("metric_filled", "sum"),
    )
    .reset_index()
    .assign(per_user_metric=lambda x: x["metric_sum"]/x["user_count"])
    .pivot_table(
        values="per_user_metric",
        index="week_num",
        columns="treatment_name",
    )
    .reset_index()
    .assign(
        diff_between_variant_and_control_metric_per_user=lambda x: x["variant"] - x["control"],
    )
    # retain only desired columns
    .get(["week_num", "diff_between_variant_and_control_metric_per_user"])
)
```

A few comments:

- In practice, I would probably break this code up into a few discrete chunks, as I had done above, although I admit it looks fairly elegant when written as a single stream
- The workhorse functions (`.assign`, `.groupby`, `.merge`, `.query` do the vast majority of the work, although others (`np.select`, `.pivot_table` , etc.) also contribute.
- There are many other ways to get the same result, some of which are probably more concise, but I find the code I’ve written to be highly readable because of its use of this limited/”simple” vocabulary.
