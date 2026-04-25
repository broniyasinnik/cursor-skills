# Marimo Design Checklist

Use this checklist before finalizing a notebook.

## Structure

- The notebook has a clear goal: analysis, app, tutorial, monitoring, or conversion.
- Imports, config, helpers, data, controls, results, and interpretation appear in a sensible order.
- The notebook can be edited comfortably in Cursor.

## Reactivity

- Cells are idempotent.
- Expensive downstream work is gated with `mo.stop`, forms, or run buttons when needed.
- Mutable state is minimized.
- Cross-cell mutations are avoided.
- Reactive dependencies are clear.

## UX

- The title explains the notebook purpose immediately.
- Controls are grouped and labeled well.
- The primary output is obvious.
- Secondary detail is hidden behind tabs, accordions, or lower on the page.
- Markdown explains interaction and interpretation.

## Visual design

- Layout uses `mo.vstack` and `mo.hstack` intentionally.
- `mo.sidebar`, `mo.stat`, `mo.callout`, and `mo.ui.tabs` are used only when they improve clarity.
- There is enough whitespace and separation between sections.
- The notebook avoids clutter and duplicated outputs.

## Code quality

- Global names are descriptive and limited in number.
- Pure helper functions are extracted where helpful.
- Comments are sparse and useful.
- The final file is easy to diff in git.
