---
id: plots
title: Real-time Plots
sidebar_label: Real-time Plots
---

Whether your data is static or updating in real time, Deephaven empowers you to visualize it seamlessly. Deephaven provides a native [plotting suite](https://deephaven.io/core/pydoc/code/deephaven.plot.html#module-deephaven.plot) for all your plotting needs.

## Creating basic plots

:::note
Plotting with pip-installed Deephaven from Jupyter requires some additional setup you can read about [here](https://deephaven.io/core/docs/how-to-guides/jupyter/).
:::

Deephaven's native plotting library provides a wide range of the most common plot types. Create a simple [line plot](https://en.wikipedia.org/wiki/Line_chart) that shows how data evolves over time:

```python
from deephaven import time_table
from deephaven.plot.figure import Figure

t = time_table("PT0.2s").update(
    [
        "X = ii",
        "Y1 = X + 5 * sin(X) + randomGaussian(0.0, 1.0)",
        "Y2 = 3 * X + 2 * sin(X / 2) + randomGaussian(0.0, 1.0)",
        "Y3 = X / 2 + 10 * sin(X * 3) + randomGaussian(0.0, 1.0)",
    ]
)

plot = Figure().plot_xy(series_name="Y1 trend", t=t, x="Timestamp", y="Y1").show()
```

You can plot multiple series together:

```python
import deephaven.updateby as uby

t = t.update_by(
    uby.rolling_avg_tick(
        ["RollAvgY1 = Y1", "RollAvgY2 = Y2", "RollAvgY3 = Y3"],
        rev_ticks=10,
        fwd_ticks=10,
    )
)

plot = (
    Figure()
    .plot_xy(series_name="Y1 trend", t=t, x="Timestamp", y="Y1")
    .plot_xy(series_name="Y1 rolling average", t=t, x="Timestamp", y="RollAvgY1")
    .show()
)
```

## Plotting with multiple axes

Deephaven's plots have two methods, [`x_twin`](https://deephaven.io/core/pydoc/code/deephaven.plot.figure.html#deephaven.plot.figure.Figure.x_twin) and [`y_twin`](https://deephaven.io/core/pydoc/code/deephaven.plot.figure.html#deephaven.plot.figure.Figure.y_twin), that support multiple `x` or multiple `y` axes on the same plot. Use [`x_twin`](https://deephaven.io/core/pydoc/code/deephaven.plot.figure.html#deephaven.plot.figure.Figure.x_twin) to create a plot with two different series that _share_ an `x` axis, but have different `y` axes:

```python
plot = (
    Figure()
    .new_chart()
    .plot_xy(series_name="Y1 trend", t=t, x="Timestamp", y="Y1")
    .x_twin()
    .plot_xy(series_name="Y2 trend", t=t, x="Timestamp", y="Y2")
    .show()
)
```

## Subplots

Deephaven's [subplots](https://deephaven.io/core/docs/how-to-guides/plotting/subplots/) allow you to arrange multiple plots into a grid. The [`Figure`](https://deephaven.io/core/pydoc/code/deephaven.plot.figure.html#deephaven.plot.figure.Figure) object supports `rows` and `cols` arguments that let you specify how many rows and columns you want in the grid. Then, the [`new_chart`](https://deephaven.io/core/pydoc/code/deephaven.plot.figure.html#deephaven.plot.figure.Figure.new_chart) method is used to position each plot in the larger grid. The following example creates a grid with one column and two rows:

```python
plot = (
    Figure(rows=3, cols=1)
    .new_chart(row=0, col=0)
    .plot_xy(series_name="Y1 trend", t=t, x="Timestamp", y="Y1")
    .plot_xy(series_name="Y1 rolling average", t=t, x="Timestamp", y="RollAvgY1")
    .new_chart(row=1, col=0)
    .plot_xy(series_name="Y3 trend", t=t, x="Timestamp", y="Y3")
    .plot_xy(series_name="Y3 rolling average", t=t, x="Timestamp", y="RollAvgY3")
    .show()
)
```

Deephaven offers many kinds of plots, like [histograms](https://deephaven.io/core/docs/how-to-guides/plotting/histogram/), [pie charts](https://deephaven.io/core/docs/how-to-guides/plotting/pie/), [scatter plots](https://deephaven.io/core/docs/how-to-guides/plotting/xy-series/#xy-series-as-a-scatter-plot), and more. There are also work-in-progress integrations with [Plotly-express](https://plotly.com/python/plotly-express/), [Matplotlib](https://matplotlib.org), and [Seaborn](https://seaborn.pydata.org) to extend real-time plotting to these popular libraries.
