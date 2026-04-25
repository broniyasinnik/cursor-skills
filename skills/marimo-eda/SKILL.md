---
name: marimo-eda
description: build and extend marimo notebooks for exploratory data analysis on tabular data from pandas dataframes or csv/parquet file paths. use when a user wants a full eda workflow with code cells, interactive exploration, chart recommendations, publication-quality tables, or best-practice visualization choices across altair, plotly, bokeh, seaborn, matplotlib, arviz, wigglystuff, great-tables, and marimo widgets. especially useful for profiling columns, distributions, missingness, correlations, group comparisons, outlier checks, interactive filtering, and polished report-ready tables in marimo.
---

# Marimo EDA

Create a complete, practical EDA workflow for **tabular data** in **marimo notebooks**. Prefer code cells the user can run immediately. Accept these starting points:

- a `pandas.DataFrame` already loaded in memory
- a CSV file path
- a parquet file path

Assume **pandas** unless the user explicitly asks otherwise.

## Default workflow

Unless the user asks for something narrower, structure the notebook in this order:

1. Imports and data loading
2. Quick dataset overview
3. Type cleanup and parsing candidates
4. Missingness analysis
5. Univariate distributions
6. Targeted bivariate analysis
7. Grouped summaries for important categorical columns
8. Correlation / association checks
9. Outlier and data quality checks
10. Interactive exploration controls with marimo widgets
11. Concise findings and suggested next questions

Prefer small, focused cells instead of one large block.

## Input handling

Choose the simplest valid entry point:

### If the user provides a DataFrame name
Use it directly and avoid reloading data.

Typical pattern:

```python
import pandas as pd
import marimo as mo

# assume df already exists
preview = df.head()
preview
```

### If the user provides a CSV path
Use `pd.read_csv` with a light-touch approach first. Do not guess too many parsing options up front.

```python
import pandas as pd
import marimo as mo

path = "data.csv"
df = pd.read_csv(path)
df.head()
```

### If the user provides a parquet path

```python
import pandas as pd
import marimo as mo

path = "data.parquet"
df = pd.read_parquet(path)
df.head()
```

### Early normalization
Add a cleanup cell early when useful:

```python
import pandas as pd

# optional cleanup
# df.columns = df.columns.str.strip().str.lower().str.replace(" ", "_")
```

Do not silently mutate important business columns unless the user asked for aggressive cleaning.

## What to inspect first

Always create code that answers these questions early:

- How many rows and columns are there?
- What are the apparent dtypes?
- Which columns have high missingness?
- Which numeric columns are continuous vs low-cardinality encoded categories?
- Which object columns may actually be dates, booleans, IDs, free text, or categories?
- Which columns are likely identifiers and should usually be excluded from distributions/correlations?

Useful starter cell:

```python
summary = pd.DataFrame({
    "dtype": df.dtypes.astype(str),
    "non_null": df.notna().sum(),
    "nulls": df.isna().sum(),
    "null_pct": (df.isna().mean() * 100).round(2),
    "n_unique": df.nunique(dropna=True),
})
summary.sort_values(["null_pct", "n_unique"], ascending=[False, False])
```

## Visualization selection rules

Actively choose the library that best matches the task. Do not rotate through libraries for its own sake.

### Altair: default for declarative tabular EDA
Prefer **Altair** for:

- histograms and density-like distribution views
- bar charts for counts and top categories
- scatter plots with faceting / color encodings
- heatmaps for missingness summaries or aggregated metrics
- linked interactive filtering when lightweight interaction is enough

Why: concise grammar, sensible defaults, strong aggregation support, good fit for tidy tabular exploration.

Use Altair first when the chart is mostly analytic rather than dashboard-like.

### Plotly: default for rich hover / zoom / dashboard-like interaction
Prefer **Plotly** for:

- interactive scatter plots where hover details matter
- line charts with zoom and range navigation
- box/violin charts the user will inspect interactively
- charts where users benefit from hover tooltips for many variables

Why: strong built-in interactivity with minimal setup.

