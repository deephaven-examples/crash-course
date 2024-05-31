---
id: create-tables
title: Create Your First Tables
sidebar_label: Create Tables
---

## Static tables

To get going, create some simple static tables using [`new_table`](https://deephaven.io/core/docs/reference/table-operations/create/newTable/) and [`empty_table`](https://deephaven.io/core/docs/how-to-guides/empty-table/):

```python
from deephaven import new_table, empty_table
from deephaven.column import int_col, long_col, double_col

static_table1 = new_table(
    [
        int_col("IntColumn", [0, 1, 2, 3, 4]),
        long_col("LongColumn", [0, 1, 2, 3, 4]),
        double_col("DoubleColumn", [0.12345, 2.12345, 4.12345, 6.12345, 8.12345]),
    ]
)

static_table2 = empty_table(5).update(
    [
        "IntColumn = i",
        "LongColumn = ii",
        "DoubleColumn = IntColumn + LongColumn + 0.12345",
    ]
)
```

These tables are identical but were created in two different ways: [`new_table`](https://deephaven.io/core/docs/reference/table-operations/create/newTable/) builds a Deephaven table from Python objects and helper functions for column type specification. [`empty_table`](https://deephaven.io/core/docs/reference/table-operations/create/emptyTable/) creates an empty table with a specified number of rows, and you can use DQL to add new columns programmatically.

## Ticking tables

Ticking tables are usually the endpoints for live data sources. In Deephaven, you can also synthesize ticking data to get a feel for live data, such as with [`time_table`](https://deephaven.io/core/docs/reference/table-operations/create/timeTable/):

```python
from deephaven import time_table

ticking_table = time_table("PT1s")
```

The `"PT1s"` argument is an [ISO-8641-formatted duration string](https://www.digi.com/resources/documentation/digidocs/90001488-13/reference/r_iso_8601_duration_format.htm) that indicates the table will tick once every second.

New ticking tables can be derived from older ones using DQL, just as in the static case:

```python
new_ticking_table = ticking_table.update("TimestampPlusOneSecond = Timestamp + 'PT1s'")
```

This exemplifies Deephaven's use of the DAG - `ticking_table` is a parent table / root node, and `new_ticking_table` is a downstream table. Because `ticking_table` is ticking, `new_ticking_table` is also ticking. Every update to `ticking_table` propagates down the DAG to `new_ticking_table`, and the compute-on-deltas model ensures that only the updated rows get re-evaluated with each update cycle.

## Ingesting static data

Deephaven supports reading from various common file formats like CSV, Parquet, and Arrow. You can easily import a CSV file from a URL if you don't have the data stored locally:

```python
from deephaven import read_csv

crypto = read_csv(
    "https://media.githubusercontent.com/media/deephaven/examples/main/CryptoCurrencyHistory/CSV/FakeCryptoTrades_20230209.csv"
)
```

If you're running Deephaven in a Docker container, reading your own files requires that you mount the local directory containing the files to a volume in the Docker container, which you can learn more about in the guide on [Docker data volumes](https://deephaven.io/core/docs/conceptual/docker-data-volumes/).

In addition to CSV, Deephaven provides a [Parquet library](https://deephaven.io/core/pydoc/code/deephaven.parquet.html#module-deephaven.parquet), an [Arrow library](https://deephaven.io/core/pydoc/code/deephaven.arrow.html#module-deephaven.arrow), and a [SQL library](https://deephaven.io/core/docs/reference/data-import-export/SQL/read_sql/) for reading data from these various formats.

## Real-world ticking data

Real-time data is Deephaven's mission statement. One of the easiest ways to work with realistic ticking data is by using the [`TableReplayer`](https://deephaven.io/core/docs/reference/table-operations/create/Replayer/) to replay the static data. Use it to replay the data ingested above:

```python
from deephaven.replay import TableReplayer

replayer = TableReplayer("2021-09-22T16:00:11.78 ET", "2021-09-22T20:11:34.60 ET")
replayed_crypto = replayer.add_table(
    crypto.sort("Timestamp"), "Timestamp"
).sort_descending("Timestamp")
replayer.start()
```

Most real-world use cases for ticking data involve connecting to data streams that are constantly being updated. For this, Deephaven's [Apache Kafka integration](https://deephaven.io/core/docs/how-to-guides/data-import-export/kafka-stream/) is first-in-class, and almost any imaginable real-time streaming source can be wrangled with the [`TablePublisher`](https://deephaven.io/core/docs/how-to-guides/table-publisher/) or [`DynamicTableWriter`](https://deephaven.io/core/docs/reference/table-operations/create/DynamicTableWriter/). That said, setting up the pipelines for Kafka streams or other real-time data sources can be very complex, and is outside the scope of this guide.
