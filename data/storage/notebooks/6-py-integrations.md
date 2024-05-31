---
id: py-integrations
title: Powerful Python Integrations
sidebar_label: Python Integrations
---

Deephaven empowers Python developers by providing efficient integrations with popular Python libraries. This section covers some highlights of Deephaven's Python interoperability as well as the inherent limitations of static Python data structures.

## Pandas

The [`deephaven.pandas`](https://deephaven.io/core/docs/how-to-guides/use-pandas/) module is the gateway to [Pandas](https://pandas.pydata.org) interoperability. The module is simple, containing only two key functions: [`to_pandas`](https://deephaven.io/core/pydoc/code/deephaven.pandas.html#deephaven.pandas.to_pandas), which converts a Deephaven table to a [Pandas DataFrame](https://pandas.pydata.org/docs/reference/frame.html), and [`to_table`](https://deephaven.io/core/pydoc/code/deephaven.pandas.html#deephaven.pandas.to_table), which converts a Pandas DataFrame to a Deephaven table:

```python
from deephaven.pandas import to_pandas, to_table
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

t_df = to_pandas(t)
```

Note that the appropriate column types are automatically inferred:

```python
print(t_df.dtypes)
```

Pandas operations can be applied to the DataFrame and the result can be converted back to a Deephaven table when you're ready to use Deephaven again:

```python
t_df_group_avg = t_df.groupby("Group").mean()
t_group_avg = to_table(t_df_group_avg)
```

Using DQL to do this particular job would be more efficient, but the integration is still useful in some workflows.

Note that Pandas DataFrames are inherently static. Converting a _ticking_ Deephaven table to a Pandas DataFrame _snapshots_ the table.

## Numpy

[Numpy](https://numpy.org) is a popular package in the Python ecosystem that implements [arrays](<https://en.wikipedia.org/wiki/Array_(data_structure)>) and array operations. Deephaven provides the [`deephaven.numpy`](https://deephaven.io/core/docs/how-to-guides/use-numpy/) module for Numpy interoperability.

It's easy to convert between Numpy arrays and Deephaven tables with [`to_numpy`](https://deephaven.io/core/pydoc/code/deephaven.numpy.html#deephaven.numpy.to_numpy) and [`to_table`](https://deephaven.io/core/pydoc/code/deephaven.numpy.html#deephaven.numpy.to_numpy):

```python
from deephaven.numpy import to_numpy, to_table
from deephaven import empty_table

t = empty_table(10).update(["X = ii", "Y = X + 2", "Z = sin(X + Y)"])

t_np = to_numpy(t.view("X = (double)X"))
print(t_np)

t_np_transposed = t_np.transpose()
print(t_np_transposed)

col_names = [
    "Col1",
    "Col2",
    "Col3",
    "Col4",
    "Col5",
    "Col6",
    "Col7",
    "Col8",
    "Col9",
    "Col10",
]
t_transposed = to_table(t_np_transposed, col_names)
```

Note that we cast column `X` from a `long` to a `double` in this example. This is because Numpy arrays only have a _single_ data type, while Deephaven tables have one type per column. So, if you need to convert a Deephaven table to a Numpy array, you must ensure all columns have the same type. Otherwise, the conversion will fail:

```python
t = empty_table(10).update(["X = ii", "Y = sin(X)", "Z = Y > 0 ? 1 : 0"])

t_np = to_numpy(t)
```

## The `deephaven.learn` library

Aimed at ML/data science practitioners, [`deephaven.learn`](https://deephaven.io/core/docs/how-to-guides/use-deephaven-learn/) provides a general-purpose framework for efficient integration with [PyTorch](https://pytorch.org), [Tensorflow](https://www.tensorflow.org), [Scikit-learn](https://scikit-learn.org/stable/), and more. The architecture of [`deephaven.learn`](https://deephaven.io/core/docs/how-to-guides/use-deephaven-learn/) is fundamentally geared towards ticking tables, enabling real-time machine learning applications with unprecedented ease.

This module utilizes the [gather-compute-scatter paradigm](https://deephaven.io/core/docs/how-to-guides/use-deephaven-learn/#the-gather-compute-scatter-paradigm) to ensure optimal performance. Data is [gathered](https://deephaven.io/core/docs/how-to-guides/use-deephaven-learn/#gather) from Deephaven tables, outputs are [computed](https://deephaven.io/core/docs/how-to-guides/use-deephaven-learn/#compute) by user-defined functions, and the results are [scattered](https://deephaven.io/core/docs/how-to-guides/use-deephaven-learn/#scatter) back to the Deephaven table.

This simple example uses [`deephaven.learn`](https://deephaven.io/core/docs/how-to-guides/use-deephaven-learn/) to compute the likelihood of some toy data under three different distributions:

```python
import os

os.system("pip install scipy")

from deephaven import empty_table
from deephaven.learn import gather
from deephaven import learn

import scipy.stats as stats
import numpy as np

t = empty_table(10).update(
    [
        "Timestamp = '2015-01-01T00:00:00 ET' + 'PT1m'.multipliedBy(ii)",
        "Group = randomInt(1, 4)",
        "GroupMean = Group == 1 ? -10.0 : Group == 2 ? 0.0 : Group == 3 ? 10.0 : NULL_DOUBLE",
        "GroupStd = Group == 1 ? 2.5 : Group == 2 ? 0.5 : Group == 3 ? 1.0 : NULL_DOUBLE",
        "X = randomGaussian(GroupMean, GroupStd)",
    ]
)


# "compute" function
def likelihood(data):
    return np.array(
        [
            stats.norm.pdf(data, loc=-10.0, scale=2.5),
            stats.norm.pdf(data, loc=0.0, scale=0.5),
            stats.norm.pdf(data, loc=10.0, scale=1.0),
        ]
    )


# "gather" function
def table_to_np(rows, columns):
    return gather.table_to_numpy_2d(rows, columns, np_type=np.double)


# "scatter" function
def np_to_table(data, idx, group):
    return data[group][idx]


# The learn function call
result = learn.learn(
    table=t,
    model_func=likelihood,
    inputs=[learn.Input(["X"], table_to_np)],
    outputs=[
        learn.Output(
            "Group1Likelihood",
            lambda data, idx: np_to_table(data, idx, group=0),
            "double",
        ),
        learn.Output(
            "Group2Likelihood",
            lambda data, idx: np_to_table(data, idx, group=1),
            "double",
        ),
        learn.Output(
            "Group3Likelihood",
            lambda data, idx: np_to_table(data, idx, group=2),
            "double",
        ),
    ],
    batch_size=10,
)
```

There are details here that are outside of the scope of this document. To learn more, check out the [user guide](https://deephaven.io/core/docs/how-to-guides/use-deephaven-learn/). Then, head over to [this blog post](https://deephaven.io/blog/2022/02/02/learn-scikit/) to see [`deephaven.learn`](https://deephaven.io/core/docs/how-to-guides/use-deephaven-learn/) in action on a real-time machine learning task.

## Function-generated tables

[Function-generated tables](https://deephaven.io/core/docs/how-to-guides/function-generated-tables/) allow you to execute Python functions at regular intervals, and populate a Deephaven table with the results. This example re-generates data for a table every second:

```python
from deephaven import empty_table, function_generated_table


def regenerate():
    return empty_table(10).update(
        [
            "Group = randomInt(1, 4)",
            "GroupMean = Group == 1 ? -10.0 : Group == 2 ? 0.0 : Group == 3 ? 10.0 : NULL_DOUBLE",
            "GroupStd = Group == 1 ? 2.5 : Group == 2 ? 0.5 : Group == 3 ? 1.0 : NULL_DOUBLE",
            "X = randomGaussian(GroupMean, GroupStd)",
        ]
    )


fgt = function_generated_table(table_generator=regenerate, refresh_interval_ms=1000)
```

This is especially useful when using plain Python to pull real-time data sources, like [websockets](https://en.wikipedia.org/wiki/WebSocket). [Here's a blog post](https://deephaven.io/blog/2023/10/06/function-generated-tables/) on connecting to a websocket using a popular Python framework, and using a function-generated table as a single endpoint for all of the live data.
