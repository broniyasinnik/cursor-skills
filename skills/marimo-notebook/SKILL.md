---
name: marimo-notebook
description: write, design, and polish marimo notebooks. use when the user wants to create a marimo notebook, convert a jupyter or script workflow into marimo, add interactive ui, improve notebook structure, or make a notebook look and feel more polished, app-like, reactive, git-friendly, and presentation-ready. covers cell structure, PEP 723 metadata, output rendering, marimo check, variable naming, layout, and best practices for reactivity, layout, readability, and reproducibility.
---

# Marimo Notebook

Build marimo notebooks that are beautiful, reactive, maintainable, and easy to run in Cursor.

marimo notebooks are pure Python, reactive, executable as scripts, and deployable as apps. Favor notebook code that reads like a clean Python module with excellent UX layered on top.

## 1. Notebook Format

marimo uses Python to create notebooks, unlike Jupyter which uses JSON. Here's an example notebook:

```python
# /// script
# dependencies = [
#     "marimo",
#     "numpy==2.4.3",
# ]
# requires-python = ">=3.14"
# ///

import marimo

__generated_with = "0.20.4"
app = marimo.App(width="medium")


@app.cell
def _():
    import marimo as mo
    import numpy as np

    return mo, np


@app.cell
def _():
    print("hello world")
    return


@app.cell
def _(np, slider):
    np.array([1,2,3]) + slider.value
    return


@app.cell
def _(mo):
    slider = mo.ui.slider(1, 10, 1, label="number to add")
    slider
    return (slider,)


@app.cell
def _():
    return


if __name__ == "__main__":
    app.run()

```

Each cell is defined with the `@app.cell` decorator. The inputs/outputs of the function are the inputs/outputs of the cell. marimo takes care of dependencies between cells automatically.

### PEP 723 Dependencies

Notebooks created via `marimo edit --sandbox` have these dependencies added to the top of the file automatically but it is a good practice to make sure these exist when creating a notebook too:

```python
# /// script
# requires-python = ">=3.12"
# dependencies = [
#     "marimo",
#     "torch>=2.0.0",
# ]
# ///
```

## 2. Running Notebooks

```bash
# Run as script (non-interactive, for testing)
uv run <notebook.py>

# Run interactively in browser
uv run marimo run <notebook.py>

# Edit interactively
uv run marimo edit <notebook.py>
```

### Script Mode Detection

Use `mo.app_meta().mode == "script"` to detect CLI vs interactive:

```python
@app.cell
def _(mo):
    is_script_mode = mo.app_meta().mode == "script"
    return (is_script_mode,)
```

**Show all UI elements always.** Only change the data source in script mode.

- Sliders, buttons, widgets should always be created and displayed
- In script mode, just use synthetic/default data instead of waiting for user input
- Don't wrap everything in `if not is_script_mode` conditionals
- Don't use try/except for normal control flow

## 3. Notebook Shapes

First classify the request. Choose one of these shapes:

- **analysis notebook**: exploratory data analysis, model evaluation, experiments, reports
- **app notebook**: dashboards, parameterized tools, interactive demos, internal utilities
- **tutorial notebook**: teaching a concept step by step with narrative and examples
- **monitoring notebook**: recurring refresh, status views, scorecards, metrics
- **conversion task**: transform Jupyter, Streamlit, or a plain Python script into marimo

Use the chosen shape to drive the UI and file structure.

### Analysis notebook
Use a light UI, strong headings, clear sections, and concise controls. Prioritize readability over decoration.

Recommended structure:
- imports and configuration
- data loading
- controls
- transformations
- metrics summary
- visualizations
- detailed tables / drill-down
- notes or caveats

### App notebook
Treat it like a small product. Use sidebars, tabs, stats, and compact controls. Hide complexity behind helper functions.

Recommended structure:
- title and intro
- sidebar or top control panel
- KPI stats
- primary chart area
- supporting tables / diagnostics
- download or export area when relevant

