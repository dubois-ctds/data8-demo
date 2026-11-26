# Data 8 to Polars Reference

Mapping of [Data 8 Python Reference](https://data8.org/datascience/reference-nb/datascience-reference.html) functions and methods to Polars equivalents. Use this when converting notebooks from the <code style="color:#c7254e;background-color:#f9f2f4">datascience</code> library to Polars.

**Conventions**

- <code style="color:#002fa7;background-color:#eef2fb">df</code> is a Polars DataFrame. `s` is a **Polars Series**, created as <code style="color:#002fa7;background-color:#eef2fb">pl.Series("name", values)</code>. A Pandas Series appears only after <code style="color:#002fa7;background-color:#eef2fb">df.to&#95;pandas()</code>.
- Inside <code style="color:#002fa7;background-color:#eef2fb">filter</code>, <code style="color:#002fa7;background-color:#eef2fb">with&#95;columns</code>, <code style="color:#002fa7;background-color:#eef2fb">select</code>, and <code style="color:#002fa7;background-color:#eef2fb">group&#95;by</code>, refer to a column with the lazy expression <code style="color:#002fa7;background-color:#eef2fb">pl.col("name")</code>. The same expression works on a DataFrame and on <code style="color:#002fa7;background-color:#eef2fb">df.lazy()</code>.
- **Arrays are built with** <code style="color:#c7254e;background-color:#f9f2f4">make&#95;array</code>**.** <code style="color:#c7254e;background-color:#f9f2f4">make&#95;array(1, 2)</code> stays <code style="color:#c7254e;background-color:#f9f2f4">make&#95;array(1, 2)</code>, imported with <code style="color:#c7254e;background-color:#f9f2f4">from datascience import make&#95;array</code>. It returns an ordinary NumPy array. <code style="color:#c7254e;background-color:#f9f2f4">np.array(...)</code> is used only to *convert* an existing sequence, such as a row tuple (<code style="color:#002fa7;background-color:#eef2fb"><span style="color:#c7254e">np.array</span>(df.row(0))</code>), never to build an array from values written out by hand. <code style="color:#c7254e;background-color:#f9f2f4">make&#95;array()</code> is the empty float array.
- **Arrays, not lists.** Values are never written as a Python list `[...]`. A list stays only where a library asks for one: the list of Series in <code style="color:#002fa7;background-color:#eef2fb">pl.DataFrame(&#91;...&#93;)</code> and <code style="color:#002fa7;background-color:#eef2fb">pl.concat(&#91;...&#93;)</code>, column names (`y=[...]`, `kdims`), colors (`color=[...]`), map coordinates (`[lat, lon]`), HoloViews shape lists (<code style="color:#002fa7;background-color:#eef2fb">hv.Segments(&#91;(x0, y0, x1, y1)&#93;)</code>, <code style="color:#002fa7;background-color:#eef2fb">hv.Rectangles</code>), and a column whose cells are themselves lists. A list also stays for five patterns kept as Python lists on purpose: plot coordinates (<code style="color:#002fa7;background-color:#eef2fb">hv.Scatter((&#91;x&#93;, &#91;y&#93;))</code>, <code style="color:#002fa7;background-color:#eef2fb">hv.Curve(&#91;(x1, y1), (x2, y2)&#93;)</code>, <code style="color:#002fa7;background-color:#eef2fb">hv.Curve((&#91;x1, x2&#93;, &#91;y1, y2&#93;))</code>), a growing Python list (`errors = []` + `errors.append(v)`, Ch 17.6), the Ch 15.3 `sample = [[x, y], ...]` point list looped over for error lines, repeated labels (<code style="color:#c7254e;background-color:#f9f2f4">np.array(&#91;'A'&#93;&#42;n + &#91;'B'&#93;&#42;m)</code>, Ch 18), and several column names (<code style="color:#002fa7;background-color:#eef2fb">group&#95;by(&#91;"Flavor", "Color"&#93;)</code>, `sort([...])`, `select([...])`).
- **Array or Series?** A standalone array, one whose values are never added to a DataFrame, stays a **NumPy array** and uses NumPy functions: <code style="color:#c7254e;background-color:#f9f2f4">np.mean(arr)</code>, <code style="color:#c7254e;background-color:#f9f2f4">np.append(arr, x)</code>, <code style="color:#c7254e;background-color:#f9f2f4">np.count&#95;nonzero(arr)</code>. An array that is added to a DataFrame is wrapped in a named Series, <code style="color:#002fa7;background-color:#eef2fb">pl.Series("name", <span style="color:#c7254e">arr</span>)</code>, and from then on uses Series or <code style="color:#002fa7;background-color:#eef2fb">pl.col</code> methods. See [Arrays: standalone array or DataFrame column?](#arrays-standalone-array-or-dataframe-column).
- **Building a DataFrame:** pass a list of named Series, each built from an array, <code style="color:#002fa7;background-color:#eef2fb">pl.DataFrame(&#91;pl.Series("a", <span style="color:#c7254e">make&#95;array(...)</span>), pl.Series("b", <span style="color:#c7254e">make&#95;array(...)</span>)&#93;)</code>, and add a column with <code style="color:#002fa7;background-color:#eef2fb">df.with&#95;columns(pl.Series("c", values))</code>. Do not build a DataFrame from a dictionary (<code style="color:#002fa7;background-color:#eef2fb">pl.DataFrame({"a": &#91;...&#93;})</code>).
- Integer sequences and loops use <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code>, never Python `range`: <code style="color:#c7254e;background-color:#f9f2f4">for i in np.arange(5000)</code>. A float grid also stays <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code>; do not replace it with <code style="color:#002fa7;background-color:#eef2fb">pl.linear&#95;space</code>. Inside an expression, an index column can be <code style="color:#002fa7;background-color:#eef2fb">pl.int&#95;range(0, pl.len())</code>.
- Group reductions use a [GroupBy method](https://docs.pola.rs/api/python/stable/reference/dataframe/group_by.html) such as <code style="color:#002fa7;background-color:#eef2fb">.sum()</code> or <code style="color:#002fa7;background-color:#eef2fb">.max()</code>. See [GroupBy methods](#groupby-methods).

**Description:** The "Description" column gives a brief description of what the Polars function does, including what each argument is used for.

**Polars input/output:** The "Polars input" and "Polars output" columns follow the format of the Data 8 reference sheet and are checked against the [Polars Python API reference](https://docs.pola.rs/api/python/stable/reference/) (DataFrame, Series, Expr, I/O, and GroupBy).

- **Input** lists each argument as **type**: what it is for. When there are several, they are numbered in order (1., 2., ...), and an argument that can be left out starts with "(Optional)". "None" means the call takes no arguments, as with a method like <code style="color:#002fa7;background-color:#eef2fb">s.mean()</code> or an attribute like <code style="color:#002fa7;background-color:#eef2fb">df.height</code>.
- **Output** gives the **type** that comes back, then what it holds, for example "**DataFrame**: a copy of the original DataFrame with only the rows that match". A call that only displays something, such as <code style="color:#002fa7;background-color:#eef2fb">df.show(n)</code>, has the output "None: displays ...".
- **Code colors:** pink code is the Data 8 <code style="color:#c7254e;background-color:#f9f2f4">datascience</code> library or NumPy (the same pink as the code on the [Data 8 reference page](https://data8.org/fa26/reference/)). Blue code is Polars or hvPlot. Inside a line of Polars code, a <code style="color:#c7254e;background-color:#f9f2f4">make&#95;array(...)</code> or <code style="color:#c7254e;background-color:#f9f2f4">np.</code> call stays pink. The colors show in a markdown preview that keeps inline HTML styles, such as VS Code; GitHub removes them.
- The types are **string**, **int**, **float**, **bool**, **number** (int or float), **scalar** (one value), **array** (NumPy), **list**, **dict**, **tuple**, **function**, **Series**, **DataFrame**, **Expr** (a <code style="color:#002fa7;background-color:#eef2fb">pl.col</code> expression), and **plot** (an hvPlot chart).

---

## Built-In Python Functions

| Data 8                                     | Polars / Python                           | Description                                                                             | Polars input                                                                                                                              | Polars output                                                                 | Notes                                                                                                                |
| ------------------------------------------ | ----------------------------------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| <code style="color:#c7254e;background-color:#f9f2f4">str(val)</code> | <code style="color:#002fa7;background-color:#eef2fb">str(val)</code> | Converts `val` to a **string**. For a Polars Series or lazy column, cast every element. | **any**: the value to convert | **string**: the value as text | Scalars: `str(val)`. Series: <code style="color:#002fa7;background-color:#eef2fb">s.cast(pl.Utf8)</code>. Expression: <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").cast(pl.Utf8)</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">int(num)</code> | <code style="color:#002fa7;background-color:#eef2fb">int(num)</code> | Converts `num` to an **int**. For a column, cast every element. | **float** or **string**: the number to convert | **int**: the converted number | Scalars: `int(num)`. Series: <code style="color:#002fa7;background-color:#eef2fb">s.cast(pl.Int64)</code>. Expression: <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").cast(pl.Int64)</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">float(num)</code> | <code style="color:#002fa7;background-color:#eef2fb">float(num)</code> | Converts `num` to a **float**. For a column, cast every element. | **int** or **string**: the number to convert | **float**: the converted number | Scalars: `float(num)`. Series: <code style="color:#002fa7;background-color:#eef2fb">s.cast(pl.Float64)</code>. Expression: <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").cast(pl.Float64)</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">len(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">len(s)</code> or <code style="color:#002fa7;background-color:#eef2fb">s.len()</code> or <code style="color:#002fa7;background-color:#eef2fb">df.height</code> | Length of a Polars Series, or the number of rows in a DataFrame. | **Series** or **DataFrame**: the values or table to measure (None for <code style="color:#002fa7;background-color:#eef2fb">s.len()</code> and <code style="color:#002fa7;background-color:#eef2fb">df.height</code>) | **int**: the number of elements or rows | Row count inside an expression: <code style="color:#002fa7;background-color:#eef2fb">pl.len()</code>. Python `len(s)` works on a Polars Series, so `sum(s) / len(s)` can stay. |
| <code style="color:#c7254e;background-color:#f9f2f4">max(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.max()</code> | Maximum value in a Polars Series. Per group, use <code style="color:#002fa7;background-color:#eef2fb">GroupBy.max()</code>. | None | **scalar**: the largest value in the Series | <code style="color:#002fa7;background-color:#eef2fb">s = df.get&#95;column("col")</code>. Expression: <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").max()</code> inside <code style="color:#002fa7;background-color:#eef2fb">select</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">min(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.min()</code> | Minimum value in a Polars Series. Per group, use <code style="color:#002fa7;background-color:#eef2fb">GroupBy.min()</code>. | None | **scalar**: the smallest value in the Series | Series method, or <code style="color:#002fa7;background-color:#eef2fb">pl.col</code> inside <code style="color:#002fa7;background-color:#eef2fb">select</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">sum(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.sum()</code> | Sum of the values in a Polars Series. Per group, use <code style="color:#002fa7;background-color:#eef2fb">GroupBy.sum()</code>. | None | **scalar**: the sum of the values in the Series | Series method, or <code style="color:#002fa7;background-color:#eef2fb">pl.col</code> inside <code style="color:#002fa7;background-color:#eef2fb">select</code>. Python <code style="color:#002fa7;background-color:#eef2fb">sum(s)</code> also works on a Polars Series. |
| <code style="color:#c7254e;background-color:#f9f2f4">abs(num)</code> | <code style="color:#002fa7;background-color:#eef2fb">abs(num)</code> or <code style="color:#002fa7;background-color:#eef2fb">s.abs()</code> | Absolute value of a number, or of each element. | **number**: the value for <code style="color:#002fa7;background-color:#eef2fb">abs(num)</code> (None for <code style="color:#002fa7;background-color:#eef2fb">s.abs()</code>) | **number** or **Series**: the absolute value of the number or of each element | Expression form stays lazy inside <code style="color:#002fa7;background-color:#eef2fb">with&#95;columns</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">round(number)</code> / <code style="color:#c7254e;background-color:#f9f2f4">round(number, ndigits)</code> | <code style="color:#002fa7;background-color:#eef2fb">round(x, ndigits)</code> or <code style="color:#002fa7;background-color:#eef2fb">s.round(ndigits)</code> | Rounds a number or each element. `ndigits` is digits after the decimal point. | 1. **number**: the value to round (left out for <code style="color:#002fa7;background-color:#eef2fb">s.round</code>) 2. (Optional) **int**: <code style="color:#002fa7;background-color:#eef2fb">ndigits</code>, the number of digits after the decimal point | **number** or **Series**: the rounded value(s) | Scalar, Series, or <code style="color:#002fa7;background-color:#eef2fb">pl.col</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">print(input, ...)</code> | <code style="color:#002fa7;background-color:#eef2fb">print(...)</code> | Prints the input. Multiple inputs are separated by spaces by default. | **any**: one or more values to print | None: prints the values | Same. |
| <code style="color:#c7254e;background-color:#f9f2f4">type(object)</code> | <code style="color:#002fa7;background-color:#eef2fb">type(object)</code> | Returns the type of `object`. | **any**: the object | **type**: the type of the object | Same. For a column dtype, use <code style="color:#002fa7;background-color:#eef2fb">s.dtype</code> or <code style="color:#002fa7;background-color:#eef2fb">df.schema</code>. |

---

## Arrays: standalone array or DataFrame column?

A Data 8 array becomes one of two things, depending on **where its values go**. Ask: *will these values be added to a DataFrame?*

| The array… | Polars + NumPy | Then use |
| --- | --- | --- |
| is never added to a DataFrame: a simulation's results, the options to draw from, histogram bins, an index list | a **NumPy array**: <code style="color:#c7254e;background-color:#f9f2f4">make&#95;array(...)</code>, <code style="color:#c7254e;background-color:#f9f2f4">np.arange(...)</code>, <code style="color:#c7254e;background-color:#f9f2f4">np.append(arr, x)</code> | NumPy functions: <code style="color:#c7254e;background-color:#f9f2f4">np.mean(arr)</code>, <code style="color:#c7254e;background-color:#f9f2f4">np.count&#95;nonzero(arr)</code>, <code style="color:#c7254e;background-color:#f9f2f4">arr.item(0)</code> |
| is added to a DataFrame: a new column, or the values of a table built with <code style="color:#002fa7;background-color:#eef2fb">pl.DataFrame</code> | a **named Series**: <code style="color:#002fa7;background-color:#eef2fb">pl.Series("name", <span style="color:#c7254e">arr</span>)</code>, passed to <code style="color:#002fa7;background-color:#eef2fb">pl.DataFrame(&#91;...&#93;)</code> or <code style="color:#002fa7;background-color:#eef2fb">df.with&#95;columns(...)</code> | Series or <code style="color:#002fa7;background-color:#eef2fb">pl.col</code> methods: <code style="color:#002fa7;background-color:#eef2fb">pl.col("name").mean()</code>, <code style="color:#002fa7;background-color:#eef2fb">s.mean()</code> |
| comes out of a DataFrame: <code style="color:#002fa7;background-color:#eef2fb">df.get&#95;column("col")</code> | already a Polars Series | Series methods (<code style="color:#002fa7;background-color:#eef2fb">s.mean()</code>, <code style="color:#002fa7;background-color:#eef2fb">s.sum()</code>), or <code style="color:#002fa7;background-color:#eef2fb">s.to&#95;numpy()</code> when the next step needs NumPy |

Don't wrap a standalone array in <code style="color:#002fa7;background-color:#eef2fb">pl.Series</code> just to call a Series method, and don't convert back and forth. The same values can start as a NumPy array and become a Series only at the point they are added to a table:

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7"><span style="color:#c7254e">import numpy as np</span>
<span style="color:#c7254e">from datascience import make_array</span>
import polars as pl

<span style="color:#6a737d"># Standalone: the simulated outcomes are not a table yet, so they stay NumPy</span>
die = <span style="color:#c7254e">np.arange(1, 7)</span>
outcomes = <span style="color:#c7254e">make_array()</span>
for i in <span style="color:#c7254e">np.arange(1000)</span>:
    outcomes = <span style="color:#c7254e">np.append(outcomes, np.random.choice(die))</span>
print(<span style="color:#c7254e">np.mean(outcomes)</span>, <span style="color:#c7254e">np.count_nonzero(outcomes == 6)</span>)

<span style="color:#6a737d"># Added to a DataFrame: wrap the array in a named Series at that point</span>
rolls = pl.DataFrame([pl.Series("Outcome", outcomes)])
rolls = rolls.with_columns(pl.Series("Roll", <span style="color:#c7254e">np.arange(1, rolls.height + 1)</span>))
print(rolls.filter(pl.col("Outcome") == 6).height)</code></pre>

In the table below, the **Standalone array** column is the NumPy form; the **Polars Series** column is the form for values that are a DataFrame column or a Series taken from one.

| Data 8                                                                        | Standalone array (NumPy)                                               | Polars Series (a column)                                                                              | Description                                                                                                                                                                                                                    | Polars input                                                                                                                   | Polars output                                                                                                                            | Notes                                                                                                                                                                                                                                                                                         |
| ----------------------------------------------------------------------------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <code style="color:#c7254e;background-color:#f9f2f4">make&#95;array(val1, val2, ...)</code> | <code style="color:#c7254e;background-color:#f9f2f4">make&#95;array(val1, val2, ...)</code> (unchanged) | <code style="color:#002fa7;background-color:#eef2fb">pl.Series("name", <span style="color:#c7254e">make&#95;array(val1, val2, ...)</span>)</code> | One Polars Series of scalar values. Never <code style="color:#c7254e;background-color:#f9f2f4">np.array(&#91;...&#93;)</code> and never a list <code style="color:#002fa7;background-color:#eef2fb">pl.Series("name", &#91;...&#93;)</code>. For a column of lists, use <code style="color:#002fa7;background-color:#eef2fb">pl.concat&#95;list</code>. | 1. **string**: the name of the Series 2. **array**: the values, <code style="color:#c7254e;background-color:#f9f2f4">make&#95;array(...)</code> | **Series**: a named Polars Series holding the values | Example: <code style="color:#002fa7;background-color:#eef2fb">pl.Series("highs", <span style="color:#c7254e">make&#95;array(v1, v2, v3, v4)</span>)</code>. [example](#example-plseries-vs-plconcat_list) |
| <code style="color:#c7254e;background-color:#f9f2f4">arr.size</code> | <code style="color:#c7254e;background-color:#f9f2f4">arr.size</code> or <code style="color:#c7254e;background-color:#f9f2f4">len(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.len()</code> | Number of elements. | None | **int**: the number of elements | Data 8 and NumPy use the attribute `size`. Polars uses the method `len()`, as in `highs.len()`. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.array(obj)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.array(obj)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.Series("x", obj)</code> | Converts an existing sequence `obj` (a NumPy array, or a `row` from <code style="color:#002fa7;background-color:#eef2fb">df.row(i)</code>) into a Polars Series. As in Data 8, <code style="color:#c7254e;background-color:#f9f2f4">np.array</code> *converts*; <code style="color:#c7254e;background-color:#f9f2f4">make&#95;array</code> *creates*. The result name in arithmetic comes from the first Series. | 1. **string**: the name of the Series 2. **array**, **tuple**, or **list**: the existing sequence to convert | **Series**: a named Series with the same values | <code style="color:#002fa7;background-color:#eef2fb">pl.Series("x", <span style="color:#c7254e">make&#95;array(1, 2, 3)</span>)</code>; a table row: <code style="color:#002fa7;background-color:#eef2fb">pl.Series("row", df.row(0))</code>. See [Converting between arrays and Series](#converting-between-arrays-and-series). [example](#example-series-arithmetic) |
| <code style="color:#c7254e;background-color:#f9f2f4">max(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">max(arr)</code> or <code style="color:#c7254e;background-color:#f9f2f4">np.max(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.max()</code> | Maximum value of the Series. | None | **scalar**: the largest value in the Series | Same as the built-in. Python `max(s)` also works. |
| <code style="color:#c7254e;background-color:#f9f2f4">min(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">min(arr)</code> or <code style="color:#c7254e;background-color:#f9f2f4">np.min(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.min()</code> | Minimum value of the Series. | None | **scalar**: the smallest value in the Series | Same as the built-in. Python `min(s)` also works. |
| <code style="color:#c7254e;background-color:#f9f2f4">sum(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">sum(arr)</code> or <code style="color:#c7254e;background-color:#f9f2f4">np.sum(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.sum()</code> | Sum of the values in the Series. | None | **scalar**: the sum of the values in the Series | Same as the built-in. Python <code style="color:#002fa7;background-color:#eef2fb">sum(s)</code> also works. |
| <code style="color:#c7254e;background-color:#f9f2f4">len(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">len(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">len(s)</code> or <code style="color:#002fa7;background-color:#eef2fb">s.len()</code> | Number of elements in the Series. | **Series**: the Series to measure (None for <code style="color:#002fa7;background-color:#eef2fb">s.len()</code>) | **int**: the number of elements | Same as the built-in. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.mean(arr)</code> / <code style="color:#c7254e;background-color:#f9f2f4">np.average(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.mean(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.mean()</code> | Mean of the Series. Per group: <code style="color:#002fa7;background-color:#eef2fb">GroupBy.mean()</code>. | None | **float**: the mean of the Series | Grouped form selects numeric columns first. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.std(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.std(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.std(ddof=0)</code> | Standard deviation of the Series. `ddof` is the delta degrees of freedom: the divisor is `n - ddof`. | (Optional) **int**: <code style="color:#002fa7;background-color:#eef2fb">ddof</code>, the delta degrees of freedom (default <code style="color:#002fa7;background-color:#eef2fb">1</code>; pass <code style="color:#002fa7;background-color:#eef2fb">ddof=0</code> to match <code style="color:#c7254e;background-color:#f9f2f4">np.std</code>) | **float**: the standard deviation of the Series | <code style="color:#c7254e;background-color:#f9f2f4">np.std</code> divides by `n` (population SD), so pass <code style="color:#002fa7;background-color:#eef2fb">ddof=0</code>. Polars <code style="color:#002fa7;background-color:#eef2fb">s.std()</code> defaults to `ddof=1` (sample SD) and gives a different number. Expression form: <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").std(ddof=0)</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.sum(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.sum(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.sum()</code> | Sum of the Series. Per group: <code style="color:#002fa7;background-color:#eef2fb">GroupBy.sum()</code>. | None | **scalar**: the sum of the values in the Series | Same as the built-in for one Series. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.prod(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.prod(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.product()</code> | Product of the values in the Series. | None | **scalar**: the product of the values in the Series | There is no GroupBy product method. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.sqrt(num)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.sqrt(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.sqrt()</code> | Square root of each element. A Python number uses `num ** 0.5`. | None | **Series**: the square root of each element | [example](#example-series-arithmetic) |
| <code style="color:#c7254e;background-color:#f9f2f4">abs(num)</code> / <code style="color:#c7254e;background-color:#f9f2f4">np.abs(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.abs(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.abs()</code> | Absolute value of each element of the Series. A Python number uses `abs(num)`. | None | **Series**: the absolute value of each element | Keeps the Series name and integer dtype. |
| <code style="color:#c7254e;background-color:#f9f2f4">round(num)</code> / <code style="color:#c7254e;background-color:#f9f2f4">np.round(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.round(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.round(0)</code> | Rounds each element to the nearest integer. <code style="color:#002fa7;background-color:#eef2fb">decimals</code> is the number of digits after the decimal point. | (Optional) **int**: <code style="color:#002fa7;background-color:#eef2fb">decimals</code>, the number of digits after the decimal point (default <code style="color:#002fa7;background-color:#eef2fb">0</code>) | **Series**: each element rounded to the nearest integer | Ties round to the nearest even value, as in NumPy (`2.5` becomes `2.0`). A float Series stays float. Scalars: `round(num)`. Expression: <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").round(0)</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">round(num, decimals)</code> / <code style="color:#c7254e;background-color:#f9f2f4">np.round(arr, decimals)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.round(arr, decimals)</code> (negative <code style="color:#c7254e;background-color:#f9f2f4">decimals</code> works) | <code style="color:#002fa7;background-color:#eef2fb">s.round(decimals)</code> | Rounds each element to <code style="color:#002fa7;background-color:#eef2fb">decimals</code> digits after the decimal point. | **int**: <code style="color:#002fa7;background-color:#eef2fb">decimals</code>, the number of digits after the decimal point (must be <code style="color:#002fa7;background-color:#eef2fb">&gt;= 0</code>) | **Series**: each element rounded to <code style="color:#002fa7;background-color:#eef2fb">decimals</code> digits | **Negative** <code style="color:#002fa7;background-color:#eef2fb">decimals</code> **raises an error in Polars.** To remove digits of precision, scale first: `(s / 100).round(0) * 100` for `decimals=-2`. The result is a float. See [Rounding with negative decimals](#example-rounding-with-negative-decimals). |
| <code style="color:#c7254e;background-color:#f9f2f4">np.arange(stop)</code> / <code style="color:#c7254e;background-color:#f9f2f4">np.arange(start, stop)</code> / <code style="color:#c7254e;background-color:#f9f2f4">np.arange(start, stop, step)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.arange(start, stop, step)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.Series("name", <span style="color:#c7254e">np.arange(start, stop, step)</span>)</code>, or the expression <code style="color:#002fa7;background-color:#eef2fb">pl.int&#95;range(start, end, step)</code> | Integers from `start` up to but excluding `stop`, stepping by an integer `step`. Defaults: `start=0`, `step=1`. One argument is the end. | 1. (Optional) **int**: <code style="color:#002fa7;background-color:#eef2fb">start</code> (default <code style="color:#002fa7;background-color:#eef2fb">0</code>) 2. **int**: <code style="color:#002fa7;background-color:#eef2fb">stop</code>, which is excluded 3. (Optional) **int**: <code style="color:#002fa7;background-color:#eef2fb">step</code> (default <code style="color:#002fa7;background-color:#eef2fb">1</code>) | **Series** of ints (<code style="color:#002fa7;background-color:#eef2fb">Int64</code>): the integers from <code style="color:#002fa7;background-color:#eef2fb">start</code> up to but excluding <code style="color:#002fa7;background-color:#eef2fb">stop</code> (an **Expr** for <code style="color:#002fa7;background-color:#eef2fb">pl.int&#95;range</code> without <code style="color:#002fa7;background-color:#eef2fb">eager=True</code>) | Output dtype is an integer such as <code style="color:#002fa7;background-color:#eef2fb">Int64</code>. Start is inclusive and end is exclusive. An eager <code style="color:#002fa7;background-color:#eef2fb">pl.int&#95;range(..., eager=True)</code> Series is named `"literal"` unless <code style="color:#002fa7;background-color:#eef2fb">.alias</code> changes it. Loops use <code style="color:#c7254e;background-color:#f9f2f4">for i in np.arange(10)</code>, never Python `range`. A float step stays <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.arange(start, stop, float&#95;step)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.arange(start, stop, float&#95;step)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.arange(start, stop, float&#95;step)</code> | Equal-spaced floats. Keep <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code>. | 1. **float**: <code style="color:#002fa7;background-color:#eef2fb">start</code> 2. **float**: <code style="color:#002fa7;background-color:#eef2fb">stop</code>, which is excluded 3. **float**: <code style="color:#002fa7;background-color:#eef2fb">step</code> | **array** of floats: equally spaced values from <code style="color:#002fa7;background-color:#eef2fb">start</code> up to but excluding <code style="color:#002fa7;background-color:#eef2fb">stop</code> | Do not replace this with <code style="color:#002fa7;background-color:#eef2fb">pl.linear&#95;space</code>. Histogram bins and other float grids stay <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code>, including <code style="color:#c7254e;background-color:#f9f2f4">np.arange(1.5, -2, -0.5)</code> and <code style="color:#c7254e;background-color:#f9f2f4">np.arange(30.5, 69.6, 1)</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.count&#95;nonzero(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.count&#95;nonzero(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">(s != 0).sum()</code> | Number of non-zero elements. On a boolean Series, <code style="color:#002fa7;background-color:#eef2fb">s.sum()</code> counts `True`. | None | **int**: the number of non-zero (or <code style="color:#002fa7;background-color:#eef2fb">True</code>) elements | <code style="color:#002fa7;background-color:#eef2fb">(pl.col("col") != 0).sum()</code> inside <code style="color:#002fa7;background-color:#eef2fb">select</code>. A boolean Series has no `!=` comparison, so count those with <code style="color:#002fa7;background-color:#eef2fb">s.sum()</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.all(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.all(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">(s != 0).all()</code> | True when every numeric element is non-zero. | None | **bool**: <code style="color:#002fa7;background-color:#eef2fb">True</code> if every element is non-zero | <code style="color:#002fa7;background-color:#eef2fb">Series.all</code> requires a boolean Series. A boolean Series uses <code style="color:#002fa7;background-color:#eef2fb">s.all()</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.any(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.any(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">(s != 0).any()</code> | True when any numeric element is non-zero. | None | **bool**: <code style="color:#002fa7;background-color:#eef2fb">True</code> if any element is non-zero | <code style="color:#002fa7;background-color:#eef2fb">Series.any</code> requires a boolean Series. A boolean Series uses <code style="color:#002fa7;background-color:#eef2fb">s.any()</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.append(arr, item)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.append(arr, item)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.concat(&#91;s, pl.Series(s.name, <span style="color:#c7254e">make&#95;array(item)</span>)&#93;)</code> | A new Series with `item` at the end. The original Series is unchanged. | 1. **Series**: the original Series 2. **Series**: a one-element Series holding <code style="color:#002fa7;background-color:#eef2fb">item</code>, with the same name | **Series**: a new Series with <code style="color:#002fa7;background-color:#eef2fb">item</code> at the end | Polars builds a new Series. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.cumsum(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.cumsum(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.cum&#95;sum()</code> | Cumulative sum. Each element includes itself and everything before it. | None | **Series**: the running total at each position | Running total. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.diff(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.diff(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.diff()</code> | Consecutive differences <code style="color:#002fa7;background-color:#eef2fb">s&#91;i&#93; - s&#91;i-1&#93;</code>. The first value is null. | (Optional) **string**: <code style="color:#002fa7;background-color:#eef2fb">null&#95;behavior="drop"</code> to drop the leading null | **Series**: the differences between adjacent elements, the same length as the original with a null first value | The Series keeps its length and starts with null. NumPy returns one fewer value, and its first value is the second element minus the first. On the temperature Series, `highs.diff()` is null, then 0.787, 0.198, and 0.579. <code style="color:#002fa7;background-color:#eef2fb">s.diff(null&#95;behavior="drop")</code> matches the shorter NumPy result. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.sort(arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.sort(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.sort()</code> | A new Series sorted ascending. This does not sort table rows. | (Optional) <code style="color:#002fa7;background-color:#eef2fb">descending=True</code> | **Series**: a sorted copy of the Series | Table order: <code style="color:#002fa7;background-color:#eef2fb">df.sort("col")</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">arr.item(i)</code> | <code style="color:#c7254e;background-color:#f9f2f4">arr.item(i)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.item(i)</code> | The element at index `i` (indexes start at 0). | **int**: the index of the element | **scalar**: the element at that index | Also <code style="color:#002fa7;background-color:#eef2fb">s&#91;i&#93;</code>. On a one-row frame: <code style="color:#002fa7;background-color:#eef2fb">df.get&#95;column("col").item(0)</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">arr.take(indices)</code> | <code style="color:#c7254e;background-color:#f9f2f4">arr.take(indices)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.gather(indices)</code> | Elements at the given indexes. `indices` is an array (<code style="color:#c7254e;background-color:#f9f2f4">make&#95;array</code>, <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code>) or a Series. | **array** or **Series** of ints: the indices of the elements | **Series**: the elements at those indices | Row selection on a frame is <code style="color:#002fa7;background-color:#eef2fb">df.gather(indices)</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.random.choice(options)</code> / <code style="color:#c7254e;background-color:#f9f2f4">np.random.choice(options, k)</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.random.choice(arr)</code> / <code style="color:#c7254e;background-color:#f9f2f4">np.random.choice(arr, k)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.sample(1).item()</code> / <code style="color:#002fa7;background-color:#eef2fb">s.sample(n=k, with&#95;replacement=True)</code> | One draw, or `k` draws with replacement. | 1. **int**: <code style="color:#002fa7;background-color:#eef2fb">n</code>, the number of draws 2. (Optional) <code style="color:#002fa7;background-color:#eef2fb">with&#95;replacement=True</code> to match Data 8 (Polars default is <code style="color:#002fa7;background-color:#eef2fb">False</code>) | **scalar** for one draw (with <code style="color:#002fa7;background-color:#eef2fb">.item()</code>), or **Series** of <code style="color:#002fa7;background-color:#eef2fb">n</code> draws | NumPy draws with replacement unless <code style="color:#c7254e;background-color:#f9f2f4">replace=False</code>. Polars <code style="color:#002fa7;background-color:#eef2fb">Series.sample</code> draws without replacement, so the Data 8 default passes <code style="color:#002fa7;background-color:#eef2fb">with&#95;replacement=True</code>. <code style="color:#c7254e;background-color:#f9f2f4">replace=False</code> is <code style="color:#002fa7;background-color:#eef2fb">with&#95;replacement=False</code>. There is no `p=` weights argument. |
| shuffle the elements | <code style="color:#c7254e;background-color:#f9f2f4">np.random.permutation(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.shuffle()</code> | Random permutation of the Series. Every value is kept once. | (Optional) **int**: <code style="color:#002fa7;background-color:#eef2fb">seed</code> | **Series**: the same values in random order | Also <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").shuffle()</code> inside <code style="color:#002fa7;background-color:#eef2fb">select</code> or <code style="color:#002fa7;background-color:#eef2fb">with&#95;columns</code>. <code style="color:#002fa7;background-color:#eef2fb">DataFrame</code> has no <code style="color:#002fa7;background-color:#eef2fb">.shuffle()</code>. Chapter 12 shuffles a label column this way. |
| <code style="color:#c7254e;background-color:#f9f2f4">percentile(p, arr)</code> | <code style="color:#c7254e;background-color:#f9f2f4">percentile(p, arr)</code> (<code style="color:#c7254e;background-color:#f9f2f4">from datascience import percentile</code>, unchanged) | <code style="color:#002fa7;background-color:#eef2fb">s.quantile(p / 100)</code> | Quantile of the Series. Polars uses 0–1, so divide a percentile by 100. | 1. **float**: the quantile, between 0 and 1 (<code style="color:#002fa7;background-color:#eef2fb">p / 100</code>) 2. (Optional) **string**: <code style="color:#002fa7;background-color:#eef2fb">interpolation</code> | **float**: the value at that quantile | Per group: <code style="color:#002fa7;background-color:#eef2fb">GroupBy.quantile(p / 100)</code>. |

---

## Converting between arrays and Series

A NumPy array and a Polars Series hold the same kind of data, so you can move between them. Convert only at the boundary: an array becomes a Series when it is added to a DataFrame, and a column becomes an array when the next step needs NumPy. Always **name** the Series when you build one.

### Array → Series

| From | Polars | Notes |
| --- | --- | --- |
| Values written out by hand | <code style="color:#002fa7;background-color:#eef2fb">pl.Series("name", <span style="color:#c7254e">make&#95;array(v1, v2, ...)</span>)</code> | The dtype carries over (<code style="color:#002fa7;background-color:#eef2fb">Int64</code>, <code style="color:#002fa7;background-color:#eef2fb">Float64</code>, `String`, `Boolean`). Do not write a list <code style="color:#002fa7;background-color:#eef2fb">pl.Series("name", &#91;...&#93;)</code>. |
| NumPy array (<code style="color:#c7254e;background-color:#f9f2f4">make&#95;array</code>, <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code>, ...) | <code style="color:#002fa7;background-color:#eef2fb">pl.Series("name", <span style="color:#c7254e">arr</span>)</code> | One-dimensional arrays only. The dtype carries over (`int64` becomes <code style="color:#002fa7;background-color:#eef2fb">Int64</code>). |
| NumPy array with <code style="color:#c7254e;background-color:#f9f2f4">np.nan</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.Series("name", <span style="color:#c7254e">arr</span>, nan&#95;to&#95;null=True)</code> | Turns `nan` into <code style="color:#002fa7;background-color:#eef2fb">null</code>. Without it, `nan` stays a float `NaN`. |
| `row` object (<code style="color:#c7254e;background-color:#f9f2f4">tbl.row(i)</code>) | <code style="color:#002fa7;background-color:#eef2fb">pl.Series("name", df.row(i))</code> | <code style="color:#002fa7;background-color:#eef2fb">df.row(i)</code> is a tuple of the row's values. |
| Table column | <code style="color:#002fa7;background-color:#eef2fb">df.get&#95;column("col")</code> | Already a Series. No conversion needed. |
| 2D NumPy array | <code style="color:#002fa7;background-color:#eef2fb">pl.from&#95;numpy(<span style="color:#c7254e">arr</span>, schema=&#91;"a", "b"&#93;)</code> | Gives a DataFrame, one column per array column. |

### Series → array

| To | Polars | Notes |
| --- | --- | --- |
| NumPy array | <code style="color:#002fa7;background-color:#eef2fb">s.to&#95;numpy()</code> | Returns a read-only <code style="color:#c7254e;background-color:#f9f2f4">np.ndarray</code>. For a copy you can modify, use <code style="color:#002fa7;background-color:#eef2fb">s.to&#95;numpy(writable=True)</code>. A Series with <code style="color:#002fa7;background-color:#eef2fb">null</code> becomes a float array with `nan`. |
| Python list | <code style="color:#002fa7;background-color:#eef2fb">s.to&#95;list()</code> | Use for plain Python code such as <code style="color:#002fa7;background-color:#eef2fb">"-".join(s.to&#95;list())</code>, `xticks=...`, or `bins=...`. |
| One element | <code style="color:#002fa7;background-color:#eef2fb">s.item(i)</code> or <code style="color:#002fa7;background-color:#eef2fb">s&#91;i&#93;</code> | A Python scalar. |
| Table column to array | <code style="color:#002fa7;background-color:#eef2fb">df.get&#95;column("col").to&#95;numpy()</code> | Or <code style="color:#002fa7;background-color:#eef2fb">df.to&#95;numpy()</code> for the whole frame as a 2D array. |

Follow the [array-or-Series rule](#arrays-standalone-array-or-dataframe-column): values that are never added to a DataFrame stay a NumPy array and use NumPy functions. A column taken from a DataFrame is already a Series, and Series arithmetic, `sum`, `mean`, `std`, `sqrt`, and `abs` work on it directly. Wrap an array in <code style="color:#002fa7;background-color:#eef2fb">pl.Series("name", <span style="color:#c7254e">arr</span>)</code> when it is added to a DataFrame.

### Example: converting between arrays and Series

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7"><span style="color:#c7254e">import numpy as np</span>
<span style="color:#c7254e">from datascience import make_array</span>
import polars as pl

<span style="color:#6a737d"># Array -&gt; Series</span>
x = pl.Series("x", <span style="color:#c7254e">make_array(1, 2, 3)</span>)           <span style="color:#6a737d"># values written out by hand</span>
y = pl.Series("y", <span style="color:#c7254e">np.arange(4.0, 7.0)</span>)           <span style="color:#6a737d"># from another NumPy array</span>
df = pl.DataFrame([pl.Series("a", <span style="color:#c7254e">make_array(1, 2)</span>), pl.Series("b", <span style="color:#c7254e">make_array(3, 4)</span>)])
r = pl.Series("row", df.row(0))                    <span style="color:#6a737d"># from a table row</span>
m = pl.Series("m", <span style="color:#c7254e">make_array(1.0, np.nan)</span>, nan_to_null=True)

print(y)
print(r)
print(m)

<span style="color:#6a737d"># Series -&gt; array</span>
<span style="color:#c7254e">arr</span> = x.to_numpy()
print(<span style="color:#c7254e">arr</span>, type(<span style="color:#c7254e">arr</span>).__name__, <span style="color:#c7254e">arr</span>.dtype)
print(x.to_list())

<span style="color:#6a737d"># Round trip: do NumPy work, then wrap the result in a named Series again</span>
doubled = pl.Series("x", <span style="color:#c7254e">arr</span> * 10)
print(doubled.to_list())

<span style="color:#6a737d"># A NumPy array can be edited only on a writable copy</span>
w = x.to_numpy(writable=True)
w[0] = 99
print(w)</code></pre>

Output:

```text
shape: (3,)
Series: 'y' [f64]
[
	4.0
	5.0
	6.0
]
shape: (2,)
Series: 'row' [i64]
[
	1
	3
]
shape: (2,)
Series: 'm' [f64]
[
	1.0
	null
]
[1 2 3] ndarray int64
[1, 2, 3]
[10, 20, 30]
[99  2  3]
```

---

## String Methods and Operations

On a DataFrame, call string methods on <code style="color:#002fa7;background-color:#eef2fb">pl.col("col")</code> and add the result with <code style="color:#002fa7;background-color:#eef2fb">.alias("name")</code>. On a Series you already hold, call <code style="color:#002fa7;background-color:#eef2fb">s.str...</code>.

| Data 8                                                              | Polars Equivalent                                | Description                                                                                                       | Polars input                                                                                                                                          | Polars output                                                                           | Notes                                                                                         |
| ------------------------------------------------------------------- | ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| <code style="color:#c7254e;background-color:#f9f2f4">s.upper()</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").str.to&#95;uppercase()</code> | Uppercase copy of each string. | None | **Expr** or **Series**: each string in uppercase | Use inside <code style="color:#002fa7;background-color:#eef2fb">with&#95;columns</code> / <code style="color:#002fa7;background-color:#eef2fb">select</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">s.lower()</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").str.to&#95;lowercase()</code> | Lowercase copy of each string. | None | **Expr** or **Series**: each string in lowercase | Column of strings. |
| <code style="color:#c7254e;background-color:#f9f2f4">s.replace(old, new)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").str.replace&#95;all(old, new)</code> | Replace every `old` with `new`. `str.replace` replaces the first match only. | 1. **string**: <code style="color:#002fa7;background-color:#eef2fb">old</code>, the text to find 2. **string**: <code style="color:#002fa7;background-color:#eef2fb">new</code>, the replacement 3. (Optional) <code style="color:#002fa7;background-color:#eef2fb">literal=True</code> to treat <code style="color:#002fa7;background-color:#eef2fb">old</code> as plain text, not a pattern | **Expr** or **Series**: each string with every <code style="color:#002fa7;background-color:#eef2fb">old</code> replaced | First match only: `str.replace(old, new)`. |
| <code style="color:#c7254e;background-color:#f9f2f4">s.split()</code> / <code style="color:#c7254e;background-color:#f9f2f4">s.split(separator)</code> / <code style="color:#c7254e;background-color:#f9f2f4">s.split(separator, maxsplit)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").str.split(separator)</code> | Split each string into a list. Omit `separator` to split on whitespace. `n` limits the number of splits. | 1. (Optional) **string**: <code style="color:#002fa7;background-color:#eef2fb">by</code>, the separator 2. (Optional) **int**: <code style="color:#002fa7;background-color:#eef2fb">n</code>, the maximum number of splits | **Expr** or **Series** of lists: the pieces of each string | Returns a list column. |
| <code style="color:#c7254e;background-color:#f9f2f4">s.join(iterable)</code> | <code style="color:#002fa7;background-color:#eef2fb">separator.join(s.to&#95;list())</code> or <code style="color:#002fa7;background-color:#eef2fb">pl.concat&#95;str</code> | Join elements into one string. A Python list uses `separator.join(iterable)`. A list column uses <code style="color:#002fa7;background-color:#eef2fb">pl.concat&#95;str</code>. | 1. **string**: the separator 2. **list**: the strings to join, <code style="color:#002fa7;background-color:#eef2fb">s.to&#95;list()</code> | **string**: the joined text (an **Expr** for <code style="color:#002fa7;background-color:#eef2fb">pl.concat&#95;str</code>) | <code style="color:#002fa7;background-color:#eef2fb">"-".join(s.to&#95;list())</code> for one Series. |
| <code style="color:#c7254e;background-color:#f9f2f4">s.strip(chars)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").str.strip&#95;chars(chars)</code> | Trim `chars` from both ends. Omit `chars` to trim whitespace. | (Optional) **string**: the characters to remove (default whitespace) | **Expr** or **Series**: each string with those characters trimmed from both ends | Default strips whitespace. |
| <code style="color:#c7254e;background-color:#f9f2f4">substr in s</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").str.contains(substr)</code> | `True` where the string contains `substr`. | 1. **string**: the substring 2. (Optional) <code style="color:#002fa7;background-color:#eef2fb">literal=True</code> to match plain text, not a pattern | **Expr** or **Series** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> where the string contains the substring | Use in <code style="color:#002fa7;background-color:#eef2fb">df.filter(...)</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">substr not in s</code> | <code style="color:#002fa7;background-color:#eef2fb">&#126;pl.col("col").str.contains(substr)</code> | `True` where the string does not contain `substr`. | 1. **string**: the substring 2. (Optional) <code style="color:#002fa7;background-color:#eef2fb">literal=True</code> to match plain text, not a pattern | **Expr** or **Series** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> where the string does not contain the substring | Negate with `~`. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.char.count(arr, substr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.str.count&#95;matches(substr, literal=True)</code> | Count of a literal substring in each element. | 1. **string**: the substring 2. <code style="color:#002fa7;background-color:#eef2fb">literal=True</code> | **Series** of ints: the number of times the substring appears in each element | Without `literal=True` the pattern is a regular expression. <code style="color:#002fa7;background-color:#eef2fb">.</code> then matches every character. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.char.find(arr, substr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.str.find(substr, literal=True)</code> | First index of `substr` in each element. | 1. **string**: the substring 2. <code style="color:#002fa7;background-color:#eef2fb">literal=True</code> | **Series** of ints: the first index of the substring in each element (null if missing) | A missing search is null. NumPy returns -1. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.char.rfind(arr, substr)</code> | — | Last index of `substr` in each element. | — | — | Polars has no Series method for the last position. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.char.isalpha(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.str.contains(r"^&#92;p{Alphabetic}+&#36;")</code> | True when every character is a letter. | **string**: a regular expression pattern | **Series** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> where every character is a letter |  |
| <code style="color:#c7254e;background-color:#f9f2f4">np.char.isnumeric(arr)</code> | <code style="color:#002fa7;background-color:#eef2fb">s.str.contains(r"^&#92;d+&#36;")</code> | True when every character is a decimal digit. | **string**: a regular expression pattern | **Series** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> where every character is a digit | Matches decimal digits, not every Unicode numeric character. |

---

## Naming Columns with `.alias()`

Data 8 names a new column in the call: <code style="color:#c7254e;background-color:#f9f2f4">tbl.with&#95;column("pair", values)</code>. Polars writes the expression first, then names it with <code style="color:#002fa7;background-color:#eef2fb">.alias("pair")</code>.

<code style="color:#002fa7;background-color:#eef2fb">.alias("name")</code> works on:

- a **Polars Series**, such as <code style="color:#002fa7;background-color:#eef2fb">pl.Series("tmp", <span style="color:#c7254e">make&#95;array(1, 2, 3)</span>)</code>
- a **lazy expression**, such as <code style="color:#002fa7;background-color:#eef2fb">pl.col("a")</code>, <code style="color:#002fa7;background-color:#eef2fb">pl.col("a") &#42; 2</code>, <code style="color:#002fa7;background-color:#eef2fb">pl.concat&#95;list(&#91;pl.col("a"), pl.col("b")&#93;)</code>, or <code style="color:#002fa7;background-color:#eef2fb">pl.when(...)</code>

It only changes the **column name**. The values stay the same, and the original column is unchanged.

| Polars | Description | Input | Output |
| --- | --- | --- | --- |
| <code style="color:#002fa7;background-color:#eef2fb">s.alias("name")</code> | Names a Polars Series `"name"`. | **string**: the new name | **Series**: the same values with the new name |
| <code style="color:#002fa7;background-color:#eef2fb">expr.alias("name")</code> | Names a lazy expression `"name"`. This is the form used for <code style="color:#002fa7;background-color:#eef2fb">pl.col</code>. | **string**: the new name | **Expr**: the same expression with the new name |
| <code style="color:#002fa7;background-color:#eef2fb">df.with&#95;columns(name=expr)</code> | The keyword is the new column name. Same result as <code style="color:#002fa7;background-color:#eef2fb">.alias("name")</code>. | keyword argument <code style="color:#002fa7;background-color:#eef2fb">name=expr</code>: the new column name and its **Series** or **Expr** | **DataFrame**: a copy of the original DataFrame with the named column added |

**When you would use it**

- **Computed column.** The conversion notebook adds one with <code style="color:#002fa7;background-color:#eef2fb">(pl.col("foo") + pl.col("bar")).alias("baz")</code>.
- **Series of ready-made values.** <code style="color:#002fa7;background-color:#eef2fb">pl.Series("Blocks from campus", <span style="color:#c7254e">np.arange(n)</span>)</code> is already named. Call <code style="color:#002fa7;background-color:#eef2fb">.alias</code> only when that name should change.
- **A new label after a group reduction.** GroupBy methods keep the original column names. Rename with <code style="color:#002fa7;background-color:#eef2fb">.rename({"score": "total&#95;score"})</code>.

<code style="color:#002fa7;background-color:#eef2fb">.alias</code> is for a column you are creating. <code style="color:#002fa7;background-color:#eef2fb">df.rename({"old": "new"})</code> changes a label that already exists.

<code style="color:#002fa7;background-color:#eef2fb">df.with&#95;columns(pl.col("a") &#42; 2)</code> overwrites `a`. <code style="color:#002fa7;background-color:#eef2fb">df.with&#95;columns((pl.col("a") &#42; 2).alias("a&#95;times&#95;2"))</code> keeps `a` and adds <code style="color:#002fa7;background-color:#eef2fb">a&#95;times&#95;2</code>.

See [example](#example-alias).

---

## Tables and Table Methods

Build a frame from named Series. In Polars 2.0 the Series length is the frame height, so pass them to the constructor:

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7"><span style="color:#c7254e">import numpy as np</span>
import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

df = pl.DataFrame([
    pl.Series("foo", <span style="color:#c7254e">make_array(1, 2)</span>),
    pl.Series("bar", <span style="color:#c7254e">make_array(3, 4)</span>),
])

<span style="color:#6a737d"># Computed column: lazy pl.col expressions, then .alias</span>
df = df.with_columns(
    (pl.col("foo") + pl.col("bar")).alias("baz")
)

<span style="color:#6a737d"># A column of already-computed values. Length must match the frame.</span>
df = df.with_columns(
    pl.Series("idx", <span style="color:#c7254e">np.arange</span>(df.height))
)

<span style="color:#6a737d"># The same index as a lazy expression (no Python row count)</span>
df = df.with_columns(
    pl.int_range(0, pl.len()).alias("idx")
)</code></pre>

<code style="color:#002fa7;background-color:#eef2fb">with&#95;columns</code> adds columns to a frame that already has rows. See [example](#example-adding-columns-with-plcol).

| Data 8                                                         | Polars Equivalent                                                                                   | Description                                                                                                                                    | Polars input                                                                                                                                                                                                                                                                                                                                                       | Polars output                                                                                     | Notes                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | Example                                           |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| <code style="color:#c7254e;background-color:#f9f2f4">Table()</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.DataFrame()</code> | Empty DataFrame. | None | **DataFrame**: an empty DataFrame | [API](https://docs.pola.rs/api/python/stable/reference/dataframe/). To start from columns, pass Series: <code style="color:#002fa7;background-color:#eef2fb">pl.DataFrame(&#91;pl.Series("a", <span style="color:#c7254e">make&#95;array(1)</span>), ...&#93;)</code>. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">Table().read&#95;table(filename)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.read&#95;csv(filename)</code> or <code style="color:#002fa7;background-color:#eef2fb">pl.read&#95;parquet(filename)</code> | DataFrame from a file. Pick the reader that matches the file type. | **string**: the name of the file | **DataFrame** with the contents of the data file | [I/O reference](https://docs.pola.rs/api/python/stable/reference/io.html). <code style="color:#002fa7;background-color:#eef2fb">pl.scan&#95;csv</code> returns a LazyFrame; the same <code style="color:#002fa7;background-color:#eef2fb">pl.col</code> expressions apply. Every CSV in the repo (textbook, lectures, demo, and <code style="color:#002fa7;background-color:#eef2fb">data8-polars-instructions/data</code>) uses standard line endings, so do not pass `eol_char`. (A file that ends rows with a carriage return `\r` only would need `eol_char='\r'`; none in the textbook do.) |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.with&#95;column(name, values)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.with&#95;columns(pl.Series(name, values))</code> or <code style="color:#002fa7;background-color:#eef2fb">df.with&#95;columns(expr.alias(name))</code> | A DataFrame with an additional or replaced column. | **Series** or **Expr**: the new column, named with <code style="color:#002fa7;background-color:#eef2fb">pl.Series(name, values)</code> or <code style="color:#002fa7;background-color:#eef2fb">expr.alias(name)</code> | **DataFrame**: a copy of the original DataFrame with the new column added | Computed columns use <code style="color:#002fa7;background-color:#eef2fb">pl.col</code>. A constant uses <code style="color:#002fa7;background-color:#eef2fb">pl.lit(x).alias(name)</code>. | [example](#example-adding-columns-with-plcol) |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.with&#95;columns(n1, v1, n2, v2, ...)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.DataFrame(&#91;pl.Series(n1, v1), pl.Series(n2, v2)&#93;)</code> or several expressions in one <code style="color:#002fa7;background-color:#eef2fb">with&#95;columns</code> | Several columns in one call. | **Series** or **Expr**, one per column: each new column, with its name | **DataFrame**: a new DataFrame, or a copy of the original with the new columns added | Name every Series. | [example](#example-adding-columns-with-plcol) |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.column(col)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.get&#95;column("col")</code> | The column as a Polars Series. | **string**: the column name | **Series**: the values in that column | Inside <code style="color:#002fa7;background-color:#eef2fb">filter</code> / <code style="color:#002fa7;background-color:#eef2fb">with&#95;columns</code> / <code style="color:#002fa7;background-color:#eef2fb">group&#95;by</code>, write <code style="color:#002fa7;background-color:#eef2fb">pl.col("col")</code> and keep it lazy. One element: <code style="color:#002fa7;background-color:#eef2fb">.item(0)</code>. | [example](#example-get_column) |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.column(index)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.to&#95;series(index)</code> | The column at that position as a Series. Positions start at 0. | **int**: the column index | **Series**: the values in that column | `to_series(1)` is the second column. By name, use <code style="color:#002fa7;background-color:#eef2fb">get&#95;column</code>. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.num&#95;rows</code> | <code style="color:#002fa7;background-color:#eef2fb">df.height</code> or <code style="color:#002fa7;background-color:#eef2fb">len(df)</code> | Number of rows. | None | **int**: the number of rows in the DataFrame | Inside an expression: <code style="color:#002fa7;background-color:#eef2fb">pl.len()</code>. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.num&#95;columns</code> | <code style="color:#002fa7;background-color:#eef2fb">df.width</code> or <code style="color:#002fa7;background-color:#eef2fb">len(df.columns)</code> | Number of columns. | None | **int**: the number of columns in the DataFrame | Also <code style="color:#002fa7;background-color:#eef2fb">df.shape&#91;1&#93;</code>. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.labels</code> | <code style="color:#002fa7;background-color:#eef2fb">df.columns</code> | Column names. | None | **list** of strings: the names of each column | [API:](https://docs.pola.rs/api/python/stable/reference/dataframe/) <code style="color:#002fa7;background-color:#eef2fb">DataFrame.columns</code> |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.select(col1, col2, ...)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.select("col1", "col2", ...)</code> | Copy with only some columns. Arguments may be names or <code style="color:#002fa7;background-color:#eef2fb">pl.col</code> expressions. | **string** or **Expr**: column name(s) or <code style="color:#002fa7;background-color:#eef2fb">pl.col</code> expression(s) | **DataFrame** with the selected columns | Select numeric columns before <code style="color:#002fa7;background-color:#eef2fb">GroupBy.sum()</code> when other columns are strings. | [example](#example-select) |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.drop(col1, col2, ...)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.drop("col1", "col2", ...)</code> | Copy without those columns. | **string**: column name(s) | **DataFrame** without the selected columns | Original frame stays unless you reassign. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.drop(...)</code> returning the column | <code style="color:#002fa7;background-color:#eef2fb">df.drop&#95;in&#95;place("col")</code> | Removes the column in place and returns it as a Series. | **string**: the column name | **Series**: the removed column (<code style="color:#002fa7;background-color:#eef2fb">df</code> no longer has it) | Mutates <code style="color:#002fa7;background-color:#eef2fb">df</code>. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.relabeled(old&#95;label, new&#95;label)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.rename({old&#95;label: new&#95;label})</code> | New DataFrame with one or more columns renamed. | **dict**: <code style="color:#002fa7;background-color:#eef2fb">{old&#95;label: new&#95;label}</code>, one entry per column to rename | **DataFrame**: a new DataFrame; the original is unchanged | Original is unchanged. After a group reduction, <code style="color:#002fa7;background-color:#eef2fb">.rename</code> is how you label `total_score`. | [example](#example-dfrename) |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.relabeled(column&#95;index, new&#95;label)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.rename({df.columns&#91;column&#95;index&#93;: new&#95;label})</code> | Rename the column at that position. Positions start at 0. | **dict**: <code style="color:#002fa7;background-color:#eef2fb">{df.columns&#91;column&#95;index&#93;: new&#95;label}</code> | **DataFrame**: a new DataFrame; the original is unchanged | `relabeled(3, "SALARY")` renames column 3. In `nba_salaries.csv` that column is `"'15-'16 SALARY"`, so `rename({"'15-'16 SALARY": "SALARY"})` is the same rename. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.show(n)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.show(n)</code> | Prints the first `n` rows. Data 8 `show(n)` displays those rows. The textbook notebooks use <code style="color:#002fa7;background-color:#eef2fb">.show(n)</code> for every cell that only displays rows. | (Optional) **int**: number of rows you want to display (default 5) | None: displays the first <code style="color:#002fa7;background-color:#eef2fb">n</code> rows | Data 8 `show()` with no argument displays the first 10 rows. A Polars display shows the first five and the last five. <code style="color:#002fa7;background-color:#eef2fb">show</code> prints and returns `None`, so nothing can be chained after it. Cells like `nba_salaries.sort("PLAYER").show(5)` end the chain with <code style="color:#002fa7;background-color:#eef2fb">show</code>. |  |
| rows that feed a later step | <code style="color:#002fa7;background-color:#eef2fb">df.head(n)</code> | First `n` rows as a DataFrame. Use it only when the result is used again, because it is not a display call. | **int**: the number of rows | **DataFrame**: the first <code style="color:#002fa7;background-color:#eef2fb">n</code> rows | Notebook examples: <code style="color:#002fa7;background-color:#eef2fb">histogram&#95;elements.head(num&#95;bins).with&#95;columns(...)</code> and <code style="color:#002fa7;background-color:#eef2fb">movies&#95;and&#95;years.group&#95;by('Year').len(name='count').sort('Year').head(10).with&#95;columns(...).hvplot.barh(...)</code>. Both continue the chain, so <code style="color:#002fa7;background-color:#eef2fb">.show</code> would break them. Also <code style="color:#002fa7;background-color:#eef2fb">df.limit(n)</code>. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.show()</code> | <code style="color:#002fa7;background-color:#eef2fb">df.show(None)</code> | Every row. `None` prints all rows. | <code style="color:#002fa7;background-color:#eef2fb">None</code>: display every row | None: displays every row | `show()` with no argument prints the first five rows. A bare DataFrame still shows the first five and the last five. |  |
| long strings in a display | <code style="color:#002fa7;background-color:#eef2fb">with pl.Config(fmt&#95;str&#95;lengths=160): display(df)</code> | How many characters of each string to show. | **int**: <code style="color:#002fa7;background-color:#eef2fb">fmt&#95;str&#95;lengths</code>, the number of characters to show in each string | None: displays the DataFrame with longer strings | The default stops at about 30 characters. Chapter text uses 160 so the opening sentence is visible. |  |
| last rows | <code style="color:#002fa7;background-color:#eef2fb">df.tail(n)</code> | Last `n` rows. | **int**: the number of rows | **DataFrame**: the last <code style="color:#002fa7;background-color:#eef2fb">n</code> rows | Data 8 has no <code style="color:#002fa7;background-color:#eef2fb">show</code> for the last rows, and the chapter 1–12 notebooks never call <code style="color:#002fa7;background-color:#eef2fb">.tail</code>. To display the last rows, a bare <code style="color:#002fa7;background-color:#eef2fb">df.tail(n)</code> as the last line of a cell works. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.sort(column&#95;name)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.sort("column&#95;name")</code> | Copy sorted by a column, ascending. | 1. **string** or **Expr**: the column name 2. (Optional) <code style="color:#002fa7;background-color:#eef2fb">descending=True</code> | **DataFrame**: a copy of the original DataFrame sorted by that column | Sort key can be <code style="color:#002fa7;background-color:#eef2fb">pl.col("column&#95;name")</code>. `by` may be positional or named: <code style="color:#002fa7;background-color:#eef2fb">df.sort("SALARY")</code> and <code style="color:#002fa7;background-color:#eef2fb">df.sort(by="SALARY")</code> match. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.sort(column&#95;index)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.sort(pl.nth(column&#95;index))</code> | Copy sorted by the column at that position, ascending. Positions start at 0. | **Expr**: <code style="color:#002fa7;background-color:#eef2fb">pl.nth(column&#95;index)</code>, the column at that position | **DataFrame**: a copy of the original DataFrame sorted by that column | <code style="color:#002fa7;background-color:#eef2fb">nba&#95;salaries.sort(pl.nth(0)).show(5)</code> sorts by the first column and shows five rows. If column 0 is `"PLAYER"`, this matches <code style="color:#002fa7;background-color:#eef2fb">df.sort("PLAYER").show(5)</code>. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.sort(column&#95;name, descending=True)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.sort("column&#95;name", descending=True)</code> | Copy sorted descending. | 1. **string**: the column name 2. <code style="color:#002fa7;background-color:#eef2fb">descending=True</code>, passed by name | **DataFrame**: a copy of the original DataFrame sorted by that column, largest first | Pass <code style="color:#002fa7;background-color:#eef2fb">descending</code> by name. A second positional argument is another column to sort by, not <code style="color:#002fa7;background-color:#eef2fb">descending</code>: <code style="color:#002fa7;background-color:#eef2fb">df.sort("SALARY", True)</code> does not sort descending. <code style="color:#002fa7;background-color:#eef2fb">df.sort(by="SALARY", descending=True)</code> is the same call as <code style="color:#002fa7;background-color:#eef2fb">df.sort("SALARY", descending=True)</code>. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.where(column, predicate)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.filter(pl.col("column") &lt;op&gt; value)</code> | Copy with only the rows that match. Combine comparisons with `&` (and) or a vertical bar (or), and wrap each comparison in parentheses. | **Expr**: a boolean <code style="color:#002fa7;background-color:#eef2fb">pl.col</code> comparison. See the Table.where predicates table. | **DataFrame**: a copy of the original DataFrame with only the rows that match | The notebook writes <code style="color:#002fa7;background-color:#eef2fb">df.filter(pl.col("Price") &gt; 5)</code>. | [example](#example-filter-with-plcol) |
| rows that fail the predicate | <code style="color:#002fa7;background-color:#eef2fb">df.filter(&#126;(pl.col("column") &lt;op&gt; value))</code> | Copy with the matching rows removed. | **Expr**: the comparison, negated with <code style="color:#002fa7;background-color:#eef2fb">&#126;</code> | **DataFrame**: a copy of the original DataFrame without the rows that match | See [Table.where Predicates](#tablewhere-predicates). |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.take(np.arange(k))</code> / <code style="color:#c7254e;background-color:#f9f2f4">tbl.take(0)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.head(k)</code> / <code style="color:#002fa7;background-color:#eef2fb">df.head(1)</code> | <code style="color:#c7254e;background-color:#f9f2f4">take</code> of the first rows is <code style="color:#002fa7;background-color:#eef2fb">head</code>. | **int**: the number of rows | **DataFrame**: the first <code style="color:#002fa7;background-color:#eef2fb">k</code> rows | The index list <code style="color:#c7254e;background-color:#f9f2f4">np.arange(k)</code> is the first `k` rows. If the cell only displays them, it ends with <code style="color:#002fa7;background-color:#eef2fb">.show(k)</code> instead. See [Table.take: head, tail, or gather](#tabletake-head-tail-or-gather). |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.take(np.arange(num&#95;rows - k, num&#95;rows))</code> | <code style="color:#002fa7;background-color:#eef2fb">df.tail(k)</code> | <code style="color:#c7254e;background-color:#f9f2f4">take</code> of the last rows is <code style="color:#002fa7;background-color:#eef2fb">tail</code>. | **int**: the number of rows | **DataFrame**: the last <code style="color:#002fa7;background-color:#eef2fb">k</code> rows | The index list ends at the last row. See [Table.take: head, tail, or gather](#tabletake-head-tail-or-gather). |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.take(row&#95;indices)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.gather(row&#95;indices)</code> | Rows at the given indexes. Use <code style="color:#002fa7;background-color:#eef2fb">gather</code> only when the rows are neither the first few nor the last few. | **array** or **Series** of ints: the indices of the rows to include | **DataFrame**: a copy of the original DataFrame with only the rows at the given indices | <code style="color:#c7254e;background-color:#f9f2f4">gather(make&#95;array(3, 18, 100))</code> and <code style="color:#c7254e;background-color:#f9f2f4">gather(np.arange(3, 6))</code> stay <code style="color:#002fa7;background-color:#eef2fb">gather</code>. Keep <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code> for the index list. See [Table.take: head, tail, or gather](#tabletake-head-tail-or-gather). |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.sample(k)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.sample(n=k, with&#95;replacement=True)</code> | `k` rows, drawn with replacement. | 1. **int**: <code style="color:#002fa7;background-color:#eef2fb">n</code>, the sample size 2. <code style="color:#002fa7;background-color:#eef2fb">with&#95;replacement=True</code> (Polars default is <code style="color:#002fa7;background-color:#eef2fb">False</code>) | **DataFrame**: a new DataFrame with <code style="color:#002fa7;background-color:#eef2fb">n</code> rows | Data 8 <code style="color:#c7254e;background-color:#f9f2f4">Table.sample</code> draws with replacement unless <code style="color:#002fa7;background-color:#eef2fb">with&#95;replacement=False</code> is passed. Polars <code style="color:#002fa7;background-color:#eef2fb">DataFrame.sample</code> draws without replacement, and with no `n` it draws one row. Pass <code style="color:#002fa7;background-color:#eef2fb">with&#95;replacement=True</code> to match the Data 8 default. Polars `sample` has no weights. <code style="color:#c7254e;background-color:#f9f2f4">sample&#95;proportions</code> stays the unchanged <code style="color:#c7254e;background-color:#f9f2f4">datascience</code> function (<code style="color:#c7254e;background-color:#f9f2f4">from datascience import make&#95;array, sample&#95;proportions</code>). It returns a NumPy array, so call <code style="color:#002fa7;background-color:#eef2fb">.item(i)</code> on it directly; wrap it in <code style="color:#002fa7;background-color:#eef2fb">pl.Series("name", ...)</code> only when it is added to a DataFrame (Chapter 11.2 `Random Sample`). |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.sample(k, with&#95;replacement=False)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.sample(n=k, with&#95;replacement=False)</code> | `k` distinct rows. | 1. **int**: <code style="color:#002fa7;background-color:#eef2fb">n</code>, the sample size 2. <code style="color:#002fa7;background-color:#eef2fb">with&#95;replacement=False</code> | **DataFrame**: a new DataFrame with <code style="color:#002fa7;background-color:#eef2fb">n</code> distinct rows | Both sides draw without replacement. Polars still needs `n`: omitting it draws one row. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.sample()</code> | <code style="color:#002fa7;background-color:#eef2fb">df.sample(n=df.height, with&#95;replacement=True)</code> | A bootstrap resample: as many rows as the table, drawn with replacement. | 1. **int**: <code style="color:#002fa7;background-color:#eef2fb">n=df.height</code> 2. <code style="color:#002fa7;background-color:#eef2fb">with&#95;replacement=True</code> | **DataFrame**: a bootstrap resample with as many rows as the original | Data 8 <code style="color:#c7254e;background-color:#f9f2f4">Table.sample(k=None, with&#95;replacement=True)</code>: with no arguments it draws `num_rows` rows *with* replacement, so it is not a shuffle. Polars with no `n` returns one row and draws without replacement, so both arguments are needed. Chapters 13 and 16 resample this way. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.sample(with&#95;replacement=False)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.sample(n=df.height, with&#95;replacement=False, shuffle=True)</code> | A shuffle of every row. | 1. **int**: <code style="color:#002fa7;background-color:#eef2fb">n=df.height</code> 2. <code style="color:#002fa7;background-color:#eef2fb">with&#95;replacement=False</code> 3. <code style="color:#002fa7;background-color:#eef2fb">shuffle=True</code> | **DataFrame**: every row of the original, in random order | Data 8 with <code style="color:#002fa7;background-color:#eef2fb">with&#95;replacement=False</code> and no `k` permutes the whole table. Polars with no `n` returns one row, so the call passes <code style="color:#002fa7;background-color:#eef2fb">n=df.height</code>. <code style="color:#002fa7;background-color:#eef2fb">DataFrame</code> has no <code style="color:#002fa7;background-color:#eef2fb">.shuffle()</code>. To shuffle one label column, use the next row instead. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.sample(with&#95;replacement=False).column(col)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").shuffle()</code> | Random permutation of one column. Every value is kept once. | (Optional) **int**: <code style="color:#002fa7;background-color:#eef2fb">seed</code> | **Expr** or **Series**: the column's values in random order | Chapter 12 label shuffles: <code style="color:#002fa7;background-color:#eef2fb">select(pl.col("Maternal Smoker").shuffle()).to&#95;series(0)</code>, or <code style="color:#002fa7;background-color:#eef2fb">with&#95;columns(pl.col("Group").shuffle().alias("Shuffled Label"))</code>. <code style="color:#002fa7;background-color:#eef2fb">s.shuffle()</code> is the Series form. <code style="color:#002fa7;background-color:#eef2fb">DataFrame</code> has no <code style="color:#002fa7;background-color:#eef2fb">.shuffle()</code>. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.bin(column)</code> / <code style="color:#c7254e;background-color:#f9f2f4">tbl.bin(column, bins)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.get&#95;column(col).hist(bin&#95;count=10)</code> or <code style="color:#002fa7;background-color:#eef2fb">hist(bins=edges)</code> | Counts of values in each interval. Default is 10 equal-width bins from min to max. | (Optional) **int**: <code style="color:#002fa7;background-color:#eef2fb">bin&#95;count</code>, the number of equal-width bins (default 10), OR **list** of ints/floats: <code style="color:#002fa7;background-color:#eef2fb">bins</code>, the bin boundaries | **DataFrame**: one row per bin, with <code style="color:#002fa7;background-color:#eef2fb">breakpoint</code>, <code style="color:#002fa7;background-color:#eef2fb">category</code>, and <code style="color:#002fa7;background-color:#eef2fb">count</code> columns | To label bins and then count them: <code style="color:#002fa7;background-color:#eef2fb">bin&#95;intervals</code> plus <code style="color:#002fa7;background-color:#eef2fb">GroupBy.len()</code>. <code style="color:#002fa7;background-color:#eef2fb">Series.cut</code> is deprecated. | [example](#example-histogram-and-bins) |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.apply(function)</code> / <code style="color:#c7254e;background-color:#f9f2f4">tbl.apply(function, col1, col2, ...)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.with&#95;columns(pl.col("col").map&#95;elements(function, return&#95;dtype=...).alias("Name"))</code> | Applies a Python function to each element. Slower than a native expression. | 1. **function**: function to apply to each element 2. <code style="color:#002fa7;background-color:#eef2fb">return&#95;dtype</code>: <code style="color:#002fa7;background-color:#eef2fb">int</code> or <code style="color:#002fa7;background-color:#eef2fb">float</code>, the type the function returns | **Expr**: a column with one value per element of the original column, after applying the function | Prefer <code style="color:#002fa7;background-color:#eef2fb">pl.col("a") &#42; 2</code> when a native expression exists. For <code style="color:#002fa7;background-color:#eef2fb">return&#95;dtype</code>, use Python `int` or `float` (they become Polars <code style="color:#002fa7;background-color:#eef2fb">Int64</code> / <code style="color:#002fa7;background-color:#eef2fb">Float64</code>). Do not write <code style="color:#002fa7;background-color:#eef2fb">pl.Int64</code> / <code style="color:#002fa7;background-color:#eef2fb">pl.Float64</code> here. Several columns: <code style="color:#002fa7;background-color:#eef2fb">pl.struct(&#91;"a", "b"&#93;).map&#95;elements(...)</code>. | [example](#example-map_elements) |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.group(column&#95;or&#95;columns)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.group&#95;by(pl.col("col")).len().sort("col")</code> | One row per distinct value. The default Data 8 summary is a count. | **string** or **Expr**, or a **list** of them: column(s) on which to group | **DataFrame**: one row per group, with the count in a <code style="color:#002fa7;background-color:#eef2fb">len</code> column | `len(name="count")` names the count column. Polars returns groups in no fixed order; Data 8 sorts them by the grouped column, so end with <code style="color:#002fa7;background-color:#eef2fb">.sort("col")</code>. Do not use <code style="color:#002fa7;background-color:#eef2fb">maintain&#95;order=True</code>. | [example](#example-group_by-methods) |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.group(column&#95;or&#95;columns, function)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.group&#95;by(pl.col("col")).max().sort("col")</code> and the other GroupBy methods | Reduce the other columns with a [GroupBy method](#groupby-methods). | 1. **string** or **Expr**, or a **list** of them: column(s) on which to group 2. GroupBy method: how to aggregate the other columns, e.g. <code style="color:#002fa7;background-color:#eef2fb">.sum()</code>, <code style="color:#002fa7;background-color:#eef2fb">.mean()</code>, <code style="color:#002fa7;background-color:#eef2fb">.max()</code>. See [GroupBy methods](#groupby-methods). | **DataFrame**: one row per group, with the other columns aggregated | Select columns before <code style="color:#002fa7;background-color:#eef2fb">.sum()</code> or <code style="color:#002fa7;background-color:#eef2fb">.mean()</code> when a string column is present. <code style="color:#002fa7;background-color:#eef2fb">.max()</code> can run on strings. | [example](#example-group_by-methods) |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.pivot(col1, col2)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.pivot(col1, index=col2, aggregate&#95;function="len")</code> | Pivot: each value of `col1` becomes a column, each value of `col2` becomes a row. Cells are counts. | 1. **string**: <code style="color:#002fa7;background-color:#eef2fb">on</code>, column whose unique values make up the columns 2. **string**: <code style="color:#002fa7;background-color:#eef2fb">index</code>, column whose unique values make up the rows 3. **string**: <code style="color:#002fa7;background-color:#eef2fb">aggregate&#95;function="len"</code> to count | **DataFrame**: a pivot table of counts | The first argument is `on` (formerly `columns`). | [example](#example-group_by-methods) |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.pivot(col1, col2)</code> on a table with only those two columns | <code style="color:#002fa7;background-color:#eef2fb">df.pivot(col1, index=col2, values=col1, aggregate&#95;function="len")</code> | Same count pivot. | 1. **string**: <code style="color:#002fa7;background-color:#eef2fb">on</code>, column whose unique values make up the columns 2. **string**: <code style="color:#002fa7;background-color:#eef2fb">index</code>, column whose unique values make up the rows 3. **string**: <code style="color:#002fa7;background-color:#eef2fb">values=col1</code> 4. **string**: <code style="color:#002fa7;background-color:#eef2fb">aggregate&#95;function="len"</code> to count | **DataFrame**: a pivot table of counts | Polars needs a values column; with only two columns none is left, and the plain form raises <code style="color:#002fa7;background-color:#eef2fb">InvalidOperationError</code>. Chapter 18. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.pivot(col1, col2, values, collect)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.pivot(col1, index=col2, values=values, aggregate&#95;function=collect)</code> | Same pivot, aggregating a third column. | 1. **string**: <code style="color:#002fa7;background-color:#eef2fb">on</code>, column whose unique values make up the columns 2. **string**: <code style="color:#002fa7;background-color:#eef2fb">index</code>, column whose unique values make up the rows 3. **string**: <code style="color:#002fa7;background-color:#eef2fb">values</code>, column that describes the values of the cells 4. **string**: <code style="color:#002fa7;background-color:#eef2fb">aggregate&#95;function</code>, how the values are collected: <code style="color:#002fa7;background-color:#eef2fb">"min"</code>, <code style="color:#002fa7;background-color:#eef2fb">"max"</code>, <code style="color:#002fa7;background-color:#eef2fb">"first"</code>, <code style="color:#002fa7;background-color:#eef2fb">"last"</code>, <code style="color:#002fa7;background-color:#eef2fb">"sum"</code>, <code style="color:#002fa7;background-color:#eef2fb">"mean"</code>, <code style="color:#002fa7;background-color:#eef2fb">"median"</code>, or <code style="color:#002fa7;background-color:#eef2fb">"len"</code> | **DataFrame**: a pivot table | Notebook: <code style="color:#002fa7;background-color:#eef2fb">df.pivot("Flavor", index="Color", values="Price", aggregate&#95;function="max")</code>. | [example](#example-group_by-methods) |
| <code style="color:#c7254e;background-color:#f9f2f4">tblA.join(colA, tblB)</code> / <code style="color:#c7254e;background-color:#f9f2f4">tblA.join(colA, tblB, colB)</code> | <code style="color:#002fa7;background-color:#eef2fb">dfA.join(dfB, on="colA")</code> or <code style="color:#002fa7;background-color:#eef2fb">dfA.join(dfB, left&#95;on="colA", right&#95;on="colB")</code> | Rows whose key appears in both frames, with columns from both. | 1. **DataFrame**: the other DataFrame 2. **string**: <code style="color:#002fa7;background-color:#eef2fb">on</code>, the shared column name; or <code style="color:#002fa7;background-color:#eef2fb">left&#95;on</code> and <code style="color:#002fa7;background-color:#eef2fb">right&#95;on</code> when the names differ 3. (Optional) **string**: <code style="color:#002fa7;background-color:#eef2fb">how</code>: <code style="color:#002fa7;background-color:#eef2fb">"inner"</code> (default), <code style="color:#002fa7;background-color:#eef2fb">"left"</code>, <code style="color:#002fa7;background-color:#eef2fb">"right"</code>, <code style="color:#002fa7;background-color:#eef2fb">"full"</code>, <code style="color:#002fa7;background-color:#eef2fb">"semi"</code>, <code style="color:#002fa7;background-color:#eef2fb">"anti"</code>, or <code style="color:#002fa7;background-color:#eef2fb">"cross"</code> | **DataFrame**: a new DataFrame with the columns of both | Default `how` is inner. |  |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.with&#95;row(values)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.concat(&#91;df, pl.DataFrame(&#91;dict(zip(df.columns, values))&#93;)&#93;)</code> or <code style="color:#002fa7;background-color:#eef2fb">df.extend(other)</code> | Adds one row. Match column names and types. | **DataFrame**: a one-row DataFrame with the same column names and types | **DataFrame**: a new DataFrame with the row added (<code style="color:#002fa7;background-color:#eef2fb">extend</code> changes <code style="color:#002fa7;background-color:#eef2fb">df</code> in place) | `extend` modifies in place. `concat` returns a new frame. | [example](#example-dataframeextend-adding-rows) |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.with&#95;rows(list&#95;of&#95;rows)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.extend(pl.from&#95;dicts(list&#95;of&#95;rows))</code> or <code style="color:#002fa7;background-color:#eef2fb">pl.concat(&#91;df, other&#93;)</code> | Adds many rows. | **DataFrame**: the rows to add, with the same schema | **DataFrame** with the rows added | <code style="color:#002fa7;background-color:#eef2fb">pl.from&#95;dicts</code> builds the extra rows from a list of dicts. | [example](#example-dataframeextend-adding-rows) |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.row(row&#95;index)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.row(row&#95;index, named=True)</code> | One row as a dict, or a tuple when `named=False`. | 1. **int**: row index 2. (Optional) <code style="color:#002fa7;background-color:#eef2fb">named=True</code> for a dict | **dict** of column name to value, or **tuple** of the row's values | A single cell from a filtered frame: <code style="color:#002fa7;background-color:#eef2fb">df.filter(...).get&#95;column("col").item(0)</code>. | [example](#example-one-value-from-a-filtered-row) |

### Table.take: head, tail, or gather

<code style="color:#c7254e;background-color:#f9f2f4">tbl.take</code> has three Polars forms. Pick one by looking at which rows the index list names.

| The index list names | Use | Data 8 example | Polars example |
| --- | --- | --- | --- |
| the first rows, starting at 0 with no gaps | <code style="color:#002fa7;background-color:#eef2fb">df.head(k)</code> | <code style="color:#c7254e;background-color:#f9f2f4">top.take(np.arange(5))</code> | <code style="color:#002fa7;background-color:#eef2fb">top.head(5)</code> |
| one row, row 0 | <code style="color:#002fa7;background-color:#eef2fb">df.head(1)</code> | <code style="color:#c7254e;background-color:#f9f2f4">nba.take(0)</code> | <code style="color:#002fa7;background-color:#eef2fb">nba.head(1)</code> |
| the last rows, ending at the last row with no gaps | <code style="color:#002fa7;background-color:#eef2fb">df.tail(k)</code> | <code style="color:#c7254e;background-color:#f9f2f4">top.take(np.arange(top.num&#95;rows - 3, top.num&#95;rows))</code> | <code style="color:#002fa7;background-color:#eef2fb">top.tail(3)</code> |
| any other rows: the middle, a list, or every n-th row | <code style="color:#002fa7;background-color:#eef2fb">df.gather(indices)</code> | <code style="color:#c7254e;background-color:#f9f2f4">nba.take(np.arange(3, 6))</code> | <code style="color:#c7254e;background-color:#f9f2f4">nba.gather(np.arange(3, 6))</code> |

Rules:

- <code style="color:#002fa7;background-color:#eef2fb">head</code> **and** <code style="color:#002fa7;background-color:#eef2fb">tail</code> **need no gaps.** <code style="color:#c7254e;background-color:#f9f2f4">np.arange(5)</code> is rows 0 to 4, so it is `head(5)`. <code style="color:#c7254e;background-color:#f9f2f4">np.arange(3, 6)</code> skips rows 0 to 2, so it is not <code style="color:#002fa7;background-color:#eef2fb">head</code>. Use <code style="color:#c7254e;background-color:#f9f2f4">gather(np.arange(3, 6))</code> or <code style="color:#002fa7;background-color:#eef2fb">df.slice(3, 3)</code>.
- <code style="color:#002fa7;background-color:#eef2fb">gather</code> **takes a NumPy array (**<code style="color:#c7254e;background-color:#f9f2f4">make&#95;array</code>**,** <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code>**) or a Series of positions.** Keep <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code> for the index list. <code style="color:#c7254e;background-color:#f9f2f4">top.take(make&#95;array(3, 18, 100))</code> is <code style="color:#c7254e;background-color:#f9f2f4">top.gather(make&#95;array(3, 18, 100))</code>; <code style="color:#c7254e;background-color:#f9f2f4">make&#95;array</code> is unchanged. <code style="color:#c7254e;background-color:#f9f2f4">top.take(np.arange(start, top.num&#95;rows, 10))</code> is <code style="color:#c7254e;background-color:#f9f2f4">top.gather(np.arange(start, top.height, 10))</code>, because it keeps every tenth row.
- **Keep** <code style="color:#002fa7;background-color:#eef2fb">head</code> **or** <code style="color:#002fa7;background-color:#eef2fb">tail</code> **when the result feeds another step.** `head(10)` stays when the chain continues, as in <code style="color:#002fa7;background-color:#eef2fb">group&#95;by('Year').len(name='count').sort('Year').head(10).with&#95;columns(...)</code>. <code style="color:#002fa7;background-color:#eef2fb">.show</code> would end the chain.
- **Use** `show(k)` **when the cell only displays the first rows.** <code style="color:#c7254e;background-color:#f9f2f4">sort("SALARY", descending=True).take(np.arange(5))</code> ends the cell, so it is <code style="color:#002fa7;background-color:#eef2fb">.show(5)</code>. <code style="color:#002fa7;background-color:#eef2fb">show</code> prints and returns `None`.
- **There is no** <code style="color:#002fa7;background-color:#eef2fb">show</code> **for the last rows.** Use <code style="color:#002fa7;background-color:#eef2fb">df.tail(k)</code> as the last line of the cell. The chapter 1–12 notebooks never take the last rows.
- **Position, not label.** All three work on row positions. A sorted frame gives sorted positions, so sort first and then <code style="color:#002fa7;background-color:#eef2fb">head</code>, as Data 8 does with <code style="color:#c7254e;background-color:#f9f2f4">sort(...).take(np.arange(k))</code>.

Notebook cases:

| Data 8 | Polars |
| --- | --- |
| <code style="color:#c7254e;background-color:#f9f2f4">nba.take(0)</code> | <code style="color:#002fa7;background-color:#eef2fb">nba.head(1)</code> |
| <code style="color:#c7254e;background-color:#f9f2f4">nba.take(np.arange(3, 6))</code> | <code style="color:#c7254e;background-color:#f9f2f4">nba.gather(np.arange(3, 6))</code> |
| <code style="color:#c7254e;background-color:#f9f2f4">sort("SALARY", descending=True).take(np.arange(5))</code> | <code style="color:#002fa7;background-color:#eef2fb">sort("SALARY", descending=True).show(5)</code> |
| <code style="color:#c7254e;background-color:#f9f2f4">group('Year').take(np.arange(10)).barh('Year')</code> | <code style="color:#002fa7;background-color:#eef2fb">group&#95;by('Year').len(name='count').sort('Year').head(10)</code> then the chart |
| <code style="color:#c7254e;background-color:#f9f2f4">take(np.arange(num&#95;rows - 1))</code> | <code style="color:#002fa7;background-color:#eef2fb">head(height - 1)</code> |
| <code style="color:#c7254e;background-color:#f9f2f4">top.take(make&#95;array(3, 18, 100))</code> | <code style="color:#c7254e;background-color:#f9f2f4">top.gather(make&#95;array(3, 18, 100))</code> |
| <code style="color:#c7254e;background-color:#f9f2f4">top.take(np.arange(start, top.num&#95;rows, 10))</code> | <code style="color:#c7254e;background-color:#f9f2f4">top.gather(np.arange(start, top.height, 10))</code> |

### GroupBy methods

<code style="color:#002fa7;background-color:#eef2fb">DataFrame.group&#95;by(...)</code> returns a <code style="color:#002fa7;background-color:#eef2fb">GroupBy</code>. Call one of the methods below. These replace <code style="color:#002fa7;background-color:#eef2fb">group&#95;by(...).agg(...)</code>.

Reference: [GroupBy](https://docs.pola.rs/api/python/stable/reference/dataframe/group_by.html).

| GroupBy method | Data 8 group function | Call |
| --- | --- | --- |
| <code style="color:#002fa7;background-color:#eef2fb">.len(name="len")</code> | <code style="color:#c7254e;background-color:#f9f2f4">tbl.group(col)</code> (count rows) | <code style="color:#002fa7;background-color:#eef2fb">df.group&#95;by(pl.col("col")).len().sort("col")</code> |
| <code style="color:#002fa7;background-color:#eef2fb">.sum()</code> | <code style="color:#c7254e;background-color:#f9f2f4">sum</code> | <code style="color:#002fa7;background-color:#eef2fb">df.select("col", "value").group&#95;by(pl.col("col")).sum().sort("col")</code> |
| <code style="color:#002fa7;background-color:#eef2fb">.mean()</code> | <code style="color:#c7254e;background-color:#f9f2f4">np.mean</code> / <code style="color:#c7254e;background-color:#f9f2f4">np.average</code> | <code style="color:#002fa7;background-color:#eef2fb">df.select("col", "value").group&#95;by(pl.col("col")).mean().sort("col")</code> |
| <code style="color:#002fa7;background-color:#eef2fb">.min()</code> | <code style="color:#c7254e;background-color:#f9f2f4">min</code> | <code style="color:#002fa7;background-color:#eef2fb">df.select("col", "value").group&#95;by(pl.col("col")).min().sort("col")</code> |
| <code style="color:#002fa7;background-color:#eef2fb">.max()</code> | <code style="color:#c7254e;background-color:#f9f2f4">max</code> | <code style="color:#002fa7;background-color:#eef2fb">df.group&#95;by(pl.col("col")).max().sort("col")</code> |
| <code style="color:#002fa7;background-color:#eef2fb">.median()</code> | median | <code style="color:#002fa7;background-color:#eef2fb">df.select("col", "value").group&#95;by(pl.col("col")).median().sort("col")</code> |
| <code style="color:#002fa7;background-color:#eef2fb">.quantile(q)</code> | <code style="color:#c7254e;background-color:#f9f2f4">percentile(p)</code> with <code style="color:#c7254e;background-color:#f9f2f4">q = p / 100</code> | <code style="color:#002fa7;background-color:#eef2fb">df.select("col", "value").group&#95;by(pl.col("col")).quantile(p / 100).sort("col")</code> |
| <code style="color:#002fa7;background-color:#eef2fb">.n&#95;unique()</code> | count of distinct values in each column | <code style="color:#002fa7;background-color:#eef2fb">df.group&#95;by(pl.col("col")).n&#95;unique().sort("col")</code> |
| <code style="color:#002fa7;background-color:#eef2fb">.first()</code> / <code style="color:#002fa7;background-color:#eef2fb">.last()</code> | first or last value in each column | <code style="color:#002fa7;background-color:#eef2fb">df.group&#95;by(pl.col("col")).first().sort("col")</code> |
| <code style="color:#002fa7;background-color:#eef2fb">.head(n)</code> / <code style="color:#002fa7;background-color:#eef2fb">.tail(n)</code> | first or last <code style="color:#c7254e;background-color:#f9f2f4">n</code> rows of each group | <code style="color:#002fa7;background-color:#eef2fb">df.group&#95;by(pl.col("col")).head(1).sort("col")</code> |
| <code style="color:#002fa7;background-color:#eef2fb">.all()</code> | each column collected into a list | <code style="color:#002fa7;background-color:#eef2fb">df.group&#95;by(pl.col("col")).all().sort("col")</code> |
| <code style="color:#002fa7;background-color:#eef2fb">.having(&#42;predicates)</code> | keep groups that match, then reduce | <code style="color:#002fa7;background-color:#eef2fb">df.group&#95;by(pl.col("col")).having(pl.len() &gt; 1).len().sort("col")</code> |
| <code style="color:#002fa7;background-color:#eef2fb">.map&#95;groups(function)</code> | a custom Python function | <code style="color:#002fa7;background-color:#eef2fb">df.group&#95;by("col").map&#95;groups(function).sort("col")</code> |

Each reducer runs on every column that is not a group key, and the result keeps those column names. Chain <code style="color:#002fa7;background-color:#eef2fb">.rename({"value": "total"})</code> when the new column needs a different label.

From the conversion notebook:

- <code style="color:#002fa7;background-color:#eef2fb">.sum()</code> on a string column raises <code style="color:#002fa7;background-color:#eef2fb">InvalidOperationError</code>. Select the group key and the numeric columns first: <code style="color:#002fa7;background-color:#eef2fb">df.select("Flavor", "Price").group&#95;by(pl.col("Flavor")).sum().sort("Flavor")</code>.
- <code style="color:#002fa7;background-color:#eef2fb">.max()</code> can run on strings (lexicographic maximum), so <code style="color:#002fa7;background-color:#eef2fb">df.group&#95;by(pl.col("Flavor")).max().sort("Flavor")</code> keeps `Color` and `Price`.
- <code style="color:#002fa7;background-color:#eef2fb">map&#95;groups</code> takes column **names** as the group keys. <code style="color:#002fa7;background-color:#eef2fb">group&#95;by(pl.col("Flavor")).map&#95;groups(...)</code> is not allowed; write <code style="color:#002fa7;background-color:#eef2fb">group&#95;by("Flavor")</code>.
- Polars returns the groups in no fixed order, and it can change from run to run. Data 8 `group` sorts the result by the grouped column, so every <code style="color:#002fa7;background-color:#eef2fb">group&#95;by</code> chain ends with <code style="color:#002fa7;background-color:#eef2fb">.sort(key)</code>, for example <code style="color:#002fa7;background-color:#eef2fb">.sort("Flavor")</code>. Do not use <code style="color:#002fa7;background-color:#eef2fb">maintain&#95;order=True</code>: it keeps first-seen order, which is not what Data 8 shows.

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">df.group_by(pl.col("Flavor")).max().sort("Flavor")
df.select("Flavor", "Price").group_by(pl.col("Flavor")).sum().sort("Flavor")</code></pre>

---

## Visualization Functions

Polars does not plot. Chapters 1–12 draw with **hvPlot** on the DataFrame. Import <code style="color:#002fa7;background-color:#eef2fb">hvplot.polars</code> next to <code style="color:#002fa7;background-color:#eef2fb">import polars as pl</code>. The chart is a method on <code style="color:#002fa7;background-color:#eef2fb">df</code>, so the column names stay in the call. Data 8 blue is `#002fa7` and gold is `#706700`. The third bar color is `#006a94` and the fourth is `#1e6400`.

Titles, axis limits, and marker overlays are hvPlot / HoloViews options, not matplotlib: <code style="color:#002fa7;background-color:#eef2fb">.opts(title=..., ylim=...)</code> and <code style="color:#002fa7;background-color:#eef2fb">chart &#42; hv.Scatter((&#91;x&#93;, &#91;0&#93;)).opts(color="red")</code> (Chapters 11–12).

| Data 8                                            | hvPlot                                                                   | Description                                                              | Polars input                                                                                                                                                                                                             | Polars output                       | Notes                                                                                                                                                                                                                                                          |
| ------------------------------------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.scatter(x&#95;column, y&#95;column)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.hvplot.scatter(x="x&#95;column", y="y&#95;column")</code> | One point per row. `x` is the horizontal column and `y` is the vertical. | 1. **string**: <code style="color:#002fa7;background-color:#eef2fb">x</code>, name of the column on the x-axis 2. **string**: <code style="color:#002fa7;background-color:#eef2fb">y</code>, name of the column on the y-axis | **plot**: draws a scatter plot | Chapter 7: <code style="color:#002fa7;background-color:#eef2fb">actors.hvplot.scatter(x='Number of Movies', y='Total Gross')</code>. A filter stays in front: <code style="color:#002fa7;background-color:#eef2fb">actors.filter(pl.col('Number of Movies') &gt; 10).hvplot.scatter(...)</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.plot(x&#95;column, y&#95;column)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.hvplot.line(x="x&#95;column", y="y&#95;column")</code> | One point per row, connected in order. Same `x` and `y` as scatter. | 1. **string**: <code style="color:#002fa7;background-color:#eef2fb">x</code>, name of the column on the x-axis 2. **string**: <code style="color:#002fa7;background-color:#eef2fb">y</code>, name of the column on the y-axis | **plot**: draws a line graph | Chapter 7: <code style="color:#002fa7;background-color:#eef2fb">movies&#95;by&#95;year.hvplot.line(x='Year', y='Number of Movies')</code>. Integer year ticks: <code style="color:#002fa7;background-color:#eef2fb">xticks=df.get&#95;column('Year').to&#95;list()</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.barh(categories, values)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.hvplot.barh("categories", "values")</code> | Horizontal bars, one per category. Category first, frequency second. | 1. **string**: name of the column with categories 2. **string**: name of the column with values for the categories | **plot**: draws a bar chart | Chapter 7.1: <code style="color:#002fa7;background-color:#eef2fb">icecream.hvplot.barh('Flavor', 'Number of Cartons')</code>. Sort first when the bars should run longest to shortest. |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.barh(categories)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.hvplot.barh("categories", "values")</code> | Same chart. hvPlot still needs the frequency column. | 1. **string**: name of the column with categories 2. **string**: name of the column with values for the categories | **plot**: draws a bar chart | One argument used to mean "the other column." Name both columns. |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.hist(column)</code> / <code style="color:#c7254e;background-color:#f9f2f4">tbl.hist(column, unit=...)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.hvplot.hist("column", bins=10, normed=True, xlabel="column (unit)")</code> | Density histogram of a numeric column. | 1. **string**: name of the column 2. (Optional) **int**: <code style="color:#002fa7;background-color:#eef2fb">bins</code>, the number of bins (default 20; pass 10 to match Data 8) 3. <code style="color:#002fa7;background-color:#eef2fb">normed=True</code> for a density 4. (Optional) **string**: <code style="color:#002fa7;background-color:#eef2fb">xlabel</code>, the x-axis label with units | **plot**: draws a histogram | `hvplot.hist` counts unless <code style="color:#002fa7;background-color:#eef2fb">normed=True</code>, and its default is 20 bins. Data 8 `hist` is a density on 10 bins. `unit` becomes <code style="color:#002fa7;background-color:#eef2fb">xlabel</code>. Chapter 7.2: <code style="color:#002fa7;background-color:#eef2fb">millions.hvplot.hist('Adjusted Gross', bins=10, normed=True, xlabel='Adjusted Gross (Million Dollars)')</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.hist(column, bins=edges)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.hvplot.hist("column", bins=edges, normed=True)</code> | Histogram with those bin edges. | 1. **string**: name of the column 2. **list** or **array** of ints/floats: <code style="color:#002fa7;background-color:#eef2fb">bins</code>, the bin boundaries 3. <code style="color:#002fa7;background-color:#eef2fb">normed=True</code> for a density | **plot**: draws a histogram | A Polars Series of edges is <code style="color:#002fa7;background-color:#eef2fb">.to&#95;list()</code> before `bins`. Keep <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code> for a float edge list. An explicit <code style="color:#002fa7;background-color:#eef2fb">normed=False</code> stays <code style="color:#002fa7;background-color:#eef2fb">normed=False</code>; that figure is counts, not a histogram. |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.scatter(x)</code> / <code style="color:#c7254e;background-color:#f9f2f4">tbl.plot(x)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.hvplot.scatter(x="x", y=&#91;...&#93;)</code> / <code style="color:#002fa7;background-color:#eef2fb">df.hvplot.line(x="x", y=&#91;...&#93;)</code> | Every other column against the common axis. | 1. **string**: <code style="color:#002fa7;background-color:#eef2fb">x</code>, name of the shared x-axis column 2. **list** of strings: <code style="color:#002fa7;background-color:#eef2fb">y</code>, names of the columns to plot 3. (Optional) **list**: <code style="color:#002fa7;background-color:#eef2fb">color</code>, one color per column | **plot**: draws overlaid graphs | A single axis argument does not mean "every other column." Name the vertical columns in `y`. See [Overlaid graphs](#overlaid-graphs). |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.barh(categories)</code> with several value columns | <code style="color:#002fa7;background-color:#eef2fb">df.hvplot.barh("categories", &#91;"c1", "c2", ...&#93;, color=&#91;...&#93;)</code> | Several bar charts on the same axes. | 1. **string**: name of the column with categories 2. **list** of strings: names of the value columns 3. (Optional) **list**: <code style="color:#002fa7;background-color:#eef2fb">color</code>, one color per column | **plot**: draws overlaid bar charts | Chapter 7.3 uses the four Data 8 colors in column order. A selected pair uses the first two. |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.plot(column&#95;for&#95;xticks=k)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.hvplot.line(x="col&#95;k", y=&#91;...&#93;, title=...)</code> | Every other column against column `k`. | 1. **string**: <code style="color:#002fa7;background-color:#eef2fb">x</code>, name of column <code style="color:#002fa7;background-color:#eef2fb">k</code> 2. **list** of strings: <code style="color:#002fa7;background-color:#eef2fb">y</code>, names of the other columns 3. (Optional) **string**: <code style="color:#002fa7;background-color:#eef2fb">title</code> | **plot**: draws a line graph | Chapter 1.3.1: <code style="color:#002fa7;background-color:#eef2fb">cum&#95;counts.hvplot.line(x="Chapter", y=&#91;"Jim", "Tom", "Huck"&#93;, title="Cumulative Number of Times Each Name Appears")</code>. Name the vertical columns in `y`. |

### Overlaid graphs

Chapter 7.3 draws several series on one pair of axes. Select the columns first, then pass the common axis and a list of the other columns.

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">df.hvplot.scatter(x="son", y=["father", "mother"], alpha=0.5, color=["#002fa7", "#706700"])
df.hvplot.line(x="AGE", y=["2014", "2019"], color=["#002fa7", "#706700"])
df.hvplot.barh("Ethnicity/Race", ["USA All", "CA All"], color=["#002fa7", "#706700"])</code></pre>

- **Scatter and line.** `x` is the shared horizontal column. `y` is the list of vertical columns.
- **Barh.** The first argument is the category column. The second is the list of value columns.
- **Colors.** Father / 2014 / USA All are blue `#002fa7`. Mother / 2019 / CA All are gold `#706700`.
- **Categorical bars that should be equally spaced.** Cast a numeric category to string before `barh`, as in <code style="color:#002fa7;background-color:#eef2fb">with&#95;columns(pl.col("Year").cast(pl.String)).hvplot.barh("Year", "count")</code>. A numeric Year would space 1921 far from 1937.

---

## Patterns from Chapters 13–18

| Data 8 | Polars / NumPy / hvPlot | Notes |
| --- | --- | --- |
| <code style="color:#c7254e;background-color:#f9f2f4">minimize(f)</code> | <code style="color:#c7254e;background-color:#f9f2f4">from datascience import minimize</code>, unchanged | Like <code style="color:#c7254e;background-color:#f9f2f4">make&#95;array</code>, <code style="color:#c7254e;background-color:#f9f2f4">sample&#95;proportions</code> and <code style="color:#c7254e;background-color:#f9f2f4">percentile</code>. Returns a NumPy array. |
| <code style="color:#c7254e;background-color:#f9f2f4">standard&#95;units(x)</code> | <code style="color:#002fa7;background-color:#eef2fb">(x - x.mean())/x.std(ddof=0)</code> | Works on a NumPy array and on a Series. <code style="color:#c7254e;background-color:#f9f2f4">np.mean</code> / <code style="color:#c7254e;background-color:#f9f2f4">np.std</code> on a Polars Series can fail. |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.scatter(x, y, fit&#95;line=True)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.hvplot.scatter(x=x, y=y) &#42; hv.Slope(slope, intercept)</code> | Data 8 blue line. |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.scatter(x, y, group=col)</code> with a color column | <code style="color:#002fa7;background-color:#eef2fb">df.hvplot.scatter(x=x, y=y, color="Color", cmap={"darkblue": "#002fa7", "gold": "#706700"})</code> | `by=` assigns colors in first-seen order, which can swap them. |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.move&#95;to&#95;start(c)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.select(c, pl.exclude(c))</code> | Returns a new DataFrame. |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.apply(fn)</code> over whole rows | <code style="color:#002fa7;background-color:#eef2fb">for row in df.iter&#95;rows():</code> + <code style="color:#c7254e;background-color:#f9f2f4">np.append</code>, or <code style="color:#002fa7;background-color:#eef2fb">pl.struct(...).map&#95;elements(fn, return&#95;dtype=...)</code> | <code style="color:#002fa7;background-color:#eef2fb">iter&#95;rows</code> yields tuples, like <code style="color:#002fa7;background-color:#eef2fb">df.row(i)</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.split(k)</code> | <code style="color:#002fa7;background-color:#eef2fb">shuffled = df.sample(n=df.height, with&#95;replacement=False, shuffle=True)</code>, then <code style="color:#002fa7;background-color:#eef2fb">shuffled.head(k)</code> and <code style="color:#002fa7;background-color:#eef2fb">shuffled.tail(df.height - k)</code> |  |
| <code style="color:#c7254e;background-color:#f9f2f4">Table(labels)</code> + <code style="color:#c7254e;background-color:#f9f2f4">.append(row)</code> in a loop | <code style="color:#002fa7;background-color:#eef2fb">pl.DataFrame(schema={...})</code> + <code style="color:#002fa7;background-color:#eef2fb">pl.concat(&#91;df, pl.DataFrame(&#91;pl.Series("a", <span style="color:#c7254e">make&#95;array(a)</span>), pl.Series("b", <span style="color:#c7254e">make&#95;array(b)</span>)&#93;)&#93;)</code> | Chapter 16.3. When the row is one array, <code style="color:#c7254e;background-color:#f9f2f4">row = make&#95;array(...)</code>, use <code style="color:#002fa7;background-color:#eef2fb">pl.DataFrame(&#91;row&#93;, schema=df.schema, orient="row")</code> (Chapter 9.4). |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.join(...)</code> row order | <code style="color:#002fa7;background-color:#eef2fb">dfA.join(dfB, ..., maintain&#95;order="left")</code>, then <code style="color:#002fa7;background-color:#eef2fb">.sort(key)</code> | Data 8 sorts joined rows by the key. |
| <code style="color:#c7254e;background-color:#f9f2f4">np&#95;int &#42; s</code> or <code style="color:#c7254e;background-color:#f9f2f4">s&#91;np&#95;int&#93;</code> | <code style="color:#002fa7;background-color:#eef2fb">s &#42; np&#95;int</code>, <code style="color:#002fa7;background-color:#eef2fb">s.item(i)</code> | Polars 2.1 rejects a NumPy integer on the left of Series arithmetic and as a Series index; loop counters from <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code> are NumPy integers. |
| interval bar <code style="color:#c7254e;background-color:#f9f2f4">plots.plot(&#91;l, r&#93;, &#91;0, 0&#93;, color="gold", lw=8)</code> | <code style="color:#002fa7;background-color:#eef2fb">hv.Segments(&#91;(l, 0, r, 0)&#93;).opts(color="#706700", line&#95;width=8)</code> |  |
| <code style="color:#c7254e;background-color:#f9f2f4">plots.plot(x, y)</code> curve | <code style="color:#002fa7;background-color:#eef2fb">hv.Curve((x, y))</code> | Several panels: HoloViews `+` layout, <code style="color:#002fa7;background-color:#eef2fb">.cols(1)</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">plots.plot(&#91;x1, x2&#93;, &#91;y1, y2&#93;)</code> line | <code style="color:#002fa7;background-color:#eef2fb">hv.Curve(&#91;(x1, y1), (x2, y2)&#93;)</code> | Coordinates stay Python lists (a kept list pattern). |
| <code style="color:#c7254e;background-color:#f9f2f4">plots.scatter(x, y)</code> one marker | <code style="color:#002fa7;background-color:#eef2fb">hv.Scatter((&#91;x&#93;, &#91;y&#93;))</code> | Coordinates stay Python lists. |
| a growing Python list <code style="color:#c7254e;background-color:#f9f2f4">errors = &#91;&#93;</code>, <code style="color:#c7254e;background-color:#f9f2f4">errors.append(v)</code> | unchanged: <code style="color:#002fa7;background-color:#eef2fb">errors = &#91;&#93;</code>, <code style="color:#002fa7;background-color:#eef2fb">errors.append(v)</code> | Chapter 17.6; a kept list pattern. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.array(&#91;'A'&#93;&#42;n + &#91;'B'&#93;&#42;m)</code> | unchanged | Chapter 18; <code style="color:#c7254e;background-color:#f9f2f4">np.array</code> converts the repeated list. |
| <code style="color:#c7254e;background-color:#f9f2f4">np.array(&#91;row&#95;array&#93;)</code> (one-row 2D array) | <code style="color:#002fa7;background-color:#eef2fb">row&#95;array.reshape(1, -1)</code> | Chapter 17.6, for <code style="color:#002fa7;background-color:#eef2fb">pl.from&#95;numpy</code>. |

---

## Table.where Predicates

Filter with a lazy comparison:

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">df.filter(pl.col("age") &gt; 18)
df.filter((pl.col("age") &gt; 18) &amp; (pl.col("city") == "Berkeley"))
df.filter((pl.col("age") &lt; 18) | (pl.col("age") &gt; 65))</code></pre>

| Data 8 predicate                | Polars expression                               | Description                                     | Polars input                                                                                         | Polars output                                                            | Notes                                                                                                 |
| ------------------------------- | ----------------------------------------------- | ----------------------------------------------- | ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------- |
| <code style="color:#c7254e;background-color:#f9f2f4">are.equal&#95;to(Z)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col") == Z</code> | Equal to `Z`. | **number**, **string**, or **Expr**: the value to compare with | **Expr** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> for the rows to keep, used in <code style="color:#002fa7;background-color:#eef2fb">df.filter(...)</code> | Column vs column: <code style="color:#002fa7;background-color:#eef2fb">pl.col("x") == pl.col("y")</code> |
| <code style="color:#c7254e;background-color:#f9f2f4">are.not&#95;equal&#95;to(Z)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col") != Z</code> | Not equal to `Z`. | **number**, **string**, or **Expr**: the value to compare with | **Expr** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> for the rows to keep, used in <code style="color:#002fa7;background-color:#eef2fb">df.filter(...)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("x") != pl.col("y")</code> |
| <code style="color:#c7254e;background-color:#f9f2f4">are.above(x)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col") &gt; x</code> | Greater than `x`. | **number**, **string**, or **Expr**: the value to compare with | **Expr** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> for the rows to keep, used in <code style="color:#002fa7;background-color:#eef2fb">df.filter(...)</code> | Notebook: <code style="color:#002fa7;background-color:#eef2fb">pl.col("Price") &gt; 5</code> |
| <code style="color:#c7254e;background-color:#f9f2f4">are.above&#95;or&#95;equal&#95;to(x)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col") &gt;= x</code> | Greater than or equal to `x`. | **number**, **string**, or **Expr**: the value to compare with | **Expr** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> for the rows to keep, used in <code style="color:#002fa7;background-color:#eef2fb">df.filter(...)</code> |  |
| <code style="color:#c7254e;background-color:#f9f2f4">are.below(x)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col") &lt; x</code> | Less than `x`. | **number**, **string**, or **Expr**: the value to compare with | **Expr** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> for the rows to keep, used in <code style="color:#002fa7;background-color:#eef2fb">df.filter(...)</code> |  |
| <code style="color:#c7254e;background-color:#f9f2f4">are.below&#95;or&#95;equal&#95;to(x)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col") &lt;= x</code> | Less than or equal to `x`. | **number**, **string**, or **Expr**: the value to compare with | **Expr** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> for the rows to keep, used in <code style="color:#002fa7;background-color:#eef2fb">df.filter(...)</code> |  |
| <code style="color:#c7254e;background-color:#f9f2f4">are.between(x, y)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").is&#95;between(x, y, closed="left")</code> | Greater than or equal to `x` and less than `y`. | 1. **number**: the lower bound <code style="color:#002fa7;background-color:#eef2fb">x</code> 2. **number**: the upper bound <code style="color:#002fa7;background-color:#eef2fb">y</code> 3. **string**: <code style="color:#002fa7;background-color:#eef2fb">closed="left"</code> | **Expr** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> for the rows to keep, used in <code style="color:#002fa7;background-color:#eef2fb">df.filter(...)</code> |  |
| <code style="color:#c7254e;background-color:#f9f2f4">are.between&#95;or&#95;equal&#95;to(x, y)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").is&#95;between(x, y, closed="both")</code> | From `x` through `y`, inclusive. | 1. **number**: the lower bound <code style="color:#002fa7;background-color:#eef2fb">x</code> 2. **number**: the upper bound <code style="color:#002fa7;background-color:#eef2fb">y</code> 3. **string**: <code style="color:#002fa7;background-color:#eef2fb">closed="both"</code> | **Expr** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> for the rows to keep, used in <code style="color:#002fa7;background-color:#eef2fb">df.filter(...)</code> | Bounds may themselves be <code style="color:#002fa7;background-color:#eef2fb">pl.col(...)</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">are.strictly&#95;between(x, y)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").is&#95;between(x, y, closed="none")</code> | Greater than `x` and less than `y`. | 1. **number**: the lower bound <code style="color:#002fa7;background-color:#eef2fb">x</code> 2. **number**: the upper bound <code style="color:#002fa7;background-color:#eef2fb">y</code> 3. **string**: <code style="color:#002fa7;background-color:#eef2fb">closed="none"</code> | **Expr** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> for the rows to keep, used in <code style="color:#002fa7;background-color:#eef2fb">df.filter(...)</code> |  |
| <code style="color:#c7254e;background-color:#f9f2f4">are.contained&#95;in(A)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").is&#95;in(A)</code> | Value is an element of `A`. | **array** or **Series**: <code style="color:#002fa7;background-color:#eef2fb">A</code>, the allowed values (<code style="color:#c7254e;background-color:#f9f2f4">make&#95;array(...)</code>) | **Expr** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> for the rows to keep, used in <code style="color:#002fa7;background-color:#eef2fb">df.filter(...)</code> | A string `A` meaning "value is a substring of `A`" needs a small Python predicate via <code style="color:#002fa7;background-color:#eef2fb">map&#95;elements</code>. |
| <code style="color:#c7254e;background-color:#f9f2f4">are.containing(S)</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("col").str.contains(S)</code> | String contains `S`. | 1. **string**: <code style="color:#002fa7;background-color:#eef2fb">S</code>, the substring 2. (Optional) <code style="color:#002fa7;background-color:#eef2fb">literal=True</code> to match plain text, not a pattern | **Expr** of bools: <code style="color:#002fa7;background-color:#eef2fb">True</code> for the rows to keep, used in <code style="color:#002fa7;background-color:#eef2fb">df.filter(...)</code> | <code style="color:#002fa7;background-color:#eef2fb">df.filter(pl.col("name").str.contains("Smith"))</code> |

**Example**

<code style="color:#c7254e;background-color:#f9f2f4">tbl.where("age", are.above(18))</code> → <code style="color:#002fa7;background-color:#eef2fb">df.filter(pl.col("age") &gt; 18)</code>

<code style="color:#c7254e;background-color:#f9f2f4">tbl.where("name", are.containing("Smith"))</code> → <code style="color:#002fa7;background-color:#eef2fb">df.filter(pl.col("name").str.contains("Smith"))</code>

---

## Conditional Columns

<code style="color:#002fa7;background-color:#eef2fb">pl.when().then().otherwise()</code> builds an if-then-else column. The condition is a <code style="color:#002fa7;background-color:#eef2fb">pl.col</code> comparison. This replaces <code style="color:#c7254e;background-color:#f9f2f4">np.where</code> and a row-wise `apply`.

| Data 8                                                                         | Polars Equivalent                                                             | Description                                                                                                                         | Polars input                                                                                                                                                             | Polars output                                                | Notes                                                                                                                                                                         | Example                    |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| <code style="color:#c7254e;background-color:#f9f2f4">tbl.with&#95;column("name", np.where(tbl.column("col") &gt; x, then&#95;val, else&#95;val))</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.when(pl.col("col") &gt; x).then(then&#95;val).otherwise(else&#95;val).alias("name")</code> | `then_val` where the condition is true, otherwise `else_val`. Extra branches are more <code style="color:#002fa7;background-color:#eef2fb">.when().then()</code> calls; the first match wins. | 1. **Expr**: a boolean condition, a <code style="color:#002fa7;background-color:#eef2fb">pl.col</code> comparison 2. value for <code style="color:#002fa7;background-color:#eef2fb">.then(...)</code>, used where the condition is true 3. value for <code style="color:#002fa7;background-color:#eef2fb">.otherwise(...)</code>, used everywhere else | **Expr**: the new column, used in <code style="color:#002fa7;background-color:#eef2fb">with&#95;columns</code> or <code style="color:#002fa7;background-color:#eef2fb">select</code> | [API](https://docs.pola.rs/api/python/stable/reference/expressions/api/polars.when.html). String results use <code style="color:#002fa7;background-color:#eef2fb">pl.lit("x")</code>. Put the expression in <code style="color:#002fa7;background-color:#eef2fb">with&#95;columns</code> or <code style="color:#002fa7;background-color:#eef2fb">select</code>. | [example](#example-select) |

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">df.with_columns(
    pl.when(pl.col("col") &gt; x).then(pl.lit("high")).otherwise(pl.lit("low")).alias("band")
)</code></pre>

---

## Polars Series and lazy `pl.col` expressions

Two ways to name a column, used for different jobs:

| Job | Write | Why |
| --- | --- | --- |
| Values you already have that will be added to a DataFrame | <code style="color:#002fa7;background-color:#eef2fb">pl.Series("x", <span style="color:#c7254e">make&#95;array(1, 2, 3)</span>)</code> | Eager Series, passed to <code style="color:#002fa7;background-color:#eef2fb">pl.DataFrame</code> or <code style="color:#002fa7;background-color:#eef2fb">with&#95;columns</code>. A standalone array is just <code style="color:#c7254e;background-color:#f9f2f4">make&#95;array(1, 2, 3)</code>. |
| A column inside <code style="color:#002fa7;background-color:#eef2fb">filter</code>, <code style="color:#002fa7;background-color:#eef2fb">with&#95;columns</code>, <code style="color:#002fa7;background-color:#eef2fb">select</code>, or <code style="color:#002fa7;background-color:#eef2fb">group&#95;by</code> | <code style="color:#002fa7;background-color:#eef2fb">pl.col("x")</code> | Lazy expression. It also runs on <code style="color:#002fa7;background-color:#eef2fb">df.lazy()</code> and still refers to the right column after you reassign <code style="color:#002fa7;background-color:#eef2fb">df</code>. |
| An integer index column | <code style="color:#002fa7;background-color:#eef2fb">pl.Series("idx", <span style="color:#c7254e">np.arange(n)</span>)</code> or <code style="color:#002fa7;background-color:#eef2fb">pl.int&#95;range(0, pl.len())</code> | Keep <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code>, never Python `range`. <code style="color:#002fa7;background-color:#eef2fb">pl.int&#95;range</code> is the lazy form. |
| A float grid | <code style="color:#c7254e;background-color:#f9f2f4">np.arange(start, stop, step)</code> | Keep <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code>. Do not replace it with <code style="color:#002fa7;background-color:#eef2fb">pl.linear&#95;space</code>. |

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7"><span style="color:#6a737d"># Works on an eager frame and on a LazyFrame</span>
df.lazy().filter(pl.col("Price") &gt; 5).select(pl.col("Flavor"), pl.col("Price")).collect()</code></pre>

<code style="color:#002fa7;background-color:#eef2fb">df.get&#95;column("Price")</code> materializes a Series. Use that when you need a Series in Python (a loop over values, <code style="color:#002fa7;background-color:#eef2fb">.item(i)</code>, or a plotting function). Use <code style="color:#002fa7;background-color:#eef2fb">pl.col("Price")</code> when the next step is still a DataFrame method.

---

## Examples

### Example: `pl.Series` vs `pl.concat_list`

<code style="color:#002fa7;background-color:#eef2fb">pl.Series("nums", <span style="color:#c7254e">make&#95;array(1, 2, 3)</span>)</code> is one Series of scalars: the column form of the standalone array <code style="color:#c7254e;background-color:#f9f2f4">make&#95;array(1, 2, 3)</code>.

<code style="color:#002fa7;background-color:#eef2fb">pl.concat&#95;list</code> builds a list in each row from other columns. Pass <code style="color:#002fa7;background-color:#eef2fb">pl.col</code> expressions or column-name strings.

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

s = pl.Series("nums", <span style="color:#c7254e">make_array(1, 2, 3)</span>)
print(s)
print("dtype:", s.dtype)
print("first element:", s.item(0))

df = pl.DataFrame([
    pl.Series("a", <span style="color:#c7254e">make_array(1, 2, 3)</span>),
    pl.Series("b", <span style="color:#c7254e">make_array(10, 20, 30)</span>),
])
out = df.with_columns(
    pl.concat_list([pl.col("a"), pl.col("b")]).alias("pair")
)
print(out)
print("pair dtype:", out.get_column("pair").dtype)</code></pre>

Output:

```text
shape: (3,)
Series: 'nums' [i64]
[
	1
	2
	3
]
dtype: Int64
first element: 1
shape: (3, 3)
┌─────┬─────┬───────────┐
│ a   ┆ b   ┆ pair      │
│ --- ┆ --- ┆ ---       │
│ i64 ┆ i64 ┆ list[i64] │
╞═════╪═════╪═══════════╡
│ 1   ┆ 10  ┆ [1, 10]   │
│ 2   ┆ 20  ┆ [2, 20]   │
│ 3   ┆ 30  ┆ [3, 30]   │
└─────┴─────┴───────────┘
pair dtype: List(Int64)
```

<code style="color:#002fa7;background-color:#eef2fb">out.get&#95;column("pair")</code> has three elements, and each element is a list: `[1, 10]`, `[2, 20]`, `[3, 30]`.

### Example: Series arithmetic

Arithmetic on a named Series, such as a column taken from a DataFrame. The result keeps the first Series' name. A standalone array would stay NumPy (<code style="color:#c7254e;background-color:#f9f2f4">3 &#42; make&#95;array(1, 2, 3) + 4</code>). Loops use <code style="color:#c7254e;background-color:#f9f2f4">np.arange</code>, not Python `range`.

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7"><span style="color:#c7254e">import numpy as np</span>
<span style="color:#c7254e">from datascience import make_array</span>
import polars as pl

print(pl.Series("x", <span style="color:#c7254e">make_array(1, 2, 3)</span>))
print(3 * pl.Series("x", <span style="color:#c7254e">make_array(1, 2, 3)</span>) + 4)
print(pl.Series("x", <span style="color:#c7254e">make_array(1, 2, 3)</span>) + pl.Series("y", <span style="color:#c7254e">make_array(1, 10, 100)</span>))
print(pl.Series("x", <span style="color:#c7254e">make_array(1.0, 4.0, 9.0)</span>).sqrt())

for i in <span style="color:#c7254e">np.arange(4)</span>:
    print(i)</code></pre>

Output of the three prints:

```text
shape: (3,)
Series: 'x' [i64]
[
	1
	2
	3
]
shape: (3,)
Series: 'x' [i64]
[
	7
	10
	13
]
shape: (3,)
Series: 'x' [i64]
[
	2
	12
	103
]
shape: (3,)
Series: 'x' [f64]
[
	1.0
	2.0
	3.0
]
```

### Example: rounding with negative decimals

<code style="color:#c7254e;background-color:#f9f2f4">np.round(arr, -2)</code> rounds to the nearest hundred. Polars <code style="color:#002fa7;background-color:#eef2fb">Series.round</code> rejects a negative <code style="color:#002fa7;background-color:#eef2fb">decimals</code>, so divide, round to an integer, and multiply back. Also <code style="color:#c7254e;background-color:#f9f2f4">np.std</code> uses <code style="color:#002fa7;background-color:#eef2fb">ddof=0</code>.

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

s = pl.Series("x", <span style="color:#c7254e">make_array(1234.5678, 1250.0, 1350.0)</span>)

print(s.round(1))
print((s / 100).round(0) * 100)
print(s.std(ddof=0))</code></pre>

Output:

```text
shape: (3,)
Series: 'x' [f64]
[
	1234.6
	1250.0
	1350.0
]
shape: (3,)
Series: 'x' [f64]
[
	1200.0
	1200.0
	1400.0
]
51.16720476772423
```

### Example: `.alias`

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

df = pl.DataFrame([
    pl.Series("a", <span style="color:#c7254e">make_array(1, 2, 3)</span>),
    pl.Series("b", <span style="color:#c7254e">make_array(10, 20, 30)</span>),
])

<span style="color:#6a737d"># pl.col * 2 is still named "a", so with_columns replaces column a</span>
print(df.with_columns(pl.col("a") * 2))

<span style="color:#6a737d"># .alias adds a new column</span>
print(df.with_columns((pl.col("a") * 2).alias("a_times_2")))

<span style="color:#6a737d"># Keyword argument is the same as .alias("a_times_2")</span>
print(df.with_columns(a_times_2=pl.col("a") * 2))

<span style="color:#6a737d"># A Series you built in Python keeps the name you gave it</span>
print(pl.Series("tmp", <span style="color:#c7254e">make_array(10, 20, 30)</span>).alias("tens"))</code></pre>

<code style="color:#002fa7;background-color:#eef2fb">pl.col("a") &#42; 2</code> replaces `a`. The <code style="color:#002fa7;background-color:#eef2fb">.alias</code> and keyword forms add <code style="color:#002fa7;background-color:#eef2fb">a&#95;times&#95;2</code> and leave `a` as 1, 2, 3.

GroupBy methods keep the source column names. Rename afterward:

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

grouped = pl.DataFrame([
    pl.Series("g", <span style="color:#c7254e">make_array</span>("x", "x", "y")),
    pl.Series("a", <span style="color:#c7254e">make_array(1, 2, 3)</span>),
    pl.Series("b", <span style="color:#c7254e">make_array(10, 20, 30)</span>),
])
print(
    grouped.group_by(pl.col("g"))
    .sum()
    .sort("g")
    .rename({"a": "total_a", "b": "total_b"})
)</code></pre>

Output:

```text
shape: (2, 3)
┌─────┬─────────┬─────────┐
│ g   ┆ total_a ┆ total_b │
│ --- ┆ ---     ┆ ---     │
│ str ┆ i64     ┆ i64     │
╞═════╪═════════╪═════════╡
│ x   ┆ 3       ┆ 30      │
│ y   ┆ 3       ┆ 30      │
└─────┴─────────┴─────────┘
```

### Example: adding columns with `pl.col`

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

df = pl.DataFrame([
    pl.Series("foo", <span style="color:#c7254e">make_array(1, 2)</span>),
    pl.Series("bar", <span style="color:#c7254e">make_array(3, 4)</span>),
])
print(df.with_columns((pl.col("foo") + pl.col("bar")).alias("baz")))</code></pre>

Output:

```text
shape: (2, 3)
┌─────┬─────┬─────┐
│ foo ┆ bar ┆ baz │
│ --- ┆ --- ┆ --- │
│ i64 ┆ i64 ┆ i64 │
╞═════╪═════╪═════╡
│ 1   ┆ 3   ┆ 4   │
│ 2   ┆ 4   ┆ 6   │
└─────┴─────┴─────┘
```

### Example: `get_column`

<code style="color:#002fa7;background-color:#eef2fb">df.get&#95;column("col")</code> is Data 8 <code style="color:#c7254e;background-color:#f9f2f4">tbl.column(col)</code>: the whole column as a Polars Series. One element is <code style="color:#002fa7;background-color:#eef2fb">.item(0)</code>. Inside <code style="color:#002fa7;background-color:#eef2fb">filter</code>, <code style="color:#002fa7;background-color:#eef2fb">with&#95;columns</code>, and <code style="color:#002fa7;background-color:#eef2fb">group&#95;by</code>, write <code style="color:#002fa7;background-color:#eef2fb">pl.col("col")</code> and keep it lazy.

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

df = pl.DataFrame([
    pl.Series("Flavor", <span style="color:#c7254e">make_array</span>("strawberry", "chocolate", "chocolate")),
    pl.Series("Price", <span style="color:#c7254e">make_array(3.55, 4.75, 5.25)</span>),
])

print(df.get_column("Price"))
print(df.get_column("Price").item(0))
print(df.filter(pl.col("Price") &gt; 4))
print(df.with_columns((pl.col("Price") * 2).alias("double")))
print(df.group_by(pl.col("Flavor")).len().sort("Flavor"))
print(df.filter(pl.col("Flavor") == "strawberry").get_column("Price").item(0))</code></pre>

Output:

```text
shape: (3,)
Series: 'Price' [f64]
[
	3.55
	4.75
	5.25
]
3.55
shape: (2, 2)
┌───────────┬───────┐
│ Flavor    ┆ Price │
│ ---       ┆ ---   │
│ str       ┆ f64   │
╞═══════════╪═══════╡
│ chocolate ┆ 4.75  │
│ chocolate ┆ 5.25  │
└───────────┴───────┘
shape: (3, 3)
┌────────────┬───────┬────────┐
│ Flavor     ┆ Price ┆ double │
│ ---        ┆ ---   ┆ ---    │
│ str        ┆ f64   ┆ f64    │
╞════════════╪═══════╪════════╡
│ strawberry ┆ 3.55  ┆ 7.1    │
│ chocolate  ┆ 4.75  ┆ 9.5    │
│ chocolate  ┆ 5.25  ┆ 10.5   │
└────────────┴───────┴────────┘
shape: (2, 2)
┌────────────┬─────┐
│ Flavor     ┆ len │
│ ---        ┆ --- │
│ str        ┆ u32 │
╞════════════╪═════╡
│ chocolate  ┆ 2   │
│ strawberry ┆ 1   │
└────────────┴─────┘
3.55
```

<code style="color:#002fa7;background-color:#eef2fb">get&#95;column("Price")</code> is the Series. <code style="color:#002fa7;background-color:#eef2fb">.item(0)</code> is the first element, `3.55`. <code style="color:#002fa7;background-color:#eef2fb">filter(pl.col("Price") &gt; 4)</code> keeps the two chocolate rows. <code style="color:#002fa7;background-color:#eef2fb">with&#95;columns</code> adds `double`. <code style="color:#002fa7;background-color:#eef2fb">group&#95;by(pl.col("Flavor")).len().sort("Flavor")</code> counts 2 chocolate and 1 strawberry. After a filter, <code style="color:#002fa7;background-color:#eef2fb">.get&#95;column("Price").item(0)</code> is the strawberry price, `3.55`.

### Example: `.select`

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

df = pl.DataFrame([
    pl.Series("foo", <span style="color:#c7254e">make_array(1, 2, 3)</span>),
    pl.Series("bar", <span style="color:#c7254e">make_array(6, 7, 8)</span>),
    pl.Series("ham", <span style="color:#c7254e">make_array</span>("a", "b", "c")),
])

out = df.select(
    "foo",
    (pl.col("bar") + 1).alias("bar_plus_1"),
    threshold=pl.when(pl.col("foo") &gt; 2).then(10).otherwise(0),
    foo_times_bar=pl.col("foo") * pl.col("bar"),
)
print(out)</code></pre>

Output:

```text
shape: (3, 4)
┌─────┬────────────┬───────────┬───────────────┐
│ foo ┆ bar_plus_1 ┆ threshold ┆ foo_times_bar │
│ --- ┆ ---        ┆ ---       ┆ ---           │
│ i64 ┆ i64        ┆ i32       ┆ i64           │
╞═════╪════════════╪═══════════╪═══════════════╡
│ 1   ┆ 7          ┆ 0         ┆ 6             │
│ 2   ┆ 8          ┆ 0         ┆ 14            │
│ 3   ┆ 9          ┆ 10        ┆ 24            │
└─────┴────────────┴───────────┴───────────────┘
```

### Example: filter with `pl.col`

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

cones = pl.DataFrame([
    pl.Series("Flavor", <span style="color:#c7254e">make_array</span>("strawberry", "chocolate", "chocolate", "strawberry", "chocolate", "bubblegum")),
    pl.Series("Color", <span style="color:#c7254e">make_array</span>("pink", "light brown", "dark brown", "pink", "dark brown", "pink")),
    pl.Series("Price", <span style="color:#c7254e">make_array(3.55, 4.75, 5.25, 5.25, 5.25, 4.75)</span>),
])

print(cones.filter(pl.col("Price") &gt; 5))</code></pre>

`Price` above 5 keeps the three rows whose price is 5.25 (two chocolate, one strawberry). The same expression runs lazily:

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">cones.lazy().filter(pl.col("Price") &gt; 5).select(pl.col("Flavor"), pl.col("Price")).collect()</code></pre>

### Example: `df.rename`

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

df = pl.DataFrame([
    pl.Series("old_label", <span style="color:#c7254e">make_array(1, 2, 3)</span>),
    pl.Series("x", <span style="color:#c7254e">make_array(4, 5, 6)</span>),
])

df2 = df.rename({"old_label": "new_label"})
print(df)   <span style="color:#6a737d"># original unchanged</span>
print(df2)

df3 = df.rename(lambda name: name.upper())
print(df3)</code></pre>

`df2` has columns `new_label` and `x`. `df3` has columns `OLD_LABEL` and `X`. Values are unchanged.

### Example: `group_by` methods

These calls are the conversion notebook, plus <code style="color:#002fa7;background-color:#eef2fb">.len()</code> for Data 8's default count.

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

cones = pl.DataFrame([
    pl.Series("Flavor", <span style="color:#c7254e">make_array</span>("strawberry", "chocolate", "chocolate", "strawberry", "chocolate", "bubblegum")),
    pl.Series("Color", <span style="color:#c7254e">make_array</span>("pink", "light brown", "dark brown", "pink", "dark brown", "pink")),
    pl.Series("Price", <span style="color:#c7254e">make_array(3.55, 4.75, 5.25, 5.25, 5.25, 4.75)</span>),
])

print(cones.group_by(pl.col("Flavor")).max().sort("Flavor"))
print(cones.select("Flavor", "Price").group_by(pl.col("Flavor")).sum().sort("Flavor"))
print(cones.group_by(pl.col("Flavor")).len().sort("Flavor"))
print(cones.group_by(pl.col("Flavor")).having(pl.len() &gt; 1).len().sort("Flavor"))

print(cones.pivot("Flavor", index="Color", values="Price", aggregate_function="max"))</code></pre>

<code style="color:#002fa7;background-color:#eef2fb">.max()</code> keeps `Color` (string maximum) and `Price`. <code style="color:#002fa7;background-color:#eef2fb">.sum()</code> is called only after `select("Flavor", "Price")`. <code style="color:#002fa7;background-color:#eef2fb">.having(pl.len() &gt; 1).len()</code> keeps flavors that appear more than once. <code style="color:#002fa7;background-color:#eef2fb">.sort("Flavor")</code> lists bubblegum, then chocolate, then strawberry, as Data 8 does.

<code style="color:#002fa7;background-color:#eef2fb">.max()</code> on `Price` is 4.75, 5.25, and 5.25. <code style="color:#002fa7;background-color:#eef2fb">.sum()</code> on `Price` is 4.75, 15.25, and 8.8. <code style="color:#002fa7;background-color:#eef2fb">.len()</code> is 1, 3, and 2. The pivot's `dark brown` / `chocolate` cell is 5.25.

A custom Python function uses <code style="color:#002fa7;background-color:#eef2fb">map&#95;groups</code>, with the group key passed as a column name:

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">cones.select("Flavor", "Price").group_by("Flavor").map_groups(
    lambda g: g.select(pl.col("Flavor").first(), pl.col("Price").max())
).sort("Flavor")</code></pre>

Several numeric columns can be summed together, then renamed:

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

movies = pl.DataFrame([
    pl.Series("genre", <span style="color:#c7254e">make_array</span>("Comedy", "Comedy", "Drama", "Drama", "Comedy")),
    pl.Series("score", <span style="color:#c7254e">make_array(72, 65, 88, 91, 70)</span>),
    pl.Series("gross", <span style="color:#c7254e">make_array(100.0, 85.0, 120.0, 150.0, 95.0)</span>),
])
print(
    movies.group_by(pl.col("genre"))
    .sum()
    .sort("genre")
    .rename({"score": "total_score", "gross": "total_gross"})
)
print(
    movies.group_by(pl.col("genre"))
    .mean()
    .sort("genre")
    .rename({"score": "avg_score", "gross": "avg_gross"})
)</code></pre>

Sums: Comedy 207 and 280.0; Drama 179 and 270.0. Means: Comedy 69.0 and about 93.33; Drama 89.5 and 135.0.

### Example: histogram and bins

<code style="color:#002fa7;background-color:#eef2fb">get&#95;column(...).hist(bin&#95;count=10)</code> is the count table for <code style="color:#c7254e;background-color:#f9f2f4">tbl.bin</code> (10 equal-width bins). <code style="color:#002fa7;background-color:#eef2fb">pl.col(...).hist(bins=...)</code> inside <code style="color:#002fa7;background-color:#eef2fb">select</code> returns one count per bin. <code style="color:#002fa7;background-color:#eef2fb">bin&#95;intervals</code> plus <code style="color:#002fa7;background-color:#eef2fb">GroupBy.len()</code> is the form that uses a GroupBy method.

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

df = pl.DataFrame([pl.Series("a", <span style="color:#c7254e">make_array(1, 3, 8, 8, 2, 1, 3)</span>)])

print(df.select(pl.col("a").hist(bins=[1, 2, 3])))
print(df.get_column("a").hist(bin_count=10))

print(
    df.with_columns(
        pl.col("a").bin_intervals([2, 4], labels=["low", "mid", "high"]).alias("bin")
    ).group_by(pl.col("bin")).len().sort("bin")
)</code></pre>

`hist(bins=[1, 2, 3])` returns counts 3 and 2. <code style="color:#002fa7;background-color:#eef2fb">hist(bin&#95;count=10)</code> returns `breakpoint`, `category`, and `count` (the non-empty bins are `[1.0, 1.7]` count 2, `(1.7, 2.4]` count 1, `(2.4, 3.1]` count 2, and `(7.3, 8.0]` count 2). The <code style="color:#002fa7;background-color:#eef2fb">bin&#95;intervals</code> counts are low 2, mid 3, high 2.

### Example: `map_elements`

Use this when the calculation cannot be written with <code style="color:#002fa7;background-color:#eef2fb">pl.col</code> arithmetic. Doubling is native: <code style="color:#002fa7;background-color:#eef2fb">pl.col("a") &#42; 2</code>. Pass Python `int` or `float` as <code style="color:#002fa7;background-color:#eef2fb">return&#95;dtype</code> (not <code style="color:#002fa7;background-color:#eef2fb">pl.Int64</code> / <code style="color:#002fa7;background-color:#eef2fb">pl.Float64</code>); Polars stores those as <code style="color:#002fa7;background-color:#eef2fb">Int64</code> / <code style="color:#002fa7;background-color:#eef2fb">Float64</code>.

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

df = pl.DataFrame([pl.Series("a", <span style="color:#c7254e">make_array(1, 2, 3)</span>)])

df2 = df.with_columns(
    pl.col("a").map_elements(lambda x: x * 2, return_dtype=int).alias("a_times_2")
)
print(df2)

<span style="color:#6a737d"># Float results use return_dtype=float</span>
<span style="color:#6a737d"># pl.col("x").map_elements(predict, return_dtype=float).alias("Prediction")</span></code></pre>

Polars warns that <code style="color:#002fa7;background-color:#eef2fb">map&#95;elements</code> is slower than <code style="color:#002fa7;background-color:#eef2fb">pl.col("a") &#42; 2</code>. The column <code style="color:#002fa7;background-color:#eef2fb">a&#95;times&#95;2</code> is 2, 4, 6.

### Example: `DataFrame.extend` adding rows

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7">import polars as pl
<span style="color:#c7254e">from datascience import make_array</span>

tbl = pl.DataFrame([
    pl.Series("a", <span style="color:#c7254e">make_array(1, 2)</span>),
    pl.Series("b", <span style="color:#c7254e">make_array</span>("x", "y")),
])
tbl.extend(pl.DataFrame([
    pl.Series("a", <span style="color:#c7254e">make_array(3)</span>),
    pl.Series("b", <span style="color:#c7254e">make_array</span>("z")),
]))
print(tbl)

tbl2 = pl.DataFrame([
    pl.Series("a", <span style="color:#c7254e">make_array(1, 2)</span>),
    pl.Series("b", <span style="color:#c7254e">make_array(4, 5)</span>),
])
tbl2.extend(pl.from_dicts([
    {"a": 3, "b": 6},
    {"a": 4, "b": 7},
]))
print(tbl2)</code></pre>

<code style="color:#c7254e;background-color:#f9f2f4">tbl</code> gains the row `(3, "z")`. `tbl2` gains `(3, 6)` and `(4, 7)`.

### Example: one value from a filtered row

<code style="color:#c7254e;background-color:#f9f2f4">np.arange(4)</code> is kept from Data 8 (do not replace it with `range(4)`). It becomes a column, so it is wrapped in <code style="color:#002fa7;background-color:#eef2fb">pl.Series</code>.

<pre style="background-color:#eef2fb;padding:12px;border-radius:4px"><code style="color:#002fa7"><span style="color:#c7254e">import numpy as np</span>
<span style="color:#c7254e">from datascience import make_array</span>
import polars as pl

streets = pl.Series("Street", <span style="color:#c7254e">make_array</span>("Bancroft", "Durant", "Channing", "Haste"))
southside = pl.DataFrame([streets]).with_columns(
    pl.Series("Blocks from campus", <span style="color:#c7254e">np.arange(4)</span>)
)
print(southside.filter(pl.col("Street") == "Channing").get_column("Street").item(0))</code></pre>

Output:

```text
Channing
```