### Bokeh: linked brushing and coordinated views
Prefer **Bokeh** when you need:

- linked selections across multiple plots
- brushing and cross-filtering patterns
- more explicit interactive exploration than Altair provides

Use it selectively; do not choose it for simple static summaries.

### seaborn: statistical defaults and compact pairwise analysis
Prefer **seaborn** for:

- boxplots, violin plots, strip/swarm overlays
- pairplots on a small curated subset of columns
- clustered heatmaps or quick statistical views

Use seaborn when the statistical convention matters more than deep interactivity.

### matplotlib: low-level fallback and exact control
Prefer **matplotlib** for:

- simple fallback charts when another library adds no value
- custom axis handling
- precise control of annotations or subplot layout

Do not default to matplotlib when Altair or Plotly would express the analysis more clearly.

### ArviZ: only for Bayesian inference objects
Use **ArviZ only** if the notebook contains posterior samples, inference data, MCMC diagnostics, posterior predictive checks, or Bayesian model comparison objects. It is **not** a general-purpose tabular EDA library.

### wigglystuff: experimental / highly interactive visuals
Use **wigglystuff** only when the user explicitly wants exploratory experimental interaction that benefits from it, and keep a conventional fallback nearby. Do not make it the primary dependency for standard EDA.

### Great Tables: publication-quality tables

Use **Great Tables** (`great-tables`) when you need to move beyond simple interactive DataFrames and present bespoke, publication-quality tables in reports or presentations. It renders rich HTML tables from pandas or Polars DataFrames via a fluent chaining API.

For quick exploratory inspection, prefer `mo.ui.dataframe(df)` or plain `display()`. Switch to GT when the table itself is a deliverable.

#### When to reach for Great Tables

- **Professional reporting** -- polished tables with titles, subtitles, source notes, footnotes, and row grouping.
- **Contextual data formatting** -- render currencies, dates, percentages, bytes, compact large numbers, markdown, images, flags, or icons in cells without manual string manipulation.
- **Reactive dashboards** -- build tables that update automatically when marimo widget values change.
- **In-cell visualizations** -- embed nanoplots (sparklines or bar charts) directly inside table cells to show trends at a glance.

#### Core table-building pattern

```python
from great_tables import GT, md, html, loc, style, nanoplot_options

(
    GT(df, rowname_col="name")
    .tab_header(
        title="Monthly Revenue by Product",
        subtitle="Q1 2026 — all figures in USD",
    )
    .tab_spanner(label="Revenue", columns=["jan", "feb", "mar"])
    .fmt_currency(columns=["jan", "feb", "mar"], currency="USD")
    .fmt_date(columns="report_date", date_style="wd_m_day_year")
    .tab_source_note(source_note=md("Source: internal billing system."))
    .opt_stylize(style=1, color="blue")
)
```

Always chain methods in this general order: structure (`tab_header`, `tab_stub`, `tab_spanner`) -> formatting (`fmt_*`, `data_color`, `sub_missing`) -> styling (`tab_style`, `opt_stylize`) -> source notes.

#### Formatting methods reference

Pick the formatter that matches the data semantics:

| Method | Use for |
|---|---|
| `fmt_number` / `fmt_integer` | general numerics, counts |
| `fmt_percent` | ratios already in 0-1 or 0-100 range |
| `fmt_currency` | monetary values (supports currency codes and locale) |
| `fmt_date` / `fmt_time` / `fmt_datetime` | temporal columns |
| `fmt_bytes` | file sizes, memory usage |
| `fmt_markdown` | rich text, links in cells |
| `fmt_image` | thumbnail images from file paths |
| `fmt_icon` / `fmt_flag` | inline icons or country flags from codes |
| `data_color` | conditional cell background coloring by value |
| `sub_missing` / `sub_zero` | clean display of nulls and zeros |

Use `fmt_number(columns=..., compact=True)` for large values that benefit from "1.2M" style abbreviations.

#### Nanoplots (in-cell sparklines)

Embed tiny line or bar charts inside cells with `fmt_nanoplot`. Input values can be space-separated strings or list-typed columns.