### Tutorial notebook
Use narrative markdown and progressive disclosure. Each section should teach exactly one idea.

Recommended structure:
- title and learning goal
- setup
- concept explanation
- small interactive demo
- interpretation
- next step

### Monitoring notebook
Optimize for quick scanning and safe refresh behavior. Use refresh controls only when they clearly help.

Recommended structure:
- last updated time
- summary stats
- trend views
- anomalies / alerts
- detail table

## 4. Cell Design and Reactivity

### Cell ordering

Prefer this order:
1. imports
2. constants / config
3. pure helper functions
4. data acquisition
5. UI controls
6. filtered / transformed data
7. metrics
8. plots / tables
9. narrative conclusions

### Reactivity rules

- Variables between cells define the reactivity of the notebook for 99% of use-cases. No special state management needed.
- Write idempotent cells. Running the same cell with the same references should produce the same result.
- Minimize mutations. Prefer new variables over mutating objects across cells.
- Do not split declaration and mutation across different cells.
- Use `mo.stop(...)` to gate expensive downstream execution.
- Prefer reactive execution over manual callback-heavy patterns.
- Avoid `on_change` handlers unless there is a clear reason; prefer marimo's built-in reactivity.
- Avoid `mo.state()` unless you need bidirectional UI sync or accumulated callback state. See [STATE.md](references/STATE.md) for details.
- If notebook logic becomes large, move helper code into nearby Python modules and import it.
- Use `@mo.cache` on helper functions to prevent long-running tasks from executing multiple times during reactive updates.

### Naming guidance

- Use descriptive global names. Keep the number of globals small.
- Use names like `sales_df`, `selected_region`, `filtered_orders`, `summary_stats`
- Avoid vague names like `data`, `tmp`, `x`, `result2`
- Temporary one-off values should stay local to a single cell
- Variables in `for` loops that would conflict across cells need underscore prefix:

```python
for _name, _model in items:
    ...
```

You have a tendency to overdo `_prefix` variables though. It's a best practice to add these when a conflict is likely to arise, but overdoing it makes the notebook look unpythonic. Please try and avoid that.

### Cell output rendering

Marimo only renders the **final expression** of a cell. Indented or conditional expressions won't render:

```python
# BAD - indented expression won't render
@app.cell
def _(mo, condition):
    if condition:
        mo.md("This won't show!")  # WRONG - indented
    return

# GOOD - final expression renders
@app.cell
def _(mo, condition):
    result = mo.md("Shown!") if condition else mo.md("Also shown!")
    result  # This renders because it's the final expression
    return
```

### Don't guard cells with `if` statements

Marimo's reactivity means cells only run when their dependencies are ready. Don't add unnecessary guards:

```python
# BAD - the if statement prevents the chart from showing
@app.cell
def _(plt, training_results):
    if training_results:  # WRONG - don't do this
        fig, ax = plt.subplots()
        ax.plot(training_results['losses'])
        fig
    return

# GOOD - let marimo handle the dependency
@app.cell
def _(plt, training_results):
    fig, ax = plt.subplots()
    ax.plot(training_results['losses'])
    fig
    return
```

The cell won't run until `training_results` has a value anyway.

### Don't use try/except for control flow

Don't wrap code in try/except blocks unless you're handling a specific, expected exception. Let errors surface naturally.

```python
# BAD - hiding errors behind try/except
@app.cell
def _(scatter_widget, np, torch):
    try:
        X, y = scatter_widget.widget.data_as_X_y
        X = np.array(X, dtype=np.float32)
        # ...
    except Exception as e:
        return None, None, f"Error: {e}"

# GOOD - let it fail if something is wrong
@app.cell
def _(scatter_widget, np, torch):
    X, y = scatter_widget.widget.data_as_X_y
    X = np.array(X, dtype=np.float32)
    # ...
```

Only use try/except when:
- You're handling a specific, known exception type
- The exception is expected in normal operation (e.g., file not found)
- You have a meaningful recovery action

## 5. Layout and Beauty

Beauty should come from clarity, spacing, hierarchy, and restraint.

