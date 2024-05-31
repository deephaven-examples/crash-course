---
id: query-strings
title: Deephaven's Query Strings
sidebar_label: Query Strings
---

Deephaven query strings are the primary way of expressing commands directly to the Deephaven engine. They are responsible for translating the user's intention into compiled code that the engine can execute. These query strings can contain a mix of Java and Python code and are the entry point to a universe of powerful built-in tools and Python-Java interoperability.

## Syntax

Query strings should be written in _double-quoted_ strings. They are always passed as arguments to table operations:

```python
from deephaven import empty_table

t = empty_table(10).update("NewColumn = 1")
```

Here, the query string `"NewColumn = 1"` defines a formula for the engine to execute, and the [`update`](https://deephaven.io/core/docs/reference/table-operations/select/update/) table operation understands that formula as a recipe for creating a new column.

## Literals

Query strings often utilize [literals](<https://en.wikipedia.org/wiki/Literal_(computer_programming)>). The engine can infer different data types from literals:

```python
literals = empty_table(10).update(
    [
        # Booleans are represented by 'true' and 'false'
        "BooleanCol = true",
        # 32b integers are represented by integers such as '1'
        "IntCol = 1",
        # 64b integers are represented by integers with an 'L' suffix such as '10L'
        "LongCol = 10L",
        # 32b floating point numbers are represented by numbers with an 'f' suffix such as '1.2345f'
        "FloatCol = 1.2345f",
        # 64b floating point numbers are represented by numbers with a decimal point such as '2.3456'
        "DoubleCol = 2.3456",
        # Strings must be enclosed with backticks
        "StringCol = `Hello!`",
        # Date-time values must be enclosed with single quotes
        "DateCol = '2020-01-01'",
        "TimeCol = '12:30:45.000'",
        "DateTimeCol = '2020-01-01T12:30:45.000Z'",
        # Durations must be enclosed with single quotes and have a leading PT
        "DurationCol = 'PT1m'",
        # For object types, 'null' is the literal for null values
        "NullCol = null",
    ]
)
```

:::note
Query strings are enclosed in double quotes, string literals are enclosed in backticks, and date-time literals are enclosed in single quotes.
:::

The table attribute [`meta_table`](https://deephaven.io/core/docs/reference/table-operations/metadata/meta_table/) is useful for assessing data types. Use it to confirm that the literals are interpreted correctly:

```python
literals_meta = literals.meta_table
```

## Special variables & constants

Deephaven provides a number of [special variables and constants](https://deephaven.io/core/docs/reference/query-language/variables/special-variables/). The most commonly used of these are `i` and `ii`. Both represent row indices in a table as [different Java primitive types](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html). `i` is an `int`, while `ii` is a `long`:

```python
special_vars = empty_table(10).update(["IdxInt = i", "IdxLong = ii"])

special_meta = special_vars.meta_table
```

These variables are expensive to compute and are therefore not computed for most ticking tables. [Append-only tables](https://deephaven.io/core/docs/conceptual/table-types/#append-only) are an exception.

Additionally, Deephaven provides a range of common constants that can be accessed from query strings. These constants are always written with snake case in capital letters. They include [minimum and maximum values for various types](https://deephaven.io/core/javadoc/io/deephaven/util/QueryConstants.html), [conversion factors for time types](https://deephaven.io/core/javadoc/io/deephaven/time/DateTimeUtils.html), and more. Of particular interest are the null constants for primitive types:

```python
null_values = empty_table(1).update(
    [
        "StandardNull = null",
        "CharNull = NULL_CHAR",
        "BoolNull = NULL_BOOLEAN",
        "LongNull = NULL_LONG",
        "FloatNull = NULL_FLOAT",
        "IntNull = NULL_INT",
    ]
)

null_values_meta = null_values.meta_table
```

These are useful for representing null values of a specific type and ensuring that operations involving null values do not throw errors.

## Common operations

Mathematical operations like `+`, `-`, `*`, `/`, and `%` are supported between many different numeric types:

```python
math_ops = empty_table(10).update(
    [
        "Col1 = ii",
        # Math operations work between columns and literals
        "Col2 = 10 * (Col1 + 1.2345) / 4",
        # And they work between columns
        "Col3 = (Col1 - Col2) % 6",
    ]
)
```

The `+` operator can also be used for string concatenation:

```python
add_strings = empty_table(10).update("StringCol = `Hello ` + `there!`")
```

Additionally, `+` and `-` are defined for date-time types, making it easy to do arithmetic on timestamps:

```python
time_ops = empty_table(10).update(
    [
        # Times in nanoseconds can be added or subtracted from date-times
        "Timestamp = '2021-07-11T12:00:00.000Z' + (ii * HOUR)",
        # Durations or Periods can be added or subtracted from date-times
        "TimestampPlusOneYear = Timestamp + 'P365d'",
        "TimestampMinusOneHour = Timestamp - 'PT1h'",
        # Timestamps can be subtracted to get their difference in nanoseconds
        # Use constants for unit conversion
        "DaysBetween = ('2023-01-02T20:00:00.000Z' - Timestamp) / DAY",
    ]
)
```

Logical operations and expressions and comparison operators are supported:

```python
logical_ops = empty_table(10).update(
    [
        "IdxCol = ii",
        "IsEven = IdxCol % 2 == 0",
        "IsDivisibleBy6 = IsEven && (IdxCol % 3 == 0)",
        "IsGreaterThan6 = IdxCol > 6",
    ]
)
```

Deephaven provides an [in-line conditional operator](https://deephaven.io/core/docs/reference/query-language/control-flow/ternary-if/) that makes writing conditional expressions compact and efficient:

```python
conditional = empty_table(10).update(
    [
        # Read as 'condition ? if true : if false'
        "Parity = ii % 2 == 0 ? `Even!` : `Odd...`",
        # Any logical expression is a valid condition
        "IsDivisibleBy6 = ((ii % 2 == 0) && (ii % 3 == 0)) ? true : false",
        # In-line conditionals can be chained together
        "RandomNumber = randomGaussian(0.0, 1.0)",
        "Score = RandomNumber < -1.282 ? `Bottom 10%` : RandomNumber > 1.282 ? `Top 10%` : `Middle of the pack`",
    ]
)
```

In Deephaven, typecasting is done in query strings:

```python
typecasted = empty_table(10).update(
    [
        "LongCol = ii",
        # Use (type) to cast a column to the desired type
        "IntCol = (int)LongCol",
        "FloatCol = (float)LongCol",
        # Or, use a cast as an intermediate step
        "IntSquare = i * (int)LongCol",
    ]
)
```

Applying any of these basic operations to the basic `null` value results in an error:

```python
will_fail = empty_table(10).update("Col = ii + null")
```

However, all of these operations will work with Deephaven's null constants and return the appropriate null type:

```python
null_ops = empty_table(10).update(
    [
        "IntNull = i + NULL_INT",
        "FloatNull = (float)NULL_LONG",
        "BoolNull = true || NULL_BOOLEAN",
    ]
)
```

There are many more such operators supported in Deephaven. See the [guide on operators](https://deephaven.io/core/docs/reference/query-language/formulas/operators/) to learn more.

## Built-in functions

Aside from the common operations, Deephaven supplies a large library of functions known as [built-in or auto-imported functions](https://deephaven.io/core/docs/reference/query-language/query-library/auto-imported-functions/) that can be used in query strings.

The [numeric subset of this library](https://deephaven.io/core/javadoc/io/deephaven/function/package-summary.html) is full of functions for performing common mathematical operations on numeric types. These include exponentials, trigonometric functions, random number generators, and more:

```python
math_functions = empty_table(10).update(
    [
        # Each function can be applied to many different types
        "Exponential = exp(ii)",
        "NatLog = log(i)",
        "Square = pow((float)ii, 2)",
        "Sqrt = sqrt((double)i)",
        "Trig = sin(ii)",
        # Functions can be composed
        "AbsVal = abs(cos(ii))",
        # Random number generators generate one value for each cell by default
        "RandInt = randomInt(0, 5)",
        "RandNormal = randomGaussian(0, 1)",
    ]
)
```

These functions can be combined with the literals and operators discussed previously to generate complex expressions:

```python
fake_data = empty_table(100).update(
    [
        "Group = randomInt(1, 4)",
        "GroupIntercept = Group == 1 ? 0 : Group == 2 ? 5 : 10",
        "GroupSlope = abs(GroupIntercept * randomGaussian(0, 1))",
        "GroupVariance = pow(sin(Group), 2)",
        "Data = GroupIntercept + GroupSlope * ii + randomGaussian(0.0, GroupVariance)",
    ]
)
```

The built-in library also contains a number of functions for [working with date-time values](https://deephaven.io/core/javadoc/io/deephaven/time/DateTimeUtils.html):

```python
time_functions = empty_table(10).update(
    [
        # The now() function provides the current time according to the Deephaven clock
        "CurrentTime = now()",
        "Timestamp = CurrentTime + (ii * DAY)",
        # Many time functions require timezone information, typically provided as literals
        "DayOfWeek = dayOfWeek(Timestamp, 'ET')",
        "DayOfMonth = dayOfMonth(Timestamp, 'ET')",
        "Weekend = DayOfWeek == 6 || DayOfWeek == 7 ? true : false",
        "SecondsSinceY2K = nanosToSeconds(Timestamp - '2000-01-01T00:00:00 ET')",
    ]
)
```

Functions that begin with `parse` are useful for converting strings to date-time types:

```python
string_date_times = empty_table(10).update(
    [
        "DateTime = `2022-04-03T19:34:22.000 UTC`",
        "Date = `2013-07-07`",
        "Time = `14:04:39.123`",
        "Duration = `PT6m`",
        "Period = `P10d`",
        "TimeZone = `America/Chicago`",
    ]
)

converted_date_times = string_date_times.update(
    [
        "DateTime = parseInstant(DateTime)",
        "Date = parseLocalDate(Date)",
        "Time = parseLocalTime(Time)",
        "Duration = parseDuration(Duration)",
        "Period = parsePeriod(Period)",
        "TimeZone = parseTimeZone(TimeZone)",
    ]
)

string_meta = string_date_times.meta_table
converted_meta = converted_date_times.meta_table
```

The [`upperBin`](<https://deephaven.io/core/javadoc/io/deephaven/time/DateTimeUtils.html#upperBin(java.time.Instant,long)>) and [`lowerBin`](<https://deephaven.io/core/javadoc/io/deephaven/time/DateTimeUtils.html#lowerBin(java.time.Instant,long)>) functions are commonly used for timestamp binning, and are particularly useful for time-based aggregations:

```python
binned_timestamps = empty_table(60).update(
    [
        "Timestamp = now() + ii * MINUTE",
        # Rounds Timestamp to the last 5 minute mark
        "Last5Mins = lowerBin(Timestamp, 5 * MINUTE)",
        # Rounds Timestamp to the next 10 minute mark
        "Next10Mins = upperBin(Timestamp, 10 * MINUTE)",
    ]
)
```

You can learn more about time-specific operations in the [time guide](https://deephaven.io/core/docs/conceptual/time-in-deephaven/).

These functions are just a glimpse of what Deephaven's built-in library has to offer. There are modules for [sorting](https://deephaven.io/core/javadoc/io/deephaven/function/Sort.html), [searching](https://deephaven.io/core/javadoc/io/deephaven/function/BinSearch.html), [string parsing](https://deephaven.io/core/javadoc/io/deephaven/function/Parse.html), [null handling](<https://deephaven.io/core/javadoc/io/deephaven/function/Basic.html#isNull(byte)>), and much more. See the document on [auto-imported functions](../../reference/query-language/query-library/auto-imported-functions.md) for a comprehensive list, or the [module summary page](https://deephaven.io/core/javadoc/io/deephaven/function/package-summary.html) for a high-level overview of what's offered.

## Java methods

The data structures that underlie Deephaven tables are Java data structures. So, many of the literals, operators, and functions that we've spoken about are, at some level, Java objects. These Java objects have _methods_ attached to them that can be called from query strings, unlocking new levels of functionality and efficiency for Deephaven users.

To discover these, use [`meta_table`](https://deephaven.io/core/docs/reference/table-operations/metadata/meta_table/) to inspect a columns underlying data type:

```python
call_methods = empty_table(1).update("Timestamp = '2024-03-03T15:00:00.000 UTC'")

call_methods_meta = call_methods.meta_table
```

This column is a Java [`Instant`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Instant.html). If you navigate to this type's [documentation page](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Instant.html), you'll find a host of [methods](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Instant.html#method.summary) that can be called. Here are just a few:

```python
call_methods = call_methods.update(
    [
        "SecondsSinceEpoch = Timestamp.getEpochSecond()",
        "TimestampInTokyo = Timestamp.atZone('Asia/Tokyo')",
        "StringRepresentation = Timestamp.toString()",
    ]
)
```

Some basic understanding of [how to read Javadocs](https://deephaven.io/core/docs/how-to-guides/read-javadocs/) will help you make the most of these built-in methods.

Additionally, there are several ways to create Java objects and call methods in query strings. The following example uses the [`jpy`](https://jpy.readthedocs.io/en/latest/index.html) library to create new instances of Java's [`URL`](https://docs.oracle.com/javase%2F7%2Fdocs%2Fapi%2F%2F/java/net/URL.html) class outside of query strings:

```python
import jpy

# Use 'new' keyword to create a new URL object, then call methods
t1 = empty_table(2).update(
    [
        "URL = new java.net.URL(`https://deephaven.io:` + i)",
        "Protocol = URL.getProtocol()",
        "Host = URL.getHost()",
        "Port = URL.getPort()",
        "URI = URL.toURI()",
    ]
)
m1 = t1.meta_table

# Use 'jpy' to create a new URL object, pass object to query string, then call methods
URL = jpy.get_type("java.net.URL")
url = URL("https://deephaven.io")

t2 = empty_table(1).update(
    [
        "URL = url",
        "Protocol = URL.getProtocol()",
        "Host = URL.getHost()",
        "Port = URL.getPort()",
        "URI = URL.toURI()",
    ]
)
m2 = t2.meta_table

# Use 'jpy' to create a new URL object, call methods, then pass results to query string
URL = jpy.get_type("java.net.URL")
url = URL("https://deephaven.io")
protocol = url.getProtocol()
host = url.getHost()
port = url.getPort()
uri = url.toURI()

t3 = empty_table(1).update(
    [
        "URL = url",
        "Protocol = protocol",
        "Host = host",
        "Port = port",
        "URI = uri",
    ]
)
m3 = t3.meta_table
```

## Python in query strings

Python objects can also be used in query strings. Variables are the simplest case:

```python
a = 1
b = 2

add_vars = empty_table(1).update("Sum = a + b")
```

Python functions may be used in query strings:

```python
def my_sum(x, y):
    return x + y


a = 1
b = 2

add_vars_func = empty_table(1).update(["Sum1 = my_sum(1, 2)", "Sum2 = my_sum(a, b)"])
```

Query strings can even utilize classes:

```python
class MyMathClass:

    def __init__(self, x):
        self.x = x

    def sum(self, y):
        return self.x + y

    @classmethod
    def class_sum(self, x, y):
        return x + y

    @staticmethod
    def static_sum(x, y):
        return x + y


a = 1
b = 2
class_instance = MyMathClass(a)

add_vars_class = empty_table(1).update(
    [
        "Sum1 = class_instance.sum(b)",
        "Sum2 = MyMathClass.class_sum(a, b)",
        "Sum3 = MyMathClass.static_sum(a, b)",
    ]
)
```

When Deephaven is unable to infer the type of a column, as is the case for un-type-hinted Python functions or method calls, Deephaven stores the results as a [`PyObject`](https://deephaven.io/core/docs/how-to-guides/pyobjects/) or a [Java `Object`](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html):

```python
add_vars_meta = add_vars.meta_table
add_vars_func_meta = add_vars_func.meta_table
add_vars_class_meta = add_vars_class.meta_table
```

This is usually not ideal, as [`PyObject`s](https://deephaven.io/core/docs/how-to-guides/pyobjects/) and [Java `Objects`s](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html) do not support many of the DQL features we've discussed. To rectify this, functions and classes should utilize [type hints](https://peps.python.org/pep-0484/). The Deephaven engine will then automatically infer the correct column types for functions. Calling a class's methods currently requires a type cast, but Deephaven will soon also infer their correct column types:

```python
from deephaven import empty_table


def my_sum(x, y) -> int:
    return x + y


class MyMathClass:

    def __init__(self, x):
        self.x = x

    def sum(self, y) -> int:
        return self.x + y

    @classmethod
    def class_sum(self, x, y) -> int:
        return x + y

    @staticmethod
    def static_sum(x, y) -> int:
        return x + y


a = 1
b = 2
class_instance = MyMathClass(a)

add_vars_func = empty_table(1).update(["Sum1 = my_sum(1, 2)", "Sum2 = my_sum(a, b)"])

# Note the (int) casts
add_vars_class = empty_table(1).update(
    [
        "Sum1 = (int)class_instance.sum(b)",
        "Sum2 = (int)MyMathClass.class_sum(a, b)",
        "Sum3 = (int)MyMathClass.static_sum(a, b)",
    ]
)

add_vars_func_meta = add_vars_func.meta_table
add_vars_class_meta = add_vars_class.meta_table
```

To learn more about using Python in query strings, see the user guides on [functions](https://deephaven.io/core/docs/how-to-guides/simple-python-function/) and [classes](https://deephaven.io/core/docs/how-to-guides/python-classes/) for more details.

Scoping in Deephaven follows Python's [LEGB](https://realpython.com/python-scope-legb-rule/) scoping rules. Functions that return tables or otherwise make use of query strings should pay careful attention to scoping details:

```python
def f(a, b) -> int:
    return a * b


def compute(source, a):
    return source.update("X = f(a, A)")


source = empty_table(5).update("A = i")

result1 = compute(source, 10)
result2 = compute(source, 3)
```

For more information, see the [guide on scoping](https://deephaven.io/core/docs/how-to-guides/query-scope-how-to/#encapsulate-query-logic-in-functions).

Lastly, when Python functions are used in conjunction with Deephaven tables, it's important to mind whether the functions are stateless or stateful. Generally, stateless functions have no side effects - they don't modify any objects outside of their scope. Also, they are invariant to execution order, so function calls can be evaluated in any order without affecting the result. This stateless function extracts elements from a list in a query string:

```python
my_list = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]


def get_element(idx) -> int:
    return my_list[idx]


t = empty_table(10).update("X = get_element(ii)")
```

The function `get_element` does not modify any objects outside of its local scope. Further, the 10 calls to `get_element` (one for each row) could be evaluated in any order and still give the correct result. So, it is a stateless function.

On the other hand, stateful functions _do_ modify objects outside of their scope, and they do not leave the world as they found it. They may also depend on execution order. This stateful function achieves the same resulting table:

```python
my_list = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
idx = 0


def get_element() -> int:
    global idx
    idx += 1  # This modifies idx!
    return my_list[idx - 1]


t = empty_table(10).update("X = get_element()")
```

The function `get_element` is stateful - it has the side effect of modifying the value of `idx`. This can be verified by printing the current value of `idx`:

```python
print(idx)
```

Hopefully, it's also clear that if `get_element` were called on each row in a different order, the results would not be correct.

Stateless functions should be used wherever possible in Deephaven. Because they have no side effects and are invariant to execution order, they are deterministic and amenable to many under-the-hood optimizations, such as query parallelization. Parallelization will soon be the default in Deephaven, and serial execution will require explicit specification.

Stateful functions, while currently functional with static or append-only tables, will require explicit annotations so that they are executed serially when Deephaven defaults to query parallelization. Without the annotations, stateful functions may give inconsistent and surprising results.

## Arrays

Occasionally, you might run into arrays stored in Deephaven tables. Typically, these arise as the result of a [`group_by`](https://deephaven.io/core/docs/reference/table-operations/group-and-aggregate/groupBy/) operation:

```python
t = empty_table(10).update(["Group = ii % 2 == 0 ? `A` : `B`", "X = ii"])

t_grouped = t.group_by("Group")
```

Many built-in functions support array arguments:

```python test-set=4
t_array_funcs = t_grouped.update(
    [
        "IsEven = X % 2 == 0",
        "PlusOne = X + 1",
        "GroupSum = sum(X)",
        "GroupAvg = avg(X)",
        "NumUnique = countDistinct(X)",
    ]
)
```

These results can then be ungrouped with [`ungroup`](https://deephaven.io/core/docs/reference/table-operations/group-and-aggregate/ungroup/), which is essentially the inverse of [`group_by`](https://deephaven.io/core/docs/reference/table-operations/group-and-aggregate/groupBy/):

```python test-set=4
t_array_funcs_ungrouped = t_array_funcs.ungroup()
```

:::note
In most cases, aggregating with the [`agg`](https://deephaven.io/core/docs/how-to-guides/combined-aggregations/) suite is more performant than aggregating with a [`group_by`](https://deephaven.io/core/docs/reference/table-operations/group-and-aggregate/groupBy/) operation and a query language function.
:::

Deephaven provides an array indexing and slicing syntax for such arrays:

```python
t_indexed = t_grouped.update(
    [
        "First = X[0]",
        "Third = X[2]",
        # Note that .subVector() includes the first arg and does not include the second
        "FirstThree = X.subVector(0, 3)",
        "MiddleThree = X.subVector(1, 4)",
    ]
)
```

Lastly, the columns of a Deephaven table can be interpreted as arrays:

```python
column_as_array = empty_table(10).update(
    [
        "X = ii",
        # To interpret X as an array, use a trailing underscore
        "ArrX = X_",
        # Functions that take array inputs can now be used
        "SumX = sum(X_)",
        "AvgX = avg(X_)",
        # Indexing and slicing work just the same
        "IdxX = X_[ii]",
        "RollingGroup = X_.subVector(i, i+3)",
        "RollingAvg = avg(RollingGroup)",
    ]
)
```

This functionality is only supported for static tables and append-only ticking tables. Check out the guide on [working with arrays in Deephaven](https://deephaven.io/core/docs/how-to-guides/work-with-arrays/) for more information.
