# Data 8 to Polars Reference

Mapping of [Data 8 Python Reference](https://data8.org/datascience/reference-nb/datascience-reference.html) functions and methods to Polars equivalents. Use this when converting notebooks from the `datascience` library to Polars.

Patterns below follow [polars_conversion_1.ipynb](polars_conversion_1.ipynb) and the [GroupBy API](https://docs.pola.rs/api/python/stable/reference/dataframe/group_by.html).

**Conventions**

- `df` is a Polars DataFrame. `s` is a **Polars Series**, created as `pl.Series("name", values)`. A Pandas Series appears only after `df.to_pandas()`.
- Inside `filter`, `with_columns`, `select`, and `group_by`, refer to a column with the lazy expression `pl.col("name")`. The same expression works on a DataFrame and on `df.lazy()`.
- A standalone array (Data 8 `make_array` or a NumPy array) is a named Polars Series. An integer range used as a Series is `pl.int_range` (`Int64`; start included, end excluded). A fractional range stays `np.arange`. Do not replace `np.arange` with `pl.linear_space`. An eager `pl.int_range` Series is named `"literal"` unless `.alias` gives it a name.
- Group reductions use a [GroupBy method](https://docs.pola.rs/api/python/stable/reference/dataframe/group_by.html) such as `.sum()` or `.max()`. See [GroupBy methods](#groupby-methods).

**Description:** The "Description" column gives a brief description of what the Polars function does, including what each argument is used for.

**Polars input/output:** The "Polars input" and "Polars output" columns are filled from the [Polars Python API reference](https://docs.pola.rs/api/python/stable/reference/) (DataFrame, Series, Expr, I/O, and GroupBy).

---

## Built-In Python Functions

| Data 8 | Polars / Python | Description | Polars input | Polars output | Notes |
| --- | --- | --- | --- | --- | --- |
| `str(val)` | `str(val)` | Converts `val` to a **string**. For a Polars Series or lazy column, cast every element. | — | — | Scalars: `str(val)`. Series: `s.cast(pl.Utf8)`. Expression: `pl.col("col").cast(pl.Utf8)`. |
| `int(num)` | `int(num)` | Converts `num` to an **int**. For a column, cast every element. | — | — | Scalars: `int(num)`. Series: `s.cast(pl.Int64)`. Expression: `pl.col("col").cast(pl.Int64)`. |
| `float(num)` | `float(num)` | Converts `num` to a **float**. For a column, cast every element. | — | — | Scalars: `float(num)`. Series: `s.cast(pl.Float64)`. Expression: `pl.col("col").cast(pl.Float64)`. |
| `len(arr)` | `len(s)` or `s.len()` or `df.height` | Length of a Polars Series, or the number of rows in a DataFrame. | attribute or `Series.len()` | `int` | Row count inside an expression: `pl.len()`. Python `len(s)` works on a Polars Series, so `sum(s) / len(s)` can stay. |
| `max(arr)` | `s.max()` | Maximum value in a Polars Series. Per group, use `GroupBy.max()`. | Series, or `pl.col("col").max()` in `select` | scalar | `s = df.get_column("col")`. |
| `min(arr)` | `s.min()` | Minimum value in a Polars Series. Per group, use `GroupBy.min()`. | Series, or `pl.col("col").min()` in `select` | scalar | Series method, or `pl.col` inside `select`. |
| `sum(arr)` | `s.sum()` | Sum of the values in a Polars Series. Per group, use `GroupBy.sum()`. | Series, or `pl.col("col").sum()` in `select` | scalar | Series method, or `pl.col` inside `select`. Python `sum(s)` also works on a Polars Series. |
| `abs(num)` | `abs(num)` or `s.abs()` | Absolute value of a number, or of each element. | scalar, Series, or `pl.col("col").abs()` | scalar, Series, or Expr | Expression form stays lazy inside `with_columns`. |
| `round(number)` / `round(number, ndigits)` | `round(x, ndigits)` or `s.round(ndigits)` | Rounds a number or each element. `ndigits` is digits after the decimal point. | `ndigits` (optional); or `pl.col("col").round(ndigits)` | scalar, Series, or Expr | Scalar, Series, or `pl.col`. |
| `print(input, ...)` | `print(...)` | Prints the input. Multiple inputs are separated by spaces by default. | — | — | Same. |
| `type(object)` | `type(object)` | Returns the type of `object`. | — | — | Same. For a column dtype, use `s.dtype` or `df.schema`. |

---

## Arrays: `make_array` and NumPy → Polars Series

Data 8 arrays and one-dimensional NumPy arrays become a **named Polars Series**. The conversion notebook always passes a name: `pl.Series("x", [1, 2, 3])`.

| Data 8 | Polars Series | Description | Polars input | Polars output | Notes |
| --- | --- | --- | --- | --- | --- |
| `make_array(val1, val2, ...)` | `pl.Series("name", [val1, val2, ...])` | One Polars Series of scalar values. For a column of lists, use `pl.concat_list`. | name, values; or expressions in `pl.concat_list([pl.col("a"), pl.col("b")])` | `Series` or `Expr` (list) | Example: `pl.Series("highs", [v1, v2, v3, v4])`. [example](#example-plseries-vs-plconcat_list) |
| `arr.size` | `s.len()` | Number of elements. | `Series.len()` | `int` | Data 8 and NumPy use the attribute `size`. Polars uses the method `len()`, as in `highs.len()`. |
| `np.array([1, 2, 3])` | `pl.Series("x", [1, 2, 3])` | Same construction as `make_array`. The result name in arithmetic comes from the first Series. | name, Python list | `Series` | [example](#example-series-arithmetic) |
| `np.mean(arr)` / `np.average(arr)` | `s.mean()` | Mean of the Series. Per group: `GroupBy.mean()`. | Series, or `pl.col("col").mean()` | scalar or Expr | Grouped form selects numeric columns first. |
| `np.sum(arr)` | `s.sum()` | Sum of the Series. Per group: `GroupBy.sum()`. | Series, or `pl.col("col").sum()` | scalar or Expr | Same as the built-in for one Series. |
| `np.prod(arr)` | `s.product()` | Product of the values in the Series. | Series, or `pl.col("col").product()` | scalar or Expr | There is no GroupBy product method. |
| `np.sqrt(num)` | `s.sqrt()` | Square root of each element. A Python number uses `num ** 0.5`. | Series or `pl.col("col").sqrt()` | Series or Expr | [example](#example-series-arithmetic) |
| `np.arange(stop)` / `np.arange(start, stop)` / `np.arange(start, stop, step)` | `pl.int_range(start, end, step, eager=True)` or `pl.Series("name", range(start, stop, step))` | Integers from `start` up to but excluding `stop`, stepping by an integer `step`. Defaults: `start=0`, `step=1`. One argument is the end. | `eager=True` returns a Series. Omit it for an expression. `step` must be an int. | `Series` of `Int64`, or `Expr` | Output dtype is an integer such as `Int64`. Start is inclusive and end is exclusive. An eager Series is named `"literal"` unless `.alias` changes it. Loops use `for i in range(10)`. A float step stays `np.arange`. |
| `np.arange(start, stop, float_step)` | `np.arange(start, stop, float_step)` | Equal-spaced floats. Keep `np.arange`. | start, stop, step | ndarray | Do not replace this with `pl.linear_space`. Histogram bins and other float grids stay `np.arange`, including `np.arange(1.5, -2, -0.5)` and `np.arange(30.5, 69.6, 1)`. |
| `np.count_nonzero(arr)` | `(s != 0).sum()` | Number of non-zero elements. On a boolean Series, `s.sum()` counts `True`. | Series or `pl.col("col") != 0` | scalar or Expr | `(pl.col("col") != 0).sum()` inside `select`. A boolean Series has no `!=` comparison, so count those with `s.sum()`. |
| `np.all(arr)` | `(s != 0).all()` | True when every numeric element is non-zero. | boolean Expr from `s != 0`, or `pl.col("col") != 0` | scalar or Expr | `Series.all` requires a boolean Series. A boolean Series uses `s.all()`. |
| `np.any(arr)` | `(s != 0).any()` | True when any numeric element is non-zero. | boolean Expr from `s != 0`, or `pl.col("col") != 0` | scalar or Expr | `Series.any` requires a boolean Series. A boolean Series uses `s.any()`. |
| `np.append(arr, item)` | `pl.concat([s, pl.Series(s.name, [item])])` | A new Series with `item` at the end. The original Series is unchanged. | two Series | `Series` | Polars builds a new Series. |
| `np.cumsum(arr)` | `s.cum_sum()` | Cumulative sum. Each element includes itself and everything before it. | Series or `pl.col("col").cum_sum()` | Series or Expr | Running total. |
| `np.diff(arr)` | `s.diff()` | Consecutive differences `s[i] - s[i-1]`. The first value is null. | Series or `pl.col("col").diff()` | Series or Expr | The Series keeps its length and starts with null. NumPy returns one fewer value, and its first value is the second element minus the first. On the temperature Series, `highs.diff()` is null, then 0.787, 0.198, and 0.579. `s.diff(null_behavior="drop")` matches the shorter NumPy result. |
| `np.sort(arr)` | `s.sort()` | A new Series sorted ascending. This does not sort table rows. | Series; table rows use `df.sort` | `Series` | Table order: `df.sort("col")`. |
| `arr.item(i)` | `s.item(i)` | The element at index `i` (indexes start at 0). | `i`: int | scalar | Also `s[i]`. On a one-row frame: `df.get_column("col").item(0)`. |
| `arr.take(indices)` | `s.gather(indices)` | Elements at the given indexes. `indices` is a list or a Series. | `indices`: list or Series | `Series` | Row selection on a frame is `df.gather(indices)`. |
| `np.random.choice(options)` / `np.random.choice(options, k)` | `s.sample(1).item()` / `s.sample(n=k, with_replacement=True)` | One draw, or `k` draws with replacement. | `n`; `with_replacement=False` by default | scalar or `Series` | NumPy draws with replacement unless `replace=False`. Polars `Series.sample` draws without replacement, so the Data 8 default passes `with_replacement=True`. `replace=False` is `with_replacement=False`. There is no `p=` weights argument. |
| shuffle the elements | `s.shuffle()` | Random permutation of the Series. Every value is kept once. | optional `seed` | `Series` | Also `pl.col("col").shuffle()` inside `select` or `with_columns`. `DataFrame` has no `.shuffle()`. Chapter 12 shuffles a label column this way. |
| `percentile(p, arr)` | `s.quantile(p / 100)` | Quantile of the Series. Polars uses 0–1, so divide a percentile by 100. | `quantile` in 0–1; optional `interpolation` | scalar | Per group: `GroupBy.quantile(p / 100)`. |

---

## String Methods and Operations

On a DataFrame, call string methods on `pl.col("col")` and add the result with `.alias("name")`. On a Series you already hold, call `s.str...`.

| Data 8 | Polars Equivalent | Description | Polars input | Polars output | Notes |
| --- | --- | --- | --- | --- | --- |
| `s.upper()` | `pl.col("col").str.to_uppercase()` | Uppercase copy of each string. | string Expr or `s.str.to_uppercase()` | Expr or Series | Use inside `with_columns` / `select`. |
| `s.lower()` | `pl.col("col").str.to_lowercase()` | Lowercase copy of each string. | string Expr or Series | Expr or Series | Column of strings. |
| `s.replace(old, new)` | `pl.col("col").str.replace_all(old, new)` | Replace every `old` with `new`. `str.replace` replaces the first match only. | `pattern`, `value`; optional `literal` | Expr or Series | First match only: `str.replace(old, new)`. |
| `s.split()` / `s.split(separator)` / `s.split(separator, maxsplit)` | `pl.col("col").str.split(separator)` | Split each string into a list. Omit `separator` to split on whitespace. `n` limits the number of splits. | optional `by`, `n` | Expr or Series of lists | Returns a list column. |
| `s.join(iterable)` | `separator.join(s.to_list())` or `pl.concat_str` | Join elements into one string. A Python list uses `separator.join(iterable)`. A list column uses `pl.concat_str`. | separator; list Expr | `str` or Expr | `"-".join(s.to_list())` for one Series. |
| `s.strip(chars)` | `pl.col("col").str.strip_chars(chars)` | Trim `chars` from both ends. Omit `chars` to trim whitespace. | optional `characters` | Expr or Series | Default strips whitespace. |
| `substr in s` | `pl.col("col").str.contains(substr)` | `True` where the string contains `substr`. | `pattern`; optional `literal`, `strict` | boolean Expr or Series | Use in `df.filter(...)`. |
| `substr not in s` | `~pl.col("col").str.contains(substr)` | `True` where the string does not contain `substr`. | same as above | boolean Expr or Series | Negate with `~`. |
| `np.char.count(arr, substr)` | `s.str.count_matches(substr, literal=True)` | Count of a literal substring in each element. | `pattern`; `literal=True` | Series | Without `literal=True` the pattern is a regular expression. `.` then matches every character. |
| `np.char.find(arr, substr)` | `s.str.find(substr, literal=True)` | First index of `substr` in each element. | `pattern`; `literal=True` | Series | A missing search is null. NumPy returns -1. |
| `np.char.rfind(arr, substr)` | — | Last index of `substr` in each element. | — | — | Polars has no Series method for the last position. |
| `np.char.isalpha(arr)` | `s.str.contains(r"^\p{Alphabetic}+$")` | True when every character is a letter. | regex pattern | boolean Series | |
| `np.char.isnumeric(arr)` | `s.str.contains(r"^\d+$")` | True when every character is a decimal digit. | regex pattern | boolean Series | Matches decimal digits, not every Unicode numeric character. |

---

## Naming Columns with `.alias()`

Data 8 names a new column in the call: `tbl.with_column("pair", values)`. Polars writes the expression first, then names it with `.alias("pair")`.

`.alias("name")` works on:

- a **Polars Series**, such as `pl.Series("tmp", [1, 2, 3])`
- a **lazy expression**, such as `pl.col("a")`, `pl.col("a") * 2`, `pl.concat_list([pl.col("a"), pl.col("b")])`, or `pl.when(...)`

It only changes the **column name**. The values stay the same, and the original column is unchanged.

| Polars | Description | Input | Output |
| --- | --- | --- | --- |
| `s.alias("name")` | Names a Polars Series `"name"`. | `name`: string | Series (same values, new name) |
| `expr.alias("name")` | Names a lazy expression `"name"`. This is the form used for `pl.col`. | `name`: string | Expr |
| `df.with_columns(name=expr)` | The keyword is the new column name. Same result as `.alias("name")`. | keyword plus Series or expression | `DataFrame` |

**When you would use it**

- **Computed column.** The conversion notebook adds one with `(pl.col("foo") + pl.col("bar")).alias("baz")`.
- **Series of ready-made values.** `pl.Series("Blocks from campus", range(n))` is already named. Call `.alias` only when that name should change.
- **A new label after a group reduction.** GroupBy methods keep the original column names. Rename with `.rename({"score": "total_score"})`.

`.alias` is for a column you are creating. `df.rename({"old": "new"})` changes a label that already exists.

`df.with_columns(pl.col("a") * 2)` overwrites `a`. `df.with_columns((pl.col("a") * 2).alias("a_times_2"))` keeps `a` and adds `a_times_2`.

See [example](#example-alias).

---

## Tables and Table Methods

Build a frame from named Series. In Polars 2.0 the Series length is the frame height, so pass them to the constructor:

```python
import polars as pl

df = pl.DataFrame([
    pl.Series("foo", [1, 2]),
    pl.Series("bar", [3, 4]),
])

# Computed column: lazy pl.col expressions, then .alias
df = df.with_columns(
    (pl.col("foo") + pl.col("bar")).alias("baz")
)

# A column of already-computed values. Length must match the frame.
df = df.with_columns(
    pl.Series("idx", range(df.height))
)

# The same index as a lazy expression (no Python row count)
df = df.with_columns(
    pl.int_range(0, pl.len()).alias("idx")
)
```

`with_columns` adds columns to a frame that already has rows. See [example](#example-adding-columns-with-plcol).

| Data 8 | Polars Equivalent | Description | Polars input | Polars output | Notes | Example |
| --- | --- | --- | --- | --- | --- | --- |
| `Table()` | `pl.DataFrame()` | Empty DataFrame. | `data`, `schema`, `schema_overrides`, `strict=True`, `orient` — [API](https://docs.pola.rs/api/python/stable/reference/dataframe/) | `DataFrame` | To start from columns, pass Series: `pl.DataFrame([pl.Series("a", [1]), ...])`. | |
| `Table().read_table(filename)` | `pl.read_csv(filename)` or `pl.read_parquet(filename)` | DataFrame from a file. Pick the reader that matches the file type. | [I/O reference](https://docs.pola.rs/api/python/stable/reference/io.html) | `DataFrame` | `pl.scan_csv` returns a LazyFrame; the same `pl.col` expressions apply. A file that separates rows with carriage returns needs `eol_char='\r'`. A carriage return is the character `\r`. Most files end each row with a line feed `\n`, or with both (`\r\n`). `movies_by_year.csv`, `minard.csv`, `educ_inc.csv`, and `family_heights.csv` end each row with `\r` only. | |
| `tbl.with_column(name, values)` | `df.with_columns(pl.Series(name, values))` or `df.with_columns(expr.alias(name))` | A DataFrame with an additional or replaced column. | `*exprs`, `**named_exprs` | `DataFrame` | Computed columns use `pl.col`. A constant uses `pl.lit(x).alias(name)`. | [example](#example-adding-columns-with-plcol) |
| `tbl.with_columns(n1, v1, n2, v2, ...)` | `pl.DataFrame([pl.Series(n1, v1), pl.Series(n2, v2)])` or several expressions in one `with_columns` | Several columns in one call. | Series list, or `df.with_columns(*exprs, **named_exprs)` | `DataFrame` | Name every Series. | [example](#example-adding-columns-with-plcol) |
| `tbl.column(col)` | `df.get_column("col")` | The column as a Polars Series. | `name`: string | `Series` | Inside `filter` / `with_columns` / `group_by`, write `pl.col("col")` and keep it lazy. One element: `.item(0)`. | [example](#example-get_column) |
| `tbl.column(index)` | `df.to_series(index)` | The column at that position as a Series. Positions start at 0. | `index`: int | `Series` | `to_series(1)` is the second column. By name, use `get_column`. | |
| `tbl.num_rows` | `df.height` or `len(df)` | Number of rows. | attribute | `int` | Inside an expression: `pl.len()`. | |
| `tbl.num_columns` | `df.width` or `len(df.columns)` | Number of columns. | attribute | `int` | Also `df.shape[1]`. | |
| `tbl.labels` | `df.columns` | Column names. | attribute | `list[str]` | [API: `DataFrame.columns`](https://docs.pola.rs/api/python/stable/reference/dataframe/) | |
| `tbl.select(col1, col2, ...)` | `df.select("col1", "col2", ...)` | Copy with only some columns. Arguments may be names or `pl.col` expressions. | `*exprs`, `**named_exprs` | `DataFrame` | Select numeric columns before `GroupBy.sum()` when other columns are strings. | [example](#example-select) |
| `tbl.drop(col1, col2, ...)` | `df.drop("col1", "col2", ...)` | Copy without those columns. | `*columns`, `strict` | `DataFrame` | Original frame stays unless you reassign. | |
| `tbl.drop(...)` returning the column | `df.drop_in_place("col")` | Removes the column in place and returns it as a Series. | column name | `Series` | Mutates `df`. | |
| `tbl.relabeled(old_label, new_label)` | `df.rename({old_label: new_label})` | New DataFrame with one or more columns renamed. | `mapping` or a function of the name; `strict` | `DataFrame` | Original is unchanged. After a group reduction, `.rename` is how you label `total_score`. | [example](#example-dfrename) |
| `tbl.relabeled(column_index, new_label)` | `df.rename({df.columns[column_index]: new_label})` | Rename the column at that position. Positions start at 0. | index into `df.columns` | `DataFrame` | `relabeled(3, "SALARY")` renames column 3. In `nba_salaries.csv` that column is `"'15-'16 SALARY"`, so `rename({"'15-'16 SALARY": "SALARY"})` is the same rename. | |
| `tbl.show(n)` | `df.show(n)` | Prints the first `n` rows. Data 8 `show(n)` displays those rows. The textbook notebooks use `.show(n)` for every cell that only displays rows. | `limit`: int or `None` | `None` | Data 8 `show()` with no argument displays the first 10 rows. A Polars display shows the first five and the last five. `show` prints and returns `None`, so nothing can be chained after it. Cells like `nba_salaries.sort("PLAYER").show(5)` end the chain with `show`. | |
| rows that feed a later step | `df.head(n)` | First `n` rows as a DataFrame. Use it only when the result is used again, because it is not a display call. | `n` | `DataFrame` | Notebook examples: `histogram_elements.head(num_bins).with_columns(...)` and `movies_and_years.group_by('Year').len(name='count').sort('Year').head(10).with_columns(...).hvplot.barh(...)`. Both continue the chain, so `.show` would break them. Also `df.limit(n)`. | |
| `tbl.show()` | `df.show(None)` | Every row. `None` prints all rows. | `limit`: int or `None`. Default is 5. | `None` | `show()` with no argument prints the first five rows. A bare DataFrame still shows the first five and the last five. | |
| long strings in a display | `with pl.Config(fmt_str_lengths=160): display(df)` | How many characters of each string to show. | `fmt_str_lengths` | display | The default stops at about 30 characters. Chapter text uses 160 so the opening sentence is visible. | |
| last rows | `df.tail(n)` | Last `n` rows. | `n` | `DataFrame` | Data 8 has no `show` for the last rows, and the chapter 1–12 notebooks never call `.tail`. To display the last rows, a bare `df.tail(n)` as the last line of a cell works. | |
| `tbl.sort(column_name)` | `df.sort("column_name")` | Copy sorted by a column, ascending. | `by`, `*more_by`, `descending=False`, `nulls_last`, `maintain_order` | `DataFrame` | Sort key can be `pl.col("column_name")`. `by` may be positional or named: `df.sort("SALARY")` and `df.sort(by="SALARY")` match. | |
| `tbl.sort(column_index)` | `df.sort(pl.nth(column_index))` | Copy sorted by the column at that position, ascending. Positions start at 0. | `pl.nth(index)` as `by` | `DataFrame` | `nba_salaries.sort(pl.nth(0)).show(5)` sorts by the first column and shows five rows. If column 0 is `"PLAYER"`, this matches `df.sort("PLAYER").show(5)`. | |
| `tbl.sort(column_name, descending=True)` | `df.sort("column_name", descending=True)` | Copy sorted descending. | `descending` is named | `DataFrame` | Pass `descending` by name. A second positional argument is another column to sort by, not `descending`: `df.sort("SALARY", True)` does not sort descending. `df.sort(by="SALARY", descending=True)` is the same call as `df.sort("SALARY", descending=True)`. | |
| `tbl.where(column, predicate)` | `df.filter(pl.col("column") <op> value)` | Copy with only the rows that match. Combine comparisons with `&` (and) or a vertical bar (or), and wrap each comparison in parentheses. | boolean Expr from `pl.col` | `DataFrame` | The notebook writes `df.filter(pl.col("Price") > 5)`. | [example](#example-filter-with-plcol) |
| rows that fail the predicate | `df.filter(~(pl.col("column") <op> value))` | Copy with the matching rows removed. | boolean Expr | `DataFrame` | See [Table.where Predicates](#tablewhere-predicates). | |
| `tbl.take(np.arange(k))` / `tbl.take(0)` | `df.head(k)` / `df.head(1)` | `take` of the first rows is `head`. | `n` | `DataFrame` | The index list `np.arange(k)` is the first `k` rows. If the cell only displays them, it ends with `.show(k)` instead. See [Table.take: head, tail, or gather](#tabletake-head-tail-or-gather). | |
| `tbl.take(np.arange(num_rows - k, num_rows))` | `df.tail(k)` | `take` of the last rows is `tail`. | `n` | `DataFrame` | The index list ends at the last row. See [Table.take: head, tail, or gather](#tabletake-head-tail-or-gather). | |
| `tbl.take(row_indices)` | `df.gather(row_indices)` | Rows at the given indexes. Use `gather` only when the rows are neither the first few nor the last few. | list, `np.arange`, or Series of indexes | `DataFrame` | `gather([3, 18, 100])` and `gather(np.arange(3, 6))` stay `gather`. Keep `np.arange` for the index list. See [Table.take: head, tail, or gather](#tabletake-head-tail-or-gather). | |
| `tbl.sample(k)` | `df.sample(n=k, with_replacement=True)` | `k` rows, drawn with replacement. | `n`; `with_replacement=True` | `DataFrame` | Data 8 `Table.sample` draws with replacement unless `with_replacement=False` is passed. Polars `DataFrame.sample` draws without replacement, and with no `n` it draws one row. Pass `with_replacement=True` to match the Data 8 default. Polars `sample` has no weights. `sample_proportions` stays a helper that draws with `np.random.choice(..., p=proportions)`, or the unchanged `datascience` function, and the draws are stored in a Series. | |
| `tbl.sample(k, with_replacement=False)` | `df.sample(n=k, with_replacement=False)` | `k` distinct rows. | `n`; `with_replacement=False` | `DataFrame` | Both sides draw without replacement. Polars still needs `n`: omitting it draws one row. | |
| `tbl.sample()` / `tbl.sample(with_replacement=False)` | `df.sample(n=df.height, with_replacement=False, shuffle=True)` | A shuffle of every row. | `n=df.height`; `with_replacement=False`; `shuffle=True` | `DataFrame` | Data 8 with no `k` permutes the whole table. Polars with no `n` returns one row, so the call passes `n=df.height`. `DataFrame` has no `.shuffle()`. | |
| `tbl.sample(with_replacement=False).column(col)` | `pl.col("col").shuffle()` | Random permutation of one column. Every value is kept once. | optional `seed` | `Expr` or `Series` | Chapter 12 label shuffles: `select(pl.col("Maternal Smoker").shuffle()).to_series(0)`, or `with_columns(pl.col("Group").shuffle().alias("Shuffled Label"))`. `s.shuffle()` is the Series form. `DataFrame` has no `.shuffle()`. | |
| `tbl.bin(column)` / `tbl.bin(column, bins)` | `df.get_column(col).hist(bin_count=10)` or `hist(bins=edges)` | Counts of values in each interval. Default is 10 equal-width bins from min to max. | `bin_count` or `bins` | `DataFrame` with breakpoint, category, and count | To label bins and then count them: `bin_intervals` plus `GroupBy.len()`. `Series.cut` is deprecated. | [example](#example-histogram-and-bins) |
| `tbl.apply(function)` / `tbl.apply(function, col1, col2, ...)` | `df.with_columns(pl.col("col").map_elements(function, return_dtype=...).alias("Name"))` | Applies a Python function to each element. Slower than a native expression. | `function`, `return_dtype` | Expr | Prefer `pl.col("a") * 2` when a native expression exists. Several columns: `pl.struct(["a", "b"]).map_elements(...)`. | [example](#example-map_elements) |
| `tbl.group(column_or_columns)` | `df.group_by(pl.col("col")).len()` | One row per distinct value. The default Data 8 summary is a count. | `pl.col` or a list of columns; optional `maintain_order=True` | `DataFrame` | `len(name="count")` names the count column. | [example](#example-group_by-methods) |
| `tbl.group(column_or_columns, function)` | `df.group_by(pl.col("col")).max()` and the other GroupBy methods | Reduce the other columns with a [GroupBy method](#groupby-methods). | group key, then `.sum()`, `.mean()`, `.max()`, `.min()`, `.median()`, `.quantile()`, `.first()`, `.last()`, `.n_unique()`, `.head()`, `.tail()`, `.all()`, `.having()`, or `.map_groups()` | `DataFrame` | Select columns before `.sum()` or `.mean()` when a string column is present. `.max()` can run on strings. | [example](#example-group_by-methods) |
| `tbl.pivot(col1, col2)` | `df.pivot(col1, index=col2, aggregate_function="len")` | Pivot: each value of `col1` becomes a column, each value of `col2` becomes a row. Cells are counts. | `on` (positional), `index`, `values`, `aggregate_function` (`"min"`, `"max"`, `"first"`, `"last"`, `"sum"`, `"mean"`, `"median"`, `"len"`) | `DataFrame` | The first argument is `on` (formerly `columns`). | [example](#example-group_by-methods) |
| `tbl.pivot(col1, col2, values, collect)` | `df.pivot(col1, index=col2, values=values, aggregate_function=collect)` | Same pivot, aggregating a third column. | `aggregate_function="max"`, `"sum"`, `"mean"`, and the other names above | `DataFrame` | Notebook: `df.pivot("Flavor", index="Color", values="Price", aggregate_function="max")`. | [example](#example-group_by-methods) |
| `tblA.join(colA, tblB)` / `tblA.join(colA, tblB, colB)` | `dfA.join(dfB, on="colA")` or `dfA.join(dfB, left_on="colA", right_on="colB")` | Rows whose key appears in both frames, with columns from both. | `on` or `left_on`/`right_on`; `how` (`inner`, `left`, `right`, `full`, `semi`, `anti`, `cross`) | `DataFrame` | Default `how` is inner. | |
| `tbl.with_row(values)` | `pl.concat([df, pl.DataFrame([dict(zip(df.columns, values))])])` or `df.extend(other)` | Adds one row. Match column names and types. | a one-row DataFrame | `DataFrame` | `extend` modifies in place. `concat` returns a new frame. | [example](#example-dataframeextend-adding-rows) |
| `tbl.with_rows(list_of_rows)` | `df.extend(pl.from_dicts(list_of_rows))` or `pl.concat([df, other])` | Adds many rows. | DataFrame with the same schema | `DataFrame` | `pl.from_dicts` builds the extra rows from a list of dicts. | [example](#example-dataframeextend-adding-rows) |
| `tbl.row(row_index)` | `df.row(row_index, named=True)` | One row as a dict, or a tuple when `named=False`. | `index`; `named` | `dict` or `tuple` | A single cell from a filtered frame: `df.filter(...).get_column("col").item(0)`. | [example](#example-one-value-from-a-filtered-row) |

### Table.take: head, tail, or gather

`tbl.take` has three Polars forms. Pick one by looking at which rows the index list names.

| The index list names | Use | Data 8 example | Polars example |
| --- | --- | --- | --- |
| the first rows, starting at 0 with no gaps | `df.head(k)` | `top.take(np.arange(5))` | `top.head(5)` |
| one row, row 0 | `df.head(1)` | `nba.take(0)` | `nba.head(1)` |
| the last rows, ending at the last row with no gaps | `df.tail(k)` | `top.take(np.arange(top.num_rows - 3, top.num_rows))` | `top.tail(3)` |
| any other rows: the middle, a list, or every n-th row | `df.gather(indices)` | `nba.take(np.arange(3, 6))` | `nba.gather(np.arange(3, 6))` |

Rules:

- **`head` and `tail` need no gaps.** `np.arange(5)` is rows 0 to 4, so it is `head(5)`. `np.arange(3, 6)` skips rows 0 to 2, so it is not `head`. Use `gather(np.arange(3, 6))` or `df.slice(3, 3)`.
- **`gather` takes a list, `np.arange`, or a Series of positions.** Keep `np.arange` for the index list. `top.take(make_array(3, 18, 100))` is `top.gather([3, 18, 100])`. `top.take(np.arange(start, top.num_rows, 10))` is `top.gather(np.arange(start, top.height, 10))`, because it keeps every tenth row.
- **Keep `head` or `tail` when the result feeds another step.** `head(10)` stays when the chain continues, as in `group_by('Year').len(name='count').sort('Year').head(10).with_columns(...)`. `.show` would end the chain.
- **Use `show(k)` when the cell only displays the first rows.** `sort("SALARY", descending=True).take(np.arange(5))` ends the cell, so it is `.show(5)`. `show` prints and returns `None`.
- **There is no `show` for the last rows.** Use `df.tail(k)` as the last line of the cell. The chapter 1–12 notebooks never take the last rows.
- **Position, not label.** All three work on row positions. A sorted frame gives sorted positions, so sort first and then `head`, as Data 8 does with `sort(...).take(np.arange(k))`.

Notebook cases:

| Data 8 | Polars |
| --- | --- |
| `nba.take(0)` | `nba.head(1)` |
| `nba.take(np.arange(3, 6))` | `nba.gather(np.arange(3, 6))` |
| `sort("SALARY", descending=True).take(np.arange(5))` | `sort("SALARY", descending=True).show(5)` |
| `group('Year').take(np.arange(10)).barh('Year')` | `group_by('Year').len(name='count').sort('Year').head(10)` then the chart |
| `take(np.arange(num_rows - 1))` | `head(height - 1)` |
| `top.take(make_array(3, 18, 100))` | `top.gather([3, 18, 100])` |
| `top.take(np.arange(start, top.num_rows, 10))` | `top.gather(np.arange(start, top.height, 10))` |

### GroupBy methods

`DataFrame.group_by(...)` returns a `GroupBy`. Call one of the methods below. These replace `group_by(...).agg(...)`.

Reference: [GroupBy](https://docs.pola.rs/api/python/stable/reference/dataframe/group_by.html).

| GroupBy method | Data 8 group function | Call |
| --- | --- | --- |
| `.len(name="len")` | `tbl.group(col)` (count rows) | `df.group_by(pl.col("col")).len()` |
| `.sum()` | `sum` | `df.select("col", "value").group_by(pl.col("col")).sum()` |
| `.mean()` | `np.mean` / `np.average` | `df.select("col", "value").group_by(pl.col("col")).mean()` |
| `.min()` | `min` | `df.select("col", "value").group_by(pl.col("col")).min()` |
| `.max()` | `max` | `df.group_by(pl.col("col")).max()` |
| `.median()` | median | `df.select("col", "value").group_by(pl.col("col")).median()` |
| `.quantile(q)` | `percentile(p)` with `q = p / 100` | `df.select("col", "value").group_by(pl.col("col")).quantile(p / 100)` |
| `.n_unique()` | count of distinct values in each column | `df.group_by(pl.col("col")).n_unique()` |
| `.first()` / `.last()` | first or last value in each column | `df.group_by(pl.col("col")).first()` |
| `.head(n)` / `.tail(n)` | first or last `n` rows of each group | `df.group_by(pl.col("col")).head(1)` |
| `.all()` | each column collected into a list | `df.group_by(pl.col("col")).all()` |
| `.having(*predicates)` | keep groups that match, then reduce | `df.group_by(pl.col("col")).having(pl.len() > 1).len()` |
| `.map_groups(function)` | a custom Python function | `df.group_by("col").map_groups(function)` |

Each reducer runs on every column that is not a group key, and the result keeps those column names. Chain `.rename({"value": "total"})` when the new column needs a different label.

From the conversion notebook:

- `.sum()` on a string column raises `InvalidOperationError`. Select the group key and the numeric columns first: `df.select("Flavor", "Price").group_by(pl.col("Flavor")).sum()`.
- `.max()` can run on strings (lexicographic maximum), so `df.group_by(pl.col("Flavor")).max()` keeps `Color` and `Price`.
- `map_groups` takes column **names** as the group keys. `group_by(pl.col("Flavor")).map_groups(...)` is not allowed; write `group_by("Flavor")`.
- Pass `maintain_order=True` when groups should stay in first-seen order, which is what Data 8 does. The notebook omits it.

```python
df.group_by(pl.col("Flavor")).max()
df.select("Flavor", "Price").group_by(pl.col("Flavor")).sum()
```

---

## Visualization Functions

Polars does not plot. Chapter 7 draws with **hvPlot** on the DataFrame. Import `hvplot.polars` next to `import polars as pl`. The chart is a method on `df`, so the column names stay in the call. Data 8 blue is `#002fa7` and gold is `#706700`. The third bar color is `#006a94` and the fourth is `#1e6400`.

A later `plt.title` or `plots.scatter` on the current axes still needs matplotlib. See [visualization_library_choices.md](visualization_library_choices.md).

| Data 8 | hvPlot | Description | Polars input | Polars output | Notes |
| --- | --- | --- | --- | --- | --- |
| `tbl.scatter(x_column, y_column)` | `df.hvplot.scatter(x="x_column", y="y_column")` | One point per row. `x` is the horizontal column and `y` is the vertical. | column names | plot | Chapter 7: `actors.hvplot.scatter(x='Number of Movies', y='Total Gross')`. A filter stays in front: `actors.filter(pl.col('Number of Movies') > 10).hvplot.scatter(...)`. |
| `tbl.plot(x_column, y_column)` | `df.hvplot.line(x="x_column", y="y_column")` | One point per row, connected in order. Same `x` and `y` as scatter. | column names | plot | Chapter 7: `movies_by_year.hvplot.line(x='Year', y='Number of Movies')`. Integer year ticks: `xticks=df.get_column('Year').to_list()`. |
| `tbl.barh(categories, values)` | `df.hvplot.barh("categories", "values")` | Horizontal bars, one per category. Category first, frequency second. | two column names | plot | Chapter 7.1: `icecream.hvplot.barh('Flavor', 'Number of Cartons')`. Sort first when the bars should run longest to shortest. |
| `tbl.barh(categories)` | `df.hvplot.barh("categories", "values")` | Same chart. hvPlot still needs the frequency column. | two column names | plot | One argument used to mean "the other column." Name both columns. |
| `tbl.hist(column)` / `tbl.hist(column, unit=...)` | `df.hvplot.hist("column", bins=10, normed=True, xlabel="column (unit)")` | Density histogram of a numeric column. | column name; `bins`; `normed`; `xlabel` | plot | `hvplot.hist` counts unless `normed=True`, and its default is 20 bins. Data 8 `hist` is a density on 10 bins. `unit` becomes `xlabel`. Chapter 7.2: `millions.hvplot.hist('Adjusted Gross', bins=10, normed=True, xlabel='Adjusted Gross (Million Dollars)')`. |
| `tbl.hist(column, bins=edges)` | `df.hvplot.hist("column", bins=edges, normed=True)` | Histogram with those bin edges. | `bins` as a count or a list of edges | plot | A Polars Series of edges is `.to_list()` before `bins`. Keep `np.arange` for a float edge list. An explicit `normed=False` stays `normed=False`; that figure is counts, not a histogram. |
| `tbl.scatter(x)` / `tbl.plot(x)` | `df.hvplot.scatter(x="x", y=[...])` / `df.hvplot.line(x="x", y=[...])` | Every other column against the common axis. | `x`; `y` as a list of column names; optional `color` | plot | A single axis argument does not mean "every other column." Name the vertical columns in `y`. See [Overlaid graphs](#overlaid-graphs). |
| `tbl.barh(categories)` with several value columns | `df.hvplot.barh("categories", ["c1", "c2", ...], color=[...])` | Several bar charts on the same axes. | category column; list of value columns; `color` | plot | Chapter 7.3 uses the four Data 8 colors in column order. A selected pair uses the first two. |
| `tbl.plot(column_for_xticks=k)` | one matplotlib `plots.plot` per remaining column | Every other column against column `k`. | Series via `get_column` | plot | Chapter 1, not Chapter 7. The next lines are `plt.title` and `plt.show` on the current axes, so this stays matplotlib. |

### Overlaid graphs

Chapter 7.3 draws several series on one pair of axes. Select the columns first, then pass the common axis and a list of the other columns.

```python
df.hvplot.scatter(x="son", y=["father", "mother"], alpha=0.5, color=["#002fa7", "#706700"])
df.hvplot.line(x="AGE", y=["2014", "2019"], color=["#002fa7", "#706700"])
df.hvplot.barh("Ethnicity/Race", ["USA All", "CA All"], color=["#002fa7", "#706700"])
```

- **Scatter and line.** `x` is the shared horizontal column. `y` is the list of vertical columns.
- **Barh.** The first argument is the category column. The second is the list of value columns.
- **Colors.** Father / 2014 / USA All are blue `#002fa7`. Mother / 2019 / CA All are gold `#706700`.
- **Categorical bars that should be equally spaced.** Cast a numeric category to string before `barh`, as in `with_columns(pl.col("Year").cast(pl.String)).hvplot.barh("Year", "count")`. A numeric Year would space 1921 far from 1937.

---

## Table.where Predicates

Filter with a lazy comparison:

```python
df.filter(pl.col("age") > 18)
df.filter((pl.col("age") > 18) & (pl.col("city") == "Berkeley"))
df.filter((pl.col("age") < 18) | (pl.col("age") > 65))
```

| Data 8 predicate | Polars expression | Description | Polars input | Polars output | Notes |
| --- | --- | --- | --- | --- | --- |
| `are.equal_to(Z)` | `pl.col("col") == Z` | Equal to `Z`. | value or Expr | boolean Expr | Column vs column: `pl.col("x") == pl.col("y")` |
| `are.not_equal_to(Z)` | `pl.col("col") != Z` | Not equal to `Z`. | value or Expr | boolean Expr | `pl.col("x") != pl.col("y")` |
| `are.above(x)` | `pl.col("col") > x` | Greater than `x`. | value or Expr | boolean Expr | Notebook: `pl.col("Price") > 5` |
| `are.above_or_equal_to(x)` | `pl.col("col") >= x` | Greater than or equal to `x`. | value or Expr | boolean Expr | |
| `are.below(x)` | `pl.col("col") < x` | Less than `x`. | value or Expr | boolean Expr | |
| `are.below_or_equal_to(x)` | `pl.col("col") <= x` | Less than or equal to `x`. | value or Expr | boolean Expr | |
| `are.between(x, y)` | `pl.col("col").is_between(x, y, closed="left")` | Greater than or equal to `x` and less than `y`. | `lower_bound`, `upper_bound`, `closed` | boolean Expr | <ul><li>`closed="left"`: include the lower bound and exclude the upper bound (`x` <= value < `y`)</li><li>`closed="right"`: exclude the lower bound and include the upper bound (`x` < value <= `y`)</li><li>`closed="both"`: include both bounds (`x` <= value <= `y`)</li></ul> |
| `are.between_or_equal_to(x, y)` | `pl.col("col").is_between(x, y, closed="both")` | From `x` through `y`, inclusive. | `closed="both"` | boolean Expr | Bounds may themselves be `pl.col(...)`. |
| `are.strictly_between(x, y)` | `pl.col("col").is_between(x, y, closed="none")` | Greater than `x` and less than `y`. | `closed="none"` | boolean Expr | |
| `are.contained_in(A)` | `pl.col("col").is_in(A)` | Value is an element of list `A`. | collection or Series | boolean Expr | A string `A` meaning "value is a substring of `A`" needs a small Python predicate via `map_elements`. |
| `are.containing(S)` | `pl.col("col").str.contains(S)` | String contains `S`. | `pattern`; optional `literal` | boolean Expr | `df.filter(pl.col("name").str.contains("Smith"))` |

**Example**

`tbl.where("age", are.above(18))` → `df.filter(pl.col("age") > 18)`

`tbl.where("name", are.containing("Smith"))` → `df.filter(pl.col("name").str.contains("Smith"))`

---

## Conditional Columns

`pl.when().then().otherwise()` builds an if-then-else column. The condition is a `pl.col` comparison. This replaces `np.where` and a row-wise `apply`.

| Data 8 | Polars Equivalent | Description | Polars input | Polars output | Notes | Example |
| --- | --- | --- | --- | --- | --- | --- |
| `tbl.with_column("name", np.where(tbl.column("col") > x, then_val, else_val))` | `pl.when(pl.col("col") > x).then(then_val).otherwise(else_val).alias("name")` | `then_val` where the condition is true, otherwise `else_val`. Extra branches are more `.when().then()` calls; the first match wins. | boolean Expr, then value, otherwise value — [API](https://docs.pola.rs/api/python/stable/reference/expressions/api/polars.when.html) | `Expr` | String results use `pl.lit("x")`. Put the expression in `with_columns` or `select`. | [example](#example-select) |

```python
df.with_columns(
    pl.when(pl.col("col") > x).then(pl.lit("high")).otherwise(pl.lit("low")).alias("band")
)
```

---

## Polars Series and lazy `pl.col` expressions

Two ways to name a column, used for different jobs:

| Job | Write | Why |
| --- | --- | --- |
| A Data 8 array, or a list of values you already have | `pl.Series("x", [1, 2, 3])` | Eager Series. Arithmetic works on it: `3 * pl.Series("x", [1, 2, 3]) + 4`. |
| A column inside `filter`, `with_columns`, `select`, or `group_by` | `pl.col("x")` | Lazy expression. It also runs on `df.lazy()` and still refers to the right column after you reassign `df`. |
| An integer index column | `pl.Series("idx", range(n))` or `pl.int_range(0, pl.len())` | `range` replaces `np.arange` in Python. `pl.int_range` is the lazy form. |
| A float grid | `np.arange(start, stop, step)` | Keep `np.arange`. Do not replace it with `pl.linear_space`. |

```python
# Works on an eager frame and on a LazyFrame
df.lazy().filter(pl.col("Price") > 5).select(pl.col("Flavor"), pl.col("Price")).collect()
```

`df.get_column("Price")` materializes a Series. Use that when you need a Series in Python (a loop over values, `.item(i)`, or a plotting function). Use `pl.col("Price")` when the next step is still a DataFrame method.

---

## Examples

### Example: `pl.Series` vs `pl.concat_list`

`pl.Series("nums", [1, 2, 3])` is one Series of scalars, like `make_array(1, 2, 3)`.

`pl.concat_list` builds a list in each row from other columns. Pass `pl.col` expressions or column-name strings.

```python
import polars as pl

s = pl.Series("nums", [1, 2, 3])
print(s)
print("dtype:", s.dtype)
print("first element:", s.item(0))

df = pl.DataFrame([
    pl.Series("a", [1, 2, 3]),
    pl.Series("b", [10, 20, 30]),
])
out = df.with_columns(
    pl.concat_list([pl.col("a"), pl.col("b")]).alias("pair")
)
print(out)
print("pair dtype:", out.get_column("pair").dtype)
```

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

`out.get_column("pair")` has three elements, and each element is a list: `[1, 10]`, `[2, 20]`, `[3, 30]`.

### Example: Series arithmetic

From the conversion notebook: declare a named Series, then use ordinary arithmetic. The result keeps the first Series' name. Loops use `range`, not `np.arange`.

```python
import polars as pl

print(pl.Series("x", [1, 2, 3]))
print(3 * pl.Series("x", [1, 2, 3]) + 4)
print(pl.Series("x", [1, 2, 3]) + pl.Series("y", [1, 10, 100]))
print(pl.Series("x", [1.0, 4.0, 9.0]).sqrt())

for i in range(4):
    print(i)
```

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

### Example: `.alias`

```python
import polars as pl

df = pl.DataFrame([
    pl.Series("a", [1, 2, 3]),
    pl.Series("b", [10, 20, 30]),
])

# pl.col * 2 is still named "a", so with_columns replaces column a
print(df.with_columns(pl.col("a") * 2))

# .alias adds a new column
print(df.with_columns((pl.col("a") * 2).alias("a_times_2")))

# Keyword argument is the same as .alias("a_times_2")
print(df.with_columns(a_times_2=pl.col("a") * 2))

# A Series you built in Python keeps the name you gave it
print(pl.Series("tmp", [10, 20, 30]).alias("tens"))
```

`pl.col("a") * 2` replaces `a`. The `.alias` and keyword forms add `a_times_2` and leave `a` as 1, 2, 3.

GroupBy methods keep the source column names. Rename afterward:

```python
import polars as pl

grouped = pl.DataFrame({
    "g": ["x", "x", "y"],
    "a": [1, 2, 3],
    "b": [10, 20, 30],
})
print(
    grouped.group_by(pl.col("g"), maintain_order=True)
    .sum()
    .rename({"a": "total_a", "b": "total_b"})
)
```

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

```python
import polars as pl

df = pl.DataFrame([
    pl.Series("foo", [1, 2]),
    pl.Series("bar", [3, 4]),
])
print(df.with_columns((pl.col("foo") + pl.col("bar")).alias("baz")))
```

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

`df.get_column("col")` is Data 8 `tbl.column(col)`: the whole column as a Polars Series. One element is `.item(0)`. Inside `filter`, `with_columns`, and `group_by`, write `pl.col("col")` and keep it lazy.

```python
import polars as pl

df = pl.DataFrame([
    pl.Series("Flavor", ["strawberry", "chocolate", "chocolate"]),
    pl.Series("Price", [3.55, 4.75, 5.25]),
])

print(df.get_column("Price"))
print(df.get_column("Price").item(0))
print(df.filter(pl.col("Price") > 4))
print(df.with_columns((pl.col("Price") * 2).alias("double")))
print(df.group_by(pl.col("Flavor"), maintain_order=True).len())
print(df.filter(pl.col("Flavor") == "strawberry").get_column("Price").item(0))
```

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
│ strawberry ┆ 1   │
│ chocolate  ┆ 2   │
└────────────┴─────┘
3.55
```

`get_column("Price")` is the Series. `.item(0)` is the first element, `3.55`. `filter(pl.col("Price") > 4)` keeps the two chocolate rows. `with_columns` adds `double`. `group_by(pl.col("Flavor"))` counts 1 strawberry and 2 chocolate. After a filter, `.get_column("Price").item(0)` is the strawberry price, `3.55`.

### Example: `.select`

```python
import polars as pl

df = pl.DataFrame([
    pl.Series("foo", [1, 2, 3]),
    pl.Series("bar", [6, 7, 8]),
    pl.Series("ham", ["a", "b", "c"]),
])

out = df.select(
    "foo",
    (pl.col("bar") + 1).alias("bar_plus_1"),
    threshold=pl.when(pl.col("foo") > 2).then(10).otherwise(0),
    foo_times_bar=pl.col("foo") * pl.col("bar"),
)
print(out)
```

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

```python
import polars as pl

cones = pl.DataFrame([
    pl.Series("Flavor", ["strawberry", "chocolate", "chocolate", "strawberry", "chocolate", "bubblegum"]),
    pl.Series("Color", ["pink", "light brown", "dark brown", "pink", "dark brown", "pink"]),
    pl.Series("Price", [3.55, 4.75, 5.25, 5.25, 5.25, 4.75]),
])

print(cones.filter(pl.col("Price") > 5))
```

`Price` above 5 keeps the three rows whose price is 5.25 (two chocolate, one strawberry). The same expression runs lazily:

```python
cones.lazy().filter(pl.col("Price") > 5).select(pl.col("Flavor"), pl.col("Price")).collect()
```

### Example: `df.rename`

```python
import polars as pl

df = pl.DataFrame([
    pl.Series("old_label", [1, 2, 3]),
    pl.Series("x", [4, 5, 6]),
])

df2 = df.rename({"old_label": "new_label"})
print(df)   # original unchanged
print(df2)

df3 = df.rename(lambda name: name.upper())
print(df3)
```

`df2` has columns `new_label` and `x`. `df3` has columns `OLD_LABEL` and `X`. Values are unchanged.

### Example: `group_by` methods

These calls are the conversion notebook, plus `.len()` for Data 8's default count.

```python
import polars as pl

cones = pl.DataFrame([
    pl.Series("Flavor", ["strawberry", "chocolate", "chocolate", "strawberry", "chocolate", "bubblegum"]),
    pl.Series("Color", ["pink", "light brown", "dark brown", "pink", "dark brown", "pink"]),
    pl.Series("Price", [3.55, 4.75, 5.25, 5.25, 5.25, 4.75]),
])

print(cones.group_by(pl.col("Flavor"), maintain_order=True).max())
print(cones.select("Flavor", "Price").group_by(pl.col("Flavor"), maintain_order=True).sum())
print(cones.group_by(pl.col("Flavor"), maintain_order=True).len())
print(cones.group_by(pl.col("Flavor"), maintain_order=True).having(pl.len() > 1).len())

print(cones.pivot("Flavor", index="Color", values="Price", aggregate_function="max"))
```

`.max()` keeps `Color` (string maximum) and `Price`. `.sum()` is called only after `select("Flavor", "Price")`. `.having(pl.len() > 1).len()` keeps flavors that appear more than once. `maintain_order=True` lists strawberry, then chocolate, then bubblegum.

`.max()` on `Price` is 5.25, 5.25, and 4.75. `.sum()` on `Price` is 8.8, 15.25, and 4.75. `.len()` is 2, 3, and 1. The pivot's `dark brown` / `chocolate` cell is 5.25.

A custom Python function uses `map_groups`, with the group key passed as a column name:

```python
cones.select("Flavor", "Price").group_by("Flavor", maintain_order=True).map_groups(
    lambda g: g.select(pl.col("Flavor").first(), pl.col("Price").max())
)
```

Several numeric columns can be summed together, then renamed:

```python
import polars as pl

movies = pl.DataFrame({
    "genre": ["Comedy", "Comedy", "Drama", "Drama", "Comedy"],
    "score": [72, 65, 88, 91, 70],
    "gross": [100.0, 85.0, 120.0, 150.0, 95.0],
})
print(
    movies.group_by(pl.col("genre"), maintain_order=True)
    .sum()
    .rename({"score": "total_score", "gross": "total_gross"})
)
print(
    movies.group_by(pl.col("genre"), maintain_order=True)
    .mean()
    .rename({"score": "avg_score", "gross": "avg_gross"})
)
```

Sums: Comedy 207 and 280.0; Drama 179 and 270.0. Means: Comedy 69.0 and about 93.33; Drama 89.5 and 135.0.

### Example: histogram and bins

`get_column(...).hist(bin_count=10)` is the count table for `tbl.bin` (10 equal-width bins). `pl.col(...).hist(bins=...)` inside `select` returns one count per bin. `bin_intervals` plus `GroupBy.len()` is the form that uses a GroupBy method.

```python
import polars as pl

df = pl.DataFrame([pl.Series("a", [1, 3, 8, 8, 2, 1, 3])])

print(df.select(pl.col("a").hist(bins=[1, 2, 3])))
print(df.get_column("a").hist(bin_count=10))

print(
    df.with_columns(
        pl.col("a").bin_intervals([2, 4], labels=["low", "mid", "high"]).alias("bin")
    ).group_by(pl.col("bin"), maintain_order=True).len()
)
```

`hist(bins=[1, 2, 3])` returns counts 3 and 2. `hist(bin_count=10)` returns `breakpoint`, `category`, and `count` (the non-empty bins are `[1.0, 1.7]` count 2, `(1.7, 2.4]` count 1, `(2.4, 3.1]` count 2, and `(7.3, 8.0]` count 2). The `bin_intervals` counts are low 2, mid 3, high 2.

### Example: `map_elements`

Use this when the calculation cannot be written with `pl.col` arithmetic. Doubling is native: `pl.col("a") * 2`.

```python
import polars as pl

df = pl.DataFrame([pl.Series("a", [1, 2, 3])])

df2 = df.with_columns(
    pl.col("a").map_elements(lambda x: x * 2, return_dtype=pl.Int64).alias("a_times_2")
)
print(df2)
```

Polars warns that `map_elements` is slower than `pl.col("a") * 2`. The column `a_times_2` is 2, 4, 6.

### Example: `DataFrame.extend` adding rows

```python
import polars as pl

tbl = pl.DataFrame([
    pl.Series("a", [1, 2]),
    pl.Series("b", ["x", "y"]),
])
tbl.extend(pl.DataFrame([
    pl.Series("a", [3]),
    pl.Series("b", ["z"]),
]))
print(tbl)

tbl2 = pl.DataFrame([
    pl.Series("a", [1, 2]),
    pl.Series("b", [4, 5]),
])
tbl2.extend(pl.from_dicts([
    {"a": 3, "b": 6},
    {"a": 4, "b": 7},
]))
print(tbl2)
```

`tbl` gains the row `(3, "z")`. `tbl2` gains `(3, 6)` and `(4, 7)`.

### Example: one value from a filtered row

From the lecture example in the conversion notebook. `range(4)` replaces `np.arange(4)`.

```python
import polars as pl

streets = pl.Series("Street", ["Bancroft", "Durant", "Channing", "Haste"])
southside = pl.DataFrame([streets]).with_columns(
    pl.Series("Blocks from campus", range(4))
)
print(southside.filter(pl.col("Street") == "Channing").get_column("Street").item(0))
```

Output:

```text
Channing
```
