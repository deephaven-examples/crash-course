---
id: architecture-overview
title: An Overview of Deephaven's Architecture
sidebar_label: Architecture Overview
---

Deephaven's power is largely due to the concept that _everything_ is a table. Think of Deephaven tables like you think of [Pandas DataFrames](https://pandas.pydata.org/pandas-docs/stable/user_guide/dsintro.html#dataframe), except they support real-time operations! The Deephaven table is the key abstraction that unites static and real-time data for a seamless, integrated experience. This section will discuss the conceptual building blocks of Deephaven tables before diving into some real code.

## The Deephaven table API

If you're familiar with tabular data (think Excel, Pandas, and many more), Deephaven will feel natural to you. In addition to representing data as tables, you must be able to manipulate, transform, extract, and analyze the data to produce valuable insight. Deephaven represents transformations to the data stored in a table as _operations_ on that table. These table operations can be chained together to produce new perspectives on the original table's data:

```python
from deephaven import new_table
from deephaven.column import string_col, int_col

source_table = new_table(
    [string_col("Group", ["A", "B", "A", "B"]), int_col("Data", [1, 2, 3, 4])]
)

new_table = (
    source_table.update("NewData = Data * 2")
    .sum_by("Group")
    .rename_columns(["DataSum=Data", "NewDataSum=NewData"])
)
```

This paradigm of representing data transformations with methods that act on tables is not new. However, Deephaven takes this a step further. In addition to its large collection of table operations, Deephaven provides a vast library of efficient functions and operations, accessible through _query strings_, that get invoked from _within_ table operations. These query strings, once understood, are a superpower that is totally unique to Deephaven. More on these soon.

Collectively, the representation of data as tables, the table operations that act upon them, and the query strings used within table operations are known as the _Deephaven Query Language_, or DQL for short. The data transformations written with DQL (like the code above) are called _Deephaven queries_. The fundamental principle of the Deephaven experience is this:

<Pullquote>
Deephaven queries are unaware and indifferent to whether the underlying data source is static or streaming.
</Pullquote>

The magic here is hard to overstate. The same queries written to parse a simple CSV file with a particular schema can be used to analyze a similarly fashioned living dataset, evolving at tens of thousands of rows per second with absolutely zero change to the underlying code. Real-time data analysis does not get simpler.

## Immutable schema, mutable data

When Deephaven tables are created, they follow a well-defined recipe that _cannot_ be modified after creation. This means that the number of columns in a table, the column names, and the data types of those columns - collectively known as the table's _schema_ - are immutable. Even so, the data stored in a table can change. Here's a simple example using [`time_table`](https://deephaven.io/core/docs/reference/table-operations/create/timeTable/):

```python
from deephaven import time_table

# Create a table that ticks once per second - defaults to a single "Timestamp" column
ticking_table = time_table("PT1s")
```

New data is added to this table each second, but its schema is untouched.

Some queries appear to modify a table's schema - they may add or remove a column, modify a column's type, or rename a column. For example, the following query appears to rename the `Timestamp` column in the table above to `RenamedTimestamp`:

```python
# Apply 'rename_columns' table operation to ticking_table
ticking_table = ticking_table.rename_columns("RenamedTimestamp=Timestamp")
```

Since the schema of `ticking_table` is immutable, its columns can never be renamed. Instead, Deephaven efficiently creates a _brand new_ table with a new schema and stores it in the variable `ticking_table`.

An important consequence of this design choice is that Deephaven does not support [in-place operations](https://en.wikipedia.org/wiki/In-place_algorithm#:~:text=In%20computer%20science%2C%20an%20in,copy%20of%20the%20data%20structure.). Therefore, you will never see a proper Deephaven query written like this:

```python
from deephaven import empty_table

# Create an empty table
my_table = empty_table(5)

# Attempt to add a column to my_table
my_table.update("NewCol = `New column here!`")  # Don't do this
```

Since Deephaven queries produce new tables with new schemas, the results should always be captured by a variable:

```python
from deephaven import empty_table

# Create an empty table
my_table = empty_table(5)

# Create a new table from a recipe that contains my_table and a new column
my_new_table = my_table.update("NewCol = `New column here!`")
```

It's perfectly valid to use the same variable for an updated table, giving the impression that the existing table really has been updated:

```python
from deephaven import empty_table

# Create an empty table
my_table = empty_table(5)

# Create a new table with the same name
my_table = my_table.update("NewCol = `New column here!`")
```

## The engine

The Deephaven engine is the powerhouse that implements everything we've discussed so far. The engine is responsible for processing streaming data as efficiently as possible, and all Deephaven queries must flow through the engine's guts.

For the sake of efficiency and scalability, the engine is implemented in [Java](<https://en.wikipedia.org/wiki/Java_(programming_language)>). Although you primarily interface with tables and write queries from Python, _all_ the underlying data in a table is stored in Java data structures, and all low-level table operations are implemented in Java. Most Deephaven users will interact with Java at least a little bit, and there are [important things to consider](https://deephaven.io/core/docs/conceptual/python-java-boundary/) when passing data from Python to Java or vice-versa.

The engine represents relationships between tables as [directed-acyclic graphs](https://deephaven.io/core/docs/conceptual/dag/), or DAGs. This representation is key to Deephaven's efficiency, as updates to parent tables are propagated to all relevant downstream tables, and downstream tables are always ready for updates from their parents. Additionally, when the parent receives updates, _only the [updated information](https://deephaven.io/blog/2022/03/23/deephaven-materialize-deltas/)_ gets propagated to the children, and no unnecessary computation is wasted on parts of the table that have not changed.