### Visual principles

- Start with a strong title and a one- to three-line explanation.
- Use section headings that scan well.
- Group controls together instead of scattering them.
- Put key outputs near the controls that affect them.
- Use whitespace and stacked layouts to create rhythm.
- Keep a single obvious focal area per screen.
- Prefer a small number of high-value charts over many mediocre ones.
- Use tables for drill-down, not as the first thing the user sees.
- End sections with brief interpretation, not just raw output.

### Layout components to prefer

Use these marimo layout APIs where appropriate:

- `mo.vstack` for vertical page composition
- `mo.hstack` for side-by-side panels
- `mo.sidebar` for app-like control panels
- `mo.ui.tabs` for multiple related views
- `mo.stat` for KPI callouts
- `mo.callout` for notes, caveats, and guidance
- `mo.accordion` for optional detail
- `mo.nav_menu` or `mo.outline` for larger notebook navigation
- `mo.lazy` for expensive content that should load later

### UI composition patterns

**Dashboard pattern**
- sidebar with filters
- top row of 3 to 5 stats
- one main chart
- one support chart or table
- optional diagnostics in tabs

**Explorer pattern**
- controls on top
- chart left, selected rows / details right
- detail table below

**Tutorial pattern**
- markdown explanation
- small UI control
- compact output
- takeaway callout

### Narrative polish

Use markdown as product copy, not filler. Good notebook text should:
- explain what the user is seeing
- explain how to interact with controls
- explain how to interpret results
- stay concise

Prefer short markdown blocks over large walls of text.

## 6. Reusable Patterns

### Script mode pattern

```python
# Always show the widget
@app.cell
def _(ScatterWidget, mo):
    scatter_widget = mo.ui.anywidget(ScatterWidget())
    scatter_widget
    return (scatter_widget,)

# Only change data source based on mode
@app.cell
def _(is_script_mode, make_moons, scatter_widget, np, torch):
    if is_script_mode:
        X, y = make_moons(n_samples=200, noise=0.2)
        X_data = torch.tensor(X, dtype=torch.float32)
        y_data = torch.tensor(y)
        data_error = None
    else:
        X, y = scatter_widget.widget.data_as_X_y
        # ... process data ...
    return X_data, y_data, data_error

# Always show sliders - use their .value in both modes
@app.cell
def _(mo):
    lr_slider = mo.ui.slider(start=0.001, stop=0.1, value=0.01)
    lr_slider
    return (lr_slider,)

# Auto-run in script mode, wait for button in interactive
@app.cell
def _(is_script_mode, train_button, lr_slider, run_training, X_data, y_data):
    if is_script_mode:
        results = run_training(X_data, y_data, lr=lr_slider.value)
    else:
        if train_button.value:
            results = run_training(X_data, y_data, lr=lr_slider.value)
    return (results,)
```

### Simple app scaffold

Use this shape for internal tools and dashboards:

```python
import marimo as mo

app = marimo.App()

@app.cell
def _():
    import marimo as mo
    import pandas as pd
    return mo, pd

@app.cell
def _(mo):
    mo.md("""
    # Notebook title
    Short description of what this notebook does.
    """)
    return

@app.cell
def _(mo):
    controls = {
        "category": mo.ui.dropdown(options=["All", "A", "B"], value="All", label="Category"),
        "limit": mo.ui.slider(10, 200, value=50, label="Rows"),
    }
    return (controls,)

@app.cell
def _(controls, mo):
    mo.hstack([controls["category"], controls["limit"]])
    return
```

Adapt this scaffold to the actual task; do not force it when a lighter structure is better.

### Caching expensive computations

Use `@mo.cache` to memoize long-running functions, such as data fetching, heavy transformations, or model training. Because marimo is reactive, `@mo.cache` is fully integrated into the graph: the cache automatically invalidates if the function's arguments change or if any global variables referenced inside the function change.

