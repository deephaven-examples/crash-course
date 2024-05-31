---
id: table-ops
title: Basic Table Operations
sidebar_label: Table Operations
---

Table operations are an integral component of DQL. You've already seen a couple: [`update`](https://deephaven.io/core/docs/reference/table-operations/select/update/) and [`where`](https://deephaven.io/core/docs/reference/table-operations/filter/where/). You can think of these as different transformations being applied to the data in the table. This section will outline some basic table operations that make up the backbones of the most common queries.

:::note
This section makes use of query strings, covered in the [next section](./5-query-strings.md).
:::

## Basic column manipulation

Previous sections used the [`update`](https://deephaven.io/core/docs/reference/table-operations/select/update/) table operation to create new columns in a table. In particular, [`update`](https://deephaven.io/core/docs/reference/table-operations/select/update/) creates [_in-memory_](https://deephaven.io/core/docs/how-to-guides/use-select-view-update/#choose-the-right-column-selection-method) columns, meaning that the values in the new columns get computed immediately and stored in your computer's memory. Use [`update`](https://deephaven.io/core/docs/reference/table-operations/select/update/) with four query strings to create four new in-memory columns:

```python
from deephaven import empty_table

t = empty_table(60 * 24).update(
    [
        "Timestamp = '2015-01-01T00:00:00 ET' + 'PT1m'.multipliedBy(ii)",
        "X = ii",
        "Y = X + 5 * sin(X) + randomGaussian(0.0, 1.0)",
        "Group = randomInt(1, 10)",
    ]
)
```

In this case, the four query strings passed to [`update`](https://deephaven.io/core/docs/reference/table-operations/select/update/) are enclosed in `[]` - this is required to pass multiple query strings to table operations.

When datasets get really big, or the computations for new columns are fast, you can elect to work with _formula_ columns. In contrast to in-memory columns, formula columns store only the formula and compute their values on the fly, as needed. This can reduce overall workload when only a small portion of the data needs to be accessed. Use the [`update_view`](https://deephaven.io/core/docs/reference/table-operations/select/update-view/) table operation to add a formula column to `t`:

```python test-set=5
t_updated = t.update_view("Z = Y + 10")
```

Here, only one query string is passed to the [`update_view`](https://deephaven.io/core/docs/reference/table-operations/select/update-view/) table operation, so the enclosing `[]` are left out. For singular query strings, they are optional.

Deephaven also provides a [`lazy_update`](https://deephaven.io/core/docs/reference/table-operations/select/lazy-update/) table operation for creating columns. [`lazy_update`](https://deephaven.io/core/docs/reference/table-operations/select/lazy-update/) only does calculations for _unique_ values in the input columns and stores the results. So, if the input columns have a small number of unique values relative to the size of the table, it will perform far fewer calculations than [`update`](https://deephaven.io/core/docs/reference/table-operations/select/update/) or [`update_view`](https://deephaven.io/core/docs/reference/table-operations/select/update-view/):

```python test-set=5
t_lazy_updated = t.lazy_update("GroupSqrt = sqrt(Group)")
```

The [`select`](https://deephaven.io/core/docs/reference/table-operations/select/) and [`view`](https://deephaven.io/core/docs/reference/table-operations/select/view/) operations are used to select columns from existing tables and produce new in-memory and formula tables, respectively:

```python
t_subset_in_memory = t_updated.select(["Timestamp", "Z"])
t_subset_formula = t_updated.view(["Timestamp", "Y"])
```

These operations can also accept formulas, making it easy to create and extract new columns with one operation:

```python test-set=5 order=new_t_subset_in_memory,new_t_subset_formula
new_t_subset_in_memory = t_updated.select(
    ["NewTimestamp = Timestamp + 'PT5s'", "Sum = X + Y"]
)
new_t_subset_formula = t_updated.view(
    ["NewTimestamp = Timestamp + 'PT5s'", "Product = X * Y"]
)
```

Next, use [`drop_columns`](https://deephaven.io/core/docs/reference/table-operations/select/drop-columns/) to drop columns from existing tables:

```python
t_dropped = t_updated.drop_columns(["X", "Y"])
```

There's a lot to consider when choosing between in-memory, formula, and memoized columns in queries. For more information, see [choose the right selection method](https://deephaven.io/core/docs/how-to-guides/use-select-view-update/#choose-the-right-column-selection-method).

## Sort data

Deephaven implements table sorting with [`sort`](https://deephaven.io/core/docs/reference/table-operations/sort/), which orders the rows in a table by the values in a column from smallest to largest, and [`sort_descending`](https://deephaven.io/core/docs/reference/table-operations/sort/sort-descending/), which does the same from largest to smallest:

```python
t_sort_y = t.sort("Y")
t_sort_time = t.sort_descending("Timestamp")
```

You can sort by multiple columns at once:

```python
t_sort_group = t.sort(["Group", "Timestamp"])
```

Multiple sort directions can be used in a single function call:

```python
from deephaven import SortDirection

asc = SortDirection.ASCENDING
desc = SortDirection.DESCENDING

t_sort_multi = t.sort(["Group", "Timestamp"], order=[desc, asc])
```

You can learn more about sorting in the [guide on sorting table data](https://deephaven.io/core/docs/how-to-guides/sort/).

## Filter by rows

Filtering data by specific values in a column is a common use case. Deephaven provides several filtering operations, the first of which is [`where`](https://deephaven.io/core/docs/reference/table-operations/filter/where/):

```python
t_filtered_1 = t.where(["Group % 2 == 0", "hourOfDay(Timestamp, 'ET', false) >= 12"])
```

Using multiple query strings in a [`where`](https://deephaven.io/core/docs/reference/table-operations/filter/where/) operation like above ensures that only rows that satisfy _all_ conditions are included in the result. To filter for rows that meet _at least one_ condition described by the query strings, use [`where_one_of`](https://deephaven.io/core/docs/reference/table-operations/filter/where-one-of/):

```python
t_filtered_2 = t.where_one_of(
    ["Group % 2 == 0", "hourOfDay(Timestamp, 'ET', false) >= 12"]
)
```

Deephaven also provides the [`where_in`](https://deephaven.io/core/docs/reference/table-operations/filter/where-in/) and [`where_not_in`](https://deephaven.io/core/docs/reference/table-operations/filter/where-not-in/) methods that filter a table based on values in another table:

```python
t_new = empty_table(60 * 24).update(
    [
        "Timestamp = '2015-01-01T12:00:00 ET' + 'PT1m'.multipliedBy(ii)",
        "X = ii",
        "Y = X + 5 * sin(X) + randomGaussian(0.0, 1.0)",
        "OtherGroup = randomInt(5, 15)",
    ]
)

t_filtered_3 = t.where_in(t_new, "Timestamp")
t_filtered_4 = t.where_not_in(t_new, "Group=OtherGroup")
```

There are many more filtering operations like [`head`](https://deephaven.io/core/docs/reference/table-operations/filter/head/), [`tail`](https://deephaven.io/core/docs/reference/table-operations/filter/tail/), and [`slice_pct`](https://deephaven.io/core/docs/reference/table-operations/filter/slice-pct/). You can read about these operations and more in the [filter guide](https://deephaven.io/core/docs/how-to-guides/use-filters/).

## Grouping and Aggregating

Deephaven provides the [`group_by`](https://deephaven.io/core/docs/reference/table-operations/group-and-aggregate/groupBy/) and [`ungroup`](https://deephaven.io/core/docs/reference/table-operations/group-and-aggregate/ungroup/) operations to group data into arrays and ungroup data from arrays:

```python
t_grouped = t.group_by("Group")
t_ungrouped = t_grouped.ungroup()
```

Aggregation is key in data analysis. Deephaven's [dedicated aggregations](https://deephaven.io/core/docs/how-to-guides/dedicated-aggregations/) are table operations that apply aggregations directly to tables. These are best suited to performing a single aggregation on a given group. Use the dedicated aggregator [`avg_by`](https://deephaven.io/core/docs/reference/table-operations/group-and-aggregate/avgBy/) to get the average of `X` and `Y` from the table `t`.

```python
t_avg = t.view(["X", "Y"]).avg_by()
```

You have the option to perform _grouped_ aggregations. Use these to compute the average of `X` and `Y` for each unique value in the `Group` column:

```python
t_group_avg = t.view(["X", "Y", "Group"]).avg_by("Group").sort("Group")
```

You can even aggregate on groups defined by sets of columns:

```python
t_multi_group_avg = (
    t.update_view("EvenMinuteGroup = X % 2 == 0 ? `A` : `B`")
    .view(["X", "Y", "Group", "EvenMinuteGroup"])
    .avg_by(["Group", "EvenMinuteGroup"])
    .sort(["Group", "EvenMinuteGroup"])
)
```

A comprehensive list of available aggregations is available [here](https://deephaven.io/core/docs/how-to-guides/dedicated-aggregations/#single-aggregators).

## Multiple aggregations

When you want to apply more than one aggregation to a set of columns efficiently, look to combined aggregations. Combined aggregations in Deephaven are facilitated by the [`agg_by`](https://deephaven.io/core/docs/reference/table-operations/group-and-aggregate/aggBy/) table operation and the [`deephaven.agg`](https://deephaven.io/core/pydoc/code/deephaven.agg.html#module-deephaven.agg) Python module.

You can use [`agg_by`](https://deephaven.io/core/docs/reference/table-operations/group-and-aggregate/aggBy/) and [`agg.avg`](https://deephaven.io/core/docs/reference/table-operations/group-and-aggregate/AggAvg/) to accomplish the same tasks as the dedicated aggregations above:

```python
from deephaven import agg

t_avg_multi = t.agg_by(agg.avg(["X", "Y"]))

t_group_avg_multi = t.agg_by(agg.avg(["X", "Y"]), by="Group").sort("Group")

t_multi_group_avg_multi = (
    t.update_view("EvenMinuteGroup = X % 2 == 0 ? `A` : `B`")
    .agg_by(agg.avg(["X", "Y"]), by=["Group", "EvenMinuteGroup"])
    .sort(["Group", "EvenMinuteGroup"])
)
```

The [`agg_by`](https://deephaven.io/core/docs/reference/table-operations/group-and-aggregate/aggBy/) operator can accept a list of aggregations from [`deephaven.agg`](https://deephaven.io/core/pydoc/code/deephaven.agg.html#module-deephaven.agg), enabling efficient multiple aggregations. Use query strings like `"AvgX = X"` to rename the columns that result from the aggregation:

```python
import deephaven.agg as agg

t_avg_std = t.agg_by([agg.avg(["AvgX=X", "AvgY=Y"]), agg.std(["StdX=X", "StdY=Y"])])

t_group_avg_std = t.agg_by(
    [agg.avg(["AvgX=X", "AvgY=Y"]), agg.std(["StdX=X", "StdY=Y"])], by="Group"
).sort("Group")

t_multi_group_avg_std = (
    t.update_view("EvenMinuteGroup = X % 2 == 0 ? `A` : `B`")
    .agg_by(
        [agg.avg(["AvgX=X", "AvgY=Y"]), agg.std(["StdX=X", "StdY=Y"])],
        by=["Group", "EvenMinuteGroup"],
    )
    .sort(["Group", "EvenMinuteGroup"])
)
```

For more guidance on multiple aggregations, check out the [user guide on combined aggregations](https://deephaven.io/core/docs/how-to-guides/combined-aggregations/).

## Cumulative, moving, and windowed aggregations

Most platforms offer aggregation functionality similar to the dedicated and multiple aggregations presented above (though none will work so easily on real-time data). However, Deephaven is unique and powerful in its vast library of cumulative, moving, and windowed calculations, facilitated by the [`update_by`](https://deephaven.io/core/docs/reference/table-operations/update-by-operations/updateBy/) table operation and the [`deephaven.updateby`](https://deephaven.io/core/pydoc/code/deephaven.updateby.html) Python module.

Cumulative, moving, and windowed calculations, like [time-based exponential moving averages](https://deephaven.io/core/docs/reference/table-operations/update-by-operations/ema-time/), are mission-critical to Deephaven's real-time enterprise customers. Use [`updateby.ema_time`](https://deephaven.io/core/docs/reference/table-operations/update-by-operations/ema-time/) and [`updateby.emstd_time`](https://deephaven.io/core/docs/reference/table-operations/update-by-operations/emstd-time/) to compute the time-based exponential moving average and standard deviation for multiple columns at once:

```python
import deephaven.updateby as uby

t_ema_emstd = t.update_by(
    [
        uby.ema_time(ts_col="Timestamp", decay_time="PT30m", cols=["EmaX=X", "EmaY=Y"]),
        uby.emstd_time(
            ts_col="Timestamp", decay_time="PT30m", cols=["EmstdX=X", "EmstdY=Y"]
        ),
    ]
)
```

Like [`agg_by`](https://deephaven.io/core/docs/reference/table-operations/group-and-aggregate/aggBy/), the `by` argument in [`update_by`](https://deephaven.io/core/docs/reference/table-operations/update-by-operations/updateBy/) can be used to define grouping columns. Compute the cumulative minimum and maximum of `X` and `Y` in each `EvenMinuteGroup`:

```python
t_cum_avg_group = t.update("EvenMinuteGroup = X % 2 == 0 ? `A` : `B`").update_by(
    [uby.cum_min(["MinX=X", "MinY=Y"]), uby.cum_max(["MaxX=X", "MaxY=Y"])],
    by="EvenMinuteGroup",
)
```

The [`deephaven.updateby`](https://deephaven.io/core/pydoc/code/deephaven.updateby.html) module includes other kinds of useful functionality, like the [`delta`](https://deephaven.io/core/docs/reference/table-operations/update-by-operations/delta/) function to compute pairwise consecutive differences:

```python
t_detrend = t.update_by(uby.delta("DetrendY=Y"))
```

The [`foward_fill`](https://deephaven.io/core/docs/reference/table-operations/update-by-operations/forward-fill/) function is useful for filling in missing values:

```python
t_missing = t.update(
    ["X = Group > 5 ? null : X", "Y = randomInt(0, 3) == 0 ? null : Y"]
)

t_filled = t_missing.update_by(uby.forward_fill("X")).update_by(
    uby.forward_fill("Y"), by="Group"
)
```

This only scratches the surface of [`update_by`](https://deephaven.io/core/docs/reference/table-operations/update-by-operations/updateBy/). The utility of these kinds of computations for real-time data is hard to overstate. To learn more, check out [this blog post](https://deephaven.io/blog/2023/06/22/update-by/#what-is-update_by) or the [`update_by` user guide](https://deephaven.io/core/docs/how-to-guides/use-update-by/).

## Merge tables

Combining disparate data sources can often yield new insight. In Deephaven, there are several ways to combine datasets.

First, create some tables that can be easily combined with `t`:

```python
from deephaven import empty_table

t = empty_table(60 * 24).update(
    [
        "Timestamp = '2015-01-01T00:00:00 ET' + 'PT1m'.multipliedBy(ii)",
        "X = ii",
        "Y = X + 5 * sin(X) + randomGaussian(0.0, 1.0)",
        "Group = randomInt(1, 100)",
    ]
)

t2 = empty_table(60 * 24).update(
    [
        "Timestamp = '2015-01-02T00:00:00 ET' + 'PT1m'.multipliedBy(ii)",
        "X = ii + 60*24",
        "Y = X + 5 * sin(X) + randomGaussian(0.0, 1.0)",
        "Group = randomInt(1, 100)",
    ]
)

t3 = empty_table(12 * 24 * 2).update(
    [
        "Timestamp = '2015-01-01T00:00:00 ET' + 'PT5m'.multipliedBy(ii)",
        "X = 3 * ii",
        "Y = X + 5 * cos(X) + randomGaussian(0.0, 1.0)",
    ]
)

t4 = empty_table(2 * 60 * 24 * 2).update(
    [
        "Timestamp = '2015-01-01T00:00:00 ET' + 'PT30s'.multipliedBy(ii)",
        "X = 0.5 * ii",
        "Y = X + 5 * tan(X) + randomGaussian(0.0, 1.0)",
    ]
)
```

The [`merge`](https://deephaven.io/core/docs/reference/table-operations/merge/) operation can be used to [stack tables](https://deephaven.io/core/docs/how-to-guides/merge-tables/) that have the same schema:

```python
from deephaven import merge

t_merged = merge([t, t2])
```

## Join data

Deephaven provides a host of different join operations, which you can learn about [here](https://deephaven.io/core/docs/how-to-guides/joins-exact-relational/#which-method-should-you-use). Most join operations perform the same basic task: they combine two tables, a _left_ and a _right_ table, based on one or more columns that they both have in common, called _key columns_. Typically, data is extracted from the right table and combined into the left table, and the results will change if the order is reversed. Join operations differ in how they combine tables when key columns contain zero, one, or multiple matches.

The [`join`](https://deephaven.io/core/docs/reference/table-operations/join/) will _only_ include rows where the key columns match exactly:

```python
joined_1 = t_merged.join(t3, on="Timestamp", joins="Y2=Y")
```

Alternatively, the [`natural_join`](https://deephaven.io/core/docs/reference/table-operations/join/natural-join/) will include _all_ rows from the left table, and leave the joined elements blank where there are no matches in the right table:

```python
joined_2 = t_merged.natural_join(t3, on="Timestamp", joins="Y2=Y")
```

:::note
The [`join`](https://deephaven.io/core/docs/reference/table-operations/join/) method computes the cross product of the input tables, and is generally much less performant than [`natural_join`](https://deephaven.io/core/docs/reference/table-operations/join/natural-join/). [`natural_join`](https://deephaven.io/core/docs/reference/table-operations/join/natural-join/) should be preferred where possible.
:::

If you require that every value has one _and only one_ exact match, use [`exact_join`](https://deephaven.io/core/docs/reference/table-operations/join/exact-join/):

```python
# This will produce an error, since some rows in t_merged (left) have no matches in t3 (right)
joined_3 = t_merged.exact_join(t3, on="Timestamp", joins="Y2=Y")
```

## Join time series data

The previous examples assumed that the right table has some _exact matches_ in the left table. This is not always the case, _especially_ in the world of real-time data, where two timestamps are very rarely exactly the same. Create a table with timestamps that are _close to_ the timestamps in `t_merged`:

```python
t5 = empty_table(12 * 24 * 2).update(
    [
        "Timestamp = '2015-01-01T00:00:00 ET' + 'PT5m'.multipliedBy(ii) + 'PT0.1s'.multipliedBy(randomInt(0, 10*60*5-1))",
        "X = 3 * ii",
        "Y = X + 5 * cos(X) + randomGaussian(0.0, 1.0)",
    ]
)
```

In this case, _all_ of the previous join operations will fail, because the `Timestamp` column in `t5` has no exact matches in `t_merged`. For this, Deephaven offers the as-of ([`aj`](https://deephaven.io/core/docs/reference/table-operations/join/aj/)) and reverse-as-of ([`raj`](https://deephaven.io/core/docs/reference/table-operations/join/raj/)) join operations. The as-of join is used to join data from the right table immediately before or at the time of an event in the left table, while the reverse-as-of join is used to join data from the right table immediately after or at the time of an event in the left table. Here they are in action:

```python
joined_4 = t_merged.aj(t5, on="Timestamp", joins="Y2=Y")
joined_5 = t_merged.raj(t5, on="Timestamp", joins="Y2=Y")
```

As you can see, the fact that we had no exact matches did not deter [`aj`](https://deephaven.io/core/docs/reference/table-operations/join/aj/) or [`raj`](https://deephaven.io/core/docs/reference/table-operations/join/raj/). These methods are so useful for timestamped data that you'll wonder how you ever did without them.

## Time and calendars

As a real-time data platform, Deephaven provides extensive support for time-related data. To this end, the guides on [working with time](https://deephaven.io/core/docs/conceptual/time-in-deephaven/) and [working with calendars](https://deephaven.io/core/docs/how-to-guides/business-calendar/) are great places to start with learning about these toolkits.
