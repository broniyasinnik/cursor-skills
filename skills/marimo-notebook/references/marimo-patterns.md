# Marimo Patterns

## Use these components often

- `mo.vstack` for page flow
- `mo.hstack` for balanced side-by-side content
- `mo.sidebar` for filter-heavy notebooks
- `mo.ui.tabs` for multiple result views
- `mo.stat` for KPI summaries
- `mo.callout` for short explanatory notes
- `mo.accordion` for optional details
- `mo.lazy` for expensive or low-priority content

## Good notebook openings

### App-style opening

- Title
- One-sentence description
- Control panel
- KPI row
- Primary chart

### Tutorial opening

- Title
- Learning objective
- Short explanation
- Minimal interactive example
- Key takeaway

### Analysis opening

- Title
- Dataset/context note
- Filters
- High-signal summary chart
- Supporting table

## Conversion heuristics

### Jupyter to marimo

- Replace sequential hidden-state assumptions with explicit dataflow.
- Collapse scratch cells into helper functions.
- Replace imperative rerun instructions with reactive controls.
- Replace long prose gaps with concise markdown sections.

### Streamlit to marimo

- Convert sidebar controls to marimo UI elements.
- Replace callback-driven logic with reactive cell dependencies.
- Reorganize the script into small, readable cells.

## Anti-patterns

Avoid these unless the user explicitly wants them:

- Too many charts on one screen
- Unlabeled controls
- Large mutable globals
- Heavy callback logic when reactivity is enough
- Narrative markdown longer than needed
- Repeated imports in many cells without a reason