```python
# GOOD - encapsulate expensive work in a cached function
@app.cell
def _(mo, time):
    @mo.cache
    def load_and_process_data(file_path, threshold):
        # Simulating a long-running task
        time.sleep(5)
        return f"Processed {file_path} with threshold {threshold}"
        
    return (load_and_process_data,)

@app.cell
def _(load_and_process_data, file_picker, threshold_slider):
    # This cell evaluates instantly if the inputs haven't changed
    data = load_and_process_data(file_picker.value, threshold_slider.value)
    return (data,)
```
Caching Guidelines:

The cache lives in memory for the duration of the active kernel session.

To combine caching with UI optimization, place cached functions inside mo.ui.tabs(..., lazy=True) so the computation only triggers when the user views the tab, and is instant on subsequent views.

Do not use @mo.cache for functions that cause side effects (like writing to a database or mutating external state), as they will be skipped on cache hits.

### Expensive computation gate

```python
run = mo.ui.run_button(label="Run analysis")
run
```

```python
mo.stop(not run.value, mo.callout("Click **Run analysis** to compute results."))
```

### Form gate

```python
filters = mo.ui.text(label="Search").form()
filters
```

```python
mo.stop(filters.value is None, mo.md("Submit the form to continue."))
```

## 7. Cursor-Specific Guidance

When the request is for Cursor, optimize for iterative editing by an AI coding assistant and a human developer.

- Keep helper functions near the top or in small adjacent modules.
- Make notebook sections obvious so targeted edits are easy.
- Add brief comments only where they help orientation.
- Avoid over-abstracting simple notebooks.
- Prefer standard plotting libraries and ordinary Python data tools unless the user asks for something else.
- Ensure imports are explicit and easy for Cursor to patch.
- If converting from another notebook format, preserve intent while restructuring for marimo's reactive model.

## 8. Tooling

### marimo check

When working on a notebook it is important to check if the notebook can run. marimo provides a `check` command that acts as a linter to find common mistakes.

```bash
uvx marimo check <notebook.py>
```

Make sure these are checked before handing a notebook back to the user.

### API docs

If the user specifically wants you to use a marimo function, you can locally check the docs via:

```
uv --with marimo run python -c "import marimo as mo; help(mo.ui.form)"
```

### Tests

By default, marimo discovers and executes tests inside your notebook.
When the optional `pytest` dependency is present, marimo runs `pytest` on cells that
consist exclusively of test code - i.e. functions whose names start with `test_`.
If the user asks you to add tests, make sure to add the `pytest` dependency is added and that
there is a cell that contains only test code.

For more information on testing with pytest see [PYTEST.md](references/PYTEST.md)

Once tests are added, you can run pytest from the commandline on the notebook to run pytest.

```
pytest <notebook.py>
```

## 9. Quality Bar

Before finalizing, check:

- Is the notebook obviously useful at first glance?
- Are controls grouped and labeled clearly?
- Does each section earn its space?
- Are globals limited and well named?
- Are expensive steps gated?
- Is the code idiomatic Python, not a tangled notebook script?
- Does the result feel like marimo, not Jupyter with cosmetic changes?

## 10. Additional Resources

- For SQL use in marimo see [SQL.md](references/SQL.md)
- For UI elements in marimo [UI.md](references/UI.md)
- For exposing functions/classes as top level imports [TOP-LEVEL-IMPORTS.md](references/TOP-LEVEL-IMPORTS.md)
- For exporting notebooks (PDF, HTML, markdown, etc.) [EXPORTS.md](references/EXPORTS.md)
- For state management and reactivity [STATE.md](references/STATE.md)
- For deployment of marimo notebooks [DEPLOYMENT.md](references/DEPLOYMENT.md)
- For custom interactive widgets with anywidget [ANYWIDGET.md](references/ANYWIDGET.md)
- For design checklist [marimo-design-checklist.md](references/marimo-design-checklist.md)
- For common patterns [marimo-patterns.md](references/marimo-patterns.md)
- For Cursor prompt templates [cursor-prompt-templates.md](references/cursor-prompt-templates.md)
