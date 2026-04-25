# cursor-skills

My personal collection of [Agent Skills](https://agentskills.io) for Cursor and other agent-skill–compatible coding agents (Claude Code, Codex, OpenCode, etc.).

These skills are project-agnostic and meant to be installed via the [`vercel-labs/skills`](https://github.com/vercel-labs/skills) CLI. Most of them I wrote myself; two (`marimo-notebook`, `marimo-pair`) are vendored copies of the upstream marimo-team skills that I customize for personal use.

## Skills

| Skill | What it does |
|---|---|
| [`bayesian-pymc`](skills/bayesian-pymc/) | Build, review, debug, and explain Bayesian analyses in PyMC + ArviZ, in the style of *Statistical Rethinking*. |
| [`marimo-batch`](skills/marimo-batch/) | Prepare a marimo notebook for a scheduled / batch run. |
| [`marimo-eda`](skills/marimo-eda/) | Build marimo notebooks for exploratory data analysis on tabular data. |
| [`marimo-notebook`](skills/marimo-notebook/) | Write, design, and polish marimo notebooks. (vendored from [`marimo-team/marimo-notebook`](https://github.com/marimo-team/marimo-notebook)) |
| [`marimo-pair`](skills/marimo-pair/) | Work inside a running marimo notebook's kernel — execute code, create cells, build a notebook as an artifact. (vendored from [`marimo-team/marimo-pair`](https://github.com/marimo-team/marimo-pair)) |
| [`jupyter-to-marimo`](skills/jupyter-to-marimo/) | Convert a Jupyter `.ipynb` to a marimo `.py` notebook. |
| [`streamlit-to-marimo`](skills/streamlit-to-marimo/) | Convert a Streamlit app to a marimo notebook. |
| [`wasm-compatibility`](skills/wasm-compatibility/) | Check whether a marimo notebook is WebAssembly-compatible. |
| [`add-molab-badge`](skills/add-molab-badge/) | Add an "Open in molab" badge linking to a marimo notebook. |
| [`anywidget-generator`](skills/anywidget-generator/) | Generate `anywidget` components for marimo notebooks. |
| [`implement-paper`](skills/implement-paper/) | Implement a research paper as an interactive marimo notebook. |

> The `marimo-notebook` and `marimo-pair` skills are not git-forks. They are plain folder copies that I edit freely. There is no automatic upstream sync — if I want a future improvement from `marimo-team`, I have to look at their repo manually and copy the changes I want.

## Install

Requires [`npx`](https://docs.npmjs.com/cli/v10/commands/npx) (ships with Node.js).

### List available skills

```bash
npx skills add broniyasinnik/cursor-skills --list
```

### Install all skills into the current project (recommended)

```bash
cd /path/to/your/project
npx skills add broniyasinnik/cursor-skills --all -a cursor
```

This drops the skills into `./.agents/skills/`. Cursor reads from there automatically. Add `.agents/` to your project's `.gitignore` so the symlinks aren't committed.

### Install a specific skill

```bash
npx skills add broniyasinnik/cursor-skills --skill bayesian-pymc -a cursor
```

### Install globally (every project on this machine)

```bash
npx skills add broniyasinnik/cursor-skills --all -g -a cursor
```

`-g` installs to `~/.cursor/skills/`.

## Update

```bash
npx skills update      # project scope
npx skills update -g   # global scope
```

## List installed skills

```bash
npx skills list
```

## Remove

```bash
npx skills remove --skill bayesian-pymc
```

## Contributing to my own collection

To add a new skill:

1. Create a new folder under `skills/`.
2. Add a `SKILL.md` with valid YAML frontmatter (`name` and `description` are required).
3. Verify it's discovered: `npx skills add . --list`.
4. Commit, push, then `npx skills update` in each project that uses it.

## License

MIT, except where individual skills carry their own LICENSE files (e.g. the vendored marimo-team skills retain their original licenses).
