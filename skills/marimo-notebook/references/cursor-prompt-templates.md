# Cursor Prompt Templates

Use or adapt these prompts when helping generate marimo notebooks.

## Create a notebook from scratch

Create a polished marimo notebook in a single `.py` file for [goal].

Requirements:
- use marimo idioms and reactive execution
- keep globals limited and descriptive
- build a clean, app-like layout
- use `mo.vstack` / `mo.hstack` and other layout helpers only where they improve clarity
- gate expensive work with `mo.stop` or a run control when needed
- include concise markdown that explains how to use the notebook
- make the result friendly to edit in Cursor

## Convert an existing notebook

Convert this notebook into marimo.

Requirements:
- preserve the core analytical intent
- remove hidden-state assumptions
- refactor into clear reactive cells
- improve the UX and visual hierarchy
- use marimo controls and layout helpers appropriately
- keep the output as a clean `.py` marimo notebook

## Polish an existing marimo notebook

Refactor this marimo notebook to make it more beautiful and maintainable.

Focus on:
- better section hierarchy
- clearer controls
- improved layout and spacing
- cleaner helper functions
- fewer unnecessary globals
- concise narrative markdown
