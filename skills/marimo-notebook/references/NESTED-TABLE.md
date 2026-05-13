# NestedTable (wigglystuff) in marimo

`NestedTable` from [`wigglystuff`](https://github.com/koaning/wigglystuff) is an **anywidget** that renders a recursive, expandable table for hierarchical data. Each row shows the node name plus one column per numeric value key (or a single `Value` column for scalar leaves). Optional **share-of-root** percentage columns sit next to selected value columns.

Add to PEP 723 when needed:

```python
# dependencies = [
#     "marimo",
#     "wigglystuff>=0.4.0",
# ]
```

Official API reference: [nested-table.md](https://koaning.github.io/wigglystuff/reference/nested-table.md).

## When to use it

Prefer `NestedTable` when **all** of these apply:

1. **Real hierarchy** — e.g. team → project → person, region → country → city, dept → team, taxonomy, path-shaped keys.
2. **Additive metrics** — counts, hours, revenue, tickets, tokens so roll-ups and `% of root` make sense.
3. **Progressive disclosure** — drill-down beats a long flat scroll.

## When not to use it

| Situation | Prefer instead |
|-----------|----------------|
| Flat tabular exploration | `mo.ui.dataframe(df)` or `mo.ui.table(...)` |
| Sort/filter only, no parent/child meaning | `mo.ui.table` / `mo.ui.dataframe` |
| Publication-quality static tables | Great Tables (`great-tables`) — see `marimo-eda` skill |
| Non-additive metrics (rates, conversion %, ratios) | Flat table; if you still use `NestedTable`, set `show_percent=False` |

Marimo has no separate built-in “nested grid” widget; hierarchical **tabular** drill-down is the sweet spot for `NestedTable`. For plain tree labels without multi-column metrics, other UI patterns may suffice.

## marimo integration

Always wrap the widget so marimo tracks it:

```python
nested_widget = mo.ui.anywidget(NestedTable.from_dataframe(...))
nested_widget
```

Synced traitlets useful downstream:

- `nested_widget.selected_path` — `list[str]` path of the last clicked row name segment chain (reactive across cells).
- `nested_widget.expanded_paths` — list of expanded paths.

Use these as normal reactive dependencies in later cells (no `observe` callbacks needed unless you want imperative hooks).

## Pattern A — tidy dataframe (most common)

```python
from wigglystuff import NestedTable

nested_widget = mo.ui.anywidget(
    NestedTable.from_dataframe(
        sales_df,
        path_cols=["region", "country", "city"],
        value_cols=["revenue", "orders"],
        root_name="sales",
        format={"revenue": lambda v: f"${v:,.0f}", "orders": lambda v: f"{int(v):,}"},
        show_percent=["revenue"],
        initial_expand_depth=1,
    )
)
nested_widget
```

`value_cols` can be a single string, a list (column order), or `None` to auto-detect numeric fields. Works with **pandas** or **polars** (anything supporting the helpers in wigglystuff).

## Pattern B — path keys to leaf dicts

Use when you already have `{ "a/b/c": {"hours": 12.5, "count": 3}, ... }`:

```python
nested_widget = mo.ui.anywidget(
    NestedTable.from_paths(
        {
            "analytics/cluster/Agg": {"hours": 39.5, "count": 12},
            "animate/Easing": {"hours": 84.0, "count": 24},
        },
        sep="/",
        root_name="projects",
        format={"hours": lambda v: f"{v:.1f}h", "count": lambda v: f"{int(v)}"},
        show_percent=["hours"],
        initial_expand_depth=2,
    )
)
nested_widget
```

## Pattern C — drill-down into detail / chart

A later cell can branch on `selected_path` (empty until the user clicks a row):

```python
path_segments = nested_widget.selected_path
mo.stop(
    len(path_segments) == 0,
    mo.callout("Click a row in the nested table to filter the detail view."),
)
# Example: filter leaves whose path prefix matches
prefix = "/".join(path_segments)
detail_df = leaves_df[leaves_df["path_key"].str.startswith(prefix)]
```

Adapt filtering to your schema (`path_cols` vs string paths).

## `show_percent`

- Default constructor behavior enables share columns broadly; **prefer an explicit list** (`show_percent=["revenue"]`) whenever some columns are not additive or percentages would mislead.
- Only list columns that exist in `value_cols` / auto-detected columns; unknown names raise `ValueError`.
- Use `show_percent=False` for scalar-only trees or when percentages add noise.

## `format`

- Pass a single callable for all numeric cells, or a **`dict[str, Callable]`** per column.
- Formatters receive the **aggregated** numeric value at parent nodes after roll-up, not only leaf values.

## `initial_expand_depth`

- **`1`** — safe default for deep trees (`>3` levels).
- **`2`** — reasonable for shallow hierarchies when you want context without opening everything.
- Avoid expanding all levels for large trees (noise and layout cost).

## Common pitfalls

1. **No `mo.ui.anywidget` wrapper** — widget may not participate cleanly in marimo’s reactive UI model.
2. **`show_percent=True` with mixed metric types** — hides misleading `%` on non-additive columns by using an explicit column list or `False`.
3. **Huge depth + high `initial_expand_depth`** — cluttered first paint.
4. **Missing dependency** — add `wigglystuff` to PEP 723 / project env.
5. **`selected_path` before interaction** — guard with `mo.stop` or default branch so downstream cells do not assume a selection exists.
