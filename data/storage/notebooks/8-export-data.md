---
id: export-data
title: Export Data
sidebar_label: Export Data
---

Deephaven supports writing data out to various formats, such as CSV and Parquet. Deephaven can also export real-time data to new streaming sources, such as [Kafka streams](https://deephaven.io/core/docs/how-to-guides/data-import-export/kafka-stream/#write-to-a-kafka-stream).

If you ran the Docker command in the introduction, it created a directory called `data` locally, and linked it to a directory called `/data` in the running Docker container. Files saved in `/data` from Deephaven will be written to your local `data` directory. So, all of the examples below save data to the `/data` directory. For more details and customization options, see [this guide](https://deephaven.io/core/docs/conceptual/docker-data-volumes/).

In the case of pip-installed Deephaven, you can write to any directory on your local machine. Deephaven still creates a `data` directory to store some state files, but its location is OS-dependent. See [this guide](https://deephaven.io/core/docs/how-to-guides/configuration/native-application/#data-directory) to learn more about the `data` directory.

First, create a table to use for the remainder of this section:

```python test-set=12
from deephaven import empty_table

t = empty_table(10).update(
    [
        "Timestamp = '2015-01-01T00:00:00 ET' + 'PT1m'.multipliedBy(ii)",
        "Group = randomInt(1, 4)",
        "GroupMean = Group == 1 ? -10.0 : Group == 2 ? 0.0 : Group == 3 ? 10.0 : NULL_DOUBLE",
        "GroupStd = Group == 1 ? 2.5 : Group == 2 ? 0.5 : Group == 3 ? 1.0 : NULL_DOUBLE",
        "X = randomGaussian(GroupMean, GroupStd)",
    ]
)
```

## To CSV

Deephaven's [`write_csv`](https://deephaven.io/core/docs/reference/data-import-export/CSV/writeCsv/) function enables you to write Deephaven tables out to CSV files:

```python test-set=12
from deephaven import write_csv

write_csv(t, "/data/t.csv")
```

## To Parquet

Use the [`deephaven.parquet`](https://deephaven.io/core/pydoc/code/deephaven.parquet.html#module-deephaven.parquet) module to write data out to the Parquet file format:

```python test-set=12
from deephaven import parquet

parquet.write(t, "/data/t.parquet")
```

Deephaven supports advanced Parquet features, such as [partitioned Parquet files](https://deephaven.io/core/docs/how-to-guides/data-import-export/parquet-directory/#write-and-read-a-partitioned-parquet-file). See the guides on [Parquet formats](https://deephaven.io/core/docs/conceptual/parquet-formats/), [single Parquet files](https://deephaven.io/core/docs/how-to-guides/data-import-export/parquet-single/), and [multiple Parquet files](https://deephaven.io/core/docs/how-to-guides/data-import-export/parquet-directory/) for more information.

## Publish to Kafka stream

Deephaven can publish ticking data to Kafka streams. Setting up Kafka streams is outside the scope of this guide, but the [guide on Kafka streams](https://deephaven.io/core/docs/how-to-guides/data-import-export/kafka-stream/) will teach you everything you need to know.