```python
import pandas as pd
from great_tables import GT, nanoplot_options

spark_df = pd.DataFrame({
    "product": ["Widget A", "Widget B", "Widget C"],
    "monthly_sales": [
        "12 19 14 22 28 31 25",
        "5 8 11 9 13 17 20",
        "30 28 25 22 20 18 15",
    ],
})

(
    GT(spark_df)
    .fmt_nanoplot(
        columns="monthly_sales",
        plot_type="line",
        reference_line="mean",
        autoscale=True,
    )
)
```

Key options:
- `plot_type="bar"` for bar charts (negative values render below a zero line).
- `reference_line` / `reference_area` accept literal values or keywords (`"mean"`, `"median"`, `"min"`, `"max"`, `"q1"`, `"q3"`).
- `autoscale=True` shares the y-axis range across all rows for magnitude comparison.
- Pass `options=nanoplot_options(...)` for fine-grained control over colors, sizes, and element visibility.

#### Marimo-reactive Great Tables

GT tables participate in marimo reactivity. Define widgets in one cell and reference their `.value` in the cell that builds the table.

Widget cell:

```python
import marimo as mo

style_picker = mo.ui.dropdown(
    options=list(range(1, 7)), value=1, label="table style"
)
color_picker = mo.ui.dropdown(
    options=["blue", "cyan", "pink", "green", "red", "gray"],
    value="blue",
    label="accent color",
)
date_range = mo.ui.date_range(label="report period")

mo.hstack([style_picker, color_picker, date_range])
```

Table cell (reacts to widget changes):

```python
from great_tables import GT

filtered = df[
    (df["date"] >= date_range.value[0])
    & (df["date"] <= date_range.value[1])
]

(
    GT(filtered)
    .tab_header(title="Filtered Report")
    .fmt_currency(columns="revenue")
    .opt_stylize(style=style_picker.value, color=color_picker.value)
)
```

You can also embed marimo widgets directly inside a GT table using the `html()` helper to render widget HTML into header, spanner, or source-note positions.

#### Styling and structure helpers

- **Row grouping**: `tab_stub(rowname_col=..., groupname_col=...)` to create row labels and groups.
- **Spanner labels**: `tab_spanner(label=..., columns=...)` to group related columns under a shared header.
- **Column management**: `cols_label(...)` to rename, `cols_hide(...)` to suppress, `cols_move_to_start(...)` / `cols_move_to_end(...)` to reorder.
- **Targeted styling**: combine `tab_style(style=style.fill(...), locations=loc.body(columns=..., rows=...))` for precise cell-level color, font, or border changes.
- **Quick themes**: `opt_stylize(style=N, color=...)` (styles 1-6) for a one-line polished look.

#### What to avoid with Great Tables

- Do not use GT for quick exploratory inspection; `mo.ui.dataframe` or plain display is faster and interactive.
- Do not feed hundreds of rows into a GT table; aggregate or paginate first. GT renders full HTML and is best for curated summaries (roughly <100 rows).
- Do not over-style; keep formatting purposeful and tied to the data semantics.
- Do not mix GT and other table renderers in the same notebook section without a clear reason.

## Marimo widget patterns

Use marimo widgets to make exploration fast without turning the notebook into an app.

Good default widgets:

- dropdown for selecting a column
- multiselect for choosing columns to compare
- slider for histogram bins
- checkbox for log scale
- number input for top-N category display
- dropdown for aggregation statistic
- text input for substring filtering on categorical columns

Typical pattern:

```python
column = mo.ui.dropdown(options=df.columns.tolist(), value=df.columns[0], label="column")
bins = mo.ui.slider(start=5, stop=100, value=30, step=1, label="bins")
log_scale = mo.ui.checkbox(label="log x", value=False)

mo.vstack([column, bins, log_scale])
```

Then reference `column.value`, `bins.value`, and similar widget values in later cells.

### Widget design rules

- Keep controls near the chart they affect.
- Use safe defaults that render immediately.
- Avoid dozens of controls at once; add only the ones that change the analysis meaningfully.
- For high-cardinality categoricals, default to top-N rather than rendering everything.
- When a widget can produce an empty result, handle it gracefully and display a small explanatory message.

## Standard analysis patterns

### Missingness
Start with a sortable summary table. Then choose one visual:

- **Altair** bar chart for null percentage by column
- **Plotly** if hover details or long labels matter
- **seaborn/matplotlib** only for a matrix-style missingness view when needed

### Numeric distributions
For a single numeric column:

- **Altair histogram** by default
- add a log-scale widget if skew is likely
- use **Plotly** when hover/zoom matters
- use **seaborn** for KDE/violin context when comparing groups

Include robust summary stats when useful:

```python
num_stats = df.select_dtypes(include="number").agg(["count", "mean", "median", "std", "min", "max"]).T
num_stats
```

### Categorical columns
For counts and composition:

- **Altair** bar chart by default
- show top-N only when cardinality is high
- include counts and percentages in a table when categories are important

### Numeric vs categorical
For comparing a numeric column across groups:

- **seaborn** boxplot/violinplot for statistical readability
- **Plotly** box/violin if the user benefits from hover and interactive inspection
- optionally overlay sample sizes in a separate summary table

### Numeric vs numeric
For relationships:

- **Altair** scatter with opacity by default
- add color for one meaningful category only
- use **Plotly** for interactive hover
- sample or aggregate if the dataset is very large

### Correlations and associations
For numeric columns:

- compute a curated correlation matrix after excluding obvious IDs and near-constant columns
- show a heatmap with **Altair** or **seaborn**
- do not dump a giant unreadable matrix; choose top correlated pairs or a filtered subset

For categorical association, prefer concise tables or targeted comparisons. Do not force a huge association matrix unless the user asked.

### Outliers
Prefer robust methods and visuals:

- IQR-based flags for quick screening
- boxplots for grouped inspection
- scatter plots for suspicious relationships

Do not label everything far from the mean as an outlier without context.

## Data-quality heuristics

Call out likely issues in code comments or markdown when present:

- duplicate rows
- duplicate keys in likely ID columns
- impossible values
- mixed types in object columns
- date columns stored as strings
- numeric columns encoded as strings with commas or symbols
- category explosion from spelling/case variants
- leakage-prone columns if a target is present

Useful cells:

```python
duplicates = df.duplicated().sum()
duplicates
```

```python
likely_id_cols = [
    c for c in df.columns
    if df[c].nunique(dropna=True) >= 0.95 * len(df)
]
likely_id_cols
```

## Scale and performance guidance

When the table is large:

- avoid pairplots across many columns
- avoid plotting every category in high-cardinality columns
- sample rows for scatter plots when needed, but say so in code comments
- aggregate before plotting where possible
- keep interactive widgets responsive by limiting options to sensible subsets

If row count is very large, prefer summary tables plus aggregated visuals over raw-point plots.

## Notebook output style

Write code cells that are easy to scan. Favor this pattern:

- one cell for controls
- one cell for filtered/derived data
- one cell for the chart
- one cell for a compact interpretation table or note

When adding markdown explanations, keep them short and analytical.

## What to avoid

- Do not use every plotting library in one notebook just to showcase them.
- Do not produce giant pairplots, giant heatmaps, or all-columns-at-once dashboards by default.
- Do not use ArviZ unless Bayesian objects are actually present.
- Do not make wigglystuff a hard requirement for normal EDA.
- Do not use Great Tables for quick exploratory inspection; prefer `mo.ui.dataframe` or plain display for that.
- Do not over-clean the dataset before the user has seen the raw shape of the data.
- Do not choose charts that hide uncertainty, sample size, or skew when those are central.

## Final deliverable pattern

When building the notebook, aim for:

1. runnable imports and load cell
2. compact structural summary table
3. targeted charts chosen with the rules above
4. marimo widgets for the highest-value comparisons
5. brief findings cell summarizing the most notable patterns, caveats, and next analyses

If the user asks for "full EDA", return a complete notebook-style sequence of cells, not just high-level advice.
